<overview>
Creative GSAP effects transform static designs into engaging experiences. This reference covers parallax, stagger patterns, interactive effects, and advanced techniques for building memorable animations.
</overview>

<parallax>
**Basic scroll parallax:**
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
// HTML: <div data-speed="0.5">...</div>
gsap.utils.toArray("[data-speed]").forEach(el => {
  const speed = parseFloat(el.dataset.speed);

  gsap.to(el, {
    y: () => -ScrollTrigger.maxScroll(window) * speed,
    ease: "none",
    scrollTrigger: {
      start: 0,
      end: "max",
      scrub: true,
      invalidateOnRefresh: true
    }
  });
});
```

**Horizontal parallax:**
```javascript
gsap.to(".parallax-element", {
  x: -100,
  ease: "none",
  scrollTrigger: {
    trigger: ".container",
    start: "left right",
    end: "right left",
    horizontal: true,
    scrub: true
  }
});
```

**Mouse parallax:**
```javascript
const layers = gsap.utils.toArray("[data-mouse-parallax]");

document.addEventListener("mousemove", (e) => {
  const xPos = (e.clientX / window.innerWidth - 0.5) * 2;
  const yPos = (e.clientY / window.innerHeight - 0.5) * 2;

  layers.forEach(layer => {
    const depth = parseFloat(layer.dataset.mouseParallax);
    gsap.to(layer, {
      x: xPos * depth * 50,
      y: yPos * depth * 50,
      duration: 0.5,
      ease: "power2.out"
    });
  });
});
```
</parallax>

<stagger_effects>
**Cascade reveal:**
```javascript
gsap.from(".card", {
  y: 100,
  opacity: 0,
  duration: 0.8,
  stagger: {
    each: 0.1,
    from: "start"
  }
});
```

**Center outward:**
```javascript
gsap.from(".item", {
  scale: 0,
  opacity: 0,
  duration: 0.5,
  stagger: {
    each: 0.05,
    from: "center"
  }
});
```

**Grid wave:**
```javascript
gsap.from(".grid-item", {
  y: 50,
  opacity: 0,
  duration: 0.6,
  stagger: {
    grid: [4, 5],    // 4 rows, 5 columns
    from: "start",
    axis: "y",       // wave direction
    each: 0.08
  }
});
```

**Random stagger:**
```javascript
gsap.from(".particle", {
  scale: 0,
  rotation: gsap.utils.random(-180, 180),
  duration: 0.5,
  stagger: {
    each: 0.02,
    from: "random"
  }
});
```

**Zipper effect:**
```javascript
const wrap = gsap.utils.wrap([-50, 50]); // alternating values

gsap.from(".item", {
  x: index => wrap(index),
  opacity: 0,
  duration: 0.6,
  stagger: 0.1
});
```
</stagger_effects>

<hover_effects>
**Scale with ease:**
```javascript
const cards = gsap.utils.toArray(".card");

cards.forEach(card => {
  const tl = gsap.timeline({ paused: true });

  tl.to(card, {
    scale: 1.05,
    boxShadow: "0 20px 40px rgba(0,0,0,0.2)",
    duration: 0.3,
    ease: "power2.out"
  });

  card.addEventListener("mouseenter", () => tl.play());
  card.addEventListener("mouseleave", () => tl.reverse());
});
```

**Magnetic button:**
```javascript
const button = document.querySelector(".magnetic-btn");
const strength = 0.3;

button.addEventListener("mousemove", (e) => {
  const rect = button.getBoundingClientRect();
  const x = e.clientX - rect.left - rect.width / 2;
  const y = e.clientY - rect.top - rect.height / 2;

  gsap.to(button, {
    x: x * strength,
    y: y * strength,
    duration: 0.3
  });
});

button.addEventListener("mouseleave", () => {
  gsap.to(button, {
    x: 0,
    y: 0,
    duration: 0.5,
    ease: "elastic.out(1, 0.3)"
  });
});
```

**Reveal on hover:**
```javascript
const items = gsap.utils.toArray(".menu-item");

items.forEach(item => {
  const line = item.querySelector(".underline");

  gsap.set(line, { scaleX: 0, transformOrigin: "left" });

  item.addEventListener("mouseenter", () => {
    gsap.to(line, { scaleX: 1, duration: 0.3 });
  });

  item.addEventListener("mouseleave", () => {
    gsap.to(line, {
      scaleX: 0,
      transformOrigin: "right",
      duration: 0.3
    });
  });
});
```
</hover_effects>

<scroll_effects>
**Pin and animate:**
```javascript
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".hero",
    start: "top top",
    end: "+=150%",
    pin: true,
    scrub: 1
  }
});

