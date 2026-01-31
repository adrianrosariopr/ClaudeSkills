<overview>
Image optimization impacts both SEO (page speed, image search) and accessibility (alt text). Well-optimized images improve Core Web Vitals and user experience.
</overview>

<why_optimize>
**Why Image Optimization Matters**

- Images account for ~50% of page weight on average
- Unoptimized images hurt LCP (Largest Contentful Paint)
- Missing dimensions cause CLS (Cumulative Layout Shift)
- Alt text enables image search and accessibility
- Fast-loading images reduce bounce rates
</why_optimize>

<format_selection>

<format name="webp">
**WebP (Recommended for most images)**

- 25-34% smaller than JPEG at same quality
- Supports transparency (like PNG)
- Supported by all modern browsers

```html
<picture>
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Description">
</picture>
```
</format>

<format name="avif">
**AVIF (Best compression, newer)**

- 50% smaller than JPEG
- Better than WebP for photos
- Growing browser support

```html
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Description">
</picture>
```
</format>

<format name="jpeg">
**JPEG (Photos, fallback)**

- Good for photographs
- No transparency support
- Universal browser support
- Use quality 80-85 for web
</format>

<format name="png">
**PNG (Graphics with transparency)**

- Lossless compression
- Supports transparency
- Large file sizes for photos
- Best for logos, icons, graphics
</format>

<format name="svg">
**SVG (Vector graphics)**

- Infinitely scalable
- Small file sizes for simple graphics
- Perfect for logos and icons
- Can be styled with CSS

```html
<img src="logo.svg" alt="Company Logo">
```
</format>

<decision_tree>
**Format Decision Tree:**

```
Is it a photo/complex image?
├── Yes → Use AVIF with WebP and JPEG fallback
└── No → Does it need transparency?
    ├── Yes → Is it simple (logo, icon)?
    │   ├── Yes → Use SVG
    │   └── No → Use PNG or WebP
    └── No → Use WebP with JPEG fallback
```
</decision_tree>

</format_selection>

<compression>
**Image Compression**

**Tools:**
- **TinyPNG/TinyJPG:** Simple web-based compression
- **Squoosh:** Google's advanced compression tool
- **ImageOptim (Mac):** Desktop batch processing
- **Sharp (Node.js):** Programmatic optimization

**Target file sizes:**
- Hero images: < 200KB
- Content images: < 100KB
- Thumbnails: < 30KB

**Command line example (Sharp):**
```javascript
const sharp = require('sharp');

sharp('input.jpg')
  .resize(800, 600)
  .webp({ quality: 80 })
  .toFile('output.webp');
```
</compression>

<dimensions>
**Always Specify Dimensions**

Prevents CLS (Cumulative Layout Shift).

```html
<!-- Always include width and height -->
<img src="photo.jpg" width="800" height="600" alt="Description">

<!-- Or use CSS aspect-ratio -->
<style>
.image-container {
  aspect-ratio: 16 / 9;
}
.image-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>
```

**Why it matters:**
- Browser reserves space before image loads
- No layout shift when image appears
- Critical for passing CLS threshold
</dimensions>

<responsive_images>
**Responsive Images**

Serve different sizes for different screen sizes.

**srcset and sizes:**
```html
<img srcset="photo-400.jpg 400w,
             photo-800.jpg 800w,
             photo-1200.jpg 1200w"
     sizes="(max-width: 600px) 400px,
            (max-width: 1200px) 800px,
            1200px"
     src="photo-800.jpg"
     alt="Description"
     width="800"
     height="600">
```

**picture element for art direction:**
```html
<picture>
  <source media="(max-width: 600px)" srcset="photo-mobile.jpg">
  <source media="(max-width: 1200px)" srcset="photo-tablet.jpg">
  <img src="photo-desktop.jpg" alt="Description">
</picture>
```
</responsive_images>

<lazy_loading>
**Lazy Loading**

Defer loading images until they're needed.

