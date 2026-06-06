# form-field

A **composition pattern**, not a component. The structure that ties a label, an input, optional help text, and an optional error message into one coherent field — and the conventions that govern how they relate.

No new CSS ships with this pattern. form-field consolidates conventions already established in the controls layer (text-input, checkbox, select, radio) into one place, so there's a single answer to "how do I build a form in seedbank?" rather than five READMEs to cross-reference.

---

## What a field is

A field is one question and the means to answer it. In markup, that's up to four parts:

1. **Label** — the question. Always present.
2. **Control** — the input, select, checkbox, radio group. Always present.
3. **Help text** — optional clarification, format hints, examples.
4. **Error message** — optional, appears only after a failed validation attempt.

The form-field pattern is the structure that holds these four together with the right associations for assistive tech, the right visual order, and the right behaviour when validation fails.

---

## The canonical structure

The default field — a labelled text input with help text and a slot for an error:

```html
<div class="field">
  <label for="email">Email address</label>
  <p class="field-help" id="email-help">We'll only use this for community notices.</p>
  <input type="email" id="email" aria-describedby="email-help email-error">
  <p class="error-message" id="email-error">Enter a valid email address.</p>
</div>
```

The pieces and why they're ordered this way:

- **Label first.** The question comes before the answer. `for="email"` ties it to the input's `id`.
- **Help text before the input.** Format hints and clarifications belong before the user starts typing, not after. Placing help below the input means the user reads it only after they've already attempted an answer.
- **Input.** `aria-describedby` references both the help text and the error message IDs (space-separated). Screen readers announce both when the input is focused. The error ID can be referenced even before the error is shown — it just describes nothing until the error message has content.
- **Error message last**, inside the field wrapper, carrying `.error-message`.

`.field` and `.field-help` are your fork's classes to define; `.error-message` is a **system convention** (see below).

---

## The wrapping-label alternative

text-input, checkbox, and radio support a wrapping `<label>` pattern as an alternative to `<label for="…">`. The label wraps both the text and the control, providing the click target without needing matched `id`/`for` attributes:

```html
<label class="checkbox-row">
  <input type="checkbox">
  <span>Subscribe to the monthly digest</span>
</label>
```

Use the wrapping label when:

- The label text is short and sits naturally beside the control (checkboxes, radios)
- You'd rather not manage `id`/`for` pairs

Use explicit `<label for="…">` when:

- The field has help text and an error message to associate via `aria-describedby` (the wrapping pattern gets awkward with multiple associated elements)
- The control is a text input or select where the label sits above, not beside

Both are valid. The control READMEs document which each control uses by default; text-input and select lean explicit-label, checkbox and radio lean wrapping-label.

---

## The `.error-message` convention

**This is a load-bearing system convention, not a fork choice.** Codified across checkbox, radio, and select, and consolidated here.

An error message element carries `class="error-message"` and lives **inside the field's wrapping container**:

- For a control with a `.field` wrapper div: as a sibling of the input, inside the `.field`
- For a control with a wrapping `<label>`: as a sibling of the `<label>`, inside the field wrapper
- For grouped controls: inside the `<fieldset>`

```html
<div class="field">
  <label for="postcode">Postcode</label>
  <input type="text" id="postcode" aria-invalid="true" aria-describedby="postcode-error">
  <p class="error-message" id="postcode-error">Enter a valid UK postcode.</p>
</div>
```

The placement matters because the CSS uses `:has()` to detect when the requirement is satisfied and clear the message — that detection relies on the message being a sibling of the control inside a shared container.

### Clear-on-satisfied

When the user satisfies the requirement that caused the error, the CSS clears both the red border and the error message immediately — the form stops nagging the moment the requirement is met, without waiting for re-submission.

The mechanism:

- CSS uses `:has()` to detect satisfaction (a checkbox becoming `:checked`, a select gaining a valid selection, a radio in the group becoming `:checked`)
- The error message is hidden via `visibility: hidden`, **not** `display: none` — this preserves the layout space so the form doesn't jump when the message clears
- The `aria-invalid="true"` attribute stays on the input in markup — only JS can remove it — but the visual stops nagging

