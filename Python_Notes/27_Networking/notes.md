# Networking in Python

Python provides rich built-in modules (`socket`, `urllib`, `smtplib`, `ftplib`, `http`) as well as widely adopted third-party libraries (`requests`, `flask`, `beautifulsoup4`) for building networked applications, consuming web APIs, scraping web pages, sending emails, and transferring files.

---

## Table of Contents

1. [Networking Basics (TCP/IP, UDP, Ports, Sockets)](#1-networking-basics-tcpip-udp-ports-sockets)
2. [`socket` Module Overview](#2-socket-module-overview)
3. [Creating a TCP Server](#3-creating-a-tcp-server)
4. [Creating a TCP Client](#4-creating-a-tcp-client)
5. [Creating a UDP Server and Client](#5-creating-a-udp-server-and-client)
6. [HTTP Requests with `urllib`](#6-http-requests-with-urllib)
7. [HTTP Requests with `requests` Library](#7-http-requests-with-requests-library)
8. [GET, POST, PUT, DELETE Requests](#8-get-post-put-delete-requests)
9. [Handling Responses (Status Code, Headers, JSON, Text)](#9-handling-responses-status-code-headers-json-text)
10. [Session Objects and Cookies](#10-session-objects-and-cookies)
11. [Authentication (Basic, Token)](#11-authentication-basic-token)
12. [REST API Concepts](#12-rest-api-concepts)
13. [Building a Simple API with Flask](#13-building-a-simple-api-with-flask)
14. [Web Scraping Basics (BeautifulSoup)](#14-web-scraping-basics-beautifulsoup)
15. [Email Sending (`smtplib`)](#15-email-sending-smtplib)
16. [FTP Operations (`ftplib`)](#16-ftp-operations-ftplib)

---

## 1. Networking Basics (TCP/IP, UDP, Ports, Sockets)

Computer networking enables programs on different machines (or the same machine) to exchange data over a network.

### IP Addresses and Hostnames

- **IPv4**: 32-bit address written as four octets, e.g., `192.168.1.10`.
- **IPv6**: 128-bit address written in hexadecimal groups, e.g., `2001:0db8:85a3::8a2e:0370:7334`.
- **Loopback (`localhost` / `127.0.0.1` / `::1`)**: Refers to the local machine itself; useful for testing servers and clients locally.
- **`0.0.0.0`**: When a server binds to `0.0.0.0`, it listens on all available network interfaces.
- **DNS (Domain Name System)**: Translates human-readable hostnames like `example.com` into IP addresses.

### TCP vs. UDP

| Feature | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
| :--- | :--- | :--- |
| **Connection** | Connection-oriented (3-way handshake) | Connectionless |
| **Reliability** | Guaranteed delivery, acknowledgments, retransmission | Best-effort; packets may drop or arrive duplicated |
| **Ordering** | Preserves byte-stream order | No ordering guarantee |
| **Speed / Overhead** | Higher overhead due to flow/congestion control | Low overhead, fast |
| **Data Model** | Continuous byte stream (`SOCK_STREAM`) | Discrete datagrams/messages (`SOCK_DGRAM`) |
| **Common Uses** | HTTP/HTTPS, SSH, FTP, SMTP, databases | DNS, video/audio streaming, online gaming, VoIP |

### Ports

A **port** is a 16-bit integer (`0`–`65535`) identifying a specific process or service on a host.

- **Well-known ports (`0`–`1023`)**: Reserved for standard services (`21` FTP, `22` SSH, `25` SMTP, `53` DNS, `80` HTTP, `443` HTTPS). Binding to these on Unix/Linux usually requires root privileges.
- **Registered ports (`1024`–`49151`)**: Assigned to user applications and databases (`3306` MySQL, `5000` Flask dev default, `5432` PostgreSQL, `8080` alt HTTP).
- **Ephemeral/dynamic ports (`49152`–`65535`)**: Automatically assigned by the OS to client connections.

### Sockets

A **socket** is a software endpoint for two-way communication between programs, uniquely identified by the tuple `(protocol, local_ip, local_port, remote_ip, remote_port)` for connected sockets.

---

## 2. `socket` Module Overview

Python's built-in `socket` module provides low-level access to the OS networking interface.

### Key Constants

- **Address Families**:
  - `socket.AF_INET` — IPv4
  - `socket.AF_INET6` — IPv6
  - `socket.AF_UNIX` — Unix domain sockets (local IPC on POSIX systems)
- **Socket Types**:
  - `socket.SOCK_STREAM` — TCP (stream socket)
  - `socket.SOCK_DGRAM` — UDP (datagram socket)

### Common Functions and Methods

| Function / Method | Description |
| :--- | :--- |
| `socket.socket(family, type)` | Creates a new socket object |
| `sock.bind((host, port))` | Binds the socket to an IP address and port |
| `sock.listen(backlog)` | Enables a TCP server to accept incoming connections |
| `sock.accept()` | Blocks until a client connects; returns `(conn_socket, client_addr)` |
| `sock.connect((host, port))` | Connects a client socket to a remote server |
| `sock.send(bytes)` / `sock.sendall(bytes)` | Sends bytes (`sendall` ensures all data is transmitted) |
| `sock.recv(bufsize)` | Receives up to `bufsize` bytes from a TCP socket |
| `sock.sendto(bytes, addr)` | Sends a UDP datagram to `(host, port)` |
| `sock.recvfrom(bufsize)` | Receives a UDP datagram; returns `(data, sender_addr)` |
| `sock.settimeout(seconds)` | Sets a timeout on blocking socket operations |
| `sock.setsockopt(level, optname, value)` | Configures socket options (e.g., `SO_REUSEADDR`) |
| `sock.close()` | Closes the socket (use `with` statement for automatic closing) |
| `socket.gethostbyname(hostname)` | Resolves a hostname to an IPv4 address |
| `socket.gethostname()` | Returns the local machine's hostname |

### DNS & Host Utility Example

```python
import socket

hostname = socket.gethostname()
print(f"Local hostname: {hostname}")

ip = socket.gethostbyname("example.com")
print(f"example.com IPv4: {ip}")

# Detailed address info (supports IPv4 and IPv6)
info = socket.getaddrinfo("example.com", 80, proto=socket.IPPROTO_TCP)
for family, socktype, proto, canonname, sockaddr in info:
    print(family, sockaddr)
```

> **Note on Bytes vs. Strings:** Sockets transmit `bytes`, not Python `str`. Always encode strings before sending (`msg.encode("utf-8")`) and decode received bytes (`data.decode("utf-8")`).

---

## 3. Creating a TCP Server

A TCP server follows a standard lifecycle:
`socket()` -> `bind()` -> `listen()` -> `accept()` -> `recv()` / `send()` -> `close()`

### Basic Single-Client Echo Server

```python
import socket

HOST = "127.0.0.1"  # Standard loopback interface (localhost)
PORT = 65432        # Non-privileged port (> 1023)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server_sock:
    # Allow immediate reuse of the port after server restarts
    server_sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_sock.bind((HOST, PORT))
    server_sock.listen()
    print(f"TCP server listening on {HOST}:{PORT}...")

    conn, addr = server_sock.accept()
    with conn:
        print(f"Connected by {addr}")
        while True:
            data = conn.recv(1024)
            if not data:
                # Empty bytes means the client closed the connection
                break
            print(f"Received: {data.decode('utf-8')}")
            conn.sendall(data)  # Echo back the received bytes
```

### Handling Multiple Clients with `threading`

A real server handles multiple clients concurrently instead of blocking on one connection:

```python
import socket
import threading

HOST = "127.0.0.1"
PORT = 65432


def handle_client(conn: socket.socket, addr: tuple[str, int]) -> None:
    print(f"[NEW CONNECTION] {addr}")
    with conn:
        while True:
            data = conn.recv(1024)
            if not data:
                break
            message = data.decode("utf-8")
            print(f"[{addr}] {message}")
            conn.sendall(f"Echo: {message}".encode("utf-8"))
    print(f"[DISCONNECTED] {addr}")


def start_server() -> None:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
        server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        server.bind((HOST, PORT))
        server.listen()
        print(f"Multi-threaded TCP server listening on {HOST}:{PORT}...")

        while True:
            conn, addr = server.accept()
            thread = threading.Thread(target=handle_client, args=(conn, addr), daemon=True)
            thread.start()


if __name__ == "__main__":
    start_server()
```

---

## 4. Creating a TCP Client

A TCP client follows:
`socket()` -> `connect()` -> `sendall()` / `recv()` -> `close()`

```python
import socket

HOST = "127.0.0.1"
PORT = 65432

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client_sock:
    client_sock.settimeout(5.0)  # Avoid hanging indefinitely
    client_sock.connect((HOST, PORT))

    message = "Hello, TCP Server!"
    client_sock.sendall(message.encode("utf-8"))

    response = client_sock.recv(1024)
    print(f"Server replied: {response.decode('utf-8')}")
```

### Interactive Client Example

```python
import socket

HOST = "127.0.0.1"
PORT = 65432

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    sock.connect((HOST, PORT))
    print("Connected. Type 'quit' to exit.")

    while True:
        msg = input("> ")
        if msg.lower() == "quit":
            break
        sock.sendall(msg.encode("utf-8"))
        data = sock.recv(1024)
        if not data:
            print("Server closed connection.")
            break
        print("Server:", data.decode("utf-8"))
```

> **TCP Stream Caveat:** TCP is a byte stream, not a message stream. A single `sendall()` might require multiple `recv()` calls for large payloads, or multiple small sends might be coalesced into one `recv()`. For real protocols, frame messages with a delimiter (like `\n`) or a fixed-length size prefix.

---

## 5. Creating a UDP Server and Client

UDP uses `SOCK_DGRAM`. There is no `listen()`, `accept()`, or `connect()` handshake—datagrams are sent and received directly with addresses.

### UDP Server

```python
import socket

HOST = "127.0.0.1"
PORT = 65433

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as udp_server:
    udp_server.bind((HOST, PORT))
    print(f"UDP server listening on {HOST}:{PORT}...")

    while True:
        data, client_addr = udp_server.recvfrom(1024)
        message = data.decode("utf-8")
        print(f"Received from {client_addr}: {message}")

        reply = f"UDP ACK: {message}".encode("utf-8")
        udp_server.sendto(reply, client_addr)
```

### UDP Client

```python
import socket

SERVER_HOST = "127.0.0.1"
SERVER_PORT = 65433

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as udp_client:
    udp_client.settimeout(3.0)

    message = "Hello, UDP Server!"
    udp_client.sendto(message.encode("utf-8"), (SERVER_HOST, SERVER_PORT))

    try:
        data, server_addr = udp_client.recvfrom(1024)
        print(f"Reply from {server_addr}: {data.decode('utf-8')}")
    except socket.timeout:
        print("No response received (UDP packet may have been lost).")
```

---

## 6. HTTP Requests with `urllib`

`urllib` is part of Python's standard library (no third-party installation required). Its key submodules are:
- `urllib.request`: Opening and reading URLs
- `urllib.parse`: Parsing and encoding URLs and query parameters
- `urllib.error`: Exceptions (`HTTPError`, `URLError`)

### GET Request with Query Parameters

```python
import json
import urllib.parse
import urllib.request
import urllib.error

params = urllib.parse.urlencode({"userId": 1, "_limit": 3})
url = f"https://jsonplaceholder.typicode.com/posts?{params}"

req = urllib.request.Request(
    url,
    headers={"User-Agent": "Python-urllib/3"},
)

try:
    with urllib.request.urlopen(req, timeout=10) as response:
        print("Status:", response.status)
        print("Content-Type:", response.headers.get("Content-Type"))
        raw_body = response.read().decode("utf-8")
        data = json.loads(raw_body)
        print(f"Fetched {len(data)} posts.")
except urllib.error.HTTPError as e:
    print(f"HTTP error {e.code}: {e.reason}")
except urllib.error.URLError as e:
    print(f"Network error: {e.reason}")
```

### POST Request with JSON Body

```python
import json
import urllib.request

url = "https://jsonplaceholder.typicode.com/posts"
payload = json.dumps({"title": "Python urllib", "body": "Built-in HTTP", "userId": 1}).encode("utf-8")

req = urllib.request.Request(
    url,
    data=payload,
    method="POST",
    headers={
        "Content-Type": "application/json",
        "Accept": "application/json",
    },
)

with urllib.request.urlopen(req, timeout=10) as response:
    print("Status:", response.status)
    result = json.loads(response.read().decode("utf-8"))
    print("Created:", result)
```

---

## 7. HTTP Requests with `requests` Library

`requests` is the de facto standard third-party library for HTTP in Python because of its clean, human-friendly API.

### Installation

```bash
pip install requests
```

### Basic Usage

```python
import requests

response = requests.get("https://httpbin.org/get", timeout=10)
print(response.status_code)
print(response.json())
```

### Passing Query Parameters, Headers, and Timeouts

```python
import requests

url = "https://httpbin.org/get"
params = {"search": "python networking", "page": 2}
headers = {"Accept": "application/json", "User-Agent": "MyApp/1.0"}

try:
    response = requests.get(url, params=params, headers=headers, timeout=(3.05, 10))
    # timeout=(connect_timeout, read_timeout)
    response.raise_for_status()  # Raises HTTPError for 4xx/5xx responses
    print("Final URL:", response.url)
    print("Args received by server:", response.json()["args"])
except requests.exceptions.Timeout:
    print("Request timed out.")
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")
except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```

> **Best Practice:** Always pass `timeout=...` in `requests` calls. By default, `requests` has no timeout and can hang indefinitely on stalled connections.

---

## 8. GET, POST, PUT, DELETE Requests

HTTP methods express the action to perform on a resource:

| Method | Purpose | Idempotent? | Request Body |
| :--- | :--- | :--- | :--- |
| `GET` | Retrieve a resource or list | Yes | No |
| `POST` | Create a new resource | No | Yes |
| `PUT` | Replace an entire resource | Yes | Yes |
| `PATCH` | Partially update a resource | No (usually) | Yes |
| `DELETE` | Remove a resource | Yes | Optional |

### Examples with `requests`

```python
import requests

BASE_URL = "https://jsonplaceholder.typicode.com/posts"

# 1. GET — Read resource
r_get = requests.get(f"{BASE_URL}/1", timeout=10)
print("GET:", r_get.status_code, r_get.json()["title"])

# 2. POST — Create resource (json= automatically serializes and sets Content-Type)
new_post = {"title": "Async Networking", "body": "Using sockets and HTTP", "userId": 10}
r_post = requests.post(BASE_URL, json=new_post, timeout=10)
print("POST:", r_post.status_code, r_post.json())

# 3. PUT — Full replacement of resource #1
updated_post = {"id": 1, "title": "Updated Title", "body": "Replaced body", "userId": 1}
r_put = requests.put(f"{BASE_URL}/1", json=updated_post, timeout=10)
print("PUT:", r_put.status_code, r_put.json())

# 4. PATCH — Partial update of resource #1
r_patch = requests.patch(f"{BASE_URL}/1", json={"title": "Patched Title"}, timeout=10)
print("PATCH:", r_patch.status_code, r_patch.json())

# 5. DELETE — Delete resource #1
r_delete = requests.delete(f"{BASE_URL}/1", timeout=10)
print("DELETE:", r_delete.status_code)
```

### Sending Form Data and Uploading Files

```python
import requests

# Form-encoded POST (application/x-www-form-urlencoded)
r_form = requests.post("https://httpbin.org/post", data={"username": "alice", "role": "admin"}, timeout=10)
print(r_form.json()["form"])

# Multipart file upload (multipart/form-data)
with open("example.txt", "rb") as f:
    r_file = requests.post("https://httpbin.org/post", files={"file": ("example.txt", f)}, timeout=10)
    print(r_file.json()["files"])
```

---

## 9. Handling Responses (Status Code, Headers, JSON, Text)

When `requests` returns a `Response` object, you can inspect every part of the HTTP response.

### Common HTTP Status Code Ranges

- `2xx` (Success): `200 OK`, `201 Created`, `204 No Content`
- `3xx` (Redirection): `301 Moved Permanently`, `302 Found`, `304 Not Modified`
- `4xx` (Client Error): `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests`
- `5xx` (Server Error): `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`

### Inspecting Response Attributes

```python
import requests

resp = requests.get("https://httpbin.org/json", timeout=10)

# Status code & check
print("Status code:", resp.status_code)   # e.g., 200
print("Reason:", resp.reason)             # e.g., 'OK'
print("Success?:", resp.ok)               # True if status_code < 400
resp.raise_for_status()                   # Raises HTTPError if 4xx or 5xx

# Headers (case-insensitive dictionary)
print("All headers:", dict(resp.headers))
print("Content-Type:", resp.headers.get("content-type"))

# Body formats
print("Raw bytes:", resp.content[:60])    # bytes (ideal for images, PDFs, binaries)
print("Encoding:", resp.encoding)         # e.g., 'utf-8'
print("Decoded text:", resp.text[:100])   # str

# JSON parsing
try:
    data = resp.json()
    print("Parsed JSON keys:", list(data.keys()))
except requests.exceptions.JSONDecodeError:
    print("Response is not valid JSON.")
```

### Streaming Large Downloads

For large files, avoid loading the entire response into memory at once:

```python
import requests

url = "https://httpbin.org/bytes/102400"
with requests.get(url, stream=True, timeout=15) as resp:
    resp.raise_for_status()
    with open("downloaded.bin", "wb") as f:
        for chunk in resp.iter_content(chunk_size=8192):
            if chunk:
                f.write(chunk)
```

---

## 10. Session Objects and Cookies

A `requests.Session` object persists cookies, default headers, authentication, and underlying TCP connections (via HTTP keep-alive connection pooling) across multiple requests.

### Using `Session`

```python
import requests

with requests.Session() as session:
    # Default headers applied to every request in this session
    session.headers.update({"User-Agent": "MyClient/2.0", "Accept": "application/json"})

    # Server sets a cookie
    session.get("https://httpbin.org/cookies/set/theme/dark", timeout=10)

    # Subsequent request automatically sends the cookie back
    resp = session.get("https://httpbin.org/cookies", timeout=10)
    print("Cookies on server:", resp.json())

    # Inspect cookies stored in the session's CookieJar
    print("Local CookieJar:", session.cookies.get_dict())
```

### Sending and Reading Cookies Manually (Without Session)

```python
import requests

# Send custom cookies on a single request
resp = requests.get(
    "https://httpbin.org/cookies",
    cookies={"session_id": "abc123", "prefs": "compact"},
    timeout=10,
)
print(resp.json())

# Read cookies returned by the server
resp2 = requests.get("https://httpbin.org/cookies/set?token=xyz", timeout=10)
print("Returned cookies:", resp2.cookies.get_dict())
```

---

## 11. Authentication (Basic, Token)

APIs commonly require authentication to verify client identity and enforce access control.

### HTTP Basic Authentication

Basic auth sends `Authorization: Basic <base64(username:password)>`. Always use **HTTPS** so credentials are encrypted in transit.

```python
import requests
from requests.auth import HTTPBasicAuth

# Explicit HTTPBasicAuth
resp = requests.get(
    "https://httpbin.org/basic-auth/alice/secret123",
    auth=HTTPBasicAuth("alice", "secret123"),
    timeout=10,
)
print("Basic Auth status:", resp.status_code, resp.json())

# Shorthand tuple syntax (equivalent)
resp2 = requests.get(
    "https://httpbin.org/basic-auth/alice/secret123",
    auth=("alice", "secret123"),
    timeout=10,
)
```

### Bearer Token Authentication (JWT / OAuth2 / API Tokens)

Bearer tokens are sent in the `Authorization` header: `Authorization: Bearer <token>`.

```python
import os
import requests

token = os.environ.get("API_TOKEN", "my-demo-token")

headers = {
    "Authorization": f"Bearer {token}",
    "Accept": "application/json",
}

resp = requests.get("https://httpbin.org/bearer", headers=headers, timeout=10)
print("Bearer status:", resp.status_code, resp.json())
```

### API Key Authentication

Some APIs expect an API key in a custom header or query parameter:

```python
import requests

# Custom header (preferred over query string so keys aren't logged in URLs)
resp = requests.get(
    "https://httpbin.org/headers",
    headers={"X-API-Key": "secret-api-key"},
    timeout=10,
)
print(resp.json()["headers"])
```

### Reusable Custom Auth Class in `requests`

```python
import requests
from requests.auth import AuthBase


class BearerAuth(AuthBase):
    def __init__(self, token: str) -> None:
        self.token = token

    def __call__(self, request: requests.PreparedRequest) -> requests.PreparedRequest:
        request.headers["Authorization"] = f"Bearer {self.token}"
        return request


with requests.Session() as session:
    session.auth = BearerAuth("my-jwt-token")
    r = session.get("https://httpbin.org/bearer", timeout=10)
    print(r.json())
```

---

## 12. REST API Concepts

**REST** (Representational State Transfer) is an architectural style for designing networked APIs over HTTP.

### Core Principles

1. **Resources & URIs**: Everything is a resource identified by a URL using nouns, not verbs:
   - Good: `GET /api/books`, `GET /api/books/42`, `DELETE /api/books/42`
   - Avoid: `GET /api/getBooks`, `POST /api/deleteBook?id=42`
2. **HTTP Methods for Operations (CRUD)**:
   - `POST` -> Create
   - `GET` -> Read
   - `PUT` / `PATCH` -> Update
   - `DELETE` -> Delete
3. **Statelessness**: Each request contains all information needed to process it (e.g., auth token, parameters); the server does not rely on stored client session state between API calls.
4. **Representations**: Resources are exchanged in standard formats—most commonly **JSON** (`Content-Type: application/json`).
5. **Meaningful Status Codes**:
   - `200 OK`: Successful read/update
   - `201 Created`: Resource created (often paired with `Location` header)
   - `204 No Content`: Successful delete/action with empty body
   - `400 Bad Request`: Validation error / malformed JSON
   - `401 Unauthorized` / `403 Forbidden`: Missing/invalid auth or insufficient permissions
   - `404 Not Found`: Resource does not exist
   - `409 Conflict`: Duplicate or conflicting state

---

## 13. Building a Simple API with Flask

**Flask** is a lightweight Python web framework ideal for building REST APIs and microservices.

### Installation

```bash
pip install flask
```

### Complete CRUD REST API Example (`app.py`)

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# In-memory store for demonstration
books = [
    {"id": 1, "title": "Fluent Python", "author": "Luciano Ramalho"},
    {"id": 2, "title": "Effective Python", "author": "Brett Slatkin"},
]
next_id = 3


def find_book(book_id: int) -> dict | None:
    return next((b for b in books if b["id"] == book_id), None)


@app.get("/api/books")
def list_books():
    return jsonify({"books": books}), 200


@app.get("/api/books/<int:book_id>")
def get_book(book_id: int):
    book = find_book(book_id)
    if not book:
        return jsonify({"error": "Book not found"}), 404
    return jsonify(book), 200


@app.post("/api/books")
def create_book():
    global next_id
    data = request.get_json(silent=True)
    if not data or "title" not in data or "author" not in data:
        return jsonify({"error": "Both 'title' and 'author' are required"}), 400

    book = {"id": next_id, "title": data["title"], "author": data["author"]}
    next_id += 1
    books.append(book)
    return jsonify(book), 201


@app.put("/api/books/<int:book_id>")
def update_book(book_id: int):
    book = find_book(book_id)
    if not book:
        return jsonify({"error": "Book not found"}), 404

    data = request.get_json(silent=True) or {}
    if "title" not in data or "author" not in data:
        return jsonify({"error": "Both 'title' and 'author' are required"}), 400

    book["title"] = data["title"]
    book["author"] = data["author"]
    return jsonify(book), 200


@app.delete("/api/books/<int:book_id>")
def delete_book(book_id: int):
    book = find_book(book_id)
    if not book:
        return jsonify({"error": "Book not found"}), 404

    books.remove(book)
    return "", 204


if __name__ == "__main__":
    app.run(host="127.0.0.1", port=5000, debug=True)
```

### Testing the Flask API with `requests`

```python
import requests

BASE = "http://127.0.0.1:5000/api/books"

print(requests.get(BASE, timeout=5).json())
created = requests.post(BASE, json={"title": "Python Cookbook", "author": "David Beazley"}, timeout=5)
print("Created:", created.status_code, created.json())
```

---

## 14. Web Scraping Basics (BeautifulSoup)

**BeautifulSoup** (`bs4`) parses HTML and XML documents into a navigable tree so you can extract text, links, tables, and attributes.

### Installation

```bash
pip install beautifulsoup4 requests lxml
```

### Parsing HTML and Extracting Data

```python
import requests
from bs4 import BeautifulSoup

html = """
<html>
  <head><title>Book Store</title></head>
  <body>
    <h1 id="main-heading">Featured Books</h1>
    <ul class="book-list">
      <li class="book" data-id="101">
        <a href="/books/101" class="title">Python Crash Course</a>
        <span class="price">$39.99</span>
      </li>
      <li class="book" data-id="102">
        <a href="/books/102" class="title">Automate the Boring Stuff</a>
        <span class="price">$29.99</span>
      </li>
    </ul>
  </body>
</html>
"""

soup = BeautifulSoup(html, "html.parser")  # or "lxml" for faster parsing

# 1. Accessing tags directly
print("Page title:", soup.title.text)

# 2. Finding a single element
heading = soup.find("h1", id="main-heading")
print("Heading:", heading.get_text(strip=True))

# 3. Finding all matching elements
for item in soup.find_all("li", class_="book"):
    book_id = item["data-id"]
    link = item.find("a", class_="title")
    price = item.find("span", class_="price").text
    print(f"ID={book_id} | Title={link.text} | URL={link['href']} | Price={price}")

# 4. CSS selectors with select() and select_one()
titles = [el.get_text(strip=True) for el in soup.select("ul.book-list li.book a.title")]
print("Titles via CSS selector:", titles)
```

### Fetching and Scraping a Live Page

```python
import requests
from bs4 import BeautifulSoup

url = "https://example.com"
resp = requests.get(url, headers={"User-Agent": "Mozilla/5.0"}, timeout=10)
resp.raise_for_status()

soup = BeautifulSoup(resp.text, "html.parser")
print("Title:", soup.title.string if soup.title else "No title")
for a in soup.find_all("a", href=True):
    print(f"{a.get_text(strip=True)} -> {a['href']}")
```

> **Scraping Etiquette & Legal Notes:** Always check a site's `robots.txt` and Terms of Service, add delays between requests (`time.sleep`) to avoid overloading servers, and prefer official APIs when available.

---

## 15. Email Sending (`smtplib`)

Python's built-in `smtplib` sends emails via the **SMTP** (Simple Mail Transfer Protocol), typically paired with `email.message.EmailMessage` to format headers, plain text, HTML bodies, and attachments.

### Common SMTP Ports

- **`465`**: Implicit SSL/TLS (`smtplib.SMTP_SSL`)
- **`587`**: Explicit TLS via `STARTTLS` (`smtplib.SMTP` + `server.starttls()`)

### Sending Plain Text and HTML Emails

```python
import os
import smtplib
import ssl
from email.message import EmailMessage

SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 587  # STARTTLS port
SENDER_EMAIL = os.environ["SENDER_EMAIL"]
APP_PASSWORD = os.environ["SMTP_PASSWORD"]  # Use an app-specific password, never hardcode
RECIPIENT_EMAIL = "recipient@example.com"

msg = EmailMessage()
msg["Subject"] = "Python Networking Report"
msg["From"] = SENDER_EMAIL
msg["To"] = RECIPIENT_EMAIL

# Plain-text fallback
msg.set_content("Hello,\n\nThis email was sent from Python using smtplib.")

# Optional HTML alternative
msg.add_alternative(
    """\
    <html>
      <body>
        <h2>Python Networking Report</h2>
        <p>This email was sent from <strong>Python</strong> using <code>smtplib</code>.</p>
      </body>
    </html>
    """,
    subtype="html",
)

context = ssl.create_default_context()

with smtplib.SMTP(SMTP_SERVER, SMTP_PORT, timeout=15) as server:
    server.ehlo()
    server.starttls(context=context)  # Upgrade connection to TLS
    server.ehlo()
    server.login(SENDER_EMAIL, APP_PASSWORD)
    server.send_message(msg)

print("Email sent successfully.")
```

### Sending an Email with an Attachment (Using Port 465 `SMTP_SSL`)

```python
import mimetypes
import smtplib
import ssl
from email.message import EmailMessage
from pathlib import Path

msg = EmailMessage()
msg["Subject"] = "Monthly Log"
msg["From"] = "sender@example.com"
msg["To"] = "recipient@example.com"
msg.set_content("Please find the attached report.")

file_path = Path("report.pdf")
mime_type, _ = mimetypes.guess_type(file_path)
maintype, subtype = (mime_type or "application/octet-stream").split("/", 1)

with open(file_path, "rb") as f:
    msg.add_attachment(f.read(), maintype=maintype, subtype=subtype, filename=file_path.name)

context = ssl.create_default_context()
with smtplib.SMTP_SSL("smtp.example.com", 465, context=context) as server:
    server.login("sender@example.com", "app-password")
    server.send_message(msg)
```

---

## 16. FTP Operations (`ftplib`)

Python's built-in `ftplib` implements the **File Transfer Protocol (FTP)** client (`FTP`) and **FTPS** (FTP over TLS: `FTP_TLS`).

> **Security Note:** Plain `FTP` sends credentials and files unencrypted. Whenever the server supports it, prefer `ftplib.FTP_TLS` (or SFTP via `paramiko` over SSH).

### Common `ftplib` Operations

```python
from ftplib import FTP, error_perm

HOST = "ftp.dlptest.com"
USER = "dlpuser"
PASS = "rNrKYTX9g7z3RgJRmxWuGHbeu"

with FTP(HOST, timeout=15) as ftp:
    ftp.login(user=USER, passwd=PASS)
    print("Welcome:", ftp.getwelcome())
    print("Current directory:", ftp.pwd())

    # List files and directories
    print("Directory listing:")
    ftp.retrlines("LIST")

    # Upload a file (binary mode)
    with open("local_upload.txt", "rb") as f:
        ftp.storbinary("STOR remote_upload.txt", f)
    print("Uploaded remote_upload.txt")

    # Download a file (binary mode)
    with open("downloaded_file.txt", "wb") as f:
        ftp.retrbinary("RETR remote_upload.txt", f.write)
    print("Downloaded remote_upload.txt")

    # Directory & file management
    try:
        ftp.mkd("test_dir")
        ftp.cwd("test_dir")
        print("Changed into:", ftp.pwd())
        ftp.cwd("..")
        ftp.rmd("test_dir")
    except error_perm as e:
        print("Permission/FTP error:", e)

    # Delete a file
    ftp.delete("remote_upload.txt")
```

### Secure FTP over TLS (`FTP_TLS`)

```python
from ftplib import FTP_TLS

with FTP_TLS("ftp.example.com", timeout=15) as ftps:
    ftps.login(user="alice", passwd="secret")
    ftps.prot_p()  # Encrypt the data channel in addition to the control channel
    ftps.retrlines("LIST")
```

### Summary of Useful `ftplib.FTP` Methods

| Method | Purpose |
| :--- | :--- |
| `login(user, passwd)` | Authenticate with the FTP server |
| `pwd()` / `cwd(path)` | Get / change current working directory |
| `nlst()` / `retrlines("LIST")` | List filenames / detailed directory listing |
| `storbinary("STOR name", fp)` | Upload a binary file |
| `retrbinary("RETR name", cb)` | Download a binary file via callback (e.g., `f.write`) |
| `mkd(dir)` / `rmd(dir)` | Create / remove directory |
| `rename(from_name, to_name)` | Rename a remote file or directory |
| `delete(filename)` | Delete a remote file |
| `size(filename)` | Get remote file size in bytes |
