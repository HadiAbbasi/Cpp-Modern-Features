<div align="center">

[🇺🇸 English](./[[unlikely]].md) | [🇮🇷 فارسی](../../fa/cpp20/[[unlikely]].md)

</div>

---

# Understanding the `[[unlikely]]` Attribute in C++20

## Introduction

The standard `[[unlikely]]` attribute was introduced in C++20 to allow programmers to provide the compiler with guidance about the expected likelihood of executing a particular control-flow path.

It is an optimization hint that tells the compiler that a particular statement or execution path is expected to be executed less frequently.

Its primary purpose is to help optimize code in which there is a clear distinction between frequently executed and rarely executed paths, such as a successful operation versus its error-handling path.

The `[[unlikely]]` attribute was introduced alongside `[[likely]]`. Both are standard attributes available since C++20.

An important point is that these attributes are only hints to the compiler. They do not guarantee faster execution or any particular machine-code generation strategy.

## Concept and Design Purpose

### The Problem It Solves

In many programs, not all branches of a conditional statement are executed with equal frequency.

For example:

- Most network requests complete successfully, while errors are relatively rare.
- A parser usually processes valid input, but it must also handle invalid input.
- Most loop iterations continue execution, while only the final iteration terminates the loop.
- In cache-based systems, Cache Hits generally occur more frequently than Cache Misses.

Compilers can estimate branch probabilities using program structure, language rules, and information obtained from profiling.

However, programmers sometimes have knowledge about the actual workload that cannot be inferred from the source code alone.

The `[[unlikely]]` attribute provides a standard way to communicate this information to the compiler.

### Hint Versus Guarantee

The presence of `[[unlikely]]` does not mean that the associated statement will necessarily execute less frequently.

The attribute merely expresses the programmer's expectation.

Therefore, this attribute:

- Does not introduce a new condition.
- Does not change the semantic execution order of statements.
- Does not measure the actual probability of branch execution.
- Does not eliminate or restrict branch execution.
- Does not prove that a branch is unreachable.
- Does not guarantee faster machine code.

Even if a branch marked with `[[unlikely]]` turns out to be frequently executed, the program must still behave correctly.

## Syntax and Usage

### Basic Syntax

The `[[unlikely]]` attribute is a standard attribute that can be applied to permitted statement and label positions.

One of its most common applications is marking an `else` branch:

```
#include <stdexcept>

int parse_value(bool input_is_valid) {
    if (input_is_valid) {
        return 42;
    } else [[unlikely]] {
        throw std::invalid_argument{"invalid input"};
    }
}
```

In this example, the programmer expects the error-handling path to execute rarely.

The `[[unlikely]]` attribute applies to the `else` branch, not to the `input_is_valid` condition or the entire function.

### Using the Attribute with if Statements

Both attributes can be used to identify frequently and infrequently executed paths:

```
int process(int value) {
    if (value >= 0) [[likely]] {
        return value * 2;
    } else [[unlikely]] {
        return -1;
    }
}
```

In this example:

- The `if` branch is marked with `[[likely]]`.
- The `else` branch is marked with `[[unlikely]]`.

Using both attributes is not always necessary. In many situations, marking only one branch is sufficient to communicate the intended information.

The choice should be based on actual workload characteristics rather than assumptions.

### Using the Attribute with switch Statements

In `switch` statements, a `case` or `default` label can also be marked with this attribute.

For example:

```
int handle_status(int status) {
    switch (status) {
        case 0:
            [[likely]] return 0;

        case 1:
            return 1;

        default:
            [[unlikely]] return -1;
    }
}
```

This usage is useful in situations such as:

- Status code handling
- State machine implementations
- Processing different system states
- Dispatching operations
- Handling exceptional states

In this example, zero is assumed to be the most frequent status, while unknown statuses are expected to occur rarely.

### Using the Attribute with Loops

Since the attribute can apply to the relevant statement, it can also provide guidance about the expected likelihood of executing a loop statement.

For example:

```
void consume_items() {
    while (has_more_items()) [[likely]] {
        consume_one_item();
    }
}
```

In this example, continuing the loop is expected to be more likely than exiting it.

However, `[[likely]]` and `[[unlikely]]` are not tools for measuring the number of loop iterations and should not replace profiling.

## Differences Between [[unlikely]] and [[likely]]

Both attributes were standardized in C++20 and belong to the same family.

| Attribute      | Meaning                                                                  |
| -------------- | ------------------------------------------------------------------------ |
| `[[likely]]`   | The associated statement or path is expected to execute more frequently. |
| `[[unlikely]]` | The associated statement or path is expected to execute less frequently. |

These attributes do not specify numerical probabilities.

For example, `[[unlikely]]` does not mean that a branch has a one-percent probability of execution or less. It only expresses a qualitative expectation about its relative frequency.

These attributes should also not be used to establish logical contracts.

If a path is genuinely unreachable, an appropriate mechanism should be used to express that property. Marking it with `[[unlikely]]` does not prove that it cannot execute.

## Understanding Compiler Behavior

### The Role of the Attribute in Optimization

A compiler may use branch-probability hints when making optimization decisions involving:

