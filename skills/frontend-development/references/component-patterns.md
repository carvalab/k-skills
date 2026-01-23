# Component Patterns

React component patterns for Next.js App Router.

> **Performance**: For memoization, re-render optimization, see `vercel-react-best-practices` skill.

## Server Component (Default)

```typescript
// app/users/page.tsx
export default async function UsersPage() {
    const users = await db.user.findMany();
    return (
        <ul>
            {users.map(user => <li key={user.id}>{user.name}</li>)}
        </ul>
    );
}
```

No `'use client'` needed. This is a Server Component.

## Client Component

```typescript
'use client';

import { useState } from 'react';

export function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**Rule**: Keep `'use client'` components small and leaf-level.

## Props Pattern

```typescript
interface UserCardProps {
    /** User data to display */
    user: User;
    /** Callback when clicked */
    onSelect?: (id: string) => void;
}

export function UserCard({ user, onSelect }: UserCardProps) {
    return (
        <div onClick={() => onSelect?.(user.id)}>
            {user.name}
        </div>
    );
}
```

## Hooks Order

```typescript
'use client';

export function MyComponent({ id }: { id: string }) {
    // 1. Context hooks
    const { user } = useAuth();

    // 2. Action state
    const [state, action, pending] = useActionState(myAction, null);

    // 3. Local state
    const [isOpen, setIsOpen] = useState(false);

    // 4. Effects (avoid when possible)
    useEffect(() => {...}, []);

    // 5. Event handlers
    const handleClick = () => setIsOpen(true);

    // 6. Render
    return <div>...</div>;
}
```

## Composition Pattern

```typescript
// app/dashboard/page.tsx (Server)
import { InteractiveChart } from './InteractiveChart';

export default async function DashboardPage() {
    const stats = await db.stats.get();
    return (
        <div>
            <DashboardStats stats={stats} />  {/* Server */}
            <InteractiveChart />               {/* Client */}
        </div>
    );
}
```

```typescript
// InteractiveChart.tsx (Client)
'use client';

export function InteractiveChart() {
    const [range, setRange] = useState('week');
    return <Chart range={range} onRangeChange={setRange} />;
}
```

## Form Pattern (Server Actions)

```typescript
'use client';

import { useActionState } from 'react';
import { createPost } from './actions';

export function CreatePostForm() {
    const [state, action, pending] = useActionState(createPost, null);

    return (
        <form action={action}>
            <input name='title' required />
            {state?.error && <p className='error'>{state.error.formErrors}</p>}
            <button disabled={pending}>
                {pending ? 'Creating...' : 'Create'}
            </button>
        </form>
    );
}
```

## Dialog Pattern

```typescript
'use client';

import { Dialog, DialogTitle, DialogContent, DialogActions, Button, IconButton } from '@mui/material';
import { Close } from '@mui/icons-material';

interface ConfirmDialogProps {
    open: boolean;
    onClose: () => void;
    onConfirm: () => void;
    title: string;
    children: React.ReactNode;
}

export function ConfirmDialog({ open, onClose, onConfirm, title, children }: ConfirmDialogProps) {
    return (
        <Dialog open={open} onClose={onClose} maxWidth='sm' fullWidth>
            <DialogTitle sx={{ display: 'flex', justifyContent: 'space-between' }}>
                {title}
                <IconButton onClick={onClose} size='small'><Close /></IconButton>
            </DialogTitle>
            <DialogContent>{children}</DialogContent>
            <DialogActions>
                <Button onClick={onClose}>Cancel</Button>
                <Button onClick={onConfirm} variant='contained'>Confirm</Button>
            </DialogActions>
        </Dialog>
    );
}
```

## Loading Pattern

Use `loading.tsx` for automatic Suspense:

```typescript
// app/users/loading.tsx
export default function Loading() {
    return <UserListSkeleton />;
}
```

For granular loading:

```typescript
import { Suspense } from 'react';

export default function Page() {
    return (
        <Suspense fallback={<Skeleton />}>
            <SlowComponent />
        </Suspense>
    );
}
```

## Error Boundary

```typescript
// app/users/error.tsx
'use client';

export default function Error({ error, reset }: { error: Error; reset: () => void }) {
    return (
        <div>
            <h2>Something went wrong!</h2>
            <button onClick={reset}>Try again</button>
        </div>
    );
}
```

## Context Pattern

```typescript
// contexts/site-context.tsx
'use client';

import { createContext, useContext } from 'react';

interface SiteContextData {
    user: User | null;
    locale: string;
    deviceType: 'desktop' | 'mobile' | 'tablet';
}

const SiteContext = createContext<SiteContextData | null>(null);

export function SiteProvider({ children, value }: { children: React.ReactNode; value: SiteContextData }) {
    return <SiteContext.Provider value={value}>{children}</SiteContext.Provider>;
}

export function useSiteContext() {
    const context = useContext(SiteContext);
    if (!context) throw new Error('useSiteContext must be used within SiteProvider');
    return context;
}
```

**Use Context for**: Global config (user, locale, theme). Keep state minimal.

## Export Pattern

```typescript
// Named export (most components)
export function MyComponent() {...}

// Default export (pages/layouts/loading/error)
export default function Page() {...}
```

## When to Split

**Split when**: >200 lines, multiple responsibilities, reusable parts.

**Keep together when**: <150 lines, single responsibility, not reused.
