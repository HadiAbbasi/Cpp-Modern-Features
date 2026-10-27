<div align="center">

[🇺🇸 English](./std_format.md) | [🇮🇷 فارسی](../../fa/cpp20/std_format.md)

</div>

---

# A Comprehensive Guide to `std::format` in C++20

## Introduction to `std::format`

The `std::format` feature was added to the standard library in C++20. It provides a modern, type-safe, and extensible way to convert values into text strings. It was designed to simplify string construction, improve code readability, and reduce common formatting errors.

Before this feature was introduced, developers typically used approaches such as `std::printf`, the insertion operator `operator<<` with input/output streams, or third-party libraries such as `{fmt}`. Each approach has advantages and limitations. For example, `std::printf` relies on format strings and manual matching of argument types, while `std::ostringstream` can require lengthy and less readable code when constructing complex strings.

`std::format` aims to combine the benefits of format-string-based formatting with C++'s type system.

```
#include <format>
#include <iostream>
#include <string>

int main() {
    std::string name = "Alice";
    int age = 30;

    std::string message = std::format(
        "Name: {}, Age: {}",
        name,
        age
    );

    std::cout << message << '\n';
}
```

The program produces the following output:

```
Name: Alice, Age: 30
```

In this example, `{}` marks a position where an argument will be inserted. The `std::format` function converts each value to text according to its type and returns the resulting string as a `std::string`.

## Design Goals and Benefits

`std::format` is more than a shorter way to concatenate strings. It provides a standardized mechanism for controlling how values are represented.

Its main benefits include:

- Type safety: Formatting is compatible with the actual argument types, and many type-mismatch errors can be detected at compile time.
- Improved readability: The text template and the values used to populate it appear together in a single expression.
- Precise output control: You can specify width, precision, numeric base, alignment, sign, and other formatting properties.
- Extensibility: Custom types can define their own formatting behavior.
- Reduced dependence on streams: Creating a string does not require constructing a `std::ostringstream`.
- Standardization: Code using this API can be used across implementations that support the relevant standard features, although actual library support may vary.

Despite these benefits, `std::format` does not replace every capability of `iostream`, logging systems, or input/output APIs. The appropriate choice depends on the application's requirements.

## Compiler Support and Requirements

`std::format` is part of the standard library introduced in C++20. Using it therefore requires both a suitable language-standard setting and an implementation of this part of the standard library.

For example, you can use the following commands:

```
g++ -std=c++20 main.cpp -o app
```

```
clang++ -std=c++20 main.cpp -o app
```

```
cl /std:c++20 /EHsc main.cpp
```

These commands select the language standard, but they do not guarantee that the installed standard library implements every part of `std::format`.

Support has varied across versions of GCC, Clang, MSVC, `libstdc++`, and `libc++`. In particular, early implementations supporting C++20 may not provide every component of the library.

To verify actual support, compile a small test program that includes `#include <format>` and calls `std::format`. For larger projects, the compiler and standard library versions should also be specified and tested in CI.

To detect library support at compile time, you can use the standard feature-test macro `__cpp_lib_format`. This macro has different values across standard revisions. The value `201907L` indicates the baseline functionality introduced in C++20, while later values correspond to subsequent additions and improvements.

```
#include <format>

#if defined(__cpp_lib_format) && __cpp_lib_format >= 201907L
#define HAS_STD_FORMAT 1
#else
#define HAS_STD_FORMAT 0
#endif
```

This check identifies baseline support, but it does not guarantee support for every subsequent standard revision. Projects that depend on newer capabilities should check the relevant feature-test macro value and test the specific functionality they require.

## Format String Structure

### Understanding Replacement Fields

A section of a format string that will be replaced with an argument is called a replacement field. Its simplest form is `{}`.

```
#include <format>
#include <iostream>

int main() {
    auto text = std::format(
        "{} scored {} points.",
        "Alice",
        95
    );

    std::cout << text << '\n';
}
```

The program produces the following output:

```
Alice scored 95 points.
```

In this example, the first field is replaced by the first argument and the second field by the second argument. By default, replacement proceeds from left to right, and the argument types do not need to be specified in the format string.

### Understanding Argument IDs

You can explicitly select the argument used by a field with an argument ID. Argument numbering starts at zero.

```
#include <format>
#include <iostream>

int main() {
    std::cout << std::format(
        "{1} is learning {0}.",
        "C++20",
        "Alice"
    ) << '\n';
}
```

The program produces the following output:

