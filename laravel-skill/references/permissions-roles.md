<overview>
Core operations for creating, assigning, and checking roles and permissions with Spatie laravel-permission.
</overview>

<creating_permissions>
```php
use Spatie\Permission\Models\Permission;

// Create single permission
Permission::create(['name' => 'edit articles']);

// Create multiple permissions
$permissions = ['view articles', 'create articles', 'edit articles', 'delete articles'];
foreach ($permissions as $permission) {
    Permission::create(['name' => $permission]);
}

// With guard (for APIs)
Permission::create(['name' => 'edit articles', 'guard_name' => 'api']);
```
</creating_permissions>

<creating_roles>
```php
use Spatie\Permission\Models\Role;

// Create role
$role = Role::create(['name' => 'editor']);

// Give permissions to role
$role->givePermissionTo('edit articles');
$role->givePermissionTo(['view articles', 'create articles']);

// Sync permissions (replaces all)
$role->syncPermissions(['view articles', 'edit articles']);

// Revoke permission
$role->revokePermissionTo('delete articles');
```
</creating_roles>

<assigning_to_users>
```php
// Assign role
$user->assignRole('editor');
$user->assignRole(['editor', 'writer']);

// Remove role
$user->removeRole('editor');

// Sync roles (replaces all)
$user->syncRoles(['editor']);

// Give direct permission (bypasses roles)
$user->givePermissionTo('delete articles');

// Revoke direct permission
$user->revokePermissionTo('delete articles');

// Sync direct permissions
$user->syncPermissions(['edit articles', 'delete articles']);
```
</assigning_to_users>

<checking_permissions>
```php
// Check single permission
$user->hasPermissionTo('edit articles');
$user->can('edit articles');  // Laravel native

// Check any permission
$user->hasAnyPermission(['edit articles', 'delete articles']);

// Check all permissions
$user->hasAllPermissions(['edit articles', 'delete articles']);

// Check via role (NOT recommended for app logic)
$user->hasRole('editor');
$user->hasAnyRole(['editor', 'writer']);
$user->hasAllRoles(['editor', 'writer']);

// Get all permissions (direct + via roles)
$user->getAllPermissions();

// Get direct permissions only
$user->getDirectPermissions();

// Get permissions via roles
$user->getPermissionsViaRoles();
```
</checking_permissions>

<blade_directives>
```blade
{{-- Check permission --}}
@can('edit articles')
    <a href="{{ route('articles.edit', $article) }}">Edit</a>
@endcan

{{-- Check role (use sparingly) --}}
@role('admin')
    <a href="{{ route('admin.dashboard') }}">Admin</a>
@endrole

{{-- Check any role --}}
@hasanyrole('admin|editor')
    <p>You have elevated access</p>
@endhasanyrole

{{-- Check permission with else --}}
@can('delete articles')
    <button>Delete</button>
@else
    <span>No delete access</span>
@endcan
```
</blade_directives>

<route_middleware>
```php
// Single permission
Route::get('/articles/create', [ArticleController::class, 'create'])
    ->middleware('permission:create articles');

// Multiple permissions (AND)
Route::get('/admin', [AdminController::class, 'index'])
    ->middleware('permission:manage users|manage settings');

// Role middleware
Route::get('/admin', [AdminController::class, 'index'])
    ->middleware('role:admin');

// Role or permission
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('role_or_permission:admin|view dashboard');

// Group with middleware
Route::middleware(['permission:manage articles'])->group(function () {
    Route::get('/articles', [ArticleController::class, 'index']);
    Route::post('/articles', [ArticleController::class, 'store']);
});
```
</route_middleware>

<controller_checks>
```php
class ArticleController extends Controller
{
    public function __construct()
    {
        // Apply middleware in constructor
        $this->middleware('permission:view articles')->only('index', 'show');
        $this->middleware('permission:create articles')->only('create', 'store');
        $this->middleware('permission:edit articles')->only('edit', 'update');
        $this->middleware('permission:delete articles')->only('destroy');
    }

    public function update(Request $request, Article $article)
    {
        // Manual check in method
        if (!auth()->user()->can('edit articles')) {
            abort(403);
        }

        // Or use authorize()
        $this->authorize('update', $article);

        // ...
    }
}
```
</controller_checks>

<form_request_authorization>
```php
// app/Http/Requests/UpdateArticleRequest.php
class UpdateArticleRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('edit articles');
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'body' => 'required|string',
        ];
    }
}
```
</form_request_authorization>

<query_scopes>
```php
// Get users with specific role
$editors = User::role('editor')->get();

// Get users without role
$nonAdmins = User::withoutRole('admin')->get();

// Get users with permission
$canEdit = User::permission('edit articles')->get();
```
</query_scopes>

<wildcard_permissions>
```php
// Create wildcard permission
Permission::create(['name' => 'articles.*']);

// Check wildcard
$user->givePermissionTo('articles.*');
$user->can('articles.view');   // true
$user->can('articles.edit');   // true
$user->can('articles.delete'); // true
```
</wildcard_permissions>
