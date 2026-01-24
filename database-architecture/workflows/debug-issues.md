# Workflow: Debug Database Issues

<required_reading>
**Read these reference files based on the issue type:**
- Connection issues → Database-specific reference
- Query errors → references/query-optimization.md
- Data issues → references/anti-patterns.md
- Performance issues → references/indexing-strategies.md
</required_reading>

<process>

<step name="categorize-issue">
## Step 1: Categorize the Issue

Ask the user to describe the issue. Common categories:

| Category | Symptoms |
|----------|----------|
| Connection | Can't connect, connection timeouts, too many connections |
| Query Error | SQL syntax errors, constraint violations, type mismatches |
| Data Integrity | Missing data, duplicate data, orphaned records |
| Performance | Slow queries, timeouts, high CPU/memory |
| Locking | Deadlocks, lock wait timeouts, blocked queries |
| Replication | Lag, inconsistent reads, sync failures |
</step>

<step name="gather-information">
## Step 2: Gather Diagnostic Information

**Get the error message:**
```
What's the exact error message?
When does it occur? (Always, intermittently, under load)
What changed recently? (Code, data volume, infrastructure)
```

**Check database status:**

**PostgreSQL:**
```sql
-- Connection status
SELECT count(*) FROM pg_stat_activity;
SELECT * FROM pg_stat_activity WHERE state != 'idle';

-- Lock status
SELECT * FROM pg_locks WHERE NOT granted;

-- Replication status
SELECT * FROM pg_stat_replication;

-- Table statistics
SELECT relname, n_live_tup, n_dead_tup, last_vacuum, last_analyze
FROM pg_stat_user_tables;
```

**MySQL:**
```sql
-- Connection status
SHOW PROCESSLIST;
SHOW STATUS LIKE 'Threads_connected';

-- Lock status
SHOW ENGINE INNODB STATUS;
SELECT * FROM information_schema.INNODB_LOCKS;

-- Replication status
SHOW SLAVE STATUS;
```

**MongoDB:**
```javascript
// Server status
db.serverStatus()

// Current operations
db.currentOp()

// Replica set status
rs.status()

// Collection stats
db.collection.stats()
```
</step>

<step name="diagnose-connection-issues">
## Step 3a: Diagnose Connection Issues

**"Connection refused":**
- Database not running? Check service status
- Wrong host/port? Verify connection string
- Firewall blocking? Check network rules

**"Too many connections":**
```sql
-- PostgreSQL: Check max connections
SHOW max_connections;
SELECT count(*) FROM pg_stat_activity;

-- Increase if needed (requires restart)
-- postgresql.conf: max_connections = 200

-- Or use connection pooling (PgBouncer, etc.)
```

**"Connection timeout":**
- Network latency? Check ping to database
- Slow query blocking connections? Check active queries
- DNS resolution slow? Use IP instead of hostname
</step>

<step name="diagnose-query-errors">
## Step 3b: Diagnose Query Errors

**Syntax errors:**
- Check SQL syntax against database version
- Verify table/column names exist
- Check for reserved words used as identifiers

**Constraint violations:**
```sql
-- Unique constraint violation
-- Find duplicates:
SELECT column, COUNT(*) FROM table GROUP BY column HAVING COUNT(*) > 1;

-- Foreign key violation
-- Check if referenced row exists:
SELECT * FROM parent_table WHERE id = [referenced_id];

-- Not null violation
-- Check for nulls:
SELECT * FROM table WHERE required_column IS NULL;
```

**Type mismatches:**
```sql
-- Check column types
SELECT column_name, data_type FROM information_schema.columns
WHERE table_name = 'table_name';

-- Common fixes:
-- Cast values: CAST(value AS integer)
-- Use correct format for dates: '2024-01-15'
```
</step>

<step name="diagnose-data-issues">
## Step 3c: Diagnose Data Integrity Issues

