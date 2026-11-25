# Introduction to Python

Python is one of the most popular, versatile, and beginner-friendly programming languages in the world. Its clean, readable syntax and massive ecosystem make it a top choice for everything from quick automation scripts to large-scale web applications and cutting-edge artificial intelligence.

---

## 1. What is Python? History and Evolution

**Python** is a high-level, general-purpose, interpreted programming language designed with an emphasis on **code readability** and developer productivity. Its design philosophy encourages writing clear, logical code for both small and large projects.

### Origin and Naming
- **Creator:** Guido van Rossum
- **Origin:** Started as a hobby project during Christmas 1989 at **Centrum Wiskunde & Informatica (CWI)** in the Netherlands, as a successor to the ABC programming language.
- **First Public Release:** February 1991 (**Python 0.9.0**).
- **Why "Python"?** Contrary to popular belief, Python is **not** named after the snake. Guido van Rossum was a fan of the British comedy series *Monty Python's Flying Circus* and wanted a short, unique, and slightly mysterious name.

> [!NOTE]
> Guido van Rossum was known in the Python community as the **BDFL** (*Benevolent Dictator For Life*) until he stepped down from that role in July 2018. Python's evolution is now governed by an elected **Steering Council** and the **Python Enhancement Proposal (PEP)** process.

### Major Version Milestones

| Version | Release Year | Key Highlights |
| :--- | :--- | :--- |
| **Python 0.9.0** | 1991 | Functions, exception handling, core data types (`list`, `dict`, `str`), classes with inheritance |
| **Python 1.0** | 1994 | Functional programming tools (`lambda`, `map`, `filter`, `reduce`) |
| **Python 2.0** | 2000 | List comprehensions, cycle-detecting garbage collector, full Unicode support introduced |
| **Python 3.0** | 2008 | Major backwards-incompatible cleanup (`print()` function, Unicode strings by default, cleaner integer division) |
| **Python 2.7 EOL** | 2020 | Official End-of-Life for Python 2 on January 1, 2020; all modern development uses Python 3 |
| **Python 3.8 – 3.9** | 2019–2020 | Assignment expressions (`:=` walrus operator), positional-only parameters, dictionary union (`\|`) |
| **Python 3.10** | 2021 | Structural pattern matching (`match`/`case`), much clearer syntax error messages |
| **Python 3.11+** | 2022–Present | Significant speedups (Faster CPython project), exception groups, improved typing, free-threaded mode (experimental in 3.13) |

---

## 2. Features of Python

Python combines simplicity with power through several defining characteristics:

- **Easy to Learn and Read:** Python uses English-like keywords and indentation instead of curly braces, making programs look almost like pseudocode.
- **Interpreted:** Code is executed directly by the Python interpreter without requiring a separate manual compilation step into machine code. This speeds up the edit-test-debug cycle.
- **Dynamically Typed (and Strongly Typed):**
  - *Dynamic:* You do not declare variable types explicitly; types are associated with values at runtime and can change.
  - *Strong:* Python does not silently perform unsafe implicit conversions (for example, adding a number to a string raises a `TypeError`).
- **High-Level Language:** Abstracts away low-level hardware details such as memory allocation, pointers, and CPU registers.
- **Automatic Memory Management:** Built-in reference counting and cyclic garbage collection automatically reclaim unused memory.
- **Cross-Platform / Portable:** The same Python code typically runs without modification on Windows, macOS, and Linux.
- **Multi-Paradigm:** Supports **procedural**, **object-oriented (OOP)**, and **functional** programming styles.
- **Extensive Standard Library ("Batteries Included"):** Ships with modules for file I/O, networking, regular expressions, JSON, dates, math, testing, and more.
- **Extensible and Embeddable:** Performance-critical parts can be written in C/C++ and called from Python (used heavily by libraries like NumPy).

```python
# Dynamic typing: the same variable name can refer to different object types
data = 42
print(type(data))       # <class 'int'>

data = "Hello, Python"
print(type(data))       # <class 'str'>

# Strong typing: prevents accidental mixing of incompatible types
# print("Age: " + 25)   # Raises TypeError: can only concatenate str (not "int") to str
print("Age: " + str(25))  # Explicit conversion works -> Age: 25
```

