<div align="center">

[🇺🇸 English](./ranges_to.md) | [🇮🇷 فارسی](../../fa/cpp23/ranges_to.md)

</div>

# std::ranges::to

`std::ranges::to` is a C++23 feature that allows you to **convert a Range into a container**.

In simple terms, it takes a Range, such as a `vector`, `views::iota`, or a combination of Views, and converts its elements into a container such as `std::vector`, `std::list`, or `std::set`.

One of its biggest advantages is that it makes converting Views into containers much easier and more readable.

##  Basic syntax

```cpp
#include <ranges>

auto container = std::ranges::to<Container>(range);
```

For example:

```cpp
std::vector<int> v =
    std::ranges::to<std::vector<int>>(some_range);
```

You can also use the pipe syntax:

```cpp
auto v = some_range | std::ranges::to<std::vector>();
```

The second form uses a container template rather than specifying all its template arguments.

##  Basic example: Converting a View into a vector

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    auto r = std::views::iota(1, 6);

    auto v = std::ranges::to<std::vector<int>>(r);

    for (int x : v)
        std::cout << x << ' ';
}
```

**Output:**

```txt
1 2 3 4 5
```

Here:

- `std::views::iota(1, 6)` generates a range containing numbers from 1 to 5.
- `std::ranges::to<std::vector<int>>` converts that range into a `std::vector<int>`.

Unlike a View, the resulting vector owns its elements.

##  Using pipe syntax

One of the most convenient features of `std::ranges::to` is its integration with the Ranges pipeline.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    auto v = std::views::iota(1, 11)
           | std::views::filter([](int x)
             {
                 return x % 2 == 0;
             })
           | std::ranges::to<std::vector>();

    for (int x : v)
        std::cout << x << ' ';
}
```

**Output:**

```txt
2 4 6 8 10
```

This example:

1. Generates numbers from 1 to 10.
2. Filters out odd numbers.
3. Converts the resulting View into a vector.

Notice how we don't need to manually iterate over the View and insert elements into a vector.

##  Converting into different containers

`std::ranges::to` is not limited to `std::vector`.

For example, converting into a `std::list`:

```cpp
#include <list>
#include <ranges>

int main()
{
    auto r = std::views::iota(1, 6);

    auto values = std::ranges::to<std::list<int>>(r);
}
```

Converting into a `std::set`:

```cpp
#include <ranges>
#include <set>

int main()
{
    auto r = std::views::iota(1, 6);

    auto values = std::ranges::to<std::set<int>>(r);
}
```

The resulting containers are:

```txt
list: 1, 2, 3, 4, 5
set:  1, 2, 3, 4, 5
```

Remember that `std::set` automatically stores unique elements in sorted order.

##  Using it with transform

You can combine `to` with `transform` to generate transformed elements and store them in a container.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    auto v = std::views::iota(1, 6)
           | std::views::transform([](int x)
             {
                 return x * x;
             })
           | std::ranges::to<std::vector>();

    for (int x : v)
        std::cout << x << ' ';
}
```

**Output:**

```txt
1 4 9 16 25
```

The transformation is applied lazily, and `std::ranges::to` materializes the resulting elements into a vector.

##  Using it with strings

You can also convert a Range of characters into a `std::string`.

```cpp
#include <iostream>
#include <ranges>
#include <string>

int main()
{
    auto r = std::views::iota('A', 'F');

    auto s = std::ranges::to<std::string>(r);

    std::cout << s;
}
```

**Output:**

```txt
ABCDE
```

This is particularly useful when working with character Views.

##  Converting a Range into a container with a different element type

`std::ranges::to` can also perform element conversions when the target container supports construction or insertion from the source elements.

For example:

```cpp
#include <ranges>
#include <vector>

int main()
{
    auto r = std::views::iota(1, 6);

    auto v = std::ranges::to<std::vector<double>>(r);
}
```

The resulting vector contains:

```txt
1.0, 2.0, 3.0, 4.0, 5.0
```

##  Technical characteristics

| Feature              | Description                      |
| -------------------- | -------------------------------- |
| Standard             | C++23                            |
| Header               | `<ranges>`                       |
| Function             | `std::ranges::to`                |
| Purpose              | Convert a Range into a container |
| Supports Views       | Yes                              |
| Supports pipe syntax | Yes                              |
| Element conversion   | Supported when constructible     |
| Result               | An owning container              |

A few important points:

- `std::ranges::to` is a conversion utility, not a View.
- The resulting container owns its elements.
- It can work with lazy Views.
- It can be combined with `filter`, `transform`, `take`, and other Range adaptors.
- The destination container must satisfy the construction requirements of the algorithm.

##  Comparison with traditional approaches

Before C++23, converting a Range into a container often required manually inserting elements.

Traditional approach:

```cpp
std::vector<int> v;

for (int x : std::views::iota(1, 6))
{
    v.push_back(x);
}
```

**C++23 approach:**

```cpp
auto v = std::views::iota(1, 6)
       | std::ranges::to<std::vector>();
```

The C++23 version is shorter, cleaner, and integrates naturally with Range pipelines.

## **Important comparison:&#x20;**`std::views`**&#x20;vs&#x20;**`std::ranges::to`

Consider this example:

```cpp
auto r = std::views::iota(1, 6);

auto v = r | std::ranges::to<std::vector>();
```

The difference is:

| `std::views`                      | `std::ranges::to`          |
| --------------------------------- | -------------------------- |
| Creates a View                    | Creates a container        |
| Usually lazy                      | Materializes elements      |
| Does not necessarily own elements | Owns its elements          |
| Useful for composing operations   | Useful for storing results |

**The key idea:** Use `std::views` to describe how you want to process data, and use `std::ranges::to` when you want to materialize the result into a real container.

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>