- Selecting the layout of basic blocks in machine code.
- Placing frequently and infrequently executed paths in different locations.
- Making branch-layout and fall-through decisions.
- Potentially improving instruction-cache utilization.
- Indirectly influencing other optimization decisions.

These are examples of possible decisions, not standard requirements.

The C++ standard does not specify exactly which assembly instructions a compiler must generate or how it must arrange the code.

### The Possibility of Ignoring the Attribute

A compiler is allowed to ignore this hint.

For a particular compiler version, optimization level, or target architecture, the generated assembly might not show any observable difference.

Even if the assembly changes, that does not necessarily mean performance has improved.

### Differences Between Compilers

The language-level meaning of this attribute is standardized, but the quality of its implementation and its ultimate performance impact can vary between compilers.

For example, practical behavior may be influenced by:

- GCC version
- Clang version
- MSVC version
- CPU architecture
- Optimization settings
- Profile information
- Overall program structure

Therefore, evaluating its effectiveness requires examining the compiler version, build options, and actual workload.

## Relationship with Profile-Guided Optimization

### Comparison with Manual Hints

The `[[unlikely]]` attribute communicates information that the programmer already knows to the compiler.

In contrast, Profile-Guided Optimization (PGO) uses information collected from actual program executions to estimate the behavior of different execution paths.

A typical PGO workflow involves the following steps:

- Building the program with appropriate instrumentation.
- Running the program with a representative workload.
- Collecting information about execution-path frequencies.
- Using the collected profile information during compiler optimization.

When a reliable profile is available, its information is generally more accurate than manual assumptions because it reflects actual input distributions and program behavior.

### Potential Conflicts with Profile Information

If manual hints conflict with profile information, the outcome depends on the compiler and its optimization pipeline.

You should not assume that manual attributes always take precedence over profile data, or vice versa.

For accurate decisions, consult the compiler documentation and examine the actual build output.

### Recommended Strategy

For performance-sensitive projects, a reasonable strategy is:

- First, write simple and readable code.
- Measure the program using representative workloads with benchmarks or profilers.
- Use PGO when practical.
- Add manual hints only when reliable information about branch frequencies is available.
- Reevaluate performance after making changes on the target architectures.

## Practical Applications

### Handling Error Paths

One of the most common applications of `[[unlikely]]` is marking an error-handling path that is expected to execute less frequently than the successful path.

For example:

```
int read_value(bool available) {
    if (available) [[likely]] {
        return 42;
    } else [[unlikely]] {
        return -1;
    }
}
```

In this example, the successful path is expected to be more frequent than the error path.

This pattern is useful in:

- Network request processing
- Reading information from files
- Input validation
- System error handling
- Resource availability checks

### Error Handling with std::expected

The `std::expected` type was introduced into the standard library in C++23. It represents an operation that can produce either a successful result or an error.

Although this type is not part of C++20, combining it conceptually with branch hints can be useful:

```
#include <expected>
#include <string>

std::expected<int, std::string> read_value(bool available) {
    if (available) [[likely]] {
        return 42;
    }

    return std::unexpected{std::string{"value unavailable"}};
}
```

This example requires C++23 support.

For projects that must be built with C++20, an existing error-handling mechanism or a custom result type can be used instead.

### Handling Invalid Input in Parsers

In parsers where valid input is dominant, the invalid-input path may be relatively infrequent.

For example:

```
#include <string_view>

bool parse_token(std::string_view token) {
    if (!token.empty()) [[likely]] {
        return token.front() != '#';
    } else [[unlikely]] {
        return false;
    }
}
```

This hint is worth considering only when the actual input distribution follows this pattern.

If empty input is common in the workload, this annotation could be misleading.

### Managing State Machines

In a state machine, the normal state may be very frequent, while a recovery state may occur rarely.

For example:

```
enum class State {
    Ready,
    Processing,
    Recovering
};

void start_processing();
void continue_processing();
void recover();

void handle(State state) {
    switch (state) {
        case State::Ready:
            [[likely]] start_processing();
            break;

        case State::Processing:
            continue_processing();
            break;

        case State::Recovering:
            [[unlikely]] recover();
            break;
    }
}
```

In this example, the `Recovering` state is assumed to occur infrequently.

For professional applications, state frequencies should ideally be derived from reliable telemetry or profiling information.

## Limitations and Subtle Considerations

### No Numerical Probability

This attribute does not provide a way to specify the exact probability of a branch.

For example, there is no standard syntax like the following:

```
[[unlikely(0.01)]]
```

The standard forms are simply `[[likely]]` and `[[unlikely]]`.

### No Guaranteed Performance Improvement

Branch-probability optimization is only one of many factors affecting program performance.

Other factors may have a much greater impact, including:

- Cache Misses
- Memory Access
- Data Dependencies
- Instruction-Level Parallelism
- Vectorization
- Inlining
- Function call overhead
- Algorithmic complexity

Under some circumstances, adding a hint provides no benefit or may even result in a less efficient code layout.

### No Change to Program Semantics

Even if a path marked with `[[unlikely]]` turns out to execute frequently, the program must remain correct.

