<overview>
Python anti-patterns and simplification patterns. Follows PEP 8 style. Applies to Python 3.8+.
</overview>

<anti_patterns>

<pattern name="manual_list_building">
**Bloated:**
```python
squares = []
for x in range(10):
    squares.append(x ** 2)
```

**Minimal:**
```python
squares = [x ** 2 for x in range(10)]
```
</pattern>

<pattern name="manual_dict_building">
**Bloated:**
```python
user_map = {}
for user in users:
    user_map[user.id] = user.name
```

**Minimal:**
```python
user_map = {u.id: u.name for u in users}
```
</pattern>

<pattern name="verbose_conditional_assignment">
**Bloated:**
```python
if value is not None:
    result = value
else:
    result = default
```

**Minimal:**
```python
result = value if value is not None else default
```

Or with walrus operator:
```python
result = value or default  # if falsy ok
```
</pattern>

<pattern name="unnecessary_else_after_return">
**Bloated:**
```python
def get_status(code):
    if code == 200:
        return 'ok'
    else:
        return 'error'
```

**Minimal:**
```python
def get_status(code):
    if code == 200:
        return 'ok'
    return 'error'
```
</pattern>

<pattern name="verbose_boolean_check">
**Bloated:**
```python
if len(items) > 0:
    process(items)
```

**Minimal:**
```python
if items:
    process(items)
```
</pattern>

<pattern name="verbose_none_check">
**Bloated:**
```python
if value == None:
    ...
```

**Minimal:**
```python
if value is None:
    ...
```
</pattern>

<pattern name="manual_string_join">
**Bloated:**
```python
result = ''
for word in words:
    result += word + ' '
result = result.strip()
```

**Minimal:**
```python
result = ' '.join(words)
```
</pattern>

<pattern name="verbose_file_handling">
**Bloated:**
```python
f = open('file.txt', 'r')
content = f.read()
f.close()
```

**Minimal:**
```python
with open('file.txt') as f:
    content = f.read()
```
</pattern>

<pattern name="manual_enumerate">
**Bloated:**
```python
i = 0
for item in items:
    print(i, item)
    i += 1
```

**Minimal:**
```python
for i, item in enumerate(items):
    print(i, item)
```
</pattern>

<pattern name="verbose_dict_get">
**Bloated:**
```python
if 'key' in data:
    value = data['key']
else:
    value = 'default'
```

**Minimal:**
```python
value = data.get('key', 'default')
```
</pattern>

<pattern name="multiple_isinstance">
**Bloated:**
```python
if isinstance(x, int) or isinstance(x, float):
    ...
```

**Minimal:**
```python
if isinstance(x, (int, float)):
    ...
```
</pattern>

<pattern name="verbose_swap">
**Bloated:**
```python
temp = a
a = b
b = temp
```

**Minimal:**
```python
a, b = b, a
```
</pattern>

<pattern name="unnecessary_list_call">
**Bloated:**
```python
for item in list(dict.keys()):
    ...
```

**Minimal:**
```python
for key in dict:
    ...
```
</pattern>

<pattern name="verbose_default_argument">
**Bloated:**
```python
def greet(name):
    if name is None:
        name = 'World'
    return f'Hello, {name}'
```

**Minimal:**
```python
def greet(name='World'):
    return f'Hello, {name}'
```
</pattern>

<pattern name="redundant_return_none">
**Bloated:**
```python
def process():
    do_something()
    return None
```

**Minimal:**
```python
def process():
    do_something()
```
</pattern>

<pattern name="verbose_tuple_unpacking">
**Bloated:**
```python
point = (3, 4)
x = point[0]
y = point[1]
```

**Minimal:**
```python
x, y = (3, 4)
```
</pattern>

<pattern name="chained_or_for_default">
**Bloated:**
```python
if a:
    value = a
elif b:
    value = b
else:
    value = c
```

**Minimal:**
```python
value = a or b or c
```
</pattern>

</anti_patterns>

<idioms>

<idiom name="comprehensions">
```python
# List
[x * 2 for x in items]
[x for x in items if x > 0]

# Dict
{k: v for k, v in pairs}
{x: x**2 for x in range(5)}

# Set
{x for x in items}

# Generator (memory efficient)
(x * 2 for x in items)
```
</idiom>

<idiom name="unpacking">
```python
first, *rest = items           # first and rest
*head, last = items            # all but last
a, b, c = tuple_of_three       # exact unpack
```
</idiom>

<idiom name="walrus_operator">
```python
if (n := len(items)) > 10:
    print(f'{n} items')

while (line := file.readline()):
    process(line)
```
</idiom>

<idiom name="f_strings">
```python
f'Hello, {name}'
f'{value:.2f}'                 # formatting
f'{items=}'                    # debug (shows "items=[...]")
```
</idiom>

<idiom name="dict_methods">
```python
dict.get(key, default)         # safe access
dict.setdefault(key, default)  # get or set
dict.items()                   # iterate pairs
dict.keys()                    # iterate keys
dict.values()                  # iterate values
{**d1, **d2}                   # merge (3.9+: d1 | d2)
```
</idiom>

<idiom name="builtin_functions">
```python
any(x > 0 for x in items)      # at least one true
all(x > 0 for x in items)      # all true
sum(items)                     # total
max(items), min(items)         # extremes
sorted(items, key=lambda x: x.name)  # sort
zip(list1, list2)              # pair up
map(fn, items)                 # transform
filter(fn, items)              # keep matching
```
</idiom>

<idiom name="context_managers">
```python
with open('file.txt') as f:
    ...

with db.transaction():
    ...

# Multiple
with open('a') as a, open('b') as b:
    ...
```
</idiom>

<idiom name="type_hints">
```python
def process(items: list[str]) -> dict[str, int]:
    ...

# Optional
from typing import Optional
def find(id: int) -> Optional[User]:
    ...

# Union (3.10+)
def parse(data: str | bytes) -> dict:
    ...
```
</idiom>

</idioms>

<pep8_essentials>
- 4 spaces for indentation
- 79 character line limit (99 for modern projects)
- snake_case for functions and variables
- PascalCase for classes
- UPPER_CASE for constants
- Two blank lines between top-level definitions
- One blank line between methods
</pep8_essentials>
