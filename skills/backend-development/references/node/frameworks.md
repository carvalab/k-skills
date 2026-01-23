# Node.js Frameworks

NestJS, Express, Fastify patterns.

## NestJS (Enterprise-Grade)

```typescript
// Module
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}

// Controller
@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()
  @UseGuards(JwtAuthGuard)
  async create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}

// Service
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepo: Repository<User>,
  ) {}

  async findOne(id: string): Promise<User> {
    const user = await this.usersRepo.findOne({ where: { id } });
    if (!user) throw new NotFoundException();
    return user;
  }
}
```

### DDD Bounded Contexts

For larger applications:

```
src/
├── contexts/
│   ├── shared/              # Shared domain & infrastructure
│   │   ├── domain/
│   │   └── infrastructure/
│   ├── users/               # User bounded context
│   │   ├── domain/
│   │   ├── infrastructure/
│   │   └── users.module.ts
│   └── orders/
├── platform/                # Server setup, config
└── index.ts
```

```typescript
// contexts/users/infrastructure/users.repository.ts
@Injectable()
export class UsersRepository {
  constructor(private readonly db: DatabaseService) {}

  async findById(id: string): Promise<User | null> {
    const result = await this.db.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0] || null;
  }
}

// contexts/users/users.module.ts
@Module({
  providers: [UsersService, UsersRepository],
  controllers: [UsersController],
  exports: [UsersService],
})
export class UsersModule {}
```

### Project Structure

```
src/
├── modules/
│   ├── users/
│   │   ├── dto/
│   │   ├── entities/
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── users.module.ts
│   │   └── users.repository.ts
│   └── auth/
├── common/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   └── decorators/
├── config/
├── app.module.ts
└── main.ts
```

## Express (Lightweight)

```typescript
import express from 'express';

const app = express();
app.use(express.json());

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await userService.findById(req.params.id);
    if (!user) return res.status(404).json({ error: 'Not found' });
    res.json(user);
  } catch (error) {
    next(error);
  }
});

// Error handler
app.use((err, req, res, next) => {
  logger.error(err);
  res.status(500).json({ error: 'Internal server error' });
});
```

### Express + Clean Architecture

See `architecture.md` for full 4-layer structure.

```typescript
// interface/controllers/user.controller.ts
export class UserController {
  private getUserUseCase: GetUserUseCase;

  constructor() {
    // Manual DI setup
    const repository = new UserRepositoryImpl(db);
    const mapper = new UserMapper();
    this.getUserUseCase = new GetUserUseCase(repository, mapper);
  }

  async getUser(req: Request, res: Response): Promise<void> {
    try {
      const result = await this.getUserUseCase.execute(req.params.id);
      res.json(result);
    } catch (error) {
      if (error instanceof NotFoundError) {
        res.status(404).json({ error: error.message });
      } else {
        res.status(500).json({ error: 'Internal server error' });
      }
    }
  }
}

// routes/user.routes.ts
const controller = new UserController();
router.get('/users/:id', (req, res) => controller.getUser(req, res));
```

## Fastify (High Performance)

```typescript
import Fastify from 'fastify';

const fastify = Fastify({ logger: true });

fastify.get('/users/:id', async (request, reply) => {
  const { id } = request.params;
  return userService.findById(id);
});

await fastify.listen({ port: 3000 });
```
