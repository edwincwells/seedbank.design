# focus-ring

The focus indicator applied to every interactive element. Settled once here so every component inherits a consistent focus appearance.

This is a **foundation** — it is not opted into per-component, it propagates globally. Include `focus-ring.css` once in your fork and every interactive element receives the ring on keyboard focus.

---

## What it does

- Suppresses the browser's default focus ring (which varies wildly across browsers and is often invisible against the system's surface colours)
- Applies a 2px solid outline in the `--focus` colour to every focusable element on `:focus-visible` — the state browsers use to indicate "this focus came from a keyboard or assistive technology, not a mouse click"
- Uses `outline-offset: 2px` to give the ring breathing room from the element edge
- Respects forced-colors mode (Windows High Contrast and similar) by switching to the system `Highlight` colour automatically

---

## What it covers

Every standard interactive element type:

- `<button>`, `<a>`, `<input>`, `<select>`, `<textarea>`, `<summary>`
- Any element with `role="button"`
- Any element with `tabindex` other than `-1`

Elements with `tabindex="-1"` (programmatically focusable but not in the tab order) do not receive the ring. Plain non-focusable elements do not receive the ring.

---

## Tokens used

| Token | Used for |
|-------|----------|
| `--focus` | Ring colour. Resolves to `--focus-light` or `--focus-dark` by mode. |
| `--border-focus` | Ring width (2px). Lives in `tokens.css` section 6. |

The offset (2px) is currently hardcoded. If a fork needs to remap it, change the value in `focus-ring.css` directly.

---

## How to use it in your fork

1. Include `tokens/tokens.css` in your stylesheet bundle (you already do, for everything else)
2. Include `components/foundations/focus-ring/focus-ring.css`
3. Write semantic HTML — `<button>`, `<a>`, `<input>`. The ring appears on keyboard focus automatically.

You do not need to add classes. You do not need to opt in per element. This is the point of a foundation.

---

## How to override

**To change the ring colour or width globally:** remap `--focus` or `--border-focus` in `tokens.css`. This is the recommended path — it keeps the system coherent.

**To change the offset globally:** edit `outline-offset` in `focus-ring.css`. This is local-only because the offset is rarely something a fork needs to change.

**To override for a specific element:** write a more specific selector in your own CSS. Consider whether the override reduces accessibility before doing so. A common case where override is legitimate: a custom control whose visual edge is far from its hit area, where `outline-offset: 0` reads better.

**Do not** suppress the ring without replacing it. A focusable element with no visible focus state is unusable for keyboard users and fails WCAG 2.4.7 (Focus Visible).

---

## Edge cases worth knowing

- **Compact controls (checkbox, radio):** the 2px offset can push the ring outside a touch target on small native inputs. Acceptable trade-off — the ring's purpose is keyboard visibility, not touch.
- **Dense lists:** in a list of focusable rows with no margin between them, focus rings on adjacent rows can visually collide. If this becomes a problem in a real component, increase the row spacing in that component, not the focus offset.
- **Buttons on activity-coloured surfaces:** if the surface is `--share` blue and the focus ring is `--focus` blue, the ring vanishes. The system does not currently use activity-coloured surfaces, so this is a flag-for-later, not a current bug. If a fork introduces such a surface, override the focus colour for that context.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
