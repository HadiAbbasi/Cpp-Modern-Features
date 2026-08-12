<div align="right">

[🇺🇸 English](./tuple.md) | [🇮🇷 فارسی](../../fa/cpp17/tuple.md)

</div>

---
# `std::tuple` in C++17

## Table of Contents

- [Introduction](#introduction)
- [Required Header](#required-header)
- [Creating a `tuple`](#creating-a-tuple)
- [Accessing Elements with `std::get<Index>`](#accessing-elements-with-stdgetindex)
- [Accessing Elements by Type](#accessing-elements-by-type)
- [Modifying Elements](#modifying-elements)
- [`std::make_tuple`](#stdmake_tuple)
- [CTAD in C++17](#ctad-in-c17)
- [Structured Bindings in C++17](#structured-bindings-in-c17)
- [`std::tuple_size` and `std::tuple_element`](#stdtuple_size-and-stdtuple_element)
- [`std::apply` in C++17](#stdapply-in-c17)
- [`std::tie`](#stdtie)
- [Comparing `tuple`s](#comparing-tuples)
- [Returning Multiple Values from a Function](#returning-multiple-values-from-a-function)
- [Difference Between `tuple` and `pair`](#difference-between-tuple-and-pair)
- [Common Errors](#common-errors)
- [Complete Example](#complete-example)
- [Exercises](#exercises)
- [Conclusion](#conclusion)

---

## Introduction

`std::tuple` is a general-purpose class template for storing a fixed number of values that may have different types. Unlike `std::pair`, which always contains exactly two elements, a `tuple` can contain three, four, or more elements.

Example use case:

```cpp
std::tuple<int, std::string, double> record{101, "Ali", 18.75};
```

> `std::tuple` existed before C++17. However, C++17 features such as Structured Bindings, CTAD, and `std::apply` make working with tuples simpler and more readable.

---

## Required Header

To use `std::tuple`, include the `<tuple>` header:

```cpp
#include <tuple>
```

Examples involving strings and output may also require:

```cpp
#include <iostream>
#include <string>
```

---

## Creating a `tuple`

### Explicitly Specifying the Types

```cpp
std::tuple<int, std::string, double> student{
    101, "Sara", 19.5
};
```

### Creating an Empty Tuple

```cpp
std::tuple<> empty;
```

### Creating a Tuple with `std::make_tuple`

```cpp
auto data = std::make_tuple(10, std::string{"C++"}, 3.14);
```

---

## Accessing Elements with `std::get<Index>`

Tuple elements do not have names such as `first` and `second`. They are usually accessed by index:

```cpp
#include <iostream>
#include <string>
#include <tuple>

int main() {
    std::tuple<int, std::string, double> data{10, "C++", 3.14};

    std::cout << std::get<0>(data) << '\\n';
    std::cout << std::get<1>(data) << '\\n';
    std::cout << std::get<2>(data) << '\\n';
}
```

Tuple indices start at zero. The index must be known at compile time. Therefore, the following code is invalid:

```cpp
// int index = 1;
// std::get<index>(data); // Error: index is not a compile-time constant
```

---

## Accessing Elements by Type

If a type appears exactly once in a tuple, you can access the corresponding element by type:

```cpp
std::tuple<int, std::string, double> data{10, "C++", 3.14};

std::cout << std::get<int>(data) << '\\n';
std::cout << std::get<std::string>(data) << '\\n';
```

If a type appears more than once, type-based access is ambiguous:

```cpp
std::tuple<int, int, std::string> values{1, 2, "data"};

// std::get<int>(values); // Error: int appears more than once
```

In such cases, use an index instead.

---

## Modifying Elements

Tuple elements can be modified through `std::get` when the tuple is not `const`:

```cpp
std::tuple<int, std::string, double> data{10, "C++", 3.14};

std::get<0>(data) = 20;
std::get<1>(data) = "C++17";
std::get<double>(data) = 6.28;
```

Elements of a `const` tuple cannot be modified:

```cpp
const auto data = std::make_tuple(10, 3.14);
// std::get<0>(data) = 20; // Error
```

---

## `std::make_tuple`

`std::make_tuple` deduces the types of its elements from its arguments:

```cpp
auto item = std::make_tuple(42, std::string{"Book"}, 99.9);
```

In many situations, `make_tuple` is readable and convenient. However, in C++17 code, CTAD can also be used to construct a tuple without explicitly writing its template arguments.

---

## CTAD in C++17

Since C++17, you can omit the tuple's template arguments when constructing it:

```cpp
std::tuple data{42, std::string{"Ali"}, 18.5};
```

The compiler deduces the tuple's element types from the constructor arguments.

Be careful with string literals:

```cpp
std::tuple data{42, "Ali", 18.5};
```

In this example, the second element is deduced as a pointer type suitable for the string literal rather than as `std::string`. If you want to store a `std::string`, construct it explicitly:

```cpp
std::tuple data{42, std::string{"Ali"}, 18.5};
```

---

## Structured Bindings in C++17

Structured Bindings allow you to unpack tuple elements into named variables:

```cpp
std::tuple<int, std::string, double> data{42, "Ali", 18.5};
auto [id, name, score] = data;
```

This is often more readable than calling `std::get` several times.

### Avoiding Copies

```cpp
const auto& [id, name, score] = data;
```

This binds to the original tuple without copying its elements and prevents modification through the bindings.

### Modifying the Original Elements

```cpp
auto& [id, name, score] = data;
score = 20.0;
```

The bindings refer directly to the tuple's original elements.

The number of Structured Binding variables must match the number of tuple elements:

```cpp
// auto [id, name] = data; // Error: data has three elements
```

---

## `std::tuple_size` and `std::tuple_element`

These type traits are useful in generic and template programming.

### Getting the Number of Elements with `tuple_size`

```cpp
using Data = std::tuple<int, std::string, double>;

constexpr std::size_t count = std::tuple_size_v<Data>;
```

In C++17, the `_v` variable-template form is a shorter version of:

```cpp
std::tuple_size<Data>::value
```

### Getting an Element Type with `tuple_element`

```cpp
using SecondType = std::tuple_element_t<1, Data>;
```

The older form is:

```cpp
using SecondType = typename std::tuple_element<1, Data>::type;
```

---

## `std::apply` in C++17

`std::apply` passes the elements of a tuple as separate arguments to a callable object or function.

```cpp
#include <iostream>
#include <tuple>

void print(int id, double score) {
    std::cout << id << ": " << score << '\\n';
}

int main() {
    auto data = std::make_tuple(10, 19.5);
    std::apply(print, data);
}
```

Output:

```text
10: 19.5
```

### Using `std::apply` with a Lambda

```cpp
#include <iostream>
#include <tuple>

int main() {
    auto data = std::make_tuple(1, 2, 3);

    std::apply([](int a, int b, int c) {
        std::cout << a + b + c << '\\n';
    }, data);
}
```

Output:

```text
6
```

`std::apply` is especially useful for writing generic functions that work with tuples of different sizes and element types.

---

## `std::tie`

`std::tie` creates a tuple of references from existing variables. It can be used to assign tuple elements to those variables:

```cpp
int id;
std::string name;
double score;

std::tie(id, name, score) = data;
```

In C++17, Structured Bindings are usually more readable when you are simply reading a function's result. However, `std::tie` is useful when the target variables already exist or when you want to assign several existing variables at once.

You can ignore an element with `std::ignore`:

```cpp
std::tie(id, std::ignore, score) = data;
```

---

## Comparing `tuple`s

Tuples are compared lexicographically when their corresponding elements are comparable:

```cpp
std::tuple<int, int> a{1, 5};
std::tuple<int, int> b{1, 8};

bool result = a < b; // true
```

Comparison starts with the first element. The next element is compared only if the previous elements are equal.

In C++17, the following comparison operators are available:

```text
==  !=  <  <=  >  >=
```

---

## Returning Multiple Values from a Function

A function can return a tuple containing multiple related values:

```cpp
#include <string>
#include <tuple>

std::tuple<int, std::string, bool> get_result() {
    return {200, "OK", true};
}
```

Usage:

```cpp
#include <iostream>

int main() {
    auto [status, message, successful] = get_result();

    std::cout << status << '\\n';
    std::cout << message << '\\n';
    std::cout << std::boolalpha << successful << '\\n';
}
```

Output:

```text
200
OK
true
```

---

## Difference Between `tuple` and `pair`

| Feature | `std::pair` | `std::tuple` |
|---|---|---|
| Number of elements | Exactly two | Any fixed number, including zero |
| Direct member access | `first` and `second` | None |
| Typical access | Named members | `std::get<index>` or Structured Bindings |
| Readability for two values | Usually better | Usually lower |
| Suitable use | Two related values | Multiple heterogeneous values |
| Header | `<utility>` | `<tuple>` |

If there are exactly two related values, `std::pair` is usually simpler. If there are more than two values, `std::tuple` is a suitable option.

If the elements have clear domain-specific meanings, a `struct` is often more readable than either `pair` or `tuple`:

```cpp
struct User {
    int id;
    std::string name;
    double score;
};
```

---

## Common Errors

### 1. Using an Out-of-Range Index

```cpp
std::tuple<int, double> data{1, 2.5};
// std::get<2>(data); // Error: only indices 0 and 1 are valid
```

### 2. Ambiguous Type-Based Access

```cpp
std::tuple<int, int> values{1, 2};
// std::get<int>(values); // Error: int appears more than once
```

### 3. Confusing a `tuple` with an Array

A `tuple` is intended for a fixed number of elements that may have different types. For a collection of same-type elements whose size may change at runtime, `std::vector` is usually more appropriate.

### 4. Forgetting the C++17 Standard Flag

Compile with C++17 enabled:

```bash
g++ -std=c++17 main.cpp -o main
```

`std::apply` was added to the standard library in C++17.

### 5. Unintentional Copies in Structured Bindings

```cpp
auto [a, b, c] = data; // Copies the elements
```

For read-only access without copying:

```cpp
const auto& [a, b, c] = data;
```

### 6. Using a `tuple` Instead of a Named Type

If the data members have clear roles, this code:

```cpp
std::tuple<int, std::string, double> user;
```

may be less readable than:

```cpp
struct User {
    int id;
    std::string name;
    double score;
};
```

---

## Complete Example

```cpp
#include <iostream>
#include <string>
#include <tuple>

std::tuple<int, std::string, double> get_student() {
    return {101, "Sara", 19.25};
}

void print_student(int id, const std::string& name, double score) {
    std::cout << id << " | " << name << " | " << score << '\\n';
}

int main() {
    std::tuple student{101, std::string{"Ali"}, 18.75};

    std::cout << std::get<0>(student) << '\\n';
    std::cout << std::get<std::string>(student) << '\\n';

    auto& [id, name, score] = student;
    score = 20.0;

    std::cout << id << " | " << name << " | " << score << '\\n';

    auto result = get_student();
    std::apply(print_student, result);

    constexpr auto count = std::tuple_size_v<decltype(student)>;
    std::cout << "Members: " << count << '\\n';
}
```

Compile and run:

```bash
g++ -std=c++17 -Wall -Wextra -pedantic main.cpp -o main
./main
```

---

## Exercises

1. Write a function that returns a student's name, age, and score as a `tuple`.
2. Create a four-element `tuple` and print its elements using `std::get`.
3. Unpack the same `tuple` using Structured Bindings.
4. Use `std::apply` to write a function that prints all elements of a `tuple`.
5. Create a `tuple` containing two `int` elements and explain why `std::get<int>` is not valid for it.
6. Use `std::tie` to extract only the first and third elements of a `tuple`, ignoring the second element with `std::ignore`.
7. Explain the difference between `std::make_tuple` and CTAD with an example.

---

## Conclusion

`std::tuple` is useful for storing multiple values with potentially different types. The most important tools for working with it are:

- `std::get<Index>` for index-based access
- `std::get<Type>` for type-based access when the type is unique
- `std::make_tuple` for convenient construction
- **CTAD in C++17** for omitting template arguments during construction
- **Structured Bindings in C++17** for unpacking elements
- **`std::apply` in C++17** for passing tuple elements to a function
- `std::tuple_size` and `std::tuple_element` for generic programming

Nevertheless, when the data members have clear names and domain-specific meanings, a `struct` is often a more professional and readable choice.

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [Ordikhani](https://github.com/Ordikhani) | []() | []() | []() | []() |

</div>