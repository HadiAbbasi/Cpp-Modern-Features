<div align="center">

[🇺🇸 English](./constexpr.md) | [🇮🇷 فارسی](../../fa/cpp20/constexpr.md)

</div>

---

# `constexpr` Improvements in C++20

## Introduction

The `constexpr` feature allows the compiler to evaluate expressions and functions during translation when the required conditions are met. Its purpose is not merely to make programs faster. Compile-time evaluation can also be used to create constant values, validate inputs, generate data tables, implement generic algorithms, and detect errors before a program runs.

C++20 significantly expanded this capability. Earlier restrictions on `constexpr` function bodies were reduced, dynamic memory allocation became possible during constant evaluation, and a large part of the standard library was prepared for use in constant expressions. Alongside these changes, features such as `consteval`, `constinit`, and `std::is_constant_evaluated` give programmers more precise tools for controlling evaluation.

An important point is that `constexpr` does not mean “always executed at compile time.” A `constexpr` function can also be called at runtime. Compile-time evaluation is required only when the result is used in a context that requires a constant expression.

## Understanding Compile-Time Evaluation

### The Difference Between `constexpr` and Ordinary Execution

A `constexpr` function is a function that can be evaluated at compile time when the rules for constant expressions are satisfied. The compiler is free to evaluate other calls to the same function at runtime.

```cpp
constexpr int square(int value) {
    return value * value;
}

constexpr int compile_time_result = square(8);

int runtime_square(int value) {
    return square(value);
}
```

In this example, `compile_time_result` must be a valid constant expression, so `square(8)` is evaluated at compile time. In contrast, the call inside `runtime_square` depends on a runtime input and can be evaluated when the program runs.

### Three Different Concepts: `constexpr`, `consteval`, and `constinit`

These three keywords address related but distinct concerns:

- `constexpr` permits compile-time evaluation and, when applied to variables, requires a constant-expression initializer.
- `consteval`, introduced in C++20, defines an *immediate function*. Its valid calls must satisfy the requirements for immediate evaluation and compile-time evaluation.
- `constinit`, also introduced in C++20, guarantees that a variable with static or thread storage duration is statically initialized. It does not make the variable `const`, nor does it by itself mean that the variable's value can be used in every constant expression.

```cpp
constexpr int multiply(int a, int b) {
    return a * b;
}

consteval int compile_time_multiply(int a, int b) {
    return a * b;
}

constinit int global_counter = 0;

int main() {
    constexpr int a = multiply(3, 4);
    constexpr int b = compile_time_multiply(5, 6);

    global_counter = a + b;
}
```

In this example, `global_counter` remains modifiable after initialization. Also, `consteval` is appropriate when compile-time evaluation is part of a function's design contract, rather than merely a possible optimization.

## Major Language Changes in C++20

### Fewer Restrictions on `constexpr` Function Bodies

In earlier standards, a `constexpr` function had to have a highly restricted body. These restrictions were gradually relaxed across standard versions, and C++20 continued that trend. In C++20, a `constexpr` function can use more familiar constructs, including local variables, loops, branching, exception-handling constructs, and certain dynamic-memory operations. Actual constant evaluation must still comply with the rules for constant expressions.

```cpp
constexpr int sum_until(int limit) {
    int total = 0;

    for (int i = 0; i <= limit; ++i) {
        if (i % 2 == 0) {
            total += i;
        }
    }

    return total;
}

static_assert(sum_until(6) == 12);
```

This example uses a modifiable local variable, a loop, and a conditional. Such constructs are natural in modern `constexpr` functions; there is no need to rewrite the algorithm recursively.

### Default Initialization in `constexpr` Functions

The C++20 change associated with proposal `P1331R2` permits certain simple local variables to be declared without immediate initialization, provided that the variable is initialized along the executed path before its value is read.

```cpp
constexpr int first_positive(const int* values, int count) {
    int result;

    for (int i = 0; i < count; ++i) {
        if (values[i] > 0) {
            result = values[i];
            return result;
        }
    }

    return 0;
}

constexpr int values[] = {-3, 0, 7, 9};
static_assert(first_positive(values, 4) == 7);
```

This does not make it valid to read an uninitialized variable. If an execution path reads `result` before assigning to it, the program has undefined behavior, and that path cannot produce a valid constant evaluation. Explicit initialization is usually clearer in general-purpose code; use this feature only when the algorithm's structure justifies it.

### Using `try` and `catch` in a `constexpr` Function

C++20 allows a `constexpr` function body to contain `try` and `catch` constructs. This does not mean that every exception can be thrown during constant evaluation. The evaluated path must still obey the rules for constant expressions. In particular, a path that throws an exception during compile-time evaluation generally cannot be used to produce a valid constant expression.

