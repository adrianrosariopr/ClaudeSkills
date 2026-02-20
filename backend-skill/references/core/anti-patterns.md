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

<copy_paste_fixes>
**Anti-Pattern: Copy-Pasting the Same Fix Across Multiple Files**

```php
// BAD - Same config fix manually applied to 10 similar service classes:
// EmailImporter.php:  $this->apiKey = config('services.provider.key');
// CsvImporter.php:    $this->apiKey = config('services.provider.key');
// ApiImporter.php:    $this->apiKey = config('services.provider.key');
// ... 7 more files with the identical change

// BAD - Identical error handling block copy-pasted across importers:
} catch (\Exception $e) {
    \Log::warning('Record import skipped', [
        'source' => 'email',    // Only this string changes per file
        'record_id' => $id,
        'error' => $e->getMessage(),
    ]);
    return 'skipped';
}
```

**Why it's bad:**
- Future changes require touching all files again
- Easy to miss a file, creating inconsistency
- Burns time on mechanical repetition
- Identical code in N files means N chances to introduce a typo

**Instead:**
```php
// GOOD - Extract to base class FIRST, then all children inherit
abstract class BaseImporter
{
    abstract protected function getSourceName(): string;

    protected function getApiKey(): string
    {
        return config('services.provider.key');
    }

    protected function logSkippedRecord(\Exception $e, string $id): void
    {
        \Log::warning('Record import skipped', [
            'source' => $this->getSourceName(),
            'record_id' => $id,
            'error' => $e->getMessage(),
        ]);
    }
}

// Each child just overrides what's unique:
class EmailImporter extends BaseImporter
{
    protected function getSourceName(): string { return 'email'; }
    // Inherits getApiKey(), logSkippedRecord()
}
```

**Rule:** When you see the same code in 3+ files, extract it to a shared location (base class, trait, or helper) BEFORE applying the fix. Fix once, apply everywhere.
</copy_paste_fixes>

<megacommit>
**Anti-Pattern: Megacommits Mixing Multiple Concerns**

```bash
# BAD - One massive commit with 5+ different types of changes:
git commit -m "Major cleanup: formatting + dead code + security + config changes"
# Contains: Prettier reformatting + SSL removal + URL centralization +
# dead code deletion + feature fixes + middleware additions
```

**Why it's bad:**
- Impossible to review (hundreds of files, tens of thousands of lines)
- Can't revert one change without reverting everything
- Git blame becomes useless for every touched file
- Hides bugs — real logic changes buried under formatting noise

**Instead:**
```bash
# GOOD - Atomic commits, one concern each:

# 1. Formatting (mechanical, tool-driven)
php artisan pint && npm run format
git add . && git commit -m "style: Apply code formatting"

# 2. Configuration centralization (search-replace)
git commit -m "refactor: Replace env() calls with config()"

# 3. Security fix
git commit -m "security: Enable SSL verification for HTTP calls"

# 4. Dead code removal
git commit -m "cleanup: Remove unused controllers and services"

# 5. Feature/logic changes
git commit -m "fix: Expand search to include all supported types"
```

**Rule:** Each commit should have ONE type of change. If you're writing "and" in the commit message, it should probably be multiple commits. Formatting, security, refactoring, features, and dead code removal are separate concerns.
</megacommit>

<mechanical_search_replace>
**Anti-Pattern: Manual Editing for Mechanical Changes**

```php
// BAD - Manually opening 15+ files to make the same string replacement:
// File 1: env('API_KEY') → config('services.api.key')
// File 2: env('API_KEY') → config('services.api.key')
// ... 13 more files

// BAD - Manually removing the same option from every HTTP call:
// File 1: Remove Http::withOptions(['verify' => false])->
// File 2: Remove Http::withOptions(['verify' => false])->
// ... 10 more files
```

**Why it's bad:**
- Slow, error-prone, wastes time
- Easy to miss occurrences
- Each manual edit is a chance to introduce a typo or inconsistency

**Instead:**
```bash
# GOOD - Use search-replace tools for mechanical changes

# 1. Find all occurrences first
grep -rn "env('API_KEY')" app/ --include="*.php"

# 2. Replace with sed or IDE "Find and Replace in Files"
sed -i '' "s|env('API_KEY')|config('services.api.key')|g" app/**/*.php

# 3. Verify with git diff
git diff --stat
```

**Rule:** If the same string appears in 3+ files and needs the same replacement, use search-replace tooling. Save manual editing for changes that require human judgment.
</mechanical_search_replace>

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
| Copy-paste fixes | Extract to base class/trait first |
| Megacommits | One concern per commit |
| Manual search-replace | Use tooling for mechanical changes |
</summary>
