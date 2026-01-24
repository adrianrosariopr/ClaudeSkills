# Workflow: Debug Animation

<required_reading>
**Read these reference files NOW:**
1. references/anti-patterns.md
2. references/core-concepts.md
3. references/performance.md
</required_reading>

<process>

<step name="identify-symptom">
## Step 1: Identify the Symptom

**Animation doesn't play at all:**
- Plugin not registered
- Target element doesn't exist
- Animation already completed
- Animation is paused

**Animation is janky/stuttery:**
- Animating layout properties
- Too many simultaneous animations
- CSS transitions conflicting
- Heavy JavaScript blocking main thread

**Animation looks wrong:**
- Wrong easing
- Conflicting animations on same property
- Initial state not set correctly
- Transform origin wrong

**ScrollTrigger issues:**
- Wrong start/end positions
- Pinning breaks layout
- Animations fire at wrong time
</step>

<step name="check-console">
## Step 2: Check Console for Errors

```javascript
// Common errors:

// "GSAP target not found"
// → Element doesn't exist or selector is wrong
gsap.to(".nonexistent", { x: 100 }); // Warns in console

// "Plugin not registered"
// → Forgot to register plugin
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger); // Required!

// No error but nothing happens
// → Check if animation is paused or already complete
console.log(tl.paused()); // true = it's paused
console.log(tl.progress()); // 1 = already complete
```
</step>

<step name="verify-targets">
## Step 3: Verify Targets Exist

```javascript
// Check element exists
const targets = gsap.utils.toArray(".my-selector");
console.log("Found targets:", targets.length, targets);

// If 0, your selector is wrong or element doesn't exist yet

// Common issues:
// 1. Element not in DOM yet (SSR, async loading)
// 2. Typo in class name
// 3. Element inside Shadow DOM (need different approach)
```

**DOM timing issues:**
```javascript
// BAD - runs before DOM ready
gsap.to(".box", { x: 100 });

// GOOD - wait for DOM
document.addEventListener("DOMContentLoaded", () => {
  gsap.to(".box", { x: 100 });
});

// In React - use useGSAP or useEffect
useGSAP(() => {
  gsap.to(".box", { x: 100 });
}, { scope: containerRef });
```
</step>

<step name="check-conflicts">
## Step 4: Check for Conflicts

**CSS transitions conflict:**
```css
/* BAD - conflicts with GSAP */
.box {
  transition: transform 0.3s ease;
}

/* GOOD - no CSS transitions on GSAP targets */
.box {
  /* no transition property */
}
```

**Multiple animations on same property:**
```javascript
// BAD - these conflict
gsap.to(".box", { x: 100, duration: 1 });
gsap.to(".box", { x: 200, duration: 1 }); // overwrites first

// GOOD - use timeline or overwrite mode
gsap.to(".box", { x: 100, duration: 1, overwrite: "auto" });
gsap.to(".box", { x: 200, duration: 1, overwrite: "auto" });
```

**Check active tweens:**
```javascript
// See all active tweens on an element
console.log(gsap.getTweensOf(".box"));

// Kill conflicting tweens
gsap.killTweensOf(".box");
```
</step>

<step name="debug-scrolltrigger">
## Step 5: Debug ScrollTrigger (if applicable)

**Enable markers:**
```javascript
ScrollTrigger.create({
  trigger: ".section",
  markers: true, // Shows green (start) and red (end) markers
  start: "top center",
  end: "bottom center"
});
```

**Log trigger state:**
```javascript
ScrollTrigger.create({
  trigger: ".section",
  onUpdate: self => {
    console.log(
      "progress:", self.progress.toFixed(2),
      "direction:", self.direction,
      "isActive:", self.isActive
    );
  }
});
```

**Common ScrollTrigger issues:**

