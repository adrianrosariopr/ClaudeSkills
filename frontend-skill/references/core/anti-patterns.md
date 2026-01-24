<overview>
Common frontend anti-patterns and mistakes to avoid: architecture, React, CSS, performance, and accessibility issues.
</overview>

<react_anti_patterns>
<anti_pattern name="everything-is-client">
**Problem:** Making all components Client Components

```tsx
// Bad: Everything is 'use client'
'use client';
function ProductPage() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetch('/api/products').then(r => r.json()).then(setProducts);
  }, []);

  return <ProductList products={products} />;
}
```

**Why it's bad:**
- Larger JavaScript bundle
- Slower initial load
- No streaming/suspense benefits
- Extra network request for data

**Instead:**
```tsx
// Good: Server Component by default
async function ProductPage() {
  const products = await db.products.findMany();
  return <ProductList products={products} />;
}

// Only client where needed
'use client';
function AddToCartButton({ productId }) {
  return <button onClick={() => addToCart(productId)}>Add</button>;
}
```
</anti_pattern>

<anti_pattern name="useeffect-for-data">
**Problem:** Using useEffect for data fetching

```tsx
// Bad: useEffect data fetching
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(setUser)
      .finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <Spinner />;
  return <Profile user={user} />;
}
```

**Why it's bad:**
- Waterfalls (parent loads, then children)
- No caching
- Race conditions
- Manual loading/error state

**Instead:**
```tsx
// Good: Server Component
async function UserProfile({ userId }) {
  const user = await getUser(userId);
  return <Profile user={user} />;
}

// Or: TanStack Query for client
function UserProfile({ userId }) {
  const { data: user, isLoading } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => getUser(userId),
  });

  if (isLoading) return <Spinner />;
  return <Profile user={user} />;
}
```
</anti_pattern>

<anti_pattern name="prop-drilling">
**Problem:** Passing props through many levels

```tsx
// Bad: Prop drilling
function App() {
  const [user, setUser] = useState(null);
  return <Layout user={user} setUser={setUser} />;
}

function Layout({ user, setUser }) {
  return <Sidebar user={user} setUser={setUser} />;
}

function Sidebar({ user, setUser }) {
  return <UserMenu user={user} setUser={setUser} />;
}
```

**Instead:**
```tsx
// Good: Context for global state
const UserContext = createContext(null);

function App() {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Layout />
    </UserContext.Provider>
  );
}

function UserMenu() {
  const { user, setUser } = useContext(UserContext);
  // Use directly
}

// Or: Composition
function Layout({ children }) {
  return <div className="layout">{children}</div>;
}

function App() {
  const [user, setUser] = useState(null);
  return (
    <Layout>
      <Sidebar>
        <UserMenu user={user} setUser={setUser} />
      </Sidebar>
    </Layout>
  );
}
```
</anti_pattern>

<anti_pattern name="global-state-overuse">
**Problem:** Putting everything in global state

```tsx
// Bad: Everything in Redux/Zustand
const useStore = create((set) => ({
  modalOpen: false,
  formData: {},
  hoveredItem: null,  // Really?
  // ...50 more ephemeral states
}));
```

**Why it's bad:**
- Unnecessary complexity
- Performance (global updates trigger more re-renders)
- Hard to reason about

**Instead:**
- Local state for UI (modal open, form values)
- Server state in TanStack Query
- Global state only for truly app-wide data (user, theme)
</anti_pattern>
</react_anti_patterns>

<css_anti_patterns>
<anti_pattern name="div-for-everything">
**Problem:** Using div for everything

```html
<!-- Bad -->
<div class="button" onclick="handleClick()">Click me</div>
<div class="link" onclick="navigate()">Go somewhere</div>
<div class="heading">Page Title</div>
```

**Why it's bad:**
- No keyboard accessibility
- No screen reader semantics
- No native behaviors (form submission, link following)

**Instead:**
```html
<!-- Good -->
<button onclick="handleClick()">Click me</button>
<a href="/somewhere">Go somewhere</a>
<h1>Page Title</h1>
```
</anti_pattern>

<anti_pattern name="important-everywhere">
**Problem:** Using !important to fix specificity

```css
/* Bad: Specificity war */
.button {
  background: blue !important;
  padding: 10px !important;
}

.button.primary {
  background: green !important;  /* Even more important? */
}
```

**Why it's bad:**
- Makes CSS unmaintainable
- Creates specificity wars
- Hard to override later

