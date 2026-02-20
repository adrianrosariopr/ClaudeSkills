<overview>
Database anti-patterns are common mistakes that lead to poor performance, data integrity issues, or maintenance nightmares. This reference catalogs anti-patterns across schema design, queries, and operations, explaining why they're problematic and how to fix them.
</overview>

<schema_anti_patterns>
**Schema Design Anti-Patterns**

<anti_pattern name="no-primary-key">
**No Primary Key**

```sql
-- BAD: Table without primary key
CREATE TABLE events (
    event_type VARCHAR(50),
    event_data TEXT,
    created_at TIMESTAMP
);
```

**Problems:**
- Cannot uniquely identify rows
- Cannot efficiently update or delete specific rows
- Replication and CDC tools may fail
- JOINs become unreliable

**Fix:**
```sql
CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY,  -- Or UUID
    event_type VARCHAR(50),
    event_data TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```
</anti_pattern>

<anti_pattern name="eav">
**Entity-Attribute-Value (EAV)**

```sql
-- BAD: EAV pattern
CREATE TABLE entity_attributes (
    entity_id INT,
    attribute_name VARCHAR(255),
    attribute_value TEXT
);

-- Querying is painful
SELECT
    MAX(CASE WHEN attribute_name = 'name' THEN attribute_value END) AS name,
    MAX(CASE WHEN attribute_name = 'email' THEN attribute_value END) AS email
FROM entity_attributes
WHERE entity_id = 123
GROUP BY entity_id;
```

**Problems:**
- Complex, slow queries
- No type safety
- Cannot enforce constraints
- Indexing is difficult

**Fix:**
- Use JSONB for flexible attributes
- Use proper columns for known attributes
- Consider document database if schema truly variable

```sql
CREATE TABLE entities (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    custom_attributes JSONB  -- For truly dynamic fields
);
```
</anti_pattern>

<anti_pattern name="no-foreign-keys">
**Missing Foreign Key Constraints**

```sql
-- BAD: No foreign key
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT,  -- No constraint!
    total DECIMAL(10,2)
);
```

**Problems:**
- Orphaned records (orders without users)
- Data integrity issues
- Application must enforce relationships

**Fix:**
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    total DECIMAL(10,2)
);
CREATE INDEX idx_orders_user_id ON orders(user_id);
```
</anti_pattern>

<anti_pattern name="wrong-data-types">
**Wrong Data Types**

```sql
-- BAD: Inappropriate types
CREATE TABLE products (
    id INT,  -- Should be SERIAL/UUID
    price FLOAT,  -- Should be DECIMAL
    created_at VARCHAR(50),  -- Should be TIMESTAMPTZ
    is_active INT,  -- Should be BOOLEAN
    status CHAR(1)  -- Should be VARCHAR or ENUM
);
```

**Problems:**
- FLOAT for money: Rounding errors
- VARCHAR for timestamps: Sorting issues, no validation
- INT for boolean: No semantic meaning

**Fix:**
```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    is_active BOOLEAN NOT NULL DEFAULT true,
    status VARCHAR(20) CHECK (status IN ('draft', 'active', 'discontinued'))
);
```
</anti_pattern>

<anti_pattern name="god-table">
**God Table (Too Many Columns)**

```sql
-- BAD: 100+ columns, mixed concerns
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    -- ... 50 more user fields
    -- Billing info
    billing_address VARCHAR(255),
    billing_city VARCHAR(100),
    -- ... 20 more billing fields
    -- Preferences
    theme VARCHAR(20),
    notifications_enabled BOOLEAN,
    -- ... 30 more preference fields
);
```

**Problems:**
- Hard to understand and maintain
- Performance issues (wide rows)
- Changes affect entire table
- Locks more data than needed

**Fix:** Split into related tables
```sql
CREATE TABLE users (id UUID PRIMARY KEY, name VARCHAR(255), email VARCHAR(255));
CREATE TABLE user_billing (user_id UUID PRIMARY KEY REFERENCES users(id), ...);
CREATE TABLE user_preferences (user_id UUID PRIMARY KEY REFERENCES users(id), ...);
```
</anti_pattern>

<anti_pattern name="polymorphic-fk">
**Polymorphic Foreign Key**

```sql
-- BAD: One FK column references multiple tables
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    commentable_type VARCHAR(50),  -- 'Post', 'Photo', 'Video'
    commentable_id INT,  -- Could be post_id, photo_id, or video_id
    content TEXT
);
```

**Problems:**
- Cannot use foreign key constraints
- Complex queries
- No referential integrity

**Fix:** Use separate columns or separate tables
```sql
-- Option 1: Separate columns
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INT REFERENCES posts(id),
    photo_id INT REFERENCES photos(id),
    video_id INT REFERENCES videos(id),
    content TEXT,
    CHECK (
        (post_id IS NOT NULL)::int +
        (photo_id IS NOT NULL)::int +
        (video_id IS NOT NULL)::int = 1
    )
);

