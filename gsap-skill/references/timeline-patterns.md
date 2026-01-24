<overview>
Timelines are GSAP's primary tool for organizing complex animations. This reference covers advanced timeline patterns for building maintainable, reusable animation sequences.
</overview>

<basic_structure>
```javascript
const tl = gsap.timeline({
  defaults: {
    duration: 0.6,
    ease: "power2.out"
  },
  paused: true,      // start paused
  repeat: -1,        // infinite repeat
  yoyo: true,        // reverse on repeat
  repeatDelay: 1,    // delay between repeats
  onComplete: () => console.log("done")
});

tl.to(".a", { x: 100 })
  .to(".b", { y: 50 })
  .to(".c", { opacity: 0 });
```
</basic_structure>

<position_parameter>
The position parameter controls when animations start.

```javascript
const tl = gsap.timeline();

// Sequential (default)
tl.to(".a", { x: 100 })      // 0s
  .to(".b", { y: 50 })        // after .a

// Offset from previous end
  .to(".c", { opacity: 0 }, "+=0.5")  // 0.5s after .b ends
  .to(".d", { scale: 1.2 }, "-=0.3")  // 0.3s before .c ends (overlap)

// Relative to previous start
  .to(".e", { rotation: 90 }, "<")    // same start as .d
  .to(".f", { x: 50 }, "<0.1")        // 0.1s after .d starts

// Relative to previous end
  .to(".g", { y: 100 }, ">")          // same end as .f
  .to(".h", { x: -50 }, ">-0.2")      // 0.2s before .f ends

// Absolute position
  .to(".i", { opacity: 1 }, 2);       // at exactly 2 seconds
```

**Position cheat sheet:**
| Position | Description |
|----------|-------------|
| (none) | After previous |
| `"+=0.5"` | 0.5s after previous ends |
| `"-=0.3"` | 0.3s before previous ends |
| `"<"` | Same start as previous |
| `"<0.2"` | 0.2s after previous starts |
| `">"` | Same end as previous |
| `">-0.1"` | 0.1s before previous ends |
| `2` | At absolute time 2s |
</position_parameter>

<labels>
Labels mark positions in the timeline for reference.

```javascript
const tl = gsap.timeline();

tl.addLabel("intro")
  .to(".title", { opacity: 1 })
  .to(".subtitle", { opacity: 1 })
  .addLabel("content")
  .to(".cards", { y: 0, stagger: 0.1 })
  .addLabel("cta")
  .to(".button", { scale: 1 });

// Reference labels
tl.to(".background", { opacity: 0.5 }, "intro");
tl.to(".highlight", { x: 100 }, "content+=0.2");

// Jump to label
tl.play("content");
tl.seek("cta");
```
</labels>

<nesting>
Nest timelines for modular, reusable animations.

```javascript
// Create sub-timelines
function createHeaderAnimation() {
  const tl = gsap.timeline();
  tl.from(".logo", { x: -50, opacity: 0 })
    .from(".nav-item", { y: -20, opacity: 0, stagger: 0.1 });
  return tl;
}

function createHeroAnimation() {
  const tl = gsap.timeline();
  tl.from(".hero-title", { y: 50, opacity: 0 })
    .from(".hero-subtitle", { y: 30, opacity: 0 }, "-=0.3")
    .from(".hero-cta", { scale: 0.8, opacity: 0 }, "-=0.2");
  return tl;
}

// Compose into master timeline
const master = gsap.timeline();
master
  .add(createHeaderAnimation())
  .add(createHeroAnimation(), "-=0.5")  // overlap by 0.5s
  .addLabel("loaded");
```
</nesting>

<controls>
```javascript
const tl = gsap.timeline({ paused: true });

// Playback
tl.play();              // play from current
tl.play("label");       // play from label
tl.pause();             // pause
tl.resume();            // resume from pause
tl.reverse();           // play backwards
tl.restart();           // restart from beginning

// Seek
tl.progress(0.5);       // jump to 50%
tl.seek(2);             // jump to 2 seconds
tl.seek("label");       // jump to label

// Speed
tl.timeScale(2);        // 2x speed
tl.timeScale(0.5);      // half speed

// State
tl.paused();            // returns true/false
tl.reversed();          // returns true/false
tl.progress();          // returns 0-1
tl.time();              // returns current time in seconds
tl.duration();          // returns total duration

// Destroy
tl.kill();              // kill timeline and all children
```
</controls>

