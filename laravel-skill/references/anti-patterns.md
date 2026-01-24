<overview>
Common mistakes and anti-patterns when using Laravel with Spatie permissions. Learn what NOT to do.
</overview>

<checking_roles_instead_of_permissions>
**Anti-pattern:**

```php
// BAD - Couples code to role names
if ($user->hasRole('editor')) {
    // Allow editing
}

@role('admin')
    <button>Delete</button>
@endrole
```

**Why it's bad:**
- Role definitions change; code shouldn't
- Can't easily modify what editors can do
- Scattered role checks are hard to maintain

**Correct approach:**

```php
// GOOD - Check capability, not identity
if ($user->can('edit articles')) {
    // Allow editing
}

@can('delete articles')
    <button>Delete</button>
@endcan
```

Now you can change what "editor" means without changing code.
</checking_roles_instead_of_permissions>

<forgetting_cache_reset>
**Anti-pattern:**

```php
// BAD - Direct DB manipulation without cache reset
DB::table('permissions')->insert(['name' => 'new permission', 'guard_name' => 'web']);
```

**Why it's bad:**
- Cache still has old data
- Permission checks fail unexpectedly
- Intermittent 403 errors

**Correct approach:**

```php
// GOOD - Use package methods (auto-clears cache)
Permission::create(['name' => 'new permission']);

// Or manually clear cache after direct DB operations
DB::table('permissions')->insert([...]);
app()[\Spatie\Permission\PermissionRegistrar::class]->forgetCachedPermissions();
```
</forgetting_cache_reset>

<duplicate_permission_creation>
**Anti-pattern:**

```php
// BAD - Throws exception if run twice
public function run(): void
{
    Permission::create(['name' => 'edit articles']);
    Permission::create(['name' => 'edit articles']); // Exception!
}
```

**Correct approach:**

```php
// GOOD - Idempotent
Permission::firstOrCreate(['name' => 'edit articles']);

// Or check first
if (!Permission::where('name', 'edit articles')->exists()) {
    Permission::create(['name' => 'edit articles']);
}
```
</duplicate_permission_creation>

<seeding_in_production>
**Anti-pattern:**

```bash
# BAD - Running full seeder in production
php artisan db:seed --class=RolesAndPermissionsSeeder
```

**Why it's bad:**
- May truncate existing data
- Overwrites user role assignments
- Not idempotent

**Correct approach:**

Use migrations for production permission changes:

```php
// GOOD - Migration for new permission
public function up(): void
{
    Permission::firstOrCreate(['name' => 'new feature']);
}
```
</seeding_in_production>

<guard_mismatch>
**Anti-pattern:**

```php
// BAD - Permission created for web guard
Permission::create(['name' => 'api.read']); // defaults to 'web'

// API route using sanctum guard
Route::middleware(['auth:sanctum', 'permission:api.read'])->get('/data', ...);
// Error: Permission should use guard web instead of sanctum
```

**Correct approach:**

```php
// GOOD - Explicit guard
Permission::create(['name' => 'api.read', 'guard_name' => 'api']);

// Or force user model to use single guard
class User extends Authenticatable
{
    protected $guard_name = 'web';
}
```
</guard_mismatch>

<n_plus_one_queries>
**Anti-pattern:**

```php
// BAD - N+1 in loop
@foreach ($users as $user)
    {{ $user->getRoleNames()->implode(', ') }}
@endforeach
```

**Why it's bad:**
- One query per user
- Slow with many users
- Unnecessary database load

**Correct approach:**

```php
// GOOD - Eager load
$users = User::with('roles')->get();

@foreach ($users as $user)
    {{ $user->roles->pluck('name')->implode(', ') }}
@endforeach
```
</n_plus_one_queries>

<over_permissioning>
**Anti-pattern:**

```php
// BAD - Too few, too broad permissions
Permission::create(['name' => 'manage everything']);

// Or giving admin all permissions directly
$user->givePermissionTo(Permission::all());
```

**Why it's bad:**
- No granular control
- Can't audit who can do what
- Principle of least privilege violated

