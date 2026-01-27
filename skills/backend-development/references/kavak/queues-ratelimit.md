# Internal Queues & Rate Limiting

Job queues and rate limiting for Kavak microservices.

## Architecture: Redis Buffer → PostgreSQL Queue

**Pattern:** Buffer in Redis for spikes → Batch insert to PostgreSQL → Process with queue.

```
Webhook/API → Redis List (buffer) → Batch Processor → PostgreSQL (River/pg-boss) → Workers
                    ↓                      ↓
              Handle spikes         COPY FROM (bulk insert)
```

**Why this pattern?**

- Redis handles traffic spikes (fast writes)
- PostgreSQL stores jobs durably (ACID, retries)
- Batch insert via COPY FROM is efficient
- Reference: `growth-user-notification-api` (Go implementation)

## Technology Selection

| Need          | Go                     | Node.js              |
| ------------- | ---------------------- | -------------------- |
| Job queue     | River Queue            | pg-boss              |
| Spike buffer  | Redis list             | Redis list           |
| Batch insert  | River `InsertManyFast` | pg-boss `send` batch |
| Rate limiting | River snooze           | pg-boss throttle     |

**Note:** Don't use BullMQ. Use pg-boss with Redis buffer pattern for consistency.

## River Queue (Go)

PostgreSQL-backed job queue. Use for background jobs, not real-time events.

```go
import "github.com/riverqueue/river"

// Define job args
type SendEmailArgs struct {
    UserID  string `json:"user_id"`
    Subject string `json:"subject"`
}

func (SendEmailArgs) Kind() string { return "send_email" }

// Define worker
type SendEmailWorker struct {
    river.WorkerDefaults[SendEmailArgs]
    emailService EmailService
}

func (w *SendEmailWorker) Work(ctx context.Context, job *river.Job[SendEmailArgs]) error {
    return w.emailService.Send(ctx, job.Args.UserID, job.Args.Subject)
}

// Insert job
client.Insert(ctx, SendEmailArgs{UserID: "123", Subject: "Welcome"}, nil)
```

### Queue Configuration

```go
riverClient, err := river.NewClient(riverpgxv5.New(pool), &river.Config{
    Queues: map[string]river.QueueConfig{
        "default":       {MaxWorkers: 10},
        "high_priority": {MaxWorkers: 1},
        "notification":  {MaxWorkers: 5},
    },
    JobTimeout:        10 * time.Minute,
    MaxAttempts:       3,
    RescueStuckJobsAfter: 1 * time.Hour,
})
```

## Rate Limiting with Snooze

For external APIs with rate limits (WhatsApp, email providers):

```go
func (w *WhatsAppWorker) Work(ctx context.Context, job *river.Job[WhatsAppArgs]) error {
    // Check rate limit before calling external API
    blocksAhead, err := w.rateLimiter.Limit(ctx, fmt.Sprint(job.ID))
    if err == limiter.ErrLimitExhausted {
        // Snooze job until rate limit window passes
        snoozeDuration := time.Duration(blocksAhead) * 60 * time.Second
        return river.JobSnooze(snoozeDuration)
    }

    // Rate limit OK, call external API
    return w.whatsappClient.Send(ctx, job.Args)
}
```

### Redis Rate Limiter

```go
type RateLimiter struct {
    redis    *redis.Client
    capacity int           // Max requests per window
    window   time.Duration // Time window (e.g., 60s)
}

func (r *RateLimiter) Limit(ctx context.Context, key string) (blocksAhead int, err error) {
    // Add to sorted set with TTL
    now := time.Now().Unix()
    r.redis.ZAdd(ctx, key, redis.Z{Score: float64(now), Member: uuid.New()})
    r.redis.Expire(ctx, key, r.window)

    // Count requests in window
    count := r.redis.ZCount(ctx, key, fmt.Sprint(now-int64(r.window.Seconds())), "+inf")

    if count > int64(r.capacity) {
        blocksAhead = int((count + int64(r.capacity) - 1) / int64(r.capacity))
        return blocksAhead, ErrLimitExhausted
    }
    return 0, nil
}
```

**Common limits:**

- WhatsApp Global: 20,000/minute
- Default API: 5,000/minute

## High-Volume Buffer Pattern

For webhooks receiving thousands of events/second, buffer in Redis before River:

```
Webhook → Redis List → Batch Processor (periodic) → River Queue → Workers
```

### Redis Buffer

```go
// Insert to Redis buffer (fast, non-blocking)
func (q *QueueService) Insert(ctx context.Context, args river.JobArgs) error {
    wrapper := JobWrapper{
        Kind:    args.Kind(),
        Payload: args,
    }
    data, _ := json.Marshal(wrapper)
    return q.redis.RPush(ctx, "queue:batch:processor", data).Err()
}
```

### Batch Processor (Periodic Job)

```go
func (w *BatchWorker) Work(ctx context.Context, job *river.Job[BatchArgs]) error {
    pipe := w.redis.Pipeline()

    // Atomic: get batch + remove from list
    getCmd := pipe.LRange(ctx, "queue:batch:processor", 0, 49999)
    pipe.LTrim(ctx, "queue:batch:processor", 50000, -1)
    pipe.Exec(ctx)

    items := getCmd.Val()
    if len(items) == 0 {
        return nil
    }

    // Deserialize and insert to River
    var riverJobs []river.InsertManyParams
    for _, item := range items {
        var wrapper JobWrapper
        json.Unmarshal([]byte(item), &wrapper)
        args := w.registry.GetArgsFactory(wrapper.Kind)()
        json.Unmarshal(wrapper.Payload, args)
        riverJobs = append(riverJobs, river.InsertManyParams{Args: args})
    }

    _, err := w.riverClient.InsertManyFast(ctx, riverJobs)
    if err != nil {
        // Restore to Redis on failure
        w.restoreToRedis(ctx, items)
        return err
    }
    return nil
}
```

### Periodic Job Setup

```go
river.AddWorker(workers, &BatchWorker{})

periodicJobs := []*river.PeriodicJob{
    river.NewPeriodicJob(
        river.PeriodicInterval(5*time.Second),
        func() (river.JobArgs, *river.InsertOpts) {
            return BatchArgs{}, &river.InsertOpts{Queue: "high_priority"}
        },
        nil,
    ),
}
```

## When to Use What

| Scenario                     | Go                   | Node.js                |
| ---------------------------- | -------------------- | ---------------------- |
| Low-volume jobs              | River Queue directly | pg-boss directly       |
| High-volume webhook          | Redis buffer → River | Redis buffer → pg-boss |
| External API with rate limit | River snooze         | pg-boss throttle       |
| Event to other services      | kbroker              | kbroker                |
| Scheduled/cron task          | River periodic       | pg-boss schedule       |

## pg-boss (Node.js)

PostgreSQL-backed queue. Use with Redis buffer for high volume.

```typescript
import { PgBoss } from 'pg-boss';

const boss = new PgBoss('postgres://user:pass@host/database');
boss.on('error', console.error);
await boss.start();

// Create queue with retry config
await boss.createQueue('email-send', {
  retryLimit: 3,
  retryDelay: 30,
  retryBackoff: true,
  deadLetter: 'email-dlq',
});

// Direct send (low volume)
await boss.send('email-send', { email: 'user@example.com' });

// Worker
await boss.work('email-send', { batchSize: 10 }, async (jobs) => {
  for (const job of jobs) {
    await sendEmail(job.data);
  }
});

// Throttle: 1 job per user per hour
await boss.sendThrottled(
  'user-notification',
  { message: 'Update' },
  {},
  3600, // 1 hour window
  userId // Throttle key
);

// Cron scheduling
await boss.schedule('daily-report', '0 8 * * *', { type: 'daily' });
```

## Redis Buffer Pattern (Node.js)

For high-volume webhooks, buffer in Redis first (same pattern as Go).

```typescript
import { Redis } from 'ioredis';
import { PgBoss } from 'pg-boss';

const redis = new Redis();
const boss = new PgBoss(pgConnection);
const BUFFER_KEY = 'queue:batch:processor';

// 1. Webhook handler: Push to Redis (fast)
async function handleWebhook(data: any) {
  const wrapper = JSON.stringify({ kind: 'email-send', payload: data });
  await redis.rpush(BUFFER_KEY, wrapper);
}

// 2. Batch processor: Redis → pg-boss (runs periodically)
async function processBatch() {
  const pipe = redis.pipeline();
  pipe.lrange(BUFFER_KEY, 0, 49999); // Get up to 50k items
  pipe.ltrim(BUFFER_KEY, 50000, -1); // Remove processed
  const [[, items]] = await pipe.exec();

  if (!items?.length) return;

  // Batch insert to pg-boss
  for (const item of items as string[]) {
    const { kind, payload } = JSON.parse(item);
    await boss.send(kind, payload);
  }
}

// 3. Run batch processor every 5 seconds
setInterval(processBatch, 5000);
```
