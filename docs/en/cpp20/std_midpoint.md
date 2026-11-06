<div align="center">

[🇺🇸 English](./std_midpoint.md) | [🇮🇷 فارسی](../../fa/cpp20/std_midpoint.md)

</div>

---

# Introduction to `std::midpoint` in C++20

## Introduction

Calculating the midpoint of two numeric values may seem straightforward at first: simply add the two values and divide the result by two. However, the expression `(a + b) / 2` is not always safe. Adding two large integers may cause overflow, and in floating-point calculations, the sum may exceed the representable range of the type even when the midpoint itself is representable.

The `std::midpoint` function, introduced into the standard library in C++20, was designed to address this problem. It calculates the midpoint between two numeric values or two pointers belonging to the same array. Each overload has specific rules concerning overflow, rounding, and valid inputs.

This feature is part of the Standard Library, not a language feature or a compiler-specific extension. Its declaration is provided in the `<numeric>` header.

## Function Overview and Signature

The `std::midpoint` function has two primary overloads:

```
#include <numeric>

template<class T>
constexpr T midpoint(T a, T b) noexcept;

template<class T>
constexpr T* midpoint(T* a, T* b);
```

For the first overload, `T` must be an arithmetic type other than `bool`. Therefore, standard integer and floating-point types, including `int`, `unsigned int`, `long long`, `float`, `double`, and `long double`, can be used.

For the second overload, the pointer type must point to a complete object type. Both pointers must belong to the same array. A pointer to the hypothetical element of a single-element array and a one-past-the-end pointer can also be used, provided they satisfy the standard Pointer Arithmetic rules.

An important point is that the function does not automatically select a common type for two arguments of different types. Both arguments must be compatible with the parameters of the same overload. Therefore, it is generally preferable to keep the input types identical and perform any necessary type conversions explicitly and deliberately.

## Why `std::midpoint` Exists

### The Risk of Overflow in Integer Arithmetic

The following common expression may overflow before the division by two takes place:

```
int middle = (a + b) / 2;
```

For example, if both inputs are close to the maximum value of the integer type, their sum may not be representable in that type. For unsigned integer types, the result is calculated modulo the type's range. For signed integer types, overflow results in Undefined Behavior.

Even rewriting the expression as follows is not a complete, general solution:

```
int middle = a + (b - a) / 2;
```

In this version, the subtraction `b - a` may overflow. This can happen, for example, when `a` is negative and `b` is positive, and the distance between them exceeds the representable range of the type.

Therefore, simply changing the order of operations does not guarantee safety.

### The Risk of Overflow in Floating-Point Arithmetic

In Floating-Point arithmetic, the expression `(a + b) / 2` may also produce a sum greater than the maximum representable value, even when the mathematical average of the two inputs is entirely within the type's representable range.

The `std::midpoint` function is defined to prevent this kind of overflow when calculating a midpoint.

### Calculating the Midpoint of Pointers

To find the middle element of an array, a common approach is to use an expression such as `first + (last - first) / 2`. This approach depends on subtracting two pointers, and the result of that subtraction must be representable by `std::ptrdiff_t`.

Using the pointer overload of `std::midpoint` makes it possible to calculate the midpoint without explicitly writing this subtraction.

## Understanding the Numeric Overload

### Integer Arithmetic

For integer inputs, the result is half the mathematical sum of the two arguments, without overflow.

If the sum of the two integer inputs is odd, the result is rounded toward the first argument (`a`).

This means that, when the exact result is halfway between two integers, the integer closer to the first argument is selected. Consequently, the function does not necessarily behave like a rounding operation that always rounds down or always rounds toward zero.

```
#include <iostream>
#include <numeric>

int main() {
    std::cout << std::midpoint(2, 5) << '\n';
    std::cout << std::midpoint(5, 2) << '\n';
    std::cout << std::midpoint(-2, 1) << '\n';
    std::cout << std::midpoint(1, -2) << '\n';
}
```

Output:

```
3
4
-1
0
```

