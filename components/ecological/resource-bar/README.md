# resource-bar

A segmented shared-capacity bar: a collective resource — an energy budget, shared water, a fund — shown as parts of a whole, with the available remainder named explicitly. A target/progress variant folds in for "X of Y toward a goal".

This is the fifth and final component of the **ecological** layer. It's principle 03 made visual (shared capacity, not individual usage) and principle 01 under its hardest test in the layer — because a bar is the most colour-dependent thing the system draws, and the system forbids meaning that rests on colour *alone*.

---

## What it styles

- `.resource-bar` — the container (no chrome — see below)
- `.resource-bar-header`, `.resource-bar-title`, `.resource-bar-summary` — the optional header
- `.resource-bar-track` — the aria-hidden visual bar
- `.resource-bar-segment` — a used segment (width from `--portion`, hue from `--resource-fill`)
- `.resource-bar-marker` — the optional target threshold marker
- `.resource-bar-legend`, `.resource-bar-legend-item`, `.resource-bar-swatch`, `.resource-bar-share` — the canonical breakdown
- `.resource-bar-source` — an optional static source/timestamp line

---

## How segments are told apart — the load-bearing decision

Used segments are coloured from the **activity palette** (`--grow`, `--repair`, `--share`, `--make`) — distinct **hues**, assigned per segment via `--resource-fill`, with a matching legend swatch as the join key. The available remainder is **not** coloured: it's the unfilled track (`--surface-sunken`), with a hairline-bordered neutral swatch. So "used categories" read as colour, "available" reads as empty.

**Why hue, and why this is correct rather than the earlier ramp.** The segments are different *kinds* — categorical data — not a sequence. Categorical data wants distinct hues, which say "different categories" without implying order. A light-to-dark luminance ramp (an earlier draft of this component) is a *sequential* encoding: it falsely implies the segments are a progression or a magnitude ranking, which Heating / Workshop / Comms are not. Distinct hues carry the distinction at a glance and imply nothing that isn't true.

**Colour is decoration, not the load-bearing channel.** Principle 01 forbids meaning resting on colour *alone* — and it doesn't here. The legend's **label + share is the canonical text channel** underneath, so the breakdown survives grayscale, e-ink, print, colour-blind reading, and no-CSS. The swatch joins a legend row to its block when colour is available; the text carries it when colour isn't. This is the layering the cut list intended when it called colour "decoration": a non-colour channel carries the meaning, and colour rides on top to make it fast.

```html
<div class="resource-bar-track" aria-hidden="true">
  <div class="resource-bar-segment" style="--portion: 38%; --resource-fill: var(--grow)"></div>
  <div class="resource-bar-segment" style="--portion: 18%; --resource-fill: var(--repair)"></div>
  <div class="resource-bar-segment" style="--portion: 12%; --resource-fill: var(--share)"></div>
</div>

<ul class="resource-bar-legend">
  <li class="resource-bar-legend-item">
    <span class="resource-bar-swatch" style="--resource-fill: var(--grow)" aria-hidden="true"></span>
    <span class="resource-bar-label">Heating</span> <span class="resource-bar-share">38%</span>
  </li>
  …
  <li class="resource-bar-legend-item">
    <span class="resource-bar-swatch" style="--resource-fill: var(--surface-sunken)" aria-hidden="true"></span>
    <span class="resource-bar-label">Available</span> <span class="resource-bar-share">32%</span>
  </li>
</ul>
```

A segment and its legend swatch must carry the **same** `--resource-fill`.

**Why the activity palette rather than a new one.** It's already a four-way, AA-tested, light/dark-paired, visually distinct categorical set — close to purpose-built for this — and reusing it costs zero new tokens. The tokens are mode-aware, so segments stay AA-distinct on the dark track in dark mode. Four hues also sets the natural ceiling: beyond ~4 series, use a table.

**The caveat: overloading.** Using activity hues for arbitrary categories means a `--grow`-coloured "Heating" segment is not asserting "Heating is a grow-activity," and a green segment may sit near a green activity-status badge elsewhere. The legend and the obvious chart context mitigate this, and the original prototype coloured segments this way and read fine. Two honest exits if it bothers your fork:

- **Segments that genuinely are activities.** If a bar shows a community's effort split across Growing / Repairing / Sharing / Making, the activity palette is not overloading — it's exactly right. Assign by meaning, not position.
- **A dedicated chart palette.** If you want hard separation from the activity identity, or more than four series, add a small categorical chart palette as new tokens and point `--resource-fill` at it. That breaks the layer's zero-new-token streak, which the layer's own rules permit when a genuine need surfaces.

---

## The legend is canonical; the bar is decoration

The source of truth is the **legend** — a `<ul>` listing each segment's swatch, label, and share, plus the available remainder. With no CSS that's a readable breakdown. The **track** is a separate `aria-hidden` row of segment divs that mirrors the same shares — exactly the pattern eco-stat uses, where the value lives in the text and the band is aria-hidden geometry. The segment divs carry no text, so with no CSS they collapse and only the legend remains.

