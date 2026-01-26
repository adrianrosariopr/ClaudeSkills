<overview>
Testing authorization logic with PHPUnit and Pest. Covers testing permissions, roles, policies, and middleware.
</overview>

<test_setup>
**Base TestCase with permission cache reset:**

```php
// tests/TestCase.php
use Spatie\Permission\PermissionRegistrar;

abstract class TestCase extends BaseTestCase
{
    protected function setUp(): void
    {
        parent::setUp();
        app()[PermissionRegistrar::class]->forgetCachedPermissions();
    }
}
```
</test_setup>

<testing_permissions_phpunit>
```php
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
}
```
</testing_permissions_phpunit>

<testing_permissions_pest>
```php
use App\Models\User;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

beforeEach(function () {
    app()[\Spatie\Permission\PermissionRegistrar::class]->forgetCachedPermissions();
});

it('assigns permission to user', function () {
    $user = User::factory()->create();
    Permission::create(['name' => 'edit articles']);

    $user->givePermissionTo('edit articles');

    expect($user->can('edit articles'))->toBeTrue();
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
public function test_route_requires_permission(): void
{
    Permission::create(['name' => 'access admin']);

    $user = User::factory()->create();
    $this->actingAs($user)
        ->get('/admin')
        ->assertStatus(403);

    $user->givePermissionTo('access admin');
    $this->actingAs($user)
        ->get('/admin')
        ->assertStatus(200);
}
```
</testing_middleware>

<testing_policies>
```php
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
}
```
</testing_policies>

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

// Usage
$admin = User::factory()->admin()->create();
$editor = User::factory()->editor()->create();
```
</factory_with_roles>

<seeder_for_tests>
```php
class TestPermissionsSeeder extends Seeder
{
    public function run(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        $permissions = [
            'view articles', 'create articles', 'edit articles', 'delete articles',
        ];

        foreach ($permissions as $permission) {
            Permission::create(['name' => $permission]);
        }

        $admin = Role::create(['name' => 'admin']);
        $admin->givePermissionTo(Permission::all());

        $editor = Role::create(['name' => 'editor']);
        $editor->givePermissionTo(['view articles', 'create articles', 'edit articles']);
    }
}

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
