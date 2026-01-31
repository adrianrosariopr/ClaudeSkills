# Workflow: Optimize Content for SEO

<required_reading>
**Read these reference files NOW:**
1. references/technical-seo/meta-tags.md
2. references/accessibility/semantic-html.md
</required_reading>

<process>

## Step 1: Analyze Current Content Structure

**Check existing headings:**
- Is there one `<h1>` that matches page topic?
- Do `<h2>` headings outline main sections?
- Is hierarchy logical (no skipped levels)?

**Check content quality:**
- Does content answer user's search intent?
- Is it comprehensive for the topic?
- Is it up-to-date?

## Step 2: Optimize Heading Structure

**H1: Primary topic (one per page):**
```html
<!-- Include primary keyword naturally -->
<h1>Complete Guide to Technical SEO</h1>
```

**H2: Main sections:**
```html
<h2>What is Technical SEO?</h2>
<h2>Core Web Vitals Explained</h2>
<h2>How to Audit Your Site</h2>
```

**H3+: Subsections:**
```html
<h2>Core Web Vitals Explained</h2>
  <h3>Largest Contentful Paint (LCP)</h3>
  <h3>Interaction to Next Paint (INP)</h3>
  <h3>Cumulative Layout Shift (CLS)</h3>
```

**Heading guidelines:**
- Front-load keywords when natural
- Make headings descriptive (not "Introduction")
- Keep headings scannable (under 70 chars)
- Use headings as outline (should make sense alone)

## Step 3: Optimize for Search Intent

**Understand the four intent types:**

| Intent | Example Query | Content Type |
|--------|--------------|--------------|
| Informational | "what is SEO" | Guide, tutorial, explanation |
| Navigational | "google search console" | Brand/product page |
| Transactional | "buy SEO tools" | Product/pricing page |
| Commercial | "best SEO tools 2025" | Comparison, review |

**Match content to intent:**
```html
<!-- Informational: Answer the question -->
<h1>What is Technical SEO? A Complete Guide</h1>

<!-- Commercial: Compare options -->
<h1>10 Best SEO Tools Compared (2025)</h1>

<!-- Transactional: Enable action -->
<h1>SEO Audit Services - Get Started Today</h1>
```

## Step 4: Optimize Keyword Usage

**Primary keyword placement:**
- H1 heading (required)
- First paragraph (natural mention)
- URL slug
- Title tag
- Meta description

**Secondary keywords:**
- H2/H3 headings (where natural)
- Body content (LSI/related terms)
- Image alt text (when describing image)

**Avoid keyword stuffing:**
```html
<!-- BAD: Forced and repetitive -->
<p>Our SEO services provide the best SEO for your SEO needs.
   Our SEO experts do SEO optimization for SEO results.</p>

<!-- GOOD: Natural and varied -->
<p>Our search optimization services help improve your site's
   visibility. Our experts analyze rankings and implement
   strategies that drive organic traffic.</p>
```

## Step 5: Improve Readability

**Use short paragraphs:**
```html
<!-- BAD: Wall of text -->
<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit.
   Sed do eiusmod tempor incididunt ut labore et dolore magna
   aliqua. Ut enim ad minim veniam, quis nostrud exercitation
   ullamco laboris nisi ut aliquip ex ea commodo consequat...</p>

<!-- GOOD: Scannable chunks -->
<p>Short paragraphs are easier to scan.</p>

<p>Aim for 2-3 sentences per paragraph on the web.</p>

<p>This improves readability and engagement.</p>
```

**Use lists for multiple items:**
```html
<!-- Instead of comma-separated items in a paragraph -->
<ul>
  <li>Improve page speed</li>
  <li>Optimize meta tags</li>
  <li>Fix broken links</li>
  <li>Add internal links</li>
</ul>
```

**Use formatting for emphasis:**
```html
<p>The most important factor is <strong>content quality</strong>.
   Google's E-E-A-T guidelines emphasize <em>expertise</em> and
   <em>trustworthiness</em>.</p>
```

## Step 6: Add Structured Content Elements

