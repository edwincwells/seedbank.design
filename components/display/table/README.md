# table

A semantic data table for read-only display. Editorial visual treatment — hairline horizontal row separators, no vertical lines, no zebra striping. Optional row-level status via activity-coloured left-rules.

---

## What it styles

- `<table>`, `<caption>`, `<thead>`, `<tbody>`, `<tfoot>`, `<tr>`, `<th>`, `<td>` — styled as a unit via element selectors
- `.tr-grow`, `.tr-repair`, `.tr-share`, `.tr-make` — optional row-level status
- `.td-num` — optional numeric column alignment (applied to both `<th>` and `<td>` in that column)
- `.table-wrap` — optional wrapping `<div>` for horizontal scroll at narrow viewports

That's the whole component. No sortable headers, no filter UI, no row selection, no pagination, no sticky headers, no expand/collapse, no responsive transformation.

---

## When to use it

- Inventories, registers, lists with multiple columns (tools, members, items, events)
- Numeric summaries with totals (energy usage, costs, counts)
- Comparison tables (options, periods, categories)
- Anywhere "rows of data with shared structure" describes the content

When *not* to use it:

- **Don't use a table for page layout.** Use grid or flex. Tables are for tabular data.
- **Don't use a table for a list with one column.** That's a `<ul>` — see `foundations/lists`.
- **Don't use a table for a sortable / filterable data grid.** Build that on top of this component if you need it — don't bend this one into something it isn't.
- **Don't use a table to display a single record.** That's a definition list (`<dl>`) or a card.

---

## Visual treatment — editorial, not data-grid

This is the most consequential choice in the component. The system uses an **editorial** treatment, not a data-grid treatment:

- Hairline horizontal row separators only — no vertical lines, no cell borders
- No zebra striping
- No rounded corners or contained card-style chrome
- One density (no compact variant)
- Uppercase eyebrow column headers, in `--text-subtle`
- `--text` body cells with comfortable padding

Reasons:

- **Consistent with the two-vocabulary structure.** Editorial elements (cards, list items, inputs, dividers, tables) use hairlines and `--radius-none`. Interactive controls feel like software. A table is not interactive.
- **Vertical lines are visual noise.** Columns of aligned text already imply the column structure. Adding lines costs ink/pixels without adding meaning.
- **Zebra striping is contested for accessibility** (interferes with reading at low contrast) and adds complexity around dark mode and the activity-coloured row left-rule.
- **The community deployment contexts** — printed minutes, e-ink, low-bandwidth — all favour the editorial treatment.

Forks that need a data-grid look can override (see below). The system holds the editorial line.

---

## Header treatment

`<th>` cells inside `<thead>` get an **eyebrow** treatment:

- `--text-xs` (11px), uppercase, `--tracking-label` letter-spacing
- `--weight-medium` (500), `--text-subtle` colour
- Left-aligned by default; numeric columns override via `.td-num`

The eyebrow treatment mirrors `h6` from the prose foundation and the small-section-label pattern used throughout the system. Column headers label columns; they don't compete with the data.

`<th>` cells inside `<tbody>` — used for row headers in two-axis tables — get a **different** treatment:

- `--text-sm` (13px), sentence case
- `--weight-medium`, default text colour

The role differs: a column header is an eyebrow naming a category, a row header is the label *of* a row. Different roles, different treatments. Screen readers benefit either way — `<th>` with `scope="row"` is an accessibility win for navigating tabular data.

---

## Row status — activity-coloured left-rule

When a row carries a status, apply one of four classes to the `<tr>`:

```html
<tr class="tr-grow">…</tr>
<tr class="tr-repair">…</tr>
<tr class="tr-share">…</tr>
<tr class="tr-make">…</tr>
```

The first cell of the row gets a 5px left-rule in the matching activity colour. The convention matches `patterns/card-status-row` (when that lands) — a row carrying status reads the same whether it's a row in a table or a card in a list.

**Why classes here, not attributes.** This is the first place in the system where state is expressed via a class instead of an attribute. The attribute-driven convention (`disabled`, `required`, `aria-invalid`) works well for binary states. Activity tokens are a four-way categorical axis (grow / repair / share / make), and the attribute approach (`data-status="repair"`) would produce uglier CSS selectors without clear benefit. The class names map directly to the activity tokens, keeping the system's vocabulary consistent. The same pattern applies to `card-status-row`.

**Implementation note.** Under `border-collapse: collapse`, you can't put a reliable border on a `<tr>` element. The left-rule is applied to the row's first cell via `tr.tr-repair > :first-child`. This works whether the first cell is a `<th>` or a `<td>` and survives column reordering.

**A row without a status class** carries no left-rule. Don't force every row to have a status colour — neutral rows are fine and common.

---

## Numeric columns

Apply `.td-num` to both the `<th>` in the header AND every `<td>` in that column:

```html
<thead>
  <tr>
    <th>Space</th>
    <th class="td-num">kWh used</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Workshop</td>
    <td class="td-num">142.8</td>
  </tr>
</tbody>
```

The class right-aligns the column and applies `font-variant-numeric: tabular-nums` so digits have equal widths — decimal points line up across rows, and 999 sits under 100 cleanly.

**Why not mono font for numbers.** A mono switch (Plex Mono) is the obvious alternative but reads visually loud against the editorial calm of the rest of the table. `tabular-nums` gives the alignment benefit without breaking the visual register.

---

## Caption

Every data table should have a `<caption>`. It's the table's accessible name — the first thing a screen reader reads.

