# Structured Bindings in C++17

## Table of Contents

- [Introduction](#introduction)
- [Required Header](#required-header)
- [Syntax](#syntax)
- [What a Structured Binding Really Is](#what-a-structured-binding-really-is)
- [Binding Process](#binding-process)
- [The Three Binding Cases](#the-three-binding-cases)
- [Memory Model and Lifetime](#memory-model-and-lifetime)
- [Value Category and `decltype`](#value-category-and-decltype)
- [Initialization Order](#initialization-order)
- [Structured Binding Packs in C++26](#structured-binding-packs-in-c26)
- [Customizing Tuple-Like Types](#customizing-tuple-like-types)
- [Common Use Cases](#common-use-cases)
- [Common Errors](#common-errors)
- [Feature-Test Macro](#feature-test-macro)
- [Complete Example](#complete-example)
- [Exercises](#exercises)
- [Conclusion](#conclusion)
- [Contributors](#contributors)

---

## Introduction

Structured bindings are a C++17 language feature that lets you decompose an object into named bindings in a single declaration.

They are commonly used with:

- arrays,
- `std::pair`,
- `std::tuple`,
- map-like elements in range loops,
- classes with public data members.

Example:

```cpp
std::pair<int, int> p{1, 2};
auto [x, y] = p;
```

This is more readable than accessing `p.first` and `p.second` repeatedly, and it scales well when the number of components is small and fixed.

---

## Required Header

Structured bindings are a language feature, so they do not require a dedicated header.

You will usually include headers based on the objects you decompose:

```cpp
#include <tuple>
#include <utility>
#include <array>
#include <map>
#include <iostream>
```

---

## Syntax

The standard syntax is:

```cpp
attr(optional) decl-specifier-seq ref-qualifier(optional) [ identifier-list ] initializer ;
```

Common examples:

```cpp
auto [a, b] = value;
const auto& [x, y] = pair;
auto&& [i, j] = make_tuple();
```

Notes:

- `attr` is optional.
- `ref-qualifier` is `&` or `&&`.
- The initializer may be `= expr`, `{ expr }`, or `( expr )`.
- The expression must not be a top-level comma expression.

---

## What a Structured Binding Really Is

A structured binding declaration introduces a hidden object, often described as `e`, to hold or refer to the initializer.

The names inside `[...]` are not independent new storage locations in the usual sense. They are bindings to subobjects, tuple elements, or members of that hidden object.

That matters for two reasons:

- the hidden object controls lifetime,
- `decltype(name)` does not always behave like an ordinary local variable.

The wording in the standard is precise because structured bindings interact with references, temporaries, tuples, and class member access in different ways depending on the initializer type.

---

## Binding Process

The binding process starts by introducing the hidden variable `e`.

If the initializer is an array and no ref-qualifier is present, the hidden variable is formed from the array object itself. Otherwise, the declaration creates `e` using the declared specifiers and ref-qualifier.

Then the implementation determines the type `E` of the hidden object. The structured binding size of `E` is the number of bindings required to decompose it.

If the number of identifiers does not match that size, the declaration is ill-formed, except for the C++26 structured binding pack rules.

The binding then proceeds in one of three cases.

---

## The Three Binding Cases

### Case 1: Array binding

If `E` is an array type of known bound, each binding names the corresponding array element.

```cpp
int a[3] = {1, 2, 3};

auto [x, y, z] = a;
```

### Case 2: Tuple-like binding

If `E` is a non-union class type and `std::tuple_size<E>` is a complete type with a member named `value`, the tuple-like protocol is used.

This is how structured bindings work for:

- `std::pair`,
- `std::tuple`,
- `std::array`,
- user-defined tuple-like types.

### Case 3: Member binding

If `E` is a non-union class type and `std::tuple_size<E>` is not a complete type, the bindings refer to the accessible non-static data members of the class.

This is the case for ordinary aggregates with public fields.

---

## Memory Model and Lifetime

Structured bindings are aliases to existing objects, but the hidden object `e` is what actually owns or refers to the underlying storage.

That hidden object determines lifetime in the same way a normal variable or reference would.

### Binding a temporary safely

```cpp
const auto& [x, y] = std::pair{1, 2};
```

Here, `e` binds to the temporary pair by reference, which extends the lifetime of the temporary to the lifetime of `e`. The bindings `x` and `y` remain valid as long as `e` remains valid.

### Binding without a reference

```cpp
auto [x, y] = std::pair{1, 2};
```

Here, `e` is a separate object initialized from the temporary. The bindings refer to subobjects of `e`, not to the original temporary.

### Why this matters

The visible names `x` and `y` may look like independent values, but the memory they refer to can come from:

- the original array,
- the hidden object `e`,
- the tuple element object inside `e`,
- a class member of `e`.

So the correct mental model is: a structured binding is a named projection of a hidden binding object, not a new object with its own unrelated storage.

---

## Value Category and `decltype`

The type reported by `decltype(name)` for a structured binding is the referenced type, not necessarily the exact type of the hidden reference used internally.

This is important for tuple-like bindings because the standard introduces reference variables behind the scenes, but `decltype` reports the element type described by `std::tuple_element`.

### Example

```cpp
#include <tuple>
#include <utility>

std::tuple<int, int&> make();

auto [x, y] = make();

static_assert(std::is_same_v<decltype(x), int>);
static_assert(std::is_same_v<decltype(y), int&>);
```

The result may surprise people who expect `decltype(y)` to be a reference to the hidden implementation detail. The standard deliberately hides that detail from the public type of the binding.

---

## Initialization Order

If `valI` denotes the object or reference named by the `I`th binding:

1. `e` is initialized first.
2. `val0` is initialized before `val1`.
3. `val1` is initialized before `val2`, and so on.

This sequencing avoids dependence on unspecified evaluation order when the binding list contains expressions that can observe side effects through accessors.

---

## Structured Binding Packs in C++26

C++26 adds structured binding packs, which allow one binding list entry to absorb zero or more elements.

Example:

```cpp
struct C { int x, y, z; };

auto [a, ...rest] = C{};   // rest binds y and z
auto [...head, tail] = C{}; // head binds x and y, tail binds z
```

Rules to remember:

- only one pack can appear in a binding list,
- the pack may be empty,
- the remaining non-pack bindings must still fit the structured binding size.

This feature makes decomposition more flexible when you only care about a prefix or suffix of a fixed-size object.

---

## Customizing Tuple-Like Types

You can make your own type work with structured bindings by specializing `std::tuple_size` and `std::tuple_element`, and by providing `get` access.

### Example

```cpp
#include <tuple>
#include <utility>

struct Point {
    int x;
    int y;
};

namespace std {
    template<> struct tuple_size<Point> : integral_constant<size_t, 2> {};
    template<> struct tuple_element<0, Point> { using type = int; };
    template<> struct tuple_element<1, Point> { using type = int; };
}

template <size_t I>
auto get(const Point& p) {
    if constexpr (I == 0) return p.x;
    else return p.y;
}
```

Once the tuple-like protocol is in place, you can write:

```cpp
Point p{3, 4};
auto [x, y] = p;
```

For library-quality code, follow the standard rules carefully and provide the overload set that matches the value category you want to support.

---

## Common Use Cases

### Decomposing a pair

```cpp
std::pair<std::string, int> p{"C++", 17};
auto [name, version] = p;
```

### Iterating over a map

```cpp
std::map<std::string, int> scores{{"math", 98}, {"physics", 92}};
for (const auto& [subject, score] : scores) {
    std::cout << subject << " = " << score << '\n';
}
```

### Working with tuples

```cpp
auto row = std::tuple<int, std::string, double>{100, "tea", 3.5};
auto [id, item, price] = row;
```

### Decomposing a simple struct

```cpp
struct GeoCoordinate {
    std::string name;
    double latitude;
    double longitude;
};

GeoCoordinate city{"Bogota", 4.711, -74.0721};
const auto& [n, lat, lon] = city;
```

### Decomposing arrays

```cpp
int rgb[3]{255, 128, 0};
auto [r, g, b] = rgb;
```

These examples cover the most common everyday uses.

---

## Common Errors

1. Using a constrained placeholder such as `C auto [x, y]`, which is not allowed.
2. Expecting the bindings themselves to be references in every case. The visible type is not always the implementation type.
3. Forgetting that tuple-like binding wins when `std::tuple_size<E>` is complete, even if that makes the program ill-formed.
4. Returning or storing bindings that outlive the hidden object `e`.
5. Trying to decompose an entity whose members are not accessible in the required case.
6. Assuming `auto [x, y] = some_tuple;` binds directly to the tuple object rather than to subobjects of the hidden object.
7. Mismatching the number of identifiers and the structured binding size.

---

## Feature-Test Macro

The feature-test macro for structured bindings is `__cpp_structured_bindings`.

- `201606L` - C++17 structured bindings
- `202403L` - structured binding packs in C++26

Use this macro when you need to conditionally compile code that depends on the feature.

---

## Complete Example

```cpp
#include <array>
#include <iostream>
#include <map>
#include <string>
#include <tuple>

struct Book {
    std::string title;
    int year;
};

int main() {
    std::pair<std::string, int> version{"C++", 17};
    auto [lang, year] = version;
    std::cout << lang << ' ' << year << '\n';

    std::array<int, 3> rgb{255, 128, 0};
    auto [r, g, b] = rgb;
    std::cout << r << ' ' << g << ' ' << b << '\n';

    Book book{"The Standard Library", 2025};
    const auto& [title, pub_year] = book;
    std::cout << title << ' ' << pub_year << '\n';

    std::map<std::string, int> scores{{"math", 98}, {"physics", 92}};
    for (const auto& [subject, score] : scores) {
        std::cout << subject << '=' << score << '\n';
    }
}
```

This example shows all three standard binding forms in one place: pair-like, array-like, and member-based decomposition.

---

## Exercises

1. Write a function that returns `std::pair<int, int>` and decompose the result with structured bindings.
2. Iterate over a `std::map<std::string, std::string>` using `const auto& [key, value]`.
3. Create a small struct and access its fields through structured bindings.
4. Explain the difference between `auto [x, y] = pair;` and `auto& [x, y] = pair;`.
5. Add `tuple_size` and `tuple_element` support to a custom type and bind it like a tuple.
6. Try a binding pack in C++26 and observe when the pack becomes empty.

---

## Conclusion

Structured bindings are a core C++17 feature for decomposing composite values into readable names.

They work for arrays, tuple-like types, and classes with public data members. The hidden object model matters for lifetime and type deduction, so it is worth understanding the rules rather than treating the feature as simple syntax sugar.

Use structured bindings when you want local names that describe parts of a value directly and clearly.

---

## Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |  |  |  |
