<div align="center">

[🇺🇸 English](./std_print.md) | [🇮🇷 فارسی](../../fa/cpp23/std_print.md)

</div>

---

# **A Comprehensive Guide to std::print in C++23**

## **Introduction**

`std::print` is an important feature introduced in the C++23 standard library that provides a modern, simple, and type-safe way to print formatted text.

Before this feature was introduced, C++ developers generally relied on two main approaches for producing output:

- Using `std::cout` and the `operator<<` insertion operator through `<iostream>`.
- Using C functions such as `printf` and `fprintf` through `<cstdio>`.

Both approaches have their own use cases, but they also have limitations. For example, chaining multiple `operator<<` expressions can reduce code readability. On the other hand, the `printf` family relies on format specifiers whose incorrect matching with argument types can result in undefined behavior.

`std::print` addresses many of these issues by using the formatting infrastructure introduced in C++20.

The most important characteristics of this feature include:

- Support for type-safe formatting.
- Use of standard format strings.
- Direct output to `stdout` and `stderr`.
- Support for `std::ostream`.
- Compatibility with custom formatters.
- Support for numeric, textual, and structured formatting.
- Reduced complexity when writing output-related code.

An important distinction is that `std::print` is a standard library feature, not a new language feature in C++. Therefore, using it requires not only C++23 compiler support but also a standard library implementation that provides this functionality.

## **Introduction to std::print**

### **Understanding std::print**

The `std::print` function is used to print formatted data to an output destination.

It is declared in the standard `<print>` header.

The simplest example is:

```
#include <print>

int main() {
    std::print("Hello, World!\n");
}
```

Output:

```
Hello, World!
```

Compared with `std::cout`, this approach provides a simpler structure when printing constant text or combining text with variable values.

```
#include <iostream>
#include <print>

int main() {
    std::string name = "Alice";
    int age = 25;

    std::cout << "Name: " << name
              << ", Age: " << age << '\n';

    std::print("Name: {}, Age: {}\n", name, age);
}
```

In the second example, there is no need to chain multiple output operators. The order of the arguments is determined by the placeholders in the format string.

### **Difference Between std::print and std::println**

C++23 introduces two main functions for formatted output:

- `std::print`
- `std::println`

Their primary difference is whether they automatically append a newline.

The `std::print` function does not automatically append a newline after printing its content.

```
#include <print>

int main() {
    std::print("Hello ");
    std::print("World");
    std::print("\n");
}
```

Output:

```
Hello World
```

In contrast, `std::println` appends a newline after printing its content.

```
#include <print>

int main() {
    std::println("Hello");
    std::println("World");
}
```

Output:

```
Hello
World
```

Therefore:

| Function       | Behavior                                         |
| -------------- | ------------------------------------------------ |
| `std::print`   | Prints without automatically appending a newline |
| `std::println` | Prints and appends a newline                     |

For printing messages that should each occupy a separate line, `std::println` is generally the more convenient choice.

Version note: The no-argument overload `std::println()`, which only prints a newline, belongs to C++26. In C++23, use `std::println("")` to print an empty line.

## **Understanding Format Strings**

### **The Concept of Placeholders**

One of the most important aspects of `std::print` is its use of format strings.

In this syntax, `{}` acts as a placeholder for an argument value.

```
#include <print>

int main() {
    std::print("Name: {}, Age: {}\n", "Alice", 25);
}
```

Output:

```
Name: Alice, Age: 25
```

In this example:

- The first `{}` is replaced with `"Alice"`.
- The second `{}` is replaced with `25`.

By default, arguments are matched to placeholders according to their order of appearance.

### **Using Argument Indexing**

Instead of automatic indexing, you can explicitly specify argument indices in a format string.

```
#include <print>

int main() {
    std::print("{1} {0}\n", "World", "Hello");
}
```

Output:

```
Hello World
```

In this example:

- `{0}` refers to the first argument.
- `{1}` refers to the second argument.

