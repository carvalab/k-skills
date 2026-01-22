# Backend Debugging

Profiling, debugging tools, common issues, observability.

## Debugging Process

1. **Reproduce** - Reliably trigger the issue
2. **Observe** - Gather logs, metrics, traces
3. **Hypothesize** - Form theories about cause
4. **Test** - Verify or disprove
5. **Fix & Verify** - Apply solution, confirm it works

## Debugging Tools

### Node.js

```bash
node --inspect-brk app.js    # Chrome DevTools (chrome://inspect)
DEBUG=app:* node app.js      # Debug module
```

### Go (Delve)

```bash
go install github.com/go-delve/delve/cmd/dlv@latest
dlv debug main.go
# Commands: b (breakpoint), c (continue), n (next), s (step), p (print)
```

## Database Debugging

### PostgreSQL

```sql
-- Analyze query performance
EXPLAIN ANALYZE SELECT u.name, COUNT(o.id)
FROM users u LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id LIMIT 10;

-- Find slow queries
SELECT query, calls, mean_exec_time
FROM pg_stat_statements ORDER BY mean_exec_time DESC LIMIT 10;

-- Kill long-running query
SELECT pid, now() - query_start as duration, query FROM pg_stat_activity WHERE state = 'active';
SELECT pg_terminate_backend(pid);
```

### Redis

```bash
redis-cli MONITOR           # Real-time commands
redis-cli SLOWLOG GET 10    # Slow commands
redis-cli --bigkeys         # Memory analysis
```

## Performance Profiling

### Node.js

```bash
npm install -g 0x clinic
0x node app.js                     # CPU flamegraph
clinic doctor -- node app.js       # CPU profiling
clinic heapprofiler -- node app.js # Memory
```

### Go (pprof)

```go
import _ "net/http/pprof"
go func() { http.ListenAndServe("localhost:6060", nil) }()
```

```bash
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
go tool pprof http://localhost:6060/debug/pprof/heap
```

## Common Issues

### Memory Leaks

```typescript
// BAD: Event listener leak
eventBus.on('data', handler);  // Never removed

// GOOD: Remove listeners
destroy() { this.eventBus.off('data', this.handler); }

// BAD: Unbounded cache
const cache = new Map();  // Grows forever

// GOOD: LRU with limit
const cache = new LRU({ max: 1000, ttl: 3600000 });
```

### Connection Pool Exhaustion

```typescript
// BAD: Never released
const client = await pool.connect();
return await client.query(...);

// GOOD: Always release
const client = await pool.connect();
try { return await client.query(...); }
finally { client.release(); }
```

## OpenTelemetry + DataDog Tracing

**Go:**
```go
import (
    "go.opentelemetry.io/otel"
    "github.com/DataDog/dd-trace-go/v2/ddtrace/opentelemetry"
    "github.com/DataDog/dd-trace-go/v2/ddtrace/tracer"
)

func initTracer() func() {
    tracer.Start(
        tracer.WithEnv(os.Getenv("KAVAK_ENVIRONMENT")),
        tracer.WithService("my-service"),
    )
    provider := opentelemetry.NewTracerProvider()
    otel.SetTracerProvider(provider)
    return func() { provider.Shutdown(); tracer.Stop() }
}
```

**Node.js:**
```typescript
// Initialize BEFORE other imports
import tracer from 'dd-trace';
tracer.init({ service: 'my-api', env: process.env.KAVAK_ENVIRONMENT });

const span = tracer.startSpan('custom.operation');
try { /* work */ span.setTag('result', 'success'); }
catch (e) { span.setTag('error', e); throw e; }
finally { span.finish(); }
```

## DataDog Metrics Cardinality

**CRITICAL: Never use IDs as metric tags** - causes metric explosion.

```go
// BAD: High cardinality
statsd.Incr("api.request", []string{"user_id:" + userId}, 1)

// GOOD: Low cardinality (~100 max unique values per tag)
statsd.Incr("api.request", []string{
    "endpoint:/users",
    "method:GET",
    "status:2xx",
}, 1)
```

**Rules:**
- Max ~100 unique values per tag
- Group values (status: 2xx, 4xx, 5xx)
- IDs belong in traces/logs, not metrics

## Structured Logging

```typescript
// Node.js (Pino)
import pino from 'pino';
const logger = pino({ level: process.env.LOG_LEVEL || 'info' });
logger.info({ userId: '123', action: 'login' }, 'User logged in');
logger.error({ err: error, userId: '123' }, 'Operation failed');
```

```go
// Go (Zap)
logger, _ := zap.NewProduction()
defer logger.Sync()
logger.Info("user logged in", zap.String("user_id", "123"))
logger.Error("operation failed", zap.Error(err))
```

**Log Levels:** DEBUG (dev only), INFO (normal flow), WARN (potential issues), ERROR (failures)

**DO:** Request metadata, errors with context, performance metrics
**DON'T:** Passwords, PII, tokens, full request bodies

## Debugging Checklist

- [ ] Read error message completely
- [ ] Check logs around timestamp
- [ ] Reproduce reliably
- [ ] Enable debug logging
- [ ] Check DB queries (EXPLAIN ANALYZE)
- [ ] Profile if slow
- [ ] Add regression test after fix
