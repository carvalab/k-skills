# Kavak Docker Images

**Kavak-only.** Docker images for Kavak applications. Golden images are deprecated - use `docker-debian` as base.

## ECR Registry

```bash
# Authenticate with ECR
aws --profile development ecr get-login-password --region us-west-2 | \
  docker login --username AWS --password-stdin 306027853247.dkr.ecr.us-west-2.amazonaws.com
```

## Recommended Base Images

| Purpose | Image |
|---------|-------|
| **Production runtime** | `docker-debian:trixie-slim` |
| **Go build** | `docker-golang-build:1.25` |
| **Go runtime** | `docker-golang:1.25` or `docker-debian:trixie-slim` |
| **Node.js** | `docker-node:24` |
| **Java runtime** | `docker-java:21` |
| **Python** | `docker-python:3.13` |

## Go Application (Recommended)

```dockerfile
# Build stage
FROM 306027853247.dkr.ecr.us-west-2.amazonaws.com/docker-golang-build:1.25 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY cmd/ cmd/
COPY internal/ internal/
RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o myapp ./cmd/main.go

# Runtime stage - use docker-debian (most secure)
FROM 306027853247.dkr.ecr.us-west-2.amazonaws.com/docker-debian:trixie-slim
COPY --chown=app:app .kavak /app/.kavak
COPY --from=builder --chown=app:app /app/myapp /app/myapp
EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8080/_/health || exit 1

CMD ["/app/myapp"]
```

## Node.js Application

```dockerfile
# Build stage
FROM 306027853247.dkr.ecr.us-west-2.amazonaws.com/docker-node:24 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM 306027853247.dkr.ecr.us-west-2.amazonaws.com/docker-debian:trixie-slim
COPY --chown=app:app .kavak /app/.kavak
COPY --from=builder --chown=app:app /app/dist /app/
COPY --from=builder --chown=app:app /app/node_modules /app/node_modules
EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8080/_/health || exit 1

CMD ["node", "index.js"]
```

## docker-debian Base Image

Security-hardened Debian with CIS Enhanced Level 1 compliance:

- **Non-root user**: `app` (UID/GID 1000)
- **Minimal packages**: ~129 packages (~193.5MB)
- **SBOM included**: `/sbom.txt`
- **Startup time**: <100ms

### Included Packages

- `ca-certificates` - SSL/TLS verification
- `curl` - HTTP client for health checks
- `tzdata` - Timezone data (UTC)
- `procps` - Process monitoring

### Security Features

- CIS Docker Benchmark Enhanced Level 1
- SLSA Level 3 build process
- Zero-trust security model
- Setuid/setgid removal
- Core dump protection

## Requirements

### Application Directory

If using `.kavak/` configuration, place it in `/app/.kavak`:

```dockerfile
WORKDIR /app
COPY --chown=app:app .kavak /app/.kavak
```

### Non-Root User

All containers run as UID 1000:

```dockerfile
# docker-debian already has app user
# If you need root temporarily:
USER root
RUN apt-get update && apt-get install -y --no-install-recommends your-package \
    && rm -rf /var/lib/apt/lists/*
USER app:app
```

### Health Check

**Required endpoint**: `GET /_/health` returning HTTP 200

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8080/_/health || exit 1
```

### Read-Only Filesystem

Containers have read-only filesystem. Use `/tmp` for writes or request a mounted volume from Platform team.

## Build Tools

### docker-golang-build:1.25

| Tool | Version |
|------|---------|
| Go SDK | 1.25 |
| golangci-lint | v2.4.0 |
| goreleaser | v2.11.2 |
| dbmate | v2.27.0 |

### CI Usage

```yaml
lint:
  image: 306027853247.dkr.ecr.us-west-2.amazonaws.com/docker-golang-build:1.25
  script:
    - golangci-lint run

build:
  image: 306027853247.dkr.ecr.us-west-2.amazonaws.com/docker-golang-build:1.25
  script:
    - go build -ldflags="-s -w" -o app ./cmd/main.go
```

## Multi-Architecture

All images support `amd64` and `arm64`. For custom binaries:

```dockerfile
RUN ARCH=$(case "$(uname -m)" in \
    'aarch64'|'aarch64_be'|'armv8b'|'armv8l') echo "arm64" ;; \
    *) echo "amd64" ;; \
esac) && \
curl -fsSL -o /usr/local/bin/tool \
    "https://example.com/releases/v1.0.0/tool-linux-${ARCH}"
```

## Go Environment

Pre-configured in all Go images:

```dockerfile
ENV GOPROXY=https://proxy.golang.org,direct
ENV GOSUMDB=sum.golang.org
ENV GOPRIVATE=gitlab.com/kavak-it/*
ENV GO111MODULE=on
```

## Best Practices

### Do

- Use `docker-debian:trixie-slim` for production
- Use multi-stage builds
- Pin specific version tags
- Use `COPY --chown=app:app`
- Override health check with app-specific endpoint
- Order Dockerfile by least likely to change

### Don't

- Use `COPY . .` (invalidates cache)
- Use `latest` tag
- Run as root in production
- Skip health checks
- Use golden images (deprecated)

## Security Validation

```bash
# Scan for vulnerabilities
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image your-image

# Check running as non-root
docker run --rm your-image whoami  # Should output: app

# Check SBOM
docker run --rm your-image cat /sbom.txt

# Check compliance
docker run --rm your-image cat /etc/compliance-marker
```

## Source Repositories

| Image | Repository |
|-------|------------|
| docker-debian | [gitlab.com/kavak-it/docker-debian](https://gitlab.com/kavak-it/docker-debian) |
| docker-golang-build | [gitlab.com/kavak-it/docker-golang-build](https://gitlab.com/kavak-it/docker-golang-build) |
| docker-golang | [gitlab.com/kavak-it/docker-golang](https://gitlab.com/kavak-it/docker-golang) |
| docker-node | [gitlab.com/kavak-it/docker-node](https://gitlab.com/kavak-it/docker-node) |
| docker-java | [gitlab.com/kavak-it/docker-java](https://gitlab.com/kavak-it/docker-java) |
| docker-python | [gitlab.com/kavak-it/docker-python](https://gitlab.com/kavak-it/docker-python) |
