# provenance — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 9 June 2026.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

With the stylesheet `<link>` tags and the reference-page `<style>` block deleted in dev tools, the chain rendered as a **numbered ordered list**, stages in sequence (1–5) — the order, which is the core meaning, was intact. Each stage read as location, distance, detail, and verification text + rationale link; nothing was carried only by the rail. The undisclosed stage still appeared, with "Not disclosed" and a "—" distance — the gap was visible, not omitted. The composed eco-stat on the weaving stage read as label / value / range. The header read as title + summary. The rail (dots + connectors) disappeared entirely, confirming nothing essential lived only in it.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom the rail scaled with the text — dots stayed aligned to the title line and connectors still bridged dot-to-dot with no drift. Location and distance held their shared baseline; long locations wrapped without colliding with the distance. The composed verification badges and the eco-stat scaled cleanly. No clipping, no horizontal scroll.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px width the gutter and rail held, and stage content fit without horizontal scroll. Long locations ("Retting & spinning — Belgium") wrapped cleanly, with the distance dropping below the location rather than forcing the row wider. Verification wrappers (badge + rationale link) wrapped gracefully. The composed eco-stat held within its stage.

---

## Edge cases — all confirmed

- **Rail optical alignment** — the dot at `--prov-offset: 0.45em` read as aligned with the first line of each stage title at the default size; no nudge wanted.
- **Connector meets the next dot** — the hairline connector reached the next dot with no gap and no overshoot at the shipped spacing.
- **Last stage has no connector** — the trailing line was correctly absent on the final stage.
- **Undisclosed stage** — the not-disclosed verification rendered as the neutral badge and the "—" distance read as a gap.
- **Composed eco-stat in a stage** — the compact eco-stat sat cleanly within the weaving stage without disrupting the rail.
- **Bare chain (no card)** — the second demo (no card wrapper) read correctly on the page surface, confirming the component ships legibly without chrome.
- **Dark mode** — the neutral rail (dots + connectors) read against the dark surface; distances and details stayed legible; composed verification badges and links read correctly (post the --link/--link-visited foundation fix).
- **Print** — stages did not split across a page break (`break-inside: avoid`), the rail printed legibly, and the rationale links expanded to their URLs.
- **Focus ring on rationale links** — the seedbank ring appeared on each stage's link when tabbed through.

---

## Open questions for later

- **Per-leg vs cumulative distance** — the demo mixes per-leg distances with a "2,410 km total" on the final stage; read clearly here, but worth revisiting whether the total belongs only in the header summary for longer chains.
- **eco-stat composition depth** — only one stage carries an eco-stat here. If real chains commonly attach a quantity to every stage, revisit whether a denser layout is wanted.
- **Rationale-link affordance** — provenance, verification-status, and methodology all use `rel="external"` evidence links; settle a consistent treatment across the layer (backlog item l).
- **Print preview** — optional fourth check; the `break-inside` behaviour was confirmed in print emulation.
- **Composition with an action** — provenance ships without an action footer by design. When the dashboard/card composition work happens, confirm the data+action pairing (e.g. "request disclosure" beside the undisclosed stage) reads well.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