`<meter>` and `<progress>` were considered for their free semantics and native degradation, and rejected: they're a cross-browser styling tarpit (the same reason `checkbox` went custom rather than fighting the UA control), and a text-canonical + aria-hidden-visual split keeps resource-bar consistent with its sibling eco-stat.

---

## The target / progress variant

Progress toward a goal is the degenerate segmented bar: one used segment (the progress) plus the remainder (the distance to the goal). With a single fill there's nothing to differentiate, so the segment takes one activity hue (`--grow` by default). The canonical text states "X of Y"; an optional `aria-hidden` marker shows an intermediate threshold — a budget cap, a warning line, a stretch goal:

```html
<div class="resource-bar-track" aria-hidden="true">
  <div class="resource-bar-segment" style="--portion: 63%; --resource-fill: var(--grow)"></div>
  <span class="resource-bar-marker" style="--at: 90%"></span>
</div>
```

The marker's meaning is in the summary text ("load-shedding threshold at 90%"), never in its position or colour alone. It's one component with a marker, not a second component.

---

## No chrome, no action footer, static only

`.resource-bar` ships with **no border, background, or radius** — bare content a card or dashboard composes, consistent with provenance and the rest of the layer. There is **no action footer**: the decision a budget supports — shift load to the generation window, ration, request more — is the composer's to add (principle 03). The component is the data.

Everything the cut list kills is absent: no slider simulator, no pulsing live-dot, no "Live" framing, no bar-width transition animation, no sentiment-coloured values. The bar is **static**. A live-updating reading is a documented fork JS concern, not shipped behaviour — and deliberately so: a live number implies infrastructure most communities don't have and reads as more certain than the data warrants.

---

## Source / meta line

Optional. Static metadata in mono — where the figure came from, when it was last read:

```html
<p class="resource-bar-source">Source: roof array · last read 4 minutes ago</p>
```

"Last read 4 minutes ago" is static text the page was built with, not a live timer.

---

## Tokens used

| Token | Used for |
|---|---|
| `--grow`, `--repair`, `--share`, `--make` | The categorical hue palette for used segments and their swatches |
| `--surface-sunken` | Track background and the "available" swatch (the available remainder) |
| `--text` | Target marker; legend share figures |
| `--text-muted`, `--text-subtle` | Legend labels; summary and source |
| `--border` | Swatch hairline border |
| `--text-md`, `--text-sm`, `--text-xs` | Title / legend and summary / source |
| `--weight-medium` | Title |
| `--font-mono` | Legend shares and source line |
| `--font-sans` | Everything else |
| `--space-1` … `--space-4` | Track height, swatch size, legend gaps, header spacing |
| `--radius-sm` | Track and swatch corners (matches eco-stat's band) |
| `--border-thick` | Inter-segment cut and marker width |
| `--border-hairline` | Swatch border |

No new tokens — the palette reuses the activity tokens the system already ships. The track height is `--space-2` (8px) and the swatch is `--space-3` (12px) — both on the spacing scale, not hardcoded off-scale values.

Author-set **custom properties** (presentational, not design tokens): `--portion` (per segment, its width), `--resource-fill` (the hue for a segment and its swatch), `--seg-gap` (the cut between segments), `--at` (the marker position). The width property is `--portion`, deliberately **not** `--share` — a presentational custom property must never reuse an activity-token name (`--grow` / `--repair` / `--share` / `--make`), or `var(--share)` inside the same element resolves to the local width value, not the colour token.

---

## Accessibility

- The legend is a real `<ul>`, announced as a list with its items — the canonical breakdown for assistive tech.
- The track and the swatches are `aria-hidden` — they're a visual mirror of the legend, so nothing is announced twice.
- No meaning rests on colour: distinct hues differentiate at a glance, but the legend names every part in text, so colour-blind, grayscale, e-ink, print, and no-CSS readers get the full breakdown. (Green/orange adjacency in the activity palette is exactly the case the text fallback covers.)

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `foundations/prose` (the header and legend are text)
3. Include `components/ecological/resource-bar/resource-bar.css`
4. Write the `aria-hidden` track with a segment per used share, each with an activity-hue `--resource-fill`; write a `<ul class="resource-bar-legend">` whose rows carry the matching swatch, label, and share, plus the available remainder

---

## How to override

**To assign the palette:** set `--resource-fill` per segment (and the matching legend swatch) to an activity token — `--grow`, `--repair`, `--share`, `--make`. Keep the segment and its swatch in sync. If your segments *are* activities, assign by meaning; otherwise assign in order.

**To use a dedicated chart palette** (avoid overloading the activity tokens, or need >4 series): define your palette in your fork's tokens and point `--resource-fill` at it. Keep the legend carrying label + share in text so principle 01 still holds.

**To change the inter-segment cut:** set `--seg-gap` on the `.resource-bar-track`.

**To wrap it in a card by default:** apply your card chrome around `.resource-bar`. The component stays bare so the choice is yours.

---

## Print rules

The legend is canonical, so the printed page is complete even when the browser drops the track and swatch backgrounds to save ink (most do) — the breakdown survives as bordered swatch rows of label + share. The one rule is `break-inside: avoid` so the component doesn't split across a page break.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
