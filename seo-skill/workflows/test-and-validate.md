# Workflow: Test and Validate SEO and Accessibility

<required_reading>
**Read these reference files NOW:**
1. references/core/tools.md
2. references/accessibility/testing-tools.md
</required_reading>

<process>

## Step 1: Run Lighthouse Audit

**Via Chrome DevTools:**
1. Open Chrome DevTools (F12)
2. Go to Lighthouse tab
3. Select categories: Performance, Accessibility, Best Practices, SEO
4. Choose device (Mobile recommended)
5. Click "Analyze page load"

**Via CLI (recommended for automation):**
```bash
# Single page audit
npx lighthouse https://example.com \
  --output=json,html \
  --output-path=./lighthouse-report

# Mobile and desktop
npx lighthouse https://example.com --preset=desktop
npx lighthouse https://example.com --preset=perf  # Mobile (default)
```

**Interpret scores:**
- 90-100: Good
- 50-89: Needs improvement
- 0-49: Poor

## Step 2: Run Automated Accessibility Tests

**axe-core (recommended for accuracy):**
```bash
# CLI scan
npx axe-cli https://example.com

# With specific rules
npx axe-cli https://example.com --rules wcag2a,wcag2aa

# Save results
npx axe-cli https://example.com --save results.json
```

**pa11y (good for CI/CD):**
```bash
# Basic scan
npx pa11y https://example.com

# WCAG 2.1 AA standard
npx pa11y https://example.com --standard WCAG2AA

# JSON output for processing
npx pa11y https://example.com --reporter json > results.json

# Multiple pages
npx pa11y-ci --config .pa11yci.json
```

**pa11y-ci config example (.pa11yci.json):**
```json
{
  "defaults": {
    "standard": "WCAG2AA",
    "timeout": 30000
  },
  "urls": [
    "https://example.com",
    "https://example.com/about",
    "https://example.com/contact"
  ]
}
```

**WAVE (visual evaluation):**
1. Install WAVE browser extension
2. Navigate to page
3. Click WAVE icon
4. Review errors, alerts, and features

## Step 3: Test Core Web Vitals

**PageSpeed Insights (field + lab data):**
1. Go to https://pagespeed.web.dev
2. Enter URL
3. Review both Mobile and Desktop
4. Check field data (real users) vs lab data (simulated)

**Chrome DevTools Performance:**
1. Open DevTools → Performance
2. Click Record
3. Reload page
4. Stop recording
5. Check Core Web Vitals in summary

**Web Vitals Chrome Extension:**
1. Install "Web Vitals" extension
2. Navigate to pages
3. View real-time LCP, INP, CLS

**CrUX Dashboard (historical data):**
1. Go to https://developers.google.com/web/tools/chrome-user-experience-report
2. Access BigQuery or Data Studio templates
3. View historical Core Web Vitals

## Step 4: Validate Schema Markup

**Google Rich Results Test:**
1. Go to https://search.google.com/test/rich-results
2. Enter URL or paste code
3. Review detected schema types
4. Check for errors and warnings

**Schema.org Validator:**
1. Go to https://validator.schema.org
2. Paste JSON-LD code
3. Check for structural issues

**Google Search Console:**
1. Go to Search Console
2. Navigate to Enhancements section
3. Check each schema type (FAQ, Product, etc.)
4. Review any errors or warnings

## Step 5: Manual Keyboard Testing

**Tab navigation checklist:**
- [ ] Can reach all interactive elements with Tab
- [ ] Focus order matches visual order
- [ ] Focus indicator is visible (2px+ outline)
- [ ] Can activate buttons with Enter
- [ ] Can activate links with Enter
- [ ] Can toggle checkboxes with Space
- [ ] Can escape modals with Escape
- [ ] Skip link works (Tab → Enter → lands on main)

**Test procedure:**
1. Click browser address bar
2. Press Tab repeatedly
3. Note any:
   - Skipped elements
   - Trapped focus
   - Invisible focus
   - Unexpected order

