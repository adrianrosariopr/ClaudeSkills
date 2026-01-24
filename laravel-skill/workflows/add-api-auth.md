# Workflow: Add API Authentication with Permissions

<required_reading>
**Read these reference files NOW:**
1. references/api-auth.md
2. references/architecture.md
3. references/testing.md
</required_reading>

<process>
## Step 1: Install Laravel Sanctum

Sanctum is included in Laravel 12 starter kits. For manual installation:

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

## Step 2: Configure User Model

Add both HasApiTokens and HasRoles traits:

```php
// app/Models/User.php
namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Sanctum\HasApiTokens;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasApiTokens, HasRoles;

    // Force web guard to avoid guard conflicts
    protected $guard_name = 'web';
}
```

## Step 3: Configure Authentication Guards

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
    'guard' => 'web',
    'passwords' => 'users',
],
```

## Step 4: Create API Routes with Permission Middleware

```php
// routes/api.php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\ArticleController;
use App\Http\Controllers\Api\UserController;
use App\Http\Controllers\Api\AuthController;

// Public routes
Route::post('/login', [AuthController::class, 'login']);
Route::post('/register', [AuthController::class, 'register']);

// Protected routes
Route::middleware('auth:sanctum')->group(function () {
    // User info
    Route::get('/user', [AuthController::class, 'user']);
    Route::post('/logout', [AuthController::class, 'logout']);

    // Articles - permission protected
    Route::get('/articles', [ArticleController::class, 'index'])
        ->middleware('permission:view articles');

    Route::post('/articles', [ArticleController::class, 'store'])
        ->middleware('permission:create articles');

    Route::get('/articles/{article}', [ArticleController::class, 'show'])
        ->middleware('permission:view articles');

    Route::put('/articles/{article}', [ArticleController::class, 'update'])
        ->middleware('permission:edit articles');

    Route::delete('/articles/{article}', [ArticleController::class, 'destroy'])
        ->middleware('permission:delete articles');

    // Users - admin only
    Route::middleware('permission:manage users')->group(function () {
        Route::apiResource('users', UserController::class);
    });
});
```

## Step 5: Create Auth Controller

```php
// app/Http/Controllers/Api/AuthController.php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    public function login(Request $request)
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
            'device_name' => 'required',
        ]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        $token = $user->createToken($request->device_name)->plainTextToken;

        return response()->json([
            'token' => $token,
            'user' => $user,
            'permissions' => $user->getAllPermissions()->pluck('name'),
            'roles' => $user->getRoleNames(),
        ]);
    }

    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:8|confirmed',
            'device_name' => 'required',
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        // Assign default role
        $user->assignRole('user');

        $token = $user->createToken($request->device_name)->plainTextToken;

        return response()->json([
            'token' => $token,
            'user' => $user,
        ], 201);
    }

    public function user(Request $request)
    {
        $user = $request->user();

        return response()->json([
            'user' => $user,
            'permissions' => $user->getAllPermissions()->pluck('name'),
            'roles' => $user->getRoleNames(),
        ]);
    }

    public function logout(Request $request)
    {
        // Revoke current token
        $request->user()->currentAccessToken()->delete();

        return response()->json(['message' => 'Logged out']);
    }
}
```

## Step 6: Create API Controller with Authorization

```php
// app/Http/Controllers/Api/ArticleController.php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Article;
use Illuminate\Http\Request;

class ArticleController extends Controller
{
    public function index()
    {
        return Article::with('user:id,name')->paginate(15);
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'body' => 'required|string',
        ]);

        $article = $request->user()->articles()->create($validated);

        return response()->json($article, 201);
    }

    public function show(Article $article)
    {
        return $article->load('user:id,name');
    }

    public function update(Request $request, Article $article)
    {
        // Additional ownership check via policy
        $this->authorize('update', $article);

        $validated = $request->validate([
            'title' => 'sometimes|required|string|max:255',
            'body' => 'sometimes|required|string',
        ]);

        $article->update($validated);

        return response()->json($article);
    }

    public function destroy(Article $article)
    {
        $article->delete();

        return response()->noContent();
    }
}
```

## Step 7: Handle 403 Responses

```php
// app/Exceptions/Handler.php (or bootstrap/app.php in Laravel 11+)
use Spatie\Permission\Exceptions\UnauthorizedException;
use Illuminate\Auth\Access\AuthorizationException;

