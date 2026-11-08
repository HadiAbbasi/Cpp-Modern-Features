<div align="center">

[🇺🇸 English](./[[likely]].md) | [🇮🇷 فارسی](../../fa/cpp20/[[likely]].md)

</div>

---

# Understanding `[[likely]]` in C++20

## Introduction

The `[[likely]]` attribute is a C++20 language feature designed to provide the compiler with information about the expected execution paths of a program. It tells the compiler that a particular execution path, such as a conditional branch, is more likely to be selected.

In C++ programs, decisions about instruction ordering and conditional branch selection are important aspects of code optimization. Compilers typically use statistical information, control flow analysis, and techniques such as Profile-Guided Optimization (PGO) to optimize frequently executed paths.

However, in some situations, the compiler may not have sufficient information about the program's execution patterns. The `[[likely]]` attribute allows developers to provide explicit guidance.

This feature is part of the C++20 language standard and belongs to the category of standard attributes.

An important point is that `[[likely]]` does not control program execution. It is merely an optimization hint. Therefore, using it must not change the observable behavior of a program.

## Understanding Attributes in C++

In C++, attributes provide a standardized mechanism for supplying additional information to the compiler.

This information can help the compiler make decisions about code processing, optimization, or checking certain program properties.

The general syntax of a standard attribute is:

```
[[attribute_name]]
```

For example, the `[[nodiscard]]` attribute tells the compiler that ignoring the return value of a specified function or object may be a mistake.

Similarly, the `[[deprecated]]` attribute marks APIs whose use is discouraged.

The `[[likely]]` attribute belongs to the same family, but its purpose is to provide information about the probability of different execution paths.

## Introducing the [[likely]] Attribute

The `[[likely]]` attribute was introduced in the C++20 standard.

It tells the compiler that a particular execution path is expected to be selected more frequently than alternative paths.

Its syntax is:

```
[[likely]]
```

This attribute is commonly used with labels and conditional statements.

### Using [[likely]] with if Statements

One of the most common applications of `[[likely]]` is identifying the more probable branch of a conditional statement.

```
#include <iostream>

int main() {
    int value = 100;

    if (value > 0) [[likely]] {
        std::cout << "Positive value\n";
    } else {
        std::cout << "Non-positive value\n";
    }
}
```

In this example, the programmer tells the compiler that the condition `value > 0` is expected to be true most of the time.

As a result, the compiler can consider this information when generating machine code and optimizing the more probable execution path.

However, if `value` is negative, the `else` branch will still execute correctly.

Therefore, `[[likely]]` does not affect the logical result of the condition.

### Using [[likely]] with else

The `else` branch can also be identified as the more probable execution path.

```
#include <iostream>

int main() {
    int value = -10;

    if (value > 0) {
        std::cout << "Positive value\n";
    } else [[likely]] {
        std::cout << "Non-positive value\n";
    }
}
```

In this example, the programmer expects the condition to be false most of the time.

Consequently, the `else` branch is identified as the more probable path.

### Using [[likely]] with switch

The `[[likely]]` attribute can also be used with `switch` statements.

```
#include <iostream>

enum class Status {
    Success,
    Error,
    Pending
};

int main() {
    Status status = Status::Success;

    switch (status) {
        case Status::Success: [[likely]] {
            std::cout << "Operation completed\n";
            break;
        }

        case Status::Error: {
            std::cout << "Operation failed\n";
            break;
        }

        case Status::Pending: {
            std::cout << "Operation is pending\n";
            break;
        }
    }
}
```

In this example, `Status::Success` is identified as the more probable execution path.

This information may help the compiler optimize the different paths of the `switch` statement.

However, the actual impact depends on the compiler implementation and the generated machine code.

## Introducing the [[unlikely]] Attribute

The `[[unlikely]]` attribute complements `[[likely]]` and was introduced in C++20.

It tells the compiler that a particular execution path is expected to be selected infrequently.

Its syntax is:

```
[[unlikely]]
```

For example:

