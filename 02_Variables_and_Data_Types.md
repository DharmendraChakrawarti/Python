# 02. Variables and Data Types in Python

## Table of Contents

1. [Variables: Declaration, Assignment, Naming Rules & Conventions](#1-variables-declaration-assignment-naming-rules--conventions)
2. [Dynamic Typing in Python](#2-dynamic-typing-in-python)
3. [Data Types Overview](#3-data-types-overview)
4. [Numeric Types (`int`, `float`, `complex`)](#4-numeric-types-int-float-complex)
5. [Boolean Type (`bool`) & Truthy / Falsy Values](#5-boolean-type-bool--truthy--falsy-values)
6. [Type Checking: `type()` vs `isinstance()`](#6-type-checking-type-vs-isinstance)
7. [Type Conversion / Casting (Implicit and Explicit)](#7-type-conversion--casting-implicit-and-explicit)
8. [Constants in Python](#8-constants-in-python)
9. [Multiple Assignment and Swapping Variables](#9-multiple-assignment-and-swapping-variables)
10. [`id()` Function and Memory Management](#10-id-function-and-memory-management)
11. [Mutable vs Immutable Types](#11-mutable-vs-immutable-types)
12. [Python's Object Model (Everything Is an Object)](#12-pythons-object-model-everything-is-an-object)

---

## 1. Variables: Declaration, Assignment, Naming Rules & Conventions

In Python, a **variable** is a symbolic name (identifier) bound to an object in memory. Unlike C or Java, Python has **no separate variable declaration step**—a variable is created the moment you assign a value to it using the assignment operator (`=`).

```python
# Assignment creates the variable and binds it to an object
name = "Alice"
age = 28
height = 1.68
is_student = False
```

> **Note:** Think of a Python variable as a **label or reference** attached to an object, rather than a typed box that holds bytes directly.

### Variable Naming Rules (Enforced by Python)

If you break any of these rules, Python raises a `SyntaxError`:

1. **Allowed characters:** Letters (`a–z`, `A–Z`, plus Unicode letters), digits (`0–9`), and underscores (`_`).
2. **Must not start with a digit:** `user1` is valid; `1user` is invalid.
3. **No spaces or special symbols:** Characters like `-`, `@`, `$`, `%`, or spaces are not allowed.
4. **Case-sensitive:** `age`, `Age`, and `AGE` are three distinct variables.
5. **Cannot be a reserved keyword:** Words like `if`, `for`, `class`, `True`, `None`, `return`, `import`, etc. cannot be used as variable names.

```python
# Valid variable names
user_name = "dc"
_private_flag = True
score2 = 95

# Invalid variable names (would raise SyntaxError)
# 2score = 95        # Starts with a digit
# user-name = "dc"   # Contains hyphen
# class = "Math"     # Reserved keyword
```

You can inspect all reserved keywords in your installed Python version using the `keyword` module:

```python
import keyword

print(keyword.kwlist)
print("Is 'async' a keyword?", keyword.iskeyword("async"))  # True
```

### Naming Conventions (PEP 8 Style Guide)

While Python enforces syntax rules, **PEP 8** defines community conventions for readable code:

| Identifier Type | Convention | Example |
| :--- | :--- | :--- |
| Regular variables & functions | `snake_case` (lowercase with underscores) | `total_price`, `max_retry_count` |
| Constants | `UPPER_SNAKE_CASE` | `PI`, `MAX_CONNECTIONS` |
| Classes | `PascalCase` (`CapWords`) | `BankAccount`, `HttpResponse` |
| Internal / "private" use | Leading single underscore `_name` | `_cache`, `_internal_id` |
| Avoid keyword collision | Trailing single underscore `name_` | `class_`, `type_` |
| Name mangling in classes | Leading double underscore `__name` | `__balance` |
| Special / "dunder" methods | Double underscores on both sides | `__init__`, `__len__` |

> **Tip:** Avoid shadowing built-in names like `list`, `str`, `id`, `sum`, or `type`. Doing `list = [1, 2, 3]` works syntactically, but breaks the built-in `list()` function in that scope.

---

## 2. Dynamic Typing in Python

Python is **dynamically typed** and **strongly typed**:

- **Dynamically typed:** Type checking happens at **runtime**, not compile time. A variable name does not have a fixed type; only the **object** it currently references has a type. You can rebind the same name to objects of different types.
- **Strongly typed:** Python does not silently perform unsafe conversions between unrelated types (for example, `"5" + 2` raises a `TypeError` rather than `"52"` or `7`).

```python
value = 42
print(value, type(value))       # 42 <class 'int'>

value = "forty-two"
print(value, type(value))       # forty-two <class 'str'>

value = [1, 2, 3]
print(value, type(value))       # [1, 2, 3] <class 'list'>
```

### Optional Type Hints (PEP 484)

Modern Python supports **type annotations** for documentation, IDE autocompletion, and static type checkers (like `mypy` or `pyright`). Note that **Python still does not enforce type hints at runtime**:

```python
count: int = 10
greeting: str = "Hello"

# Runtime still allows rebinding unless checked by an external static checker
count = "ten"  # No runtime error
```

---

## 3. Data Types Overview

Python provides built-in core scalar (single-value) types alongside built-in collection types. The fundamental scalar types covered here are:

| Category | Type | Description | Example Literals |
| :--- | :--- | :--- | :--- |
| Numeric | `int` | Arbitrary-precision integers | `0`, `-42`, `1_000_000` |
| Numeric | `float` | Double-precision floating-point numbers | `3.14`, `-0.5`, `1.5e-3` |
| Numeric | `complex` | Complex numbers (`real + imag*j`) | `2 + 3j`, `1j`, `-4.5j` |
| Text | `str` | Immutable sequence of Unicode characters | `"hello"`, `'Python'`, `"""multi"""` |
| Boolean | `bool` | Logical truth values (subclass of `int`) | `True`, `False` |
| Null / Empty | `NoneType` | Represents the absence of a value | `None` |

### Quick Look at `str` and `NoneType`

```python
# Strings (str)
single = 'Python'
double = "Data Types"
multiline = """Line 1
Line 2"""

# NoneType
result = None
print(type(result))        # <class 'NoneType'>
print(result is None)      # True  (always use 'is' / 'is not' to check for None)
```

> **Note:** `None` is a singleton object—there is only ever one `None` instance in a Python process. Use `x is None` instead of `x == None`.

Other built-in collection types you will use alongside these include `list`, `tuple`, `set`, `frozenset`, `dict`, `bytes`, and `bytearray`.

---

## 4. Numeric Types (`int`, `float`, `complex`)

### Integers (`int`)

- Whole numbers (positive, negative, or zero) with **arbitrary precision**—limited only by available memory (no 32-bit or 64-bit integer overflow).
- Underscores `_` can be used as visual separators in numeric literals.
- Supports binary (`0b`), octal (`0o`), and hexadecimal (`0x`) literals.

```python
a = 42
big_num = 10 ** 50                 # Arbitrary precision works seamlessly
readable = 1_000_000_000           # 1000000000

binary_val = 0b1010                # 10
octal_val = 0o12                   # 10
hex_val = 0xFF                     # 255
print(binary_val, octal_val, hex_val)
```

### Floating-Point Numbers (`float`)

- Represented using IEEE 754 64-bit double precision.
- Supports standard decimal notation and scientific notation (`e` or `E`).
- Includes special values: `float("inf")`, `float("-inf")`, and `float("nan")`.

```python
pi = 3.14159
avogadro = 6.022e23
tiny = 2.5e-4                      # 0.00025

# Floating-point representation quirk (IEEE 754 binary approximation)
print(0.1 + 0.2)                   # 0.30000000000000004
print(round(0.1 + 0.2, 2) == 0.3)  # True
```

> **Tip:** For exact base-10 arithmetic (such as financial calculations), use `decimal.Decimal` instead of `float`.

### Complex Numbers (`complex`)

- Written as `a + bj`, where `a` is the real part and `b` is the imaginary part (both stored internally as `float`).
- Use `j` (or `J`), not `i`.

```python
z1 = 3 + 4j
z2 = complex(1, -2)                # 1 - 2j

print(z1.real)                     # 3.0
print(z1.imag)                     # 4.0
print(z1.conjugate())              # (3-4j)
print(abs(z1))                     # 5.0  (magnitude: sqrt(3^2 + 4^2))
print(z1 + z2)                     # (4+2j)
```

### Numeric Operations

| Operator | Operation | Example | Result | Notes |
| :---: | :--- | :--- | :--- | :--- |
| `+` | Addition | `7 + 3` | `10` | |
| `-` | Subtraction | `7 - 3` | `4` | |
| `*` | Multiplication | `7 * 3` | `21` | |
| `/` | True division | `7 / 2` | `3.5` | Always returns a `float` (or `complex`) |
| `//` | Floor division | `7 // 2` | `3` | Rounds down toward `-inf` (`-7 // 2 == -4`) |
| `%` | Modulo (remainder) | `7 % 3` | `1` | Sign matches divisor (`-7 % 3 == 2`) |
| `**` | Exponentiation | `2 ** 5` | `32` | Right-associative (`2 ** 3 ** 2 == 512`) |

> **Note:** Floor division (`//`) and modulo (`%`) are **not supported** for `complex` numbers (`TypeError`).

### Numeric Conversions

```python
print(int(3.99))        # 3 (truncates toward zero, does NOT round)
print(int(-3.99))       # -3
print(float(5))         # 5.0
print(complex(5))       # (5+0j)
print(complex(2, 3))    # (2+3j)

# Base conversions to int from strings
print(int("1010", 2))   # 10
print(int("ff", 16))    # 255
```

---

## 5. Boolean Type (`bool`) & Truthy / Falsy Values

Python's `bool` type has exactly two values: `True` and `False` (case-sensitive).

Interestingly, `bool` is a **subclass of `int`**:
- `True` behaves like `1` in numeric contexts (`True == 1` is `True`).
- `False` behaves like `0` in numeric contexts (`False == 0` is `True`).

```python
print(issubclass(bool, int))  # True
print(True + True)            # 2
print(False * 10)             # 0
print(isinstance(True, int))  # True
```

### Logical Operators

| Operator | Meaning | Example |
| :---: | :--- | :--- |
| `and` | True if both operands are truthy | `True and False` → `False` |
| `or` | True if at least one operand is truthy | `True or False` → `True` |
| `not` | Negates truthiness | `not True` → `False` |

**Short-circuit evaluation:** `and` and `or` evaluate from left to right and return **one of the actual operand values** (not necessarily a `bool`), stopping as soon as the result is determined:

```python
name = "" or "Guest"          # "Guest"
config = {"debug": True} and "OK"  # "OK"
```

### Truthy and Falsy Values

Every Python object can be evaluated in a boolean context (e.g., `if x:` or `bool(x)`).

**Falsy values** (evaluate to `False`):
- `False`
- `None`
- Zero of any numeric type: `0`, `0.0`, `0j`, `Decimal(0)`, `Fraction(0, 1)`
- Empty sequences and collections: `""`, `[]`, `()`, `{}`, `set()`, `range(0)`, `b""`
- Custom objects whose `__bool__()` returns `False` or `__len__()` returns `0`

**Truthy values:** Everything else (non-zero numbers, non-empty strings/containers, most objects).

```python
samples = [0, 1, -5, 0.0, 0.1, "", "hi", [], [0], {}, None, True, False]
for item in samples:
    print(f"{item!r:6} -> {bool(item)}")
```

> **Pitfall:** `"False"` (a non-empty string) is **truthy** (`bool("False") == True`), and `[0]` (a non-empty list) is **truthy**!

---

## 6. Type Checking: `type()` vs `isinstance()`

Python offers two primary built-in functions for checking an object's type:

### `type(obj)`

Returns the **exact** class/type of an object. It does **not** account for inheritance.

```python
x = 42
print(type(x))              # <class 'int'>
print(type(x) is int)       # True
print(type(True) is int)    # False (exact type of True is bool, not int)
```

### `isinstance(obj, class_or_tuple)`

Checks whether `obj` is an instance of `class` **or any of its subclasses** (and also accepts a tuple of types). **This is the preferred Pythonic way** in almost all situations.

```python
print(isinstance(42, int))                 # True
print(isinstance(True, bool))              # True
print(isinstance(True, int))               # True (because bool inherits from int!)
print(isinstance(3.14, (int, float)))      # True (matches either int or float)
```

### When to Use Which

| Scenario | Recommended Function | Why |
| :--- | :--- | :--- |
| General type validation | `isinstance(x, T)` | Respects inheritance and polymorphism |
| Checking multiple allowed types | `isinstance(x, (int, float))` | Clean tuple support |
| Distinguishing `bool` from `int` | Check `bool` first or use `type(x) is int` | Because `isinstance(True, int)` is `True` |
| Debugging / inspecting exact runtime class | `type(x)` | Shows exact class |

```python
# Watch out for bool/int order when using isinstance:
def describe_number(val):
    if isinstance(val, bool):
        return "boolean"
    elif isinstance(val, int):
        return "integer"
    return "other"

print(describe_number(True))  # boolean
print(describe_number(1))     # integer
```

---

## 7. Type Conversion / Casting (Implicit and Explicit)

### Implicit Type Conversion (Coercion)

Python automatically promotes narrower numeric types to wider ones in mixed-type expressions to avoid data loss:

`bool` → `int` → `float` → `complex`

```python
result1 = 5 + 2.5         # int + float -> float (7.5)
result2 = 3 + True        # int + bool  -> int (4)
result3 = 2.0 + (1 + 3j)  # float + complex -> complex (3+3j)

print(type(result1), type(result2), type(result3))
```

Python does **not** implicitly convert between strings and numbers:

```python
# "Age: " + 25   # TypeError: can only concatenate str (not "int") to str
```

### Explicit Type Conversion (Casting)

Use built-in constructor functions to explicitly convert values:

| Function | Converts to | Common Example | Notes / Gotchas |
| :--- | :--- | :--- | :--- |
| `int(x)` | Integer | `int("42")` → `42` | `int("3.14")` raises `ValueError`; use `int(float("3.14"))` |
| `float(x)` | Float | `float("3.14")` → `3.14` | `float("inf")` works; `float("abc")` raises `ValueError` |
| `complex(r, i)` | Complex | `complex("2+3j")` → `(2+3j)` | No spaces allowed around `+` in string form |
| `str(x)` | String | `str(100)` → `"100"` | Works on almost any Python object |
| `bool(x)` | Boolean | `bool("")` → `False` | Uses truthiness rules |

```python
# Explicit conversions
num_str = "42"
num_int = int(num_str)
num_float = float(num_str)
print(num_int, num_float)            # 42 42.0

# Handling decimal strings to int
decimal_str = "9.8"
truncated = int(float(decimal_str))  # 9

# String formatting instead of concatenation
age = 25
print("Age: " + str(age))            # Age: 25
print(f"Age: {age}")                 # Preferred: Age: 25
```

---

## 8. Constants in Python

Python **does not have a `const` or `final` keyword that prevents reassignment at runtime**. Instead, constants are represented by **naming convention**:

- Name constants in `UPPER_SNAKE_CASE`.
- Typically define them at the **module level** (top of the file).

```python
PI = 3.141592653589793
MAX_LOGIN_ATTEMPTS = 5
DEFAULT_TIMEOUT_SECONDS = 30
API_BASE_URL = "https://api.example.com/v1"
```

### Static Enforcement with `typing.Final`

For static type checkers (`mypy`, `pyright`, IDE linters), you can annotate constants with `Final` (Python 3.8+) to catch accidental reassignments before running the code:

```python
from typing import Final

MAX_USERS: Final[int] = 100
# MAX_USERS = 200  # Flagged by mypy/pyright, though Python won't stop it at runtime
```

> **Tip:** If you need true immutability for grouped constants, consider `enum.Enum`, a `tuple`, or a `@dataclass(frozen=True)`.

---

## 9. Multiple Assignment and Swapping Variables

Python provides clean, concise syntax for assigning multiple variables at once.

### 1. Tuple Unpacking (Parallel Assignment)

Assign multiple variables to distinct values on one line:

```python
x, y, z = 10, 20, "Python"
print(x, y, z)  # 10 20 Python

# Extended iterable unpacking with *
first, *middle, last = [1, 2, 3, 4, 5]
print(first)    # 1
print(middle)   # [2, 3, 4]
print(last)     # 5
```

### 2. Chained Assignment

Bind multiple variables to the **same object**:

```python
a = b = c = 0
print(a, b, c)  # 0 0 0
```

> **Warning (Mutable Trap):** Never use chained assignment with mutable objects unless you intentionally want all variables to share the exact same object:
> ```python
> list_a = list_b = []
> list_a.append(1)
> print(list_b)  # [1]  <-- Both variables point to the same list!
>
> # Safe alternative:
> list_a, list_b = [], []
> ```

### 3. Swapping Variables

In Python, swapping two (or more) variables does **not** require a temporary variable. The right-hand side (`b, a`) is evaluated and packed into a tuple first, then unpacked into the left-hand side (`a, b`):

```python
a = 5
b = 10
a, b = b, a
print(a, b)  # 10 5

# Rotating three variables
x, y, z = 1, 2, 3
x, y, z = z, x, y
print(x, y, z)  # 3 1 2
```

---

## 10. `id()` Function and Memory Management

### The `id()` Function and `is` vs `==`

Every object in Python has a unique **identity** during its lifetime, returned as an integer by `id(obj)`. In CPython (the standard Python implementation), `id(obj)` is the object's **memory address**.

- `==` compares **values** (equality).
- `is` compares **identities** (`id(a) == id(b)` — whether two names point to the exact same object in memory).

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)          # True  (same contents)
print(a is b)          # False (two separate list objects in memory)
print(a is c)          # True  (c references the same object as a)
print(id(a), id(b), id(c))
```

### CPython Memory Management

1. **Reference Counting (Primary Mechanism):**
   Every object stores a count of how many references point to it. Assigning a variable or adding an item to a list increments the count; `del`, reassigning a variable, or leaving a function scope decrements it. When the reference count drops to `0`, CPython immediately frees the memory.

   ```python
   import sys

   nums = [10, 20, 30]
   print(sys.getrefcount(nums))  # 2 (nums + the temporary argument passed to getrefcount)
   alias = nums
   print(sys.getrefcount(nums))  # 3
   del alias
   print(sys.getrefcount(nums))  # 2
   ```

2. **Cyclic Garbage Collector (GC):**
   Reference counting alone cannot clean up **reference cycles** (e.g., object A references object B, and B references A, while neither is reachable from anywhere else). Python's generational garbage collector (`gc` module) periodically detects and reclaims reference cycles.

3. **Object Interning / Caching (Optimization):**
   To save memory and speed up lookups, CPython caches and reuses certain immutable objects:
   - **Small integers** in the range `[-5, 256]` are pre-allocated on startup.
   - **Short / identifier-like strings** are often interned automatically (or manually via `sys.intern()`).

   ```python
   x = 256
   y = 256
   print(x is y)  # True (cached small int)

   # Large ints created separately at runtime may have different ids
   # (Note: within the same compiled code block/line, Python may fold constants)
   p = int("1000")
   q = int("1000")
   print(p == q)  # True
   print(p is q)  # False (separate objects)
   ```

   > **Rule of thumb:** Always use `==` to compare values (numbers, strings, lists), and reserve `is` for singletons (`x is None`, `x is True`).

---

## 11. Mutable vs Immutable Types

In Python, types are classified by whether an object's internal state (value) can be changed **after creation**:

- **Immutable:** Cannot be modified in place. Any operation that "changes" it actually creates a **new object** and rebinds the variable.
- **Mutable:** Can be modified in place without changing the object's `id()`.

### Classification Table

| Immutable Types | Mutable Types |
| :--- | :--- |
| `int`, `float`, `complex` | `list` |
| `bool` | `dict` |
| `str` | `set` |
| `tuple` | `bytearray` |
| `frozenset` | User-defined classes (by default) |
| `bytes`, `range`, `NoneType` | |

### Demonstrating the Difference with `id()`

```python
# Immutable: int / str creates a new object upon "modification"
s = "hello"
print("Before:", id(s))
s = s + " world"
print("After: ", id(s))  # Different id! A new str object was created

# Mutable: list modifies the existing object in place
items = [1, 2, 3]
print("Before:", id(items))
items.append(4)
print("After: ", id(items))  # Same id! Modified in place
```

### Aliasing & Side Effects with Mutable Objects

When two variables reference the same mutable object, mutating through one affects the other:

```python
original = [1, 2, 3]
alias = original
alias.append(4)

print(original)  # [1, 2, 3, 4]

# To avoid modifying the original, make a copy:
shallow_copy = original.copy()  # or list(original) or original[:]
shallow_copy.append(5)
print(original)      # [1, 2, 3, 4]
print(shallow_copy)  # [1, 2, 3, 4, 5]
```

> **Note on Nested Structures:** `.copy()` creates a *shallow copy*. For nested mutable structures (like a list of lists), use `copy.deepcopy()`.

### Subtle Gotcha: Mutable Elements Inside a `tuple`

A `tuple` is immutable—you cannot reassign its slots—but if a slot holds a mutable object (like a `list`), that inner object can still be mutated:

```python
t = (1, [2, 3])
# t[0] = 99       # TypeError: 'tuple' object does not support item assignment
t[1].append(4)    # Allowed! The tuple still points to the same list object
print(t)          # (1, [2, 3, 4])
```

---

## 12. Python's Object Model (Everything Is an Object)

In Python, **everything is an object**—numbers, strings, functions, classes, modules, and even `None` and types themselves.

Every object has three fundamental properties:
1. **Identity** (`id(obj)`) — never changes once created.
2. **Type** (`type(obj)`) — determines supported operations and methods; generally never changes.
3. **Value** — the data held by the object (changeable if mutable, fixed if immutable).

### Even Literals, Functions, and Types Are Objects

Because everything is an object, you can inspect attributes, call methods directly on literals, assign functions to variables, and pass types as arguments:

```python
# 1. Methods and attributes on primitive literals
num = 42
print(num.bit_length())       # 6
print((3.5).is_integer())     # False
print("python".upper())       # PYTHON

# 2. Functions are first-class objects
def greet(name):
    return f"Hello, {name}!"

say_hi = greet                # Assign function object to another variable
print(say_hi("World"))        # Hello, World!
print(type(greet))            # <class 'function'>

# 3. Classes and types themselves are objects (instances of 'type')
print(type(int))              # <class 'type'>
print(type(type))             # <class 'type'>
print(isinstance(42, object)) # True
print(isinstance(greet, object))  # True
print(isinstance(int, object))    # True
```

### The `object` and `type` Hierarchy

- **`object`** is the root base class from which every class in Python inherits (directly or indirectly).
- **`type`** is the metaclass whose instances are classes themselves (including `int`, `str`, `object`, and `type` itself).

```python
print(issubclass(int, object))     # True
print(issubclass(str, object))     # True
print(issubclass(type, object))    # True
print(isinstance(object, type))    # True
```

---

## Summary Cheat Sheet

- Variables are **labels** bound to objects; no declaration keyword is needed.
- Python is **dynamically** (runtime type checking) and **strongly** (no unsafe implicit coercion) typed.
- Core scalar types: `int`, `float`, `complex`, `str`, `bool` (subclass of `int`), and `NoneType` (`None`).
- Prefer `isinstance(x, T)` over `type(x) == T`, and check `x is None` rather than `x == None`.
- Constants use `UPPER_SNAKE_CASE` by convention (and optional `typing.Final` for static checkers).
- Swap cleanly with `a, b = b, a`; avoid `a = b = []` for mutable objects.
- `==` checks **value equality**; `is` checks **object identity** (`id()`).
- **Immutable** (`int`, `float`, `str`, `tuple`, `bool`, `frozenset`) vs **Mutable** (`list`, `dict`, `set`, `bytearray`).
- Every entity in Python—including numbers, functions, and classes—is an **object** with identity, type, and value.
