---
name: backend-development
description: PROACTIVELY build backend APIs with Node.js/TypeScript and Go. Use when designing APIs, implementing auth, or building microservices. Applies Clean Architecture, SOLID, DRY, YAGNI, KISS principles.
tools: Read, Edit, Bash, Grep, Glob
model: opus
---

# Backend Development

Node.js/TypeScript and Go. PostgreSQL, Redis. REST APIs.

> **Related Skills:**
>
> - `kavak-documentation` - **USE FIRST** for Kavak-specific patterns, kbroker, STS, GitLab CI, Docker templates
> - `test-driven-development` - **USE PROACTIVELY** for new features and bug fixes (Red-Green-Refactor)
> - Check `.claude/CLAUDE.md` or `.cursor/rules/*` for project-specific conventions
>
> **MCP**: Use `kavak-platform/plati_query` tool to query Kavak internal documentation before implementing.

## 🔴 MANDATORY: Code Reuse Analysis (GATE)

**This is a GATE. You cannot write new code until you complete this analysis.**

### Why This Matters

AI-generated code often duplicates existing logic because the model doesn't "see" all relevant files. This creates:
- Bugs that need fixing in multiple places
- Inconsistent behavior across the codebase
- Maintenance nightmare for humans

**Your job: Find existing code FIRST, reuse it, extend it, or extract shared logic.**

### Step 1: Search Comprehensively

**STOP and think:** Before searching, extract keywords from YOUR task:
1. **VERBS** - What actions? (create, save, send, process, calculate, update, delete, etc.)
2. **NOUNS** - What domain concepts? (the entities mentioned in your task)
3. **OUTPUT** - What type of result? (Model, Response, Event, etc.)

**Then search using YOUR task's actual keywords:**

```bash
# Search pattern - replace KEYWORD with your actual task keywords:
grep -rn "KEYWORD" --include="*.go" --include="*.ts" . | head -50

# Search for existing services/usecases with your domain term:
grep -rn "func.*KEYWORD\|KEYWORD.*Service\|KEYWORD.*Repository" --include="*.go" . | head -30

# Use codebase_search for semantic search:
codebase_search "how does existing code [your task action]"
codebase_search "where is [your domain concept] implemented"
```

**The goal:** Find ANY existing code that does something similar to what your task needs.

### Step 2: Read and Evaluate Found Code

For EACH potentially relevant file, read it and answer:

| Question | If YES... |
|----------|-----------|
| Does this do exactly what I need? | **Call it directly** - inject as dependency |
| Does this do 70%+ of what I need? | **Extend it** - add method/parameter |
| Does this share 50%+ logic with what I need? | **Extract shared helper first** |
| Is this only 30% similar? | OK to create new, but document why |

### Step 3: Document Your Decision (MANDATORY - Must Output This)

**Before writing ANY code, you MUST output this analysis in your response:**

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ REUSE ANALYSIS                                                  ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃ Task: [brief description of what you need to implement]         ┃
┃                                                                 ┃
┃ Keywords searched:                                              ┃
┃   - Verbs: [actual verbs you searched]                          ┃
┃   - Nouns: [actual domain terms you searched]                   ┃
┃                                                                 ┃
┃ Existing code found:                                            ┃
┃   1. [file:line] - [function name] - [what it does]             ┃
┃   2. [file:line] - [function name] - [what it does]             ┃
┃                                                                 ┃
┃ Similarity assessment:                                          ┃
┃   - [file1]: [X]% similar because [reason]                      ┃
┃                                                                 ┃
┃ Decision: [REUSE | EXTEND | EXTRACT | NEW]                      ┃
┃ Rationale: [1-2 sentences why]                                  ┃
┃ Action: [specific action - e.g., "inject existing service"]     ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

**If you skip this analysis or create code without it, the task FAILS.**

### Step 4: Apply Decision Matrix

| Similarity | Action | Example |
|------------|--------|---------|
| **>70%** | **REUSE directly** | Inject existing service as dependency, call its method |
| **50-70%** | **EXTRACT shared helper** | Create helper function, refactor existing + use in new |
| **30-50%** | **Consider interface** | Create interface both can implement |
| **<30%** | **OK to create new** | Document why existing code doesn't fit |

### 🔴 Failure Conditions

Your implementation FAILS the Code Reuse Analysis if:
- [ ] You create a new file >100 lines without documenting reuse analysis
- [ ] You duplicate >50% of logic from an existing file
- [ ] You copy-paste code instead of calling existing functions
- [ ] You create a new method that does the same thing as an existing one

---

## Core Principles

**Architecture:** Clean Architecture (4-layer) for new projects → `references/architecture.md`

**Design Principles:**

- **SOLID** - Single Responsibility, Open/Closed, Liskov, Interface Segregation, Dependency Inversion
- **DRY** - Don't Repeat Yourself (extract reusable code) → `references/dry-detection.md`
- **YAGNI** - You Aren't Gonna Need It (don't build unused features)
- **KISS** - Keep It Simple, Stupid (simplest solution that works)

**Comments Rule:** Code should be self-documenting. Only add comments when:

- Logic is complex/non-obvious (explain "why", not "what")
- Workarounds or edge cases need context
- Public API documentation (JSDoc/GoDoc)

```typescript
// ❌ BAD: Obvious comment
const age = user.age; // Get user age

// ✅ GOOD: Explains WHY
// Add 1 day buffer for timezone edge cases in billing cycle
const billingDate = addDays(cycleEnd, 1);
```

See `references/code-quality.md` for detailed examples.

## ⚠️ Existing Projects Rule

**ALWAYS respect existing project structure and patterns:**