-- Option 2: Separate join tables
CREATE TABLE post_comments (post_id INT REFERENCES posts, comment_id INT REFERENCES comments);
```
</anti_pattern>
</schema_anti_patterns>

<query_anti_patterns>
**Query Anti-Patterns**

<anti_pattern name="select-star">
**SELECT ***

```sql
-- BAD
SELECT * FROM orders WHERE user_id = 123;
```

**Problems:**
- Fetches unnecessary data
- Breaks if columns added/removed
- Cannot use covering indexes
- More network/memory usage

**Fix:**
```sql
SELECT id, status, total, created_at FROM orders WHERE user_id = 123;
```
</anti_pattern>

<anti_pattern name="function-on-column">
**Functions on Indexed Columns**

```sql
-- BAD: Index cannot be used
SELECT * FROM users WHERE YEAR(created_at) = 2024;
SELECT * FROM users WHERE LOWER(email) = 'test@example.com';
SELECT * FROM orders WHERE amount / 100 > 10;
```

**Problems:**
- Database must evaluate function for every row
- Index is useless

**Fix:**
```sql
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
SELECT * FROM users WHERE email = 'test@example.com';  -- Store lowercase
SELECT * FROM orders WHERE amount > 1000;
```
</anti_pattern>

<anti_pattern name="implicit-conversion">
**Implicit Type Conversion**

```sql
-- BAD: String compared to integer (may prevent index use)
SELECT * FROM users WHERE id = '123';
SELECT * FROM products WHERE price = '29.99';
```

**Fix:**
```sql
SELECT * FROM users WHERE id = 123;
SELECT * FROM products WHERE price = 29.99;
```
</anti_pattern>

<anti_pattern name="n-plus-one">
**N+1 Queries**

```javascript
// BAD: N+1 queries
const users = await db.query('SELECT * FROM users');
for (const user of users) {
    const orders = await db.query(
        'SELECT * FROM orders WHERE user_id = $1', [user.id]
    );
}
```

**Problems:**
- 1 query for users + N queries for orders
- Massive database overhead
- Exponentially slower with more data

**Fix:**
```javascript
// JOIN or batch query
const data = await db.query(`
    SELECT u.*, o.id AS order_id, o.total
    FROM users u
    LEFT JOIN orders o ON o.user_id = u.id
`);

// Or batch
const orders = await db.query(
    'SELECT * FROM orders WHERE user_id = ANY($1)', [userIds]
);
```
</anti_pattern>

<anti_pattern name="cartesian-product">
**Cartesian Product**

```sql
-- BAD: Missing or wrong join condition
SELECT * FROM users, orders;  -- Returns users × orders rows

SELECT * FROM users u
JOIN orders o ON 1=1;  -- Same problem
```

**Fix:**
```sql
SELECT * FROM users u JOIN orders o ON o.user_id = u.id;
```
</anti_pattern>

<anti_pattern name="distinct-bandaid">
**DISTINCT as Band-Aid**

```sql
-- BAD: Using DISTINCT to hide bad JOINs
SELECT DISTINCT u.* FROM users u
JOIN orders o ON o.user_id = u.id;
```

**Problems:**
- Hides the real issue (wrong join, missing condition)
- Performance overhead of deduplication

**Fix:** Fix the query to not produce duplicates
```sql
SELECT u.* FROM users u WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```
</anti_pattern>

<anti_pattern name="sql-in-loop">
**SQL in Application Loop**

```javascript
// BAD: Building SQL in loop
for (const item of items) {
    await db.query('INSERT INTO items VALUES ($1, $2)', [item.id, item.name]);
}
```

**Fix:** Batch insert
```javascript
const values = items.map(i => `('${i.id}', '${i.name}')`).join(',');
await db.query(`INSERT INTO items VALUES ${values}`);

// Or use COPY/bulk insert
```
</anti_pattern>
</query_anti_patterns>

<document_anti_patterns>
**Document Database Anti-Patterns**

<anti_pattern name="unbounded-arrays">
**Unbounded Array Growth**

```javascript
// BAD: Array can grow indefinitely
{
    _id: "user::123",
    followers: [/* millions of IDs */]  // Will hit 16MB limit
}
```

**Fix:** Reference or paginate
```javascript
// Separate collection
db.followers.find({ userId: "user::123" })

