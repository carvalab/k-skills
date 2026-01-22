# Deletion Log Format

How to document code removals in `docs/DELETION_LOG.md`.

## Template

```markdown
# Code Deletion Log

Track all code removal for audit and potential recovery.

---

## [YYYY-MM-DD] Session Title

**Author:** Name
**Branch:** feature/cleanup-xyz
**Commit:** abc1234

### Dependencies Removed

| Package | Version | Reason | Size |
|---------|---------|--------|------|
| lodash | ^4.17.21 | Replaced by native methods | 72KB |
| moment | ^2.29.4 | Replaced by date-fns | 290KB |

### Files Deleted

| File | Reason | Replacement |
|------|--------|-------------|
| src/old-component.tsx | Unused | N/A |
| lib/deprecated-util.ts | Consolidated | lib/utils.ts |

### Exports Removed

| File | Export | Reason |
|------|--------|--------|
| src/utils/helpers.ts | formatDate() | No references |
| src/utils/helpers.ts | parseQuery() | No references |

### Duplicate Code Consolidated

| Original Files | Merged Into | Reason |
|----------------|-------------|--------|
| Button1.tsx, Button2.tsx | Button.tsx | Identical logic |

### Impact Summary

- **Files deleted:** 15
- **Dependencies removed:** 5
- **Exports removed:** 23
- **Lines of code:** -2,300
- **Bundle size:** -45KB

### Verification

- [x] Build passes
- [x] All tests pass
- [x] Manual testing complete
- [x] No console errors

### Notes

Any special considerations or things to watch for.

---
```

## Example Entry

```markdown
## [2024-01-15] Q1 Dead Code Cleanup

**Author:** Developer Name
**Branch:** refactor/dead-code-cleanup
**Commit:** a1b2c3d

### Dependencies Removed

| Package | Version | Reason | Size |
|---------|---------|--------|------|
| axios | ^1.6.0 | Replaced by fetch | 15KB |
| classnames | ^2.3.0 | Using template literals | 1KB |

### Files Deleted

| File | Reason | Replacement |
|------|--------|-------------|
| src/components/LegacyHeader.tsx | Replaced 6 months ago | Header.tsx |
| src/utils/oldHelpers.ts | No imports found | N/A |
| src/hooks/useDeprecated.ts | Marked deprecated, unused | N/A |

### Exports Removed

| File | Export | Reason |
|------|--------|--------|
| src/lib/api.ts | fetchLegacy() | No references |
| src/lib/api.ts | oldEndpoint | Constant unused |

### Impact Summary

- **Files deleted:** 8
- **Dependencies removed:** 2
- **Exports removed:** 12
- **Lines of code:** -847
- **Bundle size:** -18KB

### Verification

- [x] Build passes
- [x] All tests pass (142/142)
- [x] Manual testing complete
- [x] No console errors

### Notes

- `oldHelpers.ts` had TODO comment from 2023, safe to remove
- Header component was already migrated in PR #234
```

## Best Practices

1. **One entry per session** - Group related removals
2. **Include commit hash** - Easy to find/revert
3. **Note replacements** - Where did functionality go?
4. **Track size impact** - Validates the effort
5. **Date everything** - For historical context
