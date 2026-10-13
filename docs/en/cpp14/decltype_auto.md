<div align="center">

[🇺🇸 English](./decltype_auto.md) | [🇮🇷 فارسی](../../fa/cpp14/decltype_auto.md)

</div>

---

# A Comprehensive Guide to `decltype(auto)` in C++14

## Introduction and Design Goals

`decltype(auto)` is a C++14 language feature that provides more precise control over type deduction. It is particularly important when writing generic functions, wrapper functions, expression-dependent return types, and code that must preserve reference semantics.

In many situations, a programmer wants the type of a variable or function return value to be deduced automatically. However, ordinary `auto` deduction does not always preserve the exact type that is required. For example, an expression may have type `int&`, but using `auto` to deduce the type of a variable initialized from that expression produces `int`. When preserving the reference is necessary, `decltype(auto)` can provide the appropriate solution.

To use this feature correctly, it is essential to understand the differences between `auto`, `decltype`, and `decltype(auto)`, as well as the rules governing lvalues, rvalues, and references.

## The Core Concept and Why It Was Introduced

`decltype(auto)` was introduced in the C++14 standard. It combines two existing language mechanisms:

- `decltype`, which determines the type of a name or expression according to specific language rules.
- `auto`, which enables automatic type deduction.

The key difference is that `decltype(auto)` does not use the ordinary `auto` deduction rules to determine the type. Instead, it uses the rules of `decltype`.

### The Basic Difference Between `auto` and `decltype(auto)`

Consider the following code:

```
int value = 42;
int& reference = value;

auto first = reference;
decltype(auto) second = reference;
```

In this example:

- The type of `first` is `int`.
- The type of `second` is `int&`.

The difference occurs because `auto` removes the reference from the deduced type in this declaration, whereas `decltype(auto)` applies the rules of `decltype` to the initializer expression.

Consequently, `decltype(auto)` can preserve properties such as reference and `const` qualification when necessary.

### Why This Feature Matters

Suppose you write a generic function that returns the result of calling another function. If the original function returns a reference, you may want the wrapper to preserve that reference. Using `auto` for the return type can discard this property, whereas `decltype(auto)` makes it possible to preserve the return type more precisely.

However, preserving the exact type is not always preferable. Sometimes, removing the reference and returning an independent value is safer and more appropriate. The choice should depend on the function's contract and the lifetime of the referenced object.

## Understanding Type Deduction with `decltype`

To fully understand `decltype(auto)`, we first need to examine the rules of `decltype`.

### Type Deduction from a Variable Name

When the argument to `decltype` is the unparenthesized name of a variable, the result is the declared type of that variable.

```
int value = 10;
const int constantValue = 20;
int& reference = value;

decltype(value) a = 1;
decltype(constantValue) b = 2;
decltype(reference) c = value;
```

The resulting types are:

| Variable | Deduced type |
| -------- | ------------ |
| `a`      | `int`        |
| `b`      | `const int`  |
| `c`      | `int&`       |

Under this rule, `decltype(reference)` is `int&` because the declared type of `reference` is already `int&`.

### How Parentheses Affect `decltype`

One of the most important aspects of using `decltype(auto)` is that adding parentheses around an expression can change the deduced type.

For an expression that is not an unparenthesized variable name, the general rule depends on the expression's value category:

- If the expression is an lvalue, the result of `decltype` is `T&`.
- If the expression is an xvalue, the result is `T&&`.
- If the expression is a prvalue, the result is `T`.

Here, `T` represents the type of the expression or object, with the relevant type and qualifier rules applied.

Consider the following example:

```
int value = 42;

decltype(value) first = value;
decltype((value)) second = value;
```

The variables have different types:

- The type of `first` is `int`.
- The type of `second` is `int&`.

With `decltype(value)`, the declared type of the variable is used because `value` is an unparenthesized variable name. With `decltype((value))`, the expression is no longer an unparenthesized variable name. Since `(value)` is an lvalue, the result is `int&`.

This subtle distinction is one of the most common sources of mistakes in code that uses `decltype(auto)`.

### The Effect of `const` and References

The rules of `decltype` can preserve `const`, `volatile`, and reference qualifiers. For example:

```
const int value = 42;

decltype(value) first = 10;
decltype((value)) second = value;
```

