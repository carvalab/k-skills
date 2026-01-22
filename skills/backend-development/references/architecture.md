# Backend Architecture Patterns

Microservices, event-driven architecture, and scalability patterns (2026).

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
]

// Reconstruct state by replaying
const balance = events.reduce((acc, event) => {
  if (event.type === 'MoneyDeposited') return acc + event.amount;
  if (event.type === 'MoneyWithdrawn') return acc - event.amount;
  return acc;
}, 0);
```

### Kafka (Event Streaming)

```typescript
import { Kafka } from 'kafkajs';

const kafka = new Kafka({ clientId: 'order-service', brokers: ['kafka:9092'] });

// Producer
await producer.send({
  topic: 'order-events',
  messages: [{ key: order.id, value: JSON.stringify(event) }],
});

// Consumer
await consumer.subscribe({ topic: 'order-events' });
await consumer.run({
  eachMessage: async ({ message }) => {
    const event = JSON.parse(message.value.toString());
    await processEvent(event);
  },
});
```

### RabbitMQ (Task Queues)

```typescript
import amqp from 'amqplib';

// Producer
channel.sendToQueue('email-queue', Buffer.from(JSON.stringify(emailData)));

// Consumer
await channel.consume('email-queue', async (msg) => {
  const data = JSON.parse(msg.content.toString());
  await sendEmail(data);
  channel.ack(msg);
});
```

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