1. **Read first** - Explore the codebase before making changes
2. **Follow existing patterns** - If project uses a pattern, continue using it
3. **Don't force architecture** - Don't refactor to Clean Architecture unless asked
4. **Incremental improvement** - Apply good practices to NEW code you write
5. **Ask if unclear** - When patterns conflict, ask the user

```
Existing project has flat structure? → Keep flat structure
Existing project has no interfaces? → Don't add interfaces everywhere
Existing project uses callbacks? → Use callbacks (unless migrating)
```

**Only apply Clean Architecture patterns to:**

- New greenfield projects
- New modules/features in existing projects (when it fits)
- When explicitly asked to refactor

## Quick Start

| Language | Test            | Lint                |
| -------- | --------------- | ------------------- |
| Node/TS  | `npm test`      | `npm run lint`      |
| Go       | `go test ./...` | `golangci-lint run` |

## Technology Selection

| Need               | Choose                             |
| ------------------ | ---------------------------------- |
| TypeScript API     | NestJS                             |
| High concurrency   | Go + Chi                           |
| Go database        | pgx/v5 + sqlc                      |
| Node database      | Drizzle (new), keep existing       |
| Go Redis           | rueidis (new), keep existing       |
| Go job queue       | River (PostgreSQL-backed)          |
| Node job queue     | **pg-boss** (PostgreSQL-backed)    |
| **Events (Kavak)** | **kbroker** (Kafka-by-REST)        |
| **Rate limiting**  | pg-boss throttle / River snooze    |
| Testing            | Vitest (new Node), Jest (existing) |
| Tracing            | OpenTelemetry + dd-trace           |

> **Note:** Use kbroker for events between services. Use pg-boss (Node) or River (Go) for job queues.

## Common Workflows

### New API Endpoint (TDD Approach)

> **Use `test-driven-development` skill** for Red-Green-Refactor cycle

1. **Write failing test first** → `test-driven-development` skill
2. Design endpoint → `references/api-design.md`
3. Set up handler → `references/go/http-handlers.md` or `references/node/frameworks.md`
4. Add database access → `references/go/database.md` or `references/node/database.md`
5. Implement auth → `references/authentication.md`
6. Refactor with tests passing → `references/code-quality.md`

### Add Similar Feature (DRY-First Approach)

> **CRITICAL: When task says "do X like Y" or "add similar to existing"**

1. **Find existing implementation first** - search for the feature mentioned in the task
2. **Analyze similarity** → Is >50% of logic the same?
3. **If similar, REUSE or EXTRACT:**
   - **>70% similar**: Inject existing service, call its method
   - **50-70% similar**: Extract shared helper, refactor existing to use it
   - **<50% similar**: OK to create new, but keep it small
4. **New code should be <50 lines** - if more, you probably missed reuse opportunity
5. **Test both old and new paths** → Ensure refactoring didn't break existing

**WRONG vs RIGHT:**

| Approach | Result |
|----------|--------|
| ❌ WRONG | Create new file (50-100+ lines) that duplicates existing logic |
| ✅ RIGHT | Inject existing service, call its method (5-10 lines) |

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

| Reference                        | When to Use                                                |
| -------------------------------- | ---------------------------------------------------------- |
| **Go**                           |                                                            |
| `references/go/http-handlers.md` | Set up Chi routes, handlers, middleware                    |
| `references/go/database.md`      | Implement pgx/v5, sqlc queries                             |
| `references/go/patterns.md`      | Apply error handling, validation, testing                  |
| `references/go/redis-queues.md`  | Add rueidis caching, River job queue                       |
| **Node.js**                      |                                                            |
| `references/node/frameworks.md`  | Set up NestJS, Express, Fastify                            |
| `references/node/database.md`    | Implement Drizzle, Prisma, caching                         |
| `references/node/patterns.md`    | Apply validation, errors, testing                          |
| **DevOps**                       |                                                            |
| `references/devops/docker.md`    | Write Dockerfiles, compose, health checks, DataDog metrics |
| `references/devops/ci-cd.md`     | Configure GitLab CI pipelines                              |
| **Cross-Cutting**                |                                                            |
| `references/architecture.md`     | **Clean Architecture (4-layer)**, microservices, events    |
| `references/code-quality.md`     | **SOLID, DRY, YAGNI, KISS**, design patterns               |
| `references/dry-detection.md`    | **MANDATORY** - DRY detection, code reuse patterns         |
| `references/api-design.md`       | Design REST endpoints, versioning                          |
| `references/authentication.md`   | Implement OAuth 2.1, JWT, RBAC                             |
| `references/security.md`         | Apply OWASP Top 10, input validation                       |
| `references/performance.md`      | Optimize caching, queries, pooling                         |
| `references/testing.md`          | Write unit, integration, E2E tests                         |
| `references/debugging.md`        | Debug with logs, profilers, tracing                        |

## Kavak-Only References

Use when working on Kavak projects.

| Reference                               | When to Use                                      |
| --------------------------------------- | ------------------------------------------------ |
| `references/kavak/kbroker.md`           | **Publish/subscribe events** between services    |
| `references/kavak/queues-ratelimit.md`  | **Job queues, rate limiting** with River/pg-boss |
| `references/kavak/microservice-auth.md` | Authenticate between services (STS)              |
| `references/kavak/gitlab-ci.md`         | Set up kavak-it/ci-jobs v3 pipelines             |
| `references/kavak/docker-images.md`     | Build with docker-debian base                    |
| `references/kavak/workload-config.md`   | Configure `.kavak/` dir, cron jobs               |
| `references/kavak/logging-metrics.md`   | Use kvklog, kvkmetric SDKs                       |
