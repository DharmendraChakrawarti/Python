# File Handling in Python

File handling allows Python programs to read, write, modify, and manage data stored persistently on disk. Python treats files as streams of text (decoded characters) or raw bytes, providing built-in functions alongside standard-library modules (`os`, `pathlib`, `shutil`, `csv`, `json`, `xml.etree.ElementTree`, `tempfile`) for structured data and filesystem operations.

---

## Table of Contents

1. [Opening Files (`open()` and File Modes)](#1-opening-files-open-and-file-modes)
2. [Reading Files (`read`, `readline`, `readlines`)](#2-reading-files-read-readline-readlines)
3. [Writing Files (`write`, `writelines`)](#3-writing-files-write-writelines)
4. [Appending to Files](#4-appending-to-files)
5. [The `with` Statement (Context Manager)](#5-the-with-statement-context-manager)
6. [File Pointer and `seek()` / `tell()`](#6-file-pointer-and-seek--tell)
7. [Binary File Operations](#7-binary-file-operations)
8. [Working with CSV Files (`csv` Module)](#8-working-with-csv-files-csv-module)
9. [Working with JSON Files (`json` Module)](#9-working-with-json-files-json-module)
10. [Working with XML Files](#10-working-with-xml-files)
11. [`os` Module for File Operations](#11-os-module-for-file-operations)
12. [`pathlib` Module (Modern Path Handling)](#12-pathlib-module-modern-path-handling)
13. [`shutil` Module (High-Level File Operations)](#13-shutil-module-high-level-file-operations)
14. [`tempfile` Module (Temporary Files & Directories)](#14-tempfile-module-temporary-files--directories)
15. [Reading and Writing Large Files Efficiently](#15-reading-and-writing-large-files-efficiently)

---

## 1. Opening Files (`open()` and File Modes)

Python's built-in `open()` function opens a file and returns a **file object** (stream).

### Syntax

```python
file_obj = open(
    file,
    mode="r",
    buffering=-1,
    encoding=None,
    errors=None,
    newline=None,
)
```

- `file`: Path to the file (string, `bytes`, or `os.PathLike` such as `pathlib.Path`).
- `mode`: String specifying how the file is opened (defaults to `"r"`).
- `encoding`: Character encoding in text mode (always specify `encoding="utf-8"` explicitly for cross-platform consistency, since platform defaults vary).
- `errors`: How encoding/decoding errors are handled (e.g., `"strict"`, `"ignore"`, `"replace"`).
- `newline`: Controls universal newline translation (`None`, `""`, `"\n"`, `"\r"`, `"\r\n"`).

### File Modes

You combine a **primary access mode** (`r`, `w`, `a`, `x`) with optional **type** (`t`, `b`) and **read/write modifier** (`+`):

| Mode | Meaning | File Must Exist? | Truncates Existing? | Initial Pointer Position |
| :--- | :--- | :--- | :--- | :--- |
| `r` | Read (default) | Yes (`FileNotFoundError`) | No | Beginning |
| `w` | Write | No (creates it) | **Yes (erases content)** | Beginning |
| `a` | Append | No (creates it) | No | End |
| `x` | Exclusive creation | **Must NOT exist** (`FileExistsError`) | N/A | Beginning |
| `t` | Text mode (default) | — | — | — |
| `b` | Binary mode | — | — | — |
| `+` | Read and write (updating) | Depends on `r`/`w`/`a`/`x` | Depends on `r`/`w`/`a`/`x` | Depends on `r`/`w`/`a`/`x` |

### Common Mode Combinations

- `"rt"` / `"r"`: Read text (default).
- `"wt"` / `"w"`: Write text (overwrites existing file or creates new).
- `"at"` / `"a"`: Append text.
- `"xt"` / `"x"`: Create new text file; fail if it already exists.
- `"rb"` / `"wb"` / `"ab"` / `"xb"`: Binary equivalents (read/write `bytes`, no `encoding` argument allowed).
- `"r+"`: Read and write without truncating (fails if file doesn't exist; pointer starts at 0).
- `"w+"`: Write and read (truncates file to zero length or creates new).
- `"a+"`: Append and read (writes always go to end; initial pointer is at end).

### Manual Opening and Closing

If you open a file without `with`, you must call `.close()` (typically inside `try...finally`) to flush buffers and release OS file descriptors:

```python
f = open("example.txt", mode="r", encoding="utf-8")
try:
    content = f.read()
    print(content)
finally:
    f.close()
```

---

## 2. Reading Files (`read`, `readline`, `readlines`)

Suppose `sample.txt` contains:

```text
First line
Second line
Third line
```

### `read(size=-1)`

Reads up to `size` characters (in text mode) or `size` bytes (in binary mode). With no argument or `-1`, reads the entire remaining file into a single string.

```python
with open("sample.txt", "r", encoding="utf-8") as f:
    full_text = f.read()
    print(full_text)

with open("sample.txt", "r", encoding="utf-8") as f:
    first_five = f.read(5)   # "First"
    next_five = f.read(5)    # " line"
    print(repr(first_five), repr(next_five))
```

### `readline(size=-1)`

Reads a single line up to and including the newline character `"\n"`. Returns `""` (empty string) at EOF (end of file).

```python
with open("sample.txt", "r", encoding="utf-8") as f:
    line1 = f.readline()  # "First line\n"
    line2 = f.readline()  # "Second line\n"
    print(line1.strip(), "|", line2.strip())
```

### `readlines(hint=-1)`

Reads all remaining lines into a **list of strings**, preserving trailing newlines.

```python
with open("sample.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()
    print(lines)
    # ['First line\n', 'Second line\n', 'Third line\n']
```

### Direct Iteration (Recommended for Line-by-Line Reading)

File objects are iterators that yield one line at a time without loading the whole file into memory:

```python
with open("sample.txt", "r", encoding="utf-8") as f:
    for line_num, line in enumerate(f, start=1):
        print(f"{line_num}: {line.rstrip()}")
```

---

## 3. Writing Files (`write`, `writelines`)

Opening with `"w"` creates the file if missing, or **truncates it to 0 bytes** if it already exists.

### `write(text)`

Writes a single string and returns the number of characters written. Unlike `print()`, `write()` does **not** add a newline automatically.

```python
with open("output.txt", "w", encoding="utf-8") as f:
    chars1 = f.write("Hello, Python!\n")
    chars2 = f.write("File handling is straightforward.\n")
    print(f"Wrote {chars1 + chars2} characters.")
```

### `writelines(lines)`

Writes an iterable of strings sequentially. It does **not** insert line separators between items—you must include `"\n"` yourself.

```python
rows = ["Alpha\n", "Beta\n", "Gamma\n"]

with open("greek.txt", "w", encoding="utf-8") as f:
    f.writelines(rows)
```

### Exclusive Creation (`"x"`)

Prevents accidentally overwriting an existing file:

```python
try:
    with open("config.txt", "x", encoding="utf-8") as f:
        f.write("default_timeout=30\n")
except FileExistsError:
    print("config.txt already exists; refusing to overwrite.")
```

### Using `print(..., file=f)`

You can also redirect `print()` output to a writable text file:

```python
with open("log.txt", "w", encoding="utf-8") as f:
    print("Status:", "OK", sep=" ", file=f)
```

---

## 4. Appending to Files

Opening a file with `"a"` preserves existing contents and writes new data at the end.

```python
# Initial write
with open("app.log", "w", encoding="utf-8") as f:
    f.write("[INFO] Application started\n")

# Append later
with open("app.log", "a", encoding="utf-8") as f:
    f.write("[INFO] User logged in\n")
    f.write("[WARN] Disk usage at 80%\n")
```

### Append + Read (`"a+"`)

With `"a+"`, the initial file pointer starts at the **end** of the file. To read existing data first, move the pointer to the beginning with `seek(0)`. On POSIX and Windows, writes in `"a"` / `"a+"` mode always append to the end of the file regardless of where `seek()` placed the pointer.

```python
with open("app.log", "a+", encoding="utf-8") as f:
    f.write("[INFO] Request completed\n")
    f.seek(0)
    print(f.read())
```

---

## 5. The `with` Statement (Context Manager)

The `with` statement wraps file operations in a **context manager**. When execution leaves the `with` block—normally or due to an exception—Python automatically calls the file object's `__exit__()` method, flushing and closing the file.

### Why Use `with`?

- Prevents resource/file-descriptor leaks.
- Ensures buffered data is flushed to disk.
- Eliminates boilerplate `try...finally` blocks.

### Opening Multiple Files in One `with` Statement

Python 3.10+ supports parenthesized multi-line context managers:

```python
with (
    open("source.txt", "r", encoding="utf-8") as src,
    open("uppercase.txt", "w", encoding="utf-8") as dst,
):
    for line in src:
        dst.write(line.upper())
```

---

## 6. File Pointer and `seek()` / `tell()`

Python maintains a **file pointer** (cursor) tracking the current read/write offset in the stream.

- `f.tell()` returns the current position (an opaque cookie in text mode, or an exact byte offset from the start in binary mode).
- `f.seek(offset, whence=0)` moves the pointer to a new position.

### `whence` Constants (`io` / `os` module)

| Constant | Value | Reference Point |
| :--- | :--- | :--- |
| `os.SEEK_SET` | `0` | Start of file (default) |
| `os.SEEK_CUR` | `1` | Current pointer position |
| `os.SEEK_END` | `2` | End of file |

### Important Text-Mode Restriction

In **text mode** (`"r"`, `"w"`, etc.), `seek()` only supports:
- `seek(0)` (beginning of file)
- `seek(0, 2)` (end of file)
- `seek(cookie)` using a value previously returned by `tell()`

Relative seeks from the current position or end with non-zero offsets (`seek(-5, 2)` or `seek(3, 1)`) raise `io.UnsupportedOperation` in text mode—use **binary mode** (`"rb"`) for arbitrary byte-offset seeking.

### Example (Binary Mode Seeking)

```python
import os

with open("data.bin", "wb") as f:
    f.write(b"0123456789ABCDEF")

with open("data.bin", "rb") as f:
    print("Start pos:", f.tell())        # 0
    print("First 4 bytes:", f.read(4))   # b'0123'
    print("After read:", f.tell())       # 4

    f.seek(2, os.SEEK_CUR)               # Skip 2 bytes ('45')
    print("At pos 6:", f.read(4))        # b'6789'

    f.seek(-4, os.SEEK_END)              # 4 bytes before EOF
    print("Last 4 bytes:", f.read())     # b'CDEF'

    f.seek(0, os.SEEK_SET)               # Rewind to start
    print("Rewound:", f.read(2))         # b'01'
```

---

## 7. Binary File Operations

Binary mode (`"rb"`, `"wb"`, `"ab"`, `"xb"`, `"r+b"`, `"w+b"`) reads and writes raw **`bytes`** (or `bytearray` / `memoryview`) with **no newline translation** and **no character encoding/decoding**. Use binary mode for images, audio, executables, compressed archives, pickled objects, or custom binary protocols.

### Copying a Binary File

```python
def copy_binary(src_path: str, dst_path: str, chunk_size: int = 65536) -> None:
    with open(src_path, "rb") as src, open(dst_path, "wb") as dst:
        while chunk := src.read(chunk_size):
            dst.write(chunk)
```

### Reading File Magic Bytes (Signature Check)

```python
def is_png(path: str) -> bool:
    png_signature = b"\x89PNG\r\n\x1a\n"
    with open(path, "rb") as f:
        header = f.read(8)
    return header == png_signature
```

### Packing/Unpacking Structured Binary Data (`struct`)

```python
import struct

# Pack an unsigned int (I) and a float (f) in little-endian (<)
record = struct.pack("<If", 42, 3.14)

with open("record.bin", "wb") as f:
    f.write(record)

with open("record.bin", "rb") as f:
    raw = f.read(struct.calcsize("<If"))
    item_id, value = struct.unpack("<If", raw)
    print(item_id, round(value, 2))  # 42 3.14
```

---

## 8. Working with CSV Files (`csv` Module)

Python's `csv` module handles quoting, escaping, delimiters, and embedded newlines properly—always prefer it over manual `.split(",")`.

> **Tip:** Always pass `newline=""` when opening CSV files so the `csv` module can manage `\r\n` and `\n` line endings correctly across platforms.

### Writing and Reading with `writer` and `reader`

```python
import csv

rows = [
    ["name", "role", "score"],
    ["Alice", "Engineer", 95],
    ["Bob", "Designer, Senior", 88],
]

with open("users.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(rows)

with open("users.csv", "r", encoding="utf-8", newline="") as f:
    reader = csv.reader(f)
    header = next(reader)
    print("Header:", header)
    for row in reader:
        print(row)
```

### Writing and Reading with `DictWriter` and `DictReader`

`DictReader` and `DictWriter` map columns by header name instead of numeric index:

```python
import csv

records = [
    {"name": "Alice", "role": "Engineer", "score": 95},
    {"name": "Carol", "role": "Manager", "score": 91},
]

with open("users_dict.csv", "w", encoding="utf-8", newline="") as f:
    fieldnames = ["name", "role", "score"]
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(records)

with open("users_dict.csv", "r", encoding="utf-8", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(f"{row['name']} ({row['role']}): {row['score']}")
```

### Custom Delimiters (e.g., TSV)

```python
import csv

with open("data.tsv", "r", encoding="utf-8", newline="") as f:
    reader = csv.reader(f, delimiter="\t", quotechar='"')
    for row in reader:
        print(row)
```

---

## 9. Working with JSON Files (`json` Module)

The `json` module serializes Python objects to JSON and deserializes JSON back into Python types (`dict`, `list`, `str`, `int`, `float`, `bool`, `None`).

| Function | Input / Output | Use Case |
| :--- | :--- | :--- |
| `json.dump(obj, fp, ...)` | Writes to file object | Save JSON to file |
| `json.dumps(obj, ...)` | Returns `str` | Serialize to JSON string |
| `json.load(fp, ...)` | Reads from file object | Load JSON from file |
| `json.loads(s, ...)` | Parses `str` / `bytes` | Parse JSON string |

### Writing and Reading JSON Files

```python
import json

config = {
    "app_name": "NotesApp",
    "version": 2.1,
    "debug": False,
    "plugins": ["spellcheck", "markdown"],
    "theme": None,
}

# Write pretty-printed JSON
with open("config.json", "w", encoding="utf-8") as f:
    json.dump(config, f, indent=4, sort_keys=True, ensure_ascii=False)

# Read JSON back
try:
    with open("config.json", "r", encoding="utf-8") as f:
        loaded = json.load(f)
    print(loaded["plugins"])
except json.JSONDecodeError as err:
    print(f"Invalid JSON at line {err.lineno}, col {err.colno}: {err.msg}")
```

### Custom Object Serialization (`default`)

Non-JSON-native types (like `Path` or `datetime`) raise `TypeError` unless handled via `default`:

```python
import json
from datetime import datetime, timezone
from pathlib import Path

def json_default(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    if isinstance(obj, Path):
        return str(obj)
    raise TypeError(f"Object of type {type(obj).__name__} is not JSON serializable")

payload = {
    "created_at": datetime.now(timezone.utc),
    "location": Path("/tmp/report.txt"),
}

print(json.dumps(payload, default=json_default, indent=2))
```

---

## 10. Working with XML Files

Python's standard library provides `xml.etree.ElementTree` (commonly imported as `ET`) for parsing, querying, modifying, and writing XML.

> **Security Note:** Standard `xml.etree.ElementTree` is not safe against maliciously crafted XML (such as billion-laughs entity expansion attacks). For untrusted external XML, prefer the third-party `defusedxml` package (`defusedxml.ElementTree`).

### Creating and Writing an XML File

```python
import xml.etree.ElementTree as ET

library = ET.Element("library")

book1 = ET.SubElement(library, "book", attrib={"id": "101", "lang": "en"})
ET.SubElement(book1, "title").text = "Fluent Python"
ET.SubElement(book1, "author").text = "Luciano Ramalho"

book2 = ET.SubElement(library, "book", attrib={"id": "102", "lang": "en"})
ET.SubElement(book2, "title").text = "Effective Python"
ET.SubElement(book2, "author").text = "Brett Slatkin"

tree = ET.ElementTree(library)
ET.indent(tree, space="  ")  # Available in Python 3.9+
tree.write("books.xml", encoding="utf-8", xml_declaration=True)
```

### Reading and Querying an XML File

```python
import xml.etree.ElementTree as ET

tree = ET.parse("books.xml")
root = tree.getroot()

print("Root tag:", root.tag)

for book in root.findall("book"):
    book_id = book.get("id")
    title = book.findtext("title")
    author = book.findtext("author")
    print(f"[{book_id}] {title} — {author}")

# XPath-like attribute lookup
match = root.find("./book[@id='102']/title")
if match is not None:
    print("Found book 102:", match.text)
```

### Modifying and Saving Back

```python
import xml.etree.ElementTree as ET

tree = ET.parse("books.xml")
root = tree.getroot()

for book in root.findall("book"):
    if book.get("id") == "101":
        book.set("edition", "2nd")

ET.indent(tree, space="  ")
tree.write("books_updated.xml", encoding="utf-8", xml_declaration=True)
```

---

## 11. `os` Module for File Operations

The `os` and `os.path` modules provide low-level, string-based filesystem operations.

### Path Inspection (`os.path`)

```python
import os

path = os.path.join("projects", "demo", "main.py")
print("Joined:", path)
print("Abs path:", os.path.abspath(path))
print("Dirname:", os.path.dirname(path))    # projects/demo
print("Basename:", os.path.basename(path))  # main.py
print("Splitext:", os.path.splitext(path))  # ('projects/demo/main', '.py')

print("Exists?", os.path.exists(path))
print("Is file?", os.path.isfile(path))
print("Is dir?", os.path.isdir(path))
```

### Directory & File Manipulation

```python
import os

# Create single directory (fails if parent missing or dir exists)
if not os.path.exists("logs"):
    os.mkdir("logs")

# Create nested directories safely
os.makedirs("data/2026/reports", exist_ok=True)

# List directory entries (names only)
entries = os.listdir(".")
print("Entries:", entries)

# Faster directory iteration with metadata (os.DirEntry)
with os.scandir(".") as it:
    for entry in it:
        kind = "DIR" if entry.is_dir() else "FILE"
        print(f"{kind:4} {entry.name}")

# Rename / move (os.replace overwrites destination atomically when possible)
with open("old_name.txt", "w", encoding="utf-8") as f:
    f.write("temp")
os.rename("old_name.txt", "new_name.txt")

# Remove file and empty directory
os.remove("new_name.txt")
os.rmdir("logs")  # Only works on empty directories
```

### Walking a Directory Tree (`os.walk`)

```python
import os

for root_dir, dirs, files in os.walk("data"):
    print(f"Directory: {root_dir}")
    print(f"  Subdirs: {dirs}")
    print(f"  Files:   {files}")
```

---

## 12. `pathlib` Module (Modern Path Handling)

Introduced in Python 3.4, `pathlib` offers an **object-oriented** interface (`Path`) that replaces most `os.path` and `os` boilerplate. It is the preferred choice in modern Python codebases.

### Constructing Paths and Accessing Parts

```python
from pathlib import Path

base = Path.cwd()
file_path = base / "reports" / "2026" / "summary.tar.gz"

print("Full:", file_path)
print("Name:", file_path.name)         # summary.tar.gz
print("Stem:", file_path.stem)         # summary.tar
print("Suffix:", file_path.suffix)     # .gz
print("Suffixes:", file_path.suffixes) # ['.tar', '.gz']
print("Parent:", file_path.parent)     # .../reports/2026
print("Home:", Path.home())
```

### Reading and Writing with `Path`

`Path` provides convenient one-shot helpers for small files, plus `.open()` for streaming:

```python
from pathlib import Path

p = Path("notes_demo.txt")

# Write and read text
p.write_text("Line 1\nLine 2\n", encoding="utf-8")
text = p.read_text(encoding="utf-8")
print(text)

# Write and read bytes
bin_path = Path("bytes_demo.bin")
bin_path.write_bytes(b"\x00\x01\x02")
print(bin_path.read_bytes())

# Standard streaming with .open()
with p.open("a", encoding="utf-8") as f:
    f.write("Line 3\n")
```

### Directory Creation, Globbing, Renaming, and Deletion

```python
from pathlib import Path

out_dir = Path("output/nested")
out_dir.mkdir(parents=True, exist_ok=True)

# Check type / existence
print(out_dir.exists(), out_dir.is_dir(), out_dir.is_file())

# Iterate immediate children
for item in Path(".").iterdir():
    print(item.name)

# Pattern matching (glob and recursive rglob)
for py_file in Path(".").rglob("*.py"):
    print(py_file)

# Rename and delete
tmp = Path("temp_file.txt")
tmp.write_text("hello", encoding="utf-8")
renamed = tmp.rename("renamed_file.txt")
renamed.unlink(missing_ok=True)  # Delete file safely
```

---

## 13. `shutil` Module (High-Level File Operations)

While `os` and `pathlib` work on individual files and empty directories, `shutil` handles high-level copying, moving, recursive directory deletion, and archiving.

| Function | Behavior |
| :--- | :--- |
| `shutil.copyfile(src, dst)` | Copies file contents only (neither metadata nor permissions); `dst` must be a file path. |
| `shutil.copy(src, dst)` | Copies file contents and permission bits; `dst` can be a directory. |
| `shutil.copy2(src, dst)` | Like `copy()`, plus preserves metadata (timestamps, etc.) when possible. |
| `shutil.copytree(src, dst, dirs_exist_ok=False)` | Recursively copies an entire directory tree. |
| `shutil.move(src, dst)` | Moves a file or directory (works across filesystems by copying + deleting). |
| `shutil.rmtree(path)` | Recursively deletes a directory and **all** its contents. |
| `shutil.make_archive(base_name, format, root_dir)` | Creates `.zip`, `.tar`, `.gztar`, etc. archives. |
| `shutil.disk_usage(path)` | Returns `(total, used, free)` bytes for the filesystem. |

### Examples

```python
import shutil
from pathlib import Path

Path("src_dir").mkdir(exist_ok=True)
Path("src_dir/hello.txt").write_text("Hello!", encoding="utf-8")

# Copy single file preserving metadata
shutil.copy2("src_dir/hello.txt", "src_dir/hello_backup.txt")

# Copy entire directory tree
shutil.copytree("src_dir", "dst_dir", dirs_exist_ok=True)

# Move file or directory
shutil.move("dst_dir/hello_backup.txt", "dst_dir/moved.txt")

# Check disk space
usage = shutil.disk_usage(".")
print(f"Free space: {usage.free / (1024 ** 3):.2f} GB")

# Create a zip archive: creates backup.zip from dst_dir
shutil.make_archive("backup", "zip", "dst_dir")

# Recursively delete directories (use with caution!)
shutil.rmtree("src_dir")
shutil.rmtree("dst_dir")
```

---

## 14. `tempfile` Module (Temporary Files & Directories)

The `tempfile` module creates temporary files and directories securely (with unique names and restrictive permissions) and cleans them up automatically when used as context managers.

### Main Classes & Functions

- `tempfile.TemporaryFile(mode="w+b")`: Anonymous temporary file (deleted immediately on close; may have no visible path on Unix).
- `tempfile.NamedTemporaryFile(mode="w+b", delete=True, suffix="", prefix="tmp")`: Temporary file with a visible filesystem path (`.name`). Note: on Windows, another process generally cannot open it while it is still open unless `delete=False` and closed first.
- `tempfile.SpooledTemporaryFile(max_size=0)`: Stays in memory until size exceeds `max_size` bytes, then spills over to disk.
- `tempfile.TemporaryDirectory()`: Creates a temporary directory and recursively deletes it on context exit.
- `tempfile.gettempdir()`: Returns the default system temp directory (e.g., `/tmp` on Linux).

### Examples

```python
import tempfile
from pathlib import Path

# 1. Anonymous temporary file
with tempfile.TemporaryFile(mode="w+", encoding="utf-8") as tmp:
    tmp.write("scratch data")
    tmp.seek(0)
    print("TemporaryFile:", tmp.read())

# 2. Named temporary file (useful when passing a path to external tools)
with tempfile.NamedTemporaryFile(
    mode="w+", encoding="utf-8", suffix=".txt", delete=True
) as named_tmp:
    print("Temp path:", named_tmp.name)
    named_tmp.write("visible on disk during context")
    named_tmp.flush()

# 3. Temporary directory
with tempfile.TemporaryDirectory() as tmp_dir:
    work_file = Path(tmp_dir) / "stage.txt"
    work_file.write_text("processing...", encoding="utf-8")
    print("Inside temp dir:", work_file.read_text(encoding="utf-8"))
# tmp_dir and everything inside it are deleted here
```

---

## 15. Reading and Writing Large Files Efficiently

Calling `.read()` or `.readlines()` loads the entire file into RAM, which can exhaust memory on multi-gigabyte files. Instead, process files **lazily** in lines or fixed-size chunks.

### 1. Streaming Line by Line (Text Files)

Iterating directly over the file object keeps memory usage near $O(1)$ (one line at a time):

```python
def count_errors(log_path: str) -> int:
    count = 0
    with open(log_path, "r", encoding="utf-8", errors="replace") as f:
        for line in f:
            if "ERROR" in line:
                count += 1
    return count
```

### 2. Chunked Reading with a Generator (Binary or Arbitrary Files)

Using `iter(callable, sentinel)` reads fixed-size blocks cleanly without loading the whole file:

```python
def read_in_chunks(file_path: str, chunk_size: int = 1024 * 1024):
    """Yield chunks of up to `chunk_size` bytes (default 1 MiB)."""
    with open(file_path, "rb") as f:
        for chunk in iter(lambda: f.read(chunk_size), b""):
            yield chunk

# Example: Compute SHA-256 hash of a multi-GB file with tiny RAM footprint
import hashlib

def sha256_file(file_path: str) -> str:
    digest = hashlib.sha256()
    for chunk in read_in_chunks(file_path):
        digest.update(chunk)
    return digest.hexdigest()
```

### 3. Fast Stream-to-Stream Copying (`shutil.copyfileobj`)

`shutil.copyfileobj()` copies data between two already-open file objects in buffered chunks:

```python
import shutil

with open("huge_input.bin", "rb") as src, open("huge_output.bin", "wb") as dst:
    shutil.copyfileobj(src, dst, length=1024 * 1024)  # 1 MiB buffer
```

### 4. Memory-Mapped Files (`mmap`)

For random-access reads or fast substring searches across large binary files, `mmap` maps the file directly into virtual memory so the OS pages data in on demand:

```python
import mmap

with open("large_data.bin", "rb") as f:
    with mmap.mmap(f.fileno(), length=0, access=mmap.ACCESS_READ) as mm:
        pos = mm.find(b"NEEDLE")
        print("Found at byte offset:", pos)
```

### 5. Buffered / Streaming Writes

When writing millions of rows, keep the file open across the entire loop (don't reopen on every iteration) and yield/process records lazily instead of building a huge list in memory first:

```python
def write_large_report(out_path: str, total_rows: int) -> None:
    with open(out_path, "w", encoding="utf-8", buffering=1024 * 1024) as f:
        for i in range(total_rows):
            f.write(f"row,{i},{i * i}\n")
```

---

## Quick Reference Cheat Sheet

| Task | Recommended Approach |
| :--- | :--- |
| Open text file safely | `with open(path, "r", encoding="utf-8") as f:` |
| Read line by line | `for line in f:` |
| Read/write small file at once | `Path(path).read_text(encoding="utf-8")` / `.write_text(...)` |
| Modern path manipulation | `from pathlib import Path` (`p / "sub" / "file.txt"`) |
| CSV with headers | `csv.DictReader` / `csv.DictWriter` with `newline=""` |
| JSON files | `json.load(f)` / `json.dump(obj, f, indent=4)` |
| Copy/move/delete trees | `shutil.copy2`, `shutil.move`, `shutil.rmtree` |
| Temporary workspace | `tempfile.TemporaryDirectory()` / `NamedTemporaryFile()` |
| Large binary processing | Chunked `iter(lambda: f.read(1024 * 1024), b"")` or `mmap` |
