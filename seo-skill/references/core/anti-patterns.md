<overview>
Common mistakes in SEO and accessibility that hurt rankings, user experience, and legal compliance. Avoid these patterns.
</overview>

<seo_anti_patterns>

<anti_pattern name="keyword-stuffing">
**Problem:** Overusing keywords unnaturally to manipulate rankings.

```html
<!-- BAD -->
<h1>Best SEO Services - SEO Company - SEO Agency - SEO Experts</h1>
<p>Our SEO services provide the best SEO for your SEO needs.
   Contact our SEO team for SEO optimization and SEO results.</p>

<!-- GOOD -->
<h1>Professional Search Engine Optimization Services</h1>
<p>Our team helps businesses improve their search visibility
   through technical optimization, content strategy, and link building.</p>
```

**Why it's bad:**
- Google penalizes keyword stuffing
- Poor user experience
- Looks spammy to visitors
</anti_pattern>

<anti_pattern name="duplicate-content">
**Problem:** Same content appearing at multiple URLs.

**Common causes:**
- WWW vs non-WWW versions
- HTTP vs HTTPS versions
- Trailing slash variations
- URL parameters
- Printer-friendly pages

**Fix:** Use canonical tags.
```html
<link rel="canonical" href="https://example.com/page">
```
</anti_pattern>

<anti_pattern name="blocking-crawlers">
**Problem:** Accidentally preventing search engines from accessing your site.

```
# BAD: Blocking everything
User-agent: *
Disallow: /

# BAD: Blocking CSS/JS (breaks rendering)
Disallow: /css/
Disallow: /js/

# GOOD: Block only private areas
User-agent: *
Disallow: /admin/
Disallow: /api/
Allow: /
```

**Also check:**
- `noindex` meta tags on important pages
- X-Robots-Tag HTTP headers
- Accidental password protection
</anti_pattern>

<anti_pattern name="thin-content">
**Problem:** Pages with little or no valuable content.

**Examples:**
- Pages under 300 words with no real value
- Doorway pages targeting variations
- Auto-generated content
- Scraped content from other sites

**Fix:** Create comprehensive, original content that satisfies user intent.
</anti_pattern>

<anti_pattern name="ignoring-mobile">
**Problem:** Designing primarily for desktop.

**Issues:**
- Text too small to read
- Touch targets too close together
- Horizontal scrolling required
- Popups covering content
- Slow mobile load times

**Why it's bad:** Google uses mobile-first indexing. Your mobile experience determines rankings.
</anti_pattern>

<anti_pattern name="slow-page-speed">
**Problem:** Pages that take too long to load.

**Common causes:**
- Unoptimized images (use WebP, compress)
- Render-blocking CSS/JS
- No caching
- Too many HTTP requests
- No CDN
- Large JavaScript bundles

**Impact:**
- Core Web Vitals failure
- Lower rankings
- Higher bounce rates
</anti_pattern>

<anti_pattern name="buying-links">
**Problem:** Paying for backlinks to manipulate rankings.

**What Google considers paid links:**
- Exchanging money for links
- Exchanging goods/services for links
- Excessive link exchanges
- Large-scale guest posting campaigns for links

**Consequences:**
- Manual penalty
- Algorithmic demotion
- Potential de-indexing
</anti_pattern>

<anti_pattern name="cloaking">
**Problem:** Showing different content to search engines than to users.

**Examples:**
- Showing keyword-stuffed content to bots
- Redirecting users but not bots
- Hidden text visible only to crawlers

**Consequence:** Severe penalties, potential de-indexing.
</anti_pattern>

<anti_pattern name="orphan-pages">
**Problem:** Pages with no internal links pointing to them.

**Why it's bad:**
- Crawlers may never find them
- No PageRank flows to them
- Users can't discover them through navigation

**Fix:** Ensure every page has at least one internal link from another page.
</anti_pattern>

</seo_anti_patterns>

<accessibility_anti_patterns>

<anti_pattern name="removing-focus-outlines">
**Problem:** Hiding focus indicators for aesthetic reasons.

```css
/* BAD: Removes focus for keyboard users */
:focus {
  outline: none;
}

*:focus {
  outline: 0;
}

/* GOOD: Custom but visible focus */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

**Why it's bad:** Keyboard users can't see where they are on the page.
</anti_pattern>

<anti_pattern name="click-here-links">
**Problem:** Using generic link text.

```html
<!-- BAD -->
<p>For more information, <a href="/pricing">click here</a>.</p>
<p>Read more about our services <a href="/services">here</a>.</p>

<!-- GOOD -->
<p>View our <a href="/pricing">pricing plans</a> for more information.</p>
<p><a href="/services">Explore our services</a> to learn more.</p>
```

**Why it's bad:** Screen reader users navigate by links. "Click here" is meaningless out of context.
</anti_pattern>

<anti_pattern name="missing-alt-text">
**Problem:** Images without alternative text.

```html
<!-- BAD: Missing alt -->
<img src="hero.jpg">

<!-- BAD: Useless alt -->
<img src="hero.jpg" alt="image">
<img src="hero.jpg" alt="photo.jpg">

<!-- GOOD: Descriptive alt -->
<img src="hero.jpg" alt="Team collaborating on a whiteboard">

