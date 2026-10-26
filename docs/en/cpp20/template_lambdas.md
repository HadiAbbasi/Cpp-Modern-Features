<div align="center">

[🇺🇸 English](./template_lambdas.md) | [🇮🇷 فارسی](../../fa/cpp20/template_lambdas.md)

</div>

---

# Lambda Templates in C++20

## Introduction

Lambda expressions have provided a way to define unnamed, usually short functions directly where they are needed since C++11. In C++14, the introduction of generic lambdas made it possible to write callables that work with multiple types by using `auto` parameters. C++20 further expanded this mechanism with features such as explicit template parameter lists, `requires` constraints, the use of lambdas in unevaluated contexts, improved support for captureless closures, and an expansion of `init-capture` to support parameter packs.

The purpose of these improvements is not merely to make code shorter. They allow lambdas to play a more precise and controllable role in generic programming, API design, metaprogramming, standard algorithms, and compile-time code.

## Overview of the Lambda Mechanism

A lambda expression creates an object of an unnamed type, commonly called a closure object. The type of this object, the closure type, is defined by the compiler and includes a mechanism for invoking the lambda body. When surrounding variables are captured, the closure usually contains data used to store them.

A basic example:

```
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 5, 2, 8, 3};

    auto is_even = [](int value) {
        return value % 2 == 0;
    };

    const auto count = std::count_if(values.begin(), values.end(), is_even);
    return count == 2 ? 0 : 1;
}
```

In this example, the variable `is_even` is a closure object. It behaves like a callable function, and because it captures no variables, its closure has no stored state from the surrounding environment.

### Ordinary Lambdas and Generic Lambdas

Since C++14, a lambda is considered a `generic lambda` when one or more of its parameter types are `auto`. In this case, the closure's call operator is a function template, and the parameter types are deduced at each invocation.

```
auto twice = [](auto value) {
    return value + value;
};

auto a = twice(10);    // int
auto b = twice(2.5);   // double
```

In the example above, the parameter type is deduced separately for each invocation. Before C++20, however, there was no direct way to control and name template parameters in the lambda's own syntax. The new capability removes this limitation.

## Introducing Explicit Template Parameter Lists

The most important change to generic lambdas in C++20 is support for an explicit template parameter list. This syntax allows a lambda to declare template parameters directly, much like an ordinary function template.

```
auto combine = []<typename T>(T left, T right) {
    return left + right;
};

auto result = combine(10, 20);
```

In this code, both parameters use the same `T`. This differs from `[](auto left, auto right)`: in the latter form, each parameter has its own template parameter, so the two arguments may have different types, provided the body is valid for those types.

### Use Cases Enabled by Explicit Template Parameters

An explicit template parameter list is useful when parameter types must be deduced consistently, a type must be named in the body, non-type template parameters are needed, or `Concepts` constraints can be expressed more clearly.

```
#include <concepts>

auto add_integral = []<std::integral T>(T left, T right) {
    return left + right;
};

int main() {
    const auto sum = add_integral(10, 20);
    // add_integral(1.5, 2.5); // Does not satisfy std::integral
    return sum == 30 ? 0 : 1;
}
```

In this example, `T` must satisfy the `std::integral` concept. As a result, the type requirement is part of the function's interface, and invalid argument types can be rejected during overload resolution rather than merely producing a confusing error inside the body.

The explicit list can also contain non-type template parameters:

```
auto scale = []<typename T, int Factor>(T value) {
    return value * Factor;
};

// A generic lambda's call operator is a member template.
// Explicit template arguments are supplied through operator().
int main() {
    auto value = scale.operator()<int, 3>(7);
    return value == 21 ? 0 : 1;
}
```

In C++20, template parameters cannot be supplied directly to a lambda object by writing `scale<int, 3>(7)`. To specify them explicitly, the member template call operator must be invoked. In a dependent context, the `template` keyword may be needed to disambiguate the syntax.

### Type Deduction and Choosing Between `auto` and `T`

Using `auto` for each parameter is usually the simpler and more flexible choice. An explicit `T` is appropriate when the type relationship between multiple parameters matters, or when `T` itself must be referenced in a `requires` clause or in the body.

```
auto independent = [](auto first, auto second) {
    return first == second;
};

auto same_type = []<typename T>(T first, T second) {
    return first == second;
};
```