**Instead:**
```css
/* Good: Use cascade layers or lower specificity */
@layer components {
  .button {
    background: blue;
    padding: 10px;
  }

  .button.primary {
    background: green;
  }
}

/* Or use CSS custom properties */
.button {
  background: var(--button-bg, blue);
}

.button.primary {
  --button-bg: green;
}
```
</anti_pattern>

<anti_pattern name="magic-numbers">
**Problem:** Arbitrary values instead of a system

```css
/* Bad: Magic numbers */
.card {
  padding: 17px;
  margin-bottom: 23px;
  border-radius: 7px;
}

.header {
  padding: 19px;
  margin-bottom: 31px;
}
```

**Instead:**
```css
/* Good: Design system values */
:root {
  --spacing-sm: 0.5rem;   /* 8px */
  --spacing-md: 1rem;     /* 16px */
  --spacing-lg: 1.5rem;   /* 24px */
  --radius-md: 0.5rem;
}

.card {
  padding: var(--spacing-md);
  margin-bottom: var(--spacing-lg);
  border-radius: var(--radius-md);
}
```
</anti_pattern>

<anti_pattern name="inline-styles">
**Problem:** Inline styles everywhere

```tsx
// Bad
<div style={{ padding: '20px', marginBottom: '15px', backgroundColor: '#f0f0f0' }}>
  <h2 style={{ fontSize: '24px', fontWeight: 'bold', color: '#333' }}>
    Title
  </h2>
</div>
```

**Why it's bad:**
- Highest specificity (hard to override)
- No reusability
- No pseudo-classes (:hover, :focus)
- Larger bundle size

**Instead:**
```tsx
// Good: Utility classes or CSS modules
<div className="p-5 mb-4 bg-gray-100">
  <h2 className="text-2xl font-bold text-gray-800">
    Title
  </h2>
</div>
```
</anti_pattern>
</css_anti_patterns>

<performance_anti_patterns>
<anti_pattern name="premature-optimization">
**Problem:** Optimizing without measuring

```tsx
// Bad: useMemo/useCallback everywhere "just in case"
function Component({ data }) {
  const processed = useMemo(() => data.map(x => x), [data]);
  const handleClick = useCallback(() => {}, []);
  const style = useMemo(() => ({ color: 'red' }), []);
  // ...
}
```

**Why it's bad:**
- React Compiler handles most cases
- Adds complexity
- May not actually improve performance
- Can even hurt performance (memoization overhead)

**Instead:**
- Measure first (React Profiler, Lighthouse)
- Optimize bottlenecks
- Trust React Compiler for most cases
</anti_pattern>

<anti_pattern name="large-bundles">
**Problem:** Importing entire libraries

```tsx
// Bad
import _ from 'lodash';
import * as Icons from 'lucide-react';
import moment from 'moment';

// Uses only:
_.debounce(fn, 300);
<Icons.X />
moment().format('YYYY');
```

**Instead:**
```tsx
// Good: Import only what you need
import debounce from 'lodash/debounce';
import { X } from 'lucide-react';
import { format } from 'date-fns';

debounce(fn, 300);
<X />
format(new Date(), 'yyyy');
```
</anti_pattern>

<anti_pattern name="render-blocking">
**Problem:** Loading everything upfront

```tsx
// Bad: All components loaded on initial page
import HeavyChart from './HeavyChart';
import PDFViewer from './PDFViewer';
import VideoPlayer from './VideoPlayer';

function Dashboard() {
  return (
    <>
      <HeavyChart />
      <PDFViewer />
      <VideoPlayer />
    </>
  );
}
```

**Instead:**
```tsx
// Good: Lazy load heavy components
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));
const PDFViewer = lazy(() => import('./PDFViewer'));
const VideoPlayer = lazy(() => import('./VideoPlayer'));

function Dashboard() {
  return (
    <>
      <Suspense fallback={<ChartSkeleton />}>
        <HeavyChart />
      </Suspense>
      {/* Load others on demand */}
    </>
  );
}
```
</anti_pattern>
<anti_pattern name="redundant-lazy-loading">
**Problem:** Adding `loading="lazy"` to Next.js Image

```tsx
// Bad: Redundant - Next.js Image already handles lazy loading
<Image
  src="/photo.jpg"
  alt="Photo"
  fill
  loading="lazy"  // ❌ Unnecessary
/>
```