This feature is useful when the display order differs from the order in which arguments are passed.

You can also reference the same argument multiple times:

```
#include <print>

int main() {
    std::print("{0}, {0}, {1}\n", "Hello", "World");
}
```

Output:

```
Hello, Hello, World
```

Automatic indexing and explicit indexing cannot be mixed within the same format string.

For example, the following format string is invalid:

```
#include <print>

int main() {
    // Invalid: automatic and manual indexing are mixed.
    // std::print("{} {1}\n", "Hello", "World");
}
```

### **Printing Literal Braces**

Sometimes you need to display the actual `{` and `}` characters in the output.

To do this, use doubled braces.

```
#include <print>

int main() {
    std::print("{{Hello}}\n");
}
```

Output:

```
{Hello}
```

Therefore:

| Format syntax | Result            |
| ------------- | ----------------- |
| `{}`          | Placeholder       |
| `{{`          | The `{` character |
| `}}`          | The `}` character |

This feature is useful when printing structures such as JSON, code snippets, and text containing braces.

## **Formatting Numeric Values**

One of the important features of format strings is the ability to control how numbers are displayed.

This functionality uses the `std::format` rules introduced in C++20.

### **Formatting Integer Values**

You can specify the representation of an integer using format specifiers.

```
#include <print>

int main() {
    int value = 255;

    std::println("Decimal: {}", value);
    std::println("Binary: {:b}", value);
    std::println("Octal: {:o}", value);
    std::println("Hexadecimal: {:x}", value);
    std::println("Uppercase Hex: {:X}", value);
}
```

Output:

```
Decimal: 255
Binary: 11111111
Octal: 377
Hexadecimal: ff
Uppercase Hex: FF
```

You can also use `#` to include the appropriate base prefix.

```
#include <print>

int main() {
    int value = 255;

    std::println("{:#b}", value);
    std::println("{:#o}", value);
    std::println("{:#x}", value);
}
```

Output:

```
0b11111111
0377
0xff
```

### **Specifying Output Width**

You can specify the minimum output field width by placing a number after `:`.

```
#include <print>

int main() {
    int value = 42;

    std::println("|{:5}|", value);
    std::println("|{:10}|", value);
}
```

Output:

```
|   42|
|        42|
```

By default, the value is right-aligned.

You can specify alignment using the following characters:

| Specifier | Meaning          |
| --------- | ---------------- |
| `<`       | Left alignment   |
| `>`       | Right alignment  |
| `^`       | Center alignment |

Example:

```
#include <print>

int main() {
    std::println("|{:<10}|", 42);
    std::println("|{:>10}|", 42);
    std::println("|{:^10}|", 42);
}
```

Output:

```
|42        |
|        42|
|    42    |
```

### **Using a Fill Character**

In addition to alignment, you can specify the character used to fill the remaining field width.

```
#include <print>

int main() {
    std::println("{:*<10}", 42);
    std::println("{:*>10}", 42);
    std::println("{:*^10}", 42);
}
```

Output:

```
42********
********42
****42****
```

This feature is useful for generating simple tables and textual reports.

### **Zero-Padding Numbers**

You can add zeros to the beginning of numbers using zero padding.

```
#include <print>

int main() {
    int number = 42;

    std::println("{:05}", number);
    std::println("{:08}", number);
}
```

Output:

```
00042
00000042
```

This technique is useful for displaying identifiers, sequence numbers, and values with fixed-width representations.

## **Formatting Floating-Point Numbers**

### **Specifying Precision**

You can control the number of decimal digits using precision.

```
#include <print>

int main() {
    double value = 3.1415926535;

    std::println("{:.2f}", value);
    std::println("{:.4f}", value);
    std::println("{:.6f}", value);
}
```

Output:

```
3.14
3.1416
3.141593
```

In this syntax, `f` indicates fixed-point notation.

### **Using Scientific Notation**

You can display numbers in scientific notation using `e` or `E`.

