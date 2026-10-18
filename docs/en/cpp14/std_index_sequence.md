<div align="center">

[🇺🇸 English](./std_index_sequence.md) | [🇮🇷 فارسی](../../fa/cpp14/std_index_sequence.md)

</div>

---

# Comprehensive Guide to `std::index_sequence` in C++14

## Introduction

One of the important features of template metaprogramming in the C++14 standard library is the `std::integer_sequence` family, particularly `std::index_sequence`. These utilities make it possible to represent a sequence of integer values as a type at compile time, allowing developers to implement operations that require consecutive indices without manually generating code for each index.

For example, suppose you have a `std::tuple` containing several values of different types and want to pass all its elements as arguments to another function. You can iterate over elements at runtime in some contexts, but accessing each element independently through `std::get` generally requires the index to be known at compile time. This is one of the primary use cases for `std::index_sequence`.

Introduced in C++14, this feature is a powerful tool for writing generic, reusable code when combined with Variadic Templates, Parameter Pack Expansion, and `std::tuple`.

## Understanding `std::integer_sequence`

`std::integer_sequence` is a class template that represents a sequence of integer values as non-type template parameters.

Its conceptual definition can be expressed as follows:

```
template<class T, T... Values>
class integer_sequence;
```

This snippet illustrates the conceptual template definition. For actual use, include the standard `<utility>` header and use `std::integer_sequence`.

In this template:

- `T` specifies the type of the values.
- The parameter pack `Values...` represents a sequence of compile-time constant values.
- `integer_sequence` is a type-level sequence, not a runtime array.

For example, the following type represents a sequence containing the three values `10`, `20`, and `30`:

```
#include <utility>

using Values = std::integer_sequence<int, 10, 20, 30>;
```

In this example, `Values` is the name of a type. This declaration does not create an array, nor are the three values stored as data members in an object. The sequence information is encoded in the type's template parameters.

### Examining the Main Capabilities

`std::integer_sequence` provides two important members for examining the sequence's size and value type:

```
#include <type_traits>
#include <utility>

using Values = std::integer_sequence<int, 10, 20, 30>;

static_assert(Values::size() == 3);
static_assert(
    std::is_same<Values::value_type, int>::value
);
```

The `value_type` member identifies the type of the sequence's values, while `size()` returns the number of values as a compile-time constant.

## Introducing the `std::integer_sequence` Family

C++14 introduces several core utilities for creating and using integer sequences, each with a specific purpose.

| Utility                      | Purpose                                                                      |
| ---------------------------- | ---------------------------------------------------------------------------- |
| `std::integer_sequence`      | Represents a sequence of integer values of a specified type                  |
| `std::make_integer_sequence` | Automatically generates a sequence of integer values                         |
| `std::index_sequence`        | Represents a sequence of `std::size_t` indices                               |
| `std::make_index_sequence`   | Generates a sequence of consecutive indices starting at zero                 |
| `std::index_sequence_for`    | Generates an index sequence matching the number of types in a parameter pack |

These utilities are defined in the standard `<utility>` header.

## Differences Between `std::index_sequence` and `std::integer_sequence`

`std::index_sequence` is an alias template for `std::integer_sequence<std::size_t, Indices...>`.

Its conceptual definition is:

```
template<std::size_t... Indices>
using index_sequence =
    std::integer_sequence<std::size_t, Indices...>;
```

Therefore, `std::index_sequence` does not introduce functionality independent of `std::integer_sequence`. Instead, it is a specialized form intended for representing indices.

For example, the following two definitions represent equivalent types:

```
#include <cstddef>
#include <type_traits>
#include <utility>

using A = std::index_sequence<0, 1, 2, 3>;

using B = std::integer_sequence<
    std::size_t, 0, 1, 2, 3
>;

static_assert(std::is_same<A, B>::value);
```

Using `std::index_sequence` is generally more appropriate when the values are intended for indexing, accessing elements, or expanding a parameter pack. When a sequence of another integer type is required, `std::integer_sequence` is the more general choice.

## Generating a Sequence with `std::make_index_sequence`

The `std::make_index_sequence` template is one of the most important utilities in this family. Given a non-negative integer template argument, it generates a type containing consecutive indices from zero through one less than that number.

Its usage is:

```
std::make_index_sequence<N>
```

For example, the following type is equivalent to `std::index_sequence<0, 1, 2, 3, 4>`:

```
#include <utility>

using Indices = std::make_index_sequence<5>;
```

The generation rule can be expressed as follows:

\\[ \operatorname{make\\\_index\\\_sequence}\langle N\rangle = \operatorname{index\\\_sequence}\langle 0,1,\ldots,N-1\rangle \\]

When \\(N=0\\), the result is the empty sequence `std::index_sequence<>`.

### Examining Several Examples

```
#include <type_traits>
#include <utility>

using Empty = std::make_index_sequence<0>;
using One   = std::make_index_sequence<1>;
using Three = std::make_index_sequence<3>;

static_assert(
    std::is_same<Empty, std::index_sequence<>>::value
);

static_assert(
    std::is_same<One, std::index_sequence<0>>::value
);

static_assert(
    std::is_same<
        Three,
        std::index_sequence<0, 1, 2>
    >::value
);
```

The important distinction is that the input specifies the number of indices, not the last index. Consequently, `std::make_index_sequence<3>` generates the three indices `0`, `1`, and `2`; index `3` is not included.

## Generating a Sequence with `std::make_integer_sequence`

The `std::make_integer_sequence` template follows the same principle but allows the sequence to use a specified integer type.

Its syntax is:

```
std::make_integer_sequence<T, N>
```

Here, `T` specifies the type of the values, while `N` specifies the number of generated values.

For example:

```
#include <type_traits>
#include <utility>

using Sequence =
    std::make_integer_sequence<int, 5>;

using Expected =
    std::integer_sequence<int, 0, 1, 2, 3, 4>;

static_assert(
    std::is_same<Sequence, Expected>::value
);
```

This template is suitable for generating general integer sequences. However, the generated sequence starts at zero, and its values have type `T`.

For example, when `T` is `unsigned int`, the values in the sequence are also of type `unsigned int`.

### Generating a Sequence Versus Defining One Manually

Manually defining a sequence may seem straightforward when only a few indices are needed:

```
using Indices = std::index_sequence<0, 1, 2, 3, 4>;
```

However, when the number of indices depends on the size of an array, a tuple, or a parameter pack, manually defining the sequence is no longer practical.

In these situations, `std::make_index_sequence` allows the number of indices to be derived directly from the data structure. The code remains valid when the number of elements changes.

## Using `std::index_sequence_for`

The `std::index_sequence_for` template is designed for situations where you need as many indices as there are types in a parameter pack.

Its conceptual definition is:

```
template<class... Types>
using index_sequence_for =
    std::make_index_sequence<sizeof...(Types)>;
```

Here, `sizeof...(Types)` determines the number of types in the parameter pack.

For example:

```
#include <type_traits>
#include <utility>

using Indices =
    std::index_sequence_for<int, double, char>;

using Expected =
    std::index_sequence<0, 1, 2>;

static_assert(
    std::is_same<Indices, Expected>::value
);
```

This template is particularly useful in functions that operate on a parameter pack of types and also need a corresponding parameter pack of indices.

An important detail is that `std::index_sequence_for` counts the types in the pack. It does not inspect their values or contents.

## Understanding Parameter Pack Expansion

To understand the practical applications of `std::index_sequence`, it is helpful to understand Parameter Pack Expansion.

In variadic templates, a parameter pack can contain multiple parameters. Using an appropriate pattern, the compiler expands the pack into multiple expressions.

For example:

```
template<class... Types>
void process(Types... values);
```

This function can accept a variable number of arguments. However, when you need to access the individual elements of a tuple, having a parameter pack alone is not sufficient: accessing `std::get<I>` requires the index `I` to be known at compile time.

In such situations, `std::index_sequence` provides a pack of indices that can be passed to a helper function for expansion.

For example, a helper function can use a pattern like this:

```
template<std::size_t... Indices>
void helper(std::index_sequence<Indices...>);
```

When this function is instantiated with `std::index_sequence<0, 1, 2>`, the `Indices...` pack contains the three values `0`, `1`, and `2`.

These values can then be used in an expansion such as `std::get<Indices>(tuple)...`, producing three separate expressions.

## Practical Application: Iterating Over a `std::tuple`

