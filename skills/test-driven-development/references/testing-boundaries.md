# Testing Boundaries

What to mock, what not to mock, and where the boundaries are.

## The Boundary Principle

**Mock at system boundaries, not internal collaborators.**

```
┌─────────────────────────────────────────────────────┐
│                    Your System                       │
│                                                      │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐      │
│  │ Service  │───▶│ Business │───▶│ Repository│      │
│  │          │    │  Logic   │    │          │      │
│  └──────────┘    └──────────┘    └──────────┘      │
│       │                              │              │
└───────┼──────────────────────────────┼──────────────┘
        │                              │
        ▼                              ▼
   ┌─────────┐                   ┌─────────┐
   │ HTTP    │  ◀── MOCK HERE    │ Database│  ◀── MOCK HERE
   │ Client  │                   │         │
   └─────────┘                   └─────────┘
```

## What to Mock

### 1. External HTTP Services

```go
// Define interface for external API calls
type HTTPClient interface {
    Get(url string) (*http.Response, error)
}

// Generate mock with mockery:
// mockery --name=HTTPClient --with-expecter --output=mocks

// In tests - use generated mock
func TestFetchData(t *testing.T) {
    mockClient := mocks.NewMockHTTPClient(t)
    mockClient.EXPECT().
        Get("https://api.example.com/data").
        Return(&http.Response{StatusCode: 200, Body: ...}, nil)

    svc := NewDataService(mockClient)
    data, err := svc.Fetch()

    assert.NoError(t, err)
}
```

### 2. Databases

```typescript
// Use in-memory implementation
interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

// In tests
class InMemoryUserRepository implements UserRepository {
  private users: Map<string, User> = new Map();

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) ?? null;
  }

  async save(user: User): Promise<void> {
    this.users.set(user.id, user);
  }
}
```

### 3. File System

```python
# Inject file system operations
class FileStorage:
    def read(self, path: str) -> bytes:
        raise NotImplementedError

class LocalFileStorage(FileStorage):
    def read(self, path: str) -> bytes:
        with open(path, 'rb') as f:
            return f.read()

# In tests
class InMemoryFileStorage(FileStorage):
    def __init__(self):
        self.files: dict[str, bytes] = {}

    def read(self, path: str) -> bytes:
        return self.files.get(path, b'')
```

### 4. Time/Clock

```java
// Inject clock
public class TokenService {
    private final Clock clock;

    public TokenService(Clock clock) {
        this.clock = clock;
    }

    public boolean isExpired(Token token) {
        return clock.instant().isAfter(token.expiresAt());
    }
}

// In tests
Clock fixedClock = Clock.fixed(
    Instant.parse("2024-01-01T00:00:00Z"),
    ZoneOffset.UTC
);
TokenService service = new TokenService(fixedClock);
```

### 5. Random/UUID Generation

```go
// Inject ID generator
type IDGenerator interface {
    Generate() string
}

type UUIDGenerator struct{}
func (g *UUIDGenerator) Generate() string {
    return uuid.New().String()
}

// In tests
type SequentialIDGenerator struct {
    counter int
}
func (g *SequentialIDGenerator) Generate() string {
    g.counter++
    return fmt.Sprintf("id-%d", g.counter)
}
```

## What NOT to Mock

### 1. Internal Collaborators

```typescript
// BAD - Mocking internal class
class OrderService {
  constructor(
    private validator: OrderValidator, // DON'T MOCK
    private calculator: PriceCalculator // DON'T MOCK
  ) {}
}

// GOOD - Test the whole unit together
test('processOrder validates and calculates correctly', () => {
  const service = new OrderService(
    new OrderValidator(),
    new PriceCalculator()
  );

  const result = service.processOrder(orderData);

  expect(result.total).toBe(expectedTotal);
});
```

### 2. Value Objects

