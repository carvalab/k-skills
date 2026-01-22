# Anti-Patterns

What NOT to do when creating skills.

## Anti-Pattern 1: Documentation Dumps

**Wrong:** Treating skills like documentation repositories

```markdown
# cloudflare skill (BAD)

Here's everything about Cloudflare Workers...
[800 lines of API docs]
[200 lines of configuration]
[300 lines of examples]
```

**Right:** Skills are workflow capabilities

```markdown
# devops skill (GOOD)

Deploy infrastructure with Cloudflare, Docker, GCloud.

## Workflow
1. Choose deployment target
2. Configure environment
3. Deploy

## References
- `references/cloudflare.md` - Cloudflare specifics
- `references/docker.md` - Docker specifics
```

## Anti-Pattern 2: One Tool = One Skill

**Wrong:** Creating separate skills for each tool

```
skills/
├── cloudflare/
├── cloudflare-workers/
├── cloudflare-pages/
├── docker/
├── docker-compose/
└── gcloud/
```

**Right:** Group by workflow capability

```
skills/
├── devops/           # All deployment tools
├── ui-styling/       # All styling tools
└── web-frameworks/   # All framework tools
```

## Anti-Pattern 3: Monolithic SKILL.md

**Wrong:** Everything in one giant file

```markdown
# SKILL.md (1,200 lines)

## Overview
## Quick Start
## Detailed Setup (200 lines)
## Configuration (150 lines)
## API Reference (400 lines)
## Examples (300 lines)
## Troubleshooting (150 lines)
```

**Right:** Progressive disclosure

```markdown
# SKILL.md (150 lines)

## Overview
## Quick Start
## Workflow (high-level)
## References (table linking to details)
```

## Anti-Pattern 4: Vague Descriptions

**Wrong:**
```yaml
description: Helps with development tasks.
description: Cloudflare documentation.
description: Useful skill for coding.
```

**Right:**
```yaml
description: Deploy serverless functions with Cloudflare Workers. Use for edge computing and API deployment.
description: Dead code cleanup. Use PROACTIVELY for removing unused code and dependencies.
```

## Anti-Pattern 5: Second Person Voice

**Wrong:**
```markdown
You should run the build command.
You will need to configure the settings.
You can find the documentation at...
```

**Right:**
```markdown
Run the build command.
Configure the settings.
Documentation at...
```

## Anti-Pattern 6: Nested Ternaries in Examples

**Wrong:**
```typescript
const status = code === 200 ? 'ok' : code === 404 ? 'not found' : 'error'
```

**Right:**
```typescript
function getStatus(code) {
  switch (code) {
    case 200: return 'ok'
    case 404: return 'not found'
    default: return 'error'
  }
}
```

## Anti-Pattern 7: No Reference Table

**Wrong:** References exist but aren't discoverable

```markdown
For more details, check the references folder.
```

**Right:** Clear navigation table

```markdown
## References

| Reference | Purpose |
|-----------|---------|
| `references/setup.md` | Initial setup |
| `references/api.md` | API reference |
```

## Anti-Pattern 8: Oversized References

**Wrong:** References that are 500+ lines

```
references/
└── everything.md     # 800 lines
```

**Right:** Split into focused files

```
references/
├── setup.md          # 150 lines
├── configuration.md  # 200 lines
├── api.md            # 250 lines
└── examples.md       # 200 lines
```

## Anti-Pattern 9: Missing Metrics

**Wrong:** No way to validate skill quality

**Right:** Clear success criteria

```markdown
## Metrics

After creating skill:
- Entry point: <200 lines ✓
- Each reference: 200-300 lines ✓
- Cold start load: <500 lines ✓
```

## Self-Check Questionnaire

Before publishing a skill:

1. Is SKILL.md under 200 lines?
2. Are references 200-300 lines each?
3. Is the description action-oriented?
4. Does it describe capability, not tool?
5. Is there a reference navigation table?
6. Can Claude find what it needs quickly?
7. Would cold start load <500 lines?
