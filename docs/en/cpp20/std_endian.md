<div align="center">

[🇺🇸 English](./std_endian.md) | [🇮🇷 فارسی](../../fa/cpp20/std_endian.md)

</div>

---

# A Comprehensive Guide to `std::endian` in C++20

## Introduction

Endianness specifies the order in which the bytes of a multi-byte value are addressed when the value is stored in memory. This detail is usually hidden in programs that work only with the language's abstract values. However, it becomes important when working with binary files, network protocols, shared memory, hardware, serialization, and data produced on different architectures.

C++20 introduced the `std::endian` enumeration in the `<bit>` header. It provides a standard, compile-time way to query the byte order of scalar types in the execution environment. This feature does not convert data, define the byte order of a file or protocol, or replace proper data-format design. It only provides information about the current execution environment.

## Understanding Byte Order

### Byte Order in Multi-Byte Values

To understand endianness, consider a 32-bit integer with the hexadecimal representation `0x12345678`. If this value is stored in four consecutive bytes, the two common byte orders look like this:

| Increasing memory address | Big-endian | Little-endian |
|---|---|---|
| First address | `0x12` | `0x78` |
| Second address | `0x34` | `0x56` |
| Third address | `0x56` | `0x34` |
| Fourth address | `0x78` | `0x12` |

In Big-endian order, the most significant byte (MSB) is stored at the lowest address. In Little-endian order, the least significant byte (LSB) is stored at the lowest address.

This distinction describes only the byte order in the memory representation. The abstract numeric value is still `0x12345678` in either environment. The difference becomes visible when inspecting an object's representation as bytes or transferring data between environments without the appropriate conversion.

### Endianness Versus Bit Order

Byte order should not be confused with bit transmission order or bit numbering. Endianness concerns the order of the bytes of a multi-byte value in memory. Bit order within a byte and the order in which bits are transmitted over a hardware interface are separate concerns and must be handled according to the relevant format or protocol specification.

### Common Byte Orders and Other Cases

- **Little-endian:** The least significant byte is stored at the lowest address. This order is common among many modern processors.
- **Big-endian:** The most significant byte is stored at the lowest address. Many binary formats and network conventions define a specific byte order independently of the host's endianness.
- **Mixed-endian:** Byte order is not consistent across all scalar types, or their representations do not follow the simple Big-endian or Little-endian pattern. Such environments are less common today, but the standard does not disregard them conceptually.

## Introducing `std::endian`

### Header and Standard Version

`std::endian` has been defined in the standard `<bit>` header since C++20. It is a Standard Library feature, not a language keyword or a compiler-specific extension.

```cpp
#include <bit>

constexpr std::endian native_order = std::endian::native;
```

The conceptual definition of this type is:

```cpp
namespace std {
    enum class endian {
        little = /* implementation-defined */,
        big    = /* implementation-defined */,
        native = /* implementation-defined */
    };
}
```

The underlying numeric values of the enumerators are implementation-defined. Therefore, do not assume that `little` is necessarily zero, `big` is necessarily one, or `native` has any particular numeric value. Portable code should compare the enumerators with one another rather than rely on their underlying numeric values.

### Meaning of `little` and `big`

The value `std::endian::little` represents Little-endian byte order, while `std::endian::big` represents Big-endian byte order. In an environment where multi-byte scalar types consistently use one byte order, these two values are distinct.

### Meaning of `native`

The value `std::endian::native` describes the native byte order of the execution environment. It can be used to inspect the current platform:

- If all scalar types are Little-endian, `native` is equal to `little`.
- If all scalar types are Big-endian, `native` is equal to `big`.
- In a Mixed-endian environment, `native` is equal to neither `little` nor `big`.
- If all scalar types have a size of exactly one byte, all three values—`little`, `big`, and `native`—are equal, because multi-byte ordering is not relevant in this case.

The important point is that `native` is not necessarily synonymous with Little-endian. A program that checks only the Little-endian and Big-endian cases must also handle the third case explicitly.

## Checking Endianness at Compile Time

### Simple Detection with `if constexpr`

Because the values of `std::endian` are compile-time constants, `if constexpr` can be used to select the appropriate branch:

```cpp
#include <bit>
#include <iostream>

int main() {
    if constexpr (std::endian::native == std::endian::little) {
        std::cout << "Native byte order: little-endian\n";
    } else if constexpr (std::endian::native == std::endian::big) {
        std::cout << "Native byte order: big-endian\n";
    } else {
        std::cout << "Native byte order: mixed or otherwise non-standard\n";
    }
}
```

This approach is suitable for selecting an architecture-dependent implementation, enabling an optimized code path, or reporting environment information. An `if constexpr` statement discards an unusable branch at compile time, although all branches must still be syntactically valid.

### Using `static_assert`

If a component supports only a particular byte order, that restriction can be enforced at compile time:

```cpp
#include <bit>

static_assert(
    std::endian::native == std::endian::little,
    "This component requires a little-endian target"
);
```

This pattern is appropriate for code that genuinely depends on a hardware-specific contract, such as a highly specialized low-level component. However, using it in a general-purpose library or an application intended to run on multiple architectures unnecessarily limits portability. In such cases, explicitly converting the bytes is usually preferable to rejecting the platform.

### Detection in Generic Code

In template code, `std::endian::native` can participate in compile-time decisions. Keep in mind that this information describes the execution environment and implementation, not the template type by itself. The destination type, its size, and the data contract must still be considered separately.

## Standard Guarantees and Scope

### Relationship to Scalar Types

The standard defines `std::endian` to describe the endianness of scalar types in the execution environment. In the C++ type model, scalar types include categories such as arithmetic types and pointer types. This concept should not be casually equated with every class type or composite structure.

For a `struct` containing multiple members, `std::endian` does not specify the overall member order, padding, or structure layout. Even when the members are scalar types, member placement and padding are separate concerns.

### One-Byte Types

If every scalar type in an implementation has a size of one byte, the standard permits all three enumerators to have the same value. Consequently, the following condition can be true in such an environment:

```cpp
std::endian::native == std::endian::little
```

However, this equality does not necessarily indicate a distinguishable byte order. It simply represents the special case in which all scalar types are one byte wide.

### Mixed-Endian Environments

In a Mixed-endian environment, `native` is equal to neither `little` nor `big`. Therefore, the following code is incomplete:

```cpp
if constexpr (std::endian::native == std::endian::little) {
    // Little-endian path
} else {
    // Incorrectly assumes every other platform is big-endian
}
```

The `else` branch is not necessarily Big-endian. If support for such environments matters, handle the third case explicitly or document and enforce the platform restriction.

## Object Representation and Memory

### What Is Object Representation?

Every object in C++ has an Object Representation: the sequence of bytes used to represent the object in memory. Endianness is one of the properties that can affect the byte order of the representation of multi-byte scalar types.

However, Object Representation involves more than endianness. Type size, padding bits, permitted representations, alignment, and type-specific rules also matter. Therefore, inspecting an object's bytes and storing them directly in a file does not, by itself, produce a portable serialization format.

### Inspecting the Bytes of a Value

The byte representation of an object can be inspected through `unsigned char` or `std::byte`. The following example prints the byte order of a 32-bit value:

```cpp
#include <cstdint>
#include <iostream>

int main() {
    const std::uint32_t value = 0x12345678;
    const auto* bytes = reinterpret_cast<const unsigned char*>(&value);

    for (std::size_t i = 0; i < sizeof(value); ++i) {
        std::cout << std::hex
                  << static_cast<unsigned int>(bytes[i]) << ' ';
    }

    std::cout << '\n';
}
```

This code is useful for inspecting an object's representation during the current program execution. On a Little-endian platform, the output will typically be `78 56 34 12`; on a Big-endian platform, it will typically be `12 34 56 78`. This example is not an alternative to `std::endian` for detecting byte order, nor is it sufficient for converting input or output data.

### The Role of `std::byte`

The `std::byte` type was introduced in C++17 to represent raw byte data. It can be used when working with buffers and object representations, but it does not perform any endianness conversion by itself.

```cpp
#include <array>
#include <bit>
#include <cstddef>
#include <cstdint>

int main() {
    const std::uint32_t value = 0x12345678;
    const auto bytes = std::bit_cast<std::array<std::byte, sizeof(value)>>(value);

    (void)bytes;
}
```

