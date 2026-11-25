# Functions in Python

Functions are reusable, self-contained blocks of code designed to perform a specific task. They promote modularity, eliminate code duplication (DRY — *Don't Repeat Yourself*), improve readability, and make programs easier to test and maintain.

---

## Table of Contents

1. [Defining Functions (`def` Keyword)](#1-defining-functions-def-keyword)
2. [Function Parameters and Arguments](#2-function-parameters-and-arguments)
3. [Default Parameters](#3-default-parameters)
4. [Keyword Arguments](#4-keyword-arguments)
5. [`*args` and `**kwargs` (Variable-Length Arguments)](#5-args-and-kwargs-variable-length-arguments)
6. [The `return` Statement](#6-the-return-statement)
7. [Docstrings](#7-docstrings)
8. [Function Annotations & Type Hints](#8-function-annotations--type-hints)
9. [Nested Functions](#9-nested-functions)
10. [First-Class Functions (Functions as Objects)](#10-first-class-functions-functions-as-objects)
11. [Higher-Order Functions](#11-higher-order-functions)
12. [Recursive Functions](#12-recursive-functions)
13. [Anonymous Functions (`lambda`)](#13-anonymous-functions-lambda)
14. [`map()`, `filter()`, and `reduce()`](#14-map-filter-and-reduce)
15. [Global vs. Local Variables](#15-global-vs-local-variables)
16. [The `global` and `nonlocal` Keywords](#16-the-global-and-nonlocal-keywords)
17. [Function Caching (`functools.lru_cache`)](#17-function-caching-functoolslru_cache)
18. [Pure Functions and Side Effects](#18-pure-functions-and-side-effects)

---

## 1. Defining Functions (`def` Keyword)

In Python, a function is defined using the `def` keyword, followed by the function name, parentheses `()` (which may hold parameters), and a colon `:`. The function body is indented.

### Syntax

```python
def function_name(parameters):
    """Optional docstring."""
    # Function body
    return value  # Optional
```

### Example

```python
def greet():
    print("Hello, welcome to Python!")


# Calling (invoking) the function
greet()  # Output: Hello, welcome to Python!
```

### Key Points

- **Naming conventions (PEP 8):** Use `snake_case` (lowercase words separated by underscores), such as `calculate_total` or `get_user_by_id`.
- **Definition vs. Execution:** Python creates the function object when `def` is executed, **not** when the file is merely parsed, and the body runs only when the function is called.
- **`pass` placeholder:** Use `pass` (or `...`) when sketching out a function stub to avoid an `IndentationError`.

```python
def todo_feature():
    pass
```

---

## 2. Function Parameters and Arguments

Although often used interchangeably in casual conversation, **parameters** and **arguments** have distinct meanings:

- **Parameter:** A variable listed inside the parentheses in the function **definition**.
- **Argument:** The actual value passed to the function when it is **called**.

```python
def add(a, b):  # 'a' and 'b' are parameters
    return a + b


result = add(3, 5)  # 3 and 5 are arguments
print(result)       # Output: 8
```

### Positional Arguments

By default, arguments are matched to parameters by their position (order):

```python
def describe_pet(animal, name):
    print(f"I have a {animal} named {name}.")


describe_pet("dog", "Rex")  # I have a dog named Rex.
describe_pet("Rex", "dog")  # I have a Rex named dog. (order matters!)
```

### Positional-Only (`/`) and Keyword-Only (`*`) Parameters (Python 3.8+)

You can restrict how arguments are passed using `/` and `*` in the signature:

- Parameters **before `/`** are **positional-only**.
- Parameters **after `*`** are **keyword-only**.
- Parameters **between `/` and `*`** can be passed either positionally or by keyword.

```python
def configure(host, port, /, protocol="https", *, timeout=30, verify=True):
    print(f"{protocol}://{host}:{port} (timeout={timeout}, verify={verify})")


# Valid calls:
configure("example.com", 443)
configure("example.com", 8080, protocol="http", timeout=10)

# Invalid calls (raise TypeError):
# configure(host="example.com", port=443)        # 'host' and 'port' are positional-only
# configure("example.com", 443, "http", 10)      # 'timeout' is keyword-only
```

### Call-by-Object-Reference (Pass-by-Assignment)

Python passes **object references** by value. Rebinding a parameter inside a function does not affect the caller's variable, but mutating a mutable object (like a `list` or `dict`) in place *does* affect the caller's object.

```python
def modify(num, items):
    num = num + 10         # Rebinds local 'num'; caller's int is unchanged
    items.append(4)        # Mutates the list object in place


x = 5
my_list = [1, 2, 3]
modify(x, my_list)

print(x)        # Output: 5
print(my_list)  # Output: [1, 2, 3, 4]
```

---

## 3. Default Parameters

Default parameters let you specify a fallback value when an argument is omitted by the caller.

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"


print(greet("Alice"))              # Output: Hello, Alice!
print(greet("Bob", "Good morning"))  # Output: Good morning, Bob!
```

### Ordering Rule

Parameters **with** default values must come **after** parameters **without** default values:

```python
# Valid
def power(base, exponent=2):
    return base ** exponent

# Invalid: SyntaxError: parameter without a default follows parameter with a default
# def power(exponent=2, base):
#     return base ** exponent
```

### Warning: The Mutable Default Argument Pitfall

Default values are evaluated **once** when the `def` statement is executed — **not** each time the function is called. Using a mutable object (like `[]` or `{}`) as a default causes state to leak across calls.

```python
# BAD: 'items' list is shared across all calls that omit it
def append_bad(value, items=[]):
    items.append(value)
    return items


print(append_bad(1))  # [1]
print(append_bad(2))  # [1, 2]  <-- Unexpected!


# GOOD: Use None as a sentinel and create a fresh object inside the body
def append_good(value, items=None):
    if items is None:
        items = []
    items.append(value)
    return items


print(append_good(1))  # [1]
print(append_good(2))  # [2]
```

---

## 4. Keyword Arguments

Keyword arguments explicitly bind values to parameter names at the call site (`name=value`). This improves readability and removes dependence on parameter order.

```python
def create_profile(username, role, active=True):
    return {"username": username, "role": role, "active": active}


# Order does not matter when using keyword arguments
user1 = create_profile(role="admin", username="ada")
user2 = create_profile("grace", active=False, role="editor")

print(user1)  # {'username': 'ada', 'role': 'admin', 'active': True}
print(user2)  # {'username': 'grace', 'role': 'editor', 'active': False}
```

### Rules for Mixing Positional and Keyword Arguments

1. All positional arguments must come **before** keyword arguments.
2. You cannot pass a value for the same parameter both positionally and by keyword (raises `TypeError`).

```python
# SyntaxError: positional argument follows keyword argument
# create_profile(username="ada", "admin")

# TypeError: create_profile() got multiple values for argument 'username'
# create_profile("ada", username="grace", role="admin")
```

---

## 5. `*args` and `**kwargs` (Variable-Length Arguments)

Sometimes a function needs to accept an arbitrary number of arguments.

- `*args` collects extra **positional** arguments into a **tuple**.
- `**kwargs` collects extra **keyword** arguments into a **dictionary**.

*(Note: `args` and `kwargs` are conventions; the `*` and `**` operators are what carry the special syntax.)*

### Using `*args`

```python
def sum_all(*args):
    print(type(args), args)
    return sum(args)


print(sum_all(1, 2, 3))       # <class 'tuple'> (1, 2, 3) -> 6
print(sum_all(10, 20, 30, 40)) # <class 'tuple'> (10, 20, 30, 40) -> 100
```

### Using `**kwargs`

```python
def build_url(base, **kwargs):
    print(type(kwargs), kwargs)
    query = "&".join(f"{k}={v}" for k, v in kwargs.items())
    return f"{base}?{query}" if query else base


print(build_url("https://api.example.com/search", q="python", page=2))
# <class 'dict'> {'q': 'python', 'page': 2}
# https://api.example.com/search?q=python&page=2
```

### Standard Parameter Ordering

When combining all parameter kinds in a single signature, follow this exact order:

```python
def full_signature(pos_only, /, standard, *args, kw_only="default", **kwargs):
    print("pos_only:", pos_only)
    print("standard:", standard)
    print("args:", args)
    print("kw_only:", kw_only)
    print("kwargs:", kwargs)


full_signature(1, 2, 3, 4, kw_only="custom", extra_a=10, extra_b=20)
# pos_only: 1
# standard: 2
# args: (3, 4)
# kw_only: custom
# kwargs: {'extra_a': 10, 'extra_b': 20}
```

### Unpacking Arguments at the Call Site

You can also use `*` (unpacking an iterable) and `**` (unpacking a mapping) when **calling** a function:

```python
def point_3d(x, y, z):
    return f"({x}, {y}, {z})"


coords = [10, 20, 30]
print(point_3d(*coords))  # (10, 20, 30)

coord_dict = {"x": 1, "y": 2, "z": 3}
print(point_3d(**coord_dict))  # (1, 2, 3)
```

---

## 6. The `return` Statement

The `return` statement immediately exits a function and sends a value back to the caller.

### Implicit `None` Return

If a function has no `return` statement, or uses a bare `return`, it implicitly returns `None`:

```python
def log_message(msg):
    print(f"[LOG] {msg}")


res = log_message("Started")  # Output: [LOG] Started
print(res)                    # Output: None
```

### Returning a Single Value

```python
def square(n):
    return n * n


print(square(6))  # Output: 36
```

### Returning Multiple Values

Python allows returning multiple comma-separated expressions. Under the hood, Python packs them into a **tuple**, which the caller can unpack.

```python
def min_max(numbers):
    return min(numbers), max(numbers)


result = min_max([4, 9, 1, 7])
print(result)       # Output: (1, 9)
print(type(result)) # Output: <class 'tuple'>

low, high = min_max([4, 9, 1, 7])  # Tuple unpacking
print(f"Min: {low}, Max: {high}")  # Output: Min: 1, Max: 9
```

### Early Returns (Guard Clauses)

Returning early keeps nesting shallow and makes edge cases obvious:

```python
def divide(a, b):
    if b == 0:
        return None
    return a / b
```

---

## 7. Docstrings

A **docstring** (documentation string) is a string literal placed as the very first statement inside a function (or module/class) to document what it does, its parameters, return value, and exceptions.

Unlike `#` comments, docstrings are retained at runtime on the function's `__doc__` attribute and displayed by `help()`.

### Single-Line & Multi-Line Docstrings (PEP 257)

```python
def celsius_to_fahrenheit(celsius):
    """Convert a temperature from Celsius to Fahrenheit."""
    return (celsius * 9 / 5) + 32


print(celsius_to_fahrenheit.__doc__)
# Output: Convert a temperature from Celsius to Fahrenheit.
```

### Google-Style Docstring Example

```python
def fetch_items(items, limit=10, reverse=False):
    """Return a slice of items with optional ordering.

    Args:
        items (list): The source sequence of items.
        limit (int, optional): Maximum number of items to return. Defaults to 10.
        reverse (bool, optional): If True, reverse the list before slicing.
            Defaults to False.

    Returns:
        list: A new list containing up to `limit` items.

    Raises:
        ValueError: If `limit` is negative.
    """
    if limit < 0:
        raise ValueError("limit must be non-negative")
    ordered = list(reversed(items)) if reverse else list(items)
    return ordered[:limit]


help(fetch_items)
```

---

## 8. Function Annotations & Type Hints

Function annotations (PEP 3107) and type hints (PEP 484) allow you to attach metadata — typically expected types — to parameters and return values.

- Syntax: `param: type` and `-> return_type`.
- **Important:** Python **does not** enforce type hints at runtime. They are used by static type checkers (`mypy`, `pyright`), linters, and IDEs for autocomplete and bug detection.

### Basic Type Hints

```python
def greet(name: str, times: int = 1) -> str:
    return (f"Hello, {name}! " * times).strip()


print(greet("Alice", 2))
# Output: Hello, Alice! Hello, Alice!
```

### Inspecting Annotations at Runtime

```python
print(greet.__annotations__)
# Output: {'name': <class 'str'>, 'times': <class 'int'>, 'return': <class 'str'>}
```

### Container, Union, and Callable Hints (Python 3.10+)

Modern Python supports built-in generic types (`list[int]`, `dict[str, float]`) and the union operator (`|`) directly without importing `List`, `Dict`, or `Union` from `typing`.

```python
from collections.abc import Callable


def process_scores(
    scores: list[int],
    bonus: int | None = None,
    formatter: Callable[[float], str] = lambda avg: f"{avg:.2f}",
) -> dict[str, str | float]:
    total = sum(scores) + (bonus or 0)
    avg = total / len(scores)
    return {"average": avg, "display": formatter(avg)}


print(process_scores([80, 90, 100], bonus=15))
# Output: {'average': 95.0, 'display': '95.00'}
```

---

## 9. Nested Functions

A **nested function** (or inner function) is a function defined inside another function.

### Why Use Nested Functions?

1. **Encapsulation / Helper logic:** Hide internal helper routines that shouldn't be part of the module's public namespace.
2. **Closures:** An inner function remembers variables from the enclosing function's scope even after the outer function has finished executing.

```python
def format_report(title, lines):
    def format_line(index, text):
        return f"  {index}. {text.strip()}"

    header = f"=== {title.upper()} ==="
    body = [format_line(i, line) for i, line in enumerate(lines, start=1)]
    return "\n".join([header, *body])


print(format_report("tasks", [" write code ", "run tests "]))
# === TASKS ===
#   1. write code
#   2. run tests
```

### Closures

When an outer function returns an inner function that references variables from the outer scope, that inner function plus its captured environment is called a **closure**:

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor  # 'factor' is captured from the enclosing scope
    return multiply


double = make_multiplier(2)
triple = make_multiplier(3)

print(double(5))  # Output: 10
print(triple(5))  # Output: 15
```

---

## 10. First-Class Functions (Functions as Objects)

In Python, functions are **first-class objects** (first-class citizens). Anything you can do with an integer or a string, you can do with a function:

1. Assign a function to a variable.
2. Store functions in data structures (`list`, `dict`, etc.).
3. Pass a function as an argument to another function.
4. Return a function from another function.
5. Attach or inspect attributes on a function object.

```python
def shout(text):
    return text.upper() + "!"


def whisper(text):
    return text.lower() + "..."


# 1. Assign to another variable (no parentheses!)
yell = shout
print(yell("hello"))  # Output: HELLO!

# 2. Store in a dictionary (dispatch table pattern)
actions = {
    "loud": shout,
    "quiet": whisper,
}
print(actions["quiet"]("HELLO"))  # Output: hello...

# 3. Inspect function object attributes
print(shout.__name__)  # Output: shout
shout.category = "text_transform"
print(shout.category)  # Output: text_transform
```

---

## 11. Higher-Order Functions

A **higher-order function** is a function that does at least one of the following:

- Takes one or more functions as arguments.
- Returns a function as its result.

Built-in examples include `sorted(..., key=...)`, `min(..., key=...)`, `map()`, and `filter()`.

### Taking a Function as an Argument

```python
def apply_operation(values, func):
    return [func(v) for v in values]


def cube(x):
    return x ** 3


print(apply_operation([1, 2, 3, 4], cube))
# Output: [1, 8, 27, 64]
```

### Returning a Function (and Simple Decorators)

Decorators are a classic use case of higher-order functions — wrapping a function with extra behavior without modifying its source code:

```python
def log_calls(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper


@log_calls
def add(a, b):
    return a + b


add(2, 3)
# Calling add with args=(2, 3), kwargs={}
# add returned 5
```

---

## 12. Recursive Functions

A **recursive function** is a function that calls itself to solve a problem by breaking it down into smaller, self-similar subproblems.

Every recursive function needs two parts:

1. **Base case:** A condition where the function returns directly without making another recursive call (prevents infinite recursion).
2. **Recursive case:** The function reduces the problem toward the base case and calls itself.

### Example 1: Factorial

Mathematical definition: $n! = n \times (n - 1)!$ with $0! = 1$.

```python
def factorial(n):
    if n < 0:
        raise ValueError("n must be non-negative")
    if n in (0, 1):  # Base case
        return 1
    return n * factorial(n - 1)  # Recursive case


print(factorial(5))  # Output: 120
```

### Example 2: Fibonacci Sequence

Mathematical definition: $F(0) = 0, F(1) = 1, F(n) = F(n - 1) + F(n - 2)$.

```python
def fibonacci(n):
    if n < 0:
        raise ValueError("n must be non-negative")
    if n <= 1:  # Base case
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)


print([fibonacci(i) for i in range(8)])
# Output: [0, 1, 1, 2, 3, 5, 8, 13]
```

*(Note: Naive recursive Fibonacci has exponential $O(2^n)$ time complexity due to repeated subproblems. See [Section 17](#17-function-caching-functoolslru_cache) for how memoization fixes this!)*

### Example 3: Recursive Sum of a Nested List

Recursion shines on tree-like or arbitrarily nested structures:

```python
def nested_sum(data):
    total = 0
    for item in data:
        if isinstance(item, list):
            total += nested_sum(item)  # Recursive case
        else:
            total += item              # Base case
    return total


print(nested_sum([1, [2, [3, 4]], 5]))  # Output: 15
```

### Recursion Limit

Python limits call stack depth (typically 1000 frames) to prevent C stack overflows. Exceeding it raises `RecursionError`. Python does **not** perform tail-call optimization.

```python
import sys

print(sys.getrecursionlimit())  # Typically 1000
```

---

## 13. Anonymous Functions (`lambda`)

A `lambda` function is a small, unnamed (anonymous) function defined with the `lambda` keyword.

### Syntax

```python
lambda arguments: expression
```

- Can take any number of arguments (including defaults, `*args`, `**kwargs`).
- Can contain only a **single expression** (no statements like `if`/`for`/`return` blocks, though conditional expressions `a if cond else b` are allowed).
- Automatically returns the evaluated expression.

### Examples

```python
# Simple arithmetic
square = lambda x: x ** 2
print(square(5))  # Output: 25

# Conditional expression inside lambda
parity = lambda n: "even" if n % 2 == 0 else "odd"
print(parity(7))  # Output: odd
```

### Common Use Case: `key` Functions

Lambdas are most idiomatic as short inline callbacks passed to higher-order functions like `sorted()`, `min()`, or `max()`:

```python
students = [
    ("Alice", 88),
    ("Bob", 95),
    ("Charlie", 79),
]

# Sort by score (index 1), descending
ranked = sorted(students, key=lambda s: s[1], reverse=True)
print(ranked)
# Output: [('Bob', 95), ('Alice', 88), ('Charlie', 79)]
```

### `lambda` vs. `def` (PEP 8 Note)

PEP 8 recommends using `def` instead of binding a `lambda` to a variable name (`f = lambda x: ...`), because `def` gives the function a proper `__name__` for tracebacks and debugging and supports docstrings.

---

## 14. `map()`, `filter()`, and `reduce()`

These three functional programming utilities process iterables using functions.

### `map(func, iterable, ...)`

Applies `func` to every item of `iterable` (or multiple iterables in parallel) and returns a **lazy iterator**.

```python
nums = [1, 2, 3, 4]
squared = list(map(lambda x: x ** 2, nums))
print(squared)  # Output: [1, 4, 9, 16]

# Multiple iterables
a = [1, 2, 3]
b = [10, 20, 30]
sums = list(map(lambda x, y: x + y, a, b))
print(sums)  # Output: [11, 22, 33]
```

### `filter(func, iterable)`

Yields items from `iterable` for which `func(item)` is truthy. If `func` is `None`, it filters out all falsy items (`0`, `""`, `None`, `False`, etc.). Returns a **lazy iterator**.

```python
nums = [-3, -1, 0, 2, 5, 8]
positives = list(filter(lambda x: x > 0, nums))
print(positives)  # Output: [2, 5, 8]

# Filtering out falsy values with None
mixed = ["hello", "", None, "world", 0]
truthy = list(filter(None, mixed))
print(truthy)  # Output: ['hello', 'world']
```

### `functools.reduce(func, iterable[, initializer])`

Unlike `map` and `filter`, `reduce` lives in the `functools` module. It cumulatively applies a two-argument function from left to right to reduce an iterable to a **single value**.

```python
from functools import reduce

nums = [1, 2, 3, 4, 5]

# Computes ((((1 * 2) * 3) * 4) * 5)
product = reduce(lambda acc, x: acc * x, nums)
print(product)  # Output: 120

# With an explicit initial accumulator value (handles empty iterables safely)
total_plus_ten = reduce(lambda acc, x: acc + x, nums, 10)
print(total_plus_ten)  # Output: 25
```

### Pythonic Alternative: Comprehensions & Built-ins

In Python, list/generator comprehensions and built-ins like `sum()`, `any()`, and `all()` are often clearer than `map`/`filter`/`reduce`:

```python
nums = [1, 2, 3, 4, 5, 6]

# Instead of: list(map(lambda x: x ** 2, filter(lambda x: x % 2 == 0, nums)))
even_squares = [x ** 2 for x in nums if x % 2 == 0]
print(even_squares)  # Output: [4, 16, 36]
```

---

## 15. Global vs. Local Variables

A variable's **scope** determines where in the program it can be accessed.

- **Local variable:** Created inside a function; exists only during that function call and cannot be accessed outside it.
- **Global variable:** Defined at the top level of a module; readable from anywhere in that module.

### The LEGB Scope Resolution Rule

When Python looks up a variable name, it searches scopes in this exact order:

1. **L — Local:** Inside the current function.
2. **E — Enclosing:** Inside any enclosing outer functions (for nested functions).
3. **G — Global:** At the module (top) level.
4. **B — Built-in:** Python's built-in namespace (`len`, `print`, `Exception`, etc.).

```python
x = "global x"  # Global scope


def outer():
    x = "enclosing x"  # Enclosing scope

    def inner():
        x = "local x"  # Local scope
        print("Inside inner:", x)

    inner()
    print("Inside outer:", x)


outer()
print("At module level:", x)

# Output:
# Inside inner: local x
# Inside outer: enclosing x
# At module level: global x
```

### Shadowing and `UnboundLocalError`

If you **assign** to a variable anywhere inside a function (without `global` or `nonlocal`), Python treats that variable as **local** throughout the entire function. Trying to read it before it is assigned locally raises `UnboundLocalError`:

```python
count = 10


def broken_increment():
    # Python sees 'count =' below, so 'count' is marked local for the whole function.
    # Reading it before assignment fails!
    count = count + 1  # Raises UnboundLocalError: cannot access local variable 'count'...
```

---

## 16. The `global` and `nonlocal` Keywords

By default, assignments inside a function create or update local variables. To **rebind** a variable in an outer scope, use `global` or `nonlocal`.

*(Note: You do **not** need `global` or `nonlocal` simply to **read** an outer variable or to mutate a mutable object like `my_list.append(1)`.)*

### The `global` Keyword

Declares that a name inside a function refers to the module-level (global) variable, allowing the function to rebind it.

```python
counter = 0


def increment():
    global counter
    counter += 1


increment()
increment()
print(counter)  # Output: 2
```

*Best practice:* Avoid overusing `global`, as shared mutable global state makes code harder to reason about and test. Prefer passing arguments and returning values.

### The `nonlocal` Keyword

Used inside a **nested function** to rebind a variable in the nearest **enclosing** function scope (not global).

```python
def make_counter(start=0):
    count = start

    def increment(step=1):
        nonlocal count
        count += step
        return count

    return increment


counter_a = make_counter(10)
print(counter_a())    # Output: 11
print(counter_a(5))   # Output: 16
```

---

## 17. Function Caching (`functools.lru_cache`)

Memoization caches a function's return values based on its input arguments so repeated calls with the same arguments return the cached result immediately instead of recomputing.

Python provides `@functools.lru_cache` (Least Recently Used cache) and, since Python 3.9, `@functools.cache` (an unbounded cache equivalent to `lru_cache(maxsize=None)`).

### Example: Speeding Up Recursive Fibonacci

```python
from functools import lru_cache


@lru_cache(maxsize=128)
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)


print(fibonacci(50))  # Instant: 12586269025
print(fibonacci.cache_info())
# CacheInfo(hits=48, misses=51, maxsize=128, currsize=51)

# Clear the cache if needed
fibonacci.cache_clear()
```

### Important Parameters & Caveats

- **`maxsize`:** Maximum number of entries to keep (default `128`). Set `maxsize=None` for unlimited caching.
- **`typed=False`:** If `True`, arguments of different types are cached separately (e.g., `f(3)` vs. `f(3.0)`).
- **Hashable arguments required:** Because the cache is backed by a dictionary, all arguments passed to a cached function must be **hashable** (passing a `list` or `dict` raises `TypeError: unhashable type`).
- **Best for pure functions:** Caching functions with side effects or time-dependent outputs can return stale results and suppress expected side effects.

---

## 18. Pure Functions and Side Effects

### Pure Functions

A function is **pure** if it satisfies two conditions:

1. **Deterministic:** Given the same inputs, it always produces the exact same output.
2. **No side effects:** It does not alter any state outside itself or interact with the external world (no mutating input arguments, no modifying global variables, no I/O such as printing, reading files, or making network calls).

```python
# PURE function
def calculate_discount(price, rate):
    return price * (1 - rate)


# PURE function that returns a new list instead of mutating the input
def add_item_pure(items, new_item):
    return [*items, new_item]
```

### Side Effects

A **side effect** is any observable interaction a function has outside returning a value:

- Modifying a global variable or class attribute.
- Mutating a mutable argument passed in (e.g., `items.append(...)` or `items.sort()`).
- Printing to stdout, writing to a file, updating a database, or sending HTTP requests.
- Depending on hidden/external mutable state (current time, random numbers, global config).

```python
# IMPURE function (mutates the input list AND reads/writes external state)
log = []


def add_item_impure(items, new_item):
    items.append(new_item)          # Side effect 1: mutates caller's list
    log.append(f"Added {new_item}") # Side effect 2: mutates global state
    print(f"Added {new_item}")      # Side effect 3: console I/O
```

### Why Prefer Pure Functions Where Practical?

- **Easy to test:** No mocking of globals, databases, or files needed — just assert `func(input) == expected`.
- **Easy to reason about:** Behavior depends solely on explicit arguments.
- **Safe to cache & parallelize:** Referential transparency means calls can be memoized (`@lru_cache`) or run concurrently without race conditions.

*(Note: Useful programs eventually need side effects — e.g., displaying output or saving files. A clean design pushes pure logic into small, testable functions and isolates side effects at the boundaries of the application.)*
