---
name: seo-skill
description: Expert SEO and web accessibility knowledge for optimizing sites from audit through implementation. Covers technical SEO, Core Web Vitals, WCAG 2.2/ADA compliance, schema markup, and the SEO-accessibility overlap. Full lifecycle - audit, fix, optimize, validate. Use for improving search rankings, ensuring legal compliance, or learning SEO/a11y patterns.
---

<essential_principles>

## How SEO and Accessibility Work Together

### 1. Search Engines and Screen Readers See Sites Similarly

Both Googlebot and assistive technologies rely on:
- Clean HTML structure and semantic markup
- Proper heading hierarchy (h1 → h2 → h3)
- Descriptive alt text for images
- Meaningful link text (not "click here")

Fixing accessibility often improves SEO, and vice versa.

### 2. Core Web Vitals Are Non-Negotiable

Google's ranking factors include page experience metrics:
- **LCP** (Largest Contentful Paint): < 2.5s
- **INP** (Interaction to Next Paint): < 200ms
- **CLS** (Cumulative Layout Shift): < 0.1

Sites meeting these thresholds rank 28% higher on average.

### 3. WCAG 2.2 AA Is the Legal Standard

ADA Title II requires WCAG 2.1 AA by April 2026 (gov sites). Courts use WCAG as the benchmark for ADA Title III (private businesses). Over 4,000 ADA lawsuits were filed in 2024.

Key point: **Overlay widgets do NOT provide compliance** and have been cited in lawsuits.

### 4. Automated Testing Catches Only 30-40%

Tools like Lighthouse, axe, and WAVE catch structural issues but miss context. Manual testing with screen readers is essential for true accessibility.

### 5. Content Quality Remains #1 Ranking Factor

E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness) drives rankings. Technical SEO enables discovery; quality content drives rankings.

</essential_principles>

<intake>

## What would you like to do?

**Auditing:**
1. **Audit a site** - Full SEO and accessibility audit

**Implementation:**
2. **Fix accessibility issues** - WCAG compliance fixes
3. **Optimize technical SEO** - Meta tags, crawlability, Core Web Vitals
4. **Optimize content for SEO** - Headings, keywords, structure
5. **Implement schema markup** - JSON-LD structured data
6. **Test and validate** - Run accessibility and SEO tests

**Knowledge:**
7. **Core Web Vitals** - LCP, INP, CLS optimization strategies
8. **WCAG 2.2 requirements** - New criteria and compliance
9. **SEO-accessibility overlap** - Where they benefit each other
10. **Best practices** - Recommended patterns
11. **Anti-patterns** - Common mistakes to avoid
12. **Tools reference** - Testing and analysis tools

**Wait for response before proceeding.**
</intake>

<routing>

## Task Routing

**Workflows (read and follow):**

| Response | Workflow |
|----------|----------|
| 1, "audit", "analyze", "check" | `workflows/audit-site.md` |
| 2, "accessibility", "a11y", "wcag", "ada" | `workflows/fix-accessibility-issues.md` |
| 3, "technical", "meta", "crawl", "speed", "vitals" | `workflows/optimize-technical-seo.md` |
| 4, "content", "headings", "keywords", "copy" | `workflows/optimize-content.md` |
| 5, "schema", "structured data", "json-ld", "rich results" | `workflows/implement-schema.md` |
| 6, "test", "validate", "lighthouse", "axe" | `workflows/test-and-validate.md` |

**Knowledge references (read directly):**

| Response | Reference |
|----------|-----------|
| 7, "core web vitals", "lcp", "inp", "cls", "performance" | `references/core/core-web-vitals.md` |
| 8, "wcag", "2.2", "new criteria", "focus", "target size" | `references/accessibility/wcag-22-new.md` |
| 9, "overlap", "seo accessibility", "both", "connection" | `references/core/seo-accessibility-overlap.md` |
| 10, "best practice", "recommended", "good" | `references/core/best-practices.md` |
| 11, "anti-pattern", "mistake", "avoid", "don't" | `references/core/anti-patterns.md` |
| 12, "tools", "lighthouse", "screaming frog", "axe", "wave" | `references/core/tools.md` |

</routing>

<verification_loop>

## After Every Change

**Accessibility check:**
```bash
# Quick browser check
npx axe-cli https://yoursite.com

# Or with pa11y
npx pa11y https://yoursite.com
```

**SEO check:**
```bash
# Lighthouse audit
npx lighthouse https://yoursite.com --output=json --output-path=./audit.json

# Core Web Vitals (if using Screaming Frog)
# Configure PageSpeed Insights API in Screaming Frog → Configuration → API Access
```

**Manual verification:**
- Tab through the page - is focus visible?
- Use VoiceOver/NVDA on key pages
- Check Google Search Console for indexing issues
- Validate schema at https://validator.schema.org

</verification_loop>

<reference_index>

## Domain Knowledge

### Core (Cross-Cutting)
All in `references/core/`:

- best-practices.md - Recommended patterns for SEO and accessibility
- anti-patterns.md - Common mistakes to avoid
- core-web-vitals.md - LCP, INP, CLS optimization
- seo-accessibility-overlap.md - Where SEO and a11y benefit each other
- tools.md - Testing and analysis tools

### Technical SEO
All in `references/technical-seo/`:

- meta-tags.md - Title, description, canonical, robots
- internal-linking.md - Site architecture and crawl budget
- schema-markup.md - JSON-LD structured data types
- image-optimization.md - Alt text, WebP, lazy loading
- site-architecture.md - URL structure, flat hierarchy

### Accessibility
All in `references/accessibility/`:

- wcag-overview.md - WCAG principles and conformance levels
- wcag-22-new.md - New success criteria in WCAG 2.2
- semantic-html.md - Proper HTML element usage
- aria-landmarks.md - Landmarks and ARIA roles
- testing-tools.md - axe, WAVE, pa11y, screen readers

</reference_index>

<workflows_index>

## Workflows

All in `workflows/`:

| File | Purpose |
|------|---------|
| audit-site.md | Full SEO and accessibility audit |
| fix-accessibility-issues.md | WCAG compliance fixes |
| optimize-technical-seo.md | Meta tags, crawlability, performance |
| optimize-content.md | Headings, keywords, readability |
| implement-schema.md | JSON-LD structured data |
| test-and-validate.md | Run and interpret tests |

</workflows_index>

<quick_reference>

## Key Thresholds

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| LCP | < 2.5s | 2.5s - 4s | > 4s |
| INP | < 200ms | 200ms - 500ms | > 500ms |
| CLS | < 0.1 | 0.1 - 0.25 | > 0.25 |

## WCAG Conformance Levels

| Level | Description | Legal Standard |
|-------|-------------|----------------|
| A | Minimum | Baseline only |
| AA | Mid-range | **ADA/Legal requirement** |
| AAA | Highest | Recommended for gov/healthcare |

## Essential Schema Types

| Type | Use Case |
|------|----------|
| Organization | Brand/company identity |
| LocalBusiness | Physical locations |
| Product | E-commerce items |
| Article | Blog posts/news |
| FAQPage | FAQ sections |
| BreadcrumbList | Navigation breadcrumbs |

## Quick Wins

**SEO:**
- Add unique title tags (50-60 chars)
- Write meta descriptions (150-160 chars)
- Use descriptive URLs with keywords
- Compress images to WebP format
- Add internal links between related content

**Accessibility:**
- Add alt text to all images
- Ensure color contrast ratio >= 4.5:1
- Make all interactive elements keyboard accessible
- Use semantic HTML (`<nav>`, `<main>`, `<button>`)
- Ensure visible focus indicators

</quick_reference>
