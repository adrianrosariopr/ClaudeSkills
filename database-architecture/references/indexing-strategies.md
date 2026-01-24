<overview>
Database indexes are data structures that improve query performance by allowing the database to find rows without scanning the entire table. Understanding when and how to use different index types is critical for database performance. This reference covers indexing strategies across relational and document databases.
</overview>

<index_fundamentals>
**How Indexes Work**

Without index: Full table scan (O(n))
With index: Index lookup (O(log n) for B-tree)

**Trade-offs:**
- Faster reads, slower writes (index maintenance)
- Additional storage space
- Query planner overhead

**Rule of Thumb:**
Every index must justify its existence through actual query patterns. Measure, don't guess.
</index_fundamentals>

<index_types>
<type name="btree">
**B-Tree Index (Default)**

Balanced tree structure, excellent for most use cases.

**Best for:**
- Equality comparisons: `WHERE id = 123`
- Range queries: `WHERE date BETWEEN '2024-01-01' AND '2024-12-31'`
- Sorting: `ORDER BY created_at`
- Prefix matching: `WHERE name LIKE 'John%'`

**Not suitable for:**
- Suffix matching: `WHERE name LIKE '%son'`
- Low-cardinality columns (few distinct values)

```sql
-- PostgreSQL/MySQL
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_created ON orders(created_at);
```
</type>

<type name="hash">
**Hash Index**

Hash function maps key to location. O(1) lookup.

**Best for:**
- Equality only: `WHERE id = 123`
- High-cardinality columns

**Limitations:**
- No range queries
- No sorting
- No partial matching

```sql
-- PostgreSQL
CREATE INDEX idx_users_email ON users USING HASH (email);
```

**Recommendation:** Stick with B-tree unless you have a proven need for hash indexes. B-tree is more versatile.
</type>

<type name="gin">
**GIN (Generalized Inverted Index)**

PostgreSQL index for composite values.

**Best for:**
- JSONB columns
- Array columns
- Full-text search

```sql
-- JSONB index
CREATE INDEX idx_products_attrs ON products USING GIN (attributes);
SELECT * FROM products WHERE attributes @> '{"color": "red"}';

-- Array index
CREATE INDEX idx_users_tags ON users USING GIN (tags);
SELECT * FROM users WHERE tags @> ARRAY['admin'];

-- Full-text search
CREATE INDEX idx_articles_search ON articles USING GIN (to_tsvector('english', title || ' ' || body));
SELECT * FROM articles WHERE to_tsvector('english', title || ' ' || body) @@ to_tsquery('database & performance');
```
</type>

<type name="gist">
**GiST (Generalized Search Tree)**

PostgreSQL index for geometric and range types.

**Best for:**
- Geospatial queries (PostGIS)
- Range types
- Nearest-neighbor searches

```sql
-- PostGIS
CREATE INDEX idx_locations ON stores USING GiST (location);
SELECT * FROM stores WHERE ST_DWithin(location, ST_MakePoint(-73.9, 40.7), 1000);

-- Range types
CREATE INDEX idx_events_during ON events USING GiST (during);
SELECT * FROM events WHERE during && '[2024-01-01, 2024-01-31]'::daterange;
```
</type>

<type name="brin">
**BRIN (Block Range Index)**

PostgreSQL index for large, naturally ordered data.

**Best for:**
- Time-series data
- Sequential/append-only tables
- Very large tables

**Advantages:**
- Much smaller than B-tree
- Low maintenance overhead

```sql
-- Time-series data
CREATE INDEX idx_events_time ON events USING BRIN (created_at);

-- Works best when data is physically ordered by the indexed column
```
</type>

<type name="bitmap">
**Bitmap Index**

Available in some databases (Oracle, PostgreSQL uses internally).

**Best for:**
- Low-cardinality columns
- Data warehouse queries
- Complex AND/OR conditions
</type>
</index_types>

<composite_indexes>
**Composite (Multi-Column) Indexes**

Index on multiple columns in specific order.

```sql
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

**Column Order Matters:**

Index `(a, b, c)` can be used for:
- `WHERE a = ?`
- `WHERE a = ? AND b = ?`
- `WHERE a = ? AND b = ? AND c = ?`
- `WHERE a = ? ORDER BY b`

Cannot efficiently use:
- `WHERE b = ?` (first column not used)
- `WHERE a = ? AND c = ?` (gap in columns)

**ESR Rule for Composite Indexes:**

Order columns as: **E**quality → **S**ort → **R**ange

```sql
-- Query: WHERE user_id = ? AND status IN (...) ORDER BY created_at
-- Index: user_id (equality), status (range/IN), created_at (sort)
CREATE INDEX idx_orders_composite ON orders(user_id, created_at, status);
```
</composite_indexes>

<covering_indexes>
**Covering Indexes**

Include all columns needed by query to avoid table lookup.

```sql
-- PostgreSQL: INCLUDE clause
CREATE INDEX idx_orders_covering ON orders(user_id) INCLUDE (status, total);