```html
<table>
  <caption>October energy usage by space</caption>
  …
</table>
```

The caption is styled as a small title above the table, left-aligned. To visually hide it while keeping it for assistive tech, apply a `.sr-only` utility (not yet in the system — small addition when first needed; don't reach for `display: none`, which removes it from the accessibility tree entirely).

---

## `tfoot` — totals and summaries

If your table has totals or a summary row, use `<tfoot>`. A stronger top border separates the foot from the body so totals don't read as "just another row".

```html
<tfoot>
  <tr>
    <td>Total</td>
    <td class="td-num">344.2</td>
  </tr>
</tfoot>
```

Omit `<tfoot>` when there's nothing to total.

---

## The 320px reality — wrap your tables

Tables don't reflow. They have a minimum width set by their content. At 320px, a four-column table will exceed the viewport and force horizontal page scroll.

The recommended composition is to wrap the `<table>` in a scrollable div:

```html
<div class="table-wrap" tabindex="0">
  <table>
    …
  </table>
</div>
```

The wrap doesn't change the table's appearance — it just contains the overflow to a scrollable region inside the page. `tabindex="0"` lets keyboard users focus the wrap and use arrow keys to scroll horizontally.

**Wrap any table that could exceed the viewport** — which in practice is most tables. The wrap costs nothing visually; the page-scroll alternative looks broken on narrow viewports.

**What about stacking into cards on mobile?** Some design systems transform tables into stacked cards on narrow viewports. We don't. Reasons:

- It requires per-table column-label hints in markup or JavaScript to remap headers to each cell — significant added complexity
- The stacked layout loses the cross-row scanning that's the point of a table
- Horizontal scroll inside a wrap works at 320px without remapping anything and degrades gracefully

If a fork genuinely needs the stack-into-cards transformation for a specific table, it's an override pattern, not a system default.

---

## Tokens used

| Token | Used for |
|---|---|
| `--text` | Body text, row header text, caption colour |
| `--text-subtle` | Column header (eyebrow) colour |
| `--font-sans` | All table text |
| `--text-xs` | Column header size |
| `--text-sm` | Row header and caption size |
| `--text-base` | Body cell size |
| `--weight-medium` | Headers, captions, totals |
| `--tracking-label` | Column header letter-spacing |
| `--leading-base` | Cell line-height |
| `--border-hairline` | Row separators |
| `--border-thick` | `<tfoot>` top border |
| `--border` | Row separator colour |
| `--border-strong` | Header and footer division colour |
| `--border-rule` | Activity-coloured left-rule width (5px) |
| `--grow`, `--repair`, `--share`, `--make` | Row status left-rule colours |
| `--space-2`, `--space-3` | Cell padding |

No values are hardcoded.

---

## Accessibility

- **Caption is essential.** Every data table needs `<caption>` so screen readers can announce what the table contains.
- **`<th>` for headers, with `scope`.** Use `<th scope="col">` for column headers and `<th scope="row">` for row labels. The component styles both correctly; the scope attribute helps assistive tech navigate two-axis tables.
- **Activity colour is not the only signal.** The left-rule is a visual marker; the meaning of each status is carried by the cell content (typically a badge or text saying "Under repair", "Available", etc.). Don't rely on colour alone to communicate row status.
- **`.table-wrap tabindex="0"`** makes the scrollable region focusable so keyboard users can pan horizontally with arrow keys.
- **Sortable / filterable tables** — if your fork builds those behaviours, ensure the sort controls are real `<button>` elements (not styled `<th>`), with `aria-sort` reflecting the current state.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/display/table/table.css`
3. Write semantic table markup — `<table>` with `<caption>`, `<thead>`, `<tbody>`, optional `<tfoot>`
4. Wrap in `<div class="table-wrap" tabindex="0">` if the table could exceed viewport width
5. Add `.tr-*` classes to rows that carry status; add `.td-num` to numeric columns

---

## How to override

**To switch to a data-grid look** (vertical cell borders, zebra striping, compact density):

```css
table {
  border: var(--border-hairline) solid var(--border);
}
tbody td,
tbody th,
thead th,
tfoot td {
  border-right: var(--border-hairline) solid var(--border);
  padding: var(--space-1) var(--space-2);  /* tighter */
}
tbody tr:nth-child(even) {
  background: var(--surface-raised);
}
```

Be explicit about why your fork needs this. The editorial default is a deliberate choice — see "Visual treatment" above.

**To add a compact density variant:**

```css
.table-compact td,
.table-compact th {
  padding: var(--space-1) var(--space-2);
  font-size: var(--text-sm);
}
```

**To use a mono font for numeric columns** instead of `tabular-nums`:

```css
.td-num {
  font-family: var(--font-mono);
}
```

Visually louder. Only worth it for tables that are predominantly numeric and where the mono register actively helps.

**To stack rows as cards on mobile** — this is significant custom CSS plus markup changes (column labels duplicated via `data-label` attributes on every cell, displayed via `::before`). Not a recommended override for system-wide use; build it for the specific table that needs it.

---

## Print rules

- `display: table-header-group` on `<thead>` makes column headers repeat on every page if the table breaks across multiple pages — a real print-quality win
- `<tfoot>` similarly repeats via `display: table-footer-group`
- `page-break-inside: avoid` on `<tr>` keeps rows intact across page breaks
- Activity-coloured row left-rules become `--text` for monochrome legibility (same approach as badge)
- `.table-wrap` overflow is suppressed — the table flows naturally on paper

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
