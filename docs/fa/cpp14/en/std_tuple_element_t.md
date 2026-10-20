<div align="center">

[🇺🇸 English](./std_tuple_element_t.md) | [🇮🇷 فارسی](../../fa/cpp14/std_tuple_element_t.md)

</div>

---

# A Comprehensive Guide to `std::tuple_element_t` in C++14

## Introduction and Its Place in the Standard Library

The `std::tuple_element_t` facility was introduced in the C++14 standard and provides a way to access the type of a specific element in a multi-element structure at compile time. It is part of the standard library and belongs to the family of facilities associated with `std::tuple`. It can be used with `std::tuple`, `std::pair`, and `std::array`.

In generic programming and template design, it is sometimes necessary to extract the type of a member without constructing an object, accessing its value, or performing any runtime operation. For example, you might want to obtain the type of the second element of a `std::tuple` to declare a variable, determine a function's return type, or define another template.

The key characteristic of this facility is that its result is a type determined at compile time, not a value computed at runtime.

The following example demonstrates its basic usage:

```
#include <iostream>
#include <string>
#include <tuple>
#include <type_traits>

int main() {
    using Data = std::tuple<int, double, std::string>;

    using FirstType = std::tuple_element_t<0, Data>;
    using SecondType = std::tuple_element_t<1, Data>;
    using ThirdType = std::tuple_element_t<2, Data>;

    static_assert(std::is_same_v<FirstType, int>);
    static_assert(std::is_same_v<SecondType, double>);
    static_assert(std::is_same_v<ThirdType, std::string>);

    FirstType number = 42;
    SecondType value = 3.14;
    ThirdType text = "Hello";

    std::cout << number << '\n';
    std::cout << value << '\n';
    std::cout << text << '\n';
}
```

In this example, three different types are extracted from a `std::tuple`. No `tuple` object is required to perform the extraction, and all type-related checks take place at compile time.

## Core Concepts and Type Extraction Mechanism

### Understanding `std::tuple_element`

`std::tuple_element` is a class template in the `std` namespace that extracts the type of an element at a specified index from a supported type.

Its general form is:

```
template<std::size_t I, class T>
struct tuple_element;
```

In this definition:

- `I` is the index of the desired element and has type `std::size_t`.
- `T` is the type of the structure whose elements are being examined.
- The `type` member of `std::tuple_element<I, T>` identifies the type of the selected element.

For example, the following code extracts the type of the third element of a `tuple`:

```
using TupleType = std::tuple<int, float, long>;
using ElementType = std::tuple_element<2, TupleType>::type;
```

In C++14, the same expression can be written more concisely:

```
using ElementType = std::tuple_element_t<2, TupleType>;
```

### Differences Between `std::tuple_element` and `std::tuple_element_t`

`std::tuple_element` is a class template, whereas `std::tuple_element_t` is an alias template.

Conceptually, the alias was defined in C++14 as follows:

```
template<std::size_t I, class T>
using tuple_element_t = typename tuple_element<I, T>::type;
```

This definition shows that `std::tuple_element_t` does not introduce a new type-extraction mechanism. Instead, it simplifies access to the `type` member.

| Feature            | `std::tuple_element`            | `std::tuple_element_t`                                 |
| ------------------ | ------------------------------- | ------------------------------------------------------ |
| Definition         | Class template                  | Alias template                                         |
| Access to the type | Through `::type`                | Directly                                               |
| Standard version   | C++11                           | C++14                                                  |
| Common use         | Accessing the trait's structure | Defining types with `using` and working with templates |

In modern code, `std::tuple_element_t` is generally more readable. However, `std::tuple_element` remains useful for understanding the structure of type traits and working with older code.

### Differences Between an Element's Type and Its Value

The fundamental point is that `std::tuple_element_t` extracts an element's type, not its value.

For example, in the following code, the first element has the value `42`, but `std::tuple_element_t` produces the type `int`:

```
#include <tuple>
#include <type_traits>

int main() {
    using Data = std::tuple<int, double>;

    Data values{42, 3.14};

    using FirstType = std::tuple_element_t<0, Data>;

    static_assert(std::is_same_v<FirstType, int>);

    (void)values;
}
```

The `values` object is not necessary for type extraction. Even if the object did not exist, the type of the first element could still be determined from the definition of `Data`.

## Supported Standard Types and Their Usage

### Using `std::tuple`

A `std::tuple` can hold multiple values of different types. This makes it suitable for returning multiple values from a function, building generic data structures, and representing heterogeneous collections.

For example:

```
#include <tuple>
#include <type_traits>

using Record = std::tuple<int, double, const char*>;

using IdType = std::tuple_element_t<0, Record>;
using ScoreType = std::tuple_element_t<1, Record>;
using NameType = std::tuple_element_t<2, Record>;

static_assert(std::is_same_v<IdType, int>);
static_assert(std::is_same_v<ScoreType, double>);
static_assert(std::is_same_v<NameType, const char*>);
```

In this example, each type is extracted using the index corresponding to its position in the `tuple`. The order of the types matters: changing the order of the template arguments in `std::tuple` can change the result of the extraction.

### Using `std::pair`

`std::pair` contains two members with independent types and is also supported by `std::tuple_element`.

```
#include <string>
#include <tuple>
#include <type_traits>
#include <utility>

using Entry = std::pair<int, std::string>;

using KeyType = std::tuple_element_t<0, Entry>;
using ValueType = std::tuple_element_t<1, Entry>;

static_assert(std::is_same_v<KeyType, int>);
static_assert(std::is_same_v<ValueType, std::string>);
```

This is particularly useful when designing generic algorithms. An algorithm that works with a key-value pair can extract the type of each member without depending on the actual values.

### Using `std::array`

`std::array` represents a collection of elements that all have the same type. Its element type can also be extracted through `std::tuple_element`.

```
#include <array>
#include <tuple>
#include <type_traits>

using Numbers = std::array<int, 5>;

using ElementType = std::tuple_element_t<0, Numbers>;

static_assert(std::is_same_v<ElementType, int>);
```

The important difference is that all elements of a `std::array` have the same type. Therefore, for an array with a positive number of elements, extracting the type at any valid index produces the same element type.

### Handling `const` and `volatile`

The C++ standard provides specializations of `std::tuple_element` for `const`, `volatile`, and `const volatile`-qualified types, including the supported `tuple`, `pair`, and `array` types. Consequently, qualifiers on the outer type are preserved in the extracted element type.

```
#include <tuple>
#include <type_traits>

using Data = std::tuple<int, double>;

using MutableType = std::tuple_element_t<0, Data>;
using ConstType = std::tuple_element_t<0, const Data>;
using VolatileType = std::tuple_element_t<0, volatile Data>;
using ConstVolatileType = std::tuple_element_t<0, const volatile Data>;

static_assert(std::is_same_v<MutableType, int>);
static_assert(std::is_same_v<ConstType, const int>);
static_assert(std::is_same_v<VolatileType, volatile int>);
static_assert(std::is_same_v<ConstVolatileType, const volatile int>);
```

This behavior is important in generic code because the extracted type is not necessarily identical to the underlying stored type. It may also carry qualifiers from the outer type.

## Indexing Rules and Access Limitations

### Zero-Based Indexing

Element indices in `std::tuple_element_t` start at zero. Therefore, for a `tuple` containing \\(N\\) elements, a valid index must satisfy:

\\[ 0 \le I < N \\]

For example, if a `tuple` contains three elements, the valid indices are `0`, `1`, and `2`.

```
#include <tuple>
#include <type_traits>

using Data = std::tuple<int, double, char>;

static_assert(std::is_same_v<std::tuple_element_t<0, Data>, int>);
static_assert(std::is_same_v<std::tuple_element_t<1, Data>, double>);
static_assert(std::is_same_v<std::tuple_element_t<2, Data>, char>);
```

