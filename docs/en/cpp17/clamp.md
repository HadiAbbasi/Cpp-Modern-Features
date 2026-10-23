<div align="center">

[🇺🇸 English](./clamp.md) | [🇮🇷 فارسی](../../fa/cpp17/clmap.md)

</div>

# std::clamp

The std::clamp function, introduced in C++17, is a clean and very useful utility in the C++ Standard Library. Its purpose is to constrain a value within a specified range (between a minimum and maximum bound).

Here is a complete guide to its syntax, usage, and examples.

**To use std::clamp, you need to include the <algorithm> header:**

## use Function
It generally comes in two overloads:

### Using the default < operator

```cpp
namespace std {
    constexpr const T& clamp( const T& v, const T& lo, const T& hi );
}
```
**Parameters:**
- v: The value to be clamped.
- lo: The lower boundary (minimum allowed value).
- hi: The upper boundary (maximum allowed value).

**Return Value:**
- Returns lo if v is less than lo.
- Returns hi if v is greater than hi.
- Returns v otherwise.


###  Using a custom comparator
If you need to compare values using a custom rule:

```cpp
namespace std {
    constexpr const T& clamp( const T& v, const T& lo, const T& hi, Compare comp );
}
```

## Basic Example
```cpp
#include <iostream>
#include <algorithm>

int main() {
    int val = 15;
    int min_val = 0;
    int max_val = 10;

    // Since 15 is greater than 10, the result will be clamped to 10
    int result = std::clamp(val, min_val, max_val);

    std::cout << result << '\n'; // Output: 10
}
```
**Different Scenarios for std::clamp(v, lo, hi):**
1. If v = 5, lo = 0, hi = 10 ➔ Output: 5 (within range)
2. If v = -5, lo = 0, hi = 10 ➔ Output: 0 (less than minimum, clamped to lo)
3. If v = 20, lo = 0, hi = 10 ➔ Output: 10 (greater than maximum, clamped to hi)


## Important Notes
- Precondition: The behavior is undefined if lo > hi (the lower bound must be less than or equal to the upper bound).
- Returns a Reference: std::clamp returns a reference to the constrained value (const T&), avoiding unnecessary copies for complex objects.
- constexpr Support: It is marked constexpr, meaning it can be evaluated at compile-time.

## Comparison with Pre-C++17 Code
Before C++17, developers had to nest std::min and std::max to achieve clamping, which was more verbose and less readable:
```cpp
// Old way (Pre-C++17)
int result = std::max(min_val, std::min(val, max_val));
// Modern way (C++17 and later)
int result = std::clamp(val, min_val, max_val);
```


---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>