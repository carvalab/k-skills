# TDD Examples: Go

Complete TDD example for Go projects.

## Example: User Registration Service

### Step 1: RED - Write Failing Test

```go
// user_service_test.go
package user

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestUserService_Register_CreatesUserWithHashedPassword(t *testing.T) {
    // Arrange
    repo := NewInMemoryUserRepository()
    hasher := NewBcryptHasher()
    svc := NewUserService(repo, hasher)

    // Act
    user, err := svc.Register("john@example.com", "password123")

    // Assert
    assert.NoError(t, err)
    assert.NotEmpty(t, user.ID)
    assert.Equal(t, "john@example.com", user.Email)
    assert.NotEqual(t, "password123", user.PasswordHash)
}

func TestUserService_Register_RejectsInvalidEmail(t *testing.T) {
    repo := NewInMemoryUserRepository()
    hasher := NewBcryptHasher()
    svc := NewUserService(repo, hasher)

    _, err := svc.Register("invalid-email", "password123")

    assert.Error(t, err)
    assert.Contains(t, err.Error(), "invalid email")
}

func TestUserService_Register_RejectsDuplicateEmail(t *testing.T) {
    repo := NewInMemoryUserRepository()
    hasher := NewBcryptHasher()
    svc := NewUserService(repo, hasher)

    svc.Register("john@example.com", "password123")
    _, err := svc.Register("john@example.com", "different")

    assert.Error(t, err)
    assert.Contains(t, err.Error(), "already exists")
}
```

