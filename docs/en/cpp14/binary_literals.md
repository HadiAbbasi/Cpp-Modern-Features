<div align="center">

[🇺🇸 English](./binary_literals.md) | [🇮🇷 فارسی](../../fa/cpp14/binary_literals.md)

</div>

---

# Introduction to Binary Literals in C++14

## Introduction

Binary representation is one of the fundamental concepts in systems programming, bit manipulation, algorithm design, and low-level data processing. Although computers process information at the hardware level as bits, programmers typically write integer values in source code using decimal notation.

For example, the decimal number 10 is represented as `1010` in binary. Before C++14, C++ supported integer literals written in decimal, octal, and hexadecimal notation, but there was no standard way to write an integer literal directly in binary.

C++14 addressed this limitation by introducing binary literals with the `0b` or `0B` prefix. This feature allows binary values to be written directly in source code, making bitwise operations easier to understand.

## Understanding Binary Literals

A literal in C++ is a value written directly in source code. For example, `42`, `0x2A`, and `3.14` are all literals.

Binary literals allow integer values to be expressed using only the binary digits `0` and `1`. The `0b` or `0B` prefix indicates that the digits following it represent a number in base 2.

```
#include <iostream>

int main()
{
    int decimal = 10;
    int binary = 0b1010;
    int hexadecimal = 0xA;
    int octal = 012;

    std::cout << decimal << '\n';
    std::cout << binary << '\n';
    std::cout << hexadecimal << '\n';
    std::cout << octal << '\n';
}
```

The output is:

```
10
10
10
10
```

In this example, all four literals represent the same numeric value, but they use different notations.

The important point is that the notation used to write a literal does not, by itself, determine its data type or the way its value is stored. For example, `0b1010` is an integer literal whose decimal value is 10. It does not represent a string containing the characters `1`, `0`, `1`, and `0`.

## Writing Binary Literals

### Using Binary Prefixes

C++14 supports two prefixes for binary literals:

```
int a = 0b1010;
int b = 0B1010;
```

Both expressions are valid and represent the decimal value 10.

The prefix must appear directly before the digits. Using lowercase or uppercase letters makes no difference to the meaning.

### Valid Digits

Only the digits `0` and `1` are permitted in the binary portion of a literal.

```
int a = 0b0;
int b = 0b1;
int c = 0b101;
int d = 0b11111111;
```

The following examples are invalid:

```
// Invalid binary literals
// int a = 0b102;
// int b = 0b1202;
// int c = 0b2;
```

In these examples, the presence of the digit `2` makes the expression an invalid binary literal.

The binary prefix must also be followed by at least one valid digit. Therefore, `0b` by itself is not a complete integer literal.

### Using Digit Separators

C++14 also introduced digit separators. The apostrophe character `'` can be used to divide the digits of a literal into more readable groups.

```
int flags = 0b1101'0010;
int mask = 0b1111'0000;
int value = 0b1010'1010'0101'0101;
```

These separators are purely for readability and have no effect on the numeric value.

For example, the following expressions represent the same value:

```
int a = 0b11010010;
int b = 0b1101'0010;
```

This feature is particularly useful when writing masks, bit patterns, and long binary values.

A separator must appear between digits. It cannot replace the prefix or be used as an independent part of the number.

## Converting Binary Literals to Numeric Values

### Calculating a Binary Value

Each binary digit contributes a value determined by its position. Starting from the right, the positional weights are \\(2^0\\), \\(2^1\\), \\(2^2\\), and so on.

For example, the value of `0b101101` is calculated as follows:

\\[ 1 \times 2^5 + 0 \times 2^4 + 1 \times 2^3 + 1 \times 2^2 + 0 \times 2^1 + 1 \times 2^0 = 45 \\]

Therefore, the decimal value of this literal is 45.

### The Difference Between Numeric Notation and Data Types

The `0b` prefix specifies only the notation used to write the number. The resulting integer type is determined by the language's integer literal type-selection rules.

For example:

```
auto value = 0b1010;
```

The type of `value` is `int`, because `int` is the first suitable type in the candidate list for an unsuffixed binary integer literal that can represent the value.

Consequently, writing a number in binary does not automatically select `unsigned int` or a special bit-oriented type.

## Type Selection Rules for Binary Literals

One important aspect of professional binary literal usage is understanding the rules that determine the type of an integer literal.

In C++, the type of an unsuffixed integer literal depends on its value and the applicable list of candidate types. For binary literals, the candidate list consists of the following standard integer types:

- `int`
- `unsigned int`
- `long int`
- `unsigned long int`
- `long long int`
- `unsigned long long int`

The compiler examines these types in order and selects the first one capable of representing the literal's value.

For example:

```
auto a = 0b1010;
auto b = 0b11111111;
```

Both values fit comfortably within `int`, so the deduced type of both variables is `int`.

If a literal's value exceeds the range of `int`, the next suitable type in the candidate list is considered. The available ranges depend on the implementation and the supported integer types.

### Using Integer Suffixes

Like other integer literals, binary literals can use suffixes such as `U`, `L`, and `LL`.

