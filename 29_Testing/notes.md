# Testing in Python

Software testing is the process of verifying that your code behaves as expected under both normal and edge-case conditions. Python has rich built-in testing tools (`unittest`, `doctest`, `unittest.mock`) as well as a thriving third-party ecosystem led by `pytest` and `coverage.py`.

---

## Table of Contents

1. [Why Testing Matters](#1-why-testing-matters)
2. [Types of Testing](#2-types-of-testing)
3. [The `unittest` Module](#3-the-unittest-module)
4. [Writing Test Cases with `unittest`](#4-writing-test-cases-with-unittest)
5. [Test Discovery and Running Tests](#5-test-discovery-and-running-tests)
6. [The `pytest` Framework](#6-the-pytest-framework)
7. [`pytest` Assertions and Markers](#7-pytest-assertions-and-markers)
8. [`pytest` Parametrize](#8-pytest-parametrize)
9. [Mocking (`unittest.mock`, `MagicMock`, `patch`)](#9-mocking-unittestmock-magicmock-patch)
10. [Test Coverage (`coverage.py`)](#10-test-coverage-coveragepy)
11. [The `doctest` Module](#11-the-doctest-module)
12. [Test-Driven Development (TDD) Basics](#12-test-driven-development-tdd-basics)
13. [Continuous Integration (CI) Basics](#13-continuous-integration-ci-basics)
14. [Best Practices for Writing Tests](#14-best-practices-for-writing-tests)

---

## 1. Why Testing Matters

Manual testing (running your script and visually checking output) does not scale as a codebase grows. Automated tests provide a repeatable safety net.

### Key Benefits

- **Early Bug Detection:** Catch regressions and logic errors before code reaches production.
- **Confidence to Refactor:** Safely restructure or optimize code knowing tests will alert you if existing behavior breaks.
- **Living Documentation:** Well-written tests demonstrate how functions, classes, and APIs are intended to be used, including edge cases.
- **Better Software Design:** Code that is easy to test tends to be modular, loosely coupled, and have clear responsibilities.
- **Faster Team Collaboration:** Pull requests with automated tests give reviewers confidence that changes work and don't break other features.

### Manual vs. Automated Testing

```python
# src/math_utils.py
def discount_price(price: float, percent: float) -> float:
    if price < 0:
        raise ValueError("Price cannot be negative")
    if not 0 <= percent <= 100:
        raise ValueError("Discount percent must be between 0 and 100")
    return round(price * (1 - percent / 100), 2)
```

Instead of manually running `print(discount_price(100, 20))` every time you edit `math_utils.py`, automated tests verify normal values, boundary values (`0%`, `100%`), and invalid inputs in milliseconds.

---

## 2. Types of Testing

Testing is often visualized as the **Testing Pyramid**: many fast, isolated unit tests at the base, fewer integration/functional tests in the middle, and a small number of end-to-end tests at the top.

| Type | Scope | Speed | Dependencies | Goal |
| :--- | :--- | :--- | :--- | :--- |
| **Unit Testing** | Single function, method, or class | Very fast | Mocked/stubbed | Verify isolated logic works correctly |
| **Integration Testing** | Multiple components working together | Medium | Real or test DB/filesystem/APIs | Verify modules interact properly |
| **Functional Testing** | Business requirements / user-facing features | Medium | Full app or subsystem (often with test client) | Verify *what* the system does against specs |
| **End-to-End (E2E) Testing** | Entire application stack | Slow | Real browser/UI, server, database | Simulate real user workflows from start to finish |

### Practical Examples

1. **Unit Test:** Testing `calculate_tax(100, 0.08)` returns `8.0` without touching a database or network.
2. **Integration Test:** Testing that a `UserRepository.save(user)` method inserts a row into a SQLite/PostgreSQL test database and `UserRepository.get(id)` reads it back.
3. **Functional Test:** Sending a `POST /orders` request via a framework test client (e.g., FastAPI `TestClient` or Django `Client`) and verifying it returns HTTP `201` with the expected order payload.
4. **End-to-End Test:** Using Playwright or Selenium to open a browser, log in, add an item to a cart, submit checkout, and confirm the confirmation page renders.

---

## 3. The `unittest` Module

`unittest` is Python's built-in testing framework (inspired by Java's JUnit). It requires no external installation and organizes tests as methods inside subclasses of `unittest.TestCase`.

### Core Concepts

- **`unittest.TestCase`**: Base class for test suites. Test method names **must** start with `test_`.
- **`setUp(self)`**: Runs **before** each individual test method to prepare fresh test state.
- **`tearDown(self)`**: Runs **after** each individual test method to clean up resources (even if the test failed).
- **`setUpClass(cls)` / `tearDownClass(cls)`**: Class methods (`@classmethod`) that run **once** before/after all tests in the class (useful for expensive setup like starting a server or database connection).

### Lifecycle Order

For a class with two test methods (`test_one` and `test_two`):

1. `setUpClass()`
2. `setUp()` -> `test_one()` -> `tearDown()`
3. `setUp()` -> `test_two()` -> `tearDown()`
4. `tearDownClass()`

### Common `unittest` Assertions

| Method | Checks that |
| :--- | :--- |
| `assertEqual(a, b)` | `a == b` |
| `assertNotEqual(a, b)` | `a != b` |
| `assertTrue(x)` / `assertFalse(x)` | `bool(x)` is `True` / `False` |
| `assertIs(a, b)` / `assertIsNot(a, b)` | `a is b` / `a is not b` |
| `assertIsNone(x)` / `assertIsNotNone(x)` | `x is None` / `x is not None` |
| `assertIn(a, b)` / `assertNotIn(a, b)` | `a in b` / `a not in b` |
| `assertIsInstance(a, cls)` | `isinstance(a, cls)` |
| `assertAlmostEqual(a, b, places=7)` | `round(a - b, places) == 0` (for floats) |
| `assertRaises(Exc)` | Calling code raises `Exc` |
| `assertRaisesRegex(Exc, pattern)` | Exception message matches regex `pattern` |
| `assertCountEqual(a, b)` | `a` and `b` have the same elements regardless of order |

---

## 4. Writing Test Cases with `unittest`

Suppose we have a `BankAccount` class in `bank.py`:

```python
# bank.py
class InsufficientFundsError(Exception):
    pass


class BankAccount:
    def __init__(self, owner: str, balance: float = 0.0) -> None:
        if balance < 0:
            raise ValueError("Initial balance cannot be negative")
        self.owner = owner
        self.balance = balance

    def deposit(self, amount: float) -> float:
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self.balance += amount
        return self.balance

    def withdraw(self, amount: float) -> float:
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self.balance:
            raise InsufficientFundsError("Not enough funds")
        self.balance -= amount
        return self.balance
```

Here is a complete `unittest` suite in `test_bank.py`:

```python
# test_bank.py
import unittest
from bank import BankAccount, InsufficientFundsError


class TestBankAccount(unittest.TestCase):
    @classmethod
    def setUpClass(cls) -> None:
        print("\n[setUpClass] Starting BankAccount test suite")

    @classmethod
    def tearDownClass(cls) -> None:
        print("[tearDownClass] Finished BankAccount test suite")

    def setUp(self) -> None:
        # Runs before every test method so tests stay isolated
        self.account = BankAccount(owner="Alice", balance=100.0)

    def tearDown(self) -> None:
        # Clean up resources if needed (files, connections, temp state)
        del self.account

    def test_initial_balance(self) -> None:
        self.assertEqual(self.account.owner, "Alice")
        self.assertAlmostEqual(self.account.balance, 100.0)

    def test_negative_initial_balance_raises(self) -> None:
        with self.assertRaisesRegex(ValueError, "cannot be negative"):
            BankAccount(owner="Bob", balance=-50.0)

    def test_deposit_increases_balance(self) -> None:
        new_balance = self.account.deposit(50.25)
        self.assertAlmostEqual(new_balance, 150.25)
        self.assertAlmostEqual(self.account.balance, 150.25)

    def test_withdraw_decreases_balance(self) -> None:
        new_balance = self.account.withdraw(40.0)
        self.assertAlmostEqual(new_balance, 60.0)

    def test_withdraw_too_much_raises_insufficient_funds(self) -> None:
        with self.assertRaises(InsufficientFundsError):
            self.account.withdraw(200.0)

    def test_multiple_deposit_amounts_with_subtest(self) -> None:
        # subTest reports each failing case individually instead of stopping at the first
        invalid_amounts = [0, -1, -100.5]
        for amount in invalid_amounts:
            with self.subTest(amount=amount):
                with self.assertRaises(ValueError):
                    self.account.deposit(amount)

    @unittest.skip("Demonstrating how to skip a test unconditionally")
    def test_future_feature(self) -> None:
        self.fail("Not implemented yet")


if __name__ == "__main__":
    unittest.main()
```

### Useful `unittest` Decorators

- `@unittest.skip("reason")`: Always skip this test.
- `@unittest.skipIf(condition, "reason")`: Skip when `condition` is `True` (e.g., OS-specific tests).
- `@unittest.skipUnless(condition, "reason")`: Skip unless `condition` is `True`.
- `@unittest.expectedFailure`: Mark a test that is currently expected to fail (fails the suite if it unexpectedly passes).

---

## 5. Test Discovery and Running Tests

### Typical Project Structure

```text
my_project/
├── src/
│   ├── __init__.py
│   └── calculator.py
└── tests/
    ├── __init__.py        # Needed for unittest discovery in older/package layouts
    ├── test_calculator.py
    └── test_bank.py
```

### Running Tests with `unittest`

```bash
# Run a single test file directly
python test_bank.py

# Run using the unittest module with verbose output (-v)
python -m unittest -v test_bank.py

# Run a specific class or single test method
python -m unittest test_bank.TestBankAccount
python -m unittest test_bank.TestBankAccount.test_deposit_increases_balance

# Auto-discover all test*.py files in the current directory and subdirectories
python -m unittest discover

# Discover tests in a specific directory with a custom pattern and start directory
python -m unittest discover -s tests -p "test_*.py" -t . -v
```

**Common `unittest discover` flags:**
- `-s` / `--start-directory`: Directory to start discovery (default `.`).
- `-p` / `--pattern`: Filename pattern to match (default `test*.py`).
- `-t` / `--top-level-directory`: Top-level directory of the project (helps resolve imports like `from src.calculator import add`).
- `-v` / `--verbose`: Show individual test names and statuses.
- `-f` / `--failfast`: Stop on the first failure or error.

---

## 6. The `pytest` Framework

`pytest` is the most widely used third-party testing framework in Python. Compared to `unittest`, it removes boilerplate (no need to inherit from `TestCase`), uses Python's plain `assert` statement with rich failure diffs, and provides a powerful fixture and plugin system.

### Installation

```bash
pip install pytest
# Verify installation
pytest --version
```

### Writing Simple `pytest` Tests

`pytest` discovers files named `test_*.py` or `*_test.py`, functions named `test_*`, and classes named `Test*` (with no `__init__` method).

```python
# test_strings.py
def normalize_username(name: str) -> str:
    return name.strip().lower().replace(" ", "_")


def test_normalize_username_basic() -> None:
    assert normalize_username("  Alice Smith ") == "alice_smith"


def test_normalize_username_empty() -> None:
    assert normalize_username("   ") == ""
```

Run it with:

```bash
pytest -v
```

### `pytest` Fixtures

Fixtures are reusable functions decorated with `@pytest.fixture` that provide test data, objects, or setup/teardown logic. A test requests a fixture simply by accepting its name as a parameter (dependency injection).

```python
# test_fixtures.py
import pytest
from pathlib import Path


@pytest.fixture
def sample_user() -> dict:
    return {"id": 1, "name": "Ada Lovelace", "role": "admin"}


@pytest.fixture
def temp_log_file(tmp_path: Path):
    # Setup: create a temporary file using pytest's built-in tmp_path fixture
    log_file = tmp_path / "app.log"
    log_file.write_text("START\n", encoding="utf-8")

    yield log_file  # Test runs here and receives log_file

    # Teardown: runs after the test finishes (even if it fails)
    if log_file.exists():
        log_file.unlink()


def test_user_is_admin(sample_user: dict) -> None:
    assert sample_user["role"] == "admin"


def test_append_to_log(temp_log_file: Path) -> None:
    with temp_log_file.open("a", encoding="utf-8") as f:
        f.write("EVENT\n")
    content = temp_log_file.read_text(encoding="utf-8")
    assert content == "START\nEVENT\n"
```

### Fixture Scopes & `conftest.py`

- **Scopes** control how often a fixture runs:
  - `@pytest.fixture(scope="function")` *(default)*: Once per test function.
  - `@pytest.fixture(scope="class")`: Once per test class.
  - `@pytest.fixture(scope="module")`: Once per `.py` test file.
  - `@pytest.fixture(scope="session")`: Once for the entire `pytest` run.
- **`conftest.py`**: Place shared fixtures in `tests/conftest.py`. Any test file in that directory or its subdirectories can use those fixtures automatically without importing them.
- **Built-in `pytest` fixtures**: `tmp_path` (temporary `Path` directory), `monkeypatch` (temporarily modify env vars, attributes, or dict entries), `capsys` (capture `stdout`/`stderr`), `caplog` (capture `logging` output).

---

## 7. `pytest` Assertions and Markers

### Rich `pytest` Assertions

You use standard Python `assert` statements. When an assertion fails, `pytest` rewrites the AST to display the exact values of subexpressions.

```python
import pytest


def divide(a: float, b: float) -> float:
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return a / b


def test_divide_success() -> None:
    assert divide(10, 2) == 5.0
    assert divide(1, 3) == pytest.approx(0.333333, rel=1e-5)


def test_divide_by_zero() -> None:
    with pytest.raises(ZeroDivisionError, match="Cannot divide by zero") as exc_info:
        divide(10, 0)
    assert "zero" in str(exc_info.value)
```

### `pytest` Markers

Markers attach metadata to tests so you can control execution or filter test runs.

**Built-in markers:**
- `@pytest.mark.skip(reason="...")`: Unconditionally skip.
- `@pytest.mark.skipif(condition, reason="...")`: Skip if condition is true.
- `@pytest.mark.xfail(reason="...")`: Expected to fail (reports `XFAIL` if it fails, `XPASS` if it passes).

**Custom markers:**

Register custom markers in `pyproject.toml` (or `pytest.ini`) to avoid warnings:

```toml
# pyproject.toml
[tool.pytest.ini_options]
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "integration: marks tests requiring external services",
]
```

Use them in tests:

```python
import sys
import pytest


@pytest.mark.slow
def test_large_dataset_processing() -> None:
    assert sum(range(1_000_000)) == 499_999_500_000


@pytest.mark.skipif(sys.platform == "win32", reason="POSIX-only behavior")
def test_unix_permissions() -> None:
    assert True


@pytest.mark.xfail(reason="Known bug #42: rounding edge case")
def test_known_issue() -> None:
    assert round(2.5) == 3  # Python uses round-half-to-even (2)
```

Run tests selectively by marker:

```bash
pytest -m "slow"           # Run only slow tests
pytest -m "not slow"       # Skip slow tests
pytest -m "integration"    # Run integration tests
```

---

## 8. `pytest` Parametrize

`@pytest.mark.parametrize` runs the same test function multiple times with different input/output combinations, eliminating copy-pasted test functions.

### Basic Usage

```python
import pytest


def is_palindrome(text: str) -> bool:
    cleaned = "".join(ch.lower() for ch in text if ch.isalnum())
    return cleaned == cleaned[::-1]


@pytest.mark.parametrize(
    "text, expected",
    [
        ("racecar", True),
        ("A man, a plan, a canal: Panama", True),
        ("python", False),
        ("", True),
        ("12321", True),
    ],
)
def test_is_palindrome(text: str, expected: bool) -> None:
    assert is_palindrome(text) is expected
```

### Custom Test IDs and `pytest.param`

Use `ids` or `pytest.param(..., id=..., marks=...)` to make `-v` output readable and attach markers to individual test cases:

```python
import pytest


def parse_port(value: str) -> int:
    port = int(value)
    if not 1 <= port <= 65535:
        raise ValueError("Port out of range")
    return port


@pytest.mark.parametrize(
    "raw, expected",
    [
        pytest.param("80", 80, id="http"),
        pytest.param("443", 443, id="https"),
        pytest.param("65535", 65535, id="max-port"),
        pytest.param("0", None, marks=pytest.mark.xfail(raises=ValueError), id="zero-invalid"),
    ],
)
def test_parse_port(raw: str, expected: int) -> None:
    assert parse_port(raw) == expected
```

### Stacking `@pytest.mark.parametrize`

Stacking two decorators produces the **Cartesian product** of all parameter sets (great for testing combinations):

```python
@pytest.mark.parametrize("x", [1, 2])
@pytest.mark.parametrize("y", [10, 20])
def test_multiplication_commutative(x: int, y: int) -> None:
    # Runs 4 times: (1, 10), (2, 10), (1, 20), (2, 20)
    assert x * y == y * x
```

---

## 9. Mocking (`unittest.mock`, `MagicMock`, `patch`)

Mocking replaces real dependencies (HTTP APIs, databases, clocks, emails, payment gateways) with controllable fake objects during tests so unit tests remain fast, deterministic, and isolated.

### `Mock` vs. `MagicMock`

- **`Mock`**: General-purpose mock object. Tracks calls (`assert_called_once_with`, `call_count`, etc.) and returns configured `return_value` or triggers `side_effect`.
- **`MagicMock`**: Subclass of `Mock` that pre-implements Python magic/dunder methods (`__len__`, `__iter__`, `__getitem__`, `__enter__`/`__exit__`, `__str__`, etc.). Prefer `MagicMock` when mocking containers, context managers, or objects using special methods.

```python
from unittest.mock import Mock, MagicMock

# Basic Mock
api_client = Mock()
api_client.fetch_user.return_value = {"id": 7, "name": "Grace"}

user = api_client.fetch_user(7)
assert user["name"] == "Grace"
api_client.fetch_user.assert_called_once_with(7)

# side_effect for dynamic returns, sequences, or raising exceptions
flaky_service = Mock()
flaky_service.call.side_effect = [ConnectionError("Timeout"), "OK"]

# MagicMock supports dunder methods out of the box
mock_list = MagicMock()
mock_list.__len__.return_value = 3
assert len(mock_list) == 3
```

### Using `patch`

`patch` temporarily replaces an object in a target module during a `with` block or decorated test function.

> **Golden Rule of `patch`:** Patch where the object is **looked up / used**, NOT where it is originally defined.
> If `weather.py` does `import requests` and calls `requests.get(...)`, patch `"weather.requests.get"`.
> If `weather.py` does `from requests import get` and calls `get(...)`, patch `"weather.get"`.

```python
# weather.py
import requests


def get_temperature(city: str) -> float:
    response = requests.get(f"https://api.example.com/weather?city={city}", timeout=5)
    response.raise_for_status()
    return response.json()["temp_c"]
```

```python
# test_weather.py
from unittest.mock import patch, Mock
import pytest
import weather


# 1. Using patch as a decorator (mocks are injected as arguments from bottom-up)
@patch("weather.requests.get")
def test_get_temperature_success(mock_get: Mock) -> None:
    mock_response = Mock()
    mock_response.json.return_value = {"temp_c": 22.5}
    mock_get.return_value = mock_response

    temp = weather.get_temperature("London")

    assert temp == 22.5
    mock_get.assert_called_once_with(
        "https://api.example.com/weather?city=London",
        timeout=5,
    )
    mock_response.raise_for_status.assert_called_once()


# 2. Using patch as a context manager
def test_get_temperature_http_error() -> None:
    with patch("weather.requests.get") as mock_get:
        mock_get.return_value.raise_for_status.side_effect = RuntimeError("503 Error")
        with pytest.raises(RuntimeError, match="503 Error"):
            weather.get_temperature("Paris")
```

### Safety Tip: `autospec=True`

Pass `autospec=True` (or `create_autospec`) so your mock enforces the real function/class signature and catches typos in method names or invalid arguments:

```python
@patch("weather.requests.get", autospec=True)
def test_with_autospec(mock_get):
    # mock_get.non_existent_method() -> raises AttributeError!
    ...
```

---

## 10. Test Coverage (`coverage.py`)

Test coverage measures which lines or branches of your production code were executed when running your test suite. High coverage does not guarantee bug-free code, but low coverage guarantees untested code paths.

### Installation

```bash
pip install coverage
# Optional pytest integration plugin:
pip install pytest-cov
```

### Using `coverage.py` Directly

```bash
# Run tests under coverage tracking (measuring source in 'src/')
coverage run --source=src -m pytest

# Or with unittest:
coverage run --source=src -m unittest discover

# Print a terminal report showing missing line numbers
coverage report -m

# Generate an interactive HTML report in htmlcov/index.html
coverage html
```

### Using `pytest-cov`

```bash
# Run pytest and show missing lines in one command
pytest --cov=src --cov-report=term-missing

# Include branch coverage (checks both True and False paths of if/else)
pytest --cov=src --cov-branch --cov-report=term-missing

# Fail the build if coverage drops below 85%
pytest --cov=src --cov-fail-under=85
```

### Example Output

```text
Name                 Stmts   Miss  Cover   Missing
--------------------------------------------------
src/__init__.py          0      0   100%
src/bank.py             21      2    90%   14, 22
--------------------------------------------------
TOTAL                   21      2    90%
```

### Configuration in `pyproject.toml`

```toml
[tool.coverage.run]
source = ["src"]
branch = true
omit = ["*/tests/*", "*/__init__.py"]

[tool.coverage.report]
show_missing = true
fail_under = 80
```

---

## 11. The `doctest` Module

`doctest` is a built-in Python module that searches docstrings for pieces of text that look like interactive Python sessions (`>>>`), executes them, and verifies they produce the exact documented output.

### Example

```python
# factorial.py
def factorial(n: int) -> int:
    """Return the factorial of a non-negative integer n.

    >>> factorial(0)
    1
    >>> factorial(5)
    120
    >>> factorial(-1)
    Traceback (most recent call last):
        ...
    ValueError: n must be >= 0
    """
    if n < 0:
        raise ValueError("n must be >= 0")
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result


if __name__ == "__main__":
    import doctest
    doctest.testmod()
```

### Running `doctest`

```bash
# Run doctests silently (only prints output on failure)
python factorial.py

# Verbose mode
python -m doctest -v factorial.py

# Run doctests inside pytest alongside regular tests
pytest --doctest-modules
```

### When to Use `doctest`

- **Good for:** Short usage examples in docstrings, ensuring documentation never goes stale.
- **Not ideal for:** Complex setup/teardown, mocking, or exhaustive edge-case suites (use `pytest` or `unittest` for those).

---

## 12. Test-Driven Development (TDD) Basics

Test-Driven Development (TDD) is a workflow where you write a failing test **before** writing the production code that makes it pass.

### The Red-Green-Refactor Cycle

1. **Red:** Write a small, focused test for a behavior that doesn't exist yet. Run the test suite and watch it **fail** (proving the test actually checks something).
2. **Green:** Write the **minimal** amount of production code needed to make the test pass. Don't over-engineer yet.
3. **Refactor:** Clean up both production code and test code (remove duplication, improve names, simplify logic) while keeping all tests green.

### Mini Walkthrough: Building `fizzbuzz(n)`

**Step 1 (Red):** Write the first test in `test_fizzbuzz.py`:

```python
# test_fizzbuzz.py
from fizzbuzz import fizzbuzz


def test_regular_number() -> None:
    assert fizzbuzz(1) == "1"
    assert fizzbuzz(2) == "2"
```

Running `pytest` fails with `ImportError` / `NameError`.

**Step 2 (Green):** Implement minimal code in `fizzbuzz.py`:

```python
# fizzbuzz.py
def fizzbuzz(n: int) -> str:
    return str(n)
```

Running `pytest` passes!

**Step 3 (Red -> Green -> Refactor):** Add tests for multiples of `3`, `5`, and `15` one at a time, updating `fizzbuzz.py` until you arrive at a clean implementation:

```python
# fizzbuzz.py
def fizzbuzz(n: int) -> str:
    if n % 15 == 0:
        return "FizzBuzz"
    if n % 3 == 0:
        return "Fizz"
    if n % 5 == 0:
        return "Buzz"
    return str(n)
```

```python
# test_fizzbuzz.py
import pytest
from fizzbuzz import fizzbuzz


@pytest.mark.parametrize(
    "n, expected",
    [
        (1, "1"),
        (3, "Fizz"),
        (5, "Buzz"),
        (15, "FizzBuzz"),
        (30, "FizzBuzz"),
    ],
)
def test_fizzbuzz(n: int, expected: str) -> None:
    assert fizzbuzz(n) == expected
```

---

## 13. Continuous Integration (CI) Basics

Continuous Integration (CI) automatically runs your test suite, linters, and type checkers on every commit or pull request in a clean, reproducible environment.

### What a Python CI Pipeline Typically Runs

1. Checkout repository code.
2. Set up the target Python version(s) (often a matrix like 3.10, 3.11, 3.12).
3. Install dependencies (`pip install -r requirements.txt` or `pyproject.toml`).
4. Run linting/formatting/type checks (`ruff`, `flake8`, `mypy`).
5. Run the test suite with coverage (`pytest --cov=src`).

### Example GitHub Actions Workflow

Save as `.github/workflows/tests.yml`:

```yaml
name: Python Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.10", "3.11", "3.12"]

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: "pip"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov

      - name: Run test suite with coverage
        run: pytest --cov=src --cov-report=term-missing --cov-fail-under=80
```

---

## 14. Best Practices for Writing Tests

- **Follow the AAA Pattern (Arrange-Act-Assert):** Structure each test clearly into setting up inputs, executing the behavior under test, and asserting the outcome.

  ```python
  def test_apply_coupon() -> None:
      # Arrange
      cart = Cart(subtotal=100.0)
      coupon = Coupon(code="SAVE20", percent=20)

      # Act
      total = cart.apply_coupon(coupon)

      # Assert
      assert total == 80.0
  ```

- **Keep Tests Independent & Isolated (F.I.R.S.T. Principles):**
  - **F**ast: Tests should run in milliseconds/seconds so developers run them frequently.
  - **I**ndependent/Isolated: Never rely on test execution order or shared mutable global state.
  - **R**epeatable: Tests must produce the same result on any machine, at any time (mock clocks, randomness, and network calls).
  - **S**elf-validating: Tests return pass/fail automatically without manual log inspection.
  - **T**imely: Write tests alongside (or before) the code they verify.
- **Test One Behavior per Test Function:** Smaller focused tests make failures immediately obvious.
- **Use Descriptive Test Names:** Prefer `test_withdraw_raises_when_balance_is_insufficient` over `test_withdraw_2`.
- **Test Edge Cases and Error Paths:** Test empty inputs, `None`, boundaries (`0`, `-1`, max limits), and expected exceptions—not just the "happy path".
- **Avoid Over-Mocking:** Mock external boundaries (network, filesystem, time, third-party APIs), not your own internal business logic—otherwise you end up testing your mocks instead of your code.
- **Keep Test Logic Simple:** Avoid complex loops or conditionals inside test bodies; use `@pytest.mark.parametrize` or `subTest` instead.
- **Don't Ignore Flaky Tests:** A test that passes sometimes and fails other times erodes trust in the entire suite—fix non-determinism (race conditions, time dependencies, external calls) immediately.
