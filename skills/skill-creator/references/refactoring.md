# Refactoring Bloated Skills

How to split oversized skills into progressive disclosure architecture.

## Signs You Need to Refactor

- SKILL.md over 200 lines
- Loading multiple skills causes context issues
- Activation time noticeably slow
- 90% of loaded content isn't used for typical tasks
- Skill contains "documentation dumps"

## Before/After Example

### Before (870 lines in one file)
```
skills/claude-code/
└── SKILL.md    # 870 lines - everything in one file
```

### After (181 lines + 13 references)
```
skills/claude-code/
├── SKILL.md                    # 181 lines
└── references/
    ├── configuration.md        # 150 lines
    ├── commands.md             # 200 lines
    ├── mcp-servers.md          # 180 lines
    ├── hooks.md                # 160 lines
    ├── memory-management.md    # 140 lines
    └── ... (8 more files)
```

**Result:** 79% reduction in initial load, 4.8x better token efficiency

## Refactoring Process

### Step 1: Identify Sections

Read through the bloated SKILL.md and mark logical sections:

```markdown
<!-- Current SKILL.md -->

# My Skill

## Overview          ← KEEP in SKILL.md
[50 lines]

## Quick Start       ← KEEP in SKILL.md
[30 lines]

## Detailed Setup    ← MOVE to references/setup.md
[200 lines]

## API Reference     ← MOVE to references/api.md
[300 lines]

## Examples          ← MOVE to references/examples.md
[250 lines]

## Troubleshooting   ← MOVE to references/troubleshooting.md
[100 lines]
```

### Step 2: Create Reference Files

For each section to extract:

1. Create file in `references/`
2. Add clear title and brief intro
3. Move content verbatim
4. Ensure it's self-contained

```markdown
# references/setup.md

# Detailed Setup Guide

Complete setup instructions for [skill].

[moved content here]
```

### Step 3: Update SKILL.md

Replace moved sections with brief summaries + links:

```markdown
## Setup

Basic setup:
\`\`\`bash
npm install
\`\`\`

See `references/setup.md` for detailed configuration.
```

### Step 4: Add Navigation Table

```markdown
## References

| Reference | Purpose |
|-----------|---------|
| `references/setup.md` | Detailed setup guide |
| `references/api.md` | API reference |
| `references/examples.md` | Code examples |
```

### Step 5: Verify Metrics

```bash
# Check line counts
wc -l SKILL.md
# Should be <200

wc -l references/*.md
# Each should be 200-300

# Total check
find . -name "*.md" -exec cat {} \; | wc -l
# Total content preserved
```

## Common Extraction Patterns

### Documentation → Reference
```
Before: ## API Reference [300 lines]
After:  See `references/api.md`
```

### Examples → Reference
```
Before: ## Examples [250 lines of code]
After:  See `references/examples.md`
```

### Setup Guide → Reference
```
Before: ## Detailed Setup [200 lines]
After:  Quick start + `references/setup.md`
```

### Troubleshooting → Reference
```
Before: ## Troubleshooting [150 lines]
After:  See `references/troubleshooting.md`
```

## What to Keep in SKILL.md

- Capability description (2-3 sentences)
- Quick start (essential commands only)
- High-level workflow (steps, not details)
- Reference navigation table
- Success metrics/checklist

## Consolidation Opportunity

While refactoring, consider consolidating related skills:

```
Before: 36 tool-specific skills
After:  20 workflow-capability groups

Examples:
- cloudflare + docker + gcloud → devops
- nextjs + turborepo → web-frameworks
- shadcn + tailwind → ui-styling
```

Group by **workflow capability**, not by tool.
