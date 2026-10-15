<div align="center">

[🇺🇸 English](./relaxed_constexpr.md) | [🇮🇷 فارسی](../../fa/cpp14/relaxed_constexpr.md)

</div>

---

# Relaxed `constexpr` in C++14

## Introduction

The `constexpr` feature introduced in C++11 allowed programmers to define functions and variables that could be evaluated at compile time. However, this capability was subject to relatively strict restrictions. For example, the body of a `constexpr` function in C++11 was effectively limited to a single `return` statement and could not use local variables, loops, or common control-flow constructs as an ordinary function would.

C++14 relaxed these restrictions. This change, commonly known as **Relaxed `constexpr`**, made compile-time evaluable functions more natural to write and allowed algorithms involving multiple computational steps, local variables, and loops to be evaluated at compile time, provided they satisfy the rules for constant evaluation.

An important point is that Relaxed `constexpr` does not mean every `constexpr` function is necessarily executed at compile time. Such a function can also be called at runtime; the evaluation context and whether constant evaluation is possible determine how it is evaluated.

## Understanding `constexpr` Before C++14

In C++11, `constexpr` functions were defined with strict restrictions to keep compile-time evaluation manageable. In general, the body of such a function had to contain a `return` statement with an expression suitable for constant evaluation.

For example, the following function is valid in C++11:

```cpp
constexpr int square(int value)
{
    return value * value;
}
```

In this example, if the function argument is a constant expression, the result can also be computed at compile time:

```cpp
constexpr int result = square(5);

static_assert(result == 25);
```

However, implementing the same calculation with a local variable was not allowed in C++11:

```cpp
constexpr int square(int value)
{
    int result = value * value; // Not allowed in C++11 constexpr function bodies
    return result;
}
```

The reason for this restriction was that the original `constexpr` rules deliberately limited function bodies to a very simple form. This design was suitable for simple calculations but did not provide enough flexibility for real algorithms.

## The Main Changes in C++14

In C++14, a `constexpr` function body can contain many statements commonly used in ordinary functions, including:

- Defining local variables of suitable types;
- Using `if` and `else` conditions;
- Using `for`, `while`, and `do-while` loops;
- Using multiple sequential statements and assignments to local variables;
- Using multiple `return` statements on different execution paths.

This flexibility is not unlimited, however. The function must still obey the rules for constant evaluation and cannot perform operations that the standard prohibits in a constant evaluation.

### Example: Calculating a Square with a Local Variable

In C++14, the previous function can be written as follows:

```cpp
constexpr int square(int value)
{
    int result = value * value;
    return result;
}

constexpr int result = square(5);
static_assert(result == 25);
```

In this version, defining the local variable `result` is allowed. When the function is used in a constant expression, the compiler can perform the calculation at compile time.

### Example: Using a Conditional

The following function returns the larger of two numbers:

```cpp
constexpr int maxValue(int a, int b)
{
    if (a > b)
        return a;

    return b;
}

static_assert(maxValue(7, 4) == 7);
static_assert(maxValue(2, 9) == 9);
```

Using an `if` statement does not prevent a function from being `constexpr`. In C++14, the selected execution path can be evaluated during constant evaluation, provided the operations on that path are permitted.

### Example: Using a Loop

One of the most important improvements is the ability to express repeated calculations with loops. For example, the following function calculates the factorial of a non-negative integer:

```cpp
constexpr unsigned long long factorial(unsigned int n)
{
    unsigned long long result = 1;

    for (unsigned int i = 2; i <= n; ++i)
        result *= i;

    return result;
}

static_assert(factorial(0) == 1);
static_assert(factorial(5) == 120);
```

This function can be evaluated at compile time in C++14. Keep in mind that the return type has a limited capacity; for large inputs, arithmetic overflow can invalidate the result. The number of iterations and compiler resource limits can also affect whether constant evaluation is feasible.

## Compile-Time Evaluation vs. Runtime Evaluation

The presence of the `constexpr` keyword alone does not guarantee that every call to a function will be evaluated at compile time. The keyword makes the function eligible for constant evaluation, but the calling context determines whether constant evaluation is required.

```cpp
constexpr int square(int value)
{
    int result = value * value;
    return result;
}

constexpr int compileTimeValue = square(6);

int runtimeInput = 6;
int runtimeValue = square(runtimeInput);
```

In this example, `compileTimeValue` must be computable as a constant expression. By contrast, the call whose argument comes from `runtimeInput` depends on a runtime value and is ordinarily evaluated at runtime. The compiler may optimize it when possible, but that is different from a language requirement for constant evaluation.

To require constant evaluation, use a context that requires a constant expression, such as initializing a `constexpr` variable or supplying a condition to `static_assert`.

## Important Rules and Limitations

Relaxed `constexpr` reduces restrictions on function bodies, but the rules for constant evaluation still apply.

### Local Variables and Their Initialization

Local variables can be used in calculations inside a `constexpr` function, provided the operations and types involved comply with the rules for constant evaluation.

```cpp
constexpr int sumTo(int n)
{
    int sum = 0;

    for (int i = 1; i <= n; ++i)
        sum += i;

    return sum;
}

static_assert(sumTo(4) == 10);
```

In this function, the variables `sum` and `i` are used to build a multi-step calculation. This pattern is much more practical in C++14 than restricting the function to a single `return` expression.

