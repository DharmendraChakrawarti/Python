# Lists in Python

A **list** is Python's built-in, ordered, mutable sequence type. Lists can hold items of any data type (including mixed types and other lists), grow or shrink dynamically, and support indexing, slicing, and a rich set of methods.

---

## Table of Contents

1. [Creating Lists](#1-creating-lists)
2. [Accessing Elements (Indexing & Slicing)](#2-accessing-elements-indexing--slicing)
3. [List Mutability](#3-list-mutability)
4. [List Methods](#4-list-methods)
5. [List Operations](#5-list-operations)
6. [Nested Lists (2D Lists & Matrices)](#6-nested-lists-2d-lists--matrices)
7. [List Comprehensions](#7-list-comprehensions)
8. [List Iteration Techniques](#8-list-iteration-techniques)
9. [List Unpacking and Starred Expressions](#9-list-unpacking-and-starred-expressions)
10. [Sorting Lists (`sort` vs `sorted`)](#10-sorting-lists-sort-vs-sorted)
11. [Copying Lists (Shallow vs Deep Copy)](#11-copying-lists-shallow-vs-deep-copy)
12. [List as Stack and Queue](#12-list-as-stack-and-queue)
13. [Common List Patterns and Pitfalls](#13-common-list-patterns-and-pitfalls)

---

## 1. Creating Lists

Lists can be created using square brackets `[]`, the `list()` constructor, repetition, or comprehensions.

```python
# Empty list
empty_1 = []
empty_2 = list()

# List literal with elements
numbers = [1, 2, 3, 4, 5]
fruits = ["apple", "banana", "cherry"]

# Mixed data types (allowed, though homogeneous lists are more common)
mixed = [42, "hello", 3.14, True, None, [1, 2]]

# Converting other iterables with list()
chars = list("python")          # ['p', 'y', 't', 'h', 'o', 'n']
from_tuple = list((10, 20, 30)) # [10, 20, 30]
from_range = list(range(5))     # [0, 1, 2, 3, 4]
from_set = list({3, 1, 2})      # Order depends on set iteration

# Repetition (best used with immutable elements like numbers/strings/None)
zeros = [0] * 5                 # [0, 0, 0, 0, 0]
```

> **Note:** Avoid using `list` as a variable name (e.g., `list = [1, 2]`), as it shadows the built-in `list()` constructor.

---

## 2. Accessing Elements (Indexing & Slicing)

Lists are zero-indexed sequences and support both positive and negative indices as well as slice notation `list[start:stop:step]`.

### Indexing

```python
colors = ["red", "green", "blue", "yellow", "purple"]
# Index:    0       1       2        3         4
# Neg:     -5      -4      -3       -2        -1

print(colors[0])    # 'red'    (first element)
print(colors[2])    # 'blue'
print(colors[-1])   # 'purple' (last element)
print(colors[-2])   # 'yellow' (second-to-last element)

# Accessing an out-of-range index raises IndexError:
# colors[10]        # IndexError: list index out of range
```

### Slicing

Slicing returns a **new list** containing the selected range `[start, stop)` (inclusive of `start`, exclusive of `stop`). Out-of-bounds slice indices do not raise `IndexError`; they clamp gracefully.

```python
nums = [10, 20, 30, 40, 50, 60, 70]

print(nums[1:4])     # [20, 30, 40]
print(nums[:3])      # [10, 20, 30]       (from start to index 3)
print(nums[4:])      # [50, 60, 70]       (from index 4 to end)
print(nums[:])       # [10, 20, 30, 40, 50, 60, 70] (shallow copy)
print(nums[-3:])     # [50, 60, 70]       (last 3 items)
print(nums[:-2])     # [10, 20, 30, 40, 50] (all except last 2)

# Step parameter
print(nums[::2])     # [10, 30, 50, 70]   (every 2nd element)
print(nums[1::2])    # [20, 40, 60]       (every 2nd starting at index 1)
print(nums[::-1])    # [70, 60, 50, 40, 30, 20, 10] (reversed copy)
print(nums[5:1:-1])  # [60, 50, 40, 30]   (reverse slice)

# Out-of-range slices do not error
print(nums[3:100])   # [40, 50, 60, 70]
print(nums[100:200]) # []
```

You can also name slice patterns with `slice(start, stop, step)`:

```python
first_three = slice(0, 3)
print(nums[first_three])  # [10, 20, 30]
```

---

## 3. List Mutability

Unlike strings and tuples, lists are **mutable**: their elements can be modified, added, or removed **in place** without creating a new list object (keeping the same `id()`).

```python
items = ["a", "b", "c", "d", "e"]
print(id(items))

# Modify a single element
items[1] = "B"            # ['a', 'B', 'c', 'd', 'e']

# Modify a slice (can change list length!)
items[2:4] = ["X", "Y", "Z"]  # ['a', 'B', 'X', 'Y', 'Z', 'e']

# Insert via an empty slice
items[1:1] = ["inserted"]     # ['a', 'inserted', 'B', 'X', 'Y', 'Z', 'e']

# Delete via slice assignment or del
items[1:2] = []               # ['a', 'B', 'X', 'Y', 'Z', 'e']
del items[0]                  # ['B', 'X', 'Y', 'Z', 'e']
del items[1:3]                # ['B', 'Z', 'e']

print(id(items))              # Same object id as before
```

### Shared References (Aliasing)

Because variables hold references to objects, assigning one list variable to another makes both point to the same underlying list:

```python
a = [1, 2, 3]
b = a               # b is an alias for a, NOT a copy
b.append(4)
print(a)            # [1, 2, 3, 4]
print(a is b)       # True
```

---

## 4. List Methods

Most mutating list methods modify the list **in place** and return `None` (not the modified list).

| Method | Description | Return Value | Time Complexity (avg) |
| --- | --- | --- | --- |
| `append(x)` | Add `x` to the end | `None` | $O(1)$ amortized |
| `extend(iterable)` | Append all items from `iterable` | `None` | $O(k)$ where $k = \text{len(iterable)}$ |
| `insert(i, x)` | Insert `x` before index `i` | `None` | $O(n)$ |
| `remove(x)` | Remove first occurrence of `x` | `None` | $O(n)$ |
| `pop([i])` | Remove & return item at `i` (default last) | Removed item | $O(1)$ at end, $O(n)$ at index `i` |
| `clear()` | Remove all items | `None` | $O(n)$ |
| `index(x[, start[, end]])` | First index of `x` | `int` | $O(n)$ |
| `count(x)` | Number of occurrences of `x` | `int` | $O(n)$ |
| `sort(key=None, reverse=False)` | Sort list in place | `None` | $O(n \log n)$ |
| `reverse()` | Reverse list in place | `None` | $O(n)$ |
| `copy()` | Return a shallow copy | New `list` | $O(n)$ |

### Examples of Each Method

```python
nums = [10, 20, 30]

# 1. append(x): adds a single object (even if x is a list)
nums.append(40)           # [10, 20, 30, 40]
nums.append([50, 60])     # [10, 20, 30, 40, [50, 60]]
nums.pop()                # removes [50, 60] -> [10, 20, 30, 40]

# 2. extend(iterable): unpacks iterable and appends each element
nums.extend([50, 60])     # [10, 20, 30, 40, 50, 60]
nums.extend("ab")         # [10, 20, 30, 40, 50, 60, 'a', 'b']

# 3. insert(i, x): inserts x at index i, shifting elements right
letters = ["a", "c", "d"]
letters.insert(1, "b")    # ['a', 'b', 'c', 'd']
letters.insert(100, "z")  # Clamps to end -> ['a', 'b', 'c', 'd', 'z']

# 4. remove(x): removes the FIRST matching value; raises ValueError if missing
vals = [1, 2, 3, 2, 4]
vals.remove(2)            # [1, 3, 2, 4]
# vals.remove(99)         # ValueError: list.remove(x): x not in list

# 5. pop([i]): removes and returns element at index i (default: -1)
last = vals.pop()         # last = 4, vals = [1, 3, 2]
first = vals.pop(0)       # first = 1, vals = [3, 2]
# [].pop()                # IndexError: pop from empty list

# 6. clear(): empties the list in place
temp = [1, 2, 3]
temp.clear()              # temp is now []

# 7. index(x[, start[, end]]): finds first index of x; raises ValueError if missing
data = ["x", "y", "z", "y"]
print(data.index("y"))       # 1
print(data.index("y", 2))    # 3 (search starting at index 2)
# data.index("y", 1, 1)      # ValueError

# 8. count(x): counts occurrences of x
print(data.count("y"))       # 2
print(data.count("w"))       # 0

# 9. sort(): sorts in place (covered in detail in Section 10)
scores = [85, 92, 78, 95]
scores.sort()                # [78, 85, 92, 95]

# 10. reverse(): reverses in place (returns None)
scores.reverse()             # [95, 92, 85, 78]

# 11. copy(): returns a shallow copy (equivalent to scores[:])
scores_copy = scores.copy()
scores_copy.append(100)      # Does not affect `scores`
```

---

## 5. List Operations

Lists support concatenation, repetition, membership testing, length, and lexicographical comparisons.

```python
a = [1, 2, 3]
b = [4, 5]

# Concatenation (+ creates a new list; += mutates like extend)
c = a + b                # [1, 2, 3, 4, 5] (a and b unchanged)
a += b                   # a becomes [1, 2, 3, 4, 5] in place

# Repetition (* creates a new list)
repeated = [1, 2] * 3    # [1, 2, 1, 2, 1, 2]

# Membership testing (in / not in) -> O(n) linear scan
print(3 in c)            # True
print(99 not in c)       # True

# Length, min, max, sum
print(len(c))            # 5
print(min(c))            # 1
print(max(c))            # 5
print(sum(c))            # 15

# Lexicographical comparison (element-by-element from index 0)
print([1, 2, 3] == [1, 2, 3])   # True
print([1, 2, 3] < [1, 2, 4])    # True
print([1, 2] < [1, 2, 0])       # True (shorter prefix is smaller)
```

---

## 6. Nested Lists (2D Lists & Matrices)

Lists can contain other lists to represent grids, tables, or matrices.

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]

# Accessing elements: matrix[row][col]
print(matrix[0])         # [1, 2, 3] (first row)
print(matrix[1][2])      # 6         (row 1, column 2)

# Modifying an element
matrix[2][0] = 70        # [[1, 2, 3], [4, 5, 6], [70, 8, 9]]

# Iterating over a 2D list
for row in matrix:
    for val in row:
        print(val, end=" ")
    print()
```

### Creating a 2D Grid Safely

Never use `[[0] * cols] * rows`—every row will reference the exact same inner list! Use a list comprehension instead:

```python
rows, cols = 3, 4

# WRONG: all rows share the same inner list reference
bad_grid = [[0] * cols] * rows
bad_grid[0][0] = 99
print(bad_grid)
# [[99, 0, 0, 0], [99, 0, 0, 0], [99, 0, 0, 0]]  <-- BUG!

# CORRECT: each row is an independent list
good_grid = [[0] * cols for _ in range(rows)]
good_grid[0][0] = 99
print(good_grid)
# [[99, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]]
```

---

## 7. List Comprehensions

List comprehensions provide a concise, readable, and fast way to construct lists.

**General syntax:**

```python
[expression for item in iterable if condition]
```

### Basic Comprehension

```python
squares = [x ** 2 for x in range(1, 6)]
# [1, 4, 9, 16, 25]

upper_words = [w.upper() for w in ["hello", "world"]]
# ['HELLO', 'WORLD']
```

### Comprehension with Conditions

1. **Filtering (`if` at the end):** keeps only items matching the condition.
2. **Conditional expression (`if-else` before `for`):** transforms every item based on a condition.

```python
nums = range(1, 11)

# Filter: keep only even numbers
evens = [x for x in nums if x % 2 == 0]
# [2, 4, 6, 8, 10]

# Transform: label each number as 'even' or 'odd'
labels = ["even" if x % 2 == 0 else "odd" for x in range(1, 5)]
# ['odd', 'even', 'odd', 'even']

# Combined: square only positive numbers, clamp/transform as needed
vals = [-3, -1, 0, 2, 4]
pos_squares = [x ** 2 for x in vals if x > 0]
# [4, 16]
```

### Nested List Comprehensions

There are two distinct patterns:

1. **Multiple `for` clauses (flattening / Cartesian product):** read left to right, identical to nested `for` loops.
2. **Comprehension inside a comprehension (building a 2D list):** outer comprehension runs inside-out.

```python
# 1. Flatten a 2D list (equivalent to: for row in grid: for val in row)
grid = [[1, 2], [3, 4], [5, 6]]
flat = [val for row in grid for val in row]
# [1, 2, 3, 4, 5, 6]

# Cartesian product
pairs = [(x, y) for x in [1, 2] for y in ["a", "b"]]
# [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]

# 2. Build a 2D multiplication table (outer loop is on the right)
table = [[r * c for c in range(1, 4)] for r in range(1, 4)]
# [[1, 2, 3], [2, 4, 6], [3, 6, 9]]

# Transpose a matrix
transposed = [[row[i] for row in grid] for i in range(len(grid[0]))]
# [[1, 3, 5], [2, 4, 6]]
```

> **Readability rule:** If a comprehension has more than two `for` clauses or complex conditions, prefer a regular `for` loop.

---

## 8. List Iteration Techniques

Python offers clean idioms for iterating without manual index counters.

```python
names = ["Alice", "Bob", "Charlie"]
scores = [88, 95, 79]

# 1. Direct iteration (most Pythonic when index isn't needed)
for name in names:
    print(name)

# 2. enumerate(): get both index and value
for idx, name in enumerate(names):
    print(f"{idx}: {name}")

# Start index at 1 instead of 0
for rank, name in enumerate(names, start=1):
    print(f"#{rank} {name}")

# 3. zip(): iterate over multiple lists in parallel (stops at shortest)
for name, score in zip(names, scores):
    print(f"{name} scored {score}")

# Strict zip (Python 3.10+): raises ValueError if lengths differ
for name, score in zip(names, scores, strict=True):
    print(name, score)

# 4. itertools.zip_longest(): pad shorter lists
from itertools import zip_longest
for a, b in zip_longest([1, 2, 3], ["x"], fillvalue="-"):
    print(a, b)  # (1, 'x'), (2, '-'), (3, '-')

# 5. reversed(): iterate backwards without copying the list
for name in reversed(names):
    print(name)  # Charlie, Bob, Alice

# 6. Index-based iteration (when mutating elements in place by index)
nums = [1, 2, 3, 4]
for i in range(len(nums)):
    nums[i] *= 10
# nums is now [10, 20, 30, 40]
```

---

## 9. List Unpacking and Starred Expressions

Sequence unpacking assigns list elements directly to variables. A starred target (`*var`) captures zero or more remaining items as a `list`.

```python
# Basic unpacking (count must match exactly)
point = [10, 20, 30]
x, y, z = point          # x=10, y=20, z=30

# Starred unpacking (*rest is always a list, even if empty)
first, *middle, last = [1, 2, 3, 4, 5]
# first=1, middle=[2, 3, 4], last=5

head, *tail = [10, 20, 30]
# head=10, tail=[20, 30]

only, *rest = [42]
# only=42, rest=[]

# Ignoring elements with _ or *_
first, *_, last = [100, 200, 300, 400]
# first=100, last=400

# Unpacking inside loops
records = [("Alice", 90, 95), ("Bob", 70, 80, 85)]
for name, *grades in records:
    print(name, grades)
# Alice [90, 95]
# Bob [70, 80, 85]

# Merging lists using * (unpacking operator)
a = [1, 2]
b = [3, 4]
merged = [0, *a, *b, 5]  # [0, 1, 2, 3, 4, 5]
```

> **Constraint:** Only **one** starred expression is allowed per unpacking target (`*a, *b = [1, 2, 3]` raises `SyntaxError`).

---

## 10. Sorting Lists (`sort` vs `sorted`)

Python uses **Timsort**, a stable, adaptive sorting algorithm with $O(n \log n)$ worst-case time complexity. *Stable* means items with equal keys retain their original relative order.

### `list.sort()` vs `sorted()`

| Feature | `list.sort()` | `sorted(iterable)` |
| --- | --- | --- |
| Behavior | Sorts the list **in place** | Returns a **new sorted list** |
| Return value | `None` | New `list` |
| Works on | Lists only | Any iterable (list, tuple, set, dict, str, generator) |
| Memory | Mutates original (more memory efficient) | Allocates a new list |

```python
nums = [3, 1, 4, 1, 5, 9]

# sorted() leaves original intact
new_nums = sorted(nums)       # [1, 1, 3, 4, 5, 9]
print(nums)                   # [3, 1, 4, 1, 5, 9]

# .sort() mutates original and returns None
result = nums.sort()
print(result)                 # None
print(nums)                   # [1, 1, 3, 4, 5, 9]
```

### `reverse` Parameter

```python
nums = [3, 1, 4, 1, 5]
print(sorted(nums, reverse=True))   # [5, 4, 3, 1, 1]
nums.sort(reverse=True)             # [5, 4, 3, 1, 1]
```

### `key` Parameter

`key` accepts a single-argument function applied to each element before comparing.

```python
words = ["banana", "Pie", "apple", "fig"]

# Case-insensitive alphabetical sort
print(sorted(words, key=str.lower))
# ['apple', 'banana', 'fig', 'Pie']

# Sort by length
print(sorted(words, key=len))
# ['Pie', 'fig', 'banana', 'apple']  (stable: 'Pie' stays before 'fig')

# Multi-key sorting using a tuple key
students = [
    ("Alice", 90),
    ("Bob", 85),
    ("Charlie", 90),
    ("Diana", 85),
]
# Sort by score descending, then name ascending
ranked = sorted(students, key=lambda s: (-s[1], s[0]))
# [('Alice', 90), ('Charlie', 90), ('Bob', 85), ('Diana', 85)]

# Using operator.itemgetter / attrgetter (faster than lambda)
from operator import itemgetter
by_score = sorted(students, key=itemgetter(1), reverse=True)
```

---

## 11. Copying Lists (Shallow vs Deep Copy)

Because lists store references to objects, copying a list has two levels:

- **Shallow copy:** Creates a new outer list, but inner mutable objects (e.g., nested lists, dicts) are still shared between the original and the copy.
- **Deep copy:** Recursively copies the outer list *and* all nested objects so nothing is shared.

```python
import copy

original = [[1, 2], [3, 4]]

# Ways to make a shallow copy:
shallow_1 = original.copy()
shallow_2 = original[:]
shallow_3 = list(original)
shallow_4 = copy.copy(original)

# Mutating a top-level element affects only shallow_1:
shallow_1.append([5, 6])
print(original)   # [[1, 2], [3, 4]]

# Mutating a nested element affects BOTH original and shallow_1!
shallow_1[0][0] = 999
print(original)   # [[999, 2], [3, 4]]

# Deep copy: completely independent at all nesting levels
deep = copy.deepcopy(original)
deep[0][0] = 1
print(original)   # [[999, 2], [3, 4]]  (unchanged)
print(deep)       # [[1, 2], [3, 4]]
```

> **Rule of thumb:** Use `.copy()` or `[:]` for flat lists (or lists of immutables like ints/strings/tuples). Use `copy.deepcopy()` when you have nested mutable structures that will be mutated independently.

---

## 12. List as Stack and Queue

### List as a Stack (LIFO — Last-In, First-Out)

Lists work efficiently as stacks because `append()` and `pop()` at the end are $O(1)$ amortized.

```python
stack = []

# Push
stack.append("task1")
stack.append("task2")
stack.append("task3")

# Peek at top without removing
top = stack[-1]          # 'task3'

# Pop
print(stack.pop())       # 'task3'
print(stack.pop())       # 'task2'
print(stack)             # ['task1']
```

### List as a Queue (FIFO — First-In, First-Out)

Using `list.insert(0, x)` or `list.pop(0)` for a queue is **$O(n)$** because all remaining elements must shift in memory. For real queues, always prefer `collections.deque`, which provides $O(1)$ operations at both ends.

```python
# Inefficient with plain list (O(n) per pop(0))
bad_queue = ["a", "b", "c"]
first = bad_queue.pop(0)

# Recommended: collections.deque (O(1) append and popleft)
from collections import deque

queue = deque(["a", "b", "c"])
queue.append("d")        # Enqueue at right end -> deque(['a', 'b', 'c', 'd'])
item = queue.popleft()   # Dequeue from left end -> 'a'
print(queue)             # deque(['b', 'c', 'd'])
```

---

## 13. Common List Patterns and Pitfalls

### Common Patterns

```python
# 1. Deduplicate while preserving order
items = [3, 1, 2, 1, 4, 3, 2]
unique = list(dict.fromkeys(items))    # [3, 1, 2, 4]

# 2. Check if all / any elements match a condition
nums = [2, 4, 6, 8]
all_even = all(n % 2 == 0 for n in nums)   # True
has_gt_5 = any(n > 5 for n in nums)        # True

# 3. Chunking a list into fixed-size batches
data = [1, 2, 3, 4, 5, 6, 7]
size = 3
chunks = [data[i:i + size] for i in range(0, len(data), size)]
# [[1, 2, 3], [4, 5, 6], [7]]

# 4. Safe element removal without ValueError
vals = [1, 2, 3]
target = 99
if target in vals:
    vals.remove(target)

# 5. Finding all indices of a matching value
letters = ["a", "b", "a", "c", "a"]
a_indices = [i for i, ch in enumerate(letters) if ch == "a"]
# [0, 2, 4]

# 6. Checking emptiness (Pythonic truthiness)
if not vals:
    print("List is empty")
```

### Common Pitfalls

#### Pitfall 1: Mutable Default Argument in Functions

Default arguments are evaluated **once** at function definition time, not on each call.

```python
# BUG: the same list is shared across calls
def add_item_bad(item, target=[]):
    target.append(item)
    return target

print(add_item_bad(1))  # [1]
print(add_item_bad(2))  # [1, 2]  <-- unexpected!

# FIX: use None and create a new list inside the function
def add_item_good(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target
```

#### Pitfall 2: Mutating a List While Iterating Over It

Removing or inserting elements shifts indices and causes skipped items.

```python
# BUG: skips elements after removal
nums = [1, 2, 3, 4, 5, 6]
for n in nums:
    if n % 2 == 0:
        nums.remove(n)   # Unpredictable / buggy for consecutive matches

# FIX 1: build a new list with a comprehension
nums = [n for n in nums if n % 2 != 0]

# FIX 2: iterate over a shallow copy (nums[:]) if in-place mutation is required
nums = [1, 2, 2, 3, 4]
for n in nums[:]:
    if n == 2:
        nums.remove(n)
```

#### Pitfall 3: Assigning the Return Value of In-Place Methods

```python
# BUG: sort(), reverse(), append() return None!
nums = [3, 1, 2]
nums = nums.sort()
print(nums)              # None

# FIX: call in-place without reassignment, or use sorted()/reversed()
nums = [3, 1, 2]
nums.sort()              # nums is [1, 2, 3]
```

#### Pitfall 4: `append()` vs `extend()` Confusion

```python
a = [1, 2]
a.append([3, 4])         # [1, 2, [3, 4]]  (adds list as single element)

b = [1, 2]
b.extend([3, 4])         # [1, 2, 3, 4]    (adds each element)
```

#### Pitfall 5: `is` vs `==`

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)            # True  (same values)
print(a is b)            # False (different objects in memory)
```
