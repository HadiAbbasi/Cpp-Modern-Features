<div align="right">

[🇺🇸 English](./consteval.md) | [🇮🇷 فارسی](../../fa/cpp20/consteval.md)

</div>

---
# A Simple Guide to constinit in C++: Controlling Initialization

## Table of Contents

* [Introduction](#introduction)
* [What is constinit?](#what-is-constinit)
* [Why was constinit introduced?](#why-was-constinit-introduced)
* [What is the problem with dynamic initialization?](#what-is-the-problem-with-dynamic-initialization)
* [How to use constinit](#how-to-use-constinit)
* [Can the value of a constinit variable be changed?](#can-the-value-of-a-constinit-variable-be-changed)
* [What happens if we do not provide an initial value?](#what-happens-if-we-do-not-provide-an-initial-value)
* [constinit vs const](#constinit-vs-const)
* [constinit vs constexpr](#constinit-vs-constexpr)
* [constinit vs consteval](#constinit-vs-consteval)
* [Final comparison: const, constexpr, consteval, and constinit](#final-comparison-const-constexpr-consteval-and-constinit)
* [Practical examples](#practical-examples)
* [An advanced example with static initialization](#an-advanced-example-with-static-initialization)
* [Limitations of constinit](#limitations-of-constinit)
* [What problem would we have without constinit?](#what-problem-would-we-have-without-constinit)
* [When should we use constinit?](#when-should-we-use-constinit)
* [Conclusion](#conclusion)

---

# Introduction

The `constinit` keyword is a feature introduced in **C++20** that is designed to control the initialization of variables with **static storage duration** or **thread storage duration**.

The main idea behind `constinit` is very simple:

> `constinit` tells the compiler that the variable **must be initialized using constant initialization**.

An important point is that `constinit` does **not** make a variable `const`.

This means that a variable such as the following can still be modified after initialization:

```cpp
constinit int counter = 10;

counter = 20;
counter = 30;
```

In this example, the initial value of `counter` must be established through constant initialization, but `counter` itself remains a mutable variable.

---

# What is constinit?

The `constinit` keyword is a **specifier** in C++ that can be applied to variables with static or thread storage duration.

A simple example looks like this:

```cpp
constinit int value = 42;
```

Here, the compiler is required to verify that `value` has a **constant initializer**.

The following example is also valid:

```cpp
constinit int value = 10 + 20;
```

The reason is that the initial value `30` can be determined as a constant.

However, the purpose of `constinit` is not simply to tell the compiler to evaluate an expression at compile time.

Its main purpose is to make sure that the variable does **not require dynamic initialization**.

---

# Why was constinit introduced?

The main problem addressed by `constinit` is related to the initialization of variables with **static storage duration**.

Global variables and `static` variables can be initialized as part of the program startup process.

In general, this initialization can happen in two important ways:

```text
constant initialization
dynamic initialization
```

Constant initialization can generally be completed before normal program execution begins.

Dynamic initialization, on the other hand, may require code to execute during program startup.

Problems can occur when a programmer expects a variable to have a particular value before other parts of the program begin using it, while its initialization actually requires dynamic initialization.

---

# What is the problem with dynamic initialization?

Suppose we have two separate source files.

The first file contains:

```cpp
int getValue();

int globalValue = getValue();
```

The second file contains:

```cpp
int otherValue = globalValue;
```

In such a situation, the initialization order of variables across different translation units can become a problem.

A well-known issue related to this is the **Static Initialization Order Fiasco**.

In other words, one global variable may be used before another global variable that it depends on has been dynamically initialized.

For example, imagine:

```cpp
// file_a.cpp

int getNumber()
{
    return 42;
}

int number = getNumber();
```

And in another file:

```cpp
// file_b.cpp

extern int number;

int value = number;
```

Here, `number` is used to initialize `value`.

If `number` requires dynamic initialization, the initialization order across different translation units can become problematic.

---

# What was the old solution?

One common solution was to make the variable `constexpr`.

For example:

```cpp
constexpr int number = 42;
```

This works very well, but there is an important difference.

A `constexpr` variable must be a **constant**.

That means its value cannot be changed later:

```cpp
constexpr int number = 42;

// compilation error
number = 100;
```

Sometimes this is exactly what we do **not** want.

Sometimes we need the initial value of a variable to be established through constant initialization, while the variable itself must remain mutable.

This is where `constinit` becomes useful.

---

# How to use constinit

The simplest way to use `constinit` is:

```cpp
constinit int value = 42;
```

Here, `value` is an ordinary mutable variable.

That means the following code is perfectly valid:

```cpp
constinit int value = 42;

int main()
{
    value = 100;
}
```

After the program executes, `value` will contain `100`.

The important point is that `constinit` only restricts the **initialization process**. It does not restrict how the variable can be used after initialization.

---

# Can the value of a constinit variable be changed?

Yes.

This is one of the most important differences between `constinit` and `const` or `constexpr`.

The following code is completely valid:

```cpp
constinit int counter = 0;

void increment()
{
    ++counter;
}
```

Here, `counter` is not constant.

The only guarantee is that its initial initialization is performed through constant initialization.

Therefore, we can think about the distinction like this:

```text
constinit
    controls initialization

const
    controls whether the variable can be modified

constexpr
    is about constant expressions and immutable objects
```

---

# What happens if we do not provide an initial value?

This part is slightly subtle.

Consider:

```cpp
constinit int value;
```

In this situation, `value` can be zero-initialized, resulting in an initial value of `0`.

Therefore, omitting the initializer does not necessarily mean that the code is ill-formed.

For example:

```cpp
constinit int value;

int main()
{
    // value is zero
}
```

The important point is that `constinit` still imposes its constant-initialization requirement.

On the other hand, if the declaration is an `extern` declaration, the declaration can omit the initializer and the actual definition can appear elsewhere.

For example:

```cpp
// header.hpp

extern constinit int counter;
```

And then:

```cpp
// source.cpp

constinit int counter = 100;
```

This pattern can be useful for global variables.

---

# constinit vs const

The keywords `const` and `constinit` may look similar, but they serve completely different purposes.

The `const` keyword means:

> The object cannot be modified through this name after initialization.

For example:

```cpp
const int value = 42;
```

In this case:

```cpp
value = 100;
```

is not allowed.

The `constinit` keyword means:

> The variable must be initialized through constant initialization.

For example:

```cpp
constinit int value = 42;
```

Here:

```cpp
value = 100;
```

is completely valid.

Therefore, the main difference can be summarized as follows:

| Feature                                           | `const` | `constinit` |
| ------------------------------------------------- | ------- | ----------- |
| Makes the variable immutable                      | Yes     | No          |
| Requires constant initialization                  | No      | Yes         |
| Specifically intended for static/thread storage   | No      | Yes         |
| Introduced in C++20                               | No      | Yes         |
| Main purpose is preventing dynamic initialization | No      | Yes         |

---

# constinit vs constexpr

The difference between `constinit` and `constexpr` is one of the most important parts of this topic.

The `constexpr` keyword is used to express that a value or function can participate in **constant expressions**.

For example:

```cpp
constexpr int size = 100;
```

This value can be used in contexts where a constant expression is required.

For example:

```cpp
int array[size];
```

On the other hand:

```cpp
constinit int size = 100;
```

does not primarily mean that `size` is a constant expression that can be used everywhere a constant expression is required.

`constinit` also does not make the variable immutable.

Therefore:

```cpp
constexpr int a = 10;
```

and:

```cpp
constinit int b = 10;
```

do not have the same semantics.

Variable `a` cannot be modified:

```cpp
// error
a = 20;
```

But `b` can be modified:

```cpp
b = 20;
```

---

# constinit vs consteval

The `consteval` keyword is also fundamentally different from `constinit`.

The `consteval` keyword is used to define an **immediate function**.

For example:

```cpp
consteval int square(int x)
{
    return x * x;
}
```

A call to such a function must be evaluated at compile time in the appropriate context:

```cpp
constexpr int value = square(10);
```

Therefore:

```text
consteval
    is about functions.

constexpr
    is about compile-time evaluability.

constinit
    is about initialization of static or thread-local variables.

const
    is about making an object immutable.
```

These four keywords should not be treated as interchangeable simply because their names look similar.

Each one solves a different problem.

---

# Final comparison: const, constexpr, consteval, and constinit

The following table provides an overall comparison:

| Feature                         |              `const` |          `constexpr` | `consteval` | `constinit` |
| ------------------------------- | -------------------: | -------------------: | ----------: | ----------: |
| Applies to variables            |                  Yes |                  Yes |          No |         Yes |
| Applies to functions            |                   No |                  Yes |         Yes |          No |
| Makes a variable immutable      |                  Yes |                  Yes |           — |          No |
| Forces compile-time execution   |                   No | In required contexts |         Yes |          No |
| Prevents dynamic initialization | Not its main purpose | In appropriate cases |           — |         Yes |
| Requires static/thread storage  |                   No |                   No |           — |         Yes |
| Introduced in C++20             |                   No |                   No |         Yes |         Yes |

---

# Practical examples

Suppose we have a global variable whose initial value comes from a constant expression:

```cpp
constinit int maxConnections = 100;
```

We can then modify it during program execution:

```cpp
void configure()
{
    maxConnections = 200;
}
```

This is one of the appropriate use cases for `constinit`.

---

# A simple example with a function

Now suppose the initial value comes from a function:

```cpp
int getDefaultValue()
{
    return 100;
}

constinit int value = getDefaultValue();
```

This code is not valid.

The reason is that `getDefaultValue()` is an ordinary function, and calling it cannot provide a constant initializer.

If the goal is for the function to be evaluated at compile time, we can use `constexpr` when appropriate:

```cpp
constexpr int getDefaultValue()
{
    return 100;
}

constinit int value = getDefaultValue();
```

In this case, the initialization of `value` can be constant initialization.

The subtle point here is that `constexpr` and `constinit` can be used together, but they have different roles.

The `constexpr` keyword concerns the ability of the function to be evaluated at compile time, while `constinit` applies to the variable and requires its initialization to be constant initialization.

---

# An advanced example with thread_local

The `constinit` keyword is not limited to ordinary global variables.

`thread_local` variables can also use it:

```cpp
constinit thread_local int threadCounter = 0;
```

Here, every thread has its own instance of `threadCounter`.

The initial value of each instance must also satisfy the constant-initialization requirement.

Each thread can then modify its own value:

```cpp
++threadCounter;
```

---

# An advanced example with a static member

Static data members can also use `constinit` in appropriate designs.

For example:

```cpp
struct Config
{
    static inline int timeout;
};

constinit int Config::timeout = 30;
```

However, in modern C++, it is important to understand the differences between `inline static` members and separately defined static data members.

The main point is that `constinit` becomes relevant when the variable has the appropriate storage duration.

---

# An important example: preventing dynamic initialization

Suppose we have a function that produces a value during program execution:

```cpp
int loadConfiguration()
{
    // Read configuration
    return 100;
}

int configuration = loadConfiguration();
```

Here, `configuration` may require dynamic initialization.

If the programmer expects this variable to have its value established before normal program execution begins, it is worth checking whether the value can actually be obtained through constant initialization.

If the function can be evaluated at compile time:

```cpp
consteval int defaultConfiguration()
{
    return 100;
}
```

we can write:

```cpp
constinit int configuration = defaultConfiguration();
```

In this case, both `consteval` and `constinit` have distinct roles.

The `consteval` keyword guarantees that the function is evaluated at compile time.

The `constinit` keyword guarantees that the variable has constant initialization.

---

# An advanced example with static initialization

Suppose a library has the following variable:

```cpp
// config.hpp

extern constinit int globalTimeout;
```

And in the implementation file:

```cpp
// config.cpp

constinit int globalTimeout = 30;
```

Other parts of the program can then use it:

```cpp
#include "config.hpp"

void resetTimeout()
{
    globalTimeout = 30;
}
```

The advantage of this pattern is that an important initialization contract is explicitly expressed in the declaration.

In other words, `constinit` is not merely an optimization hint.

It is a language-level requirement.

If the initializer cannot satisfy the requirements for constant initialization, the program is ill-formed.

---

# constinit is not merely an optimization

A common misconception is:

> `constinit` simply tells the compiler to make the code faster.

That interpretation is not accurate.

The `constinit` keyword creates a **compile-time requirement**.

For example:

```cpp
int getValue();

constinit int value = getValue();
```

The compiler cannot simply accept this code because it might somehow optimize the call away.

The issue is that the initializer must satisfy the requirements for constant initialization.

If it does not, the program is ill-formed.

---

# What problem would we have without constinit?

Before C++20, C++ did not have a dedicated mechanism with the exact semantics of `constinit` that could directly express:

> This variable must remain mutable, but its initial initialization must be constant initialization.

We could use `constexpr`, but `constexpr` makes the variable immutable.

We could use `const`, but `const` by itself does not provide the specific guarantee we want about constant initialization.

We could also use different design techniques to avoid initialization-order problems, but the language did not provide a direct keyword for expressing this particular contract.

Therefore, `constinit` fills a specific gap.

It allows the programmer to separate **whether the variable can be modified** from **how its initial value is established**.

---

# An example that shows the main difference

Consider these four declarations together:

```cpp
const int a = 10;

constexpr int b = 10;

constinit int c = 10;

consteval int getValue()
{
    return 10;
}
```

Each one expresses a different concept.

Variable `a` cannot be modified after initialization.

Variable `b` is also immutable, and its value can participate in constant expressions.

Variable `c` is mutable, but its initialization must be constant initialization.

Function `getValue` is an immediate function, so calls to it must be evaluated at compile time.

Therefore, the similarity in the names of these keywords should not lead us to treat them as interchangeable.

---

# Limitations of constinit

The `constinit` keyword cannot be applied to every variable.

The variable must have **static storage duration** or **thread storage duration**.

Therefore, something like this is not appropriate:

```cpp
void function()
{
    constinit int value = 10;
}
```

A normal local variable does not have the required storage duration.

In contrast, this is an appropriate kind of variable:

```cpp
constinit int globalValue = 10;
```

And this can also be appropriate:

```cpp
thread_local constinit int threadValue = 10;
```

Another important point is that `constinit` does not make a variable a constant expression.

For example, we should not assume that:

```cpp
constinit int value = 10;
```

is always equivalent to:

```cpp
constexpr int value = 10;
```

These declarations have different language semantics.

---

# An important note about constinit and constexpr

The `constexpr` specifier on a variable generally makes that variable `const`.

For example:

```cpp
constexpr int value = 10;
```

In contrast:

```cpp
constinit int value = 10;
```

keeps the variable mutable.

Therefore, if our requirement is:

> The value must never change and must be usable in constant expressions.

`constexpr` is usually the natural choice.

If our requirement is:

> The initial value must use constant initialization, but the variable must remain mutable afterward.

Then `constinit` is the more appropriate tool.

---

# An important note about constinit and const

These two specifiers can also be combined:

```cpp
constinit const int value = 42;
```

Here, we have two separate guarantees.

The `const` keyword says that the variable cannot be modified after initialization.

The `constinit` keyword says that its initialization must be constant initialization.

Therefore:

```cpp
constinit const int value = 42;
```

conceptually expresses the following contract:

> The initial value must be established through constant initialization, and the variable must not be modified afterward.

However, in many situations, `constexpr` is a simpler and more expressive choice for this kind of requirement.

---

# When should we use constinit?

Using `constinit` makes sense when we have a variable with static or thread storage duration and we want to establish an important invariant about its initialization.

A typical appropriate scenario has these characteristics:

```text
global or thread_local variable
        +
initial value must use constant initialization
        +
the variable must remain mutable afterward
```

In such a situation, `constinit` is an excellent choice.

The following scenario is generally not what `constinit` is for:

```text
I simply need a constant value.
```

In that case, `constexpr` is usually the better choice.

Another scenario is:

```text
I simply want the variable to be immutable.
```

In that case, `const` may be sufficient.

Another scenario is:

```text
I want a function to always execute at compile time.
```

In that case, `consteval` is the relevant tool.

---

# Conclusion

The `constinit` keyword is an important C++20 feature for controlling the initialization of variables with static or thread storage duration.

Its main purpose is to **prevent unwanted dynamic initialization** and establish an explicit requirement for constant initialization.

The most important point is that `constinit` does not mean `const`.

For example:

```cpp
constinit int counter = 0;

counter++;
```

The code above is completely valid.

In contrast, `constexpr` is intended for objects that can be used in constant expressions, and a `constexpr` variable is immutable.

The `consteval` keyword is different again and is used for immediate functions.

At a high level, the four concepts can be remembered like this:

```text
const
    "Do not modify me afterward."

constexpr
    "My value can be used during compile time."

consteval
    "This function must be evaluated at compile time."

constinit
    "My initialization must be constant initialization."
```

Therefore, when you have a global or `thread_local` variable whose **initialization must be performed as constant initialization, while the variable itself must remain mutable**, `constinit` is the feature specifically designed for that requirement.

🤝

## Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>