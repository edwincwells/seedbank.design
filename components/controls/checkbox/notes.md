# checkbox — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 12 May 2026.
**Amendments tested:** Chrome on macOS by Edwin, 18 May 2026 (see Controls-pass amendments below).

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the five stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` in Chrome dev tools. With no CSS applied:

- Every checkbox fell back to the browser's native checkbox chrome — fully functional, clickable, visibly a checkbox
- Wrapping `<label>` patterns still associated text with their inputs — clicking the label text still toggled the checkbox
- Checked, unchecked, disabled, and indeterminate states all rendered via UA defaults
- The `<fieldset>` and `<legend>` group fell back to the browser's default fieldset rendering (the classic UA border with inset legend)
- The parent-child JS continued working — checkboxes synchronised correctly even though the custom indeterminate styling was lost (the UA indeterminate rendering took over)
- Invalid checkbox lost the red border (CSS effect) but the linked error message remained visible and accessibly associated via `aria-describedby`
- Wrong-use blocks degraded to plain paragraphs

The custom box geometry, SVG checkmark, activity-coloured fill, sunken background, indeterminate dash, and disabled-label dimming were all lost — expected, all CSS effects. Functionality, semantics, and accessibility associations of every checkbox remained intact. Form submission behaviour unaffected.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome:

- All checkboxes scaled proportionally — the 18px boxes effectively rendered at 36px without geometry breaking
- The SVG checkmark scaled cleanly without pixelation — vector rendering paid off
- Label text alongside each checkbox scaled correctly and stayed aligned with the box's top edge via the `align-items: flex-start` rule in the demo CSS
- Indented children remained visually nested under their parent
- The fieldset/legend group remained coherent — items stayed in their column, legend stayed attached
- Focus rings remained visible at appropriate proportions

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width:

- Single checkboxes fit comfortably with their labels
- Long label text (the "Receive notifications about upcoming work parties" example) wrapped cleanly — the box stayed aligned with the first line via the flex layout, the text wrapped underneath without disrupting alignment
- The parent-child indented pattern fit without horizontal scroll; the 26px indent (18px box + 8px gap) worked at the narrow viewport
- The fieldset group remained usable; legend wrapped where needed without breaking the group structure
- Error message under the invalid checkbox wrapped correctly
- Wrong-use blocks held content without horizontal scroll
- Inline `<code>` references in paragraphs wrapped correctly

---

## Edge cases noted during testing

**Focus ring visibility on the 18px box.** Focus ring rendered clearly around the small checkbox. The `outline`-based approach gave the ring its own visual space without making the small target feel cramped.

**Focus ring on a checked checkbox.** Both the `--action` fill and the blue focus ring rendered simultaneously without visual clash — the ring sits outside the box cleanly.

**Focus ring on an invalid checkbox.** The red border and the blue focus ring both rendered legibly together. Two distinct signals, both clear.

**Indeterminate visual.** The standalone indeterminate demo loaded with the horizontal dash visible, not a tick. The JS-on-load pattern worked as documented.

**Parent-child sync.** Tested in both directions:
- Clicking individual children moved the parent through the three states correctly: 0 checked → unchecked, 1-2 checked → indeterminate, all 3 checked → checked
- Clicking the parent synced all children to match the parent's new state
- Initial state on load reflected the markup correctly (2 of 3 children checked → parent indeterminate)

**Label click target.** Clicking anywhere on the label text — including the long-text label that wrapped to a second line — toggled the checkbox. The wrapping-`<label>` pattern delivered the full-label click target without any extra wiring.

**Disabled label dimming via `:has()`.** Both the input and the label text rendered at 0.5 opacity for disabled checkboxes. Hovering the disabled label showed `cursor: not-allowed` as expected. `:has()` worked correctly in Chrome.

**Keyboard activation.** Tab to a checkbox, Space toggled it, focus remained on the checkbox after toggling. Standard native checkbox behaviour preserved by `appearance: none` not stripping the keyboard handling.

**Form submission behaviour.** Confirmed in a quick form test: only checked items submitted; the indeterminate state has no submission effect (parent submits based on its actual `checked` property, regardless of indeterminate).

**Hover state on the box.** Hovering the unchecked box shifted the border to `--text` colour. Hovering the label area without hovering the box itself did not trigger the border change — hover is properly scoped to the input element.

**Print preview.** The checkmark inverted correctly to dark on the now-transparent background. Indeterminate dash inverted the same way. Printed checkboxes remained readable; the printed page accurately recorded which items were checked.

**SVG colour correctness.** The hardcoded `#fafaf7` rendered correctly against the `--action` fill in light mode. In dark mode, `--action` resolves to the dark-mode green (`#7fc960`) and the light checkmark remained legible against it — sufficient contrast despite both colours being on the lighter side of the dark surface. Worth noting: the SVG colour is hardcoded light, so it does NOT invert in dark mode. The check stays legible by virtue of the `--action` fill being darker than the checkmark in both modes. Confirms the SVG-colour-encoding trade-off is acceptable for the default palette.

