<div align="center">

[🇺🇸 English](./std_format improvements.md) | [🇮🇷 فارسی](../../fa/cpp23/std_format improvements.md)

</div>

---

# **Improvements to&#x20;**`std::format`**&#x20;in C++23**

## **Introduction**

The Formatting Library, introduced in C++20, provides a type-safe and extensible way to create formatted strings. In many applications, it is a suitable replacement for the `printf` family and a useful complement to traditional input/output streams such as `std::cout`. However, its initial version had several limitations: many Containers could not be formatted directly, some standard library types did not have formatters, the handling of Unicode text width and fill characters was not sufficiently precise, and additional capabilities were needed for dynamic format strings and more generic tools.

C++23 extends this infrastructure in several directions: standard support for formatting Ranges and Containers, improved facilities for writing custom formatters, refinements related to Unicode and display width, better string escaping, and support for types such as Thread IDs and Stacktraces. It also incorporates several technical corrections and compatibility improvements to facilities introduced in C++20.

An important point is that not everything available in C++23 is necessarily a completely new language feature. `std::format` is a **standard library** feature provided through the `<format>` header. Consequently, enabling the `-std=c++23` compiler option alone does not guarantee that every component has been implemented in the installed standard library.

## **An Overview of&#x20;**`std::format`

### **C++20 Fundamentals**

The `std::format` facility was introduced in C++20, and its main functions are declared in the following header:

```
#include <format>
```

The `std::format` function produces its result as a new `std::string`:

```
#include <format>
#include <iostream>
#include <string>

int main() {
    const std::string user = "Ava";
    const int score = 97;

    const auto message = std::format(
        "User: {}, score: {:03}",
        user,
        score
    );

    std::cout << message << '\n';
}
```

In this example, `{}` is a placeholder for an argument, while `{:03}` formats the number using at least three digits, padding it with leading zeros. By default, arguments correspond to fields in their order of appearance. Explicit indices such as `{0}` and `{1}` can also be used.

### **The Formatting Function Family**

In addition to `std::format`, several functions are available for different scenarios:

| Function              | Purpose                                                     |
| --------------------- | ----------------------------------------------------------- |
| `std::format`         | Creates a new string                                        |
| `std::format_to`      | Writes output to an Output Iterator                         |
| `std::format_to_n`    | Writes at most a specified number of characters             |
| `std::formatted_size` | Calculates the number of characters in the formatted output |
| `std::vformat`        | Formats using type-erased arguments                         |
| `std::vformat_to`     | Type-erased version that writes to an Output Iterator       |

For example, when the destination is a Buffer or Iterator, `std::format_to` can avoid creating an intermediate string:

```
#include <format>
#include <iterator>
#include <string>

int main() {
    std::string output;
    output.reserve(64);

    std::format_to(
        std::back_inserter(output),
        "id={}, active={}",
        42,
        true
    );
}
```

These APIs are part of C++20. Their relevance to C++23 is that the new Range Formatting and custom Formatter capabilities build on the same infrastructure.

## **Major Formatting Improvements in C++23**

### **Direct Formatting of Ranges**

One of the most significant changes in C++23 is support for formatting Ranges, introduced through the standardization proposal P2286R8. Before this change, printing a `std::vector` or many other Containers generally required iterating over the elements with a loop or writing a custom Formatter.

In C++23, supported formattable Range types can be passed directly to `std::format`:

```
#include <format>
#include <iostream>
#include <vector>

int main() {
    const std::vector<int> values{10, 20, 30, 40};

    std::cout << std::format("values = {}", values) << '\n';
}
```

The default representation of a Range generally includes element separators and delimiters that identify the range. The exact representation depends on the Range type and its classification under the library rules. Therefore, the default representation should not be relied upon for output that is part of a Protocol or a stable storage format.

#### **Range Types and Formatting Categories**

The library uses `std::format_kind` and `std::range_format` to select how Ranges are displayed. Important categories include:

