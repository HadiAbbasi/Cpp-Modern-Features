<div align="center">

[🇺🇸 English](./variadic_templates.md) | [🇮🇷 فارسی](./../fa/cpp11/variadic_templates.md)

</div>

---

# Variadic Templates in C++11

Variadic Templates, introduced in C++11, make it possible to define function templates and class templates that accept a variable number of arguments or template parameters. This feature is one of the foundations of generic programming in modern C++ and plays an important role in implementing facilities such as `std::tuple`, `std::make_shared`, generic function wrappers, and many components of the standard library.

Before Variadic Templates, supporting different numbers of arguments often required defining multiple overloads or using approaches with less flexibility and weaker type safety. Variadic Templates eliminate this limitation by allowing a single generic pattern to handle a variable-length collection of parameters.

This document covers the underlying structure of Variadic Templates, implementation techniques, deduction rules, differences from C-style variadic functions, limitations, common mistakes, and their relationship to features introduced in later C++ standards.

## Understanding Variadic Templates

In C++11, a `template parameter pack` is a collection of a variable number of template parameters. It can contain type parameters, non-type template parameters, or template-template parameters. A `function parameter pack` is a collection of a variable number of function parameters.

The ellipsis (`...`) is used to declare and expand parameter packs. Its position determines which kind of pack is involved.

### Defining a Variadic Type Parameter Pack

In the following example, `Types` is a pack of type parameters:

```
template <typename... Types>
struct TypeList {
};
```

Here, `Types` is a `template parameter pack`. Depending on how `TypeList` is instantiated, the pack can be empty or contain one or more types.

```
TypeList<> empty;
TypeList<int> one;
TypeList<int, double, char> several;
```

All three declarations are valid because the definition does not require `Types` to contain any minimum number of elements.

### Defining a Variadic Function Parameter Pack

In the following function, `Args` is a `function parameter pack`:

```
template <typename... Args>
void printTypes(Args... args) {
}
```

This declaration contains two packs:

- `Args` is a collection of type parameters.
- `args` is a collection of function parameters, each of whose type corresponds to the respective type in `Args`.

For example, the following call passes three arguments of different types:

```
printTypes(10, 3.14, "hello");
```

The compiler deduces the type of each argument and generates the appropriate function-template specialization.

The important point is that the function above does not perform any operation on its arguments by itself. To process pack elements, we need mechanisms such as `pack expansion` or recursive function calls.

## Differences Between Template Parameter Packs and Function Parameter Packs

These two concepts are closely related, but they serve different purposes.

| Feature                     | `Template Parameter Pack`             | `Function Parameter Pack`                                     |
| --------------------------- | ------------------------------------- | ------------------------------------------------------------- |
| Purpose                     | Stores template parameters            | Stores function parameters                                    |
| Example                     | `typename... Types`                   | `Args... args`                                                |
| Contents                    | Types, values, or template parameters | Function parameters                                           |
| When the size is determined | During template specialization        | During function-template specialization and argument matching |

In the following example, a template parameter pack is used to determine the types of the function parameters:

```
template <typename... Types>
void process(Types... values) {
}
```

The `Types` and `values` packs have corresponding elements and matching lengths. If the function is called with three arguments, both packs contain three elements.

This relationship does not mean that every pack must be used as a function parameter. Packs can also appear in other template constructs.

## Understanding Pack Expansion

The most important part of working with Variadic Templates is `pack expansion`. This mechanism allows a specified pattern to be expanded for every element of a pack.

Consider the following declaration:

```
template <typename... Args>
void process(Args... values) {
}
```

In this function, `Args...` means that the parameter pattern is expanded for every element in `Args`. If `Args` contains `int`, `double`, and `char`, the conceptual form of the function parameters is:

```
void process(int, double, char);
```

This is only a conceptual representation. The compiler generates the function through template instantiation.

### Using an Expansion Pattern

In a pack expansion, the pattern preceding `...` specifies what is expanded. The compiler applies that pattern to the corresponding pack elements.

The following example prints all the argument values:

```
#include <iostream>

template <typename... Args>
void printAll(Args... args) {
    using expander = int[];
    (void)expander{0, ((std::cout << args << ' '), 0)...};
    std::cout << '\n';
}

int main() {
    printAll(10, 3.14, "hello");
}
```

Output:

```
10 3.14 hello
```

In this implementation, the expression `((std::cout << args << ' '), 0)...` generates an array element for each element of `args`. The comma operator ensures that printing occurs first and that zero is then produced as the corresponding array element.

This was a common pre-C++17 technique for applying an operation to every pack element. C++17 introduced `fold expression`, which makes this kind of code substantially simpler, as discussed later.

### Expanding Multiple Packs Simultaneously

Sometimes two packs must be used together. When multiple packs appear in the same expansion pattern, their corresponding elements are expanded simultaneously.

The following example processes a single pack:

```
#include <iostream>

template <typename... Types>
void printPairs(Types... values) {
    using expander = int[];
    (void)expander{0, ((std::cout << values << '\n'), 0)...};
}

int main() {
    printPairs(10, 20, 30);
}
```

The example above contains only one pack, so each value is processed independently. To understand simultaneous expansion of two packs, consider the following example:

```
#include <iostream>

template <typename... Types>
void printPairs(Types... left, Types... right) {
    using expander = int[];
    (void)expander{
        0,
        ((std::cout << left << " : " << right << '\n'), 0)...
    };
}

int main() {
    printPairs<int>(1, 2, 10, 20);
}
```

This example is not suitable for general use because of template deduction rules and the difficulty of determining the boundaries of trailing parameter packs. As written, it does not reliably implement the intended calling pattern.

For two collections of equal length, it is better to use two independent packs in an appropriate structure, such as tuples.

A valid implementation can use `std::index_sequence` in later C++ standards. For understanding the fundamentals of C++11, however, the key point is that all packs appearing in a single expansion must satisfy the compatibility rules for that expansion.

### Important Pack Expansion Rules

- The ellipsis (`...`) is not a complete operation on its own; it must be associated with an expandable pattern.
- Pack expansion can occur in function argument lists, template parameter lists, initializer lists, and several other contexts.
- Expanding an empty pack can produce no elements in contexts where an empty expansion is permitted.
- When multiple packs appear in the same expansion, their sizes must satisfy the rules for that expansion. In the usual case, packs with different numbers of elements cause a compilation error.

## Implementing Recursive Functions with Variadic Templates

In C++11, one of the most common ways to process pack elements is through a `recursive variadic function`. In this approach, the function processes one argument at each step and then calls itself with the remaining arguments.

### Defining a Base Function

To terminate recursion, we typically define an overload that takes no arguments or a special overload for the final argument.

```
#include <iostream>

void printAll() {
    std::cout << '\n';
}

template <typename T, typename... Rest>
void printAll(const T& first, const Rest&... rest) {
    std::cout << first << ' ';
    printAll(rest...);
}

int main() {
    printAll(10, 3.14, "hello");
}
```

Output:

```
10 3.14 hello
```

In this design:

- `first` receives the current argument.
- `rest` stores the remaining arguments in a pack.
- The function prints one value at each step and recursively calls itself with a smaller pack.
- When no arguments remain, the zero-argument overload is invoked.

### Using an Overload for the Final Argument

Some algorithms do not require a zero-argument overload. Instead, we can define one overload for a single argument and another for multiple arguments.

```
#include <iostream>

template <typename T>
void printAll(const T& value) {
    std::cout << value << '\n';
}

template <typename T, typename... Rest>
void printAll(const T& first, const Rest&... rest) {
    std::cout << first << ' ';
    printAll(rest...);
}

int main() {
    printAll(1, 2, 3, 4);
}
```

In this version, the single-argument overload processes the final argument. If the function is called with just one argument, the more general overload does not need to recurse.

### Advantages and Limitations of Recursion

Recursive techniques are available in C++11 and work well with many common algorithms. However, each step typically requires another specialization to be instantiated, and deep recursion can run into compiler limits.

The actual runtime cost must also be considered. The compiler may optimize away many layers of function calls, but this does not guarantee that every cost will disappear in every situation.

