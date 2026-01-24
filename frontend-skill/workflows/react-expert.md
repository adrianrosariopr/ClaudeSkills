# Workflow: React Expert

<required_reading>
**Read these reference files NOW:**
1. references/react-patterns.md
2. references/performance.md
3. references/accessibility.md
</required_reading>

<context>
React 19 (2025) brings Server Components as default, the React Compiler for automatic memoization, and new hooks like `useActionState`, `useOptimistic`, and `use`. Default to Server Components; only use Client Components when you need interactivity.
</context>

<process>

<step name="understand-the-task">
Clarify what the user needs:
- Component architecture/patterns?
- Server vs Client Component decision?
- State management approach?
- Form handling?
- Performance optimization?
- Hook usage?
</step>

<step name="server-vs-client">
**Default to Server Components.** Only add `'use client'` when you need:
- Event handlers (`onClick`, `onChange`)
- Hooks (`useState`, `useEffect`, `useRef`)
- Browser APIs (`localStorage`, `window`)

```tsx
// Server Component (default) - no directive needed
async function ProductList() {
  const products = await db.products.findMany();
  return (
    <ul>
      {products.map(p => <ProductCard key={p.id} product={p} />)}
    </ul>
  );
}

// Client Component - needs interactivity
'use client';
function AddToCartButton({ productId }) {
  const [pending, setPending] = useState(false);

  return (
    <button onClick={() => addToCart(productId)} disabled={pending}>
      Add to Cart
    </button>
  );
}
```
</step>

<step name="composition-patterns">
**Compound Components:**
```tsx
function Tabs({ children }) {
  const [activeTab, setActiveTab] = useState(0);
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
}

Tabs.List = function TabList({ children }) { /* ... */ };
Tabs.Tab = function Tab({ children, index }) { /* ... */ };
Tabs.Panels = function TabPanels({ children }) { /* ... */ };
Tabs.Panel = function TabPanel({ children, index }) { /* ... */ };

// Usage
<Tabs>
  <Tabs.List>
    <Tabs.Tab index={0}>Tab 1</Tabs.Tab>
    <Tabs.Tab index={1}>Tab 2</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panels>
    <Tabs.Panel index={0}>Content 1</Tabs.Panel>
    <Tabs.Panel index={1}>Content 2</Tabs.Panel>
  </Tabs.Panels>
</Tabs>
```
</step>

<step name="form-handling">
**React 19 Actions (Server Actions):**
```tsx
// Server Action
async function createUser(formData: FormData) {
  'use server';
  const name = formData.get('name');
  await db.users.create({ data: { name } });
  revalidatePath('/users');
}

// Form component
function CreateUserForm() {
  return (
    <form action={createUser}>
      <input name="name" required />
      <SubmitButton />
    </form>
  );
}

// Submit button with pending state
'use client';
function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? 'Saving...' : 'Save'}</button>;
}
```

**useActionState for form state:**
```tsx
'use client';
function Form() {
  const [state, formAction, isPending] = useActionState(createUser, null);

  return (
    <form action={formAction}>
      {state?.error && <p className="error">{state.error}</p>}
      <input name="name" />
      <button disabled={isPending}>Submit</button>
    </form>
  );
}
```
</step>

<step name="optimistic-updates">
**useOptimistic for instant feedback:**
```tsx
'use client';
function LikeButton({ initialLikes, postId }) {
  const [likes, setLikes] = useState(initialLikes);
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    likes,
    (current, increment) => current + increment
  );

  async function handleLike() {
    addOptimisticLike(1);
    await likePost(postId);
    setLikes(prev => prev + 1);
  }

  return <button onClick={handleLike}>{optimisticLikes} likes</button>;
}
```
</step>

<step name="custom-hooks">
Extract reusable logic:
```tsx
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    const item = window.localStorage.getItem(key);
    return item ? JSON.parse(item) : initialValue;
  });

  const setValue = (value: T) => {
    setStoredValue(value);
    window.localStorage.setItem(key, JSON.stringify(value));
  };

  return [storedValue, setValue] as const;
}
```
</step>

<step name="state-management">
**Local state:** `useState` for component-specific state
**Server state:** TanStack Query for data fetching/caching
**Global state:** Context for theme/auth, Zustand for complex app state

```tsx
// TanStack Query for server state
function Products() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['products'],
    queryFn: () => fetch('/api/products').then(r => r.json()),
  });

  if (isLoading) return <Skeleton />;
  if (error) return <Error message={error.message} />;
  return <ProductList products={data} />;
}
```
</step>

<step name="verify">
Check the implementation:
1. Verify Server/Client boundary is correct
2. Test form submissions and error states
3. Check for unnecessary re-renders (React DevTools Profiler)
4. Test with slow network (throttle in DevTools)
5. Verify accessibility (keyboard navigation, screen readers)
</step>

</process>

<anti_patterns>
Avoid:
- Making everything a Client Component
- Using `useEffect` for data fetching (use Server Components or TanStack Query)
- Prop drilling (use Context or composition)
- Overusing global state (most state is local or server state)
- Manual memoization everywhere (React Compiler handles this)
</anti_patterns>

<success_criteria>
- Server Components for data fetching and static content
- Client Components only for interactivity
- Forms use Server Actions with useFormStatus/useActionState
- Optimistic updates for better UX
- Custom hooks extract reusable logic
- No unnecessary re-renders
- Accessible by default
</success_criteria>
