<overview>
Essential HTML meta tags for SEO. These tags help search engines understand and display your content correctly in search results.
</overview>

<essential_tags>

<tag name="title">
**Title Tag**

**Purpose:** Primary ranking factor, appears in search results and browser tabs.

**Format:**
```html
<title>Primary Keyword - Secondary Info | Brand Name</title>
```

**Guidelines:**
- 50-60 characters (Google truncates at ~60)
- Include primary keyword early
- Make each page's title unique
- Include brand name (usually at end)
- Write for humans first, then optimize

**Examples:**
```html
<!-- Homepage -->
<title>Acme Digital - Web Design & Development Agency</title>

<!-- Service page -->
<title>Technical SEO Services | Acme Digital</title>

<!-- Blog post -->
<title>Complete Guide to Core Web Vitals (2025) | Acme Blog</title>

<!-- Product page -->
<title>Blue Widget Pro - Fast Shipping | Acme Store</title>
```

**Common mistakes:**
- Duplicate titles across pages
- Keyword stuffing
- Missing brand name
- Too long (gets truncated)
- Generic titles ("Home", "Welcome")
</tag>

<tag name="description">
**Meta Description**

**Purpose:** Summary shown in search results. Doesn't directly affect rankings but influences click-through rate.

**Format:**
```html
<meta name="description" content="Compelling description with keyword. Include call-to-action. 150-160 characters.">
```

