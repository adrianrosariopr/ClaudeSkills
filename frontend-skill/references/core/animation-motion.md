<overview>
Animation and motion design: CSS transitions/animations, Framer Motion, micro-interactions, performance, and accessibility considerations.
</overview>

<when_to_animate>
**Animate to:**
- Provide feedback (button click, form submit)
- Guide attention (new notification, error state)
- Show relationships (expand/collapse, navigation)
- Create delight (subtle hover effects, loading states)

**Don't animate:**
- Everything (too distracting)
- Critical information (may be missed)
- Without purpose (animation for animation's sake)
- Long durations for frequent actions
</when_to_animate>

<css_transitions>
**Basic transitions:**
```css
.button {
  background: var(--color-primary);
  transition: background-color 150ms ease-out;
}

.button:hover {
  background: var(--color-primary-dark);
}

/* Multiple properties */
.card {
  transition:
    transform 200ms ease-out,
    box-shadow 200ms ease-out;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: var(--shadow-lg);
}

/* All properties (use sparingly) */
.element {
  transition: all 150ms ease-out;
}
```

**Transition timing:**
| Duration | Use case |
|----------|----------|
| 100-150ms | Micro-interactions (hover, click) |
| 200-300ms | Small elements (buttons, cards) |
| 300-500ms | Medium elements (modals, drawers) |
| 500ms+ | Large/complex animations |

**Easing functions:**
```css
/* Built-in */
ease        /* Default, good for most */
ease-in     /* Starts slow, ends fast */
ease-out    /* Starts fast, ends slow (recommended for enter) */
ease-in-out /* Slow start and end */
linear      /* Constant speed */

/* Custom cubic-bezier */
transition: transform 300ms cubic-bezier(0.34, 1.56, 0.64, 1);
```
</css_transitions>

<css_keyframes>
**Keyframe animations:**
```css
/* Fade in */
@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fade-in {
  animation: fade-in 300ms ease-out forwards;
}

/* Pulse */
@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.animate-pulse {
  animation: pulse 2s ease-in-out infinite;
}

/* Spin */
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.animate-spin {
  animation: spin 1s linear infinite;
}

/* Slide in from right */
@keyframes slide-in-right {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}
```

**Animation properties:**
```css
.element {
  animation-name: fade-in;
  animation-duration: 300ms;
  animation-timing-function: ease-out;
  animation-delay: 100ms;
  animation-iteration-count: 1;  /* or infinite */
  animation-direction: normal;   /* or reverse, alternate */
  animation-fill-mode: forwards; /* Keep end state */

  /* Shorthand */
  animation: fade-in 300ms ease-out 100ms forwards;
}
```
</css_keyframes>

<framer_motion>
**Basic animations:**
```tsx
import { motion } from 'framer-motion';

// Simple animation
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.3 }}
>
  Content
</motion.div>

// Hover and tap
<motion.button
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.95 }}
  transition={{ type: 'spring', stiffness: 400, damping: 17 }}
>
  Click me
</motion.button>

// Exit animations
<AnimatePresence>
  {isVisible && (
    <motion.div
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      exit={{ opacity: 0 }}
    >
      Content
    </motion.div>
  )}
</AnimatePresence>
```

**Variants for complex animations:**
```tsx
const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1,
    },
  },
};

const itemVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0 },
};

function List({ items }) {
  return (
    <motion.ul
      variants={containerVariants}
      initial="hidden"
      animate="visible"
    >
      {items.map(item => (
        <motion.li key={item.id} variants={itemVariants}>
          {item.name}
        </motion.li>
      ))}
    </motion.ul>
  );
}
```

**Layout animations:**
```tsx
// Automatic layout animation
<motion.div layout>
  Content that changes size
</motion.div>

// Shared layout animation (element moves between positions)
<motion.div layoutId="shared-element">
  This element animates between positions
</motion.div>

// In different components
function Card({ isExpanded }) {
  return (
    <motion.div layout>
      <motion.h2 layout="position">Title</motion.h2>
      {isExpanded && <motion.p layout>Details...</motion.p>}
    </motion.div>
  );
}
```

**Gestures:**
```tsx
<motion.div
  drag
  dragConstraints={{ left: 0, right: 300, top: 0, bottom: 300 }}
  dragElastic={0.2}
  onDragEnd={(e, info) => {
    if (info.offset.x > 100) {
      // Swiped right
    }
  }}
>
  Draggable content
</motion.div>
```
</framer_motion>

<micro_interactions>
**Button feedback:**
```tsx
// With Framer Motion
<motion.button
  whileHover={{ scale: 1.02 }}
  whileTap={{ scale: 0.98 }}
  className="bg-primary text-white px-4 py-2 rounded-md"
>
  Click me
</motion.button>

// With CSS
.button {
  transition: transform 100ms ease-out;
}
.button:hover {
  transform: scale(1.02);
}
.button:active {
  transform: scale(0.98);
}
```

**Input focus:**
```css
.input {
  border: 2px solid var(--color-gray-300);
  transition: border-color 150ms ease-out, box-shadow 150ms ease-out;
}

.input:focus {
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px var(--color-primary-light);
  outline: none;
}
```

**Loading states:**
```tsx
// Skeleton loading
<div className="animate-pulse">
  <div className="h-4 bg-gray-200 rounded w-3/4 mb-2" />
  <div className="h-4 bg-gray-200 rounded w-1/2" />
</div>

// Spinner
<motion.div
  animate={{ rotate: 360 }}
  transition={{ duration: 1, repeat: Infinity, ease: 'linear' }}
  className="w-6 h-6 border-2 border-primary border-t-transparent rounded-full"
/>
```

**Toggle switch:**
```tsx
function Toggle({ checked, onChange }) {
  return (
    <button
      role="switch"
      aria-checked={checked}
      onClick={() => onChange(!checked)}
      className={`
        relative w-12 h-6 rounded-full transition-colors
        ${checked ? 'bg-primary' : 'bg-gray-300'}
      `}
    >
      <motion.span
        className="absolute top-1 left-1 w-4 h-4 bg-white rounded-full"
        animate={{ x: checked ? 24 : 0 }}
        transition={{ type: 'spring', stiffness: 500, damping: 30 }}
      />
    </button>
  );
}
```
</micro_interactions>

<performance>
**GPU-accelerated properties (prefer these):**
- `transform` (translate, scale, rotate)
- `opacity`

**Trigger layout/paint (avoid animating):**
- `width`, `height`
- `top`, `left`, `right`, `bottom`
- `margin`, `padding`
- `border-width`
- `font-size`

**Optimization tips:**
```css
/* Hint to browser for optimization */
.will-animate {
  will-change: transform, opacity;
}

/* Remove after animation */
.animated {
  will-change: auto;
}

/* Use transform instead of position */
.bad { left: 100px; }
.good { transform: translateX(100px); }

/* Use opacity instead of visibility changes */
.bad { visibility: hidden; }
.good { opacity: 0; pointer-events: none; }
```

**Framer Motion performance:**
```tsx
// Use layout animations sparingly
// Prefer transform-based animations
<motion.div
  initial={{ opacity: 0, scale: 0.9 }}
  animate={{ opacity: 1, scale: 1 }}
  // Not: initial={{ height: 0 }} animate={{ height: 'auto' }}
/>
```
</performance>

<accessibility>
**Respect user preferences:**
```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

```tsx
// React hook
function usePrefersReducedMotion() {
  const [prefersReduced, setPrefersReduced] = useState(false);

  useEffect(() => {
    const query = window.matchMedia('(prefers-reduced-motion: reduce)');
    setPrefersReduced(query.matches);

    const handler = (e) => setPrefersReduced(e.matches);
    query.addEventListener('change', handler);
    return () => query.removeEventListener('change', handler);
  }, []);

  return prefersReduced;
}

// Usage with Framer Motion
function AnimatedComponent() {
  const prefersReduced = usePrefersReducedMotion();

  return (
    <motion.div
      initial={prefersReduced ? false : { opacity: 0 }}
      animate={{ opacity: 1 }}
      transition={{ duration: prefersReduced ? 0 : 0.3 }}
    >
      Content
    </motion.div>
  );
}
```

**Animation guidelines:**
- Keep animations short (< 500ms for most)
- Avoid flashing (no more than 3 flashes per second)
- Don't auto-play videos with motion
- Provide pause/stop controls for long animations
- Essential information shouldn't depend on animation
</accessibility>

<choosing_tools>
| Tool | Best for |
|------|----------|
| CSS Transitions | Simple state changes (hover, focus) |
| CSS Keyframes | Repeating animations (loading, pulse) |
| Framer Motion | React apps, complex animations, gestures |
| GSAP | Complex timelines, scroll animations |
| Lottie | Vector animations from After Effects |
| View Transitions API | Page transitions |

**Decision tree:**
1. Simple hover/focus effect? → CSS transition
2. Looping animation? → CSS keyframes
3. React + complex animation? → Framer Motion
4. Scroll-driven or timeline? → GSAP
5. Designer-created animation? → Lottie
</choosing_tools>