**Native lazy loading:**
```html
<!-- Above the fold: load immediately -->
<img src="hero.jpg" alt="Hero" fetchpriority="high">

<!-- Below the fold: lazy load -->
<img src="photo.jpg" alt="Description" loading="lazy">
```

**Rules:**
- **Never lazy load LCP image** (above-fold hero)
- Lazy load all below-fold images
- Can improve LCP by 15-30% on image-heavy pages
- Native `loading="lazy"` has good browser support
</lazy_loading>

<alt_text>
**Alt Text for SEO and Accessibility**

Alt text serves both search engines and screen reader users.

**Guidelines:**
```html
<!-- Informative image: describe content -->
<img src="chart.png" alt="Bar chart showing 25% revenue growth in Q4 2024">

<!-- Functional image: describe action -->
<img src="search-icon.svg" alt="Search">

<!-- Decorative image: empty alt -->
<img src="decorative-line.png" alt="">

<!-- Image as link: describe destination -->
<a href="/products">
  <img src="product.jpg" alt="View our product catalog">
</a>
```

**Writing good alt text:**
- Describe the image's purpose, not just appearance
- Include relevant keywords naturally (not stuffed)
- Keep under 125 characters
- Don't start with "Image of" or "Picture of"
- For complex images, use adjacent text description
</alt_text>

<file_naming>
**Descriptive File Names**

File names are a minor SEO signal.

```
BAD:  IMG_20240115_123456.jpg
BAD:  photo1.jpg
BAD:  DSC00234.jpg

GOOD: blue-widget-product-photo.jpg
GOOD: core-web-vitals-chart.png
GOOD: team-meeting-office.jpg
```

**Guidelines:**
- Use hyphens, not underscores
- Include descriptive keywords
- Keep reasonably short
- Use lowercase
</file_naming>

<lcp_optimization>
**LCP Image Optimization**

The LCP element is often an image. Optimize it specifically.

```html
<!-- Preload LCP image -->
<link rel="preload" as="image" href="hero.webp" fetchpriority="high">

<!-- Mark as high priority -->
<img src="hero.webp"
     alt="Hero description"
     width="1200"
     height="600"
     fetchpriority="high"
     decoding="async">
```

**LCP image checklist:**
- [ ] Preloaded in head
- [ ] Optimized format (WebP/AVIF)
- [ ] Compressed appropriately
- [ ] Correct dimensions specified
- [ ] fetchpriority="high"
- [ ] Not lazy loaded
- [ ] Served from CDN
</lcp_optimization>

<framework_examples>

<framework name="next">
**Next.js Image Optimization**

```jsx
import Image from 'next/image';

// Automatic optimization
<Image
  src="/hero.jpg"
  alt="Hero description"
  width={1200}
  height={600}
  priority  // For LCP image
/>

// Responsive
<Image
  src="/photo.jpg"
  alt="Description"
  fill
  sizes="(max-width: 768px) 100vw, 50vw"
/>
```
</framework>

<framework name="nuxt">
**Nuxt Image Optimization**

```vue
<NuxtImg
  src="/hero.jpg"
  alt="Hero description"
  width="1200"
  height="600"
  loading="eager"
  format="webp"
/>

<NuxtPicture
  src="/photo.jpg"
  alt="Description"
  sizes="sm:100vw md:50vw"
/>
```
</framework>

</framework_examples>

<checklist>
**Image Optimization Checklist:**

**Format & Compression:**
- [ ] Using WebP or AVIF with fallbacks
- [ ] Images compressed (< 200KB for hero, < 100KB for content)
- [ ] SVG for logos and simple graphics

**Dimensions & Layout:**
- [ ] Width and height attributes on all images
- [ ] Responsive images with srcset/sizes
- [ ] Aspect ratio preserved

**Loading:**
- [ ] LCP image preloaded and high priority
- [ ] Below-fold images use loading="lazy"
- [ ] Images served from CDN

**Accessibility & SEO:**
- [ ] All informative images have alt text
- [ ] Decorative images have alt=""
- [ ] Descriptive file names
- [ ] Alt text includes relevant keywords naturally
</checklist>
