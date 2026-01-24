<overview>
React 19 (2025) patterns and best practices. Server Components by default, React Compiler for automatic optimization, new hooks (useActionState, useOptimistic, use), and modern state management.
</overview>

<server_vs_client>
**Default to Server Components:**

```tsx
// Server Component (default - no directive)
async function ProductList() {
  const products = await db.products.findMany();

  return (
    <ul className="grid gap-4">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </ul>
  );
}

// Server Component with streaming
async function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<ChartSkeleton />}>
        <AsyncChart />  {/* Streams in when ready */}
      </Suspense>
    </div>
  );
}
```

**Use Client Components when you need:**
```tsx
'use client';

import { useState, useEffect } from 'react';

// Event handlers
function LikeButton({ postId }) {
  const handleClick = () => likePost(postId);
  return <button onClick={handleClick}>Like</button>;
}

// Hooks (useState, useEffect, useRef, etc.)
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}

// Browser APIs
function GeolocationButton() {
  const [location, setLocation] = useState(null);

  useEffect(() => {
    navigator.geolocation.getCurrentPosition(pos => {
      setLocation(pos.coords);
    });
  }, []);

  return <div>{location?.latitude}, {location?.longitude}</div>;
}
```

**Composition pattern - Server wraps Client:**
```tsx
// Server Component
async function ProductPage({ id }) {
  const product = await getProduct(id);

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      {/* Pass data to Client Component */}
      <AddToCartButton productId={id} price={product.price} />
    </div>
  );
}

// Client Component
'use client';
function AddToCartButton({ productId, price }) {
  const [loading, setLoading] = useState(false);

  async function handleAdd() {
    setLoading(true);
    await addToCart(productId);
    setLoading(false);
  }

  return (
    <button onClick={handleAdd} disabled={loading}>
      {loading ? 'Adding...' : `Add to Cart - $${price}`}
    </button>
  );
}
```
</server_vs_client>

<react_19_hooks>
**useActionState - Form state management:**
```tsx
'use client';

import { useActionState } from 'react';

async function createUser(prevState, formData) {
  'use server';

  const name = formData.get('name');
  if (!name) {
    return { error: 'Name is required' };
  }

  await db.users.create({ data: { name } });
  return { success: true };
}

function CreateUserForm() {
  const [state, formAction, isPending] = useActionState(createUser, null);

  return (
    <form action={formAction}>
      {state?.error && <p className="text-red-500">{state.error}</p>}
      {state?.success && <p className="text-green-500">User created!</p>}

      <input name="name" required />
      <button disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>
    </form>
  );
}
```

**useFormStatus - Submit button state:**
```tsx
'use client';

import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

// Usage in form
function Form() {
  return (
    <form action={serverAction}>
      <input name="email" />
      <SubmitButton />  {/* Automatically knows form state */}
    </form>
  );
}
```

**useOptimistic - Instant UI feedback:**
```tsx
'use client';

import { useOptimistic } from 'react';

function TodoList({ todos }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, { ...newTodo, pending: true }]
  );

  async function handleAdd(formData) {
    const text = formData.get('text');
    addOptimisticTodo({ id: Date.now(), text });
    await createTodo(text);
  }

  return (
    <>
      <form action={handleAdd}>
        <input name="text" />
        <button>Add</button>
      </form>
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id} className={todo.pending ? 'opacity-50' : ''}>
            {todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

**use() - Read promises in render:**
```tsx
import { use, Suspense } from 'react';

function UserProfile({ userPromise }) {
  const user = use(userPromise);  // Suspends until resolved
  return <div>{user.name}</div>;
}

