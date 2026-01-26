# Workflow: API Expert

<required_reading>
**Read these reference files NOW:**
1. references/core/api-design.md
2. references/core/security.md
3. Framework-specific patterns file for the user's framework
</required_reading>

<context>
Expert guidance on API design, REST principles, GraphQL, versioning, documentation, and error handling. Good APIs are predictable, consistent, and developer-friendly.
</context>

<process>

<step name="understand-requirements">
Clarify the API requirements:

- What resources need to be exposed?
- Who are the consumers? (Web app, mobile, third-party)
- What operations are needed? (CRUD, custom actions)
- Authentication requirements?
- Rate limiting needs?
- Versioning strategy?
</step>

<step name="design-resources">
Design resource structure:

```
# Core resources (nouns, not verbs)
/users
/posts
/orders

# Nested for relationships
/users/{id}/posts
/orders/{id}/items

# Filter with query params
/posts?status=published&author=123
```

Map HTTP methods to operations:
- GET: Read (safe, idempotent)
- POST: Create
- PUT: Replace
- PATCH: Partial update
- DELETE: Remove
</step>

<step name="define-response-format">
Establish consistent response structure:

```json
// Success
{
  "data": { ... },
  "meta": { ... }
}

// Error
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human readable message",
    "details": [...]
  }
}

// Collection
{
  "data": [...],
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 100
  }
}
```
</step>

<step name="implement-endpoints">
Implement with framework patterns:

**Laravel:**
```php
Route::apiResource('posts', PostController::class);
```

**Node.js (Express):**
```typescript
router.get('/posts', postsController.list);
router.post('/posts', validate(createPostSchema), postsController.create);
```

**Python (FastAPI):**
```python
@router.get("/posts", response_model=PostList)
async def list_posts(page: int = 1, limit: int = 20):
    ...
```

**Go (Gin):**
```go
api.GET("/posts", handlers.ListPosts)
api.POST("/posts", handlers.CreatePost)
```
</step>

<step name="add-validation">
Validate all input:

- Required fields
- Data types (string, int, email)
- Length limits
- Custom business rules
- Return 422 for validation errors
</step>

<step name="implement-error-handling">
Consistent error responses:

- 400: Malformed request
- 401: Not authenticated
- 403: Not authorized
- 404: Resource not found
- 422: Validation failed
- 429: Rate limited
- 500: Server error
</step>

<step name="add-documentation">
Document with OpenAPI/Swagger:

- All endpoints
- Request/response schemas
- Error responses
- Authentication requirements
- Example requests
</step>

<step name="verify">
Test the API:

```bash
# Test endpoints
curl -X GET http://localhost:8000/api/posts
curl -X POST http://localhost:8000/api/posts -H "Content-Type: application/json" -d '{"title": "Test"}'

# Verify error handling
curl -X GET http://localhost:8000/api/posts/999  # 404
curl -X POST http://localhost:8000/api/posts -d '{}'  # 422

# Check response format consistency
```
</step>

</process>

<anti_patterns>
Avoid:
- Verbs in URLs (`/getUsers` instead of `/users`)
- Inconsistent response formats
- Exposing internal errors to clients
- Missing pagination on collections
- No rate limiting
- Breaking changes without versioning
</anti_patterns>

<success_criteria>
A well-designed API:
- Uses consistent resource naming (nouns, plural)
- Proper HTTP methods and status codes
- Consistent response structure
- Comprehensive validation
- Meaningful error messages (without leaking internals)
- Documentation (OpenAPI)
- Rate limiting enabled
- Versioning strategy defined
</success_criteria>