One of the most common uses of `std::index_sequence` is accessing all the elements of a tuple through Parameter Pack Expansion.

In C++14, you can write a function that sends every tuple element to `std::cout` individually.

```
#include <iostream>
#include <tuple>
#include <utility>

template<class Tuple, std::size_t... Indices>
void printTupleImpl(
    const Tuple& tuple,
    std::index_sequence<Indices...>
) {
    ((std::cout << std::get<Indices>(tuple) << ' '), ...);
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

Note: The Fold Expression used in this example, `(..., ...)`, is a C++17 feature. Therefore, this code is a C++17 implementation and must not be presented as code that can be compiled in C++14.

In C++14, the pack must be expanded using a technique compatible with Variadic Templates, such as an initializer list or another helper mechanism.

A C++14-compatible version is:

```
#include <iostream>
#include <tuple>
#include <utility>

template<class Tuple, std::size_t... Indices>
void printTupleImpl(
    const Tuple& tuple,
    std::index_sequence<Indices...>
) {
    using Expand = int[];
    (void)Expand{
        0,
        ((std::cout << std::get<Indices>(tuple) << ' '), 0)...
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

In this implementation, the expression `((std::cout << std::get<Indices>(tuple) << ' '), 0)...` generates one expression for each index. The helper array provides a way to force these expressions to be evaluated in C++14.

This is a well-known pre-Fold Expression technique. In C++17, Fold Expressions are generally more readable and are a better choice for such applications.

## Advanced Application: Implementing a Function Similar to `std::apply`

The standard function `std::apply` was introduced in C++17. It invokes a callable object with the elements of a tuple as its arguments. Before this feature was introduced, similar behavior could be implemented in C++14 using `std::index_sequence`.

Suppose we have a function that accepts three arguments of different types:

```
#include <iostream>

void calculate(int a, double b, char c) {
    std::cout << a << ", " << b << ", " << c << '\n';
}
```

The goal is to pass the tuple elements directly to this function without hard-coding the number or types of arguments in the helper function.

### Implementing a Generic C++14 Version

```
#include <functional>
#include <iostream>
#include <tuple>
#include <type_traits>
#include <utility>

template<class Function, class Tuple, std::size_t... Indices>
auto applyImpl(
    Function&& function,
    Tuple&& tuple,
    std::index_sequence<Indices...>
)
    -> decltype(
        std::forward<Function>(function)(
            std::get<Indices>(std::forward<Tuple>(tuple))...
        )
    )
{
    return std::forward<Function>(function)(
        std::get<Indices>(std::forward<Tuple>(tuple))...
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
                    typename std::remove_reference<Tuple>::type
                >::value
            >{}
        )
    )
{
    return applyImpl(
        std::forward<Function>(function),
        std::forward<Tuple>(tuple),
        std::make_index_sequence<
            std::tuple_size<
                typename std::remove_reference<Tuple>::type
            >::value
        >{}
    );
}

void calculate(int a, double b, char c) {
    std::cout << a << ", " << b << ", " << c << '\n';
}

int main() {
    auto values = std::make_tuple(10, 3.14, 'A');

    apply(calculate, values);
}
```

Several important concepts are combined in this implementation:

- The `apply` function obtains the number of tuple elements using `std::tuple_size`.
- The `std::make_index_sequence` template generates the required sequence of indices.
- The `applyImpl` function expands the index pack using `std::get`.
- Using `std::forward` makes it possible to preserve the value categories of the arguments.
- The `decltype` expression determines the return type according to the result of invoking the callable.

The design principle is that the code illustrates the implementation pattern, but a general-purpose library implementation must carefully consider additional cases, including empty tuples, `void`-returning callables, forwarding of reference arguments, and callable constraints. In addition, using `apply` in the global namespace is not a good choice for a public library because `std::apply` has existed since C++17.

In projects using C++17 or later, the standard `std::apply` should generally be preferred.

## Application: Constructing Objects from an Array

Another application of `std::index_sequence` is converting the elements of an array into arguments for a constructor or function.

For example, suppose we have a class with a three-parameter constructor:

```
class Point {
public:
    Point(int x, int y, int z)
        : x_(x), y_(y), z_(z) {}

private:
    int x_;
    int y_;
    int z_;
};
```

If the values are already stored in a `std::array`, compile-time indices can be used to pass them to the constructor.

```
#include <array>
#include <cstddef>
#include <utility>

class Point {
public:
    Point(int x, int y, int z)
        : x_(x), y_(y), z_(z) {}

private:
    int x_;
    int y_;
    int z_;
};

template<class T, std::size_t Size, std::size_t... Indices>
Point makePointImpl(
    const std::array<T, Size>& values,
    std::index_sequence<Indices...>
) {
    return Point(values[Indices]...);
}

template<class T, std::size_t Size>
Point makePoint(const std::array<T, Size>& values) {
    static_assert(Size == 3, "Point requires exactly three values");

    return makePointImpl(
        values,
        std::make_index_sequence<Size>{}
    );
}
```

This example demonstrates how `std::index_sequence` can transform an indexed data structure into an argument pack.

However, the return type is fixed as `Point`, and the number of elements must be exactly three. For a more generic design, both the destination type and the number of constructor arguments could be parameterized.

## Application: Executing an Operation for Each Index

Sometimes the goal is not to access a tuple, but to execute an operation for a collection of indices. For example, you might want to invoke a function once for each index from zero through `N - 1`.

In C++14, an initializer-list technique can be used to expand the pack while ensuring a specified evaluation order:

```
#include <cstddef>
#include <iostream>
#include <utility>

template<std::size_t... Indices>
void printIndices(std::index_sequence<Indices...>) {
    using Expand = int[];

    (void)Expand{
        0,
        ((std::cout << Indices << '\n'), 0)...
    };
}

int main() {
    printIndices(std::make_index_sequence<5>{});
}
```

The program produces the following output:

```
0
1
2
3
4
```

In this example, each value in the `Indices...` pack is expanded into a separate expression.

This technique is particularly useful when a set of similar operations must be generated at compile time, but each operation must execute with a different index.

However, if the operation is simply a conventional runtime loop over data, an ordinary loop is simpler and more readable.

## Using `std::index_sequence` in Template Metaprogramming

In template metaprogramming, types can carry information that is inspected and processed at compile time.

For example, a function template can report the number of indices in a sequence:

```
#include <cstddef>
#include <iostream>
#include <utility>

template<std::size_t... Indices>
void reportSize(std::index_sequence<Indices...>) {
    std::cout << sizeof...(Indices) << '\n';
}

int main() {
    reportSize(std::make_index_sequence<8>{});
}
```

Here, `sizeof...(Indices)` computes the number of elements in the parameter pack at compile time. As a result, the function prints `8` for a sequence containing eight indices.

This example is simple, but the same pattern can be used in more complex templates for tuples, generic classes, data-structure transformations, and algorithms based on the number of parameters.

## A Conceptual Implementation of `std::make_index_sequence`

The C++ standard does not require developers to know the internal algorithm used to generate an index sequence. However, understanding common implementation techniques helps explain compile-time costs and the design of generic templates.

One educational approach is recursive sequence generation. In this method, a sequence of length `N` is constructed by first generating a sequence of length `N - 1` and then adding the next index.

The following simplified version is intended only to illustrate the concept; it is not a complete replacement for the standard implementation:

```
#include <cstddef>

template<std::size_t... Indices>
struct IndexSequence {};

template<std::size_t N, std::size_t... Indices>
struct MakeIndexSequenceImpl
    : MakeIndexSequenceImpl<N - 1, N - 1, Indices...> {};

template<std::size_t... Indices>
struct MakeIndexSequenceImpl<0, Indices...> {
    using type = IndexSequence<Indices...>;
};

template<std::size_t N>
using MakeIndexSequence =
    typename MakeIndexSequenceImpl<N>::type;
```

For example, requesting `MakeIndexSequence<3>` leads the recursive process to generate the values `0`, `1`, and `2`.

This implementation illustrates a simple recursive approach to compile-time sequence generation. However, it should not be assumed that the standard library necessarily uses the same algorithm. Real implementations may use more efficient divide-and-conquer techniques or compiler intrinsics.

Furthermore, this example does not comprehensively handle invalid inputs, such as certain values outside the representable range of the type involved. Its purpose is only to demonstrate the idea of compile-time sequence generation.

## Important Considerations About Compile-Time Costs

Using `std::index_sequence` does not, by itself, introduce a runtime loop. The sequence is represented as a type, and the compiler uses it to generate the necessary template instantiations and pack expansions.

However, for very large sequences, the complexity of the consuming templates and the number of instantiations can increase compilation time and compiler memory consumption.

The important point is that the actual cost depends on the structure of the consuming templates, the sequence-generation algorithm, and the compiler implementation. Therefore, it is not reasonable to assume that every use of `std::index_sequence` has the same cost.

In large projects, avoid unnecessarily generating equivalent sequences multiple times in redundant structures. If compile-time performance becomes a concern, measure it with appropriate tools.

## Limitations and Common Mistakes

### Confusing the Number of Elements with the Last Index

The template `std::make_index_sequence<N>` generates a sequence containing `N` elements, not a sequence whose last element is `N`.

For example, `std::make_index_sequence<4>` is equivalent to `std::index_sequence<0, 1, 2, 3>`.

### Using an Out-of-Range Index

Generating a sequence does not guarantee that every index is valid for a particular data structure. If you use a sequence larger than the number of elements in an array or tuple, the resulting access may be invalid or cause a compilation error.

Therefore, the number of indices must match the size of the data structure.

### Forgetting the Empty-Sequence Case

In C++14, `std::make_index_sequence<0>` produces a valid empty sequence. However, consuming code must also be able to handle an empty parameter pack.

Some initializer-list-based implementations or helper functions may require adjustments because of incorrect assumptions about the number of elements.

### Assuming the Sequence Stores Data

The type `std::index_sequence` is not a runtime array or collection. It represents a set of integer values in template parameters. Using those values to access data requires an actual data structure or an operation that consumes the indices.

### Using It Unnecessarily in Simple Code

If the goal is simply to iterate over a conventional array at runtime, a `for` loop or range-based `for` loop is often the simpler and more appropriate choice.

`std::index_sequence` is most useful when compile-time indices, parameter pack expansion, or the generation of distinct expressions are required.

## Comparing `std::index_sequence` with Features in Newer C++ Standards

Newer versions of C++ have simplified some common applications of `std::index_sequence`, but the utility remains useful for generating and passing index packs.

| Feature                                | Standard version | Main difference                                              |
| -------------------------------------- | ---------------- | ------------------------------------------------------------ |
| `std::integer_sequence` and its family | C++14            | Represents and generates integer sequences                   |
| `std::apply`                           | C++17            | Passes tuple elements to a callable                          |
| Fold Expressions                       | C++17            | Simplifies operations over parameter packs                   |
| Concepts                               | C++20            | Improves constraints and documentation for generic templates |

For example, in C++17, many applications involving passing tuple elements to a callable can use `std::apply`. Fold Expressions can also replace older initializer-list-based techniques in many operations over parameter packs.

However, these features are not complete replacements for `std::index_sequence`. For example, `std::apply` itself can rely on an index-based mechanism to access tuple elements in typical implementations.

## Summary

`std::index_sequence` is a useful foundational tool for template programming in C++14. It represents a sequence of compile-time constant indices and enables parameter pack expansion in contexts where individual indices are required.

The most important points to remember are:

- `std::index_sequence` belongs to the `std::integer_sequence` family and represents indices using `std::size_t`.
- The template `std::make_index_sequence<N>` generates a sequence containing `N` indices, from zero through `N - 1`.
- The template `std::index_sequence_for<Types...>` converts the number of types in a parameter pack into a corresponding sequence of indices.
- Important applications include accessing tuple elements, expanding parameter packs, and passing a variable number of arguments to functions and constructors.
- In C++14, initializer-list-based techniques are a common way to expand parameter packs and control expression evaluation order.
- In C++17 and later, features such as `std::apply` and Fold Expressions make many of these patterns more readable.
- `std::index_sequence` is most valuable when a problem genuinely requires compile-time indices, rather than merely replacing ordinary loops.

Ultimately, the best way to master this feature is to understand its relationship with Variadic Templates and Parameter Pack Expansion. Once that relationship becomes clear, `std::index_sequence` no longer seems like a complex or mysterious utility. Instead, it becomes a natural way to transform a collection of compile-time constant indices into expressions that can be used in generic templates.

---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>