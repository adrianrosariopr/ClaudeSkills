<overview>
GSAP (GreenSock Animation Platform) is an industry-standard JavaScript animation library. Core concepts include tweens, timelines, and easing. Understanding these fundamentals is essential for creating performant, maintainable animations.
</overview>

<tweens>
A **tween** is a single animation of one or more properties over time.

<tween_types>
**gsap.to()** - Animate FROM current state TO specified values
```javascript
gsap.to(".box", {
  x: 100,          // end value
  opacity: 0.5,    // end value
  duration: 1
});
```

**gsap.from()** - Animate FROM specified values TO current state
```javascript
gsap.from(".box", {
  y: -50,          // start value
  opacity: 0,      // start value
  duration: 1
});
// Box animates from y:-50, opacity:0 to its current CSS values
```

**gsap.fromTo()** - Specify both FROM and TO values
```javascript
gsap.fromTo(".box",
  { x: 0, opacity: 0 },       // from
  { x: 100, opacity: 1 }      // to
);
```

**gsap.set()** - Instantly set values (no animation)
```javascript
gsap.set(".box", { x: 100, opacity: 0.5 });
// Immediate, no animation
```
</tween_types>

<when_to_use_each>
| Method | Use When |
|--------|----------|
| `to()` | Most cases - animate to a destination |
| `from()` | Entrance animations - animate in from off-screen/invisible |
| `fromTo()` | Need precise control of both start and end |
| `set()` | Initial state setup, instant changes |
</when_to_use_each>

<tween_properties>
```javascript
gsap.to(".box", {
  // Transform properties (GPU-accelerated)
  x: 100,           // translateX in pixels
  y: 50,            // translateY in pixels
  xPercent: 50,     // translateX as percentage of element width
  yPercent: 50,     // translateY as percentage of element height
  scale: 1.5,       // uniform scale
  scaleX: 2,        // horizontal scale
  scaleY: 0.5,      // vertical scale
  rotation: 45,     // degrees
  rotationX: 45,    // 3D rotation X
  rotationY: 45,    // 3D rotation Y

  // Opacity (GPU-accelerated)
  opacity: 0.5,
  autoAlpha: 0,     // opacity + visibility:hidden when 0

  // Timing
  duration: 1,      // seconds
  delay: 0.5,       // seconds before start

  // Easing
  ease: "power2.out",

  // Callbacks
  onStart: () => {},
  onUpdate: () => {},
  onComplete: () => {},

  // Special
  stagger: 0.1,     // delay between each target
  repeat: 2,        // repeat count (-1 = infinite)
  yoyo: true,       // reverse on repeat
  overwrite: "auto" // handle conflicting tweens
});
```
</tween_properties>
</tweens>

<timelines>
A **timeline** is a container for sequencing multiple tweens and controlling them as a unit.

<why_timelines>
**Without timeline (hard to maintain):**
```javascript
gsap.to(".a", { x: 100, duration: 0.5 });
gsap.to(".b", { y: 50, duration: 0.5, delay: 0.5 });
gsap.to(".c", { opacity: 0, duration: 0.5, delay: 1 });
// Changing timing requires updating all delays!
```

**With timeline (easy to maintain):**
```javascript
const tl = gsap.timeline();
tl.to(".a", { x: 100, duration: 0.5 })
  .to(".b", { y: 50, duration: 0.5 })
  .to(".c", { opacity: 0, duration: 0.5 });
// Timing automatically chains
```
</why_timelines>

<timeline_defaults>
```javascript
const tl = gsap.timeline({
  defaults: {
    duration: 0.6,
    ease: "power2.out"
  }
});

// All children inherit defaults
tl.to(".a", { x: 100 })     // duration: 0.6, ease: power2.out
  .to(".b", { y: 50 })      // duration: 0.6, ease: power2.out
  .to(".c", { opacity: 0, duration: 1 }); // override: duration: 1
```
</timeline_defaults>

<position_parameter>
Control when animations start relative to each other:

```javascript
const tl = gsap.timeline();

tl.to(".a", { x: 100 })              // starts at 0
  .to(".b", { y: 50 })               // starts when .a ends
  .to(".c", { opacity: 0 }, "+=0.5") // 0.5s after .b ends
  .to(".d", { scale: 1.2 }, "-=0.3") // 0.3s before .c ends
  .to(".e", { rotation: 90 }, "<")   // same START time as previous
  .to(".f", { x: 50 }, ">")          // same END time as previous
  .to(".g", { y: 100 }, 2);          // at absolute time 2s
```

| Position | Meaning |
|----------|---------|
| (none) | After previous animation |
| `"+=0.5"` | 0.5s after previous ends |
| `"-=0.3"` | 0.3s before previous ends (overlap) |
| `"<"` | Same start time as previous |
| `">"` | Same end time as previous |
| `2` | At absolute time 2 seconds |
| `"myLabel"` | At label position |
| `"myLabel+=0.5"` | 0.5s after label |
</position_parameter>

<timeline_controls>
```javascript
const tl = gsap.timeline({ paused: true });

tl.play();           // play from current position
tl.pause();          // pause at current position
tl.reverse();        // play backwards
tl.restart();        // restart from beginning
tl.progress(0.5);    // jump to 50%
tl.seek(2);          // jump to 2 seconds
tl.timeScale(2);     // 2x speed
tl.kill();           // destroy timeline
```
</timeline_controls>

<labels>
```javascript
const tl = gsap.timeline();

tl.addLabel("start")
  .to(".title", { opacity: 1 })
  .addLabel("titleDone")
  .to(".content", { y: 0 }, "titleDone+=0.2")
  .addLabel("end");

// Use labels
tl.play("titleDone");
tl.seek("start");
```
</labels>
</timelines>

<easing>
Easing controls the rate of change during an animation.

<easing_types>
**Power eases (most common):**
- `power1` (linear-ish)
- `power2` (default, smooth)
- `power3` (more dramatic)
- `power4` (very dramatic)

**Direction suffixes:**
- `.out` - Fast start, slow end (use for entrances) - **90% of UI cases**
- `.in` - Slow start, fast end (use for exits)
- `.inOut` - Slow both ends (use for loops)

**Special eases:**
- `back.out(1.7)` - Overshoot effect
- `elastic.out(1, 0.3)` - Bouncy spring
- `bounce.out` - Bouncing ball
- `expo.out` - Very fast start
- `circ.out` - Circular motion feel
- `none` / `linear` - Constant speed (use with scrub)
</easing_types>

<easing_recommendations>
| Use Case | Ease |
|----------|------|
| UI animations (90% of cases) | `power2.out` |
| More dramatic entrance | `power3.out` |
| Button press | `power2.in` |
| Playful/bouncy | `back.out(1.7)` |
| Very bouncy | `elastic.out(1, 0.3)` |
| Scroll-linked (scrub) | `none` |
| Looping animations | `power2.inOut` |
</easing_recommendations>

<custom_ease>
```javascript
import { CustomEase } from "gsap/CustomEase";
gsap.registerPlugin(CustomEase);

// Create custom ease from SVG path
CustomEase.create("myEase", "M0,0 C0.5,0 0.5,1 1,1");

gsap.to(".box", { x: 100, ease: "myEase" });
```
</custom_ease>
</easing>

<stagger>
Stagger creates delays between multiple targets.

<basic_stagger>
```javascript
// Simple stagger
gsap.to(".card", {
  y: 0,
  opacity: 1,
  stagger: 0.1  // 0.1s between each card
});
```
</basic_stagger>

<advanced_stagger>
```javascript
gsap.to(".card", {
  y: 0,
  opacity: 1,
  stagger: {
    each: 0.1,           // time between each
    // OR
    amount: 1,           // total time for all staggers

    from: "start",       // "start", "end", "center", "edges", "random"
    ease: "power2.in",   // easing of the stagger timing
    grid: [4, 5],        // for grid layouts [rows, cols]
    axis: "x"            // "x" or "y" for grid
  }
});
```
</advanced_stagger>

<stagger_from_options>
| from | Effect |
|------|--------|
| `"start"` | First to last (default) |
| `"end"` | Last to first |
| `"center"` | Center outward |
| `"edges"` | Edges inward |
| `"random"` | Random order |
| `[0, 0]` | From specific index |
</stagger_from_options>
</stagger>

<utility_functions>
```javascript
// Convert selector to array
const elements = gsap.utils.toArray(".card");

// Clamp value between min and max
const clamp = gsap.utils.clamp(0, 100);
clamp(150); // 100

// Map value from one range to another
const mapRange = gsap.utils.mapRange(0, 100, 0, 1);
mapRange(50); // 0.5

// Wrap value (for infinite loops)
const wrap = gsap.utils.wrap(0, 5);
wrap(7); // 2

// Interpolate between values
const interp = gsap.utils.interpolate(0, 100);
interp(0.5); // 50

// Pick from array
const colors = gsap.utils.wrap(["red", "blue", "green"]);
colors(0); // "red"
colors(3); // "red" (wraps)

// Random value
gsap.utils.random(0, 100);          // number between 0-100
gsap.utils.random([1, 5, 10]);      // random from array
gsap.utils.random(0, 100, 5);       // snapped to nearest 5
gsap.utils.random(0, 100, 5, true); // returns function
```
</utility_functions>
