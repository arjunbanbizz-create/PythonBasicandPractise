# Google Python Style Guide — Learn by Example

> A progressive, example-driven guide based on the [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html).  
> Each section includes a **concept**, **❌ bad** and **✅ good** examples, and a **takeaway**.

---

## 🟢 Beginner

---

### 1. Naming Conventions

**Outcome:** Write names that are immediately understandable to any reader.

| Type | Convention | Example |
|------|-----------|---------|
| Variables & functions | `snake_case` | `user_name`, `get_total()` |
| Classes | `PascalCase` | `UserProfile` |
| Constants | `ALL_CAPS` | `MAX_RETRIES = 3` |
| Private/internal | `_underscore` prefix | `_internal_cache` |

```python
# ❌ Bad
def CalcT(x, y):
    RESULT = x + y
    return RESULT

# ✅ Good
def calculate_total(price: float, tax: float) -> float:
    result = price + tax
    return result
```

**Takeaway:** Names should read like English prose. Avoid abbreviations unless they are universally understood.

---

### 2. Imports

**Outcome:** Keep imports organised, unambiguous, and conflict-free.

```python
# ❌ Bad — ambiguous, may import wrong module
import jodie
from os.path import *

# ✅ Good — explicit, ordered: stdlib → third-party → local
import os
import sys

import numpy as np
from absl import flags

from myproject.utils import helpers
```

**Rules:**
- `import x` for packages/modules.
- `from x import y` when you want a short name.
- `from x import y as z` only to resolve name conflicts or use standard abbreviations (`np`, `pd`).
- Never use wildcard imports (`from x import *`).

**Takeaway:** Every name should have an obvious origin. Use full package paths.

---

### 3. Line Length & Formatting

**Outcome:** Code that fits on screen and is easy to scan.

- Maximum line length: **80 characters**.
- Use parentheses `()` for line continuation — never backslash `\`.
- 4 spaces per indent level. No tabs.

```python
# ❌ Bad
result = some_function(argument_one, argument_two, argument_three, argument_four, argument_five)

# ✅ Good
result = some_function(
    argument_one,
    argument_two,
    argument_three,
    argument_four,
)
```

**Takeaway:** Wrap long expressions with parentheses and align arguments for readability.

---

### 4. True/False Evaluations

**Outcome:** Write clean, Pythonic boolean checks.

```python
# ❌ Bad
if len(my_list) == 0:
    pass
if my_var == False:
    pass
if my_var == None:
    pass

# ✅ Good
if not my_list:          # empty sequences are falsy
    pass
if not my_var:           # False is falsy
    pass
if my_var is None:       # always use 'is' for None checks
    pass
```

**Takeaway:** Use implicit truthiness for sequences and objects. Use `is None` / `is not None` for explicit None checks.

---

### 5. Comments & Inline Notes

**Outcome:** Write comments that explain *why*, not *what*.

```python
# ❌ Bad — restates the code
x = x + 1  # increment x by 1

# ✅ Good — explains the reason
x = x + 1  # compensate for the border pixel offset

# ✅ Good — TODO with owner and context
# TODO(jane): Replace with vectorised lookup once dataset > 10k rows.
```

**Takeaway:** Comments add context the code can't express. Use `# TODO(name):` for deferred work.

---

## 🔵 Intermediate

---

### 6. Docstrings

**Outcome:** Every public function, class, and module is self-documenting.

```python
# ❌ Bad — no docstring
def connect(minimum):
    if minimum < 1024:
        raise ValueError("too low")
    return _find_port(minimum)


# ✅ Good — Google-style docstring
def connect_to_next_port(minimum: int) -> int:
    """Connects to the next available port at or above the minimum.

    Args:
        minimum: A port value greater than or equal to 1024.

    Returns:
        The port number that was successfully connected.

    Raises:
        ValueError: If minimum is below 1024.
        ConnectionError: If no available port is found.
    """
    if minimum < 1024:
        raise ValueError(f"Min. port must be at least 1024, not {minimum}.")
    port = _find_next_open_port(minimum)
    if port is None:
        raise ConnectionError(f"No port available at or above {minimum}.")
    return port
```

**Takeaway:** Use `Args:`, `Returns:`, and `Raises:` sections. One-line docstrings are fine for trivial functions.

---

### 7. Exceptions

**Outcome:** Raise specific errors; catch only what you can handle.