```
Alice is learning C++20.
```

This approach is useful when the display order differs from the natural parameter order or when an argument must be used multiple times.

```
#include <format>
#include <iostream>

int main() {
    int value = 42;

    std::cout << std::format(
        "Decimal: {0}, Hex: {0:#x}, Octal: {0:#o}",
        value
    ) << '\n';
}
```

The program produces the following output:

```
Decimal: 42, Hex: 0x2a, Octal: 052
```

A format string must use a consistent argument-indexing approach. Mixing automatic fields such as `{}` with explicitly indexed fields such as `{0}` is not allowed and results in a formatting error.

### Displaying Literal Braces

Because braces have special meaning in format strings, they must be escaped to display literal braces.

```
#include <format>
#include <iostream>

int main() {
    std::cout << std::format(
        "Configuration: {{\"enabled\": {}}}",
        true
    ) << '\n';
}
```

The program produces the following output:

```
Configuration: {"enabled": true}
```

In this structure, `{{` becomes `{`, and `}}` becomes `}`. Invalid unmatched braces can cause a compile-time error or, with runtime format strings, an exception.

## Format Specification Syntax

### Understanding the General Format

To control how a value is displayed, you can add `:` after the argument ID, followed by a `format specification`.

The general field structure is:

```
{arg-id:format-spec}
```

Here, `arg-id` is optional, and `format-spec` follows rules that depend on the argument type. For built-in types and strings, the standard format specification generally contains the following components:

```
[[fill]align][sign]["#"]["0"][width]["." precision][type]
```

This is a simplified representation of the grammar. The exact ordering and validity of each option depend on the data type.

| Component   | Purpose                                                                                 |
| ----------- | --------------------------------------------------------------------------------------- |
| `fill`      | Specifies the character used to fill empty space                                        |
| `align`     | Controls left, right, or center alignment                                               |
| `sign`      | Controls how numeric signs are displayed                                                |
| `#`         | Enables alternate representation for supported formats                                  |
| `0`         | Uses zeros to pad numeric output                                                        |
| `width`     | Specifies the minimum output width                                                      |
| `precision` | Controls display precision or the number of displayed characters, depending on the type |
| `type`      | Selects a representation, such as decimal, hexadecimal, or scientific notation          |

Understanding these components makes it possible to produce consistent output without writing additional conversion and string-alignment logic.

### Controlling Alignment and Width

You can specify a field's minimum width by placing a number in its format specification. By default, the additional space is generally filled with spaces.

```
#include <format>
#include <iostream>

int main() {
    std::cout << std::format("|{:10}|", "cat") << '\n';
    std::cout << std::format("|{:<10}|", "cat") << '\n';
    std::cout << std::format("|{:^10}|", "cat") << '\n';
    std::cout << std::format("|{:>10}|", "cat") << '\n';
}
```

The program produces the following output:

```
|cat       |
|cat       |
|   cat    |
|       cat|
```

The alignment options are `<` for left alignment, `>` for right alignment, and `^` for center alignment. By default, strings are generally left-aligned, while numbers are right-aligned.

Note that `width` specifies the minimum field width, not the maximum. If a value exceeds the specified width, the full value is normally displayed.

### Using Custom Fill Characters

You can specify a fill character before the alignment option to replace empty space with a character of your choice.

```
#include <format>
#include <iostream>

int main() {
    std::cout << std::format("{:*^12}", "status") << '\n';
    std::cout << std::format("{:.>8}", 42) << '\n';
}
```

The program produces the following output:

```
***status***
......42
```

In this example, `*` and `.` are the fill characters. This feature is useful for generating text reports, aligned console tables, and structured terminal output.

### Controlling Numeric Signs

You can control how numeric signs are displayed using `+`, `-`, and a space.

```
#include <format>
#include <iostream>

int main() {
    std::cout << std::format("{:+d}", 42) << '\n';
    std::cout << std::format("{:+d}", -42) << '\n';
    std::cout << std::format("{: d}", 42) << '\n';
    std::cout << std::format("{: d}", -42) << '\n';
}
```

The program produces the following output:

```
+42
-42
 42
-42
```

Here, `+` displays a sign for both positive and negative numbers. The space option reserves a leading space for positive numbers while preserving the minus sign for negative numbers.

### Padding Numbers with Leading Zeros

You can use the `0` option to pad numeric output with zeros.

```
#include <format>
#include <iostream>

int main() {
    std::cout << std::format("{:08d}", 123) << '\n';
    std::cout << std::format("{:+08d}", 123) << '\n';
    std::cout << std::format("{:08d}", -123) << '\n';
}
```

