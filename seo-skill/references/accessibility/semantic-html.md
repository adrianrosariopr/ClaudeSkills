<overview>
Semantic HTML uses elements that convey meaning, not just appearance. It's the foundation of web accessibility and benefits SEO by helping search engines understand content structure.
</overview>

<why_semantic>
**Why Semantic HTML Matters**

1. **Accessibility:** Screen readers announce element types and navigate by structure
2. **SEO:** Search engines understand content hierarchy and meaning
3. **Maintainability:** Code is self-documenting
4. **Future-proof:** Works with any styling approach
5. **Performance:** Often requires less CSS/JS
</why_semantic>

<document_structure>
**Document Structure Elements**

<element name="header">
**`<header>`**

Introductory content, typically navigation and branding.

```html
<header>
  <nav>...</nav>
  <h1>Site Title</h1>
</header>
```

**Implicit role:** `banner` (when direct child of body)
</element>

<element name="nav">
**`<nav>`**

Major navigation blocks.

```html
<nav aria-label="Main">
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>
```

**Implicit role:** `navigation`

**Best practices:**
- Use aria-label to distinguish multiple navs
- Limit to 3 nav elements per page maximum
- Don't use for every group of links
</element>

<element name="main">
**`<main>`**

Primary content of the page. Skip links point here.

```html
<main id="main-content">
  <!-- Primary page content -->
</main>
```

**Implicit role:** `main`

**Rules:**
- Only one `<main>` per page
- Should not be inside `<article>`, `<aside>`, `<header>`, `<footer>`, or `<nav>`
</element>

<element name="article">
**`<article>`**

Self-contained, independently distributable content.

```html
<article>
  <h2>Blog Post Title</h2>
  <p>Content that makes sense on its own...</p>
</article>
```

**Implicit role:** `article`

**Use for:**
- Blog posts
- News articles
- Forum posts
- Product cards
- Comments
</element>

<element name="section">
**`<section>`**

Thematic grouping of content, typically with a heading.

```html
<section aria-labelledby="features-heading">
  <h2 id="features-heading">Features</h2>
  <p>Content about features...</p>
</section>
```

**Implicit role:** `region` (when labeled)

**Use for:**
- Chapters
- Tab panels
- Thematic sections of a page
</element>

<element name="aside">
**`<aside>`**

Content tangentially related to surrounding content.

```html
<aside>
  <h2>Related Articles</h2>
  <ul>...</ul>
</aside>
```

**Implicit role:** `complementary`

**Use for:**
- Sidebars
- Pull quotes
- Advertising
- Related links
</element>

<element name="footer">
**`<footer>`**

Footer for its nearest ancestor sectioning content or root.

```html
<footer>
  <p>&copy; 2025 Company Name</p>
  <nav aria-label="Footer">...</nav>
</footer>
```

**Implicit role:** `contentinfo` (when direct child of body)
</element>

</document_structure>

<text_elements>
**Text Semantics**

<element name="headings">
**`<h1>` - `<h6>`**

Headings create document outline. Screen readers navigate by heading.

```html
<h1>Page Title</h1>
  <h2>Section</h2>
    <h3>Subsection</h3>
  <h2>Another Section</h2>
```

**Rules:**
- One `<h1>` per page
- Don't skip levels (h1 → h2 → h3, not h1 → h3)
- Use for structure, not styling (use CSS for visual size)
</element>

<element name="p">
**`<p>`**

Paragraph of text.

```html
<p>A paragraph of content.</p>
```
</element>

<element name="strong-em">
**`<strong>` and `<em>`**

Semantic emphasis, not just styling.

```html
<!-- Strong importance -->
<p><strong>Warning:</strong> This action cannot be undone.</p>

<!-- Emphasis (stress) -->
<p>You <em>must</em> complete this step first.</p>
```

**Note:** `<b>` and `<i>` are for styling without semantic meaning.
</element>

<element name="blockquote-q">
**`<blockquote>` and `<q>`**

Quotations.

```html
<!-- Block quotation -->
<blockquote cite="https://source.com">
  <p>Quoted text here.</p>
  <footer>— Author Name</footer>
</blockquote>

<!-- Inline quotation -->
<p>The expert said <q>this is important</q>.</p>
```
</element>

<element name="code-pre">
**`<code>` and `<pre>`**

Code and preformatted text.

```html
<!-- Inline code -->
<p>Use the <code>npm install</code> command.</p>

<!-- Code block -->
<pre><code>function hello() {
  console.log('Hello');
}</code></pre>
```
</element>

<element name="abbr">
**`<abbr>`**

Abbreviations and acronyms.

```html
<abbr title="Web Content Accessibility Guidelines">WCAG</abbr>
```
</element>

</text_elements>

