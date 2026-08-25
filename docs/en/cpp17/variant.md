<div align="right">

[🇺🇸 English](./variant.md) | [🇮🇷 فارسی](../../fa/cpp17/variant.md)

</div>
---

# `std::variant` in C++17

## Table of Contents

- [Introduction](#introduction)
- [Required Header](#required-header)
- [Creating a `variant`](#creating-a-variant)
- [Accessing the Current Value with `std::get<Index>`](#accessing-the-current-value-with-stdgetindex)
- [Accessing the Current Value by Type](#accessing-the-current-value-by-type)
- [Safe Access with `std::get_if`](#safe-access-with-stdget_if)
- [Checking the Active Alternative with `std::holds_alternative`](#checking-the-active-alternative-with-stdholds_alternative)
- [The `index()` Member Function](#the-index-member-function)
- [Modifying the Value of a `variant`](#modifying-the-value-of-a-variant)
- [`std::visit` — the Visitor Pattern](#stdvisit--the-visitor-pattern)
- [Handling All Alternatives with the `overloaded` Idiom](#handling-all-alternatives-with-the-overloaded-idiom)
- [`std::monostate`](#stdmonostate)
- [Comparing `variant`s](#comparing-variants)
- [`std::variant_size` and `std::variant_alternative`](#stdvariant_size-and-stdvariant_alternative)
- [`variant` vs. `union`](#variant-vs-union)
- [`variant` vs. `optional` vs. `any`](#variant-vs-optional-vs-any)
- [Common Errors](#common-errors)
- [Complete Example](#complete-example)
- [Exercises](#exercises)
- [Conclusion](#conclusion)

---

## Introduction

`std::variant` is a type-safe union added to the standard library in C++17. It stores a value of exactly one of several alternative types at a time.

Unlike a raw `union`, a `variant`:

- keeps track of which alternative is currently active,
- destroys the old value automatically when a new one is assigned,
- works with non-trivial types such as `std::string`.

Example use case:

```cpp
std::variant<int, std::string, double> value{42};
```

> `std::variant` was one of the major library additions of C++17. It is the safe, modern replacement for `union` in most code.

---

## Required Header

To use `std::variant`, include the `<variant>` header:

```cpp
#include <variant>
```

Examples involving strings and output may also require:

```cpp
#include <iostream>
#include <string>
```

---

## Creating a `variant`

### Default Construction

The first alternative is used as the default and is value-initialized:

```cpp
std::variant<int, std::string> value; // holds int, value 0
```

If the first alternative cannot be default-constructed, the variant itself is not default-constructible. In that case, use `std::monostate`.

### Constructing with a Value

The compiler selects the alternative that best matches the constructor argument:

```cpp
std::variant<int, std::string, double> value{42};     // holds int
std::variant<int, std::string, double> text{"hello"}; // holds std::string
```

### Constructing in Place

Use in-place construction to select a specific alternative directly or avoid an extra temporary:

```cpp
std::variant<int, std::string> v{std::in_place_type<std::string>, "hello"};
std::variant<int, std::string> w{std::in_place_index<1>, "world"};
```

### CTAD in C++17

Since C++17, the alternative types can be deduced:

```cpp
std::variant v{42, 3.14}; // std::variant<int, double>
```

---

## Accessing the Current Value with `std::get<Index>`

`std::get<Index>` returns a reference to the stored value if the requested alternative is active. Otherwise, it throws `std::bad_variant_access`:

```cpp
std::variant<int, std::string> value{42};
std::cout << std::get<0>(value) << '\n'; // 42
```

---

## Accessing the Current Value by Type

If a type is unique among the alternatives, you can access it by type:

```cpp
auto n = std::get<int>(value); // OK
```

---

## Safe Access with `std::get_if`

`std::get_if` returns a pointer to the stored value, or `nullptr` if the requested alternative is not active. It never throws:

```cpp
if (auto* p = std::get_if<int>(&value)) {
    std::cout << "int: " << *p << '\n';
}
```

---

## Checking the Active Alternative with `std::holds_alternative`

```cpp
if (std::holds_alternative<int>(value)) {
    // Currently holds an int
}
```

---

## The `index()` Member Function

`index()` returns the zero-based index of the active alternative:

```cpp
std::variant<int, std::string> value{"hi"};
std::cout << value.index() << '\n'; // 1
```

---

## `std::visit` — the Visitor Pattern

`std::visit` invokes a callable on the value currently held by the variant. This is the idiomatic way to process a variant:

```cpp
std::variant<int, std::string> value{3.14};
std::visit([](auto&& arg) {
    std::cout << arg << '\n';
}, value);
```

---

## Handling All Alternatives with the `overloaded` Idiom

A common helper for visiting variants with different logic for each type:

```cpp
template <typename... Ts> struct overloaded : Ts... { using Ts::operator()...; };
template <typename... Ts> overloaded(Ts...) -> overloaded<Ts...>;

std::visit(overloaded{
    [](int i) { std::cout << "int: " << i << '\n'; },
    [](std::string s) { std::cout << "string: " << s << '\n'; }
}, value);
```

---

## `std::monostate`

If the first alternative is not default-constructible, use `std::monostate` as the first type to allow default construction:

```cpp
std::variant<std::monostate, NoDefaultConstructor> v; // OK
```

---

## Common Errors

1. **Accessing the wrong alternative**: `std::get` throws `std::bad_variant_access`.
2. **Ambiguous type access**: `std::get<int>` on `variant<int, int>` is a compile-time error.
3. **Valueless by exception**: Rarely, a variant can become valueless if an exception occurs during assignment. Check `v.valueless_by_exception()`.

---

## Exercises

1. Create a `variant` that can hold an `int`, `double`, or `std::string`.
2. Assign a value to it and print its `index()`.
3. Use `std::get_if` to safely print the value only if it is a `double`.
4. Implement the `overloaded` visitor to handle each type differently.

---

## Conclusion

`std::variant` provides a type-safe, modern alternative to `union`. With `std::visit` and `std::get_if`, it becomes a powerful tool for handling heterogeneous data in C++17.

---
## 🤝 Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |  |  |  |
