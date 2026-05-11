# links

Inline link styling for every `<a href>` in the system. Settled once here so links read consistently across body prose, lists, tables, and component content.

This is a **foundation** — it applies globally via element selectors. Include `links.css` once in your fork and every link receives the system styling. No classes needed.

---

## What it styles

- Every `<a>` with an `href` attribute
- Hover, visited, and focus states
- External links marked with `rel="external"`
- Print rendering (URL expansion after the link text)

It does **not** style:
- `<a>` elements without an `href` (placeholders, anchor targets — these are not links)
- Buttons rendered as `<a>` — that's the `button` component's job, applied via class
- Focus ring — handled by `foundations/focus-ring`

---

## The default styling

| State | Effect |
|-------|--------|
| Default | Underlined, `--link` colour (default: blue, same family as `--info`). Underline skips descenders. |
| Hover | Underline thickens from 1px to 2px and sits 1px further from the baseline. Colour unchanged. |
| Visited | Colour shifts to `--link-visited` (desaturated indigo). Browser restricts what else can change. |
| Focus | Focus ring from `foundations/focus-ring`. No override here. |
| External (`rel~="external"`) | Trailing `↗` glyph after the link text |

---

## Three decisions worth understanding

### 1. Always underlined in body prose

Colour-only link signalling fails on monochrome print, e-ink in some modes, and for users with colour vision differences. Per **principle 01 — designed for the edges, not the centre**, underline is the affordance and colour is reinforcement.

This applies to links in headings, lists, table cells, card content — everywhere. There is no carve-out for "clean" prose contexts.

### 2. Visited state is honoured

Some systems suppress `:visited` for privacy or aesthetic consistency. Given the user base — community organisers reading through directories of resources, repair guides, mutual aid links — visited state is a real affordance.

The visited colour is hue-shifted toward indigo, not same-hue-darker. This carries the "you've been here" signal users bring from every other website, without pulling toward `--make` (the system's purple activity token). The desaturation keeps the categorical signal clean.

**To suppress visited styling**: in `tokens.css`, map `--link-visited` to `var(--link)`. Do not edit `links.css`.

### 3. External links are marked by attribute

Authors apply `rel="external"` explicitly:

```html
<a href="https://example.com" rel="external">Example site</a>
```

This renders with a trailing `↗`. The system does **not** auto-detect external links on `http(s):` protocol — auto-detection breaks for same-origin absolute URLs and is too clever. Explicit attribution aligns with how the rest of the system treats state as named, not inferred.

The selector uses `~=` (whitespace-separated match), so combined `rel` values still work:

```html
<a href="https://example.com" rel="external noopener">Still gets the arrow</a>
```

---

## Tokens used

| Token | Used for |
|-------|----------|
| `--link` | Default link colour |
| `--link-visited` | Visited link colour |
| `--text-xs` | URL expansion size in print |
| `--text-subtle` | URL expansion colour in print |

No values are hardcoded except the underline thickness (1px default, 2px hover), the underline offset (2px default, 3px hover), and `text-decoration-skip-ink: auto`. These are below the system's token granularity. `skip-ink` breaks the underline cleanly around descenders (`g`, `p`, `y`, `q`) — a small accessibility win at no cost.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/focus-ring/focus-ring.css`
3. Include `components/foundations/links/links.css`
4. Write semantic HTML — `<a href="...">`. The styling applies automatically.

For external links:

```html
<a href="https://example.com" rel="external">Example</a>
```

---

## How to override

**To change the link colour:** remap `--link` (and `--link-dark` if your fork supports dark mode) in `tokens.css`. The same applies to `--link-visited`.

**To suppress visited styling:** map `--link-visited` to `var(--link)` in `tokens.css`.

**To remove the external link indicator:** delete the `a[href][rel~="external"]::after` block in `links.css`. Or, alternatively, leave it in place and simply stop using `rel="external"` in your content — the indicator only appears when the attribute is present.

**To remove print URL expansion:** delete the `@media print` block in `links.css`. Forks that distribute primarily via screen and never via print may want this. Most should keep it.

**To use a different external indicator glyph:** change the `content` value in the `a[href][rel~="external"]::after` block. Keep the leading non-breaking space (`\00a0`) so the glyph stays attached to the link's last word.

---

## What this foundation does **not** handle

**Button-styled links.** When the `button` component is built, it will apply a `.button` class to `<a>` elements. The `.button` class will override link styling. Both will coexist cleanly because link styling targets bare `a[href]` and button styling will be class-scoped.

**Nav menus.** No nav pattern exists yet. When one is built, nav links may need their own treatment (e.g. unobtrusive underlines, more prominent active state). That decision belongs to the nav pattern, not this foundation.

**Skip links.** Not yet specified. When accessibility patterns land, skip links will likely be styled separately because they have specific visibility requirements (hidden until focused).

---

## Print rules

The foundation expands link URLs in print so they can be transcribed:

```
Read more about this on example.com (https://example.com/path).
```

The expansion is suppressed for:
- In-page anchors (`href` starting with `#`) — no useful URL to transcribe
- JavaScript handlers (`href` starting with `javascript:`) — not a real URL
- Links inside headings — would disrupt heading rhythm

For external links with `rel="external"`, print rendering replaces the `↗` glyph with the URL expansion. On paper, the URL is the more useful artefact.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