In this example:

- The type of `first` is `const int`.
- The type of `second` is `const int&`.

Because `value` is an lvalue of type `const int`, the parenthesized expression `(value)` produces the type `const int&`.

This behavior is essential when writing generic code that must preserve the exact type and value category of an expression.

## Using `decltype(auto)` for Variable Declarations

In C++14, `decltype(auto)` can be used to deduce a variable's type from its initializer.

```
int value = 42;
const int constantValue = 100;
int& reference = value;

decltype(auto) a = value;
decltype(auto) b = (value);
decltype(auto) c = constantValue;
decltype(auto) d = reference;
```

The resulting types are:

| Variable | Initializer expression | Resulting type |
| -------- | ---------------------- | -------------- |
| `a`      | `value`                | `int`          |
| `b`      | `(value)`              | `int&`         |
| `c`      | `constantValue`        | `const int`    |
| `d`      | `reference`            | `int&`         |

In this example, parenthesizing `value` causes `b` to become a reference. By contrast, the unparenthesized name `reference` retains its declared type, `int&`.

### Syntactic Restrictions on Variable Declarations

When using `decltype(auto)`, remember that this feature is designed to deduce a type from a single initializer expression. Unlike some uses of `auto`, multiple independently deduced variables cannot be declared in a single declaration using `decltype(auto)`.

For example, the following declaration is invalid:

```
int x = 1;
int y = 2;

// Invalid declaration
// decltype(auto) a = x, b = y;
```

Each variable must be declared separately.

As with `auto`, the type must be deduced from an initializer, so an uninitialized declaration is not permitted:

```
// Invalid declaration
// decltype(auto) value;
```

### Differences Between Direct, Copy, and List Initialization

To better understand the behavior of this feature, consider the following example:

```
int value = 10;

decltype(auto) first = value;
decltype(auto) second(value);
decltype(auto) third{value};
```

All three declarations above deduce the type `int`, because each initializer leads to that type under the applicable `decltype` rules.

However, this result must not be generalized to declarations in which an expression is explicitly parenthesized. The following declaration produces a reference:

```
decltype(auto) reference = (value);
```

The type of `reference` is `int&`.

The important point is that `decltype(auto)` applies the `decltype` rules to the initializer expression. Consequently, the exact form of the expression is part of the program's semantics.

## Using `decltype(auto)` as a Function Return Type

One of the most important applications of this feature is return type deduction.

In C++14, functions can use `auto` to deduce their return types. However, ordinary `auto` return type deduction generally discards references, whereas `decltype(auto)` can preserve them.

### Comparing `auto` and `decltype(auto)` Return Types

```
int globalValue = 42;

auto getValue() {
    return globalValue;
}

decltype(auto) getReference() {
    return (globalValue);
}
```

In this example:

- `getValue` returns an `int`.
- `getReference` returns an `int&`.

In the first function, the return type is deduced using the rules of `auto`, so the expression's reference semantics are not preserved. In the second function, the expression `(globalValue)` is an lvalue under the `decltype` rules, so the return type is `int&`.

This distinction affects program behavior:

```
getValue() = 10;      // Invalid: the result is not a modifiable lvalue
getReference() = 10; // Valid: modifies globalValue
```

The first expression is invalid because `getValue()` produces a prvalue of type `int`, which cannot be used as a modifiable lvalue. The second expression is valid and modifies `globalValue`.

### The Importance of Parentheses in `return` Expressions

When a function uses `decltype(auto)` as its return type, parentheses around the expression in a `return` statement can make a fundamental difference.

```
int value = 42;

decltype(auto) first() {
    return value;
}

decltype(auto) second() {
    return (value);
}
```

In this example, the return type of `first` is `int`, while the return type of `second` is `int&`.

The reason is the special `decltype` rule for unparenthesized variable names. In `first`, the expression `return value;` yields the variable's declared type. In `second`, the expression `(value)` is an lvalue, so the return type becomes `int&`.

Therefore, adding or removing a pair of parentheses in such a function is not merely a stylistic change. It can change the function's contract and the behavior of its callers.

### Return Type Deduction Requirements

For a function whose return type is declared as `decltype(auto)`, the return type is determined from its `return` expressions using the rules of `decltype`. Consequently, if a function has multiple return paths, their deduced types must be compatible.

