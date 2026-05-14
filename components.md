# md2html — Component Catalog

This file is the **single source of truth** for the HTML snippets you (the AI) must use when filling `template.html`.

**Rules:**
- Copy snippets verbatim, only replace the bracketed `{{...}}` placeholders.
- Never invent new CSS classes — every visual element MUST be one of these components or vanilla markdown HTML (`<h2>`, `<p>`, `<ul>`, etc.).
- All sample text in this catalog is illustrative — replace with real content from the source `.md`.
- **Language follows the source**: if source `.md` is English, output English UI labels; if Vietnamese, output Vietnamese. See the label table below. For other languages, use the equivalent translations.
- **Use SVG icons via the sprite, never emojis.** All icons reference IDs defined in `template.html`'s `<svg class="icon-sprite">`. Form: `<svg class="icon" viewBox="0 0 24 24" aria-hidden="true"><use href="#i-NAME"/></svg>`. See §13 for the full catalog of available icon IDs.

## Language label table

The HTML's `<html lang="...">` attribute MUST be set to match the source language (ISO 639-1: `en`, `vi`, `zh`, `ja`, `ko`, `es`, `fr`, `de`, `ru`, `ar`, `th`, …). The "Recommended" badge label is set via the `--rec-label` CSS variable on `<html>` — any language works without CSS edits.

```html
<!-- example: Chinese source -->
<html lang="zh" data-theme="light" style="--rec-label: '★ 推荐'">

<!-- example: Japanese source -->
<html lang="ja" data-theme="light" style="--rec-label: '★ 推奨'">
```

### Common UI labels by language

| Key                | EN                  | VI                  | ZH (中文)         | JA (日本語)         | KO (한국어)         | ES                   | FR                    | DE                  |
|---                 |---                  |---                  |---                |---                  |---                  |---                   |---                    |---                  |
| `{{TOC_TITLE}}`    | Contents            | Mục lục             | 目录              | 目次                | 목차                | Contenido            | Sommaire              | Inhalt              |
| `{{REC_LABEL}}`    | ★ Recommended       | ★ Đề xuất           | ★ 推荐            | ★ 推奨              | ★ 추천              | ★ Recomendado        | ★ Recommandé          | ★ Empfohlen         |
| `{{PRINT_TOOLTIP}}`| Print / Save PDF    | In / Lưu PDF        | 打印 / 保存 PDF   | 印刷 / PDF 保存     | 인쇄 / PDF 저장     | Imprimir / Guardar   | Imprimer / PDF        | Drucken / PDF       |
| `{{THEME_TOOLTIP}}`| Toggle theme        | Đổi theme           | 切换主题          | テーマ切替          | 테마 전환           | Cambiar tema         | Changer thème         | Theme wechseln      |
| Read-time suffix   | ~N min read         | ~N phút đọc         | ~N 分钟阅读       | ~N 分で読了         | ~N분 소요           | ~N min de lectura    | ~N min de lecture     | ~N Min. Lesezeit    |
| Highlight label    | Key point           | Ý chính             | 要点              | 要点                | 핵심                | Idea clave           | Point clé             | Kernpunkt           |
| Pros heading       | ✓ Pros              | ✓ Ưu điểm           | ✓ 优点            | ✓ 長所              | ✓ 장점              | ✓ Ventajas           | ✓ Avantages           | ✓ Vorteile          |
| Cons heading       | ✕ Cons              | ✕ Nhược điểm        | ✕ 缺点            | ✕ 短所              | ✕ 단점              | ✕ Desventajas        | ✕ Inconvénients       | ✕ Nachteile         |
| Source: prefix     | Source:             | Nguồn:              | 来源:             | ソース:             | 소스:               | Fuente:              | Source :              | Quelle:             |
| Brand — PLAN       | Plan                | Kế hoạch            | 计划              | 計画                | 계획                | Plan                 | Plan                  | Plan                |
| Brand — SPEC       | Spec                | Đặc tả              | 规格              | 仕様                | 사양                | Especificación       | Spécification         | Spezifikation       |
| Brand — SYSTEM DESIGN | System Design    | Thiết kế hệ thống   | 系统设计          | システム設計        | 시스템 설계         | Diseño de sistema    | Conception système    | Systemdesign        |
| Brand — RFC / RUNBOOK / POSTMORTEM | (keep) | (keep)            | (keep)            | (keep)              | (keep)              | (keep)               | (keep)                | (keep)              |
| Brand — NOTES      | Notes               | Ghi chú             | 笔记              | メモ                | 메모                | Notas                | Notes                 | Notizen             |

