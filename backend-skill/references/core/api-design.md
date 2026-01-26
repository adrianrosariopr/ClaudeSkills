<overview>
REST API design principles, GraphQL patterns, versioning strategies, and error handling. Good API design makes systems predictable, maintainable, and developer-friendly.
</overview>

<rest_principles>
**Resource-Oriented Design:**

```
# Resources are nouns, not verbs
GET    /users           # List users
GET    /users/123       # Get user 123
POST   /users           # Create user
PUT    /users/123       # Replace user 123
PATCH  /users/123       # Partially update user 123
DELETE /users/123       # Delete user 123

# Nested resources for relationships
GET    /users/123/posts         # User's posts
POST   /users/123/posts         # Create post for user
GET    /users/123/posts/456     # Specific post

# Query parameters for filtering/pagination
GET    /posts?status=published&author=123
GET    /posts?page=2&limit=20
GET    /posts?sort=-created_at  # Descending
```

**HTTP Methods:**

| Method | Idempotent | Safe | Use Case |
|--------|------------|------|----------|
| GET | Yes | Yes | Retrieve data |
| POST | No | No | Create resource |
| PUT | Yes | No | Replace resource |
| PATCH | Yes | No | Partial update |
| DELETE | Yes | No | Remove resource |

**Use HTTP semantics correctly:**
- GET requests should never modify data
- PUT replaces the entire resource
- PATCH updates only provided fields
- DELETE is idempotent (deleting twice = same result)
</rest_principles>

<response_format>
**Consistent Response Structure:**

```json
// Success response
{
  "data": {
    "id": 123,
    "type": "user",
    "attributes": {
      "name": "John Doe",
      "email": "john@example.com"
    }
  },
  "meta": {
    "request_id": "abc-123"
  }
}

// Collection response
{
  "data": [
    { "id": 1, "name": "Item 1" },
    { "id": 2, "name": "Item 2" }
  ],
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 100,
    "total_pages": 5
  },
  "links": {
    "first": "/items?page=1",
    "prev": null,
    "next": "/items?page=2",
    "last": "/items?page=5"
  }
}

// Error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The given data was invalid.",
    "details": [
      { "field": "email", "message": "Email is required" },
      { "field": "password", "message": "Password must be at least 8 characters" }
    ]
  },
  "meta": {
    "request_id": "abc-123"
  }
}
```
</response_format>

<error_handling>
**Error Response Best Practices:**

```json
// Use appropriate HTTP status codes
400 - Bad Request (malformed JSON, missing fields)
401 - Unauthorized (no/invalid auth token)
403 - Forbidden (valid token but no permission)
404 - Not Found (resource doesn't exist)
409 - Conflict (duplicate email, race condition)
422 - Unprocessable Entity (valid JSON but semantic error)
429 - Too Many Requests (rate limited)
500 - Internal Server Error (unexpected failure)

// Include machine-readable error codes
{
  "error": {
    "code": "USER_NOT_FOUND",        // Machine-readable
    "message": "User not found",     // Human-readable
    "details": {
      "user_id": 123
    }
  }
}

// Never expose stack traces in production
// Log full details server-side, return safe message to client
```

**Error Code Naming:**
```
VALIDATION_ERROR
RESOURCE_NOT_FOUND
UNAUTHORIZED
FORBIDDEN
RATE_LIMIT_EXCEEDED
DUPLICATE_RESOURCE
INTERNAL_ERROR
```
</error_handling>

<versioning>
**API Versioning Strategies:**

**URL Path (Recommended):**
```
/api/v1/users
/api/v2/users
```
- Clear and explicit
- Easy to route
- Can run multiple versions simultaneously

**Query Parameter:**
```
/api/users?version=1
/api/users?version=2
```
- Less common
- Harder to cache

**Header:**
```
Accept: application/vnd.api+json; version=1
```
- Clean URLs
- Harder to test in browser

**When to version:**
- Breaking changes to response format
- Removing fields from response
- Changing field types
- Removing endpoints
- NOT for adding new optional fields (backwards compatible)
</versioning>

<pagination>
**Pagination Patterns:**

**Offset-based (simple):**
```
GET /posts?page=2&limit=20

Response:
{
  "data": [...],
  "meta": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "total_pages": 8
  }
}
```
- Easy to implement
- Can jump to any page
- Inconsistent with real-time data (items shift)

**Cursor-based (better for large datasets):**
```
GET /posts?cursor=eyJpZCI6MTAwfQ&limit=20

Response:
{
  "data": [...],
  "meta": {
    "next_cursor": "eyJpZCI6MTIwfQ",
    "has_more": true
  }
}
```
- Consistent results
- Better performance on large tables
- Cannot jump to arbitrary page
</pagination>

<graphql_patterns>
**GraphQL Best Practices:**

```graphql
# Define clear types
type User {
  id: ID!
  email: String!
  name: String
  posts(first: Int, after: String): PostConnection!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}

# Mutations return the affected object
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}

type CreateUserPayload {
  user: User
  errors: [UserError!]
}
```

**Avoid:**
- Deeply nested queries (max depth limit)
- Unbounded lists (always paginate)
- Exposing internal IDs directly
</graphql_patterns>

<openapi>
**OpenAPI/Swagger Documentation:**

```yaml
openapi: 3.1.0
info:
  title: My API
  version: 1.0.0

paths:
  /users:
    get:
      summary: List all users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
    post:
      summary: Create a user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUser'
      responses:
        '201':
          description: Created
        '422':
          description: Validation error

components:
  schemas:
    User:
      type: object
      required: [id, email]
      properties:
        id:
          type: integer
        email:
          type: string
          format: email
        name:
          type: string
```
</openapi>

<decision_tree>
**REST vs GraphQL:**

Use REST when:
- Simple CRUD operations
- Public API with many consumers
- Caching is critical (HTTP caching)
- Team is familiar with REST

Use GraphQL when:
- Complex, nested data requirements
- Multiple frontends with different data needs
- Rapid iteration on data requirements
- Real-time subscriptions needed
</decision_tree>
