<overview>
Common GSAP mistakes that cause bugs, performance issues, or maintenance headaches. Learn what to avoid and the correct patterns to use instead.
</overview>

<layout_properties>
**DON'T animate layout properties:**
```javascript
// BAD - triggers layout recalculation (slow)
gsap.to(".box", {
  left: 100,
  top: 50,
  width: 200,
  height: 150
});
```

**DO use transforms:**
```javascript
// GOOD - GPU accelerated
gsap.to(".box", {
  x: 100,
  y: 50,
  scaleX: 2,
  scaleY: 1.5
});
```

**Why:** Layout properties force the browser to recalculate positions of all affected elements. Transforms are handled by the GPU and don't trigger layout.
</layout_properties>

<css_transitions>
**DON'T mix CSS transitions with GSAP:**
```css
/* BAD */
.animated {
  transition: transform 0.3s ease, opacity 0.3s ease;
}
```

```javascript
gsap.to(".animated", { x: 100, opacity: 0.5 }); // Conflicts!
```

**DO let GSAP handle all animation:**
```css
/* GOOD */
.animated {
  /* No transition property */
}
```

**Why:** CSS transitions and GSAP fight for control, causing erratic behavior and poor performance.
</css_transitions>

<from_animations>
**DON'T use `.from()` for repeatedly triggered animations:**
```javascript
// BAD - clicking rapidly breaks it
button.addEventListener("click", () => {
  gsap.from(".box", { opacity: 0, y: 50 });
});
// If clicked before animation completes, end value becomes
// the current (mid-animation) value
```

**DO use `.fromTo()` or reset first:**
```javascript
// GOOD - explicit start and end
button.addEventListener("click", () => {
  gsap.fromTo(".box",
    { opacity: 0, y: 50 },
    { opacity: 1, y: 0 }
  );
});

// OR reset first
button.addEventListener("click", () => {
  gsap.set(".box", { opacity: 0, y: 50 });
  gsap.to(".box", { opacity: 1, y: 0 });
});
```

**Why:** `.from()` animates FROM specified values TO current values. If current values change mid-animation, results are unpredictable.
</from_animations>

<completed_timelines>
**DON'T add tweens to completed timelines:**
```javascript
// BAD
const tl = gsap.timeline();
tl.to(".a", { x: 100 });
// ...later...
tl.to(".b", { y: 50 }); // Won't play if tl already completed!
```

**DO create new timelines or use control methods:**
```javascript
// GOOD - new timeline
gsap.to(".b", { y: 50 });

// OR restart the timeline
tl.add(gsap.to(".b", { y: 50 }));
tl.restart();
```

**Why:** Once a timeline completes, adding new tweens doesn't automatically replay it.
</completed_timelines>

<broad_selectors>
**DON'T use broad selectors for single elements:**
```javascript
// BAD - affects ALL .button elements
document.querySelector(".button").addEventListener("click", () => {
  gsap.to(".button", { scale: 0.9 }); // Wrong!
});
```

**DO target the specific element:**
```javascript
// GOOD - use event target or loop
document.querySelectorAll(".button").forEach(btn => {
  btn.addEventListener("click", () => {
    gsap.to(btn, { scale: 0.9 }); // Correct!
  });
});
```

**Why:** Broad selectors affect all matching elements, not just the one being interacted with.
</broad_selectors>

<scrolltrigger_nested>
**DON'T put ScrollTriggers on nested timeline tweens:**
```javascript
// BAD
const tl = gsap.timeline();
tl.to(".a", {
    x: 100,
    scrollTrigger: { trigger: ".a" } // Don't do this!
  })
  .to(".b", {
    y: 50,
    scrollTrigger: { trigger: ".b" } // Don't do this!
  });
```

**DO apply ScrollTrigger to parent timeline:**
```javascript
// GOOD
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".section",
    start: "top center",
    end: "bottom center",
    scrub: 1
  }
});

tl.to(".a", { x: 100 })
  .to(".b", { y: 50 });
```

**Why:** Nested ScrollTriggers cause unpredictable behavior. One ScrollTrigger per animation group.
</scrolltrigger_nested>

<pin_animation>
**DON'T animate the pinned element itself:**
```javascript
// BAD - breaks pin measurements
gsap.to(".section", {
  x: 100,
  scrollTrigger: {
    trigger: ".section",
    pin: true
  }
});
```

**DO animate children of the pinned element:**
```javascript
// GOOD
gsap.to(".section-content", {
  x: 100,
  scrollTrigger: {
    trigger: ".section",
    pin: true
  }
});
```

**Why:** ScrollTrigger pre-calculates positions. Animating the pinned element throws off those calculations.
</pin_animation>

<ease_with_scrub>
**DON'T expect easing with scrub:**
```javascript
// BAD - ease is ignored
gsap.to(".box", {
  x: 100,
  ease: "power2.out", // Ignored!
  scrollTrigger: {
    scrub: true
  }
});
```

**DO understand scrub overrides easing:**
```javascript
// GOOD - use scrub smoothing instead
gsap.to(".box", {
  x: 100,
  scrollTrigger: {
    scrub: 1 // 1 second smoothing
  }
});
```