**Correct approach:**

```php
// GOOD - Granular permissions
$permissions = [
    'view articles', 'create articles', 'edit articles', 'delete articles',
    'publish articles', 'archive articles',
];

// Roles get specific subsets
$editor->givePermissionTo(['view articles', 'create articles', 'edit articles']);
$publisher->givePermissionTo(['view articles', 'publish articles']);
```
</over_permissioning>

<hardcoding_permission_names>
**Anti-pattern:**

```php
// BAD - Strings scattered everywhere
$user->can('edit-articles');  // typo: dash vs space
$user->can('edit articles');  // correct
$user->can('editArticles');   // different format
```

**Correct approach:**

```php
// GOOD - Constants or enum
class Permissions
{
    public const VIEW_ARTICLES = 'view articles';
    public const CREATE_ARTICLES = 'create articles';
    public const EDIT_ARTICLES = 'edit articles';
    public const DELETE_ARTICLES = 'delete articles';
}

// Usage
$user->can(Permissions::EDIT_ARTICLES);

// Or PHP 8.1+ enum
enum Permission: string
{
    case ViewArticles = 'view articles';
    case EditArticles = 'edit articles';
}

$user->can(Permission::EditArticles->value);
```
</hardcoding_permission_names>

<not_testing_permissions>
**Anti-pattern:**

```php
// BAD - No tests for authorization
public function test_article_can_be_updated(): void
{
    $article = Article::factory()->create();
    $response = $this->put("/articles/{$article->id}", ['title' => 'New']);
    $response->assertOk();
}
```

**Why it's bad:**
- Doesn't verify authorization works
- Could pass even if permissions broken
- Security gaps undetected

**Correct approach:**

```php
// GOOD - Test both allowed and denied
public function test_user_with_permission_can_update_article(): void
{
    $user = User::factory()->create();
    $user->givePermissionTo('edit articles');
    $article = Article::factory()->create();

    $this->actingAs($user)
        ->put("/articles/{$article->id}", ['title' => 'New'])
        ->assertOk();
}

public function test_user_without_permission_cannot_update_article(): void
{
    $user = User::factory()->create();
    $article = Article::factory()->create();

    $this->actingAs($user)
        ->put("/articles/{$article->id}", ['title' => 'New'])
        ->assertForbidden();
}
```
</not_testing_permissions>

<ignoring_super_admin>
**Anti-pattern:**

```php
// BAD - Super admin must have every permission explicitly
$superAdmin->givePermissionTo(Permission::all());

// Adding new permission requires updating super admin
Permission::create(['name' => 'new feature']);
$superAdmin->givePermissionTo('new feature'); // Easy to forget!
```

**Correct approach:**

```php
// GOOD - Gate::before bypass
// In AuthServiceProvider
Gate::before(function (User $user, string $ability) {
    if ($user->hasRole('super-admin')) {
        return true;
    }
});

// Super admin automatically passes all checks
// No need to assign every permission
```
</ignoring_super_admin>

<wrong_middleware_order>
**Anti-pattern (Teams):**

```php
// BAD - SubstituteBindings before team context
$middleware->priority([
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    \App\Http\Middleware\SetTeamContext::class,  // Too late!
]);
```

**Why it's bad:**
- Route model binding happens before team is set
- 404 errors instead of 403
- Permission checks fail

**Correct approach:**

```php
// GOOD - Team context set first
$middleware->priority([
    \App\Http\Middleware\SetTeamContext::class,
    \Spatie\Permission\Middleware\PermissionMiddleware::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
]);
```
</wrong_middleware_order>

<summary>
**Quick checklist to avoid anti-patterns:**

1. Check permissions, not roles
2. Always reset cache after direct DB changes
3. Use `firstOrCreate` for idempotent operations
4. Use migrations (not seeders) in production
5. Match guard names correctly
6. Eager load relationships
7. Create granular permissions
8. Use constants/enums for permission names
9. Test both allowed and denied scenarios
10. Use `Gate::before` for super admin
11. Set correct middleware priority for teams
</summary>
