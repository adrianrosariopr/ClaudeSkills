<overview>
Go backend patterns covering Gin, Echo, GORM, and Go idioms. Focus on building high-performance, concurrent APIs with Go's simplicity.
</overview>

<project_structure>
**Standard Go Project Layout:**

```
project/
├── cmd/
│   └── api/
│       └── main.go        # Entry point
├── internal/              # Private application code
│   ├── handlers/          # HTTP handlers
│   ├── services/          # Business logic
│   ├── repository/        # Data access
│   ├── models/            # Domain models
│   ├── middleware/        # HTTP middleware
│   └── config/            # Configuration
├── pkg/                   # Public libraries
├── migrations/
├── go.mod
├── go.sum
└── Makefile
```
</project_structure>

<gin_patterns>
**Gin Setup:**

```go
package main

import (
    "github.com/gin-gonic/gin"
    "github.com/gin-contrib/cors"
)

func main() {
    r := gin.Default()

    // Middleware
    r.Use(cors.Default())
    r.Use(gin.Logger())
    r.Use(gin.Recovery())

    // Routes
    api := r.Group("/api/v1")
    {
        users := api.Group("/users")
        {
            users.GET("", handlers.ListUsers)
            users.POST("", handlers.CreateUser)
            users.GET("/:id", handlers.GetUser)
            users.PUT("/:id", handlers.UpdateUser)
            users.DELETE("/:id", handlers.DeleteUser)
        }
    }

    r.Run(":8080")
}
```

**Handlers:**

```go
package handlers

import (
    "net/http"
    "strconv"

    "github.com/gin-gonic/gin"
)

type UserHandler struct {
    service *services.UserService
}

func NewUserHandler(service *services.UserService) *UserHandler {
    return &UserHandler{service: service}
}

func (h *UserHandler) List(c *gin.Context) {
    page, _ := strconv.Atoi(c.DefaultQuery("page", "1"))
    limit, _ := strconv.Atoi(c.DefaultQuery("limit", "20"))

    users, total, err := h.service.List(c, page, limit)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "data":  users,
        "total": total,
        "page":  page,
    })
}

func (h *UserHandler) Create(c *gin.Context) {
    var input CreateUserInput
    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    user, err := h.service.Create(c, input)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusCreated, user)
}

func (h *UserHandler) Get(c *gin.Context) {
    id, err := strconv.ParseUint(c.Param("id"), 10, 64)
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid id"})
        return
    }

    user, err := h.service.Get(c, uint(id))
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            c.JSON(http.StatusNotFound, gin.H{"error": "user not found"})
            return
        }
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, user)
}
```
</gin_patterns>

<echo_patterns>
**Echo Alternative:**

```go
package main

import (
    "github.com/labstack/echo/v4"
    "github.com/labstack/echo/v4/middleware"
)

func main() {
    e := echo.New()

    // Middleware
    e.Use(middleware.Logger())
    e.Use(middleware.Recover())
    e.Use(middleware.CORS())

    // Routes
    api := e.Group("/api/v1")
    api.GET("/users", handlers.ListUsers)
    api.POST("/users", handlers.CreateUser)

    e.Logger.Fatal(e.Start(":8080"))
}

// Handler with context
func (h *UserHandler) Create(c echo.Context) error {
    var input CreateUserInput
    if err := c.Bind(&input); err != nil {
        return echo.NewHTTPError(http.StatusBadRequest, err.Error())
    }

    if err := c.Validate(&input); err != nil {
        return echo.NewHTTPError(http.StatusUnprocessableEntity, err.Error())
    }

    user, err := h.service.Create(c.Request().Context(), input)
    if err != nil {
        return err
    }

    return c.JSON(http.StatusCreated, user)
}
```
</echo_patterns>

<gorm>
**GORM Models:**

