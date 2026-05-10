# Testing

Every component in seedbank.design must pass the three criteria below before it ships. They are the operational form of design principle 01 — *Designed for the edges, not the centre*. A component that fails any of them does not ship; it gets revised or dropped.

These criteria are deliberately minimal. They cover the failure modes that matter most for the communities this system is built for, without imposing a testing burden that excludes contributors without specialist tooling. Anyone can run all three checks with a browser and ten minutes.

Per-component testing notes live in each component's `notes.md`, recording how the component was tested and what was found. The notes file is the receipt; this document is the spec.

---

## The three criteria

### 01 — Renders correctly without CSS loaded

The component must be usable when its stylesheet fails to load. Semantic HTML carries meaning the browser can render on its own; if the component depends on CSS to communicate state, hierarchy, or grouping, it has failed.

This is not a degradation test — it is a structural test. A button must read as a button. A label must be associated with its input. A heading must be a heading. A list must be a list.

**How to check:**
1. Open the component's reference HTML page in a browser
2. Open developer tools, disable all stylesheets (or use a "disable styles" browser extension)
3. The page should remain comprehensible — every interactive element should still work, every relationship between elements should still be readable, and no content should be hidden

**What this rules out:**
- Icon-only controls (the icon is a stylesheet artefact; the label is in the markup)
- State communicated by colour alone
- Layout-dependent meaning (e.g. proximity implying grouping without semantic markup)
- Hidden text revealed by hover

### 02 — Works at 200% zoom

The component must remain usable when browser zoom is set to 200%. This is the WCAG 2.1 baseline (success criterion 1.4.4) and the practical floor for users with low vision.

Usability at 200% means: no content is clipped, no interactive element overlaps another, no text becomes unreadable, no layout breaks in a way that hides functionality. Reflow is expected; breakage is not.

**How to check:**
1. Open the component's reference HTML page in a browser at default zoom
2. Increase zoom to 200% (Ctrl/Cmd + several times, or set explicitly)
3. Every state shown on the page should remain reachable, readable, and operable
4. No horizontal scrollbar should appear unless the component is intrinsically wide (a table with many columns)

**What this rules out:**
- Fixed-pixel widths on text containers
- Absolute positioning that doesn't reflow
- Truncation that hides essential content
- Tightly-packed controls that overlap when scaled

### 03 — Works on a 320px viewport

The component must remain usable on a 320px-wide viewport. This is the smallest viewport width the system commits to supporting — older Android devices, older iPhones, embedded browsers, and constrained environments where the viewport is narrow by hardware limit, not user choice.

**How to check:**
1. Open the component's reference HTML page in a browser
2. Open developer tools, enable device toolbar / responsive mode
3. Set viewport width to 320px
4. Every state shown on the page should remain reachable, readable, and operable
5. No horizontal scroll should be required to access core functionality

**What this rules out:**
- Multi-column layouts that don't collapse
- Fixed-width containers wider than 320px
- Side-by-side patterns that assume desktop space
- Touch targets smaller than ~44px square in interactive elements

---

## What this checklist does not cover

These three criteria are necessary but not sufficient. Other things matter — keyboard navigation, screen reader behaviour, print rendering, dark mode, high-contrast mode, slow connections — and components should be tested against them where relevant. They are not in the standing checklist because either (a) they are covered by the design principles already and don't need re-testing per component, or (b) they require tooling that not every contributor has.

The standing checklist is the floor, not the ceiling. A component that passes all three criteria can still be a bad component. A component that fails any one of them is not ready.

---

*seedbank.design — licensed under MIT (code) and CC BY-SA 4.0 (documentation and design assets). This file is a design system asset — it ships with every clone. Forks are encouraged to adapt it; adaptations should be documented.*
