# Common Issues and Fixes

## Error Handling

### Missing Error Handling

**Issue**: Async operations without try/catch or .catch()
```typescript
// Bad
const data = await fetchData();
processData(data);

// Good
try {
  const data = await fetchData();
  processData(data);
} catch (error) {
  console.error('Failed to fetch data:', error);
  throw error; // or handle gracefully
}
```

### Silent Error Swallowing

**Issue**: Catching errors without logging or handling
```typescript
// Bad
try {
  await riskyOperation();
} catch {
  // silently ignored
}

// Good
try {
  await riskyOperation();
} catch (error) {
  console.error('Operation failed:', error);
  // Either re-throw or return a meaningful default
}
```

---

## TypeScript Issues

### Using `any` Type

**Issue**: Defeats TypeScript's purpose
```typescript
// Bad
function process(data: any) { ... }

// Good
function process(data: ProcessInput) { ... }
// or if truly unknown:
function process(data: unknown) { ... }
```

### Missing Null Checks

**Issue**: Accessing properties on potentially null values
```typescript
// Bad
const name = user.profile.name;

// Good
const name = user?.profile?.name ?? 'Unknown';
```

---

## Performance Issues

### N+1 Queries

**Issue**: Querying in a loop
```typescript
// Bad
for (const user of users) {
  const posts = await db.query('SELECT * FROM posts WHERE user_id = ?', user.id);
}

// Good
const userIds = users.map(u => u.id);
const posts = await db.query('SELECT * FROM posts WHERE user_id IN (?)', userIds);
```

### Unnecessary Re-renders (React)

**Issue**: Missing memoization or dependency arrays
```typescript
// Bad
const Component = ({ items }) => {
  const sorted = items.sort(); // sorts on every render
  return <List items={sorted} />;
};

// Good
const Component = ({ items }) => {
  const sorted = useMemo(() => [...items].sort(), [items]);
  return <List items={sorted} />;
};
```

---

## Security Issues

### SQL Injection

**Issue**: String concatenation in queries
```typescript
// Bad
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// Good
const query = db.query('SELECT * FROM users WHERE id = ?', [userId]);
```

### Exposed Secrets

**Issue**: Hardcoded credentials
```typescript
// Bad
const API_KEY = 'sk-1234567890';

// Good
const API_KEY = process.env.API_KEY;
```

---

## Code Quality

### Deep Nesting

**Issue**: Hard to read nested conditions
```typescript
// Bad
if (user) {
  if (user.isActive) {
    if (user.hasPermission) {
      doSomething();
    }
  }
}

// Good (early returns)
if (!user) return;
if (!user.isActive) return;
if (!user.hasPermission) return;
doSomething();
```

### Magic Numbers

**Issue**: Unexplained numeric values
```typescript
// Bad
if (retries > 3) { ... }
setTimeout(fn, 86400000);

// Good
const MAX_RETRIES = 3;
const ONE_DAY_MS = 24 * 60 * 60 * 1000;
if (retries > MAX_RETRIES) { ... }
setTimeout(fn, ONE_DAY_MS);
```

### Duplicated Code

**Issue**: Same logic repeated
```typescript
// Bad
const userAge = Math.floor((Date.now() - user.birthDate) / 31536000000);
const accountAge = Math.floor((Date.now() - account.createdAt) / 31536000000);

// Good
function yearsAgo(date: Date): number {
  const MS_PER_YEAR = 365.25 * 24 * 60 * 60 * 1000;
  return Math.floor((Date.now() - date.getTime()) / MS_PER_YEAR);
}
const userAge = yearsAgo(user.birthDate);
const accountAge = yearsAgo(account.createdAt);
```

---

## Async/Await Issues

### Missing await

**Issue**: Promise not awaited
```typescript
// Bad
async function save() {
  db.insert(data); // Promise ignored
  return { success: true };
}

// Good
async function save() {
  await db.insert(data);
  return { success: true };
}
```

### Sequential When Parallel Is Possible

**Issue**: Unnecessary sequential execution
```typescript
// Bad
const user = await getUser(id);
const posts = await getPosts(id);
const comments = await getComments(id);

// Good (if independent)
const [user, posts, comments] = await Promise.all([
  getUser(id),
  getPosts(id),
  getComments(id),
]);
```

---

## Test Issues

### Testing Implementation, Not Behavior

**Issue**: Tests break on refactoring
```typescript
// Bad
expect(component.state.count).toBe(1);

// Good
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

### No Error Path Tests

**Issue**: Only testing happy path
```typescript
// Bad
test('creates user', async () => {
  const user = await createUser({ name: 'Test' });
  expect(user.id).toBeDefined();
});

// Good - also test error cases
test('fails with invalid name', async () => {
  await expect(createUser({ name: '' })).rejects.toThrow('Name required');
});
```
