<overview>
Frontend performance optimization (2025): Core Web Vitals, bundle optimization, rendering performance, and measurement tools. Measure first, optimize second.
</overview>

<core_web_vitals>
**The metrics that matter:**

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| LCP (Largest Contentful Paint) | < 2.5s | 2.5s - 4s | > 4s |
| INP (Interaction to Next Paint) | < 200ms | 200ms - 500ms | > 500ms |
| CLS (Cumulative Layout Shift) | < 0.1 | 0.1 - 0.25 | > 0.25 |

**LCP - How fast does main content appear?**
- Optimize images (format, size, lazy loading)
- Preload critical resources
- Use Server Components for initial HTML
- Minimize render-blocking resources

**INP - How responsive are interactions?**
- Break up long tasks (< 50ms chunks)
- Use `requestIdleCallback` for non-critical work
- Optimize event handlers
- Reduce JavaScript execution time

**CLS - Does content shift unexpectedly?**
- Set explicit dimensions on images/videos
- Reserve space for dynamic content
- Avoid inserting content above existing content
- Use `transform` for animations (not layout properties)
</core_web_vitals>

<bundle_optimization>
**Code splitting:**
```tsx
// Route-based splitting (Next.js does this automatically)
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

**Dynamic imports for heavy libraries:**
```tsx
// Don't: Import at top level
import { Chart } from 'chart.js';

// Do: Import when needed
async function renderChart(canvas, data) {
  const { Chart } = await import('chart.js');
  new Chart(canvas, { type: 'line', data });
}

// With React
function ChartComponent({ data }) {
  const [Chart, setChart] = useState(null);

  useEffect(() => {
    import('chart.js').then(mod => setChart(() => mod.Chart));
  }, []);

  if (!Chart) return <ChartSkeleton />;
  return <Chart data={data} />;
}
```

**Tree shaking:**
```tsx
// Bad: Imports entire library
import _ from 'lodash';
_.debounce(fn, 300);

// Good: Named import (tree-shakeable)
import { debounce } from 'lodash-es';
debounce(fn, 300);

// Best: Individual module import
import debounce from 'lodash/debounce';
debounce(fn, 300);
```

**Bundle analysis:**
```bash
# Webpack
npm install --save-dev webpack-bundle-analyzer
npx webpack-bundle-analyzer stats.json

# Next.js
npm install --save-dev @next/bundle-analyzer
# Add to next.config.js and run: ANALYZE=true npm run build

# Vite
npm install --save-dev rollup-plugin-visualizer
```
</bundle_optimization>

<server_components>
**Zero client-side JavaScript:**
```tsx
// Server Component - no JS shipped
async function ProductList() {
  const products = await db.products.findMany();

  return (
    <ul>
      {products.map(p => (
        <li key={p.id}>
          <h3>{p.name}</h3>
          <p>${p.price}</p>
        </li>
      ))}
    </ul>
  );
}

// Only interactive parts need client JS
'use client';
function AddToCartButton({ productId }) {
  return <button onClick={() => addToCart(productId)}>Add</button>;
}
```

**Benefits:**
- Smaller bundle size (less JS to download)
- Faster initial load (HTML streamed from server)
- Better SEO (content in initial HTML)
- Direct database access (no API layer needed)
</server_components>

<render_optimization>
**React Compiler handles most memoization, but avoid these patterns:**

```tsx
// Bad: New object every render
<Component style={{ color: 'red' }} />

// Good: Stable reference
const style = { color: 'red' };  // Outside component
<Component style={style} />

// Or use Tailwind (no object)
<Component className="text-red-500" />

// Bad: Inline function creating new reference
<Child onClick={() => handleClick(id)} />

// Good: Stable callback (React Compiler usually handles this)
const handleChildClick = () => handleClick(id);
<Child onClick={handleChildClick} />
```

**When to use manual memoization:**
```tsx
// Expensive computation
const sortedData = useMemo(
  () => data.sort((a, b) => complexCompare(a, b)),
  [data]
);

