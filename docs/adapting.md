# Adapting seedbank.design

This is the guide for people who have cloned the template and are about to start changing it.

You own your copy completely. There is no upstream to stay in sync with, no review process, no obligations — the only thing you might ever send back is a one-line entry in `MADE_WITH.md`, and even that is optional. So this guide is not a set of rules. It is the small number of system-wide ideas worth understanding *before* you start editing, because they are the ones where a five-second change can quietly break something with no error message to warn you.

If you read nothing else, read this: **when you edit `tokens/tokens.css`, do not delete token lines you think you aren't using.** A missing token can silently switch off a focus ring or turn every link the colour of body text, with nothing in the console to tell you. The section [Foundation token references carry fallbacks](#foundation-token-references-carry-fallbacks) explains why, and what protects you.

The rest of the guide is in two parts. **Part 1** walks one imagined fork through a full rebrand, touching the load-bearing concepts in the order you'd actually meet them. **Part 2** is reference — the place to come back to when you want to look one thing up.

---

## Part 1 — A worked fork: the desert land trust

Imagine a community land trust in an arid region. They've clicked **Use this template** and now have their own repo. Their identity is dust-red and ochre, not the green the default palette ships with. Here is the whole journey, and what they learn at each step.

### 1. Open `tokens/tokens.css`

Everything visual in the system resolves to a token defined here. Components never hardcode a colour or a size (with a few documented exceptions, listed in Part 2). This means the trust can re-skin the entire system without opening a single component file — that's the point of the token layer, and it's why theming is treated as self-determination rather than configuration.

The file is organised into sections. The colour tokens come in light/dark pairs and are grouped by what they *mean*, not by hue. That grouping is the first thing to understand.

### 2. Recolour the identity: `--grow` becomes terracotta

The trust does a lot of cultivation work, so `--grow` is their most-used activity colour. They change its light/dark pair to terracotta:

```css
/* before */
--grow-light: #2f7a1f;
--grow-dark:  #7bcb6a;

/* after — the trust's terracotta */
--grow-light: #b5532a;
--grow-dark:  #e08a5f;
```

That single change propagates everywhere `--grow` is used: left-rules on "growing" status rows, the grow activity badge, the grow segment in a resource bar. They didn't touch any component.

### 3. Leave the system reports alone: why `--success` stays green

Here's the first place the token grouping matters. The trust might assume that because they've recoloured "grow," they should also recolour "success." They shouldn't — and the system is built so they don't have to.

There are **three** semantic axes, and they are independent on purpose:

- **Activity tokens** (`--grow`, `--repair`, `--share`, `--make`) name *what the community does in the world* — cultivating, fixing, sharing, making.
- **Functional tokens** (`--success`, `--warning`, `--info`, `--error`, `--focus`) name *what the interface is reporting* — a form passed, a warning fired, an error occurred.
- **Content tokens** (`--link`, `--link-visited`) name *what a piece of content is* — a link is a link regardless of status.

A "growing" plot and a "saved successfully" toast are different kinds of meaning. The trust wants their cultivation colour to be terracotta, but they almost certainly want a successful save to still read as green, because green-means-go is a convention their members already understand. Because the axes are separate, recolouring `--grow` leaves `--success` untouched. The system's reports stay recognisable while the identity changes.

The rule of thumb: **change the activity axis to match your identity; leave the functional axis close to web convention unless you have a strong reason.**

### 4. The overlap that isn't aliasing

When the trust opens the file, they'll notice something that looks like a mistake: in the default palette, `--grow` and `--success` are *the same green* (`#2f7a1f`). So are `--share`, `--info`, `--focus`, and `--link` — all the same blue.

This is not aliasing. None of those tokens is defined in terms of another; each carries its own value. They happen to land on the same colour because the palette is constrained to pass WCAG AA contrast, and there are only so many AA-passing greens and blues that sit well on a warm off-white surface.

The practical consequence is the one the trust just relied on: **they remap independently.** Changing `--grow` to terracotta does not drag `--success` with it, even though they started identical. If you ever find yourself thinking "but they're the same colour, so editing one edits the other" — they aren't, and it doesn't.

### 5. Recolour links without touching `--info`

The trust wants their links a touch warmer than the default blue. `--link` is its own content token, so:

```css
--link-light: #1f5fa8;   /* default — change freely */
--link-dark:  #6ba3e0;
```

…can be edited without disturbing `--info`, even though both ship as the same blue family. Links recolour; info toasts stay put. Same lesson as the activity/functional split, applied to the content axis.