```
#include <print>

int main() {
    double value = 123456.789;

    std::println("{:e}", value);
    std::println("{:E}", value);
    std::println("{:.2e}", value);
}
```

### **Displaying Numbers in General Format**

You can use `g` or `G` for general floating-point formatting.

```
#include <print>

int main() {
    double value = 123456.789;

    std::println("{:g}", value);
    std::println("{:.4g}", value);
}
```

This format selects between fixed-point and scientific notation depending on the value.

### **Formatting Numeric Signs**

You can control how positive and negative signs are displayed using different sign options.

```
#include <print>

int main() {
    std::println("{:+}", 42);
    std::println("{:+}", -42);
    std::println("{: }", 42);
}
```

The `+` sign causes positive values to be displayed with an explicit plus sign.

## **Formatting Strings**

### **Specifying String Width**

Strings also support width and alignment formatting.

```
#include <print>

int main() {
    std::println("|{:10}|", "Alice");
    std::println("|{:<10}|", "Alice");
    std::println("|{:^10}|", "Alice");
}
```

### **Limiting String Length**

You can use precision to limit the number of characters displayed.

```
#include <print>

int main() {
    std::println("{:.5}", "Programming");
    std::println("{:.8}", "Programming");
}
```

Output:

```
Progr
Programm
```

This feature is useful for displaying shortened text in tables and reports.

## **Printing Different Standard Types**

`std::print` supports standard formatters for various data types.

### **Printing bool**

```
#include <print>

int main() {
    bool enabled = true;

    std::println("{}", enabled);
    std::println("{:d}", enabled);
}
```

Output:

```
true
1
```

By default, Boolean values are displayed as `true` or `false`.

### **Printing Characters**

```
#include <print>

int main() {
    char letter = 'A';

    std::println("{}", letter);
    std::println("{:d}", letter);
}
```

Output:

```
A
65
```

With numeric formatting, the numeric value corresponding to the character is displayed.

### **Printing Pointers**

You can use pointer formatting to display a pointer value.

```
#include <print>

int main() {
    int value = 42;
    int* pointer = &value;

    std::println("Pointer: {}", static_cast<const void*>(pointer));
}
```

The exact representation of an address depends on the implementation and execution environment.

### **Printing nullptr**

You can also use the formatter for null pointer values.

```
#include <print>

int main() {
    std::nullptr_t pointer = nullptr;

    std::println("{}", pointer);
}
```

### **Printing Enums**

When formatting enums, you should consider their underlying types and how their values are represented. If you need numeric output, an explicit conversion is generally clearer.

```
#include <print>

enum class Status : int {
    Pending = 0,
    Active = 1,
    Finished = 2
};

int main() {
    Status status = Status::Active;

    std::println("Status: {}", static_cast<int>(status));
}
```

If you want to display the semantic name of an enum value, it is preferable to define a custom formatter.

## **Relationship Between std::print and std::format**

### **Difference in Output**

The `std::format` feature was introduced in C++20 to generate formatted strings.

In contrast, `std::print`, introduced in C++23, writes the formatted result directly to an output destination.

Example using `std::format`:

```
#include <format>
#include <string>

int main() {
    std::string message = std::format(
        "Name: {}, Age: {}",
        "Alice",
        25
    );
}
```

Example using `std::print`:

```
#include <print>

int main() {
    std::print("Name: {}, Age: {}\n", "Alice", 25);
}
```

The practical differences are:

| Feature                         | std::format                 | std::print |
| ------------------------------- | --------------------------- | ---------- |
| Produces a `std::string`        | Yes                         | No         |
| Prints directly                 | No                          | Yes        |
| Supports format specifiers      | Yes                         | Yes        |
| Supports custom formatters      | Yes                         | Yes        |
| Suitable for storing the result | Yes                         | No         |
| Suitable for direct output      | Requires an additional step | Yes        |

If you need the resulting string for storage, processing, or passing to another API, `std::format` is appropriate.

If your goal is simply to print output, `std::print` is generally the simpler choice.

