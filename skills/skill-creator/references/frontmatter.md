# YAML Frontmatter Specification

Complete specification for skill metadata.

## Required Fields

### name
Unique identifier for the skill.

```yaml
name: my-skill-name
```

- Use kebab-case
- Must match directory name
- Keep short (1-3 words)

### description
One-line description of when to use this skill.

```yaml
description: Infrastructure deployment with Cloudflare, Docker, GCloud. Use for serverless functions and containers.
```

- Start with capability, not tool name
- Include trigger phrases ("Use when...")
- Keep under 150 characters
- Action-oriented

## Optional Fields

### tools
List of tools this skill uses.

```yaml
tools: Read, Write, Edit, Bash, Grep, Glob
```

Available tools:
- `Read` - Read files
- `Write` - Create files
- `Edit` - Modify files
- `Bash` - Run commands
- `Grep` - Search content
- `Glob` - Find files

### model
Preferred model for this skill.

```yaml
model: opus
```

Options:
- `opus` - Complex reasoning tasks
- `sonnet` - General tasks
- `haiku` - Simple/fast tasks

### tags
Categories for skill organization.

```yaml
tags: [devops, deployment, infrastructure]
```

### version
Skill version (semantic versioning).

```yaml
version: 1.0.0
```

## Complete Example

```yaml
---
name: devops
description: Infrastructure deployment with Cloudflare, Docker, GCloud. Use PROACTIVELY for deploying serverless functions, containers, and cloud resources.
tools: Read, Write, Edit, Bash
model: opus
tags: [infrastructure, deployment, cloud]
version: 1.2.0
---
```

## Description Writing Guidelines

**Good descriptions:**
```yaml
# Capability-focused
description: Dead code cleanup and consolidation. Use PROACTIVELY for removing unused code and dependencies.

# Clear trigger
description: Documentation and codemap specialist. Use for updating codemaps and READMEs.

# Action-oriented
description: Code simplification for clarity and maintainability. Focuses on recently modified code.
```

**Bad descriptions:**
```yaml
# Tool-focused (wrong)
description: Cloudflare Workers documentation and API reference.

# Vague (wrong)
description: Helps with various development tasks.

# Too long (wrong)
description: This skill provides comprehensive documentation for deploying applications using multiple cloud providers including Cloudflare Workers, Google Cloud Platform, Amazon Web Services, and Docker containers with support for various frameworks and runtimes.
```

## Trigger Phrases

Include phrases that help Claude match tasks to skills:

- "Use when..." - Clear activation trigger
- "Use PROACTIVELY..." - Auto-activate without prompting
- "Use for..." - Task categories

```yaml
# Examples
description: ... Use when deploying to production.
description: ... Use PROACTIVELY after code changes.
description: ... Use for refactoring and cleanup tasks.
```
