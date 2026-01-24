<overview>
ScrollTrigger is GSAP's scroll-based animation plugin. It can trigger animations at specific scroll positions, scrub animation progress to scroll position, and pin elements in place. ScrollTrigger is highly optimized and handles all the complexity of scroll-based animations.
</overview>

<setup>
```javascript
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);
```
</setup>

<basic_trigger>
**Simplest form - trigger by selector:**
```javascript
gsap.to(".box", {
  x: 100,
  scrollTrigger: ".box" // triggers when .box enters viewport
});
```

**With configuration:**
```javascript
gsap.to(".box", {
  x: 100,
  scrollTrigger: {
    trigger: ".box",
    start: "top center", // when top of .box hits center of viewport
    end: "bottom center",
    markers: true // DEBUG: shows start/end markers
  }
});
```
</basic_trigger>

<start_end_syntax>
Format: `"triggerPosition viewportPosition"`

```javascript
start: "top center"    // trigger's top at viewport's center
start: "top 80%"       // trigger's top at 80% down viewport
start: "top top"       // trigger's top at viewport's top
start: "center center" // trigger's center at viewport's center
start: "bottom bottom" // trigger's bottom at viewport's bottom
start: "top+=100 center" // 100px below trigger's top at center
```

| Trigger Position | Meaning |
|-----------------|---------|
| `top` | Top edge of trigger element |
| `center` | Center of trigger element |
| `bottom` | Bottom edge of trigger element |
| `+=100` | Offset in pixels |

| Viewport Position | Meaning |
|------------------|---------|
| `top` | Top of viewport |
| `center` | Center of viewport |
| `bottom` | Bottom of viewport |
| `80%` | 80% from top of viewport |
</start_end_syntax>

<toggle_actions>
Control what happens at each scroll event:

```javascript
scrollTrigger: {
  toggleActions: "play pause resume reset"
  //              onEnter onLeave onEnterBack onLeaveBack
}
```

| Action | Effect |
|--------|--------|
| `play` | Play animation |
| `pause` | Pause animation |
| `resume` | Resume from paused |
| `restart` | Restart animation |
| `reset` | Reset to beginning (instant) |
| `complete` | Skip to end (instant) |
| `reverse` | Play backwards |
| `none` | Do nothing |

**Common patterns:**
```javascript
// Play once, stay at end
toggleActions: "play none none none"

// Play on enter, reverse on leave
toggleActions: "play none none reverse"

// Restart every time
toggleActions: "restart none none reset"
```
</toggle_actions>

<scrubbing>
Link animation progress directly to scroll position:

```javascript
scrollTrigger: {
  scrub: true     // instant (can feel jerky)
  scrub: 1        // 1 second smooth catch-up (recommended)
  scrub: 0.5      // 0.5 second smooth catch-up
}
```

**Important:** When using `scrub`, easing has no effect (scroll position controls progress).

```javascript
gsap.to(".box", {
  x: 500,
  ease: "power2.out", // IGNORED when scrubbing
  scrollTrigger: {
    trigger: ".section",
    start: "top top",
    end: "bottom top",
    scrub: 1
  }
});
```
</scrubbing>

<pinning>
Pin an element in place while scrolling:

```javascript
ScrollTrigger.create({
  trigger: ".panel",
  start: "top top",
  end: "+=1000",  // pin for 1000px of scroll
  pin: true
});
```

**With animation:**
```javascript
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".section",
    start: "top top",
    end: "+=2000",
    pin: true,
    scrub: 1
  }
});

tl.to(".content", { x: -1000 });
```

**CRITICAL: Don't animate the pinned element itself:**
```javascript
// BAD - breaks measurements
gsap.to(".pinned", {
  x: 100,
  scrollTrigger: { trigger: ".pinned", pin: true }
});

// GOOD - animate children
gsap.to(".pinned-content", {
  x: 100,
  scrollTrigger: { trigger: ".pinned", pin: true }
});
```
</pinning>

<horizontal_scroll>
Horizontal scrolling sections:

```javascript
const sections = gsap.utils.toArray(".panel");

gsap.to(sections, {
  xPercent: -100 * (sections.length - 1),
  ease: "none",
  scrollTrigger: {
    trigger: ".container",
    pin: true,
    scrub: 1,
    snap: 1 / (sections.length - 1), // snap to each panel
    end: () => "+=" + document.querySelector(".container").offsetWidth
  }
});
```

```css
.container {
  display: flex;
  width: 400vw; /* 4 panels * 100vw each */
}
.panel {
  width: 100vw;
  height: 100vh;
}
```
</horizontal_scroll>

<callbacks>
```javascript
ScrollTrigger.create({
  trigger: ".section",
  start: "top center",
  end: "bottom center",
  onEnter: () => console.log("entered from top"),
  onLeave: () => console.log("left from bottom"),
  onEnterBack: () => console.log("entered from bottom"),
  onLeaveBack: () => console.log("left from top"),
  onUpdate: (self) => {
    console.log("progress:", self.progress.toFixed(2));
    console.log("direction:", self.direction); // 1 or -1
    console.log("velocity:", self.getVelocity());
  },
  onToggle: (self) => {
    console.log("active:", self.isActive);
  }
});
```
</callbacks>

<batching>
Efficiently handle many similar triggers:

```javascript
// Instead of creating individual triggers
ScrollTrigger.batch(".card", {
  onEnter: (batch) => {
    gsap.to(batch, {
      opacity: 1,
      y: 0,
      stagger: 0.1
    });
  },
  onLeave: (batch) => {
    gsap.to(batch, { opacity: 0, y: 50 });
  },
  start: "top 80%"
});
```
</batching>

<refresh>
Update ScrollTrigger after content changes:

```javascript
// After dynamic content loads
ScrollTrigger.refresh();

// After images load
ScrollTrigger.refresh(true); // recalculates all positions

// After route change (SPA)
ScrollTrigger.getAll().forEach(t => t.kill());
// Then recreate triggers
```
</refresh>

<snap>
Snap to positions:

```javascript
scrollTrigger: {
  snap: 0.5,            // snap to 50% increments
  snap: 1 / 4,          // snap to quarters
  snap: [0, 0.5, 1],    // snap to specific progress values
  snap: {
    snapTo: [0, 0.5, 1],
    duration: 0.5,
    ease: "power2.inOut"
  }
}
```
</snap>

<debugging>
```javascript
// Enable markers
scrollTrigger: {
  markers: true,
  // or customize
  markers: {
    startColor: "green",
    endColor: "red",
    fontSize: "12px"
  }
}

// Log all ScrollTriggers
console.log(ScrollTrigger.getAll());

// Get specific trigger
const trigger = ScrollTrigger.getById("myId");
```

**Remove markers for production:**
```javascript
markers: process.env.NODE_ENV === "development"
```
</debugging>

<common_patterns>
**Reveal on scroll:**
```javascript
gsap.utils.toArray(".reveal").forEach(el => {
  gsap.from(el, {
    y: 50,
    opacity: 0,
    duration: 0.8,
    scrollTrigger: {
      trigger: el,
      start: "top 80%",
      toggleActions: "play none none reverse"
    }
  });
});
```

**Parallax:**
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

**Progress indicator:**
```javascript
gsap.to(".progress-bar", {
  scaleX: 1,
  ease: "none",
  scrollTrigger: {
    trigger: "body",
    start: "top top",
    end: "bottom bottom",
    scrub: true
  }
});
```
</common_patterns>

<anti_patterns>
**DON'T:**
- Put ScrollTriggers on nested timeline children
- Animate the pinned element directly
- Forget to refresh after dynamic content
- Use ease with scrub (it's ignored)
- Leave markers in production
- Create triggers before DOM is ready

**DO:**
- Apply ScrollTrigger to parent timeline
- Animate children of pinned elements
- Call `ScrollTrigger.refresh()` after content changes
- Use `scrub: 1` for smooth scrolling
- Remove markers in production
- Create triggers in DOMContentLoaded/useEffect
</anti_patterns>
