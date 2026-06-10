# Using seedbank.design

This is the build-a-page companion to `adapting.md`. Where the adaptation guide is about *changing* the system, this one is about *assembling* things with it — getting a page up, composing components, and knowing which to reach for.

It does not restate the components. **Each component's `README.md` is the source of truth for that component** — its states, its overrides, its rationale. This guide is the connective tissue between them: how to start, how the layers fit together, and how to choose between near-neighbours.

---

## Getting started

### 1. The one mandatory import

`tokens/tokens.css` is the only file you must include. Everything else resolves to the tokens it defines, so it has to load first.

```html
<link rel="stylesheet" href="path/to/tokens/tokens.css">
```

Because the tokens ship as light/dark pairs swapped via `prefers-color-scheme`, **dark mode works automatically** the moment you're on tokens — there's nothing to wire up.

### 2. Link what you use

There are two kinds of stylesheet, and they're included differently:

- **Foundations** (`focus-ring`, `prose`, `links`, `dividers`, `lists`) style your semantic HTML by *element selector* — `h1`, `a`, `hr`, not `.heading`. Once you link a foundation file, it applies to all matching elements with no classes to remember. Link `prose.css` wherever you have text, `focus-ring.css` wherever anything is focusable.
- **Controls and display** (`button`, `text-input`, `badge`, `table`, …) are à la carte. Link the ones you need, ignore the rest.

A page importing tokens plus a couple of layers looks like:

```html
<link rel="stylesheet" href="../tokens/tokens.css">
<link rel="stylesheet" href="../components/foundations/prose/prose.css">
<link rel="stylesheet" href="../components/foundations/focus-ring/focus-ring.css">
<link rel="stylesheet" href="../components/controls/button/button.css">
```

(The depth of the relative path depends on where your page sits. The component reference pages import from `../../../tokens/tokens.css` because they live three levels deep; your own pages will differ.)

### 3. Your first control

The default button needs no class — primary *is* the default:

```html
<button>Save</button>
<button class="btn-destructive">Delete</button>
```

State comes from attributes, never classes: `disabled`, `aria-busy="true"` for loading. (See [state from attributes](adapting.md) in the adaptation guide for why.)

### 4. Compose a field

Form structure follows the **form-field** pattern — label, optional help, control, error — in that order:

```html
<label>
  Plot name <span class="required" aria-hidden="true">*</span>
  <input type="text" required>
</label>
```

form-field is a *pattern*, not a component (see below): it ships markup conventions, not a CSS file. Its README is where the canonical field structure, `aria-describedby` association, and error placement live.

### 5. Compose a card from primitives

A list item that carries status is a `card-status-row` — bare on the surface, hairlines top and bottom, an activity left-rule *because it has status*:

```html
<div class="card-status-row card-repair">
  <div class="card-body">
    <div class="card-title">Cargo bike</div>
    <div class="card-meta">Workshop A · last serviced March 2026</div>
  </div>
  <span class="badge badge-repair">Under repair</span>
</div>
```

The activity left-rule (`card-repair`) and the badge (`badge-repair`) match because they're the same status — the rule is the signal you scan down a list, the badge is the explicit label. The inner class names (`card-body`, `card-title`, `card-meta`) are illustrative: cards ships no CSS, so these are yours to define. If the row had *no* status, you'd drop the `card-repair` rule and it would read as a plain bare row — never colour the rule by what the card is *about*.

### 6. Compose an ecological stat

The ecological components are where composition pays off — they assemble everything beneath them. A trustworthy, honest stat is several pieces working together:

```html
<div class="card-status-row">
  <div class="eco-stat">
    <p class="eco-stat-label">Energy generated this week</p>
    <p class="eco-stat-value">
      <data value="142">142</data><span class="eco-stat-unit">kWh</span>
    </p>
    <p class="eco-stat-range">estimated <data>120–165</data> kWh</p>
  </div>

  <details class="methodology">
    <summary>How this was calculated</summary>
    <div class="methodology-body">
      <p>…</p>
    </div>
  </details>

  <p class="verification" data-verification="verified">
    <span class="badge">Verified</span>
    <a href="…" rel="external">see rationale</a>
  </p>

  <p>Enough to run the workshop for three days. <a href="…">Adjust the budget →</a></p>
</div>
```

