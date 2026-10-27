<div align="center">

[🇺🇸 English](./std_source_location.md) | [🇮🇷 فارسی](../../fa/cpp20/std_source_location.md)

</div>

---

# Introduction to `std::source_location` in C++20

The `std::source_location` feature, introduced in the C++20 standard, provides a standardized way to access information about the location of code in a source file. It allows developers to obtain information such as the file name, line number, function name, and source location at runtime without manually passing this information to functions or using custom macros for every call.

One of its most important applications is developing logging systems, recording errors, building diagnostic tools, and improving code maintainability. For example, a logging function can automatically identify the location from which it was called. This eliminates the need for developers to manually specify the file name and line number when recording messages.

## Understanding `std::source_location`

In many C++ programs, knowing the exact location of an event is essential for debugging. For example, when a function records an error, it may be necessary to determine which file, line, and function caused the message to be generated.

Before C++20, a common approach was to use the following predefined macros:

- `__FILE__` to obtain the source file name.
- `__LINE__` to obtain the current line number.
- `__func__` to obtain the current function name.
- `__DATE__` and `__TIME__` to obtain the date and time of compilation. These provide compilation-time information, not the location of a function call.

Using these macros typically required designing custom interfaces and could introduce code complexity, macro dependencies, and maintenance difficulties.

Furthermore, if a regular function is called with `__FILE__` and `__LINE__`, the caller's location is passed to the function. However, if these values are evaluated inside the function's own body, they generally identify the location in the function's implementation rather than the location from which it was called.

The `std::source_location` feature solves this problem by providing a standard C++ library type.

### The Basic Structure of This Feature

The `std::source_location` type is defined in the following standard header:

```
#include <source_location>
```

This type represents information about a location in source code. Its public interface includes the following member functions:

| Member function   | Purpose                                                                     |
| ----------------- | --------------------------------------------------------------------------- |
| `line()`          | Returns the source line number.                                             |
| `column()`        | Returns the source column number, where supported by the implementation.    |
| `file_name()`     | Returns the source file name.                                               |
| `function_name()` | Returns the function name associated with the recorded location.            |
| `current()`       | Creates an object containing information about the current source location. |

The first four functions expose the information stored in the object as readable values. The `current()` function captures information about a source location, and its behavior depends on where the call appears in the program.

It is important to note that `std::source_location` is a standard library feature introduced in C++20. Therefore, the compiler and standard library implementation must provide the necessary support before it can be used.

## Understanding How `std::source_location` Works

The key feature of this facility is its ability to capture a source location when `current()` is used. Understanding this behavior is essential for using it correctly.

### A Simple Example of Retrieving a Source Location

The following example demonstrates how to retrieve information about the current source location:

```
#include <iostream>
#include <source_location>

int main() {
    const auto location = std::source_location::current();

    std::cout << "File: " << location.file_name() << '\n';
    std::cout << "Line: " << location.line() << '\n';
    std::cout << "Column: " << location.column() << '\n';
    std::cout << "Function: " << location.function_name() << '\n';
}
```

In this example, the call to `std::source_location::current()` appears inside `main`. Therefore, the captured information corresponds to that call's location in the source file.

The program might produce output similar to the following:

```
File: main.cpp
Line: 6
Column: 48
Function: int main()
```

The exact output depends on the compiler, the way the function is called, and the standard library implementation. For example, the format of the function name may differ between compilers, and the file name may be a relative or absolute path.

The value returned by `column()` is also subject to implementation support. Therefore, applications should not assume that every target environment provides an identical or fully reliable column value without verifying the behavior of the relevant toolchain.

## The Important Role of Default Function Parameters

One of the most important applications of `std::source_location` is using it as a default function parameter. When the caller does not provide an argument for the source location, the default argument is evaluated at the function call site.

This behavior allows a function to record the caller's location rather than its own definition or implementation location.

### Example: Automatically Recording the Call Site

```
#include <iostream>
#include <source_location>
#include <string_view>

void log_message(
    std::string_view message,
    const std::source_location& location =
        std::source_location::current()
) {
    std::cout
        << location.file_name()
        << ':'
        << location.line()
        << " in "
        << location.function_name()
        << " -> "
        << message
        << '\n';
}

void process_data() {
    log_message("Processing started");
}

int main() {
    log_message("Application started");
    process_data();
}
```

