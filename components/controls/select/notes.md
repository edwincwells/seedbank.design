# select — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 18 May 2026.
**Amendments tested:** Chrome on macOS by Edwin, 18 May 2026 (see Controls-pass amendments below).

The body font-family bug surfaced during the first test pass triggered a foundation amendment to `prose.css` (see Findings below). Tests were re-run with the updated foundation and reference page in place; the receipts below reflect the post-amendment state.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the four stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` in Chrome dev tools. With no CSS applied, every `<select>` rendered with the browser's default UA styling — visibly a dropdown, distinct from surrounding text. The `selected` attribute on `<option value="weekly" selected>` continued to select that option. The `required` attribute remained functional — UA validation message appeared on submit attempt. The `disabled` attribute produced the UA-default greyed state. The `<optgroup>` example rendered with the UA's grouping treatment (group labels visibly distinct from options). `<select multiple>` rendered as the UA's default list-box. Option text remained legible across all selects. Semantic interactivity intact throughout: opening, choosing, keyboard navigation, form submission all worked. The placeholder option pattern degraded cleanly — without styling, it's just an option, and the prompt label ("Select a region…") carries the meaning without visual placeholder treatment.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome, every `<select>` remained readable and proportional. The chevron arrow scaled correctly and remained visible against the sunken background. Padding scaled with the font size; option labels did not visually collide with the arrow. The required asterisk, help text, and error message all scaled proportionally. The disabled state's dimming remained visible. The `<select multiple>` list-box rendered without horizontal overflow. The error-state border colour remained `--error` red and was visibly distinct from the hover-state dark border.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width (tested via Chrome dev tools responsive mode), selects filled the available width via `width: 100%`. Long option labels (e.g. "Childcare network", "Decision-making notices") rendered correctly within the closed-state box — Chrome truncates with the OS-default truncation, no horizontal scroll appeared on the page. The chevron held its right position; the right padding correctly cleared it. The label, help text, and error message wrapped cleanly. The `<select multiple>` list-box did not overflow. The wrong-use blocks rendered legibly.

---

## Edge cases noted during testing

**Open-panel rendering across platforms.** Chrome on macOS draws a clean native dropdown panel that doesn't match the system's visual vocabulary — as expected and documented. The control remains usable; the panel is the OS's, not the system's. iOS and Android testing deferred; the documented limitation framing covers them.

**Dark mode.** Arrow visibility was correct in both modes. The light-mode SVG (`#1a1a1a` stroke) renders cleanly against the warm sunken background; the dark-mode SVG (`#f0f0f0` stroke) renders cleanly against the dark sunken surface. The mode switch via `prefers-color-scheme` worked as expected.

**Focus ring on the select.** The focus-ring foundation's `outline` approach sat cleanly outside the select's border with no visual clash. Verified in both default and invalid states — the focus ring remained the same blue regardless of border colour, which is the intended behaviour (focus is its own signal, independent of validation state).

**Invalid + hover interaction.** Hovering over an invalid select preserved the red `--error` border colour — the explicit `select[aria-invalid="true"]:hover` rule prevented the default-hover dark colour from masking the invalid signal.

**`<optgroup>` rendering.** Chrome on macOS rendered group labels with the UA's bold-italic-indented styling. The component does not style `<optgroup>` labels by design — confirmed that the half-styled alternative would have looked worse than the OS default.

**`<select multiple>` `:checked` styling.** The two initial selections ("Repair café" and "Childcare network") were highlighted on page load with the `--action` background and `--surface` text colour. Click-to-select and Cmd-click-to-multi-select both worked correctly. The README's steering toward checkbox groups remains correct — the interaction is functional on desktop but discoverability is low without instruction.

**Keyboard navigation.** Tab moved focus through selects in source order. Arrow keys changed selection inline on focused selects without opening the panel. Pressing Enter on a focused select inside a form submitted the form, as expected and documented.

**Required + placeholder validation.** Submitting the contact form with the placeholder option selected triggered the UA validation message. The `aria-invalid="true"` state on the demo invalid select displayed the red border and the linked error message correctly.

**Placeholder-option behaviour.** After picking a real value from a select, the placeholder option remained selectable from the panel (since the demo placeholders are not `disabled`). The trade-off discussed in the open questions below remains — worth resolving before `form-field` lands.

---

## Findings that surfaced during testing

**Reference-page label classes were rendering in UA serif.**

During the closed-state and pre-selected-value tests, the `<label>` elements (`.field-label`) were rendering in a serif font despite the rest of the page rendering in Plex Sans. Cause: the reference-page style block set `font-size`, `font-weight`, and `color` on `.field-label`, `.field-help`, and `.field-error`, but not `font-family`. The label inherited from `<body>`, which inherited from the UA default.

Earlier reference pages (text-input, checkbox) avoided this by setting `font-family: var(--font-sans)` on every text-bearing class. The select reference page didn't.

**Resolution:** two-layer fix.

1. **In `prose.css`:** added a new Section 1 (PAGE DEFAULT) that sets `font-family: var(--font-sans)` on `<body>`. This means every descendant element — including form chrome (label, fieldset, legend) and input controls (input, select, textarea, button) — inherits the system's sans family by default. Components that need a different family (mono for tokenised values) set it explicitly on themselves.

