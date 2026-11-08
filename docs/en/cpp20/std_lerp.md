<div align="center">

[🇺🇸 English](./std_lerp.md) | [🇮🇷 فارسی](../../fa/cpp20/std_lerp.md)

</div>

---

# Introduction to `std::lerp` in C++20

## Introduction

Calculating a value between two points is one of the most common operations in programming. It is widely used in areas such as Computer Graphics, Game Development, Animation, Signal Processing, Physics Simulation, Numerical Computing, and User Interface development.

One of the most common techniques for calculating a value between two values is called Linear Interpolation, commonly abbreviated as `Lerp`.

In C++20, the standard library introduced `std::lerp` as a standardized function for performing Linear Interpolation. This function is declared in the `<cmath>` header and provides important numerical properties, including Monotonicity and protection against certain Overflow scenarios.

Before this function was introduced, developers typically implemented Interpolation using manually written formulas. Although these formulas work correctly in many simple situations, they can encounter problems in Floating-Point calculations, including Overflow, loss of precision, and violations of Monotonicity.

`std::lerp` was designed to address these considerations by providing a standardized, reusable, and more numerically reliable approach to Linear Interpolation.

## Understanding Linear Interpolation

### Mathematical Definition

Linear Interpolation calculates a value between two specified values. Suppose we have two values, `a` and `b`, and want to calculate the value corresponding to a relative position `t`.

The mathematical formula is:

\\[ \operatorname{lerp}(a,b,t)=a+t(b-a) \\]

The parameters represent:

- `a`: The starting value or Start Value.
- `b`: The ending value or End Value.
- `t`: The Interpolation parameter that specifies the relative position.
- Result: The calculated value between the two values or outside their range.

The most important cases are:

| Value of `t` | Result                              | Meaning                        |
| ------------ | ----------------------------------- | ------------------------------ |
| `0`          | `a`                                 | Starting value                 |
| `0.25`       | 25% of the distance from `a` to `b` | One-quarter of the distance    |
| `0.5`        | Midpoint between `a` and `b`        | Halfway point                  |
| `0.75`       | 75% of the distance from `a` to `b` | Three-quarters of the distance |
| `1`          | `b`                                 | Ending value                   |

For example, if the starting value is `10`, the ending value is `30`, and the Interpolation parameter is `0.5`, the result is `20`.

\\[ 10+0.5(30-10)=20 \\]

### Understanding the Interpolation Parameter

The `t` parameter specifies what proportion of the distance between two values should be traversed.

An important detail is that `t` does not necessarily have to be between zero and one.

When `t` belongs to the interval `[0, 1]`, the operation performs Interpolation within the range defined by the two values.

However, when `t` falls outside this interval, the operation becomes Extrapolation.

For example:

\\[ \operatorname{lerp}(10,30,1.5)=40 \\]

In this example, the calculation extends beyond the ending point.

Similarly:

\\[ \operatorname{lerp}(10,30,-0.5)=0 \\]

In this case, the result lies before the starting point.

Therefore, Interpolation and Extrapolation use the same mathematical formula. The difference is the range of the `t` parameter.

## Introducing std::lerp

### Declaration and Required Standard Version

`std::lerp` is a C++20 standard library feature.

To use this function, include the following header:

```
#include <cmath>
```

Its primary signature is:

```
namespace std {

    template<class T>
    constexpr T lerp(T a, T b, T t) noexcept;

}
```

The main characteristics of this function include:

- Standard: Available since C++20.
- Header: `<cmath>`.
- Namespace: `std`.
- Function type: Function Template.
- Return type: The same type as `T`.
- Parameter types: Three arguments of the same type `T`.
- constexpr: Can be used in Constant Expressions when the applicable requirements are satisfied.
- noexcept: The function guarantees that it does not throw exceptions.

The type `T` must be a Floating-Point Type. Therefore, types such as `float`, `double`, and `long double` are suitable.

### Basic Example

The following example uses `std::lerp` to calculate the midpoint between two numbers:

```
#include <cmath>
#include <iostream>

int main() {
    double start = 10.0;
    double end = 30.0;
    double t = 0.5;

    double result = std::lerp(start, end, t);

    std::cout << result << '\n';
}
```

Output:

```
20
```

In this example, the function calculates the midpoint of the range from `10.0` to `30.0`.

### Examining Different Values

The following example demonstrates how the function behaves at different positions:

```
#include <cmath>
#include <iomanip>
#include <iostream>

int main() {
    constexpr double start = 10.0;
    constexpr double end = 30.0;

    for (int i = 0; i <= 10; ++i) {
        double t = i / 10.0;

        double value = std::lerp(start, end, t);

        std::cout << std::fixed
                  << std::setprecision(2)
                  << "t = " << t
                  << ", value = " << value
                  << '\n';
    }
}
```

