<div align="center">

[🇺🇸 English](./digit_separators.md) | [🇮🇷 فارسی](../../fa/cpp14/digit_separators.md)

</div>

---

# Digit Separators in C++14

## Introduction

Reading and inspecting large numeric values in C++ code can be difficult. For example, identifying the number of zeros in `1000000000` or examining the different parts of a long hexadecimal value is not always straightforward, especially in code involving memory, time, file sizes, and financial calculations.

C++14 introduced the Digit Separators feature, which allows the apostrophe character (`'`) to be placed between digits in a numeric literal. This feature improves readability without changing the numeric value or type.

For example, the following two literals represent the same value:

```
int population = 1000000;
int readablePopulation = 1'000'000;
```

In this example, both variables have a value of one million. The apostrophe serves only to improve readability and is not part of the stored numeric value.

An important point is that Digit Separators are a language feature introduced in C++14. They do not depend on the standard library, and no function, class, or special header is required to use them.

## Understanding Digit Separators in C++14

The Digit Separators feature allows programmers to place apostrophes within different parts of a numeric literal to make its structure easier to read.

For example, the following code demonstrates several ways to represent the same value:

```
#include <iostream>

int main() {
    int a = 1000000;
    int b = 1'000'000;
    int c = 10'00'00;

    std::cout << a << '\n';
    std::cout << b << '\n';
    std::cout << c << '\n';
}
```

The program produces the following output:

```
1000000
1000000
1000000
```

In this example, all three literals have the same value because the placement of separators does not affect how the number is interpreted.

Syntactically, a separator must appear between two valid digits within the same part of a literal. Therefore, this feature is not merely a visual formatting convention; it is part of the language's rules for writing numeric literals.

## Why Digit Separators Were Introduced

This feature provides several practical benefits:

- Improved readability: Long numbers can be divided into smaller groups, making them easier to inspect.
- Fewer human errors: Counting zeros and identifying the magnitude of numbers become easier.
- Clearer representation of technical values: Binary constants, hexadecimal values, memory sizes, and protocol-related values become easier to read.
- Better code documentation: The grouping of digits can reflect the readability conventions used throughout a project.

An important point is that separators do not automatically detect logical errors. For example, if the intended number is `1'000'000` but a programmer writes `10'000'000`, the code remains valid; only the value is incorrect.

## Using Separators in Numeric Literals

### Decimal Integer Literals

The most common application of this feature is improving the readability of decimal integers.

```
int population = 1'250'000;
long long nationalDebt = 12'345'678'901LL;
unsigned int timeoutMilliseconds = 30'000;
```

In these examples, the separators make the numbers easier to read without affecting their values or types.

For integers, grouping digits in sets of three is a common convention. However, the language standard does not require this pattern. The choice of group size depends on the numeric representation and the readability conventions of the project.

### Hexadecimal Literals

In hexadecimal notation, separators are particularly useful for long constants, masks, identifiers, and hardware register values.

```
#include <cstdint>

std::uint32_t color = 0xFF'80'40'20;
std::uint64_t identifier = 0x1234'5678'9ABC'DEF0ULL;
std::uint32_t registerMask = 0xFFFF'0000;
```

In the first example, the hexadecimal value is divided into two-digit groups, which can correspond to the byte representation of a color.

In the second example, grouping the digits in sets of four makes the identifier easier to read. In the third example, the separator clarifies the boundaries between different parts of the mask.

An important point is that the apostrophe is not part of the hexadecimal value and is not stored as part of the resulting number.

### Binary Literals

In C++14, separators can also be used in binary literals. Binary literals were introduced in C++14 and use the `0b` or `0B` prefix.

```
#include <cstdint>

std::uint8_t flags = 0b1010'0101;
std::uint16_t permissions = 0b1111'0000'1010'0101;
std::uint32_t bitMask = 0b1111'1111'0000'0000;
```

In these examples, grouping the bits makes binary patterns easier to inspect.

For instance, bits can be grouped according to nibble size, byte size, or the structure of a field in a protocol. However, this grouping is purely for readability and does not introduce any new bit-level meaning.

### Octal Literals

Separators can also be used in octal literals. Octal notation begins with a leading zero and permits digits from `0` through `7`.

```
int octalValue = 0'123'456;
```

In this example, the literal remains octal because its leading zero is preserved. Its value in decimal is `42798`.

An important point is that separators must not cause confusion about the numeric base. For example, `0'123` is an octal number, not a decimal number that has merely been grouped using apostrophes.

## Using Separators in Floating-Point Literals

Digit Separators are not limited to integers. They can also be used in floating-point literals to improve the readability of the integer part, fractional part, and exponent digits.

### Integer and Fractional Parts

```
double piApproximation = 3.141'592'653'589;
double measurement = 12'345.678'901;
double scientificValue = 1'234.5'678;
```

In these examples, separators make the different parts of the literals easier to read without changing their numeric values.

An important point is that a separator must appear between two digits. Therefore, it cannot be placed directly next to the decimal point.

### Exponents in Scientific Notation

In scientific notation, a separator can also appear between the digits of the exponent.

```
double smallValue = 1.0e1'0;
double largeValue = 2.5e+1'2;
```

In the first example, the exponent is `10`, so the literal represents \\(1.0 \times 10^{10}\\).

In the second example, the exponent is `12`, so the literal represents \\(2.5 \times 10^{12}\\).

An important point is that the characters `e`, `E`, `+`, and `-` are not digits. Therefore, a separator must appear within the numeric part of the exponent, between two digits.

## Syntax Rules and Valid Separator Positions

Digit Separators follow specific rules. Understanding these rules is essential for avoiding compilation errors.

### The Rule of Placement Between Two Digits

The fundamental rule is that an apostrophe must appear between two valid digits within a numeric literal.

```
int validA = 1'000;
int validB = 12'34'56;
int validC = 0xAB'CD;
int validD = 0b1010'1100;
double validE = 1'234.5'678;
double validF = 1.0e1'2;
```

All the examples above are syntactically valid.

In contrast, the following examples are invalid:

```
// Invalid examples
// int a = '1000;
// int b = 1000';
// int c = 1''000;
// int d = 0x'FF;
// double e = 1.'25;
// double f = 1.25'e3;
```

These examples are written as comments so that copying the entire example into a source file does not make the program fail to compile. If uncommented, each line would violate the syntax rules for numeric literals.

The reasons these expressions are invalid are, respectively: placing a separator before the first digit, placing it after the last digit, using two consecutive separators, placing it between a base prefix and the first digit, placing it next to the decimal point, and placing it between the fractional part and the exponent marker.

### Separators Versus Whitespace

An apostrophe is not equivalent to a space or other whitespace character. Whitespace cannot replace a separator within a numeric literal.

```
int validValue = 1'000;

// This is not one numeric literal:
// int invalidValue = 1 000;
```

In this example, the second line does not form a single numeric literal and is syntactically invalid.

An important point is that separators are part of the literal's syntax. Spaces, newlines, and other arbitrary characters cannot be used to achieve the same effect.

## Effects on Type and Value

One of the most important properties of Digit Separators is that they do not change a literal's type, value, or the rules used to determine its numeric type.

### The Difference Between a Literal and Its Runtime Value

In the following code, both variables have the same type and value:

```
#include <type_traits>

int a = 1000000;
int b = 1'000'000;

static_assert(std::is_same_v<decltype(a), decltype(b)>);
static_assert(a == b);
```

This example uses `std::is_same_v`, which is available starting in C++17. Digit Separators themselves are supported from C++14 onward, so this particular example requires C++17 or a later standard.

For use specifically with C++14, the values can be compared without relying on newer standard library features:

```
int a = 1000000;
int b = 1'000'000;

if (a == b) {
    // Both values are equal.
}
```

In both cases, the separator has no role in determining the variable's type or final value.

### Effects on Literal Type Selection

In C++, the type of a numeric literal is determined by rules involving its base, suffix, value, and the applicable list of candidate types. Separators do not change these rules.

```
auto a = 1'000;
auto b = 1'000LL;
auto c = 0xFF'FFU;
auto d = 1'000.0;
```

In these examples, the suffixes `LL` and `U`, as well as the floating-point notation, continue to influence the type of each literal. The apostrophe only separates digits.

An important point is that separators do not increase the range of a numeric type. For example, if a literal without separators falls outside the range of the required type, adding apostrophes does not make it valid or representable.

## Practical Applications in C++ Projects

### Improving the Readability of Memory Sizes

In code involving buffer sizes, memory capacity, or element counts, separators can make the magnitude of values easier to recognize.

```
#include <cstddef>

constexpr std::size_t bufferSize = 1'048'576;
constexpr std::size_t elementCount = 250'000;
```

In this example, the values are not grouped exclusively into sets of three. Instead, the grouping is chosen to make the numeric structure and its relationship to common memory sizes easier to understand.

