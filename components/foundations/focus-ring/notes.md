# focus-ring — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 11 May 2026.
**Amendment tested:** Chrome on macOS by Edwin, 7 June 2026 (see Foundation regression and fix below).

---

## 01 — Renders correctly without CSS loaded

**Pass.**

With both `tokens.css` and `focus-ring.css` failing to load (file opened from a location where the relative paths could not resolve), the reference page reverted to user-agent default rendering. Every element remained focusable via keyboard. Buttons, links, inputs, and tabbable elements all showed the browser's native focus indicator. No content was hidden, no relationships were obscured, no functionality was lost.

This is the expected behaviour for a foundation that styles existing focusable behaviour rather than creating it. The semantic HTML carries the focusability; the stylesheet only changes the appearance of the indicator.

**Caveat:** the user-agent default focus ring varies by browser and can be hard to see on some surfaces. This is exactly why the system applies its own ring — but the absence of our ring is not a structural failure, only a visual regression.

---

## 02 — Works at 200% zoom

**Pass.**

The 2px outline scaled with the rest of the page at 200% zoom. The 2px offset remained proportional. No content was clipped, no overlap appeared between adjacent focusable elements, no horizontal scroll was introduced. The reference page remained fully usable; every focusable element remained reachable and the ring remained clearly visible.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width (tested via Chrome dev tools responsive mode), the focus ring rendered identically to its desktop appearance — outline width and offset are independent of viewport. No clipping, no overlap, no horizontal scroll.

---

## Edge cases noted during testing

**Radio group focus is one tab stop, not one per radio.** Browsers treat a radio group (`<input type="radio" name="x">` elements sharing a `name`) as a single tab stop. Tabbing into the group focuses either the currently-checked radio or the first radio if none are checked; arrow keys move focus between radios within the group; Tab moves out of the group entirely. This means it is not possible to see the focus ring on an *unchecked* radio without first checking it. This is correct browser behaviour per WAI-ARIA Authoring Practices for radio groups, not a bug in the focus ring.

To be documented in `controls/radio` when that component is built — the arrow-key navigation pattern is not widely known and needs explicit guidance for forkers.

**`:focus-visible` heuristic varies by element type.** Tab navigation reliably triggers the ring on every element. Mouse click on a button does not show the ring (correct). Mouse click on an `<input type="text">` does show the ring (correct — the input becomes editable). This is standard browser behaviour and the system inherits it without intervention.

**Forced-colors mode.** Tested by enabling Chrome's "Emulate CSS forced-colors: active" in dev tools. The ring switched to the system `Highlight` colour as expected. Authored `--focus` value is overridden, which is the correct behaviour.

**Disabled buttons.** A `<button disabled>` is not focusable and correctly received no ring.

**`tabindex="-1"`.** Not in the tab order, correctly excluded by the selector.

**Outline collision with rounded corners.** On elements with `border-radius`, modern Chromium follows the rounded shape for the outline. Older browsers will show a rectangular outline around a rounded element. Acceptable degradation.

---

## Foundation regression and fix (7 June 2026)

A system-wide regression was caught during `methodology` component testing: **no focus ring rendered on any element**, even though focus was landing correctly (keyboard tabbing reached elements, activation worked, and `document.activeElement` returned the focused element).

**Root cause.** `focus-ring.css` paints the ring with `outline: var(--border-focus) solid var(--focus)`, but `--border-focus` was **not defined** in the committed `tokens.css` — confirmed in the console, where the property resolved to an empty string. A `var()` reference to an undefined custom property with no fallback invalidates the whole declaration at computed-value time, so the `outline` shorthand was dropped silently, with no error. The ring never painted, on any focusable element in the system.

Two things had masked this until now:
1. macOS Chrome does not tab to non-form focusable elements unless "Keyboard navigation" is enabled in System Settings — so the elements that would most obviously show the gap (links, `summary`, `tabindex` elements) were being skipped in the tester's default configuration.
2. methodology's `<summary>` was the first non-form, natively-focusable element to be keyboard-tested after the regression was present, which is what finally surfaced it.

**Fix.**
1. Added `--border-focus: 2px;` to `tokens.css` section 6 (the token the foundation already expected).
2. Added a fallback in this foundation: `outline: var(--border-focus, 2px) solid var(--focus)` — so a fork that drops the token degrades gracefully instead of silently disabling every focus indicator. For a clone-and-own system this is the more important half of the fix: the failure mode was invisible, and the fallback removes the trap.

**Re-test (7 June 2026, Chrome on macOS, keyboard navigation enabled).** All pass:
- `summary` (methodology) — ring renders under keyboard focus.
- A form control (button) and a link — ring renders.
- `.table-wrap` and `pre[tabindex="0"]` (table) — ring renders. **This closes the previously parked focus-ring-on-non-form-focusable-elements investigation:** the root cause was the missing token, not the elements themselves.

**Consequence recorded.** The focus-ring observations in the `button`, `checkbox`, and `radio` notes predated this fix (they passed under a token state that no longer matched the committed `tokens.css`). Each has been re-validated for focus rendering and annotated 7 June 2026.

---

## Open questions for v0.2 and beyond

- The 2px offset may need revisiting when `controls/checkbox` and `controls/radio` are built — depending on the styling we choose, the ring may need to sit closer to the visible mark. *(Resolved in practice: checkbox and radio both tested with the ring sitting cleanly outside the mark.)*
- If `display/code` introduces focusable code blocks (for copy buttons), test focus ring against the mono-typed background.
- If a fork introduces an activity-coloured surface, document the focus colour override pattern at that point.
- Engine coverage beyond Chromium (Firefox / Gecko, Safari / WebKit) not yet tested. Standards-compliant CSS used throughout, so no known engine-specific risks. Broaden coverage if a fork reports an issue.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
