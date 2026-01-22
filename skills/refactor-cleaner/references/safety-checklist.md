# Safety Checklist

Comprehensive safety procedures for code removal.

## Pre-Removal Checklist

### 1. Detection Verification
- [ ] Run knip
- [ ] Run depcheck
- [ ] Run ts-prune
- [ ] Cross-reference findings

### 2. Reference Search
```bash
# Search for direct imports
grep -r "from './module'" src/

# Search for string references (dynamic imports)
grep -r "module" src/ --include="*.ts"

# Search in tests
grep -r "module" tests/ --include="*.test.ts"

# Search in configs
grep -r "module" *.config.* package.json
```

### 3. Dynamic Import Check
```bash
# Look for dynamic imports
grep -r "import(" src/
grep -r "require(" src/

# Check for string-based requires
grep -r "require\s*\(" src/ | grep -v "from"
```

### 4. Public API Verification
- [ ] Not exported from package entry point
- [ ] Not documented in API docs
- [ ] Not used by external packages
- [ ] Not in public types (.d.ts)

### 5. Git History Review
```bash
# When was it last modified?
git log -1 --format="%ai %s" -- path/to/file

# Who worked on it?
git log --format="%an" -- path/to/file | sort -u

# Why was it created?
git log --diff-filter=A -- path/to/file
```

## Risk Categories

### SAFE to Remove
- Unused private functions
- Unused internal exports
- Unused devDependencies
- Commented-out code
- Empty files
- Test files for deleted code

### CAREFUL - Verify First
- Exports from index files
- Utilities in shared folders
- Code with JSDoc comments
- Recently modified code
- Code with TODO comments

### NEVER Remove Without Review
- Authentication code
- Database clients
- API route handlers
- Payment/transaction logic
- Security utilities
- Error boundaries

## Post-Removal Verification

### Immediate Checks
- [ ] `npm run build` succeeds
- [ ] `npm test` passes
- [ ] `npm run lint` clean
- [ ] No TypeScript errors
- [ ] No console errors in dev

### Extended Checks
- [ ] Run E2E tests if available
- [ ] Manual smoke test key features
- [ ] Check bundle size changed as expected
- [ ] Verify no broken imports

## Rollback Procedure

If anything breaks:

```bash
# 1. Revert the commit
git revert HEAD --no-edit

# 2. Reinstall dependencies
rm -rf node_modules
npm install

# 3. Verify recovery
npm run build
npm test

# 4. Document the failure
echo "FAILED: [item] - [reason]" >> .analysis/failures.log
```

## DO NOT REMOVE List

Maintain a list of items that should never be removed:

```markdown
# .analysis/DO_NOT_REMOVE.md

## Protected Items

### Auth/Security
- src/lib/auth/* - Authentication core
- src/middleware/auth.ts - Session validation

### Database
- src/lib/db.ts - Database client
- src/lib/supabase.ts - Supabase client

### External Integrations
- src/lib/openai.ts - AI integration
- src/lib/stripe.ts - Payments

### Reason: Dynamic Imports
- src/components/icons/* - Loaded dynamically
- src/locales/* - i18n dynamic loading
```
