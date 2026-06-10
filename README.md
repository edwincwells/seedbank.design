# seedbank.design

**A design system for communities, not companies.**

Built to be cloned, adapted, and owned entirely by the people who use it. There is no upstream to stay consistent with, no brand to protect, and no approval process for changes. Once you clone it, it's yours.

---

## Who this is for

Any group building something together outside mainstream institutions — housing cooperatives, community land trusts, repair cafés, mutual aid networks, local food systems, countercultural collectives, grassroots campaigns, post-capitalist projects of all kinds.

This is a starting point that reflects your values, not a corporation's.

---

## What's in here

```
seedbank.design/
├── README.md              ← you are here
├── PRINCIPLES.md          ← the design principles, shipped as an asset
├── TESTING.md             ← per-component acceptance bar
├── MADE_WITH.md           ← voluntary registry of projects built with this
├── LICENSE                ← MIT (code) + CC BY-SA 4.0 (docs/assets)
├── tokens/
│   └── tokens.css         ← the full design token system
├── components/
│   ├── foundations/       ← focus ring, prose, links, dividers, lists
│   ├── controls/          ← buttons, inputs, selects, checkboxes, radios
│   ├── display/           ← badges, tables, code
│   ├── patterns/          ← cards, activity badges, form fields
│   └── ecological/        ← energy budgets, carbon ranges, provenance chains
└── docs/                  ← usage and adaptation guide
```

**Tokens** are the foundation. Every visual property — colour, spacing, type — is a CSS custom property with a value-based name. `--grow`, `--repair`, `--share`, `--make`. Remap them without touching a single component.

**Primitive components** are the building blocks: buttons, badges, cards, type scale, form inputs. Minimal opinion, maximum adaptability. Written in semantic HTML and CSS custom properties — no framework required.

**Ecological components** are for communities that track shared resources: energy budgets, carbon estimates with honest uncertainty ranges, material provenance chains. Use them if they're relevant, ignore them if they're not.

**Docs** cover how to adapt the token system, how to extend or remove components, and how to use the system without design or development experience.

---

## How to use it

### 1. Clone the template

Click **Use this template** at the top of this repository. GitHub will create a new independent repo in your account — no link back to this one, no obligations, no upstream relationship. It's yours.

### 2. Remap the tokens

Open `tokens/tokens.css`. This is the one file that controls the visual identity of the entire system. Change the values — colours, type scale, spacing — to fit your community's context. You don't need to touch any component files.

```css
:root {
  --grow:   #4a9e4a;  /* primary action, affirmative states */
  --repair: #c47c2b;  /* maintenance, in-progress, caution */
  --share:  #3d7ebf;  /* collaborative, communal */
  --make:   #7b5ea7;  /* creative, generative */
}
```

### 3. Read the design principles

`PRINCIPLES.md` explains the three principles the system is built around. They ship with every clone because they make the design decisions legible — so adaptations are conscious, not accidental.

If your context requires changing or dropping a principle, that's a valid fork. Document why.

### 4. Adapt and build

Add components, remove ones you don't need, extend the token system. The docs cover common adaptation patterns. Nothing is locked.

---

## Design decisions

A few choices are worth understanding before you adapt the system, because they affect everything downstream.

**Tokens are named for values, not visuals.** `--repair` is not named `--orange`. The name carries meaning that survives a palette change — a community that remaps `--repair` to a different colour still understands what it's for.

**No gradients, no decorative effects.** Every component works on e-ink, printed on A4, and rendered over a slow connection. This is not a constraint imposed on the system — it's the design brief. Communities using this system are often operating in exactly these conditions.

**Dark mode is mandatory, not optional.** All components are tested in both light and dark modes before release.

**Icons always have visible labels.** No icon-only controls. Ever.

**Ecological components show uncertainty, not false precision.** A carbon estimate is displayed as a range with a methodology link, not a single authoritative number. Data gaps are shown as gaps, not hidden.

**Data is aggregated, not per-user.** Ecological indicators show collective metrics. No per-household breakdowns, no individual surveillance dressed as sustainability data.

---

## What this is not

It is not a brand system. There is no correct way to implement it.

It is not a governed commons. There are no stewards to approve your changes, no RFC process to follow, no upstream to report back to.

It is not finished. It is a starting point. The expectation is that you will change it.

---

## If you build something with it

You're not obligated to tell anyone. But if you do, add a line to `MADE_WITH.md` in your own repo — and optionally open a PR to add it to the registry here. One line: your project name, location, what you changed, a URL if you have one.

This is how the seed bank grows. Not through governance, but through visibility.

---

## Licence

Code: [MIT](LICENSE)
Documentation and design assets (including `PRINCIPLES.md`): [CC BY-SA 4.0](LICENSE)

You can use, adapt, and redistribute freely. Attribution appreciated, not required for the code. Required for the docs and assets under CC BY-SA — derivative works must use the same licence.

---

*seedbank.design — a design system for communities, not companies.*
