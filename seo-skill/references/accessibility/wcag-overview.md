<overview>
WCAG (Web Content Accessibility Guidelines) are the international standards for web accessibility. Understanding WCAG is essential for creating accessible websites and meeting legal requirements.
</overview>

<what_is_wcag>
**Web Content Accessibility Guidelines**

- Developed by W3C (World Wide Web Consortium)
- Current version: WCAG 2.2 (October 2023)
- International standard: ISO/IEC 40500
- The benchmark for accessibility compliance worldwide
</what_is_wcag>

<principles>
**POUR: The Four Principles**

All WCAG guidelines are organized under four principles:

<principle name="perceivable">
**1. Perceivable**

Information must be presentable in ways users can perceive.

**Key requirements:**
- Text alternatives for images (alt text)
- Captions and transcripts for media
- Content adaptable to different presentations
- Sufficient color contrast
- Text resizable without loss of functionality
</principle>

<principle name="operable">
**2. Operable**

Interface must be usable by all users.

**Key requirements:**
- All functionality available via keyboard
- No time limits (or adjustable)
- No content that could cause seizures
- Skip navigation links
- Clear page titles and headings
- Focus visible and logical
</principle>

<principle name="understandable">
**3. Understandable**

Content and interface must be understandable.

**Key requirements:**
- Language of page declared
- Consistent navigation
- Clear labels and instructions
- Error prevention and recovery
- Predictable behavior
</principle>

<principle name="robust">
**4. Robust**

Content must be robust enough for assistive technologies.

**Key requirements:**
- Valid HTML
- Name, role, value for custom components
- Status messages communicated programmatically
</principle>

</principles>

<conformance_levels>
**Conformance Levels: A, AA, AAA**

WCAG has three levels of conformance:

<level name="A">
**Level A (Minimum)**

Basic accessibility requirements. Without these, some users cannot access content at all.

**Examples:**
- 1.1.1 Non-text Content (alt text)
- 1.3.1 Info and Relationships (semantic structure)
- 2.1.1 Keyboard (all functionality via keyboard)
- 2.4.1 Bypass Blocks (skip links)
</level>

<level name="AA">
**Level AA (Mid-range) - Legal Standard**

Removes significant barriers. This is the level required by most laws and policies.

**Examples:**
- 1.4.3 Contrast (Minimum) - 4.5:1 for text
- 1.4.4 Resize Text - 200% without loss of content
- 2.4.6 Headings and Labels - descriptive
- 2.4.7 Focus Visible - keyboard focus indicator

**Legal requirements:**
- ADA Title II (US government): WCAG 2.1 AA by April 2026
- European Accessibility Act: WCAG 2.1 AA (now in effect)
- ADA Title III (private businesses): Courts typically use WCAG AA
</level>

<level name="AAA">
**Level AAA (Highest)**

Provides the most accessibility. Not required by law but recommended for government and healthcare.

**Examples:**
- 1.4.6 Contrast (Enhanced) - 7:1 for text
- 1.4.8 Visual Presentation - user control over colors, spacing
- 2.4.9 Link Purpose (Link Only) - link text alone describes purpose
- 2.4.10 Section Headings - headings organize content
</level>

</conformance_levels>

<key_success_criteria>
**Most Commonly Failed Success Criteria**

Based on accessibility audits, these are the most frequently violated:

<criterion id="1.1.1">
**1.1.1 Non-text Content (Level A)**

All images need text alternatives.

```html
<!-- Informative image -->
<img src="chart.png" alt="Bar chart showing 25% growth">

<!-- Decorative image -->
<img src="border.png" alt="">

<!-- Functional image (button, link) -->
<button><img src="search.svg" alt="Search"></button>
```
</criterion>

<criterion id="1.4.3">
**1.4.3 Contrast (Minimum) (Level AA)**

Text must have sufficient contrast with background.

- Normal text: 4.5:1 minimum
- Large text (18pt+ or 14pt bold): 3:1 minimum

```css
/* GOOD: High contrast */
color: #333333; /* on white = 12.6:1 */

/* BAD: Low contrast */
color: #999999; /* on white = 2.8:1 */
```
</criterion>

<criterion id="2.1.1">
**2.1.1 Keyboard (Level A)**

All functionality must be available via keyboard.

- Tab to navigate
- Enter/Space to activate
- Arrow keys for widgets
- Escape to close

```html
<!-- GOOD: Native elements are keyboard accessible -->
<button onclick="submit()">Submit</button>

<!-- BAD: Div not keyboard accessible -->
<div onclick="submit()">Submit</div>
```
</criterion>

<criterion id="2.4.4">
**2.4.4 Link Purpose (In Context) (Level A)**

Link text must describe destination.

```html
<!-- BAD -->
<a href="/pricing">Click here</a>
<a href="/docs">Read more</a>

<!-- GOOD -->
<a href="/pricing">View pricing plans</a>
<a href="/docs">Read the documentation</a>
```
</criterion>

<criterion id="2.4.7">
**2.4.7 Focus Visible (Level AA)**

Keyboard focus must be visible.

```css
/* BAD: Removes focus indicator */
:focus { outline: none; }

/* GOOD: Custom but visible focus */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```
</criterion>

<criterion id="4.1.2">
**4.1.2 Name, Role, Value (Level A)**

Custom components must expose their state to assistive technology.

```html
<!-- Expandable button -->
<button aria-expanded="false" aria-controls="menu">
  Menu
</button>
<ul id="menu" hidden>...</ul>

<!-- Tab -->
<button role="tab" aria-selected="true">Tab 1</button>
```
</criterion>

</key_success_criteria>

<wcag_versions>
**WCAG Version History**

| Version | Released | Status |
|---------|----------|--------|
| WCAG 1.0 | 1999 | Obsolete |
| WCAG 2.0 | 2008 | ISO standard |
| WCAG 2.1 | 2018 | Current legal standard |
| WCAG 2.2 | 2023 | Latest (adds 9 criteria) |
| WCAG 3.0 | TBD | In development |

**WCAG 2.2 is backward compatible with 2.1** - all 2.1 criteria still apply.
</wcag_versions>

<testing_approach>
**Testing for WCAG Compliance**

**Automated testing (catches 30-40%):**
- Lighthouse accessibility audit
- axe DevTools
- WAVE
- Pa11y

**Manual testing (catches the rest):**
- Keyboard navigation
- Screen reader testing
- Color contrast checking
- Zoom to 200%
- Text-only view

**User testing:**
- People with disabilities
- Various assistive technologies
- Different browsers and devices
</testing_approach>

<common_misconceptions>
**Common Misconceptions**

**"Automated tools catch everything"**
No - they catch 30-40% of issues. Manual testing is essential.

**"Accessibility overlays make sites compliant"**
No - they don't fix underlying issues and may cause more problems.

**"We don't have disabled users"**
You do - 15% of world population has a disability. Plus aging users.

**"Accessibility is expensive"**
It's cheaper when built in from the start. Retrofitting costs more.

**"WCAG AAA is the goal"**
No - AA is the legal standard. AAA is aspirational.
</common_misconceptions>

<resources>
**Official Resources:**

- WCAG 2.2: https://www.w3.org/TR/WCAG22/
- Understanding WCAG: https://www.w3.org/WAI/WCAG22/Understanding/
- Techniques: https://www.w3.org/WAI/WCAG22/Techniques/
- Quick Reference: https://www.w3.org/WAI/WCAG22/quickref/
</resources>
