<overview>
Backend testing strategies covering unit tests, integration tests, API tests, and test organization. Well-tested code is maintainable code.
</overview>

<testing_pyramid>
**Test Pyramid:**

```
       /\
      /  \     E2E Tests (few)
     /----\
    /      \   Integration Tests (some)
   /--------\
  /          \ Unit Tests (many)
 /------------\
```

**Unit Tests:** Fast, isolated, test single functions
**Integration Tests:** Test components working together
**E2E Tests:** Test full user flows (slowest, most brittle)
</testing_pyramid>

<unit_testing>
**Unit Test Patterns:**

```php
// Test one thing per test
public function test_user_can_be_created(): void
{
    $user = User::factory()->create([
        'email' => 'test@example.com',
    ]);

    $this->assertDatabaseHas('users', [
        'email' => 'test@example.com',
    ]);
}

// Arrange, Act, Assert pattern
public function test_order_total_calculation(): void
{
    // Arrange
    $order = new Order();
    $order->addItem(new Item(price: 100, quantity: 2));
    $order->addItem(new Item(price: 50, quantity: 1));

    // Act
    $total = $order->calculateTotal();

    // Assert
    $this->assertEquals(250, $total);
}

// Test edge cases
public function test_empty_cart_has_zero_total(): void
{
    $cart = new Cart();
    $this->assertEquals(0, $cart->total());
}

public function test_negative_quantity_throws_exception(): void
{
    $this->expectException(InvalidArgumentException::class);
    new Item(price: 100, quantity: -1);
}
```
</unit_testing>

<api_testing>
**API/HTTP Tests:**

```php
// Laravel Feature Test
public function test_can_create_post(): void
{
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->postJson('/api/posts', [
            'title' => 'My Post',
            'body' => 'Post content',
        ]);

    $response->assertStatus(201)
        ->assertJson([
            'data' => [
                'title' => 'My Post',
            ],
        ]);

    $this->assertDatabaseHas('posts', [
        'title' => 'My Post',
        'user_id' => $user->id,
    ]);
}

// Test validation errors
public function test_post_requires_title(): void
{
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->postJson('/api/posts', [
            'body' => 'Content without title',
        ]);

    $response->assertStatus(422)
        ->assertJsonValidationErrors(['title']);
}

// Test authorization
public function test_cannot_delete_others_post(): void
{
    $owner = User::factory()->create();
    $other = User::factory()->create();
    $post = Post::factory()->create(['user_id' => $owner->id]);

    $response = $this->actingAs($other)
        ->deleteJson("/api/posts/{$post->id}");

    $response->assertStatus(403);
}
```
</api_testing>

<database_testing>
**Database Testing:**

```php
use RefreshDatabase;  // Reset DB for each test

// Or use DatabaseTransactions for faster tests
use DatabaseTransactions;

// Factories
$user = User::factory()->create();  // Persist to DB
$user = User::factory()->make();    // In-memory only

// Factory states
User::factory()->admin()->create();
User::factory()->unverified()->create();

// Relationships
$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();

// Assert database state
$this->assertDatabaseHas('users', ['email' => 'test@example.com']);
$this->assertDatabaseMissing('users', ['email' => 'deleted@example.com']);
$this->assertDatabaseCount('users', 5);
$this->assertSoftDeleted('posts', ['id' => $post->id]);
```
</database_testing>

<mocking>
**Mocking External Services:**

```php
// Mock HTTP client
Http::fake([
    'api.stripe.com/*' => Http::response(['id' => 'ch_123'], 200),
    'api.sendgrid.com/*' => Http::response([], 202),
]);

$response = $this->postJson('/api/charge', ['amount' => 1000]);
$response->assertStatus(200);

Http::assertSent(function ($request) {
    return $request->url() === 'https://api.stripe.com/v1/charges';
});

// Mock any class
$this->mock(PaymentGateway::class, function ($mock) {
    $mock->shouldReceive('charge')
        ->once()
        ->with(1000)
        ->andReturn(new Charge(['id' => 'ch_123']));
});

// Spy on calls
$spy = $this->spy(Mailer::class);
// ... do stuff ...
$spy->shouldHaveReceived('send')->once();

// Fake events
Event::fake();
// ... do stuff ...
Event::assertDispatched(OrderCreated::class);

// Fake queues
Queue::fake();
// ... do stuff ...
Queue::assertPushed(SendEmail::class);
```
</mocking>

