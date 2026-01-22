# Backend Code Quality

SOLID principles, design patterns, and clean code (2026).

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
class User { /* data only */ }
class UserRepository { save(user: User) {} }
class EmailService { sendWelcomeEmail(user: User) {} }
class ReportGenerator { generateUserReport(user: User) {} }
```

### Open/Closed (OCP)

```typescript
// BAD: Modify to add new payment
class PaymentProcessor {
  process(amount: number, method: string) {
    if (method === 'stripe') { /* ... */ }
    else if (method === 'paypal') { /* ... */ }
  }
}

// GOOD: Strategy pattern
interface PaymentStrategy {
  process(amount: number): Promise<PaymentResult>;
}

class StripePayment implements PaymentStrategy {
  async process(amount: number) { /* Stripe logic */ }
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
class Bird { fly() {} }
class Penguin extends Bird {
  fly() { throw new Error("Can't fly"); }
}

// GOOD: Proper abstraction
interface Bird { move(): void; }
class FlyingBird implements Bird { move() { this.fly(); } }
class Penguin implements Bird { move() { this.swim(); } }
```

### Interface Segregation (ISP)

```typescript
// BAD: Robot forced to implement eat/sleep
interface Worker { work(); eat(); sleep(); }

// GOOD: Segregated interfaces
interface Workable { work(): void; }
interface Eatable { eat(): void; }

class Human implements Workable, Eatable { /* ... */ }
class Robot implements Workable { /* ... */ }
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
interface Notification { send(message: string): Promise<void>; }

class NotificationFactory {
  static create(type: 'email' | 'sms' | 'push'): Notification {
    switch (type) {
      case 'email': return new EmailNotification();
      case 'sms': return new SMSNotification();
      case 'push': return new PushNotification();
    }
  }
}
```

### Observer Pattern

```typescript
interface Observer { update(event: any): void; }

class EventEmitter {
  private observers: Map<string, Observer[]> = new Map();

  subscribe(event: string, observer: Observer) {
    if (!this.observers.has(event)) this.observers.set(event, []);
    this.observers.get(event)!.push(observer);
  }

  emit(event: string, data: any) {
    this.observers.get(event)?.forEach(o => o.update(data));
  }
}
```

## Clean Code Practices

### Meaningful Names

```typescript
// BAD
function d(a: number, b: number) { return a * b * 0.0254; }

// GOOD
function calculateAreaInMeters(widthInInches: number, heightInInches: number) {
  const INCHES_TO_METERS = 0.0254;
  return widthInInches * heightInInches * INCHES_TO_METERS;
}
```

### Small Functions

```typescript
// BAD: 200-line function doing everything

// GOOD: Composed small functions
async function processOrder(orderId: string) {
  const order = await validateOrder(orderId);
  await checkInventory(order);
  const payment = await processPayment(order);
  await updateOrderStatus(orderId, 'paid');
  await sendConfirmationEmail(order);
}
```

### Avoid Magic Numbers

```typescript
// BAD
if (user.age < 18) { throw new Error('Too young'); }
setTimeout(fetchData, 86400000);

// GOOD
const MINIMUM_AGE = 18;
const ONE_DAY_MS = 24 * 60 * 60 * 1000;

if (user.age < MINIMUM_AGE) { throw new Error('Too young'); }
setTimeout(fetchData, ONE_DAY_MS);
```

### Error Handling

```typescript
// BAD
try {
  const user = await db.findUser(id);
  return user;
} catch (e) {
  console.log(e);
  return null;
}

// GOOD
try {
  const user = await db.findUser(id);
  if (!user) throw new UserNotFoundError(id);
  return user;
} catch (error) {
  logger.error('Failed to fetch user', { userId: id, error: error.message });
  throw new DatabaseError('User fetch failed', { cause: error });
}
```

### DRY (Don't Repeat Yourself)

```typescript
// BAD: Duplicated validation
app.post('/api/users', (req, res) => {
  if (!req.body.email || !req.body.email.includes('@')) {
    return res.status(400).json({ error: 'Invalid email' });
  }
});
app.put('/api/users/:id', (req, res) => {
  if (!req.body.email || !req.body.email.includes('@')) {
    return res.status(400).json({ error: 'Invalid email' });
  }
});

// GOOD: Reusable validation
function validateEmail(email: string) {
  if (!email || !email.includes('@')) {
    throw new ValidationError('Invalid email');
  }
}
```

## Refactoring Techniques

### Extract Method

```typescript
// Before
function renderOrder(order: Order) {
  console.log('Order Details:');
  console.log(`ID: ${order.id}`);
  console.log('Items:');
  order.items.forEach(item => console.log(`- ${item.name}`));
}

// After
function renderOrder(order: Order) {
  printOrderHeader(order);
  printOrderItems(order.items);
}
```

### Replace Conditional with Polymorphism

```typescript
// Before
function getShippingCost(method: string) {
  if (method === 'standard') return 5;
  if (method === 'express') return 15;
  if (method === 'overnight') return 30;
}

// After
interface ShippingMethod { getCost(): number; }
class StandardShipping implements ShippingMethod { getCost() { return 5; } }
class ExpressShipping implements ShippingMethod { getCost() { return 15; } }
```

## Code Quality Checklist

- [ ] SOLID principles applied
- [ ] Functions < 20 lines
- [ ] Meaningful names
- [ ] No magic numbers
- [ ] Proper error handling
- [ ] No duplication (DRY)
- [ ] Comments explain "why"
- [ ] Design patterns appropriate
- [ ] Dependency injection
- [ ] Readable > clever
