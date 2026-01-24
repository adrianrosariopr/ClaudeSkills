# Workflow: Debug Permission Issues

<required_reading>
**Read these reference files NOW:**
1. references/caching.md
2. references/anti-patterns.md
3. references/architecture.md
</required_reading>

<process>
## Step 1: Identify the Problem

Common symptoms:
- **403 Forbidden** - User denied when they should have access
- **Intermittent 403s** - Works sometimes, fails other times
- **Works locally, fails in production** - Environment differences
- **Permission not found** - Exception when checking permissions

## Step 2: Clear All Caches

This fixes 80% of permission issues:

```bash
php artisan permission:cache-reset
php artisan config:clear
php artisan cache:clear
php artisan route:clear
```

Test again. If fixed, the issue was stale cache.

## Step 3: Verify User Has Permission

```bash
php artisan tinker
```

```php
use App\Models\User;

$user = User::find(YOUR_USER_ID);

// Check role
$user->getRoleNames();  // Should show assigned roles

// Check all permissions (direct + via roles)
$user->getAllPermissions()->pluck('name');

// Check specific permission
$user->can('your permission name');

// Check direct permissions only
$user->getDirectPermissions()->pluck('name');

// Check permissions via roles
$user->getPermissionsViaRoles()->pluck('name');
```

## Step 4: Verify Permission Exists

```php
use Spatie\Permission\Models\Permission;

// List all permissions
Permission::all()->pluck('name');

// Check specific permission exists
Permission::where('name', 'your permission name')->first();

// Check guard
Permission::where('name', 'your permission name')->first()->guard_name;
```

## Step 5: Check Guard Mismatch

**Common error:** "The given role or permission should use guard `web` instead of `sanctum`"

```php
// Check user's guard
$user = User::first();

// In User model, check if guard is forced
// Look for: protected $guard_name = 'web';

// Check permission guard
Permission::where('name', 'your permission')->first()->guard_name;
```

**Fix:** Force guard on User model:

```php
// app/Models/User.php
class User extends Authenticatable
{
    use HasRoles;

    protected $guard_name = 'web';
}
```

## Step 6: Verify Middleware Registration

```bash
php artisan route:list | grep middleware
```

Check `bootstrap/app.php` for middleware aliases:

```php
$middleware->alias([
    'role' => RoleMiddleware::class,
    'permission' => PermissionMiddleware::class,
    'role_or_permission' => RoleOrPermissionMiddleware::class,
]);
```

## Step 7: Check Route Protection

```bash
php artisan route:list
```

Verify the route has the expected middleware:

```
GET|HEAD  articles ......... permission:view articles
```

## Step 8: Test in Isolation

Create a test route:

```php
// routes/web.php (temporarily)
Route::get('/debug-permission', function () {
    $user = auth()->user();

    return [
        'user_id' => $user->id,
        'roles' => $user->getRoleNames(),
        'all_permissions' => $user->getAllPermissions()->pluck('name'),
        'can_view_articles' => $user->can('view articles'),
        'has_role_admin' => $user->hasRole('admin'),
    ];
})->middleware('auth');
```

## Step 9: Check Database Directly

```sql
-- Check permissions table
SELECT * FROM permissions;

-- Check roles table
SELECT * FROM roles;

-- Check user's roles
SELECT * FROM model_has_roles WHERE model_id = YOUR_USER_ID;

-- Check role's permissions
SELECT p.name
FROM permissions p
JOIN role_has_permissions rp ON p.id = rp.permission_id
JOIN roles r ON r.id = rp.role_id
WHERE r.name = 'your-role';

-- Check user's direct permissions
SELECT p.name
FROM permissions p
JOIN model_has_permissions mp ON p.id = mp.permission_id
WHERE mp.model_id = YOUR_USER_ID;
```

## Step 10: Common Issues and Fixes

**Issue: Cache not clearing**

```php
// Force cache clear programmatically
app()[\Spatie\Permission\PermissionRegistrar::class]->forgetCachedPermissions();

// Check cache driver
// config/cache.php - if using 'array', cache doesn't persist
```

**Issue: Permission created but not found**

```php
// Created without guard, defaults may differ
Permission::create(['name' => 'new permission']);  // Uses default guard

// Specify guard explicitly
Permission::create(['name' => 'new permission', 'guard_name' => 'web']);
```

**Issue: Works in tinker, fails in HTTP**

Session/auth context differs:
```php
// In controller, check actual authenticated user
dd(auth()->user()?->id, auth()->guard()->name);
```

**Issue: Seeder ran but permissions missing**

```php
// Check if seeder actually ran
// Look at permissions table count

// Re-run seeder
php artisan db:seed --class=RolesAndPermissionsSeeder

// Don't forget cache reset
php artisan permission:cache-reset
```

**Issue: N+1 causing slow permission checks**

```php
// Eager load relationships
$users = User::with('roles', 'permissions')->get();
```

## Step 11: Enable Debug Logging

Temporarily add to controller:

```php
use Illuminate\Support\Facades\Log;

public function index()
{
    $user = auth()->user();

    Log::info('Permission debug', [
        'user_id' => $user->id,
        'roles' => $user->getRoleNames()->toArray(),
        'can_access' => $user->can('view articles'),
        'guard' => auth()->guard()->name ?? 'none',
    ]);

    // ... rest of method
}
```

Check `storage/logs/laravel.log` for output.
</process>

<checklist>
**Debug Checklist:**

- [ ] Cache cleared (`permission:cache-reset`, `config:clear`, `cache:clear`)
- [ ] Permission exists in database
- [ ] User has role that has permission (or direct permission)
- [ ] Guard names match (user, permission, middleware)
- [ ] Middleware registered in `bootstrap/app.php`
- [ ] Route has correct middleware applied
- [ ] Tested in tinker successfully
- [ ] Tested via HTTP request
</checklist>

<success_criteria>
Issue resolved when:
- [ ] User can access protected resources
- [ ] Permission checks return expected boolean
- [ ] No 403 errors for authorized users
- [ ] No exceptions thrown
- [ ] Debug logging shows correct permissions
</success_criteria>