Three things to notice, each a system principle in practice:

- **The range has no point marker** — the band is the message. That's honest uncertainty, not decoration.
- **The "verified" green only appears because the rationale link is present.** Drop the link and it degrades to neutral; trust is structurally earned.
- **The data is paired with a decision** ("Adjust the budget") *at this composition level*, not baked into eco-stat. The primitive stays reusable; the page supplies the agency.

---

## The layer map

Five layers, built (and composed) bottom-up. Each composes the ones below it.

| Layer | Components | Reach for it when… |
|---|---|---|
| **Foundations** *(global, element selectors)* | focus-ring · prose · links · dividers · lists | always — these style your raw HTML |
| **Controls** *(opt-in CSS)* | button · text-input · checkbox · select · radio | you need input |
| **Display** *(opt-in CSS)* | badge · table · code | you're presenting status, tabular data, or code |
| **Patterns** *(markdown only)* | activity-badge · cards · form-field | you're assembling the above into a recognised shape |
| **Ecological** *(compose everything)* | eco-stat · methodology · verification-status · provenance · resource-bar | you're showing measured, sourced, or collective data |

The build order — foundations → controls → display → patterns → ecological — is also the **dependency direction**. When you assemble a page, work the same way: get your type and controls right, then your display elements, then compose them into patterns and ecological displays.

---

## Patterns are recipes, not imports

Three of the entries above are **patterns**, and they behave differently from components. A pattern ships a `README.md` and nothing else — no CSS file to link, no component to import. Some carry *illustrative* CSS, but it's framed explicitly as **structure, not API**: copy it, adapt it, rename it; it isn't a contract you're held to.

- **activity-badge** — the writing convention for activity-coloured status (when to badge it, when to leave it as coloured text in prose).
- **cards** — the two card varieties and the chooser rule.
- **form-field** — the canonical field structure and the controls-layer conventions consolidated in one place.

You don't *use* a pattern by importing it; you *follow* it. If you find yourself looking for `cards.css` to link, that's the signal you've mistaken a pattern for a component.

---

## Choosing between near-neighbours

The judgement calls that come up most. Each component README has the full reasoning; these are the one-line "which one."

- **card-status-row vs card-editorial** — status-row for items in a *list*; editorial for *content blocks*. Lists carry status and hairlines; content blocks sit raised on `--surface-raised`.
- **Badge vs unbadged activity text** — badge it for structured status (lists, tables, card rows); leave it as coloured text for status mentioned *in narrative prose*. (activity-badge pattern.)
- **eco-stat range vs exact value** — if the value is uncertain, show the low–high band with no point marker; if it's known exactly, use the exact-value form with no range. Don't fake a range, and don't drop one that exists.
- **verification-status: three states, no fourth** — verified / self-reported / not-disclosed. There is deliberately no "estimated" state — *estimation is uncertainty*, and uncertainty is eco-stat's axis (the range), not verification's. Keep the two axes apart: verification is about *who vouches for the claim*, eco-stat is about *how precise the number is*.
- **resource-bar vs table** — a segmented bar carries up to ~4 categorical series cleanly (the activity-palette ceiling). More than that, use a table, or add a dedicated chart palette (see `adapting.md`). A bar that needs a fifth hue is usually telling you it wanted to be a table.
- **radio vs select** — radios for a small set of options the user should *see all of* (roughly <5); a select for longer lists. radio's reference page documents the refusal explicitly — don't reach for a select to save space when seeing the options matters.

---

## Where the detail lives

This guide deliberately doesn't duplicate the components. For anything beyond "which one and how do they fit together":

- **Per-component states, overrides, and rationale** → that component's `README.md`.
- **Field structure, validation, grouping, `aria-*` association** → the form-field pattern README.
- **Rebranding, token edits, colour rules, the editor reference tables (SVG re-encoding, sizing, JS-required features)** → `adapting.md`.
- **The three-criterion acceptance bar and how to test a fork's changes** → `testing-lowfi.md`.

---

*Code is MIT; this guide is CC BY-SA 4.0.*
