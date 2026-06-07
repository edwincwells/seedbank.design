# methodology — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 7 June 2026.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

With the stylesheet `<link>` tags and the reference-page `<style>` block removed:

- Each `<details>` rendered with the browser's native disclosure triangle and the summary stayed clickable. The toggle still worked — clicking the summary opened and closed the body. The disclosure functions with no CSS, which is the headline degradation property.
- The `open` example rendered expanded; the default examples rendered collapsed.
- The open body read as plain prose — scope, the excluded/gap line, and the source link all legible.
- The gap text lost its subtle colour (a CSS effect) but the word "excluded" still carried the meaning.
- The composed example showed the eco-stat and the methodology disclosure stacked as siblings, structure intact, no nesting.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom: summary text and chevron scaled together and stayed baseline-aligned; the chevron rotated correctly on open/close at the larger size; the body left-rule and indent scaled and prose wrapped without clipping; no horizontal scroll.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px: the summary held on one line where space allowed and wrapped gracefully when not, the chevron staying with the text; body prose wrapped cleanly within the indent and the left-rule held; the composed card example fit without horizontal scroll.

---

## Edge cases checked

All passed.

- **No-JS toggle** — with JavaScript disabled, the disclosure still opened and closed. Native `<details>`; the core claim of the component holds.
- **Chevron direction** — closed points right, open points down. Rotation reads correctly.
- **Chevron optical centring** — read as centred against the text at default and compact sizes; no nudge needed.
- **Reduced motion** — with `prefers-reduced-motion` enabled the chevron flipped instantly, no animated rotation, but still flipped.
- **Keyboard focus on summary** — see the foundation finding below. After the focus-ring fix, Tab to the summary showed the seedbank ring, and Enter/Space toggled it.
- **Print reveals closed body** — printing a collapsed methodology revealed the body (forced open) with the chevron gone.
- **Dark mode** — `prefers-color-scheme` swapped automatically via tokens; summary, chevron (currentColor), body, gap, and left-rule all read against the dark surface.
- **Composed with eco-stat** — the sibling relationship read correctly: stat above, disclosure below, one container, no collision.

---

## Foundation finding surfaced during this component's testing

Keyboard focus on the summary initially showed **no focus ring at all** — and the same was true of the links inside the body, and (on investigation) every focusable element in the system. This was **not a methodology bug**; methodology's files were not changed.

Root cause: `focus-ring.css` paints the ring with `outline: var(--border-focus) solid var(--focus)`, but `--border-focus` was **not defined** in the committed `tokens.css` (confirmed in console — the custom property resolved to an empty string). An undefined custom property with no fallback invalidates the whole `outline` shorthand, so the ring silently did not render anywhere, with no error.

Fix (committed to the foundation, not this component):
1. Added `--border-focus: 2px;` to `tokens.css` section 6.
2. Added a fallback — `outline: var(--border-focus, 2px) solid var(--focus)` — in `focus-ring.css`, so a fork that drops the token degrades gracefully rather than silently disabling all focus indicators.

Re-test after the fix: the summary ring rendered correctly under keyboard focus; a form control and a link were spot-checked and also correct; `table.html` (`.table-wrap` / `pre[tabindex="0"]`) was re-checked with macOS keyboard navigation on and rendered the ring correctly — which closes the previously parked non-form-focusable focus-ring investigation.

Note for the foundation: the focus-ring pass verdicts recorded in `button`, `checkbox`, and `radio` notes predate this fix and are stale; a `focus-ring` re-test pass across those components is warranted.

---

## Open questions for later

- **`.methodology-gap` vs the future data-gap primitive** — this local class is a placeholder; when the standalone gap treatment lands, decide whether to fold it in.
- **Source-link convention** — settle whether ecological sources are always external links and whether they want a consistent affordance across the layer.
- **Accordion behaviour** — native `<details name="...">` exclusive grouping is available if a fork wants it; out of scope here.
- **Print preview** — run as part of this test (print-forces-open confirmed); recommend keeping it in the standing checklist for components that make print claims.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
