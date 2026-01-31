<overview>
Recommended patterns for SEO and web accessibility that work together to improve both search visibility and user experience.
</overview>

<seo_best_practices>

<practice name="write-for-humans-optimize-for-robots">
**Principle:** Create content that satisfies user intent first, then optimize for search engines.

**Implementation:**
- Answer the user's question thoroughly
- Use natural language (avoid keyword stuffing)
- Structure content with clear headings
- Include relevant keywords where they fit naturally

**Why it works:** Google's algorithms increasingly reward content that satisfies user intent. E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness) is the guiding framework.
</practice>

<practice name="mobile-first-design">
**Principle:** Design and optimize for mobile before desktop.

**Implementation:**
- Use responsive design (not separate mobile site)
- Ensure touch targets are 48x48px minimum
- Test at 320px viewport width
- Prioritize above-fold content loading

**Why it works:** Google uses mobile-first indexing. Your mobile experience determines your rankings.
</practice>

<practice name="fast-page-loads">
**Principle:** Speed is a ranking factor and user experience factor.

**Targets:**
- LCP (Largest Contentful Paint): < 2.5 seconds
- INP (Interaction to Next Paint): < 200 milliseconds
- CLS (Cumulative Layout Shift): < 0.1

**Key optimizations:**
- Compress images (WebP format)
- Minimize CSS/JS
- Use CDN
- Enable caching
- Lazy load below-fold images
</practice>

<practice name="clear-site-structure">
**Principle:** Organize content so both users and crawlers can navigate easily.

**Implementation:**
- Flat hierarchy (key pages within 3 clicks)
- Logical URL structure (/category/subcategory/page)
- Internal links between related content
- XML sitemap for all important pages
- Breadcrumb navigation

**Why it works:** Helps crawlers discover content and distributes page authority through internal links.
</practice>

<practice name="unique-descriptive-meta">
**Principle:** Each page should have unique, descriptive meta tags.

**Title tag (50-60 characters):**
- Include primary keyword
- Front-load important words
- Include brand name (end)

**Meta description (150-160 characters):**
- Summarize page content
- Include call-to-action
- Include primary keyword naturally

**Example:**
```html
<title>Technical SEO Checklist 2025 | Your Brand</title>
<meta name="description" content="Complete technical SEO checklist for 2025. Learn how to audit crawlability, Core Web Vitals, and indexing issues. Free downloadable template included.">
```
</practice>

<practice name="structured-data-markup">
**Principle:** Help search engines understand your content with schema.org markup.

**Priority schema types:**
1. Organization (all sites)
2. BreadcrumbList (navigation)
3. Article (blog posts)
4. Product (e-commerce)
5. FAQPage (FAQ sections)
6. LocalBusiness (physical locations)

**Benefits:**
- Rich snippets in search results
- 20-30% higher click-through rates
- Better AI/answer engine visibility
</practice>

<practice name="quality-backlinks">
**Principle:** Earn links from authoritative, relevant sites.

**Good link building:**
- Create linkable content (guides, research, tools)
- Guest posting on relevant sites
- Digital PR and outreach
- Broken link building

**Avoid:**
- Buying links
- Link farms
- Excessive reciprocal links
- Irrelevant directory submissions
</practice>

</seo_best_practices>

<accessibility_best_practices>

<practice name="semantic-html-first">
**Principle:** Use native HTML elements before ARIA.

**Correct usage:**
```html
<!-- Use native elements -->
<button>Click me</button>           <!-- Not <div role="button"> -->
<nav>Navigation</nav>               <!-- Not <div role="navigation"> -->
<main>Main content</main>           <!-- Not <div role="main"> -->

<!-- Only use ARIA when no native equivalent exists -->
<div role="tablist">...</div>       <!-- No native tab element -->
```

**Why it works:** Native HTML has built-in accessibility. ARIA only communicates semantics, doesn't add behavior.
</practice>

<practice name="keyboard-accessible">
**Principle:** All functionality must be available via keyboard.

**Requirements:**
- All interactive elements focusable
- Logical tab order
- Visible focus indicators
- No keyboard traps
- Escape closes modals

**Implementation:**
```css
/* Visible focus */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```
</practice>

<practice name="sufficient-color-contrast">
**Principle:** Text must be readable against its background.

**WCAG AA requirements:**
- Normal text: 4.5:1 contrast ratio
- Large text (18pt+ or 14pt bold): 3:1
- UI components: 3:1

**Tools:**
- WebAIM Contrast Checker
- Chrome DevTools (inspect element)
- Lighthouse accessibility audit
</practice>

<practice name="descriptive-alt-text">
**Principle:** All informative images need alt text.

**Guidelines:**
```html
<!-- Informative: describe content -->
<img src="chart.png" alt="Bar chart showing 25% revenue growth Q4 2024">

<!-- Functional: describe action -->
<img src="search.svg" alt="Search">

<!-- Decorative: empty alt -->
<img src="border.png" alt="">
```

**Writing good alt text:**
- Describe the image's purpose, not just appearance
- Keep under 125 characters
- Don't start with "Image of" or "Picture of"
- Include relevant keywords naturally
</practice>

<practice name="form-accessibility">
**Principle:** Forms must be usable by everyone.

**Requirements:**
```html
<!-- Always associate labels -->
<label for="email">Email address</label>
<input type="email" id="email" name="email" required aria-required="true">

<!-- Group related fields -->
<fieldset>
  <legend>Shipping Address</legend>
  <!-- Address fields -->
</fieldset>

<!-- Associate error messages -->
<input id="email" aria-describedby="email-error" aria-invalid="true">
<span id="email-error" role="alert">Please enter a valid email</span>
```
</practice>

<practice name="meaningful-link-text">
**Principle:** Link text should describe the destination.

**Examples:**
```html
<!-- BAD -->
<a href="/pricing">Click here</a>
<a href="/docs">Read more</a>
<a href="/about">Learn more</a>

<!-- GOOD -->
<a href="/pricing">View pricing plans</a>
<a href="/docs">Read the API documentation</a>
<a href="/about">Learn about our team</a>
```

**Why it works:** Screen reader users often navigate by links. "Click here" out of context is meaningless.
</practice>

</accessibility_best_practices>

<combined_practices>

<practice name="heading-hierarchy">
**Benefits both SEO and accessibility.**

**SEO benefit:** Search engines use headings to understand content structure and topic relevance.

**Accessibility benefit:** Screen reader users navigate by headings. Proper hierarchy enables efficient navigation.

**Implementation:**
- One H1 per page (main topic)
- H2 for major sections
- H3 for subsections
- Never skip levels (H1 → H3)
- Make headings descriptive
</practice>

<practice name="descriptive-urls">
**Benefits both SEO and accessibility.**

**SEO benefit:** Keywords in URLs signal relevance. Clean URLs are more shareable.

**Accessibility benefit:** Screen readers announce full URLs. Descriptive URLs help users understand destination.

**Implementation:**
```
Good: /blog/technical-seo-guide
Bad:  /blog/post?id=12345
Bad:  /blog/2024/01/15/a-really-long-title-that-goes-on-forever
```
</practice>

<practice name="responsive-images">
**Benefits both SEO and accessibility.**

**SEO benefit:** Faster loading improves Core Web Vitals. Proper sizing prevents CLS.

**Accessibility benefit:** Images don't break layout. Content remains readable on all devices.

**Implementation:**
```html
<img src="photo.jpg"
     srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
     sizes="(max-width: 600px) 400px, (max-width: 1200px) 800px, 1200px"
     alt="Descriptive alt text"
     width="800"
     height="600"
     loading="lazy">
```
</practice>

</combined_practices>

<checklist>
**SEO Essentials:**
- [ ] Unique title tags on all pages
- [ ] Meta descriptions on all pages
- [ ] Clean URL structure
- [ ] XML sitemap submitted
- [ ] robots.txt configured
- [ ] Core Web Vitals passing
- [ ] Mobile-friendly design
- [ ] Schema markup implemented
- [ ] Internal linking strategy

**Accessibility Essentials:**
- [ ] Semantic HTML structure
- [ ] All images have alt text
- [ ] Keyboard accessible
- [ ] Visible focus indicators
- [ ] Sufficient color contrast
- [ ] Forms properly labeled
- [ ] Descriptive link text
- [ ] No auto-playing media
- [ ] Skip link to main content
</checklist>
