# Common Dead Code Patterns

Patterns to identify and remove.

## Unused Imports

```typescript
// ❌ Before: unused imports
import { useState, useEffect, useMemo, useCallback } from 'react'
// Only useState is actually used

// ✅ After: only used imports
import { useState } from 'react'
```

Detection:
```bash
npx eslint . --rule 'no-unused-vars: error'
```

## Unused Exports

```typescript
// ❌ Before: exported but never imported
export function unusedHelper() {
  return 'never called'
}

export const UNUSED_CONSTANT = 'never used'

// ✅ After: removed entirely
// (file deleted or exports removed)
```

Detection:
```bash
npx ts-prune
```

## Dead Code Branches

```typescript
// ❌ Before: unreachable code
if (false) {
  doSomething() // never executes
}

if (process.env.NODE_ENV === 'development' && false) {
  debugCode() // never executes
}

// ✅ After: removed entirely
```

## Commented-Out Code

```typescript
// ❌ Before: commented code blocks
function processData(data) {
  // Old implementation - keeping just in case
  // const result = oldProcess(data)
  // return transform(result)

  return newProcess(data)
}

// ✅ After: comments removed (git has history)
function processData(data) {
  return newProcess(data)
}
```

## Duplicate Components

```
// ❌ Before: multiple similar components
components/
├── Button.tsx
├── PrimaryButton.tsx
├── SecondaryButton.tsx
└── NewButton.tsx

// ✅ After: consolidated with variants
components/
└── Button.tsx  // with variant prop
```

Consolidation pattern:
```typescript
// Single component with variants
type ButtonProps = {
  variant?: 'primary' | 'secondary'
  // ... other props
}

function Button({ variant = 'primary', ...props }: ButtonProps) {
  // Handle variants internally
}
```

## Unused Dependencies

```json
// ❌ Before: installed but not imported
{
  "dependencies": {
    "lodash": "^4.17.21",    // Not used
    "moment": "^2.29.4",     // Replaced by date-fns
    "axios": "^1.6.0"        // Replaced by fetch
  }
}

// ✅ After: removed
{
  "dependencies": {
    "date-fns": "^3.0.0"
  }
}
```

Detection:
```bash
npx depcheck
```

## Unused Type Definitions

```typescript
// ❌ Before: types defined but never used
interface OldUserResponse {
  legacy: boolean
  data: unknown
}

type DeprecatedConfig = {
  oldField: string
}

// ✅ After: removed entirely
```

Detection:
```bash
npx knip --include types
```

## Empty or Stub Files

```typescript
// ❌ Before: placeholder file
// src/utils/placeholder.ts
export {}

// Or file with only comments
/**
 * TODO: Implement this later
 */

// ✅ After: deleted
```

## Unused CSS/Styles

```css
/* ❌ Before: classes not referenced in code */
.old-component {
  display: none;
}

.legacy-button {
  background: red;
}

/* ✅ After: removed */
```

Detection:
```bash
# Search for class usage
grep -r "old-component" src/ --include="*.tsx"
```

## Feature Flags (Expired)

```typescript
// ❌ Before: expired feature flag
if (featureFlags.newCheckout) {
  // This is always true now
  return <NewCheckout />
}
return <OldCheckout />

// ✅ After: removed flag check
return <NewCheckout />
// Also delete OldCheckout component
```

## Removal Priority

1. **Unused dependencies** - Immediate bundle size win
2. **Unused files** - Clear wins, easy to verify
3. **Commented code** - Git has history
4. **Unused exports** - Verify no dynamic usage first
5. **Duplicate code** - Requires careful consolidation
