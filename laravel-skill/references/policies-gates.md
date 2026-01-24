<overview>
Integrating Spatie permissions with Laravel's native authorization system (Gates and Policies) for clean, maintainable code.
</overview>

<gates_with_spatie>
**Registering Gates in AuthServiceProvider:**

```php
// app/Providers/AuthServiceProvider.php
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    // Simple gate using Spatie permission
    Gate::define('access-admin', fn (User $user) => $user->can('access admin'));

    // Gate with model
    Gate::define('update-article', function (User $user, Article $article) {
        return $user->can('edit articles') || $user->id === $article->user_id;
    });

    // Super admin bypass - runs before ALL gates
    Gate::before(function (User $user, string $ability) {
        if ($user->hasRole('super-admin')) {
            return true;
        }
    });
}
```

**Using Gates:**

```php
// In controller
if (Gate::allows('access-admin')) {
    // ...
}

// With response
Gate::authorize('access-admin'); // Throws 403 if denied

// In Blade
@can('access-admin')
    <a href="/admin">Admin Panel</a>
@endcan
```
</gates_with_spatie>

<policy_creation>
**Create policy:**

```bash
php artisan make:policy ArticlePolicy --model=Article
```

**Policy with Spatie permissions:**

```php
// app/Policies/ArticlePolicy.php
namespace App\Policies;

use App\Models\Article;
use App\Models\User;

class ArticlePolicy
{
    /**
     * Runs before all other methods.
     * Return true to allow, false to deny, null to continue to method.
     */
    public function before(User $user, string $ability): ?bool
    {
        if ($user->hasRole('super-admin')) {
            return true;
        }

        return null; // Fall through to specific method
    }

    public function viewAny(User $user): bool
    {
        return $user->can('view articles');
    }

    public function view(User $user, Article $article): bool
    {
        return $user->can('view articles');
    }

    public function create(User $user): bool
    {
        return $user->can('create articles');
    }

    public function update(User $user, Article $article): bool
    {
        // Permission + ownership check
        return $user->can('edit articles') || $user->id === $article->user_id;
    }

    public function delete(User $user, Article $article): bool
    {
        // Only direct permission (no ownership fallback)
        return $user->can('delete articles');
    }

    public function restore(User $user, Article $article): bool
    {
        return $user->can('restore articles');
    }

    public function forceDelete(User $user, Article $article): bool
    {
        return $user->can('force delete articles');
    }
}
```
</policy_creation>

<policy_registration>
**Auto-discovery (Laravel convention):**

Policies in `app/Policies` following naming convention `{Model}Policy` are auto-discovered.

**Manual registration (if needed):**

```php
// app/Providers/AuthServiceProvider.php
protected $policies = [
    Article::class => ArticlePolicy::class,
    Comment::class => CommentPolicy::class,
];
```
</policy_registration>

<using_policies>
**In Controllers:**

```php
class ArticleController extends Controller
{
    public function index()
    {
        $this->authorize('viewAny', Article::class);
        return view('articles.index', ['articles' => Article::all()]);
    }

    public function show(Article $article)
    {
        $this->authorize('view', $article);
        return view('articles.show', compact('article'));
    }

    public function update(Request $request, Article $article)
    {
        $this->authorize('update', $article);
        $article->update($request->validated());
        return redirect()->route('articles.show', $article);
    }
}
```

**In Form Requests:**

```php
class UpdateArticleRequest extends FormRequest
{
    public function authorize(): bool
    {
        $article = $this->route('article');
        return $this->user()->can('update', $article);
    }
}
```

**In Blade:**

```blade
@can('update', $article)
    <a href="{{ route('articles.edit', $article) }}">Edit</a>
@endcan

@can('create', App\Models\Article::class)
    <a href="{{ route('articles.create') }}">New Article</a>
@endcan

@cannot('delete', $article)
    <p>You cannot delete this article</p>
@endcannot
```

**Via User Model:**

```php
$user->can('update', $article);
$user->cannot('delete', $article);
```
</using_policies>

<resource_controller_authorization>
**Authorize entire resource controller:**

```php
class ArticleController extends Controller
{
    public function __construct()
    {
        $this->authorizeResource(Article::class, 'article');
    }

    // Methods now auto-authorized:
    // index -> viewAny
    // create -> create
    // store -> create
    // show -> view
    // edit -> update
    // update -> update
    // destroy -> delete
}
```
</resource_controller_authorization>

<policy_responses>
**Custom denial messages:**

```php
use Illuminate\Auth\Access\Response;

public function update(User $user, Article $article): Response
{
    if ($user->can('edit articles')) {
        return Response::allow();
    }

    if ($user->id === $article->user_id) {
        return Response::allow();
    }

    return Response::deny('You do not own this article and lack edit permission.');
}
```

**Using responses:**

```php
$response = Gate::inspect('update', $article);

if ($response->allowed()) {
    // Update article
} else {
    echo $response->message(); // "You do not own this article..."
}
```
</policy_responses>

<best_practices>
1. **Use policies for model authorization** - Keeps logic organized
2. **Use Spatie permissions inside policies** - Centralized permission definitions
3. **Use `before()` for super-admin** - Single bypass point
4. **Combine permission + ownership** - Common pattern for user content
5. **Keep policies focused** - One policy per model
6. **Test policies directly** - Write unit tests for policy methods
</best_practices>
