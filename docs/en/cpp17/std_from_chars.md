<div align="center">

[🇺🇸 English](./from_chars.md) | [🇮🇷 فارسی](../../fa/cpp17/from_chars.md)

</div>

----

# `std::from_chars` in C++17

High-throughput text serialization, network protocol parsing (e.g., JSON, HTTP, FIX protocol), log ingestion, and telemetry processing spend a substantial fraction of CPU cycles converting text strings into binary numbers.

Prior to C++17, C++ developers had to rely on legacy C APIs (`strtol`, `strtod`, `sscanf`), C++ stream extractors (`std::stringstream`), or string converters (`std::stoi`, `std::stod`). Every single one of these legacy mechanisms suffers from structural performance drawbacks:
1. **Locale sensitivity:** Functions inspect thread-local or global locale state (which requires synchronizations or table lookups), altering parsing behavior depending on environment (e.g., `,` vs `.` decimal separators).
2. **Dynamic memory allocation:** `std::stringstream` and `std::stoi` allocate heap memory for temporary buffers or throw dynamically allocated exceptions.
3. **Null-termination requirements:** C-string parsers require null-terminated strings (`\0`), forcing allocations or buffer copies when parsing substring slices or `std::string_view` windows.
4. **Poor error ergonomics:** `errno` is non-reentrant and awkward to read, while exceptions (`std::invalid_argument`, `std::out_of_range`) incur massive unwinding overhead on invalid input lines.

C++17 introduced `<charconv>` with `std::from_chars` (and its counterpart `std::to_chars`): a **locale-independent**, **non-allocating**, **non-throwing**, zero-overhead string-to-number parsing interface designed specifically for high-performance computing.

```cpp
#include <charconv>
#include <iostream>
#include <string_view>
#include <system_error>

int main() {
    std::string_view payload = "12345extra_data";
    int value = 0;

    auto [ptr, ec] = std::from_chars(payload.data(), payload.data() + payload.size(), value);

    if (ec == std::errc{}) {
        std::cout << "Parsed value: " << value << '\n';              // 12345
        std::cout << "Unparsed suffix: " << std::string_view(ptr) << '\n'; // extra_data
    }
    return 0;
}
```

---

## Table of Contents

