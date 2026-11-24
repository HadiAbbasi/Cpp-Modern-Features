<div align="center">

[🇺🇸 English](./to_underlying.md) | [🇮🇷 فارسی](../../fa/cpp23/to_underlying.md)

</div>

# `std::to_underlying` 

**`std::to_underlying()`** is a useful C++23 utility, declared in the `<utility>` header. It converts an enum value to its **underlying integer type**—the type used to store the enum’s values.


##  The Problem Before C++23

With scoped enums (`enum class`), converting an enum value to a number requires an explicit cast. Before C++23, you might write:

```cpp
#include <cstdint>
#include <type_traits>

enum class Status : std::uint8_t {
    OK = 0,
    NotFound = 404,
    Error = 500
};

auto value = static_cast<std::underlying_type_t<Status>>(Status::OK);
```

This works, but it is more verbose than necessary. If the enum’s underlying type changes, spelling out that type manually can also lead to mistakes.


## The C++23 Solution

With `std::to_underlying()`, the conversion is simpler:

```cpp
#include <cstdint>
#include <iostream>
#include <utility>

enum class ErrorCode : std::uint16_t {
    Success = 0,
    Timeout = 1001,
    AccessDenied = 1002
};

int main() {
    ErrorCode code = ErrorCode::Timeout;

    auto raw_value = std::to_underlying(code);

    std::cout << "Error code: " << raw_value << '\n';
    std::cout << "Size: " << sizeof(raw_value) << " bytes\n";
}
```

Here, `raw_value` has the enum’s underlying type, `std::uint16_t`.


##  How It Works

Conceptually, the function is similar to:

```cpp
template <typename Enum>
constexpr std::underlying_type_t<Enum> to_underlying(Enum value) noexcept {
    return static_cast<std::underlying_type_t<Enum>>(value);
}
```

Key properties:

- **`constexpr`**: It can be evaluated at compile time.
- **`noexcept`**: It does not throw exceptions.
- **Enum-only use**: It is intended for enum values, not ordinary integers or unrelated types.

---

## Common Uses

### Bit flags

```cpp
#include <cstdint>
#include <utility>

enum class Permissions : std::uint8_t {
    Read    = 1 << 0,
    Write   = 1 << 1,
    Execute = 1 << 2
};

auto mask =
    std::to_underlying(Permissions::Read) |
    std::to_underlying(Permissions::Write);
```

### Serialization and network protocols

It’s also useful when writing enum values to a file or byte buffer, or passing them to an API that expects an integer.


## Summary

- **Header:** `<utility>`
- **Purpose:** Converts an enum value to its underlying integer type.
- **Benefit:** Avoids manually naming that type, making code shorter and less error-prone.
- **Standard:** C++23.

---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>