### 6. The token you deleted: why every foundation reference has a fallback

Tidying up, the trust notices a token they're "not using" and deletes the line. This is the single most damaging thing you can do to a clone, and it's worth understanding exactly why.

A CSS `var()` reference to a token that doesn't exist, *with no fallback*, doesn't just skip that one value — it invalidates the **entire declaration** at computed-value time. Silently. No console error. The property is simply dropped as if you'd never written it.

This bit the project itself, twice:

- `--border-focus` went missing from `tokens.css`. The focus ring is painted with `outline: var(--border-focus) solid var(--focus)`. With `--border-focus` undefined, the whole `outline` shorthand became invalid, and **no focus ring rendered on any element in the entire system** — invisible until someone keyboard-tested a fresh page.
- `--link` and `--link-visited` went missing the same way. Every link *outside* a prose block fell back to inherited near-black text — unreadable in dark mode, and again, no error.

The protection, now a system convention, is that **every foundation CSS reference to a token carries a sane fallback**:

```css
outline: var(--border-focus, 2px) solid var(--focus, currentColor);
color:   var(--link, var(--info));
```

If the trust now deletes `--border-focus`, the ring degrades to a visible 2px outline instead of vanishing. The fallback converts a silent, system-wide accessibility failure into graceful, visible degradation. **If you write any new component CSS, follow this convention** — and if you delete tokens, check that the references to them carry fallbacks first.

### 7. A fifth resource segment: categorical colour, not a ramp

Later, the trust builds a resource bar showing how a shared water budget is split between four uses, and wants to add a fifth. This is where they meet the system's most instructive colour decision.

The segments are told apart by **distinct hues** drawn from the activity palette, with a matching legend swatch as the join key. It's tempting to reach instead for a light-to-dark ramp of a single colour — it looks tidy. Don't. A luminance ramp is a *sequential* encoding: it implies the segments are ranked or ordered by magnitude. Resource segments are *categorical* — different kinds, no order. Using a sequential channel for categorical data tells a lie about the data.

So: distinct hues for categories, never a ramp. But there's a ceiling. The activity palette gives you **four** AA-tested, light/dark-paired hues. The trust's fifth segment is one past that ceiling. Their options:

1. Use a table instead of a bar — often the honest answer when there are this many parts.
2. Add a dedicated categorical chart palette to `tokens.css` (following the flat-semantic convention — no primitive layer) and repoint `--resource-fill` at it. This also buys hard separation from the activity identity, so a chart hue is never mistaken for a status.

