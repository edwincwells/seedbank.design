# lists

The typographic baseline for content lists — unordered, ordered, and definition. Applies globally via element selectors. Include `lists.css` once in your fork and every `<ul>`, `<ol>`, and `<dl>` in your HTML receives the system styling.

This is a **foundation**, not a pattern. It styles content lists only — lists of words, lists of items, lists of definitions. UI list patterns (status rows, card lists, navigation, anything with borders or left-rules) belong in the patterns layer and will reset this foundation's styling on the elements they use.

---

## What it styles

- `<ul>` — unordered lists
- `<ol>` — ordered lists
- `<dl>` — definition lists (with `<dt>` and `<dd>`)
- Nesting of `<ul>` and `<ol>` to any depth

It does **not** style:
- List-like UI patterns — those live in `patterns/`
- Navigation lists styled as horizontal bars — that's a pattern concern
- Markdown task lists (`<input type="checkbox">` inside `<li>`) — covered when controls land

---

## The defaults

| Element | Marker | Indent | Item gap |
|---|---|---|---|
| `<ul>` | `disc` | 24px (`--space-5`) | 4px (`--space-1`) |
| `<ol>` | `decimal` in `--font-mono`, muted | 24px (`--space-5`) | 4px (`--space-1`) |
| `<dl>` | n/a — stacked layout | `<dd>` indents 16px (`--space-4`) | 12px between pairs (`--space-3`) |

All three list types have `max-width: 65ch` for the same reason paragraphs do — line length affects readability.

---

## Marker choices

**`<ul>` uses disc markers.** The universally recognised "this is a list" signal. Custom dash or em-dash markers via `::marker` were considered and rejected:

- Custom markers have inconsistent rendering across browsers, especially older ones — this conflicts with the edges-first principle.
- The em-dash convention is reserved for activity status badges (`— Available`, `— Under repair`). Reusing dashes for list markers would dilute that signal.
- Discs survive being printed and rendered without CSS.

**`<ol>` uses decimal markers rendered in `--font-mono`.** Numbers in lists are identifiers, and the system uses mono for tokenised metadata (timestamps, codes, counts). Applying mono to list markers extends that convention to where it most clearly fits. The marker is also dropped to `--text-sm` because mono digits read optically larger than proportional digits at the same size, and tinted to `--text-muted` so the numbers anchor without competing with the content.

**Nesting does not change marker style.** Every nested `<ul>` level is still `disc`; every nested `<ol>` level is still `decimal`. The browser-default cascade (disc → circle → square; decimal → lower-alpha → lower-roman) was considered and rejected:

- UA defaults vary between browsers, and forks have less control over the cascade.
- Consistent markers are easier to reason about under fork-and-adapt conditions.
- Deeply nested lists are usually a sign the content should be restructured, not a problem markers can solve.

Forks that prefer the UA-style cascade can override locally — see "How to override" below.

---

## Indentation

Lists indent by 24px (`--space-5`) using `padding-inline-start`. The logical property is deliberate — it flips correctly under RTL languages without requiring overrides. List markers hang in this gutter rather than pushing the content further right.

Nested lists step in by another 24px at each level. Nested lists also have their top and bottom margins reset so they sit immediately under their parent `<li>` without creating a double-gap.

---

## Item spacing

Top-level list items have a 4px bottom margin — enough to distinguish them at a glance, not enough to break the rhythm of dense lists.

Inside `<article>` or `.prose-long`, list items get an 8px bottom margin and the list itself gets `--leading-loose` line-height (1.7). This matches the pattern established by the prose foundation — long-form reading gets more breathing room than UI prose.

---

## Definition lists

`<dl>` renders stacked: `<dt>` on its own line, `<dd>` indented below.

- `<dt>` is `--weight-medium` (500) — the term anchors the pair as a label
- `<dd>` indents 16px from the start, with margin reset from the user-agent default
- Adjacent pairs get 12px of separation via a top margin on `<dt>` following a `<dd>`

The stacked layout works at any width. Grid or inline `<dl>` layouts (term on left, definition on right) were considered and deferred — they introduce layout complexity that conflicts with the foundation's "applies globally without configuration" remit. A fork that wants a grid `<dl>` for a glossary page can override per-context.

---

## Tokens used

| Token | Used for |
|---|---|
| `--text`, `--text-muted` | Body text colour; `<ol>` marker colour |
| `--font-sans`, `--font-mono` | List body; `<ol>` marker family |
| `--text-base`, `--text-sm` | List body size; `<ol>` marker size |
| `--weight-regular`, `--weight-medium` | List body weight; `<dt>` weight |
| `--leading-base`, `--leading-loose` | UI lists; long-form lists |
| `--space-1` to `--space-5` | Item gaps, indentation, pair separation, list margins |

No values are hardcoded.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/lists/lists.css`
3. Write semantic HTML — `<ul>`, `<ol>`, `<dl>`. Styling applies automatically.

---

## How to override

**To use UA-style marker cascading on nested lists:**

```css
ul ul { list-style-type: circle; }
ul ul ul { list-style-type: square; }
ol ol { list-style-type: lower-alpha; }
ol ol ol { list-style-type: lower-roman; }
```

**To remove the mono numerals on `<ol>`:**

```css
ol > li::marker {
  font-family: inherit;
  font-size: inherit;
  color: inherit;
}
```

**To use dash markers on `<ul>`:** if your fork's brand language calls for editorial dashes instead of discs, set them via `::marker`. Note: this duplicates the em-dash convention used for activity status badges, so consider whether the visual signal stays clear.

```css
ul {
  list-style: none;
  padding-inline-start: var(--space-5);
}
ul > li {
  position: relative;
}
ul > li::before {
  content: "–";
  position: absolute;
  left: calc(var(--space-5) * -1);
  color: var(--text-muted);
}
```

**To use a grid layout for `<dl>`:** override per-context, not in the foundation.

```css
.glossary dl {
  display: grid;
  grid-template-columns: max-content 1fr;
  gap: var(--space-3) var(--space-4);
}
.glossary dt { margin: 0; }
.glossary dd { margin: 0; }
```

**To change the 65ch max-width:** override per-context.

```css
.my-component ul,
.my-component ol,
.my-component dl { max-width: none; }
```

---

## Print rules

The foundation includes a `@media print` block:

- `list-style-position: outside` — markers print in the margin, not in the content flow
- `<li>` gets `page-break-inside: avoid` and `orphans: 2; widows: 2;` so short items don't break across pages and single items aren't stranded at the top or bottom of a page
- `<dt>` gets `page-break-after: avoid` so a term doesn't print at the bottom of a page separated from its definition

---

## A note on lists in UI patterns

When patterns use `<ul>`/`<li>` markup for UI rows (status cards, activity lists, navigation), they will typically reset the foundation's styling:

```css
.status-rows {
  list-style: none;
  padding-inline-start: 0;
  max-width: none;
}
.status-rows li {
  margin-bottom: 0;
  /* row styling here */
}
```

This is by design. Keeping content lists and UI list patterns separate makes both easier to reason about — the foundation handles the typographic case, patterns handle the structural case.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
