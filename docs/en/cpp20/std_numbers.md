<div align="center">

[🇺🇸 English](./std_numbers.md) | [🇮🇷 فارسی](../../fa/cpp20/std_numbers.md)

</div>

---

# Introduction to `std::numbers` in C++20

## Introduction

One of the common requirements in scientific programming, engineering, simulation, signal processing, computer graphics, and mathematical computing is the use of mathematical constants. Values such as pi, Euler's number, the square root of two, and natural logarithms play an essential role in many algorithms and mathematical formulas.

In older versions of C++, programmers typically defined these values manually, used macros available in certain environments, or depended on external libraries. These approaches introduced problems such as platform dependencies, lack of type safety, possible redefinition, and limitations when working with different data types.

C++20 introduced a new header named `<numbers>`, providing a standardized and type-safe way to access a collection of mathematical constants.

This functionality resides in the `std::numbers` namespace and includes constants such as `pi`, `e`, and `sqrt2`, along with their variable template counterparts.

The `std::numbers` library allows developers to use standard mathematical constants with different floating-point types without having to define those constants manually.

## Introducing the `std::numbers` Library

The `std::numbers` library is part of the C++20 Standard Library and is designed to provide commonly used mathematical constants.

To use this functionality, include the following header:

```
#include <numbers>
```

All constants in this library are defined in the following namespace:

```
std::numbers
```

Here is a simple example:

```
#include <iostream>
#include <numbers>

int main() {
    constexpr double radius = 5.0;

    constexpr double area =
        std::numbers::pi * radius * radius;

    std::cout << "Area: " << area << '\n';
}
```

In this example, the value of pi is obtained through `std::numbers::pi`.

Using these constants makes code more readable and eliminates the need to manually define approximate values such as `3.14159`.

## Why `std::numbers` Was Introduced

### Problems with Manually Defining Mathematical Constants

In many older programs, mathematical constants were defined as follows:

```
constexpr double PI = 3.14159265358979323846;
constexpr double E = 2.71828182845904523536;
```

This approach can work for small programs, but it introduces several problems in larger projects:

- Repeated definitions: Different parts of a project may define the same constant with different values.
- Difficult maintenance: Changing or correcting constants requires checking multiple locations throughout the project.
- Data type dependencies: Defining a value for `double` does not necessarily make it suitable for calculations involving `float` or `long double`.
- Lack of a common standard: Different libraries may use different names or values.
- Potential human error: Mistyping digits in long constant definitions can lead to calculation errors.

### Limitations of Legacy Macros

Some environments provide macros such as `M_PI`:

```
#include <cmath>

double circumference = 2.0 * M_PI * radius;
```

However, `M_PI` is not a standard C++ constant that can be relied upon across all implementations. Its availability may depend on the platform, C library, or compiler settings.

By contrast, `std::numbers::pi` is part of the standard C++20 Standard Library.

### Benefits of a Standardized Design

The `std::numbers` library offers several important benefits:

- Provides commonly recognized mathematical constants under standardized names.
- Supports `constexpr` for compile-time use.
- Provides variable template versions for floating-point types.
- Avoids dependencies on non-standard macros.
- Improves code readability and maintainability.
- Can be used directly in generic programming algorithms.

## Mathematical Constants in C++

Mathematical constants are values that have a specific meaning within a problem or algorithm and, unlike ordinary variables, should not change during program execution.

For example, pi is the ratio of a circle's circumference to its diameter and is widely used in geometric calculations.

In C++, constants can be defined using `const` or `constexpr`.

The important difference between them concerns when their values are evaluated:

- `const` guarantees that a value cannot be modified, but by itself does not require that the value be evaluated at compile time.
- `constexpr` is used for values that must be capable of being evaluated at compile time.

The constants in `std::numbers` are `inline constexpr`, meaning that they are immutable and suitable for use in constant expressions.

## Structure and Members of the Library

The `std::numbers` namespace contains two main groups of constants:

- Non-template constants of type `double`.
- Variable templates that allow a constant to be obtained using a specified floating-point type.

