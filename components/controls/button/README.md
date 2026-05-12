# button

The primary interactive control of the system. Four variants by emphasis, two sizes, plus loading and disabled states.

This is the first **control** component — opt-in, used where needed, not applied globally. It sets the pattern for the system's two-vocabulary structure: editorial elements (cards, dividers, lists) use `--radius-none` and feel like print; interactive controls (buttons and onward) use `--radius-md` and feel like software.

---

## What it styles

- `<button>` — the primary case, in any of four variants
- `<input type="submit">`, `<input type="button">`, `<input type="reset">` — same styling, same selectors
- `<a class="btn">` — for actions that genuinely navigate to a URL

---

## Variants

| Variant | Class | When to use |
|---|---|---|
| Primary | _(no class)_ | The affirmative action on a page. Usually one per view. |
| Secondary | `btn-secondary` | Neutral alternative. Pairs with a primary as "cancel" or "alternate". |
| Ghost | `btn-ghost` | Lowest emphasis. "Skip", "Maybe later", inline "more" controls. |
| Destructive | `btn-destructive` | Delete, remove, irreversible actions. |

**The default `<button>` is primary.** No class needed for the common case — write `<button>Save</button>` and the styling is correct. Use the variant classes for the less-common cases.

**On the destructive variant.** It starts as an outline (red border, red text, no fill) and fills with red on hover. This is deliberate: a fully-filled red button gets clicked accidentally. The outlined-then-filled pattern adds friction at exactly the moment a user is committing to an irreversible action.

---

## Sizes

| Size | Class | Use |
|---|---|---|
| Default | _(no class)_ | Standard buttons in forms, dialogs, page-level actions. |
| Small | `btn-sm` | Inline actions in tables, secondary controls in card headers, dense UI. |

The default size has a `min-height: 36px` which meets WCAG touch target guidance. The small size drops to `min-height: 28px` — acceptable for dense desktop UI but worth avoiding on primary mobile actions.

---

## Modifiers

- `btn-block` — full-width. Common in mobile forms. Apply alongside any variant: `<button class="btn-block">Submit</button>` or `<button class="btn-secondary btn-block">Cancel</button>`.

---

## States

| State | Trigger | Effect |
|---|---|---|
| Default | _(none)_ | Variant's base appearance |
| Hover | `:hover` | Background lift per variant |
| Active | `:active` | `filter: brightness(0.92)` — tactile feedback |
| Focus | `:focus-visible` | Handled by the `focus-ring` foundation |
| Disabled | `disabled` attribute | Opacity 0.5, `cursor: not-allowed`, no hover |
| Loading | `aria-busy="true"` | Spinner appears, pointer events disabled |

**Both disabled and loading use HTML attributes, not classes.** The project convention is that state is inferred from attributes — `disabled`, `aria-busy`, `aria-invalid`, `aria-current` — never from class names like `.btn-disabled` or `.btn-loading`. Using attributes keeps the markup honest about what the control's state actually is, and makes the styling consistent with assistive technology behaviour.

---

## The loading state

Add `aria-busy="true"` to the button when an action starts; remove it when finished. The CSS handles the rest — a CSS-only spinner appears before the label, the label remains visible, and pointer events are disabled so the button can't be clicked while loading.

```html
<!-- Idle -->
<button>Save changes</button>

<!-- Loading -->
<button aria-busy="true">Saving…</button>
```

Usually a few lines of JS toggle the attribute, often handled by your form library. The spinner pauses (but stays visible as a partial ring) under `prefers-reduced-motion`.

---

## Icon and label

The system rule: **icons always paired with visible labels, never icon-only.** This rule comes from the "designed for the edges" principle — icon fonts fail to load, users don't all recognise the same symbols, printed pages lose icon meaning.

```html
<!-- Correct: icon and label -->
<button>
  <svg viewBox="0 0 16 16" aria-hidden="true">...</svg>
  Add member
</button>

<!-- Wrong: icon only -->
<button>
  <svg viewBox="0 0 16 16">...</svg>
</button>
```

The icon goes inside the button as a sibling of the label text. The component handles the layout — `inline-flex` with `gap` between icon and label. Icons should:

- Have `aria-hidden="true"` (the label provides the accessible name)
- Be sized with `width: 1em; height: 1em` (the CSS does this automatically for `<svg>` and `.icon` children)
- Use `currentColor` for fills and strokes so they inherit the button's text colour across variants

