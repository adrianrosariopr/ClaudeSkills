<overview>
Accessibility testing requires a combination of automated tools and manual testing. Automated tools catch 30-40% of issues; manual testing catches the rest.
</overview>

<testing_strategy>
**Comprehensive Testing Strategy**

1. **Automated scanning** (30-40% of issues)
2. **Keyboard testing** (critical for operability)
3. **Screen reader testing** (essential for perceivability)
4. **Visual testing** (contrast, zoom, layout)
5. **User testing** (real users with disabilities)
</testing_strategy>

<automated_tools>

<tool name="axe">
**axe-core / axe DevTools**

**Best for:** Development integration, CI/CD, accurate results

**Zero false positives** - only reports definite violations.

**Browser extension:**
1. Install "axe DevTools" from Chrome/Firefox store
2. Open DevTools → axe DevTools tab
3. Click "Scan ALL of my page"
4. Review issues by severity

**CLI:**
```bash
npm install -g @axe-core/cli

# Scan a URL
npx axe-cli https://example.com

# Save results as JSON
npx axe-cli https://example.com --save results.json

# Specific standards
npx axe-cli https://example.com --rules wcag2a,wcag2aa
```

**Integration with testing frameworks:**
```javascript
// Cypress
import 'cypress-axe';

cy.visit('/');
cy.injectAxe();
cy.checkA11y();

// Playwright
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('accessibility', async ({ page }) => {
  await page.goto('/');
  const results = await new AxeBuilder({ page }).analyze();
  expect(results.violations).toEqual([]);
});
```
</tool>

<tool name="lighthouse">
**Lighthouse**

**Best for:** Quick audits, performance + accessibility together

**Browser:**
1. Open Chrome DevTools → Lighthouse tab
2. Select "Accessibility" category
3. Click "Analyze page load"

**CLI:**
```bash
npm install -g lighthouse

# Run audit
npx lighthouse https://example.com --only-categories=accessibility

# Output as HTML report
npx lighthouse https://example.com --output=html --output-path=./report.html

# Specific audits only
npx lighthouse https://example.com --only-audits=color-contrast,image-alt
```

**Scores:** 0-100 (90+ is good)
</tool>

<tool name="pa11y">
**Pa11y**

**Best for:** CI/CD integration, batch scanning

**CLI:**
```bash
npm install -g pa11y

# Basic scan
npx pa11y https://example.com

# WCAG 2.1 AA
npx pa11y https://example.com --standard WCAG2AA

# JSON output
npx pa11y https://example.com --reporter json > results.json

# Ignore specific issues
npx pa11y https://example.com --ignore "WCAG2AA.Principle1.Guideline1_4.1_4_3.G18"
```

**CI/CD configuration (.pa11yci.json):**
```json
{
  "defaults": {
    "standard": "WCAG2AA",
    "timeout": 30000,
    "wait": 1000
  },
  "urls": [
    "https://example.com",
    "https://example.com/about",
    "https://example.com/contact"
  ]
}
```

```bash
npm install -g pa11y-ci
pa11y-ci --config .pa11yci.json
```
</tool>

<tool name="wave">
**WAVE (Web Accessibility Evaluation Tool)**

**Best for:** Visual evaluation, learning, client presentations

**Browser extension:**
1. Install WAVE from webaim.org
2. Navigate to page
3. Click WAVE icon
4. Review visual annotations

**Categories:**
- **Errors:** Must fix (red)
- **Alerts:** Potential issues (yellow)
- **Features:** Things done right (green)
- **Structural:** Page structure
- **Contrast:** Color contrast issues

**Online tool:** https://wave.webaim.org/
</tool>

</automated_tools>

<keyboard_testing>
**Keyboard Testing**

**Essential for WCAG 2.1.1 Keyboard**

**Basic navigation:**
| Key | Action |
|-----|--------|
| Tab | Move to next focusable element |
| Shift+Tab | Move to previous element |
| Enter | Activate links and buttons |
| Space | Activate buttons, toggle checkboxes |
| Arrow keys | Navigate within widgets |
| Escape | Close modals/dialogs |

**Testing checklist:**
1. Click in address bar
2. Press Tab repeatedly through the page
3. Verify:
   - [ ] All interactive elements receive focus
   - [ ] Focus order matches visual order
   - [ ] Focus indicator is visible
   - [ ] No focus traps (can always Tab away)
   - [ ] Can activate all buttons/links
   - [ ] Can escape modals
   - [ ] Skip link works

**Focus visibility test:**
```css
/* Temporarily make focus very obvious */
*:focus {
  outline: 5px solid red !important;
}
```
</keyboard_testing>

<screen_reader_testing>
**Screen Reader Testing**

<reader name="voiceover">
**VoiceOver (macOS) - Built-in, Free**

