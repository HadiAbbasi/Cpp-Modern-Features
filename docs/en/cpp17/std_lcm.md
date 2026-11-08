<div align="center">

[🇺🇸 English](./lcm.md) | [🇮🇷 فارسی](../../fa/cpp17/lcm.md)

</div>

----

# `std::lcm` (Least Common Multiple) in C++17

Computing the **Least Common Multiple (LCM)** and **Greatest Common Divisor (GCD)** is a foundational requirement in domains such as real-time scheduling (e.g., synchronizing periodic tasks with different frequencies), cryptographic algorithms, rational number arithmetic, and spatial grid calculations. 

Prior to C++17, C and C++ developers had to write custom, error-prone implementations of integer arithmetic utilities or rely on non-standard extensions (`<numeric>` lacked built-in helpers, and `boost::integer::lcm` required external dependencies). 

C++17 introduced `<numeric>` utilities `std::lcm` and `std::gcd`, providing standardized, constexpr-enabled utilities whose representability requirements must be respected for all standard integer types.

```cpp
#include <iostream>
#include <numeric>

int main() {
    constexpr int a = 4;
    constexpr int b = 6;
    
    // Compile-time LCM computation
    constexpr auto result = std::lcm(a, b);
    
    std::cout << "LCM of " << a << " and " << b << " is: " << result << '\n'; // 12
    return 0;
}
```

---

## Table of Contents

