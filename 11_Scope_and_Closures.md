# Scope and Closures in Python

Understanding **scope** and **closures** is essential for writing bug-free Python code, building decorators, creating function factories, and managing state cleanly without relying on global variables.

---

## Table of Contents

1. [LEGB Rule (Local, Enclosing, Global, Built-in)](#1-legb-rule-local-enclosing-global-built-in)
2. [Local Scope](#2-local-scope)
3. [Enclosing Scope (`nonlocal`)](#3-enclosing-scope-nonlocal)
4. [Global Scope (`global` Keyword)](#4-global-scope-global-keyword)
5. [Built-in Scope](#5-built-in-scope)
6. [Variable Shadowing](#6-variable-shadowing)
7. [Closures: Definition and How They Work](#7-closures-definition-and-how-they-work)
8. [Closure Use Cases (Data Hiding, Function Factories)](#8-closure-use-cases-data-hiding-function-factories)
9. [Free Variables](#9-free-variables)
10. [The `__closure__` Attribute](#10-the-__closure__-attribute)
11. [Common Closure Pitfalls (Late Binding & More)](#11-common-closure-pitfalls-late-binding--more)
12. [Practical Examples of Closures](#12-practical-examples-of-closures)

---

## 1. LEGB Rule (Local, Enclosing, Global, Built-in)

**Scope** defines the region of a program where a variable (name) is directly accessible. Python resolves variable names using **lexical (static) scoping**—meaning a variable's scope is determined by where it is written in the source code, not where the function is called.

When you reference a name, Python searches namespaces in a strict hierarchy known as the **LEGB Rule**:

```text
+-------------------------------------------------------+
| Built-in (B) : len, int, print, Exception, ...        |
|   +-----------------------------------------------+   |
|   | Global (G) : Module-level names               |   |
|   |   +---------------------------------------+   |   |
|   |   | Enclosing (E) : Outer function names  |   |   |
|   |   |   +-------------------------------+   |   |   |
|   |   |   | Local (L) : Current function  |   |   |   |
|   |   |   +-------------------------------+   |   |   |
|   |   +---------------------------------------+   |   |
|   +-----------------------------------------------+   |
+-------------------------------------------------------+
```

| Order | Scope | Description |
| :--- | :--- | :--- |
| **1. L** | **Local** | Names assigned inside the current function (`def` or `lambda`), including parameters. |
| **2. E** | **Enclosing** | Names in the local scopes of any enclosing (outer) functions, from inner to outer. |
| **3. G** | **Global** | Names assigned at the top level of a module file, or declared `global` inside a function. |
| **4. B** | **Built-in** | Names preloaded in Python's `builtins` module (`len`, `range`, `ValueError`, etc.). |

If Python fails to find the name in all four scopes, it raises a `NameError`.

```python
x = "global"


def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print("Inside inner:", x)      # Finds 'x' in Local (L)
        print("Built-in len:", len(x)) # Finds 'len' in Built-in (B)

    inner()
    print("Inside outer:", x)          # Finds 'x' in outer's Local (E for inner)


outer()
print("At module level:", x)           # Finds 'x' in Global (G)
```

Output:
```text
Inside inner: local
Built-in len: 5
Inside outer: enclosing
At module level: global
```

> **Note:** Blocks such as `if`, `for`, `while`, `try`, and `with` do **not** create a new scope in Python. Variables assigned inside them belong to the enclosing function or module scope (exception: comprehension variables in Python 3 have their own isolated scope).

---

## 2. Local Scope

A **local scope** is created every time a function is called and is destroyed when the function returns. Variables defined inside a function (including its parameters) exist only in that function's local namespace.

- You can inspect the current local namespace as a dictionary using `locals()`.
- Different calls to the same function get fresh, independent local scopes.

```python
def calculate_area(width, height):
    area = width * height  # 'width', 'height', and 'area' are local
    print("Locals:", locals())
    return area


result = calculate_area(5, 3)
print("Area:", result)

# Accessing 'area' or 'width' outside the function fails:
# print(area)  # NameError: name 'area' is not defined
```

Output:
```text
Locals: {'width': 5, 'height': 3, 'area': 15}
Area: 15
```

### How Python Determines Local Variables at Compile Time

Python decides which variables are local when it **compiles** the function body, before running it. Any name assigned (`=`, `+=`, `for x in ...`, `import`, `def`) anywhere inside a function is treated as local throughout the entire function—unless declared `global` or `nonlocal`.

This causes a classic error called `UnboundLocalError`:

```python
count = 10


def increment_broken():
    # Python sees 'count =' below, so it marks 'count' as LOCAL for the whole function.
    # Reading 'count' before it has been assigned locally fails.
    print(count)  # UnboundLocalError: cannot access local variable 'count' where it is not associated with a value
    count = 20


# increment_broken()
```

---

## 3. Enclosing Scope (`nonlocal`)

When one function is defined inside another function, the outer function's local scope becomes the **enclosing scope** (also called the *non-local* scope) for the inner function.

- An inner function can **read** variables from an enclosing scope automatically.
- However, assigning to an enclosing variable inside the inner function creates a *new local variable* by default rather than modifying the outer variable.

### Modifying Enclosing Variables with `nonlocal`

To **rebind** (reassign) a variable from an enclosing function scope, declare it with the `nonlocal` keyword.

```python
def outer():
    message = "hello"

    def inner_read_only():
        # Creates a separate local 'message'; does not touch outer's 'message'
        message = "hi"
        print("inner_read_only:", message)

    def inner_modify():
        nonlocal message
        message = "world"  # Rebinds outer's 'message'

    inner_read_only()
    print("After inner_read_only:", message)  # Still "hello"

    inner_modify()
    print("After inner_modify:", message)     # Now "world"


outer()
```

Output:
```text
inner_read_only: hi
After inner_read_only: hello
After inner_modify: world
```

### Important Rules for `nonlocal`

1. `nonlocal` searches **enclosing function scopes only**—it will **not** resolve to the global (module) scope.
2. The variable must already exist in an enclosing function scope when the inner function is compiled; otherwise, Python raises a `SyntaxError`.
3. If there are multiple nested functions, `nonlocal` binds to the **nearest** enclosing scope that defines the variable.

```python
x = 100  # Global


def bad_example():
    # nonlocal x  # SyntaxError: no binding for nonlocal 'x' found
    pass
```

> **Note:** Mutating a mutable object (like calling `list.append()` or `dict[k] = v`) does **not** require `nonlocal`, because you are modifying the object in place rather than rebinding the variable name.

```python
def collect_items():
    items = []

    def add(item):
        items.append(item)  # Mutates the list; no 'nonlocal' needed
        return items

    return add


adder = collect_items()
print(adder("apple"))   # ['apple']
print(adder("banana"))  # ['apple', 'banana']
```

---

## 4. Global Scope (`global` Keyword)

A **global scope** corresponds to the top level of a single Python module (`.py` file). Every module has its own global namespace, which can be inspected with `globals()`.

- Functions can **read** global variables freely.
- To **rebind** a global variable from inside a function, you must declare it with the `global` keyword.

```python
total_requests = 0  # Global variable


def handle_request():
    global total_requests
    total_requests += 1
    print(f"Handled request #{total_requests}")


handle_request()
handle_request()
print("Total:", total_requests)
```

Output:
```text
Handled request #1
Handled request #2
Total: 2
```

### `global` vs. `nonlocal`

| Feature | `global` | `nonlocal` |
| :--- | :--- | :--- |
| **Target scope** | Module (global) scope | Nearest enclosing function scope |
| **Must already exist?** | No (assigning creates it at module level) | Yes (`SyntaxError` if missing in enclosing functions) |
| **Stops at module boundary?** | Targets module directly | Never reaches module scope |

### Best Practice

Minimize the use of `global` for mutable state. Shared global variables make code harder to test, debug, and reason about. Prefer passing arguments and returning values, using classes, or using closures when state needs to be encapsulated.

---

## 5. Built-in Scope

The **built-in scope** is the outermost scope in the LEGB hierarchy. It contains Python's built-in functions (`print`, `len`, `sum`, `max`, `enumerate`, `zip`), types (`int`, `str`, `list`, `dict`), constants (`True`, `False`, `None`), and exceptions (`ValueError`, `KeyError`, `Exception`).

All of these live in the standard `builtins` module, which is automatically available in every module without importing.

```python
import builtins

print(len([1, 2, 3]))          # 3
print(builtins.len([1, 2, 3])) # 3 (same function)

# See a few names available in the built-in namespace
builtin_names = [name for name in dir(builtins) if not name.startswith("_")]
print("Count of public builtins:", len(builtin_names))
print("Sample:", builtin_names[:8])
```

Python only reaches the Built-in scope if a name is not found in Local, Enclosing, or Global scopes.

---

## 6. Variable Shadowing

**Variable shadowing** occurs when a variable in an inner scope has the **same name** as a variable in an outer scope (enclosing, global, or built-in). Within the inner scope, the inner variable *shadows* (hides) the outer one.

### Shadowing Global or Enclosing Variables

```python
tax_rate = 0.08  # Global


def calculate_price(amount):
    tax_rate = 0.20  # Shadows global 'tax_rate' inside this function
    return amount * (1 + tax_rate)


print(calculate_price(100))  # 120.0 (uses local 0.20)
print(tax_rate)              # 0.08  (global is unchanged)
```

### Shadowing Built-in Names (Common Mistake)

Because Built-in (`B`) is checked last, assigning to names like `list`, `dict`, `str`, `id`, `type`, `sum`, or `len` in a local or global scope shadows the built-in function and breaks code that tries to call it later in that scope.

```python
# Bad practice: shadowing built-ins
list = [1, 2, 3]
# numbers = list((4, 5, 6))  # TypeError: 'list' object is not callable

# Restore it if shadowed at module level:
del list
numbers = list((4, 5, 6))
print(numbers)  # [4, 5, 6]
```

**Tip:** Never name variables `list`, `dict`, `str`, `int`, `id`, `type`, `input`, `open`, `file`, `sum`, `min`, or `max`. If you need a name like `type` or `id`, append an underscore (e.g., `user_id`, `node_type`).

---

## 7. Closures: Definition and How They Work

### What Is a Closure?

A **closure** is a function object that **remembers values from its enclosing lexical scope** even after the outer function has finished executing and its local stack frame is gone.

Three conditions must be met for a closure to exist in Python:

1. There is a **nested (inner) function** defined inside an **outer function**.
2. The nested function **references** at least one variable from the outer function's scope.
3. The outer function **returns** the nested function (or passes it elsewhere so it outlives the outer call).

### How Closures Work Under the Hood

Normally, when an outer function finishes executing, its local variables are destroyed. However, if Python detects at compile time that an inner function references an outer local variable:

1. Instead of storing that variable solely in the outer function's temporary stack frame, Python wraps it in a **cell object** (`cell`).
2. Both the outer function (via `co_cellvars`) and the inner function (via `co_freevars` and `__closure__`) hold a reference to that same `cell` object.
3. Even after the outer function returns, the `cell` object stays alive in memory because the returned inner function still references it.

```python
def make_greeter(greeting):
    # 'greeting' is stored in a cell object because 'greet' uses it
    def greet(name):
        return f"{greeting}, {name}!"

    return greet  # Return the function object (without calling it)


say_hello = make_greeter("Hello")
say_hola = make_greeter("Hola")

print(say_hello("Alice"))  # Hello, Alice!
print(say_hola("Bob"))     # Hola, Bob!
```

Even though `make_greeter("Hello")` has returned, `say_hello` still has access to `"Hello"`, and `say_hola` has its own independent cell holding `"Hola"`.

---

## 8. Closure Use Cases (Data Hiding, Function Factories)

Closures offer a lightweight, Pythonic alternative to single-method classes.

### A. Data Hiding (Encapsulation)

Variables captured in a closure cannot be accessed or overwritten directly from outside the function—only through the interface exposed by the inner function(s).

```python
def create_vault(secret, pin):
    attempts_left = 3

    def access(entered_pin):
        nonlocal attempts_left
        if attempts_left <= 0:
            return "Vault locked."
        if entered_pin == pin:
            return secret
        attempts_left -= 1
        return f"Wrong PIN. Attempts left: {attempts_left}"

    return access


vault = create_vault("launch-codes-42", "9999")
print(vault("1111"))  # Wrong PIN. Attempts left: 2
print(vault("9999"))  # launch-codes-42
```

Notice that callers cannot directly inspect or reset `attempts_left` or `secret` via normal attribute access.

### B. Function Factories

A **function factory** is an outer function that generates and returns specialized functions configured with different parameters.

```python
def make_power(exponent):
    def power(base):
        return base ** exponent
    return power


square = make_power(2)
cube = make_power(3)

print(square(5))  # 25
print(cube(5))    # 125
```

### C. Callbacks & Event Handlers with Pre-bound Context

Closures let you attach configuration or state to a callback expected by an API (such as sorting keys, GUI callbacks, or filters) without modifying the callback's signature.

```python
def make_threshold_filter(minimum):
    def is_above(value):
        return value >= minimum
    return is_above


scores = [45, 72, 88, 60, 95]
passing = list(filter(make_threshold_filter(70), scores))
print(passing)  # [72, 88, 95]
```

### D. Decorators

Every Python decorator is built on closures: it takes a function, wraps it in an inner function that captures the original function, and returns the wrapper.

---

## 9. Free Variables

A **free variable** is a variable used inside a code block (such as a function) that is **not defined locally** in that block and **not global**—specifically, it is bound in an enclosing scope.

From the compiler's perspective:
- In the **outer** function, a variable captured by an inner function is a **cell variable** (`__code__.co_cellvars`).
- In the **inner** function, that same variable is a **free variable** (`__code__.co_freevars`).

```python
def outer(a):
    b = 10

    def inner(c):
        d = 20
        return a + b + c + d  # 'a' and 'b' are free variables in inner

    return inner


fn = outer(5)
print("outer cellvars:", outer.__code__.co_cellvars)  # ('a', 'b')
print("inner freevars:", fn.__code__.co_freevars)     # ('a', 'b')
print("inner varnames:", fn.__code__.co_varnames)     # ('c', 'd')
```

Output:
```text
outer cellvars: ('a', 'b')
inner freevars: ('a', 'b')
inner varnames: ('c', 'd')
```

> **Note:** A function without free variables from an enclosing scope is **not** a closure—even if it is nested inside another function.

```python
def outer_no_closure():
    def inner():
        return 42  # Uses no enclosing variables
    return inner


f = outer_no_closure()
print(f.__code__.co_freevars)  # ()
print(f.__closure__)           # None
```

---

## 10. The `__closure__` Attribute

Every Python function object has a `__closure__` attribute:

- If the function is **not** a closure (no free variables), `fn.__closure__` is `None`.
- If the function **is** a closure, `fn.__closure__` is a **tuple of `cell` objects**, one for each free variable in `fn.__code__.co_freevars` (in the same order).
- Each cell object exposes its stored value via `.cell_contents`.

```python
def make_multiplier(factor, offset=0):
    def multiply(value):
        return value * factor + offset
    return multiply


times_3_plus_1 = make_multiplier(3, 1)

print("Closure tuple:", times_3_plus_1.__closure__)
print("Free var names:", times_3_plus_1.__code__.co_freevars)

for name, cell in zip(
    times_3_plus_1.__code__.co_freevars,
    times_3_plus_1.__closure__,
):
    print(f"  {name} = {cell.cell_contents!r} (cell id: {hex(id(cell))})")
```

Sample output:
```text
Closure tuple: (<cell at 0x...: int object at 0x...>, <cell at 0x...: int object at 0x...>)
Free var names: ('factor', 'offset')
  factor = 3 (cell id: 0x...)
  offset = 1 (cell id: 0x...)
```

### Shared Cells Across Multiple Inner Functions

When two inner functions capture the same enclosing variable, they share the **exact same `cell` object**. Updating the variable with `nonlocal` in one inner function immediately updates what the other inner function sees.

```python
def make_counter():
    count = 0

    def inc():
        nonlocal count
        count += 1
        return count

    def get():
        return count

    return inc, get


inc, get = make_counter()
print(inc.__closure__[0] is get.__closure__[0])  # True (same cell object!)
print(inc())  # 1
print(inc())  # 2
print(get())  # 2
```

---

## 11. Common Closure Pitfalls (Late Binding & More)

### Pitfall 1: Late Binding in Loops (The #1 Closure Bug)

Closures capture **variables (cell references)**, not the **values** those variables held at the time the function was defined. Python looks up the cell's value when the closure is **called** (*late binding*), not when it is created.

```python
# BUGGY: All functions share the same loop variable 'i'
multipliers = []
for i in range(4):
    def fn(x):
        return x * i
    multipliers.append(fn)

print([m(2) for m in multipliers])
# Output: [6, 6, 6, 6]  <-- Expected [0, 2, 4, 6]!
```

Why did this happen? By the time the functions are called, the loop has finished and `i` is `3`. Every function references the same `i`.

The same bug occurs with `lambda` and list comprehensions:

```python
funcs = [lambda x: x + i for i in range(3)]
print([f(10) for f in funcs])  # [12, 12, 12] instead of [10, 11, 12]
```

#### Fix A: Default Argument Binding (Early Binding)

Default argument values in Python are evaluated **once when the function is defined**, locking in the current value of `i`:

```python
multipliers = []
for i in range(4):
    def fn(x, factor=i):  # Evaluates 'i' immediately at definition time
        return x * factor
    multipliers.append(fn)

print([m(2) for m in multipliers])  # [0, 2, 4, 6]
```

With `lambda`:

```python
funcs = [lambda x, i=i: x + i for i in range(3)]
print([f(10) for f in funcs])  # [10, 11, 12]
```

#### Fix B: A Helper Factory Function

Creating a separate function call for each iteration creates a distinct enclosing scope (and cell) per iteration:

```python
def make_multiplier(factor):
    return lambda x: x * factor


multipliers = [make_multiplier(i) for i in range(4)]
print([m(2) for m in multipliers])  # [0, 2, 4, 6]
```

#### Fix C: `functools.partial`

```python
from functools import partial


def multiply(factor, x):
    return x * factor


multipliers = [partial(multiply, i) for i in range(4)]
print([m(2) for m in multipliers])  # [0, 2, 4, 6]
```

---

### Pitfall 2: Forgetting `nonlocal` When Updating State

If you reassign an enclosing variable without `nonlocal`, Python treats it as a local variable and raises `UnboundLocalError`:

```python
def broken_counter():
    count = 0

    def inc():
        count += 1  # Equivalent to: count = count + 1 -> count is local!
        return count

    return inc


c = broken_counter()
# c()  # UnboundLocalError: cannot access local variable 'count' where it is not associated with a value
```

**Fix:** Add `nonlocal count` inside `inc()`.

---

### Pitfall 3: Accidentally Sharing Mutable Enclosing State

If a factory accepts a mutable object (like a `list` or `dict`) without copying it, external code holding a reference to that object can mutate the closure's internal state unexpectedly.

```python
def make_checker(allowed_roles):
    # Defensive copy prevents external mutations from affecting the closure
    roles = frozenset(allowed_roles)

    def has_access(role):
        return role in roles

    return has_access


roles_list = ["admin", "editor"]
check = make_checker(roles_list)
roles_list.clear()           # External mutation
print(check("admin"))        # True (protected by frozenset copy)
```

---

## 12. Practical Examples of Closures

### Example 1: Stateful Bank Account (Multiple Coordinated Closures)

A factory can return a dictionary or tuple of functions sharing the same enclosed state—providing clean encapsulation without a class.

```python
def create_account(owner, initial_balance=0):
    balance = initial_balance

    def deposit(amount):
        nonlocal balance
        if amount <= 0:
            raise ValueError("Deposit amount must be positive.")
        balance += amount
        return balance

    def withdraw(amount):
        nonlocal balance
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive.")
        if amount > balance:
            raise ValueError("Insufficient funds.")
        balance -= amount
        return balance

    def get_balance():
        return f"{owner}: ${balance:.2f}"

    return {
        "deposit": deposit,
        "withdraw": withdraw,
        "balance": get_balance,
    }


acct = create_account("Maya", 100)
acct["deposit"](50)
acct["withdraw"](30)
print(acct["balance"]())  # Maya: $120.00
```

### Example 2: Memoization (Caching Function Results)

A closure can maintain a private `cache` dictionary across calls to speed up expensive computations.

```python
def memoize(func):
    cache = {}

    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]

    return wrapper


@memoize
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)


print(fib(35))  # 9227465 (instant, instead of millions of redundant calls)
```

### Example 3: Configurable Logger / Prefix Formatter

```python
from datetime import datetime


def make_logger(component, level="INFO"):
    def log(message):
        timestamp = datetime.now().strftime("%H:%M:%S")
        print(f"[{timestamp}] [{level}] [{component}] {message}")
    return log


auth_log = make_logger("AUTH", level="WARN")
db_log = make_logger("DATABASE")

auth_log("Failed login attempt for user 'alice'")
db_log("Connection pool initialized")
```

### Example 4: Running Average Calculator

```python
def make_averager():
    total = 0.0
    count = 0

    def add_sample(value):
        nonlocal total, count
        total += value
        count += 1
        return total / count

    return add_sample


avg = make_averager()
print(avg(10))  # 10.0
print(avg(20))  # 15.0
print(avg(30))  # 20.0
```

### Example 5: Retry Decorator with Configurable Attempts

```python
def retry(max_attempts=3):
    def decorator(func):
        def wrapper(*args, **kwargs):
            last_error = None
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as exc:
                    last_error = exc
                    print(f"Attempt {attempt}/{max_attempts} failed: {exc}")
            raise last_error
        return wrapper
    return decorator


attempts = 0


@retry(max_attempts=3)
def flaky_fetch():
    global attempts
    attempts += 1
    if attempts < 3:
        raise ConnectionError("Temporary network glitch")
    return "Success!"


print(flaky_fetch())
```

Output:
```text
Attempt 1/3 failed: Temporary network glitch
Attempt 2/3 failed: Temporary network glitch
Success!
```

---

## Quick Summary Cheat Sheet

| Concept | Key Takeaway |
| :--- | :--- |
| **LEGB** | Lookup order: **L**ocal → **E**nclosing → **G**lobal → **B**uilt-in. |
| **`nonlocal`** | Rebinds a variable in the nearest enclosing function scope. |
| **`global`** | Rebinds (or creates) a variable in the module-level global scope. |
| **Shadowing** | Inner variable hides an outer or built-in name of the same name. |
| **Closure** | Nested function + references enclosing variable + outlives outer function. |
| **Free variable** | Variable used in a function that is bound in an enclosing scope (`co_freevars`). |
| **`__closure__`** | Tuple of `cell` objects holding free variables (`cell.cell_contents`), or `None`. |
| **Late binding** | Loop closures share the loop variable; fix with `arg=i` or a factory function. |
