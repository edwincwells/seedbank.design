# provenance

The stepped provenance chain: where a material, energy source, or other resource came from, stage by stage — each stage carrying a location, a distance, and a verification status.

This is the fourth component in the **ecological** layer and the one that composes the others: `verification-status` for each stage, and `eco-stat` for an uncertain per-stage quantity. It's the most structural piece in the layer, and the whole design turns on one decision — the chain *is* an ordered list, so it degrades to a plain numbered sequence with no CSS.

---

## What it styles

- `.provenance` — the container (no chrome — see below)
- `.provenance-header`, `.provenance-title`, `.provenance-summary` — the optional header
- `.provenance-chain` — the `<ol>`
- `.provenance-stage` — each `<li>`, and the rail dot + connector drawn on it
- `.provenance-stage-head`, `.provenance-location`, `.provenance-distance`, `.provenance-detail` — stage content
- Composed inside each stage: a `verification-status` wrapper, and optionally an `eco-stat`

---

## The markup is an `<ol>`, and that's the degradation story

A provenance chain is fundamentally an ordered sequence, so each stage is an `<li>` in an `<ol>`:

```html
<div class="provenance">
  <ol class="provenance-chain">
    <li class="provenance-stage">
      <div class="provenance-stage-head">
        <span class="provenance-location">Flax grown — Normandy, France</span>
        <span class="provenance-distance">0 km</span>
      </div>
      <p class="provenance-detail">Rainfed. No synthetic inputs. Organic certification, 2023.</p>
      <span class="verification" data-verification="verified">
        <span class="badge">Verified</span>
        <a href="…" rel="external">organic cert</a>
      </span>
    </li>
    …
  </ol>
</div>
```

With no CSS, the `<ol>` renders as a numbered list — stages in order, each with its location, distance, detail, verification text, and rationale link, all readable. `list-style: none` removes the numbers only when the stylesheet loads (the rail replaces them); strip the CSS and the numbers come back to carry the sequence. The timeline rail is pure enhancement built on top, and it disappears cleanly because it's all pseudo-elements.

A `<table>` was the alternative — stages do have column-ish data (stage / location / verification / distance) — but order is the primary meaning here, and a table implies independent rows. The ordered list degrades more honestly.

---

## Each stage composes `verification-status`

Per stage: a **title** (stage + location), an optional **detail** line, the **`verification-status`** component (verified / self-reported / not-disclosed, with its rationale link), and a **distance**. This is where verification-status earns its keep — and it confirms the three verification states are sufficient: every stage of a real chain is confirmed, claimed-but-unchecked, or unknown. No fourth "estimated" state is needed.

An **undisclosed stage is shown, not omitted** — it appears in the chain with a "Not disclosed" verification and a "—" distance. The gap is part of the provenance, per principle 02.

---

## The rail is neutral — verification is not in the dots

The timeline dots and connectors are **sequence and geometry only**, drawn in neutral border tokens. They are *not* colour-coded by verification status. Two reasons:

- **Meaning never rests on colour** (principle 01). The verification of a stage is carried by the composed badge, which has both a colour *and* a text label, so it survives e-ink, print, and colour-blind reading. The rail doesn't need to repeat it.
- **No double-coding.** Colouring the dot green/amber/grey would say the same thing the badge already says, in a less accessible way.

The prototype colour-coded the dots; this component deliberately doesn't. The rail is built from `::before` (the dot) and `::after` (the connector) on each `<li>` — decorative, never announced to assistive tech, and gone when CSS is absent. The last stage's connector is removed.

---

## Distance is mono metadata, not a forced `eco-stat`

Each stage's distance ("320 km", "2,410 km total") is plain mono text — a tokenised-metadata number, per the system's mono convention. It is **not** forced into a full `eco-stat` block; that would be too heavy for a compact timeline row.

`eco-stat` composes in only when a stage reports a genuinely uncertain **quantity** — for example a per-leg carbon estimate with a range. The reference page shows this on the weaving stage:

```html
<div class="eco-stat eco-stat-compact">
  <p class="eco-stat-label">Carbon, this leg</p>
  <p class="eco-stat-value"><data value="0.4">0.4</data> <span class="eco-stat-unit">kg CO₂e</span></p>
  <p class="eco-stat-range">estimated <data value="0.2">0.2</data>–<data value="0.7">0.7</data> kg CO₂e</p>
</div>
```

So "composes eco-stat" means *available as an optional per-stage quantity slot*, not *every distance is an eco-stat*.

---

## No chrome — provenance is composed, not framed

`.provenance` ships with **no border, background, or radius**. It's bare content that a card or dashboard wraps — consistent with the rest of the layer, where chrome and the data+action pairing live at composition level. The only structural line the component draws is the hairline under the header. The reference page shows it inside a stand-in card to illustrate composition, but that card is not part of the component.

Likewise there is **no action footer**. The decision a provenance chain supports — request disclosure for the undisclosed stage, find a closer source for the long leg — is the composer's to add (principle 03). The component is the data.

---

## The header

Optional. A title and an honest summary line:

```html
<div class="provenance-header">
  <h3 class="provenance-title">Linen fabric — 2m offcut</h3>
  <p class="provenance-summary">3 of 5 stages verified · 2,410 km total</p>
</div>
```

The verified-count is a deliberate principle-02 touch — it surfaces, up front, how much of the chain is actually backed up rather than claimed. The title's heading level is the author's to choose; the class only sets appearance.

---

## Tokens used

| Token | Used for |
|---|---|
| `--border-strong` | Rail dots |
| `--border`, `--border-hairline` | Rail connectors; header rule |
| `--text`, `--text-muted`, `--text-subtle` | Location / detail / distance + summary |
| `--text-md`, `--text-sm` | Title and location / detail and distance |
| `--weight-medium` | Title and location |
| `--font-mono` | Distance |
| `--font-sans` | Everything else |
| `--space-1` … `--space-5` | Rail gutter, dot size, inter-stage gap, header spacing |

No new tokens. The dot is sized from `--space-2` (8px) — on the spacing scale, not a hardcoded off-scale value. Verification colour and any eco-stat come from the composed components.

---

## Accessibility

- The chain is a real `<ol>`, so assistive tech announces it as an ordered list with a count and reads the stages in sequence.
- The rail is entirely `::before`/`::after` pseudo-elements — decorative, never announced.
- Verification is carried by the composed badge's text label, not by rail colour, so no meaning rests on colour.
- The rationale links are normal focusable links; the focus ring comes from `foundations/focus-ring`.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include the foundations the stages need: `prose`, `links`, `focus-ring`
3. Include the composed components: `display/badge`, `ecological/verification-status`, and `ecological/eco-stat` (only if a stage uses an eco-stat quantity)
4. Include `components/ecological/provenance/provenance.css`
5. Write the `<ol class="provenance-chain">` with a `<li class="provenance-stage">` per stage, composing a `verification-status` in each

---

## How to override

**To restore colour-coded dots** (a fork that wants the prototype's coloured rail): set `background` on `.provenance-stage::before` per a state class or attribute you add. Consider first whether it earns the double-coding against the badge — the neutral rail is a deliberate accessibility choice.

**To wrap provenance in a card by default:** apply your card chrome to a container around `.provenance`, or to `.provenance` itself in your fork's stylesheet. The component stays bare so the choice is yours.

**To tune the rail:** the dot size and vertical offset are the two custom properties on `.provenance-chain` (`--prov-dot`, `--prov-offset`); adjust them in one place.

---

## Print rules

The rail is thin and neutral, so it prints as-is with negligible ink. The one print rule is `break-inside: avoid` on each stage, so a stage's dot doesn't get orphaned from its content across a page break. Verification badges suppress their tint in print (their own rule); the rationale links expand to their URLs (the links foundation).

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
