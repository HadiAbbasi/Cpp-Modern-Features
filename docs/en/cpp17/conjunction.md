
<div align="center">

[🇺🇸 English](./conjunction.md) | [🇮🇷 فارسی](../../fa/cpp17/conjunction.md)

</div>

---

# std::conjunction 

`std::conjunction` is a class template in the C++ Standard Library, introduced in C++17. It belongs to the Type Traits library and is used to perform a logical AND operation on multiple type traits at compile time.

**In simple terms:**

- If all conditions are `true`, the result is `true`.
- If at least one condition is `false`, the result is `false`.

##  Introduction to std::conjunction

First, include the `<type_traits>` header:

```cpp
#include <type_traits>
```

**General syntax:**

```cpp
std::conjunction<Trait1, Trait2, Trait3, ...>
```

Each argument must be a type that provides a compile-time Boolean value, such as `std::true_type`, `std::false_type`, or another type trait.

##  A Simple Example

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::conjunction<
        std::is_integral<int>,
        std::is_signed<int>
    >;

    std::cout << Result::value << '\n';

}
```

**Output:**

```txt
1
```

**Explanation:**

- `std::is_integral<int>` checks whether `int` is an integral type. Result: `true`.
- `std::is_signed<int>` checks whether `int` is a signed type. Result: `true`.
- `std::conjunction` performs a logical AND operation.

**Therefore:**

```txt
true && true == true
```

##  Example with a False Condition

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::conjunction<
        std::is_integral<int>,
        std::is_floating_point<int>
    >;

    std::cout << Result::value << '\n';

}
```

**Output:**

```txt
0
```

**Explanation:**

- `std::is_integral<int>` returns `true`.
- `std::is_floating_point<int>` returns `false`.

**Therefore:**

```txt
true && false == false
```

The final result is `false`.

##  Using Multiple Conditions

One of the advantages of `std::conjunction` is that you can combine several type traits.

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::conjunction<
        std::is_integral<int>,
        std::is_signed<int>,
        std::is_arithmetic<int>,
        std::is_object<int>
    >;

    std::cout << Result::value << '\n';

}
```

**Output:**

```txt
1
```

All four conditions are true, so the final result is true.

##  Using std::conjunction with Templates

Now let's look at a more practical example.

Suppose we want to create a template function that accepts only signed integral types.

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
void process(T value) {

    static_assert(
        std::conjunction<
            std::is_integral<T>,
            std::is_signed<T>
        >::value,
        "T must be a signed integral type"
    );

    std::cout << value << '\n';
}

int main() {

    process<int>(100);

    // process<unsigned int>(100); // Compilation error
    // process<double>(10.5);      // Compilation error

}
```

**Explanation:**

The `std::conjunction` checks two conditions:

1. Is `T` an integral type?
2. Is `T` a signed type?

Both conditions must be true.

If either condition is false, `static_assert` produces a compilation error.

##  Shorter Syntax: std::conjunction_v

C++17 also provides a variable template called `std::conjunction_v`.

Instead of writing:

```cpp
std::conjunction<
    std::is_integral<int>,
    std::is_signed<int>
>::value
```

You can write:

```**
std::conjunction_v<
    std::is_integral<int>,
    std::is_signed<int>
>
```

**Complete example:**

```cpp
#include <iostream>
#include <type_traits>

int main() {

    constexpr bool result =
        std::conjunction_v<
            std::is_integral<int>,
            std::is_signed<int>
        >;

    std::cout << result << '\n';

}
```

**Output:**

```txt
1
```

## An Important Feature: Short-Circuit Evaluation

One of the most interesting features of `std::conjunction` is its compile-time short-circuit behavior.

It does not need to evaluate the remaining traits after encountering the first false trait.

Consider:

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
struct Check {
    static_assert(sizeof(T) == 0,
                  "Check was instantiated!");

    static constexpr bool value = true;
};

using Result = std::conjunction<
    std::false_type,
    Check<int>
>;

int main() {

    std::cout << Result::value;

}
```

**Output:**

```txt
0
```

Why doesn't this produce a compilation error?

Because the first argument is `std::false_type`.

`std::conjunction` does not need to instantiate `Check<int>` to determine the result.



**Important:**

&#x20;This behavior is useful when working with complex templates, SFINAE, and conditional type validation.



##  Difference Between conjunction, disjunction, and negation

These three utilities are fundamental compile-time logical operations in C++17.

| Type Trait         | Logical Operator | Meaning                             |
| ------------------ | ---------------- | ----------------------------------- |
| `std::conjunction` | AND (`&&`)       | All conditions must be true         |
| `std::disjunction` | OR (`\\|\\|`)    | At least one condition must be true |
| `std::negation`    | NOT (`!`)        | Reverses a Boolean result           |

Mathematical representation:

\\[ A \land B \\]

\\[ A \lor B \\]

\\[ \neg A \\]


Recommended next step: Learn `std::disjunction` next. Once you understand `conjunction`, `disjunction`, and `negation`, you'll have a solid foundation for compile-time Boolean logic in C++17, including advanced template programming and SFINAE.

---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>
