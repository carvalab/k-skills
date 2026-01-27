# Node.js Database Access

**New projects:** Drizzle (best performance).
**Existing projects:** Keep current ORM.

## Drizzle (Recommended for New)

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { pgTable, text, timestamp, uuid } from 'drizzle-orm/pg-core';
import { eq, ilike } from 'drizzle-orm';
import { Pool } from 'pg';

// Schema definition
export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: text('email').notNull().unique(),
  name: text('name').notNull(),
  createdAt: timestamp('created_at').defaultNow(),
});

export const posts = pgTable('posts', {
  id: uuid('id').primaryKey().defaultRandom(),
  title: text('title').notNull(),
  content: text('content'),
  authorId: uuid('author_id').references(() => users.id),
  createdAt: timestamp('created_at').defaultNow(),
});

// Setup
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const db = drizzle(pool);

// CRUD operations
const user = await db.insert(users).values({ email: 'test@example.com', name: 'Test' }).returning();

const found = await db.select().from(users).where(eq(users.email, email));

const usersWithPosts = await db.select().from(users).leftJoin(posts, eq(users.id, posts.authorId)).where(ilike(users.email, '%@example.com'));

await db.update(users).set({ name: 'Updated' }).where(eq(users.id, userId));

await db.delete(users).where(eq(users.id, userId));

// Transactions
await db.transaction(async (tx) => {
  await tx.insert(users).values({ email, name });
  await tx.insert(posts).values({ title, authorId });
});
```

## Prisma (Existing Projects)

Keep using if already in project.

```typescript
// schema.prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  posts     Post[]
  createdAt DateTime @default(now())
}

// Usage
const user = await prisma.user.create({
  data: { email: 'test@example.com', name: 'Test' },
});

const usersWithPosts = await prisma.user.findMany({
  include: { posts: true },
  where: { email: { contains: '@example.com' } },
});
```

## Raw Queries (pg)

For complex queries not suited for ORM.

```typescript
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
});

// Parameterized query (safe)
const result = await pool.query('SELECT * FROM users WHERE email = $1', [email]);
```

## Caching (Redis + Fallback)

```typescript
import { createClient } from 'redis';
import NodeCache from 'node-cache';

class CacheService {
  private redis: ReturnType<typeof createClient>;
  private localCache = new NodeCache({ stdTTL: 60 });

  async get<T>(key: string): Promise<T | null> {
    const local = this.localCache.get<T>(key);
    if (local) return local;

    try {
      const value = await this.redis.get(key);
      if (value) {
        const parsed = JSON.parse(value) as T;
        this.localCache.set(key, parsed);
        return parsed;
      }
    } catch {
      // Redis unavailable
    }
    return null;
  }

  async set(key: string, value: unknown, ttlSeconds = 300): Promise<void> {
    this.localCache.set(key, value, ttlSeconds);
    try {
      await this.redis.setEx(key, ttlSeconds, JSON.stringify(value));
    } catch {
      // Redis unavailable, local only
    }
  }
}
```
