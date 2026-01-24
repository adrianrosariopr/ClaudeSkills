<overview>
GSAP is highly optimized, but animation performance ultimately depends on what you animate and how. This reference covers GPU acceleration, what properties to animate, and optimization techniques.
</overview>

<gpu_acceleration>
Modern browsers can offload certain CSS properties to the GPU for smooth 60fps animations.

<gpu_accelerated_properties>
**GPU-accelerated (animate these):**
- `transform` (x, y, scale, rotation, etc.)
- `opacity`

**NOT GPU-accelerated (avoid animating):**
- `left`, `top`, `right`, `bottom`
- `width`, `height`
- `margin`, `padding`
- `border`
- `font-size`
- `background-color` (triggers paint)
- `box-shadow` (triggers paint)
</gpu_accelerated_properties>

<force3d>
GSAP's `force3D` option controls GPU layer creation:

```javascript
// "auto" (default) - GPU during animation, releases after
gsap.to(".box", { x: 100 }); // force3D: "auto"

// true - Keep on GPU permanently
gsap.to(".box", { x: 100, force3D: true });

// false - Never use GPU (2D transforms only)
gsap.to(".box", { x: 100, force3D: false });
```

**When to use each:**
| force3D | Use Case |
|---------|----------|
| `"auto"` (default) | Most animations |
| `true` | Frequently animated elements (cursors, continuous animations) |
| `false` | Many simple animations, memory-constrained |
</force3d>

<will_change>
`will-change` hints to browsers about upcoming animations:

```javascript
// Add before heavy animation
gsap.set(".element", { willChange: "transform, opacity" });

// IMPORTANT: Remove after to free GPU memory
gsap.to(".element", {
  x: 100,
  onComplete: () => {
    gsap.set(".element", { willChange: "auto" });
  }
});
```

**Warning:** Too many elements with `will-change` hurts performance. Use sparingly.
</will_change>
</gpu_acceleration>

<property_mapping>
Map layout properties to transform equivalents:

| Instead of | Use | Notes |
|------------|-----|-------|
| `left: 100px` | `x: 100` | translateX |
| `top: 50px` | `y: 50` | translateY |
| `width: 200px` | `scaleX: 2` | or animate wrapper |
| `height: 150px` | `scaleY: 1.5` | or animate wrapper |
| `margin-left: 20px` | `x: 20` | translateX |

```javascript
// BAD - triggers layout
gsap.to(".box", { left: 100, top: 50, width: 200 });

// GOOD - GPU accelerated
gsap.to(".box", { x: 100, y: 50, scaleX: 2 });
```
</property_mapping>

<css_conflicts>
**CSS transitions conflict with GSAP:**

```css
/* BAD - browser and GSAP fight for control */
.animated {
  transition: transform 0.3s ease;
}

/* GOOD - let GSAP handle everything */
.animated {
  /* no transition */
}
```

**CSS animations also conflict:**
```css
/* BAD */
.box { animation: pulse 2s infinite; }

/* GOOD - use GSAP for all animation */
```
</css_conflicts>

<stagger_vs_loops>
```javascript
const items = gsap.utils.toArray(".item");

// BAD - creates 100 separate tweens
items.forEach((item, i) => {
  gsap.to(item, { y: 0, delay: i * 0.05 });
});

// GOOD - single optimized tween
gsap.to(items, {
  y: 0,
  stagger: 0.05
});
```
</stagger_vs_loops>

<quickto>
For frequently updating values (mousemove, scroll), use `gsap.quickTo`:

```javascript
// BAD - creates new tween every event
document.addEventListener("mousemove", (e) => {
  gsap.to(cursor, { x: e.clientX, y: e.clientY, duration: 0.3 });
});

// GOOD - reuses single tween
const xTo = gsap.quickTo(cursor, "x", { duration: 0.3 });
const yTo = gsap.quickTo(cursor, "y", { duration: 0.3 });

document.addEventListener("mousemove", (e) => {
  xTo(e.clientX);
  yTo(e.clientY);
});
```
</quickto>

<throttling>
Throttle high-frequency events:

```javascript
// Throttle with requestAnimationFrame
let rafId;

document.addEventListener("mousemove", (e) => {
  if (rafId) return;

  rafId = requestAnimationFrame(() => {
    gsap.set(element, { x: e.clientX });
    rafId = null;
  });
});

// Or use GSAP ticker
gsap.ticker.add(() => {
  // Runs in sync with GSAP's render cycle
  // 60fps max
});
```
</throttling>

<scrolltrigger_performance>
```javascript
// Use scrub smoothing
scrollTrigger: {
  scrub: 1  // 1 second smooth catch-up
}

// Batch similar triggers
ScrollTrigger.batch(".card", {
  onEnter: batch => gsap.to(batch, { opacity: 1 }),
  start: "top 80%"
});

// Kill unused triggers
ScrollTrigger.getAll().forEach(t => {
  if (!document.body.contains(t.trigger)) {
    t.kill();
  }
});

// Refresh after dynamic content
ScrollTrigger.refresh();
```
</scrolltrigger_performance>

<dom_elements>
Limit DOM elements created by animations:

```javascript
// BAD - 500+ DOM elements for long paragraph
new SplitText(".paragraph", { type: "chars" });

// GOOD - ~20 DOM elements
new SplitText(".paragraph", { type: "lines" });
```
</dom_elements>

<measuring_performance>
**DevTools Performance Panel:**
1. Record during animation
2. Look for:
   - Frame time >16ms (causes jank)
   - Layout events (purple) during animation
   - Frequent paint events (green)

**GSAP ticker debugging:**
```javascript
gsap.ticker.add(() => {
  const frameTime = gsap.ticker.deltaRatio() * 16.67;
  if (frameTime > 20) {
    console.warn("Slow frame:", frameTime.toFixed(2), "ms");
  }
});
```
</measuring_performance>

<optimization_checklist>
- [ ] Animating only `transform` and `opacity`?
- [ ] No CSS `transition` on animated elements?
- [ ] Using `stagger` instead of loops?
- [ ] Using `gsap.quickTo` for frequent updates?
- [ ] `will-change` removed after animation?
- [ ] ScrollTrigger using `scrub` smoothing?
- [ ] Reasonable number of animated elements?
- [ ] No layout properties animated?
</optimization_checklist>