// In bootstrap/app.php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (UnauthorizedException $e, Request $request) {
        if ($request->expectsJson()) {
            return response()->json([
                'message' => 'You do not have permission to perform this action.',
                'required_permissions' => $e->getRequiredPermissions(),
            ], 403);
        }
    });

    $exceptions->render(function (AuthorizationException $e, Request $request) {
        if ($request->expectsJson()) {
            return response()->json([
                'message' => $e->getMessage() ?: 'Unauthorized.',
            ], 403);
        }
    });
})
```

## Step 8: (Optional) Proxy Permissions to Token Abilities

Make tokens use user's Spatie permissions:

```php
// app/Models/CustomPersonalAccessToken.php
namespace App\Models;

use Laravel\Sanctum\PersonalAccessToken;

class CustomPersonalAccessToken extends PersonalAccessToken
{
    /**
     * Determine if the token has a given ability.
     * Proxies to user's Spatie permissions.
     */
    public function can($ability)
    {
        // Check if token was issued with specific abilities
        if (parent::can($ability)) {
            return true;
        }

        // Fall back to user's permissions
        return $this->tokenable?->can($ability) ?? false;
    }
}
```

Register in AppServiceProvider:

```php
// app/Providers/AppServiceProvider.php
use App\Models\CustomPersonalAccessToken;
use Laravel\Sanctum\Sanctum;

public function boot(): void
{
    Sanctum::usePersonalAccessTokenModel(CustomPersonalAccessToken::class);
}
```

## Step 9: Create API Tests

```php
// tests/Feature/Api/AuthTest.php
namespace Tests\Feature\Api;

use App\Models\User;
use Database\Seeders\TestPermissionsSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class AuthTest extends TestCase
{
    use RefreshDatabase;

    protected function setUp(): void
    {
        parent::setUp();
        $this->seed(TestPermissionsSeeder::class);
    }

    public function test_user_can_login_and_get_token(): void
    {
        $user = User::factory()->create([
            'password' => bcrypt('password'),
        ]);

        $response = $this->postJson('/api/login', [
            'email' => $user->email,
            'password' => 'password',
            'device_name' => 'test',
        ]);

        $response->assertOk()
            ->assertJsonStructure(['token', 'user', 'permissions', 'roles']);
    }

    public function test_user_with_permission_can_access_protected_route(): void
    {
        $user = User::factory()->create();
        $user->givePermissionTo('view articles');

        $response = $this->actingAs($user, 'sanctum')
            ->getJson('/api/articles');

        $response->assertOk();
    }

    public function test_user_without_permission_gets_403(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user, 'sanctum')
            ->getJson('/api/articles');

        $response->assertForbidden();
    }
}
```

## Step 10: Document API for Consumers

```markdown
## Authentication

### Login
POST /api/login
{
    "email": "user@example.com",
    "password": "password",
    "device_name": "mobile-app"
}

Response:
{
    "token": "1|abc123...",
    "user": {...},
    "permissions": ["view articles", "create articles"],
    "roles": ["editor"]
}

### Using Token
Include in requests:
Authorization: Bearer 1|abc123...

### Permissions Required
- GET /api/articles: `view articles`
- POST /api/articles: `create articles`
- PUT /api/articles/{id}: `edit articles`
- DELETE /api/articles/{id}: `delete articles`
```
</process>

<anti_patterns>
Avoid:
- Not setting guard_name on User model (causes guard conflicts)
- Forgetting to seed permissions in tests
- Not handling 403 responses for JSON requests
- Exposing tokens in responses unnecessarily
</anti_patterns>

<success_criteria>
API auth complete when:
- [ ] Sanctum installed and configured
- [ ] User model has both HasApiTokens and HasRoles
- [ ] Guard name set on User model
- [ ] API routes protected with permission middleware
- [ ] Auth controller handles login/logout/register
- [ ] 403 responses formatted for JSON
- [ ] API tests passing
- [ ] API documented for consumers
</success_criteria>