**Dark mode.** All states rendered correctly:
- Sunken background remained visibly distinct from page surface
- Activity fill on checked boxes read correctly in dark mode green
- SVG checkmark remained legible (see SVG note above)
- Invalid red border read clearly against dark surface
- Indeterminate dash followed the same logic as the checkmark

---

## Controls-pass amendments (18 May 2026)

Three changes made during the controls-layer pass alongside the radio v0.1 work. All resolved in CSS; no foundation-level changes required for this pass.

**1. `[disabled]` → `:disabled` on per-input rules.**

Radio testing surfaced that the attribute selector misses the fieldset-disabled case — inside `<fieldset disabled>`, the browser marks descendants as `:disabled` but does NOT add the `disabled` attribute to the markup. The same issue applied to checkbox: any checkbox inside a `<fieldset disabled>` would have shown the wrong cursor and would have been hoverable.

Resolution: swapped the per-input rules to use `:disabled` (pseudo-class). The `fieldset[disabled]` selectors elsewhere are correct as-is. The existing reference-page examples don't exercise the fieldset-disabled case, so no visible regression to test against from the existing demos.

**2. Clear-on-checked for invalid state.**

The "I agree to the terms" case: a required checkbox fails validation on submit, the user checks the box, but the red border stays until JS removes `aria-invalid`. Same problem identified during radio testing — visually confusing and dishonest about state.

Resolution: added two CSS rules:
- `label:has(input:checked) input[type="checkbox"][aria-invalid="true"]` — clears the border on the now-checked invalid checkbox by re-instating the `--action` border via specificity
- `fieldset:has(input:checked) input[type="checkbox"][aria-invalid="true"]:not(:checked)` — for grouped checkbox cases, clears the border on unchecked invalid siblings when any checkbox in the group has been checked

**3. `.error-message` convention for error message clearing.**

Added rules that hide the `.error-message` element (using `visibility: hidden` to preserve layout) when the user has satisfied the requirement:
- Single checkbox: `label:has(input:checked) + .error-message` — error follows the wrapping label as a sibling
- Grouped checkbox: `fieldset:has(input:checked) .error-message` — error inside the fieldset

The reference page's invalid demo error `<p>` now carries `class="demo-error error-message"` so the convention is exercised.

### Amendment receipts

**Disabled-via-fieldset behaviour:** verified by wrapping an existing checkbox group in `<fieldset disabled>` via Chrome dev tools. Cursor became `not-allowed` on both the input and the label; hover did not trigger the border colour shift. The previously-broken behaviour is now correct.

**Clear-on-checked for the "I agree" case:** in the existing invalid demo, checking the box cleared both the red border AND the error message immediately. The label and asterisk remained visible; only the error `<p>` was hidden. Re-unchecking the box brought both back, as expected — the rule fires on the live `:checked` state, not as a one-way transition.

**Layout integrity on clear:** confirmed — `visibility: hidden` preserved the message's layout space. No vertical jump on check or uncheck.

**Grouped-checkbox clear behaviour:** not exercised by the existing reference page (no grouped checkbox is marked invalid). The rule is in place; the canonical "select at least one" composition will surface during real fork usage.

---

## Open questions for v0.2 and beyond

- **SVG checkmark colour in heavily-remapped forks.** The hardcoded `#fafaf7` works for the default palette because `--action` resolves to colours dark enough for a light checkmark to read against them. A fork that remaps `--action` to a lighter colour (pale yellow, for instance) would lose checkmark contrast. The README documents the re-encoding requirement; v0.2 might add a contrast-checking note to the adapting guide.
- **Indeterminate state without JS.** Some communities may not be able to wire JS at all (e.g. static-site-only deployments). The indeterminate state isn't reachable in those contexts. Worth a documented note in the adapting guide about which features require JS — checkbox indeterminate, parent-child sync, button aria-busy toggling. Currently scattered across READMEs; would benefit from a consolidated reference.
- **`:has()` fallback.** Works in current Chrome (and all current browsers per caniuse). The graceful-degradation behaviour in older browsers (label stays full-opacity while input dims, error stays visible after check) isn't worse than the no-CSS state. No action needed, but worth tracking if forks report it as an issue.
- **Grouped-checkbox invalid pattern.** The "select at least one" group case is covered by the clear-on-checked rule, but the reference page doesn't demonstrate it. Worth adding a demo block in a future iteration so the rule is visibly exercised in the reference, not just present in the CSS.
- **The `.error-message` convention is now load-bearing across checkbox, radio, and select.** When `form-field` is built, this convention should be codified explicitly: error messages live inside the field's wrapping container (label/fieldset/wrapper div) and carry the `.error-message` class.
- Print preview check, as flagged for prior components — confirmed working for checkbox specifically; still worth adding to the standing checklist.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