Output:

```
t = 0.00, value = 10.00
t = 0.10, value = 12.00
t = 0.20, value = 14.00
t = 0.30, value = 16.00
t = 0.40, value = 18.00
t = 0.50, value = 20.00
t = 0.60, value = 22.00
t = 0.70, value = 24.00
t = 0.80, value = 26.00
t = 0.90, value = 28.00
t = 1.00, value = 30.00
```

In this example, the `t` parameter gradually increases, and the output moves linearly from the starting value to the ending value.

## Differences Between std::lerp and Manual Formulas

### A Typical Linear Interpolation Implementation

Before C++20, developers commonly implemented Linear Interpolation manually.

The conventional formula looks like this:

```
double lerp(double a, double b, double t) {
    return a + t * (b - a);
}
```

This implementation is mathematically correct, but it is not always the best choice for Floating-Point calculations.

### The Overflow Problem

One important limitation of the conventional formula is the possibility of Overflow when calculating the difference between two values.

Consider the following example:

```
#include <iostream>
#include <limits>

int main() {
    const double max =
        std::numeric_limits<double>::max();

    double a = -max;
    double b = max;
    double t = 0.5;

    double result = a + t * (b - a);

    std::cout << result << '\n';
}
```

Mathematically, the expected result is zero.

However, the expression:

```
b - a
```

exceeds the representable range of `double` in this example and may produce Infinity.

Consequently, the manual formula cannot guarantee the expected result.

In contrast, `std::lerp` was designed with Floating-Point limitations in mind. For finite values of `a` and `b`, it guarantees a finite result when `t` is between zero and one.

Therefore, `std::lerp` is a more appropriate choice in these situations.

### The Monotonicity Problem

Monotonicity means that as the `t` parameter increases, the result must not move in the opposite direction.

For example, if `a` is smaller than `b`, we expect the output not to decrease as `t` increases.

In manual implementations, Floating-Point rounding errors can sometimes violate this property.

`std::lerp` provides a Monotonicity guarantee for finite inputs under the specified standard conditions.

More precisely, when `a` and `b` are finite and `t` belongs to `[0, 1]`, the result behaves monotonically with respect to `t`:

- If `a < b`, the result is non-decreasing.
- If `a > b`, the result is non-increasing.

This property is particularly important in Animation, numerical calculations, and systems that require continuous and predictable movement.

### Comparing the Two Approaches

| Feature                | Manual Formula                                     | `std::lerp`                                           |
| ---------------------- | -------------------------------------------------- | ----------------------------------------------------- |
| Linear Interpolation   | Yes                                                | Yes                                                   |
| Standardized           | Custom implementation                              | C++20                                                 |
| Endpoint guarantees    | Depends on implementation                          | Yes                                                   |
| Monotonicity guarantee | No                                                 | Yes, under specified conditions                       |
| Overflow protection    | Depends on implementation                          | Guarantees a finite result within the specified range |
| constexpr support      | Depends on the implementation and language version | Yes                                                   |
| Exception-free         | Depends on the function                            | Yes                                                   |

It is important to understand that `std::lerp` is not a replacement for every Interpolation algorithm. It is specifically designed for Linear Interpolation between Floating-Point values.

## Numerical Properties and Standard Guarantees

### Endpoint Guarantees

One important property of `std::lerp` is that it returns the starting and ending values exactly when the Interpolation parameter is zero or one.

```
#include <cassert>
#include <cmath>

int main() {
    constexpr double a = 10.5;
    constexpr double b = 99.75;

    static_assert(std::lerp(a, b, 0.0) == a);
    static_assert(std::lerp(a, b, 1.0) == b);
}
```

This guarantee is important because Floating-Point calculations in manual implementations can introduce unwanted differences at the endpoints.

For example, in an Animation system, we may expect the value to be exactly equal to the Target Value when the specified duration has elapsed.

### Preserving the Value When Both Inputs Are Equal

When the starting and ending values are equal, the result must also be equal to that value:

\\[ \operatorname{lerp}(a,a,t)=a \\]

For example:

```
#include <cmath>
#include <iostream>

int main() {
    double value = std::lerp(42.0, 42.0, 0.75);

    std::cout << value << '\n';
}
```

Output:

```
42
```

This behavior also applies when `t` is outside the `[0, 1]` interval.

### Finite Result Guarantee

One of the most important numerical properties of this function is its finite-result guarantee under specified conditions.

If both `a` and `b` are finite and the `t` parameter belongs to the following interval:

\\[ 0 \leq t \leq 1 \\]

the result of `std::lerp` is also finite.

This property is important in calculations involving very large or very small values.

However, this guarantee does not imply unlimited precision or eliminate all Floating-Point errors. The result is still calculated and rounded using the destination Floating-Point type.

### Symmetry Property

The following mathematical relationship holds:

\\[ \operatorname{lerp}(a,b,t) = \operatorname{lerp}(b,a,1-t) \\]

This relationship means that moving from `a` to `b` using parameter `t` is mathematically equivalent to moving from `b` to `a` using parameter `1-t`.

For example:

```
#include <cmath>
#include <iostream>

int main() {
    double a = 10.0;
    double b = 50.0;
    double t = 0.25;

    double forward = std::lerp(a, b, t);
    double backward = std::lerp(b, a, 1.0 - t);

    std::cout << forward << '\n';
    std::cout << backward << '\n';
}
```

Output:

```
20
20
```

Keep in mind that this relationship holds in exact mathematics, but Floating-Point rounding and differences in evaluation paths mean that bitwise equality should not be expected for every possible input.

### Monotonicity Guarantee Within the Interpolation Range

An important property of `std::lerp` is that it preserves the direction of movement between the two endpoints.

For example, consider the following code:

```
#include <cassert>
#include <cmath>

int main() {
    double a = 100.0;
    double b = 200.0;

    double previous = std::lerp(a, b, 0.0);

    for (int i = 1; i <= 100; ++i) {
        double t = i / 100.0;

        double current = std::lerp(a, b, t);

        assert(current >= previous);

        previous = current;
    }
}
```

In this example, the output must not decrease as `t` increases.

When the direction of movement is reversed, meaning `a` is greater than `b`, the Monotonicity behavior is reversed as well.

## Using constexpr

### Using Constant Expressions

`std::lerp` has been declared `constexpr` since C++20.

This makes it possible to use the function in compile-time calculations.

```
#include <cmath>

constexpr double start = 10.0;
constexpr double end = 50.0;

constexpr double middle =
    std::lerp(start, end, 0.5);

static_assert(middle == 30.0);
```

In this example, the midpoint calculation can be performed during compilation.

Using `constexpr` is useful for constant calculations, Lookup Tables, and compile-time configuration values.

### Using constexpr Functions

You can also use `std::lerp` inside a `constexpr` function:

```
#include <cmath>

constexpr double interpolate(
    double start,
    double end,
    double progress
) {
    return std::lerp(start, end, progress);
}

static_assert(interpolate(0.0, 100.0, 0.25) == 25.0);
static_assert(interpolate(0.0, 100.0, 0.75) == 75.0);
```

However, all inputs and operations must satisfy the requirements for Constant Expressions before the function can be evaluated at compile time.

## Understanding Data Types and Type Deduction

### Supported Floating-Point Types

`std::lerp` is designed for Floating-Point types.

The following examples are valid:

```
#include <cmath>

int main() {
    float f = std::lerp(0.0f, 100.0f, 0.5f);

    double d = std::lerp(0.0, 100.0, 0.5);

    long double ld = std::lerp(
        0.0L,
        100.0L,
        0.5L
    );
}
```

In all three examples, the parameter types are identical.

### Problems with Mixing Different Types

Since `std::lerp` is a Function Template, the type `T` is deduced from its arguments.

Consequently, directly mixing different types may cause Template Argument Deduction to fail.

For example:

```
#include <cmath>

int main() {
    double result = std::lerp(
        0.0f,
        100.0,
        0.5
    );
}
```

This code is invalid because the argument types differ during Template Argument Deduction.

To solve this problem, convert all inputs to a common type:

```
#include <cmath>

int main() {
    double start = 0.0f;
    double end = 100.0;
    double progress = 0.5f;

    double result = std::lerp(
        start,
        end,
        progress
    );
}
```

Another approach is to use `static_cast`:

```
#include <cmath>

int main() {
    float start = 10.0f;
    double end = 100.0;
    float progress = 0.5f;

    double result = std::lerp(
        static_cast<double>(start),
        end,
        static_cast<double>(progress)
    );
}
```

In larger applications, selecting a single Floating-Point type for the entire calculation generally improves readability and predictability.

### Using Integer Types

`std::lerp` is not designed for Integer values.

For example:

```
#include <cmath>

int main() {
    auto result = std::lerp(10, 20, 0);
}
```

This usage is invalid because the deduced type is `int`, whereas `T` must be a Floating-Point type.

To perform Interpolation on integer values, first perform the calculation using a Floating-Point type and then convert the result to an Integer if necessary:

```
#include <cmath>
#include <iostream>

int main() {
    int start = 10;
    int end = 20;

    double progress = 0.5;

    double interpolated = std::lerp(
        static_cast<double>(start),
        static_cast<double>(end),
        progress
    );

    int result = static_cast<int>(interpolated);

    std::cout << result << '\n';
}
```

Output:

```
15
```

Keep in mind that converting a Floating-Point value to an Integer can discard its fractional part. This conversion does not automatically round to the nearest integer.

## Practical Applications of std::lerp

### Animation

One of the most common applications of Linear Interpolation is moving an Object from a starting position to an ending position.

Suppose an Object must move from position zero to position one hundred over two seconds.

In this situation, `std::lerp` can calculate its current position:

```
#include <cmath>
#include <iostream>

int main() {
    constexpr double startPosition = 0.0;
    constexpr double endPosition = 100.0;

    constexpr double duration = 2.0;

    double elapsedTime = 0.75;

    double progress = elapsedTime / duration;

    double currentPosition = std::lerp(
        startPosition,
        endPosition,
        progress
    );

    std::cout << currentPosition << '\n';
}
```

Output:

```
37.5
```

In this example, after 0.75 seconds of a two-second Animation, `progress` is equal to `0.375`.

Therefore, the Object has traveled 37.5% of the total distance.

An important consideration is that in a real Animation system, `progress` should generally be constrained to the `[0, 1]` interval:

```
#include <algorithm>
#include <cmath>

double calculatePosition(
    double start,
    double end,
    double elapsed,
    double duration
) {
    double progress = elapsed / duration;

    progress = std::clamp(progress, 0.0, 1.0);

    return std::lerp(start, end, progress);
}
```

This implementation prevents unwanted Extrapolation after the Animation has completed.

Also, `duration` must not be zero, and the time inputs must contain valid values.

### Game Development

In games, many Object properties change gradually over time.

Examples include:

- Character position
- Object movement speed
- Health or Energy values
- Light intensity
- Transparency
- Camera Zoom
- Audio source volume

`std::lerp` can be used to calculate these values.

The following example gradually changes light intensity between two values:

```
#include <cmath>
#include <iostream>

int main() {
    double minimumIntensity = 0.2;
    double maximumIntensity = 1.0;

    double transitionProgress = 0.65;

    double intensity = std::lerp(
        minimumIntensity,
        maximumIntensity,
        transitionProgress
    );

    std::cout << intensity << '\n';
}
```

Output:

```
0.72
```

This approach is suitable for implementing linear changes in Real-Time systems.

### Color Transitions

In Graphics systems, gradually changing colors is another common application of Interpolation.

Suppose we want to transition a color from red to blue.

Each RGB color consists of three Components:

- Red
- Green
- Blue

Each Component can be interpolated independently:

```
#include <cmath>
#include <iostream>

struct Color {
    double r;
    double g;
    double b;
};

Color interpolateColor(
    const Color& start,
    const Color& end,
    double t
) {
    return {
        std::lerp(start.r, end.r, t),
        std::lerp(start.g, end.g, t),
        std::lerp(start.b, end.b, t)
    };
}

int main() {
    Color red{1.0, 0.0, 0.0};
    Color blue{0.0, 0.0, 1.0};

    Color result = interpolateColor(red, blue, 0.5);

    std::cout << result.r << ' '
              << result.g << ' '
              << result.b << '\n';
}
```

Output:

```
0.5 0 0.5
```

In this example, the resulting color is calculated by linearly interpolating the RGB Components.

However, professional Graphics applications must also consider the Color Space. Direct Interpolation in RGB space does not necessarily produce the best perceptual or physically accurate results.

In some applications, performing Interpolation in Linear RGB or other color spaces may be more appropriate.

### Camera Zoom

In Camera systems, the Zoom value can be changed gradually.

```
#include <cmath>
#include <iostream>

int main() {
    double initialZoom = 1.0;
    double targetZoom = 2.5;

    double progress = 0.4;

    double currentZoom = std::lerp(
        initialZoom,
        targetZoom,
        progress
    );

    std::cout << currentZoom << '\n';
}
```

Output:

```
1.6
```

This approach is suitable for implementing linear Zoom transitions over a specified time interval.

### Signal Processing

In Digital Signal Processing, Interpolation can be used to estimate signal values between two Samples.

Suppose we have two consecutive Samples:

\\[ x_0=2.0 \\]

\\[ x_1=6.0 \\]

To estimate the value at a relative position of `0.25`, we can use:

\\[ x=\operatorname{lerp}(x_0,x_1,0.25) \\]

The result is `3.0`.