// Reference stability for third-party libs
const chartOptions = useMemo(() => ({
  responsive: true,
  plugins: { legend: { position: 'top' } },
}), []);
```
</render_optimization>

<list_virtualization>
**For lists with 100+ items:**

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,  // Estimated row height
    overscan: 5,  // Render 5 extra items outside viewport
  });

  return (
    <div
      ref={parentRef}
      style={{ height: '400px', overflow: 'auto' }}
    >
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```
</list_virtualization>

<image_optimization>
**Next.js Image:**
```tsx
import Image from 'next/image';

// Above the fold - preload
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>

// Below the fold - lazy load (default)
<Image
  src="/feature.jpg"
  alt="Feature"
  width={600}
  height={400}
  loading="lazy"
/>

// Fill container
<div className="relative aspect-video">
  <Image
    src="/cover.jpg"
    alt="Cover"
    fill
    className="object-cover"
    sizes="(max-width: 768px) 100vw, 50vw"
  />
</div>
```

**Without Next.js:**
```html
<!-- Responsive images -->
<img
  src="image-800.jpg"
  srcset="
    image-400.jpg 400w,
    image-800.jpg 800w,
    image-1200.jpg 1200w
  "
  sizes="(max-width: 600px) 100vw, 50vw"
  loading="lazy"
  decoding="async"
  alt="Description"
/>

<!-- Modern formats with fallback -->
<picture>
  <source srcset="image.avif" type="image/avif" />
  <source srcset="image.webp" type="image/webp" />
  <img src="image.jpg" alt="Description" loading="lazy" />
</picture>
```
</image_optimization>

<laravel_react_image_optimization>
**Complete pattern for Laravel + Inertia.js + React + S3 (proven 68% LCP improvement)**

This pattern reduced LCP from 20.8s to 6.7s on a TCG card collection app with thousands of images.

**Problem:** Images stored at full resolution (733x1024) but displayed at ~200x280. Browser downloads 3-4x more data than needed.

**Solution architecture:**
```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────┐
│ Laravel Backend │────▶│ S3 Storage       │────▶│ React Front │
│ (generates      │     │ card.webp (733w) │     │ (uses srcSet│
│  variants)      │     │ card-md.webp(400)│     │  for sizes) │
│                 │     │ card-sm.webp(200)│     │             │
└─────────────────┘     └──────────────────┘     └─────────────┘
```

**1. Backend: Image service with responsive variants (Laravel/PHP)**
```php
// app/Services/ImageConversionService.php
class ImageConversionService
{
    // Define responsive breakpoints
    public const RESPONSIVE_SIZES = [
        ''     => [733, 85],  // Original/large: width, quality
        '-md'  => [400, 80],  // Medium
        '-sm'  => [200, 75],  // Small/thumbnail
    ];

    public function generateResponsiveVariants(string $s3Path): array
    {
        // Download original from S3
        $originalContent = Storage::disk('s3')->get($s3Path);
        $image = Image::read($originalContent);

        $basePath = preg_replace('/\.webp$/', '', $s3Path);
        $results = [];

        foreach (self::RESPONSIVE_SIZES as $suffix => [$width, $quality]) {
            $variantPath = $basePath . $suffix . '.webp';

            // Skip if variant already exists
            if (Storage::disk('s3')->exists($variantPath)) {
                continue;
            }

            // Create resized variant
            $resized = (clone $image)->scaleDown(width: $width);
            $webpContent = $resized->toWebp(quality: $quality)->toString();

            Storage::disk('s3')->put($variantPath, $webpContent, [
                'visibility' => 'public',
                'ContentType' => 'image/webp',
                'CacheControl' => 'public, max-age=31536000',
            ]);

            $results[$suffix] = $variantPath;
        }

        return ['success' => true, 'variants' => $results];
    }
}
```

**2. Batch processing with parallel workers (Laravel Command + Job)**
```php
// app/Console/Commands/GenerateResponsiveImages.php
class GenerateResponsiveImages extends Command
{
    protected $signature = 'images:generate-responsive
        {game?}
        {--workers=10 : Number of parallel queue workers}
        {--limit=}
        {--dry-run}';

    public function handle(): int
    {
        $cards = TcgCard::whereNotNull('image_small')
            ->where('image_small', 'like', '%storage.bml.gg%')
            ->get();

        // Dispatch jobs to queue
        foreach ($cards as $card) {
            $s3Path = $this->extractS3Path($card->image_small);
            GenerateResponsiveImageVariants::dispatch($card->id, $s3Path);
        }

        // Start parallel workers
        $workers = $this->option('workers');
        $processes = [];
        for ($i = 0; $i < $workers; $i++) {
            $processes[] = Process::start('php artisan queue:work --once --queue=default');
        }

        // Wait for completion
        foreach ($processes as $process) {
            $process->wait();
        }

        return Command::SUCCESS;
    }
}

// app/Jobs/GenerateResponsiveImageVariants.php
class GenerateResponsiveImageVariants implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public int $cardId,
        public string $s3Path,
    ) {}

    public function handle(ImageConversionService $imageService): void
    {
        $imageService->generateResponsiveVariants($this->s3Path);
    }
}
```

**3. Frontend: srcSet utility (TypeScript)**
```typescript
// resources/js/lib/image-utils.ts
const RESPONSIVE_VARIANTS = {
  sm: { suffix: '-sm', width: 200 },
  md: { suffix: '-md', width: 400 },
  lg: { suffix: '', width: 733 },
} as const;

const S3_DOMAIN = 'storage.bml.gg';

function isS3CardImage(url: string): boolean {
  return url.includes(S3_DOMAIN) && url.endsWith('.webp');
}

export function generateResponsiveSrcSet(url: string | null | undefined): string | undefined {
  if (!url || !isS3CardImage(url)) return undefined;

  const baseUrl = url.replace(/\.webp$/, '');
  return [
    `${baseUrl}${RESPONSIVE_VARIANTS.sm.suffix}.webp ${RESPONSIVE_VARIANTS.sm.width}w`,
    `${baseUrl}${RESPONSIVE_VARIANTS.md.suffix}.webp ${RESPONSIVE_VARIANTS.md.width}w`,
    `${baseUrl}.webp ${RESPONSIVE_VARIANTS.lg.width}w`,
  ].join(', ');
}

export function getSmallVariantUrl(url: string | null | undefined): string | undefined {
  if (!url || !isS3CardImage(url)) return url ?? undefined;
  return url.replace(/\.webp$/, '-sm.webp');
}
```

**4. Frontend: Image component with srcSet**
```tsx
// resources/js/components/card-image.tsx
import { generateResponsiveSrcSet } from "@/lib/image-utils";

const sizes = {
  sm: "(max-width: 640px) 45vw, 120px",
  md: "(max-width: 640px) 45vw, 160px",
  lg: "(max-width: 640px) 45vw, 200px",
} as const;

export function CardImage({ src, variant = "md", alt, ...props }) {
  const srcSet = generateResponsiveSrcSet(src);

  return (
    <img
      src={src}
      srcSet={srcSet}
      sizes={sizes[variant]}
      alt={alt}
      loading="lazy"
      decoding="async"
      {...props}
    />
  );
}
```

**5. LCP preload for first visible image**
```tsx
// resources/js/pages/cards/index.tsx
import { Head } from "@inertiajs/react";
import { getSmallVariantUrl } from "@/lib/image-utils";

export default function CardsIndex({ cards }) {
  const firstCardImage = cards[0]?.image_small;
  const preloadImageUrl = getSmallVariantUrl(firstCardImage);

  return (
    <>
      <Head title="Browse Cards">
        {preloadImageUrl && (
          <link rel="preload" as="image" href={preloadImageUrl} type="image/webp" />
        )}
      </Head>
      {/* ... */}
    </>
  );
}
```

