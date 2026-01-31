# Workflow: Optimize Technical SEO

<required_reading>
**Read these reference files NOW:**
1. references/technical-seo/meta-tags.md
2. references/technical-seo/site-architecture.md
3. references/core/core-web-vitals.md
</required_reading>

<process>

## Step 1: Optimize Meta Tags

**Title tags (50-60 characters):**
```html
<!-- BAD -->
<title>Home</title>
<title>Welcome to Our Amazing Company - We Do Many Things</title>

<!-- GOOD -->
<title>Primary Keyword - Brand Name</title>
<title>Web Design Services | Acme Digital</title>
```

**Meta descriptions (150-160 characters):**
```html
<meta name="description" content="Clear, compelling description with primary keyword. Include a call-to-action. This appears in search results.">
```

**Canonical URLs:**
```html
<!-- Prevent duplicate content issues -->
<link rel="canonical" href="https://example.com/page">
```

**Robots meta:**
```html
<!-- Default: index and follow -->
<meta name="robots" content="index, follow">

<!-- Prevent indexing (staging, duplicates) -->
<meta name="robots" content="noindex, nofollow">
```

## Step 2: Optimize URL Structure

**Clean, descriptive URLs:**
```
BAD:  /page?id=123&cat=5
BAD:  /2024/01/15/post-title-here-is-very-long
GOOD: /blog/seo-best-practices
GOOD: /products/blue-widget
```

**URL guidelines:**
- Use hyphens, not underscores
- Keep under 60 characters
- Include primary keyword
- Use lowercase only
- Avoid parameters when possible

## Step 3: Implement XML Sitemap

**Create sitemap.xml:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/</loc>
    <lastmod>2025-01-15</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://example.com/about</loc>
    <lastmod>2025-01-10</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

**Submit to Search Console:**
1. Go to Google Search Console
2. Sitemaps → Add new sitemap
3. Enter: sitemap.xml
4. Submit

## Step 4: Configure robots.txt

**Basic robots.txt:**
```
User-agent: *
Allow: /

Sitemap: https://example.com/sitemap.xml

# Block admin/private areas
Disallow: /admin/
Disallow: /api/
Disallow: /private/
```

**Common mistakes to avoid:**
```
# BAD: Blocking CSS/JS (breaks rendering)
Disallow: /css/
Disallow: /js/

# BAD: Blocking entire site accidentally
User-agent: *
Disallow: /
```

## Step 5: Optimize Core Web Vitals

**LCP (Largest Contentful Paint) < 2.5s:**

```html
<!-- Preload critical images -->
<link rel="preload" as="image" href="hero.webp">

<!-- Optimize hero image -->
<img src="hero.webp"
     alt="Hero description"
     width="1200"
     height="600"
     fetchpriority="high">
```

```css
/* Avoid render-blocking CSS */
@media print {
  /* Move print styles to separate file with media="print" */
}
```

**INP (Interaction to Next Paint) < 200ms:**

```javascript
// BAD: Long task blocking main thread
function processData() {
  // Heavy computation for 500ms
}

// GOOD: Break up long tasks
async function processData() {
  for (const chunk of dataChunks) {
    processChunk(chunk);
    await new Promise(r => setTimeout(r, 0)); // Yield to main thread
  }
}
```

**CLS (Cumulative Layout Shift) < 0.1:**

```html
<!-- Always specify dimensions -->
<img src="photo.jpg" width="400" height="300" alt="...">
<video width="640" height="360">...</video>
<iframe width="560" height="315">...</iframe>
```

```css
/* Reserve space for dynamic content */
.ad-container {
  min-height: 250px;
}

/* Avoid inserting content above existing content */
.notification {
  position: fixed; /* Doesn't shift layout */
}
```

## Step 6: Optimize Images

**Use modern formats:**
```html
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Description" width="800" height="600">
</picture>
```

**Lazy load below-fold images:**
```html
<!-- Above fold: load immediately -->
<img src="hero.webp" alt="..." fetchpriority="high">

<!-- Below fold: lazy load -->
<img src="photo.webp" alt="..." loading="lazy">
```

**Responsive images:**
```html
<img srcset="small.jpg 400w,
             medium.jpg 800w,
             large.jpg 1200w"
     sizes="(max-width: 600px) 400px,
            (max-width: 1200px) 800px,
            1200px"
     src="medium.jpg"
     alt="Description">
```

## Step 7: Optimize Internal Linking

**Flat site architecture:**
- Key pages within 3 clicks from homepage
- Clear navigation hierarchy
- Breadcrumbs for deep pages

```html
<nav aria-label="Breadcrumb">
  <ol itemscope itemtype="https://schema.org/BreadcrumbList">
    <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
      <a itemprop="item" href="/"><span itemprop="name">Home</span></a>
      <meta itemprop="position" content="1">
    </li>
    <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
      <a itemprop="item" href="/products"><span itemprop="name">Products</span></a>
      <meta itemprop="position" content="2">
    </li>
  </ol>
</nav>
```

**Content clusters:**
- Pillar page links to all related content
- Related content links back to pillar
- Related content links to each other

## Step 8: Mobile Optimization

**Responsive viewport:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

**Touch targets (min 48x48px):**
```css
button, a {
  min-height: 48px;
  min-width: 48px;
  padding: 12px;
}
```

**Test mobile experience:**
- Chrome DevTools → Device Mode
- Google Mobile-Friendly Test

## Step 9: Verify Optimizations

```bash
# Run Lighthouse
npx lighthouse https://example.com --only-categories=performance,seo

# Check Core Web Vitals in Search Console
# Google Search Console → Core Web Vitals report
```

</process>

<success_criteria>
Technical SEO is optimized when:
- [ ] All pages have unique title tags (50-60 chars)
- [ ] All pages have meta descriptions (150-160 chars)
- [ ] Canonical URLs set on all pages
- [ ] XML sitemap created and submitted
- [ ] robots.txt properly configured
- [ ] Core Web Vitals passing (LCP < 2.5s, INP < 200ms, CLS < 0.1)
- [ ] Images optimized (WebP, lazy loading, dimensions specified)
- [ ] Internal linking connects related content
- [ ] Site is mobile-friendly
- [ ] No crawl errors in Search Console
</success_criteria>

<anti_patterns>
Avoid:
- Blocking CSS/JS in robots.txt
- Duplicate title tags across pages
- Missing image dimensions (causes CLS)
- Lazy loading above-fold images
- Orphan pages with no internal links
- Deep nesting (more than 3 clicks from home)
- Keyword stuffing in URLs
</anti_patterns>
