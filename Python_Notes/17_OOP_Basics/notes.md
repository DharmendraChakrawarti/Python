# Object-Oriented Programming (OOP) — Basics

Object-Oriented Programming (OOP) is a programming paradigm centered around **objects**—bundles of data (attributes) and behavior (methods)—rather than standalone functions and raw data structures. Python supports multiple paradigms, and OOP is foundational to its standard library, frameworks, and everyday Python design (in Python, *everything is an object*).

---

## Table of Contents

1. [What is OOP? Procedural vs OOP](#1-what-is-oop-procedural-vs-oop)
2. [Classes and Objects](#2-classes-and-objects)
3. [Creating a Class (`class` keyword)](#3-creating-a-class-class-keyword)
4. [The `__init__` Method (Initializer / Constructor)](#4-the-__init__-method-initializer--constructor)
5. [The `self` Parameter](#5-the-self-parameter)
6. [Instance Variables and Methods](#6-instance-variables-and-methods)
7. [Class Variables and Methods (`@classmethod`)](#7-class-variables-and-methods-classmethod)
8. [Static Methods (`@staticmethod`)](#8-static-methods-staticmethod)
9. [Instance Method vs Class Method vs Static Method](#9-instance-method-vs-class-method-vs-static-method)
10. [Access Modifiers (`public`, `_protected`, `__private`)](#10-access-modifiers-public-_protected-__private)
11. [Name Mangling](#11-name-mangling)
12. [The `__str__` and `__repr__` Methods](#12-the-__str__-and-__repr__-methods)
13. [The `__del__` Method (Destructor)](#13-the-__del__-method-destructor)
14. [The `@property` Decorator (Getter, Setter, Deleter)](#14-the-property-decorator-getter-setter-deleter)
15. [Encapsulation](#15-encapsulation)
16. [Object Introspection (`dir`, `hasattr`, `getattr`, `setattr`, `delattr`)](#16-object-introspection-dir-hasattr-getattr-setattr-delattr)

---

## 1. What is OOP? Procedural vs OOP

### Procedural Programming
In **procedural programming**, a program is structured as a sequence of instructions and reusable functions that operate on separate data (such as dictionaries, lists, and global variables).

- Focuses on **actions/verbs** (functions) and step-by-step execution.
- Data and the functions that manipulate that data live separately.
- Works well for small scripts, pipelines, and straightforward tasks, but can become difficult to maintain as state and rules grow complex.

### Object-Oriented Programming (OOP)
In **OOP**, data and the operations on that data are grouped together into **objects** created from **classes**.

- Focuses on **entities/nouns** (objects) and how they interact.
- Encourages modularity, reusability, clear boundaries around state, and modeling real-world or domain concepts directly.
- Core pillars of OOP: **Encapsulation**, **Abstraction**, **Inheritance**, and **Polymorphism** (this note focuses on OOP basics and encapsulation).

### Procedural vs OOP Comparison

| Aspect | Procedural Programming | Object-Oriented Programming |
| :--- | :--- | :--- |
| **Primary unit** | Functions and procedures | Classes and objects |
| **Data & logic** | Stored separately; data is passed into functions | Bundled together inside an object |
| **State management** | Often relies on global variables or explicit dicts | Managed internally by each object (`self`) |
| **Scalability** | Harder to coordinate as codebase grows | Easier to organize into modular, independent units |
| **Reusability** | Achieved by calling utility functions | Achieved via classes, composition, and inheritance |

### Side-by-Side Example

```python
# --- Procedural Approach ---
account = {"owner": "Alice", "balance": 1000}

def deposit(acc, amount):
    acc["balance"] += amount

def withdraw(acc, amount):
    if amount <= acc["balance"]:
        acc["balance"] -= amount

deposit(account, 250)
withdraw(account, 100)
print(account["balance"])  # 1150


# --- Object-Oriented Approach ---
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount

    def withdraw(self, amount):
        if amount <= self.balance:
            self.balance -= amount

acc = BankAccount("Alice", 1000)
acc.deposit(250)
acc.withdraw(100)
print(acc.balance)  # 1150
```

---

## 2. Classes and Objects

- **Class**: A blueprint, template, or user-defined type that describes what attributes (data) and methods (behavior) its instances will have. Defining a class does not represent a specific entity on its own.
- **Object (Instance)**: A concrete entity created from a class, occupying its own memory and holding its own state. Calling a class like a function (`ClassName(...)`) **instantiates** an object.

Think of a **class** as an architectural blueprint for a house, and **objects** as the actual houses built from that blueprint—each has the same layout, but each can have a different paint color, address, and occupants.

```python
class Car:
    pass

# Creating two independent objects (instances) of Car
car1 = Car()
car2 = Car()

print(type(car1))       # <class '__main__.Car'>
print(isinstance(car1, Car))  # True
print(car1 is car2)     # False (two distinct objects in memory)
```

> **Note:** In Python, built-in types like `int`, `str`, and `list` are also classes, and values like `42`, `"hello"`, and `[1, 2]` are objects (instances) of those classes.

---

## 3. Creating a Class (`class` keyword)

Classes are defined using the `class` keyword followed by the class name and a colon. By Python convention (**PEP 8**), class names use `PascalCase` (CapWords), such as `BankAccount` or `HttpClient`.

```python
class Dog:
    """Represents a dog with a name and breed."""

    species = "Canis familiaris"  # Class attribute

    def bark(self):
        return "Woof!"

# Instantiating the class
my_dog = Dog()
print(my_dog.species)  # Canis familiaris
print(my_dog.bark())   # Woof!
print(Dog.__doc__)     # Represents a dog with a name and breed.
```

### Anatomy of a Class Definition
1. `class Dog:` — Declares the class `Dog`.
2. **Docstring** (`"""..."""`) — Optional (strongly recommended) documentation describing the class.
3. **Class body** — Indented block containing attributes, methods, and nested definitions.
4. Use `pass` inside an empty class body if you only want a placeholder class.

---

## 4. The `__init__` Method (Initializer / Constructor)

`__init__` is a special method (often called a **dunder** method for *double underscore*) that Python automatically runs immediately after creating a new instance of a class. Its job is to **initialize** the new object's attributes.

> **Terminology note:** Technically, `__new__` creates and returns the new instance (the true constructor), while `__init__` initializes the already-created instance. In everyday Python, `__init__` is commonly referred to as the constructor.

```python
class Student:
    def __init__(self, name, grade=0):
        # Set initial state on the newly created instance
        self.name = name
        self.grade = grade

s1 = Student("Maya", 92)
s2 = Student("Liam")  # Uses default grade=0

print(s1.name, s1.grade)  # Maya 92
print(s2.name, s2.grade)  # Liam 0
```

### Key Rules for `__init__`
- Must be named exactly `__init__` (two underscores before and after `init`).
- Its first parameter is always `self`.
- It **must return `None`** (i.e., have no `return` value, or only bare `return`). Returning anything other than `None` raises a `TypeError`.
- You can define default parameter values, type hints, and validation inside `__init__`.

---

## 5. The `self` Parameter

`self` is a reference to the **current instance** on which a method is being called. It lets an object read and modify its own attributes and call its own methods.

- Python passes the instance automatically as the **first argument** when you call a method on an object.
- Naming it `self` is a strong universal convention (PEP 8), not a reserved keyword—always use `self`.

```python
class Counter:
    def __init__(self, start=0):
        self.value = start

    def increment(self, step=1):
        self.value += step

    def display(self):
        print(f"Current value: {self.value}")

c1 = Counter(10)
c2 = Counter(100)

c1.increment(5)
# Under the hood, Python translates c1.increment(5) to:
# Counter.increment(c1, 5)

c1.display()  # Current value: 15
c2.display()  # Current value: 100
```

Notice how `c1` and `c2` share the same `Counter.increment` method code, yet `self` ensures `c1.increment(5)` only modifies `c1.value` and leaves `c2.value` untouched.

---

## 6. Instance Variables and Methods

### Instance Variables (Instance Attributes)
Variables attached to `self` (usually inside `__init__`, though they can be added or modified in other methods too).
- **Unique to each object**: changing `obj1.attr` does not affect `obj2.attr`.
- Stored in each object's own `__dict__` dictionary (unless `__slots__` is used).

### Instance Methods
Functions defined inside a class that take `self` as their first parameter.
- Can read and modify **instance state** (`self.x`).
- Can also access **class state** (`self.__class__.y` or `ClassName.y`).

```python
class Book:
    def __init__(self, title, author, pages):
        # Instance variables
        self.title = title
        self.author = author
        self.pages = pages
        self.current_page = 1

    # Instance methods
    def read(self, num_pages):
        self.current_page = min(self.pages, self.current_page + num_pages)

    def progress(self):
        pct = (self.current_page / self.pages) * 100
        return f"'{self.title}' by {self.author}: {pct:.1f}% read"

b1 = Book("1984", "George Orwell", 328)
b2 = Book("Dune", "Frank Herbert", 412)

b1.read(81)
print(b1.progress())  # '1984' by George Orwell: 25.0% read
print(b2.progress())  # 'Dune' by Frank Herbert: 0.2% read
print(b1.__dict__)    # {'title': '1984', 'author': 'George Orwell', 'pages': 328, 'current_page': 82}
```

---

## 7. Class Variables and Methods (`@classmethod`)

### Class Variables (Class Attributes)
Variables defined directly inside the class body, outside any method.
- **Shared across the class and all its instances**.
- Useful for constants, default configuration, or tracking class-wide state (such as instance counts).

```python
class Employee:
    company = "TechCorp"   # Class variable (shared)
    employee_count = 0     # Class variable (shared counter)

    def __init__(self, name):
        self.name = name   # Instance variable (per-object)
        Employee.employee_count += 1

e1 = Employee("Ana")
e2 = Employee("Bob")

print(e1.company, e2.company)      # TechCorp TechCorp
print(Employee.employee_count)     # 2
```

#### Attribute Lookup Gotcha: Shadowing a Class Variable
When you read `obj.attr`, Python checks `obj.__dict__` first, then `Class.__dict__`.
When you **assign** `obj.attr = value`, Python always creates/updates an **instance variable** on `obj`, shadowing the class variable for that instance only!

```python
e1.company = "StartupX"       # Creates instance attribute on e1 only!
print(e1.company)             # StartupX (from e1.__dict__)
print(e2.company)             # TechCorp (still from Employee)
print(Employee.company)       # TechCorp

Employee.company = "MegaCorp" # Modifies class variable
print(e2.company)             # MegaCorp
print(e1.company)             # StartupX (still shadowed on e1)
```

> **Warning with mutable class variables:** Avoid using mutable objects (like `[]` or `{}`) as class variables intended to hold per-instance data, because mutating `self.items.append(...)` modifies the single shared list across all instances! Initialize mutable per-instance attributes inside `__init__`.

### Class Methods (`@classmethod`)
Methods decorated with `@classmethod` receive the **class itself** (`cls`) as their first parameter instead of an instance (`self`).
- Can access and modify **class state** (`cls.attr`).
- Cannot directly access instance-specific attributes (`self.attr`) because no instance is passed.
- Very commonly used as **alternative constructors** (factory methods) or to update class-level settings.

```python
class Date:
    default_separator = "-"

    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, date_str):
        """Alternative constructor parsing 'YYYY-MM-DD'."""
        year, month, day = map(int, date_str.split(cls.default_separator))
        return cls(year, month, day)  # Calls Date(year, month, day) — inheritance-friendly!

    @classmethod
    def set_separator(cls, sep):
        cls.default_separator = sep

d1 = Date(2026, 11, 26)
d2 = Date.from_string("2026-11-26")
print(d2.year, d2.month, d2.day)  # 2026 11 26
```

Using `cls(...)` rather than hardcoding `Date(...)` inside a `@classmethod` ensures subclasses calling `SubDate.from_string(...)` return an instance of `SubDate`.

---

## 8. Static Methods (`@staticmethod`)

Methods decorated with `@staticmethod` do **not** receive `self` or `cls` automatically. They behave like regular functions, except they live inside the class's namespace because they are logically related to that class.

- Cannot access or modify instance state (`self`) or class state (`cls`).
- Useful for helper utilities, validation, formatting, or pure computations related to the class.
- Can be called via the class (`ClassName.method(...)`) or an instance (`obj.method(...)`).

```python
class MathUtils:
    @staticmethod
    def is_even(n):
        return n % 2 == 0

    @staticmethod
    def clamp(value, low, high):
        return max(low, min(value, high))

print(MathUtils.is_even(4))         # True
print(MathUtils.clamp(120, 0, 100)) # 100

m = MathUtils()
print(m.is_even(7))                 # False
```

---

## 9. Instance Method vs Class Method vs Static Method

| Feature | Instance Method | Class Method (`@classmethod`) | Static Method (`@staticmethod`) |
| :--- | :--- | :--- | :--- |
| **Decorator** | None | `@classmethod` | `@staticmethod` |
| **First implicit parameter** | `self` (the instance) | `cls` (the class) | None |
| **Can read/modify instance state?** | Yes (`self.x`) | No | No |
| **Can read/modify class state?** | Yes (via `self.__class__` or `ClassName`) | Yes (`cls.x`) | No (unless hardcoding class name) |
| **Typical use case** | Operations on a specific object's data | Alternative constructors, class-wide config | Pure utility/helper functions tied to the class |
| **Called on** | Instance (usually) | Class or instance | Class or instance |

### Combined Example

```python
class Pizza:
    tax_rate = 0.08  # Class variable

    def __init__(self, ingredients):
        self.ingredients = ingredients  # Instance variable

    # 1. Instance method: needs `self`
    def describe(self):
        return f"Pizza with {', '.join(self.ingredients)}"

    # 2. Class method: needs `cls` (alternative constructor)
    @classmethod
    def margherita(cls):
        return cls(["mozzarella", "tomatoes", "basil"])

    @classmethod
    def set_tax_rate(cls, rate):
        cls.tax_rate = rate

    # 3. Static method: needs neither `self` nor `cls`
    @staticmethod
    def is_valid_size(inches):
        return inches in (10, 12, 14, 16)

p = Pizza.margherita()
print(p.describe())             # Pizza with mozzarella, tomatoes, basil
print(Pizza.is_valid_size(14))  # True
```

---

## 10. Access Modifiers (`public`, `_protected`, `__private`)

Unlike languages such as Java or C++, Python does **not** enforce strict compile-time or runtime access control. Instead, it follows the philosophy: *"We are all consenting adults here."* Access levels are expressed through **naming conventions** (and name mangling for double underscores):

| Modifier | Syntax | Meaning | Accessible Outside Class? |
| :--- | :--- | :--- | :--- |
| **Public** | `name` | Part of the public API; freely readable and writable | Yes |
| **Protected** | `_name` | Internal/implementation detail; intended for the class and its subclasses | Yes (by convention, avoid direct external access) |
| **Private** | `__name` | Strongly internal to the defining class; triggers **name mangling** | Not directly as `obj.__name` (raises `AttributeError`) |

> **Note:** Names with double underscores on *both* sides like `__init__` or `__str__` are **dunder (magic) methods** reserved for Python's data model—they are *not* private and do *not* undergo name mangling.

```python
class Account:
    def __init__(self, owner, balance, pin):
        self.owner = owner        # Public
        self._balance = balance   # Protected (convention)
        self.__pin = pin          # Private (name-mangled)

    def check_pin(self, guess):
        return guess == self.__pin

acc = Account("Sam", 500, 1234)

print(acc.owner)        # Sam (public: OK)
print(acc._balance)     # 500 (protected: works, but discouraged outside class/subclasses)
print(acc.check_pin(1234))  # True

# Direct access to __pin fails:
# print(acc.__pin)      # AttributeError: 'Account' object has no attribute '__pin'
```

---

## 11. Name Mangling

When an identifier inside a class starts with **at least two leading underscores** and ends with **at most one trailing underscore** (e.g., `__pin` or `__helper_`), Python automatically rewrites its name internally to:

```text
_ClassName__attributeName
```

This mechanism is called **name mangling**.

### Why Does Name Mangling Exist?
Its primary purpose is **avoiding accidental name collisions in subclasses**, not strict security. If a parent class and a subclass both define an internal attribute `__id`, name mangling keeps `_Parent__id` and `_Child__id` separate so one doesn't silently overwrite the other.

```python
class SecretBox:
    def __init__(self, secret):
        self.__secret = secret

    def reveal(self):
        return self.__secret

box = SecretBox("treasure")

print(box.__dict__)            # {'_SecretBox__secret': 'treasure'}
print(box.reveal())            # treasure
print(box._SecretBox__secret)  # treasure (still accessible via mangled name)
```

### Preventing Collisions in Subclasses

```python
class Base:
    def __init__(self):
        self.__value = "base"

    def get_base_value(self):
        return self.__value

class Child(Base):
    def __init__(self):
        super().__init__()
        self.__value = "child"  # Does NOT overwrite Base's __value

    def get_child_value(self):
        return self.__value

c = Child()
print(c.get_base_value())   # base
print(c.get_child_value())  # child
print(c.__dict__)           # {'_Base__value': 'base', '_Child__value': 'child'}
```

---

## 12. The `__str__` and `__repr__` Methods

Both `__str__` and `__repr__` control how an object is converted into a string, but they serve different audiences:

- **`__repr__(self)`** — **Developer-facing** representation.
  - Called by `repr(obj)`, interactive REPL prompts, and inside containers (`print([obj])`).
  - Goal: Be **unambiguous**; ideally return a valid Python expression that could recreate the object (e.g., `Point(3, 4)`).
  - Used as a **fallback** when `__str__` is not defined.
- **`__str__(self)`** — **User-facing** representation.
  - Called by `str(obj)`, `print(obj)`, and `f"{obj}"`.
  - Goal: Be **readable** and friendly for end users.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point(x={self.x!r}, y={self.y!r})"

    def __str__(self):
        return f"({self.x}, {self.y})"

p = Point(3, 4)

print(p)         # (3, 4)                  -> calls __str__
print(str(p))    # (3, 4)                  -> calls __str__
print(repr(p))   # Point(x=3, y=4)         -> calls __repr__
print([p])       # [Point(x=3, y=4)]       -> containers use __repr__
print(f"{p!r}")  # Point(x=3, y=4)         -> !r forces __repr__
```

> **Best Practice:** Always implement `__repr__` first. If you only implement one, implement `__repr__` because `str(obj)` automatically falls back to `__repr__`.

---

## 13. The `__del__` Method (Destructor)

`__del__(self)` is a special **finalizer** method called when an object's reference count drops to zero and it is about to be destroyed by Python's garbage collector.

```python
class TempResource:
    def __init__(self, name):
        self.name = name
        print(f"Acquired {self.name}")

    def __del__(self):
        print(f"Released {self.name}")

res = TempResource("cache_handle")  # Acquired cache_handle
del res                             # Released cache_handle (when refcount hits 0)
```

### Important Caveats About `__del__`
1. **`del obj` does NOT directly call `__del__`**: `del obj` only removes the variable name and decreases the reference count by 1. `__del__` only runs if no other references to the object remain.
2. **Unpredictable timing**: With reference cycles, garbage collection timing is non-deterministic, and `__del__` might run much later or at interpreter shutdown (when globals/modules may already be torn down).
3. **Prefer Context Managers (`with` statement)**: For deterministic cleanup of files, sockets, locks, or database connections, implement `__enter__` and `__exit__` (or use `contextlib`) instead of relying on `__del__`.

```python
a = TempResource("A")
b = a         # Reference count is now 2
del a         # Reference count drops to 1 — __del__ does NOT run yet!
print("After del a")
del b         # Reference count drops to 0 — NOW __del__ runs
```

---

## 14. The `@property` Decorator (Getter, Setter, Deleter)

`@property` lets you define methods that are accessed **like regular attributes** (`obj.temp` instead of `obj.get_temp()`). This enables validation, computed attributes, and read-only attributes without changing how callers access the attribute.

- `@property` defines the **getter** (read access).
- `@<prop>.setter` defines the **setter** (write access, optional).
- `@<prop>.deleter` defines the **deleter** (delete access with `del obj.prop`, optional).

```python
class Celsius:
    def __init__(self, temperature=0):
        # Calls the @temperature.setter during initialization!
        self.temperature = temperature

    @property
    def temperature(self):
        """Getter: returns temperature in Celsius."""
        return self._temperature

    @temperature.setter
    def temperature(self, value):
        """Setter: enforces absolute zero constraint."""
        if value < -273.15:
            raise ValueError("Temperature cannot be below -273.15°C")
        self._temperature = value

    @temperature.deleter
    def temperature(self):
        """Deleter: resets or removes the backing attribute."""
        print("Resetting temperature...")
        del self._temperature

    @property
    def fahrenheit(self):
        """Read-only computed property (no setter defined)."""
        return (self._temperature * 9 / 5) + 32


c = Celsius(25)
print(c.temperature)   # 25      (invokes getter)
print(c.fahrenheit)    # 77.0    (computed dynamically)

c.temperature = 100    # (invokes setter)
print(c.fahrenheit)    # 212.0

# c.temperature = -300 # Raises ValueError
# c.fahrenheit = 50    # Raises AttributeError (read-only property)

del c.temperature      # Resetting temperature... (invokes deleter)
```

> **Common Pitfall:** Inside a property getter/setter, store the actual value in a backing attribute like `self._temperature` (with a leading underscore), **not** `self.temperature`. Writing `self.temperature = value` inside `@temperature.setter` causes infinite recursion!

---

## 15. Encapsulation

**Encapsulation** is the practice of bundling data (attributes) and the methods that operate on that data into a single unit (a class), while **restricting direct uncontrolled access** to internal state and exposing a clean, validated public interface.

### Why Encapsulation Matters
- **Data Integrity**: Prevents invalid states (e.g., negative bank balance or invalid age) through validation in methods and setters.
- **Flexibility**: You can change internal implementation details (e.g., renaming `_items` or switching from a list to a dictionary) without breaking external code that uses the public methods.
- **Clarity**: Separates *what* an object does (public interface) from *how* it does it (internal details).

### Pythonic Encapsulation Example
In Python, encapsulation combines `_protected` / `__private` attributes with public methods and `@property`:

```python
class BankAccount:
    def __init__(self, owner, initial_balance=0):
        self.owner = owner
        self.__balance = 0
        if initial_balance < 0:
            raise ValueError("Initial balance cannot be negative")
        self.__balance = initial_balance

    @property
    def balance(self):
        """Read-only view of the balance."""
        return self.__balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self.__balance += amount

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self.__balance:
            raise ValueError("Insufficient funds")
        self.__balance -= amount

acc = BankAccount("Elena", 500)
acc.deposit(200)
acc.withdraw(150)
print(acc.balance)  # 550

# acc.balance = 999999  # AttributeError: property 'balance' has no setter
# acc.withdraw(1000)    # ValueError: Insufficient funds
```

---

## 16. Object Introspection (`dir`, `hasattr`, `getattr`, `setattr`, `delattr`)

**Introspection** is the ability to inspect and manipulate an object's type, attributes, and methods at **runtime**. Python provides five core built-in functions for attribute introspection:

| Function | Signature | Purpose |
| :--- | :--- | :--- |
| `dir()` | `dir(obj)` | Returns a sorted list of valid attribute and method names for `obj` |
| `hasattr()` | `hasattr(obj, name)` | Returns `True` if `obj` has attribute `name`, else `False` |
| `getattr()` | `getattr(obj, name[, default])` | Returns the value of attribute `name` (or `default` if missing; raises `AttributeError` if no default) |
| `setattr()` | `setattr(obj, name, value)` | Sets `obj.name = value` dynamically (creates it if it doesn't exist) |
| `delattr()` | `delattr(obj, name)` | Deletes attribute `name` from `obj` (equivalent to `del obj.name`) |

> **Note:** In `hasattr`, `getattr`, `setattr`, and `delattr`, `name` is passed as a **string**, making them ideal when attribute names are stored in variables or determined dynamically at runtime.

### Example

```python
class User:
    role = "member"

    def __init__(self, username, email):
        self.username = username
        self.email = email

    def greet(self):
        return f"Hi, I'm {self.username}"

u = User("jdoe", "jdoe@example.com")

# 1. dir() — list attributes & methods (including inherited dunder attributes)
public_attrs = [attr for attr in dir(u) if not attr.startswith("__")]
print(public_attrs)  # ['email', 'greet', 'role', 'username']

# 2. hasattr() — check existence safely
print(hasattr(u, "username"))  # True
print(hasattr(u, "age"))       # False

# 3. getattr() — retrieve dynamically with optional fallback
print(getattr(u, "username"))            # jdoe
print(getattr(u, "age", "Not specified"))  # Not specified

# Calling a method dynamically via getattr
greet_fn = getattr(u, "greet")
print(greet_fn())  # Hi, I'm jdoe

# 4. setattr() — set or add attribute dynamically
setattr(u, "age", 28)
setattr(u, "username", "jane_doe")
print(u.age, u.username)  # 28 jane_doe

# 5. delattr() — remove attribute dynamically
delattr(u, "age")
print(hasattr(u, "age"))  # False
```

### Other Useful Introspection Built-ins
- `type(obj)` — Returns the exact class of `obj`.
- `isinstance(obj, ClassOrTuple)` — Checks whether `obj` is an instance of a class (or any subclass of it).
- `issubclass(Sub, Base)` — Checks whether `Sub` is a subclass of `Base`.
- `id(obj)` — Returns the unique identity (memory address in CPython) of `obj`.
- `vars(obj)` / `obj.__dict__` — Returns the dictionary of writable instance attributes.

---

## Quick Summary Checklist

- Use `class Name:` and `__init__(self, ...)` to define blueprints and initialize per-object state.
- Use `self` for instance attributes/methods, `@classmethod` (`cls`) for class-level logic and factory constructors, and `@staticmethod` for class-namespaced helpers.
- Signal intent with `_protected` and avoid subclass collisions with `__private` (name mangling: `_Class__attr`).
- Implement `__repr__` (developer string) and `__str__` (user string); prefer context managers over `__del__` for cleanup.
- Use `@property` + `.setter` + `.deleter` to achieve clean, Pythonic encapsulation without boilerplate getters/setters.
- Use `dir`, `hasattr`, `getattr`, `setattr`, and `delattr` to inspect and manipulate objects dynamically at runtime.
