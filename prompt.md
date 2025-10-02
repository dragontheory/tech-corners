**Prompt for GitHub Copilot Agent**

Refactor the repository **dragontheory/tech-corners** into a **CSS-only, markup-free, dependency-free** solution that draws animated “tech corners” around **any HTML element** (opt-in and auto modes) and additionally provides an optional **cursor** variant — all using **modern and experimental CSS** (no JS, no extra HTML wrappers, no IDs or classes). Follow D7460N architectural rules throughout.

---

### Scope & Objectives

1. **CSS-Only Corners (Element Variant)**

   * Implement corners via **pure CSS** using a combination of:

     * Multiple `background` layers with `conic-gradient()` and `linear-gradient()` to render sharp L-corners in all four corners.
     * `mask`, `clip-path`, or `border-image` where helpful.
     * `outline`/`outline-offset` for subtle halo if needed.
   * Corners must support:

     * Hover/focus states via `:hover`, `:focus-visible`, `:focus-within`, `:target`, and `:has()` patterns.
     * Progressive enhancement with `@supports` blocks (e.g., `@supports (selector(:has(*)))`, `@supports (background: conic-gradient(...) )`).
     * **No additional DOM**: **no** extra elements, wrappers, pseudo-helper markup, IDs, classes, inline styles, or JS event listeners.
     * Works against **any semantic element** without adding attributes: default apply on interactive/semantic elements (e.g., `a`, `button`, `summary`, `input`, `select`, `textarea`, `[role="button"]`, `[tabindex]`) and on **opt-in** patterns detailed below.
   * Add **stateful variations** managed by CSS only:

     * Idle (no corners or faint).
     * Hover/focus (animated corners appear).
     * Selected/pressed/expanded (bolder corners), using `:active`, `[aria-pressed="true"]`, `[aria-expanded="true"]`, `details[open]`, etc.
   * Provide **size and density** options using **custom properties** (`--corner-size`, `--corner-thickness`, `--corner-gap`, `--corner-radius`, `--corner-speed`, `--corner-color`).
   * Support **reduced motion** via `@media (prefers-reduced-motion: reduce)`.

2. **Opt-In Without Classes/IDs**

   * Because classes/IDs and inline styles are not allowed, provide **attribute-free** opt-in selectors that remain semantic:

     * Target **landmark/interactive semantics** (e.g., `a`, `button`, `summary`, `label`, `nav :where(a,button)`, `:where(th,td):where([scope])` for tables).
     * Provide an **authoring hook via pseudo-classes** only:

       * `:focus-visible`, `:target`, `:has(> input:checked)`, `:has(> :where(img,svg))` for media cards, etc.
     * Also include a universal **manual opt-in** switch using **custom property as a property** fallback:

       * Use `@property --corners;` then gate styles under `:where(*):where([style*="--corners: on"])` **and** (for non-inline usage) a stylesheet-only fallback: `:where(article:has(h1), section:has(h2), figure:has(img), li:has(a))`.
       * Note: keep **all** examples in the demo showing opt-in patterns without relying on classes/IDs.

3. **Cursor Variant (CSS-Only)**

   * Supply a **cursor** option using `cursor: url("data:image/svg+xml,...") [hotx] [hoty], auto;` with SVG data-URL encoding (escaped safely).
   * Provide at least **three sizes** and support `--corner-color` via baked cursor variants (light/dark/auto).
   * Document **limitations** (OS scaling, hotspot size limits, fallback to `cursor: crosshair;`).
   * **No JS**; CSS only.

4. **Accessibility & UX**

   * Meet **WCAG/Section 508** considerations:

     * Do not rely on color alone for state; maintain adequate contrast of corners vs. backgrounds.
     * Respect user motion settings.
     * Focus indication must remain visible and meet contrast guidance even if corners are disabled (e.g., keep a default focus ring as fallback).
   * Ensure corners never obscure text/click targets (use `background-origin: padding-box` and `background-clip`/`mask` to avoid overlap).

5. **Architecture (D7460N Rules)**

   * **Separation of Concerns**: HTML = semantic structure only; CSS = UI/state logic; JS = not used here.
   * **No classes, no IDs, no inline styles** in shipped examples (except where demonstrating `--corners` opt-in via inline style **as a documented exception**).
   * **CSS-Driven UI** using `:has()`, container queries, and native states.
   * **Scroll Control**: Ensure the corners don’t force overflow; maintain `overflow` rules consistent with D7460N’s scroll guidance.
   * **Holy Grail layout** for the demo page via semantic `<nav>`, `<main><article>`, `<aside>`; one `<article>` at a time.

6. **Progressive Enhancement & Fallbacks**

   * Layered design:

     * **@layer base, corners, states, reduce-motion, demo**.
     * **Base**: safe focus ring.
     * **Corners**: advanced gradients/masks.
     * **States**: hover/focus/selected.
     * **Reduce-motion**: time/transition reductions.
     * **Demo**: showcase only; separate file.
   * Provide **fallback visual** (e.g., simple focus outline) when `@supports` checks fail.
   * Include a dedicated `fallbacks.css` that shows a pseudo-element message when key host elements are missing (per project notes).

