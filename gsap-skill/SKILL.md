---
name: gsap-skill
description: Build performant, creative GSAP animations from scratch through optimization. Full lifecycle - create, debug, optimize, integrate. Covers ScrollTrigger, SplitText, SVG animations, and framework integration.
---

<essential_principles>

<principle name="animate-transforms-only">
**Only animate transform and opacity.** These properties are GPU-accelerated and don't trigger layout recalculation.

- **Use:** `x`, `y`, `scale`, `scaleX`, `scaleY`, `rotation`, `opacity`
- **Avoid:** `left`, `top`, `width`, `height`, `margin`, `padding`

```javascript
// Good - GPU accelerated
gsap.to(".box", { x: 100, opacity: 0.5 });

// Bad - triggers layout recalculation
gsap.to(".box", { left: 100, width: 200 });
```
</principle>

<principle name="use-timelines">
**Use timelines for sequences, not delays.** Timelines make animations maintainable, reversible, and controllable.

```javascript
// Good - use timeline
const tl = gsap.timeline();
tl.to(".a", { x: 100 })
  .to(".b", { y: 50 })
  .to(".c", { opacity: 0 });

// Bad - manual delays
gsap.to(".a", { x: 100 });
gsap.to(".b", { y: 50, delay: 0.5 });
gsap.to(".c", { opacity: 0, delay: 1 });
```
</principle>

<principle name="cleanup-in-frameworks">
**Always clean up in React/Next.js.** Use `useGSAP` hook or manually kill animations on unmount.

```javascript
import { useGSAP } from "@gsap/react";

useGSAP(() => {
  gsap.to(".box", { x: 100 });
}, { scope: containerRef }); // Auto-cleanup on unmount
```
</principle>

<principle name="no-css-transitions">
**Never mix CSS transitions with GSAP.** CSS transitions conflict with GSAP and cause performance issues.

```css
/* Bad - conflicts with GSAP */
.box { transition: transform 0.3s; }

/* Good - let GSAP handle all animation */
.box { /* no transition */ }
```
</principle>

<principle name="force3d-default">
**GSAP uses `force3D: "auto"` by default.** This enables GPU acceleration during animation and releases GPU memory after. Don't override unless you have a specific reason.
</principle>

</essential_principles>

<intake>
What would you like to do?

1. Create a new animation
2. Add scroll-triggered effects
3. Add text animations
4. Add interactivity (hover, click, drag)
5. Debug a broken/janky animation
6. Optimize animation performance
7. Integrate with React/Next.js/Vue
8. Something else

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "new", "create", "build", "animation" | `workflows/create-animation.md` |
| 2, "scroll", "scrolltrigger", "parallax", "pin" | `workflows/add-scroll-effects.md` |
| 3, "text", "splittext", "reveal", "letters" | `workflows/add-text-animation.md` |
| 4, "hover", "click", "drag", "interactive" | `workflows/add-interactivity.md` |
| 5, "debug", "broken", "fix", "janky", "not working" | `workflows/debug-animation.md` |
| 6, "optimize", "performance", "slow", "laggy" | `workflows/optimize-performance.md` |
| 7, "react", "next", "vue", "framework" | `workflows/integrate-framework.md` |
| 8, other | Clarify, then select workflow or references |

**After reading the workflow, follow it exactly.**
</routing>

<verification_loop>
After every animation change:

```javascript
// 1. Check console for errors
// Open DevTools Console

// 2. Verify targets exist
gsap.utils.toArray(".your-selector"); // Should return elements

// 3. Test animation
const tl = gsap.timeline();
tl.play(); // Does it run?
tl.reverse(); // Does it reverse?
tl.progress(0.5); // Can you scrub?

// 4. Check performance
// DevTools > Performance > Record while scrolling/animating
// Look for long frames (>16ms) or layout thrashing
```

Report to user:
- "Animation running: ✓"
- "Performance: X fps average"
- "Issues found: [list or none]"
</verification_loop>

<reference_index>

**Core Concepts:** core-concepts.md (tweens, timelines, easing)
**Performance:** performance.md (GPU, force3D, optimization)
**ScrollTrigger:** scrolltrigger.md (triggers, pinning, scrubbing)
**Text:** text-animations.md (SplitText, reveals, masking)
**SVG:** svg-animations.md (DrawSVG, MorphSVG, paths)
**Frameworks:** framework-integration.md (React, Next.js, Vue)
**Timelines:** timeline-patterns.md (organization, nesting)
**Creative:** creative-effects.md (parallax, staggers, interactions)
**Anti-patterns:** anti-patterns.md (common mistakes)

</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| create-animation.md | Build new animations from scratch |
| add-scroll-effects.md | ScrollTrigger, parallax, scroll-linked |
| add-text-animation.md | SplitText, text reveals, character animations |
| add-interactivity.md | Hover, click, drag interactions |
| debug-animation.md | Fix broken/janky animations |
| optimize-performance.md | Profile and improve performance |
| integrate-framework.md | React, Next.js, Vue integration |
</workflows_index>

<quick_reference>

**GSAP 3.13+ (2024-2025):** All plugins now FREE including SplitText, MorphSVG, ScrollSmoother (thanks to Webflow acquisition).

**Installation:**
```bash
npm install gsap
```

**Basic Import:**
```javascript
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { SplitText } from "gsap/SplitText";

gsap.registerPlugin(ScrollTrigger, SplitText);
```

**Essential Easing:**
- `power2.out` - Default for 90% of UI animations
- `.out` - Use for entrances (responds immediately)
- `.in` - Use for exits
- `.inOut` - Use for loops

</quick_reference>