In the first example, `first` and `second` may have different types, provided the comparison expression is valid. In the second example, deduction of `T` from both parameters must be consistent with a single type, so some calls with arguments of different types are rejected.

This distinction should be intentional: using `T` does not make a lambda inherently better; it imposes a constraint on how the lambda can be called.

## Constraining Lambdas with Concepts and `requires`

C++20 allows a `requires-clause` to be written in a generic lambda. The constraint can appear after the template parameter list or after the function parameter list.

```
#include <concepts>

auto integral_add = []<typename T>(T left, T right)
    requires std::integral<T>
{
    return left + right;
};

auto ordered = [](const auto& left, const auto& right)
    requires requires { left < right; }
{
    return left < right;
};
```

In the first example, `requires` constrains the type `T`. In the second, a `requires-expression` checks whether the expression `left < right` is valid. This pattern is useful for generic code that should only be callable with types supporting a particular operation.

### The Difference Between Template Constraints and Function Constraints

In a lambda with an explicit template parameter list, a constraint placed before the function parameter list constrains the template parameters. A constraint placed after the function parameter list constrains whether the callable specialization is viable, based on its signature and deduced types.

```
#include <concepts>
#include <type_traits>

auto integral_identity = []<std::integral T>(T value) {
    return value;
};

auto numeric_identity = [](auto value)
    requires std::is_arithmetic_v<decltype(value)>
{
    return value;
};
```

The second example requires `<type_traits>`. The practical difference is that the first form connects the constraint directly to the named template parameter, while the second uses the deduced parameter type.

Using `requires` is appropriate for expressing genuine semantic requirements. Adding complex, unnecessary constraints can reduce readability and make diagnostics worse. Also, `requires` does not guarantee that every error in the lambda body becomes a manageable overload-resolution failure. Expressions used only in the body must still be valid for the selected specialization.

## Using Lambdas in Unevaluated Contexts

Before C++20, placing a lambda expression in certain unevaluated contexts, such as `decltype`, was not permitted. Since C++20, lambdas can be used in these contexts, which is useful when defining type aliases and in some metaprogramming patterns.

```
#include <type_traits>

using Transform = decltype([](int value) {
    return value * 2;
});

static_assert(std::is_empty_v<Transform>);
static_assert(std::is_default_constructible_v<Transform>);

int main() {
    Transform transform{};
    return transform(21) == 42 ? 0 : 1;
}
```

In this example, the lambda expression inside `decltype` is not evaluated; the goal is to obtain the closure type. In C++20, an object of this captureless closure type can then be default-constructed.

Every lambda expression has its own distinct closure type, even when two expressions look identical. Therefore, two independent expressions such as `[](int x) { return x; }` do not necessarily produce the same type. If the same type must be used in multiple places, define it once through an alias or a variable and reuse that type.

## Changes to Captureless Closures

Another C++20 improvement is that the closure type of a captureless lambda can be default-constructible and copy-assignable. These capabilities were not available in the same way in earlier standards.

```
int main() {
    auto is_positive = [](int value) {
        return value > 0;
    };

    auto another = decltype(is_positive){};
    another = is_positive;
    return another(1) ? 0 : 1;
}
```

This capability is useful in APIs that store a callable as a policy, predicate, or template parameter and need to make it easy to construct or replace that callable.

### Use in Standard Templates

For example, a captureless closure type can be used as a predicate in mechanisms that store the callable type and require it to be constructible and assignable. This is particularly useful alongside the use of lambdas as template arguments.

However, not every lambda is assignable. A lambda with captures usually has stored state in its closure, and its assignment operator is not available in the same way as that of a captureless closure. This distinction matters when designing containers and custom wrappers.

## Using Captureless Lambdas as Non-type Template Parameters

Since C++20, the closure type of a captureless lambda is a structural type. Together with C++20 support for class types as non-type template parameters (NTTPs), this allows a captureless lambda expression itself to be used as a template argument.

```
template<auto Operation>
struct ConstantOperation {
    template<typename T>
    static constexpr auto apply(T value) {
        return Operation(value);
    }
};

using SquareOperation = ConstantOperation<[](int value) {
    return value * value;
}>;

static_assert(SquareOperation::apply(6) == 36);
```

In this example, `Operation` is a non-type template parameter whose value is a captureless callable. Behavior can therefore become part of a specialization's identity at compile time, without requiring a function pointer or type erasure such as `std::function`.

