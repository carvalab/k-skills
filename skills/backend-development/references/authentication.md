# Backend Authentication & Authorization

OAuth 2.1, JWT, RBAC, and MFA patterns (2026).

## OAuth 2.1 (2026 Standard)

### Key Changes from OAuth 2.0

**Mandatory:**

- PKCE for all clients
- Exact redirect URI matching
- State parameter for CSRF

**Deprecated:**

- Implicit grant flow
- Resource owner password credentials
- Bearer token in query strings

### Authorization Code Flow with PKCE

```typescript
import crypto from 'crypto';

// Step 1: Generate code verifier and challenge
const codeVerifier = crypto.randomBytes(32).toString('base64url');
const codeChallenge = crypto.createHash('sha256').update(codeVerifier).digest('base64url');

// Step 2: Build authorization URL
const authUrl = new URL('https://auth.example.com/authorize');
authUrl.searchParams.set('client_id', 'your-client-id');
authUrl.searchParams.set('redirect_uri', 'https://app.example.com/callback');
authUrl.searchParams.set('response_type', 'code');
authUrl.searchParams.set('scope', 'openid profile email');
authUrl.searchParams.set('state', crypto.randomBytes(16).toString('hex'));
authUrl.searchParams.set('code_challenge', codeChallenge);
authUrl.searchParams.set('code_challenge_method', 'S256');

// Step 3: Exchange code for token
const tokenResponse = await fetch('https://auth.example.com/token', {
  method: 'POST',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body: new URLSearchParams({
    grant_type: 'authorization_code',
    code: authCode,
    redirect_uri: redirectUri,
    client_id: clientId,
    code_verifier: codeVerifier,
  }),
});
```

## JWT (JSON Web Tokens)

### Best Practices

- **Short expiration:** Access: 15 min, Refresh: 7 days
- **Use RS256:** Asymmetric signing for public APIs
- **Validate everything:** Signature, issuer, audience, expiration
- **Minimal claims:** Don't include sensitive data
- **Refresh token rotation:** New refresh token on each use

### Implementation

```typescript
import jwt from 'jsonwebtoken';

// Generate JWT
const accessToken = jwt.sign({ sub: user.id, email: user.email, roles: user.roles }, process.env.JWT_PRIVATE_KEY, {
  algorithm: 'RS256',
  expiresIn: '15m',
  issuer: 'https://api.example.com',
  audience: 'https://app.example.com',
});

// Verify JWT
const decoded = jwt.verify(token, process.env.JWT_PUBLIC_KEY, {
  algorithms: ['RS256'],
  issuer: 'https://api.example.com',
  audience: 'https://app.example.com',
});
```

### Go JWT

```go
import "github.com/golang-jwt/jwt/v5"

type Claims struct {
    UserID string   `json:"sub"`
    Email  string   `json:"email"`
    Roles  []string `json:"roles"`
    jwt.RegisteredClaims
}

func GenerateToken(user *User, privateKey *rsa.PrivateKey) (string, error) {
    claims := Claims{
        UserID: user.ID,
        Email:  user.Email,
        Roles:  user.Roles,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(15 * time.Minute)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            Issuer:    "https://api.example.com",
            Audience:  []string{"https://app.example.com"},
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodRS256, claims)
    return token.SignedString(privateKey)
}

func ValidateToken(tokenString string, publicKey *rsa.PublicKey) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(t *jwt.Token) (interface{}, error) {
        return publicKey, nil
    })
    if err != nil {
        return nil, err
    }

    claims, ok := token.Claims.(*Claims)
    if !ok || !token.Valid {
        return nil, errors.New("invalid token")
    }
    return claims, nil
}
```

## RBAC (Role-Based Access Control)

### NestJS Implementation

```typescript
export enum Role { ADMIN = 'admin', EDITOR = 'editor', VIEWER = 'viewer' }

export const Roles = (...roles: Role[]) => SetMetadata('roles', roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<Role[]>('roles', context.getHandler());
    if (!requiredRoles) return true;

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some((role) => user.roles?.includes(role));
  }
}

// Usage
@Post()
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles(Role.ADMIN, Role.EDITOR)
async createPost(@Body() dto: CreatePostDto) {
  return this.postsService.create(dto);
}
```

### Go RBAC Middleware

```go
func RequireRoles(roles ...string) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            claims := r.Context().Value("claims").(*Claims)

            hasRole := false
            for _, required := range roles {
                for _, userRole := range claims.Roles {
                    if required == userRole {
                        hasRole = true
                        break
                    }
                }
            }

            if !hasRole {
                http.Error(w, "Forbidden", http.StatusForbidden)
                return
            }

            next.ServeHTTP(w, r)
        })
    }
}

// Usage with Chi
r.With(RequireRoles("admin", "editor")).Post("/posts", createPost)
```

## MFA (Multi-Factor Authentication)

### TOTP

```typescript
import speakeasy from 'speakeasy';
import QRCode from 'qrcode';

// Generate secret
const secret = speakeasy.generateSecret({ name: 'MyApp', issuer: 'MyCompany' });
const qrCode = await QRCode.toDataURL(secret.otpauth_url);

// Verify token
const verified = speakeasy.totp.verify({
  secret: secret.base32,
  encoding: 'base32',
  token: userToken,
  window: 2,
});
```

## Password Security

### Argon2id (2026 Standard)

```typescript
import argon2 from 'argon2';

// Hash
const hash = await argon2.hash('password123', {
  type: argon2.argon2id,
  memoryCost: 65536, // 64 MB
  timeCost: 3,
  parallelism: 4,
});

// Verify
const valid = await argon2.verify(hash, 'password123');
```

### Go Argon2id

```go
// Go: golang.org/x/crypto/argon2
hash := argon2.IDKey([]byte(password), salt, 3, 64*1024, 4, 32)
```

### Password Policy (NIST)

- Minimum 12 characters
- No composition rules (allow passphrases)
- Check against breach databases
- No periodic rotation (only on compromise)

## Session Management

```typescript
import session from 'express-session';
import RedisStore from 'connect-redis';

app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: true, // HTTPS only
      httpOnly: true, // No JavaScript access
      sameSite: 'strict', // CSRF protection
      maxAge: 1000 * 60 * 15, // 15 minutes
    },
  })
);
```

## API Key Authentication

```typescript
// Generate API key
const apiKey = `sk_${env}_${crypto.randomBytes(24).toString('base64url')}`;

// Store hashed
const hashedKey = crypto.createHash('sha256').update(apiKey).digest('hex');
await db.apiKeys.create({ userId, hashedKey, scopes: ['read'] });

// Validate
const providedHash = crypto.createHash('sha256').update(providedKey).digest('hex');
const keyRecord = await db.apiKeys.findOne({ hashedKey: providedHash });
```

## Authentication Decision Matrix

| Use Case         | Approach               |
| ---------------- | ---------------------- |
| Web/Mobile/SPA   | OAuth 2.1 + PKCE + JWT |
| Server-to-server | Client credentials     |
| Third-party API  | API keys with scopes   |
| High-security    | WebAuthn + MFA         |

**Security essentials:** OAuth 2.1 + PKCE, 15min JWT expiry, refresh token rotation, RBAC deny-by-default, Argon2id passwords, secure cookies (HttpOnly, Secure, SameSite), rate limiting, audit logging.
