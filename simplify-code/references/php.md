<overview>
PHP anti-patterns and simplification patterns. Follows PSR-12 style. Applies to vanilla PHP and frameworks.
</overview>

<anti_patterns>

<pattern name="verbose_null_check">
**Bloated:**
```php
if ($user !== null && $user->profile !== null) {
    return $user->profile->name;
}
return 'Unknown';
```

**Minimal (PHP 8+):**
```php
return $user?->profile?->name ?? 'Unknown';
```
</pattern>

<pattern name="isset_ternary">
**Bloated:**
```php
$name = isset($data['name']) ? $data['name'] : 'default';
```

**Minimal:**
```php
$name = $data['name'] ?? 'default';
```
</pattern>

<pattern name="verbose_array_loop">
**Bloated:**
```php
$names = [];
foreach ($users as $user) {
    $names[] = $user->name;
}
```

**Minimal:**
```php
$names = array_map(fn($u) => $u->name, $users);
```

Or with collection:
```php
$names = collect($users)->pluck('name');
```
</pattern>

<pattern name="nested_if_else">
**Bloated:**
```php
if ($status === 'active') {
    if ($type === 'admin') {
        return 'admin-active';
    } else {
        return 'user-active';
    }
} else {
    return 'inactive';
}
```

**Minimal:**
```php
if ($status !== 'active') return 'inactive';
return $type === 'admin' ? 'admin-active' : 'user-active';
```
</pattern>

<pattern name="unnecessary_else_after_return">
**Bloated:**
```php
if ($condition) {
    return $a;
} else {
    return $b;
}
```

**Minimal:**
```php
if ($condition) return $a;
return $b;
```
</pattern>

<pattern name="verbose_string_concat">
**Bloated:**
```php
$message = 'Hello, ' . $name . '! You have ' . $count . ' messages.';
```

**Minimal:**
```php
$message = "Hello, {$name}! You have {$count} messages.";
```
</pattern>

<pattern name="array_push_single">
**Bloated:**
```php
array_push($items, $newItem);
```

**Minimal:**
```php
$items[] = $newItem;
```
</pattern>

<pattern name="verbose_constructor_assignment">
**Bloated:**
```php
class User {
    private string $name;
    private string $email;

    public function __construct(string $name, string $email) {
        $this->name = $name;
        $this->email = $email;
    }
}
```

**Minimal (PHP 8+):**
```php
class User {
    public function __construct(
        private string $name,
        private string $email,
    ) {}
}
```
</pattern>

<pattern name="manual_array_filter">
**Bloated:**
```php
$active = [];
foreach ($users as $user) {
    if ($user->active) {
        $active[] = $user;
    }
}
```

**Minimal:**
```php
$active = array_filter($users, fn($u) => $u->active);
```
</pattern>

<pattern name="unnecessary_variable">
**Bloated:**
```php
$result = $this->processData($input);
return $result;
```

**Minimal:**
```php
return $this->processData($input);
```
</pattern>

<pattern name="verbose_boolean_expression">
**Bloated:**
```php
if ($value === true) {
    return true;
} else {
    return false;
}
```

**Minimal:**
```php
return $value === true;
```

Or if already boolean:
```php
return $value;
```
</pattern>

<pattern name="empty_check_variations">
**Bloated:**
```php
if (count($array) > 0) { ... }
if (strlen($string) > 0) { ... }
```

**Minimal:**
```php
if ($array) { ... }
if ($string) { ... }
```

Or use empty() for explicit intent:
```php
if (!empty($array)) { ... }
```
</pattern>

<pattern name="multiple_return_statements">
**Bloated:**
```php
public function getStatus($code) {
    switch ($code) {
        case 200:
            return 'ok';
            break;
        case 404:
            return 'not found';
            break;
        default:
            return 'error';
            break;
    }
}
```

**Minimal (PHP 8+):**
```php
public function getStatus($code) {
    return match ($code) {
        200 => 'ok',
        404 => 'not found',
        default => 'error',
    };
}
```
</pattern>

</anti_patterns>

<idioms>

<idiom name="null_coalescing">
```php
$value = $input ?? 'default';           // if null
$value = $input ?: 'default';           // if falsy
$value ??= 'default';                   // assign if null
```
</idiom>

<idiom name="nullsafe_operator">
```php
$name = $user?->profile?->name;         // returns null if any is null
$result = $obj?->method()?->property;
```
</idiom>

<idiom name="arrow_functions">
```php
$doubled = array_map(fn($n) => $n * 2, $numbers);
$filtered = array_filter($items, fn($i) => $i->active);
```
</idiom>

<idiom name="named_arguments">
```php
// Instead of positional with nulls
createUser(null, null, 'admin');

// Named arguments
createUser(role: 'admin');
```
</idiom>

<idiom name="match_expression">
```php
$result = match ($status) {
    'draft' => 'Draft',
    'published', 'active' => 'Live',
    default => 'Unknown',
};
```
</idiom>

<idiom name="array_functions">
```php
array_map()     // transform each
array_filter()  // keep matching
array_reduce()  // accumulate
array_column()  // extract column
array_combine() // keys + values
array_merge()   // combine arrays
```
</idiom>

<idiom name="spread_operator">
```php
$merged = [...$array1, ...$array2];
function sum(...$numbers) { return array_sum($numbers); }
```
</idiom>

</idioms>

<psr12_essentials>
- One class per file
- Opening brace on same line for control structures, new line for classes/methods
- Use strict types: `declare(strict_types=1);`
- Type declarations for parameters and returns
- No closing `?>` tag in pure PHP files
</psr12_essentials>