### **Using std::vformat for Dynamic Format Strings**

Normally, `std::print` uses `std::format_string` to enable compile-time format string checking.

However, sometimes the format string is determined at runtime. For example, it might be read from a configuration file or received as program input.

In C++23, you can use `std::vformat` for this scenario.

```
#include <format>
#include <print>
#include <string>

int main() {
    std::string formatString = "Value: {}";

    std::string result = std::vformat(
        formatString,
        std::make_format_args(42)
    );

    std::println("{}", result);
}
```

In this approach, the formatted string is generated first and then printed.

An important detail is that `std::make_format_args` provides formatting arguments through a type-erased structure. This structure must not be retained beyond the lifetime of the arguments on which it depends.

Also, an invalid format string in this scenario may cause `std::format_error` to be thrown at runtime.

## **Printing to Different Destinations**

One of the important features of `std::print` is its support for different output destinations.

### **Printing to stdout**

By default, output is sent to `stdout`.

```
#include <print>

int main() {
    std::print("Application started\n");
}
```

This behavior is common in command-line applications.

### **Printing to stderr**

You can use `stderr` to print error messages and diagnostic information.

```
#include <cstdio>
#include <print>

int main() {
    std::println(stderr, "Error: File not found");
}
```

Using `stderr` makes it possible to separate error output from normal program output.

For example:

```
./app > output.txt 2> errors.txt
```

In this case, normal output and error messages are stored in separate files.

### **Printing to FILE\***

In addition to `stdout` and `stderr`, you can pass another `std::FILE*` to the function.

```
#include <cstdio>
#include <print>

int main() {
    std::FILE* file = std::fopen("output.txt", "w");

    if (!file) {
        return 1;
    }

    std::println(file, "Hello from std::print");
    std::println(file, "Value: {}", 42);

    std::fclose(file);
}
```

In this example, the output is written directly to a file.

For more reliable resource management in real-world applications, using RAII and appropriate wrappers for `FILE*` is recommended. This ensures that the file is closed even if an exception occurs.

### **Printing to std::ostream**

C++23 also provides overloads for `std::ostream` that support formatted output.

```
#include <fstream>
#include <ostream>

int main() {
    std::ofstream file("output.txt");

    if (file) {
        std::print(file, "Name: {}, Age: {}\n", "Alice", 25);
    }
}
```

This functionality makes it possible to use `std::print` with streams such as:

- `std::ofstream`
- `std::ostringstream`
- `std::stringstream`
- `std::cout`
- `std::cerr`

To use these overloads, you should also consider including the appropriate `<ostream>` header.

## **Supporting Custom Types**

One of the powerful features of the `std::format` infrastructure is the ability to define formatters for user-defined types.

Because `std::print` uses the same infrastructure, custom formatters can also be used with it.

### **Defining a Custom Formatter**

Suppose we have the following structure:

```
struct Point {
    int x;
    int y;
};
```

By default, the standard library does not know how an object of type `Point` should be displayed.

To solve this problem, we need to specialize `std::formatter`.

```
#include <format>
#include <print>
#include <string>

struct Point {
    int x;
    int y;
};

template <>
struct std::formatter<Point> : std::formatter<std::string> {
    auto format(const Point& point, std::format_context& context) const {
        return std::formatter<std::string>::format(
            std::format("({}, {})", point.x, point.y),
            context
        );
    }
};

int main() {
    Point point{10, 20};

    std::println("Position: {}", point);
}
```

Output:

```
Position: (10, 20)
```

In this example, the formatter for `Point` uses the string formatter to generate the textual representation of the object.

For this implementation, `<string>` should also be included directly. In production code, avoid relying on transitive includes.

### **Implementing a Formatter with More Control**

In the previous example, we used `std::format` to generate a temporary string.

For types that are formatted frequently, you can write directly to the output iterator.

