<overview>
Document modeling for NoSQL databases differs fundamentally from relational design. Instead of normalization, design is driven by query patterns and access patterns. This reference covers document modeling strategies for MongoDB, Couchbase, and other document databases.
</overview>

<core_principle>
**Design for Queries, Not Entities**

The driving question: "How will the application access this data?"

**Relational mindset:**
"What entities exist and how are they related?"

**Document mindset:**
"What queries will the application run and what data do they need?"

This shift means:
- Duplicate data if it serves query patterns
- Embed data that's accessed together
- Structure documents around access patterns, not logical entities
</core_principle>

<embedding_vs_referencing>
<embedding>
**When to Embed (Denormalize)**

Embed when:
1. **Data is accessed together** - Read performance priority
2. **One-to-few relationship** - Bounded number of children
3. **Child doesn't exist independently** - Lifecycle tied to parent
4. **Need atomic updates** - Update parent and children together

```javascript
// Good embedding: Address always accessed with user
{
  _id: "user::123",
  name: "John Doe",
  email: "john@example.com",
  addresses: [
    { type: "home", street: "123 Main St", city: "NYC", zip: "10001" },
    { type: "work", street: "456 Corp Ave", city: "NYC", zip: "10002" }
  ]
}
```

**Embedding Limits:**
- MongoDB: 16MB document size limit
- Couchbase: 20MB default (configurable)
- Watch for unbounded array growth
</embedding>

<referencing>
**When to Reference (Normalize)**

Reference when:
1. **Data accessed independently** - Queried on its own
2. **One-to-many (unbounded)** - Could grow indefinitely
3. **Many-to-many** - Shared relationships
4. **Data updates frequently** - Avoid updating many documents
5. **Document size concerns** - Would exceed limits

```javascript
// User document (references orders)
{
  _id: "user::123",
  name: "John Doe",
  email: "john@example.com",
  orderIds: ["order::001", "order::002", "order::003"]
}

// Order document (referenced)
{
  _id: "order::001",
  userId: "user::123",
  items: [...],
  total: 99.99,
  createdAt: ISODate("2024-01-15")
}
```
</referencing>

<hybrid>
**Hybrid Approach**

Often the best solution combines both:

```javascript
// User with subset of order data embedded
{
  _id: "user::123",
  name: "John Doe",
  email: "john@example.com",
  // Embedded: Recent orders for quick display
  recentOrders: [
    { orderId: "order::003", total: 149.99, date: ISODate("2024-01-15") },
    { orderId: "order::002", total: 79.99, date: ISODate("2024-01-10") }
  ],
  totalOrders: 47,
  totalSpent: 3456.78
}
```
</hybrid>
</embedding_vs_referencing>

<design_patterns>
<pattern name="subset">
**Subset Pattern**

Keep working set embedded, full data referenced:

```javascript
// Product with recent reviews embedded
{
  _id: "product::laptop-001",
  name: "Gaming Laptop",
  price: 1299.99,
  // Subset: Top 5 reviews for product page
  topReviews: [
    { rating: 5, text: "Amazing!", userId: "user::456", date: ISODate("...") },
    { rating: 5, text: "Best laptop ever", userId: "user::789", date: ISODate("...") }
  ],
  reviewStats: {
    count: 1234,
    average: 4.7
  }
}

// Full reviews in separate collection
// db.reviews.find({ productId: "product::laptop-001" })
```
</pattern>

<pattern name="bucket">
**Bucket Pattern**

Group time-series or related data into buckets:

```javascript
// Instead of one document per event (millions of documents)
// Bucket by hour/day
{
  _id: "sensor::temp-001::2024-01-15",
  sensorId: "temp-001",
  date: ISODate("2024-01-15"),
  readings: [
    { time: ISODate("2024-01-15T00:00:00"), value: 23.5 },
    { time: ISODate("2024-01-15T00:05:00"), value: 23.6 },
    // ... readings throughout the day
  ],
  stats: {
    count: 288,
    min: 22.1,
    max: 25.3,
    avg: 23.8
  }
}
```

Benefits:
- Fewer documents (better index performance)
- Pre-computed aggregates
- Efficient time-range queries
</pattern>

<pattern name="extended-reference">
**Extended Reference Pattern**

Store frequently accessed fields from referenced document:

```javascript
// Order with extended user reference
{
  _id: "order::001",
  // Extended reference: Key user fields for display
  user: {
    _id: "user::123",
    name: "John Doe",      // Duplicated for display
    email: "john@example.com"  // Duplicated for notifications
  },
  items: [...],
  total: 99.99
}
```

Trade-off: Data duplication vs query simplicity. Update strategy needed when referenced data changes.
</pattern>

