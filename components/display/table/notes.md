# table — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 24 May 2026.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` via Chrome dev tools. With no CSS applied:

- Every `<table>` fell back to UA table styling — visibly tabular, rows aligned by column
- Every `<caption>` rendered above its table per UA default
- `<thead>`, `<tbody>`, and `<tfoot>` retained their structural roles — column headers visible, totals row present
- Row headers (`<th>` in `<tbody>`) rendered as bold text by UA default — distinguishable from `<td>`
- Numeric columns lost their right-alignment (`.td-num` is a CSS effect) but the numbers remained readable and present in the cells
- Activity-coloured row status (`.tr-grow`, `.tr-repair`, etc.) lost the left-rule but rows remained in document order with content intact
- The wrapped table (`<div class="table-wrap">`) lost its overflow scroll but the table content was still present — table extended beyond viewport, page forced horizontal scroll. Acceptable graceful degradation
- Status badges in cells retained their text content

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome:

- Column headers remained readable and proportional at the eyebrow size
- Row headers and body text remained clearly differentiated
- Numeric column right-alignment held; decimal points still lined up via tabular-nums
- Activity-coloured left-rules remained visible at their effective doubled width — still a marker, not a slab
- Cell padding scaled proportionally; cells did not appear cramped or stretched
- The wrapped table's horizontal scroll engaged correctly at the now-larger effective viewport
- `<tfoot>` separator (the thicker top border) remained visibly heavier than body row separators

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width (Chrome dev tools responsive mode):

- Wrapped tables (`<div class="table-wrap">`) contained their horizontal overflow — the table scrolled inside the wrap, not the page
- Unwrapped tables (basic, row-status, numeric) pushed the page horizontally as expected — confirms the wrap is the documented fix, not a nice-to-have
- The wide solar-generation table (7 columns) scrolled within its wrap as designed
- Captions remained readable and did not overflow
- Cell content did not break in unexpected ways — long text wrapped, numbers stayed on a single line via tabular-nums
- Row left-rules remained visible at the leftmost column even when the table was scrolled

---

## Edge cases checked

**Mixed content in cells (text + badge + icon).** Cell vertical alignment held via `vertical-align: top`. Cells with badges aligned cleanly with neighbouring text-only cells.

**Empty cells / em-dash placeholder.** The "Heat press" row's em-dash in the "Last service" column rendered correctly. Row height stayed even with neighbouring rows.

**Long text in a narrow column.** Cell content wrapped to multiple lines; `vertical-align: top` kept the wrapped text aligned with its neighbours.

**Caption length.** Caption wrapped cleanly above the table without layout disruption.

**`<tfoot>` rendering.** The thicker top border read as structural separation, not visual noise. Total row visibly distinct from body rows.

**Activity-coloured left-rule with first cell being a `<th>` vs `<td>`.** The `:first-child` selector applied the rule identically regardless of whether the first cell was a row header or data cell.

**Dark mode.** Verified via system preference. Hairline row separators remained visible on `--surface-dark`. Activity-coloured left-rules used their dark-mode token values cleanly.

---

## Open items for future iterations

- **Focus ring on `.table-wrap`.** Investigated during testing — the `tabindex="0"` is correctly applied and the focus-ring foundation's selector covers `[tabindex]:not([tabindex="-1"]):focus-visible`. On macOS Chrome by default, tab navigation skips non-form-element focusable elements unless "Press Tab to highlight each item on a webpage" is enabled (Chrome → Settings → Accessibility). Even with that enabled, the seedbank focus ring was not visibly rendering on the wrap — Chrome's own focus highlight showed instead. Diagnosis incomplete; deferred to a future investigation. The wrap is focusable in principle (confirmed via `document.querySelector('.table-wrap').focus()` succeeding and `document.activeElement` returning the wrap), so keyboard accessibility is not blocked — only the visual indicator is uncertain.
- **Print preview check**, as flagged in prior components — still optional, still worth promoting to the standing checklist if the habit holds.
- **Badge inside a button** edge case from the badge component — `display/table` doesn't compose with this either; flagged for whenever it surfaces in a real pattern.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