**Tables for comparisons:**
```html
<table>
  <caption>SEO Tool Comparison</caption>
  <thead>
    <tr>
      <th scope="col">Tool</th>
      <th scope="col">Price</th>
      <th scope="col">Best For</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Screaming Frog</td>
      <td>$259/year</td>
      <td>Technical audits</td>
    </tr>
  </tbody>
</table>
```

**Definition lists for terms:**
```html
<dl>
  <dt>LCP (Largest Contentful Paint)</dt>
  <dd>Measures loading performance. Should be under 2.5 seconds.</dd>

  <dt>INP (Interaction to Next Paint)</dt>
  <dd>Measures responsiveness. Should be under 200 milliseconds.</dd>
</dl>
```

**FAQ sections (with schema):**
```html
<section itemscope itemtype="https://schema.org/FAQPage">
  <h2>Frequently Asked Questions</h2>

  <div itemscope itemprop="mainEntity" itemtype="https://schema.org/Question">
    <h3 itemprop="name">What is technical SEO?</h3>
    <div itemscope itemprop="acceptedAnswer" itemtype="https://schema.org/Answer">
      <p itemprop="text">Technical SEO refers to optimizations that help
         search engines crawl and index your site effectively...</p>
    </div>
  </div>
</section>
```

## Step 7: Optimize for Featured Snippets

**Target paragraph snippets:**
```html
<h2>What is Core Web Vitals?</h2>
<p>Core Web Vitals are a set of three metrics that measure user
   experience: Largest Contentful Paint (LCP) for loading,
   Interaction to Next Paint (INP) for interactivity, and
   Cumulative Layout Shift (CLS) for visual stability.</p>
```

**Target list snippets:**
```html
<h2>How to Improve Page Speed</h2>
<ol>
  <li>Compress and optimize images</li>
  <li>Minimize CSS and JavaScript</li>
  <li>Enable browser caching</li>
  <li>Use a CDN</li>
  <li>Reduce server response time</li>
</ol>
```

**Target table snippets:**
```html
<h2>WCAG Conformance Levels</h2>
<table>
  <tr><th>Level</th><th>Description</th></tr>
  <tr><td>A</td><td>Minimum accessibility</td></tr>
  <tr><td>AA</td><td>Legal standard (ADA)</td></tr>
  <tr><td>AAA</td><td>Highest accessibility</td></tr>
</table>
```

## Step 8: Internal Linking Within Content

**Link to related content:**
```html
<p>For more details on implementing structured data,
   see our <a href="/guides/schema-markup">complete schema markup guide</a>.</p>
```

**Use descriptive anchor text:**
```html
<!-- BAD -->
<a href="/pricing">Click here</a>
<a href="/docs">Read more</a>

<!-- GOOD -->
<a href="/pricing">View our pricing plans</a>
<a href="/docs">Read the complete documentation</a>
```

**Link density:**
- Aim for 3-5 internal links per 1000 words
- Link to related content naturally
- Ensure links add value for readers

## Step 9: Verify Content Optimization

**Check with tools:**
- Yoast SEO / Rank Math (WordPress)
- Clearscope / Surfer SEO (content optimization)
- Hemingway Editor (readability)

**Manual review:**
- Read content aloud - does it flow?
- Scan headings only - is the structure clear?
- Check keyword placement - is it natural?

</process>

<success_criteria>
Content is optimized when:
- [ ] One H1 with primary keyword
- [ ] Logical heading hierarchy (H2, H3, etc.)
- [ ] Content matches search intent
- [ ] Keywords placed naturally (not stuffed)
- [ ] Short, scannable paragraphs
- [ ] Lists and tables where appropriate
- [ ] Internal links to related content
- [ ] Descriptive anchor text
- [ ] Structured for featured snippets
- [ ] Readable at 8th-grade level
</success_criteria>

<anti_patterns>
Avoid:
- Multiple H1 tags per page
- Skipping heading levels (H1 → H3)
- Keyword stuffing
- Thin content (under 300 words for main pages)
- "Click here" link text
- Walls of text without breaks
- Ignoring search intent
- Writing for bots, not humans
</anti_patterns>
