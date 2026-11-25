# Operators in Python

Operators are special symbols or keywords in Python that perform operations on one or more values (**operands**) and return a result. Mastering Python's operators—along with their subtleties around truthiness, object identity, short-circuiting, and precedence—is essential for writing clean, bug-free, and Pythonic code.

---

## Table of Contents

1. [Arithmetic Operators](#1-arithmetic-operators)
2. [Comparison / Relational Operators](#2-comparison--relational-operators)
3. [Logical Operators](#3-logical-operators)
4. [Assignment Operators](#4-assignment-operators)
5. [Bitwise Operators](#5-bitwise-operators)
6. [Membership Operators](#6-membership-operators)
7. [Identity Operators](#7-identity-operators)
8. [Ternary / Conditional Operator](#8-ternary--conditional-operator)
9. [Operator Precedence and Associativity](#9-operator-precedence-and-associativity)
10. [Walrus Operator (`:=`) — Python 3.8+](#10-walrus-operator---python-38)
11. [Chained Comparisons](#11-chained-comparisons)
12. [Short-Circuit Evaluation](#12-short-circuit-evaluation)

---

## 1. Arithmetic Operators

Arithmetic operators perform mathematical operations on numeric types (`int`, `float`, `complex`) and, in a few cases, sequence types (`str`, `list`, `tuple`).

| Operator | Name | Description | Example | Result |
| :--- | :--- | :--- | :--- | :--- |
| `+` | Addition | Adds two operands | `10 + 3` | `13` |
| `-` | Subtraction | Subtracts right operand from left | `10 - 3` | `7` |
| `*` | Multiplication | Multiplies two operands | `10 * 3` | `30` |
| `/` | True Division | Divides and **always** returns a `float` | `10 / 2` | `5.0` |
| `//` | Floor Division | Divides and rounds **down** toward $-\infty$ | `10 // 3` | `3` |
| `%` | Modulus | Returns the remainder of division | `10 % 3` | `1` |
| `**` | Exponentiation | Raises left operand to the power of right | `2 ** 3` | `8` |

> **Note:** Python also supports unary `+x` (identity/numeric promotion), unary `-x` (negation), and the matrix multiplication operator `@` (used primarily by libraries like NumPy).

### Basic Examples

```python
a, b = 17, 5

print(a + b)   # 22
print(a - b)   # 12
print(a * b)   # 85
print(a / b)   # 3.4   (always returns float)
print(a // b)  # 3     (floor division)
print(a % b)   # 2     (remainder)
print(a ** b)  # 1419857 (17^5)
```

### Important Nuances & Edge Cases

1. **True division (`/`) vs. Floor division (`//`)**:
   - `/` always returns a `float`, even when dividing evenly (`8 / 2` is `4.0`).
   - `//` returns an `int` if both operands are `int`, and a `float` if either operand is a `float` (`10.0 // 3` is `3.0`).
   - With negative numbers, `//` floors toward **negative infinity**, not toward zero:
     ```python
     print(7 // 2)    # 3
     print(-7 // 2)   # -4  (since -3.5 rounded down toward -inf is -4)
     print(7 // -2)   # -4
     ```

2. **Modulus (`%`) with negative numbers**:
   - Python guarantees the identity: `a == (a // b) * b + (a % b)`.
   - The sign of `a % b` always matches the sign of the **divisor** (`b`):
     ```python
     print(-7 % 2)    # 1   ((-4 * 2) + 1 == -7)
     print(7 % -2)    # -1  ((-4 * -2) + (-1) == 7)
     ```

3. **Zero division**:
   - Dividing or taking modulo by `0` or `0.0` raises `ZeroDivisionError`.

4. **`divmod()` built-in**:
   - Returns `(a // b, a % b)` in a single operation:
     ```python
     quotient, remainder = divmod(17, 5)  # (3, 2)
     ```

5. **Arithmetic operators on sequences**:
   - `+` concatenates sequences of the same type; `*` repeats a sequence:
     ```python
     print("Hi " + "there!")  # "Hi there!"
     print("Ha" * 3)          # "HaHaHa"
     print([1, 2] * 2)        # [1, 2, 1, 2]
     ```

---

## 2. Comparison / Relational Operators

Comparison operators compare two values and evaluate to a `bool` (`True` or `False`).

| Operator | Meaning | Example | Result |
| :--- | :--- | :--- | :--- |
| `==` | Equal to | `5 == 5` | `True` |
| `!=` | Not equal to | `5 != 3` | `True` |
| `>` | Greater than | `5 > 3` | `True` |
| `<` | Less than | `5 < 3` | `False` |
| `>=` | Greater than or equal to | `5 >= 5` | `True` |
| `<=` | Less than or equal to | `3 <= 5` | `True` |

### Examples

```python
x, y = 10, 20

print(x == y)   # False
print(x != y)   # True
print(x < y)    # True
print(x >= 10)  # True
```

### Important Nuances & Edge Cases

1. **Cross-type comparisons**:
   - Numbers of different numeric types can be compared (`5 == 5.0` is `True`, `1 == True` is `True` because `bool` subclasses `int`).
   - Comparing incompatible types with `==` or `!=` returns `False` / `True` (`5 == "5"` is `False`).
   - Comparing incompatible types with `<`, `<=`, `>`, `>=` raises `TypeError` (`5 < "5"` raises `TypeError`).

2. **Lexicographical comparisons on strings & sequences**:
   - Strings compare character-by-character using Unicode code points (`ord()`):
     ```python
     print("apple" < "banana")  # True
     print("Apple" < "apple")   # True ('A' is 65, 'a' is 97)
     ```
   - Lists and tuples compare element-by-element from left to right:
     ```python
     print([1, 2, 3] < [1, 2, 4])  # True
     print((1, 2) < (1, 2, 0))     # True (shorter prefix is smaller)
     ```

3. **Floating-point precision**:
   - Due to binary floating-point representation, exact `==` checks on floats can fail unexpectedly. Use `math.isclose()` instead:
     ```python
     import math

     print(0.1 + 0.2 == 0.3)             # False (0.30000000000000004)
     print(math.isclose(0.1 + 0.2, 0.3)) # True
     ```

---

## 3. Logical Operators

Logical operators combine or invert boolean/truthy expressions.

| Operator | Description | Example | Result |
| :--- | :--- | :--- | :--- |
| `and` | True if **both** operands are truthy | `True and False` | `False` |
| `or` | True if **at least one** operand is truthy | `True or False` | `True` |
| `not` | Inverts truthiness (always returns `bool`) | `not True` | `False` |

### Truthiness in Python

Every Python object can be evaluated in a boolean context (`bool(obj)`).
The following values are **falsy** (evaluate to `False`):
- `False`, `None`
- Numeric zeros: `0`, `0.0`, `0j`, `Decimal(0)`, `Fraction(0, 1)`
- Empty sequences/collections: `""`, `b""`, `[]`, `()`, `{}`, `set()`, `range(0)`
- Custom objects whose `__bool__()` returns `False` or `__len__()` returns `0`

Almost everything else is **truthy** (including non-empty containers, non-zero numbers, and functions).

### Return Values of `and`, `or`, and `not`

Crucially, `and` and `or` **do not always return a `bool`**—they return **one of the actual operands** evaluated!

- `x and y`: returns `x` if `x` is falsy; otherwise returns `y`.
- `x or y`: returns `x` if `x` is truthy; otherwise returns `y`.
- `not x`: always returns `True` or `False` (`bool`).

```python
print(True and False)    # False
print(5 and 10)          # 10   (5 is truthy, returns second operand)
print(0 and 10)          # 0    (0 is falsy, returns first operand)

print("" or "default")   # "default" ("" is falsy, returns second operand)
print("Alice" or "Bob")  # "Alice"   ("Alice" is truthy, returns first operand)

print(not [])            # True
print(not "hello")       # False
```

---

## 4. Assignment Operators

Assignment operators bind names (variables) to objects. Augmented assignment operators perform an operation and reassign in a single statement.

| Operator | Equivalent To | Example |
| :--- | :--- | :--- |
| `=` | Simple assignment | `x = 5` |
| `+=` | `x = x + y` | `x += 3` |
| `-=` | `x = x - y` | `x -= 3` |
| `*=` | `x = x * y` | `x *= 3` |
| `/=` | `x = x / y` | `x /= 2` |
| `//=` | `x = x // y` | `x //= 2` |
| `%=` | `x = x % y` | `x %= 2` |
| `**=` | `x = x ** y` | `x **= 2` |
| `&=` | `x = x & y` | `x &= 1` |
| `\|=` | `x = x \| y` | `x \|= 2` |
| `^=` | `x = x ^ y` | `x ^= 1` |
| `>>=` | `x = x >> y` | `x >>= 1` |
| `<<=` | `x = x << y` | `x <<= 1` |
| `@=` | `x = x @ y` | `mat @= other` |

### Examples

```python
score = 10
score += 5    # 15
score *= 2    # 30
score //= 4   # 7
score **= 2   # 49
print(score)  # 49

# Chained and tuple-unpacking assignments
a = b = c = 0
x, y = 1, 2
first, *rest = [10, 20, 30, 40]  # first = 10, rest = [20, 30, 40]
```

### Mutable vs. Immutable Behavior (`+=` vs. `+`)

For **immutable** types (`int`, `str`, `tuple`), `x += y` creates a new object and rebinds `x`.
For **mutable** types like `list`, `x += y` invokes `__iadd__` and mutates the list **in-place** (unlike `x = x + y`, which creates a new list):

```python
a = [1, 2]
b = a         # b references the same list as a
a += [3, 4]   # mutates the list in place
print(b)      # [1, 2, 3, 4]

c = [1, 2]
d = c
c = c + [3, 4]  # creates a new list and rebinds c
print(d)        # [1, 2]
```

---

## 5. Bitwise Operators

Bitwise operators act directly on the binary (bit-level) representation of integers.

| Operator | Name | Description | Example (`a = 6` (`0b110`), `b = 3` (`0b011`)) | Result |
| :--- | :--- | :--- | :--- | :--- |
| `&` | Bitwise AND | `1` if both bits are `1` | `6 & 3` (`0b110 & 0b011`) | `2` (`0b010`) |
| `\|` | Bitwise OR | `1` if at least one bit is `1` | `6 \| 3` (`0b110 \| 0b011`) | `7` (`0b111`) |
| `^` | Bitwise XOR | `1` if bits differ | `6 ^ 3` (`0b110 ^ 0b011`) | `5` (`0b101`) |
| `~` | Bitwise NOT | Inverts all bits (`-(x + 1)`) | `~6` | `-7` |
| `<<` | Left Shift | Shifts bits left (fills with `0`) | `6 << 2` | `24` (`0b11000`) |
| `>>` | Right Shift | Shifts bits right (sign-extending) | `6 >> 1` | `3` (`0b011`) |

### Examples & Common Idioms

```python
a = 0b1010  # 10
b = 0b1100  # 12

print(bin(a & b), a & b)    # 0b1000 8
print(bin(a | b), a | b)    # 0b1110 14
print(bin(a ^ b), a ^ b)    # 0b110  6
print(~a)                   # -11 (two's complement: -(10 + 1))
print(a << 2)               # 40  (10 * 2^2)
print(a >> 1)               # 5   (10 // 2^1)
```

### Practical Use Cases & Gotchas

- **Check even/odd**: `n & 1` is `1` if `n` is odd, `0` if even.
- **Bitmasks / Permissions**:
  ```python
  READ, WRITE, EXEC = 0b100, 0b010, 0b001
  perms = READ | WRITE       # 0b110 (enable READ and WRITE)
  can_write = bool(perms & WRITE)  # True
  perms &= ~WRITE            # clear WRITE bit -> 0b100
  perms ^= EXEC              # toggle EXEC bit -> 0b101
  ```
- **Common Gotcha (`^` is NOT exponentiation!)**: In Python, `2 ^ 3` is `1` (XOR), **not** `8`. Always use `**` for exponentiation.
- **Why `~x == -(x + 1)`**: Python integers have arbitrary precision (infinite conceptual sign bits in two's complement), so flipping every bit of `x` yields `-(x + 1)`.

---

## 6. Membership Operators

Membership operators test whether a value exists inside a sequence, collection, or mapping.

| Operator | Description | Example | Result |
| :--- | :--- | :--- | :--- |
| `in` | `True` if value is found | `'a' in 'cat'` | `True` |
| `not in` | `True` if value is **not** found | `4 not in [1, 2, 3]` | `True` |

### Examples

```python
# Strings (checks for substring)
print("py" in "python")        # True
print("Java" not in "python")  # True

# Lists and Tuples (linear search: O(n))
fruits = ["apple", "banana", "cherry"]
print("banana" in fruits)      # True

# Sets (hash lookup: O(1) average)
allowed = {"read", "write"}
print("read" in allowed)       # True

# Dictionaries (checks KEYS by default, O(1) average)
user = {"name": "Ada", "role": "admin"}
print("name" in user)              # True  (key check)
print("Ada" in user)               # False (not a key!)
print("Ada" in user.values())      # True  (value check)
print(("role", "admin") in user.items())  # True
```

> **Performance Tip:** Use `set` or `dict` instead of `list` or `tuple` when performing repeated `in` checks on large collections (`O(1)` vs. `O(n)`).

---

## 7. Identity Operators

Identity operators compare **memory addresses / object identity** (`id(a) == id(b)`), checking whether two variables refer to the exact same object in memory—not just whether they hold equal values.

| Operator | Description | Example |
| :--- | :--- | :--- |
| `is` | `True` if both operands refer to the **same object** | `x is None` |
| `is not` | `True` if operands refer to **different objects** | `x is not None` |

### `is` vs. `==`

- `==` checks **value equality** (calls `__eq__`).
- `is` checks **object identity** (cannot be overridden; compares `id()`).

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)  # True  (same contents)
print(a is b)  # False (two distinct list objects in memory)
print(a is c)  # True  (both names point to the same list object)
```

### Best Practices & Pitfalls

1. **Always use `is` / `is not` with singletons (`None`, `True`, `False`)**:
   ```python
   # Recommended (PEP 8)
   if result is None:
       print("No result")

   # Avoid
   if result == None:
       ...
   ```
   Using `is None` is faster and safe even if a custom object overrides `__eq__`.

2. **Never use `is` to compare numbers or strings**:
   CPython caches small integers (`-5` to `256`) and interns certain strings as an internal optimization. Relying on `is` for values leads to subtle bugs:
   ```python
   x = 256
   y = 256
   print(x is y)  # True (cached small int)

   p = 1000
   q = 1000
   # May be False in separate statements/REPL lines! Always use ==
   print(p == q)  # True
   ```

---

## 8. Ternary / Conditional Operator

Python's conditional expression provides a concise single-line `if / else` expression.

### Syntax

```python
value_if_true if condition else value_if_false
```

### Examples

```python
age = 20
status = "Adult" if age >= 18 else "Minor"
print(status)  # "Adult"

# Computing a value inline
a, b = 15, 22
maximum = a if a > b else b
print(maximum)  # 22
```

### Important Behavior

1. **Lazy evaluation**: Only the selected branch is evaluated. The unchosen branch is never executed:
   ```python
   divisor = 0
   result = (10 / divisor) if divisor != 0 else 0  # Safe! 10 / 0 is never evaluated
   ```

2. **Both `if` and `else` are required**: Unlike a statement `if`, a conditional expression must always include `else` because it must evaluate to a value.

3. **Chaining (use sparingly)**:
   You can chain ternary operators, though a standard `if / elif / else` block is usually clearer:
   ```python
   score = 85
   grade = "A" if score >= 90 else ("B" if score >= 80 else "C")
   ```

---

## 9. Operator Precedence and Associativity

When an expression contains multiple operators, **precedence** determines which operator is evaluated first, and **associativity** determines the grouping order when operators share the same precedence.

### Full Precedence Table (Highest to Lowest)

| Precedence | Operators | Description | Associativity |
| :--- | :--- | :--- | :--- |
| 1 (Highest) | `(expr...)`, `[expr...]`, `{k: v...}`, `{expr...}` | Parentheses, list/dict/set displays | Left-to-right |
| 2 | `x[i]`, `x[i:j]`, `x(args...)`, `x.attr` | Subscription, slicing, call, attribute access | Left-to-right |
| 3 | `await x` | Await expression | N/A |
| 4 | `**` | Exponentiation | **Right-to-left** |
| 5 | `+x`, `-x`, `~x` | Unary plus, unary minus, bitwise NOT | Right-to-left |
| 6 | `*`, `@`, `/`, `//`, `%` | Multiplication, matrix mult, division, floor div, modulus | Left-to-right |
| 7 | `+`, `-` | Addition, subtraction | Left-to-right |
| 8 | `<<`, `>>` | Bitwise left and right shifts | Left-to-right |
| 9 | `&` | Bitwise AND | Left-to-right |
| 10 | `^` | Bitwise XOR | Left-to-right |
| 11 | `\|` | Bitwise OR | Left-to-right |
| 12 | `in`, `not in`, `is`, `is not`, `<`, `<=`, `>`, `>=`, `!=`, `==` | Comparisons, membership, identity | Left-to-right (chained) |
| 13 | `not x` | Boolean NOT | Right-to-left |
| 14 | `and` | Boolean AND | Left-to-right |
| 15 | `or` | Boolean OR | Left-to-right |
| 16 | `if ... else` | Conditional (ternary) expression | **Right-to-left** |
| 17 | `lambda` | Lambda expression | Right-to-left |
| 18 (Lowest) | `:=` | Assignment expression (Walrus operator) | Right-to-left |

### Key Examples & Surprises

```python
# 1. Multiplication before addition
print(2 + 3 * 4)        # 14 (not 20)

# 2. Exponentiation is RIGHT-associative
print(2 ** 3 ** 2)      # 512 -> 2 ** (3 ** 2) == 2 ** 9, NOT (2 ** 3) ** 2 == 64

# 3. Exponentiation vs. Unary minus
print(-2 ** 2)          # -4  -> -(2 ** 2)
print((-2) ** 2)        # 4
print(2 ** -2)          # 0.25 -> 2 ** (-2) (unary binds tighter on the right side of **)

# 4. not > and > or
print(True or False and False)      # True  -> True or (False and False)
print(not False and False)          # False -> (not False) and False

# 5. Comparisons bind tighter than boolean operators
print(1 < 2 and 3 < 4)              # True  -> (1 < 2) and (3 < 4)
```

> **Best Practice:** Use parentheses `(...)` whenever mixing operators from different groups (especially bitwise + arithmetic, or `and` + `or`) to make intent unmistakable.

---

## 10. Walrus Operator (`:=`) — Python 3.8+

The **assignment expression** operator (`:=`), nicknamed the **walrus operator** because it resembles a walrus's eyes and tusks, assigns a value to a variable **as part of an expression** and returns that value.

### Syntax

```python
(name := expr)
```

### Common Use Cases

#### 1. Avoiding redundant calls in `if` statements

```python
import re

text = "Order #4821 confirmed"
if (match := re.search(r"#(\d+)", text)):
    print(f"Found order ID: {match.group(1)}")
```

#### 2. Streamlining `while` loops (loop-and-a-half pattern)

```python
# Without walrus:
# line = input("Enter text (or 'quit'): ")
# while line != "quit":
#     print(f"You typed: {line}")
#     line = input("Enter text (or 'quit'): ")

# With walrus:
while (line := input("Enter text (or 'quit'): ")) != "quit":
    print(f"You typed: {line}")
```

#### 3. Reusing expensive computations in comprehensions

```python
def slow_square(n):
    return n * n

numbers = [1, 2, 3, 4, 5, 6]

# slow_square(x) is called ONLY ONCE per item
large_squares = [sq for x in numbers if (sq := slow_square(x)) > 10]
print(large_squares)  # [16, 25, 36]
```

#### 4. Assigning and checking length or lookup

```python
items = [10, 20, 30, 40]
if (n := len(items)) > 3:
    print(f"List is too long ({n} items)")
```

### Rules & Restrictions

1. **Parentheses are required at top level**: `x := 5` is a `SyntaxError`; use `x = 5` for normal assignment, or `(x := 5)` if used as an expression.
2. **Low precedence**: Always parenthesize `:=` inside comparisons, e.g., `while (n := len(data)) > 0:`. Without parentheses, `n := (len(data) > 0)` would assign a `bool` to `n`!
3. **Only simple variable names**: You cannot use `:=` with attribute assignment (`obj.x := 1`), subscript assignment (`lst[0] := 1`), or tuple unpacking (`a, b := 1, 2`).
4. **Comprehension scope**: A variable bound by `:=` inside a list/set/dict comprehension leaks into the enclosing function/module scope (whereas the loop variable itself stays local to the comprehension).

---

## 11. Chained Comparisons

Python allows chaining any number of comparison, membership, or identity operators together cleanly—just like mathematical notation.

### Syntax & Semantics

`a op1 b op2 c` is equivalent to:

```python
(a op1 b) and (b op2 c)
```

...with one important bonus: **`b` is evaluated only once** (and short-circuiting applies if `a op1 b` is `False`).

### Examples

```python
x = 5

# Mathematical range check
print(1 < x < 10)          # True  (1 < x and x < 10)
print(0 <= x <= 5)         # True

# Multiple equality
a = b = c = 42
print(a == b == c)         # True

# Mixed comparison operators
print(1 < 3 > 2)           # True  (1 < 3 and 3 > 2)
print(1 < 2 < 3 < 4)       # True

# Single evaluation of the middle operand
def get_val():
    print("called!")
    return 5

print(1 < get_val() < 10)  # Prints "called!" ONLY ONCE, then True
```

### Watch Out for Unintended Chaining!

Because `in`, `not in`, `is`, `is not`, `==`, `!=` are all in the same comparison precedence group, chaining them can produce surprising results:

```python
# Evaluates as: (False == False) and (False in [False]) -> True and True -> True
print(False == False in [False])  # True!

# Evaluates as: (1 < 2) and (2 == True) -> True and False -> False
print(1 < 2 == True)              # False! (even though (1 < 2) == True is True)
```

Always add parentheses when mixing `==`/`!=` with `in` or when comparing the boolean result of a comparison.

---

## 12. Short-Circuit Evaluation

**Short-circuit evaluation** means Python stops evaluating a boolean expression as soon as the final outcome is already guaranteed.

- For `A and B`: if `A` is **falsy**, the entire expression cannot be truthy, so `B` is **never evaluated** and `A` is returned.
- For `A or B`: if `A` is **truthy**, the entire expression is already truthy, so `B` is **never evaluated** and `A` is returned.

### Practical Examples

#### 1. Guarding against errors (`ZeroDivisionError`, `AttributeError`, `IndexError`)

```python
divisor = 0
if divisor != 0 and (10 / divisor) > 1:
    print("Valid")
# Safe: (10 / divisor) is never executed because divisor != 0 is False

user = None
if user is not None and user.is_active:
    print("Active user")
# Safe: user.is_active is never accessed when user is None

items = []
if items and items[0] == "start":
    print("Starts properly")
# Safe: items[0] is never accessed on an empty list
```

#### 2. Skipping expensive function calls

```python
def expensive_check():
    print("Running expensive check...")
    return True

is_cached = True
if is_cached or expensive_check():
    print("Done!")
# Output:
# Done!
# ("Running expensive check..." is NOT printed)
```

#### 3. Providing fallback defaults with `or`

```python
user_input = ""
username = user_input or "Guest"
print(username)  # "Guest"
```

> **Caveat with `or` defaults:** Because `or` checks truthiness rather than `is None`, valid falsy values like `0`, `0.0`, `""`, or `False` will trigger the fallback! When `0` or `""` is a valid value, use `x if x is not None else default` instead:
> ```python
> count = 0
> print(count or 10)                  # 10 (bug if 0 is valid!)
> print(count if count is not None else 10)  # 0 (safe)
> ```

#### 4. Short-circuiting in `any()` and `all()`

Built-in functions `any()` and `all()` also short-circuit when given a generator expression:

```python
nums = [2, 4, 6, 7, 8]

# Stops immediately at 7 without checking 8
has_odd = any(n % 2 != 0 for n in nums)  # True

# Stops immediately at 7
all_even = all(n % 2 == 0 for n in nums)  # False
```

---

## Quick Reference Summary

- **Division**: `/` always returns `float`; `//` floors toward $-\infty$; `%` sign follows the divisor.
- **Equality vs. Identity**: `==` compares values; `is` compares object identity (reserve `is` for `None`, `True`, `False`, and sentinels).
- **Logical operators**: `and` / `or` short-circuit and return an operand; `not` always returns `bool`.
- **Exponentiation**: `**` is right-associative and binds tighter than unary `-` on the left (`-2 ** 2 == -4`).
- **Bitwise XOR**: Use `^` for bitwise XOR and `**` for exponentiation.
- **Walrus (`:=`)**: Assigns and returns a value inside expressions; parenthesize carefully.
- **Chained comparisons**: `a < b < c` evaluates `b` once and checks `(a < b) and (b < c)`.
