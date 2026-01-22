# PR Template for Cleanup

Template for pull requests containing code removal.

## Template

```markdown
## Refactor: Dead Code Cleanup

### Summary

Dead code cleanup removing unused exports, dependencies, and consolidating duplicates.

### Detection Method

- [x] knip analysis
- [x] depcheck analysis
- [x] ts-prune analysis
- [x] Manual grep verification

### Changes

#### Dependencies Removed
- `package-a` - Reason for removal
- `package-b` - Replaced by X

#### Files Deleted
- `src/unused-file.ts` - No references found
- `src/old-component.tsx` - Replaced by NewComponent

#### Exports Removed
- `helperFunction()` from utils.ts - Unused
- `CONSTANT` from config.ts - Unused

#### Duplicates Consolidated
- `Component1.tsx` + `Component2.tsx` → `Component.tsx`

### Impact

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Bundle size | 250KB | 205KB | -45KB |
| Dependencies | 45 | 40 | -5 |
| Files | 120 | 105 | -15 |
| Lines of code | 8000 | 5700 | -2300 |

### Testing

- [x] `npm run build` passes
- [x] `npm test` passes (all X tests)
- [x] `npm run lint` clean
- [x] Manual smoke test completed
- [x] No console errors

### Risk Assessment

🟢 **LOW** - Only removed verifiably unused code

All items were:
- Confirmed unused by detection tools
- Verified with grep search
- Not part of public API
- Not dynamically imported

### Documentation

- [x] DELETION_LOG.md updated
- [x] Commit messages descriptive

### Rollback Plan

If issues found in production:
1. `git revert <commit-hash>`
2. `npm install`
3. Deploy previous version

### Checklist

- [x] Detection tools run
- [x] All references verified
- [x] Tests passing
- [x] Build succeeds
- [x] DELETION_LOG.md updated
- [x] No breaking changes to public API

---

See `docs/DELETION_LOG.md` for complete removal details.
```

## Commit Message Format

```
refactor: remove unused [category]

- Remove X unused dependencies
- Delete Y unused files
- Remove Z unused exports

Detection: knip, depcheck, ts-prune
Verification: grep search, test suite

Bundle size: -XXkb
Lines removed: -XXXX

See DELETION_LOG.md for details
```

## Review Guidelines

When reviewing cleanup PRs:

1. **Verify detection method** - Which tools were used?
2. **Check high-risk items** - Any shared utilities removed?
3. **Validate testing** - All tests still pass?
4. **Review DELETION_LOG** - Is it comprehensive?
5. **Check rollback plan** - Can we easily revert?

## Labels

Suggested PR labels:
- `refactor` - Code cleanup
- `chore` - Maintenance
- `low-risk` - Verified unused code only
- `needs-review` - Requires careful review
