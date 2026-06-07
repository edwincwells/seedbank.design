# methodology

The "how this was calculated" disclosure. A reusable affordance that puts methodology one click from any estimate — an eco-stat, a carbon figure, a provenance stage, a dashboard number — without burying it in a footnote.

This is the second component in the **ecological** layer and the one that makes Principle 02's "methodology is always one tap or click away" operable as a repeatable mechanic rather than a per-component treatment.

---

## Why it's built on `<details>`

Native `<details>`/`<summary>`. This is deliberate and load-bearing, not a shortcut:

- **No JavaScript.** The toggle is native HTML. This component is the layer's proof that an interactive disclosure needs no script at all — which is what makes it safe for static-site and no-JS deployments. Everything else in the layer that wants a disclosure composes this.
- **Degrades with no CSS.** Without the stylesheet, the browser draws its own disclosure triangle and the summary still toggles the body open. Nothing in `methodology.css` is required for the component to *work* — the CSS only restyles a control that already functions. That is the "designed for the edges" line holding: the affordance survives the loss of both JS and CSS.

This is the cleanest graceful-degradation story in the layer, and it's worth keeping that way. A fork tempted to replace `<details>` with a JS toggle would lose exactly the property that makes this component matter.

---

## What it styles

- `.methodology` — the `<details>` container
- `.methodology > summary` — the always-visible trigger (muted text + chevron)
- `.methodology-body` — the expanded detail, set off by a faint left-rule
- `.methodology-gap` — marks explicit unknowns in the subtle colour

---

## Markup

```html
<details class="methodology">
  <summary>How this was calculated</summary>
  <div class="methodology-body">
    <p>Lifecycle estimate, cradle-to-gate. Scope 1 and 2 only.</p>
    <p><span class="methodology-gap">Transport excluded — data unavailable.</span></p>
    <p>Source: <a href="#" rel="external">supplier disclosure, 2024</a>.</p>
  </div>
</details>
```

**Closed by default.** The summary is always visible, so the *existence and availability* of methodology is never hidden — only the detail is deferred. That satisfies "one click away" without cluttering a dashboard of stats. To foreground the detail (a single hero figure where the calculation is the point), add the `open` attribute:

```html
<details class="methodology" open>
```

---

## The summary, and why it isn't a link

The summary reads as **muted text plus a chevron — not link colour.** A disclosure is not navigation; colouring it like a link would falsely imply it goes somewhere, against the system's don't-imply-navigation stance (the same reason links use `rel="external"` rather than protocol-sniffing). The interactivity is carried instead by the chevron, the pointer cursor, the focus ring, and — for assistive tech — the native expanded/collapsed announcement that `<summary>` provides for free.

Provide your own summary wording; it's context-specific ("How this was calculated", "How this was estimated", "What's measured here"). The component styles the trigger; the words are the fork's.

---

## The chevron

Drawn in **pure CSS** — a small square showing only its right and bottom borders, rotated -45° (points right, closed) and 45° (points down, open), in `currentColor`. No SVG, no data-URI, no hardcoded hex. This escapes the SVG-colour exception that the checkbox and select carry, the same way the radio dot does — a fork changing surface or text colours never has to re-encode anything here. The rotation animates, and drops to an instant flip under `prefers-reduced-motion`.

The native marker is suppressed two ways for cross-browser coverage (`list-style: none` plus `::-webkit-details-marker`). If the stylesheet doesn't load, the native marker returns and the control still works.

---

## Composition — sibling of the estimate, never a child

methodology attaches as a **sibling** beneath the thing it explains, inside the same composing container:

```html
<div class="some-card">
  <div class="eco-stat"> … </div>
  <details class="methodology"> … </details>
</div>
```

It is never nested inside `eco-stat` — that primitive stays atomic by design, and methodology stays independently reusable for anything else that needs a disclosure. The card or dashboard cell owns the relationship between them.

---

## The body, and marking gaps

The body is a **free prose container**, not an imposed schema. The recommended honest shape is: what is measured, what is excluded, the method or standard followed, and a source. Write it as ordinary prose.

Mark what is *not* covered with `.methodology-gap` so the gap reads as a gap in the subtle colour rather than hiding:

```html
<p><span class="methodology-gap">Transport excluded — data unavailable.</span></p>
```

**On `.methodology-gap` as a local affordance.** The general, reusable data-gap treatment is deferred to its own primitive later in the layer. This class is a local hook so methodology can mark unknowns honestly now; when the standalone gap primitive lands, this may fold into it. The divergence is intentional and flagged here so it isn't mistaken for the final shape.

---

## States

| State | Trigger | Effect |
|---|---|---|
| Closed | _(default)_ | Summary visible, body hidden |
| Open | `open` attribute | Summary visible, body shown, chevron points down |
| Hover | `:hover` on summary | Summary text shifts from muted to full `--text` |
| Focus | `:focus-visible` on summary | Focus ring from `foundations/focus-ring` |

Open/closed is the native `<details>` state, carried by the `open` attribute — which is attribute-driven by construction, consistent with the system's state-from-attributes convention.

---

## Tokens used

| Token | Used for |
|---|---|
| `--font-sans` | Summary and body text |
| `--text-sm` | Summary and container size |
| `--text-muted`, `--text` | Summary default / hover colour |
| `--text-subtle` | Gap marker colour |
| `--text` | Body colour |
| `currentColor` | Chevron (inherits summary colour, both modes) |
| `--border-thick` | Chevron stroke width |
| `--border`, `--border-hairline` | Body left-rule |
| `--space-2`, `--space-3`, `--space-4` | Gap, body offset, indent |

No new tokens, no hardcoded values, no SVG data-URI. Dark mode is automatic via the tokens — there is no component-level dark block.

---

## Accessibility

- **Keyboard:** `<summary>` is natively focusable and in the tab order. The focus ring is provided by `foundations/focus-ring`, whose selector set includes `summary` — so the disclosure shows the system ring under keyboard navigation with no extra work. Always include `focus-ring.css` in pages that use methodology.
- **Screen readers:** `<details>`/`<summary>` is announced as a disclosure with an expanded/collapsed state natively. The muted (non-link) styling does not change this — semantics come from the element, not the colour.
- **No colour-only meaning:** the chevron and the gap marker both communicate through more than hue (rotation; the word "excluded" alongside the subtle colour), so they survive on e-ink and monochrome print.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/focus-ring/focus-ring.css` (the summary needs it)
3. Include `components/foundations/prose/prose.css` (the body is prose) and `components/foundations/links/links.css` if your body contains a source link
4. Include `components/ecological/methodology/methodology.css`
5. Write the `<details>` markup as a sibling beneath the estimate it explains

---

## How to override

**To open every methodology by default:** add the `open` attribute in your markup. There is no CSS toggle to flip — open is a native attribute, so this is a content choice, not a style override.

**To keep the native disclosure triangle:** remove the marker-suppression and chevron rules (`list-style`, `::-webkit-details-marker`, `summary::before`). Simpler, at the cost of cross-platform inconsistency.

**To restyle the summary as a link-coloured affordance:** set `color: var(--link)` on `.methodology > summary`. Consider first whether it will read as "navigates somewhere" to your users — the muted default is a deliberate honesty choice.

**To remove the body left-rule:** drop `border-left` and `padding-left` from `.methodology-body`.

---

## Print rules

- The body is **forced open** in print regardless of its on-screen state — the calculation belongs on a printed record, and honest uncertainty shouldn't vanish when a page is printed.
- The chevron is suppressed (a control affordance with no meaning on paper).
- The left-rule prints, as editorial framing.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
