<overview>
Essential tools for SEO and accessibility auditing, testing, and monitoring. Includes both free and paid options.
</overview>

<seo_tools>

<tool name="google-search-console">
**Google Search Console (Free)**

**URL:** https://search.google.com/search-console

**What it does:**
- Shows how Google sees your site
- Reports indexing issues
- Shows search performance (queries, clicks, impressions)
- Core Web Vitals report (field data)
- Mobile usability issues
- Manual actions (penalties)

**Key features:**
- URL Inspection: Check individual page indexing
- Coverage: Find indexing errors
- Core Web Vitals: Real user performance data
- Links: Internal and external link reports

**When to use:** Daily/weekly monitoring, debugging indexing issues.
</tool>

<tool name="lighthouse">
**Google Lighthouse (Free)**

**Access:** Chrome DevTools → Lighthouse tab, or CLI

**What it does:**
- Audits Performance, Accessibility, Best Practices, SEO
- Lab-based testing (simulated conditions)
- Provides specific recommendations

**CLI usage:**
```bash
# Install
npm install -g lighthouse

# Run audit
npx lighthouse https://example.com --output=html --output-path=./report.html

# Specific categories
npx lighthouse https://example.com --only-categories=accessibility,seo
```

**Scores:**
- 90-100: Good
- 50-89: Needs improvement
- 0-49: Poor

**When to use:** During development, before deployments, regular audits.
</tool>

<tool name="pagespeed-insights">
**PageSpeed Insights (Free)**

**URL:** https://pagespeed.web.dev

**What it does:**
- Combines field data (real users) and lab data (Lighthouse)
- Shows Core Web Vitals status
- Provides optimization recommendations

**Key features:**
- Field data from Chrome User Experience Report (if available)
- Lab data from Lighthouse
- Specific fix recommendations
- Mobile and desktop views

**When to use:** Quick performance checks, sharing reports with stakeholders.
</tool>

<tool name="screaming-frog">
**Screaming Frog SEO Spider**

**URL:** https://www.screamingfrog.co.uk/seo-spider/

**Pricing:** Free (500 URLs), £199/year (unlimited)

**What it does:**
- Crawls websites like a search engine
- Identifies 300+ SEO issues
- Exports data for analysis

**Key features:**
- Find broken links (404s)
- Audit redirects
- Analyze page titles and meta descriptions
- Check heading structure
- Generate XML sitemaps
- Integrate with Google Analytics, Search Console
- PageSpeed Insights API integration for CWV

**When to use:** Comprehensive technical audits, large site analysis.
</tool>

<tool name="ahrefs-semrush">
**Ahrefs / SEMrush / Moz (Paid)**

**What they do:**
- Keyword research
- Competitor analysis
- Backlink analysis
- Rank tracking
- Site audits

**When to use:** Keyword strategy, competitive research, link building campaigns.

**Pricing:** $99-$499+/month depending on plan
</tool>

</seo_tools>

<accessibility_tools>

<tool name="axe">
**axe-core / axe DevTools**

**URL:** https://www.deque.com/axe/

**Pricing:** Free (browser extension, CLI), Paid (DevTools Pro)

**What it does:**
- Automated accessibility testing
- Zero false positives design
- Detects ~57% of accessibility issues

**CLI usage:**
```bash
# Install
npm install -g @axe-core/cli

# Run scan
npx axe-cli https://example.com

# Save results
npx axe-cli https://example.com --save results.json

# Specific rules
npx axe-cli https://example.com --rules wcag2a,wcag2aa
```

**Browser extension:**
1. Install "axe DevTools" extension
2. Open DevTools → axe DevTools tab
3. Click "Scan ALL of my page"

**When to use:** Development testing, CI/CD integration, quick audits.
</tool>

<tool name="wave">
**WAVE (Web Accessibility Evaluation Tool)**

**URL:** https://wave.webaim.org/

**Pricing:** Free (browser extension, online tool)

**What it does:**
- Visual accessibility evaluation
- Shows errors directly on page
- Highlights structure and features

**Key features:**
- Errors (must fix)
- Alerts (potential issues)
- Features (things done right)
- Structural elements visualization
- Contrast checker

**When to use:** Quick visual checks, client presentations, learning accessibility.
</tool>

<tool name="pa11y">
**Pa11y**

**URL:** https://pa11y.org/

**Pricing:** Free (open source)

**What it does:**
- Command-line accessibility testing
- CI/CD integration
- Dashboard for multiple pages

**CLI usage:**
```bash
# Install
npm install -g pa11y

# Run scan
npx pa11y https://example.com

# WCAG 2.1 AA standard
npx pa11y https://example.com --standard WCAG2AA

# JSON output
npx pa11y https://example.com --reporter json > results.json
```