```
#include <format>
#include <print>

struct Point {
    int x;
    int y;
};

template <>
struct std::formatter<Point> {
    constexpr auto parse(std::format_parse_context& context) {
        return context.begin();
    }

    auto format(const Point& point, std::format_context& context) const {
        return std::format_to(
            context.out(),
            "({}, {})",
            point.x,
            point.y
        );
    }
};

int main() {
    Point point{10, 20};

    std::println("Point: {}", point);
}
```

This approach avoids creating an intermediate string inside the formatter implementation.

An important requirement is that the formatter must satisfy the standard contracts for `parse` and `format`.

### **Designing Professional Formatters**

When designing formatters for custom types, consider the following principles:

- The default representation should be readable and predictable.
- A formatter should not modify the state of the object.
- Unnecessary side effects should be avoided.
- Custom format specifiers can be introduced for complex types.
- Unnecessary memory allocations should be reduced in frequently executed paths.
- Formatter behavior should be as deterministic as possible.

For example, a formatter for a financial type might support specifying the number of decimal places, while a formatter for a geometric type might provide control over coordinate representation.

## **Understanding Unicode Behavior**

One important consideration when using `std::print` is Unicode handling.

Correctly printing multilingual text does not depend solely on the output API. Source-file encoding, destination encoding, locale, and terminal capabilities are also important.

### **Understanding Encoding**

In C++, the `char` type alone does not specify that its data must be UTF-8.

Similarly, `char8_t`, introduced in C++20, is designed to represent UTF-8 code units and is a distinct type from `char`.

For example:

```
#include <print>

int main() {
    const char* text = "Hello";
    std::println("{}", text);
}
```

When working with Unicode text, you must also consider the encoding expected by the output destination.

### **std::print and Unicode**

The standard design of this feature accounts for Unicode and non-Unicode output paths.

In particular, the `FILE*` overloads use Unicode-oriented printing mechanisms so that, in supported environments, text can be written using an appropriate encoding.

However, you should not assume that `std::print` automatically converts every arbitrary encoding to UTF-8.

For reliable multilingual output:

- Save source files using an appropriate encoding, usually UTF-8.
- Check the encoding expected by the output destination.
- Understand the difference between `char` and `char8_t`.
- Verify that the terminal supports the required Unicode characters.
- Consider locale and execution-environment behavior.

Also, `std::print` is not responsible for text shaping, the bidirectional text algorithm (BiDi), or font selection.

This is particularly important for Persian text because correct display also depends on the rendering layer's support for the relevant text-processing algorithms.

## **Managing Output in Multithreaded Applications**

In multithreaded applications, simultaneous output from multiple threads can cause messages to become interleaved.

For example, consider two threads producing output simultaneously:

```
#include <print>
#include <thread>

int main() {
    std::thread first([] {
        for (int i = 0; i < 5; ++i) {
            std::println("Thread A: {}", i);
        }
    });

    std::thread second([] {
        for (int i = 0; i < 5; ++i) {
            std::println("Thread B: {}", i);
        }
    });

    first.join();
    second.join();
}
```

In this example, the overall order of output lines is not deterministic because thread scheduling is controlled by the operating system and runtime.

To prevent different parts of a message from becoming interleaved, it is preferable to construct the complete message and print it in a single call.

For more precise control, you can also use synchronization mechanisms.

```
#include <format>
#include <mutex>
#include <print>
#include <thread>

std::mutex outputMutex;

void printMessage(int id, int value) {
    std::string message = std::format(
        "Thread {}: {}\n",
        id,
        value
    );

    std::lock_guard lock{outputMutex};

    std::print("{}", message);
}

int main() {
    std::thread first(printMessage, 1, 100);
    std::thread second(printMessage, 2, 200);

    first.join();
    second.join();
}
```

In this example, the mutex ensures that threads using the same mutex do not perform their printing operations simultaneously.

An important detail is that this mutex only coordinates cooperating threads. Other code that writes to the same output destination without using the mutex is not covered by this synchronization.

