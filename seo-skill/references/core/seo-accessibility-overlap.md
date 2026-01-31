<overview>
SEO and web accessibility share many common practices. Improvements in one area often benefit the other. This reference documents where they overlap and how to maximize both.
</overview>

<core_insight>
"Googlebot and screen readers experience your site in strikingly similar ways. Both rely on clean HTML structure, proper heading hierarchy, and semantic tags to understand your content."

Google doesn't directly use WCAG compliance as a ranking signal, but the practices that make sites accessible are the same ones Google rewards.
</core_insight>

<overlap_areas>

<area name="alt-text">
**Image Alternative Text**

**Accessibility benefit (WCAG 1.1.1):**
Screen reader users need text descriptions to understand images. Without alt text, they hear "image" or the filename.

**SEO benefit:**
Alt text is the primary signal for Google Image Search. It helps search engines understand image content and when to display images in results.

**Best practice:**
```html
<!-- Informative images: describe content -->
<img src="chart.png" alt="Bar chart showing 40% revenue growth in Q4">

<!-- Functional images: describe action -->
<img src="search-icon.svg" alt="Search">

<!-- Decorative images: empty alt -->
<img src="decorative-line.png" alt="">
```

**Tips:**
- Describe the image's purpose, not just appearance
- Include relevant keywords naturally (not stuffed)
- Keep under 125 characters
- Don't start with "Image of" or "Picture of"
</area>

<area name="heading-structure">
**Heading Hierarchy**

**Accessibility benefit (WCAG 1.3.1, 2.4.6):**
Screen reader users navigate by headings. They can jump from heading to heading to understand page structure and find content quickly. Skipped levels (h1 → h3) break this navigation.

**SEO benefit:**
Search engines use headings to understand content organization. The h1 typically indicates the main topic. Heading structure helps Google identify key themes and relevance.

**Best practice:**
```html
<h1>Main Page Topic (include primary keyword)</h1>
  <h2>First Major Section</h2>
    <h3>Subsection</h3>
    <h3>Subsection</h3>
  <h2>Second Major Section</h2>
    <h3>Subsection</h3>
```

**Rules:**
- One h1 per page
- Don't skip levels (h1 → h2 → h3)
- Make headings descriptive (not just "Introduction")
- Include keywords naturally in headings
</area>

<area name="link-text">
**Descriptive Link Text**

**Accessibility benefit (WCAG 2.4.4):**
Screen reader users often navigate by links (pressing 'K' in NVDA). Hearing "click here, click here, click here" is useless. Links must make sense out of context.

**SEO benefit:**
Anchor text signals relevance. "View our pricing plans" tells Google the linked page is about pricing. "Click here" provides no information.

**Best practice:**
```html
<!-- BAD -->
<a href="/pricing">Click here</a>
<a href="/docs">Read more</a>
<a href="/contact">Here</a>

<!-- GOOD -->
<a href="/pricing">View pricing plans</a>
<a href="/docs">Read the API documentation</a>
<a href="/contact">Contact our support team</a>
```
</area>

<area name="semantic-html">
**Semantic HTML Structure**

**Accessibility benefit:**
Semantic elements (`<nav>`, `<main>`, `<article>`) create landmarks that screen reader users can navigate between. They also convey meaning (this is navigation, this is the main content).

**SEO benefit:**
Search engines prefer well-structured, semantic HTML. It helps them understand content roles and relationships.

**Best practice:**
```html
<header>
  <nav aria-label="Main">Site navigation</nav>
</header>

<main>
  <article>
    <h1>Article title</h1>
    <p>Content...</p>
  </article>
</main>

<aside>Related content</aside>

<footer>Site footer</footer>
```
</area>

<area name="page-titles">
**Unique Page Titles**

**Accessibility benefit (WCAG 2.4.2):**
Page titles are the first thing screen readers announce. They help users understand where they are and distinguish between browser tabs.

**SEO benefit:**
Title tags are a major ranking factor. They appear in search results and influence click-through rates.

**Best practice:**
```html
<title>Primary Keyword - Secondary Info | Brand Name</title>

<!-- Examples -->
<title>Technical SEO Guide 2025 | Acme Digital</title>
<title>Contact Us - Get Support | Acme Digital</title>
```

**Guidelines:**
- Keep under 60 characters
- Include primary keyword
- Make each page's title unique
- Front-load important words
</area>

<area name="video-transcripts">
**Video and Audio Transcripts**

**Accessibility benefit (WCAG 1.2.1, 1.2.2):**
Deaf and hard-of-hearing users need transcripts or captions to access audio content. Transcripts also help users who can't play audio (quiet environments).

