<div align="center">

[🇺🇸 English](./structured_bindings_improvements.md) | [🇮🇷 فارسی](../../fa/cpp20/structured_bindings_improvements.md)

</div>

---

# Structured Bindings Improvements in C++20

## Introduction

Structured Bindings are an important feature introduced in C++17. They make it possible to extract multiple components from an object and bind them to individual names. This feature makes working with `std::pair`, `std::tuple`, arrays, and simple data structures more readable and natural.

Despite its widespread usefulness, the initial version had several limitations. For example, C++17 did not allow the names introduced by Structured Bindings to be captured directly in a `lambda`. It also did not support certain storage-duration specifiers and related declaration features.

C++20 extended this functionality, bringing Structured Bindings closer to ordinary variable declarations.

The most important improvements include:

- Direct capture of Structured Bindings in lambda expressions.
- Support for both value and reference capture.
- Support for `static`.
- Support for `thread_local`.
- Improvements related to attributes and declaration rules.
- Changes to class-member accessibility rules through related standardization proposals.

To understand these changes correctly, we first need to examine how Structured Bindings work internally and what limitations existed in their initial version.

## Understanding the Fundamentals of Structured Bindings

### The Concept of Structured Bindings

In C++17, Structured Bindings allow the components of an object to be bound to separate names.

For example, the following function returns a `std::pair`:

```
#include <iostream>
#include <string>
#include <utility>

std::pair<int, std::string> getUser() {
    return {42, "Alice"};
}

int main() {
    auto [id, name] = getUser();

    std::cout << id << '\n';
    std::cout << name << '\n';
}
```

In this example, the names `id` and `name` are bound to the two components of the returned object.

Without Structured Bindings, we would have to access the `first` and `second` members:

```
auto user = getUser();

auto id = user.first;
auto name = user.second;
```

Structured Bindings make the code shorter and more readable, particularly when a function returns multiple related values.

### Internal Mechanism and the Backing Variable

Despite their simple appearance, Structured Bindings are not merely equivalent to declaring several independent variables.

The standard specifies a backing variable for each Structured Binding declaration. This variable holds the original object, while the introduced names are bound to its components.

For example:

```
auto [x, y] = std::pair{10, 20};
```

Conceptually, its behavior can be illustrated as follows:

```
auto backing = std::pair{10, 20};

auto& x = backing.first;
auto& y = backing.second;
```

This code is only a conceptual illustration; it is not an exact representation of the standard's rules. The actual types of the bindings and how they refer to the components depend on the data type and the rules governing Structured Bindings.

The backing variable is also not an implementation detail directly exposed in user code. Rather, it is part of the semantic model defined by the language standard.

### Types That Can Be Used with Structured Bindings

C++ supports three main categories of types for Structured Bindings.

#### Arrays

Arrays with a known number of elements can be decomposed directly:

```
int values[] = {10, 20, 30};

auto [first, second, third] = values;
```

In this case, the number of names must exactly match the number of array elements.

#### Tuple-like Types

Types such as the following support Structured Bindings:

- `std::pair`
- `std::tuple`
- `std::array`

These types use the tuple-like protocol, which includes `std::tuple_size` and `std::tuple_element`.

```
#include <array>

int main() {
    std::array<int, 3> values{10, 20, 30};

    auto [first, second, third] = values;
}
```

#### Class Types

Classes whose non-static data members satisfy the rules for Structured Bindings can also be decomposed:

```
struct Point {
    int x;
    int y;
};

int main() {
    Point point{10, 20};

    auto [x, y] = point;
}
```

In this case, the data members must satisfy the standard's requirements for class decomposition, including restrictions related to class structure, inheritance, and access control.

### Differences Between `auto`, `auto&`, and `auto&&`

Choosing the declaration form is important when using Structured Bindings.

```
#include <string>

struct User {
    int age;
    std::string name;
};

int main() {
    User user{25, "Alice"};

    auto [age1, name1] = user;
    auto& [age2, name2] = user;
    auto&& [age3, name3] = user;
}
```

