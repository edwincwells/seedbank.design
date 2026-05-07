# Design Principles

These three principles govern how things built with seedbank.design should look, behave, and communicate. They are design tiebreakers — when a component decision is contested, these are what get cited.

They are shipped as a design system asset. Forks inherit them. Adaptations should be conscious, not accidental — if your context requires dropping or modifying a principle, document why.

---

## 01 — Designed for the edges, not the centre

The system must work on e-ink, print, low bandwidth, and solar-powered servers — and be comprehensible without design or technical literacy. Labels alongside icons, plain language over convention, explicit states over implied ones. Graceful degradation is a first-class concern, not an afterthought.

Mainstream design systems optimise for the centre: fast connections, capable devices, design-literate users. seedbank.design optimises for the edges. This is not a constraint to work around — it is the design brief.

**Concrete implications:**
- No gradients, no decorative effects, no font weights that disappear on cheap screens
- Every component is tested in print mode and at low bandwidth before release
- Icons always paired with visible labels — never icon-only
- States are explicit and named, not implied by colour alone
- Documentation is written for non-designers first

---

## 02 — Honest uncertainty over false precision

Ecological and provenance data is inherently uncertain. Displaying a clean number implies confidence the underlying data doesn't warrant. Uncertainty is information — ranges, methodology links, and explicit data gaps are first-class UI elements, not disclaimers buried in footnotes.

A component that shows "2.3 kg CO₂e" on a clean progress bar is making a claim the data cannot support. The honest version shows the range, names the methodology, and surfaces what is unknown. This principle applies beyond ecological data: wherever the system displays information, it should represent what is known, what is estimated, and what is missing.

**Concrete implications:**
- Carbon components show ranges (1.8 ± 0.6 kg CO₂e), not point estimates
- Provenance chains show verification status at every stage — verified, self-reported, not disclosed
- Gaps in data are displayed as gaps, not omitted
- Methodology is always one tap or click away
- No green badges without linked rationale

---

## 03 — Collective agency, not individual surveillance

Data is aggregated, not per-user. And aggregated data is only worth showing if the community can act on it. Every indicator is paired with a decision it supports — energy generation paired with load-shifting, provenance gaps paired with sourcing alternatives. Information without agency is noise.

Ecological indicators that guilt individual users are ideologically incoherent with the communities this system is built for. The interesting design space is collective dashboards — a community's shared energy budget, a cooperative's material flows — where data builds shared understanding and supports shared decisions.

**Concrete implications:**
- No per-user or per-household breakdowns stored or displayed
- Anomalies trigger collective discussion, not individual alerts
- Every data component includes or links to an action
- Contribution metrics, if shown, reflect system health not individual output
- Aggregate data is labelled explicitly as such

---

*seedbank.design — licensed under MIT (code) and CC BY-SA 4.0 (documentation and design assets). This file is a design system asset — it ships with every clone. Forks are encouraged to adapt it; adaptations should be documented.*
