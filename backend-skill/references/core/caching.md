<overview>
Caching strategies for backend applications. Proper caching can dramatically improve performance, but incorrect caching causes subtle bugs. Cache invalidation is one of the hardest problems in computer science.
</overview>

<cache_layers>
**Where to Cache:**

```
Client <-> CDN <-> Load Balancer <-> App Server <-> Cache <-> Database
  |         |                            |           |
Browser   Edge                        Redis/      Query
Cache     Cache                      Memcached    Cache
```

**1. Browser Cache (HTTP caching):**
```
Cache-Control: max-age=3600, public
ETag: "abc123"
Last-Modified: Wed, 15 Jan 2025 12:00:00 GMT
```

**2. CDN/Edge Cache:**
- Static assets (images, CSS, JS)
- API responses (with care)
- Full page caching

**3. Application Cache (Redis/Memcached):**
- Database query results
- Computed values
- Session data
- Rate limiting counters

**4. Database Query Cache:**
- Built-in query caching
- Materialized views
</cache_layers>

<redis_patterns>
**Basic Operations:**

```php
// Laravel
Cache::put('user:123', $user, 3600);  // 1 hour
$user = Cache::get('user:123');
Cache::forget('user:123');

// Check and set
$user = Cache::remember('user:123', 3600, function () {
    return User::find(123);
});

// Tags for group invalidation
Cache::tags(['users', 'profiles'])->put('user:123', $user, 3600);
Cache::tags('users')->flush();  // Invalidate all user caches
```

**Cache Key Patterns:**

```
# Namespace:type:identifier:version
users:profile:123:v1
orders:list:user:123:page:1
api:response:products:hash:abc123
```

**Atomic Operations:**

```php
// Increment/decrement
Cache::increment('views:post:123');
Cache::decrement('stock:product:456');

// Locks (prevent race conditions)
$lock = Cache::lock('process-order:123', 10);
if ($lock->get()) {
    try {
        processOrder();
    } finally {
        $lock->release();
    }
}
```
</redis_patterns>

<cache_strategies>
**Cache-Aside (Lazy Loading):**

```php
function getUser(int $id): User
{
    $cached = Cache::get("user:$id");
    if ($cached) {
        return $cached;
    }

    $user = User::find($id);
    Cache::put("user:$id", $user, 3600);
    return $user;
}
```
- Most common pattern
- Cache only what's accessed
- Risk of cache stampede

**Write-Through:**

```php
function updateUser(int $id, array $data): User
{
    $user = User::find($id);
    $user->update($data);

    Cache::put("user:$id", $user, 3600);  // Update cache immediately
    return $user;
}
```
- Cache always consistent
- Higher write latency

**Write-Behind (Async):**

```php
function updateUser(int $id, array $data): User
{
    Cache::put("user:$id", $data, 3600);  // Update cache first
    dispatch(new PersistUserJob($id, $data));  // Async DB write
    return $data;
}
```
- Fastest writes
- Risk of data loss
- Complex consistency

**Cache Stampede Prevention:**

```php
// Lock while regenerating
function getExpensiveData(): array
{
    $cached = Cache::get('expensive:data');
    if ($cached) {
        return $cached;
    }

    $lock = Cache::lock('expensive:data:lock', 30);
    if ($lock->get()) {
        try {
            $data = computeExpensiveData();
            Cache::put('expensive:data', $data, 3600);
            return $data;
        } finally {
            $lock->release();
        }
    }

    // Wait for other process to fill cache
    sleep(1);
    return Cache::get('expensive:data');
}

// Early expiration (probabilistic)
$ttl = 3600;
$grace = 300;  // 5 minute grace period
$expires = time() + $ttl - rand(0, $grace);
```
</cache_strategies>

<invalidation>
**Cache Invalidation Strategies:**

**Time-Based (TTL):**
```php
Cache::put('data', $value, 3600);  // Expires in 1 hour
```
- Simple
- May serve stale data

**Event-Based:**
```php
// When data changes, invalidate
class User extends Model
{
    protected static function booted()
    {
        static::updated(function ($user) {
            Cache::forget("user:{$user->id}");
            Cache::tags('users')->flush();
        });
    }
}
```

**Version-Based:**
```php
// Increment version instead of deleting
$version = Cache::increment('user:123:version');
$key = "user:123:data:v{$version}";
Cache::put($key, $user, 3600);
```

**Tag-Based:**
```php
Cache::tags(['products', 'category:5'])->put('product:123', $product);
Cache::tags('category:5')->flush();  // All category 5 products invalidated
```
</invalidation>

<http_caching>
**HTTP Cache Headers:**

```php
// Immutable assets (hashed filenames)
Cache-Control: public, max-age=31536000, immutable

// API responses (short cache)
Cache-Control: private, max-age=60
ETag: "abc123"

// No caching
Cache-Control: no-store, no-cache, must-revalidate

// Conditional requests
if ($request->header('If-None-Match') === $etag) {
    return response('', 304);
}
```

**Vary Header:**
```
# Different cache per auth state
Vary: Authorization

# Different cache per accept type
Vary: Accept, Accept-Encoding
```
</http_caching>

<cache_sizing>
**Memory Management:**

```
# Redis maxmemory policies
maxmemory 4gb
maxmemory-policy allkeys-lru  # Evict least recently used

# Common policies:
- noeviction: Error when full
- allkeys-lru: Evict LRU keys
- volatile-lru: Evict LRU keys with TTL
- allkeys-random: Random eviction
```

**Monitoring:**
```bash
# Redis memory stats
redis-cli INFO memory

# Key count by pattern
redis-cli --scan --pattern "user:*" | wc -l
```
</cache_sizing>

<anti_patterns>
**Cache Anti-Patterns:**

- **Caching too much:** Not everything needs caching
- **No TTL:** Cache should eventually expire
- **Cache as primary storage:** Always have source of truth
- **Ignoring cold cache:** System should work without cache
- **Not monitoring hit rate:** Measure effectiveness
- **Serialization overhead:** Large objects may not benefit
</anti_patterns>

<decision_tree>
**Redis vs Memcached:**

Redis:
- Data structures (lists, sets, hashes)
- Persistence
- Pub/sub
- Lua scripting
- Cluster support

Memcached:
- Simple key-value
- Lower memory overhead
- Multi-threaded
- Simpler operations

**When to cache:**
- Expensive computations
- Frequent reads, infrequent writes
- Data that can be stale temporarily
- Database query results
- External API responses
</decision_tree>