**Why it's bad:**
- Next.js Image already lazy loads by default
- Adding it explicitly can interfere with Next.js's intelligent loading strategy
- Adds noise without benefit

**Instead:**
```tsx
// Good: Let Next.js handle it (default is lazy)
<Image
  src="/photo.jpg"
  alt="Photo"
  fill
/>

// Only use priority for above-the-fold images
<Image
  src="/hero.jpg"
  alt="Hero"
  fill
  priority  // Preload this image
/>
```
</anti_pattern>

<anti_pattern name="useless-memo">
**Problem:** Using React.memo() with unstable props

```tsx
// Bad: data is recreated every render, memo doesn't help
const Chart = memo(function Chart({ data }) {
  return <BarChart data={data} />;
});

// Parent component
function Dashboard() {
  // This creates a NEW array every render
  const chartData = [
    { name: 'A', value: 10 },
    { name: 'B', value: 20 },
  ];

  return <Chart data={chartData} />;  // memo is useless here
}
```

**Why it's bad:**
- memo() compares props by reference
- New array/object = new reference = memo doesn't prevent re-render
- Adds overhead without benefit (comparison + wrapper)

**Instead:**
```tsx
// Option 1: Stabilize the data with useMemo
function Dashboard() {
  const chartData = useMemo(() => [
    { name: 'A', value: 10 },
    { name: 'B', value: 20 },
  ], []);  // Empty deps = stable reference

  return <Chart data={chartData} />;  // Now memo works
}

// Option 2: Don't use memo at all (React Compiler handles most cases)
function Chart({ data }) {
  return <BarChart data={data} />;
}
```
</anti_pattern>

<anti_pattern name="promise-all-no-await">
**Problem:** Using Promise.all without await

```tsx
// ❌ CRITICAL BUG: Missing await
async function loadData() {
  Promise.all([
    fetchUser(id),
    fetchPosts(id)
  ]);
  return;  // Returns BEFORE promises complete!
}

// The promises run but:
// - Code continues immediately
// - Errors are silently swallowed
// - State updates may happen after unmount
```

**Instead:**
```tsx
// ✅ Always await Promise.all
async function loadData() {
  await Promise.all([
    fetchUser(id),
    fetchPosts(id)
  ]);
  // Now this runs after both complete
}

// ✅ Or capture results
const [user, posts] = await Promise.all([
  fetchUser(id),
  fetchPosts(id)
]);
```
</anti_pattern>
</performance_anti_patterns>

<accessibility_anti_patterns>
<anti_pattern name="missing-alt-text">
**Problem:** Images without alt text

```html
<!-- Bad -->
<img src="chart.png" />
<img src="decorative.svg" />
```

**Instead:**
```html
<!-- Good: Descriptive alt for meaningful images -->
<img src="chart.png" alt="Sales chart showing 20% growth in Q4" />

<!-- Good: Empty alt for decorative images -->
<img src="decorative.svg" alt="" />
```
</anti_pattern>

<anti_pattern name="color-only-feedback">
**Problem:** Relying only on color for information

```html
<!-- Bad: Color is the only indicator -->
<span class="text-red-500">Error</span>
<span class="text-green-500">Success</span>
```

**Instead:**
```html
<!-- Good: Color + icon + text -->
<span class="text-red-500">
  <XCircle aria-hidden="true" />
  Error: Invalid email address
</span>

<span class="text-green-500">
  <CheckCircle aria-hidden="true" />
  Success: Changes saved
</span>
```
</anti_pattern>

<anti_pattern name="missing-focus-styles">
**Problem:** Removing focus outlines without replacement

```css
/* Bad: Removes all focus indication */
*:focus {
  outline: none;
}
```

**Instead:**
```css
/* Good: Custom focus style */
*:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

/* Remove only for mouse users */
*:focus:not(:focus-visible) {
  outline: none;
}
```
</anti_pattern>

<anti_pattern name="keyboard-trap">
**Problem:** Users can't escape with keyboard

```tsx
// Bad: Modal with no escape handling
function Modal({ children }) {
  return <div className="modal">{children}</div>;
}
```

**Instead:**
```tsx
// Good: Handle escape, trap focus
function Modal({ isOpen, onClose, children }) {
  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === 'Escape') onClose();
    };
    document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }, [onClose]);

  return (
    <dialog open={isOpen} onClose={onClose}>
      {children}
      <button onClick={onClose}>Close</button>
    </dialog>
  );
}
```
</anti_pattern>
</accessibility_anti_patterns>

