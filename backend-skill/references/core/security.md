<overview>
Backend security best practices covering OWASP top 10, input validation, rate limiting, and secure coding patterns. Security is not optional - every endpoint is a potential attack surface.
</overview>

<input_validation>
**Never Trust Client Input:**

```php
// BAD - SQL Injection
$users = DB::select("SELECT * FROM users WHERE id = " . $request->id);

// GOOD - Parameterized queries
$users = DB::select("SELECT * FROM users WHERE id = ?", [$request->id]);

// GOOD - ORM (escapes automatically)
$user = User::find($request->id);

// Validate EVERYTHING
$validated = $request->validate([
    'email' => 'required|email|max:255',
    'age' => 'required|integer|min:0|max:150',
    'url' => 'required|url',
    'file' => 'file|mimes:pdf,doc|max:10240',
]);
```

**Type Coercion:**

```php
// BAD - Type juggling attacks
if ($password == $userPassword) { ... }

// GOOD - Strict comparison
if (hash_equals($hashedPassword, $providedHash)) { ... }

// Always cast to expected types
$userId = (int) $request->input('user_id');
$isActive = (bool) $request->input('active');
```
</input_validation>

<sql_injection>
**Preventing SQL Injection:**

```php
// ALWAYS use parameterized queries
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
$stmt->execute(['email' => $email]);

// ORMs handle this automatically
User::where('email', $email)->first();

// Raw queries with bindings
DB::select('SELECT * FROM users WHERE id = ?', [$id]);

// NEVER concatenate user input into queries
// NEVER use user input in ORDER BY without whitelist
$allowedColumns = ['name', 'created_at', 'email'];
$orderBy = in_array($request->sort, $allowedColumns) ? $request->sort : 'created_at';
```
</sql_injection>

<xss_prevention>
**Cross-Site Scripting Prevention:**

```php
// Template engines escape by default
// Laravel Blade
{{ $userInput }}  // Escaped
{!! $trustedHtml !!}  // Unescaped (dangerous)

// Always escape dynamic content
$safe = htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');

// Content Security Policy header
Content-Security-Policy: default-src 'self'; script-src 'self'

// Sanitize rich text (use a library)
use HTMLPurifier;
$clean = $purifier->purify($dirtyHtml);
```
</xss_prevention>

<csrf_protection>
**CSRF Prevention:**

```php
// Laravel includes CSRF automatically
// Token in forms
<form method="POST">
    @csrf
    ...
</form>

// For AJAX requests
<meta name="csrf-token" content="{{ csrf_token() }}">

$.ajaxSetup({
    headers: { 'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content') }
});

// API routes typically exempt (use token auth instead)
// But protect cookie-authenticated APIs with CSRF
```

**SameSite Cookies:**

```php
// Prevent CSRF by restricting cookie sending
Set-Cookie: session=abc; SameSite=Strict; Secure; HttpOnly
```
</csrf_protection>

<rate_limiting>
**Rate Limiting:**

```php
// Laravel
Route::middleware('throttle:60,1')->group(function () {
    // 60 requests per minute
});

// Per-user throttling
Route::middleware('throttle:api')->group(function () {
    // Uses RateLimiter configured in RouteServiceProvider
});

// Custom rate limiters
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()
        ? Limit::perMinute(10)->by($request->user()->id)
        : Limit::perMinute(2)->by($request->ip());
});

// Response when limited
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
```
</rate_limiting>

<secrets_management>
**Secrets Management:**

```bash
# Environment variables (not in code)
DATABASE_PASSWORD=secret
API_KEY=abc123
STRIPE_SECRET=sk_live_xxx

# Never commit secrets
.env  # in .gitignore

# Use secret managers in production
- AWS Secrets Manager
- HashiCorp Vault
- Google Secret Manager
- Azure Key Vault
```

**Key Rotation:**

