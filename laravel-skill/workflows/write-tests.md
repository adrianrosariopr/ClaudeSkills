# Workflow: Write Tests for Authorization

<required_reading>
**Read these reference files NOW:**
1. references/testing.md
2. references/policies-gates.md
3. references/permissions-roles.md
</required_reading>

<process>
## Step 1: Set Up Test Base Class

Add permission cache reset to TestCase:

```php
// tests/TestCase.php
namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;
use Spatie\Permission\PermissionRegistrar;

abstract class TestCase extends BaseTestCase
{
    protected function setUp(): void
    {
        parent::setUp();

        // Reset permission cache before each test
        app()[PermissionRegistrar::class]->forgetCachedPermissions();
    }
}
```

## Step 2: Create Test Permissions Seeder

```php
// database/seeders/TestPermissionsSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;
use Spatie\Permission\PermissionRegistrar;

class TestPermissionsSeeder extends Seeder
{
    public function run(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        $permissions = [
            'view articles', 'create articles', 'edit articles', 'delete articles',
            'view users', 'create users', 'edit users', 'delete users',
            'access admin',
        ];

        foreach ($permissions as $permission) {
            Permission::firstOrCreate(['name' => $permission]);
        }

        $admin = Role::firstOrCreate(['name' => 'admin']);
        $admin->syncPermissions(Permission::all());

        $editor = Role::firstOrCreate(['name' => 'editor']);
        $editor->syncPermissions(['view articles', 'create articles', 'edit articles']);
    }
}
```

## Step 3: Create User Factory States

```php
// database/factories/UserFactory.php
public function admin(): static
{
    return $this->afterCreating(function (User $user) {
        $user->assignRole('admin');
    });
}

public function editor(): static
{
    return $this->afterCreating(function (User $user) {
        $user->assignRole('editor');
    });
}

public function withPermission(string $permission): static
{
    return $this->afterCreating(function (User $user) use ($permission) {
        $user->givePermissionTo($permission);
    });
}
```

## Step 4: Write Permission Tests (PHPUnit)

```php
// tests/Feature/PermissionTest.php
namespace Tests\Feature;

use App\Models\User;
use Database\Seeders\TestPermissionsSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;
use Tests\TestCase;

class PermissionTest extends TestCase
{
    use RefreshDatabase;

    protected function setUp(): void
    {
        parent::setUp();
        $this->seed(TestPermissionsSeeder::class);
    }

    public function test_user_can_be_assigned_role(): void
    {
        $user = User::factory()->create();

        $user->assignRole('editor');

        $this->assertTrue($user->hasRole('editor'));
    }

    public function test_user_with_role_has_role_permissions(): void
    {
        $user = User::factory()->create();
        $user->assignRole('editor');

        $this->assertTrue($user->can('view articles'));
        $this->assertTrue($user->can('edit articles'));
        $this->assertFalse($user->can('delete articles'));
    }

    public function test_user_can_have_direct_permission(): void
    {
        $user = User::factory()->create();

        $user->givePermissionTo('delete articles');

        $this->assertTrue($user->can('delete articles'));
    }

    public function test_revoking_permission_works(): void
    {
        $user = User::factory()->create();
        $user->givePermissionTo('edit articles');

        $user->revokePermissionTo('edit articles');

        $this->assertFalse($user->can('edit articles'));
    }
}
```

## Step 5: Write Middleware Tests

```php
// tests/Feature/MiddlewareTest.php
namespace Tests\Feature;

use App\Models\User;
use Database\Seeders\TestPermissionsSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class MiddlewareTest extends TestCase
{
    use RefreshDatabase;

    protected function setUp(): void
    {
        parent::setUp();
        $this->seed(TestPermissionsSeeder::class);
    }

    public function test_user_with_permission_can_access_protected_route(): void
    {
        $user = User::factory()->create();
        $user->givePermissionTo('view articles');

        $response = $this->actingAs($user)->get('/articles');

        $response->assertStatus(200);
    }

    public function test_user_without_permission_gets_403(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->get('/articles');

        $response->assertStatus(403);
    }

    public function test_guest_is_redirected_to_login(): void
    {
        $response = $this->get('/articles');

        $response->assertRedirect('/login');
    }

    public function test_admin_can_access_admin_routes(): void
    {
        $user = User::factory()->admin()->create();

        $response = $this->actingAs($user)->get('/admin');

        $response->assertStatus(200);
    }
}
```

## Step 6: Write Policy Tests

```php
// tests/Unit/ArticlePolicyTest.php
namespace Tests\Unit;

use App\Models\Article;
use App\Models\User;
use App\Policies\ArticlePolicy;
use Database\Seeders\TestPermissionsSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ArticlePolicyTest extends TestCase
{
    use RefreshDatabase;

    private ArticlePolicy $policy;

    protected function setUp(): void
    {
        parent::setUp();
        $this->seed(TestPermissionsSeeder::class);
        $this->policy = new ArticlePolicy();
    }

    public function test_user_with_permission_can_update_any_article(): void
    {
        $user = User::factory()->create();
        $user->givePermissionTo('edit articles');
        $article = Article::factory()->create();

        $this->assertTrue($this->policy->update($user, $article));
    }

    public function test_owner_can_update_own_article(): void
    {
        $user = User::factory()->create();
        $article = Article::factory()->create(['user_id' => $user->id]);

        $this->assertTrue($this->policy->update($user, $article));
    }

    public function test_non_owner_without_permission_cannot_update(): void
    {
        $user = User::factory()->create();
        $article = Article::factory()->create(); // Different owner

        $this->assertFalse($this->policy->update($user, $article));
    }

    public function test_user_with_delete_permission_can_delete(): void
    {
        $user = User::factory()->create();
        $user->givePermissionTo('delete articles');
        $article = Article::factory()->create();

        $this->assertTrue($this->policy->delete($user, $article));
    }
}
```

## Step 7: Write Pest Tests (Alternative)

```php
// tests/Feature/PermissionTest.php
use App\Models\User;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

beforeEach(function () {
    $this->seed(\Database\Seeders\TestPermissionsSeeder::class);
});

describe('Permission assignment', function () {
    it('assigns role to user', function () {
        $user = User::factory()->create();

        $user->assignRole('editor');

        expect($user->hasRole('editor'))->toBeTrue();
    });

    it('grants role permissions to user', function () {
        $user = User::factory()->create();
        $user->assignRole('editor');

        expect($user->can('view articles'))->toBeTrue()
            ->and($user->can('edit articles'))->toBeTrue()
            ->and($user->can('delete articles'))->toBeFalse();
    });

    it('grants direct permission to user', function () {
        $user = User::factory()->create();

        $user->givePermissionTo('delete articles');

        expect($user->can('delete articles'))->toBeTrue();
    });
});

describe('Route protection', function () {
    it('allows user with permission', function () {
        $user = User::factory()->create();
        $user->givePermissionTo('view articles');

        $this->actingAs($user)
            ->get('/articles')
            ->assertOk();
    });

    it('denies user without permission', function () {
        $user = User::factory()->create();

        $this->actingAs($user)
            ->get('/articles')
            ->assertForbidden();
    });
});
```

## Step 8: Run Tests

```bash
# Run all tests
php artisan test

# Run specific test file
php artisan test tests/Feature/PermissionTest.php

# Run with filter
php artisan test --filter=permission

# Run with coverage
php artisan test --coverage
```
</process>

<anti_patterns>
Avoid:
- Not seeding permissions in test setup
- Forgetting to reset permission cache
- Testing only happy paths (test denials too)
- Using real database instead of RefreshDatabase
- Not testing both role-based and direct permissions
</anti_patterns>

<success_criteria>
Tests are complete when:
- [ ] Permission cache reset in TestCase setUp
- [ ] Test seeder creates all needed permissions/roles
- [ ] Factory states for common roles
- [ ] Tests cover: assignment, revocation, checking
- [ ] Tests cover: middleware allows/denies
- [ ] Tests cover: policies with permissions
- [ ] Both positive and negative cases tested
- [ ] All tests pass
</success_criteria>
