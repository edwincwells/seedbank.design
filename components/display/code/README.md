# code

Inline `<code>` and block `<pre><code>` treatments for displaying code. Applied via element selectors — no class needed. No syntax highlighting (that's a composition concern for forks).

---

## What it styles

- `<code>` — inline code in running prose
- `<pre>` — block containers for multi-line code
- `<pre><code>` — the nested combination that's the conventional markup for code blocks

The component applies via element selectors, not classes. This is a deliberate inconsistency with `display/badge` and `display/table` (which are class-based opt-ins within the display layer). The restraint of the code treatment makes it safe to apply globally — every `<code>` in a fork's content gets the styling without needing to remember a class.

---

## When to use it

- **Inline `<code>`** for class names, token names, filenames, function calls, short snippets in running prose: "the `--action` token", "set `aria-busy="true"`", "open `tokens.css`"
- **`<pre><code>`** for multi-line code examples: CSS rules, markup patterns, shell commands, JS snippets

When *not* to use it:

- **Don't use `<code>` for stylistic monospace.** It's semantic — screen readers announce the content as code. If you want monospace text for visual effect, write a class.
- **Don't use `<code>` inside `<pre>` for highlighting.** The two together render as the standard code block. Per-token highlighting needs syntax highlighting (see below).
- **Don't use `<pre>` alone for block code.** It works without the nested `<code>`, but `<pre><code>` is the conventional accessible markup — screen readers announce it as a code block, not just "preformatted text".

---

## Why no syntax highlighting

The system doesn't ship syntax highlighting. Reasons:

1. **It requires JS or a build step.** Every credible highlighter (Prism, highlight.js, Shiki, Pygments) needs runtime JavaScript or server-side processing. The system has held a strict no-JS-in-components line.
2. **It's a fork concern.** Different forks will document different things — CSS, HTML, shell, Python, configuration files. Each language wants its own grammar. Shipping highlighting in the system means picking a subset of languages and a colour scheme that may not fit every fork.
3. **It's composable.** Forks that want highlighting layer it on. The system provides the container; the fork fills it however it likes.

If your fork's documentation needs highlighting, pick a library that suits the languages you document and the deployment context (static-site-friendly options like Shiki work without runtime JS).

---

## Inline treatment

```html
<p>The system's primary action colour is set via the <code>--action</code> token.</p>
```

- `--font-mono` (Plex Mono)
- `0.9em` size — em-relative so it scales with surrounding text
- `--surface-sunken` background tint with `--space-1` horizontal padding
- `--radius-sm` corners
- `word-break: break-word` for long content (URLs, paths)

**The 0.9em shrink.** Mono fonts render visually larger than sans at the same point size. The 0.9em multiplier equalises them so inline code doesn't feel oversized against its prose context. Because it's relative (em, not px), it scales correctly inside any container — a `<code>` inside an `<h1>` shrinks relative to the heading size, not to the body default.

**The tinted background.** Matches the convention from documentation sites (MDN, GitHub, Stripe docs). Reads clearly as code against prose. The `--surface-sunken` tone is the same recessed register used by text-input and checkbox backgrounds — it signals "different from prose" without competing with badge tints or pulling weight from surrounding content.

---

## Block treatment

```html
<pre tabindex="0"><code>.badge-grow {
  background: color-mix(in srgb, var(--grow) 15%, var(--surface));
  color: var(--grow);
}</code></pre>
```

- `--font-mono`
- `--text-sm` (13px) — code lines are typically longer than prose lines, so stepping down a size fits more characters per row
- `--surface-sunken` background
- `--space-3 --space-4` padding (12px / 16px)
- `--radius-md` corners — same as buttons and inputs; code blocks are a "software" element, not editorial
- `overflow-x: auto` for lines that exceed the container width
- `white-space: pre` to preserve the formatting authors put into the code
- No `max-width` — code blocks are deliberately full-width in their container

**The `<pre><code>` nesting.** Always nest a `<code>` inside the `<pre>`. The `<pre>` provides the block container; the `<code>` provides the semantic "this is code" signal for assistive tech. The component's CSS handles both — the inline tinted-background treatment doesn't apply inside `<pre>` (overridden via `pre code` selector).

**The `tabindex="0"` convention.** Code blocks with horizontal overflow need to be keyboard-focusable so users can arrow-scroll. Add `tabindex="0"` to the `<pre>` in your markup. The focus-ring foundation matches `[tabindex]:not([tabindex="-1"])` so the ring should apply — but see the caveat below.

**On focus rings.** Same caveat as `display/table`'s `.table-wrap`: on macOS Chrome by default, tab navigation skips non-form-element focusable elements unless "Press Tab to highlight each item on a webpage" is enabled. The element *is* focusable in principle (programmatic `.focus()` works), so keyboard accessibility isn't blocked — but the visual indicator behaviour on these elements is currently an open investigation across the system.

---

## Code inside other contexts

`<code>` is designed to work inside any prose context: badge content, button labels, table cells, list items, headings. The `0.9em` scales with the surrounding text; the tinted background and padding are subtle enough not to disrupt layout.

```html
<h2>The <code>--action</code> token in a heading</h2>
<p>The system uses <code>--grow</code> as the default action colour.</p>
<button>Edit <code>tokens.css</code></button>
```

All of these render correctly without any additional rules.

---

## Tokens used

| Token | Used for |
|---|---|
| `--font-mono` | Both inline and block code |
| `--text` | Code text colour |
| `--text-sm` | Block code size |
| `--surface-sunken` | Inline and block background tint |
| `--radius-sm` | Inline code corners |
| `--radius-md` | Block code corners |
| `--space-1`, `--space-3`, `--space-4` | Padding |
| `--border-hairline` | Print-mode border |
| `--leading-base` | Block code line height |

No values are hardcoded except the `0.9em` size shift on inline code (which is a relative ratio, not a token-mappable value).

---

## Accessibility

- **`<code>` is semantic.** Screen readers announce content as code, which is what you want for actual code references and not what you want for stylistic monospace text.
- **`<pre><code>` is the conventional markup.** Screen readers announce the block as a code block. Just `<pre>` alone announces as "preformatted text", which is less informative.
- **`tabindex="0"` on `<pre>`** makes the block keyboard-focusable so users can arrow-scroll horizontally. Add this to any `<pre>` that might overflow.
- **Contrast.** The mono text on `--surface-sunken` passes WCAG AA in both light and dark modes — the sunken background is light enough not to interfere with text legibility.

---

## How to use it in your fork

1. Include `tokens/tokens.css`
2. Include `components/foundations/focus-ring/focus-ring.css` (for focusable code blocks)
3. Include `components/display/code/code.css`
4. Write semantic HTML — `<code>` for inline, `<pre tabindex="0"><code>...</code></pre>` for blocks

---

## How to override

**To remove the inline tinted background** (forks that want pure-monospace inline code with no background):

```css
code {
  background: none;
  padding: 0;
  border-radius: 0;
}
```

**To use a different background colour** for code blocks (a dark-on-light treatment, for instance):

```css
pre {
  background: var(--text);
  color: var(--surface);
}
pre code {
  color: inherit;
}
```

**To add syntax highlighting:** layer a highlighting library on top of the component. The library will inject `<span>` elements with classes (e.g. `.token.keyword`, `.token.string`) inside the `<code>` block — write CSS for those classes in your fork. Don't override the base `<pre><code>` styles; the library and the component are designed to compose.

**To restrict inline `<code>` size shift:**

```css
code {
  font-size: 1em;  /* match surrounding text instead of 0.9em */
}
```

Be aware that mono fonts render visually larger than sans at the same point — you may want to also bump the size of inline code to compensate.

---

## Print rules

- Inline `<code>`: background removed, padding removed. Mono font kept so code is still distinguishable from surrounding prose.
- `<pre><code>` blocks: background removed, hairline border added for definition (same approach as badge). `white-space` changes to `pre-wrap` so long lines wrap rather than being clipped at the page edge.
- `page-break-inside: avoid` on `<pre>` keeps code blocks together where possible.

---

## Testing

See `notes.md` for the testing record against `TESTING.md` criteria.
