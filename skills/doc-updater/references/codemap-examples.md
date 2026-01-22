# Codemap Examples

Real-world examples for common architectures.

## Frontend Codemap

```markdown
# Frontend Architecture

**Last Updated:** YYYY-MM-DD
**Framework:** Next.js 15 (App Router)
**Entry Point:** src/app/layout.tsx

## Structure

src/
├── app/                # Next.js App Router
│   ├── api/           # API routes
│   ├── dashboard/     # Dashboard pages
│   └── settings/      # Settings pages
├── components/        # React components
├── hooks/             # Custom hooks
└── lib/               # Utilities

## Key Components

| Component | Purpose | Location |
|-----------|---------|----------|
| Header | Navigation | components/Header.tsx |
| Dashboard | Main view | app/dashboard/page.tsx |
| SearchBar | Search UI | components/SearchBar.tsx |

## Data Flow

User → Page Component → API Route → Database → Response

## External Dependencies

- Next.js 15 - Framework
- React 19 - UI library
- Tailwind CSS - Styling
```

## Backend Codemap (Node.js)

```markdown
# Backend Architecture

**Last Updated:** YYYY-MM-DD
**Runtime:** Node.js / Bun
**Entry Point:** src/index.ts

## API Routes

| Route | Method | Purpose |
|-------|--------|---------|
| /api/users | GET | List users |
| /api/users/:id | GET | Get user |
| /api/tasks | POST | Create task |

## Data Flow

API Route → Service Layer → Database → Response

## External Services

- PostgreSQL - Primary database
- Redis - Caching
- S3 - File storage
```

## Backend Codemap (Go)

```markdown
# Backend Architecture

**Last Updated:** YYYY-MM-DD
**Runtime:** Go 1.22
**Module:** github.com/org/project
**Entry Points:** cmd/api/main.go, cmd/worker/main.go

## Structure

cmd/
├── api/              # HTTP API server
│   └── main.go
└── worker/           # Background worker
    └── main.go

internal/
├── handlers/         # HTTP handlers
├── services/         # Business logic
├── repository/       # Data access layer
├── models/           # Domain models
└── config/           # Configuration

pkg/
└── client/           # Public SDK

## Key Packages

| Package | Purpose | Exports |
|---------|---------|---------|
| internal/handlers | HTTP handlers | UserHandler, AuthHandler |
| internal/services | Business logic | UserService, OrderService |
| internal/repository | Database access | UserRepo, OrderRepo |
| pkg/client | Public SDK | Client, NewClient() |

## API Routes

| Route | Method | Handler |
|-------|--------|---------|
| /api/v1/users | GET | handlers.ListUsers |
| /api/v1/users/:id | GET | handlers.GetUser |
| /api/v1/orders | POST | handlers.CreateOrder |

## Data Flow

Request → Router → Handler → Service → Repository → Database

## Dependencies (go.mod)

- github.com/gin-gonic/gin v1.9 - HTTP framework
- github.com/jackc/pgx/v5 - PostgreSQL driver
- go.uber.org/zap - Structured logging
- github.com/redis/go-redis/v9 - Redis client
```

## Integrations Codemap

```markdown
# External Integrations

**Last Updated:** YYYY-MM-DD

## Authentication
- Provider: Auth0 / Clerk / Privy
- Methods: OAuth, email, wallet

## Database
- Provider: PostgreSQL / Supabase
- ORM: Prisma / Drizzle

## Search
- Provider: Elasticsearch / Typesense
- Features: Full-text, vector search

## Storage
- Provider: S3 / Cloudflare R2
- Usage: File uploads, assets
```

## Database Codemap

```markdown
# Database Schema

**Last Updated:** YYYY-MM-DD
**Provider:** PostgreSQL
**ORM:** Prisma

## Tables

| Table | Purpose | Key Relations |
|-------|---------|---------------|
| users | User accounts | has many tasks |
| tasks | Work items | belongs to user |
| comments | Task comments | belongs to task |

## Indexes

- users.email (unique)
- tasks.user_id (foreign key)
- tasks.status (filtered queries)

## Migrations

Located in: prisma/migrations/
```
