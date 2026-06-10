# links — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 11 May 2026.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by removing all three stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` in Chrome dev tools. With no CSS applied, every link retained browser-default styling — underlined and coloured. The `↗` external glyph disappeared (expected — it's a CSS-generated pseudo-element), but `rel="external"` links remained identifiable from their text content. Unhrefd `<a>` elements rendered as plain text with no link affordance. No content hidden, no structural meaning lost.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome, link colour, underline weight, and offset remained proportional and legible. The 1px default underline scaled cleanly; `text-decoration-skip-ink` still broke around descenders correctly. Hover state (2px thickness, 3px offset) remained perceptible. The `↗` external glyph scaled with surrounding text and stayed attached to its preceding word. Focus ring from `focus-ring.css` remained visible and proportional. No clipping or overlap on the standard reference page.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width via Chrome dev tools responsive mode, inline links wrapped correctly mid-sentence without breaking layout. Long URLs in external links wrapped within their container — no horizontal scroll. The non-breaking space before the `↗` glyph held; the arrow stayed attached to the link's last word. Raised-surface card example presented links legibly. No horizontal scroll anywhere on the page.

---

## Edge cases noted during testing

**Descender clearance.** `text-decoration-skip-ink: auto` is breaking the underline cleanly around `g`, `p`, `y`, `q` in default state. On hover, the 2px rule sitting 3px below the baseline gives enough breathing room that descenders no longer crowd the underline. The earlier issue (visible in pre-fix screenshots) is resolved at both default and 200% zoom.

**Hover thickness.** The 1px → 2px shift combined with the 2px → 3px offset bump is perceptible at default zoom. The two changes reinforce each other — thickness alone was subtle, offset alone was odd, the pair reads as a clear state change.

**Visited state.** Browser restrictions on `:visited` mean only colour shifts apply. The indigo `--link-visited` is perceptibly distinct from the default blue `--link` on both light and dark surfaces, and pulled far enough from `--make` (system purple) that the categorical signal stays clean.

**Multi-value `rel` attribute.** The `~=` selector matched `rel="external noopener"` correctly. The `↗` glyph rendered as expected.

**Links inside `strong`, `em`, `small`, headings.** All inherited weight, style, and size from context. Underline and colour applied without conflict. The heading case is the strongest test — link styling at h3 size with medium weight reads correctly and remains an obvious affordance.

**Links on `--surface-raised`.** Link colour passed contrast against the raised card background. No override needed.

**Unhrefd `<a>`.** Rendered as plain text — no underline, no colour. The `a[href]` selector scoping is working as intended.

---

## Print check (optional fourth criterion)

**Pass.**

Tested via Chrome's print preview (Cmd+P). URLs expanded after link text in body prose, in `--text-xs` size and `--text-subtle` colour. In-page anchors (`#hover-target`, `#focus-target`) correctly suppressed URL expansion. Links inside headings did not expand. External links showed the URL in place of the `↗` glyph — confirms the print-rule override is taking precedence over the external indicator. Long URLs wrapped via `word-break: break-all` without breaking surrounding text flow.

---

## Token-fallback fix (9 June 2026)

A foundation-level issue surfaced during `verification-status` testing: rationale links rendered near-black and illegible in dark mode when placed outside a prose element (links inside `<p>` were fine).

**Root cause.** `--link` and `--link-visited` were absent from the committed `tokens.css`, and `links.css` referenced them with no fallback (`color: var(--link)`). The undefined custom property invalidated the colour declaration, so links inherited their container's text colour — light `--text` inside prose (legible), but the body's near-black default in a bare container (illegible on the dark surface). The third instance of the `--border-focus` class of silent-token failure.

**Fix.**
1. `tokens.css` — defined `--link` / `--link-visited` with light/dark pairs (`--link` = the `--info` blue family; `--link-visited` a distinct indigo).
2. `links.css` — added fallbacks: `color: var(--link, var(--info))` on the base link and `color: var(--link-visited, var(--make))` on visited, per the "Foundation token references carry fallbacks" convention.

**Re-test (Edwin, Chrome on macOS, 9 June 2026).** Links read legibly in both light and dark mode. Confirmed across a prose link, a bare (non-prose) link, and a visited link — so the fix holds system-wide, not just in the component where it surfaced.

**Contrast verification (computed, 10 June 2026).** The candidate `--link-visited` values were checked against both surfaces and against the neighbouring tokens:

- AA: `#4a3a99` on `#fafaf7` = **8.52:1**; `#a99ce6` on `#1a1a1a` = **7.12:1**. Both pass AA (and AAA) for link-size text.
- Distinct from `--link` (blue): ΔE76 ≈ 27.6 (light) / 24.8 (dark) — clearly separable.
- Distinct from `--make` (purple): ΔE76 ≈ 9.2 (light) / 8.1 (dark) — **marginal**. They pass contrast and won't be mistaken for the blue link, but visited-link and a `--make`-coloured element sit close in the same purple family. This read clean in the 11 May visual test (links and purple content rarely sit adjacent); if hard categorical separation matters, nudge the indigo bluer — raising ΔE from `--make` while staying clear of `--link`.

Backlog item (m) is resolved on the AA question (pass). The `--make` proximity is a judgement call, not a blocker.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
