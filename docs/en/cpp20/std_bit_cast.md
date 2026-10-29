<div align="center">

[🇺🇸 English](./std_bit_cast.md) | [🇮🇷 فارسی](../../fa/cpp20/std_bit_cast.md)

</div>

---

# A Comprehensive Guide to `std::bit_cast` in C++20

## Introduction

`std::bit_cast` is an important feature introduced in C++20 that allows a value of one type to be created based on the bit representation of an existing object.

This feature is part of the standard library and is defined in the `<bit>` header. It is designed for situations where a programmer needs access to the bit representation of a value rather than performing a conventional numeric conversion.

For example, suppose you have a variable of type `float` and want to examine its underlying bits as a `std::uint32_t` value.

A conventional type conversion using `static_cast` does not accomplish this because `static_cast` converts the numeric value, not its bit representation.

In contrast, `std::bit_cast` uses the bits representing the source object to construct a new value of the destination type.

This distinction is particularly important in low-level programming, binary data analysis, floating-point representation inspection, debugging tools, and algorithms that operate directly on bits.

## Introducing `std::bit_cast`

`std::bit_cast` was introduced in C++20 as part of the standard library and is defined in the `<bit>` header.

Its primary function signature is:

```
template<class To, class From>
constexpr To bit_cast(const From& from) noexcept;
```

In this declaration:

- `To` specifies the destination type.
- `From` specifies the source type.
- `from` is the object whose bit representation is used to construct the destination value.
- The return value is a new object of type `To`.

Two fundamental requirements must be satisfied when using this function:

- The source and destination types must have the same size.
- Both types must satisfy the `TriviallyCopyable` requirement.

In other words:

```
sizeof(To) == sizeof(From)
```

And:

```
std::is_trivially_copyable_v<To>
std::is_trivially_copyable_v<From>
```

If these requirements are not satisfied, the corresponding function call is not valid.

### A First Example

The following example demonstrates how to convert the bit representation of a `float` value into a `std::uint32_t` value:

```
#include <bit>
#include <cstdint>
#include <iostream>

int main() {
    float value = 1.0f;

    std::uint32_t bits =
        std::bit_cast<std::uint32_t>(value);

    std::cout << std::hex << bits << '\n';
}
```

On common implementations that use IEEE 754 representation for `float`, the output is:

```
3f800000
```

This value represents the bit pattern of `1.0f`, not its numeric conversion to an integer.

An important detail is that the C++ standard does not guarantee that every implementation uses IEEE 754 representation for `float`. Therefore, the output of this example depends on implementation-specific characteristics.

## Understanding Object Representation and Value Representation

To use `std::bit_cast` correctly, it is essential to understand two important concepts in the C++ object model.

### Object Representation

The term `Object Representation` refers to the sequence of bits occupied by an object in memory.

For example, if a type occupies four bytes, its object representation consists of 32 bits.

```
#include <cstdint>

int main() {
    std::uint32_t value = 42;
}
```

In this example, assuming `std::uint32_t` is available in the implementation, the object `value` occupies exactly 32 bits.

### Value Representation

The term `Value Representation` refers to the set of bits used to represent the value of an object.

For some data types, not every bit in the object representation necessarily contributes to determining its value.

Some bits may be used as `Padding Bits`.

For example, a data type might occupy 64 bits while not all of those bits participate in its value representation.

This distinction is important when using `std::bit_cast`, because the operation depends on the value representation of the destination type, and the state of padding bits must also be considered.

### Understanding Padding Bits

Padding bits are part of the storage occupied by an object but are not used to represent its value.

Their presence can cause two objects with the same value to have different object representations.

Therefore, comparing the complete byte representations of two objects is not necessarily equivalent to comparing their values.

This distinction matters when using `std::bit_cast`, performing binary comparisons, and designing serialization mechanisms.

## Numeric Conversion vs. Bitwise Conversion

