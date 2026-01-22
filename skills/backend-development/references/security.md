# Backend Security

OWASP Top 10 mitigation and security best practices (2026).

## OWASP Top 10 (2026)

### 1. Broken Access Control (28% of vulnerabilities)

**Mitigation:**
- RBAC with deny-by-default
- Server-side authorization (never client-side)
- Log access control failures

```typescript
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('admin')
async deleteUser(@Param('id') id: string) {
  return this.usersService.delete(id);
}
```

### 2. Cryptographic Failures

**Mitigation:**
- Argon2id for passwords
- TLS 1.3 for transit
- AES-256 for data at rest
- crypto.randomBytes() for tokens

### 3. Injection (98% reduction with parameterized queries)

```typescript
// BAD: SQL injection vulnerable
const query = `SELECT * FROM users WHERE email = '${email}'`;

// GOOD: Parameterized query
const query = 'SELECT * FROM users WHERE email = $1';
const result = await db.query(query, [email]);
```

### 4. Insecure Design

**Mitigation:**
- Threat modeling during design
- Defense in depth
- Principle of least privilege

### 5. Security Misconfiguration

**Mitigation:**
- Remove default accounts
- Use security headers
- Minimize attack surface

```typescript
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: { defaultSrc: ["'self'"] },
  },
  hsts: { maxAge: 31536000, includeSubDomains: true },
}));
```

### 6. Vulnerable Components

```bash
# Check dependencies
npm audit fix
```

### 7. Authentication Failures

**Mitigation:**
- MFA for admin accounts
- Rate limiting (10 attempts/minute)
- Strong passwords (12+ chars)
- Session timeout (15 min idle)

### 8. Software & Data Integrity

**Mitigation:**
- Code signing
- Lock file integrity
- Secure CI/CD pipelines

### 9. Logging & Monitoring Failures

**Mitigation:**
- Log authentication events
- Centralized logging
- Alerting on suspicious patterns

### 10. SSRF (Server-Side Request Forgery)

**Mitigation:**
- Validate/sanitize URLs
- Allow-list remote resources
- Network segmentation

## Input Validation (Prevents 70%+ vulnerabilities)

### Type Validation (NestJS)

```typescript
import { IsEmail, IsString, MinLength, IsInt, Min } from 'class-validator';

class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(12)
  password: string;

  @IsInt()
  @Min(18)
  age: number;
}
```

### Sanitization

```typescript
import DOMPurify from 'isomorphic-dompurify';
const clean = DOMPurify.sanitize(userInput);
```

### Allow-lists

```typescript
const allowedFields = ['name', 'email', 'age'];
const sanitized = Object.keys(input)
  .filter(key => allowedFields.includes(key))
  .reduce((obj, key) => ({ ...obj, [key]: input[key] }), {});
```

## Rate Limiting

```typescript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  message: 'Too many requests',
});

app.use('/api/', limiter);
```

**Recommended Limits:**
- Authentication: 10/15 min
- Public APIs: 100/15 min
- Authenticated: 1000/15 min

## Security Headers

```typescript
{
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
  'Content-Security-Policy': "default-src 'self'",
  'X-Frame-Options': 'DENY',
  'X-Content-Type-Options': 'nosniff',
  'Referrer-Policy': 'strict-origin-when-cross-origin',
}
```

## Secrets Management

### Best Practices

1. Never commit secrets
2. Environment-specific secrets
3. Rotate every 90 days
4. Encrypt at rest
5. Least privilege access

### Tools

- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault

```typescript
const dbPassword = process.env.DB_PASSWORD;
if (!dbPassword) throw new Error('DB_PASSWORD not set');
```

## Security Checklist

- [ ] HTTPS/TLS 1.3 only
- [ ] OAuth 2.1 + JWT
- [ ] Rate limiting
- [ ] Input validation
- [ ] Parameterized queries
- [ ] Security headers
- [ ] CORS configured (not `*`)
- [ ] Error messages sanitized
- [ ] Authentication logging
- [ ] MFA for admins
- [ ] Regular audits
