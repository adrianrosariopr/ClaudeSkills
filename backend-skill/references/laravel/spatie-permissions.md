<overview>
Spatie laravel-permission for role-based access control. Check permissions (not roles) in application code. Combine with Laravel policies for clean authorization.
</overview>

<core_philosophy>
**Check permissions, not roles.** Your application code should test for specific permissions, not role names.

```php
// GOOD - checks specific capability
if ($user->can('edit articles')) { ... }

// BAD - couples code to role names
if ($user->hasRole('Editor')) { ... }
```

This allows you to:
- Change role definitions without changing application code
- Treat permission names as static (developer-controlled)
- Use Laravel's native `@can` and `can()` everywhere
</core_philosophy>

<creating_permissions>
```php
use Spatie\Permission\Models\Permission;

// Create single permission
Permission::create(['name' => 'edit articles']);

// Create multiple permissions
$permissions = ['view articles', 'create articles', 'edit articles', 'delete articles'];
foreach ($permissions as $permission) {
    Permission::create(['name' => $permission]);
}

// With guard (for APIs)
Permission::create(['name' => 'edit articles', 'guard_name' => 'api']);
```
</creating_permissions>

<creating_roles>
```php
use Spatie\Permission\Models\Role;

// Create role
$role = Role::create(['name' => 'editor']);

// Give permissions to role
$role->givePermissionTo('edit articles');
$role->givePermissionTo(['view articles', 'create articles']);

// Sync permissions (replaces all)
$role->syncPermissions(['view articles', 'edit articles']);

// Revoke permission
$role->revokePermissionTo('delete articles');
```
</creating_roles>

<assigning_to_users>
```php
// Assign role
$user->assignRole('editor');
$user->assignRole(['editor', 'writer']);

// Remove role
$user->removeRole('editor');

// Sync roles (replaces all)
$user->syncRoles(['editor']);

// Give direct permission (bypasses roles)
$user->givePermissionTo('delete articles');

// Revoke direct permission
$user->revokePermissionTo('delete articles');
```
</assigning_to_users>

<checking_permissions>
```php
// Check single permission
$user->hasPermissionTo('edit articles');
$user->can('edit articles');  // Laravel native

// Check any permission
$user->hasAnyPermission(['edit articles', 'delete articles']);

// Check all permissions
$user->hasAllPermissions(['edit articles', 'delete articles']);

// Get all permissions (direct + via roles)
$user->getAllPermissions();

// Get direct permissions only
$user->getDirectPermissions();

// Get permissions via roles
$user->getPermissionsViaRoles();
```
</checking_permissions>

<blade_directives>
```blade
{{-- Check permission --}}
@can('edit articles')
    <a href="{{ route('articles.edit', $article) }}">Edit</a>
@endcan

{{-- Check role (use sparingly) --}}
@role('admin')
    <a href="{{ route('admin.dashboard') }}">Admin</a>
@endrole

{{-- Check any role --}}
@hasanyrole('admin|editor')
    <p>You have elevated access</p>
@endhasanyrole
```
</blade_directives>

<route_middleware>
```php
// Single permission
Route::get('/articles/create', [ArticleController::class, 'create'])
    ->middleware('permission:create articles');

// Multiple permissions (AND)
Route::get('/admin', [AdminController::class, 'index'])
    ->middleware('permission:manage users|manage settings');

// Role middleware
Route::get('/admin', [AdminController::class, 'index'])
    ->middleware('role:admin');

// Role or permission
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('role_or_permission:admin|view dashboard');
```
</route_middleware>

<policies_with_spatie>
**Create policy:**

```bash
php artisan make:policy ArticlePolicy --model=Article
```

**Policy with Spatie permissions:**

```php
class ArticlePolicy
{
    public function before(User $user, string $ability): ?bool
    {
        if ($user->hasRole('super-admin')) {
            return true;
        }
        return null;
    }

    public function view(User $user, Article $article): bool
    {
        return $user->can('view articles');
    }

    public function update(User $user, Article $article): bool
    {
        return $user->can('edit articles') || $user->id === $article->user_id;
    }

    public function delete(User $user, Article $article): bool
    {
        return $user->can('delete articles');
    }
}
```

**Using policies:**

```php
// In controller
$this->authorize('update', $article);

// In Blade
@can('update', $article)
    <a href="{{ route('articles.edit', $article) }}">Edit</a>
@endcan
```
</policies_with_spatie>

<gates_with_spatie>
```php
// In AuthServiceProvider
Gate::define('access-admin', fn (User $user) => $user->can('access admin'));

// Super admin bypass
Gate::before(function (User $user, string $ability) {
    if ($user->hasRole('super-admin')) {
        return true;
    }
});
```
</gates_with_spatie>

<role_hierarchy_patterns>
**Pattern 1: Flat Roles**
```php
$admin->givePermissionTo(['create', 'edit', 'delete', 'view']);
$editor->givePermissionTo(['create', 'edit', 'view']);
$viewer->givePermissionTo(['view']);
```

**Pattern 2: Hierarchical (via permissions)**
```php
$viewPerms = ['view articles', 'view comments'];
$editPerms = ['edit articles', 'edit comments'];
$adminPerms = ['delete articles', 'manage users'];

$viewer->givePermissionTo($viewPerms);
$editor->givePermissionTo([...$viewPerms, ...$editPerms]);
$admin->givePermissionTo([...$viewPerms, ...$editPerms, ...$adminPerms]);
```
</role_hierarchy_patterns>

<permission_naming>
Use consistent naming:

```
{action} {resource}

Examples:
- view articles
- create articles
- edit articles
- delete articles
- publish articles
- manage users
```
</permission_naming>

<query_scopes>
```php
// Get users with specific role
$editors = User::role('editor')->get();

// Get users with permission
$canEdit = User::permission('edit articles')->get();
```
</query_scopes>