```go
package models

import (
    "time"
    "gorm.io/gorm"
)

type User struct {
    ID        uint           `gorm:"primaryKey" json:"id"`
    Email     string         `gorm:"uniqueIndex;not null" json:"email"`
    Password  string         `gorm:"-" json:"-"`                          // Excluded from DB
    PassHash  string         `gorm:"column:password" json:"-"`
    Name      string         `json:"name"`
    Posts     []Post         `gorm:"foreignKey:AuthorID" json:"posts,omitempty"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"-"`
}

type Post struct {
    ID        uint      `gorm:"primaryKey" json:"id"`
    Title     string    `gorm:"not null" json:"title"`
    Content   string    `json:"content"`
    Published bool      `gorm:"default:false" json:"published"`
    AuthorID  uint      `gorm:"index" json:"author_id"`
    Author    *User     `json:"author,omitempty"`
    CreatedAt time.Time `json:"created_at"`
}
```

**GORM Queries:**

```go
package repository

import (
    "context"
    "gorm.io/gorm"
)

type UserRepository struct {
    db *gorm.DB
}

func (r *UserRepository) FindByID(ctx context.Context, id uint) (*User, error) {
    var user User
    err := r.db.WithContext(ctx).First(&user, id).Error
    if err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, ErrNotFound
        }
        return nil, err
    }
    return &user, nil
}

func (r *UserRepository) FindAll(ctx context.Context, page, limit int) ([]User, int64, error) {
    var users []User
    var total int64

    offset := (page - 1) * limit

    err := r.db.WithContext(ctx).
        Model(&User{}).
        Count(&total).
        Error
    if err != nil {
        return nil, 0, err
    }

    err = r.db.WithContext(ctx).
        Preload("Posts", func(db *gorm.DB) *gorm.DB {
            return db.Where("published = ?", true).Limit(5)
        }).
        Offset(offset).
        Limit(limit).
        Find(&users).
        Error

    return users, total, err
}

func (r *UserRepository) Create(ctx context.Context, user *User) error {
    return r.db.WithContext(ctx).Create(user).Error
}

// Transaction
func (r *UserRepository) CreateWithProfile(ctx context.Context, user *User, profile *Profile) error {
    return r.db.WithContext(ctx).Transaction(func(tx *gorm.DB) error {
        if err := tx.Create(user).Error; err != nil {
            return err
        }
        profile.UserID = user.ID
        return tx.Create(profile).Error
    })
}
```
</gorm>

<validation>
**Validation with go-playground/validator:**

```go
package handlers

import (
    "github.com/go-playground/validator/v10"
)

var validate = validator.New()

type CreateUserInput struct {
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=8"`
    Name     string `json:"name" validate:"max=100"`
}

func (h *UserHandler) Create(c *gin.Context) {
    var input CreateUserInput
    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    if err := validate.Struct(&input); err != nil {
        errors := err.(validator.ValidationErrors)
        c.JSON(http.StatusUnprocessableEntity, gin.H{
            "errors": formatValidationErrors(errors),
        })
        return
    }

    // Process...
}

func formatValidationErrors(errors validator.ValidationErrors) []map[string]string {
    var result []map[string]string
    for _, err := range errors {
        result = append(result, map[string]string{
            "field":   err.Field(),
            "message": err.Tag(),
        })
    }
    return result
}
```
</validation>

<middleware>
**Custom Middleware:**

```go
package middleware

import (
    "strings"
    "github.com/gin-gonic/gin"
)

func AuthMiddleware(jwtSecret string) gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if !strings.HasPrefix(authHeader, "Bearer ") {
            c.AbortWithStatusJSON(401, gin.H{"error": "unauthorized"})
            return
        }

        token := strings.TrimPrefix(authHeader, "Bearer ")
        claims, err := ValidateJWT(token, jwtSecret)
        if err != nil {
            c.AbortWithStatusJSON(401, gin.H{"error": "invalid token"})
            return
        }

        c.Set("userID", claims.UserID)
        c.Next()
    }
}

