<overview>
GSAP works with React, Next.js, Vue, and other frameworks but requires proper handling of component lifecycles, state management, and cleanup. The `@gsap/react` package provides the `useGSAP` hook for seamless React integration.
</overview>

<react_setup>
```bash
npm install gsap @gsap/react
```

**Create a GSAP config file:**
```javascript
// lib/gsap.js
"use client"; // Next.js App Router

import gsap from "gsap";
import { useGSAP } from "@gsap/react";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { SplitText } from "gsap/SplitText";

gsap.registerPlugin(ScrollTrigger, SplitText);

// Optional: Set defaults
gsap.defaults({
  ease: "power2.out",
  duration: 0.6
});

export { gsap, useGSAP, ScrollTrigger, SplitText };
```
</react_setup>

<usegsap_hook>
The `useGSAP` hook handles:
- Automatic cleanup on unmount
- Scoped selectors (won't affect other components)
- React Strict Mode compatibility

```javascript
import { useRef } from "react";
import { gsap, useGSAP } from "@/lib/gsap";

function AnimatedComponent() {
  const containerRef = useRef(null);

  useGSAP(() => {
    // Selectors are scoped to containerRef
    gsap.to(".box", { x: 100 });
  }, { scope: containerRef });

  return (
    <div ref={containerRef}>
      <div className="box">Animated</div>
    </div>
  );
}
```
</usegsap_hook>

<dependencies>
Re-run animations when state changes:

```javascript
function Component() {
  const containerRef = useRef(null);
  const [isOpen, setIsOpen] = useState(false);

  useGSAP(() => {
    gsap.to(".panel", {
      height: isOpen ? "auto" : 0,
      duration: 0.3
    });
  }, {
    scope: containerRef,
    dependencies: [isOpen]  // re-runs when isOpen changes
  });

  return (
    <div ref={containerRef}>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      <div className="panel">Content</div>
    </div>
  );
}
```
</dependencies>

<without_usegsap>
If you can't use the hook, use `gsap.context()`:

```javascript
import { useEffect, useRef } from "react";
import gsap from "gsap";

function Component() {
  const containerRef = useRef(null);

  useEffect(() => {
    const ctx = gsap.context(() => {
      gsap.to(".box", { x: 100 });
    }, containerRef);

    return () => ctx.revert();  // cleanup
  }, []);

  return (
    <div ref={containerRef}>
      <div className="box">Animated</div>
    </div>
  );
}
```
</without_usegsap>

<scrolltrigger_react>
```javascript
function ScrollComponent() {
  const containerRef = useRef(null);

  useGSAP(() => {
    gsap.from(".reveal-item", {
      y: 50,
      opacity: 0,
      stagger: 0.1,
      scrollTrigger: {
        trigger: ".reveal-section",
        start: "top 80%",
        toggleActions: "play none none reverse"
      }
    });

    // Cleanup ScrollTriggers on unmount
    return () => {
      ScrollTrigger.getAll().forEach(t => t.kill());
    };
  }, { scope: containerRef });

  return (
    <div ref={containerRef}>
      <section className="reveal-section">
        <div className="reveal-item">Item 1</div>
        <div className="reveal-item">Item 2</div>
      </section>
    </div>
  );
}
```
</scrolltrigger_react>

<reusable_hooks>
**Fade in on scroll:**
```javascript
// hooks/useFadeIn.js
"use client";

import { useRef } from "react";
import { gsap, useGSAP } from "@/lib/gsap";

export function useFadeIn(options = {}) {
  const ref = useRef(null);

  useGSAP(() => {
    if (!ref.current) return;

    gsap.from(ref.current, {
      y: options.y ?? 30,
      opacity: 0,
      duration: options.duration ?? 0.6,
      scrollTrigger: {
        trigger: ref.current,
        start: options.start ?? "top 80%"
      }
    });
  }, { scope: ref });

  return ref;
}

// Usage
function MyComponent() {
  const fadeRef = useFadeIn({ y: 50 });
  return <div ref={fadeRef}>Fades in</div>;
}
```

**Hover animation:**
```javascript
// hooks/useHoverAnimation.js
export function useHoverAnimation(options = {}) {
  const ref = useRef(null);
  const tlRef = useRef(null);

  useGSAP(() => {
    if (!ref.current) return;

    const tl = gsap.timeline({ paused: true });
    tl.to(ref.current, {
      scale: options.scale ?? 1.05,
      duration: options.duration ?? 0.3
    });

    tlRef.current = tl;

    const handleEnter = () => tl.play();
    const handleLeave = () => tl.reverse();

    ref.current.addEventListener("mouseenter", handleEnter);
    ref.current.addEventListener("mouseleave", handleLeave);

    return () => {
      ref.current?.removeEventListener("mouseenter", handleEnter);
      ref.current?.removeEventListener("mouseleave", handleLeave);
    };
  }, { scope: ref });

  return ref;
}
```
</reusable_hooks>

<nextjs_specifics>
**App Router (Server Components):**
```javascript
// Component must be client-side
"use client";

import { useRef } from "react";
import { gsap, useGSAP } from "@/lib/gsap";

export default function AnimatedSection() {
  // ... animation code
}
```

**Dynamic import (avoid hydration issues):**
```javascript
import dynamic from "next/dynamic";

const AnimatedComponent = dynamic(
  () => import("./AnimatedComponent"),
  { ssr: false }
);
```

**Handle route changes:**
```javascript
useGSAP(() => {
  // Create animations

  return () => {
    // Kill ScrollTriggers on route change
    ScrollTrigger.getAll().forEach(t => t.kill());
  };
}, { scope: containerRef });
```
</nextjs_specifics>

<hydration_issues>
**Problem:** Mismatch between server and client render.

**Solution 1:** Match initial CSS state
```css
.animated-element {
  opacity: 0;
  transform: translateY(30px);
}
```

```javascript
// Animation reveals from this state
gsap.to(".animated-element", { opacity: 1, y: 0 });
```

**Solution 2:** Use `gsap.set()` before animation
```javascript
useGSAP(() => {
  gsap.set(".box", { opacity: 0, y: 30 });
  gsap.to(".box", { opacity: 1, y: 0 });
}, { scope: containerRef });
```

**Solution 3:** Disable SSR for component
```javascript
const AnimatedComponent = dynamic(
  () => import("./AnimatedComponent"),
  { ssr: false }
);
```
</hydration_issues>

<vue_integration>
```vue
<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import gsap from 'gsap';

const boxRef = ref(null);
let ctx;

onMounted(() => {
  ctx = gsap.context(() => {
    gsap.to(boxRef.value, { x: 100 });
  });
});

onUnmounted(() => {
  ctx.revert();
});
</script>

<template>
  <div ref="boxRef">Animated</div>
</template>
```

**Vue composable:**
```javascript
// composables/useGsap.js
import { onMounted, onUnmounted, ref } from 'vue';
import gsap from 'gsap';

export function useGsap(animationFn) {
  const scopeRef = ref(null);
  let ctx;

  onMounted(() => {
    ctx = gsap.context(() => {
      animationFn(gsap);
    }, scopeRef.value);
  });

  onUnmounted(() => {
    ctx?.revert();
  });

  return scopeRef;
}
```
</vue_integration>

<key_rules>
1. **Always scope animations** - Use `containerRef` to prevent affecting other components
2. **Always cleanup** - Use `useGSAP` or manually revert context
3. **Don't modify React state with GSAP** - Use React's setState for state
4. **Register plugins once** - At app level, not in components
5. **Handle SSR** - Use "use client" or dynamic imports in Next.js
</key_rules>

<anti_patterns>
**DON'T:**
```javascript
// Bad: No cleanup
useEffect(() => {
  gsap.to(".box", { x: 100 });
}, []);

// Bad: Modifying state with GSAP
gsap.to(stateRef, { value: 100 });

// Bad: Unscoped selector affects other components
gsap.to(".card", { y: 0 }); // Affects ALL .card elements!

// Bad: Registering plugins in component
function Component() {
  gsap.registerPlugin(ScrollTrigger); // Don't do this
}
```

**DO:**
```javascript
// Good: Proper cleanup with useGSAP
useGSAP(() => {
  gsap.to(".box", { x: 100 });
}, { scope: containerRef });

// Good: Scoped selectors
useGSAP(() => {
  gsap.to(".card", { y: 0 }); // Only affects .card in this container
}, { scope: containerRef });

// Good: Register plugins once in lib/gsap.js
```
</anti_patterns>
