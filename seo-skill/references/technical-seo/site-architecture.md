<overview>
Site architecture affects crawlability, user experience, and how search engines understand your content hierarchy. Good architecture makes important content easy to find for both users and crawlers.
</overview>

<principles>

<principle name="flat-hierarchy">
**Keep It Flat**

Important pages should be reachable within 3 clicks from the homepage.

**Good structure:**
```
Home (depth 0)
├── Category A (depth 1)
│   ├── Subcategory A1 (depth 2)
│   │   └── Product/Article (depth 3)
│   └── Subcategory A2 (depth 2)
├── Category B (depth 1)
├── About (depth 1)
└── Contact (depth 1)
```

**Why it matters:**
- Crawlers find content faster
- PageRank flows more efficiently
- Users find what they need
- Important pages get more authority
</principle>

<principle name="logical-grouping">
**Group Related Content**

Organize content into logical categories and subcategories.

**URL structure reflects hierarchy:**
```
example.com/
├── /products/
│   ├── /products/widgets/
│   │   ├── /products/widgets/blue-widget
│   │   └── /products/widgets/red-widget
│   └── /products/gadgets/
├── /blog/
│   ├── /blog/seo/
│   │   ├── /blog/seo/technical-seo-guide
│   │   └── /blog/seo/keyword-research
│   └── /blog/marketing/
└── /services/
```
</principle>

<principle name="clear-navigation">
**Clear, Consistent Navigation**

Navigation should be predictable and comprehensive.

**Main navigation:**
- Include top-level categories
- Keep to 5-7 items max
- Use clear, descriptive labels
- Consistent across all pages

**Breadcrumbs:**
- Show current location in hierarchy
- Enable easy backtracking
- Implement schema markup
</principle>

</principles>

<url_structure>
**URL Best Practices**

**Do:**
```
example.com/products/blue-widget
example.com/blog/seo-guide
example.com/services/web-design
```

**Don't:**
```
example.com/p?id=123
example.com/index.php?cat=5&item=42
example.com/2024/01/15/post-title-is-very-long
```

**Guidelines:**
- Use hyphens, not underscores
- Keep URLs short (under 60 characters)
- Include primary keyword
- Use lowercase only
- Avoid URL parameters when possible
- Avoid dates in URLs (content appears outdated)
- Avoid unnecessary words (the, a, and)
</url_structure>

<crawl_budget>
**Managing Crawl Budget**

Googlebot has limited resources for each site. Use them wisely.

**What wastes crawl budget:**
- Infinite scroll without pagination
- Faceted navigation creating many URLs
- Duplicate content from URL parameters
- Redirect chains
- Soft 404s (error pages returning 200)
- Large amounts of low-value content

**Solutions:**

**robots.txt for blocking:**
```
User-agent: *
Disallow: /admin/
Disallow: /api/
Disallow: /search?
Disallow: /filter?
```

**Canonical for duplicates:**
```html
<link rel="canonical" href="https://example.com/products">
```

**noindex for low-value pages:**
```html
<meta name="robots" content="noindex, follow">
```

**URL parameters in Search Console:**
- Configure how Google handles URL parameters
- Tell Google which parameters don't change content
</crawl_budget>

<information_architecture>
**Information Architecture Patterns**

<pattern name="hub-and-spoke">
**Hub and Spoke (Topic Clusters)**

Central "hub" page links to related "spoke" pages, which link back.

```
         ┌─────────────────────┐
         │   Pillar Page      │
         │   "Technical SEO"   │
         └─────────┬───────────┘
    ┌──────────────┼──────────────┐
    ▼              ▼              ▼
┌───────┐    ┌───────┐    ┌───────┐
│ CWV   │◄──►│Schema │◄──►│Crawl  │
│ Guide │    │ Guide │    │Budget │
└───────┘    └───────┘    └───────┘
```

**Benefits:**
- Establishes topical authority
- Distributes link equity
- Clear user pathways
</pattern>

<pattern name="siloed">
**Siloed Architecture**

Content grouped into distinct silos with minimal cross-linking.

```
example.com/
├── /seo/           ← SEO content silo
│   ├── /seo/guide
│   └── /seo/tools
├── /marketing/     ← Marketing content silo
│   ├── /marketing/guide
│   └── /marketing/tools
└── /analytics/     ← Analytics content silo
```

**When to use:**
- Large sites with distinct topics
- E-commerce with many categories
- When topics shouldn't cross-pollinate
</pattern>

</information_architecture>

<technical_setup>
**Technical Implementation**

**XML Sitemap:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/</loc>
    <lastmod>2025-01-15</lastmod>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://example.com/products</loc>
    <lastmod>2025-01-10</lastmod>
    <priority>0.8</priority>
  </url>
</urlset>
```

**robots.txt:**
```
User-agent: *
Allow: /

Sitemap: https://example.com/sitemap.xml

Disallow: /admin/
Disallow: /api/
Disallow: /search?
```

**Breadcrumb implementation:**
```html
<nav aria-label="Breadcrumb">
  <ol itemscope itemtype="https://schema.org/BreadcrumbList">
    <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
      <a itemprop="item" href="/">
        <span itemprop="name">Home</span>
      </a>
      <meta itemprop="position" content="1">
    </li>
    <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
      <a itemprop="item" href="/products">
        <span itemprop="name">Products</span>
      </a>
      <meta itemprop="position" content="2">
    </li>
    <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
      <span itemprop="name">Blue Widget</span>
      <meta itemprop="position" content="3">
    </li>
  </ol>
</nav>
```
</technical_setup>

<common_issues>

<issue name="orphan-pages">
**Orphan Pages**

Pages with no internal links pointing to them.

**Finding them:**
- Screaming Frog crawl vs sitemap comparison
- Compare indexed pages in Search Console to crawled pages

**Fixing:**
- Add internal links from relevant content
- Include in navigation or sitemap
- If truly orphaned, consider removing
</issue>

<issue name="deep-pages">
**Pages Too Deep**

Important pages requiring 4+ clicks to reach.

**Fixing:**
- Flatten hierarchy
- Add direct links from homepage
- Improve navigation
- Add to category pages
</issue>

<issue name="broken-paths">
**Broken Navigation Paths**

Links that don't work or lead to wrong places.

**Finding:**
- Regular crawls with Screaming Frog
- Monitoring 404s in Search Console
- User testing

**Fixing:**
- Fix or remove broken links
- Set up 301 redirects for moved content
- Update navigation
</issue>

</common_issues>

<migration_considerations>
**URL Changes and Migrations**

When changing URLs or site structure:

1. **Map old URLs to new URLs**
2. **Implement 301 redirects**
3. **Update internal links** (don't rely on redirects internally)
4. **Update sitemap**
5. **Submit new sitemap to Search Console**
6. **Monitor 404s and fix**
7. **Update canonical tags**

```apache
# .htaccess redirect example
Redirect 301 /old-page https://example.com/new-page
```
</migration_considerations>

<checklist>
**Site Architecture Checklist:**

**Structure:**
- [ ] Important pages within 3 clicks of homepage
- [ ] Logical URL hierarchy (/category/subcategory/page)
- [ ] Clean, descriptive URLs
- [ ] No orphan pages

**Navigation:**
- [ ] Clear main navigation (5-7 items)
- [ ] Breadcrumbs on all pages
- [ ] Footer links to key pages
- [ ] Related content links

**Technical:**
- [ ] XML sitemap created and submitted
- [ ] robots.txt configured correctly
- [ ] No broken internal links
- [ ] Canonical tags on all pages

**Crawlability:**
- [ ] No crawl budget waste (parameters, duplicates)
- [ ] No infinite scroll without pagination
- [ ] No redirect chains
</checklist>
