<overview>
Common backend anti-patterns and how to avoid them. Learn from others' mistakes to write better code.
</overview>

<n_plus_one>
**Anti-Pattern: N+1 Queries**

```php
// BAD - Executes N+1 queries
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name;  // 1 query per post
}
// 1 query for posts + N queries for authors = N+1

// GOOD - Eager load relationships
$posts = Post::with('author')->get();
foreach ($posts as $post) {
    echo $post->author->name;  // No additional queries
}
// 2 queries total: posts + authors
```

**Detection:**
- Enable query logging in development
- Use Laravel Debugbar or similar
- Monitor query count in tests
</n_plus_one>

<fat_controllers>
**Anti-Pattern: Fat Controllers**

```php
// BAD - Controller does everything
class OrderController
{
    public function store(Request $request)
    {
        // Validation
        $request->validate([...]);

        // Business logic
        $order = new Order();
        $order->user_id = auth()->id();
        // ... 50 more lines of logic

        // Send email
        Mail::to($user)->send(new OrderConfirmation($order));

        // Update inventory
        foreach ($order->items as $item) {
            $item->product->decrement('stock', $item->quantity);
        }

        return redirect()->route('orders.show', $order);
    }
}

// GOOD - Thin controller, delegate to services
class OrderController
{
    public function store(StoreOrderRequest $request, OrderService $orderService)
    {
        $order = $orderService->create($request->validated());
        return redirect()->route('orders.show', $order);
    }
}

class OrderService
{
    public function create(array $data): Order
    {
        return DB::transaction(function () use ($data) {
            $order = Order::create($data);
            $this->processItems($order, $data['items']);
            event(new OrderCreated($order));
            return $order;
        });
    }
}
```
</fat_controllers>

<god_models>
**Anti-Pattern: God Models**

```php
// BAD - Model does everything
class User extends Model
{
    public function sendWelcomeEmail() { ... }
    public function chargeSubscription() { ... }
    public function generateReport() { ... }
    public function syncWithCRM() { ... }
    // 500+ lines of methods
}

// GOOD - Single responsibility
class User extends Model
{
    // Only persistence and relationships
    public function orders() { return $this->hasMany(Order::class); }
    public function subscription() { return $this->hasOne(Subscription::class); }
}

// Delegate to services
class UserService { public function sendWelcomeEmail(User $user) { ... } }
class BillingService { public function chargeSubscription(User $user) { ... } }
class ReportService { public function generateForUser(User $user) { ... } }
```
</god_models>

<premature_optimization>
**Anti-Pattern: Premature Optimization**

```php
// BAD - Optimizing without measuring
$users = User::select(DB::raw('id, CONCAT(first_name, " ", last_name) as name'))
    ->whereRaw('YEAR(created_at) = 2024')
    ->get();

// GOOD - Simple first, optimize when needed
$users = User::whereYear('created_at', 2024)->get();
// Measure. If slow, THEN optimize with evidence.
```

**Rule:** Make it work, make it right, make it fast (in that order).
</premature_optimization>

<stringly_typed>
**Anti-Pattern: Stringly Typed Code**

```php
// BAD - Magic strings everywhere
$order->status = 'pending';
if ($order->status === 'procesing') { ... }  // Typo!
$order->update(['status' => 'shipped']);

// GOOD - Enums/constants
enum OrderStatus: string
{
    case Pending = 'pending';
    case Processing = 'processing';
    case Shipped = 'shipped';
    case Delivered = 'delivered';
}

$order->status = OrderStatus::Pending;
if ($order->status === OrderStatus::Processing) { ... }
```
</stringly_typed>

<catching_exception>
**Anti-Pattern: Catching Exception (Too Broad)**

```php
// BAD - Swallows all errors
try {
    $user = $this->createUser($data);
} catch (\Exception $e) {
    return null;  // What went wrong? Who knows!
}

// GOOD - Catch specific exceptions
try {
    $user = $this->createUser($data);
} catch (DuplicateEmailException $e) {
    return redirect()->back()->withErrors(['email' => 'Email already exists']);
} catch (ValidationException $e) {
    return redirect()->back()->withErrors($e->errors());
}
// Let unexpected exceptions bubble up
```
</catching_exception>

<exposing_ids>
**Anti-Pattern: Exposing Sequential IDs**