-- Query can be satisfied from index alone
SELECT status, total FROM orders WHERE user_id = 123;

-- MySQL: Include columns in index
CREATE INDEX idx_orders_covering ON orders(user_id, status, total);
```

**Benefits:**
- Avoids "index scan + table lookup"
- Significant performance improvement for read-heavy queries
</covering_indexes>

<partial_indexes>
**Partial (Filtered) Indexes**

Index only rows matching a condition.

```sql
-- PostgreSQL
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';
CREATE INDEX idx_pending_orders ON orders(created_at) WHERE status = 'pending';

-- Smaller index, faster to maintain
-- Query must match the WHERE clause to use the index
SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at;
```

**Use cases:**
- Hot/cold data (only index hot data)
- Status-based filtering
- Soft deletes (only index non-deleted)
</partial_indexes>

<document_db_indexes>
**Document Database Indexes**

<mongodb>
**MongoDB**

```javascript
// Single field
db.users.createIndex({ email: 1 })

// Compound (order matters)
db.orders.createIndex({ userId: 1, createdAt: -1 })

// Unique
db.users.createIndex({ email: 1 }, { unique: true })

// Sparse (only docs with field)
db.users.createIndex({ nickname: 1 }, { sparse: true })

// TTL (auto-delete)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })

// Text (full-text search)
db.articles.createIndex({ title: "text", body: "text" })

// Partial
db.orders.createIndex(
  { createdAt: 1 },
  { partialFilterExpression: { status: "active" } }
)

// Wildcard (dynamic fields)
db.products.createIndex({ "attributes.$**": 1 })

// Check if query uses index
db.collection.find({...}).explain("executionStats")
// Look for: IXSCAN (good) vs COLLSCAN (bad)
```
</mongodb>

<couchbase>
**Couchbase (N1QL)**

```sql
-- Secondary index
CREATE INDEX idx_users_email ON `users`(email);

-- Composite index
CREATE INDEX idx_orders ON `orders`(userId, status, createdAt DESC);

-- Partial index
CREATE INDEX idx_active ON `users`(email) WHERE status = 'active';

-- Array index
CREATE INDEX idx_tags ON `users`(DISTINCT ARRAY t FOR t IN tags END);

-- Check query plan
EXPLAIN SELECT * FROM `users` WHERE email = 'test@example.com';
```
</couchbase>
</document_db_indexes>

<best_practices>
**Indexing Best Practices**

<practice name="analyze">
**Always Analyze First**

```sql
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS) SELECT ...

-- MySQL
EXPLAIN ANALYZE SELECT ...

-- Look for:
-- Seq Scan / Full Table Scan → Need index
-- Index Scan → Good
-- Index Only Scan → Best (covering index)
```
</practice>

<practice name="foreign-keys">
**Index Foreign Keys**

Foreign keys are NOT automatically indexed (PostgreSQL, MySQL InnoDB).

```sql
-- Always index foreign keys
CREATE INDEX idx_orders_user_id ON orders(user_id);
```
</practice>

<practice name="avoid-over-indexing">
**Avoid Over-Indexing**

Each index:
- Slows down INSERT/UPDATE/DELETE
- Uses storage space
- Adds query planner overhead

Remove unused indexes:

```sql
-- PostgreSQL: Find unused indexes
SELECT indexrelname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0 AND indexrelname NOT LIKE '%_pkey';
```
</practice>

<practice name="index-selectivity">
**Consider Selectivity**

High selectivity = good for indexing
Low selectivity = often not worth indexing

```sql
-- High selectivity (good): email, user_id
-- Low selectivity (often bad): status, is_active, gender

-- Exception: Partial index on low-selectivity column
CREATE INDEX idx_pending ON orders(created_at) WHERE status = 'pending';
```
</practice>

<practice name="maintenance">
**Index Maintenance**

```sql
-- PostgreSQL: Rebuild bloated indexes
REINDEX INDEX idx_name;

-- PostgreSQL: Concurrent rebuild (no lock)
REINDEX INDEX CONCURRENTLY idx_name;

-- MySQL: Optimize table
OPTIMIZE TABLE table_name;

-- Update statistics
ANALYZE table_name;  -- PostgreSQL
ANALYZE TABLE table_name;  -- MySQL
```
</practice>
</best_practices>

<common_mistakes>
**Common Indexing Mistakes**

1. **Indexing low-cardinality columns** - Few distinct values, full scan often faster
2. **Too many indexes** - Slows writes, wastes space
3. **Wrong column order in composite** - Leading column must be in query
4. **Functions on indexed columns** - `WHERE YEAR(date) = 2024` can't use index
5. **Missing foreign key indexes** - JOINs become slow
6. **Not analyzing query plans** - Guessing instead of measuring
</common_mistakes>