### Execution Paths and Invalid Operations

During constant evaluation, only the operations on the selected execution path matter. The presence of an operation that would not be permitted in constant evaluation on another path does not necessarily make every call to the function invalid. The question is whether the particular call can be evaluated as a valid constant expression.

For example, division by zero on the executed path cannot produce a valid constant evaluation:

```cpp
constexpr int divideOrZero(int value, int divisor)
{
    if (divisor == 0)
        return 0;

    return value / divisor;
}

static_assert(divideOrZero(10, 2) == 5);
static_assert(divideOrZero(10, 0) == 0);
```

In this example, the condition prevents division by zero. If the function directly evaluates `value / divisor` with a zero divisor, that constant evaluation would not be valid.

### Type and Operation Restrictions

Whether a type can be used in a `constexpr` function depends on the rules of the relevant standard and on the operations being performed. C++14 still has restrictions compared with newer standards, particularly regarding changes to the state of dynamic objects, use of the standard library, and side effects.

For example, Relaxed `constexpr` does not mean that every I/O operation, every memory allocation, or every library function can be executed at compile time in C++14. Each operation must independently comply with the standard's rules for constant evaluation.

## Practical Applications

### Computing Constant Values

For calculations whose results must be known at compile time, `constexpr` can avoid repeated runtime calculations and make the required values available to mechanisms such as `static_assert`.

```cpp
constexpr unsigned int powerOfTwo(unsigned int exponent)
{
    unsigned int result = 1;

    for (unsigned int i = 0; i < exponent; ++i)
        result *= 2;

    return result;
}

static_assert(powerOfTwo(0) == 1);
static_assert(powerOfTwo(8) == 256);
```

This implementation is suitable for inputs whose results remain within the range of `unsigned int`. In a real design, the behavior for large inputs should also be specified.

### Compile-Time Validation

`constexpr` functions can be used to check properties of constant values and use the results in `static_assert`:

```cpp
constexpr bool isEven(int value)
{
    return value % 2 == 0;
}

static_assert(isEven(10));
static_assert(!isEven(7));
```

This pattern is especially useful for configuration constants, sizes, numeric properties, and conditions that must be checked during compilation.

### Implementing Simple Algorithms

Loops and local variables make it possible to implement algorithms such as summing elements, simple searching, exponentiation, and checking numeric properties. However, whether an algorithm can be evaluated at compile time depends on the data types, permitted operations, and compiler resources.

## Design Considerations and Common Mistakes

### Assuming Compile-Time Execution Is Mandatory

A common mistake is to assume that a `constexpr` function always executes at compile time. In reality, such a function can also be called at runtime. If compile-time execution is a design requirement, the result must be used in a context that requires a constant expression.

### Ignoring Overflow and Input Ranges

The fact that a function can be evaluated at compile time does not mean its algorithm is correct for every input. Calculations such as factorials and powers can exceed the range of a numeric type. Signed integer overflow generally causes undefined behavior, while unsigned integer arithmetic is performed modulo the type's range. Input ranges and boundary behavior should be considered explicitly.

### Using `constexpr` Without a Clear Reason

If a function will not be used in a constant-expression context and enabling that capability offers no clear design benefit, adding `constexpr` may provide little practical value. Nevertheless, for small functions and calculations that reasonably should be evaluable at compile time, the feature can be useful.

### Confusing a Language Feature with Compiler Optimization

Constant evaluation is a language concept; compiler optimization is a separate matter. A compiler may optimize a call to an ordinary function and compute its result efficiently, but that does not mean the result can be used in every context that requires a constant expression.

## Comparing `constexpr` in C++11 and C++14

| Capability | C++11 | C++14 |
|---|---|---|
| Very restricted body with a single `return` | Yes | Restriction relaxed |
| Defining local variables in a `constexpr` function | Generally not allowed | Allowed, subject to the rules |
| Using `if` and `else` conditions | Severely restricted | Supported |
| Using loops | Not allowed | Supported |
| Multiple calculation steps and local assignments | Very limited | Supported |
| Guaranteeing that all calls execute at compile time | No | No |

The table highlights the main difference: C++14 brought the way `constexpr` functions are written closer to ordinary functions, but it did not remove the requirements for constant evaluation.

## Differences from Features in Newer Standards

Relaxed `constexpr` specifically refers to the changes introduced in C++14. Later standards expanded the use of `constexpr` further; for example, C++17 and C++20 introduced additional changes to the language and standard library. Therefore, C++14 limitations should not be applied to newer standards without checking, and features introduced later should not be assumed to be part of C++14.

In projects that must compile under multiple language standards, the code should be checked against the target standard. Compiler support for a feature alone does not prove that the feature belongs to the intended standard.

## Summary

Relaxed `constexpr` in C++14 was an important improvement over C++11. It allowed compile-time-evaluable functions to use local variables, conditions, loops, and multiple computational steps, making it much easier to implement real algorithms with `constexpr`.

However, `constexpr` does not guarantee that every call will be evaluated at compile time, and not every operation is permitted during constant evaluation. Professional use requires understanding the difference between eligibility for constant evaluation and mandatory compile-time evaluation, the limitations of the target standard, valid input ranges, and the behavior of numeric calculations.


---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>