### Behavior with an Invalid Index

Using an index outside the range of elements in a `tuple`, `pair`, or `array` is not a valid use of this trait, and the standard does not define a usable type result for such an index. In practice, the compiler will generally issue an error.

```
#include <tuple>

using Data = std::tuple<int, double>;

// Invalid: index 2 is outside the valid range [0, 2).
// using InvalidType = std::tuple_element_t<2, Data>;
```

In this example, the error occurs at compile time, not at runtime. To prevent such errors, if the index comes from a template parameter, consider checking its validity with template constraints or `static_assert`.

### Behavior with `std::tuple<>`

An empty `tuple` contains no elements, so no index is valid for extracting an element type.

```
#include <tuple>

using Empty = std::tuple<>;

// Invalid: Empty has no elements.
// using ElementType = std::tuple_element_t<0, Empty>;
```

This limitation matters when designing generic templates. If your algorithm must also accept an empty `tuple`, do not use `std::tuple_element_t<0, T>` without first checking the number of elements.

## Applications in Generic Template Design

### Extracting a Type to Declare a Variable

One practical use of this facility is declaring a variable whose type is extracted from another structure. This avoids repeating type names and reduces the coupling between code and implementation details.

```
#include <string>
#include <tuple>

using UserRecord = std::tuple<int, std::string, double>;

using NameType = std::tuple_element_t<1, UserRecord>;

NameType userName = "Ali";
```

In this example, if the type of the name changes in the definition of `UserRecord`, `NameType` automatically reflects the updated definition.

This approach can help prevent duplication and type inconsistencies in libraries that accept composite types as parameters.

### Using the Extracted Type as a Template's Result Type

In some designs, the output type of a generic class or function depends on the type of one of its input's elements.

```
#include <tuple>
#include <type_traits>

template<class Tuple>
struct FirstElement {
    using type = std::tuple_element_t<0, Tuple>;
};

using Data = std::tuple<long, double, char>;
using Result = FirstElement<Data>::type;

static_assert(std::is_same_v<Result, long>);
```

In this example, `FirstElement` extracts the type of the first element and exposes it as `type`. This pattern is useful in custom type traits and generic infrastructure.

However, if the goal is simply to define an alias, `std::tuple_element_t` can be used directly, without introducing an intermediate class.

### Combining It with `std::conditional_t`

The extracted type can be combined with other metaprogramming utilities to select a final type based on a compile-time condition.

```
#include <tuple>
#include <type_traits>

using Data = std::tuple<int, double, char>;

using SecondType = std::tuple_element_t<1, Data>;

using SelectedType = std::conditional_t<
    std::is_floating_point_v<SecondType>,
    long,
    int
>;

static_assert(std::is_same_v<SelectedType, long>);
```

In this example, the type of the second element is extracted first. Then, `std::conditional_t` selects either `long` or `int` based on that type's properties.

Note that `std::conditional_t` does not guarantee that only the selected type argument must be valid. If one of the type arguments is invalid, compilation may fail even when that argument corresponds to the unselected branch.

### Using `decltype` to Extract an Expression's Type

Both `std::tuple_element_t` and `decltype` are useful for extracting type information, but they serve different purposes.

- `std::tuple_element_t` extracts an element's type from a structure's type and an index.
- `decltype` extracts the type of a specific expression, and its result can depend on the expression's value category.

For example:

```
#include <tuple>
#include <type_traits>
#include <utility>

using Data = std::tuple<int, double>;

using ElementType = std::tuple_element_t<0, Data>;

using LvalueReferenceType =
    decltype(std::get<0>(std::declval<Data&>()));

using ConstLvalueReferenceType =
    decltype(std::get<0>(std::declval<const Data&>()));

static_assert(std::is_same_v<ElementType, int>);
static_assert(std::is_same_v<LvalueReferenceType, int&>);
static_assert(std::is_same_v<ConstLvalueReferenceType, const int&>);
```