**Guidelines:**
- 150-160 characters (Google truncates at ~160)
- Include primary keyword naturally
- Write compelling copy (it's an ad for your page)
- Include call-to-action when appropriate
- Make each page's description unique

**Examples:**
```html
<!-- Homepage -->
<meta name="description" content="Award-winning web design agency. We build fast, accessible websites that rank. Get a free consultation today.">

<!-- Service page -->
<meta name="description" content="Professional technical SEO services to improve your site's visibility. Core Web Vitals optimization, schema markup, and site audits. Request a quote.">

<!-- Blog post -->
<meta name="description" content="Learn everything about Core Web Vitals - LCP, INP, and CLS. Includes optimization tips, tools, and a free checklist.">
```

**Common mistakes:**
- Duplicate descriptions
- Too short or missing
- Keyword stuffing
- Not compelling (no reason to click)
- Auto-generated from content
</tag>

<tag name="canonical">
**Canonical URL**

**Purpose:** Tells search engines which URL is the "main" version when content exists at multiple URLs.

**Format:**
```html
<link rel="canonical" href="https://example.com/page">
```

**When to use:**
- Same content at multiple URLs (www vs non-www, http vs https)
- URL parameters (sorting, filtering)
- Syndicated content
- Print-friendly versions
- Mobile URLs (if separate from desktop)

**Examples:**
```html
<!-- Self-referencing canonical (recommended on all pages) -->
<link rel="canonical" href="https://example.com/products/widget">

<!-- Canonical to main version when parameters exist -->
<!-- On: https://example.com/products?sort=price -->
<link rel="canonical" href="https://example.com/products">

<!-- Canonical for syndicated content -->
<link rel="canonical" href="https://original-site.com/article">
```

**Rules:**
- Always use absolute URLs (not relative)
- Use HTTPS if available
- Include on every page (self-referencing is fine)
- Only one canonical per page
</tag>

<tag name="robots">
**Robots Meta Tag**

**Purpose:** Controls how search engines index and follow links on a page.

**Format:**
```html
<meta name="robots" content="directive1, directive2">
```

**Common directives:**
| Directive | Meaning |
|-----------|---------|
| `index` | Allow indexing (default) |
| `noindex` | Prevent indexing |
| `follow` | Follow links (default) |
| `nofollow` | Don't follow links |
| `noarchive` | Don't show cached version |
| `nosnippet` | Don't show description snippet |
| `max-snippet:N` | Limit snippet to N characters |
| `max-image-preview:large` | Allow large image previews |

**Examples:**
```html
<!-- Default: index and follow (can omit) -->
<meta name="robots" content="index, follow">

<!-- Prevent indexing (staging, duplicates, thin pages) -->
<meta name="robots" content="noindex, nofollow">

<!-- Index but don't follow links -->
<meta name="robots" content="index, nofollow">

<!-- Control snippet appearance -->
<meta name="robots" content="max-snippet:150, max-image-preview:large">
```

**Google-specific:**
```html
<meta name="googlebot" content="noindex">
```

**When to use noindex:**
- Staging/development sites
- Thank you/confirmation pages
- Search results pages
- Admin/private pages
- Duplicate content that can't be canonicalized
</tag>

<tag name="viewport">
**Viewport**

**Purpose:** Controls how page is displayed on mobile devices. Required for mobile-friendly design.

**Format:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

**This is the only viewport tag you need.** Don't add:
- `maximum-scale=1` (prevents zooming - accessibility issue)
- `user-scalable=no` (prevents zooming - accessibility issue)
</tag>

</essential_tags>

<social_tags>

<tag name="open-graph">
**Open Graph (Facebook, LinkedIn, etc.)**

**Purpose:** Controls how page appears when shared on social media.

**Essential tags:**
```html
<meta property="og:title" content="Page Title">
<meta property="og:description" content="Page description for social sharing">
<meta property="og:image" content="https://example.com/image.jpg">
<meta property="og:url" content="https://example.com/page">
<meta property="og:type" content="website">
<meta property="og:site_name" content="Site Name">
```

**Image guidelines:**
- Minimum: 1200 x 630 pixels
- Aspect ratio: 1.91:1
- File size: < 8MB
- Format: JPG, PNG, GIF
</tag>

<tag name="twitter">
**Twitter Cards**

**Purpose:** Controls how page appears when shared on Twitter/X.

**Summary card with large image:**
```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@yourusername">
<meta name="twitter:title" content="Page Title">
<meta name="twitter:description" content="Page description">
<meta name="twitter:image" content="https://example.com/image.jpg">
```

**Twitter often falls back to Open Graph tags, so OG tags are more important.
</tag>

</social_tags>

<technical_tags>

<tag name="charset">
**Character Encoding**

```html
<meta charset="UTF-8">
```

Always include. Should be first tag in `<head>`.
</tag>

<tag name="language">
**Language Declaration**

```html
<html lang="en">
```

For multilingual sites:
```html
<link rel="alternate" hreflang="en" href="https://example.com/page">
<link rel="alternate" hreflang="es" href="https://example.com/es/page">
<link rel="alternate" hreflang="x-default" href="https://example.com/page">
```
</tag>

<tag name="favicon">
**Favicon**

```html
<link rel="icon" href="/favicon.ico" sizes="any">
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">
```
</tag>

</technical_tags>

<complete_template>
**Complete Head Section Template:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Character encoding (must be first) -->
  <meta charset="UTF-8">

  <!-- Viewport for mobile -->
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <!-- SEO essentials -->
  <title>Page Title - Brand Name</title>
  <meta name="description" content="Compelling page description under 160 characters.">
  <link rel="canonical" href="https://example.com/page">

  <!-- Robots (if non-default) -->
  <!-- <meta name="robots" content="noindex, nofollow"> -->

  <!-- Open Graph -->
  <meta property="og:title" content="Page Title">
  <meta property="og:description" content="Page description">
  <meta property="og:image" content="https://example.com/og-image.jpg">
  <meta property="og:url" content="https://example.com/page">
  <meta property="og:type" content="website">

  <!-- Twitter -->
  <meta name="twitter:card" content="summary_large_image">

  <!-- Favicon -->
  <link rel="icon" href="/favicon.ico">
  <link rel="apple-touch-icon" href="/apple-touch-icon.png">

  <!-- Preload critical resources -->
  <link rel="preload" as="image" href="/hero.webp">
  <link rel="preload" as="font" href="/font.woff2" crossorigin>

  <!-- CSS -->
  <link rel="stylesheet" href="/styles.css">
</head>
```
</complete_template>

<checklist>
**Meta Tags Checklist:**

- [ ] Unique title tag (50-60 chars) with keyword
- [ ] Unique meta description (150-160 chars)
- [ ] Self-referencing canonical URL
- [ ] Viewport meta tag
- [ ] Open Graph tags (og:title, og:description, og:image, og:url)
- [ ] Twitter card meta tag
- [ ] Language declared on html element
- [ ] Charset UTF-8 as first meta tag
- [ ] No accidental noindex on important pages
</checklist>