### Default callout titles (override with source-specific phrases when possible)

| Variant                  | EN                | VI                | ZH                | JA                | KO                | ES                  |
|---                       |---                |---                |---                |---                |---                |---                  |
| info                     | Context           | Bối cảnh          | 背景              | 背景              | 배경              | Contexto            |
| warn                     | Heads up          | Cảnh báo          | 注意              | 注意              | 주의              | Atención            |
| danger                   | Do not do this    | Không được làm    | 禁止操作          | 禁止               | 금지              | No hacer            |
| success                  | Done              | Đã hoàn thành     | 已完成            | 完了              | 완료              | Hecho               |
| decision                 | Decision          | Quyết định        | 决定              | 決定              | 결정              | Decisión            |
| tip                      | Tip               | Gợi ý             | 提示              | ヒント            | 팁                | Consejo             |

**Notes:**
- For a language not listed, translate using the same conventions — every label has a natural equivalent.
- The "Doc-type eyebrow" (`PLAN`, `SPEC`, `RFC`, `RUNBOOK`, `POSTMORTEM`, `NOTES`) stays as the uppercase English code in **all languages** — it's a universal tag, not a translatable label.
- Body content (paragraphs, step descriptions, callout bodies, headings H2/H3) is always in the source language — paraphrased but never translated to another language.
- CJK languages (zh/ja/ko) inherit `font-family: var(--font-sans)` which falls back to system fonts that include CJK glyphs (PingFang on macOS, Microsoft YaHei on Windows, Noto Sans CJK on Linux). No font change needed for correct rendering.

---

## 1. Title block (in `<header class="doc-header">`)

Replace `{{TITLE}}`, `{{SUBTITLE}}`, `{{DOC_TYPE}}`, `{{SOURCE_FILE}}`, `{{DATE}}`, `{{READ_TIME}}` in `template.html`.

- `{{DOC_TYPE}}` examples: `PLAN`, `SPEC`, `SYSTEM DESIGN`, `RFC`, `NOTES`, `RUNBOOK`, `POSTMORTEM`.
- `{{READ_TIME}}` format: `~5 phút đọc` (estimate ~250 words/minute).

---

## 2. TOC entry — goes inside `<nav class="toc-nav">`

```html
<a href="#section-id" class="lvl-2">Tên section</a>
<a href="#subsection-id" class="lvl-3">Tên sub-section</a>
```

- One `<a>` per H2/H3 in the document.
- The `href` must match an `id="..."` on the target heading: `<h2 id="section-id">...</h2>`.
- Use `class="lvl-2"` for top-level (H2), `class="lvl-3"` for nested (H3). Skip H4 to avoid clutter.

---

## 3. Step card / Timeline

For numbered sequences (action items, plan steps, migration steps, workflow). Wrap multiple `.step` in a `.timeline`.

```html
<div class="timeline">
  <article class="step">
    <div class="step-num">1</div>
    <div class="step-body">
      <h3>Tiêu đề bước</h3>
      <p>Mô tả ngắn gọn 1-2 câu về việc cần làm và lý do.</p>
      <div class="step-tags">
        <span class="tag">backend</span>
        <span class="tag">~2 ngày</span>
      </div>
    </div>
  </article>
  <article class="step">
    <div class="step-num">2</div>
    <div class="step-body">
      <h3>Bước tiếp theo</h3>
      <p>...</p>
    </div>
  </article>
</div>
```

- Add `class="step done"` to mark a completed step (circle filled).
- Tags are optional — use for owner, ETA, area, dependencies.
- Keep `<h3>` short (≤ 60 chars); detail goes in `<p>`.

---

## 4. Callouts

Use for important asides. Pick the variant that matches semantic meaning.

