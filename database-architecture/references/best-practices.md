<overview>
This reference consolidates database best practices across schema design, querying, indexing, security, and operations. Use this as a checklist when designing, reviewing, or optimizing databases.
</overview>

<schema_design>
**Schema Design Best Practices**

<practice name="primary-keys">
**Always Define Primary Keys**

Every table must have a primary key. No exceptions.

```sql
-- Good: Clear primary key
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE
);

-- For join tables
CREATE TABLE user_roles (
    user_id UUID REFERENCES users(id),
    role_id UUID REFERENCES roles(id),
    PRIMARY KEY (user_id, role_id)
);
```
</practice>

<practice name="foreign-keys">
**Use Foreign Key Constraints**

Enforce referential integrity at the database level.

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    -- ...
);

-- Always index foreign keys
CREATE INDEX idx_orders_user_id ON orders(user_id);
```
</practice>

<practice name="data-types">
**Choose Appropriate Data Types**

| Data | Type | Why |
|------|------|-----|
| Money | DECIMAL(10,2) | Exact precision |
| Timestamps | TIMESTAMPTZ | Timezone-aware |
| UUIDs | UUID | Native type, efficient |
| Status | VARCHAR + CHECK | Flexible, validated |
| JSON | JSONB (PG) | Binary, indexable |
| Boolean | BOOLEAN | Not CHAR(1) or INT |
</practice>

<practice name="constraints">
**Add Appropriate Constraints**

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    status VARCHAR(20) NOT NULL DEFAULT 'draft'
        CHECK (status IN ('draft', 'active', 'discontinued')),
    sku VARCHAR(50) UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```
</practice>

<practice name="naming">
**Consistent Naming Conventions**

- Tables: singular, snake_case (`user`, `order_item`)
- Columns: snake_case (`created_at`, `user_id`)
- Indexes: `idx_{table}_{columns}`
- Foreign keys: `{table}_fk_{column}`
- Boolean: `is_`, `has_`, `can_` prefix
</practice>

<practice name="audit-columns">
**Include Audit Columns**

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    -- ... business columns ...
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by UUID REFERENCES users(id),
    updated_by UUID REFERENCES users(id)
);
```
</practice>
</schema_design>

<indexing>
**Indexing Best Practices**

<practice name="index-what-you-query">
**Index What You Query**

- Columns in WHERE clauses
- Columns in JOIN conditions
- Columns in ORDER BY (if sorting large result sets)
- Foreign key columns (not auto-indexed)
</practice>

<practice name="composite-order">
**Composite Index Column Order**

Order: Equality → Sort → Range (ESR rule)

```sql
-- Query: WHERE user_id = ? AND status = ? ORDER BY created_at WHERE amount > ?
-- Index: equality first, then sort, then range
CREATE INDEX idx ON orders(user_id, status, created_at, amount);
```
</practice>

<practice name="covering-indexes">
**Use Covering Indexes for Hot Queries**

```sql
-- Query returns only status and total
SELECT status, total FROM orders WHERE user_id = ?;

-- Covering index
CREATE INDEX idx_orders_cover ON orders(user_id) INCLUDE (status, total);
```
</practice>

<practice name="partial-indexes">
**Use Partial Indexes for Subsets**

```sql
-- Only 5% of orders are pending, but queried frequently
CREATE INDEX idx_pending_orders ON orders(created_at)
WHERE status = 'pending';
```
</practice>

<practice name="avoid-over-indexing">
**Don't Over-Index**

- Each index slows INSERT/UPDATE/DELETE
- Monitor and remove unused indexes
- Rule: Justify each index with actual query patterns
</practice>
</indexing>

<querying>
**Query Best Practices**

<practice name="select-columns">
**Select Only Needed Columns**

```sql
-- Bad
SELECT * FROM users WHERE id = 123;

-- Good
SELECT id, name, email FROM users WHERE id = 123;
```
</practice>

<practice name="sargable">
**Write Sargable Queries**

```sql
-- Bad: Function prevents index use
WHERE YEAR(created_at) = 2024
WHERE LOWER(email) = 'test@example.com'

