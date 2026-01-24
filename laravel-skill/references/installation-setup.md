<overview>
Complete setup guide for Laravel 12 with Spatie laravel-permission v6. Covers fresh installation and adding to existing projects.
</overview>

<requirements>
- PHP 8.2 - 8.4
- Laravel 12.x
- Spatie laravel-permission 6.x (supports Laravel 8.12 - 12)
</requirements>

<fresh_installation>
**Step 1: Create Laravel 12 project**

```bash
composer create-project laravel/laravel my-app
cd my-app
```

**Step 2: Install Spatie package**

```bash
composer require spatie/laravel-permission:^6.0
```

**Step 3: Publish config and migrations**

```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

This creates:
- `config/permission.php` - Package configuration
- `database/migrations/xxxx_create_permission_tables.php` - Database tables

**Step 4: Run migrations**

```bash
php artisan migrate
```

**Step 5: Add HasRoles trait to User model**

```php
// app/Models/User.php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;

    // ...
}
```
</fresh_installation>

<existing_project>
For existing Laravel 12 projects:

```bash
# Install package
composer require spatie/laravel-permission:^6.0

# Publish files
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"

# Run migrations
php artisan migrate

# Add trait to User model (see above)
```
</existing_project>

<middleware_setup>
**Laravel 12 middleware registration** (in `bootstrap/app.php`):

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
</middleware_setup>

<config_options>
Key options in `config/permission.php`:

```php
return [
    'models' => [
        'permission' => Spatie\Permission\Models\Permission::class,
        'role' => Spatie\Permission\Models\Role::class,
    ],

    'table_names' => [
        'roles' => 'roles',
        'permissions' => 'permissions',
        'model_has_permissions' => 'model_has_permissions',
        'model_has_roles' => 'model_has_roles',
        'role_has_permissions' => 'role_has_permissions',
    ],

    // Enable for multi-tenant apps
    'teams' => false,

    // Cache settings
    'cache' => [
        'expiration_time' => \DateInterval::createFromDateString('24 hours'),
        'key' => 'spatie.permission.cache',
        'store' => 'default',
    ],
];
```
</config_options>

<database_tables>
After migration, these tables are created:

| Table | Purpose |
|-------|---------|
| `permissions` | Stores permission records |
| `roles` | Stores role records |
| `model_has_permissions` | Direct permission assignments |
| `model_has_roles` | Role assignments to users |
| `role_has_permissions` | Permissions assigned to roles |
</database_tables>

<starter_kit_integration>
**With Laravel Breeze:**

```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
npm install && npm run build
```

**With Laravel Jetstream:**

```bash
composer require laravel/jetstream
php artisan jetstream:install livewire
npm install && npm run build
```

Add HasRoles trait after installing either starter kit.
</starter_kit_integration>

<verification>
Verify installation:

```bash
# Check package is installed
composer show spatie/laravel-permission

# Check migrations ran
php artisan migrate:status

# Test in tinker
php artisan tinker
>>> use Spatie\Permission\Models\Role;
>>> Role::create(['name' => 'test']);
>>> Role::all();
```
</verification>

<common_issues>
**Issue: Cache table not found**
If using `CACHE_STORE=database`, install cache tables first:
```bash
php artisan cache:table
php artisan migrate
```

**Issue: Class not found**
Clear autoload cache:
```bash
composer dump-autoload
php artisan config:clear
```
</common_issues>
