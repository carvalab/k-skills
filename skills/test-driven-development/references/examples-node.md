# TDD Examples: Node/TypeScript

Complete TDD example for Node.js and TypeScript projects.

## Example: Order Processing Service

### Step 1: RED - Write Failing Test

```typescript
// order.service.spec.ts
import { OrderService } from './order.service';
import { InMemoryOrderRepository } from './test/in-memory-order.repository';
import { FakePaymentGateway } from './test/fake-payment.gateway';

describe('OrderService', () => {
  let orderService: OrderService;
  let orderRepo: InMemoryOrderRepository;
  let paymentGateway: FakePaymentGateway;

  beforeEach(() => {
    orderRepo = new InMemoryOrderRepository();
    paymentGateway = new FakePaymentGateway();
    orderService = new OrderService(orderRepo, paymentGateway);
  });

  describe('processOrder', () => {
    it('should charge customer and mark order as paid', async () => {
      // Arrange
      const order = { id: '1', items: [{ price: 100 }], total: 100 };
      await orderRepo.save(order);

      // Act
      const result = await orderService.processOrder('1');

      // Assert
      expect(result.status).toBe('paid');
      expect(paymentGateway.charges).toHaveLength(1);
      expect(paymentGateway.charges[0].amount).toBe(100);
    });

    it('should throw error when order not found', async () => {
      await expect(orderService.processOrder('999'))
        .rejects.toThrow('Order not found');
    });

    it('should mark order as failed when payment fails', async () => {
      const order = { id: '1', items: [{ price: 100 }], total: 100 };
      await orderRepo.save(order);
      paymentGateway.shouldFail = true;

      const result = await orderService.processOrder('1');

      expect(result.status).toBe('payment_failed');
    });
  });
});
```

Run: `npm test` → Should fail

### Step 2: GREEN - Minimal Implementation

```typescript
// order.service.ts
export interface Order {
  id: string;
  items: { price: number }[];
  total: number;
  status?: string;
}

export interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}

export interface PaymentGateway {
  charge(amount: number): Promise<boolean>;
}

export class OrderService {
  constructor(
    private orderRepo: OrderRepository,
    private paymentGateway: PaymentGateway
  ) {}

  async processOrder(orderId: string): Promise<Order> {
    const order = await this.orderRepo.findById(orderId);
    if (!order) {
      throw new Error('Order not found');
    }

    const paymentSuccess = await this.paymentGateway.charge(order.total);
    order.status = paymentSuccess ? 'paid' : 'payment_failed';

    await this.orderRepo.save(order);
    return order;
  }
}
```

### Step 3: Test Doubles

```typescript
// test/in-memory-order.repository.ts
export class InMemoryOrderRepository implements OrderRepository {
  private orders: Map<string, Order> = new Map();

  async findById(id: string): Promise<Order | null> {
    return this.orders.get(id) ?? null;
  }

  async save(order: Order): Promise<void> {
    this.orders.set(order.id, { ...order });
  }
}

// test/fake-payment.gateway.ts
export class FakePaymentGateway implements PaymentGateway {
  charges: { amount: number }[] = [];
  shouldFail = false;

  async charge(amount: number): Promise<boolean> {
    if (this.shouldFail) {
      return false;
    }
    this.charges.push({ amount });
    return true;
  }
}
```

---

## Test Structure Patterns

### Describe Blocks

```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid email', () => { ... });
    it('should reject invalid email', () => { ... });
    it('should hash password before saving', () => { ... });
  });

  describe('deleteUser', () => {
    it('should remove user from database', () => { ... });
    it('should throw if user not found', () => { ... });
  });
});
```

### Setup and Teardown

```typescript
describe('DatabaseService', () => {
  let db: TestDatabase;

  beforeAll(async () => {
    db = await TestDatabase.create();
  });

  afterAll(async () => {
    await db.destroy();
  });

  beforeEach(async () => {
    await db.clear();
  });

  it('should insert record', async () => {
    await db.insert({ id: '1', name: 'Test' });
    expect(await db.findById('1')).toBeDefined();
  });
});
```

---

## Test Doubles in TypeScript

### Fake (In-Memory Implementation)

```typescript
class InMemoryUserRepository implements UserRepository {
  private users: Map<string, User> = new Map();

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) ?? null;
  }

  async save(user: User): Promise<void> {
    this.users.set(user.id, { ...user });
  }

  async delete(id: string): Promise<void> {
    this.users.delete(id);
  }
}
```

### Stub (Returns Canned Data)

```typescript
const stubUserService = {
  getUser: jest.fn().mockResolvedValue({ id: '1', name: 'Test User' }),
  createUser: jest.fn().mockResolvedValue({ id: '2', name: 'New User' }),
};
```

### Spy (Records Calls)

```typescript
class SpyLogger implements Logger {
  logs: { level: string; message: string }[] = [];

  log(level: string, message: string): void {
    this.logs.push({ level, message });
  }
}

// In test
it('should log error on failure', async () => {
  const spy = new SpyLogger();
  const service = new PaymentService(failingGateway, spy);

  await service.process(order);

  expect(spy.logs).toContainEqual({
    level: 'error',
    message: expect.stringContaining('payment failed')
  });
});
```

### Mock with Jest

```typescript
// Auto-mock module
jest.mock('./email.service');

// Manual mock
const mockEmailService = {
  send: jest.fn().mockResolvedValue(true),
};

// Verify calls
expect(mockEmailService.send).toHaveBeenCalledWith(
  'user@example.com',
  expect.stringContaining('Welcome')
);
expect(mockEmailService.send).toHaveBeenCalledTimes(1);
```

---

## Testing Async Code

```typescript
// Promises
it('should resolve with user', async () => {
  const user = await userService.getUser('1');
  expect(user.name).toBe('John');
});

// Rejections
it('should reject for invalid id', async () => {
  await expect(userService.getUser('invalid'))
    .rejects.toThrow('User not found');
});

// Callbacks (wrap in Promise)
it('should callback with result', (done) => {
  legacyService.fetchData((err, data) => {
    expect(err).toBeNull();
    expect(data).toBeDefined();
    done();
  });
});
```

---

## Commands

```bash
# Run all tests
npm test

# Watch mode
npm test -- --watch

# Run specific test file
npm test -- order.service.spec.ts

# Run tests matching pattern
npm test -- --testNamePattern="should create user"

# Coverage report
npm test -- --coverage

# Update snapshots
npm test -- --updateSnapshot

# Verbose output
npm test -- --verbose
```

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/*.spec.ts', '**/*.test.ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/index.ts',
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
    },
  },
};
```
