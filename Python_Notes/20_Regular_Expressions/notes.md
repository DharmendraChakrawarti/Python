# Regular Expressions (`re` Module) in Python

Regular expressions (commonly called **regex** or **regexp**) are a concise, powerful mini-language for describing patterns in text. Python's built-in `re` module provides full support for Perl-style regular expressions to search, match, extract, validate, split, and transform strings.

---

## Table of Contents

1. [What Are Regular Expressions?](#1-what-are-regular-expressions)
2. [`re` Module Overview](#2-re-module-overview)
3. [`re.match()`, `re.search()`, `re.findall()`, `re.finditer()`](#3-rematch-research-refindall-refinditer)
4. [`re.sub()`, `re.split()`, `re.compile()`](#4-resub-resplit-recompile)
5. [Match Object Methods (`group`, `groups`, `start`, `end`, `span`)](#5-match-object-methods-group-groups-start-end-span)
6. [Metacharacters (`. ^ $ * + ? {} [] \ | ()`)](#6-metacharacters------------)
7. [Character Classes (`[abc]`, `[a-z]`, `[^abc]`)](#7-character-classes-abc-a-z-abc)
8. [Special Sequences (`\d`, `\D`, `\w`, `\W`, `\s`, `\S`, `\b`, `\B`)](#8-special-sequences-d-d-w-w-s-s-b-b)
9. [Quantifiers (`*`, `+`, `?`, `{n}`, `{n,}`, `{n,m}`)](#9-quantifiers----n-n-nm)
10. [Greedy vs Non-Greedy Matching](#10-greedy-vs-non-greedy-matching)
11. [Grouping and Capturing (`()`, `(?:)`, `(?P<name>)`)](#11-grouping-and-capturing---pname)
12. [Lookahead and Lookbehind Assertions](#12-lookahead-and-lookbehind-assertions)
13. [Flags (`re.IGNORECASE`, `re.MULTILINE`, `re.DOTALL`, `re.VERBOSE`)](#13-flags-reignorecase-remultiline-redotall-reverbose)
14. [Common Regex Patterns (Email, Phone, URL, IP Address, Date)](#14-common-regex-patterns-email-phone-url-ip-address-date)
15. [Best Practices and Performance Tips](#15-best-practices-and-performance-tips)

---

## 1. What Are Regular Expressions?

A **regular expression** is a sequence of characters that defines a search pattern. Instead of checking for an exact literal substring (like `"cat" in text`), regex lets you describe *shapes* of text—such as "any four-digit year", "an email address", or "words starting with a capital letter".

### Why Use Regular Expressions?

- **Validation**: Verify whether input matches an expected format (dates, postal codes, usernames).
- **Searching**: Locate substrings matching flexible patterns in logs, documents, or HTML.
- **Extraction**: Pull structured pieces (like IDs, prices, or timestamps) out of unstructured text.
- **Transformation**: Find and replace complex patterns cleanly (masking sensitive data, normalizing whitespace).

### Always Use Raw Strings (`r"..."`)

In Python string literals, backslashes (`\`) start escape sequences such as `\n` (newline) or `\t` (tab). Regular expressions also use backslashes heavily (`\d`, `\s`, `\b`). Writing patterns as **raw strings** (`r"..."`) ensures Python passes backslashes directly to the regex engine without interpreting them first.

```python
import re

# Without raw string: Python interprets \b as ASCII backspace (0x08)!
bad_pattern = "\bword\b"

# With raw string: regex engine receives literal \bword\b (word boundaries)
good_pattern = r"\bword\b"

text = "A word in a sentence."
print(re.search(bad_pattern, text))   # None
print(re.search(good_pattern, text))  # <re.Match object; span=(2, 6), match='word'>
```

---

## 2. `re` Module Overview

Python includes the `re` module in its standard library—no third-party installation is required.

```python
import re
```

### Core Functions at a Glance

| Function / Class | Purpose | Return Value |
| :--- | :--- | :--- |
| `re.match(pattern, string)` | Match pattern **only at the start** of `string` | `Match` object or `None` |
| `re.fullmatch(pattern, string)` | Match pattern against the **entire** `string` | `Match` object or `None` |
| `re.search(pattern, string)` | Find the **first** match anywhere in `string` | `Match` object or `None` |
| `re.findall(pattern, string)` | Find **all** non-overlapping matches | `list` of `str` (or `tuple` if groups exist) |
| `re.finditer(pattern, string)` | Iterate lazily over **all** non-overlapping matches | Iterator of `Match` objects |
| `re.sub(pattern, repl, string)` | Replace matches with `repl` (string or callable) | Modified `str` |
| `re.subn(pattern, repl, string)` | Same as `sub`, also returns replacement count | `(new_str, count)` tuple |
| `re.split(pattern, string)` | Split `string` wherever `pattern` matches | `list` of `str` |
| `re.compile(pattern, flags=0)` | Pre-compile pattern into a reusable `Pattern` object | `re.Pattern` object |
| `re.escape(pattern)` | Escape all regex metacharacters in a literal string | Escaped `str` |

---

## 3. `re.match()`, `re.search()`, `re.findall()`, `re.finditer()`

### `re.match()`

Checks for a match **only at index `0`** of the string (even with `re.MULTILINE`). If the pattern occurs later in the string, `re.match()` returns `None`.

```python
import re

text = "Python 3.12 was released"

m1 = re.match(r"Python", text)
print(m1)  # <re.Match object; span=(0, 6), match='Python'>

m2 = re.match(r"3\.12", text)
print(m2)  # None (not at the beginning)
```

> **Tip**: If you want to verify the *entire* string matches a pattern, use `re.fullmatch(pattern, text)` rather than `re.match()` (which only anchors the start, not the end).

```python
import re

print(re.match(r"\d+", "123abc"))      # <re.Match object; span=(0, 3), match='123'>
print(re.fullmatch(r"\d+", "123abc"))  # None
print(re.fullmatch(r"\d+", "123"))     # <re.Match object; span=(0, 3), match='123'>
```

### `re.search()`

Scans through the entire string and returns a `Match` object for the **first** location where the pattern matches, or `None` if no match exists.

```python
import re

text = "Order #4821 shipped on 2026-11-26"
match = re.search(r"#(\d+)", text)

if match:
    print("Found:", match.group(0))  # Found: #4821
    print("Order ID:", match.group(1))  # Order ID: 4821
```

### `re.findall()`

Returns a **list** of all non-overlapping matches from left to right.

**Important behavior with capturing groups**:
- If the pattern has **no capturing groups**, `findall()` returns a list of matched strings.
- If the pattern has **1 capturing group**, `findall()` returns a list of that group's strings.
- If the pattern has **2 or more capturing groups**, `findall()` returns a list of tuples of strings.

```python
import re

text = "Apples: 12, Bananas: 5, Cherries: 30"

# No capturing groups -> list of full matches
print(re.findall(r"\d+", text))
# ['12', '5', '30']

# One capturing group -> list of group 1 strings
print(re.findall(r"(\w+): \d+", text))
# ['Apples', 'Bananas', 'Cherries']

# Multiple capturing groups -> list of tuples
print(re.findall(r"(\w+): (\d+)", text))
# [('Apples', '12'), ('Bananas', '5'), ('Cherries', '30')]
```

### `re.finditer()`

Returns an **iterator** yielding `Match` objects for every non-overlapping match. This is memory-efficient for large texts and gives full access to match positions (`.start()`, `.end()`, `.span()`) and named groups.

```python
import re

log = "ERROR at 10:15, WARN at 10:20, ERROR at 10:45"

for match in re.finditer(r"(ERROR|WARN) at (\d{2}:\d{2})", log):
    level, timestamp = match.groups()
    print(f"{level} @ {timestamp} (pos {match.start()}-{match.end()})")

# Output:
# ERROR @ 10:15 (pos 0-14)
# WARN @ 10:20 (pos 16-29)
# ERROR @ 10:45 (pos 31-44)
```

---

## 4. `re.sub()`, `re.split()`, `re.compile()`

### `re.sub(pattern, repl, string, count=0, flags=0)`

Replaces occurrences of `pattern` in `string` with `repl`. By default (`count=0`), replaces **all** matches.

#### Replacing with a String (and Backreferences)

```python
import re

text = "Contact: 555-123-4567 or 555-987-6543"

# Mask phone numbers
masked = re.sub(r"\d{3}-\d{3}-(\d{4})", r"***-***-\1", text)
print(masked)
# Contact: ***-***-4567 or ***-***-6543

# Replace only the first match
first_only = re.sub(r"\d+", "X", "a1 b2 c3", count=1)
print(first_only)
# aX b2 c3
```

#### Replacing with a Function (`repl` Callable)

When `repl` is a function, it receives each `Match` object and returns the replacement string. This enables dynamic transformations.

```python
import re

def double_number(match: re.Match) -> str:
    value = int(match.group(0))
    return str(value * 2)

result = re.sub(r"\d+", double_number, "Items: 3, price: 15")
print(result)
# Items: 6, price: 30
```

> **Note**: `re.subn()` works identically to `re.sub()`, except it returns `(new_string, num_replacements)`.

### `re.split(pattern, string, maxsplit=0, flags=0)`

Splits `string` at every match of `pattern`, similar to `str.split()` but supporting arbitrary regex delimiters.

```python
import re

# Split on commas, semicolons, or whitespace
data = "apple, banana;  cherry\torange"
items = re.split(r"[,;\s]+", data)
print(items)
# ['apple', 'banana', 'cherry', 'orange']

# Limit splits with maxsplit
print(re.split(r"\s+", "one two three four", maxsplit=2))
# ['one', 'two', 'three four']

# Capturing groups in the delimiter are preserved in the output list!
expr = "10+20-5*2"
tokens = re.split(r"([+\-*/])", expr)
print(tokens)
# ['10', '+', '20', '-', '5', '*', '2']
```

### `re.compile(pattern, flags=0)`

Compiles a regex string into a `re.Pattern` object. Compiling is useful when reusing a pattern many times (e.g., inside a loop or module-level constant), and keeps code clean and self-documenting.

```python
import re

EMAIL_RE = re.compile(r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$")

emails = ["alice@example.com", "invalid@", "bob.smith@corp.co.uk"]
valid = [e for e in emails if EMAIL_RE.match(e)]
print(valid)
# ['alice@example.com', 'bob.smith@corp.co.uk']
```

Compiled `Pattern` objects also support optional `pos` and `endpos` slicing arguments on `.search()`, `.match()`, `.findall()`, etc.:

```python
pattern = re.compile(r"\d+")
print(pattern.search("id=123, code=456", pos=7))
# <re.Match object; span=(13, 16), match='456'>
```

---

## 5. Match Object Methods (`group`, `groups`, `start`, `end`, `span`)

When `match()`, `fullmatch()`, `search()`, or `finditer()` succeeds, it returns a `re.Match` object.

```python
import re

text = "Date: 2026-11-26 (Thursday)"
m = re.search(r"(?P<year>\d{4})-(\d{2})-(\d{2})", text)

if m:
    # group(0) or group() -> entire matched substring
    print("Full match:", m.group(0))        # '2026-11-26'
    print("Group 1 (year):", m.group(1))    # '2026'
    print("Named group:", m.group("year"))  # '2026'
    print("Multiple:", m.group(1, 3))       # ('2026', '26')

    # groups() -> tuple of all captured groups (excluding group 0)
    print("All groups:", m.groups())        # ('2026', '11', '26')

    # groupdict() -> dict of named capturing groups
    print("Named dict:", m.groupdict())     # {'year': '2026'}

    # start(), end(), span() -> character indices in original string
    print("Start:", m.start())              # 6
    print("End:", m.end())                  # 16
    print("Span:", m.span())                # (6, 16)

    # Indices for a specific group
    print("Group 2 span:", m.span(2))       # (11, 13)
    print("Group 2 start:", m.start(2))     # 11
    print("Group 2 end:", m.end(2))         # 13
```

### Summary of Match Object Methods

| Method | Description |
| :--- | :--- |
| `m.group()` / `m.group(0)` | The entire matched string |
| `m.group(n)` / `m.group("name")` | Substring captured by group `n` or named group `"name"` |
| `m.group(1, 2, ...)` | Tuple of substrings for the requested groups |
| `m.groups(default=None)` | Tuple of all captured groups (`default` used if an optional group did not participate) |
| `m.groupdict(default=None)` | Dictionary mapping named group names to matched substrings |
| `m.start(group=0)` | Starting index of the match (or specified group); `-1` if group didn't match |
| `m.end(group=0)` | Ending index (exclusive) of the match (or specified group); `-1` if group didn't match |
| `m.span(group=0)` | `(start(group), end(group))` tuple |

---

## 6. Metacharacters (`. ^ $ * + ? {} [] \ | ()`)

Metacharacters have special meaning in regular expressions instead of matching themselves literally. To match a metacharacter literally, escape it with a backslash (`\.`, `\$`, `\\`, etc.) or use `re.escape()`.

| Metacharacter | Meaning | Example | Matches |
| :--- | :--- | :--- | :--- |
| `.` | Any character except `\n` (unless `re.DOTALL` is set) | `r"c.t"` | `"cat"`, `"cut"`, `"c9t"` |
| `^` | Start of string (or start of each line with `re.MULTILINE`) | `r"^Hello"` | `"Hello world"` (not `"Say Hello"`) |
| `$` | End of string (or just before `\n` at end; end of each line with `re.MULTILINE`) | `r"end$"` | `"the end"` (not `"end here"`) |
| `*` | 0 or more repetitions of the preceding token | `r"ab*c"` | `"ac"`, `"abc"`, `"abbbc"` |
| `+` | 1 or more repetitions of the preceding token | `r"ab+c"` | `"abc"`, `"abbbc"` (not `"ac"`) |
| `?` | 0 or 1 repetition (optional), or makes a quantifier non-greedy | `r"colou?r"` | `"color"`, `"colour"` |
| `{...}` | Explicit repetition count (`{n}`, `{n,}`, `{n,m}`) | `r"\d{4}"` | `"2026"` |
| `[...]` | Character class (matches any single character inside) | `r"[aeiou]"` | `"a"`, `"e"`, `"i"`, `"o"`, `"u"` |
| `\` | Escapes metacharacters or introduces special sequences (`\d`, `\s`, etc.) | `r"\$10\.00"` | `"$10.00"` |
| `\|` | Alternation (logical OR) | `r"cat\|dog"` | `"cat"` or `"dog"` |
| `(...)` | Grouping and capturing | `r"(ha)+"` | `"ha"`, `"haha"`, `"hahaha"` |

```python
import re

# Escaping metacharacters to match literal characters
price_text = "Total: $19.99 (tax included)"
print(re.search(r"\$\d+\.\d{2}", price_text).group())  # '$19.99'

# Alternation with grouping
print(re.findall(r"file\.(?:txt|csv|json)", "a: file.txt, b: file.png, c: file.csv"))
# ['file.txt', 'file.csv']
```

---

## 7. Character Classes (`[abc]`, `[a-z]`, `[^abc]`)

A **character class** (character set) matches **one single character** from the defined set.

### Basic Character Class: `[abc]`

Matches any single character listed inside the brackets.

```python
import re

print(re.findall(r"gr[ae]y", "gray grey grog"))
# ['gray', 'grey']
```

### Ranges: `[a-z]`, `[A-Z]`, `[0-9]`

Use a hyphen `-` to define a range of characters (by Unicode code point order). Multiple ranges can be combined in one class.

```python
import re

# Lowercase letters
print(re.findall(r"[a-z]+", "Hello 123 World"))
# ['ello', 'orld']

# Hexadecimal digits (0-9, a-f, A-F)
print(re.findall(r"[0-9a-fA-F]+", "Num: #1A2b3C, ok: XYZ"))
# ['1A2b3C']
```

> **Note**: To include a literal `-` inside `[...]`, place it at the very start or end (e.g., `[-a-z]` or `[a-z-]`), or escape it (`\-`).

### Negated Character Class: `[^abc]`

When `^` is the **first** character inside `[...]`, it negates the set—matching any single character **not** in the brackets.

```python
import re

# Match anything except digits
print(re.findall(r"[^0-9]+", "abc123def45"))
# ['abc', 'def']

# Match anything except vowels and spaces
print(re.findall(r"[^aeiou ]", "hello world"))
# ['h', 'l', 'l', 'w', 'r', 'l', 'd']
```

### Metacharacters Inside `[...]`

Inside character classes, most metacharacters (`.`, `*`, `+`, `?`, `(`, `)`, `$`, `|`) lose their special meaning and match literally! Only `\`, `]`, `^` (at start), and `-` (between characters) are special.

```python
import re

# Matches literal dot, plus, or question mark
print(re.findall(r"[.+?]", "a.b+c?d"))
# ['.', '+', '?']
```

---

## 8. Special Sequences (`\d`, `\D`, `\w`, `\W`, `\s`, `\S`, `\b`, `\B`)

Special sequences provide shorthand for common character sets and zero-width position assertions.

| Sequence | Meaning (Unicode default in Python 3) | Equivalent (ASCII-only with `re.ASCII`) |
| :--- | :--- | :--- |
| `\d` | Any Unicode decimal digit | `[0-9]` |
| `\D` | Any character that is **not** a digit | `[^0-9]` |
| `\w` | Any Unicode word character (letters, digits, `_`) | `[a-zA-Z0-9_]` |
| `\W` | Any character that is **not** a word character | `[^a-zA-Z0-9_]` |
| `\s` | Any whitespace character (` `, `\t`, `\n`, `\r`, `\f`, `\v`, etc.) | `[ \t\n\r\f\v]` |
| `\S` | Any non-whitespace character | `[^ \t\n\r\f\v]` |
| `\b` | **Word boundary** (between `\w` and `\W`, or start/end of string next to `\w`) | Zero-width assertion |
| `\B` | **Non-word boundary** (everywhere `\b` does not match) | Zero-width assertion |

*(Also useful: `\A` matches strictly the start of the string and `\Z` matches strictly the end of the string, unaffected by `re.MULTILINE`.)*

### Examples

```python
import re

text = "User_1: café costs $42\n"

print(re.findall(r"\d+", text))   # ['1', '42']
print(re.findall(r"\w+", text))   # ['User_1', 'café', 'costs', '42']
print(re.findall(r"\S+", text))   # ['User_1:', 'café', 'costs', '$42']

# ASCII-only \w ignores accented characters like 'é'
print(re.findall(r"\w+", text, flags=re.ASCII))
# ['User_1', 'caf', 'costs', '42']
```

### Word Boundaries (`\b` vs `\B`)

`\b` matches a *position* (zero characters wide), not a character itself. It is essential for whole-word matching.

```python
import re

sentence = "The cat scattered the caterpillar's food."

# Without \b: matches 'cat' inside 'scattered' and 'caterpillar'
print(re.findall(r"cat", sentence))
# ['cat', 'cat', 'cat']

# With \b: matches only the standalone word 'cat'
print(re.findall(r"\bcat\b", sentence))
# ['cat']

# \B matches 'cat' when NOT at a word boundary on the left/right
print(re.findall(r"\Bcat\B", sentence))
# ['cat']  (from 'scattered')
```

---

## 9. Quantifiers (`*`, `+`, `?`, `{n}`, `{n,}`, `{n,m}`)

Quantifiers specify how many times the immediately preceding character, character class, or group must repeat.

| Quantifier | Meaning | Example | Matches |
| :--- | :--- | :--- | :--- |
| `*` | 0 or more times | `r"go*gle"` | `"ggle"`, `"gogle"`, `"google"` |
| `+` | 1 or more times | `r"go+gle"` | `"gogle"`, `"google"` (not `"ggle"`) |
| `?` | 0 or 1 time (optional) | `r"https?"` | `"http"`, `"https"` |
| `{n}` | Exactly `n` times | `r"\d{4}"` | `"2026"` |
| `{n,}` | `n` or more times | `r"\d{2,}"` | `"12"`, `"123"`, `"12345"` |
| `{n,m}` | Between `n` and `m` times (inclusive) | `r"\d{2,4}"` | `"12"`, `"123"`, `"1234"` |

```python
import re

# Validate a 4-to-8 character alphanumeric username
pattern = re.compile(r"^[a-zA-Z0-9_]{4,8}$")

for name in ["abc", "user_1", "longusername", "alice99"]:
    print(f"{name:14} -> {bool(pattern.match(name))}")

# Output:
# abc            -> False
# user_1         -> True
# longusername   -> False
# alice99        -> True
```

---

## 10. Greedy vs Non-Greedy Matching

By default, all quantifiers (`*`, `+`, `?`, `{n,m}`) are **greedy**: they consume as many characters as possible while still allowing the rest of the pattern to match.

Adding a trailing `?` (`*?`, `+?`, `??`, `{n,m}?`) makes the quantifier **non-greedy (lazy)**: it consumes as *few* characters as possible while still satisfying the match.

### Classic Example: HTML/XML Tags

```python
import re

html = "<b>bold</b> and <i>italic</i>"

# Greedy: .* eats everything from the first '<' to the last '>'
print(re.findall(r"<.*>", html))
# ['<b>bold</b> and <i>italic</i>']

# Non-greedy: .*? stops at the earliest '>'
print(re.findall(r"<.*?>", html))
# ['<b>', '</b>', '<i>', '</i>']
```

### Comparing Greedy vs Lazy Quantifiers

| Greedy | Non-Greedy (Lazy) | Behavior |
| :--- | :--- | :--- |
| `*` | `*?` | 0 or more (fewest possible) |
| `+` | `+?` | 1 or more (fewest possible) |
| `?` | `??` | 0 or 1 (prefers 0) |
| `{n,m}` | `{n,m}?` | `n` to `m` (prefers `n`) |

```python
import re

text = "12345"
print(re.search(r"\d{2,4}", text).group())   # '1234' (greedy: takes max 4)
print(re.search(r"\d{2,4}?", text).group())  # '12'   (lazy: takes min 2)
```

> **Performance tip**: Instead of `r"<.*?>"` (which backtracks), negated character classes like `r"<[^>]*>"` are often faster and clearer because they can never cross `>`.

---

## 11. Grouping and Capturing (`()`, `(?:)`, `(?P<name>)`)

Parentheses `(...)` serve two roles:
1. **Grouping** tokens together so quantifiers or alternation apply to the whole sub-expression.
2. **Capturing** the matched text for extraction or backreferencing.

### Capturing Group: `(...)`

Captures the substring matched inside the parentheses, numbered left-to-right starting at `1` (by opening parenthesis).

```python
import re

m = re.search(r"(\w+)@(\w+\.\w+)", "Email: alice@example.com")
print(m.group(0))  # 'alice@example.com'
print(m.group(1))  # 'alice'
print(m.group(2))  # 'example.com'
```

### Non-Capturing Group: `(?:...)`

Groups tokens for quantifiers or alternation **without** creating a capture group. This avoids cluttering `.groups()` and slightly improves performance.

```python
import re

# Non-capturing group for alternation, capturing group for domain
pattern = r"(?:http|https)://([^\s/]+)"
print(re.findall(pattern, "Visit https://python.org and http://example.com"))
# ['python.org', 'example.com']
```

### Named Capturing Group: `(?P<name>...)`

Assigns a readable name to a group, accessible via `m.group("name")` or `m.groupdict()`.

```python
import re

log_line = "2026-11-26 [ERROR] Connection timed out"
pattern = r"(?P<date>\d{4}-\d{2}-\d{2}) \[(?P<level>\w+)\] (?P<msg>.+)"

m = re.match(pattern, log_line)
if m:
    print(m.group("level"))  # 'ERROR'
    print(m.groupdict())
    # {'date': '2026-11-26', 'level': 'ERROR', 'msg': 'Connection timed out'}
```

### Backreferences (`\1`, `(?P=name)`)

Match the exact same text previously captured by a group:

```python
import re

# Find repeated consecutive words (e.g., "the the")
text = "This is a a test of the the regex."
print(re.findall(r"\b(\w+)\s+\1\b", text))
# ['a', 'the']

# Backreference to a named group
html = "<h1>Title</h1> <p>Bad</div>"
print(re.findall(r"<(?P<tag>\w+)>.*?</(?P=tag)>", html))
# ['h1']  (only valid matching open/close tag)
```

*(In `re.sub()` replacement strings, refer to named groups with `\g<name>` and numbered groups with `\1` or `\g<1>`.)*

---

## 12. Lookahead and Lookbehind Assertions

**Lookarounds** are zero-width assertions: they check whether a pattern matches **before** or **after** the current position **without consuming any characters** in the match.

| Syntax | Name | Meaning |
| :--- | :--- | :--- |
| `(?=...)` | Positive Lookahead | Succeeds if `...` matches immediately **after** current position |
| `(?!...)` | Negative Lookahead | Succeeds if `...` does **not** match immediately **after** current position |
| `(?<=...)` | Positive Lookbehind | Succeeds if `...` matches immediately **before** current position |
| `(?<!...)` | Negative Lookbehind | Succeeds if `...` does **not** match immediately **before** current position |

### Examples

```python
import re

text = "Prices: $50, €30, $120, £15"

# Positive lookbehind: digits preceded by '$', without including '$' in the match
print(re.findall(r"(?<=\$)\d+", text))
# ['50', '120']

# Negative lookbehind: digits NOT preceded by '$' or another digit
print(re.findall(r"(?<![\$\d])\b\d+", text))
# ['30', '15']

# Positive lookahead: word followed by 'px'
sizes = "width: 100px; height: 200em; margin: 16px;"
print(re.findall(r"\d+(?=px)", sizes))
# ['100', '16']

# Negative lookahead: numbers NOT followed by 'px'
print(re.findall(r"\b\d+(?!\d|px)", sizes))
# ['200']
```

### Practical Use Case: Password Strength Validation

Multiple lookaheads from the start of the string (`^`) check independent rules simultaneously:

```python
import re

# At least 8 chars, 1 lowercase, 1 uppercase, 1 digit
PASSWORD_RE = re.compile(r"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$")

print(bool(PASSWORD_RE.match("weakpass")))    # False
print(bool(PASSWORD_RE.match("Strong1")))     # False (only 7 chars)
print(bool(PASSWORD_RE.match("StrongPass1"))) # True
```

> **Python `re` Limitation on Lookbehind**: Lookbehind patterns (`(?<=...)` and `(?<!...)`) in Python's standard `re` module must match a **fixed width** (no variable quantifiers like `*`, `+`, or `{1,3}` inside the lookbehind; fixed alternatives of equal length like `(?<=cat|dog)` are allowed). If you need variable-length lookbehinds, consider the third-party `regex` library or restructuring the pattern with capturing groups.

---

## 13. Flags (`re.IGNORECASE`, `re.MULTILINE`, `re.DOTALL`, `re.VERBOSE`)

Flags modify how the regex engine interprets a pattern. Combine multiple flags using the bitwise OR operator `|`.

| Flag | Short Alias | Description |
| :--- | :--- | :--- |
| `re.IGNORECASE` | `re.I` | Case-insensitive matching (`"a"` matches `"a"` and `"A"`) |
| `re.MULTILINE` | `re.M` | `^` matches the start of **each line**; `$` matches the end of **each line** |
| `re.DOTALL` | `re.S` | `.` matches **every** character, including `\n` |
| `re.VERBOSE` | `re.X` | Allows whitespace and `# comments` inside patterns for readability |
| `re.ASCII` | `re.A` | Makes `\w`, `\W`, `\b`, `\B`, `\d`, `\D`, `\s`, `\S` ASCII-only |

### Examples

#### `re.IGNORECASE` (`re.I`)

```python
import re

print(re.findall(r"python", "Python, PYTHON, python", flags=re.IGNORECASE))
# ['Python', 'PYTHON', 'python']
```

#### `re.MULTILINE` (`re.M`)

```python
import re

text = "first line\nsecond line\nthird line"

# Without MULTILINE: ^ only matches index 0
print(re.findall(r"^\w+", text))
# ['first']

# With MULTILINE: ^ matches after every newline
print(re.findall(r"^\w+", text, flags=re.MULTILINE))
# ['first', 'second', 'third']
```

#### `re.DOTALL` (`re.S`)

```python
import re

block = "/* line 1\n   line 2 */"

print(re.search(r"/\*.*\*/", block))
# None (because '.' does not match '\n')

print(re.search(r"/\*.*\*/", block, flags=re.DOTALL).group())
# '/* line 1\n   line 2 */'
```

#### `re.VERBOSE` (`re.X`)

Allows formatting complex patterns across multiple lines with comments. (Remember: literal spaces inside `re.VERBOSE` are ignored, so match spaces explicitly with `\ ` or `[ ]` or `\s`.)

```python
import re

PHONE_RE = re.compile(r"""
    ^                       # Start of string
    (?:\+?1[-.\s]?)?        # Optional country code (+1)
    \(?(\d{3})\)?           # Area code (with or without parentheses)
    [-.\s]?                 # Optional separator
    (\d{3})                 # First 3 digits
    [-.\s]?                 # Optional separator
    (\d{4})                 # Last 4 digits
    $                       # End of string
""", re.VERBOSE | re.IGNORECASE)

m = PHONE_RE.match("+1 (555) 234-5678")
print(m.groups())  # ('555', '234', '5678')
```

---

## 14. Common Regex Patterns (Email, Phone, URL, IP Address, Date)

Below are practical patterns for everyday parsing and validation tasks, along with important caveats.

### 1. Email Address (Practical Validation)

*(Full RFC 5322 compliance is notoriously complex; for most applications, a clean practical pattern plus sending a confirmation email is best.)*

```python
import re

EMAIL_RE = re.compile(r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$")

print(bool(EMAIL_RE.match("jane.doe+work@example.org")))  # True
print(bool(EMAIL_RE.match("bad-email@.com")))             # False
```

### 2. Phone Number (US / North American Format)

Matches formats like `555-123-4567`, `(555) 123-4567`, `+1 555.123.4567`, `5551234567`:

```python
import re

PHONE_RE = re.compile(r"(?:\+?1[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}")

text = "Call (800) 555-0199 or +1-415-555-0100 today."
print(PHONE_RE.findall(text))
# ['(800) 555-0199', '+1-415-555-0100']
```

### 3. URL (HTTP/HTTPS)

```python
import re

URL_RE = re.compile(
    r"https?://(?:www\.)?[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)+(?:/[^\s]*)?"
)

text = "Docs at https://docs.python.org/3/library/re.html or http://example.com."
print(URL_RE.findall(text))
# ['https://docs.python.org/3/library/re.html', 'http://example.com']
```

*(Note: when extracting URLs from prose, trailing punctuation like `.` or `,` may need stripping or exclusion, e.g., using `(?:/[^\s.,;!?]*)?` or `urllib.parse` for strict URL parsing.)*

### 4. IPv4 Address (Strict `0–255` Octet Range)

A naive `\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}` matches invalid IPs like `999.999.999.999`. To enforce `0–255` per octet:

```python
import re

OCTET = r"(?:25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)"
IPV4_RE = re.compile(rf"\b{OCTET}\.{OCTET}\.{OCTET}\.{OCTET}\b")

for ip in ["192.168.1.1", "255.255.255.0", "256.100.1.1", "10.0.0.256"]:
    print(f"{ip:16} -> {bool(IPV4_RE.fullmatch(ip))}")

# Output:
# 192.168.1.1      -> True
# 255.255.255.0    -> True
# 256.100.1.1      -> False
# 10.0.0.256       -> False
```

*(For production IP validation supporting both IPv4 and IPv6, Python's built-in `ipaddress.ip_address()` is even cleaner.)*

### 5. Date (`YYYY-MM-DD`)

```python
import re

DATE_RE = re.compile(r"\b(?P<year>\d{4})-(?P<month>0[1-9]|1[0-2])-(?P<day>0[1-9]|[12]\d|3[01])\b")

text = "Events: 2026-11-26, invalid 2026-13-01, and 2027-01-05"
for m in DATE_RE.finditer(text):
    print(m.group(0), "->", m.groupdict())

# Output:
# 2026-11-26 -> {'year': '2026', 'month': '11', 'day': '26'}
# 2027-01-05 -> {'year': '2027', 'month': '01', 'day': '05'}
```

*(Note: regex checks syntax, not calendar rules like leap years or February 30; combine with `datetime.date.fromisoformat()` for full calendar validation.)*

---

## 15. Best Practices and Performance Tips

### 1. Don't Use Regex When Simple String Methods Suffice

Built-in string methods (`in`, `.startswith()`, `.endswith()`, `.split()`, `.replace()`) are implemented in C without regex overhead and are easier to read.

```python
# Prefer this:
if filename.endswith(".py"):
    ...

# Over this:
if re.search(r"\.py$", filename):
    ...
```

### 2. Pre-Compile Patterns Reused in Loops or Hot Paths

Although `re` maintains an internal LRU cache for recent patterns, compiling once with `re.compile()` avoids repeated cache lookups and improves code clarity.

```python
WORD_RE = re.compile(r"\b[a-z]+\b", re.IGNORECASE)

words_per_line = [WORD_RE.findall(line) for line in lines]
```

### 3. Beware of Catastrophic Backtracking (ReDoS)

Nested quantifiers like `(a+)+`, `([a-zA-Z]+)*`, or overlapping alternatives inside a loop can cause exponential runtime on non-matching input.

```python
# DANGEROUS: nested quantifiers -> exponential backtracking on "aaaaaaaaaaaaaaaaaaaa!"
bad = re.compile(r"^(a+)+$")

# SAFE: collapse into a single quantifier
good = re.compile(r"^a+$")
```

### 4. Use Specific Patterns Instead of `.*`

- Prefer negated character classes (e.g., `[^"]*` instead of `.*?`) when matching delimited content.
- Anchor patterns with `^` / `$` or `\A` / `\Z` whenever you only care about start/end/full matches so the engine fails fast instead of scanning the entire string.

### 5. Use `(?:...)` When You Don't Need Captured Text

Non-capturing groups avoid allocating group entries and keep `.groups()` and `re.findall()` outputs clean.

### 6. Always Escape Dynamic User Input with `re.escape()`

If you build a regex dynamically from a variable (like a search term entered by a user), escape it so characters like `.`, `*`, `?`, or `[` don't break the pattern or alter its meaning.

```python
import re

user_query = "C++ (2026)"
safe_pattern = re.compile(re.escape(user_query))
print(bool(safe_pattern.search("Learning C++ (2026) basics")))  # True
```

### 7. Make Complex Patterns Readable with `re.VERBOSE`

Never ship an unreadable one-line regex if it takes more than a few seconds to parse mentally. Use `re.VERBOSE` with indentation and inline comments.

### 8. Know When *Not* to Use Regex

- **HTML/XML parsing**: Use `html.parser`, `xml.etree.ElementTree`, or `BeautifulSoup`.
- **JSON/CSV parsing**: Use Python's `json` and `csv` modules.
- **URL/IP/Date semantics**: Pair regex extraction with `urllib.parse`, `ipaddress`, or `datetime` for strict validation.
