<overview>
Modern CSS features (2025) that are production-ready: container queries, :has() selector, subgrid, view transitions, cascade layers, and modern color spaces.
</overview>

<container_queries>
**Component-based responsive design:**

```css
/* Define containment on wrapper */
.card-container {
  container-type: inline-size;
  container-name: card;
}

/* Style based on container width, not viewport */
.card {
  display: flex;
  flex-direction: column;
  padding: 1rem;
}

@container card (min-width: 400px) {
  .card {
    flex-direction: row;
    gap: 1.5rem;
  }

  .card-image {
    width: 200px;
    flex-shrink: 0;
  }
}

@container card (min-width: 600px) {
  .card {
    padding: 1.5rem;
  }

  .card-title {
    font-size: 1.5rem;
  }
}
```

**Container query units:**
```css
.card-title {
  /* Size relative to container */
  font-size: clamp(1rem, 5cqi, 2rem);  /* cqi = container query inline */
}
```

**When to use:**
- Reusable components that appear in different contexts
- Sidebars, modals, cards that need to adapt
- Replace most media queries for component styling
</container_queries>

<has_selector>
**The parent selector - style based on children:**

```css
/* Card that contains an image gets different layout */
.card:has(img) {
  display: grid;
  grid-template-columns: 200px 1fr;
}

.card:not(:has(img)) {
  display: block;
}

/* Form group with invalid input */
.form-group:has(:invalid) {
  --border-color: var(--color-error);
}

.form-group:has(:focus) {
  --border-color: var(--color-primary);
}

/* Navigation item containing active link */
.nav-item:has(a.active) {
  background: var(--color-primary-light);
}

/* Disable submit when form has invalid fields */
form:has(:invalid) button[type="submit"] {
  opacity: 0.5;
  pointer-events: none;
}

/* Table row with checked checkbox */
tr:has(input:checked) {
  background: var(--color-selected);
}

/* Show clear button only when input has value */
.search-wrapper:has(input:not(:placeholder-shown)) .clear-btn {
  display: block;
}
```

**Combine with other selectors:**
```css
/* Element that is both conditions */
.card:has(img):has(.badge) {
  /* Has both image AND badge */
}

/* Element matching either condition */
.card:has(img, .video) {
  /* Has image OR video */
}
```
</has_selector>

<subgrid>
**Child elements align to parent grid:**

```css
.parent-grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  gap: 1rem;
}

.child-spanning-all {
  grid-column: 1 / -1;  /* Span all columns */
  display: grid;
  grid-template-columns: subgrid;  /* Inherit parent columns */
}

/* Now children of .child-spanning-all align to parent grid */
```

**Practical example - consistent card alignment:**
```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.5rem;
}

.card {
  display: grid;
  grid-template-rows: auto 1fr auto;  /* Image, content, footer */
  grid-row: span 3;
  grid-template-rows: subgrid;
}

/* Now all card images, content, and footers align across cards */
```
</subgrid>

<view_transitions>
**Native page transitions:**

```css
/* Opt elements into transitions */
.page-title {
  view-transition-name: page-title;
}

.hero-image {
  view-transition-name: hero-image;
}

/* Customize transition animations */
::view-transition-old(page-title) {
  animation: slide-out-left 300ms ease-out;
}

::view-transition-new(page-title) {
  animation: slide-in-right 300ms ease-out;
}

/* Default cross-fade for unnamed elements */
::view-transition-old(root) {
  animation: fade-out 200ms ease-out;
}

::view-transition-new(root) {
  animation: fade-in 200ms ease-out;
}
```

**JavaScript trigger:**
```js
// Start a view transition
document.startViewTransition(() => {
  // Update the DOM
  updateContent();
});

// With async
document.startViewTransition(async () => {
  await fetchAndUpdateContent();
});
```
</view_transitions>

<cascade_layers>
**Control specificity with @layer:**

```css
/* Define layer order (first = lowest priority) */
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; box-sizing: border-box; }
}

@layer base {
  body { font-family: var(--font-sans); }
  h1 { font-size: 2rem; }
}

@layer components {
  .button { /* component styles */ }
  .card { /* component styles */ }
}

@layer utilities {
  .sr-only { /* screen reader only */ }
  .truncate { /* text truncation */ }
}

/* Styles outside layers have highest specificity */
.override { /* will override layered styles */ }
```

**Tailwind uses layers automatically:**
```css
@import "tailwindcss";
/* Tailwind creates: @layer theme, base, components, utilities */
```
</cascade_layers>

<modern_colors>
**OKLCH - perceptually uniform color space:**

```css
:root {
  /* OKLCH: lightness, chroma, hue */
  --color-primary: oklch(0.6 0.15 250);  /* Blue */
  --color-success: oklch(0.65 0.2 145);  /* Green */
  --color-warning: oklch(0.8 0.15 85);   /* Yellow */
  --color-error: oklch(0.6 0.25 25);     /* Red */

  /* Predictable lightness adjustments */
  --color-primary-light: oklch(0.8 0.1 250);
  --color-primary-dark: oklch(0.4 0.15 250);
}

/* Color mixing for transparency */
.overlay {
  background: color-mix(in oklch, var(--color-primary) 50%, transparent);
}

/* Relative color syntax */
.hover-state {
  /* Darken by reducing lightness */
  background: oklch(from var(--color-primary) calc(l - 0.1) c h);
}
```

**Why OKLCH:**
- Perceptually uniform (10% lighter looks 10% lighter)
- Better for generating color scales
- Consistent saturation across hues
- Wide gamut support (P3 displays)
</modern_colors>

<modern_selectors>
```css
/* :is() - matches any in list (forgiving) */
:is(h1, h2, h3, h4):hover {
  color: var(--color-primary);
}

/* :where() - same as :is() but zero specificity */
:where(.card, .panel, .box) {
  padding: 1rem;  /* Easy to override */
}

/* :not() - enhanced to take selector list */
.item:not(.active, .disabled) {
  opacity: 0.7;
}

/* :focus-visible - keyboard focus only */
button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

/* :empty - empty elements */
.message:empty {
  display: none;
}
```
</modern_selectors>

<progressive_enhancement>
**Feature detection:**

```css
/* Container queries */
@supports (container-type: inline-size) {
  .card-wrapper {
    container-type: inline-size;
  }
}

/* :has() selector */
@supports selector(:has(*)) {
  .form-group:has(:invalid) {
    border-color: red;
  }
}

/* Subgrid */
@supports (grid-template-columns: subgrid) {
  .card {
    display: grid;
    grid-template-columns: subgrid;
  }
}

/* View transitions */
@supports (view-transition-name: test) {
  .page-title {
    view-transition-name: page-title;
  }
}
```

**Reduced motion:**
```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```
</progressive_enhancement>

<browser_support>
**Production-ready (2025):**
- Container queries: All modern browsers
- :has() selector: All modern browsers
- Subgrid: All modern browsers
- View transitions: Chrome, Edge, Safari (Firefox in progress)
- Cascade layers: All modern browsers
- OKLCH colors: All modern browsers

**Always add @supports for view transitions and cutting-edge features.**
</browser_support>
