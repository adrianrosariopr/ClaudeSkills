# Workflow: Fix Accessibility Issues

<required_reading>
**Read these reference files NOW:**
1. references/accessibility/wcag-overview.md
2. references/accessibility/semantic-html.md
3. references/accessibility/aria-landmarks.md
</required_reading>

<process>

## Step 1: Prioritize Issues

Order fixes by impact and legal risk:

1. **Level A violations** (baseline compliance)
2. **Level AA violations** (legal standard)
3. **Keyboard accessibility** (affects many users)
4. **Screen reader compatibility** (blind users)
5. **Visual accessibility** (low vision users)

## Step 2: Fix Image Accessibility

**Missing alt text:**
```html
<!-- BAD -->
<img src="hero.jpg">

<!-- GOOD: Informative image -->
<img src="hero.jpg" alt="Team collaborating around a whiteboard">

<!-- GOOD: Decorative image -->
<img src="decorative-border.png" alt="">
```

**Complex images (charts, diagrams):**
```html
<figure>
  <img src="sales-chart.png" alt="Q4 sales increased 25% over Q3">
  <figcaption>Quarterly sales comparison showing 25% growth</figcaption>
</figure>
```

## Step 3: Fix Heading Structure

**Ensure logical hierarchy:**
```html
<!-- BAD: Skipped levels -->
<h1>Page Title</h1>
<h3>Section</h3>  <!-- Wrong! Should be h2 -->

<!-- GOOD: Proper hierarchy -->
<h1>Page Title</h1>
<h2>Section</h2>
<h3>Subsection</h3>
```

**One h1 per page:**
```html
<h1>Main Page Title</h1>  <!-- Only one -->
<h2>Section 1</h2>
<h2>Section 2</h2>
```

## Step 4: Fix Keyboard Accessibility

**Make custom elements focusable:**
```html
<!-- BAD: div not keyboard accessible -->
<div onclick="doThing()">Click me</div>

<!-- GOOD: Button is keyboard accessible -->
<button onclick="doThing()">Click me</button>

<!-- GOOD: If must use div, add tabindex and role -->
<div role="button" tabindex="0" onclick="doThing()" onkeydown="if(event.key==='Enter') doThing()">
  Click me
</div>
```

**Visible focus indicators:**
```css
/* BAD: Removing focus outline */
:focus {
  outline: none;
}

/* GOOD: Custom but visible focus */
:focus {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

/* GOOD: Focus-visible for mouse vs keyboard */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

**Skip links:**
```html
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>
  <header>...</header>
  <main id="main-content">...</main>
</body>
```

```css
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  padding: 8px;
  background: #000;
  color: #fff;
  z-index: 100;
}
.skip-link:focus {
  top: 0;
}
```

## Step 5: Fix Color Contrast

**Minimum ratios (WCAG AA):**
- Normal text: 4.5:1
- Large text (18pt+ or 14pt bold): 3:1
- UI components: 3:1

**Check and fix:**
```css
/* BAD: Low contrast */
.light-text {
  color: #999; /* Gray on white = ~2.8:1 */
}

/* GOOD: Sufficient contrast */
.accessible-text {
  color: #595959; /* ~7:1 on white */
}
```

**Tools to check:**
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
- Chrome DevTools → Inspect element → Shows contrast ratio

## Step 6: Fix Form Accessibility

**Labels for all inputs:**
```html
<!-- BAD: No label -->
<input type="email" placeholder="Email">

<!-- GOOD: Explicit label -->
<label for="email">Email address</label>
<input type="email" id="email" name="email">

<!-- GOOD: Implicit label -->
<label>
  Email address
  <input type="email" name="email">
</label>
```

**Error messages:**
```html
<label for="email">Email address</label>
<input type="email" id="email" aria-describedby="email-error" aria-invalid="true">
<span id="email-error" role="alert">Please enter a valid email address</span>
```

**Required fields:**
```html
<label for="name">
  Name <span aria-hidden="true">*</span>
  <span class="sr-only">(required)</span>
</label>
<input type="text" id="name" required aria-required="true">
```

## Step 7: Add Landmark Regions

**Use semantic HTML:**
```html
<header>Site header</header>           <!-- role="banner" -->
<nav>Main navigation</nav>             <!-- role="navigation" -->
<main>Primary content</main>           <!-- role="main" -->
<aside>Sidebar</aside>                 <!-- role="complementary" -->
<footer>Site footer</footer>           <!-- role="contentinfo" -->
```

**Label multiple navs:**
```html
<nav aria-label="Main">...</nav>
<nav aria-label="Footer">...</nav>
```

## Step 8: Fix Link Text

**Make links descriptive:**
```html
<!-- BAD -->
<a href="/pricing">Click here</a>
<a href="/docs">Learn more</a>

<!-- GOOD -->
<a href="/pricing">View pricing plans</a>
<a href="/docs">Read the documentation</a>
```

**Links that open new windows:**
```html
<a href="https://external.com" target="_blank" rel="noopener">
  External resource <span class="sr-only">(opens in new tab)</span>
</a>
```

## Step 9: Verify Fixes

**Re-run automated tests:**
```bash
npx axe-cli https://example.com
```

**Manual verification:**
- Tab through fixed components
- Test with screen reader
- Check color contrast
- Verify at 200% zoom

</process>

<success_criteria>
Fixes are complete when:
- [ ] All images have appropriate alt text
- [ ] Heading hierarchy is logical (h1 → h2 → h3)
- [ ] All interactive elements are keyboard accessible
- [ ] Focus indicators are visible
- [ ] Color contrast meets 4.5:1 minimum
- [ ] All form inputs have labels
- [ ] Error messages are associated with inputs
- [ ] Landmark regions are properly used
- [ ] Link text is descriptive
- [ ] Automated scan shows reduced violations
- [ ] Manual testing confirms improvements
</success_criteria>

<anti_patterns>
Avoid:
- Using ARIA when native HTML works (`<button>` not `<div role="button">`)
- Hiding focus outlines without replacement
- Using color alone to convey information
- Auto-playing audio or video
- Using "click here" or "read more" as link text
- Relying solely on placeholder text for labels
</anti_patterns>