- `sequence`: Displays a sequence of elements, as with many ordered Containers.
- `map`: Displays key/value Ranges, such as Maps.
- `set`: Displays set-like Ranges.
- `string`: Treats a type as a string rather than as a sequence of individual characters.
- `debug_string`: Displays a string using Debug formatting and appropriate escaping rules.
- `disabled`: Disables generic Range formatting for the type in question.

These categories are part of the library's formatting machinery and are important when controlling or customizing Range Formatters. For example, `std::string` should not be printed as a generic Container of `char` elements; its string behavior must be preserved.

#### **Formatting Different Container Types**

```
#include <format>
#include <iostream>
#include <map>
#include <set>
#include <string>
#include <vector>

int main() {
    const std::vector<int> numbers{1, 2, 3};
    const std::set<int> unique_numbers{4, 5, 6};
    const std::map<std::string, int> ages{
        {"Ada", 36},
        {"Linus", 54}
    };

    std::cout << std::format("{}\n", numbers);
    std::cout << std::format("{}\n", unique_numbers);
    std::cout << std::format("{}\n", ages);
}
```

Range formatting is particularly useful for Logging, generating Diagnostic messages, testing, and displaying the state of Containers. However, it should not be considered a replacement for Serialization: formatted output is not necessarily a portable, versioned contract for data exchange.

### **Controlling Range Formatting with Format Specifiers**

Range Formatting is not limited to the default representation. `std::range_formatter` provides infrastructure for defining how a Range is formatted and allows control over element-related specifications, separators, and delimiters.

For example, a custom Formatter for a Range-like type can be implemented using `std::range_formatter`. The following example shows a simple Wrapper that uses Range-based formatting:

```
#include <format>
#include <string>
#include <vector>

struct Scores {
    std::vector<int> values;
};

template <>
struct std::formatter<Scores> : std::range_formatter<int> {
    auto format(const Scores& scores, std::format_context& ctx) const {
        return std::range_formatter<int>::format(scores.values, ctx);
    }
};

int main() {
    const Scores scores{{80, 90, 100}};
    const auto text = std::format("{}", scores);
}
```

This example uses `std::range_formatter` to reuse the Range-formatting logic. In a real project, the Character type and Context must also be considered when designing a Formatter. Specializing only for `char` is not sufficient if support for `wchar_t` or other Character types is required.

### **Improvements to Default Container Formatting**

Proposal P2585R1, titled *Improving default container formatting*, is related to completing and refining the default formatting behavior of Containers alongside Range Formatting. The goal is to make the representation of standard Containers more useful and consistent for everyday use and Diagnostics.

The key design point is that the default representation is optimized for human readability, not for guaranteeing an independent, parseable Syntax. If an application stores output in a file, on a network, or in a database and needs to Parse it later, it should choose a defined Serialization Format such as JSON or a custom format with a versioned contract.

### **The&#x20;**`std::formattable`**&#x20;Concept**

C++23 introduces the `std::formattable` Concept. It determines whether a type can be formatted for a specified Character type.

```
#include <concepts>
#include <format>
#include <string>

static_assert(std::formattable<int, char>);
static_assert(std::formattable<std::string, char>);
```

The second parameter specifies the Character type used for formatting. Therefore, a type being Formattable for `char` does not necessarily mean it is also Formattable for `wchar_t`.

This Concept is useful when writing Generic APIs. Instead of allowing complex errors related to `std::formatter` to appear deep inside Template instantiations, the constraint can be expressed directly in the function signature:

```
#include <format>
#include <string>

template <typename T>
requires std::formattable<T, char>
std::string make_message(const T& value) {
    return std::format("value = {}", value);
}
```

In this example, the function can only be used with types suitable for Narrow Character formatting with `char`.

### **Improved Support for Non-**`const`**-Formattable Types**

