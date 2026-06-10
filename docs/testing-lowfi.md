# Testing your fork (low-fidelity)

`TESTING.md` at the repo root holds the canonical acceptance bar — the three criteria every component must pass. This guide is the practical *how*: how to run those checks, what a pass looks like, and — the part that matters most for a fork — how to catch the things that break **silently** when you edit tokens or components.

It's low-fidelity on purpose. The whole premise of "designed for the edges" is that the checks that matter can be run in a browser with no tooling, no CI, no test runner. If you can open devtools, you can validate a fork.

The reference environment is Chrome on macOS, but the methods are browser-generic except where noted.

---

## The three-criterion bar

This is the floor every component already clears, and the bar your changes must keep clearing. `TESTING.md` is canonical; here's how to actually run each one.

### 1. Renders correctly without CSS

**How:** disable styles entirely — devtools can do it, or use the browser's reader view, or temporarily comment out the stylesheet links.

**Pass condition:** the page is still *readable and ordered*. Content makes sense top to bottom, nothing relies on CSS to convey meaning. An eco-stat still reads as a value and a range; a resource-bar still reads as a legend of labels and shares; provenance still reads as a numbered list of stages. If removing CSS removes *information* (not just styling), that's a fail — the meaning was riding on presentation.

### 2. Works at 200% zoom

**How:** Cmd/Ctrl-`+` until the browser reports 200%.

**Pass condition:** nothing clips, overlaps, or gets cut off. No content becomes unreachable. Controls stay operable and focus rings stay visible. Text reflows rather than forcing a horizontal scrollbar.

### 3. Works at a 320px viewport

**How:** devtools responsive mode, set width to 320px.