In this example, C++20's `std::bit_cast` transfers the object representation into an array of the same size. The array's byte order is the native representation of the value on the current platform; this operation does not convert it to a specific Little-endian or Big-endian format.

## Converting Byte Order

### Detection Is Not Conversion

`std::endian` only reports information about the environment. It does not provide a function such as `to_big_endian` or `swap_bytes`. To convert data, the destination format must be specified, and the bytes must then be rearranged according to that format.

For example, if a protocol defines an integer as Big-endian, the program must convert the value to a Big-endian representation regardless of the host's endianness. Merely checking `native` and writing the memory directly is not sufficient.

### Using `std::byteswap` in C++23

The `std::byteswap` function was introduced in C++23 in the standard `<bit>` header. It reverses the bytes of an integer value, but the program is responsible for deciding when it should be called.

```cpp
#include <bit>
#include <cstdint>

constexpr std::uint32_t to_big_endian(std::uint32_t value) {
    if constexpr (std::endian::native == std::endian::little) {
        return std::byteswap(value);
    } else if constexpr (std::endian::native == std::endian::big) {
        return value;
    } else {
        static_assert(
            std::endian::native == std::endian::little ||
            std::endian::native == std::endian::big,
            "Mixed-endian platforms require a dedicated conversion"
        );
    }
}
```

This example is designed for platforms with a uniform Little-endian or Big-endian byte order. A Mixed-endian environment requires a dedicated conversion. Also, `std::byteswap` is defined for integer types, so its input-type restrictions must be considered. It is not a general-purpose way to reverse the bytes of any arbitrary type, such as `float` or a `struct`.

### Implementing Manual Conversion in C++20

If a project is limited to C++20 and does not have access to `std::byteswap`, the conversion can be implemented for a specific unsigned type using bitwise operations:

```cpp
#include <cstdint>

constexpr std::uint32_t byte_swap(std::uint32_t value) noexcept {
    return ((value & 0x000000FFu) << 24) |
           ((value & 0x0000FF00u) << 8)  |
           ((value & 0x00FF0000u) >> 8)  |
           ((value & 0xFF000000u) >> 24);
}
```

This function is written specifically for `std::uint32_t` and should not be generalized to other types without review. Its advantages are that the operations are explicit and do not rely on aliasing. In production code, it is generally preferable to use a standard facility available in the language version being targeted or a well-tested project abstraction.

## Applications in Binary Files and Networking

### Binary Files

A file format should explicitly define the byte order of its multi-byte fields. If a file is defined to use Little-endian order, a program running on a Big-endian host must perform the necessary conversion when reading and writing the file.

Writing a variable's memory directly with `write(reinterpret_cast<const char*>(&value), sizeof(value))` is generally insufficient for a portable file format. The type size, endianness, padding, representation, and even versioning contract may differ on another platform.

### Network Protocols

A protocol should specify byte order independently of the host. Many network conventions use Network Byte Order, which means Big-endian in common IP-based protocols. For compatibility, encode and decode data according to the protocol contract; do not assume that all hosts have the same native byte order.

### Shared Memory and IPC

With Shared Memory, data may be shared among processes running on the same system and usually using the same ABI. Even in this situation, the layout contract should be clear. If data is to be exchanged between different application versions, different architectures, or independent systems, an explicit binary format and controlled conversion are more reliable.

## Relationship to Other Standard Features

### Difference from `std::bit_cast`

The C++20 `std::bit_cast` function transfers the bit representation of one object to another object of equal size, subject to the standard's constraints. It is not an endianness conversion function. Its result depends on the Object Representation of the source and destination types and on the current platform.

### Difference from `reinterpret_cast`

The `reinterpret_cast` operator performs a type conversion at the language level and should not be treated as a general solution for serialization or safe access to arbitrary memory. To inspect an object's bytes, accessing them through `char`, `unsigned char`, or `std::byte` is a more appropriate pattern. To transfer a representation between equal-sized types, `std::bit_cast` is the standard C++20 option when its constraints are satisfied.

### Difference from `std::memcpy`

