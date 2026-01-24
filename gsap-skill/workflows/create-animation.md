# Workflow: Create New Animation

<required_reading>
**Read these reference files NOW:**
1. references/core-concepts.md
2. references/timeline-patterns.md
3. references/creative-effects.md
</required_reading>

<process>

<step name="identify-elements">
## Step 1: Identify Elements to Animate

Determine what elements need animation and assign appropriate selectors.

```javascript
// Use specific class names for animation targets
// Convention: prefix with "anim-" for clarity
<div class="anim-hero-title">...</div>
<div class="anim-hero-image">...</div>
<div class="anim-cta-button">...</div>

// Verify elements exist
const elements = gsap.utils.toArray(".anim-hero-title");
console.log("Found elements:", elements.length);
```
</step>

<step name="choose-animation-type">
## Step 2: Choose Animation Type

**Single element, simple animation → Tween**
```javascript
gsap.to(".box", { x: 100, duration: 1 });
```

**Multiple elements, sequenced → Timeline**
```javascript
const tl = gsap.timeline();
tl.to(".a", { x: 100 })
  .to(".b", { y: 50 })
  .to(".c", { opacity: 0 });
```

**Multiple elements, staggered → Stagger**
```javascript
gsap.to(".cards", {
  y: 0,
  opacity: 1,
  stagger: 0.1,
  duration: 0.6
});
```
</step>

<step name="set-initial-state">
## Step 3: Set Initial State

Use `gsap.set()` to establish starting positions. This prevents flash of unstyled content (FOUC).

```javascript
// Set initial state immediately (no animation)
gsap.set(".anim-hero-title", {
  y: 50,
  opacity: 0
});

// Also add CSS fallback for no-JS
// .anim-hero-title { opacity: 0; }
```
</step>

<step name="build-animation">
## Step 4: Build the Animation

**Basic Tween:**
```javascript
gsap.to(".anim-hero-title", {
  y: 0,
  opacity: 1,
  duration: 0.8,
  ease: "power2.out"
});
```

**Timeline with defaults:**
```javascript
const tl = gsap.timeline({
  defaults: {
    duration: 0.6,
    ease: "power2.out"
  }
});

tl.to(".anim-hero-title", { y: 0, opacity: 1 })
  .to(".anim-hero-subtitle", { y: 0, opacity: 1 }, "-=0.3") // overlap
  .to(".anim-cta-button", { scale: 1, opacity: 1 }, "-=0.2");
```

**Staggered animation:**
```javascript
gsap.to(".anim-cards", {
  y: 0,
  opacity: 1,
  duration: 0.6,
  ease: "power2.out",
  stagger: {
    each: 0.1,        // 0.1s between each
    from: "start",    // or "end", "center", "edges", "random"
    ease: "power1.in" // easing of the stagger timing
  }
});
```
</step>

<step name="add-polish">
## Step 5: Add Polish

**Ease selection:**
- `power2.out` - Standard UI (90% of cases)
- `power3.out` - More dramatic entrance
- `back.out(1.7)` - Overshoot effect
- `elastic.out(1, 0.3)` - Bouncy effect
- `expo.out` - Very fast start, slow end

**Position parameter for timing:**
```javascript
tl.to(".a", { x: 100 })           // starts at 0
  .to(".b", { y: 50 }, "+=0.5")   // 0.5s after previous ends
  .to(".c", { opacity: 0 }, "-=0.3") // 0.3s before previous ends
  .to(".d", { scale: 1.2 }, "<")  // same start as previous
  .to(".e", { rotation: 90 }, ">"); // same end as previous
```

**Labels for complex timelines:**
```javascript
tl.addLabel("intro")
  .to(".title", { opacity: 1 })
  .to(".subtitle", { opacity: 1 })
  .addLabel("content")
  .to(".cards", { y: 0, stagger: 0.1 });

// Jump to label
tl.play("content");
```
</step>

<step name="verify">
## Step 6: Verify Animation

```javascript
// 1. Check it plays
tl.play();

// 2. Check it reverses
tl.reverse();

// 3. Check specific progress
tl.progress(0.5);

// 4. Log duration
console.log("Total duration:", tl.duration());

// 5. Add visual debugging
tl.eventCallback("onUpdate", () => {
  console.log("Progress:", tl.progress().toFixed(2));
});
```

Open DevTools > Performance tab, record while animation plays. Check for:
- Frames >16ms (causes jank)
- Layout thrashing (purple bars)
- Excessive paint (green bars)
</step>

</process>

<anti_patterns>
**Avoid:**
- Animating `left`, `top`, `width`, `height` (use `x`, `y`, `scale`)
- Using delays instead of timelines
- Forgetting to set initial state (causes FOUC)
- Animating too many properties at once
- Using `!important` in CSS on animated properties
- CSS transitions on GSAP-animated elements
</anti_patterns>

<success_criteria>
A well-built GSAP animation:
- Uses transforms and opacity only
- Has explicit initial state set with `gsap.set()`
- Uses timeline for sequences
- Has appropriate easing (usually `power2.out`)
- Runs at 60fps (check Performance tab)
- Can be reversed and scrubbed
- Has no console errors
</success_criteria>