Run: `go test ./...` → Should fail (types don't exist)

### Step 2: GREEN - Minimal Implementation

```go
// user_service.go
package user

import (
    "errors"
    "regexp"

    "github.com/google/uuid"
)

type User struct {
    ID           string
    Email        string
    PasswordHash string
}

type UserRepository interface {
    Save(user *User) error
    FindByEmail(email string) (*User, error)
}

type PasswordHasher interface {
    Hash(password string) (string, error)
}

type UserService struct {
    repo   UserRepository
    hasher PasswordHasher
}

func NewUserService(repo UserRepository, hasher PasswordHasher) *UserService {
    return &UserService{repo: repo, hasher: hasher}
}

func (s *UserService) Register(email, password string) (*User, error) {
    if !isValidEmail(email) {
        return nil, errors.New("invalid email format")
    }

    existing, _ := s.repo.FindByEmail(email)
    if existing != nil {
        return nil, errors.New("email already exists")
    }

    hash, err := s.hasher.Hash(password)
    if err != nil {
        return nil, err
    }

    user := &User{
        ID:           uuid.New().String(),
        Email:        email,
        PasswordHash: hash,
    }

    if err := s.repo.Save(user); err != nil {
        return nil, err
    }

    return user, nil
}

var emailRegex = regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)

func isValidEmail(email string) bool {
    return emailRegex.MatchString(email)
}
```

### Step 3: Test Doubles

```go
// test_helpers.go
package user

type InMemoryUserRepository struct {
    users map[string]*User
}

func NewInMemoryUserRepository() *InMemoryUserRepository {
    return &InMemoryUserRepository{users: make(map[string]*User)}
}

func (r *InMemoryUserRepository) Save(user *User) error {
    r.users[user.Email] = user
    return nil
}

func (r *InMemoryUserRepository) FindByEmail(email string) (*User, error) {
    return r.users[email], nil
}

type BcryptHasher struct{}

func NewBcryptHasher() *BcryptHasher {
    return &BcryptHasher{}
}

func (h *BcryptHasher) Hash(password string) (string, error) {
    // In real code, use bcrypt.GenerateFromPassword
    return "hashed_" + password, nil
}
```

---

## Table-Driven Tests

```go
func TestCalculator_Add(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"zeros", 0, 0, 0},
        {"negative numbers", -1, -2, -3},
        {"mixed signs", -5, 10, 5},
    }

    calc := NewCalculator()

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := calc.Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d",
                    tt.a, tt.b, result, tt.expected)
            }
        })
    }
}
```

---

## Test Doubles in Go

### Mockery (Recommended for Interfaces)

Mockery generates mock implementations from interfaces. Use it when you need to verify interactions or control return values.

**Setup:**

```bash
# Install mockery
go install github.com/vektra/mockery/v2@latest

# Generate mocks for all interfaces (add to Makefile)
mockery --all --with-expecter --output=mocks

# Generate mock for specific interface
mockery --name=UserRepository --with-expecter --output=mocks
```

**Configuration (.mockery.yaml):**

```yaml
with-expecter: true
packages:
  github.com/yourproject/internal/user:
    interfaces:
      UserRepository:
      PasswordHasher:
  github.com/yourproject/internal/notification:
    interfaces:
      EmailSender:
```

**Using Generated Mocks:**

```go
// user_service_test.go
package user

import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/yourproject/mocks"
)

func TestUserService_Register_WithMockery(t *testing.T) {
    // Create mocks
    mockRepo := mocks.NewMockUserRepository(t)
    mockHasher := mocks.NewMockPasswordHasher(t)

    // Set expectations
    mockRepo.EXPECT().
        FindByEmail("john@example.com").
        Return(nil, nil)

    mockHasher.EXPECT().
        Hash("password123").
        Return("hashed_password", nil)

    mockRepo.EXPECT().
        Save(mock.MatchedBy(func(u *User) bool {
            return u.Email == "john@example.com"
        })).
        Return(nil)

    // Test
    svc := NewUserService(mockRepo, mockHasher)
    user, err := svc.Register("john@example.com", "password123")

    assert.NoError(t, err)
    assert.Equal(t, "john@example.com", user.Email)
}

func TestUserService_Register_DuplicateEmail_WithMockery(t *testing.T) {
    mockRepo := mocks.NewMockUserRepository(t)
    mockHasher := mocks.NewMockPasswordHasher(t)

    // Return existing user
    mockRepo.EXPECT().
        FindByEmail("existing@example.com").
        Return(&User{Email: "existing@example.com"}, nil)

    svc := NewUserService(mockRepo, mockHasher)
    _, err := svc.Register("existing@example.com", "password")

    assert.Error(t, err)
    assert.Contains(t, err.Error(), "already exists")
}
```

**Mockery Matchers:**

```go
// Exact match
mockRepo.EXPECT().FindByID("123").Return(user, nil)

// Any value
mockRepo.EXPECT().FindByID(mock.Anything).Return(user, nil)

// Custom matcher
mockRepo.EXPECT().Save(mock.MatchedBy(func(u *User) bool {
    return u.Email != "" && u.PasswordHash != ""
})).Return(nil)

// Called N times
mockRepo.EXPECT().Save(mock.Anything).Return(nil).Times(2)

// Never called
mockRepo.EXPECT().Delete(mock.Anything).Never()
```

---

### Fake (In-Memory Implementation)

Use fakes when you need stateful behavior (e.g., repositories that store and retrieve data).

```go
type InMemoryRepo struct {
    data map[string]Entity
    mu   sync.RWMutex
}

func NewInMemoryRepo() *InMemoryRepo {
    return &InMemoryRepo{data: make(map[string]Entity)}
}

func (r *InMemoryRepo) Save(e Entity) error {
    r.mu.Lock()
    defer r.mu.Unlock()
    r.data[e.ID] = e
    return nil
}

func (r *InMemoryRepo) Find(id string) (Entity, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()
    if e, ok := r.data[id]; ok {
        return e, nil
    }
    return Entity{}, ErrNotFound
}
```

---

## When to Use Mockery vs Fakes

| Use Case | Recommendation |
|----------|----------------|
| Verify method was called | Mockery |
| Verify call arguments | Mockery |
| Control return values | Mockery |
| Test error handling | Mockery |
| Need stateful behavior | Fake |
| Integration-like tests | Fake |
| Complex query logic | Fake |

---

## Commands

```bash
# Run all tests
go test ./...

# Verbose output
go test -v ./...

# Run specific test
go test -v -run TestUserService_Register ./...

# Coverage report
go test -cover ./...

# Coverage with HTML report
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out

# Race detection
go test -race ./...

# Benchmark
go test -bench=. ./...
```
