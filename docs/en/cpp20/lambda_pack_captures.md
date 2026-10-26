<div align="center">

[🇺🇸 English](./lambda_pack_captures.md) | [🇮🇷 فارسی](../../fa/cpp20/lambda_pack_captures.md)

</div>

---

# A Comprehensive Guide to Lambda Pack Captures in C++20

## Introduction

Pack Captures in lambda expressions are a feature introduced in C++20 that allows a variable number of values to be captured in a lambda expression. This feature is particularly useful when working with Variadic Templates, template parameters, and functions that accept a variable number of arguments.

Before C++20, directly capturing a Parameter Pack in a lambda's Capture List was not supported by standard syntax. To work around this limitation, developers typically relied on alternatives such as creating a Tuple or using a Helper Function.

With Pack Captures, we can expand a Parameter Pack directly in the Capture List and use its elements inside the lambda body.

For example, in C++20, we can write:

```
#include <iostream>
#include <utility>

template <typename... Args>
auto makeLambda(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        ((std::cout << values << ' '), ...);
        std::cout << '\n';
    };
}

int main()
{
    auto lambda = makeLambda(10, 20, 30);
    lambda();
}
```

Output:

```
10 20 30
```

In this example, each argument becomes a separate member of the Closure Object. The lambda can subsequently access these captured values without depending on the lifetime of the local variables in the `makeLambda` function.

It is important to understand that Pack Captures are not, by themselves, a new memory allocation or memory management mechanism. Instead, they are a language feature that extends the syntax of lambda captures and combines with existing mechanisms.

## Understanding Lambda Captures and the Role of Closure Objects

To understand Pack Captures correctly, we first need to understand how captures work in a lambda.

A lambda expression in C++ creates a Closure Object, an object of a unique, unnamed type defined by the compiler. Variables included in the Capture List become part of the state stored in this object and can be accessed through the lambda.

For example:

```
int value = 42;

auto lambda = [value] {
    return value;
};
```

In this example, `value` is captured by value and stored in the Closure Object. Subsequent changes to the original variable do not affect the captured value.

In contrast, capturing by reference allows access to the original variable:

```
int value = 42;

auto lambda = [&value] {
    return value;
};

value = 100;

std::cout << lambda(); // 100
```

Therefore, a capture determines how a lambda accesses the state it needs. Pack Captures extend this capability by allowing multiple values to be handled as a Parameter Pack in the Capture List.

## Understanding Parameter Packs in C++

Variadic Templates were introduced in C++11 and allow us to write templates that accept a variable number of parameters.

For example:

```
template <typename... Types>
void process(Types... values)
{
}
```

Here, `Types` is a Template Parameter Pack, and `values` is a Function Parameter Pack.

When the function is called as follows:

```
process(10, 3.14, 'A');
```

the arguments correspond to three different types:

- `int`
- `double`
- `char`

In many cases, we need to transfer these values into a lambda so they remain available after the current function finishes executing. Pack Captures provide a direct way to achieve this.

## Introducing Pack Capture Syntax

In C++20, we can use the pack expansion operator `...` in the Capture List to expand a Parameter Pack into a sequence of captures.

The general syntax is:

```
[...capturedPack = expressionPack] {
    // Use capturedPack...
}
```

In this syntax, `expressionPack` must be an expression that can be expanded into a sequence of initializers. A separate captured value is created for each element of the pack.

An important detail is that `capturedPack` remains a pack inside the lambda body. Therefore, when accessing its elements in a context that requires expansion, we must use `...`.

For example:

```
#include <iostream>
#include <utility>

template <typename... Args>
auto makePrinter(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        ((std::cout << values << '\n'), ...);
    };
}

int main()
{
    auto printer = makePrinter(10, 20, 30);
    printer();
}
```

In this example, `values` is a pack of captured values, and the Fold Expression prints all its elements in order.

### Pack Expansion vs. Pack Capture

These two concepts are related, but they are not identical.

- Pack Expansion is a mechanism for expanding a Parameter Pack in a language context where expansion is permitted.
- Pack Capture uses Pack Expansion in a Capture List to create multiple independent captures in a Closure Object.

For example, the following expression uses Pack Expansion:

```
printAll(values...);
```

The following expression, however, uses Pack Expansion to create individual captures:

```
[...captured = values] {
    // Use captured...
}
```

In the second example, each value corresponding to an element of `values` is stored in the Closure Object as a separate captured value.

## Understanding Init-Capture in Pack Captures

One of the most important applications of this feature is combining it with Init-Capture.

Init-Capture was introduced in C++14 and allows us to create variables directly in the Capture List, even when no variable with the same name exists in the surrounding scope.

For example:

```
auto lambda = [value = 42] {
    return value;
};
```

In C++20, we can extend the same idea to a pack:

```
template <typename... Args>
auto makeLambda(Args... args)
{
    return [...values = args] {
        return sizeof...(values);
    };
}
```

In this example, each element of `args` becomes an individual capture corresponding to an element in the `values` pack.

However, this example receives its parameters by value. To avoid unnecessary copies and preserve appropriate behavior when handling temporary objects, using Forwarding References is generally a better choice:

```
template <typename... Args>
auto makeLambda(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        return sizeof...(values);
    };
}
```

In this version, the Value Category and type of each argument are taken into account when constructing the captures. The final behavior still depends on the actual argument types and the applicable type deduction rules.

## Managing Copy and Move in Pack Captures

One important consideration when using Pack Captures professionally is understanding the exact behavior of Copy and Move.

Consider the following example:

```
template <typename... Args>
auto captureByCopy(Args&&... args)
{
    return [...values = args] {
        // Use values...
    };
}
```

Although `args` is declared using Forwarding References, each named parameter is an Lvalue Expression. Therefore, this approach does not inherently guarantee Move semantics.

If we want to forward each argument according to its deduced type, we should write:

```
template <typename... Args>
auto captureWithForwarding(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        // Use values...
    };
}
```

In this case, if an original argument is an Lvalue, the appropriate copy or construction from an Lvalue is performed. If an argument is an Rvalue, Move semantics can be used.

However, Forwarding only preserves the Value Category correctly. Whether a Copy or Move actually occurs depends on the destination type and the available constructors.

### Transferring Ownership with Move

Suppose we want to transfer an object that cannot be copied into a lambda. In this case, we can use Pack Captures together with `std::move` or perfect forwarding.

```
#include <memory>
#include <utility>

template <typename... Args>
auto makeTask(Args&&... args)
{
    return [...items = std::forward<Args>(args)]() mutable {
        // Work with the captured objects.
        ((items.reset()), ...);
    };
}

int main()
{
    auto task = makeTask(
        std::make_unique<int>(10),
        std::make_unique<int>(20)
    );

    task();
}
```

In this example, the `std::unique_ptr` objects are transferred into the Closure Object. Since `unique_ptr` is not copyable, transferring these objects requires Move semantics.

Also, `mutable` is necessary because, by default, a lambda with value captures exposes those captures through a `const` call operator. Adding `mutable` allows the lambda to modify its captured state.

## Understanding Mutable and Non-Mutable Captures

In ordinary lambdas, value captures cannot be directly modified inside the lambda body by default.

For example:

```
int value = 10;

auto lambda = [value] {
    // value = 20; // Error
    return value;
};
```

Adding `mutable` allows the captured value to be modified:

```
int value = 10;

auto lambda = [value]() mutable {
    value += 5;
    return value;
};
```

The same principle applies to Pack Captures:

```
template <typename... Args>
auto makeIncrementer(Args... args)
{
    return [...values = args]() mutable {
        ((++values), ...);
    };
}
```

In this example, each captured element is modifiable, and its value is incremented every time the lambda is called.

Note that `mutable` does not modify the original variables because their values were copied into the Closure Object. Also, `mutable` only changes the `const` qualification of the lambda's call operator; it does not imply Thread Safety.

## Value and Reference Captures in Pack Captures

An important detail is that Pack Captures do not necessarily mean copying every value. The type of each capture is determined by its initializer.

For example, we can store independent values:

```
template <typename... Args>
auto captureValues(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        // Use owned captured values.
    };
}
```

In this case, the Closure Object stores values independent of the original arguments. These values may be constructed through Copy or Move.

