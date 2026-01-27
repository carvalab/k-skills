# Common Go Issues and Fixes

## Error Handling

### Ignoring Errors

**Issue**: Not checking returned errors

```go
// Bad
file, _ := os.Open("config.json")
data, _ := io.ReadAll(file)

// Good
file, err := os.Open("config.json")
if err != nil {
    return fmt.Errorf("open config: %w", err)
}
defer file.Close()

data, err := io.ReadAll(file)
if err != nil {
    return fmt.Errorf("read config: %w", err)
}
```

### Naked Returns with Named Results

**Issue**: Hard to follow, especially in long functions

```go
// Bad
func process(data []byte) (result string, err error) {
    // ... 50 lines later ...
    return // what's being returned?
}

// Good
func process(data []byte) (string, error) {
    // ... processing ...
    return result, nil
}
```

### Not Wrapping Errors

**Issue**: Loses context for debugging

```go
// Bad
if err != nil {
    return err
}

// Good
if err != nil {
    return fmt.Errorf("process user %d: %w", userID, err)
}
```

---

## Resource Management

### Forgetting defer for Cleanup

**Issue**: Resource leaks

```go
// Bad
func readFile(path string) ([]byte, error) {
    f, err := os.Open(path)
    if err != nil {
        return nil, err
    }
    // f.Close() never called if error below
    data, err := io.ReadAll(f)
    if err != nil {
        return nil, err
    }
    f.Close()
    return data, nil
}

// Good
func readFile(path string) ([]byte, error) {
    f, err := os.Open(path)
    if err != nil {
        return nil, err
    }
    defer f.Close()
    return io.ReadAll(f)
}
```

### defer in Loop

**Issue**: Resources not released until function exits

```go
// Bad - all files open until function returns
for _, path := range paths {
    f, _ := os.Open(path)
    defer f.Close() // won't close until loop ends
    process(f)
}

// Good - use closure or explicit close
for _, path := range paths {
    func() {
        f, _ := os.Open(path)
        defer f.Close()
        process(f)
    }()
}
```

---

## Concurrency

### Data Race on Shared Variable

**Issue**: Concurrent access without synchronization

```go
// Bad
var counter int
for i := 0; i < 10; i++ {
    go func() {
        counter++ // race condition
    }()
}

// Good - use mutex or atomic
var counter int64
for i := 0; i < 10; i++ {
    go func() {
        atomic.AddInt64(&counter, 1)
    }()
}
```

### Goroutine Leak

**Issue**: Goroutine blocks forever

```go
// Bad - no way to stop
func startWorker() {
    go func() {
        for {
            doWork()
        }
    }()
}

// Good - use context for cancellation
func startWorker(ctx context.Context) {
    go func() {
        for {
            select {
            case <-ctx.Done():
                return
            default:
                doWork()
            }
        }
    }()
}
```

### Loop Variable Capture

**Issue**: All goroutines see same value (fixed in Go 1.22+)

```go
// Bad (Go < 1.22)
for _, item := range items {
    go func() {
        process(item) // all goroutines see last item
    }()
}

// Good
for _, item := range items {
    item := item // shadow variable
    go func() {
        process(item)
    }()
}
```

---

## Slices and Maps

### Nil Map Write

**Issue**: Panic on write to nil map

```go
// Bad
var users map[string]User
users["alice"] = User{} // panic!

// Good
users := make(map[string]User)
users["alice"] = User{}
```

### Slice Append Side Effect

**Issue**: Modifying underlying array

```go
// Bad - may modify original
func addItem(items []string, item string) []string {
    return append(items, item)
}

// Good - copy first if you need isolation
func addItem(items []string, item string) []string {
    result := make([]string, len(items), len(items)+1)
    copy(result, items)
    return append(result, item)
}
```

---

## Interface Design

### Empty Interface Overuse

**Issue**: Loses type safety

```go
// Bad
func process(data interface{}) { ... }

// Good - use generics or specific types
func process[T any](data T) { ... }
// or
func processUser(user User) { ... }
```

### Returning Concrete Type for Interface

**Issue**: Prevents nil interface issues

```go
// Bad - can return (*MyError)(nil) which != nil
func validate() error {
    var err *MyError
    // ... validation ...
    return err // dangerous!
}

// Good
func validate() error {
    // ... validation ...
    if invalid {
        return &MyError{msg: "invalid"}
    }
    return nil // explicit nil
}
```

---

## Testing

### Not Using Table-Driven Tests

**Issue**: Repetitive test code

```go
// Bad
func TestAdd(t *testing.T) {
    if Add(1, 2) != 3 {
        t.Error("1+2 should be 3")
    }
    if Add(0, 0) != 0 {
        t.Error("0+0 should be 0")
    }
}

// Good
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 1, 2, 3},
        {"zeros", 0, 0, 0},
        {"negative", -1, 1, 0},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.expected {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, got, tt.expected)
            }
        })
    }
}
```

---

## Performance

### String Concatenation in Loop

**Issue**: Creates many allocations

```go
// Bad
var result string
for _, s := range items {
    result += s // O(n²) allocations
}

// Good
var builder strings.Builder
for _, s := range items {
    builder.WriteString(s)
}
result := builder.String()
```

### Unnecessary Allocations

**Issue**: Allocating when not needed

```go
// Bad
func contains(items []string, target string) bool {
    set := make(map[string]bool) // unnecessary allocation
    for _, item := range items {
        set[item] = true
    }
    return set[target]
}

// Good
func contains(items []string, target string) bool {
    for _, item := range items {
        if item == target {
            return true
        }
    }
    return false
}
```