**6. Additional optimizations in Laravel Blade template**
```html
<!-- resources/views/app.blade.php -->
<head>
  <!-- Preconnect to S3 CDN -->
  <link rel="preconnect" href="https://storage.bml.gg" crossorigin>
  <link rel="dns-prefetch" href="https://storage.bml.gg">

  <!-- Non-blocking font loading -->
  <link rel="preload" href="https://fonts.bunny.net/css?family=inter:400,500,600&display=swap"
        as="style" onload="this.onload=null;this.rel='stylesheet'">
  <noscript>
    <link href="https://fonts.bunny.net/css?family=inter:400,500,600&display=swap" rel="stylesheet">
  </noscript>
</head>
```

**7. CSS content-visibility for off-screen cards**
```css
/* resources/css/app.css */
.card-grid > * {
  content-visibility: auto;
  contain-intrinsic-size: 0 280px;
}

@media (min-width: 768px) {
  .card-grid > * {
    contain-intrinsic-size: 0 260px;
  }
}
```

**8. Reduce initial pagination (backend)**
```php
// Reduce from 250 to 48 cards per initial load
$perPage = $isSingleSet ? 10000 : 48;
```

**Results achieved:**
| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| LCP | 20.8s | 6.7s | 68% faster |
| Performance Score | 55% | 69% | +14 points |
| CLS | variable | 0 | Perfect |
| TBT | variable | 50ms | Excellent |

**Key takeaways:**
1. Always serve images at display size, not storage size
2. Use srcSet with multiple breakpoints for different viewports
3. Preload LCP image (first visible card)
4. Use WebP format with quality 75-85 for cards
5. Add content-visibility for off-screen rendering skip
6. Reduce initial payload (fewer cards loaded upfront)
7. Use parallel workers for batch image processing
</laravel_react_image_optimization>

