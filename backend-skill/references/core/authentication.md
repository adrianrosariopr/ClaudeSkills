<overview>
Authentication (who are you?) and authorization (what can you do?) patterns. Security-critical code that must be implemented correctly.
</overview>

<auth_patterns>
**Session-Based Authentication:**

```
Client                    Server                    Session Store
  |                         |                           |
  |-- POST /login --------->|                           |
  |   {email, password}     |                           |
  |                         |-- Store session --------->|
  |                         |   {user_id, expires}      |
  |<-- Set-Cookie: sid -----|                           |
  |                         |                           |
  |-- GET /profile -------->|                           |
  |   Cookie: sid           |-- Lookup session -------->|
  |                         |<-- {user_id} -------------|
  |<-- 200 {user data} -----|                           |
```

**Pros:** Simple, revocable, works well with browsers
**Cons:** Requires session storage, harder to scale, CSRF concerns

**Token-Based (JWT):**

```
Client                    Server
  |                         |
  |-- POST /login --------->|
  |   {email, password}     |
  |                         |-- Generate JWT
  |<-- {token: "eyJ..."} ---|   (signed, contains claims)
  |                         |
  |-- GET /profile -------->|
  |   Authorization: Bearer |-- Verify signature
  |   eyJ...                |-- Check expiration
  |<-- 200 {user data} -----|-- Extract claims
```

**Pros:** Stateless, works across services, mobile-friendly
**Cons:** Cannot revoke (until expiry), token size, must secure storage
</auth_patterns>

<jwt_implementation>
**JWT Structure:**

```
Header.Payload.Signature

Header:
{
  "alg": "RS256",
  "typ": "JWT"
}

Payload:
{
  "sub": "123",           // Subject (user ID)
  "email": "user@example.com",
  "role": "admin",
  "iat": 1704067200,      // Issued at
  "exp": 1704153600       // Expires
}

Signature:
RSASHA256(base64(header) + "." + base64(payload), privateKey)
```

**Best Practices:**

```php
// Short-lived access tokens (15-60 min)
$accessToken = JWT::encode([
    'sub' => $user->id,
    'exp' => time() + 900,  // 15 minutes
], $privateKey, 'RS256');

// Longer-lived refresh tokens (7-30 days)
$refreshToken = JWT::encode([
    'sub' => $user->id,
    'type' => 'refresh',
    'exp' => time() + 604800,  // 7 days
], $privateKey, 'RS256');

// Store refresh token in httpOnly cookie
// Store access token in memory (not localStorage)
```

**Never put in JWT:**
- Passwords
- Credit card numbers
- Sensitive PII
- Large amounts of data
</jwt_implementation>

<oauth2>
**OAuth 2.0 Flows:**

**Authorization Code (web apps):**
```
1. Redirect to provider
   /authorize?client_id=X&redirect_uri=Y&scope=email&state=random

2. User logs in, grants permission

3. Provider redirects back with code
   /callback?code=AUTH_CODE&state=random

4. Exchange code for tokens (server-side)
   POST /token
   {grant_type: "authorization_code", code: AUTH_CODE, client_secret: SECRET}

5. Receive access_token, refresh_token
```

**PKCE (mobile/SPA - no client secret):**
```
1. Generate code_verifier (random string)
2. Generate code_challenge = SHA256(code_verifier)
3. Include code_challenge in authorize request
4. Include code_verifier in token exchange
```

**Client Credentials (server-to-server):**
```
POST /token
{
  grant_type: "client_credentials",
  client_id: "...",
  client_secret: "..."
}
```
</oauth2>

<password_security>
**Password Handling:**

```php
// Always hash passwords (bcrypt or Argon2)
$hash = password_hash($password, PASSWORD_ARGON2ID, [
    'memory_cost' => 65536,
    'time_cost' => 4,
    'threads' => 3,
]);

// Verify
if (password_verify($inputPassword, $storedHash)) {
    // Success
}

// Password requirements
- Minimum 8 characters (NIST recommends 8+)
- Check against breach databases (HaveIBeenPwned)
- Don't require special characters (encourages bad patterns)
- Allow long passphrases
```

**Never:**
- Store plain text passwords
- Use MD5/SHA1 for passwords
- Roll your own crypto
- Email passwords
</password_security>

<authorization_patterns>
**Role-Based Access Control (RBAC):**

```php
// Roles have permissions
$adminRole->givePermissionTo(['users.create', 'users.delete']);
$editorRole->givePermissionTo(['posts.create', 'posts.edit']);

// Users have roles
$user->assignRole('editor');

// Check permission (NOT role)
if ($user->can('posts.edit')) {
    // Allow action
}
```

**Attribute-Based Access Control (ABAC):**

```php
// More flexible - based on attributes
function canEditPost(User $user, Post $post): bool
{
    // Owner can edit
    if ($post->user_id === $user->id) return true;

    // Admin can edit
    if ($user->hasRole('admin')) return true;

    // Editor can edit if in same organization
    if ($user->can('posts.edit') && $user->org_id === $post->org_id) return true;

    return false;
}
```

**Policy Pattern:**

```php
class PostPolicy
{
    public function view(User $user, Post $post): bool
    {
        return $post->published || $user->id === $post->user_id;
    }

    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->hasRole('admin');
    }
}
```
</authorization_patterns>

<api_authentication>
**API Key Authentication:**

```php
// Simple but limited - good for server-to-server
// Store hashed, compare with timing-safe function

$apiKey = request()->header('X-API-Key');
$hashedKey = hash('sha256', $apiKey);

$token = ApiToken::where('hashed_key', $hashedKey)
    ->where('expires_at', '>', now())
    ->first();

if (!$token) {
    return response()->json(['error' => 'Invalid API key'], 401);
}
```

**Token Abilities (Scopes):**

```php
// Laravel Sanctum
$token = $user->createToken('api', ['posts:read', 'posts:write']);

// Check ability
if ($request->user()->tokenCan('posts:write')) {
    // Allow write
}
```
</api_authentication>

<security_headers>
**Authentication-Related Headers:**

```
# HTTPS only
Strict-Transport-Security: max-age=31536000; includeSubDomains

# Prevent MIME sniffing
X-Content-Type-Options: nosniff

# XSS protection
Content-Security-Policy: default-src 'self'

# Cookie security
Set-Cookie: session=abc; HttpOnly; Secure; SameSite=Strict
```
</security_headers>

<decision_tree>
**Session vs JWT:**

Use Sessions when:
- Traditional web app with server-rendered pages
- Need immediate revocation
- Don't need to share auth across services
- Simpler security model

Use JWT when:
- API-first architecture
- Multiple services need to verify auth
- Mobile apps
- Microservices

**OAuth vs Custom:**

Use OAuth when:
- "Login with Google/GitHub" social auth
- Third-party integrations
- Delegated access to resources

Use Custom when:
- Simple email/password auth
- Full control over user data
- No third-party dependencies needed
</decision_tree>
