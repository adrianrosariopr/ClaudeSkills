<overview>
Deploying Laravel applications with Spatie permissions. Covers seeding strategies, migrations, and production considerations.
</overview>

<seeding_vs_migrations>
**Migrations (Recommended for Production)**

Use migrations for permission changes:
- Run once per environment
- Tracked in `migrations` table
- Can be rolled back
- Won't accidentally duplicate

```php
// database/migrations/xxxx_create_article_permissions.php
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

Use seeders for initial setup and test data.
</seeding_vs_migrations>

<adding_permissions_safely>
**Idempotent migrations:**

```php
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
```bash
#!/bin/bash
set -e

echo "Deploying application..."

git pull origin main
composer install --no-dev --optimize-autoloader

php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan permission:cache-reset
php artisan queue:restart

echo "Deployment complete!"
```

**Laravel Forge:**

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
```php
// app/Console/Commands/SyncPermissions.php
class SyncPermissions extends Command
{
    protected $signature = 'permissions:sync';

    public function handle(): int
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        $permissions = [
            'view articles', 'create articles', 'edit articles', 'delete articles',
            'view users', 'create users', 'edit users', 'delete users',
        ];

        foreach ($permissions as $name) {
            Permission::firstOrCreate(['name' => $name]);
        }

        $roles = [
            'admin' => Permission::all()->pluck('name')->toArray(),
            'editor' => ['view articles', 'create articles', 'edit articles'],
        ];

        foreach ($roles as $roleName => $rolePermissions) {
            $role = Role::firstOrCreate(['name' => $roleName]);
            $role->syncPermissions($rolePermissions);
        }

        $this->info('Permissions synced!');
        return Command::SUCCESS;
    }
}
```
</sync_permissions_command>

<zero_downtime>
**Add permissions before code that uses them:**
1. Deploy migration first
2. Then deploy code changes

**Remove permissions after code stops using them:**
1. Deploy code changes first
2. Then deploy migration to remove

**Rename permissions:**
```php
public function up(): void
{
    $old = Permission::findByName('edit posts');
    $new = Permission::create(['name' => 'edit articles']);

    foreach ($old->roles as $role) {
        $role->givePermissionTo($new);
    }

    foreach ($old->users as $user) {
        $user->givePermissionTo($new);
    }
}
```
</zero_downtime>

<rollback_strategy>
Always write `down()` methods:

```php
public function down(): void
{
    app()[PermissionRegistrar::class]->forgetCachedPermissions();

    $permission = Permission::findByName('new permission');
    if ($permission) {
        $permission->roles()->detach();
        $permission->users()->detach();
        $permission->delete();
    }
}
```
</rollback_strategy>
