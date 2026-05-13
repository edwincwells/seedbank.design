# text-input — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 12 May 2026.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the seven stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` in Chrome dev tools. With no CSS applied:

- Every input remained visibly an input — the browser's default UA chrome (inset border, white background) took over
- Labels remained visible and associated with their inputs via the wrapping `<label>` pattern — clicking a label still focused its input
- Placeholders remained visible at the UA default colour
- The `disabled` attribute produced the UA-default greyed, non-interactive input
- The `readonly` attribute produced a non-editable but still focusable input — selectable and copyable as expected
- The `aria-invalid="true"` input remained typeable; the red border was lost (expected — CSS effect) but the linked error message paragraph remained visible and accessibly associated via `aria-describedby`
- The `<textarea>` resized in both axes per UA default
- The form structure and wrong-use blocks degraded to plain document flow

The sunken background, matched-with-button height, custom placeholder colour, invalid border, suppressed UA spinners on search and number inputs, and the textarea vertical-only resize constraint were all lost — expected. Semantic structure of every input intact. Tab order intact. Form-submission behaviour preserved.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome:

- All input types remained readable and proportional
- The sunken background remained visibly distinct from the page surface
- Placeholders remained legible at the larger render — the `--text-subtle` colour held up against the sunken background
- The inline button-and-input example maintained visual alignment — both elements scaled together, heights stayed matched
- The invalid border remained clearly visible without becoming overwhelming
- The textarea remained usable; the vertical resize handle remained visible and draggable
- Touch targets became more comfortably tappable at the zoomed size

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width:

- All inputs spanned the full container width via `width: 100%`
- No horizontal scroll appeared on any input or demo region
- The inline button-and-input example fit on a single row, with the input flexing to fill the available space and the button at fixed width — alignment held
- The textarea fit without overflowing; the resize handle remained accessible
- The error message text under the invalid input wrapped correctly within the 320px viewport
- Wrong-use blocks held their content without horizontal scroll
- Inline `<code>` references in paragraphs wrapped correctly

---

## Edge cases noted during testing

**Focus ring on the sunken background.** Focus ring was clearly visible against `--surface-sunken`. The `outline`-based approach (sitting outside the input's box) gave the ring its own visual space without competing with the input's border.

**Focus ring on an invalid input.** Tab to the `aria-invalid="true"` input — both the red border and the blue focus ring rendered simultaneously without visual clash. The ring sat outside the red border cleanly, so each signal stayed legible.

**Placeholder contrast.** `--text-subtle` against `--surface-sunken` held legibility in both light and dark modes. Worth re-checking with a formal contrast tool if a fork's brand colours change the relationship, but the default palette is solid.

**Disabled vs readonly visual distinction.** The two states read as semantically different at a glance — disabled is muted via opacity, readonly has the distinctly lighter `--surface` background. A user can tell which they're looking at without context.

**Search input WebKit clear button.** Typed into the search input — no UA "x" appeared on Chrome. The `::-webkit-search-cancel-button` suppression worked.

**Number input spinners.** Clicked into the number input and hovered — no up/down arrows appeared. The `::-webkit-inner-spin-button` rule did its job in Chrome; the `-moz-appearance: textfield` rule is documented in CSS for Firefox parity.

**Textarea resize.** Dragged the textarea's resize handle — vertical resize worked, horizontal did not. The `resize: vertical` rule held.

**Email validation in browser.** Typed an invalid value into the email input and attempted submit — browser's built-in validation fired correctly. The component doesn't interfere with native validation.

**Inline `<label>` association.** Clicked each `demo-label` and focus moved to the associated input. The wrapping-`<label>` pattern provides the association without needing `for`/`id` pairs.

**Form submission with disabled vs readonly.** Verified in a quick form test: disabled fields are excluded from submission, readonly fields are included. The semantic distinction holds end-to-end.

**Dark mode.** All states rendered correctly in dark mode:
- Sunken background remained visibly distinct from the page surface in dark mode
- Placeholder contrast held against the dark sunken surface
- Invalid red border read clearly against the dark surface
- Readonly's lighter background (now `--surface-dark` against `--surface-sunken-dark`) inverted the relationship correctly — still reads as "this is set, not editable"

---

## Open questions for v0.2 and beyond

- **`<textarea>` with very long initial content.** Not in the reference page. The 80px min-height handles empty/short cases well; uncertain how the vertical-resize handle behaves when a textarea is auto-sized to a long initial value. Worth a real-content test in v0.2.
- **Inputs with prefix/suffix affordances** — currency symbols inside the input, search icons, password-visibility toggles. These are out of scope for this component but will come up in real form-field patterns. Worth deciding whether they belong in `form-field` or warrant their own input-group component.
- **`aria-invalid="true"` timing in real forms.** The README states "after validation, not on pristine fields" — but `form-field` will need to either enforce this with documentation or provide a JS helper. Worth resolving when `form-field` is written.
- Print preview check, as flagged for prior components — still optional, still worth adding to the standing checklist.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
