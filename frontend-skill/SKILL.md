---
name: frontend-skill
description: Expert frontend developer and UI/UX design master. Build beautiful, performant interfaces across React/Next.js, Vue/Nuxt, Laravel/Blade, and React Native. Expertise in Tailwind CSS, modern CSS, component patterns, and optimization. Full lifecycle coverage from design through shipping.
---

<essential_principles>

<principle name="beauty-through-fundamentals">
Beautiful UI comes from mastering fundamentals, not adding more. Visual hierarchy, typography, spacing, and color theory create elegance. When something looks "off," check these first:
- Is there clear visual hierarchy? (size, weight, color contrast)
- Is spacing consistent? (8px grid, consistent padding/margins)
- Is typography limited? (2-3 font sizes max per component)
- Is there breathing room? (whitespace is not wasted space)
</principle>

<principle name="performance-is-ux">
A beautiful site that loads slowly is not beautiful. Performance IS user experience. Prioritize:
- Core Web Vitals (LCP, INP, CLS)
- Bundle size (code-split, lazy load)
- Render performance (minimize re-renders)
- Asset optimization (images, fonts, animations)
</principle>

<principle name="accessibility-is-non-negotiable">
Accessible design is better design for everyone. Every component must:
- Use semantic HTML first (ARIA only when HTML is insufficient)
- Support keyboard navigation
- Have sufficient color contrast (4.5:1 for text)
- Respect `prefers-reduced-motion`
- Work with screen readers (test with VoiceOver/NVDA)
</principle>

<principle name="utility-first-with-purpose">
Tailwind's utility-first approach works when you understand WHY. Don't just copy classes:
- Understand the design system (spacing scale, color palette)
- Extract components for repeated patterns
- Use `@apply` sparingly (prefer component extraction)
- Configure design tokens in CSS (`@theme` in Tailwind v4)
</principle>

<principle name="style-system-not-scattered">
When implementing a visual style (retro, minimal, cyberpunk, etc.):
- Create a design SYSTEM with reusable utility classes, not scattered one-off styles
- Apply to ALL pages in the same session - inconsistency is immediately noticed
- Identify 2-3 "signature elements" that define the style (e.g., hard shadows, thick borders)
- Style must ENHANCE UX, not replace it - hierarchy, contrast, and spacing still matter
- If you can't describe the style in 3 words, it's not clear enough to implement well
See `references/core/ui-style-lessons.md` for real pass/fail examples.
</principle>

<principle name="simplest-solution-first">
When fixing layout or responsive issues, reach for the simplest CSS tool first:
- z-index layering before breakpoint micromanagement
- overflow: hidden before calc() repositioning
- Padding before min-height for spacing
- One breakpoint (show/hide) before five granular size steps
- Pin to edges (left:0/right:0) before calc(50% - Xpx)
- `min-w-0` on flex items before overflow hacks on children (sidebar layouts!)
If your fix requires more than 2-3 custom breakpoints, you're solving the wrong problem.
See `references/core/ui-style-lessons.md` for the decorative-elements case study.
See `references/core/anti-patterns.md` `flex-item-overflow-sidebar-layout` for the sidebar overflow case study.
</principle>

</essential_principles>

<intake>
## What framework are you using?

1. **React / Next.js** - React patterns, Server Components, hooks
2. **Vue / Nuxt** - Composition API, Nuxt UI, Vue patterns
3. **Laravel / Blade** - Blade components, Livewire, Inertia
4. **React Native** - Mobile app styling, native components
5. **Framework-agnostic** - CSS, Tailwind, design principles

**After framework selection, what would you like help with?**

1. **Tailwind Expert** - Advanced Tailwind CSS, design systems, responsive patterns
2. **CSS Expert** - Modern CSS features, Grid, Flexbox, animations, container queries
3. **Component Optimization** - Performance, bundle size, rendering
4. **Review UI** - Critique and improve existing designs
5. **Build Component** - Create a new component from scratch
6. **Debug Styling** - Fix layout, responsiveness, or visual issues
7. **Something else** - Describe what you need

**Wait for response before proceeding.**
</intake>

<routing>
## Framework Selection

| Response | Framework | References to Load |
|----------|-----------|-------------------|
| 1, "react", "next", "nextjs" | React/Next.js | `references/react/patterns.md` + `references/react/performance.md` |
| 2, "vue", "nuxt" | Vue/Nuxt | `references/vue/patterns.md` |
| 3, "laravel", "blade", "livewire", "inertia" | Laravel | `references/laravel/patterns.md` |
| 4, "react native", "mobile", "ios", "android" | React Native | `references/react-native/patterns.md` |
| 5, "css", "tailwind", "general", "agnostic" | Core only | Core references only |