**Pass condition:** no horizontal overflow — with one deliberate exception, `.table-wrap`, which is *meant* to scroll horizontally on narrow viewports (and carries `tabindex="0"` so it's keyboard-scrollable). Everything else reflows. Touch targets stay large enough to hit.

---

## Why the floor is set here

The three criteria are a floor, not a ceiling — but they're a *well-chosen* floor. No-CSS, 200% zoom, and 320px aren't arbitrary; they're proxies for the real deployment conditions this system targets: e-ink, print, low bandwidth, screen readers, old or constrained devices, solar-powered servers that drop assets. A component that survives all three has demonstrated graceful degradation, which is the design brief, not an afterthought. The system is full of worked proof it's achievable — buttons whose loading labels carry meaning without the spinner, the entire JS-free ecological layer — so the bar is a floor you should expect to clear, not a stretch goal.

---

## The optional fourth: print

Print preview is currently an optional fourth criterion (`TESTING.md` flags the open question of whether to promote it for v0.2).

**How:** browser print preview (Cmd/Ctrl-`P`), check both light output and the page breaks.

**Pass condition, where it applies:** methodology disclosures force *open* in print (a closed `<details>` would hide its content on paper); provenance stages don't break across a page boundary (`break-inside: avoid`); a resource-bar legend survives even if the track doesn't print; table headers repeat on each page. The ecological layer exercised print deliberately, so the habit is there — if you add components, matching it is worth the small effort.

---

## Beyond the floor

Other concerns the three criteria don't cover, and when to check them.

### Keyboard navigation

**How:** tab through the page. Every focusable element should show a visible focus ring; tab order should match reading order.

**The macOS gotcha that hides bugs — read this.** macOS Chrome (and Safari) do **not** tab to non-form focusable elements — links, `<summary>`, anything with `tabindex` — unless **System Settings → Keyboard → "Keyboard navigation"** is turned on. With it off, you'll tab only between form controls and conclude focus works fine. This is exactly what masked the focus-ring regression: the ring was broken on *everything*, but the only elements being tabbed to were form controls, and the earlier receipts predated the break. **Turn this setting on before keyboard-testing**, then tab to a link, a `<summary>`, and a `.table-wrap` — not just inputs.

### Screen reader

**How:** run VoiceOver (Cmd-F5 on macOS) over the component.

**Pass condition:** the aria-hidden *visuals* are silent — eco-stat's band, provenance's rail, resource-bar's track should announce nothing, because the text beside them already carries the meaning. The canonical text *is* announced. Form associations work: `aria-invalid` is conveyed, `<fieldset>`/`<legend>` groups read as a group, `aria-describedby` help/error text is announced with the control.

### Dark mode

**How:** devtools → Rendering → "Emulate CSS prefers-color-scheme: dark" (no need to change your OS theme).

**Pass condition:** the dark token pairs hold contrast. This matters most *after a token edit* — recolouring `--grow` light without checking its dark pair is a common way to ship an unreadable dark mode.

### Colour-independence

**How:** devtools → Rendering → emulate a vision deficiency, or apply a grayscale filter.

**Pass condition:** meaning survives with no hue. This is the single best check for catching a fork that's accidentally made colour load-bearing — if a resource-bar segment or a verification state becomes ambiguous in grayscale, the text channel underneath isn't doing its job.

---

## Testing what your fork changed

This is the part no upstream test suite can do for you, because only you know what you edited. Most of these are checks for **silent** failures — the ones with no console error, visible only in the rendered result.

### After recolouring tokens

- **Contrast (AA).** Run your new values through a contrast checker against the surface they sit on. The default palette was constrained to pass AA; your recolour isn't, until you've checked it.
- **Dark pair.** Check the dark value, not just the light one (see Dark mode above).
- **Grayscale.** Confirm nothing you recoloured now relies on hue alone.

### After deleting or editing tokens — the silent-failure checks

These are the two failure modes the project has actually hit. Both are invisible in the console.

- **Missing token (absent → declaration dropped).** If you removed any token, keyboard-test the focus ring specifically — it's the thing that silently vanished last time — and check links *outside* a prose block in dark mode (the other thing that silently broke). If your foundation CSS references carry fallbacks, these degrade visibly instead of disappearing; if you've written new references, confirm they have fallbacks.
- **Shadowed token (name reused → resolves to the wrong thing).** If you added any inline/presentational custom property, confirm its name doesn't collide with a token — especially an activity token (`--grow`, `--repair`, `--share`, `--make`). A segment that paints *nothing* is the signature of this bug. Prefix presentational properties (`--portion`, `--at`, …) so they can't collide.

### After re-encoding an SVG

If you changed `--surface-light`, `--text-light`, or `--text-dark`, you had to re-encode the hardcoded hex in the checkbox checkmark and select chevron data-URIs (see `adapting.md`). **Visually confirm both render in the right colour in both light and dark mode** — a wrong re-encode shows up as an invisible or mis-coloured checkmark/chevron, and nothing will warn you.

### After adding a component

Run the full three-criterion bar on it, then confirm the two conventions that keep new work from breaking silently: no presentational custom property reuses a token name, and every foundation token reference carries a fallback.

---

## A minimal pre-commit checklist

Keep this near your commits. It's the floor plus the silent-failure checks, nothing more.

```
[ ] Renders readable with CSS disabled
[ ] No clipping / overflow at 200% zoom
[ ] No horizontal overflow at 320px (except .table-wrap)
[ ] Keyboard: focus ring visible on a link, a <summary>, and an input
    (macOS: "Keyboard navigation" enabled in System Settings first)
[ ] Dark mode holds contrast (emulate prefers-color-scheme: dark)
[ ] Meaning survives grayscale
If you edited tokens:
[ ] New colours pass AA on their surface, light and dark
[ ] Removed a token? focus ring + non-prose links still work
[ ] Added a presentational --property? name doesn't collide with a token
[ ] Re-encoded an SVG? checkmark/chevron render correctly, light and dark
```

---

*Code is MIT; this guide is CC BY-SA 4.0. The acceptance bar itself lives in `TESTING.md` at the repo root — this guide is its practical companion, not a replacement.*
