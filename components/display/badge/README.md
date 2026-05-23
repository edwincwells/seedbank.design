# badge

A small pill-shaped label for status, category, or count indicators. Non-interactive by default — a `<span>`, not a button or link. One visual treatment, nine colour options.

This is the first **display** component. Display components are opt-in (like controls) — they don't apply globally via element selectors, they're invoked by class.

---

## What it styles

- `<span class="badge">` — the only selector, with optional colour variant classes

That's the whole component. No `<button>`-shaped variant, no removable-chip variant, no notification-dot variant.

---

## When to use it

- Status markers on cards, list items, table cells (— "Active", "Draft", "Archived")
- Category tags (— "Tools", "Food", "Childcare")
- Count indicators (— "12 new", "3 pending")

When *not* to use it:

- **Don't use a badge for emphasis.** A badge is a label, not a button. If the user is meant to click it, wrap it in an `<a>` or `<button>` — don't add hover states to the badge itself.
- **Don't use a badge as a notification dot.** A notification dot is a different affordance (smaller, no label, position-anchored to its target). If you need one, build it separately.
- **Don't use a badge for removable chips.** A chip with an × is interactive and needs its own component. Deferred — not in v0.1.

---

## Variants

The variants split into three groups along the same axis as the token system. Pick from the group that matches what the badge is communicating.

### Neutral (default)

```html
<span class="badge">Draft</span>
```

No colour class. Uses `--text` on `--surface-raised`. This is the right choice for most badges — categories, drafts, archived items, generic tags. **Reach for a coloured variant only when the colour is doing work.**

### Activity variants

For status that names what the community does in the world: cultivating, repairing, sharing, making.

| Class | Token | Typical use |
|---|---|---|
| `.badge-grow` | `--grow` | Available, growing, cultivating |
| `.badge-repair` | `--repair` | Under repair, in maintenance |
| `.badge-share` | `--share` | In use, shared, communal |
| `.badge-make` | `--make` | In progress, being made |

### Functional variants

For status that the interface is reporting: a thing succeeded, a thing needs attention, etc.

| Class | Token | Typical use |
|---|---|---|
| `.badge-success` | `--success` | Submitted, completed, approved |
| `.badge-warning` | `--warning` | Needs review, attention required |
| `.badge-info` | `--info` | New, informational |
| `.badge-error` | `--error` | Failed, rejected |

### Activity vs functional — which to pick

In the default palette, `.badge-grow` and `.badge-success` produce identical visuals (both derive from the same green). They are still semantically distinct. Pick by what the badge means:

- A garden plot's status is "Growing" → `.badge-grow` (activity)
- A form submission's status is "Submitted" → `.badge-success` (functional)

A fork that remaps `--grow` to terracotta keeps `--success` green. The garden plot badge becomes terracotta; the form submission badge stays green. The distinction matters when forks remap activity colours — see `tokens.css` section 2.

---

## Visual treatment

One treatment only: a subtle tint background with saturated coloured text.

The tint is derived per-token via `color-mix()`:

```css
background: color-mix(in srgb, var(--grow) 15%, var(--surface));
color: var(--grow);
```

A fork that remaps `--grow` gets the new tint automatically — no need to re-derive hex values. Dark mode bumps the mix ratio from 15% to 22% because the lighter dark-mode token values need more saturation to lift off the dark surface.

**Why not a solid-filled variant.** A solid filled badge competes visually with buttons, and the system already has buttons for that emphasis. If your fork genuinely needs solid badges (e.g. for table cells where subtle tints disappear on busy backgrounds), it's a six-line override — see below.

**Why `color-mix()`.** Browser support is ~94% globally. Older browsers fall back to showing the text colour with no background tint — the badge is still readable, still semantic, still passes WCAG AA. The trade-off is acceptable; the alternative (hardcoded tint hexes per token) would force forks to re-derive every tint on every remap.

**The neutral-only hairline border.** The neutral default (`.badge` with no colour class) uses `--surface-raised` as its background, which would visually disappear if the badge is placed on a card or row already using `--surface-raised`. A hairline border in `--border` carries the badge's definition on any of the three surface tokens. Coloured variants don't need this — their tint background provides the lift, and a border on top of the tint reads as visual noise. If your fork adds a new colour variant, follow the same pattern: set the tint background and add `border-color: transparent;` to suppress the inherited hairline.

---

## Sizing

One size only. Padding is roughly 3px / 12px, with `--text-xs` (11px) at medium weight.

No `.badge-sm` variant. A badge smaller than this is illegible; a badge larger is a button. If your fork needs denser badges for a specific table or dashboard, override locally:

```css
.dense-table .badge {
  padding: 2px var(--space-2);
  font-size: 10px;
}
```

---

## Icons

The base `.badge` uses `inline-flex` with a 4px gap, so a badge with an icon child works without any additional class:

```html
<span class="badge badge-grow">
  <svg viewBox="0 0 16 16" aria-hidden="true">…</svg>
  Verified
</span>
```

Icons:

- Have `aria-hidden="true"` (the label provides the accessible name)
- Are sized to `1em` square automatically by the CSS
- Use `currentColor` for strokes/fills so they inherit the badge's text colour

The system's icon-with-label rule applies: **a badge with an icon must still have visible text alongside it.** No icon-only badges. Ever. This is the same rule as buttons — see PRINCIPLES.md #01.

---

## Clickable badges — wrap, don't restyle

Badges are non-interactive labels. To make one clickable (filter by tag, navigate to a category), wrap the `<span>` in an `<a>` or `<button>`:

```html
<a href="/categories/workshops" style="text-decoration: none;">
  <span class="badge badge-make">Workshops</span>
</a>
```

The badge styling stays unchanged. The wrapper provides the interaction, the focus ring, and the cursor. Keeping these concerns separated means the badge component stays honest about what it is — a label — and the wrapper handles the semantics correctly.

Don't:

- Add `cursor: pointer` to `.badge`
- Add `:hover` states to `.badge`
- Use `<button class="badge">` to "make a clickable badge" — the button's default chrome will need overriding anyway, and the result will be a badge-shaped button, not a clickable badge

---

## Tokens used

| Token | Used for |
|---|---|
| `--text` | Default text colour |
| `--surface`, `--surface-raised` | Neutral background; mix base for tints |
| `--border` | Neutral default hairline border |
| `--grow`, `--repair`, `--share`, `--make` | Activity variant backgrounds and text |
| `--success`, `--warning`, `--info`, `--error` | Functional variant backgrounds and text |
| `--font-sans` | Badge text |
| `--text-xs` | Badge font size |
| `--weight-medium` | Badge font weight |
| `--leading-snug` | Badge line height |
| `--space-1`, `--space-3` | Icon-label gap; horizontal padding |
| `--radius-pill` | Pill shape |
| `--border-hairline` | Print-mode border |

No values are hardcoded except the 3px vertical padding (the one value in the system that doesn't align to the spacing scale — tested down from `--space-1` for a tighter pill).

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/display/badge/badge.css`
3. Write `<span class="badge">Label</span>`, adding a colour variant class only when colour is doing work

The badge needs no other foundation. Prose is optional but recommended (it sets the body font).

---

## How to override

**To add a solid-filled variant:**

```css
.badge-solid {
  background: var(--action);
  color: var(--surface);
}
.badge-solid.badge-error {
  background: var(--error);
  color: var(--surface);
}
/* …and so on per variant */
```

Be explicit about why your fork needs this — the subtle-only default is a deliberate choice.

**To remove the icon support** (forks that want strict label-only badges):

```css
.badge {
  display: inline-block;
}
.badge > svg,
.badge > .icon {
  display: none;
}
```

**To change the default neutral background:** the neutral default uses `--surface-raised`. Override in your fork's stylesheet:

```css
.badge {
  background: var(--surface-sunken);
  /* or any other token */
}
```

**To swap to a hardcoded-tint approach** (forks deploying to older browsers without `color-mix()` support):

```css
.badge-grow {
  background: #e8f5e0;  /* your light-mode tint */
  color: var(--grow);
}
@media (prefers-color-scheme: dark) {
  .badge-grow {
    background: #1a3311;
  }
}
/* …and so on for each variant */
```

You lose automatic remapping — change `--grow` and you'll also need to update the hardcoded tint. Worth it only if you genuinely need to support browsers without `color-mix()`.

---

## Accessibility

- **Badges are not focusable.** They're labels. The wrapping `<a>` or `<button>`, if present, is the focusable element.
- **Colour is not the only signal.** Every badge has a text label. A screen reader user gets "Under repair" regardless of whether the badge is amber.
- **Contrast.** All variant text-on-tint combinations pass WCAG AA at the `--text-xs` size in both light and dark modes. This is verified by the token system's contrast checks plus the tint mixing ratios.
- **Em-dash convention** keeps the dash in the text content, so screen readers read what users see — not a pseudo-element artefact.

---

## Print rules

- Tint backgrounds removed (saves ink)
- Text forced to `--text` (effectively black) — some tinted variants print poorly as coloured text on white
- Hairline border added so the pill shape remains legible as a bounded unit

Printed badges still read as badges, just monochromatically.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
