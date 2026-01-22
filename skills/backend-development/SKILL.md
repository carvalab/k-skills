---
name: backend-development
description: PROACTIVELY build backend APIs with Node.js/TypeScript and Go. Use when designing APIs, implementing auth, or building microservices.
tools: Read, Edit, Bash, Grep, Glob
model: opus
---

# Backend Development

Node.js/TypeScript and Go. PostgreSQL, Redis. REST APIs.

## Quick Start

| Language | Test | Lint |
|----------|------|------|
| Node/TS | `npm test` | `npm run lint` |
| Go | `go test ./...` | `golangci-lint run` |

## Technology Selection

| Need | Choose |
|------|--------|
| TypeScript API | NestJS |
| High concurrency | Go + Chi |
| Go database | pgx/v5 + sqlc |
| Node database | Drizzle (new), keep existing |
| Go Redis | rueidis (new), keep existing |
| Go job queue | River |
| Node job queue | BullMQ |
| Testing | Vitest (new Node), Jest (existing) |
| Tracing | OpenTelemetry + dd-trace |

## Common Workflows

### New API Endpoint

1. Design endpoint → `references/api-design.md`
2. Set up handler → `references/go/http-handlers.md` or `references/node/frameworks.md`
3. Add database access → `references/go/database.md` or `references/node/database.md`
4. Implement auth → `references/authentication.md`
5. Write tests → `references/testing.md`

### Fix Slow Query

1. Profile query → `references/debugging.md` (EXPLAIN ANALYZE)
2. Add indexes → `references/performance.md`
3. Add caching → `references/go/redis-queues.md` or `references/node/database.md`

### Deploy New Service

1. Write Dockerfile → `references/devops/docker.md`
2. Set up CI/CD → `references/devops/ci-cd.md`

### Debug Production Issue

1. Check logs/traces → `references/debugging.md`
2. Profile if slow → `references/debugging.md` (pprof/clinic.js)
3. Check DB queries → `references/debugging.md` (pg_stat_statements)

## References

| Reference | When to Use |
|-----------|-------------|
| **Go** | |
| `references/go/http-handlers.md` | Set up Chi routes, handlers, middleware |
| `references/go/database.md` | Implement pgx/v5, sqlc queries |
| `references/go/patterns.md` | Apply error handling, validation, testing |
| `references/go/redis-queues.md` | Add rueidis caching, River job queue |
| **Node.js** | |
| `references/node/frameworks.md` | Set up NestJS, Express, Fastify |
| `references/node/database.md` | Implement Drizzle, Prisma, caching |
| `references/node/patterns.md` | Apply validation, errors, testing |
| **DevOps** | |
| `references/devops/docker.md` | Write Dockerfiles, compose, health checks, DataDog metrics |
| `references/devops/ci-cd.md` | Configure GitLab CI pipelines |
| **Cross-Cutting** | |
| `references/api-design.md` | Design REST endpoints, versioning |
| `references/architecture.md` | Design microservices, events |
| `references/authentication.md` | Implement OAuth 2.1, JWT, RBAC |
| `references/security.md` | Apply OWASP Top 10, input validation |
| `references/performance.md` | Optimize caching, queries, pooling |
| `references/testing.md` | Write unit, integration, E2E tests |
| `references/debugging.md` | Debug with logs, profilers, tracing |
| `references/code-quality.md` | Apply SOLID, design patterns |

## Kavak-Only References

Use when working on Kavak projects.

| Reference | When to Use |
|-----------|-------------|
| `references/kavak/microservice-auth.md` | Authenticate between services (STS) |
| `references/kavak/gitlab-ci.md` | Set up kavak-it/ci-jobs v3 pipelines |
| `references/kavak/docker-images.md` | Build with docker-debian base |
| `references/kavak/workload-config.md` | Configure `.kavak/` dir, cron jobs |
| `references/kavak/logging-metrics.md` | Use kvklog, kvkmetric SDKs |