One of the most important aspects of using `std::bit_cast` is understanding how it differs from conventional type conversions.

Consider the following example:

```
#include <bit>
#include <cstdint>
#include <iostream>

int main() {
    float value = 3.0f;

    int numericValue = static_cast<int>(value);

    std::uint32_t bitRepresentation =
        std::bit_cast<std::uint32_t>(value);

    std::cout << numericValue << '\n';

    std::cout << std::hex
              << bitRepresentation << '\n';
}
```

This program performs two different operations.

Operation 1: Numeric conversion

```
int numericValue = static_cast<int>(value);
```

In this case, the numeric value `3.0f` is converted to the integer `3`.

Operation 2: Bit representation conversion

```
std::uint32_t bitRepresentation =
    std::bit_cast<std::uint32_t>(value);
```

In this case, the bits representing the `float` object are used to construct a `std::uint32_t` value.

On common IEEE 754 implementations, the bit representation of `3.0f` is:

```
0x40400000
```

These two operations therefore serve different purposes.

The general selection rule is:

- Use `static_cast` for numeric conversions.
- Use `std::bit_cast` to construct a new value from the bit representation of an object.

## Inspecting the Bit Representation of float

One important application of `std::bit_cast` is inspecting the internal representation of floating-point numbers.

Many modern systems use the IEEE 754 format for the `float` type.

In this format, a floating-point representation consists of three main components:

- `Sign Bit`: Determines the sign of the number.
- `Exponent`: Represents the exponent.
- `Fraction`: Represents the fractional part or significand.

For ordinary single-precision IEEE 754 floating-point numbers, these components are organized as follows:

| Component | Number of Bits |
| --------- | -------------- |
| Sign      | 1              |
| Exponent  | 8              |
| Fraction  | 23             |
| Total     | 32             |

Using `std::bit_cast`, we can inspect the bit representation of these values.

```
#include <bit>
#include <cstdint>
#include <iostream>
#include <limits>

int main() {
    static_assert(sizeof(float) == sizeof(std::uint32_t));

    float value = 1.5f;

    std::uint32_t bits =
        std::bit_cast<std::uint32_t>(value);

    std::uint32_t sign = (bits >> 31) & 0x1;

    std::uint32_t exponent = (bits >> 23) & 0xFF;

    std::uint32_t fraction = bits & 0x7FFFFF;

    std::cout << "Sign: " << sign << '\n';

    std::cout << "Exponent: " << exponent << '\n';

    std::cout << "Fraction: " << fraction << '\n';
}
```

This example assumes that the `float` representation is compatible with the expected 32-bit IEEE 754 layout.

An important detail is that `sizeof(float) == sizeof(std::uint32_t)` only checks that the two types have the same size. It does not guarantee that their representations follow compatible formats.

One important floating-point characteristic can be checked using:

```
std::numeric_limits<float>::is_iec559
```

However, this check is not a substitute for documenting and validating all platform-specific assumptions.

## Inspecting NaN and Infinity

`NaN` and `Infinity` values are particularly important in floating-point calculations.

In IEEE 754, their bit representations differ from those of ordinary numeric values.

Using `std::bit_cast`, we can inspect the representations of these special values.

```
#include <bit>
#include <cstdint>
#include <iostream>
#include <limits>

int main() {
    float infinity = std::numeric_limits<float>::infinity();

    float nan = std::numeric_limits<float>::quiet_NaN();

    auto infinityBits =
        std::bit_cast<std::uint32_t>(infinity);

    auto nanBits =
        std::bit_cast<std::uint32_t>(nan);

    std::cout << std::hex << infinityBits << '\n';

    std::cout << nanBits << '\n';
}
```

In this example, the resulting bit patterns depend on the internal representation of these values.

For detailed NaN analysis, several concepts are particularly relevant:

- `Quiet NaN`
- `Signaling NaN`
- `NaN Payload`
- Sign Bit
- Exponent Field
- Fraction Field