```
#include <iostream>

int main() {
    int value = -1;

    if (value < 0) [[unlikely]] {
        std::cerr << "Invalid value\n";
    } else {
        std::cout << "Valid value\n";
    }
}
```

In this example, the programmer expects negative values to occur infrequently.

Therefore, the error-handling branch is marked with `[[unlikely]]`.

This information may help the compiler optimize the normal execution path more effectively.

### Comparing [[likely]] and [[unlikely]]

These two attributes complement each other in terms of their purpose.

| Attribute      | Meaning                      | Typical Usage                        |
| -------------- | ---------------------------- | ------------------------------------ |
| `[[likely]]`   | More probable execution path | Normal and frequently executed paths |
| `[[unlikely]]` | Less probable execution path | Errors and exceptional conditions    |

For example, in a request-processing function, receiving a valid request may be much more common than receiving an invalid one.

```
#include <iostream>

bool processRequest(int request) {
    if (request < 0) [[unlikely]] {
        std::cerr << "Invalid request\n";
        return false;
    }

    [[likely]] return true;
}

int main() {
    processRequest(10);
}
```

In this example, the invalid-request path is marked as less probable.

The successful return path is also identified as more probable.

However, applying `[[likely]]` to a `return` statement should be understood in the context of the attribute rules and the actual code structure. Here, the attribute applies to the statement, not to the returned value itself.

## How These Attributes Work Internally

The `[[likely]]` and `[[unlikely]]` attributes do not directly change how a program executes.

Instead, they provide the compiler with information about the expected probability of different execution paths.

### Impact on Code Generation

The compiler can use this information when deciding how to generate machine code.

For example, on modern processors, the placement of instructions in memory can affect execution performance.

One optimization objective is to arrange frequently executed paths in a way that improves code access and execution efficiency.

Under these circumstances, the compiler may use the supplied hints for tasks such as:

- Improving Basic Block ordering.
- Optimizing Branch Layout.
- Reducing the execution cost of frequently executed paths.
- Improving Instruction Cache Locality.
- Optimizing certain Branch Prediction-related decisions.

However, none of these outcomes is guaranteed.

### Relationship with Branch Prediction

Modern processors use a hardware mechanism called Branch Prediction to predict the next execution path of a program.

This capability allows the processor to begin processing instructions from a predicted path before the condition's outcome is definitively known.

If the prediction is incorrect, the processor may need to discard some of the work already performed and continue along the correct path.

This can increase execution costs.

The `[[likely]]` attribute may help the compiler generate code that is better suited to frequently executed paths.

However, it is important to understand that this attribute is not a direct hardware command for configuring the processor's Branch Predictor.

There is also no guarantee that the compiler will translate this hint into a specific hardware instruction.

### Relationship with Basic Blocks

Compilers generally divide program code into units called Basic Blocks.

A Basic Block is a sequence of consecutive instructions with a single entry point and typically a single exit point at its end.

Conditional statements can create multiple Basic Blocks and different paths in a Control Flow Graph.

Information about the probability of executing these paths can help the compiler arrange and optimize the blocks.

For example, the compiler may arrange frequently executed paths in a way that makes program execution more efficient.

## Usage Rules and Limitations

The `[[likely]]` and `[[unlikely]]` attributes have specific rules defined by the C++20 standard.

These rules are important because attributes are not merely decorative annotations; their placement matters according to the language syntax.

### Applying Attributes to Statements

These attributes are Statement Attributes.

Therefore, they must appear in positions where they apply to a Statement or Label according to the language rules.

For example:

```
if (condition) [[likely]] {
    performOperation();
}
```

They can also be used with `else`:

```
if (condition) {
    handleError();
} else [[likely]] {
    performSuccess();
}
```

### Applying Attributes to Labels

These attributes can also be applied to labels in `switch` statements.

```
switch (status) {
    case 0: [[likely]]
        handleSuccess();
        break;

    case 1: [[unlikely]]
        handleFailure();
        break;
}
```

