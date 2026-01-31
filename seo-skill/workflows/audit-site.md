# Workflow: Audit Site for SEO and Accessibility

<required_reading>
**Read these reference files NOW:**
1. references/core/tools.md
2. references/core/core-web-vitals.md
3. references/accessibility/wcag-overview.md
</required_reading>

<process>

## Step 1: Gather Basic Information

Ask user for:
- Site URL (or local development URL)
- Primary target audience/market
- Any known issues or areas of concern
- Framework/CMS being used (if known)

## Step 2: Run Automated Accessibility Tests

**Option A: Browser-based (quick)**
```bash
# Using axe-cli
npx axe-cli https://example.com --save audit-results.json

# Or using pa11y
npx pa11y https://example.com --reporter json > pa11y-results.json
```

**Option B: Lighthouse (comprehensive)**
```bash
npx lighthouse https://example.com \
  --output=json \
  --output-path=./lighthouse-audit.json \
  --only-categories=accessibility,performance,seo,best-practices
```

**Interpret results:**
- Critical/Serious = Must fix (legal liability)
- Moderate = Should fix (user experience)
- Minor = Nice to fix (polish)

## Step 3: Run Technical SEO Checks

**Check indexability:**
```bash
# Check robots.txt
curl https://example.com/robots.txt

# Check for sitemap
curl https://example.com/sitemap.xml
```

**Check meta tags (inspect page source or use):**
- Title tag present and unique?
- Meta description present?
- Canonical URL set correctly?
- Open Graph tags for social sharing?

**Check Core Web Vitals:**
- Use PageSpeed Insights: https://pagespeed.web.dev/
- Or Chrome DevTools → Performance → Core Web Vitals

## Step 4: Manual Accessibility Testing

**Keyboard navigation:**
1. Press Tab repeatedly through the page
2. Can you reach all interactive elements?
3. Is focus indicator visible?
4. Can you activate buttons/links with Enter/Space?
5. Can you escape modal dialogs with Escape?

**Screen reader check (pick one):**
- macOS: Enable VoiceOver (Cmd+F5)
- Windows: Use NVDA (free) or JAWS
- Test: Can you understand page structure?

**Visual checks:**
- Color contrast (use browser devtools or WebAIM contrast checker)
- Text resizes properly at 200%?
- No horizontal scroll at 320px width?

## Step 5: Content and Structure Analysis

**Heading hierarchy:**
- Is there exactly one `<h1>`?
- Do headings follow logical order (h1 → h2 → h3)?
- Do headings describe content sections?

**Link text:**
- Are links descriptive (not "click here")?
- Do links clearly indicate destination?

**Images:**
- Do all images have alt text?
- Is alt text descriptive and useful?
- Are decorative images marked with `alt=""`?

## Step 6: Schema Markup Check

**Validate existing schema:**
- Use Google Rich Results Test: https://search.google.com/test/rich-results
- Or Schema Validator: https://validator.schema.org

**Check for opportunities:**
- Organization/LocalBusiness schema present?
- Article schema on blog posts?
- Product schema on product pages?
- FAQPage schema if FAQ sections exist?

## Step 7: Compile Audit Report

Organize findings by priority:

**Critical (Fix Immediately):**
- WCAG Level A violations
- Missing alt text on informative images
- Keyboard traps
- Missing page titles
- Blocked indexing issues

**High Priority:**
- WCAG Level AA violations
- Core Web Vitals failures
- Missing meta descriptions
- Poor heading structure

**Medium Priority:**
- Missing schema markup
- Internal linking gaps
- Image optimization opportunities

**Low Priority:**
- WCAG Level AAA recommendations
- Minor SEO enhancements
- Polish items

</process>

<audit_template>

## Audit Report Template

```markdown
# SEO & Accessibility Audit: [Site Name]
Date: [Date]
URL: [URL]

## Executive Summary
[2-3 sentence overview of site health]

## Scores
- Lighthouse Accessibility: X/100
- Lighthouse Performance: X/100
- Lighthouse SEO: X/100
- Core Web Vitals: Pass/Fail

## Critical Issues (Must Fix)
1. [Issue]: [Location] - [How to fix]
2. ...

## High Priority Issues
1. [Issue]: [Location] - [How to fix]
2. ...

## Medium Priority Issues
1. ...

## Recommendations
1. ...

## Next Steps
1. ...
```

</audit_template>

<success_criteria>
Audit is complete when:
- [ ] Automated accessibility scan completed
- [ ] Lighthouse audit run (a11y, performance, SEO, best practices)
- [ ] Manual keyboard testing performed
- [ ] Screen reader spot-check done
- [ ] Heading structure analyzed
- [ ] Schema markup checked
- [ ] Core Web Vitals measured
- [ ] Findings organized by priority
- [ ] Report delivered with actionable recommendations
</success_criteria>

<anti_patterns>
Avoid:
- Running only automated tests (they catch 30-40%)
- Ignoring mobile experience (Google uses mobile-first indexing)
- Testing only homepage (check key user journeys)
- Reporting issues without fix recommendations
- Overwhelming client with too many minor issues
</anti_patterns>