**SEO benefit:**
Video and audio content isn't crawlable. Transcripts make the content indexable, allowing pages to rank for keywords mentioned in the media.

**Best practice:**
```html
<video controls>
  <source src="tutorial.mp4" type="video/mp4">
  <track kind="captions" src="captions.vtt" srclang="en" label="English">
</video>

<!-- Full transcript below -->
<details>
  <summary>View transcript</summary>
  <p>Full text transcript of the video...</p>
</details>
```
</area>

<area name="mobile-responsive">
**Mobile Responsiveness**

**Accessibility benefit:**
Users with motor disabilities may use mobile devices because touch interfaces are easier than mouse/keyboard. Responsive design ensures content is accessible on all devices.

**SEO benefit:**
Google uses mobile-first indexing. Your mobile experience determines your rankings. Poor mobile UX hurts rankings.

**Best practice:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

```css
/* Touch targets minimum 48x48px */
button, a {
  min-height: 48px;
  min-width: 48px;
}

/* Readable text without zooming */
body {
  font-size: 16px;
  line-height: 1.5;
}
```
</area>

<area name="page-speed">
**Page Performance**

**Accessibility benefit:**
Slow pages are frustrating for everyone, but especially for users with cognitive disabilities who may lose focus, or users on assistive technology where delays compound.

**SEO benefit:**
Core Web Vitals (LCP, INP, CLS) are confirmed ranking factors. Slow sites rank lower and have higher bounce rates.

**Overlap:**
- Optimized images (smaller, faster) benefit both
- Efficient code reduces load times
- Avoiding layout shifts helps everyone
</area>

<area name="skip-links">
**Skip Navigation Links**

**Accessibility benefit (WCAG 2.4.1):**
Keyboard users shouldn't have to tab through navigation on every page. Skip links let them jump directly to main content.

**SEO benefit:**
While not a direct ranking factor, good UX signals (lower bounce rates, higher engagement) indirectly benefit SEO.

**Best practice:**
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
  padding: 8px 16px;
  background: #000;
  color: #fff;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```
</area>

</overlap_areas>

<metrics_correlation>
**Core Web Vitals and Accessibility**

There's strong correlation between accessibility improvements and better Core Web Vitals:

| Accessibility Fix | CWV Impact |
|------------------|------------|
| Add image dimensions | ↓ CLS |
| Clean semantic HTML | ↓ INP (faster parsing) |
| Reduce DOM complexity | ↓ INP, ↓ LCP |
| Optimize images | ↓ LCP |
| Remove layout-shifting ads | ↓ CLS |
| Efficient focus management | ↓ INP |
</metrics_correlation>

<business_case>
**Why Invest in Both**

**Legal compliance:**
- ADA lawsuits exceeded 4,000 in 2024
- WCAG 2.1 AA required for government sites by April 2026
- European Accessibility Act in effect June 2025

**Search visibility:**
- Sites meeting Core Web Vitals rank 28% higher
- Rich snippets from proper markup increase CTR 20-30%
- Mobile-first indexing means mobile UX = rankings

**User experience:**
- Accessible sites are easier for everyone
- Clear structure helps all users find content
- Fast sites reduce bounce rates

**Market reach:**
- 15% of world population has a disability
- Aging population needs accessibility features
- Mobile users benefit from accessibility practices
</business_case>

<implementation_priority>
**Start with fixes that benefit both:**

1. **Add alt text to images** (a11y + image SEO)
2. **Fix heading hierarchy** (a11y + content structure)
3. **Use descriptive link text** (a11y + anchor text signals)
4. **Improve page speed** (a11y + Core Web Vitals)
5. **Ensure mobile responsiveness** (a11y + mobile-first indexing)
6. **Add video transcripts** (a11y + indexable content)
7. **Use semantic HTML** (a11y + better crawling)
8. **Optimize title tags** (a11y + SERP presence)

Each fix improves both accessibility and SEO simultaneously.
</implementation_priority>

<checklist>
**Combined SEO + Accessibility Checklist:**

- [ ] All images have descriptive alt text
- [ ] One h1 per page with primary keyword
- [ ] Heading hierarchy is logical (no skipped levels)
- [ ] All links have descriptive text
- [ ] Page has semantic landmarks (nav, main, footer)
- [ ] Title tags are unique and descriptive
- [ ] Videos have captions/transcripts
- [ ] Site is mobile-responsive
- [ ] Core Web Vitals passing
- [ ] Skip link to main content
- [ ] Keyboard navigation works
- [ ] Color contrast meets 4.5:1
</checklist>
