# Code Review Checklist

## Before Starting

- [ ] Identify all changed files: `git diff --name-only HEAD~1`
- [ ] Read project rules in `.cursor/rules/` or `AGENTS.md`
- [ ] Understand the purpose of the changes

## Design & Architecture

- [ ] Changes fit existing system patterns
- [ ] No unnecessary coupling introduced
- [ ] Clear separation of concerns
- [ ] Module boundaries respected
- [ ] Dependencies flow in correct direction

## Complexity & Maintainability

- [ ] Flat control flow (minimal nesting)
- [ ] Low cyclomatic complexity
- [ ] DRY - no duplicated logic
- [ ] No dead code
- [ ] Functions do one thing

## Functionality & Correctness

- [ ] Valid inputs handled correctly
- [ ] Invalid inputs handled gracefully
- [ ] Edge cases covered
- [ ] Error handling present and appropriate
- [ ] No off-by-one errors

## Readability & Naming

- [ ] Variable names reveal intent
- [ ] Function names describe behavior
- [ ] Comments explain WHY, not WHAT
- [ ] No magic numbers/strings
- [ ] Consistent terminology

## Best Practices

- [ ] Language idioms followed
- [ ] Framework patterns used correctly
- [ ] SOLID principles applied
- [ ] Resources properly cleaned up
- [ ] Async/await used correctly

## Test Coverage

- [ ] Happy path tested
- [ ] Error paths tested
- [ ] Edge cases tested
- [ ] Assertions are meaningful
- [ ] Tests are readable

## Standardization & Style

- [ ] Follows project style guide
- [ ] No new linter warnings
- [ ] Consistent formatting
- [ ] Import order consistent

## Security

- [ ] Input validated/sanitized
- [ ] No secrets in code
- [ ] Auth/authz properly implemented
- [ ] SQL injection prevented
- [ ] XSS prevented (if applicable)

## Performance

- [ ] No N+1 queries
- [ ] Efficient algorithms
- [ ] Proper caching where needed
- [ ] Memory management considered
- [ ] No unnecessary iterations

## After Review

- [ ] All issues fixed and committed
- [ ] Quality gates pass (typecheck, build, test)
- [ ] Documentation updated if needed
