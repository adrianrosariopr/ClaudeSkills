# Workflow: Optimize Animation Performance

<required_reading>
**Read these reference files NOW:**
1. references/performance.md
2. references/anti-patterns.md
3. references/core-concepts.md
</required_reading>

<process>

<step name="measure-first">
## Step 1: Measure Current Performance

**DevTools Performance Panel:**
1. Open DevTools → Performance tab
2. Click Record (or Cmd+Shift+E)
3. Trigger animations (scroll, hover, etc.)
4. Stop recording
5. Analyze:
   - **Frame rate**: Green bars should be consistent 60fps
   - **Long tasks**: Red/yellow blocks indicate jank
   - **Layout/Reflow**: Purple events (bad for animation)
   - **Paint**: Green events (expensive if frequent)

**GSAP's built-in debugging:**
```javascript
// Log render time
gsap.ticker.add(() => {
  console.log("Frame time:", gsap.ticker.deltaRatio() * 16.67, "ms");
});
```
</step>

<step name="fix-layout-properties">
## Step 2: Replace Layout Properties with Transforms

**Properties that trigger layout (AVOID):**
- `left`, `top`, `right`, `bottom`
- `width`, `height`
- `margin`, `padding`
- `border-width`

**Transform equivalents (USE):**
```javascript
// BAD
gsap.to(".box", {
  left: 100,      // triggers layout
  top: 50,        // triggers layout
  width: 200,     // triggers layout
  height: 150     // triggers layout
});

// GOOD
gsap.to(".box", {
  x: 100,         // transform: translateX
  y: 50,          // transform: translateY
  scaleX: 1.5,    // transform: scaleX (instead of width)
  scaleY: 1.2     // transform: scaleY (instead of height)
});
```

**For actual size changes, animate a wrapper:**
```javascript
// If you MUST change actual dimensions
gsap.to(".box-wrapper", {
  width: 200,
  height: 150,
  duration: 0.3
});

// But animate visual effects with transforms
gsap.to(".box-content", {
  scale: 1.1,
  duration: 0.3
});
```
</step>

<step name="remove-css-conflicts">
## Step 3: Remove CSS Conflicts

**Remove CSS transitions:**
```css
/* BAD */
.animated-element {
  transition: transform 0.3s, opacity 0.3s;
}

/* GOOD */
.animated-element {
  /* No transition - GSAP handles animation */
}
```

**Remove animations from animated properties:**
```css
/* BAD - CSS animation conflicts */
.box {
  animation: pulse 2s infinite;
}

/* Let GSAP handle it */
.box {
  /* No CSS animation */
}
```
</step>

<step name="optimize-gpu">
## Step 4: Optimize GPU Usage

**GSAP's force3D (default is "auto"):**
```javascript
// Default: GPU during animation, releases after
gsap.to(".box", { x: 100 }); // force3D: "auto" by default

// Keep on GPU (for frequently animated elements)
gsap.to(".frequently-animated", {
  x: 100,
  force3D: true  // stays on GPU
});

// Disable GPU (for simple animations with many elements)
gsap.to(".simple-move", {
  x: 100,
  force3D: false  // uses 2D transforms
});
```

**Use will-change sparingly:**
```javascript
// Add before heavy animation
gsap.set(".heavy-element", { willChange: "transform, opacity" });

// IMPORTANT: Remove after animation to free memory
gsap.to(".heavy-element", {
  x: 100,
  onComplete: () => {
    gsap.set(".heavy-element", { willChange: "auto" });
  }
});
```

**WARNING:** Too many elements with will-change causes performance issues.
</step>

<step name="reduce-complexity">
## Step 5: Reduce Animation Complexity

**Limit simultaneous animations:**
```javascript
// BAD - animating many properties
gsap.to(".box", {
  x: 100,
  y: 50,
  scale: 1.2,
  rotation: 45,
  opacity: 0.8,
  backgroundColor: "#ff0000",
  borderRadius: "50%",
  boxShadow: "0 10px 20px rgba(0,0,0,0.3)"
});

// GOOD - animate only what's needed
gsap.to(".box", {
  x: 100,
  opacity: 0.8
});
```

**Reduce DOM elements:**
```javascript
// BAD - SplitText on long paragraph into chars
const split = new SplitText(".long-paragraph", { type: "chars" });
// Creates 500+ DOM elements!

// GOOD - split into lines or words instead
const split = new SplitText(".long-paragraph", { type: "lines" });
// Creates ~20 DOM elements
```

**Use stagger amount, not individual tweens:**
```javascript
// BAD - creates 100 tweens
items.forEach((item, i) => {
  gsap.to(item, { y: 0, delay: i * 0.05 });
});

// GOOD - single tween with stagger
gsap.to(items, {
  y: 0,
  stagger: 0.05  // GSAP optimizes this internally
});
```
</step>

<step name="optimize-scrolltrigger">
## Step 6: Optimize ScrollTrigger

**Use scrub smoothing:**
```javascript
// Instant scrub (can feel jerky)
scrollTrigger: { scrub: true }

// Smoothed scrub (1 second catch-up)
scrollTrigger: { scrub: 1 }
```

**Batch ScrollTrigger creation:**
```javascript
// GOOD - batch similar triggers
ScrollTrigger.batch(".card", {
  onEnter: batch => gsap.to(batch, { opacity: 1, y: 0 }),
  start: "top 80%"
});
```

**Limit callbacks:**
```javascript
// Reduce callback frequency
scrollTrigger: {
  onUpdate: self => {
    // This fires on every scroll event
  }
}

// Better: use throttled updates
let lastProgress = 0;
scrollTrigger: {
  onUpdate: self => {
    if (Math.abs(self.progress - lastProgress) > 0.01) {
      // Only update on significant change
      lastProgress = self.progress;
    }
  }
}
```

**Kill unused triggers:**
```javascript
// When content is removed
ScrollTrigger.getAll().forEach(trigger => {
  if (!document.body.contains(trigger.trigger)) {
    trigger.kill();
  }
});
```
</step>

<step name="optimize-event-handlers">
## Step 7: Optimize Event Handlers

**Use gsap.quickTo for frequent updates:**
```javascript
// BAD - creates new tween on every mousemove
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

**Throttle with requestAnimationFrame:**
```javascript
let rafId;

document.addEventListener("mousemove", (e) => {
  if (rafId) return;

  rafId = requestAnimationFrame(() => {
    gsap.set(cursor, { x: e.clientX, y: e.clientY });
    rafId = null;
  });
});
```
</step>

<step name="verify-improvements">
## Step 8: Verify Improvements

**Re-measure performance:**
1. Record new Performance trace
2. Compare frame rates
3. Check for reduced layout/paint events
4. Verify smooth 60fps

**Target metrics:**
- Frame time: <16ms consistently
- No long tasks during animation
- No layout thrashing (purple events)
- Minimal paint events (green events)

**Test on lower-end devices:**
```javascript
// Simulate slower device in DevTools
// Performance → CPU throttling → 4x slowdown
```
</step>

</process>

<optimization_checklist>
- [ ] Animating only transform and opacity?
- [ ] No CSS transitions on animated elements?
- [ ] Using stagger instead of individual tweens?
- [ ] ScrollTrigger using scrub smoothing?
- [ ] gsap.quickTo for frequent updates?
- [ ] will-change removed after animation?
- [ ] Reasonable number of animated elements?
- [ ] No layout properties being animated?
- [ ] Event handlers throttled?
- [ ] Unused ScrollTriggers killed?
</optimization_checklist>

<success_criteria>
Animation is optimized when:
- Consistent 60fps (16ms or less per frame)
- No visible jank or stuttering
- Works smoothly on mobile devices
- No layout thrashing in Performance trace
- GPU memory stays reasonable
- Smooth scrolling with ScrollTrigger
</success_criteria>
