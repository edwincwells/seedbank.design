# verification-status — testing notes

Tested against the three criteria in `TESTING.md`.

**Tested in:** _TBD — Edwin, Chrome on macOS._

---

## 01 — Renders correctly without CSS loaded

**TBD.**

Method: delete the stylesheet `<link>` tags and the reference-page `<style>` block in dev tools. Expected behaviour to confirm:

- Each state reads as its label plus (where present) the rationale link — "Verified audit", "Self-reported source", "Not disclosed". Fully meaningful; no state's meaning was carried only by colour.
- The verified-without-link "Avoid" case reads as plain "Verified" — the label is honest text even with no styling.
- The rationale links remain real, clickable links.
- The composition-preview eco-stat still reads as label / value / unit beside its verification.

---

## 02 — Works at 200% zoom

**TBD.**

Method: 200% zoom in Chrome. Expected behaviour to confirm:

- Badge pills and their rationale links scale together and hold a shared baseline.
- The gap between badge and link stays proportional; they don't collide.
- No clipping or horizontal scroll.

---

## 03 — Works on a 320px viewport

**TBD.**

Method: 320px width via responsive mode. Expected behaviour to confirm:

- Each verification wrapper holds on one line where space allows; the link wraps below the badge gracefully when not, without breaking the baseline alignment badly.
- No horizontal scroll.

---

## Edge cases to check during testing

- **Verified WITH link → green.** Confirm the success tint renders only when the link is present.
- **Verified WITHOUT link → neutral.** The core enforcement: confirm a verified wrapper with no `<a>` shows the neutral badge, not green. No unearned green anywhere.
- **`:has()` absent (safe degradation).** Simulate by testing the verified-with-link case in a context without `:has()` support (or reason about it): it should fall back to neutral, i.e. under-claim. Confirm there is no path to green without the link.
- **Self-reported colour independent of link.** Confirm both self-reported examples (with and without a source link) render the same amber.
- **Not-disclosed neutrality.** Confirm it's the plain neutral badge — no tint, hairline border present, label legible.
- **Dark mode.** Toggle `prefers-color-scheme`. Confirm the verified green and self-reported amber tints read against the dark surface (22% tint), text uses the dark functional tokens, and not-disclosed's neutral reads correctly.
- **Print preview.** Confirm tints drop to transparent, the labels still differentiate the states, the pill borders print, and the rationale links remain.
- **Focus ring on the rationale link.** Tab to a link; confirm the seedbank focus ring renders (foundation fix is in place).
- **Composition with eco-stat.** Confirm the verification sits cleanly beside the stat without collision.

---

## Open questions for later

- **"Estimated" as a fourth state** — deliberately excluded (it's eco-stat's uncertainty axis). Re-confirm when building provenance that no stage needs an "estimated" verification distinct from "self-reported".
- **not-disclosed and the deferred data-gap primitive** — not-disclosed is the verification-axis gap. When the standalone data-gap treatment lands, check whether the two should share styling.
- **Rationale-link affordance consistency** — settle (with methodology's source link) whether ecological rationale/source links want a consistent treatment across the layer.
- **Badge tint duplication** — the two tinted states mirror badge's functional-variant formula. If badge's tint percentages ever change, this file must follow. Worth a note in the adapting guide's SVG/tint-exception section, or a check when badge is next touched.

---

*Tested against TESTING.md criteria. Receipt only — the spec is in TESTING.md.*
