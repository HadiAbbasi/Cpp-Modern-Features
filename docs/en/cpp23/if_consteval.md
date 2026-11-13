<div align="center">

[🇺🇸 English](./if_consteval.md) | [🇮🇷 فارسی](../../fa/cpp23/if_consteval.md)

</div>

---

# **A Comprehensive Guide to if consteval in C++23**

## **Introduction**

`if consteval` is an important feature introduced in the C++23 standard that allows developers to distinguish between compile-time and runtime evaluation contexts directly at the language level. It makes it possible to select different execution paths depending on whether a function is being evaluated during compile time or at runtime.

This feature is particularly useful when designing `constexpr` functions, implementing algorithms that can be evaluated at compile time, developing generic libraries, and working with `consteval` functions.

Before C++23, developers typically used `std::is_constant_evaluated()` to detect constant evaluation. Although this approach remains valid and useful, it has certain limitations, particularly when an execution path needs to be placed directly within an Immediate Function Context.

`if consteval` addresses this limitation by introducing a dedicated control-flow construct into the language.

This document covers the syntax and behavior of this feature, its relationship with `constexpr`, `consteval`, and `std::is_constant_evaluated()`, and its practical applications in modern C++ development.

## **Understanding Constant Evaluation in C++**

To understand `if consteval` correctly, it is important to distinguish between compile-time and runtime execution.

In C++, certain expressions can be evaluated at compile time. For example, a `constexpr` variable must be initialized with a constant expression.

```
constexpr int square(int value) {
    return value * value;
}

constexpr int result = square(5);
```

In this example, the compiler must evaluate `square(5)` at compile time.

However, the same function can also be used at runtime:

```
int main() {
    int value = 5;

    int result = square(value);
}
```

In this case, because the value of `value` is determined at runtime, the function call is also evaluated at runtime.

Therefore, a `constexpr` function is not necessarily executed at compile time in every situation.

This distinction is important because sometimes a function needs to use different algorithms depending on its evaluation context.

For example:

- At compile time, an algorithm compatible with constant expression requirements may be necessary.
- At runtime, a different algorithm with better performance may be preferable.
- Some operations are only available at runtime.
- Some calculations are better performed during compilation.

The `if consteval` feature is designed specifically to handle these situations.

## **Introduction to if consteval**

The `if consteval` feature was introduced in the C++23 standard.

It is a selection statement that determines which branch should execute:

- If the expression is evaluated in a Manifestly Constant-Evaluated Context, the constant-evaluation branch is selected.
- Otherwise, the runtime branch is selected.

### **Basic Syntax**

The basic syntax is:

```
if consteval {
    // Compile-time path
}
else {
    // Runtime path
}
```

An important detail is that, unlike an ordinary `if` statement, this construct does not accept an arbitrary condition.

Similarly, unlike `if constexpr`, the keyword `constexpr` does not appear after `if`.

The compiler selects the appropriate branch based on the evaluation context.

### **Simple Example**

```
#include <iostream>

constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return value + value;
    }
}

int main() {
    constexpr int compileTimeResult = calculate(5);

    int value = 5;
    int runtimeResult = calculate(value);

    std::cout << compileTimeResult << '\n';
    std::cout << runtimeResult << '\n';
}
```

Output:

```
25
10
```

In this example, the function is called twice.

In the first call, the function is invoked in a constant-evaluation context, so the first branch is executed.

In the second call, the function is evaluated at runtime, so the second branch is selected.

The important point is that branch selection depends on the evaluation context, not on the argument's value.

## **Understanding Manifestly Constant-Evaluated Contexts**

One of the most important concepts related to `if consteval` is the Manifestly Constant-Evaluated Context.

This term refers to contexts in which the C++ standard explicitly requires an expression to be evaluated as a constant expression.

For example:

```
constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return value + value;
    }
}

constexpr int result = calculate(10);
```

In this example, initializing `result` requires a constant expression. Therefore, the function call occurs in a Manifestly Constant-Evaluated Context.

In contrast:

```
int main() {
    int value = 10;

    int result = calculate(value);
}
```

In this case, constant evaluation is not required, so the function executes at runtime.

### **The Difference Between Constant Evaluation and Compile-Time Execution**

There is an important distinction in C++: just because the compiler can evaluate an expression during compilation does not mean that its evaluation context is Manifestly Constant-Evaluated.