**Why:** Scrub directly ties animation progress to scroll position. Easing has no effect.
</ease_with_scrub>

<new_timelines_events>
**DON'T create new timelines in frequent events:**
```javascript
// BAD - creates new timeline every event
document.addEventListener("mousemove", (e) => {
  gsap.timeline()
    .to(cursor, { x: e.clientX })
    .to(cursor, { y: e.clientY });
});
```

**DO use quickTo or reuse timelines:**
```javascript
// GOOD - quickTo
const xTo = gsap.quickTo(cursor, "x", { duration: 0.3 });
const yTo = gsap.quickTo(cursor, "y", { duration: 0.3 });

document.addEventListener("mousemove", (e) => {
  xTo(e.clientX);
  yTo(e.clientY);
});

// OR reuse paused timeline
const tl = gsap.timeline({ paused: true });
// ... setup ...
button.addEventListener("click", () => tl.restart());
```

**Why:** Creating objects repeatedly causes memory churn and GC pauses.
</new_timelines_events>

<delays_vs_timelines>
**DON'T use manual delays for sequences:**
```javascript
// BAD - hard to maintain
gsap.to(".a", { x: 100, duration: 0.5 });
gsap.to(".b", { y: 50, delay: 0.5 });
gsap.to(".c", { opacity: 0, delay: 1 });
// Changing .a's duration requires updating all delays!
```

**DO use timelines:**
```javascript
// GOOD - automatically sequences
const tl = gsap.timeline();
tl.to(".a", { x: 100, duration: 0.5 })
  .to(".b", { y: 50 })
  .to(".c", { opacity: 0 });
```

**Why:** Manual delays create brittle code. Changing one animation requires updating all subsequent delays.
</delays_vs_timelines>

<fouc>
**DON'T forget initial state (Flash of Unstyled Content):**
```javascript
// BAD - element visible before animation starts
gsap.from(".hero", {
  opacity: 0,
  y: 50,
  delay: 0.5
});
// User sees hero at full opacity, then it resets and animates
```

**DO set initial state in CSS or with gsap.set:**
```css
/* CSS solution */
.hero {
  opacity: 0;
  transform: translateY(50px);
}
```

```javascript
// OR gsap.set solution
gsap.set(".hero", { opacity: 0, y: 50 });
gsap.to(".hero", {
  opacity: 1,
  y: 0,
  delay: 0.5
});
```

**Why:** Without initial state, elements flash their final state before the animation runs.
</fouc>

<framework_cleanup>
**DON'T forget cleanup in React/Vue:**
```javascript
// BAD - memory leak
useEffect(() => {
  gsap.to(".box", { x: 100 });
  ScrollTrigger.create({ trigger: ".section" });
}, []);
// No cleanup on unmount!
```

**DO use proper cleanup:**
```javascript
// GOOD - with useGSAP
useGSAP(() => {
  gsap.to(".box", { x: 100 });
}, { scope: containerRef }); // Auto cleanup

// OR manual cleanup
useEffect(() => {
  const ctx = gsap.context(() => {
    gsap.to(".box", { x: 100 });
  }, containerRef);

  return () => ctx.revert(); // Cleanup
}, []);
```

**Why:** Animations and ScrollTriggers persist after component unmount, causing memory leaks and errors.
</framework_cleanup>

<overwriting>
**DON'T forget about conflicting animations:**
```javascript
// BAD - unpredictable result
gsap.to(".box", { x: 100, duration: 2 });
gsap.to(".box", { x: 200, duration: 1 }); // Conflicts!
```

**DO use overwrite or kill previous:**
```javascript
// GOOD - overwrite mode
gsap.to(".box", { x: 100, overwrite: "auto" });
gsap.to(".box", { x: 200, overwrite: "auto" });

// OR kill previous
gsap.killTweensOf(".box");
gsap.to(".box", { x: 200 });
```

**Why:** Multiple animations on the same property fight for control unless properly managed.
</overwriting>

<will_change_abuse>
**DON'T overuse will-change:**
```javascript
// BAD - too many elements
gsap.utils.toArray(".card").forEach(card => {
  gsap.set(card, { willChange: "transform" });
});
// 50 cards = 50 GPU layers = performance nightmare
```

**DO use sparingly and remove after:**
```javascript
// GOOD - temporary, specific use
gsap.set(".heavy-animation", { willChange: "transform" });
gsap.to(".heavy-animation", {
  x: 500,
  onComplete: () => {
    gsap.set(".heavy-animation", { willChange: "auto" });
  }
});
```

**Why:** Too many elements with will-change consumes GPU memory and hurts performance.
</will_change_abuse>

<checklist>
**Before shipping, verify:**
- [ ] No CSS transitions on animated elements
- [ ] No layout properties animated (left, top, width, height)
- [ ] Timelines reused, not created in events
- [ ] ScrollTrigger markers removed
- [ ] Proper cleanup in React/Vue
- [ ] Initial states set (no FOUC)
- [ ] No ScrollTriggers on nested tweens
- [ ] will-change removed after animation
- [ ] Conflicting animations handled
</checklist>