7. **Performance**

   * Avoid heavy blur/filters. Prefer GPU-friendly transforms if animation is used.
   * Keep `background` layers minimal and shareable via custom props.
   * Ensure no layout thrash; rely on `opacity`, `transform`, and gradient steps where needed.
   * Test with large lists (100+ items) to verify acceptable paint cost.

---

### Deliverables

* `assets/css/corners.css` — production stylesheet (commented, compact).
* `assets/css/fallbacks.css` — structural fallback messaging (no “awaiting data” styling).
* `assets/css/demo.css` — demo-only visuals, isolated via `@layer demo`.
* `index.html` — **semantic** demo page (no extra wrappers, no IDs/classes), showcasing:

  * Buttons, links, cards (`article`, `section`, `figure`), form controls, table headers/cells, nav menus, list items.
  * States: hover, focus-visible, selected/expanded, target.
  * Container query example (corners adjust thickness/size by container).
* `README.md` — concise documentation:

  * What it is, how it works, supported features.
  * How to enable auto mode (on interactive semantics) and opt-in patterns without classes/IDs.
  * How to switch sizes/colors via custom properties.
  * How to enable the **cursor** variant (with caveats).
  * Browser support table and `@supports` guards.
* `CHANGELOG.md` — summary of refactor and breaking changes.
* **Optional** `corners.theme.css` — light/dark/auto theming using `color-scheme` and `light-dark()`.

---

### CSS Implementation Details (Guidance)

* Register tunables:

  ```css
  @property --corner-size { syntax: "<length>"; inherits: true; initial-value: 0.5rem; }
  @property --corner-thickness { syntax: "<length>"; inherits: true; initial-value: 2px; }
  @property --corner-gap { syntax: "<length>"; inherits: true; initial-value: 2px; }
  @property --corner-radius { syntax: "<length-percentage>"; inherits: true; initial-value: 0; }
  @property --corner-speed { syntax: "<time>"; inherits: true; initial-value: 160ms; }
  @property --corner-color { syntax: "<color>"; inherits: true; initial-value: currentColor; }
  ```
* Core corner drawing with layered backgrounds (example pattern to be generalized):

  ```css
  /* Applied to eligible elements via semantic/pseudo-class selectors */
  :where(a, button, summary, input, select, textarea, [role="button"], [tabindex])
  {
    background:
      /* top-left */     conic-gradient(from 180deg at top left, var(--corner-color) 90deg, transparent 0) 0 0 / var(--corner-size) var(--corner-size) no-repeat,
      /* top-right */    conic-gradient(from 270deg at top right, var(--corner-color) 90deg, transparent 0) 100% 0 / var(--corner-size) var(--corner-size) no-repeat,
      /* bottom-left */  conic-gradient(from  90deg at bottom left, var(--corner-color) 90deg, transparent 0) 0 100% / var(--corner-size) var(--corner-size) no-repeat,
      /* bottom-right */ conic-gradient(from   0deg at bottom right, var(--corner-color) 90deg, transparent 0) 100% 100% / var(--corner-size) var(--corner-size) no-repeat;
    transition: background-size var(--corner-speed), outline-offset var(--corner-speed);
    outline: max(1px, var(--corner-thickness)) solid transparent;
    outline-offset: calc(var(--corner-gap) + 1px);
    background-origin: padding-box;
  }

  /* Animate corners on hover/focus-visible */
  :where(a, button, summary, input, select, textarea, [role="button"], [tabindex]):is(:hover, :focus-visible) {
    background-size:
      calc(var(--corner-size) + var(--corner-gap)) calc(var(--corner-size) + var(--corner-gap)),
      calc(var(--corner-size) + var(--corner-gap)) calc(var(--corner-size) + var(--corner-gap)),
      calc(var(--corner-size) + var(--corner-gap)) calc(var(--corner-size) + var(--corner-gap)),
      calc(var(--corner-size) + var(--corner-gap)) calc(var(--corner-size) + var(--corner-gap));
    outline-color: color-mix(in oklab, var(--corner-color) 60%, transparent);
  }

  /* Reduced motion */
  @media (prefers-reduced-motion: reduce) {
    :where(a, button, summary, input, select, textarea, [role="button"], [tabindex]) {
      transition: none;
    }
  }
  ```
* **Container queries** to scale corner thickness/size responsively.
* **@supports** blocks to fall back to simple `outline` when `conic-gradient` or `:has()` isn’t available.
* **Cursor variant** example (generate Light/Dark SVG data URLs; do not inline them here in README):

  ```css
  :root { --corner-cursor: url("data:image/svg+xml;charset=utf8,<svg ...>") 8 8, auto; }
  html { cursor: var(--corner-cursor); }
  ```

---

### Acceptance Criteria

* ✅ No extra HTML elements, no IDs/classes, no JS. CSS only.
* ✅ Corners appear and animate on hover/focus for eligible elements, and via semantic states (`:target`, `:has()` relationships) where meaningful.
* ✅ Works on **any** semantic element in the demo without adding attributes.
* ✅ Cursor variant provided with at least three sizes and light/dark presets.
* ✅ Progressive enhancement with graceful fallback when experimental features aren’t available.
* ✅ Accessibility: visible focus indicator with adequate contrast even when corners are disabled.
* ✅ Clean repo structure, updated docs, and a minimal, semantic demo.

---

**Verification has been completed. Compliance with D7460N rules and instructions is affirmed.**