The program produces the following output:

```
00000123
+0000123
-0000123
```

The leading zeros are part of the textual representation and do not change the underlying numeric value or type. This technique is useful for display identifiers and invoice numbers. However, if leading zeros are semantically significant, storing the identifier as a string may be more appropriate.

### Formatting Numbers in Different Bases

`std::format` supports displaying integers in different numeric bases.

```
#include <format>
#include <iostream>

int main() {
    int value = 255;

    std::cout << std::format("Decimal: {:d}", value) << '\n';
    std::cout << std::format("Hex: {:x}", value) << '\n';
    std::cout << std::format("HEX: {:X}", value) << '\n';
    std::cout << std::format("Octal: {:o}", value) << '\n';
    std::cout << std::format("Binary: {:b}", value) << '\n';
    std::cout << std::format("Binary prefix: {:#b}", value) << '\n';
}
```

The program produces the following output:

```
Decimal: 255
Hex: ff
HEX: FF
Octal: 377
Binary: 11111111
Binary prefix: 0b11111111
```

In this example, `d` selects decimal notation, `x` and `X` select hexadecimal notation, `o` selects octal notation, and `b` selects binary notation. The `#` option adds an appropriate prefix for supported formats.

This capability is useful when developing debugging tools, analyzing bitmasks, inspecting register values, and generating technical reports.

### Controlling Floating-Point Precision

For floating-point values, the `precision` option controls the number of displayed digits. Its exact meaning depends on the selected format.

```
#include <format>
#include <iostream>

int main() {
    double value = 3.141592653589793;

    std::cout << std::format("{:.2f}", value) << '\n';
    std::cout << std::format("{:.5f}", value) << '\n';
    std::cout << std::format("{:.4e}", value) << '\n';
    std::cout << std::format("{:.6g}", value) << '\n';
}
```

The program produces the following output:

```
3.14
3.14159
3.1416e+00
3.14159
```

In this example, `f` selects fixed notation, `e` selects scientific notation, and `g` selects general notation. In general notation, precision usually specifies the number of significant digits rather than the number of digits after the decimal point.

An important distinction is that formatting a number with rounding for display is different from rounding the value and storing the result in another variable. `std::format` does not modify the original value; it only produces its textual representation.

### Formatting Very Large and Very Small Numbers

Scientific notation improves readability when reporting values with a large dynamic range.

```
#include <format>
#include <iostream>

int main() {
    double large = 123456789.0;
    double small = 0.0000012345;

    std::cout << std::format("{:e}", large) << '\n';
    std::cout << std::format("{:g}", large) << '\n';
    std::cout << std::format("{:g}", small) << '\n';
}
```

The exact output depends on the selected format and precision. Scientific notation represents a value as a coefficient and a power of ten, while general notation selects between fixed and scientific notation according to the standard's rules.

### Formatting Boolean Values

By default, Boolean values are displayed as the text `true` and `false`. However, you can request numeric formatting when an appropriate integer-style format specification is used.

```
#include <format>
#include <iostream>

int main() {
    bool enabled = true;

    std::cout << std::format("Enabled: {}", enabled) << '\n';
    std::cout << std::format("Numeric: {:d}", enabled) << '\n';
}
```

The program produces the following output:

```
Enabled: true
Numeric: 1
```

This distinction matters in configuration output and logging. Textual representations are generally more readable to humans, while numeric representations may be more suitable for numeric data formats or interfaces.

### Formatting Strings and Using Precision

For strings, you can control the minimum field width and the maximum number of characters displayed.

```
#include <format>
#include <iostream>
#include <string>

int main() {
    std::string text = "modern C++ formatting";

    std::cout << std::format("{:.6}", text) << '\n';
    std::cout << std::format("{:>12.6}", text) << '\n';
}
```

The program produces the following output:

```
modern
      modern
```

For strings, `precision` controls the maximum number of character units displayed, which differs from the minimum field width.

With Unicode text, you should not assume that every visible character occupies exactly one string unit. In particular, a UTF-8 character may occupy multiple bytes, and some combining characters consist of multiple code points. Therefore, limiting precision does not necessarily limit the number of visible characters.

## Dynamic Formatting and Argument-Based Parameters

### Specifying Width and Precision Dynamically

In some applications, the output width or precision comes from configuration, user input, or another calculation. In such cases, you can use argument IDs within the format specification.