The behavior of these declarations can be summarized as follows:

| Declaration     | Behavior                                                                       |
| --------------- | ------------------------------------------------------------------------------ |
| `auto [x, y]`   | Creates an independent backing variable initialized from the initializer.      |
| `auto& [x, y]`  | Binds the backing variable to the original lvalue.                             |
| `auto&& [x, y]` | Binds the backing variable according to the value category of the initializer. |

In the example above, changing `age1` does not affect the original object because an independent backing variable is created.

By contrast, changing `age2` or `age3` directly modifies the corresponding member of `user`.

An important point is that `auto [x, y]` does not necessarily create two independent copies of the members. There is only one backing variable, and the bindings refer to its components.

### Differences Between the Binding Type and the Expression Type

One important detail of Structured Bindings concerns the behavior of `decltype`.

The type reported by `decltype` for a binding is determined according to the special rules for Structured Bindings. It should not be confused with the apparent internal reference type.

```
#include <tuple>
#include <type_traits>

int main() {
    int value = 10;

    std::tuple<int, int&> data{20, value};

    auto [first, second] = data;

    static_assert(std::is_same_v<decltype(first), int>);
    static_assert(std::is_same_v<decltype(second), int&>);

    second = 50;
}
```

In this example, `second` remains bound to the original variable `value` because the second tuple element has type `int&`.

This demonstrates that the behavior of Structured Bindings is not determined solely by the apparent declaration type.

## Major Improvements in C++20

### Direct Capture in Lambdas

One of the most significant limitations of C++17 was that names introduced by Structured Bindings could not be captured directly in lambda expressions.

For example, the following code was not valid in C++17:

```
auto [x, y] = std::pair{10, 20};

auto sum = [x, y] {
    return x + y;
};
```

C++20 removed this restriction.

Structured Binding names can now be placed directly in a lambda's capture list:

```
#include <iostream>
#include <utility>

int main() {
    auto [x, y] = std::pair{10, 20};

    auto sum = [x, y] {
        return x + y;
    };

    std::cout << sum() << '\n';
}
```

This change eliminates the need to declare temporary variables or use init-capture just to capture the components of an object.

### Capture by Value

With value capture, the values of the bindings are stored in the closure object:

```
#include <iostream>
#include <utility>

int main() {
    auto [x, y] = std::pair{10, 20};

    auto callback = [x, y] {
        return x + y;
    };

    x = 100;
    y = 200;

    std::cout << callback() << '\n';
}
```

Output:

```
30
```

In this example, the lambda retains the original values of `x` and `y`.

Changes to the original variables after the lambda is created do not affect the values stored in the closure object.

This behavior is useful for callbacks that should remain independent of subsequent changes to local state.

### Capture by Reference

With reference capture, the lambda is bound to the existing bindings:

```
#include <iostream>
#include <utility>

int main() {
    auto [x, y] = std::pair{10, 20};

    auto callback = [&x, &y] {
        x += 5;
        y += 5;
    };

    callback();

    std::cout << x << ' ' << y << '\n';
}
```

Output:

```
15 25
```

In this case, changes made inside the lambda are applied directly to the components of the backing object.

This approach is appropriate when a lambda needs to modify existing state and its lifetime does not exceed the lifetime of the referenced objects.

### Implicit Capture with a Default Capture

As with ordinary variables, Structured Bindings can also be captured implicitly by using a capture default.

```
#include <iostream>
#include <utility>

int main() {
    auto [width, height] = std::pair{640, 480};

    auto calculateArea = [=] {
        return width * height;
    };

    auto scale = [&] {
        width *= 2;
        height *= 2;
    };

    scale();

    std::cout << calculateArea() << '\n';
}
```

In the first lambda, `[=]` captures the used bindings by value.

In the second lambda, `[&]` captures them by reference.

An important detail is that implicit capture applies only to bindings that are used in the lambda body.

### Direct Capture vs. Init-capture

Init-capture has been available since C++14.

Therefore, even before C++20, we could transfer extracted values into a lambda:

```
#include <utility>

int main() {
    auto [x, y] = std::pair{10, 20};

    auto callback = [sum = x + y] {
        return sum;
    };
}
```

However, this approach differs from direct capture.

In C++20, we can write:

```
auto callback = [x, y] {
    return x + y;
};
```

The advantage of direct capture is that it eliminates additional names and simplifies the code.

Nevertheless, init-capture remains useful in the following situations:

- Renaming variables during capture.
- Computing a new value when creating the lambda.
- Transferring ownership of resources.
- Using `std::move`.
- Creating independent state with a different type or value.

For example:

```
#include <memory>

int main() {
    auto resource = std::make_unique<int>(42);

    auto callback = [ptr = std::move(resource)] {
        return *ptr;
    };
}
```

In this example, init-capture transfers ownership of the resource to the closure object.

### Limitations of Reference Capture

Reference capture is not always an appropriate choice.

If a lambda is used after the original object has reached the end of its lifetime, a dangling reference may occur.

```
#include <functional>
#include <utility>

int main() {
    std::function<int()> callback;

    {
        auto [x, y] = std::pair{10, 20};

        callback = [&x, &y] {
            return x + y;
        };
    }

    // Calling callback here causes undefined behavior.
}
```

After leaving the inner scope, the backing variable is destroyed, and the references held by the closure are no longer valid.

Therefore, if a lambda is going to be stored or used beyond the current scope, value capture or init-capture with ownership transfer is generally a safer choice.

### Capture Limitations for Bit-fields

An important edge case involves bit-field members.

Structured Bindings can decompose bit-field members, but a bit-field cannot be used with reference capture in a lambda.

```
struct Flags {
    unsigned int enabled : 1;
    unsigned int visible : 1;
};

int main() {
    Flags flags{1, 0};

    auto [enabled, visible] = flags;

    auto callback = [enabled] {
        return enabled;
    };
}
```

In this example, value capture is permitted.

However, directly capturing the binding corresponding to a bit-field by reference is not permitted.

In such situations, if the value needs to be used in a callback, value capture or an independent variable can be used instead.

## Managing Lifetime with Storage-Class Specifiers

### Using `static`

One of the important improvements in C++20 is the ability to use `static` with Structured Bindings.

```
#include <string>
#include <utility>

void updateUser() {
    static auto [id, name] =
        std::pair{42, std::string{"Alice"}};

    ++id;
    name += " Smith";
}
```

In this example, the backing variable has static storage duration.

As a result:

- Initialization is performed only once.
- Values are preserved between function calls.
- The bindings remain valid throughout the lifetime of the backing variable.
- All function calls access the same state.

This feature is suitable for maintaining small, multi-component states.

An important point is that `static` applies to the declaration and its backing object. It does not define each binding as an independent variable with a separate storage duration.

### Using `thread_local`

C++20 also permits the use of `thread_local` with Structured Bindings.

```
#include <utility>

void processRequest() {
    thread_local auto [requestCount, errorCount] =
        std::pair{0, 0};

    ++requestCount;

    if (requestCount % 10 == 0) {
        ++errorCount;
    }
}
```

In this example, each thread has its own independent instance of the backing variable.

This feature is useful for:

- Per-thread counters.
- Thread-local caches.
- Request-processing statistics.
- Maintaining independent state in worker threads.

However, `thread_local` is not a replacement for synchronization.

If data must be shared between multiple threads, using a `mutex`, atomic operations, or an appropriate concurrency mechanism is still necessary.

### Combining `static` and `const`

`static` can be combined with `const`:

```
#include <string>
#include <utility>

void getConfiguration() {
    static const auto [port, protocol] =
        std::pair{8080, std::string{"https"}};
}
```

In this example, the backing variable is const, so the bindings cannot directly modify its ordinary members.

This pattern can improve readability when working with fixed, multi-component configuration values.

### Using Attributes

Structured Bindings support attributes permitted at the declaration.

One useful application is `[[maybe_unused]]`:

```
#include <utility>

int main() {
    [[maybe_unused]] auto [width, height] =
        std::pair{640, 480};
}
```

This attribute is useful when a declaration is logically necessary, but some or all of its names are not subsequently used in the program.

However, the placement of the attribute and its scope of effect must be considered according to the standard's rules.

## Changes Related to Language Rules

### Access to Private Class Members

One change related to Structured Bindings was introduced through the standardization proposal P0969R0 — Allow structured bindings to accessible members.

The original rules imposed a general public-member restriction on class decomposition.

The change replaced this restriction with the concept of member accessibility in the context of the declaration.

For example:

```
class Point {
private:
    int x;
    int y;

public:
    Point(int xValue, int yValue)
        : x{xValue}, y{yValue} {}

    int sum() const {
        auto [first, second] = *this;

        return first + second;
    }
};
```

In this example, the class member function has access to the private members, so the Structured Binding is valid in that context.

However, the following code remains invalid:

```
class Point {
private:
    int x;
    int y;

public:
    Point(int xValue, int yValue)
        : x{xValue}, y{yValue} {}
};

int main() {
    Point point{10, 20};

    // Private members are inaccessible here.
    // auto [x, y] = point;
}
```

This change does not mean that encapsulation has been removed.

An important standardization detail is that P0969R0 is also treated as a defect resolution for the Structured Bindings rules. It should not be considered a feature exclusive to C++20 mode.

### The Tuple-like Protocol

When decomposing tuple-like types, the standard mechanism consists of three main components:

- `std::tuple_size`: Specifies the number of decomposable elements.
- `std::tuple_element`: Determines the type of each element.
- `get`: Provides each element's value to the Structured Binding.

For example:

```
#include <iostream>
#include <tuple>

int main() {
    std::tuple<int, double, char> data{
        10,
        2.5,
        'A'
    };

    auto [count, ratio, marker] = data;

    std::cout << count << ' '
              << ratio << ' '
              << marker << '\n';
}
```

In this process, the lookup rules for the `get` function are important.

If qualified lookup finds a member named `get`, that member function is used. Otherwise, unqualified lookup is performed using ADL.

When designing a custom tuple-like type, these rules and the requirements for `std::tuple_size` and `std::tuple_element` must be followed carefully.

Specializations can also be added to the `std` namespace only in cases permitted by the standard. For user-defined types, the permitted specializations of `tuple_size` and `tuple_element` are applicable.

## Practical Applications in Software Development

### Decomposing Function Return Values

One of the most common uses of Structured Bindings is receiving multiple values from a function.

```
#include <string>
#include <utility>

std::pair<bool, std::string> validateInput(int value) {
    if (value >= 0) {
        return {true, "valid"};
    }

    return {false, "negative value"};
}

int main() {
    auto [success, message] = validateInput(10);

    if (!success) {
        // Handle validation failure.
    }
}
```

This approach is suitable for APIs that return an operation's status and result.

However, if there are many return values or each value has a complex meaning, defining a dedicated type with named members may provide a better design.

### Using Structured Bindings with Containers

One of the most common applications of Structured Bindings is iterating over a `std::map`.

```
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<int, std::string> users{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"}
    };

    for (const auto& [id, name] : users) {
        std::cout << id << ": " << name << '\n';
    }
}
```

In this example:

- `id` is bound to the map key.
- `name` is bound to the corresponding value.
- `const auto&` prevents unnecessary copying of the pair.
- The keys of `std::map` cannot be modified because of the container's properties.

If the map values need to be modified, `auto&` can be used:

```
for (auto& [id, name] : users) {
    name += " Updated";
}
```

In this case, changes to the values are applied directly to the container.

### Using Structured Bindings with Standard Algorithms

Structured Bindings can improve readability when combined with standard algorithms.

```
#include <algorithm>
#include <utility>
#include <vector>

int main() {
    std::vector<std::pair<int, int>> points{
        {1, 2},
        {3, 4},
        {5, 6}
    };

    auto [minX, maxX] = std::pair{2, 4};

    auto count = std::count_if(
        points.begin(),
        points.end(),
        [minX, maxX](const auto& point) {
            return point.first >= minX &&
                   point.first <= maxX;
        }
    );
}
```