For example, compiler optimization may evaluate a calculation before runtime. This alone does not make the operation a constant evaluation according to the language rules.

Additionally, certain contexts may undergo trial constant evaluation when the language rules require checking whether an expression can be a constant expression.

Therefore, the behavior of `if consteval` should not be interpreted solely according to when the compiler actually performs a calculation.

The relevant criterion is the language's constant-evaluation rules, not the compiler's optimization decisions.

## **Understanding Immediate Function Contexts**

The most important difference between `if consteval` and earlier approaches to detecting constant evaluation is that the constant-evaluation branch creates an Immediate Function Context.

To understand why this matters, we first need to understand Immediate Functions.

### **Understanding Immediate Functions**

The `consteval` feature was introduced in C++20.

A `consteval` function is an immediate function. Its invocations must satisfy the language requirements for constant evaluation.

```
consteval int square(int value) {
    return value * value;
}
```

A valid invocation is:

```
constexpr int result = square(5);
```

However, the following invocation is invalid:

```
int main() {
    int value = 5;

    int result = square(value);
}
```

This is because the value of `value` is determined at runtime.

### **The Problem of Calling Immediate Functions from constexpr Functions**

Suppose we have a `constexpr` function and want to call a `consteval` function from inside it.

```
consteval int square(int value) {
    return value * value;
}

constexpr int calculate(int value) {
    return square(value);
}
```

This implementation is invalid.

The reason is that the parameter `value` is not necessarily a constant expression when the function is defined.

Even if `calculate` is eventually called during compile-time evaluation, we cannot generally assume that `square(value)` is valid in every possible evaluation context.

### **Solving the Problem with if consteval**

The `if consteval` feature solves this problem:

```
consteval int square(int value) {
    return value * value;
}

constexpr int calculate(int value) {
    if consteval {
        return square(value);
    }
    else {
        return value * value;
    }
}
```

In this example, the first branch is an Immediate Function Context.

As a result, calling `square` with the parameter `value` in this branch is permitted.

During constant evaluation, the first branch is selected, and the `consteval` function can be called successfully.

At runtime, the second branch is executed, avoiding an invalid immediate-function invocation.

**This is the most important advantage of&#x20;**`if consteval`**&#x20;over&#x20;**`std::is_constant_evaluated()`**.**

## **Introducing the Negated Form: if !consteval**

C++23 also provides a negated form of this construct:

```
if !consteval {
    // Runtime path
}
else {
    // Compile-time path
}
```

Semantically, this syntax is equivalent to the following structure:

```
if consteval {
    // Compile-time path
}
else {
    // Runtime path
}
```

The only difference is the order of the branches.

In the negated form:

- The main branch handles runtime execution.
- The `else` branch handles constant evaluation.

### **Practical Example**

```
#include <iostream>

constexpr int calculate(int value) {
    if !consteval {
        std::cout << "Runtime execution\n";

        return value * 2;
    }
    else {
        return value * value;
    }
}

int main() {
    constexpr int compileTimeResult = calculate(5);

    int runtimeResult = calculate(5);

    std::cout << compileTimeResult << '\n';
    std::cout << runtimeResult << '\n';
}
```

Output:

```
Runtime execution
25
10
```

In this example, the `std::cout` operation is placed only in the runtime branch.

Because this branch is not executed during constant evaluation, its presence does not prevent the function from being used in a constant-evaluation context.

However, this feature does not make every operation valid during constant evaluation. Any operation actually executed along the compile-time path must still satisfy the requirements for constant expressions.

### **Difference from the Logical NOT Operator**

The `!` in `if !consteval` is not being used to negate an arbitrary Boolean expression.

This syntax is a dedicated form of selection statement.

It is also important to note that the negated form is semantically equivalent to reversing the branches of the positive form. Consequently, its constant-evaluation branch still provides an Immediate Function Context.

## **Comparing if consteval and if constexpr**

One common mistake when using modern C++ features is treating `if consteval` and `if constexpr` as interchangeable.

These two constructs have different purposes and behaviors.

### **Understanding if constexpr**

The `if constexpr` feature was introduced in C++17.

It selects a branch based on a compile-time condition.

```
#include <type_traits>

template <typename T>
constexpr auto getValue(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    }
    else {
        return value;
    }
}
```

In this example, branch selection depends on the type of `T`.

If `T` is an integral type, the first branch is selected.

Otherwise, the second branch is selected.