## Step 6: Screen Reader Testing

**VoiceOver (macOS):**
```
Enable: Cmd + F5
Navigate: VO + Arrow keys (VO = Ctrl + Option)
Headings: VO + Cmd + H
Links: VO + Cmd + L
Disable: Cmd + F5
```

**NVDA (Windows - free):**
```
Download from nvaccess.org
Navigate: Arrow keys
Headings: H
Links: K
Landmarks: D
Stop speaking: Ctrl
```

**Testing checklist:**
- [ ] Page title announced correctly
- [ ] Headings describe content structure
- [ ] Images have meaningful alt text
- [ ] Links clearly indicate destination
- [ ] Forms announce labels and errors
- [ ] Dynamic content changes announced

## Step 7: Color Contrast Testing

**WebAIM Contrast Checker:**
1. Go to https://webaim.org/resources/contrastchecker/
2. Enter foreground and background colors
3. Check WCAG AA and AAA compliance

**Chrome DevTools:**
1. Inspect text element
2. View contrast ratio in Styles panel
3. Check the contrast indicator

**Automated checking:**
```bash
# axe includes contrast checks
npx axe-cli https://example.com

# Lighthouse includes contrast audit
npx lighthouse https://example.com --only-audits=color-contrast
```

## Step 8: Mobile Usability Testing

**Google Mobile-Friendly Test:**
1. Go to https://search.google.com/test/mobile-friendly
2. Enter URL
3. Review mobile rendering issues

**Chrome DevTools Device Mode:**
1. Open DevTools
2. Click device icon (Ctrl/Cmd + Shift + M)
3. Select device or set custom dimensions
4. Test at 320px, 375px, 768px, 1024px

**Touch target testing:**
- All buttons/links at least 48x48px
- Adequate spacing between targets
- No overlapping touch areas

## Step 9: Create Test Report

**Report template:**
```markdown
# SEO & Accessibility Test Report
Date: [Date]
URL: [URL]

## Summary Scores
| Category | Score | Status |
|----------|-------|--------|
| Lighthouse Performance | X/100 | Pass/Fail |
| Lighthouse Accessibility | X/100 | Pass/Fail |
| Lighthouse SEO | X/100 | Pass/Fail |
| axe Violations | X critical, Y serious | Pass/Fail |
| Core Web Vitals | LCP: Xs, INP: Xms, CLS: X | Pass/Fail |

## Critical Issues
1. [Issue description] - [Location] - [Fix]

## Recommendations
1. [Recommendation]

## Test Environment
- Browser: Chrome X.X
- Tools: Lighthouse X.X, axe-core X.X
- Date: [Date]
```

## Step 10: Set Up Continuous Monitoring

**GitHub Actions example:**
```yaml
name: Accessibility Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run pa11y
        run: |
          npm install -g pa11y-ci
          pa11y-ci --config .pa11yci.json
```

**Lighthouse CI:**
```yaml
name: Lighthouse CI
on: [push]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v10
        with:
          urls: |
            https://example.com
          budgetPath: ./lighthouse-budget.json
```

</process>

<success_criteria>
Testing is complete when:
- [ ] Lighthouse audit run (all categories)
- [ ] Automated accessibility scan completed (axe or pa11y)
- [ ] Core Web Vitals measured and recorded
- [ ] Schema markup validated
- [ ] Manual keyboard testing performed
- [ ] Screen reader spot-check completed
- [ ] Color contrast verified
- [ ] Mobile usability tested
- [ ] Report generated with findings
- [ ] CI/CD monitoring configured (optional)
</success_criteria>

<anti_patterns>
Avoid:
- Relying only on automated tests (miss 60-70% of issues)
- Testing only on desktop (mobile-first indexing)
- Skipping manual screen reader testing
- Testing only homepage (check key journeys)
- Ignoring field data (real users) for lab data only
- Running tests once instead of continuously
</anti_patterns>
