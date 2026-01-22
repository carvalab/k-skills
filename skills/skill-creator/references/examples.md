# Complete Skill Examples

Real-world skill structures following progressive disclosure.

## Example 1: code-simplifier

A focused skill for code refinement.

### Structure
```
skills/code-simplifier/
├── SKILL.md                    (73 lines)
└── references/
    ├── patterns.md             (145 lines)
    ├── anti-patterns.md        (123 lines)
    └── examples.md             (162 lines)
```

### SKILL.md
```markdown
---
name: code-simplifier
description: Simplifies code for clarity and maintainability while preserving functionality. Focuses on recently modified code.
model: opus
---

# Code Simplifier

Enhance code clarity while preserving functionality.

## Workflow

### 1. Discover Project Standards
Check: `.claude/CLAUDE.md`, `.cursor/rules/*`, `AGENTS.md`

### 2. Identify Target Code
Default: Recently modified files (`git diff`)

### 3. Apply Refinements
- Preserve functionality
- Apply project standards
- Enhance clarity
- Maintain balance

### 4. Verify
Run tests, linters, formatters

## References

| Reference | Purpose |
|-----------|---------|
| `references/patterns.md` | Simplification patterns |
| `references/anti-patterns.md` | Code smells to fix |
| `references/examples.md` | Before/after examples |
```

## Example 2: devops (Consolidated)

Multiple tools grouped by capability.

### Structure
```
skills/devops/
├── SKILL.md                    (180 lines)
└── references/
    ├── cloudflare.md           (250 lines)
    ├── docker.md               (280 lines)
    ├── gcloud.md               (260 lines)
    ├── vercel.md               (200 lines)
    └── common-patterns.md      (150 lines)
```

### SKILL.md
```markdown
---
name: devops
description: Infrastructure deployment with Cloudflare, Docker, GCloud, Vercel. Use for serverless functions, containers, and cloud resources.
tools: Bash, Read, Write
model: opus
---

# DevOps

Deploy infrastructure across multiple platforms.

## Quick Start

\`\`\`bash
# Cloudflare Workers
npx wrangler deploy

# Docker
docker compose up -d

# GCloud
gcloud run deploy
\`\`\`

## Workflow

1. Choose deployment target
2. Configure environment
3. Build artifacts
4. Deploy
5. Verify health

## Platform Selection

| Platform | Use Case |
|----------|----------|
| Cloudflare | Edge functions, low latency |
| Docker | Containerized apps |
| GCloud | Full cloud infrastructure |
| Vercel | Next.js, frontend |

## References

| Reference | Purpose |
|-----------|---------|
| `references/cloudflare.md` | Workers, Pages, KV |
| `references/docker.md` | Containers, Compose |
| `references/gcloud.md` | Cloud Run, Functions |
| `references/vercel.md` | Next.js deployment |
```

## Example 3: Minimal Skill

Smallest valid skill structure.

### Structure
```
skills/lint-fix/
└── SKILL.md                    (45 lines)
```

### SKILL.md
```markdown
---
name: lint-fix
description: Fix linting errors automatically. Use after code changes.
tools: Bash
---

# Lint Fix

Automatically fix linting errors.

## Workflow

1. Run linter with fix flag
2. Review changes
3. Commit fixes

## Commands

\`\`\`bash
# ESLint
npx eslint . --fix

# Prettier
npx prettier --write .

# Both
npm run lint:fix
\`\`\`

## Notes

- Always review auto-fixes before committing
- Some fixes may change behavior
- Run tests after fixing
```

No references needed for simple skills.

## Metrics Comparison

| Skill | Entry Point | References | Total | Efficiency |
|-------|-------------|------------|-------|------------|
| code-simplifier | 73 | 430 | 503 | 86% saved |
| devops | 180 | 940 | 1120 | 84% saved |
| lint-fix | 45 | 0 | 45 | N/A |

"Efficiency" = lines NOT loaded on activation