### 4a. Info — context, background, FYI
```html
<aside class="callout callout-info">
  <svg class="callout-icon" viewBox="0 0 24 24" aria-hidden="true"><use href="#i-info"/></svg>
  <div class="callout-body">
    <p class="callout-title">Bối cảnh</p>
    <p>Hệ thống hiện tại dùng cron 5 phút, gây trễ end-to-end ~3 phút.</p>
  </div>
</aside>
```

### 4b. Warning — gotcha, edge case, risk
```html
<aside class="callout callout-warn">
  <svg class="callout-icon" viewBox="0 0 24 24" aria-hidden="true"><use href="#i-warn"/></svg>
  <div class="callout-body">
    <p class="callout-title">Cảnh báo</p>
    <p>Migration này khoá bảng <code>orders</code> ~30s khi chạy production.</p>
  </div>
</aside>
```

### 4c. Danger — blocker, breaking change, must-not-do
```html
<aside class="callout callout-danger">
  <svg class="callout-icon" viewBox="0 0 24 24" aria-hidden="true"><use href="#i-danger"/></svg>
  <div class="callout-body">
    <p class="callout-title">Không được làm</p>
    <p>Tuyệt đối không drop column trước khi rollout deploy cũ.</p>
  </div>
</aside>
```

### 4d. Success — confirmation, what's already done
```html
<aside class="callout callout-success">
  <svg class="callout-icon" viewBox="0 0 24 24" aria-hidden="true"><use href="#i-success"/></svg>
  <div class="callout-body">
    <p class="callout-title">Đã hoàn thành</p>
    <p>API mới đã được test load 10k RPS, p99 = 80ms.</p>
  </div>
</aside>
```

### 4e. Decision — recorded decision / ADR
```html
<aside class="callout callout-decision">
  <svg class="callout-icon" viewBox="0 0 24 24" aria-hidden="true"><use href="#i-decision"/></svg>
  <div class="callout-body">
    <p class="callout-title">Quyết định</p>
    <p>Chọn Postgres thay vì MongoDB vì cần transaction ACID cho luồng thanh toán.</p>
  </div>
</aside>
```

### 4f. Tip — recommendation, best practice
```html
<aside class="callout callout-tip">
  <svg class="callout-icon" viewBox="0 0 24 24" aria-hidden="true"><use href="#i-tip"/></svg>
  <div class="callout-body">
    <p class="callout-title">Gợi ý</p>
    <p>Cache kết quả truy vấn này 5 phút để giảm 80% tải DB.</p>
  </div>
</aside>
```

### 4g. Security — using lock icon for auth/security notes
```html
<aside class="callout callout-info">
  <svg class="callout-icon" viewBox="0 0 24 24" aria-hidden="true"><use href="#i-lock"/></svg>
  <div class="callout-body">
    <p class="callout-title">Bảo mật</p>
    <p>Webhook chạy nội bộ trong VPC, không qua public internet.</p>
  </div>
</aside>
```

---

## 5. Key-point highlight

For the single most important insight/conclusion of a section.

```html
<!-- VI -->
<div class="highlight">
  <span class="highlight-label">Ý chính</span>
  <p>Tóm lại: chuyển từ polling sang webhook giảm độ trễ từ 3 phút xuống &lt;5s và tiết kiệm 60% lượng API call.</p>
</div>

<!-- EN -->
<div class="highlight">
  <span class="highlight-label">Key point</span>
  <p>Bottom line: moving from polling to webhooks cuts latency from 3 minutes to &lt;5s and saves 60% of API calls.</p>
</div>
```

- Use sparingly (≤ 1 per major section). If everything is highlighted, nothing is.
- The `highlight-label` text comes from the language label table above.

---

## 6. Mermaid diagram

Detect these patterns in source `.md` and convert them:
- "flow / luồng / step A → step B → step C" → `graph LR` or `flowchart TD`
- "client gọi server, server gọi DB" → `sequenceDiagram`
- "table A có FK đến table B" → `erDiagram`
- "state machine / trạng thái" → `stateDiagram-v2`
- "phases / roadmap / timeline" → `gantt`

