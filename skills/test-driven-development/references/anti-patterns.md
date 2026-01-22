# TDD Anti-Patterns

Comprehensive catalog of testing anti-patterns and how to fix them.

## The Liar

**Problem**: Test passes but doesn't verify actual behavior.

```go
// BAD - The Liar
func TestUserService_CreateUser(t *testing.T) {
    svc := NewUserService()
    svc.CreateUser("john@example.com")
    // No assertion! Test always passes
}

// GOOD - Actually verify behavior
func TestUserService_CreateUser(t *testing.T) {
    svc := NewUserService()
    user, err := svc.CreateUser("john@example.com")

    assert.NoError(t, err)
    assert.Equal(t, "john@example.com", user.Email)
    assert.NotEmpty(t, user.ID)
}
```

**Fix**: Every test must have meaningful assertions.

---

## The Mockery

**Problem**: Over-mocking hides real bugs and tests implementation details.

```typescript
// BAD - Testing mock behavior, not real behavior
test('should call repository save', () => {
  const mockRepo = { save: jest.fn() };
  const service = new UserService(mockRepo);

  service.createUser({ email: 'test@test.com' });

  expect(mockRepo.save).toHaveBeenCalled(); // Tests implementation, not behavior
});

// GOOD - Test actual behavior
test('should persist user and return with ID', async () => {
  const repo = new InMemoryUserRepository();
  const service = new UserService(repo);

  const user = await service.createUser({ email: 'test@test.com' });

  expect(user.id).toBeDefined();
  expect(await repo.findById(user.id)).toEqual(user);
});
```

**Fix**: Mock at boundaries (HTTP, DB, external services), not internal collaborators.

---

## Excessive Setup

**Problem**: 50 lines of setup for 2 lines of test.

```python
# BAD - Excessive setup
def test_order_total():
    # 30 lines of setup
    customer = Customer(id=1, name="John", email="john@test.com")
    customer.address = Address(street="123 Main", city="NYC", zip="10001")
    product1 = Product(id=1, name="Widget", price=10.00, category="Tools")
    product2 = Product(id=2, name="Gadget", price=20.00, category="Electronics")
    inventory = Inventory()
    inventory.add(product1, 100)
    inventory.add(product2, 50)
    cart = ShoppingCart(customer)
    cart.add(product1, 2)
    cart.add(product2, 1)
    order = Order(cart, inventory)
    # ... 20 more lines

    total = order.calculate_total()

    assert total == 40.00

# GOOD - Use builders/fixtures
def test_order_total():
    order = OrderBuilder().with_items([
        ("Widget", 10.00, 2),
        ("Gadget", 20.00, 1)
    ]).build()

    assert order.calculate_total() == 40.00
```

**Fix**: Use test builders, fixtures, or factory functions. If setup is complex, your SUT may have too many dependencies.

---

## The Slow Poke

**Problem**: Tests take so long developers avoid running them.

```java
// BAD - Real database, real HTTP calls
@Test
void shouldProcessOrder() {
    // Takes 5 seconds because it hits real DB and external API
    Order order = new Order(...);
    orderService.process(order); // Real DB write
    paymentGateway.charge(order); // Real HTTP call
}

// GOOD - Fast, isolated test
@Test
void shouldProcessOrder() {
    // Takes 5ms
    var orderRepo = new InMemoryOrderRepository();
    var paymentGateway = new FakePaymentGateway();
    var service = new OrderService(orderRepo, paymentGateway);

    service.process(testOrder);

    assertThat(orderRepo.findById(testOrder.id())).isPresent();
    assertThat(paymentGateway.charges()).hasSize(1);
}
```

**Fix**: Mock I/O boundaries. Use in-memory implementations. Run integration tests separately.

---

## The Local Hero

**Problem**: Test passes on developer's machine, fails elsewhere.

```go
// BAD - Environment dependent
func TestConfig_LoadsFromFile(t *testing.T) {
    config := LoadConfig("/Users/john/project/config.json") // Hardcoded path
    assert.NotNil(t, config)
}

// GOOD - Environment independent
func TestConfig_LoadsFromFile(t *testing.T) {
    // Use temp file or embedded test data
    tmpFile := createTempConfig(t, `{"key": "value"}`)
    defer os.Remove(tmpFile)

    config := LoadConfig(tmpFile)

    assert.Equal(t, "value", config.Key)
}
```