An important point is that `std::size_t` is an appropriate type for many memory-related sizes and counts, but type selection must still account for the expected value range and the requirements of the application.

### Improving the Readability of Masks and Hardware Values

In low-level programming, binary and hexadecimal values often have a well-defined structure. Separators help make that structure visible.

```
#include <cstdint>

constexpr std::uint32_t readPermission = 0x0000'0001;
constexpr std::uint32_t writePermission = 0x0000'0002;
constexpr std::uint32_t executePermission = 0x0000'0004;

constexpr std::uint32_t permissionMask =
    readPermission | writePermission | executePermission;
```

In this example, four-digit grouping makes the positions of the least significant bits and the higher-order portions clearer.

An important point is that separators do not replace precise definitions of bit-level conventions. In real systems, type sizes, byte order, bit mapping, and hardware requirements must be considered separately.

### Improving the Readability of Time Constants

In code involving time measured in milliseconds or retry counts, grouping numbers can make their values easier to understand.

```
constexpr int timeoutMilliseconds = 30'000;
constexpr int retryDelayMilliseconds = 1'500;
constexpr int maximumRetries = 10'000;
```

In this example, the values are clearly recognizable as thirty thousand, one thousand five hundred, and ten thousand.

An important point is that separators do not specify units of measurement. To avoid ambiguity, variable names or appropriate types should communicate the units clearly.

## Limitations and Common Mistakes

### Standard Support and Compilation Modes

Digit Separators have been part of the C++ language since C++14. Therefore, code that uses this feature must be compiled in an appropriate language mode, and the compiler must support the relevant standard.

For example, GCC and Clang can use the following option:

```
g++ -std=c++14 main.cpp -o main
```

Similarly, with Clang, you can write:

```
clang++ -std=c++14 main.cpp -o main
```

An important point is that actual support depends on the compiler version and its configuration. If code is compiled in a language mode older than C++14, the separators may be rejected as invalid syntax.

### Inconsistent Grouping

Although the standard does not restrict separator placement to a particular readability pattern, inconsistent grouping can reduce readability.

```
int clearValue = 123'456'789;
int confusingValue = 12'3456'78'9;
```

Both literals can be syntactically valid, but the second example presents the number using an inconsistent grouping pattern.

An important point is that grouping should follow the conventions of the project. Groups of three digits are generally suitable for decimal numbers, whereas grouping by byte, nibble, or field structure may be more useful for hexadecimal or binary values.

### Confusing Separators with Quotation Marks

The apostrophe character has other uses in C++, including in character literals. However, its role in a numeric literal differs from its role in a character literal.

```
char apostrophe = '\'';
int number = 1'000;
```

In this example, the first line stores an apostrophe character, while the second defines an integer literal with a digit separator.

An important point is that escaping rules for character literals should not be confused with the rules for Digit Separators.

### Formatting and Output Limitations

Digit Separators are part of the numeric literal's source representation. They are not automatically reproduced when the number is displayed in program output.

```
#include <iostream>

int main() {
    int value = 1'000'000;
    std::cout << value << '\n';
}
```

The program produces the following output:

```
1000000
```

If the goal is to display a number with separators in the output, the text formatting must be implemented separately or an appropriate formatting tool must be used. Digit Separators do not provide this functionality.

## Comparison with Older Approaches

Before C++14, programmers generally wrote numbers without separators or improved readability through appropriate comments and naming conventions.

```
constexpr long long oldStyleValue = 1234567890LL;
constexpr long long modernStyleValue = 1'234'567'890LL;
```

In this example, both approaches are valid, but the second makes the numeric structure more apparent.

An important point is that Digit Separators do not replace meaningful names, well-defined constants, or technical documentation. The feature improves the readability of literals only; appropriate names and types are still necessary to represent the meaning of a value.

## Summary

The Digit Separators feature introduced in C++14 provides a simple and effective way to improve the readability of numeric literals. By using apostrophes, programmers can write integer literals, hexadecimal literals, binary literals, octal literals, and floating-point values in a more readable form.

The most important points are:

- Separators are part of the language syntax and do not require a special header or library.
- An apostrophe must appear between two valid digits.
- Separators do not change a literal's value, type, or numeric base.
- Digit grouping should follow the project's readability conventions.
- Separators in source code are not automatically displayed in program output.
- To use this feature, the compiler and language mode must support C++14 or a later standard.

Ultimately, Digit Separators are most useful when code contains long numbers, bit masks, hardware constants, and technical values. Following a consistent grouping convention improves the readability and maintainability of professional C++ projects.

---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>