<overview>
Tailwind CSS v4 (2025) patterns, configuration, and best practices. CSS-first configuration, design tokens with `@theme`, and utility-first approach done right.
</overview>

<tailwind_v4_setup>
**Minimal setup (no config file needed):**
```css
/* app.css */
@import "tailwindcss";
```

**With design tokens:**
```css
@import "tailwindcss";

@theme {
  /* Colors - use OKLCH for perceptual uniformity */
  --color-primary: oklch(0.6 0.15 250);
  --color-primary-light: oklch(0.8 0.1 250);
  --color-primary-dark: oklch(0.4 0.15 250);

  --color-secondary: oklch(0.65 0.12 180);
  --color-accent: oklch(0.7 0.18 30);

  --color-success: oklch(0.65 0.15 145);
  --color-warning: oklch(0.75 0.15 85);
  --color-error: oklch(0.6 0.2 25);

  /* Neutrals */
  --color-gray-50: oklch(0.98 0 0);
  --color-gray-100: oklch(0.95 0 0);
  --color-gray-200: oklch(0.9 0 0);
  --color-gray-300: oklch(0.8 0 0);
  --color-gray-400: oklch(0.65 0 0);
  --color-gray-500: oklch(0.5 0 0);
  --color-gray-600: oklch(0.4 0 0);
  --color-gray-700: oklch(0.3 0 0);
  --color-gray-800: oklch(0.2 0 0);
  --color-gray-900: oklch(0.12 0 0);

  /* Spacing scale (8px base) */
  --spacing-0: 0;
  --spacing-1: 0.25rem;  /* 4px */
  --spacing-2: 0.5rem;   /* 8px */
  --spacing-3: 0.75rem;  /* 12px */
  --spacing-4: 1rem;     /* 16px */
  --spacing-5: 1.25rem;  /* 20px */
  --spacing-6: 1.5rem;   /* 24px */
  --spacing-8: 2rem;     /* 32px */
  --spacing-10: 2.5rem;  /* 40px */
  --spacing-12: 3rem;    /* 48px */
  --spacing-16: 4rem;    /* 64px */

  /* Border radius */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);

  /* Typography */
  --font-sans: "Inter", system-ui, sans-serif;
  --font-mono: "JetBrains Mono", monospace;
}
```
</tailwind_v4_setup>

<dark_mode>
**Enabling dark mode:**
```css
@import "tailwindcss";

/* Class-based dark mode (default) */
@variant dark (&:where(.dark, .dark *));

/* Or system preference based */
@variant dark (&:where(@media (prefers-color-scheme: dark), .dark *));
```

**Using dark mode:**
```html
<div class="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">
  <h1 class="text-gray-900 dark:text-white">Title</h1>
  <p class="text-gray-600 dark:text-gray-400">Description</p>
</div>
```

**Toggle implementation:**
```tsx
function DarkModeToggle() {
  const [dark, setDark] = useState(false);

  useEffect(() => {
    document.documentElement.classList.toggle('dark', dark);
  }, [dark]);

  return (
    <button onClick={() => setDark(!dark)}>
      {dark ? '☀️' : '🌙'}
    </button>
  );
}
```
</dark_mode>

<responsive_patterns>
**Mobile-first approach:**
```html
<!-- Stack on mobile, row on tablet+ -->
<div class="flex flex-col md:flex-row gap-4">
  <div class="flex-1">Content</div>
  <div class="flex-1">Content</div>
</div>

<!-- Hide/show by breakpoint -->
<nav class="hidden lg:flex">Desktop navigation</nav>
<button class="lg:hidden">Mobile menu</button>

<!-- Responsive grid -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
  <!-- Cards -->
</div>

<!-- Responsive typography -->
<h1 class="text-2xl md:text-3xl lg:text-4xl font-bold">
  Responsive Heading
</h1>

<!-- Responsive spacing -->
<section class="px-4 md:px-8 lg:px-16 py-8 md:py-12 lg:py-16">
  Content
</section>
```

