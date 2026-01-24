# Workflow: Deploy Laravel App with Permissions

<required_reading>
**Read these reference files NOW:**
1. references/deployment.md
2. references/caching.md
3. references/anti-patterns.md
</required_reading>

<process>
## Step 1: Convert Seeder to Migration

If permissions were created via seeder, convert to migration for production:

```bash
php artisan make:migration setup_initial_permissions
```

```php
// database/migrations/xxxx_setup_initial_permissions.php
use Illuminate\Database\Migrations\Migration;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;
use Spatie\Permission\PermissionRegistrar;

return new class extends Migration
{
    public function up(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        // Create permissions (idempotent)
        $permissions = [
            'view articles', 'create articles', 'edit articles', 'delete articles',
            'view users', 'create users', 'edit users', 'delete users',
            'access admin', 'manage settings',
        ];

        foreach ($permissions as $name) {
            Permission::firstOrCreate(['name' => $name]);
        }

        // Create roles (idempotent)
        $admin = Role::firstOrCreate(['name' => 'admin']);
        $admin->syncPermissions(Permission::all());

        $editor = Role::firstOrCreate(['name' => 'editor']);
        $editor->syncPermissions([
            'view articles', 'create articles', 'edit articles',
            'view users',
        ]);

        $viewer = Role::firstOrCreate(['name' => 'viewer']);
        $viewer->syncPermissions(['view articles', 'view users']);
    }

    public function down(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        Role::whereIn('name', ['admin', 'editor', 'viewer'])->delete();

        Permission::whereIn('name', [
            'view articles', 'create articles', 'edit articles', 'delete articles',
            'view users', 'create users', 'edit users', 'delete users',
            'access admin', 'manage settings',
        ])->delete();
    }
};
```

## Step 2: Create Deployment Script

```bash
#!/bin/bash
# deploy.sh

set -e  # Exit on error

echo "Starting deployment..."

# 1. Enable maintenance mode
php artisan down --refresh=15

# 2. Pull latest code
git pull origin main

# 3. Install dependencies
composer install --no-dev --optimize-autoloader

# 4. Run migrations
php artisan migrate --force

# 5. Clear and cache config
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 6. Reset permission cache (CRITICAL)
php artisan permission:cache-reset

# 7. Restart queue workers
php artisan queue:restart

# 8. Disable maintenance mode
php artisan up

echo "Deployment complete!"
```

## Step 3: Configure Production Cache

```env
# .env.production
CACHE_STORE=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=null
```

```php
// config/permission.php
'cache' => [
    'expiration_time' => \DateInterval::createFromDateString('24 hours'),
    'key' => 'spatie.permission.cache',
    'store' => 'redis',
],
```

## Step 4: Create Permission Sync Command

For ongoing permission management:

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
    protected $description = 'Sync permissions from code to database';

    private array $permissions = [
        'view articles', 'create articles', 'edit articles', 'delete articles',
        'view users', 'create users', 'edit users', 'delete users',
        'access admin', 'manage settings',
    ];

    private array $roles = [
        'admin' => ['*'],  // All permissions
        'editor' => ['view articles', 'create articles', 'edit articles', 'view users'],
        'viewer' => ['view articles', 'view users'],
    ];

    public function handle(): int
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        $this->info('Syncing permissions...');

        // Create/update permissions
        foreach ($this->permissions as $name) {
            Permission::firstOrCreate(['name' => $name]);
        }

        // Create/update roles
        foreach ($this->roles as $roleName => $rolePermissions) {
            $role = Role::firstOrCreate(['name' => $roleName]);

            if ($rolePermissions === ['*']) {
                $role->syncPermissions(Permission::all());
            } else {
                $role->syncPermissions($rolePermissions);
            }

            $this->line("  - {$roleName}: " . $role->permissions->count() . " permissions");
        }

        $this->info('Permissions synced!');

        return Command::SUCCESS;
    }
}
```

Add to deployment: `php artisan permissions:sync`

## Step 5: Set Up First Admin User

Create command for initial admin:

```php
// app/Console/Commands/CreateAdminUser.php
namespace App\Console\Commands;

use App\Models\User;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Hash;

class CreateAdminUser extends Command
{
    protected $signature = 'user:create-admin
                            {email : Admin email}
                            {--password= : Admin password}';

    protected $description = 'Create an admin user';

    public function handle(): int
    {
        $email = $this->argument('email');
        $password = $this->option('password') ?? $this->secret('Password');

        if (User::where('email', $email)->exists()) {
            $this->error("User {$email} already exists.");
            return Command::FAILURE;
        }

        $user = User::create([
            'name' => 'Admin',
            'email' => $email,
            'password' => Hash::make($password),
        ]);

        $user->assignRole('admin');

        $this->info("Admin user {$email} created successfully.");

        return Command::SUCCESS;
    }
}
```

Run on production:
```bash
php artisan user:create-admin admin@example.com --password=secure_password
```

## Step 6: Verify Deployment

```bash
# Check migrations ran
php artisan migrate:status

# Verify permissions exist
php artisan tinker
>>> Permission::count();
>>> Role::count();

# Test permission check
>>> $admin = User::where('email', 'admin@example.com')->first();
>>> $admin->can('access admin');

# Check cache is working
>>> app()[\Spatie\Permission\PermissionRegistrar::class]->getCacheStore();
```

## Step 7: Monitor Post-Deployment

Check logs for permission errors:

```bash
tail -f storage/logs/laravel.log | grep -i "permission\|403\|unauthorized"
```

Set up error monitoring (Sentry, Bugsnag, etc.) to catch 403 errors.

## Step 8: Document for Team

Create deployment checklist:

```markdown
## Deployment Checklist

Pre-deployment:
- [ ] All permission migrations committed
- [ ] Tests passing locally
- [ ] Reviewed permission changes

Deployment:
- [ ] Enable maintenance mode
- [ ] Pull latest code
- [ ] Run migrations
- [ ] Reset permission cache
- [ ] Restart queue workers
- [ ] Disable maintenance mode

Post-deployment:
- [ ] Verify admin can login
- [ ] Test permission-protected routes
- [ ] Check error logs
- [ ] Monitor for 403 errors
```
</process>

<rollback>
If deployment fails:

```bash
# Rollback migration
php artisan migrate:rollback --step=1

# Clear caches
php artisan permission:cache-reset
php artisan config:clear
php artisan cache:clear

# Restore previous code
git checkout HEAD~1
composer install
```
</rollback>

<success_criteria>
Deployment successful when:
- [ ] Migrations run without errors
- [ ] Permission cache reset
- [ ] Admin user can login
- [ ] Permission-protected routes work
- [ ] No 403 errors for authorized users
- [ ] Error monitoring configured
- [ ] Team has deployment documentation
</success_criteria>
