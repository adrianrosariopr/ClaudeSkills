# Workflow: Recommend Schema Design or Database Choice

<required_reading>
**Read these reference files NOW before recommending:**
1. references/schema-design.md
2. references/document-modeling.md
3. references/best-practices.md
4. Database-specific reference based on user's context
</required_reading>

<process>

<step name="gather-requirements">
## Step 1: Gather Requirements

Ask the user about:

**Data Characteristics:**
- What entities/data types need to be stored?
- What are the relationships between entities?
- Expected data volume (rows/documents, growth rate)
- Data consistency requirements (ACID vs eventual)

**Access Patterns:**
- Read:write ratio (read-heavy, write-heavy, balanced)
- Query complexity (simple lookups, complex joins, aggregations)
- Real-time requirements (latency tolerance)
- Concurrent users/connections

**Constraints:**
- Existing infrastructure (cloud provider, existing databases)
- Team expertise
- Budget considerations
- Compliance/regulatory requirements
</step>

<step name="recommend-database">
## Step 2: Recommend Database Type

Based on requirements, recommend:

**PostgreSQL when:**
- Complex queries with multiple JOINs
- Need for ACID transactions
- JSON + relational data (JSONB support)
- Advanced features (CTEs, window functions, full-text search)
- Geospatial data (PostGIS)
- Analytics workloads alongside OLTP

**MySQL when:**
- Simple CRUD operations at high volume
- Read replicas for scaling reads
- Team has MySQL expertise
- Simpler operational requirements
- WordPress/PHP ecosystem

**MongoDB when:**
- Rapidly evolving schema requirements
- Hierarchical/nested data structures
- Horizontal scaling requirement
- Document-centric data model
- Aggregation pipeline needs

**Couchbase when:**
- Need built-in caching layer
- Mobile offline-first applications
- SQL-like queries on JSON (N1QL)
- Sub-millisecond latency requirements
- Session/profile data

**Multiple databases when:**
- Different access patterns for different data
- Polyglot persistence (right tool for each job)
- Example: PostgreSQL for transactions + MongoDB for logs
</step>

<step name="design-schema">
## Step 3: Design Schema Structure

**For Relational Databases:**

1. Identify entities and attributes
2. Determine relationships (1:1, 1:N, N:M)
3. Normalize to 3NF initially
4. Identify denormalization opportunities based on query patterns
5. Define primary keys (prefer UUIDs or auto-increment based on use case)
6. Define foreign keys with appropriate ON DELETE behavior
7. Add appropriate constraints (NOT NULL, UNIQUE, CHECK)

```sql
-- Example well-designed schema
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    status VARCHAR(50) NOT NULL DEFAULT 'pending',
    total_amount DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT valid_status CHECK (status IN ('pending', 'paid', 'shipped', 'delivered', 'cancelled'))
);

-- Index for common query pattern
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status) WHERE status != 'delivered';
```

**For Document Databases:**

1. Model based on access patterns, not entities
2. Embed data that's accessed together
3. Reference data that's accessed independently or grows unbounded
4. Consider the subset pattern for large arrays
5. Plan for document size limits (16MB MongoDB)

```javascript
// Example well-designed document
{
  _id: ObjectId("..."),
  email: "user@example.com",
  profile: {
    name: "John Doe",
    avatar: "https://..."
  },
  // Embed: accessed with user, bounded size
  preferences: {
    theme: "dark",
    notifications: true
  },
  // Reference: accessed independently, can grow large
  orderIds: [ObjectId("..."), ObjectId("...")],
  // Subset pattern: recent orders embedded, full list referenced
  recentOrders: [
    { orderId: ObjectId("..."), total: 99.99, date: ISODate("...") }
  ],
  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```
</step>

<step name="plan-indexes">
## Step 4: Plan Index Strategy

Based on query patterns, plan indexes:

**Relational:**
```sql
-- Primary keys (automatic)
-- Foreign keys (always index)
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Frequently filtered columns
CREATE INDEX idx_orders_status ON orders(status);

-- Composite for common query patterns
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial indexes for subset queries
CREATE INDEX idx_active_orders ON orders(created_at) WHERE status = 'pending';

-- Covering indexes for read-heavy queries
CREATE INDEX idx_orders_covering ON orders(user_id) INCLUDE (status, total_amount);
```

**Document:**
```javascript
// Single field indexes
db.orders.createIndex({ userId: 1 })

// Compound indexes (query pattern order matters)
db.orders.createIndex({ userId: 1, status: 1, createdAt: -1 })

// Text indexes for search
db.products.createIndex({ name: "text", description: "text" })

// TTL indexes for expiring data
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 86400 })
```
</step>

<step name="document-design">
## Step 5: Document the Design

Create a design document:

```markdown
# Database Design: [Project Name]

## Database Choice
**Recommended:** [Database]
**Rationale:** [Why this database fits the requirements]

## Schema Design

### Tables/Collections

#### [Entity Name]
- Purpose: [What this stores]
- Relationships: [Related entities]
- Expected size: [Rows/documents, growth]

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | UUID | PK | Auto-generated |
| ... | ... | ... | ... |

### Indexes

| Index | Columns | Type | Purpose |
|-------|---------|------|---------|
| idx_... | col1, col2 | B-tree | [Query pattern it supports] |

## Data Flow
[How data moves through the system]

## Migration Path
[If migrating from existing system]

## Trade-offs
- [Trade-off 1 and reasoning]
- [Trade-off 2 and reasoning]
```
</step>

</process>

<anti_patterns>
Avoid:
- Recommending a database based on popularity rather than requirements
- Over-normalizing without considering query patterns
- Under-indexing during initial design
- Ignoring data growth projections
- Designing without understanding access patterns
</anti_patterns>

<success_criteria>
Recommendation is complete when:
- [ ] Requirements fully understood and documented
- [ ] Database type recommended with clear rationale
- [ ] Schema designed matching access patterns
- [ ] Index strategy planned
- [ ] Trade-offs explicitly documented
- [ ] User has clear path to implementation
</success_criteria>
