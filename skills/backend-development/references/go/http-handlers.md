# Go HTTP Handlers

Chi router, handlers, middleware patterns.

## Chi Router Setup

```go
package main

import (
    "net/http"
    "github.com/go-chi/chi/v5"
    "github.com/go-chi/chi/v5/middleware"
)

func main() {
    r := chi.NewRouter()

    // Middleware
    r.Use(middleware.Logger)
    r.Use(middleware.Recoverer)
    r.Use(middleware.RequestID)
    r.Use(middleware.RealIP)
    r.Use(middleware.Timeout(60 * time.Second))

    // Routes
    r.Route("/api/v1", func(r chi.Router) {
        r.Route("/users", func(r chi.Router) {
            r.Get("/", listUsers)
            r.Post("/", createUser)
            r.Get("/{id}", getUser)
            r.Put("/{id}", updateUser)
            r.Delete("/{id}", deleteUser)
        })
    })

    http.ListenAndServe(":3000", r)
}
```

## Handler Pattern

```go
type UserHandler struct {
    service *UserService
}

func NewUserHandler(service *UserService) *UserHandler {
    return &UserHandler{service: service}
}

func (h *UserHandler) List(w http.ResponseWriter, r *http.Request) {
    page, _ := strconv.Atoi(r.URL.Query().Get("page"))
    limit, _ := strconv.Atoi(r.URL.Query().Get("limit"))

    if page < 1 { page = 1 }
    if limit < 1 || limit > 100 { limit = 50 }

    users, total, err := h.service.List(r.Context(), page, limit)
    if err != nil {
        respondError(w, http.StatusInternalServerError, err.Error())
        return
    }

    respondJSON(w, http.StatusOK, map[string]interface{}{
        "data": users,
        "pagination": map[string]interface{}{
            "page": page, "limit": limit, "total": total,
        },
    })
}

func (h *UserHandler) Get(w http.ResponseWriter, r *http.Request) {
    id := chi.URLParam(r, "id")
    user, err := h.service.FindByID(r.Context(), id)
    if err != nil {
        respondError(w, http.StatusNotFound, "User not found")
        return
    }
    respondJSON(w, http.StatusOK, user)
}

func (h *UserHandler) Create(w http.ResponseWriter, r *http.Request) {
    var dto CreateUserDTO
    if err := json.NewDecoder(r.Body).Decode(&dto); err != nil {
        respondError(w, http.StatusBadRequest, "Invalid JSON")
        return
    }
    if err := dto.Validate(); err != nil {
        respondError(w, http.StatusBadRequest, err.Error())
        return
    }
    user, err := h.service.Create(r.Context(), dto)
    if err != nil {
        respondError(w, http.StatusInternalServerError, err.Error())
        return
    }
    respondJSON(w, http.StatusCreated, user)
}
```

## Response Helpers

```go
func respondJSON(w http.ResponseWriter, status int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(data)
}

func respondError(w http.ResponseWriter, status int, message string) {
    respondJSON(w, status, map[string]interface{}{
        "error": map[string]interface{}{
            "code":      http.StatusText(status),
            "message":   message,
            "timestamp": time.Now().UTC().Format(time.RFC3339),
        },
    })
}
```

## Auth Middleware

```go
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" {
            respondError(w, http.StatusUnauthorized, "Missing authorization")
            return
        }
        claims, err := validateToken(token)
        if err != nil {
            respondError(w, http.StatusUnauthorized, "Invalid token")
            return
        }
        ctx := context.WithValue(r.Context(), "user", claims)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// Usage
r.Route("/api/v1", func(r chi.Router) {
    r.Use(AuthMiddleware)
    r.Get("/protected", protectedHandler)
})
```

## net/http (Standard Library)

```go
func main() {
    mux := http.NewServeMux()

    mux.HandleFunc("GET /api/v1/users", listUsers)
    mux.HandleFunc("POST /api/v1/users", createUser)
    mux.HandleFunc("GET /api/v1/users/{id}", getUser)

    server := &http.Server{
        Addr:         ":3000",
        Handler:      mux,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
    }
    server.ListenAndServe()
}

func getUser(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    user, err := userService.FindByID(r.Context(), id)
    if err != nil {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}
```
