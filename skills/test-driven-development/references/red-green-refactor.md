# Red-Green-Refactor Cycle

The fundamental TDD rhythm. Master this and TDD becomes natural.

## Overview

```
┌─────────┐     ┌─────────┐     ┌───────────┐
│   RED   │────▶│  GREEN  │────▶│ REFACTOR  │
│  (fail) │     │ (pass)  │     │ (improve) │
└─────────┘     └─────────┘     └───────────┘
     ▲                                │
     └────────────────────────────────┘
```

## Phase 1: RED (Write Failing Test)

### Purpose
- Define the behavior you want
- Verify the test can actually fail
- Create a specification

### Rules
1. Write ONE test only
2. Test must fail
3. Test must fail for the RIGHT reason
4. Don't write production code yet

### Example (Go)

```go
func TestCalculator_Add_ReturnsSum(t *testing.T) {
    // Arrange
    calc := NewCalculator()

    // Act
    result := calc.Add(2, 3)

    // Assert
    if result != 5 {
        t.Errorf("Add(2, 3) = %d; want 5", result)
    }
}
```

Run: `go test ./...` → Should fail (Calculator doesn't exist)

### Example (Node/TypeScript)

```typescript
describe('Calculator', () => {
  it('should return sum when adding two numbers', () => {
    // Arrange
    const calc = new Calculator();

    // Act
    const result = calc.add(2, 3);

    // Assert
    expect(result).toBe(5);
  });
});
```

Run: `npm test` → Should fail

### Example (Python)

```python
def test_calculator_add_returns_sum():
    # Arrange
    calc = Calculator()

    # Act
    result = calc.add(2, 3)

    # Assert
    assert result == 5
```

Run: `pytest` → Should fail

### Example (Java)

```java
@Test
void shouldReturnSumWhenAddingTwoNumbers() {
    // Arrange
    Calculator calc = new Calculator();

    // Act
    int result = calc.add(2, 3);

    // Assert
    assertEquals(5, result);
}
```

Run: `./mvnw test` → Should fail

## Phase 2: GREEN (Make It Pass)

### Purpose
- Get to green as fast as possible
- Prove the test can pass
- Defer design decisions

### Rules
1. Write MINIMAL code to pass
2. Don't add extra functionality
3. "Fake it" is acceptable
4. Don't refactor yet

### Minimal Implementation

```go
// Go - Just enough to pass
type Calculator struct{}

func NewCalculator() *Calculator {
    return &Calculator{}
}

func (c *Calculator) Add(a, b int) int {
    return 5 // Fake it! This passes the test
}
```

```typescript
// TypeScript - Just enough to pass
class Calculator {
  add(a: number, b: number): number {
    return 5; // Fake it!
  }
}
```

### Why "Fake It" Works

1. Proves the test infrastructure works
2. Gives you a green baseline
3. Forces you to write another test to drive real implementation

### Next Test Forces Real Implementation

```go
func TestCalculator_Add_DifferentNumbers(t *testing.T) {
    calc := NewCalculator()
    result := calc.Add(10, 20)
    if result != 30 {
        t.Errorf("Add(10, 20) = %d; want 30", result)
    }
}
```

Now you must implement properly:

```go
func (c *Calculator) Add(a, b int) int {
    return a + b // Real implementation
}
```

## Phase 3: REFACTOR (Improve Design)

### Purpose
- Clean up code
- Remove duplication
- Improve readability
- KEEP TESTS GREEN

### Rules
1. Run tests after EVERY change
2. Small steps only
3. If tests fail, undo immediately
4. Refactor production code AND test code

### Common Refactorings

| Refactoring | When |
|-------------|------|
| Extract Method | Duplicated code |
| Rename | Unclear names |
| Extract Constant | Magic numbers |
| Simplify Conditionals | Complex if/else |
| Extract Interface | Reduce coupling |

### Refactoring Example

Before:
```go
func (c *Calculator) Process(a, b int, op string) int {
    if op == "add" {
        return a + b
    } else if op == "subtract" {
        return a - b
    } else if op == "multiply" {
        return a * b
    }
    return 0
}
```

After (tests still pass):
```go
func (c *Calculator) Process(a, b int, op string) int {
    operations := map[string]func(int, int) int{
        "add":      func(a, b int) int { return a + b },
        "subtract": func(a, b int) int { return a - b },
        "multiply": func(a, b int) int { return a * b },
    }

    if fn, ok := operations[op]; ok {
        return fn(a, b)
    }
    return 0
}
```

## Cycle Timing

| Phase | Time | Goal |
|-------|------|------|
| RED | 1-3 min | Write failing test |
| GREEN | 1-5 min | Make it pass |
| REFACTOR | 2-10 min | Clean up |

If any phase takes longer, you're taking too big a step. Break it down.

## Signs You're Doing It Right

- Tests fail when you expect them to
- You feel confident making changes
- Refactoring is safe and easy
- Tests document behavior
- Code coverage is naturally high

## Signs You're Doing It Wrong

- Writing tests after code
- Skipping the RED phase
- Not running tests between changes
- Tests don't fail when behavior breaks
- Large refactorings without tests