<callbacks>
```javascript
const tl = gsap.timeline({
  onStart: () => console.log("Timeline started"),
  onUpdate: () => console.log("Progress:", tl.progress()),
  onComplete: () => console.log("Timeline complete"),
  onRepeat: () => console.log("Repeating"),
  onReverseComplete: () => console.log("Reverse complete")
});

// Per-tween callbacks
tl.to(".box", {
  x: 100,
  onStart: () => console.log("Tween started"),
  onComplete: () => console.log("Tween complete")
});
```
</callbacks>

<patterns>
**Intro/Outro pattern:**
```javascript
const tl = gsap.timeline({ paused: true });

// Intro
tl.addLabel("intro")
  .from(".modal", { opacity: 0, scale: 0.9 })
  .from(".modal-content", { y: 20, opacity: 0 }, "-=0.2");

// Outro (reverse order)
tl.addLabel("outro")
  .to(".modal-content", { y: 20, opacity: 0 })
  .to(".modal", { opacity: 0, scale: 0.9 }, "-=0.2");

// Usage
function openModal() {
  tl.tweenTo("outro"); // play intro only
}

function closeModal() {
  tl.play("outro"); // play from outro
}
```

**Toggle pattern:**
```javascript
const tl = gsap.timeline({ paused: true });

tl.to(".panel", { height: "auto", duration: 0.3 })
  .to(".content", { opacity: 1, duration: 0.2 }, "-=0.1");

let isOpen = false;

function toggle() {
  if (isOpen) {
    tl.reverse();
  } else {
    tl.play();
  }
  isOpen = !isOpen;
}

// Or simpler:
function toggle() {
  tl.reversed(!tl.reversed());
}
```

**Sequence with parallel elements:**
```javascript
const tl = gsap.timeline();

// Step 1: Multiple things happen together
tl.to(".bg", { opacity: 1 }, "step1")
  .to(".title", { y: 0 }, "step1")
  .to(".subtitle", { y: 0 }, "step1+=0.1")

// Step 2: Next group
  .to(".cards", { y: 0, stagger: 0.1 }, "step2")
  .to(".sidebar", { x: 0 }, "step2");
```

**Responsive timeline:**
```javascript
function createAnimation() {
  const tl = gsap.timeline();
  const isMobile = window.innerWidth < 768;

  tl.to(".box", {
    x: isMobile ? 50 : 200,
    duration: isMobile ? 0.3 : 0.6
  });

  return tl;
}

// Recreate on resize
let currentTl = createAnimation();

window.addEventListener("resize", () => {
  currentTl.kill();
  currentTl = createAnimation();
});
```
</patterns>

<defaults_inheritance>
```javascript
const tl = gsap.timeline({
  defaults: {
    duration: 0.5,
    ease: "power2.out"
  }
});

// All children inherit defaults
tl.to(".a", { x: 100 })              // duration: 0.5, ease: power2.out
  .to(".b", { y: 50 })               // duration: 0.5, ease: power2.out
  .to(".c", { opacity: 0, duration: 1 })  // duration: 1 (overridden)
  .to(".d", { scale: 1.2, ease: "back.out" }); // ease overridden
```
</defaults_inheritance>

<performance_tips>
- Use `paused: true` for reusable timelines
- Don't create new timelines in event handlers (reuse paused ones)
- Kill unused timelines: `tl.kill()`
- Use nested timelines for modularity
- Set defaults to reduce repetition
</performance_tips>

<anti_patterns>
**DON'T:**
```javascript
// Bad: New timeline every click
button.addEventListener("click", () => {
  gsap.timeline()
    .to(".box", { x: 100 })
    .to(".box", { y: 50 });
});

// Bad: Hardcoded delays instead of position parameter
gsap.to(".a", { x: 100, duration: 0.5 });
gsap.to(".b", { y: 50, delay: 0.5 }); // Hard to maintain
```

**DO:**
```javascript
// Good: Reuse paused timeline
const tl = gsap.timeline({ paused: true });
tl.to(".box", { x: 100 })
  .to(".box", { y: 50 });

button.addEventListener("click", () => {
  tl.restart();
});

// Good: Use position parameter
const tl = gsap.timeline();
tl.to(".a", { x: 100 })
  .to(".b", { y: 50 }); // Automatically sequences
```
</anti_patterns>