tl.to(".hero-title", { y: -100, opacity: 0 })
  .to(".hero-image", { scale: 1.2 }, "<")
  .to(".hero-cta", { y: 50, opacity: 0 }, "<0.2");
```

**Horizontal scroll section:**
```javascript
const sections = gsap.utils.toArray(".panel");

gsap.to(sections, {
  xPercent: -100 * (sections.length - 1),
  ease: "none",
  scrollTrigger: {
    trigger: ".horizontal-container",
    pin: true,
    scrub: 1,
    snap: 1 / (sections.length - 1),
    end: () => "+=" + document.querySelector(".horizontal-container").offsetWidth
  }
});
```

**Progress indicator:**
```javascript
gsap.to(".progress-bar", {
  scaleX: 1,
  transformOrigin: "left",
  ease: "none",
  scrollTrigger: {
    trigger: "body",
    start: "top top",
    end: "bottom bottom",
    scrub: true
  }
});
```

**Scroll velocity effects:**
```javascript
ScrollTrigger.create({
  onUpdate: (self) => {
    const velocity = self.getVelocity();

    // Skew based on scroll speed
    gsap.to(".content", {
      skewY: velocity * 0.001,
      duration: 0.5
    });
  }
});
```
</scroll_effects>

<text_effects>
**Glitch text:**
```javascript
const glitchText = (element) => {
  const tl = gsap.timeline({ repeat: -1, repeatDelay: 3 });

  tl.to(element, {
    skewX: 20,
    duration: 0.1,
    ease: "power1.inOut"
  })
  .to(element, { skewX: 0, duration: 0.04 })
  .to(element, { opacity: 0.8, duration: 0.04 })
  .to(element, { opacity: 1, duration: 0.04 })
  .to(element, { x: -5, duration: 0.04 })
  .to(element, { x: 0, duration: 0.04 });

  return tl;
};
```

**Typewriter:**
```javascript
gsap.to(".typewriter", {
  text: "Hello, I'm typing...",
  duration: 2,
  ease: "none",
  delay: 1
});
```

**Letter shuffle:**
```javascript
const split = new SplitText(".title", { type: "chars" });

gsap.from(split.chars, {
  opacity: 0,
  y: gsap.utils.wrap([-100, 100]),
  rotation: gsap.utils.random(-90, 90),
  duration: 0.8,
  stagger: {
    each: 0.03,
    from: "random"
  }
});
```
</text_effects>

<loading_animations>
**Spinner:**
```javascript
gsap.to(".spinner", {
  rotation: 360,
  duration: 1,
  ease: "none",
  repeat: -1
});
```

**Progress loader:**
```javascript
const loadProgress = gsap.timeline({ paused: true });

loadProgress
  .to(".loader-bar", { scaleX: 1, duration: 2 })
  .to(".loader-text", {
    textContent: 100,
    duration: 2,
    snap: { textContent: 1 },
    ease: "none"
  }, "<");

// Update with actual progress
loadProgress.progress(0.5); // 50% loaded
```

**Skeleton loading:**
```javascript
gsap.to(".skeleton", {
  backgroundPosition: "200% 0",
  duration: 1.5,
  ease: "none",
  repeat: -1
});
```
</loading_animations>

<micro_interactions>
**Button press:**
```javascript
const button = document.querySelector(".btn");

button.addEventListener("mousedown", () => {
  gsap.to(button, { scale: 0.95, duration: 0.1 });
});

button.addEventListener("mouseup", () => {
  gsap.to(button, { scale: 1, duration: 0.2, ease: "back.out(2)" });
});
```

**Ripple effect:**
```javascript
document.querySelectorAll(".ripple-btn").forEach(btn => {
  btn.addEventListener("click", (e) => {
    const ripple = document.createElement("span");
    ripple.classList.add("ripple");
    btn.appendChild(ripple);

    const rect = btn.getBoundingClientRect();
    gsap.set(ripple, {
      x: e.clientX - rect.left,
      y: e.clientY - rect.top
    });

    gsap.to(ripple, {
      scale: 50,
      opacity: 0,
      duration: 0.6,
      onComplete: () => ripple.remove()
    });
  });
});
```

**Toggle switch:**
```javascript
const toggle = document.querySelector(".toggle");
const knob = toggle.querySelector(".knob");
let isOn = false;

toggle.addEventListener("click", () => {
  isOn = !isOn;
  gsap.to(knob, {
    x: isOn ? 24 : 0,
    backgroundColor: isOn ? "#10b981" : "#6b7280",
    duration: 0.2
  });
});
```
</micro_interactions>

<performance_tips>
- Use `gsap.quickTo()` for frequent updates (mouse follow)
- Reuse paused timelines for hover effects
- Use transforms only (no layout properties)
- Batch similar ScrollTriggers
- Kill unused animations
</performance_tips>
