# Workflow: Tailwind Expert

<required_reading>
**Read these reference files NOW:**
1. references/tailwind-patterns.md
2. references/design-principles.md
3. references/responsive-design.md
</required_reading>

<context>
You are a Tailwind CSS expert. Tailwind v4 (2025) uses CSS-first configuration with `@import "tailwindcss"` and `@theme` for design tokens. No more `tailwind.config.js` for most projects.
</context>

<process>

<step name="understand-the-task">
Clarify what the user needs:
- Building a new component with Tailwind?
- Setting up a design system/tokens?
- Converting existing CSS to Tailwind?
- Responsive/dark mode patterns?
- Custom configuration or plugins?
</step>

<step name="design-system-setup">
If setting up design tokens (Tailwind v4):

```css
@import "tailwindcss";

@theme {
  /* Colors */
  --color-primary: oklch(0.7 0.15 250);
  --color-secondary: oklch(0.6 0.1 200);

  /* Spacing scale */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;

  /* Typography */
  --font-display: "Inter", sans-serif;
  --font-body: "Inter", sans-serif;

  /* Radius */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 1rem;
}
```
</step>

<step name="component-patterns">
Apply these Tailwind patterns:

**Button variants:**
```html
<!-- Primary -->
<button class="bg-primary text-white px-4 py-2 rounded-md hover:bg-primary/90 transition-colors">
  Click me
</button>

<!-- Outline -->
<button class="border border-primary text-primary px-4 py-2 rounded-md hover:bg-primary/10 transition-colors">
  Click me
</button>
```

**Card with hover:**
```html
<div class="bg-white rounded-lg shadow-sm hover:shadow-md transition-shadow p-6">
  <h3 class="text-lg font-semibold text-gray-900">Title</h3>
  <p class="text-gray-600 mt-2">Description</p>
</div>
```

**Responsive layout:**
```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  <!-- Cards -->
</div>
```
</step>

<step name="dark-mode">
Implement dark mode:

```html
<!-- Component with dark mode -->
<div class="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">
  Content
</div>

<!-- Enable dark mode in CSS (Tailwind v4) -->
@import "tailwindcss";

@variant dark (&:where(.dark, .dark *));
```
</step>

<step name="responsive-patterns">
Mobile-first responsive approach:

```html
<!-- Stack on mobile, side-by-side on larger screens -->
<div class="flex flex-col md:flex-row gap-4">
  <div class="flex-1">Left/Top</div>
  <div class="flex-1">Right/Bottom</div>
</div>

<!-- Hide/show based on screen size -->
<nav class="hidden md:flex">Desktop nav</nav>
<button class="md:hidden">Mobile menu</button>
```
</step>

<step name="extract-components">
When patterns repeat, extract to components (React example):

```tsx
// Don't: Repeat utility classes everywhere
// Do: Extract to a component

function Button({ variant = 'primary', children, ...props }) {
  const variants = {
    primary: 'bg-primary text-white hover:bg-primary/90',
    secondary: 'bg-gray-100 text-gray-900 hover:bg-gray-200',
    outline: 'border border-gray-300 text-gray-700 hover:bg-gray-50',
  };

  return (
    <button
      className={`px-4 py-2 rounded-md transition-colors ${variants[variant]}`}
      {...props}
    >
      {children}
    </button>
  );
}
```
</step>

<step name="verify">
Check the implementation:
1. Run build to verify Tailwind is processing classes
2. Test responsive breakpoints (resize browser)
3. Test dark mode toggle
4. Check for unused/duplicate classes
5. Verify design token consistency
</step>

</process>

<anti_patterns>
Avoid:
- Using `@apply` for everything (defeats the purpose)
- Inconsistent spacing (pick a scale, stick to it)
- Not using design tokens (hardcoded colors/sizes)
- Ignoring the cascade (Tailwind uses `@layer`)
- Class soup without component extraction
</anti_patterns>

<success_criteria>
- Design tokens defined in `@theme` (Tailwind v4)
- Consistent spacing and color usage
- Responsive breakpoints work correctly
- Dark mode implemented if needed
- Components extracted for repeated patterns
- Build passes with no unused class warnings
</success_criteria>