This applies to:

- **checkbox** — clear-on-checked
- **radio** — clear-on-selected
- **select** — clear-on-valid-selection (an option with a non-empty value chosen)

**Caveat:** forms with per-option validation beyond "must select / must check" need a different signal mechanism, because the clear rule will suppress red that should still be present. If your field's validity depends on *which* option, not just *that* an option was picked, you can't rely on clear-on-satisfied — wire the clearing in JS instead.

**Browser support:** `:has()` is ~94% globally. Older browsers fall back to "red stays after the action until re-validation" — annoying but not broken; the form's own JS can still clear `aria-invalid` and the message text on re-validation.

---

## The `aria-invalid` timing rule

**Never apply `aria-invalid="true"` on a pristine field.** It's applied only *after* a validation attempt — a submit, or a blur-validation if your fork does that.

Wrong:

```html
<!-- Don't do this — the field hasn't been validated yet -->
<input type="email" required aria-invalid="true">
```

Right: the field starts clean, and `aria-invalid="true"` is added by your validation code after the user has had a chance to interact and the validation has run.

This rule prevents the form from showing red before the user has done anything wrong — a common, hostile pattern where every required field is pre-marked invalid on page load. A field is innocent until validated.

The control READMEs document this per-control; it's the same rule everywhere. The CSS reads `aria-invalid="true"` to apply the error border; the timing of when that attribute appears is the form's responsibility.

---

## Required fields

Mark required fields with the `required` attribute on the control AND a visible asterisk:

```html
<label for="name">
  Your name <span class="required" aria-hidden="true">*</span>
</label>
<input type="text" id="name" required>
```

The asterisk markup convention: `<span class="required" aria-hidden="true">*</span>`.

- `aria-hidden="true"` because the `required` attribute already conveys the requirement to assistive tech — the visual asterisk would otherwise be announced redundantly ("asterisk").
- The `required` attribute is the machine-readable signal; the asterisk is the human-visible one. Both are needed — neither alone covers both audiences.

**For radio and checkbox groups, the asterisk goes in the `<legend>`**, not on each option — the group's question is the required thing, not each individual radio:

```html
<fieldset>
  <legend>How would you like to be contacted? <span class="required" aria-hidden="true">*</span></legend>
  <label><input type="radio" name="contact" value="email" required> Email</label>
  <label><input type="radio" name="contact" value="signal"> Signal</label>
</fieldset>
```

**Note on asterisk styling:** the control reference pages currently use either inline `style="color: var(--error)"` (checkbox, radio) or a `.required` class (select). This inconsistency should be resolved before this pattern is treated as canonical — pick one. The `.required` class is the better choice (no inline styles, fork can restyle), and this pattern documents the `.required` class form above. Forks should colour it via `.required { color: var(--error); }` in their stylesheet. _(Flagged as an open consistency item — see project docs backlog item 8.)_

---

## Grouped controls — fieldset and legend

For controls that share a single question — a set of radios (pick one), a set of checkboxes (pick any) — wrap them in `<fieldset>` with a `<legend>`:

```html
<fieldset>
  <legend>How would you like to be contacted?</legend>
  <label><input type="checkbox" name="contact" value="email"> Email</label>
  <label><input type="checkbox" name="contact" value="signal"> Signal</label>
  <label><input type="checkbox" name="contact" value="phone"> Phone call</label>
</fieldset>
```

`<fieldset>` and `<legend>` give the group an accessible name — screen readers announce "How would you like to be contacted?" as the group's label when the user enters it. Without the fieldset, each checkbox is an orphan with no shared context.

For a grouped field's error message, the `.error-message` goes **inside the fieldset**:

```html
<fieldset>
  <legend>How would you like to be contacted? <span class="required" aria-hidden="true">*</span></legend>
  <label><input type="checkbox" name="contact" value="email" required> Email</label>
  <label><input type="checkbox" name="contact" value="signal"> Signal</label>
  <p class="error-message" id="contact-error">Choose at least one way to be contacted.</p>
</fieldset>
```

