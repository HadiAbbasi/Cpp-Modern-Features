<div align="right">

[🇺🇸 English](./constexpr.md) | [🇮🇷 Persian](../../fa/cpp11/constexpr.md)

</div>

---

# Understanding `constexpr` in C++: From Basic Concepts to Advanced Applications

## Table of Contents

* [Introduction: What Is `constexpr`?](#introduction-what-is-constexpr)
* [Where Did the Need for `constexpr` Come From?](#where-did-the-need-for-constexpr-come-from)
* [Compile-Time vs. Runtime Computation](#compile-time-vs-runtime-computation)
* [What Is a `constexpr` Variable?](#what-is-a-constexpr-variable)
* [Can a `constexpr` Value Be Changed?](#can-a-constexpr-value-be-changed)
* [What Happens If We Don't Initialize a `constexpr`?](#what-happens-if-we-dont-initialize-a-constexpr)
* [Can We Define a `constexpr` Function?](#can-we-define-a-constexpr-function)
* [Do `constexpr` Function Arguments Have to Be `constexpr`?](#do-constexpr-function-arguments-have-to-be-constexpr)
* [The Benefit of Using `constexpr` with Function Arguments](#the-benefit-of-using-constexpr-with-function-arguments)
* [Difference Between `const` and `constexpr`](#difference-between-const-and-constexpr)
* [Difference Between `constexpr` and `consteval`](#difference-between-constexpr-and-consteval)
* [Difference Between `constexpr` and `constinit`](#difference-between-constexpr-and-constinit)
* [Complete Comparison of `const`, `constexpr`, `consteval`, and `constinit`](#complete-comparison)
* [Combined Example: Putting the Concepts Together](#combined-example)
* [Advanced Example: Choosing When Computation Happens](#advanced-example)
* [Important Notes and Common Mistakes](#important-notes-and-common-mistakes)
* [Conclusion](#conclusion)

---

## Introduction: What Is `constexpr`?

`constexpr` is one of the important features of the C++ language that allows a programmer to specify that a value or function **can be evaluated at compile time**.

The word "can" is very important in this definition, because `constexpr` does not necessarily mean that every use of it will always be evaluated at compile time.

Consider the following code:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

The `square` function is a `constexpr` function.

The following expression can be evaluated at compile time:

```cpp
constexpr int x = square(10);
```

In this case, the compiler can calculate the value `100` during compilation.

However, the same function can also be called with a value that is only known at runtime:

```cpp
int n;
std::cin >> n;

int result = square(n);
```

In this case, the value of `n` is not known during compilation, so the computation takes place at runtime.

Therefore, we can think of `constexpr` in simple terms as follows:

> `constexpr` allows the compiler to perform a value or computation at compile time whenever the required conditions are met.

---

## Where Did the Need for `constexpr` Come From?

From the beginning, C++ has tried to balance **high performance** with **flexibility**.

In many programs, there are computations whose results can be determined in advance.

For example, suppose we have the following function:

```cpp
int square(int x)
{
    return x * x;
}
```

If we write:

```cpp
int x = square(10);
```

From a logical perspective, the compiler knows that the result is `100`.

However, the language needed to specify whether such a function could be used in contexts where the value must be known at compile time.

For example, array sizes or certain template parameters require compile-time values.

Older versions of C++ had limited facilities for expressing this concept.

The `constexpr` feature was introduced in C++11 so that programmers could explicitly indicate that a value or function **can be evaluated at compile time**.

---

## What Problem Would Exist Without `constexpr`?

The absence of `constexpr` would not mean that C++ compilers were unable to perform any computations at compile time.

Compilers had already been performing optimizations such as **constant folding** long before `constexpr` existed.

For example, a compiler might effectively transform:

```cpp
int x = 10 * 20;
```

into a value of `200`.

However, the main problem was that programmers had no standard and reliable way to tell the language:

> "This value or function is semantically required to be capable of being evaluated at compile time."

This distinction is very important.

`constexpr` is not merely a request for an optimization; it is part of the **semantic rules of the C++ language**.

---

## Compile-Time vs. Runtime Computation

To understand `constexpr`, we first need to understand the difference between these two concepts.

A computation performed at **compile time** takes place before the program runs.

A computation performed at **runtime** takes place while the program is executing and is performed by the CPU.

For example:

```cpp
constexpr int value = 10 * 20;
```

In this example, the value `200` is known during compilation.

However, in the following example, the value is only known while the program is running:

```cpp
int x;
std::cin >> x;

int value = x * 20;
```

Here, the compiler cannot know the value of `x` in advance.

---

## What Is a `constexpr` Variable?

A `constexpr` variable is a variable whose initializer must be a valid **constant expression**.

For example:

```cpp
constexpr int max_users = 100;
```

Here, the value `100` is known at compile time.

Another example:

```cpp
constexpr int x = 10;
constexpr int y = x * 2;
```

Here, the value of `y` can also be calculated at compile time.

The following example is also valid:

```cpp
constexpr int square(int x)
{
    return x * x;
}

constexpr int result = square(5);
```

The compiler can determine that `result` is equal to `25`.

---

## Can a `constexpr` Value Be Changed?

No.

A `constexpr` variable cannot be modified.

For example:

```cpp
constexpr int x = 10;

x = 20; // error
```

The reason is that a `constexpr` variable is effectively a **constant**.

More precisely, a `constexpr` variable also has `const` semantics.

Therefore:

```cpp
constexpr int x = 10;
```

behaves as a constant with respect to constness.

---

## What Happens If We Don't Initialize a `constexpr`?

A `constexpr` variable must be initialized at the point where it is defined.

Therefore, the following code is invalid:

```cpp
constexpr int x; // error
```

The compiler cannot accept a `constexpr` variable without an initializer.

The correct form is:

```cpp
constexpr int x = 10;
```

This is one of the important differences between `constexpr` and an ordinary variable.

Even `const` generally needs to be initialized when defining an ordinary object:

```cpp
const int x = 10;
```

However, the important point is that `constexpr` imposes a stronger requirement regarding **the ability to evaluate the value at compile time**, in addition to constness.

---

## Can We Define a `constexpr` Variable or Function?

Yes.

`constexpr` can be used with several kinds of entities in C++, the most important ones being variables and functions.

For example, for a variable:

```cpp
constexpr int size = 100;
```

And for a function:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

`constexpr` is also useful when designing classes and constructors:

```cpp
class Point
{
public:
    constexpr Point(int x, int y)
        : x_(x), y_(y)
    {
    }

    constexpr int x() const
    {
        return x_;
    }

    constexpr int y() const
    {
        return y_;
    }

private:
    int x_;
    int y_;
};
```

We can now construct an object in compile-time contexts:

```cpp
constexpr Point p(10, 20);

static_assert(p.x() == 10);
static_assert(p.y() == 20);
```

---

## Is a `constexpr` Function Always Executed at Compile Time?

No.

This is one of the most important points about `constexpr`.

Suppose we have the following function:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

If the argument is known at compile time:

```cpp
constexpr int a = square(10);
```

the computation can be performed at compile time.

However, if the argument is known only at runtime:

```cpp
int n;

std::cin >> n;

int a = square(n);
```

the same function can execute at runtime.

Therefore, `constexpr` does not mean that a function **must always execute at compile time**.

A more precise definition is:

> A `constexpr` function must be defined in a way that allows it to be evaluated as a constant expression when the required conditions are satisfied.

---

## Do `constexpr` Function Arguments Have to Be `constexpr`?

No.

This is a common misconception about `constexpr`.

Consider the following function:

```cpp
constexpr int multiply(int a, int b)
{
    return a * b;
}
```

We can use it with `constexpr` values:

```cpp
constexpr int a = 10;
constexpr int b = 20;

constexpr int result = multiply(a, b);
```

In this case, the result is also evaluated at compile time.

However, we can also use ordinary variables:

```cpp
int a = 10;
int b = 20;

int result = multiply(a, b);
```

This code is completely valid.

Even if the value of a variable changes at runtime, the function can still be used:

```cpp
int a;

std::cin >> a;

int result = multiply(a, 20);
```

In this case, `multiply` executes at runtime.

---

## The Benefit of Using `constexpr` with Function Arguments

If a function's input is known in advance, using `constexpr` can make it possible to perform the computation at compile time.

For example:

```cpp
constexpr int cube(int x)
{
    return x * x * x;
}

constexpr int value = cube(4);
```

The compiler can calculate the result in advance:

```text
value = 64
```

This capability is particularly useful in areas such as:

* Constant mathematical computations
* Array sizes
* Template parameters
* `static_assert`
* Constant data structures
* Generating lookup tables at compile time
* Metadata computations
* Template programming
* Certain embedded and performance-sensitive applications

---

## What Happens If the Input Is Not `constexpr`?

If an input is determined at runtime, the result cannot be used as a constant expression.

For example:

```cpp
constexpr int square(int x)
{
    return x * x;
}

int n;
std::cin >> n;

constexpr int result = square(n); // error
```

The reason is that `n` does not have a known value at compile time.

However, if we remove `constexpr` from the result:

```cpp
int result = square(n);
```

the code is completely valid.

So the problem is not that the function is `constexpr`.

The problem is that **whether the result is a constant expression depends on its inputs**.

---

## An Important Example: One Function, Two Types of Usage

Consider the following function:

```cpp
constexpr int factorial(int n)
{
    if (n <= 1)
        return 1;

    return n * factorial(n - 1);
}
```

We can now use it in two different ways.

Compile-time usage:

```cpp
constexpr int a = factorial(5);
```

Runtime usage:

```cpp
int n;

std::cin >> n;

int b = factorial(n);
```

Therefore, a `constexpr` function can be both **compile-time** and **runtime**.

---

## Difference Between `const` and `constexpr`

The most important difference is that `const` is about **immutability**, while `constexpr` also imposes a requirement regarding **compile-time evaluability**.

For example:

```cpp
int get_value()
{
    return 42;
}

const int a = get_value();
```

This code can be completely valid.

The value of `a` cannot be changed after initialization, but it is not necessarily a constant expression.

However:

```cpp
constexpr int b = 42;
```

The value of `b` must be computable at compile time.

We can summarize the difference as follows:

```text
const      → cannot be modified after initialization
constexpr  → the value must be evaluable at compile time
```

Therefore:

```cpp
const int a = 10;
constexpr int b = 10;
```

Both are immutable, but `b` provides a stronger guarantee about compile-time evaluation.

---

## A Real Example of the Difference Between `const` and `constexpr`

Consider the following function, which only executes at runtime:

```cpp
int get_number()
{
    return 42;
}
```

Now:

```cpp
const int a = get_number();
```

The value of `a` is constant, but `get_number()` must execute to produce it.

However, this code:

```cpp
constexpr int b = get_number();
```

is invalid because `get_number()` is an ordinary function and cannot be used as a constant expression in this context.

If we make the function `constexpr`:

```cpp
constexpr int get_number()
{
    return 42;
}
```

then:

```cpp
constexpr int b = get_number();
```

is valid.

---

## Difference Between `constexpr` and `consteval`

The difference between `constexpr` and `consteval` is very important.

A `constexpr` function **can** execute at compile time.

A `consteval` function **must** execute at compile time.

For example:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

This function can be used both at compile time and at runtime.

However:

```cpp
consteval int square(int x)
{
    return x * x;
}
```

A call to this function must produce a result that can be evaluated at compile time.

For example:

```cpp
constexpr int a = square(10);
```

is valid.

But:

```cpp
int n;

std::cin >> n;

int a = square(n); // error
```

The name `consteval` was chosen to express the concept of **mandatory constant evaluation**.

---

## The Conceptual Difference Between `constexpr` and `consteval`

You can remember these two as follows:

```text
constexpr → compile-time if possible
consteval → compile-time mandatory
```

Therefore, if we want to design an API in such a way that it **can never execute at runtime**, `consteval` is the more appropriate tool.

For example:

```cpp
consteval int square(int x)
{
    return x * x;
}
```

This feature can be extremely powerful for building compile-time utilities.

---

## Difference Between `constexpr` and `constinit`

`constinit` represents a different concept.

`constinit` is used for variables with **static storage duration** or **thread storage duration**, and its primary purpose is to guarantee **constant initialization**.

For example:

```cpp
constinit int global_value = 100;
```

This variable can still be modified:

```cpp
global_value = 200;
```

Therefore:

```cpp
constinit int x = 10;
x = 20; // valid
```

But:

```cpp
constexpr int x = 10;
x = 20; // error
```

So `constinit` does not mean immutable.

---

## Why Was `constinit` Introduced?

One important issue in C++ is the **initialization order of global objects** across different translation units.

Suppose we have several global variables whose initialization depends on one another.

Under certain circumstances, the initialization of one variable may occur before or after another variable, potentially leading to problems such as the **Static Initialization Order Fiasco**.

`constinit` helps ensure that the specified variable is initialized through **constant initialization**.

For example:

```cpp
constinit int value = 42;
```

The primary purpose here is not to make `value` constant.

The purpose is to ensure that its initialization happens at the appropriate stage and through constant initialization.

---

## An Important Difference Between `constexpr` and `constinit`

Consider the following example:

```cpp
constexpr int a = 10;
```

Here, `a` is both a compile-time constant and immutable.

However:

```cpp
constinit int b = 10;
```

Here, `b` can be modified:

```cpp
b = 20;
```

Therefore, these two keywords do not have the same purpose.

```text
constexpr → constant value / constant expression
constinit  → constant initialization
```

---

## Complete Comparison

| Feature                            | `const`                   | `constexpr`             | `consteval`                                  | `constinit`                      |
| ---------------------------------- | ------------------------- | ----------------------- | -------------------------------------------- | -------------------------------- |
| Prevents modification of the value | Yes                       | Yes                     | Applies to functions                         | No                               |
| Requires compile-time value        | No                        | Yes                     | Yes                                          | Requires constant initialization |
| For variables                      | Yes                       | Yes                     | No                                           | Yes                              |
| For functions                      | No                        | Yes                     | Yes                                          | No                               |
| Runtime execution                  | Yes                       | Yes, when needed        | No                                           | Related to initialization        |
| Initializer required               | Usually yes for an object | Yes                     | Function must have an appropriate definition | Yes                              |
| Primary purpose                    | Immutability              | Compile-time evaluation | Mandatory compile-time evaluation            | Constant initialization          |

---

## Combined Example: Putting the Concepts Together

Now let's put the four concepts together:

```cpp
constexpr int compile_time_value = 10;

const int runtime_constant = get_value();

consteval int force_compile_time(int x)
{
    return x * 2;
}

constinit int global_value = 100;
```

Each keyword has a different role in this example.

The `compile_time_value` variable is a compile-time constant.

The `runtime_constant` variable cannot be modified after initialization, but its value is not necessarily determined at compile time.

The `force_compile_time` function must be evaluable at compile time.

The `global_value` variable must have constant initialization, but it is not immutable.

---

## Advanced Example: Combining `constexpr` with `static_assert`

One of the most important applications of `constexpr` is using it together with `static_assert`.

For example:

```cpp
constexpr int square(int x)
{
    return x * x;
}

constexpr int value = square(5);

static_assert(value == 25);
```

Here, `static_assert` is also evaluated at compile time.

If the value is incorrect:

```cpp
static_assert(value == 30);
```

the compilation will fail.

This feature is extremely useful for building software where certain conditions need to be verified at compile time.

---

## Advanced Example: Using `constexpr` with Templates

One powerful use of `constexpr` is in combination with templates.

For example:

```cpp
constexpr int square(int x)
{
    return x * x;
}

template<int Size>
class Buffer
{
    char data[Size];
};

Buffer<square(4)> buffer;
```

Here, `square(4)` must produce a value that is known at compile time.

Therefore, the result of `square(4)` is `16`, and the class will effectively be instantiated as:

```cpp
Buffer<16> buffer;
```

This example demonstrates that `constexpr` is not merely a tool for making code faster.

It can actually become part of the **structure of a program at compile time**.

---

## Advanced Example: Computing an Array at Compile Time

Suppose we want to generate a constant lookup table.

```cpp
constexpr int square(int x)
{
    return x * x;
}

constexpr int table_size = 10;

constexpr auto create_table()
{
    std::array<int, table_size> result{};

    for (int i = 0; i < table_size; ++i)
        result[i] = square(i);

    return result;
}

constexpr auto table = create_table();
```

In newer C++ standards, the capabilities of `constexpr` have become much broader, and many operations that were only possible at runtime in older versions can now be performed during constant evaluation.

However, the exact capabilities of a `constexpr` function also depend on the C++ standard version being used.

---

## Advanced Example: `constexpr` for Objects

`constexpr` is not limited to `int` and other simple types.

If a type provides the capabilities required for constant evaluation, objects of that type can also be constructed at compile time.

For example:

```cpp
struct Point
{
    int x;
    int y;

    constexpr int distance_squared() const
    {
        return x * x + y * y;
    }
};

constexpr Point p{3, 4};

constexpr int distance = p.distance_squared();

static_assert(distance == 25);
```

In this example, the `Point` object itself is constructed at compile time.

---

## Does `constexpr` Always Make a Program Faster?

Not necessarily.

The primary purpose of `constexpr` is not simply to increase runtime performance.

In many cases, the compiler can optimize constant computations even without `constexpr`.

The real value of `constexpr` is that it allows us to make **computations part of the program's compile-time computation**.

This can reduce runtime work, but more importantly, it can make it possible to use the result in contexts that require a constant expression.

Therefore, we should not equate these two statements:

```text
constexpr = optimization
```

and:

```text
constexpr = a language feature for expressing and enabling compile-time evaluation
```

The second statement is more accurate.

---

## Important Notes and Common Mistakes

### Mistake 1: Assuming `constexpr` Always Executes at Compile Time

The following function:

```cpp
constexpr int add(int a, int b)
{
    return a + b;
}
```

can also be used with runtime inputs:

```cpp
int a;
std::cin >> a;

int result = add(a, 10);
```

Therefore, `constexpr` by itself does not prohibit runtime execution.

---

### Mistake 2: Assuming Function Arguments Must Be `constexpr`

The following code is completely valid:

```cpp
constexpr int add(int a, int b)
{
    return a + b;
}

int x = 10;
int y = 20;

int result = add(x, y);
```

Here, the function is `constexpr`, but its arguments are not `constexpr`.

---

### Mistake 3: Assuming `const` Is the Same as `constexpr`

The following:

```cpp
const int x = get_value();
```

only says that `x` cannot be modified after initialization.

However:

```cpp
constexpr int x = get_value();
```

requires `get_value()` to be evaluable as a constant expression, in addition to the value being constant.

---

### Mistake 4: Assuming `consteval` Is the Same as `constexpr`

A `constexpr` function can execute at runtime.

A `consteval` function must be evaluated at compile time.

For example:

```cpp
constexpr int f(int x)
{
    return x * 2;
}
```

combined with:

```cpp
int x = 10;
int y = f(x);
```

is allowed.

However:

```cpp
consteval int f(int x)
{
    return x * 2;
}
```

cannot be used in a way that requires runtime execution.

---

### Mistake 5: Assuming `constinit` Means the Value Is Constant

The following code is completely valid:

```cpp
constinit int counter = 0;

counter++;
counter++;
```

Here, `counter` can be modified.

Therefore, `constinit` should not be confused with `const` or `constexpr`.

---

## A Simple Mental Model for Remembering the Differences

To remember these four keywords, we can ask four different questions.

If the question is:

> Can I change this value after initialization?

The relevant keyword is:

```cpp
const
```

If the question is:

> Does this value need to be a constant expression?

The relevant keyword is:

```cpp
constexpr
```

If the question is:

> Must this function definitely execute at compile time?

The relevant keyword is:

```cpp
consteval
```

If the question is:

> Must the initialization of this global/static variable be constant initialization?

The relevant keyword is:

```cpp
constinit
```

---

## A Final Example Comparing All Four Concepts

Suppose we have the following code:

```cpp
int runtime_value()
{
    return 42;
}

constexpr int compile_time_value()
{
    return 42;
}

const int a = runtime_value();

constexpr int b = compile_time_value();

consteval int double_value(int x)
{
    return x * 2;
}

constinit int global_counter = 0;
```

In this example, `a` is a constant value, but producing its value may happen at runtime.

The variable `b` is a compile-time constant.

The function `double_value` must perform its evaluation during constant evaluation when called.

The variable `global_counter` is initialized using constant initialization, but it can still be modified.

---

## Conclusion

`constexpr` is one of the key tools in modern C++ for moving part of a computation from runtime to compile time.

A `constexpr` variable must have an initializer that can be evaluated at compile time, and it cannot be modified after initialization.

A `constexpr` function can execute at compile time or at runtime, depending on its context and inputs.

A `consteval` function goes one step further and makes compile-time evaluation mandatory.

The `constinit` keyword solves a different problem and focuses on **how static-storage-duration or thread-storage-duration variables are initialized**, rather than on making a variable immutable.

Therefore, these four concepts can be remembered in one line:

```text
const      → don't modify it
constexpr  → compile-time when possible/required
consteval  → compile-time mandatory
constinit  → perform constant initialization
```

The most important point is that we should not think of `constexpr` merely as an **optimization hint**.

`constexpr` is a language feature for expressing that a value, function, or operation **can participate in constant evaluation**, and this capability is fundamental to many techniques in modern C++, including template metaprogramming, `static_assert`, compile-time data generation, and the design of compile-time APIs.

## 🤝 Contributions

<div align="center">

| GitHub     | LinkedIn    | Email       | Site      | Telegram    |
| ---------- | ----------- | ----------- | --------- | ----------- |
| HadiAbbasi | Hadi Abbasi | Hadi Abbasi | Hiens.org | Hadi Abbasi |

</div>
