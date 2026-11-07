<div align="center">

[🇺🇸 English](./disjunction.md) | [🇮🇷 فارسی](../../fa/cpp17/disjunction.md)

</div>

---

#  std::disjunction 

`std::disjunction` is a class template in the C++ Standard Library, introduced in C++17. It belongs to the Type Traits library and is used to perform a logical OR operation on multiple type traits at compile time.

**In simple terms:**

- If at least one condition is `true`, the result is `true`.
- The result is `false` only when all conditions are `false`.

##  Introduction to std::disjunction

First, include the following header:

```cpp
#include <type_traits>
```

**General syntax:**

```cpp
std::disjunction<Trait1, Trait2, Trait3, ...>
```

Each argument must be a type that provides a compile-time Boolean value, such as:

- `std::true_type`
- `std::false_type`
- `std::is_integral<T>`
- `std::is_floating_point<T>`

##  A Simple Example

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::disjunction<
        std::is_integral<int>,
        std::is_floating_point<int>
    >;

    std::cout << Result::value << '\n';

}
```

**Output:**

```txt
1
```

**Explanation:**

- `std::is_integral<int>` returns `true`.
- `std::is_floating_point<int>` returns `false`.
- `std::disjunction` combines these results using logical OR.

Therefore:

```txt
true || false == true
```

The final result is `true`.

##  An Example with Multiple False Conditions

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::disjunction<
        std::is_floating_point<int>,
        std::is_array<int>,
        std::is_pointer<int>
    >;

    std::cout << Result::value << '\n';

}
```

**Output:**

```
0
```

**Explanation:**

- `std::is_floating_point<int>` returns `false`.
- `std::is_array<int>` returns `false`.
- `std::is_pointer<int>` returns `false`.

Therefore:

```txt
false || false || false == false
```

The final result is `false`.

##  Using Multiple Conditions

One important feature of `std::disjunction` is that we can combine multiple type traits.

For example, suppose we want to check whether a type is an integral type, floating-point type, or pointer type.

```cpp
#include <iostream>
#include <type_traits>
#include <string>

template<typename T>
void check() {

    using Result = std::disjunction<
        std::is_integral<T>,
        std::is_floating_point<T>,
        std::is_pointer<T>
    >;

    if constexpr (Result::value) {

        std::cout << "Supported type\n";

    } else {

        std::cout << "Unsupported type\n";

    }
}

int main() {

    check<int>();
    check<double>();
    check<int*>();
    check<std::string>();

}
```

**Output:**

```cpp
Supported type
Supported type
Supported type
Unsupported type
```

**Explanation:**

In this example, at least one of the following conditions must be true:

1. Is the type integral?
2. Is the type floating-point?
3. Is the type a pointer?

If any one of these conditions is true, `std::disjunction` returns `true`.

##  Using std::disjunction with Templates

Now let's look at a more practical example.

Suppose we want to create a template that accepts only integral and floating-point types.

```cpp
#include <iostream>
#include <type_traits>
#include <string>

template<typename T>
void process(T value) {

    static_assert(
        std::disjunction<
            std::is_integral<T>,
            std::is_floating_point<T>
        >::value,
        "T must be an arithmetic type"
    );

    std::cout << value << '\n';
}

int main() {

    process<int>(100);
    process<double>(10.5);

    // process<std::string>("Hello");
    // Compilation error

}
```

**Explanation:**

We have two conditions:

- `std::is_integral<T>`
- `std::is_floating_point<T>`

Only one of these conditions needs to be true for the `static_assert` to pass.

| Type          | Integral | Floating Point | Result |
| ------------- | -------- | -------------- | ------ |
| `int`         | true     | false          | true   |
| `double`      | false    | true           | true   |
| `float`       | false    | true           | true   |
| `std::string` | false    | false          | false  |

##  Shorter Syntax: std::disjunction_v

C++17 also provides a shorter version called `std::disjunction_v`.

Instead of writing:

```cpp
std::disjunction<
    std::is_integral<int>,
    std::is_floating_point<int>
>::value
```

We can simply write:

```cpp
std::disjunction_v<
    std::is_integral<int>,
    std::is_floating_point<int>
>
```

Complete example:

```cpp
#include <iostream>
#include <type_traits>

int main() {

    constexpr bool result =
        std::disjunction_v<
            std::is_integral<int>,
            std::is_floating_point<int>
        >;

    std::cout << result << '\n';

}
```

**Output:**

```
1
```

##  An Important Feature: Short-Circuit Evaluation

One important feature of `std::disjunction` is its compile-time short-circuit behavior.

This means that if the first condition is `true`, the remaining type traits do not need to be instantiated to determine the result.

Example:

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
struct Check {

    static_assert(sizeof(T) == 0,
                  "Check was instantiated!");

    static constexpr bool value = false;
};

using Result = std::disjunction<
    std::true_type,
    Check<int>
>;

int main() {

    std::cout << Result::value;

}
```

**Output:**

```txt
1
```

**Why don't we get a compilation error?**

Because the first argument is `std::true_type`.

Therefore, `std::disjunction` does not need to instantiate `Check<int>` to determine the result.

Important

This feature is particularly useful in:

- Template Programming
- SFINAE
- Type Validation
- Compile-Time Programming

##  Difference Between conjunction, disjunction, and negation

Now that we have studied all three utilities, let's compare them.

| Type Trait         | Logical Operator | Meaning                             |
| ------------------ | ---------------- | ----------------------------------- |
| `std::conjunction` | AND (`&&`)       | All conditions must be true         |
| `std::disjunction` | OR (`\\|\\|`)    | At least one condition must be true |
| `std::negation`    | NOT (`!`)        | Reverses a Boolean result           |

### Mathematical representation:

**Conjunction:**

\\[ A \land B \\]

Disjunction:

\\[ A \lor B \\]

Negation:

\\[ \neg A \\]

You have now learned three important compile-time logical utilities in C++17:

1. `std::conjunction` — AND
2. `std::disjunction` — OR
3. `std::negation` — NOT



---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>
