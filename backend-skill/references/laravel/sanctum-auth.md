<overview>
Integrating Spatie permissions with Laravel Sanctum for API authentication. Covers guards, token abilities, and common pitfalls.
</overview>

<sanctum_setup>
**Add HasApiTokens to User:**

```php
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

**Solution 1: Force single guard (recommended)**

```php
class User extends Authenticatable
{
    use HasApiTokens, HasRoles;

    protected $guard_name = 'web';
}
```

**Solution 2: Separate API permissions**

```php
Permission::create(['name' => 'api.read', 'guard_name' => 'api']);
```
</guard_configuration>

<api_routes>
```php
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/articles', [ArticleController::class, 'index'])
        ->middleware('permission:view articles');

    Route::post('/articles', [ArticleController::class, 'store'])
        ->middleware('permission:create articles');
});
```
</api_routes>

<token_creation>
```php
// Basic token
$token = $user->createToken('api-token');

// Token with abilities
$token = $user->createToken('limited-token', ['read']);

return response()->json([
    'token' => $token->plainTextToken,
]);

// Check token abilities
if ($request->user()->tokenCan('read')) {
    // ...
}
```
</token_creation>

<proxy_permissions_to_tokens>
**Make tokens use Spatie permissions:**

```php
// app/Models/CustomPersonalAccessToken.php
class CustomPersonalAccessToken extends PersonalAccessToken
{
    public function can($ability)
    {
        return $this->tokenable->can($ability);
    }
}

// Register in AppServiceProvider
Sanctum::usePersonalAccessTokenModel(CustomPersonalAccessToken::class);
```
</proxy_permissions_to_tokens>

<api_controller_example>
```php
class ArticleController extends Controller
{
    public function index(Request $request)
    {
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
}
```
</api_controller_example>

<testing_api_auth>
```php
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

// Using actingAs
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

SPA requests automatically use session-based auth and Spatie permissions work normally.
</spa_authentication>