For example, `std::numbers::pi` has type `double`, while `std::numbers::pi_v<float>` provides the same constant as a `float`.

### Non-Template Constants

The non-template members of this library are:

| Name         | Mathematical meaning                              | Approximate value |
| ------------ | ------------------------------------------------- | ----------------- |
| `e`          | Euler's number, the base of the natural logarithm | 2.718281828459045 |
| `log2e`      | Base-2 logarithm of e                             | 1.442695040888963 |
| `log10e`     | Base-10 logarithm of e                            | 0.434294481903252 |
| `pi`         | Pi                                                | 3.141592653589793 |
| `inv_pi`     | Reciprocal of pi                                  | 0.318309886183791 |
| `inv_sqrtpi` | Reciprocal of the square root of pi               | 0.564189583547756 |
| `ln2`        | Natural logarithm of two                          | 0.693147180559945 |
| `ln10`       | Natural logarithm of ten                          | 2.302585092994046 |
| `sqrt2`      | Square root of two                                | 1.414213562373095 |
| `sqrt3`      | Square root of three                              | 1.732050807568877 |
| `inv_sqrt3`  | Reciprocal of the square root of three            | 0.577350269189626 |
| `egamma`     | Euler–Mascheroni constant                         | 0.577215664901533 |
| `phi`        | Golden ratio                                      | 1.618033988749895 |

These values are intended for general-purpose mathematical calculations. The digits in the table are approximate representations and should not be treated as manual definitions more precise than the standard constants.

### Template Versions

A corresponding variable template with the `_v` suffix is available for each of the constants listed above. These templates are defined in the `std::numbers` namespace.

For example:

```
std::numbers::pi
std::numbers::pi_v<float>
std::numbers::pi_v<double>
std::numbers::pi_v<long double>
```

The template versions allow the type of a constant to match the type used in a calculation.

The template names are:

```
std::numbers::e_v<T>
std::numbers::log2e_v<T>
std::numbers::log10e_v<T>
std::numbers::pi_v<T>
std::numbers::inv_pi_v<T>
std::numbers::inv_sqrtpi_v<T>
std::numbers::ln2_v<T>
std::numbers::ln10_v<T>
std::numbers::sqrt2_v<T>
std::numbers::sqrt3_v<T>
std::numbers::inv_sqrt3_v<T>
std::numbers::egamma_v<T>
std::numbers::phi_v<T>
```

In this pattern, `T` must be one of the standard floating-point types, such as `float`, `double`, or `long double`.

## Important Mathematical Constants

### Pi with `pi`

Pi is one of the best-known mathematical constants. It represents the ratio of the circumference of any circle to its diameter.

The circumference of a circle is calculated using:

\\[ C = 2\pi r \\]

The area of a circle is calculated using:

\\[ A = \pi r^2 \\]

Example:

```
#include <iostream>
#include <numbers>

int main() {
    constexpr double radius = 4.0;

    constexpr double circumference =
        2.0 * std::numbers::pi * radius;

    constexpr double area =
        std::numbers::pi * radius * radius;

    std::cout << circumference << '\n';
    std::cout << area << '\n';
}
```

The `pi` constant is widely used in geometric calculations, trigonometric functions, signal processing, and simulation.

### Euler's Number with `e`

Euler's number, represented by the mathematical symbol \\(e\\), is the base of the natural logarithm.

This constant is used in exponential growth and decay, differential equations, probability, statistics, and continuous-system modeling.

For example, an exponential growth function is commonly written as:

\\[ f(t) = e^{kt} \\]

Example:

```
#include <cmath>
#include <iostream>
#include <numbers>

int main() {
    constexpr double growthRate = 0.15;
    constexpr double time = 3.0;

    const double result =
        std::exp(growthRate * time);

    std::cout << result << '\n';
}
```

In this example, `std::exp()` is used to calculate the exponential function. The `e` constant is useful when a formula explicitly requires the exponential base itself.

### Logarithms Related to `e`

The `log2e` constant represents the base-2 logarithm of Euler's number:

\\[ \log_2(e) \\]

The `log10e` constant represents the base-10 logarithm of Euler's number:

\\[ \log\_{10}(e) \\]

These values are useful when converting between logarithmic bases and in certain scientific calculations.

For example, the change-of-base relationship can be used to express the natural logarithm in terms of the base-2 logarithm:

\\[ \ln(x) = \log_2(x) \times \ln(2) \\]

Alternatively:

\\[ \log_2(x) = \ln(x) \times \log_2(e) \\]

In practice, standard `<cmath>` functions such as `std::log()`, `std::log2()`, and `std::log10()` should be used to calculate logarithms. The constants in `std::numbers` are not replacements for these functions.

### Natural Logarithms of Two and Ten

The `ln2` constant provides the natural logarithm of two, while `ln10` provides the natural logarithm of ten.

These values are useful in algorithm analysis, information theory, logarithmic conversions, and certain numerical algorithms.

For example:

\\[ \ln(10x) = \ln(10) + \ln(x) \\]

In such expressions, `std::numbers::ln10` can be used.

### Square Roots of Two and Three

The `sqrt2` constant provides the square root of two, while `sqrt3` provides the square root of three.

These values are used in geometry, vector calculations, Euclidean distance, 3D graphics, and numerical algorithms.

For example, the diagonal of a square with side length \\(a\\) is:

\\[ d = a\sqrt{2} \\]

Example:

```
#include <iostream>
#include <numbers>

int main() {
    constexpr double side = 10.0;

    constexpr double diagonal =
        side * std::numbers::sqrt2;

    std::cout << diagonal << '\n';
}
```

### Reciprocal Constants

The library provides the constants `inv_pi`, `inv_sqrtpi`, and `inv_sqrt3`.

These names refer, respectively, to the reciprocal of pi, the reciprocal of the square root of pi, and the reciprocal of the square root of three.

For example:

\\[ \text{inv\\\_pi} = \frac{1}{\pi} \\]

And:

\\[ \text{inv\\\_sqrtpi} = \frac{1}{\sqrt{\pi}} \\]

These values can be used in probability formulas, statistical distributions, signal processing, and scientific calculations.

Having these constants available means that when a formula directly requires one of these values, there is no need to calculate or define it manually at every point of use.

### The Euler–Mascheroni Constant with `egamma`

The `egamma` constant is represented by the symbol \\(\gamma\\) and refers to the Euler–Mascheroni constant.

This value appears in number theory, asymptotic analysis, special mathematical functions, and certain areas of mathematical analysis.

It should not be confused with Euler's number, `e`. These are two different constants with different applications.

### The Golden Ratio with `phi`

The `phi` constant represents the golden ratio:

\\[ \varphi = \frac{1 + \sqrt{5}}{2} \\]

This value is used in geometry, the Fibonacci sequence, design, and certain mathematical problems.

It can be used in code as follows:

```
#include <iostream>
#include <numbers>

int main() {
    constexpr double goldenRatio =
        std::numbers::phi;

    std::cout << goldenRatio << '\n';
}
```

## Using Template Constants

One of the important features of `std::numbers` is that it provides template versions of its constants.

### Difference Between the Regular and Template Versions

The following constant has type `double`:

```
constexpr double value = std::numbers::pi;
```

If a `float` value is needed, the template version can be used:

```
constexpr float value = std::numbers::pi_v<float>;
```

Similarly, for calculations using `long double`:

```
constexpr long double value =
    std::numbers::pi_v<long double>;
```

This feature is particularly important in generic code and libraries that work with multiple floating-point types.

### Why Does the Data Type Matter?

In floating-point calculations, the data type affects precision, representable range, and computational cost.

Common types include:

| Type          | General characteristics                                                                        |
| ------------- | ---------------------------------------------------------------------------------------------- |
| `float`       | Less precision and storage space than `double` in common implementations                       |
| `double`      | A common choice for most general-purpose calculations                                          |
| `long double` | May provide greater precision or range; its exact characteristics depend on the implementation |

Using a template version of a constant ensures that its type matches the requested type and avoids unnecessary implicit conversions.

However, `long double` should not be assumed to have greater precision than `double` on every platform. This depends on the implementation.

### Comparing Data Types

