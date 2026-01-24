<overview>
UI/UX design fundamentals: visual hierarchy, typography, spacing, color theory, and the principles that make interfaces beautiful and usable.
</overview>

<visual_hierarchy>
**Guide the eye through deliberate design choices:**

**Size:** Larger = more important
```
Page title: 32-48px
Section heading: 24-32px
Subsection: 18-24px
Body text: 14-16px
Small/caption: 12-14px
```

**Weight:** Bolder = more attention
```
Primary heading: font-bold (700)
Secondary heading: font-semibold (600)
Body text: font-normal (400)
Subtle text: font-normal with lower opacity
```

**Color:** High contrast = important
```
Primary action: Brand color, high saturation
Secondary action: Muted, lower contrast
Disabled: Low contrast, grayed out
Destructive: Red/danger color
```

**Position:** Top-left gets seen first (F-pattern reading)
- Logo/brand: top-left
- Primary navigation: top
- Main content: upper-left area
- Secondary actions: bottom or right

**Whitespace:** Isolation draws attention
```
Important element surrounded by space > cluttered element
```
</visual_hierarchy>

<typography>
**Font selection:**
- 1 font family for most projects
- 2 max (one for headings, one for body)
- System fonts for performance, custom for brand

**Type scale (suggested):**
```css
--text-xs: 0.75rem;   /* 12px */
--text-sm: 0.875rem;  /* 14px */
--text-base: 1rem;    /* 16px */
--text-lg: 1.125rem;  /* 18px */
--text-xl: 1.25rem;   /* 20px */
--text-2xl: 1.5rem;   /* 24px */
--text-3xl: 1.875rem; /* 30px */
--text-4xl: 2.25rem;  /* 36px */
```

**Line height:**
- Headings: 1.1 - 1.3
- Body text: 1.4 - 1.6
- Long-form content: 1.6 - 1.8

**Line length:**
- Optimal: 50-75 characters
- Maximum: 80 characters
- Use `max-width` to constrain

```css
.prose {
  max-width: 65ch;  /* ~65 characters */
  line-height: 1.6;
}
```

**Font pairing tips:**
- Contrast in style (serif + sans-serif)
- Similar x-height
- Complementary mood
```
Headings: Inter, Poppins, Montserrat (geometric sans)
Body: Inter, Source Sans, Open Sans (humanist sans)
Accent: Playfair Display, Lora (serif)
Code: JetBrains Mono, Fira Code (monospace)
```
</typography>

<spacing>
**8px grid system:**
```
4px  (0.25rem) - tight spacing, icon padding
8px  (0.5rem)  - small spacing, inline elements
16px (1rem)    - default spacing, form elements
24px (1.5rem)  - medium spacing, card padding
32px (2rem)    - large spacing, section gaps
48px (3rem)    - section spacing
64px (4rem)    - major section breaks
```

**Spacing principles:**
- Related items: closer together
- Unrelated items: further apart
- Consistent scale (don't use arbitrary values)
- More space around important elements

**Component spacing:**
```css
/* Button */
padding: 0.5rem 1rem;  /* 8px 16px */

/* Card */
padding: 1.5rem;  /* 24px */
gap: 1rem;  /* 16px between elements */

/* Form field */
margin-bottom: 1.5rem;  /* 24px */

/* Section */
padding: 4rem 0;  /* 64px vertical */
```

**Touch targets:**
- Minimum: 44x44px (Apple HIG)
- Recommended: 48x48px
- Spacing between targets: 8px minimum
</spacing>

<color>
**Color palette structure:**
```
Primary: Brand color (1-2 shades)
Secondary: Supporting color
Accent: Call-to-action, highlights
Neutral: Gray scale for text, borders, backgrounds
Semantic: Success (green), Warning (yellow), Error (red), Info (blue)
```

**Building a palette:**
```css
:root {
  /* Primary - brand color */
  --color-primary-50: oklch(0.95 0.03 250);
  --color-primary-100: oklch(0.9 0.05 250);
  --color-primary-500: oklch(0.6 0.15 250);  /* Base */
  --color-primary-600: oklch(0.5 0.15 250);
  --color-primary-900: oklch(0.25 0.1 250);

  /* Neutral - gray scale */
  --color-gray-50: oklch(0.98 0 0);
  --color-gray-100: oklch(0.95 0 0);
  --color-gray-200: oklch(0.9 0 0);
  --color-gray-500: oklch(0.5 0 0);
  --color-gray-800: oklch(0.2 0 0);
  --color-gray-900: oklch(0.12 0 0);

  /* Semantic */
  --color-success: oklch(0.65 0.15 145);
  --color-warning: oklch(0.8 0.15 85);
  --color-error: oklch(0.6 0.2 25);
}
```

**Contrast requirements (WCAG):**
- Normal text: 4.5:1 minimum
- Large text (18px+ or 14px+ bold): 3:1 minimum
- UI components: 3:1 minimum
- Use contrast checker tools

**Color usage:**
- Don't rely on color alone (add icons, text)
- Test for color blindness (Sim Daltonism, Coblis)
- Consider dark mode from the start
</color>

<whitespace>
**Whitespace is not empty space - it's a design element.**

**Benefits:**
- Improves readability
- Creates visual hierarchy
- Reduces cognitive load
- Feels premium/professional

**Types:**
- Macro whitespace: Between sections, major elements
- Micro whitespace: Between lines, letters, small elements

**Common mistakes:**
- Cramming too much content
- Inconsistent spacing
- Fear of "empty" space
- Not enough padding in containers

**Rule of thumb:** When in doubt, add more space.
</whitespace>

<visual_balance>
**Symmetrical balance:**
- Centered layouts
- Equal weight on both sides
- Formal, stable, traditional

**Asymmetrical balance:**
- Off-center focal point
- Balanced by varying element sizes/colors
- Dynamic, modern, interesting

**Visual weight factors:**
- Size: Larger = heavier
- Color: Darker/saturated = heavier
- Position: Lower/right = heavier
- Density: More detail = heavier
- Isolation: Alone = heavier
</visual_balance>

<design_checklist>
```markdown
Visual Hierarchy
- [ ] Clear primary action per screen
- [ ] Obvious reading order
- [ ] Size/weight/color establish importance

Typography
- [ ] 2-3 font sizes maximum
- [ ] Comfortable line height (1.4-1.6)
- [ ] Line length under 80 characters
- [ ] Clear hierarchy (h1 > h2 > h3 > body)

Spacing
- [ ] Consistent spacing scale (8px grid)
- [ ] Related items grouped together
- [ ] Adequate padding in containers
- [ ] Touch targets 44px+ minimum

Color
- [ ] Limited, purposeful palette
- [ ] Sufficient contrast (4.5:1 for text)
- [ ] Not relying on color alone
- [ ] Works in light and dark mode

Balance
- [ ] Intentional whitespace
- [ ] Visual weight distributed appropriately
- [ ] No cramped or cluttered areas
```
</design_checklist>
