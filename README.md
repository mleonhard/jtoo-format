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
- Example: `{msg:"你好"}` encodes as bytes `7b 6d 73 67 3a 22 e4 bd a0 e5 a5 bd 22 7d`

# Atom

- An atom is a sequence of one or more characters in `_abcdefghijklmnopqrstuvwxyz` followed by zero or more characters from `_abcdefghijklmnopqrstuvwxyz0123456789`
- A protocol or format specification MUST include all possible atoms present in a valid message.  This means that implementers can include static strings for matching atom values.  Processing an atom must not require allocating memory.
- Examples:
    - `_`
    - `abc`
    - `a1`
    - `a_b`
    - `_1`
- Parsers MUST reject atoms with upper-case letters or characters from other languages.

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
- Parsers MUST reject strings with other escaped codepoints.

# Byte String

- A byte string is a sequence of zero or more bytes, prefixed with `B`.
- Each byte is encoded in lower-case hexadecimal format.
- Examples:
    - `B` (`""`)
    - `B61` (`"a"`)
    - `B4f4b` (`"OK"`)
- The parser rejects byte strings with whitespace: `B 4f`, `B4f 4b`.

# Boolean

`Y` or `N`

# Integer

- Examples:
    - `0`
    - `1`
    - `-1`
    - `1_000`
- Parsers MUST reject integers with unnecessary leading zeros: `00`, `000`, etc.

# Decimal

- Decimals are numbers with a fractional part.
- Libraries MUST NOT lose information when encoding or decoding JTOO decimal values. This type is suitable for currency.
- Examples:
    - `0.0`
    - `1.0`
    - `-1.0`
    - `1_000.0`
- Parsers MUST reject decimal numbers with unnecessary leading or trailing zeros: `00.0`, `0.00`, etc.

# Floating Point

- An IEEE-754 floating point number.
- Examples:
    - `0.0e0`
    - `-0.0e0`
    - `1.0e2` (100.0)
    - `1.0e-2` (0.01)
    - `1_000.1e0`
    - `-1.001e0`
    - `-1.000_01e0`
    - `NaN`
    - `Inf`
    - `-Inf`
- Parsers MUST reject floating point numbers with unnecessary leading or trailing zeros: `00.0e0`, `0.00e0`, `0.0e00`, etc.

# Time

Use ISO-8601 to interpret dates and times.
Note that JTOO allows only one format for each kind of value.

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
- timestamp-second: seconds since the epoch, `S0` (the unix epoch, 1980-01-01:00:00:00Z), `S1_709_528_240`
- timestamp-millisecond`S1_709_528_240.001`
- timestamp-microsecond: `S1_709_528_240.000_001`
- timestamp-nanosecond: `S1_709_528_240.000_000_001` (milli-seconds)

# List

- `[`, then a sequence of zero or more values of any type, separated by `,`, then `]`.
- Examples:
    - `[]`
    - `[1,2,3,4]`
    - `[t,f]`
    - `[(1,2),(3,4)]`

# Set

- `(`, then a sequence of zero or more values of any type, separated by `,`, then `)`.
- The parser MUST reject sets with duplicate entries.
- Parsers should not preserve the order of entries.  If you need order, then use a list, not a set.
- Examples:
    - `()`
    - `(1,2,3)`
    - `(t,f)`
    - `([1,2],[3,4])`

# Map

- `{`, then a sequence of zero or more key=value elements of any type, separated by `,`, then `}`.
- The parser MUST reject maps with duplicate keys.
- Parsers should not preserve the order of entries.  If you need order, then use a list, not a map.
- Examples:
    - `{}`
    - `{a=1,b=2}`
    - `{"a"=1,"b"=2}`
    - `{t="a"}`
    - `{T01:02=t}`
    - `{a=[1,2,3],b=(x,y,z)}`

# HTOO

JTOO libraries may include separate functions for parsing a related format called HTOO,
a less-strict *Human-writable* variant of JTOO.

MIME Type: `application/htoo`

An HTOO parser MUST behave like a JTOO parser and also MUST:

- Allow end-of-line comments: `1 // This is a comment.`
- Allow interior non-nesting comments: `["a" /* This is a comment. */,"b"]`.
- Allow trailing commas
    - `[1,]`
    - `(1,)`
    - `{a=1,}`
- Allow whitespace between elements
    - `[1 ]`
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
- Allow decimal numbers with unnecessary trailing zeros: `0.00`
- Allow numbers without `_` separators.
- Allow numbers with `_` separators between any two digits.
- Allow timezones with unnecessary `00` minutes: `-0800`, `+0500`.

# Acknowledgements

- Thanks to Emma Wang for suggesting the name, "J-two".
- Thanks to the authors of other interchange formats for their development and documentation:
    - JSON https://www.json.org/
    - Amazon Ion https://amazon-ion.github.io/ion-docs/docs.html
    - Protocol Buffers https://protobuf.dev/