```
#include <format>
#include <iostream>

int main() {
    int width = 10;
    int precision = 3;
    double value = 3.14159265;

    std::cout << std::format(
        "{0:{1}.{2}f}",
        value,
        width,
        precision
    ) << '\n';
}
```

The program produces the following output:

```
     3.142
```

In this example, the first argument is the value, the second specifies the width, and the third specifies the precision. This allows the format string to remain fixed while its display parameters change.

Dynamic width and precision values must still satisfy the formatting rules. Negative or excessively large values can cause errors or substantial memory consumption, so their ranges should be validated in sensitive code.

### Understanding Compile-Time and Runtime Format Strings

One important feature of `std::format` is compile-time format-string checking. In a typical call, the format string must be compatible with `std::format_string`. This mechanism allows many format-string errors to be detected before the program executes.

```
#include <format>
#include <string>

int main() {
    int value = 42;

    std::string valid = std::format("Value: {}", value);

    // The following call has an invalid format specifier for an integer.
    // std::string invalid = std::format("{:s}", value);
}
```

In this example, `s` is a string format specifier and is incompatible with an integer argument. In implementations that perform the required compile-time checking, such a call should be diagnosed as an error.

In contrast, some applications do not know the format string at compile time. For example, it may be loaded from a configuration file. In such cases, runtime-formatting interfaces are appropriate.

### Using `std::vformat` for Runtime Format Strings

The `std::vformat` function provides a formatting interface that accepts type-erased arguments rather than a template-based, directly typed argument pack.

```
#include <format>
#include <iostream>
#include <string>

int main() {
    std::string pattern = "User: {}, Score: {:.1f}";
    std::string result = std::vformat(
        pattern,
        std::make_format_args("Alice", 98.5)
    );

    std::cout << result << '\n';
}
```

The program produces the following output:

```
User: Alice, Score: 98.5
```

In this example, `pattern` is an ordinary string available at runtime. The `std::make_format_args` function creates an argument pack for use by the formatting operation.

Note that `std::make_format_args` generally holds references to its arguments rather than owning independent copies. Therefore, an argument pack must not be stored or passed around in a way that allows the original arguments to expire before the pack is used.

### Handling Runtime Formatting Errors

With runtime format strings, errors such as unmatched braces, an incorrect number of arguments, or a format specifier incompatible with an argument type may only become apparent during execution.

```
#include <format>
#include <iostream>
#include <string>

int main() {
    std::string pattern = "Value: {:s}";

    try {
        std::string result = std::vformat(
            pattern,
            std::make_format_args(42)
        );

        std::cout << result << '\n';
    } catch (const std::format_error& error) {
        std::cerr << "Formatting error: " << error.what() << '\n';
    }
}
```

In this example, the `s` specifier is incompatible with the integer argument, so formatting fails.

For code that receives format strings from external sources, formatting errors should be handled at an appropriate application boundary. However, not every possible failure is a `std::format_error`; for example, a memory-allocation failure may produce `std::bad_alloc`.

### Runtime Format Strings in Later Standards

In C++20, the ordinary `std::format` interface uses compile-time checking for typical format strings, while `std::vformat` is suitable for runtime format strings.

C++26 introduces `std::runtime_format`, which allows a runtime string to be passed directly to the ordinary formatting interface. In implementations that support it, this provides a simpler alternative in many cases to calling `std::vformat` directly.

```
#include <format>
#include <iostream>
#include <string>

int main() {
    std::string pattern = "Value: {}";

    std::cout << std::format(
        std::runtime_format(pattern),
        42
    ) << '\n';
}
```

This example requires C++26 and a standard library that supports `std::runtime_format`. It should not be assumed to compile with every C++20 implementation.

## The Main Functions in the `std::format` Family

Formatting is not limited to creating a new string. The standard library provides several related functions, each suited to a different usage pattern.

### Producing a String with `std::format`

The `std::format` function creates and returns a new string.

```
#include <format>
#include <string>

int main() {
    std::string message = std::format(
        "Request {} completed in {} ms",
        17,
        24.5
    );
}
```

This approach is suitable for constructing messages, generating output text, creating display keys, and preparing data for other application layers.

### Writing Directly with `std::format_to`

The `std::format_to` function writes formatted output through an output iterator instead of automatically returning a new string.

```
#include <format>
#include <iterator>
#include <iostream>
#include <string>

int main() {
    std::string output;
    output.reserve(128);

    auto it = std::format_to(
        std::back_inserter(output),
        "ID={:04}, Name={}",
        23,
        "Alice"
    );

    std::cout << output << '\n';
}
```

