<overview>
Spatie laravel-permission caches all permissions for 24 hours by default. Understanding cache behavior is critical for avoiding 403 errors.
</overview>

<default_behavior>
- All permissions cached for 24 hours
- Cache automatically flushed when roles/permissions updated via package methods
- Direct database queries bypass cache (must flush manually)
- Cache key: `spatie.permission.cache`
</default_behavior>

<cache_configuration>
```php
// config/permission.php
'cache' => [
    'expiration_time' => \DateInterval::createFromDateString('24 hours'),
    'key' => 'spatie.permission.cache',
    'store' => 'redis',  // Use Redis for production
],
```
</cache_configuration>

<manual_cache_operations>
**Clear permission cache:**

```bash
php artisan permission:cache-reset
```

```php
app()[\Spatie\Permission\PermissionRegistrar::class]->forgetCachedPermissions();
```

**When to clear manually:**
- After direct database queries that modify permissions
- After database seeding
- After migrations that modify permission tables
- When switching tenants (multi-tenancy)
</manual_cache_operations>

<seeder_cache_handling>
```php
class RolesAndPermissionsSeeder extends Seeder
{
    public function run(): void
    {
        // Reset cached permissions
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        Permission::create(['name' => 'edit articles']);
        $role = Role::create(['name' => 'editor']);
        $role->givePermissionTo(['edit articles']);
    }
}
```
</seeder_cache_handling>

<common_cache_issues>
**Issue 1: 403 after deployment**

Fix:
```bash
php artisan permission:cache-reset
php artisan config:clear
php artisan cache:clear
```

**Issue 2: 403 after seeding**

Fix: Add cache reset to seeder.

**Issue 3: Intermittent 403s with Memcached**

Use Redis instead:
```env
CACHE_STORE=redis
```
</common_cache_issues>

<testing_cache>
```php
// tests/TestCase.php
protected function setUp(): void
{
    parent::setUp();
    app()[PermissionRegistrar::class]->forgetCachedPermissions();
}
```
</testing_cache>

<deployment_cache_strategy>
```bash
# deployment.sh
php artisan migrate --force
php artisan permission:cache-reset
php artisan config:cache
php artisan route:cache
php artisan view:cache
```
</deployment_cache_strategy>
