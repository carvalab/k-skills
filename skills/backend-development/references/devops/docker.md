# Docker & Monitoring

Multi-stage builds, Docker Compose, health checks, DataDog metrics.

## Multi-Stage Build (Node.js)

```dockerfile
FROM node:24-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:24-slim
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

RUN groupadd -g 1001 nodejs && useradd -u 1001 -g nodejs nodejs
USER nodejs

EXPOSE 3000
CMD ["node", "dist/main.js"]
```

## Multi-Stage Build (Go)

```dockerfile
FROM golang:1.25 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /bin/api ./cmd/api

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates \
    && rm -rf /var/lib/apt/lists/*
COPY --from=builder /bin/api /bin/api

RUN useradd -r -u 1001 appuser
USER appuser

EXPOSE 3000
CMD ["/bin/api"]
```

## Docker Compose (Development)

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - '3000:3000'
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - '5432:5432'

  redis:
    image: redis:7-alpine
    ports:
      - '6379:6379'

volumes:
  postgres-data:
```

## Docker Best Practices

- Use multi-stage builds (smaller images)
- Run as non-root user
- Use specific version tags (not `latest`)
- Order Dockerfile by least likely to change
- Use `.dockerignore` to exclude unnecessary files

## Health Checks

```typescript
app.get('/health/liveness', (req, res) => {
  res.json({ status: 'ok', timestamp: Date.now() });
});

app.get('/health/readiness', async (req, res) => {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
  };
  const isReady = Object.values(checks).every(Boolean);
  res.status(isReady ? 200 : 503).json({ status: isReady ? 'ready' : 'not ready', checks });
});

async function checkDatabase(): Promise<boolean> {
  try {
    await db.query('SELECT 1');
    return true;
  } catch {
    return false;
  }
}
```

## DataDog Metrics

**Go (dogstatsd):**

```go
import "github.com/DataDog/datadog-go/v5/statsd"

client, _ := statsd.New("127.0.0.1:8125")
defer client.Close()

// GOOD: Low cardinality tags (~100 max unique values per tag)
client.Incr("api.request", []string{
    "endpoint:/users",
    "method:GET",
    "status:2xx",
}, 1)

// BAD: Never use IDs as tags - causes metric explosion
// client.Incr("api.request", []string{"user_id:" + userId}, 1)
```

**Node.js (hot-shots):**

```typescript
import StatsD from 'hot-shots';

const statsd = new StatsD({ host: 'localhost', port: 8125 });

statsd.increment('api.request', {
  endpoint: '/users',
  method: 'GET',
  status: '2xx',
});
```

## Structured Logging

```typescript
// Node.js (Pino)
import pino from 'pino';
const logger = pino({ level: process.env.LOG_LEVEL || 'info' });
logger.info({ userId: '123', action: 'login' }, 'User logged in');
```

```go
// Go (Zap)
import "go.uber.org/zap"
logger, _ := zap.NewProduction()
defer logger.Sync()
logger.Info("user logged in", zap.String("user_id", "123"))
```

## Distributed Tracing (DataDog)

**Node.js:**

```typescript
// Initialize BEFORE other imports
import tracer from 'dd-trace';
tracer.init({ service: 'my-api', env: process.env.KAVAK_ENVIRONMENT });
```

**Go:**

```go
import "github.com/DataDog/dd-trace-go/v2/ddtrace/tracer"

tracer.Start(
    tracer.WithEnv(os.Getenv("KAVAK_ENVIRONMENT")),
    tracer.WithService("my-service"),
)
defer tracer.Stop()
```

## DevOps Checklist

- [ ] Health checks (liveness + readiness)
- [ ] DataDog metrics with low-cardinality tags
- [ ] Structured logging (JSON)
- [ ] Distributed tracing configured
- [ ] Non-root user in Dockerfile
- [ ] Multi-stage build for smaller images
