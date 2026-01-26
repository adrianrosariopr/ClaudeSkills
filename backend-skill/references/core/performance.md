<overview>
Backend performance optimization covering profiling, database optimization, caching strategies, and scaling patterns. Measure before optimizing - premature optimization is the root of all evil.
</overview>

<profiling>
**Profiling Tools:**

```bash
# PHP
# Xdebug + Webgrind
xdebug.mode=profile
xdebug.output_dir=/tmp/profiles

# Blackfire (production-safe)
blackfire run php artisan my:command

# Node.js
node --prof app.js
node --prof-process isolate-*.log

# Python
python -m cProfile -o output.prof script.py
# Visualize with snakeviz

# Go
import _ "net/http/pprof"
go tool pprof http://localhost:6060/debug/pprof/profile
```

**Key Metrics:**

```
- Response time (p50, p95, p99)
- Requests per second (RPS)
- Error rate
- Memory usage
- CPU usage
- Database query time
- External service latency
```
</profiling>

<database_optimization>
**Query Optimization:**

```sql
-- Use EXPLAIN ANALYZE
EXPLAIN ANALYZE SELECT * FROM posts WHERE user_id = 123;

-- Look for:
-- - Seq Scan on large tables (needs index)
-- - High "Rows Removed by Filter"
-- - Nested Loop with many iterations
-- - Sort operations on large sets

-- Add indexes for common queries
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);

-- Composite indexes for multi-column queries
CREATE INDEX idx_posts_user_status ON posts(user_id, status);

-- Partial indexes for filtered queries
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
```

**N+1 Prevention:**

```php
// BAD - N+1 queries
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name;  // Query per post
}

// GOOD - Eager loading
$posts = Post::with('author')->get();
foreach ($posts as $post) {
    echo $post->author->name;  // No additional queries
}

// Nested eager loading
$posts = Post::with([
    'author',
    'comments.user',
    'tags',
])->get();

// Conditional eager loading
$posts = Post::when($includeAuthor, function ($query) {
    $query->with('author');
})->get();
```
</database_optimization>

<connection_pooling>
**Connection Management:**

```php
// Laravel - persistent connections
'mysql' => [
    'options' => [
        PDO::ATTR_PERSISTENT => true,
    ],
],

// Connection pool sizing
// Rule of thumb: pool_size = (2 * CPU cores) + disk spindles
// For SSDs: pool_size = (2 * CPU cores) + 1

// Monitor connections
SHOW STATUS LIKE 'Threads_connected';
SHOW PROCESSLIST;
```

**Database Configuration:**

```sql
-- PostgreSQL
shared_buffers = 256MB
work_mem = 64MB
effective_cache_size = 1GB
max_connections = 100

-- MySQL
innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M
max_connections = 150
```
</connection_pooling>

<caching_strategy>
**Multi-Level Caching:**

```php
// Level 1: In-memory (request-scoped)
private static array $cache = [];

public function getUser(int $id): User
{
    return self::$cache[$id] ??= $this->fetchUser($id);
}

// Level 2: Redis (shared across requests)
public function getUser(int $id): User
{
    return Cache::remember("user:$id", 3600, fn() => User::find($id));
}

// Level 3: Database query cache
$users = User::where('active', true)->cacheFor(60)->get();

// Cache computed values
$stats = Cache::remember('dashboard:stats', 300, function () {
    return [
        'users' => User::count(),
        'orders' => Order::whereDate('created_at', today())->count(),
        'revenue' => Order::whereDate('created_at', today())->sum('total'),
    ];
});
```

**Cache Warming:**

```php
// Warm cache on deploy
Artisan::command('cache:warm', function () {
    // Pre-compute expensive queries
    $this->call('view:cache');
    $this->call('config:cache');
    $this->call('route:cache');

    // Warm application caches
    Cache::put('popular_products', Product::popular()->take(100)->get());
});
```
</caching_strategy>

<async_processing>
**Background Processing:**

```php
// Move slow operations to queues
class CreateOrderController
{
    public function __invoke(Request $request): JsonResponse
    {
        $order = Order::create($request->validated());

        // Fast - just dispatch, don't wait
        ProcessOrderJob::dispatch($order);
        SendOrderConfirmation::dispatch($order);
        UpdateInventory::dispatch($order);

        return response()->json($order, 201);
    }
}

// Chunk large operations
User::chunk(1000, function ($users) {
    foreach ($users as $user) {
        SendNewsletter::dispatch($user);
    }
});

// Parallel processing
$results = Http::pool(fn ($pool) => [
    $pool->get('https://api.service1.com/data'),
    $pool->get('https://api.service2.com/data'),
    $pool->get('https://api.service3.com/data'),
]);
```
</async_processing>

<pagination>
**Efficient Pagination:**

```php
// Standard pagination (OK for small tables)
$posts = Post::paginate(20);

// Cursor pagination (better for large tables)
$posts = Post::orderBy('id')->cursorPaginate(20);

// Avoid COUNT(*) on large tables
$posts = Post::simplePaginate(20);  // No total count

// Deferred joins for complex queries
SELECT p.* FROM posts p
INNER JOIN (
    SELECT id FROM posts
    WHERE status = 'published'
    ORDER BY created_at DESC
    LIMIT 20 OFFSET 100
) AS tmp ON p.id = tmp.id;
```
</pagination>

<response_optimization>
**Response Optimization:**

```php
// Return only needed fields
$users = User::select(['id', 'name', 'email'])->get();

// API resources with sparse fieldsets
$posts = PostResource::collection($posts)->only(['id', 'title']);

// Compression
// Enable gzip in nginx/Apache

// Streaming large responses
return response()->stream(function () {
    foreach (Post::cursor() as $post) {
        echo json_encode($post) . "\n";
        ob_flush();
        flush();
    }
}, 200, ['Content-Type' => 'application/x-ndjson']);
```
</response_optimization>

<scaling_patterns>
**Horizontal Scaling:**

```
Load Balancer (nginx, HAProxy)
      |
  +---+---+
  |   |   |
App App App
  |   |   |
  +---+---+
      |
Database (read replicas)
```

**Read Replicas:**

```php
// Laravel read/write splitting
'mysql' => [
    'read' => [
        'host' => ['replica1', 'replica2'],
    ],
    'write' => [
        'host' => 'primary',
    ],
],

// Force primary for critical reads
User::onWriteConnection()->find($id);
```

**Sharding:**

```php
// Shard by user ID
$shardId = $userId % 4;
$connection = "shard_{$shardId}";
DB::connection($connection)->table('orders')->where('user_id', $userId);
```
</scaling_patterns>

<monitoring>
**Performance Monitoring:**

```php
// Application Performance Monitoring (APM)
// - New Relic
// - Datadog
// - Sentry Performance

// Custom metrics
Metrics::timing('api.orders.create', $duration);
Metrics::increment('api.orders.count');
Metrics::gauge('queue.depth', Queue::size());

// Slow query logging
DB::listen(function ($query) {
    if ($query->time > 100) {  // > 100ms
        Log::warning('Slow query', [
            'sql' => $query->sql,
            'time' => $query->time,
        ]);
    }
});
```
</monitoring>

<checklist>
**Performance Checklist:**

- [ ] Profile before optimizing
- [ ] Enable query logging in development
- [ ] Check for N+1 queries
- [ ] Add indexes for frequently filtered columns
- [ ] Use pagination for lists
- [ ] Cache expensive computations
- [ ] Move slow operations to queues
- [ ] Enable HTTP compression
- [ ] Use connection pooling
- [ ] Monitor response times in production
- [ ] Set up alerting for slow endpoints
</checklist>
