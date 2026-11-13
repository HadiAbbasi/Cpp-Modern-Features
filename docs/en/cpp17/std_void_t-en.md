<div align="center">

[🇺🇸 English](./void_t.md) | [🇮🇷 فارسی](../../fa/cpp17/void_t.md)

</div>

----

# `std::void_t` in C++17

Introduced in **C++17** via proposal **P0006R0** ("Adopt `Type Traits Variable Templates` and `void_t` for C++17", building upon Walter E. Brown's **N3911**), `std::void_t` is a utility metafunction that maps an arbitrary list of types to the type `void`. 

Despite its deceptively minimalist implementation (`template<class...> using void_t = void;`), `std::void_t` revolutionized modern template metaprogramming. Prior to C++20 Concepts, it served as the industry-standard idiom for writing concise, expressive **SFINAE** (Substitution Failure Is Not An Error) traits to introspect types: detecting valid expressions, member functions, nested typedefs, and operator overloads at compile time without third-party libraries.

```cpp
#include <type_traits>
#include <iostream>

// Primary template: assumption is false
template <typename T, typename = void>
struct has_reserve : std::false_type {};

// Partial specialization: substituted only if T.reserve(size_t) is valid
template <typename T>
struct has_reserve<T, std::void_t<decltype(std::declval<T>().reserve(std::declval<size_t>()))>> 
    : std::true_type {};

template <typename T>
inline constexpr bool has_reserve_v = has_reserve<T>::value;
```

---

## Table of Contents

1. [Motivation & SFINAE Complexity Before C++17](#1-motivation--sfinae-complexity-before-c17)
2. [Definition & Mechanics: How It Actually Works](#2-definition--mechanics-how-it-actually-works)
3. [The Core Detection Idiom Pattern](#3-the-core-detection-idiom-pattern)
4. [Real-World Metaprogramming Examples](#4-real-world-metaprogramming-examples)
   - [A. Detecting Nested Type Aliases (`value_type`, `iterator`)](#a-detecting-nested-type-aliases-value_type-iterator)
   - [B. Detecting Member Functions & Methods](#b-detecting-member-functions--methods)
   - [C. Detecting Operators & Expressions (`operator<<`, `operator==`)](#c-detecting-operators--expressions-operator-operator)
5. [The CWG 1558 Defect: Why Naive Implementations Failed Pre-C++14](#5-the-cwg-1558-defect-why-naive-implementations-failed-pre-c14)
6. [C++17 `std::void_t` vs. C++20 `requires` & Concepts](#6-c17-stdvoid_t-vs-c20-requires--concepts)
7. [Feature-Test Macro & Backport Implementation](#7-feature-test-macro--backport-implementation)
8. [Common Pitfalls and Compilation Bugs](#8-common-pitfalls-and-compilation-bugs)
9. [Contributors](#9-contributors)

---

## 1. Motivation & SFINAE Complexity Before C++17

Prior to C++17, writing SFINAE introspection traits required substantial boilerplate involving `std::enable_if`, overloaded functions returning dummy structures of different sizes (e.g., `sizeof(yes_type) vs sizeof(no_type)`), or custom comma-operator tricks.

### Historical C++11 Workaround (The Verbose `sizeof` Trick):
```cpp
template <typename T>
class has_serialize {
    typedef char yes[1];
    typedef char no[2];

    template <typename C>
    static yes& test(decltype(&C::serialize));

    template <typename C>
    static no&  test(...);

public:
    static constexpr bool value = sizeof(test<T>(nullptr)) == sizeof(yes);
};
```

This pattern suffered from numerous flaws:
- Verbose and difficult to read or maintain.
- Only checked for the *address* of a member, making it fragile against overloads or non-exact signatures.
- Hard to compose across multiple expressions simultaneously.

`std::void_t` standardized the "Detection Idiom", turning multi-line template hacks into clean, uniform partial specializations.

---

## 2. Definition & Mechanics: How It Actually Works

The standard implementation in header `<type_traits>` is fundamentally simple:

```cpp
namespace std {
    template <typename... Ts>
    using void_t = void;
}
```

### The Step-by-Step SFINAE Resolution Engine:
1. `std::void_t` accepts a variadic pack of type arguments: `Ts...`.
2. When the compiler attempts to instantiate a partial specialization using `void_t<Expr1, Expr2, ...>`, it must first evaluate and substitute all expressions inside the angle brackets.
3. **If any expression is ill-formed** (e.g., calling a method that does not exist, dereferencing a non-pointer), template substitution fails silently (**SFINAE**). The compiler discards the partial specialization and falls back to the primary template.
4. **If all expressions are valid**, all substituted types collapse into `void`. The compiler sees a specialization for `template<T, void>`, matches it against the primary template's default parameter (`typename = void`), and selects the specialized version.

```text
+-----------------------------------------------------------------------------------------+
|                              SFINAE Evaluation Flow with void_t                         |
+-----------------------------------------------------------------------------------------+
| Expression: has_trait<MyType>                                                           |
|                                                                                         |
| 1. Matches Primary Template: has_trait<T, typename = void>                              |
|                                                                                         |
| 2. Checks Specialization: has_trait<T, std::void_t< decltype(expression) >>             |
|          │                                                                              |
|          ├── Expression is ILL-FORMED ──> SFINAE Triggers ──> Falls back to Primary     |
|          │                                                    (has_trait::value = false)|
|          │                                                                              |
|          └── Expression is VALID       ──> Becomes void   ──> Matches Specialization    |
|                                                               (has_trait::value = true) |
+-----------------------------------------------------------------------------------------+
```

---

## 3. The Core Detection Idiom Pattern

Every `std::void_t` trait follows a standardized three-part structural recipe:

```cpp
#include <type_traits>

// Step 1: Primary template declares the default false_type case
template <typename T, typename = void>
struct is_iterable : std::false_type {};

// Step 2: Partial specialization probes valid expressions inside std::void_t
template <typename T>
struct is_iterable<T, std::void_t<
    decltype(std::declval<T>().begin()),
    decltype(std::declval<T>().end())
>> : std::true_type {};

// Step 3: Helper variable template (C++17 inline constexpr convention)
template <typename T>
inline constexpr bool is_iterable_v = is_iterable<T>::value;
```

---

## 4. Real-World Metaprogramming Examples

### A. Detecting Nested Type Aliases (`value_type`, `iterator`)
Inspect whether a container provides standard container typedefs:

```cpp
template <typename T, typename = void>
struct has_value_type : std::false_type {};

template <typename T>
struct has_value_type<T, std::void_t<typename T::value_type>> : std::true_type {};

template <typename T>
inline constexpr bool has_value_type_v = has_value_type<T>::value;

static_assert(has_value_type_v<std::vector<int>>);
static_assert(!has_value_type_v<int>);
```

---

### B. Detecting Member Functions & Methods
Inspect if a type possesses a `size()` member returning an integral type:

```cpp
template <typename T, typename = void>
struct has_size_method : std::false_type {};

template <typename T>
struct has_size_method<T, std::void_t<
    decltype(std::declval<const T&>().size())
>> : std::is_integral<decltype(std::declval<const T&>().size())> {};

template <typename T>
inline constexpr bool has_size_method_v = has_size_method<T>::value;
```

---

### C. Detecting Operators & Expressions (`operator<<`, `operator==`)
Detect whether an object can be streamed into `std::ostream`:

```cpp
#include <ostream>

template <typename T, typename = void>
struct is_printable : std::false_type {};

template <typename T>
struct is_printable<T, std::void_t<
    decltype(std::declval<std::ostream&>() << std::declval<const T&>())
>> : std::true_type {};

template <typename T>
inline constexpr bool is_printable_v = is_printable<T>::value;

struct NonPrintable {};
static_assert(is_printable_v<int>);
static_assert(!is_printable_v<NonPrintable>);
```

---

## 5. The CWG 1558 Defect: Why Naive Implementations Failed Pre-C++14

In C++11, attempting to write:

```cpp
// Naive implementation in early C++11
template <typename... Ts>
using void_t = void;
```

frequently produced a baffling compiler bug where SFINAE **failed to trigger**, always selecting the primary template.

### The Standard Defect (CWG 1558):
Under the initial C++11 standard wording, unused template parameters in alias templates were not required to be instantiated during substitution if they did not affect the resulting type (`void`). As a result, compilers like GCC and Clang silently dropped the nested expression inside `void_t<decltype(...)>` without checking for substitution errors!

### The C++11/C++14 Conforming Workaround:
To force standard-compliant compilers to evaluate every parameter, an intermediate dummy helper template was required:

```cpp
namespace detail {
    template <typename... Ts>
    struct make_void {
        using type = void;
    };
}

// CWG 1558 compliant alias:
template <typename... Ts>
using void_t = typename detail::make_void<Ts...>::type;
```

In **C++17**, CWG 1558 was officially resolved: alias templates are guaranteed to instantiate all dependent template arguments.

---

## 6. C++17 `std::void_t` vs. C++20 `requires` & Concepts

While `std::void_t` remains the standard foundation for C++17 metaprogramming, **C++20 Concepts** offer a much more readable syntax with superior compiler error messages:

```text
+-----------------------------------------------------------------------------------------+
|                              Trait Comparison: C++17 vs C++20                           |
+-----------------------------------------------------------------------------------------+
| Feature                     | C++17 (std::void_t)         | C++20 (Concepts / requires) |
+-----------------------------+-----------------------------+-----------------------------+
| Paradigm                    | Template Specialization     | First-Class Language Concept|
| Syntax Volume               | High (Primary + Spec + Var) | Minimal (1 single concept)  |
| Compiler Diagnostics        | Opaque template backtraces  | Pinpoints failing constraint|
| Compilation Throughput      | Moderate template overhead  | Fast short-circuit checking |
+-----------------------------------------------------------------------------------------+
```

### Side-by-Side Equivalence:
```cpp
// C++17: std::void_t
template <typename T, typename = void>
struct is_hashable : std::false_type {};

template <typename T>
struct is_hashable<T, std::void_t<decltype(std::hash<T>{}(std::declval<T>()))>> 
    : std::true_type {};

// C++20: Concepts
template <typename T>
concept Hashable = requires(T a) {
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};
```

---

## 7. Feature-Test Macro & Backport Implementation

`std::void_t` is tracked under the feature macro `__cpp_lib_void_t`:

```cpp
#include <type_traits>

#if defined(__cpp_lib_void_t) && __cpp_lib_void_t >= 201411L
    using my_void_t = std::void_t;
#else
    // Fully CWG-1558 compliant portable backport:
    namespace compat {
        template <typename... Ts>
        struct make_void { using type = void; };

        template <typename... Ts>
        using void_t = typename make_void<Ts...>::type;
    }
    using my_void_t = compat::void_t;
#endif
```

---

## 8. Common Pitfalls and Compilation Bugs

### 1. Forgetting the Default `void` in the Primary Template
A common beginner error is declaring the primary template without default initialization:

```cpp
// BUG: Omitting '= void' makes partial specialization impossible to match automatically
template <typename T, typename> 
struct check_feature : std::false_type {};

// Calling check_feature<int> will FAIL to compile because 2 template arguments are required!
```
**Fix:** Always write `template <typename T, typename = void>`.

### 2. Forgetting `std::declval<T>()` and Calling Concrete Constructors
Attempting to construct types directly inside `decltype()` fails if the type has no default constructor:

```cpp
// BUG: T might not be default-constructible!
template <typename T>
struct has_ping<T, std::void_t<decltype(T().ping())>> : std::true_type {};

// CORRECT: std::declval<T>() works even without constructors!
template <typename T>
struct has_ping<T, std::void_t<decltype(std::declval<T>().ping())>> : std::true_type {};
```

### 3. Comma Operator Pitfall inside `decltype`
When listing multiple dependent expressions, wrap them with separate arguments inside `void_t<...>`, rather than chaining them with the comma operator:

```cpp
// PREFERRED: Multiple arguments to void_t
template <typename T>
struct trait<T, std::void_t<decltype(std::declval<T>().a()), 
                            decltype(std::declval<T>().b())>> : std::true_type {};
```

---

## 9. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |