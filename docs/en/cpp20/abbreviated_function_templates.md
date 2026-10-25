<div align="center">

[🇺🇸 English](./abbreviated_function_templates.md) | [🇮🇷 فارسی](../../fa/cpp20/abbreviated_function_templates.md)

</div>

---

# Introduction to Abbreviated Function Templates in C++20

## Introduction

**Abbreviated Function Templates** are an important feature of C++20 that makes writing Generic functions simpler and more readable. This feature allows programmers to use `auto` in a function's parameter list instead of explicitly declaring `template` parameters. The compiler creates a corresponding Template parameter for each parameter declared with `auto`.

This feature is particularly useful in helper functions, Generic algorithms, Range processing, functions based on `Concepts`, and code that works with different data types.

Consider the following function:

```cpp
template <typename T>
T add(T lhs, T rhs) {
    return lhs + rhs;
}
```

Using Abbreviated Function Templates, we can write it as follows:

```cpp
auto add(auto lhs, auto rhs) {
    return lhs + rhs;
}
```

Both definitions use Templates to generate appropriate function specializations based on the argument types. However, they are not semantically identical. In the second version, each `auto` parameter introduces an independent Template parameter. Therefore, `lhs` and `rhs` can have different deduced types, and the return type is deduced using the rules for `auto`.

To understand this feature correctly, we need to examine its differences from traditional Function Templates, the role of `Concepts`, type deduction, language limitations, and practical applications.

## Understanding Abbreviated Function Templates

### Basic Definition

In C++20, when `auto` is used as a parameter type in a function's parameter list, the function is defined as a Function Template.

For example, the following code:

```cpp
void printValue(auto value) {
    // Process value
}
```

is conceptually similar to the following definition:

```cpp
template <typename T>
void printValue(T value) {
    // Process value
}
```

In both cases, the parameter type is deduced from the supplied argument when the function is called.

An important point is that `auto` does not mean that a local variable is being declared here. In a function's parameter list, it acts as a Placeholder Type that introduces a Template parameter.

### Why Was This Feature Introduced?

Before C++20, defining a Generic function usually required explicitly declaring its Template parameters:

```cpp
template <typename T>
void process(const T& value) {
    // Process value
}
```

For a simple function with only one or two Template parameters, this structure could seem unnecessarily verbose.

Abbreviated Function Templates eliminate this repetitive syntax:

```cpp
void process(const auto& value) {
    // Process value
}
```

The main advantages of this approach include:

- Reducing Boilerplate and shortening Generic function definitions.
- Improving readability when functions have simple Template parameters.
- Allowing `Concepts` to be used directly in parameter lists.
- Better alignment with modern Generic Programming conventions.
- Simplifying the definition of helper functions for algorithms, Ranges, and different Types.

However, this feature does not replace all traditional Function Templates. The traditional approach is still necessary when explicitly naming Template parameters, defining a return type in terms of a particular parameter, or declaring Templates with multiple dependent parameters.

## Differences from Traditional Function Templates

### Comparing the Syntax

Consider the following two definitions:

```cpp
template <typename T>
void display(const T& value) {
    // Display value
}
```

```cpp
void display(const auto& value) {
    // Display value
}
```

The second definition is an Abbreviated Function Template. In both cases, the function parameter is a Reference to `const`, and the underlying type is deduced from the argument.

However, in the second definition, the Template parameter does not have an explicitly accessible name. This is not a problem for simple parameters, but it becomes a limitation when the return type, another parameter, or another Template must depend on that same type.

For example, in the following definition, `T` can be used throughout the declaration:

```cpp
template <typename T>
T identity(T value) {
    return value;
}
```

However, we cannot obtain the name `T` simply by writing `auto` in the function parameter:

```cpp
auto identity(auto value) {
    return value;
}
```

In the second definition, the return type is deduced using the rules for `auto`. If the return type must explicitly depend on the Template parameter's type, the traditional approach is more suitable.

### Differences Between Template Parameters

In the following traditional definition, both function parameters use the same Template type:

```cpp
template <typename T>
auto add(T lhs, T rhs) {
    return lhs + rhs;
}
```

Consequently, ordinary deduction for this function will fail when arguments have different types, such as `int` and `double`, because a single `T` must be compatible with both arguments.

However, in the following definition:

```cpp
auto add(auto lhs, auto rhs) {
    return lhs + rhs;
}
```

Each `auto` parameter introduces an independent Template parameter. Therefore, the function can accept arguments of different types, provided that the expression `lhs + rhs` is valid.

For example:

```cpp
#include <iostream>

int main() {
    std::cout << add(10, 2.5) << '\n';
}
```

With the second definition, this code produces the result `12.5`.

If we want both parameters to have the same type, we can use an explicit Template parameter or, where appropriate, use `Concepts` to enforce the necessary constraints.

## How Parameter Type Deduction Works

### The Rules for `auto` Deduction

`auto` parameters in a function's parameter list follow Template deduction rules. Understanding these rules is essential for avoiding unintended copies, preserving References, and managing `const` correctly.

For example:

```cpp
void byValue(auto value) {
    // value is a separate object
}
```

This function receives its parameter by Value. Consequently, deduction rules similar to those for by-value Template parameters apply, and the argument is ordinarily copied or moved into the function parameter.

In contrast, the following definition uses a Reference:

```cpp
void byReference(auto& value) {
    // value refers to the original object
}
```

This definition cannot be used with temporary arguments because a non-`const` Lvalue Reference cannot bind to an Rvalue.

We can also use `const auto&`:

```cpp
void inspect(const auto& value) {
    // Read value without copying it
}
```

This form is highly suitable for inspecting large data objects, Containers, and Types that are expensive to copy.

### Using `const auto&`

In many Generic functions, receiving a parameter as `const auto&` is a suitable choice:

```cpp
#include <iostream>
#include <string>

void print(const auto& value) {
    std::cout << value << '\n';
}

int main() {
    print(42);
    print(std::string{"Modern C++"});
}
```

In this example, the function accepts different data types without copying the arguments, provided that the corresponding types support insertion into an output stream.

Using `const auto&` does not mean that every Generic function should have this signature. If the function needs to create an independent copy of the data or take ownership of a value, passing by Value or using `auto&&` may be more appropriate.

### Using `auto&&`

A parameter declared as `auto&&` creates a Forwarding Reference:

```cpp
void forwardValue(auto&& value) {
    // value can bind to lvalues and rvalues
}
```

In this case, the parameter type is determined by the argument type and Reference Collapsing rules.

For example, if the argument is an Lvalue, the parameter becomes an Lvalue Reference. If the argument is an Rvalue, the parameter becomes an Rvalue Reference.

This approach is useful for writing Wrappers and functions that need to preserve an argument's Value Category. However, receiving a parameter as `auto&&` does not, by itself, constitute Perfect Forwarding. To correctly forward the Value Category, we must use `std::forward`:

```cpp
#include <utility>

void wrapper(auto&& value) {
    process(std::forward<decltype(value)>(value));
}
```

In this example, `process` is a hypothetical function that must be defined in the actual application.

Note that `decltype(value)` also preserves the Reference type in this context, which is essential for correct forwarding.

## Differences Between `auto`, `auto&`, and `auto&&`

The parameter declaration affects how the argument binds to the parameter and how the function behaves.

| Parameter form | Main behavior | Typical use |
|---|---|---|
| `auto value` | Pass by Value | When an independent copy of the argument is needed |
| `auto& value` | Pass by a modifiable Lvalue Reference | Directly modifying the original object |
| `const auto& value` | Pass by a read-only Reference | Inspecting data without the usual copying overhead |
| `auto&& value` | Forwarding Reference | Wrappers and Generic Forwarding |
| `const auto value` | Pass by Value with a `const` local parameter | Special cases where modifying the parameter must not be allowed |

One subtle point is that `const auto value` generally does not guarantee that the original argument is `const`. Instead, it makes the local parameter `const`. If the argument is a copyable object, an independent copy is created, and preventing modifications to the parameter does not prevent modifications to the original object.

## Using Concepts in Abbreviated Function Templates

### The Role of Concepts in Constraining Types

The `Concepts` feature in C++20 allows us to express syntactic and semantic constraints on Template types.

Instead of allowing a function to be instantiated for every type and encountering complicated errors inside the function body when an operation is invalid, we can specify the constraints directly in the function signature.

For example:

```cpp
#include <concepts>

void increment(std::integral auto& value) {
    ++value;
}
```

In this example, the parameter accepts only types that satisfy `std::integral`. This Concept is defined in the `<concepts>` header.

This constraint includes standard integral types and compatible user-defined integral types, but does not accept floating-point types.

### Using Custom Concepts

We can define a custom Concept to make the constraints required by a function clearer:

```cpp
#include <concepts>

template <typename T>
concept Addable = requires(const T& lhs, const T& rhs) {
    lhs + rhs;
};

auto add(const Addable auto& lhs, const Addable auto& rhs) {
    return lhs + rhs;
}
```

This example does not correctly express the intended constraint because the two `Addable` parameters are checked independently, and this Concept does not guarantee that the `+` operator is valid for combining the type of the first argument with the type of the second argument.

To express this requirement correctly, we must examine the relationship between the two argument types. One suitable approach is to use explicit Template parameters:

```cpp
template <typename T, typename U>
concept AddableWith = requires(const T& lhs, const U& rhs) {
    lhs + rhs;
};

template <typename T, typename U>
    requires AddableWith<T, U>
auto add(const T& lhs, const U& rhs) {
    return lhs + rhs;
}
```

In this definition, the relationship between the two types is checked explicitly.

We can also use a custom Concept with an Abbreviated Function Template when the constraint applies independently to each parameter. The choice between these approaches depends on whether the constraints concern each type individually or whether the relationship between multiple types matters.

### Using `std::same_as`

Sometimes, we need two parameters to have the same type. In such situations, `std::same_as` can be useful for enforcing constraints:

```cpp
#include <concepts>

void compare(std::same_as<int> auto lhs,
             std::same_as<int> auto rhs) {
    // Both parameters must satisfy the int constraint
}
```

This example is suitable only for parameters whose deduced types satisfy the constraint requiring `int`. However, if the goal is to require both parameters to have the same arbitrary type, using `std::same_as` in this way is insufficient because the expected type has already been fixed to `int`.

To enforce a shared, arbitrarily deduced type, it is better to use an explicit Template parameter:

```cpp
template <typename T>
void compare(T lhs, T rhs) {
    // Both parameters use the same deduced type
}
```

This distinction demonstrates that Abbreviated Function Templates are suitable for expressing independent constraints, whereas constraints involving a shared type are generally expressed more clearly using traditional Templates.

## Using Trailing Return Types

### Specifying the Return Type

In Abbreviated Function Templates, the return type can be deduced using `auto`:

```cpp
auto multiply(auto lhs, auto rhs) {
    return lhs * rhs;
}
```

The compiler deduces the return type from the `return` expressions.

If the return type must be specified explicitly, we can use a Trailing Return Type:

```cpp
auto convert(auto value) -> long {
    return static_cast<long>(value);
}
```

This form is particularly useful when the return type is independent of the parameter types or when the return type needs to be clearly visible in the function's Interface.

### Limitations on Accessing Parameter Types

One important difference between traditional Templates and Abbreviated Function Templates is that the latter do not expose explicitly named Template parameters.

For example, in the following definition, the type `T` can be used directly in the return type:

```cpp
template <typename T>
auto identity(T value) -> T {
    return value;
}
```

However, in the following definition, `T` is not a known name:

```cpp
auto identity(auto value) {
    return value;
}
```

If the return type must be exactly the same as the Template parameter type, deduction with `auto` is usually sufficient. However, in more complex cases, particularly when using `decltype`, `std::conditional_t`, or type Traits, explicitly defining the Template parameters may be clearer and more appropriate.

## Differences from Generic Lambdas

### The Relationship Between the Two Features

Generic Lambdas were introduced in C++14 and allow Lambda parameters to be declared using `auto`:

```cpp
auto lambda = [](auto value) {
    return value;
};
```

In contrast, Abbreviated Function Templates in C++20 provide the same style for ordinary functions:

```cpp
auto identity(auto value) {
    return value;
}
```

Both features use Templates to support different types. However, a Lambda also creates a Closure Type.

### When Should We Use a Lambda?

A Generic Lambda is suitable when the function's behavior is defined locally, for example, when used with algorithms:

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    std::ranges::for_each(values, [](auto value) {
        return value * 2;
    });
}
```

In this example, using a Lambda to define local behavior is appropriate. However, because `std::ranges::for_each` does not use the Lambda's return value to replace the elements, a different algorithm or a loop is needed if the goal is to modify the elements.

In contrast, when a function needs to provide an independent, reusable Interface, defining a Function Template or an Abbreviated Function Template is generally a more appropriate choice.

## Differences from `auto` in Non-Template Parameters

### Using `auto` in Variable Declarations

Since C++11, `auto` has been available for deducing variable types:

```cpp
auto value = 42;
```

Here, `auto` deduces the variable's type from its initializer.

However, in the following function definition:

```cpp
void process(auto value) {
    // Process value
}
```

`auto` introduces a Function Template.

These two uses employ the same Placeholder Type, but their positions and roles differ. In a variable declaration, `auto` determines a deduced type for that variable. In a function parameter list, `auto` introduces a Template parameter.

## Practical Applications in API Design

### Defining Helper Functions

In large projects, helper functions are often used to process different kinds of data. Abbreviated Function Templates can simplify the signatures of these functions:

```cpp
#include <iostream>

void logValue(const auto& value) {
    std::cout << value << '\n';
}
```

This function can be used with types that support output stream insertion.

When designing a public library Interface, it is better to express the necessary constraints using `Concepts` so that errors caused by incompatible types can be detected earlier and reported more precisely.

### Processing Ranges

In modern C++, functions that operate on Ranges often need to work with different Container types. For example:

```cpp
#include <iostream>
#include <ranges>
#include <vector>

void printRange(const auto& range) {
    for (const auto& value : range) {
        std::cout << value << ' ';
    }

    std::cout << '\n';
}

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};
    printRange(values);
}
```

In this example, the function is defined without depending on a specific Container type. However, its actual requirements include support for range-based `for` loops and the ability to insert the elements into an output stream.

If we want to express these requirements in the Interface, we can use `std::ranges::input_range`:

```cpp
#include <iostream>
#include <ranges>

