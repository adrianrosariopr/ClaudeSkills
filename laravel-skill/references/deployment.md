<overview>
Deploying Laravel applications with Spatie permissions. Covers seeding strategies, migrations, and production considerations.
</overview>

<seeding_vs_migrations>
**Migrations (Recommended for Production)**

Use migrations for permission changes. They:
- Run once per environment
- Are tracked in `migrations` table
- Can be rolled back
- Won't accidentally duplicate permissions

```php
// database/migrations/xxxx_create_article_permissions.php
use Illuminate\Database\Migrations\Migration;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\PermissionRegistrar;

return new class extends Migration
{
    public function up(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        Permission::create(['name' => 'view articles']);
        Permission::create(['name' => 'create articles']);
        Permission::create(['name' => 'edit articles']);
        Permission::create(['name' => 'delete articles']);
    }

    public function down(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        Permission::whereIn('name', [
            'view articles', 'create articles', 'edit articles', 'delete articles'
        ])->delete();
    }
};
```

**Seeders (For Development/Testing)**

Use seeders for:
- Initial development setup
- Test data
- Demo environments

```php
// database/seeders/RolesAndPermissionsSeeder.php
class RolesAndPermissionsSeeder extends Seeder
{
    public function run(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        // Truncate for clean slate (development only!)
        // Don't do this in production
        DB::table('role_has_permissions')->truncate();
        DB::table('model_has_roles')->truncate();
        DB::table('model_has_permissions')->truncate();
        DB::table('roles')->truncate();
        DB::table('permissions')->truncate();

        // Create permissions
        $permissions = [
            'view articles', 'create articles', 'edit articles', 'delete articles',
            'view users', 'create users', 'edit users', 'delete users',
            'access admin',
        ];

        foreach ($permissions as $permission) {
            Permission::create(['name' => $permission]);
        }

        // Create roles
        $admin = Role::create(['name' => 'admin']);
        $admin->givePermissionTo(Permission::all());

        $editor = Role::create(['name' => 'editor']);
        $editor->givePermissionTo(['view articles', 'create articles', 'edit articles']);

        $viewer = Role::create(['name' => 'viewer']);
        $viewer->givePermissionTo(['view articles']);
    }
}
```
</seeding_vs_migrations>

<adding_permissions_safely>
**Check before creating (idempotent):**

```php
// Migration that can run multiple times safely
public function up(): void
{
    app()[PermissionRegistrar::class]->forgetCachedPermissions();

    $permissions = ['view reports', 'create reports', 'export reports'];

    foreach ($permissions as $name) {
        Permission::firstOrCreate(['name' => $name]);
    }
}
```

**Adding permission to existing role:**

```php
// database/migrations/xxxx_add_export_permission_to_editor.php
public function up(): void
{
    app()[PermissionRegistrar::class]->forgetCachedPermissions();

    $permission = Permission::firstOrCreate(['name' => 'export articles']);

    $editor = Role::findByName('editor');
    $editor->givePermissionTo($permission);
}

public function down(): void
{
    app()[PermissionRegistrar::class]->forgetCachedPermissions();

    $editor = Role::findByName('editor');
    $editor->revokePermissionTo('export articles');
}
```
</adding_permissions_safely>

<deployment_script>
**Complete deployment script:**

```bash
#!/bin/bash
# deploy.sh

set -e

echo "Deploying application..."

# Pull latest code
git pull origin main

# Install dependencies
composer install --no-dev --optimize-autoloader

# Run migrations
php artisan migrate --force

# Clear and rebuild caches
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Reset permission cache (CRITICAL)
php artisan permission:cache-reset

# Restart queue workers
php artisan queue:restart

echo "Deployment complete!"
```

**Laravel Forge deploy script:**

```bash
cd /home/forge/mysite.com

git pull origin $FORGE_SITE_BRANCH

$FORGE_COMPOSER install --no-dev --optimize-autoloader

$FORGE_PHP artisan migrate --force
$FORGE_PHP artisan permission:cache-reset
$FORGE_PHP artisan config:cache
$FORGE_PHP artisan route:cache
$FORGE_PHP artisan view:cache

$FORGE_PHP artisan queue:restart
```
</deployment_script>

<sync_permissions_command>
**Custom artisan command to sync permissions:**

```php
// app/Console/Commands/SyncPermissions.php
namespace App\Console\Commands;

use Illuminate\Console\Command;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;
use Spatie\Permission\PermissionRegistrar;

class SyncPermissions extends Command
{
    protected $signature = 'permissions:sync';
    protected $description = 'Sync permissions from config to database';

    public function handle(): int
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        // Define all permissions
        $permissions = [
            'view articles', 'create articles', 'edit articles', 'delete articles',
            'view users', 'create users', 'edit users', 'delete users',
            'access admin', 'manage settings',
        ];

        // Create missing permissions
        foreach ($permissions as $name) {
            Permission::firstOrCreate(['name' => $name]);
        }

        // Define role permissions
        $roles = [
            'admin' => Permission::all()->pluck('name')->toArray(),
            'editor' => ['view articles', 'create articles', 'edit articles', 'view users'],
            'viewer' => ['view articles', 'view users'],
        ];

        foreach ($roles as $roleName => $rolePermissions) {
            $role = Role::firstOrCreate(['name' => $roleName]);
            $role->syncPermissions($rolePermissions);
        }

        $this->info('Permissions synced successfully!');
        return Command::SUCCESS;
    }
}
```

Add to deployment: `php artisan permissions:sync`
</sync_permissions_command>

<zero_downtime_deployment>
**Considerations for zero-downtime:**

1. **Add permissions before code that uses them**
   - Deploy migration first
   - Then deploy code changes

2. **Remove permissions after code stops using them**
   - Deploy code changes first
   - Then deploy migration to remove

3. **Rename permissions carefully**
   ```php
   // Migration: create new, copy assignments, remove old
   public function up(): void
   {
       $old = Permission::findByName('edit posts');
       $new = Permission::create(['name' => 'edit articles']);

       // Copy all role assignments
       foreach ($old->roles as $role) {
           $role->givePermissionTo($new);
       }

       // Copy direct user assignments
       foreach ($old->users as $user) {
           $user->givePermissionTo($new);
       }
   }
   ```
</zero_downtime_deployment>

<environment_specific>
**Different permissions per environment:**

```php
// database/seeders/DatabaseSeeder.php
public function run(): void
{
    $this->call(RolesAndPermissionsSeeder::class);

    if (app()->environment('local', 'staging')) {
        $this->call(TestUserSeeder::class);
    }
}
```

**Environment checks in migrations:**

```php
public function up(): void
{
    // Only create test data in non-production
    if (app()->environment('production')) {
        return;
    }

    // Test/demo data...
}
```
</environment_specific>

<rollback_strategy>
**Plan for rollbacks:**

1. Always write `down()` methods
2. Test rollbacks locally before deploying
3. Keep permission changes separate from other migrations
4. Document any manual steps needed

```php
public function down(): void
{
    app()[PermissionRegistrar::class]->forgetCachedPermissions();

    // Remove permission from all roles first
    $permission = Permission::findByName('new permission');
    if ($permission) {
        $permission->roles()->detach();
        $permission->users()->detach();
        $permission->delete();
    }
}
```
</rollback_strategy>
