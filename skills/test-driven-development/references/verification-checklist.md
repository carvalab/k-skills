# Test Verification Checklist

How to verify your tests actually test what they should.

## Pre-Commit Verification

Before committing, run through this checklist for each test:

### 1. Does the Test Actually Fail?

**The most important verification**: temporarily break the code and confirm the test fails.

```bash
# 1. Make a small change to break the behavior
# 2. Run the test
# 3. Verify it fails for the RIGHT reason
# 4. Revert the change
```

If the test still passes when behavior is broken, it's a Liar test.

### 2. Test Name Describes Behavior?

| Bad | Good |
|-----|------|
| `testUser` | `should_return_user_when_id_exists` |
| `test1` | `should_throw_when_email_invalid` |
| `testCreate` | `should_persist_user_with_hashed_password` |

**Format**: `should_[expected]_when_[condition]`

### 3. Arrange-Act-Assert Structure Clear?

```go
func TestCalculateDiscount(t *testing.T) {
    // ARRANGE - Set up test data
    cart := NewCart()
    cart.AddItem(Product{Price: 100})

    // ACT - Execute the behavior
    discount := cart.CalculateDiscount()

    // ASSERT - Verify the result
    assert.Equal(t, 10.0, discount)
}
```

Each section should be visually distinct. If they're mixed, refactor.

### 4. No Test-Only Code in Production?

Check for these anti-patterns:

```go
// BAD - Test-only method in production code
type UserService struct {
    // ...
}

func (s *UserService) TestOnlyResetState() { // Don't do this
    s.cache = nil
}

// BAD - Test flag in production code
func (s *UserService) CreateUser(email string, testMode bool) { // Don't do this
    if testMode {
        return mockUser
    }
    // ...
}
```

**Fix**: Use dependency injection and test doubles instead.

### 5. Mocks Verify Behavior, Not Implementation?

```typescript
// BAD - Testing implementation details
test('createUser', () => {
    const mockRepo = { save: jest.fn() };
    service.createUser(userData);

    expect(mockRepo.save).toHaveBeenCalledTimes(1); // Who cares?
    expect(mockRepo.save).toHaveBeenCalledWith(expect.objectContaining({
        _internal_flag: true // Testing internal detail
    }));
});

// GOOD - Testing observable behavior
test('createUser persists user and returns with ID', async () => {
    const user = await service.createUser(userData);

    expect(user.id).toBeDefined();
    expect(await service.getUser(user.id)).toEqual(user);
});
```

### 6. Edge Cases Covered?

| Category | Examples |
|----------|----------|
| Empty inputs | `""`, `[]`, `{}`, `null`, `undefined` |
| Boundary values | 0, -1, MAX_INT, empty string |
| Invalid inputs | wrong type, malformed data |
| Error conditions | network failure, timeout, not found |
| Concurrent access | race conditions, deadlocks |

### 7. Test Is Deterministic?

Tests must produce same result every time. Watch for:

```python
# BAD - Non-deterministic
def test_random_id():
    user = create_user()
    assert user.id  # Passes sometimes, fails sometimes

# BAD - Time-dependent
def test_expiry():
    token = create_token()
    time.sleep(1)  # Flaky in slow CI
    assert token.is_expired()

# GOOD - Deterministic
def test_expiry():
    token = create_token(created_at=datetime(2024, 1, 1))
    clock = FakeClock(datetime(2024, 1, 2))

    assert token.is_expired(clock)
```

### 8. Test Is Isolated?

```java
// BAD - Tests share state
class OrderTest {
    static Database db = new Database(); // Shared!

    @Test void testCreate() { db.insert(...); }
    @Test void testDelete() { db.delete(...); } // Depends on testCreate
}

// GOOD - Each test is independent
class OrderTest {
    @BeforeEach
    void setUp() {
        db = new InMemoryDatabase(); // Fresh for each test
    }
}
```

---

## Quick Verification Commands

### Go
```bash
# Run tests with verbose output
go test -v ./...

# Run specific test
go test -v -run TestUserService_Register ./...

# Check coverage
go test -cover ./...

# Race detection
go test -race ./...
```

### Node/TypeScript
```bash
# Run tests
npm test

# Run with coverage
npm test -- --coverage

# Run specific test
npm test -- --testNamePattern="should create user"

# Watch mode
npm test -- --watch
```

### Python
```bash
# Run tests
pytest

# Verbose output
pytest -v

# Run specific test
pytest -k "test_create_user"

# Coverage
pytest --cov=src

# Show slow tests
pytest --durations=10
```

### Java
```bash
# Run tests
./mvnw test

# Run specific test
./mvnw test -Dtest=UserServiceTest

# Run specific method
./mvnw test -Dtest=UserServiceTest#shouldCreateUser

# With coverage (JaCoCo)
./mvnw test jacoco:report
```

---

## Coverage Guidelines

### Metrics

| Metric | Target | Notes |
|--------|--------|-------|
| Line coverage | 70-80% | Minimum for critical paths |
| Branch coverage | 60-70% | All if/else branches |
| Mutation score | 60%+ | Tests catch introduced bugs |

### What to Cover

**Must cover:**
- Business logic
- Error handling
- Edge cases
- Security-sensitive code

**Can skip:**
- Trivial getters/setters
- Framework boilerplate
- Generated code

### Mutation Testing

Coverage alone doesn't prove test quality. Mutation testing does:

```bash
# Go
go install github.com/zimmski/go-mutesting/cmd/go-mutesting@latest
go-mutesting ./...

# Node/TypeScript
npx stryker run

# Python
pip install mutmut
mutmut run

# Java
./mvnw org.pitest:pitest-maven:mutationCoverage
```

---

## Red Flags in Tests

| Red Flag | Problem | Action |
|----------|---------|--------|
| No assertions | Test proves nothing | Add assertions |
| `expect(true).toBe(true)` | Meaningless | Test actual behavior |
| Commented-out tests | Broken or obsolete | Fix or delete |
| `@Ignore` / `skip()` | Tech debt | Fix or delete |
| Sleeps/delays | Flaky, slow | Use fakes/mocks |
| Complex test logic | Bug in test | Simplify |
| Many mocks | Coupling | Reduce dependencies |
| Shared state | Order-dependent | Isolate |

---

## Final Checklist

```markdown
## Pre-Commit Test Verification

[ ] Test fails when behavior removed
[ ] Test name describes behavior
[ ] Arrange-Act-Assert structure clear
[ ] No test-only code in production
[ ] Mocks verify behavior, not implementation
[ ] Edge cases covered
[ ] Test is deterministic
[ ] Test is isolated
[ ] Coverage adequate for changes
[ ] No red flags present
```