-- Good: Index-friendly
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'
WHERE email = 'test@example.com'  -- Store normalized
```
</practice>

<practice name="parameterized">
**Use Parameterized Queries**

```javascript
// Bad: SQL injection risk
db.query(`SELECT * FROM users WHERE id = '${userId}'`);

// Good: Parameterized
db.query('SELECT * FROM users WHERE id = $1', [userId]);
```
</practice>

<practice name="limit-results">
**Limit Large Result Sets**

```sql
-- Always use LIMIT for potentially large results
SELECT * FROM logs ORDER BY created_at DESC LIMIT 100;

-- Use keyset pagination for deep pages
SELECT * FROM orders WHERE id > $last_id ORDER BY id LIMIT 100;
```
</practice>

<practice name="avoid-n-plus-one">
**Avoid N+1 Queries**

```javascript
// Bad: N+1
for (const user of users) {
  const orders = await db.orders.find({ userId: user.id });
}

// Good: Batch or join
const orders = await db.orders.find({ userId: { $in: userIds } });
```
</practice>
</querying>

<document_databases>
**Document Database Best Practices**

<practice name="query-driven">
**Design for Queries**

Model documents based on access patterns, not entity relationships.

```javascript
// If user profile page shows recent orders
{
  _id: "user::123",
  name: "John",
  recentOrders: [{ id: "...", total: 99.99, date: "..." }],
  // Full orders in separate collection
}
```
</practice>

<practice name="bounded-arrays">
**Avoid Unbounded Arrays**

```javascript
// Bad: Can grow indefinitely
{ comments: [/* thousands of items */] }

// Good: Reference or subset
{
  recentComments: [/* last 5 */],
  commentCount: 5432
}
// Full comments in separate collection
```
</practice>

<practice name="index-query-patterns">
**Index for Query Patterns**

```javascript
// Compound index matching query pattern
db.orders.createIndex({ userId: 1, status: 1, createdAt: -1 })

// Query that uses the index efficiently
db.orders.find({ userId: "123", status: "pending" }).sort({ createdAt: -1 })
```
</practice>
</document_databases>

<security>
**Security Best Practices**

<practice name="least-privilege">
**Use Least Privilege**

```sql
-- Application user with minimal permissions
CREATE USER app_user WITH PASSWORD 'secure_password';
GRANT SELECT, INSERT, UPDATE ON orders TO app_user;
-- No DELETE, no DDL
```
</practice>

<practice name="no-hardcoded-creds">
**No Hardcoded Credentials**

```javascript
// Bad
const db = connect('postgres://admin:password123@localhost/db');

// Good
const db = connect(process.env.DATABASE_URL);
```
</practice>

<practice name="encrypt-sensitive">
**Encrypt Sensitive Data**

```sql
-- Encrypt at rest (database configuration)
-- Encrypt in transit (SSL/TLS connections)
-- Encrypt sensitive columns (application-level)
```
</practice>

<practice name="audit-access">
**Enable Audit Logging**

```sql
-- PostgreSQL: Enable logging
-- postgresql.conf:
-- log_statement = 'all'
-- log_connections = on
```
</practice>
</security>

<operations>
**Operational Best Practices**

<practice name="backups">
**Regular Backups**

- Automated daily backups
- Test restore procedures regularly
- Store backups in different location
- Encrypt backup files
</practice>

<practice name="monitoring">
**Monitor Key Metrics**

- Query latency (p50, p95, p99)
- Connection count
- Disk usage and growth
- Replication lag
- Slow query log
</practice>

<practice name="maintenance">
**Regular Maintenance**

```sql
-- PostgreSQL
VACUUM ANALYZE;  -- Reclaim space, update statistics
REINDEX DATABASE dbname;  -- Rebuild indexes

