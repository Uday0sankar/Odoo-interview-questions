# Python Functions, Closures, Decorators & Generators

Core Python interview questions covering function arguments, scope rules, common pitfalls, decorators, iterators, generators, and `functools` utilities.

---

### 1. Explain `*args` and `**kwargs`

- `*args` packs extra **positional** arguments into a `tuple`.
- `**kwargs` packs extra **keyword** arguments into a `dict`.
- Full parameter order in a signature is:
  `def func(pos_only, /, standard, *args, kw_only, **kwargs):`
- Putting `*` and `**` inside a **function call** unpacks an iterable or dict back into positional/keyword arguments.

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

Default argument values are evaluated **once when `def` runs**, not on every call. If you put a mutable object like `[]` or `{}` as a default and mutate it inside the function, every later call shares that same mutated object.

**Fix:** Default to `None` and create a fresh object inside the body.

```python
# Buggy: shared list across calls
def add_item_buggy(item, items=[]):
    items.append(item)
    return items

# Safe: fresh list per call
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
2. **E (Enclosing):** In any outer wrapping functions.
3. **G (Global):** At module top level.
4. **B (Built-in):** Python built-ins (`len`, `int`, `print`).

A **closure** happens when an inner function remembers variables from its enclosing scope even after the outer function has returned. Use `nonlocal` to rebind an enclosing variable and `global` to rebind a module-level variable.

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

When a loop builds lambdas or inner functions, closures look up the loop variable **when called**, not when created—so every closure ends up seeing the final value of the loop variable.

**Fix:** Capture the current value at creation time via a default argument (`i=i`) or `functools.partial`.

```python
# Buggy: every lambda returns 2
funcs_buggy = [lambda: i for i in range(3)]
print([f() for f in funcs_buggy])  # [2, 2, 2]

# Fixed: default argument binds current value
funcs_fixed = [lambda i=i: i for i in range(3)]
print([f() for f in funcs_fixed])  # [0, 1, 2]
```

---

### 5. How do Decorators work, and why is `@functools.wraps` important?

A decorator is a callable that takes a function, wraps or extends its behavour, and returns a new callable.

Without `@functools.wraps(func)`, the wrapper overwrites the original function's `__name__`, `__doc__`, and signature metadata, which breaks debugging, docs, and tools like `pytest`.

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

*(Need a decorator that takes arguments, like `@retry(times=3)`? Wrap one more factory function around `decorator`—feel free to PR an example!)*

---

### 6. Differentiate Iterators and Generators

- **Iterator:** Any object implementing `__iter__()` (returns `self`) and `__next__()` (returns next item or raises `StopIteration`).
- **Generator:** A clean shortcut for writing an iterator using `yield` (or a generator expression). Calling a generator function returns a generator object immediately without running the body until `next()` is called; local state pauses between yields.
- **`yield from`:** Delegates iteration directly to a sub-generator or iterable.

```python
def fibonacci(limit):
    a, b = 0, 1
    while a < limit:
        yield a
        a, b = b, a + b

print(list(fibonacci(10)))  # [0, 1, 1, 2, 3, 5, 8]
```

**Key properties:** Generators evaluate lazily (`O(1)` memory for large streams) and are **single-pass**—once exhausted, iterating again yields nothing.

---

### 7. Explain `lambda`, `map()`, `filter()`, and `reduce()`

- `lambda args: expr` creates a quick, single-expression anonymous function.
- `map(fn, iterable)` lazily applies `fn` to each element.
- `filter(pred, iterable)` lazily keeps elements where `pred(item)` is truthy.
- `functools.reduce(fn, iterable, initializer)` folds items cumulatively into one value.

```python
from functools import reduce

nums = [1, 2, 3, 4]
doubled = list(map(lambda x: x * 2, nums))          # [2, 4, 6, 8]
evens = list(filter(lambda x: x % 2 == 0, nums))    # [2, 4]
product = reduce(lambda a, b: a * b, nums, 1)       # 24
```

In day-to-day Python, list/generator comprehensions are usually preferred over `map`/`filter` + `lambda` for readability, and `sum()`/`math.prod()`/`any()`/`all()` are clearer than `reduce()`.

---

### 8. What are `functools.partial` and `functools.lru_cache`?

- **`functools.partial(func, *args, **kwargs)`** pre-fills some of a function's arguments and gives back a simpler callable.
- **`functools.lru_cache(maxsize=128)`** (or `@functools.cache` in Python 3.9+) memoizes return values for hashable arguments, speeding up pure recursive or repeated lookups (just don't cache functions with mutable inputs or side effects!).

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