### **Comparing Their Behavior**

| Feature                               | `if constexpr`                                           | `if consteval`                         |
| ------------------------------------- | -------------------------------------------------------- | -------------------------------------- |
| Introduced in                         | C++17                                                    | C++23                                  |
| Selection criterion                   | Compile-time condition                                   | Evaluation context                     |
| Requires a Boolean condition          | Yes                                                      | No                                     |
| Relationship with templates           | Particularly important                                   | Independent of templates               |
| Discards an unselected branch         | Yes, under the applicable rules for discarded statements | No                                     |
| Creates an Immediate Function Context | No                                                       | Yes, in the constant-evaluation branch |
| Primary purpose                       | Compile-time branching                                   | Compile-time/runtime branching         |

### **Understanding Discarded Statements**

One important characteristic of `if constexpr` is that, in template contexts, the branch that is not selected can be treated as a discarded statement.

For example:

```
#include <type_traits>

template <typename T>
constexpr auto process(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    }
    else {
        return value.length();
    }
}
```

If `T` is an integral type, the branch containing `value.length()` is treated as a discarded statement during instantiation.

However, `if consteval` does not provide this behavior.

Both branches must satisfy the applicable language rules. Branch selection only determines which execution path is taken during runtime or constant evaluation.

### **Using Both Features Together**

In some designs, both features can be used together.

```
#include <type_traits>

template <typename T>
constexpr auto process(T value) {
    if constexpr (std::is_integral_v<T>) {
        if consteval {
            return value * value;
        }
        else {
            return value + value;
        }
    }
    else {
        return value;
    }
}
```

In this example:

- `if constexpr` checks the type of the value.
- `if consteval` detects the evaluation context.

This combination is useful when developing complex templates and generic libraries.

## **Comparing if consteval with std::is_constant_evaluated()**

The `std::is_constant_evaluated()` function was introduced in C++20 and is available through the following header:

```
#include <type_traits>
```

This function determines whether its invocation occurs in a Manifestly Constant-Evaluated Context.

### **Simple Example**

```
#include <type_traits>

constexpr int calculate(int value) {
    if (std::is_constant_evaluated()) {
        return value * value;
    }
    else {
        return value + value;
    }
}
```

In terms of branch selection, this function behaves similarly to `if consteval`.

However, there is one very important difference.

### **The Immediate Function Context Problem**

`std::is_constant_evaluated()` only reports the constant-evaluation status.

Calling this function does not create an Immediate Function Context.

Consequently, the following code is invalid:

```
#include <type_traits>

consteval int square(int value) {
    return value * value;
}

constexpr int calculate(int value) {
    if (std::is_constant_evaluated()) {
        return square(value);
    }
    else {
        return value * value;
    }
}
```

The reason is that the `if` condition is simply a Boolean expression. Its body does not become an Immediate Function Context.

As a result, calling `square(value)` is still subject to the usual restrictions on immediate functions.

### **Correct Implementation with if consteval**

```
consteval int square(int value) {
    return value * value;
}

constexpr int calculate(int value) {
    if consteval {
        return square(value);
    }
    else {
        return value * value;
    }
}
```

This version resolves the problem.

### **Detailed Comparison**

| Feature                                                               | `std::is_constant_evaluated()` | `if consteval`   |
| --------------------------------------------------------------------- | ------------------------------ | ---------------- |
| Introduced in                                                         | C++20                          | C++23            |
| Feature category                                                      | Standard library function      | Language feature |
| Detects constant evaluation                                           | Yes                            | Yes              |
| Creates an Immediate Function Context                                 | No                             | Yes              |
| Can be used in Boolean expressions                                    | Yes                            | No               |
| Suitable for selecting execution paths                                | Yes                            | Yes              |
| Allows direct calls to immediate functions in the compile-time branch | No                             | Yes              |

### **An Important Note About if Conditions**

A subtle mistake when using `std::is_constant_evaluated()` is assuming that the following code always distinguishes compile-time from runtime execution:

```
if (std::is_constant_evaluated()) {
    // Compile-time branch
}
```

In fact, this condition may always evaluate to `true`.

The reason is that the condition itself is evaluated in a Manifestly Constant-Evaluated Context.

For example, evaluating the Boolean condition of an ordinary `if` statement can cause `std::is_constant_evaluated()` to report `true`, according to the language rules governing this context.

