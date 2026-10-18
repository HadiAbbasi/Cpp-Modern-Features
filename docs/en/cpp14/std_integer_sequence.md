<div align="center">

[🇺🇸 English](./std_integer_sequence.md) | [🇮🇷 فارسی](../../fa/cpp14/std_integer_sequence.md)

</div>

---

# Comprehensive Guide to `std::integer_sequence` in C++14

## Introduction

`std::integer_sequence` is an important Standard Library feature introduced in C++14 for representing a sequence of integer values as a compile-time constant sequence. It allows programmers to represent numeric values as a template parameter pack and, when needed, expand them into function arguments or use them in other compile-time operations.

This feature becomes particularly useful when working with a variable number of values at the template level. For example, it can be used to extract the elements of a `std::tuple` individually, convert a tuple into function arguments, or expand an operation over the elements of a data structure at compile time.

Before exploring this utility, it is important to understand one key distinction: `std::integer_sequence` is not a container, does not store elements in memory, and does not provide an algorithm for iterating over values at runtime. Instead, it is a type-level construct that encodes integer values as non-type template parameters in its type.

This distinction is fundamental to understanding its applications correctly.

## Understanding Compile-Time Integer Sequences

### Introducing `std::integer_sequence`

The `std::integer_sequence` type is defined in the standard `<utility>` header and has the following general form:

```
template<class T, T... Vals>
struct integer_sequence;
```

In this definition, the parameter `T` specifies the type of every value in the sequence, while `Vals...` is a parameter pack containing compile-time integer values.

For example, the following definition creates a type representing a sequence of five integers:

```
#include <utility>

using Sequence = std::integer_sequence<int, 10, 20, 30, 40, 50>;
```

In this example, the `Sequence` type encodes the following information:

- The value type is `int`.
- The number of values is five.
- The sequence contains `10`, `20`, `30`, `40`, and `50`, in that order.

These numbers appear as template arguments in the type definition. Therefore, there is no need to create five independent runtime variables to represent them.

### Examining a Sequence with Different Values

A sequence does not have to start at zero or contain consecutive numbers.

```
#include <utility>

using A = std::integer_sequence<int, 3, 7, 11>;
using B = std::integer_sequence<int, -4, 0, 100>;
using C = std::integer_sequence<unsigned, 2, 4, 8, 16>;
```

Each of these examples represents a distinct sequence type with its own values and integer type.

The important point is that `std::integer_sequence` describes a type, not a variable or an array. Consequently, two sequences with different values represent different types, even if they have the same number of elements and the same integer type.

## Understanding the Standard Members

### Examining `value_type`

This nested type specifies the type of the values in the sequence.

```
#include <type_traits>
#include <utility>

using Sequence = std::integer_sequence<long, 1, 2, 3>;

static_assert(
    std::is_same<Sequence::value_type, long>::value,
    "Unexpected value type"
);
```

In this example, `value_type` is `long` because that is the type specified when the sequence was defined.

### Examining `size()`

The `size()` member function returns the number of values in the sequence. Its return type is `std::size_t`.

```
#include <cassert>
#include <utility>

using Sequence = std::integer_sequence<int, 10, 20, 30>;

static_assert(Sequence::size() == 3, "Unexpected sequence size");

int main() {
    assert(Sequence::size() == 3);
}
```

In this example, `static_assert` performs the check at compile time. Calling `size()` inside `main` is also valid, although no runtime computation is necessary in this case.

### Examining an Empty Sequence

A sequence can contain no values:

```
#include <utility>

using EmptySequence = std::integer_sequence<int>;

static_assert(
    EmptySequence::size() == 0,
    "Sequence must be empty"
);
```

Empty sequences are important when implementing generic templates because a parameter pack may contain zero elements. Correctly designed code must handle this case as well.

## Comparing `std::integer_sequence` and `std::index_sequence`

### Introducing `std::index_sequence`

`std::index_sequence` is a standard alias template designed to represent a sequence of `std::size_t` indices.

Its conceptual definition is:

```
template<std::size_t... Indices>
using index_sequence =
    std::integer_sequence<std::size_t, Indices...>;
```

Therefore, the following sequence:

```
using Indices = std::index_sequence<0, 1, 2, 3>;
```

is equivalent in type to:

```
using Indices = std::integer_sequence<std::size_t, 0, 1, 2, 3>;
```

The advantage of `std::index_sequence` is that it communicates the intended purpose of the sequence more clearly: its values are indices, rather than arbitrary numeric data.

### Comparing Their Use Cases

For representing arbitrary numeric values, `std::integer_sequence` is the natural choice:

```
using Values = std::integer_sequence<int, 10, 20, 30>;
```

For accessing elements of a tuple, however, `std::index_sequence` is more expressive:

```
using Indices = std::index_sequence<0, 1, 2>;
```

Both types use the same underlying mechanism. Their differences primarily concern the value type, intended use, and readability of the code.

## Expanding Parameters Using a Sequence

### Understanding Parameter Pack Expansion

To understand the primary use of `std::integer_sequence`, we first need to understand parameter pack expansion.

C++ allows a collection of template parameters to be expanded using `...`. This makes it possible to perform similar operations on every element of a parameter pack.

For example:

```
#include <iostream>

template<int... Values>
void printValues() {
    ((std::cout << Values << ' '), ...);
}

int main() {
    printValues<10, 20, 30>();
}
```

This example uses a fold expression, which is available starting in C++17. Therefore, this implementation is not compatible with C++14, and another technique is needed for code that must support that standard.

The key point is that `std::integer_sequence` does not perform the expansion itself. Instead, it provides a parameter pack of values to another template, which can then expand that pack to perform the desired operation.

### Implementing Parameter Pack Expansion in C++14

C++14 does not yet have fold expressions. One common way to perform an operation on every element of a parameter pack is to use parameter pack expansion inside an initializer list.

```
#include <iostream>

template<int... Values>
void printValues() {
    using Expand = int[];

    (void)Expand{0, (std::cout << Values << ' ', 0)...};
}

int main() {
    printValues<10, 20, 30>();
}
```

In this technique, the expression inside the initializer list is expanded for every value in `Values...`.

The initial zero ensures that the initializer list remains valid even when the parameter pack is empty. In this example, the output is:

```
10 20 30
```

For generic code, however, it is important to consider the element types, the side effects of the operations, and the required evaluation order. In this pattern, initializer-list elements are evaluated in order.

## Practical Application: Accessing `std::tuple` Elements

### The Problem of Accessing Tuple Elements

The `std::tuple` type can store a collection of values with different types. Individual elements are typically accessed using `std::get`.

```
#include <iostream>
#include <tuple>

int main() {
    auto values = std::make_tuple(10, 3.14, 'A');

    std::cout << std::get<0>(values) << '\n';
    std::cout << std::get<1>(values) << '\n';
    std::cout << std::get<2>(values) << '\n';
}
```

In this example, the index of each element must be known at compile time. Consequently, we cannot simply use a regular loop with a runtime index to call `std::get<i>`, because the template argument to this function must be a compile-time constant.

This is where `std::index_sequence` becomes useful.

### Implementing a Function to Print All Tuple Elements

We can generate a sequence of indices and pass it to a helper function.

```
#include <iostream>
#include <tuple>
#include <utility>

template<class Tuple, std::size_t... Indices>
void printTupleImpl(const Tuple& tuple,
                    std::index_sequence<Indices...>) {
    using Expand = int[];

    (void)Expand{
        0,
        (static_cast<void>(
            std::cout << std::get<Indices>(tuple) << ' '
        ), 0)...
    };
}

template<class... Types>
void printTuple(const std::tuple<Types...>& tuple) {
    printTupleImpl(
        tuple,
        std::index_sequence_for<Types...>{}
    );
}

int main() {
    auto values = std::make_tuple(10, 3.14, 'A');

    printTuple(values);
}
```

The output is:

```
10 3.14 A
```

