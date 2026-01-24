# Workflow: Add Interactivity

<required_reading>
**Read these reference files NOW:**
1. references/creative-effects.md
2. references/core-concepts.md
3. references/performance.md
</required_reading>

<process>

<step name="choose-interaction-type">
## Step 1: Choose Interaction Type

**Hover effects** - Mouse enter/leave
**Click effects** - Toggle or one-shot
**Drag effects** - Draggable elements
**Mouse follow** - Elements follow cursor
**Scroll velocity** - React to scroll speed
</step>

<step name="implement-hover">
## Step 2: Implement Hover Effects

**Basic hover (play/reverse):**
```javascript
const boxes = gsap.utils.toArray(".box");

boxes.forEach(box => {
  const tl = gsap.timeline({ paused: true });

  tl.to(box, {
    scale: 1.1,
    duration: 0.3,
    ease: "power2.out"
  });

  box.addEventListener("mouseenter", () => tl.play());
  box.addEventListener("mouseleave", () => tl.reverse());
});
```

**Magnetic button effect:**
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
    duration: 0.3,
    ease: "power2.out"
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

**Staggered hover on parent:**
```javascript
const container = document.querySelector(".card-container");
const cards = gsap.utils.toArray(".card");

container.addEventListener("mouseenter", () => {
  gsap.to(cards, {
    y: -10,
    stagger: 0.05,
    duration: 0.3
  });
});

container.addEventListener("mouseleave", () => {
  gsap.to(cards, {
    y: 0,
    stagger: 0.05,
    duration: 0.3
  });
});
```
</step>

<step name="implement-click">
## Step 3: Implement Click Effects

**Toggle animation:**
```javascript
const box = document.querySelector(".box");
let isOpen = false;

const tl = gsap.timeline({ paused: true });
tl.to(box, { height: 300, duration: 0.5 })
  .to(".box-content", { opacity: 1, duration: 0.3 }, "-=0.2");

box.addEventListener("click", () => {
  if (isOpen) {
    tl.reverse();
  } else {
    tl.play();
  }
  isOpen = !isOpen;
});
```

**One-shot animation (ripple effect):**
```javascript
document.querySelectorAll(".btn").forEach(btn => {
  btn.addEventListener("click", (e) => {
    const ripple = document.createElement("span");
    ripple.classList.add("ripple");
    btn.appendChild(ripple);

    const rect = btn.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;

    gsap.set(ripple, { x, y });
    gsap.to(ripple, {
      scale: 50,
      opacity: 0,
      duration: 0.6,
      ease: "power2.out",
      onComplete: () => ripple.remove()
    });
  });
});
```
</step>

<step name="implement-drag">
## Step 4: Implement Drag Effects

**Basic draggable:**
```javascript
import { Draggable } from "gsap/Draggable";
gsap.registerPlugin(Draggable);

Draggable.create(".box", {
  type: "x,y",           // or "x", "y", "rotation"
  bounds: ".container",  // constrain to container
  inertia: true,         // throw momentum
  onDrag: function() {
    console.log("x:", this.x, "y:", this.y);
  }
});
```

**Draggable slider:**
```javascript
Draggable.create(".slider-handle", {
  type: "x",
  bounds: { minX: 0, maxX: 300 },
  onDrag: function() {
    const progress = this.x / 300;
    gsap.set(".fill", { scaleX: progress });
  }
});
```

**Sortable list:**
```javascript
const items = gsap.utils.toArray(".list-item");

items.forEach((item, index) => {
  Draggable.create(item, {
    type: "y",
    bounds: ".list-container",
    onDrag: function() {
      // Reorder logic
      items.forEach((other, i) => {
        if (other !== item) {
          const rect = other.getBoundingClientRect();
          if (this.y > rect.top && this.y < rect.bottom) {
            gsap.to(other, { y: i < index ? 50 : -50 });
          }
        }
      });
    }
  });
});
```
</step>

<step name="implement-mouse-follow">
## Step 5: Implement Mouse Follow

**Custom cursor:**
```javascript
const cursor = document.querySelector(".custom-cursor");

document.addEventListener("mousemove", (e) => {
  gsap.to(cursor, {
    x: e.clientX,
    y: e.clientY,
    duration: 0.3,
    ease: "power2.out"
  });
});

// Hover effects on interactive elements
document.querySelectorAll("a, button").forEach(el => {
  el.addEventListener("mouseenter", () => {
    gsap.to(cursor, { scale: 2, duration: 0.2 });
  });
  el.addEventListener("mouseleave", () => {
    gsap.to(cursor, { scale: 1, duration: 0.2 });
  });
});
```

**Parallax mouse follow:**
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
</step>

<step name="optimize-events">
## Step 6: Optimize Event Handlers

**Throttle mousemove:**
```javascript
let rafId;

document.addEventListener("mousemove", (e) => {
  if (rafId) return;

  rafId = requestAnimationFrame(() => {
    gsap.to(cursor, { x: e.clientX, y: e.clientY });
    rafId = null;
  });
});
```

**Use gsap.quickTo for frequent updates:**
```javascript
const xTo = gsap.quickTo(cursor, "x", { duration: 0.3 });
const yTo = gsap.quickTo(cursor, "y", { duration: 0.3 });

document.addEventListener("mousemove", (e) => {
  xTo(e.clientX);
  yTo(e.clientY);
});
```
</step>

<step name="cleanup">
## Step 7: Cleanup (especially in React)

```javascript
import { useGSAP } from "@gsap/react";
import { useRef, useEffect } from "react";

function InteractiveComponent() {
  const containerRef = useRef();

  useGSAP(() => {
    const box = containerRef.current.querySelector(".box");
    const tl = gsap.timeline({ paused: true });

    tl.to(box, { scale: 1.1, duration: 0.3 });

    const handleEnter = () => tl.play();
    const handleLeave = () => tl.reverse();

    box.addEventListener("mouseenter", handleEnter);
    box.addEventListener("mouseleave", handleLeave);

    return () => {
      box.removeEventListener("mouseenter", handleEnter);
      box.removeEventListener("mouseleave", handleLeave);
    };
  }, { scope: containerRef });

  return (
    <div ref={containerRef}>
      <div className="box">Hover me</div>
    </div>
  );
}
```
</step>

</process>

<anti_patterns>
**Avoid:**
- Creating new timelines on every event (reuse paused timelines)
- Not throttling mousemove handlers
- Forgetting to remove event listeners on cleanup
- Using CSS transitions on interactive elements (conflicts with GSAP)
- Animating layout properties on drag (use transforms)
</anti_patterns>

<success_criteria>
A well-built interactive animation:
- Reuses paused timelines (not creating new ones)
- Uses `gsap.quickTo` for frequent updates
- Throttles mousemove when needed
- Cleans up event listeners properly
- Uses transforms only (no layout thrashing)
- Feels responsive (<100ms perceived delay)
</success_criteria>