```html
<figure class="diagram">
  <pre class="mermaid">
flowchart LR
  A[User] -->|POST /order| B(API Gateway)
  B --> C{Auth?}
  C -->|yes| D[Order Service]
  C -->|no| E[401]
  D --> F[(Postgres)]
  D --> G[/Event bus/]
  </pre>
  <figcaption class="diagram-caption">Luồng tạo đơn hàng từ user đến event bus.</figcaption>
</figure>
```

**Mermaid tips for readable output:**
- Prefer `LR` (left-right) for flows with ≤ 6 nodes, `TD` for vertical hierarchies.
- Keep node labels short (≤ 3 words). Detail goes in caption.
- Use `(...)` for service/process, `[...]` for box, `[(...)]` for database, `((...)) ` for circle, `{...}` for decision, `/.../` for queue/event.
- Always add a `<figcaption>` explaining what the diagram shows.

**Convert architecture description to mermaid:**
> "Client gọi API Gateway. Gateway xác thực JWT, rồi route đến Order Service. Order Service ghi vào Postgres và publish event qua Kafka."

Becomes:
```
flowchart LR
  Client --> Gateway
  Gateway -->|verify JWT| Order[Order Service]
  Order --> DB[(Postgres)]
  Order --> Kafka[/Kafka/]
```

---

## 7. Pros / Cons table

For trade-off discussions ("Option X có ưu/nhược điểm…" / "Trade-offs of X…").

```html
<!-- VI -->
<div class="proscons">
  <div class="proscons-col pros">
    <h4>✓ Ưu điểm</h4>
    <ul>
      <li>Triển khai nhanh, không cần thay schema.</li>
      <li>Backward compatible với client cũ.</li>
    </ul>
  </div>
  <div class="proscons-col cons">
    <h4>✕ Nhược điểm</h4>
    <ul>
      <li>Tăng độ phức tạp ở tầng routing.</li>
      <li>Phải maintain 2 đường code trong 1 quý.</li>
    </ul>
  </div>
</div>

<!-- EN -->
<div class="proscons">
  <div class="proscons-col pros">
    <h4>✓ Pros</h4>
    <ul>
      <li>Fast to ship, no schema change required.</li>
      <li>Backward compatible with existing clients.</li>
    </ul>
  </div>
  <div class="proscons-col cons">
    <h4>✕ Cons</h4>
    <ul>
      <li>Adds complexity to the routing layer.</li>
      <li>Two code paths to maintain for one quarter.</li>
    </ul>
  </div>
</div>
```

- The `<h4>` text follows the language label table.

---

## 8. Comparison cards

For "Option A vs B vs C" — when there are ≥ 2 alternatives to compare.

```html
<div class="compare">
  <div class="compare-card recommended">
    <h4>Option B — Hybrid cache</h4>
    <p>Redis L1 + DB fallback. Balances latency and complexity.</p>
  </div>
  <div class="compare-card">
    <h4>Option A — DB only</h4>
    <p>Simplest but p99 latency hits 800ms at peak.</p>
  </div>
  <div class="compare-card">
    <h4>Option C — In-memory map</h4>
    <p>Fastest but loses state on restart, no horizontal scaling.</p>
  </div>
</div>
```

- Add `class="recommended"` to the preferred option → ★ badge appears automatically. The badge text comes from `--rec-label` set on `<html style="...">` (see the language table above). Falls back to `★ Recommended` if not set.
- Put the recommended option **first** so the reader sees it immediately.
- All body text follows source language — sample above is English; for a Vietnamese / Chinese / Japanese / … source, write descriptions in that language.

---

## 9. Collapsible section

For optional / deep-dive content the reader can skip.

```html
<details class="collapsible">
  <summary>Chi tiết kỹ thuật về sharding strategy</summary>
  <div class="collapsible-body">
    <p>Dùng consistent hashing với 256 virtual nodes per physical shard...</p>
    <pre><code>const shard = hash(userId) % SHARDS;</code></pre>
  </div>
</details>
```

**When to collapse:**
- Long code blocks (> 30 dòng)
- Background context không phải reader nào cũng cần
- Alternative approaches đã bị reject
- Detailed FAQ

---

