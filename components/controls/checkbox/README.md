# checkbox

A custom-styled checkbox that replaces the browser default entirely. The control is still a native `<input type="checkbox">` — accessibility, keyboard behaviour, form submission, and assistive tech announcements all work as expected. Only the visual chrome is re-skinned, via `appearance: none` and a CSS-only checkmark.

---

## What it styles

- `<input type="checkbox">` — the only selector

The wrapping `<label>` provides the full-label click target (same pattern as text-input). For groups of checkboxes, use `<fieldset>` with `<legend>` — that's a composition pattern documented below.

---

## The defaults

| Property | Value | Token |
|---|---|---|
| Size | 18px × 18px | _(hardcoded — see note below)_ |
| Radius | 4px | `--radius-sm` |
| Border | 1.5px solid neutral-strong | `--border-thick`, `--border-strong` |
| Background (unchecked) | Warm cream (sunken) | `--surface-sunken` |
| Background (checked) | Activity colour | `--action` |
| Checkmark | SVG, light colour | _(see SVG note below)_ |

**On the 18px size.** The UA default is around 13px, which is hard to hit on touch devices. 18px is slightly larger than UA but still compact enough to sit cleanly with text on a single baseline. This is one of the few hardcoded values in the system — there's no token for "checkbox size" because the value is specific to this one control. If you want a smaller or larger checkbox, override `width` and `height` directly.

**On the sunken background.** Matches text-input. The recess signals "this is interactive" against the page surface.

**On focus.** Provided by the `focus-ring` foundation. Always include `focus-ring.css` in pages that use checkboxes.

---

## Why a custom box instead of the native checkbox?

The native `<input type="checkbox">` renders differently on every platform — macOS, Windows, Linux, iOS, Android — and across browsers within each platform. A native checkbox next to a system-styled button reads as visually inconsistent.

Two alternatives were considered:

1. **`accent-color: var(--action)`.** This CSS property tints the native checkbox to match the system. It's lightweight, accessible, and supported in all current browsers. But it only changes the checked-state colour — the geometry, the border, the size, and the unchecked appearance all stay UA-default. The inconsistency between unchecked and styled remains.

2. **Native UA appearance, no styling.** Maximum accessibility safety, zero implementation surface. But the visual inconsistency between platforms is jarring against the rest of the system's deliberate visual language.

The custom box trades a small amount of implementation surface for visual consistency across platforms. The control remains a real `<input type="checkbox">` — `appearance: none` only resets the visual chrome; everything else (keyboard activation with Space, form submission, screen reader announcements as "checkbox checked / unchecked") works natively.

---

## A note on the SVG checkmark colour

The checkmark is an inline SVG encoded as a data-URI background-image:

```css
background-image: url("data:image/svg+xml,...stroke='%23fafaf7'...");
```

The colour `#fafaf7` is `--surface-light` URL-encoded. **This is the one place in the component where a token value is hardcoded rather than referenced.** CSS variables can't be used inside a data-URI — the URL is parsed before custom properties are resolved.

If your fork changes `--surface-light` to a different colour, you should also re-encode the SVG with that colour. The README in `tokens.css` flags this as a documented exception to the "tokens only" rule.

The print-mode rule (Section 6 in the CSS) re-encodes the SVG with `#1a1a1a` (dark) so the checkmark prints against a now-transparent background. Same caveat applies if you change `--text-light`.

---

## States

| State | Trigger | Effect |
|---|---|---|
| Default | _(none)_ | Sunken background, neutral border |
| Hover | `:hover` | Border colour shifts to `--text` |
| Focus | `:focus-visible` | Focus ring from foundations/focus-ring |
| Checked | `:checked` | `--action` fill with light SVG checkmark |
| Indeterminate | `.indeterminate = true` (JS only) | `--action` fill with horizontal dash |
| Invalid | `aria-invalid="true"` | Border becomes `--error` |
| Disabled | `disabled` attribute | Opacity 0.5, `cursor: not-allowed`, label also dimmed |

All states are driven by HTML attributes (or, for indeterminate, a JS property) — never by classes like `.checkbox-checked`. Project convention: state from attributes, not class names.

---

## The indeterminate state

Indeterminate is "neither checked nor unchecked" — the state of a parent checkbox when some but not all children are selected. There's no HTML attribute for it; it can only be set via JS:

```javascript
myCheckbox.indeterminate = true;
```

The CSS handles the visual: a horizontal dash on a filled `--action` background. The same value reads as "partial selection" without needing to look like either checked or unchecked.

See the parent-child pattern below for typical usage.

---

## The parent-child pattern

A common pattern: a parent checkbox toggles a group of children. The parent's state reflects the children:

- All children checked → parent is checked
- Some children checked → parent is indeterminate
- No children checked → parent is unchecked

The markup is plain — checkboxes nested in indented containers. The CSS handles the visuals. The state synchronisation needs JS, but the JS is small:

```javascript
const parent = document.getElementById('parent');
const children = document.querySelectorAll('.child');

function syncParent() {
  const checked = Array.from(children).filter(c => c.checked).length;
  if (checked === 0) {
    parent.checked = false;
    parent.indeterminate = false;
  } else if (checked === children.length) {
    parent.checked = true;
    parent.indeterminate = false;
  } else {
    parent.checked = false;
    parent.indeterminate = true;
  }
}

function syncChildren() {
  children.forEach(c => c.checked = parent.checked);
  parent.indeterminate = false;
}

children.forEach(c => c.addEventListener('change', syncParent));
parent.addEventListener('change', syncChildren);
syncParent();  // set initial state
```

Wire this up however your stack handles event listeners. The component CSS makes no JS assumptions — wire it in framework code, vanilla JS, or whatever fits your fork.

---

## Groups — `<fieldset>` and `<legend>`

For a group of checkboxes that share a question ("How would you like to be contacted?"), wrap them in a `<fieldset>` with a `<legend>`:

```html
<fieldset>
  <legend>How would you like to be contacted?</legend>
  <label><input type="checkbox" name="contact" value="email"> Email</label>
  <label><input type="checkbox" name="contact" value="signal"> Signal</label>
  <label><input type="checkbox" name="contact" value="phone"> Phone call</label>
</fieldset>
```

`<fieldset>` and `<legend>` provide the group's accessible name to screen readers. UAs render the `<fieldset>` with a default border, which the component does not override — that's a composition concern for `form-field` later. For the reference page, the demo CSS resets the fieldset border to show the unchromed pattern, but real usage may want the fieldset border to remain or be styled separately.

---

## The wrapping-`<label>` pattern

Same approach as text-input: wrap each checkbox in a `<label>`, with the input and the text both as children:

```html
<label class="checkbox-row">
  <input type="checkbox">
  <span>Subscribe to the monthly digest</span>
</label>
```

The label provides:

- Click target for the whole label (not just the box)
- Accessible name for the checkbox (no `aria-label` needed)
- The text-and-box layout (with `display: flex; gap: var(--space-2);` on the label)

The `.checkbox-row` class in the reference page is a placeholder for what `form-field` will provide. The pattern is layout-only — once `form-field` lands, this becomes its convention.

---

## Disabled state

When the `disabled` attribute is set on the input, both the input AND the wrapping label visually dim. This works via the `:has()` pseudo-class:

```css
label:has(input[type="checkbox"][disabled]) {
  opacity: 0.5;
  cursor: not-allowed;
}
```

`:has()` is supported in all current browsers (~94% global). On older browsers, the input dims correctly but the label text stays at full opacity — a mild visual inconsistency, acceptable graceful degradation.

---

## Invalid state — timing matters

Same rule as text-input: `aria-invalid="true"` is applied **after** a validation attempt, never on a pristine field.

The most common use case for an invalid checkbox is a required "I agree to the terms" checkbox that gets submitted unchecked. The form's submit handler should add `aria-invalid="true"` to the checkbox and link an error message via `aria-describedby`:

```html
<label class="checkbox-row">
  <input type="checkbox" required aria-invalid="true" aria-describedby="terms-error">
  <span>I agree to the community guidelines *</span>
</label>
<p id="terms-error" class="error-message">You must agree before joining.</p>
```

Don't apply `aria-invalid="true"` on page load. Don't apply it to the entire form's required fields preemptively. Apply it only after the user has had a chance to interact and validation has been attempted.

---

## Tokens used

| Token | Used for |
|---|---|
| `--surface-sunken` | Unchecked background |
| `--action` | Checked and indeterminate fill |
| `--border-strong`, `--border` | Default border; hover transitions |
| `--border-thick` | Border width (1.5px) |
| `--error` | Invalid border |
| `--radius-sm` | Corner radius |
| `--text` | Hover border colour |

Two values are hardcoded as documented exceptions:

- The 18px box size (no checkbox-size token exists)
- The `#fafaf7` and `#1a1a1a` colours in the SVG data-URIs (CSS variables can't be used inside a URL)

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/focus-ring/focus-ring.css`
3. Include `components/controls/checkbox/checkbox.css`
4. Write semantic HTML — `<input type="checkbox">` inside a wrapping `<label>`, with a `<span>` for the label text

---

## How to override

**To change the checked-state fill colour:**

```css
input[type="checkbox"]:checked {
  background-color: var(--share);  /* or any token */
  border-color: var(--share);
}
```

**To use `accent-color` instead of the custom box** (a fork that prefers UA controls):

```css
input[type="checkbox"] {
  appearance: auto;
  -webkit-appearance: auto;
  -moz-appearance: auto;
  /* override all the custom properties */
  background: none;
  border: none;
  width: auto;
  height: auto;
  accent-color: var(--action);
}
```

This is a significant fork decision — the custom box is the system's deliberate visual language. Document the reasoning.

**To change the box size:**

```css
input[type="checkbox"] {
  width: 22px;
  height: 22px;
}
```

The SVG checkmark scales with the box size automatically.

---

## Print rules

- Backgrounds removed (saves ink)
- Borders kept in `--text` colour
- Checkmark SVG re-encoded in dark colour so it reads against the transparent print background
- Indeterminate dash also re-encoded for print

This means the printed page accurately shows what was checked and what wasn't — important for forms that get printed as records.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
