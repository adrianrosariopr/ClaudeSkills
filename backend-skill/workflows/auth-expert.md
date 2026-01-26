# Workflow: Auth Expert

<required_reading>
**Read these reference files NOW:**
1. references/core/authentication.md
2. references/core/security.md
3. Framework-specific patterns file for the user's framework
</required_reading>

<context>
Expert guidance on authentication and authorization patterns including sessions, JWTs, OAuth, API tokens, and role-based access control.
</context>

<process>

<step name="understand-requirements">
Clarify authentication needs:

- What type of application? (SPA, mobile, traditional web, API)
- Need for social login? (Google, GitHub, etc.)
- Session-based or token-based?
- API authentication requirements?
- Role/permission complexity?
</step>

<step name="choose-strategy">
Select the right auth strategy:

**Session-Based** (traditional web apps):
- Server stores session data
- Cookie-based
- Easy to revoke

**JWT** (APIs, mobile apps):
- Stateless
- Can be verified by any service
- Self-contained claims

**OAuth 2.0** (social login, delegated access):
- "Login with Google/GitHub"
- Third-party integrations

**API Keys** (server-to-server):
- Simple but limited
- Good for service accounts
</step>

<step name="implement-authentication">
Implement based on framework:

**Laravel (Sanctum):**
```php
// SPA authentication
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens;
}

// Create token
$token = $user->createToken('api-token', ['read', 'write'])->plainTextToken;

// Protect routes
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user', fn (Request $request) => $request->user());
});
```

**Node.js (JWT):**
```typescript
import jwt from 'jsonwebtoken';

const generateToken = (userId: string) => {
    return jwt.sign({ sub: userId }, process.env.JWT_SECRET!, {
        expiresIn: '15m'
    });
};

const authMiddleware = (req, res, next) => {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return res.status(401).json({ error: 'No token' });

    try {
        const payload = jwt.verify(token, process.env.JWT_SECRET!);
        req.userId = payload.sub;
        next();
    } catch {
        res.status(401).json({ error: 'Invalid token' });
    }
};
```

**Python (FastAPI):**
```python
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Security(security),
    db: Session = Depends(get_db),
) -> User:
    token = credentials.credentials
    payload = decode_jwt(token)
    if not payload:
        raise HTTPException(401, "Invalid token")
    return await get_user(db, payload["sub"])
```
</step>

<step name="implement-authorization">
Add role-based access control:

**Laravel (Spatie):**
```php
// Define permissions
Permission::create(['name' => 'posts.edit']);

// Create roles
$editor = Role::create(['name' => 'editor']);
$editor->givePermissionTo('posts.edit');

// Assign to user
$user->assignRole('editor');

// Check permissions (NOT roles)
if ($user->can('posts.edit')) { ... }

// Middleware
Route::middleware('permission:posts.edit')->get('/posts/{post}/edit', ...);
```

**Policy Pattern:**
```php
class PostPolicy
{
    public function update(User $user, Post $post): bool
    {
        return $user->can('posts.edit') || $user->id === $post->user_id;
    }
}
```
</step>

<step name="secure-passwords">
Never store plain passwords:

```php
// Hash on registration
$user->password = Hash::make($request->password);

// Verify on login
if (!Hash::check($request->password, $user->password)) {
    throw ValidationException::withMessages([
        'email' => ['Invalid credentials.'],
    ]);
}
```

Use bcrypt or Argon2id (never MD5/SHA1 for passwords).
</step>

<step name="implement-refresh-tokens">
For JWT, implement refresh token flow:

```
Access Token: Short-lived (15 min)
Refresh Token: Long-lived (7 days), stored securely

Flow:
1. User logs in → receive access + refresh tokens
2. Access token in memory (not localStorage)
3. Refresh token in httpOnly cookie
4. When access token expires → use refresh token to get new pair
5. Revoke refresh token on logout
```
</step>

<step name="add-security-measures">
Additional security:

```php
// Rate limit login attempts
RateLimiter::for('login', function (Request $request) {
    return Limit::perMinute(5)->by($request->ip());
});

// Lockout after failed attempts
if ($this->hasTooManyLoginAttempts($request)) {
    $this->fireLockoutEvent($request);
    return $this->sendLockoutResponse($request);
}

// CSRF protection
@csrf  // In forms

// Secure cookie settings
'secure' => true,
'http_only' => true,
'same_site' => 'strict',
```
</step>

<step name="verify">
Test authentication:

```bash
# Test login
curl -X POST /api/login -d '{"email": "...", "password": "..."}'

# Test protected route without token
curl /api/user  # Should return 401

# Test with token
curl /api/user -H "Authorization: Bearer <token>"

# Test expired token
# Should return 401

# Test permission denied
# Should return 403
```
</step>

</process>

<anti_patterns>
Avoid:
- Storing plain text passwords
- Using MD5/SHA1 for passwords
- JWTs in localStorage (use httpOnly cookies for refresh)
- Long-lived access tokens
- Checking role names instead of permissions
- Missing rate limiting on login
- Not logging auth events
</anti_patterns>

<success_criteria>
Secure authentication:
- Passwords hashed with bcrypt/Argon2
- Short-lived access tokens (if JWT)
- Refresh tokens stored securely
- Rate limiting on login endpoints
- Permissions checked (not roles)
- Secure cookie settings (httpOnly, secure, sameSite)
- Auth events logged for auditing
- All tests passing
</success_criteria>