This is an example of Linear Interpolation in signal processing.

However, applications requiring higher quality may benefit from methods such as Cubic Interpolation or Spline Interpolation.

## Differences Between Interpolation and Easing

### Understanding Linear Interpolation

The `std::lerp` function performs linear movement between two values.

This means that if `t` increases at a constant rate, the output also changes at a constant rate.

For example:

```
#include <cmath>

double value = std::lerp(0.0, 100.0, 0.5);
```

In this example, the output is exactly halfway between the two endpoints.

### Understanding Easing

In Animation, linear movement does not always produce natural or desirable results.

For example, we may want movement to:

- Start slowly.
- Accelerate in the middle.
- Slow down again toward the end.

This behavior can be achieved using Easing Functions.

An important distinction is that `std::lerp` does not perform Easing by itself. However, the output of an Easing Function can be supplied as its `t` parameter.

For example:

```
#include <cmath>

double easeInOut(double t) {
    return t * t * (3.0 - 2.0 * t);
}

int main() {
    double start = 0.0;
    double end = 100.0;

    double progress = 0.5;

    double easedProgress = easeInOut(progress);

    double value = std::lerp(
        start,
        end,
        easedProgress
    );
}
```

In this example, the `progress` value is first transformed using an Easing Function, and the final value is then calculated.

Therefore, these two concepts have different responsibilities:

| Feature          | `std::lerp`                             | Easing Function                           |
| ---------------- | --------------------------------------- | ----------------------------------------- |
| Purpose          | Calculate a value between two endpoints | Change the progression rate               |
| Default behavior | Linear                                  | Depends on the function                   |
| Application      | Interpolation                           | Creating more natural movement            |
| Composability    | Can be combined with Easing             | Can be used as an Interpolation parameter |

## Differences Between std::lerp and std::midpoint

C++20 introduced two related numerical functions in the standard library: `std::lerp` and `std::midpoint`.

Although these functions are conceptually related, they serve different purposes.

### Introducing std::midpoint

The `std::midpoint` function calculates the midpoint between two values.

It is declared in the `<numeric>` header.

Example:

```
#include <iostream>
#include <numeric>

int main() {
    double a = 10.0;
    double b = 30.0;

    double middle = std::midpoint(a, b);

    std::cout << middle << '\n';
}
```

Output:

```
20
```

### The Main Difference

`std::midpoint` always calculates the midpoint between two values, whereas `std::lerp` allows you to specify an arbitrary relative position using the `t` parameter.

Mathematically, the following relationship holds:

\\[ \operatorname{midpoint}(a,b) = \operatorname{lerp}(a,b,0.5) \\]

For Floating-Point values, both functions can be used to calculate a midpoint. However, their standard guarantees and numerical behavior are not identical.

One important difference is that `std::midpoint` also supports arithmetic types other than Floating-Point, including Integer types.

For integer values, `std::midpoint` rounds toward the first argument when the exact result is not an integer.

Therefore:

- Use `std::midpoint` when you need the midpoint between two values.
- Use `std::lerp` when you need a value at an arbitrary relative position.

## Comparing Linear Interpolation with Other Methods

### Comparison with Cubic Interpolation

In Linear Interpolation, the output is calculated using a linear relationship between two endpoints.

Cubic Interpolation, on the other hand, typically uses a third-degree polynomial to calculate the result.

This approach can provide smoother transitions and greater control over the curve's slope.

Common applications include:

- Image Processing
- Animation
- Signal Processing
- Geometric Computations

Compared with Linear Interpolation, implementing Cubic Interpolation is generally more complex and may require additional information, such as neighboring points or Tangents.

### Comparison with Spline Interpolation

Spline Interpolation creates smooth curves using multiple control points.

This method is suitable when movement or value transitions should not follow a simple straight line.

For example, Spline Interpolation can provide greater control when designing Camera movement paths or complex Animations.

`std::lerp` is not a replacement for Spline Interpolation, but it can be used as part of more complex algorithms.

## Handling Extrapolation

### Behavior Outside the Zero-to-One Range

An important detail about `std::lerp` is that it does not automatically constrain `t` to the `[0, 1]` interval.

Consequently, inputs greater than one or smaller than zero result in Extrapolation.

```
#include <cmath>
#include <iostream>

int main() {
    double a = 10.0;
    double b = 20.0;

    std::cout << std::lerp(a, b, -1.0) << '\n';
    std::cout << std::lerp(a, b, 0.5) << '\n';
    std::cout << std::lerp(a, b, 2.0) << '\n';
}
```

Output:

```
0
15
30
```

This behavior is useful in applications such as linear prediction and calculations that need to extend beyond the original interval.