```php
// BAD - Enumerable IDs
/api/users/1
/api/users/2
/api/users/3  // Easy to scrape all users

// GOOD - UUIDs or hashids
/api/users/550e8400-e29b-41d4-a716-446655440000
/api/users/jR3Dw5  // Hashid

// Implementation
class User extends Model
{
    use HasUuids;  // Laravel 9+
}
```
</exposing_ids>

<no_validation>
**Anti-Pattern: No Input Validation**

```php
// BAD - Trusting client data
$order = Order::create($request->all());

// GOOD - Validate everything
$validated = $request->validate([
    'product_id' => 'required|exists:products,id',
    'quantity' => 'required|integer|min:1|max:100',
    'shipping_address' => 'required|string|max:500',
]);
$order = Order::create($validated);
```
</no_validation>

<hardcoded_config>
**Anti-Pattern: Hardcoded Configuration**

```php
// BAD - Hardcoded values
$client = new StripeClient('sk_live_xxxxx');
$apiUrl = 'https://api.production.com';
$timeout = 30;

// GOOD - Configuration files/env
$client = new StripeClient(config('services.stripe.secret'));
$apiUrl = config('services.api.url');
$timeout = config('services.api.timeout', 30);
```
</hardcoded_config>

<synchronous_external>
**Anti-Pattern: Synchronous External Calls**

```php
// BAD - User waits for all external calls
public function store(Request $request)
{
    $order = Order::create($request->validated());
    $this->stripeClient->charge($order->total);  // 500ms
    $this->emailClient->send(new OrderConfirmation($order));  // 300ms
    $this->slackClient->notify("New order #{$order->id}");  // 200ms
    return response()->json($order);  // 1+ second response
}

// GOOD - Queue external calls
public function store(Request $request)
{
    $order = Order::create($request->validated());
    ProcessOrderPayment::dispatch($order);
    SendOrderConfirmation::dispatch($order);
    NotifySlack::dispatch("New order #{$order->id}");
    return response()->json($order);  // < 100ms response
}
```
</synchronous_external>

<no_transactions>
**Anti-Pattern: No Transactions for Related Writes**

```php
// BAD - Partial failure leaves inconsistent data
$order = Order::create($data);
$order->items()->createMany($items);
$payment = Payment::create([...]);  // Fails! Order exists without payment

// GOOD - Wrap in transaction
DB::transaction(function () use ($data, $items) {
    $order = Order::create($data);
    $order->items()->createMany($items);
    Payment::create([...]);
});
// Either all succeed or all fail
```
</no_transactions>

<logging_sensitive>
**Anti-Pattern: Logging Sensitive Data**

```php
// BAD - Logging secrets
Log::info('Payment processed', [
    'card_number' => $request->card_number,
    'cvv' => $request->cvv,
    'password' => $user->password,
]);

// GOOD - Redact sensitive fields
Log::info('Payment processed', [
    'order_id' => $order->id,
    'amount' => $order->total,
    'card_last_four' => substr($request->card_number, -4),
]);
```
</logging_sensitive>

<ignoring_errors>
**Anti-Pattern: Ignoring Return Values**

```php
// BAD - Ignoring failure
$file->move($destination);
// Did it work? Who knows!

// GOOD - Handle failure
if (!$file->move($destination)) {
    throw new FileUploadException('Failed to move uploaded file');
}

// Or use exceptions
try {
    Storage::disk('s3')->put($path, $contents);
} catch (S3Exception $e) {
    Log::error('S3 upload failed', ['error' => $e->getMessage()]);
    throw new UploadFailedException('Could not save file', previous: $e);
}
```
</ignoring_errors>

<summary>
**Quick Reference:**

| Anti-Pattern | Fix |
|--------------|-----|
| N+1 queries | Eager load with `with()` |
| Fat controllers | Extract to services |
| God models | Single responsibility |
| Premature optimization | Measure first |
| Stringly typed | Use enums/constants |
| Catch all exceptions | Catch specific types |
| Sequential IDs | Use UUIDs |
| No validation | Validate all input |
| Hardcoded config | Use env/config files |
| Sync external calls | Use queues |
| No transactions | Wrap related writes |
| Logging secrets | Redact sensitive data |
| Ignoring errors | Handle failures |
</summary>