Proposal P2418R2 reduces limitations that prevented certain non-copyable objects or objects that could not be formatted through `const` access from being formatted. This change matters for types whose observation or representation-generation operations may require non-`const` access. Generators and some single-pass Ranges are important examples of this group.

Before this correction, the formatting model was sometimes too dependent on `const` access. In the revised model, the library can use a non-`const` object when necessary. However, this depends on the constraints and the Range or Formatter type; it does not mean that arbitrary State changes are permitted in all Formatters.

This change was also applied to C++20 as a Defect Report. As a result, an implementation may provide it even when operating in C++20 mode. The documentation and Feature-test Macro of the relevant library should be checked to determine whether it is available.

## **String and Unicode Improvements**

### **Refinements to Display-Width Rules**

One challenge in text formatting is the difference between the number of Code Units and the width that text occupies in a Terminal or other display environment. For example, a Unicode character may occupy several Bytes in UTF-8 while taking up only one or even two columns on screen. A Grapheme Cluster may also consist of multiple Code Points.

Proposal P2675R1 makes width estimation in `std::format` more accurate and forward-looking. This is particularly important for Alignment and Padding, because field width should not simply be assumed to equal the number of Bytes in a string.

```
#include <format>
#include <iostream>

int main() {
    std::cout << std::format("|{:^12}|\n", "hello");
}
```

In this example, the string is placed in a field of a specified width and Center-aligned. When Unicode text is used, the visual result depends on the standard's width-estimation rules, the Encoding, and the implementation's support.

It is important to note that `std::format` is not a complete Layout engine for complex text. It is not a replacement for advanced text-layout algorithms, Line Breaking, BiDi processing, or natural-language text shaping.

### **Greater Flexibility in Fill Characters**

In formatting Syntax, the Fill character is used to fill the available space around a value. P2572R1 refined the rules for Fill Characters to reduce unnecessary restrictions and clarify their behavior when Unicode characters are involved.

A simple example of Fill and Alignment:

```
#include <format>
#include <iostream>

int main() {
    std::cout << std::format("{:*^10}\n", "C++");
    std::cout << std::format("{:.>10}\n", 42);
}
```

In the first example, `*` fills the available space on both sides of the string. In the second, a period is used to fill the space to the left of the number.

However, the display width of a Fill Character is not always equal to its number of Code Units. Choosing multi-column characters or complex Graphemes may cause the final appearance to differ across Terminals. P2572R1 clarifies this area in the standard rules, but it does not eliminate differences between Fonts, Terminals, and display environments.

### **Improved String Escaping**

Proposal P2713R1 improves the escaping capabilities of `std::format`. This is useful for displaying strings more safely and clearly in Debug output, particularly when a string contains control characters, quotation marks, or characters whose direct display could make the output difficult to read.

In the string Format Specification, the Debug presentation can be selected using `?`:

```
#include <format>
#include <iostream>
#include <string>

int main() {
    const std::string text = "first line\nsecond line";

    std::cout << std::format("normal: {}\n", text);
    std::cout << std::format("debug:  {:?}\n", text);
}
```

Normal string formatting inserts the string's contents into the output as they are. Debug Formatting, on the other hand, can display special characters in escaped form. This distinction is important in Logging and error reports because a hidden Newline or Tab may disrupt the structure of a log.

Debug output should not be treated as a Serialization Format without a clearly defined contract. Its primary purpose is diagnostic display, not guaranteeing Round-trip behavior for all Encodings and data.

## **Expanding the Set of Formattable Types**

### **Formatting Thread IDs**

Proposal P2693R1 makes it possible to use `std::thread::id` directly with `std::format`:

```
#include <format>
#include <iostream>
#include <thread>

int main() {
    const std::thread::id id = std::this_thread::get_id();

    std::cout << std::format("current thread: {}\n", id);
}
```

This capability is useful for multithreaded Logging and Concurrency Diagnostics. A Thread ID is a displayable value for identifying a Thread within the execution of a program. It should not be assumed to be a stable identifier across different executions or a portable value for long-term storage.