In this example, directly capturing the binding names simplifies the lambda.

### Using Structured Bindings with Callbacks

When designing callbacks, direct capture can reduce the need for intermediate variables.

```
#include <functional>
#include <utility>

int main() {
    auto [minimum, maximum] = std::pair{10, 100};

    std::function<bool(int)> isValid =
        [minimum, maximum](int value) {
            return value >= minimum &&
                   value <= maximum;
        };
}
```

In this example, the lambda stores a numeric range in its closure object.

Because the values are captured by value, the callback does not depend on the lifetime of the local variables.

### Using Structured Bindings for State Management

Combining `static` with Structured Bindings is useful for maintaining multi-component state.

```
#include <string>
#include <utility>

std::string nextLabel() {
    static auto [index, prefix] =
        std::pair{0, std::string{"item"}};

    ++index;

    return prefix + "-" + std::to_string(index);
}
```

In this example, the counter is preserved between function calls.

However, if multiple threads call this function concurrently, subsequent modifications to the shared state require synchronization.

## Limitations and Important Considerations

### No `constexpr` Support in C++20

One important limitation of C++20 is that a Structured Binding declaration cannot be declared directly with `constexpr`.

For example:

```
// Not valid in C++20.
constexpr auto [x, y] = std::pair{10, 20};
```

Support for `constexpr` Structured Bindings has been addressed and accepted in subsequent standardization work, including C++26.

Therefore, features introduced in later standards should not be attributed to C++20.

### Limitations of `inline` and `extern`

The addition of `static` and `thread_local` does not mean that all storage-class specifiers are permitted.

Specifically, `inline` and `extern` should not be considered permitted specifiers for Structured Binding declarations in C++20.

### Limitations on Constraints

In C++20, constrained placeholders cannot be used directly to constrain a Structured Binding.

For example, the following syntax is not valid:

```
// Not valid C++20 syntax.
template <typename T>
concept PairLike = requires(T value) {
    value.first;
    value.second;
};

// Constrained structured binding syntax is not supported.
```

The appropriate approach is to apply the constraint to the template or function parameter, and then use Structured Binding inside the function body:

```
template <typename T>
requires requires(T value) {
    value.first;
    value.second;
}
auto sumMembers(const T& value) {
    auto [first, second] = value;

    return first + second;
}
```

This example is suitable for classes that have the required members and also satisfy the language rules for decomposition.

### The Requirement to Match the Number of Elements

The number of names in a Structured Binding must match the number of decomposable elements.

It is not possible to arbitrarily omit some names.

For example:

```
#include <utility>

int main() {
    auto [value, unused] = std::pair{42, 100};

    (void)unused;
}
```

In this example, both elements must be named, even if only one of them is needed.

### Copy and Move Costs

Using `auto` may cause the initializer object to be copied or moved into the backing variable.

Therefore, when working with large objects or types with expensive copy operations, it is important to consider the choice of reference qualifier.

For container iteration, the following pattern is usually appropriate:

```
for (const auto& [key, value] : container) {
    // Read-only processing.
}
```

To modify elements, you can write:

```
for (auto& [key, value] : container) {
    // Modify elements when permitted.
}
```

By contrast, using `auto` without a reference in a range-based `for` loop may create a copy of the element on each iteration.

### Lifetime Considerations

In declarations that use references, the lifetime of the original object is important.

Similarly, if bindings are captured by reference in a lambda, the closure must not outlive the referenced objects.

When designing callbacks that will be stored, ownership should be clearly defined. Value capture or init-capture should be used when necessary.

### Choosing Appropriate Names

Structured Bindings are most useful when the names of the components have clear meanings.

For example:

```
auto [width, height] = dimensions;
```

is more readable than using generic names:

```
auto [a, b] = dimensions;
```

If the data structure is complex or contains many elements, using a dedicated type with named members may be a better choice.

### Limitations Related to Inherited Classes

When directly decomposing class members, the language imposes restrictions related to inheritance and data members.

Specifically, a decomposable class must satisfy the structural requirements defined by the standard. Inherited data members or an unsuitable combination of base-class and derived-class members may prevent Structured Binding.

For complex classes, using the tuple-like protocol or explicit accessors usually provides a more controllable design.

## Comparison Across Standard Versions

| Feature                                     | Standard Version                   |
| ------------------------------------------- | ---------------------------------- |
| Introduction of Structured Bindings         | C++17                              |
| Array decomposition                         | C++17                              |
| Tuple-like type decomposition               | C++17                              |
| Class data-member decomposition             | C++17                              |
| Direct capture in lambdas                   | C++20                              |
| Value and reference capture                 | C++20                              |
| Use of `static`                             | C++20                              |
| Use of `thread_local`                       | C++20                              |
| Changes to class-member accessibility rules | Defect resolution related to C++17 |
| `constexpr` support                         | C++26                              |
| Structured Binding Packs                    | C++26                              |

An important point is that some standard corrections are also applied to earlier standard versions as defect resolutions.

Therefore, differences in compiler behavior do not always depend solely on the selected language-standard option. The compiler version and the extent to which standard corrections have been implemented are also important.

## Compiler Support

Support for C++20 features depends on the compiler version.

In particular, the following features may not be fully implemented in older compilers:

- Direct capture of Structured Bindings in lambdas.
- Use of `static` in declarations.
- Use of `thread_local`.
- Certain changes related to accessibility rules.

The following commands can be used to test the examples:

```
g++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o app
```

```
clang++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o app
```

Note that selecting `-std=c++20` only specifies the requested language standard. It does not guarantee that every feature is fully implemented in every compiler version.

For professional projects, checking compiler documentation and running compatibility tests are important.

## Common Mistakes When Using Structured Bindings

- Assuming bindings are independent: Names introduced by decomposition are bound to components of the backing object. They should not always be treated as independent variables.
- Ignoring copy costs: Using `auto` may increase the cost of creating the backing object.
- Incorrect use of reference capture: The closure must not outlive the referenced object.
- Assuming `constexpr` support in C++20: This capability belongs to later standard versions.
- Assuming unrestricted access to private members: Changes to accessibility rules do not remove encapsulation.
- Reference capture of bit-fields: Such capture is not permitted.
- Using `static` as a synchronization mechanism: `static` manages lifetime, not thread-safe concurrent access.
- Using ambiguous names: Names such as `a` and `b` may reduce readability in complex data structures.

## Conclusion

Structured Bindings were introduced in C++17 as a way to decompose objects directly, and C++20 added several important capabilities.

The most significant C++20 improvements include:

- Direct capture of bindings in lambdas.
- Support for value and reference capture.
- Support for `static`.
- Support for `thread_local`.
- Improvements that make this feature more useful when designing classes and APIs.

These capabilities make Structured Bindings more practical in modern software development, container processing, callback design, and state management.

However, using them effectively requires attention to details such as binding types, the differences between `auto` and references, object lifetime, copy costs, bit-field restrictions, and the target language-standard version.

Ultimately, Structured Bindings provide the greatest value when naming components improves code readability and clarity—not merely when it reduces the number of lines in a program.

## References

- [P1091R3 — Extending structured bindings to be more like variable declarations](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1091r3.html?utm_source=chatgpt.com)
- [P1381R1 — References to bindings](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1381r1.html?utm_source=chatgpt.com)
- [P0969R0 — Allow structured bindings to accessible members](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0969r0.pdf?utm_source=chatgpt.com)
- [cppreference — Structured binding declaration](https://en.cppreference.com/w/cpp/language/structured_binding?utm_source=chatgpt.com)
- [cppreference — Lambda expressions](https://en.cppreference.com/w/cpp/language/lambda?utm_source=chatgpt.com)

## 🤝 Contributions

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>