For example:

```
int value = 42;

decltype(auto) getValue(bool condition) {
    if (condition) {
        return value;
    }

    return 0;
}
```

This function is valid because both return paths produce the return type `int`.

However, the following example produces `int` on one path and `int&` on the other:

```
int value = 42;

decltype(auto) invalidFunction(bool condition) {
    if (condition) {
        return value;
    }

    return (value);
}
```

This function is invalid because the deduced return types do not match. When using `decltype(auto)`, every return path must be considered, not just the path normally taken at runtime.

## Applications in Generic Functions and Wrappers

One professional application of `decltype(auto)` is writing wrappers that preserve the behavior of the wrapped function as closely as possible.

Suppose you have a function whose result might be an independent value or a reference. A wrapper must decide whether to preserve the exact result type or always return an independent value.

### Preserving the Original Function's Return Type

The following example demonstrates a simple wrapper that preserves the return type:

```
#include <utility>

template <typename F, typename... Args>
decltype(auto) invokeWrapper(F&& function, Args&&... args) {
    return std::forward<F>(function)(
        std::forward<Args>(args)...
    );
}
```

This function uses `decltype(auto)` to deduce its return type and `std::forward` to preserve the value categories of its arguments.

In this example, if the original function returns an lvalue of type `T`, the wrapper can preserve `T&`. If the original function returns an independent value, the wrapper's return type is deduced from that expression accordingly.

However, this code is a simple generic wrapper and is not a complete replacement for `std::invoke` or `std::invoke_result` in every situation. In C++17, standard facilities such as `std::invoke` were introduced to provide a unified invocation mechanism for functions, member pointers, and callable objects.

### Preserving References When Accessing an Object's Members

Suppose you want to write a function that returns a member of an object while preserving the result's reference semantics:

```
struct Data {
    int value = 42;
};

Data object;

decltype(auto) getMember() {
    return (object.value);
}
```

The return type of `getMember` is `int&`. Consequently, the caller can modify the member directly:

```
getMember() = 100;
```

By contrast, omitting the parentheses produces a different result:

```
decltype(auto) getMemberCopy() {
    return object.value;
}
```

In this case, the return type is `int`, so modifying the function's result does not modify the original object's member.

### Applications in Library Wrappers

In generic libraries, a wrapper sometimes needs to forward the result of another function without introducing a new type-conversion policy. In such cases, `decltype(auto)` can prevent unnecessary copies and preserve references when appropriate.

However, this feature should not automatically be used in every wrapper. If the library's contract specifies that an independent value should be returned, `auto` may be a simpler and safer choice.

## Applications in Lambda Expressions

In C++14, lambda expressions can deduce their return types using `auto`. You can also use `decltype(auto)` in a trailing return type to preserve the exact result type.

### Return Type Deduction in Lambdas

The following code shows a lambda that returns a variable's value:

```
int value = 42;

auto getValue = [&]() {
    return value;
};

auto getReference = [&]() -> decltype(auto) {
    return (value);
};
```

In this example:

- The return type of `getValue` is `int`.
- The return type of `getReference` is `int&`.

Using `decltype(auto)` in the trailing return type causes the return type to be determined from the return expression according to the `decltype` rules.

### Choosing the Appropriate Return Type for a Lambda

When designing a lambda, determine whether its result should be an independent copy or a reference to the original data.

```
int values[] = {10, 20, 30};

auto getCopy = [&]() {
    return values[0];
};

auto getReference = [&]() -> decltype(auto) {
    return (values[0]);
};
```

In the first lambda, the result is an independent `int` value. In the second, the result is an `int&`, and modifying it changes the array element.

This distinction is especially important for lambdas used as callbacks, accessors, or parts of a public API.

## Comparing `decltype(auto)`, `auto`, and `decltype`

To choose the appropriate mechanism, it is important to understand the differences between these three related features.

