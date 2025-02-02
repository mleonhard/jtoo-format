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
- Parsers MUST reject strings with these codepoints un-escaped or with other escaped codepoints. This helps prevent request smuggling attacks.

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

- year: `D2023` (0001-9999)
- month: `D2023-01` (01-12)
- day: `D2023-01-01` (01-31)
- week: `D2023-W01` (01-53)
- week_day: `D2023-W01-1` (1-7)

Time values:

- hour: `T10` (00-23)
- minute: `T10:20` (00-59)
- second: `T10:20:30` (0-60)
- millisecond: `T10:20:30.400` (000-999)
- microsecond: `T10:20:30.400_500` (000_000-999_999)
- nanosecond: `T10:20:30.400_500_600` (000_000_000-999_999_999)

Offset values:

- No offset: `Z` (UTC)
- Positive hour offset: `+01` (Central European Time)
- Negative hour offset: `~08` (Pacific Standard Time).
  - Note: This uses the tilde instead of the minus symbol, so parsers can distinguish year-offset vs year-month values and year-month-offset vs year-month-day values.
- Hour and minute offset: `+0530` (Indian Standard Time)
- Parsers MUST reject offset values with `00` minute part. Example: `+0800`

Combinations of date + offset, week_date + offset, time + offset, date + time + offset, and week_date + time + offset. Examples:

- year-offset: `D2023Z`
- year-month-day: `D2023-12-08`
- year-month-offset: `D2023-12~08`
- year-month-offset: `D2023-12+08`
- date-hour: concatenate a date and hour, `D2023-12-30T01`
- date-minute: concatenate a date and minute: `D2023-12-30T01:02`
- date-second-offset: concatenate a date and second: `D2023-12-30T01:02:03~08`

Timestamp values:

- timestamp-second: seconds since the epoch, `S0` (the unix epoch, 1980-01-01:00:00:00Z), `S1_709_528_240`
- timestamp-millisecond`S1_709_528_240.001`
- timestamp-microsecond: `S1_709_528_240.000_001`
- timestamp-nanosecond: `S1_709_528_240.000_000_001`

TODO: Add support for time durations.

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
- Allow offsets with `00` minute part: `-0800`, `+0500`.

# JTOO Frame Format

When programs communicate, they usually do so by sending discrete messages.
When communicating through a byte stream connection, like TCP, they need a way to show show which bytes belong to each message.
This is called message framing.

Message framing schemes affect the complexity of the programs.
Some common framing schemes, starting with simple and increasing complexity:

- Length prefix - Framing code is completely de-coupled from message processing code.
- Newline-delimited - Framing code must encode and decode newlines inside messages.
- JSON objects - Framing code is coupled with JSON parser.

The JTOO frame format uses length prefixes.

A JTOO consists of these parts: the length of the JTOO section, a space, the length of the binary section, a space, the JTOO section, a space, the binary section, and a newline.
Each length is exactly 6 lower-case hexadecimal characters encoding big-endian unsigned integers.
Each length may be up to one byte less than 16 MiB.

Examples:

- A frame with zero-length JTOO and binary sections: `000000 000000 \n`
- A frame with a short JTOO section: `00000E 000000 [["code":200]] \n`
- A frame with a short binary section: `000000 000011  A binary message\0\n`
- A frame with both sections: `000016 000009 [["status":"success"]] LzsGqqfC9\n`

# JTOO Protocols

When programs communicate using a protocol based on the JTOO frame format, the initiating program sends a greeting message which is a JTOO frame.
- The greeting frame must be 16 KiB or shorter.
- The greeting frame JTOO section is a list of key-value pairs.
- There must be a single "protocol" key. Programs MUST immediately terminate the connection if the greeting contains duplicate keys, to prevent request smuggling and cross-protocol attacks.

Example greeting: `000043 000000 [["protocol":"e7/LyniPSK","id":Bc8fd0fb7,"nonce":B8da5ab6f3fdb1bb0]] \n`

If a program receives a connection and a greeting, but it does not support the specified `protocol` version, it must respond with: `00002d 000000 [["code":505,"error":"unsupported protocol"]] \n`

# Acknowledgements

- Thanks to Emma Wang for suggesting the name, "J-two".
- Thanks to the authors of other interchange formats for their development and documentation:
    - JSON https://www.json.org/
    - Amazon Ion https://amazon-ion.github.io/ion-docs/docs.html
    - Protocol Buffers https://protobuf.dev/
