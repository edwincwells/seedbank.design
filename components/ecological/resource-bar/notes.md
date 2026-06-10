# resource-bar — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 10 June 2026.

Differentiation went through two revisions before this version: pure-neutral segments (unreadable — all grey), then a luminance ramp (read as a false ordinal/magnitude relationship). This version uses distinct activity-palette hues + matching swatches, the correct categorical encoding. A name collision was then fixed: the segment-width custom property was renamed `--share` → `--portion`, because `--share` is also an activity token, so `--resource-fill: var(--share)` had been resolving to the local 12% width instead of the blue token — the Comms segment painted nothing. The verdicts below are against the final (hue + `--portion`) version.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

With the stylesheet `<link>` tags and the reference-page `<style>` block deleted, the legend rendered as a readable list for each example (label + share), carrying the whole breakdown including the named Available / Remaining remainder. The swatches collapsed (aria-hidden, empty spans) with nothing essential lost — the label + share carried each row. The header read as title + summary, and the summary's "X of Y" / threshold text carried the target meaning. The track collapsed (empty, text-free segment divs), and the marker carried nothing essential. The source line read as plain text.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom the track scaled proportionally, segment widths held their shares, and the marker stayed at its `--at` position. The legend wrapped cleanly, swatches stayed aligned with their labels, and shares stayed legible. No clipping, no horizontal scroll.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px width the track held full width without overflow and thin segments stayed visible. The legend wrapped to multiple rows gracefully via flex-wrap; long labels ("Available before threshold") didn't force horizontal scroll, and swatches stayed with their rows. The header summary wrapped cleanly.

---

## Edge cases — all confirmed

- **All segments paint** — every used segment rendered, including the blue Comms segment that was missing before the `--portion` rename. No segment was invisible — the collision fix is confirmed.
- **Readable at a glance, no false relationship** — the three used segments read as distinct categories at a glance via hue, with nothing implying a ranking or progression the way the luminance ramp had.
- **Swatch join** — each legend row matched its bar segment by hue, instantly.
- **Available reads as available** — the neutral unfilled tail and its sunken swatch read as "empty / remaining", clearly apart from the coloured used segments.
- **Available swatch visibility** — the sunken-coloured swatch was delineated by its hairline border against the card and page surfaces.
- **Colour-blind / grayscale fallback** — under grayscale emulation the legend's label + share gave the full breakdown when the hues were hard to tell apart — the principle-01 fallback working as designed.
- **Target marker position** — the marker sat at `--at: 90%`, with its meaning carried by the summary text.
- **Single-segment progress** — the bare fund example (one segment, no marker, no card) read correctly on the page surface.
- **Dark mode** — the activity hues resolved to their dark values and stayed AA-distinct on the dark sunken track; the marker and swatches read; "Available" stayed distinct.
- **Print** — the component did not split across a page break (`break-inside: avoid`), and the legend carried the full breakdown even where the track and swatch backgrounds dropped.
- **No-overflow with shares + gaps** — segment shares plus `--seg-gap` cuts did not visually overrun the track; the proportions read true.

---

## Open questions for later

- **Overloading the activity palette** — watch whether a coloured segment sitting near a genuine activity-status element (badge, left-rule) causes cross-talk in real composed dashboards. If it does, that's the trigger to add a dedicated chart palette.
- **>4 series** — four hues cap the palette; confirm the "use a table beyond ~4" ceiling holds in practice, or revisit a dedicated palette.
- **Marker label affordance** — the threshold marker has no on-bar label by design (the summary carries it). Revisit if real use wants an inline label.
- **Print preview** — optional fourth check; the `break-inside` behaviour and "legend survives, track may drop" were confirmed in print emulation.
- **Composition with an action** — resource-bar ships without an action footer. When the dashboard/card composition work happens, confirm the data+action pairing (e.g. "shift load" beside a near-full budget) reads well.
- **Custom-property naming rule** — the `--share` → `--portion` collision (a presentational custom property reusing an activity-token name) is worth promoting to a BUILD_CONVENTIONS naming rule, since any future component mixing inline custom properties with token references could repeat it.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