<!-- GOOD: Decorative image -->
<img src="divider.png" alt="">
```

**Why it's bad:** Screen reader users have no idea what the image conveys.
</anti_pattern>

<anti_pattern name="div-buttons">
**Problem:** Using divs or spans as interactive elements.

```html
<!-- BAD: Not keyboard accessible -->
<div class="button" onclick="submit()">Submit</div>
<span onclick="openMenu()">Menu</span>

<!-- GOOD: Native elements -->
<button type="submit">Submit</button>
<button type="button" aria-expanded="false">Menu</button>
```

**Why it's bad:**
- Not focusable by default
- Not announced as interactive
- Enter/Space don't work
- No native behaviors
</anti_pattern>

<anti_pattern name="color-only-information">
**Problem:** Using color as the only way to convey information.

```html
<!-- BAD: Color only -->
<p>Required fields are in red.</p>
<input style="border-color: red">

<!-- GOOD: Color + text/icon -->
<p>Required fields are marked with *</p>
<label>
  Name <span aria-hidden="true" style="color: red">*</span>
  <span class="sr-only">(required)</span>
</label>
<input required aria-required="true">
```

**Why it's bad:** Color blind users (8% of men) can't distinguish colors.
</anti_pattern>

<anti_pattern name="auto-playing-media">
**Problem:** Audio or video that plays automatically.

```html
<!-- BAD -->
<video autoplay>
<audio autoplay>

<!-- GOOD: Muted autoplay (if necessary) -->
<video autoplay muted>

<!-- BEST: User-initiated playback -->
<video controls>
```

**Why it's bad:**
- Disrupts screen reader users
- Triggers unexpected audio
- Bad UX for everyone
</anti_pattern>

<anti_pattern name="using-overlay-widgets">
**Problem:** Using accessibility overlay tools as a compliance solution.

**Popular overlays:** AccessiBe, AudioEye, UserWay, etc.

**Why they don't work:**
- Can't fix underlying code issues
- Often break screen reader compatibility
- Don't provide legal protection
- Have been named in ADA lawsuits
- Disability community actively opposes them

**Real solution:** Fix the actual accessibility issues in your code.
</anti_pattern>

<anti_pattern name="skipping-heading-levels">
**Problem:** Not following logical heading hierarchy.

```html
<!-- BAD: Skipped levels -->
<h1>Page Title</h1>
<h3>First Section</h3>  <!-- Should be h2 -->
<h4>Subsection</h4>     <!-- Should be h3 -->

<!-- BAD: Multiple h1s -->
<h1>Main Title</h1>
<h1>Another Section</h1>

<!-- GOOD: Proper hierarchy -->
<h1>Page Title</h1>
<h2>First Section</h2>
<h3>Subsection</h3>
<h2>Second Section</h2>
```

**Why it's bad:** Screen reader users navigate by headings. Skipped levels are confusing.
</anti_pattern>

<anti_pattern name="form-without-labels">
**Problem:** Form inputs without associated labels.

```html
<!-- BAD: Placeholder only -->
<input type="email" placeholder="Email">

<!-- BAD: Visual label not associated -->
<span>Email</span>
<input type="email">

<!-- GOOD: Explicit label -->
<label for="email">Email address</label>
<input type="email" id="email">

<!-- GOOD: Implicit label -->
<label>
  Email address
  <input type="email">
</label>
```

**Why it's bad:**
- Screen readers don't announce field purpose
- Clicking label doesn't focus input
- Placeholder disappears when typing
</anti_pattern>

<anti_pattern name="keyboard-traps">
**Problem:** Focus gets stuck and users can't escape with keyboard.

**Common causes:**
- Modal dialogs without escape handling
- Custom widgets that capture focus
- Infinite tab loops

**Fix:**
```javascript
// Handle escape key on modals
modal.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') {
    closeModal();
    returnFocusToTrigger();
  }
});
```
</anti_pattern>

</accessibility_anti_patterns>

<combined_anti_patterns>

<anti_pattern name="hidden-text-for-seo">
**Problem:** Hiding text to stuff keywords while keeping page "clean."

```css
/* BAD: Hidden text for SEO */
.seo-keywords {
  color: white;
  background: white;
  font-size: 0;
  position: absolute;
  left: -9999px;
}
```

**Why it's bad:**
- Google considers this spam (penalty risk)
- Screen readers may read it (confusing)
- Provides no real value
</anti_pattern>

<anti_pattern name="infinite-scroll-without-pagination">
**Problem:** Using infinite scroll with no alternative navigation.

**Issues:**
- Crawlers can't access all content
- Users can't bookmark/share specific items
- Keyboard users can't reach footer
- Back button breaks

**Fix:**
- Provide paginated alternative
- Use View Transitions API for seamless UX
- Ensure footer remains accessible
</anti_pattern>

</combined_anti_patterns>

<checklist>
**Never do these:**

**SEO:**
- [ ] Keyword stuff content
- [ ] Buy or exchange links
- [ ] Duplicate content without canonicals
- [ ] Block crawlers from CSS/JS
- [ ] Create thin/doorway pages
- [ ] Cloak content

**Accessibility:**
- [ ] Remove focus outlines
- [ ] Use "click here" links
- [ ] Skip alt text on images
- [ ] Use divs as buttons
- [ ] Rely on color alone
- [ ] Auto-play audio/video
- [ ] Use overlay widgets
- [ ] Skip heading levels
- [ ] Create forms without labels
- [ ] Trap keyboard focus
</checklist>