In this example, the type of the element itself is `int`, regardless of the access expression. However, the type of the access expression can be `int&` or `const int&`.

This distinction is particularly important when designing APIs that must preserve reference semantics.

## Relationship with `std::get`

### Differences Between Type Extraction and Value Access

`std::tuple_element_t` is closely related to `std::get`, but the two are not interchangeable.

| Feature            | `std::tuple_element_t`       | `std::get`                                            |
| ------------------ | ---------------------------- | ----------------------------------------------------- |
| Purpose            | Extracts an element's type   | Accesses an element's value                           |
| Main operation     | Compile-time type extraction | Access to an object at runtime                        |
| Primary inputs     | Structure type and index     | Structure object and index                            |
| Result             | A type                       | An expression with a specific type and value category |
| Requires an object | No                           | Yes, to access a value                                |

For example:

```
#include <tuple>
#include <type_traits>

int main() {
    std::tuple<int, double> data{10, 2.5};

    using FirstType = std::tuple_element_t<0, decltype(data)>;

    static_assert(std::is_same_v<FirstType, int>);

    int& first = std::get<0>(data);
    first = 20;
}
```

In this example, `std::tuple_element_t` extracts the element's type, whereas `std::get` allows the element's value to be read or modified.

### Preserving References When Accessing Elements

When designing templates, distinguish between the type stored in a `tuple` and the type of the expression produced by `std::get`.

`std::get` provides different overloads for various `tuple` value categories, including mutable objects, `const` objects, `rvalues`, and `const rvalues`. As a result, the type of the resulting expression can vary.

```
#include <tuple>
#include <type_traits>
#include <utility>

using Data = std::tuple<int, double>;

static_assert(
    std::is_same_v<
        std::tuple_element_t<0, Data>,
        int
    >
);

static_assert(
    std::is_same_v<
        decltype(std::get<0>(std::declval<Data&>())),
        int&
    >
);

static_assert(
    std::is_same_v<
        decltype(std::get<0>(std::declval<Data&&>())),
        int&&
    >
);
```

The design principle is straightforward: use `std::tuple_element_t` when you need the stored element type. If the reference type produced by actual access matters, use `decltype` with the appropriate access expression.

## Using Structured Bindings

### Relationship with Structured Bindings in C++17

Structured bindings were introduced in C++17. They allow a composite object to be decomposed into individually named bindings.

```
#include <string>
#include <tuple>

int main() {
    std::tuple<int, std::string, double> data{
        42,
        "Alice",
        98.5
    };

    auto [id, name, score] = data;

    (void)id;
    (void)name;
    (void)score;
}
```

In this example, the compiler uses the rules for structured bindings to make the elements of `data` accessible through the names `id`, `name`, and `score`.

For tuple-like types, structured bindings rely on mechanisms involving `std::tuple_size`, `std::tuple_element`, and `get` functions. Consequently, `std::tuple_element` is one of the underlying components that support this feature.

### Differences Between Type Extraction and Structural Decomposition

If you only need the type of one element, `std::tuple_element_t` is simpler and more direct. Structured bindings are more suitable when you want to access several elements of an object.

For example, if the goal is simply to define an alias for the type of the second element, the following is sufficient:

```
#include <tuple>

using Data = std::tuple<int, double, char>;
using SecondType = std::tuple_element_t<1, Data>;
```

In this case, there is no need to construct an object or define a structured binding.

## Implementing Custom Tuple-Like Types

### The Role of `std::tuple_element` in Custom Types

The standard library allows certain components of the tuple-like protocol to be specialized for custom types. When designing such types, you can provide appropriate specializations of `std::tuple_size` and `std::tuple_element` for your own type, along with corresponding `get` functions.

