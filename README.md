# jtoo-format

JTOO is a structured human-readable data format. It is an alternative to JSON.

# Goals

- Human readable
- Well-defined data types matching the data types in most modern programming languages
- Compact
- Quickly parsed by machine

# MIME Type

`application/jtoo`

# Document

A JTOO document is a JTOO data type, encoded in UTF-8.

Example: `{msg:"你好"}` encodes as bytes `7b 6d 73 67 3a 22 e4 bd a0 e5 a5 bd 22 7d`

# Simple Types

- atom
    - one or more characters in `_abcdefghijklmnopqrstuvwxyz` followed by zero or more characters
      from `_abcdefghijklmnopqrstuvwxyz0123456789`
    - `_`
    - `abc`
    - `a1`
    - `a_b`
    - `_1`
    - Parsers must reject atoms with upper-case characters or characters from other languages.
- string
    - `""`
    - `"abc"`
    - `"She typed \22ok\22."`
    - `"C:\5cWindows`
    - These bytes must be escaped in lower-case hex format:
        - Control codes: `\00`, `\01`, `\02`, `\03`, `\04`, `\05`, `\06`, `\07`, `\08`, `\09` tab, `\0a` line
          feed, `\0b`, `\0c`, `\0d` carriage
          return, `\0e`, `\0f`, `\10`, `\11`, `\12`, `\13`, `\14`, `\15`, `\16`, `\17`, `\18`, `\19`, `\1a`, `\1b`, `\1c`, `\1d`, `\1e`, `\1f`, `\7f`
        - Double-quote: `\22`
        - Backslash: `\5c`
    - Parsers MUST reject other escaped bytes.
- byte string
    - Byte strings are encoded as pairs of lower-case hexadecimal digits, prefixed with `B`.
    - `B` (`""`)
    - `B61` (`"a"`)
    - `B4f4b` (`"OK"`)
    - The parser rejects byte strings with whitespace: `B 4f`, `B4f 4b`.
- boolean
    - `T`
    - `F`
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
    - `0.0e0`, `-0.0e0`
    - `1.0e2` (100.0)
    - `1.0-2` (0.01)
    - `1_000.1e0`
    - `-1.001e0`
    - `-1.000_1e0`
    - `nan`
    - `inf`, `-inf`
- A `_` separator may appear between two digits only.

# Date and Time Types

- Use ISO-8601 to interpret dates and times.
- Note that JTOO allows only one format for each kind of value.
- year: `D2023`
- month: `D2023-01`
- week: `D2023-W01`
- date: `D2023-01-01`
- hour: `T01` (00-23)
- minute: `T23:01` (00-59)
- second: `T01:02:03` (0-60)
- millisecond: `T01:02:03.004` (000-999)
- microsecond: `T01:02:03.004_005` (000_000-999_999)
- nanosecond: `T01:02:03.004_005_006` (000_000_000-999_999_999)
- date-hour: concatenate a date and hour, `D2023-12-30T01`
- date-minute: concatenate a date and minute: `D2023-12-30T01:02`
- date-second: concatenate a date and minute: `D2023-12-30T01:02:03`
- date-millisecond: concatenate a date and minute: `D2023-12-30T01:02:03.004`
- date-microsecond: concatenate a date and minute: `D2023-12-30T01:02:03.004_005`
- date-nanosecond: concatenate a date and minute: `D2023-12-30T01:02:03.004_005_006`
- timezone
    - `Z` (UTC)
    - `-08` (Pacific Standard Time)
    - `+0530` (Indian Standard Time)
- date-hour-timezone: concatenate a date, an hour, and a timezone, `D2023-01-01T01Z`
- date-minute-timezone: concatenate a date, a minute, and a timezone, `D2023-01-01T23:01-08`
- date-second-timezone: concatenate a date, a second, and a timezone, `D2023-01-01T01:02:03+0530`
- date-millisecond-timezone: concatenate a date, a millisecond, and a timezone, `D2023-01-01T01:02:03.004Z`
- date-microsecond-timezone: concatenate a date, a microsecond, and a timezone, `D2023-01-01T01:02:03.004_005Z`
- date-nanosecond-timezone: concatenate a date, a nanosecond, and a timezone, `D2023-01-01T01:02:03.004_005_006Z`
- timestamp-second: seconds since the epoch, `s0` (the unix epoch, 1980-01-01:00:00:00Z), `s1_709_528_240`
- timestamp-millisecond`s1_709_528_240.001`
- timestamp-microsecond: `s1_709_528_240.000_001`
- timestamp-nanosecond: `s1_709_528_240.000_000_001` (milli-seconds)

# Collection Types

- lists
    - `[1,2,3,4]`
    - `[t,f]`
    - `[]`
- sets
    - `(1,2,3)`
    - `(t,f)`
    - `()`
    - The parser must reject sets with duplicate entries.
- maps
    - Any type can be used as the key.
    - `{"a"=1,"b"=2}`
    - `{t="a"}`
    - `{T01:02=t}`
    - `{}`
    - The parser must reject maps with duplicate keys.
- Collections may nest. Examples: `[[1,2],[3,4]]`, `{"a"=[1,2],"b"=[3,4,5]}`.

# Human-Written Mode

A parser may support a *human-written mode* which is less strict.
In human-written mode, a parser must:

- Allow line comments: `1 // This is a comment.`
- Allow inline comments: `["a" /* This is a comment. */,"b"]`
- Allow trailing commas
    - `[1,]`
    - `{a=1,}`
- Allow whitespace
    - `[1, 2]`
    - `{a=1, b=2}`
    - ```
      {
        a= 1,
        b=22,
      }
      ```
- Allow these non-standard character escapes: `\t`, `\r`, `\n`, `\"`
- Allow byte strings to contain upper-case hexadecimal digits.
- Allow numbers without `_` separators.
- Allow numbers with `_` separators between any two digits.
- Allow timezones with unnecessary `00` minutes: `-0800`, `+0500`.
