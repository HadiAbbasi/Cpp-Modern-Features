<div align="center">

[🇺🇸 English](./range_find_last.md) | [🇮🇷 فارسی](../../fa/cpp23/range_find_last.md)

</div>

#  std::range::find_last

`std::ranges::find_last` is a C++23 algorithm that searches a range **from the beginning to the end and returns the last element matching a given value**.

It is useful when you want the last occurrence of an element without manually iterating backward.

##  Basic syntax

```cpp
#include <algorithm>
#include <vector>

auto result = std::ranges::find_last(range, value);
```

The returned object is a `std::ranges::subrange` containing:

- `begin()` — iterator pointing to the last matching element.
- `end()` — iterator pointing just after that element.

If no matching element exists, both iterators point to the end of the range.

##  Example

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v = {1, 2, 3, 2, 4, 2, 5};

    auto result = std::ranges::find_last(v, 2);

    if (!result.empty())
        std::cout << *result.begin(); // 2
}
```

The last occurrence of `2` is at index `5`.

Notice that `find_last` returns a subrange, not a single iterator.

##  Using&#x20;**`find_last_if`

You can search using a condition instead of equality.

```cpp
std::vector<int> v = {1, 3, 5, 2, 7, 4, 9, 6};

auto result = std::ranges::find_last_if(
    v,
    [](int x) { return x % 2 == 0; }
);

if (!result.empty())
    std::cout << *result.begin(); // 6
```

This finds the last even number.

##  Using projection

One particularly useful feature is the projection parameter.

```cpp
#include <algorithm>
#include <string>
#include <vector>

struct Person {
    std::string name;
    int age;
};

int main()
{
    std::vector<Person> people = {
        {"Alice", 25},
        {"Bob", 30},
        {"Charlie", 25},
        {"David", 40}
    };

    auto result = std::ranges::find_last(
        people,
        25,
        &Person::age
    );

    if (!result.empty())
        std::cout << result.begin()->name; // Charlie
}
```

Here, `&Person::age` tells the algorithm to compare the `age` member instead of the entire object.

##  Important:&#x20;**`find_last`**&#x20;vs&#x20;**`find_end`

| Algorithm              | Purpose                                        |
| ---------------------- | ---------------------------------------------- |
| `ranges::find`         | Finds the first matching element               |
| `ranges::find_last`    | Finds the last matching element                |
| `ranges::find_if`      | Finds the first element satisfying a predicate |
| `ranges::find_last_if` | Finds the last element satisfying a predicate  |
| `ranges::find_end`     | Finds the last occurrence of a subsequence     |

##  Performance considerations

- Requires a `forward_range` at minimum.
- Time complexity: O(N).
- Uses at most N comparisons.
- Bidirectional and random-access iterators can benefit from more efficient implementations.

**Feature-test macro:**

```
#if defined(__cpp_lib_ranges_find_last) && \
    __cpp_lib_ranges_find_last >= 202207L
    // find_last is supported
#endif
```






---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>