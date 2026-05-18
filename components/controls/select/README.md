# select

A native `<select>` control with the UA chrome stripped and re-skinned to match the system's form vocabulary. Single-select and multi-select are both styled, with limits.

This is the third control in the system. It matches text-input visually in its closed state and adds a single custom indicator — a chevron-down arrow — to signal "this opens a panel".

---

## What it styles

- `<select>` — single-select dropdown
- `<select multiple>` — list-box multi-select, styled minimally; see the steering below

The wrapping `<label>` provides the accessible name and the click-target pattern (same as text-input and checkbox). For groups of selects (e.g. a row of related dropdowns), there's no equivalent of `<fieldset>` — each `<select>` is its own labelled control.

---

## The honest limitation, up front

`appearance: none` strips the **closed-state chrome** — the box, the bevel, the native arrow. It does **not** style the open dropdown panel. That panel is rendered by the operating system and is unreachable from CSS in every current browser.

What this means in practice:

- The closed `<select>` looks like the rest of the system's form controls.
- When the user clicks to open it, the panel that drops down (or up, or modally on mobile) is whatever macOS, Windows, iOS, or Android draws.
- The open panel's typeface, padding, hover state, and selected-state appearance are not the system's. They are the OS's.

This is a deliberate trade-off. The alternative — a fully custom listbox — requires JS, complex ARIA combobox patterns, and breaks the no-JS-in-component-CSS rule. GOV.UK, USWDS, and Shopify Polaris all make the same call: style the closed state, let the OS draw the open one. The system is honest about this rather than pretending the panel matches.

If your fork genuinely needs a styled open panel (searchable lists, virtualised long lists, multi-tag inputs), build a custom listbox in `patterns/`. It's a composition concern, not a control.

---

## The defaults

| Property | Value | Token |
|---|---|---|
| Background | Warm cream (sunken) | `--surface-sunken` |
| Border | 1.5px solid neutral-strong | `--border-thick`, `--border-strong` |
| Radius | 8px | `--radius-md` |
| Padding | 8px 32px 8px 12px | `--space-2`, `--space-6`, `--space-3` |
| Arrow | Chevron-down SVG, 12px | _(hardcoded — see below)_ |
| Font | Plex Sans, regular weight, 14px | `--font-sans`, `--weight-regular`, `--text-base` |
| Text colour | Default | `--text` |

**Closed state matches text-input exactly.** The arrow is the only visual difference. Forms read as a coherent set of controls; the arrow is the signal that this particular control opens a panel rather than accepting typed input.

**No small variant in v0.1.** Consistency with text-input (which also has no small variant) wins over matching button's `btn-sm`. If real composition needs surface a mismatch — a table row with `btn-sm` next to a default-sized select — we'll revisit. Tracked in the docs backlog.

---

## A note on the hardcoded arrow colour

The arrow is an inline SVG encoded as a data-URI background-image:

```css
background-image: url("data:image/svg+xml,...stroke='%231a1a1a'...");
```

The colour `#1a1a1a` is `--text-light` URL-encoded. **This is the same hardcoded-colour exception documented for checkbox.** CSS variables can't be referenced inside a data-URI — the URL is parsed before custom properties are resolved.

The component handles two encodings:

- **Light mode:** `#1a1a1a` (text-light)
- **Dark mode:** `#f0f0f0` (text-dark), applied via `@media (prefers-color-scheme: dark)`
- **Print:** `#1a1a1a` re-encoded inside `@media print` so the arrow prints against the transparent background

If your fork changes `--text-light` or `--text-dark` to different values, re-encode the SVG strokes to match. This is the unavoidable trade-off of using SVG-as-background-image — flagged in the docs backlog as a system-wide pattern to consolidate.

---

## States

| State | Trigger | Effect |
|---|---|---|
| Default | _(none)_ | Sunken background, neutral border, arrow |
| Hover | `:hover` | Border colour shifts to `--text` |
| Focus | `:focus-visible` | Focus ring from foundations/focus-ring |
| Invalid | `aria-invalid="true"` | Border becomes `--error`, hover preserved |
| Disabled | `disabled` attribute | Opacity 0.5, cursor not-allowed, label dims via `:has()` |

All states are driven by HTML attributes — never by class names. Same convention as text-input, checkbox, and button.

---

## The placeholder option pattern

Native `<select>` has no placeholder attribute. The convention is a first `<option>` with empty value and a prompt label:

```html
<select required>
  <option value="">Select a region…</option>
  <option value="north">North</option>
  <option value="south">South</option>
</select>
```

The empty value means form validation can detect "nothing chosen" via `required`. The component does not style this empty state differently from selected options — getting the placeholder text to look subtle would require `:has(option[value=""]:checked)`, which is fiddly and depends on the first option specifically having `value=""`. The user-facing signal of "have I chosen?" is the option text itself; the prompt label "Select a region…" carries that meaning on its own.

If your fork wants the placeholder to look genuinely placeholder-y, override locally:

```css
select:has(option[value=""]:checked) {
  color: var(--text-subtle);
}
```

---

## Invalid state — timing matters

Same rule as text-input and checkbox: `aria-invalid="true"` is applied **after** a validation attempt, never on a pristine field.

The most common case for an invalid select is a required field submitted with the placeholder option still selected. The form's submit handler adds `aria-invalid="true"` and links an error message via `aria-describedby`:

```html
<label for="contact">
  Preferred contact method <span class="required" aria-hidden="true">*</span>
</label>
<select id="contact" required aria-invalid="true" aria-describedby="contact-error">
  <option value="">Choose one…</option>
  <option value="email">Email</option>
  <option value="signal">Signal</option>
</select>
<p id="contact-error" class="field-error">Please choose a contact method.</p>
```

Don't apply `aria-invalid="true"` on page load. Don't apply it to required fields preemptively.

---

## Disabled state

The standard `disabled` attribute drives styling. Both the select AND the wrapping `<label>` visually dim, via the `:has()` pseudo-class:

```css
label:has(select[disabled]) {
  opacity: 0.5;
  cursor: not-allowed;
}
```

`:has()` is supported in all current browsers (~94% global). On older browsers, the select dims correctly but the label stays at full opacity — a mild visual inconsistency, acceptable graceful degradation. Same pattern as checkbox and text-input.

---

## `<optgroup>` — semantically supported, visually deferred

`<optgroup>` provides semantic grouping that assistive tech announces correctly. CSS reach into `<optgroup>` labels is partial and patchy across browsers — you can sometimes set `font-weight` and `color`, but support is inconsistent and the result is half-styled at best.

**The component does not attempt to style `<optgroup>` labels.** Half-styled is worse than UA-styled — a coherent OS rendering reads better than a partial override.

Use `<optgroup>` when you want screen readers to announce the grouping. Don't rely on it for visually meaningful grouping — that's a different design problem (often solved by a custom listbox, or by splitting the form into smaller selects).

---

## `<select multiple>` — usually use checkbox groups instead

Multi-select renders as a list-box rather than a dropdown — there is no closed state, no panel to open. `appearance: none` plus the system styling works reasonably well here, but the interaction model is hostile:

- **Desktop:** Cmd-click (Mac) or Ctrl-click (Windows) to add to selection; Shift-click for range. Most users don't know this.
- **Touch:** broken or inconsistent across platforms. iOS shows a modal picker; Android varies; some browsers don't allow multi-select at all on touch.

For most multi-select cases, **a group of checkboxes is the better answer.** Every option is visible, the interaction is unambiguous, and it works the same way on every device.

The component styles `<select multiple>` so forks that have a genuine need (technical admin UI, long lists where checkbox groups would dominate the page) get something consistent with the rest of the system. The styling:

- Removes the arrow (no panel to open, so the arrow would mislead)
- Pads options consistently
- Highlights `:checked` options with `--action` background

If you find yourself reaching for `<select multiple>`, ask whether checkbox groups would serve the user better first.

---

## Buttons inside forms — the same gotcha applies

`<select>` doesn't have the type-defaulting trap that `<button>` has, but the broader form-submission rule still applies: pressing Enter inside a `<select>` may submit the form. This is browser behaviour, not something the component handles.

Test forms end-to-end, especially the keyboard path: tab through fields, change a select value, press Enter. The form should submit only when the user intends to submit.

---

## Tokens used

| Token | Used for |
|---|---|
| `--surface-sunken` | Background |
| `--border-strong`, `--text` | Default and hover border colour |
| `--border-thick` | Border width (1.5px) |
| `--radius-md` | Corner radius |
| `--space-2`, `--space-3`, `--space-6` | Padding |
| `--font-sans`, `--text-base`, `--weight-regular`, `--leading-snug` | Typography |
| `--text` | Text colour |
| `--error` | Invalid border |
| `--action`, `--surface` | Multi-select `:checked` option fill and text |

Hardcoded values, as documented exceptions:

- `#1a1a1a` (light mode + print) and `#f0f0f0` (dark mode) in the arrow SVG data-URIs
- The 12px arrow size

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/focus-ring/focus-ring.css`
3. Include `components/controls/select/select.css`
4. Write semantic HTML — `<select>` inside a wrapping `<label>` (or using a `for`/`id` pair), with options as children

---

## How to override

**To change the closed-state background to match text-input differently:**

Both controls use `--surface-sunken`. To diverge, override in your fork's stylesheet:

```css
select {
  background-color: var(--surface);  /* or any token */
}
```

Consider whether the divergence from text-input is worth the visual inconsistency before making the change.

**To use a different arrow:**

Replace the `background-image` URL with your own SVG data-URI. Keep `background-position: right var(--space-3) center` and `background-size: 12px 12px` for layout consistency.

Don't forget to override the dark-mode and print-mode rules too if your arrow needs to change colour across modes.

**To allow `<select multiple>` to read as more visually distinct from single-select:**

```css
select[multiple] {
  background-color: var(--surface);  /* lift it out of the sunken treatment */
  padding: var(--space-2) var(--space-3);
}
```

Consider whether visually distinguishing it is the right call — it might encourage use in cases where checkbox groups would serve users better.

---

## Print rules

- Background removed (saves ink)
- Border kept in `--text` colour for visibility
- Arrow SVG re-encoded with the print text colour so it remains visible against the transparent background
- For `<select multiple>`: no arrow change needed; selected options lose their coloured background and rely on browser default rendering (variable across browsers — some print a checkmark, some bold, some nothing)

The printed page accurately shows the closed-state appearance with the chosen option visible. The open-state panel is never part of print, by definition.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
