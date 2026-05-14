# Plan: Chuyển hệ thống notification từ polling sang webhook

Hiện tại worker poll DB mỗi 5 phút để gửi email/push, gây trễ ~3 phút. Chuyển sang webhook đẩy realtime, giảm latency xuống <5s và giảm 60% query DB.

## Bối cảnh

Hệ thống `notification-worker` hiện đang chạy cron mỗi 5 phút, scan bảng `events` tìm row có `status = 'pending'` rồi gửi notification. Cách này có 3 vấn đề:

- Latency end-to-end trung bình 2.5 phút, p99 lên đến 6 phút.
- Mỗi lần scan đọc ~50k row, tốn 30% CPU của DB primary.
- Khi traffic spike, queue đầy → 1 vài event bị xử lý sau 30 phút.

Quý này team được giao KPI giảm notification latency xuống <10s. Đây là lý do của plan này.

## Mục tiêu

Sau khi rollout xong:

- p99 latency của notification: từ 360s → <5s
- DB query giảm 60% (đo bằng `pg_stat_statements`)
- Không có downtime, không mất event nào trong quá trình migrate

## Kiến trúc đề xuất

Producer (order-service, payment-service, …) emit event qua Kafka. Notification-router subscribe topic `events.*`, route đến đúng channel (email/push/SMS) qua webhook tới notification-worker. Worker gửi notification và ghi audit log vào Postgres.

Lưu ý: webhook giữa router và worker chạy nội bộ (cùng VPC), không qua public internet.

## Các bước thực hiện

1. **Tuần 1: Setup Kafka topic + router skeleton.** Tạo topic `events.notification` với 12 partition, retention 7 ngày. Deploy notification-router stub chỉ log message, chưa gửi đi đâu.

2. **Tuần 2: Migrate producer.** Sửa order-service và payment-service để emit event vào Kafka song song với việc insert vào bảng `events` (double-write). Verify event count khớp nhau.

3. **Tuần 3: Wire webhook router → worker.** Notification-router gọi webhook tới worker. Worker chạy paralel với cron cũ — cron vẫn quét bảng `events` nhưng skip row đã được webhook xử lý.

4. **Tuần 4: Cutover.** Tắt cron job, monitor 48h. Nếu ổn, xoá cột `status` khỏi bảng `events` (giữ bảng cho audit).

5. **Tuần 5: Cleanup.** Drop bảng `events` legacy. Tạo Grafana dashboard mới cho metric latency.

## Lựa chọn công nghệ message bus

Có 3 option chính:

### Kafka

Ưu điểm: throughput cao (đã có cluster sẵn), retention 7 ngày cho phép replay khi worker fail. Nhược: team chưa quen consumer group rebalancing.

### RabbitMQ

Đơn giản, dễ ops. Nhưng không có replay, mất message nếu consumer chết trước khi ack.

### AWS SQS

Managed, không lo về ops. Nhưng cost ~$0.4/M message, với 50M/tháng = $20/tháng, ổn. Vendor lock-in nhẹ.

Quyết định: dùng Kafka vì đã có infra sẵn và cần replay capability.

## Rủi ro

Có 2 rủi ro chính cần lưu ý:

Webhook giữa router và worker có thể bị mất nếu worker restart đúng lúc nhận request. Phải implement idempotency key + retry exponential backoff ở router.

Double-write giai đoạn tuần 2 có thể gây inconsistency nếu Kafka emit fail nhưng DB insert success. Mitigation: dùng transactional outbox pattern.

KHÔNG được drop bảng `events` trước tuần 5 vì nếu rollback cần dữ liệu này.

## Ưu / Nhược webhook approach

Ưu điểm:
- Realtime, không phải poll
- Giảm tải DB drastically
- Scale ngang theo partition Kafka

Nhược điểm:
- Cần xử lý retry/idempotency cẩn thận
- Debug khó hơn cron (state nằm ở queue)
- Phải maintain 2 codepath trong 4 tuần

## Chi tiết kỹ thuật retry policy

Khi webhook fail, router retry theo exponential backoff: 1s, 4s, 16s, 64s, 256s, 1024s. Sau 6 lần fail, push event vào dead-letter topic `events.dlq` và alert oncall. Idempotency key = `sha256(event_id + channel + recipient)`, store ở Redis TTL 24h.

Tham khảo: https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/

## Tổng kết

Chuyển sang webhook cắt giảm 99% latency và 60% DB load, đổi lại độ phức tạp tăng vừa phải. Plan 5 tuần với 1 tuần monitoring trước cutover đảm bảo rollback an toàn.
