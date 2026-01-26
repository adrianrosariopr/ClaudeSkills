# Workflow: Optimize Performance

<required_reading>
**Read these reference files NOW:**
1. references/core/performance.md
2. references/core/caching.md
3. references/core/queues.md
4. Framework-specific patterns file for the user's framework
</required_reading>

<context>
Systematic approach to identifying and resolving backend performance issues. Measure before optimizing - premature optimization is counterproductive.
</context>

<process>

<step name="measure-first">
Never optimize without measuring:

**Identify the slow parts:**
- Which endpoints are slow?
- What's the p50, p95, p99 response time?
- Where is time being spent? (DB, external API, computation)

**Tools:**
```bash
# Application Performance Monitoring
- New Relic
- Datadog
- Sentry Performance

# Laravel Telescope
php artisan telescope:install

# Query logging
DB::enableQueryLog();
// ... do stuff
dd(DB::getQueryLog());
```
</step>

<step name="profile-database">
Database is usually the bottleneck:

**Enable query logging:**
```php
// Laravel
DB::listen(function ($query) {
    if ($query->time > 100) {  // > 100ms
        Log::warning('Slow query', [
            'sql' => $query->sql,
            'bindings' => $query->bindings,
            'time' => $query->time,
        ]);
    }
});
```

**Analyze slow queries:**
```sql
EXPLAIN ANALYZE SELECT * FROM posts WHERE user_id = 1 ORDER BY created_at DESC;

-- Look for:
-- Seq Scan on large tables → needs index
-- Sort → consider index
-- Nested Loop → might need different query structure
```

**Fix N+1 queries:**
```php
// Before (N+1)
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name;  // Query per post!
}

// After (eager loading)
$posts = Post::with('author')->get();
foreach ($posts as $post) {
    echo $post->author->name;  // No additional queries
}
```

**Add missing indexes:**
```sql
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
```
</step>

<step name="implement-caching">
Cache expensive operations:

**Query caching:**
```php
// Laravel
$posts = Cache::remember('posts:published', 3600, function () {
    return Post::where('published', true)
        ->with('author')
        ->orderBy('created_at', 'desc')
        ->take(20)
        ->get();
});

// Invalidate on update
Post::updated(function ($post) {
    Cache::forget('posts:published');
});
```

**Redis for sessions:**
```php
// config/session.php
'driver' => 'redis',
```

**HTTP caching:**
```php
return response()->json($data)
    ->header('Cache-Control', 'public, max-age=60');
```
</step>

<step name="use-queues">
Move slow operations to background:

**Identify queue candidates:**
- Sending emails
- Processing uploads
- External API calls
- Report generation
- Data imports/exports

**Implement:**
```php
// Before (blocking)
public function store(Request $request)
{
    $order = Order::create($request->validated());
    Mail::to($user)->send(new OrderConfirmation($order));  // Slow!
    $this->stripe->charge($order->total);  // Slow!
    return response()->json($order, 201);
}

// After (async)
public function store(Request $request)
{
    $order = Order::create($request->validated());
    ProcessOrderJob::dispatch($order);  // Fast!
    return response()->json($order, 201);
}
```
</step>

<step name="optimize-queries">
Query optimization techniques:

**Select only needed columns:**
```php
// Before
$users = User::all();

// After
$users = User::select(['id', 'name', 'email'])->get();
```

**Efficient pagination:**
```php
// Standard (includes COUNT)
$posts = Post::paginate(20);

// Faster for large tables (no COUNT)
$posts = Post::cursorPaginate(20);
```

**Batch operations:**
```php
// Before (slow)
foreach ($users as $user) {
    $user->update(['status' => 'active']);
}

// After (fast)
User::whereIn('id', $userIds)->update(['status' => 'active']);
```

**Chunking large datasets:**
```php
User::chunk(1000, function ($users) {
    foreach ($users as $user) {
        ProcessUser::dispatch($user);
    }
});
```
</step>

<step name="optimize-external-calls">
Optimize external service calls:

**Parallel requests:**
```php
// Laravel HTTP concurrent
$responses = Http::pool(fn ($pool) => [
    $pool->get('https://api.service1.com/data'),
    $pool->get('https://api.service2.com/data'),
    $pool->get('https://api.service3.com/data'),
]);
```

**Connection pooling:**
```php
// Reuse HTTP client
$client = Http::baseUrl('https://api.example.com')
    ->timeout(30)
    ->retry(3, 100);

$response1 = $client->get('/endpoint1');
$response2 = $client->get('/endpoint2');
```

**Cache external responses:**
```php
$data = Cache::remember('external:api:data', 300, function () {
    return Http::get('https://api.example.com/data')->json();
});
```
</step>

<step name="optimize-response">
Optimize response size:

**Compress responses:**
```nginx
# nginx
gzip on;
gzip_types application/json text/plain application/javascript;
```

**Sparse fieldsets:**
```php
// Allow clients to request specific fields
// GET /posts?fields=id,title,author

$fields = $request->input('fields', '*');
$posts = Post::select(explode(',', $fields))->get();
```

**Streaming for large responses:**
```php
return response()->stream(function () {
    foreach (Post::cursor() as $post) {
        echo json_encode($post) . "\n";
        flush();
    }
}, 200, ['Content-Type' => 'application/x-ndjson']);
```
</step>

<step name="verify-improvements">
Measure the improvement:

```bash
# Before/after response times
ab -n 100 -c 10 http://localhost:8000/api/posts

# Query count before/after
# Should see reduction in queries

# Response time in logs
# Should see improvement

# Run load test
k6 run load-test.js
```

Document the changes and results.
</step>

</process>

<optimization_checklist>
**Quick Wins:**
- [ ] Enable query logging, find N+1s
- [ ] Add missing indexes on filtered/sorted columns
- [ ] Eager load relationships
- [ ] Cache expensive queries
- [ ] Move email/notifications to queue
- [ ] Enable response compression
- [ ] Use cursor pagination for large tables

**Advanced:**
- [ ] Read replicas for read-heavy loads
- [ ] Redis for sessions and cache
- [ ] CDN for static assets
- [ ] Connection pooling
- [ ] Horizontal scaling
</optimization_checklist>

<anti_patterns>
Avoid:
- Optimizing without measuring
- Caching everything (increases complexity)
- Premature optimization
- Ignoring database indexes
- Synchronous external calls in request cycle
- SELECT * everywhere
</anti_patterns>

<success_criteria>
Optimization successful when:
- Measured improvement in response times
- No increase in error rates
- Tests still passing
- Monitoring shows sustained improvement
- Documented what changed and why
</success_criteria>
