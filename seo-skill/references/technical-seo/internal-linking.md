<overview>
Internal linking connects pages within your site. Strategic internal linking improves crawlability, distributes page authority, and helps users navigate. It's a critical but often overlooked SEO factor.
</overview>

<why_it_matters>

<benefit name="crawlability">
**Helps Search Engines Discover Pages**

Google has a limited crawl budget for each site. Internal links create pathways for crawlers to discover content.

**Problems with poor internal linking:**
- Orphan pages (no internal links) may never be indexed
- Deep pages (many clicks from homepage) get crawled less frequently
- Important pages without links don't signal importance

**Solution:** Ensure every page has at least one internal link from another page.
</benefit>

<benefit name="link-equity">
**Distributes Page Authority (PageRank)**

Internal links pass authority from high-authority pages to linked pages. This helps important pages rank better.

**Strategy:**
- Link from high-traffic/high-authority pages to important pages
- Use descriptive anchor text (not "click here")
- Don't overlink (dilutes value)
</benefit>

<benefit name="user-experience">
**Improves Navigation**

Good internal linking helps users:
- Discover related content
- Navigate to important pages
- Understand site structure
</benefit>

</why_it_matters>

<best_practices>

<practice name="flat-architecture">
**Keep Important Pages Within 3 Clicks**

Flat site architecture ensures crawlers and users can reach important pages quickly.

**Good structure:**
```
Home
├── Category 1
│   ├── Subcategory A
│   │   └── Product/Article
│   └── Subcategory B
├── Category 2
└── Category 3
```

**Key pages (products, services, important articles) should be reachable in 3 clicks or less from the homepage.**
</practice>

<practice name="contextual-links">
**Add Contextual Links Within Content**

Links within body content are more valuable than navigation links.

**Example:**
```html
<p>Core Web Vitals measure user experience through three metrics.
   For a deep dive, see our
   <a href="/guides/core-web-vitals">complete Core Web Vitals guide</a>.</p>
```

**Guidelines:**
- Link where it's natural and helpful
- Use descriptive anchor text
- Link to related, relevant content
- Aim for 3-5 internal links per 1000 words
</practice>

<practice name="anchor-text">
**Use Descriptive Anchor Text**

Anchor text tells search engines what the linked page is about.

```html
<!-- BAD: Non-descriptive -->
<a href="/pricing">Click here</a>
<a href="/services">Learn more</a>
<a href="/contact">Here</a>

<!-- GOOD: Descriptive -->
<a href="/pricing">view our pricing plans</a>
<a href="/services">explore our SEO services</a>
<a href="/contact">contact our team</a>
```

**Guidelines:**
- Use natural, descriptive text
- Include keywords where natural (don't force)
- Vary anchor text (don't use exact same text every time)
- Avoid generic text ("click here", "read more")
</practice>

<practice name="topic-clusters">
**Build Topic Clusters (Pillar + Cluster Model)**

Organize related content into clusters around a central "pillar" page.

**Structure:**
```
Pillar Page: /guides/technical-seo (comprehensive guide)
    ↓ links to and from ↓
Cluster Pages:
  - /blog/core-web-vitals-optimization
  - /blog/schema-markup-guide
  - /blog/site-speed-tips
  - /blog/mobile-seo-checklist
```

**Implementation:**
1. Pillar page links to all cluster pages
2. Each cluster page links back to pillar
3. Cluster pages link to related cluster pages
4. Consistent anchor text referencing pillar topic
</practice>

<practice name="breadcrumbs">
**Implement Breadcrumb Navigation**

Breadcrumbs show page hierarchy and create internal links.

```html
<nav aria-label="Breadcrumb">
  <ol itemscope itemtype="https://schema.org/BreadcrumbList">
    <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
      <a itemprop="item" href="/"><span itemprop="name">Home</span></a>
      <meta itemprop="position" content="1">
    </li>
    <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
      <a itemprop="item" href="/blog"><span itemprop="name">Blog</span></a>
      <meta itemprop="position" content="2">
    </li>
    <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
      <span itemprop="name">Current Page</span>
      <meta itemprop="position" content="3">
    </li>
  </ol>
</nav>
```

**Benefits:**
- SEO: Creates hierarchical links with schema
- UX: Shows users where they are
- Crawling: Helps search engines understand structure
</practice>

<practice name="related-content">
**Add Related Posts/Products Sections**

Link to related content at the end of pages.

```html
<section>
  <h2>Related Articles</h2>
  <ul>
    <li><a href="/blog/article-1">Article 1 Title</a></li>
    <li><a href="/blog/article-2">Article 2 Title</a></li>
    <li><a href="/blog/article-3">Article 3 Title</a></li>
  </ul>
</section>
```

**Implementation options:**
- Manual curation (best quality)
- Tag/category matching
- Algorithm-based (views, engagement)
</practice>

<practice name="footer-links">
**Strategic Footer Links**

Footer appears on every page, making it valuable for sitewide linking.

**Include:**
- Key service/product pages
- Important informational pages
- Category pages
- Contact/support pages

**Avoid:**
- Too many links (dilutes value)
- Every page on the site
- Keyword-stuffed anchor text
</practice>

</best_practices>

<common_issues>

<issue name="orphan-pages">
**Orphan Pages**

Pages with no internal links pointing to them.

**How to find:**
- Screaming Frog: Crawl site → Orphan pages report
- Google Search Console: Check indexed pages not found in crawl

**Fix:**
- Add links from relevant content
- Include in navigation or sitemap
- Create related content that links to orphans
</issue>

<issue name="broken-links">
**Broken Internal Links**

Links pointing to pages that don't exist (404s).

**How to find:**
```bash
# Using Screaming Frog
# Crawl site → Response Codes → 4xx

# Using CLI tool
npx broken-link-checker https://example.com
```

**Fix:**
- Update link to correct URL
- Create redirect from old to new URL
- Remove link if content no longer exists
</issue>

<issue name="redirect-chains">
**Internal Redirect Chains**

Links that go through multiple redirects before reaching final page.

**Example:**
```
/old-page → 301 → /new-page → 301 → /final-page
```

**Problems:**
- Wastes crawl budget
- Slows page load
- Loses PageRank with each hop

**Fix:** Update links to point directly to final destination.
</issue>

<issue name="too-many-links">
**Too Many Links Per Page**

Having hundreds of links on a single page dilutes their value.

**Guidelines:**
- No hard limit, but be reasonable
- Prioritize quality over quantity
- Google used to recommend ~100 links max
- Focus on useful links for users
</issue>

</common_issues>

<audit_checklist>
**Internal Linking Audit Checklist:**

**Structure:**
- [ ] Key pages within 3 clicks from homepage
- [ ] No orphan pages
- [ ] Logical hierarchy (category → subcategory → page)
- [ ] Breadcrumbs implemented with schema

**Links:**
- [ ] No broken internal links (404s)
- [ ] No redirect chains (update to final URL)
- [ ] Descriptive anchor text (not "click here")
- [ ] Contextual links within content

**Strategy:**
- [ ] Topic clusters around pillar pages
- [ ] Related content sections on articles
- [ ] Important pages linked from high-authority pages
- [ ] New content linked from relevant existing content

**Navigation:**
- [ ] Clear main navigation
- [ ] Footer links to key pages
- [ ] Category/archive pages well-linked
</audit_checklist>

<tools>
**Tools for Internal Link Analysis:**

- **Screaming Frog:** Comprehensive crawl and link analysis
- **Ahrefs Site Audit:** Internal linking opportunities
- **Google Search Console:** Coverage and indexing issues
- **Internal Link Juicer (WordPress):** Automated internal linking
</tools>
