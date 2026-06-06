# activity-badge

A **writing convention**, not a component. Activity-coloured status text — used to indicate that something in the community is growing, under repair, being shared, or being made.

This pattern doesn't introduce any new CSS. It documents how to use the activity tokens (`--grow`, `--repair`, `--share`, `--make`) to express community-activity status in content, optionally in combination with the `display/badge` component.

---

## What this pattern is for

Communities using the system display activity status constantly. A tool is "Available" or "Under repair". A workshop space is "In use" or "Being set up". A plot of garden is "Growing" or "Recently harvested". The activity tokens exist for exactly this — to give those statuses a recognisable colour vocabulary across the system.

The activity-badge pattern is the convention for how to render that colour-coded status text in content. Two variants:

1. **Badged** — wrapped in a `.badge` component, for structured display in lists, tables, and cards
2. **Unbadged** — coloured text in running prose, for narrative references

Both use the activity tokens. The choice between them depends on context, not preference.

---

## When to badge vs not

**Badge it when** the status is structured information — appearing as its own field, in a column, in a card's status slot. Lists of items where each row has a state. Tables where one column reports status. Cards where the status is a metadata field.

```html
<!-- In a tool inventory list -->
<li>
  Mitre saw <span class="badge badge-grow">Available</span>
</li>

<!-- In a table cell -->
<td><span class="badge badge-repair">Under repair</span></td>

<!-- In a card metadata slot -->
<div class="card-meta">
  <span class="badge badge-share">In use until 6pm</span>
</div>
```

**Don't badge** when the status is woven into a sentence. Narrative references to status — describing what's happening, not labelling a field — should be coloured text without a badge.

```html
<!-- In running prose -->
<p>Three of the workshop's drills are <span style="color: var(--repair);">currently under repair</span> and will be back in circulation by Friday.</p>

<!-- In a heading or callout -->
<h3>What's <span style="color: var(--grow);">growing</span> in the May plot</h3>
```

The badged version of "currently under repair" mid-sentence would look like a status field accidentally pasted into prose. The unbadged version of "Available" in a table cell would look like a sentence fragment that lost its container.

---

## Picking the right activity colour

The four activity tokens name what the community does. Pick by meaning:

| Token | Use for status meaning |
|---|---|
| `--grow` | Available, growing, cultivating, ready, affirmative |
| `--repair` | Under repair, in maintenance, paused, requires attention |
| `--share` | In use, currently shared, communal, collaborative |
| `--make` | In progress, being made, work-in-flight, creative |

The mapping isn't rigid — a fork's specific use case might find that "Borrowed" fits `--share` or `--make` better depending on what the borrowing means in that community. The point is to pick consistently within your fork. A tool that's "In use" should always be `--share`; a tool that's "Being modified" should always be `--make`. Don't oscillate between activity colours for the same status meaning across different views.

**Don't use activity tokens for interface state.** A form's "Submitted" status uses `--success`, not `--grow`. A toast announcing "Saved" uses `--success`, not `--grow`. The distinction matters: a fork that remaps `--grow` to terracotta for cultural reasons should still see green success messages. See `tokens.css` and `display/badge` for the activity-vs-functional reasoning.

---

## How to apply the colour

This is a writing convention, not a CSS layer. The pattern doesn't ship a `.status-repair` class — that would duplicate what `.badge-repair` already provides for the badged case, and add a class just for the rare unbadged case.

For unbadged activity-coloured text, use the token as a text colour however your fork prefers to wire that up:

**Inline style** — pragmatic for one-off uses in handwritten content:
```html
<p>The drill is <span style="color: var(--repair);">under repair</span>.</p>
```

**Utility class** — if your fork has utility classes:
```html
<p>The drill is <span class="text-repair">under repair</span>.</p>
```

**Content-specific class** — if you write a class for your fork's status references:
```html
<p>The drill is <span class="status-text status-repair">under repair</span>.</p>
```

**Inside a heading or other element** — the colour cascades:
```html
<h3 style="color: var(--grow);">What's growing in May</h3>
```

The pattern doesn't pick one for you. Use the convention your fork uses elsewhere.

---

## Reading the colour

Activity-coloured text on `--surface` (the default page background) passes WCAG AA contrast at the body text size in both light and dark modes. The token system was tuned for this — see the contrast notes in `tokens.css` section 1.

For text on `--surface-raised` (e.g. inside an editorial card), the contrast still passes AA but is slightly lower. If you find a specific activity colour reads thin on a raised surface in your fork, the fix is to bump the token's saturation in your `tokens.css`, not to add a workaround per-context.

**Don't rely on colour alone.** Screen readers don't see colour. Activity-coloured text must still read as status from the words themselves — "under repair", not just a red span saying "x". The badge component pairs colour with text labels by design; the unbadged variant should follow the same principle in its phrasing.

---

## Composition with display/badge

When badged, the pattern composes directly with the `display/badge` component. Use the four activity variants:

```html
<span class="badge badge-grow">Available</span>
<span class="badge badge-repair">Under repair</span>
<span class="badge badge-share">In use</span>
<span class="badge badge-make">In progress</span>
```

The badge handles the pill shape, the tinted background, the padding, the text colour. The activity-badge pattern handles the editorial decision: which activity colour, what wording, where in the content it sits.

See `display/badge/README.md` for the badge component's full documentation, including icons, sizing overrides, and the activity-vs-functional distinction.

---

## Composition with cards and tables

The activity-badge pattern appears most often inside two other patterns:

**Inside cards** — a card with a status slot typically uses a badged activity-badge:
```html
<div class="card-status-row">
  <div class="card-content">…</div>
  <span class="badge badge-repair">Under repair</span>
</div>
```

When the card itself carries status (the row's primary subject is the status, not just one of its fields), the card also gets an activity-coloured left-rule. The badge and the rule should use the same activity colour. See `patterns/cards/`.

**Inside table rows** — a table row with a status column typically uses a badged activity-badge in the cell:
```html
<tr class="tr-repair">
  <td>Bandsaw</td>
  <td>Workshop A</td>
  <td><span class="badge badge-repair">Under repair</span></td>
</tr>
```

Same convention: the row's `.tr-*` class and the badge's `.badge-*` class should match. See `display/table/README.md`.

---

## What this pattern is not

- **It's not a component.** No CSS ships with this pattern.
- **It's not the em-dash convention.** Earlier drafts of the system used an em-dash prefix on status text ("— Available", "— Under repair"). That convention was dropped — the activity colour alone, combined with clear text, communicates the status without the typographic gesture.
- **It's not for functional interface state.** Use `--success`, `--warning`, `--info`, `--error` for things the interface is reporting. Activity tokens are for things the community is doing.

---

## See also

- `display/badge/README.md` — the badge component this pattern composes with
- `display/table/README.md` — table row status uses the same `.tr-*` convention
- `patterns/cards/README.md` — cards composing activity-badges in status slots
- `tokens.css` section 1 — the activity colour definitions and the activity-vs-functional distinction
