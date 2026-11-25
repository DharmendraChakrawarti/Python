# Tuples in Python

A **tuple** is an ordered, immutable collection of elements in Python. Tuples can store items of different data types (heterogeneous) or the same data type (homogeneous), allow duplicate values, and maintain insertion order.

Because tuples are immutable, they are faster and more memory-efficient than lists, and they can be used wherever a hashable (fixed) sequence is required.

---

## Table of Contents

1. [Creating Tuples](#1-creating-tuples)
2. [Single-Element Tuple Gotcha](#2-single-element-tuple-gotcha)
3. [Accessing Elements (Indexing & Slicing)](#3-accessing-elements-indexing--slicing)
4. [Tuple Immutability (and Mutable Objects Inside Tuples)](#4-tuple-immutability-and-mutable-objects-inside-tuples)
5. [Tuple Methods (`count` and `index`)](#5-tuple-methods-count-and-index)
6. [Tuple Operations (Concatenation, Repetition, Membership)](#6-tuple-operations-concatenation-repetition-membership)
7. [Tuple Packing and Unpacking](#7-tuple-packing-and-unpacking)
8. [Named Tuples (`collections.namedtuple`)](#8-named-tuples-collectionsnamedtuple)
9. [Tuple vs List — When to Use Which](#9-tuple-vs-list--when-to-use-which)
10. [Tuples as Dictionary Keys](#10-tuples-as-dictionary-keys)
11. [Tuple Comprehension? (Generator Expressions)](#11-tuple-comprehension-generator-expressions)
12. [`zip()` and Tuples](#12-zip-and-tuples)
13. [Returning Multiple Values from Functions Using Tuples](#13-returning-multiple-values-from-functions-using-tuples)

---

## 1. Creating Tuples

Tuples can be created in three main ways: using parentheses `()`, separating items with commas `,` (without parentheses), or using the built-in `tuple()` constructor.

> **Key Rule:** It is actually the **comma** `,` that creates a tuple in Python, not the parentheses (except for an empty tuple `()`). Parentheses are mainly used for clarity and to avoid ambiguity in expressions.

```python
# 1. Using parentheses
empty_tuple = ()
numbers = (1, 2, 3, 4)
mixed = (1, "hello", 3.14, True)
nested = ((1, 2), ["a", "b"], (3, 4))

print(empty_tuple)  # ()
print(numbers)      # (1, 2, 3, 4)
print(mixed)        # (1, 'hello', 3.14, True)

# 2. Without parentheses (comma-separated — tuple packing)
colors = "red", "green", "blue"
print(colors)       # ('red', 'green', 'blue')
print(type(colors)) # <class 'tuple'>

# 3. Using the tuple() constructor (converts any iterable)
from_list = tuple([10, 20, 30])
from_string = tuple("abc")
from_range = tuple(range(5))
empty_via_fn = tuple()

print(from_list)    # (10, 20, 30)
print(from_string)  # ('a', 'b', 'c')
print(from_range)   # (0, 1, 2, 3, 4)
print(empty_via_fn) # ()
```

> **Note:** When passing a tuple literal directly as a function argument, parentheses are required so commas aren't treated as argument separators:
> ```python
> print(len((1, 2, 3)))  # Valid: passes one tuple argument -> 3
> # print(len(1, 2, 3))  # TypeError: len() takes exactly one argument (3 given)
> ```

---

## 2. Single-Element Tuple Gotcha

A common beginner mistake in Python is omitting the trailing comma when creating a one-element tuple. Without a trailing comma, parentheses simply act as grouping operators (just like in math: `(5)` evaluates to `5`).

```python
# WRONG: This is NOT a tuple!
not_a_tuple_int = (42)
not_a_tuple_str = ("hello")

print(type(not_a_tuple_int))  # <class 'int'>
print(type(not_a_tuple_str))  # <class 'str'>

# RIGHT: Include a trailing comma
single_int = (42,)
single_str = ("hello",)
also_a_tuple = 42,            # Parentheses are optional, comma is mandatory

print(single_int)             # (42,)
print(type(single_int))       # <class 'tuple'>
print(len(single_int))        # 1
print(type(also_a_tuple))     # <class 'tuple'>
```

This gotcha frequently shows up when concatenating tuples or passing a single value into functions expecting a tuple (such as SQL parameter bindings or `multiprocessing.Process(args=(item,))`).

---

## 3. Accessing Elements (Indexing & Slicing)

Like lists and strings, tuples support **0-based positive indexing**, **negative indexing**, and **slicing** (`tuple[start:stop:step]`).

### Indexing

```python
fruits = ("apple", "banana", "cherry", "date", "elderberry")

# Positive indexing (from the start, 0-based)
print(fruits[0])   # 'apple'
print(fruits[2])   # 'cherry'

# Negative indexing (from the end, -1 is the last item)
print(fruits[-1])  # 'elderberry'
print(fruits[-2])  # 'date'

# Nested tuple indexing
matrix = ((1, 2, 3), (4, 5, 6), (7, 8, 9))
print(matrix[1][2])  # 6

# Accessing an out-of-range index raises IndexError
# print(fruits[10])  # IndexError: tuple index out of range
```

### Slicing

Slicing always returns a **new tuple** without modifying the original.

```python
nums = (0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

print(nums[2:6])     # (2, 3, 4, 5)      — index 2 up to (excluding) 6
print(nums[:4])      # (0, 1, 2, 3)      — from start to index 4
print(nums[6:])      # (6, 7, 8, 9)      — from index 6 to end
print(nums[-4:])     # (6, 7, 8, 9)      — last 4 elements
print(nums[::2])     # (0, 2, 4, 6, 8)   — every 2nd element
print(nums[::-1])    # (9, 8, 7, 6, 5, 4, 3, 2, 1, 0) — reversed tuple
```

---

## 4. Tuple Immutability (and Mutable Objects Inside Tuples)

Tuples are **immutable**: once created, you cannot add, remove, or reassign elements in the tuple itself.

```python
coords = (10, 20, 30)

# Reassigning an element raises TypeError
# coords[0] = 99  # TypeError: 'tuple' object does not support item assignment

# Deleting an element raises TypeError
# del coords[0]   # TypeError: 'tuple' object doesn't support item deletion

# However, you can reassign the variable name to a new tuple
coords = (99, 20, 30)
print(coords)  # (99, 20, 30)
```

### The Subtlety: Mutable Objects Inside a Tuple

A tuple stores **references** to its objects. Immutability means the **references** stored inside the tuple cannot be changed—it does **not** make the referenced objects themselves immutable.

If a tuple contains a mutable object (such as a `list` or `dict`), you can mutate that inner object in place:

```python
record = ("Alice", [85, 90, 92])

# You CANNOT replace the list with another list:
# record[1] = [100, 100]  # TypeError: 'tuple' object does not support item assignment

# You CAN mutate the list inside the tuple:
record[1].append(95)
record[1][0] = 88
print(record)  # ('Alice', [88, 90, 92, 95])
```

### Augmented Assignment Gotcha (`+=`)

Beware of using `+=` on a mutable element inside a tuple (like `t[0] += [4]`):

```python
t = ([1, 2], "ok")
# t[0] += [3]
```

What happens here?
1. `list.__iadd__` mutates `[1, 2]` in place to `[1, 2, 3]` and returns the list reference.
2. Python then attempts to reassign `t[0] = <result>`, which raises `TypeError: 'tuple' object does not support item assignment`—**even though the list inside `t` was already modified!**

To avoid this surprise, use `.append()` or `.extend()` instead of `+=` on mutable objects inside tuples.

---

## 5. Tuple Methods (`count` and `index`)

Because tuples cannot be modified in place, they only have **two** built-in methods: `count()` and `index()`. (Methods like `append()`, `sort()`, `remove()`, and `pop()` do not exist on tuples.)

### `tuple.count(value)`

Returns the number of times `value` appears in the tuple.

```python
letters = ("a", "b", "a", "c", "a", "b")

print(letters.count("a"))  # 3
print(letters.count("b"))  # 2
print(letters.count("z"))  # 0
```

### `tuple.index(value[, start[, stop]])`

Returns the index of the **first** occurrence of `value`. Raises `ValueError` if `value` is not found. Optional `start` and `stop` arguments restrict the search slice `[start:stop]`.

```python
nums = (10, 20, 30, 20, 40, 20)

print(nums.index(20))        # 1 (first occurrence)
print(nums.index(20, 2))     # 3 (search starting at index 2)
print(nums.index(20, 4, 6))  # 5 (search in indices 4..5)

# Safe lookup pattern to avoid ValueError
target = 99
if target in nums:
    print(nums.index(target))
else:
    print(f"{target} not found")  # 99 not found
```

---

## 6. Tuple Operations (Concatenation, Repetition, Membership)

Tuples support standard sequence operations. Operations that create modified sequences always return a **new** tuple.

```python
a = (1, 2, 3)
b = (4, 5)

# Concatenation (+)
combined = a + b
print(combined)      # (1, 2, 3, 4, 5)

# Repetition (*)
repeated = ("ha",) * 3
zeros = (0,) * 5
print(repeated)      # ('ha', 'ha', 'ha')
print(zeros)         # (0, 0, 0, 0, 0)

# Membership testing (in / not in)
print(2 in a)        # True
print(9 not in a)    # True

# Built-in sequence functions
scores = (88, 95, 72, 91)
print(len(scores))   # 4
print(min(scores))   # 72
print(max(scores))   # 95
print(sum(scores))   # 346
print(sorted(scores))  # [72, 88, 91, 95]  (Note: sorted() returns a list!)

# Lexicographical comparison
print((1, 2, 3) < (1, 2, 4))  # True
print((1, 2) == (1, 2))       # True
```

> **Note on `+=` with tuples:** Writing `t += (4, 5)` works without error, because it creates a new tuple and rebinds the variable `t` to that new tuple—it does not mutate the original tuple.
> ```python
> t1 = (1, 2)
> t2 = t1
> t1 += (3,)
> print(t1)  # (1, 2, 3)
> print(t2)  # (1, 2) — unchanged!
> ```

---

## 7. Tuple Packing and Unpacking

**Packing** is assigning multiple values into a tuple. **Unpacking** is extracting values from a tuple (or any iterable) into separate variables in a single statement.

### Basic Packing and Unpacking

```python
# Packing
person = "Ada", 36, "Mathematician"

# Unpacking (number of variables must match tuple length)
name, age, role = person
print(name)  # 'Ada'
print(age)   # 36
print(role)  # 'Mathematician'

# Swapping variables without a temporary variable (uses packing & unpacking)
x, y = 10, 20
x, y = y, x
print(x, y)  # 20 10
```

### Extended Unpacking with `*` (Starred Expression)

Use `*` to capture zero or more remaining items into a **list**. Only one starred target is allowed per unpacking level.

```python
numbers = (1, 2, 3, 4, 5, 6)

first, *middle, last = numbers
print(first)   # 1
print(middle)  # [2, 3, 4, 5]  (always a list)
print(last)    # 6

head, *tail = (10,)
print(head)    # 10
print(tail)    # []            (empty list when nothing remains)
```

### Ignoring Values and Nested Unpacking

By convention, `_` is used for values you want to ignore.

```python
# Ignoring unwanted values
record = ("2026-11-26", "INFO", "Server started", 42)
date, _, message, _ = record
print(date, message)  # 2026-11-26 Server started

# Ignoring multiple trailing values
first_item, *_ = (100, 200, 300, 400)
print(first_item)  # 100

# Nested unpacking
user = ("Bob", (1995, 7, 14))
username, (year, month, day) = user
print(f"{username} was born in {year}")  # Bob was born in 1995
```

---

## 8. Named Tuples (`collections.namedtuple`)

Standard tuples rely on positional indices (`point[0]`, `point[1]`), which can hurt readability. `collections.namedtuple` creates a lightweight, immutable tuple subclass whose elements can be accessed by **field name** as well as by index.

```python
from collections import namedtuple

# Define a named tuple class
Point = namedtuple("Point", ["x", "y"])
# Alternatives for field_names:
# Point = namedtuple("Point", "x y")
# Point = namedtuple("Point", "x, y")

p = Point(10, 20)
p_kw = Point(x=3, y=4)

# Access by attribute name OR by index
print(p.x, p.y)      # 10 20
print(p[0], p[1])    # 10 20
print(p)             # Point(x=10, y=20)

# Unpacking still works because it is a tuple subclass
x, y = p
print(isinstance(p, tuple))  # True
```

### Useful Named Tuple Features

```python
from collections import namedtuple

Employee = namedtuple("Employee", ["name", "role", "salary"], defaults=["Engineer", 80000])

e1 = Employee("Alice")
print(e1)  # Employee(name='Alice', role='Engineer', salary=80000)

# _replace() returns a new instance with updated fields (does not mutate original)
e2 = e1._replace(salary=95000)
print(e2)  # Employee(name='Alice', role='Engineer', salary=95000)

# _asdict() converts to a regular dict
print(e2._asdict())  # {'name': 'Alice', 'role': 'Engineer', 'salary': 95000}

# _make() creates an instance from an iterable
data = ["Charlie", "Designer", 75000]
e3 = Employee._make(data)
print(e3)  # Employee(name='Charlie', role='Designer', salary=75000)

# _fields lists the field names
print(Employee._fields)  # ('name', 'role', 'salary')
```

> **Tip:** In modern Python (3.6+), [`typing.NamedTuple`](https://docs.python.org/3/library/typing.html#typing.NamedTuple) offers the same runtime behavior with cleaner class syntax and type annotations:
> ```python
> from typing import NamedTuple
>
> class Point(NamedTuple):
>     x: float
>     y: float = 0.0
> ```

---

## 9. Tuple vs List — When to Use Which

While both tuples and lists are ordered sequences, they serve different semantic and technical purposes.

| Feature | Tuple `(...)` | List `[...]` |
| :--- | :--- | :--- |
| **Mutability** | Immutable | Mutable |
| **Memory overhead** | Smaller (fixed size, no over-allocation) | Larger (allocates extra space for growth) |
| **Iteration / Creation speed** | Slightly faster (constant-folded by compiler) | Slightly slower |
| **Hashable (usable as dict key / set item)** | Yes, if all elements are hashable | No (`TypeError: unhashable type: 'list'`) |
| **Available methods** | `count`, `index` | `append`, `extend`, `insert`, `pop`, `remove`, `sort`, `reverse`, `clear`, `copy`, `count`, `index` |
| **Typical data structure** | **Heterogeneous** fixed records (e.g., `(name, age, email)`) | **Homogeneous** dynamic collections (e.g., `[user1, user2, user3]`) |

### Rule of Thumb

- **Use a tuple** when:
  - The collection represents a **fixed structure or record** where position carries meaning (coordinates `(x, y)`, RGB `(255, 128, 0)`, database rows).
  - You want to guarantee the data **cannot be accidentally modified**.
  - You need the sequence as a **dictionary key** or **set element**.
  - Returning multiple values from a function.
- **Use a list** when:
  - You need to **add, remove, reorder, or modify** items over time.
  - All elements represent a homogeneous collection of arbitrary length (e.g., lines of a file, list of tasks).

```python
import sys

sample_list = [1, 2, 3, 4, 5]
sample_tuple = (1, 2, 3, 4, 5)

print(sys.getsizeof(sample_list))   # e.g. 104 bytes (varies slightly by Python version/arch)
print(sys.getsizeof(sample_tuple))  # e.g. 80 bytes
```

---

## 10. Tuples as Dictionary Keys

Dictionary keys (and `set` elements) in Python must be **hashable**. Lists are not hashable because they are mutable, whereas a tuple is hashable **as long as every element inside it is also hashable**.

This makes tuples ideal for composite keys like grid coordinates, `(year, month)` pairs, or graph edges.

```python
# 2D grid / coordinate map
grid = {
    (0, 0): "Start",
    (1, 2): "Treasure",
    (4, 5): "Exit",
}

print(grid[(1, 2)])  # 'Treasure'

# Memoization / caching with composite keys
distances = {}
distances[("NYC", "LON")] = 5570
distances[("NYC", "TYO")] = 10850

# Sets of tuples (e.g., unique visited coordinates)
visited = {(0, 0), (0, 1), (1, 1)}
visited.add((0, 0))  # Ignored, already in set
print(visited)       # {(0, 1), (1, 1), (0, 0)}
```

### Unhashable Element Gotcha

If a tuple contains a mutable (unhashable) object like a `list` or `dict`, the tuple itself is **not** hashable:

```python
valid_key = (1, "a", (2, 3))
print(hash(valid_key))  # Works fine

invalid_key = (1, [2, 3])
# hash(invalid_key)     # TypeError: unhashable type: 'list'
# d = {invalid_key: 1}  # TypeError: unhashable type: 'list'
```

---

## 11. Tuple Comprehension? (Generator Expressions)

Python has list comprehensions `[x for x in ...]`, set comprehensions `{x for x in ...}`, and dict comprehensions `{k: v for ...}`, **but there is no tuple comprehension syntax**.

Wrapping a comprehension in parentheses `(...)` creates a **generator expression**, not a tuple.

```python
# This creates a generator object, NOT a tuple!
gen = (x ** 2 for x in range(5))
print(gen)        # <generator object <genexpr> at 0x...>
print(type(gen))  # <class 'generator'>

# Generators compute values lazily on demand and are exhausted after one pass
print(list(gen))  # [0, 1, 4, 9, 16]
print(list(gen))  # [] (exhausted!)
```

### How to Create a Tuple from a Comprehension

Pass the generator expression to the `tuple()` constructor (you don't need double parentheses):

```python
squares = tuple(x ** 2 for x in range(5))
print(squares)        # (0, 1, 4, 9, 16)
print(type(squares))  # <class 'tuple'>

# With a filter condition
evens = tuple(x for x in range(10) if x % 2 == 0)
print(evens)          # (0, 2, 4, 6, 8)
```

---

## 12. `zip()` and Tuples

The built-in `zip()` function pairs elements from two or more iterables into an iterator of **tuples**. It stops as soon as the shortest iterable is exhausted (unless `strict=True` is passed in Python 3.10+).

```python
names = ("Alice", "Bob", "Charlie")
scores = [92, 85, 78]
grades = ["A", "B", "C"]

# zip() yields tuples lazily
zipped = zip(names, scores, grades)
print(list(zipped))
# [('Alice', 92, 'A'), ('Bob', 85, 'B'), ('Charlie', 78, 'C')]

# Common usage: iterating over parallel sequences with tuple unpacking
for name, score in zip(names, scores):
    print(f"{name}: {score}")

# Building a dictionary from two sequences
score_map = dict(zip(names, scores))
print(score_map)  # {'Alice': 92, 'Bob': 85, 'Charlie': 78}
```

### Unzipping with `zip(*...)`

You can "unzip" a sequence of tuples back into separate tuples using the unpacking operator `*`:

```python
pairs = [("a", 1), ("b", 2), ("c", 3)]
letters, numbers = zip(*pairs)

print(letters)  # ('a', 'b', 'c')
print(numbers)  # (1, 2, 3)
```

### Strict Zipping (Python 3.10+)

```python
# By default, zip silently truncates to the shortest iterable
print(list(zip((1, 2, 3), ("a", "b"))))  # [(1, 'a'), (2, 'b')]

# strict=True raises ValueError if lengths differ
# list(zip((1, 2, 3), ("a", "b"), strict=True))  # ValueError
```

---

## 13. Returning Multiple Values from Functions Using Tuples

In Python, functions can only return a single object, but **tuple packing** makes returning multiple values effortless. Writing `return a, b, c` actually packs `a`, `b`, and `c` into a single tuple `(a, b, c)`.

```python
def min_max_avg(numbers):
    """Return the minimum, maximum, and average of a non-empty sequence."""
    minimum = min(numbers)
    maximum = max(numbers)
    average = sum(numbers) / len(numbers)
    return minimum, maximum, average  # Implicitly returns a tuple

# Unpack directly at the call site
low, high, avg = min_max_avg([10, 20, 30, 40])
print(f"Low: {low}, High: {high}, Avg: {avg}")  # Low: 10, High: 40, Avg: 25.0

# Or capture the returned tuple as a single object
stats = min_max_avg([10, 20, 30, 40])
print(stats)        # (10, 40, 25.0)
print(type(stats))  # <class 'tuple'>
```

### Real-World Standard Library Examples

Many Python built-ins and standard library functions use this pattern:

```python
# divmod(a, b) returns (quotient, remainder)
quotient, remainder = divmod(17, 5)
print(quotient, remainder)  # 3 2

# enumerate(iterable) yields (index, item) tuples
for idx, fruit in enumerate(("apple", "banana", "cherry"), start=1):
    print(f"{idx}. {fruit}")

# dict.items() yields (key, value) tuples
config = {"host": "localhost", "port": 8080}
for key, value in config.items():
    print(f"{key} = {value}")
```

> **Tip:** If a function returns more than 3–4 values, consider returning a `namedtuple` (or `dataclass`) instead of a plain tuple so callers can access fields by name without having to remember positional order.
