# dividers

Thematic section breaks. The `<hr>` element styled to fit the system's editorial vocabulary — hairline weight, neutral colour, asymmetric vertical rhythm.

This is a **foundation** — it applies globally via the element selector. Include `dividers.css` once in your fork and every `<hr>` in your HTML receives the system styling. No classes needed.

---

## What it styles

- The `<hr>` element, and only the `<hr>` element

There are no variants. One divider does the job. If your fork finds real use cases that need a heavier or lighter divider, add them — but consider whether the underlying need is actually a different element (a heading, a card border, a list separator) before introducing a new divider class.

---

## The defaults

| Property | Value | Token |
|---|---|---|
| Border | 0.5px solid neutral | `--border-hairline`, `--border` |
| Width | Full container width | — |
| Top margin | 48px | `--space-7` |
| Bottom margin | 24px | `--space-5` |

**On the asymmetric margin.** Most divider implementations use symmetric vertical margins, treating the divider as a midpoint. The system uses asymmetric margins (more above, less below) because a divider's job is to mark the end of one section and the start of another — and the start of a new section usually wants to feel like a beginning, with less weight above it than the previous section's content had.

If your fork prefers symmetric spacing, override locally:

```css
hr { margin: var(--space-6) 0; }
```

**On the hairline weight.** The 0.5px hairline matches the system's existing dividers — list item separators, card edges, table row borders. Using the same weight everywhere creates a consistent quietness across the editorial vocabulary. A heavier divider would compete with headings for hierarchy, which is usually a sign the heading should be doing the work instead.

---

## When to use `<hr>`

Use a divider when you have a genuine thematic break between sections of content, and a heading alone doesn't make the transition clear enough.

Typical uses:

- Between sections of long-form prose where the topic shifts
- Between distinct content blocks on a page (e.g. above the footer, between unrelated cards)
- In documentation, between a definition and its examples, or between sections of a reference page

---

## When NOT to use `<hr>`

**Don't use it for decoration.** A purely decorative line should be a `<div>` with `aria-hidden="true"` and a `border-top`. Screen readers announce every `<hr>` as a separator, which is noise if the line carries no semantic meaning.

**Don't use it between adjacent list items.** That's a list concern — `foundations/lists` handles it via `border-bottom` on `<li>` elements.

**Don't use it where whitespace would do.** A heading's own top margin often creates enough separation. A divider plus a heading is frequently one element too many. Let the heading do the work; reach for `<hr>` only when the break needs to be louder than a heading provides.

**Don't use it inside cards as an internal separator.** That's almost always better handled by `border-bottom` on the rows or sections inside the card. Reserve `<hr>` for breaks between blocks, not within them.

---

## Accessibility

`<hr>` has an implicit `role="separator"`. Screen readers announce it as a separator or pause at it. This is the correct behaviour when the divider carries semantic meaning — a real thematic break.

If you find yourself wanting to suppress that announcement, the divider probably shouldn't be an `<hr>` in the first place. Reach for a styled `<div aria-hidden="true">` instead.

No `aria-orientation` is needed — `<hr>` defaults to horizontal, which is what the system uses.

---

## Tokens used

| Token | Used for |
|---|---|
| `--border-hairline` | Border weight (0.5px) |
| `--border` | Border colour |
| `--space-7` | Top margin (48px) |
| `--space-5` | Bottom margin (24px) |

No values are hardcoded. Forks remap by changing tokens, not by editing this file.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/dividers/dividers.css`
3. Write semantic HTML — `<hr>`. The styling applies automatically.

---

## How to override

**To change the spacing:** override `hr { margin: ... }` directly in your fork. The foundation's defaults are deliberate (see above), but the override is one line.

**To change the weight:** if you want a thicker divider across the whole system, remap `--border-hairline` in `tokens.css` — but be aware this propagates to every hairline in the system (list separators, card edges, table rows). If you only want `<hr>` thicker, override locally:

```css
hr { border-top-width: var(--border-thick); }
```

**To add a heavier or lighter variant:** add a class in your fork's stylesheet:

```css
hr.divider-strong { border-top-color: var(--border-strong); }
hr.divider-subtle { border-top-color: var(--border-subtle); }
```

The foundation deliberately ships no variants. Each variant a fork adds is a decision its own users inherit — better to add them only when real use cases surface.

**To remove the asymmetric margin pattern:** see the symmetric override in the defaults section above.

---

## Print rules

The foundation includes a `@media print` block:

- Dividers get `page-break-before: avoid` so an `<hr>` doesn't print orphaned at the top of a new page, separated from the section it was ending.

The hairline remains visible in print — browsers handle the colour rendering, and a fine grey line is the intended appearance.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