```
#include <iostream>
#include <numbers>
#include <type_traits>

int main() {
    constexpr auto piFloat =
        std::numbers::pi_v<float>;

    constexpr auto piDouble =
        std::numbers::pi_v<double>;

    constexpr auto piLongDouble =
        std::numbers::pi_v<long double>;

    static_assert(
        std::is_same_v<decltype(piFloat), const float>
    );

    static_assert(
        std::is_same_v<decltype(piDouble), const double>
    );

    static_assert(
        std::is_same_v<decltype(piLongDouble), const long double>
    );

    std::cout << piFloat << '\n';
    std::cout << piDouble << '\n';
    std::cout << piLongDouble << '\n';
}
```

In this example, `auto` deduces the type of each variable from the corresponding constant. The `static_assert` statements check the deduced types at compile time.

## Using `constexpr` with `std::numbers`

All constants in `std::numbers` can be used in constant expressions.

This means they can be used in contexts where a value must be known at compile time.

For example:

```
#include <array>
#include <numbers>

constexpr double radius = 3.0;

constexpr double area =
    std::numbers::pi * radius * radius;

constexpr std::array<double, 3> values = {
    std::numbers::pi,
    std::numbers::e,
    std::numbers::sqrt2
};

int main() {
    return area > 0.0 && values[0] > 3.0 ? 0 : 1;
}
```

In this example, the value of `area` and the elements of the array can be evaluated at compile time.

### Using Constants in Templates

`constexpr` values can be used in contexts that require constant expressions.

For example:

```
#include <numbers>

template <double Value>
struct ConstantHolder {
    static constexpr double value = Value;
};

using PiHolder = ConstantHolder<std::numbers::pi>;

int main() {
    static_assert(PiHolder::value > 3.14);
}
```

This example uses a non-type template parameter of type `double`, a capability supported since C++20.

### Using Constants in `static_assert`

Simple relationships can be checked at compile time using constants from this library:

```
#include <numbers>

static_assert(std::numbers::pi > 3.14);
static_assert(std::numbers::sqrt2 > 1.4);

int main() {
}
```

This approach is useful for checking mathematical assumptions in template code and numerical libraries.

## Using Mathematical Functions and `cmath`

The `std::numbers` library provides mathematical constants, while the standard `<cmath>` header provides mathematical functions.

These two facilities complement each other.

For example, `std::sin()` should be used to calculate the sine of an angle, while `std::numbers::pi` can be used to convert degrees to radians.

```
#include <cmath>
#include <iostream>
#include <numbers>

int main() {
    constexpr double degrees = 90.0;

    constexpr double radians =
        degrees * std::numbers::pi / 180.0;

    const double result = std::sin(radians);

    std::cout << result << '\n';
}
```

In this example, the angle is converted to radians because standard trigonometric functions such as `std::sin()` accept angles in radians.

### Converting Degrees to Radians

The conversion from degrees to radians is expressed as follows:

\\[ \text{radians} = \text{degrees} \times \frac{\pi}{180} \\]

A helper function can be defined to avoid repeating the formula:

```
#include <numbers>

constexpr double degreesToRadians(double degrees) {
    return degrees * std::numbers::pi / 180.0;
}

int main() {
    constexpr double angle =
        degreesToRadians(180.0);

    static_assert(angle > 3.14);
}
```

### Converting Radians to Degrees

The inverse relationship is:

\\[ \text{degrees} = \text{radians} \times \frac{180}{\pi} \\]

Example implementation:

```
#include <numbers>

constexpr double radiansToDegrees(double radians) {
    return radians * 180.0 / std::numbers::pi;
}

int main() {
    constexpr double angle =
        radiansToDegrees(std::numbers::pi);

    static_assert(angle > 179.0);
}
```

## Designing Generic Functions with Mathematical Constants

In modern library development, functions should generally work with different data types whenever practical.

The template versions in `std::numbers` make it possible to match a required constant to the type used in a calculation.

For example:

```
#include <numbers>
#include <type_traits>

template <typename T>
constexpr T circleArea(T radius) {
    static_assert(std::is_floating_point_v<T>);

    return std::numbers::pi_v<T> * radius * radius;
}

int main() {
    constexpr float areaFloat = circleArea(2.0f);
    constexpr double areaDouble = circleArea(2.0);
    constexpr long double areaLongDouble =
        circleArea(2.0L);

    static_assert(areaFloat > 12.0f);
    static_assert(areaDouble > 12.0);
    static_assert(areaLongDouble > 12.0L);
}
```

In this example, the function is designed for floating-point types and selects a matching version of pi based on the input type.

Using `static_assert` ensures that an error is reported at compile time if an unsuitable type is passed to the function.

### Constraining Types with Concepts

In C++20, Concepts can be used to express type constraints.

```
#include <concepts>
#include <numbers>

template <std::floating_point T>
constexpr T circleArea(T radius) {
    return std::numbers::pi_v<T> * radius * radius;
}

int main() {
    constexpr double area = circleArea(3.0);
    static_assert(area > 28.0);
}
```

This implementation uses `std::floating_point` from the `<concepts>` header to specify that the input type must be a floating-point type.

This approach expresses generic programming constraints more clearly than manually checking the type with `static_assert`.

## Practical Applications in Real Projects

### Geometric Calculations

Pi and square roots are widely used in calculations involving circles, spheres, cylinders, and other geometric shapes.

For example, the volume of a sphere is calculated using:

\\[ V = \frac{4}{3}\pi r^3 \\]

Example:

```
#include <numbers>

constexpr double sphereVolume(double radius) {
    return (4.0 / 3.0) *
           std::numbers::pi *
           radius * radius * radius;
}

int main() {
    constexpr double volume = sphereVolume(2.0);
    static_assert(volume > 33.0);
}
```

### Computer Graphics

In 2D and 3D graphics, common applications of pi and square roots include angle conversions, object rotation, geometric calculations, and coordinate generation.

For example, the coordinates of a point on a circle can be calculated as follows:

\\[ x = r\cos(\theta) \\]

\\[ y = r\sin(\theta) \\]

Example:

```
#include <cmath>
#include <numbers>
#include <utility>

std::pair<double, double> pointOnCircle(
    double radius,
    double angleRadians
) {
    return {
        radius * std::cos(angleRadians),
        radius * std::sin(angleRadians)
    };
}

int main() {
    const auto point = pointOnCircle(
        5.0,
        std::numbers::pi / 4.0
    );
}
```

### Signal Processing

In signal processing, formulas involving trigonometric functions, angular frequency, and Fourier transforms are very common.

Angular frequency is expressed as:

\\[ \omega = 2\pi f \\]

In this equation, \\(f\\) is the frequency in hertz, and \\(\omega\\) is the angular frequency in radians per second.

Example:

```
#include <numbers>

constexpr double angularFrequency(double frequency) {
    return 2.0 * std::numbers::pi * frequency;
}

int main() {
    constexpr double omega = angularFrequency(50.0);
    static_assert(omega > 314.0);
}
```

### Statistical and Scientific Calculations

The constants `pi`, `e`, and `inv_sqrtpi` appear in some formulas involving statistical distributions and special functions.

For example, the probability density function of the standard normal distribution is:

\\[ f(x) = \frac{1}{\sqrt{2\pi}}e^{-x^2/2} \\]

In applications like this, using standard constants can make formulas more readable. However, implementing a complete statistical algorithm requires additional numerical considerations.

## Differences Between `std::numbers` and Older Approaches

### Comparison with Manual Definitions

| Feature                | Manual definitions                                     | `std::numbers`                            |
| ---------------------- | ------------------------------------------------------ | ----------------------------------------- |
| Standardization        | Depends on project code                                | Part of C++20                             |
| Naming                 | Chosen by the programmer                               | Standardized names                        |
| Type-specific versions | Must be defined separately                             | Provides variable templates               |
| `constexpr` support    | Depends on how the constant is defined                 | Provided as part of the standard facility |
| Maintenance            | Potential for repeated definitions and inconsistencies | Shared use throughout the project         |
| Readability            | Depends on the chosen name                             | Uses recognized mathematical names        |

