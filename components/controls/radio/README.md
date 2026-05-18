# radio

A grouped set of mutually exclusive options. **Always grouped** — radios in isolation are nonsense and the system refuses them.

This is the fourth control. Visually it parallels checkbox closely (same sunken background, same thick border, same activity-coloured fill when selected), with one shape change (circle, not rounded square) and one implementation divergence (the inner dot is pure CSS, not an SVG data-URI).

---

## What it styles

- `<input type="radio">` — individual radio button
- `<fieldset>` + `<legend>` — the required composition pattern for a radio group

The wrapping `<label>` provides the accessible name for each individual radio and the click target for the whole label-and-circle area. The `<fieldset>` + `<legend>` provides the accessible name for the group as a whole. **Both layers are required.**

---

## The grouping rule

Radios are always in a `<fieldset>` with a `<legend>`. There is no "single radio" example in this component, and the README's wrong-use blocks refuse it explicitly.

The reason: without `<legend>`, screen readers announce each radio individually with no context — the user hears "Email, radio button, not checked" without knowing what question they're answering. The legend is the group's question; the radios are its options. Stripping one of the two breaks the accessible meaning.

This is stricter than checkbox, where ungrouped checkboxes ("I agree to the terms") are a legitimate pattern. Checkbox can stand alone because each checkbox is its own boolean question; radios only mean anything as a set of mutually exclusive options to a single question.

```html
<fieldset>
  <legend>Preferred contact method</legend>
  <label class="radio-row">
    <input type="radio" name="contact" value="email">
    <span>Email</span>
  </label>
  <label class="radio-row">
    <input type="radio" name="contact" value="signal">
    <span>Signal</span>
  </label>
</fieldset>
```

Every radio in a group **must** share the same `name` attribute. This is what tells the browser they're mutually exclusive — without a shared `name`, every radio is its own one-radio group and the exclusivity breaks. CSS doesn't enforce this; markup correctness does.

---

## Why a pure-CSS dot instead of an SVG (the divergence from checkbox)

Checkbox's tick mark is encoded as an SVG data-URI background-image because it's a path, not a shape — an angular line that genuinely needs SVG. That encoding came with a cost: the colour is baked into the URL, so checkbox has three encodings (light, dark, print) and forks that change `--surface-light` or `--text-light` have to re-encode each one.

Radio's "filled dot" is a circle inside a circle. That's trivially expressible in pure CSS with an absolutely-positioned `::before` pseudo-element. Going pure CSS means:

- The inner dot uses `var(--surface)` directly — no hardcoded hex.
- Light/dark mode switching is automatic via the token's resolution; no `@media (prefers-color-scheme: dark)` block needed for the dot.
- Print mode flips the dot to `var(--text)` in one line, no re-encoding.
- Radio doesn't add to the SVG-colour-hardcoding exception list documented for checkbox and select.

If your fork changes `--surface` or `--text`, the radio dot picks up the new value automatically. The pattern doesn't generalise to checkbox (a checkmark path won't fit a pseudo-element), but radio gets to skip the exception entirely.

---

## The defaults

| Property | Value | Token |
|---|---|---|
| Size | 18px × 18px | _(hardcoded — same as checkbox)_ |
| Inner dot | 8px × 8px | _(hardcoded — see note)_ |
| Border | 1.5px solid neutral-strong | `--border-thick`, `--border-strong` |
| Border radius | 50% (circle) | _(not a token — circles always 50%)_ |
| Background (unselected) | Warm cream (sunken) | `--surface-sunken` |
| Background (selected) | Activity colour | `--action` |
| Inner dot colour | Surface | `--surface` |

**On the 18px outer / 8px inner sizing.** The 18px outer matches checkbox so mixed forms read as a coherent set of controls. The 8px inner dot leaves ~3.5px of `--action` ring visible around it, which reads cleanly at the default size and remains visible at higher zoom levels. Both values are hardcoded for the same reason checkbox's size is hardcoded — they're specific to this control and there's no `--control-circle-size` token to reference.

**On focus.** Provided by the `focus-ring` foundation. Always include `focus-ring.css` in pages that use radios.

---

## States

| State | Trigger | Effect |
|---|---|---|
| Default | _(none)_ | Sunken background, neutral border |
| Hover | `:hover` | Border colour shifts to `--text` |
| Focus | `:focus-visible` | Focus ring from foundations/focus-ring |
| Selected | `:checked` | `--action` fill with `--surface` inner dot |
| Invalid | `aria-invalid="true"` | Border becomes `--error` |
| Disabled (individual) | `disabled` attribute | Opacity 0.5, cursor not-allowed, label dims via `:has()` |
| Disabled (whole group) | `disabled` on `<fieldset>` | Legend dims, all labels dim, radios become non-interactive |

