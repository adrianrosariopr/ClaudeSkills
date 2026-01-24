<overview>
Testing authorization logic with PHPUnit and Pest. Covers testing permissions, roles, policies, and middleware.
</overview>

<test_setup>
**Base TestCase with permission cache reset:**

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
</test_setup>

<testing_permissions_phpunit>
```php
// tests/Feature/PermissionTest.php
namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;
use Tests\TestCase;

class PermissionTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_be_assigned_permission(): void
    {
        $user = User::factory()->create();
        $permission = Permission::create(['name' => 'edit articles']);

        $user->givePermissionTo($permission);

        $this->assertTrue($user->hasPermissionTo('edit articles'));
        $this->assertTrue($user->can('edit articles'));
    }

    public function test_user_with_role_has_role_permissions(): void
    {
        $user = User::factory()->create();
        $role = Role::create(['name' => 'editor']);
        $permission = Permission::create(['name' => 'edit articles']);

        $role->givePermissionTo($permission);
        $user->assignRole($role);

        $this->assertTrue($user->hasRole('editor'));
        $this->assertTrue($user->can('edit articles'));
    }

    public function test_user_without_permission_is_denied(): void
    {
        $user = User::factory()->create();
        Permission::create(['name' => 'delete articles']);

        $this->assertFalse($user->can('delete articles'));
    }
}
```
</testing_permissions_phpunit>

<testing_permissions_pest>
```php
// tests/Feature/PermissionTest.php
use App\Models\User;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

beforeEach(function () {
    app()[\Spatie\Permission\PermissionRegistrar::class]->forgetCachedPermissions();
});

it('assigns permission to user', function () {
    $user = User::factory()->create();
    $permission = Permission::create(['name' => 'edit articles']);

    $user->givePermissionTo($permission);

    expect($user->can('edit articles'))->toBeTrue();
});

it('assigns role with permissions to user', function () {
    $user = User::factory()->create();
    $role = Role::create(['name' => 'editor']);
    Permission::create(['name' => 'edit articles']);

    $role->givePermissionTo('edit articles');
    $user->assignRole($role);

    expect($user->hasRole('editor'))->toBeTrue()
        ->and($user->can('edit articles'))->toBeTrue();
});

it('denies user without permission', function () {
    $user = User::factory()->create();
    Permission::create(['name' => 'delete articles']);

    expect($user->can('delete articles'))->toBeFalse();
});
```
</testing_permissions_pest>

<testing_middleware>
```php
// PHPUnit
public function test_route_requires_permission(): void
{
    Permission::create(['name' => 'access admin']);

    // User without permission
    $user = User::factory()->create();
    $this->actingAs($user)
        ->get('/admin')
        ->assertStatus(403);

    // User with permission
    $user->givePermissionTo('access admin');
    $this->actingAs($user)
        ->get('/admin')
        ->assertStatus(200);
}

// Pest
it('requires permission to access admin', function () {
    Permission::create(['name' => 'access admin']);
    $user = User::factory()->create();

    $this->actingAs($user)
        ->get('/admin')
        ->assertStatus(403);

    $user->givePermissionTo('access admin');

    $this->actingAs($user)
        ->get('/admin')
        ->assertStatus(200);
});
```
</testing_middleware>

<testing_policies>
```php
// tests/Unit/ArticlePolicyTest.php
namespace Tests\Unit;

use App\Models\Article;
use App\Models\User;
use App\Policies\ArticlePolicy;
use Spatie\Permission\Models\Permission;
use Tests\TestCase;

class ArticlePolicyTest extends TestCase
{
    private ArticlePolicy $policy;

    protected function setUp(): void
    {
        parent::setUp();
        $this->policy = new ArticlePolicy();
    }

    public function test_user_with_permission_can_update(): void
    {
        Permission::create(['name' => 'edit articles']);
        $user = User::factory()->create();
        $article = Article::factory()->create();

        $user->givePermissionTo('edit articles');

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
}
```
</testing_policies>

<testing_api_authorization>
```php
// With Sanctum
public function test_api_requires_permission(): void
{
    Permission::create(['name' => 'api.read', 'guard_name' => 'sanctum']);

    $user = User::factory()->create();
    $token = $user->createToken('test-token')->plainTextToken;

    // Without permission
    $this->withToken($token)
        ->getJson('/api/articles')
        ->assertStatus(403);

    // With permission
    $user->givePermissionTo('api.read');
    $this->withToken($token)
        ->getJson('/api/articles')
        ->assertStatus(200);
}
```
</testing_api_authorization>

<factory_with_roles>
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

// Usage in tests
$admin = User::factory()->admin()->create();
$editor = User::factory()->editor()->create();
```
</factory_with_roles>

<seeder_for_tests>
```php
// database/seeders/TestPermissionsSeeder.php
class TestPermissionsSeeder extends Seeder
{
    public function run(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        // Create common permissions
        $permissions = [
            'view articles', 'create articles', 'edit articles', 'delete articles',
            'view users', 'create users', 'edit users', 'delete users',
        ];

        foreach ($permissions as $permission) {
            Permission::create(['name' => $permission]);
        }

        // Create roles
        $admin = Role::create(['name' => 'admin']);
        $admin->givePermissionTo(Permission::all());

        $editor = Role::create(['name' => 'editor']);
        $editor->givePermissionTo(['view articles', 'create articles', 'edit articles']);
    }
}
```

```php
// In tests
protected function setUp(): void
{
    parent::setUp();
    $this->seed(TestPermissionsSeeder::class);
}
```
</seeder_for_tests>

<database_assertions>
```php
// Assert permission exists
$this->assertDatabaseHas('permissions', ['name' => 'edit articles']);

// Assert role has permission
$role = Role::findByName('editor');
$permission = Permission::findByName('edit articles');
$this->assertDatabaseHas('role_has_permissions', [
    'role_id' => $role->id,
    'permission_id' => $permission->id,
]);

// Assert user has role
$this->assertDatabaseHas('model_has_roles', [
    'model_id' => $user->id,
    'model_type' => User::class,
    'role_id' => $role->id,
]);
```
</database_assertions>
