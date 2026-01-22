# Kavak Application Configuration

**Kavak-only.** Configuration files in the `.kavak/` directory.

## Directory Structure

```
.kavak/
├── config.yaml          # Cron jobs, workload config
├── auth/
│   ├── auth.development.yaml
│   └── auth.production.yaml
└── openapi.yaml         # OpenAPI v3 spec
```

## config.yaml

### Cron Jobs

```yaml
# .kavak/config.yaml
jobs:
  - name: my-email-sender
    schedule: "0 4 * * *"      # 4 AM daily
    command:
      - "node"
      - "email-sender.js"

  - name: my-data-sync
    schedule: "*/15 * * * *"   # Every 15 minutes
    command:
      - "./bin/sync"
    resources:
      cpu: "1000m"
      memory: "1024Mi"

  - name: manual-task          # On-demand (no schedule)
    command:
      - "./bin/manual-task"
```

**Properties:**

| Property | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Job identifier (letters, numbers, hyphens) |
| `schedule` | No | Cron expression (omit for on-demand) |
| `command` | Yes | Command array [binary, ...args] |
| `resources.cpu` | No | CPU allocation (default: workload CPU) |
| `resources.memory` | No | Memory allocation (default: workload memory) |

**Cron expression tester**: https://www.crondrive.com/test-cron-expression

### On-Demand Jobs

Jobs without `schedule` are triggered manually via Argo CD:
1. Navigate to workload in Argo CD UI
2. Locate on-demand job
3. Click three dots menu → "Create Job"

### Limitations

- Maximum runtime: 24 hours
- Pods exceeding limit are terminated

## Auth Configuration

### auth.development.yaml / auth.production.yaml

```yaml
# .kavak/auth/auth.development.yaml
client_id: my-app
scopes:
  - users-api.public
  - orders-api.read
audiences:
  - users-api
  - orders-api
```

**Validated by CI**: `auth_validate_config_{env}` job runs on MRs when these files change.

## OpenAPI Specification

```yaml
# .kavak/openapi.yaml
openapi: 3.0.3
info:
  title: My API
  version: 1.0.0
paths:
  /users:
    get:
      summary: List users
      responses:
        '200':
          description: Success
```

**Requirements:**
- Must be OpenAPI v3.0.x
- Validated by CI: `validate_openapi_spec` job
- Used for Developer Portal documentation
- Can be used for MCP server generation

## Environment Variables

Auto-injected environment variables:

| Variable | Description |
|----------|-------------|
| `KAVAK_ENVIRONMENT` | `development` or `production` |
| `KAVAK_WORKLOAD_FULL_NAME` | Full workload name |
| `DD_AGENT_HOST` | Datadog agent host for metrics |
| `STS_API_BASE_URL` | STS API endpoint |

### Service-Specific Variables

Added when services are attached to product:

| Service | Variables |
|---------|-----------|
| Database | `DATABASE_URL`, `DB_HOST`, `DB_PORT`, etc. |
| Redis | `REDIS_URL`, `REDIS_HOST`, `REDIS_PORT` |
| S3 | `S3_BUCKET_NAME`, `AWS_REGION` |

## Workload Requirements

### Health Check Endpoint

**Required**: `GET /_/health` returning HTTP 200

```go
r.Get("/_/health", func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("ok"))
})
```

### Graceful Shutdown

Pods can be recycled anytime. Handle `SIGTERM`:

```go
func main() {
    srv := &http.Server{Addr: ":8080", Handler: router}

    go func() {
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()

    // Wait for SIGTERM
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGTERM)
    <-quit

    // Graceful shutdown (30s grace period)
    ctx, cancel := context.WithTimeout(context.Background(), 25*time.Second)
    defer cancel()
    srv.Shutdown(ctx)
}
```

### Application Directory

If using `.kavak/` configuration, place it in `/app/.kavak`:

```dockerfile
WORKDIR /app
COPY --chown=app:app .kavak /app/.kavak
```

### Non-Root User

Must run as UID 1000 (user `app` in docker-debian).

### Read-Only Filesystem

Filesystem is read-only. Use `/tmp` for writes or request volume from Platform team.

## Vault Secrets

Secrets managed via Vault. See [Secrets Management](https://developer-portal.prd.kavak.io/docs/default/Component/docs/services-and-tools/application-management/secrets.md).

## CLI Commands

```bash
# Get workload info
kavak platform workload get {WORKLOAD_NAME} --product {PRODUCT_NAME}

# Check CPU/memory allocation
kavak platform workload get my-api --product my-product
```

## Monitoring

- **OOM errors**: [Application Overview Dashboard](https://app.datadoghq.com/dashboard/kk5-arz-v69)
- **CPU throttling**: Same dashboard
- **Logs**: Grafana or Datadog

## Resources

- [Application Management](https://developer-portal.prd.kavak.io/docs/default/Component/docs/services-and-tools/application-management/)
- [Infrastructure Self-Service](https://developer-portal.prd.kavak.io/docs/default/Component/docs/services-and-tools/infrastructure-self-service/)
- [Argo CD](https://developer-portal.prd.kavak.io/docs/default/Component/docs/services-and-tools/application-management/argocd.md)
