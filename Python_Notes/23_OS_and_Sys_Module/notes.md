# Python `os`, `sys`, `subprocess`, and `platform` Modules

Python provides several built-in standard library modules for interacting with the operating system, the Python interpreter, external processes, and system/hardware metadata.

---

## Table of Contents

1. [`os` Module — Overview](#1-os-module--overview)
2. [`os.getcwd()` and `os.chdir()`](#2-osgetcwd-and-oschdir)
3. [`os.listdir()`, `os.scandir()`, and `os.walk()`](#3-oslistdir-osscandir-and-oswalk)
4. [`os.mkdir()`, `os.makedirs()`, `os.rmdir()`, and `os.removedirs()`](#4-osmkdir-osmakedirs-osrmdir-and-osremovedirs)
5. [`os.path` Utilities](#5-ospath-utilities)
6. [`os.rename()` and `os.remove()`](#6-osrename-and-osremove)
7. [`os.environ` (Environment Variables)](#7-osenviron-environment-variables)
8. [`os.system()` vs. `subprocess` Module](#8-ossystem-vs-subprocess-module)
9. [`sys` Module — Overview](#9-sys-module--overview)
10. [`sys.argv` (Command-Line Arguments)](#10-sysargv-command-line-arguments)
11. [`sys.path`](#11-syspath)
12. [`sys.stdin`, `sys.stdout`, and `sys.stderr`](#12-sysstdin-sysstdout-and-sysstderr)
13. [`sys.exit()`](#13-sysexit)
14. [`sys.getsizeof()`](#14-sysgetsizeof)
15. [`sys.version` and `sys.platform`](#15-sysversion-and-sysplatform)
16. [`subprocess` Module (`run`, `Popen`, `PIPE`)](#16-subprocess-module-run-popen-pipe)
17. [`platform` Module](#17-platform-module)

---

## 1. `os` Module — Overview

The `os` module provides a portable way to use operating-system-dependent functionality such as reading/writing file paths, managing directories, accessing environment variables, and querying OS information.

- **Key benefit**: Abstracts away differences between POSIX (Linux/macOS) and Windows where possible.
- **Useful module attributes**:
  - `os.name`: Name of the OS-dependent module imported (`'posix'` on Linux/macOS, `'nt'` on Windows).
  - `os.sep`: Directory separator (`'/'` on POSIX, `'\\'` on Windows).
  - `os.linesep`: Line terminator (`'\n'` on POSIX, `'\r\n'` on Windows).
  - `os.cpu_count()`: Number of logical CPUs in the system.

```python
import os

print("OS name:", os.name)         # e.g., 'posix' or 'nt'
print("Separator:", repr(os.sep))  # '/' or '\\'
print("CPU count:", os.cpu_count())
```

> **Tip:** For modern path manipulation (Python 3.4+), `pathlib.Path` is often preferred over `os.path` for cleaner object-oriented syntax, though `os` remains essential for many OS operations and legacy codebases.

---

## 2. `os.getcwd()` and `os.chdir()`

- **`os.getcwd()`**: Returns a string representing the **Current Working Directory (CWD)**—the directory from which the Python script is executing.
- **`os.chdir(path)`**: Changes the current working directory to `path`. Raises `FileNotFoundError` if `path` does not exist, or `NotADirectoryError` if it is a file.

```python
import os

# Get current working directory
original_dir = os.getcwd()
print("Current directory:", original_dir)

# Change to another directory (e.g., /tmp on Linux/macOS)
target_dir = os.path.dirname(original_dir)
os.chdir(target_dir)
print("New directory:", os.getcwd())

# Always restore or handle errors safely
try:
    os.chdir("/nonexistent/path")
except FileNotFoundError as e:
    print(f"Error changing directory: {e}")
finally:
    os.chdir(original_dir)
```

---

## 3. `os.listdir()`, `os.scandir()`, and `os.walk()`

### `os.listdir(path='.')`
Returns a list of **names** (strings) of all entries (files and subdirectories) in `path`, excluding `.` and `..`. It does **not** distinguish files from directories on its own.

```python
import os

entries = os.listdir(".")
print("Entries:", entries)
```

### `os.scandir(path='.')`
Introduced in Python 3.5, `os.scandir()` returns an iterator of **`os.DirEntry` objects**. It is significantly faster than `os.listdir()` + `os.path.isdir()`/`os.path.isfile()` because `DirEntry` caches file type and stat information retrieved during the directory scan. Use it with a `with` statement to ensure resources are released.

```python
import os

with os.scandir(".") as entries:
    for entry in entries:
        kind = "DIR " if entry.is_dir() else "FILE"
        size = entry.stat().st_size if entry.is_file() else 0
        print(f"{kind} | {entry.name:<25} | {size} bytes")
```

### `os.walk(top, topdown=True, ...)`
Recursively traverses a directory tree starting at `top`. On each step, it yields a 3-tuple: `(dirpath, dirnames, filenames)`:
- `dirpath`: String path to the current directory.
- `dirnames`: List of subdirectory names in `dirpath`.
- `filenames`: List of non-directory file names in `dirpath`.

```python
import os

for dirpath, dirnames, filenames in os.walk("."):
    # Skip hidden directories (modifying dirnames in-place skips recursion into them)
    dirnames[:] = [d for d in dirnames if not d.startswith(".")]

    print(f"Directory: {dirpath}")
    for filename in filenames:
        full_path = os.path.join(dirpath, filename)
        print(f"  - {full_path}")
```

| Function | Return Type | Recursive? | Performance / Best Use Case |
| :--- | :--- | :--- | :--- |
| `os.listdir()` | `list[str]` | No | Simple flat listing of entry names |
| `os.scandir()` | Iterator of `DirEntry` | No | Fast directory scan needing file/dir checks or `stat()` |
| `os.walk()` | Generator of `(dirpath, dirnames, filenames)` | Yes | Traversing entire directory trees |

---

## 4. `os.mkdir()`, `os.makedirs()`, `os.rmdir()`, and `os.removedirs()`

### Creating Directories
- **`os.mkdir(path)`**: Creates a **single** directory. Raises `FileNotFoundError` if parent directories don't exist, and `FileExistsError` if it already exists.
- **`os.makedirs(path, exist_ok=False)`**: Creates a directory **recursively**, including any missing intermediate parent directories. Set `exist_ok=True` to avoid `FileExistsError` if the target already exists.

```python
import os

# Create a single directory
if not os.path.exists("single_dir"):
    os.mkdir("single_dir")

# Create nested directories safely
os.makedirs("parent_dir/child_dir/grandchild_dir", exist_ok=True)
```

### Removing Directories
- **`os.rmdir(path)`**: Removes a **single empty** directory. Raises `OSError` if the directory is not empty.
- **`os.removedirs(path)`**: Removes `path` and then recursively removes empty parent directories going upward until a non-empty directory is encountered.

```python
import os

# Remove single empty directory
os.rmdir("single_dir")

# Remove nested empty directories from leaf upward
os.removedirs("parent_dir/child_dir/grandchild_dir")
```

> **Note:** To remove a **non-empty** directory and all of its contents, use `shutil.rmtree(path)` from the `shutil` module.

---

## 5. `os.path` Utilities

The `os.path` submodule handles pathname manipulations safely across platforms.

| Function | Purpose | Example Output (POSIX) |
| :--- | :--- | :--- |
| `os.path.join(a, *p)` | Joins path components using OS separator | `os.path.join("a", "b", "c.txt")` $\rightarrow$ `'a/b/c.txt'` |
| `os.path.exists(path)` | Returns `True` if path exists (file or dir) | `True` / `False` |
| `os.path.isfile(path)` | Returns `True` if path is an existing regular file | `True` / `False` |
| `os.path.isdir(path)` | Returns `True` if path is an existing directory | `True` / `False` |
| `os.path.basename(path)` | Returns the final component of a path | `os.path.basename("/a/b/c.txt")` $\rightarrow$ `'c.txt'` |
| `os.path.dirname(path)` | Returns the parent directory portion | `os.path.dirname("/a/b/c.txt")` $\rightarrow$ `'/a/b'` |
| `os.path.splitext(path)` | Splits path into `(root, extension)` | `os.path.splitext("a/b.tar.gz")` $\rightarrow$ `('a/b.tar', '.gz')` |
| `os.path.getsize(path)` | Returns file size in bytes (raises `OSError` if missing) | `1024` |
| `os.path.abspath(path)` | Returns normalized absolute path | `'/home/user/project/file.py'` |

```python
import os

base_dir = "/home/user/projects"
file_path = os.path.join(base_dir, "app", "main.py")

print("Joined path:", file_path)
print("Directory name:", os.path.dirname(file_path))   # /home/user/projects/app
print("Base name:", os.path.basename(file_path))       # main.py

root, ext = os.path.splitext(file_path)
print(f"Root: {root}, Extension: {ext}")               # .../main, .py

# Inspecting an existing path (e.g., current script or current directory)
abs_cur = os.path.abspath(".")
print("Absolute path of '.':", abs_cur)
print("Exists:", os.path.exists(abs_cur))
print("Is dir:", os.path.isdir(abs_cur))
print("Is file:", os.path.isfile(abs_cur))

# File size check
if os.path.isfile(__file__):
    print("Current script size (bytes):", os.path.getsize(__file__))
```

> **Gotcha with `os.path.join`:** If any argument is an absolute path (e.g., starts with `/`), all previous components are discarded:
> `os.path.join("/home/user", "/etc/passwd")` $\rightarrow$ `'/etc/passwd'`.

---

## 6. `os.rename()` and `os.remove()`

- **`os.rename(src, dst)`**: Renames or moves a file or directory from `src` to `dst`.
  - On POSIX, if `dst` is an existing file, it may silently overwrite it; on Windows, it raises `FileExistsError`. Use `os.replace(src, dst)` for consistent atomic overwriting across platforms.
- **`os.remove(path)`** (alias `os.unlink(path)`): Deletes a **file** (not a directory). Raises `IsADirectoryError` on directories and `FileNotFoundError` if missing.

```python
import os

# Create a temporary file
with open("old_name.txt", "w") as f:
    f.write("Hello, World!")

# Rename the file
os.rename("old_name.txt", "new_name.txt")
print("Exists after rename:", os.path.exists("new_name.txt"))

# Remove the file
if os.path.isfile("new_name.txt"):
    os.remove("new_name.txt")
print("Exists after remove:", os.path.exists("new_name.txt"))
```

---

## 7. `os.environ` (Environment Variables)

`os.environ` is a mapping object representing the string environment variables of the current process.

- Reading with `os.environ["VAR"]` raises `KeyError` if `VAR` is missing.
- Reading with `os.environ.get("VAR", "default")` or `os.getenv("VAR", "default")` safely returns a fallback value.
- Modifying `os.environ["MY_VAR"] = "value"` only affects the **current process and any child processes** spawned by it—it does **not** permanently change the parent shell's environment.
- Values in `os.environ` must always be strings (`str`).

```python
import os

# Safe reading
home = os.environ.get("HOME") or os.environ.get("USERPROFILE")
api_key = os.getenv("API_KEY", "not-configured")

print("Home directory:", home)
print("API key:", api_key)

# Setting and deleting environment variables for the current process
os.environ["APP_ENV"] = "development"
print("APP_ENV:", os.environ["APP_ENV"])

# Removing safely
os.environ.pop("APP_ENV", None)
```

---

## 8. `os.system()` vs. `subprocess` Module

### `os.system(command)`
Executes `command` (a string) in a subshell and returns the exit status code.

**Limitations of `os.system()`:**
1. **Security risk**: Invokes a shell, making it vulnerable to **shell injection** if user input is concatenated into the command string.
2. **No direct output capture**: Output goes directly to standard output; capturing stdout/stderr cleanly is difficult.
3. **Platform-dependent exit codes** and shell syntax.

```python
import os

# Legacy / quick-and-dirty usage (not recommended for production)
exit_code = os.system("echo Hello from os.system")
print("Exit code:", exit_code)
```

### Why `subprocess` Is Preferred
The `subprocess` module replaces `os.system()`, `os.spawn*`, and `os.popen*`. It:
- Avoids invoking a shell by default (`shell=False`), preventing shell injection.
- Lets you capture `stdout` and `stderr`, pass `stdin`, set timeouts, and check exit codes cleanly.

```python
import subprocess

result = subprocess.run(["echo", "Hello from subprocess"], capture_output=True, text=True)
print("Output:", result.stdout.strip())
print("Return code:", result.returncode)
```

---

## 9. `sys` Module — Overview

While `os` interacts with the **operating system**, the `sys` module provides access to variables and functions that interact closely with the **Python interpreter** itself.

Common capabilities include:
- Reading command-line arguments (`sys.argv`)
- Inspecting/modifying module search paths (`sys.path`)
- Accessing standard I/O streams (`sys.stdin`, `sys.stdout`, `sys.stderr`)
- Exiting the interpreter (`sys.exit`)
- Inspecting memory size (`sys.getsizeof`), interpreter version (`sys.version`), and platform (`sys.platform`)
- Checking recursion limit (`sys.getrecursionlimit()`, `sys.setrecursionlimit()`)

```python
import sys

print("Python executable:", sys.executable)
print("Recursion limit:", sys.getrecursionlimit())
print("Max int size (64-bit vs 32-bit):", sys.maxsize)
```

---

## 10. `sys.argv` (Command-Line Arguments)

`sys.argv` is a list of strings containing the command-line arguments passed to a Python script:
- `sys.argv[0]`: The script name (or path) invoked.
- `sys.argv[1:]`: The additional arguments passed by the user.

```python
# save as greet.py
import sys

print("All arguments:", sys.argv)
print("Script name:", sys.argv[0])

if len(sys.argv) < 2:
    print("Usage: python greet.py <name> [count]")
else:
    name = sys.argv[1]
    count = int(sys.argv[2]) if len(sys.argv) > 2 else 1
    for _ in range(count):
        print(f"Hello, {name}!")
```

Running `python greet.py Alice 2`:
```text
All arguments: ['greet.py', 'Alice', '2']
Script name: greet.py
Hello, Alice!
Hello, Alice!
```

> **Note:** For complex CLI tools with flags (`--verbose`, `-o file.txt`), use the standard library `argparse` module instead of parsing `sys.argv` manually.

---

## 11. `sys.path`

`sys.path` is a list of directory paths (strings) that Python searches when importing modules via `import`.

Search order:
1. The directory containing the input script (or current directory in interactive mode).
2. Directories listed in the `PYTHONPATH` environment variable.
3. Installation-dependent default paths (standard library and `site-packages`).

```python
import sys

for index, path in enumerate(sys.path):
    print(f"{index}: {path}")

# Temporarily add a custom directory at runtime
custom_lib = "/path/to/my_libs"
if custom_lib not in sys.path:
    sys.path.append(custom_lib)  # Or sys.path.insert(0, custom_lib) for priority
```

---

## 12. `sys.stdin`, `sys.stdout`, and `sys.stderr`

These are file-like stream objects connected to the interpreter's standard input, output, and error channels:

- **`sys.stdin`**: Standard input stream (used internally by `input()`).
- **`sys.stdout`**: Standard output stream (used internally by `print()`).
- **`sys.stderr`**: Standard error stream (used for error messages, tracebacks, and logging so diagnostics stay separate from normal output).

```python
import sys

# Writing directly to stdout and stderr
sys.stdout.write("Normal output message\n")
sys.stderr.write("Error or diagnostic message\n")

# Reading piped input or user lines from stdin
# Example usage in shell: echo -e "line1\nline2" | python script.py
if not sys.stdin.isatty():
    for line in sys.stdin:
        sys.stdout.write(f"Read: {line.strip()}\n")
```

### Temporarily Redirecting `sys.stdout`

```python
import io
import sys

buffer = io.StringIO()
original_stdout = sys.stdout

try:
    sys.stdout = buffer
    print("Captured inside buffer!")
finally:
    sys.stdout = original_stdout

print("Captured text:", buffer.getvalue().strip())
```

---

## 13. `sys.exit()`

`sys.exit([arg])` exits the Python program by raising the **`SystemExit`** exception.

- `sys.exit(0)` or `sys.exit()`: Normal/successful termination (exit code `0`).
- `sys.exit(1)` (any non-zero integer, typically `1`–`255`): Abnormal/error termination.
- `sys.exit("Error message")`: Prints the string to `sys.stderr` and exits with code `1`.

Because it raises `SystemExit`, `finally` blocks still execute and cleanup happens properly.

```python
import sys

def divide(a, b):
    if b == 0:
        sys.exit("Fatal error: division by zero is not allowed.")
    return a / b

try:
    print(divide(10, 0))
except SystemExit as e:
    print(f"Caught SystemExit with code/message: {e}")
```

---

## 14. `sys.getsizeof()`

`sys.getsizeof(object[, default])` returns the memory size of an object **in bytes** (only the object's direct footprint, **not** the size of objects it references inside containers).

```python
import sys

num = 42
text = "Hello, Python!"
items = [1, 2, 3, 4, 5]
empty_list = []
gen = (x for x in range(1_000_000))
big_list = [x for x in range(1_000_000)]

print("int(42):", sys.getsizeof(num), "bytes")
print("str:", sys.getsizeof(text), "bytes")
print("empty list:", sys.getsizeof(empty_list), "bytes")
print("5-item list:", sys.getsizeof(items), "bytes")
print("generator (1M items):", sys.getsizeof(gen), "bytes")
print("list (1M items):", sys.getsizeof(big_list), "bytes")
```

> **Important:** For a container like `items = ["a very long string" * 1000]`, `sys.getsizeof(items)` only measures the list pointer array itself, not the string elements inside it.

---

## 15. `sys.version` and `sys.platform`

### `sys.version` and `sys.version_info`
- **`sys.version`**: A human-readable string showing the Python version, build number, and compiler.
- **`sys.version_info`**: A named tuple `(major, minor, micro, releaselevel, serial)` ideal for programmatic version checks.

### `sys.platform`
A string identifying the OS platform (useful for conditional platform logic):
- `'linux'`: Linux
- `'win32'`: Windows (both 32-bit and 64-bit)
- `'darwin'`: macOS

```python
import sys

print("Python version string:", sys.version)
print("Version tuple:", sys.version_info)
print("Platform:", sys.platform)

if sys.version_info < (3, 10):
    print("Please upgrade to Python 3.10 or newer.")

if sys.platform.startswith("linux"):
    print("Running on Linux")
elif sys.platform == "win32":
    print("Running on Windows")
elif sys.platform == "darwin":
    print("Running on macOS")
```

---

## 16. `subprocess` Module (`run`, `Popen`, `PIPE`)

The `subprocess` module spawns new processes, connects to their input/output/error pipes, and obtains their return codes.

### `subprocess.run()` (Recommended High-Level API)
Runs a command, waits for it to finish, and returns a `CompletedProcess` instance.

Key parameters:
- `args`: List of command + arguments (e.g., `["ls", "-la"]`) or string if `shell=True`.
- `capture_output=True`: Captures both `stdout` and `stderr` (shortcut for `stdout=subprocess.PIPE, stderr=subprocess.PIPE`).
- `text=True`: Decodes byte streams to strings (`str`).
- `check=True`: Raises `subprocess.CalledProcessError` if the command returns a non-zero exit code.
- `timeout`: Raises `subprocess.TimeoutExpired` if the command takes too long.
- `input`: Passes data to the process's `stdin`.

```python
import subprocess

# 1. Basic command with captured output
res = subprocess.run(
    ["python3", "--version"],
    capture_output=True,
    text=True,
    check=True,
)
print("Return code:", res.returncode)
print("Stdout:", res.stdout.strip())

# 2. Passing input via stdin
res_input = subprocess.run(
    ["python3", "-c", "import sys; print(sys.stdin.read().upper())"],
    input="hello subprocess",
    capture_output=True,
    text=True,
)
print("Transformed output:", res_input.stdout.strip())

# 3. Error handling with check=True and timeout
try:
    subprocess.run(["ls", "/nonexistent_folder"], capture_output=True, text=True, check=True, timeout=5)
except subprocess.CalledProcessError as e:
    print(f"Command failed (code {e.returncode}): {e.stderr.strip()}")
except subprocess.TimeoutExpired:
    print("Command timed out!")
```

### `subprocess.Popen()` and `subprocess.PIPE` (Low-Level / Streaming API)
`Popen` starts a process **without blocking** immediately, making it useful for:
- Streaming real-time output line by line
- Running background tasks concurrently
- Chaining multiple processes via pipes (`PIPE`)

```python
import subprocess

# Streaming output line-by-line from a running process
proc = subprocess.Popen(
    ["python3", "-c", "import time\nfor i in range(3): print(f'step {i}'); time.sleep(0.2)"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True,
)

for line in proc.stdout:
    print("Streamed:", line.strip())

proc.wait()
print("Final exit code:", proc.returncode)
```

### Chaining Two Commands with `PIPE` (`cmd1 | cmd2`)

```python
import subprocess

# Equivalent to: printf "banana\napple\ncherry" | sort
p1 = subprocess.Popen(
    ["python3", "-c", "print('banana\\napple\\ncherry')"],
    stdout=subprocess.PIPE,
)
p2 = subprocess.Popen(
    ["sort"],
    stdin=p1.stdout,
    stdout=subprocess.PIPE,
    text=True,
)
p1.stdout.close()  # Allow p1 to receive SIGPIPE if p2 exits early

output, errors = p2.communicate()
print("Sorted output:\n" + output.strip())
```

---

## 17. `platform` Module

The `platform` module provides detailed, high-level system, OS, hardware, and Python implementation information—much more descriptive than `os.name` or `sys.platform`.

| Function | Description | Example Output |
| :--- | :--- | :--- |
| `platform.system()` | OS name | `'Linux'`, `'Windows'`, `'Darwin'` |
| `platform.release()` | OS release version | `'6.8.0-generic'` or `'10'` |
| `platform.version()` | Detailed OS build/version string | `'#45-Ubuntu SMP ...'` |
| `platform.platform()` | Combined readable OS summary | `'Linux-6.8.0-x86_64-with-glibc2.39'` |
| `platform.machine()` | Hardware architecture | `'x86_64'`, `'AMD64'`, `'arm64'` |
| `platform.processor()` | Processor name (may be empty on some systems) | `'x86_64'` |
| `platform.node()` | Network hostname of the machine | `'my-workstation'` |
| `platform.architecture()` | Bit architecture and linkage format | `('64bit', 'ELF')` |
| `platform.python_version()` | Python version string | `'3.12.3'` |
| `platform.python_implementation()` | Python runtime | `'CPython'`, `'PyPy'` |
| `platform.uname()` | Named tuple with system, node, release, version, machine, processor | `uname_result(...)` |

```python
import platform

print("System:        ", platform.system())
print("Release:       ", platform.release())
print("OS Summary:    ", platform.platform())
print("Machine:       ", platform.machine())
print("Processor:     ", platform.processor())
print("Hostname:      ", platform.node())
print("Architecture:  ", platform.architecture())
print("Python Impl:   ", platform.python_implementation())
print("Python Version:", platform.python_version())
print("Full uname:    ", platform.uname())
```

---

## Quick Summary Cheat Sheet

- **File paths & directories**: `os.getcwd()`, `os.chdir()`, `os.makedirs()`, `os.scandir()`, `os.walk()`, `os.path.*` (or `pathlib.Path`).
- **Environment variables**: `os.environ.get("KEY", "default")`, `os.getenv("KEY")`.
- **Interpreter & CLI**: `sys.argv`, `sys.path`, `sys.stdin/stdout/stderr`, `sys.exit()`, `sys.getsizeof()`, `sys.version`, `sys.platform`.
- **Running external commands**: `subprocess.run(..., capture_output=True, text=True, check=True)` or `subprocess.Popen(..., stdout=subprocess.PIPE)` for streaming/piping.
- **Detailed hardware/OS info**: `platform.system()`, `platform.machine()`, `platform.platform()`, `platform.uname()`.
