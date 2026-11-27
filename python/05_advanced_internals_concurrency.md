# Python Internals, Memory, Concurrency & Exceptions

Advanced Python interview questions covering memory management, the GIL, concurrency models (`threading`, `multiprocessing`, `asyncio`), exception handling, context managers, modules, and type hints.

---

### 1. How does Python manage Memory?

CPython uses two complementary mechanisms:
1. **Reference Counting (Primary):** Every `PyObject` keeps a count of how many references point to it (`sys.getrefcount(obj)`). Once that count hits `0`, the memory is freed right away.
2. **Generational Cyclic Garbage Collector (`gc`):** Reference counting alone can't clean up **reference cycles** (e.g., `a.ref = b; b.ref = a`). Python's cyclic GC sorts container objects into 3 generations (`0`, `1`, `2`) and periodically sweeps unreachable cycles.

---

### 2. What is the Global Interpreter Lock (GIL)?

The **GIL** is a mutex inside CPython that allows only **one thread** to run Python bytecode at a time within a single process.

- **Why it exists:** Protects CPython's internal memory structures (especially reference counts) from race conditions and keeps C-extension integration simpler.
- **Practical impact:**
  - **CPU-bound tasks** (heavy math, tight loops, image processing) do **not** scale across cores with `threading`.
  - **I/O-bound tasks** (HTTP calls, database queries, disk reads) **do** benefit from `threading` / `asyncio` because CPython releases the GIL while waiting on I/O.
- **Note:** Python 3.13 introduced experimental free-threaded builds (PEP 703, `--disable-gil`), while standard CPython builds still use the GIL.

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
- `except ExcType as e`: Runs only if a matching exception occured.
- `else`: Runs **only if no exception** was raised in `try` (keeps `try` tight so you don't accidentally catch unrelated bugs).
- `finally`: **Always** runs (even after `return` or an uncaught error)—ideal for closing files or releasing locks.
- **`raise NewError(...) from err`:** Keeps the original cause chain intact in tracebacks.

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

**Avoid bare `except:`** because it also catches `KeyboardInterrupt` and `SystemExit`.

---

### 5. How do Context Managers work?

A context manager guarantees clean resource setup and teardown (files, DB transactions, locks) via the `with` statement, even when an error happens midway.

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

In a class-based context manager, returning `True` from `__exit__(self, exc_type, exc_val, exc_tb)` suppresses the exception; returning `False` or `None` lets it bubble up. *(Feel free to contribute an `async with` / `__aenter__` example if you'd like to expand this section!)*

---

### 6. What does `if __name__ == "__main__":` do?

When you run a script directly (`python app.py`), Python sets `__name__` to `"__main__"`. When another module imports it (`import app`), `__name__` is the module name (`"app"`).

Wrapping script entry points in `if __name__ == "__main__":` stops demo code or CLI runners from firing unexpectedly when the file gets imported in tests or other modules.

---

### 7. How do Type Hints work in Python?

Type hints (PEP 484+) document expected input and return types for static checkers (`mypy`, `pyright`, IDEs) without slowing down or enforcing checks at runtime.

```python
from typing import Protocol

class Serializable(Protocol):
    def to_dict(self) -> dict[str, object]: ...

def find_user(users: dict[int, str], user_id: int) -> str | None:
    return users.get(user_id)
```

In Python 3.10+, built-in generics (`list[int]`, `dict[str, float]`) and union syntax (`int | None`) replace older `typing.List`, `typing.Dict`, and `typing.Optional`.
