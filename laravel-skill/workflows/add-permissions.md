# Workflow: Add Permissions to Existing Laravel App

<required_reading>
**Read these reference files NOW:**
1. references/installation-setup.md
2. references/permissions-roles.md
3. references/architecture.md
</required_reading>

<process>
## Step 1: Install Spatie Package

```bash
composer require spatie/laravel-permission:^6.0
```

## Step 2: Publish and Run Migrations

```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```

This creates:
- `config/permission.php`
- Permission tables: `permissions`, `roles`, `model_has_permissions`, `model_has_roles`, `role_has_permissions`

## Step 3: Add HasRoles Trait to User Model

```php
// app/Models/User.php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;

    // Keep existing code...
}
```

## Step 4: Register Middleware (Laravel 12)

Edit `bootstrap/app.php`:

```php
use Spatie\Permission\Middleware\RoleMiddleware;
use Spatie\Permission\Middleware\PermissionMiddleware;
use Spatie\Permission\Middleware\RoleOrPermissionMiddleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'role' => RoleMiddleware::class,
            'permission' => PermissionMiddleware::class,
            'role_or_permission' => RoleOrPermissionMiddleware::class,
        ]);
    })
    // ... rest of configuration
    ->create();
```

## Step 5: Create Migration for Permissions

Use migrations (not seeders) for production-safe permission creation:

```bash
php artisan make:migration create_initial_permissions
```

```php
// database/migrations/xxxx_create_initial_permissions.php
use Illuminate\Database\Migrations\Migration;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;
use Spatie\Permission\PermissionRegistrar;

return new class extends Migration
{
    public function up(): void
    {
        // Reset cache
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        // Create permissions based on your app's features
        $permissions = [
            // Adjust these to match your application
            'view dashboard',
            'view articles',
            'create articles',
            'edit articles',
            'delete articles',
        ];

        foreach ($permissions as $permission) {
            Permission::firstOrCreate(['name' => $permission]);
        }

        // Create roles
        $admin = Role::firstOrCreate(['name' => 'admin']);
        $admin->syncPermissions(Permission::all());

        $user = Role::firstOrCreate(['name' => 'user']);
        $user->syncPermissions(['view dashboard', 'view articles']);
    }

    public function down(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        // Remove in reverse order
        Role::whereIn('name', ['admin', 'user'])->delete();
        Permission::whereIn('name', [
            'view dashboard', 'view articles', 'create articles',
            'edit articles', 'delete articles',
        ])->delete();
    }
};
```

Run the migration:

```bash
php artisan migrate
```

## Step 6: Update Existing Routes

Before:
```php
Route::middleware('auth')->group(function () {
    Route::resource('articles', ArticleController::class);
});
```

After:
```php
Route::middleware('auth')->group(function () {
    Route::get('/articles', [ArticleController::class, 'index'])
        ->middleware('permission:view articles');

    Route::get('/articles/create', [ArticleController::class, 'create'])
        ->middleware('permission:create articles');

    Route::post('/articles', [ArticleController::class, 'store'])
        ->middleware('permission:create articles');

    Route::get('/articles/{article}/edit', [ArticleController::class, 'edit'])
        ->middleware('permission:edit articles');

    Route::put('/articles/{article}', [ArticleController::class, 'update'])
        ->middleware('permission:edit articles');

    Route::delete('/articles/{article}', [ArticleController::class, 'destroy'])
        ->middleware('permission:delete articles');
});
```

Or use controller-based authorization:

```php
// app/Http/Controllers/ArticleController.php
public function __construct()
{
    $this->middleware('permission:view articles')->only(['index', 'show']);
    $this->middleware('permission:create articles')->only(['create', 'store']);
    $this->middleware('permission:edit articles')->only(['edit', 'update']);
    $this->middleware('permission:delete articles')->only(['destroy']);
}
```

## Step 7: Update Views

Add permission checks to navigation and actions:

```blade
{{-- Before --}}
<a href="{{ route('articles.create') }}">New Article</a>

{{-- After --}}
@can('create articles')
    <a href="{{ route('articles.create') }}">New Article</a>
@endcan
```

```blade
{{-- In article list --}}
@foreach ($articles as $article)
    <div>
        {{ $article->title }}

        @can('edit articles')
            <a href="{{ route('articles.edit', $article) }}">Edit</a>
        @endcan

        @can('delete articles')
            <form action="{{ route('articles.destroy', $article) }}" method="POST">
                @csrf
                @method('DELETE')
                <button type="submit">Delete</button>
            </form>
        @endcan
    </div>
@endforeach
```

## Step 8: Assign Roles to Existing Users

Create a migration or run in tinker:

```bash
php artisan tinker
```

```php
use App\Models\User;

// Make first user admin
$admin = User::first();
$admin->assignRole('admin');

// Or assign to specific user
$user = User::find(5);
$user->assignRole('user');

// Bulk assign
User::whereIn('id', [1, 2, 3])->each(fn ($u) => $u->assignRole('admin'));
```

## Step 9: Verify Integration

```bash
php artisan tinker
```

```php
$user = User::first();
$user->getRoleNames();        // Check roles
$user->getAllPermissions();   // Check permissions
$user->can('edit articles');  // Test permission check
```

Test in browser:
1. Log in as user with role
2. Verify protected routes work
3. Verify unauthorized routes return 403
4. Check navigation shows/hides correctly
</process>

<anti_patterns>
Avoid:
- Adding permissions without clearing cache
- Using seeders for production permission changes
- Forgetting to update views with @can directives
- Leaving routes unprotected after adding package
</anti_patterns>

<success_criteria>
Permissions successfully added when:
- [ ] Package installed and migrations run
- [ ] User model has HasRoles trait
- [ ] Middleware registered
- [ ] Permissions created via migration (not seeder)
- [ ] Existing routes protected
- [ ] Views updated with @can directives
- [ ] Existing users assigned appropriate roles
- [ ] Permission checks work in tinker and browser
</success_criteria>