```python
# DON'T mock value objects
class Money:
    def __init__(self, amount: Decimal, currency: str):
        self.amount = amount
        self.currency = currency

    def add(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Currency mismatch")
        return Money(self.amount + other.amount, self.currency)

# Use real Money in tests, not mocks
def test_cart_total():
    cart = Cart()
    cart.add_item(Money(Decimal("10.00"), "USD"))
    cart.add_item(Money(Decimal("20.00"), "USD"))

    assert cart.total() == Money(Decimal("30.00"), "USD")
```

### 3. Data Transfer Objects

```go
// DTOs are just data - no behavior to mock
type UserDTO struct {
    ID    string `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

// Just create real instances
func TestMapUserToDTO(t *testing.T) {
    user := &User{ID: "1", Name: "John", Email: "john@example.com"}

    dto := MapToDTO(user)

    assert.Equal(t, "1", dto.ID)
    assert.Equal(t, "John", dto.Name)
}
```

### 4. Language/Framework Primitives

```java
// DON'T mock
// - String, Integer, List, Map
// - Standard library classes
// - Framework utilities

// GOOD - Use real collections
@Test
void shouldFilterActiveUsers() {
    List<User> users = List.of(
        new User("1", true),
        new User("2", false),
        new User("3", true)
    );

    List<User> active = service.filterActive(users);

    assertThat(active).hasSize(2);
}
```

## Mock Boundaries by Layer

### Web Layer (Controllers/Handlers)

```
Mock: Service layer (sometimes)
Real: Request/response parsing, validation
```

```typescript
// Controller test - may mock service
test('POST /users returns 201', async () => {
  const mockService = { createUser: jest.fn().mockResolvedValue(user) };
  const controller = new UserController(mockService);

  const response = await controller.create(request);

  expect(response.status).toBe(201);
});
```

### Service Layer (Business Logic)

```
Mock: External services, repositories (at boundary)
Real: Business logic, internal helpers
```

```go
// Service test - mock repository
func TestOrderService_CreateOrder(t *testing.T) {
    repo := NewInMemoryOrderRepository() // In-memory fake
    service := NewOrderService(repo)

    order, err := service.Create(orderData)

    assert.NoError(t, err)
    assert.NotEmpty(t, order.ID)
}
```

### Repository Layer (Data Access)

```
Mock: Database connection (use in-memory)
Real: Query logic, mapping
```

```python
# Repository test with in-memory DB
def test_find_by_email():
    repo = UserRepository(InMemorySQLite())
    repo.save(User(email="test@example.com"))

    user = repo.find_by_email("test@example.com")

    assert user is not None
    assert user.email == "test@example.com"
```

## Integration Test Boundaries

For integration tests, extend the boundary:

```
Unit Test:
  Mock: HTTP, DB, File system, External APIs

Integration Test:
  Mock: External APIs only
  Real: HTTP (test server), DB (test container), File system (temp)
```

```java
// Integration test with real DB
@SpringBootTest
@Testcontainers
class UserServiceIntegrationTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @Test
    void shouldPersistAndRetrieveUser() {
        // Real database operations
        User saved = service.createUser(userData);
        User found = service.findById(saved.getId());

        assertThat(found).isEqualTo(saved);
    }
}
```

## Decision Guide

```
Should I mock this?

1. Is it a system boundary (HTTP, DB, file, external API)?
   → YES: Mock it (or use a fake)

2. Is it an internal collaborator (helper, utility, value object)?
   → NO: Use the real implementation

3. Is it non-deterministic (time, random, UUID)?
   → YES: Inject and mock

4. Is it slow (network, disk)?
   → YES: Mock in unit tests, real in integration tests

5. Is it third-party but pure logic (lodash, moment)?
   → NO: Use real implementation
```

## Summary Table

| Type | Mock? | Why |
|------|-------|-----|
| External HTTP APIs | Yes | Slow, unreliable, costs money |
| Database | Yes (in-memory fake) | Slow, needs setup |
| File system | Yes | Environment-dependent |
| Time/Clock | Yes | Non-deterministic |
| Random/UUID | Yes | Non-deterministic |
| Internal services | No | Test real behavior |
| Value objects | No | No behavior to mock |
| DTOs | No | Just data |
| Standard library | No | Already tested |