All states are driven by HTML attributes — never by class names. Same convention as every other control.

---

## The native keyboard model — don't fight it

Within a radio group (radios sharing a `name` attribute), the browser handles keyboard navigation natively:

- **Tab** moves focus *past* the entire group to the next control.
- **Arrow keys** (↑ ↓ ← →) move focus *within* the group AND change selection, skipping disabled radios.
- **Space** selects the currently focused radio (rarely needed; arrows usually suffice).

This is correct behaviour. Some developers coming from custom React radios try to "fix" it by:

- Adding `tabindex="0"` to each radio — breaks the group navigation; now every radio is its own tab stop
- Adding JS to override arrow-key behaviour — fights what the browser already does correctly
- Treating the group like a list of focusable items — breaks the mental model users have for radio groups

**Don't do any of this.** The native model is correct. Trust it. Documented as a wrong-use block in the reference page.

---

## The no-deselect rule — optional groups need an explicit "None"

Once a radio in a group is selected, you can't unselect it. You can only switch to another radio in the same group. This is correct behaviour for "select exactly one" semantics, but it has a consequence:

**A user who clicks a radio by accident is stuck.** They can switch to a different option, but they can't return to "nothing selected."

For required groups (the user *must* choose), this is fine — the group's design assumes a choice will be made.

For optional groups (the user *may* leave the question blank), this is a problem. The fix is an explicit "None of these" or "Prefer not to say" or "No requirements" option:

```html
<fieldset>
  <legend>Dietary requirements for the next potluck</legend>
  <label><input type="radio" name="diet" value="vegetarian"> Vegetarian</label>
  <label><input type="radio" name="diet" value="vegan"> Vegan</label>
  <label><input type="radio" name="diet" value="gluten-free"> Gluten-free</label>
  <label><input type="radio" name="diet" value="none"> No requirements</label>
</fieldset>
```

The "No requirements" option provides the escape hatch. Without it, anyone who clicked "Vegan" by accident would have to either submit with the wrong answer or reload the page.

The system doesn't enforce this in CSS — it's a markup/content concern. The README's wrong-use block calls it out so authors don't miss it.

---

## Required radio groups

A required radio group means "at least one radio must be selected." The `required` attribute goes on **at least one radio in the group** (the browser checks the group as a whole), but conventional practice is to put it on every radio for explicitness:

```html
<fieldset>
  <legend>
    Are you attending the AGM? <span class="required" aria-hidden="true">*</span>
  </legend>
  <label><input type="radio" name="agm" value="yes" required> Yes</label>
  <label><input type="radio" name="agm" value="no" required> No</label>
</fieldset>
```

**The required asterisk goes in the `<legend>`, not on each radio label.** The question is the required thing — "you must answer this" — not each individual option. The asterisk on the legend reads correctly: a question with a required-marker followed by its options. An asterisk on each radio reads incorrectly: every option is required (impossible — only one can be selected).

---

## Invalid state — `aria-invalid` on every radio in the failing group

When a required radio group fails validation, apply `aria-invalid="true"` to **every radio in the group**, not just one:

```html
<fieldset aria-describedby="vote-error">
  <legend>Cast your vote <span class="required" aria-hidden="true">*</span></legend>
  <label><input type="radio" name="vote" value="for" required aria-invalid="true"> For</label>
  <label><input type="radio" name="vote" value="against" required aria-invalid="true"> Against</label>
  <label><input type="radio" name="vote" value="abstain" required aria-invalid="true"> Abstain</label>
</fieldset>
<p id="vote-error" class="error-message">You must cast a vote before the proposal can be recorded.</p>
```

The error message is linked from the `<fieldset>` via `aria-describedby` — the group as a whole owns the error context.

Why per-radio instead of group-wide? Several design systems put `aria-invalid` on a wrapping container, but `<fieldset>` doesn't reliably announce `aria-invalid` to assistive tech, and adding a non-native wrapper (`<div role="radiogroup">`) adds markup that the native `<fieldset>` already provides. Per-radio matches how every other control in the system handles invalid state (one attribute on one control), and the visual signal — red borders across the visually adjacent group — reads the group as a whole.

Same timing rule as everywhere else: only apply `aria-invalid` after a validation attempt, never on a pristine group.

### Clear-on-selected

The CSS clears both the red border AND the error message on the entire group once any radio inside has been selected. This is intentional: the "must select one" validation that flagged the group has been satisfied, and continuing to show error signals after the requirement is met is the system failing to keep up with the user's progress.

