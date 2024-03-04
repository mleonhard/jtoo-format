# jtoo-format

JTOO is a structured human-readable data format. It is an alternative to JSON.

# Goals

- Human readable
- Well-defined data types matching the data types in most modern programming languages
- Compact
- Quickly parsed by machine

# MIME Type

`application/jtoo`

# Simple Types

- string
    - `""`
    - `"abc"`
    - `"She typed \"ok\"."`
    - Other allowed escapes: `\r`, `\n`, `\t`.
- byte string
    - Byte strings are encoded as pairs of upper-case hexadecimal digits, prefixed with `b`.
    - `b` (`""`)
    - `b61` (`"a"`)
    - `b4F4B` (`"OK"`)
    - In human-written mode, the parser accepts lower-case hexadecimal digits.
    - In human-written mode, the parser does NOT allow whitespace to appear anywhere inside a byte string definition.
      These are always parsing errors: `b 4F`, `b4F 4B`.
- boolean
    - `t`
    - `f`
- integer
  - `0`
  - `1`
  - `-1`
  - `1_000`
- decimal
  - `0.0`
  - `1.0`
  - `-1.0`
  - `1_000.0`
- floating point (64-bit double-precision IEEE-754 binary64)
    - `0.0e0`
    - `1.0e2` (100.0)
    - `1.0-2` (0.01)
    - `1_000.1e0`
    - `-1.001e0`
    - `-1.000_1`
    - These values are not allowed: `-0.0`, `nan`, `inf`, `-inf`
    - A `_` separator may NOT appear after the `e` in a floating point number.
- In human-written mode, the parser accepts numbers that are missing `_` separators.
- A `_` separator may appear between two digits only.

# Date and Time Types

- Use ISO-8601 to interpret dates and times.
- Note that JTOO allows only one format for each kind of value.
- Values may contain a date part, a time part, or a date and time part.
- year: `D2023`
- month: `D2023-12`
- week: `D2023-W01`
- date: `D2023-12-30`
- time
    - `T01` (hour, `00` is 12am, `23` is 11pm)
    - `T23:01` (minute, 0-59)
    - `T01:02:03` (second, 0-60)
    - `T01:02:03.004` (milli-second)
    - `T01:02:03.004_005` (micro-second)
    - `T01:02:03.004_005_006` (nano-second)
- date-time
    - concatenate a date and time
    - Examples: `D2023-12-30T01`, `D2023-12-30T01:02:03`
- Optional timezone suffix may appear after a year, month, week, date, time, or date-time.
    - `Z` (UTC)
    - `-08` (Pacific Standard Time)
    - `+0530` (Indian Standard Time)
- timestamp, seconds since the epoch
    - `s0` (the unix epoch, 1980-01-01:00:00:00Z)
    - `s1_709_528_240`
    - `s1_709_528_240.001` (milli-seconds)
    - `s1_709_528_240.000_001` (milli-seconds)
    - `s1_709_528_240.000_000_001` (milli-seconds)

# Collection Types

- lists
    - `[1,2,3,4]`
    - `[t,f]`
- maps
    - Any type can be used as the key.
    - `{"a":1,"b":2}`
    - `{t "a a"}`
    - `{T01:02 t}`
    - In human-written mode, the parser ignores a trailing comma.
- Collections may nest. Examples: `[[1,2],[3,4]]`, `{"a":[1,2],"b":[3,4,5]}`.

# Comments and Whitespace

- In human-written mode, the parser silently ignores comments and extra whitespace between elements:
    - `1 // This is a comment.`
    - `{a 1, b 2}`
    - `{\n  a  1,\n  b 22,\n}`
