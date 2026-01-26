<overview>
Laravel 12 patterns and best practices. Covers Eloquent ORM, Spatie permissions, Sanctum authentication, queues, and modern Laravel architecture.
</overview>

<project_structure>
**Recommended Structure:**

```
app/
├── Http/
│   ├── Controllers/      # Thin controllers
│   ├── Requests/         # Form request validation
│   ├── Resources/        # API resources
│   └── Middleware/
├── Models/               # Eloquent models
├── Services/             # Business logic
├── Actions/              # Single-purpose classes
├── Events/               # Domain events
├── Listeners/            # Event handlers
├── Jobs/                 # Queue jobs
├── Policies/             # Authorization policies
└── Enums/                # PHP 8.1+ enums

config/
database/
├── migrations/
├── factories/
└── seeders/
routes/
├── web.php
├── api.php
└── console.php
```
</project_structure>

<eloquent_patterns>
**Model Best Practices:**

```php
class User extends Model
{
    use HasFactory, HasRoles;

    // Explicit fillable (safer than guarded)
    protected $fillable = ['name', 'email', 'password'];

    // Hidden from serialization
    protected $hidden = ['password', 'remember_token'];

    // Cast attributes
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'settings' => 'array',
            'status' => UserStatus::class,  // Enum cast
        ];
    }

    // Relationships
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }

    public function profile(): HasOne
    {
        return $this->hasOne(Profile::class);
    }

    // Scopes
    public function scopeActive(Builder $query): Builder
    {
        return $query->where('status', UserStatus::Active);
    }

    public function scopeCreatedAfter(Builder $query, Carbon $date): Builder
    {
        return $query->where('created_at', '>', $date);
    }

    // Accessors (Laravel 9+)
    protected function fullName(): Attribute
    {
        return Attribute::make(
            get: fn () => "{$this->first_name} {$this->last_name}",
        );
    }
}

// Usage
$users = User::active()->createdAfter(now()->subMonth())->get();
```

**Query Optimization:**

```php
// Eager load to prevent N+1
$posts = Post::with(['author', 'comments.user'])->get();

// Select only needed columns
$users = User::select(['id', 'name', 'email'])->get();

// Chunk for large datasets
User::chunk(1000, function ($users) {
    foreach ($users as $user) {
        // Process
    }
});

// Cursor for memory efficiency
foreach (User::cursor() as $user) {
    // Process one at a time
}

// Lazy loading prevention
Model::preventLazyLoading(!app()->isProduction());
```
</eloquent_patterns>

<controllers>
**Controller Patterns:**

```php
// Thin controller - delegates to service
class OrderController extends Controller
{
    public function __construct(
        private OrderService $orderService,
    ) {}

    public function store(StoreOrderRequest $request): JsonResponse
    {
        $order = $this->orderService->create(
            $request->validated(),
            $request->user()
        );

        return OrderResource::make($order)
            ->response()
            ->setStatusCode(201);
    }

    public function show(Order $order): OrderResource
    {
        $this->authorize('view', $order);
        return OrderResource::make($order->load('items'));
    }
}

// Form Request for validation
class StoreOrderRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Order::class);
    }

    public function rules(): array
    {
        return [
            'items' => 'required|array|min:1',
            'items.*.product_id' => 'required|exists:products,id',
            'items.*.quantity' => 'required|integer|min:1',
            'shipping_address' => 'required|string|max:500',
        ];
    }
}
```

**Single Action Controllers:**

```php
// For simple endpoints
class CompleteOrderController extends Controller
{
    public function __invoke(Order $order): JsonResponse
    {
        $this->authorize('complete', $order);

        $order->complete();

        return response()->json(['message' => 'Order completed']);
    }
}

// Route
Route::post('/orders/{order}/complete', CompleteOrderController::class);
```
</controllers>

<api_resources>
**API Resources:**

```php
class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'excerpt' => Str::limit($this->body, 100),
            'author' => UserResource::make($this->whenLoaded('author')),
            'comments_count' => $this->whenCounted('comments'),
            'created_at' => $this->created_at->toIso8601String(),
            'links' => [
                'self' => route('posts.show', $this),
            ],
        ];
    }
}

// Collection with meta
class PostCollection extends ResourceCollection
{
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'meta' => [
                'total' => $this->total(),
                'per_page' => $this->perPage(),
            ],
        ];
    }
}
```
</api_resources>

<services>
**Service Pattern:**

