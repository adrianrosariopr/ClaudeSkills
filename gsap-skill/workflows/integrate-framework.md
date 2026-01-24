# Workflow: Integrate with React/Next.js/Vue

<required_reading>
**Read these reference files NOW:**
1. references/framework-integration.md
2. references/core-concepts.md
3. references/anti-patterns.md
</required_reading>

<process>

<step name="install-packages">
## Step 1: Install Packages

```bash
# Core GSAP (includes all plugins since 3.13)
npm install gsap

# React integration hook
npm install @gsap/react
```

**GSAP 3.13+ Note:** All plugins (SplitText, MorphSVG, ScrollSmoother, etc.) are now included free. No separate packages needed.
</step>

<step name="setup-registration">
## Step 2: Set Up Plugin Registration

**Create a GSAP config file:**

```javascript
// lib/gsap.js (or lib/gsap.ts)
import gsap from "gsap";
import { useGSAP } from "@gsap/react";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { SplitText } from "gsap/SplitText";

// Register plugins once
gsap.registerPlugin(ScrollTrigger, SplitText);

// Optional: Set defaults
gsap.defaults({
  ease: "power2.out",
  duration: 0.6
});

// Export for use in components
export { gsap, useGSAP, ScrollTrigger, SplitText };
```

**In Next.js with App Router:**
```javascript
// lib/gsap.js
"use client"; // Required for client-side only

import gsap from "gsap";
// ... rest of setup
```
</step>

<step name="basic-animation-react">
## Step 3: Basic Animation in React

**Using useGSAP hook (recommended):**
```jsx
"use client";

import { useRef } from "react";
import { gsap, useGSAP } from "@/lib/gsap";

export default function AnimatedBox() {
  const containerRef = useRef(null);

  useGSAP(() => {
    // Animation code here
    gsap.to(".box", {
      x: 100,
      duration: 1
    });
  }, { scope: containerRef }); // Scope limits selectors to container

  return (
    <div ref={containerRef}>
      <div className="box">Animated</div>
    </div>
  );
}
```

**Key benefits of useGSAP:**
- Automatic cleanup on unmount
- Scoped selectors (won't affect other components)
- Handles React's Strict Mode properly
</step>

<step name="with-dependencies">
## Step 4: Animation with Dependencies

**Re-run animation when state changes:**
```jsx
"use client";

import { useRef, useState } from "react";
import { gsap, useGSAP } from "@/lib/gsap";

export default function ToggleAnimation() {
  const containerRef = useRef(null);
  const [isOpen, setIsOpen] = useState(false);

  useGSAP(() => {
    gsap.to(".panel", {
      height: isOpen ? "auto" : 0,
      duration: 0.3
    });
  }, { scope: containerRef, dependencies: [isOpen] });

  return (
    <div ref={containerRef}>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      <div className="panel">Content here</div>
    </div>
  );
}
```
</step>

<step name="scrolltrigger-react">
## Step 5: ScrollTrigger in React

**Basic scroll animation:**
```jsx
"use client";

import { useRef } from "react";
import { gsap, useGSAP, ScrollTrigger } from "@/lib/gsap";

export default function ScrollSection() {
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
  }, { scope: containerRef });

  return (
    <div ref={containerRef}>
      <section className="reveal-section">
        <div className="reveal-item">Item 1</div>
        <div className="reveal-item">Item 2</div>
        <div className="reveal-item">Item 3</div>
      </section>
    </div>
  );
}
```

**Handle resize/route changes:**
```jsx
useGSAP(() => {
  const triggers = ScrollTrigger.getAll();

  // Refresh on route change (Next.js App Router)
  return () => {
    triggers.forEach(t => t.kill());
  };
}, { scope: containerRef });
```
</step>

<step name="reusable-hooks">
## Step 6: Create Reusable Hooks

**Fade-in on scroll hook:**
```jsx
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
        start: options.start ?? "top 80%",
        toggleActions: "play none none reverse"
      }
    });
  }, { scope: ref });

  return ref;
}

// Usage:
function MyComponent() {
  const fadeRef = useFadeIn({ y: 50 });
  return <div ref={fadeRef}>Fades in on scroll</div>;
}
```

**Stagger children hook:**
```jsx
// hooks/useStaggerChildren.js
"use client";

import { useRef } from "react";
import { gsap, useGSAP } from "@/lib/gsap";

export function useStaggerChildren(selector, options = {}) {
  const containerRef = useRef(null);

  useGSAP(() => {
    gsap.from(selector, {
      y: options.y ?? 30,
      opacity: 0,
      stagger: options.stagger ?? 0.1,
      duration: options.duration ?? 0.6,
      ease: options.ease ?? "power2.out",
      scrollTrigger: options.scrollTrigger
    });
  }, { scope: containerRef });

  return containerRef;
}
```
</step>

<step name="page-transitions">
## Step 7: Page Transitions (Next.js)

**Basic page transition:**
```jsx
// components/PageTransition.jsx
"use client";

import { useRef } from "react";
import { gsap, useGSAP } from "@/lib/gsap";

export default function PageTransition({ children }) {
  const containerRef = useRef(null);

  useGSAP(() => {
    gsap.from(containerRef.current, {
      opacity: 0,
      y: 20,
      duration: 0.5,
      ease: "power2.out"
    });
  }, { scope: containerRef });

  return <div ref={containerRef}>{children}</div>;
}

// In layout.jsx or page.jsx:
export default function Page() {
  return (
    <PageTransition>
      <YourPageContent />
    </PageTransition>
  );
}
```

**Exit animations (requires more setup):**
For exit animations, consider using libraries like `framer-motion` for the transition wrapper, or implementing a custom solution with Next.js route events.
</step>

<step name="common-issues">
## Step 8: Handle Common Issues

**Hydration mismatch:**
```jsx
// Set initial state in CSS to match GSAP starting state
// styles.css
.animated-element {
  opacity: 0;
  transform: translateY(30px);
}

// Component
gsap.to(".animated-element", {
  opacity: 1,
  y: 0
});
```

**Strict Mode double-mounting:**
```jsx
// useGSAP handles this automatically
// If using useEffect, use context for cleanup:
useEffect(() => {
  const ctx = gsap.context(() => {
    gsap.to(".box", { x: 100 });
  }, containerRef);

  return () => ctx.revert(); // Clean cleanup
}, []);
```

**Server Component error:**
```jsx
// Add "use client" at top of file
"use client";

// Or dynamically import
import dynamic from "next/dynamic";
const AnimatedComponent = dynamic(
  () => import("./AnimatedComponent"),
  { ssr: false }
);
```
</step>

</process>

<vue_integration>
## Vue Integration

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
</vue_integration>

<success_criteria>
Framework integration is complete when:
- Plugins registered once at app level
- useGSAP (or gsap.context) used for all animations
- Animations scoped to component refs
- No memory leaks (proper cleanup)
- No hydration mismatches
- ScrollTrigger refreshes on route changes
- Works in production build
</success_criteria>
