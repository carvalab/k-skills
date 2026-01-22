# Go Redis & Job Queues

rueidis for Redis, River for PostgreSQL-backed job queues.

## Redis (rueidis)

**New projects**: Use `rueidis` (high performance).
**Existing projects**: Keep current library.

```go
import (
    "github.com/redis/rueidis"
    "github.com/redis/rueidis/rueidisotel"
)

// Setup with OpenTelemetry tracing
func NewRedisClient(ctx context.Context, addr string) (rueidis.Client, error) {
    client, err := rueidis.NewClient(rueidis.ClientOption{
        InitAddress: []string{addr},
    })
    if err != nil {
        return nil, err
    }
    return rueidisotel.NewClient(client), nil
}

// Basic operations
func (c *Cache) Get(ctx context.Context, key string) (string, error) {
    cmd := c.client.B().Get().Key(key).Build()
    return c.client.Do(ctx, cmd).ToString()
}

func (c *Cache) Set(ctx context.Context, key, value string, ttl time.Duration) error {
    cmd := c.client.B().Set().Key(key).Value(value).Ex(ttl).Build()
    return c.client.Do(ctx, cmd).Error()
}

func (c *Cache) Delete(ctx context.Context, key string) error {
    cmd := c.client.B().Del().Key(key).Build()
    return c.client.Do(ctx, cmd).Error()
}
```

### Compat Mode (Migration)

For easier migration from go-redis:

```go
import "github.com/redis/rueidis/rueidiscompat"

client, _ := rueidis.NewClient(rueidis.ClientOption{InitAddress: []string{addr}})
compat := rueidiscompat.NewAdapter(client)

// Use familiar go-redis API
compat.Set(ctx, "key", "value", time.Hour)
val, _ := compat.Get(ctx, "key").Result()
```

## Job Queue (River)

PostgreSQL-backed job queue with `riverqueue/river`.

```go
import (
    "github.com/riverqueue/river"
    "github.com/riverqueue/river/riverdriver/riverpgxv5"
)

// Define job args
type EmailJobArgs struct {
    UserID  string `json:"user_id"`
    Subject string `json:"subject"`
    Body    string `json:"body"`
}

func (EmailJobArgs) Kind() string { return "send_email" }

// Define worker
type EmailWorker struct {
    river.WorkerDefaults[EmailJobArgs]
    emailService *EmailService
}

func (w *EmailWorker) Work(ctx context.Context, job *river.Job[EmailJobArgs]) error {
    return w.emailService.Send(ctx, job.Args.UserID, job.Args.Subject, job.Args.Body)
}

// Setup client
func NewRiverClient(pool *pgxpool.Pool) (*river.Client[pgx.Tx], error) {
    workers := river.NewWorkers()
    river.AddWorker(workers, &EmailWorker{})

    return river.NewClient(riverpgxv5.New(pool), &river.Config{
        Queues: map[string]river.QueueConfig{
            river.QueueDefault: {MaxWorkers: 10},
        },
        Workers: workers,
    })
}

// Enqueue job
func (s *Service) QueueEmail(ctx context.Context, userID, subject, body string) error {
    _, err := s.riverClient.Insert(ctx, EmailJobArgs{
        UserID: userID, Subject: subject, Body: body,
    }, nil)
    return err
}

// Graceful shutdown
func (s *Service) Shutdown(ctx context.Context) error {
    softCtx, cancel := context.WithTimeout(ctx, 30*time.Second)
    defer cancel()
    if err := s.riverClient.Stop(softCtx); err != nil {
        hardCtx, cancel := context.WithTimeout(ctx, 10*time.Second)
        defer cancel()
        return s.riverClient.Stop(hardCtx)
    }
    return nil
}
```

## Observability

See `debugging.md` for full patterns.

```go
import (
    "go.opentelemetry.io/otel"
    ddotel "github.com/DataDog/dd-trace-go/v2/ddtrace/opentelemetry"
    "go.uber.org/zap"
)

// Setup OpenTelemetry with Datadog
func InitTracing() func() {
    provider := ddotel.NewTracerProvider()
    otel.SetTracerProvider(provider)
    return func() { provider.Shutdown(context.Background()) }
}

// Structured logging
logger, _ := zap.NewProduction()
defer logger.Sync()

logger.Info("user created",
    zap.String("user_id", user.ID),
    zap.String("email", user.Email),
)
```