In this program, `log_message` is called from two different locations. In both cases, the default value of the `location` parameter is evaluated at the call site of `log_message`.

Consequently, the first log entry refers to the call in `main`, while the second refers to the call in `process_data`.

This distinction is the foundation of using `std::source_location` in logging systems.

### Why Should the Parameter Have a Default Value?

If the function is defined as follows, its behavior is different:

```
void log_message(std::string_view message) {
    const auto location = std::source_location::current();

    std::cout << location.file_name()
              << ':' << location.line()
              << " -> " << message << '\n';
}
```

In this case, `current()` is called inside the function body. Therefore, the captured information corresponds to the location of that call within the implementation of `log_message`, rather than the location from which other code calls `log_message`.

This is one of the most common mistakes when using `std::source_location`.

As a practical rule, if the goal is to record the caller's location, `std::source_location::current()` should generally be used as a default function argument.

## Examining Member Functions and Available Information

The `std::source_location` type has a small interface, but each of its member functions serves a specific purpose in diagnostic tools.

### Retrieving the Line Number with `line()`

The `line()` function returns the line number associated with the recorded source location.

```
#include <iostream>
#include <source_location>

void inspect_location(
    const std::source_location& location =
        std::source_location::current()
) {
    std::cout << "Line: " << location.line() << '\n';
}

int main() {
    inspect_location();
}
```

This value is useful for associating error messages with specific lines of code and investigating lengthy logs.

Keep in mind that line numbers depend on the source file's contents and the preprocessing process. Changes to file lines or the use of the `#line` preprocessing directive can affect the location information reported by the compiler.

### Retrieving the File Name with `file_name()`

The `file_name()` function returns the name of the source file associated with the recorded location.

```
#include <iostream>
#include <source_location>

void print_file(
    const std::source_location& location =
        std::source_location::current()
) {
    std::cout << location.file_name() << '\n';
}

int main() {
    print_file();
}
```

The returned name may contain an absolute path, a relative path, or just the file name. The standard does not guarantee a uniform path format across different compilers and operating systems.

For projects that generate logs in different environments, you should not assume that the file name will always have the same representation.

Furthermore, if absolute source paths appear in program output, they may expose internal development environments or sensitive build paths. In such cases, paths should be appropriately restricted or sanitized before logs are published.

### Retrieving the Function Name with `function_name()`

The `function_name()` function provides the name of the function associated with the recorded location.

```
#include <iostream>
#include <source_location>

void inspect(
    const std::source_location& location =
        std::source_location::current()
) {
    std::cout << location.function_name() << '\n';
}

int main() {
    inspect();
}
```

The returned value is useful for diagnostic tools, but its format is implementation-dependent. For example, the string may include the return type, parameters, namespace information, or other details of the function signature.

Therefore, `function_name()` should not be treated as a portable API for parsing function names or extracting their exact signatures.

If an application requires standardized names for logging, database storage, or communication between services, it is better to define explicit and stable identifiers.

### Retrieving the Column Number with `column()`

The `column()` function returns the source column number associated with the recorded location.

```
#include <iostream>
#include <source_location>

void print_column(
    const std::source_location& location =
        std::source_location::current()
) {
    std::cout << location.column() << '\n';
}

int main() {
    print_column();
}
```

This feature can be useful in code analysis tools, more precise error reporting, and development environments. However, support and the availability of column information depend on the implementation, and reliable column data may not be available in every environment.

Applications that rely on column numbers as part of their output contract should test the behavior of their target compilers separately.

## Designing Functions with `std::source_location`

Professional use of this feature requires attention to function interface design. The parameter's position, type, and passing convention can affect readability and API behavior.

### Choosing an Appropriate Parameter Type

Because `std::source_location` is a small type with a simple interface, it can generally be passed either by value or by constant reference.

A common interface uses a constant reference:

```
#include <iostream>
#include <source_location>
#include <string_view>

void report(
    std::string_view message,
    const std::source_location& location =
        std::source_location::current()
) {
    std::cout << location.file_name()
              << ':' << location.line()
              << ": " << message << '\n';
}
```

This interface is readable and avoids unnecessary copying at the API level. However, the actual cost of copying a `std::source_location` object depends on the implementation, and excessive optimization should generally be avoided without measurement.

Using a constant reference also does not mean that the source-location object must outlive the function call. The default argument provides a valid value for the call, and the function can use it throughout that invocation.

