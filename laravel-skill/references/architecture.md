<overview>
Laravel 12 with Spatie laravel-permission provides a robust authorization architecture. Understanding when to use roles, permissions, policies, and gates is critical for building maintainable systems.
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

<authorization_layers>
**Layer 1: Gates**
Simple yes/no checkpoints for actions not tied to models.

```php
// In AuthServiceProvider
Gate::define('access-admin', fn (User $user) => $user->is_admin);

// Usage
if (Gate::allows('access-admin')) { ... }
```

**When to use:** Quick checks, non-model actions (admin dashboard access, feature flags).

**Layer 2: Policies**
Structured authorization for model-specific actions.

```php
// app/Policies/ArticlePolicy.php
public function update(User $user, Article $article): bool
{
    return $user->can('edit articles') || $user->id === $article->user_id;
}
```

**When to use:** CRUD operations, ownership checks, model-specific logic.

**Layer 3: Spatie Permissions**
Database-driven RBAC with dynamic role/permission management.

```php
// Assign permissions to roles
$role->givePermissionTo(['edit articles', 'delete articles']);

// Assign roles to users
$user->assignRole('editor');
```

**When to use:** Dynamic role management, admin UI for permissions, team-based access.
</authorization_layers>

<combining_approaches>
Best practice is to combine all three:

1. **Spatie** stores roles/permissions in database
2. **Policies** contain authorization logic (can reference Spatie permissions)
3. **Gates** handle simple, non-model checks

```php
// Policy using Spatie permission
class ArticlePolicy
{
    public function update(User $user, Article $article): bool
    {
        // Combine Spatie permission with ownership
        return $user->can('edit articles') || $user->id === $article->user_id;
    }
}
```

```php
// Gate using Spatie permission
Gate::define('manage-users', fn (User $user) => $user->can('manage users'));
```
</combining_approaches>

<project_structure>
Recommended structure for Laravel 12 with Spatie:

```
app/
├── Models/
│   └── User.php              # HasRoles trait
├── Policies/
│   ├── ArticlePolicy.php     # Model authorization
│   └── UserPolicy.php
├── Http/
│   ├── Middleware/
│   │   └── TeamMiddleware.php  # Optional: team context
│   └── Controllers/
database/
├── migrations/
│   └── xxxx_create_permissions_tables.php
├── seeders/
│   └── RolesAndPermissionsSeeder.php
config/
└── permission.php            # Spatie config
```
</project_structure>

<role_hierarchy_patterns>
**Pattern 1: Flat Roles**
Each role has independent permissions.

```php
$admin = Role::create(['name' => 'admin']);
$editor = Role::create(['name' => 'editor']);
$viewer = Role::create(['name' => 'viewer']);

$admin->givePermissionTo(['create', 'edit', 'delete', 'view']);
$editor->givePermissionTo(['create', 'edit', 'view']);
$viewer->givePermissionTo(['view']);
```

**Pattern 2: Hierarchical Roles (via permissions)**
Higher roles include lower role permissions.

```php
// Base permissions
$viewPerms = ['view articles', 'view comments'];
$editPerms = ['edit articles', 'edit comments'];
$adminPerms = ['delete articles', 'manage users'];

$viewer->givePermissionTo($viewPerms);
$editor->givePermissionTo([...$viewPerms, ...$editPerms]);
$admin->givePermissionTo([...$viewPerms, ...$editPerms, ...$adminPerms]);
```

**Pattern 3: Super Admin**
One role that bypasses all permission checks.

```php
// In AuthServiceProvider boot()
Gate::before(function (User $user, string $ability) {
    return $user->hasRole('super-admin') ? true : null;
});
```
</role_hierarchy_patterns>

<permission_naming>
Use consistent, descriptive permission names:

```
{action} {resource}

Examples:
- view articles
- create articles
- edit articles
- delete articles
- publish articles
- manage users
- access admin
```

Group by resource for clarity:
```php
Permission::create(['name' => 'articles.view']);
Permission::create(['name' => 'articles.create']);
Permission::create(['name' => 'articles.edit']);
Permission::create(['name' => 'articles.delete']);
```
</permission_naming>

<decision_tree>
**When to use what:**

If action is NOT related to a model → **Gate**
If action IS related to a model → **Policy**
If you need dynamic role/permission management → **Spatie**
If you need team-based permissions → **Spatie Teams**
If you need API token abilities → **Sanctum + Spatie**

**Can I combine them?** Yes - most production apps use all three together.
</decision_tree>