### **Formatting Stacktraces**

The same proposal also adds formatting support for Stacktrace-related types. Stacktrace support in the standard library is provided in C++23 through the `<stacktrace>` header and is useful for recording call paths when an error occurs.

```
#include <format>
#include <iostream>
#include <stacktrace>

int main() {
    const auto trace = std::stacktrace::current();

    std::cout << std::format("stack trace:\n{}\n", trace);
}
```

In environments without full Stacktrace support, or where collecting stack information is not possible, the behavior and output details may depend on the implementation and Build settings. A Stacktrace may also expose sensitive internal application information, file paths, or Symbol names. Therefore, publishing it in public Logs or network responses requires security considerations.

### **Introducing&#x20;**`std::basic_format_string`

Proposal P2508R1 made the public name `std::basic_format_string` available to programmers. This type is the foundation of type-safe Format Strings and makes it possible to define APIs whose Format Strings are checked at Compile-time.

```
#include <format>
#include <string_view>

template <typename... Args>
std::string make_text(
    std::format_string<Args...> fmt,
    Args&&... args
) {
    return std::format(fmt, std::forward<Args>(args)...);
}

int main() {
    const auto text = make_text("id = {}, name = {}", 7, "Ada");
}
```

This example uses the standard Alias `std::format_string`, which is based on `std::basic_format_string`. As a result, mismatches between the number or types of arguments and the Format Specifiers can be detected at Compile-time when they are eligible for compile-time checking.

This capability is particularly important when building Logging and Formatting Wrappers. If a Format String is received at runtime from a file or user input, it follows a different validation path. It should not be assumed that every Runtime string is checked in the same way as a Compile-time Format String.

## **Corrections and Clarifications Related to C++20**

### **Compile-Time Format String Checking**

Proposal P2216R3 strengthened Compile-time Format String checking and improved error detection. This feature is identified in the Feature-test table by the value `__cpp_lib_format == 202106L` and is also associated with C++20 as a Defect Report.

```
#include <format>

int main() {
    const auto valid = std::format("value = {}", 42);

    // This format string is intentionally invalid.
    // A conforming implementation should diagnose the mismatch.
    // const auto invalid = std::format("{:d}", "text");
}
```

Under normal circumstances, a constant Format String that can be evaluated at Compile-time is checked in a type-safe manner. However, the details of Diagnostics and the exact extent of checking may differ between Compilers. The standard specifies the semantic rules and diagnostic requirements, not identical error-message text.

### **Locale Corrections in Chrono Formatters**

Proposal P2372R3 refined Locale behavior in Formatters related to `std::chrono`. This change concerns how Locale is applied when formatting dates and times. It is important for applications that generate Locale-dependent representations of dates, month names, or other information.

Proposal P2419R2 also clarifies how Encoding is handled when formatting localized Chrono types. These corrections concern the boundary between text formatting and date/time data. Consequently, when using `L` in a Format Specifier and selecting a Locale, the output Encoding and Character type must also be considered.

A general example of time formatting:

```
#include <chrono>
#include <format>
#include <iostream>

int main() {
    const auto now = std::chrono::system_clock::now();

    std::cout << std::format("{:%Y-%m-%d %H:%M:%S}\n", now);
}
```

The Chrono Format Specifiers are part of C++20. The C++23-related corrections in this area focus primarily on Locale compatibility and Encoding clarity.

### **Reducing Parameterization in&#x20;**`std::vformat_to`

Another part of P2216R3 concerns reducing unnecessary parameterization in `std::vformat_to`. This change is mostly an implementation-level library improvement and is not usually visible directly in everyday code. However, it matters to standard library implementers and developers of APIs based on type-erased Formatting.

## **Writing Custom Formatters**

### **Creating a Formatter for a User-Defined Type**

One of the key features of the Formatting Library is the ability to define `std::formatter` for User-defined types. This capability has existed since C++20 and became more flexible in C++23 with Range Formatting and the new Concepts.

