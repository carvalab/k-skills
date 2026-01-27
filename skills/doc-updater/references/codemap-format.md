# Codemap Format Specification

Complete specification for generating architectural codemaps.

## File Structure

```
docs/CODEMAPS/
├── INDEX.md              # Overview of all areas
├── frontend.md           # Frontend structure
├── backend.md            # Backend/API structure
├── database.md           # Database schema
├── integrations.md       # External services
└── workers.md            # Background jobs
```

## Template

```markdown
# [Area] Codemap

**Last Updated:** YYYY-MM-DD
**Entry Points:** list of main files

## Architecture

[ASCII diagram of component relationships]

## Key Modules

| Module      | Purpose           | Exports      | Dependencies |
| ----------- | ----------------- | ------------ | ------------ |
| module-name | Brief description | main exports | key deps     |

## Data Flow

[Description of how data flows through this area]

## External Dependencies

- package-name - Purpose, Version
- ...

## Related Areas

Links to other codemaps that interact with this area
```

## Module Analysis

For each module, extract:

- Exports (public API)
- Imports (dependencies)
- Routes (API routes, pages)
- Database models (Supabase, Prisma)
- Queue/worker modules

## ASCII Diagram Guidelines

Keep diagrams simple and scannable:

```
┌─────────────┐     ┌─────────────┐
│   Client    │────▶│  API Route  │
└─────────────┘     └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Database   │
                    └─────────────┘
```

## Key Modules Table

Focus on:

- Entry points and main files
- Shared utilities
- External service clients
- Core business logic

Avoid listing every file - focus on architectural significance.

## Linking Strategy

Cross-reference related codemaps:

- Frontend → Backend (API calls)
- Backend → Database (queries)
- Backend → Integrations (external services)
- Workers → Database (background jobs)