**Tailwind breakpoints:**
| Prefix | Min-width | Typical use |
|--------|-----------|-------------|
| `sm` | 640px | Large phones |
| `md` | 768px | Tablets |
| `lg` | 1024px | Laptops |
| `xl` | 1280px | Desktops |
| `2xl` | 1536px | Large screens |
</responsive_patterns>

<component_patterns>
**Button:**
```html
<button class="
  inline-flex items-center justify-center
  px-4 py-2
  bg-primary text-white
  rounded-md
  font-medium text-sm
  hover:bg-primary/90
  focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-primary focus-visible:ring-offset-2
  disabled:opacity-50 disabled:cursor-not-allowed
  transition-colors
">
  Button Text
</button>
```

**Card:**
```html
<div class="
  bg-white dark:bg-gray-800
  rounded-lg
  shadow-sm hover:shadow-md
  border border-gray-200 dark:border-gray-700
  p-6
  transition-shadow
">
  <h3 class="text-lg font-semibold text-gray-900 dark:text-white">Title</h3>
  <p class="mt-2 text-gray-600 dark:text-gray-400">Description text</p>
</div>
```

**Input with label:**
```html
<div class="space-y-2">
  <label class="block text-sm font-medium text-gray-700 dark:text-gray-300">
    Email
  </label>
  <input
    type="email"
    class="
      w-full px-3 py-2
      border border-gray-300 dark:border-gray-600
      rounded-md
      bg-white dark:bg-gray-800
      text-gray-900 dark:text-white
      placeholder-gray-400
      focus:outline-none focus:ring-2 focus:ring-primary focus:border-transparent
    "
    placeholder="you@example.com"
  />
</div>
```

**Badge:**
```html
<span class="
  inline-flex items-center
  px-2.5 py-0.5
  rounded-full
  text-xs font-medium
  bg-green-100 text-green-800
  dark:bg-green-900 dark:text-green-200
">
  Active
</span>
```
</component_patterns>

<utility_patterns>
**Centering:**
```html
<!-- Flexbox center -->
<div class="flex items-center justify-center h-screen">
  Centered content
</div>

<!-- Grid center -->
<div class="grid place-items-center h-screen">
  Centered content
</div>
```

**Truncate text:**
```html
<p class="truncate">Long text that will be truncated...</p>
<p class="line-clamp-2">Text clamped to 2 lines...</p>
```

**Aspect ratio:**
```html
<div class="aspect-video bg-gray-200">
  16:9 container
</div>
<div class="aspect-square bg-gray-200">
  1:1 container
</div>
```

**Transitions:**
```html
<button class="transition-colors duration-150 ease-out hover:bg-gray-100">
  Color transition
</button>
<div class="transition-transform duration-200 hover:scale-105">
  Scale transition
</div>
```
</utility_patterns>

<anti_patterns>
**Don't:**
```html
<!-- Too much @apply (defeats utility-first purpose) -->
<style>
.btn {
  @apply px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600;
}
</style>

<!-- Dynamic class construction (gets purged) -->
<div class={`bg-${color}-500`}>Won't work</div>

<!-- Inconsistent spacing -->
<div class="p-3 mt-5 mb-7">Magic numbers</div>
```

**Do:**
```tsx
// Extract to React component instead of @apply
function Button({ children }) {
  return (
    <button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
      {children}
    </button>
  );
}

// Use complete class names for dynamic styles
const colors = {
  red: 'bg-red-500',
  blue: 'bg-blue-500',
};
<div className={colors[color]}>Works</div>

// Consistent spacing from scale
<div class="p-4 mt-6 mb-8">From scale</div>
```
</anti_patterns>

<performance>
**Tailwind v4 is fast:**
- Full builds: up to 5x faster
- Incremental builds: 100x faster (microseconds)
- 35% smaller install size

**Production optimization:**
- Unused classes automatically removed
- No configuration needed for content paths (auto-detected)
- Use `@layer` for custom CSS specificity control

```css
@layer components {
  .custom-button {
    /* Your custom component styles */
  }
}

@layer utilities {
  .custom-utility {
    /* Your custom utility */
  }
}
```
</performance>
