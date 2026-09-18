<div align="center">

[🇺🇸 English](./stdoption.md) | [🇮🇷 فارسی](../../fa/c++11/stdoption.md)

</div>

----
# Structured Bindings in C++17 and Beyond — Complete Deep-Dive Guide

Structured bindings are a C++17 language feature that lets you decompose an object into named bindings in a single declaration. They are especially useful with `std::pair`, `std::tuple`, `std::array`, ordinary arrays, map elements, and classes with accessible non-static data members.

```cpp
#include <map>
#include <string>
#include <iostream>

int main() {
    std::map<std::string, int> ages{{"Alice", 30}, {"Bob", 25}};

    for (const auto& [name, age] : ages) {
        std::cout << name << " is " << age << " years old\n";
    }
}
```

This guide combines the conceptual model of structured bindings with practical usage, lifetime rules, tuple customization, range-based loops, common pitfalls, performance considerations, C++20/C++23 notes, and C++26 structured binding packs.

---

## Table of Contents

1. [Why Structured Bindings Exist](#1-why-structured-bindings-exist)
2. [Required Headers](#2-required-headers)
3. [Syntax Overview](#3-syntax-overview)
4. [What a Structured Binding Really Is](#4-what-a-structured-binding-really-is)
5. [What You Can Bind To](#5-what-you-can-bind-to)
6. [How the Binding Process Works](#6-how-the-binding-process-works)
7. [cv-Qualifiers, References, and Lifetime](#7-cv-qualifiers-references-and-lifetime)
8. [Value Category and `decltype`](#8-value-category-and-decltype)
9. [Initialization Order](#9-initialization-order)
10. [Structured Bindings in Range-based `for` Loops](#10-structured-bindings-in-range-based-for-loops)
11. [Binding to Bit-fields](#11-binding-to-bit-fields)
12. [Customizing Tuple-Like Types](#12-customizing-tuple-like-types)
13. [Interaction with Other C++17 Features](#13-interaction-with-other-c17-features)
14. [Common Pitfalls and Gotchas](#14-common-pitfalls-and-gotchas)
15. [Performance Considerations](#15-performance-considerations)
16. [Best Practices](#16-best-practices)
17. [Structured Bindings in C++20 and Beyond](#17-structured-bindings-in-c20-and-beyond)
18. [Feature-Test Macro](#18-feature-test-macro)
19. [Quick Reference Card](#19-quick-reference-card)
20. [Complete Example](#20-complete-example)
21. [Exercises](#21-exercises)
22. [Conclusion](#22-conclusion)
23. [Contributors](#23-contributors)

---

## 1. Why Structured Bindings Exist

Before C++17, extracting multiple values from a compound object was often verbose.

### Before C++17: `std::pair`

```cpp
std::pair<std::string, int> getPerson() {
    return {"Alice", 30};
}

std::string name;
int age;
std::tie(name, age) = getPerson();
```

Or:

```cpp
auto person = getPerson();
std::string n = person.first;
int a = person.second;
```

### Before C++17: `std::tuple`

```cpp
std::tuple<int, double, std::string> data{42, 3.14, "hello"};

int i;
double d;
std::string s;
std::tie(i, d, s) = data;
```

Or:

```cpp
auto t = data;
int i2 = std::get<0>(t);
```

### Before C++17: structs

```cpp
struct Point {
    double x;
    double y;
};

Point p{1.0, 2.0};
double x = p.x;
double y = p.y;
```

The member-access form is perfectly clear for a small struct, but it does not provide a general decomposition syntax.

### Problems with older approaches

- `std::get<N>()` uses numeric indices that can obscure meaning.
- `std::tie` requires variables to be declared first and is primarily an assignment mechanism.
- `.first` and `.second` are generic names rather than names describing the local role of each value.
- Plain C arrays do not provide `.first`, `.second`, or a tuple interface.
- Repeated member access can make code noisier inside loops and conditional logic.

Structured bindings let the names appear next to the decomposition:

```cpp
auto [name, age] = getPerson();
auto [lo, hi] = std::minmax(a, b);
auto [x, y, z] = point3d;
auto [r, g, b] = rgb;
```

The feature is therefore both a convenience feature and a way to make the relationship between a composite value and its local names explicit.

---

## 2. Required Headers

Structured bindings are a **language feature**, so there is no dedicated header to include.

You include whatever headers are required by the objects you want to decompose:

```cpp
#include <array>
#include <iostream>
#include <map>
#include <string>
#include <tuple>
#include <utility>
```

For example:

- `<utility>` for `std::pair` and utilities such as `std::minmax`.
- `<tuple>` for `std::tuple`, `std::get`, `std::tuple_size`, and `std::tuple_element`.
- `<array>` for `std::array`.
- `<map>` for maps.

---

## 3. Syntax Overview

The general declaration syntax is:

```text
attr(optional) decl-specifier-seq ref-qualifier(optional) [ identifier-list ] initializer ;
```

A useful simplified form is:

```text
cv-auto ref-qualifier? [ identifier-list ] = expression ;
```

Common forms include:

```cpp
auto [a, b] = expr;
auto& [a, b] = expr;
const auto& [a, b] = expr;
auto&& [a, b] = expr;
[[maybe_unused]] auto [a, b] = expr;
```

The initializer can use `=`, `{}`, or `()` forms where the declaration grammar permits them:

```cpp
auto [a, b] = value;
auto [a, b]{value};
auto [a, b](value);
```

The expression must not be a top-level comma expression.

### The identifier list

The number of identifiers normally has to match the structured binding size of the object exactly.

```cpp
std::pair<int, int> p{1, 2};
auto [a, b] = p;       // OK
// auto [a] = p;      // error
// auto [a, b, c] = p; // error
```

Identifiers cannot be repeated:

```cpp
// auto [x, x] = p;   // error
```

There is no `_` wildcard or built-in ignore placeholder:

```cpp
// auto [a, _, c] = tuple3; // `_` is an ordinary identifier
```

If you need to ignore elements, give them distinct names or use an API such as `std::tie` when assignment to existing variables is the actual goal.

### Where structured bindings are allowed

They can be used in declarations at supported block and namespace scopes and in the range-declaration of a range-based `for` loop.

They cannot be used as ordinary function parameters, class data-member declarations, or template parameter-list entries.

For example, this is valid at namespace scope:

```cpp
static auto [g_lo, g_hi] = std::minmax(1, 2);
```

As with any namespace-scope object, consider lifetime, linkage, and header/ODR implications before putting such a declaration in a header.

---

## 4. What a Structured Binding Really Is

A structured binding declaration introduces a hidden object, conventionally described as `e`, that either contains the initializer value or refers to an existing object depending on the declaration.

For example:

```cpp
auto [x, y] = std::pair{1, 2};
```

A useful conceptual model is:

```cpp
// Conceptual model, not literal source-level generated code.
auto e = std::pair{1, 2};
// x and y designate the corresponding elements of e.
```

The names in `[...]` are not ordinary independent variables with unrelated storage. They are bindings to elements, members, or subobjects associated with `e`.

This explains two important facts:

- the hidden object controls or participates in the lifetime of the underlying storage;
- `decltype(name)` for a structured binding has special semantics.

A structured binding is therefore best understood as a **named projection of an object**, rather than as a sequence of unrelated local declarations.

---

## 5. What You Can Bind To

There are three standard binding cases.

### 5.1 Arrays

If the initializer is an array of known bound, each binding corresponds to an array element.

```cpp
int arr[3]{1, 2, 3};

auto [a, b, c] = arr;
auto& [r0, r1, r2] = arr;
```

The number of bindings is the array bound.

Multidimensional arrays can also be decomposed one level at a time:

```cpp
double pts[2][2]{{0, 1}, {2, 3}};
auto [p0, p1] = pts;
```

Each binding in the outer decomposition corresponds to one inner array.

### 5.2 Tuple-like types

If the tuple-like binding case applies, structured bindings use the tuple protocol.

Common examples are:

- `std::pair`
- `std::tuple`
- `std::array`
- user-defined types customized with `std::tuple_size`, `std::tuple_element`, and suitable `get` access

For a tuple-like type, the binding count comes from `std::tuple_size<E>::value`.

```cpp
std::pair<std::string, int> person{"Alice", 30};
auto [name, age] = person;
```

```cpp
std::tuple<int, double, std::string> value{42, 3.14, "hello"};
auto [i, d, s] = value;
```

```cpp
std::array<int, 3> rgb{255, 128, 0};
auto [r, g, b] = rgb;
```

### 5.3 Non-static data members

For a class type that goes through the member-binding case, the bindings correspond to accessible non-static data members in declaration order.

```cpp
struct Employee {
    std::string name;
    int id;
    double salary;
};

Employee e{"Bob", 7, 55000.0};
auto& [name, id, salary] = e;
```

For the C++17 member-binding rules, the relevant members must be accessible, and inherited base-class data members are not simply included in the decomposition.

For example:

```cpp
struct Base {
    int a;
};

struct Derived : Base {
    int b;
};

// auto& [a, b] = derived; // not a decomposition of both base and derived members
```

Private or otherwise inaccessible members prevent the required member-binding form.

### Summary table

| Object | Structured binding | Result |
|---|---|---|
| `int arr[3]` | `auto [a,b,c] = arr;` | Array elements |
| `std::pair<K,V>` | `auto [k,v] = p;` | Tuple elements |
| `std::tuple<A,B,C>` | `auto [a,b,c] = t;` | Tuple elements |
| `std::array<T,N>` | `auto [e0,e1,...] = a;` | Tuple-like elements |
| Public-member class | `auto [a,b] = obj;` | Data members |
| `std::vector<int>` | `auto [a,b,...] = v;` | Not directly decomposable by size |
| Custom tuple-like class | `auto [a,b] = obj;` | Possible after customization |

---

## 6. How the Binding Process Works

The language rules can be understood as a sequence of steps.

### Step 1 — Establish the hidden object `e`

Given:

```cpp
auto [a, b, c] = expr;
```

there is a hidden object that holds or refers to the initializer according to the declaration's type specifiers and ref-qualifier.

For a value binding, the conceptual model is:

```cpp
auto e = expr;
```

For a reference binding, the hidden object can instead be a reference to the existing initializer.

### Step 2 — Determine the decomposition case

The compiler determines the relevant type `E` and then selects one of the three cases:

1. array binding,
2. tuple-like binding,
3. member binding.

For tuple-like binding, a complete `std::tuple_size<E>` with a member named `value` selects the tuple protocol.

### Step 3 — Bind the names

Conceptually:

```cpp
// Struct/member case
struct Point {
    double x;
    double y;
};

Point e{1.0, 2.0};
// x designates e.x
// y designates e.y
```

For tuple-like types, each binding is associated with the corresponding `get<I>(e)` operation and `std::tuple_element<I, E>::type`.

The implementation details are intentionally hidden from the programmer. This is why the visible binding should be treated as a language-level name referring to the selected component, not as a normal reference declaration that you could simply rewrite into equivalent source code.

### Step 4 — cv-qualifiers and references apply to `e`

A declaration such as:

```cpp
const auto& [a, b] = expr;
```

controls how the hidden object is formed. The component bindings then refer into that object according to the selected binding case.

This distinction is central to understanding `auto`, `auto&`, `const auto&`, and `auto&&`.

---

## 7. cv-Qualifiers, References, and Lifetime

### 7.1 `auto` — value decomposition

```cpp
std::pair<int, int> p{1, 2};
auto [x, y] = p;
```

The hidden object is initialized as a separate object from `p`. Changes to `x` and `y` therefore affect that hidden object rather than `p`.

```cpp
x = 10; // p.first is still 1
```

This is particularly important when the elements are large objects.

### 7.2 `auto&` — bind to an existing object

```cpp
std::pair<int, int> p{1, 2};
auto& [x, y] = p;

x = 10;
```

Now `p.first` is also `10`, because the decomposition refers to the existing object.

### 7.3 `const auto&` — read without copying

```cpp
std::pair<std::string, int> p{"Alice", 30};
const auto& [name, age] = p;
```

This is a common choice when you want to inspect an existing object without copying it.

### 7.4 `auto&&` — flexible reference form

```cpp
std::pair<int, int> p{1, 2};
auto&& [x, y] = p;
```

When the initializer is an lvalue, the reference can bind to that lvalue. When the initializer is an appropriate temporary, the reference form can bind to the temporary and its lifetime is relevant to the lifetime of the hidden reference object.

This makes `auto&&` useful in generic code, but it should not be used merely because it is flexible. Prefer the simplest declaration that expresses the intended ownership and mutation semantics.

### 7.5 Temporaries and lifetime extension

A reference binding can safely bind to a temporary when the normal reference lifetime-extension rules apply:

```cpp
const auto& [x, y] = std::pair{1, 2};
```

The temporary pair remains alive for the lifetime of the reference binding.

With a value decomposition:

```cpp
auto [x, y] = std::pair{1, 2};
```

the hidden object is a separate object initialized from the temporary, and the bindings refer to that hidden object's components.

### 7.6 `auto&` cannot bind to an ordinary temporary

```cpp
// auto& [x, y] = std::pair{1, 2}; // ill-formed
const auto& [x, y] = std::pair{1, 2}; // valid
```

### 7.7 `const` and copies

Consider:

```cpp
struct S {
    int x;
    int y;
};

const S s{1, 2};

auto [a, b] = s;
```

Because this is a value decomposition, the hidden object is a copy initialized from `s`; the bindings therefore refer to the copy.

By contrast:

```cpp
auto& [c, d] = s;
```

refers to the const object, so the component bindings are read-only through that decomposition.

### 7.8 Lambda captures

C++20 permits direct lambda capture of structured binding names in situations supported by the language rules:

```cpp
auto [x, y] = std::pair{1, 2};
auto sum = [x, y] {
    return x + y;
}();
```

A reference capture remains useful when the lambda should observe the binding's referenced object rather than make its own captured value.

---

## 8. Value Category and `decltype`

Structured bindings have special `decltype` rules.

For an ordinary variable, `decltype(name)` often follows the declared type of the variable. For a structured binding, `decltype(name)` reports the referenced type associated with the selected element, rather than exposing the implementation's hidden reference machinery.

This is particularly important for tuple-like types.

For example:

```cpp
#include <tuple>
#include <type_traits>

std::tuple<int, int&> make();

auto [x, y] = make();

static_assert(std::is_same_v<decltype(x), int>);
static_assert(std::is_same_v<decltype(y), int&>);
```

The second assertion demonstrates that the element's type can itself be a reference type.

### The names are lvalues

The bindings designate their selected components and can be used as lvalue expressions when the selected component is assignable:

```cpp
std::pair<int, int> p{1, 2};
auto& [x, y] = p;

x = 10;
```

### The whole binding cannot be reassigned

There is no source-level name for the hidden object:

```cpp
auto [a, b] = std::pair{1, 2};

a = 10; // modifies the selected component

// [a, b] = std::pair{3, 4}; // not valid syntax
```

If you need to assign new values to existing variables, `std::tie` or ordinary assignment is a different tool with different semantics.

---

## 9. Initialization Order

The hidden object is initialized first. The individual binding elements are then initialized in binding order.

Conceptually:

1. initialize `e`;
2. initialize binding 0;
3. initialize binding 1;
4. continue in order.

For tuple-like bindings, this matters when `get<I>` operations have observable effects or depend on state modified by earlier operations.

The bindings are therefore not an unordered collection of declarations; their order is defined by their position in the binding list.

---

## 10. Structured Bindings in Range-based `for` Loops

Range-based loops are one of the most common uses of structured bindings.

### 10.1 Iterating over a `std::map`

```cpp
#include <iostream>
#include <map>
#include <string>

std::map<std::string, int> stock{
    {"apple", 12},
    {"pear", 5}
};

for (const auto& [item, count] : stock) {
    std::cout << item << ": " << count << '\n';
}
```

A map's value type is pair-like. Its key is const, so when iterating with `auto&` or `const auto&`, the key cannot be modified through the binding.

### 10.2 Mutating map values

```cpp
for (auto& [item, count] : stock) {
    count *= 2;
}
```

The mapped value is mutable, while the key remains const.

### 10.3 Avoid accidental copies

This can copy every element:

```cpp
for (auto [key, value] : mapOfBigObjects) {
    // value may be an element copy
}
```

For read-only iteration, prefer:

```cpp
for (const auto& [key, value] : mapOfBigObjects) {
    // no element copy
}
```

### 10.4 Containers of pairs

```cpp
std::vector<std::pair<std::string, int>> people{
    {"Ann", 31},
    {"Joe", 27}
};

for (const auto& [name, age] : people) {
    std::cout << name << " (" << age << ")\n";
}
```

### 10.5 Containers of tuples

```cpp
std::vector<std::tuple<int, std::string, double>> rows{
    {1, "tea", 3.5},
    {2, "coffee", 4.0}
};

for (const auto& [id, item, price] : rows) {
    std::cout << id << ' ' << item << ' ' << price << '\n';
}
```

### 10.6 `unordered_map::insert`

Many standard library operations return pairs whose components have meaningful roles:

```cpp
std::unordered_map<std::string, int> m;

if (auto [it, inserted] = m.insert({"key", 1}); inserted) {
    std::cout << it->second << '\n';
}
```

### 10.7 Multidimensional arrays

```cpp
int grid[2][3]{{1, 2, 3}, {4, 5, 6}};

for (const auto& [a, b, c] : grid) {
    std::cout << a << ',' << b << ',' << c << '\n';
}
```

Here each loop element is an array of three integers, which can itself be decomposed.

### 10.8 Temporary ranges

Reference choices still matter when the range expression produces a temporary. In particular, a non-const lvalue reference cannot simply bind to an ordinary temporary element sequence:

```cpp
// for (auto& [k, v] : getMap()) { ... }
```

Whether a particular form is valid depends on the range and element types and on the language version. Use `const auto&` when you only need read access to a temporary range, and be explicit about ownership when mutation is required.

---

## 11. Binding to Bit-fields

Structured bindings can bind to bit-fields in the member-binding case.

```cpp
struct Flags {
    unsigned int ready : 1;
    unsigned int error : 1;
    unsigned int code  : 6;
};

void inspect(Flags fl) {
    auto& [ready, error, code] = fl;
    bool r = ready;
}
```

A bit-field does not have an ordinary addressable object representation in the same way as a normal data member.

For example, taking its address is not allowed:

```cpp
// auto* p = &ready; // ill-formed for a bit-field
```

Bit-field bindings should therefore be treated according to the special rules that already apply to bit-fields.

---

## 12. Customizing Tuple-Like Types

A user-defined type can participate in structured bindings through the tuple-like protocol.

The core pieces are:

1. specialize `std::tuple_size`;
2. specialize `std::tuple_element` for each element;
3. provide appropriate `get<I>` access.

### 12.1 Basic example

```cpp
#include <tuple>
#include <utility>

struct Point {
    int x;
    int y;
};

namespace std {
    template<>
    struct tuple_size<Point> : integral_constant<size_t, 2> {};

    template<>
    struct tuple_element<0, Point> {
        using type = int;
    };

    template<>
    struct tuple_element<1, Point> {
        using type = int;
    };
}

template <size_t I>
auto get(const Point& p) {
    if constexpr (I == 0)
        return p.x;
    else
        return p.y;
}
```

Then:

```cpp
Point p{3, 4};
auto [x, y] = p;
```

### 12.2 Library-quality customization

A production-quality tuple-like customization generally needs an overload set that correctly handles the value categories and cv-qualifications your type is intended to support.

For example, a library may provide `get<I>` overloads for:

- `T&`
- `const T&`
- `T&&`
- `const T&&`

and corresponding `tuple_element` information as appropriate.

The exact overload set is part of the API contract. Keep `tuple_size`, `tuple_element`, and `get` consistent with each other.

### 12.3 Tuple protocol versus public-member binding

For a simple public struct, direct member decomposition is usually simpler:

```cpp
struct Point {
    double x;
    double y;
};

auto [x, y] = point;
```

The tuple protocol becomes particularly useful when you want a class to expose a decomposition interface that does not simply mirror direct public-member access.

---

## 13. Interaction with Other C++17 Features

### 13.1 `if` initializer

Structured bindings can be used in an `if` initializer:

```cpp
std::map<std::string, int> m;

if (auto [it, ok] = m.insert({"a", 1}); ok) {
    std::cout << "inserted " << it->second << '\n';
}
```

The binding is scoped to the `if` statement and its associated branches.

### 13.2 `switch` initializer

The same general mechanism can be used with `switch` initializers where the declaration and condition are valid:

```cpp
switch (auto [lo, hi] = std::minmax(v); lo) {
    // ...
}
```

The example is mainly useful for demonstrating that structured bindings participate in the broader C++17 init-statement machinery.

### 13.3 `std::tie` versus structured bindings

| Feature | `std::tie` | Structured bindings |
|---|---|---|
| Requires pre-declared variables | Yes | No |
| Can ignore elements with `std::ignore` | Yes | No built-in wildcard |
| Assigns to existing variables | Yes | No |
| Introduces readable names inline | No | Yes |
| Works with arrays | No general decomposition mechanism | Yes |
| Can be used with `const` | Not as a declaration equivalent | Yes |

Use `std::tie` when the task is assignment into already-existing variables. Use structured bindings when the task is decomposition into names at the point of use.

### 13.4 Returning multiple values from functions

A common modern C++ pattern is to return a small struct:

```cpp
struct ParseResult {
    bool ok;
    int value;
    std::string error;
};

ParseResult parseInt(std::string_view s);

if (auto [ok, value, error] = parseInt(input); !ok) {
    std::cerr << error << '\n';
}
```

A tuple can also be returned:

```cpp
std::tuple<int, int> divide(int a, int b);

auto [quotient, remainder] = divide(10, 3);
```

For public APIs, named structs can make the meaning of the returned components clearer than positional tuples.

### 13.5 `std::optional`

Structured bindings do not directly decompose an `std::optional` itself because an optional is not a tuple-like object merely by containing a value. Check the optional first, then decompose the contained object:

```cpp
std::optional<std::pair<int, int>> findDivision(int a, int b);

if (auto res = findDivision(10, 2)) {
    auto [q, r] = *res;
}
```

---

## 14. Common Pitfalls and Gotchas

### 14.1 There is no wildcard `_`

```cpp
// auto [a, _, c] = tuple3;
```

`_` is simply an identifier. It does not mean "ignore this element," and repeated `_` declarations are not permitted because the identifiers must be distinct.

### 14.2 Accidental copies

This is one of the most common practical mistakes:

```cpp
for (auto [key, value] : mapOfBigObjects) {
    // element may be copied each iteration
}
```

If you only need read access:

```cpp
for (const auto& [key, value] : mapOfBigObjects) {
    // avoids the element copy
}
```

### 14.3 Mutating a container while iterating

Structured bindings do not change iterator invalidation rules.

```cpp
for (auto& [key, value] : myMap) {
    myMap.erase(key); // can invalidate the current iterator
}
```

Use the container's documented iterator-invalidation rules and the appropriate erase pattern for the container and language version.

### 14.4 Binding names are not independent objects

Consider:

```cpp
struct P {
    int x;
    int y;
};

P p{1, 2};
auto& [x, y] = p;

int* px = &x;
x = 10;
```

Here `x` designates `p.x`, so `px` points at the same member storage.

### 14.5 Base-class members are not automatically decomposed

The C++17 member-binding model is based on the relevant class's own data members; it does not simply flatten base-class subobjects into one binding list.

```cpp
struct A {
    int a;
};

struct B : A {
    int b;
};

// auto [a, b] = B{}; // not a decomposition of base and derived members
```

### 14.6 Private members

A class whose relevant data members are inaccessible cannot use the direct member-binding case.

```cpp
class PrivatePoint {
    int x;
    int y;
};

// Direct member decomposition is not available here.
```

A suitable tuple-like customization can provide a different decomposition interface where appropriate.

### 14.7 Binding to references returned by a function

A function can return a pair whose elements are references:

```cpp
std::pair<int&, int&> refs(int& a, int& b) {
    return {a, b};
}

int i = 1;
int j = 2;

auto [r1, r2] = refs(i, j);
r1 = 10;
```

The returned pair itself and the objects referenced by its elements have different lifetimes. Ensure that the referenced objects remain alive whenever the bindings are used.

### 14.8 Element-count mismatch

The binding list normally must match the structured binding size exactly:

```cpp
std::pair<int, int> p;

// auto [a] = p;       // too few
// auto [a, b, c] = p; // too many
```

### 14.9 Tuple-like detection can take precedence

If a class has a complete `std::tuple_size<E>` with a member named `value`, the tuple-like binding case is selected. A malformed or incomplete tuple customization can therefore make a program ill-formed even if the class has public data members that would otherwise look decomposable.

### 14.10 Constrained placeholders are not a structured-binding syntax

A constrained placeholder such as:

```cpp
// C auto [x, y] = value;
```

is not a supported structured-binding declaration form.

### 14.11 Returning or storing bindings

A binding refers to a component associated with its hidden object. Do not design code that allows a binding or a reference derived from it to outlive the object whose storage it refers to.

### 14.12 Namespace-scope bindings

Namespace-scope structured bindings introduce objects with namespace scope. In headers, think carefully about linkage and ODR behavior just as you would for any other namespace-scope variable.

---

## 15. Performance Considerations

Structured bindings are primarily a language-level decomposition facility. In the normal cases, they do not add a conceptual runtime operation beyond the object access or copy implied by the declaration itself.

### 15.1 The main cost is often copying

Compare:

```cpp
for (auto [key, value] : mapOfBigStructs) {
    // value can involve an element copy
}
```

with:

```cpp
for (const auto& [key, value] : mapOfBigStructs) {
    // no element copy
}
```

If mutation is required:

```cpp
for (auto& [key, value] : mapOfBigStructs) {
    // mutate value in place
}
```

### 15.2 Return small result objects by value

Returning a small struct or tuple and decomposing it at the call site is a normal C++17 pattern:

```cpp
struct Result {
    int value;
    bool ok;
};

Result compute();

auto [value, ok] = compute();
```

Modern C++ compilers can optimize such small value-return patterns very effectively. The structured binding itself should not be treated as a replacement for a performance analysis of the surrounding algorithm.

### 15.3 Tuple-like `get` implementations

For standard tuple-like types such as `std::pair` and `std::tuple`, element access is designed to be efficient. For a custom tuple-like type, poorly designed `get<I>` functions can introduce work that would not otherwise exist.

Keep custom accessors simple, and use `constexpr`/`noexcept` and appropriate reference categories when they are part of the intended interface.

---

## 16. Best Practices

1. **Use meaningful names.** Prefer `[name, age]` to `[a, b]` when the roles are known.
2. **Prefer `const auto&` for read-only range iteration** when you do not want element copies.
3. **Use `auto&` when you intentionally need to mutate existing elements.**
4. **Use `auto` when a separate value is intended.**
5. **Use `auto&&` deliberately in generic code**, especially when preserving lvalue/rvalue behavior is important.
6. **Return named structs for public APIs** when component names carry important semantic meaning.
7. **Use tuples for short, local, positional results** when the meaning of each component is obvious.
8. **Do not expect `_` to ignore an element.**
9. **Remember that structured bindings do not change iterator invalidation rules.**
10. **Understand the hidden object when reasoning about lifetime.**
11. **Use tuple customization carefully** and keep `tuple_size`, `tuple_element`, and `get` consistent.
12. **Avoid unnecessary namespace-scope structured bindings in headers.**
13. **Use `std::tie` when the real requirement is assignment into pre-existing variables.**
14. **Do not treat structured bindings as a way to partially unpack a fixed-size object** unless the language version and binding-pack rules explicitly support what you need.

---

## 17. Structured Bindings in C++20 and Beyond

Structured bindings were introduced in C++17. Later standards added or discussed additional capabilities around them.

### 17.1 C++20: lambda capture

C++20 permits direct lambda capture of structured binding names in supported cases:

```cpp
auto [x, y] = std::pair{1, 2};

auto sum = [x, y] {
    return x + y;
}();
```

This is a useful improvement over the C++17 situation where direct capture of the structured binding name was restricted.

### 17.2 Constant-expression support

Structured bindings have had restrictions around use in constant expressions. The exact capabilities depend on the standard version and the applicable language rules. When writing portable code, check the compiler's supported language mode and the feature-test macro rather than assuming that every `constexpr` declaration form is available merely because structured bindings themselves are supported.

### 17.3 Base-class decomposition

Proposals have explored richer decomposition behavior involving base-class subobjects. The C++17 member-binding model should not be understood as automatically flattening inherited data members into a structured binding list.

### 17.4 C++26: structured binding packs

C++26 introduces structured binding packs, allowing one entry in the binding list to absorb zero or more elements.

Examples include:

```cpp
struct C {
    int x;
    int y;
    int z;
};

auto [a, ...rest] = C{};
auto [...head, tail] = C{};
```

The intended effect is that a pack can absorb the remaining bindings subject to the language rules.

Important rules include:

- only one binding pack can appear in a binding list;
- the pack can be empty;
- the non-pack bindings must still fit the structured binding size.

This is useful when decomposition needs to express a prefix or suffix without requiring a fixed number of explicit names.

### 17.5 What remains fundamental

Across versions, the core mental model remains:

- structured bindings introduce a hidden binding object;
- there are array, tuple-like, and member-binding cases;
- the selected names refer to components of that decomposition;
- lifetime and cv/ref semantics depend on how the hidden object is formed;
- there is no ordinary wildcard in the pre-pack syntax;
- structured bindings are not ordinary function parameter syntax.

---

## 18. Feature-Test Macro

The feature-test macro for structured bindings is:

```cpp
__cpp_structured_bindings
```

The values documented for the relevant standards are:

- `201606L` — C++17 structured bindings
- `202403L` — structured binding packs in C++26

A conditional compilation example is:

```cpp
#if defined(__cpp_structured_bindings)
// Structured bindings are available.
#endif
```

For C++26-specific pack code, check the macro value as well as the compiler's language mode.

---

## 19. Quick Reference Card

| You have | You can write | Main idea |
|---|---|---|
| `std::pair<K,V>` | `auto [k,v] = p;` | `first`, `second` |
| `std::tuple<T...>` | `auto [a,b,c] = t;` | Tuple element order |
| `std::array<T,N>` | `auto [e0,e1,...] = a;` | Exactly N elements in pre-C++26 syntax |
| `T arr[N]` | `auto& [x0,x1,...] = arr;` | Array elements |
| Public-member class | `auto& [a,b] = obj;` | Direct member decomposition |
| Map | `const auto& [k,v] : m` | Key/value iteration |
| Function result struct | `auto [ok,val] = f();` | Named multi-result decomposition |
| `insert()` result | `auto [it,ok] = m.insert(x);` | Iterator + success flag |
| Custom tuple-like type | `auto [a,b] = obj;` | Requires tuple protocol |

### The four common forms

```cpp
auto [a, b] = expr;          // value decomposition
auto& [a, b] = expr;         // mutable reference decomposition
const auto& [a, b] = expr;   // read-only reference decomposition
auto&& [a, b] = expr;        // forwarding/reference form
```

### Cheat rules

- Structured bindings are a C++17 language feature.
- There are three core binding cases: array, tuple-like, and member binding.
- The declaration creates a hidden binding object or reference.
- Binding names designate selected components of that object.
- Binding count normally must match the structured binding size.
- Names must be distinct.
- `_` is not a wildcard.
- `auto` can introduce copies; references can avoid them.
- `decltype(name)` follows special structured-binding rules.
- C++26 adds structured binding packs.

---

## 20. Complete Example

The following program combines several common forms in one example:

```cpp
#include <array>
#include <iostream>
#include <map>
#include <string>
#include <tuple>
#include <utility>

struct Book {
    std::string title;
    int year;
};

std::pair<int, int> divide(int a, int b) {
    return {a / b, a % b};
}

int main() {
    // Pair
    std::pair<std::string, int> version{"C++", 17};
    auto [lang, version_number] = version;
    std::cout << lang << ' ' << version_number << '\n';

    // Array-like type
    std::array<int, 3> rgb{255, 128, 0};
    auto [r, g, b] = rgb;
    std::cout << r << ' ' << g << ' ' << b << '\n';

    // Public-member class
    Book book{"The Standard Library", 2025};
    const auto& [title, pub_year] = book;
    std::cout << title << ' ' << pub_year << '\n';

    // Tuple
    auto row = std::tuple<int, std::string, double>{100, "tea", 3.5};
    auto [id, item, price] = row;
    std::cout << id << ' ' << item << ' ' << price << '\n';

    // Function returning multiple values
    auto [quotient, remainder] = divide(10, 3);
    std::cout << quotient << ' ' << remainder << '\n';

    // Map iteration
    std::map<std::string, int> scores{
        {"math", 98},
        {"physics", 92}
    };

    for (const auto& [subject, score] : scores) {
        std::cout << subject << '=' << score << '\n';
    }
}
```

This example demonstrates pair-like decomposition, array-like decomposition, member decomposition, tuple decomposition, function-result decomposition, and range-based iteration.

---

## 21. Exercises

### Exercise 1 — Return a pair

Write a function that returns `std::pair<int, int>` and decompose its result with structured bindings.

### Exercise 2 — Iterate over a map

Write a loop over:

```cpp
std::map<std::string, std::string>
```

using:

```cpp
const auto& [key, value]
```

### Exercise 3 — Modify through a binding

Create a struct with two integer members. Use `auto& [x, y]` and modify both members through the bindings.

### Exercise 4 — Copy versus reference

Explain the difference between:

```cpp
auto [x, y] = pair;
```

and:

```cpp
auto& [x, y] = pair;
```

Then demonstrate the difference with an assignment to `x`.

### Exercise 5 — Tuple customization

Create a small custom type and add `tuple_size`, `tuple_element`, and `get` support so it can be decomposed with structured bindings.

### Exercise 6 — `decltype`

Create a tuple containing both a value and a reference element. Use `static_assert` and `decltype` to inspect the resulting bindings.

### Exercise 7 — Lifetime

Compare:

```cpp
const auto& [x, y] = std::pair{1, 2};
```

with:

```cpp
// auto& [x, y] = std::pair{1, 2};
```

Explain why the first is valid and the second is not.

### Exercise 8 — Map mutation

Use:

```cpp
for (auto& [key, value] : map)
```

to modify every mapped value while leaving keys unchanged.

### Exercise 9 — `std::optional`

Create a function returning:

```cpp
std::optional<std::pair<int, int>>
```

Check the optional and then decompose its contained pair.

### Exercise 10 — C++26 binding packs

In a C++26 compiler, experiment with a structure containing several members and compare:

```cpp
auto [first, ...rest] = value;
```

and:

```cpp
auto [...head, last] = value;
```

Observe when the pack can become empty.

---

## 22. Conclusion

Structured bindings are a core C++17 feature for decomposing composite values into readable names.

They work through three fundamental cases: arrays, tuple-like types, and classes whose relevant members can be decomposed. The visible syntax is small, but the hidden-object model explains important behavior involving copies, references, `const`, lifetime, and `decltype`.

In everyday code, structured bindings are especially useful for:

- iterating maps and containers of pairs or tuples;
- unpacking function results;
- accessing small public data structures by meaningful local names;
- working with arrays and `std::array`;
- integrating decomposition with C++17 `if`/`switch` initializers.

Use the simplest form that matches your ownership and mutation requirements, and pay particular attention to accidental copies and lifetime when references or temporaries are involved.

For public APIs, prefer meaningful result types when component names matter. For local, obvious positional results, tuples and pairs remain useful.

The most useful mental model is simple: **a structured binding gives names to components of a hidden binding object; it is not merely several unrelated variable declarations.**

---

## 23. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |[Ordikhani](ordikhanifateme@gmail.com) |  | [Ordikhani](@OrdikhaniFateme) |



