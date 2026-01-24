# Workflow: CSS Expert

<required_reading>
**Read these reference files NOW:**
1. references/css-modern.md
2. references/responsive-design.md
3. references/animation-motion.md
</required_reading>

<context>
Modern CSS (2025) is powerful. Container queries, `:has()`, subgrid, and view transitions are production-ready. Know when to use CSS vs JavaScript for animations and interactions.
</context>

<process>

<step name="understand-the-task">
Clarify the CSS challenge:
- Layout problem (Grid, Flexbox, positioning)?
- Modern CSS feature implementation?
- Animation or transition?
- Responsive design?
- Browser compatibility concern?
</step>

<step name="layout-solutions">
**Flexbox** - 1D layouts (row OR column):
```css
.flex-container {
  display: flex;
  gap: 1rem;
  align-items: center;
  justify-content: space-between;
}
```

**Grid** - 2D layouts (rows AND columns):
```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
}
```

**Subgrid** - Child aligns to parent grid:
```css
.parent {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
}

.child {
  display: grid;
  grid-column: span 3;
  grid-template-columns: subgrid; /* Inherits parent's columns */
}
```
</step>

<step name="container-queries">
Component-based responsive design:

```css
/* Define containment */
.card-container {
  container-type: inline-size;
  container-name: card;
}

/* Style based on container size, not viewport */
@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 200px 1fr;
  }
}

@container card (max-width: 399px) {
  .card {
    display: flex;
    flex-direction: column;
  }
}
```
</step>

<step name="has-selector">
Parent selector - style parent based on children:

```css
/* Card with image gets different layout */
.card:has(img) {
  grid-template-columns: 200px 1fr;
}

/* Form group with invalid input */
.form-group:has(:invalid) {
  border-color: red;
}

/* Nav item with active link */
.nav-item:has(.active) {
  background: var(--color-primary-light);
}
```
</step>

<step name="modern-colors">
Use modern color spaces for better accuracy:

```css
:root {
  /* OKLCH - perceptually uniform */
  --color-primary: oklch(0.7 0.15 250);
  --color-primary-hover: oklch(0.65 0.15 250);

  /* Color mixing for opacity */
  --color-overlay: color-mix(in oklch, var(--color-primary) 50%, transparent);
}
```
</step>

<step name="animations">
CSS transitions and animations:

```css
/* Smooth transitions */
.button {
  transition: transform 150ms ease-out, box-shadow 150ms ease-out;
}

.button:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

/* Keyframe animations */
@keyframes fade-in {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.animate-in {
  animation: fade-in 300ms ease-out forwards;
}
```
</step>

<step name="view-transitions">
Page transitions (View Transitions API):

```css
/* Define transition names */
.page-title {
  view-transition-name: page-title;
}

/* Customize the transition */
::view-transition-old(page-title) {
  animation: fade-out 200ms ease-out;
}

::view-transition-new(page-title) {
  animation: fade-in 200ms ease-out;
}
```
</step>

<step name="progressive-enhancement">
Feature detection:

```css
/* Check for feature support */
@supports (container-type: inline-size) {
  .card-container {
    container-type: inline-size;
  }
}

@supports selector(:has(*)) {
  .card:has(img) {
    /* Modern layout */
  }
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```
</step>

<step name="verify">
Test the CSS:
1. Check browser DevTools for computed styles
2. Test responsive behavior
3. Test with `prefers-reduced-motion`
4. Verify in multiple browsers if using newer features
5. Check for specificity conflicts
</step>

</process>

<anti_patterns>
Avoid:
- Using `!important` (fix specificity instead)
- Magic numbers (use CSS custom properties)
- Over-nesting (keep selectors flat)
- Ignoring the cascade (understand specificity)
- Assuming all browsers support new features (use `@supports`)
</anti_patterns>

<success_criteria>
- Layout works with CSS Grid or Flexbox (no floats/hacks)
- Container queries used for component-level responsiveness
- Modern color spaces (oklch) for better color accuracy
- Animations respect `prefers-reduced-motion`
- Feature detection with `@supports` for progressive enhancement
- No specificity wars or `!important` abuse
</success_criteria>