### Comparison with `M_PI` Macros

| Feature                | `M_PI`                                                      | `std::numbers::pi`     |
| ---------------------- | ----------------------------------------------------------- | ---------------------- |
| Standardization in C++ | Not guaranteed                                              | Standard since C++20   |
| Type                   | Usually a numeric value with an implementation-defined type | `double`               |
| Namespace              | Macro, with no namespace                                    | `std::numbers`         |
| Template version       | Not available                                               | `pi_v<T>`              |
| Environment dependency | May be required                                             | Requires C++20 support |

### Comparison with C Constants and Other Libraries

Some C libraries and external mathematical libraries provide similar constants. These libraries may be necessary for backward compatibility, specialized functionality, or particular platforms.

However, if a project uses C++20 and only needs the standard constants provided by this collection, `std::numbers` is the more natural choice.

## Limitations and Important Considerations

### Limited Set of Constants

`std::numbers` provides a collection of well-known mathematical constants, not every possible mathematical constant.

If a project requires more specialized constants, such as Catalan's constant, Apéry's constant, or values specific to a particular field, they must be obtained from another suitable source or defined within the project.

### Dependence on C++20 Support

This header was introduced in C++20. Therefore, both the compiler and the Standard Library being used must support this functionality.

Simply installing a recent compiler is not sufficient; the accompanying standard library implementation must also provide the required functionality.

For example, the language standard can be enabled through the following compiler settings:

```
g++ -std=c++20 main.cpp -o app
```

Or, with Clang:

```
clang++ -std=c++20 main.cpp -o app
```

With Visual C++, the following option can be used:

```
cl /std:c++20 /EHsc main.cpp
```

Exact support depends on the compiler version and Standard Library implementation.

### Floating-Point Precision

Using a standard constant does not eliminate floating-point errors.

The `float`, `double`, and `long double` types have limitations in precision and representation. Mathematical operations on these values may introduce rounding errors.

Therefore, in precision-sensitive calculations, exact equality between two floating-point values should not be assumed without considering the requirements of the problem.

For example, the following pattern is not always suitable for comparing floating-point calculation results:

```
if (result == expected) {
    // Exact equality
}
```

In many applications, it is better to compare the difference between two values using an appropriate tolerance:

```
#include <cmath>
#include <numbers>

bool approximatelyEqual(double a, double b) {
    constexpr double tolerance = 1e-10;
    return std::abs(a - b) <= tolerance;
}

int main() {
    const double value = std::sin(std::numbers::pi);

    const bool isApproximatelyZero =
        approximatelyEqual(value, 0.0);
}
```

The tolerance must be selected according to the scale of the data, the algorithm, and the required precision. The fixed value shown above is only an educational example and is not suitable for every problem.

### Differences in Precision Between Data Types

A template constant provides its value using the requested type, but this does not mean that it offers unlimited precision.

For example, `pi_v<float>` cannot preserve all the precision representable by `long double`.

Therefore, choose the constant type according to the type used in the calculation and avoid unnecessary conversions between different types.

### Implementation Differences

The exact characteristics of floating-point types, especially `long double`, may differ between platforms and compilers.

You should also not assume that the textual representation of constants will be exactly the same in every environment. Output formatting depends on the conversion of numbers to strings and on stream settings.

## Common Mistakes

### Forgetting the Header

The `<numbers>` header must be included to use constants from this library.

```
#include <numbers>
```

Without this header, the compiler will not have access to the declarations in `std::numbers`.

### Using the Wrong Namespace

The constants are defined in the `std::numbers` namespace, not directly in `std`.

Therefore, the correct form is:

```
std::numbers::pi
```

The following form is incorrect:

```
std::pi
```

### Using a Template Name Without Specifying a Type

The desired type must be specified when using a variable template:

```
std::numbers::pi_v<float>
```

Writing `std::numbers::pi_v` without specifying a type is not the intended way to use this template.

### Mistaking `pi_v` for a Function

The `pi_v<T>` member is a variable template, not a function template.

It is therefore used as a variable:

```
constexpr float value = std::numbers::pi_v<float>;
```

