<div align="center">

[🇺🇸 English](./return_type_deduction.md) | [🇮🇷 فارسی](../../fa/cpp14/return_type_deduction.md)

</div>

---

# Return Type Deduction in C++14

## Introduction

When designing C++ functions, the return type is part of the function interface. It affects how the result is used, overload resolution, value transfer, and even code readability. In many cases, explicitly writing the return type is necessary and useful. In others, the result type can be determined from a `return` expression or the function body. **Return type deduction**, introduced in C++14, allows the compiler to deduce a function's return type under specified conditions.

This feature is related to deducing variable types with `auto`, but it does not use exactly the same mechanism in every context. In particular, ordinary functions with deduced return types differ from functions using a trailing return type and from functions using `decltype(auto)`. Understanding these distinctions helps prevent unintended copies, lost references, and compilation errors.

## Fundamentals of Return Type Deduction

### The Feature Introduced in C++14

C++11 introduced `auto` for variable type deduction and also allowed trailing return types using `->`. For example, in C++11, the result type could be specified at the end of a function declaration:

```cpp
auto add(int a, int b) -> int {
    return a + b;
}
```

C++14 added the ability to omit an explicit return type and use `auto` as the ordinary return type:

```cpp
auto add(int a, int b) {
    return a + b;
}
```

In this example, the compiler deduces the type of the return expression `a + b`. Since adding two `int` values produces an `int`, the function's return type is also `int`.

### Why the Feature Was Added

This feature simplifies several common tasks:

- It avoids repeating the type in functions whose result type is already clear from the return expression.
- It simplifies writing wrappers and generic helper functions because the result type may depend on the argument types.
- In template code, the author does not need to reconstruct a potentially complicated type manually.
- When the type of a computed expression changes, the risk of a mismatch between the declared type and the actual result is reduced.

However, automatic deduction does not always improve readability. If the return type is an important part of an API contract, specifying it explicitly can make the interface clearer and more stable.

## Basic Deduction Rules with `auto`

### Deduction from `return` Expressions

For a function whose return type is `auto`, the type is deduced from the value-bearing `return` expressions:

```cpp
auto getValue() {
    return 42;
}
```

The return type of this function is `int`. The deduction rules are similar to those used for a variable declared with `auto`: top-level cv-qualifiers and references are generally not preserved in the deduced type.

```cpp
int globalValue = 10;

auto byValue() {
    return globalValue;
}

auto byReference() {
    return (globalValue);
}
```

Both functions above return `int`, not `int&`. Parentheses around the expression in the second function do not, by themselves, preserve the reference, because a function return type declared with `auto` follows the ordinary `auto` deduction rules.

If the goal is to return a reference, declare it explicitly or use `decltype(auto)` with a clear understanding of its rules:

```cpp
int globalValue = 10;

auto& referenceToValue() {
    return globalValue;
}
```

In this example, the return type is `int&`.

### Differences Between `auto` in Variables and Functions

For a local variable, `auto` deduces the type from the initializer:

```cpp
int value = 10;
auto copy = value;
```

For a function, `auto` deduces the type from its `return` expressions:

```cpp
auto makeValue() {
    return 10;
}
```

In both cases, `auto` deduction generally drops top-level references and cv-qualifiers. Functions have additional constraints, however, including the requirement that their value-bearing `return` expressions lead to a consistent deduced type.

## Multiple `return` Expressions and Type Consistency

### Consistent Deduced Types

In a function with an `auto` return type, the `return` expressions must produce compatible deduced types. For example:

```cpp
auto absoluteLike(int value) {
    if (value < 0) {
        return -value;
    }

    return value;
}
```

Both expressions have type `int`, so deduction succeeds.

The following example is not valid:

```cpp
auto choose(bool condition) {
    if (condition) {
        return 1;
    }

    return 2.0;
}
```

One expression has type `int` and the other has type `double`. The compiler does not arbitrarily choose a common type for this function; return type deduction fails. If the result must have a specific type, declare that type explicitly or make all paths return the same type:

```cpp
double choose(bool condition) {
    if (condition) {
        return 1;
    }

    return 2.0;
}
```

### Do Not Assume Implicit Conversions Will Be Applied