You should also not assume that multiple independent `std::print` calls will print a large logical message as one atomic operation.

## **Difference Between Newline and Flush**

A common mistake is treating newline and flush as equivalent operations.

`std::println` appends a newline to the output, but this does not mean that the output buffer is necessarily flushed.

For example:

```
#include <print>

int main() {
    std::println("Application started");
}
```

The presence of a newline does not guarantee that all data is immediately delivered to the final destination.

If output must be flushed immediately, use the appropriate mechanism for the destination.

For `FILE*`, you can use `std::fflush`:

```
#include <cstdio>
#include <print>

int main() {
    std::print(stdout, "Enter command: ");

    std::fflush(stdout);
}
```

Frequent flushing can reduce application performance. Therefore, it is better to use it only when immediate output delivery is important.

## **Exception Handling**

Printing functions may throw exceptions under various conditions.

The most important possibilities include:

| Exception                | Possible cause                                       |
| ------------------------ | ---------------------------------------------------- |
| `std::bad_alloc`         | Memory allocation failure                            |
| `std::format_error`      | Formatting error                                     |
| `std::system_error`      | Failure in a system-related operation                |
| `std::ios_base::failure` | Stream operation failure under applicable conditions |

The exact exception type depends on the overload being used and the circumstances of the failure.

For example, a failed write operation in the C stream overloads can result in `std::system_error`.

For `std::ostream` overloads, error behavior depends on the stream state and its exception mask.

Example of exception handling:

```
#include <exception>
#include <print>
#include <system_error>

int main() {
    try {
        std::println("Application started");
    }
    catch (const std::system_error& error) {
        return 1;
    }
    catch (const std::exception& error) {
        return 2;
    }
}
```

In real-world applications, exceptions should generally be handled at appropriate architectural boundaries rather than creating a separate try-catch block around every printing operation.

## **Comparing std::print with printf**

`printf` is one of the oldest and most widely used output functions in C and C++.

Example:

```
#include <cstdio>

int main() {
    int age = 25;

    std::printf("Age: %d\n", age);
}
```

With `std::print`, the same output can be produced as follows:

```
#include <print>

int main() {
    int age = 25;

    std::println("Age: {}", age);
}
```

Important differences:

| Feature                 | printf            | std::print                     |
| ----------------------- | ----------------- | ------------------------------ |
| Primary language        | C                 | C++                            |
| Formatting              | Format specifiers | Format strings                 |
| Type safety             | Limited           | Stronger                       |
| Custom formatters       | No                | Yes                            |
| Direct use of C++ types | Limited           | More direct                    |
| Format string checking  | Limited           | Supports compile-time checking |
| Requires C++23          | No                | Yes                            |

One of the problems with `printf` is that incorrectly matching a format specifier with an argument type can result in undefined behavior.

For example:

```
#include <cstdio>

int main() {
    int value = 42;

    std::printf("%s\n", value);
}
```

In this example, `%s` expects a string, but the argument is an `int`.

In contrast, `std::print` uses the type system and standard formatters.

For new C++ projects, `std::print` is generally a safer and more consistent choice. However, `printf` remains useful for C interoperability and legacy code.

## **Comparing std::print with std::cout**

Both APIs are used to produce output in C++, but their design models are different.

Example using `std::cout`:

```
#include <iostream>

int main() {
    std::string name = "Alice";
    int age = 25;

    std::cout << "Name: " << name
              << ", Age: " << age << '\n';
}
```

Example using `std::print`:

```
#include <print>

int main() {
    std::string name = "Alice";
    int age = 25;

    std::println("Name: {}, Age: {}", name, age);
}
```

Practical comparison:

| Feature              | std::cout        | std::print        |
| -------------------- | ---------------- | ----------------- |
| Output model         | Stream-based     | Format-based      |
| Formatting           | Manipulators     | Format specifiers |
| Custom formatting    | `operator<<`     | `std::formatter`  |
| Stream state control | Yes              | Not directly      |
| Formatted output     | Through chaining | Directly          |
| Introduced in        | C++98            | C++23             |