Alternatively, if we want to capture references, we can use `std::ref` and `std::reference_wrapper`, or another appropriate mechanism for preserving references. However, we must be careful about the lifetime of the original objects.

For example:

```
#include <functional>
#include <iostream>
#include <utility>

template <typename... Args>
auto captureReferences(Args&... args)
{
    return [...refs = std::ref(args)] {
        ((std::cout << refs.get() << '\n'), ...);
    };
}

int main()
{
    int first = 10;
    int second = 20;

    auto lambda = captureReferences(first, second);

    first = 100;
    lambda();
}
```

Output:

```
100
20
```

In this example, `refs` contains `std::reference_wrapper` objects that refer to the original variables. Consequently, changes to `first` are reflected in the lambda's output.

Important: Using `std::ref` does not extend the lifetime of the referenced object. If the lambda is called after the original variable has been destroyed, accessing that object is invalid.

## Combining Pack Captures with Generic Lambdas

Generic Lambdas were introduced in C++14 and allow `auto` parameters in a lambda's parameter list. In C++20, we can combine Pack Captures with Generic Lambdas to build more flexible lambda expressions.

For example:

```
#include <iostream>
#include <utility>

auto makePrinter()
{
    return []<typename... Args>(Args&&... args) {
        ((std::cout << args << ' '), ...);
        std::cout << '\n';
    };
}

int main()
{
    auto printer = makePrinter();
    printer(10, 20.5, 'A');
}
```

This example does not use Pack Captures. Instead, it defines a Generic Lambda that accepts a variable number of arguments when called.

This distinction is important from a design perspective:

- Pack Captures store values in a Closure Object.
- Generic Lambdas with Parameter Packs accept a variable number of arguments when invoked.

If we need both capabilities, we can combine them:

```
#include <iostream>
#include <utility>

template <typename... Stored>
auto makeProcessor(Stored&&... stored)
{
    return [...values = std::forward<Stored>(stored)]
           <typename... Args>(Args&&... args) mutable {
        ((std::cout << values << ' '), ...);
        ((std::cout << args << ' '), ...);
        std::cout << '\n';
    };
}

int main()
{
    auto processor = makeProcessor(10, 20);
    processor(30, 40);
}
```

Output:

```
10 20 30 40
```

In this example, the lambda has both stored state and the ability to receive new arguments when invoked.

## Handling Empty Packs and Fold Expressions

Since Pack Captures are commonly used with Variadic Templates, we must also consider their behavior when the pack is empty.

Suppose the following function receives no arguments:

```
template <typename... Args>
auto makeLambda(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        return sizeof...(values);
    };
}
```

The following call is valid:

```
auto lambda = makeLambda();
```

In this case, the pack contains zero elements, and the Closure Object has no captures originating from this pack. Calling `lambda()` returns zero.

However, when using Fold Expressions inside the lambda body, we must also consider the rules governing empty packs.

For example:

```
template <typename... Args>
auto makeSum(Args... args)
{
    return [...values = args] {
        return (values + ... + 0);
    };
}
```

The expression `(values + ... + 0)` is a right fold with an initial value of zero. Therefore, if the pack is empty, the result is zero.

By contrast, unary Fold Expressions without an initial value produce a defined result for empty packs only with certain operators. The operators `&&`, `||`, and `,` have special rules, but we cannot assume that an empty pack produces a result for operators such as `+`.

Recommendation: Explicitly design the behavior of empty packs in generic and library code, especially when the lambda is part of a public API.

## Comparing Pack Captures with Pre-C++20 Techniques

Before C++20, there was no direct way to expand a pack in an Init-Capture List. However, several alternatives could achieve similar results.

### Using a Tuple

One common approach was to store the arguments in a `std::tuple`.

```
#include <tuple>
#include <utility>

template <typename... Args>
auto makeLambda(Args&&... args)
{
    auto values = std::make_tuple(std::forward<Args>(args)...);

    return [values = std::move(values)] {
        return std::tuple_size_v<decltype(values)>;
    };
}
```

This approach works in C++17 and stores the values in a Tuple object.

However, accessing Tuple elements generally requires `std::get`, metaprogramming utilities, or `std::apply`. In contrast, Pack Captures allow each element to be used directly through the pack inside the lambda.