When the only requirement is to apply an operation to every pack element, expansion-based techniques are often simpler.

## Using Variadic Templates for Generic Functions

One of the primary applications of Variadic Templates is building generic functions whose argument count and types are not known in advance.

### Creating a Function That Forwards Arguments

Suppose we want to create a function that constructs an object using an arbitrary set of arguments. Defining separate overloads for each possible argument count is not a good solution.

In C++11, we can use `std::forward` and forwarding references:

```
#include <utility>

template <typename T, typename... Args>
T createObject(Args&&... args) {
    return T(std::forward<Args>(args)...);
}
```

This function forwards its arguments to the constructor of type `T`.

Two important capabilities are combined here:

- Variadic Templates allow the function to accept a variable number of arguments.
- Perfect Forwarding preserves the arguments' value categories, including whether they are lvalues or rvalues, when the relevant rules are followed.

The type `Args&&` is a forwarding reference in this context because `Args` is deduced. If `Args` were already fixed to a specific type, the interpretation of `T&&` could be different.

The function above is an educational example. For general-purpose code, the standard facilities `std::make_unique`, introduced in C++14, or `std::make_shared`, available in C++11, are usually better choices depending on the required ownership model.

### Forwarding Arguments to Another Function

Another common pattern is a generic wrapper that forwards arguments to another function:

```
#include <utility>

template <typename Callable, typename... Args>
auto invokeWrapper(Callable&& callable, Args&&... args)
    -> decltype(
        std::forward<Callable>(callable)(
            std::forward<Args>(args)...
    )
) {
    return std::forward<Callable>(callable)(
        std::forward<Args>(args)...
    );
}
```

This C++11 implementation uses a trailing return type to determine the return type from the invocation expression.

The important limitation is that this implementation supports only direct invocation of a callable. For example, if general support for pointer-to-member operations or other special invocation forms is required, a different approach is necessary.

In C++17, `std::invoke` provides more general support for this kind of invocation. In C++11, a custom abstraction or a compatible library may be needed for such requirements.

### Using Variadic Templates to Build Wrappers

In library design, wrappers are often used to add behavior around a function call, such as measuring execution time, recording logs, or handling errors.

```
#include <iostream>
#include <utility>

template <typename Callable, typename... Args>
auto logAndCall(Callable&& callable, Args&&... args)
    -> decltype(
        std::forward<Callable>(callable)(
            std::forward<Args>(args)...
    )
) {
    std::cout << "Calling function\n";

    return std::forward<Callable>(callable)(
        std::forward<Args>(args)...
    );
}
```

This wrapper forwards arguments without requiring multiple overloads. However, fully supporting callables that return `void` or invocation forms that require `std::invoke` calls for a more careful design.

## Managing Argument Types and Value Categories

One of the most important concerns when designing variadic functions is how argument types are deduced and how arguments are passed onward. Choosing the wrong parameter type can cause unnecessary copies, loss of move behavior, or restrictions on the types that can be used.

### Comparing Value Parameters and Reference Parameters

In the following example, parameters are passed by value:

```
template <typename... Args>
void processByValue(Args... args) {
}
```

This approach can cause arguments to be copied or moved and may be expensive for large objects.

By contrast, the following version uses references to `const`:

```
template <typename... Args>
void processByReference(const Args&... args) {
}
```

This avoids copying arguments merely to pass them into the function. However, it does not allow the objects to be modified and is not always the best choice, particularly when transferring ownership is intended.

Finally, the following version is more suitable for generic argument forwarding:

```
#include <utility>

template <typename... Args>
void forwardArguments(Args&&... args) {
    anotherFunction(std::forward<Args>(args)...);
}
```

This example assumes that `anotherFunction` is defined in an appropriate scope.

### Understanding Perfect Forwarding

The goal of Perfect Forwarding is to preserve the original properties of an argument as it passes through a wrapper. If an argument is an lvalue, it should remain an lvalue in the forwarded expression. If it is an rvalue, the forwarding mechanism should allow it to be treated as an rvalue.