**CI/CD configuration (.pa11yci.json):**
```json
{
  "defaults": {
    "standard": "WCAG2AA",
    "timeout": 30000
  },
  "urls": [
    "https://example.com",
    "https://example.com/about"
  ]
}
```

**When to use:** CI/CD pipelines, automated testing, batch scanning.
</tool>

<tool name="screen-readers">
**Screen Readers**

**VoiceOver (macOS/iOS) - Built-in, Free:**
```
Enable: Cmd + F5
Navigate: Ctrl + Option + Arrow keys
Headings: Ctrl + Option + Cmd + H
Links: Ctrl + Option + Cmd + L
```

**NVDA (Windows) - Free:**
- Download: https://www.nvaccess.org/
```
Navigate: Arrow keys
Headings: H
Links: K
Landmarks: D
Stop speaking: Ctrl
```

**JAWS (Windows) - Paid:**
- Industry standard for enterprise testing
- ~$1,000/year

**When to use:** Manual testing, verifying automated results, user journey testing.
</tool>

<tool name="contrast-checkers">
**Color Contrast Tools**

**WebAIM Contrast Checker (Free):**
- URL: https://webaim.org/resources/contrastchecker/
- Enter foreground and background colors
- Shows WCAG AA and AAA compliance

**Chrome DevTools:**
- Inspect text element
- View contrast ratio in Styles panel
- Suggests accessible alternatives

**Colour Contrast Analyser (Free):**
- Desktop app for Windows/Mac
- Color picker for any screen element
- URL: https://www.tpgi.com/color-contrast-checker/
</tool>

</accessibility_tools>

<combined_tools>

<tool name="chrome-devtools">
**Chrome DevTools (Free)**

**Accessibility features:**
- Accessibility tree viewer
- Contrast ratio display
- Lighthouse audits
- Focus order visualization

**Performance features:**
- Performance panel
- Core Web Vitals overlay
- Network throttling
- Coverage analysis

**Access:** F12 or Cmd+Option+I
</tool>

<tool name="webhint">
**webhint (Free)**

**URL:** https://webhint.io/

**What it does:**
- Checks accessibility, performance, security, PWA
- Browser extension and CLI

**CLI usage:**
```bash
npm install -g hint
hint https://example.com
```
</tool>

</combined_tools>

<validation_tools>

<tool name="schema-validators">
**Schema Markup Validators**

**Google Rich Results Test:**
- URL: https://search.google.com/test/rich-results
- Shows if schema qualifies for rich results
- Identifies errors and warnings

**Schema.org Validator:**
- URL: https://validator.schema.org/
- Validates JSON-LD structure
- Shows parsed schema
</tool>

<tool name="html-validators">
**HTML Validators**

**W3C Markup Validator:**
- URL: https://validator.w3.org/
- Checks HTML syntax
- Identifies structural issues

**Nu Html Checker:**
- URL: https://validator.w3.org/nu/
- Modern HTML5 validation
- Checks for accessibility issues
</tool>

</validation_tools>

<tool_selection_guide>

**For quick checks:**
| Task | Tool |
|------|------|
| Accessibility scan | WAVE extension |
| Performance check | PageSpeed Insights |
| Schema validation | Rich Results Test |
| Contrast check | Chrome DevTools |

**For development:**
| Task | Tool |
|------|------|
| Accessibility testing | axe DevTools |
| Performance profiling | Chrome DevTools |
| HTML validation | Nu Html Checker |

**For CI/CD:**
| Task | Tool |
|------|------|
| Accessibility | Pa11y-ci or axe-core |
| Performance | Lighthouse CI |
| Scheduled audits | Screaming Frog |

**For comprehensive audits:**
| Task | Tool |
|------|------|
| Technical SEO | Screaming Frog |
| Accessibility | axe + manual testing |
| Performance | Lighthouse + Search Console |
</tool_selection_guide>

<recommended_workflow>
**Regular monitoring workflow:**

1. **Weekly:**
   - Check Search Console for new issues
   - Review Core Web Vitals trends
   - Run WAVE on new pages

2. **Monthly:**
   - Full Lighthouse audit on key pages
   - Screaming Frog crawl for technical issues
   - Pa11y batch scan

3. **Before deployments:**
   - axe scan on changed pages
   - Lighthouse CI in pipeline
   - Manual keyboard + screen reader test

4. **Quarterly:**
   - Comprehensive accessibility audit
   - Full technical SEO audit
   - Competitive analysis (Ahrefs/SEMrush)
</recommended_workflow>