In this example, the path associated with `case 0` is identified as more probable, while the path associated with `case 1` is identified as less probable.

### Applying Attributes to Standalone Statements

These attributes are not limited to conditional statements. They can also be applied to standalone statements.

```
#include <iostream>

int main() {
    [[likely]] std::cout << "Normal execution\n";
}
```

In this example, the attribute applies to the statement that prints the message.

However, this usage generally provides less optimization information than explicitly identifying conditional branches.

### Using Both Attributes on the Same Statement

A statement or label should not be marked with both attributes in a contradictory manner.

For example:

```
// Do not use contradictory likelihood attributes.
```

Applying both `[[likely]]` and `[[unlikely]]` to the same statement is semantically contradictory and should be avoided in portable, standards-conforming code.

## Comparison with Older Approaches

Before `[[likely]]` and `[[unlikely]]` were introduced in C++20, some compilers provided their own mechanisms for supplying Branch Prediction hints.

These mechanisms were generally compiler-specific and did not offer the same level of portability.

### Comparison with \_\_builtin_expect

GCC and Clang provide the compiler built-in `__builtin_expect` for supplying expected-value information to the compiler.

Example:

```
#include <iostream>

int main() {
    int value = 10;

    if (__builtin_expect(value == 0, 0)) {
        std::cerr << "Unexpected zero\n";
    } else {
        std::cout << "Normal execution\n";
    }
}
```

In this example, the second argument tells the compiler which value the first expression is expected to produce.

Here, the expected value is zero. Therefore, the condition `value == 0` is identified as the more probable condition.

This built-in is not part of the C++ standard.

In contrast, `[[likely]]` and `[[unlikely]]` are standardized language features.

### Comparing the Approaches

| Feature             | `[[likely]]`        | `[[unlikely]]`      | `__builtin_expect`                   |
| ------------------- | ------------------- | ------------------- | ------------------------------------ |
| Standard            | C++20               | C++20               | Compiler-specific                    |
| Portability         | Standardized        | Standardized        | Compiler-dependent                   |
| Readability         | High                | High                | Lower                                |
| Primary Usage       | More probable paths | Less probable paths | Providing expected-value information |
| Compiler Dependency | No                  | No                  | Yes                                  |

For new portable code, using standard attributes is generally the better choice.

However, in projects that depend on compiler-specific capabilities, built-ins may still be useful.

## Relationship with Profile-Guided Optimization

One of the most important topics related to `[[likely]]` is Profile-Guided Optimization, or PGO.

This technique allows the compiler to make more informed optimization decisions using information collected from actual program executions.

### Understanding PGO

With PGO, a program is initially instrumented using specialized tools to collect information about its actual runtime behavior.

This information may include:

- The number of times functions are executed.
- The number of times conditional branches are taken.
- Frequently executed program paths.
- Loop execution patterns.
- The frequency of different Control Flow paths.

The compiler then uses this information when building the final program.

### Differences Between PGO and [[likely]]

The main difference between these approaches is the source of the information they use.

With `[[likely]]`, the programmer provides a hint based on their knowledge of the program's expected behavior.

With PGO, information can be collected from actual program executions and representative workloads.

| Criterion               | `[[likely]]`                   | PGO                        |
| ----------------------- | ------------------------------ | -------------------------- |
| Information Source      | Programmer's prediction        | Actual runtime data        |
| Requires Test Execution | No                             | Usually yes                |
| Potential Accuracy      | Depends on prediction accuracy | Depends on Profile quality |
| Preparation Cost        | Low                            | Higher                     |
| Workload Adaptability   | Limited                        | Greater                    |

In large and performance-critical projects, PGO can be more effective than manually supplied hints.

However, PGO also produces better results when the collected Profile data accurately represents the behavior of real users and production environments.

## Practical Applications

The `[[likely]]` and `[[unlikely]]` attributes can be useful in certain practical situations.

### Error Handling

One suitable application is identifying error paths as less probable execution paths.