// Usage
function Page() {
  const userPromise = fetchUser(id);  // Start fetching immediately

  return (
    <Suspense fallback={<Loading />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```
</react_19_hooks>

<component_patterns>
**Compound Components:**
```tsx
const TabsContext = createContext(null);

function Tabs({ defaultValue, children }) {
  const [value, setValue] = useState(defaultValue);

  return (
    <TabsContext.Provider value={{ value, setValue }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

Tabs.List = function TabsList({ children }) {
  return <div className="tabs-list" role="tablist">{children}</div>;
};

Tabs.Tab = function Tab({ value, children }) {
  const ctx = useContext(TabsContext);
  const isActive = ctx.value === value;

  return (
    <button
      role="tab"
      aria-selected={isActive}
      onClick={() => ctx.setValue(value)}
      className={isActive ? 'active' : ''}
    >
      {children}
    </button>
  );
};

Tabs.Content = function TabsContent({ value, children }) {
  const ctx = useContext(TabsContext);
  if (ctx.value !== value) return null;
  return <div role="tabpanel">{children}</div>;
};

// Usage
<Tabs defaultValue="tab1">
  <Tabs.List>
    <Tabs.Tab value="tab1">Tab 1</Tabs.Tab>
    <Tabs.Tab value="tab2">Tab 2</Tabs.Tab>
  </Tabs.List>
  <Tabs.Content value="tab1">Content 1</Tabs.Content>
  <Tabs.Content value="tab2">Content 2</Tabs.Content>
</Tabs>
```

**Render Props:**
```tsx
function DataFetcher({ url, children }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);

  return children({ data, loading, error });
}

// Usage
<DataFetcher url="/api/users">
  {({ data, loading, error }) => {
    if (loading) return <Spinner />;
    if (error) return <Error message={error.message} />;
    return <UserList users={data} />;
  }}
</DataFetcher>
```
</component_patterns>

<custom_hooks>
```tsx
// Debounced value
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Local storage
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    const valueToStore = value instanceof Function ? value(storedValue) : value;
    setStoredValue(valueToStore);
    localStorage.setItem(key, JSON.stringify(valueToStore));
  };

  return [storedValue, setValue] as const;
}

// Media query
function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);
    setMatches(media.matches);

    const listener = (e: MediaQueryListEvent) => setMatches(e.matches);
    media.addEventListener('change', listener);
    return () => media.removeEventListener('change', listener);
  }, [query]);

  return matches;
}

// Click outside
function useClickOutside(ref: RefObject<HTMLElement>, handler: () => void) {
  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent) => {
      if (!ref.current || ref.current.contains(event.target as Node)) return;
      handler();
    };

    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);

    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}
```
</custom_hooks>

<state_management>
**Local state:** useState for component-specific state

**Server state:** TanStack Query
```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function Products() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['products'],
    queryFn: () => fetch('/api/products').then(r => r.json()),
  });

  if (isLoading) return <Skeleton />;
  if (error) return <Error message={error.message} />;
  return <ProductList products={data} />;
}

function CreateProduct() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newProduct) => fetch('/api/products', {
      method: 'POST',
      body: JSON.stringify(newProduct),
    }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      mutation.mutate({ name: e.target.name.value });
    }}>
      <input name="name" />
      <button disabled={mutation.isPending}>
        {mutation.isPending ? 'Creating...' : 'Create'}
      </button>
    </form>
  );
}
```

**Global state:** Zustand (simple) or Context (theme/auth)
```tsx
import { create } from 'zustand';

const useStore = create((set) => ({
  cart: [],
  addToCart: (item) => set((state) => ({
    cart: [...state.cart, item]
  })),
  removeFromCart: (id) => set((state) => ({
    cart: state.cart.filter(item => item.id !== id)
  })),
  clearCart: () => set({ cart: [] }),
}));

// Usage
function Cart() {
  const { cart, removeFromCart } = useStore();
  return (
    <ul>
      {cart.map(item => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => removeFromCart(item.id)}>Remove</button>
        </li>
      ))}
    </ul>
  );
}
```
</state_management>

<react_compiler>
**React Compiler (React 19) handles memoization automatically:**

```tsx
// Before: Manual memoization
const MemoizedComponent = memo(function Component({ data }) {
  const processed = useMemo(() => expensiveCalc(data), [data]);
  const handler = useCallback(() => doSomething(data), [data]);
  return <Child data={processed} onClick={handler} />;
});

// After: React Compiler optimizes automatically
function Component({ data }) {
  const processed = expensiveCalc(data);
  const handler = () => doSomething(data);
  return <Child data={processed} onClick={handler} />;
}
```

**When manual optimization still helps:**
- Very expensive computations
- Third-party libraries not compiled
- Specific performance bottlenecks (measure first!)
</react_compiler>