The UA renders `<fieldset>` with a default border. The controls don't override it — styling the fieldset border is this pattern's concern, and most forks will either reset it or style it deliberately. The control reference pages reset it via demo CSS to show the unchromed grouping; real usage decides per-fork.

---

## Help text association

Help text is associated with its control via `aria-describedby`:

```html
<label for="postcode">Postcode</label>
<p class="field-help" id="postcode-help">UK format, e.g. BH1 2AB.</p>
<input type="text" id="postcode" aria-describedby="postcode-help postcode-error">
<p class="error-message" id="postcode-error">Enter a valid UK postcode.</p>
```

`aria-describedby` takes a space-separated list of IDs. Reference both the help text and the error message — the screen reader announces whichever has content. The error ID can be listed even when the error is empty; it describes nothing until populated.

---

## Visual order vs. DOM order

The field's visual order is: label, help, control, error. This is also the DOM order, which keeps the accessible reading order correct — no CSS reordering that would desync what sighted users see from what screen reader users hear.

The one tension: help text before the control reads naturally top-to-bottom, but `aria-describedby` associations don't depend on DOM position, so the help could technically go anywhere. Keep it before the control in the DOM — it's the order that works for both audiences without reordering tricks.

---

## Composition — which control for which field

| Field need | Control | Reference |
|---|---|---|
| Single-line text, email, number, etc. | `<input>` | `controls/text-input` |
| One choice from many (5+) | `<select>` | `controls/select` |
| One choice from few (2–4) | radio group | `controls/radio` |
| Any number of independent toggles | checkbox(es) | `controls/checkbox` |
| A single yes/no | one checkbox | `controls/checkbox` |
| An action | `<button>` | `controls/button` |

Each control's README documents its own specifics — text-input's `type="number"` guidance, checkbox's parent-child sync, select's placeholder-option convention, radio's group requirements. This pattern documents the field structure that wraps any of them; the controls document themselves.

---

## What this pattern ships

No CSS. `.field`, `.field-help`, and the layout that positions label/help/control/error are the fork's to realise — illustrative starting point:

```css
.field {
  margin-bottom: var(--space-5);
}
.field > label {
  display: block;
  font-size: var(--text-sm);
  font-weight: var(--weight-medium);
  margin-bottom: var(--space-1);
}
.field-help {
  font-size: var(--text-sm);
  color: var(--text-muted);
  margin: 0 0 var(--space-2);
}
.error-message {
  font-size: var(--text-sm);
  color: var(--error);
  margin: var(--space-1) 0 0;
}
/* Clear the message when the field's requirement is satisfied.
   The specific :has() selector depends on the control — see the
   control READMEs for checkbox, radio, select. */
```

As with cards, this is structure, not an API. The one fixed point is the `.error-message` class name and its placement — that's a system convention the control CSS depends on. Everything else is the fork's.

---

## Accessibility checklist for a field

- Label present and associated (`for`/`id` or wrapping `<label>`)
- Required fields carry both `required` and a visible `.required` asterisk
- Help text associated via `aria-describedby`
- Error message carries `.error-message`, lives inside the field container, associated via `aria-describedby`
- `aria-invalid="true"` applied only after a validation attempt, never on a pristine field
- Grouped controls wrapped in `<fieldset>` with `<legend>`; asterisk in the legend
- Visual order matches DOM order (label, help, control, error)
- Focus styling inherited from `foundations/focus-ring` — included in the page

---

## See also

- `controls/text-input/README.md` — single-line inputs, `type="number"` guidance
- `controls/checkbox/README.md` — checkboxes, indeterminate, parent-child sync
- `controls/select/README.md` — selects, placeholder-option convention
- `controls/radio/README.md` — radio groups, fieldset requirement
- `controls/button/README.md` — submit buttons, the form-submission gotcha
- `foundations/focus-ring/README.md` — focus styling for all controls
- BUILD_CONVENTIONS.md — the `.error-message` convention, `:disabled` over `[disabled]`, clear-on-satisfied, state-from-attributes
