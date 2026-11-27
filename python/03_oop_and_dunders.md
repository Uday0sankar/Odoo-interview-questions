# Python OOP, Inheritance, MRO & Dunder Methods

Core Python interview questions covering classes, method types, multiple inheritance, MRO, magic methods, `@property`, and `@dataclass`.

---

### 1. What is the difference between Class Attributes and Instance Attributes?

- **Class attributes** are defined directly on the class body and shared across all instances unless shadowed on an instance.
- **Instance attributes** are usually assigned inside `__init__` on `self` and belong exclusively to that instance.
- **Pitfall:** Mutating a mutable class attribute (like a `list`) via `self.items.append(...)` mutates it for **all** instances; rebinding `self.items = [...]` creates an instance attribute instead.

```python
class Order:
    tax_rate = 0.18  # Class attribute (immutable float -> safe to share)

    def __init__(self, lines=None):
        self.lines = list(lines or [])  # Instance attribute (unique per Order)
```

---

### 2. Compare Instance Methods, `@classmethod`, and `@staticmethod`

| Decorator | First Parameter | Access | Common Use Case |
| :--- | :--- | :--- | :--- |
| None (Instance) | `self` | Instance & class state | Core object behavior |
| `@classmethod` | `cls` | Class state only | Alternative constructors / factory methods |
| `@staticmethod` | None | Neither `self` nor `cls` | Utility/helper logically grouped with the class |

```python
class User:
    def __init__(self, username, role="user"):
        self.username = username
        self.role = role

    @classmethod
    def from_email(cls, email):
        return cls(username=email.split("@")[0])

    @staticmethod
    def is_valid_username(name):
        return name.isalnum() and len(name) >= 3
```

---

### 3. How do Multiple Inheritance, MRO, and `super()` work?

Python resolves method lookups in inheritance hierarchies using **MRO (Method Resolution Order)** computed via **C3 linearization**:
- Inspect it with `ClassName.__mro__` or `ClassName.mro()`.
- `super()` does **not** simply call "the parent class"; it delegates to the **next class in `self`'s MRO**, enabling cooperative multiple inheritance (e.g., in the diamond problem and in Odoo's `super()` model extension chain) without calling a base class twice.

```python
class Base:
    def greet(self):
        return ["Base"]

class Left(Base):
    def greet(self):
        return ["Left"] + super().greet()

class Right(Base):
    def greet(self):
        return ["Right"] + super().greet()

class Child(Left, Right):
    def greet(self):
        return ["Child"] + super().greet()

print([c.__name__ for c in Child.__mro__])
# ['Child', 'Left', 'Right', 'Base', 'object']
print(Child().greet())
# ['Child', 'Left', 'Right', 'Base']
```

---

### 4. Differentiate `__new__` and `__init__`

- **`__new__(cls, *args, **kwargs)`** is a static method that **creates and returns** the new instance. You override it when subclassing immutable types (`int`, `str`, `tuple`) or implementing patterns like singletons.
- **`__init__(self, *args, **kwargs)`** is an instance method that **initializes** the already-created instance and must return `None`.

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

---

### 5. Explain Key Dunder (Magic) Methods

- **`__repr__` vs `__str__`:** `__repr__` targets developers (unambiguous, ideally valid Python to recreate the object; fallback if `__str__` is missing). `__str__` targets end users (readable display via `str()` and `print()`).
- **`__call__`:** Makes an instance callable like a function (`obj()`).
- **`__eq__` & `__hash__`:** Defining `__eq__` on a custom class sets `__hash__ = None` (making instances unhashable) unless you also implement `__hash__`.
- **`__enter__` & `__exit__`:** Implement the context manager protocol (`with obj:`).

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __repr__(self):
        return f"Multiplier(factor={self.factor!r})"

    def __call__(self, value):
        return value * self.factor

double = Multiplier(2)
print(repr(double))  # Multiplier(factor=2)
print(double(7))     # 14
```

---

### 6. How does `@property` work?

`@property` exposes methods as managed attributes, letting you add validation, lazy calculation, or read-only protection without breaking existing attribute-access syntax (`obj.price`).

```python
class Product:
    def __init__(self, price):
        self._price = price

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("Price cannot be negative")
        self._price = value
```

---

### 7. What are `@dataclass` and `__slots__`?

- **`@dataclass`** (Python 3.7+) automatically generates `__init__`, `__repr__`, `__eq__`, and optionally ordering/immutability (`frozen=True`) from type annotations, eliminating boilerplate.
- **`__slots__`** replaces each instance's dynamic `__dict__` with a fixed tuple-like descriptor layout, reducing per-instance memory overhead and preventing accidental creation of arbitrary attributes.

```python
from dataclasses import dataclass, field

@dataclass(frozen=True, slots=True)
class Point:
    x: float
    y: float
    tags: tuple[str, ...] = field(default_factory=tuple)
```