```
#include <iostream>
#include <stdexcept>

int divide(int numerator, int denominator) {
    if (denominator == 0) [[unlikely]] {
        throw std::invalid_argument("Division by zero");
    }

    return numerator / denominator;
}

int main() {
    std::cout << divide(20, 4) << '\n';
}
```

In this example, division by zero is assumed to occur infrequently.

Therefore, the error-handling path is marked with `[[unlikely]]`.

This may help the compiler optimize the normal execution path.

### Processing Network Requests

In Network Server systems, most requests may be valid, while only a small percentage may result in errors.

```
#include <string_view>

bool validateRequest(std::string_view request) {
    if (request.empty()) [[unlikely]] {
        return false;
    }

    return true;
}
```

In this example, an empty request is treated as an unlikely condition.

However, this assumption should be based on the actual behavior of the system.

### Processing Frequently Occurring Data

When processing large datasets, a particular value or state may occur much more frequently than other values.

```
#include <cstdint>

std::uint32_t processValue(std::uint32_t value) {
    if (value != 0) [[likely]] {
        return value * 2;
    }

    return 0;
}
```

In this example, most input values are assumed to be nonzero.

Consequently, the path that processes nonzero values is identified as more probable.

### Processing Loops

In algorithms with many iterations, a condition may evaluate in a particular way during most iterations.

```
#include <cstddef>
#include <vector>

std::size_t countPositive(const std::vector<int>& values) {
    std::size_t count = 0;

    for (int value : values) {
        if (value > 0) [[likely]] {
            ++count;
        }
    }

    return count;
}
```

In this example, most values are assumed to be positive.

If this assumption matches the actual input data, the compiler may use it for optimization.

However, the impact of the attribute in large loops should be evaluated through benchmarking.

## Common Mistakes

Incorrect use of Branch Prediction hints can be ineffective and, in some situations, may even reduce performance.

### Using Attributes Without Understanding the Workload

One common mistake is using `[[likely]]` based on personal assumptions.

For example, a programmer may believe that a condition is almost always true, while actual runtime data shows that both branches are executed with approximately equal frequency.

In such situations, the supplied hint may conflict with the program's actual behavior.

It is better to analyze program behavior through profiling before using these attributes.

### Assuming a Guaranteed Performance Improvement

The `[[likely]]` attribute does not guarantee that a program will run faster.

The compiler may ignore the supplied information or determine that another optimization strategy is more appropriate.

Therefore, adding this attribute should not automatically be expected to improve performance.

### Overusing the Attributes

Applying `[[likely]]` and `[[unlikely]]` to a large number of statements can reduce the informational value of these hints.

These attributes are more useful when applied to important paths with clearly different execution probabilities.

### Using Attributes to Control Program Execution

A common misconception is that `[[likely]]` forces a branch to execute earlier or more frequently.

This is incorrect.

For example:

```
#include <iostream>

int main() {
    bool condition = false;

    if (condition) [[likely]] {
        std::cout << "First branch\n";
    } else {
        std::cout << "Second branch\n";
    }
}
```

In this example, despite the use of `[[likely]]`, the second branch executes because the condition is `false`.

The logical behavior of the program is still determined by the condition's value.

### Replacing Profiling with Manual Hints

In complex applications, actual user behavior may differ from developers' expectations.

In such projects, relying on manually supplied hints instead of profiling can result in inappropriate optimization decisions.

For professional optimization work, actual measurements should take priority.

## Limitations of Likelihood Attributes

The `[[likely]]` and `[[unlikely]]` attributes have several important limitations that should be considered during software design.

### No Guaranteed Optimization Result

The C++ standard does not guarantee that the compiler will apply the supplied hint during code generation.

Therefore, these attributes must not be used to achieve a specific machine-level behavior.

### Compiler and Target Architecture Dependency

Different compilers may use this information in different ways.

Additionally, processor architecture, hardware capabilities, optimization levels, and program structure can affect the impact of these hints.

### Dependency on Input Data

If the input data pattern changes, the most probable execution path may also change.