The existence of an implicit conversion from one type to another does not mean that `auto` deduction will apply that conversion to make all `return` expressions have the same type. Deduction examines the expression types according to the language rules, so pay attention to the exact type of each expression when designing a function.

### Paths Without a Value-Bearing `return`

A function with an `auto` return type cannot have one value-returning path and another path that leads to an incompatible type. If there is no value-bearing `return` expression, return type deduction for `auto` does not simply produce `void`; the function must obtain its type from an appropriate return expression. For a function that intentionally returns no value, write `void` explicitly:

```cpp
void logMessage() {
    // Perform logging.
}
```

Note that a function containing `return;` and returning no value is not equivalent to a function containing `return expression;`. Use `void` for functions that do not return a result.

## Return Type Deduction in Recursive Functions

### The Limitation with Direct Recursion

For a recursive function whose return type is `auto`, the compiler must be able to determine the type while analyzing the `return` expressions. If deducing the return type requires calling the function recursively, the type may not yet be known:

```cpp
auto factorial(int n) {
    if (n <= 1) {
        return 1;
    }

    return n * factorial(n - 1);
}
```

This pattern causes a problem in C++14 because the return type must be deduced from the `return` expressions, but one of those expressions depends on a call to a function whose return type is still being deduced. The usual solution is to specify the return type explicitly:

```cpp
int factorial(int n) {
    if (n <= 1) {
        return 1;
    }

    return n * factorial(n - 1);
}
```

This limitation should not be generalized to every form of recursion. If the type can be determined from an expression independent of the recursive call, the declaration and definition details still matter. Nevertheless, an explicit return type is often the simplest and clearest choice for recursive functions.

### Mutual Recursion

In mutual recursion, two functions call each other directly or indirectly. If both return types depend on deduction, determining the types can become difficult or impossible. Explicit return types in the necessary declarations remove this dependency:

```cpp
bool isEven(int n);
bool isOdd(int n);

bool isEven(int n) {
    return n == 0 ? true : isOdd(n - 1);
}

bool isOdd(int n) {
    return n == 0 ? false : isEven(n - 1);
}
```

## Differences Between `auto` and `decltype(auto)`

### Deduction with `auto`

A return type of `auto` uses the ordinary `auto` deduction rules and generally returns the result by value:

```cpp
int value = 7;

auto getCopy() {
    return value;
}
```

The result type is `int`. This behavior is appropriate for most computational functions and factories that transfer ownership of a value to the caller.

### Deduction with `decltype(auto)`

`decltype(auto)`, available since C++14, deduces the type according to the rules of `decltype`. Depending on the form of the expression, it can preserve a reference:

```cpp
int value = 7;

decltype(auto) getReference() {
    return (value);
}
```

In this example, `(value)` is an lvalue, and `decltype((value))` is `int&`; therefore, the function returns a reference.

By contrast:

```cpp
int value = 7;

decltype(auto) getValue() {
    return value;
}
```

Here, because the operand of `decltype` is the unparenthesized name of a variable, `decltype(value)` is the variable's declared type, `int`. The function therefore returns a value, not a reference.

### Why Parentheses Matter

The special rule for an unparenthesized variable name in `decltype` differs from the rules for other expressions. For an lvalue expression such as `(value)`, `decltype` produces an lvalue reference type. Consequently, adding or removing parentheses in code using `decltype(auto)` can change the return type:

```cpp
int value = 7;

decltype(auto) byValue() {
    return value;
}

decltype(auto) byReference() {
    return (value);
}
```

The first function returns `int`, while the second returns `int&`. This is one of the most important sources of mistakes when using `decltype(auto)`.

### Choosing Between Them

- Use `auto` when the goal is to return an independent value.
- Use `decltype(auto)` when the exact type of the expression, including whether it is a reference, must be preserved.
- For public APIs, an explicit return type is often clearer and more stable when the result type or ownership is part of the contract.
- Do not use `decltype(auto)` merely to shorten a declaration: a seemingly small change to a `return` expression can change the function's contract.

## Return Type Deduction in Lambda Expressions

### The Feature in C++14

In C++11, a lambda could use a trailing return type or, in simple cases, rely on limited return type deduction. C++14 expanded return type deduction for lambdas, so the return type can usually be omitted:

```cpp
auto square = [](int value) {
    return value * value;
};
```

The return type of this lambda is deduced from the expression `value * value`.

### Consistency Across Different Paths

As with ordinary functions, the `return` expressions in a lambda must have consistent types:

```cpp
auto selectValue = [](bool condition) {
    if (condition) {
        return 10;
    }

    return 20;
};
```

This example is valid because both paths return `int`. If one path returned `int` and the other returned `std::string`, ordinary `auto` deduction would fail.

### Use with Algorithms

Return type deduction makes lambda expressions simpler when used with Standard Library algorithms:

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4};

    std::transform(
        values.begin(),
        values.end(),
        values.begin(),
        [](int value) {
            return value * value;
        }
    );
}
```

In this example, the lambda produces an `int` for each input, and there is no need to write a separate return type.

## Applications in Templates and Generic Code

### Simplifying Functions Whose Results Depend on Argument Types

In function templates, the result type may follow from an operation performed on the arguments. Using `auto` can avoid repeating the expression's type:

```cpp
template <typename T, typename U>
auto addValues(const T& left, const U& right) {
    return left + right;
}
```

The return type is deduced from the expression `left + right`. For example, combining `int` and `double` usually produces `double`. The actual type depends on the input types and the overload selected for the operator.

### Differences from a Trailing Return Type

In C++11, the result type of a function template could be expressed with `decltype` and a trailing return type:

```cpp
template <typename T, typename U>
auto addValues(const T& left, const U& right)
    -> decltype(left + right) {
    return left + right;
}
```

In C++14, the version using `auto` is shorter. However, a trailing return type remains useful when the return type needs to be specified explicitly or when a declaration needs a dependent type before the function body is available.

### Limitations of Declarations Without Definitions

For a function that is only declared and whose body is unavailable, the compiler cannot deduce an `auto` return type from `return` expressions. For example, the following interface is insufficient when the caller needs to know the result type:

```cpp
auto calculate(int value);
```

Until the function definition is available, the return type is unknown, and uses that require the type cannot be compiled. For APIs split between header and source files, it is generally preferable to declare the return type explicitly or make the definition available where deduction is required.

## Differences Between Return Type Deduction and `decltype`

### The Behavior of `decltype(expression)`

The expression `decltype(expression)` determines the exact type of an expression according to its own rules. In summary:

- If `expression` is the unparenthesized name of a variable or member, `decltype` yields its declared type.
- If the expression is an lvalue, the result is generally an lvalue reference type.
- If the expression is an xvalue, the result is generally an rvalue reference type.
- If the expression is a prvalue, the result is the type of that expression.

These rules matter in templates and when designing wrappers that must preserve an expression's value category and reference-ness.

### A Practical Comparison

```cpp
int value = 5;

auto a() {
    return (value);
}

decltype(auto) b() {
    return (value);
}

