<div align="center">

[🇺🇸 English](./variable_templates.md) | [🇮🇷 فارسی](../../fa/cpp14/variable_templates.md)

</div>

---

# Introduction to Variable Templates in C++14

## Introduction

Variable templates were introduced in the C++14 standard. They make it possible to define variables or constants whose type or value depends on template parameters. Before this feature, type-dependent values were commonly expressed using `static constexpr` members of class templates or helper structures such as `std::integral_constant`.

With variable templates, a type-dependent value can be defined directly, without creating a helper class. This is a language feature, not a standalone feature of the standard library.

## Concept and Motivation

Suppose we want a constant `pi` to be available for each numeric type, with the same type as that numeric type. In C++11, a class template could be used for this purpose, but C++14 makes the definition simpler:

```cpp
template <typename T>
constexpr T pi = T(3.1415926535897932385L);
```

In this example, `T` is a type parameter, and each instantiation of `pi<T>` creates a separate variable of type `T`. For example, `pi<float>` and `pi<double>` are two different instantiations.

The primary use of this feature is to express constants, settings, and data that depend on a type more readably than older patterns allow. However, a variable template does not, by itself, generalize an algorithm; it simply makes the variable definition parameterized.

## Basic Syntax

The general form of a definition is:

```cpp
template <template-parameter-list>
decl-specifiers variable-name = initializer;
```

The parameter list can contain type parameters, non-type template parameters, or a combination of both. The definition must have a valid type and initializer for each instantiation.

Here is a simple example:

```cpp
template <typename T>
constexpr bool is_pointer_v = false;

template <typename T>
constexpr bool is_pointer_v<T*> = true;
```

This example uses partial specialization to detect pointer types. `is_pointer_v<int>` is `false`, while `is_pointer_v<int*>` is `true`.

## Differences from Ordinary Variables and Static Members

An ordinary variable cannot be defined directly in terms of a template parameter. A variable template, by contrast, is instantiated for each set of template arguments.

```cpp
template <typename T>
inline constexpr bool always_true_v = true;
```

The `inline` variable feature became available in C++17, so this definition is not valid as standard C++14. To maintain C++14 compatibility, remove `inline`:

```cpp
template <typename T>
constexpr bool always_true_v = true;
```

`static` members of class templates can also hold type-dependent values, but using them generally requires specifying both the class and the member:

```cpp
template <typename T>
struct traits {
    static constexpr bool value = true;
};

bool a = traits<int>::value;
```

A variable template expresses the same idea more directly:

```cpp
template <typename T>
constexpr bool value_v = true;

bool b = value_v<int>;
```

## Using Variable Templates with Type Traits

One of the most common applications is providing a simpler interface to type traits. In the C++14 standard library, `_v` names were not provided across the board for all type traits; standard library variable templates such as `std::is_integral_v` were added in C++17. In C++14, however, we can define such variables ourselves:

```cpp
#include <type_traits>

template <typename T>
constexpr bool is_integral_v = std::is_integral<T>::value;
```

We can then write:

```cpp
static_assert(is_integral_v<int>, "int must be integral");
static_assert(!is_integral_v<double>, "double is not integral");
```

In C++17, the same use case is supported directly by the standard through `std::is_integral_v<T>`. If a project must compile as C++14, a custom definition or the pattern based on `::value` remains useful.

## Non-Type Template Parameters

A variable template can also use non-type template parameters. In C++14, these parameters must have types permitted by that standard; the restrictions are not the same as those under newer features such as C++20.

```cpp
template <unsigned N>
constexpr unsigned buffer_size = N * 2;

static_assert(buffer_size<8> == 16, "unexpected size");
```

In this example, `N` is a compile-time value, and the instantiation `buffer_size<8>` provides the constant value `16`.

Combining a type parameter with a non-type parameter is also possible:

```cpp
template <typename T, unsigned N>
constexpr T scale = T(N) * T(1.5);

constexpr double x = scale<double, 4>;
```

The initializer must be valid for the selected type. For example, numeric conversions may lose precision or may not be valid for some types at all.

## Specializing Variable Templates

Like other templates, variable templates support full and partial specialization. Partial specialization is often useful for a family of types, such as pointers or arrays.

```cpp
template <typename T>
constexpr int category_v = 0;

template <typename T>
constexpr int category_v<T*> = 1;

template <typename T>
constexpr int category_v<const T> = 2;
```

In this example, the specializations apply to specific type patterns. Their matching and selection must be unambiguous; the rules for selecting a specialization are similar to those for class and function templates.

Full specialization for a specific type is also possible:

```cpp
template <typename T>
constexpr int rank_v = 0;

template <>
constexpr int rank_v<bool> = 1;
```

A full specialization must be consistent with the primary template declaration. In projects with multiple translation units, the definition rules and the One Definition Rule (ODR) must also be observed.

## `constexpr` Values and Compile Time

Variable templates are often combined with `constexpr` so their values can be used at compile time. However, declaring a definition `constexpr` does not by itself guarantee that every dependent expression is valid in every context; the initializer and type must satisfy the requirements for constant expressions.

```cpp
template <typename T>
constexpr T zero_v = T{};

static_assert(zero_v<int> == 0, "zero value");
```

If the variable is instantiated for a type for which `T{}` is not a valid initializer, that instantiation will be invalid. Templates generally do not reveal such errors until the relevant instantiation is required.

## Using `decltype` to Determine the Exact Variable Type

The type of a variable template can depend on its parameters. `decltype` can be used to determine the exact type of an expression:

```cpp
template <typename T>
constexpr auto identity_v = T{};

static_assert(std::is_same<decltype(identity_v<int>), const int>::value,
              "decltype of constexpr variable is const-qualified");
```

A variable declared with `constexpr` is `const`, which is why `decltype(identity_v<int>)` yields `const int`. Meanwhile, `auto` deduces the type from the initializer and, in this example, deduces `T`; `constexpr` then makes the variable constant and const-qualified.

## Common Patterns and the `_v` Suffix

In C++ code, the `_v` suffix is a common naming convention for variable templates that represent the result of a trait or property. This suffix is not required by the language, but it is common in the standard library and public libraries.

```cpp
#include <type_traits>

template <typename T>
constexpr bool is_signed_v = std::is_signed<T>::value;

template <typename T>
constexpr bool is_arithmetic_v = std::is_arithmetic<T>::value;
```

This naming convention makes usage shorter than `std::is_signed<T>::value`. The `_v` suffix is best reserved for values that serve as traits or logical/numeric properties rather than being applied to ordinary project variables without a consistent naming convention.

## Relationship with `if constexpr`

The `if constexpr` feature was introduced in C++17 and is not part of C++14. Nevertheless, variable templates can make conditions more readable in C++17 and later code:

```cpp
#include <type_traits>

template <typename T>
constexpr bool is_integral_v = std::is_integral<T>::value;

template <typename T>
void process(T value) {
    if constexpr (is_integral_v<T>) {
        // Handle integral types
    } else {
        // Handle other types
    }
}
```

In a project compiled specifically as C++14, use mechanisms available in that version, such as overloads, specialization, or `std::enable_if`. Defining a variable template with `constexpr` is valid in C++14, but `if constexpr` is not.

## ODR Rules and Use in Header Files

Variable templates are usually defined in header files so they are available when instantiated. In C++14, a variable template definition using `constexpr` does not automatically have the same rule as a C++17 `inline variable`; do not add `inline` to C++14 code and expect it to be standard-conforming.

The One Definition Rule remains important for template definitions. A variable template definition is commonly seen in multiple translation units, but inconsistent definitions and conflicting specializations must be avoided. For non-template global variables defined in headers, the C++14 approach is different; for example, an `extern` declaration can be placed in the header and a single definition in a source file.

## Limitations and Common Mistakes

- Variable templates were introduced in C++14, so the compiler must be configured for an appropriate language standard.
- The `inline` variable feature dates from C++17 and should not be used as part of a standard C++14 solution.
- Standard names such as `std::is_integral_v` and many other `_v` traits are from C++17; in C++14, a local equivalent can be defined.
- An invalid type or initializer may only become apparent when a specialization is instantiated.
- A `constexpr` initializer must actually satisfy the rules for constant expressions; adding `constexpr` does not make every expression valid.
- Specializations must have consistent matching patterns and definitions across translation units.
- A variable template is suitable for holding a value dependent on template parameters, but it is not a complete replacement for a function or class template. If overloads, runtime behavior, or a composite type are needed, those template forms are more appropriate.

## Comparison with Older Approaches

Before C++14, a type-dependent property was commonly represented using `static constexpr` members of a class template:

```cpp
template <typename T>
struct is_pointer : std::false_type {};

template <typename T>
struct is_pointer<T*> : std::true_type {};

static_assert(is_pointer<int*>::value, "pointer type");
```

This approach is still valid and was available in C++11. The advantage of variable templates is that they provide a more direct interface:

```cpp
template <typename T>
constexpr bool is_pointer_v = is_pointer<T>::value;
```

The class-based approach remains useful when a trait needs multiple members, class specialization, or compatibility with older APIs. A variable template often serves as a more convenient interface layer over such traits.

## Choosing an Approach for a Real Project

- For simple constants or properties that depend on a type, variable templates are a readable and direct choice.
- For library traits in C++14, a local variable template can be defined as an interface over `Trait<T>::value`.
- When supporting C++17 and later, first check whether the standard library provides the required `_v` version.
- For complex type logic, runtime behavior, or overloads, use a function or class template; a variable template defines only a variable.
- In public libraries, establish the target standard, ODR requirements, naming conventions, and compatibility needs for older versions from the outset.

## Summary

Variable templates in C++14 provide a direct way to define variables and constants that depend on template parameters. They are particularly useful for type-dependent constants and simpler interfaces to type traits. At the same time, it is important to distinguish them from C++17 features such as inline variables and standard-library traits with the `_v` suffix. Choosing this feature appropriately depends on the project's target standard, specialization requirements, and the complexity of the type or behavior involved.


---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>