In the first two calls, the sum is 7. Since the exact result is 3.5, the first call rounds toward 2, while the second rounds toward 5.

This behavior demonstrates that swapping the order of the arguments can change the final result.

### An Example of Overflow Prevention

The following example demonstrates how `std::midpoint` avoids an incorrect addition near the maximum value of an unsigned integer type:

```
#include <cstdint>
#include <iostream>
#include <limits>
#include <numeric>

int main() {
    const std::uint32_t a =
        std::numeric_limits<std::uint32_t>::max();

    const std::uint32_t b = a - 2;

    std::cout << "Unsafe expression: " << (a + b) / 2 << '\n';
    std::cout << "Safe midpoint: " << std::midpoint(a, b) << '\n';
}
```

In the first expression, the addition is performed using the `std::uint32_t` type. Because unsigned integer arithmetic uses modulo behavior, the result is incorrect.

In contrast, `std::midpoint` calculates the correct midpoint without overflow.

### Floating-Point Arithmetic

For floating-point inputs, the result is half the mathematical sum of the two arguments.

The standard guarantees that calculating `std::midpoint` does not overflow and performs at most one Inexact operation.

This guarantee does not mean that the result is always exact or that all rounding errors are eliminated. The usual limitations of binary floating-point representation still apply.

```
#include <iomanip>
#include <iostream>
#include <limits>
#include <numeric>

int main() {
    const double a = std::numeric_limits<double>::max();
    const double b = a;

    const double middle = std::midpoint(a, b);

    std::cout << std::setprecision(17) << middle << '\n';
}
```

In this example, the mathematical average of the two inputs is equal to the input value itself. Directly calculating `a + b` may exceed the representable range of `double`, but `std::midpoint` calculates the result without overflow.

### Restrictions on Using `bool`

Although `bool` belongs to the general category of Arithmetic types, the numeric overload of `std::midpoint` explicitly excludes it.

Therefore, calling this function directly with two `bool` values is not valid. If necessary, the logical values can first be converted to a suitable numeric type:

```
#include <numeric>

int main() {
    const bool a = false;
    const bool b = true;

    const int middle = std::midpoint(
        static_cast<int>(a),
        static_cast<int>(b)
    );
}
```

## Understanding the Pointer Overload

### Calculating the Midpoint Pointer

The second overload calculates the midpoint of two pointers belonging to the same array.

If the first pointer refers to index `i` and the second pointer refers to index `j`, the resulting index is determined by the following relationship:

\\[ i + \frac{j-i}{2} \\]

Division in this expression is truncated toward zero. Consequently, if the distance between the two indices is odd, the selected element is closer to the first pointer.

```
#include <iostream>
#include <numeric>

int main() {
    int values[] = {10, 20, 30, 40, 50, 60};

    int* first = values;
    int* last = values + 5;

    int* middle = std::midpoint(first, last);

    std::cout << *middle << '\n';
}
```

Output:

```
30
```

In this example, `first` points to index zero and `last` points to index five. The midpoint index is two.

### Using a One-Past-the-End Pointer

In range-based algorithms, the end of a range is commonly represented by a one-past-the-end pointer.

This pointer does not refer to an actual element, but creating and using it to represent the end of an array is permitted. Such a pointer can be used as one of the arguments to `std::midpoint`, provided that both pointers belong to the same array.

```
#include <iostream>
#include <numeric>

int main() {
    int values[] = {4, 8, 12, 16, 20};

    int* first = values;
    int* last = values + 5;

    int* middle = std::midpoint(first, last);

    if (middle != last) {
        std::cout << *middle << '\n';
    }
}
```

### The Requirement That Both Pointers Belong to the Same Array

Both pointers must refer to elements of the same array and satisfy the requirements for valid Pointer Arithmetic.

If the pointers refer to two independent arrays, the behavior of the program is undefined.

This function is not a tool for calculating the numeric distance between arbitrary memory addresses and should not be used to interpolate between unrelated pointers.