### Using a Helper Function

In some designs, we can create a helper function that receives the arguments and prepares the required state for the lambda. This approach remains valid and can be appropriate when separating responsibilities or handling complex ownership requirements.

However, when the goal is to capture a pack directly, the C++20 feature generally makes the code simpler and more readable.

## Practical Applications of Pack Captures

This feature is useful in scenarios where a variadic function must receive data and retain it in a lambda for later use.

### Creating a Callback with Stored State

Suppose we have an Event or Callback system that needs to store arguments when a callback is registered and use them later.

```
#include <iostream>
#include <utility>

template <typename Function, typename... Args>
auto bindArguments(Function&& function, Args&&... args)
{
    return [...values = std::forward<Args>(args),
            function = std::forward<Function>(function)]() mutable {
        return function(values...);
    };
}

int main()
{
    auto callback = bindArguments(
        [](int a, int b, int c) {
            return a + b + c;
        },
        10, 20, 30
    );

    std::cout << callback() << '\n';
}
```

Output:

```
60
```

In this example, the arguments and the target function are stored in the Closure Object. The example is suitable for stored arguments that are compatible with the function's parameters. A more general design must also consider Copy, Move, and repeated invocations.

### Storing Configuration for a Task

In a Task system, we may need to store the values required to perform an operation so that the operation can be executed later.

```
#include <iostream>
#include <utility>

template <typename... Args>
auto makeTask(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        ((std::cout << values << ' '), ...);
        std::cout << '\n';
    };
}

int main()
{
    auto task = makeTask(10, 20, 30);
    task();
}
```

In this example, the task does not depend on the local variables of the factory function because it stores the required values in its Closure Object.

However, if the stored values include pointers, references, or handles to external resources, the validity and lifetime of those resources must still be managed.

## Common Mistakes When Using Pack Captures

### Forgetting Perfect Forwarding

In a function template, using `args` by itself generally does not perform forwarding.

```
template <typename... Args>
auto makeLambda(Args&&... args)
{
    return [...values = args] {
        // Use values...
    };
}
```

In this example, the initializers are built from the named Lvalue Expressions `args`. To preserve the Value Category of the input arguments, we generally need to use `std::forward<Args>(args)`.

### Confusing the Captured Pack with the Input Arguments

After the lambda is created, the captured pack becomes part of the Closure Object's state. Changing the original variables does not modify value captures, and modifying the captures does not automatically modify the original variables.

### Ignoring Reference Lifetimes

If the pack contains reference wrappers, pointers, or similar objects, storing them in a Closure Object does not guarantee that their target objects remain valid.

### Misusing `mutable`

The `mutable` specifier allows value captures to be modified inside the lambda body. However, it does not provide shared ownership, synchronization between threads, or concurrency safety.

### Assuming Pack Captures Are Only for Copying

By combining Init-Capture with tools such as `std::move` and `std::forward`, we can also transfer ownership of objects. The actual behavior depends on the argument types, initializers, and available constructors.

## Limitations and Design Considerations

Despite its usefulness, Pack Captures have several important considerations:

- This feature has been part of the standard language since C++20. The compiler and project configuration must support C++20 to use it.
- Pack Captures do not, by themselves, solve lifetime, ownership, or external resource management problems.
- Each captured value's type is determined by its corresponding initializer. Generic templates must therefore account for references, `const` qualification, Copy, and Move.
- If a lambda is expected to execute multiple times, we must determine whether the captured state is preserved, modified, or consumed during each invocation.
- In public APIs, the behavior of empty packs and non-copyable objects must also be considered.

## Final Summary

Pack Captures in C++20 provide a standard and readable way to capture a variable number of values in lambda expressions. This feature is particularly useful when combined with Variadic Templates, Forwarding References, and Init-Capture.

With Pack Captures, we can store the state required by callbacks, tasks, and deferred operations directly in a Closure Object without necessarily relying on Tuples or additional helper utilities.

The most important aspects of using Pack Captures professionally are understanding the relationship between Pack Expansion, Value Categories, Copy and Move semantics, and the lifetime of captured objects. Mastering these concepts helps us design more generic, safer, and more readable lambdas in modern C++.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>