## Task Routing

| Response | Workflow |
|----------|----------|
| 1, "tailwind", "utility", "design system", "tokens" | `workflows/tailwind-expert.md` |
| 2, "css", "grid", "flexbox", "container queries", ":has" | `workflows/css-expert.md` |
| 3, "performance", "optimize", "bundle", "fast", "slow" | `workflows/component-optimization.md` |
| 4, "review", "critique", "feedback", "improve" | `workflows/review-ui.md` |
| 5, "build", "create", "new component" | `workflows/build-component.md` |
| 6, "debug", "fix", "broken", "layout", "responsive" | `workflows/debug-styling.md` |
| 7, other | Clarify intent, then route to appropriate workflow |

**After identifying framework and task:**
1. Read the framework-specific references first
2. Read the workflow file
3. Always include relevant core references (design-principles, tailwind-patterns, etc.)

</routing>

<verification_loop>
After every change, verify:

**Web (React, Vue, Laravel):**
```bash
# 1. Does it build? ONLY if project allows - check CLAUDE.md first.
# Many projects prohibit running builds for verification.
# If build is restricted, rely on dev server hot reload + browser check.
npm run build  # or php artisan optimize - ONLY if explicitly allowed

# 2. Do tests pass?
npm test

# 3. Check bundle size (only if build is allowed)
npm run build -- --analyze  # if available

# 4. Lighthouse audit
# Run in Chrome DevTools > Lighthouse
```

**React Native:**
```bash
# 1. Does it build?
npx react-native run-ios  # or run-android

# 2. Check with Flipper profiler
# 3. Test on actual device
```

Report to the user:
- "Build: OK"
- "Tests: X pass, Y fail"
- "Bundle size: X KB (gzipped)"
- "Lighthouse: Performance X, Accessibility Y"
</verification_loop>

<reference_index>
## Domain Knowledge

### Core (Framework-Agnostic)
All in `references/core/`:
- tailwind-patterns.md - Tailwind CSS v4 patterns and configuration
- css-modern.md - Modern CSS features (Grid, Flexbox, container queries)
- design-principles.md - Visual hierarchy, typography, spacing, color
- accessibility.md - WCAG compliance, screen readers, keyboard nav
- responsive-design.md - Mobile-first, breakpoints, fluid design
- animation-motion.md - CSS animations, transitions, motion design
- ui-style-lessons.md - Real-world pass/fail cases
- anti-patterns.md - Common mistakes across frameworks

### React / Next.js
All in `references/react/`:
- patterns.md - React 19, Server Components, hooks, state management
- performance.md - Core Web Vitals, bundle optimization, image handling

### Vue / Nuxt
All in `references/vue/`:
- patterns.md - Composition API, Nuxt UI, composables, transitions

### Laravel
All in `references/laravel/`:
- patterns.md - Blade components, Livewire, Inertia.js, Alpine.js

### React Native
All in `references/react-native/`:
- patterns.md - StyleSheet, Flexbox, theming, animations, performance
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| tailwind-expert.md | Advanced Tailwind CSS mastery |
| css-expert.md | Modern CSS features and techniques |
| component-optimization.md | Performance and bundle optimization |
| review-ui.md | Critique and improve existing UI |
| build-component.md | Create new components from scratch |
| debug-styling.md | Fix styling and layout issues |
</workflows_index>

<quick_reference>
## Framework Component Libraries

| Framework | Recommended Library |
|-----------|---------------------|
| React/Next.js | shadcn/ui, Radix UI, Headless UI |
| Vue/Nuxt | Nuxt UI, Headless UI Vue, PrimeVue |
| Laravel | Blade components, Livewire |
| React Native | React Native Paper, NativeBase |

## Tailwind v4 Quick Setup

```css
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.6 0.15 250);
  --font-sans: "Inter", system-ui, sans-serif;
}
```
</quick_reference>

<context7_usage>
## Getting Current Documentation

**React/Next.js:**
```
mcp__context7__query-docs:
  libraryId: "/websites/react_dev"
  query: "your question"
```

**Vue/Nuxt:**
```
mcp__context7__query-docs:
  libraryId: "/websites/ui3_nuxt"
  query: "your question"
```

**React Native:**
```
mcp__context7__query-docs:
  libraryId: "/websites/reactnative_dev"
  query: "your question"
```

**Tailwind CSS:**
```
mcp__context7__query-docs:
  libraryId: "/tailwindlabs/tailwindcss"
  query: "your question"
```
</context7_usage>