It is not used as a function call.

### Using a Constant That Does Not Match the Calculation Type

If an algorithm is designed around `float`, using `pi_v<float>` can be a clearer choice than using `pi` and relying on an implicit conversion.

However, the language's type conversion rules still apply in many expressions, so the appropriate type should be selected with the entire expression in mind.

### Confusing Euler's Number with the Euler–Mascheroni Constant

`e` and `egamma` are two entirely different constants.

- `e` is the base of the natural logarithm.
- `egamma` is the Euler–Mascheroni constant.

Confusing these two values can lead to errors in scientific formulas.

### Assuming All Numeric Types Are Supported

The variable templates in this library are designed for standard floating-point types. You should not assume that they can be used directly with integral types, custom types, or arbitrary numeric classes.

If a project uses a custom numeric type, an appropriate mechanism must be designed to provide the constants required by that type.

## Design Considerations for Professional Developers

### Choosing a Constant Type Based on the Calculation Type

When designing numerical APIs, it is preferable for the types of constants to match the primary data type used by the algorithm.

For example, a template function can use `pi_v<T>` so that the type of the constant follows the type of the function parameter.

### Avoiding Redefinition of Standard Constants

If a required constant is already available in `std::numbers`, redefining it under a different name is usually unnecessary.

Using the standard constant directly improves readability and consistency throughout the project.

### Using Fully Qualified Names in Library Code

In small projects, using `using namespace std::numbers;` may be tempting. However, in public headers and libraries, it is better to use the fully qualified name:

```
constexpr double area =
    std::numbers::pi * radius * radius;
```

This avoids namespace pollution and potential name conflicts.

### Paying Attention to Unit Conventions

The `pi` constant alone does not indicate whether an API expects angles in degrees or radians.

Since standard trigonometric functions use radians, unit conventions should be made clear through function names, documentation, and data types.

In larger projects, Strong Types or Units libraries can be used to prevent unintended mixing of degrees and radians.

### Considering Numerical Stability

Having more accurate constants does not, by itself, make an algorithm numerically stable.

For example, calculations involving the subtraction of two very close values may suffer from cancellation. In such cases, the appropriate algorithm should be selected rather than simply increasing the number of digits in a constant.

## Standardization and Compatibility

The `std::numbers` facility was added to the C++20 Standard Library. It is not a new language feature in the sense of a keyword or new syntax; rather, it is a standard library feature.

Therefore, three aspects should be distinguished:

- Language version: The project must be compiled using C++20 or a later standard.
- Library implementation: The Standard Library provided with the compiler must supply the `<numbers>` header and its members.
- Build configuration: The language standard must be enabled in the build system.

For CMake-based projects, the required standard can be specified as follows:

```
cmake_minimum_required(VERSION 3.20)

project(NumbersExample LANGUAGES CXX)

add_executable(numbers_example main.cpp)

target_compile_features(numbers_example PRIVATE cxx_std_20)
```

This setting specifies the required language standard for the target. However, the compiler version and Standard Library must still support the functionality.

## Summary

The `std::numbers` library is a useful feature of the C++20 Standard Library that provides a collection of commonly used mathematical constants in a standardized and type-safe form.

Its most important features include:

- Providing constants such as `pi`, `e`, `sqrt2`, and `phi`.
- Providing `constexpr` constants for compile-time use.
- Supporting variable templates such as `pi_v<T>`.
- Allowing the type of a constant to match the type used in floating-point calculations.
- Eliminating the need to manually define many commonly used constants.
- Reducing dependence on non-standard macros such as `M_PI`.

To use this functionality, simply include the `<numbers>` header and access the desired constant through the `std::numbers` namespace.

In generic code and numerical libraries, using template versions such as `std::numbers::pi_v<T>` is generally a suitable choice because it matches the constant's type to the type used in the calculation.

Finally, keep in mind that `std::numbers` is not a replacement for the mathematical functions in `<cmath>` or for specialized numerical algorithms. This library only provides standard mathematical constants, while the final precision of calculations still depends on the data type, the algorithm, and the characteristics of the implementation.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>