<div align="center">

[🇺🇸 English](./negation.md) | [🇮🇷 فارسی](../../fa/cpp17/negation.md)

</div>

# std::negation in C++17

`std::negation` is a class template in the C++ Standard Library, introduced in C++17. It belongs to the Type Traits library and is used to logically negate the result of another type trait.

In simple terms:

- If `T::value` is `true`, `std::negation<T>::value` is `false`.
- If `T::value` is `false`, `std::negation<T>::value` is `true`.

##  Introduction to std::negation

First, include the `<type_traits>` header:

```cpp
#include <type_traits>
```

General syntax:

```cpp
std::negation<T>
```

It works with types that provide a compile-time Boolean value.

##  A Simple Example

```cpp
#include <iostream>
#include <type_traits>

int main() {

    std::cout << std::is_integral<int>::value << '\n';

    std::cout << std::negation<
        std::is_integral<int>
    >::value << '\n';

}
```

**Output:**

```txt
1
0
```

**Explanation:**

- `std::is_integral<int>` checks whether `int` is an integral type. The result is `true`.
- `std::negation` reverses that result, so the final value is `false`.

##  Example with float

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::negation<
        std::is_integral<float>
    >;

    std::cout << Result::value << '\n';

}
```

**Output:**

```txt
1
```

**Explanation:**

- `float` is not an integral type.
- Therefore, `std::is_integral<float>::value` is `false`.
- `std::negation` reverses `false` to `true`.

##  Using std::bool_constant

In C++17, we can also use `std::bool_constant` to represent a compile-time Boolean value.

```cpp
#include <type_traits>
#include <iostream>

int main() {

    using A = std::bool_constant<true>;

    using B = std::negation<A>;

    std::cout << B::value;

}
```

**Output:**

```txt
0
```

**Explanation:**

- `A` represents `true`.
- `std::negation<A>` reverses it.
- Therefore, `B::value` is `false`.

##  Difference Between std::negation and std::not_fn

Do not confuse these two utilities.

| Feature    | std::negation                        | std::not_fn                 |
| ---------- | ------------------------------------ | --------------------------- |
| Category   | Type Trait                           | Function Adapter            |
| Purpose    | Negates a type trait's Boolean value | Negates a function's result |
| Header     | `<type_traits>`                      | `<functional>`              |
| Introduced | C++17                                | C++17                       |

Example of `std::not_fn`:

```cpp
#include <iostream>
#include <functional>

int main() {

    auto isEven = [](int x) {
        return x % 2 == 0;
    };

    auto isOdd = std::not_fn(isEven);

    std::cout << isOdd(5) << '\n'; // 1
    std::cout << isOdd(4) << '\n'; // 0

}
```

##  A More Practical Example: Checking Non-Integral Types

We can combine `std::negation` with `if constexpr` in C++17.

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
void check() {

    if constexpr (
        std::negation<
            std::is_integral<T>
        >::value
    ) {

        std::cout << "Not an integral type\n";

    } else {

        std::cout << "Integral type\n";

    }
}

int main() {

    check<int>();
    check<double>();
    check<char>();

}
```

**Output:**

```txt
Integral type
Not an integral type
Integral type
```

**Explanation:**

- `int` is an integral type.
- `double` is not an integral type.
- `char` is an integral type.

The `if constexpr` statement evaluates the condition at compile time.

##  Important: std::negation_v in C++17

C++17 provides a shorter variable template version:

```cpp
std::negation_v<T>
```

Instead of writing:

```cpp
std::negation<
    std::is_integral<double>
>::value
```

You can simply write:

```cpp
std::negation_v<
    std::is_integral<double>
>
```

**Example:**

```cpp
#include <iostream>
#include <type_traits>

int main() {

    std::cout << std::negation_v<
        std::is_integral<double>
    >;

}
```

**Output:**

```txt
1
```

##  Three Important Logical Type Traits in C++17

These three utilities are closely related:

| Type Trait         | Logical Operation | Meaning                             |
| ------------------ | ----------------- | ----------------------------------- |
| `std::conjunction` | AND               | All conditions must be true         |
| `std::disjunction` | OR                | At least one condition must be true |
| `std::negation`    | NOT               | Reverses a Boolean result           |

Mathematically:

\\[ \text{conjunction}(A,B)=A\land B \\]

\\[ \text{disjunction}(A,B)=A\lor B \\]

\\[ \text{negation}(A)=\neg A \\]



Key takeaway: `std::conjunction`, `std::disjunction`, and `std::negation` are important tools for compile-time Boolean logic in C++17, especially when working with Templates, Type Traits, SFINAE, and compile-time constraints.

---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>