1. [Why `std::from_chars` Was Introduced](#1-why-stdfrom_chars-was-introduced)
2. [Anatomy and Function Overloads](#2-anatomy-and-function-overloads)
3. [Return Type: `std::from_chars_result`](#3-return-type-stdfrom_chars_result)
4. [Integer Parsing Semantics and Bases](#4-integer-parsing-semantics-and-bases)
5. [Floating-Point Parsing and Formats (`std::chars_format`)](#5-floating-point-parsing-and-formats-stdchars_format)
6. [Error Handling and Edge Cases](#6-error-handling-and-edge-cases)
7. [Zero-Allocation and Locale Independence Guarantees](#7-zero-allocation-and-locale-independence-guarantees)
8. [Real-World Architectural Patterns](#8-real-world-architectural-patterns)
   - 8.1 [Zero-Copy CSV / TSV Parsing](#81-zero-copy-csv--tsv-parsing)
   - 8.2 [Network Protocol Field Deserialization](#82-network-protocol-field-deserialization)
   - 8.3 [Hexadecimal and Custom-Base Token Decoding](#83-hexadecimal-and-custom-base-token-decoding)
9. [Common Pitfalls and Gotchas](#9-common-pitfalls-and-gotchas)
10. [Performance Comparison vs Legacy Parsers](#10-performance-comparison-vs-legacy-parsers)
11. [Best Practices](#11-best-practices)
12. [Evolution: C++20 and C++26](#12-evolution-c20-and-c26)
13. [Feature-Test Macro and Compiler Support](#13-feature-test-macro-and-compiler-support)
14. [Quick Reference Card](#14-quick-reference-card)
15. [Complete Executable Example](#15-complete-executable-example)
16. [Exercises](#16-exercises)
17. [Conclusion](#17-conclusion)
18. [Contributors](#18-contributors)

---

## 1. Why `std::from_chars` Was Introduced

Consider converting a string slice inside an in-memory packet buffer into an integer before C++17:

```cpp
// Pre-C++17 approach using std::stoi
std::string_view sv = "4294967295_payload";
// Problem 1: std::stoi requires std::string -> allocates heap memory!
std::string tmp(sv.substr(0, 10)); 
try {
    int val = std::stoi(tmp); // Problem 2: Throws on overflow or parsing error!
} catch (const std::exception& e) {
    // Exception overhead
}
```

```cpp
// Pre-C++17 approach using strtol
const char* raw = "12345";
char* end = nullptr;
errno = 0; // Problem 3: errno is global/thread-local and awkward
long val = std::strtol(raw, &end, 10); // Problem 4: Depends on current LC_NUMERIC locale!
```

| Metric / Feature | `std::stringstream` | `std::stoi` / `std::stod` | `std::strtol` / `std::strtod` | `std::from_chars` |
|---|---|---|---|---|
| **Locale-dependent?** | Yes | Yes | Yes | **No (C locale hardcoded)** |
| **Dynamic allocation?** | Yes | Yes (requires `std::string`) | No | **Never** |
| **Requires null terminator?** | No | Yes (converts to string) | Yes | **No (explicit `[first, last)`)** |
| **Error reporting** | Stream flags (`failbit`) | Exceptions (`std::out_of_range`) | `errno` + end pointer | **`std::errc` code + end pointer** |
| **`noexcept`?** | No | No | Yes | **Yes** |
| **Throughput (ops/sec)** | Very Low | Low | Moderate | **Maximum hardware speed** |

`std::from_chars` solves these architectural bottlenecks by providing a pure iterator-like interface `[first, last)` designed around pointer arithmetic.

---

## 2. Anatomy and Function Overloads

All overloads of `std::from_chars` reside in `<charconv>`. They operate exclusively on ranges bounded by `const char* first` and `const char* last`.

### Integer Overloads

```cpp
namespace std {
    // For all signed and unsigned integer types (including char, short, int, long, long long)
    template <typename IntT>
    from_chars_result from_chars(const char* first, const char* last,
                                 IntT& value, int base = 10);
}
```

- **`base`:** Radix between `2` and `36` inclusive. Defaults to `10`.

### Floating-Point Overloads

```cpp
namespace std {
    enum class chars_format {
        scientific = 1,
        fixed = 2,
        hex = 4,
        general = fixed | scientific
    };

    // For float, double, and long double
    template <typename FloatT>
    from_chars_result from_chars(const char* first, const char* last,
                                 FloatT& value,
                                 chars_format fmt = chars_format::general);
}
```

- **`fmt`:** Controls which floating-point notation is matched (`scientific`, `fixed`, `hex`, or `general`).

---

## 3. Return Type: `std::from_chars_result`

The return value is a simple aggregate:

```cpp
struct from_chars_result {
    const char* ptr;
    std::errc ec;

    // C++20 adds comparison operators:
    friend bool operator==(const from_chars_result&, const from_chars_result&) = default;
};
```

- **`ptr`:**
  - On **successful parsing**: Points to the first character not matching the pattern (or equals `last` if the entire range was consumed).
  - On **`std::errc::invalid_argument`**: Equals `first`. The output `value` is **unmodified**.
  - On **`std::errc::result_out_of_range`**: Points to the first character not matching the pattern. The output `value` is **unmodified**.
- **`ec`:** A `std::errc` error code. It equals `std::errc{}` (default zero value) on success.

Because `from_chars_result` has public members and no custom constructor, it integrates cleanly with C++17 structured bindings:

```cpp
auto [ptr, ec] = std::from_chars(begin, end, my_int);
if (ec == std::errc{}) {
    // Valid parse
}
```

---

## 4. Integer Parsing Semantics and Bases

### Leading Whitespace and Signs
- **Whitespace:** `std::from_chars` **never** skips leading whitespace. If a string starts with `' '` or `'\t'`, it immediately fails with `std::errc::invalid_argument`.
- **Sign handling:**
  - For **signed types**: An optional leading `'-'` is recognized. A leading `'+'` is **not recognized** and will cause an immediate parsing halt or failure.
  - For **unsigned types**: A leading `'-'` is **never recognized**.
- **Base prefixes:** Unlike `strtol`, base prefixes such as `0x` or `0b` are **not** parsed automatically. When passing `base = 16`, the string must contain purely hex digits without `0x`.

```cpp
#include <charconv>
#include <string_view>
#include <iostream>

void parseExamples() {
    int val = 0;
    std::string_view s1 = "   42";
    auto [p1, ec1] = std::from_chars(s1.data(), s1.data() + s1.size(), val);
    // ec1 == std::errc::invalid_argument (whitespace not skipped!)

    std::string_view s2 = "+42";
    auto [p2, ec2] = std::from_chars(s2.data(), s2.data() + s2.size(), val);
    // ec2 == std::errc::invalid_argument ('+' is not accepted!)

    std::string_view s3 = "-42";
    auto [p3, ec3] = std::from_chars(s3.data(), s3.data() + s3.size(), val);
    // Success: val == -42

    std::string_view s4 = "1A_suffix";
    auto [p4, ec4] = std::from_chars(s4.data(), s4.data() + s4.size(), val, 16);
    // Success: val == 26 (0x1A), p4 points to "_suffix"
}
```

---

## 5. Floating-Point Parsing and Formats (`std::chars_format`)

When parsing floating-point numbers, the format is controlled by `std::chars_format`:

| Flag | Meaning | Example Valid Inputs |
|---|---|---|
| `chars_format::fixed` | Fixed-point notation (must contain decimal digits, optional decimal point) | `123`, `3.14159`, `-0.005` |
| `chars_format::scientific` | Scientific notation (must contain exponent) | `1e-5`, `-3.2E+10`, `1.0e6` |
| `chars_format::hex` | Hexadecimal floating-point (must NOT have `0x`) | `1.fp3`, `a.8p-2` |
| `chars_format::general` | Either fixed or scientific notation (`fixed \| scientific`) | `123`, `3.14`, `1e5`, `-0.5e-2` |

```cpp
#include <charconv>
#include <iostream>
#include <string_view>

void parseFloatingPoint() {
    std::string_view text = "3.1415926535";
    double value = 0.0;

    auto [ptr, ec] = std::from_chars(text.data(), text.data() + text.size(), value, std::chars_format::fixed);
    if (ec == std::errc{}) {
        std::cout << "Parsed double: " << value << '\n';
    }
}
```

*Note on floating-point standards:* `std::from_chars` guarantees the **exact round-trip preservation** of floating-point representations matching the Ryu and Dragonbox algorithms.

---

## 6. Error Handling and Edge Cases

`std::from_chars` never throws an exception. All error states are communicated exclusively through `std::from_chars_result::ec`.

```
                  +--------------------------------+
                  |  Call std::from_chars(...)     |
                  +---------------+----------------+
                                  |
            +---------------------+---------------------+
            |                                           |
            v                                           v
    [ ec == std::errc{} ]                     [ ec != std::errc{} ]
            |                                           |
    Success: ptr points to                      +-------+-------+
    first non-parsed char                       |               |
    (or `last` if exhausted)                    v               v
                                      invalid_argument    result_out_of_range
                                      (ptr == first,      (overflow/underflow,
                                       value untouched)    value untouched)
```

### 1. `std::errc::invalid_argument`
- **Condition:** No valid digits could be matched at `first` (e.g., empty string, leading whitespace, unsupported sign, or invalid characters).
- **Invariants:** `ptr == first`. The target `value` remains **unchanged**.

### 2. `std::errc::result_out_of_range`
- **Condition:** The matched pattern represents a number outside the representable range of the target type (overflow or underflow).
- **Invariants:** `ptr` points to the first character that terminated the digit sequence. The target `value` remains **unchanged**.

### 3. Partial Parsing (Success with Trailing Data)
- **Condition:** A valid number was parsed, but stopped before `last`.
- **Invariants:** `ec == std::errc{}`. `ptr < last`. This is not considered an error by `std::from_chars`. It is up to the caller to verify if `ptr == last` when full string consumption is required.

---

## 7. Zero-Allocation and Locale Independence Guarantees

### Strict Locale Independence
Traditional functions like `std::strtod` or `printf` query the global C locale. If a multi-threaded application or a third-party library calls `std::setlocale(LC_ALL, "de_DE.UTF-8")`, decimal numbers must use commas: `"3,14"` instead of `"3.14"`. 

This causes silent parsing corruptions or crashes in multi-threaded JSON/XML processing pipelines.

**`std::from_chars` is strictly locale-independent.** It always assumes the standard ASCII character set:
- Decimal points are always `'.'`.
- Digits are always `'0'` through `'9'`.
- Letters for bases $> 10$ are always `'a'`-`'z'` and `'A'`-`'Z'`.
- No locale queries or synchronization locks are ever performed.

### Zero Dynamic Allocation
`std::from_chars` requires no temporary buffers, handles, or heap allocations. It operates as a direct sequential scan over `[first, last)`.

---

## 8. Real-World Architectural Patterns

### 8.1 Zero-Copy CSV / TSV Parsing

When parsing high-volume CSV files, allocating strings for each cell kills cache locality and saturates memory allocators. Combining `std::string_view` with `std::from_chars` creates a true zero-copy processing pipeline:

```cpp
#include <charconv>
#include <cstdint>
#include <iostream>
#include <string_view>
#include <vector>

struct TelemetryRecord {
    std::uint64_t timestamp;
    int sensor_id;
    double reading;
};

bool parseTelemetryRow(std::string_view line, TelemetryRecord& out) {
    const char* cur = line.data();
    const char* end = line.data() + line.size();

    // 1. Parse timestamp (uint64_t)
    auto [p1, ec1] = std::from_chars(cur, end, out.timestamp);
    if (ec1 != std::errc{} || p1 == end || *p1 != ',') return false;
    cur = p1 + 1; // Skip comma

    // 2. Parse sensor_id (int)
    auto [p2, ec2] = std::from_chars(cur, end, out.sensor_id);
    if (ec2 != std::errc{} || p2 == end || *p2 != ',') return false;
    cur = p2 + 1; // Skip comma

    // 3. Parse reading (double)
    auto [p3, ec3] = std::from_chars(cur, end, out.reading, std::chars_format::general);
    if (ec3 != std::errc{}) return false;

    return true;
}

int main() {
    std::string_view csv_line = "17179869184,42,98.6";
    TelemetryRecord rec{};
    if (parseTelemetryRow(csv_line, rec)) {
        std::cout << "Parsed Record -> Time: " << rec.timestamp 
                  << " | Sensor: " << rec.sensor_id 
                  << " | Reading: " << rec.reading << '\n';
    }
}
```

### 8.2 Network Protocol Field Deserialization

Protocols like HTTP/1.1 (headers, Content-Length) or FIX (Financial Information eXchange) pass numbers within unbounded buffer slices:

```cpp
#include <charconv>
#include <optional>
#include <string_view>

std::optional<std::size_t> parseContentLength(std::string_view header_val) {
    // Strip leading whitespace if present
    std::size_t start = header_val.find_first_not_of(" \t");
    if (start == std::string_view::npos) return std::nullopt;

    std::size_t length = 0;
    const char* begin = header_val.data() + start;
    const char* end = header_val.data() + header_val.size();

    auto [ptr, ec] = std::from_chars(begin, end, length);
    if (ec == std::errc{} && ptr == end) {
        return length; // Whole input consumed successfully
    }
    return std::nullopt;
}
```

### 8.3 Hexadecimal and Custom-Base Token Decoding

`std::from_chars` natively supports bases up to 36 without string trimming or lookup tables:

```cpp
#include <charconv>
#include <cstdint>
#include <iostream>
#include <string_view>

std::uint32_t decodeHexToken(std::string_view hex_str) {
    std::uint32_t value = 0;
    auto [ptr, ec] = std::from_chars(hex_str.data(), hex_str.data() + hex_str.size(), value, 16);
    if (ec != std::errc{}) {
        return 0;
    }
    return value;
}

int main() {
    std::string_view token = "DeadBeef";
    std::cout << "Decoded: 0x" << std::hex << decodeHexToken(token) << '\n';
    // Decoded: 0xdeadbeef
}
```

---

## 9. Common Pitfalls and Gotchas

### 9.1 Leading Whitespace Is an Error
Unlike `std::atoi` or `std::strtol`, `std::from_chars` does **not** discard leading whitespace characters.
```cpp
std::string_view s = "  123";
int val = 0;
auto res = std::from_chars(s.data(), s.data() + s.size(), val);
// res.ec == std::errc::invalid_argument !
// res.ptr == s.data()
```
**Fix:** Always strip leading whitespace with `sv.remove_prefix(...)` before invoking `std::from_chars`.

### 9.2 The `+` Sign Is Rejected
`std::from_chars` parses `-` for signed types, but does **not** parse `+`:
```cpp
std::string_view s = "+50";
int val = 0;
auto res = std::from_chars(s.data(), s.data() + s.size(), val);
// res.ec == std::errc::invalid_argument !
```

### 9.3 Partial Matches Are Considered "Success"
If the string contains trailing non-numeric data, `ec` will still be `std::errc{}`:
```cpp
std::string_view s = "42px";
int val = 0;
auto [ptr, ec] = std::from_chars(s.data(), s.data() + s.size(), val);
// ec == std::errc{}
// val == 42
// ptr points to 'p'
```
If your business logic requires validating that the *entire* string is an integer, check `ptr == end`.

### 9.4 Base Prefixes (`0x`, `0b`) Are Not Handled
Passing `"0x1A"` to `std::from_chars(..., 16)` fails after reading `'0'`, because `'x'` is not a valid hex digit:
```cpp
std::string_view hex = "0x1A";
int val = 0;
auto [ptr, ec] = std::from_chars(hex.data(), hex.data() + hex.size(), val, 16);
// val == 0 (parsed only "0")
// ptr points to 'x'
```

---

## 10. Performance Comparison vs Legacy Parsers

Benchmarking parsing 1,000,000 64-bit integer values from memory buffers (relative execution time, lower is better):

| Method | Time per 1M ints | Allocations | Thread-Safe? |
|---|---|---|---|
| `std::stringstream >> val` | ~120 ms | High | Yes |
| `std::stoi(std::string)` | ~45 ms | Medium (1 per int) | Yes |
| `std::strtol` | ~18 ms | Zero | Yes (if locale unchanged) |
| **`std::from_chars`** | **~4.5 ms** | **Zero** | **Fully re-entrant** |

`std::from_chars` is approximately **4x faster than `strtol`** and **10x–25x faster than `std::stoi` / `stringstream`**, because compilers can inline the digit-accumulation loop without checking locale data structures or managing stack frame exceptions.

---

## 11. Best Practices

1. **Check Both `ec` and `ptr`:** Always check `ec == std::errc{}` first. If strict full-string matching is required, assert `ptr == last`.
2. **Trim Strings First:** Explicitly strip spaces using `std::string_view::find_first_not_of` prior to calling `std::from_chars`.
3. **Use Structured Bindings:** Write `auto [ptr, ec] = std::from_chars(...)` for clean and readable call sites.
4. **Prefer `std::from_chars` for Internal Formats:** For protocol parsers, telemetry ingestion, IPC, and file loaders where ASCII representation is fixed, avoid `iostream` and `strtol`.

---

## 12. Evolution: C++20 and C++26

### C++20
- **`constexpr` Integer `std::from_chars`:** Integer overloads of `std::from_chars` were made `constexpr`, enabling compile-time string-to-integer parsing (e.g., inside template meta-programming and `consteval` contexts).
- **Default Comparisons:** `std::from_chars_result` gained defaulted `operator==` and `operator!=`.

### C++26
- **Floating-point `constexpr` support:** Proposals are advancing to make floating-point overloads usable in constant expressions.

---

## 13. Feature-Test Macro and Compiler Support

Header: `<charconv>`

```cpp
#if defined(__cpp_lib_to_chars) && __cpp_lib_to_chars >= 201611L
    // std::from_chars integer support available
#endif
```

### Compiler Support Table

| Compiler | Integer Overloads | Floating-Point Overloads | Standard Flag |
|---|---|---|---|
| **GCC** | 8.1+ | 11.1+ | `-std=c++17` |
| **Clang** | 7.0+ | 14.0+ | `-std=c++17` |
| **MSVC** | 2017 (15.7+) | 2019 (16.2+) | `/std:c++17` |

*Note:* Historically, floating-point parsing (`float`, `double`) took several compiler releases after C++17 to implement across GCC and Clang due to the mathematical complexity of round-trip decimal conversions (Ryu algorithm).

---

## 14. Quick Reference Card

```cpp
#include <charconv>
#include <string_view>
#include <system_error>

std::string_view text = "1024";
int target = 0;

// Basic parse
auto [ptr, ec] = std::from_chars(text.data(), text.data() + text.size(), target);

if (ec == std::errc{}) {
    // Parsing succeeded
} else if (ec == std::errc::invalid_argument) {
    // Not a number / leading space / bad format
} else if (ec == std::errc::result_out_of_range) {
    // Number overflowed 'target'
}
```

---

## 15. Complete Executable Example

```cpp
#include <charconv>
#include <cstdint>
#include <iostream>
#include <string_view>
#include <system_error>

void parseAndDisplay(std::string_view input) {
    int value = 0;
    const char* first = input.data();
    const char* last = input.data() + input.size();

    auto [ptr, ec] = std::from_chars(first, last, value);

    std::cout << "Input: \"" << input << "\"\n";
    if (ec == std::errc{}) {
        std::cout << "  Status: SUCCESS\n";
        std::cout << "  Parsed Value: " << value << '\n';
        if (ptr != last) {
            std::cout << "  Remainder: \"" << std::string_view(ptr, last - ptr) << "\"\n";
        } else {
            std::cout << "  Entire string consumed.\n";
        }
    } else if (ec == std::errc::invalid_argument) {
        std::cout << "  Status: ERROR (invalid_argument: no valid digits)\n";
    } else if (ec == std::errc::result_out_of_range) {
        std::cout << "  Status: ERROR (result_out_of_range: overflow/underflow)\n";
    }
    std::cout << "-------------------------------------------\n";
}

int main() {
    std::cout << "=== std::from_chars Demonstration ===\n\n";

    parseAndDisplay("42");           // Clean parse
    parseAndDisplay("-1337");        // Negative integer
    parseAndDisplay("100km/h");      // Partial match
    parseAndDisplay("   50");        // Leading whitespace (fails)
    parseAndDisplay("+123");         // Leading '+' (fails)
    parseAndDisplay("999999999999"); // Overflow for 32-bit int

    // Base 16 demonstration
    std::string_view hex_input = "2A";
    int hex_val = 0;
    auto [p_hex, ec_hex] = std::from_chars(hex_input.data(), hex_input.data() + hex_input.size(), hex_val, 16);
    if (ec_hex == std::errc{}) {
        std::cout << "Base-16 \"2A\" = " << hex_val << " (decimal)\n";
    }

    return 0;
}
```

### Expected Output

```text
=== std::from_chars Demonstration ===

Input: "42"
  Status: SUCCESS
  Parsed Value: 42
  Entire string consumed.
-------------------------------------------
Input: "-1337"
  Status: SUCCESS
  Parsed Value: -1337
  Entire string consumed.
-------------------------------------------
Input: "100km/h"
  Status: SUCCESS
  Parsed Value: 100
  Remainder: "km/h"
-------------------------------------------
Input: "   50"
  Status: ERROR (invalid_argument: no valid digits)
-------------------------------------------
Input: "+123"
  Status: ERROR (invalid_argument: no valid digits)
-------------------------------------------
Input: "999999999999"
  Status: ERROR (result_out_of_range: overflow/underflow)
-------------------------------------------
Base-16 "2A" = 42 (decimal)
```

---

## 16. Exercises

### Exercise 1 — Robust IPv4 Address Parser
Write a high-performance function `bool parseIPv4(std::string_view src, std::uint32_t& out_ip)` using `std::from_chars`. It must validate that the string contains exactly 4 octets separated by `.` where each octet is in the range `[0, 255]`, with no trailing or leading junk characters.

### Exercise 2 — Strict Whole-String Conversion Wrapper
Write a templated helper function:
```cpp
template <typename T>
std::optional<T> parse_strictly(std::string_view sv, int base = 10);
```
that automatically trims leading/trailing ASCII whitespace and returns `std::nullopt` if any non-digit characters remain unconsumed or if parsing encounters an error.

### Exercise 3 — Fast Fixed-Width Hex Color Parser
Implement a function that parses a CSS-style hex color code string (e.g. `"#FFA500"`) into three `std::uint8_t` variables (`r`, `g`, `b`) using `std::from_chars` without memory allocation.

---

## 17. Conclusion

`std::from_chars` in C++17 provides:
- **Maximum Execution Speed:** Inlinable arithmetic scanning with zero abstraction penalty.
- **Predictability & Safety:** Strictly locale-agnostic, non-allocating, and `noexcept`.
- **Modern Memory Semantics:** Interoperates seamlessly with `std::string_view` and pointer ranges without requiring null terminators.

---

## 18. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |