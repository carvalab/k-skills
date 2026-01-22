# Backend Performance & Scalability

Caching, query optimization, and scaling patterns (2026).

## PostgreSQL Performance

### Indexing (30% I/O reduction, 10-100x speedup)

```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (order matters)
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);

-- Partial index (filtered queries)
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM orders
WHERE user_id = 123 AND created_at > '2026-01-01';
```

**Index Types:**
- **B-tree** - Default, equality/range queries
- **Hash** - Fast equality, no range
- **GIN** - Full-text, JSONB
- **GiST** - Geospatial

**Don't Index:**
- Small tables (<1000 rows)
- Frequently updated columns
- Low-cardinality columns (boolean)

### Connection Pooling (5-10x improvement)

```typescript
import { Pool } from 'pg';

const pool = new Pool({
  host: process.env.DB_HOST,
  max: 20,
  min: 5,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

const result = await pool.query('SELECT * FROM users WHERE id = $1', [userId]);
```

**Pool Size:** `connections = (core_count * 2) + effective_spindle_count`

### N+1 Query Problem

```typescript
// BAD: N+1 queries
const posts = await Post.findAll();
for (const post of posts) {
  post.author = await User.findById(post.authorId); // N queries!
}

// GOOD: Single query with JOIN
const posts = await Post.findAll({
  include: [{ model: User, as: 'author' }],
});
```

## Redis Caching (90% DB load reduction)

### Cache-Aside Pattern

```typescript
async function getUser(userId: string) {
  const cached = await redis.get(`user:${userId}`);
  if (cached) return JSON.parse(cached);

  const user = await db.users.findById(userId);
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user)); // TTL: 1 hour
  return user;
}
```

### Write-Through Pattern

```typescript
async function updateUser(userId: string, data: UpdateUserDto) {
  const user = await db.users.update(userId, data);
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
  return user;
}
```

### Cache Invalidation

```typescript
async function deleteUser(userId: string) {
  await db.users.delete(userId);
  await redis.del(`user:${userId}`);
  await redis.del(`user:${userId}:posts`);
}
```

### Cache Best Practices

1. Cache frequently accessed data
2. Set appropriate TTL
3. Invalidate on write
4. Use key patterns: `resource:id:attribute`
5. Monitor hit rates (target >80%)

## Caching Layers

```
Client → CDN (static assets) → API Gateway → Redis → PostgreSQL
```

## Load Balancing

### Algorithms

```nginx
upstream backend {
    # Round Robin (default)
    server backend1.example.com;
    server backend2.example.com;

    # Least Connections
    least_conn;

    # IP Hash (session affinity)
    ip_hash;
}
```

### Health Checks

```typescript
app.get('/health', async (req, res) => {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
  };
  const isHealthy = checks.database && checks.redis;
  res.status(isHealthy ? 200 : 503).json(checks);
});
```

## Async Processing

### Bull (Redis-based queues)

```typescript
import Queue from 'bull';

const emailQueue = new Queue('email', { redis: { host: 'localhost' } });

// Producer
await emailQueue.add('send-welcome', { userId: user.id, email: user.email });

// Consumer
emailQueue.process('send-welcome', async (job) => {
  await sendWelcomeEmail(job.data.email);
});
```

## Scaling Patterns

### Horizontal Scaling

```
Load Balancer
    ↓
┌───┴───┬───────┬───────┐
│ App 1 │ App 2 │ App 3 │
└───────┴───────┴───────┘
         ↓
    PostgreSQL (+ replicas)
```

### Read Replicas

```
Primary (Write) → Replica 1 (Read)
               → Replica 2 (Read)
```

```typescript
await primaryDb.users.create(userData);  // Write
const users = await replicaDb.users.findAll();  // Read
```

## Performance Checklist

### Database
- [ ] Indexes on query columns
- [ ] Connection pooling
- [ ] N+1 queries eliminated
- [ ] Slow query monitoring
- [ ] EXPLAIN ANALYZE reviewed

### Caching
- [ ] Redis for hot data
- [ ] TTL configured
- [ ] Invalidation on writes
- [ ] >80% hit rate

### Application
- [ ] Async processing for long tasks
- [ ] Response compression (gzip)
- [ ] Load balancing
- [ ] Health checks
- [ ] Resource limits

## Common Pitfalls

1. No caching
2. Missing indexes
3. N+1 queries
4. Synchronous long tasks
5. No connection pooling
6. Unbounded queries (no LIMIT)