This behavior may appear inconsistent with the function's intended purpose, but it follows directly from the standard's rules.

Using `if consteval` eliminates this ambiguity because context detection is built into the language construct rather than depending on evaluating a Boolean expression.

## **Practical Applications of if consteval**

The real value of this feature becomes apparent when designing functions and libraries that support both evaluation contexts.

### **Selecting Different Algorithms for Compile-Time and Runtime**

Sometimes one algorithm is more suitable for constant evaluation, while another performs better at runtime.

For example, consider a function that calculates the power of an integer.

At compile time, an algorithm based on integer arithmetic can be used.

At runtime, an optimized standard library function may be preferable.

```
#include <cmath>
#include <cstdint>

constexpr std::uint64_t integerPower(
    std::uint64_t base,
    unsigned exponent
) {
    if consteval {
        std::uint64_t result = 1;

        while (exponent > 0) {
            if (exponent & 1U) {
                result *= base;
            }

            base *= base;
            exponent >>= 1U;
        }

        return result;
    }
    else {
        return static_cast<std::uint64_t>(
            std::pow(
                static_cast<double>(base),
                static_cast<double>(exponent)
            )
        );
    }
}

static_assert(integerPower(2, 10) == 1024);
```

In this example, the compile-time branch uses integer arithmetic.

The runtime branch uses `std::pow()`.

This example is intended for a limited range of input values. In real-world applications, integer overflow and the precision implications of floating-point conversions must also be handled.

The important point is that algorithm selection depends on the evaluation context rather than merely on whether the compiler can optimize a particular implementation.

### **Using consteval Functions in constexpr APIs**

One of the most important applications of `if consteval` is creating APIs that can be used in both compile-time and runtime contexts.

```
#include <cstddef>

consteval std::size_t compileTimeHash(const char* text) {
    std::size_t hash = 5381;

    while (*text != '\0') {
        hash = hash * 33 + static_cast<unsigned char>(*text);
        ++text;
    }

    return hash;
}

constexpr std::size_t hashText(const char* text) {
    if consteval {
        return compileTimeHash(text);
    }
    else {
        std::size_t hash = 5381;

        while (*text != '\0') {
            hash = hash * 33 + static_cast<unsigned char>(*text);
            ++text;
        }

        return hash;
    }
}

constexpr auto value = hashText("example");
```

In this design, `hashText` provides a public API that supports both evaluation contexts.

The compile-time path uses a `consteval` function.

The runtime path performs the calculation directly.

This pattern is useful when designing library APIs that need to use immediate functions.

However, this example contains two similar implementations. In real projects, it is preferable to place shared logic in a reusable function whenever possible to avoid code duplication.

### **Managing Operations That Cannot Be Used During Constant Evaluation**

Some operations are available at runtime but cannot be performed as part of a constant expression.

Examples include:

- Performing I/O operations.
- Accessing certain external resources.
- Using APIs that are not `constexpr`.
- Performing certain operating-system operations.
- Using resources that cannot be managed during constant evaluation.

These operations can be placed in the runtime branch.

```
#include <iostream>

constexpr void logMessage(int value) {
    if consteval {
        // No runtime I/O during constant evaluation
    }
    else {
        std::cout << "Value: " << value << '\n';
    }
}

int main() {
    constexpr int value = 10;

    logMessage(value);

    int runtimeValue = 20;

    logMessage(runtimeValue);
}
```

In this example, the first call is evaluated at compile time, and the I/O branch is not executed.

The second call occurs at runtime, so the message is printed.

Note that placing an operation in the runtime branch does not make that operation a valid constant expression. It simply ensures that the operation is not executed during compile-time evaluation.

### **Using if consteval in Generic Library Design**

Generic libraries often need to work in different execution contexts.

For example, an algorithm might:

- Use a simple, constant-evaluation-compatible implementation at compile time.
- Use a more optimized implementation at runtime.
- Rely on immediate functions for certain operations.

```
#include <cstddef>

consteval std::size_t compileTimeSum(
    const int* values,
    std::size_t size
) {
    std::size_t result = 0;

    for (std::size_t i = 0; i < size; ++i) {
        result += static_cast<std::size_t>(values[i]);
    }

    return result;
}

constexpr std::size_t sum(
    const int* values,
    std::size_t size
) {
    if consteval {
        return compileTimeSum(values, size);
    }
    else {
        std::size_t result = 0;

        for (std::size_t i = 0; i < size; ++i) {
            result += static_cast<std::size_t>(values[i]);
        }

        return result;
    }
}

constexpr int numbers[] = {1, 2, 3, 4, 5};

static_assert(sum(numbers, 5) == 15);
```

