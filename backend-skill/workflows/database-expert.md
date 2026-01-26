# Workflow: Database Expert

<required_reading>
**Read these reference files NOW:**
1. references/core/database-patterns.md
2. references/core/performance.md
3. Framework-specific patterns file for the user's framework
</required_reading>

<context>
Expert guidance on database schema design, migrations, query optimization, indexing strategies, and ORM best practices.
</context>

<process>

<step name="understand-requirements">
Clarify the data requirements:

- What entities need to be stored?
- What are the relationships? (1:1, 1:N, N:M)
- What queries will be most common?
- What's the expected data volume?
- Are there specific performance requirements?
</step>

<step name="design-schema">
Design normalized schema:

```sql
-- Users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Posts with foreign key
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    user_id INTEGER NOT NULL REFERENCES users(id),
    published BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Many-to-many with pivot table
CREATE TABLE post_tags (
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    tag_id INTEGER REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);
```

Apply normalization rules but consider denormalization for read-heavy patterns.
</step>

<step name="plan-indexes">
Add indexes for common queries:

```sql
-- Index on foreign keys
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- Index on frequently filtered columns
CREATE INDEX idx_posts_published ON posts(published) WHERE published = true;

-- Composite index (order matters)
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at DESC);

-- Full-text search
CREATE INDEX idx_posts_search ON posts USING gin(to_tsvector('english', title || ' ' || content));
```

Use EXPLAIN ANALYZE to verify index usage.
</step>

<step name="create-migrations">
Write migrations properly:

**Laravel:**
```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content')->nullable();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->boolean('published')->default(false);
    $table->timestamps();

    $table->index(['user_id', 'created_at']);
});
```

**Node.js (Prisma):**
```prisma
model Post {
    id        Int      @id @default(autoincrement())
    title     String
    content   String?
    userId    Int
    user      User     @relation(fields: [userId], references: [id])
    published Boolean  @default(false)
    createdAt DateTime @default(now())

    @@index([userId, createdAt(sort: Desc)])
}
```

**Python (Alembic):**
```python
def upgrade():
    op.create_table('posts',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('title', sa.String(255), nullable=False),
        sa.Column('user_id', sa.Integer, sa.ForeignKey('users.id')),
    )
    op.create_index('idx_posts_user_id', 'posts', ['user_id'])
```
</step>

<step name="optimize-queries">
Prevent common performance issues:

**N+1 Prevention:**
```php
// Laravel
$posts = Post::with(['author', 'comments'])->get();

// Prisma
const posts = await prisma.post.findMany({
    include: { author: true, comments: true }
});

// SQLAlchemy
posts = session.query(Post).options(
    selectinload(Post.author),
    selectinload(Post.comments)
).all()
```

**Query Analysis:**
```sql
EXPLAIN ANALYZE SELECT * FROM posts WHERE user_id = 1 ORDER BY created_at DESC;

-- Look for:
-- - Seq Scan (table scan) → needs index
-- - Sort → consider index on ORDER BY column
-- - Rows Removed by Filter → index might help
```
</step>

<step name="implement-transactions">
Use transactions for related operations:

```php
// Laravel
DB::transaction(function () use ($data) {
    $order = Order::create($data);
    $order->items()->createMany($data['items']);
    Inventory::decrementMultiple($data['items']);
});
```

Understand isolation levels for your use case.
</step>

<step name="verify">
Verify database design:

```bash
# Run migrations
php artisan migrate  # Laravel
npx prisma migrate dev  # Prisma
alembic upgrade head  # Python

# Check indexes are used
EXPLAIN ANALYZE SELECT ...

# Test performance with realistic data volume
php artisan db:seed

# Monitor query times
# Enable slow query logging
```
</step>

</process>

<anti_patterns>
Avoid:
- Missing indexes on foreign keys
- N+1 queries
- Over-indexing (slows writes)
- SELECT * when you need few columns
- No pagination on large tables
- Missing transactions for related operations
- Not testing with production-like data volume
</anti_patterns>

<success_criteria>
A well-designed database:
- Properly normalized schema
- Indexes on frequently queried columns
- Foreign keys with appropriate ON DELETE behavior
- Migrations are reversible
- No N+1 queries in ORM code
- Transactions for multi-table operations
- EXPLAIN ANALYZE shows index usage
- Performs well at expected data volume
</success_criteria>
