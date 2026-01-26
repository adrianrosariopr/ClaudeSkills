<overview>
Schema design is the process of defining database structure - tables, columns, data types, relationships, and constraints. Good schema design ensures data integrity, reduces redundancy, and optimizes query performance. This reference covers relational database schema design principles.
</overview>

<normalization>
**Normal Forms**

<normal_form name="1NF">
**First Normal Form (1NF)**
- All columns contain atomic (indivisible) values
- No repeating groups or arrays
- Each row is unique (has primary key)

```sql
-- Bad: Repeating groups
| id | name | phone1 | phone2 | phone3 |

-- Good: Separate table
| user_id | name |
| phone_id | user_id | phone |
```
</normal_form>

<normal_form name="2NF">
**Second Normal Form (2NF)**
- Must be in 1NF
- All non-key columns depend on the entire primary key
- No partial dependencies

```sql
-- Bad: Partial dependency (product_name depends only on product_id)
| order_id | product_id | product_name | quantity |

-- Good: Separate tables
| order_id | product_id | quantity |
| product_id | product_name |
```
</normal_form>

<normal_form name="3NF">
**Third Normal Form (3NF)**
- Must be in 2NF
- No transitive dependencies (non-key depending on non-key)

```sql
-- Bad: Transitive dependency (city depends on zip)
| id | name | zip | city |

-- Good: Separate lookup
| id | name | zip |
| zip | city |
```
</normal_form>

**When to Denormalize**

Despite normalization benefits, denormalize when:
- Read performance is critical and read:write ratio is high (10:1+)
- Frequently joined tables cause performance issues
- Acceptable to trade storage for speed
- Reporting/analytics queries need pre-computed data

```sql
-- Denormalized: Store computed/joined data
CREATE TABLE order_summary (
    order_id UUID PRIMARY KEY,
    user_id UUID,
    user_name VARCHAR(255),      -- Denormalized from users
    user_email VARCHAR(255),     -- Denormalized from users
    total_amount DECIMAL(10,2),
    item_count INTEGER,
    created_at TIMESTAMPTZ
);
```
</normalization>

<keys>
**Primary Keys**

```sql
-- Auto-increment (simple, but not distributed-friendly)
id SERIAL PRIMARY KEY                    -- PostgreSQL
id INT AUTO_INCREMENT PRIMARY KEY        -- MySQL

-- UUID (distributed-friendly, but larger)
id UUID PRIMARY KEY DEFAULT gen_random_uuid()    -- PostgreSQL
id CHAR(36) PRIMARY KEY                          -- MySQL

-- ULID/KSUID (sortable, distributed-friendly)
id VARCHAR(26) PRIMARY KEY              -- ULID
```

**Primary Key Guidelines:**
- Always have a primary key
- Prefer surrogate keys (id) over natural keys (email)
- UUIDs for distributed systems
- Auto-increment for simple single-server setups
- Consider ULID for sortable distributed IDs

**Foreign Keys**

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    -- ON DELETE options:
    -- CASCADE: Delete child rows
    -- SET NULL: Set to NULL (column must be nullable)
    -- SET DEFAULT: Set to default value
    -- RESTRICT: Prevent deletion (default)
    -- NO ACTION: Similar to RESTRICT, checked at end of transaction
);
```

**Composite Keys**

```sql
-- Many-to-many junction table
CREATE TABLE user_roles (
    user_id UUID REFERENCES users(id),
    role_id UUID REFERENCES roles(id),
    assigned_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (user_id, role_id)
);
```
</keys>

<data_types>
**Choosing Data Types**

| Use Case | PostgreSQL | MySQL | Notes |
|----------|------------|-------|-------|
| Primary key | UUID, SERIAL | CHAR(36), INT AUTO_INCREMENT | UUID for distributed |
| Short text | VARCHAR(n) | VARCHAR(n) | Specify reasonable max |
| Long text | TEXT | TEXT | No length limit |
| Boolean | BOOLEAN | BOOLEAN, TINYINT(1) | |
| Integer | INTEGER, BIGINT | INT, BIGINT | BIGINT for large counts |
| Decimal | NUMERIC(p,s), DECIMAL | DECIMAL(p,s) | For money, exact math |
| Timestamp | TIMESTAMPTZ | DATETIME, TIMESTAMP | Always use timezone-aware |
| JSON | JSONB | JSON | JSONB is binary, faster |
| Enum | ENUM type, VARCHAR | ENUM | VARCHAR more flexible |

**Common Mistakes:**
- Using VARCHAR(255) for everything (use appropriate lengths)
- Using FLOAT for money (use DECIMAL)
- Using TIMESTAMP without timezone
- Using TEXT when VARCHAR would work (harder to validate)
</data_types>

<constraints>
**Constraint Types**

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- NOT NULL: Required field
    name VARCHAR(255) NOT NULL,

    -- UNIQUE: No duplicates
    sku VARCHAR(50) UNIQUE,

    -- CHECK: Custom validation
    price DECIMAL(10,2) CHECK (price >= 0),
    status VARCHAR(20) CHECK (status IN ('active', 'inactive', 'discontinued')),

    -- DEFAULT: Auto-populate
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    is_active BOOLEAN NOT NULL DEFAULT true,

    -- FOREIGN KEY: Referential integrity
    category_id UUID REFERENCES categories(id)
);

-- Multi-column unique constraint
ALTER TABLE products ADD CONSTRAINT unique_name_category
    UNIQUE (name, category_id);

-- Named constraints for better error messages
ALTER TABLE orders ADD CONSTRAINT valid_total
    CHECK (total_amount >= 0);
```
</constraints>

