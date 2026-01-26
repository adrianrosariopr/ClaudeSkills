# Workflow: Debug Backend

<required_reading>
**Read these reference files NOW:**
1. references/core/anti-patterns.md
2. references/core/performance.md
3. Framework-specific patterns file for the user's framework
</required_reading>

<context>
Systematic approach to debugging backend issues including 500 errors, performance problems, authentication failures, and unexpected behavior.
</context>

<process>

<step name="gather-information">
Collect information about the issue:

- What is the exact error message?
- When did it start happening?
- Is it reproducible? Steps to reproduce?
- What changed recently? (deploys, config changes)
- Does it affect all users or specific ones?
- Is it intermittent or consistent?
</step>

<step name="check-logs">
Review application logs:

**Laravel:**
```bash
# Application logs
tail -f storage/logs/laravel.log

# With context
grep -A 10 "ERROR" storage/logs/laravel.log

# Specific request
grep "request_id" storage/logs/laravel.log
```

**Node.js:**
```bash
# PM2 logs
pm2 logs

# Docker logs
docker logs -f container_name
```

**Python:**
```bash
# Django
tail -f logs/django.log

# Uvicorn
uvicorn app.main:app --log-level debug
```

Look for:
- Stack traces
- Error messages
- Request/response details
- Timestamps of failures
</step>

<step name="reproduce-locally">
Reproduce the issue in development:

```bash
# Same request that's failing
curl -X POST http://localhost:8000/api/endpoint \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"field": "value"}' \
  -v  # Verbose output

# Check response headers and body
```

Enable debug mode:
```php
// Laravel .env
APP_DEBUG=true

// Django settings
DEBUG = True

// Node.js
NODE_ENV=development
```
</step>

<step name="isolate-the-problem">
Narrow down the cause:

**Is it the database?**
```sql
-- Check for slow queries
SHOW FULL PROCESSLIST;

-- PostgreSQL
SELECT * FROM pg_stat_activity;

-- Check for locks
SELECT * FROM pg_locks WHERE NOT granted;
```

**Is it authentication?**
```php
// Log auth details
Log::info('Auth check', [
    'user' => auth()->user(),
    'token' => request()->bearerToken(),
    'guard' => auth()->guard()->name,
]);
```

**Is it a specific service?**
```php
// Add logging around suspect code
Log::info('Before service call', ['input' => $input]);
try {
    $result = $service->process($input);
    Log::info('After service call', ['result' => $result]);
} catch (\Exception $e) {
    Log::error('Service failed', [
        'error' => $e->getMessage(),
        'trace' => $e->getTraceAsString(),
    ]);
    throw $e;
}
```

**Is it the request data?**
```php
Log::info('Request data', [
    'all' => $request->all(),
    'headers' => $request->headers->all(),
    'method' => $request->method(),
    'url' => $request->url(),
]);
```
</step>

<step name="common-issues">
Check for common problems:

**500 Internal Server Error:**
- Missing environment variable
- Database connection failure
- Permission issues (storage, logs)
- Missing dependency
- Syntax error in code

**401 Unauthorized:**
- Token expired or invalid
- Wrong guard configured
- Missing auth middleware
- CORS blocking preflight

**403 Forbidden:**
- Permission not assigned
- Policy returning false
- CSRF token mismatch

**404 Not Found:**
- Route not registered
- Wrong HTTP method
- Model not found (route model binding)

**422 Validation Error:**
- Check validation rules
- Review input data format

**N+1 Query (slow response):**
- Enable query logging
- Check for missing eager loads
</step>

<step name="check-configuration">
Verify configuration:

```bash
# Laravel
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear

# Check config values
php artisan tinker
>>> config('app.key')
>>> config('database.default')

# Node.js
echo $NODE_ENV
echo $DATABASE_URL

# Check .env is loaded
console.log(process.env.DATABASE_URL);
```
</step>

<step name="check-database">
Verify database state:

```bash
# Laravel tinker
php artisan tinker
>>> User::find(1)
>>> DB::connection()->getDatabaseName()

# Run raw query
>>> DB::select('SELECT * FROM users WHERE id = 1')

# Check migrations
php artisan migrate:status
```
</step>

<step name="use-debugging-tools">
Use appropriate debugging tools:

**Laravel:**
```bash
# Telescope (development)
# Shows requests, queries, logs, mail, etc.
php artisan telescope:install

# Debugbar
composer require barryvdh/laravel-debugbar --dev
```

**Node.js:**
```bash
# Node debugger
node --inspect app.js
# Connect Chrome DevTools

# Log all queries (Prisma)
const prisma = new PrismaClient({
    log: ['query', 'info', 'warn', 'error'],
});
```

**Python:**
```python
# Django Debug Toolbar
INSTALLED_APPS += ['debug_toolbar']

# pdb debugger
import pdb; pdb.set_trace()
```
</step>

<step name="fix-and-verify">
Apply fix and verify:

1. Make the fix
2. Clear caches
3. Run tests
4. Manually test the scenario
5. Check logs for new errors
6. Deploy to staging first if significant change

```bash
# Clear all caches
php artisan optimize:clear

# Run tests
php artisan test

# Check logs after fix
tail -f storage/logs/laravel.log
```
</step>

</process>

<debugging_checklist>
**Quick Checklist:**

- [ ] Read the full error message and stack trace
- [ ] Check application logs
- [ ] Reproduce locally
- [ ] Enable debug mode
- [ ] Check recent code changes (git log, git diff)
- [ ] Verify environment variables
- [ ] Check database connectivity
- [ ] Clear caches
- [ ] Check file permissions
- [ ] Review middleware order
- [ ] Test in isolation
</debugging_checklist>

<anti_patterns>
Avoid:
- Making random changes hoping something works
- Ignoring stack traces
- Not checking logs first
- Debugging in production without caution
- Removing error handling to "see what happens"
- Not reproducing before fixing
</anti_patterns>

<success_criteria>
Issue is resolved when:
- Root cause identified
- Fix applied and tested
- No new errors in logs
- All related tests pass
- Fix deployed and monitored
</success_criteria>
