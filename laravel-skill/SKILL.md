---
name: laravel-skill
description: Build Laravel 12 applications with Spatie laravel-permission from scratch through deployment. Full lifecycle coverage - scaffold, add permissions, debug, test, optimize, and ship. Expert in RBAC, policies, multi-tenancy, and API authentication.
---

<essential_principles>
**1. Check Permissions, Not Roles**

Your application code should test for specific permissions, not role names:

```php
// GOOD
if ($user->can('edit articles')) { ... }

// BAD
if ($user->hasRole('Editor')) { ... }
```

This allows role definitions to change without changing application code.

**2. Always Reset Cache After Permission Changes**

```php
app()[\Spatie\Permission\PermissionRegistrar::class]->forgetCachedPermissions();
```

Run this in seeders, migrations, and after any direct database operations.

**3. Combine Spatie + Policies + Gates**

- **Spatie** stores roles/permissions in database
- **Policies** contain model authorization logic (can use Spatie permissions)
- **Gates** handle simple, non-model checks

**4. Use Migrations for Production**

Never run seeders in production. Use migrations for permission changes - they're idempotent and tracked.

**5. Eager Load Relationships**

```php
// Prevent N+1 queries
$users = User::with('roles', 'permissions')->get();
```
</essential_principles>

<intake>
**What would you like to do?**

1. Build a new Laravel app with permissions
2. Add permissions to existing app
3. Debug permission issues (403 errors, cache problems)
4. Write tests for authorization
5. Optimize performance
6. Deploy/ship to production
7. Set up team-based permissions (multi-tenancy)
8. Add API authentication with permissions
9. Something else

**Wait for response, then load the matching workflow.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "new", "create", "scaffold", "start", "fresh" | `workflows/build-new-app.md` |
| 2, "add", "existing", "integrate", "install" | `workflows/add-permissions.md` |
| 3, "debug", "403", "forbidden", "cache", "not working", "broken" | `workflows/debug-permissions.md` |
| 4, "test", "tests", "testing", "phpunit", "pest" | `workflows/write-tests.md` |
| 5, "slow", "optimize", "performance", "n+1", "fast" | `workflows/optimize-performance.md` |
| 6, "deploy", "ship", "production", "release", "seed" | `workflows/ship-app.md` |
| 7, "team", "teams", "tenant", "multi-tenant", "organization" | `workflows/setup-teams.md` |
| 8, "api", "sanctum", "token", "jwt", "passport" | `workflows/add-api-auth.md` |
| 9, other | Clarify intent, then select workflow or provide guidance from references |

**After reading the workflow, follow it exactly.**
</routing>

<verification_loop>
After every change:

```bash
# 1. Clear caches
php artisan config:clear
php artisan permission:cache-reset

# 2. Run tests
php artisan test --filter=Permission

# 3. Verify in browser/tinker
php artisan tinker
>>> $user = User::first();
>>> $user->can('your permission');
```

Report to user:
- "Cache cleared: ✓"
- "Tests: X pass, Y fail"
- "Permission check verified: [result]"
</verification_loop>

<reference_index>
All domain knowledge in `references/`:

**Architecture:** architecture.md (roles vs permissions, combining approaches, project structure)
**Setup:** installation-setup.md (Laravel 12, Spatie v6, middleware config)
**Core Usage:** permissions-roles.md (creating, assigning, checking, middleware, Blade)
**Authorization:** policies-gates.md (policies with Spatie, Gates, resource controllers)
**Caching:** caching.md (cache config, common issues, deployment)
**Testing:** testing.md (PHPUnit, Pest, testing middleware/policies)
**API:** api-auth.md (Sanctum integration, guards, token abilities)
**Multi-tenancy:** multi-tenancy.md (teams mode, team middleware, scoped permissions)
**Deployment:** deployment.md (seeding strategies, migrations, deploy scripts)
**Performance:** performance.md (N+1 prevention, eager loading, caching strategies)
**Anti-patterns:** anti-patterns.md (common mistakes and how to avoid them)
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| build-new-app.md | Create new Laravel 12 app with Spatie from scratch |
| add-permissions.md | Add Spatie permissions to existing Laravel app |
| debug-permissions.md | Troubleshoot 403 errors, cache issues, guard mismatches |
| write-tests.md | Write PHPUnit/Pest tests for authorization |
| optimize-performance.md | Fix N+1 queries, optimize caching, profile |
| ship-app.md | Deploy with proper seeding/migrations |
| setup-teams.md | Configure team-based multi-tenant permissions |
| add-api-auth.md | Integrate Sanctum with Spatie permissions |
</workflows_index>

<quick_reference>
**Common Commands:**

```bash
# Clear permission cache
php artisan permission:cache-reset

# Show cached permissions
php artisan permission:show

# Create permission in tinker
php artisan tinker
>>> Permission::create(['name' => 'edit articles']);

# Assign role
>>> $user->assignRole('editor');

# Check permission
>>> $user->can('edit articles');
```

**Common Checks:**

```php
$user->can('permission');           // Check permission
$user->hasRole('role');             // Check role (avoid in app logic)
$user->hasAnyPermission([...]);     // Any of these
$user->hasAllPermissions([...]);    // All of these
$user->getAllPermissions();         // Get all (direct + via roles)
```

**Middleware (routes):**

```php
Route::middleware('permission:edit articles')->get(...);
Route::middleware('role:admin')->get(...);
Route::middleware('role_or_permission:admin|edit articles')->get(...);
```
</quick_reference>
