# lists — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 12 May 2026.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

Tested by deleting the four stylesheet `<link>` tags and the reference-page `<style>` block from the page's `<head>` in Chrome dev tools. With no CSS applied:

- `<ul>` items rendered with browser-default disc bullets — list structure intact
- `<ol>` items rendered with browser-default decimal numerals — mono styling lost, as expected
- Nested lists retained their nesting via UA indentation, with the UA's marker cascade taking over (disc → circle → square; decimal). Acceptable — the semantic nesting is preserved
- `<dl>` retained the term/definition relationship — `<dt>` on its own line, `<dd>` indented via UA default
- The wrong-use blocks degraded to plain paragraphs; structure remained legible

The 65ch max-width, custom 24px indent, mono numerals on `<ol>`, and the 4px/8px item gaps were all lost — expected, all CSS effects. Semantic structure of every list type intact.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom in Chrome:

- All three list types remained readable and proportional
- Markers stayed visible in the gutter — no clipping at the start of any line
- The 24px indent scaled correctly with the rest of the layout
- Nested lists remained visually distinguishable from their parent level — each level's indent compounding clearly
- `<ol>` mono numerals remained legible and aligned at the larger size

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px viewport width (Chrome dev tools responsive mode):

- All three list types fit without horizontal scroll
- Markers did not clip at the left edge — the 24px indent gave adequate room
- The three-deep nested `<ul>` example fit without pushing content off the right edge, though the third level sat close to the right margin. Acceptable but a real-world content note: deep nesting at narrow viewports is tight by definition
- `<dl>` pairs remained legible; long definitions wrapped correctly within their indented column
- The mixed `<ol>`-inside-`<ul>` example handled the marker change at the nested level without layout issues

---

## Edge cases noted during testing

**`::marker` font rendering — mono numerals confirmed.** The `--font-mono` family applied to `<ol>` numerals in Chrome on macOS as expected. The `--text-sm` step-down and `--text-muted` colour also applied. The numerals read as anchored identifiers without competing with the body text — the design intent works.

**Mono numeral alignment.** Single-digit and double-digit items in the same list aligned correctly at their start position. The mono digits sit as a left-aligned column, which is the intended effect — proportional digits would shift slightly between widths.

**`<dl>` adjacent-sibling separator.** The `dd + dt` selector produced the correct 12px top margin between pairs. Only tested with the `<dt> <dd> <dt> <dd>` pattern; the multi-`<dd>`-under-one-`<dt>` pattern allowed by HTML spec is not in the reference page — worth flagging if it comes up in real content.

**Long-form list spacing inside `<article>`.** The 8px item gap and 1.7 line-height combined into a coherent rhythm — visibly more spacious than the UI list above the article, matches the prose foundation's looser-article pattern.

**Nested list margin reset.** No double-gap above any nested list in either the three-deep `<ul>` example or the mixed `<ol>`-in-`<ul>` example. The `li > ul, li > ol` rule is doing its job.

**Dark mode.** Mono numerals in `--text-muted` (which resolves to `#b0b0b0` in dark mode) remained legible against the dark surface. All list types read correctly in both modes.

**Print preview not run.** Optional fourth check, deferred. `page-break-inside: avoid` on `<li>` and `page-break-after: avoid` on `<dt>` are present in `lists.css` but their behaviour was not verified in this round.

---

## Open questions for v0.2 and beyond

- **Deep nesting at narrow viewports.** Three-level nesting at 320px is tight by definition. Not a bug, but worth noting in adaptation guidance — communities producing dense content at small viewports may want to reduce `--space-5` to `--space-4` for the indent.
- **Multi-`<dd>` definition pattern.** HTML allows multiple `<dd>` elements under one `<dt>` (e.g. a term with multiple definitions). The current `dd + dt` selector handles this correctly by accident — subsequent `<dd>`s won't get a top margin. Worth a deliberate test case in v0.2.
- **Older-browser `::marker` fallback.** Confirmed working in current Chrome. Not yet tested in older Safari, Firefox ESR, or any e-ink browser. If mono numerals are a load-bearing brand signal in a particular fork, those forks should test their target browsers explicitly.
- Print preview check, as flagged for dividers — still optional, still worth adding to the standing checklist once a habit is established.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
