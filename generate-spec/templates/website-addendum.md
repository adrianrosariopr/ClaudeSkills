# Website Addendum Template

<usage>
Append this to the Core Spec for content-driven websites, marketing sites, and static sites. Focus on content, SEO, CMS, and information architecture.
</usage>

<template>
```markdown
---

# Website Addendum

> [!info] Project Type: Website
> This addendum covers website-specific requirements including content strategy, SEO, CMS, and information architecture.

## W1. Information Architecture

### Site Map

```
/
├── / (Home)
├── /about
│   ├── /about/team
│   └── /about/history
├── /products (or /services)
│   ├── /products/{{product-1}}
│   └── /products/{{product-2}}
├── /blog
│   ├── /blog/{{category}}
│   └── /blog/{{post-slug}}
├── /contact
└── /legal
    ├── /legal/privacy
    └── /legal/terms
```

### Navigation Structure

**Primary Navigation:**
{{List of main nav items}}

**Footer Navigation:**
{{List of footer nav items}}

**Mobile Navigation:**
{{Mobile nav behavior - hamburger, bottom nav, etc.}}

---

## W2. Page Templates

| Template | Used For | Key Components |
|----------|----------|----------------|
| Home | Landing page | {{Hero, features, CTA}} |
| Product/Service | Product pages | {{Description, pricing, CTA}} |
| Blog List | Blog index | {{Post grid, filters, pagination}} |
| Blog Post | Individual posts | {{Content, author, related}} |
| Contact | Contact page | {{Form, info, map}} |
| Legal | Privacy/Terms | {{Long-form content}} |

### Template Details

#### Home Page
- **Hero Section:** {{Description}}
- **Key Sections:** {{List of sections}}
- **CTAs:** {{Primary and secondary CTAs}}

#### Blog Post Template
- **Content Areas:** {{Title, meta, body, sidebar}}
- **Social Sharing:** {{Platforms supported}}
- **Related Content:** {{How related posts are selected}}

---

## W3. Content Strategy

### Content Types

| Type | Owner | Workflow | Frequency |
|------|-------|----------|-----------|
| Blog Posts | {{Owner}} | {{Draft → Review → Publish}} | {{Weekly/Monthly}} |
| Product Pages | {{Owner}} | {{Workflow}} | {{As needed}} |
| Landing Pages | {{Owner}} | {{Workflow}} | {{Campaign-based}} |

### Content Governance
- **Style Guide:** {{Link or "To be created"}}
- **Approval Process:** {{Who approves content}}
- **Review Cadence:** {{How often content is audited}}

### Media Management
- **Image Requirements:** {{Sizes, formats, naming}}
- **Video Hosting:** {{Platform - YouTube, Vimeo, self-hosted}}
- **Asset Organization:** {{Folder structure}}

---

## W4. SEO Requirements

### Technical SEO

**Meta Requirements:**
- [ ] Unique title tags (50-60 chars)
- [ ] Meta descriptions (150-160 chars)
- [ ] Canonical URLs on all pages
- [ ] Open Graph tags for social
- [ ] Twitter Card tags

**Structured Data:**
- [ ] Organization schema
- [ ] Breadcrumb schema
- [ ] Article schema (for blog)
- [ ] Product schema (if applicable)
- [ ] FAQ schema (where relevant)

**Crawlability:**
- [ ] robots.txt configured
- [ ] XML sitemap generated
- [ ] No orphan pages
- [ ] Proper internal linking

### URL Structure
- Pattern: `{{/category/subcategory/page-slug}}`
- Trailing slashes: {{Yes/No}}
- Case: {{lowercase}}

### Redirects
| Old URL | New URL | Type |
|---------|---------|------|
| {{/old-path}} | {{/new-path}} | 301 |

---

## W5. CMS Requirements

### CMS Platform
- **System:** {{WordPress, Strapi, Contentful, etc. or "Static"}}
- **Version:** {{Version number}}

### User Roles

| Role | Capabilities |
|------|--------------|
| Admin | Full access |
| Editor | Edit all content, publish |
| Author | Create own content, submit for review |
| Contributor | Create drafts only |

### Content Modules

| Module | Description | Fields |
|--------|-------------|--------|
| Hero | Page hero sections | {{title, subtitle, image, CTA}} |
| Feature Grid | Feature showcases | {{items[], layout}} |
| Testimonial | Customer quotes | {{quote, author, company, image}} |
| CTA Block | Call to action | {{heading, text, button}} |

### Publishing Workflow
1. {{Draft created}}
2. {{Review requested}}
3. {{Approved/Revisions}}
4. {{Scheduled/Published}}

---

## W6. Performance Requirements

### Core Web Vitals Targets

| Metric | Target | Current |
|--------|--------|---------|
| LCP (Largest Contentful Paint) | < 2.5s | {{Current}} |
| INP (Interaction to Next Paint) | < 200ms | {{Current}} |
| CLS (Cumulative Layout Shift) | < 0.1 | {{Current}} |

### Page Weight Budget

| Resource | Budget |
|----------|--------|
| HTML | < 100KB |
| CSS | < 100KB |
| JavaScript | < 300KB |
| Images (above fold) | < 500KB |
| Total Page | < 2MB |

### Performance Optimizations
- [ ] Image optimization (WebP, AVIF)
- [ ] Lazy loading for below-fold images
- [ ] Critical CSS inlined
- [ ] JavaScript deferred/async
- [ ] CDN configured
- [ ] Caching headers set
- [ ] Gzip/Brotli compression

---

## W7. Accessibility

### WCAG Target
- **Level:** {{AA or AAA}}
- **Version:** {{2.1 or 2.2}}

### Key Requirements
- [ ] Color contrast ratios met (4.5:1 text, 3:1 UI)
- [ ] All images have alt text
- [ ] Form labels properly associated
- [ ] Keyboard navigation works
- [ ] Focus indicators visible
- [ ] Skip links present
- [ ] ARIA landmarks used
- [ ] Responsive text sizing

---

## W8. Hosting & Infrastructure

### Hosting
- **Provider:** {{Vercel, Netlify, AWS, etc.}}
- **Type:** {{Static, SSR, ISR}}
- **CDN:** {{Cloudflare, CloudFront, etc.}}

### Domains
| Domain | Purpose | SSL |
|--------|---------|-----|
| {{example.com}} | Production | Yes |
| {{staging.example.com}} | Staging | Yes |

### Environment Variables
{{List of required env vars - no values}}
- `{{VAR_NAME}}` - {{Purpose}}

### Build & Deploy
- **Build Command:** `{{npm run build}}`
- **Output Directory:** `{{dist, .next, public}}`
- **Deploy Trigger:** {{Git push, manual, scheduled}}

---

## W9. Analytics & Tracking

### Analytics Platform
- **Primary:** {{Google Analytics 4, Plausible, etc.}}
- **Tag Manager:** {{GTM, Segment, etc.}}

### Key Events

| Event | Trigger | Parameters |
|-------|---------|------------|
| page_view | Page load | page_path, page_title |
| form_submit | Form submission | form_name, form_location |
| cta_click | CTA clicked | cta_text, cta_location |
| file_download | Download clicked | file_name, file_type |

### Conversion Goals
1. {{Contact form submission}}
2. {{Newsletter signup}}
3. {{Product inquiry}}

---

*Website Addendum - Generated {{YYYY-MM-DD}}*
```
</template>
