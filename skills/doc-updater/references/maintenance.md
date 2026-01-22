# Documentation Maintenance

Schedule and quality assurance for documentation.

## Maintenance Schedule

### Weekly
- Check for new files in src/ not in codemaps
- Verify README.md instructions work
- Update package.json descriptions

### After Major Features
- Regenerate all codemaps
- Update architecture documentation
- Refresh API reference
- Update setup guides

### Before Releases
- Comprehensive documentation audit
- Verify all examples work
- Check all external links
- Update version references

## Quality Checklist

Before committing documentation:

- [ ] Codemaps generated from actual code
- [ ] All file paths verified to exist
- [ ] Code examples compile/run
- [ ] Links tested (internal and external)
- [ ] Freshness timestamps updated
- [ ] ASCII diagrams are clear
- [ ] No obsolete references
- [ ] Spelling/grammar checked

## Pull Request Template

```markdown
## Docs: Update Codemaps and Documentation

### Summary
Regenerated codemaps and updated documentation to reflect current codebase state.

### Changes
- Updated docs/CODEMAPS/* from current code structure
- Refreshed README.md with latest setup instructions
- Updated docs/GUIDES/* with current API endpoints

### Generated Files
- docs/CODEMAPS/INDEX.md
- docs/CODEMAPS/frontend.md
- docs/CODEMAPS/backend.md

### Verification
- [x] All links in docs work
- [x] Code examples are current
- [x] Architecture diagrams match reality

### Impact
LOW - Documentation only, no code changes
```

## Validation Commands

```bash
# Check for broken internal links
grep -r "\[.*\](.*\.md)" docs/ | while read line; do
  # Extract and verify each link
done

# Verify code examples compile
npx tsc --noEmit docs/examples/*.ts

# Check file references exist
grep -oP '`[^`]+\.(ts|tsx|js|jsx)`' docs/**/*.md | while read file; do
  test -f "$file" || echo "Missing: $file"
done
```

## Common Issues

### Stale File References
- Run `git diff --name-status HEAD~10` to find renamed/deleted files
- Update codemaps with new paths

### Outdated Dependencies
- Compare `package.json` versions with docs
- Update External Dependencies sections

### Broken Examples
- Test all code snippets periodically
- Use actual project imports, not hypotheticals
