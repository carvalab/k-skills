# GitLab CI/CD

Pipeline configuration, deployment strategies.

## Deployment Strategies

### Blue-Green

```
Production → Blue (v1.0)
             Green (v2.0) ← Deploy & Test

Switch:
Production → Green (v2.0)
             Blue (v1.0) ← Instant rollback
```

**Pros:** Zero downtime, instant rollback. **Cons:** Double infrastructure.

### Canary

Gradual rollout: 1% → 5% → 25% → 100%. Monitor metrics before proceeding.

### Feature Flags

```typescript
const showNewFeature = await client.variation('new-feature', user, false);
if (showNewFeature) {
  return newFeatureFlow(req, res);
}
```

Decouple deployment from release. A/B testing. Kill switch for problems.

## Basic Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

test:
  stage: test
  image: node:24-slim
  script:
    - npm ci
    - npm run lint
    - npm run test:ci
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      junit: junit.xml

test:go:
  stage: test
  image: golang:1.25
  script:
    - go mod download
    - golangci-lint run
    - go test -v -race -coverprofile=coverage.out ./...
  coverage: '/total:\s+\(statements\)\s+(\d+\.\d+)%/'

security:
  stage: test
  image: node:24-slim
  script:
    - npm audit --audit-level=high
  allow_failure: true

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
  only:
    - main
    - develop

deploy:staging:
  stage: deploy
  image: alpine:3.19
  script:
    - apk add --no-cache curl
    - |
      curl -X POST \
        -H "Authorization: Bearer $DEPLOY_TOKEN" \
        -d '{"image": "'$DOCKER_IMAGE'"}' \
        $DEPLOY_WEBHOOK_URL
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy:production:
  stage: deploy
  image: alpine:3.19
  script:
    - apk add --no-cache curl
    - |
      curl -X POST \
        -H "Authorization: Bearer $DEPLOY_TOKEN" \
        -d '{"image": "'$DOCKER_IMAGE'"}' \
        $DEPLOY_WEBHOOK_URL
  environment:
    name: production
    url: https://api.example.com
  when: manual
  only:
    - main
```

## Go Pipeline

```yaml
test:go:
  stage: test
  image: golang:1.25
  variables:
    GOPATH: $CI_PROJECT_DIR/.go
  cache:
    paths:
      - .go/pkg/mod/
  script:
    - go mod download
    - golangci-lint run --timeout 5m
    - go test -v -race -coverprofile=coverage.out ./...
    - go tool cover -func=coverage.out
  coverage: '/total:\s+\(statements\)\s+(\d+\.\d+)%/'
```

## Environment Variables

```yaml
variables:
  NODE_ENV: production
  LOG_LEVEL: info

# In GitLab Settings > CI/CD > Variables (masked/protected)
# DATABASE_URL, REDIS_URL, JWT_SECRET, DEPLOY_TOKEN
```
