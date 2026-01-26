# Workflow: Optimize Database Performance

<required_reading>
**Read these reference files NOW before optimizing:**
1. references/query-optimization.md
2. references/indexing-strategies.md
3. references/anti-patterns.md
4. Database-specific reference based on user's database
</required_reading>

<process>

<step name="identify-problem">
## Step 1: Identify the Performance Problem

Ask the user:
- Which queries are slow?
- What's the current latency vs expected?
- When did the problem start? (data growth, code change, traffic spike)
- Is it consistent or intermittent?

If user doesn't know which queries are slow:

**PostgreSQL - Find slow queries:**
```sql
-- Enable logging (if not already)
-- In postgresql.conf: log_min_duration_statement = 1000

-- Query pg_stat_statements
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;
```

**MySQL - Find slow queries:**
```sql
-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;

-- Check slow queries
SELECT * FROM mysql.slow_log ORDER BY start_time DESC LIMIT 20;
```

**MongoDB - Find slow queries:**
```javascript
// Check profiler
db.setProfilingLevel(1, { slowms: 100 })
db.system.profile.find().sort({ ts: -1 }).limit(20)
```
</step>

<step name="analyze-query">
## Step 2: Analyze Query Execution

**PostgreSQL:**
```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT ... -- the slow query

-- Key things to look for:
-- - Seq Scan on large tables (missing index)
-- - Nested Loop with high row counts (consider hash/merge join)
-- - Sort operations (consider index for ORDER BY)
-- - actual time vs estimated rows mismatch (stale statistics)
```

**MySQL:**
```sql
EXPLAIN FORMAT=JSON SELECT ...
-- or
EXPLAIN ANALYZE SELECT ...

-- Look for:
-- - type: ALL (full table scan)
-- - rows: high numbers
-- - Extra: Using filesort, Using temporary
```

**MongoDB:**
```javascript
db.collection.find({...}).explain("executionStats")

// Look for:
// - COLLSCAN (collection scan, no index)
// - totalDocsExamined >> nReturned (inefficient)
// - executionTimeMillis
```
</step>

<step name="identify-root-cause">
## Step 3: Identify Root Cause

Common causes and solutions:

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| Seq Scan / COLLSCAN | Missing index | Add appropriate index |
| High row estimates | Stale statistics | ANALYZE table / rebuild stats |
| Sort operation | No index for ORDER BY | Add sorted index |
| Many nested loops | Missing join index | Index join columns |
| Temp table/filesort | Complex sorting/grouping | Optimize query or add covering index |
| High buffer reads | Index not covering | Create covering index |
| Lock waits | Contention | Optimize transactions, reduce lock scope |
</step>

<step name="optimize">
## Step 4: Apply Optimizations

**Add Missing Indexes:**

```sql
-- PostgreSQL/MySQL
CREATE INDEX idx_name ON table(column);

-- Composite index for multi-column queries
CREATE INDEX idx_multi ON table(col1, col2, col3);

-- Covering index (PostgreSQL)
CREATE INDEX idx_covering ON table(filter_col) INCLUDE (select_col1, select_col2);

-- Partial index for subset queries
CREATE INDEX idx_partial ON orders(created_at) WHERE status = 'pending';
```

```javascript
// MongoDB
db.collection.createIndex({ field: 1 })
db.collection.createIndex({ field1: 1, field2: -1 })  // compound
db.collection.createIndex({ field: 1 }, { partialFilterExpression: { status: "active" } })
```

**Optimize Query Structure:**

```sql
-- Before: Function on indexed column
SELECT * FROM users WHERE YEAR(created_at) = 2024;

-- After: Sargable query
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

```sql
-- Before: SELECT *
SELECT * FROM orders WHERE user_id = 123;

-- After: Select only needed columns
SELECT id, status, total FROM orders WHERE user_id = 123;
```

```sql
-- Before: Subquery
SELECT * FROM orders WHERE user_id IN (SELECT id FROM users WHERE status = 'active');

-- After: JOIN
SELECT o.* FROM orders o
JOIN users u ON o.user_id = u.id
WHERE u.status = 'active';
```

**Update Statistics:**

```sql
-- PostgreSQL
ANALYZE table_name;
VACUUM ANALYZE table_name;

-- MySQL
ANALYZE TABLE table_name;
```

```javascript
// MongoDB - rebuild indexes
db.collection.reIndex()
```
</step>

<step name="verify-improvement">
## Step 5: Verify Improvement

Run EXPLAIN again and compare:

```sql
-- Before optimization
-- Seq Scan on orders (cost=0.00..15234.00 rows=500000 width=100)
--   Filter: (status = 'pending')
--   Rows Removed by Filter: 450000
-- Execution Time: 234.567 ms

-- After adding index
-- Index Scan using idx_orders_status on orders (cost=0.42..1234.56 rows=50000 width=100)
--   Index Cond: (status = 'pending')
-- Execution Time: 12.345 ms
```

**Metrics to compare:**
- Execution time (before vs after)
- Rows examined vs rows returned
- Scan type (Seq Scan → Index Scan)
- Buffer/page reads

Report improvement to user:
```
Optimization Results:
- Query: SELECT * FROM orders WHERE status = 'pending'
- Before: 234ms (Seq Scan, 500K rows examined)
- After: 12ms (Index Scan, 50K rows examined)
- Improvement: 95% faster
```
</step>

<step name="prevent-regression">
## Step 6: Prevent Regression

Recommendations for ongoing performance:

1. **Monitor slow queries** - Set up logging thresholds
2. **Regular maintenance** - Schedule VACUUM/ANALYZE (PostgreSQL)
3. **Index monitoring** - Track unused indexes, add missing ones
4. **Query review** - Review new queries before deployment
5. **Load testing** - Test with production-like data volumes
</step>

</process>

<quick_wins>
Common optimizations that often help:

1. **Add indexes on foreign keys** - Often missed, always needed for JOINs
2. **Add indexes on WHERE clause columns** - Filter columns need indexes
3. **Use covering indexes** - Include selected columns to avoid table lookup
4. **Avoid SELECT *** - Select only needed columns
5. **Add LIMIT** - Don't fetch more rows than needed
6. **Batch operations** - Process in chunks instead of single massive query
</quick_wins>

<anti_patterns>
Avoid:
- Adding indexes without measuring impact
- Over-indexing (slows down writes)
- Optimizing queries that run rarely
- Ignoring the application layer (N+1 queries)
- Making changes in production without testing
</anti_patterns>

<success_criteria>
Optimization is complete when:
- [ ] Slow queries identified with metrics
- [ ] Root cause determined via EXPLAIN
- [ ] Optimization applied (index, query rewrite, etc.)
- [ ] Improvement measured and reported
- [ ] No regression in other queries
- [ ] Monitoring in place for future issues
</success_criteria>
