# Common Simplification Patterns

Patterns to apply when simplifying code.

## Early Returns

Replace nested conditionals with early returns:

```typescript
// Before
function process(data) {
  if (data) {
    if (data.isValid) {
      if (data.items.length > 0) {
        return doWork(data);
      }
    }
  }
  return null;
}

// After
function process(data) {
  if (!data) return null;
  if (!data.isValid) return null;
  if (data.items.length === 0) return null;
  return doWork(data);
}
```

## Object Destructuring

Extract repeated property access:

```typescript
// Before
function render(props) {
  return `${props.user.name} (${props.user.email}) - ${props.user.role}`;
}

// After
function render(props) {
  const { name, email, role } = props.user;
  return `${name} (${email}) - ${role}`;
}
```

## Nullish Coalescing

Replace verbose null checks:

```typescript
// Before
const value = config.timeout !== null && config.timeout !== undefined ? config.timeout : 5000;

// After
const value = config.timeout ?? 5000;
```

## Optional Chaining

Replace nested existence checks:

```typescript
// Before
const city = user && user.address && user.address.city;

// After
const city = user?.address?.city;
```

## Consolidate Conditionals

Combine related conditions:

```typescript
// Before
if (isAdmin) return true;
if (isOwner) return true;
if (isModerator) return true;
return false;

// After
return isAdmin || isOwner || isModerator;
```

## Extract to Variable

Name complex expressions:

```typescript
// Before
if (user.role === 'admin' && user.permissions.includes('write') && !user.suspended) {
  // ...
}

// After
const canWrite = user.role === 'admin' && user.permissions.includes('write') && !user.suspended;

if (canWrite) {
  // ...
}
```

## Replace Ternary Chains

Use switch or if/else for multiple conditions:

```typescript
// Before (hard to read)
const status = code === 200 ? 'ok' : code === 404 ? 'not found' : code === 500 ? 'error' : 'unknown';

// After
function getStatus(code) {
  switch (code) {
    case 200:
      return 'ok';
    case 404:
      return 'not found';
    case 500:
      return 'error';
    default:
      return 'unknown';
  }
}
```

## Array Methods

Replace loops with declarative methods:

```typescript
// Before
const results = [];
for (const item of items) {
  if (item.active) {
    results.push(item.name);
  }
}

// After
const results = items.filter((item) => item.active).map((item) => item.name);
```