**Enable:** Cmd + F5

**Basic commands:**
| Keys | Action |
|------|--------|
| Ctrl + Option + → | Next element |
| Ctrl + Option + ← | Previous element |
| Ctrl + Option + Space | Activate |
| Ctrl + Option + U | Rotor (navigate by type) |
| Ctrl + Option + Cmd + H | Next heading |
| Ctrl + Option + Cmd + L | Next link |

**Testing flow:**
1. Enable VoiceOver
2. Navigate with Ctrl+Option+arrows
3. Listen for:
   - Page title announced
   - Headings describe structure
   - Images have alt text
   - Links/buttons are labeled
   - Forms are labeled
</reader>

<reader name="nvda">
**NVDA (Windows) - Free**

**Download:** https://www.nvaccess.org/

**Basic commands:**
| Key | Action |
|-----|--------|
| ↓ | Next line |
| H | Next heading |
| K | Next link |
| B | Next button |
| F | Next form field |
| D | Next landmark |
| Ctrl | Stop speaking |
| Insert + F7 | Elements list |

**Browse vs Focus mode:**
- Browse mode: Read page content
- Focus mode: Interact with forms
- NVDA switches automatically
</reader>

<reader name="jaws">
**JAWS (Windows) - Paid**

Industry standard for enterprise testing. Similar commands to NVDA.

**Cost:** ~$1,000/year
</reader>

</screen_reader_testing>

<color_contrast>
**Color Contrast Testing**

**WCAG requirements:**
- Normal text: 4.5:1 minimum
- Large text (18pt+ or 14pt bold): 3:1 minimum
- UI components: 3:1 minimum

**Tools:**

**WebAIM Contrast Checker:**
- URL: https://webaim.org/resources/contrastchecker/
- Enter hex colors
- See AA and AAA compliance

**Chrome DevTools:**
1. Inspect element
2. View contrast ratio in Styles panel
3. Click ratio to see suggestions

**Colour Contrast Analyser (CCA):**
- Desktop app
- Color picker for any screen element
- Download: https://www.tpgi.com/color-contrast-checker/
</color_contrast>

<visual_testing>
**Visual Accessibility Testing**

**Zoom test (WCAG 1.4.4):**
1. Set browser zoom to 200%
2. Verify:
   - All content visible
   - No horizontal scrolling
   - Text doesn't overlap
   - Layout remains usable

**Text spacing test (WCAG 1.4.12):**
Use a bookmarklet to apply extreme text spacing:
- Line height: 1.5x font size
- Paragraph spacing: 2x font size
- Letter spacing: 0.12x font size
- Word spacing: 0.16x font size

**Reduced motion test (WCAG 2.3.3):**
1. Enable "Reduce motion" in OS settings
2. Verify animations are reduced/removed
3. Check `prefers-reduced-motion` media query is respected

**Dark mode test:**
1. Enable OS dark mode
2. Verify content remains readable
3. Check contrast in both modes
</visual_testing>

<testing_workflow>
**Recommended Testing Workflow**

**During development:**
1. Run axe DevTools on new components
2. Keyboard test new interactions
3. Check color contrast as you style

**Before deployment:**
```bash
# Automated scan
npx axe-cli https://staging.example.com
npx lighthouse https://staging.example.com --only-categories=accessibility

# Manual testing
# - Keyboard navigation
# - Screen reader spot check
# - Zoom to 200%
```

**Regular audits (monthly/quarterly):**
1. Full Lighthouse audit
2. Screaming Frog crawl (if large site)
3. Screen reader testing of key user journeys
4. User testing with people with disabilities

**CI/CD integration:**
```yaml
# GitHub Actions example
name: Accessibility
on: [push, pull_request]

jobs:
  a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install
        run: npm ci
      - name: Build
        run: npm run build
      - name: Accessibility test
        run: npx pa11y-ci --config .pa11yci.json
```
</testing_workflow>

<checklist>
**Accessibility Testing Checklist:**

**Automated:**
- [ ] axe scan: 0 critical/serious issues
- [ ] Lighthouse accessibility: 90+ score
- [ ] Pa11y WCAG 2.1 AA: passing

**Keyboard:**
- [ ] All interactive elements focusable
- [ ] Focus indicator visible
- [ ] Logical focus order
- [ ] No keyboard traps
- [ ] Skip link works

**Screen reader:**
- [ ] Page title announced
- [ ] Headings describe structure
- [ ] Images have alt text
- [ ] Links/buttons labeled
- [ ] Forms accessible

**Visual:**
- [ ] Color contrast 4.5:1 minimum
- [ ] Content readable at 200% zoom
- [ ] No horizontal scroll at 320px
- [ ] Reduced motion respected
</checklist>
