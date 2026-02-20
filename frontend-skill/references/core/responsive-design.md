<overview>
Responsive design: mobile-first approach, breakpoints, fluid typography, flexible layouts, and ensuring great experiences across all devices.
</overview>

<mobile_first>
**Start with mobile, enhance for larger screens:**

```css
/* Mobile base styles (no media query) */
.container {
  padding: 1rem;
}

.grid {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

/* Tablet and up */
@media (min-width: 768px) {
  .container {
    padding: 2rem;
  }

  .grid {
    flex-direction: row;
    gap: 1.5rem;
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .container {
    padding: 3rem;
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

**Why mobile-first:**
- Progressive enhancement (add features, don't remove)
- Smaller CSS (base styles apply to all)
- Forces prioritization of content
- Better performance on mobile (load less CSS)
</mobile_first>

<breakpoints>
**Common breakpoints:**
| Name | Width | Use case |
|------|-------|----------|
| Mobile | < 640px | Phones |
| sm | 640px | Large phones |
| md | 768px | Tablets |
| lg | 1024px | Laptops |
| xl | 1280px | Desktops |
| 2xl | 1536px | Large screens |

**Tailwind breakpoint classes:**
```html
<!-- Column on mobile, row on tablet+ -->
<div class="flex flex-col md:flex-row gap-4">
  ...
</div>

<!-- 1 column mobile, 2 tablet, 3 desktop, 4 large -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
  ...
</div>

<!-- Hide on mobile, show on tablet+ -->
<nav class="hidden md:flex">Desktop nav</nav>
<button class="md:hidden">Mobile menu</button>
```

**Content-based breakpoints:**
Don't just use device widths. Break when the design breaks:
```css
/* Break when sidebar gets too cramped */
@media (min-width: 800px) {
  .layout {
    display: grid;
    grid-template-columns: 250px 1fr;
  }
}
```
</breakpoints>

<fluid_typography>
**Fluid scaling between breakpoints:**

```css
/* clamp(min, preferred, max) */
h1 {
  font-size: clamp(2rem, 5vw, 4rem);
}

p {
  font-size: clamp(1rem, 2vw, 1.25rem);
}

/* Fluid spacing */
section {
  padding: clamp(2rem, 5vw, 6rem);
}
```

**Tailwind fluid typography (custom):**
```css
@theme {
  --font-size-fluid-sm: clamp(0.875rem, 1.5vw, 1rem);
  --font-size-fluid-base: clamp(1rem, 2vw, 1.25rem);
  --font-size-fluid-lg: clamp(1.25rem, 3vw, 2rem);
  --font-size-fluid-xl: clamp(2rem, 5vw, 4rem);
}
```

**Viewport units:**
- `vw` - viewport width
- `vh` - viewport height
- `vmin` - smaller of vw/vh
- `vmax` - larger of vw/vh
- `dvh` - dynamic viewport height (accounts for mobile browser UI)
- `svh` - small viewport height
- `lvh` - large viewport height
</fluid_typography>

<flexible_layouts>
**Flexbox patterns:**
```css
/* Wrap to multiple rows */
.flex-wrap {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.flex-wrap > * {
  flex: 1 1 300px;  /* Grow, shrink, min 300px */
}

/* Sidebar layout */
.sidebar-layout {
  display: flex;
  flex-wrap: wrap;
}

.sidebar {
  flex: 1 1 200px;  /* Sidebar: min 200px */
}

.main {
  flex: 2 1 400px;  /* Main: min 400px, grows more */
}
```

**Grid patterns:**
```css
/* Auto-fit columns */
.auto-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
}

/* Auto-fill vs auto-fit */
/* auto-fill: Creates empty tracks */
/* auto-fit: Collapses empty tracks */

/* Responsive grid areas */
.page-grid {
  display: grid;
  gap: 1rem;
  grid-template-areas:
    "header"
    "nav"
    "main"
    "aside"
    "footer";
}

@media (min-width: 768px) {
  .page-grid {
    grid-template-columns: 200px 1fr;
    grid-template-areas:
      "header header"
      "nav main"
      "nav aside"
      "footer footer";
  }
}
```
</flexible_layouts>

<container_queries>
**Component-level responsiveness:**

```css
/* Container setup */
.card-container {
  container-type: inline-size;
  container-name: card;
}

/* Container query */
@container card (min-width: 400px) {
  .card {
    display: flex;
    flex-direction: row;
  }
}

@container card (max-width: 399px) {
  .card {
    display: flex;
    flex-direction: column;
  }
}
```

**When to use container queries vs media queries:**
- Container queries: Reusable components that appear in different contexts
- Media queries: Page-level layout changes

```html
<!-- Same component, different containers -->
<div class="card-container w-full">  <!-- Full width -->
  <Card />  <!-- Horizontal layout -->
</div>

<div class="card-container w-1/3">  <!-- Sidebar -->
  <Card />  <!-- Vertical layout -->
</div>
```
</container_queries>

<images>
**Responsive images:**
```html
<!-- srcset for resolution switching -->
<img
  src="image-800.jpg"
  srcset="
    image-400.jpg 400w,
    image-800.jpg 800w,
    image-1200.jpg 1200w,
    image-1600.jpg 1600w
  "
  sizes="
    (max-width: 600px) 100vw,
    (max-width: 1000px) 50vw,
    800px
  "
  alt="Description"
/>

