# Backend Code Quality

SOLID, DRY, YAGNI, KISS principles and clean code patterns.

## Existing Projects Rule

**Before applying patterns, understand existing codebase:**

1. Read the code structure first
2. Follow existing patterns (consistency > "better" patterns)
3. Apply improvements to NEW code you write
4. Don't refactor unless explicitly asked

```typescript
// Project uses callbacks? → Use callbacks
// Project has flat structure? → Keep flat
// Project has no interfaces? → Don't add them everywhere
```

**Apply Clean Architecture/patterns to:**

- New greenfield projects
- New modules (when it fits existing style)
- When explicitly asked to refactor

## SOLID Principles

### Single Responsibility (SRP)

```typescript
// BAD: Multiple responsibilities
class User {
  saveToDatabase() {}
  sendWelcomeEmail() {}
  generateReport() {}
}

// GOOD: Separated
class User {
  /* data only */
}
class UserRepository {
  save(user: User) {}
}
class EmailService {
  sendWelcomeEmail(user: User) {}
}
class ReportGenerator {
  generateUserReport(user: User) {}
}
```

### Open/Closed (OCP)

```typescript
// BAD: Modify to add new payment
class PaymentProcessor {
  process(amount: number, method: string) {
    if (method === 'stripe') {
      /* ... */
    } else if (method === 'paypal') {
      /* ... */
    }
  }
}

// GOOD: Strategy pattern
interface PaymentStrategy {
  process(amount: number): Promise<PaymentResult>;
}

class StripePayment implements PaymentStrategy {
  async process(amount: number) {
    /* Stripe logic */
  }
}

class PaymentProcessor {
  constructor(private strategy: PaymentStrategy) {}
  async process(amount: number) {
    return this.strategy.process(amount);
  }
}
```

### Liskov Substitution (LSP)

```typescript
// BAD: Penguin breaks Bird contract
class Bird {
  fly() {}
}
class Penguin extends Bird {
  fly() {
    throw new Error("Can't fly");
  }
}

// GOOD: Proper abstraction
interface Bird {
  move(): void;
}
class FlyingBird implements Bird {
  move() {
    this.fly();
  }
}
class Penguin implements Bird {
  move() {
    this.swim();
  }
}
```

### Interface Segregation (ISP)

```typescript
// BAD: Robot forced to implement eat/sleep
interface Worker {
  work();
  eat();
  sleep();
}

// GOOD: Segregated interfaces
interface Workable {
  work(): void;
}
interface Eatable {
  eat(): void;
}

class Human implements Workable, Eatable {
  /* ... */
}
class Robot implements Workable {
  /* ... */
}
```

### Dependency Inversion (DIP)

```typescript
// BAD: Tight coupling
class UserService {
  private db = new MySQLDatabase();
}

// GOOD: Depend on abstraction
interface Database {
  query(sql: string, params: any[]): Promise<any>;
}

class UserService {
  constructor(private db: Database) {}
}
```

## Design Patterns

### Repository Pattern

```typescript
interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

class PostgresUserRepository implements UserRepository {
  constructor(private db: Database) {}

  async findById(id: string): Promise<User | null> {
    const row = await this.db.query('SELECT * FROM users WHERE id = $1', [id]);
    return row ? new User(row.id, row.email) : null;
  }
}
```

### Factory Pattern

```typescript
interface Notification {
  send(msg: string): Promise<void>;
}
class NotificationFactory {
  static create(type: 'email' | 'sms' | 'push'): Notification {
    const map = { email: EmailNotification, sms: SMSNotification, push: PushNotification };
    return new map[type]();
  }
}
```

### Observer Pattern

```typescript
interface Observer {
  update(event: any): void;
}
class EventEmitter {
  private observers: Map<string, Observer[]> = new Map();
  subscribe(event: string, obs: Observer) {
    /* add to map */
  }
  emit(event: string, data: any) {
    this.observers.get(event)?.forEach((o) => o.update(data));
  }
}
```

## Clean Code Practices

### Meaningful Names

```typescript
// BAD: d(a, b)  →  GOOD: calculateAreaInMeters(width, height)
```

### Small Functions

```typescript
// BAD: 200-line function  →  GOOD: Composed small functions
async function processOrder(orderId: string) {
  const order = await validateOrder(orderId);
  await processPayment(order);
  await sendConfirmationEmail(order);
}
```

### Avoid Magic Numbers

```typescript
// BAD: if (age < 18) / setTimeout(fn, 86400000)
// GOOD:
const MINIMUM_AGE = 18;
const ONE_DAY_MS = 24 * 60 * 60 * 1000;
```

### Error Handling

```typescript
// BAD: catch (e) { console.log(e); return null; }
// GOOD:
try {
  const user = await db.findUser(id);
  if (!user) throw new UserNotFoundError(id);
  return user;
} catch (error) {
  logger.error('Failed to fetch user', { userId: id, error });
  throw new DatabaseError('User fetch failed', { cause: error });
}
```

### Comments (Only When Necessary)

**Code should be self-documenting.** Comments explain "why", not "what".

```typescript
// ❌ BAD: Obvious / commented-out code
const total = price * quantity; // Calculate total
// const oldLogic = calculateOldWay(x);

// ✅ GOOD: Explains WHY (non-obvious reason)
// Retry with backoff - third-party API has rate limits
await retryWithBackoff(() => externalApi.call());

// ✅ GOOD: Public API doc
/** Creates user and sends welcome email. Throws if email exists. */
async function createUser(email: string): Promise<User> {}
```