<architecture_anti_patterns>
<anti_pattern name="no-loading-states">
**Problem:** Not handling loading/error states

```tsx
// Bad: Assumes data is always there
function UserList() {
  const { data } = useQuery(['users']);
  return data.map(user => <User key={user.id} {...user} />);
}
```

**Instead:**
```tsx
// Good: Handle all states
function UserList() {
  const { data, isLoading, error } = useQuery(['users']);

  if (isLoading) return <UserListSkeleton />;
  if (error) return <ErrorMessage error={error} />;
  if (!data?.length) return <EmptyState message="No users found" />;

  return data.map(user => <User key={user.id} {...user} />);
}
```
</anti_pattern>

<anti_pattern name="no-empty-states">
**Problem:** Blank screens when no data

```tsx
// Bad: Just shows nothing
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => <Todo key={todo.id} {...todo} />)}
    </ul>
  );
}
```

**Instead:**
```tsx
// Good: Helpful empty state
function TodoList({ todos }) {
  if (!todos.length) {
    return (
      <div className="text-center py-12">
        <ClipboardIcon className="w-12 h-12 mx-auto text-gray-400" />
        <h3 className="mt-2 text-lg font-medium">No todos yet</h3>
        <p className="mt-1 text-gray-500">Get started by creating a new todo.</p>
        <button className="mt-4">Create todo</button>
      </div>
    );
  }

  return (
    <ul>
      {todos.map(todo => <Todo key={todo.id} {...todo} />)}
    </ul>
  );
}
```
</anti_pattern>
</architecture_anti_patterns>

<ui_style_anti_patterns>
<anti_pattern name="generic-neon-gaming">
**Problem:** Using generic "gaming" aesthetics like neon gradients

```css
/* Bad: Overdone, dated "gaming" look */
background: linear-gradient(to-br, from-gray-900, to-purple-900);
color: #00ffff;  /* cyan neon */
box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);  /* glow */
```

**Why it's bad:**
- Feels dated (2010s aesthetic)
- Generic - looks like every other "gaming" site
- Hard to read text over busy gradients
- No real personality or memorability

**Instead:**
```css
/* Good: Modern, distinctive style (neobrutalism example) */
background: #f0f0f0;
border: 4px solid black;
box-shadow: 8px 8px 0 black;  /* Hard shadow */
font-weight: 700;
text-transform: uppercase;
```
</anti_pattern>

<anti_pattern name="scattered-styles">
**Problem:** One-off styles instead of a design system

```tsx
// Bad: Every component has random styles
<div className="bg-purple-900/50 backdrop-blur-sm border-cyan-500/30">
<button className="bg-gradient-to-r from-cyan-500 to-blue-600">
<card className="border-2 border-purple-400 rounded-xl">
```

**Why it's bad:**
- No consistency across components
- Impossible to maintain
- Every new component needs new decisions
- User notices the visual chaos

**Instead:**
```css
/* Good: Define system utilities once */
.arcade-btn {
  background: hsl(var(--primary));
  border: 3px solid black;
  box-shadow: 4px 4px 0 black;
}

.arcade-card {
  background: white;
  border: 4px solid black;
  box-shadow: 8px 8px 0 black;
}
```

```tsx
// Apply consistently
<button className="arcade-btn">Click</button>
<div className="arcade-card">Content</div>
```
</anti_pattern>

<anti_pattern name="partial-redesign">
**Problem:** Updating some pages but not others

**Why it's bad:**
- User navigates between pages and sees different apps
- Breaks mental model and trust
- Shows lack of design system thinking
- Creates maintenance nightmare

**Instead:**
- Update ALL pages in the same session
- Define utility classes in globals.css first
- Test navigation flow between pages
- If you touch one page's style, touch them all
</anti_pattern>

<anti_pattern name="style-over-ux">
**Problem:** Sacrificing usability for aesthetics

```css
/* Bad: "Cool" but unusable */
.text { color: rgba(255,255,255,0.3); }  /* Can't read */
.button { padding: 2px 4px; }            /* Can't click */
.modal { backdrop-filter: blur(20px); }  /* Can't see content */
```

**Why it's bad:**
- Users abandon pretty-but-unusable interfaces
- Accessibility violations
- Style should enhance content, not hide it

**Instead:**
- Maintain 4.5:1 contrast for text
- 44px minimum touch targets
- Clear visual hierarchy
- Style enhances, never obscures
</anti_pattern>
</ui_style_anti_patterns>