```javascript
// Issue: Triggers fire immediately on page load
// Fix: Set initial state and use toggleActions
gsap.set(".box", { y: 50, opacity: 0 });
gsap.to(".box", {
  y: 0,
  opacity: 1,
  scrollTrigger: {
    trigger: ".box",
    start: "top 80%",
    toggleActions: "play none none reverse"
  }
});

// Issue: Pin breaks layout
// Fix: Ensure trigger has proper height, don't animate pinned element
ScrollTrigger.create({
  trigger: ".section", // This element gets pinned
  pin: true,
  start: "top top",
  end: "+=500"
});
// Animate .section's CHILDREN, not .section itself

// Issue: Positions wrong after content changes
// Fix: Refresh ScrollTrigger
ScrollTrigger.refresh();
```
</step>

<step name="check-performance">
## Step 6: Check Performance

**Open DevTools Performance tab:**
1. Click Record
2. Trigger the animation
3. Stop recording
4. Look for:
   - Long frames (>16ms, shown as red/yellow)
   - Layout/Reflow events (purple)
   - Paint events (green)

**Common performance issues:**

```javascript
// BAD - animates layout properties
gsap.to(".box", { left: 100, width: 200 });

// GOOD - uses transforms
gsap.to(".box", { x: 100, scaleX: 1.5 });

// BAD - animates box-shadow (triggers paint)
gsap.to(".box", { boxShadow: "0 10px 20px rgba(0,0,0,0.3)" });

// GOOD - use opacity on a separate shadow element
gsap.to(".box-shadow", { opacity: 1 });
```

**Check will-change:**
```javascript
// Add will-change for heavy animations (use sparingly)
gsap.set(".heavy-animation", { willChange: "transform" });

// Remove after animation
gsap.to(".heavy-animation", {
  x: 100,
  onComplete: () => {
    gsap.set(".heavy-animation", { willChange: "auto" });
  }
});
```
</step>

<step name="isolate-and-test">
## Step 7: Isolate and Test

**Create minimal reproduction:**
```javascript
// Comment out everything except one animation
// Does it work? Add back animations one by one

// Test with simple values
gsap.to(".box", {
  x: 100,
  duration: 1,
  onStart: () => console.log("Started"),
  onComplete: () => console.log("Complete")
});

// Test timeline progress
const tl = gsap.timeline();
tl.to(".a", { x: 100 })
  .to(".b", { y: 100 });

// Manually scrub
tl.progress(0.5); // Should be halfway
console.log(tl.progress()); // Verify
```
</step>

<step name="common-fixes">
## Step 8: Apply Common Fixes

**Animation doesn't play:**
```javascript
// Check if paused
tl.paused(false);
tl.play();

// Reset and play
tl.restart();

// Kill and recreate
gsap.killTweensOf(".box");
gsap.to(".box", { x: 100 });
```

**Animation fires too early:**
```javascript
// Wait for fonts/images
document.fonts.ready.then(() => {
  // Animations here
});

// Or use GSAP's native delay
gsap.to(".box", { x: 100, delay: 0.5 });
```

**Animation looks different in production:**
```javascript
// Check if SSR is causing issues (Next.js)
"use client"; // Add to component

// Check if tree-shaking removed plugins
// Explicitly import and register
gsap.registerPlugin(ScrollTrigger, SplitText);
```
</step>

</process>

<debug_checklist>
Quick checklist for common issues:

- [ ] Plugin registered? `gsap.registerPlugin(Plugin)`
- [ ] Target exists? `gsap.utils.toArray(".selector").length > 0`
- [ ] DOM ready? Running after DOMContentLoaded/useEffect
- [ ] CSS transitions removed from target?
- [ ] Animating transforms, not layout properties?
- [ ] No conflicting animations on same property?
- [ ] ScrollTrigger: markers show correct positions?
- [ ] In React: using useGSAP with proper cleanup?
- [ ] Console errors/warnings?
</debug_checklist>

<success_criteria>
Animation is debugged when:
- No console errors or warnings
- Animation plays at expected time
- Animation looks correct (values, easing, timing)
- Performance is smooth (60fps)
- Works consistently on reload
- Works in production build
</success_criteria>
