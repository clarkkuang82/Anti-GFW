# Common Coding Skills

## 1. Clean Code

### Meaningful Names

Names should reveal intent. A reader should understand what a variable, function, or class does without needing comments.

```python
# Bad
d = 86400
def calc(a, b):
    return a * b / d

# Good
SECONDS_PER_DAY = 86400
def calculate_daily_rate(total_amount, days):
    return total_amount * days / SECONDS_PER_DAY
```

### Functions Should Do One Thing

A function should perform a single responsibility. If you can extract part of it into a separate function with a meaningful name, it's doing more than one thing.

```python
# Bad - does three things
def process_user(user_data):
    # validate
    if not user_data.get("email"):
        raise ValueError("Email required")
    # transform
    user_data["email"] = user_data["email"].lower()
    # save
    db.save(user_data)

# Good - each function does one thing
def validate_user(user_data):
    if not user_data.get("email"):
        raise ValueError("Email required")

def normalize_email(user_data):
    user_data["email"] = user_data["email"].lower()
    return user_data

def save_user(user_data):
    db.save(user_data)
```

### Keep Functions Small

Functions should be short — typically under 20 lines. If a function is long, break it into smaller functions.

### Avoid Magic Numbers

Replace literal values with named constants.

```python
# Bad
if retry_count > 3:
    raise TimeoutError()

# Good
MAX_RETRIES = 3
if retry_count > MAX_RETRIES:
    raise TimeoutError()
```

## 2. SOLID Principles

### Single Responsibility Principle (SRP)

A class should have only one reason to change.

```python
# Bad - handles both user logic and persistence
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def save_to_database(self):
        db.execute(f"INSERT INTO users ...")

# Good - separated concerns
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserRepository:
    def save(self, user):
        db.execute(f"INSERT INTO users ...")
```

### Open/Closed Principle (OCP)

Open for extension, closed for modification. Add new behavior without changing existing code.

```python
# Using polymorphism instead of conditionals
class Shape:
    def area(self):
        raise NotImplementedError

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height
```

### Liskov Substitution Principle (LSP)

Subtypes must be substitutable for their base types without breaking correctness.

### Interface Segregation Principle (ISP)

Don't force clients to depend on methods they don't use. Prefer small, focused interfaces.

### Dependency Inversion Principle (DIP)

Depend on abstractions, not concretions. High-level modules should not depend on low-level modules.

```python
# Bad - depends on concrete implementation
class OrderService:
    def __init__(self):
        self.db = MySQLDatabase()

# Good - depends on abstraction
class OrderService:
    def __init__(self, db):
        self.db = db
```

## 3. Design Patterns

### Strategy Pattern

Encapsulate interchangeable algorithms behind a common interface.

```python
class Sorter:
    def __init__(self, strategy):
        self.strategy = strategy

    def sort(self, data):
        return self.strategy(data)

# Usage
sorter = Sorter(strategy=sorted)
result = sorter.sort([3, 1, 2])
```

### Factory Pattern

Centralize object creation logic.

```python
def create_connection(db_type):
    if db_type == "postgres":
        return PostgresConnection()
    elif db_type == "mysql":
        return MySQLConnection()
    raise ValueError(f"Unknown db type: {db_type}")
```

### Observer Pattern

Notify multiple objects when state changes.

```python
class EventEmitter:
    def __init__(self):
        self.listeners = {}

    def on(self, event, callback):
        self.listeners.setdefault(event, []).append(callback)

    def emit(self, event, data=None):
        for callback in self.listeners.get(event, []):
            callback(data)
```

## 4. Refactoring Techniques

### Extract Method

Pull a block of code into its own function.

### Rename Variable/Function

Improve clarity by choosing better names.

### Remove Duplication (DRY)

If the same logic appears in multiple places, extract it into a shared function.

```python
# Before - duplicated validation
def create_user(email):
    if "@" not in email:
        raise ValueError("Invalid email")
    ...

def update_email(email):
    if "@" not in email:
        raise ValueError("Invalid email")
    ...

# After - shared validation
def validate_email(email):
    if "@" not in email:
        raise ValueError("Invalid email")

def create_user(email):
    validate_email(email)
    ...

def update_email(email):
    validate_email(email)
    ...
```

### Replace Conditional with Polymorphism

Use objects and method dispatch instead of long if/elif chains.

### Simplify Boolean Expressions

```python
# Before
if is_active == True and is_deleted == False:
    ...

# After
if is_active and not is_deleted:
    ...
```

## 5. Debugging Skills

### Read the Error Message

Error messages tell you what went wrong and where. Read the full stack trace before guessing.

### Reproduce First

Before fixing a bug, write a test or find exact steps to reproduce it consistently.

### Binary Search Debugging

When a bug is elusive, systematically eliminate half the possible causes at each step.

### Rubber Duck Debugging

Explain the problem out loud (or in writing) step by step. The act of explaining often reveals the issue.

### Use a Debugger

Step through code with a debugger instead of adding print statements. Learn your IDE's debugging tools.

## 6. Version Control Best Practices

### Commit Often, Commit Small

Each commit should represent one logical change. This makes history easy to read and revert.

