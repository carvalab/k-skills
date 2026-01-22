# Node.js Patterns

Validation, errors, async, testing, observability.

## Validation

### class-validator + class-transformer

```typescript
import { IsEmail, IsString, MinLength, IsOptional } from 'class-validator';
import { Transform } from 'class-transformer';

class CreateUserDto {
  @IsEmail()
  @Transform(({ value }) => value.toLowerCase())
  email: string;

  @IsString()
  @MinLength(12)
  password: string;

  @IsString()
  @IsOptional()
  name?: string;
}

// NestJS auto-validates
app.useGlobalPipes(new ValidationPipe({ transform: true }));
```

### Zod

```typescript
import { z } from 'zod';

const UserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(12),
  name: z.string().optional(),
});

type User = z.infer<typeof UserSchema>;

const result = UserSchema.safeParse(input);
if (!result.success) {
  throw new ValidationError(result.error.issues);
}
```

## Error Handling

```typescript
class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code?: string,
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

class ValidationError extends AppError {
  constructor(message: string, public details?: any[]) {
    super(message, 400, 'VALIDATION_ERROR');
  }
}

// NestJS global filter
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception instanceof HttpException
      ? exception.getStatus() : 500;

    logger.error(exception);
    response.status(status).json({
      error: {
        code: exception.code || 'INTERNAL_ERROR',
        message: exception.message || 'Internal server error',
        timestamp: new Date().toISOString(),
      },
    });
  }
}
```

## Async Patterns

```typescript
// Parallel (fast)
const [user, orders, preferences] = await Promise.all([
  getUser(id),
  getOrders(id),
  getPreferences(id),
]);

// Independent operations (don't fail all)
const results = await Promise.allSettled([
  sendEmail(user),
  sendSMS(user),
  sendPush(user),
]);
const failed = results.filter(r => r.status === 'rejected');
if (failed.length) logger.warn('Some notifications failed', failed);
```

## Testing

**New projects**: Vitest. **Existing**: Jest.

### Vitest

```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest';

describe('UserService', () => {
  let service: UserService;
  let mockRepo: MockUserRepository;

  beforeEach(() => {
    mockRepo = { findById: vi.fn(), save: vi.fn() };
    service = new UserService(mockRepo);
  });

  it('should return user when found', async () => {
    mockRepo.findById.mockResolvedValue({ id: '1', email: 'test@example.com' });
    const user = await service.getUser('1');
    expect(user.email).toBe('test@example.com');
    expect(mockRepo.findById).toHaveBeenCalledWith('1');
  });
});
```

### Jest (NestJS)

```typescript
import { Test, TestingModule } from '@nestjs/testing';

describe('UserService', () => {
  let service: UserService;
  let mockRepo: jest.Mocked<UserRepository>;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        { provide: UserRepository, useValue: { findById: jest.fn() } },
      ],
    }).compile();

    service = module.get(UserService);
    mockRepo = module.get(UserRepository);
  });

  it('should return user', async () => {
    mockRepo.findById.mockResolvedValue({ id: '1', email: 'test@example.com' });
    const user = await service.getUser('1');
    expect(user.email).toBe('test@example.com');
  });
});
```

## Observability

See `debugging.md` for full patterns.

```typescript
import tracer from 'dd-trace';

// Initialize BEFORE other imports
tracer.init({
  service: 'my-api',
  env: process.env.KAVAK_ENVIRONMENT,
});

// Structured logging
import pino from 'pino';
const logger = pino({ level: process.env.LOG_LEVEL || 'info' });
logger.info({ userId: user.id, action: 'created' }, 'User created');
```

## Performance Tips

1. **Connection pooling** - pg Pool, Prisma connection limit
2. **Avoid N+1** - Use includes/joins, batch queries
3. **Stream large data** - Readable streams for exports
4. **Worker threads** - CPU-intensive tasks
5. **Cache with Redis** - Hot data, sessions (with local fallback)
6. **Use async/await** - Never block event loop
