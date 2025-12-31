# Advanced Patterns

This file contains detailed examples of advanced Rust-inspired Python patterns.

## Strongly-Typed Wrappers

Use separate types to distinguish between different representations of the same concept:

```python
from dataclasses import dataclass

@dataclass
class BBoxBase:
    left: float
    top: float
    width: float
    height: float

@dataclass
class NormalizedBBox(BBoxBase):
    """Bounding box with coordinates in [0.0, 1.0] range"""
    
    def __post_init__(self):
        # Runtime validation
        assert 0.0 <= self.left <= 1.0, f"left must be in [0, 1], got {self.left}"
        assert 0.0 <= self.top <= 1.0, f"top must be in [0, 1], got {self.top}"
        assert 0.0 <= self.width <= 1.0, f"width must be in [0, 1], got {self.width}"
        assert 0.0 <= self.height <= 1.0, f"height must be in [0, 1], got {self.height}"
    
    def denormalize(self, img_width: int, img_height: int) -> "DenormalizedBBox":
        return DenormalizedBBox(
            left=self.left * img_width,
            top=self.top * img_height,
            width=self.width * img_width,
            height=self.height * img_height
        )

@dataclass
class DenormalizedBBox(BBoxBase):
    """Bounding box with coordinates in pixel space"""
    
    def normalize(self, img_width: int, img_height: int) -> "NormalizedBBox":
        return NormalizedBBox(
            left=self.left / img_width,
            top=self.top / img_height,
            width=self.width / img_width,
            height=self.height / img_height
        )

# Union type for generic handling
BBox = NormalizedBBox | DenormalizedBBox

# Conversion interface for working with either type
def process_bbox(bbox: BBox, img_width: int, img_height: int) -> DenormalizedBBox:
    """Always work with denormalized internally"""
    if isinstance(bbox, NormalizedBBox):
        return bbox.denormalize(img_width, img_height)
    return bbox
```

Benefits:
- Impossible to accidentally mix normalized and denormalized bboxes
- Runtime validation in `__post_init__`
- Clear conversion methods
- Type-safe generic handling

## Using `typing.Self`

For fluent interfaces and method chaining (Python 3.11+):

```python
from typing import Self
from dataclasses import dataclass

@dataclass
class BBoxBase:
    left: float
    top: float
    width: float
    height: float
    
    def move(self, dx: float, dy: float) -> Self:
        """Returns the same concrete type as the instance"""
        self.left += dx
        self.top += dy
        return self
    
    def scale(self, factor: float) -> Self:
        self.width *= factor
        self.height *= factor
        return self

class NormalizedBBox(BBoxBase):
    pass

class DenormalizedBBox(BBoxBase):
    pass

# Type of bbox2 is NormalizedBBox, not just BBoxBase
bbox = NormalizedBBox(0.1, 0.2, 0.3, 0.4)
bbox2 = bbox.move(0.1, 0.1).scale(1.5)
```

## Safe Mutex Pattern

Implement a mutex that makes it impossible to access protected data without locking:

```python
import contextlib
from threading import Lock
from typing import ContextManager, Generic, TypeVar

T = TypeVar("T")

class Mutex(Generic[T]):
    """Thread-safe mutex that protects data inside it"""
    
    def __init__(self, value: T):
        # Use double underscore to make it harder to accidentally access
        self.__value = value
        self.__lock = Lock()
    
    @contextlib.contextmanager
    def lock(self) -> ContextManager[T]:
        """Lock mutex and provide access to protected data"""
        self.__lock.acquire()
        try:
            yield self.__value
        finally:
            self.__lock.release()

# Usage
counter = Mutex[int](0)

def increment():
    with counter.lock() as value:
        # value is typed as int here
        value += 1  # Note: for primitives, need to return and reassign
        # Better pattern: use mutable container or object

# Better example with mutable object
from dataclasses import dataclass

@dataclass
class Counter:
    value: int = 0
    
    def increment(self):
        self.value += 1

counter = Mutex(Counter())

def increment():
    with counter.lock() as c:
        c.increment()  # Modifies object in-place
```

Benefits:
- Can't access data without locking
- Automatic unlock via context manager
- Type-safe (Generic[T] provides correct type for locked data)

## Runtime Invariant Checking

Use `__post_init__` to validate dataclass invariants:

```python
from dataclasses import dataclass

@dataclass
class Temperature:
    kelvin: float
    
    def __post_init__(self):
        if self.kelvin < 0:
            raise ValueError(f"Temperature cannot be negative: {self.kelvin}K")
    
    @staticmethod
    def from_celsius(celsius: float) -> "Temperature":
        return Temperature(kelvin=celsius + 273.15)
    
    @staticmethod
    def from_fahrenheit(fahrenheit: float) -> "Temperature":
        celsius = (fahrenheit - 32) * 5/9
        return Temperature.from_celsius(celsius)

@dataclass
class PositiveInt:
    value: int
    
    def __post_init__(self):
        if self.value <= 0:
            raise ValueError(f"Value must be positive: {self.value}")

@dataclass
class EmailAddress:
    address: str
    
    def __post_init__(self):
        if "@" not in self.address:
            raise ValueError(f"Invalid email address: {self.address}")
```

## Generic Dataclasses

Create generic dataclasses for reusable container types:

```python
from dataclasses import dataclass
from typing import Generic, TypeVar

T = TypeVar("T")

@dataclass
class Result(Generic[T]):
    """Rust-like Result type"""
    value: T | None
    error: str | None
    
    def __post_init__(self):
        # Invariant: exactly one of value or error must be set
        if (self.value is None) == (self.error is None):
            raise ValueError("Result must have either value or error, not both or neither")
    
    @staticmethod
    def ok(value: T) -> "Result[T]":
        return Result(value=value, error=None)
    
    @staticmethod
    def err(error: str) -> "Result[T]":
        return Result(value=None, error=error)
    
    def is_ok(self) -> bool:
        return self.error is None
    
    def is_err(self) -> bool:
        return self.error is not None
    
    def unwrap(self) -> T:
        if self.error:
            raise ValueError(f"Called unwrap on error: {self.error}")
        assert self.value is not None
        return self.value

# Usage
def divide(a: float, b: float) -> Result[float]:
    if b == 0:
        return Result.err("Division by zero")
    return Result.ok(a / b)

result = divide(10, 2)
if result.is_ok():
    print(f"Result: {result.unwrap()}")
else:
    print(f"Error: {result.error}")
```

## Serialization with Union Types

Use pyserde for automatic (de)serialization of union types:

```python
from dataclasses import dataclass
from serde import serde, to_dict, from_dict
from serde.json import to_json, from_json

@serde
@dataclass
class ConfigV1:
    host: str
    port: int

@serde
@dataclass  
class ConfigV2:
    host: str
    port: int
    timeout: int

@serde
@dataclass
class ConfigV3:
    host: str
    port: int
    timeout: int
    retries: int

# Union type for versioned config
Config = ConfigV1 | ConfigV2 | ConfigV3

# Serialize
config = ConfigV3(host="localhost", port=8080, timeout=30, retries=3)
json_str = to_json(config)
# {"ConfigV3": {"host": "localhost", "port": 8080, "timeout": 30, "retries": 3}}

# Deserialize - automatically determines variant
loaded_config = from_json(Config, json_str)
# ConfigV3(host='localhost', port=8080, timeout=30, retries=3)
```

Benefits:
- Automatic version detection
- Type-safe deserialization
- Backwards compatibility by supporting all versions
- No manual tagging or dispatch logic needed

## Context Manager Pattern

Use context managers to prevent misuse of stateful resources:

```python
from contextlib import contextmanager
from typing import Iterator

class DatabaseConnection:
    def __init__(self, connection_string: str):
        self._connection_string = connection_string
        self._connected = False
    
    def _connect(self):
        # Actual connection logic
        self._connected = True
    
    def _disconnect(self):
        # Cleanup logic
        self._connected = False
    
    def query(self, sql: str) -> list[dict]:
        if not self._connected:
            raise RuntimeError("Not connected")
        # Execute query
        return []

@contextmanager
def connect_database(connection_string: str) -> Iterator[DatabaseConnection]:
    """Context manager for database connections"""
    conn = DatabaseConnection(connection_string)
    conn._connect()
    try:
        yield conn
    finally:
        conn._disconnect()

# Usage - impossible to forget to disconnect
with connect_database("postgresql://...") as db:
    results = db.query("SELECT * FROM users")
    # Process results
# Automatically disconnected here
```
