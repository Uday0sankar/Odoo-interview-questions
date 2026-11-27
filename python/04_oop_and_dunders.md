# Python OOP, Inheritance, MRO & Dunder Methods

Core Python interview questions covering classes, method types, multiple inheritance, MRO, magic methods, `@property`, and `@dataclass`.

---

### 1. What is the difference between Class Attributes and Instance Attributes?

- **Class attributes** live on the class itself and are shared by every instance unless an instance shadows the name.
- **Instance attributes** are usually set inside `__init__` on `self` and belong only to that specific object.
- **Classic gotcha:** Mutating a mutable class attribute (like a `list`) via `self.items.append(...)` edits it for **all** instances; reassigning `self.items = [...]` creates an instance attribute instead.

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
| `@staticmethod` | None | Neither `self` nor `cls` | Utility helper logically grouped with the class |

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

Python resolves method lookups in inheritance trees using **MRO (Method Resolution Order)** via **C3 linearization**:
- Check the order anytime with `ClassName.__mro__` or `ClassName.mro()`.
- `super()` does **not** just mean "call my parent class"—it delegates to the **next class in `self`'s MRO**, which makes cooperative multiple inheritance work cleanly (including in Odoo's model inheritance chain) without calling a base class twice.

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

- **`__new__(cls, *args, **kwargs)`** is a static method that actually **creates and returns** the new object instance. You override it when subclassing immutable types (`int`, `str`, `tuple`) or implementing patterns like singletons.
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

- **`__repr__` vs. `__str__`:** `__repr__` is for developers (unambiguous, ideally valid Python to recreate the object; also acts as fallback when `__str__` isn't defined). `__str__` is for end users (`str()`, `print()`).
- **`__call__`:** Lets an instance be called like a regular function (`obj()`).
- **`__eq__` & `__hash__`:** Defining `__eq__` on a class automatically sets `__hash__ = None` (making instances unhashable) unless you also implement `__hash__`.
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

`@property` exposes methods as managed attributes, letting you add validation, lazy computation, or read-only protection without changing attribute-access syntax (`obj.price`).

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

- **`@dataclass`** (Python 3.7+) auto-generates `__init__`, `__repr__`, `__eq__`, and optionally ordering or immutability (`frozen=True`) from type annotations, cutting down boilerplate.
- **`__slots__`** replaces each instance's dynamic `__dict__` with a fixed attribute layout, saving memory on huge object counts and blocking accidental typo attributes.

```python
from dataclasses import dataclass, field

@dataclass(frozen=True, slots=True)
class Point:
    x: float
    y: float
    tags: tuple[str, ...] = field(default_factory=tuple)
```
