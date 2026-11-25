# Dictionaries in Python

A **dictionary** (`dict`) is a built-in, mutable mapping type that stores **key–value pairs**. Keys must be **hashable** (immutable types like `str`, `int`, `float`, `tuple` of hashables) and unique within a dictionary, while values can be any Python object and may repeat.

> **Key characteristics:**
> - **Fast lookups:** Average $O(1)$ time complexity for lookup, insertion, and deletion (implemented as a hash table).
> - **Insertion-ordered:** Preserves insertion order as a language guarantee since Python 3.7 (CPython 3.6 implementation detail).
> - **Mutable:** You can add, modify, and remove entries in place.

---

## Table of Contents

1. [Creating Dictionaries](#1-creating-dictionaries)
2. [Accessing Values](#2-accessing-values)
3. [Adding, Updating, and Deleting Items](#3-adding-updating-and-deleting-items)
4. [Dictionary Methods](#4-dictionary-methods)
5. [Iterating Over Dictionaries](#5-iterating-over-dictionaries)
6. [Dictionary Comprehension](#6-dictionary-comprehension)
7. [Nested Dictionaries](#7-nested-dictionaries)
8. [Merging Dictionaries](#8-merging-dictionaries)
9. [`OrderedDict`, `defaultdict`, and `Counter` from `collections`](#9-ordereddict-defaultdict-and-counter-from-collections)
10. [Dictionary vs Other Data Structures](#10-dictionary-vs-other-data-structures)
11. [Dictionary Unpacking](#11-dictionary-unpacking)
12. [Common Dictionary Patterns and Best Practices](#12-common-dictionary-patterns-and-best-practices)
13. [Dictionary View Objects](#13-dictionary-view-objects)

---

## 1. Creating Dictionaries

### 1.1 Dictionary Literals (`{}`)

The most common and readable way to create a dictionary is with curly braces `{}` containing `key: value` pairs.

```python
# Empty dictionary (Note: {} creates an empty dict, NOT an empty set)
empty = {}
print(type(empty))  # <class 'dict'>

# Populated dictionary
person = {
    "name": "Alice",
    "age": 30,
    "roles": ["admin", "editor"],
}

# Keys can be any hashable type
mixed_keys = {
    1: "integer key",
    "two": "string key",
    (3, 4): "tuple key",
}

# Duplicate keys: later value overwrites earlier one
dup = {"a": 1, "b": 2, "a": 99}
print(dup)  # {'a': 99, 'b': 2}
```

### 1.2 The `dict()` Constructor

`dict()` can build dictionaries from keyword arguments, sequences of `(key, value)` pairs, or another mapping.

```python
# Empty dictionary
d1 = dict()

# From keyword arguments (keys must be valid Python identifiers -> strings)
d2 = dict(name="Bob", age=25, active=True)
print(d2)  # {'name': 'Bob', 'age': 25, 'active': True}

# From an iterable of (key, value) pairs
pairs = [("a", 1), ("b", 2), ("c", 3)]
d3 = dict(pairs)
print(d3)  # {'a': 1, 'b': 2, 'c': 3}

# Using zip() to pair two sequences
keys = ["id", "title", "status"]
values = [101, "Fix bug", "open"]
d4 = dict(zip(keys, values))
print(d4)  # {'id': 101, 'title': 'Fix bug', 'status': 'open'}

# Shallow copy from an existing dictionary
d5 = dict(d2)
```

### 1.3 `dict.fromkeys()`

`dict.fromkeys(iterable, value=None)` creates a new dictionary with keys from `iterable`, all initialized to the same `value` (default `None`).

```python
fields = ["name", "email", "phone"]

# Default value is None
user_template = dict.fromkeys(fields)
print(user_template)  # {'name': None, 'email': None, 'phone': None}

# Custom default value
scores = dict.fromkeys(["Alice", "Bob", "Carol"], 0)
print(scores)  # {'Alice': 0, 'Bob': 0, 'Carol': 0}
```

> **Pitfall — Mutable default values with `fromkeys()`:**
> All keys share a reference to the **same** `value` object. Never pass a mutable object like `[]` or `{}` directly.
>
> ```python
> # WRONG: every key points to the exact same list!
> bad = dict.fromkeys(["a", "b"], [])
> bad["a"].append(1)
> print(bad)  # {'a': [1], 'b': [1]}
>
> # RIGHT: use a dictionary comprehension for independent mutable values
> good = {k: [] for k in ["a", "b"]}
> good["a"].append(1)
> print(good)  # {'a': [1], 'b': []}
> ```

### 1.4 Key Hashability Rules

Keys must implement `__hash__()` and `__eq__()`. Built-in immutable types (`int`, `float`, `str`, `bytes`, `bool`, `None`, `frozenset`, and `tuple` containing only hashable elements) are hashable. Mutable types (`list`, `dict`, `set`) are **unhashable** and raise `TypeError`.

```python
valid = {(1, 2): "ok"}       # tuple of ints -> hashable
# invalid = {[1, 2]: "no"}   # TypeError: unhashable type: 'list'
# invalid = {(1, [2]): "no"} # TypeError: unhashable type: 'list'
```

---

## 2. Accessing Values

### 2.1 Square Brackets `d[key]`

Direct indexing retrieves the value if the key exists, or raises `KeyError` if it does not.

```python
config = {"host": "localhost", "port": 8080}

print(config["host"])  # localhost
# print(config["timeout"])  # Raises KeyError: 'timeout'
```

Check existence first with `in` / `not in` (membership tests check **keys** in $O(1)$ average time):

```python
if "port" in config:
    print(f"Port is {config['port']}")

if "debug" not in config:
    print("Debug not configured")
```

### 2.2 `get(key, default=None)`

`get()` safely retrieves a value without raising `KeyError` when the key is missing. It returns `None` (or a specified `default`) instead, without modifying the dictionary.

```python
user = {"username": "jdoe", "role": "viewer"}

print(user.get("username"))             # jdoe
print(user.get("email"))                # None
print(user.get("email", "N/A"))         # N/A
print(user)                             # {'username': 'jdoe', 'role': 'viewer'} (unchanged)
```

> **Note:** If a key exists and its stored value is `None`, `d.get(key, "fallback")` returns `None` (not `"fallback"`), because the key *is* present.

### 2.3 `setdefault(key, default=None)`

`setdefault()` returns the value for `key` if `key` is in the dictionary. If not, it **inserts `key` with `default`** and returns `default`.

```python
settings = {"theme": "dark"}

# Key exists -> returns existing value, does NOT overwrite
theme = settings.setdefault("theme", "light")
print(theme)     # dark
print(settings)  # {'theme': 'dark'}

# Key missing -> inserts key with default and returns it
lang = settings.setdefault("lang", "en")
print(lang)      # en
print(settings)  # {'theme': 'dark', 'lang': 'en'}
```

> **Comparison (`get` vs `setdefault`):**
> - Use `get()` when you only want to **read** with a fallback.
> - Use `setdefault()` when you want to **ensure the key exists** (e.g., initializing a list before appending), though `collections.defaultdict` is often cleaner for repeated grouping.

---

## 3. Adding, Updating, and Deleting Items

### 3.1 Adding and Updating

```python
inventory = {"apples": 5, "bananas": 3}

# Add a new key
inventory["oranges"] = 8

# Update an existing key
inventory["apples"] = 10

# Update multiple entries at once with update()
inventory.update({"bananas": 6, "grapes": 12})
inventory.update(pears=4)  # keyword argument syntax also works

print(inventory)
# {'apples': 10, 'bananas': 6, 'oranges': 8, 'grapes': 12, 'pears': 4}
```

### 3.2 Deleting Items (`del`, `pop`, `popitem`, `clear`)

| Operation | Behavior on Missing Key | Return Value |
| :--- | :--- | :--- |
| `del d[key]` | Raises `KeyError` | None (`del` is a statement) |
| `d.pop(key)` | Raises `KeyError` | Removed value |
| `d.pop(key, default)` | Returns `default` | Removed value or `default` |
| `d.popitem()` | Raises `KeyError` (if dict empty) | `(key, value)` tuple (LIFO) |
| `d.clear()` | N/A | `None` (empties dict in place) |

```python
data = {"a": 1, "b": 2, "c": 3, "d": 4}

# 1. del statement
del data["a"]
print(data)  # {'b': 2, 'c': 3, 'd': 4}

# 2. pop(): remove by key and get its value
val = data.pop("b")
print(val, data)  # 2 {'c': 3, 'd': 4}

# pop() with a default avoids KeyError
missing = data.pop("z", -1)
print(missing)  # -1

# 3. popitem(): removes and returns the LAST inserted (key, value) pair (LIFO since Python 3.7)
last_item = data.popitem()
print(last_item, data)  # ('d', 4) {'c': 3}

# 4. clear(): removes all items in place
data.clear()
print(data)  # {}
```

---

## 4. Dictionary Methods

Complete reference of all built-in `dict` methods:

| Method | Signature | Description |
| :--- | :--- | :--- |
| `keys` | `d.keys()` | Returns a dynamic view of all keys. |
| `values` | `d.values()` | Returns a dynamic view of all values. |
| `items` | `d.items()` | Returns a dynamic view of `(key, value)` tuples. |
| `get` | `d.get(k[, default])` | Returns `d[k]` if `k` exists, else `default` (defaults to `None`). |
| `update` | `d.update([other, ]**kwargs)` | Updates `d` in place from another mapping, iterable of pairs, or kwargs. |
| `pop` | `d.pop(k[, default])` | Removes `k` and returns its value; if absent, returns `default` or raises `KeyError`. |
| `popitem` | `d.popitem()` | Removes and returns the last `(key, value)` pair (LIFO); raises `KeyError` if empty. |
| `clear` | `d.clear()` | Removes all entries in place; returns `None`. |
| `copy` | `d.copy()` | Returns a **shallow** copy of the dictionary. |
| `setdefault` | `d.setdefault(k[, default])` | Returns `d[k]` if present; otherwise sets `d[k] = default` and returns `default`. |
| `fromkeys` | `dict.fromkeys(iterable[, value])` | Class method: creates a new dict with keys from `iterable` set to `value`. |

### Example Showcase

```python
original = {"name": "Alice", "tags": ["py", "sql"]}

# copy() creates a shallow copy
shallow = original.copy()
shallow["name"] = "Bob"          # Does NOT affect original["name"]
shallow["tags"].append("go")     # Mutates shared list -> AFFECTS original["tags"]!

print(original)  # {'name': 'Alice', 'tags': ['py', 'sql', 'go']}
print(shallow)   # {'name': 'Bob', 'tags': ['py', 'sql', 'go']}

# Use copy.deepcopy() for nested mutable structures
import copy
deep = copy.deepcopy(original)
deep["tags"].append("rust")
print(original["tags"])  # ['py', 'sql', 'go']
print(deep["tags"])      # ['py', 'sql', 'go', 'rust']
```

```python
# update() accepts a dict, an iterable of pairs, or keyword arguments
cfg = {"host": "localhost"}
cfg.update([("port", 5432), ("ssl", True)], timeout=30)
print(cfg)  # {'host': 'localhost', 'port': 5432, 'ssl': True, 'timeout': 30}
```

---

## 5. Iterating Over Dictionaries

Dictionaries iterate over **keys** by default.

```python
prices = {"apple": 1.2, "banana": 0.5, "cherry": 2.5}

# 1. Iterating over keys (default)
for fruit in prices:
    print(fruit, prices[fruit])

# Explicit .keys()
for fruit in prices.keys():
    print(fruit)

# 2. Iterating over values
for price in prices.values():
    print(f"${price:.2f}")

# 3. Iterating over (key, value) pairs — most Pythonic when needing both
for fruit, price in prices.items():
    print(f"{fruit}: ${price:.2f}")

# 4. With index via enumerate()
for idx, (fruit, price) in enumerate(prices.items(), start=1):
    print(f"{idx}. {fruit} -> {price}")

# 5. Reverse iteration (Python 3.8+)
for fruit in reversed(prices):
    print(fruit)  # cherry, banana, apple
```

### Safely Modifying While Iterating

Adding or deleting keys while iterating directly over a dictionary raises `RuntimeError: dictionary changed size during iteration`. Iterate over a snapshot (`list(d)` or `list(d.items())`) or build a new dictionary instead.

```python
scores = {"alice": 85, "bob": 42, "carol": 91, "dave": 30}

# WRONG:
# for name, score in scores.items():
#     if score < 50:
#         del scores[name]  # RuntimeError!

# RIGHT (option 1): iterate over a copy of keys
for name in list(scores.keys()):
    if scores[name] < 50:
        del scores[name]

# RIGHT (option 2): rebuild with a comprehension (usually cleaner)
passing = {name: s for name, s in scores.items() if s >= 50}
```

---

## 6. Dictionary Comprehension

Dictionary comprehensions provide a concise way to construct dictionaries from iterables:

```
{key_expr: value_expr for item in iterable if condition}
```

```python
# 1. Basic transformation
squares = {n: n ** 2 for n in range(1, 6)}
print(squares)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# 2. Filtering items
raw_prices = {"laptop": 1200, "mouse": 25, "monitor": 300, "cable": 10}
expensive = {k: v for k, v in raw_prices.items() if v >= 100}
print(expensive)  # {'laptop': 1200, 'monitor': 300}

# 3. Conditional value expression (if/else goes BEFORE 'for')
labels = {k: ("high" if v >= 100 else "low") for k, v in raw_prices.items()}
print(labels)  # {'laptop': 'high', 'mouse': 'low', 'monitor': 'high', 'cable': 'low'}

# 4. Transforming keys and values (e.g., normalizing keys)
headers = {"Content-Type": "application/json", "X-Api-Key": "secret"}
lower_headers = {k.lower(): v for k, v in headers.items()}
print(lower_headers)  # {'content-type': 'application/json', 'x-api-key': 'secret'}

# 5. Inverting a dictionary (assumes values are hashable and unique)
status_codes = {200: "OK", 404: "Not Found", 500: "Server Error"}
code_by_msg = {msg: code for code, msg in status_codes.items()}
print(code_by_msg)  # {'OK': 200, 'Not Found': 404, 'Server Error': 500}
```

---

## 7. Nested Dictionaries

A nested dictionary is a dictionary whose values are themselves dictionaries — ideal for representing JSON-like hierarchical data, records, or matrices.

```python
users = {
    "u1": {
        "name": "Alice",
        "contact": {"email": "alice@example.com", "phone": "555-0100"},
        "roles": ["admin"],
    },
    "u2": {
        "name": "Bob",
        "contact": {"email": "bob@example.com"},
        "roles": ["user"],
    },
}

# Direct nested access (raises KeyError if any level is missing)
print(users["u1"]["contact"]["email"])  # alice@example.com

# Safe nested access by chaining .get() with {}
phone = users.get("u2", {}).get("contact", {}).get("phone", "No phone")
print(phone)  # No phone

# Adding a nested entry
users["u3"] = {"name": "Carol", "contact": {"email": "carol@example.com"}, "roles": []}

# Updating a nested value
users["u2"]["contact"]["phone"] = "555-0200"

# Iterating over nested dictionaries
for user_id, info in users.items():
    email = info.get("contact", {}).get("email", "unknown")
    print(f"{user_id}: {info['name']} ({email})")
```

### Deep Merging or Copying Nested Dictionaries

- `d.copy()` only copies the top-level dictionary; inner dictionaries remain shared references.
- Always use `copy.deepcopy(d)` when you need an independent clone of a nested dictionary.

---

## 8. Merging Dictionaries

When keys collide during a merge, **the right-hand / later dictionary wins**.

### 8.1 `update()` (In-Place, All Python Versions)

```python
defaults = {"host": "0.0.0.0", "port": 8000, "debug": False}
overrides = {"port": 9000, "debug": True}

merged = defaults.copy()
merged.update(overrides)
print(merged)  # {'host': '0.0.0.0', 'port': 9000, 'debug': True}
```

### 8.2 Dictionary Unpacking `**` (Creates a New Dict, Python 3.5+)

```python
a = {"x": 1, "y": 2}
b = {"y": 20, "z": 30}

merged = {**a, **b}
print(merged)  # {'x': 1, 'y': 20, 'z': 30}

# Combine multiple dicts and inline overrides
final = {**a, **b, "extra": 99}
print(final)  # {'x': 1, 'y': 20, 'z': 30, 'extra': 99}
```

### 8.3 Union Operators `|` and `|=` (Python 3.9+)

PEP 584 introduced clean set-like union syntax for dictionaries:
- `d1 | d2` creates a **new** merged dictionary.
- `d1 |= d2` updates `d1` **in place** (and unlike `|`, the right-hand side of `|=` can be any iterable of `(key, value)` pairs, just like `update()`).

```python
defaults = {"theme": "light", "font_size": 12}
user_prefs = {"theme": "dark", "line_numbers": True}

# Non-mutating merge (|)
config = defaults | user_prefs
print(config)  # {'theme': 'dark', 'font_size': 12, 'line_numbers': True}

# In-place merge (|=)
defaults |= user_prefs
print(defaults)  # {'theme': 'dark', 'font_size': 12, 'line_numbers': True}
```

### 8.4 Bonus: `collections.ChainMap` (Zero-Copy Layered View)

If you want to look up keys across multiple dictionaries in priority order without copying data:

```python
from collections import ChainMap

cli_args = {"port": 9090}
env_vars = {"port": 8080, "env": "prod"}
defaults = {"port": 8000, "env": "dev", "debug": False}

config = ChainMap(cli_args, env_vars, defaults)
print(config["port"])   # 9090 (from cli_args)
print(config["env"])    # prod (from env_vars)
print(config["debug"])  # False (from defaults)
```

---

## 9. `OrderedDict`, `defaultdict`, and `Counter` from `collections`

The standard library `collections` module provides specialized `dict` subclasses for common patterns.

### 9.1 `OrderedDict`

Since Python 3.7, standard `dict` already preserves insertion order. However, `OrderedDict` is still useful when you need:
1. **Order-sensitive equality:** `OrderedDict(a=1, b=2) == OrderedDict(b=2, a=1)` is `False`, whereas for plain `dict` it is `True`.
2. **`move_to_end(key, last=True)`:** Efficiently reposition keys (great for implementing LRU caches).
3. **`popitem(last=True)`:** Supports `last=False` to pop the **first** (oldest) item in $O(1)$ time (plain `dict.popitem()` only pops the last item).

```python
from collections import OrderedDict

od = OrderedDict([("a", 1), ("b", 2), ("c", 3)])

# Move 'a' to the end
od.move_to_end("a")
print(list(od.keys()))  # ['b', 'c', 'a']

# Move 'c' to the front
od.move_to_end("c", last=False)
print(list(od.keys()))  # ['c', 'b', 'a']

# Pop from the front (FIFO)
print(od.popitem(last=False))  # ('c', 3)

# Order-sensitive equality
print(OrderedDict(a=1, b=2) == OrderedDict(b=2, a=1))  # False
print({"a": 1, "b": 2} == {"b": 2, "a": 1})            # True
```

### 9.2 `defaultdict`

`defaultdict(default_factory)` automatically creates a default value by calling `default_factory()` whenever a missing key is accessed.

```python
from collections import defaultdict

# Grouping items into lists
words = ["apple", "ant", "bat", "ball", "cat"]
by_letter = defaultdict(list)
for w in words:
    by_letter[w[0]].append(w)

print(dict(by_letter))
# {'a': ['apple', 'ant'], 'b': ['bat', 'ball'], 'c': ['cat']}

# Counting with int (int() returns 0)
counts = defaultdict(int)
for char in "banana":
    counts[char] += 1
print(dict(counts))  # {'b': 1, 'a': 3, 'n': 2}

# Grouping unique items with set
edges = [("A", "B"), ("A", "C"), ("B", "C"), ("A", "B")]
graph = defaultdict(set)
for src, dst in edges:
    graph[src].add(dst)
print(dict(graph))  # {'A': {'B', 'C'}, 'B': {'C'}}

# Custom factory with lambda
scores = defaultdict(lambda: 100)
print(scores["new_player"])  # 100
```

> **Caution:** Simply reading `dd[missing_key]` inserts `missing_key` with the default value. Use `dd.get(missing_key)` if you want to check without inserting.

### 9.3 `Counter`

`Counter` is a `dict` subclass specifically designed for counting hashable objects. Missing keys return `0` without inserting them.

```python
from collections import Counter

# Counting from an iterable
c = Counter("abracadabra")
print(c)  # Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})

# Missing keys return 0 (and are NOT added to the Counter)
print(c["z"])  # 0

# Most common elements
print(c.most_common(2))  # [('a', 5), ('b', 2)]

# Counting words
words = ["red", "blue", "red", "green", "blue", "red"]
word_counts = Counter(words)
print(word_counts.total())  # 6 (Python 3.10+)

# Counter arithmetic
c1 = Counter(a=3, b=1)
c2 = Counter(a=1, b=2)
print(c1 + c2)  # Counter({'a': 4, 'b': 3})  — addition
print(c1 - c2)  # Counter({'a': 2})         — subtraction (drops <= 0)
print(c1 & c2)  # Counter({'a': 1, 'b': 1}) — intersection (min)
print(c1 | c2)  # Counter({'a': 3, 'b': 2}) — union (max)

# Expand back into elements
print(list(Counter(x=2, y=3).elements()))  # ['x', 'x', 'y', 'y', 'y']
```

---

## 10. Dictionary vs Other Data Structures

| Feature | `dict` | `list` | `tuple` | `set` |
| :--- | :--- | :--- | :--- | :--- |
| **Syntax** | `{"a": 1}` | `[1, 2]` | `(1, 2)` | `{1, 2}` |
| **Stores** | Key–value pairs | Ordered items | Ordered items | Unique hashable items |
| **Mutable?** | Yes | Yes | No | Yes (`frozenset` is immutable) |
| **Ordered?** | Yes (insertion order) | Yes (index order) | Yes (index order) | No |
| **Allows duplicates?** | Keys: No; Values: Yes | Yes | Yes | No |
| **Lookup / `in` complexity** | $O(1)$ average | $O(n)$ | $O(n)$ | $O(1)$ average |
| **Access by** | Hashable key | Integer index / slice | Integer index / slice | Membership check / iteration |
| **Primary use case** | Associative mapping, fast key lookup, JSON-like records | Ordered sequences, stacks, dynamic arrays | Fixed records, heterogeneous data, dict keys | Uniqueness, fast membership, set math |

### When to Choose Which

- **Use `dict`** when you need fast $O(1)$ lookups by meaningful keys (IDs, names, coordinates) or structured key–value records.
- **Use `list`** when element order/position matters, duplicates are allowed, and you iterate sequentially or index by position.
- **Use `tuple`** for fixed-size heterogeneous records (like `(lat, lon)`) or when you need an immutable sequence to use as a `dict` key.
- **Use `set`** when you only care about unique membership (no associated values) and set operations (`|`, `&`, `-`).

---

## 11. Dictionary Unpacking

### 11.1 Function Arguments (`**kwargs`)

The `**` operator unpacks a dictionary into keyword arguments when calling a function, and collects extra keyword arguments into a dictionary inside a function definition.

```python
def create_user(name, role="member", active=True):
    return f"{name} ({role}), active={active}"

payload = {"name": "Alice", "role": "admin"}
print(create_user(**payload))  # Alice (admin), active=True

# Accepting arbitrary keyword arguments
def log_event(event_type, **metadata):
    print(f"[{event_type}]", metadata)

log_event("login", user="alice", ip="10.0.0.1")
# [login] {'user': 'alice', 'ip': '10.0.0.1'}
```

### 11.2 Unpacking Keys with `*` vs Key–Value Pairs with `**`

```python
d = {"a": 1, "b": 2, "c": 3}

# Single * unpacks keys into a list/tuple/function positional args
keys_list = [*d]
print(keys_list)  # ['a', 'b', 'c']

first_key, *other_keys = d
print(first_key, other_keys)  # a ['b', 'c']

# Double ** unpacks key-value pairs into another dictionary literal
extended = {**d, "d": 4}
print(extended)  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}
```

### 11.3 Structural Pattern Matching (`match`/`case`, Python 3.10+)

Dictionaries can also be destructured with `match`/`case` (matches partial keys by default):

```python
def handle_response(resp):
    match resp:
        case {"status": 200, "data": data}:
            return f"Success: {data}"
        case {"status": code, "error": msg}:
            return f"Error {code}: {msg}"
        case _:
            return "Unknown response"

print(handle_response({"status": 200, "data": [1, 2], "meta": "ignored"}))
# Success: [1, 2]
```

---

## 12. Common Dictionary Patterns and Best Practices

### 12.1 Dispatch Tables (Replacing Long `if/elif` Chains)

```python
def add(a, b): return a + b
def sub(a, b): return a - b
def mul(a, b): return a * b

operations = {
    "+": add,
    "-": sub,
    "*": mul,
}

op = "*"
func = operations.get(op)
result = func(6, 7) if func else None
print(result)  # 42
```

### 12.2 Sorting a Dictionary

Dictionaries keep insertion order, so sorting produces a new dictionary in the desired order using `sorted()`:

```python
scores = {"Charlie": 88, "Alice": 95, "Bob": 72}

# Sort by key (alphabetically)
by_name = dict(sorted(scores.items()))
print(by_name)  # {'Alice': 95, 'Bob': 72, 'Charlie': 88}

# Sort by value (descending)
by_score_desc = dict(sorted(scores.items(), key=lambda item: item[1], reverse=True))
print(by_score_desc)  # {'Alice': 95, 'Charlie': 88, 'Bob': 72}
```

### 12.3 Finding Min / Max Key by Value

```python
prices = {"apple": 1.2, "banana": 0.5, "cherry": 2.5}

cheapest_item = min(prices, key=prices.get)
priciest_pair = max(prices.items(), key=lambda item: item[1])

print(cheapest_item)  # banana
print(priciest_pair)  # ('cherry', 2.5)
```

### 12.4 Grouping with `setdefault` vs `defaultdict`

```python
records = [("eng", "Alice"), ("hr", "Bob"), ("eng", "Carol")]

# Using setdefault (no import needed)
grouped = {}
for dept, name in records:
    grouped.setdefault(dept, []).append(name)

# Using defaultdict (cleaner and slightly faster for large loops)
from collections import defaultdict
grouped_dd = defaultdict(list)
for dept, name in records:
    grouped_dd[dept].append(name)
```

### 12.5 Inverting a Dictionary with Non-Unique Values

```python
roles = {"Alice": "admin", "Bob": "editor", "Carol": "admin"}

by_role = {}
for user, role in roles.items():
    by_role.setdefault(role, []).append(user)

print(by_role)  # {'admin': ['Alice', 'Carol'], 'editor': ['Bob']}
```

### 12.6 Best Practices Checklist

- **Prefer `in` over `d.keys()`:** Write `if k in d:` instead of `if k in d.keys():` — it is cleaner and avoids creating an extra view object.
- **Prefer `.get()` or `try/except KeyError`:** Use `d.get(k, default)` when missing keys are common; use `try/except KeyError` (EAFP: *Easier to Ask for Forgiveness than Permission*) when missing keys are rare and you want maximum speed.
- **Use `.items()` when needing both key and value:** Avoid `for k in d: ... d[k]`.
- **Beware mutable defaults:** Never use `dict.fromkeys(keys, [])` or mutable default arguments `def f(d={}):` (use `def f(d=None):`).
- **Consider `dataclass` or `NamedTuple` for fixed schemas:** If every dict has the exact same fixed set of fields (e.g., `x`, `y`, `z`), a `@dataclass` provides type hints, attribute access, and IDE autocomplete.

---

## 13. Dictionary View Objects

Calling `d.keys()`, `d.values()`, or `d.items()` returns **dictionary view objects** (`dict_keys`, `dict_values`, `dict_items`), **not** lists.

### 13.1 Dynamic Nature (Live Window)

Views reflect changes made to the underlying dictionary immediately without copying data ($O(1)$ creation).

```python
stock = {"apples": 10, "bananas": 5}
keys_view = stock.keys()
items_view = stock.items()

print(keys_view)  # dict_keys(['apples', 'bananas'])

# Modify the dictionary
stock["oranges"] = 8
del stock["apples"]

# The existing views automatically reflect the changes!
print(keys_view)   # dict_keys(['bananas', 'oranges'])
print(items_view)  # dict_items([('bananas', 5), ('oranges', 8)])
```

### 13.2 Set-Like Operations on `dict_keys` and `dict_items`

Because dictionary keys are unique and hashable, `dict_keys` behaves like a `set` (supporting `&` intersection, `|` union, `-` difference, `^` symmetric difference, `<=`, `<`, `>=`, `>`, and `isdisjoint()`).

`dict_items` also supports set operations **provided all values in the dictionary are hashable** (otherwise `TypeError` is raised). `dict_values` does **not** support set operations because values may be unhashable or duplicated.

```python
a = {"x": 1, "y": 2, "z": 3}
b = {"y": 2, "z": 99, "w": 4}

# Keys present in both a and b
print(a.keys() & b.keys())   # {'y', 'z'}

# Keys in a but not b
print(a.keys() - b.keys())   # {'x'}

# All keys across both
print(a.keys() | b.keys())   # {'x', 'y', 'z', 'w'}

# Keys in exactly one dict
print(a.keys() ^ b.keys())   # {'x', 'w'}

# Exact (key, value) pairs shared by both (requires hashable values)
print(a.items() & b.items()) # {('y', 2)}
```

### 13.3 Useful Properties of Views

- **Length and membership:** `len(view)` and `x in view` run in $O(1)$ for `keys` and `items` (and $O(n)$ for `values`).
- **Reversible (Python 3.8+):** `reversed(d.keys())`, `reversed(d.values())`, `reversed(d.items())`.
- **Back-reference via `.mapping` (Python 3.10+):** `view.mapping` returns a read-only `types.MappingProxyType` wrapping the original dictionary.
- **Convert to `list` only when needed:** Views are not subscriptable (`d.keys()[0]` raises `TypeError`). Use `list(d.keys())[0]` or `next(iter(d))` if you need indexing or a static snapshot.

```python
d = {"first": 1, "second": 2}

# Accessing the first key without building a full list
first_key = next(iter(d.keys()))
print(first_key)  # first

# Static snapshot
snapshot = list(d.items())
d["third"] = 3
print(snapshot)  # [('first', 1), ('second', 2)]
```
