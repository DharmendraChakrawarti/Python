# Strings in Python

A **string** (`str`) in Python is an **immutable sequence of Unicode characters** used to store and manipulate textual data. Because strings are sequences, they support indexing, slicing, iteration, membership testing, and length checks, while also providing a rich set of built-in methods for text processing.

---

## Table of Contents

1. [Creating Strings](#1-creating-strings)
2. [String Indexing and Slicing](#2-string-indexing-and-slicing)
3. [String Immutability](#3-string-immutability)
4. [Escape Characters and Raw Strings](#4-escape-characters-and-raw-strings)
5. [String Concatenation and Repetition](#5-string-concatenation-and-repetition)
6. [String Methods](#6-string-methods)
7. [String Formatting](#7-string-formatting)
8. [String Comparison](#8-string-comparison)
9. [String Iteration](#9-string-iteration)
10. [Multiline Strings](#10-multiline-strings)
11. [String Encoding (ASCII, Unicode, encode/decode)](#11-string-encoding-ascii-unicode-encodedecode)
12. [Regular Expressions Basics with Strings](#12-regular-expressions-basics-with-strings)
13. [Common String Operations and Patterns](#13-common-string-operations-and-patterns)

---

## 1. Creating Strings

Strings can be created using single quotes (`'...'`), double quotes (`"..."`), or triple quotes (`'''...'''` / `"""..."""`), as well as the `str()` constructor.

```python
# Single and double quotes are functionally identical
s1 = 'Hello, Python!'
s2 = "Hello, Python!"

# Empty strings
empty1 = ""
empty2 = str()

# Quotes inside strings without escaping
quote_single = "It's a great day."
quote_double = 'She said, "Python is readable."'

# Triple quotes (commonly used for multiline strings or strings with both quote types)
s3 = '''Line one'''
s4 = """He said, "It's fine." """

# Converting other types to strings with str()
num_str = str(42)        # "42"
float_str = str(3.14)    # "3.14"
bool_str = str(True)     # "True"

# Adjacent string literals are implicitly concatenated at compile time
combined = "Hello, " "world!"  # "Hello, world!"
```

> **Tip:** Single and double quotes behave identically in Python. Pick one style for consistency (many projects follow PEP 8 / Black conventions using double quotes) and use the other when the string contains literal quotes.

---

## 2. String Indexing and Slicing

Each character in a string has an integer position (**index**). Python supports both **0-based positive indexing** (from left to right) and **negative indexing** (from right to left).

```
String:    P   y   t   h   o   n
Positive:  0   1   2   3   4   5
Negative: -6  -5  -4  -3  -2  -1
```

### Indexing

```python
text = "Python"

print(text[0])    # 'P' (first character)
print(text[3])    # 'h'
print(text[-1])   # 'n' (last character)
print(text[-6])   # 'P'
print(len(text))  # 6

# Accessing an index out of range raises IndexError
# print(text[6])  # IndexError: string index out of range
```

### Slicing

Slicing extracts a substring using `s[start:stop:step]`:
- `start`: starting index (inclusive; defaults to `0` if `step > 0`).
- `stop`: ending index (exclusive; defaults to `len(s)` if `step > 0`).
- `step`: stride between characters (defaults to `1`).

```python
s = "Hello, World!"

print(s[0:5])     # 'Hello'     (indices 0, 1, 2, 3, 4)
print(s[:5])      # 'Hello'     (omit start -> from beginning)
print(s[7:])      # 'World!'    (omit stop -> to the end)
print(s[:])       # 'Hello, World!' (full copy)
print(s[-6:-1])   # 'World'     (negative indices)

# Using step
print(s[::2])     # 'Hlo ol!'   (every 2nd character)
print(s[1:10:2])  # 'el,Wr'

# Negative step reverses direction
print(s[::-1])    # '!dlroW ,olleH' (reverse string)
print(s[4::-1])   # 'olleH'

# Unlike single indexing, out-of-range slice bounds do NOT raise IndexError
print(s[7:100])   # 'World!'
print(s[50:100])  # ''
```

---

## 3. String Immutability

Python strings are **immutable**: once a string object is created in memory, its characters cannot be changed in place.

```python
word = "cat"

# Attempting in-place modification raises TypeError
# word[0] = "b"  # TypeError: 'str' object does not support item assignment
```

To "modify" a string, you construct a **new string** and optionally rebind the variable:

```python
word = "cat"
new_word = "b" + word[1:]
print(new_word)  # 'bat'
print(word)      # 'cat' (original is unchanged)

# Rebinding creates a new string object with a new identity
s = "hello"
old_id = id(s)
s = s.upper()
print(s)                 # 'HELLO'
print(id(s) != old_id)   # True
```

### Why Immutability Matters
- **Hashability:** Immutable strings can be used as `dict` keys and `set` elements.
- **Safety:** Passing strings to functions guarantees the caller's string won't be mutated unexpectedly.
- **Optimization:** Python can safely share/intern identical string literals in memory.

---

## 4. Escape Characters and Raw Strings

### Escape Characters

An **escape sequence** starts with a backslash (`\`) and represents special characters that are hard or impossible to type directly.

| Escape Sequence | Meaning | Example Output |
| :--- | :--- | :--- |
| `\\` | Literal backslash | `\` |
| `\'` | Single quote | `'` |
| `\"` | Double quote | `"` |
| `\n` | Newline (line feed) | Line break |
| `\t` | Horizontal tab | Tab spacing |
| `\r` | Carriage return | Moves cursor to line start |
| `\b` | Backspace | Removes previous char in terminal |
| `\xHH` | Character with hex value `HH` | `'\x41'` -> `'A'` |
| `\uHHHH` | 16-bit Unicode character | `'\u03B1'` -> `'α'` |
| `\UHHHHHHHH` | 32-bit Unicode character | `'\U0001F40D'` -> `'🐍'` |
| `\N{name}` | Unicode character by official name | `'\N{GREEK SMALL LETTER PI}'` -> `'π'` |

```python
print("First line\nSecond line")
print("Col1\tCol2\tCol3")
print("Path: C:\\Users\\Alice\\notes.txt")
print('It\'s working!')
print("\u00A9 2026 Python Notes")  # © 2026 Python Notes
```

### Raw Strings

Prefixing a string with `r` or `R` creates a **raw string**, where backslashes are treated as literal characters rather than escape sequences. Raw strings are ideal for **regular expressions** and **Windows file paths**.

```python
normal = "C:\new_folder\test.txt"
raw = r"C:\new_folder\test.txt"

print(normal)
# C:
# ew_folder	est.txt

print(raw)
# C:\new_folder\test.txt

# Raw strings for regex patterns
pattern = r"\d{3}-\d{2}-\d{4}"
```

> **Gotcha:** A raw string **cannot end with an odd number of backslashes**, because `\"` still escapes the closing quote during lexical parsing.
> ```python
> # invalid = r"C:\Users\Alice\"  # SyntaxError
> valid = r"C:\Users\Alice" "\\"   # 'C:\\Users\\Alice\\'
> ```

---

## 5. String Concatenation and Repetition

### Concatenation (`+` and `+=`)

```python
first = "Python"
second = "Programming"

full = first + " " + second
print(full)  # 'Python Programming'

# Augmented assignment creates a new string and rebinds 'msg'
msg = "Hello"
msg += ", World!"
print(msg)   # 'Hello, World!'

# Note: You cannot concatenate a str and non-str directly with +
# "Age: " + 25        # TypeError
print("Age: " + str(25))  # 'Age: 25'
```

### Repetition (`*`)

```python
print("Ha" * 3)      # 'HaHaHa'
print("-" * 20)      # '--------------------'
print(3 * "ab")      # 'ababab'
print("abc" * 0)     # ''
print("abc" * -2)    # ''
```

### Performance Note: `+` in Loops vs. `join()`

Because strings are immutable, repeated `+` concatenation inside a large loop allocates new strings repeatedly ($O(n^2)$ worst-case work). Prefer collecting parts in a list and calling `"".join(parts)` ($O(n)$):

```python
parts = []
for i in range(5):
    parts.append(str(i))
result = "-".join(parts)
print(result)  # '0-1-2-3-4'
```

---

## 6. String Methods

All string methods return **new values** and never modify the original string in place.

### Case Conversion Methods

| Method | Description | Example |
| :--- | :--- | :--- |
| `upper()` | All characters uppercase | `"hello".upper()` -> `'HELLO'` |
| `lower()` | All characters lowercase | `"HELLO".lower()` -> `'hello'` |
| `title()` | First letter of each word uppercase | `"hello world".title()` -> `'Hello World'` |
| `capitalize()` | First character uppercase, rest lowercase | `"hELLO world".capitalize()` -> `'Hello world'` |
| `swapcase()` | Swaps upper to lower and vice versa | `"PyThOn".swapcase()` -> `'pYtHoN'` |

```python
s = "pYtHoN pRoGrAmMiNg"

print(s.upper())       # 'PYTHON PROGRAMMING'
print(s.lower())       # 'python programming'
print(s.title())       # 'Python Programming'
print(s.capitalize())  # 'Python programming'
print(s.swapcase())    # 'PyThOn PrOgRaMmInG'

# Watch out for apostrophes with title():
print("they're here".title())  # "They'Re Here"
```

> **Note:** For Unicode case-insensitive comparisons (e.g., German `'ß'` -> `'ss'`), `str.casefold()` is even more thorough than `str.lower()`.

---

### Trimming Whitespace and Characters: `strip()`

- `strip([chars])`: removes leading and trailing whitespace (or any characters in `chars`).
- `lstrip([chars])` / `rstrip([chars])`: removes only from the left / right side.

```python
raw_input = "   \t  Hello, Python! \n  "
print(repr(raw_input.strip()))   # 'Hello, Python!'
print(repr(raw_input.lstrip()))  # 'Hello, Python! \n  '
print(repr(raw_input.rstrip()))  # '   \t  Hello, Python!'

# Passing custom characters strips ANY character in the set from both ends
url = "www.example.com"
print(url.strip("w.com"))        # 'example' (strips 'w', '.', 'c', 'o', 'm')
print("---hello---".strip("-"))  # 'hello'
```

> **Important:** `strip("abc")` treats `"abc"` as a **set of characters**, not an exact substring. To remove an exact prefix or suffix (Python 3.9+), use `removeprefix()` and `removesuffix()`:
> ```python
> "test_file.py".removeprefix("test_")  # 'file.py'
> "test_file.py".removesuffix(".py")    # 'test_file'
> ```

---

### Splitting, Partitioning, and Joining

#### `split()`
Splits a string into a list of substrings.
- Without arguments, `split()` splits on **any run of whitespace** and ignores leading/trailing whitespace.
- With a separator `sep`, it splits strictly on every occurrence of `sep` (preserving empty strings for consecutive separators).
- `maxsplit` limits the number of splits.

```python
sentence = "  one   two\tthree\nfour  "
print(sentence.split())         # ['one', 'two', 'three', 'four']

csv = "apple,banana,,cherry"
print(csv.split(","))           # ['apple', 'banana', '', 'cherry']
print(csv.split(",", 1))        # ['apple', 'banana,,cherry']
print("a.b.c".rsplit(".", 1))   # ['a.b', 'c'] (split from right)
```

#### `join()`
Called on the **separator** string to join an iterable of strings into one string.

```python
words = ["Python", "is", "awesome"]
print(" ".join(words))    # 'Python is awesome'
print("-".join(words))    # 'Python-is-awesome'
print("".join(words))     # 'Pythonisawesome'

# All elements in the iterable must be strings
nums = [1, 2, 3]
print(", ".join(map(str, nums)))  # '1, 2, 3'
```

#### `partition()`
Splits a string into a **3-tuple**: `(before, separator, after)` at the **first** occurrence of `sep`. If `sep` is not found, returns `(original_string, '', '')`. (`rpartition()` splits at the last occurrence.)

```python
email = "user@example.com"
print(email.partition("@"))       # ('user', '@', 'example.com')
print(email.partition(":"))       # ('user@example.com', '', '')

path = "/usr/local/bin/python"
print(path.rpartition("/"))       # ('/usr/local/bin', '/', 'python')
```

---

### Searching, Counting, and Replacing

#### `find()` and `count()`
- `find(sub[, start[, end]])`: returns the lowest index where `sub` is found, or **`-1`** if not found. (`rfind()` searches from the right; `index()` behaves like `find()` but raises `ValueError` if not found.)
- `count(sub[, start[, end]])`: returns the number of **non-overlapping** occurrences of `sub`.

```python
text = "banana bandana"

print(text.find("na"))       # 2
print(text.find("na", 4))    # 4 (start searching at index 4)
print(text.find("xyz"))      # -1
print(text.rfind("na"))      # 12

print(text.count("na"))      # 3
print(text.count("ana"))     # 2 ("ana" in "banana" and "bandana")
print("aaaa".count("aa"))    # 2 (non-overlapping)
```

#### `replace()`
`replace(old, new[, count])` replaces occurrences of `old` with `new` (optionally up to `count` times).

```python
msg = "one fish, two fish, red fish"
print(msg.replace("fish", "bird"))     # 'one bird, two bird, red bird'
print(msg.replace("fish", "bird", 2))  # 'one bird, two bird, red fish'
```

#### `startswith()` and `endswith()`
Check prefixes or suffixes; accept either a string or a **tuple of strings** (plus optional `start` and `end` indices).

```python
filename = "report_2026.pdf"

print(filename.startswith("report"))            # True
print(filename.endswith(".pdf"))                # True
print(filename.endswith((".pdf", ".docx")))     # True (tuple of options)
print(filename.startswith("2026", 7))           # True (starting at index 7)
```

---

### Character Classification Methods

These return `True` if the string is **non-empty** and all characters match the condition, otherwise `False`.

| Method | Condition | Examples (`True`) | Examples (`False`) |
| :--- | :--- | :--- | :--- |
| `isdigit()` | Digits (includes superscripts like `'²'`) | `"123"`, `"²³"` | `"-12"`, `"12.3"`, `""`, `"12a"` |
| `isalpha()` | Alphabetic letters (Unicode letters too) | `"Python"`, `"café"` | `"Python3"`, `"hi there"`, `""` |
| `isalnum()` | Letters or digits (no spaces/symbols) | `"Python3"`, `"abc"` | `"Python 3"`, `"a_b"`, `""` |

```python
print("42".isdigit())       # True
print("²".isdigit())        # True (note: use isdecimal() if you only want 0-9)
print("-42".isdigit())      # False ('-' is not a digit)

print("Hello".isalpha())    # True
print("Hello!".isalpha())   # False

print("User123".isalnum())  # True
print("User_123".isalnum()) # False ('_' is not alphanumeric)
print("".isalnum())         # False (empty string is always False)
```

> **Related methods:** `isdecimal()` (strictest base-10 digits), `isnumeric()` (also includes fractions like `'½'`), `isspace()` (whitespace only), `islower()`, `isupper()`, `istitle()`.

---

### Alignment and Padding Methods

- `center(width[, fillchar])`: centers the string within `width`.
- `ljust(width[, fillchar])`: left-justifies the string within `width`.
- `rjust(width[, fillchar])`: right-justifies the string within `width`.
- `zfill(width)`: pads with `'0'` on the left, **preserving a leading `+` or `-` sign**.

If the string is already `>= width`, it is returned unchanged.

```python
s = "Py"

print(repr(s.center(6)))       # '  Py  '
print(repr(s.center(7, "-")))  # '---Py--'
print(repr(s.ljust(5, ".")))   # 'Py...'
print(repr(s.rjust(5, ".")))   # '...Py'

# zfill handles signs cleanly
print("42".zfill(5))           # '00042'
print("-42".zfill(5))          # '-0042'
print("+42".zfill(5))          # '+0042'
print("-42".rjust(5, "0"))     # '00-42' (difference from zfill!)
```

---

### Character Translation: `maketrans()` and `translate()`

`str.maketrans()` builds a mapping table (dictionary of Unicode code point -> replacement/None) that `str.translate()` applies in a single fast pass.

`str.maketrans()` supports three signatures:
1. `str.maketrans(mapping_dict)`: keys are single characters or ordinals; values are replacement strings, ordinals, or `None` (to delete).
2. `str.maketrans(from_str, to_str)`: both strings must have equal length; maps each character in `from_str` to the corresponding character in `to_str`.
3. `str.maketrans(from_str, to_str, delete_str)`: also deletes every character in `delete_str`.

```python
# 1. Two-string mapping (e.g., simple substitution / DNA complement)
dna = "ATCG"
comp_table = str.maketrans("ATCG", "TAGC")
print(dna.translate(comp_table))  # 'TAGC'

# 2. Three-argument form: replace vowels and remove punctuation
clean_table = str.maketrans("aeiou", "12345", "!.,?")
print("Hello, World!".translate(clean_table))  # 'H2ll4 W4rld'

# 3. Dictionary form (allows 1-to-many replacements or deletions with None)
dict_table = str.maketrans({"&": " and ", "$": None})
print("Q&A $100".translate(dict_table))  # 'Q and A 100'
```

---

## 7. String Formatting

Python offers three main ways to format strings:

### 1. The `%` Operator (Legacy / printf-style)

```python
name = "Alice"
age = 30
score = 95.678

print("Name: %s, Age: %d" % (name, age))          # 'Name: Alice, Age: 30'
print("Score: %.2f" % score)                      # 'Score: 95.68'
print("%(user)s scored %(pts)d" % {"user": "Bob", "pts": 88})  # 'Bob scored 88'
```
Common specifiers: `%s` (string), `%d` (integer), `%f` (float), `%.2f` (2 decimal places), `%x` (hex).

---

### 2. The `.format()` Method

Introduced in Python 2.6/3.0; useful when formatting templates dynamically (e.g., stored in config files).

```python
# Positional and indexed arguments
print("{} is {} years old.".format("Alice", 30))
print("{1} comes before {0}".format("B", "A"))      # 'A comes before B'

# Keyword arguments
print("{lang} {ver}".format(lang="Python", ver=3.12))

# Format specifiers inside { :spec }
pi = 3.1415926
print("Pi = {:.2f}".format(pi))                     # 'Pi = 3.14'
print("Price: {:>8,.2f}".format(1234.5))            # 'Price: 1,234.50'
```

---

### 3. f-Strings (Formatted String Literals — Recommended)

Introduced in Python 3.6, f-strings (`f"..."` or `F"..."`) evaluate expressions directly inside `{}` at runtime. They are the **fastest** and **most readable** formatting method.

```python
name = "Alice"
age = 30
price = 1234.567

# Basic interpolation and inline expressions
print(f"{name} will be {age + 1} next year.")  # 'Alice will be 31 next year.'
print(f"Upper: {name.upper()}")                # 'Upper: ALICE'

# Number formatting
print(f"Price: ${price:,.2f}")                 # 'Price: $1,234.57'
print(f"Percent: {0.856:.1%}")                 # 'Percent: 85.6%'
print(f"Binary: {42:08b}, Hex: {42:#x}")       # 'Binary: 00101010, Hex: 0x2a'

# Alignment and padding: < (left), > (right), ^ (center)
print(f"|{'left':<10}|{'mid':^10}|{'right':>10}|")
# |left      |   mid    |     right|

print(f"{'Title':*^15}")                       # '*****Title*****'

# Debugging specifier '=' (Python 3.8+)
x = 10
y = 20
print(f"{x=}, {x + y=}")                       # 'x=10, x + y=30'

# Literal braces require doubling {{ and }}
print(f"{{key: {x}}}")                         # '{key: 10}'
```

| Feature | `%` Operator | `.format()` | f-Strings |
| :--- | :--- | :--- | :--- |
| Readability | Low for many args | Moderate | **Highest** |
| Performance | Moderate | Slower | **Fastest** |
| Reusable template string | Yes | **Yes** | No (evaluated immediately) |
| Inline expressions | No | Limited (attributes/indices) | **Full Python expressions** |

---

## 8. String Comparison

Strings are compared **lexicographically** (character by character from left to right) based on their **Unicode code points** (`ord(char)`).

```python
print("apple" == "apple")   # True
print("apple" != "Apple")   # True (case-sensitive)
print("apple" < "banana")   # True ('a' < 'b')
print("app" < "apple")      # True (shorter prefix is smaller)

# Uppercase ASCII letters (65-90) come BEFORE lowercase ASCII letters (97-122)
print(ord("Z"), ord("a"))   # 90 97
print("Zebra" < "apple")    # True! (because 'Z' (90) < 'a' (97))

# Case-insensitive comparison
print("Hello".casefold() == "hello".casefold())  # True
```

### Membership Testing (`in` and `not in`)

```python
sentence = "Python is fun"
print("Python" in sentence)      # True
print("java" not in sentence)    # True
```

### `==` vs. `is`

Always use `==` to compare string values, **not** `is` (which checks object identity in memory).

```python
a = "hello_world"
b = "".join(["hello", "_", "world"])

print(a == b)  # True  (same value)
print(a is b)  # False (different objects in memory)
```

---

## 9. String Iteration

Because strings are iterable sequences, you can loop over their characters directly.

```python
word = "Code"

# 1. Direct character iteration (most Pythonic)
for ch in word:
    print(ch, end=" ")       # C o d e
print()

# 2. With index using enumerate()
for idx, ch in enumerate(word):
    print(f"{idx}:{ch}", end=" ")  # 0:C 1:o 2:d 3:e
print()

# 3. Reverse iteration
for ch in reversed(word):
    print(ch, end="")        # edoC
print()

# 4. Pairwise iteration across two strings
for a, b in zip("ABC", "123"):
    print(f"{a}{b}", end=" ")  # A1 B2 C3
print()

# 5. Comprehensions over strings
vowels = [ch for ch in "education" if ch in "aeiou"]
print(vowels)                # ['e', 'u', 'a', 'i', 'o']
```

---

## 10. Multiline Strings

Triple quotes (`'''...'''` or `"""..."""`) preserve literal line breaks and whitespace.

```python
poem = """Roses are red,
Violets are blue,
Python is clean,
And concise too."""

print(poem)
```

### Avoiding Unwanted Indentation with `textwrap.dedent()`

Inside indented functions or classes, triple-quoted strings pick up leading spaces. Use `textwrap.dedent()` to strip common leading whitespace:

```python
import textwrap

def build_query():
    query = textwrap.dedent("""\
        SELECT id, name
        FROM users
        WHERE active = 1;
    """)
    return query

print(build_query())
# SELECT id, name
# FROM users
# WHERE active = 1;
```
*(Notice the trailing `\` after `"""` prevents an empty leading newline.)*

### Splitting Long Single-Line Strings Across Lines

If you want a **single line** of text in the result without embedded `\n`, use parentheses with implicit concatenation instead of triple quotes:

```python
long_message = (
    "This is a very long message "
    "that wraps cleanly in source code "
    "without adding newline characters."
)
print(long_message)
```

> **Note:** Triple-quoted strings placed as the first statement of a module, class, or function serve as **docstrings** (`__doc__`).

---

## 11. String Encoding (ASCII, Unicode, encode/decode)

In Python 3:
- `str` represents an abstract sequence of **Unicode code points** (characters).
- `bytes` represents a sequence of raw **8-bit integers** (`0` to `255`) used for files, network sockets, and binary protocols.

### ASCII and Unicode Helpers

- **ASCII** covers code points `0–127` (English letters, digits, basic punctuation).
- **Unicode** covers over 149,000 characters across all world scripts and emojis (`U+0000` to `U+10FFFF`).
- **UTF-8** is the dominant variable-length encoding (1 to 4 bytes per character; backward-compatible with ASCII).

```python
# ord(): character -> Unicode code point (int)
# chr(): Unicode code point (int) -> character
print(ord("A"))      # 65
print(ord("🐍"))     # 128013
print(chr(65))       # 'A'
print(chr(0x03C0))   # 'π'
```

### `encode()` and `decode()`

- `str.encode(encoding="utf-8", errors="strict")` converts `str` -> `bytes`.
- `bytes.decode(encoding="utf-8", errors="strict")` converts `bytes` -> `str`.

```python
text = "Café 🐍"

# Encode str -> bytes
utf8_bytes = text.encode("utf-8")
print(utf8_bytes)       # b'Caf\xc3\xa9 \xf0\x9f\x90\x8d'
print(len(text))        # 6 characters
print(len(utf8_bytes))  # 10 bytes ('é' is 2 bytes, '🐍' is 4 bytes)

# Decode bytes -> str
decoded = utf8_bytes.decode("utf-8")
print(decoded)          # 'Café 🐍'
```

### Error Handling Modes

When a character cannot be represented in the target encoding (e.g., encoding `'é'` or `'🐍'` as `'ascii'`), `errors` controls the behavior:

```python
sample = "Café 🐍"

# sample.encode("ascii")                      # UnicodeEncodeError (errors='strict')
print(sample.encode("ascii", "ignore"))           # b'Caf '
print(sample.encode("ascii", "replace"))          # b'Caf? ?'
print(sample.encode("ascii", "backslashreplace")) # b'Caf\\xe9 \\U0001f40d'

# Decoding invalid bytes
bad_bytes = b"hello\xffworld"
# bad_bytes.decode("utf-8")                   # UnicodeDecodeError
print(bad_bytes.decode("utf-8", errors="ignore"))   # 'helloworld'
print(bad_bytes.decode("utf-8", errors="replace"))  # 'hello\ufffdworld' (uses U+FFFD replacement char)
```

---

## 12. Regular Expressions Basics with Strings

Python's built-in `re` module provides pattern matching and text manipulation with regular expressions. Always use **raw strings** (`r"..."`) for regex patterns.

### Common `re` Functions

| Function | Purpose |
| :--- | :--- |
| `re.search(pattern, s)` | Finds the **first** match anywhere in `s` (returns `Match` or `None`) |
| `re.match(pattern, s)` | Matches **only at the beginning** of `s` |
| `re.fullmatch(pattern, s)` | Matches the **entire** string `s` |
| `re.findall(pattern, s)` | Returns a list of all non-overlapping matches |
| `re.finditer(pattern, s)` | Yields `Match` objects for every match |
| `re.sub(pattern, repl, s)` | Replaces matches with `repl` |
| `re.split(pattern, s)` | Splits `s` wherever `pattern` matches |
| `re.compile(pattern)` | Pre-compiles a pattern for repeated reuse |

### Essential Regex Tokens

- `\d` / `\D`: digit (`0–9`) / non-digit
- `\w` / `\W`: word character (letters, digits, `_`) / non-word
- `\s` / `\S`: whitespace / non-whitespace
- `.`: any character except newline (unless `re.DOTALL`)
- `^` / `$`: start / end of string (or line with `re.MULTILINE`)
- Quantifiers: `*` (0+), `+` (1+), `?` (0 or 1), `{m,n}` (between `m` and `n`)
- `[abc]`, `[^abc]`: character set / negated set
- `(...)`: capturing group; `(?:...)`: non-capturing group; `a|b`: alternation

### Practical Examples

```python
import re

log = "Order #1042 placed on 2026-11-26 by user_99; total: $49.95"

# 1. Search and extract groups
m = re.search(r"#(\d+).*?(\d{4}-\d{2}-\d{2})", log)
if m:
    print(m.group(0))  # '#1042 placed on 2026-11-26' (entire match)
    print(m.group(1))  # '1042' (1st capturing group)
    print(m.group(2))  # '2026-11-26' (2nd capturing group)

# 2. Find all numbers
print(re.findall(r"\d+(?:\.\d+)?", log))  # ['1042', '2026', '11', '26', '99', '49.95']

# 3. Validate full format
phone_pattern = re.compile(r"^\d{3}-\d{3}-\d{4}$")
print(bool(phone_pattern.fullmatch("555-123-4567")))  # True
print(bool(phone_pattern.fullmatch("555-1234")))      # False

# 4. Substitute / normalize text
messy = "Too    many   spaces\tand\nnewlines"
clean = re.sub(r"\s+", " ", messy)
print(clean)  # 'Too many spaces and newlines'

# 5. Case-insensitive matching with flags
print(re.findall(r"python", "Python, PYTHON, python", flags=re.IGNORECASE))
# ['Python', 'PYTHON', 'python']
```

---

## 13. Common String Operations and Patterns

### 1. Reversing a String & Palindrome Check

```python
s = "racecar"
rev = s[::-1]
is_palindrome = s.casefold() == rev.casefold()
print(is_palindrome)  # True

# Ignoring non-alphanumeric characters
phrase = "A man, a plan, a canal: Panama"
cleaned = "".join(ch.lower() for ch in phrase if ch.isalnum())
print(cleaned == cleaned[::-1])  # True
```

### 2. Collapsing Extra Whitespace

```python
text = "   Python   is   really   neat   "
normalized = " ".join(text.split())
print(normalized)  # 'Python is really neat'
```

### 3. Character Frequency & Anagram Check

```python
from collections import Counter

word1, word2 = "listen", "silent"
print(Counter(word1))                  # Counter({'l': 1, 'i': 1, 's': 1, 't': 1, 'e': 1, 'n': 1})
print(Counter(word1) == Counter(word2))  # True
```

### 4. Removing Punctuation

```python
import string

text = "Hello, world! Ready? Set... Go!"
no_punct = text.translate(str.maketrans("", "", string.punctuation))
print(no_punct)  # 'Hello world Ready Set Go'
```

### 5. Parsing Key-Value Pairs Safely

Using `partition("=")` or `split("=", 1)` ensures values containing `=` aren't broken:

```python
line = "TOKEN=abc=123==xyz"
key, sep, value = line.partition("=")
print(key, value)  # TOKEN abc=123==xyz
```

### 6. Masking Sensitive Information

```python
card = "4111222233334444"
masked = "*" * (len(card) - 4) + card[-4:]
print(masked)  # '************4444'
```

### 7. Truncating Long Strings with an Ellipsis

```python
def truncate(text: str, max_len: int) -> str:
    if len(text) <= max_len:
        return text
    return text[:max_len - 3] + "..."

print(truncate("Introduction to Python Strings", 15))  # 'Introduction...'
```

### 8. Building a URL Slug

```python
import re

def slugify(title: str) -> str:
    slug = title.lower().strip()
    slug = re.sub(r"[^a-z0-9\s-]", "", slug)
    slug = re.sub(r"[\s-]+", "-", slug)
    return slug.strip("-")

print(slugify("  Python 3: Strings & Regex! "))  # 'python-3-strings-regex'
```