-- MySQL
OPTIMIZE TABLE table_name;
ANALYZE TABLE table_name;
```
</practice>

<practice name="migrations">
**Safe Migrations**

- Test migrations on production data copy
- Use transactions for DDL when possible
- Have rollback plan
- Avoid long-running locks during business hours
- Use expand-contract for zero-downtime changes
</practice>

<practice name="connection-pooling">
**Use Connection Pooling**

- Application-level: Most ORMs support pooling
- External: PgBouncer (PostgreSQL), ProxySQL (MySQL)
- Configure appropriate pool size
</practice>
</operations>

<external_data_integration>
**External Data Integration Pattern: Your Database First**

<overview>
**Golden Rule:** Never query external third-party APIs at runtime for data that can be stored in your database. Sync data from external sources to YOUR database, then use ONLY your database for all application queries.

This pattern prevents rate limiting, API bans, and improves performance. Your database = the database YOU control (same DB for dev and prod is fine). External APIs = third-party services like IGDB, Steam, Stripe, etc.
</overview>

<practice name="sync-then-query">
**Sync Once, Query Your DB Forever**

External third-party APIs should be used for:
1. **Initial data import** - One-time bulk sync
2. **Scheduled updates** - Periodic incremental syncs (cron jobs)
3. **User-specific auth data** - Login/authentication only

External APIs should NEVER be used at runtime for:
- Game metadata, product details, content data
- Any data that can be stored in your database

```
┌─────────────────────────────────────────────────────────────────┐
│                    CORRECT ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Third-Party APIs ──(scheduled sync)──► YOUR Database          │
│   (IGDB, Steam, etc.)                    (dev & prod share)     │
│        │                                       │                 │
│        │                                       │                 │
│   (auth only)                           (ALL app queries)       │
│        │                                       │                 │
│        ▼                                       ▼                 │
│   ┌─────────┐                           ┌───────────┐           │
│   │  User   │                           │  App UI   │           │
│   │ Login   │                           │ (dev/prod)│           │
│   └─────────┘                           └───────────┘           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
</practice>

<practice name="mapping-tables">
**Use Mapping Tables for Cross-Platform IDs**

When syncing from external sources, create mapping tables to link external IDs to your local IDs.

```sql
-- External source data (e.g., IGDB games)
CREATE TABLE games (
    id INTEGER PRIMARY KEY,  -- IGDB ID
    name TEXT NOT NULL,
    cover_image_id TEXT,
    -- ... rich metadata
);

-- Mapping table (e.g., Steam App ID → IGDB Game ID)
CREATE TABLE steam_map (
    steam_app_id INTEGER PRIMARY KEY,
    game_id INTEGER NOT NULL REFERENCES games(id)
);

-- Cache table for unmatched external items
CREATE TABLE steam_cache (
    steam_app_id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    cached_at TIMESTAMP DEFAULT NOW()
);
```
</practice>

<practice name="name-based-matching">
**Auto-Populate Mappings via Name Matching**

When external IDs don't have direct mappings, use name-based matching to create them automatically.

```sql
-- Index for efficient case-insensitive name lookups
CREATE INDEX games_name_lower_idx ON games(LOWER(name));

-- Query: Find local game by name from external source
SELECT id, name, cover_image_id
FROM games
WHERE LOWER(name) = ANY($1)  -- Array of lowercase names
  AND cover_image_id IS NOT NULL;
```

```typescript
// Implementation pattern
async function matchByName(unmappedItems: { externalId: number; name: string }[]) {
  // 1. Search local DB by name (case-insensitive)
  const names = unmappedItems.map(i => i.name.toLowerCase())
  const matches = await db.query(
    `SELECT id, name FROM local_table WHERE LOWER(name) = ANY($1)`,
    [names]
  )

  // 2. Create mappings for matches
  const newMappings = []
  for (const item of unmappedItems) {
    const match = matches.find(m => m.name.toLowerCase() === item.name.toLowerCase())
    if (match) {
      newMappings.push({ externalId: item.externalId, localId: match.id })
    }
  }

  // 3. Batch insert new mappings
  if (newMappings.length > 0) {
    await db.query(
      `INSERT INTO mapping_table (external_id, local_id) VALUES ...
       ON CONFLICT DO NOTHING`
    )
  }

  // 4. Cache remaining unmatched items for future reference
  // ...
}
```
</practice>

<practice name="sync-strategies">
**Sync Strategies**

| Strategy | When to Use | Example |
|----------|-------------|---------|
| **Full Sync** | Initial import, schema changes | Import all 200K games from IGDB |
| **Incremental Sync** | Regular updates | Sync games modified since last sync |
| **On-Demand Population** | User-driven discovery | Match user's Steam library to local DB |

