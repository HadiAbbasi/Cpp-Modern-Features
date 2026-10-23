<div align="center">

[🇺🇸 English](./byte.md) | [🇮🇷 فارسی](../../fa/cpp17/byte.md)

</div>

# std::byte

The **`std::byte`** data type is a significant and practical feature introduced in **C++17**, specifically designed to represent **raw memory bytes**.

Here is a complete guide on why it was introduced, how it works, and practical examples of its usage.

---

## Why Do We Need `std::byte`?

Before C++17, developers typically used the following types to work with raw bytes of memory:
* `char`
* `unsigned char`
* `uint8_t`

**What was the problem?**
All of these types are inherently **arithmetic** or **character** types:
1. `char` is meant for textual characters (`'A'`, `'b'`).
2. `unsigned char` and `uint8_t` are unsigned numeric types; the compiler allows you to perform arithmetic operations like addition (`+`), subtraction (`-`), or multiplication (`*`) on them.

However, **a raw memory byte is neither a character nor a number**; it is merely a collection of bits (a bit pattern). Mathematical concepts like "multiplying two bytes of memory together" make no sense in this context.

To solve this and ensure **type safety**, `std::byte` was introduced:
* **You cannot** perform arithmetic operations (`+`, `-`, `*`) on it.
* **You cannot** implicitly assign it to or from characters or integers without an explicit cast.
* **The only allowed operations** are bitwise operations such as `&`, `|`, `^`, `~`, `<<`, and `>>`.


## Syntax & Header

To use `std::byte`, you need to include the `<cstddef>` header:

```cpp
#include <cstddef>
```

Under the hood, the standard library implements `std::byte` as an `enum class`:
```cpp
enum class byte : unsigned char {};
```

## Initialization & Basic Usage

Because `std::byte` is an `enum class` without implicit conversions, initialization requires specific syntax:

```cpp
#include <iostream>
#include <cstddef>

int main() {
    // Direct list-initialization
    std::byte b1{0};
    std::byte b2{0xFF}; // 255 in hex
    std::byte b3{0b1010'0101}; // C++14 binary literal

    // Error! Implicit conversion from int is not allowed:
    // std::byte b4 = 10; 

    // Explicit cast from numeric values:
    int x = 42;
    std::byte b5 = static_cast<std::byte>(x);
}
```

## Allowed Operations (Bitwise Operations)

You can perform all logical bitwise operations on `std::byte`:

```cpp
#include <iostream>
#include <cstddef>

int main() {
    std::byte b1{0b0000'1111};
    std::byte b2{0b1111'0000};

    // Bitwise OR
    std::byte b_or = b1 | b2; // 0b1111'1111

    // Bitwise AND
    std::byte b_and = b1 & b2; // 0b0000'0000

    // Bitwise XOR
    std::byte b_xor = b1 ^ b2;

    // Bitwise shifts
    std::byte shifted = b1 << 2; // 0b0011'1100
}
```

## Converting to Integer via `std::to_integer`

If you want to print a `std::byte` or read its numerical value, you must use the helper function `std::to_integer`:

```cpp
#include <iostream>
#include <cstddef>

int main() {
    std::byte b{0x2A}; // Number 42 in hex

    // Convert to int or another integer type
    int val = std::to_integer<int>(b);

    std::cout << "Integer value: " << val << '\n'; // Output: 42
}
```

## Practical Application: Creating a Raw Memory Buffer

A very common scenario is reading data from a network socket, a binary file, or directly manipulating memory:

```cpp
#include <iostream>
#include <cstddef>
#include <vector>

void process_raw_data(const std::byte* data, std::size_t size) {
    for (std::size_t i = 0; i < size; ++i) {
        // Read and convert bytes
        unsigned int byte_val = std::to_integer<unsigned int>(data[i]);
        std::cout << "Byte[" << i << "] = 0x" << std::hex << byte_val << '\n';
    }
}

int main() {
    // A buffer of raw bytes
    std::vector<std::byte> buffer = {
        std::byte{0x01},
        std::byte{0xFA},
        std::byte{0x10}
    };

    process_raw_data(buffer.data(), buffer.size());
}
```

---

## Summary Table

| Feature | `char` / `unsigned char` | `std::byte` |
| :--- | :--- | :--- |
| **Primary Purpose** | Text characters / Unsigned numbers | Raw memory bytes (Bit patterns) |
| **Arithmetic operations (`+`, `-`)** | Yes (Allowed) | **No** (Compile-time error) |
| **Bitwise operations (`&`, `|`, `<<`)** | Yes | **Yes** |
| **Implicit conversion from numbers** | Yes | **No** (Requires explicit cast) |
| **Standard header** | Built-in | `<cstddef>` |


---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>
