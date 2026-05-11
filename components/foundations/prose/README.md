# prose

The typographic baseline for every text element in the system. Settled once here so every component containing text inherits consistent styling.

This is a **foundation** — it applies globally via element selectors. Include `prose.css` once in your fork and every heading, paragraph, and inline element receives the system styling. No classes needed.

---

## What it styles

- All six heading levels (`h1`–`h6`)
- Paragraphs (`p`)
- Inline emphasis (`strong`, `em`, `small`)
- Long-form prose containers (`article`, `.prose-long`)
- Print rendering rules for all of the above

It does **not** style:
- Links — see `foundations/links` (when built)
- Lists — see `foundations/lists` (when built)
- Code — see `display/code` (when built)
- Tables — see `display/table` (when built)

---

## The heading scale

| Element | Size | Weight | Usage |
|---------|------|--------|-------|
| `h1` | 32px (`--text-xxl`) | 500 | Page title. Rarely used — most pages start at h2. |
| `h2` | 22px (`--text-xl`) | 500 | Major section. The most common page-level heading. |
| `h3` | 18px (`--text-lg`) | 500 | Sub-section within a major section. |
| `h4` | 15px (`--text-md`) | 500 | Component title within a sub-section. |
| `h5` | 14px (`--text-base`) | 500 | Rare. If you reach h5, consider whether the document needs restructuring. |
| `h6` | 11px (`--text-xs`) | 500 | Uppercase eyebrow label. See note below. |

**On `h6` as the uppercase eyebrow.** The system uses small uppercase section labels heavily — they are semantically headings even though they don't look like conventional ones. Giving `h6` the eyebrow treatment provides a clean semantic answer for that pattern instead of requiring a non-heading element styled separately.

If your fork doesn't use the eyebrow pattern, you can remap `h6` to be a smaller `h5` by editing the `h6` block in `prose.css`. Document the change.

---

## Paragraphs

- 14px regular weight
- Default text colour (`--text`)
- 1.55 line height for UI prose; 1.7 for long-form (see below)
- Bottom margin only — no top margin, so adjacent paragraphs space naturally
- `max-width: 65ch` by default — this is a **prescriptive readability constraint**, not a layout suggestion

**On the 65ch max-width.** Lines longer than ~75 characters hurt reading speed and comprehension. The foundation enforces this by default so prose remains readable regardless of container width.

If a fork needs full-width paragraphs (inside a wide card, a table cell, a notification banner), override locally:

```css
.my-component p { max-width: none; }
```

This is the one place in the foundation that imposes a constraint forks may need to override regularly. Documented so the override pattern is clear.

---

## Long-form prose

Paragraphs inside `<article>` get looser line height (1.7) for sustained reading. This is the foundation's preferred opt-in — it's semantic and free.

```html
<article>
  <p>Long-form reading. Loose leading applied automatically.</p>
</article>
```

A `.prose-long` class is also available for cases where `<article>` is not semantically appropriate (e.g. a long-form region inside a larger document that's not itself an article).

---

## Inline emphasis

| Element | Effect |
|---------|--------|
| `strong` | Medium weight (500). **Not bold** — the system has no bold weight. |
| `em` | Italic. Inherits weight and colour. |
| `small` | One step smaller (13px). Inherits weight and colour. |

All three inherit colour and weight from context. `<strong>` inside an `h6` stays the subtle colour; `<em>` inside a button stays the button's colour.

---

## Tokens used

| Token | Used for |
|-------|----------|
| `--text` | Default heading and paragraph colour |
| `--text-subtle` | h6 eyebrow colour |
| `--font-sans` | All prose elements |
| `--text-xs` through `--text-xxl` | Heading and paragraph sizes |
| `--weight-regular`, `--weight-medium` | Body and heading weights |
| `--leading-tight`, `--leading-snug`, `--leading-base`, `--leading-loose` | Per-level line heights |
| `--tracking-label` | h6 letter-spacing |
| `--space-2` through `--space-7` | Heading and paragraph margins |

No values are hardcoded. Forks remap by changing tokens, not by editing this file.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/prose/prose.css`
3. Write semantic HTML — `<h2>`, `<p>`, `<strong>`. The styling applies automatically.

---

## How to override

**To change a heading size:** remap the relevant `--text-*` token in `tokens.css`. This propagates everywhere the token is used, which may include components beyond prose. If you only want to change the prose use, edit the value in `prose.css` directly — but consider whether the change should be system-wide.

**To remove the `h6` eyebrow treatment:** edit the `h6` block in `prose.css`. Remove `text-transform`, `letter-spacing`, and the colour override. The size and weight should remain.

**To remove the `max-width: 65ch` on paragraphs:** either edit `prose.css` directly, or override per-component as shown above. Removing globally is not recommended — long lines are a real readability problem and the constraint exists for a reason.

**To use a different font family:** remap `--font-sans` in `tokens.css`. Do not change the family in `prose.css` directly — other components also use `--font-sans`.

---

## Print rules

The foundation includes a `@media print` block with these rules:

- Headings get `page-break-after: avoid` so they don't print at the bottom of a page separated from their content
- Paragraphs get `orphans: 3; widows: 3;` so single lines don't get stranded at the top or bottom of a page

Link URL expansion in print is deferred to `foundations/links` when that exists.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