### Constraining t with std::clamp

If the goal is to perform Interpolation only within the original range, you can use `std::clamp`.

This function has been available since C++17 and is declared in the `<algorithm>` header.

```
#include <algorithm>
#include <cmath>
#include <iostream>

int main() {
    double a = 10.0;
    double b = 20.0;

    double t = 1.5;

    t = std::clamp(t, 0.0, 1.0);

    double result = std::lerp(a, b, t);

    std::cout << result << '\n';
}
```

Output:

```
20
```

In this example, the initial value of `t` is `1.5`, but it becomes `1.0` after clamping.

As a result, the function returns the ending value.

Keep in mind that normal use of `std::clamp` requires comparable values. If NaN may appear in the input, its handling policy should be defined separately.

## Handling NaN and Infinity Values

### Behavior with NaN

NaN, or Not a Number, is a special Floating-Point value.

It typically results from invalid mathematical operations or invalid input data.

For example:

```
#include <cmath>
#include <limits>

int main() {
    double a = 10.0;
    double b = 20.0;

    double t = std::numeric_limits<double>::quiet_NaN();

    double result = std::lerp(a, b, t);
}
```

Under these conditions, you should not expect the result to be a valid numerical value suitable for application logic.

NaN behavior must not be confused with the Monotonicity or Finite Result guarantees, because those guarantees do not apply in the same way to invalid or non-finite inputs.

If inputs are received from networks, files, or external sources, validating them before performing calculations is a good practice.

You can use `std::isfinite` and `std::isnan` for this purpose:

```
#include <cmath>
#include <stdexcept>

double safeLerp(double a, double b, double t) {
    if (!std::isfinite(a) ||
        !std::isfinite(b) ||
        !std::isfinite(t)) {
        throw std::invalid_argument("Invalid interpolation input");
    }

    return std::lerp(a, b, t);
}
```

This implementation establishes an explicit policy for preventing non-finite values and NaN from entering the calculation.

### Behavior with Infinity

Infinity values must also be handled carefully.

For example:

```
#include <cmath>
#include <limits>

int main() {
    double infinity =
        std::numeric_limits<double>::infinity();

    double result = std::lerp(
        10.0,
        infinity,
        0.5
    );
}
```

Under these conditions, the finite-result guarantee for finite endpoints and `t` within `[0, 1]` does not apply.

Therefore, if your system requires a finite result, validate the endpoints before calling the function.

## Important Considerations for Floating-Point Calculations

### Precision Limitations

Despite the numerical guarantees provided by `std::lerp`, the function is still subject to Floating-Point limitations.

Types such as `float` and `double` have a limited number of bits available for representing values.

As a result, some decimal values cannot be represented exactly.

For example:

```
#include <cmath>
#include <iostream>

int main() {
    double result = std::lerp(
        0.1,
        0.2,
        0.3
    );

    std::cout.precision(17);

    std::cout << result << '\n';
}
```

The output may be displayed as:

```
0.13
```

The exact displayed representation depends on the actual stored value and the output formatting.

For Floating-Point tests, it is generally better to use a Tolerance when appropriate:

```
#include <cmath>
#include <cassert>

int main() {
    double result = std::lerp(0.1, 0.2, 0.3);

    double expected = 0.13;

    double epsilon = 1e-12;

    assert(std::abs(result - expected) < epsilon);
}
```

The selected Tolerance should be appropriate for the scale of the values, the Floating-Point type, and the application's requirements.

### Choosing Between float and double

The appropriate data type depends on the application's requirements.

| Type          | Characteristics                          | Suitable Applications                                |
| ------------- | ---------------------------------------- | ---------------------------------------------------- |
| `float`       | Lower memory usage and limited precision | Graphics and some Real-Time systems                  |
| `double`      | Greater precision                        | General-purpose and numerical calculations           |
| `long double` | Precision depends on the implementation  | Calculations requiring potentially greater precision |

For many general-purpose applications, `double` is a suitable choice.

However, in Game Engines or large-scale processing systems, `float` may be more appropriate because of compatibility with other APIs and lower memory consumption.

An important detail is that `long double` does not necessarily provide greater precision than `double` on every platform.

### Preventing Division by Zero When Calculating t

In real-world applications, the `t` parameter is usually calculated by dividing elapsed time by the total duration.

The conventional formula is:

\\[ t=\frac{elapsedTime}{duration} \\]

If `duration` is zero, the calculation may produce an invalid result.

Therefore, check the duration before calculating `t`.

