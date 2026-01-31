<overview>
Schema markup (structured data) helps search engines understand your content. It can enable rich results in search, increasing visibility and click-through rates.
</overview>

<basics>

<what_is_schema>
**What is Schema Markup?**

Schema markup is structured data that explicitly tells search engines what your content means. Instead of Google inferring that a page is a recipe, schema says "this IS a recipe with these ingredients, this cook time, these ratings."

**Format:** JSON-LD is recommended (JavaScript Object Notation for Linked Data).

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Article Title",
  "author": {
    "@type": "Person",
    "name": "Author Name"
  }
}
</script>
```
</what_is_schema>

<why_use_schema>
**Why Use Schema Markup?**

1. **Rich Results:** Enhanced search listings with ratings, prices, images
2. **Higher CTR:** Rich results get 20-30% more clicks
3. **AI Visibility:** Better representation in AI answers and features
4. **Knowledge Graph:** Entities may appear in Google's Knowledge Panel

**Not a direct ranking factor, but improves visibility and CTR.**
</why_use_schema>

</basics>

<common_types>

<type name="organization">
**Organization**

Use on every site to establish brand identity.

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Company Name",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png",
  "description": "Brief company description",
  "foundingDate": "2020",
  "sameAs": [
    "https://twitter.com/company",
    "https://linkedin.com/company/company",
    "https://facebook.com/company"
  ],
  "contactPoint": {
    "@type": "ContactPoint",
    "telephone": "+1-555-555-5555",
    "contactType": "customer service",
    "availableLanguage": "English"
  }
}
```

**Required:** name, url
**Recommended:** logo, description, sameAs, contactPoint
</type>

<type name="local-business">
**LocalBusiness**

For businesses with physical locations.

```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Business Name",
  "image": "https://example.com/photo.jpg",
  "url": "https://example.com",
  "telephone": "+1-555-555-5555",
  "priceRange": "$$",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "123 Main St",
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
  ]
}
```

**Specific types:** Restaurant, Store, MedicalBusiness, etc.
</type>

<type name="article">
**Article**

For blog posts, news articles, and editorial content.

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Article Title Here",
  "description": "Brief article summary",
  "image": "https://example.com/article-image.jpg",
  "author": {
    "@type": "Person",
    "name": "Author Name",
    "url": "https://example.com/author/name"
  },
  "publisher": {
    "@type": "Organization",
    "name": "Site Name",
    "logo": {
      "@type": "ImageObject",
      "url": "https://example.com/logo.png"
    }
  },
  "datePublished": "2025-01-15T08:00:00+00:00",
  "dateModified": "2025-01-20T10:30:00+00:00",
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://example.com/article-url"
  }
}
```

**Types:** Article, NewsArticle, BlogPosting
</type>

<type name="product">
**Product**

For e-commerce product pages.

```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Product Name",
  "image": [
    "https://example.com/product1.jpg",
    "https://example.com/product2.jpg"
  ],
  "description": "Product description",
  "sku": "SKU123",
  "brand": {
    "@type": "Brand",
    "name": "Brand Name"
  },
  "offers": {
    "@type": "Offer",
    "url": "https://example.com/product",
    "priceCurrency": "USD",
    "price": "99.99",
    "availability": "https://schema.org/InStock",
    "itemCondition": "https://schema.org/NewCondition",
    "seller": {
      "@type": "Organization",
      "name": "Store Name"
    }
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.5",
    "reviewCount": "89"
  }
}
```

**Enables:** Price, availability, ratings in search results
</type>

<type name="faqpage">
**FAQPage**

For FAQ sections.

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is the return policy?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "We offer 30-day returns on all items. Items must be unused and in original packaging."
      }
    },
    {
      "@type": "Question",
      "name": "How long does shipping take?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Standard shipping takes 5-7 business days. Express shipping is 2-3 days."
      }
    }
  ]
}
```

**Note:** Only use for actual FAQ content, not manufactured Q&A.
</type>