The `std::memcpy` function copies bytes from one memory region to another, but it does not change their order or make a data format portable. It can be useful for transferring representations between suitable objects, but responsibility for endianness and representation validity remains with the programmer.

### Relationship to `std::uintN_t`

Types such as `std::uint32_t`, when supported by the implementation, are unsigned integer types with an exact width. This property is very useful for binary formats because it makes the numeric width explicit. However, an exact width does not determine byte order. A portable format must specify both the width and the byte order.

## Checking Library Support

### Feature-Test Macro

The feature-test macro for `std::endian` is `__cpp_lib_endian`, with the standardized value `201907L`. It can be used to check whether the standard library provides this feature:

```cpp
#include <bit>

#ifndef __cpp_lib_endian
#error "std::endian is not available"
#elif __cpp_lib_endian < 201907L
#error "A sufficiently recent std::endian implementation is required"
#endif
```

This macro identifies library support and is distinct from macros that detect the language version or compiler identity. Using `std::endian` requires both an appropriate language mode and a standard library implementation that supports the feature.

## Common Mistakes and Design Considerations

### Assuming All Systems Are Little-endian

The prevalence of Little-endian systems is not a standard guarantee for all platforms. Portable code should either detect endianness and perform conversions or explicitly declare its platform restrictions.

### Assuming Every Non-Little-endian Case Is Big-endian

This assumption is incorrect in a Mixed-endian environment. Consider all three cases to fully account for the `std::endian` contract.

### Using `std::endian` to Detect a File's Byte Order

A file's byte order is a property of the file format, not of the host system. `std::endian::native` is useful only for comparing the host environment with the file format's contract; it does not discover that contract.

### Reversing an Entire Buffer Without Considering Its Fields

A binary structure may contain one-byte fields, multi-byte fields, strings, and sections with independent byte-order rules. Reversing the entire buffer can corrupt the data. Conversion should be performed according to the definition of each field.

### Storing Structures Directly

Writing a `struct`'s memory directly to a file can be non-portable because of padding, alignment, layout, and the representations of its member types. For a stable format, encode each field separately and define its size and byte order.

### Misusing `byteswap`

Reversing bytes is not always equivalent to a semantic value conversion. Use it only when the data contract specifically requires that operation and the input type is within the function's supported domain.

### Confusing Endianness with Bit Order

For protocols that define a particular bit order within bytes, checking `std::endian` alone is not sufficient. The protocol specification's rules for bit order, framing, and encoding must be followed separately.

## Choosing the Appropriate Approach

| Requirement | Recommended approach |
|---|---|
| Detect the platform's native byte order | `std::endian::native` |
| Select a code path at compile time | `if constexpr` |
| Enforce an architectural restriction | `static_assert` with a clear message |
| Inspect an object's byte representation | Access through `unsigned char` or `std::byte` |
| Transfer a representation between equal-sized types | `std::bit_cast`, subject to its constraints |
| Reverse the bytes of an integer value in C++23 | `std::byteswap` |
| Read and write a portable binary format | Explicit encoding and decoding according to the format contract |
| Network communication | Follow Network Byte Order or the order defined by the protocol |

## Summary

The C++20 `std::endian` type is a small but important Standard Library facility that reports the endianness of scalar types in the execution environment in a standard, compile-time-usable way. Its three values—`little`, `big`, and `native`—model common byte orders as well as special cases such as Mixed-endian environments. Therefore, `native` should not be assumed to mean Little-endian or Big-endian without checking.

The most important design principle is to keep host endianness separate from the data contract. For files, networks, and data exchange, byte order must be defined by the format or protocol, and the program must encode and decode data according to that contract. `std::endian` is a detection facility; `std::byteswap` in C++23 reverses the bytes of integer values; and facilities such as `std::bit_cast` and `std::memcpy` are not substitutes for explicit serialization.

## References

- [C++ Working Draft — `bit.endian`](https://timsong-cpp.github.io/cppwp/std20/bit)
- [cppreference — `std::endian`](https://en.cppreference.com/w/cpp/types/endian)
- [cppreference — `std::byteswap`](https://en.cppreference.com/w/cpp/numeric/byteswap)
- [cppreference — `std::bit_cast`](https://en.cppreference.com/w/cpp/numeric/bit_cast)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>