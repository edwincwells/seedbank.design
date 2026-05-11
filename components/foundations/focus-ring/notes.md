# focus-ring — testing notes

Tested against the three criteria in `TESTING.md`.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

With both `tokens.css` and `focus-ring.css` disabled, every element on the reference page reverts to user-agent default focus styling. Buttons, links, inputs, and tabbable elements all remain focusable via keyboard and show the browser's native focus indicator. No content is hidden, no relationships are obscured, no functionality is lost.

This is the expected behaviour for a foundation that styles existing focusable behaviour rather than creating it. The semantic HTML carries the focusability; the stylesheet only changes the appearance of the indicator.

**Caveat:** the user-agent default focus ring varies by browser and can be hard to see on some surfaces. This is exactly why the system applies its own ring — but the absence of our ring is not a structural failure, only a visual regression.

---

## 02 — Works at 200% zoom

**Pass.**

The 2px outline scales with the rest of the page at 200% zoom. The 2px offset remains proportional. No content is clipped, no overlap appears between adjacent focusable elements, no horizontal scroll is introduced.

Tested in:
- Firefox (Ctrl+= to 200%)
- Chromium (Ctrl+= to 200%)

The reference page remains fully usable at 200% zoom; every focusable element remains reachable and the ring remains clearly visible.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width, the focus ring renders identically to its desktop appearance — outline width and offset are independent of viewport. No clipping, no overlap. The ring sits within the element's outer bounds plus 2px on each side, which fits comfortably in a 320px viewport for any sensibly-sized control.

Tested in browser dev tools responsive mode at 320×568 (older iPhone reference size).

---

## Edge cases noted during testing

**`:focus-visible` heuristic varies.** Browsers determine whether to show focus-visible differently. Tab navigation reliably triggers it; mouse click on a button reliably does not (Chromium, Firefox); mouse click on an `<input type="text">` reliably does (because the input becomes editable). This is correct browser behaviour and the system inherits it without intervention.

**Forced-colors mode.** Tested by enabling Chromium's "Emulate CSS forced-colors: active" in dev tools. The ring switches to the system `Highlight` colour as expected. Authored `--focus` value is overridden, which is the correct behaviour.

**Disabled buttons.** A `<button disabled>` is not focusable and correctly receives no ring. Verified.

**`tabindex="-1"`.** Not in the tab order, correctly excluded by the selector. Verified.

**Outline collision with rounded corners.** On elements with `border-radius`, the outline follows the element's bounding box, not the rounded corners — this is browser default behaviour. Modern browsers (Chromium 94+, Firefox 88+, Safari 16.4+) now follow `border-radius` for outlines automatically. Older browsers will show a rectangular outline around a rounded element. Acceptable degradation.

---

## Open questions for v0.2 and beyond

- The 2px offset may need revisiting when `controls/checkbox` and `controls/radio` are built — depending on the styling we choose, the ring may need to sit closer to the visible mark.
- If `display/code` introduces focusable code blocks (for copy buttons), test focus ring against the mono-typed background.
- If a fork introduces an activity-coloured surface, document the focus colour override pattern at that point.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
