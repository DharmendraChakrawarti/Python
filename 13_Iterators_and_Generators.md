# Iterators and Generators in Python

Iterators and generators form the backbone of Python's iteration system. They enable clean traversal of data structures, lazy evaluation (computing values only when needed), and memory-efficient processing of massive or even infinite data streams.

---

## Table of Contents

1. [Iteration Protocol (`__iter__`, `__next__`)](#1-iteration-protocol-__iter__-__next__)
2. [Iterable vs Iterator](#2-iterable-vs-iterator)
3. [Creating Custom Iterators](#3-creating-custom-iterators)
4. [`iter()` and `next()` Built-in Functions](#4-iter-and-next-built-in-functions)
5. [`StopIteration` Exception](#5-stopiteration-exception)
6. [Generator Functions (`yield` Keyword)](#6-generator-functions-yield-keyword)
7. [Generator Expressions](#7-generator-expressions)
8. [`yield` vs `return`](#8-yield-vs-return)
9. [`send()`, `throw()`, and `close()` on Generators](#9-send-throw-and-close-on-generators)
10. [`yield from` (Delegating Generators)](#10-yield-from-delegating-generators)
11. [Generator Pipelines](#11-generator-pipelines)
12. [Infinite Generators](#12-infinite-generators)
13. [The `itertools` Module](#13-the-itertools-module)
14. [Memory Efficiency: Generators vs Lists](#14-memory-efficiency-generators-vs-lists)
15. [Use Cases for Generators](#15-use-cases-for-generators)

---

## 1. Iteration Protocol (`__iter__`, `__next__`)

The **iteration protocol** is the standard interface Python uses to step through elements one at a time (in `for` loops, comprehensions, unpacking, `sum()`, `list()`, etc.).

It consists of two dunder (magic) methods:

- **`__iter__(self)`**: Returns an iterator object. If the object itself is an iterator, it returns `self`.
- **`__next__(self)`**: Returns the next item in the sequence. When no items remain, it raises `StopIteration`.

### How a `for` Loop Actually Works Under the Hood

When you write:

```python
numbers = [10, 20, 30]
for num in numbers:
    print(num)
```

Python internally executes something equivalent to:

```python
numbers = [10, 20, 30]
iterator = iter(numbers)  # Calls numbers.__iter__()

while True:
    try:
        num = next(iterator)  # Calls iterator.__next__()
        print(num)
    except StopIteration:
        break
```

> **Note:** Python also supports an older *sequence protocol* fallback (`__getitem__` starting at index `0` until `IndexError`), which `iter()` uses if `__iter__` is not defined. Modern Python code should implement `__iter__` explicitly.

---

## 2. Iterable vs Iterator

Understanding the difference between an **iterable** and an **iterator** prevents common bugs such as accidentally exhausting a single-use iterator.

| Feature | Iterable | Iterator |
| :--- | :--- | :--- |
| **Definition** | An object capable of returning its members one at a time | An object representing a stream of data with internal state |
| **Required Method(s)** | `__iter__()` (or `__getitem__()`) | `__iter__()` **and** `__next__()` |
| **`iter(obj)` returns** | A fresh iterator object | `self` (the same iterator object) |
| **Reusable?** | Usually yes (each `iter()` creates a new iterator) | **No** — once exhausted, it stays exhausted |
| **Examples** | `list`, `tuple`, `str`, `dict`, `set`, `range` | `iter([1, 2])`, file objects, generators, `map`, `zip` |
| **ABC check** | `collections.abc.Iterable` | `collections.abc.Iterator` |

> Every **iterator** is also an **iterable** (because `Iterator` inherits from `Iterable` and implements `__iter__` returning `self`), but not every **iterable** is an **iterator**.

```python
from collections.abc import Iterable, Iterator

nums = [1, 2, 3]
nums_iter = iter(nums)

print(isinstance(nums, Iterable))       # True
print(isinstance(nums, Iterator))       # False

print(isinstance(nums_iter, Iterable))  # True
print(isinstance(nums_iter, Iterator))  # True

# Iterators return themselves from iter()
print(iter(nums_iter) is nums_iter)     # True

# Lists create a new iterator each time
it1 = iter(nums)
it2 = iter(nums)
print(it1 is it2)                       # False
print(next(it1), next(it1))             # 1 2
print(next(it2))                        # 1 (independent state)
```

### Common Pitfall: Exhausting an Iterator

```python
squares = (x * x for x in range(4))  # Generator (which is an iterator)
print(list(squares))  # [0, 1, 4, 9]
print(list(squares))  # []  <- Already exhausted!
```

---

## 3. Creating Custom Iterators

To build a custom iterator class, implement `__iter__()` and `__next__()`.

### Example 1: Countdown Iterator (Single-Pass Iterator)

```python
class Countdown:
    """Iterator that counts down from start to 1."""

    def __init__(self, start: int):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self) -> int:
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value


for n in Countdown(5):
    print(n, end=" ")  # Output: 5 4 3 2 1
```

### Example 2: Separating Iterable and Iterator (Reusable Collection)

If an object returns `self` from `__iter__()`, it can only be iterated once. To make a custom container **reusable** across multiple loops (like `range` or `list`), separate the **iterable** from its **iterator**:

```python
class NumberRange:
    """Reusable iterable (similar to range)."""

    def __init__(self, start: int, stop: int):
        self.start = start
        self.stop = stop

    def __iter__(self):
        # Return a fresh iterator instance every time
        return _NumberRangeIterator(self.start, self.stop)


class _NumberRangeIterator:
    """Iterator holding traversal state for NumberRange."""

    def __init__(self, start: int, stop: int):
        self.current = start
        self.stop = stop

    def __iter__(self):
        return self

    def __next__(self) -> int:
        if self.current >= self.stop:
            raise StopIteration
        value = self.current
        self.current += 1
        return value


my_range = NumberRange(1, 4)
print(list(my_range))  # [1, 2, 3]
print(list(my_range))  # [1, 2, 3]  (works again!)
```

---

## 4. `iter()` and `next()` Built-in Functions

Python provides built-in functions `iter()` and `next()` to drive the iteration protocol cleanly instead of calling `__iter__()` and `__next__()` directly.

### `iter()` — Two Forms

1. **`iter(iterable)`**: Calls `iterable.__iter__()` (or falls back to `__getitem__`). Raises `TypeError` if the object is not iterable.
2. **`iter(callable, sentinel)`**: Calls `callable()` repeatedly with zero arguments until it returns a value equal (`==`) to `sentinel`, at which point it raises `StopIteration`.

```python
# Form 1: Standard iterable
it = iter("ABC")
print(next(it))  # 'A'

# Form 2: Sentinel-based iteration (very useful for I/O / reading chunks)
import io

stream = io.StringIO("HelloWorldPython")
# Read 5 characters at a time until "" (EOF sentinel) is returned
chunk_iter = iter(lambda: stream.read(5), "")
print(list(chunk_iter))  # ['Hello', 'World', 'Pytho', 'n']
```

### `next()` — Two Forms

1. **`next(iterator)`**: Calls `iterator.__next__()`. Raises `StopIteration` when exhausted.
2. **`next(iterator, default)`**: Returns `default` instead of raising `StopIteration` when the iterator is exhausted.

```python
items = iter(["apple", "banana"])

print(next(items))               # 'apple'
print(next(items))               # 'banana'
print(next(items, "no more"))    # 'no more' (no StopIteration raised)
print(next(items, None))         # None
```

Using `next(..., default)` is great for finding the first matching item without creating a whole list:

```python
users = [
    {"id": 1, "role": "user"},
    {"id": 2, "role": "admin"},
    {"id": 3, "role": "admin"},
]

first_admin = next((u for u in users if u["role"] == "admin"), None)
print(first_admin)  # {'id': 2, 'role': 'admin'}
```

---

## 5. `StopIteration` Exception

`StopIteration` is a built-in exception raised by an iterator's `__next__()` method to signal that there are no more items to produce.

### Key Rules About `StopIteration`

1. **Exhausted iterators must keep raising `StopIteration`**: Once `__next__()` raises `StopIteration`, all subsequent calls to `__next__()` must also raise `StopIteration`.
2. **`for` loops and comprehensions catch it automatically**: You rarely catch `StopIteration` in normal application code unless manually calling `next()`.
3. **`StopIteration.value`**: When a generator executes `return some_value`, Python raises `StopIteration(some_value)`. The return value is stored on the exception's `.value` attribute.
4. **PEP 479 (Python 3.7+)**: Raising `StopIteration` manually inside a generator function is converted into a `RuntimeError` so accidental `next()` failures inside generators aren't silently swallowed. Use `return` to end a generator instead.

```python
def gen():
    yield 1
    return "finished"

g = gen()
print(next(g))  # 1

try:
    next(g)
except StopIteration as e:
    print(f"Generator stopped with value: {e.value!r}")
# Output: Generator stopped with value: 'finished'
```

```python
# PEP 479 example: NEVER raise StopIteration directly inside a generator
def bad_gen():
    yield 1
    raise StopIteration("oops")  # Converted to RuntimeError!

g = bad_gen()
print(next(g))  # 1
# next(g) -> RuntimeError: generator raised StopIteration
```

---

## 6. Generator Functions (`yield` Keyword)

A **generator function** is any function containing the `yield` keyword. Calling it does **not** execute the function body immediately—instead, it returns a **generator object**, which automatically implements the iterator protocol (`__iter__` and `__next__`).

Each time `next()` is called on the generator:
1. Execution resumes right after the last `yield`.
2. Local variables and instruction pointer are preserved between calls.
3. When `yield value` is reached, execution pauses and `value` is returned to the caller.
4. When the function reaches the end or hits `return`, `StopIteration` is raised.

```python
def fibonacci(limit: int):
    """Yield Fibonacci numbers up to `limit` items."""
    a, b = 0, 1
    count = 0
    while count < limit:
        yield a
        a, b = b, a + b
        count += 1


fib_gen = fibonacci(6)
print(type(fib_gen))     # <class 'generator'>
print(list(fib_gen))     # [0, 1, 1, 2, 3, 5]
```

### Visualizing Execution Flow

```python
def trace_gen():
    print("Step 1: Start")
    yield "A"
    print("Step 2: Middle")
    yield "B"
    print("Step 3: End")


g = trace_gen()      # Nothing printed yet!
print("Created:", g)

print("Got:", next(g))  # Prints "Step 1: Start", then "Got: A"
print("Got:", next(g))  # Prints "Step 2: Middle", then "Got: B"
# next(g)               # Prints "Step 3: End", then raises StopIteration
```

---

## 7. Generator Expressions

A **generator expression** is a concise, inline way to create a generator—similar to a list comprehension, but using parentheses `()` instead of square brackets `[]`.

### Syntax

```python
(expression for item in iterable if condition)
```

### Examples

```python
# List comprehension: builds entire list in memory immediately
squares_list = [x ** 2 for x in range(5)]

# Generator expression: creates a lazy generator object
squares_gen = (x ** 2 for x in range(5))

print(squares_list)       # [0, 1, 4, 9, 16]
print(squares_gen)        # <generator object <genexpr> at ...>
print(next(squares_gen))  # 0
print(next(squares_gen))  # 1
print(list(squares_gen))  # [4, 9, 16]  (remaining elements)
```

### Omitting Extra Parentheses

When a generator expression is the **only** argument to a function call, Python allows omitting the outer parentheses:

```python
total = sum(x ** 2 for x in range(1, 101))
has_negative = any(n < 0 for n in [3, 5, -1, 8])
csv_line = ", ".join(str(x) for x in [10, 20, 30])

print(total, has_negative, csv_line)  # 338350 True 10, 20, 30
```

> **Note:** Generator expressions are great for simple transformations/filtering. For multi-step logic, error handling, or readability, prefer a `def` generator function.

---

## 8. `yield` vs `return`

| Aspect | `yield` | `return` |
| :--- | :--- | :--- |
| **Function type** | Turns the function into a **generator function** | Standard function (or terminates a generator) |
| **Execution** | **Pauses** execution and saves local state | **Terminates** execution and destroys local stack frame |
| **Number of values** | Can produce **many values** one by one over time | Returns **once** per call |
| **Return type on call** | Calling `fn()` returns a `generator` object | Calling `fn()` returns the actual result |
| **Inside a generator** | Produces the next yielded value | Ends the generator by raising `StopIteration(value)` |
| **Memory** | Lazy (computes one item at a time) | Eager (must build full collection if returning many items) |

```python
# Using return: builds and returns everything at once
def get_even_list(n: int) -> list[int]:
    result = []
    for i in range(n):
        if i % 2 == 0:
            result.append(i)
    return result

# Using yield: produces one value at a time on demand
def get_even_gen(n: int):
    for i in range(n):
        if i % 2 == 0:
            yield i

print(get_even_list(6))        # [0, 2, 4]
print(get_even_gen(6))         # <generator object get_even_gen at ...>
print(list(get_even_gen(6)))   # [0, 2, 4]
```

---

## 9. `send()`, `throw()`, and `close()` on Generators

In addition to producing values, generators can **receive** values, exceptions, and termination signals via `send()`, `throw()`, and `close()`. This makes generators usable as simple coroutines.

### 1. `generator.send(value)`

Resumes the generator and **sends `value` in as the result of the current `yield` expression**, then runs until the next `yield`.

> **Priming requirement:** A newly created generator is paused at the start of its body (not at a `yield` yet). You must **prime** it first by calling `next(gen)` or `gen.send(None)` before sending any non-`None` value.

```python
def running_average():
    total = 0.0
    count = 0
    avg = None
    while True:
        value = yield avg  # Yields current avg, receives next value
        total += value
        count += 1
        avg = total / count


avg_coro = running_average()
next(avg_coro)               # Prime the generator (advances to first yield)

print(avg_coro.send(10))     # 10.0
print(avg_coro.send(20))     # 15.0
print(avg_coro.send(30))     # 20.0
```

### 2. `generator.throw(exc_type[, exc_value[, traceback]])`

Raises an exception inside the generator at the exact `yield` where it is currently paused.
- If the generator catches the exception and yields another value, `throw()` returns that yielded value.
- If the generator finishes without yielding, `StopIteration` is raised.
- If the generator does not catch it, the exception propagates back to the caller.

```python
def resilient_worker():
    try:
        while True:
            try:
                task = yield "ready"
                print(f"Processing: {task}")
            except ValueError as e:
                print(f"Handled ValueError inside generator: {e}")
    except RuntimeError:
        print("Fatal RuntimeError caught, shutting down worker.")


w = resilient_worker()
print(next(w))                           # 'ready'
print(w.send("job-1"))                   # Processing: job-1 -> 'ready'
print(w.throw(ValueError, "bad input"))  # Handled ValueError... -> 'ready'
# w.throw(RuntimeError)                  # Prints shutdown msg, raises StopIteration
```

### 3. `generator.close()`

Raises `GeneratorExit` at the suspended `yield` inside the generator so it can run `finally` cleanup blocks and exit.
- Calling `close()` on an already closed/exhausted generator is a safe no-op.
- Inside `except GeneratorExit:` or `finally:`, the generator **must not `yield` another value** (doing so raises `RuntimeError: generator ignored GeneratorExit`).

```python
def resource_stream():
    print("Opening resource...")
    try:
        i = 0
        while True:
            yield i
            i += 1
    except GeneratorExit:
        print("GeneratorExit received!")
        raise  # Or just let it exit cleanly; do NOT yield here
    finally:
        print("Closing resource cleanly.")


s = resource_stream()
print(next(s))  # Opening resource... \n 0
print(next(s))  # 1
s.close()       # GeneratorExit received! \n Closing resource cleanly.
```

---

## 10. `yield from` (Delegating Generators)

Introduced in PEP 380 (Python 3.3), `yield from iterable` delegates part of a generator's operations to another iterable or subgenerator.

### Basic Usage: Flattening / Chaining

Instead of writing:

```python
def chain_manual(*iterables):
    for it in iterables:
        for item in it:
            yield item
```

You can write:

```python
def chain_clean(*iterables):
    for it in iterables:
        yield from it


print(list(chain_clean([1, 2], ("a", "b"), range(3))))
# [1, 2, 'a', 'b', 0, 1, 2]
```

### Recursive Generators with `yield from`

`yield from` makes recursive traversal (like flattening nested lists or walking trees) concise:

```python
def flatten(nested):
    for item in nested:
        if isinstance(item, (list, tuple)):
            yield from flatten(item)
        else:
            yield item


data = [1, [2, [3, 4], 5], (6, 7)]
print(list(flatten(data)))  # [1, 2, 3, 4, 5, 6, 7]
```

### Full Delegation: Forwarding `send()`, `throw()`, `close()`, and `return`

`yield from` does much more than a `for` loop—it creates a **two-way channel** between the caller and the subgenerator:
- Automatically forwards `.send()`, `.throw()`, and `.close()` to the subgenerator.
- Evaluates to the subgenerator's **`return` value** (captured from `StopIteration.value`).

```python
def subgenerator():
    total = 0
    count = 0
    while True:
        x = yield
        if x is None:
            break
        total += x
        count += 1
    return (count, total)  # Returned as value of `yield from`


def delegator():
    result = yield from subgenerator()
    print(f"Subgenerator finished: count={result[0]}, total={result[1]}")
    yield result


d = delegator()
next(d)          # Primes delegator AND subgenerator automatically
d.send(10)
d.send(25)
summary = d.send(None)  # Ends subgenerator, prints message, yields result
print("Caller got:", summary)
# Output:
# Subgenerator finished: count=2, total=35
# Caller got: (2, 35)
```

---

## 11. Generator Pipelines

A **generator pipeline** connects multiple generators in series, where each stage consumes items from the previous stage, transforms or filters them, and yields results downstream.

Data flows **one item at a time** through the entire pipeline—no intermediate lists are stored in memory.

### Example: Log Processing Pipeline

```python
logs = [
    "2026-01-10 INFO  User login id=42",
    "2026-01-10 ERROR Disk 95% full",
    "2026-01-10 DEBUG Cache hit key=abc",
    "2026-01-10 ERROR Timeout reaching db-01",
    "2026-01-10 INFO  User logout id=42",
]


def parse_lines(lines):
    """Stage 1: Parse raw log strings into dictionaries."""
    for line in lines:
        parts = line.split(maxsplit=2)
        yield {"date": parts[0], "level": parts[1], "message": parts[2]}


def filter_level(records, level: str):
    """Stage 2: Keep only records matching a severity level."""
    for rec in records:
        if rec["level"] == level:
            yield rec


def format_alerts(records):
    """Stage 3: Format matching records as alert strings."""
    for rec in records:
        yield f"[{rec['date']}] ALERT: {rec['message']}"


# Compose the pipeline (nothing runs yet!)
pipeline = format_alerts(filter_level(parse_lines(logs), "ERROR"))

# Pull data through the pipeline
for alert in pipeline:
    print(alert)

# Output:
# [2026-01-10] ALERT: Disk 95% full
# [2026-01-10] ALERT: Timeout reaching db-01
```

Because execution is pull-driven, if you wrap `pipeline` in `itertools.islice(pipeline, 1)`, Python stops immediately after producing the first alert without even parsing the remaining lines.

---

## 12. Infinite Generators

Because generators evaluate lazily, they can represent **unbounded (infinite) sequences** in constant $O(1)$ memory. Always pair them with a stopping mechanism (`break`, `islice()`, `takewhile()`, `zip()`, etc.) so consumers don't loop forever.

### Example 1: Infinite Fibonacci Sequence

```python
def infinite_fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b


fib = infinite_fibonacci()
first_eight = [next(fib) for _ in range(8)]
print(first_eight)  # [0, 1, 1, 2, 3, 5, 8, 13]
```

### Example 2: Infinite Unique ID Generator

```python
def id_generator(prefix: str = "ID", start: int = 1):
    num = start
    while True:
        yield f"{prefix}-{num:04d}"
        num += 1


ids = id_generator("USR", 100)
print(next(ids))  # 'USR-0100'
print(next(ids))  # 'USR-0101'
print(next(ids))  # 'USR-0102'
```

### Example 3: Infinite Prime Number Generator

```python
def primes():
    """Yield an endless stream of prime numbers."""
    yield 2
    candidate = 3
    found = [2]
    while True:
        limit = int(candidate ** 0.5)
        if all(candidate % p != 0 for p in found if p <= limit):
            found.append(candidate)
            yield candidate
        candidate += 2


from itertools import islice
print(list(islice(primes(), 10)))
# [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

---

## 13. The `itertools` Module

Python's built-in `itertools` module provides fast, C-implemented, memory-efficient iterator building blocks.

### A. Infinite Iterators

#### 1. `count(start=0, step=1)`
Counts evenly spaced values indefinitely (supports floats too).

```python
from itertools import count, islice

print(list(islice(count(10, 2.5), 5)))
# [10, 12.5, 15.0, 17.5, 20.0]

# Useful with zip() for custom indexing
for idx, name in zip(count(1), ["Alice", "Bob", "Charlie"]):
    print(f"{idx}. {name}")
# 1. Alice
# 2. Bob
# 3. Charlie
```

#### 2. `cycle(iterable)`
Saves a copy of `iterable` and repeats its elements endlessly.

```python
from itertools import cycle, islice

turns = cycle(["Player 1", "Player 2"])
print(list(islice(turns, 5)))
# ['Player 1', 'Player 2', 'Player 1', 'Player 2', 'Player 1']
```

#### 3. `repeat(object[, times])`
Yields `object` repeatedly—either endlessly or `times` times.

```python
from itertools import repeat

print(list(repeat("ping", 3)))  # ['ping', 'ping', 'ping']

# Handy for passing a constant argument to map()
powers = list(map(pow, [2, 3, 4], repeat(3)))
print(powers)  # [8, 27, 64]
```

---

### B. Iterators Terminating on the Shortest / Condition / Slice

#### 4. `chain(*iterables)` and `chain.from_iterable(iterable)`
Concatenates multiple iterables into one continuous iterator without copying elements into a list.

```python
from itertools import chain

combined = chain([1, 2], (3, 4), "AB")
print(list(combined))  # [1, 2, 3, 4, 'A', 'B']

# Lazy flattening of one level using chain.from_iterable
nested = [[1, 2], [3, 4], [5]]
print(list(chain.from_iterable(nested)))  # [1, 2, 3, 4, 5]
```

#### 5. `islice(iterable, stop)` / `islice(iterable, start, stop[, step])`
Slices any iterator lazily (negative indices are not supported).

```python
from itertools import islice

gen = (x * 10 for x in range(100))
print(list(islice(gen, 2, 10, 2)))  # [20, 40, 60, 80]
```

#### 6. `zip_longest(*iterables, fillvalue=None)`
Like built-in `zip()`, but continues until the **longest** iterable is exhausted, padding shorter ones with `fillvalue`.

```python
from itertools import zip_longest

names = ["Alice", "Bob", "Charlie"]
scores = [95, 88]

print(list(zip_longest(names, scores, fillvalue=0)))
# [('Alice', 95), ('Bob', 88), ('Charlie', 0)]
```

#### 7. `starmap(function, iterable)`
Like `map()`, except each item in `iterable` is unpacked as positional arguments (`*args`) into `function`.

```python
from itertools import starmap

pairs = [(2, 5), (3, 2), (10, 3)]
print(list(starmap(pow, pairs)))  # [32, 9, 1000]
```

#### 8. `takewhile(predicate, iterable)`
Yields elements **while** `predicate(item)` is true, then stops permanently on the first false element.

```python
from itertools import takewhile

nums = [2, 4, 6, 7, 8, 10]
print(list(takewhile(lambda x: x % 2 == 0, nums)))
# [2, 4, 6]  (stops at 7; does NOT yield 8 or 10)
```

#### 9. `dropwhile(predicate, iterable)`
Drops elements **while** `predicate(item)` is true, then yields the first false element and **all remaining elements** unconditionally.

```python
from itertools import dropwhile

nums = [2, 4, 6, 7, 8, 1]
print(list(dropwhile(lambda x: x % 2 == 0, nums)))
# [7, 8, 1]  (starts yielding at 7 and keeps 8, 1)
```

#### 10. `groupby(iterable, key=None)`
Groups **consecutive** elements that share the same key.

> **Important:** `groupby()` only groups *adjacent* matching items (like Unix `uniq`). Sort the data by the same `key` first if identical keys are scattered! Also, consume or convert each group before advancing to the next group.

```python
from itertools import groupby

# Example 1: Consecutive grouping
letters = "AAABBBCCA"
for char, group in groupby(letters):
    print(char, list(group))
# A ['A', 'A', 'A']
# B ['B', 'B', 'B']
# C ['C', 'C']
# A ['A']

# Example 2: Sorting before grouping by a key
people = [
    {"name": "Alice", "dept": "Eng"},
    {"name": "Bob", "dept": "Sales"},
    {"name": "Carol", "dept": "Eng"},
]
people_sorted = sorted(people, key=lambda p: p["dept"])
for dept, members in groupby(people_sorted, key=lambda p: p["dept"]):
    print(dept, [m["name"] for m in members])
# Eng ['Alice', 'Carol']
# Sales ['Bob']
```

---

### C. Combinatoric Iterators

#### 11. `product(*iterables, repeat=1)`
Cartesian product—equivalent to nested `for` loops.

```python
from itertools import product

print(list(product(["S", "M"], ["Red", "Blue"])))
# [('S', 'Red'), ('S', 'Blue'), ('M', 'Red'), ('M', 'Blue')]

# Rolling two 6-sided dice (or repeat=2)
print(len(list(product(range(1, 7), repeat=2))))  # 36
```

#### 12. `permutations(iterable, r=None)`
All ordered arrangements of length `r` (default: `len(iterable)`), without repeating elements by position. Order matters: `('A', 'B') != ('B', 'A')`.

```python
from itertools import permutations

print(list(permutations(["A", "B", "C"], 2)))
# [('A', 'B'), ('A', 'C'), ('B', 'A'), ('B', 'C'), ('C', 'A'), ('C', 'B')]
```

#### 13. `combinations(iterable, r)`
All unordered subsets of length `r` from `iterable` (in input order, no duplicate pairs). Order does not matter: `('A', 'B')` is yielded, `('B', 'A')` is not.

```python
from itertools import combinations

print(list(combinations(["A", "B", "C"], 2)))
# [('A', 'B'), ('A', 'C'), ('B', 'C')]
```

*(Related: `itertools.combinations_with_replacement(iterable, r)` allows individual elements to be chosen more than once.)*

---

## 14. Memory Efficiency: Generators vs Lists

A list eagerly computes and stores **every** item (plus object references and over-allocated buffer space) in RAM at once. A generator stores **only its current execution frame**—taking roughly ~200 bytes regardless of whether it yields 10 items or 10 billion items.

### Measuring Memory with `sys.getsizeof()` and `tracemalloc`

```python
import sys
import tracemalloc

N = 1_000_000

list_comp = [x * 2 for x in range(N)]
gen_exp = (x * 2 for x in range(N))

print(f"List size:      {sys.getsizeof(list_comp):,} bytes")
print(f"Generator size: {sys.getsizeof(gen_exp):,} bytes")
# Typical output (64-bit Python):
# List size:      8,448,728 bytes (~8.4 MB just for pointers, plus int objects!)
# Generator size: 200 bytes
```

Using `tracemalloc` to measure total peak memory (including the integer objects created):

```python
tracemalloc.start()
_ = sum([x * 2 for x in range(1_000_000)])
current, peak_list = tracemalloc.get_traced_memory()
tracemalloc.stop()

tracemalloc.start()
_ = sum(x * 2 for x in range(1_000_000))
current, peak_gen = tracemalloc.get_traced_memory()
tracemalloc.stop()

print(f"Peak memory (list):      {peak_list / 1024 / 1024:.2f} MB")
print(f"Peak memory (generator): {peak_gen / 1024:.2f} KB")
```

### Trade-offs: When NOT to Use a Generator

Generators are **not** always better than lists:

| Requirement | Better Choice | Why |
| :--- | :--- | :--- |
| Single pass over large/unknown data | **Generator** | Minimal memory; stops early if consumer breaks |
| Need `len()`, indexing (`x[5]`), or slicing (`x[-3:]`) | **List / Sequence** | Generators don't support `len()` or random access |
| Need to iterate multiple times | **List / Tuple** | Generators exhaust after one pass |
| Very small collections looped repeatedly | **List** | List comprehension in C can be slightly faster for tiny data |

---

## 15. Use Cases for Generators

### 1. Reading Large Files Line by Line (Log / CSV Parsing)
Avoid `file.readlines()`, which loads the entire file into memory.

```python
def read_large_log(filepath: str, keyword: str):
    with open(filepath, "r", encoding="utf-8") as f:
        for line_no, line in enumerate(f, start=1):
            if keyword in line:
                yield line_no, line.strip()
```

### 2. Streaming Paginated API Responses
Hide pagination mechanics behind a clean stream so callers can just iterate over items.

```python
def fetch_all_pages(fetch_page_fn):
    page = 1
    while True:
        data = fetch_page_fn(page)
        if not data:
            break
        yield from data
        page += 1
```

### 3. Batching / Chunking Data (e.g., Bulk DB Inserts)
Process huge streams in fixed-size batches without holding everything in RAM.

```python
from itertools import islice

def batched(iterable, size: int):
    """Yield lists/tuples of up to `size` items (built into itertools.batched in Python 3.12+)."""
    it = iter(iterable)
    while batch := tuple(islice(it, size)):
        yield batch


print(list(batched(range(10), 4)))
# [(0, 1, 2, 3), (4, 5, 6, 7), (8, 9)]
```

### 4. Sliding Windows
Useful for moving averages, n-grams, and time-series analysis.

```python
from collections import deque

def sliding_window(iterable, size: int):
    window = deque(maxlen=size)
    for item in iterable:
        window.append(item)
        if len(window) == size:
            yield tuple(window)


print(list(sliding_window([10, 20, 30, 40, 50], 3)))
# [(10, 20, 30), (20, 30, 40), (30, 40, 50)]
```

### 5. Tree / Graph / Directory Traversal
Yield nodes lazily as they are discovered, allowing callers to stop as soon as a target is found.

```python
def dfs_preorder(node):
    if node is None:
        return
    yield node["value"]
    for child in node.get("children", []):
        yield from dfs_preorder(child)
```

### 6. Context Managers (`@contextlib.contextmanager`)
Python's standard library uses a single-`yield` generator to write clean setup/teardown context managers:

```python
from contextlib import contextmanager
import time

@contextmanager
def timer(label: str):
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"{label}: {elapsed:.4f}s")

with timer("Sum 1M"):
    sum(range(1_000_000))
```

---

## Quick Summary Cheat Sheet

- **Iterable**: Implements `__iter__()`; produces a fresh iterator on each call.
- **Iterator**: Implements `__iter__()` (returning `self`) and `__next__()` (raising `StopIteration` when done).
- **Generator function**: Uses `yield`; automatically creates a lazy, stateful iterator.
- **Generator expression**: `(expr for x in it)` — concise, single-pass, memory-efficient.
- **Two-way control**: `send()` pushes data in, `throw()` injects exceptions, `close()` runs cleanup via `GeneratorExit`.
- **`yield from`**: Delegates cleanly to subgenerators/iterables and captures `return` values.
- **`itertools`**: Battle-tested C-speed lazy building blocks for infinite streams, slicing, grouping, and combinatorics.