| Feature                               | `auto`                                               | `decltype`                                         | `decltype(auto)`                      |
| ------------------------------------- | ---------------------------------------------------- | -------------------------------------------------- | ------------------------------------- |
| Primary purpose                       | Type deduction using `auto` rules                    | Determine a type using `decltype` rules            | Type deduction using `decltype` rules |
| Variable declarations                 | Yes                                                  | Yes                                                | Yes                                   |
| Function return types                 | Yes                                                  | Yes, with an explicit type or trailing return type | Yes                                   |
| Reference preservation                | Usually discarded unless `auto&` or `auto&&` is used | Follows the `decltype` rules                       | Follows the `decltype` rules          |
| Sensitivity to expression parentheses | No, for ordinary deduction                           | Yes                                                | Yes                                   |
| Introduced in                         | C++11                                                | C++11                                              | C++14                                 |

### Comparing `decltype(auto)` with `auto&` and `auto&&`

`auto&` is appropriate when you want to retain a reference to an lvalue. `auto&&` is also commonly used in generic programming to accept different value categories.

```
int value = 42;

auto copy = value;
auto& lvalueReference = value;
auto&& forwardingReference = value;
decltype(auto) exactType = (value);
```

The resulting types are:

- `copy` is `int`.
- `lvalueReference` is `int&`.
- `forwardingReference` is `int&` in this example.
- `exactType` is `int&`.

Here, `auto&&` becomes `int&` because the initializer is an lvalue. This behavior follows the rules of reference collapsing.

The key difference is that `auto&` and `auto&&` determine the reference type according to `auto` deduction and the initializer's type and value category. By contrast, `decltype(auto)` uses the rules of `decltype`.

### Comparing with `std::decay` and `std::remove_reference`

In generic programming, you sometimes want the opposite behavior: removing references or certain qualifiers.

For this purpose, type traits such as `std::remove_reference` and `std::decay` are useful. These are standard library facilities, whereas `decltype(auto)` is a language feature.

```
#include <type_traits>

int value = 42;
int& reference = value;

using ExactType = decltype(reference);
using ValueType = std::remove_reference_t<ExactType>;
using DecayedType = std::decay_t<decltype(reference)>;
```

In this example:

- `ExactType` is `int&`.
- `ValueType` is `int`.
- `DecayedType` is also `int`.

The aliases `std::remove_reference_t` and `std::decay_t` were introduced in C++14. They are appropriate when type transformation is part of the design, rather than when the goal is simply to preserve an expression's type.

## The Relationship with Value Categories and Reference Collapsing

Fully understanding `decltype(auto)` requires understanding value categories because the result of `decltype` for ordinary expressions depends on their value categories.

### Understanding Lvalues, Xvalues, and Prvalues

In C++, expressions are divided into value categories that matter when working with references and type deduction.

- Lvalue: An expression with an identifiable identity that can typically be used to bind an lvalue reference, such as a named variable.
- Prvalue: An expression generally used to compute a new value, such as an integer literal.
- Xvalue: An expression that refers to an identifiable object but indicates that its resources may be eligible for reuse, such as an expression produced by `std::move`.

In this classification, both lvalues and xvalues are glvalues, while both xvalues and prvalues are rvalues.

### How Value Categories Affect the Resulting Type

```
#include <utility>

int value = 42;

decltype(value) a = value;
decltype((value)) b = value;
decltype(std::move(value)) c = std::move(value);
```

The resulting types are:

- `a` is `int`.
- `b` is `int&`.
- `c` is `int&&`.

The reason is that `value`, without parentheses, follows the special variable-name rule; `(value)` is an lvalue; and `std::move(value)` produces an xvalue.

It is important to remember that `std::move` does not move an object by itself. Instead, it converts the expression into a form that allows move semantics to be used if an appropriate operation exists and is selected by the language rules.

### Understanding Reference Collapsing

In templates and generic declarations, combining references can produce types such as `T& &` or `T&& &`. The language rules collapse these combinations into valid reference types. This process is called reference collapsing.

The general rule is that if a combination contains at least one lvalue reference, the resulting type is an lvalue reference. Only the combination `T&& &&` produces `T&&`.

These rules are important for understanding `auto&&`, forwarding references, and generic functions. Although `decltype(auto)` does not directly use these rules as its initial type-deduction mechanism, reference types it produces can participate in reference collapsing in subsequent declarations.

## Practical Limitations and Risks

`decltype(auto)` provides precise control over types, but incorrect use can lead to compilation errors or object-lifetime problems.

### The Danger of Returning a Reference to a Local Variable

One of the most important mistakes is returning a reference to a local variable.

