<div align="right">

[🇺🇸 English](./apply.md) | [🇮🇷 فارسی](../../fa/cpp17/apply.md)

</div>
---

# `std::apply` in C++17

## Table of Contents

- [Introduction](#introduction)
- [Required Header](#required-header)
- [How it Works](#how-it-works)
- [Basic Usage](#basic-usage)
- [Using with Lambda Functions](#using-with-lambda-functions)
- [Using with Class Methods](#using-with-class-methods)
- [Practical Example: `std::apply` with `std::tuple`](#practical-example-stdapply-with-stdtuple)
- [Common Use Cases](#common-use-cases)
- [Difference between `std::apply` and `std::invoke`](#difference-between-stdapply-and-stdinvoke)
- [Contributors](#contributors)

---

## Introduction

`std::apply` is a utility function introduced in C++17 that invokes a Callable object (function, lambda, functor) by "unpacking" the elements of a tuple (or any pair-like object) as separate arguments to that function.

Before C++17, implementing this required complex template metaprogramming or recursive variadic templates. `std::apply` simplifies this into a single line.

---

## Required Header

`std::apply` is defined in the `<tuple>` header.

```cpp
#include <tuple>
```

---

## How it Works

`std::apply(f, t)` essentially performs the following conceptual call:

```cpp
f(std::get<0>(t), std::get<1>(t), ..., std::get<N-1>(t))
```

It takes a function `f` and a tuple `t` containing `N` elements, and expands those elements into the argument list of `f`.

---

## Basic Usage

```cpp
#include <iostream>
#include <tuple>

int add(int a, int b) {
    return a + b;
}

int main() {
    auto my_tuple = std::make_tuple(10, 20);
    
    // Calls add(10, 20)
    int result = std::apply(add, my_tuple); 
    
    std::cout << "Result: " << result << '\n'; // 30
}
```

---

## Using with Lambda Functions

`std::apply` works perfectly with lambdas, especially for concise operations:

```cpp
auto t = std::make_tuple(10, 2.5);

std::apply([](auto a, auto b) {
    std::cout << a + b << '\n';
}, t); // Prints 12.5
```

---

## Using with Class Methods

You can use `std::apply` with member functions by passing a pointer to the member function and an instance of the class (or a reference to it) as part of the tuple:

```cpp
struct Calculator {
    int multiply(int a, int b) { return a * b; }
};

int main() {
    Calculator calc;
    auto t = std::make_tuple(&Calculator::multiply, &calc, 5, 10);
    
    // Note: The first two elements are the method pointer and object instance
    // std::apply handles the syntax to call (calc.*multiply)(5, 10)
    int result = std::apply(std::invoke, t); 
}
```

---

## Practical Example: `std::apply` with `std::tuple`

A common scenario is processing configurations stored in a tuple:

```cpp
#include <iostream>
#include <tuple>
#include <string>

void print_config(int port, const std::string& host, bool enabled) {
    std::cout << "Host: " << host << ", Port: " << port 
              << ", Enabled: " << (enabled ? "yes" : "no") << '\n';
}

int main() {
    auto config = std::make_tuple(8080, "localhost", true);
    
    std::apply(print_config, config);
}
```

---

## Common Use Cases

1. **Configuring objects**: Passing a tuple of parameters to a constructor or initialization function.
2. **Generic algorithms**: Writing generic code that processes arbitrary data structures stored in tuples.
3. **Mathematical operations**: Performing operations on tuple elements (e.g., adding a vector of numbers stored as a tuple).

---

## Difference between `std::apply` and `std::invoke`

- **`std::invoke`**: A C++17 utility to call any callable object (function, pointer to member, etc.) uniformly.
- **`std::apply`**: Specifically designed to expand a *tuple* into the arguments of a function call.

They are often used together, as shown in the class method example.

---
## 🤝 Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |  |  |  |
