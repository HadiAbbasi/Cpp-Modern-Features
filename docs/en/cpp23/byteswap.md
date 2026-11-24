<div align="center">

[🇺🇸 English](./byteswap.md) | [🇮🇷 فارسی](../../fa/cpp23/byteswap.md)

</div>



# `std::byteswap` 

C++23 introduced **`std::byteswap`** in the `<bit>` header, providing a standardized, portable, and constexpr way to reverse the bytes of an integral value. This is extremely useful when dealing with binary file I/O, network protocols (converting between Big-Endian and Little-Endian), or hardware communication.

---

## Why Do We Need `std::byteswap`?
Different computer architectures store multi-byte integers differently:
* **Little-Endian** (e.g., x86/x64, ARM): The least significant byte is stored at the lowest memory address.
* **Big-Endian** (e.g., network byte order, some older architectures): The most significant byte is stored at the lowest memory address.

Before C++23, developers had to rely on non-portable compiler extensions (like `__builtin_bswap32` in GCC/Clang or `_byteswap_ulong` in MSVC) or write manual bit-shifting code. `std::byteswap` standardizes this operation across all platforms.

---

## Syntax and Header

To use `std::byteswap`, include the `<bit>` header:

```cpp
#include <bit>

// Signature (since C++23)
template< class T >
constexpr T byteswap( T n ) noexcept;
```

* **Constraints**: `T` must be an integral type (e.g., `uint16_t`, `uint32_t`, `uint64_t`, `int`, etc.). Floating-point types are not supported directly.
* **`constexpr`**: It can be evaluated at compile-time.
* **`noexcept`**: It never throws exceptions.

---

## Complete Example

Here is a complete program demonstrating how to use `std::byteswap` with 16-bit, 32-bit, and 64-bit integers, including hex representation to clearly see the byte order reversal:

```cpp
#include <bit>
#include <iostream>
#include <iomanip>

int main() {
    // 1. 32-bit Integer Example
    uint32_t original32 = 0x12345678;
    uint32_t swapped32 = std::byteswap(original32);

    std::cout << "--- 32-bit Integer ---" << std::endl;
    std::cout << "Original: 0x" << std::hex << std::setfill('0') << std::setw(8) << original32 << std::endl;
    std::cout << "Swapped:  0x" << std::setw(8) << swapped32 << std::endl; // Output: 0x78563412

    // 2. 16-bit Integer Example
    uint16_t original16 = 0xABCD;
    uint16_t swapped16 = std::byteswap(original16);

    std::cout << "\n--- 16-bit Integer ---" << std::endl;
    std::cout << "Original: 0x" << std::setw(4) << original16 << std::endl;
    std::cout << "Swapped:  0x" << std::setw(4) << swapped16 << std::endl; // Output: 0xCDAB

    // 3. Compile-time evaluation (constexpr)
    constexpr uint32_t compileTimeSwap = std::byteswap(0x01020304U);
    static_assert(compileTimeSwap == 0x04030201U);
    std::cout << "\nCompile-time byteswap verified successfully via static_assert!" << std::endl;

    return 0;
}
```

---

## Practical Use Case: Converting Network Byte Order

Network protocols typically use Big-Endian format. If your machine is Little-Endian, you can conditionally or explicitly swap bytes:

```cpp
#include <bit>
#include <iostream>
#include <cstdint>

uint32_t readNetworkData(uint32_t netValue) {
    // If the system is little-endian, network order (big-endian) needs to be swapped
    if constexpr (std::endian::native == std::endian::little) {
        return std::byteswap(netValue);
    } else {
        return netValue;
    }
}

int main() {
    uint32_t packetData = 0x00000001; // Representing a big-endian network value
    uint32_t hostValue = readNetworkData(packetData);
    std::cout << "Host value: " << hostValue << std::endl;
}
```

---

## Key Takeaways
1. **Header**: `#include <bit>`
2. **Portability**: Replaces old platform-specific intrinsics (`__builtin_bswap`, `_byteswap_ul`, etc.).
3. **Safety & Efficiency**: It works out-of-the-box for all standard integer types and compiles down to highly optimized hardware instructions (like `bswap` on x86) when available.
4. **`constexpr` Support**: Enables byte-swapping constants at compile time.

---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>