`std::forward` is essential to this process. Replacing it with `std::move` in a generic forwarding function is usually a mistake because `std::move` casts every argument to an rvalue, potentially changing the expected behavior for lvalues.

Another important point is that `std::forward` does not move an object by itself. It performs a cast; whether the destination actually moves or copies the object depends on the overload that is selected.

### Lifetime Considerations

A function using forwarding references must not store references to received arguments for later use without carefully considering their lifetimes.

For example, returning a reference to a temporary argument can produce a dangling reference. Variadic Templates provide no automatic protection against such errors.

In public APIs, ownership and lifetime contracts should be clearly specified.

## Counting Pack Elements with sizeof...

In C++11, the `sizeof...` operator provides the number of elements in a parameter pack. It is evaluated at compile time, and its result is an integer constant of type `std::size_t`.

```
#include <iostream>

template <typename... Args>
void printCount(Args... args) {
    std::cout << sizeof...(Args) << '\n';
    std::cout << sizeof...(args) << '\n';
}

int main() {
    printCount(10, 3.14, 'A');
    printCount();
}
```

Output:

```
3
3
0
0
```

In this example, both `sizeof...(Args)` and `sizeof...(args)` calculate the number of elements in their respective packs: one is a type pack, and the other is a function parameter pack.

### Practical Uses of Pack Size

Pack size is useful when implementing structures such as tuples, validating argument counts, and creating compile-time conditions.

```
#include <type_traits>

template <typename... Args>
struct ArgumentInfo {
    static constexpr std::size_t count = sizeof...(Args);
};

static_assert(ArgumentInfo<int, double, char>::count == 3, "");
static_assert(ArgumentInfo<>::count == 0, "");
```

In this example, the number of arguments is available as a compile-time constant.

## Using Variadic Templates in Classes

Variadic Templates are not limited to functions; they are also widely used in class templates.

### Creating a Class That Represents a Collection of Types

The following simple example defines a list of types:

```
template <typename... Types>
struct TypeList {
    static constexpr std::size_t size = sizeof...(Types);
};
```

This structure can be used in metaprogramming, type registration, and generic abstractions.

```
#include <type_traits>

using MyTypes = TypeList<int, double, char>;

static_assert(MyTypes::size == 3, "");
```

This class stores types only as template parameters. It does not, by itself, store runtime objects or values.

### Understanding std::tuple

The standard class template `std::tuple` is one of the most important examples of Variadic Templates. It can store heterogeneous values of different types in a single object.

```
#include <iostream>
#include <string>
#include <tuple>

int main() {
    std::tuple<int, double, std::string> data{
        42, 3.14, "hello"
    };

    std::cout << std::get<0>(data) << '\n';
    std::cout << std::get<1>(data) << '\n';
    std::cout << std::get<2>(data) << '\n';
}
```

In this example, each tuple element has a specific type and can be accessed using `std::get` with a compile-time index.

When designing structures similar to tuples, Variadic Templates make it possible to specify the number and type of elements at compile time without defining a separate class for every possible element count.

### Defining a Class with a Variable Number of Base Classes

In C++11, packs can also be used in inheritance declarations. For example, a class can inherit from multiple base classes:

```
template <typename... Bases>
struct Combined : Bases... {
};
```

In this design, each type in `Bases` becomes a base class.

All supplied types must satisfy the language's inheritance rules. Additionally, if the base classes contain members with the same names, ambiguity may arise when accessing those members.

This pattern is useful for designing mixins and composing independent capabilities.

## Understanding Fold Expressions and Their Difference from C++11

One important limitation of Variadic Templates in C++11 is that applying an operation to every pack element often requires recursion or more complicated techniques.

C++17 introduced `fold expression`, which simplifies this task. Fold Expressions are therefore not part of C++11; they were introduced in a later standard.

### The Traditional C++11 Approach

To calculate the sum of arguments, we can use recursion:

```
int sum() {
    return 0;
}

template <typename T, typename... Rest>
auto sum(T first, Rest... rest)
    -> decltype(first + sum(rest...)) {
    return first + sum(rest...);
}
```

