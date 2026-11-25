# Comprehensions in Python

Comprehensions provide a concise, expressive, and Pythonic way to construct lists, dictionaries, sets, and generators from existing iterables. Instead of initializing an empty collection and appending items inside a `for` loop, a comprehension lets you describe *what* collection to build in a single readable expression.

---

## Table of Contents

1. [List Comprehension — Syntax and Examples](#1-list-comprehension--syntax-and-examples)
2. [List Comprehension with Conditions (Filtering & Branching)](#2-list-comprehension-with-conditions-filtering--branching)
3. [Nested List Comprehension](#3-nested-list-comprehension)
4. [Dictionary Comprehension](#4-dictionary-comprehension)
5. [Set Comprehension](#5-set-comprehension)
6. [Generator Expressions (vs. List Comprehension)](#6-generator-expressions-vs-list-comprehension)
7. [Comprehensions with Multiple Iterables](#7-comprehensions-with-multiple-iterables)
8. [The Walrus Operator (`:=`) in Comprehensions](#8-the-walrus-operator--in-comprehensions)
9. [Performance: Comprehension vs. `for` Loop](#9-performance-comprehension-vs-for-loop)
10. [When to Use (and When NOT to Use) Comprehensions](#10-when-to-use-and-when-not-to-use-comprehensions)
11. [Readability Guidelines](#11-readability-guidelines)
12. [Complex Comprehension Examples](#12-complex-comprehension-examples)

---

## 1. List Comprehension — Syntax and Examples

A **list comprehension** creates a new `list` by evaluating an expression for each item in an iterable.

### Syntax

```python
[expression for item in iterable]
```

This is functionally equivalent to:

```python
result = []
for item in iterable:
    result.append(expression)
```

> **Note on Scope (Python 3):** Loop variables inside a comprehension (like `item`) are **local to the comprehension** and do not leak into the surrounding scope.

### Basic Examples

```python
# 1. Squaring numbers
numbers = [1, 2, 3, 4, 5]
squares = [n ** 2 for n in numbers]
print(squares)  # [1, 4, 9, 16, 25]

# 2. Transforming strings
words = ["python", "comprehension", "notes"]
upper_words = [w.upper() for w in words]
print(upper_words)  # ['PYTHON', 'COMPREHENSION', 'NOTES']

# 3. Calling a function on each element
lengths = [len(w) for w in words]
print(lengths)  # [6, 13, 5]

# 4. Working with range() and tuple unpacking
pairs = [(x, x * 10) for x in range(1, 5)]
print(pairs)  # [(1, 10), (2, 20), (3, 30), (4, 40)]

coordinates = [(1, 2), (3, 4), (5, 6)]
sums = [x + y for x, y in coordinates]
print(sums)  # [3, 7, 11]
```

---

## 2. List Comprehension with Conditions (Filtering & Branching)

Conditions in comprehensions serve two distinct purposes depending on **where** they appear:

1. **Filtering (`if` at the end):** Decides whether an item is included in the output.
2. **Conditional Expression (`if ... else` before `for`):** Transforms every item into one of two values.

### 2.1 Filtering with a Trailing `if`

```python
[expression for item in iterable if condition]
```

Equivalent loop:

```python
result = []
for item in iterable:
    if condition:
        result.append(expression)
```

```python
# Keep only even numbers
nums = range(10)
evens = [n for n in nums if n % 2 == 0]
print(evens)  # [0, 2, 4, 6, 8]

# Filter and transform simultaneously
names = ["Alice", "bob", "Charlie", "dave", "Eve"]
short_upper = [name.upper() for name in names if len(name) <= 4]
print(short_upper)  # ['BOB', 'DAVE', 'EVE']
```

### 2.2 Multiple Filter Conditions

You can chain multiple `if` clauses (equivalent to logical `and`), or combine conditions with `and` / `or`.

```python
# Numbers divisible by both 2 and 3
div_by_6 = [n for n in range(1, 31) if n % 2 == 0 if n % 3 == 0]
# Same as: [n for n in range(1, 31) if n % 2 == 0 and n % 3 == 0]
print(div_by_6)  # [6, 12, 18, 24, 30]
```

### 2.3 Conditional Transformation (`if ... else` Ternary)

When you want to **keep all items** but choose between two expressions, put the conditional expression *before* `for`:

```python
[true_expr if condition else false_expr for item in iterable]
```

```python
nums = [1, 2, 3, 4, 5]
labels = ["even" if n % 2 == 0 else "odd" for n in nums]
print(labels)  # ['odd', 'even', 'odd', 'even', 'odd']

# Replace negative numbers with 0
readings = [12, -4, 7, -1, 0, 9]
clamped = [x if x > 0 else 0 for x in readings]
print(clamped)  # [12, 0, 7, 0, 0, 9]
```

### 2.4 Combining Both (`if ... else` + Trailing `if`)

```python
# Filter out None values, then label remaining numbers
raw = [10, None, -3, 0, None, 5]
classified = [
    "positive" if x > 0 else "non-positive"
    for x in raw
    if x is not None
]
print(classified)  # ['positive', 'non-positive', 'non-positive', 'positive']
```

> **Common Mistake:** Writing `[x for x in nums if x > 0 else 0]` raises a `SyntaxError` because a trailing `if` is a filter (no `else` allowed), whereas `if ... else` before `for` is a value expression.

---

## 3. Nested List Comprehension

"Nested list comprehension" refers to two related patterns:
1. **A comprehension inside another comprehension's output expression** (creates nested lists, such as 2D grids).
2. **Multiple `for` clauses in one comprehension** (iterates over nested structures to produce a flat list).

### 3.1 Creating Nested Structures (2D Lists / Matrices)

```python
# Create a 3x4 multiplication grid
grid = [[row * col for col in range(1, 5)] for row in range(1, 4)]
print(grid)
# [[1, 2, 3, 4],
#  [2, 4, 6, 8],
#  [3, 6, 9, 12]]

# Create a 3x3 zero matrix safely (each row is an independent list)
zeros = [[0 for _ in range(3)] for _ in range(3)]
zeros[0][1] = 9
print(zeros)
# [[0, 9, 0], [0, 0, 0], [0, 0, 0]]
```

> **Why not `[[0] * 3] * 3`?** That creates three references to the *same* inner list, so mutating `zeros[0][1]` would mutate all three rows. A nested comprehension avoids this bug.

### 3.2 Iterating Over Nested Structures (Flattening)

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]

flat = [num for row in matrix for num in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Order of Clauses Rule

When a single comprehension has multiple `for` clauses, they execute in **left-to-right (top-to-bottom) order**, identical to nested `for` loops:

```python
# Comprehension:
[num for row in matrix for num in row if num % 2 == 0]

# Exact loop equivalent:
result = []
for row in matrix:          # outer loop (first 'for')
    for num in row:         # inner loop (second 'for')
        if num % 2 == 0:
            result.append(num)
```

---

## 4. Dictionary Comprehension

A **dictionary comprehension** builds a `dict` from key-value pairs using curly braces `{}` and a colon `:`.

### Syntax

```python
{key_expr: value_expr for item in iterable}
{key_expr: value_expr for item in iterable if condition}
```

### Examples

```python
# 1. Mapping numbers to their squares
squares = {n: n ** 2 for n in range(1, 6)}
print(squares)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# 2. Building a dict from two lists using zip()
keys = ["name", "role", "level"]
values = ["Ada", "Engineer", 5]
profile = {k: v for k, v in zip(keys, values)}
print(profile)  # {'name': 'Ada', 'role': 'Engineer', 'level': 5}

# 3. Filtering an existing dictionary
prices = {"apple": 1.2, "banana": 0.5, "cherry": 3.0, "date": 2.5}
expensive = {item: price for item, price in prices.items() if price >= 2.0}
print(expensive)  # {'cherry': 3.0, 'date': 2.5}

# 4. Transforming keys and values (e.g., applying a discount)
discounted = {item.title(): round(price * 0.9, 2) for item, price in prices.items()}
print(discounted)  # {'Apple': 1.08, 'Banana': 0.45, 'Cherry': 2.7, 'Date': 2.25}

# 5. Inverting a dictionary (assuming unique values)
country_codes = {"US": "United States", "IN": "India", "JP": "Japan"}
by_name = {name: code for code, name in country_codes.items()}
print(by_name)  # {'United States': 'US', 'India': 'IN', 'Japan': 'JP'}
```

> **Duplicate Keys:** If `key_expr` produces the same key multiple times, later iterations overwrite earlier ones—just like normal dictionary assignment.

---

## 5. Set Comprehension

A **set comprehension** uses curly braces `{}` *without* a colon `:`, producing a `set` of unique, hashable items.

### Syntax

```python
{expression for item in iterable}
{expression for item in iterable if condition}
```

### Examples

```python
# 1. Unique word lengths
sentence = "the quick brown fox jumps over the lazy dog"
unique_lengths = {len(word) for word in sentence.split()}
print(unique_lengths)  # {3, 4, 5}

# 2. Normalizing and deduplicating tags
raw_tags = ["Python", "python ", " DATA", "data", "Web"]
clean_tags = {tag.strip().lower() for tag in raw_tags}
print(clean_tags)  # {'python', 'data', 'web'}

# 3. Filtering unique initials
names = ["Alice", "alan", "Bob", "bella", "Charlie"]
initials = {name[0].upper() for name in names if len(name) > 3}
print(initials)  # {'A', 'B', 'C'}
```

> **Important Notes:**
> - An empty `{}` creates an empty **`dict`**, not a `set`. Use `set()` for an empty set.
> - Elements yielded by `expression` must be **hashable** (e.g., `int`, `str`, `tuple`), otherwise Python raises a `TypeError: unhashable type`.
> - Sets do not preserve insertion order.

---

## 6. Generator Expressions (vs. List Comprehension)

A **generator expression** looks like a list comprehension wrapped in parentheses `(...)` instead of square brackets `[...]`. Instead of computing and storing all values in memory at once, it returns a **generator object** that yields items **lazily** (one at a time, on demand).

### Syntax

```python
(expression for item in iterable)
(expression for item in iterable if condition)
```

### Examples

```python
# List comprehension: builds entire list in memory immediately
nums_list = [n ** 2 for n in range(1_000_000)]

# Generator expression: creates a lightweight iterator; computes on demand
nums_gen = (n ** 2 for n in range(1_000_000))

print(type(nums_gen))  # <class 'generator'>
print(next(nums_gen))  # 0
print(next(nums_gen))  # 1
```

### Passing Directly to Functions

When a generator expression is the **sole argument** to a function, the extra parentheses are optional:

```python
total = sum(n ** 2 for n in range(1, 6))          # 55
has_negative = any(x < 0 for x in [3, 5, -1, 8])  # True
all_even = all(x % 2 == 0 for x in [2, 4, 6])     # True
longest = max(len(w) for w in ["hi", "hello"])    # 5
joined = ", ".join(str(n) for n in range(5))      # '0, 1, 2, 3, 4'
```

### Comparison: List Comprehension vs. Generator Expression

| Feature | List Comprehension `[...]` | Generator Expression `(...)` |
|---|---|---|
| **Evaluation** | Eager (computes all elements immediately) | Lazy (computes one element per `next()`) |
| **Memory usage** | $O(N)$ — stores all elements in RAM | $O(1)$ — stores only current state |
| **Reusability** | Reusable; can iterate multiple times | Single-use; exhausted after one pass |
| **Indexing / `len()`** | Supports `lst[0]`, slicing, `len(lst)` | Does **not** support indexing or `len()` |
| **Best suited for** | Small/medium data needing random access or reuse | Large streams, pipelines, `sum`/`any`/`all` |

```python
# Single-use gotcha:
gen = (x * 2 for x in [1, 2, 3])
print(list(gen))  # [2, 4, 6]
print(list(gen))  # []  (generator is now exhausted!)
```

---

## 7. Comprehensions with Multiple Iterables

There are two ways to work with multiple iterables in a comprehension:

### 7.1 Cartesian Product (Nested `for` Clauses)

Every item from the first iterable is paired with every item from the second iterable.

```python
colors = ["red", "blue"]
sizes = ["S", "M", "L"]

shirts = [(color, size) for color in colors for size in sizes]
print(shirts)
# [('red', 'S'), ('red', 'M'), ('red', 'L'),
#  ('blue', 'S'), ('blue', 'M'), ('blue', 'L')]
```

### 7.2 Paired / Parallel Iteration (`zip`, `enumerate`)

When you want element-by-element pairing rather than all combinations, use `zip()` or `enumerate()`:

```python
names = ["Alice", "Bob", "Charlie"]
scores = [88, 95, 72]

# Pair corresponding elements with zip()
report = [f"{name}: {score}" for name, score in zip(names, scores)]
print(report)  # ['Alice: 88', 'Bob: 95', 'Charlie: 72']

# Include index with enumerate()
indexed = [f"{idx}. {name}" for idx, name in enumerate(names, start=1)]
print(indexed)  # ['1. Alice', '2. Bob', '3. Charlie']

# Parallel calculation across two vectors
v1 = [1, 2, 3]
v2 = [4, 5, 6]
dot_terms = [a * b for a, b in zip(v1, v2)]
print(sum(dot_terms))  # 32
```

### 7.3 Dependent Iterables

The second `for` clause can depend on the loop variable of the first:

```python
# Triangular pairs where j > i
pairs = [(i, j) for i in range(4) for j in range(i + 1, 4)]
print(pairs)  # [(0, 1), (0, 2), (0, 3), (1, 2), (1, 3), (2, 3)]
```

---

## 8. The Walrus Operator (`:=`) in Comprehensions

Introduced in **Python 3.8**, the assignment expression operator `:=` (walrus operator) assigns a value to a variable *as part of an expression*. Inside comprehensions, it solves two classic problems:

### 8.1 Avoiding Redundant Function Calls

Without `:=`, if you need to filter on an expensive computation *and* keep its result, you either call the function twice or nest another comprehension.

```python
def slow_process(x):
    return x * x + 10

data = [1, 2, 3, 4, 5]

# Without walrus: slow_process(x) is called TWICE for matching items
results_old = [slow_process(x) for x in data if slow_process(x) > 20]

# With walrus: slow_process(x) is called ONCE per item
results = [val for x in data if (val := slow_process(x)) > 20]
print(results)  # [26, 35]
```

### 8.2 Capturing Intermediate Values in Dictionaries / Filtering

```python
import re

lines = ["id=101", "invalid", "id=204", "name=test"]
# Match once, filter out None, and extract group(1)
ids = [
    m.group(1)
    for line in lines
    if (m := re.match(r"id=(\d+)", line)) is not None
]
print(ids)  # ['101', '204']
```

### 8.3 Stateful Running Calculations

```python
# Running cumulative sum
nums = [1, 2, 3, 4, 5]
total = 0
running_sum = [total := total + n for n in nums]
print(running_sum)  # [1, 3, 6, 10, 15]
print(total)        # 15
```

### Important Scope Gotcha with `:=`

Unlike normal comprehension loop variables, a variable assigned with `:=` **leaks into the enclosing scope** (function or module scope), because PEP 572 defines walrus targets inside comprehensions as binding in the containing scope:

```python
x = "outer"
items = [y for n in [1, 2, 3] if (y := n * 10) > 10]
print(items)  # [20, 30]
print(y)      # 30  (leaked into enclosing scope!)
```

> **Restriction:** You cannot use `:=` to assign to the comprehension's own iteration variable (e.g., `[x := 1 for x in items]` raises a `SyntaxError`).

---

## 9. Performance: Comprehension vs. `for` Loop

List comprehensions are typically **faster** than building a list with an explicit `for` loop and `.append()`.

### Why Comprehensions Are Faster

1. **Dedicated Bytecode:** CPython compiles a list comprehension using a specialized `LIST_APPEND` bytecode instruction rather than looking up the `.append` method attribute on the list object and executing a Python function call on every iteration.
2. **Fewer Global/Local Lookups:** Loop control and append happen tightly inside the comprehension's frame (or inlined comprehension in Python 3.12+).

### Benchmark Example (`timeit`)

```python
import timeit

setup = "nums = range(10_000)"

loop_code = """
res = []
for n in nums:
    if n % 2 == 0:
        res.append(n * n)
"""

comp_code = """
res = [n * n for n in nums if n % 2 == 0]
"""

loop_time = timeit.timeit(loop_code, setup=setup, number=1000)
comp_time = timeit.timeit(comp_code, setup=setup, number=1000)

print(f"For loop:      {loop_time:.4f}s")
print(f"Comprehension: {comp_time:.4f}s")
```

On typical CPython runs, the comprehension is noticeably faster (often ~25–40% faster for simple append-heavy loops).

### Performance Caveats

- **Python 3.12 (PEP 709):** List, dict, and set comprehensions are *inlined* into the enclosing function instead of creating a separate function frame, making them even faster.
- **Function call bottleneck:** If the body calls an expensive function (e.g., network/DB I/O or heavy math), loop vs. comprehension overhead becomes negligible.
- **Memory matters too:** If you don't need the whole list in memory (e.g., passing to `sum()`, `any()`, `max()`), a **generator expression** saves memory and avoids allocating a large list altogether.

---

## 10. When to Use (and When NOT to Use) Comprehensions

### When to USE Comprehensions

- **Transforming** each element of an iterable (`map`-like operations): `[x.strip() for x in lines]`
- **Filtering** elements based on a clear condition (`filter`-like operations): `[u for u in users if u.is_active]`
- **Building dicts or sets** directly from an iterable: `{u.id: u for u in users}`
- **Replacing `map()` / `filter()` with `lambda`:** `[x * 2 for x in nums if x > 0]` is generally preferred over `list(map(lambda x: x * 2, filter(lambda x: x > 0, nums)))`.

### When NOT to Use Comprehensions

1. **Side effects only (Anti-pattern!)**
   Never build a comprehension only for its side effects (like `print()`, mutating another list, or writing to a file) and discard the resulting list. It wastes memory by building a list of `None`s and misleads readers.

   ```python
   # BAD: creates a useless list of [None, None, None]
   [print(item) for item in items]

   # GOOD: use a plain for loop
   for item in items:
       print(item)
   ```

2. **Too much nesting or complex logic**
   More than two `for` clauses or multiple nested `if-else` expressions quickly become unreadable.

   ```python
   # BAD: hard to parse at a glance
   result = [
       f(x, y, z)
       for x in a if x > 0
       for y in b if y < x
       for z in c if (x + y + z) % 2 == 0
   ]
   ```

3. **Exception handling (`try / except`)**
   Comprehensions do not support `try / except` blocks directly. Use a helper function or a standard `for` loop.

4. **Early termination (`break`)**
   You cannot `break` out of a comprehension early (for short-circuiting searches, use `next()`, `any()`, `all()`, or a `for` loop).

5. **Complex state accumulation**
   If each step depends heavily on mutating multiple variables, a standard `for` loop is much clearer than forcing `:=`.

---

## 11. Readability Guidelines

Following conventions from **PEP 8** and the **Google Python Style Guide**:

1. **Keep it short and flat:** Aim for at most **one or two** `for` clauses and **one** `if` condition per comprehension. If you need 3+ levels, switch to a `for` loop or extract a helper function.
2. **Format long comprehensions across multiple lines:** Put the expression, `for` clause(s), and `if` clause on separate aligned lines.

   ```python
   active_emails = [
       user.email.lower()
       for user in users
       if user.is_active and user.email
   ]
   ```

3. **Use meaningful loop variable names:** Avoid cryptic single-letter names (`x`, `i`) unless iterating over plain numbers or coordinates. Prefer `for user in users` over `for u in users` in longer expressions.
4. **Extract complex expressions into helper functions:**

   ```python
   # Harder to read:
   cleaned = [s.strip().lower().replace("-", "_") for s in headers if s and not s.startswith("#")]

   # Cleaner with a helper:
   def normalize_header(text: str) -> str:
       return text.strip().lower().replace("-", "_")

   cleaned = [
       normalize_header(s)
       for s in headers
       if s and not s.startswith("#")
   ]
   ```

5. **Never sacrifice clarity for cleverness:** A readable 4-line `for` loop is always better than a clever 1-line comprehension that takes 30 seconds to decipher.

---

## 12. Complex Comprehension Examples

### 12.1 Flattening a 2D List (and Arbitrary-Depth Note)

```python
nested = [[1, 2], [3, 4, 5], [], [6]]
flat = [item for sublist in nested for item in sublist]
print(flat)  # [1, 2, 3, 4, 5, 6]
```

> **Note:** A comprehension works cleanly for a **known** fixed depth (2 or 3 levels). For **arbitrary/unknown nesting depth**, use recursion rather than trying to force a comprehension.

### 12.2 Transposing a Matrix

Convert rows into columns (an $M \times N$ matrix becomes $N \times M$):

```python
matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12],
]

# Using nested list comprehension
transposed = [[row[i] for row in matrix] for i in range(len(matrix[0]))]
print(transposed)
# [[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]

# Note: zip(*matrix) is another idiomatic alternative for transposition:
transposed_zip = [list(col) for col in zip(*matrix)]
print(transposed_zip)
# [[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
```

### 12.3 Grouping / Inverting a Non-Unique Dictionary

When multiple keys share the same value, map each value to a list of its original keys:

```python
roles = {
    "Alice": "admin",
    "Bob": "user",
    "Charlie": "admin",
    "Diana": "moderator",
    "Eve": "user",
}

grouped = {
    role: [person for person, r in roles.items() if r == role]
    for role in set(roles.values())
}
print(grouped)
# {'admin': ['Alice', 'Charlie'], 'user': ['Bob', 'Eve'], 'moderator': ['Diana']}
```

*(Note: For very large dictionaries, `collections.defaultdict(list)` in a single `for` loop is $O(N)$ instead of $O(N \times K)$.)*

### 12.4 Frequency Count (Word Histogram)

```python
text = "apple banana apple cherry banana apple"
words = text.split()
freq = {word: words.count(word) for word in set(words)}
print(freq)  # {'apple': 3, 'banana': 2, 'cherry': 1}
```

*(Again, `collections.Counter(words)` is preferred for large inputs because `.count()` inside a comprehension is $O(N^2)$.)*

### 12.5 Pythagorean Triples

Find all integer right-triangle triples $(a, b, c)$ with $1 \le a < b < c \le 20$:

```python
triples = [
    (a, b, c)
    for a in range(1, 21)
    for b in range(a + 1, 21)
    for c in range(b + 1, 21)
    if a ** 2 + b ** 2 == c ** 2
]
print(triples)
# [(3, 4, 5), (5, 12, 13), (6, 8, 10), (8, 15, 17), (9, 12, 15), (12, 16, 20)]
```

### 12.6 Sliding Window (N-grams)

```python
tokens = ["I", "love", "writing", "clean", "Python"]
bigrams = [(tokens[i], tokens[i + 1]) for i in range(len(tokens) - 1)]
print(bigrams)
# [('I', 'love'), ('love', 'writing'), ('writing', 'clean'), ('clean', 'Python')]

# Using zip for a cleaner sliding window:
trigrams = [window for window in zip(tokens, tokens[1:], tokens[2:])]
print(trigrams)
# [('I', 'love', 'writing'), ('love', 'writing', 'clean'), ('writing', 'clean', 'Python')]
```

### 12.7 Flattening and Filtering JSON-like Nested Records

```python
departments = [
    {
        "name": "Engineering",
        "employees": [
            {"name": "Ada", "active": True, "salary": 140_000},
            {"name": "Bob", "active": False, "salary": 110_000},
        ],
    },
    {
        "name": "Research",
        "employees": [
            {"name": "Grace", "active": True, "salary": 155_000},
        ],
    },
]

active_high_earners = {
    emp["name"]: dept["name"]
    for dept in departments
    for emp in dept["employees"]
    if emp["active"] and emp["salary"] >= 130_000
}
print(active_high_earners)  # {'Ada': 'Engineering', 'Grace': 'Research'}
```

---

## Quick Reference Summary

| Type | Syntax | Output |
|---|---|---|
| **List comprehension** | `[x for x in it if cond]` | `list` |
| **List with if-else** | `[a if cond else b for x in it]` | `list` |
| **Dict comprehension** | `{k: v for k, v in it if cond}` | `dict` |
| **Set comprehension** | `{x for x in it if cond}` | `set` |
| **Generator expression** | `(x for x in it if cond)` | `generator` (lazy iterator) |