### **When Is std::print More Appropriate?**

In the following situations, `std::print` is generally a better choice:

- Printing formatted text messages.
- Generating simple reports.
- Displaying variable values in output.
- Printing structured data.
- Reducing long chains of `operator<<` expressions.

### **When Is std::cout More Appropriate?**

Using iostreams is still reasonable in the following situations:

- Depending on stream state.
- Using custom manipulators.
- Implementing abstractions based on `std::ostream`.
- Using custom insertion operators.
- Working with APIs that directly accept `std::ostream`.

These approaches are not absolute competitors and can coexist within the same project.

## **Practical Applications of std::print**

### **Printing Command-Line Messages**

One of the primary applications of `std::print` is producing output in CLI applications.

```
#include <cstdio>
#include <print>

int main(int argc, char* argv[]) {
    if (argc < 2) {
        std::println(stderr, "Usage: app <filename>");
        return 1;
    }

    std::println("Processing file: {}", argv[1]);

    return 0;
}
```

In this example, the usage message is sent to `stderr`, while the processing message is sent to `stdout`.

### **Displaying Tables**

You can combine width and alignment formatting to create simple textual tables.

```
#include <print>

int main() {
    std::println("{:<15} {:>10} {:>12}", "Product", "Quantity", "Price");

    std::println("{:-<39}", "");

    std::println("{:<15} {:>10} {:>12.2f}", "Keyboard", 5, 49.99);
    std::println("{:<15} {:>10} {:>12.2f}", "Mouse", 10, 19.50);
    std::println("{:<15} {:>10} {:>12.2f}", "Monitor", 2, 199.99);
}
```

### **Displaying Structured Information**

You can standardize the textual representation of complex types by defining custom formatters.

```
#include <format>
#include <print>
#include <string>

struct User {
    int id;
    std::string name;
};

template <>
struct std::formatter<User> : std::formatter<std::string> {
    auto format(const User& user, std::format_context& context) const {
        return std::formatter<std::string>::format(
            std::format("User(id={}, name={})", user.id, user.name),
            context
        );
    }
};

int main() {
    User user{10, "Alice"};

    std::println("{}", user);
}
```

### **Using std::print for Debugging**

You can use `std::print` to quickly display values during development.

```
#include <print>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    for (std::size_t i = 0; i < values.size(); ++i) {
        std::println("values[{}] = {}", i, values[i]);
    }
}
```

However, for more complex debugging tasks, using a debugger or a logging framework is more appropriate.

## **Limitations and Important Considerations**

### **Standard Library Support Is Required**

Enabling C++23 does not automatically guarantee that `std::print` is available.

The following requirements must be satisfied:

- A compiler with appropriate C++23 support.
- A standard library implementation that provides `std::print`.
- The appropriate `<print>` header.

### **Dependence on Formatters**

`std::print` cannot format every arbitrary type without an appropriate formatter.

If a custom type does not have a suitable formatter, attempting to format it may result in a compile-time error.

### **Performance Is Not Guaranteed in Every Situation**

Although `std::print` is designed for formatted output, you should not assume that it is always faster than `std::cout` or `printf`.

Performance depends on several factors:

- Standard library implementation.
- Output destination type.
- Format string complexity.
- Argument types.
- Number of calls.
- Memory allocations.
- Synchronization overhead.
- Operating system and output device behavior.

For performance-critical applications, benchmarking in the actual target environment is essential.

### **Handling Untrusted Input**

You should not use user input directly as a format string without validation.

For example, it is better to pass user data as an argument:

```
#include <print>
#include <string>

int main() {
    std::string userInput = "Hello {}";

    std::println("User input: {}", userInput);
}
```

In this case, the user's input is printed as ordinary text.

This approach prevents the user-provided content from being interpreted as a format string.

### **Not a Replacement for Logging Frameworks**

`std::print` is designed for direct output, not for managing a complete logging system.

