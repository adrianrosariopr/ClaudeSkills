<overview>
Database design patterns, schema design, indexing strategies, migrations, and ORM best practices. The database is often the performance bottleneck - design it carefully.
</overview>

<schema_design>
**Normalization Levels:**

```sql
-- First Normal Form (1NF): No repeating groups
-- BAD
CREATE TABLE orders (
  id INT PRIMARY KEY,
  items VARCHAR(255)  -- "item1,item2,item3"
);

-- GOOD
CREATE TABLE orders (id INT PRIMARY KEY);
CREATE TABLE order_items (
  id INT PRIMARY KEY,
  order_id INT REFERENCES orders(id),
  item_name VARCHAR(255)
);

-- Third Normal Form (3NF): No transitive dependencies
-- BAD
CREATE TABLE orders (
  id INT PRIMARY KEY,
  customer_id INT,
  customer_name VARCHAR(255),  -- Depends on customer_id, not order
  customer_email VARCHAR(255)
);

-- GOOD
CREATE TABLE customers (
  id INT PRIMARY KEY,
  name VARCHAR(255),
  email VARCHAR(255)
);
CREATE TABLE orders (
  id INT PRIMARY KEY,
  customer_id INT REFERENCES customers(id)
);
```

**When to Denormalize:**
- Read-heavy workloads where joins are expensive
- Reporting/analytics tables
- Caching frequently accessed data
- After measuring actual performance issues
</schema_design>

<indexing>
**Index Strategies:**

```sql
-- Primary key (automatic index)
CREATE TABLE users (
  id SERIAL PRIMARY KEY
);

-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (order matters!)
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at);
-- Helps: WHERE user_id = ? AND created_at > ?
-- Helps: WHERE user_id = ?
-- Does NOT help: WHERE created_at > ?

-- Partial index (smaller, faster)
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- Unique index
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);

-- Full-text search
CREATE INDEX idx_posts_content ON posts USING gin(to_tsvector('english', content));
```

**Index Guidelines:**
- Index columns in WHERE clauses
- Index foreign keys
- Index columns used in ORDER BY
- Composite indexes: most selective column first
- Don't over-index (slows writes)
- Use EXPLAIN ANALYZE to verify index usage
</indexing>

<query_optimization>
**Common Performance Issues:**

```sql
-- N+1 Query Problem
-- BAD: 1 query for posts + N queries for authors
posts = Post.all()
for post in posts:
    print(post.author.name)  # Triggers query each time

-- GOOD: Eager load
posts = Post.include(:author).all()

-- Using EXPLAIN
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
-- Look for:
-- - Seq Scan (table scan - often bad)
-- - Index Scan (good)
-- - Rows removed by filter (index might help)

-- Avoid SELECT *
SELECT id, name, email FROM users;  -- Only what you need

-- Use LIMIT
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20;

-- Batch operations
-- BAD
for user in users:
    user.update(status='active')

-- GOOD
UPDATE users SET status = 'active' WHERE id IN (1, 2, 3, ...);
```
</query_optimization>

<migrations>
**Migration Best Practices:**

```sql
-- Always reversible
-- UP
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
-- DOWN
ALTER TABLE users DROP COLUMN phone;

-- Add columns as nullable first, then backfill
ALTER TABLE users ADD COLUMN status VARCHAR(20);  -- nullable
UPDATE users SET status = 'active' WHERE status IS NULL;
ALTER TABLE users ALTER COLUMN status SET NOT NULL;

-- Add indexes concurrently (PostgreSQL)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Never drop columns in heavy traffic
-- 1. Stop writing to column
-- 2. Deploy code that ignores column
-- 3. Drop column in separate migration
```

**Migration Naming:**
```
2024_01_15_120000_create_users_table.php
2024_01_15_130000_add_email_to_users.php
2024_01_16_090000_create_posts_table.php
```
</migrations>

<orm_patterns>
**Repository Pattern:**

```php
// Laravel example
interface UserRepository
{
    public function find(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function save(User $user): void;
    public function delete(User $user): void;
}

class EloquentUserRepository implements UserRepository
{
    public function find(int $id): ?User
    {
        return User::find($id);
    }

    public function findByEmail(string $email): ?User
    {
        return User::where('email', $email)->first();
    }
}
```

**Eager Loading:**

```php
// Laravel
$posts = Post::with(['author', 'comments.user'])->get();

// Django
posts = Post.objects.select_related('author').prefetch_related('comments__user')

// Prisma
const posts = await prisma.post.findMany({
  include: {
    author: true,
    comments: { include: { user: true } }
  }
});
```
</orm_patterns>

<transactions>
**Transaction Patterns:**

```php
// Explicit transaction
DB::beginTransaction();
try {
    $order = Order::create([...]);
    $order->items()->createMany([...]);
    Payment::process($order);
    DB::commit();
} catch (\Exception $e) {
    DB::rollBack();
    throw $e;
}

// Automatic (Laravel)
DB::transaction(function () {
    $order = Order::create([...]);
    $order->items()->createMany([...]);
});

// Isolation levels
DB::transaction(function () {
    // ...
}, 5, 'serializable');  // 5 retries, serializable isolation
```

**When to use transactions:**
- Multiple related writes
- Money/inventory operations
- Anything that must be atomic
</transactions>

<soft_deletes>
**Soft Delete Pattern:**

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255),
  deleted_at TIMESTAMP NULL
);

-- Query active users
SELECT * FROM users WHERE deleted_at IS NULL;

-- Query including deleted
SELECT * FROM users;

-- Restore
UPDATE users SET deleted_at = NULL WHERE id = 1;
```

**When to use:**
- Audit trail requirements
- Undo functionality
- Legal data retention
- Preserve referential integrity
</soft_deletes>

<connection_pooling>
**Connection Pool Configuration:**

```yaml
# PostgreSQL (pgbouncer)
[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20

# Application level
DATABASE_POOL_SIZE=20
DATABASE_POOL_TIMEOUT=30
```

**Guidelines:**
- Pool size = (2 * CPU cores) + disk spindles
- Monitor connection wait times
- Use statement-level pooling for serverless
</connection_pooling>

<decision_tree>
**PostgreSQL vs MySQL:**

PostgreSQL:
- Better for complex queries
- JSONB support
- Full-text search
- Better standards compliance
- CTEs, window functions

MySQL:
- Simpler replication
- Better read performance for simple queries
- Larger ecosystem
- More hosting options

**SQL vs NoSQL:**

Use SQL when:
- Complex relationships
- ACID compliance needed
- Ad-hoc queries
- Structured data

Use NoSQL when:
- Massive scale writes
- Flexible schema
- Document-oriented data
- Caching layer (Redis)
</decision_tree>
