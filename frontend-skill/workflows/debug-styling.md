# Workflow: Debug Styling

<required_reading>
**Read these reference files NOW:**
1. references/css-modern.md
2. references/tailwind-patterns.md
3. references/anti-patterns.md
</required_reading>

<context>
CSS debugging is systematic. Use browser DevTools to inspect computed styles, identify specificity conflicts, and trace layout issues. Most problems are: specificity, box model, or flexbox/grid misunderstanding.
</context>

<process>

<step name="identify-the-problem">
Clarify the issue:
- Layout broken? (things not where expected)
- Spacing wrong? (too much, too little, inconsistent)
- Responsive issue? (works on desktop, breaks on mobile)
- Style not applying? (specificity conflict)
- Animation not working?
- Z-index stacking issue?

Get a screenshot or description of expected vs actual.
</step>

<step name="inspect-with-devtools">
Open Chrome/Firefox DevTools (F12):

**Elements panel:**
1. Select the problem element
2. Check Computed tab for actual applied styles
3. Check Styles tab for where styles come from
4. Look for crossed-out styles (overridden)

**Layout debugging:**
```
DevTools → Elements → Select element → Look for:
- Box model diagram (margin, border, padding, content)
- Flexbox/Grid badges (click to see overlay)
- Layout shift indicators
```
</step>

<step name="common-layout-issues">
**Element not visible:**
```css
/* Check these properties */
display: none;
visibility: hidden;
opacity: 0;
height: 0;
overflow: hidden;
position: absolute; /* with off-screen coordinates */
```

**Element in wrong position:**
```css
/* Check parent positioning */
position: relative; /* on parent for absolute child */

/* Check flexbox alignment */
align-items: center;
justify-content: center;

/* Check for unexpected margins */
margin: auto; /* can cause centering */
```

**Element overflowing:**
```css
/* Add to debug */
overflow: hidden; /* hides overflow */
overflow: auto;   /* adds scrollbar */
min-width: 0;     /* for flex children */
```
</step>

<step name="specificity-debugging">
**Style not applying?** Check specificity:

```
Specificity order (highest to lowest):
1. !important
2. Inline styles
3. #id selectors
4. .class, [attribute], :pseudo-class
5. element, ::pseudo-element
```

**Tailwind specificity issues:**
```html
<!-- Problem: custom class loses to Tailwind -->
<div class="p-4 custom-padding">  <!-- custom-padding may lose -->

<!-- Solution: Use Tailwind's important modifier -->
<div class="!p-8">  <!-- Forces this padding -->

<!-- Or ensure custom CSS loads after Tailwind -->
@layer components {
  .custom-padding {
    padding: 2rem;
  }
}
```
</step>

<step name="flexbox-debugging">
Common flexbox issues:

```css
/* Item not growing */
.flex-item {
  flex-grow: 1;    /* Allow growing */
  flex-shrink: 0;  /* Prevent shrinking */
  flex-basis: 0;   /* Start from 0 */
  /* Shorthand: flex: 1 0 0; */
}

/* Item overflowing */
.flex-item {
  min-width: 0;  /* Allow shrinking below content size */
  overflow: hidden;
}

/* Items not wrapping */
.flex-container {
  flex-wrap: wrap;
}

/* Alignment issues */
.flex-container {
  align-items: stretch;   /* Vertical (cross-axis) */
  justify-content: start; /* Horizontal (main-axis) */
}
```
</step>

<step name="grid-debugging">
Common grid issues:

```css
/* Item spanning wrong */
.grid-item {
  grid-column: span 2;  /* Spans 2 columns */
  grid-row: 1 / 3;      /* Rows 1 and 2 */
}

/* Grid not responsive */
.grid-container {
  /* Fixed columns - not responsive */
  grid-template-columns: 200px 200px 200px;

  /* Responsive - auto-fit with minmax */
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}

/* Gap not working */
.grid-container {
  gap: 1rem;  /* Modern - use this */
  grid-gap: 1rem;  /* Legacy - avoid */
}
```
</step>

<step name="responsive-debugging">
Test at specific breakpoints:

```
DevTools → Toggle device toolbar (Ctrl+Shift+M)
Or: Responsive mode and drag to resize
```

**Tailwind breakpoints:**
```
sm: 640px
md: 768px
lg: 1024px
xl: 1280px
2xl: 1536px
```

**Debug responsive classes:**
```html
<!-- Add visible indicators during debug -->
<div class="hidden sm:block md:hidden lg:block">
  <!-- Only visible at sm and lg+ -->
</div>

<!-- Temporary debug background -->
<div class="bg-red-500 sm:bg-blue-500 md:bg-green-500">
  Resize to see breakpoint changes
</div>
```
</step>

<step name="z-index-debugging">
Z-index stacking context issues:

```css
/* New stacking context created by: */
position: relative/absolute/fixed + z-index
transform
opacity < 1
filter
will-change

/* Debug: Add outline to see layers */
* { outline: 1px solid red; }

/* Common fix: Ensure parent has stacking context */
.modal-container {
  position: relative;
  z-index: 50;
}

.modal {
  position: fixed;
  z-index: 100;  /* Relative to modal-container's stacking context */
}
```
</step>

<step name="tailwind-specific">
**Class not working:**
```bash
# Check if class is being purged
# Look in build output or browser DevTools

# In Tailwind v4, ensure file is in content paths
# Check that class exists in Tailwind
```

**Dynamic classes not working:**
```tsx
// Bad: Dynamic class construction
<div className={`text-${color}-500`}>  // Won't work - gets purged

// Good: Complete class names
const colorClasses = {
  red: 'text-red-500',
  blue: 'text-blue-500',
};
<div className={colorClasses[color]}>
```
</step>

<step name="verify-fix">
After fixing:
1. Clear browser cache (Ctrl+Shift+R)
2. Test at multiple breakpoints
3. Test in different browsers if relevant
4. Check that fix doesn't break other things
</step>

</process>

<debugging_checklist>
```markdown
- [ ] Inspected element in DevTools
- [ ] Checked computed styles
- [ ] Identified specificity conflicts
- [ ] Verified box model (margin, padding, border)
- [ ] Checked flexbox/grid parent settings
- [ ] Tested responsive breakpoints
- [ ] Verified Tailwind classes aren't purged
- [ ] Cleared cache after fix
```
</debugging_checklist>

<success_criteria>
- Root cause identified (not just symptoms treated)
- Fix is minimal (doesn't add unnecessary complexity)
- No `!important` hacks (fix specificity properly)
- Works across breakpoints
- Doesn't break other components
</success_criteria>
