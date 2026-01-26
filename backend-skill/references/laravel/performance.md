<overview>
Optimizing Laravel applications with Spatie permissions. Covers N+1 prevention, eager loading, and query optimization.
</overview>

<n_plus_one_prevention>
**The Problem:**

```php
// BAD - N+1 queries
@foreach ($users as $user)
    {{ $user->getRoleNames()->implode(', ') }}
@endforeach
```

**Solution: Eager Load**

```php
$users = User::with('roles')->get();

@foreach ($users as $user)
    {{ $user->roles->pluck('name')->implode(', ') }}
@endforeach
```
</n_plus_one_prevention>

<eager_loading_patterns>
```php
// Load roles
$users = User::with('roles')->get();

// Load permissions
$users = User::with('permissions')->get();

// Load roles with their permissions
$users = User::with('roles.permissions')->get();

// Select specific fields
$users = User::with('roles:id,name')->get();

// Lazy eager loading
$users = User::all();
$users->load('roles');

// Load missing (prevents double-loading)
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

**Use Laravel Debugbar:**

```bash
composer require barryvdh/laravel-debugbar --dev
```
</detect_n_plus_one>

<query_optimization>
```php
// GOOD - Database-level filtering
$editors = User::permission('edit articles')->get();

// LESS EFFICIENT - Application-level filtering
$editors = User::all()->filter(fn ($user) => $user->can('edit articles'));

// Combine scopes
$activeEditors = User::permission('edit articles')
    ->where('is_active', true)
    ->get();

// Select only needed columns
$users = User::with('roles:id,name')
    ->select(['id', 'name', 'email'])
    ->get();
```
</query_optimization>

<blade_optimization>
**Minimize permission checks in views:**

```php
// Controller - pre-compute permissions
$permissions = [
    'canView' => $user->can('view articles'),
    'canEdit' => $user->can('edit articles'),
    'canDelete' => $user->can('delete articles'),
];
return view('articles.index', compact('permissions'));

// View
@if($permissions['canEdit'])
    // ...
@endif
```
</blade_optimization>

<menu_caching>
```php
$menuItems = Cache::remember(
    "menu.user.{$user->id}",
    now()->addMinutes(15),
    function () use ($user) {
        return collect([
            ['route' => 'dashboard', 'label' => 'Dashboard', 'permission' => null],
            ['route' => 'articles.index', 'label' => 'Articles', 'permission' => 'view articles'],
            ['route' => 'admin.settings', 'label' => 'Settings', 'permission' => 'manage settings'],
        ])->filter(function ($item) use ($user) {
            return !$item['permission'] || $user->can($item['permission']);
        });
    }
);

// Invalidate when roles change
public function pivotAttached(User $user, string $relation): void
{
    if ($relation === 'roles') {
        Cache::forget("menu.user.{$user->id}");
    }
}
```
</menu_caching>

<profiling>
```php
// Enable query log
DB::enableQueryLog();

$user->getAllPermissions();

dd(DB::getQueryLog());
```

Use Clockwork or Debugbar to monitor:
- Number of queries
- Query time
- N+1 detection
- Cache hits/misses
</profiling>
