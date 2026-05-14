---
name: md2html
description: Biến file Markdown (plan, spec, system design, RFC, runbook) thành HTML trực quan, single-file, có sơ đồ Mermaid, step cards, callouts, TOC. Theme Claude orange light+dark. Portable cross-AI.
trigger: /md2html
---

# /md2html

Convert a verbose Markdown document into a single, self-contained HTML file that a tired human can actually scan: diagrams instead of paragraphs, step cards instead of numbered lists, callouts for the parts that matter.

## Usage

```
/md2html <file.md>             # output <file>.html next to source
/md2html <file.md> --out X.html # custom output path
/md2html                       # if no arg, ask user which file
```

## Skill files (resolved relative to this SKILL.md)

- `template.html`   — HTML skeleton with embedded CSS (Claude orange light+dark), Mermaid CDN, theme toggle, TOC sidebar, footer. Contains `{{PLACEHOLDER}}` strings and `<!-- COMMENT -->` slots.
- `components.md`  — catalog of HTML snippets you must copy verbatim (step cards, callouts, mermaid blocks, pros-cons, comparison cards, collapsibles).
- `examples/`     — at least one reference `<doc>.md` → `<doc>.html` pair. Read one to calibrate output quality before starting.

**You MUST read all three before writing output.** Do not invent CSS classes or skip the catalog.

## What you must do when invoked

Follow these steps in order. Do not skip.

### Step 1 — Resolve inputs

1. Determine the source file from the user's invocation. If none given, ask: *"Tệp `.md` nào cần convert?"* and stop.
2. Read the source `.md` fully.
3. Read `template.html` and `components.md` from the same directory as this SKILL.md.
4. Read one example pair under `examples/` to calibrate.

### Step 2 — Analyze the source document

Do this analysis silently in your head (or as one short summary line to the user). Identify:

- **Language of the source** — detect from the actual prose, not the filename. Set `<html lang="...">` to the ISO 639-1 code (`en`, `vi`, `zh`, `ja`, `ko`, `es`, `fr`, `de`, `ru`, `ar`, `th`, …) and translate every UI label to that language.

  Common samples (extend to any language using the same scheme):

  | Key            | EN                 | VI                 | ZH (中文)           | JA (日本語)         | KO (한국어)         | ES (Español)        |
  |---             |---                 |---                 |---                  |---                  |---                  |---                  |
  | TOC title      | Contents           | Mục lục            | 目录                | 目次                | 목차                | Contenido           |
  | Read-time      | ~N min read        | ~N phút đọc        | ~N 分钟阅读         | ~N 分で読了         | ~N분 소요           | ~N min de lectura   |
  | Recommended    | ★ Recommended      | ★ Đề xuất          | ★ 推荐              | ★ 推奨              | ★ 추천              | ★ Recomendado       |
  | Key point      | Key point          | Ý chính            | 要点                | 要点                | 핵심                | Idea clave          |
  | Pros           | ✓ Pros             | ✓ Ưu điểm          | ✓ 优点              | ✓ 長所              | ✓ 장점              | ✓ Ventajas          |
  | Cons           | ✕ Cons             | ✕ Nhược điểm       | ✕ 缺点              | ✕ 短所              | ✕ 단점              | ✕ Desventajas       |
  | Print tooltip  | Print / Save PDF   | In / Lưu PDF       | 打印 / 保存 PDF     | 印刷 / PDF 保存     | 인쇄 / PDF 저장     | Imprimir / Guardar  |
  | Theme tooltip  | Toggle theme       | Đổi theme          | 切换主题            | テーマ切替          | 테마 전환           | Cambiar tema        |
  | Source: prefix | Source:            | Nguồn:             | 来源:               | ソース:             | 소스:               | Fuente:             |

  For any language not listed, translate using the same conventions. The "Recommended" badge is configured via the `--rec-label` CSS variable set on `<html>` (no per-language CSS needed) — see `{{REC_LABEL}}` below.

  **RTL languages** (Arabic, Hebrew, Persian) — current template is LTR-only. If source is RTL, also add `dir="rtl"` to `<html>` and consider it a known visual limitation (sidebar will stay on the left).

