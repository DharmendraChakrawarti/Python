# Exception Handling in Python

Exception handling lets a program detect, respond to, and recover from unexpected runtime conditions gracefully instead of crashing with an unhandled traceback.

---

## Table of Contents

1. [What Are Exceptions? Errors vs Exceptions](#1-what-are-exceptions-errors-vs-exceptions)
2. [Common Built-in Exceptions](#2-common-built-in-exceptions)
3. [The `try-except` Block](#3-the-try-except-block)
4. [Multiple `except` Blocks](#4-multiple-except-blocks)
5. [Catching Multiple Exceptions in One Block](#5-catching-multiple-exceptions-in-one-block)
6. [The `try-except-else` Block](#6-the-try-except-else-block)
7. [The `try-except-finally` Block](#7-the-try-except-finally-block)
8. [The `raise` Statement](#8-the-raise-statement)
9. [Re-raising Exceptions](#9-re-raising-exceptions)
10. [Custom Exception Classes](#10-custom-exception-classes)
11. [Exception Hierarchy (`BaseException` Tree)](#11-exception-hierarchy-baseexception-tree)
12. [Exception Chaining (`raise ... from ...`)](#12-exception-chaining-raise--from-)
13. [The `assert` Statement](#13-the-assert-statement)
14. [Logging Exceptions](#14-logging-exceptions)
15. [Best Practices for Exception Handling](#15-best-practices-for-exception-handling)
16. [EAFP vs LBYL](#16-eafp-vs-lbyl)
17. [Context Managers and Exceptions](#17-context-managers-and-exceptions)
18. [The `warnings` Module](#18-the-warnings-module)

---

## 1. What Are Exceptions? Errors vs Exceptions

An **exception** is an event raised during program execution that disrupts the normal flow of instructions. When an error occurs at runtime, Python creates an exception object and propagates it up the call stack until an appropriate handler catches it—or the interpreter terminates the program and prints a traceback.

In Python terminology, people often distinguish between **syntax errors** (compile-time/parse-time issues) and **runtime exceptions**:

| Aspect | Syntax Errors (`SyntaxError` / `IndentationError`) | Runtime Exceptions (e.g., `ValueError`, `ZeroDivisionError`) |
| --- | --- | --- |
| **When detected** | During parsing/compilation, *before* the code runs | During execution, when a specific line runs |
| **Cause** | Invalid Python grammar, missing colons, bad indentation | Valid syntax encountering invalid data, state, or environment |
| **Can `try-except` catch it in the same file?** | No (the file fails to parse before `try` runs) | Yes |
| **Note** | `SyntaxError` is still a subclass of `Exception`, so it *can* be caught when dynamically executing code with `eval()`, `exec()`, or `compile()`. | Handled directly with `try-except`. |

```python
# SyntaxError: detected before execution begins
# if True
#     print("Missing colon")

# Runtime Exception: syntax is valid, but execution fails
result = 10 / 0  # Raises ZeroDivisionError: division by zero
```

---

## 2. Common Built-in Exceptions

Python provides a rich set of built-in exception classes for standard failure modes:

| Exception | When It Occurs | Example |
| --- | --- | --- |
| `ValueError` | Right type, inappropriate value | `int("abc")` |
| `TypeError` | Operation applied to an incompatible type | `"age: " + 25` |
| `KeyError` | Dictionary key not found | `{"a": 1}["b"]` |
| `IndexError` | Sequence index out of range | `[1, 2, 3][5]` |
| `FileNotFoundError` | Trying to open a file/directory that does not exist | `open("missing.txt")` |
| `PermissionError` | Insufficient OS permissions for a file/resource | Writing to a read-only system path |
| `ZeroDivisionError` | Division or modulo by zero | `5 / 0` or `5 % 0` |
| `AttributeError` | Attribute reference or assignment fails | `"hello".non_existent()` |
| `ImportError` | An `import` statement fails to load a module or symbol | `from math import missing_fn` |
| `ModuleNotFoundError` | Subclass of `ImportError`; module itself cannot be located | `import no_such_package` |
| `NameError` | Local or global name is not defined | `print(undefined_var)` |
| `UnboundLocalError` | Subclass of `NameError`; local variable referenced before assignment | Assigning a variable in a function after reading it |
| `StopIteration` | Raised by `next()` when an iterator has no more items | `next(iter([]))` |
| `OSError` | System-related error (I/O failure, disk full, broken pipe) | Base class for `FileNotFoundError`, `PermissionError` |
| `RuntimeError` | Generic error that doesn't fit other categories | Modifying a `dict` size while iterating over it |
| `NotImplementedError` | Subclass of `RuntimeError`; abstract method left unimplemented | Base class placeholder method |
| `RecursionError` | Maximum recursion depth exceeded | Infinite recursive call |
| `MemoryError` | Operation runs out of memory | Allocating an impossibly huge list |

```python
# Quick demonstrations of common exceptions
# ValueError:
int("42a")

# TypeError:
len(123)

# KeyError:
user = {"name": "Ada"}
_ = user["email"]

# IndexError:
nums = [10, 20]
_ = nums[2]

# AttributeError:
nums.push(30)  # lists have .append(), not .push()
```

---

## 3. The `try-except` Block

Wrap risky code in a `try` block and handle potential exceptions in an `except` block.

### Syntax

```python
try:
    # Code that might raise an exception
    ...
except SomeException as err:
    # Code that runs if SomeException is raised
    ...
```

### Example

```python
def parse_age(raw: str) -> int | None:
    try:
        age = int(raw)
        return age
    except ValueError as exc:
        print(f"Invalid age {raw!r}: {exc}")
        return None


print(parse_age("28"))   # 28
print(parse_age("abc"))  # Invalid age 'abc': invalid literal for int() with base 10: 'abc' -> None
```

### Inspecting the Exception Object

Using `as exc` binds the caught exception instance to `exc`:

- `str(exc)` — human-readable error message
- `type(exc).__name__` — exact exception class name (useful when catching a base class)
- `exc.args` — tuple of arguments passed when the exception was instantiated

```python
try:
    open("non_existent_file.txt", encoding="utf-8")
except OSError as exc:
    print("Type:", type(exc).__name__)  # FileNotFoundError
    print("Message:", exc)              # [Errno 2] No such file or directory: 'non_existent_file.txt'
    print("Args:", exc.args)            # (2, 'No such file or directory')
```

---

## 4. Multiple `except` Blocks

A single `try` block can have multiple `except` clauses to handle different failure modes differently. Python evaluates `except` clauses **from top to bottom** and executes only the **first** matching handler.

> **Important:** Always place **more specific** exception classes before **more general** base classes (e.g., `FileNotFoundError` before `OSError`, or `ZeroDivisionError` before `ArithmeticError` or `Exception`).

```python
def divide_file_value(path: str, divisor: int) -> float | None:
    try:
        with open(path, encoding="utf-8") as f:
            value = int(f.read().strip())
        return value / divisor
    except FileNotFoundError:
        print(f"File not found: {path}")
    except PermissionError:
        print(f"Permission denied reading: {path}")
    except OSError as exc:
        # Catches other OS-level errors (e.g., IsADirectoryError)
        print(f"OS error: {exc}")
    except ValueError:
        print("File contents could not be converted to an integer.")
    except ZeroDivisionError:
        print("Divisor cannot be zero.")
    return None
```

If `OSError` were placed *above* `FileNotFoundError`, the `FileNotFoundError` handler would become unreachable because `FileNotFoundError` is a subclass of `OSError`.

---

## 5. Catching Multiple Exceptions in One Block

When multiple exception types require the exact same recovery logic, group them in a **tuple**:

```python
def get_int_from_mapping(data: dict[str, str], key: str) -> int | None:
    try:
        return int(data[key])
    except (KeyError, ValueError, TypeError) as exc:
        print(f"Failed to read integer for {key!r} ({type(exc).__name__}): {exc}")
        return None


print(get_int_from_mapping({"port": "8080"}, "port"))     # 8080
print(get_int_from_mapping({}, "port"))                   # KeyError handled
print(get_int_from_mapping({"port": "bad"}, "port"))      # ValueError handled
```

> **Note:** Parentheses are required around the tuple: `except (KeyError, ValueError) as exc:`. Writing `except KeyError, ValueError:` is a `SyntaxError` in Python 3.

### Exception Groups (`except*`, Python 3.11+)

When concurrent code raises multiple exceptions simultaneously (via `ExceptionGroup` / `BaseExceptionGroup`), Python 3.11+ supports `except*` to handle subsets of grouped exceptions:

```python
try:
    raise ExceptionGroup("batch errors", [ValueError("bad val"), TypeError("bad type")])
except* ValueError as eg:
    print("Handled ValueErrors:", eg.exceptions)
except* TypeError as eg:
    print("Handled TypeErrors:", eg.exceptions)
```

---

## 6. The `try-except-else` Block

The `else` block runs **only if the `try` block completed without raising any exception** (and did not exit early via `return`, `break`, or `continue`).

```python
try:
    raw = input("Enter a number: ")
    num = int(raw)
except ValueError:
    print("That was not a valid integer.")
else:
    # Runs only when int(raw) succeeded
    print(f"Success! Square is {num ** 2}")
```

### Why Use `else` Instead of Putting Everything in `try`?

Putting subsequent logic in `else` prevents you from **accidentally catching** an exception raised by code that was *not* meant to be protected by `except`.

```python
def load_config(path: str) -> dict:
    try:
        f = open(path, encoding="utf-8")
    except FileNotFoundError:
        print("Config missing; using defaults.")
        return {}
    else:
        with f:
            # If process_config() raises KeyError or ValueError, it won't be
            # confused with file-opening errors.
            return parse_config(f.read())
```

---

## 7. The `try-except-finally` Block

The `finally` block **always executes** when leaving the `try` structure, whether:
1. No exception occurred (`try` -> `else` -> `finally`),
2. An exception was caught (`try` -> `except` -> `finally`),
3. An unhandled exception occurred (`try` -> `finally` -> exception propagates), or
4. `return`, `break`, or `continue` was executed inside `try`, `except`, or `else`.

```python
def read_resource(path: str) -> str | None:
    file = None
    try:
        file = open(path, encoding="utf-8")
        return file.read()
    except FileNotFoundError:
        print(f"{path} does not exist.")
        return None
    finally:
        if file is not None:
            file.close()
            print("File closed.")
```

*(Note: For files and locks, a `with` context manager is even cleaner; see [Section 17](#17-context-managers-and-exceptions).)*

### `try-finally` Without `except`

You can omit `except` when you want cleanup to happen while letting any exception propagate to the caller:

```python
lock.acquire()
try:
    do_critical_work()
finally:
    lock.release()
```

### Pitfall: `return` (or `break` / `raise`) Inside `finally`

Avoid placing `return`, `break`, or `continue` inside `finally`. A `return` in `finally` **overrides** any `return` value from `try`/`except` and **silently discards** any active unhandled exception!

```python
def bad_finally() -> int:
    try:
        1 / 0  # Raises ZeroDivisionError
    finally:
        return 42  # Swallows ZeroDivisionError and returns 42!
```

---

## 8. The `raise` Statement

Use `raise` to trigger an exception explicitly when your code detects invalid inputs, violated preconditions, or impossible states.

```python
def withdraw(balance: float, amount: float) -> float:
    if amount <= 0:
        raise ValueError(f"Withdrawal amount must be positive, got {amount}")
    if amount > balance:
        raise RuntimeError(f"Insufficient funds: balance={balance}, requested={amount}")
    return balance - amount
```

You can raise either:
- An **exception instance**: `raise ValueError("invalid input")` (preferred—clear message and state)
- An **exception class**: `raise ValueError` (Python instantiates it with no arguments, equivalent to `raise ValueError()`)

---

## 9. Re-raising Exceptions

When you want to inspect, log, or clean up after an exception and still let callers know it failed, use a **bare `raise`** inside an `except` block.

```python
def process_order(order_id: str) -> None:
    try:
        charge_card(order_id)
    except PaymentError:
        mark_order_failed(order_id)
        raise  # Re-raises the exact original exception with its original traceback
```

### Bare `raise` vs `raise exc`

- **Bare `raise`**: Preserves the original traceback cleanly up to the original point of failure.
- **`raise exc`**: Adds the current `raise exc` line to the traceback frames (slightly noisier, though in Python 3 `__traceback__` is attached to `exc`). Prefer bare `raise` when re-raising the active exception.

---

## 10. Custom Exception Classes

Creating domain-specific exceptions makes APIs clearer and lets callers catch failures from your module selectively without catching unrelated built-in errors.

### Rules for Custom Exceptions

- Inherit from `Exception` (or a relevant subclass like `ValueError`), **never** directly from `BaseException`.
- Provide a module-level base exception (e.g., `AppError`) and inherit specific errors from it so callers can catch either specific errors or any error from your library.

```python
class PaymentError(Exception):
    """Base exception for all payment-related errors."""


class InsufficientFundsError(PaymentError):
    """Raised when an account balance is too low for a transaction."""

    def __init__(self, balance: float, amount: float) -> None:
        self.balance = balance
        self.amount = amount
        super().__init__(
            f"Cannot withdraw {amount:.2f}; available balance is {balance:.2f}."
        )


class GatewayTimeoutError(PaymentError):
    """Raised when the upstream payment provider times out."""


def charge(balance: float, amount: float) -> float:
    if amount > balance:
        raise InsufficientFundsError(balance=balance, amount=amount)
    return balance - amount


try:
    charge(50.0, 75.0)
except InsufficientFundsError as exc:
    print(f"Shortfall: {exc.amount - exc.balance:.2f}")
    print(exc)
except PaymentError as exc:
    print(f"Other payment failure: {exc}")
```

---

## 11. Exception Hierarchy (`BaseException` Tree)

All built-in exceptions inherit from `BaseException`. Understanding this tree explains why **bare `except:` is dangerous** and why **`except Exception:`** should be used when catching general errors.

```text
BaseException
 ├── BaseExceptionGroup
 ├── GeneratorExit
 ├── KeyboardInterrupt
 ├── SystemExit
 └── Exception
      ├── ArithmeticError
      │    ├── FloatingPointError
      │    ├── OverflowError
      │    └── ZeroDivisionError
      ├── AssertionError
      ├── AttributeError
      ├── BufferError
      ├── EOFError
      ├── ExceptionGroup
      ├── ImportError
      │    └── ModuleNotFoundError
      ├── LookupError
      │    ├── IndexError
      │    └── KeyError
      ├── MemoryError
      ├── NameError
      │    └── UnboundLocalError
      ├── OSError
      │    ├── BlockingIOError
      │    ├── ConnectionError (BrokenPipeError, ConnectionAbortedError, ...)
      │    ├── FileExistsError
      │    ├── FileNotFoundError
      │    ├── IsADirectoryError
      │    ├── PermissionError
      │    └── TimeoutError
      ├── ReferenceError
      ├── RuntimeError
      │    ├── NotImplementedError
      │    └── RecursionError
      ├── StopAsyncIteration
      ├── StopIteration
      ├── SyntaxError
      │    └── IndentationError (TabError)
      ├── SystemError
      ├── TypeError
      ├── ValueError
      │    └── UnicodeError (UnicodeDecodeError, UnicodeEncodeError, ...)
      └── Warning (UserWarning, DeprecationWarning, ...)
```

### Why Avoid Bare `except:` and `except BaseException:`?

`KeyboardInterrupt` (Ctrl+C), `SystemExit` (`sys.exit()`), and `GeneratorExit` inherit directly from `BaseException`, **not** `Exception`.
- A bare `except:` catches `KeyboardInterrupt` and `SystemExit`, making a script refuse to stop on Ctrl+C or ignoring `sys.exit()`.
- Use `except Exception:` if you genuinely need a catch-all boundary (for example, at a top-level worker loop or web request handler).

---

## 12. Exception Chaining (`raise ... from ...`)

Exception chaining links a new exception to the underlying exception that caused it, preserving root-cause diagnostics.

### 1. Explicit Chaining (`raise NewError from original_exc`)

Sets `NewError.__cause__ = original_exc`. Python prints:
`The above exception was the direct cause of the following exception:`

```python
class ConfigLoadError(Exception):
    """Raised when application configuration cannot be loaded."""


def load_port(raw_config: dict[str, str]) -> int:
    try:
        return int(raw_config["PORT"])
    except (KeyError, ValueError) as exc:
        raise ConfigLoadError("Invalid or missing PORT setting") from exc
```

### 2. Implicit Chaining (Automatic `__context__`)

If you raise a new exception inside an `except` block *without* `from`, Python automatically attaches the active exception to `__context__` and prints:
`During handling of the above exception, another exception occurred:`

```python
try:
    int("bad")
except ValueError:
    raise RuntimeError("Conversion step failed")
```

### 3. Suppressing Chaining (`raise NewError from None`)

When translating a low-level implementation error into a clean domain error and the original traceback would only leak noise, suppress the context with `from None` (sets `__suppress_context__ = True`):

```python
def find_user_role(roles: dict[str, str], user: str) -> str:
    try:
        return roles[user]
    except KeyError:
        raise LookupError(f"Unknown user: {user}") from None
```

---

## 13. The `assert` Statement

`assert` tests internal assumptions/invariants during development and testing. If the expression evaluates to `False`, Python raises `AssertionError`.

```python
assert condition, "Optional diagnostic message"
```

### Example

```python
def compute_average(values: list[float]) -> float:
    assert len(values) > 0, "compute_average() requires a non-empty list"
    return sum(values) / len(values)
```

### Critical Caveat: `-O` Optimization Flag

When Python runs with `-O` (optimized mode, `python -O script.py`), **`assert` statements are completely stripped out** (and `__debug__` becomes `False`).

Therefore:
- **DO use `assert` for:** internal sanity checks, debugging invariants, and unit tests.
- **DO NOT use `assert` for:** validating user input, checking permissions/authentication, or enforcing business logic—use explicit `if ...: raise ValueError(...)` or `PermissionError(...)` instead.

```python
# BAD: stripped if run with `python -O`
# assert user.is_admin, "Admin required"

# GOOD: always enforced
if not user.is_admin:
    raise PermissionError("Admin privileges required")
```

---

## 14. Logging Exceptions

In real applications, printing errors with `print()` loses stack traces, timestamps, and severity levels. Use the `logging` module instead.

### `logging.exception()`

Call `logger.exception(...)` **inside an `except` block**. It logs at `ERROR` level and automatically appends the full traceback.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
)
logger = logging.getLogger(__name__)


def parse_payload(raw: str) -> int | None:
    try:
        return int(raw)
    except ValueError:
        logger.exception("Failed to parse payload %r", raw)
        return None
```

### Logging with `exc_info=True` at Other Levels

If you want to include a traceback at `WARNING`, `DEBUG`, or `CRITICAL` level, pass `exc_info=True` (or pass the exception instance):

```python
try:
    connect_cache()
except ConnectionError as exc:
    logger.warning("Cache unavailable, continuing without cache", exc_info=True)
```

### Formatting Tracebacks Manually (`traceback` module)

```python
import traceback

try:
    1 / 0
except ZeroDivisionError:
    tb_text = traceback.format_exc()
    # Save or send tb_text to an error tracker
```

---

## 15. Best Practices for Exception Handling

1. **Catch specific exceptions, not everything.**
   Avoid bare `except:` and avoid overly broad `except Exception:` unless you are at a top-level boundary (worker loop, CLI entry point, web handler) and log the error properly.
2. **Never silently swallow exceptions.**
   `except Exception: pass` hides bugs and makes failures impossible to diagnose. At minimum, log it or use `contextlib.suppress(...)` intentionally for expected benign cases (like deleting a file that might already be absent).
3. **Keep `try` blocks as small as possible.**
   Wrap only the specific operation(s) that can raise the expected exception; move post-success logic into `else`.
4. **Do not use exceptions for ordinary control flow** where a simple check is clearer (though EAFP is idiomatic in Python when checking would be redundant or racy).
5. **Preserve tracebacks and context.**
   Use bare `raise` when re-raising, and `raise ... from exc` when translating exceptions.
6. **Clean up resources reliably.**
   Prefer `with` context managers (`open`, locks, DB connections) over manual `try ... finally: close()`.
7. **Write actionable error messages.**
   Include relevant context (bad value, key, path, limits) without leaking secrets (passwords, tokens, PII).
8. **Document exceptions your public functions raise.**
   Mention raised exceptions in docstrings (`Raises:` section) so callers know what to handle.

---

## 16. EAFP vs LBYL

Two contrasting coding styles for handling potential errors:

- **LBYL (Look Before You Leap):** Explicitly check preconditions *before* performing an operation.
- **EAFP (Easier to Ask for Forgiveness than Permission):** Attempt the operation directly and handle the exception if it fails. **EAFP is the standard Pythonic idiom.**

### Comparison Example

```python
# LBYL
if "port" in config:
    port = int(config["port"])
else:
    port = 8000

# EAFP
try:
    port = int(config["port"])
except (KeyError, ValueError):
    port = 8000
```

### Why Python Favors EAFP

1. **Avoids TOCTOU race conditions:** Checking `if os.path.exists(path): open(path)` can still fail if another process deletes the file between the check and `open()`. `try: with open(path)... except FileNotFoundError:` handles both cleanly.
2. **Single lookup / faster happy path:** In dictionaries or attributes, checking `if key in d: val = d[key]` performs two lookups on success; `try: val = d[key]` performs only one. (For simple dict defaults, `d.get(key, default)` is cleaner still.)
3. **Duck typing friendly:** EAFP cares whether an object supports the operation rather than checking `isinstance(...)` up front.

### When LBYL Makes Sense

- When exceptions happen frequently in a hot loop (raising/catching exceptions has overhead if failures are common rather than exceptional).
- When validating user input or preventing side effects before starting an expensive/irreversible operation.

---

## 17. Context Managers and Exceptions

Context managers (`with` statements) guarantee deterministic setup and teardown—even when an exception occurs inside the `with` block.

### How `__exit__` Interacts with Exceptions

A class-based context manager implements `__enter__` and `__exit__`:

```python
class ManagedResource:
    def __enter__(self):
        print("Acquiring resource")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb) -> bool:
        print("Releasing resource")
        if exc_type is not None:
            print(f"Exception inside with-block: {exc_type.__name__}: {exc_val}")
        # Return True to suppress the exception; False/None to let it propagate
        return False


with ManagedResource():
    print("Working...")
    # Even if ValueError is raised here, __exit__ still runs!
```

- If **no exception** occurred, `exc_type`, `exc_val`, and `exc_tb` are all `None`.
- If an **exception** occurred, `__exit__` receives its type, instance, and traceback:
  - Returning a truthy value (`True`) **suppresses** the exception.
  - Returning `False` or `None` **propagates** the exception after cleanup.

### Function-Based Context Managers (`@contextlib.contextmanager`)

Use `try ... finally` around `yield` so cleanup runs even if the body of the `with` block raises:

```python
from contextlib import contextmanager


@contextmanager
def temporary_state(obj: dict, key: str, value: object):
    had_key = key in obj
    old_value = obj.get(key)
    obj[key] = value
    try:
        yield obj
    finally:
        if had_key:
            obj[key] = old_value
        else:
            obj.pop(key, None)
```

### Suppressing Expected Exceptions Cleanly (`contextlib.suppress`)

`contextlib.suppress` replaces empty `try ... except SomeError: pass` blocks with explicit intent:

```python
import os
from contextlib import suppress

with suppress(FileNotFoundError):
    os.remove("temp_cache.json")
```

---

## 18. The `warnings` Module

Warnings notify developers or users about non-fatal issues—such as deprecated APIs, questionable usage, or upcoming breaking changes—**without** halting program execution like an exception.

### Common Warning Categories

All warning classes inherit from `Warning`, which itself inherits from `Exception`:

| Class | Purpose |
| --- | --- |
| `Warning` | Base class for all warning categories |
| `UserWarning` | Default category for `warnings.warn()` |
| `DeprecationWarning` | Deprecated features (ignored by default outside `__main__`) |
| `PendingDeprecationWarning` | Features planned for future deprecation (ignored by default) |
| `FutureWarning` | Changes in behavior affecting end-users (shown by default) |
| `RuntimeWarning` | Dubious runtime behavior (e.g., numeric overflow in libraries) |
| `SyntaxWarning` | Dubious syntax (e.g., `is` comparison with a literal) |
| `ResourceWarning` | Unclosed files/sockets (ignored by default unless `-W` or `dev` mode is enabled) |

### Emitting Warnings

```python
import warnings


def old_api(x: int) -> int:
    warnings.warn(
        "old_api() is deprecated; use new_api() instead.",
        DeprecationWarning,
        stacklevel=2,  # Points the warning to the caller's line, not this line
    )
    return x * 2
```

> **Tip:** Always pass `stacklevel=2` (or higher for deeper wrappers) so Python reports the caller site rather than inside the helper itself.

### Filtering and Controlling Warnings

Filter actions: `"default"`, `"error"`, `"ignore"`, `"always"`, `"module"`, `"once"`.

```python
import warnings

# Turn DeprecationWarnings into real exceptions (great for CI/testing)
warnings.simplefilter("error", DeprecationWarning)

# Silence a specific warning by regex message
warnings.filterwarnings("ignore", message=r".*old_api.*", category=DeprecationWarning)
```

You can also control filters on the CLI with `-W`:
- `python -W error script.py` — treat all warnings as exceptions
- `python -W default script.py` — show all warnings (including `DeprecationWarning`)
- `python -X dev script.py` — enables development mode (shows `DeprecationWarning` and `ResourceWarning`)

### Testing Warnings with `warnings.catch_warnings`

```python
import warnings

with warnings.catch_warnings(record=True) as caught:
    warnings.simplefilter("always")
    old_api(5)

assert len(caught) == 1
assert issubclass(caught[0].category, DeprecationWarning)
assert "deprecated" in str(caught[0].message)
```
