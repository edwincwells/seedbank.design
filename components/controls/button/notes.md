# button — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 12 May 2026.
**Focus-ring re-validated:** Chrome on macOS by Edwin, 7 June 2026 (see Focus-ring re-validation below).
**Disabled-selector migration:** button.css migrated `[disabled]` → `:disabled` on 10 June 2026; re-tested by Edwin (Chrome on macOS, 10 June 2026) — Pass (see Disabled-selector migration below).

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the six stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` in Chrome dev tools. With no CSS applied:

- Every `<button>` element rendered with the browser's default UA styling — visibly a button, distinct from surrounding text
- Button labels remained legible. Icons rendered as inline SVGs at their natural size
- The `disabled` attribute produced the UA-default greyed state — visibly disabled
- The `aria-busy="true"` buttons lost the spinner pseudo-element (expected) but the textual labels — "Saving…", "Loading", "Working", "Deleting…" — communicated the loading state on their own. This is the correct graceful-degradation behaviour: the label carries the meaning, the spinner is decoration
- The form example continued to work as a form. The `type="button"` and `type="submit"` attributes preserved their behaviour without any CSS involvement
- The wrong-use blocks degraded to plain paragraphs

The variant differentiation, the loading spinner, the size variations, the destructive friction model, and the flex layout (which produced run-together label text like "Primary actionSecondaryGhost") were all lost — expected, all CSS effects. Semantic interactivity of every button intact. Focus order intact.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome:

- All button variants remained readable and proportional
- The icon-and-label gap scaled correctly — icons did not visually merge with their labels
- The `btn-sm` size remained distinguishable from the default size — visibly smaller, but still legible and tap-friendly at the larger render
- The loading spinner remained proportional to the label and aligned correctly on the baseline
- The `btn-block` examples filled the demo region without overflow
- The form example reflowed cleanly — inputs and buttons stayed visibly grouped

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width:

- Variant rows wrapped onto multiple lines via flex-wrap without horizontal scroll
- Icon-and-label buttons held their labels on a single line where space allowed; wrapped gracefully when not
- The `btn-block` example filled the full available width
- The form example wrapped cleanly — input on one row, buttons on the next, no broken layout
- Wrong-use blocks held their content without horizontal scroll
- Inline `<code>` references inside paragraphs wrapped correctly

---

## Edge cases noted during testing

**Hover state across variants.** All four variants produced visible, distinct hover treatments. The destructive variant's outlined-to-filled transition was visibly the most distinctive — confirms the friction-model intent works as designed.

**Active state filter.** `filter: brightness(0.92)` produced visible feedback on every variant, including ghost (where the hover background dims slightly on press, giving the click a tactile feel even without a default fill).

**Focus ring visibility.** The focus ring was clearly visible on every variant. On the destructive variant, the blue focus outline sat outside the red border without visual clash — the `outline` (rather than `box-shadow`) approach in the focus-ring foundation gives the ring its own visual space. *(See the Focus-ring re-validation section below — this observation predated the foundation fix and has been re-confirmed.)*

**Disabled hover suppression.** Hovering over each disabled button produced no background change. The verbose per-variant disabled-hover overrides did their job. *(This observation covered directly-disabled buttons; the selectors have since migrated `[disabled]` → `:disabled` so the suppression also reaches fieldset-disabled buttons — see the Disabled-selector migration section below.)*

**Loading spinner alignment.** Spinner was vertically centred against the label text on every variant and size. The 1em sizing matched the label's effective height correctly via `inline-flex` alignment.

**Spinner under reduced motion.** Enabling `prefers-reduced-motion` in dev tools emulation paused the spinner mid-rotation. The partial ring remained visible — the loading state still communicates clearly, just statically.

**Form submission gotcha.** Clicking "Show password" fired the inline `onclick` alert WITHOUT submitting the form. Clicking "Sign in" fired the form's `onsubmit`. The form gotcha is correctly documented and the reference page demonstrates the correct pattern in action.

**Link-as-button keyboard behaviour.** Tab to the `<a class="btn">` and press Space — no activation, as expected. Press Enter — link follows. This is the documented difference between buttons and links and confirms why `<a class="btn">` should only be used for genuine navigation.

**Icon colour inheritance.** All icons rendered in the correct colour per variant via `currentColor`. Primary's icons were light against the dark fill; secondary, ghost, and default-state destructive icons inherited the variant's text colour; destructive on hover correctly switched icon colour with the rest of the button.

**Dark mode.** All variants rendered correctly in dark mode:
- Primary's text colour resolved to the light surface against the `--action` fill
- Secondary's border remained visible against the dark surface
- Destructive's red read correctly against the dark surface in both default and hover states
- Loading spinner used `currentColor` and inherited per-variant correctly

---

## Focus-ring re-validation (7 June 2026)

The focus-ring foundation referenced `--border-focus`, a token that was absent from the committed `tokens.css`, which silently invalidated the `outline` shorthand and removed the focus ring system-wide. This was caught during `methodology` testing and fixed in the foundation (`--border-focus: 2px` added to `tokens.css` §6; a `var(--border-focus, 2px)` fallback added to `focus-ring.css`). See `focus-ring/notes.md` for the full record.

The "Focus ring visibility" observation above predates that fix. Re-tested 7 June 2026 with the fix in place: the focus ring renders correctly under keyboard focus on every variant (primary, secondary, ghost, destructive) at both default and small sizes, sitting cleanly outside each variant's border including the destructive red. **Pass.**

---

## Disabled-selector migration ([disabled] → :disabled) (10 June 2026)

A docs-phase review of `button.css` found that button keyed its disabled styling off the `[disabled]` attribute selector, not the `:disabled` pseudo-class. Button was the one interactive control the radio-triggered `:disabled` pass (checkbox, select, radio) had missed. Per the system convention, a control inside `<fieldset disabled>` is marked `:disabled` by the browser but receives no `disabled` content attribute — so the attribute selector skipped it, and a fieldset-disabled button would be functionally disabled by the browser while still rendering as active, hover lift included.

**Change.** All five disabled selector blocks — the main disabled rule (§8) and the four per-variant hover-suppression rules — migrated from `[disabled]` to `:disabled`. `[aria-disabled="true"]` is retained on every rule for the `<a class="btn">` case: an anchor is not form-associated and cannot be natively disabled, so ARIA stays its mechanism. The `[aria-busy="true"]` loading state (§9) is unrelated and untouched. The header state summary and the §8 comment were updated to document the convention.

**Unchanged behaviour.** For a directly-disabled control (`<button disabled>`), `:disabled` matches exactly where `[disabled]` did, so the directly-disabled receipts above — UA disabled state with no CSS, disabled hover suppression, dark mode — remain valid as recorded.

**Re-tested by Edwin (Chrome on macOS, 10 June 2026; keyboard navigation enabled for focus checks per `testing-lowfi.md`) — all four Pass:**

- **Pass.** A `<button>` inside `<fieldset disabled>` renders the disabled treatment (opacity 0.5, `not-allowed` cursor) and suppresses the hover lift, across all four variants.
- **Pass.** An `<a class="btn">` inside `<fieldset disabled>` is correctly NOT affected by `:disabled` (anchors aren't form-associated); it reads disabled only once `aria-disabled="true"` is set.
- **Pass.** Regression check: the invalid markup `<a class="btn" disabled>` (anchors don't support `disabled`) is no longer styled disabled, and no reference-page demo relies on it.
- **Pass.** No regression to directly-disabled buttons or the loading state — `<button disabled>` and `<button aria-busy="true">` render as before.

**Verdict: Pass.** All four checks confirmed by Edwin on 10 June 2026 (Chrome on macOS).

---

## Open questions for v0.2 and beyond

- **`btn-block` with icon.** Not in the reference page; worth a test case in v0.2 when real content surfaces the pattern. The icon-and-label gap should hold even at full width, but uncertain how the centred alignment reads when one button is much taller than its label.
- **`btn-sm` with loading state.** The spinner at small size renders at the smaller 1em — works, but visually quite tight. Worth checking if forks find it readable in real dense UI.
- **Icon-only escape hatch.** README points to `aria-label` for the rare case where a label genuinely can't fit (toolbar close buttons). No class is provided. If forks consistently need this, worth considering whether to provide a documented `.btn-icon-only` variant in v0.2, even though the system rule discourages it.
- Print preview check, as flagged for prior components — still optional, still worth adding to the standing checklist once a habit is established.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