<relationships>
**One-to-Many**

```sql
-- Parent
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name VARCHAR(255)
);

-- Child (many orders per user)
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    total DECIMAL(10,2)
);

CREATE INDEX idx_orders_user ON orders(user_id);
```

**Many-to-Many**

```sql
-- Junction/Association table
CREATE TABLE order_products (
    order_id UUID REFERENCES orders(id) ON DELETE CASCADE,
    product_id UUID REFERENCES products(id) ON DELETE RESTRICT,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (order_id, product_id)
);

CREATE INDEX idx_order_products_product ON order_products(product_id);
```

**One-to-One**

```sql
-- Option 1: Same table (if always accessed together)
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name VARCHAR(255),
    -- Profile fields
    bio TEXT,
    avatar_url VARCHAR(500)
);

-- Option 2: Separate table (if large or optional)
CREATE TABLE user_profiles (
    user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    bio TEXT,
    avatar_url VARCHAR(500)
);
```

**Self-Referential**

```sql
-- Hierarchical data (categories, org chart)
CREATE TABLE categories (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    parent_id UUID REFERENCES categories(id)
);

CREATE INDEX idx_categories_parent ON categories(parent_id);

-- Query hierarchy with CTE
WITH RECURSIVE category_tree AS (
    SELECT id, name, parent_id, 0 AS depth
    FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.name, c.parent_id, ct.depth + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree;
```
</relationships>

<naming_conventions>
**Naming Best Practices**

| Element | Convention | Example |
|---------|------------|---------|
| Tables | Singular, snake_case | `user`, `order_item` |
| Columns | snake_case | `created_at`, `user_id` |
| Primary keys | `id` or `{table}_id` | `id`, `user_id` |
| Foreign keys | `{referenced_table}_id` | `user_id`, `order_id` |
| Indexes | `idx_{table}_{columns}` | `idx_orders_user_id` |
| Constraints | `{table}_{type}_{columns}` | `orders_fk_user_id` |
| Boolean columns | `is_`, `has_`, `can_` prefix | `is_active`, `has_paid` |

**Avoid:**
- Reserved words (user, order, group - use quotes or rename)
- Abbreviations that aren't obvious
- Spaces or special characters
- Mixed case (PostgreSQL lowercases unquoted identifiers)
</naming_conventions>

<common_patterns>
<pattern name="soft-delete">
**Soft Delete**

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    deleted_at TIMESTAMPTZ
);

-- Soft delete
UPDATE users SET deleted_at = NOW() WHERE id = '...';

-- Query active records
SELECT * FROM users WHERE deleted_at IS NULL;

-- Partial unique index (email unique among active users)
CREATE UNIQUE INDEX idx_users_email_active
    ON users(email) WHERE deleted_at IS NULL;
```
</pattern>

<pattern name="audit">
**Audit Columns**

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    -- ... business columns ...

    -- Audit columns
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by UUID REFERENCES users(id),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_by UUID REFERENCES users(id)
);

-- Auto-update trigger (PostgreSQL)
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER orders_updated_at
    BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```
</pattern>

<pattern name="enum-table">
**Enum as Table**

```sql
-- Instead of VARCHAR or database ENUM
CREATE TABLE order_statuses (
    id VARCHAR(20) PRIMARY KEY,
    display_name VARCHAR(50),
    sort_order INTEGER
);

INSERT INTO order_statuses VALUES
    ('pending', 'Pending', 1),
    ('paid', 'Paid', 2),
    ('shipped', 'Shipped', 3),
    ('delivered', 'Delivered', 4),
    ('cancelled', 'Cancelled', 5);

CREATE TABLE orders (
    id UUID PRIMARY KEY,
    status_id VARCHAR(20) REFERENCES order_statuses(id)
);
```
</pattern>
</common_patterns>
