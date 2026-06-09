# verification-status — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** Chrome on macOS by Edwin, 9 June 2026.

A finding surfaced during testing (rationale-link colour in dark mode) and was traced to a foundation-level token gap, fixed in `tokens.css` and `links.css` rather than in this component. See the labelled section below. The receipts here reflect the post-fix state.

---

## 01 — Renders correctly without CSS loaded

**Pass.**

With the stylesheet `<link>` tags and the reference-page `<style>` block removed:

- Each state read as its label plus (where present) the rationale link — "Verified audit", "Self-reported source", "Not disclosed". No state's meaning was carried only by colour.
- The verified-without-link "Avoid" case read as plain "Verified" — honest text even unstyled.
- The rationale links remained real, clickable links.
- The composition-preview eco-stat still read as label / value / unit beside its verification.

---

## 02 — Works at 200% zoom

**Pass.**

At 200% zoom: badge pills and their rationale links scaled together on a shared baseline, the gap between them stayed proportional with no collision, and there was no clipping or horizontal scroll.

---

## 03 — Works on a 320px viewport

**Pass.**

At 320px: each verification wrapper held on one line where space allowed and the link wrapped below the badge gracefully when not, with no horizontal scroll.

---

## Edge cases checked

All passed.

- **Verified WITH link → green.** The success tint renders only when the link is present.
- **Verified WITHOUT link → neutral.** The core enforcement holds: a verified wrapper with no `<a>` shows the neutral badge, not green. No unearned green anywhere.
- **Safe degradation direction.** The green is an upgrade via `:has(a)`, so the no-`:has()` and no-CSS paths under-claim (neutral) rather than over-claim — there is no path to green without the link.
- **Self-reported colour independent of link.** Both self-reported examples (with and without a source link) render the same amber.
- **Not-disclosed neutrality.** Plain neutral badge — no tint, hairline border present, label legible.
- **Dark mode.** Verified green and self-reported amber tints (22%) read against the dark surface, text uses the dark functional tokens, not-disclosed's neutral reads correctly, and — after the foundation fix below — the rationale links read legibly in both modes.
- **Print.** Tints drop to transparent, labels still differentiate the states, pill borders print, rationale links remain.
- **Focus ring on the rationale link.** Tab to a link shows the seedbank focus ring.
- **Composition with eco-stat.** Verification sits cleanly beside the stat without collision.

---

## Finding during testing — rationale-link colour in dark mode (foundation fix)

**Symptom.** In dark mode, the rationale links in the demo rows rendered near-black and illegible, while the link in the composition-preview row (inside a `<p>`) was light and legible. Light mode was fine.

**Not a verification-status bug.** `links.css` colours every `a[href]` globally, so the cause was not a context/scoping issue in this component. A console check confirmed the root cause:

```
getComputedStyle(document.documentElement).getPropertyValue('--link-visited')  // → ""
```

`--link` and `--link-visited` were absent from the committed `tokens.css`. `links.css` referenced them with no fallback, so the colour declaration was invalid and dropped — links inherited their container's text colour. In a `<p>` that's prose's light `--text` (legible); in a bare flex container there was no light colour to inherit, so links took the body's near-black default and went illegible on the dark surface. The third instance of the `--border-focus` class of bug (a missing token silently disabling a feature).

**Fix (foundation, not this component):**
1. Defined `--link` and `--link-visited` in `tokens.css` section 2 with light/dark pairs (`--link` reuses the `--info` blue family; `--link-visited` is a distinct indigo).
2. Added fallbacks in `links.css` — `var(--link, var(--info))` and `var(--link-visited, var(--make))` — per the "Foundation token references carry fallbacks" convention, so a fork dropping the tokens degrades gracefully.

**Interim component patch — tried and reverted.** A `.verification a { color: var(--link, var(--info)) }` override was added first, then reverted: it only fixed this component and overrode `links.css`'s deliberate visited affordance. The foundation fix is the correct layer, and `verification-status.css` now relies on it with no link-colour rule of its own.

**Re-test (post-fix).** Rationale links read legibly in both light and dark mode, in the demo rows and the composition preview. Confirmed across a prose link, a bare link, and a visited link, so the fix holds system-wide, not just here.

---

## Open questions for later

- **"Estimated" as a fourth state** — deliberately excluded (it's eco-stat's uncertainty axis). Re-confirm when building provenance that no stage needs an "estimated" verification distinct from "self-reported".
- **not-disclosed and the deferred data-gap primitive** — not-disclosed is the verification-axis gap. When the standalone data-gap treatment lands, check whether the two should share styling.
- **`--link-visited` final values** — the indigo values added to `tokens.css` were candidates flagged for contrast verification; confirm the committed values pass AA and read distinct from `--link` (blue) and `--make` (purple).
- **Rationale-link affordance consistency** — settle (with methodology's source link) whether ecological rationale/source links want a consistent treatment across the layer.
- **Badge tint duplication** — the two tinted states mirror badge's functional-variant formula. If badge's tint percentages ever change, this file must follow.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
