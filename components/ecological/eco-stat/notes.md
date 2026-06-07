# eco-stat — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 7 June 2026.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the three stylesheet `<link>` tags and the reference-page `<style>` block in dev tools. With no CSS applied:

- Each stat read as a complete line — label, then value + unit, then range — via block defaults. No content hidden.
- The `<data>` numerals rendered as their visible text; the `value` attribute remained present in the DOM but invisible, as expected.
- The exact-value stat showed no range line — correct, there is none to show.
- The unmeasured stat showed the "—" placeholder and its reason; the gap stayed legible.
- The track and scale `<div>`s rendered as empty / inline fragments carrying no meaning — correct, they are `aria-hidden` decoration. Nothing essential was found to live only in the track.
- The compact grid collapsed to stacked blocks — density is a CSS effect, expected to be lost.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome:

- Value, unit, and range scaled proportionally and held a shared baseline.
- The track rail and band scaled with the container; the band kept its relative position and width.
- The compact grid reflowed without clipping or overlap.
- No horizontal scroll on the standard reference page.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width:

- Value + unit held on one line where space allowed and wrapped gracefully when not.
- The track spanned the available width without overflow.
- The compact grid dropped to two columns (the demo breakpoint) without horizontal scroll.
- Range text wrapped cleanly rather than forcing the row wider.

---

## Edge cases checked

All passed.

- **Negative and signed values** — the −23% compact stat rendered the minus sign cleanly in mono; tabular-nums held alignment.
- **Long unit strings** — "litres / day" wrapped beside the large numeral without breaking the baseline alignment.
- **Asymmetric track** — the water-use band (20–90%) read as visibly off-centre. The geometry reflects the asymmetry rather than centring — the low–high model lands visually, which was the point of choosing it over ±.
- **No point marker** — confirmed no dot on any track; the band alone carries the visual.
- **Dark mode** — toggling `prefers-color-scheme` swapped correctly via tokens, with no component-level dark rules. Value, label, and range contrast held, and the track rail and band both read against the dark surface.
- **Without prose.css** — loaded with only `tokens.css` and `eco-stat.css`; the stat held its shape (every property is set explicitly).

---

## Open questions for later

- **Track without scale endpoints** — the bare rail (no endpoints) is expected to be the common case; confirm it reads acceptably on its own when a real component omits the scale.
- **Compact value at `--text-xl`** — 18px read acceptably as the headline in the density check; revisit if a real dashboard wants a middle size.
- **Print preview** — optional fourth check, not run. Worth confirming the track prints as hairline outlines with the band still distinguishable.
- **Aggregation hook** — the `<data value>` attribute is intended as the script seam for the eventual dashboard/collective component. Re-confirm it carries the right precision when that component lands.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
