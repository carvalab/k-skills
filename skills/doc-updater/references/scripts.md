# Documentation Generation Scripts

Scripts for automated codemap and documentation generation.

## Dependency Graph

```bash
# Install madge
npm install -g madge

# Generate visual dependency graph
npx madge --image docs/CODEMAPS/graph.svg src/

# Generate JSON dependency data
npx madge --json src/ > docs/CODEMAPS/deps.json

# Check for circular dependencies
npx madge --circular src/
```

## JSDoc Extraction

```bash
# Install jsdoc-to-markdown
npm install -g jsdoc-to-markdown

# Generate API docs
npx jsdoc2md src/**/*.ts > docs/api.md

# Generate for specific files
npx jsdoc2md src/lib/*.ts > docs/lib-api.md
```

## ts-morph Analysis

```typescript
/**
 * Generate codemaps from TypeScript AST
 * Usage: npx tsx scripts/codemaps/generate.ts
 */

import { Project } from 'ts-morph';

async function generateCodemaps() {
  const project = new Project({
    tsConfigFilePath: 'tsconfig.json',
  });

  const sourceFiles = project.getSourceFiles('src/**/*.{ts,tsx}');

  for (const file of sourceFiles) {
    const exports = file.getExportedDeclarations();
    const imports = file.getImportDeclarations();

    // Build module info
    console.log(`File: ${file.getFilePath()}`);
    console.log(`Exports: ${exports.size}`);
    console.log(`Imports: ${imports.length}`);
  }
}

generateCodemaps();
```

## Documentation Update Script

```typescript
/**
 * Update documentation from code
 * Usage: npx tsx scripts/docs/update.ts
 */

import * as fs from 'fs';
import * as path from 'path';

async function updateDocs() {
  // 1. Read package.json for project info
  const pkg = JSON.parse(fs.readFileSync('package.json', 'utf-8'));

  // 2. Scan src/ for structure
  const structure = scanDirectory('src');

  // 3. Update README sections
  await updateReadme(pkg, structure);

  // 4. Regenerate codemaps
  await regenerateCodemaps(structure);
}

function scanDirectory(dir: string): string[] {
  // Recursively scan directory
  return fs.readdirSync(dir, { recursive: true }) as string[];
}

updateDocs();
```

## npm Scripts

Add to `package.json`:

```json
{
  "scripts": {
    "docs:codemaps": "tsx scripts/codemaps/generate.ts",
    "docs:api": "jsdoc2md src/**/*.ts > docs/api.md",
    "docs:deps": "madge --image docs/graph.svg src/",
    "docs:all": "npm run docs:codemaps && npm run docs:api && npm run docs:deps"
  }
}
```
