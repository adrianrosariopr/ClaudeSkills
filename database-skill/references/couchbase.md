<overview>
Couchbase is a distributed NoSQL document database with an integrated caching layer. It combines the flexibility of JSON documents with the performance of a memory-first architecture. Couchbase excels at low-latency access patterns, caching, and mobile synchronization with Couchbase Mobile.
</overview>

<when_to_use>
**Choose Couchbase when:**
- Sub-millisecond latency requirements
- Built-in caching needed (replaces Redis for some use cases)
- Mobile offline-first applications (Couchbase Mobile/Lite)
- Session management, user profiles, real-time personalization
- SQL-like queries on JSON data (N1QL)
- High-performance key-value access
- Need both document and key-value patterns

**Consider alternatives when:**
- Simple document storage without caching needs (MongoDB may be simpler)
- Complex aggregation pipelines (MongoDB better)
- Team lacks Couchbase expertise
- Smaller scale where memory-first isn't cost-effective
- Need broader cloud availability (MongoDB Atlas more regions)
</when_to_use>

<architecture>
**Couchbase Services**

| Service | Purpose |
|---------|---------|
| Data | Key-value and document storage |
| Index | Secondary indexes for queries |
| Query | N1QL query engine |
| Search | Full-text search (Bleve-based) |
| Analytics | Real-time analytics (separate from operational) |
| Eventing | Server-side functions and triggers |

**Multi-Dimensional Scaling (MDS)**

Each service can scale independently:
```
Node 1: Data + Index
Node 2: Data + Index
Node 3: Query + Search
Node 4: Analytics
```
</architecture>

<n1ql>
**N1QL Query Language**

N1QL (pronounced "nickel") provides SQL-like querying for JSON:

```sql
-- Basic SELECT
SELECT name, email FROM `users` WHERE status = 'active';

-- Nested field access
SELECT address.city, address.zip
FROM `users`
WHERE address.country = 'US';

-- Array operations
SELECT name, ARRAY_LENGTH(orders) AS order_count
FROM `users`
WHERE ANY o IN orders SATISFIES o.total > 100 END;

-- JOINs (using document keys)
SELECT u.name, o.total
FROM `users` u
JOIN `orders` o ON KEYS u.orderIds;

-- Aggregations
SELECT status, COUNT(*) AS count, AVG(total) AS avg_total
FROM `orders`
GROUP BY status;

-- UPSERT
UPSERT INTO `users` (KEY, VALUE)
VALUES ("user::123", {"name": "John", "email": "john@example.com"});

-- UPDATE with nested fields
UPDATE `users`
SET address.verified = true
WHERE META().id = "user::123";
```
</n1ql>

<indexes>
**Index Types**

```sql
-- Primary index (required for ad-hoc queries, avoid in production)
CREATE PRIMARY INDEX ON `bucket`;

-- Secondary index
CREATE INDEX idx_users_email ON `users`(email);

-- Composite index
CREATE INDEX idx_orders_user_status ON `orders`(userId, status, createdAt DESC);

-- Partial index
CREATE INDEX idx_active_users ON `users`(email) WHERE status = 'active';

-- Array index
CREATE INDEX idx_user_tags ON `users`(DISTINCT ARRAY t FOR t IN tags END);

-- Covering index (include additional fields)
CREATE INDEX idx_covering ON `users`(email, name, createdAt);
```

**Index Best Practices:**
- Never rely on primary index in production
- Create specific secondary indexes for queries
- Use covering indexes to avoid document fetches
- Order composite index fields: equality, sort, range
</indexes>

<data_modeling>
**Key Patterns**

```javascript
// Use meaningful key patterns
"user::123"           // User document
"order::456"          // Order document
"user::123::profile"  // Related sub-document

// Key-value access (fastest)
bucket.get("user::123")
bucket.upsert("user::123", document)
```

**Embedding vs Referencing**

Similar principles to MongoDB:

```javascript
// Embedded: frequently accessed together, bounded size
{
  "type": "user",
  "name": "John Doe",
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}

// Referenced: independent access, unbounded
{
  "type": "user",
  "name": "John Doe",
  "orderKeys": ["order::001", "order::002"]
}
```

**Document Design Patterns**

```javascript
// Counter document
{
  "type": "counter",
  "name": "user_count",
  "value": 12345
}
// Use atomic increment: bucket.counter("counter::users", delta)

// Lookup document (secondary key)
{
  "type": "email_lookup",
  "userId": "user::123"
}
// Key: "email::john@example.com"
```
</data_modeling>

<caching>
**Built-in Caching**

Couchbase's memory-first architecture provides:
- Managed cache (integrated, not separate)
- Automatic cache warming on restart
- Configurable ejection policies

**Memory Quotas**

```
Bucket Memory Quota: 1024 MB
- Active data in RAM
- Automatically managed
- Ejection when full (value or full ejection)
```

**vs. Redis:**
- Couchbase: Persistence + Caching integrated
- Redis: Pure cache, need separate persistence
- Couchbase: Query capability on cached data
- Redis: Limited querying, better for simple K/V
</caching>

<mobile>
**Couchbase Mobile**

**Couchbase Lite:**
- Embedded database for mobile/edge
- Offline-first capability
- Runs on iOS, Android, .NET, Java

**Sync Gateway:**
- Synchronization between Lite and Server
- Conflict resolution
- Access control and channels

```javascript
// Channel assignment for sync
{
  "type": "document",
  "channels": ["user::123", "public"]
}
```
</mobile>

<performance>
**Performance Tuning**

**Bucket Configuration:**
- Memory quota: Allocate sufficient RAM
- Replicas: 1-3 for durability (writes to replicas for safety)
- Ejection: Value-only (metadata in RAM) vs Full

**Query Optimization:**
```sql
-- Use EXPLAIN to analyze queries
EXPLAIN SELECT * FROM `users` WHERE email = 'test@example.com';

-- Check index usage
-- Look for: IndexScan (good) vs PrimaryScan (bad)
```

**Sub-document Operations:**

For partial updates, use sub-document API (more efficient):

```javascript
// Instead of fetching entire document
const result = await collection.lookupIn("user::123", [
  couchbase.LookupInSpec.get("name"),
  couchbase.LookupInSpec.get("email")
]);

// Partial update
await collection.mutateIn("user::123", [
  couchbase.MutateInSpec.upsert("lastLogin", new Date())
]);
```
</performance>

<comparison_mongodb>
**Couchbase vs MongoDB**

| Feature | Couchbase | MongoDB |
|---------|-----------|---------|
| Query language | N1QL (SQL-like) | MQL (JSON-based) |
| Caching | Built-in, memory-first | Requires external cache |
| Mobile sync | Native (Couchbase Mobile) | Realm (acquired) |
| Aggregation | N1QL aggregations | Aggregation pipeline (richer) |
| Latency | Sub-millisecond (cached) | Milliseconds |
| Cloud | Capella (fewer regions) | Atlas (more regions) |
| Ecosystem | Smaller | Larger, more tooling |
</comparison_mongodb>

<version_info>
**Current Versions (2024-2025)**
- Couchbase Server 7.x: Current major version
- Couchbase Capella: Managed cloud service

**Key Features:**
- Scopes and Collections (like schemas/tables)
- Transactions support
- Eventing functions
- Analytics service
</version_info>
