# Modules and Packages in Python

## Table of Contents

1. [What are Modules?](#1-what-are-modules)
2. [The `import` Statement (`import`, `from...import`, `as`)](#2-the-import-statement-import-fromimport-as)
3. [Creating Your Own Modules](#3-creating-your-own-modules)
4. [The `__name__ == '__main__'` Idiom](#4-the-__name__--__main__-idiom)
5. [Module Search Path (`sys.path`)](#5-module-search-path-syspath)
6. [The `dir()` Function with Modules](#6-the-dir-function-with-modules)
7. [What are Packages?](#7-what-are-packages)
8. [The `__init__.py` File](#8-the-__init__py-file)
9. [Relative vs. Absolute Imports](#9-relative-vs-absolute-imports)
10. [Installing Packages with `pip`](#10-installing-packages-with-pip)
11. [Virtual Environments (`venv`)](#11-virtual-environments-venv)
12. [`requirements.txt`](#12-requirementstxt)
13. [Important Standard Library Modules](#13-important-standard-library-modules)
14. [Third-Party Packages Overview](#14-third-party-packages-overview)
15. [Creating Distributable Packages (`setup.py`, `pyproject.toml`)](#15-creating-distributable-packages-setuppy-pyprojecttoml)

---

## 1. What are Modules?

A **module** in Python is simply a file containing Python definitions and statements (with a `.py` extension). The module's name is the filename without the `.py` suffix.

### Why Use Modules?

- **Organization:** Break large programs into smaller, manageable, logical files.
- **Reusability:** Write functions, classes, and constants once and reuse them across multiple scripts or projects.
- **Namespace Isolation:** Each module has its own private symbol table (namespace), preventing naming collisions between variables or functions with the same name in different files.
- **Maintainability:** Easier to debug, test, and update isolated components.

### Types of Modules

1. **Built-in / Standard Library Modules:** Written in C or Python and bundled with Python (e.g., `math`, `sys`, `os`, `json`).
2. **User-Defined Modules:** `.py` files you create in your project.
3. **Third-Party Modules:** Installed from external repositories like PyPI using `pip` (e.g., `requests`, `numpy`).

```python
# Every imported module is an object of type 'module'
import math

print(type(math))      # <class 'module'>
print(math.__name__)   # 'math'
print(math.pi)         # 3.141592653589793
```

---

## 2. The `import` Statement (`import`, `from...import`, `as`)

Python provides several ways to bring names from a module into your current namespace.

### Basic `import`

Imports the entire module object. You access its contents using dot notation (`module.name`).

```python
import math

result = math.sqrt(16)
print(result)  # 4.0
```

You can also import multiple modules on separate lines (recommended by PEP 8):

```python
import os
import sys
import json
```

### Importing Specific Attributes (`from ... import ...`)

Imports specific functions, classes, or variables directly into the current namespace so you don't need the module prefix.

```python
from math import sqrt, pi

print(sqrt(25))  # 5.0
print(pi)        # 3.141592653589793
```

### Aliasing with `as`

Renames a module or imported attribute in the current scope. Useful for shortening long module names or resolving naming conflicts.

```python
import datetime as dt
from math import factorial as fact

today = dt.date.today()
print(fact(5))  # 120
```

Common community conventions:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

### Wildcard Import (`from module import *`)

Imports all public names (those not starting with `_`, or those listed in `__all__`) into the current namespace.

```python
from math import *

print(sin(0))  # 0.0
```

> **Best Practice:** Avoid `from module import *` in production code. It pollutes the namespace, makes it hard to tell where a name came from, and can silently overwrite existing names.

### How Python Executes Imports

1. Python checks `sys.modules` (an in-memory cache of already-imported modules). If found, it reuses the cached module object—modules are only executed **once** per interpreter session unless explicitly reloaded with `importlib.reload()`.
2. If not cached, Python searches `sys.path`, compiles the `.py` file to bytecode (`.pyc` stored in `__pycache__/`), executes the module top-to-bottom, and caches it in `sys.modules`.

---

## 3. Creating Your Own Modules

Any `.py` file can act as a module. Suppose you create a file named `math_utils.py`:

```python
# math_utils.py
"""Utility functions for basic arithmetic and geometry."""

PI = 3.141592653589793
_SECRET_FACTOR = 42  # Conventionally private (not exported by import *)


def add(a: float, b: float) -> float:
    """Return the sum of a and b."""
    return a + b


def circle_area(radius: float) -> float:
    """Return the area of a circle given its radius."""
    return PI * (radius ** 2)


class Calculator:
    """Simple stateful calculator."""

    def __init__(self, initial: float = 0.0) -> None:
        self.value = initial

    def add(self, amount: float) -> float:
        self.value += amount
        return self.value
```

Now, in `main.py` located in the same directory:

```python
# main.py
import math_utils
from math_utils import Calculator, circle_area

print(math_utils.add(3, 5))       # 8
print(circle_area(4))             # 50.26548245743669

calc = Calculator(10)
print(calc.add(5))                # 15
```

### Controlling Wildcard Exports with `__all__`

If a module defines `__all__`, `from module import *` will only import the names listed in `__all__`:

```python
# math_utils.py
__all__ = ["add", "circle_area"]
```

### Naming Rules for Custom Modules

- Use lowercase with underscores (PEP 8), e.g., `string_helpers.py`.
- Avoid shadowing standard library names (do **not** name your file `random.py`, `json.py`, `email.py`, `test.py`, or `typing.py`).

---

## 4. The `__name__ == '__main__'` Idiom

Every Python module has a built-in `__name__` attribute set automatically by the interpreter:

- When a file is **run directly** (`python my_module.py`), `__name__` is set to `'__main__'`.
- When a file is **imported** by another module (`import my_module`), `__name__` is set to the module's name (e.g., `'my_module'`).

This lets a module function both as a reusable library and as an executable script.

```python
# temperature.py
def celsius_to_fahrenheit(c: float) -> float:
    return (c * 9 / 5) + 32


def main() -> None:
    print("Running temperature self-test...")
    print(f"0°C = {celsius_to_fahrenheit(0)}°F")
    print(f"100°C = {celsius_to_fahrenheit(100)}°F")


if __name__ == "__main__":
    # Executes ONLY when run directly: python temperature.py
    # Skipped when imported: import temperature
    main()
```

**Why it matters:**
- Prevents unintended side effects (like printing output, launching servers, or running tests) when another script imports your module.
- Essential for `multiprocessing` on Windows/macOS (where child processes re-import the main module).

---

## 5. Module Search Path (`sys.path`)

When Python executes `import foo`, it searches for `foo.py` (or a `foo/` package directory) in the directories listed in `sys.path`, in order:

1. **The directory containing the input script** (or the current working directory in interactive mode).
2. **`PYTHONPATH` environment variable** (if set; a colon-separated list on Linux/macOS or semicolon-separated on Windows).
3. **Standard library directories** (installation-dependent defaults).
4. **`site-packages` directory** (where third-party packages installed via `pip` live).

> Note: Built-in modules compiled into the interpreter (found in `sys.builtin_module_names`, like `sys` or `builtins`) are checked before `sys.path`.

### Inspecting and Modifying `sys.path`

```python
import sys

# Print all search directories in order
for index, path in enumerate(sys.path):
    print(f"{index}: {path}")

# Temporarily add a custom directory at runtime
sys.path.append("/path/to/custom/modules")
```

### Checking Where a Module Was Loaded From

```python
import json

print(json.__file__)
# e.g., '/usr/lib/python3.12/json/__init__.py'
```

### Common Pitfall: Shadowing

Because the script directory (`sys.path[0]`) is searched *before* the standard library and `site-packages`, creating a file named `requests.py` or `math.py` in your project folder will cause `import requests` or `import math` to load your local file instead of the real module.

---

## 6. The `dir()` Function with Modules

`dir()` returns a sorted list of attribute names defined in a module (functions, classes, variables, and dunder attributes).

- `dir(module)`: Lists all names defined inside `module`.
- `dir()` (with no arguments): Lists names in the **current** local scope.

```python
import math

all_names = dir(math)
print(all_names[:10])
# ['__doc__', '__loader__', '__name__', '__package__', '__spec__', 'acos', 'acosh', 'asin', 'asinh', 'atan']

# Filter out private/dunder attributes to see public API
public_names = [name for name in dir(math) if not name.startswith("_")]
print(public_names)
```

### Inspecting Documentation Alongside `dir()`

```python
import json

print(dir(json))
help(json.dumps)       # Detailed docstring and signature
print(json.__doc__)    # Module-level docstring
```

---

## 7. What are Packages?

A **package** is a way of structuring Python's module namespace using **dotted module names** (e.g., `ecommerce.payments.stripe`). At the filesystem level, a package is simply a **directory** containing modules and (typically) subpackages.

### Example Package Structure

```text
ecommerce/
├── __init__.py
├── config.py
├── models/
│   ├── __init__.py
│   ├── user.py
│   └── product.py
└── payments/
    ├── __init__.py
    ├── stripe.py
    └── paypal.py
```

### Importing from Packages

```python
# Import a module inside a package
import ecommerce.payments.stripe
ecommerce.payments.stripe.charge(100)

# Import with an alias
import ecommerce.payments.stripe as stripe
stripe.charge(100)

# Import a specific module or symbol
from ecommerce.payments import stripe
from ecommerce.models.user import User
```

### Regular Packages vs. Namespace Packages (PEP 420)

- **Regular Package:** A directory containing an `__init__.py` file. Most projects use this.
- **Namespace Package (Python 3.3+):** A directory *without* `__init__.py`. Allows splitting a single logical package across multiple directories or distributions (e.g., plugin systems like `zope.interface` and `zope.schema`).

---

## 8. The `__init__.py` File

`__init__.py` marks a directory as a regular Python package and runs automatically the **first time** the package (or any submodule inside it) is imported.

### Common Uses of `__init__.py`

1. **Empty Marker File:** Simply indicates the directory is a package.
2. **Expose a Clean Public API:** Re-export key classes/functions at the package root so users don't have to know internal file paths.
3. **Define `__all__`:** Control `from package import *`.
4. **Package-Level Initialization & Metadata:** Set `__version__`, configure package loggers, or validate environment requirements.

```python
# ecommerce/__init__.py
"""E-commerce core package."""

__version__ = "1.0.0"

# Re-export commonly used objects at the top-level package
from .models.user import User
from .models.product import Product
from .payments import stripe

__all__ = ["User", "Product", "stripe"]
```

With that `__init__.py`, callers can write:

```python
import ecommerce

print(ecommerce.__version__)  # "1.0.0"
u = ecommerce.User("Alice")
```

---

## 9. Relative vs. Absolute Imports

### Absolute Imports

Specify the full path starting from the project's top-level package (on `sys.path`).

```python
# Inside ecommerce/payments/stripe.py
from ecommerce.config import API_KEY
from ecommerce.models.user import User
```

**Pros:** Explicit, unambiguous, easy to read, and recommended by PEP 8.

### Relative Imports

Specify the path relative to the **current module's** position inside the package using leading dots:

- `.` = current package (same directory)
- `..` = parent package (one directory up)
- `...` = grandparent package (two directories up)

```python
# Inside ecommerce/payments/stripe.py
from . import paypal          # ecommerce/payments/paypal.py
from .. import config         # ecommerce/config.py
from ..models.user import User  # ecommerce/models/user.py
```

**Pros:** Convenient when refactoring/renaming top-level packages or working inside deeply nested subpackages.

### Common Relative Import Error

Relative imports rely on `__package__` and fail if you run a file inside a package directly as a script:

```bash
# Fails with: ImportError: attempted relative import with no known parent package
python ecommerce/payments/stripe.py

# Correct: run as a module from the parent directory using -m
python -m ecommerce.payments.stripe
```

---

## 10. Installing Packages with `pip`

`pip` is Python's standard package installer, downloading packages from **PyPI** (Python Package Index, `https://pypi.org`).

> **Tip:** Prefer `python -m pip ...` (or `python3 -m pip ...`) over bare `pip ...` to ensure you install into the exact Python interpreter you are running.

### Essential `pip` Commands

```bash
# Check pip version and which Python it targets
python -m pip --version

# Install the latest version of a package
python -m pip install requests

# Install a specific version or version range
python -m pip install "requests==2.32.3"
python -m pip install "numpy>=1.26,<2.0"

# Upgrade a package
python -m pip install --upgrade requests

# Uninstall a package
python -m pip uninstall requests

# List installed packages
python -m pip list

# Show detailed metadata, dependencies, and install location
python -m pip show requests

# Install packages from a requirements file
python -m pip install -r requirements.txt

# Install a local project in editable (development) mode
python -m pip install -e .
```

### Version Specifiers

| Specifier | Meaning |
| :--- | :--- |
| `==1.2.3` | Exact version `1.2.3` |
| `>=1.2.0` | Version `1.2.0` or higher |
| `<2.0.0` | Strictly below `2.0.0` |
| `~=1.4.2` | Compatible release (`>=1.4.2, ==1.4.*`) |
| `!=1.5.0` | Exclude a specific buggy release |

---

## 11. Virtual Environments (`venv`)

A **virtual environment** is an isolated directory containing its own Python binary and `site-packages` folder.

### Why Use Virtual Environments?

- **Dependency Isolation:** Project A can use Django 4.2 while Project B uses Django 5.1 without conflict.
- **System Safety:** Avoids breaking OS tools that rely on the system Python (especially on Linux).
- **Reproducibility:** Keeps project dependencies clean and easy to freeze into `requirements.txt`.

### Creating, Activating, and Managing `venv`

```bash
# 1. Create a virtual environment named .venv in your project folder
python3 -m venv .venv

# 2. Activate it
# Linux / macOS (bash/zsh):
source .venv/bin/activate

# Windows (Command Prompt):
.venv\Scripts\activate.bat

# Windows (PowerShell):
.venv\Scripts\Activate.ps1

# 3. Verify the active Python & pip now point inside .venv
which python     # /path/to/project/.venv/bin/python (Linux/macOS)
where python     # Windows

# 4. Install packages inside the isolated environment
python -m pip install requests flask

# 5. Deactivate when finished
deactivate
```

> **Note:** Never commit the `.venv/` directory to Git—add `.venv/` to `.gitignore`. Instead, track dependencies via `requirements.txt` or `pyproject.toml`.

---

## 12. `requirements.txt`

`requirements.txt` is a plain-text file listing the packages required by a project so anyone can recreate the exact environment.

### Generating and Installing `requirements.txt`

```bash
# Export all currently installed packages and exact versions
python -m pip freeze > requirements.txt

# Recreate the environment elsewhere
python -m pip install -r requirements.txt
```

### Example `requirements.txt`

```text
# Web framework and HTTP client
flask==3.0.3
requests>=2.31.0,<3.0.0

# Data processing
numpy~=1.26.4
pandas==2.2.2

# Include another requirements file
-r requirements-dev.txt
```

### Best Practice: Separating Production vs. Development Dependencies

- `requirements.txt`: Core runtime dependencies (`flask`, `requests`).
- `requirements-dev.txt`: Testing, linting, and formatting tools (`pytest`, `ruff`, `mypy`), starting with `-r requirements.txt`.

---

## 13. Important Standard Library Modules

Python ships with a rich **"batteries included"** standard library.

### `os` — Operating System Interface

Interacts with environment variables, processes, and low-level filesystem operations.

```python
import os

print(os.getcwd())                     # Current working directory
print(os.listdir("."))                 # List directory entries
os.makedirs("logs/2026", exist_ok=True)
home = os.environ.get("HOME", "/tmp")  # Read environment variable safely
```

### `sys` — System-Specific Parameters and Functions

Accesses interpreter details, command-line arguments, stdin/stdout, and `sys.path`.

```python
import sys

print(sys.version)       # Python version string
print(sys.platform)      # e.g., 'linux', 'win32', 'darwin'
print(sys.argv)          # List of CLI arguments: ['script.py', 'arg1', ...]

if len(sys.argv) < 2:
    sys.stderr.write("Usage: script.py <name>\n")
    sys.exit(1)
```

### `math` — Mathematical Functions

Provides floating-point math functions and constants.

```python
import math

print(math.sqrt(81))          # 9.0
print(math.ceil(4.2))         # 5
print(math.floor(4.8))        # 4
print(math.factorial(6))      # 720
print(math.gcd(48, 18))       # 6
print(math.log(100, 10))      # 2.0
print(math.pi, math.e)
```

### `random` — Pseudo-Random Number Generation

Generates pseudo-random numbers and selections (use `secrets` for cryptographic security).

```python
import random

print(random.randint(1, 10))              # Inclusive integer in [1, 10]
print(random.uniform(0.0, 1.0))           # Float in [0.0, 1.0]
choices = ["apple", "banana", "cherry"]
print(random.choice(choices))             # Single random item
print(random.sample(choices, k=2))        # Unique items without replacement
random.shuffle(choices)                   # In-place shuffle
```

### `datetime` — Dates, Times, and Timezones

Handles date/time arithmetic, formatting, parsing, and timezones.

```python
from datetime import datetime, date, timedelta, timezone

now = datetime.now(timezone.utc)
tomorrow = now + timedelta(days=1)

formatted = now.strftime("%Y-%m-%d %H:%M:%S")
parsed = datetime.strptime("2026-11-26 14:30", "%Y-%m-%d %H:%M")

print(formatted, parsed.date())
```

### `collections` — Specialized Container Datatypes

High-performance alternatives to built-in `dict`, `list`, `set`, and `tuple`.

```python
from collections import Counter, defaultdict, deque, namedtuple

# Counter: frequency counting
counts = Counter("abracadabra")
print(counts.most_common(2))  # [('a', 5), ('b', 2)]

# defaultdict: automatic default value for missing keys
groups = defaultdict(list)
groups["fruits"].append("apple")

# deque: O(1) appends and pops from both ends
queue = deque(["a", "b", "c"])
queue.appendleft("start")
queue.pop()

# namedtuple: lightweight immutable object with named fields
Point = namedtuple("Point", ["x", "y"])
p = Point(10, 20)
print(p.x, p.y)
```

### `functools` — Higher-Order Functions and Operations on Callables

```python
from functools import lru_cache, partial, wraps

# Memoization for expensive pure functions
@lru_cache(maxsize=128)
def fib(n: int) -> int:
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

# Pre-fill arguments of a function
base_two_int = partial(int, base=2)
print(base_two_int("1010"))  # 10

# Preserve wrapped function metadata in decorators
def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

### `itertools` — Efficient Iterators for Looping

```python
import itertools

# Combine iterables lazily
print(list(itertools.chain([1, 2], [3, 4])))         # [1, 2, 3, 4]
print(list(itertools.combinations("ABC", 2)))        # [('A','B'), ('A','C'), ('B','C')]
print(list(itertools.permutations("AB", 2)))         # [('A','B'), ('B','A')]
print(list(itertools.islice(itertools.count(10, 2), 4)))  # [10, 12, 14, 16]
```

### `json` — JSON Encoding and Decoding

```python
import json

data = {"name": "Alice", "skills": ["Python", "SQL"], "active": True}

# Serialize Python object -> JSON string / file
json_str = json.dumps(data, indent=2)
with open("user.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2)

# Deserialize JSON string / file -> Python object
parsed = json.loads(json_str)
with open("user.json", "r", encoding="utf-8") as f:
    loaded = json.load(f)
```

### `re` — Regular Expressions

Pattern matching and text extraction/replacement.

```python
import re

text = "Contact us at support@example.com or sales@example.org"
emails = re.findall(r"[\w.+-]+@[\w-]+\.[\w.-]+", text)
print(emails)  # ['support@example.com', 'sales@example.org']

masked = re.sub(r"\d{4}", "XXXX", "Card: 1234-5678")
print(masked)  # Card: XXXX-XXXX
```

### `pathlib` — Object-Oriented Filesystem Paths

Modern, cross-platform replacement for many `os.path` operations.

```python
from pathlib import Path

base = Path(".")
config_path = base / "configs" / "app.json"

print(config_path.name)        # 'app.json'
print(config_path.stem)        # 'app'
print(config_path.suffix)      # '.json'
print(config_path.parent)      # PosixPath('configs')

# Read/write and glob files cleanly
for py_file in Path(".").rglob("*.py"):
    print(py_file)
```

### `typing` — Type Hints Support

Improves readability, IDE autocompletion, and static type checking (with tools like `mypy` or `pyright`).

```python
from typing import Optional, Callable, TypeVar, Protocol

T = TypeVar("T")

def find_user(user_id: int) -> Optional[dict[str, str]]:
    if user_id == 1:
        return {"name": "Alice"}
    return None

def apply_twice(fn: Callable[[int], int], val: int) -> int:
    return fn(fn(val))
```

### `logging` — Flexible Event Logging

Replaces `print()` for production diagnostics with configurable severity levels and output targets.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
)
logger = logging.getLogger(__name__)

logger.debug("Detailed debug info (hidden at INFO level)")
logger.info("Application started")
logger.warning("Disk space below 15%")
logger.error("Failed to connect to database")
```

### `unittest` — Built-in Unit Testing Framework

```python
import unittest

def multiply(a: int, b: int) -> int:
    return a * b

class TestMathOperations(unittest.TestCase):
    def test_multiply(self):
        self.assertEqual(multiply(3, 4), 12)
        self.assertEqual(multiply(-2, 5), -10)

if __name__ == "__main__":
    unittest.main()
```

### `argparse` — Command-Line Argument Parsing

```python
import argparse

parser = argparse.ArgumentParser(description="Process a data file.")
parser.add_argument("filename", help="Path to the input file")
parser.add_argument("-v", "--verbose", action="store_true", help="Enable verbose output")
parser.add_argument("-n", "--limit", type=int, default=10, help="Max rows to process")

# args = parser.parse_args()
# print(args.filename, args.verbose, args.limit)
```

### `subprocess` — Running External Commands and Processes

```python
import subprocess

result = subprocess.run(
    ["python3", "--version"],
    capture_output=True,
    text=True,
    check=True,
)
print(result.stdout.strip())
```

> **Security Tip:** Avoid `shell=True` with untrusted input to prevent shell injection vulnerabilities.

### `threading` — Thread-Based Concurrency

Best suited for **I/O-bound** tasks (network requests, disk reads, waiting on sockets), because CPython's Global Interpreter Lock (GIL) allows only one thread to execute Python bytecode at a time.

```python
import threading
import time

def worker(name: str) -> None:
    print(f"Thread {name} starting")
    time.sleep(1)
    print(f"Thread {name} finished")

threads = [threading.Thread(target=worker, args=(f"T{i}",)) for i in range(3)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

### `multiprocessing` — Process-Based Parallelism

Spawns separate OS processes, each with its own Python interpreter and memory space—bypassing the GIL for **CPU-bound** work.

```python
from multiprocessing import Pool

def square(n: int) -> int:
    return n * n

if __name__ == "__main__":
    with Pool(processes=4) as pool:
        results = pool.map(square, [1, 2, 3, 4, 5])
    print(results)  # [1, 4, 9, 16, 25]
```

---

## 14. Third-Party Packages Overview

Third-party libraries extend Python across web development, data science, automation, and more.

### `requests` — Human-Friendly HTTP Client

Simplifies calling REST APIs, sending headers/JSON payloads, and handling cookies or timeouts.

```python
import requests

response = requests.get("https://api.github.com/repos/python/cpython", timeout=5)
response.raise_for_status()
repo = response.json()
print(repo["full_name"], repo["stargazers_count"])
```

### `numpy` — Numerical Computing

Provides fast, C-backed N-dimensional arrays (`ndarray`), vectorized math, and linear algebra.

```python
import numpy as np

arr = np.array([1, 2, 3, 4])
print(arr * 2)           # [2 4 6 8]
print(np.mean(arr))      # 2.5

matrix = np.array([[1, 2], [3, 4]])
print(matrix.T)          # Transpose
```

### `pandas` — Data Analysis and Manipulation

Built on top of NumPy; introduces `Series` (1D) and `DataFrame` (2D tabular data) for cleaning, filtering, grouping, and reading CSV/Excel/SQL data.

```python
import pandas as pd

df = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie"],
    "score": [88, 95, 72],
})
high_scorers = df[df["score"] >= 80]
print(high_scorers.describe())
```

### `flask` — Lightweight Web Microframework

Minimalist, flexible framework ideal for APIs, microservices, and small-to-medium web apps.

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/health")
def health():
    return jsonify({"status": "ok"})

if __name__ == "__main__":
    app.run(debug=True)
```

### `django` — Full-Featured "Batteries-Included" Web Framework

Follows the Model-Template-View (MTV) architecture and includes an ORM, built-in admin panel, authentication, migrations, and form handling out of the box—ideal for large, database-driven web applications.

| Feature | Flask | Django |
| :--- | :--- | :--- |
| Philosophy | Microframework (minimal core, bring your own extensions) | Full-stack ("batteries included") |
| Database / ORM | None built-in (commonly uses SQLAlchemy) | Built-in ORM & schema migrations |
| Admin UI | Third-party extensions | Built-in auto-generated admin |
| Best For | Microservices, REST APIs, custom architectures | Content sites, enterprise apps, rapid full-stack builds |

---

## 15. Creating Distributable Packages (`setup.py`, `pyproject.toml`)

To share your library internally or publish it to PyPI, you structure it as a distributable package.

### Recommended `src/` Layout

```text
my_awesome_pkg/
├── pyproject.toml
├── README.md
├── LICENSE
├── src/
│   └── my_awesome_pkg/
│       ├── __init__.py
│       └── core.py
└── tests/
    └── test_core.py
```

Using a `src/` layout ensures tests import the *installed* package rather than accidentally importing the local source folder.

### Modern Standard: `pyproject.toml` (PEP 517 / 518 / 621)

`pyproject.toml` is the modern, declarative standard for configuring Python builds, metadata, dependencies, and tool settings (replacing standalone `setup.py` for most projects).

```toml
# pyproject.toml
[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-awesome-pkg"
version = "0.1.0"
description = "A short summary of what the package does."
readme = "README.md"
requires-python = ">=3.10"
license = { text = "MIT" }
authors = [
    { name = "Your Name", email = "you@example.com" },
]
dependencies = [
    "requests>=2.31.0",
]

[project.optional-dependencies]
dev = ["pytest>=8.0", "ruff>=0.4"]

[project.scripts]
awesome-cli = "my_awesome_pkg.core:main"

[tool.setuptools.packages.find]
where = ["src"]
```

### Legacy Approach: `setup.py`

Older projects (or packages with complex C/C++ extension builds) use `setup.py` with `setuptools`:

```python
# setup.py
from setuptools import setup, find_packages

setup(
    name="my-awesome-pkg",
    version="0.1.0",
    description="A short summary of what the package does.",
    author="Your Name",
    author_email="you@example.com",
    package_dir={"": "src"},
    packages=find_packages(where="src"),
    python_requires=">=3.10",
    install_requires=[
        "requests>=2.31.0",
    ],
    entry_points={
        "console_scripts": [
            "awesome-cli=my_awesome_pkg.core:main",
        ],
    },
)
```

### Building and Publishing a Package

```bash
# 1. Install build and upload tools
python -m pip install --upgrade build twine

# 2. Build source distribution (.tar.gz) and wheel (.whl) into dist/
python -m build

# 3. Verify the distribution files
python -m twine check dist/*

# 4. Upload to TestPyPI first (recommended) or PyPI
python -m twine upload --repository testpypi dist/*
python -m twine upload dist/*
```

### Summary Comparison

| File | Role | Status |
| :--- | :--- | :--- |
| `pyproject.toml` | Unified build system, package metadata, dependencies, tool config | **Current standard** (PEP 621) |
| `setup.py` | Imperative Python build script for `setuptools` | **Legacy** (still used for custom C extensions) |
| `dist/*.whl` | Pre-built binary/pure-Python wheel for fast `pip` installs | Built artifact |
| `dist/*.tar.gz` | Source distribution (sdist) | Built artifact |
