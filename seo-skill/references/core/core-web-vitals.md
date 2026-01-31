<overview>
Core Web Vitals are Google's metrics for measuring user experience. They're confirmed ranking factors and critical for SEO performance. As of March 2024, the three metrics are LCP, INP, and CLS.
</overview>

<metrics>

<metric name="LCP">
**Largest Contentful Paint**

**What it measures:** Loading performance - how long until the largest content element is visible.

**Thresholds:**
| Rating | Time |
|--------|------|
| Good | ≤ 2.5s |
| Needs Improvement | 2.5s - 4s |
| Poor | > 4s |

**What counts as LCP:**
- `<img>` elements
- `<video>` poster images (or first frame)
- Elements with `background-image` (CSS)
- Block-level elements containing text

**Common causes of poor LCP:**
- Slow server response time
- Render-blocking CSS/JS
- Slow resource load times
- Client-side rendering

**Optimization strategies:**

1. **Optimize server response:**
```
- Use CDN
- Cache HTML pages
- Optimize database queries
- Use HTTP/2 or HTTP/3
```

2. **Preload critical resources:**
```html
<!-- Preload hero image -->
<link rel="preload" as="image" href="hero.webp">

<!-- Preload critical fonts -->
<link rel="preload" as="font" href="font.woff2" crossorigin>
```

3. **Optimize images:**
```html
<!-- Use modern formats -->
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="...">
</picture>

<!-- Prioritize LCP image -->
<img src="hero.webp" fetchpriority="high" alt="...">
```

4. **Eliminate render-blocking resources:**
```html
<!-- Defer non-critical JS -->
<script src="app.js" defer></script>

<!-- Inline critical CSS -->
<style>/* Critical styles */</style>
<link rel="stylesheet" href="styles.css" media="print" onload="this.media='all'">
```
</metric>

<metric name="INP">
**Interaction to Next Paint**

**What it measures:** Responsiveness - how long from user interaction to visual feedback.

**Replaced FID (First Input Delay) in March 2024.**

**Thresholds:**
| Rating | Time |
|--------|------|
| Good | ≤ 200ms |
| Needs Improvement | 200ms - 500ms |
| Poor | > 500ms |

**Key differences from FID:**
- FID measured only first interaction
- INP measures ALL interactions throughout page lifecycle
- Reports the worst interaction (or near-worst for pages with many interactions)

**What counts as an interaction:**
- Clicks
- Taps
- Key presses

**Does NOT count:**
- Scrolling
- Hovering

**Common causes of poor INP:**
- Long JavaScript tasks blocking main thread
- Large DOM size
- Heavy event handlers
- Forced synchronous layouts

**Optimization strategies:**

1. **Break up long tasks:**
```javascript
// BAD: Long blocking task
function processAllData(data) {
  data.forEach(item => heavyProcessing(item));
}

// GOOD: Yield to main thread
async function processAllData(data) {
  for (const item of data) {
    heavyProcessing(item);
    await scheduler.yield(); // or setTimeout(0)
  }
}
```

2. **Optimize event handlers:**
```javascript
// BAD: Heavy work in handler
button.onclick = () => {
  // 500ms of processing...
  updateUI();
};

// GOOD: Immediate feedback, defer work
button.onclick = () => {
  button.textContent = 'Processing...';
  requestAnimationFrame(() => {
    // Heavy work on next frame
    heavyProcessing();
  });
};
```

3. **Reduce DOM size:**
- Keep under 1,500 DOM elements
- Avoid deeply nested structures
- Use virtual scrolling for long lists

4. **Minimize main thread work:**
```javascript
// Use Web Workers for heavy computation
const worker = new Worker('heavy-task.js');
worker.postMessage(data);
worker.onmessage = (e) => updateUI(e.data);
```
</metric>

<metric name="CLS">
**Cumulative Layout Shift**

**What it measures:** Visual stability - how much content unexpectedly shifts during loading.

**Thresholds:**
| Rating | Score |
|--------|-------|
| Good | ≤ 0.1 |
| Needs Improvement | 0.1 - 0.25 |
| Poor | > 0.25 |

**How it's calculated:**
```
Layout Shift Score = Impact Fraction × Distance Fraction
```
- Impact Fraction: % of viewport affected
- Distance Fraction: How far elements moved

**Common causes of poor CLS:**
- Images/videos without dimensions
- Ads or embeds without reserved space
- Dynamically injected content
- Web fonts causing FOIT/FOUT
- Content inserted above existing content

**Optimization strategies:**

