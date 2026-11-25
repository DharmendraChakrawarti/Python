# 26. Multithreading, Multiprocessing, and Asyncio in Python

Modern applications frequently need to handle multiple tasks efficiently—whether waiting on network responses, reading files, or crunching large datasets across multiple CPU cores. Python provides three primary concurrency and parallelism models in the standard library:

- **`threading`**: OS-level threads sharing memory (best for blocking I/O-bound work).
- **`multiprocessing`**: Separate OS processes with independent memory spaces (best for CPU-bound work).
- **`asyncio`**: Single-threaded cooperative multitasking via an event loop (best for high-concurrency I/O-bound work).

---

## Table of Contents

1. [Concurrency vs Parallelism](#1-concurrency-vs-parallelism)
2. [The GIL (Global Interpreter Lock)](#2-the-gil-global-interpreter-lock)
3. [The `threading` Module Basics](#3-the-threading-module-basics)
4. [Creating Threads](#4-creating-threads)
5. [Thread Methods and Attributes](#5-thread-methods-and-attributes)
6. [Thread Synchronization](#6-thread-synchronization)
7. [Thread-Safe Data Structures (`queue`)](#7-thread-safe-data-structures-queue)
8. [The `multiprocessing` Module Basics](#8-the-multiprocessing-module-basics)
9. [The `Process` Class](#9-the-process-class)
10. [`Pool` for Parallel Execution](#10-pool-for-parallel-execution)
11. [Sharing Data Between Processes](#11-sharing-data-between-processes)
12. [`concurrent.futures`](#12-concurrentfutures)
13. [`asyncio` Basics](#13-asyncio-basics)
14. [`asyncio.gather` and `asyncio.create_task`](#14-asynciogather-and-asynciocreate_task)
15. [When to Use `threading` vs `multiprocessing` vs `asyncio`](#15-when-to-use-threading-vs-multiprocessing-vs-asyncio)
16. [Common Concurrency Patterns and Pitfalls](#16-common-concurrency-patterns-and-pitfalls)

---

## 1. Concurrency vs Parallelism

Although often used interchangeably, **concurrency** and **parallelism** describe different execution properties:

| Concept | Definition | Analogy | Hardware Requirement |
| :--- | :--- | :--- | :--- |
| **Concurrency** | Structuring a program to **manage** multiple tasks whose lifetimes overlap. Tasks make progress by interleaving execution (time-slicing or yielding while waiting). | One chef chopping vegetables while waiting for water to boil. | Can run on a **single CPU core**. |
| **Parallelism** | **Executing** multiple computations simultaneously at the exact same instant. | Two chefs working at two separate stoves at the same time. | Requires **multiple CPU cores** (or hardware threads). |

### Key Takeaways
- **Concurrency is about structure**: dealing with many things at once (e.g., handling 1,000 open network sockets).
- **Parallelism is about execution**: doing many things at once (e.g., computing matrix operations across 8 CPU cores).
- A program can be:
  - **Concurrent but not parallel**: `asyncio` or Python threads running CPU-bound Python bytecode under the GIL.
  - **Parallel and concurrent**: `multiprocessing` running independent workers across multiple CPU cores.

---

## 2. The GIL (Global Interpreter Lock)

### What Is the GIL?
The **Global Interpreter Lock (GIL)** is a mutex (lock) inside **CPython** (the standard Python implementation) that allows **only one thread to execute Python bytecode at a time** within a single process, even on multi-core machines.

### Why Does the GIL Exist?
CPython manages memory using **reference counting**. Every Python object tracks how many references point to it (`sys.getrefcount(obj)`). When the count hits `0`, the memory is freed.
- Without the GIL, every object's reference count would need fine-grained locks to prevent race conditions and memory corruption.
- Fine-grained locking adds heavy overhead to single-threaded programs and risks deadlocks inside the interpreter itself.
- The GIL was a pragmatic design choice: it keeps CPython's C-API simple, safe for C extensions, and fast for single-threaded execution.

### Why It Matters
1. **CPU-Bound Threads Do Not Scale Across Cores**:
   If you spawn 4 Python threads to compute prime numbers or hash values in pure Python, only one thread runs bytecode at any given moment. Total runtime is often *slower* than single-threaded execution due to thread-switching overhead.
2. **I/O-Bound Threads Still Benefit**:
   CPython automatically **releases the GIL** whenever a thread performs blocking I/O (network sockets, disk reads/writes, `time.sleep()`) and in certain C extensions (like NumPy, hashlib, or compression libraries). While one thread waits on the network, other threads can run Python code.
3. **Free-Threaded Python (PEP 703)**:
   Starting with Python 3.13+, an experimental **free-threaded** build (`python3.13t`) allows disabling the GIL (`--disable-gil`), enabling true multi-core parallelism with threads. However, in standard CPython builds, the GIL remains active by default.

```python
import threading
import time


def cpu_work(n: int) -> int:
    total = 0
    for i in range(n):
        total += i * i
    return total


# Sequential execution
start = time.perf_counter()
cpu_work(20_000_000)
cpu_work(20_000_000)
print(f"Sequential: {time.perf_counter() - start:.2f}s")

# Multithreaded CPU-bound execution (bound by the GIL!)
start = time.perf_counter()
t1 = threading.Thread(target=cpu_work, args=(20_000_000,))
t2 = threading.Thread(target=cpu_work, args=(20_000_000,))
t1.start()
t2.start()
t1.join()
t2.join()
print(f"2 Threads:  {time.perf_counter() - start:.2f}s")
```

---

## 3. The `threading` Module Basics

The `threading` module provides high-level support for OS-managed threads within a single Python process.

### Characteristics of Threads
- **Shared memory**: All threads in the same process share the same heap, global variables, and open file descriptors.
- **Lightweight creation**: Creating a thread is much faster and uses less memory than creating a new OS process.
- **Preemptive multitasking**: The OS scheduler decides when to pause one thread and resume another—context switches can happen between almost any two bytecode instructions.
- **Useful inspection helpers**:
  - `threading.current_thread()` — returns the `Thread` object for the currently executing thread.
  - `threading.main_thread()` — returns the main `Thread` object.
  - `threading.active_count()` — returns the number of currently alive `Thread` objects.
  - `threading.enumerate()` — returns a list of all currently alive `Thread` objects.

```python
import threading
import time


def worker():
    current = threading.current_thread()
    print(f"Running in: {current.name} (ident={current.ident})")
    time.sleep(0.5)


print(f"Main thread: {threading.main_thread().name}")
t = threading.Thread(target=worker, name="Worker-1")
t.start()
print(f"Active threads: {threading.active_count()}")
t.join()
print(f"Active threads after join: {threading.active_count()}")
```

---

## 4. Creating Threads

There are two standard ways to create a thread with `threading.Thread`.

### Method 1: Passing a `target` Function (Recommended)

Pass the callable to `target` and pass positional/keyword arguments via `args` and `kwargs`.

> **Tip**: Always include a trailing comma for single-element `args` tuples: `args=(item,)`, not `args=(item)`.

```python
import threading
import time


def download_file(url: str, timeout: int = 5) -> None:
    print(f"[{threading.current_thread().name}] Downloading {url} (timeout={timeout}s)...")
    time.sleep(1)
    print(f"[{threading.current_thread().name}] Finished {url}")


threads = []
urls = ["https://example.com/a", "https://example.com/b", "https://example.com/c"]

for idx, url in enumerate(urls, start=1):
    t = threading.Thread(
        target=download_file,
        args=(url,),
        kwargs={"timeout": 10},
        name=f"Downloader-{idx}",
    )
    t.start()
    threads.append(t)

for t in threads:
    t.join()

print("All downloads complete.")
```

### Method 2: Subclassing `threading.Thread`

Override the `run()` method (and optionally `__init__()`). Always call `super().__init__()` when overriding `__init__()`.

```python
import threading
import time


class WorkerThread(threading.Thread):
    def __init__(self, task_id: int, delay: float) -> None:
        super().__init__(name=f"Worker-{task_id}")
        self.task_id = task_id
        self.delay = delay
        self.result: str | None = None

    def run(self) -> None:
        print(f"{self.name} starting...")
        time.sleep(self.delay)
        self.result = f"Task {self.task_id} completed"
        print(f"{self.name} finished.")


w1 = WorkerThread(1, 0.5)
w2 = WorkerThread(2, 0.3)

w1.start()
w2.start()

w1.join()
w2.join()

print(w1.result, "|", w2.result)
```

---

## 5. Thread Methods and Attributes

| Method / Attribute | Description |
| :--- | :--- |
| `t.start()` | Schedules the thread to begin executing `run()` in a new OS thread. Can only be called **once** per `Thread` instance. |
| `t.run()` | Contains the thread's logic. Calling `t.run()` directly runs it **synchronously in the current thread**—do not confuse with `t.start()`. |
| `t.join(timeout=None)` | Blocks the calling thread until thread `t` finishes, or until `timeout` seconds elapse. |
| `t.is_alive()` | Returns `True` from the moment `start()` is called until `run()` completes. |
| `t.daemon` | Boolean property (`True`/`False`). Must be set **before** `start()`, or passed via `Thread(..., daemon=True)`. |
| `t.name` | Human-readable string name of the thread. |
| `t.ident` | Unique OS thread identifier (assigned after `start()`, `None` before). |
| `t.native_id` | OS-assigned thread ID (TID) on platforms that support it (Python 3.8+). |

### Understanding `daemon` Threads
- **Non-daemon threads (`daemon=False`, default)**: The Python process will **wait** for all non-daemon threads to finish before exiting, even if the main thread reaches the end of the script.
- **Daemon threads (`daemon=True`)**: Background threads that are **abruptly terminated** when all non-daemon threads exit. Use daemon threads only for background tasks that do not hold resources needing graceful cleanup (e.g., heartbeat logs or monitors), because `finally` blocks and open file flushes may not run on exit.

```python
import threading
import time


def background_heartbeat() -> None:
    while True:
        print("[Heartbeat] alive...")
        time.sleep(0.4)


def main_task() -> None:
    print("[MainTask] working...")
    time.sleep(1.0)
    print("[MainTask] done.")


heartbeat = threading.Thread(target=background_heartbeat, daemon=True)
task = threading.Thread(target=main_task)

heartbeat.start()
task.start()

print(f"Task alive? {task.is_alive()}")
task.join(timeout=2.0)
print(f"Task alive after join? {task.is_alive()}")
# Process exits here even though heartbeat is still in an infinite loop,
# because heartbeat.daemon is True.
```

---

## 6. Thread Synchronization

When multiple threads read and write shared mutable state, interleaved operations can cause **race conditions**. Synchronization primitives coordinate access to shared resources.

### 1. `threading.Lock` (Mutual Exclusion)
A `Lock` has two states: *locked* and *unlocked*. Only one thread can hold the lock at a time. Always prefer the `with lock:` context manager so the lock is released even if an exception occurs.

```python
import threading

counter = 0
lock = threading.Lock()


def increment(n: int) -> None:
    global counter
    for _ in range(n):
        with lock:  # Acquires on enter, releases on exit
            counter += 1


threads = [threading.Thread(target=increment, args=(100_000,)) for _ in range(4)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"Final counter: {counter}")  # Always 400000
```

> **Warning**: A standard `Lock` is **not reentrant**. If a thread holding a `Lock` calls `lock.acquire()` again before releasing it, it deadlocks itself.

### 2. `threading.RLock` (Reentrant Lock)
An `RLock` can be acquired **multiple times by the same thread** that already owns it. The thread must call `release()` the exact same number of times before another thread can acquire it. Useful for recursive functions or methods calling other locked methods on the same object.

```python
import threading


class BankAccount:
    def __init__(self, balance: float) -> None:
        self.balance = balance
        self._lock = threading.RLock()

    def deposit(self, amount: float) -> None:
        with self._lock:
            self.balance += amount

    def transfer_bonus(self, amount: float) -> None:
        with self._lock:
            # Calls deposit(), which acquires the same RLock again safely
            self.deposit(amount)
            self.deposit(amount * 0.05)


acct = BankAccount(100)
acct.transfer_bonus(100)
print(f"Balance: {acct.balance}")  # 205.0
```

### 3. `threading.Semaphore` (and `BoundedSemaphore`)
A `Semaphore` maintains an internal counter initialized to `value` (default `1`).
- `acquire()` decrements the counter (blocks if counter is `0`).
- `release()` increments the counter.
Use it to limit concurrent access to a fixed-capacity resource (e.g., max 3 database connections or API requests at once). Use `BoundedSemaphore` to raise a `ValueError` if `release()` is accidentally called more times than `acquire()`.

```python
import threading
import time

db_pool = threading.BoundedSemaphore(value=2)


def query_db(worker_id: int) -> None:
    with db_pool:
        print(f"Worker {worker_id} acquired DB connection")
        time.sleep(0.5)
        print(f"Worker {worker_id} releasing DB connection")


threads = [threading.Thread(target=query_db, args=(i,)) for i in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

### 4. `threading.Event`
An `Event` wraps a boolean flag for simple one-to-many thread signaling:
- `event.set()` — sets the flag to `True` and wakes up all waiting threads.
- `event.clear()` — resets the flag to `False`.
- `event.wait(timeout=None)` — blocks until the flag is `True` (or timeout expires).
- `event.is_set()` — returns current flag status without blocking.

```python
import threading
import time

server_ready = threading.Event()


def client(client_id: int) -> None:
    print(f"Client {client_id} waiting for server...")
    server_ready.wait()
    print(f"Client {client_id} connected!")


def start_server() -> None:
    print("Server initializing...")
    time.sleep(1.0)
    print("Server ready!")
    server_ready.set()


clients = [threading.Thread(target=client, args=(i,)) for i in range(3)]
for c in clients:
    c.start()

threading.Thread(target=start_server).start()
for c in clients:
    c.join()
```

### 5. `threading.Condition`
A `Condition` combines a lock (`Lock` or `RLock`) with a notification queue (`wait`, `notify`, `notify_all`). It lets threads wait until a specific application-level state condition becomes true.

> **Important**: Always check the condition inside a **`while` loop**, not an `if` statement, to protect against spurious wakeups or another thread consuming the item before the woken thread runs.

```python
import threading
import time

items: list[int] = []
condition = threading.Condition()


def consumer() -> None:
    with condition:
        while not items:
            print("Consumer waiting for items...")
            condition.wait()
        item = items.pop(0)
        print(f"Consumer processed: {item}")


def producer() -> None:
    time.sleep(0.5)
    with condition:
        items.append(42)
        print("Producer added: 42")
        condition.notify()  # Wake up one waiting thread (or notify_all())


t_con = threading.Thread(target=consumer)
t_pro = threading.Thread(target=producer)
t_con.start()
t_pro.start()
t_con.join()
t_pro.join()
```

---

## 7. Thread-Safe Data Structures (`queue`)

Sharing plain `list` or `dict` structures across threads requires manual locking for compound operations. Python's `queue` module provides **thread-safe, synchronized queues** with built-in locking and blocking semantics.

### Queue Types
- `queue.Queue(maxsize=0)` — FIFO (First-In, First-Out).
- `queue.LifoQueue(maxsize=0)` — LIFO (stack).
- `queue.PriorityQueue(maxsize=0)` — Retrieves lowest-valued entries first (often tuples like `(priority, data)`).

### Key Methods
- `q.put(item, block=True, timeout=None)` — Adds an item; blocks if `maxsize` is reached (raises `queue.Full` if non-blocking/timed out).
- `q.get(block=True, timeout=None)` — Removes and returns an item; blocks if empty (raises `queue.Empty` if non-blocking/timed out).
- `q.task_done()` — Signals that a previously retrieved item has been fully processed.
- `q.join()` — Blocks until all items put into the queue have been retrieved **and** marked with `task_done()`.

```python
import queue
import threading
import time

task_queue: queue.Queue[str | None] = queue.Queue(maxsize=5)


def worker(worker_id: int) -> None:
    while True:
        item = task_queue.get()
        if item is None:  # Sentinel value signals shutdown
            task_queue.task_done()
            break
        print(f"Worker {worker_id} processing {item}")
        time.sleep(0.2)
        task_queue.task_done()


workers = [threading.Thread(target=worker, args=(i,)) for i in range(2)]
for w in workers:
    w.start()

for job_id in range(1, 7):
    task_queue.put(f"job-{job_id}")

# Wait until all jobs are processed
task_queue.join()

# Send one sentinel per worker to stop them cleanly
for _ in workers:
    task_queue.put(None)

for w in workers:
    w.join()

print("All jobs finished cleanly.")
```

---

## 8. The `multiprocessing` Module Basics

The `multiprocessing` module spawns **separate OS processes**, each with its own Python interpreter and private memory space. Because each process has its own GIL, `multiprocessing` achieves **true multi-core parallelism** for CPU-bound Python code.

### Key Concepts
1. **Isolated Memory**: Modifying a global variable in a child process does **not** affect the parent process or sibling processes.
2. **Higher Overhead**: Spawning processes takes more time and RAM than threads, and passing objects between processes requires serialization via **`pickle`**.
3. **The `if __name__ == "__main__":` Guard**:
   On Windows and macOS (and on Linux when using `spawn` or `forkserver`), child processes import the main script to locate the target function. Without `if __name__ == "__main__":`, the child process would recursively spawn infinite child processes and crash. **Always protect entry-point process creation with this guard.**

### Process Start Methods
You can inspect or configure how processes start via `multiprocessing.set_start_method()` or `multiprocessing.get_context()`:
- **`spawn`** (default on Windows & macOS; default changing from `fork` to `forkserver`/`spawn` in Python 3.14 on POSIX): Starts a fresh Python interpreter process. Slowest, but safest with threads and libraries.
- **`fork`** (POSIX/Linux): Clones the parent process using OS `fork()`. Fast, inherits memory copy-on-write, **unsafe** if the parent already created threads or locks.
- **`forkserver`** (POSIX): Starts a clean single-threaded server process that forks new workers on request. Safe and faster than `spawn`.

```python
import multiprocessing
import os


def show_info() -> None:
    print(f"Process name: {multiprocessing.current_process().name}, PID: {os.getpid()}")


if __name__ == "__main__":
    print(f"CPU cores available: {multiprocessing.cpu_count()}")
    print(f"Default start method: {multiprocessing.get_start_method()}")
    show_info()
```

---

## 9. The `Process` Class

`multiprocessing.Process` mirrors the `threading.Thread` API (`start()`, `join()`, `is_alive()`, `daemon`, `name`, `pid`), making it straightforward to migrate between threads and processes.

Additional process-specific methods:
- `p.pid` — OS process ID (assigned after `start()`).
- `p.exitcode` — `None` while running, `0` on clean exit, negative `-N` if terminated by signal `N`.
- `p.terminate()` / `p.kill()` — Terminates the process (use `join()` afterward to clean up zombies).

```python
import multiprocessing
import os
import time


def compute_square(number: int) -> None:
    pid = os.getpid()
    print(f"[PID {pid}] Computing square of {number}")
    time.sleep(0.3)
    print(f"[PID {pid}] {number}^2 = {number ** 2}")


if __name__ == "__main__":
    processes: list[multiprocessing.Process] = []
    for num in [2, 4, 6, 8]:
        p = multiprocessing.Process(target=compute_square, args=(num,), name=f"Proc-{num}")
        p.start()
        processes.append(p)

    for p in processes:
        p.join()
        print(f"{p.name} exited with code {p.exitcode}")
```

---

## 10. `Pool` for Parallel Execution

Managing individual `Process` instances manually can be tedious when processing collections of data. `multiprocessing.Pool` manages a fixed pool of worker processes and distributes tasks to them.

> **Best Practice**: Always use `with multiprocessing.Pool(...) as pool:` so worker processes are properly closed and joined when exiting the block.

### Core `Pool` Methods

| Method | Execution | Arguments | Blocking? | Preserves Input Order? |
| :--- | :--- | :--- | :--- | :--- |
| `pool.map(func, iterable)` | Multiple items | Single arg per call | Yes | Yes |
| `pool.starmap(func, iterable)` | Multiple items | Unpacks tuple of args (`func(*args)`) | Yes | Yes |
| `pool.imap(func, iterable)` | Lazy iterator | Single arg per call | Yields results lazily | Yes |
| `pool.apply(func, args)` | Single call | `args` tuple, `kwargs` dict | Yes | N/A |
| `pool.apply_async(func, args)` | Single call | `args` tuple, `kwargs` dict | No (returns `AsyncResult`) | N/A |
| `pool.map_async(func, iterable)` | Multiple items | Single arg per call | No (returns `AsyncResult`) | Yes |

```python
import multiprocessing
import time


def heavy_square(x: int) -> int:
    time.sleep(0.1)
    return x * x


def power(base: int, exp: int) -> int:
    return base ** exp


if __name__ == "__main__":
    numbers = [1, 2, 3, 4, 5, 6]

    with multiprocessing.Pool(processes=4) as pool:
        # 1. pool.map: blocks and returns ordered list
        squares = pool.map(heavy_square, numbers)
        print("map:", squares)

        # 2. pool.starmap: unpacks tuples as multiple arguments
        powers = pool.starmap(power, [(2, 3), (3, 2), (5, 3)])
        print("starmap:", powers)

        # 3. pool.apply: single synchronous call in a pool worker
        single = pool.apply(power, args=(10, 3))
        print("apply:", single)

        # 4. pool.apply_async: non-blocking, returns AsyncResult
        async_res = pool.apply_async(heavy_square, args=(12,))
        async_res2 = pool.apply_async(
            power,
            args=(2, 10),
            callback=lambda res: print(f"Callback received: {res}"),
        )
        print("Doing other work while async tasks run...")
        print("apply_async result:", async_res.get(timeout=5))
        async_res2.wait()
```

---

## 11. Sharing Data Between Processes

Because processes do not share memory by default, `multiprocessing` provides explicit mechanisms for sharing state and exchanging messages.

### 1. Shared Memory: `Value` and `Array`
`Value` and `Array` allocate shared C-compatible memory segments using type codes (`'i'` for signed int, `'d'` for double/float, `'b'` for byte, etc.). They come with a built-in lock (`get_lock()`), which you should use for compound read-modify-write operations like `+=`.

```python
import multiprocessing


def update_shared(shared_num, shared_arr) -> None:
    with shared_num.get_lock():
        shared_num.value += 1

    with shared_arr.get_lock():
        for i in range(len(shared_arr)):
            shared_arr[i] *= 2


if __name__ == "__main__":
    num = multiprocessing.Value("i", 0)
    arr = multiprocessing.Array("d", [1.0, 2.0, 3.0])

    procs = [multiprocessing.Process(target=update_shared, args=(num, arr)) for _ in range(3)]
    for p in procs:
        p.start()
    for p in procs:
        p.join()

    print("Shared Value:", num.value)    # 3
    print("Shared Array:", list(arr))    # [8.0, 16.0, 24.0]
```

### 2. Server Process Objects: `Manager`
`multiprocessing.Manager()` launches a separate server process that hosts Python objects (`list`, `dict`, `Namespace`, `Lock`, etc.) and exposes proxy objects to workers.
- **Pros**: Supports arbitrary Python types (`dict`, nested lists).
- **Cons**: Slower than `Value`/`Array` because every access involves IPC (inter-process communication) and pickling.

> **Gotcha with `Manager` nested structures**: Mutating a nested mutable object directly (e.g., `shared_dict["items"].append(1)`) may not propagate back to the manager unless you reassign `shared_dict["items"] = ...`.

```python
import multiprocessing


def record_worker(shared_dict, shared_list, worker_id: int) -> None:
    shared_dict[f"worker_{worker_id}"] = worker_id * 10
    shared_list.append(worker_id)


if __name__ == "__main__":
    with multiprocessing.Manager() as manager:
        d = manager.dict()
        lst = manager.list()

        procs = [
            multiprocessing.Process(target=record_worker, args=(d, lst, i))
            for i in range(4)
        ]
        for p in procs:
            p.start()
        for p in procs:
            p.join()

        print("Manager dict:", dict(d))
        print("Manager list:", list(lst))
```

### 3. Message Passing: `multiprocessing.Queue` and `Pipe`
Prefer passing messages over sharing mutable state whenever possible:
- `multiprocessing.Queue`: Multi-producer, multi-consumer FIFO queue (pickles items over a pipe).
- `multiprocessing.Pipe()`: Fast two-way connection between two endpoints (`parent_conn, child_conn = Pipe()`).

```python
import multiprocessing


def producer(q: multiprocessing.Queue) -> None:
    for item in ["task-A", "task-B", "task-C"]:
        q.put(item)


if __name__ == "__main__":
    q: multiprocessing.Queue = multiprocessing.Queue()
    p = multiprocessing.Process(target=producer, args=(q,))
    p.start()
    p.join()

    while not q.empty():
        print("Received:", q.get())
```

---

## 12. `concurrent.futures`

The `concurrent.futures` module provides a **unified, high-level interface** for executing callables asynchronously using either threads (`ThreadPoolExecutor`) or processes (`ProcessPoolExecutor`). Switching between threads and processes often requires changing just one class name!

### Core Concepts
- **`Executor.submit(fn, *args, **kwargs)`**: Schedules one task and immediately returns a **`Future`** object.
- **`Executor.map(func, *iterables, timeout=None)`**: Like built-in `map()`, runs in parallel and returns results in **input order** (raises exceptions when iterating over the result).
- **`Future` object**: Represents an eventual result:
  - `future.result(timeout=None)` — blocks until complete and returns the value (or re-raises any exception raised inside the task).
  - `future.done()` — `True` if completed or cancelled.
  - `future.add_done_callback(fn)` — registers a callback when finished.
- **`concurrent.futures.as_completed(futures)`**: Yields `Future` objects **as soon as each one finishes** (completion order, not submission order).
- **`concurrent.futures.wait(futures, return_when=ALL_COMPLETED)`**: Waits for all, first completed, or first exception.

### Example 1: `ThreadPoolExecutor` (I/O-Bound)

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time


def fetch_page(page_id: int) -> str:
    delay = 0.5 if page_id % 2 == 0 else 0.1
    time.sleep(delay)
    if page_id == 3:
        raise ValueError(f"Failed to fetch page {page_id}")
    return f"Content of page {page_id}"


with ThreadPoolExecutor(max_workers=4) as executor:
    future_to_id = {executor.submit(fetch_page, pid): pid for pid in range(1, 5)}

    for future in as_completed(future_to_id):
        pid = future_to_id[future]
        try:
            data = future.result()
            print(f"Page {pid}: {data}")
        except Exception as exc:
            print(f"Page {pid} raised an error: {exc}")
```

### Example 2: `ProcessPoolExecutor` (CPU-Bound)

```python
from concurrent.futures import ProcessPoolExecutor


def sum_of_squares(n: int) -> int:
    return sum(i * i for i in range(n))


if __name__ == "__main__":
    inputs = [1_000_000, 2_000_000, 3_000_000, 4_000_000]

    with ProcessPoolExecutor() as executor:  # Defaults to os.cpu_count()
        for n, result in zip(inputs, executor.map(sum_of_squares, inputs)):
            print(f"sum_of_squares({n}) = {result}")
```

---

## 13. `asyncio` Basics

`asyncio` is Python's standard framework for **asynchronous I/O** using **cooperative multitasking** inside a **single thread**.

### How It Works
- Instead of the OS forcibly preempting threads, functions explicitly yield control using `await` whenever they wait on I/O (network, timers, async DB drivers).
- While one task is suspended at `await`, the **event loop** switches to another ready task.
- Because everything runs on one thread, there is **no OS thread-switching overhead** and **far fewer race conditions** on shared in-memory state (though races across `await` boundaries are still possible!).

### Key Terms
- **Coroutine function**: Defined with `async def`. Calling it returns a **coroutine object** without executing its body yet.
- **`await`**: Suspends the current coroutine until the awaited *awaitable* (coroutine, `Task`, or `Future`) completes. Can only be used inside an `async def` function.
- **Event loop**: The scheduler that drives coroutines, handles I/O readiness notifications, and runs callbacks.
- **`asyncio.run(coro)`**: Entry point (Python 3.7+) that creates a new event loop, runs the top-level coroutine until completion, and cleanly closes the loop.

```python
import asyncio
import time


async def greet(name: str, delay: float) -> str:
    print(f"Starting greeting for {name}")
    await asyncio.sleep(delay)  # Non-blocking sleep (NEVER use time.sleep here!)
    print(f"Finished greeting for {name}")
    return f"Hello, {name}!"


async def main() -> None:
    start = time.perf_counter()

    # Sequential awaiting: takes 1.0 + 1.0 = ~2.0s
    r1 = await greet("Alice", 1.0)
    r2 = await greet("Bob", 1.0)
    print(r1, r2)
    print(f"Sequential await took {time.perf_counter() - start:.2f}s")


asyncio.run(main())
```

> **Crucial Rule**: Never call blocking functions like `time.sleep()`, `requests.get()`, or synchronous file I/O directly inside an `async` coroutine—they freeze the entire event loop. Use async equivalents (`asyncio.sleep()`, `aiohttp`/`httpx`) or offload blocking calls with `await asyncio.to_thread(blocking_fn, *args)` (Python 3.9+).

---

## 14. `asyncio.gather` and `asyncio.create_task`

Merely writing `await coro()` runs coroutines **one after another**. To run coroutines **concurrently**, wrap them as **Tasks** or group them with `asyncio.gather()` / `asyncio.TaskGroup`.

### `asyncio.create_task()`
Schedules a coroutine to run concurrently on the event loop immediately and returns a `Task` object.
- Keep a strong reference to created tasks (e.g., in a variable or set) so they aren't garbage-collected mid-execution.
- Supports `task.cancel()`, `task.done()`, and `await task`.

### `asyncio.gather()`
Takes multiple awaitables, schedules any raw coroutines as tasks concurrently, waits for all of them, and returns a list of results in the **same order** as the inputs.
- `return_exceptions=False` (default): If any task raises an exception, `gather` immediately propagates it (other tasks continue running in the background unless cancelled).
- `return_exceptions=True`: Exceptions are returned as items in the result list instead of raising.

```python
import asyncio
import time


async def fetch_data(source: str, delay: float) -> dict[str, str]:
    print(f"Fetching {source}...")
    await asyncio.sleep(delay)
    print(f"Done {source}")
    return {"source": source, "status": "ok"}


async def main() -> None:
    start = time.perf_counter()

    # Approach 1: create_task() for fine-grained control
    task1 = asyncio.create_task(fetch_data("API-1", 1.0))
    task2 = asyncio.create_task(fetch_data("API-2", 0.5))
    print("Both tasks scheduled; doing local setup...")
    res1 = await task1
    res2 = await task2
    print("create_task results:", res1, res2)

    # Approach 2: asyncio.gather() for running a batch concurrently
    results = await asyncio.gather(
        fetch_data("DB-1", 0.8),
        fetch_data("DB-2", 0.8),
        fetch_data("Cache", 0.3),
    )
    print("gather results:", results)
    print(f"Total elapsed: {time.perf_counter() - start:.2f}s")  # ~1.8s total, not 3.4s


asyncio.run(main())
```

### Modern Alternative: `asyncio.TaskGroup` (Python 3.11+)
`TaskGroup` provides **structured concurrency**: if any task inside the `async with` block fails, all remaining tasks in the group are automatically cancelled and exceptions are raised as an `ExceptionGroup`.

```python
import asyncio


async def job(i: int) -> int:
    await asyncio.sleep(0.2)
    return i * 10


async def main() -> None:
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(job(i)) for i in range(1, 4)]

    print([t.result() for t in tasks])  # [10, 20, 30]


asyncio.run(main())
```

---

## 15. When to Use `threading` vs `multiprocessing` vs `asyncio`

| Criteria | `threading` | `multiprocessing` | `asyncio` |
| :--- | :--- | :--- | :--- |
| **Workload Type** | Blocking **I/O-bound** | **CPU-bound** | High-concurrency **I/O-bound** |
| **Execution Model** | Preemptive (OS threads) | Parallel (OS processes) | Cooperative (single-threaded event loop) |
| **Bypass CPython GIL?** | No | **Yes** | No (not needed—single thread) |
| **Memory Sharing** | Shared heap (needs locks) | Isolated (needs IPC/shared memory) | Single thread (no locks for non-awaited blocks) |
| **Memory & Startup Cost** | Medium | High | Very low |
| **Scale Limit** | Hundreds to a few thousand threads | Roughly `os.cpu_count()` workers | Tens of thousands of concurrent connections |
| **Library Ecosystem** | Works with all standard blocking libraries (`requests`, `sqlite3`) | Works with picklable functions/data | Requires async-native libraries (`aiohttp`, `asyncpg`) |

### Quick Decision Guide
1. **Is the task CPU-bound** (image processing, ML inference, parsing huge files, heavy math in pure Python)?
   -> Use **`multiprocessing`** or **`ProcessPoolExecutor`** (or vectorized C libraries like NumPy).
2. **Is the task I/O-bound with high concurrency** (web servers, WebSockets, scraping 5,000 URLs)?
   -> Use **`asyncio`**.
3. **Is the task I/O-bound using synchronous/blocking libraries** (calling `requests`, legacy DB drivers, file downloads with 10–50 workers)?
   -> Use **`threading`** or **`ThreadPoolExecutor`**.
4. **Hybrid workloads** (an `asyncio` web server that occasionally does CPU-heavy work or calls a blocking library)?
   -> Combine them! Use `await asyncio.to_thread(...)` for blocking I/O, or `loop.run_in_executor(process_pool, cpu_func, ...)` for CPU work.

---

## 16. Common Concurrency Patterns and Pitfalls

### Common Patterns

#### 1. Producer–Consumer
Decouples work generation from work processing using a bounded `queue.Queue` (or `asyncio.Queue`). Prevents fast producers from exhausting memory via backpressure (`maxsize`).

#### 2. Fan-Out / Fan-In
Distribute a list of independent tasks across a pool (`ThreadPoolExecutor` / `ProcessPoolExecutor` / `asyncio.gather`) and aggregate results once all finish.

#### 3. Graceful Shutdown with `Event` or Sentinels
Use a `threading.Event` (`stop_event.is_set()`) or sentinel values (`None`) in queues instead of abruptly killing threads/processes.

---

### Common Pitfalls (and How to Avoid Them)

#### 1. Race Conditions (Non-Atomic Operations)
Even `counter += 1` is **not atomic** in Python—it compiles to multiple bytecode instructions (load, add, store). Two threads can read the same value and overwrite each other.
- **Fix**: Wrap shared mutations in a `threading.Lock()` or use thread-safe structures like `queue.Queue`.

#### 2. Deadlocks
Occurs when Thread A holds Lock 1 and waits for Lock 2, while Thread B holds Lock 2 and waits for Lock 1.
- **Fix**:
  - Always acquire multiple locks in a **consistent global order**.
  - Always use `with lock:` so exceptions never leave a lock acquired.
  - Use timeouts (`lock.acquire(timeout=...)`) where appropriate.

#### 3. Blocking the `asyncio` Event Loop
Calling `time.sleep()`, `requests.get()`, or heavy CPU loops inside an `async def` stalls every single concurrent coroutine.
- **Fix**: Replace blocking calls with `await asyncio.sleep()`, async libraries, or `await asyncio.to_thread(...)`.

#### 4. Forgetting `if __name__ == "__main__":` in `multiprocessing`
Causes infinite recursive process spawning on `spawn`/`forkserver` platforms (Windows, macOS, newer Linux defaults) with `RuntimeError`.
- **Fix**: Always wrap process/pool creation inside `if __name__ == "__main__":`.

#### 5. Unpicklable Objects in `multiprocessing`
Passing `lambda` functions, local nested functions, open sockets, locks, or generators to `Process` or `Pool` fails with `PicklingError` or `AttributeError`.
- **Fix**: Define worker functions at the **module top level**, use `functools.partial` instead of `lambda`, and pass serializable arguments (IDs/paths rather than open handles).

#### 6. Passing `func()` Instead of `func` to `Thread` / `Process` / `submit`
Writing `Thread(target=worker())` executes `worker()` immediately on the main thread and passes its return value as `target`.
- **Fix**: Pass the function reference without parentheses: `Thread(target=worker, args=(...,))`.

#### 7. Ignoring Exceptions in Threads or Background Tasks
An uncaught exception in a `Thread` prints a traceback to `stderr` but does **not** crash the main thread; in `asyncio.create_task()`, exceptions are silent until the task is awaited or garbage-collected.
- **Fix**: Use `concurrent.futures` (`future.result()` re-raises exceptions), `asyncio.TaskGroup`, or explicitly `await` / attach done callbacks to tasks.

#### 8. Dropping References to `asyncio.create_task()`
`asyncio` only holds weak references to running tasks. Creating a fire-and-forget task without storing it can cause the garbage collector to destroy it mid-flight!
- **Fix**: Keep a set of active tasks and remove them via a done callback:

```python
import asyncio

background_tasks: set[asyncio.Task] = set()


def spawn_background(coro) -> asyncio.Task:
    task = asyncio.create_task(coro)
    background_tasks.add(task)
    task.add_done_callback(background_tasks.discard)
    return task
```