**Missing data:**
```sql
-- Check if data exists
SELECT COUNT(*) FROM table WHERE condition;

-- Check for orphaned records
SELECT c.* FROM child_table c
LEFT JOIN parent_table p ON c.parent_id = p.id
WHERE p.id IS NULL;

-- Check for soft deletes
SELECT * FROM table WHERE deleted_at IS NOT NULL;
```

**Duplicate data:**
```sql
-- Find duplicates
SELECT columns, COUNT(*)
FROM table
GROUP BY columns
HAVING COUNT(*) > 1;

-- Find and remove duplicates (keep oldest)
DELETE FROM table a USING table b
WHERE a.id > b.id AND a.unique_column = b.unique_column;
```

**Data corruption:**
```sql
-- PostgreSQL: Check for corruption
SELECT * FROM pg_catalog.pg_class WHERE relkind = 'r';

-- Verify constraints
ALTER TABLE table_name VALIDATE CONSTRAINT constraint_name;
```
</step>

<step name="diagnose-locking-issues">
## Step 3d: Diagnose Locking Issues

**PostgreSQL - Find locks:**
```sql
-- Blocked queries
SELECT blocked_locks.pid AS blocked_pid,
       blocked_activity.usename AS blocked_user,
       blocking_locks.pid AS blocking_pid,
       blocking_activity.usename AS blocking_user,
       blocked_activity.query AS blocked_statement,
       blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.relation = blocked_locks.relation
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;

-- Kill blocking query if needed
SELECT pg_terminate_backend(pid);
```

**MySQL - Find locks:**
```sql
-- Show locks
SELECT * FROM information_schema.INNODB_LOCKS;
SELECT * FROM information_schema.INNODB_LOCK_WAITS;

-- Kill blocking connection
KILL connection_id;
```

**MongoDB - Find locks:**
```javascript
// Current operations with locks
db.currentOp({ "waitingForLock": true })

// Kill operation
db.killOp(opid)
```

**Prevent future locking:**
- Keep transactions short
- Access tables in consistent order
- Use appropriate isolation levels
- Add indexes to reduce lock scope
</step>

<step name="apply-fix">
## Step 4: Apply the Fix

Based on diagnosis:

1. **Document the issue** - Root cause and symptoms
2. **Test fix in development** - Never fix directly in production
3. **Apply fix** - With rollback plan ready
4. **Verify resolution** - Confirm issue is resolved
5. **Monitor** - Watch for recurrence
</step>

<step name="prevent-recurrence">
## Step 5: Prevent Recurrence

Recommendations based on issue type:

**Connection issues:**
- Implement connection pooling
- Set appropriate timeouts
- Monitor connection counts

**Query errors:**
- Add input validation
- Use parameterized queries
- Add database constraints

**Data integrity:**
- Add foreign key constraints
- Add unique constraints
- Implement application-level validation

**Locking:**
- Review transaction boundaries
- Add appropriate indexes
- Consider optimistic locking
</step>

</process>

<emergency_procedures>
**Database is down:**
1. Check service status: `systemctl status postgresql`
2. Check disk space: `df -h`
3. Check logs: `/var/log/postgresql/`
4. Restart if safe: `systemctl restart postgresql`

**Database is unresponsive:**
1. Check active connections: `SELECT * FROM pg_stat_activity`
2. Identify blocking queries
3. Kill problematic connections if necessary
4. Investigate root cause after recovery

**Data corruption suspected:**
1. Stop writes immediately
2. Create backup of current state
3. Check for hardware issues
4. Run integrity checks
5. Restore from backup if needed
</emergency_procedures>

<anti_patterns>
Avoid:
- Making changes in production without testing
- Killing connections without understanding why they're stuck
- Ignoring the root cause after fixing symptoms
- Not having backups before attempting fixes
</anti_patterns>

<success_criteria>
Debugging is complete when:
- [ ] Issue categorized and understood
- [ ] Root cause identified
- [ ] Fix tested in non-production environment
- [ ] Fix applied and verified
- [ ] Prevention measures documented
- [ ] Monitoring in place for recurrence
</success_criteria>
