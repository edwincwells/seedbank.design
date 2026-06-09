# verification-status

How trustworthy a claim is — **verified**, **self-reported**, or **not disclosed**. The honest-uncertainty mechanic that labels the provenance of a piece of data, so a reader can tell a third-party-audited figure from an unverified supplier claim from a gap.

This is the third component in the **ecological** layer. It composes the `badge` display component, and `provenance` (the next component) composes it in turn for each stage of a chain.

---

## Why it's a component, not a pattern

verification-status would be a markdown pattern over `badge` if it were only "use these labels with these badge variants and remember to add a rationale link." What earns it CSS is making one rule **structural rather than documented**: principle 02's *no green badges without linked rationale*.

A "verified" badge turns green **only when it actually contains a rationale link** (`:has(a)`). A verified claim with no linked evidence stays the neutral badge — the system withholds the green it hasn't earned. An author cannot ship a bare green "Verified"; the CSS refuses to render it. That enforcement is the component's reason to exist.

And it degrades in the **safe direction**: without `:has()` support, or with no CSS at all, a verified-with-link renders neutral rather than green — under-claiming, never over-claiming.

---

## What it styles

- `.verification` — the wrapper, carrying the state and holding the label + optional link
- The `.badge` inside it — re-coloured per state (composed from the `badge` component)

State is driven by `data-verification` on the wrapper. The colour comes from the attribute, never from the author picking a badge variant class.

---

## The three states

| State | `data-verification` | Colour | Rule |
|---|---|---|---|
| Verified | `verified` | Green (`--success` tint) | **Only with a rationale link** — otherwise neutral |
| Self-reported | `self-reported` | Amber (`--warning` tint) | Always amber; a link is allowed but doesn't change the colour |
| Not disclosed | `not-disclosed` | Neutral | No colour, by design — the plainness is the signal |

**Why these three.** They are the principle-02 trio, and exactly what a provenance chain needs to describe a stage: confirmed by evidence, claimed but unchecked, or unknown. "Estimated" is deliberately *not* a fourth state here — that's about a value's uncertainty, which is `eco-stat`'s range mechanic, a different axis. Keeping them separate stops the two from blurring.

**Why not-disclosed has no colour.** The absence of a verification colour is itself the message: no endorsement, unknown. Colouring a gap would overstate it. This matches how the rest of the layer treats gaps (`eco-stat`'s unmeasured state, `methodology`'s `.methodology-gap`), and keeps the neutral badge safely within contrast.

---

## Markup

```html
<span class="verification" data-verification="verified">
  <span class="badge">Verified</span>
  <a href="…" rel="external">audit</a>
</span>

<span class="verification" data-verification="self-reported">
  <span class="badge">Self-reported</span>
  <a href="…" rel="external">source</a>   <!-- optional -->
</span>

<span class="verification" data-verification="not-disclosed">
  <span class="badge">Not disclosed</span>
</span>
```

With no CSS, each reads as its label plus (where present) a link — fully meaningful.

**The badge stays neutral in markup.** Note there is no `badge-success` or `badge-warning` class in the markup. The colour is applied by `verification-status.css` based on the attribute (and, for verified, the link). This is what makes the green un-fakeable and keeps the mapping attribute-driven.

---

## The rationale link

The link is the evidence: an audit, a certificate, a methodology, a source. It's a sibling of the badge inside the wrapper, and carries `rel="external"` so the links foundation renders the external indicator — don't hand-type an arrow.

- On **verified**, the link is required for the green. No link → neutral badge (non-conforming; see below).
- On **self-reported**, a link is optional and doesn't affect the colour — it just points to the unverified source.
- On **not-disclosed**, there's nothing to link.

The link text is author-supplied and context-specific ("audit", "organic certificate", "supplier statement"). The component supplies the structure; you supply the words.

---

## The non-conforming case

A `verified` wrapper with **no** rationale link renders as the neutral badge, not green. This is intentional and is the component doing its job: a green verified badge without linked rationale is non-conforming under principle 02, so the system presents the word "Verified" plainly and withholds the endorsement.

If you reach this state, you have two honest options: add the rationale link, or — if the claim genuinely isn't verified — use `self-reported`. The reference page documents this as an "Avoid" case.

---

## Tokens used

| Token | Used for |
|---|---|
| `--success` | Verified tint + text (conditional on the link) |
| `--warning` | Self-reported tint + text |
| `--surface` | The `color-mix` base for both tints |
| `--text-sm` | Wrapper / link size |
| `--space-2` | Gap between badge and link |

No new tokens. not-disclosed inherits the neutral `badge` entirely. The two tinted states re-derive badge's functional-variant treatment (15% tint light / 22% dark) so the tint can be bound to the attribute and to `:has()`; `badge.css` remains the source of truth for that formula. Dark mode is automatic via the tokens — only the tint percentage changes.

---

## Accessibility

- **No colour-only meaning.** The label text ("Verified" / "Self-reported" / "Not disclosed") differentiates the states without any reliance on colour, so they hold up on e-ink, in print, and for colour-blind readers. The colour is reinforcement, not the signal — which is also why there's no icon (it would be redundant decoration that reintroduced the SVG-hex maintenance burden).
- **The rationale link** is a normal focusable link; the focus ring comes from `foundations/focus-ring`. Include `focus-ring.css` on pages that use this component.
- **Contrast:** the tints are subtle `color-mix` backgrounds with functional-token text that passes AA on the surface; not-disclosed uses badge's safe neutral text.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/display/badge/badge.css` (this component composes it)
3. Include `components/foundations/links/links.css` and `components/foundations/focus-ring/focus-ring.css` (for the rationale link)
4. Include `components/ecological/verification-status/verification-status.css`
5. Write the markup above, setting `data-verification` and — for verified — including a rationale link

---

## How to override

**To change which token a state maps to:** edit the relevant rule in `verification-status.css`. Keep the verified rule gated on `:has(a)` unless you are deliberately dropping the no-green-without-rationale enforcement (document it if you do — it's a principle-02 decision, not a style tweak).

**To add a state** (e.g. a fork that genuinely needs "estimated" as a verification axis): add a `data-verification` value and a colour rule. Consider first whether the value belongs on the verification axis or the uncertainty axis (`eco-stat`).

**To allow green without a link** (not recommended): change the verified selector from `:has(a)` to an unconditional `[data-verification="verified"]`. This removes the principle-02 safeguard — the reason the component exists — so document the reasoning.

---

## Print rules

Tints are suppressed to save ink; the labels differentiate the states on their own and the rationale link remains, so no information is lost. Matches the badge/control print convention (transparent fill, `--text` border and text).

---

## Composition note

`provenance` composes this component for each stage of a chain, and any `eco-stat` whose value carries a verification status can sit beside one. The data+action pairing (principle 03) is not this component's job — it lives at the composition level, as with the rest of the layer.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