auto c() -> decltype((value)) {
    return (value);
}
```

In this example, `a` returns an `int` value, while `b` and `c` return `int&`. This demonstrates that `auto`, `decltype(auto)`, and `decltype` are related mechanisms, but they are not interchangeable.

## Returning References and Object Lifetime

### The Danger of References to Local Variables

Correct type deduction alone does not guarantee safety. Returning a reference to a local variable that is destroyed when the function exits creates an invalid reference:

```cpp
auto& invalidReference() {
    int localValue = 42;
    return localValue;
}
```

This function returns a reference to a local object whose lifetime ends when the function returns. Using the resulting reference has undefined behavior. Compilers may issue a warning, but code should not rely on warnings to prevent this problem.

### The Risk of `decltype(auto)` in Wrappers

In wrappers, `decltype(auto)` can preserve the reference returned by a called function. This is desirable in some cases, but it can be dangerous if the result refers to a temporary or local object. Lifetime guarantees must be considered separately from type deduction.

```cpp
decltype(auto) identity(int& value) {
    return (value);
}
```

This function returns a reference to the existing argument and can be appropriate as long as the original object remains alive. However, a version that accepts the argument by value and then returns a reference to it is unsafe:

```cpp
decltype(auto) invalidIdentity(int value) {
    return (value);
}
```

In this version, the result is `int&`, referring to a local parameter. The reference dangles after the function returns.

### Choosing Ownership Deliberately

When designing a function, decide whether the result should be an independent value, a reference to an existing object, or a reference to a temporary object. `auto` is often the simpler and safer choice when returning a value. Use `auto&`, `auto&&`, and `decltype(auto)` when preserving a reference is an intentional part of the function's contract.

## Return Type Deduction Versus `auto` Parameters

### Function Parameters in C++14

Return type deduction with `auto` should not be confused with `auto` parameters. In C++14, ordinary functions cannot declare their parameters using `auto` alone. For example, the following is not a valid ordinary function in C++14:

```cpp
auto add(auto left, auto right) {
    return left + right;
}
```

Deduced parameter types in this form are associated with **generic lambdas** in C++14, not ordinary functions. For a generic ordinary function, use a template:

```cpp
template <typename T, typename U>
auto add(T left, U right) {
    return left + right;
}
```

Abbreviated function templates with `auto` parameters were introduced in C++20 and should not be attributed to C++14.

## Limitations and Common Mistakes

### Returning Inconsistent Types

If different paths through a function produce different types, ordinary `auto` deduction fails. Specify the output type explicitly or convert the expressions to a common, intended type.

### Assuming `auto` Preserves References

A return type of `auto` does not automatically preserve a reference. If an API must return a reference, state that explicitly or use `decltype(auto)` with a full understanding of the `decltype` rules.

### Relying on Deduction in Public Declarations

For a function whose prototype alone appears in a header, `auto` is generally not a suitable choice because the caller needs access to the definition to deduce the result type.

### Forgetting Lifetime

Even when the deduced return type is exactly the intended type, a reference to a destroyed object is still invalid. Lifetime analysis is separate from type analysis.

### Confusing `auto` with `decltype(auto)`

These two forms do not always deduce the same type. With `decltype(auto)`, the difference between `value` and `(value)` can change the return type from `int` to `int&`.

### Using Deduction Without a Clear Reason

Deduction helps reduce repetition and maintain type consistency. However, in public APIs, ABI-sensitive interfaces, or code where the result type is part of the documented contract, an explicit return type may be preferable. In addition, changing the body of a function with a deduced return type can unintentionally change its API type.

## Comparison of Return Type Declaration Techniques

| Technique | Standard | Main Behavior | Appropriate Use |
|---|---|---|---|
| Explicit type, such as `int` | Before C++11 | Specifies the contract type directly | Public APIs and stable contracts |
| `auto` return type | C++14 | Deduces the type from `return` expressions using `auto` deduction rules | Computational functions and templates |
| Trailing return type, such as `auto f() -> T` | C++11 | Places the return type after the parameter list | Dependent types and explicit declarations |
| `decltype(auto)` | C++14 | Deduces the type according to `decltype` rules | Wrappers and cases where references must be preserved |

## Choosing an Approach in Real Projects

### When to Use `auto`

Use `auto` when the result type is clear from the return expression, when different input types in a template can produce different output types, or when repeating the type would be error-prone. This approach is especially useful for internal helper functions and lambdas.

### When to Choose an Explicit Type

Prefer an explicit type when the return type is part of a public contract, the definition is separated from the declaration, API stability matters, or readers should be able to identify the result type immediately from the function signature. An explicit type also often removes deduction ambiguity in recursive and mutually recursive functions.

### When to Use `decltype(auto)`

Use `decltype(auto)` when the exact type of an expression must be preserved, especially in forwarding wrappers or functions that should return the result of another call without changing its value category. Use it carefully, with close attention to object lifetime and the rules of `decltype`.

## Summary

Return type deduction in C++14 allows a function's return type to be deduced from its `return` expressions, simplifying generic functions, lambdas, and helper functions. However, `auto` does not automatically preserve references, and multiple `return` expressions must have consistent types. Also, a function declaration without an accessible definition cannot provide its deduced return type to callers.

To choose correctly, distinguish among three goals: **simplifying declarations**, **making the contract clear**, and **preserving the exact type of an expression**. For many ordinary functions, `auto` is sufficient. For stable interfaces, an explicit type is often clearer. For wrappers that need to preserve references, `decltype(auto)` can be appropriate, provided object lifetime and the rules of `decltype` are carefully considered.


---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>