### Write Descriptive Commit Messages

```
# Bad
fix bug

# Good
Fix off-by-one error in pagination that skipped the last page
```

### Use Branches

Work on features and fixes in separate branches. Merge via pull requests with code review.

### Never Commit Secrets

Keep API keys, passwords, and credentials out of version control. Use environment variables or secret managers.

## 7. Code Review Skills

### Review for Correctness

Does the code do what it's supposed to? Are edge cases handled?

### Review for Readability

Can another developer understand this code without extra explanation?

### Review for Simplicity

Is there a simpler way to achieve the same result?

### Give Constructive Feedback

```
# Bad
"This is wrong."

# Good
"This could cause a null pointer exception when `user` is None — 
consider adding a guard clause."
```

## 8. Test-Driven Development (Red-Green-Refactor)

TDD is a methodology where tests are written **before** production code. The core cycle is **Red-Green-Refactor**.

### The TDD Cycle

#### RED - Write a Failing Test

Write a test that defines the desired behavior. Run it and watch it **fail**.

- The test should be small and focused on one behavior
- It must fail for the right reason (not due to syntax errors)
- The failure message should clearly describe what's missing

```python
# test_calculator.py
def test_add_two_numbers():
    calc = Calculator()
    result = calc.add(2, 3)
    assert result == 5
```

Running this test produces a **RED** failure: `NameError: name 'Calculator' is not defined`.

#### GREEN - Write the Minimum Code to Pass

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

#### REFACTOR - Improve the Code

Clean up the code while keeping all tests passing.

- Remove duplication
- Improve naming
- Simplify logic
- Extract methods or classes if needed
- Run tests after every change to ensure nothing breaks

### TDD Key Principles

**Write the Test First** — Never write production code without a failing test. The test defines the specification.

**Baby Steps** — Each cycle should be small — ideally under 5 minutes. If you're stuck for longer, revert and try a smaller step.

**One Assertion Per Test** — Each test should verify one behavior. This makes failures easy to diagnose.

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

**Triangulation** — When the simplest implementation is a hard-coded return value, add another test case to force generalization.

```python
# First test — could be solved with `return 5`
def test_add_2_and_3():
    assert Calculator().add(2, 3) == 5

# Second test — forces real implementation
def test_add_1_and_1():
    assert Calculator().add(1, 1) == 2
```

**The Three Laws of TDD (Robert C. Martin):**

1. You may not write production code until you have a failing test.
2. You may not write more of a test than is sufficient to fail.
3. You may not write more production code than is sufficient to pass the test.

### TDD Workflow Example: FizzBuzz

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

### Common TDD Mistakes

| Mistake | Fix |
|---------|-----|
| Writing tests after code | Discipline — always RED first |
| Making steps too large | Break into smaller behaviors |
| Refactoring while RED | Get to GREEN first, then refactor |
| Skipping the refactor step | Schedule it — it's not optional |
| Testing implementation details | Test behavior and outcomes, not internals |
| Ignoring failing tests | Fix or delete — never leave broken tests |

### When to Use TDD

- Business logic and algorithms
- Data transformations
- API endpoints and request handling
- Utility functions
- Bug fixes (write a test that reproduces the bug first)

### When TDD May Not Fit

- Exploratory prototyping (but write tests before shipping)
- UI layout and styling
- One-off scripts
- Integration with external systems (use integration tests separately)

## 9. Testing Best Practices

### Test Pyramid

- **Unit tests** (many): Fast, isolated, test single functions
- **Integration tests** (some): Test component interactions
- **End-to-end tests** (few): Test full user workflows

### Arrange-Act-Assert (AAA)

Structure every test in three clear phases:

```python
def test_withdraw_reduces_balance():
    # Arrange
    account = Account(balance=100)

    # Act
    account.withdraw(30)

    # Assert
    assert account.balance == 70
```

### Test Edge Cases

- Empty inputs
- Boundary values (0, -1, max int)
- Null/None values
- Large inputs
- Concurrent access

### Tests as Documentation

Good tests describe the system's behavior. Name them to read like specifications:

```python
def test_expired_coupon_is_rejected():
    ...

def test_new_user_receives_welcome_email():
    ...

def test_overdraft_raises_insufficient_funds():
    ...
```

## 10. Performance Awareness

### Know Your Data Structures

| Operation | List | Set | Dict |
|-----------|------|-----|------|
| Lookup | O(n) | O(1) | O(1) |
| Insert | O(1)* | O(1) | O(1) |
| Delete | O(n) | O(1) | O(1) |

Choose the right structure for the job.

### Avoid Premature Optimization

Make it work, make it right, then make it fast — only if profiling shows a bottleneck.

### Profile Before Optimizing

Use profiling tools to find actual bottlenecks rather than guessing.

## 11. Communication Skills

### Write Clear Documentation

Document the *why*, not the *what*. Code shows what; comments and docs explain why.

### Ask Good Questions

When stuck:
1. Describe what you're trying to do
2. Show what you've tried
3. Share the error or unexpected result
4. Include relevant code snippets

### Share Knowledge

Write things down. If you learned something solving a problem, document it for your future self and your team.
