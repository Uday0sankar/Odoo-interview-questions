# Python Data Structures & Mutability

Core Python interview questions covering built-in collections, mutability, copying semantics, identity vs. equality, comprehensions, and Big-O complexity.

---

### 1. Compare `list`, `tuple`, `dict`, and `set`

| Structure | Ordered | Mutable | Duplicates Allowed | Syntax | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`list`** | Yes | Yes | Yes | `[1, 2, 3]` | Dynamic, ordered sequence of items |
| **`tuple`** | Yes | No | Yes | `(1, 2, 3)` | Fixed records, heterogeneous data, dict keys |
| **`dict`** | Yes (3.7+) | Yes | Keys: No / Values: Yes | `{"a": 1}` | Fast key-value lookup and structured mapping |
| **`set`** | No | Yes | No | `{1, 2, 3}` | Membership testing, deduplication, set algebra |

```python
# Note: {} creates an empty dict, NOT an empty set.
empty_dict = {}
empty_set = set()
single_tuple = (42,)  # Trailing comma is required for a 1-element tuple
```

---

### 2. What is Mutability vs. Hashability?

- **Mutable** objects can change their value in place without changing their memory identity (`id()`). Examples: `list`, `dict`, `set`, custom objects.
- **Immutable** objects cannot be altered after creation; operations produce new objects. Examples: `int`, `float`, `str`, `tuple`, `frozenset`, `bool`.
- **Hashable** objects implement `__hash__()` and `__eq__()`, and their hash value never changes during their lifetime. Immutable built-ins are hashable **except** a `tuple` that contains a mutable element (e.g., `([1, 2], 3)` raises `TypeError: unhashable type: 'list'` when used as a `dict` key or `set` member).

```python
valid_key = (1, "odoo")
d = {valid_key: True}

# Invalid: tuple containing a mutable list
# {[1, 2]: "bad"}  # TypeError: unhashable type: 'list'
```

---

### 3. Explain Shallow Copy vs. Deep Copy

- **Assignment (`b = a`)**: Does not copy at all—both names point to the same object.
- **Shallow Copy (`a.copy()`, `list(a)`, `a[:]`, `copy.copy(a)`)**: Creates a new outer container, but nested objects are still shared by reference.
- **Deep Copy (`copy.deepcopy(a)`)**: Recursively copies the outer container and every nested object so mutations are completely isolated.

```python
import copy

original = [[1, 2], [3, 4]]

shallow = copy.copy(original)
deep = copy.deepcopy(original)

shallow[0].append(99)
print(original)  # [[1, 2, 99], [3, 4]] -> nested list was shared
print(deep)      # [[1, 2], [3, 4]]     -> completely independent
```

---

### 4. What is the difference between `==` and `is`?

- `==` checks **value equality** (calls `__eq__()`).
- `is` checks **object identity** (`id(a) == id(b)` — whether both variables reference the exact same object in memory).

**Best practice:** Always use `is` / `is not` when comparing with `None`, `True`, or `False`, and use `==` for numbers, strings, and collections. CPython caches small integers (`-5` to `256`) and interns some short strings, which can make `is` appear to work on literals in tests while failing on larger values.

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(a == b)  # True  (same values)
print(a is b)  # False (two distinct list objects)

val = None
print(val is None)  # True (idiomatic)
```

---

### 5. How do Comprehensions work, and when should you use Generator Expressions?

Comprehensions offer a concise, fast syntax for constructing lists, dicts, and sets. Generator expressions use parentheses `(...)` and compute items **lazily** on demand instead of allocating the entire collection in memory.

```python
nums = range(1, 6)

# List comprehension
squares = [n * n for n in nums if n % 2 == 1]  # [1, 9, 25]

# Dict comprehension
square_map = {n: n * n for n in nums}           # {1: 1, 2: 4, ...}

# Set comprehension
unique_mods = {n % 3 for n in nums}             # {0, 1, 2}

# Generator expression (O(1) memory, ideal for sum/any/all on large streams)
total = sum(n * n for n in range(1_000_000))
```

---

### 6. How does Slicing work in Python?

Slicing follows `sequence[start:stop:step]`, where `start` is inclusive, `stop` is exclusive, and `step` controls stride and direction.

```python
items = ["a", "b", "c", "d", "e"]

print(items[1:4])    # ['b', 'c', 'd']
print(items[-2:])    # ['d', 'e']
print(items[::2])    # ['a', 'c', 'e']
print(items[::-1])   # ['e', 'd', 'c', 'b', 'a'] (reversed shallow copy)
```

---

### 7. What are the safest ways to access and merge Dictionaries?

- Use `d.get(key, default)` instead of `d[key]` when a key might be missing to avoid `KeyError`.
- Use `d.setdefault(key, default)` or `collections.defaultdict` when grouping values.
- Use `|` (Python 3.9+) or `{**d1, **d2}` to merge dictionaries cleanly without mutating the originals.

```python
from collections import defaultdict

counts = {"apple": 2}
print(counts.get("banana", 0))  # 0

grouped = defaultdict(list)
for role, name in [("dev", "Alice"), ("dev", "Bob"), ("qa", "Carol")]:
    grouped[role].append(name)

merged = {"a": 1, "b": 2} | {"b": 20, "c": 3}  # {'a': 1, 'b': 20, 'c': 3}
```

---

### 8. What are the Time Complexities of common `list`, `dict`, and `set` operations?

| Operation | `list` | `dict` / `set` |
| :--- | :--- | :--- |
| Index / Key lookup (`x[i]`, `d[k]`) | `O(1)` | `O(1)` average |
| Append / Add (`append`, `add`) | `O(1)` amortized | `O(1)` average |
| Insert / Pop at beginning (`insert(0, x)`, `pop(0)`) | `O(n)` | N/A |
| Membership test (`x in collection`) | `O(n)` | `O(1)` average |
| Delete (`pop()`, `del d[k]`) | `O(1)` at end, `O(n)` middle | `O(1)` average |

**Interview tip:** Converting a `list` to a `set` before doing repeated `x in ...` lookups reduces overall complexity from `O(n * m)` to `O(n + m)`. For frequent queue operations at both ends (`popleft`/`appendleft`), use `collections.deque` (`O(1)`).