```python
# ❌ Bad — swallows all errors silently
try:
    result = risky_operation()
except:
    pass

# ✅ Good — specific, minimal try block, meaningful error
try:
    result = risky_operation()
except TimeoutError as e:
    logging.warning("Operation timed out: %s", e)
    raise

# ✅ Good — use ValueError for bad input
def set_age(age: int) -> None:
    if age < 0:
        raise ValueError(f"Age cannot be negative, got {age}.")
```

**Rules:**
- Never use bare `except:` — it catches `Ctrl+C`, `SystemExit`, and more.
- Catch `Exception` only to log and re-raise.
- Keep `try` blocks as small as possible.

**Takeaway:** Be precise about what can fail and why. Let unexpected errors bubble up.

---

### 8. Default Argument Values

**Outcome:** Avoid the classic mutable default argument bug.

```python
# ❌ Bad — the list is shared across all calls!
def append_item(item, collection=[]):
    collection.append(item)
    return collection

# ✅ Good — use None as sentinel
def append_item(item: str, collection: list | None = None) -> list:
    if collection is None:
        collection = []
    collection.append(item)
    return collection
```

**Takeaway:** Default arguments are evaluated *once* at module load. Use `None` as a default for any mutable type.

---

### 9. Comprehensions & Generators

**Outcome:** Write concise data transformations that stay readable.

```python
# ❌ Bad — nested comprehension is hard to follow
pairs = [(x, y) for x in range(10) for y in range(5) if x * y > 10]

# ✅ Good — keep it to one for-clause and one condition
squares = [x**2 for x in range(10) if x % 2 == 0]

# ✅ Good — use a regular loop for complex multi-clause logic
pairs = []
for x in range(10):
    for y in range(5):
        if x * y > 10:
            pairs.append((x, y))

# ✅ Good — generator expression avoids building a full list
total = sum(x**2 for x in range(1_000_000))
```

**Takeaway:** One `for` clause, one optional `if`. Go beyond that → use a loop or a function.

---

### 10. Default Iterators

**Outcome:** Iterate Pythonically — no unnecessary method calls.

```python
# ❌ Bad
for key in my_dict.keys():
    print(key)

for line in my_file.readlines():
    print(line)

# ✅ Good
for key in my_dict:
    print(key)

for line in my_file:
    print(line)

for key, value in my_dict.items():
    print(key, value)
```

**Takeaway:** Built-in container types have efficient default iterators. Use them directly.

---

### 11. Strings

**Outcome:** Format strings safely and consistently.

```python
# ❌ Bad — use + for concatenation in loops (creates many objects)
result = ""
for part in parts:
    result += part

# ✅ Good — join is efficient
result = "".join(parts)

# ✅ Good — f-strings for readability
name = "Alice"
greeting = f"Hello, {name}! You have {len(messages)} messages."

# ✅ Good — % formatting for logging (lazy evaluation)
logging.info("User %s logged in from %s", user_id, ip_address)
```

**Takeaway:** Use f-strings for display strings. Use `%`-formatting for logging. Use `"".join()` for building strings in loops.

---

### 12. Files and Resources

**Outcome:** Always release resources, even when errors occur.

```python
# ❌ Bad — file may never be closed if an error occurs
f = open("data.txt")
data = f.read()
f.close()

# ✅ Good — context manager guarantees cleanup
with open("data.txt", encoding="utf-8") as f:
    data = f.read()
```

**Takeaway:** Always use `with` for files, sockets, database connections, and any resource that must be released.

---

## 🔴 Advanced

---

### 13. Type Annotations

**Outcome:** Catch type errors before runtime; improve IDE support and readability.

```python
# ❌ Bad — no type information
def process(items, threshold):
    return [x for x in items if x > threshold]


# ✅ Good — annotated signature
from collections.abc import Sequence

def process(items: Sequence[float], threshold: float) -> list[float]:
    return [x for x in items if x > threshold]


# ✅ Good — Optional / union types
def find_user(user_id: int) -> str | None:
    ...


# ✅ Good — type aliases for complex types
from typing import TypeAlias
Vector: TypeAlias = list[float]

def magnitude(v: Vector) -> float:
    return sum(x**2 for x in v) ** 0.5
```

**Takeaway:** Annotate all public functions. Use `X | None` (Python 3.10+) instead of `Optional[X]`.

---

### 14. Properties

**Outcome:** Expose computed attributes naturally without breaking the public API.

```python
# ❌ Bad — unnecessary property for a simple attribute
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        self._radius = value  # no logic — just make it public!


# ✅ Good — property used for derived/validated data
class Circle:
    def __init__(self, radius: float) -> None:
        if radius < 0:
            raise ValueError("Radius must be non-negative.")
        self.radius = radius  # public attribute, no property needed

    @property
    def area(self) -> float:
        """Computed on access; read-only."""
        return 3.14159 * self.radius ** 2
```