This example demonstrates how to provide a single API that selects an appropriate implementation for each evaluation context.

## **Important Language Rules and Limitations**

Despite its simple syntax, `if consteval` has several important rules that must be considered when designing APIs.

### **Compound Statements Are Required**

In the positive form of `if consteval`, the first branch must be a compound statement.

Therefore, the following code is invalid:

```
constexpr int calculate(int value) {
    if consteval
        return value * value;
    else
        return value + value;
}
```

The correct implementation is:

```
constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return value + value;
    }
}
```

The body of the main branch must also be a compound statement in the negated form:

```
constexpr int calculate(int value) {
    if !consteval {
        return value + value;
    }
    else {
        return value * value;
    }
}
```

### **Both Branches Must Be Semantically Valid**

Unlike `if constexpr`, the unselected branch of `if consteval` is not generally discarded.

For example:

```
constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return undefinedFunction(value);
    }
}
```

If `undefinedFunction` has not been declared, the code is invalid.

Even if the runtime branch is not executed during a particular constant-evaluation invocation, the compiler must still check it according to the applicable language rules.

This is one of the most important differences between `if consteval` and `if constexpr`.

### **Restrictions on Immediate Function Usage**

`if consteval` allows immediate functions to be called in the appropriate branch, but it does not remove the general restrictions associated with `consteval`.

For example, the result of constant evaluation must still satisfy the requirements for a constant expression.

Similarly, restrictions involving references, pointers, and constant-expression results must still be respected.

Therefore, `if consteval` is a mechanism for managing evaluation contexts, not a way to bypass constant-evaluation restrictions.

### **Control-Flow Restrictions**

The rules concerning labels, `goto`, and control-flow transfers also matter when using this construct.

For example, jumping into one of the branches of an `if consteval` statement through `goto` or `switch` is not permitted.

The rules governing `case`, `default`, and local labels must also be respected.

In general, it is preferable to avoid complicated control-flow transfers around this construct.

### **Cannot Be Used as an Expression**

`if consteval` is a statement, not an expression.

Therefore, it cannot be used like a conditional operator.

The following code is invalid:

```
constexpr int value =
    if consteval {
        10;
    }
    else {
        20;
    };
```

For such cases, use a `constexpr` function or an ordinary expression construct.

### **Not Restricted to constexpr Functions**

Using `if consteval` is not limited to `constexpr` functions.

It is a language feature for detecting evaluation contexts.

However, its greatest practical value is usually found inside `constexpr` functions, since these functions can be invoked during both compile-time and runtime evaluation.

## **Advanced Design and Implementation Considerations**

### **Distinguishing Branch Selection from Compiler Optimization**

Constant evaluation and constant folding must be distinguished.

Constant folding is an optimization that a compiler may perform to evaluate expressions whose results are known.

However, `if consteval` is part of the language rules.

Consequently, even if a compiler can calculate the result of a runtime call during optimization, this does not necessarily cause the compile-time branch to be selected.

For example:

```
constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return value + value;
    }
}

int main() {
    int result = calculate(5);
}
```

In this example, `result` is a runtime variable.

Even if the compiler can determine the final result through optimization, branch selection is governed by the language's evaluation-context rules, not by when the optimization takes place.

### **Avoiding Code Duplication with Shared Functions**

One potential problem when using `if consteval` is duplicating logic across the two branches.

To avoid this, shared functionality can be placed in separate functions.

```
constexpr int commonCalculation(int value) {
    return value * value + 10;
}

consteval int compileTimeCalculation(int value) {
    return commonCalculation(value);
}

constexpr int calculate(int value) {
    if consteval {
        return compileTimeCalculation(value);
    }
    else {
        return commonCalculation(value);
    }
}
```

In this design:

- The main calculation logic is implemented in `commonCalculation`.
- The `compileTimeCalculation` function is an immediate function.
- The `calculate` function selects the appropriate execution path.

This approach improves maintainability.

### **Considering Overflow and Resource Limits**

Constant evaluation has implementation and execution limits.

For example, compilers typically impose limits on the number of operations, recursion depth, and resources used during constant evaluation.

These limits may vary between compiler implementations.

