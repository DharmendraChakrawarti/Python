# Control Flow Statements in Python

Control flow determines the order in which statements in a Python program execute. By default, Python runs code sequentially from top to bottom. **Control flow statements** allow you to make decisions (branching), repeat actions (looping), and alter loop execution dynamically.

---

## Table of Contents

1. [`if` Statement](#1-if-statement)
2. [`if-else` Statement](#2-if-else-statement)
3. [`if-elif-else` Ladder](#3-if-elif-else-ladder)
4. [Nested `if` Statements](#4-nested-if-statements)
5. [`match-case` Statement (Python 3.10+)](#5-match-case-statement-python-310)
6. [`for` Loop (Iterating Over Sequences & `range()`)](#6-for-loop-iterating-over-sequences--range)
7. [`while` Loop](#7-while-loop)
8. [Nested Loops](#8-nested-loops)
9. [`break` Statement](#9-break-statement)
10. [`continue` Statement](#10-continue-statement)
11. [`pass` Statement](#11-pass-statement)
12. [`else` Clause with Loops (`for-else`, `while-else`)](#12-else-clause-with-loops-for-else-while-else)
13. [Loop Control Techniques](#13-loop-control-techniques)
14. [Infinite Loops and How to Handle Them](#14-infinite-loops-and-how-to-handle-them)
15. [`enumerate()` and `zip()` in Loops](#15-enumerate-and-zip-in-loops)

---

## 1. `if` Statement

The `if` statement evaluates a boolean condition (or any expression with a truthy/falsy value). If the condition evaluates to `True`, the indented block of code beneath it runs; otherwise, Python skips the block.

### Syntax

```python
if condition:
    # Code executed when condition is True
```

> **Note on Indentation:** Python uses indentation (conventionally 4 spaces) rather than braces `{}` to define code blocks.

### Example

```python
temperature = 32

if temperature > 30:
    print("It is hot outside.")
    print("Stay hydrated!")

print("Forecast complete.")
# Output:
# It is hot outside.
# Stay hydrated!
# Forecast complete.
```

### Truthiness in Conditions

Python evaluates non-boolean objects in `if` conditions automatically:

- **Falsy values:** `False`, `None`, `0`, `0.0`, `""`, `[]`, `()`, `{}`, `set()`
- **Truthy values:** Most other objects (non-zero numbers, non-empty collections/strings)

```python
items = ["apple", "banana"]

if items:
    print(f"Cart has {len(items)} items.")
# Output: Cart has 2 items.
```

---

## 2. `if-else` Statement

The `if-else` statement provides two mutually exclusive paths: one executes when the condition is `True`, and the other executes when the condition is `False`.

### Syntax

```python
if condition:
    # Runs if condition is True
else:
    # Runs if condition is False
```

### Example

```python
age = 16

if age >= 18:
    print("Eligible to vote.")
else:
    print("Not eligible to vote yet.")
# Output: Not eligible to vote yet.
```

### Conditional Expression (Ternary Operator)

For simple `if-else` value assignments, Python offers an inline conditional expression:

```python
age = 20
status = "Adult" if age >= 18 else "Minor"
print(status)  # Output: Adult
```

---

## 3. `if-elif-else` Ladder

When you need to test multiple conditions in sequence, use `if-elif-else` (`elif` stands for "else if"). Python evaluates conditions from top to bottom and executes **only the first** branch whose condition is `True`. If none are `True`, the optional `else` block runs.

### Syntax

```python
if condition_1:
    # Block 1
elif condition_2:
    # Block 2
elif condition_3:
    # Block 3
else:
    # Fallback block
```

### Example

```python
score = 84

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"

print(f"Grade: {grade}")  # Output: Grade: B
```

> **Tip:** Order matters! Put more specific or higher-threshold conditions before broader ones, because subsequent `elif` branches are skipped once a match is found.

---

## 4. Nested `if` Statements

A nested `if` statement is an `if` (or `if-else`) placed inside another `if` or `else` block. This allows you to check secondary conditions only after an outer condition succeeds.

### Example

```python
has_ticket = True
age = 14
has_adult_companion = True

if has_ticket:
    if age >= 18:
        print("Entry granted.")
    elif has_adult_companion:
        print("Entry granted with adult companion.")
    else:
        print("Entry denied: underage without companion.")
else:
    print("Entry denied: ticket required.")
# Output: Entry granted with adult companion.
```

### Avoiding Deep Nesting (Guard Clauses)

Excessive nesting hurts readability. When possible, combine conditions with `and`/`or` or use early returns (guard clauses) inside functions:

```python
def check_entry(has_ticket, age, has_adult_companion):
    if not has_ticket:
        return "Entry denied: ticket required."
    if age >= 18 or has_adult_companion:
        return "Entry granted."
    return "Entry denied: underage without companion."
```

---

## 5. `match-case` Statement (Python 3.10+)

Introduced in Python 3.10 (PEP 634), `match-case` implements **structural pattern matching**. Unlike a basic C/Java `switch` statement, Python's `match-case`:
- Does **not** fall through between cases (no `break` needed).
- Can match literals, sequences, mappings, class attributes, and bind variables.
- Supports guard clauses (`if ...`) on patterns.

### Syntax

```python
match subject:
    case pattern_1:
        ...
    case pattern_2 if guard_condition:
        ...
    case _:
        # Wildcard / default case
```

### Example 1: Matching Literals and OR Patterns (`|`)

```python
http_status = 404

match http_status:
    case 200 | 201:
        print("Success")
    case 400:
        print("Bad Request")
    case 404:
        print("Not Found")
    case 500 | 502 | 503:
        print("Server Error")
    case _:
        print(f"Other status: {http_status}")
# Output: Not Found
```

### Example 2: Structural Unpacking and Guards

```python
command = ("move", 10, -5)

match command:
    case ("quit",):
        print("Exiting program.")
    case ("move", x, y) if x == 0 and y == 0:
        print("No movement.")
    case ("move", x, y):
        print(f"Moving by dx={x}, dy={y}")
    case ("draw", *points):
        print(f"Drawing points: {points}")
    case _:
        print("Unknown command")
# Output: Moving by dx=10, dy=-5
```

### Example 3: Matching Dictionaries

```python
event = {"type": "click", "x": 120, "y": 45}

match event:
    case {"type": "click", "x": x, "y": y}:
        print(f"Clicked at ({x}, {y})")
    case {"type": "keypress", "key": key}:
        print(f"Pressed key: {key}")
    case _:
        print("Unhandled event")
# Output: Clicked at (120, 45)
```

---

## 6. `for` Loop (Iterating Over Sequences & `range()`)

Python's `for` loop iterates directly over items of any **iterable** (lists, tuples, strings, dictionaries, sets, generators, files, `range()`, etc.).

### Syntax

```python
for variable in iterable:
    # Loop body
```

### Iterating Over Common Types

```python
# 1. List
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# 2. String (character by character)
for ch in "Py":
    print(ch)  # P, then y

# 3. Dictionary
scores = {"Alice": 95, "Bob": 88}
for name, score in scores.items():
    print(f"{name}: {score}")
```

### Using `range()`

`range()` generates a memory-efficient sequence of integers:
- `range(stop)` -> `0` to `stop - 1`
- `range(start, stop)` -> `start` to `stop - 1`
- `range(start, stop, step)` -> increments (or decrements) by `step`

```python
# 0, 1, 2, 3, 4
for i in range(5):
    print(i, end=" ")
print()

# 2, 4, 6, 8
for i in range(2, 10, 2):
    print(i, end=" ")
print()

# Countdown: 5, 4, 3, 2, 1
for i in range(5, 0, -1):
    print(i, end=" ")
```

---

## 7. `while` Loop

A `while` loop repeats a block of code **as long as** a condition remains `True`. Use `while` when the number of iterations isn't known beforehand.

### Syntax

```python
while condition:
    # Loop body (must eventually make condition False or break)
```

### Example

```python
count = 1

while count <= 4:
    print(f"Count is {count}")
    count += 1
# Output:
# Count is 1
# Count is 2
# Count is 3
# Count is 4
```

### `for` vs `while`

| Feature | `for` loop | `while` loop |
|---|---|---|
| Primary use | Fixed / known iterables | Condition-driven repetition |
| Loop variable update | Automatic via iterator | Manual inside loop body |
| Risk of infinite loop | Low (unless iterable is infinite) | Higher if condition never becomes `False` |

---

## 8. Nested Loops

A nested loop is a loop inside another loop. For every single iteration of the outer loop, the inner loop runs to completion.

### Example 1: Multiplication Table

```python
for row in range(1, 4):
    for col in range(1, 4):
        print(f"{row * col:3}", end="")
    print()  # Newline after each row
# Output:
#   1  2  3
#   2  4  6
#   3  6  9
```

### Example 2: Flattening a 2D Matrix

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
]

flat = []
for row in matrix:
    for value in row:
        flat.append(value)

print(flat)  # Output: [1, 2, 3, 4, 5, 6]
```

> **Performance Note:** Nested loops multiply iteration counts (e.g., $O(n \times m)$). When breaking out of a nested loop, `break` only exits the **innermost** loop.

---

## 9. `break` Statement

The `break` statement immediately terminates the **innermost enclosing loop** (`for` or `while`) and transfers execution to the statement immediately following the loop.

### Example

```python
numbers = [3, 7, 12, 18, 25]

for num in numbers:
    if num > 15:
        print(f"Found first number > 15: {num}")
        break
    print(f"Checked: {num}")
# Output:
# Checked: 3
# Checked: 7
# Checked: 12
# Found first number > 15: 18
```

---

## 10. `continue` Statement

The `continue` statement skips the remainder of the current iteration and jumps straight to the next iteration of the loop (`while` re-evaluates its condition; `for` advances to the next item).

### Example

```python
for num in range(1, 8):
    if num % 2 == 0:
        continue  # Skip even numbers
    print(num, end=" ")
# Output: 1 3 5 7
```

### `continue` with `while` Caution

Always update the loop counter **before** calling `continue` in a `while` loop, or you risk creating an infinite loop:

```python
i = 0
while i < 5:
    i += 1  # Update BEFORE continue
    if i == 3:
        continue
    print(i, end=" ")
# Output: 1 2 4 5
```

---

## 11. `pass` Statement

`pass` is a **null operation** (no-op). When executed, nothing happens. It acts as a syntactic placeholder where Python requires an indented block, such as empty loops, functions, classes, or branches under development.

### Example

```python
# Placeholder in conditional branches
x = 10
if x > 0:
    pass  # TODO: handle positive values later
else:
    print("Non-positive")

# Placeholder in a loop or function
def future_feature():
    pass

for _ in range(3):
    pass
```

> **Note:** Unlike `continue` (which skips to the next loop iteration), `pass` does nothing and continues executing any statements that follow it in the same block.

---

## 12. `else` Clause with Loops (`for-else`, `while-else`)

Python supports an optional `else` block on `for` and `while` loops. The `else` block runs **only if the loop finished normally** (i.e., was **not** exited via a `break` statement).

Think of `else` here as **"no-break"**.

### Example 1: Searching with `for-else`

```python
target = 42
data = [10, 20, 30, 40]

for item in data:
    if item == target:
        print("Target found!")
        break
else:
    print("Target was NOT found.")
# Output: Target was NOT found.
```

### Example 2: Prime Check

```python
n = 29

for divisor in range(2, int(n ** 0.5) + 1):
    if n % divisor == 0:
        print(f"{n} is not prime (divisible by {divisor}).")
        break
else:
    print(f"{n} is prime.")
# Output: 29 is prime.
```

### Example 3: `while-else`

```python
attempts = 3
while attempts > 0:
    pin = "1234"  # Simulated input
    if pin == "0000":
        print("Unlocked!")
        break
    attempts -= 1
else:
    print("Account locked: too many failed attempts.")
# Output: Account locked: too many failed attempts.
```

---

## 13. Loop Control Techniques

Beyond basic `break` and `continue`, several idiomatic techniques make Python loops cleaner, safer, and easier to control:

### 1. Breaking Out of Nested Loops

Because `break` only exits one loop level, you can handle multi-level exits cleanly in three ways:

```python
# Technique A: Wrap in a function and use `return` (cleanest)
def find_target(grid, target):
    for r, row in enumerate(grid):
        for c, val in enumerate(row):
            if val == target:
                return (r, c)
    return None

# Technique B: Flag variable
found = False
for i in range(3):
    for j in range(3):
        if i * j == 4:
            found = True
            break
    if found:
        break

# Technique C: for-else propagation
for i in range(3):
    for j in range(3):
        if i * j == 4:
            break
    else:
        continue  # Inner loop didn't break -> continue outer loop
    break         # Inner loop broke -> break outer loop
```

### 2. Comprehensions and `any()` / `all()` Instead of Manual Loops

```python
nums = [2, 4, 6, 8]

# Check conditions cleanly without manual break/flags
all_even = all(n % 2 == 0 for n in nums)     # True
has_gt_5 = any(n > 5 for n in nums)          # True

# Filter + transform with a list comprehension
squares_of_evens = [n ** 2 for n in range(10) if n % 2 == 0]
```

### 3. Walrus Operator (`:=`) in `while` Loops (Python 3.8+)

Avoids duplicating an initial read before a `while` loop ("loop-and-a-half" pattern):

```python
# Reading chunks or lines until empty
lines = ["first", "second", ""]
it = iter(lines)

while (line := next(it)) != "":
    print(f"Processing: {line}")
```

### 4. Iterating Safely Without Mutating the Sequence

Modifying a list while iterating over it causes skipped elements or bugs. Iterate over a **copy** (`items[:]`) or build a new list instead:

```python
nums = [1, 2, 3, 4, 5, 6]
# Safe: build filtered list
odd_nums = [n for n in nums if n % 2 != 0]
```

---

## 14. Infinite Loops and How to Handle Them

An **infinite loop** is a loop whose termination condition is never satisfied.

### Intentional Infinite Loops (`while True`)

Used when the exit check naturally happens in the middle of the loop (such as event loops, servers, or input validation):

```python
while True:
    user_input = input("Enter 'quit' to exit: ").strip().lower()
    if user_input == "quit":
        print("Goodbye!")
        break
    print(f"You entered: {user_input}")
```

### Accidental Infinite Loops (Common Causes)

1. Forgetting to update the loop variable:
   ```python
   i = 0
   while i < 5:
       print(i)
       # Missing `i += 1` -> runs forever!
   ```
2. Using `continue` before updating the loop variable.
3. Condition that can never become `False` (e.g., `while x != 10:` when `x` increments by 3 starting from 0).

### How to Prevent and Handle Infinite Loops

- **Enforce a maximum iteration guard (`max_attempts` / `max_steps`):**
  ```python
  max_retries = 5
  attempt = 0

  while attempt < max_retries:
      attempt += 1
      # Try operation...
      success = False
      if success:
          break
  else:
      print("Stopped after reaching maximum retries.")
  ```
- **Use time-based timeouts for polling loops:**
  ```python
  import time

  deadline = time.monotonic() + 5.0  # 5-second timeout
  while time.monotonic() < deadline:
      # Check condition...
      time.sleep(0.2)
  ```
- **Interrupting a stuck script manually:** Press `Ctrl + C` in the terminal to raise `KeyboardInterrupt` (which you can also catch cleanly with `try ... except KeyboardInterrupt:`).

---

## 15. `enumerate()` and `zip()` in Loops

Python provides built-in functions `enumerate()` and `zip()` so you rarely need manual index counters like `for i in range(len(items))`.

### `enumerate()`: Index + Value Together

`enumerate(iterable, start=0)` yields `(index, item)` pairs.

```python
languages = ["Python", "Rust", "Go"]

# Default start=0
for idx, lang in enumerate(languages):
    print(f"{idx}: {lang}")

# Custom start=1 (great for 1-based numbering)
for rank, lang in enumerate(languages, start=1):
    print(f"#{rank} {lang}")
# Output:
# #1 Python
# #2 Rust
# #3 Go
```

### `zip()`: Iterating Over Multiple Iterables in Parallel

`zip(*iterables, strict=False)` pairs corresponding elements from two or more iterables.

```python
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
roles = ["Engineer", "Designer", "Manager"]

for name, age, role in zip(names, ages, roles):
    print(f"{name} ({age}) - {role}")
# Output:
# Alice (25) - Engineer
# Bob (30) - Designer
# Charlie (35) - Manager
```

#### Handling Unequal Lengths with `zip()`

- **Default (`strict=False`):** Stops at the **shortest** iterable.
- **Python 3.10+ (`strict=True`):** Raises `ValueError` if lengths differ—useful for catching data mismatches early.
- **`itertools.zip_longest()`:** Continues until the **longest** iterable is exhausted, filling missing values with `fillvalue` (default `None`).

```python
from itertools import zip_longest

a = [1, 2, 3]
b = ["a", "b"]

print(list(zip(a, b)))                       # [(1, 'a'), (2, 'b')]
print(list(zip_longest(a, b, fillvalue="-")))  # [(1, 'a'), (2, 'b'), (3, '-')]

# strict=True raises ValueError when lengths don't match:
# list(zip(a, b, strict=True))
```

### Combining `enumerate()` and `zip()`

```python
questions = ["Name?", "Language?"]
answers = ["Ada", "Python"]

for i, (q, a) in enumerate(zip(questions, answers), start=1):
    print(f"Q{i}: {q} -> {a}")
# Output:
# Q1: Name? -> Ada
# Q2: Language? -> Python
```
