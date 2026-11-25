# Lambda Functions, `map`, `filter`, and `reduce`

Python supports both procedural/object-oriented styles and key **functional programming** constructs. Anonymous functions (`lambda`) and higher-order functions (`map`, `filter`, `reduce`, `sorted`, `max`, `min`) allow concise, expressive data transformations without verbose boilerplate.

---

## Table of Contents

1. [Lambda Functions — Syntax and Usage](#1-lambda-functions--syntax-and-usage)
2. [Lambda with Multiple Arguments](#2-lambda-with-multiple-arguments)
3. [Lambda Limitations](#3-lambda-limitations)
4. [When to Use `lambda` vs `def`](#4-when-to-use-lambda-vs-def)
5. [`map()` Function — Syntax and Examples](#5-map-function--syntax-and-examples)
6. [`map()` with Multiple Iterables](#6-map-with-multiple-iterables)
7. [`filter()` Function — Syntax and Examples](#7-filter-function--syntax-and-examples)
8. [`reduce()` Function (`functools.reduce`) — Syntax and Examples](#8-reduce-function-functoolsreduce--syntax-and-examples)
9. [Combining `map`, `filter`, and `reduce`](#9-combining-map-filter-and-reduce)
10. [`map` / `filter` vs List Comprehensions](#10-map--filter-vs-list-comprehensions)
11. [`sorted()` with `key=lambda`](#11-sorted-with-keylambda)
12. [`max()` and `min()` with `key=lambda`](#12-max-and-min-with-keylambda)
13. [Lambda in GUI Callbacks and Event Handling](#13-lambda-in-gui-callbacks-and-event-handling)
14. [Functional Programming Concepts in Python](#14-functional-programming-concepts-in-python)
15. [`functools` Module Highlights (`partial`, `reduce`, `lru_cache`, `wraps`)](#15-functools-module-highlights-partial-reduce-lru_cache-wraps)

---

## 1. Lambda Functions — Syntax and Usage

A **lambda function** is a small, anonymous (unnamed) inline function defined with the `lambda` keyword instead of `def`.

### Syntax

```python
lambda parameters: expression
```

- `lambda`: keyword that starts the anonymous function.
- `parameters`: zero or more comma-separated arguments (just like in `def`).
- `expression`: a **single** expression that is evaluated and automatically returned (no `return` keyword).

### Basic Examples

```python
# Standard function
def square_def(x):
    return x ** 2

# Equivalent lambda function
square = lambda x: x ** 2

print(square_def(5))  # 25
print(square(5))      # 25

# Zero-argument lambda
greet = lambda: "Hello, Python!"
print(greet())        # Hello, Python!

# Immediately Invoked Function Expression (IIFE)
result = (lambda x: x * 10)(4)
print(result)         # 40
```

> **Note:** Assigning a lambda to a variable (`square = lambda x: x ** 2`) works, but PEP 8 discourages it—use `def` whenever you want to bind a function to a name. Lambdas shine when passed directly as arguments to other functions.

---

## 2. Lambda with Multiple Arguments

Lambda functions support the full range of Python parameter styles: positional, default, `*args`, `**kwargs`, and keyword-only arguments.

```python
# Multiple positional arguments
add = lambda a, b: a + b
print(add(3, 7))              # 10

# Three arguments
volume = lambda l, w, h: l * w * h
print(volume(2, 3, 4))        # 24

# Default argument values
power = lambda base, exp=2: base ** exp
print(power(5))               # 25
print(power(5, 3))            # 125

# Variable positional arguments (*args)
total = lambda *args: sum(args)
print(total(1, 2, 3, 4))      # 10

# Variable keyword arguments (**kwargs)
format_user = lambda **kwargs: ", ".join(f"{k}={v}" for k, v in kwargs.items())
print(format_user(name="Alice", role="Admin"))  # name=Alice, role=Admin

# Conditional expression (ternary operator) inside lambda
classify = lambda n: "even" if n % 2 == 0 else "odd"
print(classify(7))            # odd
```

---

## 3. Lambda Limitations

Because `lambda` is designed for concise, inline expressions, Python enforces strict constraints on what can appear inside one:

1. **Single expression only:** Cannot contain multiple statements or multi-line logic.
2. **No statements allowed:** Keywords like `return`, `pass`, `raise`, `assert`, `del`, `import`, `for`, `while`, or standard `if`/`else` blocks are syntax errors (only inline conditional expressions `a if cond else b` are allowed).
3. **No variable assignment (`=`):** Standard assignment is a statement, not an expression. (The walrus operator `:=` is an expression, though using it inside lambdas reduces readability.)
4. **No type annotations:** Unlike `def`, lambda parameters and return values cannot have type hints (`lambda x: int: ...` is invalid syntax).
5. **No docstrings:** Lambdas cannot carry a docstring (`__doc__` is `None`).
6. **Poor debugging & tracebacks:** All lambdas share the generic name `"<lambda>"` in tracebacks and `repr()`, making stack traces harder to inspect.

```python
# Invalid lambda examples (SyntaxError):
# bad1 = lambda x: return x + 1        # 'return' statement not allowed
# bad2 = lambda x: y = x * 2; y + 1    # assignment & multiple statements not allowed
# bad3 = lambda x: int: x + 1          # type annotations not allowed

# Traceback behavior
boom = lambda x: 1 / x
print(boom.__name__)  # <lambda>
# Calling boom(0) shows: ZeroDivisionError in <lambda>
```

---

## 4. When to Use `lambda` vs `def`

| Criteria | Use `lambda` | Use `def` |
|---|---|---|
| **Lifetime / Reuse** | One-off, throwaway function passed inline | Reused in multiple places or named for clarity |
| **Complexity** | Simple, single expression | Multiple steps, loops, error handling, or side effects |
| **Readability** | Short enough to read effortlessly in place | Logic requires a descriptive name or docstring |
| **Debugging / Typing** | Trivial logic unlikely to need stack traces | Needs type hints, docstrings, or clear tracebacks |
| **PEP 8 Guidance** | Pass directly as an argument (`key=...`) | Never bind a `lambda` to an identifier (`f = lambda ...`) |

```python
# Good use of lambda: short, inline key function
names = ["charlie", "Alice", "bob"]
print(sorted(names, key=lambda s: s.lower()))
# ['Alice', 'bob', 'charlie']

# Bad use of lambda: complex, hard to read, and assigned to a name
parse_score = lambda s: int(s.strip()) if s.strip().isdigit() else 0

# Better with def: clear name, docstring, type hints, readable branching
def parse_score(raw: str) -> int:
    """Convert a string to an integer score, defaulting to 0 if invalid."""
    cleaned = raw.strip()
    return int(cleaned) if cleaned.isdigit() else 0
```

---

## 5. `map()` Function — Syntax and Examples

`map()` applies a given function to every item of an iterable and returns a **lazy iterator** (`map` object) yielding the transformed results.

### Syntax

```python
map(function, iterable, ...)
```

- `function`: a callable (often a `lambda` or built-in function) applied to each item.
- `iterable`: one or more iterables (list, tuple, set, generator, etc.).
- **Returns:** a `map` iterator. Wrap in `list()`, `tuple()`, or iterate over it to consume values.

### Examples

```python
numbers = [1, 2, 3, 4, 5]

# Using lambda
squared = map(lambda x: x ** 2, numbers)
print(squared)         # <map object at 0x...>
print(list(squared))   # [1, 4, 9, 16, 25]

# Note: iterators are exhausted once consumed
print(list(squared))   # []

# Using a named function or built-in
words = ["  alice ", "BOB", " Charlie"]
cleaned = list(map(lambda w: w.strip().title(), words))
print(cleaned)         # ['Alice', 'Bob', 'Charlie']

# Passing a built-in function directly (no lambda needed)
str_nums = ["10", "20", "30"]
ints = list(map(int, str_nums))
print(ints)            # [10, 20, 30]
```

---

## 6. `map()` with Multiple Iterables

When multiple iterables are passed to `map()`, the function must accept as many arguments as there are iterables. `map()` takes one element from each iterable in parallel and **stops when the shortest iterable is exhausted** (just like `zip()`).

```python
a = [1, 2, 3]
b = [10, 20, 30]

# Pairwise addition
sums = list(map(lambda x, y: x + y, a, b))
print(sums)  # [11, 22, 33]

# Unequal lengths: stops at the shortest iterable
prices = [100, 200, 300, 400]
discounts = [0.10, 0.25]

final_prices = list(map(lambda p, d: p * (1 - d), prices, discounts))
print(final_prices)  # [90.0, 150.0]

# Three iterables
first_names = ["Ada", "Grace"]
last_names = ["Lovelace", "Hopper"]
years = [1815, 1906]

profiles = list(map(lambda f, l, y: f"{f} {l} ({y})", first_names, last_names, years))
print(profiles)  # ['Ada Lovelace (1815)', 'Grace Hopper (1906)']
```

---

## 7. `filter()` Function — Syntax and Examples

`filter()` constructs a **lazy iterator** from elements of an iterable for which a predicate function returns a truthy value.

### Syntax

```python
filter(function, iterable)
```

- `function`: a callable returning `True` or `False` (or truthy/falsy). If `None` is passed, `filter(None, iterable)` removes all falsy values (`False`, `0`, `""`, `None`, `[]`, etc.).
- `iterable`: the sequence or iterable to filter.

### Examples

```python
nums = [-3, -1, 0, 2, 4, 5, 8]

# Keep only even numbers
evens = list(filter(lambda x: x % 2 == 0, nums))
print(evens)      # [0, 2, 4, 8]

# Keep positive numbers
positives = list(filter(lambda x: x > 0, nums))
print(positives)  # [2, 4, 5, 8]

# Filter dictionaries
users = [
    {"name": "Alice", "active": True},
    {"name": "Bob", "active": False},
    {"name": "Clara", "active": True},
]
active_names = [u["name"] for u in filter(lambda u: u["active"], users)]
print(active_names)  # ['Alice', 'Clara']

# Using None as the function to drop falsy items
mixed = ["hello", "", None, 0, 42, [], "world"]
truthy_only = list(filter(None, mixed))
print(truthy_only)   # ['hello', 42, 'world']
```

---

## 8. `reduce()` Function (`functools.reduce`) — Syntax and Examples

Unlike `map()` and `filter()`, `reduce()` is not a built-in in Python 3—it lives in the `functools` module. It applies a two-argument function cumulatively to items of an iterable from left to right, reducing the iterable to a **single value**.

### Syntax

```python
from functools import reduce

reduce(function, iterable[, initializer])
```

- `function`: takes two arguments `(accumulator, current_item)` and returns the updated accumulator.
- `iterable`: sequence of items to combine.
- `initializer` *(optional)*: starting value placed before the items of `iterable` (and returned if `iterable` is empty). Without `initializer`, an empty iterable raises `TypeError`.

### How `reduce()` Evaluates

For `reduce(lambda acc, x: acc + x, [1, 2, 3, 4])`:

1. `acc = 1, x = 2` -> `3`
2. `acc = 3, x = 3` -> `6`
3. `acc = 6, x = 4` -> `10`

### Examples

```python
from functools import reduce

nums = [1, 2, 3, 4, 5]

# Product of all elements
product = reduce(lambda acc, x: acc * x, nums)
print(product)  # 120

# With an initializer (starting accumulator = 100)
sum_with_base = reduce(lambda acc, x: acc + x, nums, 100)
print(sum_with_base)  # 115

# Safe on empty iterable when initializer is provided
empty_sum = reduce(lambda acc, x: acc + x, [], 0)
print(empty_sum)  # 0

# Finding maximum without max()
maximum = reduce(lambda a, b: a if a > b else b, [14, 3, 29, 7])
print(maximum)  # 29

# Flattening a list of lists
nested = [[1, 2], [3, 4], [5]]
flat = reduce(lambda acc, lst: acc + lst, nested, [])
print(flat)  # [1, 2, 3, 4, 5]
```

> **Tip:** Prefer dedicated built-ins like `sum()`, `math.prod()`, `max()`, `min()`, `any()`, `all()`, and `"".join()` over `reduce()` when they fit—they are faster and clearer.

---

## 9. Combining `map`, `filter`, and `reduce`

`map()`, `filter()`, and `reduce()` compose cleanly into data-processing pipelines. Because `map()` and `filter()` are lazy, intermediate lists are never allocated until `reduce()` (or `list()`) consumes the stream.

### Example: Processing Order Line Items

Calculate the total revenue from paid orders after applying a 10% tax:

```python
from functools import reduce

orders = [
    {"id": 1, "amount": 50.0, "paid": True},
    {"id": 2, "amount": 120.0, "paid": False},
    {"id": 3, "amount": 80.0, "paid": True},
    {"id": 4, "amount": 20.0, "paid": True},
]

# Step 1: filter paid orders
paid_orders = filter(lambda o: o["paid"], orders)

# Step 2: map each order to its amount with 10% tax
taxed_amounts = map(lambda o: o["amount"] * 1.10, paid_orders)

# Step 3: reduce to the grand total
total_revenue = reduce(lambda acc, val: acc + val, taxed_amounts, 0.0)

print(f"Total revenue: ${total_revenue:.2f}")  # Total revenue: $165.00
```

### Example: Text Normalization Pipeline

```python
from functools import reduce

raw_words = ["  python ", "", "LAMBDA", "  ", "Map  ", "filter"]

# Keep non-empty trimmed words, normalize to lowercase, join with " | "
non_empty = filter(lambda s: s.strip() != "", raw_words)
normalized = map(lambda s: s.strip().lower(), non_empty)
joined = reduce(lambda a, b: f"{a} | {b}", normalized)

print(joined)  # python | lambda | map | filter
```

---

## 10. `map` / `filter` vs List Comprehensions

In idiomatic Python, **list comprehensions** (and **generator expressions**) are usually preferred over `map()` and `filter()` with `lambda` because they read more naturally and combine transformation and filtering in one construct.

### Side-by-Side Comparison

```python
nums = [1, 2, 3, 4, 5, 6]

# 1. Transform only
m1 = list(map(lambda x: x ** 2, nums))
c1 = [x ** 2 for x in nums]

# 2. Filter only
m2 = list(filter(lambda x: x % 2 == 0, nums))
c2 = [x for x in nums if x % 2 == 0]

# 3. Filter + Transform combined
m3 = list(map(lambda x: x ** 2, filter(lambda x: x % 2 == 0, nums)))
c3 = [x ** 2 for x in nums if x % 2 == 0]

print(m3 == c3)  # True -> [4, 16, 36]
```

### Lazy Evaluation Comparison

If you don't need a `list` immediately, a **generator expression** matches the laziness of `map`/`filter`:

```python
# Both are lazy and use O(1) memory
lazy_map = map(lambda x: x ** 2, range(1_000_000))
lazy_gen = (x ** 2 for x in range(1_000_000))
```

### When Each Wins

- **Use list comprehensions / generator expressions** when you need a `lambda` in `map`/`filter`, or when transforming and filtering together.
- **Use `map()` / `filter()`** when you already have an existing function (no `lambda` needed), e.g. `map(str.strip, lines)` or `map(int, tokens)`, or when mapping across multiple iterables (`map(pow, bases, exponents)`).

---

## 11. `sorted()` with `key=lambda`

`sorted(iterable, *, key=None, reverse=False)` returns a new sorted list. The `key` parameter accepts a single-argument function that computes a **comparison key** for each element before sorting.

```python
# Sort strings by length
words = ["banana", "fig", "apple", "kiwi"]
print(sorted(words, key=lambda w: len(w)))
# ['fig', 'kiwi', 'apple', 'banana']

# Sort list of tuples by second element
pairs = [(1, "c"), (3, "a"), (2, "b")]
print(sorted(pairs, key=lambda p: p[1]))
# [(3, 'a'), (2, 'b'), (1, 'c')]

# Sort dictionaries by a field
students = [
    {"name": "Alice", "grade": 88},
    {"name": "Bob", "grade": 95},
    {"name": "Clara", "grade": 95},
]

# Sort by grade descending, then name ascending (tuple keys!)
ranked = sorted(students, key=lambda s: (-s["grade"], s["name"]))
print(ranked)
# [{'name': 'Bob', 'grade': 95}, {'name': 'Clara', 'grade': 95}, {'name': 'Alice', 'grade': 88}]

# Sort dictionary items by value
scores = {"Alice": 85, "Bob": 92, "Charlie": 78}
by_score = sorted(scores.items(), key=lambda item: item[1], reverse=True)
print(by_score)
# [('Bob', 92), ('Alice', 85), ('Charlie', 78)]
```

> **Performance tip:** For simple indexing/attribute lookups, `operator.itemgetter` and `operator.attrgetter` are slightly faster alternatives to `lambda` (e.g., `key=itemgetter(1)`).

---

## 12. `max()` and `min()` with `key=lambda`

Like `sorted()`, `max()` and `min()` accept a `key` function to determine how elements are compared, while still returning the **original element** (not the key value).

```python
words = ["apple", "strawberry", "fig", "blueberry"]

longest = max(words, key=lambda w: len(w))
shortest = min(words, key=lambda w: len(w))
print(longest)   # strawberry
print(shortest)  # fig

# Find cheapest and most expensive product
products = [
    {"name": "Laptop", "price": 1200},
    {"name": "Mouse", "price": 25},
    {"name": "Monitor", "price": 350},
]

cheapest = min(products, key=lambda p: p["price"])
priciest = max(products, key=lambda p: p["price"])
print(cheapest)  # {'name': 'Mouse', 'price': 25}
print(priciest)  # {'name': 'Laptop', 'price': 1200}

# Find key with highest value in a dict
inventory = {"apples": 15, "bananas": 42, "oranges": 8}
most_stocked = max(inventory, key=lambda k: inventory[k])
print(most_stocked)  # bananas

# Guarding against empty iterables with default=
empty = []
print(max(empty, key=lambda x: x["price"], default=None))  # None
```

---

## 13. Lambda in GUI Callbacks and Event Handling

GUI frameworks (such as **Tkinter**, PyQt, or custom event dispatchers) frequently require a **zero-argument callable** as a callback (e.g., `command=...`). When you need to pass arguments to the handler, `lambda` lets you wrap the call without creating dozens of tiny helper functions.

### Basic Callback Wrapping (Tkinter)

```python
import tkinter as tk

def handle_click(action_name):
    print(f"Button clicked: {action_name}")

root = tk.Tk()

# Passing an argument to the callback using lambda
save_btn = tk.Button(root, text="Save", command=lambda: handle_click("Save"))
quit_btn = tk.Button(root, text="Quit", command=lambda: handle_click("Quit"))

save_btn.pack()
quit_btn.pack()
# root.mainloop()
```

### Event Binding with the `event` Object

When a widget passes an `event` object automatically, the lambda accepts `event` and forwards extra parameters:

```python
entry = tk.Entry(root)
entry.bind("<Return>", lambda event: handle_click(f"Submitted: {entry.get()}"))
```

### Crucial Gotcha: Late Binding in Loops

When creating callbacks inside a `for` loop, lambdas capture the loop variable **by reference**, not by its value at that iteration (late binding). Every button ends up using the final value of the loop variable!

```python
# Simulate a button callback registry
callbacks_buggy = []
for i in range(3):
    callbacks_buggy.append(lambda: print(f"Button {i}"))

for cb in callbacks_buggy:
    cb()
# Output (BUG):
# Button 2
# Button 2
# Button 2
```

#### Fix 1: Capture the Value with a Default Argument

Default parameter values are evaluated at function definition time:

```python
callbacks_fixed = []
for i in range(3):
    callbacks_fixed.append(lambda i=i: print(f"Button {i}"))

for cb in callbacks_fixed:
    cb()
# Output:
# Button 0
# Button 1
# Button 2
```

#### Fix 2: Use `functools.partial` (Cleaner)

```python
from functools import partial

def on_button(index):
    print(f"Button {index}")

callbacks_partial = [partial(on_button, i) for i in range(3)]
for cb in callbacks_partial:
    cb()
# Output:
# Button 0
# Button 1
# Button 2
```

---

## 14. Functional Programming Concepts in Python

Python is multi-paradigm, not purely functional, yet it provides strong support for core functional programming (FP) principles:

1. **First-Class Functions:** Functions are objects—they can be assigned to variables, stored in data structures, passed as arguments, and returned from other functions.
2. **Higher-Order Functions (HOFs):** Functions that take functions as arguments (`map`, `filter`, `sorted`) or return functions (decorators, closures, `partial`).
3. **Pure Functions:** Functions whose output depends solely on their inputs and produce **no side effects** (no mutating arguments or global state, no I/O). Pure functions are easy to test, cache, and reason about.
4. **Immutability Preference:** Instead of mutating lists in place (`lst.sort()`), FP prefers creating new values (`sorted(lst)`, tuples, `frozenset`, `dataclasses` with `frozen=True`).
5. **Lazy Evaluation:** Generators, generator expressions, `map()`, `filter()`, and `itertools` produce items on demand rather than materializing entire collections in memory.
6. **Closures:** Inner functions remember variables from their enclosing lexical scope even after the outer function finishes executing.

```python
# Closure & Higher-Order Function example
def make_multiplier(factor):
    def multiply(value):
        return value * factor
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)
print(double(5), triple(5))  # 10 15

# Pure vs Impure function
def add_pure(items, new_item):
    """Pure: returns a new tuple without mutating the input."""
    return (*items, new_item)

def add_impure(items, new_item):
    """Impure: mutates the caller's list in place."""
    items.append(new_item)
    return items
```

> **Note on Recursion:** Unlike Haskell or Scheme, Python does **not** perform tail-call optimization (TCO) and enforces a recursion limit (typically 1000 frames via `sys.getrecursionlimit()`). Prefer loops, comprehensions, or `itertools` over deep recursion.

---

## 15. `functools` Module Highlights (`partial`, `reduce`, `lru_cache`, `wraps`)

The standard library `functools` module provides tools for working with higher-order functions and callables.

### `functools.partial`

Freezes some portion of a function's positional and/or keyword arguments, returning a new callable with a simpler signature.

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))  # 25
print(cube(5))    # 125

# Useful with map() or callbacks
base2 = partial(int, base=2)
print(list(map(base2, ["101", "110", "111"])))  # [5, 6, 7]
```

### `functools.reduce`

Cumulatively applies a binary function across an iterable (covered in detail in [Section 8](#8-reduce-function-functoolsreduce--syntax-and-examples)).

```python
from functools import reduce
import operator

nums = [2, 3, 4, 5]
print(reduce(operator.mul, nums))  # 120
```

### `functools.lru_cache`

Memoizes a function by caching recent return values in a Least Recently Used (LRU) cache, avoiding redundant computation on repeated calls with the same hashable arguments.

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(50))           # 12586269025 (computed instantly)
print(fibonacci.cache_info())  # CacheInfo(hits=48, misses=51, maxsize=128, currsize=51)
fibonacci.cache_clear()        # Clear cached entries if needed
```

> **Tip:** Set `maxsize=None` (or use `@functools.cache` in Python 3.9+) for an unbounded cache. Arguments must be hashable.

### `functools.wraps`

Used inside **decorators** to copy metadata (`__name__`, `__doc__`, `__annotations__`, `__wrapped__`) from the original function onto the wrapper function. Without `@wraps`, debugging and introspection tools see the wrapper's name instead of the decorated function's name.

```python
from functools import wraps

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args}, {kwargs}")
        return func(*args, **kwargs)
    return wrapper

@log_calls
def greet(name):
    """Return a greeting message."""
    return f"Hello, {name}!"

print(greet("Alice"))   # Calling greet with ('Alice',), {} -> Hello, Alice!
print(greet.__name__)   # greet  (would be 'wrapper' without @wraps)
print(greet.__doc__)    # Return a greeting message.
```

---

## Quick Reference Summary

| Tool | Purpose | Returns | Notes |
|---|---|---|---|
| `lambda args: expr` | Inline anonymous function | Function object | Single expression only |
| `map(fn, *iters)` | Transform items element-wise | Lazy `map` iterator | Stops at shortest iterable |
| `filter(fn, iter)` | Keep items where `fn(item)` is truthy | Lazy `filter` iterator | `fn=None` drops falsy items |
| `reduce(fn, iter, init)` | Collapse iterable to one value | Single reduced value | Imported from `functools` |
| `sorted(iter, key=...)` | Sort using custom key | New `list` | Original iterable unchanged |
| `max` / `min(iter, key=...)` | Extreme item by custom key | Original element | Supports `default=` for empty |
| `functools.partial` | Pre-fill function arguments | Callable `partial` object | Safer than loop lambdas |
| `functools.lru_cache` | Memoize pure function calls | Decorated function | Requires hashable args |
| `functools.wraps` | Preserve wrapped function metadata | Decorator helper | Standard practice in decorators |
