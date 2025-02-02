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

# DateTime

Use ISO-8601 to interpret dates and times.
Note that JTOO allows only one format for each kind of value.
JTOO can represent dates 0001-01-01 through 9999-12-31.

A JTOO datetime has the format `D` year `-` month `-` day `T` hour `-` minute `-` second offset.

- year: `0001` - `9999`
- month: `01` - `12`
- day: `01` - `31`
- hour: `00` - `23`
- minute: `00` - `59`
- second:
    - `00` - `60`
    - `00.000` - `60.999`
    - `00.000` _ `000-60.999_999`
    - `00.000` _ `000_000-60.999_999_999`
- offset
    - `Z` (UTC)
    - `+01` through `+23`
    - `+0001` through `+2359`
    - `-01` through `-23`
    - `-0001` through `-2359`
    - Parsers MUST reject offset values with a `00` minute part. Example: `+0800`.

Examples:

- `D2023-01-01T10:20:30Z`
- `D2023-01-01T10:20:30.400+1`
- `D2023-01-01T10:20:30.400_500-08`
- `D2023-01-01T10:20:30.400_500_600+0530`

# Timestamp

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
- Allow offsets with `00` minute part: `-0800`, `+0500`.
- Allow zero offsets, treat them as `Z`: `-00`, `+00`, `-0000`, `+0000`
- Allow extra zeros in offsets: `+01000`
- Allow single-digit offset hour: `-1`, `+1`
- Allow colons in offsets: `+5:30`
- Allow truncated dates and times and missing offset. For missing parts, use values from `D1970-01-01T00:00:00Z`.
    - `D2025` (year)
    - `D2025-1` (month)
    - `D2025T-1` (year with offset)
    - `D2025-1-1` (date)
    - `D2025-1T-1` (month with offset)
    - `D2025-1-1-1` (date with offset)
    - `D2025-1-1T-1` (date with offset)
    - `D2025-1-1T0-1` (hour with offset)
    - `D-1` (only offset)
    - `DT-1` (only offset)

# JTOO Frame Format

When programs communicate, they usually do so by sending discrete messages.
When communicating through a byte stream connection, like TCP, they need a way to show which bytes belong to each message.
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
- To prevent request smuggling and cross-protocol attackes, programs MUST immediately terminate the connection if the greeting contains:
    - duplicate keys
    - keys with characters not in `a-z`, `0-9`, `_`

Example greeting: `000043 000000 [["protocol","e7/LyniPSK"],["id",Bc8fd0fb7],["nonce",B8da5ab6f3fdb1bb0]] \n`

If a program receives a connection and a greeting, but it does not support the specified `protocol` version, it must
respond with: `00002d 000000 [["code",505],["error","unsupported protocol"]] \n`

# Acknowledgements

- Thanks to Emma Wang for suggesting the name, "J-two".
- Thanks to the authors of other interchange formats for their development and documentation:
    - JSON https://www.json.org/
    - Amazon Ion https://amazon-ion.github.io/ion-docs/docs.html
    - Protocol Buffers https://protobuf.dev/
