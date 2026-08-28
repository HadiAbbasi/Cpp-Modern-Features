# `std::any` in C++17

## Table of Contents

- [Introduction](#introduction)
- [Required Header](#required-header)
- [How `std::any` Works](#how-stdany-works)
- [Basic Usage](#basic-usage)
- [Checking Whether a Value Exists](#checking-whether-a-value-exists)
- [Retrieving Values with `std::any_cast`](#retrieving-values-with-stdany_cast)
- [Handling Invalid Casts](#handling-invalid-casts)
- [Pointer Form of `std::any_cast`](#pointer-form-of-stdany_cast)
- [Inspecting the Stored Type](#inspecting-the-stored-type)
- [Creating Values with `emplace` and `make_any`](#creating-values-with-emplace-and-make_any)
- [Resetting and Swapping Values](#resetting-and-swapping-values)
- [Copy and Move Behavior](#copy-and-move-behavior)
- [`std::any` and Dynamic Allocation](#stdany-and-dynamic-allocation)
- [`std::any` vs. `void*`](#stdany-vs-void)
- [`std::any` vs. `std::variant`](#stdany-vs-stdvariant)
- [When to Use `std::any`](#when-to-use-stdany)
- [Common Errors](#common-errors)
- [Complete Example](#complete-example)
- [Exercises](#exercises)
- [Conclusion](#conclusion)
- [Contributors](#contributors)

---

## Introduction

`std::any` is a type-safe container introduced in C++17 that can store one value of almost any type.

Unlike a normal variable, which has a fixed type, a `std::any` object can hold an `int`, then later hold a `std::string`, a custom class, or another copyable type.

```cpp
#include <any>
#include <string>

std::any value = 42;                 // Stores an int
value = std::string("Hello");        // Now stores a std::string
value = 3.14;                        // Now stores a double
```

At any given time, a `std::any` object is in one of two states:

1. **Empty:** It does not contain a value.
2. **Non-empty:** It contains exactly one object of a specific type.

`std::any` is useful when the type of a value is not known at compile time, such as in plugin systems, metadata containers, configuration systems, event payloads, or generic property bags.

---

## Required Header

To use `std::any`, include the `<any>` header:

```cpp
#include <any>
```

`std::any` is available starting from C++17.

Compile with:

```bash
g++ main.cpp -std=c++17 -Wall -Wextra -o app
```

For MSVC:

```powershell
cl main.cpp /std:c++17 /EHsc
```

---

## How `std::any` Works

`std::any` uses a technique called **type erasure**.

Internally, it stores:

- the contained value;
- information about the value's actual type;
- operations needed to copy, move, destroy, and access the value safely.

For example:

```cpp
std::any value = 10;
```

The object contains an `int`.

Later:

```cpp
value = std::string("C++");
```

The previously stored `int` is destroyed, and a `std::string` is stored instead.

Important: `std::any` does **not** perform implicit conversions between stored types.

```cpp
std::any value = 10; // Stores an int

// This is invalid at runtime because the stored type is int, not double.
double number = std::any_cast<double>(value);
```

Even though C++ can normally convert `int` to `double`, `std::any_cast<double>` requires the stored type to match `double`.

---

## Basic Usage

A `std::any` can be initialized with a value of any copy-constructible type.

```cpp
#include <any>
#include <iostream>
#include <string>

int main() {
    std::any value = 42;

    std::cout << std::any_cast<int>(value) << '
';

    value = std::string("Hello, std::any!");

    std::cout << std::any_cast<std::string>(value) << '
';

    value = true;

    std::cout << std::boolalpha
              << std::any_cast<bool>(value)
              << '
';
}
```

Possible output:

```text
42
Hello, std::any!
true
```

The same `std::any` object stores a different type each time it is assigned.

---

## Checking Whether a Value Exists

Use `has_value()` to determine whether a `std::any` object currently contains a value.

```cpp
#include <any>
#include <iostream>

int main() {
    std::any value;

    std::cout << std::boolalpha;
    std::cout << value.has_value() << '
'; // false

    value = 100;

    std::cout << value.has_value() << '
'; // true
}
```

An empty `std::any` cannot be safely extracted using the value-returning form of `std::any_cast`.

```cpp
std::any value;

// Throws std::bad_any_cast
int number = std::any_cast<int>(value);
```

---

## Retrieving Values with `std::any_cast`

Use `std::any_cast<T>` to extract the value stored inside a `std::any`.

```cpp
std::any value = 42;

int number = std::any_cast<int>(value);
```

The requested type must match the stored type.

```cpp
std::any value = std::string("Hello");

std::string text = std::any_cast<std::string>(value);
```

You can also retrieve values by reference to avoid unnecessary copies.

```cpp
#include <any>
#include <iostream>
#include <string>

int main() {
    std::any value = std::string("Hello, C++17");

    const std::string& text =
        std::any_cast<const std::string&>(value);

    std::cout << text << '
';
}
```

For a modifiable stored object:

```cpp
std::any value = std::string("hello");

std::string& text = std::any_cast<std::string&>(value);
text += " world";

std::cout << std::any_cast<std::string>(value) << '
';
```

Output:

```text
hello world
```

---

## Handling Invalid Casts

The value-returning and reference-returning forms of `std::any_cast` throw `std::bad_any_cast` when the requested type does not match the stored type.

```cpp
#include <any>
#include <iostream>
#include <string>

int main() {
    std::any value = 42;

    try {
        auto text = std::any_cast<std::string>(value);
        std::cout << text << '
';
    }
    catch (const std::bad_any_cast& exception) {
        std::cerr << "Invalid cast: "
                  << exception.what()
                  << '
';
    }
}
```

Possible output:

```text
Invalid cast: bad any_cast
```

Use exception handling when a type mismatch represents an exceptional situation.

However, when type mismatches are expected during normal program flow, the pointer form of `std::any_cast` is usually cleaner.

---

## Pointer Form of `std::any_cast`

The pointer overloads of `std::any_cast` return a pointer to the contained object when the type matches.

If the type does not match, they return `nullptr` instead of throwing an exception.

```cpp
#include <any>
#include <iostream>
#include <string>

int main() {
    std::any value = std::string("Hello");

    if (auto* text = std::any_cast<std::string>(&value)) {
        std::cout << "String: " << *text << '
';
    }

    if (auto* number = std::any_cast<int>(&value)) {
        std::cout << "Integer: " << *number << '
';
    } else {
        std::cout << "The value does not contain an int.
";
    }
}
```

Output:

```text
String: Hello
The value does not contain an int.
```

This is often the preferred approach:

```cpp
if (const auto* value = std::any_cast<MyType>(&container)) {
    // Use *value safely.
} else {
    // Handle missing or mismatched type.
}
```

The pointer form is especially useful when processing heterogeneous collections.

---

## Inspecting the Stored Type

Use `type()` to inspect the runtime type of the contained value.

```cpp
#include <any>
#include <iostream>
#include <typeinfo>

int main() {
    std::any value = 42;

    if (value.type() == typeid(int)) {
        std::cout << "value contains an int
";
    }

    value = 3.14;

    if (value.type() == typeid(double)) {
        std::cout << "value contains a double
";
    }
}
```

Output:

```text
value contains an int
value contains a double
```

You may print `type().name()` for debugging:

```cpp
std::cout << value.type().name() << '
';
```

However, do not rely on the output of `name()` for application logic.

Its output is implementation-defined and varies across compilers.

For example:

- GCC may print `i` for `int`.
- MSVC may print `int`.
- Complex types may have mangled or implementation-specific names.

Use this instead:

```cpp
if (value.type() == typeid(std::string)) {
    // ...
}
```

---

## Creating Values with `emplace` and `make_any`

### Using `emplace`

`emplace<T>()` destroys the currently stored object, then constructs a new object of type `T` directly inside the `std::any`.

```cpp
#include <any>
#include <iostream>
#include <string>

int main() {
    std::any value;

    value.emplace<std::string>(5, '*');

    std::cout << std::any_cast<std::string>(value) << '
';
}
```

Output:

```text
*****
```

This is useful when an object needs constructor arguments.

```cpp
#include <any>
#include <iostream>

struct Point {
    double x;
    double y;

    Point(double x, double y)
        : x(x), y(y) {}
};

int main() {
    std::any value;

    value.emplace<Point>(10.5, 20.25);

    const auto& point = std::any_cast<const Point&>(value);

    std::cout << "Point(" << point.x << ", " << point.y << ")
";
}
```

### Using `std::make_any`

`std::make_any<T>()` creates and returns a `std::any` containing an object of type `T`.

```cpp
#include <any>
#include <iostream>
#include <string>

int main() {
    auto value = std::make_any<std::string>(4, 'A');

    std::cout << std::any_cast<std::string>(value) << '
';
}
```

Output:

```text
AAAA
```

Example with a custom type:

```cpp
struct User {
    std::string name;
    int age;

    User(std::string name, int age)
        : name(std::move(name)), age(age) {}
};

std::any value = std::make_any<User>("Alice", 30);
```

---

## Resetting and Swapping Values

### `reset()`

Use `reset()` to destroy the contained object and make the `std::any` empty.

```cpp
#include <any>
#include <iostream>

int main() {
    std::any value = 42;

    std::cout << value.has_value() << '
'; // 1

    value.reset();

    std::cout << value.has_value() << '
'; // 0
}
```

### `swap()`

Use `swap()` to exchange the contents of two `std::any` objects.

```cpp
#include <any>
#include <iostream>
#include <string>

int main() {
    std::any first = 42;
    std::any second = std::string("Hello");

    first.swap(second);

    std::cout << std::any_cast<std::string>(first) << '
';
    std::cout << std::any_cast<int>(second) << '
';
}
```

Output:

```text
Hello
42
```

You can also use `std::swap`:

```cpp
std::swap(first, second);
```

---

## Copy and Move Behavior

`std::any` supports copy and move operations.

```cpp
std::any first = std::string("Hello");

std::any second = first;            // Copies the contained string
std::any third = std::move(first);  // Moves the contained value
```

A type stored in `std::any` must be **CopyConstructible**.

This works:

```cpp
std::any value = std::string("Hello");
```

This does not work because `std::unique_ptr` is move-only:

```cpp
#include <any>
#include <memory>

std::any value = std::make_unique<int>(42); // Compilation error
```

If shared ownership is appropriate, `std::shared_ptr` can be stored because it is copyable:

```cpp
#include <any>
#include <memory>

std::any value = std::make_shared<int>(42);
```

Be careful: copying a `std::any` also copies the contained object.

```cpp
std::any first = LargeObject{};
std::any second = first; // May perform an expensive copy
```

For large objects, consider storing a smart pointer:

```cpp
std::any value = std::make_shared<LargeObject>();
```

---

## `std::any` and Dynamic Allocation

The C++ standard allows implementations to allocate memory dynamically when storing an object inside `std::any`.

Many standard library implementations optimize small objects using a technique similar to **Small Object Optimization (SOO)**.

This means that small types may be stored directly inside the `std::any` object without heap allocation.

However, this behavior is not guaranteed by the standard.

```cpp
std::any number = 42;                  // May avoid allocation
std::any text = std::string(1000, 'X'); // May allocate memory
```

Implementations are encouraged to avoid allocations for small objects only when the stored type is nothrow move constructible.

Therefore:

- do not assume `std::any` never allocates;
- avoid using it in extremely performance-sensitive paths unless measured;
- use `std::variant` when the possible types are known in advance.

---

## `std::any` vs. `void*`

Before `std::any`, developers sometimes used `void*` to store arbitrary values.

```cpp
void* raw = new int(42);
```

This approach is unsafe because type information is lost.

```cpp
int* number = static_cast<int*>(raw);

// Incorrect cast: undefined behavior
double* decimal = static_cast<double*>(raw);
```

It also creates ownership and memory-management problems:

```cpp
delete static_cast<int*>(raw);
```

`std::any` is safer because it remembers the actual type and manages the object's lifetime automatically.

```cpp
std::any value = 42;

if (const auto* number = std::any_cast<int>(&value)) {
    std::cout << *number << '
';
}
```

| Feature | `void*` | `std::any` |
|---|---|---|
| Stores arbitrary data | Yes | Yes |
| Preserves type information | No | Yes |
| Automatic lifetime management | No | Yes |
| Type-safe extraction | No | Yes |
| Requires manual `new` / `delete` | Often | No |
| Recommended for general value storage | No | Yes |

---

## `std::any` vs. `std::variant`

Both `std::any` and `std::variant` can store one value at a time, but they solve different problems.

`std::any` can store almost any copyable type:

```cpp
std::any value = 42;
value = std::string("Hello");
value = 3.14;
```

`std::variant` can only store one type from a predefined list:

```cpp
#include <variant>
#include <string>

std::variant<int, double, std::string> value = 42;

value = std::string("Hello");
```

| Feature | `std::any` | `std::variant` |
|---|---|---|
| Available since | C++17 | C++17 |
| Allowed types | Any copyable type | Predefined list of types |
| Type knowledge | Runtime | Compile-time type list |
| Value access | `std::any_cast<T>` | `std::get<T>` / `std::visit` |
| Wrong type handling | `std::bad_any_cast` or `nullptr` | `std::bad_variant_access` |
| Heap allocation | Possible | Usually unnecessary |
| Best use case | Open-ended types | Known set of alternatives |

Use `std::variant` when the possible types are known:

```cpp
using ConfigValue = std::variant<int, bool, std::string>;
```

Use `std::any` when your program must support arbitrary or extensible types:

```cpp
std::unordered_map<std::string, std::any> plugin_data;
```

---

## When to Use `std::any`

Use `std::any` when:

1. The type of the value is not known at compile time.
2. The set of possible types is open-ended.
3. You are implementing a plugin or extension system.
4. You need to attach arbitrary metadata to objects.
5. You are building a generic property bag.
6. You need a safer alternative to `void*`.

Example: application metadata.

```cpp
#include <any>
#include <string>
#include <unordered_map>

std::unordered_map<std::string, std::any> metadata;

metadata["service_name"] = std::string("Exposure Monitor");
metadata["port"] = 8080;
metadata["debug"] = true;
metadata["timeout_seconds"] = 15.5;
```

Reading a value safely:

```cpp
if (const auto* port = std::any_cast<int>(&metadata["port"])) {
    std::cout << "Port: " << *port << '
';
}
```

Do not use `std::any` merely to avoid designing proper types.

If your data structure has a fixed schema, a `struct` is usually better:

```cpp
struct ServerConfig {
    std::string service_name;
    int port;
    bool debug;
    double timeout_seconds;
};
```

---

## Common Errors

### 1. Casting to the Wrong Type

```cpp
std::any value = 42;

auto number = std::any_cast<double>(value); // Throws std::bad_any_cast
```

Correct:

```cpp
auto number = std::any_cast<int>(value);
```

---

### 2. Assuming Implicit Conversion Happens

```cpp
std::any value = 42;

// int is not automatically converted to double.
double number = std::any_cast<double>(value); // Invalid
```

Correct:

```cpp
int integer = std::any_cast<int>(value);
double number = static_cast<double>(integer);
```

---

### 3. Forgetting That `std::any` May Be Empty

```cpp
std::any value;

// Throws std::bad_any_cast.
auto number = std::any_cast<int>(value);
```

Safer:

```cpp
if (value.has_value()) {
    // The object is not empty.
}
```

Or use the pointer form:

```cpp
if (auto* number = std::any_cast<int>(&value)) {
    std::cout << *number << '
';
}
```

---

### 4. Making Unnecessary Copies

```cpp
std::any value = std::string("Large text");

// Copies the stored string.
std::string text = std::any_cast<std::string>(value);
```

When you only need to read the value, use a const reference:

```cpp
const std::string& text =
    std::any_cast<const std::string&>(value);
```

---

### 5. Passing `std::any` by Value

```cpp
void print(std::any value) {
    // Copies the std::any and potentially its contained object.
}
```

Prefer a reference when copying is unnecessary:

```cpp
void print(const std::any& value) {
    // No copy of std::any is made.
}
```

---

### 6. Trying to Store Move-Only Types

```cpp
std::any value = std::make_unique<int>(10); // Compilation error
```

`std::any` requires copyable stored objects.

Use `std::shared_ptr`, a wrapper type with copy semantics, or redesign the API if ownership transfer is required.

---

### 7. Using `type().name()` for Program Logic

```cpp
if (value.type().name() == "int") {
    // Incorrect and non-portable.
}
```

Correct:

```cpp
if (value.type() == typeid(int)) {
    // Portable type comparison.
}
```

---

## Complete Example

```cpp
#include <any>
#include <iostream>
#include <string>
#include <unordered_map>

struct Point {
    double x;
    double y;
};

std::ostream& operator<<(std::ostream& os, const Point& point) {
    return os << "Point(" << point.x << ", " << point.y << ")";
}

template <typename T>
void print_if_type(const std::any& value, const std::string& label) {
    if (const auto* item = std::any_cast<T>(&value)) {
        std::cout << label << ": " << *item << '
';
    } else {
        std::cout << label << ": type mismatch
";
    }
}

int main() {
    std::cout << std::boolalpha;

    std::any value;

    std::cout << "Initially has value: "
              << value.has_value()
              << "

";

    value = 42;
    print_if_type<int>(value, "Integer");

    value = std::string("Hello, std::any");
    print_if_type<std::string>(value, "Text");

    value.emplace<Point>(Point{10.5, 20.25});
    print_if_type<Point>(value, "Point");

    std::cout << "
Stored type matches Point: "
              << (value.type() == typeid(Point))
              << '
';

    if (const auto* point = std::any_cast<Point>(&value)) {
        std::cout << "Coordinates: x = "
                  << point->x
                  << ", y = "
                  << point->y
                  << '
';
    }

    std::cout << "
Testing an invalid cast:
";

    if (const auto* number = std::any_cast<int>(&value)) {
        std::cout << *number << '
';
    } else {
        std::cout << "The value does not contain an int.
";
    }

    std::unordered_map<std::string, std::any> config;

    config["host"] = std::string("127.0.0.1");
    config["port"] = 8080;
    config["debug"] = true;

    std::cout << "
Configuration:
";

    if (const auto* host = std::any_cast<std::string>(&config["host"])) {
        std::cout << "Host: " << *host << '
';
    }

    if (const auto* port = std::any_cast<int>(&config["port"])) {
        std::cout << "Port: " << *port << '
';
    }

    if (const auto* debug = std::any_cast<bool>(&config["debug"])) {
        std::cout << "Debug: " << *debug << '
';
    }

    value.reset();

    std::cout << "
After reset, has value: "
              << value.has_value()
              << '
';

    return 0;
}
```

Possible output:

```text
Initially has value: false

Integer: 42
Text: Hello, std::any
Point: Point(10.5, 20.25)

Stored type matches Point: true
Coordinates: x = 10.5, y = 20.25

Testing an invalid cast:
The value does not contain an int.

Configuration:
Host: 127.0.0.1
Port: 8080
Debug: true

After reset, has value: false
```

---

## Exercises

1. Create a `std::unordered_map<std::string, std::any>` named `settings` that stores:
   - a server hostname as `std::string`;
   - a port as `int`;
   - TLS status as `bool`;
   - a timeout as `double`.

2. Write a template function named `try_get<T>()` that accepts a `const std::any&` and returns a pointer to the contained value, or `nullptr` when the type does not match.

3. Create a class named `Event` with:
   - a `std::string name`;
   - a `std::any payload`.

   Store different payload types such as `int`, `std::string`, and a custom `Point` structure.

4. Compare `std::any` and `std::variant<int, double, std::string>` for a configuration system with a fixed schema. Explain which option is more appropriate and why.

5. Store a large custom object in `std::any`, then measure the difference between:
   - extracting it by value;
   - extracting it as `const T&`;
   - storing it through `std::shared_ptr<T>`.

---

## Conclusion

`std::any` is a flexible, type-safe C++17 utility for storing one value of an arbitrary copyable type.

Its key features are:

- it can store values of different types over its lifetime;
- it preserves runtime type information;
- it provides safe access through `std::any_cast`;
- it supports empty states through `has_value()` and `reset()`;
- it is safer and easier to manage than `void*`;
- it is best suited to open-ended and extensible type systems.

Use the pointer form of `std::any_cast` when a type mismatch is expected:

```cpp
if (const auto* value = std::any_cast<MyType>(&data)) {
    // Safely use *value.
}
```

If the possible types are known in advance, prefer `std::variant` for stronger compile-time guarantees and usually more predictable performance.

---

## 🤝 Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |  |  |  |