Depending on the implementation and the operations performed, the exact representation of NaN can vary.

Therefore, it should not be assumed that every implementation uses the same bit pattern for NaN.

## Converting Back and Restoring the Original Value

Another application of `std::bit_cast` is converting a value's bit representation into another type and then reconstructing the original type.

For example:

```
#include <bit>
#include <cstdint>

constexpr float original = 1.5f;

constexpr std::uint32_t bits =
    std::bit_cast<std::uint32_t>(original);

constexpr float restored =
    std::bit_cast<float>(bits);

static_assert(restored == original);
```

In this example, the bit representation of `original` is first transferred into an integer value.

The same bits are then used to construct a new `float` value.

For simple types with valid representations, this approach allows the original value to be restored.

However, this technique should not be generalized to every data type without considering padding, indeterminate bits, and the representation rules of the involved types.

It is also important to understand that numeric comparison using `==` does not necessarily prove bitwise equality. For example, `+0.0f` and `-0.0f` compare equal numerically but have different bit representations in IEEE 754.

## Using std::bit_cast in constexpr

One important feature of `std::bit_cast` is its support for `constexpr`.

This allows bitwise conversions to be performed at compile time when the applicable requirements are satisfied.

For example:

```
#include <bit>
#include <cstdint>

constexpr std::uint32_t bits = 0x3F800000u;

constexpr float value =
    std::bit_cast<float>(bits);
```

On a suitable implementation, this operation can be evaluated at compile time.

However, the fact that the function is declared `constexpr` does not mean that every invocation is permitted during constant evaluation.

### Constant Evaluation Restrictions

Constant evaluation imposes specific restrictions on the source and destination types.

Some important restrictions include:

- `union` types are not permitted for this purpose.
- Pointer and pointer-to-member types are restricted.
- `volatile`-qualified types are not permitted.
- The presence of reference-type non-static data members can prevent constant evaluation.
- The rules concerning indeterminate bits and valid destination representations still apply.

Consequently, an invocation that is valid at runtime might not be valid in a `constexpr` context.

For portable code, compile-time evaluation support should be verified for the target implementation.

## Understanding TriviallyCopyable

One of the main requirements of `std::bit_cast` is that both the source and destination types must be `TriviallyCopyable`.

This concept is important in the C++ object model.

A `TriviallyCopyable` type is a type whose copy and move operations follow specific rules that make its object representation suitable for certain low-level operations.

The following type trait can be used to check this property:

```
#include <type_traits>
#include <iostream>

struct Point {
    float x;
    float y;
};

struct NonTrivial {
    NonTrivial() {}

    int value;
};

int main() {
    std::cout
        << std::is_trivially_copyable_v<Point>
        << '\n';

    std::cout
        << std::is_trivially_copyable_v<NonTrivial>
        << '\n';
}
```

In this example, `Point` is a simple trivially copyable type.

However, `NonTrivial` is not trivially copyable in this example because its user-provided constructor makes the type non-trivially default constructible.

An important detail is that being `TriviallyCopyable` does not mean that every arbitrary interpretation of the type's bits is safe.

This property is only one of the prerequisites for using `std::bit_cast`.

## Using std::bit_cast with Custom Types

This function is not limited to numeric types.

It can also be used with custom types, provided that the size and `TriviallyCopyable` requirements are satisfied.

For example:

```
#include <bit>
#include <cstdint>
#include <iostream>

struct RGB {
    std::uint8_t red;
    std::uint8_t green;
    std::uint8_t blue;
    std::uint8_t alpha;
};

int main() {
    std::uint32_t packed = 0xFF112233;

    static_assert(sizeof(RGB) == sizeof(std::uint32_t));

    RGB color = std::bit_cast<RGB>(packed);

    std::cout << static_cast<int>(color.red) << '\n';
    std::cout << static_cast<int>(color.green) << '\n';
    std::cout << static_cast<int>(color.blue) << '\n';
    std::cout << static_cast<int>(color.alpha) << '\n';
}
```

