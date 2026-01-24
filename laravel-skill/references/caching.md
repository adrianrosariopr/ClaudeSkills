<overview>
Spatie laravel-permission caches all permissions and roles for 24 hours by default. Understanding cache behavior is critical for avoiding 403 errors and performance issues.
</overview>

<default_behavior>
- All permissions cached for 24 hours
- Cache automatically flushed when roles/permissions are updated via package methods
- Direct database queries bypass cache (must flush manually)
- Cache key: `spatie.permission.cache`
</default_behavior>

<cache_configuration>
```php
// config/permission.php
'cache' => [
    // Cache duration
    'expiration_time' => \DateInterval::createFromDateString('24 hours'),

    // Cache key prefix
    'key' => 'spatie.permission.cache',

    // Cache store (uses default if not specified)
    'store' => 'default',
],
```

**Change cache store:**

```php
'cache' => [
    'store' => 'redis',  // Use Redis for permissions cache
],
```
</cache_configuration>

<manual_cache_operations>
**Clear permission cache:**

```bash
# Artisan command
php artisan permission:cache-reset
```

```php
// Programmatically
app()[\Spatie\Permission\PermissionRegistrar::class]->forgetCachedPermissions();
```

**When to clear manually:**
- After direct database queries that modify permissions/roles
- After database seeding
- After migrations that modify permission tables
- When switching tenants (multi-tenancy)
</manual_cache_operations>

<seeder_cache_handling>
**Always reset cache in seeders:**

```php
// database/seeders/RolesAndPermissionsSeeder.php
use Spatie\Permission\PermissionRegistrar;

class RolesAndPermissionsSeeder extends Seeder
{
    public function run(): void
    {
        // Reset cached permissions
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        // Create permissions
        Permission::create(['name' => 'edit articles']);
        Permission::create(['name' => 'delete articles']);

        // Create roles and assign permissions
        $role = Role::create(['name' => 'editor']);
        $role->givePermissionTo(['edit articles']);
    }
}
```
</seeder_cache_handling>

<common_cache_issues>
**Issue 1: 403 after deployment**

Symptoms: Works locally, 403 in production.

Fix:
```bash
php artisan permission:cache-reset
php artisan config:clear
php artisan cache:clear
```

**Issue 2: 403 after seeding**

Symptoms: Permissions work initially, then fail.

Fix: Add cache reset to seeder (see above).

**Issue 3: Cache not working with database driver**

If `CACHE_STORE=database`, install cache tables first:
```bash
php artisan cache:table
php artisan migrate
```

**Issue 4: Intermittent 403s with Memcached**

Memcached has known issues with this package. Use Redis instead:
```env
CACHE_STORE=redis
```

**Issue 5: Cache not clearing**

Using array cache in local bypasses persistence:
```php
// config/cache.php - for development
'default' => env('CACHE_STORE', 'file'),
```
</common_cache_issues>

<testing_cache>
**Reset cache in test setup:**

```php
// tests/TestCase.php
use Spatie\Permission\PermissionRegistrar;

protected function setUp(): void
{
    parent::setUp();

    // Reset permission cache before each test
    app()[PermissionRegistrar::class]->forgetCachedPermissions();
}
```

**Or in specific tests:**

```php
public function test_user_can_edit_article(): void
{
    app()[PermissionRegistrar::class]->forgetCachedPermissions();

    $permission = Permission::create(['name' => 'edit articles']);
    $user = User::factory()->create();
    $user->givePermissionTo($permission);

    $this->assertTrue($user->can('edit articles'));
}
```
</testing_cache>

<deployment_cache_strategy>
**Add to deployment script:**

```bash
# deployment.sh
php artisan migrate --force
php artisan permission:cache-reset
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

**Laravel Forge/Envoyer deploy hook:**

```bash
cd /home/forge/mysite.com
php artisan permission:cache-reset
```
</deployment_cache_strategy>

<performance_considerations>
**Cache improves performance:**
- Without cache: queries on every permission check
- With cache: single cache lookup

**Don't disable caching:**
The package always uses caching. Configure the cache store appropriately rather than trying to bypass it.

**Redis recommended for production:**
```env
CACHE_STORE=redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```
</performance_considerations>
