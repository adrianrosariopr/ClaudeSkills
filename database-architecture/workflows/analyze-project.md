# Workflow: Analyze Project Database Structure

<required_reading>
**Read these reference files NOW before analyzing:**
1. references/best-practices.md
2. references/anti-patterns.md
3. references/schema-design.md (if relational)
4. references/document-modeling.md (if document database)
</required_reading>

<process>

<step name="identify-database-type">
## Step 1: Identify Database Type and Structure

Examine the project to identify:
- Database type (PostgreSQL, MySQL, MongoDB, Couchbase, or multiple)
- ORM or database driver being used
- Schema definition location (migrations, models, schema files)

**Look for:**
```bash
# Common schema locations
db/schema.rb          # Rails
prisma/schema.prisma  # Prisma
drizzle/schema.ts     # Drizzle
migrations/           # SQL migrations
models/               # ORM models
```

**Configuration files:**
```bash
# Database connection hints
.env                  # Connection strings
database.yml          # Rails
drizzle.config.ts     # Drizzle
prisma/schema.prisma  # Prisma
```
</step>

<step name="extract-schema">
## Step 2: Extract and Document Current Schema

**For SQL databases:**
```sql
-- PostgreSQL: List all tables and columns
SELECT table_name, column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_schema = 'public'
ORDER BY table_name, ordinal_position;

-- List indexes
SELECT indexname, indexdef FROM pg_indexes WHERE schemaname = 'public';

-- List foreign keys
SELECT tc.table_name, kcu.column_name, ccu.table_name AS foreign_table
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage ccu ON tc.constraint_name = ccu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY';
```

**For MongoDB:**
```javascript
// List collections
db.getCollectionNames()

// Sample documents to understand schema
db.collection.findOne()

// Check indexes
db.collection.getIndexes()
```

Document the schema structure clearly.
</step>

<step name="identify-access-patterns">
## Step 3: Identify Query Access Patterns

Search the codebase for database queries:

**ORM queries:**
```bash
# Prisma
grep -r "prisma\." --include="*.ts" --include="*.js"

# Drizzle
grep -r "db\." --include="*.ts" --include="*.js"

# Mongoose
grep -r "\.find\|\.findOne\|\.aggregate" --include="*.ts" --include="*.js"
```

**Raw SQL:**
```bash
grep -r "SELECT\|INSERT\|UPDATE\|DELETE" --include="*.ts" --include="*.js" --include="*.sql"
```

Identify:
- Most frequently executed queries
- Queries with JOINs or aggregations
- Queries with WHERE clauses (index candidates)
- N+1 query patterns
</step>

<step name="check-indexes">
## Step 4: Analyze Index Coverage

For each identified query pattern, verify index coverage:

**PostgreSQL:**
```sql
EXPLAIN ANALYZE SELECT ... -- Run actual queries
```

**MySQL:**
```sql
EXPLAIN SELECT ...
```

**MongoDB:**
```javascript
db.collection.find({...}).explain("executionStats")
```

Look for:
- Sequential scans on large tables (missing indexes)
- Unused indexes (write overhead with no benefit)
- Composite index opportunities
- Covering index opportunities
</step>

<step name="identify-issues">
## Step 5: Identify Anti-Patterns and Issues

Check against common anti-patterns:

**Schema Issues:**
- [ ] Tables without primary keys
- [ ] Missing foreign key constraints
- [ ] Inappropriate data types (VARCHAR for dates, etc.)
- [ ] Entity-Attribute-Value (EAV) patterns
- [ ] Over-normalization or under-normalization

**Query Issues:**
- [ ] N+1 query patterns
- [ ] SELECT * instead of specific columns
- [ ] Functions on indexed columns in WHERE clauses
- [ ] Missing LIMIT on large result sets
- [ ] Cartesian products from missing JOIN conditions

**Index Issues:**
- [ ] Missing indexes on frequently queried columns
- [ ] Missing indexes on foreign keys
- [ ] Redundant indexes
- [ ] Over-indexing (too many indexes)

**Document DB Issues:**
- [ ] Unbounded array growth
- [ ] Documents approaching 16MB limit
- [ ] Excessive embedding causing document bloat
- [ ] Missing references where data is duplicated
</step>

<step name="generate-report">
## Step 6: Generate Analysis Report

Create a structured report:

```markdown
# Database Analysis Report

## Overview
- Database: [PostgreSQL/MySQL/MongoDB/etc.]
- Tables/Collections: [count]
- Total estimated size: [size]

## Schema Summary
[List main tables/collections and their purposes]

## Identified Issues

### Critical (Fix Immediately)
1. [Issue]: [Description and impact]
   - Location: [file:line or table/collection]
   - Recommendation: [How to fix]

### Important (Plan to Fix)
1. [Issue]: [Description and impact]
   - Recommendation: [How to fix]

### Minor (Consider Fixing)
1. [Issue]: [Description]

## Performance Observations
- [Observation about query patterns]
- [Index coverage assessment]

## Recommendations
1. [Priority 1 recommendation]
2. [Priority 2 recommendation]
3. [Priority 3 recommendation]
```
</step>

</process>

<anti_patterns>
Avoid during analysis:
- Making changes without understanding the full picture
- Assuming issues without measuring (use EXPLAIN)
- Ignoring application-level caching that affects query patterns
- Missing the forest for the trees (focus on high-impact issues first)
</anti_patterns>

<success_criteria>
Analysis is complete when:
- [ ] Database type and schema fully documented
- [ ] Query patterns identified from codebase
- [ ] Index coverage analyzed with EXPLAIN
- [ ] Anti-patterns identified and categorized by severity
- [ ] Actionable recommendations provided with priorities
- [ ] Report delivered to user
</success_criteria>
