# 02. Variables and Data Types (Easy Guide)

---

## 1️⃣ What is a variable?
A variable is just a **name** that points to a value.

```python
name = "Alice"   # "name" now points to the string "Alice"
age = 30         # "age" points to the integer 30
```
You **don’t** need to declare the type first – the type is decided by the value you assign.

## 2️⃣ Naming rules (what you *can* and *cannot* do)
| Rule | Example |
|------|---------|
| Only letters, numbers and **_** (underscore) | `user_name`, `score2`
| Cannot start with a number | `2score` **❌**
| No spaces or symbols like `-`, `@` | `my-var` **❌**
| Case‑sensitive (`age` ≠ `Age`) | `age`, `Age`
| Cannot use Python keywords (`if`, `class`, `True`…) | `class` **❌**

## 3️⃣ PEP 8 style (recommended naming)
| Kind | Style |
|------|-------|
| Normal variables & functions | `snake_case` (e.g. `total_price`)
| Constants | `UPPER_SNAKE_CASE` (e.g. `MAX_USERS`)
| Classes | `PascalCase` (e.g. `BankAccount`)
| “private” variables | start with a single underscore `_hidden`

## 4️⃣ Dynamic typing
Python decides the type **when the program runs**.
You can re‑assign a variable to a completely different type.

```python
x = 10          # x is an int now
x = "hello"    # x becomes a string
```
Because of this, Python is **strongly typed** – it won’t silently mix types:
```python
"5" + 2   # ❌ TypeError
```

## 5️⃣ Basic data types you’ll see
| Type | Example values |
|------|----------------|
| `int` | `0`, `-7`, `12345` |
| `float` | `3.14`, `-0.001`, `2e5` |
| `complex` | `2+3j` |
| `str` | `'hello'`, `"world"` |
| `bool` | `True`, `False` |
| `None` | `None` (means “nothing”) |

## 6️⃣ Numbers in a nutshell
* **Integers** (`int`) – whole numbers, unlimited size.
* **Floats** (`float`) – decimal numbers, stored in binary (tiny rounding errors sometimes).
* **Complex** (`complex`) – written like `a + bj` (use `j` for the imaginary part).

### Simple operations
```python
a = 5 + 2      # 7
b = 5 / 2      # 2.5 (always a float)
c = 5 // 2     # 2   (floor division, drops the fraction)
```

## 7️⃣ Boolean values
`True` and `False` are the only two boolean values.
They behave like numbers (`True` == `1`, `False` == `0`).
Use them in conditions:
```python
if is_ready:
    print("Go!")
```

## 8️⃣ Checking a value’s type
* `type(x)` – gives the exact type (no inheritance).
* `isinstance(x, T)` – checks if `x` is of type `T` *or* a subclass of `T`. **Preferred**.
```python
isinstance(5, int)          # True
isinstance(True, int)      # True (bool is a subclass of int)
```

## 9️⃣ Converting / casting
Use the constructor functions:
```python
int("42")      # 42
float("3.14")  # 3.14
str(100)        # "100"
bool("")       # False
```
Be careful – converting a non‑numeric string raises `ValueError`.

## 🔟 Constants (values you don’t intend to change)
Python has no built‑in `const`. By convention, write them in **UPPER_SNAKE_CASE**:
```python
PI = 3.14159
MAX_RETRIES = 5
```
Static type checkers can enforce this with `typing.Final`, but at runtime it’s just a convention.

## 1️⃣1️⃣ Multiple assignment & swapping
Assign several variables at once:
```python
x, y, z = 1, 2, "three"
```
Swap without a temporary variable:
```python
a, b = b, a
```
## 1️⃣2️⃣ `id()` – object identity
Every object has a unique identity during its lifetime.
```python
a = [1, 2]
print(id(a))
a.append(3)
print(id(a))   # same id – list was modified in place
```
Use `is` to compare identity (`a is b`). Use `==` for value equality.

## 1️⃣3️⃣ Mutable vs immutable
| Immutable (cannot change) | Mutable (can change) |
|--------------------------|----------------------|
| `int`, `float`, `str`, `tuple`, `bool`, `frozenset` | `list`, `dict`, `set`, `bytearray` |

If you modify a mutable object, **all** variables that point to it see the change.
```python
lst1 = []
lst2 = lst1
lst1.append(1)
print(lst2)   # [1]
```
Create a copy when you want independent objects.

---

### Quick recap
* Variables are just names that point to values.
* Python decides the type at runtime – you can change it anytime.
* Follow simple naming rules and **PEP 8** style.
* Use `isinstance` for type checks, and constructor functions for conversion.
* Remember the mutable vs immutable distinction – it affects how data is shared.

Happy coding! 🎉