# Detection Tools

Tools for finding dead code, unused exports, and dependencies.

## knip

Find unused files, exports, dependencies, and types.

```bash
# Full analysis
npx knip

# JSON output for processing
npx knip --reporter json

# Check specific workspace
npx knip --workspace packages/core

# Ignore certain patterns
npx knip --ignore "**/*.test.ts"
```

Configuration (`knip.json`):
```json
{
  "$schema": "https://unpkg.com/knip@latest/schema.json",
  "entry": ["src/index.ts"],
  "project": ["src/**/*.ts"],
  "ignore": ["**/*.d.ts", "**/*.test.ts"],
  "ignoreDependencies": ["@types/*"]
}
```

## depcheck

Identify unused npm dependencies.

```bash
# Basic check
npx depcheck

# JSON output
npx depcheck --json

# Ignore specific packages
npx depcheck --ignores="eslint,prettier"

# Skip dev dependencies
npx depcheck --skip-missing
```

## ts-prune

Find unused TypeScript exports.

```bash
# Find all unused exports
npx ts-prune

# Ignore index files
npx ts-prune --ignore "index.ts"

# Check specific directory
npx ts-prune --project tsconfig.json src/
```

## ESLint

Check for unused variables and disable directives.

```bash
# Unused disable directives
npx eslint . --report-unused-disable-directives

# Unused variables (requires rule enabled)
npx eslint . --rule 'no-unused-vars: error'
```

## madge

Find circular dependencies.

```bash
# Check for circular dependencies
npx madge --circular src/

# Generate dependency graph
npx madge --image graph.svg src/

# JSON output
npx madge --json src/
```

## Combined Analysis Script

```bash
#!/bin/bash
# scripts/analyze-dead-code.sh

echo "=== Running knip ==="
npx knip --reporter json > .analysis/knip.json

echo "=== Running depcheck ==="
npx depcheck --json > .analysis/depcheck.json

echo "=== Running ts-prune ==="
npx ts-prune > .analysis/ts-prune.txt

echo "=== Checking circular deps ==="
npx madge --circular src/

echo "Analysis complete. Check .analysis/ folder."
```

## Interpreting Results

### knip Output
- `unused files` - Safe to delete after verification
- `unused exports` - Can be removed or made internal
- `unused dependencies` - Safe to uninstall
- `unlisted dependencies` - Need to be added to package.json

### depcheck Output
- `dependencies` - Unused production deps (safe to remove)
- `devDependencies` - Unused dev deps (safe to remove)
- `missing` - Used but not in package.json (need to add)

### ts-prune Output
- Lists exports with no references
- Check each with grep before removing
- Some may be used dynamically
