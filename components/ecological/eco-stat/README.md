# eco-stat

The honest value display. A single quantity — carbon, energy, water, a count, a percentage — shown with its uncertainty, never as a false-precise number.

This is the first component in the **ecological** layer and the primitive the rest of the layer composes. It is where Principle 02 (honest uncertainty over false precision) stops being a value and becomes a reusable mechanic.

---

## What it styles

- `.eco-stat` — the container
- `.eco-stat-label` — what the number is
- `.eco-stat-value` — the headline numeral (mono) and its unit (sans)
- `.eco-stat-range` — the uncertainty band, in words (the honest core)
- `.eco-stat-track` — optional band geometry (decorative, `aria-hidden`)
- `.eco-stat-scale` — optional rail endpoints (decorative, `aria-hidden`)
- `.eco-stat-compact` — density modifier
- `[data-unmeasured]` — the not-known state

It composes nothing below it directly, but the unit/label rhythm is built to sit inside cards and dashboard cells later.

---

## The honesty rules

These are the reason the component exists. They are not optional polish.

**The range is low–high, not ±.** Real ecological uncertainty is rarely symmetric — an estimate of 1.8 often sits inside a 1.2–2.4 band, closer to one end than the other. The data model is two explicit bounds. `±0.6` is only a display shorthand for the symmetric case; the general, honest form names both ends so it cannot hide skew.

**There is no point marker on the track.** The prototype drew a dot at the estimate. We don't. A dot at "1.8" reasserts exactly the precision the range exists to refuse. The band *is* the message: the answer is somewhere in here. The headline numeral already gives the best single estimate for those who want one.

**No uncertainty means no range — honesty runs both ways.** A genuine count ("14 households") carries no range line. Don't fabricate a range any more than you'd fabricate a point. Use the exact form (see below) when the value is actually exact.

**A missing value is shown, not hidden.** A stat with no data renders an explicit gap (`[data-unmeasured]`), never blank space and never a fabricated zero. The gap is information.

---

## Markup

```html
<div class="eco-stat">
  <p class="eco-stat-label">Carbon footprint — linen offcut</p>
  <p class="eco-stat-value">
    <data value="1.8">1.8</data>
    <span class="eco-stat-unit">kg CO₂e</span>
  </p>
  <p class="eco-stat-range">estimated <data value="1.2">1.2</data>–<data value="2.4">2.4</data> kg CO₂e</p>
</div>
```

With no CSS this reads as a complete line: *Carbon footprint — linen offcut / 1.8 kg CO₂e / estimated 1.2–2.4 kg CO₂e.* Label, value, and range all survive.

**Why `<data>`.** The visible text is for people; the `value` attribute carries the machine-readable number. They can differ in precision — `<data value="1834">1.8k</data>` shows a rounded figure while preserving the real one. It also gives forks a clean hook for scripts that read or aggregate stats without scraping rendered text.

---

## The optional track

The track is a **visual enhancement, not the component.** It depicts the band as a filled region between the low and high positions on a rail. It is `aria-hidden` (the range text already says everything), carries no colour-dependent meaning (so it holds up on e-ink and in print), and has no point marker.

```html
<div class="eco-stat-track" aria-hidden="true"
     style="--range-start: 30%; --range-end: 60%"></div>
<div class="eco-stat-scale" aria-hidden="true">
  <span>0</span><span>4 kg</span>
</div>
```

The two percentages are **data-derived geometry**, computed by the fork from its own scale (the rail's 0% and 100% ends). This is the one place the component takes inline values — the same category as a progress bar's width, not hardcoded styling. The scale endpoints are optional; use them only when the absolute bounds matter.

If you never need the visual, omit the track entirely. The text form is whole on its own.

---

## States and variants

| | Trigger | Effect |
|---|---|---|
| Default | _(none)_ | Label, value, range text |
| With track | add `.eco-stat-track` | Band geometry below the text |
| Exact | omit `.eco-stat-range` | Headline value with no range — for genuine counts |
| Unmeasured | `data-unmeasured` on `.eco-stat` | "—" placeholder, reason in the range line |
| Compact | `.eco-stat-compact` | Smaller value and type, for dashboard density |

**On `data-unmeasured` as an attribute, not a class.** State from attributes, per project convention. Compact, by contrast, is a *variant* (a presentation choice, like `.btn-sm`), so it stays a class. The distinction is the same one the rest of the system draws.

**On the "—" placeholder.** It is the long-standing data-table glyph for "no value here," not the decorative em-dash status prefix the system dropped. Different job: that one was a typographic flourish in front of status text; this one stands in for an absent number.

---

## Tokens used

| Token | Used for |
|---|---|
| `--font-mono` | Value and range numerals (tokenised-metadata convention) |
| `--font-sans` | Label, unit, qualifier words |
| `--text-xxl`, `--text-xl` | Value (default / compact) |
| `--text-md` | Unit |
| `--text-sm`, `--text-xs` | Label and range (default / compact) |
| `--text`, `--text-muted`, `--text-subtle` | Value / label+unit / range+gap colours |
| `--weight-medium`, `--weight-regular` | Value vs everything else |
| `--surface-sunken` | Track rail |
| `--border-strong` | Track band fill |
| `--border-hairline` | Print outlines |
| `--radius-pill` | Track rail and band |
| `--space-1`, `--space-2` | Internal rhythm |

No new tokens were added for this component, and no values are hardcoded. The track positions are per-instance data, not style.

---

## Accessibility

- The track and scale are `aria-hidden="true"`: they duplicate the range text, so announcing them would be redundant. The essential information — value, unit, range — is all live text.
- `font-variant-numeric: tabular-nums` keeps figures aligned, which matters in compact grids.
- No colour-only meaning anywhere. The band is conveyed by text and, secondarily, by position — never by hue. This is what lets it survive on e-ink and monochrome print.
- There is no focusable element in this component, so it does not depend on `focus-ring`.

---

## Composition notes

**No action footer here, by design.** Principle 03 (data paired with the decision it supports) is real, but it belongs at composition level — a card or dashboard pairs a stat with an action. Baking a footer into every `eco-stat` would mean a six-stat dashboard carried six footers. The primitive stays atomic; the pairing is the composer's job.

**Methodology disclosure is a separate component.** A stat that needs a "how this was calculated" affordance composes the `methodology` component (next in the layer) as a sibling. eco-stat ships without it so the primitive stays clean.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/prose/prose.css` (the stat uses `<p>` elements; prose sets their baseline, though eco-stat sets every property it needs explicitly and works without it)
3. Include `components/ecological/eco-stat/eco-stat.css`
4. Write the markup above. Supply low–high bounds whenever the value is an estimate; omit the range only when the value is genuinely exact.

---

## How to override

**To change the value size:** remap `--text-xxl` in `tokens.css` (system-wide) or override `.eco-stat-value` font-size locally.

**To colour the value by activity** (e.g. a grow-coloured headline in a cultivation context): set `color` on `.eco-stat-value` to an activity token. Resist colouring it by *sentiment* (green good / amber bad) — that was a prototype habit this layer dropped, and it pushes toward the surveillance framing Principle 03 refuses.

**To use ± display for a symmetric case:** that is purely a content choice — write `±0.6` in the range text. The component does not care; only your numbers do. Keep low–high for anything asymmetric.

**To drop the track project-wide:** simply never add the `.eco-stat-track` element. There is nothing to remove from the CSS.

---

## Print rules

- The text form prints unchanged.
- The track rail and band print as hairline outlines (no fills) so the uncertainty band stays legible on paper without spending ink.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