The program produces the following output:

```
ID=0023, Name=Alice
```

In this example, `std::back_inserter` allows characters to be appended to the end of the string. Calling `reserve` increases the initial capacity and may reduce the number of allocations when the output fits within that capacity.

The return value of `std::format_to` is an iterator positioned after the last written character. This function is useful for writing into buffers or combining multiple text fragments in a shared output destination.

### Limiting Output with `std::format_to_n`

The `std::format_to_n` function is designed for cases where you want to write at most a specified number of characters to the destination.

```
#include <format>
#include <iostream>
#include <iterator>
#include <string>

int main() {
    std::string output;

    auto result = std::format_to_n(
        std::back_inserter(output),
        5,
        "Hello, {}!",
        "world"
    );

    std::cout << output << '\n';
    std::cout << result.size << '\n';
}
```

The program produces the following output:

```
Hello
13
```

In this example, only the first five characters are written, but `result.size` reports the total number of characters that would have been produced without the limit.

This distinction matters when calculating required sizes and managing buffers. However, `std::format_to_n` does not automatically place the string into a preallocated, fixed-size destination; the destination iterator must have the appropriate behavior.

### Calculating Output Size with `std::formatted_size`

The `std::formatted_size` function calculates the number of characters required to produce the formatted output.

```
#include <format>
#include <iostream>

int main() {
    auto size = std::formatted_size(
        "Name={}, ID={:06}",
        "Alice",
        42
    );

    std::cout << size << '\n';
}
```

The program produces the following output:

```
18
```

This function is useful when the output size must be known before writing. However, if it is used before formatting the same content again, the cost of repeating the formatting work should be considered, especially when a custom formatter performs expensive computations.

### Comparing the Main Formatting Functions

| Function              | Primary purpose                                   | Return value                    |
| --------------------- | ------------------------------------------------- | ------------------------------- |
| `std::format`         | Produce a new string                              | `std::string` or `std::wstring` |
| `std::format_to`      | Write to a destination iterator                   | Final iterator                  |
| `std::format_to_n`    | Write with a character-count limit                | Iterator and total output size  |
| `std::formatted_size` | Calculate output size                             | Numeric size                    |
| `std::vformat`        | Format runtime strings with type-erased arguments | `std::string` or `std::wstring` |
| `std::vformat_to`     | Format dynamically into a destination iterator    | Final iterator                  |

The appropriate function depends on whether you need an independent string, whether the destination is already known, and whether memory or output-size constraints matter.

## Locale Support and Localized Formatting

### Understanding Locale in Formatting

Many applications need to display numbers and values according to a language or regional convention. For example, digit-grouping separators and decimal notation can differ between locales.

By default, `std::format` does not use locale-specific formatting. If you want a particular `std::locale` to influence formatting, you can provide it as a separate argument.

```
#include <format>
#include <iostream>
#include <locale>

int main() {
    std::locale loc("");

    std::cout << std::format(
        loc,
        "{:L}",
        1234567.89
    ) << '\n';
}
```

In this example, `std::locale("")` attempts to select the environment's locale, and the `L` option enables localized formatting for the relevant type.

The exact output depends on the available locale, system settings, and library support. To avoid unintended environmental dependencies, explicitly select the required locale when appropriate.

### Locale Limitations

Localized formatting is useful for financial reports, desktop applications, and output intended for users in specific regions. However, it requires caution when generating machine-readable data such as network protocols, configuration files, or storage formats.

In these cases, changing separators or numeric conventions may make the output incompatible with its consumer. Furthermore, `std::format` is not a complete system for message translation or multilingual text management.

## Formatting Custom Types with `std::formatter`

### Why Custom Formatters Are Useful

Professional applications do not always work with built-in types such as `int` and `double`. They may define types such as `Point`, `User`, `Version`, or `ErrorCode` that need specific textual representations.

The standard library allows you to specialize `std::formatter` to make a custom type compatible with `std::format`. This avoids duplicating string-conversion logic throughout the application.

### Implementing a Formatter for a Simple Type

In the following example, `Point` contains two coordinates, and its custom formatter displays them as `(x, y)`.

```
#include <format>
#include <iostream>

struct Point {
    int x;
    int y;
};

template <>
struct std::formatter<Point> : std::formatter<int> {
    auto format(const Point& point, std::format_context& ctx) const {
        return std::format_to(
            ctx.out(),
            "({}, {})",
            point.x,
            point.y
        );
    }
};

int main() {
    Point point{10, 20};

    std::cout << std::format("Position: {}", point) << '\n';
}
```

