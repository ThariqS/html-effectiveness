---
name: as-html-docs
description: Build self-contained HTML docs (plans, reports, write-ups) from a curated block library. Use when user explicitly asks for "as HTML" — e.g. "make this plan an HTML page", "write the post-mortem as HTML". SKIP for Angular/component templates, app UI, .component.html, README/markdown, or auto-converting existing markdown.
user-invocable: true
---

# as-html-docs

Produce a **single self-contained HTML file** for a human to read — visual-first, skimmable, no build step. Adapted from the html-effectiveness templates (Apache-2.0; see `ATTRIBUTION.md`).

## When to use / not use

**Use** only on an explicit request to author a _document_ as HTML: implementation plans, status/incident reports, research/learning write-ups, code-review write-ups, design/value galleries — things a person opens and reads.

**Do NOT use** for:

- Angular/app HTML, `*.component.html`, any UI that ships in the product.
- Plain markdown docs, READMEs, or Jira/Confluence content.
- Silently converting an existing markdown plan — only when HTML was asked for.

If the request is ambiguous about HTML vs markdown, ask before generating.

## Hard constraints

1. **One file.** Everything inline — CSS in `<style>`, any JS in `<script>`, any image as inline SVG or data URI. No asset directory, no CDN/framework link, no external font.
2. **Opens by double-click; prints clean.** No server needed. The shell already has a print stylesheet.
3. **Visual-first, but text stays.** Use blocks (timelines, cards, diagrams, tables, collapsibles) to carry structure; keep the prose a reader needs. Don't drop content to look pretty.
4. **Internal-only.** These docs are never user-facing product output, so plain inline styling here is fine — this is the one place the repo's "no inline styles / AUI-only" frontend rules do **not** apply.
5. **Don't hand-edit generated artifacts** elsewhere; this skill only writes the HTML doc you were asked for.

## Workflow

1. **Confirm intent + destination.** Default path follows the repo plan convention: `docs/plans/<version>/<topic>/<name>.html` (in the current worktree). Confirm or take the user's path.
2. **Read `base-shell.html`** from the skill directory (`/Users/johan/.claude/skills/as-html-docs/base-shell.html`), then copy it to the output path. Set `<title>`, the `.doc-head` (eyebrow, h1, lede, meta pills/ticket/date/author), and the `--accent` token if a different accent is wanted.
3. **Read the block files** you need from `blocks/` in the skill directory, then paste each fragment into `<main class="page">` in reading order. All block CSS already lives in the shell, so fragments render as-is. Do **not** recreate blocks from memory — always read the source file.
4. **Add a table of contents.** After the `<header class="doc-head">` and before the first `<section>`, insert a `<nav class="toc">` block. Give every `<section>` an `id` attribute (kebab-case of the heading text) and link to each from the TOC. Use the `toc` fragment from `blocks/toc.html` as the template.
5. **Fill with real content.** Replace every placeholder. Escape `< > &` inside `<pre>`/code as `&lt; &gt; &amp;`.
6. **Prune.** Delete unused blocks. Optionally remove their now-unused CSS rules to keep the file lean.
7. **Verify** (see below), then report the path.

## Situation → block index

| You are showing…                 | Block file                   | Notes                                        |
| -------------------------------- | ---------------------------- | -------------------------------------------- |
| Key numbers/status at a glance   | `blocks/summary-cards.html`  | 2–6 cells; put near top                      |
| Ordered phases / roadmap         | `blocks/timeline.html`       | `dot.done` marks completed phases            |
| What happened at specific times  | `blocks/event-timeline.html` | incidents, debugging, deploys                |
| One important note inline        | `blocks/callout.html`        | note / warn / success / decision             |
| Two options or before/after code | `blocks/code-grid.html`      | escape code; trade-offs under each           |
| Comparable rows (risks, options) | `blocks/table.html`          | severity chips low/med/high                  |
| Scannable, expandable detail     | `blocks/collapsible.html`    | native `<details>`, no JS                    |
| A flow / architecture picture    | `blocks/diagram.html`        | inline SVG; palette hex (no CSS vars in SVG) |
| A set of discrete values         | `blocks/swatches.html`       | colors, states, enum variants                |
| What the doc was built from      | `blocks/sources.html`        | provenance footer, put last                  |
| Anchor-linked section index      | `blocks/toc.html`            | always add after doc-head; auto-links to `id` |

Compose freely — a typical implementation-plan doc is: header → **toc** → summary-cards → timeline → code-grid → table (risks) → sources.

## Verify before claiming done

- File is self-contained — run both checks and confirm neither returns output:
  - HTML tags: `grep -iE '<(script|link|img|iframe)[^>]+(src|href)=.*(https?:|cdn)' <file>`
  - CSS imports/URLs: `grep -iE '@import|url\s*\(\s*["'"'"']?https?:' <file>`
  - No `../` asset paths.
- Open it (or screenshot via the browser tools) and confirm it renders, the chosen blocks are populated with real content, and no placeholder text remains.
- It is a _document_, not an app: no build, no install.

## Evolving this skill

The block set is intentionally small. To add a block: add `blocks/<name>.html` (lead with a `<!-- BLOCK: -->` comment explaining when to use it), add its CSS to `base-shell.html`'s `<style>`, and add a row to the index above. Keep the shared token palette so every block stays visually consistent.
