<div align="right">

[🇺🇸 English](./pair.md) | [🇮🇷 فارسی](../../fa/cpp17/pair.md)

</div>
---

# `std::pair` in C++17: A Comprehensive Guide

## Table of Contents

- [Introduction](#introduction)
- [Required Headers](#required-headers)
- [Structure of `std::pair`](#structure-of-stdpair)
- [Creating a `pair`](#creating-a-pair)
- [Accessing Members](#accessing-members)
- [Modifying a `pair`](#modifying-a-pair)
- [`std::make_pair`](#stdmake_pair)
- [Class Template Argument Deduction (CTAD) in C++17](#class-template-argument-deduction-ctad-in-c17)
- [Structured Bindings in C++17](#structured-bindings-in-c17)
- [Returning Multiple Values from Functions](#returning-multiple-values-from-functions)
- [Unpacking with `std::tie` and `std::ignore`](#unpacking-with-stdtie-and-stdignore)
- [Comparing `pair`s](#comparing-pairs)
- [Copy, Move Semantics, and `const`](#copy-move-semantics-and-const)
- [Using `std::pair` in `std::map`](#using-stdpair-in-stdmap)
- [Differences: `std::pair` vs. `std::tuple` vs. `struct`](#differences-stdpair-vs-stdtuple-vs-struct)
- [Common Pitfalls](#common-pitfalls)
- [Complete Executable Example](#complete-executable-example)
- [Exercises & Solutions](#exercises--solutions)
- [Conclusion](#conclusion)

---

## Introduction

`std::pair` is a class template in the C++ Standard Library that stores exactly two values. The values may have different types, such as `std::string` and `int`.

It is useful when two values are logically related and need to be stored or passed together, for example:

- Key-value pairs, as used by `std::map`
- An operation status and a returned value
- Two-dimensional coordinates, such as `{x, y}`
- Range boundaries, such as `{min, max}`
- A quotient and a remainder

`std::pair` existed before C++17. However, C++17 features such as **Class Template Argument Deduction (CTAD)** and **Structured Bindings** make it easier and more readable to use.

---

## Required Headers

To use `std::pair`, include `<utility>`:

```cpp
#include <utility>
```

For examples involving strings, include `<string>`:

```cpp
#include <string>
```

Other features may require additional headers:

```cpp
#include <algorithm> // std::minmax_element
#include <iostream>  // std::cout
#include <map>       // std::map
#include <tuple>     // std::tie, std::ignore
#include <vector>    // std::vector
```

---

## Structure of `std::pair`

The general form is:

```cpp
std::pair<FirstType, SecondType> variable;
```

A `pair` exposes two public data members:

```cpp
variable.first   // The first element
variable.second  // The second element
```

Conceptually, its declaration is similar to:

```cpp
template <typename T1, typename T2>
struct pair;
```

The actual standard-library implementation contains additional constructors, assignment operators, comparison operators, and helper functionality.

---

## Creating a `pair`

### Brace Initialization

Brace initialization is the most common modern syntax:

```cpp
#include <string>
#include <utility>

std::pair<std::string, int> user{"Sara", 24};
```

### Constructor Initialization

You can also initialize a pair using parentheses:

```cpp
std::pair<int, double> measurement(10, 3.14);
```

### Default Initialization

```cpp
std::pair<int, std::string> item;
```

The members are value-initialized. Therefore, the `int` becomes `0`, and the `std::string` becomes an empty string.

### Assigning After Construction

```cpp
std::pair<int, int> point;
point.first = 2;
point.second = 4;
```

---

## Accessing Members

### Direct Member Access

Use `.first` and `.second`:

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    std::pair<std::string, int> user{"Ali", 30};

    std::cout << user.first << '\\n';
    std::cout << user.second << '\\n';
}
```

Output:

```text
Ali
30
```

### Access with `std::get`

You can access pair elements by index:

```cpp
std::pair<int, std::string> data{42, "Universe"};

int number = std::get<0>(data);
std::string text = std::get<1>(data);
```

You can also access an element by type if the type occurs exactly once:

```cpp
#include <string>
#include <utility>

std::pair<int, std::string> data{42, "Universe"};

int number = std::get<int>(data);
std::string text = std::get<std::string>(data);
```

The following is invalid because the type is ambiguous:

```cpp
std::pair<int, int> values{10, 20};
// std::get<int>(values); // Error: int occurs twice
```

For a `pair`, `.first` and `.second` are usually the clearest access syntax.

---

## Modifying a `pair`

The members are public and can be modified directly unless the pair is `const`:

```cpp
std::pair<std::string, int> user{"Ali", 30};

user.first = "Reza";
user.second = 35;
```

You can assign another pair to it:

```cpp
std::pair<int, int> point{2, 4};
point = std::make_pair(10, 20);
```

You can also assign using a braced value:

```cpp
point = {100, 200};
```

---

## `std::make_pair`

`std::make_pair` deduces the pair's element types from its arguments:

```cpp
#include <string>
#include <utility>

auto user = std::make_pair(std::string{"Mina"}, 27);
```

The type of `user` is effectively:

```cpp
std::pair<std::string, int>
```

### C-style String Note

In this example, the second element is a C-style string pointer:

```cpp
auto value = std::make_pair(10, "C++");
```

Its type is effectively:

```cpp
std::pair<int, const char*>
```

If you want a real `std::string`, construct one explicitly:

```cpp
auto value = std::make_pair(10, std::string{"C++"});
```

---

## Class Template Argument Deduction (CTAD) in C++17

C++17 allows the compiler to deduce class template arguments from constructor arguments:

```cpp
std::pair user{"Ali", 25};
```

The deduced type is effectively:

```cpp
std::pair<const char*, int>
```

To obtain `std::string` instead of `const char*`, write:

```cpp
std::pair user{std::string{"Ali"}, 25};
```

The deduced type is then effectively:

```cpp
std::pair<std::string, int>
```

> CTAD and `auto` are related but separate language features. CTAD applies to declarations such as `std::pair user{...}`, while `auto` deduces the type of a variable from its initializer.

---

## Structured Bindings in C++17

Structured Bindings allow you to decompose a pair into named variables:

```cpp
std::pair<std::string, int> user{"Ali", 25};
auto [name, age] = user;
```

### Binding by Value

```cpp
auto [name, age] = user;
```

`name` and `age` are used as value bindings. Changes to them do not modify `user`.

### Binding by Reference

```cpp
auto& [name, age] = user;

name = "Sara";
age = 28;
```

Here, `name` and `age` refer to the original members, so modifying them modifies `user`.

### Read-Only Reference Binding

```cpp
const auto& [name, age] = user;
```

This avoids copying while preventing modification through the bindings.

### Moving into Structured Bindings

You may also move a pair into structured bindings:

```cpp
auto [name, age] = std::move(user);
```

This can move the pair's members into the new variables when the member types support moving.

---

## Returning Multiple Values from Functions

A function can return a `std::pair` containing two related results:

```cpp
#include <utility>

std::pair<int, int> divide(int dividend, int divisor) {
    return {dividend / divisor, dividend % divisor};
}
```

Usage:

```cpp
#include <iostream>

int main() {
    auto [quotient, remainder] = divide(17, 5);

    std::cout << "Quotient: " << quotient << '\\n';
    std::cout << "Remainder: " << remainder << '\\n';
}
```

Output:

```text
Quotient: 3
Remainder: 2
```

For more than two results, or when the results need meaningful names, a custom `struct` may be more expressive.

---

## Unpacking with `std::tie` and `std::ignore`

`std::tie` can unpack a pair into variables that already exist. It is especially useful when one of the values should be ignored:

```cpp
#include <tuple>
#include <utility>

std::pair<int, double> get_sensor_data() {
    return {42, 21.5};
}

int main() {
    int id;

    std::tie(id, std::ignore) = get_sensor_data();
}
```

In modern C++17 code, Structured Bindings are usually more readable when all values are needed:

```cpp
auto [id, temperature] = get_sensor_data();
```

However, `std::tie` remains useful when assigning to existing variables.

---

## Comparing `pair`s

Pairs are compared lexicographically, similar to dictionary ordering:

1. The `.first` elements are compared.
2. If the first elements are equivalent, the `.second` elements are compared.

Example:

```cpp
#include <utility>

std::pair<int, int> a{1, 5};
std::pair<int, int> b{1, 8};

bool result = a < b; // true
```

Since the first elements are equal, `5` is compared with `8`.

In C++17, relational operators include:

```text
==  !=  <  <=  >  >=
```

The element types must support the required comparisons. This behavior is useful when sorting pairs or using them in ordered containers such as `std::map` and `std::set`.

> C++20 additionally provides three-way comparison support through `<=>` when the element types support it.

---

## Copy, Move Semantics, and `const`

A pair can be copied like any other copyable object:

```cpp
std::pair<std::string, int> original{"Ali", 25};
auto copy = original;
```

A read-only reference avoids copying:

```cpp
const auto& view = original;
```

### Move Semantics

If a pair contains expensive-to-copy objects, such as `std::vector` or a large `std::string`, it can be moved:

```cpp
#include <string>
#include <utility>
#include <vector>

std::pair<std::string, std::vector<int>> original{
    "data", {1, 2, 3}
};

auto destination = std::move(original);
```

Moving transfers or reuses resources when possible. After moving, `original` remains valid, but its values should not be assumed to be unchanged.

### References in Range-Based Loops

Use `const auto&` for read-only iteration without copying:

```cpp
for (const auto& item : users) {
    // Read-only use of item
}
```

Use `auto&` when you need to modify the elements:

```cpp
for (auto& item : users) {
    item.second++;
}
```

---

## Using `std::pair` in `std::map`

A `std::map<Key, T>` stores each element as:

```cpp
std::pair<const Key, T>
```

The key is `const` so that changing it would not violate the map's ordering invariants.

Structured Bindings make map iteration concise:

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> scores{
        {"Ali", 18},
        {"Sara", 20}
    };

    for (const auto& [name, score] : scores) {
        std::cout << name << ": " << score << '\\n';
    }
}
```

Without Structured Bindings, the same elements can be accessed with `.first` and `.second`:

```cpp
for (const auto& entry : scores) {
    std::cout << entry.first << ": " << entry.second << '\\n';
}
```

---

## Differences: `std::pair` vs. `std::tuple` vs. `struct`

| Feature | `std::pair` | `std::tuple` | `struct` |
|---|---|---|---|
| Number of members | Exactly two | Zero or more | Custom |
| Member access | `.first`, `.second` | `std::get<N>` or Structured Bindings | Custom names |
| Semantic naming | Limited | Limited | Excellent |
| Suitable use | Two related values | Several heterogeneous values | Domain models |
| Header | `<utility>` | `<tuple>` | No special header |

Use `std::pair` when exactly two values are related and generic names such as `first` and `second` are sufficient.

Use `std::tuple` when you need three or more values but do not need named domain-specific members.

Use a `struct` when readability, maintainability, and meaningful member names are important:

```cpp
struct User {
    std::string name;
    int age;
};
```

---

## Common Pitfalls

### 1. Accidentally Copying Structured-Binding Variables

```cpp
auto [name, age] = user;  // Value binding
```

If you need to modify the original pair, use:

```cpp
auto& [name, age] = user;  // Reference binding
```

### 2. Ambiguous Type-Based Access

This is invalid because `int` appears twice:

```cpp
std::pair<int, int> values{10, 20};
// std::get<int>(values); // Compilation error
```

Use an index instead:

```cpp
int first = std::get<0>(values);
int second = std::get<1>(values);
```

### 3. C-String Deduction

This declaration deduces the first member as `const char*`:

```cpp
std::pair user{"Ali", 25};
```

Use an explicit `std::string` when string semantics are desired:

```cpp
std::pair user{std::string{"Ali"}, 25};
```

### 4. Forgetting the C++17 Compiler Flag

Compile with C++17 enabled:

```bash
g++ -std=c++17 main.cpp -o main
```

For Clang:

```bash
clang++ -std=c++17 main.cpp -o main
```

### 5. Expecting Custom Member Names

A `std::pair` always exposes `.first` and `.second`. It does not provide members such as `.name` or `.age`.

If semantic names are important, use a `struct` instead.

### 6. Ignoring Empty-Range Cases

Functions using algorithms such as `std::minmax_element` must handle an empty range before dereferencing returned iterators.

---

## Complete Executable Example

```cpp
#include <iostream>
#include <map>
#include <string>
#include <utility>

std::pair<std::string, int> get_user() {
    return {"Nima", 29};
}

std::pair<int, int> divide(int dividend, int divisor) {
    return {dividend / divisor, dividend % divisor};
}

int main() {
    std::pair user{std::string{"Ali"}, 25};

    std::cout << user.first << " - " << user.second << '\\n';

    auto [name, age] = user;
    std::cout << name << " is " << age << " years old\\n";

    auto [returned_name, returned_age] = get_user();
    std::cout << returned_name << " - " << returned_age << '\\n';

    auto [quotient, remainder] = divide(17, 5);
    std::cout << "Quotient: " << quotient << '\\n';
    std::cout << "Remainder: " << remainder << '\\n';

    std::map<std::string, int> scores{
        {"Ali", 18},
        {"Sara", 20}
    };

    for (const auto& [student, score] : scores) {
        std::cout << student << ": " << score << '\\n';
    }
}
```

Compile and run:

```bash
g++ -std=c++17 -Wall -Wextra -pedantic main.cpp -o main
./main
```

---

## Exercises & Solutions

### Exercise 1: Find the Minimum and Maximum

Write a function that returns the minimum and maximum values of a `std::vector<int>` as a pair.

#### Solution

```cpp
#include <algorithm>
#include <stdexcept>
#include <utility>
#include <vector>

std::pair<int, int> find_min_max(const std::vector<int>& values) {
    if (values.empty()) {
        throw std::invalid_argument("The vector must not be empty");
    }

    auto [min_it, max_it] =
        std::minmax_element(values.begin(), values.end());

    return {*min_it, *max_it};
}
```

### Exercise 2: Compare Two Pairs

Create two `std::pair<int, int>` objects and determine their ordering using `<`.

#### Solution

```cpp
#include <iostream>
#include <utility>

int main() {
    std::pair<int, int> first{1, 5};
    std::pair<int, int> second{1, 8};

    if (first < second) {
        std::cout << "first is less than second\\n";
    }
}
```

### Exercise 3: Iterate over a Map

Create a `std::map<std::string, double>` and iterate over it using Structured Bindings.

#### Solution

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, double> prices{
        {"Book", 12.50},
        {"Pen", 2.25}
    };

    for (const auto& [product, price] : prices) {
        std::cout << product << ": " << price << '\\n';
    }
}
```

### Exercise 4: Return Quotient and Remainder

Write a function that returns the quotient and remainder of an integer division.

#### Solution

```cpp
#include <utility>

std::pair<int, int> divide(int dividend, int divisor) {
    return {dividend / divisor, dividend % divisor};
}
```

### Exercise 5: Compare Copy and Reference Bindings

Create a pair and unpack it once by value and once by `const` reference. Explain the difference.

#### Solution

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    std::pair<std::string, int> user{"Ali", 25};

    auto [copied_name, copied_age] = user;
    const auto& [referenced_name, referenced_age] = user;

    copied_name = "Reza";

    std::cout << user.first << '\\n'; // Ali
    std::cout << copied_name << '\\n'; // Reza
    std::cout << referenced_name << '\\n'; // Ali
}
```

The first binding creates independent values. The second binding refers to the original pair without copying and does not allow modification.

---

## Conclusion

`std::pair` is a simple and standard way to store exactly two related values. It is particularly useful for return values, key-value relationships, coordinates, ranges, and other small logical groupings.

C++17 makes `std::pair` more convenient through:

- **Class Template Argument Deduction (CTAD)** for shorter declarations
- **Structured Bindings** for readable decomposition
- Continued support for copy and move semantics

Use `std::pair` when generic member names such as `first` and `second` are sufficient. If your data has more than two members or requires meaningful domain-specific names, prefer `std::tuple` or, often, a custom `struct`.

---
## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [Ordikhani](https://github.com/Ordikhani) | []() | []() | []() | []() |

</div>