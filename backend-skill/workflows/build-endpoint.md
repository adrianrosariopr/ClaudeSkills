# Workflow: Build Endpoint

<required_reading>
**Read these reference files NOW:**
1. references/core/api-design.md
2. references/core/security.md
3. Framework-specific patterns file for the user's framework
</required_reading>

<context>
Step-by-step guide to building a new API endpoint from scratch, following best practices for validation, authorization, error handling, and testing.
</context>

<process>

<step name="clarify-requirements">
Before building, understand:

- What resource/action does this endpoint handle?
- HTTP method: GET, POST, PUT, PATCH, DELETE?
- Authentication required?
- What permissions needed?
- What input parameters?
- What response format?
</step>

<step name="define-route">
Add the route:

**Laravel:**
```php
// routes/api.php
Route::middleware(['auth:sanctum'])->group(function () {
    Route::post('/posts', [PostController::class, 'store']);
    Route::put('/posts/{post}', [PostController::class, 'update']);
});
```

**Node.js (Express):**
```typescript
// routes/posts.ts
router.post('/', authenticate, validate(createPostSchema), postController.create);
router.put('/:id', authenticate, validate(updatePostSchema), postController.update);
```

**Python (FastAPI):**
```python
@router.post("/", response_model=PostResponse, status_code=201)
async def create_post(
    post: CreatePostInput,
    user: User = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    ...
```

**Go (Gin):**
```go
posts := api.Group("/posts")
posts.Use(authMiddleware)
{
    posts.POST("", handlers.CreatePost)
    posts.PUT("/:id", handlers.UpdatePost)
}
```
</step>

<step name="create-validation">
Validate all input:

**Laravel (Form Request):**
```php
class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Post::class);
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'content' => 'required|string',
            'published' => 'boolean',
            'tags' => 'array|max:10',
            'tags.*' => 'exists:tags,id',
        ];
    }
}
```

**Node.js (Zod):**
```typescript
const createPostSchema = z.object({
    title: z.string().min(1).max(255),
    content: z.string().min(1),
    published: z.boolean().optional().default(false),
    tags: z.array(z.number()).max(10).optional(),
});
```

**Python (Pydantic):**
```python
class CreatePostInput(BaseModel):
    title: str = Field(..., min_length=1, max_length=255)
    content: str
    published: bool = False
    tags: List[int] = Field(default_factory=list, max_items=10)
```
</step>

<step name="implement-handler">
Write the controller/handler:

**Laravel:**
```php
class PostController extends Controller
{
    public function store(StorePostRequest $request): JsonResponse
    {
        $post = $request->user()->posts()->create([
            'title' => $request->title,
            'content' => $request->content,
            'published' => $request->boolean('published'),
        ]);

        if ($request->has('tags')) {
            $post->tags()->sync($request->tags);
        }

        return PostResource::make($post->load('tags'))
            ->response()
            ->setStatusCode(201);
    }
}
```

**Node.js:**
```typescript
async create(req: Request, res: Response) {
    const { title, content, published, tags } = req.body;

    const post = await prisma.post.create({
        data: {
            title,
            content,
            published,
            authorId: req.userId,
            tags: tags ? { connect: tags.map(id => ({ id })) } : undefined,
        },
        include: { tags: true },
    });

    res.status(201).json(post);
}
```

**Python (FastAPI):**
```python
@router.post("/", response_model=PostResponse, status_code=201)
async def create_post(
    data: CreatePostInput,
    user: User = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    post = Post(**data.dict(), author_id=user.id)
    db.add(post)
    db.commit()
    db.refresh(post)
    return post
```
</step>

<step name="add-authorization">
Ensure proper authorization:

```php
// Laravel Policy
class PostPolicy
{
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->can('posts.edit');
    }
}

// In controller
public function update(UpdatePostRequest $request, Post $post): JsonResponse
{
    $this->authorize('update', $post);
    // ...
}
```
</step>

<step name="format-response">
Use consistent response format:

**Laravel Resource:**
```php
class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'content' => $this->content,
            'published' => $this->published,
            'author' => UserResource::make($this->whenLoaded('author')),
            'tags' => TagResource::collection($this->whenLoaded('tags')),
            'created_at' => $this->created_at->toIso8601String(),
        ];
    }
}
```
</step>

<step name="handle-errors">
Return appropriate error responses:

```php
// 400 - Bad Request (malformed input)
// 401 - Unauthorized (not authenticated)
// 403 - Forbidden (not authorized)
// 404 - Not Found
// 422 - Validation failed
// 500 - Server error

// Laravel exception handling
try {
    $result = $service->process($data);
} catch (NotFoundException $e) {
    abort(404, 'Resource not found');
} catch (ValidationException $e) {
    // Automatically returns 422
    throw $e;
}
```
</step>

<step name="write-tests">
Test the endpoint:

**Laravel:**
```php
public function test_can_create_post(): void
{
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->postJson('/api/posts', [
            'title' => 'Test Post',
            'content' => 'Test content',
        ]);

    $response->assertStatus(201)
        ->assertJsonPath('data.title', 'Test Post');

    $this->assertDatabaseHas('posts', [
        'title' => 'Test Post',
        'user_id' => $user->id,
    ]);
}

public function test_guest_cannot_create_post(): void
{
    $response = $this->postJson('/api/posts', [
        'title' => 'Test Post',
        'content' => 'Test content',
    ]);

    $response->assertStatus(401);
}

public function test_post_requires_title(): void
{
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->postJson('/api/posts', [
            'content' => 'Test content',
        ]);

    $response->assertStatus(422)
        ->assertJsonValidationErrors(['title']);
}
```
</step>

<step name="verify">
Verify the endpoint works:

```bash
# Build passes
php artisan route:list | grep posts
npm run build

# Tests pass
php artisan test --filter=PostTest

# Manual test
curl -X POST http://localhost:8000/api/posts \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title": "Test", "content": "Content"}'
```
</step>

</process>

<anti_patterns>
Avoid:
- Skipping validation
- Not checking authorization
- Inconsistent response formats
- Missing error handling
- Fat controllers (move logic to services)
- Not writing tests
- Exposing sensitive data in response
</anti_patterns>

<success_criteria>
A well-built endpoint:
- Route defined with appropriate middleware
- All input validated
- Authorization checked
- Business logic in service layer
- Consistent response format
- Appropriate HTTP status codes
- Error handling complete
- Tests written and passing
</success_criteria>