### Difference from Direct Pointer Subtraction

Consider the following common approach:

```
auto* middle = first + (last - first) / 2;
```

This expression depends on pointer subtraction. The result of the subtraction must be representable by `std::ptrdiff_t`.

When an array is larger than `PTRDIFF_MAX`, direct subtraction may not be usable, even if both pointers belong to the same array.

Using `std::midpoint(first, last)` is a more appropriate standard solution for this purpose.

## Understanding `constexpr` and `noexcept`

### Compile-Time Evaluation

Both overloads have been `constexpr` since C++20. Therefore, when the inputs are constant and the operations are valid during constant evaluation, they can be used in a Constant Expression.

```
#include <numeric>

constexpr int middle = std::midpoint(10, 20);

static_assert(middle == 15);

constexpr int values[] = {2, 4, 6, 8, 10};

constexpr const int* pointerMiddle =
    std::midpoint(values, values + 5);

static_assert(*pointerMiddle == 6);
```

This feature is useful for calculations that must be performed at compile time, including constant data structures and `static_assert` checks.

### Exception Guarantees

The numeric overload is declared `noexcept` and does not throw exceptions.

The pointer overload is not declared `noexcept` in its standard signature. However, for valid inputs, it performs ordinary pointer operations and does not throw ordinary exceptions.

The absence of `noexcept` in the pointer overload's signature should not be confused with a guarantee of behavior for invalid inputs. Violating the pointer preconditions may result in Undefined Behavior, not necessarily an exception being thrown.

## Practical Applications in Programming

### Using It in Binary Search

One of the best-known applications of midpoint calculation is Binary Search.

In an index-based implementation, using `(low + high) / 2` may cause overflow when the indices are large.

A safe and readable alternative is to use `std::midpoint`:

```
#include <numeric>
#include <vector>

int binarySearch(const std::vector<int>& values, int target) {
    std::size_t low = 0;
    std::size_t high = values.size();

    while (low < high) {
        const std::size_t middle = std::midpoint(low, high);

        if (values[middle] < target) {
            low = middle + 1;
        } else if (values[middle] > target) {
            high = middle;
        } else {
            return static_cast<int>(middle);
        }
    }

    return -1;
}
```

In this example, the range is defined as a half-open interval, `[low, high)`.

This design allows `high` to be equal to `values.size()` without using it as an element index inside the loop.

In real-world projects, the function's return type should be chosen to match the size of the container. Using `int` in this example is simply intended to keep the example straightforward.

### Processing a Range with Pointers

In low-level code or algorithms that use pointer-based ranges, the pointer overload can provide the middle element without requiring manual pointer subtraction.

However, in modern C++, Iterators, `std::span`, and standard algorithms are generally preferred over direct pointer manipulation. Pointer usage should be justified by the actual low-level requirements of the program.

### Applications in Divide and Conquer Algorithms

In Divide and Conquer algorithms, ranges are divided into smaller sections.

Using `std::midpoint` to calculate the division boundary prevents errors caused by adding two bounds and makes the purpose of the code clearer.

### Applications in Numerical and Geometric Calculations

When the midpoint between two coordinates or numeric values is needed, `std::midpoint` is a suitable option for calculating the average.

However, this function is not a complete replacement for Interpolation. For interpolation between two values using an arbitrary parameter, the related C++20 feature `std::lerp` is more appropriate.

## Comparing Midpoint Calculation Methods

| Method                | Advantage                                                                            | Limitation                                                              |
| --------------------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| `(a + b) / 2`         | Simple and familiar                                                                  | May overflow during addition                                            |
| `a + (b - a) / 2`     | Avoids addition overflow in many cases                                               | Subtraction may overflow; pointer subtraction is limited by `ptrdiff_t` |
| `std::midpoint(a, b)` | Standard guarantee against overflow in the numeric overload and support for pointers | Defined only for specific types and valid preconditions                 |

When choosing an appropriate method, you should not focus solely on whether an expression happens to work with the current architecture and compiler.

