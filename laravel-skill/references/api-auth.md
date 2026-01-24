<overview>
Integrating Spatie permissions with Laravel Sanctum for API authentication. Covers guards, token abilities, and common pitfalls.
</overview>

<sanctum_setup>
**Install Sanctum (Laravel 12):**

Sanctum is included in Laravel 12 starter kits. For manual installation:

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

**Add HasApiTokens to User:**

```php
// app/Models/User.php
use Laravel\Sanctum\HasApiTokens;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasApiTokens, HasRoles;
}
```
</sanctum_setup>

<guard_configuration>
**The Guard Problem:**

Spatie uses predefined guards (web, api). Sanctum uses the `sanctum` guard. This causes:
```
The given role or permission should use guard `web, api` instead of `sanctum`
```

**Solution 1: Force single guard (recommended for most apps)**

```php
// app/Models/User.php
class User extends Authenticatable
{
    use HasApiTokens, HasRoles;

    // Force web guard for permissions
    protected $guard_name = 'web';
}
```

**Solution 2: Separate API permissions**

Create permissions with `api` guard:

```php
Permission::create(['name' => 'api.read', 'guard_name' => 'api']);
Permission::create(['name' => 'api.write', 'guard_name' => 'api']);
```

**Solution 3: Configure Sanctum to use web guard**

```php
// config/auth.php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'sanctum' => [
        'driver' => 'sanctum',
        'provider' => 'users',
    ],
],

'defaults' => [
    'guard' => 'web',  // Use web as default
],
```
</guard_configuration>

<api_routes_with_permissions>
```php
// routes/api.php
use Illuminate\Support\Facades\Route;

Route::middleware('auth:sanctum')->group(function () {
    // Check permission via middleware
    Route::get('/articles', [ArticleController::class, 'index'])
        ->middleware('permission:view articles');

    Route::post('/articles', [ArticleController::class, 'store'])
        ->middleware('permission:create articles');

    // Or check in controller
    Route::apiResource('users', UserController::class);
});
```
</api_routes_with_permissions>

<token_creation>
**Create token with abilities:**

```php
// Basic token (inherits user permissions)
$token = $user->createToken('api-token');

// Token with specific abilities
$token = $user->createToken('limited-token', ['read']);

// Return token
return response()->json([
    'token' => $token->plainTextToken,
]);
```

**Check token abilities:**

```php
// In controller
if ($request->user()->tokenCan('read')) {
    // ...
}
```
</token_creation>

<proxy_permissions_to_tokens>
**Make tokens use Spatie permissions:**

```php
// app/Models/CustomPersonalAccessToken.php
namespace App\Models;

use Laravel\Sanctum\PersonalAccessToken;

class CustomPersonalAccessToken extends PersonalAccessToken
{
    public function can($ability)
    {
        // Proxy to user's Spatie permissions
        return $this->tokenable->can($ability);
    }
}
```

**Register in AppServiceProvider:**

```php
// app/Providers/AppServiceProvider.php
use App\Models\CustomPersonalAccessToken;
use Laravel\Sanctum\Sanctum;

public function boot(): void
{
    Sanctum::usePersonalAccessTokenModel(CustomPersonalAccessToken::class);
}
```

Now token abilities mirror user permissions.
</proxy_permissions_to_tokens>

<api_controller_example>
```php
// app/Http/Controllers/Api/ArticleController.php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Article;
use Illuminate\Http\Request;

class ArticleController extends Controller
{
    public function index(Request $request)
    {
        // Check via user permission
        if (!$request->user()->can('view articles')) {
            return response()->json(['error' => 'Forbidden'], 403);
        }

        return Article::paginate();
    }

    public function store(Request $request)
    {
        $this->authorize('create', Article::class);

        $article = Article::create($request->validated());

        return response()->json($article, 201);
    }

    public function update(Request $request, Article $article)
    {
        $this->authorize('update', $article);

        $article->update($request->validated());

        return response()->json($article);
    }

    public function destroy(Article $article)
    {
        $this->authorize('delete', $article);

        $article->delete();

        return response()->noContent();
    }
}
```
</api_controller_example>

<form_request_api>
```php
// app/Http/Requests/Api/StoreArticleRequest.php
namespace App\Http\Requests\Api;

use Illuminate\Foundation\Http\FormRequest;

class StoreArticleRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create articles');
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
</form_request_api>

<testing_api_auth>
```php
// tests/Feature/Api/ArticleApiTest.php
use App\Models\User;
use App\Models\Article;
use Spatie\Permission\Models\Permission;

public function test_user_with_permission_can_list_articles(): void
{
    Permission::create(['name' => 'view articles']);
    $user = User::factory()->create();
    $user->givePermissionTo('view articles');

    $token = $user->createToken('test')->plainTextToken;

    $this->withToken($token)
        ->getJson('/api/articles')
        ->assertStatus(200);
}

public function test_user_without_permission_gets_403(): void
{
    $user = User::factory()->create();
    $token = $user->createToken('test')->plainTextToken;

    $this->withToken($token)
        ->getJson('/api/articles')
        ->assertStatus(403);
}

// Using actingAs with Sanctum
public function test_using_acting_as(): void
{
    $user = User::factory()->create();
    $user->givePermissionTo('view articles');

    $this->actingAs($user, 'sanctum')
        ->getJson('/api/articles')
        ->assertStatus(200);
}
```
</testing_api_auth>

<spa_authentication>
**For SPA (same domain):**

Sanctum's SPA authentication uses cookies, not tokens.

```php
// config/sanctum.php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 'localhost')),
```

```php
// routes/web.php
Route::post('/login', function (Request $request) {
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    if (Auth::attempt($credentials)) {
        $request->session()->regenerate();
        return response()->json(['user' => Auth::user()]);
    }

    return response()->json(['error' => 'Invalid credentials'], 401);
});
```

SPA requests automatically use session-based auth and Spatie permissions work normally.
</spa_authentication>