This mechanism allows generic code to extract member type information from a custom structure in a way similar to `std::tuple`.

However, specializing `std::tuple_element` alone does not make a type fully tuple-like. To support structured bindings and algorithms based on the tuple-like protocol, the required components must be implemented consistently.

### Example of a Custom Type with Two Members

The following example defines a simple structure that can be used with the tuple-like approach.

```
#include <cstddef>
#include <tuple>
#include <type_traits>
#include <utility>

struct Point {
    int x;
    double y;
};

namespace std {

template<>
struct tuple_size<Point>
    : integral_constant<size_t, 2> {};

template<>
struct tuple_element<0, Point> {
    using type = int;
};

template<>
struct tuple_element<1, Point> {
    using type = double;
};

}

template<std::size_t I>
decltype(auto) get(Point& point) {
    static_assert(I < 2);

    if constexpr (I == 0) {
        return (point.x);
    } else {
        return (point.y);
    }
}

template<std::size_t I>
decltype(auto) get(const Point& point) {
    static_assert(I < 2);

    if constexpr (I == 0) {
        return (point.x);
    } else {
        return (point.y);
    }
}

int main() {
    using XType = std::tuple_element_t<0, Point>;
    using YType = std::tuple_element_t<1, Point>;

    static_assert(std::is_same_v<XType, int>);
    static_assert(std::is_same_v<YType, double>);

    Point point{10, 20.5};

    auto [x, y] = point;

    (void)x;
    (void)y;
}
```

In this example, the specializations of `std::tuple_size` and `std::tuple_element` provide information about the number and types of the members. The `get` functions provide access to the members by index.

The use of `if constexpr` in this code requires C++17. Therefore, although the example illustrates the relationship between `std::tuple_element_t` and tuple-like types, the complete code cannot be compiled as C++14.

This implementation also illustrates only the general structure of the protocol. A production-quality generic type should provide appropriate overloads for different reference categories, `const` correctness, access to `rvalues`, and any other details required by the intended use. Specializations of standard templates in `std` must also be limited to cases explicitly permitted by the standard. In this example, the custom type `Point` belongs to the application.

## Limitations and Design Considerations

### Extracting a Type from Runtime Data

`std::tuple_element_t` cannot select different element types based on a runtime index.

For example, if a `tuple` contains elements of different types, a runtime integer cannot be used directly as the index template argument of `std::tuple_element_t`, because the index must be a compile-time constant.

```
#include <cstddef>
#include <tuple>

using Data = std::tuple<int, double, char>;

template<std::size_t I>
using ElementType = std::tuple_element_t<I, Data>;

using FirstType = ElementType<0>;
using ThirdType = ElementType<2>;
```

In this example, the index is specified at each use, allowing the compiler to determine the corresponding type.

If the element must be selected according to runtime input, a different design is generally needed, such as `std::variant`, separate functions, or a dispatch layer. The appropriate approach depends on the problem.

### Differences from `std::remove_cvref_t`

`std::tuple_element_t` does not automatically remove all qualifiers from the input type. If the goal is to extract an element type without `const`, `volatile`, or reference qualifiers, an appropriate utility must be used.

`std::remove_cvref_t` was introduced in C++20 and is not available in C++14. For C++14 compatibility, `std::remove_reference_t` can be combined with `std::remove_cv_t`.

```
#include <tuple>
#include <type_traits>

using Data = std::tuple<int, double>;

using ElementType = std::tuple_element_t<0, const Data>;

using NormalizedType = std::remove_cv_t<
    std::remove_reference_t<ElementType>
>;

static_assert(std::is_same_v<ElementType, const int>);
static_assert(std::is_same_v<NormalizedType, int>);
```

In this example, `ElementType` is `const int` because `Data` is `const`, whereas `NormalizedType` becomes `int` after the qualifiers are removed.

However, if a reference or qualifier is part of your API's type contract, removing it may discard important information. Type normalization should therefore be applied deliberately.