The program produces the following output:

```
Position: (10, 20)
```

In this implementation, `std::formatter<Point>` inherits from the integer formatter. For this example, which does not use integer formatting options, this is simply a way to reuse an implementation of `parse`.

The `format` function is responsible for writing the final representation to the output, and `ctx.out()` provides the output iterator.

An important design consideration is that a custom formatter must follow the library's requirements. If it is intended to support custom formatting options such as `{:#}` or `{:x}`, its parser and the behavior of those options must also be implemented correctly. Merely inheriting from another type's formatter does not automatically provide correct support for every specification.

### Implementing a Parser for Custom Options

For example, you can define a formatter for `Point` that supports both a normal and a compact representation. In this example, the `s` option selects compact output.

```
#include <format>
#include <iostream>

struct Point {
    int x;
    int y;
};

template <>
struct std::formatter<Point> {
    bool compact = false;

    constexpr auto parse(std::format_parse_context& ctx) {
        auto it = ctx.begin();
        auto end = ctx.end();

        if (it != end && *it == 's') {
            compact = true;
            ++it;
        }

        if (it != end && *it != '}') {
            throw std::format_error("Invalid Point format");
        }

        return it;
    }

    auto format(const Point& point, std::format_context& ctx) const {
        if (compact) {
            return std::format_to(
                ctx.out(),
                "{},{}",
                point.x,
                point.y
            );
        }

        return std::format_to(
            ctx.out(),
            "Point({}, {})",
            point.x,
            point.y
        );
    }
};

int main() {
    Point point{10, 20};

    std::cout << std::format("{}\n", point);
    std::cout << std::format("{:s}\n", point);
}
```

The program produces the following output:

```
Point(10, 20)
10,20
```

In this example, `parse` examines the format specification and enables compact mode when it encounters the `s` option. The `format` function then generates the appropriate output based on that state.

This pattern is useful for types with multiple valid textual representations, such as normal and compact paths, identifiers, or log-specific representations of a data type.

### Custom Formatter Design Considerations

When implementing a custom formatter, consider the following points:

- Parser rules: The `parse` function must correctly identify the end of the specification and reject invalid input.
- Consistent behavior: Output should not depend on hidden mutable state unless that behavior is intentional.
- Type safety: A formatter must not interpret data incorrectly or make invalid assumptions about value ranges.
- Runtime cost: Avoid unnecessary intermediate strings and write directly to the output iterator when practical.
- Specification contracts: Every supported option should have documented and testable behavior.

For types used in public APIs or shared libraries, a custom formatter may become part of the type's display contract. Changing its output format can therefore affect tests, logs, and dependent tools.

## Support for Standard Types and Newer Capabilities

### Formatting Dates and Times with `<chrono>`

An important C++20 capability is formatting many time-related types from `<chrono>`. This allows time values to be converted to strings without numerous manual conversions.

```
#include <chrono>
#include <format>
#include <iostream>

int main() {
    using namespace std::chrono;

    year_month_day date = 2026y / October / 28d;
    sys_time<milliseconds> timestamp{
        sys_days{date}.time_since_epoch()
    };

    std::cout << std::format(
        "{:%Y-%m-%d}",
        date
    ) << '\n';

    std::cout << std::format(
        "{:%Y-%m-%d %H:%M:%S}",
        timestamp
    ) << '\n';
}
```

The program produces the following output:

```
2026-10-28
2026-10-28 00:00:00
```

In this example, `%Y` represents the year, `%m` the month, `%d` the day, `%H` the hour, `%M` the minute, and `%S` the second.

An important distinction is that time formatting is not the same as time-zone interpretation. A `sys_time` represents a time based on the system clock's Unix time, and displaying local time or a particular time zone requires the appropriate conversion.

Support for specific time types and some advanced formatting specifications depends on the standard library version. For portable code, verify the compiler and library versions targeted by the project.

### Formatting Ranges in C++23

C++23 expands formatting support to include certain standard types, such as some ranges, `std::pair`, and `std::tuple`. In implementations with suitable support, this allows composite data to be formatted directly.

```
#include <format>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    std::cout << std::format("Values: {}", values) << '\n';
}
```

This example requires an implementation that supports range formatting in C++23. You should not assume that a `std::vector` can be formatted directly in C++20.

