<overview>
ARIA (Accessible Rich Internet Applications) landmarks help assistive technology users navigate pages. This reference covers landmarks, roles, and when to use ARIA.
</overview>

<first_rule>
**The First Rule of ARIA**

"If you can use a native HTML element or attribute with the semantics and behavior you require already built in, instead of re-purposing an element and adding an ARIA role, state or property to make it accessible, then do so."

**In practice:**
```html
<!-- DON'T: Using ARIA when native element exists -->
<div role="button" tabindex="0" onclick="submit()">Submit</div>

<!-- DO: Use native element -->
<button onclick="submit()">Submit</button>
```

**ARIA only communicates semantics. It doesn't add behavior.** A `<div role="button">` isn't keyboard accessible without additional JavaScript.
</first_rule>

<landmark_roles>
**Landmark Roles**

Landmarks allow screen reader users to jump between major page sections.

<landmark name="banner">
**banner**

Site header, typically containing logo and main navigation.

```html
<!-- Preferred: semantic element -->
<header>
  <nav>...</nav>
</header>

<!-- Alternative: ARIA role -->
<div role="banner">...</div>
```

**HTML equivalent:** `<header>` (when direct child of body)
**Use:** Once per page maximum
</landmark>

<landmark name="navigation">
**navigation**

Major navigation blocks.

```html
<!-- Preferred: semantic element -->
<nav aria-label="Main">...</nav>

<!-- Alternative: ARIA role -->
<div role="navigation" aria-label="Main">...</div>
```

**HTML equivalent:** `<nav>`
**Use:** Multiple allowed, label each uniquely
</landmark>

<landmark name="main">
**main**

Primary content of the page.

```html
<!-- Preferred: semantic element -->
<main id="main-content">...</main>

<!-- Alternative: ARIA role -->
<div role="main" id="main-content">...</div>
```

**HTML equivalent:** `<main>`
**Use:** Once per page only
</landmark>

<landmark name="complementary">
**complementary**

Supporting content, separate from main content.

```html
<!-- Preferred: semantic element -->
<aside>
  <h2>Related Articles</h2>
  ...
</aside>

<!-- Alternative: ARIA role -->
<div role="complementary">...</div>
```

**HTML equivalent:** `<aside>`
**Use:** Sidebars, related links, tangential content
</landmark>

<landmark name="contentinfo">
**contentinfo**

Site footer with copyright, contact info, etc.

```html
<!-- Preferred: semantic element -->
<footer>
  <p>&copy; 2025 Company</p>
</footer>

<!-- Alternative: ARIA role -->
<div role="contentinfo">...</div>
```

**HTML equivalent:** `<footer>` (when direct child of body)
**Use:** Once per page maximum
</landmark>

<landmark name="search">
**search**

Search functionality.

```html
<search>
  <form action="/search">
    <input type="search" name="q" aria-label="Search site">
    <button type="submit">Search</button>
  </form>
</search>

<!-- Or with role -->
<form role="search" action="/search">
  <input type="search" name="q" aria-label="Search site">
  <button type="submit">Search</button>
</form>
```

**HTML equivalent:** `<search>` (new in HTML5.2)
</landmark>

<landmark name="region">
**region**

Generic landmark (only when labeled).

```html
<section aria-labelledby="news-heading">
  <h2 id="news-heading">Latest News</h2>
  ...
</section>
```

**HTML equivalent:** `<section>` (when labeled with aria-label or aria-labelledby)
**Use:** Important sections that don't fit other landmarks
</landmark>

</landmark_roles>

<labeling_landmarks>
**Labeling Multiple Landmarks**

When you have multiple landmarks of the same type, label them to distinguish.

```html
<!-- Multiple navigation landmarks -->
<nav aria-label="Main">
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/products">Products</a></li>
  </ul>
</nav>

<nav aria-label="Footer">
  <ul>
    <li><a href="/privacy">Privacy</a></li>
    <li><a href="/terms">Terms</a></li>
  </ul>
</nav>

<!-- Using aria-labelledby -->
<section aria-labelledby="features-heading">
  <h2 id="features-heading">Features</h2>
  ...
</section>
```

**Screen reader announces:** "Main navigation", "Footer navigation"
</labeling_landmarks>

<aria_attributes>
**Common ARIA Attributes**

<attribute name="aria-label">
**aria-label**

Provides accessible name when visible text is absent.

```html
<button aria-label="Close">X</button>
<a href="https://twitter.com" aria-label="Follow us on Twitter">
  <svg>...</svg>
</a>
```
</attribute>

