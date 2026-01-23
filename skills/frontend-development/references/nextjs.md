# Next.js App Router

Server-first architecture. Data fetching + mutations built-in.

> **Performance**: For React.cache() deduplication, bundle optimization, waterfalls, memoization → `vercel-react-best-practices` skill.

## Server Components (Default)

```typescript
// app/users/page.tsx - No 'use client' needed
export default async function UsersPage() {
    const users = await db.user.findMany();
    return <UserList users={users} />;
}
```

## Client Components

Add `'use client'` only for interactivity:

```typescript
'use client';

import { useState } from 'react';

export function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**When needed**: `useState`, `useEffect`, `onClick`, browser APIs.

**Rule**: Keep Client Components small and leaf-level.

## Server Actions (Mutations)

```typescript
// app/posts/actions.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';

const Schema = z.object({ title: z.string().min(1) });

export async function createPost(formData: FormData) {
    const parsed = Schema.safeParse({ title: formData.get('title') });
    if (!parsed.success) return { error: parsed.error.flatten() };

    await db.post.create({ data: parsed.data });
    revalidatePath('/posts');
    return { success: true };
}
```

## Forms with useActionState

```typescript
'use client';

import { useActionState } from 'react';
import { createPost } from './actions';

export function CreateForm() {
    const [state, action, pending] = useActionState(createPost, null);

    return (
        <form action={action}>
            <input name='title' required />
            {state?.error && <p>{state.error.formErrors}</p>}
            <button disabled={pending}>
                {pending ? 'Creating...' : 'Create'}
            </button>
        </form>
    );
}
```

## Server Actions vs Route Handlers

| Use Case | Solution |
|----------|----------|
| Form submissions | Server Actions |
| Internal mutations | Server Actions |
| Public APIs | Route Handlers |
| Webhooks | Route Handlers |

## Cache Invalidation

```typescript
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';

export async function updateUser(id: string, data: FormData) {
    await db.user.update({ where: { id }, data: {...} });
    revalidatePath(`/users/${id}`);  // Or: revalidateTag('users');
}
```

## Caching with `use cache`

```typescript
async function getUsers() {
    'use cache';
    cacheTag('users');
    return await db.user.findMany();
}
```

## Parallel Data Fetching

```typescript
export default async function DashboardPage() {
    const [users, posts] = await Promise.all([
        db.user.findMany(),
        db.post.findMany(),
    ]);
    return <Dashboard users={users} posts={posts} />;
}
```

## Streaming with Suspense

```typescript
import { Suspense } from 'react';

export default function Page() {
    return (
        <div>
            <Header />
            <Suspense fallback={<Skeleton />}>
                <SlowData />
            </Suspense>
        </div>
    );
}
```

Or use `loading.tsx` for automatic Suspense.

## File Conventions

```
app/
├── layout.tsx          # Root layout
├── page.tsx            # /
├── loading.tsx         # Suspense fallback
├── error.tsx           # Error boundary ('use client')
├── not-found.tsx       # 404
├── users/
│   ├── page.tsx        # /users
│   ├── [id]/page.tsx   # /users/:id
│   └── actions.ts      # Server Actions
└── api/webhook/route.ts
```

## Dynamic Routes

```typescript
// app/users/[id]/page.tsx
export default async function UserPage({
    params,
}: {
    params: Promise<{ id: string }>;
}) {
    const { id } = await params;
    const user = await db.user.findUnique({ where: { id } });
    if (!user) notFound();
    return <UserProfile user={user} />;
}
```

## Metadata

```typescript
import type { Metadata } from 'next';

export async function generateMetadata({ params }): Promise<Metadata> {
    const { id } = await params;
    const user = await db.user.findUnique({ where: { id } });
    return { title: user?.name ?? 'User' };
}
```

## Route Segment Config

```typescript
export const dynamic = 'force-dynamic';  // Always SSR
export const revalidate = 60;            // ISR: 60s
```

## Middleware

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
    // Auth check
    const token = request.cookies.get('token');
    if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
        return NextResponse.redirect(new URL('/login', request.url));
    }

    // Inject headers for Server Components (access via headers())
    const requestHeaders = new Headers(request.headers);
    requestHeaders.set('x-pathname', request.nextUrl.pathname);
    requestHeaders.set('x-device-type', getDeviceType(request));

    return NextResponse.next({ request: { headers: requestHeaders } });
}

export const config = { matcher: ['/((?!api|_next|.*\\..*).*)'] };
```

## Dynamic Component Registry

For CMS-driven content, map component IDs to dynamic imports:

```typescript
// components-registry.ts
import dynamic from 'next/dynamic';

const COMPONENTS: Record<string, React.ComponentType<any>> = {
    'hero-banner': dynamic(() => import('@/features/hero/HeroBanner')),
    'product-grid': dynamic(() => import('@/features/products/ProductGrid')),
};

// content.tsx
export function DynamicContent({ sections }: { sections: Section[] }) {
    return sections.map(({ id, componentId, props }) => {
        const Component = COMPONENTS[componentId];
        return Component ? <Component key={id} {...props} /> : null;
    });
}
```

## Environment Variables

```typescript
// Server-only (safe)
const dbUrl = process.env.DATABASE_URL;

// Client (must prefix NEXT_PUBLIC_)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

**Never expose non-NEXT_PUBLIC_ vars to client.**