### Choosing the Parameter's Position in the Function Signature

If a `std::source_location` parameter has a default argument, all subsequent ordinary parameters must also have default arguments.

For example, the following declaration is invalid:

```
void report(
    std::string_view message,
    const std::source_location& location =
        std::source_location::current(),
    int level
);
```

Here, `level` has no default argument but appears after a parameter that does. This ordering is not permitted in an ordinary function declaration.

Instead, the source-location parameter can be placed at the end of the parameter list:

```
#include <source_location>
#include <string_view>

void report(
    std::string_view message,
    int level,
    const std::source_location& location =
        std::source_location::current()
);
```

This design places the required parameters before the optional source-location parameter and keeps calls straightforward.

### Using Default Arguments in Public APIs

For public library functions, adding a `std::source_location` parameter can provide valuable diagnostic information. However, this capability should be considered part of the interface design rather than merely an implementation detail.

If a function is part of a stable public API, consider whether adding a parameter will remain compatible with previous versions. Changing a function's signature can affect ABI compatibility, function pointers, overload resolution, and consumer code.

In widely used libraries, incorporating this capability into the initial API design can avoid incompatible changes later.

## Behavior in Template Functions and Generic Programming

`std::source_location` can also be used in template functions. In these cases, it is important to distinguish between the template's definition location, its instantiation location, and the function's call site.

### An Example of a Template Function

```
#include <iostream>
#include <source_location>

template <typename T>
void inspect_value(
    const T& value,
    const std::source_location& location =
        std::source_location::current()
) {
    std::cout << "Value: " << value << '\n';
    std::cout << "File: " << location.file_name() << '\n';
    std::cout << "Line: " << location.line() << '\n';
}

int main() {
    inspect_value(42);
    inspect_value(3.14);
}
```

In this example, the default source-location argument corresponds to the location where the template function is called. The captured information is not automatically replaced with the template definition's location or the line where the compiler generates a particular template specialization.

This behavior is useful in generic programming libraries because it allows the implementation of a general-purpose utility to access information about where it is used.

### Using `std::source_location` in Template Classes

The same approach works with member functions of class templates:

```
#include <iostream>
#include <source_location>

template <typename T>
class Container {
public:
    void inspect(
        const std::source_location& location =
            std::source_location::current()
    ) const {
        std::cout << "Type-specific operation\n";
        std::cout << location.file_name() << ':'
                  << location.line() << '\n';
    }
};

int main() {
    Container<int> first;
    Container<double> second;

    first.inspect();
    second.inspect();
}
```

Here, the source-location information is captured at the member function's call site. This can be useful when investigating how template-based APIs are used and diagnosing unexpected behavior.

## Interaction with Lambdas and Recursive Functions

In modern C++, lambdas and recursive functions are important parts of program design. `std::source_location` can be used with these constructs as well, but the intended location must be clearly defined.

### Using `std::source_location` in a Lambda

```
#include <iostream>
#include <source_location>

int main() {
    auto report = [](
        const std::source_location& location =
            std::source_location::current()
    ) {
        std::cout << location.file_name() << ':'
                  << location.line() << '\n';
    };

    report();
}
```

In this example, the default argument for the lambda parameter is evaluated at the call site. Therefore, the captured information corresponds to the location of `report()`, rather than simply the location where the lambda body was defined.

This capability is useful for callbacks and diagnostic utilities that are invoked from multiple locations.

### Propagating Source Locations in Recursive Functions

In a recursive function, if the goal is to record where an operation began, it is better to capture the source location once at the function's entry point and propagate it through recursive calls when necessary.

If a new source location is captured at every stage, the resulting logs may point to an internal or recursive call site rather than the location where the original operation began.

This issue is primarily about how information flows through the program, rather than a limitation of `std::source_location` itself.

## Important Considerations for Lifetime and Storage

One consideration when using `std::source_location` is the distinction between storing the object itself and storing pointers or strings obtained from it.

### Storing a Source-Location Object

A `std::source_location` object can be stored in a variable and used later:

```
#include <iostream>
#include <source_location>

std::source_location saved_location;

void capture_location(
    const std::source_location& location =
        std::source_location::current()
) {
    saved_location = location;
}

int main() {
    capture_location();

    std::cout << saved_location.file_name() << ':'
              << saved_location.line() << '\n';
}
```