This implementation combines several important concepts:

- The `printTuple` function deduces the tuple element types through template argument deduction.
- `std::index_sequence_for` generates a sequence matching the number of tuple elements.
- The `std::index_sequence<Indices...>` parameter passes the index values to the helper function.
- The expression `std::get<Indices>(tuple)...` expands for every index.

As a result, instead of writing a separate call for every element, we can use a single generic template to process all tuple elements.

### Handling an Empty Tuple

A generic implementation must also handle the case of an empty tuple.

```
#include <iostream>
#include <tuple>
#include <utility>

template<class Tuple, std::size_t... Indices>
void printTupleImpl(const Tuple& tuple,
                    std::index_sequence<Indices...>) {
    using Expand = int[];

    (void)Expand{
        0,
        (static_cast<void>(
            std::cout << std::get<Indices>(tuple) << ' '
        ), 0)...
    };
}

template<class... Types>
void printTuple(const std::tuple<Types...>& tuple) {
    printTupleImpl(
        tuple,
        std::index_sequence_for<Types...>{}
    );
}

int main() {
    std::tuple<> empty;

    printTuple(empty);
}
```

In this example, `Indices...` is empty, so no calls to `std::get` are generated. The initial zero also ensures that the initializer list remains valid.

This example demonstrates how index sequences can simplify generic template design compared with writing separate code for different cases.

## Practical Application: Converting a Tuple into Function Arguments

### Understanding Tuple Unpacking

An important application of `std::index_sequence` is converting the elements of a tuple into independent function arguments. This technique is commonly known as tuple unpacking or tuple expansion.

For example, suppose we have a function that accepts three arguments:

```
int add(int a, int b, int c) {
    return a + b + c;
}
```

We now want to pass the values stored in a tuple directly to this function.

### Implementing a Helper Function

In C++14, this functionality can be implemented by combining `std::index_sequence`, `std::get`, and parameter pack expansion.

```
#include <iostream>
#include <tuple>
#include <utility>

template<class Function, class Tuple, std::size_t... Indices>
auto applyImpl(Function&& function,
               Tuple&& tuple,
               std::index_sequence<Indices...>)
    -> decltype(
        std::forward<Function>(function)(
            std::get<Indices>(
                std::forward<Tuple>(tuple)
            )...
        )
    ) {
    return std::forward<Function>(function)(
        std::get<Indices>(
            std::forward<Tuple>(tuple)
        )...
    );
}

template<class Function, class Tuple>
auto apply(Function&& function, Tuple&& tuple)
    -> decltype(
        applyImpl(
            std::forward<Function>(function),
            std::forward<Tuple>(tuple),
            std::make_index_sequence<
                std::tuple_size<
                    typename std::decay<Tuple>::type
                >::value
            >{}
        )
    ) {
    return applyImpl(
        std::forward<Function>(function),
        std::forward<Tuple>(tuple),
        std::make_index_sequence<
            std::tuple_size<
                typename std::decay<Tuple>::type
            >::value
        >{}
    );
}

int add(int a, int b, int c) {
    return a + b + c;
}

int main() {
    auto values = std::make_tuple(10, 20, 30);

    std::cout << apply(add, values) << '\n';
}
```

The output is:

```
60
```

In this example, the number of indices is obtained from `std::tuple_size`, after which `std::make_index_sequence` generates the required sequence.

The helper function `applyImpl` extracts the tuple elements using `std::get` and forwards them as separate arguments to the target function.

An important point is that this implementation is intended to demonstrate the underlying mechanism. Starting in C++17, the standard library provides `std::apply` for the same general purpose, and it should usually be preferred.

### Comparing with `std::apply`

The `std::apply` function has been available in the `<tuple>` header since C++17.

```
#include <iostream>
#include <tuple>

int add(int a, int b, int c) {
    return a + b + c;
}

int main() {
    auto values = std::make_tuple(10, 20, 30);

    std::cout << std::apply(add, values) << '\n';
}
```