<font_optimization>
```tsx
// Next.js font optimization
import { Inter, JetBrains_Mono } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',  // Show fallback while loading
  preload: true,
  variable: '--font-sans',
});

const jetbrains = JetBrains_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-mono',
});

export default function RootLayout({ children }) {
  return (
    <html className={`${inter.variable} ${jetbrains.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

**Self-hosted fonts:**
```css
@font-face {
  font-family: 'Inter';
  src: url('/fonts/Inter.woff2') format('woff2');
  font-weight: 400 700;
  font-display: swap;
  unicode-range: U+0000-00FF;  /* Latin subset */
}
```
</font_optimization>

<measurement_tools>
**Lighthouse (Chrome DevTools):**
- Performance score and metrics
- Specific recommendations
- Simulated throttling

**React DevTools Profiler:**
- Component render times
- Re-render causes
- Commit information

**Web Vitals library:**
```tsx
import { onCLS, onINP, onLCP } from 'web-vitals';

onCLS(console.log);
onINP(console.log);
onLCP(console.log);

// Send to analytics
function sendToAnalytics({ name, value, id }) {
  analytics.send({
    metric: name,
    value: Math.round(value),
    id,
  });
}

onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);
```

**Chrome DevTools Performance tab:**
- Record interactions
- Analyze main thread activity
- Identify long tasks
- View network waterfall
</measurement_tools>

<performance_checklist>
```markdown
- [ ] Lighthouse Performance score 90+
- [ ] LCP < 2.5s
- [ ] INP < 200ms
- [ ] CLS < 0.1
- [ ] Bundle size analyzed and optimized
- [ ] Code split by route
- [ ] Heavy libraries dynamically imported
- [ ] Images optimized (format, size, lazy loading)
- [ ] Fonts optimized (preload, swap)
- [ ] Server Components used for static content
- [ ] No unnecessary re-renders (check Profiler)
```
</performance_checklist>

<parallel_data_fetching>
**Always await Promise.all for parallel fetches:**

```tsx
// ❌ WRONG - Missing await (bug: code continues before fetches complete)
Promise.all([
  fetchUserProfile(id),
  fetchGames(id)
]);
return; // Returns immediately, fetches may never complete properly

// ✅ CORRECT - Always await
await Promise.all([
  fetchUserProfile(id),
  fetchGames(id)
]);

// ✅ With destructuring for results
const [user, games] = await Promise.all([
  fetchUserProfile(id),
  fetchGames(id)
]);
```

**When to use Promise.all:**
- Operations are independent (neither depends on the other's result)
- You want to run them in parallel for speed
- Both operations need to complete before continuing

**When NOT to use:**
- One operation needs the result of another (use sequential await)
- Partial failures need different handling (use Promise.allSettled)

```tsx
// Sequential (when dependent)
const user = await fetchUser(id);
const posts = await fetchUserPosts(user.id); // Needs user.id

// Parallel (when independent)
const [user, settings, notifications] = await Promise.all([
  fetchUser(id),
  fetchSettings(id),
  fetchNotifications(id)
]);

// Handle partial failures
const results = await Promise.allSettled([
  fetchUser(id),
  fetchPosts(id)
]);
results.forEach(result => {
  if (result.status === 'fulfilled') console.log(result.value);
  if (result.status === 'rejected') console.error(result.reason);
});
```
</parallel_data_fetching>

<external_cdn_images>
**Use `unoptimized` for pre-optimized external CDN images:**

```tsx
// Steam CDN images are already optimized by Cloudflare
// Skip Next.js processing to reduce server load

// ✅ Game covers from Steam CDN
<Image
  src={`https://cdn.cloudflare.steamstatic.com/steam/apps/${appId}/library_600x900.jpg`}
  alt={gameName}
  fill
  unoptimized  // Skip Next.js optimization - already optimized
/>

// ✅ Steam avatars (small, fixed size)
<Image
  src={avatarUrl}  // avatars.steamstatic.com
  alt="Profile"
  width={56}
  height={56}
  unoptimized  // Pre-optimized by Steam CDN
/>

// ❌ DON'T use unoptimized for your own images
<Image
  src="/my-image.jpg"  // Your own asset
  alt="Hero"
  fill
  // Let Next.js optimize this
/>
```

**When to use `unoptimized`:**
- Images from established CDNs (Steam, Cloudflare, imgix, etc.)
- CDN already handles format conversion, resizing, compression
- Fixed-size thumbnails/avatars from external sources
- Reduces server processing load

**When NOT to use:**
- Your own images that benefit from Next.js optimization
- Images without CDN optimization
- When you need responsive srcset generation
</external_cdn_images>

<api_caching_strategy>
**Choose ONE caching strategy per endpoint:**

```tsx
// Option 1: Next.js built-in (simpler, recommended)
const response = await fetch(url, {
  next: { revalidate: 3600 }  // Cache for 1 hour
});

// Option 2: Manual Cache-Control headers (for CDN control)
export async function GET() {
  const data = await fetchData();
  const response = NextResponse.json(data);

  // 24-hour cache with stale-while-revalidate
  response.headers.set(
    'Cache-Control',
    'public, s-maxage=86400, stale-while-revalidate=604800'
  );
  return response;
}

// ❌ DON'T mix both strategies
const response = await fetch(url, {
  next: { revalidate: 3600 }  // Next.js caching
});
// AND
response.headers.set('Cache-Control', '...');  // Confusing!
```

**Cache-Control header breakdown:**
- `public` - CDN can cache
- `s-maxage=86400` - CDN caches for 24 hours
- `max-age=86400` - Browser caches for 24 hours
- `stale-while-revalidate=604800` - Serve stale while fetching fresh (7 days)

**Good use cases for manual headers:**
- Semi-static API data (game metadata, product info)
- Data that changes infrequently
- When you need fine-grained CDN control
</api_caching_strategy>