For example, an application that usually receives valid requests during development may encounter a large number of invalid requests in production.

In such situations, previously supplied manual hints may no longer be appropriate.

### No Changes to Semantics

These attributes must not affect:

- The logical results of conditions.
- The semantic execution order of statements.
- Exception Handling rules.
- Computational correctness.
- The observable behavior of the program.

Any code whose behavior changes solely because of the presence of `[[likely]]` or `[[unlikely]]` indicates an incorrect use of these attributes.

## Evaluating Performance with Benchmarking

The best way to determine whether `[[likely]]` and `[[unlikely]]` are useful is to compare program performance under equivalent conditions.

For this purpose, one version of the program should be built without hints and another with appropriate hints.

### Benchmarking Example

```
#include <cstddef>
#include <cstdint>
#include <vector>

std::uint64_t countWithoutHint(const std::vector<int>& values) {
    std::uint64_t count = 0;

    for (int value : values) {
        if (value > 0) {
            ++count;
        }
    }

    return count;
}

std::uint64_t countWithHint(const std::vector<int>& values) {
    std::uint64_t count = 0;

    for (int value : values) {
        if (value > 0) [[likely]] {
            ++count;
        }
    }

    return count;
}
```

In this example, both functions have identical behavior, but only one uses a hint.

For an accurate evaluation, the following conditions should be observed:

- Input data must be identical for both tests.
- The number of benchmark iterations must be sufficient.
- Measurements should be repeated multiple times.
- The effects of Cache behavior and other hardware factors should be considered.
- The compiler and optimization level must be identical.
- Optimizer-driven elimination of computations must be prevented.

It is also preferable to perform the test using data patterns that represent the actual workload.

An important point is that the compiler may generate identical machine code for both versions. In that case, there will be no measurable performance difference.

## Compiler Support

The `[[likely]]` and `[[unlikely]]` attributes are standard C++20 features.

Using them requires a compiler that supports C++20 and the corresponding attributes.

Modern versions of widely used compilers provide support for these features, including:

- GCC
- Clang
- Microsoft Visual C++

To enable C++20 mode in GCC and Clang, the following command can typically be used:

```
g++ -std=c++20 main.cpp -O2 -o app
```

Alternatively:

```
clang++ -std=c++20 main.cpp -O2 -o app
```

In Microsoft Visual C++, the following options can be used:

```
cl /std:c++20 /O2 main.cpp
```

An important distinction is that supporting C++20 and actually using these hints during optimization are two different things.

Even if a compiler correctly accepts these attributes, this does not necessarily mean that they will produce a measurable optimization in every situation.

## Professional Design and Usage Recommendations

The following principles are recommended when using `[[likely]]` and `[[unlikely]]`:

- Use `[[likely]]` for paths that are genuinely executed frequently.
- Use `[[unlikely]]` for exceptional and infrequently executed paths.
- Whenever possible, use profiling data to determine branch probabilities.
- In performance-critical code, evaluate the impact of these hints through benchmarking.
- Avoid unnecessary and widespread use of these attributes.
- Use standard language mechanisms, such as conditional statements, to control program behavior rather than likelihood attributes.
- In public libraries, avoid making unrealistic assumptions about the input patterns of consumers.
- Reevaluate existing hints whenever the application's workload changes.

## Summary

The `[[likely]]` and `[[unlikely]]` attributes were introduced in C++20 to allow programmers to provide the compiler with information about the expected execution probabilities of different program paths.

They are used to identify more probable and less probable execution paths, respectively.

Their main advantage is providing a standardized, readable, and portable way to express Branch Prediction hints.

However, these attributes do not guarantee performance improvements and do not change the logical behavior of a program.

In professional projects, they are best used alongside profiling, benchmarking, and appropriate performance optimization techniques.

Final Note: The use of `[[likely]]` and `[[unlikely]]` should be based on an understanding of the program's actual behavior, not merely on assumptions about its execution paths. In C++ optimization, measuring and analyzing real performance is always more reliable than relying on intuition.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>