```php
// Support multiple keys during rotation
$keys = [
    'current' => env('ENCRYPTION_KEY'),
    'previous' => env('ENCRYPTION_KEY_OLD'),
];

// Decrypt with any valid key
foreach ($keys as $key) {
    try {
        return decrypt($value, $key);
    } catch (\Exception $e) {
        continue;
    }
}
```
</secrets_management>

<file_uploads>
**Secure File Uploads:**

```php
$request->validate([
    'file' => [
        'required',
        'file',
        'mimes:pdf,doc,docx',  // Whitelist extensions
        'max:10240',  // 10MB limit
    ],
]);

// Store outside web root
$path = $request->file('file')->store('uploads', 'private');

// Generate new filename (never use original)
$filename = Str::uuid() . '.' . $request->file('file')->extension();

// Validate file content (not just extension)
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mimeType = finfo_file($finfo, $file->path());
$allowed = ['application/pdf', 'image/jpeg', 'image/png'];
if (!in_array($mimeType, $allowed)) {
    abort(422, 'Invalid file type');
}
```
</file_uploads>

<headers>
**Security Headers:**

```php
// Middleware to add security headers
public function handle($request, $next)
{
    $response = $next($request);

    return $response
        // Prevent clickjacking
        ->header('X-Frame-Options', 'DENY')
        // Prevent MIME sniffing
        ->header('X-Content-Type-Options', 'nosniff')
        // Enable XSS filter
        ->header('X-XSS-Protection', '1; mode=block')
        // HTTPS only
        ->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains')
        // Content Security Policy
        ->header('Content-Security-Policy', "default-src 'self'")
        // Referrer policy
        ->header('Referrer-Policy', 'strict-origin-when-cross-origin');
}
```
</headers>

<logging>
**Security Logging:**

```php
// Log security events
Log::channel('security')->warning('Failed login attempt', [
    'email' => $request->email,
    'ip' => $request->ip(),
    'user_agent' => $request->userAgent(),
]);

// Audit trail for sensitive operations
Log::channel('audit')->info('User permissions changed', [
    'admin_id' => auth()->id(),
    'target_user_id' => $user->id,
    'old_roles' => $oldRoles,
    'new_roles' => $newRoles,
]);

// Never log sensitive data
// BAD
Log::info('User login', ['password' => $password]);
// GOOD
Log::info('User login', ['user_id' => $user->id]);
```
</logging>

<common_vulnerabilities>
**OWASP Top 10 (2021):**

1. **Broken Access Control** - Check permissions on every request
2. **Cryptographic Failures** - Use strong encryption, secure protocols
3. **Injection** - Parameterized queries, input validation
4. **Insecure Design** - Threat modeling, secure architecture
5. **Security Misconfiguration** - Secure defaults, remove debug
6. **Vulnerable Components** - Update dependencies regularly
7. **Authentication Failures** - MFA, rate limiting, secure sessions
8. **Software Integrity Failures** - Verify dependencies, CI/CD security
9. **Logging Failures** - Log security events, monitor anomalies
10. **SSRF** - Validate URLs, whitelist allowed hosts
</common_vulnerabilities>

<dependency_security>
**Dependency Security:**

```bash
# Check for vulnerabilities
composer audit
npm audit
pip-audit

# Automated updates
dependabot
renovate

# Lock file best practices
composer.lock  # Commit this
package-lock.json  # Commit this
```
</dependency_security>

<checklist>
**Security Checklist:**

- [ ] All user input validated and sanitized
- [ ] Parameterized queries everywhere
- [ ] CSRF tokens on state-changing requests
- [ ] Rate limiting on all endpoints
- [ ] Secrets in environment variables
- [ ] Security headers configured
- [ ] HTTPS enforced
- [ ] Password hashing (bcrypt/Argon2)
- [ ] Session security (httpOnly, secure, sameSite)
- [ ] File upload validation
- [ ] Security logging enabled
- [ ] Dependencies audited
- [ ] Error messages don't leak info
</checklist>
