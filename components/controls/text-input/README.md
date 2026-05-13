# text-input

Text-shaped form inputs and `<textarea>`. Covers every input where the user types text.

This component styles the input element itself. Composition concerns — how `<label>`, help text, inline errors, and the required indicator fit together — are handled by the `form-field` pattern. An input with a wrapping `<label>` is a complete, accessible control on its own; `form-field` will wrap that pattern later to handle more elaborate compositions.

---

## What it styles

| Element | Notes |
|---|---|
| `<input type="text">` | The default case |
| `<input type="email">` | Same styling; browser provides email validation |
| `<input type="password">` | Same styling; browser handles masking |
| `<input type="url">` | Same styling; browser provides URL validation |
| `<input type="tel">` | Same styling; mobile browsers may show numeric keypad |
| `<input type="search">` | Same styling; UA "x" clear button is suppressed |
| `<input type="number">` | Same styling; UA up/down spinners are suppressed |
| `<textarea>` | Inherits all of the above plus multi-line specifics |

It does **not** style:
- `date`, `time`, `color`, `file` — visually distinct, UA-specific affordances. Deferred or covered separately
- `checkbox`, `radio` — separate components
- `<select>` — separate component
- The `<label>` element or any composition around the input — that's `form-field`

---

## The defaults

| Property | Value | Token |
|---|---|---|
| Padding | 8px 12px | `--space-2`, `--space-3` |
| Min height | 36px | _(matches button)_ |
| Border | 1.5px solid neutral | `--border-thick`, `--border` |
| Background | Warm cream (sunken) | `--surface-sunken` |
| Radius | 8px | `--radius-md` |
| Font | Sans, 14px, regular | `--font-sans`, `--text-base`, `--weight-regular` |

**On the sunken background.** Inputs use `--surface-sunken` rather than matching the page surface so they read as "this is where you type" against an editorial page. The form-like recess is the strongest affordance for an empty field.

**On the height matching buttons.** A search input next to a submit button, an email field next to an "Add" button — these need to align cleanly. The 36px min-height is shared with the button component for exactly this reason.

**On focus.** The input's own border does NOT change colour on focus. The focus signal is provided entirely by the `focus-ring` foundation, which uses `outline` (sitting outside the input's box). This means there's no layout shift between unfocused and focused states. Always include `focus-ring.css` in pages that use inputs.

---

## States

| State | Trigger | Effect |
|---|---|---|
| Default | _(none)_ | Sunken background, neutral border |
| Focus | `:focus-visible` | Focus ring from foundations/focus-ring |
| Invalid | `aria-invalid="true"` | Border becomes `--error` colour, same width (no layout shift) |
| Disabled | `disabled` attribute | Opacity 0.5, `cursor: not-allowed`, not focusable |
| Readonly | `readonly` attribute | Lighter background (`--surface`), default cursor, still focusable |

**All states are driven by HTML attributes, not classes.** State is inferred from `disabled`, `readonly`, `aria-invalid`, `aria-current` — never from class names like `.input-error` or `.input-disabled`. The styling stays consistent with what assistive technology sees, and the markup stays honest about what the field's state actually is.

---

## Disabled vs readonly — they're different

| | `disabled` | `readonly` |
|---|---|---|
| Can be focused? | No | Yes |
| Can be selected/copied? | No | Yes |
| Submitted with the form? | No | Yes |
| Visual treatment | Opacity 0.5 | Lighter background |
| Use for | Field is genuinely inactive (e.g. disabled until a prior field is filled) | Field shows fixed content the user shouldn't edit (e.g. an auto-generated handle, a confirmed value) |

Use the one that matches the actual semantics. If the value should be submitted, it must be `readonly` — disabled fields don't get submitted.

---

## The invalid state — timing matters

The component styles `aria-invalid="true"` with a red border. Apply the attribute carefully:

**Do** apply `aria-invalid="true"` **after** a validation attempt:
- On blur, after the user has left the field
- On submit, when the form is being validated
- After the user has typed and you've checked their input

**Don't** apply `aria-invalid="true"` on:
- A field the user hasn't interacted with yet
- All required fields on page load
- Fields the user is still typing in

Marking pristine fields invalid is one of the most common form accessibility failures. It tells assistive tech (and the user) that something is wrong before they've had a chance to do anything wrong.

The associated error message should be linked via `aria-describedby`:

```html
<input type="email" aria-invalid="true" aria-describedby="email-error">
<p id="email-error">Enter a valid email address.</p>
```

