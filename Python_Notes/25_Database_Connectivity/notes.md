# Database Connectivity in Python

Working with databases is a core skill for backend engineering, data processing, automation, and application development. Python provides a standardized relational database interface (**DB-API 2.0**), a built-in database engine (**`sqlite3`**), dedicated drivers for enterprise RDBMSs (**MySQL**, **PostgreSQL**), powerful **ORMs** (**SQLAlchemy**), and drivers for NoSQL document stores (**MongoDB** via **PyMongo**).

---

## Table of Contents

1. [Python DB-API 2.0 Overview](#1-python-db-api-20-overview)
2. [SQLite with the `sqlite3` Module](#2-sqlite-with-the-sqlite3-module)
3. [Connecting to a Database & Creating a Cursor](#3-connecting-to-a-database--creating-a-cursor)
4. [Creating Tables](#4-creating-tables)
5. [CRUD Operations (`INSERT`, `SELECT`, `UPDATE`, `DELETE`)](#5-crud-operations-insert-select-update-delete)
6. [Parameterized Queries (Preventing SQL Injection)](#6-parameterized-queries-preventing-sql-injection)
7. [Fetching Data (`fetchone`, `fetchall`, `fetchmany`)](#7-fetching-data-fetchone-fetchall-fetchmany)
8. [Transaction Management (`commit`, `rollback`)](#8-transaction-management-commit-rollback)
9. [Context Managers with Databases](#9-context-managers-with-databases)
10. [MySQL Connectivity (`mysql-connector-python`)](#10-mysql-connectivity-mysql-connector-python)
11. [PostgreSQL Connectivity (`psycopg2`) Overview](#11-postgresql-connectivity-psycopg2-overview)
12. [ORM Concepts — SQLAlchemy Basics](#12-orm-concepts--sqlalchemy-basics)
13. [MongoDB with PyMongo Overview](#13-mongodb-with-pymongo-overview)
14. [Connection Pooling](#14-connection-pooling)
15. [Best Practices for Database Operations](#15-best-practices-for-database-operations)

---

## 1. Python DB-API 2.0 Overview

**PEP 249** defines the **Python Database API Specification v2.0 (DB-API 2.0)**. Its goal is to provide a consistent API across different relational database drivers (`sqlite3`, `psycopg2`, `mysql-connector-python`, `oracledb`, etc.) so switching databases requires minimal changes to application logic.

### Core Components

| Component | Purpose |
| :--- | :--- |
| `connect(...)` | Module-level constructor that opens a session and returns a `Connection` object. |
| `Connection` | Manages the database session, transactions (`commit()`, `rollback()`), and creates cursors (`cursor()`). |
| `Cursor` | Executes SQL statements (`execute()`, `executemany()`) and retrieves result rows (`fetchone()`, `fetchmany()`, `fetchall()`). |
| `apilevel` | String constant indicating DB-API version (`"2.0"`). |
| `threadsafety` | Integer (`0`–`3`) indicating how safe the module is across threads. |
| `paramstyle` | Placeholder syntax expected by the driver (`qmark`, `numeric`, `named`, `format`, `pyformat`). |

### Standard Exception Hierarchy

```text
Exception
 └── Warning
 └── Error
      ├── InterfaceError      (driver/API misuse)
      └── DatabaseError       (database-side issues)
           ├── DataError          (bad values, overflow, division by zero)
           ├── OperationalError   (disconnects, missing DB/table, locks)
           ├── IntegrityError     (PK/FK/UNIQUE/CHECK constraint violations)
           ├── InternalError      (internal DB error, invalid cursor)
           ├── ProgrammingError   (SQL syntax error, wrong param count)
           └── NotSupportedError  (unsupported feature/API)
```

### Parameter Styles (`paramstyle`)

| Style | Syntax Example | Used By |
| :--- | :--- | :--- |
| `qmark` | `WHERE id = ?` | `sqlite3`, `pyodbc` |
| `named` | `WHERE id = :user_id` | `sqlite3`, `oracledb` |
| `format` | `WHERE id = %s` | `mysql-connector-python`, `psycopg2` |
| `pyformat` | `WHERE id = %(user_id)s` | `psycopg2`, `mysql-connector-python` |
| `numeric` | `WHERE id = :1` | `oracledb` |

```python
import sqlite3

print("API Level:", sqlite3.apilevel)        # '2.0'
print("Thread Safety:", sqlite3.threadsafety)
print("Param Style:", sqlite3.paramstyle)    # 'qmark'
```

---

## 2. SQLite with the `sqlite3` Module

**SQLite** is an embedded, serverless, zero-configuration SQL database engine that stores an entire database in a single cross-platform file (or in RAM). Python includes the `sqlite3` module in the **standard library**, making it ideal for prototyping, local tools, CLI apps, desktop software, caching, and testing.

### Key Characteristics of `sqlite3`

- **No separate server process**: Reads/writes directly to disk.
- **In-memory mode**: Passing `":memory:"` creates a fast, ephemeral database that vanishes when the connection closes.
- **Dynamic typing (type affinity)**: SQLite uses storage classes (`NULL`, `INTEGER`, `REAL`, `TEXT`, `BLOB`).
- **Row factories**: Setting `conn.row_factory = sqlite3.Row` allows accessing columns by name as well as index.

```python
import sqlite3

# File-backed database (created automatically if it does not exist)
disk_conn = sqlite3.connect("app.db")

# Ephemeral in-memory database (great for unit tests)
mem_conn = sqlite3.connect(":memory:")

# Enable dictionary-like column access by name
mem_conn.row_factory = sqlite3.Row

disk_conn.close()
mem_conn.close()
```

> **Note:** In SQLite, foreign key constraints are parsed by default but **not enforced** unless enabled per connection with `PRAGMA foreign_keys = ON;`.

---

## 3. Connecting to a Database & Creating a Cursor

Every DB-API workflow follows the same lifecycle:

1. **Connect** to the database (`Connection`).
2. **Create a cursor** (`Cursor`).
3. **Execute** SQL queries.
4. **Fetch** results (for `SELECT`) or **commit** changes (for writes).
5. **Close** the cursor and connection.

```python
import sqlite3

# 1. Establish a connection
conn = sqlite3.connect("company.db")

# 2. Create a cursor object
cursor = conn.cursor()

# 3. Execute a simple diagnostic query
cursor.execute("SELECT sqlite_version();")
version = cursor.fetchone()
print("SQLite version:", version[0])

# 4. Always clean up resources
cursor.close()
conn.close()
```

### Useful `sqlite3.connect()` Parameters

- `database`: File path (`"company.db"`), `pathlib.Path`, or `":memory:"`.
- `timeout`: Seconds to wait for a database lock to clear before raising `OperationalError` (default `5.0`).
- `detect_types`: Enables automatic conversion for `DATE` and `TIMESTAMP` columns (`sqlite3.PARSE_DECLTYPES | sqlite3.PARSE_COLNAMES`).
- `isolation_level`: Controls transaction behavior (`""` / `"DEFERRED"` / `"IMMEDIATE"` / `"EXCLUSIVE"` or `None` for autocommit).

---

## 4. Creating Tables

Use `CREATE TABLE IF NOT EXISTS` so initialization scripts are **idempotent** (safe to run multiple times). Define clear primary keys, data types, and constraints (`NOT NULL`, `UNIQUE`, `CHECK`, `DEFAULT`, `FOREIGN KEY`).

```python
import sqlite3

conn = sqlite3.connect("store.db")
cursor = conn.cursor()

# Enable foreign key enforcement for this connection
cursor.execute("PRAGMA foreign_keys = ON;")

# Create parent table: customers
cursor.execute("""
    CREATE TABLE IF NOT EXISTS customers (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        full_name TEXT NOT NULL,
        email TEXT NOT NULL UNIQUE,
        created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
    );
""")

# Create child table: orders with foreign key and check constraint
cursor.execute("""
    CREATE TABLE IF NOT EXISTS orders (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        customer_id INTEGER NOT NULL,
        product_name TEXT NOT NULL,
        quantity INTEGER NOT NULL CHECK (quantity > 0),
        total_price REAL NOT NULL CHECK (total_price >= 0),
        FOREIGN KEY (customer_id) REFERENCES customers(id) ON DELETE CASCADE
    );
""")

conn.commit()
cursor.close()
conn.close()
```

### Multi-Statement Scripts with `executescript()`

For schema initialization scripts containing multiple semicolon-separated statements, `sqlite3` provides `cursor.executescript()` (note: it issues an implicit `COMMIT` first and does not accept query parameters):

```python
with sqlite3.connect("store.db") as conn:
    conn.executescript("""
        PRAGMA foreign_keys = ON;
        CREATE INDEX IF NOT EXISTS idx_orders_customer_id
            ON orders(customer_id);
    """)
```

---

## 5. CRUD Operations (`INSERT`, `SELECT`, `UPDATE`, `DELETE`)

CRUD maps directly to the four fundamental SQL data operations:

| Operation | SQL Statement | Typical Cursor Attributes Used |
| :--- | :--- | :--- |
| **Create** | `INSERT INTO ...` | `cursor.lastrowid`, `cursor.rowcount` |
| **Read** | `SELECT ... FROM ...` | `fetchone()`, `fetchmany()`, `fetchall()` |
| **Update** | `UPDATE ... SET ... WHERE ...` | `cursor.rowcount` |
| **Delete** | `DELETE FROM ... WHERE ...` | `cursor.rowcount` |

```python
import sqlite3

conn = sqlite3.connect(":memory:")
conn.row_factory = sqlite3.Row
cursor = conn.cursor()

cursor.execute("""
    CREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT NOT NULL UNIQUE,
        role TEXT NOT NULL DEFAULT 'member',
        active INTEGER NOT NULL DEFAULT 1
    );
""")

# --- CREATE (Single Insert) ---
cursor.execute(
    "INSERT INTO users (username, role) VALUES (?, ?)",
    ("alice", "admin"),
)
print("Inserted user ID:", cursor.lastrowid)

# --- CREATE (Bulk Insert with executemany) ---
new_users = [
    ("bob", "member"),
    ("charlie", "editor"),
    ("diana", "member"),
]
cursor.executemany(
    "INSERT INTO users (username, role) VALUES (?, ?)",
    new_users,
)
conn.commit()
print("Bulk inserted rows:", cursor.rowcount)

# --- READ ---
cursor.execute("SELECT id, username, role FROM users WHERE active = ?", (1,))
for row in cursor.fetchall():
    print(f"ID={row['id']} | user={row['username']} | role={row['role']}")

# --- UPDATE ---
cursor.execute(
    "UPDATE users SET role = ? WHERE username = ?",
    ("admin", "charlie"),
)
conn.commit()
print("Updated rows:", cursor.rowcount)

# --- DELETE ---
cursor.execute("DELETE FROM users WHERE username = ?", ("diana",))
conn.commit()
print("Deleted rows:", cursor.rowcount)

cursor.close()
conn.close()
```

---

## 6. Parameterized Queries (Preventing SQL Injection)

**Never** build SQL queries using string formatting (`f"..."`, `%`, `+`, or `.format()`). Untrusted input can break out of quotes and alter the SQL query structure (**SQL injection**).

### Vulnerable vs. Safe Queries

```python
import sqlite3

conn = sqlite3.connect(":memory:")
cursor = conn.cursor()
cursor.execute("CREATE TABLE accounts (id INTEGER PRIMARY KEY, username TEXT, balance REAL)")
cursor.execute("INSERT INTO accounts (username, balance) VALUES ('alice', 5000.0)")
conn.commit()

user_input = "' OR '1'='1"

# BAD / VULNERABLE: String interpolation treats input as executable SQL!
bad_query = f"SELECT * FROM accounts WHERE username = '{user_input}'"
cursor.execute(bad_query)
print("Vulnerable query returned:", cursor.fetchall())  # Leaks all rows!

# GOOD / SAFE: Positional parameter binding (qmark style)
cursor.execute("SELECT * FROM accounts WHERE username = ?", (user_input,))
print("Safe positional query returned:", cursor.fetchall())  # []

# GOOD / SAFE: Named parameter binding (named style)
cursor.execute(
    "SELECT * FROM accounts WHERE username = :name AND balance >= :min_bal",
    {"name": "alice", "min_bal": 100.0},
)
print("Safe named query returned:", cursor.fetchall())

conn.close()
```

### Important Caveats

1. **Single-element tuple syntax**: Always include a trailing comma for a 1-tuple: `(value,)`, not `(value)`.
2. **Parameters only work for values**, not for table names, column names, or SQL keywords (`ORDER BY`, `ASC`/`DESC`). If you must choose a column or table dynamically, validate it against an explicit allowlist:

```python
ALLOWED_SORT_COLUMNS = {"username", "balance"}

def get_accounts_sorted(cursor, sort_by: str):
    if sort_by not in ALLOWED_SORT_COLUMNS:
        raise ValueError(f"Invalid sort column: {sort_by}")
    # Safe because sort_by is strictly validated against a fixed allowlist
    return cursor.execute(f"SELECT id, username, balance FROM accounts ORDER BY {sort_by}").fetchall()
```

---

## 7. Fetching Data (`fetchone`, `fetchall`, `fetchmany`)

Once a `SELECT` query runs, the cursor acts as a stateful iterator over the result set.

| Method | Return Value | Best Used When |
| :--- | :--- | :--- |
| `fetchone()` | Single row (`tuple`/`Row`), or `None` when exhausted | Looking up 1 record (e.g., by primary key, `COUNT(*)`) |
| `fetchmany(size)` | List of up to `size` rows (`[]` when exhausted) | Processing large tables in batches without exhausting RAM |
| `fetchall()` | List of all remaining rows (`[]` if none) | Small-to-medium result sets needed all at once |
| `for row in cursor:` | Streams one row at a time | Memory-efficient row-by-row processing |

```python
import sqlite3

conn = sqlite3.connect(":memory:")
conn.row_factory = sqlite3.Row  # Enables row["column"] and dict(row)
cursor = conn.cursor()

cursor.execute("CREATE TABLE items (id INTEGER PRIMARY KEY, name TEXT, price REAL)")
cursor.executemany(
    "INSERT INTO items (name, price) VALUES (?, ?)",
    [(f"Item {i}", i * 2.5) for i in range(1, 11)],
)
conn.commit()

# 1. fetchone(): Get a single row
cursor.execute("SELECT * FROM items WHERE id = ?", (3,))
one_row = cursor.fetchone()
if one_row:
    print("fetchone:", dict(one_row))

# 2. fetchmany(size): Stream in batches of 3
cursor.execute("SELECT * FROM items ORDER BY id")
while True:
    batch = cursor.fetchmany(3)
    if not batch:
        break
    print("fetchmany batch:", [row["name"] for row in batch])

# 3. fetchall(): Load all rows into a list
cursor.execute("SELECT name, price FROM items WHERE price > ?", (15.0,))
expensive_items = cursor.fetchall()
print("fetchall count:", len(expensive_items))

# 4. Direct cursor iteration (memory-efficient)
cursor.execute("SELECT name FROM items LIMIT 3")
for row in cursor:
    print("iterated:", row["name"])

conn.close()
```

> **Tip:** `fetchone()` / `fetchmany()` / `fetchall()` consume rows from the cursor buffer. Calling `fetchall()` after iterating over the cursor returns `[]`.

---

## 8. Transaction Management (`commit`, `rollback`)

A **transaction** groups one or more SQL operations into a single atomic unit of work adhering to **ACID** properties (**A**tomicity, **C**onsistency, **I**solation, **D**urability):

- **`conn.commit()`**: Persists all pending changes made during the current transaction.
- **`conn.rollback()`**: Reverts all pending changes since the last `commit()`, restoring the database to its previous consistent state.

### Explicit Transaction Pattern (`try / except / finally`)

```python
import sqlite3

conn = sqlite3.connect(":memory:")
cursor = conn.cursor()
cursor.execute("CREATE TABLE bank (owner TEXT PRIMARY KEY, balance REAL CHECK (balance >= 0))")
cursor.executemany("INSERT INTO bank VALUES (?, ?)", [("Alice", 1000.0), ("Bob", 200.0)])
conn.commit()

def transfer_funds(connection: sqlite3.Connection, sender: str, receiver: str, amount: float) -> None:
    cur = connection.cursor()
    try:
        # Begin explicit transaction
        cur.execute("UPDATE bank SET balance = balance - ? WHERE owner = ?", (amount, sender))
        if cur.rowcount == 0:
            raise ValueError(f"Sender {sender} not found")

        cur.execute("UPDATE bank SET balance = balance + ? WHERE owner = ?", (amount, receiver))
        if cur.rowcount == 0:
            raise ValueError(f"Receiver {receiver} not found")

        connection.commit()
        print(f"Transferred {amount} from {sender} to {receiver}.")
    except (sqlite3.Error, ValueError) as exc:
        connection.rollback()
        print(f"Transfer failed ({exc}); transaction rolled back.")
    finally:
        cur.close()

# Succeeds
transfer_funds(conn, "Alice", "Bob", 250.0)

# Fails CHECK (balance >= 0) -> rolls back both updates atomically
transfer_funds(conn, "Alice", "Bob", 5000.0)

print("Final balances:", cursor.execute("SELECT * FROM bank").fetchall())
cursor.close()
conn.close()
```

---

## 9. Context Managers with Databases

Using Python's `with` statement ensures cleaner error handling, but **be careful about what each driver's context manager actually does**:

### Crucial `sqlite3` Gotcha

In `sqlite3`, `with conn:` manages the **transaction** (commits if the block succeeds, rolls back if an exception occurs), **NOT the connection lifecycle** — it does **not** close `conn` on exit!

```python
import sqlite3
from contextlib import closing

# Pattern A: Nested closing() + transaction context manager
with closing(sqlite3.connect("app.db")) as conn:
    with conn:  # Commits on success, rolls back on exception
        conn.execute("CREATE TABLE IF NOT EXISTS logs (msg TEXT)")
        conn.execute("INSERT INTO logs (msg) VALUES (?)", ("Started",))
# Here: transaction is committed/rolled back AND conn is closed.
```

### Reusable Custom Context Manager

You can also define a clean helper that guarantees commit/rollback **and** closes both cursor and connection:

```python
import sqlite3
from contextlib import contextmanager

@contextmanager
def db_session(db_path: str):
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()

# Usage
with db_session(":memory:") as conn:
    conn.execute("CREATE TABLE notes (id INTEGER PRIMARY KEY, body TEXT)")
    conn.execute("INSERT INTO notes (body) VALUES (?)", ("Hello context manager",))
```

*(Note: In `psycopg2`, `with conn:` also commits/rolls back transactions without closing the connection. In `psycopg` v3 and SQLAlchemy `Session`, `with` can manage closing or transactions depending on the object.)*

---

## 10. MySQL Connectivity (`mysql-connector-python`)

To connect Python to a MySQL or MariaDB server, Oracle's official pure-Python/C-extension driver is **`mysql-connector-python`** (alternatives include `PyMySQL` and `mysqlclient`).

### Installation

```bash
pip install mysql-connector-python
```

### Connecting and Performing Operations

MySQL uses `%s` (format style) for parameterized queries regardless of the column's actual data type.

```python
import os
import mysql.connector
from mysql.connector import Error

conn = None
cursor = None

try:
    conn = mysql.connector.connect(
        host=os.getenv("MYSQL_HOST", "localhost"),
        port=int(os.getenv("MYSQL_PORT", "3306")),
        user=os.getenv("MYSQL_USER", "app_user"),
        password=os.environ["MYSQL_PASSWORD"],
        database=os.getenv("MYSQL_DB", "inventory"),
        autocommit=False,
    )

    # dictionary=True returns each row as a dict instead of a tuple
    cursor = conn.cursor(dictionary=True)

    # Create table
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS products (
            id INT AUTO_INCREMENT PRIMARY KEY,
            sku VARCHAR(32) NOT NULL UNIQUE,
            name VARCHAR(120) NOT NULL,
            stock INT NOT NULL DEFAULT 0
        )
    """)

    # Parameterized insert (%s placeholders)
    cursor.execute(
        "INSERT INTO products (sku, name, stock) VALUES (%s, %s, %s)",
        ("SKU-101", "Mechanical Keyboard", 25),
    )
    conn.commit()

    # Parameterized select
    cursor.execute("SELECT id, sku, name, stock FROM products WHERE stock > %s", (10,))
    for row in cursor.fetchall():
        print(row)  # {'id': 1, 'sku': 'SKU-101', 'name': 'Mechanical Keyboard', 'stock': 25}

except Error as err:
    if conn and conn.is_connected():
        conn.rollback()
    print("MySQL error:", err)
finally:
    if cursor is not None:
        cursor.close()
    if conn is not None and conn.is_connected():
        conn.close()
```

> **Note:** In `mysql-connector-python`, if you execute a `SELECT` query and don't fetch all rows before running another query on the same connection, it raises `InternalError: Unread result found`. Either fetch all rows or pass `buffered=True` to `conn.cursor(buffered=True)`.

---

## 11. PostgreSQL Connectivity (`psycopg2`) Overview

**`psycopg2`** is the classic, battle-tested DB-API 2.0 PostgreSQL adapter for Python. (For new projects, **`psycopg`** v3 is its modern successor with native async support, though `psycopg2` remains widely used across existing codebases and SQLAlchemy.)

### Installation

```bash
pip install psycopg2-binary   # Prebuilt wheels for development
# Production builds typically compile from source via: pip install psycopg2
```

### Connecting and Querying with `psycopg2`

- Uses `%s` or `%(name)s` placeholders.
- Provides `psycopg2.extras.RealDictCursor` to return dictionary-like rows.
- `with conn:` commits on clean exit and rolls back on exception (remember to still call `conn.close()`).

```python
import os
import psycopg2
from psycopg2.extras import RealDictCursor

conn = None
try:
    conn = psycopg2.connect(
        host=os.getenv("PGHOST", "localhost"),
        port=int(os.getenv("PGPORT", "5432")),
        dbname=os.getenv("PGDATABASE", "analytics"),
        user=os.getenv("PGUSER", "postgres"),
        password=os.environ["PGPASSWORD"],
    )

    with conn:  # Transaction scope (commit / rollback)
        with conn.cursor(cursor_factory=RealDictCursor) as cur:
            cur.execute("""
                CREATE TABLE IF NOT EXISTS events (
                    id SERIAL PRIMARY KEY,
                    event_type TEXT NOT NULL,
                    payload JSONB NOT NULL DEFAULT '{}'::jsonb
                );
            """)

            cur.execute(
                "INSERT INTO events (event_type, payload) VALUES (%s, %s) RETURNING id;",
                ("user_signup", '{"plan": "pro"}'),
            )
            new_id = cur.fetchone()["id"]
            print("Inserted PostgreSQL row ID:", new_id)

            cur.execute(
                "SELECT id, event_type, payload FROM events WHERE event_type = %(etype)s;",
                {"etype": "user_signup"},
            )
            print("Rows:", cur.fetchall())

except psycopg2.Error as exc:
    print("PostgreSQL error:", exc)
finally:
    if conn is not None:
        conn.close()
```

---

## 12. ORM Concepts — SQLAlchemy Basics

An **Object-Relational Mapper (ORM)** bridges the gap between object-oriented Python code and relational database tables:

| SQL Concept | ORM Equivalent |
| :--- | :--- |
| Table | Python `class` (Model) |
| Row | Class instance (Object) |
| Column | Class attribute (`Mapped[...]`) |
| Foreign Key Join | Object relationship (`relationship()`) |

### Pros & Cons of Using an ORM

- **Pros**: Database portability, type-safe Python models, automatic parameter binding (SQL injection protection), unit-of-work tracking, relationship loading, schema migrations (with **Alembic**).
- **Cons**: Abstraction overhead on complex analytical queries, risk of **N+1 query** pitfalls if relationships aren't eager-loaded, steeper learning curve.

### SQLAlchemy 2.0 Example (Declarative ORM)

```bash
pip install sqlalchemy
```

```python
from sqlalchemy import ForeignKey, String, create_engine, select
from sqlalchemy.orm import DeclarativeBase, Mapped, Session, mapped_column, relationship


class Base(DeclarativeBase):
    pass


class Author(Base):
    __tablename__ = "authors"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100), nullable=False)
    books: Mapped[list["Book"]] = relationship(
        back_populates="author",
        cascade="all, delete-orphan",
    )

    def __repr__(self) -> str:
        return f"Author(id={self.id!r}, name={self.name!r})"


class Book(Base):
    __tablename__ = "books"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(200), nullable=False)
    author_id: Mapped[int] = mapped_column(ForeignKey("authors.id"), nullable=False)
    author: Mapped["Author"] = relationship(back_populates="books")

    def __repr__(self) -> str:
        return f"Book(id={self.id!r}, title={self.title!r})"


# 1. Create engine (swap URL to postgresql+psycopg2://... or mysql+mysqlconnector://...)
engine = create_engine("sqlite:///:memory:", echo=False)

# 2. Create tables from models
Base.metadata.create_all(engine)

# 3. Create & Read via Session
with Session(engine) as session:
    author = Author(
        name="Ada Lovelace",
        books=[Book(title="Notes on the Analytical Engine")],
    )
    session.add(author)
    session.commit()

    # Query using SQLAlchemy 2.0 select()
    stmt = select(Book).join(Book.author).where(Author.name == "Ada Lovelace")
    for book in session.scalars(stmt):
        print(book.title, "by", book.author.name)

    # Update
    book_to_update = session.scalar(select(Book).where(Book.id == 1))
    if book_to_update:
        book_to_update.title = "Sketch of the Analytical Engine"
        session.commit()

    # Delete
    if book_to_update:
        session.delete(book_to_update)
        session.commit()
```

---

## 13. MongoDB with PyMongo Overview

**MongoDB** is a **NoSQL document database**. Instead of rigid tables and rows, it stores flexible JSON-like documents (**BSON**) inside **collections**. **`pymongo`** is the official MongoDB driver for Python.

| Relational (SQL) | MongoDB (NoSQL) |
| :--- | :--- |
| Database | Database |
| Table | Collection |
| Row | Document (Python `dict`) |
| Column | Field |
| Primary Key (`id`) | `_id` (auto-generated `ObjectId`) |

### Installation

```bash
pip install pymongo
```

### PyMongo CRUD Example

```python
import os
from pymongo import MongoClient
from pymongo.errors import PyMongoError

uri = os.getenv("MONGODB_URI", "mongodb://localhost:27017/")

# MongoClient manages an internal connection pool automatically
with MongoClient(uri, serverSelectionTimeoutMS=5000) as client:
    try:
        # Access database and collection (created lazily on first write)
        db = client["ecommerce"]
        users_col = db["users"]

        # Ensure unique index on email
        users_col.create_index("email", unique=True)

        # CREATE: insert_one / insert_many
        insert_res = users_col.insert_one({
            "name": "Maya",
            "email": "maya@example.com",
            "tags": ["premium", "beta"],
            "orders": 3,
        })
        print("Inserted _id:", insert_res.inserted_id)

        # READ: find_one / find with query operators ($gte, $in, etc.)
        user = users_col.find_one({"email": "maya@example.com"})
        print("Found user:", user)

        active_users = users_col.find(
            {"orders": {"$gte": 1}},
            {"name": 1, "email": 1, "_id": 0},  # Projection
        )
        print("Active users:", list(active_users))

        # UPDATE: update_one with $set / $inc
        update_res = users_col.update_one(
            {"email": "maya@example.com"},
            {"$inc": {"orders": 1}, "$set": {"tier": "gold"}},
        )
        print("Modified count:", update_res.modified_count)

        # DELETE: delete_one / delete_many
        delete_res = users_col.delete_one({"email": "maya@example.com"})
        print("Deleted count:", delete_res.deleted_count)

    except PyMongoError as exc:
        print("MongoDB error:", exc)
```

---

## 14. Connection Pooling

Opening a new TCP connection, performing TLS negotiation, and authenticating with a database server for every request is expensive. **Connection pooling** maintains a reusable cache of open database connections:

```text
Application Threads / Requests
   │         │         │
   ▼         ▼         ▼
┌─────────────────────────┐
│     Connection Pool     │  (Borrow -> Use -> Return, instead of Connect/Close)
└─────────────────────────┘
   │         │         │
   ▼         ▼         ▼
      Database Server
```

- **When you need it**: Multi-threaded web APIs (FastAPI, Flask, Django), worker queues, or high-throughput scripts hitting PostgreSQL/MySQL. *(SQLite is in-process, so TCP pooling doesn't apply; PyMongo pools automatically inside `MongoClient`.)*

### 1. MySQL Pool (`mysql.connector.pooling`)

```python
import os
from mysql.connector import pooling

mysql_pool = pooling.MySQLConnectionPool(
    pool_name="app_pool",
    pool_size=5,
    pool_reset_session=True,
    host="localhost",
    user="app_user",
    password=os.environ["MYSQL_PASSWORD"],
    database="inventory",
)

# Borrow a connection; .close() returns it to the pool instead of destroying it
conn = mysql_pool.get_connection()
try:
    with conn.cursor() as cur:
        cur.execute("SELECT 1")
        print(cur.fetchone())
finally:
    conn.close()
```

### 2. PostgreSQL Pool (`psycopg2.pool`)

```python
import os
from psycopg2 import pool

pg_pool = pool.ThreadedConnectionPool(
    minconn=1,
    maxconn=10,
    host="localhost",
    dbname="analytics",
    user="postgres",
    password=os.environ["PGPASSWORD"],
)

conn = pg_pool.getconn()
try:
    with conn.cursor() as cur:
        cur.execute("SELECT NOW()")
        print(cur.fetchone())
    conn.commit()
finally:
    # Return connection to pool
    pg_pool.putconn(conn)

# On application shutdown:
# pg_pool.closeall()
```

### 3. SQLAlchemy Built-in Pooling (`QueuePool`)

SQLAlchemy uses `QueuePool` by default for server-backed databases:

```python
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql+psycopg2://user:pass@localhost:5432/analytics",
    pool_size=10,        # Persistent connections kept open
    max_overflow=20,     # Temporary burst connections allowed above pool_size
    pool_timeout=30,     # Seconds to wait before raising TimeoutError
    pool_recycle=1800,   # Recycle connections older than 30 min (prevents stale sockets)
    pool_pre_ping=True,  # Test connection liveness before checkout
)
```

---

## 15. Best Practices for Database Operations

1. **Always use parameterized queries**: Never interpolate user input into SQL strings. Validate dynamic table/column identifiers against strict allowlists.
2. **Never hardcode credentials**: Load connection strings, passwords, and API keys from environment variables (`os.environ`) or a secrets manager, never commit them to Git.
3. **Follow the Principle of Least Privilege**: Application database users should only have the permissions they require (e.g., `SELECT`, `INSERT`, `UPDATE`, `DELETE` on specific schemas — not superuser `DROP DATABASE` rights).
4. **Close resources deterministically**: Use context managers (`with`) and `try...finally` blocks so cursors, connections, and sessions don't leak under exceptions.
5. **Keep transactions short and explicit**: Hold locks for the minimum time necessary; never wait on external HTTP requests or user input while a database transaction is open.
6. **Use connection pooling in concurrent apps**: Reuse connections rather than opening/closing TCP sockets per request, and enable health checks (`pool_pre_ping=True`) and timeouts.
7. **Batch bulk operations**: Prefer `executemany()`, bulk inserts, or `fetchmany(size)` over looping single-row queries across thousands of records.
8. **Avoid `SELECT *` in production**: Explicitly list needed columns to reduce network I/O, memory usage, and breakage when schemas evolve.
9. **Index thoughtfully and prevent N+1 queries**: Add indexes on columns frequently used in `WHERE`, `JOIN`, and `ORDER BY` clauses; in ORMs, use eager loading (`selectinload` / `joinedload`) rather than lazy-loading inside loops.
10. **Manage schema changes with migrations**: Use a migration tool like **Alembic** (for SQLAlchemy) or **Django Migrations** instead of manual ad-hoc `ALTER TABLE` statements in production.
11. **Enable WAL mode for concurrent SQLite reads**: For local apps with concurrent readers and a writer in SQLite, `PRAGMA journal_mode=WAL;` significantly improves concurrency alongside `PRAGMA foreign_keys=ON;`.
12. **Handle transient failures & back up data**: Catch specific DB exceptions (`IntegrityError` vs `OperationalError`), log context cleanly, and automate encrypted backups with periodic restore testing.
