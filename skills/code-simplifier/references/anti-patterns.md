# Anti-Patterns to Refactor

Code smells that indicate simplification opportunity.

## Deeply Nested Code

**Symptom**: 3+ levels of indentation

**Solution**: Early returns, extract functions

```typescript
// Problem
if (a) {
  if (b) {
    if (c) {
      doSomething();
    }
  }
}

// Solution
if (!a || !b || !c) return;
doSomething();
```

## Repeated Property Access

**Symptom**: Same object.property accessed 3+ times

**Solution**: Destructure or assign to variable

## Boolean Blindness

**Symptom**: Functions returning/accepting multiple booleans

**Solution**: Use named constants or objects

```typescript
// Problem
createUser(true, false, true);

// Solution
createUser({
  isAdmin: true,
  sendEmail: false,
  verified: true,
});
```

## Premature Abstraction

**Symptom**: Abstraction used only once

**Solution**: Inline the code

```typescript
// Problem
function formatUserName(user) {
  return `${user.first} ${user.last}`;
}
// ... called only once

// Solution
const displayName = `${user.first} ${user.last}`;
```

## Dead Code

**Symptom**: Unreachable code, unused variables, commented-out blocks

**Solution**: Delete it. Git has history.

## Magic Numbers/Strings

**Symptom**: Unexplained literals in logic

**Solution**: Named constants

```typescript
// Problem
if (status === 3) { ... }

// Solution
const STATUS_COMPLETE = 3
if (status === STATUS_COMPLETE) { ... }
```

## Callback Hell

**Symptom**: Nested callbacks 3+ levels deep

**Solution**: async/await or Promise chains

## God Functions

**Symptom**: Function doing multiple unrelated things

**Solution**: Split into focused functions (but don't over-split)

## Redundant Comments

**Symptom**: Comments describing obvious code

```typescript
// Problem
// Increment counter by 1
counter++;

// Solution: Just delete the comment
counter++;
```

## Over-Engineering

**Symptom**: Abstractions, patterns, or flexibility for hypothetical future needs

**Solution**: Delete it. Build for today's requirements.

## Type Assertion Abuse

**Symptom**: `as any` or `as Type` everywhere

**Solution**: Fix the actual types or use proper type guards
