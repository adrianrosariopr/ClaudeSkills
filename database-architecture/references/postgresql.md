<overview>
PostgreSQL is an advanced open-source object-relational database known for extensibility, standards compliance, and robust feature set. It excels at complex queries, JSON support, and analytics workloads. As of 2024-2025, PostgreSQL has become the most admired database according to Stack Overflow surveys.
</overview>

<when_to_use>
**Choose PostgreSQL when:**
- Complex queries with multiple JOINs, CTEs, window functions
- Need JSONB for semi-structured data alongside relational data
- Geospatial data (PostGIS extension)
- Full-text search requirements
- Analytics/OLAP alongside OLTP
- Need for custom types, functions, or extensions
- Data integrity is critical
- Row-level security needed

**Consider alternatives when:**
- Extreme write-heavy workloads (MySQL may have edge)
- Simple CRUD only (MySQL is simpler operationally)
- Need native sharding (consider CockroachDB, Citus, or document DBs)
</when_to_use>

<key_features>
<feature name="jsonb">
**JSONB - Binary JSON**

Store and query JSON efficiently:

```sql
-- Create table with JSONB
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    attributes JSONB
);

-- Insert JSON data
INSERT INTO products (name, attributes)
VALUES ('Laptop', '{"brand": "Dell", "specs": {"ram": 16, "storage": 512}}');

-- Query JSON fields
SELECT * FROM products WHERE attributes->>'brand' = 'Dell';
SELECT * FROM products WHERE attributes->'specs'->>'ram' = '16';
SELECT * FROM products WHERE attributes @> '{"brand": "Dell"}';

-- Index JSONB for performance
CREATE INDEX idx_products_attributes ON products USING GIN (attributes);
CREATE INDEX idx_products_brand ON products ((attributes->>'brand'));
```
</feature>

<feature name="cte-window">
**CTEs and Window Functions**

```sql
-- Common Table Expression (WITH clause)
WITH monthly_sales AS (
    SELECT
        date_trunc('month', created_at) AS month,
        SUM(amount) AS total
    FROM orders
    GROUP BY 1
)
SELECT month, total,
       LAG(total) OVER (ORDER BY month) AS prev_month,
       total - LAG(total) OVER (ORDER BY month) AS growth
FROM monthly_sales;

-- Window functions for analytics
SELECT
    user_id,
    amount,
    SUM(amount) OVER (PARTITION BY user_id ORDER BY created_at) AS running_total,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY amount DESC) AS rank,
    PERCENT_RANK() OVER (ORDER BY amount) AS percentile
FROM orders;
```
</feature>

<feature name="indexes">
**Advanced Index Types**

```sql
-- B-tree (default): Range queries, equality
CREATE INDEX idx_btree ON table(column);

-- GIN: JSONB, arrays, full-text search
CREATE INDEX idx_gin ON table USING GIN (jsonb_column);
CREATE INDEX idx_gin_array ON table USING GIN (array_column);

-- GiST: Geometric, full-text, range types
CREATE INDEX idx_gist ON table USING GiST (point_column);

-- BRIN: Large sequential data (time-series)
CREATE INDEX idx_brin ON events USING BRIN (created_at);

-- Partial index: Subset of rows
CREATE INDEX idx_partial ON orders(created_at) WHERE status = 'pending';

-- Covering index: Include non-key columns
CREATE INDEX idx_covering ON orders(user_id) INCLUDE (status, amount);
```
</feature>

<feature name="partitioning">
**Table Partitioning**

```sql
-- Range partitioning (common for time-series)
CREATE TABLE events (
    id BIGSERIAL,
    created_at TIMESTAMPTZ NOT NULL,
    data JSONB
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2024_q1 PARTITION OF events
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');
CREATE TABLE events_2024_q2 PARTITION OF events
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

-- List partitioning
CREATE TABLE orders (
    id SERIAL,
    region VARCHAR(50),
    amount DECIMAL
) PARTITION BY LIST (region);

CREATE TABLE orders_us PARTITION OF orders FOR VALUES IN ('us-east', 'us-west');
CREATE TABLE orders_eu PARTITION OF orders FOR VALUES IN ('eu-west', 'eu-central');
```
</feature>

<feature name="rls">
**Row Level Security**

```sql
-- Enable RLS
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their own documents
CREATE POLICY user_documents ON documents
    FOR ALL
    USING (user_id = current_setting('app.current_user_id')::uuid);

-- Set user context in application
SET app.current_user_id = 'user-uuid-here';
SELECT * FROM documents; -- Only returns user's documents
```
</feature>
</key_features>

<extensions>
**Popular Extensions**

| Extension | Purpose |
|-----------|---------|
| PostGIS | Geospatial data and queries |
| pg_trgm | Fuzzy text matching, LIKE optimization |
| uuid-ossp | UUID generation |
| pgcrypto | Encryption functions |
| pg_stat_statements | Query performance monitoring |
| timescaledb | Time-series optimization |
| citus | Distributed/sharded PostgreSQL |

```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Use extension
SELECT uuid_generate_v4();
SELECT * FROM products WHERE name % 'laptop'; -- fuzzy match
```
</extensions>

<performance_tuning>
**Key Configuration Parameters**

```ini
# Memory
shared_buffers = 25% of RAM (e.g., 4GB for 16GB system)
effective_cache_size = 75% of RAM
work_mem = 64MB (per operation, be careful with concurrency)
maintenance_work_mem = 512MB

# WAL/Checkpoints
wal_buffers = 64MB
checkpoint_completion_target = 0.9
max_wal_size = 4GB

# Connections
max_connections = 100 (use connection pooling for more)

# Query planner
random_page_cost = 1.1 (for SSD, default 4.0 for HDD)
effective_io_concurrency = 200 (for SSD)
```

**Connection Pooling**

Use PgBouncer for connection pooling:
- Transaction mode: Connection returned after each transaction
- Session mode: Connection held for session lifetime
- Statement mode: Connection returned after each statement
</performance_tuning>

<common_patterns>
<pattern name="upsert">
**Upsert (INSERT ON CONFLICT)**

```sql
INSERT INTO users (email, name, updated_at)
VALUES ('user@example.com', 'John', NOW())
ON CONFLICT (email)
DO UPDATE SET
    name = EXCLUDED.name,
    updated_at = NOW();
```
</pattern>

<pattern name="returning">
**RETURNING Clause**

```sql
-- Get inserted/updated row
INSERT INTO users (email) VALUES ('new@example.com') RETURNING *;
UPDATE users SET name = 'Jane' WHERE id = 1 RETURNING id, name;
DELETE FROM users WHERE id = 1 RETURNING *;
```
</pattern>

<pattern name="advisory-locks">
**Advisory Locks**

```sql
-- Application-level locking
SELECT pg_advisory_lock(12345); -- Acquire lock
-- Do work
SELECT pg_advisory_unlock(12345); -- Release lock

-- Try lock (non-blocking)
SELECT pg_try_advisory_lock(12345);
```
</pattern>
</common_patterns>

<version_info>
**Current Versions (2024-2025)**
- PostgreSQL 17: Latest major release
- PostgreSQL 16: Current stable, widely deployed
- PostgreSQL 15: Still supported

**Key PostgreSQL 17 Features:**
- Improved JSON handling
- Better partitioning performance
- Enhanced logical replication
</version_info>