### Structural Type Limitations

The key requirement is that the lambda has no captures. If a closure captures a variable, its type is not a structural type and it cannot be used as an NTTP in this way. The template argument must also satisfy the other requirements for a constant template argument; being captureless alone does not make every possible use valid.

This technique is useful for small, fixed policies, compile-time computations, and generic algorithms with well-defined behavior. By contrast, if behavior changes at runtime or runtime data must be stored, an ordinary callable object or function parameter is usually simpler and more appropriate.

## Expanding Init-capture to Support Parameter Packs

Since C++14, `init-capture` has allowed a closure to create a variable with a new name. For example, an argument can be stored in the capture using `std::move` or `std::forward`. In C++20, the init-capture syntax was expanded to support `pack expansion`, allowing a set of arguments to be captured at once.

```
#include <tuple>
#include <type_traits>
#include <utility>

template<typename... Args>
auto make_tuple_factory(Args&&... args) {
    return [...values = std::forward<Args>(args)]() mutable {
        return std::tuple<std::decay_t<Args>...>{std::move(values)...};
    };
}

int main() {
    auto make_values = make_tuple_factory(10, 2.5, 30L);
    auto values = make_values();
    return std::get<0>(values) == 10 ? 0 : 1;
}
```

In this example, each element of `args...` is forwarded appropriately into its corresponding capture. The lambda then creates a tuple by moving the stored values. This approach is useful for building tasks, callbacks, or factories that need to retain a set of arguments for later use.

### Important Considerations When Capturing a Pack

Because the closure stores the arguments in this example, the lifetime of the captured values is independent of the lifetime of the factory function's parameters. This is one of the key benefits of the approach. However, copying or moving each argument depends on that argument's type semantics, and move-only types must be handled correctly.

The factory is also deliberately `mutable` and passes its stored members to `std::move` when invoked. As a result, subsequent calls on the same closure may observe moved-from values or states. If the callable must run multiple times without consuming its state, its copy and move policy should be designed to match that requirement.

## Managing Explicit `this` Capture and Avoiding Implicit Capture Pitfalls

In C++, a default capture of `[=]` does not mean that the entire current object is copied. If a lambda body accesses non-static class members, `this` is generally captured implicitly. As a result, the closure stores the `this` pointer and continues to access the original object.

Since C++20, the capture can be written as `[=, this]` to make this intent explicit. In the same standard, implicit capture of `this` through `[=]` was deprecated because it can be mistaken for copying the object itself.

```
#include <functional>

class Counter {
public:
    explicit Counter(int value) : value_(value) {}

    auto make_incrementer() {
        return [=, this](int amount) {
            value_ += amount;
            return value_;
        };
    }

private:
    int value_;
};

int main() {
    Counter counter{10};
    auto increment = counter.make_incrementer();
    return increment(5) == 15 ? 0 : 1;
}
```

In this example, `[=, this]` captures the `this` pointer, and modifying `value_` changes the `Counter` object itself. The member values are not copied into the closure.

### The Difference Between Capturing `this` and Capturing `*this`

The `[*this]` capture was introduced in C++17 and differs fundamentally from `[=, this]`. With `[=, this]`, the closure stores the `this` pointer; with `[*this]`, the current object itself is copied into the closure.

Copying the object can be useful for taking a snapshot of its state, but it depends on the object being copy-constructible and on the intended semantics. If the object manages an external resource, copying it does not necessarily make that resource independent. In both cases, lifetime must be considered: if a closure that captures `this` as a pointer is used after the object has been destroyed, it can cause a dangling pointer and undefined behavior.

### Choosing an Appropriate Capture Strategy

For a short, synchronous callback that executes during the object's lifetime, capturing `this` may be a natural choice. For callbacks that are stored, sent to another thread, or may continue running after the object is destroyed, ownership and lifetime should be designed explicitly. For example, independent state can be copied, or a `std::shared_ptr` can be captured when appropriate for the ownership model.

Using `[=, this]` alone does not solve lifetime problems; this syntax only makes the capture intent clearer.

## `constexpr` Evaluation and `consteval` Lambdas

Since C++17, a lambda can be implicitly `constexpr` if it satisfies the relevant requirements; the `constexpr` keyword does not necessarily have to be written. C++20 extends this model by allowing `consteval` to be specified for a lambda's call operator.

