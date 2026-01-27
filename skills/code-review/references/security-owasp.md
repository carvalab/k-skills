# OWASP Security Checklist

Security review based on OWASP Top 10 (2021) vulnerabilities.

## A01: Broken Access Control

**Check for:**

- [ ] Authorization on every endpoint
- [ ] User can only access their own data
- [ ] Admin functions protected
- [ ] No IDOR (Insecure Direct Object Reference)

```typescript
// Bad - No authorization check
app.get('/api/users/:id', async (req, res) => {
  const user = await db.users.find(req.params.id);
  res.json(user);
});

// Good - Verify ownership
app.get('/api/users/:id', async (req, res) => {
  if (req.params.id !== req.user.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  const user = await db.users.find(req.params.id);
  res.json(user);
});
```

## A02: Cryptographic Failures

**Check for:**

- [ ] No hardcoded secrets
- [ ] Sensitive data encrypted at rest
- [ ] HTTPS for data in transit
- [ ] Strong hashing for passwords (bcrypt, argon2)

```typescript
// Bad
const password = 'admin123';
const hash = md5(userPassword);

// Good
const password = process.env.ADMIN_PASSWORD;
const hash = await bcrypt.hash(userPassword, 12);
```

## A03: Injection

**Check for:**

- [ ] Parameterized queries (no string concatenation)
- [ ] Input validation/sanitization
- [ ] ORM/query builder usage

```typescript
// Bad - SQL injection
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// Good - Parameterized
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
```

## A04: Insecure Design

**Check for:**

- [ ] Rate limiting on sensitive endpoints
- [ ] Account lockout after failed attempts
- [ ] Proper session management
- [ ] Defense in depth

## A05: Security Misconfiguration

**Check for:**

- [ ] No debug mode in production
- [ ] Security headers configured
- [ ] Default credentials changed
- [ ] Unnecessary features disabled

## A06: Vulnerable Components

**Check for:**

- [ ] Dependencies up to date
- [ ] No known vulnerabilities (`npm audit`)
- [ ] Minimal dependency footprint

## A07: Authentication Failures

**Check for:**

- [ ] Strong password requirements
- [ ] MFA where appropriate
- [ ] Secure session tokens
- [ ] Proper logout (invalidate sessions)

## A08: Data Integrity Failures

**Check for:**

- [ ] Signed/verified updates
- [ ] Integrity checks on critical data
- [ ] No unsafe deserialization

## A09: Logging & Monitoring

**Check for:**

- [ ] Security events logged
- [ ] No sensitive data in logs
- [ ] Alerting on suspicious activity

```typescript
// Bad - Logging sensitive data
logger.info(`Login attempt: ${email} / ${password}`);

// Good - No sensitive data
logger.info(`Login attempt for: ${email}`);
```

## A10: Server-Side Request Forgery (SSRF)

**Check for:**

- [ ] URL validation before fetching
- [ ] Allowlist for external requests
- [ ] No user-controlled URLs to internal services

```typescript
// Bad - User controls URL
const data = await fetch(req.body.url);

// Good - Validate against allowlist
const ALLOWED_HOSTS = ['api.trusted.com'];
const url = new URL(req.body.url);
if (!ALLOWED_HOSTS.includes(url.hostname)) {
  throw new Error('Invalid URL');
}
```

## Quick Security Scan

```bash
# NPM audit
npm audit

# Check for secrets in code
git secrets --scan

# SAST scanning (if available)
semgrep --config auto .
```