```
#include <format>
#include <string>

struct User {
    std::string name;
    int age;
};

template <>
struct std::formatter<User> : std::formatter<std::string> {
    auto format(const User& user, std::format_context& ctx) const {
        const auto text = std::format(
            "{} ({})",
            user.name,
            user.age
        );

        return std::formatter<std::string>::format(text, ctx);
    }
};

int main() {
    const User user{"Ada", 36};
    const auto text = std::format("{}", user);
}
```

In this pattern, the Formatter for `User` reuses the string Formatter so that the basic Alignment and Width rules do not need to be implemented again. For a more generic Formatter, note that `std::format_context` is specifically the Context for `char`. If support for other Character types is required, a Template pattern appropriate for `CharT` and the corresponding Context should be used.

### **Designing a Custom Format Specifier**

A custom Formatter can define its own Format Specifier. This is generally done by implementing `parse` and `format`:

```
#include <format>
#include <string_view>

struct Point {
    int x;
    int y;
};

template <>
struct std::formatter<Point> {
    bool compact = false;

    constexpr auto parse(std::format_parse_context& ctx) {
        auto it = ctx.begin();

        if (it != ctx.end() && *it == 'c') {
            compact = true;
            ++it;
        }

        if (it != ctx.end() && *it != '}') {
            throw std::format_error("invalid Point format");
        }

        return it;
    }

    auto format(const Point& point, std::format_context& ctx) const {
        if (compact) {
            return std::format_to(
                ctx.out(),
                "({}, {})",
                point.x,
                point.y
            );
        }

        return std::format_to(
            ctx.out(),
            "Point{{x={}, y={}}}",
            point.x,
            point.y
        );
    }
};

int main() {
    const Point p{10, 20};

    const auto normal = std::format("{}", p);
    const auto compact = std::format("{:c}", p);
}
```

This example shows how a Formatter can define a small Grammar for its Format Specifier. In production code, the behavior of `parse` must be precise for empty input, the end of a Field, and invalid input. Format Strings should also remain eligible for Compile-time checking whenever possible.

### **Using&#x20;**`std::range_formatter`

When a custom type essentially stores a Range, `std::range_formatter` can handle much of the work. This avoids duplicating the logic for separators, delimiters, and element Formatters.

Before choosing this approach, determine whether your type should genuinely be represented semantically as a Range or whether it has its own domain-specific representation. For example, a matrix, network path, or set of transactions may require formatting that is more than a simple list of elements.

## **Performance and API Design**

### **Choosing Between&#x20;**`format`**&#x20;and&#x20;**`format_to`

`std::format` creates a new string. This behavior is readable and suitable for most messages. If the destination is already a Buffer or Output Iterator, `std::format_to` can avoid creating a temporary string.

```
#include <format>
#include <iterator>
#include <string>

int main() {
    std::string log_line;
    log_line.reserve(128);

    std::format_to(
        std::back_inserter(log_line),
        "[{}] user={} status={}",
        1234,
        "Ada",
        200
    );
}
```

Choosing `format_to` does not, by itself, guarantee that no Allocation will occur. The destination, Buffer growth, and argument Formatters may still allocate memory. To achieve meaningful optimizations, the complete output-generation path should be Benchmarked.

### **Non-Locking Formatter Optimization**

C++23 introduces the variable template `std::enable_nonlocking_formatter_optimization`. This Trait allows an implementation to choose a more efficient path for types whose Formatters do not require Locking mechanisms.

This facility is a tool for standard library implementations and should not be interpreted as a general guarantee of Thread-safety or the elimination of all Locks. Programmers should provide a specialization for their types only when the standard's requirements have been carefully verified. Incorrectly declaring this property can cause the program to violate implementation requirements.

## **Detecting Library and Compiler Support**

### **Important Feature-Test Macros**

For library features, checking Feature-test Macros is more accurate than relying on the Compiler version number. These Macros are available in the relevant headers or through `<version>`.

