<overview>
React-specific anti-patterns and simplification patterns. Use alongside javascript.md for React/JSX/TSX code.
</overview>

<anti_patterns>

<pattern name="unnecessary_fragment">
**Bloated:**
```jsx
return (
  <>
    <div>Content</div>
  </>
);
```

**Minimal:**
```jsx
return <div>Content</div>;
```

Only use fragments when returning multiple siblings.
</pattern>

<pattern name="verbose_conditional_render">
**Bloated:**
```jsx
{isVisible ? <Component /> : null}
```

**Minimal:**
```jsx
{isVisible && <Component />}
```
</pattern>

<pattern name="unnecessary_state">
**Bloated:**
```jsx
const [fullName, setFullName] = useState('');

useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);
```

**Minimal (derived state):**
```jsx
const fullName = `${firstName} ${lastName}`;
```

Don't store what you can compute.
</pattern>

<pattern name="over_memoization">
**Bloated:**
```jsx
const value = useMemo(() => a + b, [a, b]);
const handler = useCallback(() => doThing(), []);
```

**Minimal (if not expensive):**
```jsx
const value = a + b;
const handler = () => doThing();
```

Only memoize expensive computations or when passing to memoized children.
</pattern>

<pattern name="verbose_event_handler">
**Bloated:**
```jsx
<button onClick={(e) => handleClick(e)}>Click</button>
```

**Minimal:**
```jsx
<button onClick={handleClick}>Click</button>
```
</pattern>

<pattern name="inline_object_style">
**Bloated (causes re-renders):**
```jsx
<div style={{ color: 'red', fontSize: 16 }}>
```

**Minimal (if static):**
```jsx
const styles = { color: 'red', fontSize: 16 };
// ...
<div style={styles}>
```

Or use CSS/Tailwind classes.
</pattern>

<pattern name="prop_drilling_state">
**Bloated:**
```jsx
function Parent() {
  const [value, setValue] = useState(0);
  return <Child value={value} setValue={setValue} />;
}
function Child({ value, setValue }) {
  return <GrandChild value={value} setValue={setValue} />;
}
function GrandChild({ value, setValue }) {
  return <button onClick={() => setValue(v => v + 1)}>{value}</button>;
}
```

**Minimal (context or composition):**
```jsx
function Parent() {
  const [value, setValue] = useState(0);
  return (
    <GrandChild value={value} onIncrement={() => setValue(v => v + 1)} />
  );
}
```
</pattern>

<pattern name="class_component_when_function_works">
**Bloated:**
```jsx
class Button extends React.Component {
  render() {
    return <button>{this.props.label}</button>;
  }
}
```

**Minimal:**
```jsx
function Button({ label }) {
  return <button>{label}</button>;
}
```

Or even:
```jsx
const Button = ({ label }) => <button>{label}</button>;
```
</pattern>

<pattern name="unnecessary_useEffect_for_transform">
**Bloated:**
```jsx
const [items, setItems] = useState(data);

useEffect(() => {
  setItems(data.filter(d => d.active));
}, [data]);
```

**Minimal:**
```jsx
const items = data.filter(d => d.active);
```

useEffect is for side effects, not data transformation.
</pattern>

<pattern name="verbose_form_handling">
**Bloated:**
```jsx
const [name, setName] = useState('');
const [email, setEmail] = useState('');
const [phone, setPhone] = useState('');

<input value={name} onChange={e => setName(e.target.value)} />
<input value={email} onChange={e => setEmail(e.target.value)} />
<input value={phone} onChange={e => setPhone(e.target.value)} />
```

**Minimal:**
```jsx
const [form, setForm] = useState({ name: '', email: '', phone: '' });

const update = (field) => (e) => setForm(f => ({ ...f, [field]: e.target.value }));

<input value={form.name} onChange={update('name')} />
<input value={form.email} onChange={update('email')} />
<input value={form.phone} onChange={update('phone')} />
```
</pattern>

<pattern name="nested_ternaries">
**Bloated:**
```jsx
{status === 'loading' ? <Loader /> : status === 'error' ? <Error /> : <Content />}
```

**Minimal:**
```jsx
{status === 'loading' && <Loader />}
{status === 'error' && <Error />}
{status === 'success' && <Content />}
```

Or use early returns in component body.
</pattern>

</anti_patterns>

<idioms>

<idiom name="component_composition">
Prefer composition over configuration:
```jsx
// Not this
<Card title="Hello" body="World" footer={<Button />} />

// This
<Card>
  <Card.Title>Hello</Card.Title>
  <Card.Body>World</Card.Body>
  <Card.Footer><Button /></Card.Footer>
</Card>
```
</idiom>

<idiom name="early_return_in_render">
```jsx
function Component({ data, loading, error }) {
  if (loading) return <Loader />;
  if (error) return <Error message={error} />;
  if (!data) return null;

  return <Content data={data} />;
}
```
</idiom>

<idiom name="key_for_lists">
Always use stable, unique keys:
```jsx
{items.map(item => <Item key={item.id} {...item} />)}
```

Never use index as key if list can reorder.
</idiom>

<idiom name="spread_props">
```jsx
const { className, ...rest } = props;
return <div className={className} {...rest} />;
```
</idiom>

<idiom name="default_props_destructuring">
```jsx
function Button({ variant = 'primary', size = 'md', children }) {
  // use directly
}
```
</idiom>

</idioms>

<hooks_patterns>

<pattern name="custom_hook_extraction">
When useState + useEffect are coupled, extract to custom hook:
```jsx
// Before: in component
const [data, setData] = useState(null);
const [loading, setLoading] = useState(true);
useEffect(() => {
  fetch(url).then(r => r.json()).then(setData).finally(() => setLoading(false));
}, [url]);

// After: custom hook
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  useEffect(() => {
    fetch(url).then(r => r.json()).then(setData).finally(() => setLoading(false));
  }, [url]);
  return { data, loading };
}

// In component
const { data, loading } = useFetch(url);
```
</pattern>

</hooks_patterns>