```python
# "Batteries included": using the built-in math and datetime modules
import math
from datetime import date

print(math.sqrt(144))   # 12.0
print(date.today())     # Prints current date (e.g., 2026-11-26)
```

---

## 3. Python 2 vs Python 3 Differences

Python 3 was released in 2008 to fix fundamental design flaws in Python 2 that could not be resolved without breaking backwards compatibility. **Python 2 is obsolete** and no longer receives security updates.

### Quick Comparison

| Feature | Python 2 (Legacy / EOL) | Python 3 (Modern Standard) |
| :--- | :--- | :--- |
| **`print`** | Statement: `print "Hello"` | Function: `print("Hello")` |
| **Integer Division** | `5 / 2` evaluates to `2` (truncates) | `5 / 2` evaluates to `2.5` (`//` gives `2`) |
| **Default Strings** | ASCII `str` (8-bit); `u"text"` for Unicode | Unicode `str` (UTF-8 source); `b"bytes"` for raw bytes |
| **User Input** | `raw_input()` for strings; `input()` evaluated code | `input()` always returns a string |
| **Ranges** | `range()` creates a full list; `xrange()` is lazy | `range()` is lazy by default (`xrange()` removed) |
| **Exceptions** | `except Exception, e:` | `except Exception as e:` |
| **Dictionary Views** | `.keys()`, `.values()`, `.items()` returned lists | Return memory-efficient dynamic view objects |

### Code Comparison

```python
# --- Python 2 style (DO NOT USE in new code) ---
# print "Result:", 5 / 2        # Output: Result: 2
# name = raw_input("Name: ")
# for i in xrange(5):
#     print i

# --- Python 3 style (Current Standard) ---
print("Result:", 5 / 2)         # Output: Result: 2.5
print("Floor division:", 5 // 2)  # Output: Floor division: 2

# input() safely returns a string without evaluating expressions
# name = input("Name: ")

# range() is memory-efficient and generates numbers on demand
for i in range(5):
    print(i, end=" ")           # Output: 0 1 2 3 4
```

> [!IMPORTANT]
> Always write new projects in **Python 3**. You may only encounter Python 2 syntax when reading very old tutorials or migrating legacy codebases.

---

## 4. Installing Python (Windows, Mac, Linux)