```
#include <algorithm>
#include <cmath>
#include <stdexcept>

double animate(
    double start,
    double end,
    double elapsedTime,
    double duration
) {
    if (!std::isfinite(duration) || duration <= 0.0) {
        throw std::invalid_argument("Invalid duration");
    }

    double progress = elapsedTime / duration;

    if (!std::isfinite(progress)) {
        throw std::invalid_argument("Invalid progress");
    }

    progress = std::clamp(progress, 0.0, 1.0);

    return std::lerp(start, end, progress);
}
```

In addition to validating the duration, this implementation prevents invalid Progress values from entering the calculation.

## Common Mistakes When Using std::lerp

### Using Integer Types Instead of Floating-Point Types

One common mistake is passing Integer values directly to `std::lerp`.

```
std::lerp(10, 20, 0);
```

This code is invalid because the deduced type is `int`.

Instead, use Floating-Point literals:

```
std::lerp(10.0, 20.0, 0.5);
```

### Expecting t to Be Clamped Automatically

Some developers assume that `std::lerp` automatically constrains `t` to the `[0, 1]` interval.

This assumption is incorrect.

The function also supports Extrapolation.

If clamping is required, use `std::clamp`.

### Assuming Floating-Point Errors Are Completely Eliminated

Although `std::lerp` is designed to improve numerical behavior, its calculations are still subject to Floating-Point limitations.

Therefore, unlimited mathematical precision should not be expected.

### Using It for Nonlinear Interpolation

`std::lerp` is designed for Linear Interpolation.

If the goal is to create Ease-In, Ease-Out, Bézier curves, or Splines, an appropriate algorithm for that specific problem should be used.

### Using It Without Validating Inputs

In applications that receive inputs from external sources, you should not assume that all values are valid.

In particular, Graphics, Real-Time, and Network Programming systems should have clearly defined policies for handling NaN, Infinity, and out-of-range values.

## Performance and Design Considerations

### Computational Cost

`std::lerp` is a lightweight numerical operation and is generally suitable for use in processing loops and Animations.

However, the C++ standard does not provide a direct guarantee about the number of machine instructions or the execution speed of the function.

Depending on optimization settings, processor architecture, and Floating-Point requirements, the Compiler may generate different implementations.

Therefore, if Interpolation performance is important to your application, evaluate it using Benchmarks in the actual target environment.

### Using It in Processing Loops

In applications such as Game Engines, Interpolation may be performed thousands of times per Frame.

For example:

```
#include <cmath>
#include <vector>

int main() {
    std::vector<double> positions(1000);

    for (std::size_t i = 0; i < positions.size(); ++i) {
        double t = static_cast<double>(i) /
                   static_cast<double>(positions.size() - 1);

        positions[i] = std::lerp(
            0.0,
            100.0,
            t
        );
    }
}
```

In this example, a value between zero and one hundred is calculated for each element.

In real applications, the Container size and the method used to calculate `t` must also be considered. For example, if the number of elements is zero or one, Progress calculation requires separate handling.

### Considering SIMD

`std::lerp` is not a SIMD API by itself.

Although a Compiler may Vectorize some calculations, you should not assume that calling this function always produces SIMD instructions.

For explicit vectorized processing, you can use appropriate Compiler features, Intrinsics, or SIMD libraries.

The appropriate approach should be selected according to performance requirements and code portability.

### Determinism Considerations

In systems that require Deterministic Computation, such as certain simulations and Multiplayer Game Engines, differences in Floating-Point calculations across architectures and Compiler configurations must be considered.

Using `std::lerp` alone does not guarantee that calculations will produce Bitwise Identical results across all processors and Compilers.

If strict Determinism is essential, the numerical calculation policy, data types, Floating-Point settings, and testing methods must be designed consistently.

## Differences Between C++ Standard Versions

### C++17 and Earlier

`std::lerp` is not available in C++17 or earlier versions.

In these versions, developers generally need to use a custom implementation or an external library.

For example:

```
double lerp(double a, double b, double t) {
    return a + t * (b - a);
}
```

This implementation is suitable for simple applications, but it does not provide the numerical guarantees of `std::lerp`.

### Introduction in C++20

`std::lerp` was added to the standard library in C++20.

Its most important characteristics include:

- A standardized API for Linear Interpolation.
- `constexpr` support.
- `noexcept` guarantee.
- Exact preservation of endpoints.
- Monotonicity guarantees under specified conditions.
- A finite-result guarantee for finite endpoints and `t` within `[0, 1]`.

### Status in C++23 and C++26

`std::lerp` has been part of the standard library since C++20 and remains available in subsequent standards.

Using this API does not require any C++23-specific or C++26-specific features.

Therefore, any project supporting C++20 can use this function.

## Configuring the Compiler to Use std::lerp

