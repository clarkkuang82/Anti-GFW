# Red-Green-Refactor: Test-Driven Development

## Overview

Test-Driven Development (TDD) is a software development methodology where tests are written **before** the production code. The core cycle is known as **Red-Green-Refactor**.

## The TDD Cycle

### 1. RED - Write a Failing Test

Write a test that defines the desired behavior. Run it and watch it **fail**.

- The test should be small and focused on one behavior
- It must fail for the right reason (not due to syntax errors)
- The failure message should clearly describe what's missing

```python
# Example: Building a calculator

# test_calculator.py
def test_add_two_numbers():
    calc = Calculator()
    result = calc.add(2, 3)
    assert result == 5
```

Running this test produces a **RED** failure: `NameError: name 'Calculator' is not defined`.

### 2. GREEN - Write the Minimum Code to Pass

Write the **simplest possible code** that makes the test pass. No more, no less.

- Do not over-engineer
- Do not add features not required by the test
- It's okay if the code is ugly — that's what Refactor is for

```python
# calculator.py
class Calculator:
    def add(self, a, b):
        return a + b
```

Running the test now produces a **GREEN** pass.

### 3. REFACTOR - Improve the Code

Clean up the code while keeping all tests passing.

- Remove duplication
- Improve naming
- Simplify logic
- Extract methods or classes if needed
- Run tests after every change to ensure nothing breaks

## Key Principles

### Write the Test First

Never write production code without a failing test. The test defines the specification.

### Baby Steps

Each cycle should be small — ideally under 5 minutes. If you're stuck for longer, revert and try a smaller step.

### One Assertion Per Test

Each test should verify one behavior. This makes failures easy to diagnose.

```python
# Good - focused
def test_add_positive_numbers():
    assert Calculator().add(2, 3) == 5

def test_add_negative_numbers():
    assert Calculator().add(-1, -2) == -3

# Bad - testing multiple behaviors
def test_calculator():
    calc = Calculator()
    assert calc.add(2, 3) == 5
    assert calc.subtract(5, 3) == 2
    assert calc.multiply(2, 4) == 8
```

### Triangulation

When the simplest implementation is a hard-coded return value, add another test case to force generalization.

```python
# First test — could be solved with `return 5`
def test_add_2_and_3():
    assert Calculator().add(2, 3) == 5

# Second test — forces real implementation
def test_add_1_and_1():
    assert Calculator().add(1, 1) == 2
```

### The Three Laws of TDD (Robert C. Martin)

1. You may not write production code until you have a failing test.
2. You may not write more of a test than is sufficient to fail.
3. You may not write more production code than is sufficient to pass the test.

## TDD Workflow Example

Building a `FizzBuzz` function step by step:

**Cycle 1 — RED:**
```python
def test_returns_1_for_1():
    assert fizzbuzz(1) == "1"
```

**Cycle 1 — GREEN:**
```python
def fizzbuzz(n):
    return str(n)
```

**Cycle 2 — RED:**
```python
def test_returns_fizz_for_3():
    assert fizzbuzz(3) == "Fizz"
```

**Cycle 2 — GREEN:**
```python
def fizzbuzz(n):
    if n % 3 == 0:
        return "Fizz"
    return str(n)
```

**Cycle 3 — RED:**
```python
def test_returns_buzz_for_5():
    assert fizzbuzz(5) == "Buzz"
```

**Cycle 3 — GREEN:**
```python
def fizzbuzz(n):
    if n % 3 == 0:
        return "Fizz"
    if n % 5 == 0:
        return "Buzz"
    return str(n)
```

**Cycle 4 — RED:**
```python
def test_returns_fizzbuzz_for_15():
    assert fizzbuzz(15) == "FizzBuzz"
```

**Cycle 4 — GREEN:**
```python
def fizzbuzz(n):
    if n % 15 == 0:
        return "FizzBuzz"
    if n % 3 == 0:
        return "Fizz"
    if n % 5 == 0:
        return "Buzz"
    return str(n)
```

**Cycle 4 — REFACTOR:**
```python
def fizzbuzz(n):
    result = ""
    if n % 3 == 0:
        result += "Fizz"
    if n % 5 == 0:
        result += "Buzz"
    return result or str(n)
```

## Common TDD Mistakes

| Mistake | Fix |
|---------|-----|
| Writing tests after code | Discipline — always RED first |
| Making steps too large | Break into smaller behaviors |
| Refactoring while RED | Get to GREEN first, then refactor |
| Skipping the refactor step | Schedule it — it's not optional |
| Testing implementation details | Test behavior and outcomes, not internals |
| Ignoring failing tests | Fix or delete — never leave broken tests |

## When to Use TDD

- Business logic and algorithms
- Data transformations
- API endpoints and request handling
- Utility functions
- Bug fixes (write a test that reproduces the bug first)

## When TDD May Not Fit

- Exploratory prototyping (but write tests before shipping)
- UI layout and styling
- One-off scripts
- Integration with external systems (use integration tests separately)

## Benefits of TDD

- **Confidence**: Every behavior is tested
- **Design feedback**: Hard-to-test code signals design problems
- **Documentation**: Tests describe what the code does
- **Regression safety**: Refactoring without fear
- **Focus**: Solve one problem at a time
