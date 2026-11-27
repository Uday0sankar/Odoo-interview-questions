# Python Functions, Closures, Decorators & Generators

Core Python interview questions covering function arguments, scope rules, common pitfalls, decorators, iterators, generators, and functional utilities.

---

### 1. Explain `*args` and `**kwargs`

- `*args` collects extra **positional** arguments into a `tuple`.
- `**kwargs` collects extra **keyword** arguments into a `dict`.
- Standard parameter order in a signature is:
  `def func(pos_or_kw, /, standard, *args, kw_only, **kwargs):`
- Using `*` and `**` in a **function call** unpacks an iterable or mapping into positional/keyword arguments.

```python
def log_event(event_name, *tags, level="INFO", **metadata):
    print(event_name, tags, level, metadata)

log_event("login", "auth", "web", level="DEBUG", user_id=42)
# login ('auth', 'web') DEBUG {'user_id': 42}

args = ("auth", "api")
meta = {"ip": "10.0.0.1"}
log_event("logout", *args, **meta)
```

---

### 2. Why are Mutable Default Arguments dangerous, and how do you avoid them?

Default argument values in Python are evaluated **once at function definition time**, not on each call. If a mutable object (like `[]` or `{}`) is used as a default and mutated inside the function, every subsequent call shares that mutated object.

**Fix:** Default to `None` and initialize inside the function body.

```python
# Bad: shared list across calls
def add_item_buggy(item, items=[]):
    items.append(item)
    return items

# Good: fresh list per call
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

---

### 3. Explain Closures and Python's LEGB Scope Rule

Python resolves variable names using **LEGB**:
1. **L (Local):** Inside the current function.
2. **E (Enclosing):** In any enclosing (outer) functions.
3. **G (Global):** At module top level.
4. **B (Built-in):** Python built-ins (`len`, `int`, etc.).

A **closure** occurs when an inner function remembers variables from its enclosing scope even after the outer function has finished executing. Use `nonlocal` to rebind an enclosing variable and `global` to rebind a module-level variable.

```python
def make_multiplier(factor):
    def multiply(value):
        return value * factor  # 'factor' is captured from enclosing scope
    return multiply

times_3 = make_multiplier(3)
print(times_3(10))  # 30
```

---

### 4. What is the Late Binding Pitfall in Closures?

In a loop that creates functions/lambdas, closures look up the loop variable **when called**, not when created—so every closure sees the final value of the loop variable.

**Fix:** Bind the current value at creation time via a default argument (`i=i`) or `functools.partial`.

```python
# Buggy: all lambdas return 2
funcs_buggy = [lambda: i for i in range(3)]
print([f() for f in funcs_buggy])  # [2, 2, 2]

# Fixed: default argument captures current value
funcs_fixed = [lambda i=i: i for i in range(3)]
print([f() for f in funcs_fixed])  # [0, 1, 2]
```

---

### 5. How do Decorators work, and why is `@functools.wraps` important?

A decorator is a callable that takes a function, wraps or extends its behavior, and returns a new callable.

Without `@functools.wraps(func)`, the wrapper replaces the original function's `__name__`, `__doc__`, and signature metadata, breaking introspection, debugging, and tools like `sphinx` or `pytest`.

```python
import functools
import time

def timed(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        try:
            return func(*args, **kwargs)
        finally:
            elapsed = time.perf_counter() - start
            print(f"{func.__name__} took {elapsed:.4f}s")
    return wrapper

@timed
def compute(n):
    """Sum numbers up to n."""
    return sum(range(n))

print(compute.__name__)  # 'compute' (preserved by @functools.wraps)
```

For a **decorator with arguments**, add one outer factory layer (`def repeat(times): def decorator(func): ...`).

---

### 6. Differentiate Iterators and Generators

- **Iterator:** Any object implementing `__iter__()` (returns `self`) and `__next__()` (returns next item or raises `StopIteration`).
- **Generator:** A simpler way to create an iterator using a function with `yield` (or a generator expression). Calling a generator function returns a generator object without executing the body until `next()` is called; state is suspended between yields.
- **`yield from`:** Delegates iteration directly to a sub-generator or iterable.

```python
def fibonacci(limit):
    a, b = 0, 1
    while a < limit:
        yield a
        a, b = b, a + b

print(list(fibonacci(10)))  # [0, 1, 1, 2, 3, 5, 8]
```

**Key properties:** Generators evaluate lazily (`O(1)` memory for streaming data) and are **single-use**—once exhausted, iterating again yields nothing.

---

### 7. Explain `lambda`, `map()`, `filter()`, and `reduce()`

- `lambda args: expr` creates an anonymous, single-expression function.
- `map(fn, iterable)` lazily applies `fn` to each item.
- `filter(pred, iterable)` lazily keeps items where `pred(item)` is truthy.
- `functools.reduce(fn, iterable, initializer)` cumulatively combines items into a single value.

```python
from functools import reduce

nums = [1, 2, 3, 4]
doubled = list(map(lambda x: x * 2, nums))          # [2, 4, 6, 8]
evens = list(filter(lambda x: x % 2 == 0, nums))    # [2, 4]
product = reduce(lambda a, b: a * b, nums, 1)       # 24
```

**Pythonic note:** List/generator comprehensions are usually preferred over `map`/`filter` + `lambda` for readability, and `sum()`/`math.prod()`/`any()`/`all()` are clearer than `reduce()`.

---

### 8. What are `functools.partial` and `functools.lru_cache`?

- **`functools.partial(func, *args, **kwargs)`** freezes a subset of a function's arguments, producing a new callable with fewer parameters.
- **`functools.lru_cache(maxsize=128)`** (or `@functools.cache` in 3.9+ for unbounded memoization) caches return values for hashable arguments, turning expensive pure recursive calls (like Fibonacci) from exponential to linear time.

```python
from functools import lru_cache, partial

def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
print(square(5))  # 25

@lru_cache(maxsize=None)
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)
```
