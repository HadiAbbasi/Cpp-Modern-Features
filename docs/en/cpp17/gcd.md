<div align="center">

[🇺🇸 English](./gcd.md) | [🇮🇷 فارسی](../../fa/cpp17/gcd.md)

</div>

# std::gcd

The **`std::gcd`** function is a standard mathematical utility introduced in **C++17** under the C++ Standard Library. 

`gcd` stands for **Greatest Common Divisor**. It calculates the largest positive integer that divides each of the integers without a remainder.


##  Header & Syntax

To use `std::gcd`, you must include the **`<numeric>`** header:

```cpp
#include <numeric>
```

 **Syntax:**
```cpp
namespace std {
    template< class M, class N >
    constexpr std::common_type_t<M, N> gcd( M m, N n );
}
```

* **Parameters (`m` and `n`):** Must be of **integer types** (such as `int`, `long`, `short`, `unsigned int`).
* **Return Value:** Returns the greatest common divisor of the common type.
* **`constexpr`:** It is marked `constexpr`, meaning it can be evaluated at compile-time if the inputs are known.


##  Basic Example

```cpp
#include <iostream>
#include <numeric>

int main() {
    int a = 24;
    int b = 36;

    // Divisors of 24: 1, 2, 3, 4, 6, 8, 12, 24
    // Divisors of 36: 1, 2, 3, 4, 6, 9, 12, 18, 36
    // Greatest common divisor: 12

    int result = std::gcd(a, b);

    std::cout << "GCD of " << a << " and " << b << " is: " << result << '\n';
    // Output: GCD of 24 and 36 is: 12
}
```

##  Behavior with Negative Numbers and Zero (Important Rules)

1. **Negative Numbers:** The return value of `std::gcd` is always **non-negative** (positive or zero), even if one or both arguments are negative:
   ```cpp
   std::gcd(-24, 36);  // Output: 12
   std::gcd(-24, -36); // Output: 12
   ```

2. **Zero:**
   * If one argument is zero, it returns the absolute value of the other argument:
     ```cpp
     std::gcd(0, 15); // Output: 15
     ```
   * If both arguments are zero, it returns `0`:
     ```cpp
     std::gcd(0, 0);  // Output: 0
     ```

## 4. Practical Application: Simplifying Fractions

One of the most common use cases for GCD is simplifying fractions by dividing both the numerator and denominator by their greatest common divisor:

```cpp
#include <iostream>
#include <numeric>

struct Fraction {
    int num; // Numerator
    int den; // Denominator

    void simplify() {
        int common = std::gcd(num, den);
        if (common != 0) {
            num /= common;
            den /= common;
        }
    }
};

int main() {
    Fraction f{42, 56}; // Fraction 42/56

    f.simplify();

    std::cout << "Simplified: " << f.num << "/" << f.den << '\n';
    // Output: Simplified: 3/4
}
```

## Compile-Time Evaluation (`constexpr`)

Because the function is `constexpr`, you can use it to define compile-time constants or array sizes:

```cpp
#include <numeric>

constexpr int buffer_size = std::gcd(1024, 768); // Computed at compile-time: 256
int buffer[buffer_size]; // Array of size 256
```

## Companion Function: `std::lcm` (Least Common Multiple)

Alongside `std::gcd`, C++17 also introduced **`std::lcm`** (**Least Common Multiple**) in the same `<numeric>` header:

```cpp
#include <iostream>
#include <numeric>

int main() {
    int a = 4;
    int b = 6;

    std::cout << "GCD: " << std::gcd(a, b) << '\n'; // GCD: 2
    std::cout << "LCM: " << std::lcm(a, b) << '\n'; // LCM: 12
}
```
---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>