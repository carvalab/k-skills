# Constructive Feedback Guide

How to give effective code review feedback that helps, not hurts.

## Core Principles

### 1. Feedback is a Gift, Not a Critique

**Focus on:**
- The code, not the person
- Learning opportunities
- Improving the codebase together

**Avoid:**
- "You should have..."
- "Why didn't you..."
- "This is wrong"

### 2. Ask Before Assuming

```markdown
# Bad - Assumes incompetence
"This doesn't handle null values"

# Good - Asks for clarification
"What happens if `user` is null here? I might be missing context."
```

### 3. Explain the WHY

```markdown
# Bad - Just says what to do
"Use early returns here"

# Good - Explains why
"Consider early returns here - it would reduce nesting from 4 levels to 1, making the happy path clearer."
```

## Feedback Templates

### For Bugs

```markdown
**Potential issue:** [description]
**Scenario:** [when this could happen]
**Suggestion:** [how to fix]

Example:
**Potential issue:** Race condition in counter update
**Scenario:** Two concurrent requests could read the same value
**Suggestion:** Consider using atomic increment or a transaction
```

### For Suggestions

```markdown
**Idea:** [suggestion]
**Benefit:** [why it helps]
**Trade-off:** [any downsides]

Example:
**Idea:** Extract this into a custom hook
**Benefit:** Could be reused in UserProfile and Settings
**Trade-off:** Slight indirection, may not be worth it if only used twice
```

### For Questions

```markdown
**Question:** [what you want to understand]
**Context:** [why you're asking]

Example:
**Question:** Why use `any` here instead of a specific type?
**Context:** Trying to understand if there's a constraint I'm missing
```

## Tone Examples

### Severity: Critical

```markdown
# Bad
"CRITICAL BUG: You're exposing user passwords!"

# Good
"🔴 Security concern: The password field is included in the API response.
This should be excluded before sending to the client.
Happy to help fix this - want me to suggest a change?"
```

### Severity: Major

```markdown
# Bad
"This will cause N+1 queries and kill performance"

# Good
"Performance note: This queries the database inside the loop,
which could be slow with many users. Consider batching:
`const posts = await getPosts(userIds)`

Let me know if you'd like to discuss alternatives."
```

### Severity: Minor

```markdown
# Bad
"Wrong naming convention"

# Good
"Nit: Our convention is camelCase for functions (`getUserData` vs `get_user_data`).
Not blocking, but worth updating for consistency."
```

### Enhancement

```markdown
# Bad
"You should refactor this"

# Good
"Future idea: This could be simplified with the new `useQuery` hook
we added last sprint. Not urgent - just flagging for when you're
in this area again."
```

## Positive Feedback

Don't just point out problems. Highlight good work:

```markdown
✨ Nice use of early returns - much cleaner than nested ifs
✨ Good error handling - I like that you included the original error
✨ Clear naming - `isValidEmailFormat` is self-documenting
✨ Thoughtful test coverage - the edge cases are well covered
```

## When to Talk Instead

**Use synchronous discussion when:**
- Multiple back-and-forth comments
- Architectural disagreement
- Explaining complex context
- Sensitive feedback

```markdown
"This is getting complex in comments - want to hop on a quick call
to discuss the approach? I have some ideas but want to understand
your constraints first."
```

## Avoid These Anti-Patterns

| Anti-Pattern | Better Approach |
|--------------|-----------------|
| "LGTM" with no context | "LGTM - the error handling is solid" |
| Nitpicking style in large PRs | Focus on logic, let linters handle style |
| Blocking on preferences | "I'd do it differently, but this works" |
| Delayed reviews | Review within 24 hours |
| Review bombing | Limit to 5-7 actionable comments |

## The Golden Rule

Before posting a comment, ask:
> "If I received this feedback, would I feel helped or attacked?"

If attacked → rewrite with empathy.
