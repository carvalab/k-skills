# Backend Testing Strategies

Testing approaches, frameworks, and quality practices (2026).

## Test Pyramid (70-20-10)

```
        /\
       /E2E\     10% - End-to-End
      /------\
     /Integr.\ 20% - Integration
    /----------\
   /   Unit     \ 70% - Unit Tests
  /--------------\
```

## Unit Testing

### Node.js (Vitest - 50% faster than Jest)

```typescript
import { describe, it, expect, beforeEach } from 'vitest';

describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', async () => {
      const userData = { email: 'test@example.com', name: 'Test' };
      const user = await userService.createUser(userData);

      expect(user).toMatchObject(userData);
      expect(user.id).toBeDefined();
    });

    it('should throw error with duplicate email', async () => {
      const userData = { email: 'existing@example.com', name: 'Test' };
      await expect(userService.createUser(userData))
        .rejects.toThrow('Email already exists');
    });
  });
});
```

### Go (testing + testify)

```go
func TestUserService_CreateUser(t *testing.T) {
    tests := []struct {
        name    string
        input   CreateUserInput
        wantErr bool
    }{
        {"valid user", CreateUserInput{Email: "test@example.com"}, false},
        {"invalid email", CreateUserInput{Email: "invalid"}, true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            user, err := svc.CreateUser(tt.input)
            if tt.wantErr {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
                assert.NotEmpty(t, user.ID)
            }
        })
    }
}
```

## Integration Testing

### API Tests

```typescript
import request from 'supertest';

describe('POST /api/users', () => {
  beforeEach(async () => {
    await db.users.deleteMany({});
  });

  it('should create user and return 201', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', name: 'Test User' })
      .expect(201);

    expect(response.body).toMatchObject({
      email: 'test@example.com',
      name: 'Test User',
    });

    const user = await db.users.findOne({ email: 'test@example.com' });
    expect(user).toBeDefined();
  });
});
```

### TestContainers (Real PostgreSQL)

```typescript
import { GenericContainer } from 'testcontainers';

let container;

beforeAll(async () => {
  container = await new GenericContainer('postgres:15')
    .withEnvironment({ POSTGRES_PASSWORD: 'test' })
    .withExposedPorts(5432)
    .start();

  const port = container.getMappedPort(5432);
  db = await createConnection({ host: 'localhost', port, password: 'test' });
}, 60000);

afterAll(async () => {
  await container.stop();
});
```

## Contract Testing (Microservices)

### Pact

```typescript
import { Pact } from '@pact-foundation/pact';

const provider = new Pact({
  consumer: 'UserService',
  provider: 'AuthService',
});

it('should validate user token', async () => {
  await provider.addInteraction({
    state: 'user token exists',
    uponReceiving: 'a request to validate token',
    withRequest: {
      method: 'POST',
      path: '/auth/validate',
      body: { token: 'valid-token-123' },
    },
    willRespondWith: {
      status: 200,
      body: { valid: true, userId: '123' },
    },
  });

  const response = await authClient.validateToken('valid-token-123');
  expect(response.valid).toBe(true);
});
```

## Load Testing (k6)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '5m', target: 100 },
    { duration: '2m', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
  },
};

export default function () {
  const res = http.get('https://api.example.com/users');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  sleep(1);
}
```

**Thresholds:**
- Response time: p95 < 500ms, p99 < 1s
- Throughput: 1000+ req/sec
- Error rate: < 1%

## E2E Testing (Playwright)

```typescript
import { test, expect } from '@playwright/test';

test('user can register and login', async ({ page }) => {
  await page.goto('https://app.example.com/register');
  await page.fill('input[name="email"]', 'test@example.com');
  await page.fill('input[name="password"]', 'SecurePass123!');
  await page.click('button[type="submit"]');

  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('h1')).toContainText('Welcome');
});
```

## Database Migration Testing

```typescript
it('should migrate without data loss', async () => {
  await db.query(`INSERT INTO users (id, email) VALUES (1, 'test@example.com')`);

  await runMigration('v2-add-created-at.sql');

  const result = await db.query('SELECT * FROM users WHERE id = 1');
  expect(result.rows[0]).toMatchObject({
    id: 1,
    email: 'test@example.com',
    created_at: expect.any(Date),
  });
});
```

## Security Testing

```bash
# SAST
semgrep --config auto src/

# Dependency scanning
npm audit
snyk test

# DAST
docker run owasp/zap2docker-stable zap-baseline.py -t https://api.example.com
```

## Coverage Targets

- Overall: 80%+
- Critical paths: 100%
- New code: 90%+

```bash
vitest run --coverage
jest --coverage --coverageThreshold='{"global":{"lines":80}}'
```

## Testing Checklist

- [ ] Unit tests (70%)
- [ ] Integration tests (20%)
- [ ] E2E tests (10%)
- [ ] Contract tests (microservices)
- [ ] Load tests
- [ ] Migration tests
- [ ] Security scanning
- [ ] Coverage reports
- [ ] Tests in CI/CD
