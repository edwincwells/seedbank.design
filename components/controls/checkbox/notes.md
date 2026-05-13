# checkbox — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 12 May 2026.

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

## Open questions for v0.2 and beyond

- **SVG checkmark colour in heavily-remapped forks.** The hardcoded `#fafaf7` works for the default palette because `--action` resolves to colours dark enough for a light checkmark to read against them. A fork that remaps `--action` to a lighter colour (pale yellow, for instance) would lose checkmark contrast. The README documents the re-encoding requirement; v0.2 might add a contrast-checking note to the adapting guide.
- **Indeterminate state without JS.** Some communities may not be able to wire JS at all (e.g. static-site-only deployments). The indeterminate state isn't reachable in those contexts. Worth a documented note in the adapting guide about which features require JS — checkbox indeterminate, parent-child sync, button aria-busy toggling. Currently scattered across READMEs; would benefit from a consolidated reference.
- **`:has()` fallback.** Works in current Chrome (and all current browsers per caniuse). The graceful-degradation behaviour in older browsers (label stays full-opacity while input dims) isn't worse than the no-CSS state. No action needed, but worth tracking if forks report it as an issue.
- Print preview check, as flagged for prior components — confirmed working for checkbox specifically; still worth adding to the standing checklist.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