Crucially, colour here is **decoration over a text channel, not the channel itself.** The legend carries each segment's label and share in text, so the breakdown survives grayscale, e-ink, print, colour-blindness, and CSS failing to load entirely. That's what makes distinct hues safe to use — see [Status colour vs categorical colour](#status-colour-vs-categorical-colour).

### What the fork changed, and what it never touched

The trust rebranded the entire system by editing colour values in one file. They never opened a component. They never touched the functional report colours, so their interface still speaks the web's shared language for success and error. And the one genuinely dangerous move — deleting a token — was caught by the fallback convention before it could disable anything silently.

That's the model working as intended: **deep visual change, shallow structural risk.**

---

## Part 2 — Reference

### Tokens

#### The three semantic axes

| Axis | Tokens | Names… | Remap to… |
|---|---|---|---|
| Activity | `--grow` `--repair` `--share` `--make` | what the community *does* | match your identity |
| Functional | `--success` `--warning` `--info` `--error` `--focus` | what the interface *reports* | leave near web convention |
| Content | `--link` `--link-visited` | what the content *is* | recolour links freely |

The axes are independent. Remapping one never requires touching another. This is the most load-bearing idea in the whole token system; the desert fork in Part 1 is the worked example.

#### Default palette overlap is not aliasing

Several tokens across the axes resolve to identical values in the default palette — `--grow` ≈ `--success` (both `#2f7a1f`); `--share` ≈ `--info` ≈ `--focus` ≈ `--link` (all `#1f5fa8`). This is a consequence of the AA-passing colour space, not a definition. Each token holds its own literal value and remaps independently. Document this for your own contributors so nobody assumes editing one edits the others.

#### Foundation token references carry fallbacks

*(The absent-token failure mode.)* A `var(--token)` reference with no fallback invalidates its whole declaration if the token is undefined — silently, no console error. Foundation CSS must always supply a fallback: `var(--token, sensible-default)`. This matters most for accessibility-critical properties (focus outlines above all), which should never be one deleted token away from vanishing without a trace. Worked examples: the `--border-focus` focus-ring incident and the `--link` / `--link-visited` link incident, both in Part 1, step 6.

#### Presentational custom properties must not reuse token names

*(The shadowed-token failure mode — the mirror image of the one above.)* When you set an inline/presentational custom property — a segment width, a marker offset — never give it the name of a real token, especially an activity token. If you do, a `var(--name)` meant to read the token instead resolves to your local value, because the local definition shadows the token in that subtree. Same silent failure: invalid value, dropped declaration, no error.

This bit resource-bar: a segment-width property was named `--share`, colliding with the `--share` activity token, so `--resource-fill: var(--share)` resolved to a width (`12%`) instead of the blue, and the segment painted nothing. The fix was renaming it `--portion`. **Prefix presentational properties** so they can't collide — `--portion`, `--at`, `--seg-gap`, `--prov-offset`, `--prov-dot` are the conventions already in use.

These two rules are the project's two classes of silent custom-property failure. One is a token being *absent* (nothing to resolve to); the other is a token name being *shadowed* (resolves to the wrong thing). Both are invisible in the console and visible only in the rendered result.

---

### Colour

#### Status colour vs categorical colour

Two different jobs colour does, governed by different rules — keep them apart.

- **Status colour** is an activity token reporting a row's or item's state: a `--repair`-coloured left-rule meaning "under repair." Rule: activity-coloured means *the row carries status*; never colour-code by content *category*; meaning must never rest on colour alone.
- **Categorical colour** is distinct hues separating series in a chart or legend (resource-bar segments). This is permitted, and is *not* the prohibited "colour-coding by category" — **provided a canonical non-colour channel carries the meaning underneath** (a legend of label + value in text, an aria-hidden visual). Colour is decoration over that channel, never the load-bearing signal.

The "no colour-coding by category" rule is about *status* semantics, and it's easy to over-apply to legitimate chart colour. When a text channel is present, distinct hues for chart series are correct.

#### Categorical vs sequential encoding

Match the encoding to the data type. **Categorical** data (different kinds, no order) wants distinct *hues*. A light-to-dark **luminance ramp** is a *sequential* encoding and falsely implies order or magnitude — don't use it for nominal data. resource-bar landed this only on its third attempt (neutral segments → a misleading luminance ramp → distinct hues); the ramp was rejected precisely because it lied about ranking.

#### The four-hue ceiling

The activity palette doubles as a ready-made four-way categorical palette (AA-tested, light/dark-paired). Reusing it costs no new tokens but overloads the activity colours — a chart hue is not a status; mitigate with the legend and obvious chart context. Beyond ~4 series, either switch to a table or add a dedicated categorical chart palette and repoint `--resource-fill`. Adding tokens is in-scope when a genuine need surfaces; follow the flat-semantic convention and never add a primitive layer.

---

### Structure and state

#### State from attributes, not class names

Components infer state from HTML attributes — `disabled`, `required`, `aria-invalid`, `aria-busy`, `aria-current` — never from class names like `.btn-disabled` or `.input-error`. Markup honesty over class proliferation; forks shouldn't have to memorise class-state mappings. State is in the markup; the CSS reads it.

The same principle extends to *not* auto-detecting state from value patterns: the external-link indicator keys off `rel="external"`, not a URL scheme, because auto-detection is too clever and breaks edge cases.

#### The two documented class exceptions

Two places legitimately use classes for state, because the thing they encode is a four-way *categorical* axis, not a binary state:

- `.tr-grow` / `.tr-repair` / `.tr-share` / `.tr-make` — activity row status on tables.
- `.card-status-row` / `.card-editorial` — card variety.

Activity isn't on/off; it's one-of-four. Attributes suit binary states; a categorical axis is the documented exception where a class is the right tool.

#### `:disabled`, not `[disabled]`

Per-input disabled rules use the `:disabled` pseudo-class, not the `[disabled]` attribute selector. Inside `<fieldset disabled>`, the browser marks every descendant `:disabled` but does **not** add a `disabled` attribute to each one — the attribute selector would miss them. (`fieldset[disabled]` itself is fine, because the fieldset genuinely carries the attribute.)

#### Two-vocabulary structure

The system speaks two visual dialects on purpose:

- **Editorial** elements (cards, list items, inputs, dividers) use `--radius-none` and hairline borders — they feel like print.
- **Software** elements (buttons) use a control radius and solid fills — they feel like interface.

If you add a component, decide which dialect it belongs to and stay consistent with it.

#### Left-rule colour

The 5px left-rule accent: **activity-coloured = the row carries status; neutral = no status.** Never colour-code a left-rule by content *category* (e.g. don't make all "energy" cards one colour just because they're about energy). This holds even for dedicated component types like ecological cards.

---

### Cards

Two varieties, one chooser rule:

- **`card-status-row`** — bare on `--surface`, hairlines above and below, an activity left-rule *when the row carries status*. For lists of items.
- **`card-editorial`** — raised on `--surface-raised`, defined by the surface change, an optional neutral rule. For content blocks.

**Chooser:** status-row for lists, editorial for content. The cards pattern ships illustrative CSS framed as *structure, not API* — copy and adapt it; it isn't a fixed contract.

---

### Forms

#### `.error-message` and clear-on-satisfied

An invalid-state error message is a `<p class="error-message">` placed inside the field's wrapping container (sibling of the wrapping `<label>`; or inside the `<fieldset>` for groups; or sibling of the input in a `.field` wrapper). The CSS uses `:has()` to detect when the requirement is satisfied and `visibility: hidden` (not `display: none`) to hide it, so layout doesn't jump.

**Clear-on-satisfied:** when the user meets the requirement (checks the box, picks a radio, selects a valid option), the CSS clears the red border *and* the message immediately. The `aria-invalid` attribute stays in the markup — only JS can remove it — but the visual stops nagging the moment the requirement is met. Caveat: forms validating *beyond* "must select / must check" need their own signal, since this rule will suppress red that should remain. `:has()` is ~94% global; older browsers fall back to "red stays after the action," which is annoying but not broken.

#### Required asterisk, and where its colour lives

Markup: `<span class="required" aria-hidden="true">*</span>` alongside the `required` attribute. For radio groups, the asterisk goes in the `<legend>` — the group's question is the required thing, not each option. `.required` is the canonical class.

**Open question, resolved here:** the design system ships no global stylesheet, so there is currently no shared home for `.required { color: var(--error); }` — the reference pages each style it locally. **Recommendation for forks: add `.required { color: var(--error); }` once to your own global/app stylesheet** rather than repeating it per page. The system can't own this rule without shipping global CSS (which would violate the opt-in model for controls); the fork owns it. If a future form-field CSS file ships, that becomes the natural home — until then, it's yours.

#### Placeholder `<option>`: disable for required, keep for optional

**Open question, resolved here.** Whether the placeholder `<option value="">` should carry `disabled` depends on whether the field is required:

- **Required select → `disabled` the placeholder.** This stops the user from reverting to "nothing chosen" after they've picked, which is what you want when a choice is mandatory.
- **Optional select → leave the placeholder enabled.** Here the empty option is a legitimate "clear this / no preference" choice, and disabling it would block a valid intent.

Docs examples follow this split. There is no single default that's right for both, so the rule is conditional on the field, not the component.

#### `type="number"` guidance

Use `type="number"` only for values where spinner increment/decrement and numeric-only input genuinely help (quantities, counts). Avoid it for things that look numeric but aren't quantities — phone numbers, PINs, codes, years entered as identifiers — where it causes more trouble (scroll-to-change, locale quirks) than it solves. For those, `inputmode="numeric"` on a text input is usually better.

---

### Honest uncertainty (principle 02) as a toolkit

Uncertainty isn't handled per-component with bespoke tricks; the ecological layer established a small reusable toolkit you can compose anywhere information is displayed:

- **Value-with-range** (eco-stat): show a low–high band with **no point marker** — the band *is* the message. A precise dot in the middle of an uncertain range is false precision.
- **Methodology disclosure** (native `<details>`): a "how this was calculated" toggle that needs no JS and works even with no CSS.
- **The `:has(a)`-gated verified green** (verification-status): the "verified" green appears *only* when a rationale link is present, so trust is structurally earned, never asserted. It degrades in the safe direction — neutral, never unearned green.
- **Explicit data gaps**: eco-stat's unmeasured state, methodology's gap note, verification's "not disclosed" (deliberately given *no* colour — its plainness is the signal). A gap is shown, not omitted.

This applies beyond ecological data — wherever your fork displays information, represent what's known, estimated, and missing.

---

### Collective agency (principle 03) as structure

Three structural commitments, not stylistic ones:

- **Data + action pairing lives at composition level**, not baked into primitives. A stat is paired with the decision it supports where you assemble the page — the primitive stays reusable.
- **Aggregate framing is explicit.** Indicators describe the collective, not the individual.
- **There is no per-user component, on purpose.** The absence is the policy. Don't add per-user breakdowns or individual alerts; that's surveillance wearing a dashboard.

---

### Text-canonical, visual-aria-hidden — and the neutral-rail rule

The ecological components share one anchor worth copying if you build your own data displays: **the text form is canonical; the visual is aria-hidden decoration that degrades to the text.**

- eco-stat: the value + range *text* is canonical; the band is an aria-hidden visual.
- resource-bar: the legend (label + share in text) is canonical; the track is an aria-hidden visual that mirrors it.
- provenance: the `<ol>` of stages is canonical; the timeline rail is aria-hidden enhancement that vanishes cleanly with no CSS.

The corollary is the **neutral-rail / no-double-coding rule**: provenance's rail is neutral, never colour-coded by verification status, because verification is already carried by the composed badge's colour *and* text. Coding it twice — once in the badge, again in the rail colour — would re-encode a signal in a less accessible channel. Carry each meaning once, in its most accessible channel.

---

### Degrading gracefully — designed for the edges

Graceful degradation is a positive design goal here, not a fallback you tolerate. The system is full of worked proof that things stay usable when CSS, JS, or colour drop away:

- **Buttons**: the loading-state label carries meaning without the spinner.
- **Tables and code**: readable with no CSS loaded.
- **eco-stat**: the text range survives with no CSS.
- **methodology**: native `<details>` works with no JS *and* no CSS.
- **verification-status**: the label text carries the meaning, colour-independently.
- **provenance**: degrades to a plain numbered list.
- **resource-bar**: the legend is a full breakdown on its own.
- **The entire ecological layer requires no JS** — the proof that rich behaviour can be fully native (native `<details>`, native `<ol>`, static bars).

When you add to your fork, the test is the same three-criterion bar every component faced: does it render without CSS, work at 200% zoom, and work at 320px? The low-fidelity testing guide covers how to check.

---

### Reference tables for editors

#### SVG hex you must re-encode if you change surface/text tokens

A few components bake hex values directly into CSS data-URI SVGs, because a data-URI can't read a CSS variable. If you change the corresponding token, **you must re-encode the SVG by hand** — it won't update automatically.

| Component | Hardcoded hex | Tied to token | Re-encode if you change… |
|---|---|---|---|
| checkbox | `#fafaf7`, `#1a1a1a` | surface / text | `--surface-light`, `--text-light`/`--text-dark` |
| select | `#1a1a1a`, `#f0f0f0` | text / surface | `--text-light`/`--text-dark`, `--surface-light` |

Where geometry allows, prefer the pure-CSS approach instead of an SVG data-URI: radio's dot is a `::before` pseudo-element and methodology's chevron is a CSS border-corner — neither hardcodes hex, both inherit `currentColor` and need no re-encoding. Use that pattern for new work when the shape is simple enough.

#### Hardcoded sizing exceptions

A handful of small fixed sizes are not yet tokenised:

| Value | Where |
|---|---|
| 18px | checkbox box, radio circle |
| 8px | radio dot |
| 3px | badge vertical padding |
| 0.9em | inline code shrink |

If your fork lands more controls and these start to multiply, that's the signal to introduce a `--control-size-sm` token — until then, tokenising one or two values in isolation isn't worth it.

#### Features that need JS, and what they degrade to

The component CSS ships no JavaScript. A few behaviours genuinely require it; here's what each degrades to without it, which matters for static-site-only deployments:

| Feature | Needs JS for | Without JS |
|---|---|---|
| button `aria-busy` | toggling the loading state | static button; label still meaningful |
| checkbox indeterminate | setting `.indeterminate` + parent/child sync | normal checked/unchecked |
| `aria-invalid` | adding/removing on validate | clear-on-satisfied CSS still hides red visually; attribute stays |

The positive counter-example to keep in view: the **entire ecological layer needs no JS at all.** If you can express a behaviour natively (`<details>`, `<ol>`, CSS `:has()`), do — it's one less thing to break at the edges.

#### External / rationale link affordance

methodology's source link, verification-status's rationale link, and provenance's per-stage links all use `rel="external"` for the indicator. If your fork leans heavily on evidence links, settle a single consistent treatment (wording and indicator) across them rather than letting each drift — consistency here is part of the honesty story, since these links are what back up the "verified" claims.

---

*Code in this repo is MIT; documentation and design assets, including this guide and `tokens.css`, are CC BY-SA 4.0. Your fork owes nothing upstream.*
