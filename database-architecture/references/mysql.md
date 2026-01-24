<overview>
MySQL is a widely-used open-source relational database known for simplicity, speed, and reliability. It excels at high-throughput read/write operations and simple CRUD workloads. MySQL is the backbone of many web applications, particularly in the LAMP/WordPress ecosystem.
</overview>

<when_to_use>
**Choose MySQL when:**
- Simple CRUD operations at high volume
- Read replicas needed for scaling reads
- Team has MySQL expertise
- WordPress, PHP, or LAMP stack
- Simpler operational requirements
- Well-understood replication patterns

**Consider alternatives when:**
- Need complex analytical queries (PostgreSQL better)
- Need JSONB with advanced querying (PostgreSQL better)
- Need custom types or extensions (PostgreSQL better)
- Need native horizontal sharding (consider Vitess, PlanetScale, or CockroachDB)
</when_to_use>

<storage_engines>
**InnoDB (Default - Use This)**
- ACID compliant
- Row-level locking
- Foreign key support
- Crash recovery
- MVCC for concurrent reads

**MyISAM (Legacy - Avoid)**
- Table-level locking
- No transactions
- No foreign keys
- Only use for specific read-heavy scenarios (rarely)

**Memory**
- Data stored in RAM
- Volatile (lost on restart)
- Use for temporary tables, caching

```sql
-- Check/set engine
SHOW TABLE STATUS WHERE Name = 'table_name';
ALTER TABLE table_name ENGINE = InnoDB;
```
</storage_engines>

<key_features>
<feature name="json">
**JSON Support**

```sql
-- Create table with JSON
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    attributes JSON
);

-- Insert JSON
INSERT INTO products (name, attributes)
VALUES ('Laptop', '{"brand": "Dell", "specs": {"ram": 16}}');

-- Query JSON
SELECT * FROM products WHERE JSON_EXTRACT(attributes, '$.brand') = 'Dell';
SELECT * FROM products WHERE attributes->'$.brand' = '"Dell"';

-- Index JSON (generated column)
ALTER TABLE products
ADD COLUMN brand VARCHAR(100) GENERATED ALWAYS AS (attributes->>'$.brand'),
ADD INDEX idx_brand (brand);
```

Note: MySQL's JSON support is less mature than PostgreSQL's JSONB.
</feature>

<feature name="window-functions">
**Window Functions (MySQL 8+)**

```sql
SELECT
    user_id,
    amount,
    SUM(amount) OVER (PARTITION BY user_id ORDER BY created_at) AS running_total,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY amount DESC) AS rank
FROM orders;

-- CTEs (MySQL 8+)
WITH monthly AS (
    SELECT DATE_FORMAT(created_at, '%Y-%m') AS month, SUM(amount) AS total
    FROM orders GROUP BY 1
)
SELECT * FROM monthly ORDER BY month;
```
</feature>

<feature name="replication">
**Replication**

```sql
-- Check replication status (on replica)
SHOW REPLICA STATUS\G

-- Key metrics:
-- Seconds_Behind_Source: Replication lag
-- Replica_IO_Running: Should be Yes
-- Replica_SQL_Running: Should be Yes
```

**Replication Topologies:**
- Source-Replica: Single source, multiple read replicas
- Source-Source: Bidirectional (complex, use carefully)
- Group Replication: Automatic failover, consensus-based
</feature>

<feature name="partitioning">
**Table Partitioning**

```sql
-- Range partitioning
CREATE TABLE events (
    id BIGINT AUTO_INCREMENT,
    created_at DATETIME NOT NULL,
    data JSON,
    PRIMARY KEY (id, created_at)
) PARTITION BY RANGE (YEAR(created_at)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);

-- List partitioning
CREATE TABLE orders (
    id INT AUTO_INCREMENT,
    region VARCHAR(50),
    PRIMARY KEY (id, region)
) PARTITION BY LIST COLUMNS (region) (
    PARTITION p_us VALUES IN ('us-east', 'us-west'),
    PARTITION p_eu VALUES IN ('eu-west', 'eu-central')
);
```
</feature>
</key_features>

