# Self-Review Workflow

Automated code review during development - review your own changes before proceeding.

## Core Principle

**Review early, review often.** Catch issues before they cascade.

## When to Trigger Self-Review

### Mandatory

| Trigger                            | Why                                      |
| ---------------------------------- | ---------------------------------------- |
| After each task in multi-step work | Catch issues before they compound        |
| After completing major feature     | Verify implementation meets requirements |
| Before merging to main             | Final quality gate                       |

### Optional but Valuable

| Trigger                  | Why                                     |
| ------------------------ | --------------------------------------- |
| When stuck               | Fresh perspective on own code           |
| Before refactoring       | Establish baseline                      |
| After fixing complex bug | Verify fix doesn't introduce new issues |

## How to Self-Review

### 1. Get Review Range

```bash
# Review against previous commit
BASE_SHA=$(git rev-parse HEAD~1)
HEAD_SHA=$(git rev-parse HEAD)

# Review against main branch
BASE_SHA=$(git merge-base main HEAD)
HEAD_SHA=$(git rev-parse HEAD)

# Review specific range
BASE_SHA=abc1234
HEAD_SHA=def5678
```

### 2. Scope the Changes

```bash
# Files changed
git diff --name-only $BASE_SHA..$HEAD_SHA --diff-filter=ACMR

# Stats
git diff --stat $BASE_SHA..$HEAD_SHA

# Full diff
git diff $BASE_SHA..$HEAD_SHA
```

### 3. Compare Against Requirements

Check implementation against:

- Original task/issue description
- Plan document (if exists)
- Acceptance criteria
- Expected behavior

Ask: "Does this implementation satisfy what was requested?"

### 4. Run Review Checklist

For each changed file:

- Design & Architecture fit?
- Error handling complete?
- Edge cases covered?
- Tests meaningful?
- Security considerations?

### 5. Fix Issues Immediately

Don't just note issues - fix them:

1. Fix the code
2. Verify fix works
3. Commit: `git commit -m "fix: description"`
4. Continue review

### 6. Provide Verdict

```markdown
### Assessment

**Ready to proceed?** [Yes / Yes with notes / No]

**Reasoning:** [Technical justification]
```

## Integration with Workflows

### Multi-Task Development

```
Task 1 → Self-Review → Fix → Commit
    ↓
Task 2 → Self-Review → Fix → Commit
    ↓
Task 3 → Self-Review → Fix → Commit
    ↓
Final Review → Merge
```

### Plan Execution

```
Batch 1 (Tasks 1-3) → Self-Review → Fix
    ↓
Batch 2 (Tasks 4-6) → Self-Review → Fix
    ↓
Final Review → Merge
```

## Output Template

```markdown
## Self-Review: [Feature/Task Name]

**Range:** {BASE_SHA}..{HEAD_SHA}
**Files:** [count] changed

### Strengths

- [Specific positive finding with file:line]
- [Another strength]

### Issues Found

#### Critical

[None / List with file:line, issue, fix applied]

#### Major

[None / List with file:line, issue, fix applied]

#### Minor

[None / List with file:line, issue, fix applied]

### Quality Gates

- [ ] Lint passes
- [ ] Types check
- [ ] Build succeeds
- [ ] Tests pass

### Assessment

**Ready to proceed?** Yes / No

**Reasoning:** [1-2 sentences]
```

## Red Flags

**Never:**

- Skip review because "it's simple"
- Ignore Critical issues
- Proceed with unfixed Major issues
- Mark nitpicks as Critical

**If unsure:**

- Re-read the requirements
- Trace execution path completely
- Run the code manually
- Add more tests

## Example

```markdown
## Self-Review: Add user verification

**Range:** a7981ec..3df7661
**Files:** 4 changed

### Strengths

- Clean separation between verify and repair (verifier.ts:15-42)
- Comprehensive test coverage (18 tests)
- Good error messages with context (verifier.ts:85-92)

### Issues Found

#### Critical

None

#### Major

1. **Missing progress indicator** - verifier.ts:130
   - Long operations show no progress
   - Fixed: Added progress callback

#### Minor

1. Magic number 100 for batch size - verifier.ts:45
   - Noted for future: extract to config

### Quality Gates

- [x] Lint passes
- [x] Types check
- [x] Build succeeds
- [x] Tests pass (18/18)

### Assessment

**Ready to proceed?** Yes

**Reasoning:** Core implementation solid. Major issue (progress) fixed. Minor issue noted but not blocking.
```