2. **In `select.html`:** renamed `.field-label`, `.field-help`, `.field-error` to `.demo-label`, `.demo-help`, `.demo-error` to match the convention established in text-input and checkbox reference pages. Also added `font-family: var(--font-sans)` to each class for belt-and-braces — the prose body rule covers them now, but the per-class declaration matches the earlier pages' explicit style.

The `prose.css` amendment is the meaningful change. It closes a latent trap: any element prose doesn't explicitly style (label, fieldset, input, table cells, etc.) was previously falling back to the UA default font, which is serif on macOS and Windows. The body rule fixes the whole class of bug at the foundation.

Re-test after the amendment confirmed all three criteria pass. The receipts above reflect the post-amendment state.

---

## Controls-pass amendments (18 May 2026)

Three changes made during the controls-layer pass alongside the radio v0.1 work. All resolved in CSS.

**1. `[disabled]` → `:disabled` on per-element rules.**

Radio testing surfaced that the attribute selector misses the fieldset-disabled case — inside `<fieldset disabled>`, the browser marks descendants as `:disabled` but does NOT add the `disabled` attribute to the markup. The same issue applied to select: a `<select>` inside a `<fieldset disabled>` would have shown the wrong cursor and would have been hoverable.

Resolution: swapped the per-element rules to use `:disabled` (pseudo-class). The `fieldset[disabled]` selectors elsewhere are correct as-is. The existing reference-page examples don't exercise the fieldset-disabled case, so no visible regression to test against from the existing demos.

**2. Clear-on-valid-selection for invalid state.**

The most common required-select failure: user submits with the placeholder option still selected, then picks a real option. The red border stayed until JS removed `aria-invalid`. Same problem identified during radio testing.

Resolution: added rules that detect when a non-placeholder option has been chosen:
- `select[aria-invalid="true"]:has(option:checked:not([value=""]))` — the "user has picked something real" condition. Clears the red border by setting it back to `--border-strong`.
- A paired `:hover` rule re-instates the standard hover state for the now-valid select.
- A general-sibling combinator (`~`) rule hides the `.error-message` that follows the select within the same wrapper.

The CSS detects the satisfied state by looking inside the select for a checked option whose value isn't the empty placeholder.

**3. `.error-message` convention.**

The reference page's invalid demo error `<p>` now carries `class="demo-error error-message"` so the convention is exercised. The error must sit as a sibling of the `<select>` within a common wrapper (the existing `.field` div in the demo).

### Amendment receipts

**Disabled-via-fieldset behaviour:** verified by wrapping the existing locked-region select in `<fieldset disabled>` via Chrome dev tools. Cursor became `not-allowed` on both the select and the label; hover did not trigger the border colour shift. The previously-broken behaviour is now correct.

**Clear-on-valid-selection:** in the existing invalid demo, picking any non-placeholder option ("Email", "Signal", or "Phone call") cleared both the red border AND the error message immediately. Re-selecting the placeholder option (`"Choose one…"`) brought both back — the rule fires on the live `:has(option:checked:not([value=""]))` state, so the round-trip works in both directions as expected.

**Layout integrity on clear:** confirmed — `visibility: hidden` preserved the message's layout space. No vertical jump on selection or re-selection of the placeholder.

**Hover state on the now-valid select:** verified — picking a real option restored the default hover behaviour (border shifts to `--text` on hover). The invalid hover override no longer fires once `aria-invalid` has been "visually resolved" by the `:has()` rule.

---

## Open questions for v0.2 and beyond

- **Placeholder option `disabled` convention.** Should the placeholder option be `disabled` by default in documentation examples? Trade-off: prevents reverting to "nothing chosen" after a real pick (good for required fields), but blocks legitimate "actually I want to clear this" intents (bad for optional fields). May vary by field requiredness. Worth resolving before `form-field` lands so the pattern can codify it.
- **Per-option validation patterns.** The clear-on-valid-selection rule fires when ANY non-placeholder option is chosen. A fork validating that the chosen option is itself valid (e.g. "this option is no longer available") would need a different signal mechanism. Document if it becomes a real concern.
- **The general-sibling combinator dependency.** The `.error-message` hide rule uses `~`, which requires the error to live in the same parent as the `<select>`. This works with the current `.field` wrapping pattern. When `form-field` is built, this becomes part of its codified composition.
- **`--control-size-sm` token.** A small-size token for selects, inputs, and (potentially) other controls. Currently each control hardcodes its sizing; if dense-UI compositions surface a real need, consolidate then.
- **Custom listbox in `patterns/`.** When, if ever, to build. The barrier is high (JS, ARIA combobox, focus management) but real fork needs may justify it. Track demand.
- **Print rendering of `<select multiple>` selections.** Browsers vary in how they show selected options without their coloured background. May be a fork-specific concern; revisit if a fork reports trouble.
- **`<optgroup>` styling.** If a future CSS specification makes the open panel reachable, revisit. No such proposal is currently at a useful stage.
- **iOS / Android testing.** Deferred from this v0.1 receipt. The documented limitation framing covers the open-panel rendering question, but device-specific behaviour on touch (especially `<select multiple>`) is worth verifying when a fork deploys to mobile-first contexts.
- **The `.error-message` convention is now load-bearing across checkbox, radio, and select.** When `form-field` is built, this convention should be codified explicitly: error messages live inside the field's wrapping container (label/fieldset/wrapper div) and carry the `.error-message` class.
- Print preview check, as flagged for prior components — still optional, still worth adding to the standing checklist if a habit forms.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
