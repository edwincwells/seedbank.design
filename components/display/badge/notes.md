# badge — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 23 May 2026.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` via Chrome dev tools. With no CSS applied:

- Every `<span class="badge">` fell back to inline text — readable, in document order, with label content intact
- The neutral default and the eight coloured variants were visually indistinguishable without CSS (expected — colour is a CSS effect)
- Adjacent labels in a cluster ran together without their visual chrome ("DraftArchivedToolsChildcare12 new", "GrowingUnder repairSharedIn progress", "SubmittedNeeds reviewNewFailed") — `inline-flex` gap is a CSS effect, but each label is still present and in document order. Same graceful-degradation pattern as the button no-CSS test
- Icons inside badges rendered at their natural SVG viewBox size — present, not hidden. The "Verified" / "Pending" / "Needs review" / "New" labels still sat alongside them, just below rather than aligned by the inline-flex layout
- Table cells containing badges remained aligned in their columns via UA table styling

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome:

- All badge variants remained readable and proportional
- Icon-and-label gap scaled correctly — icons did not visually merge with their labels
- Pill radius remained pill-shaped at zoom — `--radius-pill` (9999px) scales correctly without flattening
- Badges in the demo card rows did not push the rows out of alignment
- Badges in the demo table cells remained inside their cells without overflow
- Tint backgrounds remained visibly distinct from the surface in both card-on-surface and table-cell contexts

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width (Chrome dev tools responsive mode):

- Badge clusters wrapped onto multiple lines via flex-wrap without horizontal scroll
- The demo cards (badge on the right) reflowed cleanly when the row was too narrow for title + badge on one line
- The demo table held its columns without horizontal scroll at this width
- Badges with longer labels ("Under repair", "Needs review") did not overflow their containers
- `white-space: nowrap` on the badge held individual badges on a single line — only the surrounding cluster wrapped between badges

---

## Edge cases noted during testing

**Default neutral badge on `--surface-raised`.** The second demo card block exercises this case. The hairline border in `--border` gives the neutral badge clear definition against the matching card background — visibly differentiated without competing with the card's own visual weight.

**Coloured variant text at `--text-xs` (11px).** All eight tinted variants remained readable at the small size in both light and dark modes. `.badge-warning` and `.badge-make` — the variants flagged as potentially trickiest — held AA contrast in both modes without standing out as either washed-out or overly saturated.

**Two badges side-by-side.** Without the demo-cluster gap, badges sit directly adjacent. The component handles its own padding correctly — the demo class wasn't doing structural work the component depends on.

**Hairline border rendering.** The 0.5px hairline on the neutral default rendered as expected. Did not read as visually heavier than the surrounding card hairlines.

**Dark mode.** Verified separately via system preference. The 22% mix ratio gave all eight tinted variants enough lift off `--surface-dark` while preserving the subtle character. No variant disappeared or shouted.

**Hover behaviour.** Badges are non-interactive — no cursor change, no background change on hover. Confirmed.

**Wrapped `<a>` focus.** Tab to the link-wrapped badge — focus ring renders around the link, not the badge. The badge component stays out of the focus model, as designed.

---

## Open items for future iterations

- **Badge inside a button** (count indicator like "Notifications (3)" inside a button) — not in the reference page; worth a real-content test if/when this composition appears in a downstream pattern or display component.
- Print preview check, as flagged in prior components — still optional, still worth promoting to the standing checklist if the habit holds.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