Large applications often require features such as:

- Log levels
- Timestamps
- Log rotation
- Multiple sinks
- Filtering
- Structured logging
- Thread-safe logging
- Asynchronous logging

For these requirements, a dedicated logging framework is generally more appropriate.

## **Checking Compiler Support**

You can use a feature-test macro to check whether the standard library supports formatted printing.

The macro is named `__cpp_lib_print`.

```
#include <version>

#if defined(__cpp_lib_print) && __cpp_lib_print >= 202207L
    #include <print>
#else
    #error "std::print is not supported by this standard library"
#endif
```

The feature-test macro value for the original formatted printing functionality in C++23 is `202207L`.

An important consideration is that support for a feature in the compiler and its standard library is not necessarily introduced at the same time.

Therefore, when checking compatibility, you should also consider the version of the standard library being used.

### **Compiling with GCC**

To enable C++23 mode in GCC, you can use the following command:

```
g++ -std=c++23 -Wall -Wextra -pedantic main.cpp -o app
```

### **Compiling with Clang**

An example compilation command for Clang is:

```
clang++ -std=c++23 -Wall -Wextra -pedantic main.cpp -o app
```

With Clang, you should also consider the selected standard library because Clang can be used with either libc++ or libstdc++.

### **Compiling with MSVC**

With MSVC, you can use the following option to select the language standard:

```
cl /std:c++latest /EHsc main.cpp
```

Actual `std::print` support depends on the installed MSVC version and its Microsoft STL implementation.

## **Best Practices for Professional Projects**

To use `std::print` correctly and maintainably, consider the following principles:

- **Prefer constant format strings:** In normal situations, prefer compile-time format strings so formatting errors can be detected earlier.
- **Choose the appropriate function:** Use `std::println` for separate lines and `std::print` for continuous output.
- **Separate normal and error output:** Send normal messages to `stdout` and error messages to `stderr`.
- **Use custom formatters:** Design readable and consistent textual representations for complex types.
- **Avoid unnecessary flushing:** Frequent buffer flushing can reduce performance.
- **Manage concurrency:** Establish a clear synchronization policy for multithreaded output.
- **Protect sensitive information:** Avoid printing sensitive data such as passwords, tokens, and cryptographic keys.
- **Measure performance:** Benchmark formatting and I/O costs in performance-critical paths.
- **Avoid transitive dependencies:** Include the headers required by the APIs you use directly.
- **Separate application logic from output:** When designing public libraries, avoid imposing direct printing behavior on callers.

## **Conclusion**

The `std::print` feature introduced in C++23 provides a standard, modern, and type-safe way to print formatted text.

By using the `std::format` infrastructure introduced in C++20, it supports format strings, custom formatters, numeric formatting, and direct output to different destinations.

Its most important advantage over older approaches is the combination of simple format strings with the capabilities of the C++ type system.

To use this feature professionally, you should understand not only its syntax but also format specifiers, custom formatters, output destinations, Unicode, exceptions, buffering, and synchronization.

Ultimately, `std::print` is a suitable choice for many direct-output scenarios in modern C++ applications. However, it is not a complete replacement for iostreams or specialized logging frameworks.

## **References**

- [cppreference - std::print](https://en.cppreference.com/w/cpp/io/print?utm_source=chatgpt.com)
- [cppreference - std::println](https://en.cppreference.com/w/cpp/io/println?utm_source=chatgpt.com)
- [cppreference - std::format](https://en.cppreference.com/w/cpp/utility/format?utm_source=chatgpt.com)
- [cppreference - std::formatter](https://en.cppreference.com/w/cpp/utility/format/formatter?utm_source=chatgpt.com)
- [cppreference - std::vformat](https://en.cppreference.com/w/cpp/utility/format/vformat?utm_source=chatgpt.com)
- [C++ Standard Draft](https://eel.is/c++draft/?utm_source=chatgpt.com)
- [P2093R14 - Formatted output](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2093r14.html?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>