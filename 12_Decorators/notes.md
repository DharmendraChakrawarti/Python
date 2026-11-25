# Decorators in Python

Decorators are one of Python's most expressive features. They let you modify or extend the behavior of functions, methods, and classes cleanly—without changing their underlying source code.

---

## Table of Contents

1. [What Are Decorators? Concept and Motivation](#1-what-are-decorators-concept-and-motivation)
2. [Functions as First-Class Objects (Prerequisite)](#2-functions-as-first-class-objects-prerequisite)
3. [Simple Decorator (Without Arguments)](#3-simple-decorator-without-arguments)
4. [Decorating Functions with Arguments](#4-decorating-functions-with-arguments)
5. [`@functools.wraps` — Preserving Function Metadata](#5-functoolswraps--preserving-function-metadata)
6. [Decorators with Arguments (Decorator Factories)](#6-decorators-with-arguments-decorator-factories)
7. [Multiple Decorators (Stacking / Chaining)](#7-multiple-decorators-stacking--chaining)
8. [Class Decorators](#8-class-decorators)
9. [Built-in Decorators (`@staticmethod`, `@classmethod`, `@property`)](#9-built-in-decorators-staticmethod-classmethod-property)
10. [`@dataclass` Decorator](#10-dataclass-decorator)
11. [Real-World Decorator Examples](#11-real-world-decorator-examples)
12. [Decorators vs. Other Patterns](#12-decorators-vs-other-patterns)
13. [Creating a Decorator Class Using `__call__`](#13-creating-a-decorator-class-using-__call__)

---

## 1. What Are Decorators? Concept and Motivation

A **decorator** is a callable (usually a function or a class) that takes another function or class as an argument, adds or alters behavior, and returns a replacement callable (or modified class).

### Why Use Decorators?

In real applications, you often have **cross-cutting concerns**—behavior needed across many functions that isn't part of their core business logic:

- Logging function calls and return values
- Measuring execution time
- Enforcing authentication or permissions
- Caching / memoizing expensive results
- Retrying failed network calls
- Validating inputs

**Without decorators**, you would have to duplicate boilerplate inside every function:

```python
import time


def fetch_users():
    start = time.perf_counter()
    print("Calling fetch_users...")
    # Core logic
    users = ["Alice", "Bob"]
    elapsed = time.perf_counter() - start
    print(f"fetch_users finished in {elapsed:.4f}s")
    return users
```

**With decorators**, you separate the cross-cutting concern from the core logic:

```python
@log_and_time
def fetch_users():
    return ["Alice", "Bob"]
```

### Syntactic Sugar

The `@decorator` syntax is pure syntactic sugar for reassigning the function name:

```python
@my_decorator
def greet():
    print("Hello!")
```

is **exact shorthand** for:

```python
def greet():
    print("Hello!")

greet = my_decorator(greet)
```

> **Key Insight:** Decoration happens at **definition time** (when Python executes the `def` or `class` statement), not when the decorated function is called—though the wrapper runs at call time.

---

## 2. Functions as First-Class Objects (Prerequisite)

Decorators work in Python because functions are **first-class objects** (just like integers, strings, or lists). That means functions can be:

1. **Assigned to variables**
2. **Passed as arguments** to other functions
3. **Returned from** other functions
4. **Defined inside** other functions (nested functions + **closures**)

### 2.1 Assigning Functions to Variables

```python
def shout(text: str) -> str:
    return text.upper()


yell = shout  # Reference to the function object (no parentheses)
print(yell("hello"))  # Output: HELLO
```

### 2.2 Passing Functions as Arguments

```python
def apply(func, value):
    return func(value)


print(apply(shout, "python"))  # Output: PYTHON
```

### 2.3 Defining and Returning Inner Functions (Closures)

A **closure** occurs when a nested function remembers variables from its enclosing scope even after the outer function has finished executing.

```python
def make_multiplier(factor: int):
    def multiply(number: int) -> int:
        # `factor` is captured from the enclosing scope
        return number * factor

    return multiply


double = make_multiplier(2)
triple = make_multiplier(3)

print(double(5))  # Output: 10
print(triple(5))  # Output: 15
```

Every decorator relies on this exact mechanism: an outer function accepts `func`, defines an inner `wrapper` that closes over `func`, and returns `wrapper`.

---

## 3. Simple Decorator (Without Arguments)

Here is a minimal decorator that prints messages before and after a zero-argument function runs.

### Manual Application vs. `@` Syntax

```python
def announce(func):
    def wrapper():
        print(f"--> Before calling {func.__name__}")
        result = func()
        print(f"<-- After calling {func.__name__}")
        return result

    return wrapper


# Using @ syntax
@announce
def say_hello():
    print("Hello, world!")


say_hello()
```

**Output:**

```text
--> Before calling say_hello
Hello, world!
<-- After calling say_hello
```

### Step-by-Step Execution

1. Python defines `say_hello`.
2. Python evaluates `announce(say_hello)`, which creates and returns `wrapper` (holding a reference to the original `say_hello`).
3. Python binds the name `say_hello` to `wrapper`.
4. Calling `say_hello()` actually invokes `wrapper()`, which runs the extra logic, calls the original `func()`, and returns `result`.

> **Tip:** Always `return result` from `wrapper` unless you intentionally want to discard or replace the decorated function's return value.

---

## 4. Decorating Functions with Arguments

The simple `wrapper()` above fails with `TypeError` if the decorated function takes any arguments:

```python
@announce
def add(a, b):
    return a + b

# add(2, 3)  # TypeError: wrapper() takes 0 positional arguments but 2 were given
```

### Solution: `*args` and `**kwargs`

By accepting `*args, **kwargs` in `wrapper` and forwarding them to `func(*args, **kwargs)`, a single decorator works for functions with **any** signature (positional, keyword, default values, or none):

```python
def trace(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result!r}")
        return result

    return wrapper


@trace
def add(a: int, b: int = 0) -> int:
    return a + b


print(add(3, b=4))
```

**Output:**

```text
Calling add with args=(3,), kwargs={'b': 4}
add returned 7
7
```

---

## 5. `@functools.wraps` — Preserving Function Metadata

When you replace a function with `wrapper`, the decorated function loses its original identity and metadata (`__name__`, `__doc__`, `__annotations__`, `__module__`, `__wrapped__`).

### The Problem

```python
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        """Wrapper docstring."""
        return func(*args, **kwargs)

    return wrapper


@bad_decorator
def multiply(x: int, y: int) -> int:
    """Multiply two integers."""
    return x * y


print(multiply.__name__)  # Output: wrapper   (lost original name!)
print(multiply.__doc__)   # Output: Wrapper docstring.
```

This breaks debugging, stack traces, `help()`, documentation generators, and frameworks (like Flask, FastAPI, or `pytest`) that inspect function names and signatures.

### The Fix: `@functools.wraps(func)`

`functools.wraps` is itself a decorator applied to `wrapper`. It copies `__name__`, `__doc__`, `__qualname__`, `__annotations__`, `__module__`, and sets `__wrapped__` to the original function (so `inspect.signature()` works accurately too).

```python
from functools import wraps


def good_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        """Wrapper docstring."""
        return func(*args, **kwargs)

    return wrapper


@good_decorator
def multiply(x: int, y: int) -> int:
    """Multiply two integers."""
    return x * y


print(multiply.__name__)       # Output: multiply
print(multiply.__doc__)        # Output: Multiply two integers.
print(multiply.__wrapped__)    # Output: <function multiply at 0x...>
```

> **Best Practice:** Always decorate your inner `wrapper` with `@wraps(func)`.

---

## 6. Decorators with Arguments (Decorator Factories)

Sometimes you want to configure a decorator when you apply it—for example:

```python
@repeat(times=3)
def greet(name):
    print(f"Hi, {name}!")
```

To support arguments, add **one more level of nesting**. The outer function is a **decorator factory**: it accepts the configuration arguments and returns an actual decorator.

### Equivalent Desugared Form

```python
greet = repeat(times=3)(greet)
```

1. `repeat(times=3)` returns `decorator`.
2. `decorator(greet)` returns `wrapper`.
3. `greet` is bound to `wrapper`.

### Implementation

```python
from functools import wraps


def repeat(times: int = 2):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = func(*args, **kwargs)
            return result

        return wrapper

    return decorator


@repeat(times=3)
def greet(name: str) -> None:
    print(f"Hello, {name}!")


greet("Alice")
```

**Output:**

```text
Hello, Alice!
Hello, Alice!
Hello, Alice!
```

### Optional-Argument Decorator Pattern

What if you want a decorator to work **both** as `@my_dec` and `@my_dec(prefix="LOG")`? You can inspect whether the first argument is callable:

```python
from functools import wraps


def tag(func=None, *, prefix="INFO"):
    def decorator(fn):
        @wraps(fn)
        def wrapper(*args, **kwargs):
            print(f"[{prefix}] Running {fn.__name__}")
            return fn(*args, **kwargs)

        return wrapper

    if func is not None:
        # Used as @tag without parentheses
        return decorator(func)
    # Used as @tag(prefix="DEBUG") or @tag()
    return decorator


@tag
def task_a():
    print("Task A")


@tag(prefix="DEBUG")
def task_b():
    print("Task B")


task_a()
task_b()
```

**Output:**

```text
[INFO] Running task_a
Task A
[DEBUG] Running task_b
Task B
```

---

## 7. Multiple Decorators (Stacking / Chaining)

You can apply multiple decorators to a single function by stacking them one above another.

### Order of Application vs. Order of Execution

```python
@decorator_a
@decorator_b
def my_func():
    ...
```

This is equivalent to:

```python
my_func = decorator_a(decorator_b(my_func))
```

- **Wrapping (decoration) order:** **Bottom-up** (`decorator_b` wraps `my_func` first, then `decorator_a` wraps the result).
- **Call-time execution order:** **Top-down** on the way in (`decorator_a`'s wrapper runs first, calls `decorator_b`'s wrapper, which calls `my_func`), and **bottom-up** on the way out.

### Example

```python
from functools import wraps


def make_bold(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"

    return wrapper


def make_italic(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"

    return wrapper


@make_bold
@make_italic
def get_text(msg: str) -> str:
    return msg


print(get_text("Hello"))  # Output: <b><i>Hello</i></b>
```

Notice how `make_italic` runs closest to `"Hello"` (`<i>Hello</i>`), and `make_bold` wraps that outer string (`<b><i>Hello</i></b>`).

---

## 8. Class Decorators

Decorators aren't limited to functions—they can also decorate **classes**. A class decorator takes a `cls` object, modifies or wraps it, and returns a class (often the same `cls` after mutating it, or a new subclass/wrapper).

### 8.1 Adding Methods or Attributes to a Class

```python
def add_repr(cls):
    def __repr__(self):
        attrs = ", ".join(f"{k}={v!r}" for k, v in self.__dict__.items())
        return f"{cls.__name__}({attrs})"

    cls.__repr__ = __repr__
    return cls


@add_repr
class Point:
    def __init__(self, x: int, y: int):
        self.x = x
        self.y = y


p = Point(3, 4)
print(p)  # Output: Point(x=3, y=4)
```

### 8.2 Registering Classes Automatically (Plugin Registry)

```python
EXPORTERS = {}


def register_exporter(format_name: str):
    def decorator(cls):
        EXPORTERS[format_name] = cls
        return cls

    return decorator


@register_exporter("csv")
class CSVExporter:
    def export(self, data):
        return "CSV data"


@register_exporter("json")
class JSONExporter:
    def export(self, data):
        return "JSON data"


print(EXPORTERS)
# {'csv': <class '__main__.CSVExporter'>, 'json': <class '__main__.JSONExporter'>}
```

### 8.3 Enforcing the Singleton Pattern

```python
from functools import wraps


def singleton(cls):
    instances = {}

    @wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return get_instance


@singleton
class AppConfig:
    def __init__(self, env: str = "prod"):
        self.env = env


c1 = AppConfig("dev")
c2 = AppConfig("prod")
print(c1 is c2)  # Output: True
print(c2.env)    # Output: dev
```

---

## 9. Built-in Decorators (`@staticmethod`, `@classmethod`, `@property`)

Python ships with three essential built-in decorators for object-oriented programming.

| Decorator | First Parameter | Bound To | Common Use Case |
| :--- | :--- | :--- | :--- |
| *(regular method)* | `self` | Instance | Operates on instance attributes |
| `@classmethod` | `cls` | Class | Alternative constructors, class-wide state |
| `@staticmethod` | *(none)* | Neither | Utility functions logically grouped with the class |
| `@property` | `self` | Instance | Managed attributes (computed values, validation) |

### 9.1 `@staticmethod`

Behaves like a regular function placed inside the class namespace. It receives neither `self` nor `cls`, and can be called on the class or an instance.

```python
class MathUtils:
    @staticmethod
    def is_even(n: int) -> bool:
        return n % 2 == 0


print(MathUtils.is_even(4))    # Output: True
print(MathUtils().is_even(5))  # Output: False
```

### 9.2 `@classmethod`

Receives the class (`cls`) rather than an instance (`self`). Because `cls` is passed dynamically, classmethods work cleanly with inheritance.

```python
from datetime import date


class Person:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

    @classmethod
    def from_birth_year(cls, name: str, birth_year: int):
        current_year = date.today().year
        return cls(name, current_year - birth_year)


alice = Person.from_birth_year("Alice", 1998)
print(alice.name, alice.age)
```

### 9.3 `@property`

Turns a method into a **managed attribute** using Python's descriptor protocol, allowing getter, setter, and deleter logic behind normal `obj.attr` access.

```python
class Temperature:
    def __init__(self, celsius: float):
        self.celsius = celsius  # Triggers the setter for validation

    @property
    def celsius(self) -> float:
        return self._celsius

    @celsius.setter
    def celsius(self, value: float) -> None:
        if value < -273.15:
            raise ValueError("Temperature cannot be below absolute zero (-273.15°C).")
        self._celsius = value

    @property
    def fahrenheit(self) -> float:
        """Read-only computed property."""
        return (self._celsius * 9 / 5) + 32


t = Temperature(25)
print(t.celsius)     # Output: 25
print(t.fahrenheit)  # Output: 77.0

t.celsius = 100
print(t.fahrenheit)  # Output: 212.0
# t.celsius = -300   # Raises ValueError
```

---

## 10. `@dataclass` Decorator

Introduced in Python 3.7 (`dataclasses` module), `@dataclass` is a **class decorator** that inspects type annotations on a class and automatically generates boilerplate special methods: `__init__`, `__repr__`, `__eq__`, and optionally `__lt__`/`__le__`/`__gt__`/`__ge__`, `__hash__`, etc.

### Basic Example

```python
from dataclasses import dataclass, field


@dataclass
class Product:
    name: str
    price: float
    quantity: int = 1
    tags: list[str] = field(default_factory=list)

    def total_cost(self) -> float:
        return self.price * self.quantity


item = Product("Laptop", 999.99, 2, ["electronics"])
print(item)               # Product(name='Laptop', price=999.99, quantity=2, tags=['electronics'])
print(item.total_cost())  # 1999.98
```

### Key `@dataclass` Options

| Option | Default | Effect |
| :--- | :--- | :--- |
| `frozen=True` | `False` | Makes instances immutable (raises `FrozenInstanceError` on attribute assignment) and generates `__hash__` |
| `order=True` | `False` | Generates `__lt__`, `__le__`, `__gt__`, `__ge__` based on field order |
| `slots=True` | `False` | *(Python 3.10+)* Generates `__slots__` for lower memory usage and faster attribute access |
| `kw_only=True` | `False` | *(Python 3.10+)* Makes all fields keyword-only in `__init__` |

```python
@dataclass(frozen=True, order=True)
class Version:
    major: int
    minor: int
    patch: int = 0


v1 = Version(1, 2, 0)
v2 = Version(1, 3, 0)
print(v1 < v2)  # Output: True
# v1.major = 2  # Raises dataclasses.FrozenInstanceError
```

> **Watch Out:** Never use a mutable literal like `tags: list[str] = []` directly in a dataclass—use `field(default_factory=list)` so each instance gets its own list.

---

## 11. Real-World Decorator Examples

### 11.1 Timing Execution (`@timer`)

```python
import time
from functools import wraps


def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        try:
            return func(*args, **kwargs)
        finally:
            elapsed = time.perf_counter() - start
            print(f"[TIMER] {func.__name__} took {elapsed:.6f}s")

    return wrapper


@timer
def sum_squares(n: int) -> int:
    return sum(i * i for i in range(n))


sum_squares(1_000_000)
```

Using `try ... finally` ensures the elapsed time is still printed even if `func` raises an exception.

---

### 11.2 Structured Logging (`@log_call`)

```python
import logging
from functools import wraps

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def log_call(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        args_repr = [repr(a) for a in args]
        kwargs_repr = [f"{k}={v!r}" for k, v in kwargs.items()]
        signature = ", ".join(args_repr + kwargs_repr)
        logger.info("Calling %s(%s)", func.__name__, signature)
        try:
            result = func(*args, **kwargs)
            logger.info("%s returned %r", func.__name__, result)
            return result
        except Exception as exc:
            logger.exception("%s raised %s", func.__name__, type(exc).__name__)
            raise

    return wrapper


@log_call
def divide(a: float, b: float) -> float:
    return a / b


divide(10, 2)
```

---

### 11.3 Authentication / Authorization (`@require_role`)

```python
from functools import wraps


class PermissionDeniedError(Exception):
    pass


def require_role(required_role: str):
    def decorator(func):
        @wraps(func)
        def wrapper(user: dict, *args, **kwargs):
            if not user.get("is_authenticated"):
                raise PermissionDeniedError("Authentication required.")
            if required_role not in user.get("roles", []):
                raise PermissionDeniedError(
                    f"User {user.get('username')!r} lacks role {required_role!r}."
                )
            return func(user, *args, **kwargs)

        return wrapper

    return decorator


@require_role("admin")
def delete_database(user: dict, db_name: str) -> str:
    return f"Database {db_name!r} deleted by {user['username']}."


admin = {"username": "alice", "is_authenticated": True, "roles": ["admin"]}
guest = {"username": "bob", "is_authenticated": True, "roles": ["viewer"]}

print(delete_database(admin, "staging"))
# delete_database(guest, "staging")  # Raises PermissionDeniedError
```

---

### 11.4 Memoization / Caching (`@memoize` and `@functools.lru_cache`)

Memoization caches the results of expensive pure functions keyed by their arguments.

#### Custom `@memoize` Decorator

```python
from functools import wraps


def memoize(func):
    cache = {}

    @wraps(func)
    def wrapper(*args, **kwargs):
        # Convert kwargs to a sorted tuple so the key is hashable
        key = (args, tuple(sorted(kwargs.items())))
        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]

    wrapper.cache = cache  # Expose cache for inspection/clearing if needed
    return wrapper


@memoize
def fib(n: int) -> int:
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)


print(fib(35))  # Instant instead of exponential!
```

#### Built-in `@functools.lru_cache` / `@functools.cache`

In production code, prefer Python's built-in `functools.lru_cache` (bounded LRU cache, thread-safe, provides `.cache_info()` and `.cache_clear()`) or `functools.cache` (Python 3.9+, unbounded):

```python
from functools import lru_cache


@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)


print(fibonacci(50))
print(fibonacci.cache_info())
```

---

### 11.5 Retry with Exponential Backoff (`@retry`)

Useful for transient failures like flaky network requests or database lock contention.

```python
import time
from functools import wraps


def retry(
    max_attempts: int = 3,
    delay: float = 0.5,
    backoff: float = 2.0,
    exceptions: tuple[type[Exception], ...] = (Exception,),
):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            current_delay = delay
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as exc:
                    if attempt == max_attempts:
                        raise
                    print(
                        f"Attempt {attempt}/{max_attempts} failed ({exc}). "
                        f"Retrying in {current_delay:.2f}s..."
                    )
                    time.sleep(current_delay)
                    current_delay *= backoff

        return wrapper

    return decorator


attempts = 0


@retry(max_attempts=3, delay=0.1, backoff=2.0, exceptions=(ConnectionError,))
def fetch_remote_data():
    global attempts
    attempts += 1
    if attempts < 3:
        raise ConnectionError("Temporary network glitch")
    return {"status": "ok"}


print(fetch_remote_data())
```

**Output:**

```text
Attempt 1/3 failed (Temporary network glitch). Retrying in 0.10s...
Attempt 2/3 failed (Temporary network glitch). Retrying in 0.20s...
{'status': 'ok'}
```

---

## 12. Decorators vs. Other Patterns

Decorators are not always the right tool. Here is how they compare with related Python mechanisms and design patterns:

| Pattern | When It Applies | Strengths | Limitations / When Not to Use |
| :--- | :--- | :--- | :--- |
| **Decorators** | Wrapping functions, methods, or classes at **definition time** | Declarative `@` syntax, composable, zero changes to function body | Static per-function; harder to toggle dynamically per call without passing flags |
| **Context Managers (`with`)** | Wrapping a **block of code** with setup & teardown (files, locks, DB transactions) | Scoped to specific statements inside a function; deterministic resource cleanup | Doesn't automatically wrap every call to a function from outside |
| **Higher-Order Functions** | Passing behavior explicitly at **call time** (e.g., `sorted(items, key=...)`, `map`) | Explicit, flexible per invocation | No automatic wrapping across all call sites |
| **Inheritance / Mixins** | Extending or overriding behavior of **classes** | Clear OOP hierarchy, `super()` support, easy to inspect with `isinstance` | Can cause deep hierarchies or MRO complexity if overused |
| **Metaclasses** | Customizing **class creation itself** across an entire hierarchy | Applies automatically to all subclasses | Much more complex and magical than class decorators |
| **Python Decorator vs. GoF Decorator Pattern** | Both add responsibilities dynamically | Python's `@decorator` wraps at definition time; GoF Decorator wraps an object instance at runtime | Don't confuse the two: GoF Decorator uses object composition at runtime (`BufferedReader(FileReader(...))`) |

### Rule of Thumb

- Wrap **every call** to a function/class cleanly → **Decorator**.
- Manage **setup/teardown around a code block** (e.g., timing one loop, opening a connection) → **Context Manager** (`with`). (Note: `contextlib.ContextDecorator` lets a single class act as *both*!)
- Customize **a class without needing automatic inheritance propagation** → **Class Decorator** instead of a metaclass.

---

## 13. Creating a Decorator Class Using `__call__`

Any object whose class implements `__call__` is callable. Class-based decorators are especially clean when your decorator needs to **maintain state** across calls (like call counts, rate-limit timestamps, or configuration) or break complex logic into methods.

### 13.1 Class Decorator Without Arguments

```python
from functools import update_wrapper


class CallCounter:
    def __init__(self, func):
        self.func = func
        self.count = 0
        # Preserves metadata (__name__, __doc__, __wrapped__, etc.)
        update_wrapper(self, func)

    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"{self.func.__name__} has been called {self.count} time(s).")
        return self.func(*args, **kwargs)


@CallCounter
def say_hi(name: str) -> str:
    return f"Hi, {name}!"


print(say_hi("Alice"))
print(say_hi("Bob"))
print("Total calls:", say_hi.count)
```

**Output:**

```text
say_hi has been called 1 time(s).
Hi, Alice!
say_hi has been called 2 time(s).
Hi, Bob!
Total calls: 2
```

### Caveat: Decorating Methods Inside a Class

If you apply a class-based decorator (like `CallCounter` above) to a **method inside a class**, `self.func` receives no bound instance `self` unless your decorator implements the **descriptor protocol** (`__get__`), because plain Python objects do not bind `self` automatically the way regular functions do.

```python
from functools import update_wrapper
from types import MethodType


class CallCounter:
    def __init__(self, func):
        self.func = func
        self.count = 0
        update_wrapper(self, func)

    def __call__(self, *args, **kwargs):
        self.count += 1
        return self.func(*args, **kwargs)

    def __get__(self, instance, owner):
        if instance is None:
            return self
        # Bind `self` (CallCounter) as a method of `instance`
        return MethodType(self, instance)


class Greeter:
    @CallCounter
    def greet(self, name: str) -> str:
        return f"Welcome, {name}!"


g = Greeter()
print(g.greet("Sam"))          # Output: Welcome, Sam!
print(Greeter.greet.count)     # Output: 1
```

### 13.2 Class Decorator With Arguments

When a decorator class accepts arguments:

- `__init__` receives the **decorator arguments** (not `func`).
- `__call__` receives `func` and returns the `wrapper` function.

```python
from functools import wraps


class RateLimit:
    def __init__(self, max_calls: int):
        self.max_calls = max_calls

    def __call__(self, func):
        calls = 0

        @wraps(func)
        def wrapper(*args, **kwargs):
            nonlocal calls
            if calls >= self.max_calls:
                raise RuntimeError(
                    f"{func.__name__} exceeded limit of {self.max_calls} calls."
                )
            calls += 1
            return func(*args, **kwargs)

        return wrapper


@RateLimit(max_calls=2)
def ping():
    return "pong"


print(ping())  # pong
print(ping())  # pong
# print(ping())  # Raises RuntimeError: ping exceeded limit of 2 calls.
```

Because `RateLimit.__call__` returns a regular function (`wrapper`), it automatically works on both standalone functions and class methods without needing `__get__`.

---

## Summary Checklist

- **Functions are first-class objects** → decorators wrap callables using closures.
- Use `*args, **kwargs` in `wrapper` to support any function signature.
- Always apply **`@functools.wraps(func)`** (or `functools.update_wrapper`) to preserve metadata.
- Add an outer factory function to pass **arguments to a decorator**.
- Stacked decorators wrap **bottom-up** and execute **top-down**.
- Leverage built-ins (`@staticmethod`, `@classmethod`, `@property`, `@dataclass`, `@functools.lru_cache`) before writing custom boilerplate.
- Use a **class with `__call__`** when stateful decorators or clean OOP organization benefit from it (and add `__get__` if instances wrap class methods directly).
