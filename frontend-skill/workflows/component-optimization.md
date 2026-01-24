# Workflow: Component Optimization

<required_reading>
**Read these reference files NOW:**
1. references/performance.md
2. references/react-patterns.md
3. references/anti-patterns.md
</required_reading>

<context>
Performance optimization in 2025: React Compiler handles most memoization automatically. Focus on architecture (Server Components), bundle size, and avoiding unnecessary work. Measure before optimizing.
</context>

<process>

<step name="measure-first">
Never optimize without measuring:

```bash
# Bundle analysis
npm run build -- --analyze

# Or with webpack-bundle-analyzer
npx webpack-bundle-analyzer stats.json
```

**React DevTools Profiler:**
1. Open React DevTools → Profiler tab
2. Click Record
3. Interact with the component
4. Stop recording
5. Analyze render times and re-render causes

**Lighthouse:**
- Open Chrome DevTools → Lighthouse
- Run audit for Performance
- Focus on: LCP, FID/INP, CLS
</step>

<step name="bundle-optimization">
**Code splitting by route:**
```tsx
// Next.js - automatic per-page splitting
// React Router - manual lazy loading
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
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
// Don't import at top level
// import { Chart } from 'chart.js';

// Do: Import when needed
async function showChart(data) {
  const { Chart } = await import('chart.js');
  new Chart(canvas, { data });
}
```

**Tree shaking:**
```tsx
// Bad - imports entire library
import _ from 'lodash';
_.debounce(fn, 300);

// Good - imports only what's needed
import debounce from 'lodash/debounce';
debounce(fn, 300);
```
</step>

<step name="server-components">
Move work to the server:

```tsx
// Before: Client Component fetching data
'use client';
function Products() {
  const [products, setProducts] = useState([]);
  useEffect(() => {
    fetch('/api/products').then(r => r.json()).then(setProducts);
  }, []);
  return <ProductList products={products} />;
}

// After: Server Component (no JS shipped for this)
async function Products() {
  const products = await db.products.findMany();
  return <ProductList products={products} />;
}
```
</step>

<step name="render-optimization">
**React Compiler (React 19)** handles memoization automatically. But still avoid:

```tsx
// Bad: Creates new object every render
<Component style={{ color: 'red' }} />

// Good: Stable reference
const style = { color: 'red' };
<Component style={style} />

// Or with Tailwind (no style object)
<Component className="text-red-500" />
```

**When React Compiler isn't enough:**
```tsx
// Expensive computation
const sortedItems = useMemo(
  () => items.sort((a, b) => expensiveCompare(a, b)),
  [items]
);

// Stable callback for child components
const handleClick = useCallback(
  (id) => dispatch({ type: 'SELECT', id }),
  [dispatch]
);
```
</step>

<step name="list-virtualization">
For long lists (100+ items):

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: virtualRow.start,
              height: virtualRow.size,
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
</step>

<step name="image-optimization">
```tsx
// Next.js Image (automatic optimization)
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority  // For above-the-fold images
/>

// Below-fold images lazy load by default (no prop needed)
<Image
  src="/feature.jpg"
  alt="Feature"
  width={600}
  height={400}
/>
```
</step>

<step name="font-optimization">
```tsx
// Next.js font optimization
import { Inter } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',  // Prevent FOIT
  preload: true,
});

export default function Layout({ children }) {
  return <html className={inter.className}>{children}</html>;
}
```
</step>

<step name="verify">
After optimization, measure again:

1. **Bundle size:** Should be smaller
2. **Lighthouse Performance:** Should be 90+
3. **Core Web Vitals:**
   - LCP < 2.5s
   - INP < 200ms
   - CLS < 0.1
4. **React Profiler:** Fewer re-renders, faster render times
</step>

</process>

<anti_patterns>
Avoid:
- Premature optimization (measure first!)
- Over-memoizing (React Compiler handles most cases)
- Importing entire libraries
- Client Components for static content
- Inline object/function props without reason
- Loading all data upfront (paginate, virtualize)
</anti_patterns>

<success_criteria>
- Bundle size measured and reduced
- Server Components used for static content
- Code split by route
- Heavy libraries dynamically imported
- Long lists virtualized
- Images optimized (format, size, lazy loading)
- Fonts optimized (preload, swap)
- Lighthouse Performance score 90+
- Core Web Vitals all green
</success_criteria>
