# Workflow: Build New Laravel App with Permissions

<required_reading>
**Read these reference files NOW before building:**
1. references/installation-setup.md
2. references/architecture.md
3. references/permissions-roles.md
</required_reading>

<process>
## Step 1: Create Laravel 12 Project

```bash
composer create-project laravel/laravel my-app
cd my-app
```

Or with a starter kit:

```bash
# With Breeze (Blade)
composer create-project laravel/laravel my-app
cd my-app
composer require laravel/breeze --dev
php artisan breeze:install blade
npm install && npm run build

# With Breeze (React/Vue)
php artisan breeze:install react  # or vue
npm install && npm run build
```

## Step 2: Install Spatie Permission Package

```bash
composer require spatie/laravel-permission:^6.0
```

Publish config and migrations:

```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

Run migrations:

```bash
php artisan migrate
```

## Step 3: Configure User Model

Add `HasRoles` trait to User model:

```php
// app/Models/User.php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;

    // ... rest of model
}
```

## Step 4: Register Middleware (Laravel 12)

Edit `bootstrap/app.php`:

```php
use Spatie\Permission\Middleware\RoleMiddleware;
use Spatie\Permission\Middleware\PermissionMiddleware;
use Spatie\Permission\Middleware\RoleOrPermissionMiddleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'role' => RoleMiddleware::class,
            'permission' => PermissionMiddleware::class,
            'role_or_permission' => RoleOrPermissionMiddleware::class,
        ]);
    })
    ->create();
```

## Step 5: Create Permissions Seeder

```bash
php artisan make:seeder RolesAndPermissionsSeeder
```

Edit the seeder:

```php
// database/seeders/RolesAndPermissionsSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;
use Spatie\Permission\PermissionRegistrar;

class RolesAndPermissionsSeeder extends Seeder
{
    public function run(): void
    {
        // Reset cached roles and permissions
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        // Create permissions
        $permissions = [
            'view articles',
            'create articles',
            'edit articles',
            'delete articles',
            'publish articles',
            'view users',
            'create users',
            'edit users',
            'delete users',
            'access admin',
        ];

        foreach ($permissions as $permission) {
            Permission::create(['name' => $permission]);
        }

        // Create roles and assign permissions
        $admin = Role::create(['name' => 'admin']);
        $admin->givePermissionTo(Permission::all());

        $editor = Role::create(['name' => 'editor']);
        $editor->givePermissionTo([
            'view articles', 'create articles', 'edit articles', 'publish articles',
        ]);

        $author = Role::create(['name' => 'author']);
        $author->givePermissionTo([
            'view articles', 'create articles', 'edit articles',
        ]);

        $viewer = Role::create(['name' => 'viewer']);
        $viewer->givePermissionTo(['view articles']);
    }
}
```

Run the seeder:

```bash
php artisan db:seed --class=RolesAndPermissionsSeeder
```

## Step 6: Set Up Super Admin Bypass (Optional)

In `app/Providers/AppServiceProvider.php`:

```php
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    Gate::before(function ($user, $ability) {
        return $user->hasRole('super-admin') ? true : null;
    });
}
```

## Step 7: Create Test User with Role

```bash
php artisan tinker
```

```php
$user = \App\Models\User::factory()->create([
    'name' => 'Admin User',
    'email' => 'admin@example.com',
]);
$user->assignRole('admin');
```

## Step 8: Protect Routes

```php
// routes/web.php
use Illuminate\Support\Facades\Route;

Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);

    // Protected by permission
    Route::middleware('permission:access admin')->group(function () {
        Route::get('/admin', [AdminController::class, 'index']);
    });

    // Protected by role (use sparingly)
    Route::middleware('role:admin')->group(function () {
        Route::resource('users', UserController::class);
    });
});
```

## Step 9: Add Authorization to Views

```blade
{{-- resources/views/layouts/navigation.blade.php --}}
<nav>
    <a href="{{ route('dashboard') }}">Dashboard</a>

    @can('view articles')
        <a href="{{ route('articles.index') }}">Articles</a>
    @endcan

    @can('access admin')
        <a href="{{ route('admin.index') }}">Admin</a>
    @endcan
</nav>
```

## Step 10: Verify Installation

```bash
php artisan tinker
```

```php
use App\Models\User;
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;

// Check permissions exist
Permission::all()->pluck('name');

// Check roles exist
Role::all()->pluck('name');

// Test user permission
$user = User::first();
$user->can('view articles'); // Should return true/false
$user->getRoleNames(); // Should show assigned roles
```
</process>

<anti_patterns>
Avoid:
- Checking roles instead of permissions in application code
- Forgetting to run the seeder
- Not registering middleware aliases
- Skipping the cache reset in seeder
</anti_patterns>

<success_criteria>
A well-built Laravel app with permissions:
- [ ] Spatie package installed and configured
- [ ] User model has HasRoles trait
- [ ] Middleware aliases registered in bootstrap/app.php
- [ ] Permissions and roles seeded
- [ ] Routes protected with permission middleware
- [ ] Views use @can directives
- [ ] Test user can be assigned role and checked
- [ ] `php artisan tinker` permission checks work
</success_criteria>