```cpp
constexpr int safe_divide(int numerator, int denominator) {
    try {
        if (denominator == 0) {
            throw "division by zero";
        }

        return numerator / denominator;
    } catch (...) {
        return 0;
    }
}

static_assert(safe_divide(12, 3) == 4);
```

The call above follows the path without an exception and is valid as a constant expression. This example should not be taken to mean that throwing and catching exceptions at compile time is a general replacement for input validation. When an invalid input must be diagnosed at compile time, consider `static_assert`, `consteval`, or a design that makes invalid input impossible in the first place.

### Support for `constexpr` Virtual Functions

C++20 relaxed restrictions related to `constexpr` virtual functions, allowing them to play a role in polymorphic designs. However, not every virtual call can necessarily be evaluated at compile time. The object's dynamic type, the call path, and the rules for constant expressions still determine whether evaluation is possible.

```cpp
struct Base {
    constexpr virtual int value() const {
        return 1;
    }

    constexpr virtual ~Base() = default;
};

struct Derived : Base {
    constexpr int value() const override {
        return 2;
    }
};

constexpr int read_value(const Base& object) {
    return object.value();
}

constexpr Derived derived{};
static_assert(read_value(derived) == 2);
```

This feature can be useful for constant data models and small class hierarchies, but it does not automatically eliminate the complexity of polymorphism. For designs that require predictable compile-time evaluation, simple types, `std::variant`, or template-based patterns may be more suitable choices.

### `constexpr` Destructors

C++20 allows destructors to be declared `constexpr`. This matters for types that manage resources during compile-time evaluation and must be destroyed correctly before that evaluation ends.

```cpp
struct Measurement {
    int value;

    constexpr Measurement(int v) : value(v) {}

    constexpr ~Measurement() = default;
};

constexpr int get_measurement() {
    Measurement item{42};
    return item.value;
}

static_assert(get_measurement() == 42);
```

A destructor being usable during compile-time evaluation does not mean that every operation inside it is permitted. Destruction must also comply with the rules for constant evaluation, and temporary resources created during evaluation must be released according to the applicable rules.

## Dynamic Memory Allocation at Compile Time

### Support for `new` and `delete`

One of the important changes in C++20, introduced through proposal `P0784R7`, is the ability to allocate and deallocate dynamic memory during constant evaluation. Memory allocated through permitted mechanisms during constant evaluation must be released before that evaluation ends. This is commonly referred to as *transient allocation*.

```cpp
constexpr int sum_dynamic(int count) {
    int* values = new int[count];

    int sum = 0;
    for (int i = 0; i < count; ++i) {
        values[i] = i + 1;
        sum += values[i];
    }

    delete[] values;
    return sum;
}

static_assert(sum_dynamic(5) == 15);
```

In this example, memory is allocated during the evaluation and released before the function returns. If memory is allocated with `new` but remains allocated at the end of constant evaluation, that evaluation generally cannot form a valid constant expression.

### Why Is Compile-Time Dynamic Allocation Useful?

Before C++20, many compile-time algorithms had to use fixed-size arrays, predetermined capacities, or complicated recursive approaches. Dynamic allocation makes it possible to build temporary data structures whose sizes depend on inputs. For example, an algorithm can run using temporary storage and ultimately return only a number or another constant result.

This capability is useful for generating constant tables, parsing strings, validating configuration, and performing calculations that need temporary storage. However, compiler resource limits and the cost of constant evaluation still matter. An algorithm that is appropriate at runtime may be too expensive to run during compilation.

### Limitations of Dynamic Memory in Constant Expressions

C++20 does not mean that every memory resource can be carried from compile time into the running program. Temporary allocations must be released correctly during constant evaluation, and the final result must be of a kind permitted by the constant-expression rules.

```cpp
constexpr int* invalid_escape() {
    return new int(10);
}

// This is not a valid constant expression:
// constexpr int* ptr = invalid_escape();
```

The example is intentionally explained in a comment because returning a pointer to memory allocated during this evaluation, without releasing that memory, is not permitted when producing a constant value. If data must remain available in the program, use a suitable representation for constant data, such as a `std::array` with a known size or a static-storage object that is initialized correctly.

## `constexpr` Support in the Standard Library

### Using `std::vector` at Compile Time

In C++20, many `std::vector` operations became usable in constant evaluation. This makes it possible to use a variable-sized container in compile-time calculations, provided that its dynamic allocations are released before the constant evaluation ends.

```cpp
#include <vector>

constexpr int sum_vector() {
    std::vector<int> values;

    values.push_back(10);
    values.push_back(20);
    values.push_back(30);

    int sum = 0;
    for (int value : values) {
        sum += value;
    }

    return sum;
}

static_assert(sum_vector() == 60);
```

This example requires a standard-library implementation that provides the relevant C++20 `constexpr` support. Using `std::vector` in a constant evaluation does not mean that an ordinary `std::vector` can retain compile-time allocated storage in the final program without restriction; the lifetime constraints on allocations still apply.

### Using `std::string` at Compile Time

In C++20, many `std::string` functions also became `constexpr`. As a result, constructing, modifying, and inspecting strings can be part of compile-time calculations.

```cpp
#include <string>

constexpr bool has_expected_size() {
    std::string text = "compile-time";
    text += "-string";
    return text.size() == 18;
}

static_assert(has_expected_size());
```

The ability to evaluate this code at compile time must be compatible with the standard-library implementation and the rules for dynamic memory. For small constant data, `std::string_view` is often a lighter option. However, it does not own its memory, so it must not refer to data whose lifetime has ended.

### Standard Library Algorithms

C++20 expanded `constexpr` support across many standard-library algorithms and functions. For example, many `<algorithm>` algorithms, including `std::sort` and `std::find`, became usable in constant evaluation in C++20, provided that the types, iterators, and operations involved are also compatible with constant evaluation.

```cpp
#include <algorithm>
#include <array>

constexpr bool sorted_values() {
    std::array<int, 5> values{5, 1, 4, 2, 3};
    std::sort(values.begin(), values.end());

    return values == std::array<int, 5>{1, 2, 3, 4, 5};
}

static_assert(sorted_values());
```

This capability reduces the need to implement simple algorithms manually and makes it easier to reuse standard algorithms. However, verify that every operation used along the constant-evaluation path is supported. The fact that an algorithm is `constexpr` does not automatically make every iterator type or comparison function usable at compile time.

### Checking Library Types and Functions

`constexpr` support was not added to every part of the standard library all at once. Some types and functions had already been `constexpr` in earlier standards, while others were expanded in C++20. Therefore, when targeting multiple compilers or libraries, check implementation documentation and the status of standard support. Merely enabling `-std=c++20` does not guarantee that every library feature is fully implemented in the installed toolchain.

## Detecting the Evaluation Context with `std::is_constant_evaluated`

### Introducing `std::is_constant_evaluated`

`std::is_constant_evaluated`, introduced in C++20 in the `<type_traits>` header, indicates whether a call is being evaluated in a context in which constant evaluation is being performed.

```cpp
#include <type_traits>

constexpr int choose_value(int value) {
    if (std::is_constant_evaluated()) {
        return value * 2;
    }

    return value * 3;
}

static_assert(choose_value(10) == 20);

int main() {
    int result = choose_value(10);
    return result == 30 ? 0 : 1;
}
```

The `static_assert` uses the constant-evaluation path. The runtime call in `main` takes the other path. This can sometimes be useful for selecting a more suitable algorithm at compile time or at runtime.

### Appropriate Use Cases

This function can be useful when two implementations produce the same result but one is better suited to constant evaluation. For example, the compile-time path might use a simple algorithm that consumes fewer compiler resources, while the runtime path uses hardware optimization or a faster technique.

Do not use it to introduce unexpected semantic differences into an API. If a function's result depends on whether it is called during compile time or runtime, that difference should be an intentional and documented part of the function's contract.

### A Subtle Point About Potential Evaluation

In some contexts, a compiler may evaluate an expression to determine whether constant evaluation is possible, even if the expression is not ultimately used as a constant value. For this reason, `std::is_constant_evaluated` should not be treated as a simple query about the compiler's final decision to generate code. It reports the evaluation context; it is not a general-purpose way to detect every compiler optimization.

## `consteval` Functions and Mandatory Compile-Time Computation

### The Difference Between `consteval` and `constexpr`

A `constexpr` function can be used both at compile time and at runtime. By contrast, a `consteval` function in C++20 is an *immediate function*, and calls to it must satisfy the requirements for immediate evaluation.

```cpp
consteval int checked_port(int port) {
    if (port < 1 || port > 65535) {
        throw "invalid port";
    }

    return port;
}

constexpr int server_port = checked_port(8080);
```

This example can be evaluated for a valid value. If an argument is not known at compile time or the call does not meet the required conditions, the program is ill-formed. Throwing an exception on the invalid path does not allow an invalid value to be produced normally; such a call cannot complete a valid immediate evaluation in this example.

### When to Use `consteval`

This keyword is suitable for APIs whose values must be computed during translation, such as generating identifiers, validating fixed formats, or calculating configuration values. In contrast, if a function must also process runtime inputs, `constexpr` is generally the more flexible choice.

## Practical Design Patterns

### Validating Configuration at Compile Time

Combining `constexpr` with `static_assert` allows some configuration errors to be detected before the program runs.

```cpp
constexpr bool valid_buffer_size(int size) {
    return size >= 64 && size <= 65536 && (size & (size - 1)) == 0;
}

constexpr int buffer_size = 1024;
static_assert(valid_buffer_size(buffer_size),
              "buffer_size must be a supported power of two");
```