This example can satisfy the size and type requirements, but the resulting byte order depends on the system's endianness.

Consequently, we cannot assume that the value `0xFF112233` will be mapped to the `RGB` members in the same order on every architecture.

For portable code, it is better to extract the fields explicitly:

```
#include <cstdint>

struct RGB {
    std::uint8_t red;
    std::uint8_t green;
    std::uint8_t blue;
    std::uint8_t alpha;
};

RGB unpackColor(std::uint32_t packed) {
    return {
        static_cast<std::uint8_t>((packed >> 24) & 0xFF),
        static_cast<std::uint8_t>((packed >> 16) & 0xFF),
        static_cast<std::uint8_t>((packed >> 8) & 0xFF),
        static_cast<std::uint8_t>(packed & 0xFF)
    };
}
```

This approach explicitly defines the desired field order independently of the byte order used by the system's memory representation.

## Differences Between std::bit_cast and reinterpret_cast

One important motivation behind introducing `std::bit_cast` was to provide a standard mechanism for transferring bit representations between different types.

Before C++20, programmers sometimes used `reinterpret_cast` to reinterpret memory.

For example:

```
float value = 1.0f;

auto bits = *reinterpret_cast<std::uint32_t*>(&value);
```

This approach attempts to access a `float` object through an lvalue of type `std::uint32_t`.

Such access can conflict with the `Strict Aliasing` rules and the rules governing permitted object access, potentially resulting in undefined behavior.

In contrast:

```
std::uint32_t bits =
    std::bit_cast<std::uint32_t>(value);
```

This creates a new value of the destination type without requiring access to the source object through an unrelated type.

However, these two mechanisms do not serve identical purposes.

`reinterpret_cast` is useful for certain pointer conversions and low-level operations involving memory addresses, whereas `std::bit_cast` is designed to create a new value from an object's bit representation.

Therefore, `std::bit_cast` should not be considered a replacement for every use of `reinterpret_cast`.

## Differences Between std::bit_cast and std::memcpy

Before `std::bit_cast` was introduced, one standard approach for copying object representations between equally sized objects was to use `std::memcpy`.

For example:

```
#include <cstdint>
#include <cstring>

float convertToFloat(std::uint32_t bits) {
    float result;

    static_assert(sizeof(result) == sizeof(bits));

    std::memcpy(&result, &bits, sizeof(result));

    return result;
}
```

Unlike accessing an object through an unrelated pointer type, this approach uses the permitted object-representation copying behavior of `std::memcpy`.

With the introduction of C++20, `std::bit_cast` provided a more direct and readable way to express this particular operation.

A comparison of the two approaches:

| Feature                                 | `std::bit_cast`                 | `std::memcpy`                        |
| --------------------------------------- | ------------------------------- | ------------------------------------ |
| Standard version                        | C++20                           | C++98                                |
| Result                                  | A value of the destination type | A memory copy                        |
| Equal-size requirement                  | Yes                             | Yes, for this use case               |
| `TriviallyCopyable` requirement         | Yes, for both types             | Different object-copying rules apply |
| `constexpr` support                     | Under applicable conditions     | No, in C++20                         |
| Readability for bitwise conversion      | High                            | Lower                                |
| Availability in older language versions | No                              | Yes                                  |

For new C++20 code, `std::bit_cast` is generally the more appropriate choice. However, `std::memcpy` remains useful for general binary data copying and compatibility with older language versions.

## Relationship with std::endian

One important topic when working with bit representations is `Endianness`.

Byte order determines how the bytes of a multi-byte value are arranged in memory.

The two common byte orders are:

- `Little Endian`
- `Big Endian`

C++20 introduced `std::endian` in the `<bit>` header.

It can be used to inspect the native byte order of a system.

