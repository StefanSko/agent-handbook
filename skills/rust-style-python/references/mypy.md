# Mypy Configuration and Common Issues

## Recommended mypy Configuration

Create a `mypy.ini` or `pyproject.toml` file:

### mypy.ini
```ini
[mypy]
python_version = 3.11
warn_return_any = True
warn_unused_configs = True
disallow_untyped_defs = True
disallow_any_unimported = True
no_implicit_optional = True
warn_redundant_casts = True
warn_unused_ignores = True
warn_no_return = True
check_untyped_defs = True
strict_equality = True
```

### pyproject.toml
```toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_any_unimported = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
check_untyped_defs = true
strict_equality = true
```

## Running mypy

```bash
# Check single file
mypy file.py

# Check with strict mode
mypy --strict file.py

# Check entire package
mypy src/

# Check and show error codes
mypy --show-error-codes file.py

# Generate HTML report
mypy --html-report ./mypy-report file.py
```

## Common Type Errors and Solutions

### Error: Function is missing a return type annotation

```python
# Bad
def calculate_total(items):
    return sum(items)

# Good
def calculate_total(items: list[float]) -> float:
    return sum(items)
```

### Error: Need type annotation for variable

```python
# Bad
items = []  # type unknown

# Good
items: list[str] = []
# or
items = []  # type: list[str]
```

### Error: Incompatible types in assignment

```python
# Bad
def get_user_id() -> int | None:
    return None

user_id: int = get_user_id()  # Error!

# Good
user_id: int | None = get_user_id()
if user_id is not None:
    # user_id is narrowed to int here
    process_user(user_id)
```

### Error: Cannot call function of unknown type

```python
# Bad - callable type unknown
def apply_function(fn, value):
    return fn(value)

# Good
from typing import Callable, TypeVar

T = TypeVar("T")
U = TypeVar("U")

def apply_function(fn: Callable[[T], U], value: T) -> U:
    return fn(value)
```

### Error: Incompatible return value type

```python
# Bad
def find_user(user_id: int) -> User:
    user = database.get(user_id)
    return user  # Error: might be None

# Good - option 1: change return type
def find_user(user_id: int) -> User | None:
    return database.get(user_id)

# Good - option 2: raise exception if not found
def find_user(user_id: int) -> User:
    user = database.get(user_id)
    if user is None:
        raise ValueError(f"User {user_id} not found")
    return user
```

### Error: Argument has incompatible type

```python
# Bad
UserId = NewType("UserId", int)
ProductId = NewType("ProductId", int)

def get_user(user_id: UserId) -> User: ...

product_id = ProductId(42)
get_user(product_id)  # Error!

# Good
user_id = UserId(42)
get_user(user_id)
```

## Working with Third-Party Libraries

### Install type stubs

```bash
# Install type stubs for libraries
pip install types-requests
pip install types-redis
pip install types-PyYAML

# For Django
pip install django-stubs

# For numpy, pandas (built-in stubs)
pip install numpy pandas
```

### Ignore missing stubs selectively

```python
import obscure_library  # type: ignore[import-untyped]

# Or in mypy.ini
[mypy-obscure_library.*]
ignore_missing_imports = True
```

## Advanced Type Checking

### Type narrowing with isinstance

```python
def process(value: int | str) -> str:
    if isinstance(value, int):
        # value is narrowed to int here
        return str(value * 2)
    else:
        # value is narrowed to str here
        return value.upper()
```

### Type guards

```python
from typing import TypeGuard

def is_string_list(val: list[object]) -> TypeGuard[list[str]]:
    return all(isinstance(x, str) for x in val)

def process(items: list[object]) -> None:
    if is_string_list(items):
        # items is narrowed to list[str] here
        for item in items:
            print(item.upper())
```

### Literal types

```python
from typing import Literal

def set_log_level(level: Literal["debug", "info", "warning", "error"]) -> None:
    ...

set_log_level("debug")  # OK
set_log_level("trace")  # Error: invalid literal
```

### Protocol types (structural subtyping)

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

def render(shape: Drawable) -> None:
    shape.draw()

# Both work without explicit inheritance
render(Circle())
render(Square())
```

## Best Practices

1. **Start with `--strict` mode** - It's easier to maintain from the start
2. **Add types incrementally** - Use `# type: ignore` temporarily, then fix
3. **Use `reveal_type()` for debugging** - Shows inferred types during development
4. **Run mypy in CI/CD** - Prevent type errors from reaching production
5. **Keep mypy up to date** - New versions have better inference

## Common Patterns to Avoid

### Don't use `Any` unnecessarily

```python
# Bad
def process(data: Any) -> Any:
    return data.process()

# Good
from typing import Protocol

class Processable(Protocol):
    def process(self) -> str: ...

def process(data: Processable) -> str:
    return data.process()
```

### Don't ignore types silently

```python
# Bad
result = complicated_function()  # type: ignore

# Good - explain why
result = complicated_function()  # type: ignore[misc]  # third-party lib has no stubs
```

### Don't use mutable default arguments without proper typing

```python
# Bad
def append_item(item: str, items: list[str] = []) -> list[str]:
    items.append(item)
    return items

# Good
def append_item(item: str, items: list[str] | None = None) -> list[str]:
    if items is None:
        items = []
    items.append(item)
    return items
```