- **Title** — from first H1 or filename. Title should be ≤ 80 chars.
- **Subtitle** — first paragraph after H1, or the document's TL;DR sentence. ≤ 200 chars.
- **Doc type** — infer one of: `PLAN`, `SPEC`, `SYSTEM DESIGN`, `RFC`, `RUNBOOK`, `POSTMORTEM`, `NOTES`. The uppercase code in the eyebrow stays universal; the topbar `BRAND_LABEL` localizes (Plan / Kế hoạch).
- **Reading time** — words ÷ 250, round to nearest minute. Format: `~N min read` (EN) or `~N phút đọc` (VI).
- **Section map** — walk each H2/H3 and tag with the BEST component using §11 cheatsheet in `components.md`:
  - numbered action list → Timeline
  - architecture/flow prose → Mermaid
  - "ưu/nhược", "pros/cons" → Pros-Cons
  - "option A vs B" → Comparison cards
  - critical conclusion → Key-point highlight
  - warnings/decisions → Callouts
  - long appendix → Collapsible
  - everything else → plain `<h2>` + `<p>`

### Step 3 — Build the output HTML

1. **Copy** the full `template.html` content into a string. Do NOT use Read-then-Edit on a file you haven't created; instead, build the output buffer in memory then `Write` once.
2. **Replace placeholders** in the template (all values come from Step 2 analysis, language-matched):
   - `{{LANG}}` → ISO 639-1 code: `en` / `vi` / `zh` / `ja` / `ko` / `es` / …
   - `{{REC_LABEL}}` → text shown on the "Recommended" comparison-card badge, e.g. `★ Recommended` / `★ Đề xuất` / `★ 推荐` / `★ 추천`. Sets the `--rec-label` CSS variable on `<html>`. If you forget this, CSS falls back to `★ Recommended`.
   - `{{TITLE}}` (appears twice: `<title>` and `.doc-title`)
   - `{{SUBTITLE}}`
   - `{{DOC_TYPE}}` → universal uppercase code (`PLAN`, `SPEC`, `RFC`, …)
   - `{{SOURCE_FILE}}` → basename of source (e.g. `plan.md`)
   - `{{DATE}}` → ISO date or localized "Updated <today>"
   - `{{READ_TIME}}` → localized reading time, e.g. `~3 min read` / `~3 phút đọc` / `~3 分钟阅读`
   - `{{BRAND_LABEL}}` → localized doc-type label for the topbar
   - `{{TOC_TITLE}}` → localized "Contents" (also used as `aria-label` for the TOC drawer)
   - `{{PRINT_TOOLTIP}}` → localized print tooltip
   - `{{THEME_TOOLTIP}}` → localized theme-toggle tooltip
   - `{{CLOSE_LABEL}}` → localized "Close" (used for the mobile TOC drawer close button), e.g. `Close` / `Đóng` / `关闭` / `閉じる`
   - `{{SKIP_LINK_LABEL}}` → localized skip-to-content link text, e.g. `Skip to content` / `Bỏ qua menu` / `跳到正文`
   - `{{FOOTER_NOTE}}` → localized source attribution (e.g. `Source: plan.md` / `Nguồn: plan.md` / `来源: plan.md`)