```
#include <bit>
#include <iostream>

int main() {
    if constexpr (
        std::endian::native == std::endian::little
    ) {
        std::cout << "Little Endian\n";
    }
    else if constexpr (
        std::endian::native == std::endian::big
    ) {
        std::cout << "Big Endian\n";
    }
    else {
        std::cout << "Mixed Endian\n";
    }
}
```

An important detail is that `std::endian::native` is not necessarily equal to either `little` or `big` on every implementation.

Furthermore, `std::bit_cast` does not change byte order or perform endianness conversion.

Therefore, when receiving data from a network or a file, its byte order must be handled separately.

## Relationship with std::byteswap

`std::byteswap` was introduced in C++23 in the `<bit>` header.

Its purpose is to reverse the byte order of an integer value.

For example:

```
#include <bit>
#include <cstdint>
#include <iostream>

int main() {
    std::uint32_t value = 0x12345678;

    auto reversed = std::byteswap(value);

    std::cout << std::hex << reversed << '\n';
}
```

Output:

```
78563412
```

This function is useful for handling byte order in network protocols, file formats, and binary data.

Note that `std::byteswap` is intended for integer types, and the type must not have padding bits.

This function complements `std::bit_cast` rather than replacing it.

## Limitations and Undefined Behavior

Using `std::bit_cast` does not mean that any arbitrary sequence of bits can be safely converted into any data type.

Several important limitations must be considered.

### Valid Destination Representations

If the resulting bit representation cannot represent a valid value of the destination type, the program has undefined behavior.

For example, we should not assume that every arbitrary 32-bit pattern necessarily represents a valid `float` value.

### Padding Bits

If the destination type contains padding bits, their values are not necessarily specified.

Therefore, the result must not be used in a way that depends on specific values for those padding bits.

### Indeterminate Bits

The presence of indeterminate bits in the source object's representation can affect the result.

The CWG 2482 defect report, associated with proposal P1272R4, clarified the rules concerning indeterminate bits in `std::bit_cast` for C++20.

Consequently, uninitialized memory should not be treated as a reliable source for constructing a destination value.

### Different Type Sizes

If the sizes of the two types differ, `std::bit_cast` cannot be used.

For example:

```
float value = 1.0f;

// Invalid: source and destination sizes may differ.
auto result = std::bit_cast<std::uint64_t>(value);
```

This code is invalid on typical implementations because the source and destination types do not have equal sizes.

### Pointer Restrictions

`std::bit_cast` is not a general-purpose mechanism for converting pointers.

Even if two pointer types have the same size, this does not automatically make their bitwise conversion valid.

Pointer rules, representation validity, object lifetime, and constant-evaluation restrictions must be considered separately.

## Using std::bit_cast for Serialization and Deserialization

One common mistake is using `std::bit_cast` to convert external data directly into complex structures.

For example, a programmer might want to convert data received from a network directly into a struct.

This is generally not an appropriate approach because several issues must be considered:

- `Endianness`
- `Padding`
- `Alignment`
- Type sizes
- ABI differences
- Input data validity
- Differences in type representations across architectures

`std::bit_cast` does not automatically solve any of these problems.

For example, if a network protocol contains a 32-bit Big Endian integer, the four bytes must first be received, their order must be handled appropriately, and the numeric value must then be constructed.

In such situations, an explicit and carefully controlled parser is generally a more portable choice.

## Important Design Considerations for APIs

When using `std::bit_cast` in low-level libraries and API design, several considerations are important.

- Explicit assumptions: If the code depends on IEEE 754 or a specific endianness, document these assumptions.
- Size verification: Use `static_assert` to verify that the source and destination types have equal sizes.
- Type verification: When necessary, use `std::is_trivially_copyable_v` to validate the involved types.
- Avoiding ABI dependencies: Do not treat the internal representations of structs as stable storage or data-transfer formats.
- Input validation: Before constructing values from external binary data, validate that the input is complete and valid.
- Avoiding hidden assumptions: Equal sizes do not necessarily imply that two types have equivalent meanings or representations.

