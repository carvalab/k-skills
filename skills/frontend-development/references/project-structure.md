# Project Structure

File organization for Next.js App Router applications.

## Top-Level Structure

```
├── app/                # Next.js App Router
├── components/         # Reusable UI components
├── features/           # Domain-specific features
├── lib/                # Utilities, API clients
├── types/              # Shared TypeScript types
├── hooks/              # Shared hooks
└── public/             # Static assets
```

## app/ Directory (Routes)

```
app/
├── layout.tsx          # Root layout
├── page.tsx            # Home (/)
├── loading.tsx         # Global loading
├── error.tsx           # Global error
├── not-found.tsx       # 404
├── providers.tsx       # Client providers
├── globals.css         # Global styles
├── users/
│   ├── layout.tsx      # /users layout
│   ├── page.tsx        # /users
│   ├── loading.tsx     # /users loading
│   ├── [id]/
│   │   └── page.tsx    # /users/:id
│   └── actions.ts      # Server Actions
├── (auth)/             # Route group (no URL)
│   ├── login/page.tsx
│   └── register/page.tsx
└── api/
    └── webhook/route.ts
```

## features/ vs components/

| Directory | Purpose | Examples |
|-----------|---------|----------|
| `features/` | Domain-specific with logic, API, state | users, posts, auth |
| `components/` | Generic, reusable UI primitives | Button, Card, Modal |

### features/

Use for domain-specific code:

```
features/
├── users/
│   ├── api/
│   │   └── usersApi.ts      # API service
│   ├── components/
│   │   ├── UserCard.tsx
│   │   └── UserList.tsx
│   ├── hooks/
│   │   └── useUsers.ts
│   ├── types/
│   │   └── index.ts
│   └── index.ts             # Public exports
└── posts/
```

### components/

Use for truly reusable UI:

```
components/
├── ui/
│   ├── Button.tsx
│   ├── Card.tsx
│   └── Modal.tsx
├── layout/
│   ├── Header.tsx
│   └── Sidebar.tsx
└── forms/
    └── FormField.tsx
```

## Feature Structure

```
features/users/
├── api/
│   └── usersApi.ts          # API calls
├── components/
│   ├── UserCard.tsx         # Server or Client
│   ├── UserList.tsx
│   └── UserForm.tsx         # 'use client'
├── hooks/
│   └── useUsers.ts          # TanStack Query hooks
├── types/
│   └── index.ts             # TypeScript types
├── actions.ts               # Server Actions
└── index.ts                 # Public exports
```

### Public Exports (index.ts)

```typescript
// features/users/index.ts
export { UserCard } from './components/UserCard';
export { UserList } from './components/UserList';
export { useUsers } from './hooks/useUsers';
export { usersApi } from './api/usersApi';
export type { User, CreateUserInput } from './types';
```

**Usage:**
```typescript
import { UserCard, useUsers } from '@/features/users';
```

## Import Aliases

Configure in `tsconfig.json`:

```json
{
    "compilerOptions": {
        "paths": {
            "@/*": ["./src/*"],
            "@/components/*": ["./src/components/*"],
            "@/features/*": ["./src/features/*"],
            "@/lib/*": ["./src/lib/*"],
            "@/types/*": ["./src/types/*"]
        }
    }
}
```

**Usage:**
```typescript
import { Button } from '@/components/ui/Button';
import { usersApi } from '@/features/users';
import { apiClient } from '@/lib/apiClient';
import type { User } from '@/types/user';
```

## File Naming

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `UserCard.tsx` |
| Hooks | camelCase, `use` prefix | `useUsers.ts` |
| API services | camelCase | `usersApi.ts` |
| Server Actions | camelCase | `actions.ts` |
| Types | camelCase | `types/index.ts` |
| Pages | lowercase | `page.tsx` |

## Server Actions Location

**Collocated** (preferred for single feature):
```
app/users/
├── page.tsx
└── actions.ts       # Actions for /users routes
```

**Feature-based** (for reusable actions):
```
features/users/
├── actions.ts       # Reusable user actions
└── ...
```

## lib/ Directory

```
lib/
├── apiClient.ts     # Axios/fetch wrapper
├── db.ts            # Database client
├── auth.ts          # Auth utilities
└── utils.ts         # General utilities
```

## types/ Directory

```
types/
├── user.ts          # User-related types
├── post.ts          # Post-related types
├── api.ts           # API response types
└── index.ts         # Re-exports
```

## When to Create a Feature

**Create feature when:**
- Multiple related components (3+)
- Has own API endpoints
- Domain-specific logic
- Will grow over time

**Keep in components/ when:**
- Used across 3+ features
- Generic, no domain logic
- Pure presentation

## Import Order

```typescript
// 1. React/Next
import { useState } from 'react';
import { notFound } from 'next/navigation';

// 2. Third-party
import { z } from 'zod';

// 3. Aliases
import { Button } from '@/components/ui/Button';
import { usersApi } from '@/features/users';

// 4. Types
import type { User } from '@/types/user';

// 5. Relative
import { UserCard } from './UserCard';
```
