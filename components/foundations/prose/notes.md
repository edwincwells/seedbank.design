# prose — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 11 May 2026.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the three stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` in Chrome dev tools. With no CSS applied, every heading retained its hierarchy via browser defaults — h1 largest, h6 smallest. Paragraphs separated naturally. strong rendered bold (browser default), em rendered italic. h6 reverted to a small heading without the uppercase eyebrow styling — semantic remained clear. Article paragraphs were indistinguishable from regular paragraphs (expected — leading is a CSS effect). No content hidden, no structural meaning lost.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome, every heading remained proportional and readable. The 65ch paragraph max-width scaled with the font size, remaining ~65 characters per line. Line spacing remained comfortable. No clipping, no overlap, no horizontal scroll on the standard reference page.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width (tested via Chrome dev tools responsive mode), paragraphs filled the available width — the viewport was narrower than 65ch, so the max-width constraint correctly stopped applying. Headings did not overflow horizontally. No horizontal scroll appeared. Long inline content (the inline `<code>` references) wrapped correctly.

---

## Edge cases noted during testing

**First-child margin handling.** The page-top h1 sat flush against the body padding with no top margin — `:first-child` selector working as intended.

**Last-child margin handling.** The final paragraph in the "real prose" article had no extra space below it — `:last-child` selector working as intended.

**Article leading.** Paragraphs inside the `<article>` element had visibly looser leading than paragraphs outside it. The difference between 1.55 (base) and 1.7 (loose) is subtle but readable.

**Max-width on narrow viewports.** At 320px, the 65ch constraint was wider than the viewport, so paragraphs filled the viewport rather than stopping early. Correct behaviour — `max-width` is a ceiling, not a target.

**Inline `<code>` rendering.** The reference page uses inline `<code>` in a few places. Without a `display/code` foundation yet, these render in browser-default monospace at the inherited size. Acceptable for the demo; flagged for when `display/code` lands.

---

## Open questions for v0.2 and beyond

- The `.prose-long` class is documented but never used in the reference page. Worth a real example once we have one (perhaps in `docs/`).
- The 65ch max-width may need a per-context override pattern when component CSS starts being written. Track whether forks need to override it often.
- Print rules for blockquotes, lists, and other elements not in this foundation will need adding to those foundations' CSS when they're built.
- Print preview check was not run as part of this test — optional fourth check, worth running before v1.0.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