In this implementation, the base function returns zero for an empty argument list. The function template adds the first value to the result of processing the remaining arguments.

This approach works for types that support the addition operator, although the resulting return type and behavior depend on overload resolution and the argument types.

### A Simpler Approach in C++17

With Fold Expressions, the same algorithm can be expressed as follows:

```
template <typename... Args>
auto sum(Args... args) {
    return (args + ... + 0);
}
```

This example uses a right fold. However, adding `0` as an initial value may be inappropriate for certain types that are not compatible with integers.

For example, the following version is suitable for non-empty collections of types that support compatible addition operations in C++17:

```
template <typename First, typename... Rest>
auto sum(First first, Rest... rest) {
    return (first + ... + rest);
}
```

This version requires at least one argument, and the final type depends on the results of the addition operations.

Fold Expressions do not replace recursion in every situation. When processing each element depends on the result of the previous step, or when complex conditional logic is required, recursion may still be the more appropriate choice.

## Differences Between Variadic Templates and C-Style Variadic Functions

C++ supports both Variadic Templates and the older variadic function mechanism, which uses `<cstdarg>` and facilities such as `va_list`.

Although these approaches address similar needs, they differ significantly in type safety, type deduction, and argument access.

### Examining a Traditional Variadic Function

The following example defines a function that accepts a variable number of arguments:

```
#include <cstdarg>
#include <iostream>

double sumValues(int count, ...) {
    va_list args;
    va_start(args, count);

    double total = 0.0;

    for (int i = 0; i < count; ++i) {
        total += va_arg(args, double);
    }

    va_end(args);
    return total;
}

int main() {
    std::cout << sumValues(3, 1.0, 2.0, 3.0) << '\n';
}
```

In this example, the function receives the number of arguments and extracts each argument as a `double`.

With this mechanism, the compiler does not guarantee that the actual type of every argument matches the type expected by `va_arg`. Incorrect use can result in undefined behavior.

### Comparing the Two Approaches

| Feature                        | Variadic Templates                   | C-Style Variadic Functions                  |
| ------------------------------ | ------------------------------------ | ------------------------------------------- |
| Compile-time type checking     | Supported                            | Limited                                     |
| Support for different types    | Through template deduction           | Subject to restrictions and promotion rules |
| Access to each argument's type | Through template parameters          | Usually requires a separate contract        |
| Perfect Forwarding             | Supported                            | Not supported                               |
| Manual argument counting       | Usually unnecessary                  | Often required                              |
| Typical applications           | Generic APIs and generic programming | Legacy APIs and C-compatible interfaces     |

For new C++ APIs, Variadic Templates are generally the better choice. However, the traditional mechanism may still be necessary for C-compatible interfaces or APIs that follow an existing variadic argument contract.

## Type Deduction Rules in Variadic Templates

Understanding `template argument deduction` is essential when working with Variadic Templates. These rules determine how the compiler matches argument types to template parameters.

### Deduction in Function Parameters

In the following example, the compiler deduces the types of the arguments:

```
template <typename... Args>
void process(Args... args) {
}

int main() {
    process(10, 3.14, 'A');
}
```

In this call, the type pack contains `int`, `double`, and `char`.

When the parameters instead use `const Args&...`, reference deduction rules apply, allowing the arguments to be received without copying them into function parameters.

### Limitations of Deduction with Trailing Parameter Packs

In the following function, the pack appears at the end of the parameter list:

```
template <typename T, typename... Args>
void process(T first, Args... rest) {
}
```

This structure is suitable for many variadic APIs because the first argument is assigned to `T`, while the remaining arguments are captured by `Args`.

By contrast, placing a separate parameter after a pack can make deduction difficult or impossible, especially when the compiler cannot determine the boundary between the pack elements and the remaining arguments.

For this reason, the pattern of one explicitly positioned parameter followed by a trailing pack is generally easier to use and reason about.

## Empty Packs and Edge Cases

Variadic Templates can represent an empty collection of parameters. This feature provides flexibility but also requires careful handling of edge cases.

### Accepting Zero Arguments

The following function can be called without any arguments:

```
template <typename... Args>
void process(Args... args) {
}

int main() {
    process();
}
```

In this case, both `Args` and `args` are empty.

However, an empty pack does not mean that every operation involving it is valid. For example, if an algorithm needs to access the first pack element, the empty case must be handled separately.

### Empty Collections Versus Initial Values

When designing algorithms such as summation, the behavior for an empty collection should be explicitly defined.

For example, using zero as an initial value is reasonable for numeric addition, but choosing an appropriate initial value for operations such as finding a maximum depends on the data type.

Therefore, the contract for an empty pack should be explicit rather than relying on an arbitrary initial value merely to make the code compile.

## Common Mistakes and Limitations

Despite their power, Variadic Templates can lead to difficult compilation errors or poorly designed APIs when used incorrectly.

### Misusing the Ellipsis

The ellipsis (`...`) appears in several contexts, including pack declarations and pack expansions. These uses are syntactically and semantically different.

For example, `typename... Args` declares a type parameter pack, whereas `std::forward<Args>(args)...` performs a pack expansion.

Understanding this distinction is essential for reading template code.

### Ignoring the Difference Between Lvalues and Rvalues

When a function is intended to forward arguments generically, passing parameters by value or through references to `const` may change the intended behavior.

In such cases, correctly using forwarding references and `std::forward` helps preserve the arguments' value categories.

### Creating Ambiguity in Overload Resolution

Variadic functions can interact with other overloads. For example, a generic overload that accepts any type and any number of arguments may compete with a more specialized overload for certain calls.

In these situations, the rules of overload resolution and template partial ordering must be considered. Restricting the accepted types or providing more specific overloads can sometimes produce a more predictable design.

### Increasing Compilation Complexity

Extensive use of templates can increase compilation time, produce lengthy error messages, and generate many specializations.

This becomes particularly important when multiple layers of recursive templates or generic abstractions are composed.

Variadic Templates should therefore be used where their flexibility justifies the additional complexity.

## Guidelines for Designing APIs with Variadic Templates

When designing generic APIs, the appropriate structure depends on the function's purpose, the data types involved, and the intended usage contract.

### Choosing Between Recursion and Pack Expansion

If the goal is to apply the same operation to every pack element, pack expansion is usually a direct solution. If each step depends on the result of the previous step, recursion may be easier to understand.

In C++17, Fold Expressions are also a suitable option for repetitive operations over pack elements.

### Using Perfect Forwarding When Needed

For wrappers that forward arguments to another function, forwarding references and `std::forward` are often appropriate choices.

However, for a function that only reads its arguments, using `const T&` or passing by value may be simpler, depending on the data types and design goals.

### Documenting Type Requirements

Variadic Templates do not guarantee that every combination of types is valid. If an algorithm requires a particular operator, constructor, or function, that prerequisite should be made clear.

In C++11, compile-time checks can be implemented using `static_assert`, `std::enable_if`, and type traits. Later standards provide tools such as Concepts in C++20, which make it possible to express type requirements more directly.

## Summary

Variadic Templates, introduced in C++11, are a fundamental tool for building generic APIs and implementing generic programming. They make it possible to handle variable numbers of template parameters and function arguments without defining multiple overloads.

The most important concepts related to this feature include:

- `template parameter pack` and `function parameter pack` for representing variable numbers of parameters.
- `pack expansion` for expanding a pattern over pack elements.
- Recursive techniques for processing arguments in C++11.
- The `sizeof...` operator for counting pack elements at compile time.
- The use of Variadic Templates in structures such as `std::tuple` and in generic wrappers.
- The combination of Variadic Templates with `std::forward` to implement Perfect Forwarding.
- The differences between Variadic Templates and traditional variadic functions based on `<cstdarg>`.
- The relationship between this feature and newer facilities such as Fold Expressions in C++17 and Concepts in C++20.

Ultimately, Variadic Templates are most valuable when an API genuinely needs to support a variable number of arguments or types. Choosing appropriately among pack expansion, recursion, Perfect Forwarding, and standard-library abstractions leads to code that is safer, clearer, and easier to maintain.

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>
