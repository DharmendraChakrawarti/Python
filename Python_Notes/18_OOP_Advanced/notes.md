# Object-Oriented Programming — Advanced

Advanced Object-Oriented Programming in Python goes beyond basic classes and instances to cover inheritance architectures, Python's data model (dunder methods), type protocols, design principles, and metaprogramming tools like descriptors, metaclasses, `__slots__`, dataclasses, and enums.

---

## Table of Contents

1. [Inheritance (Single Inheritance)](#1-inheritance-single-inheritance)
2. [`super()` Function](#2-super-function)
3. [Method Overriding](#3-method-overriding)
4. [Multiple Inheritance](#4-multiple-inheritance)
5. [Method Resolution Order (MRO) and C3 Linearization](#5-method-resolution-order-mro-and-c3-linearization)
6. [Mixins](#6-mixins)
7. [Abstract Classes and Methods (`abc` Module)](#7-abstract-classes-and-methods-abc-module)
8. [Interfaces in Python](#8-interfaces-in-python)
9. [Polymorphism (Duck Typing)](#9-polymorphism-duck-typing)
10. [Operator Overloading (Magic / Dunder Methods)](#10-operator-overloading-magic--dunder-methods)
11. [Composition vs Inheritance](#11-composition-vs-inheritance)
12. [SOLID Principles in Python](#12-solid-principles-in-python)
13. [Design Patterns Basics](#13-design-patterns-basics)
14. [Metaclasses (`type`, `__new__` vs `__init__`)](#14-metaclasses-type-__new__-vs-__init__)
15. [Descriptors (`__get__`, `__set__`, `__delete__`)](#15-descriptors-__get__-__set__-__delete__)
16. [`__slots__`](#16-__slots__)
17. [Dataclasses (`@dataclass`)](#17-dataclasses-dataclass)
18. [Enums (`enum` Module)](#18-enums-enum-module)

---

## 1. Inheritance (Single Inheritance)

**Single inheritance** allows a **child class** (subclass / derived class) to inherit attributes and methods from one **parent class** (base class / superclass). It models an **"is-a"** relationship (e.g., a `Dog` *is an* `Animal`).

### Key Concepts
- All classes in Python implicitly inherit from `object` (`class Foo:` is equivalent to `class Foo(object):`).
- Subclasses automatically inherit non-private attributes and methods of the parent and can add new ones.
- Use `isinstance(obj, Class)` to check if an object is an instance of `Class` or any of its subclasses.
- Use `issubclass(Sub, Base)` to check class hierarchy relationships.

```python
class Employee:
    company = "Acme Corp"

    def __init__(self, name: str, base_salary: float) -> None:
        self.name = name
        self.base_salary = base_salary

    def get_pay(self) -> float:
        return self.base_salary

    def describe(self) -> str:
        return f"{self.name} works at {self.company}"


class Developer(Employee):
    def __init__(self, name: str, base_salary: float, language: str) -> None:
        super().__init__(name, base_salary)
        self.language = language

    def write_code(self) -> str:
        return f"{self.name} is writing {self.language}"


dev = Developer("Alice", 95_000, "Python")
print(dev.describe())     # Alice works at Acme Corp (inherited)
print(dev.write_code())   # Alice is writing Python (new method)
print(dev.get_pay())      # 95000 (inherited)

print(isinstance(dev, Developer))    # True
print(isinstance(dev, Employee))     # True
print(issubclass(Developer, Employee))  # True
```

---

## 2. `super()` Function

`super()` returns a **proxy object** that delegates method calls to the next class in the **Method Resolution Order (MRO)**—not necessarily just the direct parent class.

### Why Use `super()` Instead of `ParentClass.method(self, ...)`?
1. **Maintainability**: If you rename or change the base class, `super()` calls do not need to be updated.
2. **Cooperative Multiple Inheritance**: Calling a parent class directly by name can cause base methods (like `__init__`) to be called multiple times or skipped in diamond hierarchies. `super()` ensures each class in the MRO runs once.
3. **Python 3 Zero-Argument Syntax**: Inside a class method, `super()` automatically resolves to `super(CurrentClass, self)` (or `super(CurrentClass, cls)` in `@classmethod`).

```python
class Logger:
    def log(self, message: str) -> None:
        print(f"[LOG] {message}")


class TimestampLogger(Logger):
    def log(self, message: str) -> None:
        from datetime import datetime, timezone
        ts = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M:%S")
        # Delegates to the next class in MRO (Logger)
        super().log(f"{ts} - {message}")


logger = TimestampLogger()
logger.log("Server started")
# [LOG] 2026-11-26 02:20:00 - Server started
```

> **Note:** In Python, calling `super().__init__(...)` is **not** automatic—if a subclass defines `__init__`, it must explicitly invoke `super().__init__(...)` if parent initialization is needed.

---

## 3. Method Overriding

**Method overriding** occurs when a subclass defines a method with the same name as a method in its parent class, replacing or extending the parent's behavior.

- **Complete replacement**: Implement a new body without calling `super()`.
- **Extending behavior**: Call `super().method(...)` and add pre- or post-processing.
- **Type-checker safety (Python 3.12+)**: `@typing.override` marks a method as intentionally overriding a base method so static type checkers (mypy/pyright) catch typos or stale overrides.

```python
from typing import override


class Notification:
    def send(self, recipient: str, text: str) -> str:
        return f"Sending generic notification to {recipient}: {text}"


class EmailNotification(Notification):
    @override
    def send(self, recipient: str, text: str) -> str:
        # Completely replaces parent implementation
        return f"Email -> {recipient} | Subject: Alert | Body: {text}"


class AuditedEmailNotification(EmailNotification):
    @override
    def send(self, recipient: str, text: str) -> str:
        # Extends parent implementation using super()
        result = super().send(recipient, text)
        print(f"Audit trail recorded for {recipient}")
        return result


notifier = AuditedEmailNotification()
print(notifier.send("bob@example.com", "Disk 90% full"))
# Audit trail recorded for bob@example.com
# Email -> bob@example.com | Subject: Alert | Body: Disk 90% full
```

---

## 4. Multiple Inheritance

Python allows a class to inherit from **more than one** base class: `class Child(BaseA, BaseB):`.

### The Diamond Problem & Cooperative `super()`
When two parent classes share a common ancestor, naive inheritance could invoke the common ancestor twice:

```text
      Base
     /    \
  Left    Right
     \    /
     Child
```

To make multiple inheritance work cleanly with `super()`, classes should use **cooperative calls** passing `**kwargs` forward so every `__init__` in the MRO can consume the keyword arguments it knows about and forward the rest.

```python
class Device:
    def __init__(self, device_id: str, **kwargs) -> None:
        super().__init__(**kwargs)
        self.device_id = device_id


class Networked(Device):
    def __init__(self, ip_address: str, **kwargs) -> None:
        super().__init__(**kwargs)
        self.ip_address = ip_address


class Powered(Device):
    def __init__(self, wattage: int, **kwargs) -> None:
        super().__init__(**kwargs)
        self.wattage = wattage


class SmartPlug(Networked, Powered):
    def __init__(self, name: str, **kwargs) -> None:
        super().__init__(**kwargs)
        self.name = name


plug = SmartPlug(
    name="Desk Lamp",
    device_id="SP-101",
    ip_address="192.168.1.42",
    wattage=15,
)
print(plug.name, plug.device_id, plug.ip_address, plug.wattage)
# Desk Lamp SP-101 192.168.1.42 15
```

Because `Device.__init__` calls `super().__init__(**kwargs)` and `object.__init__` accepts no extra arguments, all keyword arguments must have been consumed by the time the chain reaches `object`.

---

## 5. Method Resolution Order (MRO) and C3 Linearization

**MRO** is the deterministic order in which Python searches base classes when looking up an attribute or method.

### C3 Linearization Rules
Python computes the MRO using the **C3 superclass linearization** algorithm, which guarantees three properties:
1. **Subclasses precede superclasses**: A child is always checked before its parents.
2. **Left-to-right base order**: In `class C(A, B):`, `A` is preferred over `B`.
3. **Monotonicity**: If `A` precedes `B` in one class's MRO, `A` must never appear *after* `B` in any subclass's MRO.

### Inspecting the MRO
- `ClassName.__mro__` (tuple of classes)
- `ClassName.mro()` (list of classes)

```python
class A:
    def ping(self) -> str:
        return "A"


class B(A):
    def ping(self) -> str:
        return "B -> " + super().ping()


class C(A):
    def ping(self) -> str:
        return "C -> " + super().ping()


class D(B, C):
    def ping(self) -> str:
        return "D -> " + super().ping()


print([cls.__name__ for cls in D.__mro__])
# ['D', 'B', 'C', 'A', 'object']

print(D().ping())
# D -> B -> C -> A
```

### Inconsistent Hierarchies (`TypeError`)
If no ordering satisfies all C3 constraints, Python rejects the class definition at creation time:

```python
class X:
    pass

class Y:
    pass

class A(X, Y):
    pass

class B(Y, X):
    pass

try:
    class Invalid(A, B):
        pass
except TypeError as exc:
    print(exc)
    # Cannot create a consistent method resolution order (MRO) for bases X, Y
```

---

## 6. Mixins

A **mixin** is a small, focused class designed to provide a reusable piece of functionality to other classes via multiple inheritance, rather than standing on its own as an independent entity.

### Mixin Conventions
- Named with a `Mixin` suffix (e.g., `JsonMixin`, `ComparableMixin`).
- Usually do **not** define `__init__` (or use cooperative `super().__init__` if they do).
- Placed **to the left** of the main base class (`class User(JsonMixin, BaseModel):`) so mixin methods take precedence in the MRO.
- Rely on attributes/methods provided by the host class or `self.__dict__`.

```python
import json
from datetime import datetime, timezone


class JsonSerializableMixin:
    def to_json(self) -> str:
        return json.dumps(self.__dict__, default=str)


class TimestampMixin:
    def stamp(self) -> None:
        self.updated_at = datetime.now(timezone.utc).isoformat()


class Record:
    def __init__(self, record_id: int) -> None:
        self.record_id = record_id


class UserRecord(JsonSerializableMixin, TimestampMixin, Record):
    def __init__(self, record_id: int, username: str) -> None:
        super().__init__(record_id)
        self.username = username
        self.stamp()


user = UserRecord(1, "alice")
print(user.to_json())
# {"record_id": 1, "username": "alice", "updated_at": "2026-11-26T..."}
```

---

## 7. Abstract Classes and Methods (`abc` Module)

An **Abstract Base Class (ABC)** defines a common blueprint for subclasses and enforces that subclasses implement required methods before they can be instantiated.

### Key Features of `abc`
- Inherit from `abc.ABC` (or set `metaclass=abc.ABCMeta`).
- Decorate required methods with `@abstractmethod`.
- Can combine `@property`, `@classmethod`, or `@staticmethod` with `@abstractmethod` (`@abstractmethod` must be the **innermost** decorator).
- ABCs **can** contain concrete (implemented) methods shared by all subclasses.
- Virtual subclasses can be registered with `MyABC.register(UnrelatedClass)` so `isinstance()` / `issubclass()` return `True` without inheritance (though abstract method enforcement does not apply to registered virtual subclasses).

```python
from abc import ABC, abstractmethod


class PaymentProcessor(ABC):
    @property
    @abstractmethod
    def provider_name(self) -> str:
        """Name of the payment provider."""

    @abstractmethod
    def charge(self, amount: float) -> bool:
        """Process a charge and return success status."""

    def receipt(self, amount: float) -> str:
        # Concrete helper shared by all subclasses
        return f"[{self.provider_name}] Charged ${amount:.2f}"


class StripeProcessor(PaymentProcessor):
    @property
    def provider_name(self) -> str:
        return "Stripe"

    def charge(self, amount: float) -> bool:
        return amount > 0


# PaymentProcessor() -> TypeError: Can't instantiate abstract class
processor = StripeProcessor()
if processor.charge(49.99):
    print(processor.receipt(49.99))  # [Stripe] Charged $49.99
```

---

## 8. Interfaces in Python

Python does not have an `interface` keyword. Instead, interfaces are expressed in two primary ways:

| Approach | Mechanism | Typing Style | Runtime Enforcement |
| :--- | :--- | :--- | :--- |
| **`abc.ABC`** | Inheritance | **Nominal** (must explicitly subclass) | Prevents instantiation if methods are missing |
| **`typing.Protocol`** | Structural subtyping | **Structural** (static duck typing—no inheritance required) | Static type checkers verify; optional `@runtime_checkable` for `isinstance()` |

### Using `typing.Protocol` (Modern Python Interfaces)

```python
from typing import Protocol, runtime_checkable


@runtime_checkable
class Serializable(Protocol):
    def serialize(self) -> bytes: ...


class Report:
    # Does NOT inherit from Serializable
    def __init__(self, title: str) -> None:
        self.title = title

    def serialize(self) -> bytes:
        return self.title.encode("utf-8")


def save_to_disk(item: Serializable) -> int:
    data = item.serialize()
    return len(data)


report = Report("Q4 Summary")
print(isinstance(report, Serializable))  # True (checks method presence, not signatures)
print(save_to_disk(report))              # 10
```

> **Tip:** Prefer `Protocol` when decoupling libraries or writing clean interfaces without forcing callers into a class hierarchy; prefer `ABC` when you want to share concrete base implementation and enforce completeness at instantiation time.

---

## 9. Polymorphism (Duck Typing)

**Polymorphism** lets different types be used interchangeably through a shared interface.

### Duck Typing
> *"If it walks like a duck and quacks like a duck, then it is a duck."*

In Python, an object's suitability is determined by the **methods and attributes it supports** at runtime rather than its inheritance hierarchy. Python pairs duck typing with **EAFP** (*Easier to Ask for Forgiveness than Permission*): try the operation and handle `AttributeError` / `TypeError` if needed, rather than over-checking types upfront.

```python
class FileStorage:
    def write(self, data: str) -> None:
        print(f"Writing to file: {data}")


class InMemoryBuffer:
    def __init__(self) -> None:
        self.items: list[str] = []

    def write(self, data: str) -> None:
        self.items.append(data)


def export_records(target, records: list[str]) -> None:
    # Works with any object providing .write(str)
    for rec in records:
        target.write(rec)


mem = InMemoryBuffer()
export_records(FileStorage(), ["row1", "row2"])
export_records(mem, ["row1", "row2"])
print(mem.items)  # ['row1', 'row2']
```

### Ad-hoc Polymorphism with `functools.singledispatch`
When you need behavior to vary by the runtime type of the first argument without modifying the classes themselves:

```python
from functools import singledispatch


@singledispatch
def format_value(val) -> str:
    return str(val)


@format_value.register
def _(val: float) -> str:
    return f"{val:.2f}"


@format_value.register
def _(val: list) -> str:
    return ", ".join(map(str, val))


print(format_value(3.14159))   # 3.14
print(format_value([1, 2, 3])) # 1, 2, 3
print(format_value("hello"))   # hello
```

---

## 10. Operator Overloading (Magic / Dunder Methods)

Python's data model uses **dunder** (double-underscore) methods to integrate custom classes with operators, built-in functions, containers, hashing, truthiness, and `with` statements.

### Return `NotImplemented` in Binary Operators
When `__add__`, `__sub__`, `__mul__`, `__eq__`, or `__lt__` encounters an unsupported type, return `NotImplemented` (not raise `TypeError`). This signals Python to try the reflected method on the other operand (e.g., `other.__radd__(self)`).

### Arithmetic & Comparison Methods
- `__add__(self, other)` -> `self + other`
- `__sub__(self, other)` -> `self - other`
- `__mul__(self, other)` -> `self * other`
- `__eq__(self, other)` -> `self == other`
- `__lt__(self, other)` -> `self < other` (with `@functools.total_ordering`, defining `__eq__` and `__lt__` auto-generates `<=`, `>`, `>=`)

```python
from functools import total_ordering


@total_ordering
class Vector:
    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

    def __repr__(self) -> str:
        return f"Vector({self.x}, {self.y})"

    def __add__(self, other):
        if not isinstance(other, Vector):
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        if not isinstance(other, Vector):
            return NotImplemented
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        if not isinstance(scalar, (int, float)):
            return NotImplemented
        return Vector(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):
        return self.__mul__(scalar)

    def __eq__(self, other) -> bool:
        if not isinstance(other, Vector):
            return NotImplemented
        return (self.x, self.y) == (other.x, other.y)

    def __lt__(self, other) -> bool:
        if not isinstance(other, Vector):
            return NotImplemented
        return (self.x**2 + self.y**2) < (other.x**2 + other.y**2)

    def __hash__(self) -> int:
        # Must be consistent with __eq__: equal objects must have equal hashes
        return hash((self.x, self.y))


v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)     # Vector(4, 6)
print(v2 - v1)     # Vector(2, 2)
print(v1 * 3)      # Vector(3, 6)
print(3 * v1)      # Vector(3, 6)
print(v1 < v2)     # True
print(v2 >= v1)    # True (from total_ordering)
print({v1: "start"})  # Works as dict key because __hash__ is implemented
```

> **Important `__hash__` Rule:** If you define `__eq__` on a class, Python automatically sets `__hash__ = None` (making instances unhashable) unless you explicitly implement `__hash__`. Only implement `__hash__` on **immutable** (or hash-stable) objects where equal instances always produce identical hashes.

---

### Container, Callable, and Truthiness Methods
- `__len__(self)` -> `len(obj)` (also used as fallback for `bool(obj)` if `__bool__` is absent)
- `__getitem__(self, key)` -> `obj[key]` (supports indexing, slicing, and iteration fallback)
- `__setitem__(self, key, value)` -> `obj[key] = value`
- `__contains__(self, item)` -> `item in obj`
- `__call__(self, *args, **kwargs)` -> makes instances callable like functions: `obj(...)`
- `__bool__(self)` -> `bool(obj)`, `if obj:`

```python
class ConfigBag:
    def __init__(self, initial: dict[str, str] | None = None) -> None:
        self._data: dict[str, str] = dict(initial or {})

    def __len__(self) -> int:
        return len(self._data)

    def __getitem__(self, key: str) -> str:
        return self._data[key]

    def __setitem__(self, key: str, value: str) -> None:
        self._data[key] = value

    def __contains__(self, key: object) -> bool:
        return key in self._data

    def __bool__(self) -> bool:
        return bool(self._data)

    def __call__(self, key: str, default: str = "") -> str:
        return self._data.get(key, default)


cfg = ConfigBag()
print(bool(cfg))          # False
cfg["env"] = "prod"       # __setitem__
print(cfg["env"])         # prod (__getitem__)
print("env" in cfg)       # True (__contains__)
print(len(cfg))           # 1 (__len__)
print(bool(cfg))          # True (__bool__)
print(cfg("port", "8080"))  # 8080 (__call__)
```

---

### Context Manager Protocol (`__enter__`, `__exit__`)
Objects implementing `__enter__` and `__exit__` can be used in `with` statements for deterministic resource management.
- `__enter__(self)`: Runs on entry; its return value is bound to the `as` variable.
- `__exit__(self, exc_type, exc_val, exc_tb)`: Runs on exit even if an exception occurred inside the `with` block. Returning a truthy value (like `True`) suppresses the exception; returning `None` or `False` lets it propagate.

```python
import time


class Timer:
    def __enter__(self) -> "Timer":
        self._start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb) -> bool:
        elapsed = time.perf_counter() - self._start
        print(f"Elapsed: {elapsed:.4f}s")
        if exc_type is not None:
            print(f"Exception inside block: {exc_val}")
        return False  # Propagate any exception normally


with Timer() as t:
    total = sum(range(100_000))
# Elapsed: 0.0021s
```

---

## 11. Composition vs Inheritance

| Aspect | Inheritance ("is-a") | Composition ("has-a") |
| :--- | :--- | :--- |
| **Coupling** | Tight (subclasses depend on parent internals) | Loose (depends only on component's public API) |
| **Flexibility** | Static (decided at class definition time) | Dynamic (components can be swapped at runtime) |
| **Hierarchy Risk** | Deep trees lead to the *fragile base class* problem | Flat, modular, easier to test with mocks/fakes |
| **Best Used For** | True taxonomic specialization or shared interface | Reusing behavior, combining orthogonal capabilities |

**Guideline:** *"Favor object composition over class inheritance."*

```python
class Engine:
    def start(self) -> str:
        return "Combustion engine started"


class ElectricMotor:
    def start(self) -> str:
        return "Electric motor humming silently"


class Car:
    # Composition: Car HAS-A propulsion unit that can be swapped
    def __init__(self, propulsion) -> None:
        self.propulsion = propulsion

    def drive(self) -> str:
        return f"Car moving -> {self.propulsion.start()}"


gas_car = Car(Engine())
ev_car = Car(ElectricMotor())
print(gas_car.drive())  # Car moving -> Combustion engine started
print(ev_car.drive())   # Car moving -> Electric motor humming silently
```

---

## 12. SOLID Principles in Python

SOLID is a set of five object-oriented design principles aimed at maintainable, extensible software.

### 1. Single Responsibility Principle (SRP)
A class should have **one reason to change**—each class owns one cohesive responsibility.

```python
# Good: separate data model from persistence
class Invoice:
    def __init__(self, items: list[float]) -> None:
        self.items = items

    def total(self) -> float:
        return sum(self.items)


class InvoiceRepository:
    def save(self, invoice: Invoice, path: str) -> None:
        with open(path, "w", encoding="utf-8") as f:
            f.write(f"Total: {invoice.total():.2f}\n")
```

### 2. Open/Closed Principle (OCP)
Software entities should be **open for extension, closed for modification**. Add new behavior via new classes/functions rather than editing existing `if/elif` chains.

```python
from abc import ABC, abstractmethod


class Discount(ABC):
    @abstractmethod
    def apply(self, price: float) -> float: ...


class PercentDiscount(Discount):
    def __init__(self, pct: float) -> None:
        self.pct = pct

    def apply(self, price: float) -> float:
        return price * (1 - self.pct / 100)


class FlatDiscount(Discount):
    def __init__(self, amount: float) -> None:
        self.amount = amount

    def apply(self, price: float) -> float:
        return max(0.0, price - self.amount)


def checkout(price: float, discount: Discount) -> float:
    return discount.apply(price)
```

### 3. Liskov Substitution Principle (LSP)
Subtypes must be **substitutable** for their base types without breaking caller expectations (no unexpected exceptions, narrowed inputs, or broken invariants).

```python
from abc import ABC, abstractmethod


# Violation example:
# Square inheriting from Rectangle and overriding set_width() to also change height
# breaks code expecting width and height on a Rectangle to vary independently.

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...


class Rectangle(Shape):
    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    def area(self) -> float:
        return self.width * self.height


class Square(Shape):
    def __init__(self, side: float) -> None:
        self.side = side

    def area(self) -> float:
        return self.side ** 2
```

### 4. Interface Segregation Principle (ISP)
Clients should not be forced to depend on methods they do not use. Prefer **small, focused protocols** over fat base classes.

```python
from typing import Protocol


class Printable(Protocol):
    def print_doc(self, content: str) -> None: ...


class Scannable(Protocol):
    def scan_doc(self) -> str: ...


class SimplePrinter:
    # Only needs to implement Printable, not Scannable
    def print_doc(self, content: str) -> None:
        print(f"Printing: {content}")
```

### 5. Dependency Inversion Principle (DIP)
High-level modules should not depend on low-level concrete implementations; **both should depend on abstractions**.

```python
from typing import Protocol


class Notifier(Protocol):
    def notify(self, msg: str) -> None: ...


class SMSNotifier:
    def notify(self, msg: str) -> None:
        print(f"SMS: {msg}")


class OrderService:
    # Depends on the Notifier abstraction, not concrete SMSNotifier
    def __init__(self, notifier: Notifier) -> None:
        self.notifier = notifier

    def place_order(self, order_id: int) -> None:
        self.notifier.notify(f"Order #{order_id} confirmed")


service = OrderService(SMSNotifier())
service.place_order(42)
```

---

## 13. Design Patterns Basics

### 1. Singleton
Ensures a class has only one shared instance. *(In Python, a module itself is often the simplest singleton; when a class is needed, override `__new__`.)*

```python
class AppConfig:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.debug = False
        return cls._instance


c1 = AppConfig()
c2 = AppConfig()
c1.debug = True
print(c1 is c2, c2.debug)  # True True
```

### 2. Factory
Encapsulates object creation logic so callers don't couple directly to concrete constructors.

```python
class JSONParser:
    def parse(self, raw: str) -> dict:
        import json
        return json.loads(raw)


class CSVParser:
    def parse(self, raw: str) -> list[list[str]]:
        return [line.split(",") for line in raw.strip().splitlines()]


def get_parser(fmt: str):
    parsers = {"json": JSONParser, "csv": CSVParser}
    if fmt not in parsers:
        raise ValueError(f"Unsupported format: {fmt}")
    return parsers[fmt]()


parser = get_parser("csv")
print(parser.parse("a,b\n1,2"))  # [['a', 'b'], ['1', '2']]
```

### 3. Observer
Defines a one-to-many subscription mechanism so observers are automatically notified when a subject's state changes.

```python
from typing import Callable


class EventBus:
    def __init__(self) -> None:
        self._subscribers: list[Callable[[str], None]] = []

    def subscribe(self, fn: Callable[[str], None]) -> None:
        self._subscribers.append(fn)

    def publish(self, event: str) -> None:
        for fn in self._subscribers:
            fn(event)


bus = EventBus()
bus.subscribe(lambda e: print(f"Email alert: {e}"))
bus.subscribe(lambda e: print(f"Metrics logged: {e}"))
bus.publish("user_signup")
```

### 4. Strategy
Defines a family of interchangeable algorithms and lets the caller swap them cleanly at runtime (functions or callable objects work naturally in Python).

```python
from typing import Callable


def standard_shipping(weight: float) -> float:
    return 5.0 + 1.5 * weight


def express_shipping(weight: float) -> float:
    return 15.0 + 3.0 * weight


class Cart:
    def __init__(self, shipping_strategy: Callable[[float], float]) -> None:
        self.shipping_strategy = shipping_strategy

    def shipping_cost(self, weight: float) -> float:
        return self.shipping_strategy(weight)


cart = Cart(standard_shipping)
print(cart.shipping_cost(4.0))  # 11.0
cart.shipping_strategy = express_shipping
print(cart.shipping_cost(4.0))  # 27.0
```

---

## 14. Metaclasses (`type`, `__new__` vs `__init__`)

In Python, **classes themselves are objects**—instances of a **metaclass**, which by default is `type`.

- `type(obj)` with 1 argument returns the object's class.
- `type(name, bases, namespace)` with 3 arguments dynamically constructs a new class.

### `__new__` vs `__init__` in a Metaclass
- `Meta.__new__(mcs, name, bases, namespace)`: Runs **before** the class object is created. Use it when you need to inspect or modify `bases` or the `namespace` dictionary (adding, wrapping, or renaming attributes/methods) before constructing the class via `super().__new__(...)`.
- `Meta.__init__(cls, name, bases, namespace)`: Runs **after** the class object `cls` already exists. Use it to register `cls`, validate class structure, or mutate attributes on `cls`.

```python
class RegistryMeta(type):
    registry: dict[str, type] = {}

    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict):
        # Can transform namespace before class creation
        if "plugin_name" in namespace:
            namespace["plugin_name"] = namespace["plugin_name"].lower()
        return super().__new__(mcs, name, bases, namespace)

    def __init__(cls, name: str, bases: tuple[type, ...], namespace: dict) -> None:
        super().__init__(name, bases, namespace)
        # Register subclasses (skip base class if desired)
        if bases:
            RegistryMeta.registry[cls.__name__] = cls


class BasePlugin(metaclass=RegistryMeta):
    pass


class MarkdownPlugin(BasePlugin):
    plugin_name = "MARKDOWN"


print(MarkdownPlugin.plugin_name)  # markdown
print(RegistryMeta.registry)       # {'MarkdownPlugin': <class '...MarkdownPlugin'>}
```

> **Modern Alternative:** For simple subclass registration or validation without full metaclass machinery, prefer `__init_subclass__(cls, **kwargs)` on a regular base class, which avoids metaclass conflicts.

---

## 15. Descriptors (`__get__`, `__set__`, `__delete__`)

A **descriptor** is any object defining at least one of `__get__`, `__set__`, or `__delete__`. Descriptors power Python's `@property`, `classmethod`, `staticmethod`, and bound methods.

### Data vs Non-Data Descriptors
- **Data descriptor**: Defines `__set__` or `__delete__` (alongside `__get__`). Has **higher precedence** than the instance's `__dict__`.
- **Non-data descriptor**: Defines only `__get__`. Has **lower precedence** than the instance's `__dict__` (e.g., regular methods).

### Lookup Precedence Order
1. Data descriptors on the class (and bases)
2. Instance attributes (`obj.__dict__`)
3. Non-data descriptors & class attributes

### Descriptor Methods
- `__get__(self, instance, owner=None)`: Called on attribute read (`instance` is `None` when accessed via the class).
- `__set__(self, instance, value)`: Called on attribute assignment.
- `__delete__(self, instance)`: Called on `del obj.attr`.
- `__set_name__(self, owner, name)` *(Python 3.6+)*: Automatically called when the owning class is created so the descriptor learns its attribute name.

```python
class PositiveNumber:
    def __set_name__(self, owner: type, name: str) -> None:
        self.private_name = f"_{name}"

    def __get__(self, instance, owner=None):
        if instance is None:
            return self
        return getattr(instance, self.private_name)

    def __set__(self, instance, value: float) -> None:
        if not isinstance(value, (int, float)) or value <= 0:
            raise ValueError(f"{self.private_name[1:]} must be a positive number")
        setattr(instance, self.private_name, value)

    def __delete__(self, instance) -> None:
        raise AttributeError("Cannot delete a required numeric attribute")


class Product:
    price = PositiveNumber()
    weight = PositiveNumber()

    def __init__(self, price: float, weight: float) -> None:
        self.price = price
        self.weight = weight


item = Product(19.99, 0.5)
print(item.price, item.weight)  # 19.99 0.5
# item.price = -5               # ValueError: price must be a positive number
```

---

## 16. `__slots__`

By default, Python stores per-instance attributes in a dynamic `dict` (`__dict__`), which has significant memory overhead when creating millions of small instances. Defining `__slots__` replaces `__dict__` with a compact, fixed C-level array of attribute slots.

### Benefits & Caveats
- **Lower memory usage** and slightly faster attribute access.
- **Prevents accidental dynamic attributes**: Assigning an unlisted attribute raises `AttributeError`.
- **Inheritance caveat**: If a subclass omits `__slots__`, it gets a `__dict__` anyway. If a subclass defines `__slots__`, it should only list *new* attributes not already in parent `__slots__`.
- **Weakrefs**: Instances won't support `weakref` unless `'__weakref__'` is included in `__slots__`.

```python
class Point3D:
    __slots__ = ("x", "y", "z")

    def __init__(self, x: float, y: float, z: float) -> None:
        self.x = x
        self.y = y
        self.z = z


p = Point3D(1.0, 2.0, 3.0)
print(p.x, p.y, p.z)       # 1.0 2.0 3.0
print(hasattr(p, "__dict__"))  # False

try:
    p.color = "red"
except AttributeError as exc:
    print(exc)  # 'Point3D' object has no attribute 'color'
```

---

## 17. Dataclasses (`@dataclass`)

Introduced in Python 3.7 (`dataclasses` module), `@dataclass` automatically generates boilerplate special methods (`__init__`, `__repr__`, `__eq__`, and optionally ordering/hashing/slots) from class-level type annotations.

### Key Options & Helpers
- `frozen=True`: Makes instances immutable and generates `__hash__`.
- `order=True`: Generates `__lt__`, `__le__`, `__gt__`, `__ge__` based on field order.
- `slots=True` *(Python 3.10+)*: Generates `__slots__` automatically.
- `kw_only=True` *(Python 3.10+)*: Makes fields keyword-only in `__init__`.
- `field(default_factory=list)`: **Required** for mutable defaults (lists, dicts, sets) so instances don't share one object.
- `__post_init__(self)`: Hook called immediately after `__init__` for validation or derived attributes.
- Helpers: `asdict()`, `astuple()`, `replace()`.

```python
from dataclasses import dataclass, field, asdict, replace


@dataclass(frozen=True, order=True, slots=True)
class Task:
    priority: int
    title: str
    tags: tuple[str, ...] = field(default=(), compare=False)
    metadata: dict[str, str] = field(default_factory=dict, compare=False)

    def __post_init__(self) -> None:
        if self.priority < 1:
            raise ValueError("priority must be >= 1")


t1 = Task(1, "Fix bug", tags=("urgent",))
t2 = replace(t1, priority=2)

print(t1)          # Task(priority=1, title='Fix bug', tags=('urgent',), metadata={})
print(t1 < t2)     # True
print(asdict(t1))  # {'priority': 1, 'title': 'Fix bug', 'tags': ('urgent',), 'metadata': {}}
```

---

## 18. Enums (`enum` Module)

The `enum` module provides symbolic, type-safe enumerations of named constant values.

### Core Classes & Features
- **`Enum`**: General-purpose enumeration; members compare by identity (`is`) or equality (`==`), not against raw ints/strings.
- **`IntEnum`**: Subclass of `int` and `Enum`; members compare equal to integers (useful for legacy APIs/protocols).
- **`StrEnum`** *(Python 3.11+)*: Subclass of `str` and `Enum`; members behave as strings.
- **`Flag` / `IntFlag`**: Supports bitwise combinations (`|`, `&`, `~`, `^`).
- **`auto()`**: Automatically assigns incrementing values (or lowercase member name in `StrEnum`).
- **`@unique`**: Class decorator that raises `ValueError` if two members share the same value (aliases are otherwise allowed).

```python
from enum import Enum, Flag, StrEnum, auto, unique


@unique
class HttpStatus(Enum):
    OK = 200
    CREATED = 201
    NOT_FOUND = 404

    def is_success(self) -> bool:
        return 200 <= self.value < 300


class Role(StrEnum):
    ADMIN = auto()   # 'admin'
    EDITOR = auto()  # 'editor'
    VIEWER = auto()  # 'viewer'


class Permission(Flag):
    NONE = 0
    READ = auto()    # 1
    WRITE = auto()   # 2
    EXECUTE = auto() # 4
    RW = READ | WRITE


status = HttpStatus.OK
print(status.name, status.value)     # OK 200
print(status.is_success())           # True
print(HttpStatus(404))               # HttpStatus.NOT_FOUND
print(HttpStatus["CREATED"])         # HttpStatus.CREATED
print(Role.ADMIN == "admin")         # True (StrEnum)

perms = Permission.READ | Permission.WRITE
print(Permission.READ in perms)      # True
print(perms == Permission.RW)        # True
```
