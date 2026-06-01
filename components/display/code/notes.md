# code — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 24 May 2026.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` via Chrome dev tools. With no CSS applied:

- Every `<code>` fell back to UA default — monospace font, no background, inline in surrounding prose. Readable and semantically distinct from surrounding text
- Every `<pre>` fell back to UA default — monospace block with preserved whitespace and line breaks. Code formatting (indentation, line breaks) remained intact
- The 0.9em size shift was lost (CSS effect) but code remained distinguishable from prose via the mono font
- Long inline code paths rendered as monospace and wrapped per UA default
- Code blocks without `overflow-x: auto` extended beyond viewport — acceptable graceful degradation, block content still present
- The `<pre><code>` nesting preserved both elements semantically

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome:

- Inline `<code>` remained readable and proportionally sized against surrounding prose
- The tinted background scaled correctly without visible gaps between tint and text
- Code blocks remained readable at the larger effective size; line-length still reasonable
- Code block horizontal scroll engaged correctly when content exceeded the container at the larger effective viewport
- Code in headings (h2, h3) remained visually balanced — neither oversized nor undersized relative to surrounding heading text

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width (Chrome dev tools responsive mode):

- Inline `<code>` wrapped cleanly within prose — long paths broke onto multiple lines without overflowing
- Code blocks contained horizontal overflow via `overflow-x: auto` — block scrolled inside its container, not the page
- The shell command example (longer single line) demonstrated the scroll behaviour at this width
- Code block padding remained comfortable; content was not compressed to unreadable
- Inline code in headings did not break heading layout

---

## Edge cases checked

**`<code>` inside headings.** The 0.9em shrink read correctly at large heading sizes — code in `<h2>` was visibly proportional to the heading text, neither too small to read nor too large to balance. Same held at h3.

**`<code>` inside `<small>`.** Nested em-relative sizing remained proportional — code inside small text was smaller than surrounding small text by the expected 0.9 ratio.

**Entity-escaped content.** The `&lt;code&gt;` examples rendered as the visible characters `<code>` (and similar for `<label>`, `<pre>`) — content was not re-interpreted as HTML.

**Long inline code wrapping.** The file system path example broke at sensible boundaries via `word-break: break-word` — not mid-word ugliness, not viewport overflow.

**Mixed indentation in code blocks.** `white-space: pre` preserved the formatting authors put in. No visible normalisation of tabs or spaces.

**Adjacent code blocks.** Bottom margin (`--space-4`) kept stacked code blocks visually separated. Inline prose between blocks rendered with its usual paragraph spacing on top.

**Dark mode.** `--surface-sunken` tint read correctly on `--surface-dark`. Code remained legible against the tint in both modes.

---

## Open items for future iterations

- **Focus ring on `<pre tabindex="0">`.** Same caveat as `table/notes.md` — the element is focusable in principle (programmatic `.focus()` succeeds) but the seedbank focus ring rendering on non-form-element focusable elements is currently an open investigation across the system. Worth tackling as a single system-wide diagnosis rather than per-component, since the same diagnosis will apply to every `tabindex="0"` element we introduce.
- **Print preview check**, as flagged in prior components — still optional, still worth promoting to the standing checklist if the habit holds.
- **Cascade effect on prior reference pages.** Once `code.css` is included in the repo, every existing reference page using inline `<code>` will pick up the new tinted treatment. Worth a glance at button, badge, table, prose pages after merging to confirm the cascade reads correctly — the visual restraint was chosen specifically to be safe globally, but worth confirming.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
