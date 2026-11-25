# Sets in Python

A **set** (`set`) is a built-in Python collection type that stores **unordered**, **unique**, and **hashable** elements inside a **mutable** container. Backed by a hash table, sets provide fast $O(1)$ average-case membership testing and clean, expressive operations modeled on mathematical set theory.

---

## Table of Contents

1. [Creating Sets (`set()`, `{}`)](#1-creating-sets-set-)
2. [Set Properties (Unordered, Unique, Mutable)](#2-set-properties-unordered-unique-mutable)
3. [Adding and Removing Elements (`add`, `update`, `remove`, `discard`, `pop`, `clear`)](#3-adding-and-removing-elements-add-update-remove-discard-pop-clear)
4. [Set Operations (`union`, `intersection`, `difference`, `symmetric_difference`)](#4-set-operations-union-intersection-difference-symmetric_difference)
5. [Set Operator Equivalents (`|`, `&`, `-`, `^`)](#5-set-operator-equivalents----)
6. [Set Methods (`issubset`, `issuperset`, `isdisjoint`)](#6-set-methods-issubset-issuperset-isdisjoint)
7. [Frozen Sets (`frozenset`)](#7-frozen-sets-frozenset)
8. [Set Comprehension](#8-set-comprehension)
9. [When to Use Sets](#9-when-to-use-sets)
10. [Set vs List Performance Comparison](#10-set-vs-list-performance-comparison)
11. [Mathematical Set Theory Operations](#11-mathematical-set-theory-operations)
12. [Common Set Patterns](#12-common-set-patterns)

---

## 1. Creating Sets (`set()`, `{}`)

Sets can be created using curly braces `{...}` with comma-separated elements, or with the `set()` constructor.

```python
# Using curly-brace literal syntax
fruits = {"apple", "banana", "cherry"}
numbers = {1, 2, 3, 2, 1}
print(numbers)  # {1, 2, 3}  (duplicates are automatically removed)

# Using the set() constructor with any iterable
from_list = set([10, 20, 20, 30])   # {10, 20, 30}
from_tuple = set(("a", "b", "a"))   # {'a', 'b'}
from_string = set("mississippi")    # {'m', 'i', 's', 'p'}
from_range = set(range(5))          # {0, 1, 2, 3, 4}
```

### The Empty Set Gotcha

Empty curly braces `{}` create an empty **dictionary** (`dict`), **not** an empty set. Always use `set()` to create an empty set.

```python
not_a_set = {}
empty_set = set()

print(type(not_a_set))  # <class 'dict'>
print(type(empty_set))  # <class 'set'>
print(len(empty_set))   # 0
```

> **Note:** Passing a dictionary directly to `set(my_dict)` iterates over its **keys** by default. Use `set(my_dict.values())` or `set(my_dict.items())` if you want values or key-value pairs.

---

## 2. Set Properties (Unordered, Unique, Mutable)

### Unordered (No Indexing or Slicing)

Sets do not maintain positional indices. Elements are stored according to their hash values, so iteration order should not be relied upon for ordering logic.

```python
colors = {"red", "green", "blue"}

# Indexing and slicing raise TypeError
# colors[0]    -> TypeError: 'set' object is not subscriptable
# colors[0:2]  -> TypeError: 'set' object is not subscriptable

for color in colors:
    print(color)  # Order depends on internal hash table slots
```

### Unique Elements

Every element in a set is distinct. If two values are considered equal (`a == b`) and share the same hash (`hash(a) == hash(b)`), only the first one inserted is kept.

```python
# Python treats 1, 1.0, and True as equal with the same hash
s = {1, 1.0, True}
print(s)  # {1}

s2 = {False, 0, 0.0}
print(s2)  # {False}
```

### Mutable Container, Hashable Elements

A `set` itself is **mutable**—you can add and remove elements in place. However, every **element inside** a set must be **hashable** (typically immutable types).

| Allowed (Hashable) | Not Allowed (Unhashable) |
| :--- | :--- |
| `int`, `float`, `bool`, `str`, `bytes` | `list` |
| `tuple` (if all its items are hashable) | `dict` |
| `frozenset` | `set` |
| Custom objects with `__hash__` & `__eq__` | `tuple` containing a `list` or `set` |

```python
# Valid elements
valid = {42, "hello", (1, 2), frozenset({3, 4})}

# Invalid elements raise TypeError
# bad = {[1, 2]}         -> TypeError: unhashable type: 'list'
# bad = {{"a": 1}}       -> TypeError: unhashable type: 'dict'
# bad = {{1, 2}}         -> TypeError: unhashable type: 'set'
# bad = {(1, [2, 3])}    -> TypeError: unhashable type: 'list'
```

Because a `set` is mutable, it is unhashable (`hash(set())` raises `TypeError`) and cannot be used as a dictionary key or inside another `set`. Use [`frozenset`](#7-frozen-sets-frozenset) when you need a hashable set.

---

## 3. Adding and Removing Elements (`add`, `update`, `remove`, `discard`, `pop`, `clear`)

All of the methods below mutate the set **in place**.

### Adding Elements

- **`add(elem)`**: Adds a single hashable element. Does nothing if the element is already present.
- **`update(*others)`**: Adds all elements from one or more iterables (lists, tuples, strings, other sets, etc.).

```python
tags = {"python", "coding"}

tags.add("sets")
tags.add("python")  # Duplicate is ignored
print(tags)         # {'python', 'coding', 'sets'}

# update() unpacks each iterable passed to it
tags.update(["data", "sets"], ("algorithms",))
print(tags)  # {'python', 'coding', 'sets', 'data', 'algorithms'}
```

### Removing Elements

- **`remove(elem)`**: Removes `elem`. Raises `KeyError` if `elem` is not found.
- **`discard(elem)`**: Removes `elem` if present. Does **not** raise an error if `elem` is missing.
- **`pop()`**: Removes and returns an **arbitrary** element. Raises `KeyError` if the set is empty.
- **`clear()`**: Removes all elements, leaving an empty `set()`.

```python
items = {"a", "b", "c", "d"}

# remove() - strict
items.remove("b")
# items.remove("z")  # KeyError: 'z'

# discard() - safe when item might not exist
items.discard("c")
items.discard("z")   # No error!

# pop() - removes and returns an arbitrary element
removed = items.pop()
print(f"Popped: {removed}, Remaining: {items}")

# clear() - empties the set
items.clear()
print(items)  # set()
# items.pop()  # KeyError: 'pop from an empty set'
```

### Quick Comparison of Removal Methods

| Method | Missing Element Behavior | Empty Set Behavior | Returns |
| :--- | :--- | :--- | :--- |
| `remove(x)` | Raises `KeyError` | Raises `KeyError` | `None` |
| `discard(x)` | No error (no-op) | No error (no-op) | `None` |
| `pop()` | N/A | Raises `KeyError` | Removed element |
| `clear()` | N/A | No error (no-op) | `None` |

---

## 4. Set Operations (`union`, `intersection`, `difference`, `symmetric_difference`)

Set operation methods return a **new set** without modifying the originals. A major advantage of the method forms is that their arguments can be **any iterable** (lists, tuples, strings, generators, etc.), not just sets.

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

# Union: elements in a, or b, or both
print(a.union(b))                 # {1, 2, 3, 4, 5, 6}

# Intersection: elements common to both a and b
print(a.intersection(b))          # {3, 4}

# Difference: elements in a that are NOT in b
print(a.difference(b))            # {1, 2}
print(b.difference(a))            # {5, 6}  (order matters!)

# Symmetric Difference: elements in either a or b, but NOT both
print(a.symmetric_difference(b))  # {1, 2, 5, 6}
```

### Passing Multiple Iterables & Non-Set Iterables

`union`, `intersection`, and `difference` accept multiple iterable arguments at once (whereas `symmetric_difference` takes exactly one iterable).

```python
a = {1, 2, 3, 4, 5}

# Accepting lists, tuples, and multiple arguments
print(a.union([5, 6], (7, 8)))         # {1, 2, 3, 4, 5, 6, 7, 8}
print(a.intersection([2, 3, 4], {3, 4, 9}))  # {3, 4}
print(a.difference([1, 2], (4,)))      # {3, 5}
print(a.symmetric_difference([4, 5, 6]))     # {1, 2, 3, 6}
```

### In-Place Mutation Counterparts

Each operation also has an in-place version that modifies `a` directly and returns `None`:

| Non-Mutating Method | In-Place Mutating Method |
| :--- | :--- |
| `a.union(*others)` | `a.update(*others)` |
| `a.intersection(*others)` | `a.intersection_update(*others)` |
| `a.difference(*others)` | `a.difference_update(*others)` |
| `a.symmetric_difference(other)` | `a.symmetric_difference_update(other)` |

---

## 5. Set Operator Equivalents (`|`, `&`, `-`, `^`)

Python provides concise binary operators for standard set operations:

| Operation | Operator | Method | Meaning |
| :--- | :---: | :--- | :--- |
| Union | `a \| b` | `a.union(b)` | Elements in `a` or `b` |
| Intersection | `a & b` | `a.intersection(b)` | Elements in both `a` and `b` |
| Difference | `a - b` | `a.difference(b)` | Elements in `a` but not `b` |
| Symmetric Difference | `a ^ b` | `a.symmetric_difference(b)` | Elements in `a` or `b`, not both |

```python
frontend = {"HTML", "CSS", "JavaScript", "TypeScript"}
backend = {"Python", "Go", "TypeScript", "SQL"}

print(frontend | backend)  # {'HTML', 'CSS', 'JavaScript', 'TypeScript', 'Python', 'Go', 'SQL'}
print(frontend & backend)  # {'TypeScript'}
print(frontend - backend)  # {'HTML', 'CSS', 'JavaScript'}
print(frontend ^ backend)  # {'HTML', 'CSS', 'JavaScript', 'Python', 'Go', 'SQL'}
```

### Augmented Assignment Operators (In-Place)

| Operator | Equivalent Method |
| :--- | :--- |
| `a \|= b` | `a.update(b)` |
| `a &= b` | `a.intersection_update(b)` |
| `a -= b` | `a.difference_update(b)` |
| `a ^= b` | `a.symmetric_difference_update(b)` |

```python
skills = {"Python", "SQL"}
skills |= {"Docker", "Git"}   # Union in-place -> {'Python', 'SQL', 'Docker', 'Git'}
skills -= {"Git"}             # Difference in-place -> {'Python', 'SQL', 'Docker'}
```

### Operators vs. Methods: Key Difference

Unlike methods, **operators require both operands to be `set` (or `frozenset`) instances**. Mixing a `set` and a `list` with an operator raises `TypeError`.

```python
s = {1, 2, 3}

# Works: method converts iterable
print(s | {3, 4})         # {1, 2, 3, 4}
print(s.union([3, 4]))    # {1, 2, 3, 4}

# Fails: operator requires set/frozenset
# print(s | [3, 4])       # TypeError: unsupported operand type(s) for |: 'set' and 'list'
```

---

## 6. Set Methods (`issubset`, `issuperset`, `isdisjoint`)

These predicate methods test relationships between sets and return a `bool` (`True` or `False`).

```python
small = {1, 2}
large = {1, 2, 3, 4}
other = {5, 6}

# issubset: True if every element of small is in large
print(small.issubset(large))    # True
print(large.issubset(large))    # True  (every set is a subset of itself)

# issuperset: True if large contains every element of small
print(large.issuperset(small))  # True

# isdisjoint: True if two sets share NO common elements
print(small.isdisjoint(other))  # True
print(small.isdisjoint(large))  # False (they share 1 and 2)
```

### Comparison Operator Equivalents

Subset and superset checks also have comparison operators (note: like `|`, `&`, `-`, `^`, operators require both sides to be sets/frozensets, while methods accept any iterable):

| Relationship | Method | Operator | Notes |
| :--- | :--- | :---: | :--- |
| Subset | `a.issubset(b)` | `a <= b` | `True` if all items of `a` are in `b` |
| Proper subset | — | `a < b` | `True` if `a <= b` and `a != b` |
| Superset | `a.issuperset(b)` | `a >= b` | `True` if all items of `b` are in `a` |
| Proper superset | — | `a > b` | `True` if `a >= b` and `a != b` |
| Disjoint | `a.isdisjoint(b)` | — | `True` if `len(a & b) == 0` |
| Equal | — | `a == b` | `True` if both have the exact same elements |

```python
a = {1, 2}
b = {1, 2}
c = {1, 2, 3}

print(a <= b)  # True  (subset)
print(a < b)   # False (not a proper subset because a == b)
print(a < c)   # True  (proper subset)
print(c > a)   # True  (proper superset)

# Method advantage: works with non-set iterables
print(a.issubset([1, 2, 3]))  # True
print(a.isdisjoint("abc"))    # True
```

---

## 7. Frozen Sets (`frozenset`)

A `frozenset` is an **immutable** and **hashable** version of a `set`. Once created, its elements cannot be added, removed, or modified.

```python
# Creating frozensets
fs1 = frozenset([1, 2, 3, 2])
fs2 = frozenset("abc")
empty_fs = frozenset()

print(fs1)  # frozenset({1, 2, 3})

# Mutating methods do NOT exist on frozenset
# fs1.add(4)     -> AttributeError: 'frozenset' object has no attribute 'add'
# fs1.remove(1)  -> AttributeError: 'frozenset' object has no attribute 'remove'
```

### Why Use `frozenset`?

1. **Sets of sets**: Since `set` is unhashable, nested sets must use `frozenset`.
2. **Dictionary keys**: Unordered combinations (like undirected graph edges `{u, v}` or multi-tag cache keys) can serve as dictionary keys when wrapped in `frozenset`.
3. **Safety / Constants**: Guarantees a collection of constants cannot be accidentally mutated.

```python
# 1. Nested sets (set of frozensets)
edges = {
    frozenset({"A", "B"}),
    frozenset({"B", "C"}),
    frozenset({"B", "A"}),  # Duplicate of {"A", "B"}!
}
print(len(edges))  # 2

# 2. Dictionary keys representing unordered pairs
distances = {
    frozenset({"NYC", "BOS"}): 215,
    frozenset({"NYC", "PHL"}): 95,
}
print(distances[frozenset({"BOS", "NYC"})])  # 215

# 3. Non-mutating operations work normally
a = frozenset({1, 2, 3})
b = frozenset({3, 4, 5})
print(a | b)  # frozenset({1, 2, 3, 4, 5})
print(a & b)  # frozenset({3})

# Mixing set and frozenset: result takes the type of the left-hand operand
s = {1, 2}
print(type(s | a))  # <class 'set'>
print(type(a | s))  # <class 'frozenset'>
```

---

## 8. Set Comprehension

Set comprehensions provide a concise way to build a set by transforming and filtering an iterable.

### Syntax

```python
{expression for item in iterable if condition}
```

### Examples

```python
# Squares of numbers 0..5
squares = {x ** 2 for x in range(6)}
print(squares)  # {0, 1, 4, 9, 16, 25}

# Unique normalized words longer than 3 characters
words = ["Python", "java", "PYTHON", "Go", "rust", "Java"]
long_words = {w.lower() for w in words if len(w) > 3}
print(long_words)  # {'python', 'java', 'rust'}

# Extracting unique file extensions
files = ["app.py", "style.css", "utils.py", "README.md", "test.py"]
extensions = {f.rsplit(".", 1)[-1] for f in files if "." in f}
print(extensions)  # {'py', 'css', 'md'}
```

> **Tip:** Be careful not to confuse `{x for x in items}` (set comprehension) with `{k: v for k, v in pairs}` (dict comprehension). Notice the colon `:` in dictionary comprehensions.

---

## 9. When to Use Sets

Use a `set` instead of a `list` or `tuple` when:

1. **Fast membership testing (`x in collection`)**: You repeatedly check whether items exist in a large collection (e.g., banned users, stop words, visited nodes).
2. **Removing duplicates**: You need unique elements and order does not matter (or you track seen elements separately).
3. **Comparing two collections**: You need elements in both, only in one, or missing between two datasets (syncing records, permission checks, tag matching).
4. **Order independence**: You are modeling mathematical sets, tags, categories, or feature flags where sequence position has no meaning.

### When NOT to Use Sets

- You need **indexing/slicing** (`items[0]`, `items[-1]`).
- You need to **preserve duplicates** (consider `list` or `collections.Counter`).
- You need **guaranteed insertion order** (use `list` or `dict`).
- Elements are **unhashable** (like lists or dictionaries).

---

## 10. Set vs List Performance Comparison

A `list` is a dynamic array of pointers, while a `set` is a hash table. This leads to dramatically different time complexities for search and removal.

### Time & Space Complexity

| Operation | `set` (Average) | `set` (Worst Case) | `list` (Average) | Notes |
| :--- | :---: | :---: | :---: | :--- |
| Membership (`x in c`) | $O(1)$ | $O(n)$ | $O(n)$ | Hash lookup vs. linear scan |
| Insert (`add` / `append`) | $O(1)$ | $O(n)$ | $O(1)$ | Amortized due to resizing |
| Remove by value (`discard` / `remove`) | $O(1)$ | $O(n)$ | $O(n)$ | List must search and shift elements |
| Remove arbitrary / last (`pop`) | $O(1)$ | $O(1)$ | $O(1)$ (end) / $O(n)$ (start) | `list.pop(0)` shifts all items |
| Length (`len(c)`) | $O(1)$ | $O(1)$ | $O(1)$ | Stored as an integer attribute |
| Union / Intersection (`a \| b`, `a & b`) | $O(\|a\| + \|b\|)$ / $O(\min(\|a\|, \|b\|))$ | $O(\|a\| \cdot \|b\|)$ | N/A | Hash-based vs. nested loops $O(n \cdot m)$ |
| Memory usage | Higher | — | Lower | Sets keep empty hash slots for low collisions |

### Benchmark Script (`timeit`)

You can verify the lookup speed difference locally with Python's `timeit` module:

```python
import timeit

n = 100_000
data_list = list(range(n))
data_set = set(data_list)
target = n - 1  # Worst-case position for linear scan in a list

list_time = timeit.timeit(lambda: target in data_list, number=1000)
set_time = timeit.timeit(lambda: target in data_set, number=1000)

print(f"List lookup (1,000x): {list_time:.6f}s")
print(f"Set lookup  (1,000x): {set_time:.6f}s")
```

With 100,000 elements, searching the list scans up to 100,000 elements per lookup, while the set jumps directly to the hashed bucket in near-constant time—typically orders of magnitude faster.

### The Memory Trade-Off

Sets trade memory for speed. Because hash tables maintain open slots (typically keeping load factor below ~2/3) plus hash values per entry, a `set` consumes noticeably more memory than a `list` storing the same objects:

```python
import sys

nums = list(range(1000))
print("List bytes:", sys.getsizeof(nums))
print("Set bytes: ", sys.getsizeof(set(nums)))
```

---

## 11. Mathematical Set Theory Operations

Python sets directly model classical set theory. Here is how mathematical notation maps to Python:

| Concept | Math Notation | Python Code |
| :--- | :---: | :--- |
| Set definition | $A = \{1, 2, 3\}$ | `A = {1, 2, 3}` |
| Empty set | $\emptyset$ | `set()` |
| Element membership | $x \in A$ | `x in A` |
| Non-membership | $x \notin A$ | `x not in A` |
| Cardinality | $\|A\|$ | `len(A)` |
| Union | $A \cup B$ | `A \| B` |
| Intersection | $A \cap B$ | `A & B` |
| Set difference | $A \setminus B$ | `A - B` |
| Symmetric difference | $A \triangle B$ | `A ^ B` |
| Subset | $A \subseteq B$ | `A <= B` |
| Proper subset | $A \subset B$ | `A < B` |
| Superset | $A \supseteq B$ | `A >= B` |
| Proper superset | $A \supset B$ | `A > B` |
| Disjoint sets | $A \cap B = \emptyset$ | `A.isdisjoint(B)` |
| Complement (given universe $U$) | $A^c = U \setminus A$ | `U - A` |
| Cartesian product | $A \times B$ | `{(x, y) for x in A for y in B}` |

### Verifying Set Theory Laws in Python

```python
U = {1, 2, 3, 4, 5, 6}  # Universal set
A = {1, 2, 3}
B = {3, 4, 5}
C = {2, 5, 6}

# Complement of A relative to U
A_comp = U - A  # {4, 5, 6}
B_comp = U - B  # {1, 2, 6}

# De Morgan's Laws:
# 1) (A ∪ B)^c == A^c ∩ B^c
assert (U - (A | B)) == (A_comp & B_comp)

# 2) (A ∩ B)^c == A^c ∪ B^c
assert (U - (A & B)) == (A_comp | B_comp)

# Distributive Law: A ∩ (B ∪ C) == (A ∩ B) ∪ (A ∩ C)
assert (A & (B | C)) == ((A & B) | (A & C))

# Symmetric Difference Identity: A △ B == (A \ B) ∪ (B \ A) == (A ∪ B) \ (A ∩ B)
assert (A ^ B) == ((A - B) | (B - A)) == ((A | B) - (A & B))

# Cartesian Product A × B
cartesian = {(a, b) for a in {1, 2} for b in {"x", "y"}}
print(cartesian)  # {(1, 'x'), (1, 'y'), (2, 'x'), (2, 'y')}
```

### Power Set (All Subsets of a Set)

Because `set` is unhashable, represent a power set as a `set` of `frozenset`s using `itertools.combinations`:

```python
from itertools import combinations

def power_set(s):
    items = list(s)
    return {
        frozenset(combo)
        for r in range(len(items) + 1)
        for combo in combinations(items, r)
    }

print(power_set({1, 2}))
# {frozenset(), frozenset({1}), frozenset({2}), frozenset({1, 2})}
```

---

## 12. Common Set Patterns

### 1. Order-Preserving Deduplication

Converting a list to `set(items)` loses the original order. Use a `seen` set (or `dict.fromkeys()`) to deduplicate while keeping the first occurrence order:

```python
def unique_ordered(items):
    seen = set()
    result = []
    for item in items:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

print(unique_ordered([3, 1, 2, 1, 4, 3, 5]))  # [3, 1, 2, 4, 5]

# Concise alternative using dict.fromkeys (Python 3.7+ preserves insertion order)
print(list(dict.fromkeys([3, 1, 2, 1, 4, 3, 5])))  # [3, 1, 2, 4, 5]
```

### 2. Checking Uniqueness & Finding Duplicates

```python
# Check if all items are unique
def all_unique(items):
    return len(items) == len(set(items))

print(all_unique([1, 2, 3]))     # True
print(all_unique([1, 2, 2, 3]))  # False

# Find all duplicate items in O(n) time
def find_duplicates(items):
    seen = set()
    duplicates = set()
    for item in items:
        if item in seen:
            duplicates.add(item)
        else:
            seen.add(item)
    return duplicates

print(find_duplicates(["a", "b", "a", "c", "b", "d"]))  # {'a', 'b'}
```

### 3. Diffing Two Snapshots (Added, Removed, Unchanged)

Great for comparing configurations, database IDs, or inventory lists:

```python
yesterday_users = {"alice", "bob", "charlie"}
today_users = {"bob", "charlie", "diana", "evan"}

added = today_users - yesterday_users      # {'diana', 'evan'}
removed = yesterday_users - today_users    # {'alice'}
unchanged = yesterday_users & today_users  # {'bob', 'charlie'}
```

### 4. Fast Whitelist / Blacklist Filtering

```python
STOP_WORDS = {"a", "an", "the", "is", "in", "of", "to"}
sentence = "the quick brown fox is in the garden"

meaningful_words = [
    word for word in sentence.split() if word not in STOP_WORDS
]
print(meaningful_words)  # ['quick', 'brown', 'fox', 'garden']
```

### 5. Permission / Requirement Validation

```python
required_scopes = {"read:users", "write:posts"}
user_scopes = {"read:users", "write:posts", "delete:comments"}

has_access = required_scopes <= user_scopes  # True
missing = required_scopes - user_scopes      # set()
```

### 6. Graph Traversal (`visited` Set)

Sets prevent infinite loops when traversing graphs or trees with cycles:

```python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue = deque([start])
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph.get(node, ()):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return order

graph = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A", "D"],
    "D": ["B", "C"],
}
print(bfs(graph, "A"))  # ['A', 'B', 'C', 'D']
```

### 7. Finding Missing Numbers in a Range

```python
nums = [1, 2, 4, 6, 7]
missing = sorted(set(range(1, 8)) - set(nums))
print(missing)  # [3, 5]
```