```
auto square_at_compile_time = [](int value) consteval {
    return value * value;
};

constexpr int result = square_at_compile_time(9);
static_assert(result == 81);
```

In this example, the call must be a valid immediate invocation, meaning that the result is computed at compile time. Unlike an ordinary `constexpr` lambda, which may also be called at runtime when permitted, a `consteval` function is intended for immediate invocations.

### The Difference Between `constexpr` and `consteval`

The `constexpr` keyword makes compile-time evaluation possible, but it does not by itself guarantee that every call happens at compile time. By contrast, `consteval` makes the call operator an immediate function and does not permit an invalid runtime call.

For simple computational functions, `constexpr` usually provides more flexibility. `consteval` is appropriate when compile-time evaluation is part of the API contract, for example, when creating constant values or validating constant inputs.

### Limitations of Compile-time Evaluation

Writing `constexpr` or `consteval` does not remove the language's restrictions. The body and the operations it uses must be permitted for the intended kind of evaluation, and runtime resources or values cannot be arbitrarily incorporated into compile-time computation. These specifiers should therefore be selected according to the function's actual contract, not merely in the hope of improving performance.

## Comparing the Main C++20 Improvements

| Feature                                             | Introduced in | Problem it solves                                                             | Design consideration                                                            |
| --------------------------------------------------- | ------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Explicit template parameter lists                   | C++20         | Type control, naming template parameters, and expressing type relationships   | Use when `auto` is not sufficiently expressive or precise                       |
| `requires` constraints in lambdas                   | C++20         | Restricting acceptable types and improving generic interfaces                 | Define constraints according to the API's actual requirements                   |
| Lambdas in unevaluated contexts                     | C++20         | Extracting a closure type through `decltype` and related patterns             | Each lambda expression still has a unique type                                  |
| Construction and assignment of captureless closures | C++20         | Making stateless callables easier to construct and replace                    | Do not generalize this behavior to closures with captures                       |
| Captureless lambdas as NTTPs                        | C++20         | Making fixed behavior part of a template specialization's identity            | The closure must be captureless, and the argument must be valid at compile time |
| Pack expansion in init-capture                      | C++20         | Storing a set of variadic arguments in a closure                              | Consider ownership, move semantics, and whether state is consumed               |
| Explicit capture with `[=, this]`                   | C++20         | Clarifying the distinction between capturing variables and the `this` pointer | The lifetime of the target object must still be guaranteed                      |
| `consteval` lambdas                                 | C++20         | Requiring invocation at compile time                                          | Appropriate when compile-time evaluation is a genuine API requirement           |

## Practical Patterns in Software Design

### Defining Generic Predicates

When a predicate should only be valid for a particular set of types, combining an explicit template parameter list with `requires` keeps the contract alongside the implementation. This pattern is especially useful in algorithms and general-purpose utilities because it reduces errors caused by instantiating the body for unsuitable types.

```
#include <concepts>
#include <algorithm>
#include <vector>

auto is_positive = []<std::integral T>(T value) {
    return value > 0;
};

int main() {
    std::vector<int> values{-2, 0, 4, 7};
    const auto count = std::count_if(values.begin(), values.end(), is_positive);
    return count == 2 ? 0 : 1;
}
```

### Building Compile-time Policies

In some APIs, a template parameter determines how an algorithm behaves. Using a captureless lambda as an NTTP can be a suitable alternative to creating separate policy types, especially when the behavior is short and local. This avoids type erasure, but each distinct behavior generally produces a distinct specialization. Consequently, a large number of unique behaviors may increase the size of the compiled code.

### Building Callbacks That Own Their Arguments

Init-capture together with pack expansion is useful for building callbacks that must retain parameters for later execution. For asynchronous callbacks, correct capture alone is not enough: it must be clear whether arguments are copied or moved, who owns the resources, and whether the callback may be invoked more than once.

## Common Mistakes and Design Considerations