<attribute name="aria-labelledby">
**aria-labelledby**

References visible text as the accessible name.

```html
<h2 id="billing-heading">Billing Information</h2>
<section aria-labelledby="billing-heading">
  ...
</section>
```
</attribute>

<attribute name="aria-describedby">
**aria-describedby**

References additional descriptive text.

```html
<label for="password">Password</label>
<input type="password" id="password" aria-describedby="password-hint">
<p id="password-hint">Must be at least 8 characters</p>
```
</attribute>

<attribute name="aria-expanded">
**aria-expanded**

Indicates if a collapsible element is expanded or collapsed.

```html
<button aria-expanded="false" aria-controls="menu">Menu</button>
<ul id="menu" hidden>
  <li>Item 1</li>
  <li>Item 2</li>
</ul>
```
</attribute>

<attribute name="aria-hidden">
**aria-hidden**

Hides element from assistive technology (but visible on screen).

```html
<!-- Hide decorative icon -->
<button>
  <span aria-hidden="true">★</span>
  Add to favorites
</button>
```

**Warning:** Never use on focusable elements.
</attribute>

<attribute name="aria-live">
**aria-live**

Announces dynamic content changes.

```html
<!-- Polite: waits for pause in user activity -->
<div aria-live="polite">
  <!-- Status messages appear here -->
</div>

<!-- Assertive: interrupts immediately -->
<div aria-live="assertive" role="alert">
  <!-- Error messages appear here -->
</div>
```
</attribute>

<attribute name="aria-current">
**aria-current**

Indicates current item in a set (like navigation).

```html
<nav>
  <a href="/">Home</a>
  <a href="/products" aria-current="page">Products</a>
  <a href="/about">About</a>
</nav>
```
</attribute>

</aria_attributes>

<widget_roles>
**Widget Roles (When Native Elements Don't Exist)**

Use these only when no native HTML element exists:

```html
<!-- Tabs -->
<div role="tablist">
  <button role="tab" aria-selected="true" aria-controls="panel1">Tab 1</button>
  <button role="tab" aria-selected="false" aria-controls="panel2">Tab 2</button>
</div>
<div role="tabpanel" id="panel1">Content 1</div>
<div role="tabpanel" id="panel2" hidden>Content 2</div>

<!-- Dialog/Modal -->
<div role="dialog" aria-labelledby="dialog-title" aria-modal="true">
  <h2 id="dialog-title">Dialog Title</h2>
  ...
</div>

<!-- Alert -->
<div role="alert">Error: Please correct the form.</div>

<!-- Status -->
<div role="status" aria-live="polite">Form saved successfully.</div>
```
</widget_roles>

<page_template>
**Complete Page Landmark Structure**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Page Title</title>
</head>
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>

  <header>
    <nav aria-label="Main">
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
      </ul>
    </nav>

    <search>
      <form action="/search">
        <input type="search" aria-label="Search site">
        <button type="submit">Search</button>
      </form>
    </search>
  </header>

  <main id="main-content">
    <h1>Page Title</h1>

    <article>
      <h2>Article Title</h2>
      <p>Content...</p>
    </article>

    <section aria-labelledby="features-heading">
      <h2 id="features-heading">Features</h2>
      <p>More content...</p>
    </section>
  </main>

  <aside>
    <h2>Related Links</h2>
    <ul>...</ul>
  </aside>

  <footer>
    <nav aria-label="Footer">
      <a href="/privacy">Privacy</a>
      <a href="/terms">Terms</a>
    </nav>
    <p>&copy; 2025 Company</p>
  </footer>
</body>
</html>
```
</page_template>

<testing_landmarks>
**Testing Landmarks**

**Screen reader navigation:**
- VoiceOver: Ctrl + Option + U → Landmarks rotor
- NVDA: D key to jump between landmarks
- JAWS: R key for regions

**Browser extensions:**
- Landmarks extension (shows page landmarks)
- WAVE (highlights regions)
- axe DevTools (validates landmark usage)
</testing_landmarks>

<checklist>
**Landmarks Checklist:**

- [ ] Page has `<header>` with `<nav>`
- [ ] Page has exactly one `<main>`
- [ ] Page has `<footer>`
- [ ] Multiple navs have unique labels
- [ ] Sections have headings or aria-labelledby
- [ ] No redundant ARIA roles on semantic elements
- [ ] Landmarks don't overlap or nest incorrectly
- [ ] Skip link targets `<main>`
</checklist>
