# Workflow: Review UI

<required_reading>
**Read these reference files NOW:**
1. references/design-principles.md
2. references/accessibility.md
3. references/anti-patterns.md
</required_reading>

<context>
UI review is systematic evaluation against design fundamentals: visual hierarchy, spacing, typography, color, accessibility, and responsiveness. Be specific and actionable in feedback.
</context>

<process>

<step name="gather-context">
Before reviewing, understand:
- What is this UI for? (purpose, user goals)
- Who are the users? (expertise level, context)
- What's the design system/brand? (constraints)
- Is there a specific concern? (or general review)

Ask the user if needed.
</step>

<step name="visual-hierarchy-check">
**Does the eye know where to go?**

Check:
- [ ] Clear primary action (one main CTA per screen)
- [ ] Headings establish content structure
- [ ] Size/weight/color create obvious importance levels
- [ ] Whitespace groups related content

Common issues:
- Everything same size = nothing stands out
- Too many CTAs competing for attention
- Dense text walls without visual breaks
</step>

<step name="spacing-consistency">
**Is spacing systematic?**

Check:
- [ ] Consistent spacing scale (4px, 8px, 16px, 24px, 32px)
- [ ] Related items closer together
- [ ] Sections clearly separated
- [ ] Adequate padding in clickable areas (min 44px tap targets)

Common issues:
- Magic numbers (17px, 23px) instead of scale
- Cramped content with no breathing room
- Inconsistent gaps between similar elements
</step>

<step name="typography-audit">
**Is type helping or hurting?**

Check:
- [ ] 2-3 font sizes max (heading, body, small)
- [ ] Line height readable (1.4-1.6 for body)
- [ ] Line length comfortable (50-75 characters)
- [ ] Font weights create hierarchy (not just size)
- [ ] Text is legible (contrast, size)

Common issues:
- Too many font sizes
- Lines too long or too short
- Poor contrast on text
</step>

<step name="color-review">
**Is color purposeful?**

Check:
- [ ] Limited color palette (1-2 primary, 1-2 accent, neutrals)
- [ ] Color has meaning (not decorative randomness)
- [ ] Sufficient contrast (4.5:1 for text, 3:1 for UI)
- [ ] Color not only indicator (accessible for colorblind)

Common issues:
- Too many colors
- Low contrast text
- Red/green only indicators
</step>

<step name="accessibility-check">
**Can everyone use this?**

Check:
- [ ] Semantic HTML (`<button>`, `<nav>`, `<main>`)
- [ ] Keyboard navigable (Tab, Enter, Escape)
- [ ] Focus states visible
- [ ] Alt text on images
- [ ] Form labels associated
- [ ] Error messages clear and helpful
- [ ] Motion respects `prefers-reduced-motion`

Test:
- Tab through the interface
- Use with screen reader (VoiceOver/NVDA)
- Check contrast with browser tools
</step>

<step name="responsive-check">
**Does it work everywhere?**

Check at:
- [ ] Mobile (375px)
- [ ] Tablet (768px)
- [ ] Desktop (1024px+)

Look for:
- Content overflow/truncation
- Touch target sizes on mobile
- Layout breaks
- Hidden important content
</step>

<step name="provide-feedback">
Structure feedback as:

**What's working:**
- Specific positives (not generic praise)

**Priority issues (fix these):**
1. Issue + specific fix
2. Issue + specific fix

**Suggestions (nice to have):**
- Lower priority improvements

Be specific:
- Bad: "The spacing feels off"
- Good: "The card padding is 12px but should be 16px to match the design system. The gap between cards is 8px but 16px would give more breathing room."
</step>

</process>

<review_template>
```markdown
## UI Review: [Component/Page Name]

### What's Working
- [Specific positive]
- [Specific positive]

### Priority Issues
1. **[Issue]**: [Description]
   - Current: [What it is now]
   - Recommended: [What it should be]

2. **[Issue]**: [Description]
   - Current: [What it is now]
   - Recommended: [What it should be]

### Suggestions
- [Nice-to-have improvement]
- [Nice-to-have improvement]

### Accessibility
- [Pass/Fail]: [Specific check]
- [Pass/Fail]: [Specific check]
```
</review_template>

<success_criteria>
- Specific, actionable feedback (not vague)
- Priority issues clearly identified
- Accessibility explicitly checked
- Responsive behavior verified
- Fixes are implementable (not just "make it better")
</success_criteria>
