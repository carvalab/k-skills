# KBroker (Internal Event Bus)

Kafka-by-REST for publishing/subscribing events between microservices.

## Overview

- **Push-based delivery**: kbroker POSTs to your HTTP endpoint
- **At-least-once**: Duplicates possible, design idempotent consumers
- **No ordering**: Messages may arrive out of order
- **7-day TTL**: Messages discarded after 7 days

## Publishing Events (Go SDK)

```go
import "gitlab.com/kavak-it/kbroker-go-sdk/v2/kbpublisher"

publisher, err := kbpublisher.NewPublisher()
if err != nil {
    return err
}

// Add auth token to context
ctx = kbpublisher.CtxWithAuthToken(ctx, authToken)

// Publish event
result, err := publisher.Publish(ctx, "orders-topic", kbpublisher.Event{
    ID:      "optional-custom-id",  // For idempotency
    Payload: orderEvent,
})

// Scheduled message (10 minutes delay)
result, err := publisher.PublishScheduled(ctx, "my-topic", kbpublisher.Event{
    Payload: event,
}, 10)
```

## Publishing Events (HTTP - Any Language)

```typescript
// Node.js/TypeScript (no SDK yet)
const response = await fetch('https://kbroker-inbox.prd.kavak.io/publish/my-topic', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${authToken}`,
    },
    body: JSON.stringify({ orderId: '123', status: 'completed' }),
});

// Scheduled message
await fetch('https://kbroker-inbox.prd.kavak.io/scheduled/my-topic', {
    method: 'POST',
    headers: {
        'X-KB-ScheduledMinutes': '10',  // 10 minutes delay (min: 5)
        'Authorization': `Bearer ${authToken}`,
    },
    body: JSON.stringify(payload),
});
```

**Endpoints:**
- Dev: `kbroker-inbox.dev.kavak.io`
- Prod: `kbroker-inbox.prd.kavak.io`

## Consuming Events (HTTP Handler)

kbroker pushes to YOUR endpoint. You don't pull.

```go
func handleKBrokerEvent(w http.ResponseWriter, r *http.Request) {
    messageID := r.Header.Get("X-KB-MessageID")
    publishTime := r.Header.Get("X-KB-PublishTime")

    var event OrderEvent
    json.NewDecoder(r.Body).Decode(&event)

    // Process event (design for idempotency!)
    if err := processEvent(event); err != nil {
        // Return 200 anyway to prevent retry storms
        // Track failures via metrics instead
        logger.Error("Failed to process event", "error", err, "messageID", messageID)
    }

    w.WriteHeader(http.StatusOK)  // ACK
}
```

**Response codes:**
- `2XX` → ACK (processed, move to next)
- `Non-2XX` → NACK (retry with exponential backoff ~5 min)

**Best practice:** Return 200 for all outcomes, track failures via DataDog metrics.

## Authentication

**Publisher (.kavak/config.yaml):**
```yaml
auth:
  enabled: true
```

**Token scopes:**
- Publisher: `kbroker-inbox.event-publish` (audience: `kbroker-inbox`)
- Consumer: `<product-workload>.event-receive`

**Consumer auth config (.kavak/auth/auth.production.yaml):**
```yaml
version: v1
available_scopes:
  - event-receive
allowed_clients:
  kbroker-outbox:
    - event-receive
```

## Subscription Management (CLI)

```bash
# Create subscription
kavak kbroker subscription create

# Pause/Resume
kavak kbroker subscription pause <name>
kavak kbroker subscription resume <name>

# Check lag
kavak kbroker subscription lag <name>

# Rewind (reprocess older messages)
kavak kbroker subscription rewind <name> <hours>

# Redrive DLQ (retry failed messages)
kavak kbroker subscription redrive <name> --max-messages=10
```

## Topic Management

```bash
# Create topic
kavak kbroker topic create

# Create/update schema (required for public topics)
kavak kbroker schema create
kavak kbroker schema update
```

**Topic types:**
- **Public**: Multiple subscribers, requires JSON schema
- **Private**: Few subscribers, schema optional

## CDC (Change Data Capture)

Listen to database changes automatically:

```bash
kavak kbroker connector create  # Select CDC
```

**Auto-generated topic:** `cdc_<product>-<database>_<table>`

**Event format:**
```json
{
  "before": { "id": 1, "name": "old" },
  "after": { "id": 1, "name": "new" },
  "op": "u"  // c=create, u=update, d=delete
}
```

## Dead Letter Queue (DLQ)

Failed messages go to `<subscription>_errors` topic after retries exhausted.

```bash
# Redrive DLQ messages
kavak kbroker subscription redrive my-subscription --max-messages=10
```

## Key Constraints

- Max payload: 64KB
- Min scheduled delay: 5 minutes
- Message TTL: 7 days
- No `$ref` in JSON schemas
