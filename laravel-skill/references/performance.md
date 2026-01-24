<overview>
Optimizing Laravel applications with Spatie permissions. Covers N+1 prevention, eager loading, caching strategies, and query optimization.
</overview>

<n_plus_one_prevention>
**The Problem:**

```php
// BAD - N+1 queries
@foreach ($users as $user)
    {{ $user->getRoleNames()->implode(', ') }}
@endforeach
```

Each iteration queries the database for roles.

**Solution: Eager Load Relationships**

```php
// GOOD - Single query
$users = User::with('roles')->get();

@foreach ($users as $user)
    {{ $user->roles->pluck('name')->implode(', ') }}
@endforeach
```
</n_plus_one_prevention>

<eager_loading_patterns>
**Load roles:**

```php
$users = User::with('roles')->get();
```

**Load permissions:**

```php
$users = User::with('permissions')->get();
```

**Load roles with their permissions:**

```php
$users = User::with('roles.permissions')->get();
```

**Load specific fields only:**

```php
$users = User::with('roles:id,name')->get();
```

**Lazy eager loading (after initial query):**

```php
$users = User::all();
$users->load('roles'); // One additional query
```

**Load missing (prevents double-loading):**

```php
$users->loadMissing('roles');
```
</eager_loading_patterns>

<detect_n_plus_one>
**Enable strict mode (development):**

```php
// app/Providers/AppServiceProvider.php
use Illuminate\Database\Eloquent\Model;

public function boot(): void
{
    Model::preventLazyLoading(!app()->isProduction());
}
```

Throws exception when lazy loading occurs in development.

**Use Laravel Debugbar:**

```bash
composer require barryvdh/laravel-debugbar --dev
```

Shows all queries with timing.
</detect_n_plus_one>

<automatic_eager_loading>
**Laravel 12.8+ feature:**

Automatic eager loading helps prevent N+1 without manual `with()` calls.

```php
// config/database.php
'prevent_lazy_loading' => env('DB_PREVENT_LAZY_LOADING', false),
```

For Spatie specifically, still prefer explicit eager loading for control.
</automatic_eager_loading>

<caching_strategies>
**Package caching (built-in):**

Spatie caches all permissions for 24 hours by default. Configure in `config/permission.php`.

**Custom caching for expensive queries:**

```php
// Cache user's permissions for the request
public function getUserPermissions(User $user): Collection
{
    return Cache::remember(
        "user.{$user->id}.permissions",
        now()->addMinutes(5),
        fn () => $user->getAllPermissions()
    );
}
```

**Invalidate on change:**

```php
// In observer or event listener
public function updated(User $user): void
{
    Cache::forget("user.{$user->id}.permissions");
}
```
</caching_strategies>

<query_optimization>
**Use permission scope efficiently:**

```php
// GOOD - Database-level filtering
$editors = User::permission('edit articles')->get();

// LESS EFFICIENT - Application-level filtering
$editors = User::all()->filter(fn ($user) => $user->can('edit articles'));
```

**Combine scopes:**

```php
$activeEditors = User::permission('edit articles')
    ->where('is_active', true)
    ->get();
```

**Select only needed columns:**

```php
$users = User::with('roles:id,name')
    ->select(['id', 'name', 'email'])
    ->get();
```
</query_optimization>

<hasPermissionTo_performance>
**Performance tip for frequent checks:**

When assigning permissions frequently, assign to role instead of querying:

```php
// More performant for bulk operations
$permission = Permission::findByName('edit articles');
$permission->assignRole($role);

// Instead of
$role->givePermissionTo('edit articles');
```

Both produce same result, but first can be faster in some scenarios.
</hasPermissionTo_performance>

<blade_optimization>
**Minimize permission checks in views:**

```php
// BAD - Multiple checks
@can('view articles')
    // ...
@endcan
@can('edit articles')
    // ...
@endcan
@can('delete articles')
    // ...
@endcan

// BETTER - Pre-compute in controller
// Controller:
$permissions = [
    'canView' => $user->can('view articles'),
    'canEdit' => $user->can('edit articles'),
    'canDelete' => $user->can('delete articles'),
];
return view('articles.index', compact('permissions'));

// View:
@if($permissions['canEdit'])
    // ...
@endif
```
</blade_optimization>

<menu_caching>
**Cache menu/navigation that depends on permissions:**

```php
// In view composer or controller
public function compose(View $view): void
{
    $user = auth()->user();

    $menuItems = Cache::remember(
        "menu.user.{$user->id}",
        now()->addMinutes(15),
        function () use ($user) {
            return collect([
                ['route' => 'dashboard', 'label' => 'Dashboard', 'permission' => null],
                ['route' => 'articles.index', 'label' => 'Articles', 'permission' => 'view articles'],
                ['route' => 'users.index', 'label' => 'Users', 'permission' => 'view users'],
                ['route' => 'admin.settings', 'label' => 'Settings', 'permission' => 'manage settings'],
            ])->filter(function ($item) use ($user) {
                return !$item['permission'] || $user->can($item['permission']);
            });
        }
    );

    $view->with('menuItems', $menuItems);
}
```

**Invalidate menu cache when roles change:**

```php
// In observer
public function pivotAttached(User $user, string $relation): void
{
    if ($relation === 'roles') {
        Cache::forget("menu.user.{$user->id}");
    }
}
```
</menu_caching>

<database_indexing>
**Ensure proper indexes:**

The Spatie migration creates indexes, but verify:

```php
Schema::table('model_has_roles', function (Blueprint $table) {
    $table->index(['model_id', 'model_type']);
});

Schema::table('model_has_permissions', function (Blueprint $table) {
    $table->index(['model_id', 'model_type']);
});
```
</database_indexing>

<profiling>
**Profile permission queries:**

```php
// Enable query log
DB::enableQueryLog();

// Your code
$user->getAllPermissions();

// Check queries
dd(DB::getQueryLog());
```

**Using Clockwork or Debugbar:**

Both tools show:
- Number of queries
- Query time
- N+1 detection
- Cache hits/misses
</profiling>