```
decltype(auto) dangerousFunction() {
    int localValue = 42;
    return (localValue);
}
```

The return type of this function is `int&`, but `localValue` ceases to exist when the function returns. Consequently, the returned reference dangles, and using it results in undefined behavior.

A solution is to return a value using `auto` or an explicit type when an independent value is required:

```
auto safeFunction() {
    int localValue = 42;
    return localValue;
}
```

In this example, the value is returned independently.

### The Danger of Returning a Reference to a Temporary

Sometimes a function returns a reference to data obtained from a temporary object. Consider the following example:

```
#include <string>

decltype(auto) dangerousString() {
    return std::string{"temporary"};
}
```

In this example, the return type is `std::string`, not a reference, because the return expression is a prvalue. Therefore, this function does not inherently have a dangling-reference problem.

However, the situation is different when a reference to a subobject of a temporary is returned:

```
#include <string>

decltype(auto) dangerousCharacter() {
    return (std::string{"temporary"}[0]);
}
```

In this example, the parenthesized expression is an lvalue of type `char`, so the return type is `char&`. When the function returns, the temporary string is destroyed, leaving the returned reference dangling.

This example demonstrates that examining the return type alone is insufficient. The lifetime of the object to which the reference refers must also be considered.

### Accidentally Changing a Function's Contract

Suppose a function previously returned an independent value, and its return type is changed to `decltype(auto)` merely to shorten the code. If the return expression is parenthesized, the return type may change from `T` to `T&`.

This change can have significant consequences:

- The caller may be able to modify an internal object.
- The lifetime of the referenced object becomes part of the function's contract.
- Using the function's result in other expressions may behave differently.
- Changing the return type of a public API may break compatibility with dependent code.

For this reason, `decltype(auto)` should be used with a clear goal of preserving the exact type, rather than as a universal replacement for `auto`.

## Advanced Details and Edge Cases

### Limitations When Deducing from an Initializer List

`decltype(auto)` does not deduce a type from a standalone braced initializer list in the same way that `auto` can. It cannot generally be used to deduce a `std::initializer_list` type directly from a braced initializer list.

For example:

```
// Invalid: decltype(auto) cannot deduce from a braced-init-list
// decltype(auto) values = {1, 2, 3};
```

In this case, the braced list does not have an independent expression type to which the `decltype` rules can be directly applied.

If the goal is to create a collection of integers, the type can be specified explicitly:

```
#include <initializer_list>

std::initializer_list<int> values = {1, 2, 3};
```

Alternatively, if the goal is to deduce a type from a single value, ordinary initialization can be used:

```
decltype(auto) value = 42;
```

### Limitations When Returning a Bit-field

In C++, bit-fields have special behavior. If an expression refers to a bit-field, it cannot be bound to a reference in the same way as an ordinary object.

For example:

```
struct Flags {
    unsigned int enabled : 1;
};

Flags flags{};

decltype(auto) getFlag() {
    return (flags.enabled);
}
```

This function is invalid because `(flags.enabled)` is an lvalue bit-field, and the `decltype` rules would deduce the return type as `unsigned int&`. However, references to bit-fields cannot be formed.

In such cases, if the goal is simply to read the member's value, returning an independent value is more appropriate:

```
unsigned int getFlagValue() {
    return flags.enabled;
}
```

This limitation demonstrates that even when a deduced type appears suitable, the language's restrictions on references still apply.

### Using `decltype(auto)` with `constexpr`

`decltype(auto)` can also be used with `constexpr` functions because return type deduction and constant evaluation are independent concepts.

```
constexpr int calculate() {
    return 42;
}

constexpr decltype(auto) getResult() {
    return calculate();
}
```

In this example, the return type of `getResult` is `int`, and the function can be used in contexts that require a constant expression.

However, declaring a function `constexpr` does not guarantee that every call will be evaluated at compile time. That depends on the calling context and the constant-expression rules.

### Using Templates and the Limitations of Error Detection

In templates, the type deduced by `decltype(auto)` may depend on template arguments, the result type of another function, or the value category of a dependent expression.

```
template <typename T>
decltype(auto) getValue(T& value) {
    return (value);
}
```

In this example, the return type is `int&` when `T = int` and `const int&` when `T = const int`.

