# Content Writing Guide

Best practices for writing skill content.

## Tone and Voice

### Use Imperative Form
```markdown
# Good
Run the build command.
Create a new file.
Check the output.

# Bad
You should run the build command.
The user needs to create a new file.
It's recommended to check the output.
```

### Be Direct
```markdown
# Good
Deploy with: `npm run deploy`

# Bad
In order to deploy the application, you will need to execute the deployment script by running the following command in your terminal.
```

### Action-Oriented
```markdown
# Good
## Workflow
1. Analyze the codebase
2. Identify issues
3. Apply fixes

# Bad
## Overview
This section describes the workflow that should be followed when using this skill. The workflow consists of several steps...
```

## Structure Principles

### Progressive Detail
```
Entry Point (SKILL.md)
├── What it does (1-2 sentences)
├── Quick start (essential commands)
├── High-level workflow (steps, not details)
└── Reference links

Reference Files
├── Detailed how-to
├── Code examples
├── Edge cases
└── Troubleshooting
```

### Single Responsibility
Each reference file = one topic

```
# Good structure
references/
├── setup.md          # Just setup
├── deployment.md     # Just deployment
├── monitoring.md     # Just monitoring

# Bad structure
references/
├── guide.md          # Everything mixed
├── more-stuff.md     # Random overflow
```

## Formatting

### Use Tables for Navigation
```markdown
## References

| Reference | Purpose |
|-----------|---------|
| `references/setup.md` | Initial project setup |
| `references/api.md` | API endpoint details |
```

### Use Code Blocks Sparingly
In SKILL.md: Only essential commands
In references: Full examples

```markdown
# SKILL.md - minimal
\`\`\`bash
npm run deploy
\`\`\`

# references/deployment.md - detailed
\`\`\`bash
# Build first
npm run build

# Deploy to staging
npm run deploy:staging

# Verify deployment
curl https://staging.example.com/health

# Deploy to production
npm run deploy:prod
\`\`\`
```

### Keep Lists Short
```markdown
# Good (scannable)
- Run analysis
- Review findings
- Apply fixes

# Bad (walls of text)
- First, you need to run the analysis tool which will scan through all the files in the project
- After the analysis completes, you should carefully review all the findings
- Once you've reviewed everything, you can begin applying the necessary fixes
```

## What Goes Where

### In SKILL.md (Entry Point)
- Brief capability description
- Quick start (1-3 commands)
- High-level workflow (numbered steps)
- Reference navigation table
- Success criteria

### In References
- Detailed step-by-step guides
- Complete code examples
- Configuration options
- Troubleshooting guides
- Edge cases

### Never in SKILL.md
- Long code blocks (>10 lines)
- Detailed API documentation
- Troubleshooting guides
- Edge cases
- Full examples

## Line Counting

Check your work:
```bash
wc -l SKILL.md
# Should be <200

wc -l references/*.md
# Each should be 200-300
```
