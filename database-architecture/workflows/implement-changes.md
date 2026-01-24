# Workflow: Implement Database Changes

<required_reading>
**Read these reference files NOW before implementing:**
1. references/best-practices.md
2. Database-specific reference (postgresql.md, mysql.md, mongodb.md, couchbase.md)
3. references/anti-patterns.md
</required_reading>

<process>

<step name="understand-change">
## Step 1: Understand the Change

Clarify with user:
- What change is needed? (new table, new column, index, migration)
- Why is the change needed? (new feature, performance, refactoring)
- What's the current state?
- Is there existing data that needs migration?
- What's the rollback strategy if something goes wrong?
</step>

<step name="assess-impact">
## Step 2: Assess Impact

Before implementing, assess:

**Data Impact:**
- Will existing data be affected?
- Is data transformation needed?
- Could data be lost? (DROP, TRUNCATE, type changes)

**Performance Impact:**
- Will the change lock tables?
- How long will the migration take?
- Can it run during production hours?

**Application Impact:**
- What code changes are needed?
- Is the change backwards compatible?
- Do queries need updating?

**Risk Level:**
- Low: Adding nullable columns, adding indexes (non-blocking)
- Medium: Adding NOT NULL columns with defaults, data migrations
- High: Dropping columns/tables, changing types, large data migrations
</step>

<step name="plan-migration">
## Step 3: Plan the Migration

**For Schema Changes:**

Create migration files following project conventions:

```bash
# Drizzle
npx drizzle-kit generate

# Prisma
npx prisma migrate dev --name add_feature

# Raw SQL
migrations/YYYYMMDDHHMMSS_description.sql
```

**Migration Best Practices:**

1. **One change per migration** - easier to debug and rollback
2. **Always include rollback** - down migration or documented reversal
3. **Test on copy of production data** - never test on production first
4. **Use transactions where possible** - atomic changes

```sql
-- Good migration structure
BEGIN;

-- Forward migration
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
CREATE INDEX idx_users_phone ON users(phone);

COMMIT;

-- Document rollback
-- ROLLBACK:
-- DROP INDEX idx_users_phone;
-- ALTER TABLE users DROP COLUMN phone;
```
</step>

<step name="handle-zero-downtime">
## Step 4: Handle Zero-Downtime Migrations

For production systems, use expand-contract pattern:

**Phase 1: Expand**
- Add new columns/tables
- Keep old structure intact
- Application writes to both old and new

**Phase 2: Migrate**
- Backfill data to new structure
- Verify data integrity

**Phase 3: Contract**
- Update application to use only new structure
- Remove old columns/tables after verification

```sql
-- Example: Renaming a column safely

-- Phase 1: Add new column
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

-- Phase 2: Backfill
UPDATE users SET full_name = name WHERE full_name IS NULL;

-- Phase 3: Application updated to use full_name
-- Then later:
ALTER TABLE users DROP COLUMN name;
```

**For Large Data Migrations:**

```sql
-- Batch processing to avoid locks
DO $$
DECLARE
    batch_size INT := 1000;
    affected INT;
BEGIN
    LOOP
        UPDATE users
        SET full_name = name
        WHERE full_name IS NULL
        LIMIT batch_size;

        GET DIAGNOSTICS affected = ROW_COUNT;
        EXIT WHEN affected = 0;

        COMMIT;
        PERFORM pg_sleep(0.1); -- Brief pause to reduce load
    END LOOP;
END $$;
```
</step>

<step name="implement-change">
## Step 5: Implement the Change

**For Drizzle ORM (current project):**

1. Update schema in `db/schema/`:
```typescript
// Example: Adding a new table
export const newTable = pgTable('new_table', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 255 }).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});
```

2. Generate migration:
```bash
npm run db:generate
```

3. Review generated SQL in `db/migrations/`

4. Apply migration:
```bash
npm run db:push  # Development
npm run db:migrate  # Production
```

**For Raw SQL:**

1. Create migration file
2. Test on development/staging
3. Apply with transaction wrapper
4. Verify with queries
</step>

<step name="verify-change">
## Step 6: Verify the Change

After applying:

```sql
-- Verify schema
\d table_name  -- PostgreSQL
DESCRIBE table_name;  -- MySQL

-- Verify data integrity
SELECT COUNT(*) FROM table_name;
SELECT * FROM table_name LIMIT 5;

-- Verify indexes
SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'table_name';

-- Verify constraints
SELECT conname, contype FROM pg_constraint WHERE conrelid = 'table_name'::regclass;
```

**For MongoDB:**
```javascript
db.collection.findOne()
db.collection.getIndexes()
db.collection.stats()
```

Report results to user with before/after comparison.
</step>

</process>

<rollback_procedures>
If something goes wrong:

**PostgreSQL/MySQL:**
```sql
-- If in transaction
ROLLBACK;

-- If committed, apply reverse migration
-- (Should have documented rollback steps)
```

**MongoDB:**
```javascript
// Document-level rollback
db.collection.updateMany(
  { migratedField: { $exists: true } },
  { $unset: { migratedField: "" } }
)
```

**Always:**
- Have database backups before major changes
- Test rollback procedure before applying migration
- Keep rollback scripts alongside migrations
</rollback_procedures>

<anti_patterns>
Avoid:
- Making schema changes directly in production without testing
- Dropping columns/tables without verifying they're unused
- Running long-running migrations during peak hours
- Forgetting to update application code after schema changes
- Not having a rollback plan
</anti_patterns>

<success_criteria>
Implementation is complete when:
- [ ] Change is clearly understood and planned
- [ ] Impact assessment completed
- [ ] Migration files created and reviewed
- [ ] Change applied successfully
- [ ] Data integrity verified
- [ ] Rollback procedure documented
- [ ] Application code updated if needed
</success_criteria>
