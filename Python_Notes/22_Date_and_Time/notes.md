# Date and Time in Python

Working with dates, times, timestamps, durations, and timezones is a core part of Python programming. Python's standard library provides three primary modules for temporal operations—`datetime`, `time`, and `calendar`—along with `zoneinfo` (added in Python 3.9) for IANA timezone support.

---

## Table of Contents

1. [`datetime` Module Overview](#1-datetime-module-overview)
2. [`date` Class (`date.today()`, Date Components)](#2-date-class-datetoday-date-components)
3. [`time` Class](#3-time-class)
4. [`datetime` Class (`datetime.now()`, `datetime.utcnow()`)](#4-datetime-class-datetimenow-datetimeutcnow)
5. [`timedelta` Class (Date Arithmetic)](#5-timedelta-class-date-arithmetic)
6. [Formatting Dates (`strftime`)](#6-formatting-dates-strftime)
7. [Parsing Date Strings (`strptime`)](#7-parsing-date-strings-strptime)
8. [Format Codes (`%Y`, `%m`, `%d`, `%H`, `%M`, `%S`, etc.)](#8-format-codes-y-m-d-h-m-s-etc)
9. [`time` Module (`time.time()`, `time.sleep()`, `time.perf_counter()`)](#9-time-module-timetime-timesleep-timeperf_counter)
10. [`calendar` Module](#10-calendar-module)
11. [Timezone Handling (`pytz`, `zoneinfo`)](#11-timezone-handling-pytz-zoneinfo)
12. [Comparing Dates](#12-comparing-dates)
13. [Date Ranges and Iterations](#13-date-ranges-and-iterations)
14. [Common Date/Time Patterns and Recipes](#14-common-datetime-patterns-and-recipes)

---

## 1. `datetime` Module Overview

The built-in `datetime` module provides object-oriented classes for manipulating dates and times in both simple and complex ways. All `datetime` objects are **immutable** and fall into two categories:

- **Naive**: Does not contain timezone or daylight saving time (DST) information (`tzinfo is None`). Easy to use for local/abstract calculations, but ambiguous across regions.
- **Aware**: Contains timezone information (`tzinfo is not None` and `tzinfo.utcoffset(d) is not None`), unambiguously representing a specific moment in time.

### Core Classes in `datetime`

| Class | Description |
| :--- | :--- |
| `datetime.date` | Idealized calendar date (`year`, `month`, `day`), assuming the Gregorian calendar. |
| `datetime.time` | Idealized time of day (`hour`, `minute`, `second`, `microsecond`, `tzinfo`). |
| `datetime.datetime` | Combination of `date` and `time` attributes. |
| `datetime.timedelta` | Duration expressing the difference between two `date`, `time`, or `datetime` instances. |
| `datetime.tzinfo` | Abstract base class for timezone objects. |
| `datetime.timezone` | Concrete `tzinfo` subclass representing a fixed offset from UTC (e.g., `timezone.utc`). |

```python
import datetime

print(datetime.MINYEAR)  # 1
print(datetime.MAXYEAR)  # 9999
```

> **Note:** `datetime` (the module) contains a class also named `datetime` (`datetime.datetime`). Importing `from datetime import datetime, date, time, timedelta` avoids repetitive `datetime.datetime` references.

---

## 2. `date` Class (`date.today()`, Date Components)

The `datetime.date` class represents a calendar date without time-of-day or timezone details.

### Creating `date` Objects

```python
from datetime import date

# Construct explicitly: date(year, month, day)
independence_day = date(1947, 8, 15)
print(independence_day)  # 1947-08-15

# Current local date
today = date.today()
print(today)  # e.g., 2026-11-26

# From a Unix timestamp (seconds since 1970-01-01 UTC)
from_ts = date.fromtimestamp(1_700_000_000)
print(from_ts)  # 2023-11-14

# From an ISO 8601 formatted string (YYYY-MM-DD)
from_iso = date.fromisoformat("2026-11-26")
print(from_iso)  # 2026-11-26

# From ISO calendar (year, week, weekday: 1=Mon .. 7=Sun)
from_isocal = date.fromisocalendar(2026, 48, 4)
print(from_isocal)  # 2026-11-26
```

### Date Components and Methods

```python
from datetime import date

d = date(2026, 11, 26)

# Attributes
print(d.year)   # 2026
print(d.month)  # 11
print(d.day)    # 26

# Day of the week
print(d.weekday())     # 3 (Monday=0, Sunday=6)
print(d.isoweekday())  # 4 (Monday=1, Sunday=7)

# ISO calendar tuple: (ISO year, ISO week number, ISO weekday)
print(d.isocalendar())  # datetime.IsoCalendarDate(year=2026, week=48, weekday=4)

# ISO 8601 string representation
print(d.isoformat())  # '2026-11-26'

# Creating a modified copy (since date is immutable)
next_year = d.replace(year=2027)
print(next_year)  # 2027-11-26

# Proleptic Gregorian ordinal (Jan 1, Year 1 == 1)
ordinal = d.toordinal()
print(ordinal)                  # 739946
print(date.fromordinal(ordinal))  # 2026-11-26
```

---

## 3. `time` Class

The `datetime.time` class represents a time of day independent of any particular date. All constructor arguments are optional and default to `0` (or `None` for `tzinfo`).

```python
from datetime import time, timezone, timedelta

# Default time (midnight)
midnight = time()
print(midnight)  # 00:00:00

# time(hour=0, minute=0, second=0, microsecond=0, tzinfo=None)
t = time(14, 30, 45, 500_000)
print(t)              # 14:30:45.500000
print(t.hour)         # 14
print(t.minute)       # 30
print(t.second)       # 45
print(t.microsecond)  # 500000
print(t.tzinfo)       # None

# From/to ISO 8601 string
parsed_time = time.fromisoformat("09:15:30")
print(parsed_time)               # 09:15:30
print(parsed_time.isoformat())   # '09:15:30'

# Replacing specific fields
adjusted = t.replace(hour=16, microsecond=0)
print(adjusted)  # 16:30:45

# Aware time with a fixed UTC offset (e.g., UTC+05:30)
ist = timezone(timedelta(hours=5, minutes=30))
aware_t = time(10, 0, tzinfo=ist)
print(aware_t.isoformat())  # '10:00:00+05:30'
```

> **Note:** You cannot directly add or subtract a `timedelta` with a `datetime.time` object. To perform time arithmetic, combine the `time` with a `date` into a `datetime` object first, apply the `timedelta`, and extract `.time()`.

---

## 4. `datetime` Class (`datetime.now()`, `datetime.utcnow()`)

The `datetime.datetime` class combines all fields of `date` and `time` into a single object.

### Constructing `datetime` Objects

```python
from datetime import datetime, date, time, timezone

# Explicit constructor: datetime(year, month, day, hour=0, minute=0, second=0, microsecond=0, tzinfo=None)
dt = datetime(2026, 11, 26, 14, 30, 0)
print(dt)  # 2026-11-26 14:30:00

# Combining a date and time object
combined = datetime.combine(date(2026, 11, 26), time(9, 0))
print(combined)  # 2026-11-26 09:00:00

# Splitting back into date and time
print(dt.date())  # 2026-11-26
print(dt.time())  # 14:30:00

# From ISO 8601 string
iso_dt = datetime.fromisoformat("2026-11-26T14:30:00+00:00")
print(iso_dt)  # 2026-11-26 14:30:00+00:00
```

### `datetime.now()` vs `datetime.utcnow()` vs `datetime.now(timezone.utc)`

```python
from datetime import datetime, timezone

# 1. Local naive datetime
local_now = datetime.now()
print("Local (naive):", local_now)

# 2. Local naive datetime (today() takes no tz argument)
today_dt = datetime.today()
print("Today (naive):", today_dt)

# 3. Current UTC time as an AWARE datetime (Recommended)
utc_aware = datetime.now(timezone.utc)
print("UTC (aware):  ", utc_aware)

# 4. Legacy naive UTC datetime (DEPRECATED in Python 3.12+)
# utcnow() returns a naive datetime with tzinfo=None, which can cause bugs
# when mixed with timezone-aware datetimes.
utc_naive = datetime.utcnow()
print("UTC (naive):  ", utc_naive)
```

> **Important (Python 3.12+):** `datetime.utcnow()` and `datetime.utcfromtimestamp()` are **deprecated** in Python 3.12 because they return naive `datetime` objects for UTC timestamps. Always prefer:
> - `datetime.now(timezone.utc)` instead of `datetime.utcnow()`
> - `datetime.fromtimestamp(ts, tz=timezone.utc)` instead of `datetime.utcfromtimestamp(ts)`

### Unix Timestamps with `datetime`

```python
from datetime import datetime, timezone

now_utc = datetime.now(timezone.utc)

# Convert datetime to Unix timestamp (float seconds since epoch)
ts = now_utc.timestamp()
print(ts)

# Convert Unix timestamp back to aware UTC datetime
restored = datetime.fromtimestamp(ts, tz=timezone.utc)
print(restored)
```

---

## 5. `timedelta` Class (Date Arithmetic)

A `timedelta` object represents a duration—the difference between two `date` or `datetime` instances.

Internally, `timedelta` stores only `days`, `seconds` (`0` to `86399`), and `microseconds` (`0` to `999999`), normalizing all other constructor units (`weeks`, `hours`, `minutes`, `milliseconds`).

```python
from datetime import datetime, date, timedelta

# Constructor: timedelta(days=0, seconds=0, microseconds=0, milliseconds=0, minutes=0, hours=0, weeks=0)
delta = timedelta(weeks=1, days=2, hours=3, minutes=30)
print(delta)               # 9 days, 3:30:00
print(delta.days)          # 9
print(delta.seconds)       # 12600 (3*3600 + 30*60)
print(delta.total_seconds())  # 790200.0 (total duration in seconds)
```

### Arithmetic Operations

```python
from datetime import datetime, date, timedelta

today = date(2026, 11, 26)
now = datetime(2026, 11, 26, 14, 0, 0)

# Adding and subtracting durations
tomorrow = today + timedelta(days=1)
last_week = today - timedelta(weeks=1)
later = now + timedelta(hours=5, minutes=45)

print(tomorrow)   # 2026-11-27
print(last_week)  # 2026-11-19
print(later)      # 2026-11-26 19:45:00

# Difference between two dates/datetimes yields a timedelta
new_year = date(2027, 1, 1)
diff = new_year - today
print(diff)       # 36 days, 0:00:00
print(diff.days)  # 36

# Arithmetic on timedeltas themselves
d1 = timedelta(hours=2)
d2 = timedelta(minutes=45)
print(d1 + d2)     # 2:45:00
print(d1 - d2)     # 1:15:00
print(d2 * 3)      # 2:15:00
print(d1 / 2)      # 1:00:00
print(d1 // d2)    # 2 (integer ratio)
print(abs(-d1))    # 2:00:00
```

> **Note:** `timedelta` does not support `months` or `years` because their lengths vary (28–31 days, 365–366 days). For calendar-aware month/year shifts, use `dateutil.relativedelta` or handle month/year rollovers explicitly.

---

## 6. Formatting Dates (`strftime`)

The `strftime(format)` method (**str**ing **f**ormat **time**) converts a `date`, `time`, or `datetime` object into a formatted string using format directives.

```python
from datetime import datetime

dt = datetime(2026, 11, 26, 14, 5, 9)

# Standard readable formats
print(dt.strftime("%Y-%m-%d"))                 # 2026-11-26
print(dt.strftime("%d/%m/%Y"))                 # 26/11/2026
print(dt.strftime("%B %d, %Y"))                # November 26, 2026
print(dt.strftime("%a, %d %b %Y %I:%M %p"))    # Thu, 26 Nov 2026 02:05 PM
print(dt.strftime("%Y-%m-%d %H:%M:%S"))        # 2026-11-26 14:05:09

# Using Python f-strings directly (calls __format__, which uses strftime codes)
print(f"Date: {dt:%Y-%m-%d}, Time: {dt:%H:%M}")  # Date: 2026-11-26, Time: 14:05

# ISO 8601 shorthand (prefer .isoformat() when standard ISO format is needed)
print(dt.isoformat())          # 2026-11-26T14:05:09
print(dt.isoformat(sep=" "))   # 2026-11-26 14:05:09
```

---

## 7. Parsing Date Strings (`strptime`)

The `datetime.strptime(date_string, format)` class method (**str**ing **p**arse **time**) parses a string into a `datetime` object according to an explicit format pattern.

```python
from datetime import datetime

# Basic parsing
dt1 = datetime.strptime("2026-11-26", "%Y-%m-%d")
print(dt1)  # 2026-11-26 00:00:00

# Parsing date + 12-hour time with AM/PM
dt2 = datetime.strptime("26/11/2026 02:30 PM", "%d/%m/%Y %I:%M %p")
print(dt2)  # 2026-11-26 14:30:00

# Parsing human-readable month names
dt3 = datetime.strptime("November 26, 2026 - 14:05:09", "%B %d, %Y - %H:%M:%S")
print(dt3)  # 2026-11-26 14:05:09

# Extract just the date or time after parsing
only_date = datetime.strptime("2026-11-26", "%Y-%m-%d").date()
only_time = datetime.strptime("14:30", "%H:%M").time()
print(only_date, only_time)  # 2026-11-26 14:30:00
```

### Handling Invalid Formats Safely

If the input string does not strictly match the format string, `strptime` raises a `ValueError`.

```python
from datetime import datetime

def parse_date_safe(text: str, formats: tuple[str, ...]) -> datetime | None:
    """Try multiple date formats and return the first match, or None."""
    for fmt in formats:
        try:
            return datetime.strptime(text.strip(), fmt)
        except ValueError:
            continue
    return None

print(parse_date_safe("26-11-2026", ("%Y-%m-%d", "%d-%m-%Y")))  # 2026-11-26 00:00:00
print(parse_date_safe("invalid", ("%Y-%m-%d", "%d-%m-%Y")))     # None
```

> **Tip:** In Python 3.11+, `datetime.fromisoformat()` accepts almost any valid ISO 8601 string (including `Z` for UTC, optional separators, and week dates like `"2026-W48-4"`), making it faster and cleaner than `strptime` for ISO timestamps.

---

## 8. Format Codes (`%Y`, `%m`, `%d`, `%H`, `%M`, `%S`, etc.)

Common `strftime()` / `strptime()` format directives (example reference: `2026-11-26 14:05:09.000123+05:30`, Thursday):

| Code | Meaning | Example |
| :--- | :--- | :--- |
| `%Y` | Year with century (4 digits) | `2026` |
| `%y` | Year without century (zero-padded) | `26` |
| `%m` | Month as zero-padded decimal (`01`–`12`) | `11` |
| `%b` | Abbreviated month name | `Nov` |
| `%B` | Full month name | `November` |
| `%d` | Day of the month (`01`–`31`) | `26` |
| `%a` | Abbreviated weekday name | `Thu` |
| `%A` | Full weekday name | `Thursday` |
| `%w` | Weekday as decimal (`0`=Sunday .. `6`=Saturday) | `4` |
| `%u` | ISO 8601 weekday (`1`=Monday .. `7`=Sunday) | `4` |
| `%j` | Day of the year (`001`–`366`) | `330` |
| `%W` | Week number of year (Monday first day, `00`–`53`) | `47` |
| `%V` | ISO 8601 week number (`01`–`53`) | `48` |
| `%H` | Hour (24-hour clock, `00`–`23`) | `14` |
| `%I` | Hour (12-hour clock, `01`–`12`) | `02` |
| `%p` | Locale's AM or PM | `PM` |
| `%M` | Minute (`00`–`59`) | `05` |
| `%S` | Second (`00`–`59`) | `09` |
| `%f` | Microsecond (`000000`–`999999`) | `000123` |
| `%z` | UTC offset (`±HHMM` or `±HH:MM`) | `+0530` |
| `%Z` | Timezone name | `IST`, `UTC` |
| `%c` | Locale's appropriate date and time | `Thu Nov 26 14:05:09 2026` |
| `%x` | Locale's appropriate date | `11/26/26` |
| `%X` | Locale's appropriate time | `14:05:09` |
| `%%` | Literal `%` character | `%` |

> **Platform note:** On Linux/macOS, adding `-` (e.g., `%-d`, `%-m`, `%-I`) removes leading zeros in `strftime`, while Windows uses `#` (`%#d`). Neither is portable across OSes—use `f"{dt.day}"` when portability matters.

---

## 9. `time` Module (`time.time()`, `time.sleep()`, `time.perf_counter()`)

The lower-level `time` module wraps OS-level C time functions. It is mainly used for **Unix timestamps**, **delays**, and **high-resolution benchmarking**.

### Key Clocks and Functions

| Function | Purpose | Monotonic? |
| :--- | :--- | :--- |
| `time.time()` | Current Unix epoch time in seconds (float). Can jump if system clock changes (NTP/manual). | No |
| `time.time_ns()` | Same as `time.time()`, returning integer nanoseconds. | No |
| `time.monotonic()` | Always-increasing clock for measuring elapsed wall time. | Yes |
| `time.perf_counter()` | Highest-resolution monotonic timer for benchmarking (includes sleep time). | Yes |
| `time.process_time()` | CPU execution time of the current process (excludes sleep/I/O wait). | Yes |
| `time.sleep(secs)` | Suspend execution of the current thread for `secs` seconds (int or float). | — |

### Examples

```python
import time

# 1. Unix timestamp
epoch_seconds = time.time()
print(f"Seconds since 1970-01-01 UTC: {epoch_seconds:.3f}")

# 2. Sleeping / rate limiting
print("Waiting 0.5 seconds...")
time.sleep(0.5)
print("Done.")

# 3. High-resolution benchmarking with perf_counter()
start = time.perf_counter()
total = sum(range(1_000_000))
elapsed = time.perf_counter() - start
print(f"Computed {total} in {elapsed:.6f} seconds")

# 4. Formatting struct_time
local_struct = time.localtime()  # time.struct_time in local timezone
utc_struct = time.gmtime()       # time.struct_time in UTC
print(time.strftime("%Y-%m-%d %H:%M:%S", local_struct))
print(time.asctime(utc_struct))  # e.g. 'Thu Nov 26 08:50:00 2026'
```

> **Best Practice:** Never use `time.time()` to measure elapsed duration or timeouts—system clock adjustments (like NTP sync) can make `end - start` negative or inaccurate. Always use `time.perf_counter()` for benchmarks and `time.monotonic()` for timeouts.

---

## 10. `calendar` Module

The `calendar` module handles calendar-level operations such as leap years, month lengths, weekday names, and text/HTML calendar layouts.

```python
import calendar

# Leap year checks
print(calendar.isleap(2024))             # True
print(calendar.isleap(2026))             # False
print(calendar.leapdays(2000, 2026))     # 7 (leap years in [2000, 2026))

# First weekday of month (0=Mon .. 6=Sun) and number of days in month
first_weekday, num_days = calendar.monthrange(2026, 2)
print(f"Feb 2026 starts on weekday {first_weekday} and has {num_days} days")  # weekday 6 (Sun), 28 days

# Weekday for a specific date
print(calendar.weekday(2026, 11, 26))    # 3 (Thursday)

# Localized/standard names
print(list(calendar.day_name))           # ['Monday', 'Tuesday', ..., 'Sunday']
print(list(calendar.month_name)[1:])     # ['January', 'February', ..., 'December']

# Print a formatted month calendar
print(calendar.month(2026, 11))

# Matrix of weeks in a month (0 represents days outside the month)
cal = calendar.Calendar(firstweekday=calendar.SUNDAY)
print(cal.monthdayscalendar(2026, 11))
```

### Converting UTC `struct_time` to Timestamp

`calendar.timegm()` is the inverse of `time.gmtime()`—useful when working with UTC tuples:

```python
import calendar
import time

utc_tuple = (2026, 11, 26, 12, 0, 0, 0, 0, 0)
ts = calendar.timegm(utc_tuple)
print(ts)
print(time.gmtime(ts))
```

---

## 11. Timezone Handling (`pytz`, `zoneinfo`)

Modern Python supports the IANA Time Zone Database (e.g., `"America/New_York"`, `"Asia/Kolkata"`, `"Europe/London"`).

### Option A: `zoneinfo` (Built-in, Python 3.9+ — Recommended)

`zoneinfo` is part of the standard library and handles DST transitions accurately without external dependencies (on Windows, install the `tzdata` package via `pip install tzdata`).

```python
from datetime import datetime, timezone
from zoneinfo import ZoneInfo

# Create an aware datetime directly in a target timezone
ny_tz = ZoneInfo("America/New_York")
tokyo_tz = ZoneInfo("Asia/Tokyo")
ist_tz = ZoneInfo("Asia/Kolkata")

dt_ny = datetime(2026, 11, 26, 9, 0, tzinfo=ny_tz)
print("New York:", dt_ny.isoformat())  # 2026-11-26T09:00:00-05:00

# Convert between timezones with .astimezone()
dt_tokyo = dt_ny.astimezone(tokyo_tz)
dt_utc = dt_ny.astimezone(timezone.utc)
dt_ist = dt_ny.astimezone(ist_tz)

print("Tokyo:   ", dt_tokyo.isoformat())  # 2026-11-26T23:00:00+09:00
print("UTC:     ", dt_utc.isoformat())    # 2026-11-26T14:00:00+00:00
print("Kolkata: ", dt_ist.isoformat())    # 2026-11-26T19:30:00+05:30

# Attach a timezone to an existing naive datetime (does NOT shift clock time)
naive = datetime(2026, 7, 1, 12, 0)
aware = naive.replace(tzinfo=ny_tz)
print("Summer NY:", aware.isoformat())  # 2026-07-01T12:00:00-04:00 (EDT)
```

#### Handling DST Ambiguity (`fold`)

During a fall-back DST transition, the same local clock time occurs twice. Python uses `fold=0` (first occurrence) and `fold=1` (second occurrence):

```python
from datetime import datetime
from zoneinfo import ZoneInfo

ny_tz = ZoneInfo("America/New_York")

# 2026-11-01 01:30 AM happened twice in New York (EDT -> EST)
first = datetime(2026, 11, 1, 1, 30, tzinfo=ny_tz, fold=0)
second = datetime(2026, 11, 1, 1, 30, tzinfo=ny_tz, fold=1)

print(first.isoformat())   # 2026-11-01T01:30:00-04:00 (EDT)
print(second.isoformat())  # 2026-11-01T01:30:00-05:00 (EST)
```

### Option B: `pytz` (Third-Party Legacy Library)

Before Python 3.9, `pytz` (`pip install pytz`) was the standard library for IANA timezones. Unlike `zoneinfo`, **`pytz` must NOT be passed directly to `datetime(..., tzinfo=...)`** because it can attach historical offsets instead of modern DST rules. Always use `tz.localize()` and `tz.normalize()`.

```python
from datetime import datetime
import pytz  # third-party package

ny_tz = pytz.timezone("America/New_York")
tokyo_tz = pytz.timezone("Asia/Tokyo")

# Correct way in pytz: use .localize()
naive_dt = datetime(2026, 11, 26, 9, 0)
ny_dt = ny_tz.localize(naive_dt)
print("New York (pytz):", ny_dt.isoformat())  # 2026-11-26T09:00:00-05:00

# Converting to another timezone
tokyo_dt = ny_dt.astimezone(tokyo_tz)
print("Tokyo (pytz):   ", tokyo_dt.isoformat())  # 2026-11-26T23:00:00+09:00

# After timedelta arithmetic across DST boundaries, call normalize()
from datetime import timedelta
dst_dt = ny_tz.localize(datetime(2026, 10, 31, 12, 0))
next_day = ny_tz.normalize(dst_dt + timedelta(days=1))
print("Normalized:     ", next_day.isoformat())
```

### `zoneinfo` vs `pytz` Summary

| Feature | `zoneinfo` (Python 3.9+) | `pytz` (Third-party) |
| :--- | :--- | :--- |
| Installation | Built-in (`tzdata` on Windows) | `pip install pytz` |
| Attaching to `datetime` | `datetime(..., tzinfo=tz)` works safely | Must use `tz.localize(naive_dt)` |
| Arithmetic across DST | Automatically adjusts offset | Requires `tz.normalize(dt)` |
| Recommendation | **Use for all new code** | Legacy codebases only |

---

## 12. Comparing Dates

`date`, `time`, and `datetime` objects support all standard comparison operators (`==`, `!=`, `<`, `<=`, `>`, `>=`) and can be sorted or passed to `min()` / `max()`.

```python
from datetime import date, datetime, timezone
from zoneinfo import ZoneInfo

# Comparing dates
d1 = date(2026, 5, 1)
d2 = date(2026, 11, 26)
print(d1 < d2)   # True
print(d1 == d2)  # False

# Sorting dates
dates = [date(2026, 12, 1), date(2026, 1, 15), date(2026, 6, 30)]
print(sorted(dates))        # [date(2026, 1, 15), date(2026, 6, 30), date(2026, 12, 1)]
print("Earliest:", min(dates))
print("Latest:  ", max(dates))

# Comparing aware datetimes across different timezones
utc_dt = datetime(2026, 11, 26, 14, 0, tzinfo=timezone.utc)
ny_dt = datetime(2026, 11, 26, 9, 0, tzinfo=ZoneInfo("America/New_York"))
ist_dt = datetime(2026, 11, 26, 20, 0, tzinfo=ZoneInfo("Asia/Kolkata"))

print(utc_dt == ny_dt)  # True (same instant in time)
print(ny_dt < ist_dt)   # True (14:00 UTC < 14:30 UTC)
```

### Pitfalls When Comparing

1. **Naive vs Aware**: Comparing (`<`, `>`, `<=`, `>=`) a naive `datetime` with an aware `datetime` raises `TypeError: can't compare offset-naive and offset-aware datetimes` (`==` returns `False`).
2. **`date` vs `datetime`**: Comparing a `date` with a `datetime` raises `TypeError` for `<`/`>` and returns `False` for `==` (even though `datetime` is a subclass of `date`). Convert using `dt.date()` first.

```python
from datetime import date, datetime

d = date(2026, 11, 26)
dt = datetime(2026, 11, 26, 10, 0)

print(d == dt)         # False
print(d == dt.date())  # True
```

---

## 13. Date Ranges and Iterations

Python's `datetime` module does not include a built-in `daterange()` function, so iterating over dates or time intervals is typically written with a generator using `timedelta`.

### Generating a Range of Dates

```python
from datetime import date, timedelta
from typing import Iterator

def date_range(start: date, end: date, step: timedelta = timedelta(days=1)) -> Iterator[date]:
    """Yield dates from start (inclusive) up to end (exclusive)."""
    current = start
    while current < end:
        yield current
        current += step

for d in date_range(date(2026, 11, 24), date(2026, 11, 29)):
    print(d)
# 2026-11-24
# 2026-11-25
# 2026-11-26
# 2026-11-27
# 2026-11-28
```

### Generating Hourly / Custom Time Slots

```python
from datetime import datetime, timedelta
from typing import Iterator

def datetime_range(start: datetime, end: datetime, step: timedelta) -> Iterator[datetime]:
    """Yield datetimes from start (inclusive) to end (exclusive)."""
    current = start
    while current < end:
        yield current
        current += step

start_dt = datetime(2026, 11, 26, 9, 0)
end_dt = datetime(2026, 11, 26, 12, 0)

slots = [dt.strftime("%H:%M") for dt in datetime_range(start_dt, end_dt, timedelta(minutes=30))]
print(slots)  # ['09:00', '09:30', '10:00', '10:30', '11:00', '11:30']
```

### Filtering Weekdays in a Date Range

```python
from datetime import date, timedelta

start = date(2026, 11, 23)
end = date(2026, 11, 30)

# Mon=0 .. Fri=4 are weekdays; Sat=5, Sun=6 are weekend
business_days = [
    d for offset in range((end - start).days)
    if (d := start + timedelta(days=offset)).weekday() < 5
]
print(business_days)
```

---

## 14. Common Date/Time Patterns and Recipes

### 1. Golden Rule: Store in UTC, Display in Local Timezone

```python
from datetime import datetime, timezone
from zoneinfo import ZoneInfo

# Store / serialize in UTC
created_at_utc = datetime.now(timezone.utc)
stored_iso = created_at_utc.isoformat()  # '2026-11-26T08:50:44.123456+00:00'

# Read back and display in user's timezone
loaded_utc = datetime.fromisoformat(stored_iso)
user_local = loaded_utc.astimezone(ZoneInfo("Asia/Kolkata"))
print(user_local.strftime("%d %b %Y, %I:%M %p %Z"))
```

### 2. Calculate Age in Years

```python
from datetime import date

def calculate_age(birth_date: date, today: date | None = None) -> int:
    today = today or date.today()
    # Subtract 1 if birthday hasn't occurred yet this year
    had_birthday = (today.month, today.day) >= (birth_date.month, birth_date.day)
    return today.year - birth_date.year - (0 if had_birthday else 1)

print(calculate_age(date(1995, 12, 15), today=date(2026, 11, 26)))  # 30
print(calculate_age(date(1995, 10, 15), today=date(2026, 11, 26)))  # 31
```

### 3. Start and End of Day / Month

```python
import calendar
from datetime import date, datetime, time

d = date(2026, 11, 26)

# Start and end of day
start_of_day = datetime.combine(d, time.min)  # 2026-11-26 00:00:00
end_of_day = datetime.combine(d, time.max)    # 2026-11-26 23:59:59.999999

# First and last day of the month
first_of_month = d.replace(day=1)
_, last_day = calendar.monthrange(d.year, d.month)
last_of_month = d.replace(day=last_day)

print(start_of_day, "|", end_of_day)
print(first_of_month, "|", last_of_month)
```

### 4. Adding Business Days (Skipping Weekends)

```python
from datetime import date, timedelta

def add_business_days(start: date, n: int) -> date:
    """Advance start date by n business days (Monday-Friday)."""
    current = start
    remaining = n
    step = 1 if n >= 0 else -1
    while remaining != 0:
        current += timedelta(days=step)
        if current.weekday() < 5:
            remaining -= step
    return current

print(add_business_days(date(2026, 11, 26), 3))  # 2026-12-01 (skips Sat/Sun)
```

### 5. Human-Readable Elapsed Time (Duration Formatting)

```python
from datetime import timedelta

def format_duration(td: timedelta) -> str:
    total_seconds = int(td.total_seconds())
    days, rem = divmod(total_seconds, 86400)
    hours, rem = divmod(rem, 3600)
    minutes, seconds = divmod(rem, 60)

    parts = []
    if days:
        parts.append(f"{days}d")
    if hours:
        parts.append(f"{hours}h")
    if minutes:
        parts.append(f"{minutes}m")
    if seconds or not parts:
        parts.append(f"{seconds}s")
    return " ".join(parts)

print(format_duration(timedelta(days=2, hours=5, minutes=12, seconds=8)))  # 2d 5h 12m 8s
```

### 6. Simple Timing Context Manager

```python
import time
from contextlib import contextmanager

@contextmanager
def timer(label: str):
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"[{label}] {elapsed:.4f}s")

with timer("Sum loop"):
    sum(range(500_000))
```

---

## Quick Reference Cheat Sheet

| Task | Recommended Code |
| :--- | :--- |
| Current local date | `date.today()` |
| Current UTC aware datetime | `datetime.now(timezone.utc)` |
| Convert to another timezone | `dt.astimezone(ZoneInfo("America/New_York"))` |
| Parse ISO 8601 string | `datetime.fromisoformat(s)` |
| Parse custom date string | `datetime.strptime(s, "%d/%m/%Y")` |
| Format date/time as string | `dt.strftime("%Y-%m-%d %H:%M:%S")` |
| Add/subtract duration | `dt + timedelta(days=7, hours=2)` |
| Measure elapsed wall time | `time.perf_counter()` |
| Pause execution | `time.sleep(1.5)` |
| Check leap year / month days | `calendar.isleap(y)` / `calendar.monthrange(y, m)` |