| Macro                     | Value     | Feature                                                               |
| ------------------------- | --------- | --------------------------------------------------------------------- |
| `__cpp_lib_format`        | `201907L` | Basic Text Formatting functionality in C++20                          |
| `__cpp_lib_format`        | `202106L` | Compile-time Format String Checks and related corrections             |
| `__cpp_lib_format`        | `202110L` | Chrono Locale corrections and support for Non-const-formattable Types |
| `__cpp_lib_format`        | `202207L` | Exposure of `std::basic_format_string` and Encoding clarification     |
| `__cpp_lib_format_ranges` | `202207L` | Range Formatting in C++23                                             |
| `__cpp_lib_formatters`    | `202302L` | Formatters for `std::thread::id` and `std::stacktrace` in C++23       |

For example, the availability of Range Formatting can be checked as follows:

```
#include <format>

#if defined(__cpp_lib_format_ranges) && \
    __cpp_lib_format_ranges >= 202207L
    #define HAS_STD_FORMAT_RANGES 1
#else
    #define HAS_STD_FORMAT_RANGES 0
#endif

int main() {
    static_assert(HAS_STD_FORMAT_RANGES == 0 ||
                  HAS_STD_FORMAT_RANGES == 1);
}
```

This check indicates whether the feature is available in the library. However, using C++23-standard Syntax and APIs still requires the Compiler and library to be configured for the appropriate language mode.

### **Why the Compiler Version Is Not Enough**

Support for `std::format` depends on two components:

- **Compiler:** It must accept the required language mode and C++23 features.
- **Standard Library:** It must implement the required APIs, such as Range Formatters.

A Compiler version may support parts of C++23 while its accompanying library still lacks some Formatting features. Conversely, a newer version of the same library may provide certain standard corrections even in C++20 mode.

For cross-platform projects, the required features should be tested in CI using the actual Compiler and Standard Library combinations targeted by the project.

## **Comparing C++20 and C++23 Behavior**

| Topic                                           | C++20                                                     | C++23                                            |
| ----------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------ |
| `std::format` and the main function family      | Introduced                                                | Retained and refined                             |
| Compile-time Format String checking             | More limited in the initial version; improved through DRs | Refined rules and better Diagnostics             |
| Generic Range formatting                        | No general standard support                               | Added through P2286R8                            |
| `std::range_formatter`                          | Not available                                             | Added                                            |
| `std::formattable`                              | Not available                                             | Added                                            |
| Formatters for `std::thread::id` and Stacktrace | No general standard support                               | Standardized in P2693R1                          |
| Fill rules and Unicode width estimation         | Had limitations and ambiguities                           | Improved through P2572R1 and P2675R1             |
| String Debug Escaping                           | More limited                                              | Improved through P2713R1                         |
| `std::basic_format_string`                      | Public name not available in the original text            | Exposed through P2508R1; also applicable as a DR |

This table describes the differences between the initial versions of the standards, not the state of every available library. Defect Reports may also make a feature or correction available to an implementation in C++20 mode.

## **Limitations and Common Mistakes**

### **Confusing Constant and Dynamic Format Strings**

Constant Format Strings used with type-safe APIs such as `std::format` can be checked at Compile-time. However, a string received from a file, network, or user input is a Runtime string. The same behavior should not be assumed for it without considering the API and the standard version.

In C++23, dynamic Format String scenarios still require checking the appropriate API and its limitations. The standard API `std::runtime_format` belongs to C++26, not C++23, and should therefore not be presented as a C++23 feature. In earlier versions, `std::vformat` is useful for type-erased Formatting and Runtime Format Strings, but the arguments and their lifetimes must be managed correctly.

### **Assuming That All Ranges Are Fully Supported**

Whether a Range is formattable depends on its Constraints and whether its elements are Formattable. Not every type that can be iterated over is necessarily formattable with every Character type. Some Ranges may also be single-pass or require non-`const` State for iteration.