func RateLimitMiddleware(limit int, window time.Duration) gin.HandlerFunc {
    store := make(map[string]*rateLimiter)
    mu := sync.Mutex{}

    return func(c *gin.Context) {
        ip := c.ClientIP()

        mu.Lock()
        limiter, exists := store[ip]
        if !exists {
            limiter = newRateLimiter(limit, window)
            store[ip] = limiter
        }
        mu.Unlock()

        if !limiter.Allow() {
            c.AbortWithStatusJSON(429, gin.H{"error": "rate limit exceeded"})
            return
        }

        c.Next()
    }
}
```
</middleware>

<concurrency>
**Go Concurrency Patterns:**

```go
// Worker pool
func processItems(ctx context.Context, items []Item) error {
    numWorkers := runtime.NumCPU()
    jobs := make(chan Item, len(items))
    results := make(chan error, len(items))

    // Start workers
    var wg sync.WaitGroup
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for item := range jobs {
                results <- processItem(ctx, item)
            }
        }()
    }

    // Send jobs
    for _, item := range items {
        jobs <- item
    }
    close(jobs)

    // Wait and collect errors
    go func() {
        wg.Wait()
        close(results)
    }()

    var errs []error
    for err := range results {
        if err != nil {
            errs = append(errs, err)
        }
    }

    if len(errs) > 0 {
        return fmt.Errorf("processing errors: %v", errs)
    }
    return nil
}

// errgroup for concurrent operations
import "golang.org/x/sync/errgroup"

func fetchAll(ctx context.Context) (*Data, error) {
    g, ctx := errgroup.WithContext(ctx)

    var users []User
    var posts []Post

    g.Go(func() error {
        var err error
        users, err = fetchUsers(ctx)
        return err
    })

    g.Go(func() error {
        var err error
        posts, err = fetchPosts(ctx)
        return err
    })

    if err := g.Wait(); err != nil {
        return nil, err
    }

    return &Data{Users: users, Posts: posts}, nil
}
```
</concurrency>

<testing>
**Testing:**

```go
package handlers_test

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/gin-gonic/gin"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
)

func TestCreateUser(t *testing.T) {
    gin.SetMode(gin.TestMode)

    mockService := new(MockUserService)
    handler := NewUserHandler(mockService)

    mockService.On("Create", mock.Anything, mock.AnythingOfType("CreateUserInput")).
        Return(&User{ID: 1, Email: "test@example.com"}, nil)

    router := gin.New()
    router.POST("/users", handler.Create)

    body := `{"email": "test@example.com", "password": "password123"}`
    req := httptest.NewRequest("POST", "/users", bytes.NewBufferString(body))
    req.Header.Set("Content-Type", "application/json")
    w := httptest.NewRecorder()

    router.ServeHTTP(w, req)

    assert.Equal(t, http.StatusCreated, w.Code)

    var response User
    json.Unmarshal(w.Body.Bytes(), &response)
    assert.Equal(t, "test@example.com", response.Email)

    mockService.AssertExpectations(t)
}

// Table-driven tests
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name     string
        email    string
        expected bool
    }{
        {"valid email", "test@example.com", true},
        {"invalid email", "not-an-email", false},
        {"empty email", "", false},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := ValidateEmail(tt.email)
            assert.Equal(t, tt.expected, result)
        })
    }
}
```
</testing>

<quick_reference>
**Common Commands:**

```bash
# Build & Run
go run cmd/api/main.go
go build -o bin/api cmd/api/main.go
./bin/api

# Testing
go test ./...
go test -v -cover ./...
go test -race ./...

# Dependencies
go mod init project
go mod tidy
go get github.com/gin-gonic/gin

# Database migrations (golang-migrate)
migrate -path migrations -database "postgres://..." up
migrate create -ext sql -dir migrations add_users_table
```
</quick_reference>