### Differences from `std::decay_t`

`std::decay_t` is also used for type normalization, but its transformations differ from those of `std::remove_cv_t` and `std::remove_reference_t`.

In addition to removing references and top-level cv-qualifiers, `std::decay_t` performs transformations similar to array-to-pointer and function-to-pointer conversions.

Consequently, if the only goal is to remove `const`, `volatile`, or reference qualifiers, using `std::decay_t` may change the type more than necessary.

## Common Mistakes and How to Avoid Them

### Choosing the Wrong Index

One of the most common mistakes is assuming one-based indexing. In `std::tuple_element_t`, index `0` always refers to the first element.

To avoid errors, document element ordering in generic code or define meaningful aliases instead of scattering numeric indices throughout the implementation.

### Assuming the Element Type Matches the Access Expression's Type

The type extracted by `std::tuple_element_t` is not necessarily the same as the type of an expression produced by `std::get`. If your function must return a reference, the element type alone is insufficient to determine the correct return type.

In such cases, use `decltype` to inspect the access expression and preserve its value category and qualifiers when necessary.

### Using Features from Newer C++ Standards

The `std::tuple_element_t` alias belongs to C++14, but tools such as `std::is_same_v` were introduced in C++17. Similarly, `std::remove_cvref_t` requires C++20.

Therefore, using these facilities in a project targeting C++14 without suitable alternatives will cause compilation errors.

For C++14 projects, `std::is_same` with `::value` and the combination of `std::remove_cv` and `std::remove_reference` can be used instead.

### Incorrectly Extending the `std` Namespace

Specializing standard templates for program-defined types is permitted in certain cases, but adding arbitrary members or overloads to the `std` namespace is not allowed unless the standard explicitly permits it.

When implementing a custom tuple-like type, provide only the permitted specializations and avoid introducing arbitrary new names into `std`.

## Compatibility Across C++ Standards

The `std::tuple_element_t` alias was added to the standard library in C++14 and has remained part of the standard library since then.

| C++ Standard    | Status and Related Notes                                                                                                                                     |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| C++11           | `std::tuple_element` is available; access the type through `::type`.                                                                                         |
| C++14           | `std::tuple_element_t` was introduced, simplifying type-oriented code.                                                                                       |
| C++17           | Structured bindings were introduced, expanding the use of tuple-like type information for decomposing objects.                                               |
| C++20           | Utilities such as `std::remove_cvref_t` became available for type normalization.                                                                             |
| C++23 and later | `std::tuple_element_t` remains useful for extracting element types; details of newer protocols and features should be checked against the targeted standard. |

To use this facility in C++14, including the `<tuple>` header is generally sufficient. The `<utility>` header is also relevant when working with `std::pair`, although `<tuple>` is required to use `std::tuple_element_t` itself.

## Summary

`std::tuple_element_t` is a small but important C++14 standard library utility that enables compile-time extraction of an element's type from a multi-element structure.

Its most important practical characteristics are:

- It extracts a type rather than providing an element's value.
- It does not require an object to exist at runtime.
- Indexing starts at zero, and the index must be within the valid range.
- It works with structures such as `std::tuple`, `std::pair`, and `std::array`.
- Top-level qualifiers such as `const` and `volatile` on the structure type can affect the extracted element type.
- To determine the reference type produced by accessing an element, `decltype` with `std::get` is generally more appropriate.
- It is an important component of generic programming, type traits, and the tuple-like protocol.

In professional code, `std::tuple_element_t` is most useful when an element's type is already determined by a structure's definition and needs to be extracted at compile time. When the goal is to access a value, manage references, or select an element based on runtime input, use the tools appropriate to that problem instead.

---

## 🤝 Contributions

<div align="center">

| GitHub                                      | LinkedIn                                                           | Email                                                  | Site                           | Telegram                               |
| ------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------ | -------------------------------------- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](mailto:hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>