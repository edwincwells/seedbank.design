# dividers — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** _TBD — fill in browser, OS, tester, date_

---

## 01 — Renders correctly without CSS loaded

_TBD._

Method: open `dividers.html` in the browser, then in dev tools delete the three stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>`. Confirm `<hr>` elements remain visible as the browser's default chiselled separator, that they retain their role as breaks between sections, and that the demo's `<p class="meta">` labels are still legible (they degrade to default paragraph rendering). The asymmetric margin and hairline weight are CSS effects and will not survive — that is expected.

---

## 02 — Works at 200% zoom

_TBD._

Method: zoom to 200% in Chrome. Confirm dividers scale proportionally with the surrounding content, the asymmetric margin remains visually correct, and no clipping or overlap occurs. The hairline should remain a visible single line at this zoom level; if it disappears or becomes inconsistent across browsers, note it.

---

## 03 — Works on a 320px viewport

_TBD._

Method: in Chrome dev tools responsive mode, set viewport to 320px wide. Confirm dividers span the full container width, no horizontal scroll appears, and the demo regions still hold their content without overflow. The `<code>` references inside the "wrong use" examples should wrap correctly.

---

## Edge cases to check

_TBD — fill in as encountered during testing._

Suggested:

- Hairline visibility at 100% on a high-DPI display vs a low-DPI display (0.5px renders inconsistently across pixel densities)
- Hairline visibility in dark mode — the dark-mode `--border` value is `#3a3a3a` on `#1a1a1a` surface; check it remains visible
- Margin collapsing between an `<hr>` and an adjacent `<h2>` — confirm the rhythm reads correctly when both have vertical margins
- Print preview — confirm `page-break-before: avoid` keeps the divider with its preceding content

---

## Open questions

_TBD — fill in if testing surfaces any._

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
