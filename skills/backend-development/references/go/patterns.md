# Go Patterns

Error handling, validation, concurrency, testing, project structure.

## Error Handling

```go
type AppError struct {
    Code    string `json:"code"`
    Message string `json:"message"`
    Status  int    `json:"-"`
    Err     error  `json:"-"`
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("%s: %v", e.Message, e.Err)
    }
    return e.Message
}

func (e *AppError) Unwrap() error { return e.Err }

var (
    ErrNotFound     = &AppError{Code: "NOT_FOUND", Message: "Resource not found", Status: 404}
    ErrInvalidInput = &AppError{Code: "INVALID_INPUT", Message: "Invalid input", Status: 400}
    ErrUnauthorized = &AppError{Code: "UNAUTHORIZED", Message: "Unauthorized", Status: 401}
)

func NewNotFoundError(resource string) *AppError {
    return &AppError{Code: "NOT_FOUND", Message: fmt.Sprintf("%s not found", resource), Status: 404}
}
```

### Error Wrapping

```go
func (s *UserService) GetUser(ctx context.Context, id string) (*User, error) {
    user, err := s.repo.FindByID(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("get user %s: %w", id, err)
    }
    if user == nil {
        return nil, NewNotFoundError("user")
    }
    return user, nil
}

// Check error type
if errors.Is(err, ErrNotFound) { /* handle */ }

var appErr *AppError
if errors.As(err, &appErr) {
    respondError(w, appErr.Status, appErr.Message)
}
```

## Validation

```go
import "github.com/go-playground/validator/v10"

type CreateUserDTO struct {
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=12"`
    Name     string `json:"name" validate:"required,min=2,max=100"`
}

var validate = validator.New()

func (dto *CreateUserDTO) Validate() error {
    return validate.Struct(dto)
}
```

## Concurrency (errgroup)

```go
import "golang.org/x/sync/errgroup"

func FetchUserData(ctx context.Context, userID string) (*UserData, error) {
    g, ctx := errgroup.WithContext(ctx)
    var user *User
    var orders []Order

    g.Go(func() error {
        var err error
        user, err = userService.GetUser(ctx, userID)
        return err
    })

    g.Go(func() error {
        var err error
        orders, err = orderService.GetOrders(ctx, userID)
        return err
    })

    if err := g.Wait(); err != nil {
        return nil, err
    }
    return &UserData{User: user, Orders: orders}, nil
}
```

## Testing

### Table-Driven Tests

```go
func TestUserService_CreateUser(t *testing.T) {
    tests := []struct {
        name    string
        input   CreateUserDTO
        wantErr bool
    }{
        {"valid user", CreateUserDTO{Email: "test@example.com", Password: "SecurePass123"}, false},
        {"invalid email", CreateUserDTO{Email: "invalid", Password: "SecurePass123"}, true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            user, err := svc.CreateUser(context.Background(), tt.input)
            if tt.wantErr {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
                assert.NotEmpty(t, user.ID)
            }
        })
    }
}
```

### Mockery

```bash
go install github.com/vektra/mockery/v2@latest
mockery --name=UserRepository --with-expecter --output=mocks
```

```go
func TestUserService_GetUser(t *testing.T) {
    mockRepo := mocks.NewMockUserRepository(t)
    mockRepo.EXPECT().
        FindByID(mock.Anything, "123").
        Return(&User{ID: "123", Email: "test@example.com"}, nil)

    svc := NewUserService(mockRepo)
    user, err := svc.GetUser(context.Background(), "123")

    assert.NoError(t, err)
    assert.Equal(t, "test@example.com", user.Email)
}
```

## Project Structure

```
cmd/
├── api/
│   └── main.go
└── worker/
    └── main.go
internal/
├── user/
│   ├── handler.go
│   ├── service.go
│   ├── repository.go
│   └── model.go
├── config/
└── middleware/
pkg/
├── database/
├── logger/
└── httputil/
```

## Commands

```bash
go test ./...                    # Run tests
go test -v ./...                 # Verbose
go test -cover ./...             # Coverage
go test -race ./...              # Race detection
go test -shuffle=on ./...        # Shuffle test order
golangci-lint run                # Lint
go build -ldflags="-s -w" -o bin/api ./cmd/api  # Build
```
