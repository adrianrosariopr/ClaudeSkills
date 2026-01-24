<overview>
Query optimization is the process of improving query performance through better query structure, proper indexing, and understanding execution plans. This reference covers practical techniques for optimizing SQL and NoSQL queries.
</overview>

<analyze_first>
**Always Analyze Before Optimizing**

Never optimize based on assumptions. Use EXPLAIN to understand actual execution.

<explain_postgresql>
**PostgreSQL EXPLAIN**

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders
WHERE user_id = 123 AND status = 'pending'
ORDER BY created_at DESC
LIMIT 10;

-- Output interpretation:
-- Seq Scan: Full table scan (usually bad for large tables)
-- Index Scan: Using index to find rows (good)
-- Index Only Scan: Data from index only (best)
-- Bitmap Heap Scan: Index scan + heap fetch (medium)
-- Sort: Sorting in memory/disk (consider index for ORDER BY)
-- Hash Join / Merge Join / Nested Loop: Different join strategies

-- Key metrics:
-- cost: Estimated cost (first number = startup, second = total)
-- rows: Estimated row count
-- actual time: Real execution time (with ANALYZE)
-- loops: Number of times operation executed
```
</explain_postgresql>

<explain_mysql>
**MySQL EXPLAIN**

```sql
EXPLAIN FORMAT=JSON SELECT ...
-- or
EXPLAIN ANALYZE SELECT ...

-- Key columns:
-- type: Access type (best to worst)
--   system > const > eq_ref > ref > range > index > ALL
-- possible_keys: Indexes that could be used
-- key: Index actually used
-- rows: Estimated rows examined
-- Extra: Additional info (Using filesort, Using temporary = bad)
```
</explain_mysql>

<explain_mongodb>
**MongoDB explain()**

```javascript
db.orders.find({ userId: "123", status: "pending" })
  .sort({ createdAt: -1 })
  .limit(10)
  .explain("executionStats")

// Key fields:
// winningPlan.stage: IXSCAN (good) vs COLLSCAN (bad)
// executionStats.totalDocsExamined: Should be close to nReturned
// executionStats.executionTimeMillis: Actual time
```
</explain_mongodb>
</analyze_first>

<query_writing>
**Query Writing Best Practices**

<technique name="sargable">
**Write Sargable Queries**

Sargable = Search ARGument ABLE (can use index)

```sql
-- NOT sargable (function on column)
SELECT * FROM users WHERE YEAR(created_at) = 2024;
SELECT * FROM users WHERE LOWER(email) = 'test@example.com';
SELECT * FROM orders WHERE amount + tax > 100;

-- Sargable (rewritten)
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
SELECT * FROM users WHERE email = 'test@example.com';  -- store lowercase
SELECT * FROM orders WHERE amount > 100 - tax;  -- if tax is constant
```
</technique>

<technique name="select-columns">
**Select Only Needed Columns**

```sql
-- Bad: Fetches all columns
SELECT * FROM orders WHERE user_id = 123;

-- Good: Only needed columns
SELECT id, status, total FROM orders WHERE user_id = 123;

-- Benefits:
-- Less data transferred
-- Can use covering index
-- Less memory usage
```
</technique>

<technique name="limit">
**Use LIMIT for Large Result Sets**

```sql
-- Bad: Returns all rows
SELECT * FROM logs WHERE level = 'error';

-- Good: Paginated
SELECT * FROM logs WHERE level = 'error' ORDER BY created_at DESC LIMIT 100;

-- For pagination, prefer keyset over OFFSET
-- Bad (slow for large offsets):
SELECT * FROM orders ORDER BY id LIMIT 10 OFFSET 10000;

-- Good (keyset pagination):
SELECT * FROM orders WHERE id > 10000 ORDER BY id LIMIT 10;
```
</technique>

<technique name="avoid-or">
**Optimize OR Conditions**

```sql
-- OR can prevent index usage
SELECT * FROM users WHERE email = 'a@b.com' OR phone = '123';

-- Better: UNION (each can use its index)
SELECT * FROM users WHERE email = 'a@b.com'
UNION
SELECT * FROM users WHERE phone = '123';

-- Or use IN for same column
SELECT * FROM users WHERE status IN ('active', 'pending');
```
</technique>

<technique name="exists-vs-in">
**EXISTS vs IN**

```sql
-- IN: Good for small lists, subquery executed once
SELECT * FROM orders WHERE user_id IN (SELECT id FROM users WHERE status = 'vip');

-- EXISTS: Good for correlated, stops at first match
SELECT * FROM orders o WHERE EXISTS (
  SELECT 1 FROM users u WHERE u.id = o.user_id AND u.status = 'vip'
);

-- General guidance:
-- Small inner result → IN
-- Large outer, small inner correlation → EXISTS
-- Test both with EXPLAIN
```
</technique>
</query_writing>

<join_optimization>
**JOIN Optimization**

<technique name="join-order">
**Join Order Matters**

```sql
-- Start with the most restrictive table
-- (smallest result set after WHERE)

