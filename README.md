# jtoo-format

JTOO is a structured human-readable data format. It is an alternative to JSON.

Author: Michael Leonhard https://tamale.net/

# Goals

- Human readable
- Well-defined data types matching the data types in most modern programming languages
- Compact
- Quickly parsed by machine

# MIME Type

`application/jtoo`

# Document

- A JTOO document is a single instance of a JTOO data type, encoded in UTF-8.
- Example: `["msg":"你好"]` encodes as bytes `5b 22 6d 73 67 22 3a 22 e4 bd a0 e5 a5 bd 22 5d`

# String

- A string is a sequence of zero or more Unicode codepoints (characters).
- Examples:
    - `""`
    - `"abc"`
    - `"She typed \22ok\22."`
    - `"C:\5cWindows`
- Parsers MUST accept these codepoints escaped in lower-case hex format:
    - Control codes: `\00`, `\01`, `\02`, `\03`, `\04`, `\05`, `\06`, `\07`, `\08`, `\09` tab, `\0a` line feed, `\0b`, `\0c`, `\0d` carriage return, `\0e`, `\0f`, `\10`, `\11`, `\12`, `\13`, `\14`, `\15`, `\16`, `\17`, `\18`, `\19`, `\1a`, `\1b`, `\1c`, `\1d`, `\1e`, `\1f`, `\7f`
    - Double-quote: `\22`
    - Backslash: `\5c`
- Parsers MUST reject strings with these codepoints un-escaped or with other escaped codepoints.  This helps prevent request smuggling attacks.

# Byte String

- A byte string is a sequence of zero or more bytes, prefixed with `B`.
- Each byte is encoded in lower-case hexadecimal format.
- Examples:
    - `B` (`""`)
    - `B61` (`"a"`)
    - `B4f4b` (`"OK"`)
- Parsers must reject byte strings with whitespace: `B 4f`, `B4f 4b`.

# Boolean

`Y` or `N`

# Integer

- Base-10
- Groups of three digits must be separated from the rest of the number with `_`, starting from the least-significant digits.
- Examples:
    - `0`
    - `1`
    - `-1`
    - `1_000`
- Parsers MUST reject integers with unnecessary leading zeros: `00`, `000`, etc.

# Decimal

- Decimals are numbers with a fractional part.
- Examples:
    - `0.0`
    - `1.0`
    - `-1.0`
  - `1_000.0`
  - `0.000_1`
- Groups of three digits must be separated from the rest of the number with `_`, starting from the decimal point.
- Parsers MUST reject decimal numbers with unnecessary leading zeros: `00.0`, `01.0`, etc.

# Time

Use ISO-8601 to interpret dates and times.
Note that JTOO allows only one format for each kind of value.
JTOO can represent dates 0001-01-01 through 9999-12-31.

Date values:
- year: `D2023`
- month: `D2023-01`
- date: `D2023-01-01`
- week: `D2023-W01`
- week_date: `D2023-W01-01`

Time values:
- hour: `T01` (00-23)
- minute: `T23:01` (00-59)
- second: `T01:02:03` (0-60)
- millisecond: `T01:02:03.004` (000-999)
- microsecond: `T01:02:03.004_005` (000_000-999_999)
- nanosecond: `T01:02:03.004_005_006` (000_000_000-999_999_999)

Timezone offset values:
- `Z` (UTC)
- Hour offset: `-08` (Pacific Standard Time)
- Hour and minute offset: `+0530` (Indian Standard Time)
- Parsers MUST reject timezone offset values with `00` minute part. Example: `+0800`

Combinations of date + timezone offset, week_date + timezone offset, time + timezone offset, date + time + timezone offset, and week_date + time + timezone offset.  Examples:
- year-timezone: `D2023Z`
- date-hour: concatenate a date and hour, `D2023-12-30T01`
- date-minute: concatenate a date and minute: `D2023-12-30T01:02`
- date-second-timezone: concatenate a date and second: `D2023-12-30T01:02:03-08`

Timestamp values:
- timestamp-second: seconds since the epoch, `S0` (the unix epoch, 1980-01-01:00:00:00Z), `S1_709_528_240`
- timestamp-millisecond`S1_709_528_240.001`
- timestamp-microsecond: `S1_709_528_240.000_001`
- timestamp-nanosecond: `S1_709_528_240.000_000_001`

# List

- `[`, then a sequence of zero or more values of any type, separated by `,`, then `]`.
- Examples:
    - `[]`
    - `[1,2,3,4]`
    - `[t,f]`
    - `[[1,2],[3,4]]`

# HTOO

JTOO libraries may include separate functions for parsing a related format called HTOO,
a less-strict *Human-writable* variant of JTOO.

MIME Type: `application/htoo`

An HTOO parser MUST behave like a JTOO parser and also MUST:

- Allow end-of-line comments: `1 // This is a comment.`
- Allow interior non-nesting comments: `["a" /* This is a comment. */,"b"]`.
- Allow trailing commas
    - `[1,]`
    - `[[1,2,],]`
- Allow whitespace between elements
    - `[1 ]`
    - `[1, 2]`
    - `[["a", 1], ["b", 2]]`
    - ```
      [
        ["a", 1],
        ["b", 2],
      ]
      ```
- Allow these non-standard character escapes: `\t`, `\r`, `\n`, `\"`
- Allow byte strings to contain upper-case hexadecimal digits.
- Allow decimal numbers with unnecessary trailing zeros: `0.00`
- Allow numbers without `_` separators.
- Allow numbers with `_` separators between any two digits.
- Allow timezones with `00` minute part: `-0800`, `+0500`.

# Acknowledgements

- Thanks to Emma Wang for suggesting the name, "J-two".
- Thanks to the authors of other interchange formats for their development and documentation:
    - JSON https://www.json.org/
    - Amazon Ion https://amazon-ion.github.io/ion-docs/docs.html
    - Protocol Buffers https://protobuf.dev/