This behavior allows the function to preserve the argument's reference semantics and `const` qualification.

However, if the return type depends on an invalid expression, template instantiation may fail with a compilation error. Return type deduction is not, by itself, a general mechanism for turning every type-related error into SFINAE, particularly when the function body must be instantiated to determine the return type.

When designing generic templates, it is better to define type requirements and valid-operation conditions explicitly. In C++20, facilities such as concepts and constraints can make these requirements easier to express.

## Choosing the Right Type in API Design

When choosing between `auto` and `decltype(auto)`, first determine the intended contract of the function or variable.

| Design requirement                               | Recommended choice                               |
| ------------------------------------------------ | ------------------------------------------------ |
| Produce an independent value                     | `auto` or an explicit type                       |
| Preserve the exact type of a return expression   | `decltype(auto)`                                 |
| Bind to an lvalue using a reference              | `auto&`                                          |
| Accept different value categories for forwarding | `auto&&` or an appropriate template parameter    |
| Preserve references in a generic wrapper         | Usually `decltype(auto)` with correct forwarding |
| Remove a reference from a type in a template     | `std::remove_reference_t`                        |
| Avoid exposing references to internal data       | Usually return an independent value              |

This table is a design guide rather than a set of absolute rules. For example, returning a reference to a container's element may be efficient and appropriate for an API, but only if the container's lifetime and the access contract are managed correctly.

### Performance Considerations

Using `decltype(auto)` does not necessarily improve performance. Its primary benefit is preserving the exact type and avoiding unintended changes in expression semantics.

If a function returns an independent value, `auto` usually provides the same intended semantics. In addition, in C++11 and later, move semantics and return value optimization can reduce the cost of returning values. Therefore, a reference should not be returned merely to avoid a hypothetical copy.

In professional design, ownership, object lifetimes, and API contracts should be established first. The decision to preserve a reference should follow from those requirements.

## Practical Examples

### Preserving a Reference in an Accessor

In this example, a generic function provides access to an object's member:

```
struct User {
    int age = 30;
};

template <typename T>
decltype(auto) getAge(T& user) {
    return (user.age);
}
```

For a modifiable object, the function returns a modifiable reference. For a `const` object, it returns a reference of type `const int&`.

```
User user;
getAge(user) = 40;

const User readOnlyUser{};
int age = getAge(readOnlyUser);
```

In this example, modifying the result of the first call changes the `age` member. The second call only reads the value.

The important design consideration is that this function is suitable only for types that have an accessible, valid `age` member. In a public generic library, the requirements on the argument type should also be clearly specified.

### Building a Wrapper That Preserves the Result's Value Category

The following example shows a wrapper that returns a function's result without imposing an additional type conversion:

```
#include <utility>

template <typename F, typename... Args>
decltype(auto) call(F&& function, Args&&... args) {
    return std::forward<F>(function)(
        std::forward<Args>(args)...
    );
}

int value = 42;

int& getReference() {
    return value;
}

int main() {
    call(getReference) = 100;
}
```

In this code, `call` returns an lvalue reference of type `int&`. Consequently, assignment to the result of the call is valid and changes `value`.

This example demonstrates an important use of `decltype(auto)` combined with perfect forwarding. However, a real wrapper intended for a public library must also consider member-function invocation, overloads, callability constraints, and compatibility with the rules implemented by `std::invoke`.

## Final Summary

`decltype(auto)`, introduced in C++14, is a precise type-deduction mechanism that follows the rules of `decltype`. It is particularly valuable when preserving references, qualifiers, and the exact type of a return expression matters.

The most important practical points are:

- `decltype(auto)` performs type deduction using the rules of `decltype`, not ordinary `auto` deduction.
- Parenthesizing an expression can change the return type from `T` to `T&`.
- This feature is useful in generic functions, wrappers, lambda expressions, and accessors.
- Incorrect use can produce dangling references or unintentionally change an API's contract.
- When returning an independent value, `auto` is often simpler and more appropriate.
- The correct choice depends on ownership, object lifetime, value category, and whether preserving a reference is genuinely necessary.

Ultimately, `decltype(auto)` is the best choice when preserving an expression's exact type is an intentional part of the design. When that behavior is unnecessary, a simpler return type often improves readability and maintainability.

---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>