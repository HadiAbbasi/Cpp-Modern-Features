<div align="center">

[🇺🇸 English](./std_println.md) | [🇮🇷 فارسی](../../fa/cpp23/std_println.md)

</div>

---

# Introduction to std::println in C++23

## Introduction

`std::println` is a C++23 standard library feature that simplifies printing formatted text followed by a newline. It belongs to the new `std::print` and `std::println` function family and uses the formatting infrastructure introduced in C++20 through `std::format`.

In older approaches, output was typically printed using `std::cout` and the insertion operator `<<`, or functions such as `printf`. These approaches are still valid, but each has limitations related to readability, formatting, type safety, or format-string handling. The `print` family provides a standard and direct way to combine a format string with arguments; `println` also appends a newline character at the end of the output.

This feature is part of the **Standard Library**, not a language syntax feature. Therefore, enabling C++23 language mode alone does not guarantee that the standard library in use implements `<print>` and `std::println`.

## The Problem std::println Solves

### Difficulties with Older Output Methods

The `std::cout`-based approach is suitable for many use cases, but formatting different values may require a chain of operators, manipulation of stream state, and the use of manipulators. For example, setting the number of decimal places or displaying a value in hexadecimal affects the stream state, and that state can also affect subsequent output.

The `printf` approach offers compact formatting, but its format specifiers are not matched with arguments in a type-safe way at compile time. A mismatch between `%d` and the argument type, for example, can result in undefined behavior.

`std::println` reduces these problems by using type-safe format strings and the standard library's formatter mechanism. It also eliminates the need to append `'\n'` manually.

### Main Benefits

- **Type-safe formatting:** For constant format strings, argument types and counts are checked at compile time.
- **Readability:** The output format and its values are specified in a single call.
- **Convenient line termination:** The function automatically appends `'\n'` to the output.
- **Consistency with `std::format`:** Its formatting rules are the same as those of the feature introduced in C++20.
- **Support for custom types:** User-defined types can be made formattable by specializing `std::formatter`.

## Required Header and Basic Usage

To use the primary overloads of `std::println`, include the `<print>` header:

```cpp
#include <print>

int main() {
    std::println("Hello, world!");
}
```

This program prints `Hello, world!` and then writes a newline.

An important point is that the name `println` means print and then move to the next line. This differs from `std::print`, which does not append a newline by itself.

## Function Signatures and Overloads

C++23 introduces two primary overloads in the `<print>` header:

```cpp
template<class... Args>
void println(std::format_string<Args...> fmt, Args&&... args);

template<class... Args>
void println(
    std::FILE* stream,
    std::format_string<Args...> fmt,
    Args&&... args
);
```

The first overload sends output to `stdout`. The second specifies the destination using a `std::FILE` pointer; therefore, output can be directed to an opened file or a standard C stream.

Both overloads format the arguments according to `fmt` and append a `'\n'` at the end. The function's return type is `void`.

### Printing to Standard Output

```cpp
#include <print>

int main() {
    std::println("Application started");
    std::println("User: {}", "Alice");
    std::println("Retry count: {}", 3);
}
```

### Printing to a File with std::FILE

```cpp
#include <cstdio>
#include <print>

int main() {
    if (std::FILE* file = std::fopen("application.log", "w")) {
        std::println(file, "Application started");
        std::println(file, "Exit code: {}", 0);
        std::fclose(file);
    }
}
```

The example above checks whether the file was opened successfully. `std::println` does not take ownership of the file or close it; managing the lifetime of the `std::FILE` object remains the caller's responsibility.

## Difference Between std::print and std::println

Both functions are provided in C++23 through the `<print>` header. Their main difference is whether a newline is appended.

```cpp
#include <print>

int main() {
    std::print("Loading");
    std::print("...");
    std::println(" done");
    std::println("Ready");
}
```

Output:

```text
Loading... done
Ready
```

`std::print` is suitable for building multi-part output on a single line. For output where each record or message should occupy its own line, `std::println` is the more natural choice.

Using `std::println` with a string that already ends in `'\n'` usually produces two consecutive newline characters. Therefore, it is important to know whether the data already contains a line ending or whether the function is expected to add one.

## Formatting Arguments

The format-string rules used by `std::println` follow those of the `std::format` family. The `{}` replacement fields are filled with arguments, and formatting specifications can be written after a colon.

### Simple Replacement Fields

```cpp
#include <print>
#include <string>

int main() {
    std::string name = "Ada";
    int age = 36;

    std::println("Name: {}, age: {}", name, age);
}
```

Replacement fields are filled with the corresponding arguments in order. In ordinary format strings, the number of replacement fields and arguments must be compatible.

### Formatting Numbers

```cpp
#include <print>

int main() {
    int value = 255;

    std::println("Decimal: {}", value);
    std::println("Hexadecimal: {:x}", value);
    std::println("Uppercase hexadecimal: {:X}", value);
    std::println("Binary: {:b}", value);
    std::println("Padded: {:08}", value);

    double pi = 3.1415926535;
    std::println("Pi: {:.2f}", pi);
}
```

Formatting specifiers make it possible to control the display base, field width, decimal precision, and similar properties. The exact rules for each type depend on that type's formatter.

### Specifying Argument Order

A format string can use indices to explicitly select arguments:

```cpp
#include <print>

int main() {
    std::println("{1} is learning {0}", "C++", "Nina");
}
```

In this example, `{1}` refers to the second argument and `{0}` refers to the first. Automatic and manual argument indexing must not be combined inconsistently within a single format string.

### Text Formatting and Alignment

```cpp
#include <print>

int main() {
    std::println("|{:<12}|", "left");
    std::println("|{:^12}|", "center");
    std::println("|{:>12}|", "right");
}
```

These specifiers align text to the left, center, or right within a field of a specified width. The field width is a minimum amount of space; if the text is longer, it is generally not truncated.

## Type Safety of Format Strings

The format parameter has the type `std::format_string<Args...>`. For constant format strings, this type makes it possible to check the compatibility of the format string and its arguments at compile time.

```cpp
#include <print>

int main() {
    int count = 42;

    std::println("Count: {}", count);

    // This format string is invalid for the supplied argument.
    // std::println("Count: {:s}", count);
}
```

In the example above, the `s` specifier is not suitable for an integer value, and a standard-conforming implementation should diagnose the error at compile time. The invalid line is commented out so that the program remains compilable.

Type safety does not mean that all runtime errors are eliminated. A custom formatter can throw an exception at runtime, and writing to the output destination can also fail.

## Formatting Custom Types

To print a user-defined type, provide a formatter compatible with the `std::format` framework. This is typically done by specializing `std::formatter`.

```cpp
#include <format>
#include <print>
#include <string>

struct User {
    std::string name;
    int id;
};

template<>
struct std::formatter<User> : std::formatter<std::string> {
    auto format(const User& user, std::format_context& ctx) const {
        return std::formatter<std::string>::format(
            user.name + " (#" + std::to_string(user.id) + ")",
            ctx
        );
    }
};

int main() {
    User user{"Ada", 7};
    std::println("User: {}", user);
}
```

This formatter reuses the string formatter and delegates the object's textual representation to it. In a real project, a formatter should be as simple and predictable as possible, and should not depend on external state.

The design principle is that a formatter is responsible for converting a value into its textual representation, not for performing side effects such as writing directly to a file, changing program state, or logging. This separation makes the type easier to reuse with `std::format` and other formatting functions.

## Unicode Output and Environment-Dependent Behavior

The printing functions in `<print>` are designed with Unicode text output in mind. In environments where the output destination is recognized as a Unicode-capable terminal, the library can use an appropriate Unicode printing path to help avoid incomplete or corrupted output.

However, correct text display also depends on factors outside the function itself, including the environment's encoding, terminal settings, the operating system locale, fonts, and console configuration. Therefore, simply using `std::println` does not guarantee that all environments will display Persian text or other non-ASCII characters in the same way.

For file output, the encoding expected by the file's consumer must also be considered. The function does not provide a general, portable policy for converting arbitrary content to an encoding specific to your application.

## Concurrency and Multithreading

The standard printing functions use internal library mechanisms to prevent output from becoming interleaved in common scenarios. However, this should not be taken to mean that multiple independent calls to `std::println` form a single transaction or establish a deterministic ordering among threads.

If multiple threads print messages simultaneously, message ordering depends on scheduling and the implementation. To write a multi-part record that must be observed as one logical unit, it is better to first construct the complete text with `std::format` and then write it using an appropriate synchronization mechanism, or coordinate access to the output with a mutex at the application level.

```cpp
#include <format>
#include <mutex>
#include <print>
#include <string>

std::mutex output_mutex;

void log_message(int thread_id, int event_id) {
    std::string message =
        std::format("thread={}, event={}", thread_id, event_id);

    std::lock_guard<std::mutex> lock(output_mutex);
    std::println("{}", message);
}
```

In this example, application-level locking controls access to the output within the protected section. If not all parts of the program follow the same policy, interleaving with output from other parts of the program may still occur.

## Errors and Exceptions

A call to `std::println` may throw exceptions under various conditions:

- If memory allocation fails, `std::bad_alloc` may be thrown.
- If a write operation on the stream fails, `std::system_error` may be thrown.
- If the formatter being used throws an exception, that exception is propagated to the caller; for example, `std::format_error` in formatting-related cases.

Therefore, if output is a critical part of an operation, an error-handling policy should be defined. In simple CLI code, allowing an exception to propagate to a higher layer may be sufficient. In sensitive systems or long-running services, however, output failures should be handled deliberately.

Also, `std::println` does not return a success value or the number of bytes written. If an API with a more precise contract for handling write status is required, other I/O options and an error policy appropriate to the application should be considered.

## Comparison with std::cout and printf

### Comparison with std::cout

`std::cout` is based on streams and a chain of `operator<<` operations. This approach is suitable for inserting different types into a stream, working with custom streams, and integrating with existing iostream functionality.

In contrast, `std::println` receives the format string and arguments as a single, explicit unit. This style is often more readable for status messages, CLI output, and simple logs. However, it is not a complete replacement for iostreams; `std::cout` remains useful for input, custom streams, and code that relies heavily on stream behavior.

### Comparison with printf

`printf` uses C format strings and variadic arguments. This API remains common in C projects and at C ABI boundaries, but matching argument types with format specifiers is not type-safe in the same way as with `std::format_string`.

`std::println` uses the modern formatting rules of the C++ library and can be extended with custom formatters. In a new C++ project, it is generally a more suitable choice for formatted output, unless compatibility with C APIs, environmental constraints, or a specific library requirement makes `printf` necessary.

| Feature | `std::println` | `std::cout` | `printf` |
|---|---|---|---|
| Formatting | Based on `std::format` | Based on `operator<<` and manipulators | Based on C format specifiers |
| Compile-time checking of fixed format strings | Yes, for valid format strings | Depends on operator overloads | No, not in a type-safe manner |
| Automatic newline | Yes | No | No |
| Custom formatters | Through `std::formatter` | Through an appropriate overload or inserter | Limited to C conventions |
| Output destination | `stdout` or `std::FILE*` in the `<print>` overloads | A stream object | `stdout` or `FILE*` |
| Standard | C++23 | Early versions of C++ | C; also usable in C++ |

## Differences Between the <ostream> Header Overloads

In the C++23 standard library, formatted printing capabilities for `std::ostream` are also provided. These APIs are not the same as the `<print>` header overloads whose destinations are `stdout` or `std::FILE*`.

For example, the `std::println` overload for `std::ostream` is available through the `<ostream>` header:

```cpp
#include <ostream>
#include <sstream>
#include <string>

int main() {
    std::ostringstream output;
    std::println(output, "Value: {}", 42);

    std::string result = output.str();
}
```

This overload is useful for writing to C++ streams. Therefore, when checking documentation or investigating a compilation error, pay attention to the header and destination type: `<print>` provides the printing family based on `stdout` and `std::FILE`, while `<ostream>` provides the stream-oriented overloads.

## Standard Version and Compiler Support

`std::println` was standardized in C++23, but practical support depends on the compiler version and, more importantly, the version of the Standard Library. A compiler may recognize C++23 mode while the accompanying library still does not fully provide the `<print>` header.

For example, standard mode is typically enabled in GCC with `-std=c++23`, and in Clang with `-std=c++23` as well. In MSVC, `/std:c++latest` or the standard-version option corresponding to the toolset can be used; the exact details depend on the tool version.

Example commands:

```bash
g++ -std=c++23 -Wall -Wextra -pedantic main.cpp -o app
clang++ -std=c++23 -Wall -Wextra -pedantic main.cpp -o app
```

For MSVC:

```bat
cl /std:c++latest /EHsc main.cpp
```

The presence of a standard-version option is not sufficient. If the compiler cannot find `<print>` or does not provide `std::println`, check the version of the standard library and the support table for the toolchain in use.

### Checking the Feature-Test Macro

The following macro can be used to detect library support:

```cpp
#include <version>

#if defined(__cpp_lib_print) && __cpp_lib_print >= 202207L
    #define HAS_STD_PRINT 1
#else
    #define HAS_STD_PRINT 0
#endif
```

The value `202207L` identifies the formatted-output capability standardized for C++23. Library feature-test macros provide a more precise way to detect the presence of a specific feature than relying on the compiler version number.

For code that must be built with a variety of toolchains, it is best to check the feature-test macro alongside the build system's standard setting and an actual compilation test.

## Limitations and Design Considerations

### This Function Is Not a Complete Logging Tool

`std::println` simply writes formatted text to a destination. It does not provide features such as log levels, file rotation, automatic timestamps, structured events, message filtering, or asynchronous delivery.

Direct use is appropriate for small programs, command-line tools, or simple diagnostic messages. For a large service or a multithreaded application with serious operational requirements, a dedicated logging library is a more suitable choice.

### Choose the Output Destination Using the Appropriate Type

The overload without a stream writes to `stdout`. If the destination must be a specific file or stream, choose the overload taking `std::FILE*` or the API for `std::ostream`, according to the destination type. A `std::ostream*` must not be passed in place of a `std::FILE*` to the `<print>` header overload; these are different types and belong to different API families.

### Do Not Append an Extra Newline

Because `std::println` appends a `'\n'` by itself, adding a newline to the end of the format string can produce an unintended blank line. If more precise control over line termination is required, `std::print` is a more suitable option.

### The Format String Must Match the Arguments

With a constant format string, an invalid replacement field or an incompatible number of arguments will generally cause a compilation error. This is an important benefit, but when migrating from `printf`, the old format specifiers must be reviewed; `%d` and `{}` do not follow the same rules.

### Function Output Is Not Necessarily Immediately Committed to Stable Storage

Appending `'\n'` does not guarantee that data is flushed to disk or even that it is immediately persistent on the storage medium. If durability or flush ordering matters, an appropriate API and buffering policy must be used. A newline and a flush are two separate concepts.

## Practical Example

The following example combines several common capabilities in a command-line program:

```cpp
#include <print>
#include <string>
#include <vector>

struct Task {
    int id;
    std::string title;
    bool completed;
};

int main() {
    const std::vector<Task> tasks{
        {1, "Review pull request", true},
        {2, "Update documentation", false},
        {3, "Run test suite", true}
    };

    std::println("Task report");
    std::println("{:-<48}", "");

    for (const Task& task : tasks) {
        const char* status = task.completed ? "done" : "pending";
        std::println("#{:<3} {:<28} [{}]", task.id, task.title, status);
    }

    std::println("{:-<48}", "");
    std::println("Total tasks: {}", tasks.size());
}
```

In this program, formatting controls the column widths, and each call to `std::println` produces a separate line. This pattern is useful for simple text reports in CLI tools.

## Common Mistakes

- **Forgetting the header:** The `<print>` header must be included for the primary overloads.
- **Assuming support based only on `-std=c++23`:** The language mode and standard library implementation are separate concerns.
- **Using `printf` formatting rules:** Specifiers such as `%d` have no meaning in a `std::println` format string.
- **Adding an unnecessary `'\n'`:** This can create unwanted blank space between messages.
- **Confusing `std::FILE*` with `std::ostream`:** The appropriate overload and header must be selected for each type.
- **Assuming a deterministic order between threads:** Concurrent calls do not provide an application-level guarantee of an overall message order.
- **Assuming that a newline flushes output:** `'\n'` is not equivalent to a flush or a guarantee of durable storage.
- **Ignoring exceptions:** A formatter error or a write failure may be propagated to the caller as an exception.

## Summary

`std::println` in C++23 provides a standard, readable, and type-safe way to print formatted text followed by a newline. It is built on the `std::format` framework and is a suitable choice for CLI output, status messages, text reports, and similar use cases.

To use it correctly, pay attention to three points: choose the header and overload appropriate for the output destination, check actual Standard Library support separately from the language mode, and define synchronization and exception-handling policies at the application level for multithreaded or error-sensitive use cases. `std::println` is not a replacement for every I/O or logging tool, but it is a simple and effective choice for many formatted-output needs in modern C++ programs.

## References and Further Reading

- [cppreference: `std::println`](https://en.cppreference.com/w/cpp/io/println)
- [cppreference: `std::print`](https://en.cppreference.com/w/cpp/io/print)
- [cppreference: `std::println` for `std::ostream`](https://en.cppreference.com/w/cpp/io/basic_ostream/println)
- [cppreference: Feature-test macros](https://en.cppreference.com/w/cpp/feature_test)
- [WG21 P2093R14: Formatted output](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2093r14.html)


---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>