3. **Replace `<!-- TOC_ENTRIES -->`** with one `<a>` per H2/H3 (see §2 in components.md). Generate stable kebab-case `id` from heading text.
4. **Replace the slot between `<!-- CONTENT_START -->` and `<!-- CONTENT_END -->`** with the document body, section by section, using components from `components.md`. Each section must:
   - Start with `<h2 id="..."> ` (matching the TOC entry).
   - Use ONE primary component per logical chunk (don't stack 3 callouts in a row).
   - Preserve original meaning — do not summarize away technical detail; condense only filler/repetition.
5. **Write** the assembled HTML to the output path.

### Step 4 — Verify

After writing, do ONE quick sanity check by re-reading just the section you generated (not the whole file):
- Every `id="..."` referenced in the TOC exists on a heading.
- No leftover `{{PLACEHOLDER}}` strings.
- Mermaid blocks have valid syntax (use `flowchart`, `sequenceDiagram`, `erDiagram`, `stateDiagram-v2`, or `gantt` — never bare `graph` without direction).
- No `<script>` tags added beyond what `template.html` already includes.

Report back to the user with:
- Output file path
- 1-line summary of what changed (e.g. *"Rendered 7 sections: 1 mermaid flow, 2 step timelines, 4 callouts. ~6 phút đọc."*)
- A reminder they can open it with `xdg-open <file>.html` (Linux) / `open <file>.html` (mac).

## Critical rules

1. **Never paraphrase technical content into vague prose.** A step `chạy migration 0042_user_schema.sql` must remain that exact filename — don't change to `chạy migration mới`.
2. **One component per chunk.** Don't wrap a callout inside a step card inside a collapsible. Keep nesting flat.
3. **Mermaid > prose for any flow ≥ 3 hops.** If the source says "A gọi B, B gọi C, C ghi DB", make a diagram.
4. **Key-point highlights are rare.** Max 1 per H2 section, ideally 2-3 total per document.
5. **UI text follows the detected source language** — including for non-EN/non-VI sources (Chinese, Japanese, Korean, Spanish, etc). Use the language sample table in Step 2 or translate equivalently. Code, commands, file names, library names, error messages stay verbatim regardless of language.
6. **Self-contained output.** No external file references except the CDN scripts already in `template.html`.
7. **Do not modify `template.html` or `components.md`** — those are the skill's source of truth. Only Write the output `.html`.
8. **Use SVG icons only — never emojis.** Every icon is `<svg class="..."><use href="#i-NAME"/></svg>` referencing the sprite at the top of `<body>`. See §13 in `components.md` for the catalog. No emoji glyphs anywhere in callouts, doc-meta, topbar, or body content.
9. **Anchor links and copy-to-clipboard auto-inject via JS** — do NOT add them manually. Just give H2/H3 a proper `id`, and put code in `<pre><code>`. The template's boot script handles the rest.
10. **Wrap wide tables in `.table-wrap`** — see components.md §14b. Tables ≥ 4 columns or with long cells need the wrapper for mobile scroll.
11. **Use `<figure>` + `<figcaption>` for images** with descriptive `alt`. See components.md §14a.

## Cross-AI compatibility

This skill is designed to run identically on:

- **Claude Code** — install at `~/.claude/skills/md2html/` (this directory, symlinked or copied). Invoke with `/md2html <file>`.
- **Codex CLI** — copy `SKILL.md` content to `~/.codex/prompts/md2html.md`, keep `template.html` and `components.md` at a stable absolute path, update the file references in SKILL.md if needed. Invoke with `/md2html`.
- **Antigravity** — add SKILL.md as a custom prompt/agent instruction, ensure the agent has Read/Write tool access to the skill folder.

The only external dependency is `mermaid` via CDN (resolved at HTML open time, not at skill execution time). No npm/pip install required for the skill itself.

## Edge cases

- **Source has no headings** — wrap content in one `<h2 id="content">Nội dung</h2>` and infer logical breaks from blank lines + topic shifts.
- **Source has existing mermaid code blocks** — keep them, just rewrap in `<figure class="diagram">` with caption.
- **Source has HTML embedded** — pass through as-is inside `<div>` if safe, else escape.
- **Source is very short (< 200 words)** — skip TOC sidebar (delete `<aside class="toc">` block), render as single column.
- **Source is very long (> 5000 words)** — collapse low-priority sections by default with `<details>`.
- **Output file already exists** — overwrite. The source `.md` is canonical; HTML is regenerated artifact.

## Anti-patterns

- ❌ Generating the HTML in many small Edit calls — produces drift. Build full string, Write once.
- ❌ Adding new CSS via `<style>` — extend `template.html` instead and tell the user.
- ❌ Translating proper nouns or code identifiers.
- ❌ "Improving" the source by adding info not in the original.
- ❌ Reporting success without running Step 4 verification.
