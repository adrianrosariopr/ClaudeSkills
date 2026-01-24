<overview>
MongoDB is a document-oriented NoSQL database that stores data in flexible, JSON-like BSON documents. It excels at handling hierarchical data, rapidly evolving schemas, and horizontal scaling. MongoDB is the most popular document database with a mature ecosystem.
</overview>

<when_to_use>
**Choose MongoDB when:**
- Schema flexibility needed (evolving data structures)
- Hierarchical/nested data naturally fits documents
- Horizontal scaling is a requirement
- Rapid development with changing requirements
- Aggregation pipeline fits your analytics needs
- Real-time analytics on operational data
- Content management, catalogs, user profiles

**Consider alternatives when:**
- Complex transactions across multiple documents
- Need for complex JOINs (relational better)
- Strong ACID requirements (use relational or MongoDB transactions carefully)
- Simple key-value needs (Redis may be better)
- Time-series focus (TimescaleDB, InfluxDB may be better)
</when_to_use>

<data_modeling>
<principle name="embed-vs-reference">
**Embedding vs Referencing**

**Embed (denormalize) when:**
- Data is accessed together
- One-to-few relationship
- Child doesn't exist without parent
- Need atomic updates

```javascript
// Embedded: Address belongs to user, accessed together
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  address: {
    street: "123 Main St",
    city: "New York",
    zip: "10001"
  }
}
```

**Reference (normalize) when:**
- Data accessed independently
- One-to-many (unbounded)
- Many-to-many relationships
- Data duplication is problematic
- Documents approaching 16MB limit

```javascript
// Referenced: Orders are independent, can be many
// User document
{
  _id: ObjectId("user1"),
  name: "John Doe",
  orderIds: [ObjectId("order1"), ObjectId("order2")]
}

// Order document
{
  _id: ObjectId("order1"),
  userId: ObjectId("user1"),
  items: [...],
  total: 99.99
}
```
</principle>

<pattern name="subset">
**Subset Pattern**

Keep frequently accessed data embedded, reference the rest:

```javascript
{
  _id: ObjectId("..."),
  title: "Product Name",
  price: 29.99,
  // Embedded subset: recent reviews for display
  recentReviews: [
    { rating: 5, text: "Great!", date: ISODate("...") },
    { rating: 4, text: "Good", date: ISODate("...") }
  ],
  // Reference: full review collection
  totalReviews: 1234
}

// Separate reviews collection for full history
db.reviews.find({ productId: ObjectId("...") })
```
</pattern>

<pattern name="bucket">
**Bucket Pattern (Time-Series)**

Group related documents to reduce document count:

```javascript
// Instead of one document per reading
// Use buckets
{
  sensorId: "sensor-001",
  date: ISODate("2024-01-15"),
  readings: [
    { time: ISODate("2024-01-15T00:00:00"), value: 23.5 },
    { time: ISODate("2024-01-15T00:01:00"), value: 23.6 },
    // ... more readings
  ],
  count: 1440,
  sum: 33840,
  avg: 23.5
}
```
</pattern>

<pattern name="polymorphic">
**Polymorphic Pattern**

Different document types in same collection:

```javascript
// Different content types in one collection
{ type: "article", title: "...", body: "..." }
{ type: "video", title: "...", url: "...", duration: 300 }
{ type: "podcast", title: "...", audioUrl: "...", transcript: "..." }

// Query by type
db.content.find({ type: "article" })
```
</pattern>
</data_modeling>

<indexes>
**Index Types**

```javascript
// Single field
db.collection.createIndex({ email: 1 })

// Compound (order matters for query optimization)
db.collection.createIndex({ userId: 1, createdAt: -1 })

// Unique
db.collection.createIndex({ email: 1 }, { unique: true })

// Sparse (only index documents with field)
db.collection.createIndex({ optionalField: 1 }, { sparse: true })

// TTL (auto-delete after time)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 86400 })

// Text (full-text search)
db.articles.createIndex({ title: "text", body: "text" })
db.articles.find({ $text: { $search: "mongodb tutorial" } })

// Geospatial
db.locations.createIndex({ coordinates: "2dsphere" })

// Partial (index subset of documents)
db.orders.createIndex(
  { createdAt: 1 },
  { partialFilterExpression: { status: "pending" } }
)
```

**Index Strategies**
- Index fields used in queries (filter, sort)
- Compound index order: equality → sort → range
- ESR rule: Equality, Sort, Range
- Check query explain for COLLSCAN (missing index)
</indexes>

<aggregation>
**Aggregation Pipeline**

```javascript
db.orders.aggregate([
  // $match: Filter documents (use early for efficiency)
  { $match: { status: "completed", createdAt: { $gte: ISODate("2024-01-01") } } },

  // $group: Aggregate by field
  { $group: {
      _id: "$userId",
      totalOrders: { $sum: 1 },
      totalAmount: { $sum: "$amount" },
      avgAmount: { $avg: "$amount" }
  }},

  // $lookup: Join with another collection
  { $lookup: {
      from: "users",
      localField: "_id",
      foreignField: "_id",
      as: "user"
  }},

  // $unwind: Flatten array
  { $unwind: "$user" },

  // $project: Shape output
  { $project: {
      userName: "$user.name",
      totalOrders: 1,
      totalAmount: 1,
      avgAmount: { $round: ["$avgAmount", 2] }
  }},

  // $sort: Order results
  { $sort: { totalAmount: -1 } },

  // $limit: Limit results
  { $limit: 10 }
])
```

**Common Aggregation Stages**
- `$match` - Filter documents
- `$group` - Group and aggregate
- `$project` - Reshape documents
- `$lookup` - Join collections
- `$unwind` - Deconstruct arrays
- `$sort` - Order results
- `$limit` / `$skip` - Pagination
- `$facet` - Multiple pipelines in parallel
</aggregation>

<transactions>
**Multi-Document Transactions (MongoDB 4.0+)**

```javascript
const session = client.startSession();

try {
  session.startTransaction();

  await db.accounts.updateOne(
    { _id: "account1" },
    { $inc: { balance: -100 } },
    { session }
  );

  await db.accounts.updateOne(
    { _id: "account2" },
    { $inc: { balance: 100 } },
    { session }
  );

  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

**Transaction Considerations:**
- Performance overhead vs single document writes
- 60-second default timeout
- Prefer single document atomic operations when possible
- Use for critical multi-document consistency
</transactions>

<replication_sharding>
**Replica Sets**
- Primary: Handles all writes
- Secondaries: Replicate data, can serve reads
- Arbiter: Voting member, no data

```javascript
// Read from secondaries (eventual consistency)
db.collection.find().readPref("secondaryPreferred")
```

**Sharding**
- Distribute data across multiple servers
- Choose shard key carefully (high cardinality, even distribution)
- Avoid scatter-gather queries

```javascript
// Enable sharding on database
sh.enableSharding("mydb")

// Shard collection
sh.shardCollection("mydb.orders", { customerId: "hashed" })
```
</replication_sharding>

<version_info>
**Current Versions (2024-2025)**
- MongoDB 8.0: Latest major release
- MongoDB 7.0: Current stable, widely deployed
- MongoDB 6.0: Still supported

**Key Recent Features:**
- Improved aggregation performance
- Better transaction support
- Time-series collections
- Queryable encryption
</version_info>
