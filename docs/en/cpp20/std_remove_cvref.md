<div align="center">

[🇺🇸 English](./std_remove_cvref.md) | [🇮🇷 فارسی](../../fa/cpp20/std_remove_cvref.md)

</div>

---

# Introduction to `std::remove_cvref` in C++20

## Introduction

In C++ Generic Programming, it is sometimes necessary to inspect a type obtained from a template parameter or an expression without considering whether it is `const`, `volatile`, or a reference type. For example, a template function may need to determine whether an argument is of type `Widget`, regardless of whether it is passed as `Widget`, `const Widget&`, or `Widget&&`.

The `std::remove_cvref` type trait, introduced in the C++20 standard library, is designed precisely for this purpose. It first removes the reference from a type and then removes the top-level `const` and `volatile` qualifiers from the resulting type.

This small utility is particularly useful in template-based code, type traits, concepts, and perfect forwarding. However, it is not equivalent to `std::decay` and should not be considered a universal replacement for it.

## Definition and Overview

The header required to use this feature is `<type_traits>`:

```
#include <type_traits>
```

The standard library provides two related forms:

- `std::remove_cvref<T>` is a class template that exposes the transformed type through its `type` member.
- `std::remove_cvref_t<T>` is an alias template that directly refers to the transformed type.

For example:

```
#include <type_traits>

static_assert(std::is_same_v<
    std::remove_cvref<const int&>::type,
    int
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<volatile int&&>,
    int
>);
```

In most everyday code, the shorter form, `std::remove_cvref_t`, is more readable. The class template form is useful when working with the structure of a type trait or when its `type` member is specifically needed.

## Why Does This Feature Exist?

### The Problem of Removing References and CV-Qualifiers

In templates, the type deduced from a parameter is not necessarily the same as the underlying type that matters semantically to the program. For example, in a function with a forwarding reference, different arguments can result in different template parameter types:

```
template <typename T>
void inspect(T&& value) {
    // T depends on the argument's value category and cv-qualification.
}
```

If the function is called with an lvalue of type `const int`, `T` is deduced as `const int&`. Consequently, the parameter type `T&&` becomes `const int&` after reference collapsing. If the same function is called with an rvalue of type `int`, `T` is deduced as `int`, and the parameter type becomes `int&&`.

Sometimes, the function's logic does not depend on these differences and only needs to inspect the underlying type. In such situations, `std::remove_cvref_t<T>` provides a standard and explicit way to normalize the type.

### Relationship with Generic Programming

In Generic Programming, an algorithm may work with different argument types while making decisions based on their types without references or top-level CV-qualifiers. Removing these characteristics can simplify `static_assert` expressions, overload selection, and type-based constraints.

An important point is that removing CV-qualifiers and references is only a type-level operation. This transformation does not create a new object, remove `const` from the original object, or change runtime behavior.

## How It Works and the Transformation Order

Conceptually, the behavior of `std::remove_cvref<T>` is equivalent to combining two type traits:

```
std::remove_cv_t<std::remove_reference_t<T>>
```

First, `std::remove_reference_t` removes the reference. Then, `std::remove_cv_t` removes the CV-qualifiers from the resulting type.

The order of these operations matters. For example, when dealing with `const int&`, removing CV-qualifiers directly from the reference type does not remove the `const` qualifier belonging to the referenced type. The reference must first be removed to obtain `const int`; only then can `const` be removed.

The following table illustrates the results of several common transformations:

| Input type `T`       | Result of `std::remove_cvref_t<T>` |
| -------------------- | ---------------------------------- |
| `int`                | `int`                              |
| `const int`          | `int`                              |
| `volatile int`       | `int`                              |
| `const volatile int` | `int`                              |
| `int&`               | `int`                              |
| `const int&`         | `int`                              |
| `volatile int&&`     | `int`                              |
| `const int*`         | `const int*`                       |
| `int* const`         | `int*`                             |
| `const int* const&`  | `const int*`                       |

## Top-Level CV-Qualifiers vs. Nested CV-Qualifiers

One of the key aspects of using this type trait is understanding the difference between top-level CV-qualifiers and qualifiers that appear inside a compound type.

For example, in `const int*`, the pointer points to a `const` object, but the pointer itself is not top-level `const`. In contrast, in `int* const`, the pointer itself is `const`.

Therefore:

```
#include <type_traits>

static_assert(std::is_same_v<
    std::remove_cvref_t<const int*>,
    const int*
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<int* const>,
    int*
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<const int* const&>,
    const int*
>);
```

In the first example, `const` belongs to the pointee type and remains unchanged. In the second example, `const` belongs to the pointer itself and is removed. In the third example, the reference is removed first, followed by the top-level `const` qualifier of the pointer. However, the `const` qualifier belonging to `int` is preserved.

The same principle applies to compound types such as pointers, member pointers, and other type constructions: `remove_cvref` removes only the top-level CV-qualification of the resulting type, not every qualifier appearing anywhere in its type structure.

## Difference Between `std::remove_cvref` and `std::decay`

Both `std::remove_cvref_t` and `std::decay_t` often produce a type without references and CV-qualifiers. However, their purposes and transformation rules are different.

The type `std::decay_t<T>` performs additional transformations similar to the parameter adjustments that occur when arguments are passed to functions:

- If the type after removing references is an array, it is converted to a pointer to its first element.
- If the type after removing references is a function type, it is converted to a function pointer.
- Otherwise, the top-level CV-qualifiers are removed.

In contrast, `std::remove_cvref_t<T>` preserves the structure of array and function types.

```
#include <type_traits>

using Array = const int[4];
using Function = void(double);

static_assert(std::is_same_v<
    std::remove_cvref_t<Array&>,
    const int[4]
>);

static_assert(std::is_same_v<
    std::decay_t<Array&>,
    const int*
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<Function&>,
    void(double)
>);

static_assert(std::is_same_v<
    std::decay_t<Function&>,
    void(*)(double)
>);
```

### Choosing the Appropriate Utility

| Requirement                                                                             | Appropriate utility       |
| --------------------------------------------------------------------------------------- | ------------------------- |
| Remove references and top-level CV-qualifiers while preserving array and function types | `std::remove_cvref_t`     |
| Model the type transformations associated with function argument decay                  | `std::decay_t`            |
| Remove references only                                                                  | `std::remove_reference_t` |
| Remove only top-level CV-qualifiers                                                     | `std::remove_cv_t`        |

As a practical rule, if the goal is simply to normalize a type for comparison or constraints without converting arrays to pointers or functions to function pointers, `std::remove_cvref_t` is usually the more precise choice. If decay semantics are actually required, use `std::decay_t`.

## Using `std::remove_cvref` in Templates and Generic Functions

### Normalizing a Deduced Type

A generic function may need to behave differently depending on the underlying type of an argument, without allowing differences in lvalue/rvalue category or `const` qualification to affect type identification.

```
#include <type_traits>

struct Message {};

template <typename T>
void process(T&& value) {
    using ValueType = std::remove_cvref_t<T>;

    static_assert(std::is_same_v<ValueType, Message>);

    // Process the message without changing the original object's type.
}
```

In this example, calling the function with `Message`, `Message&`, `const Message&`, or `Message&&` can result in the same `ValueType`, provided that template argument deduction is compatible with those arguments.

### Selecting Behavior with `if constexpr`

When compile-time branching is required, the normalized type can be inspected using `if constexpr`:

```
#include <type_traits>

struct Text {};
struct Binary {};

template <typename T>
void handle(T&& value) {
    using U = std::remove_cvref_t<T>;

    if constexpr (std::is_same_v<U, Text>) {
        // Handle text data.
    } else if constexpr (std::is_same_v<U, Binary>) {
        // Handle binary data.
    } else {
        // Handle other types.
    }
}
```

This pattern is useful for templates whose behavior depends on the category of the argument type. However, if the goal is simply to restrict the set of accepted types, using concepts in C++20 is often more readable.

## Using `std::remove_cvref` with Concepts and Constraints in C++20

One important application of `std::remove_cvref_t` is defining constraints that should accept different CV-qualified and reference forms of the same type.

For example, the following concept can be defined:

```
#include <concepts>
#include <type_traits>

struct Message {};

template <typename T>
concept MessageLike =
    std::same_as<std::remove_cvref_t<T>, Message>;

template <MessageLike T>
void send(T&& message) {
    // Send the message.
}
```

With this definition, types such as `Message`, `Message&`, and `const Message&` are normalized to the same type for constraint evaluation.

The same utility can also be used directly in a constraint:

```
#include <concepts>
#include <type_traits>

template <typename T>
    requires std::same_as<std::remove_cvref_t<T>, int>
void consume(T&& value) {
    // Consume an integer-like argument.
}
```

Using `std::remove_cvref_t` in a constraint should be intentional. If `const` qualification or reference semantics are part of the API contract, removing them may make the constraint too permissive and allow types that should not be accepted.

## Using `std::remove_cvref` in Type Traits and Type Selection

When implementing custom type traits, it is sometimes necessary to evaluate a condition independently of whether a type is CV-qualified or a reference.

For example:

```
#include <type_traits>

template <typename T>
struct is_integer_like
    : std::is_integral<std::remove_cvref_t<T>> {};

static_assert(is_integer_like<int>::value);
static_assert(is_integer_like<const int&>::value);
static_assert(is_integer_like<volatile long&&>::value);
```

This trait is a simple example of combining standard type traits. In real-world projects, check whether an appropriate standard type trait or concept already exists before introducing a custom one.

## Using `std::remove_cvref` with Perfect Forwarding

In functions that use perfect forwarding, the template parameter preserves information such as whether the argument is an lvalue or rvalue and whether it has CV-qualifiers. This information is essential for correct forwarding. However, a normalized type can be more convenient for certain compile-time decisions.

```
#include <type_traits>
#include <utility>

template <typename T>
decltype(auto) forward_to_target(T&& value) {
    using U = std::remove_cvref_t<T>;

    static_assert(std::is_class_v<U>);

    return target(std::forward<T>(value));
}
```

In this pattern, `T` remains unchanged when passed to `std::forward<T>` because removing its reference could destroy the information needed to preserve the argument's value category. The type `U` is used only for compile-time inspection.

An important design consideration is that `remove_cvref` must not be used as a replacement for `std::forward`. The former is a compile-time type transformation; the latter is a utility for preserving and forwarding value categories in expressions.

## Difference Between `std::remove_reference`, `std::remove_cv`, and `std::remove_cvref`

These three utilities are related, but they do not perform the same operation:

- `std::remove_reference_t<T>` removes only `&` or `&&`.
- `std::remove_cv_t<T>` removes only the top-level `const` and `volatile` qualifiers.
- `std::remove_cvref_t<T>` performs both operations in the appropriate order.

If only one of these properties needs to be removed, using the corresponding specialized trait is more precise. For example, when code needs to detect whether a type is `const`, normalizing it with `remove_cvref` before performing the check can discard the information required by that check.

## Conceptual Implementation and Compatibility with Older Standards

`std::remove_cvref` is part of the C++20 standard library. Therefore, its availability should not be assumed in environments where either the standard library or the compiler's selected language mode does not support C++20.

For projects using C++11, C++14, or C++17, an equivalent can be implemented by combining older type traits:

```
#include <type_traits>

template <typename T>
using remove_cvref_t =
    std::remove_cv_t<std::remove_reference_t<T>>;
```

This alias uses standard type traits available in older C++ versions. If this name is defined in a public library namespace, choose a naming convention that avoids conflicts with existing or future library components.

A feature-test macro can be used to check whether the standard library supports this feature:

```
#include <type_traits>

#if defined(__cpp_lib_remove_cvref) && \
    __cpp_lib_remove_cvref >= 201711L
// std::remove_cvref_t is available.
#endif
```

The feature-test macro associated with this functionality is `201711L`. This macro reports support for a standard library feature and is not equivalent to simply selecting a language mode in the compiler. The version and configuration of the standard library also matter.

## Considerations for Array and Function Types

Because `remove_cvref` does not perform array-to-pointer or function-to-pointer conversions, unlike `decay`, generic code that handles array or function types must account for this difference.

```
#include <type_traits>

using ArrayType = int[3];
using FunctionType = int(char);

static_assert(std::is_same_v<
    std::remove_cvref_t<ArrayType&>,
    int[3]
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<FunctionType&>,
    int(char)
>);
```

This behavior is useful when preserving type information is important, such as when array sizes or function type structures are part of compile-time decisions. Conversely, if an API requires a pointer, perform the appropriate conversion explicitly or use `std::decay_t`.

## Considerations for Pointers and Member Pointers

The CV-qualifier removal performed by this utility is limited to the top-level type. Therefore, qualifiers belonging to the type pointed to by a pointer are not removed. The same principle applies to more complex type structures.

```
#include <type_traits>

struct Object {
    int member;
};

using Pointer = int* const&;
using MemberPointer = int Object::* const&;

static_assert(std::is_same_v<
    std::remove_cvref_t<Pointer>,
    int*
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<MemberPointer>,
    int Object::*
>);
```

In the second example, `const` belongs to the member pointer itself and is removed after the reference is removed. The member type and its owning class remain unchanged.

## Considerations for Incomplete Types

The `remove_cvref` operation does not depend on a type's size or layout and does not require a class type to be complete. Therefore, it can generally be applied to references or CV-qualified references to incomplete classes.

```
#include <type_traits>

class ForwardDeclared;

using Ref = const ForwardDeclared&;
using Result = std::remove_cvref_t<Ref>;

static_assert(std::is_same_v<Result, ForwardDeclared>);
```

This does not mean that every other operation on an incomplete type is permitted. For example, traits that depend on properties of a complete type may have separate restrictions.

## Common Mistakes and Misconceptions

### Assuming That All CV-Qualifiers Are Removed

This utility removes only top-level CV-qualifiers, not every occurrence of `const` and `volatile` within a type's structure. For example, `const int*` remains `const int*` after normalization.

### Using It as a Replacement for `std::decay_t`

If the code requires arrays to become pointers or functions to become function pointers, `remove_cvref_t` does not perform those transformations. Choosing the wrong trait can lead to type incompatibilities in templates or overload resolution.

### Using It to Modify Runtime Objects

`std::remove_cvref_t` is a compile-time type transformation. It does not modify an object, remove `const` from an actual object, or grant permission to write to memory accessed through a `const` object.

### Normalizing the Type Before Perfect Forwarding

If you normalize the deduced type before using `std::forward`, you may lose the information required to preserve whether the argument is an lvalue or rvalue. Keep the original template parameter `T` for forwarding and use the normalized type only for inspection or behavior selection.

### Removing Information That Is Part of the API Contract

Sometimes the distinction between `const T&`, `T&`, and `T&&` is important to a function's semantics. Normalizing the type too early can hide these differences. Before using this utility, determine whether the decision should be based on the exact argument type or its underlying type.

## Design and Performance Considerations

`std::remove_cvref` operates at compile time and does not produce runtime objects or allocations. Using it does not introduce runtime overhead. However, as with other forms of template metaprogramming, excessive complexity in chains of type traits can affect readability and compilation time.

When designing generic APIs, it is useful to maintain a clear distinction between two concepts:

- The exact deduced type, which may carry CV-qualification and reference information.
- The normalized type, which is used for comparison, dispatch, or constraints.

It is also preferable to perform normalization as close as possible to the decision that requires it, rather than replacing the original type with its normalized version throughout the entire implementation.

## Summary

`std::remove_cvref` is a C++20 standard library type trait that first removes references and then removes the top-level CV-qualifiers from a type. Its alias form, `std::remove_cvref_t`, is generally more readable in everyday template and generic code.

Its most important difference from `std::decay_t` is that `remove_cvref` preserves array and function type structures, whereas `decay` converts them into pointers. This utility is particularly useful for type comparisons, defining concepts, implementing custom type traits, and making compile-time decisions. However, it must not be used to modify runtime objects or as a replacement for `std::forward`.

As a practical rule, whenever the goal is to inspect an underlying type without CV-qualifiers or references while preserving its structure, `std::remove_cvref_t` is an appropriate choice. When decay semantics are required, choose `std::decay_t`. When CV-qualification or reference information is part of the function's contract, preserve the exact type.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>