If a project is restricted to C++14, a custom implementation can be used. In C++17 and later, however, the standard utility is generally more readable and less error-prone.

## Implementing Generic Algorithms with Integer Sequences

### Creating an Array from Sequence Values

Sometimes, we need to convert the values of a compile-time sequence into a runtime data structure. For example, we can initialize a `std::array` using the values in a sequence.

```
#include <array>
#include <iostream>
#include <utility>

template<class T, T... Values>
constexpr auto toArray(std::integer_sequence<T, Values...>) {
    return std::array<T, sizeof...(Values)>{{Values...}};
}

int main() {
    constexpr auto values =
        toArray(std::integer_sequence<int, 10, 20, 30>{});

    static_assert(values.size() == 3, "Unexpected size");

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

The output is:

```
10 20 30
```

In this example, the template values are used directly to initialize the array. The important distinction is that, unlike `std::integer_sequence` itself, a `std::array` actually stores its elements as data.

### Computing the Sum of a Sequence at Compile Time

Another useful example is calculating the sum of a sequence's values.

```
#include <utility>

template<class T, T... Values>
constexpr T sumSequence(std::integer_sequence<T, Values...>) {
    return (T{} + ... + Values);
}

int main() {
    constexpr auto result =
        sumSequence(std::integer_sequence<int, 1, 2, 3, 4>{});

    static_assert(result == 10, "Unexpected sum");
}
```

This example uses a fold expression and requires C++17. Therefore, it must not be considered a C++14-compatible implementation.

In C++14, the same operation can be implemented using template-level recursion or an alternative pattern based on parameter pack expansion.

More importantly, `std::integer_sequence` does not necessarily imply that a computation is performed at compile time. This type only encodes values at the type level. Whether an operation on those values is actually evaluated at compile time depends on the `constexpr` rules, how the result is used, and the requirements for constant evaluation.

## Comparing the Sequence Generation Utilities

### Differences Between the Main Utilities

| Utility                      | Primary purpose                                         | Result type                                            |
| ---------------------------- | ------------------------------------------------------- | ------------------------------------------------------ |
| `std::integer_sequence`      | Represents arbitrary integer values                     | A sequence type                                        |
| `std::index_sequence`        | Represents a sequence of indices                        | An alias for `std::integer_sequence<std::size_t, ...>` |
| `std::make_integer_sequence` | Automatically generates integers from zero to \\(N-1\\) | A sequence type                                        |
| `std::make_index_sequence`   | Automatically generates indices                         | A `std::index_sequence` type                           |
| `std::index_sequence_for`    | Generates indices based on the number of types          | A `std::index_sequence` type                           |

These utilities complement one another, and each has a specific role. Generic template implementations commonly combine sequence-generation utilities with the corresponding sequence types.

## Important Considerations for Performance and Implementation

### Runtime Cost

`std::integer_sequence` is a type-level construct and does not need to create a runtime array to represent its sequence.

However, this does not mean that every piece of code using it is necessarily cost-free. If the sequence is used to generate multiple function calls, construct objects, or execute runtime operations, those operations still have their actual runtime costs.

For example, using `std::index_sequence` to expand access to tuple elements can eliminate repetitive code, but the operations performed on those elements still have their own costs.

### Compile-Time Cost

Generating large sequences can affect compilation time, compiler memory consumption, and template instantiation depth.

The standard does not mandate a specific internal algorithm for generating sequences. Consequently, implementation details may differ among standard library implementations.

Some implementations use divide-and-conquer algorithms to reduce template recursion depth. This can be particularly important when generating large sequences.

Therefore, if a project needs to generate a very large number of indices at compile time, compilation time and resource consumption should also be measured on the target compilers.

### Differences from Ordinary Loops

Ordinary loops are suitable for iterating over elements at runtime, but indices used as template arguments must be known at compile time.

For example, if we want to call `std::get<i>` for the elements of a tuple, `i` cannot simply be an ordinary loop variable. In such cases, an index sequence provides an appropriate way to generate a collection of distinct calls at compile time.

This does not mean that `std::integer_sequence` is universally better than loops. The right choice depends on the problem:

- For iterating over ordinary runtime data, loops are often simpler.
- For expanding templates and generating calls that require compile-time constant values, index sequences are more appropriate.

## Limitations and Common Mistakes

### Using a Runtime Value as a Template Argument

A common mistake is attempting to generate a sequence based on a value that becomes known only at runtime.

```
#include <utility>

