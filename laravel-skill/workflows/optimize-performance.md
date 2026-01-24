# Workflow: Optimize Permission Performance

<required_reading>
**Read these reference files NOW:**
1. references/performance.md
2. references/caching.md
3. references/anti-patterns.md
</required_reading>

<process>
## Step 1: Identify Performance Issues

Enable query logging to find N+1 problems:

```php
// app/Providers/AppServiceProvider.php
use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Facades\DB;

public function boot(): void
{
    // Prevent lazy loading in non-production
    Model::preventLazyLoading(!app()->isProduction());

    // Log slow queries
    if (!app()->isProduction()) {
        DB::listen(function ($query) {
            if ($query->time > 100) {
                logger()->warning('Slow query', [
                    'sql' => $query->sql,
                    'time' => $query->time,
                ]);
            }
        });
    }
}
```

Install Laravel Debugbar for development:

```bash
composer require barryvdh/laravel-debugbar --dev
```

## Step 2: Fix N+1 Queries

**Problem:** Loading users without eager loading roles/permissions

```php
// BAD - N+1 queries
$users = User::all();
@foreach ($users as $user)
    {{ $user->getRoleNames()->implode(', ') }}  // Query per user!
@endforeach
```

**Solution:** Eager load relationships

```php
// GOOD - 2 queries total
$users = User::with('roles')->get();
@foreach ($users as $user)
    {{ $user->roles->pluck('name')->implode(', ') }}
@endforeach
```

**Full eager loading:**

```php
// Load roles and their permissions
$users = User::with('roles.permissions')->get();

// Load user's direct permissions too
$users = User::with(['roles', 'permissions'])->get();
```

## Step 3: Optimize Controller Queries

**Before:**
```php
public function index()
{
    $users = User::paginate(20);
    return view('users.index', compact('users'));
}
```

**After:**
```php
public function index()
{
    $users = User::with('roles:id,name')
        ->select(['id', 'name', 'email', 'created_at'])
        ->paginate(20);

    return view('users.index', compact('users'));
}
```

## Step 4: Cache Expensive Permission Computations

```php
// app/Services/PermissionService.php
namespace App\Services;

use App\Models\User;
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\Cache;

class PermissionService
{
    public function getUserPermissions(User $user): Collection
    {
        return Cache::remember(
            "user.{$user->id}.permissions",
            now()->addMinutes(15),
            fn () => $user->getAllPermissions()
        );
    }

    public function clearUserCache(User $user): void
    {
        Cache::forget("user.{$user->id}.permissions");
    }
}
```

**Invalidate cache when roles change:**

```php
// app/Observers/UserObserver.php
class UserObserver
{
    public function pivotAttached(User $user, string $relation): void
    {
        if (in_array($relation, ['roles', 'permissions'])) {
            app(PermissionService::class)->clearUserCache($user);
        }
    }

    public function pivotDetached(User $user, string $relation): void
    {
        if (in_array($relation, ['roles', 'permissions'])) {
            app(PermissionService::class)->clearUserCache($user);
        }
    }
}
```

## Step 5: Optimize Menu/Navigation Rendering

**Problem:** Multiple permission checks in navigation

```blade
{{-- BAD - Many permission checks --}}
@can('view articles') ... @endcan
@can('create articles') ... @endcan
@can('view users') ... @endcan
```

**Solution:** Pre-compute permissions in view composer

```php
// app/Providers/ViewServiceProvider.php
use Illuminate\Support\Facades\View;

public function boot(): void
{
    View::composer('layouts.navigation', function ($view) {
        $user = auth()->user();

        if (!$user) {
            $view->with('nav', []);
            return;
        }

        $nav = Cache::remember(
            "nav.user.{$user->id}",
            now()->addMinutes(15),
            function () use ($user) {
                return [
                    'articles' => $user->can('view articles'),
                    'create_article' => $user->can('create articles'),
                    'users' => $user->can('view users'),
                    'admin' => $user->can('access admin'),
                ];
            }
        );

        $view->with('nav', $nav);
    });
}
```

```blade
{{-- GOOD - Single cache lookup --}}
@if($nav['articles'])
    <a href="{{ route('articles.index') }}">Articles</a>
@endif
@if($nav['admin'])
    <a href="{{ route('admin.index') }}">Admin</a>
@endif
```

## Step 6: Use Permission Scopes Efficiently

```php
// GOOD - Database-level filtering
$editors = User::permission('edit articles')->get();

// LESS EFFICIENT - Application-level filtering
$editors = User::all()->filter(fn ($u) => $u->can('edit articles'));
```

**Combine with other conditions:**

```php
$activeEditors = User::permission('edit articles')
    ->where('is_active', true)
    ->where('created_at', '>', now()->subYear())
    ->get();
```

## Step 7: Configure Spatie Caching

Ensure caching is properly configured:

```php
// config/permission.php
'cache' => [
    'expiration_time' => \DateInterval::createFromDateString('24 hours'),
    'key' => 'spatie.permission.cache',
    'store' => 'redis',  // Use Redis in production
],
```

## Step 8: Profile and Measure

**Before optimization:**
```bash
php artisan tinker
>>> DB::enableQueryLog();
>>> $users = User::paginate(20);
>>> foreach ($users as $u) { $u->getRoleNames(); }
>>> count(DB::getQueryLog());  // 21+ queries
```

**After optimization:**
```bash
>>> DB::enableQueryLog();
>>> $users = User::with('roles')->paginate(20);
>>> foreach ($users as $u) { $u->roles->pluck('name'); }
>>> count(DB::getQueryLog());  // 2 queries
```

## Step 9: Database Indexing

Verify indexes exist:

```php
// Check migrations or run SQL
Schema::table('model_has_roles', function (Blueprint $table) {
    $table->index(['model_id', 'model_type']);
});

Schema::table('model_has_permissions', function (Blueprint $table) {
    $table->index(['model_id', 'model_type']);
});
```

## Step 10: Use Redis for Production

```env
CACHE_STORE=redis
SESSION_DRIVER=redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```

Redis provides faster cache operations than file or database drivers.
</process>

<verification>
After optimization, verify:

```bash
# Check query count with Debugbar
# Should see minimal queries per page load

# Run performance test
php artisan tinker
>>> $start = microtime(true);
>>> User::with('roles')->take(100)->get()->each(fn($u) => $u->can('edit'));
>>> microtime(true) - $start;  // Should be < 0.1s
```
</verification>

<success_criteria>
Performance optimized when:
- [ ] No N+1 queries detected
- [ ] Eager loading used for all user lists
- [ ] Menu/navigation permissions cached
- [ ] Redis configured for production
- [ ] Query count minimal per page load
- [ ] Page load times acceptable
- [ ] Cache invalidation working correctly
</success_criteria>