Standard guarantees and the range of valid inputs are more important.

## Difference Between `std::midpoint` and `std::lerp`

The `std::midpoint` function calculates the midpoint between two values; in other words, it provides a special case of Interpolation with a parameter of one-half.

In contrast, `std::lerp(a, b, t)` is designed to perform linear interpolation using a parameter `t`.

```
#include <iostream>
#include <numeric>

int main() {
    const double a = 10.0;
    const double b = 30.0;

    std::cout << std::midpoint(a, b) << '\n';
    std::cout << std::lerp(a, b, 0.25) << '\n';
}
```

Output:

```
20
15
```

Use `std::midpoint` to calculate the midpoint. For interpolation with an arbitrary factor, `std::lerp` is the more appropriate choice.

Both features were added to the standard library in C++20 and are provided in the `<numeric>` header.

## Important Notes and Common Mistakes

- Argument order matters: For integer calculations with an odd sum, rounding is performed toward the first argument. Therefore, `std::midpoint(a, b)` is not necessarily equal to `std::midpoint(b, a)`.
- Choose input types deliberately: The function does not convert two input types to a common type. Implicit or explicit conversions before the call may change the range and precision of the calculation.
- The `bool` restriction: `bool` is not a valid input type for the numeric overload. If necessary, convert it to a suitable numeric type.
- Pointer validity: Using pointers that belong to two independent arrays violates the function's preconditions.
- Pay attention to half-open ranges: The midpoint pointer is not necessarily the same element that an algorithm with a different range convention expects. Define the meaning of indices and the stopping condition precisely.
- `constexpr` limitations: This feature does not mean that every input can be evaluated at compile time. The inputs and operations must satisfy the requirements for a Constant Expression.
- It does not replace bounds checking: `std::midpoint` prevents overflow during midpoint calculation, but validating indices, container sizes, and element access remains the programmer's responsibility.
- No general overload for custom types: This function is defined for Arithmetic types other than `bool` and for eligible pointers. For custom numeric types, the corresponding operations must be designed with appropriate contracts and guarantees.

## Checking Compatibility and Feature Support

The `std::midpoint` feature was introduced into the standard library in C++20.

To use it, C++20 or a later language mode must be enabled for compilation, and the Standard Library implementation must provide the feature.

Compiler language support and Standard Library support are related but separate concerns. Simply enabling the C++20 option in a compiler does not necessarily guarantee that an older library implementation provides all the corresponding library features.

The feature-test macro associated with this functionality is `__cpp_lib_interpolate`, whose value is `201902L`. This macro is defined for the interpolation utilities, including `std::midpoint` and `std::lerp`.

```
#include <numeric>

#if defined(__cpp_lib_interpolate) && __cpp_lib_interpolate >= 201902L
    // std::midpoint and std::lerp are available
#else
    #error "The standard library does not provide C++20 interpolation utilities"
#endif
```

Include the `<numeric>` header to use this macro.

In Build systems, it is also advisable to check the language standard and the version of the Standard Library being used independently.

## Summary

The `std::midpoint` function is a small but important utility in the C++20 Standard Library.

It calculates the midpoint of two Arithmetic values without overflow, defines a specific rounding rule for integer inputs, and provides a separate overload for pointers belonging to the same array.

In modern code, whenever you need the midpoint between two numeric values or two valid positions in an array, using `std::midpoint` is generally safer and clearer than writing a manual formula.

However, the pointer overload's preconditions, input types, rounding behavior, and differences from `std::lerp` should all be considered when designing an algorithm.

## References

- [cppreference: `std::midpoint`](https://en.cppreference.com/w/cpp/numeric/midpoint?utm_source=chatgpt.com)
- [C++20 Working Draft N4860, Section `numeric.ops.midpoint`](https://isocpp.org/files/papers/N4860.pdf?utm_source=chatgpt.com)
- [cppreference: `std::lerp`](https://en.cppreference.com/w/cpp/numeric/lerp?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>