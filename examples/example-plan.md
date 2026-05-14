# Plan: Migrating the notification system from polling to webhooks

The worker currently polls the database every 5 minutes to send email and push notifications, causing ~3 minutes of end-to-end latency. Switching to webhooks pushes notifications in real time, dropping latency below 5s and cutting DB load by 60%.

## Background

The `notification-worker` runs a cron job every 5 minutes that scans the `events` table for rows with `status = 'pending'` and dispatches notifications. This approach has three problems:

- End-to-end latency averages 2.5 minutes, with p99 reaching 6 minutes.
- Each scan reads ~50k rows, consuming 30% of CPU on the DB primary.
- During traffic spikes the queue backs up — some events take 30+ minutes to process.

This quarter the team has a KPI to drop notification latency below 10s. That's the motivation for this plan.

## Goals

After rollout:

- p99 notification latency: 360s → <5s
- DB query load reduced by 60% (measured via `pg_stat_statements`)
- Zero downtime, zero event loss during migration

## Proposed architecture

Producers (order-service, payment-service, …) emit events to Kafka. The `notification-router` subscribes to topic `events.*`, routes each event to the appropriate channel (email/push/SMS) via a webhook to `notification-worker`. The worker delivers the notification and writes an audit log to Postgres.

Note: the webhook between router and worker runs internally inside the same VPC — it does not traverse the public internet.

## Implementation steps

1. **Week 1: Set up Kafka topic + router skeleton.** Create topic `events.notification` with 12 partitions and 7-day retention. Deploy a `notification-router` stub that only logs incoming messages — no real dispatch yet.

2. **Week 2: Migrate producers.** Update `order-service` and `payment-service` to emit events to Kafka in parallel with inserting into the `events` table (double-write). Verify event counts match.

3. **Week 3: Wire webhook router → worker.** The router calls the webhook on the worker. The worker runs in parallel with the existing cron — cron still scans `events` but skips rows already processed by the webhook.

4. **Week 4: Cutover.** Stop the cron job, monitor for 48h. If stable, drop the `status` column from `events` (keep the table itself for audit).

5. **Week 5: Cleanup.** Drop the legacy `events` table. Create a new Grafana dashboard for the latency metric.

## Message bus choice

Three main candidates:

### Kafka

High throughput (we already have a cluster), and 7-day retention enables replay when the worker fails. Downside: the team is not yet familiar with consumer-group rebalancing.

### RabbitMQ

Simple and easy to operate. But no replay, and messages can be lost if a consumer dies before acking.

### AWS SQS

Managed, no ops burden. Costs about $0.4 per million messages — roughly $20/month at 50M messages. Slight vendor lock-in.

Decision: go with Kafka because the infrastructure already exists and we need replay capability to avoid event loss during worker restarts.

## Risks

There are two main risks to call out.

The webhook between router and worker can be lost if the worker restarts mid-request. We must implement idempotency keys plus exponential-backoff retry in the router.

The double-write phase in week 2 can produce inconsistency if a Kafka emit fails after the DB insert succeeds. Mitigation: use the transactional outbox pattern.

DO NOT drop the `events` table before week 5. If we need to roll back at week 4, that data is our only restore source.

## Pros and cons of the webhook approach

Pros:
- Real-time delivery, no polling overhead
- DB load drops drastically
- Scales horizontally with Kafka partitions

Cons:
- Retry and idempotency need careful handling
- Harder to debug than cron (state lives in the queue)
- Two code paths to maintain for four weeks

## Technical detail: retry policy

When a webhook call fails, the router retries with exponential backoff: 1s, 4s, 16s, 64s, 256s, 1024s. After six failures the event is pushed to the dead-letter topic `events.dlq` and oncall is paged. The idempotency key is `sha256(event_id + channel + recipient)`, stored in Redis with a 24-hour TTL.

Reference: https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/

## Summary

Switching from polling to webhooks cuts 99% of latency and 60% of DB load in exchange for a moderate increase in operational complexity. A five-week plan with one week of monitoring before cutover keeps rollback safe.