```typescript
// On-demand population: Auto-populate as users bring in new data
async function fetchUserLibrary(userId: string) {
  // Step 1: Get user's items from external auth (IDs only)
  const userItems = await externalApi.getUserItems(userId)  // OK: user-specific

  // Step 2: Check existing local mappings
  const existingMappings = await db.query(
    `SELECT external_id, local_id FROM mappings WHERE external_id = ANY($1)`,
    [userItems.map(i => i.id)]
  )

  // Step 3: Match unmapped items by name (uses LOCAL DB only)
  const unmapped = userItems.filter(i => !existingMappings.has(i.id))
  const newMappings = await matchByName(unmapped)  // Local query only!

  // Step 4: Return enriched data from local DB
  return await db.query(
    `SELECT * FROM local_table WHERE id = ANY($1)`,
    [allMappedIds]
  )
}
```
</practice>

<practice name="benefits">
**Benefits of Your Database First**

| Benefit | Description |
|---------|-------------|
| **No Rate Limits** | Your DB has no external throttling |
| **No API Bans** | Can't be banned from your own database |
| **Performance** | DB queries: ~1-10ms vs external API: ~100-500ms |
| **Reliability** | App works even if third-party API is down |
| **Cost Control** | No per-request API charges at runtime |
| **Data Ownership** | You control the data, can transform/enhance it |
| **Consistency** | Same data in dev and prod (shared DB) |
</practice>

<practice name="when-to-call-external">
**When External API Calls ARE Acceptable**

1. **Authentication** - OAuth, login flows (user-specific, required)
2. **User-specific data** - What games a user owns, their playtime (can't be pre-synced)
3. **Scheduled syncs** - Background jobs to refresh data (not user-facing)
4. **Webhooks** - Push notifications from external services

```typescript
// ACCEPTABLE: Auth-related external call
const userProfile = await steamApi.authenticate(token)  // ✓ Login

// ACCEPTABLE: User-specific data that changes
const userLibrary = await steamApi.getOwnedGames(userId)  // ✓ Their library

// NOT ACCEPTABLE: Game metadata at runtime
const gameDetails = await igdbApi.getGame(gameId)  // ✗ Should be local!
```
</practice>

<checklist_external>
**External Data Integration Checklist**

- [ ] Third-party APIs used only for auth and scheduled syncs
- [ ] All game/product/content data stored in YOUR database
- [ ] Mapping tables link external IDs to your internal IDs
- [ ] Name-based matching auto-populates mappings
- [ ] Functional indexes exist for name lookups (case-insensitive)
- [ ] Cache table captures unmatched items for future reference
- [ ] Scheduled sync job keeps your data fresh
- [ ] No third-party API calls in user-facing request paths (except auth)
</checklist_external>
</external_data_integration>

<checklist>
**Database Best Practices Checklist**

**Schema:**
- [ ] Every table has a primary key
- [ ] Foreign keys defined with appropriate ON DELETE
- [ ] Foreign keys are indexed
- [ ] Appropriate data types (TIMESTAMPTZ, DECIMAL, etc.)
- [ ] NOT NULL where required
- [ ] CHECK constraints for validation
- [ ] Consistent naming conventions

**Indexing:**
- [ ] Indexes on WHERE clause columns
- [ ] Indexes on JOIN columns
- [ ] Indexes on foreign keys
- [ ] Composite index order is correct (ESR)
- [ ] No unused indexes
- [ ] Covering indexes for hot queries

**Queries:**
- [ ] SELECT only needed columns
- [ ] Queries are sargable
- [ ] Parameterized (no SQL injection)
- [ ] LIMIT on large result sets
- [ ] No N+1 patterns
- [ ] EXPLAIN analyzed for slow queries

**Security:**
- [ ] Least privilege access
- [ ] No hardcoded credentials
- [ ] SSL/TLS for connections
- [ ] Sensitive data encrypted
- [ ] Audit logging enabled

**Operations:**
- [ ] Automated backups
- [ ] Restore tested
- [ ] Monitoring in place
- [ ] Maintenance scheduled
- [ ] Connection pooling configured

**External Data (if applicable):**
- [ ] External APIs used only for auth/syncs (not runtime queries)
- [ ] All cacheable data stored locally
- [ ] Mapping tables link external → local IDs
- [ ] Name-based matching auto-populates mappings
</checklist>