This approach is suitable for values derived from fixed settings, template parameters, or program constants. If a value is read from user input or a configuration file at runtime, runtime validation is still necessary.

### Generating Constant Data Tables

Compile-time calculations can generate data that is only read at runtime. This pattern is useful for conversion tables, lookup tables, and precomputed values.

```cpp
#include <array>
#include <cstddef>

constexpr std::array<int, 8> make_squares() {
    std::array<int, 8> result{};

    for (std::size_t i = 0; i < result.size(); ++i) {
        result[i] = static_cast<int>(i * i);
    }

    return result;
}

constexpr auto squares = make_squares();
static_assert(squares[5] == 25);
```

In this example, the table has a fixed size, and `std::array` provides simple, predictable ownership. For large tables, also consider compile time, object-file size, and the amount of memory required by the program.

### Using Standard Algorithms Instead of Handwritten Code

If a standard algorithm needed by a project supports constant evaluation in C++20, it is usually preferable to use that algorithm rather than write a custom implementation solely for compile-time execution. This improves readability, maintainability, and confidence in the library implementation, provided that the input types and operations are compatible with constant evaluation.

## Limitations and Common Mistakes

### Mistaking `constexpr` for a Guarantee of Compile-Time Execution

The `constexpr` specifier alone does not guarantee that every call is evaluated at compile time. If the result depends on runtime input and the context does not require a constant expression, the call can execute at runtime. Consider `consteval` when immediate evaluation must be enforced.

### Assuming All Standard Libraries Provide the Same Support

Language features and library features are separate concerns. A compiler may support C++20 language syntax while the accompanying standard library has not yet implemented every required `constexpr` operation. Testing with the actual compiler-and-library combination targeted by the project is essential.

### Retaining Dynamic Allocations After Evaluation

Temporary allocation is useful for compile-time calculations, but not every pointer or allocated resource can escape a constant evaluation. Temporary resources must be released within that evaluation. For a persistent result, use a type with suitable storage and lifetime properties.

### High Compilation Costs

Compile-time computation is not free. Complex algorithms, large loops, huge data structures, and deep recursion can increase compilation time and compiler memory consumption. Move calculations to compile time when the benefits for early validation, reduced runtime work, or constant-data generation are meaningful and measurable.

### Implementation-Dependent Behavior

The standard defines language and library rules, but compiler resource limits and the completeness of support vary between versions. If a project targets multiple platforms, run `static_assert` examples and compile-time tests in each platform's build toolchain.

### Confusing `constexpr` with `const`

A `const` variable cannot be modified through that name, but `const` alone does not guarantee compile-time initialization. By contrast, a local variable or object declared `constexpr` must have a valid initializer that can be evaluated as a constant expression. Choose between them based on whether a compile-time constant is actually required.

## Choosing the Right Tool

| Design requirement | Recommended tool | Notes |
|---|---|---|
| Allow use at compile time and runtime | `constexpr` | General-purpose and flexible |
| Require compile-time evaluation | `consteval` | Calls must meet the immediate-evaluation requirements |
| Guarantee static initialization for a variable | `constinit` | Does not make the variable `const` by itself |
| Detect the evaluation context | `std::is_constant_evaluated` | For deliberately selecting an execution path |
| Check a condition at compile time | `static_assert` | Provides a clear diagnostic for an invalid condition |
| Create a fixed-size constant array | `std::array` | Predictable ownership and fixed size |
| Create a temporary variable-sized container | `std::vector` in C++20 | Allocations must comply with constant-evaluation rules |
| Process strings at compile time | `std::string` in C++20 | Consider temporary allocations and data lifetime |

## Build and Compatibility

To enable C++20 mode, select the appropriate language-standard option for the compiler. For example, GCC and Clang generally use `-std=c++20`, while recent MSVC versions provide the corresponding standard-mode option. The exact options and level of support depend on the toolchain version.

Example compilation with GCC or Clang:

```bash
g++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o app
```

The language-standard option alone does not guarantee complete support for every language and library feature. For important capabilities, check the official documentation for the compiler and standard library versions, and keep small, testable examples using `static_assert` in the build process.

## Summary

C++20 turned `constexpr` from a relatively restricted tool for simple expressions and functions into a more practical mechanism for compile-time computation. Fewer restrictions on function bodies, support for `constexpr` destructors, dynamic allocation during constant evaluation, and expanded support for libraries such as `std::vector`, `std::string`, and standard algorithms make it possible to write more general and testable code.

In professional design, it is important to distinguish between allowing constant evaluation and requiring it. `constexpr` is appropriate for the former, `consteval` for mandatory immediate computation, and `constinit` for guaranteeing static initialization. The cost of compilation, the lifetime constraints on temporary allocations, and differences in library support must also be considered. These features are best used when compile-time computation provides a clear, measurable benefit to program correctness, design, or performance.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>