In this example, the source-location object remains usable after `capture_location` returns. This differs from storing a reference to a local parameter, because the object itself is stored.

In concurrent applications, if multiple threads can read and write the shared `saved_location` variable without synchronization, the usual C++ concurrency rules and data-race requirements must be observed.

### Storing Source-Location Strings

The `file_name()` and `function_name()` functions return `const char*` pointers. These strings should not be retained as raw pointers for long-term storage without considering their lifetime guarantees.

For example, if location information must be stored in an asynchronous logging queue or processed after the current operation completes, the strings can be copied into `std::string` objects.

```
#include <source_location>
#include <string>

struct SourceInfo {
    std::string file;
    std::string function;
    unsigned int line;
    unsigned int column;
};

SourceInfo copy_source_info(
    const std::source_location& location
) {
    return {
        location.file_name(),
        location.function_name(),
        location.line(),
        location.column()
    };
}
```

This structure stores the information in owning strings and is better suited to logging queues, event storage, and data transfer between threads.

## Limitations and Design Considerations

Although `std::source_location` simplifies access to source-location information, its limitations should be considered in professional software design.

### Implementation Dependence

The file name, function-name format, and details of column information may vary between compilers and build environments. Therefore, the output should not be assumed to follow a fixed, portable textual format without an additional standardization layer.

### No Stack Trace Information

This type provides information about a single source location. By itself, `std::source_location` does not provide a stack trace, a call chain, thread state, or the execution history of a program.

If the goal is to inspect the full call chain, additional tools are required, such as a debugger, operating-system-specific facilities, or an appropriate stack-tracing library.

### No Guarantee of Identical Behavior Across All Environments

If an application is built in multiple environments, support for the standard `<source_location>` header and compiler behavior should be verified. Simply selecting C++20 mode in a compiler does not, by itself, guarantee that every component of the standard library is fully available.

### Security Considerations

Source file paths may expose internal development environments or build infrastructure. For example, absolute paths can reveal user account names, directory structures, or internal CI paths.

If logs are sent to end users or stored in external services, file paths should be reviewed or sanitized according to the project's security policy.

### Performance Considerations

The cost of using `std::source_location` alone is generally not a sufficient reason to avoid it. However, the total cost of logging can depend on string formatting, locks, I/O operations, and data transfer.

In performance-critical code paths, it is better to measure the actual cost of the system rather than unnecessarily remove diagnostic information. If needed, conditional logging, asynchronous recording, or lazy formatting can be used.

### Limitations in Reconstructing Portable Names

The value returned by `function_name()` is suitable for display and debugging, but it does not provide a portable contract for converting the result into a unique function identifier. Using this value as a database key, a stable API identifier, or a means of determining function identity can lead to compatibility problems.

For these use cases, explicit and controlled identifiers are a more reliable choice.

## Important Practices When Using `std::source_location`

In real-world projects, following a few principles can help prevent most common mistakes.

- To record the caller's location, generally use `std::source_location::current()` as a default function argument.
- In wrapper functions, propagate the source-location object to internal functions if preserving the original call site is important.
- Do not assume that the format of `function_name()` or the path returned by `file_name()` is compiler-independent.
- For long-term storage or asynchronous transfer, copy the string information when necessary.
- In public APIs, consider signature compatibility, overloads, and ABI implications when adding a source-location parameter.
- In logging systems, combine `std::source_location` with severity levels, event timestamps, and appropriate identifiers.
- In sensitive environments, review or sanitize file paths before publishing logs.
- If stack traces or a complete execution history are required, use an appropriate complementary tool.

## Final Summary

The `std::source_location` feature introduced in C++20 provides developers with a standard and readable way to retrieve information about source-code locations. Its main advantage is that it allows the caller's location to be recorded without requiring custom macros or manually passing information such as file names and line numbers.

Using a default parameter initialized with `std::source_location::current()` is particularly useful in logging systems, validation functions, diagnostic tools, and public libraries.

However, developers must consider the distinction between the function's call site and the location where `current()` is evaluated, implementation-dependent output formats, the limitations of string data, and security and performance requirements.

Overall, for projects using C++20 or later, `std::source_location` is a useful standard tool for improving debugging and code maintainability, especially when source-location information needs to be available in a straightforward way, propagated across application layers, and obtained without unnecessary dependence on custom macros.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>