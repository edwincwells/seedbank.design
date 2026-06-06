# cards

A **composition pattern**, not a component. Two card varieties built from surfaces, hairlines, the activity left-rule, and the components beneath them. No new CSS ships with this pattern — cards are assembled from tokens and existing components.

The two varieties:

- **card-status-row** — bare on the page surface, hairlines above and below, activity-coloured left-rule when the row carries status. For lists of items.
- **card-editorial** — contained on a raised surface, no surrounding hairlines, neutral. For self-contained content blocks.

---

## Why two varieties

Most design systems ship one card: a contained box with a border, rounded corners, and often a shadow. That card does everything adequately and nothing well. seedbank.design splits the two real jobs a card does:

1. **Listing things** — a tool inventory, a member directory, an event queue, a repair backlog. These are rows in a list. They want to read as a continuous, scannable sequence. A box around each one fragments the list into disconnected cells.

2. **Containing content** — a workshop's minutes, an article summary, a document block, an image with a caption. These are self-contained editorial units. They want a boundary that says "this is one complete thing".

The status-row variety serves the first job; the editorial variety serves the second. The chooser rule:

> **status-row for lists, editorial for content.**

If you're rendering many similar items the user will scan down, use status-row. If you're rendering one self-contained block the user will read, use editorial.

---

## Shared conventions

Both varieties hold to these:

- **`--radius-none`.** Cards are editorial elements in the system's two-vocabulary structure — they feel like print, not software. No rounded corners. (Interactive controls get the rounded "software" radius; cards don't.)
- **No shadows.** Shadows don't survive e-ink, print, or low-bandwidth rendering, and they imply an elevation hierarchy the system doesn't use. Boundaries come from surface changes and hairlines, not shadows.
- **No hover state by default.** A card is a container, not a control. If a whole card is clickable (a link card), the hover belongs to the wrapping `<a>`, not the card — same principle as the badge component's "wrap, don't restyle".
- **`--space-3` to `--space-4` padding.** Comfortable internal spacing without being generous enough to waste vertical space in a long list.

---

## card-status-row

For lists of items where each item may carry a status.

### Structure

Each card is bare on the page `--surface` — no background change, no border box. Hairlines above and below separate it from its neighbours. In a list, the hairlines stack so the list reads as a continuous sequence of rows with hairline separators between them.

```html
<div class="card-status-row">
  <div class="card-body">
    <div class="card-title">Mitre saw</div>
    <div class="card-meta">Workshop A · last serviced March 2026</div>
  </div>
  <span class="badge badge-grow">Available</span>
</div>
```

The class names (`.card-status-row`, `.card-body`, `.card-title`, `.card-meta`) are **your fork's to define** — this pattern documents the composition, not a fixed class API. A minimal CSS realisation:

```css
.card-status-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3) var(--space-4);
  border-top: var(--border-hairline) solid var(--border);
}
/* Collapse doubled hairlines in a stacked list */
.card-status-row + .card-status-row {
  border-top: none;
}
.card-list {
  border-bottom: var(--border-hairline) solid var(--border);
}
.card-title {
  font-size: var(--text-md);
  font-weight: var(--weight-medium);
  color: var(--text);
}
.card-meta {
  font-family: var(--font-mono);
  font-size: var(--text-xs);
  color: var(--text-subtle);
  margin-top: 2px;
}
```

This is illustrative. The pattern is the structure (bare on surface, hairline-separated, flex layout with a status slot); the exact CSS is the fork's.

### Status left-rule

When a row carries status — when the activity state *is* the point of the row, not just one field among many — add a 5px activity-coloured left-rule:

```html
<div class="card-status-row card-grow">…</div>
<div class="card-status-row card-repair">…</div>
<div class="card-status-row card-share">…</div>
<div class="card-status-row card-make">…</div>
```

```css
.card-grow   { border-left: var(--border-rule) solid var(--grow);   padding-left: var(--space-3); }
.card-repair { border-left: var(--border-rule) solid var(--repair); padding-left: var(--space-3); }
.card-share  { border-left: var(--border-rule) solid var(--share);  padding-left: var(--space-3); }
.card-make   { border-left: var(--border-rule) solid var(--make);   padding-left: var(--space-3); }
```

This mirrors the `display/table` row-status convention exactly — `.tr-grow` / `.tr-repair` / etc. on a table row, `.card-grow` / `.card-repair` / etc. on a card. The same activity-class-on-the-element convention, the same `--border-rule` 5px width, the same colours.

**The rule and the badge should match.** A card with a `card-repair` left-rule should carry a `badge-repair` status badge. The left-rule is the scannable signal (you read it down the list); the badge is the explicit label (it tells you what the status is). They're the same colour because they're the same status.

**Don't give every card a rule.** A card with no status carries no left-rule — it's just a bare row. The left-rule is a marker for rows that have a state worth scanning for, not decoration for every item. The "Heat press — Archived" row in a tool list might carry a neutral badge and no rule, because "archived" isn't an active state the community is scanning for.

### When to use

Tool inventories, member directories, event listings, repair queues, resource catalogues, mutual-aid request lists — anywhere the content is a sequence of similar items the user scans through.

---

## card-editorial

For self-contained content blocks.

### Structure

Each card sits on `--surface-raised` — a warm cream tone one step up from the page surface. The surface change *is* the boundary; no hairlines around it, no border box. A neutral left-rule may be used to anchor the card visually, but it's neutral (`--border`), not an activity colour — an editorial card isn't reporting community status.

```html
<div class="card-editorial">
  <h3 class="card-title">Workshop minutes — March 2026</h3>
  <p class="card-meta">Authored by Mira · 4 contributors</p>
  <p>The March session focused on the bandsaw repair and the new tool-lending procedure. Fourteen members attended…</p>
</div>
```

A minimal CSS realisation:

```css
.card-editorial {
  background: var(--surface-raised);
  padding: var(--space-4) var(--space-5);
  /* Optional neutral left-rule for visual anchoring */
  border-left: var(--border-rule) solid var(--border);
}
```

The raised surface lifts the card off the page without a box. The optional neutral rule gives it a visual anchor on the left edge — use it when the card needs a stronger boundary, omit it when the surface change alone is enough.

### Why neutral, not activity-coloured

An editorial card is content, not status. It's a block of minutes, an article, a document summary — it isn't reporting that something is growing or under repair. Using an activity colour on its left-rule would imply a status the card doesn't carry.

This is the system's left-rule rule, applied: **activity-coloured rule = the row carries status; neutral rule = no status.** An editorial card doesn't carry status, so its rule (if present) is neutral. See BUILD_CONVENTIONS.md.

If an editorial card genuinely does carry an activity status — say, an article that's "In progress" / being drafted — then either (a) it's actually a status-row in disguise and should use that variety, or (b) the status belongs in a badge inside the card, not on the card's rule.

### When to use

Workshop minutes, article summaries, document blocks, image-and-caption units, decision records, contributor recognition blocks — anywhere the content is one self-contained thing the user reads rather than scans.

---

## Choosing between them

| | card-status-row | card-editorial |
|---|---|---|
| **Job** | Listing items | Containing content |
| **Surface** | Bare on `--surface` | Raised on `--surface-raised` |
| **Boundary** | Hairlines above/below | Surface change |
| **Left-rule** | Activity-coloured when status | Neutral, optional |
| **Reading mode** | Scanned down a list | Read as a block |
| **Typical content** | Tool, member, event, request | Minutes, article, document |

When you're unsure, ask: *will the user scan a list of these, or read one of these?* Scanning → status-row. Reading → editorial.

---

## Composition with other components and patterns

Cards compose several things beneath them:

- **`display/badge`** — status-row cards typically carry a badge in their status slot. The badge's activity variant should match the card's left-rule colour. See `patterns/activity-badge/README.md` for the colour-choice convention.
- **`foundations/prose`** — editorial cards contain prose (`<h3>`, `<p>`). The prose foundation styles this automatically. Note: prose's `65ch` paragraph max-width applies inside cards too — for a narrow editorial card this is fine, but a wide card may want to override it (`.card-editorial p { max-width: none; }`).
- **`foundations/links`** — a clickable card wraps its content in an `<a>`; the link foundation handles focus and the card provides the visual. Don't add hover or cursor styling to the card itself.
- **`display/code`** — an editorial card documenting a code example will contain `<pre><code>` blocks, styled automatically.

---

## Accessibility

- **Cards are not interactive by default.** No focus, no hover, no cursor. A card is a `<div>` containing content. If the whole card should be clickable, wrap it in an `<a>` (the link is the interactive element, the card is its visual).
- **Status is not colour alone.** A status-row card's left-rule is a visual scan aid, but the status meaning must come from the badge text or content — "Under repair", not just a red stripe. Screen readers don't see the rule.
- **Heading hierarchy.** Editorial cards usually start with a heading (`<h3>` or `<h4>`). Keep the heading level correct for the card's position in the document outline — don't pick a level for its visual size (that's what the prose foundation's scale is for).
- **Lists of status-row cards** are semantically a list. Consider wrapping them in `<ul>`/`<li>` (with the card structure inside each `<li>`) so assistive tech announces the count and list semantics. The flex card layout works inside an `<li>`.

---

## Why this pattern ships no CSS

Cards are compositions of tokens and existing components. Shipping card CSS would create a third cascading layer (foundations → components → patterns) and pin down class names that forks will want to own. Instead, this README documents the structure and shows illustrative CSS that forks adapt.

The illustrative CSS in this document is a starting point, not an API. Copy it, rename the classes to fit your fork, adjust the spacing and the layout. The pattern is the set of decisions — bare vs raised, hairline vs surface boundary, activity vs neutral rule, scan vs read — not the specific selectors.

---

## See also

- `patterns/activity-badge/README.md` — the status badges that appear in status-row cards
- `display/badge/README.md` — the badge component
- `display/table/README.md` — table rows use the same `.tr-*` activity-rule convention as status-row cards
- `foundations/prose/README.md` — the prose styling inside editorial cards
- BUILD_CONVENTIONS.md — the left-rule colour rule (activity = status, neutral = no status) and the two-vocabulary structure
