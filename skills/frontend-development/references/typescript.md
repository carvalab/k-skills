# TypeScript Standards

TypeScript best practices for React frontend code with strict mode enabled.

## Strict Mode Requirements

Project uses strict TypeScript:
- No implicit `any`
- Null/undefined must be handled
- Type safety enforced

## No `any` Type

```typescript
// ❌ Never use any
function handleData(data: any) { return data.something; }

// ✅ Use specific types
interface MyData { something: string; }
function handleData(data: MyData) { return data.something; }

// ✅ Use unknown for truly unknown data
function handleUnknown(data: unknown) {
    if (typeof data === 'object' && data !== null && 'something' in data) {
        return (data as MyData).something;
    }
}
```

## Explicit Return Types

```typescript
// ✅ Explicit return types
function getUser(id: number): Promise<User> {
    return api.get(`/users/${id}`);
}

function calculateTotal(items: Item[]): number {
    return items.reduce((sum, item) => sum + item.price, 0);
}

// Custom hooks
function useMyData(id: number): { data: Data; isLoading: boolean } {
    // ...
}
```

## Type Imports

Use `type` keyword for type-only imports:

```typescript
// ✅ Clear type import
import type { User } from '~types/user';
import type { SxProps, Theme } from '@mui/material';

// ❌ Ambiguous (is it a type or value?)
import { User } from '~types/user';
```

**Benefits:**
- Clearer code intent
- Better tree-shaking
- Prevents circular dependencies

## Component Props

```typescript
interface MyComponentProps {
    /** User ID to display */
    userId: number;

    /** Callback when action completes */
    onComplete?: () => void;

    /** Display mode */
    mode?: 'view' | 'edit';
}

export const MyComponent: React.FC<MyComponentProps> = ({
    userId,
    onComplete,
    mode = 'view',  // Default value
}) => {
    // ...
};
```

**Key Points:**
- JSDoc comments for each prop
- Optional props use `?`
- Defaults in destructuring

## Utility Types

```typescript
// Partial<T> - Make all properties optional
type UserUpdate = Partial<User>;

// Pick<T, K> - Select specific properties
type UserPreview = Pick<User, 'id' | 'name' | 'email'>;

// Omit<T, K> - Exclude properties
type UserWithoutPassword = Omit<User, 'password'>;

// Record<K, V> - Type-safe map
const styles: Record<string, SxProps<Theme>> = { container: { p: 2 } };

// Required<T> - Make all required
type RequiredConfig = Required<Config>;
```

## Type Guards

### Basic Type Guard

```typescript
function isUser(data: unknown): data is User {
    return (
        typeof data === 'object' &&
        data !== null &&
        'id' in data &&
        'name' in data
    );
}

if (isUser(response)) {
    console.log(response.name);  // TypeScript knows it's User
}
```

### Discriminated Unions

```typescript
type LoadingState =
    | { status: 'idle' }
    | { status: 'loading' }
    | { status: 'success'; data: Data }
    | { status: 'error'; error: Error };

function Component({ state }: { state: LoadingState }) {
    if (state.status === 'success') {
        return <Display data={state.data} />;  // data available
    }
    if (state.status === 'error') {
        return <Error error={state.error} />;  // error available
    }
    return <Loading />;
}
```

## Generics

### Generic Functions

```typescript
function getById<T extends { id: number }>(items: T[], id: number): T | undefined {
    return items.find(item => item.id === id);
}

const user = getById(users, 123);  // Type: User | undefined
```

### Generic Components

```typescript
interface ListProps<T> {
    items: T[];
    renderItem: (item: T) => React.ReactNode;
}

export function List<T>({ items, renderItem }: ListProps<T>): React.ReactElement {
    return <div>{items.map((item, i) => <div key={i}>{renderItem(item)}</div>)}</div>;
}

// Usage: <List<User> items={users} renderItem={(user) => <UserCard user={user} />} />
```

## Null/Undefined Handling

### Optional Chaining

```typescript
const name = user?.profile?.name;
```

### Nullish Coalescing

```typescript
const displayName = user?.name ?? 'Anonymous';

// ?? triggers on null/undefined only
// || triggers on '', 0, false too
```

### Non-Null Assertion (Use Carefully)

```typescript
// ✅ When you're certain value exists
const data = queryClient.getQueryData<Data>(['data'])!;

// ⚠️ Better: check explicitly
const data = queryClient.getQueryData<Data>(['data']);
if (data) { /* use data */ }
```

## Type Assertions (Sparingly)

```typescript
// ✅ When you know more than TypeScript
const element = document.getElementById('my-input') as HTMLInputElement;

// ❌ Never circumvent type safety
const data = getData() as any;
```

## Event Types

```typescript
// Form events
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
};

// Input events
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
};

// Click events
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    e.stopPropagation();
};
```

## API Response Types

```typescript
interface ApiResponse<T> {
    data: T;
    status: number;
    message?: string;
}

async function fetchUser(id: string): Promise<User> {
    const response = await apiClient.get<User>(`/users/${id}`);
    return response.data;
}
```

## Zod Runtime Validation

```typescript
import { z } from 'zod';

// Define schema
const UserSchema = z.object({
    name: z.string().min(1),
    email: z.string().email(),
    age: z.number().optional(),
});

// Infer type from schema
type User = z.infer<typeof UserSchema>;

// Validate (throws on error)
const user = UserSchema.parse(data);

// Safe validate (returns result object)
const result = UserSchema.safeParse(data);
if (result.success) {
    console.log(result.data);  // Typed as User
} else {
    console.log(result.error.flatten());
}
```

**Use Zod for**: Server Action inputs, API responses, form data, environment variables.

## Enums vs Union Types

```typescript
// ✅ Prefer union types (tree-shakable, simpler)
type Status = 'pending' | 'active' | 'completed';

// ❌ Avoid enums (not tree-shakable)
enum Status { Pending = 'pending', Active = 'active' }
```
