# Kavak GitLab CI Pipelines

**Kavak-only.** Use Kavak's v3 CI pipelines for all applications.

## Pipeline Configuration

### Go Application

```yaml
# .gitlab-ci.yml
include:
  - project: 'kavak-it/ci-jobs'
    ref: 'main'
    file: 'v3/golang-containerd-1.25.yml'
```

### Node.js Application

```yaml
# .gitlab-ci.yml
include:
  - project: 'kavak-it/ci-jobs'
    ref: 'main'
    file: 'v3/node-containerd-24.yml'
```

### Java Application

```yaml
# .gitlab-ci.yml
include:
  - project: 'kavak-it/ci-jobs'
    ref: 'main'
    file: 'v3/java-containerd-17.yml'
```

### Python Application

```yaml
# .gitlab-ci.yml
include:
  - project: 'kavak-it/ci-jobs'
    ref: 'main'
    file: 'v3/python-containerd-3.13.yml'
```

## Pipeline Stages

| Stage | Jobs | Runs On |
|-------|------|---------|
| **dependencies** | npm install (Node only) | MRs, Tags, Main |
| **test** | lint, test, secrets-scanner, dockerfile-scanner | MRs, Tags |
| **build-snapshot** | build_container_image_snapshot_{amd64,arm64} | MRs |
| **publish-snapshot** | publish_container_image_snapshot | MRs |
| **integration-tests** | integration-tests (if enabled) | MRs |
| **build-release** | build_container_image_release_{amd64,arm64} | Tags, Main |
| **publish-release** | publish_container_image_release | Tags, Main |
| **publish-docs** | publish-techdocs | Main |

## Test Stage Jobs

| Job | Languages | Purpose |
|-----|-----------|---------|
| `lint_kavak_yaml_files` | All | Lint `.kavak/` YAML files |
| `auth_validate_config_{env}` | All | Validate `.kavak/auth/auth.{env}.yaml` |
| `secrets-scanner` | All | Scan for leaked secrets |
| `dockerfile-scanner` | All | Scan Dockerfile vulnerabilities |
| `validate_openapi_spec` | All | Validate `.kavak/openapi.yaml` |
| `test` | All except Python | Run unit tests |
| `lint` | Go only | `go mod tidy` check |
| `coverage_report` | All except Python | Push coverage to metrics API |

## Integration Tests

Enable integration tests with [hurl](https://hurl.dev/):

```yaml
# .gitlab-ci.yml
include:
  - project: 'kavak-it/ci-jobs'
    ref: 'main'
    file: 'v3/golang-containerd-1.25.yml'

variables:
  ENABLE_INTEGRATION_TESTS: '1'

integration_tests:
  variables:
    KAVAK_ENVIRONMENT: local
  services:
    - name: 306027853247.dkr.ecr.us-west-2.amazonaws.com/$CI_PROJECT_NAME-snapshot:$CI_COMMIT_SHORT_SHA
      alias: myapp
```

**Required directories:**
- `resources/mock/` - HTTP service mocks (create `.gitkeep` if empty)
- `test/integration/` - `.hurl` test files

**Debug services:**
```yaml
integration_tests:
  variables:
    CI_DEBUG_SERVICES: "true"  # Prints service logs to job output
```

## Multi-Architecture Support

All images are built for both `amd64` and `arm64`. If downloading architecture-specific binaries:

```dockerfile
ARCH=$(case "$(uname -m)" in \
    'aarch64'|'aarch64_be'|'armv8b'|'armv8l') echo "arm64" ;; \
    *) echo "amd64" ;; \
esac)

curl -fsSL -o /usr/local/bin/tool \
    "https://example.com/releases/v1.0.0/tool-linux-${ARCH}"
```

## Custom Variables

Add memory for OOM issues (only on specific job):

```yaml
build_container_image_release_amd64:
  variables:
    KUBERNETES_MEMORY_REQUEST: '6Gi'
    KUBERNETES_MEMORY_LIMIT: '6Gi'
```

## Dockerfile Best Practices

```dockerfile
# Order by least likely to change
COPY go.mod go.sum ./
RUN go mod download

# Copy ONLY needed files (avoid COPY . .)
COPY cmd/ cmd/
COPY internal/ internal/

# Compile
RUN CGO_ENABLED=0 go build -o app ./cmd/server
```

**Rules:**
- Use multi-stage builds
- Never use `COPY . .` (invalidates cache, includes `.git`)
- Use `.dockerignore` to exclude files
- Use `--link` for slow COPY operations

## Secret Variables

Add CI/CD secrets via Vault. See [secrets documentation](https://developer-portal.prd.kavak.io/docs/default/Component/docs/services-and-tools/application-management/secrets.md#cicd-environment-variables-new).

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **Image build slow** | Check `COPY . .` usage, follow best practices |
| **Permission errors** | Check user ownership, `chown` before commands |
| **Job killed** | OOM - add more memory to specific job |
| **TLS internal error** | Known GitLab issue - retry the job |

## Resources

- [CI Jobs Repository](https://gitlab.com/kavak-it/ci-jobs/-/tree/main/v3)
- [Available Versions](https://gitlab.com/kavak-it/ci-jobs/-/tree/main/v3?ref_type=heads)