int main() {
    int count = 5;

    // This is invalid because count is not a constant expression.
    // using Sequence = std::make_index_sequence<count>;
}
```

The value of `N` in `std::make_index_sequence<N>` must be known at compile time. Therefore, if the number of elements is determined only while the program is running, this utility cannot directly generate a sequence of that size.

In such situations, runtime data structures and algorithms are usually more appropriate.

### Confusing a Sequence Type with an Object Containing Data

The following declaration:

```
using Sequence = std::integer_sequence<int, 10, 20, 30>;
```

does not create an array containing three elements. It simply introduces a type.

This type can be used to extract values at compile time, generate an array, or expand parameter packs. However, if the goal is to store data that can be traversed at runtime, an appropriate data structure must be used.

### Ignoring Empty Sequences

In generic templates, the number of elements may be zero. If an implementation is designed only for non-empty sequences, compilation errors may occur.

For example, in patterns based on initializer lists, adding an initial fixed element can prevent the list from becoming empty. However, each implementation must still ensure that the resulting element types and expressions are valid.

### Assuming a Specific Internal Algorithm

The standard specifies the result that `std::make_integer_sequence` must produce, but it does not mandate the internal algorithm used to generate the sequence.

Therefore, we should not assume that every compiler uses linear recursion, divide-and-conquer, or any particular implementation technique. If implementation details affect the project's performance, they should be examined in the target environment.

## Comparing with Features in Newer C++ Standards

### The Role of `std::integer_sequence` in C++17

In C++17, utilities such as `std::apply` simplify many common use cases for `std::index_sequence`.

Instead of manually implementing tuple unpacking, we can use the standard function. However, this does not mean that `std::integer_sequence` has been removed from the standard library or has become obsolete.

It remains useful in generic template implementations, index sequence generation, and the design of type-level mechanisms.

### Differences from Fold Expressions

Fold expressions, introduced in C++17, make it possible to perform operations over parameter packs with more concise code.

For example, summing a parameter pack in C++17 can be written as follows:

```
template<class... Values>
constexpr auto sum(Values... values) {
    return (values + ...);
}
```

This feature reduces the need for older parameter pack expansion patterns in many cases, but it is not a complete replacement for `std::integer_sequence`.

Fold expressions are suitable for combining the elements of a parameter pack, whereas `std::integer_sequence` is designed to represent a sequence of compile-time constant values at the type level. Both utilities can also be used together in a single implementation.

## Final Summary

`std::integer_sequence`, introduced in C++14, provides a standard way to represent a sequence of integer values at compile time. It is particularly useful when indices or the number of elements must be known at compile time in template code.

The most important points to remember are:

- `std::integer_sequence` is a type-level construct, not a runtime container.
- Its `value_type` member specifies the value type, while `size()` returns the number of elements.
- `std::make_integer_sequence` generates a sequence from zero through \\(N-1\\).
- `std::index_sequence` is designed to represent indices.
- `std::make_index_sequence` generates indices from a specified compile-time count.
- `std::index_sequence_for` derives the number of indices from the number of types in a parameter pack.
- An important application of these utilities is expanding `std::tuple` elements and converting them into independent function arguments.
- These utilities should be used when compile-time operations require them. For ordinary runtime iteration, loops and conventional containers are often simpler choices.
- The standard does not specify the internal sequence-generation algorithm, so code should not depend on a particular implementation technique.

Overall, `std::integer_sequence` is a fundamental tool for understanding and implementing advanced template metaprogramming techniques. Learning to use it correctly makes parameter pack expansion, generic template design, and the use of C++ standard library features easier to understand.

---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>