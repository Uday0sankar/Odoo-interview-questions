# Python Memory Management, Concurrency & Exceptions

Core Python interview questions covering memory management, the GIL, concurrency models, exception handling, context managers, modules, and type hints.

---

### 1. How does Python manage Memory?

CPython uses a two-part automatic memory management system:
1. **Reference Counting (Primary):** Every `PyObject` tracks how many references point to it (`sys.getrefcount(obj)`). When the reference count drops to `0`, memory is freed immediately.
2. **Generational Cyclic Garbage Collector (`gc`):** Reference counting alone cannot free **reference cycles** (e.g., `a.ref = b; b.ref = a`). Python's cyclic GC groups container objects into 3 generations (`0`, `1`, `2`) and periodically detects and collects unreachable cycles.

---

### 2. What is the Global Interpreter Lock (GIL)?

The **GIL** is a mutex in CPython that allows only **one thread** to execute Python bytecode at a time within a single process.

- **Why it exists:** Protects CPython's internal memory structures (especially reference counts) from race conditions and simplifies C extension integration.
- **Impact:**
  - **CPU-bound tasks** (math, image processing, tight loops) do **not** scale across cores with `threading`.
  - **I/O-bound tasks** (network requests, DB queries, disk I/O) **do** benefit from `threading` / `asyncio` because CPython releases the GIL while waiting on I/O.
- **Note:** Python 3.13 introduced experimental free-threaded builds (PEP 703, `--disable-gil`), while standard CPython builds still enforce the GIL.

---

### 3. Compare `threading`, `multiprocessing`, and `asyncio`

| Model | Concurrency Style | Best For | Memory Model | GIL Impact |
| :--- | :--- | :--- | :--- | :--- |
| **`threading`** | Preemptive OS threads | Blocking I/O (legacy drivers, HTTP/DB libs) | Shared memory (needs locks) | Serializes CPU Python bytecode |
| **`multiprocessing`** | Separate OS processes | CPU-bound work (data crunching, ML) | Isolated per process (IPC/Pickling) | Bypasses GIL (1 GIL per process) |
| **`asyncio`** | Cooperative single-thread event loop | High-concurrency non-blocking I/O | Single thread (minimal locking) | Runs on 1 thread |

```python
import asyncio

async def fetch_data(idx):
    await asyncio.sleep(0.1)  # Yields control back to event loop
    return f"item-{idx}"

async def main():
    results = await asyncio.gather(*(fetch_data(i) for i in range(3)))
    print(results)

# asyncio.run(main())
```

---

### 4. Explain `try` / `except` / `else` / `finally` and Exception Best Practices

- `try`: Code that might raise an exception.
- `except ExcType as e`: Runs only if a matching exception occurs.
- `else`: Runs **only if no exception** was raised in `try` (keeps the `try` block small so you don't accidentally catch unrelated errors).
- `finally`: **Always** runs (whether an exception occurred, was caught, or `return` was executed)—ideal for cleanup.
- **`raise NewError(...) from err`:** Preserves the original traceback cause chain.

```python
class InsufficientStockError(Exception):
    """Domain-specific exception."""

def parse_positive_int(raw):
    try:
        value = int(raw)
    except ValueError as err:
        raise ValueError(f"Invalid integer: {raw!r}") from err
    else:
        if value <= 0:
            raise ValueError("Must be positive")
        return value
    finally:
        pass  # Cleanup runs regardless
```

**Avoid bare `except:`** because it also swallows `KeyboardInterrupt` and `SystemExit`.

---

### 5. How do Context Managers work?

A context manager ensures deterministic resource setup and teardown (files, DB transactions, locks) via the `with` statement, even if an exception occurs.

You can write one as a class (`__enter__` / `__exit__`) or concisely with `@contextlib.contextmanager`:

```python
from contextlib import contextmanager

@contextmanager
def open_resource(name):
    print(f"Acquiring {name}")
    try:
        yield name
    finally:
        print(f"Releasing {name}")

with open_resource("db_conn") as res:
    print(f"Using {res}")
```

In a class-based context manager, returning `True` from `__exit__(self, exc_type, exc_val, exc_tb)` suppresses the exception; returning `False` or `None` lets it propagate.

---

### 6. What does `if __name__ == "__main__":` do?

When Python executes a file directly (`python app.py`), it sets `__name__` to `"__main__"`. When the same file is imported by another module (`import app`), `__name__` is set to the module's name (`"app"`).

Guarding entry-point code with `if __name__ == "__main__":` prevents scripts or tests from running accidentally upon import.

---

### 7. How do Type Hints work in Python?

Type hints (PEP 484+) document expected parameter and return types for static analyzers (`mypy`, `pyright`, IDEs) without enforcing types at runtime.

```python
from typing import Protocol

class Serializable(Protocol):
    def to_dict(self) -> dict[str, object]: ...

def find_user(users: dict[int, str], user_id: int) -> str | None:
    return users.get(user_id)
```

In Python 3.10+, built-in generics (`list[int]`, `dict[str, float]`) and union syntax (`int | None`) replace `typing.List`, `typing.Dict`, and `typing.Optional`.