For the error message clearing to work, **the error `<p>` must live inside the `<fieldset>`** and carry the `.error-message` class:

```html
<fieldset aria-describedby="vote-error">
  <legend>Cast your vote <span class="required" aria-hidden="true">*</span></legend>
  <label><input type="radio" name="vote" value="for" required aria-invalid="true"> For</label>
  <label><input type="radio" name="vote" value="against" required aria-invalid="true"> Against</label>
  <p id="vote-error" class="error-message">You must cast a vote before the proposal can be recorded.</p>
</fieldset>
```

The CSS uses `visibility: hidden` (not `display: none`) so the message keeps its space in the layout. Selecting a radio fades the error away without causing the form to jump vertically.

The `aria-invalid="true"` attribute is still on every radio in the markup — only JS can remove it. The form itself should remove `aria-invalid` and update the error message content on its next validation cycle to bring the markup into agreement with the visual. But from the user's perspective, the moment they answer the question, the form stops complaining — which is the experience that matters.

This works for the common "at least one selected" validation pattern. If your form has per-option validation beyond that — e.g. "the selected option is no longer available" — the clear-on-selected behaviour will quietly suppress the red signal that should be re-applied. In that case, use a different signal (an inline error message near the option, a class on the specific radio) or override the rule in your fork's stylesheet.

Browser support for `:has()` is ~94% globally. Older browsers fall back to "red stays after selection" — annoying but not broken; the form itself can still clear `aria-invalid` and the message text on re-validation.

---

## Disabled — individual and group-level

Two paths into disabled, both supported natively by HTML:

**Individual radio disabled:**

```html
<label><input type="radio" name="pickup" value="cafe" disabled> Café (closed for refurbishment)</label>
```

The radio dims, the wrapping label dims via `:has()`. The rest of the group remains active.

**Whole group disabled:**

```html
<fieldset disabled>
  <legend>Membership tier (set by admins)</legend>
  <label><input type="radio" name="tier" value="member" checked> Member</label>
  <label><input type="radio" name="tier" value="steward"> Steward</label>
</fieldset>
```

The browser disables every descendant control automatically. The CSS dims the legend and all labels so the group-wide locked state is visible. The previously-selected radio stays visually selected (its `:checked` state is still true) — important for read-only contexts where you want to display "this is the current value, but you can't change it."

The CSS handles a subtle stacking issue: without intervention, the per-input opacity would multiply with the per-label opacity producing ~0.25 effective opacity. A reset rule keeps the dimming at 0.5 across the whole disabled group.

---

## Tokens used

| Token | Used for |
|---|---|
| `--surface-sunken` | Unselected background |
| `--action` | Selected fill |
| `--surface` | Selected inner dot |
| `--border-strong`, `--text` | Default and hover border colour |
| `--border-thick` | Border width (1.5px) |
| `--error` | Invalid border |
| `--text` | Print-mode dot colour |

No SVG-encoded hex values. No exception-list entries. Two hardcoded sizing values (18px outer, 8px inner) — same trade-off as checkbox's 18px size, tracked in the docs backlog.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/focus-ring/focus-ring.css`
3. Include `components/controls/radio/radio.css`
4. Write semantic HTML — `<fieldset>` with `<legend>` and `<label>`-wrapped `<input type="radio">` elements sharing a `name` attribute

---

## How to override

**To change the selected-state fill colour:**

```css
input[type="radio"]:checked {
  background-color: var(--share);
  border-color: var(--share);
}
```

The inner dot stays `--surface`, which works against any activity-token fill.

**To change the size of the radios:**

```css
input[type="radio"] {
  width: 22px;
  height: 22px;
}
input[type="radio"]:checked::before {
  width: 10px;
  height: 10px;
}
```

The dot's centring uses `transform: translate(-50%, -50%)`, so it stays centred at any size. Adjust the dot size proportionally.

**To use `accent-color` instead of the custom circle** (a fork that prefers UA controls):

```css
input[type="radio"] {
  appearance: auto;
  -webkit-appearance: auto;
  -moz-appearance: auto;
  background: none;
  border: none;
  width: auto;
  height: auto;
  accent-color: var(--action);
}
input[type="radio"]:checked::before {
  display: none;
}
```

This is a significant fork decision — the custom circle is the system's deliberate visual language. Document the reasoning.

---

## Print rules

- Background removed (saves ink)
- Border kept in `--text` colour
- Selected radios: outer fill drops to transparent, inner dot flips to `--text` so it remains visible against the now-transparent outer circle
- The user can see at a glance on the printed page which option was selected — important for forms that get printed as records

No SVG re-encoding required for any mode. The pure-CSS approach lets the print rules be a few lines, not a parallel data-URI library.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
