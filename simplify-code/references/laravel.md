<overview>
Laravel-specific anti-patterns and simplification patterns. Use alongside php.md for Laravel applications.
</overview>

<anti_patterns>

<pattern name="manual_request_access">
**Bloated:**
```php
public function store(Request $request) {
    $name = $request->input('name');
    $email = $request->input('email');
    User::create(['name' => $name, 'email' => $email]);
}
```

**Minimal:**
```php
public function store(Request $request) {
    User::create($request->only(['name', 'email']));
}
```

Or with validated data:
```php
public function store(StoreUserRequest $request) {
    User::create($request->validated());
}
```
</pattern>

<pattern name="verbose_query_builder">
**Bloated:**
```php
$users = User::query()
    ->where('active', '=', true)
    ->where('role', '=', 'admin')
    ->get();
```

**Minimal:**
```php
$users = User::where('active', true)
    ->where('role', 'admin')
    ->get();
```

Or with scope:
```php
$users = User::active()->admins()->get();
```
</pattern>

<pattern name="n_plus_one_queries">
**Bloated:**
```php
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name;  // N queries!
}
```

**Minimal:**
```php
$posts = Post::with('author')->get();
foreach ($posts as $post) {
    echo $post->author->name;  // 2 queries total
}
```
</pattern>

<pattern name="manual_null_check_on_relationship">
**Bloated:**
```php
if ($user->profile !== null) {
    return $user->profile->avatar;
}
return null;
```

**Minimal:**
```php
return $user->profile?->avatar;
```

Or with optional helper:
```php
return optional($user->profile)->avatar;
```
</pattern>

<pattern name="verbose_collection_operations">
**Bloated:**
```php
$names = [];
foreach ($users as $user) {
    if ($user->active) {
        $names[] = $user->name;
    }
}
```

**Minimal:**
```php
$names = $users->where('active', true)->pluck('name');
```
</pattern>

<pattern name="manual_response_building">
**Bloated:**
```php
return response()->json([
    'data' => $user,
    'status' => 'success',
], 200);
```

**Minimal:**
```php
return response()->json($user);
```

Status 200 is default. Wrapper structure often unnecessary.
</pattern>

<pattern name="overly_verbose_validation">
**Bloated:**
```php
public function store(Request $request) {
    $validator = Validator::make($request->all(), [
        'name' => 'required|string|max:255',
        'email' => 'required|email',
    ]);

    if ($validator->fails()) {
        return redirect()->back()->withErrors($validator);
    }

    // process...
}
```

**Minimal:**
```php
public function store(Request $request) {
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'email' => 'required|email',
    ]);

    // process with $validated
}
```

Or use Form Request for reusability.
</pattern>

<pattern name="redundant_model_events">
**Bloated (in controller):**
```php
$user = User::create($data);
event(new UserCreated($user));
Cache::forget('users');
Log::info('User created', ['id' => $user->id]);
```

**Minimal (use observers):**
```php
// In UserObserver
public function created(User $user) {
    event(new UserCreated($user));
    Cache::forget('users');
    Log::info('User created', ['id' => $user->id]);
}

// In controller
$user = User::create($data);
```
</pattern>

<pattern name="verbose_route_definitions">
**Bloated:**
```php
Route::get('/users', [UserController::class, 'index']);
Route::get('/users/{user}', [UserController::class, 'show']);
Route::post('/users', [UserController::class, 'store']);
Route::put('/users/{user}', [UserController::class, 'update']);
Route::delete('/users/{user}', [UserController::class, 'destroy']);
```

**Minimal:**
```php
Route::apiResource('users', UserController::class);
```
</pattern>

<pattern name="manual_model_binding">
**Bloated:**
```php
public function show($id) {
    $user = User::findOrFail($id);
    return view('users.show', ['user' => $user]);
}
```

**Minimal:**
```php
public function show(User $user) {
    return view('users.show', compact('user'));
}
```
</pattern>

<pattern name="string_config_access">
**Bloated:**
```php
$apiKey = config('services.stripe.key');
if ($apiKey === null) {
    throw new Exception('Missing API key');
}
```

**Minimal:**
```php
$apiKey = config('services.stripe.key') ?? throw new Exception('Missing API key');
```
</pattern>

</anti_patterns>

<idioms>

<idiom name="eloquent_query_scopes">
Define reusable query constraints:
```php
// In model
public function scopeActive($query) {
    return $query->where('active', true);
}

// Usage
User::active()->get();
```
</idiom>

<idiom name="collection_methods">
```php
$users->pluck('name')           // extract single field
$users->where('active', true)   // filter
$users->map(fn($u) => ...)      // transform
$users->first()                 // get first
$users->groupBy('role')         // group
$users->sortBy('name')          // sort
$users->unique('email')         // dedupe
```
</idiom>

<idiom name="when_conditional">
```php
User::query()
    ->when($request->status, fn($q, $status) => $q->where('status', $status))
    ->when($request->role, fn($q, $role) => $q->where('role', $role))
    ->get();
```
</idiom>

<idiom name="compact_for_views">
```php
return view('users.show', compact('user', 'posts', 'comments'));
```
</idiom>

<idiom name="route_model_binding">
Type-hint the model in controller methods:
```php
public function show(User $user) { ... }
public function update(Request $request, User $user) { ... }
```
</idiom>

<idiom name="mass_assignment">
With fillable defined:
```php
User::create($request->validated());
$user->update($request->validated());
```
</idiom>

<idiom name="relationships">
```php
$user->posts                    // HasMany
$post->author                   // BelongsTo
$user->roles                    // BelongsToMany
$country->posts                 // HasManyThrough
```
</idiom>

</idioms>

<blade_patterns>

<pattern name="verbose_echo">
**Bloated:**
```blade
<?php echo htmlspecialchars($name); ?>
```

**Minimal:**
```blade
{{ $name }}
```
</pattern>

<pattern name="verbose_conditionals">
**Bloated:**
```blade
@if(count($users) > 0)
    @foreach($users as $user)
        ...
    @endforeach
@else
    <p>No users</p>
@endif
```

**Minimal:**
```blade
@forelse($users as $user)
    ...
@empty
    <p>No users</p>
@endforelse
```
</pattern>

</blade_patterns>