-- If orders has 1M rows, users has 10K rows
-- And we're filtering users to 100 rows
SELECT o.* FROM users u
JOIN orders o ON o.user_id = u.id
WHERE u.status = 'vip';  -- Reduces users first
```
</technique>

<technique name="index-join-columns">
**Index Join Columns**

```sql
-- Always index the column being joined TO
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Query
SELECT * FROM users u JOIN orders o ON o.user_id = u.id;
-- Index on orders.user_id is used
```
</technique>

<technique name="avoid-cartesian">
**Avoid Cartesian Products**

```sql
-- Bad: Missing join condition
SELECT * FROM users, orders;  -- Returns users × orders rows

-- Good: Explicit join
SELECT * FROM users u JOIN orders o ON o.user_id = u.id;
```
</technique>

<technique name="join-vs-subquery">
**JOIN vs Subquery**

```sql
-- Subquery (often slower, depends on optimizer)
SELECT * FROM orders WHERE user_id IN (
  SELECT id FROM users WHERE status = 'active'
);

-- JOIN (often more efficient)
SELECT o.* FROM orders o
JOIN users u ON o.user_id = u.id
WHERE u.status = 'active';

-- Always test both with EXPLAIN
```
</technique>
</join_optimization>

<aggregation>
**Aggregation Optimization**

<technique name="filter-before-aggregate">
**Filter Before Aggregating**

```sql
-- Bad: Aggregates all, then filters
SELECT user_id, SUM(amount) FROM orders GROUP BY user_id HAVING SUM(amount) > 1000;

-- Better: Filter early when possible
SELECT user_id, SUM(amount)
FROM orders
WHERE status = 'completed'  -- Filter before GROUP BY
GROUP BY user_id
HAVING SUM(amount) > 1000;
```
</technique>

<technique name="index-group-by">
**Index for GROUP BY**

```sql
-- Index supports both WHERE and GROUP BY
CREATE INDEX idx_orders_status_user ON orders(status, user_id);

SELECT user_id, COUNT(*) FROM orders WHERE status = 'completed' GROUP BY user_id;
```
</technique>

<technique name="approximate-counts">
**Use Approximate Counts**

```sql
-- Exact count (slow for large tables)
SELECT COUNT(*) FROM orders;

-- PostgreSQL: Approximate count (fast)
SELECT reltuples::bigint FROM pg_class WHERE relname = 'orders';

-- HyperLogLog for distinct counts (if extension available)
```
</technique>
</aggregation>

<mongodb_optimization>
**MongoDB Query Optimization**

<technique name="projection">
**Use Projection**

```javascript
// Bad: Returns all fields
db.users.find({ status: "active" })

// Good: Only needed fields
db.users.find({ status: "active" }, { name: 1, email: 1 })
```
</technique>

<technique name="aggregation-pipeline">
**Optimize Aggregation Pipeline**

```javascript
// Put $match early (uses indexes, reduces documents)
db.orders.aggregate([
  { $match: { status: "completed", createdAt: { $gte: ISODate("2024-01-01") } } },
  { $group: { _id: "$userId", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } },
  { $limit: 10 }
])

// Use $project to reduce document size early
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $project: { userId: 1, amount: 1 } },  // Reduce size before grouping
  { $group: { _id: "$userId", total: { $sum: "$amount" } } }
])
```
</technique>

<technique name="covered-queries">
**Use Covered Queries**

```javascript
// Index covers the query (no document fetch)
db.users.createIndex({ email: 1, name: 1 })

// Covered: All fields in index, _id excluded
db.users.find(
  { email: "test@example.com" },
  { email: 1, name: 1, _id: 0 }
)
```
</technique>
</mongodb_optimization>

<common_issues>
**Common Performance Issues**

<issue name="n-plus-one">
**N+1 Query Problem**

```javascript
// Bad: N+1 queries
const users = await db.users.find({});
for (const user of users) {
  user.orders = await db.orders.find({ userId: user._id });  // N queries
}

// Good: Single query with join/lookup
const users = await db.users.aggregate([
  { $lookup: { from: "orders", localField: "_id", foreignField: "userId", as: "orders" } }
]);

// Or batch query
const userIds = users.map(u => u._id);
const orders = await db.orders.find({ userId: { $in: userIds } });
```
</issue>

<issue name="missing-index">
**Missing Index Detection**

```sql
-- PostgreSQL: Log slow queries
-- In postgresql.conf:
-- log_min_duration_statement = 1000  -- Log queries > 1 second

-- Find queries with seq scans
SELECT query, calls, mean_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC LIMIT 20;
```
</issue>

<issue name="stale-stats">
**Stale Statistics**

```sql
-- PostgreSQL
ANALYZE table_name;
VACUUM ANALYZE table_name;

-- Check last analyze
SELECT relname, last_analyze, last_autoanalyze
FROM pg_stat_user_tables;

-- MySQL
ANALYZE TABLE table_name;
```
</issue>
</common_issues>

<optimization_checklist>
**Query Optimization Checklist**

- [ ] Run EXPLAIN ANALYZE on slow queries
- [ ] Check for Seq Scan / COLLSCAN on large tables
- [ ] Verify indexes exist on WHERE clause columns
- [ ] Verify indexes exist on JOIN columns
- [ ] Check for functions on indexed columns (not sargable)
- [ ] Select only needed columns (avoid SELECT *)
- [ ] Use LIMIT for large result sets
- [ ] Consider covering indexes for frequent queries
- [ ] Check for N+1 query patterns
- [ ] Update statistics (ANALYZE)
- [ ] Monitor query performance over time
</optimization_checklist>