1. **Always specify image/video dimensions:**
```html
<!-- GOOD: Dimensions specified -->
<img src="photo.jpg" width="800" height="600" alt="...">
<video width="640" height="360">...</video>
<iframe width="560" height="315">...</iframe>

<!-- GOOD: Aspect ratio box -->
<div style="aspect-ratio: 16/9;">
  <img src="photo.jpg" alt="..." style="width: 100%; height: 100%;">
</div>
```

2. **Reserve space for dynamic content:**
```css
/* Reserve space for ads */
.ad-slot {
  min-height: 250px;
  background: #f0f0f0;
}

/* Reserve space for lazy-loaded content */
.lazy-container {
  min-height: 300px;
}
```

3. **Avoid inserting content above existing content:**
```css
/* BAD: Banner pushes content down */
.notification-banner {
  position: relative; /* Causes shift */
}

/* GOOD: Overlay doesn't shift layout */
.notification-banner {
  position: fixed;
  top: 0;
}
```

4. **Optimize web fonts:**
```css
/* Use font-display to control FOIT/FOUT */
@font-face {
  font-family: 'CustomFont';
  src: url('font.woff2') format('woff2');
  font-display: swap; /* Show fallback immediately */
}

/* Or preload critical fonts */
<link rel="preload" as="font" href="font.woff2" crossorigin>
```

5. **Handle images without dimensions:**
```css
/* Fallback for images without explicit dimensions */
img {
  max-width: 100%;
  height: auto;
}
```
</metric>

</metrics>

<measuring_tools>

<tool name="field-data">
**Real User Monitoring (RUM)**

Best source of truth - actual user experiences.

**Sources:**
- Google Search Console (Core Web Vitals report)
- Chrome User Experience Report (CrUX)
- PageSpeed Insights (field data section)

**Threshold for "passing":** 75% of page loads must meet "Good" threshold.
</tool>

<tool name="lab-data">
**Simulated Testing**

Useful for debugging but doesn't reflect real user conditions.

**Tools:**
- Lighthouse (Chrome DevTools)
- PageSpeed Insights (lab data section)
- WebPageTest

**Limitations:**
- Single device/connection simulation
- No real user variability
- INP requires real interactions
</tool>

<tool name="debugging">
**Chrome DevTools Performance Panel:**
1. Open DevTools → Performance
2. Click Record → Interact with page → Stop
3. Analyze flame chart for long tasks
4. Check Core Web Vitals in summary

**Web Vitals Extension:**
- Real-time CWV display
- Click for detailed breakdown
- Shows actual field-like data
</tool>

</measuring_tools>

<framework_specific>

<framework name="react-next">
**Next.js optimizations:**
```jsx
// Use next/image for automatic optimization
import Image from 'next/image';

<Image
  src="/hero.jpg"
  width={800}
  height={600}
  priority  // For LCP image
  alt="Hero"
/>

// Use next/font for font optimization
import { Inter } from 'next/font/google';
const inter = Inter({ subsets: ['latin'] });
```
</framework>

<framework name="vue-nuxt">
**Nuxt optimizations:**
```vue
<!-- Use NuxtImg for automatic optimization -->
<NuxtImg
  src="/hero.jpg"
  width="800"
  height="600"
  loading="eager"
  alt="Hero"
/>

<!-- Prefetch critical resources -->
<Link rel="preload" as="image" :href="heroImage" />
```
</framework>

</framework_specific>

<quick_wins>
**Immediate improvements:**

1. **Add dimensions to all images/videos** (CLS)
2. **Preload LCP image** (LCP)
3. **Defer non-critical JavaScript** (LCP, INP)
4. **Use `font-display: swap`** (CLS)
5. **Compress images to WebP** (LCP)
6. **Use a CDN** (LCP)
7. **Reserve space for ads/embeds** (CLS)
8. **Avoid legacy DOM manipulation methods** (all metrics)
</quick_wins>

<checklist>
**Core Web Vitals Optimization Checklist:**

**LCP (Loading):**
- [ ] Server response time < 200ms
- [ ] LCP image preloaded
- [ ] Critical CSS inlined
- [ ] JS deferred or async
- [ ] Images optimized (WebP, compressed)
- [ ] CDN in use

**INP (Interactivity):**
- [ ] No long tasks (> 50ms)
- [ ] Event handlers are fast
- [ ] DOM size < 1,500 elements
- [ ] Heavy work offloaded to Web Workers
- [ ] Immediate visual feedback on interactions

**CLS (Visual Stability):**
- [ ] All images have width/height
- [ ] All videos/iframes have dimensions
- [ ] Space reserved for ads/embeds
- [ ] Fonts use font-display: swap
- [ ] No content inserted above fold
- [ ] Animations use transform/opacity only
</checklist>