## Checking Compiler and Library Support

Using `std::bit_cast` requires C++20 support in both the compiler and the standard library.

Simply enabling the C++20 language mode does not guarantee that every C++20 standard library feature is available.

The library's support can be checked using a feature-test macro:

```
#include <bit>

#if defined(__cpp_lib_bit_cast) && \
    __cpp_lib_bit_cast >= 201806L

// std::bit_cast is available.

#else

// std::bit_cast is not available.

#endif
```

The standard feature-test macro value for this feature is `201806L`.

This approach is useful for libraries that need to support multiple compiler and standard library versions.

## Comparison of Related Tools

The following table helps identify the appropriate tool for different operations.

| Requirement                                              | Recommended Tool                            | Standard                 |
| -------------------------------------------------------- | ------------------------------------------- | ------------------------ |
| Numeric conversion between types                         | `static_cast`                               | C++98                    |
| Constructing a value from an object's bit representation | `std::bit_cast`                             | C++20                    |
| Copying an object representation                         | `std::memcpy`                               | C++98                    |
| Inspecting byte order                                    | `std::endian`                               | C++20                    |
| Reversing byte order                                     | `std::byteswap`                             | C++23                    |
| Low-level memory address manipulation                    | Pointer-related tools and appropriate casts | Depends on the operation |

The choice of tool should be based on the actual purpose of the operation, not merely on superficial similarities between the available mechanisms.

## Common Programming Mistakes

### Confusing Bitwise and Numeric Conversion

Using `static_cast` instead of `std::bit_cast` converts the numeric value rather than its bit representation.

### Assuming Identical float Representations Across Systems

The C++ standard does not guarantee that every implementation uses IEEE 754 representation.

### Using reinterpret_cast to Access an Object's Representation

Accessing an object through an unrelated type may violate the `Strict Aliasing` rules.

### Using It for Direct Network Data Conversion

Converting structs directly into network data without handling endianness and padding can cause serious problems.

### Ignoring Indeterminate Bits

Uninitialized memory or the presence of indeterminate bits can affect the validity of the result.

### Assuming It Completely Replaces memcpy

`std::bit_cast` is designed to construct a value from a bit representation, whereas `std::memcpy` has broader applications in data copying and memory management.

## Summary

`std::bit_cast`, introduced in C++20, provides a standard mechanism for constructing a value of a destination type from the bit representation of a source object.

It is particularly useful in the following applications:

- Inspecting the internal representation of floating-point numbers
- Analyzing IEEE 754 bit patterns
- Developing debugging tools
- Building low-level libraries
- Implementing algorithms that depend on bit representations
- Converting the representations of trivially copyable objects

However, using it correctly requires a thorough understanding of concepts such as `Object Representation`, `Value Representation`, `TriviallyCopyable`, `Padding Bits`, `Endianness`, and the rules governing valid type representations.

Ultimately, `std::bit_cast` should be understood as a specialized tool for working with object representations, not as a general-purpose solution for type conversion, memory management, or external data processing.

## References and Further Reading

- [cppreference: std::bit_cast](https://en.cppreference.com/w/cpp/numeric/bit_cast?utm_source=chatgpt.com)
- [cppreference: Bit Manipulation](https://en.cppreference.com/w/cpp/utility/bit?utm_source=chatgpt.com)
- [cppreference: std::endian](https://en.cppreference.com/w/cpp/types/endian?utm_source=chatgpt.com)
- [cppreference: std::byteswap](https://en.cppreference.com/w/cpp/numeric/byteswap?utm_source=chatgpt.com)
- [P0476R2: Bit-casting object representations](https://wg21.link/P0476R2?utm_source=chatgpt.com)
- [P1272R4: Make std::bit_cast constexpr](https://wg21.link/P1272R4?utm_source=chatgpt.com)
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>