// Or subset pattern
{
    _id: "user::123",
    followerCount: 1000000,
    recentFollowers: [/* last 10 */]
}
```
</anti_pattern>

<anti_pattern name="relational-in-nosql">
**Relational Thinking in Document DB**

```javascript
// BAD: Over-normalized like SQL
// users collection
{ _id: "u1", name: "John", addressId: "a1" }
// addresses collection
{ _id: "a1", street: "123 Main", userId: "u1" }
// preferences collection
{ _id: "p1", theme: "dark", userId: "u1" }
```

**Problems:**
- Requires multiple queries
- Loses document model benefits

**Fix:** Embed related data
```javascript
{
    _id: "u1",
    name: "John",
    address: { street: "123 Main" },
    preferences: { theme: "dark" }
}
```
</anti_pattern>

<anti_pattern name="no-indexes">
**Querying Without Indexes**

```javascript
// BAD: No index on queried field
db.orders.find({ userId: "123", status: "pending" })
// Results in COLLSCAN (full collection scan)
```

**Fix:**
```javascript
db.orders.createIndex({ userId: 1, status: 1 })
```
</anti_pattern>
</document_anti_patterns>

<operational_anti_patterns>
**Operational Anti-Patterns**

<anti_pattern name="no-backups">
**No Backup Testing**

Having backups is not enough. Untested backups are worthless.

**Fix:**
- Schedule regular restore tests
- Document restore procedure
- Measure restore time (RTO)
</anti_pattern>

<anti_pattern name="production-testing">
**Testing on Production**

**Problems:**
- Data corruption risk
- Performance impact on users
- Security exposure

**Fix:**
- Use staging environment with production-like data
- Sanitize sensitive data in test environments
</anti_pattern>

<anti_pattern name="no-connection-pooling">
**No Connection Pooling**

```javascript
// BAD: New connection per query
for (const user of users) {
    const conn = await db.connect();
    await conn.query('...');
    conn.close();
}
```

**Fix:** Use connection pooling
```javascript
const pool = new Pool({ max: 20 });
const client = await pool.connect();
// ... use client
client.release();  // Return to pool
```
</anti_pattern>

<anti_pattern name="ignoring-slow-queries">
**Ignoring Slow Query Logs**

**Fix:**
- Enable slow query logging
- Review regularly
- Set up alerts for consistent slow queries
</anti_pattern>
</operational_anti_patterns>

<external_data_anti_patterns>
**External Data Anti-Patterns**

<anti_pattern name="runtime-api-calls">
**Calling Third-Party APIs at Runtime for Cacheable Data**

```typescript
// BAD: External API call every time user views a game
async function getGameDetails(steamAppId: number) {
  const game = await igdbApi.getGame(steamAppId)  // External call!
  return game
}

// BAD: External API in user-facing request path
async function getUserLibrary(userId: string) {
  const games = await steamApi.getOwnedGames(userId)
  for (const game of games) {
    game.details = await igdbApi.getGame(game.id)  // N external calls!
  }
  return games
}
```

**Problems:**
- Rate limiting from external providers
- Risk of API bans
- Slow response times (100-500ms vs 1-10ms local)
- App breaks when external API is down
- Unnecessary API costs
- No control over data availability

**Fix:** Sync data to YOUR database, query your DB only
```typescript
// GOOD: Query YOUR database only
async function getGameDetails(steamAppId: number) {
  const game = await db.query(
    `SELECT g.* FROM games g
     JOIN steam_map sm ON sm.game_id = g.id
     WHERE sm.steam_app_id = $1`,
    [steamAppId]
  )
  return game
}

// GOOD: Your DB for enrichment, third-party only for user-specific data
async function getUserLibrary(userId: string) {
  // Only call third-party for user-specific data (what they own)
  const ownedGames = await steamApi.getOwnedGames(userId)  // OK: user data

  // All enrichment from YOUR database
  const gameIds = ownedGames.map(g => g.appid)
  const details = await db.query(
    `SELECT * FROM games WHERE id = ANY($1)`,
    [gameIds]
  )
  return mergeData(ownedGames, details)
}
```
</anti_pattern>

<anti_pattern name="no-mapping-tables">
**No Mapping Between External and Local IDs**

```typescript
// BAD: Relying on external ID existing in your system
async function getGame(steamAppId: number) {
  // Assumes steam_app_id IS the game ID
  return await db.query('SELECT * FROM games WHERE id = $1', [steamAppId])
}
```

**Problems:**
- External IDs may not match your schema
- Multiple external sources have different IDs for same entity
- No way to enrich data from multiple sources

**Fix:** Use mapping tables
```sql
-- Mapping table links external IDs to your local IDs
CREATE TABLE steam_map (
    steam_app_id INTEGER PRIMARY KEY,
    game_id INTEGER REFERENCES games(id)
);

