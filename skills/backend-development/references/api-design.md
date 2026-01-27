# Backend API Design

REST API patterns and best practices (2026).

## REST API Design

### Resource-Based URLs

```
GET    /api/v1/users              # List users
GET    /api/v1/users/:id          # Get user
POST   /api/v1/users              # Create user
PUT    /api/v1/users/:id          # Update user (full)
PATCH  /api/v1/users/:id          # Update user (partial)
DELETE /api/v1/users/:id          # Delete user
GET    /api/v1/users/:id/posts    # Get user's posts
```

**Avoid:**

```
GET /api/v1/getUser?id=123        # RPC-style
POST /api/v1/createUser           # Verb in URL
```

### HTTP Status Codes

**Success:**

- `200 OK` - GET, PUT, PATCH success
- `201 Created` - POST success
- `204 No Content` - DELETE success

**Client Errors:**

- `400 Bad Request` - Invalid input
- `401 Unauthorized` - Missing/invalid auth
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - Duplicate resource
- `422 Unprocessable Entity` - Validation error
- `429 Too Many Requests` - Rate limited

**Server Errors:**

- `500 Internal Server Error` - Generic
- `503 Service Unavailable` - Temporary downtime

### Request/Response Format

**Request:**

```json
POST /api/v1/users
Content-Type: application/json

{
  "email": "user@example.com",
  "name": "John Doe"
}
```

**Success Response:**

```json
HTTP/1.1 201 Created
Location: /api/v1/users/123

{
  "id": "123",
  "email": "user@example.com",
  "name": "John Doe",
  "createdAt": "2026-01-09T12:00:00Z"
}
```

**Error Response:**

```json
HTTP/1.1 400 Bad Request

{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      { "field": "email", "message": "Invalid email format" }
    ],
    "timestamp": "2026-01-09T12:00:00Z"
  }
}
```

### Pagination

```json
GET /api/v1/users?page=2&limit=50

{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 50,
    "total": 1234,
    "totalPages": 25,
    "hasNext": true,
    "hasPrev": true
  },
  "links": {
    "first": "/api/v1/users?page=1&limit=50",
    "prev": "/api/v1/users?page=1&limit=50",
    "next": "/api/v1/users?page=3&limit=50",
    "last": "/api/v1/users?page=25&limit=50"
  }
}
```

### Filtering & Sorting

```
GET /api/v1/users?status=active&role=admin&sort=-createdAt,name&limit=20

# Filters: status=active AND role=admin
# Sort: createdAt DESC, name ASC
# Limit: 20 results
```

### API Versioning

**URL Versioning (Recommended):**

```
/api/v1/users
/api/v2/users
```

**Header Versioning:**

```
GET /api/users
Accept: application/vnd.myapi.v2+json
```

## API Implementation

### Node.js (NestJS)

```typescript
@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Get()
  async findAll(@Query('page') page = 1, @Query('limit') limit = 50) {
    return this.usersService.findAll({ page, limit });
  }

  @Get(':id')
  async findOne(@Param('id') id: string) {
    const user = await this.usersService.findOne(id);
    if (!user) throw new NotFoundException();
    return user;
  }

  @Post()
  @HttpCode(201)
  async create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }

  @Patch(':id')
  async update(@Param('id') id: string, @Body() dto: UpdateUserDto) {
    return this.usersService.update(id, dto);
  }

  @Delete(':id')
  @HttpCode(204)
  async remove(@Param('id') id: string) {
    await this.usersService.remove(id);
  }
}
```

### Go (Chi)

```go
func NewRouter(userHandler *UserHandler) *chi.Mux {
    r := chi.NewRouter()

    r.Use(middleware.Logger)
    r.Use(middleware.Recoverer)

    r.Route("/api/v1/users", func(r chi.Router) {
        r.Get("/", userHandler.List)
        r.Post("/", userHandler.Create)
        r.Get("/{id}", userHandler.Get)
        r.Put("/{id}", userHandler.Update)
        r.Delete("/{id}", userHandler.Delete)
    })

    return r
}

func (h *UserHandler) Get(w http.ResponseWriter, r *http.Request) {
    id := chi.URLParam(r, "id")

    user, err := h.service.FindByID(r.Context(), id)
    if err != nil {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}

func (h *UserHandler) Create(w http.ResponseWriter, r *http.Request) {
    var dto CreateUserDTO
    if err := json.NewDecoder(r.Body).Decode(&dto); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }

    user, err := h.service.Create(r.Context(), dto)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(user)
}
```

## OpenAPI Documentation

```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /api/v1/users:
    get:
      summary: List users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 50
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        email:
          type: string
        name:
          type: string
```

## API Security Checklist

- [ ] HTTPS only
- [ ] Authentication (OAuth 2.1, JWT)
- [ ] Authorization (RBAC)
- [ ] Rate limiting
- [ ] Input validation
- [ ] CORS configured
- [ ] Security headers
- [ ] Audit logging
- [ ] Error messages don't leak system info

## Resources

- REST Best Practices: https://restfulapi.net/
- OpenAPI Specification: https://swagger.io/specification/