**Takeaway:** Use `@property` for computed or validated attributes. If there's no logic, just use a plain attribute.

---

### 15. Generators

**Outcome:** Process large sequences without loading them into memory.

```python
# ❌ Bad — builds full list in memory
def read_records(filename: str) -> list[dict]:
    results = []
    with open(filename) as f:
        for line in f:
            results.append(parse(line))
    return results


# ✅ Good — yields one record at a time
def read_records(filename: str):
    """Yields parsed records from a file.

    Yields:
        Parsed record dict for each line in the file.
    """
    with open(filename) as f:
        for line in f:
            yield parse(line)

# Usage
for record in read_records("large_file.txt"):
    process(record)
```

**Takeaway:** Use generators for large or infinite sequences. Use `Yields:` in the docstring instead of `Returns:`.

---

### 16. Lambda Functions

**Outcome:** Use lambdas for short, obvious operations — use named functions for anything else.

```python
# ❌ Bad — lambda that's hard to read
transform = lambda x: x['score'] * 0.8 + x['bonus'] * 1.2 if x['active'] else 0

# ✅ Good — named function is clearer and testable
def calculate_weighted_score(record: dict) -> float:
    if not record['active']:
        return 0
    return record['score'] * 0.8 + record['bonus'] * 1.2

# ✅ Good — lambda fine for a simple sort key
students.sort(key=lambda s: s.grade)

# ✅ Good — use operator module for simple ops
import operator
product = reduce(operator.mul, numbers, 1)
```

**Takeaway:** Lambdas are fine for one-liners under ~60 chars. For anything more complex, use a def.

---

### 17. Avoiding Mutable Global State

**Outcome:** Keep module state predictable and testable.

```python
# ❌ Bad — mutable global modified by functions
active_users = []

def add_user(user):
    active_users.append(user)


# ✅ Good — encapsulate state in a class
class UserRegistry:
    def __init__(self) -> None:
        self._active_users: list[str] = []

    def add_user(self, user: str) -> None:
        self._active_users.append(user)

    def get_users(self) -> list[str]:
        return list(self._active_users)

# ✅ Good — module-level constants are fine
MAX_CONNECTIONS = 100
_DEFAULT_TIMEOUT = 30  # internal constant
```

**Takeaway:** Constants (ALL_CAPS) at module level are fine. Mutable state belongs in a class or is passed explicitly.

---

### 18. The `main` Guard

**Outcome:** Make modules both importable and executable.

```python
# ❌ Bad — code runs on import
print("Starting...")
run_application()


# ✅ Good — guard executable code behind main()
def main() -> None:
    """Entry point for the application."""
    print("Starting...")
    run_application()


if __name__ == "__main__":
    main()
```

**Takeaway:** Always wrap script-level code in a `main()` function and gate it with `if __name__ == "__main__":`.

---

### 19. Linting with pylint

**Outcome:** Catch bugs and style issues automatically.

```bash
# Install
pip install pylint

# Run on your code
pylint my_module.py

# List all warnings
pylint --list-msgs
```

```python
# ✅ Suppress a specific false-positive with explanation
def do_PUT(self):  # WSGI method name, so pylint: disable=invalid-name
    ...

# ✅ Delete unused args explicitly
def viking_order(spam: str, beans: str, eggs: str | None = None) -> str:
    del beans, eggs  # Unused by vikings.
    return spam * 3
```

**Takeaway:** Run pylint in CI. Suppress warnings deliberately with a comment explaining why.

---

## Quick Reference Cheatsheet

| Topic | Rule |
|-------|------|
| Naming | `snake_case` functions/vars, `PascalCase` classes, `ALL_CAPS` constants |
| Imports | Explicit paths, ordered stdlib → third-party → local |
| Line length | 80 chars max; use `()` for continuation |
| Docstrings | `Args:` / `Returns:` / `Yields:` / `Raises:` sections |
| Exceptions | Specific types; minimal `try` blocks; never bare `except:` |
| Defaults | Never use mutable defaults; use `None` as sentinel |
| Comprehensions | One `for`, one `if` max |
| Booleans | `if seq:` not `if len(seq):` · `if x is None:` not `if x == None:` |
| Resources | Always use `with` for files and connections |
| Type hints | Annotate all public functions |
| Global state | Constants only; mutable state goes in classes |
| Main guard | `if __name__ == "__main__": main()` |