Therefore, compile-time algorithms should also be evaluated in terms of time and memory complexity.

Additionally, using a different algorithm for compile-time execution does not guarantee better performance.

In some cases, the runtime algorithm may already be sufficiently efficient, and introducing a separate compile-time implementation may only increase project complexity.

### **Considering Side Effects**

Side effects are important when designing functions that support both evaluation contexts.

For example, suppose a function modifies a global variable in its runtime branch but performs no such modification during constant evaluation.

The function will then have different observable behavior in the two contexts.

This difference is not necessarily incorrect, but it should be part of the API contract.

It is recommended to make the observable behavior of a function as clear and predictable as possible in both contexts.

### **Using if consteval in Advanced Templates**

`if consteval` can be used alongside templates, concepts, and `if constexpr`.

The important point is that each feature handles a different type of decision:

- Concepts restrict the set of valid types and express requirements.
- `if constexpr` selects a branch based on a compile-time condition.
- `if consteval` selects a path based on the evaluation context.

Combining these features appropriately can lead to powerful and maintainable API designs.

## **Checking the Feature-Test Macro**

The compiler's support for this feature can be checked using its standard feature-test macro.

The macro is:

```
__cpp_if_consteval
```

Its standardized value is:

```
202106L
```

The following code can be used to check support:

```
#if defined(__cpp_if_consteval) && \
    __cpp_if_consteval >= 202106L

#define HAS_IF_CONSTEVAL 1

#else

#define HAS_IF_CONSTEVAL 0

#endif
```

This approach is useful for projects that need to support multiple compilers or compiler versions.

Note that the macro should be checked in the actual compilation environment. Merely selecting the `-std=c++23` option does not guarantee that every C++23 feature has been implemented in the compiler version being used.

## **Compiler Support**

`if consteval` is part of the C++23 standard.

Support for this feature depends on the compiler version.

To verify its implementation status, consult the official compiler documentation and feature-support tables.

Useful resources include:

- [GCC C++ Standards Support](https://gcc.gnu.org/projects/cxx-status.html?utm_source=chatgpt.com)
- [Clang C++ Support](https://clang.llvm.org/cxx_status.html?utm_source=chatgpt.com)
- [MSVC C++ Language Conformance](https://learn.microsoft.com/en-us/cpp/overview/visual-cpp-language-conformance?utm_source=chatgpt.com)

The following options can be used to test code with the corresponding compilers.

**GCC:**

```
g++ -std=c++23 -Wall -Wextra -pedantic main.cpp -o main
```

**Clang:**

```
clang++ -std=c++23 -Wall -Wextra -pedantic main.cpp -o main
```

**MSVC:**

```
cl /std:c++latest /EHsc /W4 main.cpp
```

Using `-std=c++23` or its equivalent selects the target language standard. However, the actual available features still depend on the compiler version.

## **Common Mistakes When Using if consteval**

In real-world projects, several common mistakes can lead to compilation errors or unexpected behavior.

### **Incorrectly Detecting Compile-Time Execution**

A common mistake is assuming that if a function receives a constant argument, its compile-time branch will necessarily be selected.

```
constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return value + value;
    }
}

int main() {
    int result = calculate(5);
}
```

In this example, the function call still occurs at runtime.

The fact that the argument has a constant value does not, by itself, determine which branch is selected.

### **Using if constexpr Instead of if consteval**

If the goal is to detect the evaluation context, using `if constexpr` with a constant or template-dependent condition is not an appropriate replacement.

These two features serve different purposes.

### **Assuming the Unselected Branch Is Discarded**

Some developers assume that the unselected branch is ignored in the same way as with `if constexpr`.

This assumption is incorrect.

Both branches must satisfy the applicable language rules.

### **Using Two Different Algorithms Unnecessarily**

Having two execution paths does not automatically mean better design.

If a single algorithm works well in both contexts, using one implementation is generally simpler and easier to maintain.

### **Using Runtime Operations in the Compile-Time Branch**

Placing a runtime operation in the branch selected during constant evaluation does not make that operation valid.

For example, I/O operations still cannot form part of a valid constant expression.

The compile-time branch must use only operations permitted during constant evaluation.

## **When Should You Use if consteval?**

This feature is recommended when a function genuinely needs different behavior depending on its evaluation context.

Appropriate use cases include:

- Designing `constexpr` functions with different algorithms for compile-time and runtime execution.
- Calling `consteval` functions from `constexpr` functions.
- Creating APIs that can be used in both compile-time and runtime contexts.
- Separating runtime operations from constant-evaluation paths.
- Developing advanced generic libraries.
- Using immediate-function-specific capabilities in the compile-time path.

### **When Should You Avoid It?**

In the following situations, it is generally better not to use this feature:

- When a single implementation is sufficient for both evaluation contexts.
- When the actual requirement is to inspect types or template parameters.
- When `if constexpr` provides a simpler and more precise solution.
- When only a Boolean indication of constant-evaluation status is needed.
- When its use introduces unnecessary code duplication or complexity.

In particular, if the only requirement is to determine whether constant evaluation is taking place and there is no need for an Immediate Function Context, `std::is_constant_evaluated()` remains an appropriate choice.

## **Comparing Related Features Across C++ Standards**

To understand the role of `if consteval`, it is useful to examine the evolution of constant-evaluation capabilities across different C++ standards.

| Feature                           | Standard | Purpose                                                                                |
| --------------------------------- | -------- | -------------------------------------------------------------------------------------- |
| `constexpr`                       | C++11    | Defines variables and functions that can be used in constant evaluation                |
| Expanded `constexpr` capabilities | C++14    | Provides greater flexibility when implementing constant-evaluation functions           |
| `if constexpr`                    | C++17    | Selects branches based on compile-time conditions                                      |
| `std::is_constant_evaluated()`    | C++20    | Detects constant evaluation                                                            |
| `consteval`                       | C++20    | Defines immediate functions                                                            |
| `if consteval`                    | C++23    | Selects branches based on evaluation context and creates an Immediate Function Context |

These features complement one another, and each solves a different problem.

Therefore, when designing modern C++ code, they should not be treated as direct replacements for one another.

## **Summary**

The `if consteval` feature introduced in C++23 is an important tool for controlling function behavior precisely across compile-time and runtime contexts.

Its most significant characteristic is that the constant-evaluation branch creates an Immediate Function Context.

This solves a problem that cannot be addressed by `std::is_constant_evaluated()` alone: directly calling `consteval` functions with values that are valid within an Immediate Function Context.

To use this feature effectively, developers should understand its differences from `if constexpr` and `std::is_constant_evaluated()`.

The key points are:

- `if consteval` was introduced in C++23.
- Branch selection depends on a Manifestly Constant-Evaluated Context.
- The compile-time branch is an Immediate Function Context.
- The negated form, `if !consteval`, is also supported.
- Unlike `if constexpr`, the unselected branch is not generally discarded.
- This feature is particularly useful for calling immediate functions from `constexpr` functions.
- `std::is_constant_evaluated()` remains useful when constant-evaluation status must be checked without requiring an Immediate Function Context.
- Its use should be driven by actual design requirements and should not introduce unnecessary complexity.

Overall, `if consteval` is an important feature of modern C++ for designing APIs that support both evaluation contexts. Together with `constexpr`, `consteval`, and `if constexpr`, it provides a powerful foundation for implementing modern and generic C++ libraries.

## **References and Further Reading**

The following resources are recommended for a deeper understanding of the standard and the behavior of this feature.

- **cppreference documentation:**
  - [if statement](https://en.cppreference.com/w/cpp/language/if?utm_source=chatgpt.com)
  - [consteval specifier](https://en.cppreference.com/w/cpp/language/consteval?utm_source=chatgpt.com)
  - [std::is_constant_evaluated](https://en.cppreference.com/w/cpp/types/is_constant_evaluated?utm_source=chatgpt.com)
- **Official standardization proposal:**
  - [P1938R3 — if consteval](https://www.open-std.org/JTC1/SC22/WG21/docs/papers/2021/p1938r3.html?utm_source=chatgpt.com)
- **C++ standardization documentation:**
  - [WG21 — C++ Standards Committee](https://www.open-std.org/jtc1/sc22/wg21/?utm_source=chatgpt.com)
- **Compiler feature documentation:**
  - [GCC C++ Status](https://gcc.gnu.org/projects/cxx-status.html?utm_source=chatgpt.com)
  - [Clang C++ Status](https://clang.llvm.org/cxx_status.html?utm_source=chatgpt.com)
  - [MSVC C++ Conformance](https://learn.microsoft.com/en-us/cpp/overview/visual-cpp-language-conformance?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>