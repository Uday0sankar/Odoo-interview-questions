# Python Basics: Beginner to Core

Start here if you're brushing up from scratch or want simple, clear answers to entry-level Python interview questions. Everything is explained using everyday examples before moving into deeper internals in later files.

---

### 1. What is a Variable in Python, and what does "Dynamically Typed" mean?

Think of a variable like a **sticky-note label** attached to a value in memory. Unlike C or Java, you don't need to announce the data type up front—Python figures out the type at runtime based on the value you assign.

```python
score = 10          # int
score = "ten"       # now str (same label, different value)
```

Because the label can point to a different type later, Python is **dynamically typed**. At the same time, Python is **strongly typed**: `"10" + 5` raises a `TypeError` instead of silently guessing whether you meant `"105"` or `15`.

---

### 2. What are Python's Basic Data Types?

| Type | What it holds | Example | Notes |
| :--- | :--- | :--- | :--- |
| **`int`** | Whole numbers (unlimited size) | `42`, `-7`, `1_000` | Underscores work as visual seperators |
| **`float`** | Decimal numbers | `3.14`, `-0.5` | Watch out for floating-point rounding (`0.1 + 0.2`) |
| **`str`** | Text wrapped in quotes | `"Odoo"`, `'hi'` | Immutable sequence of characters |
| **`bool`** | `True` or `False` | `is_active = True` | Subclass of `int` (`True == 1`, `False == 0`) |
| **`NoneType`** | `None` (no value / empty) | `result = None` | Similar to `null` in other languages |

```python
age = 25
price = 19.99
name = "Alice"
is_admin = False
middle_name = None

print(type(age), type(price), type(name))
```

---

### 3. Explain `==` vs. `is`, and Truthy vs. Falsy Values

- **`==`** asks: *"Do these two things have the same value?"*
- **`is`** asks: *"Are these two names pointing to the exact same object in memory?"*

Always use `is` / `is not` when checking for `None` (e.g. `if user is None:`), and use `==` for numbers, strings, and lists. *(Quick note: CPython caches small ints from `-5` to `256`, which sometimes fools beginners into thinking `is` works for all numbers—don't rely on it!)*

**Truthy and Falsy values:**
In an `if` condition, Python treats empty/zero values as **Falsy** and almost everything else as **Truthy**:
- **Falsy:** `False`, `None`, `0`, `0.0`, `""`, `[]`, `{}`, `set()`, `()`
- **Truthy:** Non-zero numbers, non-empty strings/collections (even `"False"` or `[0]`).

```python
items = []
if not items:
    print("Cart is empty")  # Runs because [] is falsy

val = None
if val is None:
    print("Nothing set yet")
```

---

### 4. How do `if` / `elif` / `else` Conditionals work?

Conditionals let your program pick one branch out of several. Python uses **indentation** (usually 4 spaces) instead of curly braces `{}` to group code blocks.

```python
temp = 28

if temp >= 35:
    print("Too hot")
elif temp >= 20:
    print("Pleasant weather")
else:
    print("A bit chilly")
```

For simple one-liners, Python also supports a ternary expression:
`status = "Adult" if age >= 18 else "Minor"`.

---

### 5. Explain `for` vs. `while` Loops, plus `break`, `continue`, and `pass`

- **`for` loop:** Best when you know what collection or `range(...)` you are stepping through.
- **`while` loop:** Keeps repeating as long as a condition stays `True` (handy when you don't know how many tries something will take).
- **`break`:** Exits the loop immediately.
- **`continue`:** Skips the rest of the current turn and jumps to the next iteration.
- **`pass`:** A "do nothing" placeholder so empty blocks don't cause an `IndentationError`.

```python
for num in range(1, 6):
    if num == 2:
        continue  # skip 2
    if num == 5:
        break     # stop before 5
    print(num)    # prints 1, 3, 4

def todo_later():
    pass          # placeholder until implemented
```

*(Note for contributors: a small `for ... else` example showing when `else` runs without `break` would be a great addition here.)*

---

### 6. What are the most useful String Formatting and String Methods?

Modern Python prefers **f-strings** (`f"..."`) because they are fast and easy to read. Since strings are **immutable**, methods like `.strip()` or `.replace()` return a *new* string rather than changing the original.

```python
raw_name = "  uday sankar  "
clean = raw_name.strip().title()           # "Uday Sankar"
words = clean.split(" ")                   # ['Uday', 'Sankar']
joined = "-".join(words)                   # "Uday-Sankar"

order_no = "SO0042"
total = 150.5
print(f"Order {order_no}: ${total:.2f}")   # Order SO0042: $150.50
```

---

### 7. What is the difference between `print()` and `return` in a Function?

This is a classic beginner interview question:
- **`print()`** just shows text on the screen for a human to read—the function still gives back `None` unless you explicitly `return` something.
- **`return`** hands a value back to whoever called the function so the program can store it, pass it into another function, or do math with it, and immediately exits the function.

```python
def add_with_print(a, b):
    print(a + b)   # displays 5, returns None

def add_with_return(a, b):
    return a + b   # hands back 5

x = add_with_return(2, 3)
print(x * 10)      # 50
```