```
auto a = 0b1010U;
auto b = 0b1010L;
auto c = 0b1010LL;
auto d = 0b1010ULL;
```

The types of these variables follow the rules associated with the `U`, `L`, `LL`, and `ULL` suffixes.

| Suffix | Meaning                           |
| ------ | --------------------------------- |
| `U`    | Selects an unsigned integer type  |
| `L`    | Uses the `long` suffix            |
| `LL`   | Uses the `long long` suffix       |
| `ULL`  | Uses an unsigned `long long` type |

Suffixes can help select an appropriate type, but they do not guarantee that every possible value can be represented by the selected type.

For example, if the literal's value cannot be represented by the required type, the program may fail to compile.

## Using Binary Literals with Bitwise Operators

One of the most important applications of binary literals is making bitwise operations easier to read. These operators work on the bit representations of integer values and are widely used in low-level programming.

### Using AND to Extract Bits

The `&` operator produces a `1` in a bit position only when both corresponding bits are `1`.

```
#include <iostream>

int main()
{
    unsigned int value = 0b11010110;
    unsigned int mask = 0b00001111;

    unsigned int result = value & mask;

    std::cout << result << '\n';
}
```

The output is `6`, because the mask preserves the four least significant bits and clears all the remaining bits.

```
6
```

Using binary literals in examples like this makes the structure of the mask directly visible.

### Using OR to Set Bits

The `|` operator produces a `1` whenever at least one of the corresponding bits is `1`.

```
unsigned int flags = 0b00000001;
unsigned int readPermission = 0b00000100;

flags |= readPermission;
```

After this operation, the bit corresponding to the desired permission is set without changing any other bits that were already set in `flags`.

### Using XOR to Toggle Bits

The `^` operator produces a `1` when the corresponding bits differ.

```
unsigned int flags = 0b00000101;
unsigned int mask = 0b00000100;

flags ^= mask;
```

In this example, the third bit from the right is toggled. If it was `0`, it becomes `1`; if it was `1`, it becomes `0`.

### Using NOT to Invert Bits

The `~` operator inverts every bit of an integer value.

```
unsigned int value = 0b00001111;
unsigned int result = ~value;
```

An important point is that the result does not necessarily have the same number of bits as the literal used to initialize the variable. The `~` operator operates on the actual integer type, and integer promotions may also apply.

Therefore, you should not assume that inverting `0b00001111` always produces `0b11110000`. That result can be expected only when the intended bit width and data type have been explicitly constrained.

## Defining and Using Bit Masks

A bit mask is a bit pattern used to check, set, clear, or toggle specific bits.

Binary literals make these patterns easier to write and understand.

```
#include <iostream>

int main()
{
    constexpr unsigned int Read  = 0b0001;
    constexpr unsigned int Write = 0b0010;
    constexpr unsigned int Exec  = 0b0100;

    unsigned int permissions = Read | Exec;

    bool canRead = (permissions & Read) != 0;
    bool canWrite = (permissions & Write) != 0;
    bool canExecute = (permissions & Exec) != 0;

    std::cout << canRead << '\n';
    std::cout << canWrite << '\n';
    std::cout << canExecute << '\n';
}
```

The output is:

```
1
0
1
```

In this example, each permission is assigned a separate bit. Combining permissions with `|` allows multiple capabilities to be stored in a single integer.

This approach is useful for:

- Managing permissions and features
- Storing a collection of independent options
- Implementing communication protocols
- Processing binary data
- Managing multiple states within a single integer

For professional designs, ensure that each flag occupies a distinct bit and that the chosen integer type has sufficient capacity for the number of flags.

## Checking and Modifying Specific Bits

Many algorithms need to check or modify a particular bit. Shift operators and binary literals can be used together for this purpose.

### Checking Whether a Bit Is Set

```
bool isBitSet(unsigned int value, unsigned int position)
{
    return (value & (1u << position)) != 0;
}
```

This function checks whether the specified bit in the input value is set to `1`.

The `position` value must be within the valid range for the type used. Otherwise, the shift operation may have undefined behavior.

### Setting a Bit

```
unsigned int setBit(unsigned int value, unsigned int position)
{
    return value | (1u << position);
}
```

The `|` operator sets the specified bit while preserving the other bits.

### Clearing a Bit

```
unsigned int clearBit(unsigned int value, unsigned int position)
{
    return value & ~(1u << position);
}
```

This approach first inverts the mask and then applies AND to the input value, clearing the specified bit.

### Toggling a Bit

```
unsigned int toggleBit(unsigned int value, unsigned int position)
{
    return value ^ (1u << position);
}
```

Using shifts to construct masks is appropriate when the bit position is determined at runtime. In contrast, when the bit pattern is fixed and known in advance, writing a binary literal directly may be more readable.

## Comparing Binary and Hexadecimal Literals

C++ supports hexadecimal integer literals in addition to binary literals. These notations can represent the same numeric values, but they differ in readability and practical use.

| Feature               | Binary Literal                        | Hexadecimal Literal                      |
| --------------------- | ------------------------------------- | ---------------------------------------- |
| Prefix                | `0b` or `0B`                          | `0x` or `0X`                             |
| Digits                | `0` and `1`                           | `0` through `9` and `A` through `F`      |
| Bit readability       | Each bit is directly visible          | Each digit represents four bits          |
| Best suited for       | Small masks and bit-oriented patterns | Large values, addresses, and binary data |
| Representation length | Usually longer                        | Usually shorter                          |

For example, the following literals represent the same value:

```
unsigned int binary = 0b1111111111111111;
unsigned int hexadecimal = 0xFFFF;
```

Both have the value 65535.

For patterns in which specific bits must be immediately visible, binary notation is usually more suitable. For larger values, hexadecimal notation can be more compact and readable.

Ultimately, the choice of notation should depend on the purpose of the code and its readability, rather than merely on which literal is shorter.

## Limitations of Binary Literals

### Data Type Limitations

Binary literals do not introduce an independent data type. Their values must still be stored using standard integer types or other appropriate types.

For example, if the goal is to store an 8-bit value, `std::uint8_t` may be an appropriate choice when the implementation provides it.

```
#include <cstdint>

std::uint8_t value = 0b11111111;
```

However, the existence of `std::uint8_t` is not guaranteed on every implementation. It is defined only when the implementation provides an unsigned integer type with exactly eight bits and no padding bits.

Additionally, when working with bit patterns of a fixed width, conversions and arithmetic operations must be considered in light of integer promotion rules and the range of the destination type.

### String Representation Limitations

Writing `0b1010` does not create a string of binary characters. It creates a numeric value.

If the goal is to produce a textual binary representation of a number, an appropriate algorithm or utility must be used to convert the numeric value into a string. Binary literals are intended for writing constant values in source code; they do not replace runtime conversion operations.

### Literal Length Limitations

A long binary literal is not inherently a syntax error, but its value must be representable by one of the candidate integer types. Otherwise, the literal is invalid.

Therefore, when defining large masks or long bit patterns, consider the appropriate integer type and its representable range.

## Standard Support and Compiler Extensions

Before C++14, some compilers supported binary literals as an extension. However, support for such an extension did not mean that the feature was part of the standard language in earlier versions.

Starting with C++14, the `0b` and `0B` prefixes became part of the C++ standard.

When using this feature in a project, check the language standard selected in the compiler configuration.

For example, with GCC and Clang, the desired standard can be specified as follows:

```
g++ -std=c++14 main.cpp -o main
```

Or:

```
clang++ -std=c++14 main.cpp -o main
```

With MSVC, the appropriate option for selecting the language standard should be used. The exact choice depends on the build tools and compiler version.

Keep in mind that enabling C++14 does not, by itself, guarantee that every feature of the standard is fully implemented without limitations in every older compiler release.

## Common Mistakes When Using Binary Literals

### Confusing Numeric Bases

A common mistake is assuming that `1010` and `0b1010` represent the same value.

```
int decimal = 1010;
int binary = 0b1010;
```

In this example, the first variable has the value 1010, while the second has the value 10.

### Assuming There Is a Dedicated Binary Data Type

The `0b` prefix does not assign a special type to a number. Its type is still determined by the integer literal type-selection rules.

Therefore, if a specific integer width is required, the appropriate data type must be selected explicitly.

### Assuming a Fixed Width for the NOT Operator

The `~` operator inverts every bit of the actual integer type, not just the bits written in the literal.

To restrict the result to a particular width, use an appropriate data type and a mask matching that width.

### Using Shift Operations Incorrectly

Shift operations can cause problems when the shift count is negative or is greater than or equal to the width of the promoted left operand.

For example, `1u << position` is valid only when `position` is within the permitted range for `unsigned int`.

In reusable and general-purpose code, validating the bit position is advisable.

### Using Binary Literals for Runtime Data

Binary literals are suitable for constant values written in source code. If a binary value comes from user input, a file, a network connection, or a runtime calculation, it must be processed using an appropriate method.

Writing a binary literal is not a substitute for string-to-integer conversion algorithms or binary data processing.

## Choosing the Appropriate Approach

The following guidelines can help you choose an appropriate way to represent integer values:

- When the readability of a bit pattern is important, binary literals are a suitable choice.
- When compact notation and readability of large binary values are important, hexadecimal literals are usually preferable.
- When a value is calculated at runtime, use appropriate arithmetic operations, shifts, masks, or conversion algorithms.
- When a specific bit width is required, explicitly consider the data type and its limitations.
- When code must remain compatible with standards older than C++14, standard binary literals cannot be used. An alternative notation is required unless you deliberately depend on a compiler-specific extension.

## Summary

Binary literals, introduced in C++14, provide a simple and readable way to write integer values in binary notation. Using the `0b` and `0B` prefixes, developers can define bit patterns directly in source code, making bit masks and bitwise operations easier to understand.

This feature is particularly useful in systems programming, flag management, binary data processing, and bit-oriented algorithms.

However, a binary literal is a notation for a constant value, not an independent data type or a mechanism for converting strings at runtime. Professional use requires an understanding of integer literal type-selection rules, integer ranges, bitwise operator behavior, and shift-operation limitations.

---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>