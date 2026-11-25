# Python Interview Questions and Answers

A comprehensive, structured collection of **125 Python interview questions and answers** organized by difficulty and topic—covering core fundamentals, intermediate constructs, Python internals and concurrency, and hands-on coding problems.

---

## Table of Contents

- [Basic Level (Questions 1–35)](#basic-level-questions-135)
  - [Python Fundamentals, Data Types, and Variables](#python-fundamentals-data-types-and-variables)
  - [Operators and Control Flow](#operators-and-control-flow)
  - [Strings, Lists, Tuples, Sets, and Dictionaries](#strings-lists-tuples-sets-and-dictionaries)
  - [Basic Functions and Scope](#basic-functions-and-scope)
  - [File Handling Basics](#file-handling-basics)
- [Intermediate Level (Questions 36–70)](#intermediate-level-questions-3670)
  - [Object-Oriented Programming (OOP)](#object-oriented-programming-oop)
  - [Decorators, Generators, and Iterators](#decorators-generators-and-iterators)
  - [Comprehensions](#comprehensions)
  - [Exception Handling](#exception-handling)
  - [Modules and Packages](#modules-and-packages)
  - [Regular Expressions](#regular-expressions)
  - [Lambda, Map, Filter, and Reduce](#lambda-map-filter-and-reduce)
  - [File Handling Advanced](#file-handling-advanced)
  - [Closures and Scope Rules](#closures-and-scope-rules)
- [Advanced Level (Questions 71–102)](#advanced-level-questions-71102)
  - [Metaclasses and Descriptors](#metaclasses-and-descriptors)
  - [GIL, Threading, and Multiprocessing](#gil-threading-and-multiprocessing)
  - [Memory Management and Garbage Collection](#memory-management-and-garbage-collection)
  - [Design Patterns](#design-patterns)
  - [Asyncio and Concurrency](#asyncio-and-concurrency)
  - [Context Managers](#context-managers)
  - [Monkey Patching](#monkey-patching)
  - [Slots and Method Resolution Order (MRO)](#slots-and-method-resolution-order-mro)
  - [Performance Optimization](#performance-optimization)
  - [Python Internals and Modern Features](#python-internals-and-modern-features)
- [Coding Problems (Questions 103–125)](#coding-problems-questions-103125)
  - [String Manipulation Problems](#string-manipulation-problems)
  - [List and Array Problems](#list-and-array-problems)
  - [Dictionary Problems](#dictionary-problems)
  - [Recursion Problems](#recursion-problems)
  - [Pattern Printing](#pattern-printing)
  - [Sorting and Searching](#sorting-and-searching)
  - [Common Algorithmic Problems](#common-algorithmic-problems)

---

## Basic Level (Questions 1–35)

### Python Fundamentals, Data Types, and Variables

#### Q1. What is Python, and what are its key features?

**Answer:**
Python is a high-level, general-purpose, dynamically typed, and interpreted programming language created by Guido van Rossum and first released in 1991. It emphasizes code readability through clean syntax and significant indentation.

Key features include:
- **Readable and expressive syntax:** Uses indentation to define code blocks instead of braces.
- **Dynamically and strongly typed:** Variable types are checked at runtime, and implicit conversions between incompatible types are avoided.
- **Multi-paradigm:** Supports procedural, object-oriented, and functional programming styles.
- **Batteries included:** Ships with a rich standard library (`os`, `pathlib`, `json`, `collections`, `itertools`, `asyncio`, etc.).
- **Automatic memory management:** Uses reference counting and a cyclic garbage collector.
- **Cross-platform and extensible:** Runs across major operating systems and integrates easily with C/C++ extensions.

**Key Points to Remember:**
- Python is both **dynamically typed** and **strongly typed**.
- Code blocks are delimited by **indentation** (PEP 8 recommends 4 spaces).
- Its reference implementation is **CPython** (written in C).

---

#### Q2. Is Python an interpreted or compiled language? How does Python execute code?

**Answer:**
Python is often described as an interpreted language, though technically CPython uses **both compilation and interpretation**:
1. **Compilation to bytecode:** When you run a `.py` file, Python's compiler parses the source code into an Abstract Syntax Tree (AST) and compiles it into platform-independent **bytecode** (`.pyc` files cached in `__pycache__`).
2. **Interpretation via PVM:** The **Python Virtual Machine (PVM)** executes the bytecode instructions one by one at runtime.

```python
import dis

def add(a, b):
    return a + b

# Inspect compiled bytecode instructions
dis.dis(add)
```

**Key Points to Remember:**
- Python compiles source code to bytecode (`.pyc`) automatically before execution—there is no separate manual build step like in C/C++.
- Bytecode is executed by the **Python Virtual Machine (PVM)**.
- Cached `.pyc` files speed up module startup time, not runtime execution speed.

---

#### Q3. Explain dynamic typing vs. strong typing in Python.

**Answer:**
- **Dynamic typing** means variable types are bound to objects at runtime rather than declared on variable names at compile time. A single variable name can reference objects of different types over its lifetime.
- **Strong typing** means Python does not silently coerce unrelated types during operations (unlike JavaScript or PHP). For example, adding an `int` and a `str` raises a `TypeError`.

```python
# Dynamic typing: the variable 'x' can point to any object type
x = 10          # int
x = "hello"     # str

# Strong typing: Python refuses implicit str + int coercion
try:
    result = "Age: " + 25
except TypeError as e:
    print(e)  # can only concatenate str (not "int") to str

# Explicit conversion is required
result = "Age: " + str(25)
```

**Key Points to Remember:**
- Variables in Python are **names/references** bound to objects; **objects** carry types, not variables.
- Type hints (`x: int = 10`) aid static analysis tools (`mypy`, `pyright`) and IDEs, but are not enforced at runtime by CPython.

---

#### Q4. What are Python's built-in data types? Distinguish between mutable and immutable types.

**Answer:**
Python's core built-in data types fall into several categories:
- **Numeric:** `int`, `float`, `complex`
- **Boolean:** `bool` (subclass of `int`)
- **Sequence:** `str`, `list`, `tuple`, `range`
- **Mapping:** `dict`
- **Set:** `set`, `frozenset`
- **Binary:** `bytes`, `bytearray`, `memoryview`
- **Null:** `NoneType` (`None`)

**Mutability** determines whether an object's internal state can be modified after creation:
- **Immutable types:** `int`, `float`, `complex`, `bool`, `str`, `tuple`, `frozenset`, `bytes`, `NoneType`. Any "modification" creates a new object in memory.
- **Mutable types:** `list`, `dict`, `set`, `bytearray`, and most custom class instances. Their contents can be changed in place without changing their `id()`.

```python
# Immutable: new object created
s = "hello"
old_id = id(s)
s += " world"
print(id(s) == old_id)  # False

# Mutable: modified in place
nums = [1, 2, 3]
old_id = id(nums)
nums.append(4)
print(id(nums) == old_id)  # True
```

**Key Points to Remember:**
- Only **immutable** (specifically hashable) objects can be dictionary keys or set elements.
- A `tuple` itself is immutable, but if it contains a mutable element like a `list` (e.g., `([1, 2], 3)`), that inner list can still be mutated (and such a tuple is not hashable).

---

#### Q5. What is the difference between `is` and `==`? Explain small integer caching and string interning.

**Answer:**
- `==` checks **value equality** (whether two objects have the same value, invoking `__eq__`).
- `is` checks **object identity** (whether two variables refer to the exact same object in memory, i.e., `id(a) == id(b)`).

CPython applies two optimizations that often confuse beginners:
- **Small integer caching:** CPython pre-allocates integers from `-5` to `256`, so variables holding those integers often share the same object identity.
- **String interning:** Short/identifier-like string literals may be interned and reused automatically.

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True  (same values)
print(a is b)  # False (different list objects in memory)

# Small integer caching (-5 to 256)
x = 256
y = 256
print(x is y)  # True

m = 1000
n = int("1000")
print(m == n)  # True
print(m is n)  # False
```

**Key Points to Remember:**
- Always use `is` / `is not` when comparing with `None`, `True`, or `False` (especially `if x is None:`).
- Always use `==` for comparing numbers, strings, and data structures by value.
- Never rely on `is` for integer or string equality—caching/interning is an implementation detail.

---

#### Q6. How do variables work in Python? Explain references, `id()`, and unpacking.

**Answer:**
In Python, assignment (`=`) does not copy data into a memory box; it binds a name (reference) to an object in memory. Multiple names can reference the same object.
- `id(obj)` returns the unique integer identifier of an object (its memory address in CPython).
- Python supports **multiple assignment** and **iterable unpacking** (including starred unpacking `*`).

```python
a = [1, 2]
b = a           # b points to the exact same list object
b.append(3)
print(a)        # [1, 2, 3]
print(id(a) == id(b))  # True

# Unpacking and swapping
x, y = 10, 20
x, y = y, x     # Swap without a temporary variable

# Extended unpacking
first, *middle, last = [1, 2, 3, 4, 5]
print(first, middle, last)  # 1 [2, 3, 4] 5
```

**Key Points to Remember:**
- Assignment creates a reference, **never** a copy.
- `x, y = y, x` evaluates the right-hand side tuple `(y, x)` first, then unpacks it.
- `*rest` in unpacking always produces a `list`.

---

#### Q7. What is `None` in Python, and how does it differ from `False`, `0`, or `""`?

**Answer:**
`None` is the sole instance of `NoneType` and represents the **absence of a value** or a null reference.
- Unlike `False` (`bool`), `0` (`int`), or `""` (`str`), `None` is a distinct singleton object.
- Although `None` evaluates to `False` in a boolean context (`bool(None)` is `False`), it is **not equal** to `False`, `0`, or `""`.

```python
print(type(None))        # <class 'NoneType'>
print(None == False)     # False
print(None == 0)         # False
print(None == "")        # False
print(bool(None))        # False

# Standard way to check for None
val = None
if val is None:
    print("Value is missing")
```

**Key Points to Remember:**
- Functions without an explicit `return` statement return `None` by default.
- Use `is None` or `is not None` instead of `== None` (PEP 8 recommendation and avoids custom `__eq__` surprises).

---

#### Q8. Explain type conversion (implicit vs. explicit) and truthy/falsy values in Python.

**Answer:**
- **Implicit conversion:** Python automatically promotes compatible numeric types without data loss (e.g., `int` + `float` -> `float`, `bool` + `int` -> `int`).
- **Explicit conversion (casting):** Done using constructors like `int()`, `float()`, `str()`, `list()`, `tuple()`, `set()`, `dict()`, `bool()`.
- **Truthy and Falsy:** Every Python object can be evaluated in a boolean context (`if`, `while`, `bool()`).
  - **Falsy values:** `False`, `None`, `0`, `0.0`, `0j`, `""`, `[]`, `()`, `{}`, `set()`, `range(0)`, or custom objects whose `__bool__()` returns `False` or `__len__()` returns `0`.
  - **Truthy values:** Everything else (including `"False"`, `"0"`, `[0]`).

```python
# Implicit conversion
print(5 + 2.5)      # 7.5 (float)
print(True + 4)     # 5   (bool is a subclass of int)

# Explicit conversion
print(int("42"))    # 42
print(bool("0"))    # True (non-empty string is truthy!)
print(bool([]))     # False (empty list is falsy)
```

**Key Points to Remember:**
- Non-empty containers and non-empty strings—even `"False"` or `[0]`—are **truthy**.
- Check emptiness idiomatically with `if not items:` rather than `if len(items) == 0:`.

---

### Operators and Control Flow

#### Q9. What is the difference between `/`, `//`, and `%` in Python, especially with negative numbers?

**Answer:**
- `/` (**True division**): Always returns a `float` (e.g., `4 / 2` -> `2.0`).
- `//` (**Floor division**): Divides and rounds **down toward negative infinity** (mathematical floor, not truncation toward zero).
- `%` (**Modulo**): Returns the remainder such that `a == (a // b) * b + (a % b)`, so the result has the **same sign as the divisor `b`**.

```python
print(7 / 2)     # 3.5
print(7 // 2)    # 3
print(-7 // 2)   # -4 (floored toward -infinity, NOT -3)
print(-7 % 2)    # 1  (because -4 * 2 + 1 = -7)
print(7 % -2)    # -1 (same sign as divisor -2)
```

**Key Points to Remember:**
- `//` performs **floor** division (toward $-\infty$), unlike C/Java integer division which truncates toward `0`.
- `divmod(a, b)` returns `(a // b, a % b)` in a single call.

---

#### Q10. Explain Python's logical operators (`and`, `or`, `not`) and short-circuit evaluation.

**Answer:**
- `not x` returns a `bool` (`True` if `x` is falsy, `False` if `x` is truthy).
- `a and b` and `a or b` **short-circuit**: they stop evaluating as soon as the outcome is determined, and they **return one of the actual operand values**, not necessarily a `bool`:
  - `a and b`: Returns `a` if `a` is falsy; otherwise evaluates and returns `b`.
  - `a or b`: Returns `a` if `a` is truthy; otherwise evaluates and returns `b`.

```python
print("" or "default")     # "default"
print("user" or "default") # "user"
print(0 and 100)           # 0
print(5 and 100)           # 100

# Short-circuiting prevents ZeroDivisionError
x = 0
if x != 0 and (10 / x) > 1:
    print("Safe")
```

**Key Points to Remember:**
- `and` / `or` return the **last evaluated operand**, preserving its type.
- Short-circuiting is commonly used for safe guards (`obj is not None and obj.valid`) and fallback defaults (`name or "Guest"`—though beware if `""` or `0` is a valid value).

---

#### Q11. What are bitwise operators, membership operators, and identity operators?

**Answer:**
- **Bitwise operators** work on binary representations of integers: `&` (AND), `|` (OR), `^` (XOR), `~` (NOT / two's complement `-(x+1)`), `<<` (left shift), `>>` (right shift).
- **Membership operators** (`in`, `not in`) test whether an item exists in a sequence, set, or dictionary (checks keys for `dict`).
- **Identity operators** (`is`, `is not`) compare object memory identity (`id()`).

```python
# Bitwise
print(5 & 3)    # 1  (0101 & 0011 = 0001)
print(5 ^ 3)    # 6  (0101 ^ 0011 = 0110)
print(1 << 3)   # 8  (multiply by 2^3)

# Membership
d = {"a": 1, "b": 2}
print("a" in d)      # True  (checks keys)
print(1 in d)        # False (values are not checked)
```

**Key Points to Remember:**
- `in` is $O(1)$ average time for `set` and `dict`, and $O(n)$ for `list` and `tuple`.
- Bitwise `~x` equals `-(x + 1)` because Python integers have arbitrary precision with conceptual infinite sign extension.

---

#### Q12. How do `break`, `continue`, and `pass` differ?

**Answer:**
- `break`: Immediately terminates the nearest enclosing loop (`for` or `while`).
- `continue`: Skips the rest of the current loop iteration and jumps to the next iteration.
- `pass`: A null operation (no-op) used purely as a syntactic placeholder where a statement is required.

```python
for i in range(6):
    if i == 1:
        pass        # Does nothing; loop continues this iteration
    if i == 2:
        continue    # Skips print(2)
    if i == 4:
        break       # Exits the loop completely
    print(i, end=" ")
# Output: 0 1 3
```

**Key Points to Remember:**
- `pass` is commonly used in empty stub functions, classes, or exception blocks during scaffolding (sometimes `...` / `Ellipsis` is also used).
- `break` prevents a loop's `else` clause from executing.

---

#### Q13. How does the `else` clause work with `for` and `while` loops in Python?

**Answer:**
In Python, a `for` or `while` loop can have an optional `else` block. The `else` block executes **only if the loop finished normally**—meaning it was **not** terminated by a `break` statement (and did not exit via `return` or an unhandled exception). Think of it as `"no break"`.

```python
def find_even(nums):
    for n in nums:
        if n % 2 == 0:
            print(f"Found even: {n}")
            break
    else:
        print("No even numbers found")

find_even([1, 3, 5])  # Output: No even numbers found
find_even([1, 2, 5])  # Output: Found even: 2
```

**Key Points to Remember:**
- Loop `else` runs even if the iterable is empty (e.g., `for x in []: ... else: ...` runs `else`).
- Think of `for ... else:` as **`if not broken:`**; it eliminates the need for manual boolean `found = False` flags in search loops.

---

#### Q14. What is the walrus operator (`:=`), and when should it be used?

**Answer:**
Introduced in Python 3.8 (PEP 572), `:=` is the **assignment expression** operator (colloquially called the walrus operator). It assigns a value to a variable **and returns that value** as part of a larger expression.

```python
# Without walrus operator
line = input("Enter text: ")
while line != "quit":
    print(f"You entered: {line}")
    line = input("Enter text: ")

# With walrus operator
while (line := input("Enter text: ")) != "quit":
    print(f"You entered: {line}")

# Useful in comprehensions to avoid calling expensive functions twice
data = ["apple", "banana", "pear"]
lengths = [n for s in data if (n := len(s)) > 4]
print(lengths)  # [5, 6]
```

**Key Points to Remember:**
- Often requires parentheses around `(var := expr)` when combined with comparison operators due to operator precedence.
- Great for `while` loops, `if` regex matches (`if (m := pattern.search(s)):`), and avoiding redundant computation in comprehensions.

---

### Strings, Lists, Tuples, Sets, and Dictionaries

#### Q15. Explain string immutability and common string methods (`split`, `join`, `strip`, `replace`, `find` vs. `index`).

**Answer:**
Strings (`str`) in Python are **immutable sequences of Unicode code points**. Any string method returns a new string rather than mutating the original.
- `split(sep)`: Splits string into a list of substrings (default splits on any whitespace and strips outer whitespace).
- `sep.join(iterable)`: Concatenates an iterable of strings using `sep` as delimiter. Much faster than `+=` in a loop ($O(n)$ vs $O(n^2)$).
- `strip()` / `lstrip()` / `rstrip()`: Removes leading/trailing whitespace (or specified characters).
- `replace(old, new, count)`: Returns a copy with occurrences replaced.
- `find(sub)` vs. `index(sub)`: Both return the lowest index of `sub`, except `find()` returns `-1` if not found while `index()` raises `ValueError`.

```python
text = "  python,java,go  "
cleaned = text.strip()                  # "python,java,go"
parts = cleaned.split(",")              # ['python', 'java', 'go']
rejoined = " | ".join(parts)            # "python | java | go"

print(cleaned.find("rust"))             # -1
# cleaned.index("rust")                 # Raises ValueError
```

**Key Points to Remember:**
- Use `"".join(parts)` instead of string concatenation (`+=`) inside loops.
- `strip("abc")` removes **any combination** of `'a'`, `'b'`, `'c'` from both ends—not the substring `"abc"`. Use `removeprefix()` / `removesuffix()` (Python 3.9+) for exact prefix/suffix removal.

---

#### Q16. Compare f-strings, `str.format()`, and `%` formatting.

**Answer:**
Python supports three primary string formatting mechanisms:
1. **f-strings (Python 3.6+, PEP 498):** Fast, readable, evaluated at runtime with inline expressions (`f"{expr:.2f}"`). Supports `f"{var=}"` debugging syntax (Python 3.8+).
2. **`str.format()`:** Useful when the template string is defined separately from the values (e.g., i18n templates).
3. **`%` formatting:** Legacy C-style formatting; still seen in `logging` calls (where lazy formatting avoids string interpolation unless logged) and older codebases.

```python
name = "Alice"
score = 93.456

print(f"{name} scored {score:.1f}%")       # Alice scored 93.5%
print(f"{score=}")                          # score=93.456
print("{} scored {:.1f}%".format(name, score))
print("%s scored %.1f%%" % (name, score))
```

**Key Points to Remember:**
- **f-strings** are the fastest and most readable option in modern Python.
- Python 3.12 lifted previous restrictions on f-strings (you can now reuse the same quotes, add comments, and include backslashes inside f-string expressions).
- Never use f-strings for SQL queries (use parameterized queries to prevent SQL injection).

---

#### Q17. How does slicing work in Python sequences (`seq[start:stop:step]`)?

**Answer:**
Slicing extracts a subsequence using `[start:stop:step]`:
- `start`: Inclusive starting index (defaults to `0` if `step > 0`, or `-1` if `step < 0`).
- `stop`: Exclusive ending index (defaults to `len(seq)` if `step > 0`, or before index `0` if `step < 0`).
- `step`: Stride/direction (defaults to `1`; negative steps traverse backward).
- Out-of-bounds indices in slices are gracefully clamped instead of raising `IndexError`.

```python
nums = [0, 1, 2, 3, 4, 5]
print(nums[1:4])     # [1, 2, 3]
print(nums[-3:])     # [3, 4, 5]
print(nums[::2])     # [0, 2, 4]
print(nums[::-1])    # [5, 4, 3, 2, 1, 0] (reversed copy)
print(nums[2:100])   # [2, 3, 4, 5] (no IndexError)
```

**Key Points to Remember:**
- `start` is **inclusive**, `stop` is **exclusive**.
- Slicing a `list` creates a **shallow copy**; `seq[::-1]` creates a reversed copy.
- Slice assignment (`nums[1:3] = [10, 20, 30]`) can resize mutable lists in place.

---

#### Q18. Compare Lists and Tuples in Python.

**Answer:**
Both `list` and `tuple` are ordered, indexable sequences that can hold heterogeneous elements, but they differ fundamentally in mutability and intent:

| Feature | `list` | `tuple` |
| :--- | :--- | :--- |
| Syntax | `[1, 2, 3]` | `(1, 2, 3)` or `1, 2, 3` |
| Mutability | Mutable | Immutable |
| Hashable | Never | Yes (if all elements are hashable) |
| Memory & Speed | Over-allocates buffer for growth; slightly heavier | Fixed size; smaller memory footprint & faster creation |
| Semantic Intent | Homogeneous collection of items | Heterogeneous record/struct (e.g., `(x, y)` coordinates) |

```python
# Single-element tuple requires a trailing comma
not_a_tuple = (42)    # int
single_tuple = (42,)  # tuple

# Tuple as a dictionary key
locations = {(37.77, -122.41): "San Francisco"}
```

**Key Points to Remember:**
- A trailing comma makes a tuple, not parentheses (`x = 1,` is a tuple; `(1)` is an `int`).
- Tuples can be dictionary keys and set members when all nested elements are hashable.

---

#### Q19. What is the difference between `append()`, `extend()`, and `insert()` on a list?

**Answer:**
- `list.append(x)`: Adds `x` as a **single element** to the end of the list ($O(1)$ amortized).
- `list.extend(iterable)`: Iterates over `iterable` and appends **each item individually** to the end ($O(k)$ where $k$ is length of iterable).
- `list.insert(i, x)`: Inserts `x` as a single element at index `i`, shifting subsequent elements right ($O(n)$).

```python
a = [1, 2]
a.append([3, 4])   # [1, 2, [3, 4]]

b = [1, 2]
b.extend([3, 4])   # [1, 2, 3, 4]

c = [1, 3]
c.insert(1, 2)     # [1, 2, 3]
```

**Key Points to Remember:**
- All three mutate the list **in place** and return `None` (never do `lst = lst.append(x)`).
- `b += [3, 4]` behaves like `b.extend([3, 4])`, mutating `b` in place.

---

#### Q20. What is the difference between `remove()`, `pop()`, `del`, and `clear()`?

**Answer:**
- `list.remove(val)`: Removes the **first occurrence** matching `val`; raises `ValueError` if not found; returns `None`.
- `list.pop(idx=-1)`: Removes and **returns** the element at `idx` (defaults to the last element); raises `IndexError` if empty or out of range.
- `del obj[idx]` / `del slice` / `del var`: Statement that deletes an item, a slice (`del lst[1:3]`), a dictionary key, or unbinds a variable name. Returns nothing.
- `list.clear()`: Removes all elements from the list in place, leaving `[]`.

```python
nums = [10, 20, 30, 20, 40]
nums.remove(20)       # [10, 30, 20, 40]
last = nums.pop()     # last=40, nums=[10, 30, 20]
del nums[0:2]         # [20]
nums.clear()          # []
```

**Key Points to Remember:**
- `pop()` is the only one that returns the removed value.
- `pop()` at the end is $O(1)$, whereas `pop(0)` and `remove()` are $O(n)$ (use `collections.deque` for $O(1)$ pops from both ends).

---

#### Q21. Explain shallow copy vs. deep copy (`copy.copy()` vs. `copy.deepcopy()`).

**Answer:**
- **Assignment (`b = a`):** Both names point to the same object; no copy is made.
- **Shallow copy (`a.copy()`, `list(a)`, `a[:]`, `copy.copy(a)`):** Creates a **new outer container**, but inserts references to the **same inner objects**. Mutating nested mutable objects affects both copies.
- **Deep copy (`copy.deepcopy(a)`):** Recursively creates new copies of the outer container **and all nested objects**.

```python
import copy

original = [[1, 2], [3, 4]]

shallow = copy.copy(original)
shallow[0][0] = 99
print(original)  # [[99, 2], [3, 4]]  <- inner list mutated!

deep = copy.deepcopy(original)
deep[0][0] = 1
print(original)  # [[99, 2], [3, 4]]  <- untouched!
```

**Key Points to Remember:**
- Shallow copy is fast and sufficient when elements are immutable (e.g., list of ints/strings).
- `copy.deepcopy()` handles cyclic references via an internal `memo` dictionary to avoid infinite recursion.

---

#### Q22. What is a `set` and a `frozenset`? What operations do sets support?

**Answer:**
A `set` is an **unordered collection of unique, hashable elements** implemented using a hash table. A `frozenset` is its **immutable** counterpart (meaning a `frozenset` is itself hashable and can be placed inside another `set` or used as a `dict` key).

Sets support mathematical set operations:
- Union: `a | b` or `a.union(b)`
- Intersection: `a & b` or `a.intersection(b)`
- Difference: `a - b` or `a.difference(b)`
- Symmetric difference: `a ^ b` or `a.symmetric_difference(b)`

```python
a = {1, 2, 3, 3}   # {1, 2, 3}
b = {3, 4, 5}

print(a | b)       # {1, 2, 3, 4, 5}
print(a & b)       # {3}
print(a - b)       # {1, 2}
print(a ^ b)       # {1, 2, 4, 5}

fs = frozenset([1, 2])
nested = {fs}      # Valid because frozenset is hashable
```

**Key Points to Remember:**
- `{}` creates an empty **dict**, NOT an empty set! Use `set()` for an empty set.
- Membership testing (`x in s`) is $O(1)$ average time.

---

#### Q23. How do dictionaries work in Python? Why must dictionary keys be hashable?

**Answer:**
A Python `dict` is a mapping of `key: value` pairs backed by a **compact hash table**.
When inserting or looking up `key`:
1. Python computes `hash(key)` to find the target slot in the hash table.
2. If slots collide, Python probes for another slot and checks equality (`==`) against existing keys.

Keys **must be hashable** (implementing `__hash__()` and `__eq__()`, with hash remaining constant over the object's lifetime). If a mutable object like a `list` were allowed as a key and later mutated, its hash would change, breaking hash-table lookup.

```python
d = {"name": "Alice", (1, 2): "point"}
print(hash("name"))

# Lists are unhashable
try:
    bad = {[1, 2]: "value"}
except TypeError as e:
    print(e)  # unhashable type: 'list'
```

**Key Points to Remember:**
- Average time complexity for lookup, insert, and delete in a `dict` is **$O(1)$**.
- Hashability rule: if `a == b`, then `hash(a) == hash(b)` must hold.
- Custom classes are hashable by default (based on `id()`), unless you define `__eq__` without defining `__hash__`.

---

#### Q24. What is the difference between `dict[key]` and `dict.get(key, default)`, and how do `setdefault()` and `update()` work?

**Answer:**
- `d[key]`: Raises `KeyError` if `key` is absent.
- `d.get(key, default=None)`: Returns `default` if `key` is absent without modifying `d`.
- `d.setdefault(key, default=None)`: Returns `d[key]` if present; **if absent, inserts `d[key] = default` and returns `default`**.
- `d.update(other)` / `d |= other` (3.9+): Merges key-value pairs from `other` into `d`, overwriting existing keys.

```python
counts = {"a": 1}
print(counts.get("b", 0))         # 0 (counts unchanged)

groups = {}
groups.setdefault("fruits", []).append("apple")
print(groups)                     # {'fruits': ['apple']}

# Dict merge operator (Python 3.9+)
merged = {"a": 1, "b": 2} | {"b": 20, "c": 3}  # {'a': 1, 'b': 20, 'c': 3}
```

**Key Points to Remember:**
- Use `collections.defaultdict` instead of `setdefault` in tight loops—`setdefault` evaluates the default expression (e.g., `[]`) on *every* call even if the key exists.
- Python 3.9 introduced `|` (merge) and `|=` (in-place update) for dictionaries.

---

#### Q25. Are Python dictionaries ordered? Explain insertion order preservation.

**Answer:**
**Yes.** Starting in **CPython 3.6** (as an implementation detail) and officially guaranteed by the language specification in **Python 3.7+**, dictionaries preserve **insertion order**. Iterating over keys, values, or items yields them in the order keys were first inserted.

Updating an existing key's value does not change its position; deleting and re-inserting a key moves it to the end.

```python
d = {"first": 1, "second": 2, "third": 3}
d["second"] = 200  # Order remains first, second, third
del d["first"]
d["first"] = 1     # Now at the end: second, third, first
print(list(d.keys()))  # ['second', 'third', 'first']
```

**Key Points to Remember:**
- `dict` equality (`d1 == d2`) still ignores order (`{"a": 1, "b": 2} == {"b": 2, "a": 1}` is `True`), whereas `collections.OrderedDict` equality considers order.
- `OrderedDict` is still useful when you need `move_to_end()` or `popitem(last=False)`.

---

### Basic Functions and Scope

#### Q26. Explain positional, keyword, default, `*args`, and `**kwargs` arguments in Python functions.

**Answer:**
Python supports flexible parameter definitions, which must follow this order in a function signature:
`def func(pos_only, /, standard, *args, kw_only, **kwargs):`
- **Positional / Keyword (`standard`):** Can be passed either positionally or by name.
- **Default parameters (`b=10`):** Provide fallback values if omitted (must follow non-default parameters).
- **`*args`:** Collects extra positional arguments into a `tuple`.
- **`**kwargs`:** Collects extra keyword arguments into a `dict`.
- **`/` (Python 3.8+) and `*`:** `/` marks preceding parameters positional-only; `*` marks following parameters keyword-only.

```python
def report(title, /, *items, uppercase=False, **meta):
    text = ", ".join(items)
    if uppercase:
        text = text.upper()
    print(f"{title}: {text} | meta={meta}")

report("Summary", "a", "b", uppercase=True, author="dc")
# Summary: A, B | meta={'author': 'dc'}
```

**Key Points to Remember:**
- `*` and `**` in a function **definition** pack arguments; `*` and `**` in a function **call** unpack iterables/dicts into arguments.
- Positional arguments must always come before keyword arguments in a function call.

---

#### Q27. Why should you avoid mutable default arguments in Python functions, and how do you fix it?

**Answer:**
Default parameter values in Python are evaluated **once when the `def` statement is executed**, not each time the function is called. If the default value is a mutable object (like `[]` or `{}`), every call that relies on the default shares and mutates the **exact same object**.

**Fix:** Use `None` as the sentinel default value and create a new mutable object inside the function body.

```python
# Buggy: shared list across calls
def add_item_bad(item, items=[]):
    items.append(item)
    return items

print(add_item_bad(1))  # [1]
print(add_item_bad(2))  # [1, 2]  <- unexpected!

# Correct: use None sentinel
def add_item_good(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

print(add_item_good(1))  # [1]
print(add_item_good(2))  # [2]
```

**Key Points to Remember:**
- Function defaults are stored in `func.__defaults__` at definition time.
- The `None` sentinel pattern is the universal Python idiom for optional mutable defaults (and `dataclasses` uses `field(default_factory=list)` for the same reason).

---

#### Q28. What is the difference between `return`, `yield`, and having no return statement in a function?

**Answer:**
- **`return expr`:** Immediately exits the function and sends `expr` back to the caller. Returning multiple comma-separated values (`return a, b`) returns a `tuple`.
- **No `return` (or bare `return`):** Returns `None` implicitly.
- **`yield expr`:** Turns the function into a **generator function**. Calling it returns a generator object without executing the body immediately; each `next()` resumes execution up to the next `yield`, pausing and preserving local state.

```python
def normal_fn():
    return 1, 2

def gen_fn():
    yield 1
    yield 2

print(normal_fn())       # (1, 2)
print(list(gen_fn()))    # [1, 2]
```

**Key Points to Remember:**
- A function containing `yield` anywhere in its body is always a generator function—even if `yield` is in an unreachable branch.
- Inside a generator, `return value` raises `StopIteration(value)` instead of returning `value` directly to `next()`.

---

#### Q29. What is variable scope in Python? Explain local vs. global scope and the `global` keyword.

**Answer:**
- **Local scope:** Variables assigned inside a function belong to that function's local namespace and cannot be accessed outside it.
- **Global scope:** Variables defined at the top level of a module are global to that module and can be **read** inside functions.
- **Assigning inside a function:** Assigning `x = ...` inside a function creates a **new local variable** unless declared `global x`. Attempting to read and then assign a global variable without `global` raises `UnboundLocalError`.

```python
count = 0

def increment():
    global count
    count += 1

increment()
print(count)  # 1

# UnboundLocalError example:
# def bad():
#     print(count)  # Python sees `count =` below, treats count as local
#     count = 5
```

**Key Points to Remember:**
- Python determines variable scope **at compile time** for the whole function body: any assignment makes a variable local unless `global` or `nonlocal` is declared.
- Modifying a mutable global (e.g., `global_list.append(1)`) does **not** require `global`, because it mutates the object without rebinding the name.

---

#### Q30. What are docstrings and type hints, and how do you inspect them?

**Answer:**
- **Docstrings:** A string literal placed as the first statement of a module, class, or function, stored in `__doc__` and displayed by `help()`. (PEP 257).
- **Type hints (annotations):** Optional syntax (`def greet(name: str) -> str:`) introduced in PEP 484 to document expected parameter and return types, stored in `__annotations__`. They are ignored at runtime by CPython and checked statically by tools like `mypy` or `ruff`/`pyright`.

```python
def add(a: int, b: int) -> int:
    """Return the sum of two integers."""
    return a + b

print(add.__doc__)          # Return the sum of two integers.
print(add.__annotations__)  # {'a': <class 'int'>, 'b': <class 'int'>, 'return': <class 'int'>}
```

**Key Points to Remember:**
- Comments (`#`) are stripped by the parser; docstrings are retained as `__doc__` attributes.
- Type hints do not enforce types at runtime unless paired with a runtime validator like `pydantic` or `typeguard`.

---

#### Q31. What is the difference between `sorted()` and `list.sort()`?

**Answer:**
- **`list.sort(key=None, reverse=False)`:** A `list` method that sorts the list **in place** and returns `None`. Only works on `list` objects.
- **`sorted(iterable, key=None, reverse=False)`:** A built-in function that accepts **any iterable** (tuple, dict, set, generator), leaves the original unchanged, and returns a **new sorted list**.

Both use **Timsort**, a stable hybrid sorting algorithm (Merge Sort + Insertion Sort) with $O(n \log n)$ worst-case time complexity.

```python
nums = [3, 1, 2]
new_nums = sorted(nums)  # [1, 2, 3]; nums is still [3, 1, 2]

ret = nums.sort()        # nums is now [1, 2, 3]; ret is None
```

**Key Points to Remember:**
- Both `sort()` and `sorted()` are **stable** (items with equal keys keep their original relative order).
- `list.sort()` uses less memory because it operates in place.

---

### File Handling Basics

#### Q32. How do you open, read, and write files in Python? Explain file modes.

**Answer:**
Python's built-in `open(file, mode='r', encoding=None)` returns a file object. Common modes include:
- `'r'`: Read (default; raises `FileNotFoundError` if missing).
- `'w'`: Write (creates file or **truncates** existing file).
- `'a'`: Append (creates file or writes at end without truncating).
- `'x'`: Exclusive creation (fails with `FileExistsError` if file already exists).
- `'b'` / `'t'`: Binary mode (e.g., `'rb'`, `'wb'`) vs. text mode (default `'t'`).
- `'+'`: Read and write together (e.g., `'r+'`, `'w+'`).

```python
with open("example.txt", "w", encoding="utf-8") as f:
    f.write("Hello, Python!\n")

with open("example.txt", "r", encoding="utf-8") as f:
    content = f.read()
```

**Key Points to Remember:**
- Always specify `encoding="utf-8"` when working with text files to avoid OS-dependent encoding differences (e.g., Windows `cp1252`).
- Mode `'w'` immediately erases existing file contents on open.

---

#### Q33. Why is the `with` statement recommended when working with files?

**Answer:**
The `with` statement invokes the file object's **context manager protocol** (`__enter__` and `__exit__`). It guarantees that `file.close()` is automatically called when leaving the `with` block—**even if an exception occurs** inside the block. Without `with`, forgetting `close()` or hitting an exception before `close()` can leak file descriptors and leave buffered writes unflushed.

```python
# Recommended: guaranteed cleanup even on exceptions
with open("data.txt", "w", encoding="utf-8") as f:
    f.write("important data")
# File is automatically closed here
print(f.closed)  # True
```

**Key Points to Remember:**
- `with` is safer and cleaner than manual `try ... finally: f.close()`.
- You can open multiple files in one statement: `with open("in.txt") as src, open("out.txt", "w") as dst:`.

---

#### Q34. Compare `read()`, `readline()`, `readlines()`, and iterating over a file object.

**Answer:**
- `f.read(size=-1)`: Reads the entire file (or up to `size` chars/bytes) into a single string/bytes object. High memory risk on huge files.
- `f.readline()`: Reads one line at a time (including trailing `\n`), returning `""` at EOF.
- `f.readlines()`: Reads all remaining lines into a `list` of strings.
- **Iterating directly (`for line in f:`):** Memory-efficient lazy reading line by line using internal buffering—the preferred way to process large text files.

```python
with open("log.txt", "r", encoding="utf-8") as f:
    for line in f:  # Streams line by line without loading whole file
        if "ERROR" in line:
            print(line.strip())
```

**Key Points to Remember:**
- Prefer `for line in f:` over `f.readlines()` for large files.
- Note that `readline()` and iteration retain the trailing newline `\n`.

---

#### Q35. How do you check if a file or directory exists and work with paths using `pathlib` vs. `os.path`?

**Answer:**
Modern Python (3.4+) recommends `pathlib.Path`, which represents filesystem paths as objects rather than raw strings, making path joining and querying cleaner and cross-platform.

```python
from pathlib import Path
import os

# Modern pathlib approach
p = Path("Python_Notes") / "notes.md"
print(p.exists())
print(p.is_file(), p.is_dir())
print(p.name, p.stem, p.suffix, p.parent)

# Legacy os.path approach
legacy_path = os.path.join("Python_Notes", "notes.md")
print(os.path.exists(legacy_path))
```

**Key Points to Remember:**
- `Path` supports the `/` operator for cross-platform path joining (`Path("dir") / "file.txt"`).
- `p.mkdir(parents=True, exist_ok=True)` safely creates nested directories without raising errors if they already exist.

---

## Intermediate Level (Questions 36–70)

### Object-Oriented Programming (OOP)

#### Q36. Explain classes, objects, `self`, and the difference between `__new__` and `__init__`.

**Answer:**
A **class** is a blueprint defining attributes and behaviors; an **object** (instance) is a concrete realization of that class.
- `self` represents the current instance passed automatically as the first argument to instance methods.
- **`__new__(cls, ...)`:** A static method responsible for **allocating and returning** a new instance. Called *before* `__init__`. Used when subclassing immutable types (`int`, `str`, `tuple`) or implementing Singletons.
- **`__init__(self, ...)`:** The initializer that **configures** the already-created instance. It must return `None`.

```python
class UpperStr(str):
    def __new__(cls, value):
        # Must use __new__ because str is immutable
        return super().__new__(cls, value.upper())

s = UpperStr("hello")
print(s)  # HELLO
```

**Key Points to Remember:**
- `__new__` **creates** the object and returns it; `__init__` **initializes** it and returns `None`.
- If `__new__` does not return an instance of `cls`, `__init__` is not invoked.
- `self` is a strong convention, not a reserved keyword.

---

#### Q37. What is the difference between instance variables and class variables?

**Answer:**
- **Class variables:** Defined directly inside the class body and shared across **all instances** of the class.
- **Instance variables:** Typically bound to `self` inside `__init__`, unique to each instance.

Assigning `obj.class_var = new_val` does **not** modify the class variable—it creates a new **instance variable** on `obj` that shadows the class variable. However, mutating a mutable class variable (`obj.shared_list.append(1)`) affects all instances.

```python
class Employee:
    company = "TechCorp"  # Class variable

    def __init__(self, name):
        self.name = name  # Instance variable

e1 = Employee("Alice")
e2 = Employee("Bob")
e1.company = "OtherCorp"  # Shadows class var on e1 only

print(e1.company, e2.company, Employee.company)
# OtherCorp TechCorp TechCorp
```

**Key Points to Remember:**
- Attribute lookup order on an instance: **instance `__dict__` -> class `__dict__` -> parent classes (MRO)** (ignoring data descriptors for a moment).
- Avoid mutable class variables unless intentionally sharing state across all instances.

---

#### Q38. Compare instance methods, `@classmethod`, and `@staticmethod`.

**Answer:**
- **Instance method:** Receives `self` (the instance); can access/modify both instance and class state.
- **`@classmethod`:** Receives `cls` (the class); cannot access instance state, but can access/modify class state and serve as an **alternative constructor** (factory method), working properly with subclasses.
- **`@staticmethod`:** Receives neither `self` nor `cls`; behaves like a plain function namespaced inside the class for logical grouping.

```python
from datetime import date

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @classmethod
    def from_birth_year(cls, name, year):
        return cls(name, date.today().year - year)

    @staticmethod
    def is_adult(age):
        return age >= 18

p = Person.from_birth_year("Sam", 2000)
print(Person.is_adult(p.age))  # True
```

**Key Points to Remember:**
- Prefer `@classmethod` over `@staticmethod` for factory constructors because `cls(...)` instantiates the actual subclass if inherited.
- Both `@classmethod` and `@staticmethod` can be called on either the class or an instance.

---

#### Q39. Explain inheritance, `super()`, and method overriding in Python.

**Answer:**
**Inheritance** allows a child (derived) class to reuse and extend attributes and methods of one or more parent (base) classes.
- **Method overriding:** Redefining a parent method in a subclass to customize behavior.
- **`super()`:** Returns a proxy object that delegates method calls to the next class in the **Method Resolution Order (MRO)**—critical for cooperative multiple inheritance.

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "..."

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)  # Delegate to Animal.__init__
        self.breed = breed

    def speak(self):
        return f"{self.name} says Woof!"

d = Dog("Rex", "Labrador")
print(d.speak())  # Rex says Woof!
```

**Key Points to Remember:**
- Python supports **multiple inheritance** (`class C(A, B):`).
- Always use `super().__init__(...)` rather than hardcoding `Parent.__init__(self, ...)`, so diamond inheritance works properly without calling shared bases twice.

---

#### Q40. What is polymorphism and duck typing in Python?

**Answer:**
- **Polymorphism** means different types expose a common interface so code can operate on them uniformly.
- **Duck typing** ("If it walks like a duck and quacks like a duck, it's a duck"): Python focuses on an object's **behavior (methods/attributes)** rather than its explicit class hierarchy. Function/operator overloading in the traditional Java/C++ sense (multiple functions with the same name and different parameter types) is replaced in Python by default arguments, `*args`, `functools.singledispatch`, and duck typing.

```python
class FileLogger:
    def write(self, msg): print(f"File: {msg}")

class ConsoleLogger:
    def write(self, msg): print(f"Console: {msg}")

def log_message(logger, msg):
    # Works for any object with a .write() method, no base class needed
    logger.write(msg)

log_message(FileLogger(), "Started")
log_message(ConsoleLogger(), "Started")
```

**Key Points to Remember:**
- Operator overloading in Python is polymorphism via dunder methods (`__add__`, `__len__`, etc.).
- `typing.Protocol` brings static type-checking support for duck typing (structural subtyping).

---

#### Q41. How does encapsulation work in Python? Explain `_attr`, `__attr` (name mangling), and `@property`.

**Answer:**
Python does not enforce strict `private` access modifiers; instead it follows `"we're all consenting adults here"` conventions:
- **Public (`attr`):** Accessible anywhere.
- **Protected convention (`_attr`):** Single leading underscore signals "internal use; do not touch externally," though Python does not block access (also skipped by `from module import *`).
- **Name mangling (`__attr`):** Double leading underscore (with at most one trailing underscore) triggers **name mangling** to `_ClassName__attr` to prevent accidental collisions in subclasses—not true security.
- **`@property`:** Exposes getter/setter/deleter logic behind normal attribute access syntax (`obj.balance`).

```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance  # Mangled to _BankAccount__balance

    @property
    def balance(self):
        return self.__balance

    @balance.setter
    def balance(self, value):
        if value < 0:
            raise ValueError("Balance cannot be negative")
        self.__balance = value

acc = BankAccount(100)
acc.balance = 200
print(acc.balance)               # 200
print(acc._BankAccount__balance) # 200 (still reachable if mangled name is used)
```

**Key Points to Remember:**
- Use `@property` instead of Java-style `get_x()` / `set_x()` methods—you can start with plain public attributes and add `@property` validation later without breaking callers.
- `__attr__` (dunders) are reserved for Python's special methods and are **not** name-mangled.

---

#### Q42. What are magic/dunder methods? Explain `__str__` vs `__repr__`, `__eq__`, `__hash__`, `__len__`, and `__call__`.

**Answer:**
**Dunder (double-underscore) methods** let custom classes integrate seamlessly with Python's built-in syntax, operators, and functions:
- `__repr__(self)`: Unambiguous, developer-oriented representation (ideally valid Python to recreate the object); returned by `repr()` and used as fallback if `__str__` is missing.
- `__str__(self)`: Human-readable representation; used by `str()` and `print()`.
- `__eq__(self, other)` & `__hash__(self)`: Control `==` equality and hashability for sets/dicts.
- `__len__(self)`: Enables `len(obj)` (and truthiness fallback when `__bool__` is absent).
- `__call__(self, *args)`: Makes an instance callable like a function (`obj()`).

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __repr__(self):
        return f"Multiplier(factor={self.factor!r})"

    def __str__(self):
        return f"x{self.factor}"

    def __call__(self, value):
        return value * self.factor

double = Multiplier(2)
print(repr(double))   # Multiplier(factor=2)
print(str(double))    # x2
print(double(15))     # 30
```

**Key Points to Remember:**
- Always implement `__repr__` on classes; implement `__str__` only if you want a friendlier display format.
- Defining `__eq__` without `__hash__` automatically sets `__hash__ = None`, making instances unhashable.

---

#### Q43. What are Abstract Base Classes (`abc.ABC`) and why are they used?

**Answer:**
An **Abstract Base Class (ABC)** defines a formal interface/contract that subclasses must implement. Using `abc.ABC` and `@abstractmethod`, Python prevents instantiating any class that has unimplemented abstract methods, raising `TypeError` at instantiation time.

```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def pay(self, amount: float) -> None:
        pass

class StripeProcessor(PaymentProcessor):
    def pay(self, amount: float) -> None:
        print(f"Charged ${amount:.2f} via Stripe")

# PaymentProcessor()  # TypeError: Can't instantiate abstract class
StripeProcessor().pay(49.99)
```

**Key Points to Remember:**
- ABCs enforce structural contracts early (at instantiation) rather than failing later with `AttributeError` or `NotImplementedError`.
- Abstract methods *can* still have a default body invoked via `super().method()`.

---

#### Q44. Compare composition vs. inheritance. When should you prefer composition?

**Answer:**
- **Inheritance ("is-a"):** Subclass inherits from a base class (`Car` is a `Vehicle`). Tightly couples subclass to parent implementation details.
- **Composition ("has-a"):** A class holds instances of other classes and delegates work to them (`Car` has an `Engine`).

Prefer **composition over inheritance** when:
- You want to combine orthogonal behaviors without creating a tangled multiple-inheritance hierarchy.
- You want to swap components at runtime (e.g., dependency injection).
- The relationship isn't truly a strict "is-a" specialization.

```python
class Engine:
    def start(self):
        return "Engine started"

class Car:
    def __init__(self, engine: Engine):
        self.engine = engine  # Composition

    def drive(self):
        return f"{self.engine.start()} -> Driving"

print(Car(Engine()).drive())
```

**Key Points to Remember:**
- Deep inheritance trees are brittle; composition keeps classes modular, testable, and decoupled.

---

### Decorators, Generators, and Iterators

#### Q45. What is the difference between an iterable and an iterator? Explain the iterator protocol.

**Answer:**
- **Iterable:** Any object you can loop over (`list`, `str`, `dict`, `range`). It implements `__iter__()`, which returns an **iterator** (or falls back to `__getitem__` starting at index 0).
- **Iterator:** An object representing a stream of data that produces one item at a time. It implements **both**:
  - `__iter__(self)` -> returns `self`
  - `__next__(self)` -> returns the next element or raises `StopIteration` when exhausted.

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        val = self.current
        self.current -= 1
        return val

for n in Countdown(3):
    print(n, end=" ")  # 3 2 1
```

**Key Points to Remember:**
- Every **iterator** is an **iterable**, but not every iterable is an iterator (`list` is iterable, `iter([1, 2])` is an iterator).
- Iterators are **single-use** (exhausted once `StopIteration` is raised); iterables like `list` create a fresh iterator each time `iter()` is called.

---

#### Q46. What are generators in Python? How do `yield` and `yield from` work?

**Answer:**
A **generator** is a concise way to build an iterator using a function with `yield` (instead of manually writing `__iter__` and `__next__`).
- On `yield`, the function pauses, emits the value, and preserves all local variables and instruction pointer until the next `next()` call.
- **`yield from iterable` (PEP 380):** Delegates iteration directly to a sub-generator or iterable, forwarding values, exceptions, and `.send()`/`.throw()` calls transparently.

```python
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

print(list(flatten([1, [2, [3, 4]], 5])))  # [1, 2, 3, 4, 5]
```

**Key Points to Remember:**
- Generators evaluate **lazily** (on demand), making them ideal for huge datasets or infinite streams.
- `yield from` is both cleaner and faster than `for x in subgen: yield x`.

---

#### Q47. Compare generator expressions vs. list comprehensions.

**Answer:**
- **List comprehension (`[x * 2 for x in range(1000000)]`):** Eagerly evaluates and allocates all items in memory at once. Supports indexing (`res[0]`), `len()`, and multiple iterations.
- **Generator expression (`(x * 2 for x in range(1000000))`):** Lazily computes one item at a time with $O(1)$ memory overhead. Supports only single-pass iteration and does not support indexing or `len()`.

```python
import sys

list_comp = [x for x in range(100_000)]
gen_expr = (x for x in range(100_000))

print(sys.getsizeof(list_comp))  # ~800,984 bytes
print(sys.getsizeof(gen_expr))   # ~200 bytes
```

**Key Points to Remember:**
- Pass a generator expression directly to reduction functions without extra parentheses: `sum(x * x for x in nums)`.
- Choose list comprehensions when you need to reuse the result, index into it, or know its length.

---

#### Q48. What is a decorator in Python and how does it work under the hood?

**Answer:**
A **decorator** is a callable (usually a function) that takes another function/class and returns a modified or wrapped callable without altering the original source code.
`@dec` above `def foo():` is purely syntactic sugar for `foo = dec(foo)`.

```python
import time
import functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.6f}s")
        return result
    return wrapper

@timer
def compute(n):
    return sum(range(n))

compute(100_000)
```

**Key Points to Remember:**
- Decorators run at **function definition time** (import time), while the inner `wrapper` runs at **call time**.
- Always accept `*args, **kwargs` in the wrapper and **return `func(*args, **kwargs)`** unless intentionally altering arguments/return values.

---

#### Q49. How do you write a parameterized decorator (decorator with arguments), and why is `functools.wraps` important?

**Answer:**
- A **decorator with arguments** requires **three nested levels**: an outer factory function accepting the configuration arguments, which returns the actual decorator, which returns the wrapper.
- **`@functools.wraps(func)`** copies `__name__`, `__doc__`, `__annotations__`, `__module__`, and sets `__wrapped__` from `func` onto `wrapper`. Without it, `wrapped_func.__name__` becomes `"wrapper"`, breaking introspection, logging, and debugging.

```python
import functools

def repeat(times):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    """Greet someone."""
    print(f"Hello, {name}")

greet("Alice")
print(greet.__name__, greet.__doc__)  # greet Greet someone.
```

**Key Points to Remember:**
- `@repeat(3)` calls `repeat(3)` -> gets `decorator` -> passes `greet` to `decorator(greet)` -> binds `greet = wrapper`.
- `functools.wraps` also enables `inspect.signature()` to see the original function's parameters.

---

#### Q50. Explain class-based decorators and stacking multiple decorators.

**Answer:**
- **Class-based decorator:** Any class implementing `__init__(self, func)` and `__call__(self, *args, **kwargs)` can act as a decorator—useful when tracking stateful metadata like call counts.
- **Stacking decorators:** Decorators execute **bottom-up** when applied (wrapping), and **top-down** when the resulting function is called.
  ```python
  @A
  @B
  def f(): ...
  # Equivalent to: f = A(B(f))
  ```

```python
import functools

class CountCalls:
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self.calls = 0

    def __call__(self, *args, **kwargs):
        self.calls += 1
        return self.func(*args, **kwargs)

@CountCalls
def say_hi():
    print("Hi!")

say_hi()
say_hi()
print(say_hi.calls)  # 2
```

**Key Points to Remember:**
- Note a subtle trap with class-based decorators on **methods inside a class**: if a class doesn't implement `__get__` (descriptor protocol), `self` won't bind automatically to the outer instance!
- Stacked decorators wrap bottom-up (`B` wraps `f`, then `A` wraps `B(f)`).

---

### Comprehensions

#### Q51. Explain list, dictionary, and set comprehensions with filtering and conditional expressions.

**Answer:**
Comprehensions provide concise syntax to construct collections from iterables:
- **Filtering (`if` at the end):** Includes items only when condition holds: `[x for x in data if x > 0]`.
- **Transformation (`if-else` before `for`):** Ternary expression applied to every item: `["even" if x % 2 == 0 else "odd" for x in data]`.

```python
nums = [1, 2, 3, 4, 5]

# List comprehension with filter
evens_sq = [x ** 2 for x in nums if x % 2 == 0]  # [4, 16]

# List comprehension with ternary if-else
labels = ["even" if x % 2 == 0 else "odd" for x in nums]

# Dict comprehension
squares_map = {x: x ** 2 for x in nums}

# Set comprehension
unique_lengths = {len(w) for w in ["hi", "hello", "hey", "hi"]}  # {2, 3, 5}
```

**Key Points to Remember:**
- `if` at the **end** filters elements; `if ... else` at the **beginning** transforms elements.
- Comprehensions run in their own local scope (Python 3), so loop variables do not leak outside.

---

#### Q52. How do nested comprehensions work, and when should you avoid them?

**Answer:**
In a nested comprehension, clauses appear in the **same left-to-right order** as standard nested `for` loops:
`[x for row in matrix for x in row]` corresponds to:
```python
for row in matrix:
    for x in row:
        ...
```
Avoid comprehensions when there are more than two nested loops, complex side effects, or long multi-condition logic—readability always wins over clever one-liners.

```python
matrix = [[1, 2], [3, 4], [5, 6]]

# Flatten a 2D matrix
flat = [num for row in matrix for num in row]  # [1, 2, 3, 4, 5, 6]

# Transpose a 2D matrix (nested comprehension inside outer comprehension)
transposed = [[row[i] for row in matrix] for i in range(len(matrix[0]))]
print(transposed)  # [[1, 3, 5], [2, 4, 6]]
```

**Key Points to Remember:**
- Outer loop comes **first**, inner loop comes **second** when flattening.
- Never use a comprehension purely for side effects like `[print(x) for x in items]` (it wastes memory building a list of `None`s).

---

#### Q53. Why isn't there a "tuple comprehension" in Python?

**Answer:**
Wrapping a comprehension in parentheses `(x for x in range(5))` creates a **generator expression**, not a `tuple`. Because tuples are immutable, creating one via comprehension syntax wasn't given dedicated brackets—if you want a tuple from a comprehension, pass a generator expression to `tuple(...)`.

```python
gen = (x * 2 for x in range(4))
print(type(gen))        # <class 'generator'>

tup = tuple(x * 2 for x in range(4))
print(type(tup), tup)   # <class 'tuple'> (0, 2, 4, 6)
```

**Key Points to Remember:**
- `(x for x in ...)` is always a **generator expression**.
- Use `tuple(x for x in ...)` to build a tuple lazily without creating an intermediate list.

---

### Exception Handling

#### Q54. Explain `try`, `except`, `else`, and `finally` blocks and their execution flow.

**Answer:**
- `try`: Code that may raise an exception.
- `except ExcType as e`: Runs only if a matching exception is raised inside `try`.
- `else`: Runs **only if `try` succeeded with no exceptions** (keeps the `try` block minimal so you don't accidentally catch errors from follow-up code).
- `finally`: **Always runs**—whether an exception occurred, was caught, went uncaught, or a `return`/`break` was executed.

```python
def divide(a, b):
    try:
        res = a / b
    except ZeroDivisionError as e:
        print(f"Error: {e}")
        return None
    else:
        print("Division succeeded")
        return res
    finally:
        print("Cleanup always runs")

print(divide(10, 2))
```

**Key Points to Remember:**
- If `finally` itself contains a `return` statement, it overrides any `return` or unhandled exception from `try`/`except` (usually an anti-pattern!).
- Keep `try` blocks as narrow as possible and put success-only logic in `else`.

---

#### Q55. How do you catch multiple exceptions, chain exceptions (`raise ... from`), and why should you avoid bare `except:`?

**Answer:**
- Catch multiple exception types using a tuple: `except (ValueError, TypeError) as e:`.
- **Exception chaining (`raise NewErr(...) from orig_err`):** Preserves the original traceback as `__cause__` when translating low-level exceptions into domain-specific ones. `raise NewErr(...) from None` suppresses the context chain.
- **Avoid bare `except:`** because it catches `BaseException` subclasses like `KeyboardInterrupt`, `SystemExit`, and `GeneratorExit`, making programs impossible to interrupt cleanly with Ctrl+C and hiding bugs. Use `except Exception as e:` when a broad catch is genuinely needed.

```python
def parse_port(raw: str) -> int:
    try:
        return int(raw)
    except ValueError as err:
        raise RuntimeError(f"Invalid port: {raw!r}") from err
```

**Key Points to Remember:**
- Order `except` blocks from **most specific to most general** (e.g., `FileNotFoundError` before `OSError`).
- Bare `raise` inside an `except` block re-raises the active exception while preserving its original traceback.

---

#### Q56. How do you create custom exception classes in Python?

**Answer:**
Custom exceptions should inherit from `Exception` (never `BaseException` directly) and can optionally store extra attributes for structured error handling.

```python
class InsufficientFundsError(Exception):
    def __init__(self, balance: float, amount: float):
        self.balance = balance
        self.amount = amount
        super().__init__(f"Cannot withdraw {amount}; balance is {balance}")

try:
    raise InsufficientFundsError(50.0, 100.0)
except InsufficientFundsError as e:
    print(e, "| Deficit:", e.amount - e.balance)
```

**Key Points to Remember:**
- Define a module/package base exception (e.g., `class AppError(Exception): pass`) so callers can catch all library errors cleanly.
- Call `super().__init__(message)` so `str(e)` and tracebacks display clearly.

---

#### Q57. Differentiate between `SyntaxError`, `TypeError`, `ValueError`, `KeyError`, `IndexError`, and `AttributeError`.

**Answer:**
- **`SyntaxError`:** Code cannot be parsed before execution begins (e.g., missing colon).
- **`TypeError`:** Operation applied to an object of an inappropriate **type** (e.g., `"a" + 1`, `len(5)`).
- **`ValueError`:** Right type, inappropriate **value** (e.g., `int("abc")`, `math.sqrt(-1)`).
- **`KeyError`:** Dictionary/mapping key lookup not found.
- **`IndexError`:** Sequence index out of range (`[1, 2][5]`).
- **`AttributeError`:** Object has no attribute/method of that name (`(5).append(1)`).

**Key Points to Remember:**
- `KeyError` and `IndexError` both inherit from `LookupError`.
- `int("abc")` raises `ValueError` (right type `str`, wrong content), whereas `int([1])` raises `TypeError` (wrong type `list`).

---

### Modules and Packages

#### Q58. What is the difference between a module and a package? Explain `__init__.py` and namespace packages.

**Answer:**
- **Module:** A single `.py` file containing Python definitions and statements.
- **Package:** A directory containing modules (and optionally subpackages), allowing hierarchical structuring like `mypkg.submod`.
- **`__init__.py`:** Marks a regular package directory, runs initialization code when the package is imported, and can re-export public APIs (e.g., setting `__all__`).
- **Namespace packages (PEP 420, Python 3.3+):** Directories **without** `__init__.py` can still be imported as packages, even spanning multiple filesystem locations.

**Key Points to Remember:**
- Defining `__all__ = ["func_a", "ClassB"]` controls what `from module import *` exports.
- A module is executed and cached in `sys.modules` only on its **first** import; subsequent imports reuse the cached module object.

---

#### Q59. What does `if __name__ == "__main__":` do and why is it important?

**Answer:**
Every Python module has a built-in `__name__` attribute:
- When a file is **run directly** (`python script.py`), Python sets `__name__ = "__main__"`.
- When a file is **imported** by another module (`import script`), Python sets `__name__ = "script"` (its module name).

Guarding executable code inside `if __name__ == "__main__":` allows a file to act both as an importable library (without executing side effects on import) and as a standalone CLI script.

```python
def main():
    print("Running as main program")

if __name__ == "__main__":
    main()
```

**Key Points to Remember:**
- Essential when using `multiprocessing` on Windows/macOS (`spawn` start method), which re-imports the main module in worker processes—without this guard, processes spawn recursively!

---

#### Q60. Explain how Python finds modules (`sys.path`), absolute vs. relative imports, and virtual environments (`venv`).

**Answer:**
When importing `foo`, Python checks:
1. `sys.modules` (already imported cache).
2. Built-in modules.
3. Directories listed in `sys.path` in order: script directory (or CWD) -> `PYTHONPATH` -> standard library -> `site-packages`.

- **Absolute imports:** Specify the full path from project root (`from mypkg.utils import helper`). Preferred by PEP 8 for clarity.
- **Relative imports:** Use leading dots inside a package (`from .utils import helper`, `from ..core import Base`). Cannot be run directly as a top-level script without `-m` (`python -m mypkg.submod`).
- **Virtual environments (`python -m venv .venv`):** Create isolated `site-packages` directories per project so package versions do not conflict globally.

**Key Points to Remember:**
- Never name a local file the same as a standard library or installed module (e.g., `json.py` or `random.py`), as it shadows the stdlib because CWD comes first in `sys.path`.
- Run package modules via `python -m pkg.mod` rather than `python pkg/mod.py`.

---

### Regular Expressions

#### Q61. Compare `re.match()`, `re.search()`, `re.findall()`, and `re.finditer()`.

**Answer:**
- `re.match(pattern, text)`: Matches **only at the beginning** of `text` (returns `Match` or `None`).
- `re.search(pattern, text)`: Scans the entire string and returns the **first** `Match` anywhere, or `None`.
- `re.findall(pattern, text)`: Returns a `list` of all non-overlapping matching strings (or tuples if capturing groups exist).
- `re.finditer(pattern, text)`: Returns a lazy **iterator** of `Match` objects (memory-efficient and provides match indices via `.start()`/`.end()`).

```python
import re

text = "Order #102 and Order #405"
print(re.match(r"\d+", text))                # None (not at start)
print(re.search(r"\d+", text).group())       # '102'
print(re.findall(r"\d+", text))              # ['102', '405']
print([m.span() for m in re.finditer(r"\d+", text)])  # [(7, 10), (22, 25)]
```

**Key Points to Remember:**
- Prefer `re.search` with `^` over `re.match` for explicitness; `re.fullmatch()` ensures the *entire* string matches.

---

#### Q62. Explain regex groups, non-capturing groups, lookarounds, and greedy vs. non-greedy quantifiers.

**Answer:**
- **Capturing groups `(...)` / Named groups `(?P<name>...)`:** Extract sub-matches via `m.group(1)` or `m.group("name")`.
- **Non-capturing groups `(?:...)`:** Group tokens for precedence/quantifiers without capturing into `group()`.
- **Lookahead / Lookbehind:** Zero-width assertions that check context without consuming characters: positive lookahead `(?=...)`, negative lookahead `(?!...)`, positive lookbehind `(?<=...)`, negative lookbehind `(?<!...)`.
- **Greedy vs. Non-greedy:** `*`, `+`, `?` are **greedy** (match as much as possible). Adding `?` (`*?`, `+?`) makes them **non-greedy/lazy** (match as little as possible).

```python
import re

html = "<b>one</b> and <b>two</b>"
print(re.findall(r"<b>.*</b>", html))    # Greedy: ['<b>one</b> and <b>two</b>']
print(re.findall(r"<b>.*?</b>", html))   # Lazy:   ['<b>one</b>', '<b>two</b>']

m = re.search(r"(?P<user>\w+)@(?P<domain>[\w.]+)", "dc@example.com")
print(m.groupdict())  # {'user': 'dc', 'domain': 'example.com'}
```

**Key Points to Remember:**
- `findall()` returns only the captured group strings if the pattern has capturing groups—use `(?:...)` to avoid altering `findall()` output.

---

#### Q63. Why should you use raw strings (`r"..."`) and `re.compile()`?

**Answer:**
- **Raw strings (`r"\d+\n"`):** Prevent Python's string parser from interpreting backslashes (`\b`, `\d`, `\n`) before the regex engine sees them. Without `r""`, `\b` means backspace (`0x08`) instead of a regex word boundary!
- **`re.compile(pattern)`:** Pre-compiles a regex pattern into a `Pattern` object. Useful when reusing the same pattern repeatedly in loops, improving readability and avoiding repeated cache lookups (though `re` internally caches the most recent ~512 patterns).

```python
import re

EMAIL_RE = re.compile(r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$")
print(bool(EMAIL_RE.match("alice@example.com")))  # True
```

**Key Points to Remember:**
- Always write regex patterns as **raw strings** (`r"..."`).

---

### Lambda, Map, Filter, and Reduce

#### Q64. What are `lambda` functions, what are their limitations, and how are they commonly used?

**Answer:**
A `lambda` creates a small, anonymous, single-expression function: `lambda args: expression`.
**Limitations:**
- Restricted to a **single expression** (no statements like `return`, `raise`, `pass`, `try/except`, or multi-line blocks).
- Poor debugging tracebacks (`<lambda>` instead of a descriptive function name).

**Common use:** Inline `key` functions in `sorted()`, `min()`, `max()`, and GUI/event callbacks.

```python
users = [("Alice", 30), ("Bob", 22), ("Charlie", 25)]
by_age = sorted(users, key=lambda u: u[1])
print(by_age)  # [('Bob', 22), ('Charlie', 25), ('Alice', 30)]
```

**Key Points to Remember:**
- Never assign a lambda to a variable (`f = lambda x: x + 1` violates PEP 8); use `def f(x): return x + 1`.
- For simple attribute/index lookups, `operator.itemgetter` or `operator.attrgetter` are faster alternatives to `lambda`.

---

#### Q65. Explain `map()`, `filter()`, and `functools.reduce()`, and compare them to comprehensions.

**Answer:**
- `map(func, iterable)`: Lazily applies `func` to each item and yields results.
- `filter(pred, iterable)`: Lazily yields items for which `pred(item)` is truthy (if `pred` is `None`, filters out falsy items).
- `functools.reduce(func, iterable[, initial])`: Cumulatively applies a two-argument function left-to-right to reduce the iterable to a single value.

```python
from functools import reduce

nums = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x ** 2, nums))           # [1, 4, 9, 16, 25]
evens = list(filter(lambda x: x % 2 == 0, nums))      # [2, 4]
product = reduce(lambda acc, x: acc * x, nums, 1)     # 120
```

**Key Points to Remember:**
- In Python 3, `map()` and `filter()` return lazy **iterators**, not lists.
- Pythonic style generally prefers **list comprehensions / generator expressions** over `map`/`filter` with `lambda`, and built-ins like `sum()`, `math.prod()`, `any()`, `all()` over `reduce()`.

---

#### Q66. Explain `any()`, `all()`, `zip()`, and `enumerate()`.

**Answer:**
- `any(iterable)`: Returns `True` if **at least one** element is truthy (short-circuits on first `True`; `False` on empty).
- `all(iterable)`: Returns `True` if **all** elements are truthy (short-circuits on first `False`; `True` on empty).
- `zip(*iterables, strict=False)`: Pairs corresponding items lazily, stopping at the shortest iterable (or raising `ValueError` if `strict=True` in Python 3.10+ when lengths differ).
- `enumerate(iterable, start=0)`: Yields `(index, item)` pairs without manual counter variables.

```python
names = ["Alice", "Bob"]
scores = [95, 88]

for idx, (name, score) in enumerate(zip(names, scores, strict=True), start=1):
    print(f"{idx}. {name}: {score}")

print(any(s > 90 for s in scores))  # True
print(all(s > 90 for s in scores))  # False
```

**Key Points to Remember:**
- Prefer `enumerate(items)` over `range(len(items))` when you need both index and value.
- `zip(*matrix)` is a classic idiom for transposing rows and columns.

---

### File Handling Advanced

#### Q67. How do you work with CSV and JSON files, handle character encodings, and stream large files?

**Answer:**
- **CSV (`csv` module):** Always open with `newline=""` so the `csv` module controls line endings cleanly. Use `csv.DictReader` / `csv.DictWriter` for header-mapped access.
- **JSON (`json` module):** `json.dump` / `json.load` work with file streams; `json.dumps` / `json.loads` work with strings. Note JSON only supports string keys and basic types (`dict`, `list`, `str`, `int`, `float`, `bool`, `None`).
- **Large files:** Read in fixed chunks (`while chunk := f.read(65536):`) or iterate line by line (`for line in f:`) to keep memory constant.

```python
import csv
import json

# JSON serialization
data = {"users": [{"id": 1, "name": "Alice"}]}
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2)

# CSV writing (newline="" prevents blank rows on Windows)
with open("users.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["id", "name"])
    writer.writeheader()
    writer.writerows(data["users"])
```

**Key Points to Remember:**
- Always pass `newline=""` to `open()` when using the `csv` module.
- Use `errors="replace"` or `errors="ignore"` only when intentionally tolerating malformed byte sequences.

---

#### Q68. Explain `seek()`, `tell()`, binary files, and why `pickle` has security risks compared to `json`.

**Answer:**
- `f.tell()`: Returns the current stream position (byte offset in binary mode).
- `f.seek(offset, whence=0)`: Moves the file pointer (`0` = start, `1` = current position, `2` = end; note in text mode only `seek(0)` or offsets from `tell()` are safe).
- **`pickle` vs. `json`:** `pickle` serializes arbitrary Python objects into binary format, whereas `json` is text-based and language-agnostic. **Never unpickle untrusted data:** `pickle.loads()` executes arbitrary Python opcodes embedded in the payload (via `__reduce__`), leading to Remote Code Execution (RCE).

```python
with open("sample.bin", "wb+") as f:
    f.write(b"ABCDEF")
    print(f.tell())  # 6
    f.seek(2)
    print(f.read(3)) # b'CDE'
```

**Key Points to Remember:**
- `pickle` is **not secure** against malicious input—use `json` or schema-validated formats across trust boundaries.

---

### Closures and Scope Rules

#### Q69. Explain the LEGB rule and the `nonlocal` keyword.

**Answer:**
Python resolves variable names using the **LEGB** hierarchy from innermost to outermost:
1. **L (Local):** Inside the current function (including comprehensions).
2. **E (Enclosing):** In outer enclosing functions (for nested functions).
3. **G (Global):** At the top level of the module.
4. **B (Built-in):** Python's `builtins` module (`len`, `print`, `Exception`).

The **`nonlocal`** keyword allows an inner function to rebind a variable in its nearest **enclosing** (non-global) function scope, whereas `global` targets module scope.

```python
def make_counter():
    count = 0
    def inc():
        nonlocal count
        count += 1
        return count
    return inc

c = make_counter()
print(c(), c())  # 1 2
```

**Key Points to Remember:**
- Avoid shadowing built-in names like `list`, `dict`, `id`, `type`, or `str` with local variables.
- `nonlocal` cannot reference module-level globals (use `global` for those).

---

#### Q70. What is a closure in Python, how does late binding cause bugs in loops, and how do you fix it?

**Answer:**
A **closure** occurs when a nested function captures and remembers variables from its enclosing scope even after the outer function has finished executing (stored in `func.__closure__`).

**Late-binding gotcha:** Closures look up the captured variable's value **when the inner function is called**, not when it is defined. Inside a `for` loop, all created functions share the same loop variable and end up using its final value!

**Fix:** Bind the current value at definition time via a default argument (`lambda x, i=i: ...`) or a factory function.

```python
# Buggy: late binding
funcs_bad = [lambda: i for i in range(3)]
print([f() for f in funcs_bad])   # [2, 2, 2]

# Fixed: capture current value with default parameter
funcs_good = [lambda i=i: i for i in range(3)]
print([f() for f in funcs_good])  # [0, 1, 2]
```

**Key Points to Remember:**
- Closures capture **variables (cell references)**, not snapshots of values at definition time.
- Using `i=i` in `lambda` or `def` evaluates `i` immediately during each loop iteration.

---

## Advanced Level (Questions 71–102)

### Metaclasses and Descriptors

#### Q71. What is a metaclass in Python, and how does class creation work step by step?

**Answer:**
In Python, classes themselves are objects—instances of a **metaclass**. By default, every class is an instance of `type`.
When Python executes `class Foo(Base, metaclass=Meta): ...`:
1. Resolves the metaclass (`Meta`, or `type`).
2. Calls `Meta.__prepare__(name, bases, **kwargs)` to obtain the namespace mapping (usually a `dict`).
3. Executes the class body inside that namespace.
4. Calls `Meta(name, bases, namespace)` (`__new__` then `__init__`) to construct and return the class object.

```python
class RegistryMeta(type):
    registry = {}

    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if bases:  # Skip base class itself
            mcs.registry[name] = cls
        return cls

class BasePlugin(metaclass=RegistryMeta):
    pass

class AuthPlugin(BasePlugin):
    pass

print(RegistryMeta.registry)  # {'AuthPlugin': <class '__main__.AuthPlugin'>}
```

**Key Points to Remember:**
- `type(obj)` inspects an object's type; `type(name, bases, dict)` dynamically constructs a new class.
- Metaclasses control **class creation**, just as classes control **instance creation**.

---

#### Q72. When should you use a custom metaclass vs. `__init_subclass__` or a class decorator?

**Answer:**
Metaclasses are powerful but complex and prone to metaclass conflicts when combining bases with different metaclasses. Prefer simpler alternatives whenever possible:
- **`__init_subclass__(cls, **kwargs)` (Python 3.6+):** Automatically called whenever a class is subclassed. Ideal for plugin registration, validating subclass attributes, and injecting methods—without metaclass conflicts.
- **Class decorator (`@dec`):** Best for transforming a single class (e.g., `@dataclass`). Does not automatically apply to subclasses.
- **Metaclass:** Needed only when you must intercept `__prepare__` (custom class namespace), alter class creation before allocation, or customize `isinstance`/`issubclass`/`__call__` on the class object itself.

```python
class BasePlugin:
    plugins = {}

    def __init_subclass__(cls, plugin_name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        if plugin_name:
            BasePlugin.plugins[plugin_name] = cls

class CSVPlugin(BasePlugin, plugin_name="csv"):
    pass

print(BasePlugin.plugins)  # {'csv': <class '__main__.CSVPlugin'>}
```

**Key Points to Remember:**
- Rule of thumb: use a **class decorator** for one class, **`__init_subclass__`** for subclass hooks, and a **metaclass** only as a last resort.

---

#### Q73. What is the descriptor protocol (`__get__`, `__set__`, `__delete__`, `__set_name__`)?

**Answer:**
A **descriptor** is any class defining at least one of `__get__`, `__set__`, or `__delete__`. Assigned as a **class attribute** on an owner class, it intercepts attribute access (`obj.attr`), assignment (`obj.attr = val`), and deletion (`del obj.attr`).
`__set_name__(self, owner, name)` (Python 3.6+) is called automatically at class creation so the descriptor knows its attribute name.

```python
class PositiveInt:
    def __set_name__(self, owner, name):
        self.private_name = f"_{name}"

    def __get__(self, instance, owner=None):
        if instance is None:
            return self
        return getattr(instance, self.private_name)

    def __set__(self, instance, value):
        if not isinstance(value, int) or value <= 0:
            raise ValueError("Must be a positive integer")
        setattr(instance, self.private_name, value)

class Product:
    price = PositiveInt()

    def __init__(self, price):
        self.price = price

p = Product(10)
print(p.price)  # 10
```

**Key Points to Remember:**
- Descriptors must live on the **class** (not inside `self.__dict__`) to trigger automatically.
- Always check `if instance is None: return self` in `__get__` to handle class-level access (`Product.price`).

---

#### Q74. Differentiate between data descriptors and non-data descriptors, and explain attribute lookup precedence.

**Answer:**
- **Data descriptor:** Defines `__get__` **and** `__set__` or `__delete__` (e.g., `@property`).
- **Non-data descriptor:** Defines **only** `__get__` (e.g., regular functions/methods, `@staticmethod`, `@classmethod`, `@functools.cached_property`).

**Attribute lookup order for `obj.attr`:**
1. **Data descriptors** found on `type(obj)` (and its MRO).
2. **`obj.__dict__`** (instance attributes).
3. **Non-data descriptors** and regular class attributes on `type(obj)` (and its MRO).
4. `__getattr__(obj, 'attr')` if defined (only called as a fallback if 1–3 fail).

**Key Points to Remember:**
- **Data descriptors override `obj.__dict__`**, while **`obj.__dict__` overrides non-data descriptors** (which is how `@functools.cached_property` caches its computed value into `obj.__dict__` after first access, and how assigning an attribute onto an instance can shadow a method).

---

#### Q75. How are `@property`, `@staticmethod`, `@classmethod`, and regular methods implemented using descriptors?

**Answer:**
Python's method binding itself is powered by descriptors! Every plain function in Python implements `__get__`:
- When accessed on a **class** (`Dog.speak`), `func.__get__(None, Dog)` returns the plain function.
- When accessed on an **instance** (`dog.speak`), `func.__get__(dog, Dog)` returns a **bound method** that prepends `dog` as `self`.
- `@staticmethod` implements `__get__` to ignore both `instance` and `owner` and return the underlying function unchanged.
- `@classmethod` implements `__get__` to bind `owner` (the class) as the first argument (`cls`).
- `@property` is a **data descriptor** storing `fget`, `fset`, and `fdel` callables.

```python
class SimpleProperty:
    def __init__(self, fget):
        self.fget = fget

    def __get__(self, instance, owner=None):
        return self if instance is None else self.fget(instance)
```

**Key Points to Remember:**
- Methods in Python are simply functions whose `__get__` descriptor binds `self` on attribute access.

---

### GIL, Threading, and Multiprocessing

#### Q76. What is the Global Interpreter Lock (GIL), why does it exist, and how does it affect workloads?

**Answer:**
The **Global Interpreter Lock (GIL)** is a mutex in standard CPython that allows **only one thread to execute Python bytecode at a time** within a single process.
- **Why it exists:** CPython's memory management (reference counting `ob_refcnt`) and many C extensions are not inherently thread-safe. A single lock avoids fine-grained locking overhead, prevents race conditions on reference counts, and keeps single-threaded performance fast.
- **Impact:**
  - **CPU-bound tasks** (math, tight Python loops): Multiple threads do **not** run in parallel across CPU cores; threading can even be slower due to lock contention and context switching.
  - **I/O-bound tasks** (network requests, disk I/O, `time.sleep`): CPython **releases the GIL** during blocking I/O syscalls (and inside C libraries like NumPy during heavy array ops), so `threading` works well for concurrent I/O.

**Key Points to Remember:**
- The GIL is a **CPython** implementation detail (Jython and IronPython have no GIL; PyPy has a GIL).
- Use **`multiprocessing`** (or C extensions / NumPy) for CPU-bound parallelism; use **`threading`** or **`asyncio`** for I/O-bound concurrency.

---

#### Q77. Compare `threading`, `multiprocessing`, and `concurrent.futures`.

**Answer:**
- **`threading`:** Shared memory space, lightweight OS threads, subject to the GIL. Best for blocking I/O (HTTP calls, DB drivers without async support).
- **`multiprocessing`:** Spawns separate OS processes, each with its own Python interpreter and GIL. True multi-core parallelism for CPU-bound work, at the cost of higher memory usage and IPC/pickling overhead.
- **`concurrent.futures`:** High-level unified API (`ThreadPoolExecutor` and `ProcessPoolExecutor`) using `Future` objects, `.submit()`, `.map()`, and `as_completed()`.

```python
from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor

def square(x):
    return x * x

if __name__ == "__main__":
    with ProcessPoolExecutor() as pool:
        results = list(pool.map(square, [1, 2, 3, 4]))
    print(results)  # [1, 4, 9, 16]
```

**Key Points to Remember:**
- `ThreadPoolExecutor` / `ProcessPoolExecutor` are context managers that automatically shut down and join workers cleanly.
- Functions and arguments passed to `ProcessPoolExecutor` must be picklable.

---

#### Q78. Explain synchronization primitives (`Lock`, `RLock`, `Semaphore`, `Event`, `Condition`) and deadlocks.

**Answer:**
Even with the GIL, compound operations (`x += 1`, check-then-act) are **not atomic** because bytecode execution can switch between threads mid-operation, causing **race conditions**:
- **`Lock`:** Basic mutual-exclusion lock. Cannot be re-acquired by the same thread before release.
- **`RLock` (Reentrant Lock):** Can be acquired multiple times by the **same thread** (must be released the same number of times). Useful for recursive methods.
- **`Semaphore`:** Maintains an internal counter allowing up to $N$ threads concurrent access (e.g., rate-limiting connections).
- **`Event`:** Simple boolean flag (`set()`, `clear()`, `wait()`) for signaling across threads.
- **`Condition`:** Combines a lock with `wait()` / `notify()` for producer-consumer coordination.
- **Deadlock:** Two or more threads waiting indefinitely for locks held by each other. Prevent via consistent lock ordering, timeouts (`lock.acquire(timeout=...)`), or `with lock:` context managers.

```python
import threading

lock = threading.Lock()
counter = 0

def safe_inc():
    global counter
    for _ in range(100_000):
        with lock:
            counter += 1
```

**Key Points to Remember:**
- Never assume the GIL makes shared Python variables thread-safe—always synchronize mutable shared state with a `Lock` or `queue.Queue`.

---

#### Q79. How do processes communicate in `multiprocessing`?

**Answer:**
Because processes do not share memory space, `multiprocessing` provides explicit IPC and shared-memory mechanisms:
- **`multiprocessing.Queue`:** Thread- and process-safe FIFO queue using pipes and locks (pickles objects).
- **`multiprocessing.Pipe`:** Two-way (or one-way) connection between two processes (faster than `Queue` for point-to-point communication).
- **`Value` / `Array`:** Typed shared memory (`ctypes`) accessible across processes without pickling (requires synchronization via `.get_lock()`).
- **`multiprocessing.Manager`:** Runs a server process holding shared `dict`/`list` proxies (flexible, but slower due to proxy RPC).

```python
from multiprocessing import Process, Queue

def worker(q):
    q.put("result from child")

if __name__ == "__main__":
    q = Queue()
    p = Process(target=worker, args=(q,))
    p.start()
    print(q.get())
    p.join()
```

**Key Points to Remember:**
- Always call `p.join()` after consuming from a `Queue`, or drain the queue before joining—otherwise a full pipe buffer can deadlock `join()`.

---

#### Q80. What is free-threaded Python (PEP 703 / no-GIL)?

**Answer:**
**PEP 703** introduced an experimental **free-threaded (no-GIL)** build of CPython starting in **Python 3.13** (`python3.13t`).
It replaces the global GIL with:
- **Biased reference counting** (fast non-atomic increments for the owning thread, atomic for others).
- **Immortal objects** (`None`, `True`, small ints, interned strings) that never modify refcounts.
- **Per-object critical-section locks** and thread-safe memory allocation (`mimalloc`).

**Key Points to Remember:**
- Free-threaded Python enables true multi-core `threading` for CPU-bound Python code within one process.
- C extensions must explicitly declare compatibility (`Py_MOD_GIL_NOT_USED`); importing an incompatible C extension re-enables the GIL at runtime.

---

### Memory Management and Garbage Collection

#### Q81. How does Python manage memory? Explain the private heap and `pymalloc`.

**Answer:**
All Python objects live in a **private heap** managed exclusively by the Python memory manager (programmers do not manually `malloc`/`free`).
CPython uses a layered allocator:
- **OS Allocator (`malloc`/`mmap`):** Requests large chunks from the OS.
- **`pymalloc` (Small Object Allocator):** Handles objects up to **512 bytes** by dividing memory into **256 KB Arenas -> 4 KB Pools -> fixed-size Blocks** (multiples of 8/16 bytes). This minimizes OS calls and reduces fragmentation.
- Objects larger than 512 bytes bypass `pymalloc` and go directly to the C `malloc` allocator.

**Key Points to Remember:**
- Python does not always immediately return freed small-object memory to the OS—arenas are retained for reuse unless completely empty.

---

#### Q82. How does reference counting work, and what are its limitations?

**Answer:**
**Reference counting** is CPython's primary memory-reclamation strategy. Every `PyObject` holds an `ob_refcnt` integer tracking how many references point to it:
- Incremented on assignment, passing to a function, or adding to a container.
- Decremented when a name goes out of scope, is reassigned, or `del`eted.
- When `ob_refcnt` hits `0`, memory is reclaimed **immediately**.

**Limitations:**
1. **Cannot collect reference cycles** (e.g., `a.ref = b; b.ref = a`—neither count reaches 0 even after `a` and `b` are unreachable).
2. Overhead of updating refcounts on every assignment/pass.

```python
import sys

x = [1, 2, 3]
print(sys.getrefcount(x))  # 2 (x + temporary argument to getrefcount)
```

**Key Points to Remember:**
- `sys.getrefcount(x)` is always at least 1 higher than expected because passing `x` to the function creates a temporary reference.

---

#### Q83. How does Python's generational garbage collector (`gc`) detect and collect reference cycles?

**Answer:**
To clean up reference cycles among container objects (`list`, `dict`, `class` instances, `tuple`), CPython runs a **generational cyclic garbage collector** (`gc` module):
- Objects are tracked across **three generations** (`0`, `1`, `2`), based on the *weak generational hypothesis* (most objects die young).
- New container objects start in Generation 0. Survivors move to Generation 1, then Generation 2.
- When allocations exceed a generation's threshold (`gc.get_threshold()`), the collector traces references among containers in that generation (and younger ones) to isolate unreachable cycles and break/free them.

```python
import gc

a = []
b = [a]
a.append(b)  # Reference cycle
del a, b

collected = gc.collect()
print(f"Collected {collected} cyclic objects")
```

**Key Points to Remember:**
- Reference counting handles ~99% of deallocations deterministically; the cyclic GC is specifically for **reference cycles**.
- In older Python (< 3.4), cycles involving custom `__del__` methods were uncollectable; **PEP 442 (Python 3.4+)** fixed this so cycles with `__del__` are safely finalized and collected.

---

#### Q84. What causes memory leaks in Python, what is `weakref`, and how do you profile memory (`tracemalloc`)?

**Answer:**
Even with GC, Python programs can "leak" memory when objects remain unintentionally **reachable**:
- Unbounded global caches, module-level lists/dicts, or growing `@lru_cache` without `maxsize`.
- Closures or callbacks retaining references to large objects.
- Unclosed resources or C extensions leaking native memory.

**`weakref`:** Creates a reference to an object **without incrementing its reference count**, allowing caches or observer registries to reference objects without keeping them alive.
**`tracemalloc`:** Standard library tool to trace memory allocations back to exact file and line numbers.

```python
import tracemalloc
import weakref

class Cacheable:
    pass

obj = Cacheable()
weak = weakref.ref(obj)
print(weak())  # <Cacheable object>
del obj
print(weak())  # None (garbage collected!)

tracemalloc.start()
data = [list(range(1000)) for _ in range(100)]
snapshot = tracemalloc.take_snapshot()
for stat in snapshot.statistics("lineno")[:2]:
    print(stat)
```

**Key Points to Remember:**
- Built-in `int`, `str`, `list`, `dict`, `tuple` cannot be weakly referenced directly (wrap or subclass if needed; `set` and custom classes support `weakref`).

---

### Design Patterns

#### Q85. How do you implement the Singleton pattern in Python, and what are its trade-offs?

**Answer:**
A **Singleton** restricts a class to a single instance across the program. Common Python approaches:
1. **Module-level instance (Most Pythonic):** Python modules are imported and cached once in `sys.modules`, so a module-level object (`config = Config()`) acts naturally as a singleton.
2. **`__new__` override** (needs a `threading.Lock` for thread safety).
3. **Metaclass `__call__` override.**

```python
import threading

class SingletonMeta(type):
    _instances = {}
    _lock = threading.Lock()

    def __call__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._instances:
                cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    pass

assert Database() is Database()
```

**Key Points to Remember:**
- Singletons introduce hidden global state and make unit testing harder—prefer **dependency injection** or a **module-level singleton** when appropriate.

---

#### Q86. Explain the Factory Method and Abstract Factory patterns in Python.

**Answer:**
- **Factory Method:** Centralizes object creation behind a function or classmethod so callers don't couple directly to concrete classes.
- **Abstract Factory:** Provides an interface for creating families of related objects without specifying their concrete classes.

```python
class JSONSerializer:
    def serialize(self, data): return f"JSON:{data}"

class XMLSerializer:
    def serialize(self, data): return f"XML:{data}"

def get_serializer(fmt: str):
    factories = {"json": JSONSerializer, "xml": XMLSerializer}
    cls = factories.get(fmt.lower())
    if not cls:
        raise ValueError(f"Unknown format: {fmt}")
    return cls()

print(get_serializer("json").serialize({"id": 1}))
```

**Key Points to Remember:**
- Because classes are first-class objects in Python, a dictionary mapping keys to classes (`{"json": JSONSerializer}`) is often cleaner than long `if/elif` factory chains.

---

#### Q87. Explain the Strategy and Observer patterns in Python.

**Answer:**
- **Strategy Pattern:** Encapsulates interchangeable algorithms so behavior can be swapped at runtime. In Python, since functions are first-class objects, you can pass functions/callables directly instead of writing verbose single-method Strategy classes.
- **Observer Pattern:** Defines a one-to-many subscription mechanism so observers are notified automatically when a subject's state changes.

```python
# Strategy using first-class functions
def no_discount(total): return total
def ten_percent_off(total): return total * 0.90

class Order:
    def __init__(self, total, discount_strategy=no_discount):
        self.total = total
        self.discount_strategy = discount_strategy

    def final_price(self):
        return self.discount_strategy(self.total)

print(Order(100, ten_percent_off).final_price())  # 90.0

# Observer pattern
class EventBus:
    def __init__(self):
        self._subscribers = []

    def subscribe(self, fn):
        self._subscribers.append(fn)

    def notify(self, event):
        for fn in self._subscribers:
            fn(event)
```

**Key Points to Remember:**
- First-class functions make Strategy and Command patterns dramatically simpler in Python than in Java/C++.

---

#### Q88. Explain the Structural Decorator Pattern vs. Python `@decorator`, and the Adapter Pattern.

**Answer:**
- **Decorator Pattern (GoF):** Dynamically wraps an object to add responsibilities without subclassing (e.g., `BufferedReader(FileReader)`). Python's `@decorator` syntax applies the same wrapping idea primarily to functions and classes.
- **Adapter Pattern:** Converts the interface of an existing class into another interface expected by clients, allowing incompatible classes to work together.

```python
class LegacyPrinter:
    def print_old(self, text):
        print(f"[LEGACY] {text}")

class PrinterAdapter:
    def __init__(self, legacy: LegacyPrinter):
        self._legacy = legacy

    def print_message(self, msg: str):  # Unified target interface
        self._legacy.print_old(msg)

PrinterAdapter(LegacyPrinter()).print_message("Hello")
```

**Key Points to Remember:**
- Use **Adapter** to bridge incompatible third-party/legacy APIs; use **Decorator** to layer additional behaviors (logging, caching, retries) transparently.

---

### Asyncio and Concurrency

#### Q89. How does `asyncio` work? Explain event loop, coroutines, `await`, `Task`, and `Future`.

**Answer:**
`asyncio` provides **single-threaded cooperative multitasking** for high-concurrency I/O:
- **Event Loop:** The central scheduler (`asyncio.run(main())`) that tracks ready tasks, polls OS I/O events (`epoll`/`kqueue`), and resumes suspended coroutines.
- **Coroutine (`async def`):** A function that returns a coroutine object; suspends execution at `await` points to yield control back to the event loop.
- **`Future`:** A low-level awaitable representing an eventual result (populated later by an I/O callback or thread).
- **`Task`:** A `Future` subclass that wraps a coroutine and schedules it to run concurrently on the event loop.

```python
import asyncio

async def fetch(name, delay):
    await asyncio.sleep(delay)
    return f"{name} done"

async def main():
    t1 = asyncio.create_task(fetch("A", 0.2))
    t2 = asyncio.create_task(fetch("B", 0.1))
    print(await t1, await t2)

asyncio.run(main())
```

**Key Points to Remember:**
- Calling `await coro()` directly runs coroutines **sequentially**; wrap them in `create_task()`, `gather()`, or `TaskGroup` to run **concurrently**.

---

#### Q90. Compare `asyncio.gather()`, `asyncio.create_task()`, `asyncio.wait()`, and `asyncio.TaskGroup`.

**Answer:**
- `asyncio.create_task(coro)`: Schedules `coro` immediately on the running loop and returns a `Task`.
- `asyncio.gather(*aws, return_exceptions=False)`: Runs awaitables concurrently and returns a list of results in input order. If one fails (with `return_exceptions=False`), `gather` raises immediately without cancelling sibling tasks by default.
- `asyncio.wait(tasks, return_when=ALL_COMPLETED)`: Lower-level primitive returning `(done, pending)` sets—useful for `FIRST_COMPLETED` or `FIRST_EXCEPTION` control.
- `asyncio.TaskGroup` (Python 3.11+): Structured concurrency context manager. Automatically awaits all created tasks on exit, **cancels sibling tasks** if any fails, and raises an `ExceptionGroup` (handled with `except*`).

```python
import asyncio

async def work(x):
    await asyncio.sleep(0.1)
    return x * 2

async def main():
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(work(i)) for i in range(3)]
    print([t.result() for t in tasks])  # [0, 2, 4]

asyncio.run(main())
```

**Key Points to Remember:**
- Prefer **`asyncio.TaskGroup`** in Python 3.11+—it avoids orphaned background tasks and cleanly propagates errors via `ExceptionGroup`.

---

#### Q91. What happens if you call blocking synchronous code inside an `async` function, and how do you fix it?

**Answer:**
Because `asyncio` runs everything on a **single thread**, calling blocking synchronous functions (e.g., `time.sleep(5)`, `requests.get()`, heavy CPU math) inside `async def` **freezes the entire event loop**—no other tasks, heartbeats, or I/O handlers can make progress.

**Fixes:**
1. Use native async libraries (`aiohttp`/`httpx`, `asyncpg`, `asyncio.sleep`).
2. Offload unavoidable blocking I/O to a worker thread using **`await asyncio.to_thread(blocking_fn, *args)`** (Python 3.9+).
3. Offload CPU-bound work to `loop.run_in_executor(process_pool, cpu_fn, *args)`.

```python
import asyncio
import time

def blocking_io():
    time.sleep(0.2)
    return "loaded"

async def main():
    res = await asyncio.to_thread(blocking_io)
    print(res)

asyncio.run(main())
```

**Key Points to Remember:**
- Never call `time.sleep()` or synchronous `requests` inside an async handler—use `await asyncio.sleep()` or `asyncio.to_thread()`.

---

#### Q92. Explain asynchronous iterators (`async for`) and asynchronous context managers (`async with`).

**Answer:**
- **Async iterator:** Implements `__aiter__(self)` (returns `self`, *not* `async def`) and `async def __anext__(self)` (raises `StopAsyncIteration` when done). Consumed with `async for item in obj:`.
- **Async context manager:** Implements `async def __aenter__(self)` and `async def __aexit__(self, exc_type, exc, tb)`. Used with `async with obj as res:` (or `@contextlib.asynccontextmanager`).

```python
import asyncio

class AsyncTicker:
    def __init__(self, count):
        self.count = count

    def __aiter__(self):
        return self

    async def __anext__(self):
        if self.count <= 0:
            raise StopAsyncIteration
        await asyncio.sleep(0.05)
        self.count -= 1
        return self.count

async def main():
    async for tick in AsyncTicker(3):
        print(tick)

asyncio.run(main())
```

**Key Points to Remember:**
- `__aiter__` is a synchronous method returning the async iterator; `__anext__`, `__aenter__`, and `__aexit__` are `async def` coroutines.

---

### Context Managers

#### Q93. How do context managers work (`__enter__`, `__exit__`), and how does `__exit__` suppress exceptions?

**Answer:**
A context manager implements `__enter__` and `__exit__` to manage setup and deterministic teardown around a `with` block:
1. `__enter__(self)` runs at entry; its return value is bound to `as var`.
2. `__exit__(self, exc_type, exc_val, exc_tb)` always runs on exit.
   - If no exception occurred, all three arguments are `None`.
   - If an exception occurred, returning a **truthy** value (`True`) **suppresses** the exception; returning `False` or `None` lets it propagate.

```python
class SuppressZeroDiv:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is ZeroDivisionError:
            print("Suppressed division by zero")
            return True  # Exception suppressed
        return False

with SuppressZeroDiv():
    _ = 1 / 0
print("Continues normally!")
```

**Key Points to Remember:**
- `contextlib.suppress(FileNotFoundError)` is a built-in clean alternative to `try: ... except FileNotFoundError: pass`.

---

#### Q94. How do you write context managers using `@contextlib.contextmanager` and `contextlib.ExitStack`?

**Answer:**
- **`@contextlib.contextmanager`:** Turns a generator function with a single `yield` into a context manager. Code before `yield` is `__enter__`; the yielded value becomes the `as` target; code after `yield` (wrap in `try...finally` to ensure cleanup on exceptions!) acts as `__exit__`.
- **`contextlib.ExitStack`:** Dynamically manages a variable number of context managers computed at runtime (e.g., opening a list of files).

```python
from contextlib import contextmanager, ExitStack
import time

@contextmanager
def timed_block(label):
    start = time.perf_counter()
    try:
        yield
    finally:
        print(f"{label}: {time.perf_counter() - start:.4f}s")

with timed_block("sum"):
    sum(range(100_000))
```

**Key Points to Remember:**
- Always wrap `yield` inside `try ... finally:` in `@contextmanager` functions—otherwise an exception in the `with` body skips cleanup code after `yield`.

---

### Monkey Patching

#### Q95. What is monkey patching in Python, when is it appropriate, and what are its risks?

**Answer:**
**Monkey patching** is dynamically modifying or replacing attributes, methods, or functions on a class or module at runtime without altering its original source code.
- **Appropriate use cases:** Unit testing/mocking (`unittest.mock.patch`, `pytest`'s `monkeypatch`), applying urgent runtime compatibility shims/hotfixes (e.g., `gevent.monkey.patch_all()`).
- **Risks:** Breaks readability and static analysis, creates fragile coupling to internal implementation details of third-party libraries, and causes subtle global side effects across unrelated modules.

```python
class Service:
    def fetch(self):
        return "real network call"

# Monkey patch during testing
Service.fetch = lambda self: "mocked response"
print(Service().fetch())  # mocked response
```

**Key Points to Remember:**
- In production code, prefer composition, adapters, or subclassing over monkey patching; in tests, always use `unittest.mock.patch` or `pytest`'s `monkeypatch` so patches auto-revert after the test.

---

### Slots and Method Resolution Order (MRO)

#### Q96. What is `__slots__`, how does it save memory, and what are its caveats?

**Answer:**
Normally, every Python instance stores attributes in a dynamic per-instance hash table (`__dict__`), which has significant memory overhead. Defining `__slots__ = ("x", "y")` tells Python to allocate a fixed-size C-level array for just those attributes and **omit `__dict__` and `__weakref__`**.

**Benefits:** Saves substantial RAM (especially for millions of small objects) and slightly speeds up attribute access.
**Caveats:**
- Prevents adding new attributes dynamically (`AttributeError`).
- Subclasses automatically get `__dict__` again unless they also declare `__slots__ = (...)` (use `__slots__ = ()` if adding no new attributes).
- Breaks `weakref` unless `"__weakref__"` is explicitly included in `__slots__`.
- Multiple inheritance with two non-empty `__slots__` bases raises `TypeError`.

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
# p.z = 3  # AttributeError: 'Point' object has no attribute 'z'
```

**Key Points to Remember:**
- `@dataclass(slots=True)` (Python 3.10+) combines dataclass ergonomics with `__slots__` memory savings.

---

#### Q97. Explain Method Resolution Order (MRO) and C3 linearization.

**Answer:**
**Method Resolution Order (MRO)** is the deterministic order in which Python searches base classes for a method or attribute in single or multiple inheritance. Python uses the **C3 linearization algorithm**, which preserves two key properties:
1. **Local precedence:** Subclasses appear before parents, and left-to-right base order (`class D(B, C):`) is preserved (`B` before `C`).
2. **Monotonicity:** A parent's MRO order is never contradicted by a child's MRO.

If no valid C3 linearization exists (e.g., `class X(A, B)` and `class Y(B, A)` combined in `class Z(X, Y)`), Python raises a `TypeError: Cannot create a consistent method resolution order`.

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

print([cls.__name__ for cls in D.mro()])
# ['D', 'B', 'C', 'A', 'object']
```

**Key Points to Remember:**
- Inspect MRO anytime via `ClassName.mro()` or `ClassName.__mro__`.
- In diamond inheritance `D(B, C)` where both inherit `A`, `A` appears **after both `B` and `C`**, never before `C`.

---

#### Q98. How does `super()` work cooperatively in diamond inheritance hierarchies?

**Answer:**
`super()` does **not** mean "call my parent class"—it means **"call the next class in `type(self).__mro__`"**.
In cooperative multiple inheritance, every class calls `super().method()` so each class in the MRO executes **exactly once**, even in a diamond hierarchy.

```python
class A:
    def greet(self):
        print("A", end=" ")

class B(A):
    def greet(self):
        print("B", end=" ")
        super().greet()

class C(A):
    def greet(self):
        print("C", end=" ")
        super().greet()

class D(B, C):
    def greet(self):
        print("D", end=" ")
        super().greet()

D().greet()  # D B C A
```

**Key Points to Remember:**
- Notice `B`'s `super().greet()` calls `C.greet()` (not `A.greet()`) when invoked on a `D` instance, because `D`'s MRO is `[D, B, C, A, object]`.
- Cooperative `__init__` methods in mixins typically accept `**kwargs` and pass remaining `**kwargs` to `super().__init__(**kwargs)`.

---

### Performance Optimization

#### Q99. What are the best practices for optimizing Python performance?

**Answer:**
1. **Algorithm & Data Structures first:** Use `set`/`dict` ($O(1)$ lookup) instead of `list` ($O(n)$), `collections.deque` for FIFO queues, `bisect` / `heapq` for sorted operations.
2. **Leverage C-implemented built-ins:** `sum()`, `min()`, `max()`, `any()`, `all()`, `"".join()`, list comprehensions, and `itertools` run loops in C instead of Python bytecode.
3. **Memoization / Caching:** `@functools.lru_cache` / `@functools.cache` for pure repeated computations.
4. **Memory efficiency:** Generators (`yield`), `__slots__`, and `array`/`memoryview`.
5. **Local variable bindings:** Local lookups (`LOAD_FAST`) are faster than global lookups (`LOAD_GLOBAL`) in hot loops.
6. **Vectorization & Native Extensions:** Offload numerical work to NumPy/Polars, Cython, or Rust/C extensions (PyO3); use PyPy for long-running pure Python workloads.

```python
from functools import cache

@cache
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)
```

**Key Points to Remember:**
- "Premature optimization is the root of all evil"—always **measure with a profiler** before optimizing.

---

#### Q100. How do you benchmark and profile CPU performance in Python (`timeit`, `cProfile`, `pstats`)?

**Answer:**
- **`timeit`:** Micro-benchmarks small snippets repeatedly while disabling GC during timing to avoid noise.
- **`cProfile`:** Deterministic function-level profiler measuring call counts, `tottime` (time inside the function itself), and `cumtime` (including subcalls).
- **`pstats`:** Parses and sorts `cProfile` output.

```python
import cProfile
import pstats
import timeit

# Micro-benchmark
t = timeit.timeit("'-'.join(str(n) for n in range(100))", number=10_000)
print(f"timeit: {t:.4f}s")

# Function profiling
def work():
    return sum(i * i for i in range(50_000))

with cProfile.Profile() as pr:
    work()

pstats.Stats(pr).sort_stats("cumtime").print_stats(5)
```

**Key Points to Remember:**
- Use `time.perf_counter()` (not `time.time()`) for manual elapsed-time measurements, as it is monotonic and highest resolution.
- For production sampling with minimal overhead, tools like `py-spy` or `scalene` profile line-by-line without slowing down code like `cProfile`.

---

### Python Internals and Modern Features

#### Q101. Explain `.pyc` files, Python bytecode, the `dis` module, and the CPython evaluation loop.

**Answer:**
When a module is imported, CPython compiles `.py` source -> AST -> **bytecode** (`PyCodeObject`) and caches it as `__pycache__/module.cpython-3xx.pyc` (including a source timestamp or hash to detect staleness).
At runtime, CPython's **evaluation loop** (`_PyEval_EvalFrameDefault` in `ceval.c`) executes stack-based bytecode instructions (`LOAD_FAST`, `BINARY_OP`, `CALL`, `RETURN_VALUE`). Python 3.11+ also uses a **Specializing Adaptive Interpreter** (PEP 659) that dynamically specializes hot bytecode instructions for specific types (e.g., `BINARY_OP_ADD_INT`).

```python
import dis

def square_sum(a, b):
    return a * a + b * b

print(square_sum.__code__.co_varnames)  # ('a', 'b')
dis.dis(square_sum)
```

**Key Points to Remember:**
- The top-level script run directly (`python app.py`) is compiled to bytecode in memory without writing a `.pyc` file—only **imported** modules create `.pyc` files.
- `dis.dis()` is invaluable for inspecting how Python executes expressions under the hood.

---

#### Q102. Explain `dataclasses`, `TypedDict`, `Protocol`, and `match`/`case` structural pattern matching.

**Answer:**
Modern Python includes powerful typing and data-modeling constructs:
- **`@dataclass` (Python 3.7+):** Auto-generates `__init__`, `__repr__`, `__eq__`, and optionally `order=True`, `frozen=True` (immutable + hashable), and `slots=True`.
- **`TypedDict`:** Type-checks dictionaries with fixed key names and value types (remains a plain `dict` at runtime).
- **`Protocol` (PEP 544):** Structural subtyping (static duck typing)—any class implementing the required methods satisfies the `Protocol` without inheriting from it.
- **`match` / `case` (Python 3.10+, PEP 634):** Structural pattern matching that destructures sequences, dicts, and class instances with optional `if` guards.

```python
from dataclasses import dataclass
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> str: ...

@dataclass(frozen=True, slots=True)
class Point:
    x: int
    y: int

    def draw(self) -> str:
        return f"({self.x}, {self.y})"

def describe(pt: Point) -> str:
    match pt:
        case Point(0, 0):
            return "Origin"
        case Point(x, y) if x == y:
            return f"Diagonal at {x}"
        case Point(x, y):
            return f"Point({x}, {y})"

print(describe(Point(3, 3)))  # Diagonal at 3
```

**Key Points to Remember:**
- `frozen=True` on `@dataclass` makes instances read-only and generates `__hash__`, allowing them in sets/dicts.
- `match`/`case` matches **structure and bindings**, not just simple equality like C's `switch`.

---

## Coding Problems (Questions 103–125)

### String Manipulation Problems

#### Q103. Reverse the words in a sentence, and check if a string is a valid palindrome (ignoring non-alphanumeric characters and case).

**Answer:**
- **Reverse words:** `split()` handles multiple spaces automatically; reverse the resulting list and join with `" "`.
- **Valid palindrome:** Use two pointers (`left`, `right`) moving inward, skipping non-alphanumeric characters (`isalnum()`) and comparing lowercased characters in $O(1)$ extra space (or filter + slice `[::-1]` in $O(n)$ space).

```python
def reverse_words(s: str) -> str:
    return " ".join(s.split()[::-1])

def is_palindrome(s: str) -> bool:
    left, right = 0, len(s) - 1
    while left < right:
        while left < right and not s[left].isalnum():
            left += 1
        while left < right and not s[right].isalnum():
            right -= 1
        if s[left].lower() != s[right].lower():
            return False
        left += 1
        right -= 1
    return True

print(reverse_words("  hello   world  "))            # "world hello"
print(is_palindrome("A man, a plan, a canal: Panama"))  # True
```

**Key Points to Remember:**
- `s.split()` with no argument collapses arbitrary whitespace and strips outer spaces, unlike `s.split(" ")`.
- Two-pointer palindrome check uses **$O(n)$ time and $O(1)$ extra space**.

---

#### Q104. Check if two strings are anagrams, and group a list of words into anagram groups.

**Answer:**
Two strings are anagrams if they contain identical character frequencies:
- Check with `collections.Counter(s1) == Counter(s2)` in $O(n)$ time.
- To group anagrams, use a `defaultdict(list)` keyed by the sorted character tuple `tuple(sorted(word))` or a 26-element frequency tuple for lowercase ASCII strings.

```python
from collections import Counter, defaultdict

def are_anagrams(s1: str, s2: str) -> bool:
    return len(s1) == len(s2) and Counter(s1) == Counter(s2)

def group_anagrams(words: list[str]) -> list[list[str]]:
    groups = defaultdict(list)
    for word in words:
        groups[tuple(sorted(word))].append(word)
    return list(groups.values())

print(are_anagrams("listen", "silent"))  # True
print(group_anagrams(["eat", "tea", "tan", "ate", "nat", "bat"]))
# [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]
```

**Key Points to Remember:**
- `Counter` comparison is $O(n)$ time, faster than `sorted(s1) == sorted(s2)` ($O(n \log n)$).
- Dictionaries preserve insertion order, so `Counter` or frequency tuples work cleanly as keys.

---

#### Q105. Find the first non-repeating character in a string, and implement basic string compression (run-length encoding).

**Answer:**
- **First unique char:** Count frequencies in a first pass (`Counter`), then iterate the string in order to return the first character with count `1`.
- **Run-length encoding:** Track consecutive character runs and append `char + count` to a list, then join.

```python
from collections import Counter

def first_unique_char(s: str) -> int:
    counts = Counter(s)
    for idx, ch in enumerate(s):
        if counts[ch] == 1:
            return idx
    return -1

def compress_string(s: str) -> str:
    if not s:
        return ""
    parts = []
    count = 1
    for i in range(1, len(s)):
        if s[i] == s[i - 1]:
            count += 1
        else:
            parts.append(f"{s[i - 1]}{count}")
            count = 1
    parts.append(f"{s[-1]}{count}")
    compressed = "".join(parts)
    return compressed if len(compressed) < len(s) else s

print(first_unique_char("swiss"))       # 1 ('w')
print(compress_string("aaabbcdddd"))    # "a3b2c1d4"
```

**Key Points to Remember:**
- Always build strings via a list + `"".join(parts)` to avoid $O(n^2)$ string concatenation.

---

#### Q106. Find the length of the longest substring without repeating characters.

**Answer:**
Use a **sliding window** with a dictionary mapping each character to its most recent index. When encountering a repeated character inside the current window (`last_seen[ch] >= left`), jump `left` directly to `last_seen[ch] + 1`.

```python
def length_of_longest_substring(s: str) -> int:
    last_seen = {}
    left = 0
    best = 0
    for right, ch in enumerate(s):
        if ch in last_seen and last_seen[ch] >= left:
            left = last_seen[ch] + 1
        last_seen[ch] = right
        best = max(best, right - left + 1)
    return best

print(length_of_longest_substring("abcabcbb"))  # 3 ("abc")
print(length_of_longest_substring("pwwkew"))    # 3 ("wke")
```

**Key Points to Remember:**
- Time complexity: **$O(n)$** (each character visited once); Space: **$O(\min(n, |\Sigma|))$**.
- Checking `last_seen[ch] >= left` ensures `left` never moves backward.

---

### List and Array Problems

#### Q107. Two Sum: Given a list of integers and a target, return indices of two numbers that add up to the target.

**Answer:**
Store previously seen numbers and their indices in a hash map (`dict`). For each element `num`, check if `target - num` is already in the map in $O(1)$ average time.

```python
def two_sum(nums: list[int], target: int) -> list[int]:
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []

print(two_sum([2, 7, 11, 15], 9))  # [0, 1]
```

**Key Points to Remember:**
- Single-pass hash map achieves **$O(n)$ time and $O(n)$ space**, improving over the $O(n^2)$ brute-force nested loop.

---

#### Q108. Rotate an array to the right by `k` steps in-place, and move all zeroes to the end while preserving relative order.

**Answer:**
- **In-place rotation (Reversal algorithm):** Normalize `k %= len(nums)`. Reverse the entire list, reverse the first `k` elements, then reverse the remaining `n - k` elements ($O(n)$ time, $O(1)$ space).
- **Move zeroes:** Maintain a `write` pointer for the next non-zero position and swap non-zero elements forward.

```python
def rotate_right(nums: list[int], k: int) -> None:
    if not nums:
        return
    k %= len(nums)

    def reverse(l: int, r: int) -> None:
        while l < r:
            nums[l], nums[r] = nums[r], nums[l]
            l += 1
            r -= 1

    reverse(0, len(nums) - 1)
    reverse(0, k - 1)
    reverse(k, len(nums) - 1)

def move_zeroes(nums: list[int]) -> None:
    write = 0
    for read in range(len(nums)):
        if nums[read] != 0:
            nums[write], nums[read] = nums[read], nums[write]
            write += 1

arr = [1, 2, 3, 4, 5]
rotate_right(arr, 2)
print(arr)  # [4, 5, 1, 2, 3]

z = [0, 1, 0, 3, 12]
move_zeroes(z)
print(z)    # [1, 3, 12, 0, 0]
```

**Key Points to Remember:**
- Always apply `k %= len(nums)` first in case `k >= len(nums)`.
- Slicing (`nums[:] = nums[-k:] + nums[:-k]`) is concise in Python but uses $O(n)$ extra space; three reversals uses $O(1)$ space.

---

#### Q109. Merge overlapping intervals.

**Answer:**
Sort intervals by start time (`intervals.sort(key=lambda x: x[0])`). Iterate through sorted intervals: if the current interval's start is `<=` the last merged interval's end, extend the end (`max`); otherwise append as a new interval.

```python
def merge_intervals(intervals: list[list[int]]) -> list[list[int]]:
    if not intervals:
        return []
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0][:]]
    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged

print(merge_intervals([[1, 3], [2, 6], [8, 10], [15, 18]]))
# [[1, 6], [8, 10], [15, 18]]
```

**Key Points to Remember:**
- Time complexity: **$O(n \log n)$** due to sorting; Space: **$O(n)$** for the output.

---

#### Q110. Kadane's Algorithm: Find the contiguous subarray with the largest sum.

**Answer:**
Maintain `curr_sum` (maximum subarray sum ending at the current index) and `best_sum`. At each element `x`, either extend the previous subarray (`curr_sum + x`) or start fresh at `x` (`max(x, curr_sum + x)`).

```python
def max_subarray(nums: list[int]) -> int:
    curr_sum = best_sum = nums[0]
    for x in nums[1:]:
        curr_sum = max(x, curr_sum + x)
        best_sum = max(best_sum, curr_sum)
    return best_sum

print(max_subarray([-2, 1, -3, 4, -1, 2, 1, -5, 4]))  # 6 ([4, -1, 2, 1])
```

**Key Points to Remember:**
- Initializing `curr_sum = best_sum = nums[0]` handles arrays where all numbers are negative!
- Time complexity: **$O(n)$**, Space: **$O(1)$**.

---

#### Q111. Flatten an arbitrarily nested list, and remove duplicates from a list while preserving original order.

**Answer:**
- **Flatten nested list:** Recursively yield elements (`yield from`) or use an explicit stack to avoid recursion limits.
- **Deduplicate preserving order:** In Python 3.7+, `list(dict.fromkeys(items))` leverages insertion-ordered dictionaries in $O(n)$ time!

```python
def flatten(nested: list) -> list:
    result = []
    for item in nested:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result

def dedup_ordered(items: list) -> list:
    return list(dict.fromkeys(items))

print(flatten([1, [2, [3, 4], 5], 6]))   # [1, 2, 3, 4, 5, 6]
print(dedup_ordered([3, 1, 2, 1, 3, 4])) # [3, 1, 2, 4]
```

**Key Points to Remember:**
- `list(set(items))` destroys original order; `list(dict.fromkeys(items))` preserves it in $O(n)$ time for hashable elements.

---

### Dictionary Problems

#### Q112. Implement an LRU (Least Recently Used) Cache with $O(1)$ `get` and `put` operations.

**Answer:**
`collections.OrderedDict` maintains a doubly linked list + hash map internally, supporting $O(1)$ `move_to_end()` and `popitem(last=False)`:
- On `get(key)`: If present, move to end (most recently used) and return value; else `-1`.
- On `put(key, val)`: Insert/update and move to end. If capacity exceeded, `popitem(last=False)` evicts the least recently used entry.

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache: OrderedDict[int, int] = OrderedDict()

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)

lru = LRUCache(2)
lru.put(1, 10)
lru.put(2, 20)
print(lru.get(1))  # 10 (key 2 is now LRU)
lru.put(3, 30)     # Evicts key 2
print(lru.get(2))  # -1
```

**Key Points to Remember:**
- Both `get` and `put` run in **$O(1)$** time.
- In standard interviews, be prepared to mention that `OrderedDict` encapsulates a **Hash Map + Doubly Linked List** (and know how to implement both if asked without `OrderedDict`).

---

#### Q113. Find the `k` most frequent elements in a list.

**Answer:**
Count frequencies using `collections.Counter`. Either use `Counter(nums).most_common(k)` (uses `heapq.nlargest`, $O(n \log k)$), or use **Bucket Sort** indexed by frequency for $O(n)$ time.

```python
from collections import Counter

def top_k_frequent(nums: list[int], k: int) -> list[int]:
    counts = Counter(nums)
    buckets = [[] for _ in range(len(nums) + 1)]
    for num, freq in counts.items():
        buckets[freq].append(num)

    res = []
    for freq in range(len(buckets) - 1, 0, -1):
        for num in buckets[freq]:
            res.append(num)
            if len(res) == k:
                return res
    return res

print(top_k_frequent([1, 1, 1, 2, 2, 3], 2))  # [1, 2]
```

**Key Points to Remember:**
- Bucket sort achieves **$O(n)$ time**; `Counter(nums).most_common(k)` is the shortest idiomatic solution in $O(n \log k)$.

---

#### Q114. Invert a dictionary (handling duplicate values), and deep-merge two nested dictionaries.

**Answer:**
- **Invert with duplicates:** Since multiple keys can map to the same value, group original keys into lists using `defaultdict(list)`.
- **Deep merge:** Recursively merge when both values at a key are `dict` instances; otherwise overwrite with the second dict's value.

```python
from collections import defaultdict

def invert_dict(d: dict) -> dict:
    inverted = defaultdict(list)
    for k, v in d.items():
        inverted[v].append(k)
    return dict(inverted)

def deep_merge(d1: dict, d2: dict) -> dict:
    result = d1.copy()
    for k, v in d2.items():
        if k in result and isinstance(result[k], dict) and isinstance(v, dict):
            result[k] = deep_merge(result[k], v)
        else:
            result[k] = v
    return result

print(invert_dict({"a": 1, "b": 2, "c": 1}))  # {1: ['a', 'c'], 2: ['b']}
print(deep_merge({"db": {"host": "localhost", "port": 5432}}, {"db": {"port": 5433}}))
# {'db': {'host': 'localhost', 'port': 5433}}
```

**Key Points to Remember:**
- A naive `{v: k for k, v in d.items()}` silently drops keys when values collide.

---

### Recursion Problems

#### Q115. Generate all permutations and all subsets (power set) of a list of distinct integers.

**Answer:**
Both are classic **backtracking** problems:
- **Subsets:** At each index, choose either to include `nums[i]` or skip it (or iterate `start` index forward). $2^n$ subsets.
- **Permutations:** Swap or track `used` elements at each depth until `len(path) == len(nums)`. $n!$ permutations.

```python
def subsets(nums: list[int]) -> list[list[int]]:
    res = []
    def backtrack(start: int, path: list[int]):
        res.append(path[:])
        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()
    backtrack(0, [])
    return res

def permutations(nums: list[int]) -> list[list[int]]:
    res = []
    used = [False] * len(nums)
    def backtrack(path: list[int]):
        if len(path) == len(nums):
            res.append(path[:])
            return
        for i in range(len(nums)):
            if not used[i]:
                used[i] = True
                path.append(nums[i])
                backtrack(path)
                path.pop()
                used[i] = False
    backtrack([])
    return res

print(subsets([1, 2]))       # [[], [1], [1, 2], [2]]
print(permutations([1, 2]))  # [[1, 2], [2, 1]]
```

**Key Points to Remember:**
- Always append a copy (`path[:]`) to `res`, because `path` is mutated during backtracking.
- Standard library `itertools.permutations` and `itertools.combinations` exist for production code.

---

#### Q116. Solve Fibonacci and Climbing Stairs using memoization vs. bottom-up dynamic programming.

**Answer:**
Naive recursive Fibonacci is $O(2^n)$.
- **Top-down (Memoization):** `@functools.cache` reduces time to $O(n)$ and space to $O(n)$ (recursion stack + cache).
- **Bottom-up (Iterative DP):** Tracks only the previous two states (`a, b`), reducing space to **$O(1)$** and avoiding Python's recursion limit (`sys.getrecursionlimit()`, default 1000).

```python
from functools import cache

@cache
def climb_stairs_memo(n: int) -> int:
    if n <= 2:
        return n
    return climb_stairs_memo(n - 1) + climb_stairs_memo(n - 2)

def climb_stairs_dp(n: int) -> int:
    if n <= 2:
        return n
    a, b = 1, 2
    for _ in range(3, n + 1):
        a, b = b, a + b
    return b

print(climb_stairs_memo(5), climb_stairs_dp(5))  # 8 8
```

**Key Points to Remember:**
- Bottom-up DP is preferred for large `n` because Python does **not** perform tail-call optimization and has a default recursion depth limit of 1000.

---

#### Q117. Validate a Binary Search Tree (BST) and perform BFS level-order traversal.

**Answer:**
- **Validate BST:** Every node must lie strictly within `(low, high)` bounds passed down recursively (checking only `node.left.val < node.val < node.right.val` is wrong because a node in the right subtree could violate an ancestor's bound!).
- **Level-order BFS:** Use `collections.deque` to pop nodes level by level in $O(n)$ time.

```python
from collections import deque
import math

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val, self.left, self.right = val, left, right

def is_valid_bst(node: TreeNode | None, low=-math.inf, high=math.inf) -> bool:
    if not node:
        return True
    if not (low < node.val < high):
        return False
    return (
        is_valid_bst(node.left, low, node.val)
        and is_valid_bst(node.right, node.val, high)
    )

def level_order(root: TreeNode | None) -> list[list[int]]:
    if not root:
        return []
    res, q = [], deque([root])
    while q:
        level = []
        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)
            if node.left: q.append(node.left)
            if node.right: q.append(node.right)
        res.append(level)
    return res
```

**Key Points to Remember:**
- Always use `collections.deque.popleft()` ($O(1)$) instead of `list.pop(0)` ($O(n)$) for BFS queues.

---

### Pattern Printing

#### Q118. Print a centered star pyramid and a diamond pattern of height `n`.

**Answer:**
For row `i` from `0` to `n - 1`, print `n - 1 - i` leading spaces followed by `2 * i + 1` stars. Mirror the upper rows (excluding the middle row) for a diamond.

```python
def print_pyramid(n: int) -> None:
    for i in range(n):
        print(" " * (n - 1 - i) + "*" * (2 * i + 1))

def print_diamond(n: int) -> None:
    for i in range(n):
        print(" " * (n - 1 - i) + "*" * (2 * i + 1))
    for i in range(n - 2, -1, -1):
        print(" " * (n - 1 - i) + "*" * (2 * i + 1))

print_diamond(3)
#   *
#  ***
# *****
#  ***
#   *
```

**Key Points to Remember:**
- String repetition (`"*" * k`) makes pattern printing concise and avoids inner loops.

---

#### Q119. Generate Pascal's Triangle (first `n` rows) and print Floyd's Triangle.

**Answer:**
- **Pascal's Triangle:** Each row starts and ends with `1`, and every interior element `row[j]` equals `prev[j - 1] + prev[j]`.
- **Floyd's Triangle:** Sequential integers printed in rows of increasing length (`1`, `2 3`, `4 5 6`, ...).

```python
def pascals_triangle(n: int) -> list[list[int]]:
    triangle = []
    for r in range(n):
        row = [1] * (r + 1)
        for j in range(1, r):
            row[j] = triangle[r - 1][j - 1] + triangle[r - 1][j]
        triangle.append(row)
    return triangle

def floyds_triangle(n: int) -> None:
    num = 1
    for r in range(1, n + 1):
        row = [str(num + i) for i in range(r)]
        num += r
        print(" ".join(row))

print(pascals_triangle(5))
# [[1], [1, 1], [1, 2, 1], [1, 3, 3, 1], [1, 4, 6, 4, 1]]
floyds_triangle(4)
```

**Key Points to Remember:**
- Pascal's Triangle runs in $O(n^2)$ time and space; row `r` gives the binomial coefficients $\binom{r}{j}$.

---

### Sorting and Searching

#### Q120. Implement Binary Search (iterative) and Search in a Rotated Sorted Array.

**Answer:**
- **Standard Binary Search:** Halves the search interval on a sorted array in $O(\log n)$ time ($O(1)$ space iteratively). Python's `bisect` module (`bisect_left`, `bisect_right`) provides this in C.
- **Rotated Sorted Array:** At any `mid`, at least one half (`[left..mid]` or `[mid..right]`) is guaranteed to be normally sorted. Check which half is sorted and whether `target` lies inside it.

```python
def binary_search(nums: list[int], target: int) -> int:
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = left + (right - left) // 2
        if nums[mid] == target:
            return mid
        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

def search_rotated(nums: list[int], target: int) -> int:
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = left + (right - left) // 2
        if nums[mid] == target:
            return mid
        if nums[left] <= nums[mid]:  # Left half is sorted
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:                        # Right half is sorted
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    return -1

print(binary_search([1, 3, 5, 7, 9], 7))         # 3
print(search_rotated([4, 5, 6, 7, 0, 1, 2], 0))  # 4
```

**Key Points to Remember:**
- Prefer iterative binary search over recursive ($O(1)$ space vs $O(\log n)$ call stack).
- In production Python, use `bisect.bisect_left` on sorted lists.

---

#### Q121. Implement Merge Sort and Quick Sort in Python, and compare their complexities.

**Answer:**
- **Merge Sort:** Divide-and-conquer, **stable**, guaranteed **$O(n \log n)$** worst-case time, **$O(n)$** auxiliary space.
- **Quick Sort:** Divide-and-conquer around a pivot. **$O(n \log n)$** average time, **$O(n^2)$** worst-case (mitigated by random/median-of-three pivot), **$O(\log n)$** stack space when done in-place, unstable.

```python
def merge_sort(arr: list[int]) -> list[int]:
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return _merge(left, right)

def _merge(left: list[int], right: list[int]) -> list[int]:
    merged, i, j = [], 0, 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            merged.append(left[i]); i += 1
        else:
            merged.append(right[j]); j += 1
    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged

def quick_sort(arr: list[int]) -> list[int]:
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    less = [x for x in arr if x < pivot]
    equal = [x for x in arr if x == pivot]
    greater = [x for x in arr if x > pivot]
    return quick_sort(less) + equal + quick_sort(greater)

print(merge_sort([5, 2, 9, 1, 5, 6]))  # [1, 2, 5, 5, 6, 9]
print(quick_sort([5, 2, 9, 1, 5, 6]))  # [1, 2, 5, 5, 6, 9]
```

**Key Points to Remember:**
- Python's built-in `sorted()` / `list.sort()` uses **Timsort** ($O(n \log n)$ worst-case, $O(n)$ best-case on nearly sorted data, stable).

---

#### Q122. Sort a dictionary by values and perform multi-key sorting on custom records.

**Answer:**
Pass a `key` function returning a tuple `(primary_key, secondary_key)` to `sorted()`. For mixed ascending/descending numeric keys, negate numbers (or do two stable sorts—secondary key first, primary key second—for strings).

```python
scores = {"Alice": 88, "Bob": 95, "Charlie": 88}

# Sort by score descending, then name ascending
sorted_items = sorted(scores.items(), key=lambda kv: (-kv[1], kv[0]))
print(dict(sorted_items))  # {'Bob': 95, 'Alice': 88, 'Charlie': 88}
```

**Key Points to Remember:**
- Because Python's sort is **stable**, multi-pass sorting (least significant key first) works for any type, even strings when mixing `reverse=True` and `reverse=False`.

---

### Common Algorithmic Problems

#### Q123. Valid Parentheses: Check if a string containing `()[]{}` is balanced.

**Answer:**
Use a **stack** (`list`): push opening brackets; when encountering a closing bracket, ensure the stack is non-empty and the top matches its corresponding opening bracket. At the end, the stack must be empty.

```python
def is_valid_parentheses(s: str) -> bool:
    pairs = {")": "(", "]": "[", "}": "{"}
    stack = []
    for ch in s:
        if ch in pairs:
            if not stack or stack.pop() != pairs[ch]:
                return False
        else:
            stack.append(ch)
    return not stack

print(is_valid_parentheses("({[]})"))  # True
print(is_valid_parentheses("([)]"))    # False
```

**Key Points to Remember:**
- Time complexity: **$O(n)$**, Space complexity: **$O(n)$**.

---

#### Q124. Find all primes up to `N` using the Sieve of Eratosthenes, and compute GCD / LCM.

**Answer:**
- **Sieve of Eratosthenes:** Maintain a boolean `is_prime` array of size `n + 1`. For each prime `p` up to $\sqrt{n}$, mark multiples starting from $p^2$ as `False`. Time complexity: $O(n \log \log n)$.
- **GCD & LCM:** Euclidean algorithm (`math.gcd` and `math.lcm` in Python 3.9+).

```python
import math

def sieve_primes(n: int) -> list[int]:
    if n < 2:
        return []
    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False
    for p in range(2, math.isqrt(n) + 1):
        if is_prime[p]:
            is_prime[p * p : n + 1 : p] = [False] * len(range(p * p, n + 1, p))
    return [i for i, flag in enumerate(is_prime) if flag]

def gcd(a: int, b: int) -> int:
    while b:
        a, b = b, a % b
    return abs(a)

print(sieve_primes(20))   # [2, 3, 5, 7, 11, 13, 17, 19]
print(gcd(48, 18))        # 6
```

**Key Points to Remember:**
- Slice assignment `is_prime[p*p::p] = ...` in CPython runs the inner marking loop in C, making the sieve very fast.

---

#### Q125. Reverse a singly linked list and detect a cycle (Floyd's Tortoise and Hare).

**Answer:**
- **Reverse linked list:** Maintain `prev = None` and `curr = head`; at each step save `nxt = curr.next`, point `curr.next = prev`, and advance `prev, curr = curr, nxt`. ($O(n)$ time, $O(1)$ space).
- **Cycle detection:** Advance `slow` by 1 step and `fast` by 2 steps; if they meet, a cycle exists ($O(n)$ time, $O(1)$ space).

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def reverse_list(head: ListNode | None) -> ListNode | None:
    prev, curr = None, head
    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt
    return prev

def has_cycle(head: ListNode | None) -> bool:
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True
    return False
```

**Key Points to Remember:**
- Use `slow is fast` (identity comparison) when checking node equality in linked-list cycle detection.
- Both algorithms run in **$O(n)$ time and $O(1)$ extra space**.
