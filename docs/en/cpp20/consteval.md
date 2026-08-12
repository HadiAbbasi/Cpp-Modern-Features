<div align="right">

[🇺🇸 English](./consteval.md) | [🇮🇷 فارسی](../../fa/cpp20/consteval.md)

</div>

---

# A Simple Guide to `consteval` in C++

## Table of Contents

* [`consteval` چیست؟](#what-is-consteval)
* [Why Was `consteval` Introduced?](#why-was-consteval-introduced)
* [What Problem Does `consteval` Solve?](#what-problem-does-consteval-solve)
* [How Does `consteval` Work?](#how-does-consteval-work)
* [The Difference Between `consteval` and `const`](#the-difference-between-consteval-and-const)
* [Can the Value of `consteval` Be Changed Later?](#can-the-value-of-consteval-be-changed-later)
* [The Difference Between `consteval` and `constexpr`](#the-difference-between-consteval-and-constexpr)
* [A More Practical Example](#a-more-practical-example)
* [What Is the Benefit of `consteval`?](#what-is-the-benefit-of-consteval)
* [Does `consteval` Make Everything Faster?](#does-consteval-make-everything-faster)
* [An Important Note About Compilation](#an-important-note-about-compilation)
* [A Common Misunderstanding](#a-common-misunderstanding)
* [When Should We Use `consteval`?](#when-should-we-use-consteval)
* [Summary](#summary)

## What Is `consteval`?

The `consteval` keyword was introduced in **C++20**.

The `consteval` keyword is used to define **functions that must be executed at compile time**.

In simple terms, when we define a function with `consteval`, we are telling the compiler:

> “This function must be executed at compile time and must never be called at runtime.”

Here is a simple example:

```cpp
consteval int square(int x)
{
    return x * x;
}

int main()
{
    constexpr int a = square(5);

    return 0;
}
```

In this example, the compiler knows that `square(5)` must be evaluated at compile time.

The result is already known:

```text
25
```

Therefore, the value `25` can be determined during compilation.

---

## Why Was `consteval` Introduced?

To understand why `consteval` exists, it helps to first look at `constexpr`.

In C++, we can define a function using `constexpr`:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

Such a function **can** be evaluated at compile time, but it is not necessarily required to be.

For example:

```cpp
constexpr int a = square(5);
```

Here, the compiler must evaluate the result at compile time.

However, this is also valid:

```cpp
int x = 10;

int y = square(x);
```

In this case, the value of `x` is determined at runtime, so `square` can also be executed at runtime.

This flexibility of `constexpr` is sometimes not what the programmer wants.

Sometimes we want to make absolutely sure that a function **can never be executed at runtime**.

That is where `consteval` comes in.

---

## What Problem Does `consteval` Solve?

Suppose we have a function that processes information that must be known at compile time.

For example:

```cpp
consteval int calculate_size(int x)
{
    return x * 10;
}
```

This is valid:

```cpp
constexpr int size = calculate_size(5);
```

But this causes a problem:

```cpp
int x = 5;

int size = calculate_size(x);
```

The reason is that the value of `x` is only known at runtime.

A `consteval` function is **not allowed to run at runtime**.

Therefore, the compiler will produce an error.

This is one of the most important features of `consteval`.

With `consteval`, we can create an **explicit compile-time requirement**.

---

## How Does `consteval` Work?

Suppose we have this function:

```cpp
consteval int add(int a, int b)
{
    return a + b;
}
```

This call is perfectly valid:

```cpp
int result = add(10, 20);
```

At first, this might seem a little strange.

The `result` variable is an ordinary variable and is not declared as `constexpr`.

The important point is that **the `add` function itself must be evaluated at compile time**.

The compiler can calculate:

```text
10 + 20 = 30
```

and then use `30` as the value of `result`.

Therefore, we should keep these two concepts separate:

```cpp
consteval int add(int a, int b);
```

means:

> “The function itself must be evaluated at compile time.”

Whereas:

```cpp
constexpr int result = add(10, 20);
```

means:

> “This variable must have a constant expression value that can be determined at compile time.”

---

## The Difference Between `consteval` and `const`

The difference between `consteval` and `const` is very important because they describe completely different concepts.

The `const` keyword is generally about **preventing a value from being modified after initialization**.

For example:

```cpp
const int x = 10;
```

After initialization, we cannot write:

```cpp
x = 20;
```

However, `const` does **not** necessarily mean that the value must be determined at compile time.

For example:

```cpp
int getNumber()
{
    return 10;
}

const int x = getNumber();
```

Here, `x` is constant and cannot be modified after initialization.

But its value could have been obtained from a function executing at runtime.

In contrast, `consteval` is about the **execution time of a function**.

For example:

```cpp
consteval int getNumber()
{
    return 10;
}
```

This function must be evaluated at compile time.

So, in a simplified form:

| Feature                           | `const`                  | `consteval`                           |
| --------------------------------- | ------------------------ | ------------------------------------- |
| Applies to                        | Objects/values           | Functions                             |
| Can the value be changed?         | No, after initialization | Not applicable to the function itself |
| Requires compile-time evaluation? | No                       | Yes                                   |
| Introduced in                     | Early C++                | C++20                                 |
| Main purpose                      | Prevent modification     | Force compile-time evaluation         |

The most important point is:

> `const` and `consteval` are not alternatives to each other.

---

## Can the Value of `consteval` Be Changed Later?

There is a common misunderstanding here.

`consteval` itself **does not have a value that can later be changed**.

`consteval` is used for **functions**, not variables.

For example:

```cpp
consteval int getNumber()
{
    return 10;
}
```

Here, `getNumber` is a `consteval` function.

We can store the result of calling it in an ordinary variable:

```cpp
int x = getNumber();
```

Now `x` is an ordinary variable.

Therefore, this is perfectly valid:

```cpp
x = 20;
```

In this case, we have changed the value of the **result of the `consteval` function**, not the value of `consteval` itself.

Here is a complete example:

```cpp
consteval int getNumber()
{
    return 10;
}

int main()
{
    int x = getNumber();

    x = 20;

    return 0;
}
```

This is valid C++.

So, more precisely:

> The result of a `consteval` function can be stored in an ordinary variable, and that variable can later be modified.

But:

> A `consteval` function itself is not a variable whose value can be changed.

---

## The Difference Between `consteval` and `constexpr`

Probably the most important comparison when learning `consteval` is the comparison with `constexpr`.

Suppose we have these two functions:

```cpp
constexpr int square1(int x)
{
    return x * x;
}

consteval int square2(int x)
{
    return x * x;
}
```

The `square1` function with `constexpr` **can** be evaluated at compile time.

However, if compile-time evaluation is not possible, it may also be executed at runtime.

For example:

```cpp
int x = 5;

int a = square1(x);
```

This can be valid.

The `square2` function, however, does not have this flexibility:

```cpp
int x = 5;

int b = square2(x);
```

This will produce an error.

The reason is that `square2` **must** be evaluated at compile time, but the value of `x` is not known until runtime.

Therefore, a simple way to remember the difference is:

```text
constexpr  →  “Evaluate it at compile time if possible.”

consteval  →  “You must evaluate it at compile time.”
```

This is a simplified explanation, but it is very useful when first learning the difference.

---

## A More Practical Example

Suppose we want to calculate a value based on several inputs, and we know that these inputs must be available at compile time.

We could write:

```cpp
consteval int buffer_size(int width, int height)
{
    return width * height;
}
```

Then:

```cpp
constexpr int size = buffer_size(10, 20);
```

The value of `size` is:

```text
200
```

But if we try to get the value from user input:

```cpp
int width;
std::cin >> width;

int size = buffer_size(width, 20);
```

we have a problem.

The reason is straightforward:

```text
User input
    ↓
Runtime
    ↓
width becomes known
```

But `consteval` says:

```text
I must be evaluated right now, at compile time.
```

Therefore, such a function call is not allowed.

This feature makes it possible to detect certain mistakes much earlier, during compilation, rather than discovering them only when the program is running.

---

## What Is the Benefit of `consteval`?

One of the biggest benefits of `consteval` is that it creates a **very clear contract for a function**.

When a programmer sees this:

```cpp
consteval int calculate(int x)
{
    return x * 2;
}
```

they immediately know that this function is not intended to be part of runtime computation.

The function must be evaluated at compile time.

This can be useful for things such as:

* Compile-time calculations
* Generating constant values
* Compile-time validation
* Building APIs whose arguments must be known at compile time
* Preventing certain calculations from accidentally happening at runtime

---

## Does `consteval` Make Everything Faster?

No, not necessarily.

It would be inaccurate to say that using `consteval` always makes a program faster.

The primary purpose of `consteval` is not to magically improve performance.

Its main purpose is to **move execution to compile time and make that requirement mandatory**.

If a calculation can be performed at compile time, then that calculation does not need to be performed again at runtime.

However, in many situations, the compiler can already perform similar optimizations on its own.

Therefore, one of the most important benefits of `consteval` is not simply performance.

The more important benefit is **guaranteeing and controlling when a function is evaluated**.

---

## An Important Note About Compilation

When we say that a `consteval` function is executed at compile time, we do not mean that the C++ program is actually running as a separate program during compilation.

The compiler evaluates the function as part of the compilation process and calculates its result.

For example:

```cpp
consteval int multiply(int a, int b)
{
    return a * b;
}

int x = multiply(5, 6);
```

The compiler can evaluate `multiply(5, 6)` during compilation.

The result is:

```text
30
```

The final program does not need to execute this function at runtime for this particular call.

---

## A Common Misunderstanding

Sometimes `consteval` is misunderstood as meaning:

```text
“This value is constant at compile time and can never be changed later.”
```

That is not what `consteval` means.

For example:

```cpp
consteval int getValue()
{
    return 100;
}

int x = getValue();

x = 500;
```

This is perfectly valid.

The `getValue` function was evaluated at compile time.

But `x` is an ordinary variable and can therefore be modified later.

If our goal is to make the **variable itself immutable**, that is a different concern:

```cpp
const int x = getValue();
```

Here, `const` prevents `x` from being modified.

If we want the variable's value to also be a compile-time constant, `constexpr` becomes relevant:

```cpp
constexpr int x = getValue();
```

So we can separate these three concepts:

```text
const
↓
Do not modify this value after initialization.

constexpr
↓
This value must be a constant expression that can be evaluated at compile time.

consteval
↓
This function must be evaluated at compile time.
```

---

## When Should We Use `consteval`?

The best time to use `consteval` is when we want to **guarantee that a function is called only in a compile-time context**.

If we simply want the compiler to evaluate a function at compile time when possible, `constexpr` is usually more flexible.

For example:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

This is useful for a general-purpose function that can work with both compile-time and runtime values.

However:

```cpp
consteval int square(int x)
{
    return x * x;
}
```

is useful when **preventing runtime execution is part of the function's contract**.

---

## Summary

`consteval` is a C++20 feature that **forces a function to be evaluated at compile time**.

The most important thing to remember is that `consteval` is used for **functions**, not variables.

The main difference between `consteval` and `constexpr` is:

```text
constexpr
→ The function can be evaluated at compile time.

consteval
→ The function must be evaluated at compile time.
```

`consteval` is also fundamentally different from `const`:

```text
const
→ Prevents a value from being modified after initialization.

consteval
→ Forces a function to be evaluated at compile time.
```

So, when you see:

```cpp
consteval int calculate(int x)
{
    return x * 2;
}
```

remember this sentence:

> **“Whenever I call this function, the compiler must be able to evaluate it at compile time.”**

If the input value is only available at runtime, calling a `consteval` function with that value is not allowed.

Therefore, `consteval` is **not a tool for making a value immutable**.

It is a tool for **forcing computation to happen at compile time**.

🤝

## Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>