Download official installers from [python.org/downloads](https://www.python.org/downloads/) or use your OS package manager.

### Windows
1. Download the latest Python 3 Windows installer from [python.org](https://www.python.org/downloads/).
2. Run the `.exe` installer.
3. **Critical Step:** Check **"Add python.exe to PATH"** at the bottom of the first screen before clicking **Install Now**.
4. Verify in Command Prompt or PowerShell:
   ```bash
   python --version
   pip --version
   ```
   *(On Windows, the Python Launcher `py` is also available: `py --version`.)*

> [!TIP]
> Alternatively, on Windows 10/11 you can install Python via `winget`:
> ```bash
> winget install Python.Python.3.12
> ```

### macOS
macOS may include system stubs, so install a clean user-managed version:
- **Option A (Official Installer):** Download the `.pkg` installer from [python.org](https://www.python.org/downloads/) and follow the prompts.
- **Option B (Homebrew — recommended for developers):**
  ```bash
  brew install python
  ```
- Verify in Terminal:
  ```bash
  python3 --version
  pip3 --version
  ```

### Linux
Most Linux distributions ship with Python 3 pre-installed for system tools, though `pip` or `venv` may need to be added separately.

- **Ubuntu / Debian:**
  ```bash
  sudo apt update
  sudo apt install python3 python3-pip python3-venv
  ```
- **Fedora / RHEL:**
  ```bash
  sudo dnf install python3 python3-pip
  ```
- **Arch Linux:**
  ```bash
  sudo pacman -S python python-pip
  ```
- Verify in Terminal:
  ```bash
  python3 --version
  ```

> [!WARNING]
> **Never uninstall or overwrite the system Python** on Linux or macOS—operating system utilities depend on it. To manage multiple Python versions safely per project, use tools like **`uv`**, **`pyenv`**, or virtual environments (`python3 -m venv .venv`).

---

## 5. Running Python Scripts

There are three primary ways to run Python code depending on your workflow.

### 1. Interactive Mode (REPL)
REPL stands for **Read–Eval–Print Loop**. It executes code one statement at a time and immediately displays the result.

Start it from your terminal:
```bash
python3
```

Inside the `>>>` prompt:
```python
>>> 2 + 3 * 4
14
>>> message = "Testing Python REPL"
>>> message.upper()
'TESTING PYTHON REPL'
>>> exit()   # Or press Ctrl+D (Linux/macOS) / Ctrl+Z then Enter (Windows)
```

- **Best for:** Quick calculations, testing syntax, and exploring modules or APIs.

### 2. Script Mode
Save your code in a file with a `.py` extension (e.g., `hello.py`) and run it from the terminal:

```bash
python3 hello.py
```

You can also pass inline commands or run built-in modules directly from the CLI:
```bash
# Run a one-liner without creating a file
python3 -c "print('Quick check:', 10 ** 2)"

# Run a module as a script (e.g., start a local web server)
python3 -m http.server 8000
```

On Linux and macOS, you can add a **shebang** line at the very top of a script to make it directly executable:
```python
#!/usr/bin/env python3
print("Running as an executable script!")
```
```bash
chmod +x hello.py
./hello.py
```

### 3. Running Inside an IDE
IDEs and editors provide a **Run** button (or keyboard shortcut like `F5` / `Ctrl+F5`) that automatically selects the configured Python interpreter, executes your script in an integrated terminal, and lets you set breakpoints for step-by-step debugging.

---

## 6. Python IDEs and Text Editors

Choosing the right environment depends on your experience level and project type.

| Tool | Type | Best For | Key Strengths |
| :--- | :--- | :--- | :--- |
| **IDLE** | Built-in IDE | Absolute beginners | Bundled with Python; zero setup; simple shell and editor |
| **VS Code** | Code Editor | General development, web, scripts | Fast, free, rich Python extension, integrated terminal, debugging, Git |
| **PyCharm** | Full IDE | Large Python projects & backends | Deep code analysis, smart refactoring, built-in test runner & debugger |
| **Jupyter** | Interactive Notebook | Data science, ML, teaching | Mixes runnable code cells, Markdown notes, tables, and charts inline |

### Closer Look
- **IDLE (Integrated Development and Learning Environment):** Installed automatically with Python on Windows/macOS. Great for writing your first programs without getting overwhelmed by configuration.
- **Visual Studio Code (VS Code):** A lightweight, extensible editor by Microsoft. Installing the official **Python** extension adds IntelliSense autocomplete, linting, formatting, debugging, and virtual environment detection.
- **PyCharm (by JetBrains):** A dedicated Python IDE available in a free **Community Edition** and a paid **Professional Edition** (which adds web framework and database tools). Excellent for navigating large codebases.
- **Jupyter Notebook / JupyterLab:** Runs in your browser and organizes code into executable cells. Widely used in data analysis, scientific research, and machine learning (alongside cloud options like Google Colab).

> [!TIP]
> If you are unsure where to start, **VS Code** (with the official Python extension) is an ideal all-around choice, while **JupyterLab** is best if your focus is data science.

---

## 7. First Python Program — Hello World

Create a file named `hello.py` and add the following line:

```python
print("Hello, World!")
```

Run it with `python3 hello.py` (or `python hello.py` on Windows):

```text
Hello, World!
```

### How It Works
- `print` is a **built-in function** that writes output to the console (standard output).
- `(...)` calls the function and passes arguments to it.
- `"Hello, World!"` is a **string literal** enclosed in double (or single) quotes.

### Useful Variations

```python
# Single and double quotes both create strings
print('Hello using single quotes!')

# Printing multiple items (separated by a space by default)
name = "Alice"
age = 28
print("Name:", name, "| Age:", age)

# Using an f-string (formatted string literal) — the modern, preferred way
print(f"Hello, {name}! Next year you will be {age + 1}.")

# Customizing separator (sep) and line ending (end)
print("2026", "11", "26", sep="-", end="!\n")  # Output: 2026-11-26!
```

Interactive greeting using `input()`:
```python
user_name = input("What is your name? ")
print(f"Welcome to Python, {user_name}!")
```

---

## 8. Python Syntax Rules

Python's syntax is intentionally minimal, relying on whitespace and clear visual structure rather than braces or semicolons.

### 1. Indentation Defines Code Blocks
Most C-style languages use `{ ... }` to group statements. Python uses **indentation** (leading whitespace). All statements inside the same block must share the exact same indentation level.

```python
score = 85

if score >= 60:
    print("Status: Passed")
    if score >= 80:
        print("Grade: Distinction")
else:
    print("Status: Needs Improvement")

print("Evaluation complete.")  # Outside the if/else blocks
```

- **Standard:** **4 spaces** per indentation level.
- **Rule:** Never mix tabs and spaces in the same file—Python 3 raises a `TabError` or `IndentationError`.

### 2. Statements, Line Continuation, and Case Sensitivity
- **One statement per line:** Semicolons (`;`) are optional and discouraged unless separating multiple short statements on one line (rare in idiomatic Python).
- **Case-sensitive:** `total`, `Total`, and `TOTAL` are three completely different names.
- **Line continuation:** Expressions inside parentheses `()`, brackets `[]`, or braces `{}` can span multiple lines naturally (implicit continuation). Outside brackets, you can use a backslash `\`, though parentheses are preferred.

```python
# Preferred: implicit line continuation inside parentheses
total = (
    100
    + 250
    + 50
)

# List spanning multiple lines
languages = [
    "Python",
    "Rust",
    "Go",
]
```

### 3. Comments
Comments explain *why* code does something and are ignored by the Python interpreter.

```python
# This is a full-line comment

price = 19.99  # This is an inline comment (separate by at least 2 spaces)

# Python does not have a dedicated /* ... */ multi-line comment syntax.
# For multi-line comments, place a '#' at the start of each line.
```

### 4. Docstrings
A **docstring** (documentation string) is a triple-quoted string (`"""..."""`) placed as the **first statement** of a module, class, or function. Unlike comments, docstrings are retained at runtime and power `help()` and documentation generators.

```python
def calculate_area(radius):
    """Return the area of a circle given its radius.

    Args:
        radius (float): Non-negative radius of the circle.
    """
    return 3.14159 * (radius ** 2)


print(calculate_area.__doc__)  # Access the docstring programmatically
# help(calculate_area)         # Displays formatted documentation
```

> [!NOTE]
> Avoid using standalone triple-quoted strings (`"""..."""`) as general comments inside code blocks—they are still parsed as string literals, whereas `#` comments are completely ignored.

---

## 9. Python Keywords and Identifiers

### Keywords
**Keywords** are reserved words that have special syntactic meaning in Python. You **cannot** use them as variable names, function names, or class names.

You can list all keywords in your installed Python version anytime:

```python
import keyword

print("Keyword count:", len(keyword.kwlist))
print(keyword.kwlist)
```

Common keywords grouped by purpose:
- **Values:** `True`, `False`, `None`
- **Control Flow:** `if`, `elif`, `else`, `for`, `while`, `break`, `continue`, `pass`
- **Functions & Classes:** `def`, `return`, `yield`, `lambda`, `class`
- **Exceptions:** `try`, `except`, `finally`, `raise`, `assert`
- **Modules & Scope:** `import`, `from`, `as`, `global`, `nonlocal`
- **Logic & Identity:** `and`, `or`, `not`, `in`, `is`
- **Context & Async:** `with`, `async`, `await`, `del`

*(Python also has **soft keywords** like `match`, `case`, `type`, and `_`, which act as keywords only in specific contexts such as pattern matching or type aliases.)*

### Identifiers
An **identifier** is a name you give to a variable, function, class, module, or other object.

**Rules for Valid Identifiers:**
1. Can contain letters (`a–z`, `A–Z`), digits (`0–9`), and underscores (`_`).
2. **Must start** with a letter or an underscore (cannot start with a digit).
3. **Cannot** be a Python keyword.
4. **Cannot** contain spaces or special symbols (`@`, `$`, `%`, `-`, `!`).
5. **Case-sensitive** (`user_age` and `user_Age` are distinct).

```python
# Valid identifiers
user_name = "Maya"
_internal_flag = True
item2 = "Notebook"
MAX_RETRIES = 3

# Invalid identifiers (would cause SyntaxError)
# 2nd_item = "Pen"      # Starts with a digit
# user-name = "Maya"    # Contains a hyphen
# class = "Math"        # Uses a reserved keyword
# total$ = 50           # Contains a special symbol
```

### Standard Naming Conventions
- `snake_case` — variables, functions, methods, modules (e.g., `calculate_tax`, `user_profile.py`)
- `PascalCase` (CapWords) — classes (e.g., `BankAccount`, `HttpResponse`)
- `UPPER_SNAKE_CASE` — constants (e.g., `PI`, `DEFAULT_TIMEOUT`)
- `_single_leading_underscore` — convention indicating internal/private use
- `__double_leading_and_trailing__` ("dunder") — special Python methods/attributes (e.g., `__init__`, `__name__`)

---

## 10. Python's PEP 8 Style Guide Basics

**PEP 8** (*Python Enhancement Proposal 8*) is the official style guide for Python code, originally written by Guido van Rossum, Barry Warsaw, and Alyssa Coghlan. Its core insight is:

> *"Code is read much more often than it is written."*

Following PEP 8 ensures your code looks clean and consistent with the rest of the Python ecosystem.

### Key PEP 8 Guidelines

1. **Indentation:** Use **4 spaces** per indentation level (never tabs).
2. **Maximum Line Length:** Limit code lines to **79 characters** (and comments/docstrings to **72 characters**). *(Many teams allow 88–100 characters with modern formatters, but 79 is the official PEP 8 limit.)*
3. **Blank Lines:**
   - **2 blank lines** before and after top-level functions and class definitions.
   - **1 blank line** between methods inside a class.
4. **Imports:**
   - Place at the top of the file, one import per line.
   - Group in this order, separated by a blank line:
     1. Standard library imports (`os`, `sys`, `math`)
     2. Third-party library imports (`requests`, `numpy`)
     3. Local application/project imports
5. **Whitespace in Expressions:**
   - Put a single space around binary operators (`x = a + b`).
   - Do **not** put spaces directly inside parentheses, brackets, or braces.
   - Do **not** align `=` signs with extra spaces across multiple lines.
6. **Idiomatic Comparisons:**
   - Use `is None` / `is not None` instead of `== None`.
   - Use truthiness directly (`if items:`) instead of `if len(items) > 0:`.
   - Use `if is_valid:` instead of `if is_valid == True:`.

### Bad vs. Good (PEP 8)

```python
# BAD (violates PEP 8)
import os, sys

def CalcSum( x,y ):
    result=x+y
    if result == None:
        return 0
    return result
```

```python
# GOOD (follows PEP 8)
import os
import sys


def calc_sum(x, y):
    """Return the sum of two numbers."""
    result = x + y
    if result is None:
        return 0
    return result
```

> [!TIP]
> You don't have to memorize every rule manually. Modern tools like **Ruff**, **Black**, and **Flake8** automatically check and format your code to follow PEP 8.

---

## 11. How Python Code is Executed (Bytecode & PVM)

Although Python is commonly called an "interpreted language," the standard **CPython** implementation actually compiles your source code to an intermediate format before executing it.

### Execution Pipeline

```text
  Source Code (.py)
         │
         ▼
  [1. Parser & Compiler]  ──► Checks syntax, builds AST, compiles to bytecode
         │
         ▼
  Bytecode (.pyc in __pycache__/)
         │
         ▼
  [2. Python Virtual Machine (PVM)]  ──► Executes bytecode instructions at runtime
         │
         ▼
  Program Output / Hardware Execution
```

1. **Source Code (`.py`):** You write human-readable Python code.
2. **Compilation to Bytecode:**
   - CPython parses the source into tokens and an **Abstract Syntax Tree (AST)**.
   - It compiles the AST into **bytecode**—a compact, low-level, platform-independent set of instructions for a stack-based virtual machine.
   - When modules are imported, Python caches this bytecode as `.pyc` files inside a `__pycache__/` folder so subsequent runs start faster.
3. **Python Virtual Machine (PVM):**
   - The PVM is the runtime engine of Python (an execution loop written in C in CPython).
   - It reads bytecode instructions one by one and translates them into machine operations.

### Inspecting Bytecode with `dis`

Python includes a built-in disassembler module (`dis`) that lets you view the bytecode generated for any function:

```python
import dis


def add_numbers(a, b):
    return a + b


dis.dis(add_numbers)
```

Running this prints the low-level bytecode instructions (such as `LOAD_FAST`, `BINARY_OP`, and `RETURN_VALUE`) that the PVM executes.

### Common Python Implementations
- **CPython:** The default reference implementation written in C (used by most developers).
- **PyPy:** An alternative implementation with a **JIT (Just-In-Time)** compiler for faster execution of long-running pure-Python code.
- **Jython / IronPython:** Implementations targeting the JVM and .NET runtimes, respectively.
- **MicroPython:** A lean implementation designed for microcontrollers and embedded systems.

---

## 12. Applications of Python

Because of its readability and vast ecosystem on **PyPI** (the Python Package Index), Python is used across nearly every software domain:

| Domain | Popular Libraries / Frameworks | Common Use Cases |
| :--- | :--- | :--- |
| **Web Development** | Django, Flask, FastAPI | REST APIs, backend services, full-stack web apps |
| **Data Science & Analytics** | NumPy, Pandas, Polars, Matplotlib, Seaborn | Data cleaning, statistical analysis, dashboards, visualization |
| **AI & Machine Learning** | PyTorch, TensorFlow, scikit-learn, Hugging Face | Deep learning, predictive models, LLMs, computer vision |
| **Automation & DevOps** | `pathlib`, `subprocess`, Ansible, Playwright, BeautifulSoup | File batching, web scraping, CI/CD, server provisioning |
| **Testing & QA** | `pytest`, `unittest`, Selenium | Unit testing, API testing, browser test automation |
| **Cybersecurity** | Scapy, Cryptography, Pwntools | Network analysis, penetration testing, security tooling |
| **Desktop & Games** | Tkinter, PyQt/PySide, Pygame | Cross-platform desktop utilities, 2D games, prototypes |
| **IoT & Embedded** | MicroPython, CircuitPython | Raspberry Pi projects, sensors, robotics |

### Example: Simple Web API with FastAPI
```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"message": "Hello from Python API!"}
```

### Example: File Automation with `pathlib`
```python
from pathlib import Path

# List all Python files in the current directory
for py_file in Path(".").glob("*.py"):
    print(f"Found script: {py_file.name} ({py_file.stat().st_size} bytes)")
```

---

## Summary & Key Takeaways

- **Python** was created by **Guido van Rossum** (first released in 1991) with readability as a core goal.
- Modern Python is **Python 3** (Python 2 reached End-of-Life in 2020).
- Python is **high-level, interpreted, dynamically and strongly typed**, and ships with a rich standard library.
- Indentation (**4 spaces**) defines code blocks; `#` creates comments; `"""..."""` creates **docstrings**.
- Follow **PEP 8** naming and formatting conventions (`snake_case` for functions/variables, `PascalCase` for classes).
- Under the hood, CPython compiles `.py` files into **bytecode** (`.pyc`) and executes them inside the **Python Virtual Machine (PVM)**.