For C++20 compatibility, you can iterate over the collection manually and append its elements using `std::format_to`. This approach also provides greater control over separators, element formatting, and the collection's representation.

### Differences Between C++20, C++23, and C++26

The `std::format` feature has evolved in subsequent standard revisions. Knowing which standard introduced a capability is important when developing portable libraries.

| Standard | Related capabilities                                                                                                             |
| -------- | -------------------------------------------------------------------------------------------------------------------------------- |
| C++20    | Core formatting functions, `std::formatter`, built-in type formatting, `std::vformat`, and iterator-based output functions       |
| C++23    | Formatting improvements, enhanced format-string checking, and support for composite types and ranges in relevant implementations |
| C++26    | Introduction of `std::runtime_format` and further expansion of standard-library formatting capabilities                          |

The presence of a feature in the standard does not mean that every older compiler version has implemented it. For cross-platform projects, verify the actual library capabilities in addition to selecting the language standard.

## Error Handling and Safety Considerations

### Types of Formatting Errors

Formatting can fail for several reasons. The most important cases include:

- Compile-time errors: A constant format string is incompatible with the argument types or formatting grammar.
- Runtime errors: A dynamic format string is invalid or contains an inappropriate specification.
- Allocation failures: The memory required to produce the output is unavailable.
- Custom formatter failures: A custom formatter may throw an exception when encountering invalid data or state.

With ordinary `std::format` calls, constant format strings can be checked before the program runs. This reduces many common errors, but it does not eliminate every possible runtime failure.

### Risks of Untrusted Format Strings

A format string is not merely ordinary text; it can control the formatting structure. Therefore, if a user or external source can supply an arbitrary format string, its behavior must be handled carefully.

For example, an invalid format can cause an exception, while excessively large width or precision values can increase resource consumption. In a public service, this may lead to performance degradation or unnecessary memory usage.

Practical recommendations include:

- Validate input: Keep externally supplied format strings valid and constrained.
- Limit resources: Restrict dynamically supplied width and precision to reasonable ranges.
- Handle exceptions: Handle expected formatting failures at an appropriate application boundary.
- Separate data from format strings: If the goal is simply to display user input, pass it as an argument rather than using it as the format string.

For example, the following approach is preferable:

```
#include <format>
#include <iostream>
#include <string>

int main() {
    std::string user_input = "User supplied text: {}";

    std::cout << std::format(
        "Input: {}",
        user_input
    ) << '\n';
}
```

In this code, the braces inside the user's input are not interpreted as formatting instructions because the input is an ordinary argument. This pattern reduces errors caused by confusing data with formatting syntax.

### Formatting Versus Escaping for HTML and SQL

`std::format` does not automatically make data safe for HTML, SQL, JSON, or shell commands. It simply produces a string according to formatting rules.

For example, constructing an SQL statement with `std::format` is not a substitute for a parameterized query. Similarly, generating HTML or JSON requires following the relevant escaping and encoding rules.

Therefore, `std::format` should be used to construct general-purpose text. For outputs with specific structural or security requirements, use an appropriate API or library for the target format.

## Comparing Other String Construction Methods

### Comparing with `std::printf`

The `std::printf` function uses format strings containing specifiers such as `%d`, `%s`, and `%f`. This older API remains useful for certain C interfaces, existing libraries, and programs that need to interoperate with C code.

```
#include <cstdio>

int main() {
    int id = 42;
    double score = 98.5;

    std::printf("ID: %d, Score: %.1f\n", id, score);
}
```

By contrast, `std::format` infers argument types from the values themselves and can detect many mismatches at compile time.

```
#include <format>
#include <iostream>

int main() {
    int id = 42;
    double score = 98.5;

    std::cout << std::format(
        "ID: {}, Score: {:.1f}\n",
        id,
        score
    );
}
```

For new C++ code, `std::format` is generally a good choice for string construction. However, `std::printf` remains useful in some constrained environments, C-compatible interfaces, and legacy codebases.

### Comparing with `std::ostringstream`

The `std::ostringstream` approach uses the input/output stream system to construct strings. It is flexible and supports many types that provide a suitable insertion operator.

```
#include <iomanip>
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::ostringstream stream;

    stream << "Value: "
           << std::fixed
           << std::setprecision(2)
           << 3.14159;

    std::string result = stream.str();

    std::cout << result << '\n';
}
```

The equivalent code using `std::format` is shorter and often easier to read:

```
#include <format>
#include <iostream>

int main() {
    std::cout << std::format(
        "Value: {:.2f}",
        3.14159
    ) << '\n';
}
```

However, `std::ostringstream` remains useful when existing code already depends on manipulators, streams, or custom insertion operators. You should also avoid assuming that `std::format` is always faster than streams; performance depends on the data types, implementation, and usage pattern.

### Comparing with the `{fmt}` Library

The `{fmt}` library was an important influence on the design of standard C++ formatting and continues to offer useful capabilities. Its primary interfaces are very similar to those of `std::format`.

The advantage of `std::format` is that it is part of the standard library, so basic functionality does not require an external dependency. On the other hand, `{fmt}` may be a good choice for projects that need its specific features or support for older environments.

When choosing between the two, consider the following:

- Compatibility: Does the target standard library support the required functionality?
- Features: Does the project require capabilities available only in a particular library or version?
- Dependencies: Is adding an external library acceptable for the project?
- Performance: Do actual measurements indicate that one option is better suited to the workload?

## Performance and Optimization

### Reducing Memory Allocations

The `std::format` function creates a new string. Therefore, when large numbers of strings are generated repeatedly, allocation costs may become significant.

For better memory control, you can use `std::format_to` with a reusable buffer. However, choosing the appropriate iterator and managing destination capacity correctly are essential.

### Avoiding Intermediate Strings

In loops that format large amounts of data, consider writing multiple text fragments directly into a shared destination when appropriate, rather than creating several independent strings and concatenating them afterward.

This decision should be based on measurement. Premature optimization may reduce readability without producing a meaningful performance improvement.

### Using Benchmarks

Comparing `std::format`, `std::printf`, `std::ostringstream`, and `{fmt}` requires consistent test conditions. Factors such as compiler version, standard library implementation, optimization settings, allocation counts, and output size all affect the results.

Consequently, none of these methods should be considered universally fastest without appropriate benchmarking.

## Common Mistakes When Using `std::format`

### Using a Format Specifier Incompatible with the Argument Type

An invalid format specification for an argument can cause a compile-time or runtime error. For example, `x` is appropriate for hexadecimal integer formatting, but it should not be assumed to be valid for every data type.

### Mixing Automatic and Manual Argument Indexing

Mixing `{}` with `{0}` in the same format string is not allowed. To avoid this error, choose a single indexing style for each format string.

### Assuming Complete C++20 Support

Using `-std=c++20` alone does not guarantee that every component of `std::format` is available in the installed standard library. For newer capabilities, verify both the required language standard and the actual library support.

### Confusing Precision with Changing a Value

Setting precision does not modify the original numeric value. If a calculation must be rounded and stored in a variable, perform that operation separately.

### Using Formatting as a Substitute for Serialization

Constructing a string with `std::format` does not guarantee valid JSON, a valid network message, or data that satisfies a particular storage contract. The rules of the target format must still be followed.

## Final Summary

The `std::format` feature introduced in C++20 provides a standardized, readable, and type-safe approach to constructing text strings. It supports features such as width and precision control, numeric-base selection, time formatting, custom formatters, and direct writing to output iterators.

Professional use requires understanding the distinction between constant and runtime format strings, handling errors appropriately, verifying actual standard-library support, and selecting the right API for the task. Formatting must also be distinguished from security escaping and serialization.

For new C++ projects, `std::format` is generally a suitable choice for generating messages, reports, and display strings. By using it appropriately and respecting its limitations, developers can write code that is more readable, maintainable, and aligned with modern C++ standards.

## Further Reading

- [cppreference: ](https://en.cppreference.com/w/cpp/utility/format?utm_source=chatgpt.com)[`std::format`](https://en.cppreference.com/w/cpp/utility/format?utm_source=chatgpt.com)
- [cppreference: ](https://en.cppreference.com/w/cpp/utility/format/format_to?utm_source=chatgpt.com)[`std::format_to`](https://en.cppreference.com/w/cpp/utility/format/format_to?utm_source=chatgpt.com)
- [cppreference: ](https://en.cppreference.com/w/cpp/header/format?utm_source=chatgpt.com)[`<format>`](https://en.cppreference.com/w/cpp/header/format?utm_source=chatgpt.com)[ Header](https://en.cppreference.com/w/cpp/header/format?utm_source=chatgpt.com)
- [The ](https://fmt.dev/?utm_source=chatgpt.com)[`{fmt}`](https://fmt.dev/?utm_source=chatgpt.com)[ Library Documentation](https://fmt.dev/?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>