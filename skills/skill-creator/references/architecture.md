# 3-Tier Architecture Deep Dive

Understanding the progressive disclosure loading system.

## Why Progressive Disclosure?

**The Problem:**
- Loading 5-7 skills = 5,000-7,000 lines in context
- 90% of information is irrelevant to current task
- Context overflow causes slow/unreliable responses

**The Solution:**
- Load only what's needed, when it's needed
- Small, focused chunks instead of monolithic dumps
- Let Claude decide what to load next

## Tier 1: Metadata

**What it is:** YAML frontmatter only
**When loaded:** Always (skill discovery)
**Size:** ~100 words

```yaml
---
name: devops
description: Infrastructure deployment with Cloudflare, Docker, GCloud. Use for deploying serverless functions, containers, and cloud resources.
tools: Bash, Read, Write
model: opus
---
```

**Purpose:**
- Claude scans to decide if skill is relevant
- No workflow details, just capability description
- Must be scannable in <1 second

## Tier 2: Entry Point (SKILL.md)

**What it is:** Overview + navigation map
**When loaded:** Skill activation
**Size:** ~200 lines MAX

**Contains:**
- Brief description (2-3 sentences)
- Quick start commands
- High-level workflow steps
- Reference navigation table
- Success metrics

**Does NOT contain:**
- Detailed implementation guides
- Long code examples
- Edge cases and troubleshooting
- Full API references

**Example structure:**
```markdown
# Skill Name

Brief description.

## Quick Start
[Essential commands]

## Workflow
[High-level steps with links to references]

## References
[Navigation table]
```

## Tier 3: References

**What it is:** Detailed documentation
**When loaded:** On-demand (Claude reads when needed)
**Size:** 200-300 lines each

**Characteristics:**
- Single-topic focused
- Self-contained (can be read independently)
- Named descriptively (not `guide1.md`)
- Linked from SKILL.md navigation table

**Good reference topics:**
- `setup-guide.md` - Initial setup
- `api-reference.md` - API details
- `examples.md` - Code examples
- `troubleshooting.md` - Common issues
- `patterns.md` - Design patterns

## Loading Flow

```
User request
    │
    ▼
Scan Tier 1 (all skills)
    │ "Is this skill relevant?"
    ▼
Load Tier 2 (relevant skills only)
    │ "What workflow applies?"
    ▼
Load Tier 3 (specific reference)
    │ "Implementation details for current step"
    ▼
Execute task
```

## Token Efficiency

**Before progressive disclosure:**
```
Skill activation → 1,131 lines loaded
Relevant content → ~100 lines (9%)
Wasted context  → ~1,000 lines (91%)
```

**After progressive disclosure:**
```
SKILL.md        → 200 lines
Reference (1-2) → 400 lines
Total           → 600 lines
Relevant        → 540 lines (90%)
```

**Result:** 4.8x better token efficiency

## Testing Your Architecture

1. **Cold start test:**
   - Clear context
   - Activate skill
   - Measure lines loaded
   - Should be <500 lines

2. **Relevance test:**
   - Read loaded content
   - How much applies to typical task?
   - Should be >80%

3. **Navigation test:**
   - Can you find specific info quickly?
   - Is reference table clear?
   - Are references well-named?