<indexes>
**Index Types**

```sql
-- B-tree (default)
CREATE INDEX idx_name ON table(column);

-- Composite index
CREATE INDEX idx_multi ON table(col1, col2, col3);

-- Unique index
CREATE UNIQUE INDEX idx_unique ON table(email);

-- Full-text index
CREATE FULLTEXT INDEX idx_ft ON articles(title, content);
SELECT * FROM articles WHERE MATCH(title, content) AGAINST('search term');

-- Prefix index (for long strings)
CREATE INDEX idx_prefix ON table(long_column(20));

-- Descending index (MySQL 8+)
CREATE INDEX idx_desc ON table(created_at DESC);
```

**Index Hints**

```sql
-- Force index usage
SELECT * FROM orders USE INDEX (idx_status) WHERE status = 'pending';
SELECT * FROM orders FORCE INDEX (idx_status) WHERE status = 'pending';
SELECT * FROM orders IGNORE INDEX (idx_status) WHERE status = 'pending';
```
</indexes>

<performance_tuning>
**Key Configuration Parameters**

```ini
# InnoDB Buffer Pool (most important)
innodb_buffer_pool_size = 70% of RAM (e.g., 11GB for 16GB)
innodb_buffer_pool_instances = 8

# Log files
innodb_log_file_size = 1G
innodb_log_buffer_size = 64M

# Connections
max_connections = 151 (default, increase with pooling)

# Query cache (disabled by default in MySQL 8)
# Don't enable - use application-level caching

# Temporary tables
tmp_table_size = 256M
max_heap_table_size = 256M
```

**Connection Pooling**

Use ProxySQL or application-level pooling:
- MySQL default max_connections is often too low
- Each connection uses memory
- Pool connections at application layer
</performance_tuning>

<common_patterns>
<pattern name="upsert">
**Upsert (INSERT ON DUPLICATE KEY)**

```sql
INSERT INTO users (email, name, updated_at)
VALUES ('user@example.com', 'John', NOW())
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    updated_at = NOW();

-- MySQL 8.0.19+ REPLACE alternative
INSERT INTO users (email, name)
VALUES ('user@example.com', 'John')
AS new_values
ON DUPLICATE KEY UPDATE
    name = new_values.name;
```
</pattern>

<pattern name="last-insert-id">
**Get Last Insert ID**

```sql
INSERT INTO users (email) VALUES ('new@example.com');
SELECT LAST_INSERT_ID();
```
</pattern>

<pattern name="locking">
**Row Locking**

```sql
-- Select for update (exclusive lock)
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- Select for share (shared lock)
SELECT * FROM accounts WHERE id = 1 FOR SHARE;

-- Skip locked rows (MySQL 8+)
SELECT * FROM tasks WHERE status = 'pending' LIMIT 1 FOR UPDATE SKIP LOCKED;
```
</pattern>
</common_patterns>

<mysql_vs_postgresql>
**Key Differences**

| Feature | MySQL | PostgreSQL |
|---------|-------|------------|
| JSON | JSON type, functional | JSONB, superior |
| Full-text | FULLTEXT index | ts_vector, more powerful |
| Replication | Mature, well-understood | Logical replication improving |
| Extensions | Limited | Rich ecosystem |
| Write performance | Slight edge | Comparable |
| Complex queries | Adequate | Superior |
| Licensing | GPL (Oracle) | BSD-like (permissive) |
</mysql_vs_postgresql>

<version_info>
**Current Versions (2024-2025)**
- MySQL 9.0: Latest major release
- MySQL 8.0: Current LTS, widely deployed
- MySQL 5.7: EOL reached, upgrade recommended

**Key MySQL 8/9 Features:**
- Window functions and CTEs
- Improved JSON support
- Better optimizer
- Atomic DDL
- Invisible indexes
</version_info>
