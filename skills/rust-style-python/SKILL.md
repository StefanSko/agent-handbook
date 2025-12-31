---
name: rust-style-python
description: Write Python code with Rust-inspired best practices including comprehensive type hints, dataclasses, algebraic data types, newtypes, and sound API design. Use mypy for type checking and modern Python typing with builtins (list, dict, set, tuple instead of List, Dict, Set, Tuple). Apply the "making illegal states unrepresentable" principle.
---

# Rust-Style Python

Write Python code with strong typing, sound API design, and Rust-inspired patterns to prevent bugs at "compile time" (type checking) rather than runtime.

## Core Principles

1. **Type hints everywhere** - All function signatures and class attributes must have type hints
2. **Modern typing syntax** - Use builtin types (list, dict, set, tuple) instead of typing module equivalents
3. **Always run mypy** - Type check code before considering it complete
4. **Make illegal states unrepresentable** - Use the type system to prevent misuse

## Required Practices

### Type Hints

Always add complete type hints to function signatures:

```python
# Bad - opaque interface
def find_item(records, check):
    ...

# Good - clear interface
def find_item(
    records: list[Item],
    check: Callable[[Item], bool]
) -> Item | None:
    ...
```

Use modern builtin types (Python 3.9+):
- `list[T]` not `List[T]`
- `dict[K, V]` not `Dict[K, V]`
- `set[T]` not `Set[T]`
- `tuple[T, ...]` not `Tuple[T, ...]`
- `X | Y` not `Union[X, Y]`
- `X | None` not `Optional[X]`

### Dataclasses Over Tuples and Dicts

Never return tuples or dicts for structured data. Always use dataclasses:

```python
from dataclasses import dataclass

# Bad - opaque return type
def find_person(...) -> tuple[str, str, int]:
    return (name, city, age)

# Bad - untyped dictionary
def find_person(...) -> dict[str, Any]:
    return {"name": ..., "city": ..., "age": ...}

# Good - explicit dataclass
@dataclass
class Person:
    name: str
    city: str
    age: int

def find_person(...) -> Person:
    return Person(name=..., city=..., age=...)
```

### Algebraic Data Types (Tagged Unions)

Use union types with dataclasses to model variants:

```python
from dataclasses import dataclass

@dataclass
class Header:
    protocol: str
    size: int

@dataclass
class Payload:
    data: bytes

@dataclass
class Trailer:
    data: bytes
    checksum: int

# Union type (ADT)
Packet = Header | Payload | Trailer

def handle_packet(packet: Packet) -> None:
    match packet:
        case Header(protocol, size):
            print(f"header {protocol} {size}")
        case Payload(data):
            print(f"payload {data}")
        case Trailer(data, checksum):
            print(f"trailer {checksum} {data}")
        case _:
            raise ValueError(f"Unknown packet type: {type(packet)}")
```

Use `typing.assert_never` (Python 3.11+) for exhaustiveness checking:

```python
from typing import assert_never

def handle_packet(packet: Packet) -> None:
    match packet:
        case Header(): ...
        case Payload(): ...
        case Trailer(): ...
        case _:
            assert_never(packet)  # Type error if case is missing
```

### NewType for Domain-Specific Types

Prevent mixing semantically different values with the same underlying type:

```python
from typing import NewType

# Define semantic types
UserId = NewType("UserId", int)
ProductId = NewType("ProductId", int)

def get_user(user_id: UserId) -> User: ...
def get_product(product_id: ProductId) -> Product: ...

user_id = UserId(42)
product_id = ProductId(100)

# Type error - prevents mixing IDs
get_user(product_id)  # mypy catches this!
```

Use for IDs, measurements, and other domain concepts that shouldn't be mixed.

### Construction Functions

Prefer static methods with descriptive names over overloaded `__init__`:

```python
@dataclass
class Rectangle:
    x: float
    y: float
    width: float
    height: float
    
    @staticmethod
    def from_corners(x1: float, y1: float, x2: float, y2: float) -> "Rectangle":
        return Rectangle(x=x1, y=y1, width=x2-x1, height=y2-y1)
    
    @staticmethod
    def from_center(cx: float, cy: float, width: float, height: float) -> "Rectangle":
        return Rectangle(x=cx-width/2, y=cy-height/2, width=width, height=height)
```

### Typestate Pattern

Encode state transitions in the type system to prevent invalid operations:

```python
# Bad - documentation-based constraints
class Client:
    """
    Rules:
    - Call connect() before authenticate()
    - Don't call send_message() before authenticate()
    - Don't call methods after close()
    """
    def connect(self): ...
    def authenticate(self): ...
    def send_message(self): ...
    def close(self): ...

# Good - impossible to misuse
class ConnectedClient:
    def authenticate(self, password: str) -> "AuthenticatedClient | None": ...
    def close(self) -> None: ...

class AuthenticatedClient:
    def send_message(self, msg: str) -> None: ...
    def close(self) -> None: ...

def connect(address: str) -> ConnectedClient | None:
    """Only way to get a client"""
    ...

# Usage
client = connect("localhost")
if client:
    auth_client = client.authenticate("password")
    if auth_client:
        auth_client.send_message("hello")  # Can't call before auth!
```

## Type Checking Workflow

After writing or modifying code, always run mypy:

```bash
mypy --strict <file.py>
```

Address all type errors before considering the code complete. Use `# type: ignore` only as a last resort with a comment explaining why.

## Common Patterns

See `references/patterns.md` for detailed examples of:
- Strongly-typed wrappers (normalized vs denormalized data)
- Safe mutex patterns
- Runtime invariant checking with `__post_init__`
- Using `typing.Self` for fluent interfaces
- Generic dataclasses

## Key Reminders

- Type hints are NOT optional - they're part of the code
- Use dataclasses for any structured data with multiple fields
- Model mutually exclusive states as separate types
- Use NewType liberally for domain concepts
- Run mypy before considering code done
- Prefer compile-time (type-check-time) errors over runtime errors