void printRange(const std::ranges::input_range auto& range) {
    for (const auto& value : range) {
        std::cout << value << ' ';
    }

    std::cout << '\n';
}
```

This constraint guarantees that the type satisfies the `Input Range` Concept. However, it still does not guarantee that the Range's elements can be inserted into `std::cout`.

### Using Callback Functions and Algorithms

In APIs that accept Callbacks, Abbreviated Function Templates can be useful for defining independent Generic functions. However, if the Callback type is part of a complex Interface or multiple Template types must be related to one another, explicit Templates are generally more appropriate.

For example:

```cpp
template <typename F, typename T>
auto apply(F&& function, T&& value) {
    return std::forward<F>(function)(
        std::forward<T>(value)
    );
}
```

This function uses explicit Templates because it needs to name `F` and `T` independently and use them with `std::forward`.

If the goal is simply to perform a straightforward operation on a parameter, an Abbreviated Function Template may be more readable. However, the traditional approach provides more control for complex forwarding operations.

## Advanced Considerations for Template Parameters

### Multiple Independent Parameters

Each `auto` parameter in a function signature introduces an independent Template parameter:

```cpp
auto combine(auto first, auto second, auto third) {
    return first + second + third;
}
```

This function can accept three arguments of different types, provided that the addition expression is valid for the deduced types.

In contrast, if all arguments must have the same type, using a shared Template parameter is more appropriate:

```cpp
template <typename T>
auto combine(T first, T second, T third) {
    return first + second + third;
}
```

These two definitions differ in their type deduction rules and in the sets of arguments they accept.

### Dependencies Between Parameters

Sometimes, the type of one parameter must depend on the type of another parameter. In such cases, explicit Templates provide a clearer solution:

```cpp
template <typename T, typename U>
auto makePair(T first, U second) {
    return std::pair<T, U>{first, second};
}
```

In this example, using `T` and `U` to define the return type is essential. Although `auto` can be used for the return type in some cases, explicitly naming the Template parameters is generally simpler when constructing a type that directly depends on them.

### Non-Type Template Parameters

Abbreviated Function Templates are intended for deducing the types of function parameters and do not replace all Template features.

For example, if a function needs a non-type parameter such as an array's size, we can use an explicit Template parameter:

```cpp
template <typename T, std::size_t N>
void processArray(const T (&array)[N]) {
    // Process array
}
```

In this example, `N` is a Non-type Template Parameter that stores the number of elements in the array.

Using `auto` in the function's parameter list cannot replace the declaration of this non-type parameter.

## Limitations and Cases Where This Feature Is Unsuitable

### No Explicit Names for Template Parameters

In Abbreviated Function Templates, Template parameters are introduced implicitly. Therefore, we cannot refer to them using names such as `T` or `U`.

If the function's implementation needs to refer to a type by name, traditional Templates are a more appropriate choice.

### Limitations in Expressing Relationships Between Types

When the second parameter must have exactly the same type as the first parameter, or the return type must be determined by a relationship between multiple types, explicit Templates are generally clearer.

In these situations, using Abbreviated Function Templates may require complicated constraints that ultimately make the code harder to understand than the traditional approach.

### Limitations of Partial Specialization

Function Templates do not directly support Partial Specialization. This limitation existed before C++20.

If a function's behavior must change depending on a particular category of types, Overloading, `Concepts`, `if constexpr`, or helper Class Templates are commonly used instead.

### Limitations in Precisely Controlling the Return Type

Return types deduced using `auto` follow their own rules. If the return type must be exactly a particular type or its Reference qualification must be preserved, the function must be designed carefully.

For example, in this definition:

```cpp
auto getValue(auto& value) {
    return value;
}
```

The return type is generally deduced without a Reference, so the Reference nature of the parameter is not preserved in the return type.

If the goal is to return a Reference, we can use `decltype(auto)`:

```cpp
decltype(auto) getValue(auto& value) {
    return value;
}
```

This form deduces the return type according to the rules of `decltype`. In this example, the Reference nature of the returned value is preserved.

However, using `decltype(auto)` requires care because small differences in the `return` expression can change the deduced return type.

## Common Mistakes and How to Avoid Them

### Incorrectly Assuming That Parameter Types Must Match

A common mistake is assuming that two `auto` parameters must have the same type.

However, in the following definition:

```cpp
auto calculate(auto lhs, auto rhs) {
    return lhs + rhs;
}
```

The two parameters are independent and may have different types.

If a shared type is required, this constraint must be explicitly enforced in the function's design.

### Incorrect Use of References

Using `auto value` can result in copying the data. This may cause problems for large or non-copyable types.

To avoid unintended copies, we can use `const auto&`. However, if the function needs to modify the data, `auto&` is more appropriate.

### Assuming That `auto&&` Always Performs Perfect Forwarding

A parameter declared as `auto&&` can accept both Lvalues and Rvalues, but `std::forward` is required to correctly preserve and forward the Value Category.

Using `std::move` in every situation is not correct because it generally converts its argument into an Rvalue, potentially changing the expected behavior for Lvalues.

### Overusing Abbreviated Function Templates

Shorter code does not always mean better design. If a function requires complex constraints, a dependent return type, or relationships between multiple types, traditional Templates are generally clearer.

The best choice is the approach that makes the function's Interface clearer and expresses its constraints in the most understandable way.

## Compiler Compatibility Considerations

Abbreviated Function Templates are part of the C++20 standard. To use them, a compiler that supports this part of the standard is required.

With common compilers, enabling C++20 mode typically involves the following options:

```bash
g++ -std=c++20 main.cpp -o main
```

```bash
clang++ -std=c++20 main.cpp -o main
```

```bat
cl /std:c++20 /EHsc main.cpp
```

The options above configure the language standard for the corresponding compilers. However, the level of support for C++20 features depends on the compiler version and build environment.

For real-world projects, it is preferable to use recent compiler versions and verify support for the required features against the official documentation for the specific implementation.

## Summary

Abbreviated Function Templates in C++20 provide a simple and readable way to define Generic functions. By using `auto` in function parameters, we can benefit from Template functionality and type deduction without explicitly writing `template <typename T>` for each parameter.

This feature is particularly useful for helper functions, Range processing, Generic algorithms, and simple APIs. Combining it with `Concepts` can also make function constraints clearer.

However, Abbreviated Function Templates are not a complete replacement for traditional Templates. When explicitly naming Template parameters, expressing relationships between multiple types, precisely controlling the return type, or declaring non-type parameters is necessary, traditional Templates are generally more appropriate.

In professional C++ design, the most important principle is to maintain a balance between syntactic brevity, readability, type control, and Interface clarity. Abbreviated Function Templates provide the greatest value when they simplify Generic code without concealing important complexities.

---

*Editorial note from the source document: Refine the custom Concepts example and complete the constraints for `auto` parameters.*

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>