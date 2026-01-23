# Kavak Unified Observability Framework

**Kavak-only.** Standardized logging and metrics using Kavak's SDKs.

## Overview

The Unified Observability Framework provides consistent observability across all Kavak applications:
- **Logging**: Structured JSON logs with RFC3339 microsecond timestamps
- **Metrics**: DataDog StatsD with automatic namespace prefixing
- **Tracing**: OpenTelemetry + dd-trace (see `debugging.md`)

## SDK Packages

| Language | Logging | Metrics |
|----------|---------|---------|
| Go | `gitlab.com/kavak-it/go-sdk/kvklog` | `gitlab.com/kavak-it/go-sdk/kvkmetric` |
| TypeScript | `@kavak-it/ts-sdk` (logger) | `@kavak-it/ts-sdk` (metricClient) |

## Logging

### Standardized Log Structure

All logs must follow this structure:

```json
{
  "date": "2024-01-15T10:30:45.123456Z",
  "status": "INFO",
  "message": "operation completed",
  "caller": "pkg/handler.go:42",
  "log": "service-name"
}
```

**Field Requirements:**
- `date`: RFC3339 with **microsecond precision** (6 digits: `.123456`)
- `status`: Uppercase (DEBUG, INFO, WARN, ERROR)
- `message`: 50-200 characters, actionable
- `caller`: Source file and line
- `log`: Service identifier

### Go Logging

```go
import "gitlab.com/kavak-it/go-sdk/kvklog"

// Logger is singleton (thread-safe via sync.Once)
kvklog.LogInfo(ctx, "user profile updated",
    zap.String("user_id", hashedUserID),
    zap.String("field", "email"),
)

kvklog.LogError(ctx, "failed to process payment",
    zap.Error(err),
    zap.String("payment_id", paymentID),
    zap.Int("retry_attempt", 3),
)
```

### TypeScript Logging

```typescript
import { logger } from '@kavak-it/ts-sdk';

// Logger is singleton (module-level export)
logger.info("User profile updated", {
  user_id: hashedUserID,
  field: "email",
});

logger.error("Failed to process payment", {
  payment_id: paymentID,
  error_code: "INVALID_CARD",
  retry_attempt: 3,
  error: err.message,
});
```

### Message Guidelines

Use state-machine pattern for operation logging:

```typescript
// Start: Present progressive
logger.info("Processing user payment", { payment_id: id });

try {
  await processPayment(id);
  // Success: Past tense
  logger.info("Payment processed", { payment_id: id });
} catch (error) {
  // Failure: Past tense with "Failed"
  logger.error("Failed to process payment", {
    payment_id: id,
    error_code: error.code,
    error_message: error.message,
  });
}
```

### Log Levels

| Level | Use Case |
|-------|----------|
| DEBUG | Development only, detailed flow tracing |
| INFO | Normal flow, state changes, business events |
| WARN | Recoverable errors, degraded functionality |
| ERROR | Unrecoverable errors, requires immediate attention |

### Environment Configuration

```bash
KAVAK_ENVIRONMENT=local|development|production  # Format selection
LOG_LEVEL=DEBUG|INFO|WARN|ERROR                 # Level override
```

- `local`: Human-readable with colors
- `development`/`production`: JSON structured

## Metrics

### Namespace Structure

All metrics auto-prefixed: `product.workload.<custom_metric>`

```text
product.workload.orders.created.total
product.workload.api.requests.count
product.workload.cache.hits.ratio
```

### Go Metrics

```go
import "gitlab.com/kavak-it/go-sdk/kvkmetric"

// Counter
kvkmetric.Count(ctx, "orders.created.total", 1, []string{
    kvkmetric.Tag("payment_method", "credit_card"),
    kvkmetric.Tag("status", "success"),
})

// Gauge
kvkmetric.Gauge(ctx, "orders.pending.count", pendingCount, []string{
    kvkmetric.Tag("priority", "high"),
})

// Histogram/Timing
kvkmetric.Timing(ctx, "api.response.time", duration, []string{
    kvkmetric.Tag("endpoint", "/users"),
    kvkmetric.Tag("method", "GET"),
})
```

### TypeScript Metrics

```typescript
import { metricClient } from '@kavak-it/ts-sdk';

// Counter
metricClient.increment('orders.created.total', 1, [
  'payment_method:credit_card',
  'status:success',
]);

// Gauge
metricClient.gauge('orders.pending.count', pendingCount, [
  'priority:high',
]);

// Histogram
metricClient.histogram('api.response.time', duration, [
  'endpoint:/users',
  'method:GET',
]);
```

### Cardinality Rules (CRITICAL)

**Never use high-cardinality values as tags:**

```typescript
// BAD: Creates millions of unique metrics
metricClient.increment('api.request', 1, [
  'user_id:user-12345',      // Unbounded
  'order_id:order-67890',    // Unbounded
  'request_id:req-abc123',   // Unbounded
]);

// GOOD: Bounded tag values (~100 max unique values per tag)
metricClient.increment('api.request', 1, [
  'method:GET',              // Finite set
  'status_class:2xx',        // Grouped (not exact code)
  'endpoint:/users',         // Finite set
  'payment_method:credit_card',
]);
```

**Tag Guidelines:**
- Max ~100 unique values per tag
- Use enums/categories, not raw values
- Group similar values (`status_class:2xx` not `status:200`)
- IDs belong in traces/logs, **never** in metrics

### Default Tags (Auto-Added)

- `sdk_name`: Language identifier
- `sdk_version`: SDK version
- `kube_namespace`: Product name
- `service`: Workload name

## PII Handling

**Never log or tag:** Passwords, tokens, API keys, credit cards, DNI/SSN

**Mask sensitive data:**

```typescript
function maskEmail(email: string): string {
  const [local, domain] = email.split('@');
  return `${local.substring(0, 3)}***@${domain}`;
}

// Masking patterns:
// Email: user***@domain.com
// Credit card: ****-****-****-1234
// DNI: ***-***-1234
// API key: abcd...xyz1 (first 4 + last 4)

logger.info("Starting backup", { user_email: maskEmail(email) });
```

## Context Objects

Standard context structures for logging:

```json
{
  "dd": {
    "trace_id": 1234567890123456789,
    "span_id": 9876543210987654321
  },
  "http": {
    "url": "https://api.example.com/users",
    "method": "GET",
    "status_code": 200,
    "request_id": "req-123-456"
  },
  "error": {
    "kind": "validation_error",
    "message": "Invalid input",
    "stack": "..."
  },
  "db": {
    "instance": "users_db",
    "statement": "SELECT * FROM users",
    "operation": "query"
  }
}
```

## Singleton Pattern

Both SDKs implement singleton pattern:

**Go:**
```go
// Thread-safe via sync.Once
var logger struct {
    sync.Once
    l *zap.Logger
}

func Logger() *zap.Logger {
    logger.Do(func() {
        logger.l = kvklogdefaults.DefaultLogger()
    })
    return logger.l
}
```

**TypeScript:**
```typescript
// Module-level export (Node.js caching ensures singleton)
export const logger = createKVKLogger();
export const metricClient = createMetricsClient();
```

## Graceful Degradation

SDKs never crash the application:
- Logging failures: Continue with basic logger
- Metrics failures: Fall back to NoOp/FakeStatsD client
- Always log warnings when fallback is used

## Best Practices

**DO:** Structured logging, include request_id/trace_id, state-machine pattern (start/success/failed), low-cardinality metric tags, mask PII.

**DON'T:** Log in tight loops, DEBUG in production, user IDs in metric tags, log secrets, unbounded cardinality.
