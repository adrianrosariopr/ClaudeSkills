<overview>
JavaScript/TypeScript anti-patterns and simplification patterns. Applies to vanilla JS, Node.js, and TypeScript.
</overview>

<anti_patterns>

<pattern name="verbose_null_checks">
**Bloated:**
```javascript
if (user && user.profile && user.profile.name) {
  return user.profile.name;
}
return 'Unknown';
```

**Minimal:**
```javascript
return user?.profile?.name ?? 'Unknown';
```
</pattern>

<pattern name="manual_array_operations">
**Bloated:**
```javascript
const results = [];
for (let i = 0; i < items.length; i++) {
  if (items[i].active) {
    results.push(items[i].name);
  }
}
```

**Minimal:**
```javascript
const results = items.filter(i => i.active).map(i => i.name);
```
</pattern>

<pattern name="unnecessary_else">
**Bloated:**
```javascript
function getStatus(code) {
  if (code === 200) {
    return 'ok';
  } else if (code === 404) {
    return 'not found';
  } else {
    return 'error';
  }
}
```

**Minimal:**
```javascript
function getStatus(code) {
  if (code === 200) return 'ok';
  if (code === 404) return 'not found';
  return 'error';
}
```
</pattern>

<pattern name="verbose_object_properties">
**Bloated:**
```javascript
const name = user.name;
const email = user.email;
const age = user.age;
```

**Minimal:**
```javascript
const { name, email, age } = user;
```
</pattern>

<pattern name="same_key_value">
**Bloated:**
```javascript
return { name: name, email: email, age: age };
```

**Minimal:**
```javascript
return { name, email, age };
```
</pattern>

<pattern name="callback_hell">
**Bloated:**
```javascript
getUser(id, function(user) {
  getOrders(user.id, function(orders) {
    getItems(orders[0].id, function(items) {
      callback(items);
    });
  });
});
```

**Minimal:**
```javascript
const user = await getUser(id);
const orders = await getOrders(user.id);
const items = await getItems(orders[0].id);
return items;
```
</pattern>

<pattern name="verbose_conditionals">
**Bloated:**
```javascript
let status;
if (isActive) {
  status = 'active';
} else {
  status = 'inactive';
}
```

**Minimal:**
```javascript
const status = isActive ? 'active' : 'inactive';
```
</pattern>

<pattern name="unnecessary_template_literal">
**Bloated:**
```javascript
const message = `${name}`;
```

**Minimal:**
```javascript
const message = name;
```
</pattern>

<pattern name="verbose_boolean_return">
**Bloated:**
```javascript
if (value > 10) {
  return true;
} else {
  return false;
}
```

**Minimal:**
```javascript
return value > 10;
```
</pattern>

<pattern name="double_negation_check">
**Bloated:**
```javascript
if (!!value) { ... }
```

**Minimal:**
```javascript
if (value) { ... }
```

Use `!!` only when you need an actual boolean (rare).
</pattern>

<pattern name="array_spread_for_copy">
**Bloated:**
```javascript
const copy = Array.from(original);
```

**Minimal:**
```javascript
const copy = [...original];
```
</pattern>

<pattern name="object_assign_for_merge">
**Bloated:**
```javascript
const merged = Object.assign({}, defaults, options);
```

**Minimal:**
```javascript
const merged = { ...defaults, ...options };
```
</pattern>

<pattern name="verbose_default_parameters">
**Bloated:**
```javascript
function greet(name) {
  name = name || 'World';
  return `Hello, ${name}`;
}
```

**Minimal:**
```javascript
function greet(name = 'World') {
  return `Hello, ${name}`;
}
```
</pattern>

<pattern name="unnecessary_bind">
**Bloated:**
```javascript
this.handleClick = this.handleClick.bind(this);
```

**Minimal (use arrow function):**
```javascript
handleClick = () => { ... }
```
</pattern>

</anti_patterns>

<idioms>

<idiom name="nullish_coalescing">
Use `??` for null/undefined defaults, `||` for all falsy:
```javascript
const value = input ?? 'default';  // only null/undefined
const value = input || 'default';  // all falsy (0, '', false)
```
</idiom>

<idiom name="optional_chaining">
Chain safely through potentially undefined:
```javascript
const name = user?.profile?.name;
const first = arr?.[0];
const result = obj?.method?.();
```
</idiom>

<idiom name="array_methods">
Prefer declarative over imperative:
- `map` - transform each element
- `filter` - keep matching elements
- `find` - get first match
- `some` - check if any match
- `every` - check if all match
- `reduce` - accumulate to single value
</idiom>

<idiom name="object_methods">
```javascript
Object.keys(obj)    // array of keys
Object.values(obj)  // array of values
Object.entries(obj) // array of [key, value]
Object.fromEntries(entries) // back to object
```
</idiom>

<idiom name="modern_async">
Prefer async/await over .then() chains:
```javascript
// Not this
fetch(url).then(r => r.json()).then(data => process(data));

// This
const response = await fetch(url);
const data = await response.json();
process(data);
```
</idiom>

</idioms>

<typescript_specific>

<pattern name="unnecessary_type_assertion">
**Bloated:**
```typescript
const name = (user as User).name;
```

**Minimal (if type is already known):**
```typescript
const name = user.name;
```
</pattern>

<pattern name="verbose_interface">
**Bloated:**
```typescript
interface Props {
  name: string;
  age: number;
}
function Component(props: Props) {
  const name = props.name;
  const age = props.age;
}
```

**Minimal:**
```typescript
function Component({ name, age }: { name: string; age: number }) {
  // use directly
}
```
</pattern>

</typescript_specific>
