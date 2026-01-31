# Workflow: Implement Schema Markup

<required_reading>
**Read these reference files NOW:**
1. references/technical-seo/schema-markup.md
</required_reading>

<process>

## Step 1: Identify Schema Opportunities

**Common schema types by page:**

| Page Type | Schema Types |
|-----------|-------------|
| Homepage | Organization, WebSite, SearchAction |
| About | Organization, Person |
| Blog post | Article, BreadcrumbList |
| Product | Product, Offer, Review, BreadcrumbList |
| FAQ page | FAQPage |
| Contact | LocalBusiness, ContactPoint |
| Event | Event |
| Recipe | Recipe |
| How-to | HowTo |

## Step 2: Implement Organization Schema

**For all sites (usually in head or footer):**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Your Company Name",
  "url": "https://yoursite.com",
  "logo": "https://yoursite.com/logo.png",
  "description": "Brief company description",
  "sameAs": [
    "https://twitter.com/yourcompany",
    "https://linkedin.com/company/yourcompany",
    "https://facebook.com/yourcompany"
  ],
  "contactPoint": {
    "@type": "ContactPoint",
    "telephone": "+1-555-555-5555",
    "contactType": "customer service",
    "availableLanguage": "English"
  }
}
</script>
```

## Step 3: Implement WebSite Schema with Search

**For sites with search functionality:**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "Your Site Name",
  "url": "https://yoursite.com",
  "potentialAction": {
    "@type": "SearchAction",
    "target": {
      "@type": "EntryPoint",
      "urlTemplate": "https://yoursite.com/search?q={search_term_string}"
    },
    "query-input": "required name=search_term_string"
  }
}
</script>
```

## Step 4: Implement Article Schema

**For blog posts and articles:**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Article Title (Max 110 characters)",
  "description": "Brief article summary",
  "image": "https://yoursite.com/article-image.jpg",
  "author": {
    "@type": "Person",
    "name": "Author Name",
    "url": "https://yoursite.com/author/name"
  },
  "publisher": {
    "@type": "Organization",
    "name": "Your Site Name",
    "logo": {
      "@type": "ImageObject",
      "url": "https://yoursite.com/logo.png"
    }
  },
  "datePublished": "2025-01-15T08:00:00+00:00",
  "dateModified": "2025-01-20T10:30:00+00:00",
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://yoursite.com/blog/article-slug"
  }
}
</script>
```

## Step 5: Implement BreadcrumbList Schema

**For pages with breadcrumb navigation:**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "https://yoursite.com"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Blog",
      "item": "https://yoursite.com/blog"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "Article Title",
      "item": "https://yoursite.com/blog/article-slug"
    }
  ]
}
</script>
```

## Step 6: Implement FAQPage Schema

**For FAQ sections:**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is technical SEO?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Technical SEO refers to optimizations that help search engines crawl, index, and render your website effectively. This includes site speed, mobile-friendliness, and structured data."
      }
    },
    {
      "@type": "Question",
      "name": "How long does SEO take to work?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "SEO typically takes 3-6 months to show significant results. However, some changes like fixing technical issues can have faster impact."
      }
    }
  ]
}
</script>
```

## Step 7: Implement Product Schema (E-commerce)

**For product pages:**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Product Name",
  "image": [
    "https://yoursite.com/product1.jpg",
    "https://yoursite.com/product2.jpg"
  ],
  "description": "Product description",
  "sku": "SKU123",
  "brand": {
    "@type": "Brand",
    "name": "Brand Name"
  },
  "offers": {
    "@type": "Offer",
    "url": "https://yoursite.com/product",
    "priceCurrency": "USD",
    "price": "99.99",
    "availability": "https://schema.org/InStock",
    "seller": {
      "@type": "Organization",
      "name": "Your Store"
    }
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.5",
    "reviewCount": "89"
  }
}
</script>
```

## Step 8: Implement LocalBusiness Schema

**For businesses with physical locations:**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Business Name",
  "image": "https://yoursite.com/storefront.jpg",
  "url": "https://yoursite.com",
  "telephone": "+1-555-555-5555",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "123 Main Street",
    "addressLocality": "City",
    "addressRegion": "State",
    "postalCode": "12345",
    "addressCountry": "US"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": 40.7128,
    "longitude": -74.0060
  },
  "openingHoursSpecification": [
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
      "opens": "09:00",
      "closes": "17:00"
    }
  ],
  "priceRange": "$$"
}
</script>
```

## Step 9: Implement HowTo Schema

**For tutorial/how-to content:**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "How to Optimize Images for SEO",
  "description": "Learn how to optimize images for better SEO performance",
  "totalTime": "PT15M",
  "step": [
    {
      "@type": "HowToStep",
      "name": "Choose the right format",
      "text": "Use WebP for photos and PNG for graphics with transparency."
    },
    {
      "@type": "HowToStep",
      "name": "Compress images",
      "text": "Use tools like TinyPNG or Squoosh to reduce file size."
    },
    {
      "@type": "HowToStep",
      "name": "Add descriptive alt text",
      "text": "Write alt text that describes the image content and context."
    }
  ]
}
</script>
```

## Step 10: Validate Schema

**Use Google's Rich Results Test:**
1. Go to https://search.google.com/test/rich-results
2. Enter URL or paste code
3. Check for errors and warnings

**Use Schema Validator:**
1. Go to https://validator.schema.org
2. Paste JSON-LD code
3. Verify structure is valid

**Check Search Console:**
1. Go to Google Search Console
2. Enhancements → Check each schema type
3. Fix any reported errors

</process>

<success_criteria>
Schema implementation is complete when:
- [ ] Organization schema on all pages
- [ ] WebSite schema on homepage
- [ ] Article schema on blog posts
- [ ] BreadcrumbList schema on pages with breadcrumbs
- [ ] FAQPage schema on FAQ sections
- [ ] Product schema on product pages (if applicable)
- [ ] LocalBusiness schema (if physical location)
- [ ] All schema validates without errors
- [ ] Rich results appearing in Search Console
</success_criteria>

<anti_patterns>
Avoid:
- Marking up content that isn't visible on page
- Using incorrect schema types for content
- Missing required properties
- Duplicate schema for same entity
- Marking up fake reviews
- Using schema for content that doesn't qualify
- Forgetting to update dateModified
</anti_patterns>