**Comment:** Complex logic, workarounds, public APIs. **Don't comment:** Obvious code, every function.

### DRY (Don't Repeat Yourself)

**See `references/dry-detection.md` for comprehensive DRY detection guide.**

```typescript
// BAD: Duplicated validation in multiple places
// GOOD: Extract to reusable function
function validateEmail(email: string) {
  if (!email || !email.includes('@')) {
    throw new ValidationError('Invalid email');
  }
}
```

#### Pre-Implementation DRY Check (MANDATORY GATE)

**You cannot write new code until you complete this analysis.**

Extract keywords from your task, then search:
```bash
# Search by VERBS from your task (what actions you need)
grep -rn "<verb1>.*\|<verb2>.*" --include="*.ts" --include="*.go" . | head -50

# Search by NOUNS from your task (domain concepts)
grep -rn "<noun1>\|<noun2>\|<noun3>" --include="*.ts" --include="*.go" . | head -50

# Search by types/interfaces you need
grep -rn "type.*<TypeName>\|interface.*<Name>" --include="*.ts" --include="*.go" . | head -30

# Semantic search
codebase_search "how does existing code <do what your task needs>"
```

**DRY Decision Matrix:**

| Similarity | Action | What To Do |
| ---------- | ------ | ---------- |
| **>70%** | **REUSE** | Inject existing service, call its method directly |
| **50-70%** | **EXTRACT** | Create shared helper, refactor existing to use it |
| **30-50%** | **CONSIDER** | Maybe create interface both implement |
| **<30%** | **NEW OK** | Document why existing doesn't fit |

#### Common Reuse Patterns

**Pattern 1: Inject and Call (>70% similar)**
```go
// ❌ WRONG: Copy existing function to new location
func (u *newUsecases) existingLogic(...) { /* N lines copied */ }

// ✅ RIGHT: Inject existing service, call its method
type newUsecases struct {
    existingService existing.Service  // inject existing
}

func (u *newUsecases) doSomething(ctx context.Context) {
    result, _ := u.existingService.ExistingMethod(ctx, input)  // reuse!
}
```

**Pattern 2: Extract Shared Helper (50-70% similar)**
```go
// ❌ WRONG: Two functions with same calculation logic
func ProcessTypeA(a *TypeA) *Result { /* 100 lines */ }
func ProcessTypeB(b *TypeB) *Result { /* same 100 lines */ }

// ✅ RIGHT: Extract shared, thin adapters
func sharedCalculation(data *CommonData) *Result { /* ONE place */ }

func ProcessTypeA(a *TypeA) *Result {
    return sharedCalculation(a.ToCommonData())  // 5 lines
}
```

**Pattern 3: Interface Adapter (multiple sources, same processing)**
```go
// ❌ WRONG: Separate functions for each type
func ProcessA(a *TypeA) { /* 50 lines */ }
func ProcessB(b *TypeB) { /* same 50 lines */ }

// ✅ RIGHT: Interface + single implementation
type Processable interface {
    GetID() string
    GetData() []Data
}

func Process(p Processable) { /* 50 lines, ONCE */ }
```

#### Self-Check Before Committing

- [ ] Did I complete the Code Reuse Analysis?
- [ ] Is my new file <100 lines? If larger, did I check for duplication?
- [ ] Would a business logic change need edits in 2+ places? If yes, extract.
- [ ] Did I inject existing services instead of recreating their logic?

### YAGNI (You Aren't Gonna Need It)

```typescript
// BAD: Building features "just in case"
class UserService {
  createUser() {}
  createUserWithRole() {} // Not needed yet
  createUserBatch() {} // Not needed yet
  createUserFromCSV() {} // Not needed yet
  createUserWithInvitation() {} // Not needed yet
}

// GOOD: Only what's needed NOW
class UserService {
  createUser() {} // Add others when actually needed
}
```

**Rule:** Don't build it until you need it. Unused code is maintenance burden.

### KISS (Keep It Simple, Stupid)

```typescript
// BAD: Over-engineered for simple task
class UserNameFormatter {
  private strategy: FormattingStrategy;
  private cache: Map<string, string>;
  constructor(strategyFactory: StrategyFactory) {
    this.strategy = strategyFactory.create('name');
    this.cache = new Map();
  }
  format(user: User): string {
    /* 50 lines */
  }
}

// GOOD: Simple solution
function formatUserName(user: User): string {
  return `${user.firstName} ${user.lastName}`.trim();
}
```

**Rule:** Choose the simplest solution that works. Complexity has cost.

## Refactoring Techniques

- **Extract Method** - Break large functions into smaller named functions
- **Replace Conditional with Polymorphism** - Use interfaces instead of if/switch
- **Introduce Parameter Object** - Group related params into object
- **Replace Magic Number** - Use named constants

## Code Quality Checklist

- [ ] **Existing patterns respected** (if existing project)
- [ ] SOLID principles applied (where appropriate)
- [ ] DRY - No duplication
- [ ] YAGNI - No unused features built
- [ ] KISS - Simplest solution chosen
- [ ] Functions < 20 lines
- [ ] Meaningful names (self-documenting)
- [ ] No magic numbers
- [ ] Proper error handling
- [ ] **Comments only for "why"** (not obvious "what")
- [ ] No commented-out code
- [ ] Readable > clever