```php
class OrderService
{
    public function __construct(
        private PaymentGateway $payments,
        private InventoryService $inventory,
    ) {}

    public function create(array $data, User $user): Order
    {
        return DB::transaction(function () use ($data, $user) {
            $order = $user->orders()->create([
                'status' => OrderStatus::Pending,
                'total' => $this->calculateTotal($data['items']),
            ]);

            foreach ($data['items'] as $item) {
                $order->items()->create($item);
                $this->inventory->reserve($item['product_id'], $item['quantity']);
            }

            event(new OrderCreated($order));

            return $order;
        });
    }

    public function complete(Order $order): void
    {
        $this->payments->charge($order->user, $order->total);

        $order->update(['status' => OrderStatus::Completed]);

        event(new OrderCompleted($order));
    }
}
```
</services>

<spatie_permissions>
**Spatie laravel-permission:**

```php
// Setup
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;
}

// Create roles and permissions (seeder)
$admin = Role::create(['name' => 'admin']);
$editor = Role::create(['name' => 'editor']);

Permission::create(['name' => 'posts.create']);
Permission::create(['name' => 'posts.edit']);
Permission::create(['name' => 'posts.delete']);

$admin->givePermissionTo(Permission::all());
$editor->givePermissionTo(['posts.create', 'posts.edit']);

// Assign to users
$user->assignRole('editor');
$user->givePermissionTo('posts.delete');

// Check permissions (NOT roles)
if ($user->can('posts.edit')) { ... }

// Middleware
Route::middleware('permission:posts.edit')->get('/posts/{post}/edit', ...);
Route::middleware('role:admin')->prefix('admin')->group(...);

// Blade
@can('posts.edit')
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@endcan
```

**Super Admin Pattern:**

```php
// AuthServiceProvider
Gate::before(function (User $user) {
    return $user->hasRole('super-admin') ? true : null;
});
```
</spatie_permissions>

<sanctum>
**API Authentication with Sanctum:**

```php
// Install
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

// Issue tokens
$token = $user->createToken('api', ['posts:read', 'posts:write']);
return ['token' => $token->plainTextToken];

// Protect routes
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user', fn (Request $request) => $request->user());
});

// Check abilities
if ($request->user()->tokenCan('posts:write')) {
    // Allow
}

// Revoke tokens
$user->currentAccessToken()->delete();  // Current
$user->tokens()->delete();  // All
```

**SPA Authentication:**

```php
// For same-domain SPAs
// 1. Call /sanctum/csrf-cookie
// 2. Include credentials in requests
// 3. Use session-based auth
```
</sanctum>

<queues>
**Queue Jobs:**

```php
class ProcessOrder implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $tries = 3;
    public $backoff = [30, 60, 120];

    public function __construct(
        public Order $order,
    ) {}

    public function handle(PaymentGateway $payments): void
    {
        $payments->charge($this->order->user, $this->order->total);
        $this->order->update(['status' => OrderStatus::Paid]);
    }

    public function failed(\Throwable $exception): void
    {
        $this->order->update(['status' => OrderStatus::Failed]);
        Log::error('Order processing failed', [
            'order_id' => $this->order->id,
            'error' => $exception->getMessage(),
        ]);
    }
}

// Dispatch
ProcessOrder::dispatch($order);
ProcessOrder::dispatch($order)->delay(now()->addMinutes(5));
ProcessOrder::dispatch($order)->onQueue('payments');
```
</queues>

<events>
**Events and Listeners:**

```php
// Event
class OrderCreated
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}
}

// Listener
class SendOrderConfirmation implements ShouldQueue
{
    public function handle(OrderCreated $event): void
    {
        Mail::to($event->order->user)->send(new OrderConfirmationMail($event->order));
    }
}

// Register in EventServiceProvider
protected $listen = [
    OrderCreated::class => [
        SendOrderConfirmation::class,
        UpdateInventory::class,
        NotifyAdmin::class,
    ],
];

// Or auto-discovery
php artisan event:generate
```
</events>

<testing>
**Testing Patterns:**

```php
class OrderTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_create_order(): void
    {
        $user = User::factory()->create();
        $product = Product::factory()->create(['price' => 1000]);

        $response = $this->actingAs($user)
            ->postJson('/api/orders', [
                'items' => [
                    ['product_id' => $product->id, 'quantity' => 2],
                ],
            ]);

        $response->assertCreated();
        $this->assertDatabaseHas('orders', [
            'user_id' => $user->id,
            'total' => 2000,
        ]);
    }

    public function test_guest_cannot_create_order(): void
    {
        $response = $this->postJson('/api/orders', []);
        $response->assertUnauthorized();
    }
}
```
</testing>

<quick_reference>
**Artisan Commands:**

```bash
# Cache
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan cache:clear

# Database
php artisan migrate
php artisan migrate:fresh --seed
php artisan db:seed

# Development
php artisan make:model Post -mfc
php artisan make:controller PostController --api
php artisan make:request StorePostRequest

# Queue
php artisan queue:work
php artisan queue:failed
php artisan queue:retry all
```
</quick_reference>