### Compiling with GCC

To use C++20 features with GCC, you can use the following command:

```
g++ -std=c++20 -Wall -Wextra -Wpedantic main.cpp -o main
```

### Compiling with Clang

For Clang, the following command can be used:

```
clang++ -std=c++20 -Wall -Wextra -Wpedantic main.cpp -o main
```

### Compiling with MSVC

In Visual Studio, you can configure the language standard to C++20.

You can also use the following MSVC command-line option:

```
cl /std:c++20 /EHsc /W4 main.cpp
```

An important consideration is that support for a language standard and support for its standard library features may depend on the Compiler version and the standard library implementation being used.

If you encounter compilation errors, check the Compiler version and the status of its standard library support.

## Complete Practical Example

### Implementing a Simple Animation

The following example implements a small class for managing Linear Animation.

The class provides the following features:

- Starting and ending values.
- Animation duration.
- Elapsed time.
- Progress calculation.
- Constraining Progress to a valid range.
- Calculating the current value using `std::lerp`.
- Detecting when the Animation has finished.

```
#include <algorithm>
#include <cmath>
#include <stdexcept>

class LinearAnimation {
public:
    LinearAnimation(
        double start,
        double end,
        double duration
    )
        : start_(start),
          end_(end),
          duration_(duration),
          elapsed_(0.0) {

        if (!std::isfinite(start_) ||
            !std::isfinite(end_) ||
            !std::isfinite(duration_) ||
            duration_ <= 0.0) {
            throw std::invalid_argument("Invalid animation parameters");
        }
    }

    void update(double deltaTime) {
        if (!std::isfinite(deltaTime) || deltaTime < 0.0) {
            throw std::invalid_argument("Invalid delta time");
        }

        elapsed_ = std::min(
            elapsed_ + deltaTime,
            duration_
        );
    }

    double progress() const noexcept {
        return elapsed_ / duration_;
    }

    double value() const noexcept {
        return std::lerp(
            start_,
            end_,
            progress()
        );
    }

    bool finished() const noexcept {
        return elapsed_ >= duration_;
    }

private:
    double start_;
    double end_;
    double duration_;
    double elapsed_;
};

int main() {
    LinearAnimation animation(0.0, 100.0, 2.0);

    animation.update(0.5);

    double currentValue = animation.value();

    bool isFinished = animation.finished();
}
```

In this implementation, the elapsed time increases every time `update` is called.

The `value` function then uses the current Progress to calculate the new value.

Because `std::min` is used, the elapsed time cannot exceed the Animation duration.

This design is suitable for a simple Animation. However, a professional Animation system may also include features such as Pause, Resume, Reverse, Loop, and Easing.

## Key Takeaways for Professional Developers

- `std::lerp` has been available since C++20 and is declared in the `<cmath>` header.
- This function is designed for Linear Interpolation between Floating-Point values.
- All arguments must have compatible types matching the Template Parameter.
- The `t` parameter is not automatically constrained to the `[0, 1]` interval.
- `std::clamp` can be used when Progress needs to be constrained.
- Using `std::lerp` instead of a conventional manual formula can help prevent important numerical problems, such as Overflow and violations of Monotonicity, under specified conditions.
- The function preserves endpoints exactly when `t == 0` or `t == 1`.
- `constexpr` support makes compile-time calculations possible.
- This function is not a replacement for Cubic Interpolation, Splines, or Easing Functions.
- Its numerical guarantees do not eliminate all Floating-Point errors.
- Invalid inputs, especially NaN and Infinity, require an explicit validation policy.
- `std::midpoint` is also available since C++20 for calculating the midpoint between two values.
- Actual performance depends on the Compiler, processor architecture, and optimization settings.
- Deterministic systems must account for potential differences in Floating-Point calculations across platforms.

## Conclusion

`std::lerp` is a useful C++20 standard library feature designed to provide a standardized and reliable way to perform Linear Interpolation.

In addition to simplifying calculations between two values, it provides important numerical properties such as endpoint preservation, Monotonicity, and protection against Overflow under specified conditions.

Using `std::lerp` in areas such as Game Development, Animation, Computer Graphics, Signal Processing, and Numerical Simulation can make code more readable and maintainable.

For C++ developers working on projects that use C++20 or later, using `std::lerp` instead of an ordinary, unprotected Linear Interpolation implementation is generally recommended, unless the project's specific requirements call for a different algorithm.

Ultimately, it is important to remember that `std::lerp` is a tool for Linear Interpolation, not a universal solution for every problem involving gradual value transitions. Choosing the appropriate algorithm, validating inputs, and understanding Floating-Point limitations remain important responsibilities of the developer.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>