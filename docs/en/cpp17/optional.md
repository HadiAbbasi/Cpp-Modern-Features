# `std::optional` in C++17

## Table of Contents

- [Introduction](#introduction)
- [Required Header](#required-header)
- [What `std::optional` Represents](#what-stdoptional-represents)
- [Construction and Empty State](#construction-and-empty-state)
- [Checking for a Value](#checking-for-a-value)
- [Accessing the Contained Value](#accessing-the-contained-value)
- [Providing Defaults](#providing-defaults)
- [Modifying the Stored Value](#modifying-the-stored-value)
- [Optional as a Return Type](#optional-as-a-return-type)
- [Why `optional` Is Better Than Magic Values](#why-optional-is-better-than-magic-values)
- [Monadic Operations in C++23](#monadic-operations-in-c23)
- [Range Support in C++26](#range-support-in-c26)
- [Common Errors](#common-errors)
- [Feature-Test Macro](#feature-test-macro)
- [Complete Example](#complete-example)
- [Exercises](#exercises)
- [Conclusion](#conclusion)
- [Contributors](#contributors)

---

## Introduction

`std::optional` is a C++17 class template that models a value that may or may not be present. It is defined in the `<optional>` header.

```cpp
template<class T>
class optional;
```

An `optional<T>` either contains a `T` object or contains nothing. That makes it a good fit for functions that may fail, configuration values that may be omitted, and APIs where absence is a normal outcome rather than an error.

Compared with sentinel values such as `-1`, `""`, or `nullptr`, `std::optional` makes absence explicit in the type system. That improves readability and reduces accidental misuse.

---

## Required Header

Include `<optional>` to use `std::optional`:

```cpp
#include <optional>
```

You may also need these headers depending on the example:

```cpp
#include <iostream>
#include <string>
```

---

## What `std::optional` Represents

An `optional<T>` has two states:

- engaged: it contains a `T`
- disengaged: it contains no value

The object is not a pointer, even though it supports `operator*()` and `operator->()`. If a value exists, it is stored inside the `optional` object itself.

A contextual conversion to `bool` tells you whether a value is present:

```cpp
std::optional<int> x = 42;
if (x) {
    // x contains a value
}
```

The standard library also provides `std::nullopt`, a tag used to represent the empty state explicitly.

---

## Construction and Empty State

### Default construction

A default-constructed `optional` is empty:

```cpp
std::optional<int> a;
```

### Constructing with a value

```cpp
std::optional<int> b = 42;
std::optional<std::string> c{"hello"};
```

### Constructing an empty optional explicitly

```cpp
std::optional<int> d = std::nullopt;
std::optional<std::string> e{};
```

### In-place construction

Use `std::in_place` when you want to construct the contained object directly:

```cpp
std::optional<std::string> s{std::in_place, 5, 'x'}; // "xxxxx"
```

### Resetting to empty

```cpp
std::optional<int> value = 10;
value.reset();
```

After `reset()`, the optional no longer contains a value.

---

## Checking for a Value

The two common ways to check whether an `optional` contains a value are `has_value()` and the boolean context conversion.

### `has_value()`

```cpp
std::optional<int> value = 7;
if (value.has_value()) {
    // use *value
}
```

### Boolean context

```cpp
if (value) {
    // also means: value has a contained object
}
```

The boolean form is concise and widely used, especially in conditions.

---

## Accessing the Contained Value

### `operator*` and `operator->`

If the optional contains a value, you can access it with dereference syntax:

```cpp
std::optional<std::string> name = "Ada";
std::cout << *name << '\n';
```

For object types, `operator->` works too:

```cpp
std::optional<std::string> name = "Ada";
std::cout << name->size() << '\n';
```

These operators do not perform a runtime check. Using them on an empty optional is undefined behavior.

### `value()`

`value()` returns the contained object, and throws `std::bad_optional_access` if the optional is empty:

```cpp
std::optional<int> n = 5;
int x = n.value();
```

Use `value()` when an empty state is exceptional and should be reported as an error.

---

## Providing Defaults

`value_or()` is the usual way to supply a fallback when the optional is empty.

```cpp
std::optional<int> port;
int actual_port = port.value_or(8080);
```

The fallback expression is evaluated when needed, and the result is returned by value.

This is often the cleanest way to keep call sites simple while still expressing optionality in the API.

---

## Modifying the Stored Value

### Assignment

```cpp
std::optional<int> value;
value = 10;
value = std::nullopt;
```

### `emplace()`

`emplace()` constructs a new contained object in place:

```cpp
std::optional<std::string> text;
text.emplace(4, 'a'); // "aaaa"
```

### `swap()`

```cpp
std::optional<int> a = 1;
std::optional<int> b = 2;
a.swap(b);
```

`swap()` exchanges both the engaged state and the contained values.

---

## Optional as a Return Type

A very common use case is a function that may fail without using exceptions.

### Example: safe division

```cpp
#include <iostream>
#include <optional>

std::optional<int> divide(int a, int b) {
    if (b == 0) {
        return std::nullopt;
    }
    return a / b;
}

int main() {
    auto result = divide(10, 2);
    if (result) {
        std::cout << *result << '\n';
    } else {
        std::cout << "division by zero\n";
    }
}
```

### Example: reading a file

```cpp
#include <fstream>
#include <optional>
#include <sstream>
#include <string>

std::optional<std::string> read_file(const std::string& path) {
    std::ifstream file(path);
    if (!file.is_open()) {
        return std::nullopt;
    }

    std::ostringstream out;
    out << file.rdbuf();
    return out.str();
}
```

Returning `optional` makes the success and failure paths visible in the type signature.

---

## Why `optional` Is Better Than Magic Values

Sentinel values are fragile because they overload a normal value with a special meaning.

### Problem with magic values

```cpp
int get_port(); // returns -1 if not set
```

This signature lies a little. It says it returns an `int`, but the caller must also know that `-1` means “missing”. That creates several problems:

- the caller may forget the special-case check
- the sentinel may collide with a real value later
- different APIs may use different sentinels
- the meaning is not visible in the type system

### Better with `optional`

```cpp
std::optional<int> get_port();
```

Now the type itself tells the truth: there may be no value. That reduces ambiguity and makes invalid states harder to ignore.

### Configuration example

```cpp
struct AppConfig {
    std::optional<int> port;

    int port_or_default() const {
        return port.value_or(8080);
    }
};
```

This is clearer than using `-1`, `0`, or another magic value to mean “unset”.

---

## Monadic Operations in C++23

C++23 adds three useful operations for chaining logic with optionals:

- `and_then()`
- `transform()`
- `or_else()`

### `and_then()`

Calls a function only when the optional contains a value. The callable must return another optional.

```cpp
std::optional<int> x = 10;
auto y = x.and_then([](int v) -> std::optional<int> {
    if (v > 0) return v * 2;
    return std::nullopt;
});
```

### `transform()`

Applies a function to the contained value and wraps the result in an optional.

```cpp
auto length = std::optional<std::string>{"hello"}
    .transform([](const std::string& s) { return s.size(); });
```

### `or_else()`

Provides an alternative optional when the original one is empty.

```cpp
auto port = std::optional<int>{}
    .or_else([] { return std::optional<int>{8080}; });
```

These operations reduce repetitive `if` blocks when processing optional data.

---

## Range Support in C++26

In C++26, `std::optional<T>` is treated as a view when it contains zero or one element. That means it can participate in range-based code more naturally.

Conceptually:

- engaged optional -> one element
- empty optional -> zero elements

This makes optional fit better into the ranges ecosystem without changing its basic meaning.

---

## Common Errors

1. Dereferencing an empty optional with `*opt` or `opt->member`.
2. Calling `value()` without handling `std::bad_optional_access`.
3. Using `optional<T>` where `T*` is the real model, such as optional ownership or polymorphic lifetime management.
4. Using `optional` for error details when a richer result type such as `std::expected` would be a better fit.
5. Returning references to local objects inside an optional.
6. Treating `optional` like a pointer and forgetting that it stores an object, not an address.

---

## Feature-Test Macro

The feature-test macro for `std::optional` is `__cpp_lib_optional`.

- `201606L` - C++17 `std::optional`
- `202106L` - fully constexpr support in C++23
- `202110L` - monadic operations in C++23

For C++26 range support, the relevant macro is `__cpp_lib_optional_range_support` with value `202406L`.

---

## Complete Example

```cpp
#include <iostream>
#include <optional>
#include <string>

std::optional<std::string> create(bool ok) {
    if (ok) {
        return std::string{"Godzilla"};
    }
    return std::nullopt;
}

int main() {
    std::optional<std::string> a = create(false);
    std::optional<std::string> b = create(true);

    std::cout << "a: " << a.value_or("empty") << '\n';

    if (b) {
        std::cout << "b: " << *b << '\n';
    }

    std::optional<int> port;
    std::cout << "port: " << port.value_or(8080) << '\n';

    port.emplace(3000);
    std::cout << "updated port: " << port.value() << '\n';
}
```

This example shows the core workflow: create an optional, test it, read it safely, fall back to a default, and update it in place.

---

## Exercises

1. Write a function that parses an integer from a string and returns `std::optional<int>`.
2. Replace a sentinel-based configuration API with `std::optional`.
3. Use `value_or()` to provide a default theme name in a settings loader.
4. Write a function that returns the length of a string only when the input is not empty.
5. Chain two optional-returning functions using `and_then()` in C++23.

---

## Conclusion

`std::optional` is the standard way to represent “value or no value” in modern C++. It makes absence explicit, removes the need for magic values, and gives callers a clear contract to work with.

Use it when a missing result is normal and should be represented directly in the type system. Use `value_or()` when a default is appropriate, and use `value()` only when empty is genuinely exceptional.

---

## Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |  |  |  |