<type name="breadcrumb">
**BreadcrumbList**

For breadcrumb navigation.

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "https://example.com"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Category",
      "item": "https://example.com/category"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "Current Page"
    }
  ]
}
```

**Note:** Last item (current page) doesn't need `item` property.
</type>

<type name="howto">
**HowTo**

For instructional content.

```json
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "How to Optimize Images for SEO",
  "description": "Learn to optimize images for better performance and SEO",
  "totalTime": "PT15M",
  "step": [
    {
      "@type": "HowToStep",
      "name": "Choose format",
      "text": "Use WebP for photos, PNG for graphics with transparency.",
      "image": "https://example.com/step1.jpg"
    },
    {
      "@type": "HowToStep",
      "name": "Compress images",
      "text": "Use TinyPNG or Squoosh to reduce file size.",
      "image": "https://example.com/step2.jpg"
    },
    {
      "@type": "HowToStep",
      "name": "Add alt text",
      "text": "Write descriptive alt text for all images.",
      "image": "https://example.com/step3.jpg"
    }
  ]
}
```

**Enables:** Step-by-step rich results
</type>

<type name="event">
**Event**

For events (concerts, webinars, conferences).

```json
{
  "@context": "https://schema.org",
  "@type": "Event",
  "name": "SEO Workshop 2025",
  "description": "Learn advanced SEO techniques",
  "startDate": "2025-03-15T09:00:00-05:00",
  "endDate": "2025-03-15T17:00:00-05:00",
  "eventStatus": "https://schema.org/EventScheduled",
  "eventAttendanceMode": "https://schema.org/OnlineEventAttendanceMode",
  "location": {
    "@type": "VirtualLocation",
    "url": "https://example.com/event"
  },
  "organizer": {
    "@type": "Organization",
    "name": "Acme Digital",
    "url": "https://example.com"
  },
  "offers": {
    "@type": "Offer",
    "price": "99.00",
    "priceCurrency": "USD",
    "availability": "https://schema.org/InStock",
    "url": "https://example.com/event/tickets"
  }
}
```
</type>

<type name="website">
**WebSite with SearchAction**

For sites with search functionality.

```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "Site Name",
  "url": "https://example.com",
  "potentialAction": {
    "@type": "SearchAction",
    "target": {
      "@type": "EntryPoint",
      "urlTemplate": "https://example.com/search?q={search_term_string}"
    },
    "query-input": "required name=search_term_string"
  }
}
```

**Enables:** Sitelinks search box in Google results
</type>

</common_types>

<validation>
**Validating Schema Markup**

**Google Rich Results Test:**
- URL: https://search.google.com/test/rich-results
- Tests if schema qualifies for rich results
- Shows detected types and any errors

**Schema.org Validator:**
- URL: https://validator.schema.org/
- Validates JSON-LD structure
- More detailed error messages

**Google Search Console:**
- Shows schema performance over time
- Reports errors across your site
- Enhancements section for each type
</validation>

<best_practices>
**Schema Best Practices:**

1. **Only mark up visible content** - Schema should reflect what users see
2. **Use specific types** - `Restaurant` not just `LocalBusiness`
3. **Include all required properties** - Check schema.org for requirements
4. **Keep schema updated** - Update dateModified, prices, availability
5. **Don't mark up fake content** - No manufactured reviews or fake FAQs
6. **Test before deploying** - Validate with Rich Results Test
7. **Monitor in Search Console** - Check for errors regularly
</best_practices>

<implementation_priority>
**Recommended Implementation Order:**

1. **Organization** - Establish brand identity (all sites)
2. **WebSite** - Enable sitelinks search box
3. **BreadcrumbList** - Show page hierarchy
4. **Article/BlogPosting** - For content sites
5. **Product** - For e-commerce
6. **FAQPage** - If you have FAQ sections
7. **LocalBusiness** - For physical locations
8. **HowTo** - For instructional content
9. **Event** - For event listings
</implementation_priority>
