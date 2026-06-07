# radio — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 18 May 2026.
**Focus-ring re-validated:** Chrome on macOS by Edwin, 7 June 2026 (see Focus-ring re-validation below).

Three issues surfaced during the first test pass and were resolved in `radio.css` (see Findings below). Tests were re-run with the updated CSS in place; the receipts below reflect the post-fix state.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the five stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` in Chrome dev tools. With no CSS applied, every `<input type="radio">` rendered with the browser's default UA styling — visibly a radio, distinct from surrounding text and from checkboxes. Radios sharing a `name` attribute remained mutually exclusive — clicking one deselected the others in its group. The `checked` attribute on pre-selected radios continued to select them. The `required` attribute remained functional (UA validation message appeared on submit attempt when no radio in a required group was selected). The `disabled` attribute on individual radios produced the UA-default greyed state. `<fieldset disabled>` disabled every descendant radio. The `<legend>` text remained visible and associated with its `<fieldset>`. Semantic interactivity intact: clicking, keyboard navigation (Tab past group, arrow keys within group), form submission all worked. The custom inner dot disappeared (it's a pure-CSS `::before` effect) — the UA's native filled-circle selected state took over instead. The wrapping `<label>` click target continued to work via native label semantics.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome, every radio remained readable and proportional. The outer circle and inner dot scaled together — the dot stayed centred via the `transform: translate(-50%, -50%)` rule regardless of zoom level. The `--action` ring around the dot remained visible at the expected ratio. Padding and gap between radios scaled with the font size. Legend text and label text remained legible and proportional. The disabled state's dimming remained visible across both individual and group-disabled cases. The required asterisk in the legend scaled correctly. The invalid state's red borders remained visible across the group, and the clear-on-selected behaviour worked at zoom — selecting a radio cleared the red across the group as expected.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width (tested via Chrome dev tools responsive mode), radio groups filled the available width — the wrapping fieldsets did not overflow. Long option labels (e.g. "Café (closed for refurbishment)", "No, sending a proxy") wrapped cleanly rather than forcing horizontal scroll. The radio circle stayed at its size; only the label text wrapped, sitting cleanly to the right of the circle thanks to `align-items: flex-start` on the wrapping label. The legend, help text, and error message wrapped cleanly. The wrong-use blocks rendered legibly.

---

## Edge cases noted during testing

**Dark mode.** The inner dot remained visible against the `--action` fill in dark mode. Both tokens re-resolve via `prefers-color-scheme`, so the dot adapted automatically without any re-encoding — the payoff of the pure-CSS approach.

**Focus ring on radios.** The focus-ring foundation's `outline` approach sat cleanly outside the circle's border in default, selected, and invalid states. No visual clash with the red invalid border. *(This observation predates the 7 June 2026 foundation fix and has been re-confirmed — see Focus-ring re-validation below.)*

**Selected + hover interaction.** Hovering over a selected radio did not produce a visible flash. The hover rule changes `border-color` to `--text`, but on a selected radio the border is `--action`, and the selected fill is also `--action` — the hover override would only show as a subtle outline shift that wasn't visible in practice. No `:not(:checked)` qualifier needed.

**Invalid + hover interaction.** Hovering over an invalid radio preserved the `--error` red border — the explicit `[aria-invalid="true"]:hover` rule prevented the default-hover dark colour from masking the invalid signal.

**Keyboard navigation.** Tab moved focus past the entire group as expected. Arrow keys (↑ ↓ ← →) moved within the group and changed selection, skipping the disabled "Café" radio in the pickup group. The fieldset-disabled "Membership tier" group could not be focused — arrow keys did not enter it from a preceding tab stop. All native, no JS, exactly as documented.

**Required group validation.** Submitting a form with no radio selected in a required group triggered the UA validation message. Applying `aria-invalid="true"` via the demo's static markup showed the red borders across the group correctly.

**Group-disabled stacking.** The per-input opacity reset rule (`fieldset[disabled] input[type="radio"] { opacity: 1 }`) worked as intended — dimming stayed consistent at 0.5 across the whole disabled group rather than multiplying to ~0.25.

**Print rendering.** The selected radio's inner dot remained visible against the now-transparent outer circle. The dot flipped from `--surface` to `--text` automatically via the `@media print` rule. No SVG re-encoding required — print-mode behaviour is a few lines, not a parallel data-URI library.

**No-deselect behaviour.** Verified by clicking a radio in the "Dietary requirements" optional group and confirming there was no way to return to "nothing selected" without choosing "No requirements" — the documented escape hatch worked as designed.

**Clear-on-selected behaviour (post-fix).** Selecting any radio in the invalid demo group cleared the red borders across all radios in the group. The selected radio showed its standard `--action` green border; the unselected radios reverted to the default `--border-strong`. The visual matched the user's mental model: "I answered, so the form should stop complaining."

---

## Findings that surfaced during testing

Three issues surfaced during the first test pass. All resolved in CSS; no foundation-level changes required.

**1. Disabled-via-fieldset radios showed pointer cursor and hover state.**

The individual-disabled case worked correctly — pointer became `not-allowed`, hover-override prevented the border colour shift. The fieldset-disabled case did not.

Root cause: the CSS used `input[type="radio"][disabled]` (attribute selector) for both the `cursor: not-allowed` rule and the hover override. Inside `<fieldset disabled>`, the browser marks every descendant radio as `:disabled` (pseudo-class), but does NOT add the `disabled` attribute to the markup. The attribute selector missed the fieldset-disabled path.

Resolution: swap `[disabled]` → `:disabled` on the per-input rules. The pseudo-class catches both paths (attribute-driven and fieldset-driven). The `fieldset[disabled]` selectors elsewhere are correct as-is, since the fieldset itself genuinely has the attribute.

**Almost certainly affects checkbox and select too.** Both controls use the same `[disabled]` attribute selector for their per-input rules. The visible symptom only appears when the control is inside a `<fieldset disabled>`, which neither reference page demonstrates. Worth a janitorial pass when those files are next touched. Flagged in the open questions below.

**2. Invalid state stayed red after a radio was selected.**

In the invalid demo, selecting a radio produced a visually confusing state: the selected radio showed both the `--action` green fill AND the `--error` red border. The user satisfying the validation requirement (selecting at least one) didn't clear the visual.

This was originally correct CSS behaviour — both signals were stacked because both attributes/states applied — but it wasn't honest UX. The validation rule the error was enforcing was "at least one radio selected." The moment that's true, the error is stale.

Resolution: a `:has()`-based rule that clears the red border on the entire group once any radio inside is selected. The selected radio gets its `--action` border via its own `:checked` rule; the unselected radios revert to the default `--border-strong`. The whole group visually agrees that the requirement is met.

**Follow-up: the error message itself also needed clearing.** Initial fix only handled the radio borders, but the error text below the fieldset stayed visible — sending a mixed signal ("you've answered" + "you must answer"). Resolved in a second pass by:
- Moving the error `<p>` inside the `<fieldset>` so a `:has()` lookup from the fieldset can reach it
- Adding `.error-message` as the system convention for error-message elements
- Adding a CSS rule that uses `visibility: hidden` (not `display: none`) to hide the message when any radio in the group is selected, so the layout doesn't jump

The `aria-invalid` attribute is still on every radio in the markup — only JS can remove it. The CSS provides a forward-looking visual that matches the user's mental model. The form itself should remove `aria-invalid` and update the error message content on its next validation cycle.

The `.error-message` convention has been backported to checkbox and select in the same pass.

Caveat documented in both the CSS comment and the README: forms with per-option validation beyond "must select one" need to handle this differently, since the clear-on-selected rule will suppress red that should still be present.

Re-test after the fix confirmed all three criteria pass. The receipts above reflect the post-fix state.

---

## Focus-ring re-validation (7 June 2026)

The focus-ring foundation referenced `--border-focus`, a token that was absent from the committed `tokens.css`, which silently invalidated the `outline` shorthand and removed the focus ring system-wide. This was caught during `methodology` testing and fixed in the foundation (`--border-focus: 2px` added to `tokens.css` §6; a `var(--border-focus, 2px)` fallback added to `focus-ring.css`). See `focus-ring/notes.md` for the full record.

The "Focus ring on radios" observation above predates that fix. Re-tested 7 June 2026 with the fix in place: the ring renders correctly under keyboard focus on the circle in default, selected, and invalid states, sitting cleanly outside the circle with no clash against the `--error` border. (As noted in `focus-ring/notes.md`, a radio group is a single tab stop and arrow keys move within it; the ring follows the focused radio as expected.) **Pass.**

---

## Open questions for v0.2 and beyond

- **A `--control-size-sm` token** (still tracked across selects, inputs, checkbox, radio). Radio adds nothing new to this question but is another instance of the pattern.
- **Toggle/switch component.** If ever added, it would face the same pseudo-element-vs-SVG decision radio just made. Pseudo-element wins again (a switch is a circle that slides), so the trade-off generalises to switches. Worth a sentence in the docs when that work begins.
- **Required-asterisk styling consistency.** Reference page currently uses inline `style="color: var(--error)"` (matching checkbox's pattern); select uses a `.required` class. Worth deciding which is canonical before `form-field` lands.
- **The `.error-message` convention is now load-bearing for three controls.** When `form-field` is built, this convention should be codified explicitly — error messages live inside the field's wrapping container (label/fieldset/wrapper div) and carry the `.error-message` class. Forks that fork this convention should know the CSS rules depend on it.
- Print preview check, as flagged for prior components — still optional, still worth adding to the standing checklist if a habit forms.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