If your interface genuinely needs an icon-only control for space reasons (a close button in a toolbar), provide an `aria-label` — but first consider whether the affordance will be clear to non-expert users.

---

## Buttons inside forms — read this

HTML has a quirk worth knowing about, because it catches almost every developer at least once.

**When you put a `<button>` inside a `<form>`, the browser assumes its `type` is `submit` unless you say otherwise.** A button labelled "Show password" or "Add row" inside a form will submit the form when clicked. Pressing Enter inside a text input will also "click" whichever submit button comes first in the markup.

The fix is to set `type="button"` explicitly on every button inside a form that isn't the submit button:

```html
<!-- Wrong: "Show password" submits the form -->
<form>
  <input type="text" name="email">
  <button>Show password</button>
  <button>Sign in</button>
</form>

<!-- Correct: explicit types -->
<form>
  <input type="text" name="email">
  <button type="button">Show password</button>
  <button type="submit">Sign in</button>
</form>
```

Being explicit about `type="submit"` on the actual submit button isn't required (it's the default) but it makes intent legible. The `type="button"` on the other button is what prevents the bug.

CSS can't fix this — it's how HTML works. Documenting it here, and demonstrating it in the reference page, is the best the system can do.

---

## Link-as-button — when, and when not

Use `<a class="btn">` only when the action genuinely navigates to a different page or URL. Browse the docs, go to settings, view the profile.

Do NOT use `<a class="btn">` for actions that:

- Submit a form
- Trigger JavaScript
- Open a modal
- Do anything that isn't a navigation

Links and buttons behave differently: Enter activates both, but Space only activates buttons. Screen readers announce them differently ("link" vs "button"). Styling a link as a button does not change its semantics — a user navigating with assistive tech will be told it's a link, and will expect navigation behaviour.

When the action isn't navigation, use `<button>`.

---

## Tokens used

| Token | Used for |
|---|---|
| `--action`, `--action-hover` | Primary background and hover |
| `--error` | Destructive border, text, and hover fill |
| `--surface`, `--surface-raised` | Light text on filled buttons; hover lift on secondary/ghost |
| `--text` | Default text colour for secondary, ghost, and link variants |
| `--border-strong` | Secondary outline |
| `--border-thick` | Border width (1.5px) |
| `--radius-md`, `--radius-sm` | Default and small corner radius |
| `--space-1` to `--space-4` | Padding and gap values |
| `--text-base`, `--text-sm` | Default and small font size |
| `--weight-medium` | All button text |
| `--font-sans` | All button text |

No values are hardcoded.

---

## Accessibility

- **Focus styling** is provided by the `focus-ring` foundation. The button CSS doesn't override it. Always include `focus-ring.css` in pages that use buttons.
- **Disabled buttons are not focusable** by default — this is browser behaviour. Forks needing focusable-disabled buttons (so screen reader users can discover them) should use `aria-disabled="true"` plus JS click-blocking, which is a fork concern, not a foundation default.
- **Loading state** is communicated to assistive tech via `aria-busy="true"`. Screen readers announce that the control is busy.
- **Icon-only buttons** need `aria-label` if used (which the system discourages — see above).
- **Touch targets:** the default size meets WCAG (36px); the small size doesn't (28px). Avoid `btn-sm` for primary mobile actions.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/focus-ring/focus-ring.css` (buttons need it)
3. Include `components/controls/button/button.css`
4. Write semantic HTML — `<button>` for actions, `<a class="btn">` for navigation styled as a button

---

## How to override

**To swap the primary colour to a different activity token:**

```css
button,
.btn {
  background: var(--share);
  border-color: var(--share);
}
button:hover,
.btn:hover {
  background: var(--share);  /* or a derived hover token */
}
```

**To change the default size to be larger:** override the padding and `min-height` in your fork's stylesheet.

**To make `<button>` the secondary variant by default and require a class for primary:** swap the base styles with the `.btn-secondary` styles, and rename. This is a significant fork — document it.

**To remove the friction model on destructive buttons** (i.e. make them filled-red by default):

```css
.btn-destructive {
  background: var(--error);
  color: var(--surface);
}
```

Consider whether the safety rationale still holds in your context before making this change.

---

## Print rules

Buttons in print:

- Backgrounds removed (saves ink)
- Borders printed in `--text` colour for visibility
- Spinner animation paused
- Disabled and loading opacity preserved so the printed page reflects what was on screen

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
