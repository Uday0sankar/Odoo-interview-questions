# Python Data Structures & Mutability

Intermediate Python interview questions covering built-in collections (`list`, `tuple`, `dict`, `set`), mutability, hashability, slicing, shallow vs. deep copies, comprehensions, and time complexity.

---

### 1. Compare `list`, `tuple`, `dict`, and `set`

| Structure | Ordered | Mutable | Duplicates Allowed | Syntax | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`list`** | Yes | Yes | Yes | `[1, 2, 3]` | Dynamic, ordered sequence of items |
| **`tuple`** | Yes | No | Yes | `(1, 2, 3)` | Fixed records, heterogeneous data, dict keys |
| **`dict`** | Yes (3.7+) | Yes | Keys: No / Values: Yes | `{"a": 1}` | Fast key-value lookup and structured mapping |
| **`set`** | No | Yes | No | `{1, 2, 3}` | Membership testing, deduplication, set math |

```python
# Watch out: {} creates an empty dict, NOT an empty set.
empty_dict = {}
empty_set = set()
single_tuple = (42,)  # Trailing comma is required for a 1-element tuple
```

---

### 2. What is Mutability vs. Hashability?

- **Mutable** objects can change their contents in place without changing their object identity (`id()`). Examples: `list`, `dict`, `set`, most custom classes.
- **Immutable** objects cannot be altered after creation; any "change" creates a new object. Examples: `int`, `float`, `str`, `tuple`, `frozenset`, `bool`.
- **Hashable** objects implement `__hash__()` and `__eq__()`, and their hash never changes during their lifetime. Immutable built-ins are hashable **except** a `tuple` containing a mutable item (e.g., `([1, 2], 3)` raises `TypeError: unhashable type: 'list'` if used as a `dict` key or inside a `set`).

```python
valid_key = (1, "odoo")
d = {valid_key: True}

# Invalid: tuple containing a mutable list
# {[1, 2]: "bad"}  # TypeError: unhashable type: 'list'
```

---

### 3. How does Slicing work in Python?

Slicing uses `sequence[start:stop:step]`, where `start` is inclusive, `stop` is exclusive, and `step` sets the stride (or direction when negative).

```python
items = ["a", "b", "c", "d", "e"]

print(items[1:4])    # ['b', 'c', 'd']
print(items[-2:])    # ['d', 'e']
print(items[::2])    # ['a', 'c', 'e']
print(items[::-1])   # ['e', 'd', 'c', 'b', 'a'] (reversed shallow copy)
```

---

### 4. Explain Shallow Copy vs. Deep Copy

- **Assignment (`b = a`):** Doesn't copy anything—both names point to the exact same object.
- **Shallow Copy (`a.copy()`, `list(a)`, `a[:]`, `copy.copy(a)`):** Creates a new outer container, while nested objects inside are still shared by reference.
- **Deep Copy (`copy.deepcopy(a)`):** Recursively clones the outer container and every nested object so edits stay completely seperate.

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

### 5. How do Comprehensions work, and when are Generator Expressions better?

Comprehensions give a concise, fast syntax for building lists, dicts, and sets. Generator expressions swap brackets for parentheses `(...)` and compute items **lazily** one by one instead of allocating the whole collection in RAM.

```python
nums = range(1, 6)

# List comprehension
squares = [n * n for n in nums if n % 2 == 1]  # [1, 9, 25]

# Dict comprehension
square_map = {n: n * n for n in nums}           # {1: 1, 2: 4, ...}

# Set comprehension
unique_mods = {n % 3 for n in nums}             # {0, 1, 2}

# Generator expression (O(1) memory, great for sum/any/all on huge streams)
total = sum(n * n for n in range(1_000_000))
```

---

### 6. What are common Dictionary Patterns (`get`, `setdefault`, `defaultdict`, `Counter`)?

- Use `d.get(key, default)` instead of `d[key]` when a key might be missing so you avoid `KeyError`.
- Use `d.setdefault(key, default)` or `collections.defaultdict` when grouping items into lists or sets.
- Use `collections.Counter` for frequency counting (and `.most_common(n)` for top counts).
- Use `|` (Python 3.9+) or `{**d1, **d2}` to merge dicts without mutating the originals.

```python
from collections import Counter, defaultdict

counts = {"apple": 2}
print(counts.get("banana", 0))  # 0

grouped = defaultdict(list)
for role, name in [("dev", "Alice"), ("dev", "Bob"), ("qa", "Carol")]:
    grouped[role].append(name)

word_freq = Counter(["odoo", "python", "odoo"])  # Counter({'odoo': 2, 'python': 1})
merged = {"a": 1, "b": 2} | {"b": 20, "c": 3}    # {'a': 1, 'b': 20, 'c': 3}
```

---

### 7. What are the Time Complexities of common `list`, `dict`, and `set` operations?

| Operation | `list` | `dict` / `set` |
| :--- | :--- | :--- |
| Index / Key lookup (`x[i]`, `d[k]`) | `O(1)` | `O(1)` average |
| Append / Add (`append`, `add`) | `O(1)` amortized | `O(1)` average |
| Insert / Pop at beginning (`insert(0, x)`, `pop(0)`) | `O(n)` | N/A |
| Membership test (`x in collection`) | `O(n)` | `O(1)` average |
| Delete (`pop()`, `del d[k]`) | `O(1)` at end, `O(n)` middle | `O(1)` average |

**Interview tip:** Converting a `list` to a `set` before doing repeated `x in ...` checks drops overall work from `O(n * m)` to `O(n + m)`. If you need fast queue operations at both ends (`popleft`/`appendleft`), reach for `collections.deque` (`O(1)`).