Program correctness must never depend on a branch being executed infrequently.

### Avoiding Excessive Usage

Using these attributes extensively without measurement can reduce the usefulness of the hints.

If almost every branch is marked with either `[[likely]]` or `[[unlikely]]`, little meaningful information is communicated about the relative importance of different paths.

### Differences Between Workloads

Branch frequencies may differ between development, testing, and production environments.

A program may also receive different input distributions under different operating conditions.

Therefore, a fixed hint in the source code is not necessarily appropriate for every workload.

### Not a Replacement for Proper Design

Error handling requires correctly designing several aspects of the program, including:

- API contracts
- Function return types
- Exception policies
- Input validation
- Resource management
- Failure behavior

The `[[unlikely]]` attribute is merely an optimization hint that complements proper design. It does not replace any of these mechanisms.

## Common Mistakes

### Assuming That Branch Execution Is Restricted

A common misconception is that the compiler restricts the execution of a branch when it encounters `[[unlikely]]`.

This does not happen.

The attribute is only a hint and does not change the program's logical behavior.

### Using It to Indicate Unreachable Code

If a branch is logically unreachable, `[[unlikely]]` is not the appropriate mechanism for expressing that property.

The attribute indicates that a path is expected to be infrequent, not impossible.

It should also not be confused with `std::unreachable`.

The `std::unreachable()` function belongs to the C++23 standard library and has a different contract.

### Assuming Identical Behavior Across Compilers

Having the same standardized syntax and language-level meaning does not imply identical machine-code output.

Evaluation should be performed on the compilers and architectures actually supported by the project.

### Adding Hints Without Benchmarking

If performance has not been measured, adding hints may increase code-maintenance complexity without providing any demonstrable benefit.

Identify the bottleneck first, then evaluate the change using reliable benchmarks.

## Comparison with Older and Compiler-Specific Techniques

Before these attributes were standardized, some compilers provided proprietary mechanisms for expressing branch probabilities.

For example, GCC and Clang support mechanisms such as `__builtin_expect`.

These mechanisms are compiler-specific extensions or built-ins and should not be considered equivalent to the standard attribute in terms of syntax and portability.

An example of the GCC and Clang-specific approach is:

```
if (__builtin_expect(error_code != 0, 0)) {
    handle_error();
}
```

In this pattern, the second argument indicates which result is expected for the expression.

This approach is not as portable as `[[unlikely]]` and may be unavailable or behave differently in other compilers.

For new, portable C++20 code, the standard attribute is generally the better choice.

However, projects that depend on older compilers or specific compiler capabilities should verify actual support in their build environments.

## Compatibility with C++ Standard Versions

The `[[likely]]` and `[[unlikely]]` attributes were standardized in C++20.

Using them requires a compiler with appropriate C++20 support.

Unlike library features, these attributes are part of the language syntax and do not require a specific header.

The C++20 language mode can be enabled through the appropriate build options.

For example:

```
g++ -std=c++20 -O2 main.cpp -o app
```

```
clang++ -std=c++20 -O2 main.cpp -o app
```

These commands are illustrative examples, and actual support depends on the compiler version.

Enabling `-std=c++20` alone also does not guarantee that all expected optimizations will be performed.

## Evaluating the Attribute in Real Projects

To evaluate the value of `[[unlikely]]`, comparisons should be performed in a controlled manner.

The recommended steps are:

- Select a realistic and repeatable workload.
- Build one version without hints and another with hints.
- Keep optimization settings consistent.
- Use the same compiler version and target architecture.
- Run benchmarks multiple times to reduce measurement noise.
- Examine Branch Misses and Instruction Count when necessary.
- Investigate Cache Behavior using profiling tools.
- Evaluate the results against other important workloads.

Examining assembly can help explain how the compiler responds to the hint, but the final evaluation should be based on program behavior under real workloads.

A difference in assembly output alone does not prove a performance improvement.

## Decision-Making Checklist

Before adding `[[unlikely]]`, consider the following questions:

- Is the target path genuinely infrequent in important workloads?
- Is this information based on benchmarks, profiling, or reliable knowledge of the input distribution?
- Do the compiler and supported compiler versions recognize this attribute?
- Do benchmarks demonstrate that the hint provides a benefit?
- Does the program remain semantically correct regardless of the hint?
- Is a more appropriate approach, such as PGO, available?
- Does the hint remain valid across all operational environments?

If the answers to these questions are unclear, it is generally better to leave the code without the hint.

## Summary

The `[[unlikely]]` attribute in C++20 provides a standard way to indicate that a particular statement or path associated with a label is expected to execute infrequently.

Its primary purpose is to help the compiler make optimization decisions related to branch probability and code layout, particularly for error-handling paths, exceptional states, and infrequently executed branches.

However, `[[unlikely]]` is only an optimization hint:

- It does not specify a numerical probability.
- It does not change branch execution.
- It does not prove that a branch is unreachable.
- It does not guarantee performance improvements.
- It does not replace profiling or PGO.

In professional projects, its usage should be limited, based on actual workload characteristics, and accompanied by benchmarking.

When reliable profile information is available, PGO generally provides more accurate information than manual assumptions.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>