---

## Placeholders — read this

Placeholders are auxiliary text. They are for example values and format hints, **not labels.**

**Never use a placeholder as a label:**

```html
<!-- Wrong: no label, just a placeholder -->
<input type="email" placeholder="Email address">
```

When the user starts typing, the placeholder disappears. The field is now unlabelled — for the user, for screen readers, for users who came back to a half-filled form, for users with cognitive disabilities who can't hold the prompt in working memory. This is one of the most common form accessibility failures.

**Always provide a real label:**

```html
<!-- Correct: visible label, placeholder gives an example -->
<label>
  Email address
  <input type="email" placeholder="you@community.org">
</label>
```

The placeholder colour is `--text-subtle` — visibly distinct from real values, but meets WCAG AA contrast for auxiliary text.

---

## The required indicator

The system uses the asterisk convention: `Label *`. The asterisk is:

- Coloured `--error` to make it stand out
- Marked `aria-hidden="true"` so screen readers don't say "asterisk" — the `required` attribute on the input is what screen readers announce

```html
<label>
  Email <span class="required" aria-hidden="true">*</span>
  <input type="email" required>
</label>
```

The actual `required` attribute on the input is what does the work — it triggers browser validation, it's announced by assistive tech, it's submitted with the form. The asterisk is purely visual.

(The `form-field` pattern will provide the proper class for this — `class="required"` in the examples is a stand-in for now.)

---

## Inputs without a visible label

Sometimes an input is its own label — a search field next to a magnifying glass icon, a single-field inline form. When there's no visible `<label>`, an `aria-label` is required:

```html
<input type="search" placeholder="Search members" aria-label="Search members">
```

A placeholder is not a substitute for an `aria-label`. Both are auxiliary text; the `aria-label` is what makes the input accessible.

---

## type="number" — usually the wrong choice

`type="number"` is for quantities — counts, measurements, things you'd do arithmetic with. Browsers apply number-specific behaviour to it:

- Up/down spinner controls (the component hides these for visual consistency, but the keyboard behaviour remains)
- Scientific notation parsing (`1e10` becomes a valid value)
- Locale-aware decimal separator handling
- Mobile browsers show a numeric keypad

For most "sequence of digits" use cases — phone numbers, postcodes, credit card numbers, room numbers, IDs — `type="number"` is the wrong choice. Use `type="text"` with `inputmode="numeric"`:

```html
<!-- Correct: phone number as text -->
<input type="tel">

<!-- Correct: postcode as numeric text -->
<input type="text" inputmode="numeric" pattern="[0-9]{5}">

<!-- Wrong: phone as number -->
<input type="number">  <!-- breaks for "+44 7700 900000" -->
```

---

## Tokens used

| Token | Used for |
|---|---|
| `--surface-sunken`, `--surface` | Default background; readonly background |
| `--border`, `--border-subtle`, `--error` | Default border; readonly border; invalid border |
| `--border-thick` | Border width (1.5px) |
| `--text`, `--text-subtle` | Value colour; placeholder colour |
| `--font-sans` | Input text |
| `--text-base` | Input text size |
| `--weight-regular` | Input text weight |
| `--leading-snug`, `--leading-base` | Single-line; textarea |
| `--radius-md` | Corner radius |
| `--space-2`, `--space-3` | Padding |

No values are hardcoded.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/focus-ring/focus-ring.css` (inputs need it)
3. Include `components/controls/text-input/text-input.css`
4. Wrap each input in a `<label>` for accessibility — or use `aria-label` if a visible label isn't possible

The `form-field` pattern will document the proper composition when it lands. Until then, the wrapping-`<label>` approach in the reference page is the simplest correct pattern.

---

## How to override

**To use a different background:** override `background` per-context. The most common variant is using `--surface` instead of `--surface-sunken` for a flatter look:

```css
.my-form input { background: var(--surface); }
```

**To match a tighter button size:** override padding and `min-height`:

```css
.compact-form input,
.compact-form textarea {
  padding: var(--space-1) var(--space-3);
  min-height: 28px;
}
```

**To disable textarea resize entirely:** uncommon and slightly paternalistic, but:

```css
textarea { resize: none; }
```

---

## Print rules

- Backgrounds removed (saves ink)
- Borders preserved in `--text` colour so the field outline still reads

The user's typed values print as part of the input's normal text rendering — the printed page is often a record of what was submitted.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
