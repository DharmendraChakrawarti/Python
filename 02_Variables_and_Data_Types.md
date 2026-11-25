# 02. Variables and Data Types (Friendly & Complete Guide)

---

## 1️⃣ What is a variable?
A **variable** is just a name that points to a value (an object) in memory. Python creates the variable the moment you assign something to it – no separate declaration needed.

```python
name = "Alice"   # "name" now points to the string "Alice"
age = 30         # "age" points to the integer 30
is_student = False
```
Think of a variable as a *label* you stick on a box; you can move the label to a different box later.

## 2️⃣ Naming rules (what you can and cannot do)
| Rule | Correct example | Incorrect example |
|------|----------------|-------------------|
| Only letters, digits, underscores | `user_name`, `score2` | `2score` ❌ (starts with a digit) |
| No spaces or special symbols | `_private_flag` | `my-var` ❌ (hyphen) |
| Case‑sensitive | `age` vs `Age` are different | — |
| Cannot use Python keywords | `class_name` | `class` ❌ (reserved word) |
| Must start with a letter or underscore | `_temp`, `value` | `9value` ❌ |

## 3️⃣ PEP 8 naming conventions (recommended style)
| Kind | Style |
|------|-------|
| Regular variables & functions | `snake_case` (e.g., `total_price`) |
| Constants | `UPPER_SNAKE_CASE` (e.g., `MAX_USERS`) |
| Classes | `PascalCase` (e.g., `BankAccount`) |
| "Private" names (intended for internal use) | `_internal_id` |
| Avoid keyword clash | `class_`, `type_` |

## 4️⃣ Dynamic (runtime) typing
Python decides the **type** of a value when the program runs. You can re‑assign a variable to a completely different type:

```python
x = 10          # int
x = "ten"      # now a string
```
Python is **strongly typed** – it won’t silently mix unrelated types:
```python
"5" + 2   # ❌ TypeError
```
If you want static checking, add **type hints** (`x: int = 5`). They are only for linters, not enforced at runtime.

## 5️⃣ Core data types you’ll encounter
| Category | Type | Example literals |
|----------|------|------------------|
| Numbers | `int` | `0`, `-42`, `1_000_000` |
|          | `float` | `3.14`, `-0.5`, `2e-3` |
|          | `complex` | `2 + 3j`, `1j` |
| Text | `str` | `'hello'`, `"Python"`, `"""multi line"""` |
| Boolean | `bool` | `True`, `False` |
| Null / empty | `NoneType` | `None` |

## 6️⃣ Numbers in a nutshell
### Integers (`int`)
* Whole numbers, unlimited size (Python uses arbitrary‑precision).
* Underscores can improve readability: `1_000_000`.
* Binary (`0b`), octal (`0o`), hexadecimal (`0x`) prefixes are supported.

```python
a = 42
big = 10**50          # huge integer
readable = 1_000_000
binary = 0b1010       # 10
hex = 0xFF            # 255
```
### Floats (`float`)
* Stored as IEEE‑754 double precision.
* Supports scientific notation: `6.02e23`.
* Small rounding quirks: `0.1 + 0.2` yields `0.30000000000000004`.
* For exact decimal work (e.g., money) use `decimal.Decimal`.

```python
pi = 3.14159
avogadro = 6.022e23
tiny = 2.5e-4   # 0.00025
```
### Complex numbers (`complex`)
* Written as `a + bj` (use `j`, not `i`).
* Both parts are stored as floats.

```python
z = 3 + 4j
print(z.real)   # 3.0
print(z.imag)   # 4.0
print(abs(z))    # 5.0 (magnitude)
```

## 7️⃣ Boolean values (`bool`)
Only two values: `True` and `False`. They are **subclasses of `int`** (`True == 1`, `False == 0`). Use them in conditionals:

```python
if is_ready:
    print("Go!")
```
Logical operators:
* `and` – both sides truthy.
* `or` – at least one truthy.
* `not` – negates.

## 8️⃣ Checking a value’s type
* `type(obj)` – returns the exact class (no inheritance).
* `isinstance(obj, cls)` – checks if `obj` is an instance of `cls` **or any subclass**. This is the preferred way.

```python
isinstance(5, int)          # True
isinstance(True, int)       # True (bool inherits int)
isinstance(5, (float, str)) # False
```
Use `type(x) is int` only when you *must* differentiate `bool` from `int`.

## 9️⃣ Converting / casting values
Built‑in constructors do the job:

```python
int("42")          # 42
float("3.14")      # 3.14
str(100)            # "100"
bool("")           # False
```
Invalid conversions raise `ValueError` (e.g., `int("abc")`).

## 🔟 Constants (values you don’t intend to change)
Python has no `const` keyword. By convention, write constant names in **UPPER_SNAKE_CASE** at the top of the file:

```python
PI = 3.14159
MAX_RETRIES = 5
```
Static type checkers can enforce immutability with `typing.Final`:
```python
from typing import Final
MAX_USERS: Final[int] = 100
```
At runtime it’s still just a normal variable.

## 1️⃣1️⃣ Multiple assignment & swapping
Assign several variables in one line:
```python
x, y, z = 1, 2, "three"
```
Swap without a temporary variable:
```python
a, b = b, a
```
Be careful with chained assignment on mutable objects:
```python
list_a = list_b = []   # both names point to the same list!
list_a.append(1)
print(list_b)   # [1]
```
Use separate assignments (`list_a, list_b = [], []`) to avoid sharing.

## 1️⃣2️⃣ `id()` – object identity
Every object has a unique identity (its memory address in CPython). Use `id()` to see it, and `is` to compare identities.
```python
a = [1, 2, 3]
print(id(a))
b = a
print(a is b)   # True – same object
c = a.copy()
print(a is c)   # False – different object
```
`==` checks **value equality**, `is` checks **identity**.

## 1️⃣3️⃣ Mutable vs immutable
| Immutable (cannot change) | Mutable (can change) |
|---------------------------|----------------------|
| `int`, `float`, `bool`, `str`, `tuple`, `frozenset`, `bytes`, `NoneType` | `list`, `dict`, `set`, `bytearray` |

If you modify a mutable object, **all** variables referencing it see the change.
```python
lst1 = []
lst2 = lst1
lst1.append(5)
print(lst2)   # [5]
```
Create a copy (`lst1.copy()`, `list(lst1)`, or `copy.deepcopy`) when you need an independent object.

---

### Quick recap (in plain language)
* **Variable** = a name pointing to a value.
* No need to declare types – Python works out the type when you assign.
* Follow simple naming rules (letters, digits, underscores) and PEP 8 style.
* Use `isinstance` to check types, not `type()`.
* Convert with `int()`, `float()`, `str()`, etc.
* Constants are just conventionally‑named uppercase variables.
* Multiple assignment and swapping are clean and Pythonic.
* `id()` shows the object's identity; `is` compares identities.
* Know which types are mutable and which are immutable – it affects how data is shared.

Happy coding! 🎉