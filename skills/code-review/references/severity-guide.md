# Issue Severity Guide

## Classification Rules

**Critical/Major require PROOF, not suspicion.**

Before flagging Critical or Major, answer:
- "I traced the code and confirmed the issue exists because [specific reason]."
- "Did I trace the complete code path, or am I assuming based on a pattern?"
- "Can I explain exactly why this fails, with specific line references?"
- "Did I check for defensive code that might prevent this issue?"

If you cannot answer YES to all, **read more code or downgrade the severity**.

---

## Critical (Priority 0)

**Definition**: Issues that could cause immediate, severe harm.

**Examples**:
- Security vulnerabilities (SQL injection, XSS, auth bypass)
- Data loss or corruption
- Broken builds that block CI/CD
- Crashes in production code paths
- Infinite loops or resource exhaustion
- Exposed secrets or credentials

**Action**: Fix immediately. Block merge until resolved.

---

## Major (Priority 1)

**Definition**: Significant issues that affect functionality or long-term health.

**Examples**:
- Bugs that affect user-facing features
- Performance issues (N+1 queries, memory leaks)
- Architectural violations that increase tech debt
- Missing error handling in critical paths
- Race conditions or concurrency issues
- Breaking API contracts

**Action**: Fix before merge. May require design discussion.

---

## Minor (Priority 2)

**Definition**: Issues that don't affect functionality but reduce code quality.

**Examples**:
- Code style inconsistencies
- Suboptimal but working implementations
- Missing tests for non-critical paths
- Redundant code or unused variables
- Poor naming that doesn't cause confusion
- Missing comments on complex logic

**Action**: Fix if time permits. Can be deferred to follow-up.

---

## Enhancement (Priority 3)

**Definition**: Nice-to-have improvements, not problems.

**Examples**:
- Optimization opportunities (not bugs)
- Alternative approaches that might be cleaner
- Suggestions for future refactoring
- Ideas for additional features
- Documentation improvements

**Action**: Note for future consideration. Don't block merge.

---

## Common Mistakes in Severity Classification

### Over-escalating to Critical/Major

**Wrong**: "This function doesn't handle null" → Critical
**Right**: First check if null is actually possible in this code path.

**Wrong**: "This could cause a memory leak" → Major
**Right**: Verify the leak actually occurs. Check for cleanup code elsewhere.

### Under-escalating Security Issues

**Wrong**: "Missing input validation" → Minor
**Right**: If it's user input to a database/command, it's Critical.

### Pattern-Matching Without Verification

**Wrong**: Flagging every `any` type as Major
**Right**: Check if the `any` actually causes problems or is intentionally flexible.

---

## Severity Decision Tree

```
Is there a security vulnerability?
  → Yes: Critical
  → No: Continue

Does it cause data loss or corruption?
  → Yes: Critical
  → No: Continue

Does it break the build or block CI?
  → Yes: Critical
  → No: Continue

Does it affect user-facing functionality?
  → Yes: Major (if significant) or Minor (if edge case)
  → No: Continue

Does it violate architecture or cause tech debt?
  → Yes: Major
  → No: Continue

Does it reduce code quality without breaking anything?
  → Yes: Minor
  → No: Enhancement
```
