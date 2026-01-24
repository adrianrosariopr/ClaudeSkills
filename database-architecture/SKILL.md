---
name: database-architecture
description: Expert database architect for analyzing, designing, and optimizing data structures. Covers relational (PostgreSQL, MySQL) and document databases (MongoDB, Couchbase), data warehousing, schema design, query optimization, and performance tuning. Use when analyzing project data structures, recommending improvements, or implementing database changes.
---

<essential_principles>

<principle name="query-driven-design">
Design schemas based on how data will be queried, not just how it's structured logically. The access patterns determine the optimal schema, indexes, and denormalization strategy.
</principle>

<principle name="right-tool-for-job">
No single database fits all use cases. Relational databases excel at complex queries and transactions. Document databases excel at flexible schemas and horizontal scaling. Choose based on actual requirements, not trends.
</principle>

<principle name="measure-before-optimizing">
Always profile and measure before optimizing. Use EXPLAIN/EXPLAIN ANALYZE, slow query logs, and monitoring tools. Premature optimization based on assumptions leads to wasted effort and sometimes worse performance.
</principle>

<principle name="normalization-vs-denormalization">
Normalize for write-heavy workloads and data integrity. Denormalize for read-heavy workloads and query performance. Most production systems use a pragmatic mix based on specific access patterns.
</principle>

<principle name="indexes-are-tradeoffs">
Indexes speed up reads but slow down writes. Every index must justify its existence through actual query patterns. Over-indexing is as harmful as under-indexing.
</principle>

</essential_principles>

<intake>
What would you like to do?

1. **Analyze** existing database/project structure
2. **Recommend** schema design or database choice
3. **Implement** database changes or migrations
4. **Optimize** query performance or indexes
5. **Debug** database issues or slow queries
6. **Learn** about a specific database topic

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "analyze", "review", "audit", "check" | `workflows/analyze-project.md` |
| 2, "recommend", "suggest", "design", "choose", "which database" | `workflows/recommend-schema.md` |
| 3, "implement", "create", "migrate", "change", "add" | `workflows/implement-changes.md` |
| 4, "optimize", "slow", "performance", "fast", "index" | `workflows/optimize-performance.md` |
| 5, "debug", "fix", "error", "problem", "issue" | `workflows/debug-issues.md` |
| 6, "learn", "explain", "what is", "how does" | Route to relevant reference file |

**After reading the workflow, follow it exactly.**
</routing>

<verification_loop>
After every database change:

```bash
# 1. Verify schema is valid
# PostgreSQL
\d table_name

# MySQL
DESCRIBE table_name;

# MongoDB
db.collection.findOne()

# 2. Test with sample queries
EXPLAIN ANALYZE SELECT ...

# 3. Check for regressions
# Compare query times before/after
```

Report to user:
- "Schema: Valid"
- "Query performance: X ms (was Y ms)"
- "Indexes: N indexes on table"
</verification_loop>

<reference_index>
All domain knowledge in `references/`:

**Relational Databases:**
- postgresql.md - PostgreSQL features, extensions, patterns
- mysql.md - MySQL features, storage engines, patterns

**Document Databases:**
- mongodb.md - MongoDB modeling, aggregation, patterns
- couchbase.md - Couchbase architecture, N1QL, patterns

**Design & Architecture:**
- schema-design.md - Relational schema design principles
- document-modeling.md - Document database modeling strategies
- data-warehousing.md - Data warehouse architecture patterns

**Performance:**
- indexing-strategies.md - Index types, when to use each
- query-optimization.md - Query tuning techniques

**Quality:**
- best-practices.md - Comprehensive best practices
- anti-patterns.md - Bad patterns to avoid
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| analyze-project.md | Analyze existing data structures and identify issues |
| recommend-schema.md | Recommend optimal schema design or database choice |
| implement-changes.md | Implement database changes safely |
| optimize-performance.md | Profile and optimize query performance |
| debug-issues.md | Diagnose and fix database problems |
</workflows_index>

<quick_decision_guide>
**Choosing between PostgreSQL and MySQL:**
- PostgreSQL: Complex queries, JSON support, extensions, analytics
- MySQL: Simple CRUD, high write throughput, replication simplicity

**Choosing between MongoDB and Couchbase:**
- MongoDB: Flexible schemas, aggregation pipeline, ecosystem
- Couchbase: Memory-first caching, SQL-like queries (N1QL), mobile sync

**Relational vs Document:**
- Relational: ACID transactions, complex joins, data integrity critical
- Document: Flexible/evolving schemas, hierarchical data, horizontal scaling

**When to denormalize:**
- Read-heavy workloads (10:1+ read:write ratio)
- Queries frequently join the same tables
- Acceptable to have eventual consistency
</quick_decision_guide>

<success_criteria>
Database architecture work is successful when:
- Schema matches actual query patterns
- Queries complete within acceptable latency
- No unnecessary indexes or missing critical indexes
- Data integrity constraints are in place
- Migration path is clear and reversible
- Performance is measured, not assumed
</success_criteria>