<pattern name="computed">
**Computed Pattern**

Store computed/derived data to avoid expensive calculations:

```javascript
{
  _id: "user::123",
  name: "John Doe",
  // Computed fields (updated on write)
  stats: {
    totalOrders: 47,
    totalSpent: 3456.78,
    averageOrderValue: 73.55,
    lastOrderDate: ISODate("2024-01-15"),
    loyaltyTier: "gold"
  }
}
```
</pattern>

<pattern name="polymorphic">
**Polymorphic Pattern**

Different document types in same collection:

```javascript
// Content collection with different types
{ type: "article", title: "...", body: "...", author: "..." }
{ type: "video", title: "...", url: "...", duration: 300 }
{ type: "podcast", title: "...", audioUrl: "...", episodeNumber: 42 }

// Index on type for efficient filtering
db.content.createIndex({ type: 1, createdAt: -1 })
```

Use when:
- Same collection, different structures
- Queried together (e.g., content feed)
- Common base fields with type-specific extensions
</pattern>

<pattern name="schema-versioning">
**Schema Versioning Pattern**

Handle evolving document structures:

```javascript
{
  _id: "user::123",
  schemaVersion: 2,
  name: "John Doe",
  // v2 added this field
  preferences: {
    theme: "dark"
  }
}

// Migration on read (lazy migration)
if (doc.schemaVersion < 2) {
  doc.preferences = { theme: "light" }; // default
  doc.schemaVersion = 2;
  // Optionally save back
}
```
</pattern>

<pattern name="tree">
**Tree Structures**

For hierarchical data:

```javascript
// Materialized Path (simple queries, complex updates)
{
  _id: "cat::electronics",
  name: "Electronics",
  path: "/",
  ancestors: []
}
{
  _id: "cat::computers",
  name: "Computers",
  path: "/electronics/",
  ancestors: ["cat::electronics"]
}
{
  _id: "cat::laptops",
  name: "Laptops",
  path: "/electronics/computers/",
  ancestors: ["cat::electronics", "cat::computers"]
}

// Query all descendants
db.categories.find({ path: /^\/electronics\// })

// Query all ancestors
db.categories.find({ _id: { $in: doc.ancestors } })
```
</pattern>
</design_patterns>

<anti_patterns>
**Document Modeling Anti-Patterns**

<anti_pattern name="unbounded-arrays">
**Unbounded Array Growth**

```javascript
// BAD: Array can grow indefinitely
{
  _id: "user::123",
  comments: [/* millions of comments */]  // Will hit 16MB limit
}

// GOOD: Reference or bucket
{
  _id: "user::123",
  recentCommentIds: [/* last 10 */],
  totalComments: 1000000
}
```
</anti_pattern>

<anti_pattern name="massive-arrays">
**Massive Arrays**

```javascript
// BAD: 10,000 items in array
{
  followers: [/* 10,000 user IDs */]
}

// GOOD: Separate collection with index
// db.followers: { userId: "...", followerId: "..." }
```
</anti_pattern>

<anti_pattern name="relational-in-nosql">
**Relational Design in NoSQL**

```javascript
// BAD: Normalized like SQL (requires multiple queries)
// users collection
{ _id: "user1", name: "John" }
// addresses collection
{ userId: "user1", street: "..." }
// preferences collection
{ userId: "user1", theme: "dark" }

// GOOD: Embed if accessed together
{
  _id: "user1",
  name: "John",
  address: { street: "..." },
  preferences: { theme: "dark" }
}
```
</anti_pattern>

<anti_pattern name="over-embedding">
**Over-Embedding**

```javascript
// BAD: Everything embedded (document too large, hard to query parts)
{
  _id: "order::001",
  user: {/* full user document */},
  items: [{
    product: {/* full product document */},
    // ...
  }]
}

// GOOD: Reference with extended reference pattern
{
  _id: "order::001",
  user: { _id: "user::123", name: "John" },  // Just needed fields
  items: [{
    productId: "product::456",
    name: "Widget",  // Snapshot at order time
    price: 29.99
  }]
}
```
</anti_pattern>
</anti_patterns>

<decision_framework>
**Embedding vs Reference Decision Tree**

```
Is the data accessed together 95%+ of the time?
├─ YES → Does it risk growing unbounded?
│        ├─ YES → Use REFERENCE or SUBSET pattern
│        └─ NO → EMBED
└─ NO → Is the data frequently updated independently?
         ├─ YES → REFERENCE
         └─ NO → Is there a many-to-many relationship?
                  ├─ YES → REFERENCE
                  └─ NO → Consider EMBED with extended reference
```
</decision_framework>