Before designing a public API, review the constraints of `std::formattable` and the requirements of Range Formatting.

### **Using Default Output for Serialization**

The default representation of Containers is designed for Debugging and Logging. It should not be assumed to be consistently parseable, version-independent, or identical across all implementations. For stable storage, choose an explicit data format and a version-compatible contract.

### **Ignoring Formatting Costs**

Formatting may involve Allocation, Encoding conversion, Range iteration, and execution of custom Formatters. For frequently executed paths, such as Logging inside performance-sensitive loops, avoid unnecessary string construction, choose an appropriate Buffer, and run real Benchmarks.

### **Ignoring Untrusted Data**

If a Format String comes from untrusted input, do not treat it as equivalent to a constant Format String in the program. An invalid Format String can cause a formatting error, so validation and error handling should be considered when designing Logging systems or general-purpose tools. Argument contents may also contain sensitive data; type-safe formatting alone does not provide a data-confidentiality policy.

### **Assuming Identical Unicode Display Across All Terminals**

The standard can specify width calculation and Fill rules, but the Font, Terminal, environment Encoding, and Grapheme support all affect the final appearance. For text interfaces where Layout is important, test the output in the target environments.

## **Choosing the Right Feature**

- For creating an ordinary string, `std::format` is a simple and readable choice.
- For writing directly to a Buffer or Iterator, consider `std::format_to`.
- For quickly displaying Containers in Logs and tests, C++23 Range Formatting is very useful.
- For building a public Generic API, a Constraint based on `std::formattable` can make the function contract clearer.
- For domain-specific types, write a custom Formatter and, where appropriate, use existing Formatters as a base.
- For displaying strings that contain control characters, consider Debug Formatting.
- For stable, exchangeable data, use a defined Serialization Format instead of the default Formatter output.
- For projects targeting multiple Compilers, rely on Feature-test Macros and actual Build tests.

## **Summary**

The Formatting improvements in C++23 turn the library from a string-construction tool with limited type support into a more complete infrastructure for displaying Ranges and building Generic Formatters. Support for `std::formattable`, `std::range_formatter`, and Container formatting simplifies everyday coding and Diagnostics. Improvements to Unicode handling, Fill Characters, and Debug Escaping also enhance text presentation, while support for Thread IDs and Stacktraces is useful for diagnostic tools.

Despite these improvements, it is important to distinguish between formatting for humans and Serialization for machines. Actual support for each feature also depends on the Standard Library version, not just the Compiler's language option. Using Feature-test Macros, testing on target Toolchains, and carefully designing custom Formatters are essential for using these capabilities reliably in professional projects.

## **References and Further Reading**

- [cppreference — Formatting library](https://en.cppreference.com/w/cpp/utility/format?utm_source=chatgpt.com)
- [cppreference — Feature-test macros](https://en.cppreference.com/w/cpp/feature_test?utm_source=chatgpt.com)
- [WG21 P2286R8 — Formatting Ranges](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2286r8.html?utm_source=chatgpt.com)
- [WG21 P2418R2 — Add support for non-const-formattable types to std::format](https://wg21.link/P2418R2?utm_source=chatgpt.com)
- [WG21 P2508R1 — Exposing std::basic_format_string](https://wg21.link/P2508R1?utm_source=chatgpt.com)
- [WG21 P2572R1 — std::format fill character allowances](https://wg21.link/P2572R1?utm_source=chatgpt.com)
- [WG21 P2675R1 — std::format's width estimation](https://wg21.link/P2675R1?utm_source=chatgpt.com)
- [WG21 P2693R1 — Formatting thread::id and stacktrace](https://wg21.link/P2693R1?utm_source=chatgpt.com)
- [WG21 P2713R1 — Escaping improvements in std::format](https://wg21.link/P2713R1?utm_source=chatgpt.com)
- [libc++ — Format status](https://libcxx.llvm.org/Status/Format.html?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>