1. [Why `std::lcm` Was Introduced](#1-why-stdlcm-was-introduced)
2. [Mathematical Definition and Corner Cases](#2-mathematical-definition-and-corner-cases)
3. [Signature and Behavior](#3-signature-and-behavior)
4. [Compile-Time Evaluation (`constexpr`)](#4-compile-time-evaluation-constexpr)
5. [Overflow and Representability Requirements](#5-overflow-and-representability-requirements)
6. [Real-World Architectural Patterns](#6-real-world-architectural-patterns)
   - 6.1 [Periodic Task Scheduling and Hyperperiods](#61-periodic-task-scheduling-and-hyperperiods)
   - 6.2 [Rational Arithmetic and Fraction Normalization](#62-rational-arithmetic-and-fraction-normalization)
   - 6.3 [Grid Alignment and Resolution Matching](#63-grid-alignment-and-resolution-matching)
7. [Common Pitfalls and Gotchas](#7-common-pitfalls-and-gotchas)
8. [Performance and Inlining](#8-performance-and-inlining)
9. [Best Practices](#9-best-practices)
10. [Evolution: C++20 Enhancements](#10-evolution-c20-enhancements)
11. [Feature-Test Macro and Compiler Support](#11-feature-test-macro-and-compiler-support)
12. [Quick Reference Card](#12-quick-reference-card)
13. [Complete Executable Example](#13-complete-executable-example)
14. [Exercises](#14-exercises)
15. [Conclusion](#15-conclusion)
16. [Contributors](#16-contributors)

---

## 1. Why `std::lcm` Was Introduced

Before C++17, computing the least common multiple required boilerplate code or manual formula application:

```cpp
// Pre-C++17 manual LCM calculation (prone to overflow and division by zero)
template <typename T>
T manual_gcd(T a, T b) {
    while (b != 0) {
        T r = a % b;
        a = b;
        b = r;
    }
    return a;
}

template <typename T>
T manual_lcm(T a, T b) {
    if (a == 0 || b == 0) return 0;
    return (a / manual_gcd(a, b)) * b; // Order matters to reduce intermediate overflow
}
```

Pitfalls of custom implementations included:
1. **Intermediate overflow:** Computing $(a \times b) / \text{gcd}(a, b)$ overflows the integer type before division occurs, even if the final LCM fits inside the type.
2. **Sign handling:** Negative inputs require careful magnitude normalization.
3. **Boilerplate overhead:** Every project maintained its own math utility header with differing edge-case semantics.

`std::lcm` standardizes this utility, ensuring safe computation with standard integer promotion rules and full `constexpr` support.

---

## 2. Mathematical Definition and Corner Cases

The least common multiple of two integers $m$ and $n$ is mathematically defined as:

$$\operatorname{lcm}(m, n) =
\begin{cases}
0, & \text{if } m = 0 \text{ or } n = 0 \\
\left|\dfrac{m}{\gcd(m, n)} \cdot n\right|, & \text{otherwise}
\end{cases}$$

### Standard Corner Cases Handled by `std::lcm`

| Input case | Result / behavior | Rationale |
|---|---|---|
| `m == 0` or `n == 0` | `0` | The least common multiple is defined as zero if either argument is zero. |
| Negative arguments | Non-negative LCM | The magnitudes of the arguments are used. |
| `|m|`, `|n|`, or the LCM is not representable in `std::common_type_t<M, N>` | Undefined behavior | The standard does not define an overflow result or an overflow exception. |
| `m == std::numeric_limits<int>::min()` with `int` as the common type | Undefined behavior | Its magnitude cannot be represented as an `int` (e.g. on two's complement systems). |

Negative inputs are permitted. The result is non-negative, provided the magnitudes of both inputs and the result are representable in `std::common_type_t<M, N>`. For example, `std::lcm(-4, 6)` evaluates to `12`.

---

## 3. Signature and Behavior

Defined in header `<numeric>`:

```cpp
namespace std {
    template <class M, class N>
    constexpr common_type_t<M, N> lcm(M m, N n);
}
```

- **Type Promotion:** Uses `std::common_type_t<M, N>` to determine the return type. Mixing `int` and `long` returns `long`.
- **Type Constraints:** Both template arguments `M` and `N` must be integer types other than `bool`. Floating-point types and `bool` do not satisfy the requirements and fail compilation.
- **Exception Specification:** `std::lcm` is not declared `noexcept`, but it does not throw standard exceptions to report overflow. Violations of representability result in undefined behavior.

---

## 4. Compile-Time Evaluation (`constexpr`)

`std::lcm` is fully `constexpr`, allowing computations at compile time for template arguments, array bounds, and static assertions:

```cpp
#include <numeric>

constexpr int task_a_period = 15;
constexpr int task_b_period = 20;

// Computed entirely at compile time
constexpr int hyperperiod = std::lcm(task_a_period, task_b_period); // 60

static_assert(hyperperiod == 60, "Hyperperiod calculation mismatch!");
```

---

## 5. Overflow and Representability Requirements

The mathematical identity is:

$$\operatorname{lcm}(m, n) =
\begin{cases}
0, & \text{if } m = 0 \text{ or } n = 0 \\
\left|\dfrac{m}{\gcd(m,n)} \cdot n\right|, & \text{otherwise}
\end{cases}$$

The return type is `std::common_type_t<M, N>`. If `|m|`, `|n|`, or the resulting LCM cannot be represented in that type, calling `std::lcm` has **undefined behavior**. The function does not provide an overflow exception, and unsigned arithmetic does not make an unrepresentable LCM safe to compute.

Dividing before multiplying is a useful technique in a manual implementation because it can reduce the risk of intermediate overflow. It does not remove `std::lcm`'s representability requirements: the magnitudes of both inputs and the final result must still be representable in the common type.

---

## 6. Real-World Architectural Patterns

### 6.1 Periodic Task Scheduling and Hyperperiods

In real-time operating systems (RTOS) and game simulation loops, tasks run at different frequencies. The **Hyperperiod** (LCM of all task periods) determines the exact scheduling window length required before the schedule repeats:

```cpp
#include <iostream>
#include <numeric>
#include <vector>

struct Task {
    const char* name;
    int period_ms;
};

int calculateHyperperiod(const std::vector<Task>& tasks) {
    if (tasks.empty()) return 0;
    int hl = tasks[0].period_ms;
    for (size_t i = 1; i < tasks.size(); ++i) {
        hl = std::lcm(hl, tasks[i].period_ms);
    }
    return hl;
}

int main() {
    std::vector<Task> tasks = {
        {"Sensor Read", 10},
        {"Control Loop", 25},
        {"Telemetry Sync", 100}
    };

    std::cout << "Schedule repeats every " << calculateHyperperiod(tasks) << " ms.\n";
    // Output: Schedule repeats every 100 ms.
}
```

### 6.2 Rational Arithmetic and Fraction Normalization

When adding or comparing fractions with differing denominators, the least common denominator is the LCM of the denominators:

```cpp
#include <numeric>

struct Fraction {
    long long num;
    long long den;

    Fraction add(const Fraction& other) const {
        long long common_den = std::lcm(den, other.den);
        long long scaled_num = num * (common_den / den) + other.num * (common_den / other.den);
        long long divisor = std::gcd(scaled_num, common_den);
        return {scaled_num / divisor, common_den / divisor};
    }
};
```

### 6.3 Grid Alignment and Resolution Matching

Synchronizing two physical grids, DMA transfers, or packet buffer alignments with different strides:

```cpp
#include <cstddef>
#include <numeric>

std::size_t alignBufferSize(std::size_t stride_a, std::size_t stride_b) {
    return std::lcm(stride_a, stride_b);
}
```

---

## 7. Common Pitfalls and Gotchas

### 7.1 Passing Floating-Point Types or `bool`

`std::lcm` only accepts integer types other than `bool`. Passing `float`, `double`, or `bool` results in a compilation failure:

```cpp
// double d = std::lcm(3.5, 2.1); // COMPILE ERROR: constraints not satisfied
// auto b = std::lcm(true, 4);    // COMPILE ERROR: bool is disallowed
```

### 7.2 Signed `numeric_limits::min()` Overflow

On two's complement systems, `std::numeric_limits<int>::min()` (e.g. `-2147483648`) has no positive counterpart in `int`. Computing its magnitude overflows, resulting in **Undefined Behavior**:

```cpp
int min_val = std::numeric_limits<int>::min();
// auto bad = std::lcm(min_val, 2); // UNDEFINED BEHAVIOR: |min_val| cannot be represented
```

### 7.3 Unsigned vs Signed Mixed Types

Mixing signed and unsigned types causes standard integer conversion rules to take effect via `std::common_type_t`. Negative signed values converted to unsigned become large positive values:

```cpp
unsigned int u = 4;
int s = -6;
// Common type is unsigned int; -6 converts to a large positive value before computation!
auto res = std::lcm(u, s); 
```

---

## 8. Performance and Inlining

Because `std::lcm` is implemented directly in standard library headers and marked `constexpr`, modern compilers (GCC, Clang, MSVC) optimize calls into **zero-overhead arithmetic instructions** or pre-compute constant values entirely at compile time without generating function call overhead.

---

## 9. Best Practices

1. **Leverage `constexpr`:** Use `std::lcm` in constant expressions wherever periods or alignments are known statically.
2. **Audit Value Ranges for Overflow:** Ensure the target integer type (`int`, `long long`, `int64_t`) has sufficient capacity to represent the product-divided-by-gcd without exceeding its maximum limit.
3. **Avoid Mixed-Sign/Unsigned Pairs:** Explicitly cast or sanitize inputs to identical signedness prior to invoking `std::lcm`.
4. **Combine with `std::gcd`:** Pair `std::gcd` and `std::lcm` together when dealing with fractional ratios, scale factors, or modular arithmetic.

---

## 10. Evolution: C++20 Enhancements

In C++20, the constraints on `std::lcm` and `std::gcd` were formally specified using concepts (`std::integral`), prohibiting non-integral types and `bool` with cleaner diagnostics during template instantiation.

---

## 11. Feature-Test Macro and Compiler Support

Header: `<numeric>`

```cpp
#if defined(__cpp_lib_gcd_lcm) && __cpp_lib_gcd_lcm >= 201606L
    // std::lcm and std::gcd available
#endif
```

### Compiler Support Table

| Compiler | Version | Standard Flag | Notes |
|---|---|---|---|
| **GCC** | 7.1+ | `-std=c++17` | Full support |
| **Clang** | 5.0+ | `-std=c++17` | Full support |
| **MSVC** | 2017 (15.3+) | `/std:c++17` | Full support |

---

## 12. Quick Reference Card

```cpp
#include <numeric>

// 1. Basic usage
auto l1 = std::lcm(12, 18); // Returns 36 (int)

// 2. Compile-time evaluation
constexpr auto l2 = std::lcm(7, 5); // Returns 35

// 3. Mixed type promotion
auto l3 = std::lcm(3, 4L); // Returns 12 (long)

// 4. Negative handling
auto l4 = std::lcm(-4, 6); // Returns 12
```

---

## 13. Complete Executable Example

```cpp
#include <iostream>
#include <numeric>

int main() {
    std::cout << "=== Demonstrating std::lcm in C++17 ===\n\n";

    // 1. Basic LCM calculations
    std::cout << "std::lcm(12, 18) = " << std::lcm(12, 18) << '\n';
    std::cout << "std::lcm(-9, 12) = " << std::lcm(-9, 12) << '\n';
    std::cout << "std::lcm(0, 5)   = " << std::lcm(0, 5) << '\n';

    // 2. Compile-time evaluation
    constexpr auto compile_time_lcm = std::lcm(8, 14);
    static_assert(compile_time_lcm == 56, "Compile-time LCM test failed!");
    std::cout << "Compile-time std::lcm(8, 14) = " << compile_time_lcm << '\n';

    // 3. Mixed type promotion
    int a = 6;
    long long b = 8;
    auto result = std::lcm(a, b);
    std::cout << "std::lcm(int{6}, long long{8}) type is long long, result = " << result << '\n';

    return 0;
}
```

### Expected Output

```text
=== Demonstrating std::lcm in C++17 ===

std::lcm(12, 18) = 36
std::lcm(-9, 12) = 36
std::lcm(0, 5)   = 0
Compile-time std::lcm(8, 14) = 56
std::lcm(int{6}, long long{8}) type is long long, result = 24
```

---

## 14. Exercises

### Exercise 1 — Compile-Time Hyperperiod
Write a `constexpr` function that computes the hyperperiod for three task frequencies: 12Hz, 18Hz, and 24Hz using `std::lcm`, verified via `static_assert`.

### Exercise 2 — Safe Fraction Addition
Implement a struct `Rational` supporting addition using `std::lcm` to find the common denominator and `std::gcd` to simplify the resulting fraction.

### Exercise 3 — Buffer Alignment Matcher
Given two device DMA buffer alignment requirements (e.g., 64 bytes and 96 bytes), write a function using `std::lcm` to determine the minimum unified buffer size satisfying both constraints.

---

## 15. Conclusion

`std::lcm` in C++17 brings clean, standardized, and efficient least-common-multiple calculations directly into the standard library:
- Replaces non-standard headers and manual implementations prone to overflow.
- Fully supports `constexpr` evaluation for zero runtime overhead.
- Operates under strict representability rules where unrepresentable values result in undefined behavior rather than silent wrap-around or exceptions.

---

## 16. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |