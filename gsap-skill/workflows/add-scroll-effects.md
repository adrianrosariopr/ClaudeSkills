# Workflow: Add Scroll-Triggered Effects

<required_reading>
**Read these reference files NOW:**
1. references/scrolltrigger.md
2. references/performance.md
3. references/creative-effects.md
</required_reading>

<process>

<step name="register-plugin">
## Step 1: Register ScrollTrigger

```javascript
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);
```
</step>

<step name="choose-scroll-type">
## Step 2: Choose Scroll Animation Type

**Trigger-based:** Animation plays when element enters viewport
```javascript
gsap.to(".box", {
  x: 100,
  scrollTrigger: ".box" // triggers when .box enters viewport
});
```

**Scrub-based:** Animation progress tied to scroll position
```javascript
gsap.to(".box", {
  x: 100,
  scrollTrigger: {
    trigger: ".box",
    scrub: true // animation follows scroll
  }
});
```

**Pinned:** Element stays fixed while animation plays
```javascript
gsap.to(".content", {
  x: "-100%",
  scrollTrigger: {
    trigger: ".section",
    pin: true,
    scrub: 1
  }
});
```
</step>

<step name="configure-trigger">
## Step 3: Configure ScrollTrigger

**Basic trigger configuration:**
```javascript
ScrollTrigger.create({
  trigger: ".section",
  start: "top center",    // trigger top hits viewport center
  end: "bottom center",   // trigger bottom hits viewport center
  markers: true,          // DEBUG: show start/end markers
  onEnter: () => console.log("entered"),
  onLeave: () => console.log("left"),
  onEnterBack: () => console.log("entered from bottom"),
  onLeaveBack: () => console.log("left from top")
});
```

**Start/End position syntax:**
```
"top center"     - element's top at viewport's center
"top 80%"        - element's top at 80% down viewport
"top top"        - element's top at viewport's top
"center center"  - element's center at viewport's center
"bottom bottom"  - element's bottom at viewport's bottom
"+=100"          - 100px after calculated position
```

**With timeline:**
```javascript
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".section",
    start: "top center",
    end: "+=500",           // 500px of scroll distance
    scrub: 1,               // 1 second smoothing
    pin: true,
    markers: true
  }
});

tl.to(".title", { y: -50, opacity: 0 })
  .to(".image", { scale: 1.2 })
  .to(".content", { x: 0 });
```
</step>

<step name="implement-parallax">
## Step 4: Implement Parallax (if needed)

**Simple parallax:**
```javascript
gsap.to(".background", {
  y: -200,
  ease: "none",
  scrollTrigger: {
    trigger: ".section",
    start: "top bottom",
    end: "bottom top",
    scrub: true
  }
});
```

**Multi-layer parallax:**
```javascript
gsap.utils.toArray("[data-speed]").forEach(el => {
  const speed = parseFloat(el.dataset.speed);
  gsap.to(el, {
    y: (i, target) => -ScrollTrigger.maxScroll(window) * speed,
    ease: "none",
    scrollTrigger: {
      start: 0,
      end: "max",
      scrub: true
    }
  });
});
```

```html
<div data-speed="0.5">Moves slower</div>
<div data-speed="1.5">Moves faster</div>
```
</step>

<step name="handle-pinning">
## Step 5: Handle Pinning (if needed)

**Basic pin:**
```javascript
ScrollTrigger.create({
  trigger: ".panel",
  start: "top top",
  end: "+=1000",  // pin for 1000px of scroll
  pin: true
});
```

**Pin with horizontal scroll:**
```javascript
const sections = gsap.utils.toArray(".panel");

gsap.to(sections, {
  xPercent: -100 * (sections.length - 1),
  ease: "none",
  scrollTrigger: {
    trigger: ".container",
    pin: true,
    scrub: 1,
    end: () => "+=" + document.querySelector(".container").offsetWidth
  }
});
```

**CRITICAL: Don't animate the pinned element itself**
```javascript
// BAD - animating pinned element throws off measurements
gsap.to(".pinned", {
  x: 100,
  scrollTrigger: { trigger: ".pinned", pin: true }
});

// GOOD - animate children of pinned element
gsap.to(".pinned-content", {
  x: 100,
  scrollTrigger: { trigger: ".pinned", pin: true }
});
```
</step>

<step name="optimize-and-debug">
## Step 6: Optimize and Debug

**Enable markers during development:**
```javascript
scrollTrigger: {
  markers: true,
  // markers: { startColor: "green", endColor: "red" }
}
```

**Refresh on dynamic content:**
```javascript
// After content loads/changes
ScrollTrigger.refresh();

// After images load
ScrollTrigger.refresh(true); // recalculates positions
```

**Performance optimization:**
```javascript
scrollTrigger: {
  // Smooth scrubbing (1 = 1 second catch-up)
  scrub: 1,

  // Limit callback frequency
  // limitCallbacks: true,

  // Batch similar triggers
  // fastScrollEnd: true
}
```

**Remove markers for production:**
```javascript
// markers: process.env.NODE_ENV === 'development'
```
</step>

<step name="verify">
## Step 7: Verify

1. Scroll through entire page slowly
2. Scroll quickly to test scrub smoothing
3. Resize window and scroll again
4. Check mobile viewport
5. Verify markers align with expected positions
6. Remove markers before shipping

```javascript
// Debug scroll position
ScrollTrigger.create({
  onUpdate: self => {
    console.log("progress:", self.progress.toFixed(3));
  }
});
```
</step>

</process>

<anti_patterns>
**Avoid:**
- Animating the pinned element itself (animate children instead)
- Nested ScrollTriggers on timeline children (apply to parent timeline)
- Forgetting to refresh after dynamic content loads
- Using `ease` with `scrub: true` (easing has no effect when scrubbing)
- Leaving markers enabled in production
- Creating ScrollTriggers before DOM is ready
</anti_patterns>

<success_criteria>
A well-built scroll animation:
- Triggers at correct scroll positions
- Scrubs smoothly (if scrub enabled)
- Pins without layout shift (if pinned)
- Works after window resize
- Has no duplicate triggers
- Performs at 60fps during scroll
- Has markers disabled in production
</success_criteria>