**Fix**: No hardcoded paths. No reliance on local environment. Use temp files or embedded test data.

---

## Test-per-Method

**Problem**: One-to-one mapping between tests and production methods.

```typescript
// BAD - Testing methods, not behaviors
describe('ShoppingCart', () => {
  it('addItem', () => { ... });
  it('removeItem', () => { ... });
  it('calculateTotal', () => { ... });
  it('applyDiscount', () => { ... });
});

// GOOD - Testing behaviors
describe('ShoppingCart', () => {
  it('should increase total when item added', () => { ... });
  it('should decrease total when item removed', () => { ... });
  it('should apply percentage discount to total', () => { ... });
  it('should not allow negative totals', () => { ... });
});
```

**Fix**: Test behaviors and scenarios, not methods. One method may need multiple tests. Multiple methods may be tested in one scenario.

---

## The Inspector

**Problem**: Test inspects internal state instead of observable behavior.

```python
# BAD - Inspecting private state
def test_user_is_active():
    user = User()
    user.activate()

    assert user._active == True  # Accessing private state
    assert user._activation_date is not None

# GOOD - Testing observable behavior
def test_user_is_active():
    user = User()
    user.activate()

    assert user.is_active()  # Public behavior
    assert user.can_login()
```

**Fix**: Test through public interface only. If you need to inspect internals, your design may need improvement.

---

## The Generous Leftovers

**Problem**: Tests share state, order matters.

```java
// BAD - Shared state between tests
class UserServiceTest {
    static List<User> users = new ArrayList<>(); // Shared!

    @Test
    void shouldAddUser() {
        users.add(new User("john"));
        assertEquals(1, users.size()); // Fails if shouldRemoveUser runs first
    }

    @Test
    void shouldRemoveUser() {
        users.clear(); // Affects other tests
    }
}

// GOOD - Isolated tests
class UserServiceTest {
    private List<User> users;

    @BeforeEach
    void setUp() {
        users = new ArrayList<>(); // Fresh for each test
    }

    @Test
    void shouldAddUser() {
        users.add(new User("john"));
        assertEquals(1, users.size());
    }
}
```

**Fix**: Each test creates its own state. Use setup/teardown properly. Tests must run in any order.

---

## The Secret Catcher

**Problem**: Test swallows exceptions, hiding failures.

```go
// BAD - Exception swallowed
func TestProcess_HandlesError(t *testing.T) {
    defer func() {
        recover() // Silently swallows any panic
    }()

    Process(invalidInput)
    // If no panic, test passes even though it should fail
}

// GOOD - Explicitly test for expected errors
func TestProcess_ReturnsErrorForInvalidInput(t *testing.T) {
    err := Process(invalidInput)

    assert.Error(t, err)
    assert.Contains(t, err.Error(), "invalid input")
}
```

**Fix**: Be explicit about expected errors. Don't catch and ignore exceptions.

---

## The Giant

**Problem**: Single test that tests everything.

```typescript
// BAD - Testing everything in one test
test('user workflow', () => {
  // Create user
  const user = createUser({ email: 'test@test.com' });
  expect(user.id).toBeDefined();

  // Update user
  user.name = 'John';
  saveUser(user);
  expect(getUser(user.id).name).toBe('John');

  // Delete user
  deleteUser(user.id);
  expect(getUser(user.id)).toBeNull();

  // ... 50 more assertions
});

// GOOD - Focused tests
test('createUser returns user with ID', () => { ... });
test('saveUser persists name changes', () => { ... });
test('deleteUser removes user', () => { ... });
```

**Fix**: One concept per test. If test name has "and", split it.

---

## Anti-Pattern Checklist

| Anti-Pattern | Detection | Fix |
|--------------|-----------|-----|
| The Liar | No assertions | Add meaningful asserts |
| The Mockery | >3 mocks per test | Mock boundaries only |
| Excessive Setup | >10 lines setup | Use builders/fixtures |
| The Slow Poke | >100ms per test | Mock I/O |
| The Local Hero | Fails in CI | No env deps |
| Test-per-Method | Test names = methods | Test behaviors |
| The Inspector | Access private state | Test public interface |
| Generous Leftovers | Order-dependent | Isolate state |
| The Secret Catcher | Empty catch blocks | Assert errors |
| The Giant | >10 assertions | Split test |
