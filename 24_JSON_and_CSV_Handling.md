# JSON and CSV Handling in Python

Modern Python applications frequently exchange, store, and persist structured data across files, APIs, and configuration systems. Python provides built-in standard library modules like `json`, `csv`, and `pickle`, alongside widely used third-party packages like `openpyxl` and `PyYAML`, to serialize, deserialize, and process structured data cleanly.

---

## Table of Contents

1. [JSON Overview and Python `json` Module](#1-json-overview-and-python-json-module)
2. [`json.dumps()` — Python to JSON String](#2-jsondumps--python-to-json-string)
3. [`json.loads()` — JSON String to Python](#3-jsonloads--json-string-to-python)
4. [`json.dump()` — Write JSON to File](#4-jsondump--write-json-to-file)
5. [`json.load()` — Read JSON from File](#5-jsonload--read-json-from-file)
6. [JSON Data Types Mapping (Python ↔ JSON)](#6-json-data-types-mapping-python--json)
7. [Pretty Printing JSON (`indent`, `sort_keys`)](#7-pretty-printing-json-indent-sort_keys)
8. [Handling Custom Objects (`default` Parameter, `JSONEncoder`)](#8-handling-custom-objects-default-parameter-jsonencoder)
9. [`csv` Module Overview](#9-csv-module-overview)
10. [`csv.reader()` — Reading CSV Files](#10-csvreader--reading-csv-files)
11. [`csv.writer()` — Writing CSV Files](#11-csvwriter--writing-csv-files)
12. [`csv.DictReader()` and `csv.DictWriter()`](#12-csvdictreader-and-csvdictwriter)
13. [Handling Different Delimiters and Quoting](#13-handling-different-delimiters-and-quoting)
14. [Working with Excel Files Overview (`openpyxl`)](#14-working-with-excel-files-overview-openpyxl)
15. [`pickle` Module (Serialization / Deserialization)](#15-pickle-module-serialization--deserialization)
16. [YAML Handling Overview](#16-yaml-handling-overview)
17. [Summary & Quick Reference](#17-summary--quick-reference)

---

## 1. JSON Overview and Python `json` Module

**JSON (JavaScript Object Notation)** is a lightweight, human-readable, text-based data interchange format. Despite originating from JavaScript, JSON is language-independent and is the standard format for REST APIs, configuration files, and NoSQL document stores.

### Key Characteristics of JSON

- Data is represented as **key-value pairs** (objects) and **ordered lists** (arrays).
- Keys must always be **double-quoted strings** (`"name"`, not `'name'`).
- Strings must use **double quotes** (`"hello"`).
- Trailing commas are **not allowed**.
- Comments (`//` or `/* */`) are **not supported** in standard JSON.

### The Python `json` Module

Python includes the `json` module in its standard library—no installation is required.

```python
import json
```

### Core Functions at a Glance

| Function | Direction | Target |
| :--- | :--- | :--- |
| `json.dumps(obj)` | Python object → JSON string | String (`str`) |
| `json.loads(s)` | JSON string → Python object | In-memory string |
| `json.dump(obj, file)` | Python object → JSON file | File-like object |
| `json.load(file)` | JSON file → Python object | File-like object |

> **Memory Tip:** Functions ending in **`s`** (`dumps`, `loads`) operate on **strings**. Functions without **`s`** (`dump`, `load`) operate on **files**.

---

## 2. `json.dumps()` — Python to JSON String

`json.dumps()` (**dump string**) serializes a Python object into a JSON-formatted `str`.

### Syntax

```python
json.dumps(
    obj,
    *,
    skipkeys=False,
    ensure_ascii=True,
    allow_nan=True,
    indent=None,
    separators=None,
    default=None,
    sort_keys=False,
)
```

### Basic Example

```python
import json

user = {
    "id": 101,
    "username": "alice_dev",
    "is_active": True,
    "roles": ["admin", "editor"],
    "middle_name": None,
}

json_str = json.dumps(user)
print(type(json_str))
print(json_str)
# Output:
# <class 'str'>
# {"id": 101, "username": "alice_dev", "is_active": true, "roles": ["admin", "editor"], "middle_name": null}
```

### Useful Parameters

- **`ensure_ascii=False`**: Keeps non-ASCII characters (like emojis or accented text) readable instead of escaping them as `\uXXXX`.
- **`separators=(",", ":")`**: Removes extra whitespace for compact payloads (useful for network transmission or storage).
- **`skipkeys=True`**: Skips dictionary keys that are not basic types (`str`, `int`, `float`, `bool`, `None`) instead of raising a `TypeError`.

```python
import json

data = {"greeting": "Namaste 🙏", "city": "München"}

print(json.dumps(data))
# {"greeting": "Namaste \ud83d\ude4f", "city": "M\u00fcnchen"}

print(json.dumps(data, ensure_ascii=False))
# {"greeting": "Namaste 🙏", "city": "München"}

print(json.dumps(data, ensure_ascii=False, separators=(",", ":")))
# {"greeting":"Namaste 🙏","city":"München"}
```

---

## 3. `json.loads()` — JSON String to Python

`json.loads()` (**load string**) parses a JSON-formatted `str`, `bytes`, or `bytearray` and converts it into the corresponding Python object.

### Syntax

```python
json.loads(s, *, cls=None, object_hook=None, parse_float=None, parse_int=None)
```

### Basic Example

```python
import json

json_payload = '''
{
    "service": "auth-api",
    "version": 2.1,
    "healthy": true,
    "endpoints": ["/login", "/logout", "/refresh"],
    "deprecated": null
}
'''

config = json.loads(json_payload)
print(type(config))          # <class 'dict'>
print(config["service"])     # auth-api
print(config["healthy"])     # True
print(config["endpoints"])   # ['/login', '/logout', '/refresh']
print(config["deprecated"])  # None
```

### Handling Invalid JSON (`json.JSONDecodeError`)

Malformed JSON (such as single quotes, trailing commas, or unquoted keys) raises `json.JSONDecodeError`, which is a subclass of `ValueError`.

```python
import json

bad_json = "{'name': 'Alice',}"  # Single quotes + trailing comma (invalid JSON)

try:
    data = json.loads(bad_json)
except json.JSONDecodeError as err:
    print(f"Invalid JSON at line {err.lineno}, column {err.colno}: {err.msg}")
# Output:
# Invalid JSON at line 1, column 2: Expecting property name enclosed in double quotes
```

### Customizing Parsing (`parse_float`, `object_hook`)

You can intercept how numbers or objects are decoded:

```python
import json
from decimal import Decimal

price_json = '{"item": "Laptop", "price": 999.99}'
parsed = json.loads(price_json, parse_float=Decimal)

print(type(parsed["price"]), parsed["price"])
# Output: <class 'decimal.Decimal'> 999.99
```

---

## 4. `json.dump()` — Write JSON to File

`json.dump()` serializes a Python object and writes it directly to a file-like object (such as an open file handle).

### Syntax

```python
json.dump(obj, fp, *, indent=None, sort_keys=False, ensure_ascii=True, default=None)
```

### Example: Writing JSON to a File

Always open JSON files with `encoding="utf-8"` to ensure consistent Unicode handling across platforms.

```python
import json

employees = [
    {"id": 1, "name": "Alice", "department": "Engineering", "remote": True},
    {"id": 2, "name": "Bob", "department": "Design", "remote": False},
]

with open("employees.json", "w", encoding="utf-8") as f:
    json.dump(employees, f, indent=4, ensure_ascii=False)
```

Contents of `employees.json`:

```json
[
    {
        "id": 1,
        "name": "Alice",
        "department": "Engineering",
        "remote": true
    },
    {
        "id": 2,
        "name": "Bob",
        "department": "Design",
        "remote": false
    }
]
```

> **Note:** `json.dump(obj, f)` writes directly to the stream and returns `None`, avoiding building the entire serialized string in memory first (unlike `f.write(json.dumps(obj))`).

---

## 5. `json.load()` — Read JSON from File

`json.load()` reads a JSON-formatted file stream and deserializes it into Python objects.

### Syntax

```python
json.load(fp, *, cls=None, object_hook=None, parse_float=None)
```

### Example: Reading JSON from a File

```python
import json

try:
    with open("employees.json", "r", encoding="utf-8") as f:
        employees = json.load(f)

    for emp in employees:
        print(f"{emp['id']}: {emp['name']} ({emp['department']})")
except FileNotFoundError:
    print("Error: employees.json not found.")
except json.JSONDecodeError as err:
    print(f"Error: Failed to parse JSON ({err}).")
# Output:
# 1: Alice (Engineering)
# 2: Bob (Design)
```

### Real-World Pattern: Read, Update, Write Back

```python
import json
from pathlib import Path

config_path = Path("settings.json")

# Create default file if missing
if not config_path.exists():
    with config_path.open("w", encoding="utf-8") as f:
        json.dump({"theme": "light", "autosave": True}, f, indent=4)

# Read, modify, and save
with config_path.open("r", encoding="utf-8") as f:
    settings = json.load(f)

settings["theme"] = "dark"
settings["font_size"] = 14

with config_path.open("w", encoding="utf-8") as f:
    json.dump(settings, f, indent=4)
```

---

## 6. JSON Data Types Mapping (Python ↔ JSON)

Because JSON supports fewer data types than Python, Python's `json` module converts values according to a strict mapping table.

### Python → JSON (Serialization / Encoding)

| Python Type | JSON Type | Example |
| :--- | :--- | :--- |
| `dict` | `object` | `{"a": 1}` → `{"a": 1}` |
| `list`, `tuple` | `array` | `(1, 2)` → `[1, 2]` |
| `str` | `string` | `"hi"` → `"hi"` |
| `int`, `float` | `number` | `42`, `3.14` → `42`, `3.14` |
| `True` | `true` | `True` → `true` |
| `False` | `false` | `False` → `false` |
| `None` | `null` | `None` → `null` |

### JSON → Python (Deserialization / Decoding)

| JSON Type | Python Type |
| :--- | :--- |
| `object` | `dict` |
| `array` | `list` |
| `string` | `str` |
| `number (int)` | `int` |
| `number (real)` | `float` |
| `true` | `True` |
| `false` | `False` |
| `null` | `None` |

### Important Caveats & Lossy Conversions

1. **Tuples become Lists:**
   A Python `tuple` serializes as a JSON array and deserializes back as a `list`.
   ```python
   import json
   original = (1, 2, 3)
   restored = json.loads(json.dumps(original))
   print(restored, type(restored))  # [1, 2, 3] <class 'list'>
   ```
2. **Dictionary Keys Become Strings:**
   JSON object keys must be strings. Integer or boolean dictionary keys are converted to strings during serialization.
   ```python
   import json
   d = {1: "one", 2: "two"}
   restored = json.loads(json.dumps(d))
   print(restored)  # {'1': 'one', '2': 'two'}
   ```
3. **Unsupported Types:**
   Types like `set`, `bytes`, `datetime`, `Decimal`, `complex`, and custom class instances raise `TypeError: Object of type ... is not JSON serializable` unless handled explicitly.

---

## 7. Pretty Printing JSON (`indent`, `sort_keys`)

Raw JSON strings are compact and hard for humans to read. Both `json.dumps()` and `json.dump()` provide formatting parameters for readable output.

### `indent` and `sort_keys`

- **`indent`**:
  - An integer (e.g., `indent=2` or `indent=4`) indents each level by that many spaces.
  - A string (e.g., `indent="\t"`) indents using that string.
  - `None` (default) produces single-line compact output.
- **`sort_keys`**:
  - When `True`, sorts dictionary keys alphabetically at every nesting level (useful for deterministic diffs in Git or tests).

```python
import json

server_info = {
    "port": 8080,
    "host": "localhost",
    "features": {"logging": True, "auth": True, "cache": False},
    "tags": ["prod", "v2"],
}

pretty_json = json.dumps(server_info, indent=4, sort_keys=True)
print(pretty_json)
```

Output:

```json
{
    "features": {
        "auth": true,
        "cache": false,
        "logging": true
    },
    "host": "localhost",
    "port": 8080,
    "tags": [
        "prod",
        "v2"
    ]
}
```

### Pretty Printing from the Command Line

Python's `json.tool` module can validate and pretty-print JSON files directly in a terminal:

```bash
python -m json.tool messy.json pretty.json --indent 2 --sort-keys
```

---

## 8. Handling Custom Objects (`default` Parameter, `JSONEncoder`)

When serializing objects like `datetime`, `set`, `Path`, `dataclass`, or custom classes, `json.dumps()` raises a `TypeError` by default. You can customize serialization using either the `default` function parameter or a `json.JSONEncoder` subclass.

### Method 1: Using the `default` Parameter

The `default` callback is invoked **only** when the encoder encounters an object it does not know how to serialize. It should return a JSON-serializable representation or raise `TypeError`.

```python
import json
from dataclasses import asdict, dataclass
from datetime import datetime
from pathlib import Path


@dataclass
class Task:
    title: str
    tags: set[str]
    created_at: datetime
    log_path: Path


def custom_serializer(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    if isinstance(obj, set):
        return sorted(obj)
    if isinstance(obj, Path):
        return str(obj)
    if hasattr(obj, "__dataclass_fields__"):
        return asdict(obj)
    raise TypeError(f"Object of type {type(obj).__name__} is not JSON serializable")


task = Task(
    title="Deploy API",
    tags={"backend", "urgent"},
    created_at=datetime(2026, 11, 26, 10, 30),
    log_path=Path("/var/log/deploy.log"),
)

print(json.dumps(task, default=custom_serializer, indent=2))
```

Output:

```json
{
  "title": "Deploy API",
  "tags": [
    "backend",
    "urgent"
  ],
  "created_at": "2026-11-26T10:30:00",
  "log_path": "/var/log/deploy.log"
}
```

### Method 2: Subclassing `json.JSONEncoder`

For reusable serialization logic across an application, subclass `json.JSONEncoder` and override its `default()` method.

```python
import json
from datetime import date, datetime


class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, (datetime, date)):
            return obj.isoformat()
        if isinstance(obj, set):
            return list(obj)
        if isinstance(obj, complex):
            return {"real": obj.real, "imag": obj.imag}
        return super().default(obj)  # Raises TypeError for unknown types


payload = {
    "event_date": date(2026, 11, 26),
    "scopes": {"read", "write"},
    "signal": 3 + 4j,
}

print(json.dumps(payload, cls=CustomEncoder, indent=2))
```

### Bonus: Reconstructing Custom Objects with `object_hook`

To turn JSON dictionaries back into custom Python objects during `json.loads()` / `json.load()`, pass an `object_hook` function.

```python
import json


class User:
    def __init__(self, name, role):
        self.name = name
        self.role = role

    def __repr__(self):
        return f"User(name={self.name!r}, role={self.role!r})"


def decode_user(dct):
    if "__user__" in dct:
        return User(dct["name"], dct["role"])
    return dct


raw = '{"__user__": true, "name": "Alice", "role": "admin"}'
user = json.loads(raw, object_hook=decode_user)
print(user, type(user))
# Output: User(name='Alice', role='admin') <class '__main__.User'>
```

---

## 9. `csv` Module Overview

**CSV (Comma-Separated Values)** is a plain-text tabular format where each line represents a row and columns are separated by a delimiter (usually a comma `,`).

### Why Use Python's `csv` Module Instead of `str.split(",")`?

Naive splitting breaks quickly on real-world CSV files:
- Fields containing commas inside quotes: `"Smith, John",35,"New York, NY"`
- Fields containing embedded newlines across multiple lines
- Escaped quotes inside quoted fields: `"He said ""Hello"""`
- Varying line endings (`\r\n` vs `\n`)

Python's built-in `csv` module handles quoting, escaping, dialects, and line endings properly according to RFC 4180 conventions.

```python
import csv
```

### Critical Rule: Always Use `newline=""`

When opening CSV files in Python, **always** pass `newline=""` to `open()`:

```python
with open("data.csv", "r", encoding="utf-8", newline="") as f:
    reader = csv.reader(f)
```

Why? The `csv` module performs its own universal newline translation. Without `newline=""`:
- On Windows, `csv.writer()` writes `\r\r\n`, producing blank lines between every row.
- Multi-line quoted fields may fail to parse correctly.

---

## 10. `csv.reader()` — Reading CSV Files

`csv.reader()` reads rows from a CSV file and yields each row as a **`list` of strings**.

### Syntax

```python
csv.reader(csvfile, dialect="excel", **fmtparams)
```

### Example: Reading a CSV File

Suppose `students.csv` contains:

```csv
id,name,grade,gpa
1,Alice,A,3.9
2,Bob,B+,3.5
3,"Charlie, Jr.",A-,3.7
```

```python
import csv

with open("students.csv", "r", encoding="utf-8", newline="") as f:
    reader = csv.reader(f)
    header = next(reader)  # Read the first row (column names)
    print(f"Columns: {header}")

    for row in reader:
        student_id, name, grade, gpa = row
        # Note: All values from csv.reader are strings; convert numbers manually
        print(f"#{int(student_id)} {name} -> Grade: {grade}, GPA: {float(gpa)}")
```

Output:

```text
Columns: ['id', 'name', 'grade', 'gpa']
#1 Alice -> Grade: A, GPA: 3.9
#2 Bob -> Grade: B+, GPA: 3.5
#3 Charlie, Jr. -> Grade: A-, GPA: 3.7
```

> **Tip:** `reader.line_num` tracks the current physical line number read from the source file, which is helpful for reporting errors in malformed CSVs.

---

## 11. `csv.writer()` — Writing CSV Files

`csv.writer()` writes sequences (lists or tuples) as formatted CSV rows to a file-like object.

### Key Methods

- `writer.writerow(row)`: Writes a single row (an iterable of values).
- `writer.writerows(rows)`: Writes multiple rows at once (an iterable of iterables).

### Example: Writing Rows to a CSV File

```python
import csv

headers = ["id", "product", "price", "in_stock"]
rows = [
    [101, "Mechanical Keyboard", 89.99, True],
    [102, 'Monitor 27" 4K', 349.50, True],
    [103, "USB-C Hub, 7-in-1", 29.99, False],
]

with open("inventory.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(headers)
    writer.writerows(rows)
```

Resulting `inventory.csv`:

```csv
id,product,price,in_stock
101,Mechanical Keyboard,89.99,True
102,"Monitor 27"" 4K",349.5,True
103,"USB-C Hub, 7-in-1",29.99,False
```

Notice how `csv.writer()` automatically quotes `"USB-C Hub, 7-in-1"` (contains a comma) and escapes the double quote in `'Monitor 27" 4K'` as `""`.

---

## 12. `csv.DictReader()` and `csv.DictWriter()`

Working with column indices (`row[0]`, `row[2]`) is brittle if column order changes. `DictReader` and `DictWriter` map each row to a Python `dict` using column names.

### `csv.DictReader()`

By default, `DictReader` uses the first row as dictionary keys.

```python
import csv

with open("inventory.csv", "r", encoding="utf-8", newline="") as f:
    reader = csv.DictReader(f)
    print("Fieldnames:", reader.fieldnames)

    for row in reader:
        print(f"{row['product']}: ${float(row['price']):.2f} (In stock: {row['in_stock']})")
```

Output:

```text
Fieldnames: ['id', 'product', 'price', 'in_stock']
Mechanical Keyboard: $89.99 (In stock: True)
Monitor 27" 4K: $349.50 (In stock: True)
USB-C Hub, 7-in-1: $29.99 (In stock: False)
```

- If a CSV file has **no header row**, pass `fieldnames=["id", "product", "price", "in_stock"]` to `DictReader`.
- Missing fields in short rows default to `restval` (`None` by default); extra columns go into `row[None]` (configurable via `restkey`).

### `csv.DictWriter()`

`DictWriter` requires the `fieldnames` parameter so it knows the column order. You must explicitly call `writeheader()` to write the header row.

```python
import csv

fieldnames = ["name", "email", "role"]
users = [
    {"name": "Alice", "email": "alice@example.com", "role": "admin"},
    {"name": "Bob", "email": "bob@example.com"},  # 'role' omitted
]

with open("users.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.DictWriter(
        f,
        fieldnames=fieldnames,
        restval="member",       # Default value if a key is missing
        extrasaction="ignore",  # Ignore extra keys in dicts instead of raising ValueError
    )
    writer.writeheader()
    writer.writerows(users)
```

Resulting `users.csv`:

```csv
name,email,role
Alice,alice@example.com,admin
Bob,bob@example.com,member
```

---

## 13. Handling Different Delimiters and Quoting

CSV files in the wild often use tabs (`\t` for TSV), semicolons (`;` in European locales), or pipes (`|`), and vary in how values are quoted.

### Common Formatting Parameters

| Parameter | Default | Description |
| :--- | :--- | :--- |
| `delimiter` | `','` | Character separating fields (e.g., `'\t'`, `';'`, `'|'`) |
| `quotechar` | `'"'` | Character used to quote fields containing special characters |
| `quoting` | `csv.QUOTE_MINIMAL` | Controls when quotes are generated/recognized |
| `doublequote` | `True` | If `True`, `quotechar` inside a field is doubled (`""`); if `False`, `escapechar` is used |
| `escapechar` | `None` | Single character used to escape delimiter/quotechar when `quoting=csv.QUOTE_NONE` or `doublequote=False` |
| `skipinitialspace` | `False` | Ignores whitespace immediately following a delimiter |

### Quoting Modes

| Constant | Behavior |
| :--- | :--- |
| `csv.QUOTE_MINIMAL` | Quote only fields containing `delimiter`, `quotechar`, or newlines (default) |
| `csv.QUOTE_ALL` | Quote **every** field |
| `csv.QUOTE_NONNUMERIC` | Writer quotes all non-numeric fields; Reader converts unquoted fields to `float` |
| `csv.QUOTE_NONE` | Never quote fields; requires `escapechar` when writing delimiters inside data |

### Example: Tab-Separated (TSV) and Semicolon Files

```python
import csv

# Writing a TSV file with all fields quoted
data = [
    ["id", "title", "score"],
    [1, "Python Basics", 95],
    [2, "Data\tHandling", 88],
]

with open("scores.tsv", "w", encoding="utf-8", newline="") as f:
    writer = csv.writer(f, delimiter="\t", quoting=csv.QUOTE_ALL)
    writer.writerows(data)

# Reading a semicolon-delimited file with leading spaces after delimiters
with open("european.csv", "w", encoding="utf-8", newline="") as f:
    f.write("item; price; qty\nBook; 12.50; 3\n")

with open("european.csv", "r", encoding="utf-8", newline="") as f:
    reader = csv.DictReader(f, delimiter=";", skipinitialspace=True)
    for row in reader:
        print(row)
# Output: {'item': 'Book', 'price': '12.50', 'qty': '3'}
```

### Detecting Format Automatically with `csv.Sniffer`

When the delimiter or quoting style of an incoming CSV is unknown, `csv.Sniffer` can inspect a sample:

```python
import csv

with open("european.csv", "r", encoding="utf-8", newline="") as f:
    sample = f.read(1024)
    f.seek(0)
    dialect = csv.Sniffer().sniff(sample, delimiters=",;\t|")
    reader = csv.reader(f, dialect)
    for row in reader:
        print(row)
```

---

## 14. Working with Excel Files Overview (`openpyxl`)

While CSV only stores plain-text tabular data, Excel `.xlsx` files support multiple worksheets, formulas, cell formatting, charts, and native data types. Python's standard library does not read `.xlsx` natively; **`openpyxl`** is the most widely used third-party library for `.xlsx` files.

### Installation

```bash
pip install openpyxl
```

### Creating and Writing an Excel Workbook

```python
from openpyxl import Workbook
from openpyxl.styles import Font

wb = Workbook()
ws = wb.active
ws.title = "Sales"

# Header row with bold styling
ws.append(["Product", "Units", "Unit Price", "Total"])
for cell in ws[1]:
    cell.font = Font(bold=True)

# Data rows (including an Excel formula)
ws.append(["Keyboard", 10, 80, "=B2*C2"])
ws.append(["Mouse", 25, 25, "=B3*C3"])

# Create a second sheet
summary = wb.create_sheet(title="Summary")
summary["A1"] = "Report Generated"

wb.save("sales_report.xlsx")
```

### Reading an Excel Workbook

```python
from openpyxl import load_workbook

# data_only=True reads evaluated formula values (if cached by Excel) instead of formula strings
wb = load_workbook("sales_report.xlsx", data_only=False)
print("Sheets:", wb.sheetnames)

ws = wb["Sales"]

# values_only=True yields cell values directly instead of Cell objects
for row in ws.iter_rows(min_row=1, values_only=True):
    print(row)
# Output:
# ('Product', 'Units', 'Unit Price', 'Total')
# ('Keyboard', 10, 80, '=B2*C2')
# ('Mouse', 25, 25, '=B3*C3')
```

> **Note:** For large data analysis pipelines, `pandas.read_excel()` (which uses `openpyxl` under the hood) is often used to load sheets directly into DataFrames.

---

## 15. `pickle` Module (Serialization / Deserialization)

Python's built-in `pickle` module converts almost any Python object hierarchy into a **binary byte stream** (`bytes`) and reconstructs it back into live Python objects.

Unlike JSON, `pickle` is **Python-specific** and binary, but it preserves complex Python types natively (custom class instances, `set`, `tuple`, functions, `datetime`, nested objects, and shared/circular references).

### Core Functions

| Function | Direction | Target |
| :--- | :--- | :--- |
| `pickle.dumps(obj)` | Python object → `bytes` | In-memory bytes |
| `pickle.loads(data)` | `bytes` → Python object | In-memory bytes |
| `pickle.dump(obj, file)` | Python object → Binary file | File opened with `"wb"` |
| `pickle.load(file)` | Binary file → Python object | File opened with `"rb"` |

### Example: In-Memory and File Pickling

```python
import pickle
from datetime import datetime

session_state = {
    "user": "alice",
    "permissions": {"read", "write"},   # set (not supported by JSON)
    "coords": (37.77, -122.41),         # tuple preserved as tuple
    "login_time": datetime.now(),       # datetime preserved
}

# Serialize to bytes
blob = pickle.dumps(session_state, protocol=pickle.HIGHEST_PROTOCOL)
restored_mem = pickle.loads(blob)
print(type(restored_mem["permissions"]), type(restored_mem["coords"]))
# Output: <class 'set'> <class 'tuple'>

# Serialize to a binary file (note 'wb' and 'rb' modes!)
with open("state.pkl", "wb") as f:
    pickle.dump(session_state, f)

with open("state.pkl", "rb") as f:
    restored_file = pickle.load(f)
print(restored_file["user"], restored_file["login_time"])
```

> **CRITICAL SECURITY WARNING:** The `pickle` module is **not secure** against erroneous or maliciously constructed data. Unpickling untrusted data can execute arbitrary Python code. **Never** call `pickle.load()` or `pickle.loads()` on files or payloads from untrusted sources (users, external APIs, public uploads). Use `json` for untrusted data exchange.

---

## 16. YAML Handling Overview

**YAML (YAML Ain't Markup Language)** is a human-friendly, indentation-based data serialization format widely used for configuration files (Docker Compose, Kubernetes, GitHub Actions, Ansible).

### Key Features of YAML

- Uses indentation (spaces, **never tabs**) instead of braces `{}`.
- Supports comments (`#`).
- Supports anchors/aliases for reusing values, multi-line strings, dates, and booleans.
- Valid JSON is also valid YAML (YAML is a superset of JSON).

Python does not include YAML support in the standard library; the standard third-party package is **`PyYAML`** (`yaml`).

### Installation

```bash
pip install pyyaml
```

### Reading and Writing YAML (`safe_load` & `safe_dump`)

```python
import yaml

config = {
    "app": {
        "name": "billing-service",
        "port": 8000,
        "debug": False,
    },
    "databases": ["postgres", "redis"],
}

# Python dict -> YAML string
yaml_text = yaml.safe_dump(config, sort_keys=False, default_flow_style=False)
print(yaml_text)
# Output:
# app:
#   name: billing-service
#   port: 8000
#   debug: false
# databases:
# - postgres
# - redis

# Write YAML to file
with open("config.yaml", "w", encoding="utf-8") as f:
    yaml.safe_dump(config, f, sort_keys=False)

# Read YAML from file
with open("config.yaml", "r", encoding="utf-8") as f:
    loaded_config = yaml.safe_load(f)

print(loaded_config["app"]["name"])  # billing-service
```

> **Security Best Practice:** Always use `yaml.safe_load()` instead of `yaml.load()` when parsing YAML. `yaml.load()` with an unsafe loader can construct arbitrary Python objects and execute code from crafted YAML input.

---

## 17. Summary & Quick Reference

### Choosing the Right Format

| Format | Module / Package | Human-Readable | Cross-Language | Best Use Cases |
| :--- | :--- | :--- | :--- | :--- |
| **JSON** | `json` (built-in) | Yes | Yes | Web APIs, structured payloads, app settings |
| **CSV** | `csv` (built-in) | Yes | Yes | Flat tabular data, data export/import |
| **Excel (`.xlsx`)** | `openpyxl` (3rd-party) | Via spreadsheet app | Yes | Multi-sheet reports, formulas, formatted workbooks |
| **Pickle** | `pickle` (built-in) | No (binary) | No (Python only) | Internal Python caching, ML objects (trusted data only) |
| **YAML** | `PyYAML` (`yaml`) | Very | Yes | DevOps configs, CI/CD pipelines, human-edited configs |

### Common Pitfalls Checklist

- **JSON:**
  - Use `dumps`/`loads` for strings, `dump`/`load` for files.
  - Use `default=` or a `JSONEncoder` subclass for `datetime`, `set`, `Path`, or custom objects.
  - Remember `tuple` becomes `list` and non-string dict keys become `str`.
- **CSV:**
  - Always open CSV files with `newline=""` and `encoding="utf-8"`.
  - Remember all values read by `csv.reader` / `csv.DictReader` are strings—cast numbers/booleans explicitly.
- **Pickle & YAML:**
  - Never unpickle untrusted data.
  - Always use `yaml.safe_load()` for YAML input.