-- Now can map Steam → IGDB → your data
SELECT g.* FROM games g
JOIN steam_map sm ON sm.game_id = g.id
WHERE sm.steam_app_id = $1;
```
</anti_pattern>

<anti_pattern name="no-fallback-cache">
**No Cache for Unmapped External Items**

```typescript
// BAD: Unmatched items disappear, no way to track them
async function matchGames(steamGames: SteamGame[]) {
  const matched = await db.query(
    'SELECT * FROM steam_map WHERE steam_app_id = ANY($1)',
    [steamGames.map(g => g.appid)]
  )
  // Unmatched games are lost! No record they exist.
  return matched
}
```

**Problems:**
- No visibility into what's missing from your DB
- Can't improve matching over time
- Can't prioritize which data to sync

**Fix:** Cache unmatched items
```typescript
async function matchGames(steamGames: SteamGame[]) {
  const matched = await db.query(...)

  // Cache unmatched for future reference
  const unmatched = steamGames.filter(g => !matched.has(g.appid))
  await db.query(
    `INSERT INTO steam_cache (steam_app_id, name)
     VALUES ${unmatched.map(...)}
     ON CONFLICT DO NOTHING`
  )

  return matched
}
```
</anti_pattern>
</external_data_anti_patterns>

<connection_anti_patterns>
**Connection & Deployment Anti-Patterns**

<anti_pattern name="sslmode-in-connection-string">
**Relying on `sslmode` in Connection Strings (Node.js `pg`)**

```typescript
// BAD: sslmode in URL overrides your ssl Pool option
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,  // Contains ?sslmode=require
  ssl: { rejectUnauthorized: false },  // IGNORED by pg library!
});
// Result: SELF_SIGNED_CERT_IN_CHAIN with AWS RDS
```

**Problems:**
- `pg` parses `sslmode=require` and enables SSL with cert validation ON
- Your explicit `ssl: { rejectUnauthorized: false }` gets overridden
- Works locally (no SSL), breaks in production (RDS requires SSL)
- Error is misleading — looks like missing certs, actually a config conflict

**Fix:** Strip sslmode from URL, control SSL through Pool options only
```typescript
const stripped = url.replace(/[?&]sslmode=[^&]*/g, '').replace(/\?$/, '');
const isLocal = url.includes('localhost') || url.includes('127.0.0.1');
const pool = new Pool({
  connectionString: stripped,
  ssl: isLocal ? undefined : { rejectUnauthorized: false },
});
```
</anti_pattern>

<anti_pattern name="multiple-pool-creation-paths">
**Multiple Pool Creation Paths That Drift**

```typescript
// BAD: Two functions create pools with different config
async function initPool() {
  pool = new Pool({ connectionString, ssl: { rejectUnauthorized: false } }); // Fixed!
}
function getPool() {
  pool = new Pool({ connectionString }); // Missing SSL! Never got the fix.
}
// Every CRUD function uses getPool() → all broken
```

**Problems:**
- Fix applied to one path doesn't fix the other
- The unused/deprecated path is often the one actually called everywhere
- Silent failures — pool creates fine, queries fail later

**Fix:** Single pool creation function, or shared config helper
</anti_pattern>

<anti_pattern name="health-check-needs-initialized-db">
**Health Check Endpoint That Requires Initialized Database**

```typescript
// BAD: Health check queries tables that may not exist yet
export async function GET() {
  const teams = await getTeams();  // ERROR: relation "teams" does not exist
  return NextResponse.json({ teams });
}
// Result: Health check fails → ECS kills task → new task → same failure → infinite loop
```

**Problems:**
- Fresh database has no tables
- ELB health check fails before anyone can trigger schema initialization
- Deployment rolls back to old version forever
- New infrastructure can never boot

**Fix:** Initialize schema in health check endpoint
```typescript
export async function GET() {
  await initializeDatabase();  // CREATE TABLE IF NOT EXISTS (idempotent)
  const teams = await getTeams();
  return NextResponse.json({ teams });
}
```
</anti_pattern>
</connection_anti_patterns>

<summary>
**Anti-Pattern Quick Reference**

| Anti-Pattern | Impact | Fix |
|--------------|--------|-----|
| No primary key | Integrity, updates | Add PK |
| EAV pattern | Performance, complexity | JSONB or proper columns |
| No foreign keys | Orphans, integrity | Add FK constraints |
| SELECT * | Performance | Select needed columns |
| Functions on columns | No index use | Rewrite sargable |
| N+1 queries | Performance | JOIN or batch |
| Unbounded arrays | Document size | Reference/subset |
| No indexes | Full scans | Add indexes |
| No backup testing | Data loss | Regular restore tests |
| **Runtime API calls** | Rate limits, bans, slow | Sync to YOUR DB |
| **No mapping tables** | Can't link external IDs | Create mapping tables |
| **No fallback cache** | Lose unmapped items | Cache for future matching |
| **sslmode in conn string** | pg overrides ssl options | Strip sslmode, use Pool opts |
| **Multiple pool creators** | Config drift, silent failures | Single creation path |
| **Health check needs init'd DB** | Infinite deploy failure loop | Init schema in health check |
</summary>