- Confusing `[=]` with copying the entire object: This capture does not, by itself, copy the current object's members into the closure. To copy the object itself, use `[*this]` while accounting for the consequences of copying.
- Ignoring lifetime: Capturing the `this` pointer or a reference to a local variable does not extend the target's lifetime. Keeping the closure after the target's lifetime ends can result in undefined behavior.
- Using explicit template parameters without a reason: If each parameter is independent and naming `T` offers no benefit, `auto` may be more readable.
- Making incorrect assumptions about closure types: Two lambda expressions that look identical do not have the same type. To rely on a particular closure type, reuse the same expression or a type alias.
- Assuming all closures are assignable: In C++20, default construction and copy assignment apply to captureless closures, not to every lambda.
- Using `consteval` without a contractual need: This specifier removes the freedom to invoke the function at runtime. If both modes are required, `constexpr` is more flexible.
- Forgetting the effect of moves in init-capture: When captured members are consumed through `std::move`, invoking the callback again may observe moved-from values. Decide whether the callback is intended for one-time or repeated use.
- Designing unnecessarily complex constraints: `requires` should express an understandable contract. Unnecessary constraints can make overload resolution and diagnostics harder to understand.
- Assuming complete compiler support: C++20 support depends on the compiler version, standard library, and implementation status of individual features. Enabling `-std=c++20` alone does not guarantee that every detail is correctly implemented in every older compiler version.

## Checking Compiler Support and Feature-test Macros

Feature-test macros can be used to detect some capabilities at compile time. An implementation that supports explicit template parameter lists for generic lambdas should define `__cpp_generic_lambdas` to at least `201707L`. For pack expansion in init-capture, `__cpp_init_captures` should be at least `201803L`.

```
#if defined(__cpp_generic_lambdas) && __cpp_generic_lambdas >= 201707L
#define HAS_EXPLICIT_LAMBDA_TEMPLATE_LIST 1
#else
#define HAS_EXPLICIT_LAMBDA_TEMPLATE_LIST 0
#endif

#if defined(__cpp_init_captures) && __cpp_init_captures >= 201803L
#define HAS_LAMBDA_INIT_CAPTURE_PACK 1
#else
#define HAS_LAMBDA_INIT_CAPTURE_PACK 0
#endif
```

These macros are useful for portable codebases that target multiple compilers or language-standard versions. In a project whose minimum standard is C++20, it is usually better to document the required minimum compiler version and test it in CI rather than scattering preprocessor conditionals throughout the codebase.

## Distinguishing C++20 Features from Later Standards

Some capabilities discussed in connection with lambdas actually belong to later standards and should not be confused with C++20 improvements. For example, a `static` lambda call operator and an explicit object parameter for a lambda are features of C++23. Capturing `*this` with `[*this]` was introduced in C++17, although it remains relevant when considering capture rules in C++20.

Understanding these boundaries is essential when setting `-std=c++20`, reviewing code, and interpreting compiler diagnostics. A feature valid in C++23 is not necessarily available in C++20 mode.

## Summary

The C++20 improvements make lambdas more than a concise way to write callbacks: they become a more powerful tool for generic programming and metaprogramming. Explicit template parameter lists and `requires` allow more precise interfaces; captureless closures can be used in type-related contexts and as NTTPs; init-capture now supports parameter packs; and the rules around `this` make programmer intent clearer.

In professional code, syntax choices should reflect the API contract, the need for type control, data lifetimes, copy and move semantics, and the potential cost of generating many specializations. These features are most valuable when they make code more precise and safer, rather than merely more complicated.

## Recommended Further Reading

- [Lambda expressions reference on cppreference](https://en.cppreference.com/w/cpp/language/lambda?utm_source=chatgpt.com)
- [Reference on changes from C++17 to C++20](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p2131r0.html?utm_source=chatgpt.com)
- [Proposal P0428R2: Template parameter lists for generic lambdas](https://wg21.link/P0428R2?utm_source=chatgpt.com)
- [Proposal P0624R2: Default construction and assignment of captureless closures](https://wg21.link/P0624R2?utm_source=chatgpt.com)
- [Proposal P0315R4: Lambdas in unevaluated contexts](https://wg21.link/P0315R4?utm_source=chatgpt.com)
- [Proposal P0780R2: Pack expansion in init-capture](https://wg21.link/P0780R2?utm_source=chatgpt.com)
- [Proposal P0806R2: Deprecating implicit capture of ](https://wg21.link/P0806R2?utm_source=chatgpt.com)[`this`](https://wg21.link/P0806R2?utm_source=chatgpt.com)[ via ](https://wg21.link/P0806R2?utm_source=chatgpt.com)[`[=]`](https://wg21.link/P0806R2?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>