<lists>
**List Elements**

<element name="ul-ol">
**`<ul>` and `<ol>`**

Unordered and ordered lists.

```html
<!-- Unordered list -->
<ul>
  <li>First item</li>
  <li>Second item</li>
</ul>

<!-- Ordered list -->
<ol>
  <li>Step one</li>
  <li>Step two</li>
</ol>
```

**Screen readers announce:** "List, 3 items"
</element>

<element name="dl">
**`<dl>`**

Description list (term-description pairs).

```html
<dl>
  <dt>HTML</dt>
  <dd>HyperText Markup Language</dd>

  <dt>CSS</dt>
  <dd>Cascading Style Sheets</dd>
</dl>
```

**Use for:**
- Glossaries
- Metadata (key-value pairs)
- FAQs (question-answer)
</element>

</lists>

<forms>
**Form Elements**

<element name="form">
**Form structure**

```html
<form action="/submit" method="post">
  <fieldset>
    <legend>Contact Information</legend>

    <label for="name">Name</label>
    <input type="text" id="name" name="name" required>

    <label for="email">Email</label>
    <input type="email" id="email" name="email" required>
  </fieldset>

  <button type="submit">Submit</button>
</form>
```
</element>

<element name="label">
**`<label>`**

Always associate labels with inputs.

```html
<!-- Explicit association -->
<label for="email">Email address</label>
<input type="email" id="email">

<!-- Implicit association -->
<label>
  Email address
  <input type="email">
</label>
```
</element>

<element name="fieldset-legend">
**`<fieldset>` and `<legend>`**

Group related form fields.

```html
<fieldset>
  <legend>Shipping Address</legend>
  <!-- Address fields -->
</fieldset>
```
</element>

<element name="button">
**`<button>`**

Always use `<button>` for clickable actions.

```html
<!-- Submit button -->
<button type="submit">Submit Form</button>

<!-- Generic button -->
<button type="button" onclick="doSomething()">Click Me</button>

<!-- DON'T use div as button -->
<div onclick="doSomething()">Click Me</div> <!-- BAD -->
```
</element>

</forms>

<media>
**Media Elements**

<element name="figure-figcaption">
**`<figure>` and `<figcaption>`**

Self-contained content with optional caption.

```html
<figure>
  <img src="chart.png" alt="Sales chart showing 25% growth">
  <figcaption>Figure 1: Q4 2024 Sales Growth</figcaption>
</figure>
```
</element>

<element name="picture">
**`<picture>`**

Responsive images with multiple sources.

```html
<picture>
  <source media="(max-width: 600px)" srcset="small.jpg">
  <source media="(max-width: 1200px)" srcset="medium.jpg">
  <img src="large.jpg" alt="Description">
</picture>
```
</element>

</media>

<tables>
**Table Elements**

```html
<table>
  <caption>Monthly Sales by Region</caption>
  <thead>
    <tr>
      <th scope="col">Region</th>
      <th scope="col">January</th>
      <th scope="col">February</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">North</th>
      <td>$10,000</td>
      <td>$12,000</td>
    </tr>
  </tbody>
</table>
```

**Key elements:**
- `<caption>`: Table title
- `<thead>`, `<tbody>`, `<tfoot>`: Table sections
- `<th>` with `scope`: Header cells
- `<td>`: Data cells
</tables>

<common_mistakes>
**Common Mistakes to Avoid**

```html
<!-- BAD: Div soup -->
<div class="header">
  <div class="nav">...</div>
</div>
<div class="main">
  <div class="article">...</div>
</div>

<!-- GOOD: Semantic elements -->
<header>
  <nav>...</nav>
</header>
<main>
  <article>...</article>
</main>

<!-- BAD: Using heading for styling -->
<h3>Not actually a heading, just want big text</h3>

<!-- GOOD: Use CSS for styling -->
<p class="large-text">Large styled text</p>

<!-- BAD: Div as button -->
<div onclick="submit()">Submit</div>

<!-- GOOD: Actual button -->
<button onclick="submit()">Submit</button>
```
</common_mistakes>

<checklist>
**Semantic HTML Checklist:**

- [ ] Using `<header>`, `<nav>`, `<main>`, `<footer>`
- [ ] One `<h1>` per page
- [ ] Heading hierarchy is logical (no skipped levels)
- [ ] Lists use `<ul>`, `<ol>`, or `<dl>`
- [ ] Buttons use `<button>`, not `<div>`
- [ ] Links use `<a>`, not `<span>` or `<div>`
- [ ] Forms have `<label>` for every input
- [ ] Related form fields grouped with `<fieldset>`
- [ ] Tables use `<th>` with `scope` for headers
- [ ] Images use `<figure>` and `<figcaption>` when appropriate
</checklist>
