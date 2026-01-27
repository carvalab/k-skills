# Go Database Access

pgx/v5 + sqlc patterns. **Never use GORM** (reflection overhead, N+1 queries).

## sqlc (Recommended)

Type-safe, compile-time SQL. Best performance.

```bash
go install github.com/sqlc-dev/sqlc/cmd/sqlc@latest
```

```yaml
# sqlc.yaml
version: '2'
sql:
  - engine: 'postgresql'
    queries: 'db/queries/'
    schema: 'db/migrations/'
    gen:
      go:
        package: 'db'
        out: 'internal/db'
        sql_package: 'pgx/v5'
        emit_json_tags: true
        emit_prepared_queries: true
```

```sql
-- db/queries/users.sql

-- name: GetUser :one
SELECT id, email, name, created_at FROM users WHERE id = $1;

-- name: ListUsers :many
SELECT id, email, name, created_at FROM users
ORDER BY created_at DESC LIMIT $1 OFFSET $2;

-- name: CreateUser :one
INSERT INTO users (id, email, name, created_at)
VALUES ($1, $2, $3, $4) RETURNING *;

-- name: UpdateUser :exec
UPDATE users SET email = $2, name = $3 WHERE id = $1;

-- name: DeleteUser :exec
DELETE FROM users WHERE id = $1;
```

```bash
sqlc generate
```

```go
// Usage - type-safe generated code
import "myapp/internal/db"

type UserRepository struct {
    queries *db.Queries
}

func NewUserRepository(pool *pgxpool.Pool) *UserRepository {
    return &UserRepository{queries: db.New(pool)}
}

func (r *UserRepository) FindByID(ctx context.Context, id string) (*db.User, error) {
    user, err := r.queries.GetUser(ctx, id)
    if err == pgx.ErrNoRows {
        return nil, nil
    }
    return &user, err
}

func (r *UserRepository) List(ctx context.Context, limit, offset int32) ([]db.User, error) {
    return r.queries.ListUsers(ctx, db.ListUsersParams{Limit: limit, Offset: offset})
}
```

## pgx/v5 (Dynamic Queries)

Use when sqlc isn't suitable (dynamic filters, complex joins).

```go
import (
    "github.com/jackc/pgx/v5"
    "github.com/jackc/pgx/v5/pgxpool"
)

func NewPool(connString string) (*pgxpool.Pool, error) {
    config, err := pgxpool.ParseConfig(connString)
    if err != nil {
        return nil, err
    }
    config.MaxConns = 20
    config.MinConns = 5
    return pgxpool.NewWithConfig(context.Background(), config)
}

// Dynamic query
func (r *UserRepository) Search(ctx context.Context, filters map[string]any) ([]User, error) {
    query := "SELECT id, email, name FROM users WHERE 1=1"
    args := []any{}
    argNum := 1

    if email, ok := filters["email"]; ok {
        query += fmt.Sprintf(" AND email ILIKE $%d", argNum)
        args = append(args, "%"+email.(string)+"%")
        argNum++
    }

    rows, err := r.pool.Query(ctx, query, args...)
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    return pgx.CollectRows(rows, pgx.RowToStructByName[User])
}
```

## Batch Operations

```go
func (r *UserRepository) CreateBatch(ctx context.Context, users []User) error {
    batch := &pgx.Batch{}
    for _, u := range users {
        batch.Queue(
            "INSERT INTO users (id, email, name) VALUES ($1, $2, $3)",
            u.ID, u.Email, u.Name,
        )
    }
    results := r.pool.SendBatch(ctx, batch)
    defer results.Close()

    for range users {
        if _, err := results.Exec(); err != nil {
            return err
        }
    }
    return nil
}
```

## Transactions

```go
func (r *UserRepository) TransferCredits(ctx context.Context, fromID, toID string, amount int) error {
    tx, err := r.pool.Begin(ctx)
    if err != nil {
        return err
    }
    defer tx.Rollback(ctx)

    _, err = tx.Exec(ctx, "UPDATE users SET credits = credits - $1 WHERE id = $2", amount, fromID)
    if err != nil {
        return err
    }
    _, err = tx.Exec(ctx, "UPDATE users SET credits = credits + $1 WHERE id = $2", amount, toID)
    if err != nil {
        return err
    }
    return tx.Commit(ctx)
}
```