## 10. Plain Markdown elements

These render correctly out-of-the-box with the CSS in `template.html` — just use HTML equivalents:

| Markdown | HTML to write |
|---|---|
| `## Heading` | `<h2 id="kebab-case-id">Heading</h2>` |
| `### Sub` | `<h3 id="kebab-case-id">Sub</h3>` |
| `**bold**` | `<strong>bold</strong>` |
| `inline code` | `<code>inline code</code>` |
| ```` ```js code``` ```` | `<pre><code>js code</code></pre>` |
| `- item` | `<ul><li>item</li></ul>` |
| `1. item` | `<ol><li>item</li></ol>` (chỉ dùng khi KHÔNG phải timeline) |
| `> quote` | `<blockquote>quote</blockquote>` |
| `--- table ---` | `<table>...</table>` |
| `[text](url)` | `<a href="url">text</a>` |

---

## 11. Component selection cheatsheet

Áp dụng heuristics này khi đọc source `.md`:

| Pattern trong source | Component nên dùng |
|---|---|
| Numbered list of action items (`1. làm X`, `2. làm Y`) | Step / Timeline (§3) |
| Architecture description với A gọi B gọi C | Mermaid flowchart (§6) |
| "Client → Server → DB" trong text | Mermaid sequence/flow (§6) |
| Schema / ERD description | Mermaid erDiagram (§6) |
| "Ưu / Nhược", "Pros / Cons", "Trade-offs" | Pros-Cons (§7) |
| "Option A / B / C", "Approaches" | Comparison cards (§8) |
| Kết luận / TL;DR đoạn quan trọng | Key-point highlight (§5) |
| "Lưu ý", "Note", "FYI", "Background" | Callout info (§4a) |
| "Cẩn thận", "Gotcha", "Risk" | Callout warn (§4b) |
| "KHÔNG được", "Tuyệt đối tránh" | Callout danger (§4c) |
| "Đã làm", "Done", "Completed" | Callout success (§4d) |
| "Quyết định", "Decision", "Chose X over Y" | Callout decision (§4e) |
| "Khuyến nghị", "Best practice", "Tip" | Callout tip (§4f) |
| Long code / appendix / FAQ | Collapsible (§9) |
| Bảng so sánh ngắn (≤ 4 cột) | Markdown table (§10) |

---

## 12. Anti-patterns — đừng làm

- ❌ Đừng dùng emoji làm icon (ℹ️ ⚠️ ⛔ 🎯 …). Dùng SVG `<use href="#i-...">` từ sprite — emoji render khác nhau giữa OS, không đổi màu theo theme, phá tone minimal.
- ❌ Đừng bọc MỌI thứ vào callout/highlight — loãng hiệu ứng nổi bật.
- ❌ Đừng dùng `<ol>` cho plan steps — dùng `.timeline` thay.
- ❌ Đừng làm mermaid quá phức tạp (> 15 nodes) — split thành nhiều diagram nhỏ.
- ❌ Đừng inline-style — mọi style đều đã ở `template.html`.
- ❌ Đừng quên `id` trên heading — TOC sẽ vỡ, anchor link cũng vỡ.
- ❌ Đừng tự thêm `<script>` hoặc tải thêm thư viện ngoài.
- ❌ Đừng dịch nguyên văn từng dòng Markdown — phải PHÂN TÍCH rồi mới chọn component.
- ❌ Đừng dùng `<h1>` trong body content — `.doc-title` đã là H1; dùng H2/H3 cho section.

---

## 13. Icon sprite catalog

`template.html` định nghĩa một SVG sprite với 18 icon Lucide-style. Reference bằng `<use href="#i-NAME"/>`. Icon kế thừa `currentColor` nên màu sẽ tự khớp parent — không cần set `fill` hay `stroke` riêng.

