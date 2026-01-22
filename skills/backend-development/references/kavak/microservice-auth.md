# Kavak Microservices Authentication (STS)

**Kavak-only.** STS-API authentication for service-to-service communication.

## Overview

Kavak uses **STS-API** (Security Token Service) for all microservice authentication:

- **Token Generation**: M2M tokens with specific scopes/audiences
- **Token Introspection**: Validate and extract claims
- **Token Exchange**: Convert external tokens (Google, Cognito, API Keys) to STS tokens

## Auth SDK Versions

| SDK | Version | Import Path |
|-----|---------|-------------|
| Go (1.19-1.24) | v1.2.2 | `gitlab.com/kavak-it/auth-go-sdk/auth` |
| Go (1.25+) | v2.0.3 | `gitlab.com/kavak-it/auth-go-sdk/v2/auth` |
| Node.js | v4.2.0+ | `@kavak/platform-auth-service-sdk` |
| Java | v1.1.2 / v2.6.2 (Spring) | `com.kavak.platform.authsdk` |
| Python | v1.1.0+ | `kavak_auth` |

## Go SDK Usage

```go
// For Go 1.25+
import "gitlab.com/kavak-it/auth-go-sdk/v2/auth"

// Create client (cache enabled by default)
client := auth.NewInternalClient()

// Or customize cache
client := auth.NewInternalClient(
    auth.WithCache(auth.NewCache(5000)), // 5000 max items
)

// Generate M2M token
input := &auth.GenerateTokenInput{
    Audience: []string{"users-api"},
    Scopes:   []string{"users-api.public"},
}
output, err := client.GenerateToken(ctx, input)
token := output.Token

// Validate token
validateInput := &auth.ValidateTokenInput{
    Token: token,
}
result, err := client.ValidateToken(ctx, validateInput)
if result.Active {
    // Token is valid
    fmt.Println(result.Sub, result.Scopes)
}
```

### TokenHolder (Auto-Refresh)

For long-running services, use TokenHolder to auto-refresh tokens:

```go
// Generate with auto-refresh (use ONLY at startup, not per-request)
input := &auth.GenerateTokenInput{
    Audience:                  []string{"users-api"},
    Scopes:                    []string{"users-api.public"},
    RefreshTokenAutomatically: true, // Refreshes 120s before expiry
}
output, _ := client.GenerateToken(ctx, input)

// Store globally, reuse everywhere
tokenHolder := output.TokenHolder

// In request handlers:
token := tokenHolder.GetToken() // Always valid
```

**CRITICAL**: Generate TokenHolder ONCE at startup, reuse everywhere. Never call `generateToken` with `refreshTokenAutomatically: true` per-request.

## Node.js SDK Usage

```typescript
import { createClient } from '@kavak/platform-auth-service-sdk';

// Create client (cache enabled by default)
const client = createClient({
    clientId: 'my-app',
    clientSecret: 'my-secret',
});

// Generate M2M token
const result = await client.generateToken({
    audience: ['users-api'],
    scopes: ['users-api.public'],
});

// Validate token
const validation = await client.validateToken({ token });
if (validation.active) {
    console.log(validation.sub, validation.scopes);
}

// TokenHolder for auto-refresh (ONLY at startup)
let tokenHolder: TokenHolder;

async function startup() {
    const result = await client.generateToken({
        audience: ['users-api'],
        scopes: ['users-api.public'],
        refreshTokenAutomatically: true,
    });
    tokenHolder = result.tokenHolder;
}

// In handlers - always use cached tokenHolder
app.get('/api/data', async (req, res) => {
    const token = tokenHolder.getToken();
    // ...
});
```

## Token Exchange (User Context)

Exchange external tokens for STS tokens to propagate user context:

```go
// Exchange Google token for STS token
exchangeInput := &auth.ExchangeTokenInput{
    SubjectToken:     googleToken,
    SubjectTokenType: "urn:kavak:token-type:google",
    Audience:         []string{"users-api"},
    Scopes:           []string{"users-api.public"},
}
result, err := client.ExchangeToken(ctx, exchangeInput)
```

### Token Types

| IDP | Token Type |
|-----|------------|
| Cognito | `urn:kavak:token-type:cognito` |
| Okta | `urn:kavak:token-type:okta` |
| Google | `urn:kavak:token-type:google` |
| Forgerock | `urn:kavak:token-type:forgerock` |
| API Key | `urn:kavak:token-type:apikey` |
| Session | `urn:kavak:token-type:session` |

## Custom Claims

STS tokens include Kavak-specific claims:

| Claim | Key | Description |
|-------|-----|-------------|
| Client ID | `client_id` | Application that generated the token |
| Initial Token Type | `itt` | Original token type before exchange |
| IDP User ID | `iui` | User ID from identity provider |
| Kavak User ID | `k_usr_id` | Olimpo user ID |
| Kavak User Email | `k_usr_email` | User email |
| Kavak Groups | `k_groups` | User group memberships |
| Kavak Tenant | `k_tenant` | Tenant identifier |

## Caching

SDK caching is enabled by default but requires server-side configuration:

```go
// Check if caching is working
// Response includes: { "cacheable": true, "cache_ttl": 600 }
```

**Cache Configuration** (request from Platform team via `kavak automation` CLI):

| Operation | Cacheable | Recommended TTL |
|-----------|-----------|-----------------|
| Token Introspection | Yes | `min(token_expiry, 300s)` |
| Token Generation | Yes (via TokenHolder) | Token lifetime - 120s |
| Token Exchange | Yes | `min(token_expiry, 60s)` |

## Environment Variables

Auto-injected by manifest-api:

| Variable | Description |
|----------|-------------|
| `STS_API_BASE_URL` | STS-API endpoint (internal: `http://sts-api.svc.kavak.io`) |
| `KAVAK_ENVIRONMENT` | Current environment (`development` or `production`) |

## Auth Configuration (.kavak/auth/)

Configure your application's auth requirements:

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

## Performance

| Operation | Typical Latency |
|-----------|-----------------|
| Token Generation (cache hit) | < 0.1ms |
| Token Generation (cache miss) | 3-8ms |
| Token Introspection (STS token) | < 1ms |
| Token Exchange | 10-100ms |

## Middleware Example (Go + Chi)

```go
func AuthMiddleware(client *auth.Client) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            token := strings.TrimPrefix(r.Header.Get("Authorization"), "Bearer ")
            if token == "" {
                http.Error(w, "Unauthorized", http.StatusUnauthorized)
                return
            }

            result, err := client.ValidateToken(r.Context(), &auth.ValidateTokenInput{
                Token: token,
            })
            if err != nil || !result.Active {
                http.Error(w, "Invalid token", http.StatusUnauthorized)
                return
            }

            // Add claims to context
            ctx := context.WithValue(r.Context(), "claims", result)
            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}
```

## Resources

- [Auth Go SDK](https://gitlab.com/kavak-it/auth-go-sdk)
- [Auth Node SDK](https://gitlab.com/kavak-it/auth-node-sdk)
- [Auth Java SDK](https://gitlab.com/kavak-it/auth-java-sdk)
- [Auth Python SDK](https://gitlab.com/kavak-it/auth-python-sdk)