<test_organization>
**Test Organization:**

```
tests/
├── Unit/                    # Isolated unit tests
│   ├── Models/
│   │   └── UserTest.php
│   └── Services/
│       └── OrderServiceTest.php
├── Feature/                 # Integration/API tests
│   ├── Auth/
│   │   └── LoginTest.php
│   └── Api/
│       └── PostsTest.php
├── TestCase.php            # Base test class
└── CreatesApplication.php
```

**Naming Conventions:**

```php
// Method names describe behavior
public function test_user_can_create_post(): void
public function test_guest_cannot_create_post(): void
public function test_post_requires_title(): void
public function test_user_can_only_edit_own_posts(): void

// Or with @test annotation
/** @test */
public function user_can_create_post(): void
```
</test_organization>

<fixtures>
**Test Data Fixtures:**

```php
// Factories (recommended)
class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'password' => bcrypt('password'),
        ];
    }

    public function admin(): static
    {
        return $this->state(['role' => 'admin']);
    }

    public function unverified(): static
    {
        return $this->state(['email_verified_at' => null]);
    }
}

// Seeders for complex test data
class TestDatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $admin = User::factory()->admin()->create([
            'email' => 'admin@example.com',
        ]);

        Category::factory()->count(5)
            ->has(Product::factory()->count(10))
            ->create();
    }
}
```
</fixtures>

<async_testing>
**Testing Async/Queue Jobs:**

```php
// Test job is dispatched
Queue::fake();

$this->postJson('/api/orders', $orderData);

Queue::assertPushed(ProcessOrder::class, function ($job) use ($order) {
    return $job->order->id === $order->id;
});

// Test job execution
public function test_process_order_job(): void
{
    $order = Order::factory()->create(['status' => 'pending']);

    (new ProcessOrder($order))->handle();

    $this->assertEquals('processed', $order->fresh()->status);
}

// Test with real queue (integration)
public function test_order_processing_flow(): void
{
    $order = Order::factory()->create();

    ProcessOrder::dispatch($order);

    // Process queue synchronously
    Artisan::call('queue:work', ['--once' => true]);

    $this->assertEquals('processed', $order->fresh()->status);
}
```
</async_testing>

<performance_testing>
**Performance Testing:**

```php
// Assert query count
$this->assertDatabaseQueryCount(2, function () {
    $posts = Post::with('author')->get();
    foreach ($posts as $post) {
        $post->author->name;
    }
});

// Assert execution time
$start = microtime(true);
$this->getJson('/api/products');
$duration = microtime(true) - $start;
$this->assertLessThan(0.5, $duration, 'Endpoint too slow');
```
</performance_testing>

<ci_integration>
**CI Integration:**

```yaml
# GitHub Actions
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql:8
        env:
          MYSQL_ROOT_PASSWORD: password
          MYSQL_DATABASE: test

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: 8.3
          coverage: xdebug

      - name: Install Dependencies
        run: composer install

      - name: Run Tests
        run: php artisan test --coverage
        env:
          DB_CONNECTION: mysql
          DB_HOST: 127.0.0.1
```
</ci_integration>

<best_practices>
**Testing Best Practices:**

- Test behavior, not implementation
- One assertion per test (when practical)
- Use descriptive test names
- Keep tests fast (mock slow things)
- Don't test framework code
- Test edge cases and error paths
- Run tests in CI on every commit
- Aim for high coverage on critical paths
- Tests are documentation
</best_practices>