| Icon ID         | Hình                        | Dùng cho                                              |
|---              |---                          |---                                                    |
| `#i-info`       | Vòng tròn có chữ "i"        | Callout info, background notes                        |
| `#i-warn`       | Tam giác cảnh báo có "!"    | Callout warn, risk, gotcha                            |
| `#i-danger`     | Vòng cấm (no entry)         | Callout danger, must-not-do                           |
| `#i-success`    | Vòng tròn có dấu check      | Callout success, completed                            |
| `#i-decision`   | Bullseye / target           | Callout decision, ADR                                 |
| `#i-tip`        | Bóng đèn                    | Callout tip, best practice                            |
| `#i-lock`       | Ổ khoá                      | Security/auth notes (dùng với callout-info)           |
| `#i-printer`    | Máy in                      | Print button (đã trong topbar)                        |
| `#i-moon`       | Mặt trăng                   | Theme toggle, light mode active                       |
| `#i-sun`        | Mặt trời                    | Theme toggle, dark mode active                        |
| `#i-menu`       | Hamburger                   | Mobile TOC trigger (đã trong topbar)                  |
| `#i-x`          | Dấu X                       | Close drawer / dismiss                                |
| `#i-file`       | File document               | doc-meta source file                                  |
| `#i-calendar`   | Lịch                        | doc-meta date                                         |
| `#i-clock`      | Đồng hồ                     | doc-meta read time                                    |
| `#i-copy`       | 2 hình vuông xếp lớp        | Copy-to-clipboard (auto-inject trên `<pre>`)          |
| `#i-check`      | Dấu check ✓                 | "Copied" state, completed                             |
| `#i-link`       | Link (chain)                | Anchor link trên heading (auto-inject)                |

**Standard size**: 18-20px cho callout, 14px cho doc-meta, 16-18px cho topbar button. Set qua CSS `.icon` size hoặc width/height attribute.

**Khi cần icon không có trong sprite**: KHÔNG tự thêm vào template. Dùng icon gần nghĩa nhất từ catalog. Nếu thật cần icon mới, báo user để bổ sung vào template.

---

## 14. Edge-case patterns

### 14a. Images
```html
<figure>
  <img src="path/to/image.png" alt="Mô tả ngắn gọn nội dung ảnh">
  <figcaption>Sơ đồ kiến trúc cluster Kafka 3-broker.</figcaption>
</figure>
```
- `alt` MUST mô tả nội dung (không viết "image" hoặc bỏ trống). Nếu ảnh decorative → `alt=""`.
- Ảnh tự responsive (`max-width: 100%`), border-radius khớp theme.

### 14b. Wide tables (nhiều cột → cần scroll ngang trên mobile)
```html
<div class="table-wrap">
  <table>
    <thead>
      <tr><th>Col 1</th><th>Col 2</th><th>Col 3</th><th>Col 4</th><th>Col 5</th></tr>
    </thead>
    <tbody>
      <tr><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td></tr>
    </tbody>
  </table>
</div>
```
- Wrap trong `.table-wrap` khi bảng có **≥ 4 cột** hoặc cell text dài.
- Bảng đơn giản (≤ 3 cột, text ngắn) có thể dùng `<table>` thường — CSS tự handle overflow-x.

### 14c. Task lists (checkboxes)
```html
<ul>
  <li class="task-list-item"><input type="checkbox" class="task-list-item-checkbox" disabled> Việc chưa làm</li>
  <li class="task-list-item"><input type="checkbox" class="task-list-item-checkbox" disabled checked> Việc đã làm</li>
</ul>
```
- Dùng khi source `.md` có `- [ ]` / `- [x]`. Checkbox `disabled` vì HTML chỉ để đọc.

### 14d. Footnotes
```html
<p>Đoạn text có chú thích<sup class="footnote-ref"><a href="#fn1" id="fnref1">1</a></sup>.</p>

<!-- Cuối document -->
<ol class="footnotes">
  <li id="fn1">Nội dung footnote 1. <a href="#fnref1">↩</a></li>
</ol>
```

### 14e. Very long title (≥ 80 chars)
Title tự `text-wrap: balance` và `clamp(28px, ...)` font-size responsive. Không cần làm gì thêm.

### 14f. Empty TOC (source không có H2)
JS tự ẩn `<aside class="toc">` khi `#toc-nav` không có link. Không cần làm gì thêm.

### 14g. Long URLs / identifiers
`.content` đã có `overflow-wrap: anywhere` — URL/identifier dài tự wrap, không phá layout.
