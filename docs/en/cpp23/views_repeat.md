<div align="center">

[🇺🇸 English](./views_repeat.md) | [🇮🇷 فارسی](../../fa/cpp23/views_repeat.md)

</div>

# std::views::repeat

`std::views::repeat` is a C++23 Ranges feature used to generate a sequence of repeated elements.

In simple terms, you provide a value, and this View produces that value a specified number of times or infinitely.

##  Syntax

```cpp
#include <ranges>

std::views::repeat(value);
std::views::repeat(value, count);
```

There are two forms:

1. Without `count`: produces an infinite range.
2. With `count`: produces a finite range containing the specified number of elements.

##  Basic example: Finite repetition

```cpp
#include <iostream>
#include <ranges>

int main()
{
    for (int x : std::views::repeat(10, 5))
    {
        std::cout << x << ' ';
    }
}
```

**Output:**

```txt
10 10 10 10 10
```

In this example, the value `10` is repeated five times.

##  Infinite repetition

If you don't specify the second argument, an infinite range is created.

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto r = std::views::repeat(42);

    for (int x : r | std::views::take(5))
    {
        std::cout << x << ' ';
    }
}
```

**Output:**

```txt
42 42 42 42 42
```

**Important:** Since the range is infinite, you should generally use `take` or another limiting mechanism when iterating over it.

##  Combining with other Views

One of the main advantages of `views::repeat` is that it can be combined with other Views from the Ranges library.

For example, combining it with `transform`:

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto r =
        std::views::repeat(5, 4)
        | std::views::transform([](int x)
        {
            return x * 2;
        });

    for (int x : r)
        std::cout << x << ' ';
}
```

**Output:**

```txt
10 10 10 10
```

Another example: combining it with `filter`:

```cpp
auto r =
    std::views::repeat(7, 10)
    | std::views::filter([](int x)
    {
        return x % 2 == 0;
    });
```

In this example, no elements remain because all elements are equal to `7`.

##  Using it to generate strings

This View is not limited to numbers. It can also work with other types.

```cpp
#include <iostream>
#include <ranges>
#include <string>

int main()
{
    for (const auto& s :
         std::views::repeat(std::string{"Hello"}, 3))
    {
        std::cout << s << '\n';
    }
}
```

**Output:**

```txt
Hello
Hello
Hello
```

##  Technical characteristics and performance

| Feature                 | Value                 |
| ----------------------- | --------------------- |
| Standard                | C++23                 |
| Header                  | `<ranges>`            |
| View type               | `repeat_view`         |
| Range category          | `random_access_range` |
| Finite version          | `sized_range`         |
| Infinite version        | Infinite Range        |
| Construction complexity | O(1)                  |
| Element generation      | Lazy                  |

Some important details:

- Elements are not stored in a separately allocated container.
- `repeat_view` stores the value being repeated.
- Random access through iterators is supported.
- For a finite range, `size()` returns the number of elements.

##  Comparison with older approaches

Before C++23, we typically used a `vector` or a loop to generate repeated elements.

Traditional approach:

```cpp
std::vector<int> v(5, 10);
```

C++23 approach:

```cpp
auto v = std::views::repeat(10, 5);
```

The important difference is that the second approach creates a lazy View and does not necessarily allocate separate storage for five elements.

##  Checking compiler support

You can check whether your standard library supports this feature using:

```
#ifdef __cpp_lib_ranges_repeat

    // std::views::repeat is supported

#endif
```

The feature-test macro value is:

```cpp
__cpp_lib_ranges_repeat == 202207L
```

This feature was introduced in C++23 through proposal P2474R2.


## Interesting note:&#x20;`std::views::iota`&#x20;vs&#x20;`std::views::repeat`

The main difference is:

```cpp
std::views::iota(1, 6);   // 1, 2, 3, 4, 5

std::views::repeat(1, 5); // 1, 1, 1, 1, 1
```

- `iota` generates an increasing sequence.
- `repeat` generates a sequence containing the same value repeatedly.

---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>