<!-- Art direction with picture -->
<picture>
  <source
    media="(min-width: 1024px)"
    srcset="hero-wide.jpg"
  />
  <source
    media="(min-width: 640px)"
    srcset="hero-medium.jpg"
  />
  <img src="hero-mobile.jpg" alt="Hero" />
</picture>

<!-- Aspect ratio container -->
<div class="aspect-video relative">
  <img
    src="cover.jpg"
    alt="Cover"
    class="absolute inset-0 w-full h-full object-cover"
  />
</div>
```

**Next.js Image:**
```tsx
<Image
  src="/hero.jpg"
  alt="Hero"
  fill
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  className="object-cover"
/>
```
</images>

<touch_targets>
**Minimum sizes:**
- Apple: 44x44px
- Google Material: 48x48px
- WCAG: 24x24px minimum, 44x44px recommended

```css
/* Ensure touch-friendly buttons */
.button {
  min-height: 44px;
  min-width: 44px;
  padding: 0.75rem 1rem;
}

/* Spacing between targets */
.button + .button {
  margin-left: 0.5rem;  /* 8px minimum */
}

/* Make entire card clickable on mobile */
@media (max-width: 768px) {
  .card {
    cursor: pointer;
  }

  .card a::after {
    content: '';
    position: absolute;
    inset: 0;
  }
}
```
</touch_targets>

<testing>
**Test at these widths:**
- 320px (small phone)
- 375px (iPhone)
- 414px (large phone)
- 768px (tablet portrait)
- 1024px (tablet landscape / small laptop)
- 1280px (laptop)
- 1440px (desktop)
- 1920px (large desktop)

**Chrome DevTools device mode:**
- Toggle device toolbar: Ctrl+Shift+M (Cmd+Shift+M on Mac)
- Test specific devices
- Throttle network/CPU
- Simulate touch events

**Real device testing:**
- iOS Safari behaves differently than Chrome
- Test on actual phones/tablets when possible
- BrowserStack, LambdaTest for cross-browser
</testing>

<decorative_elements>
**Decorative images, characters, backgrounds that sit alongside content:**

The simplest approach wins. Don't micromanage breakpoints for decorative elements:

```css
/* BAD: Fragile calc positioning that breaks at every screen size */
.character {
  position: fixed;
  left: calc(50% - 980px);  /* Assumes ~1960px viewport */
}

/* BAD: 8 custom breakpoints trying to size decorative elements */
.character {
  width: 160px;           /* tiny at base */
  @media (min-width: 1440px) { width: 230px; }
  @media (min-width: 1600px) { width: 300px; }
  @media (min-width: 1728px) { width: 360px; }
  /* ...and so on forever */
}

/* BAD: Hardcoded min-height that pushes content below the fold */
.content-area {
  min-height: 520px;  /* Eats all vertical space on laptops */
}
```

```css
/* GOOD: Pin to edges, use z-index layering, clip overflow */
body {
  overflow-x: hidden;  /* Clip decorative elements at viewport edge */
}

.character {
  position: fixed;
  bottom: 0;
  z-index: 0;          /* Behind content */
  width: 400px;        /* One reasonable size */
}

.character-left { left: 0; }
.character-right { right: 0; }

.main-content {
  position: relative;
  z-index: 1;          /* Renders on top of decorative elements */
}

/* GOOD: Let content dictate height, not hardcoded min-height */
.content-area {
  padding: 1.5rem;  /* Padding provides breathing room */
  /* No min-height - content + padding = natural height */
}
```

**The z-index layering pattern:**
- Decorative elements: `z-index: 0` (behind everything)
- Main content: `z-index: 1` (renders on top)
- `overflow-x: hidden` on body clips excess at viewport edges
- Decorative elements naturally tuck behind the content box
- On wide screens: fully visible on the sides
- On narrow screens: partially clipped, partially behind the box
- NO breakpoint calculations needed

**Rules:**
1. Pin decorative elements to viewport edges (`left: 0` / `right: 0`)
2. Use z-index to layer content on top - don't try to avoid all overlap
3. Add `overflow-x: hidden` to body to prevent horizontal scrollbar
4. Never use `min-height` for aesthetic spacing - use padding instead
5. Let content drive height; buttons/CTAs should never be pushed below the fold
6. One or two breakpoints max for decorative elements (show/hide + one size bump)
</decorative_elements>

<responsive_checklist>
```markdown
Layout
- [ ] Content readable at 320px width
- [ ] No horizontal scroll
- [ ] Touch targets 44px+ minimum
- [ ] Adequate spacing between interactive elements
- [ ] All interactive elements (buttons, CTAs) visible without scrolling on common viewports
- [ ] No hardcoded min-height pushing content below the fold

Typography
- [ ] Text readable without zooming
- [ ] Line length comfortable (50-75 chars)
- [ ] Font size scales appropriately

Images
- [ ] Images scale proportionally
- [ ] srcset/sizes for resolution switching
- [ ] Art direction for different contexts

Decorative Elements
- [ ] Decorative elements use z-index layering (not breakpoint micromanagement)
- [ ] overflow-x: hidden prevents horizontal scrollbar from decorative overflow
- [ ] Decorative elements hidden on screens too small to show them meaningfully
- [ ] No calc()-based positioning that assumes a specific viewport width

Navigation
- [ ] Accessible on mobile (hamburger, bottom nav)
- [ ] Touch-friendly menu
- [ ] Clear visual hierarchy

Performance
- [ ] Fast on mobile networks
- [ ] Images optimized for device
- [ ] Code split for mobile bundle
```
</responsive_checklist>
