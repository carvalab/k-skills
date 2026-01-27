# Backend Architecture Patterns

Clean Architecture, microservices, event-driven patterns.

## Clean Architecture (4-Layer)

```
┌─────────────────────────────────────────────────────────────┐
│                    Interface Layer                          │
│            (Controllers, HTTP handlers, CLI)                │
├─────────────────────────────────────────────────────────────┤
│                   Application Layer                         │
│              (Use Cases, DTOs, Mappers)                     │
├─────────────────────────────────────────────────────────────┤
│                 Infrastructure Layer                        │
│        (Repository Impls, External APIs, DB)                │
├─────────────────────────────────────────────────────────────┤
│                     Domain Layer                            │
│         (Entities, Repository Interfaces)                   │
└─────────────────────────────────────────────────────────────┘
```

**Dependency Rule:** Inner layers never depend on outer layers.

### Project Structure

```
src/
├── domain/                    # Business logic (no dependencies)
│   ├── entities/
│   │   └── user.entity.ts
│   └── repositories/
│       └── user.repository.interface.ts
├── application/               # Use cases orchestrate domain + infra
│   ├── use-cases/
│   │   └── get-user.use-case.ts
│   ├── dtos/
│   │   └── user.dto.ts
│   └── mappers/
│       └── user.mapper.ts
├── infrastructure/            # External integrations
│   ├── repositories/
│   │   └── user.repository.impl.ts
│   └── external-api/
│       └── api-client.ts
├── interface/                 # HTTP/Protocol layer
│   ├── controllers/
│   │   └── user.controller.ts
│   └── server.ts
└── platform/                  # Cross-cutting (config, logging)
    ├── config/
    └── utils/
```

### Key Patterns

**Repository Pattern:**

```typescript
// domain/repositories/user.repository.interface.ts
export interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<User>;
}

// infrastructure/repositories/user.repository.impl.ts
export class UserRepositoryImpl implements UserRepository {
  constructor(private db: Database) {}

  async findById(id: string): Promise<User | null> {
    const row = await this.db.query('SELECT * FROM users WHERE id = $1', [id]);
    return row ? this.toEntity(row) : null;
  }
}
```

**Use Case Pattern:**

```typescript
// application/use-cases/get-user.use-case.ts
export class GetUserUseCase {
  constructor(
    private repository: UserRepository,
    private mapper: UserMapper
  ) {}

  async execute(id: string): Promise<UserResponseDTO> {
    const user = await this.repository.findById(id);
    if (!user) throw new NotFoundError('User');
    return this.mapper.toResponseDTO(user);
  }
}
```

**Mapper Pattern:**

```typescript
// application/mappers/user.mapper.ts
export class UserMapper {
  toResponseDTO(entity: User): UserResponseDTO {
    return {
      id: entity.id,
      email: entity.email,
      createdAt: entity.createdAt.toISOString(),
    };
  }
}
```

**Manual DI (Factory):**

```typescript
// interface/controllers/user.controller.ts
export class UserController {
  private getUserUseCase: GetUserUseCase;

  constructor() {
    this.setupDependencies();
  }

  private setupDependencies(): void {
    const db = new Database(config.db);
    const repository = new UserRepositoryImpl(db);
    const mapper = new UserMapper();
    this.getUserUseCase = new GetUserUseCase(repository, mapper);
  }
}
```

## Microservices Architecture

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  User    │   │ Product  │   │  Order   │   │ Payment  │
│ Service  │   │ Service  │   │ Service  │   │ Service  │
└────┬─────┘   └────┬─────┘   └────┬─────┘   └────┬─────┘
     │              │              │              │
  ┌──▼──┐        ┌──▼──┐        ┌──▼──┐        ┌──▼──┐
  │  DB │        │  DB │        │  DB │        │  DB │
  └─────┘        └─────┘        └─────┘        └─────┘
```

**Pros:** Independent deployment, fault isolation, scaling per service
**Cons:** Distributed complexity, eventual consistency, operational overhead

## Key Patterns

### Database per Service

Each service owns its database (PostgreSQL instance or schema).

**Benefits:** Service independence, technology choice, fault isolation
**Challenges:** No joins across services, distributed transactions

### API Gateway

```
Client → API Gateway → User Service
              ↓
         Product Service
              ↓
         Order Service
```

**Responsibilities:**

- Request routing
- Authentication/authorization
- Rate limiting
- Request transformation

### Circuit Breaker

```typescript
import CircuitBreaker from 'opossum';

const breaker = new CircuitBreaker(callExternalService, {
  timeout: 3000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000,
});

breaker.fallback(() => ({ data: 'cached-response' }));
const result = await breaker.fire(requestParams);
```

**States:** Closed → Open (on failures) → Half-Open (testing)

### Saga Pattern (Distributed Transactions)

**Choreography:**

```
Order → "OrderCreated" → Payment → "PaymentReserved" → Inventory → "StockReserved"
```

**Orchestration:**

```
Saga Orchestrator → Order → Payment → Inventory → Shipping
```

If any step fails → Compensating transactions (rollback)

## Event-Driven Architecture

### Event Sourcing

Store events, not current state:

```typescript
// Events stored
[
  { type: 'AccountCreated', userId: '123' },
  { type: 'MoneyDeposited', amount: 1000 },
  { type: 'MoneyWithdrawn', amount: 500 },
];

// Reconstruct state by replaying
const balance = events.reduce((acc, event) => {
  if (event.type === 'MoneyDeposited') return acc + event.amount;
  if (event.type === 'MoneyWithdrawn') return acc - event.amount;
  return acc;
}, 0);
```

### Message Queues

**Kavak:** Use kbroker for events between services, River Queue (Go) or pg-boss (Node) for jobs.
See `references/kavak/kbroker.md` and `references/kavak/queues-ratelimit.md`.

## CQRS

Separate read and write models:

```
Write Side:               Read Side:
CreateOrder               GetOrderById
  ↓                          ↑
PostgreSQL → Events → Redis/Optimized DB
```

## Scalability Patterns

### Horizontal Scaling

```
Load Balancer
    ↓
┌───┴───┬───────┬───────┐
│ App 1 │ App 2 │ App 3 │
└───────┴───────┴───────┘
         ↓
    PostgreSQL (+ replicas)
```

### Database Read Replicas

```
Primary (Write) → Replica 1 (Read)
               → Replica 2 (Read)
```

### Caching Layers

```
Client → CDN → API Gateway Cache → Redis → PostgreSQL
```

## Architecture Checklist

- [ ] Clear service boundaries
- [ ] Database per service
- [ ] API Gateway configured
- [ ] Circuit breakers for resilience
- [ ] Event-driven communication
- [ ] Distributed tracing (OpenTelemetry)
- [ ] Health checks for all services
- [ ] Horizontal scaling capability

## Anti-Patterns

1. **Distributed Monolith** - Services all depend on each other
2. **Chatty Services** - Too many inter-service calls
3. **Shared Database** - Tight coupling
4. **No Circuit Breakers** - Cascade failures
