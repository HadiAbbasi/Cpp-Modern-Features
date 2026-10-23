<div align="center">

[🇺🇸 English](./spaceship_operator.md) | [🇮🇷 فارسی](../../fa/cpp20/spaceship_operator.md)

</div>

---

# A Comprehensive Guide to the Three-Way Comparison Operator `<=>` in C++20

## Introduction

The Three-way Comparison Operator, represented by `<=>`, is an important feature introduced in C++20. It provides a unified way to compare two values and determine their relative ordering through a single result.

Before this feature was introduced, developers typically had to define multiple operators to implement comparisons for custom types, including `operator<`, `operator<=`, `operator>`, `operator>=`, `operator==`, and `operator!=`. This approach increased code duplication and the likelihood of inconsistencies between comparison operators.

The `<=>` operator addresses much of this problem by introducing a standardized mechanism for three-way comparison. Together with the implicit generation of certain comparison operators, defaulted comparisons, and new comparison result types in the standard library, it simplifies the design of classes with well-defined comparison semantics.

## Understanding Three-Way Comparison

### What Is Three-Way Comparison?

In ordinary comparisons, expressions such as `a < b` and `a == b` produce a `bool` value. Each expression answers a specific question.

In contrast, the `<=>` operator produces a comparison result that describes the relative ordering of two values.

For example:

```
#include <compare>
#include <iostream>

int main() {
    int a = 10;
    int b = 20;

    auto result = a <=> b;

    if (result < 0) {
        std::cout << "a is less than b\n";
    } else if (result > 0) {
        std::cout << "a is greater than b\n";
    } else {
        std::cout << "a and b are equivalent\n";
    }
}
```

In this example, the comparison result has a type such as `std::strong_ordering`. The result can be compared with zero to determine the relative ordering of the two integers.

Importantly, the result of `<=>` is not necessarily an integer. C++20 introduces three primary comparison result types in the standard library, each with different semantics and guarantees.

### Why Use Three-Way Comparison?

The main advantages of this feature include:

- Less duplicated code: There is no need to independently implement every relational operator for each class.
- Greater consistency: Multiple comparison operators can share a common implementation.
- Defaulted comparisons: The compiler can automatically generate comparisons between class members.
- More precise ordering semantics: Developers can specify whether a comparison provides strong, weak, or partial ordering.
- Better library integration: Data structures, sorting algorithms, and custom types can use standardized comparison interfaces.

However, `<=>` does not replace every comparison operation. For example, semantic equality is not necessarily determined by ordering, and many designs still require an independently defined `operator==`.

## Understanding the Comparison Result Types

### The `<compare>` Header

The standard `<compare>` header, introduced in C++20, provides the primary comparison result types and related utilities.

The three most important types are:

- `std::strong_ordering`
- `std::weak_ordering`
- `std::partial_ordering`

These types express how completely a comparison determines the ordering of two values.

### Understanding `std::strong_ordering`

`std::strong_ordering` represents an ordering in which two values are either equivalent or one is strictly less than the other.

It provides three important named values:

- `std::strong_ordering::less`
- `std::strong_ordering::equal`
- `std::strong_ordering::greater`

For example, ordinary integer types have strong ordering.

```
#include <compare>
#include <iostream>

int main() {
    int a = 42;
    int b = 42;

    std::strong_ordering result = a <=> b;

    if (result == std::strong_ordering::equal) {
        std::cout << "Values are equal\n";
    }
}
```

With strong ordering, equivalent values are also equal according to the corresponding equality relation.

This type is appropriate for types whose equality and ordering must be fully consistent, such as integers, numeric identifiers, and certain key-based types.

### Understanding `std::weak_ordering`

`std::weak_ordering` represents a definite ordering in which two values may be equivalent without being exactly equal.

It provides three named values:

- `std::weak_ordering::less`
- `std::weak_ordering::equivalent`
- `std::weak_ordering::greater`

For example, consider comparing strings without distinguishing between uppercase and lowercase letters. The strings `"Alice"` and `"ALICE"` may be equivalent under such a comparison even though they are different under ordinary, case-sensitive string comparison.

```
#include <compare>
#include <cctype>
#include <string_view>

constexpr unsigned char toLower(unsigned char ch) {
    if (ch >= 'A' && ch <= 'Z') {
        return static_cast<unsigned char>(ch - 'A' + 'a');
    }

    return ch;
}

std::weak_ordering compareCaseInsensitive(
    std::string_view lhs,
    std::string_view rhs
) {
    const std::size_t commonSize =
        lhs.size() < rhs.size() ? lhs.size() : rhs.size();

    for (std::size_t i = 0; i < commonSize; ++i) {
        const auto a = toLower(static_cast<unsigned char>(lhs[i]));
        const auto b = toLower(static_cast<unsigned char>(rhs[i]));

        if (a < b) {
            return std::weak_ordering::less;
        }

        if (a > b) {
            return std::weak_ordering::greater;
        }
    }

    if (lhs.size() < rhs.size()) {
        return std::weak_ordering::less;
    }

    if (lhs.size() > rhs.size()) {
        return std::weak_ordering::greater;
    }

    return std::weak_ordering::equivalent;
}
```

In this example, two strings that differ only in letter case can be equivalent.

This implementation is limited to ASCII-style case conversion. It is not a complete replacement for Unicode case folding or locale-sensitive collation.

Furthermore, if exact string equality remains important, equivalence under this ordering must not be assumed to imply equality under `operator==`.

### Understanding `std::partial_ordering`

`std::partial_ordering` represents comparisons in which some pairs of values cannot be classified as less than, greater than, or equivalent to one another.

It provides four important named values:

- `std::partial_ordering::less`
- `std::partial_ordering::equivalent`
- `std::partial_ordering::greater`
- `std::partial_ordering::unordered`

The `unordered` state is useful when no ordering can be established between two values.

A prominent example is comparing floating-point values involving NaN.

```
#include <compare>
#include <cmath>
#include <iostream>
#include <limits>

int main() {
    const double value = 10.0;
    const double nan = std::numeric_limits<double>::quiet_NaN();

    auto result = value <=> nan;

    if (result == std::partial_ordering::unordered) {
        std::cout << "The values are unordered\n";
    }
}
```

NaN is neither less than, greater than, nor equal to an ordinary numeric value. Consequently, three-way comparison of `double` values can produce `unordered`.

This behavior matters when designing types that contain floating-point values.

### Comparing the Result Types

The following table summarizes the main differences between the three types.

| Property                     | `strong_ordering`       | `weak_ordering`                              | `partial_ordering`                       |
| ---------------------------- | ----------------------- | -------------------------------------------- | ---------------------------------------- |
| Less-than result             | Yes                     | Yes                                          | Yes                                      |
| Greater-than result          | Yes                     | Yes                                          | Yes                                      |
| Equivalent result            | Yes                     | Yes                                          | Yes                                      |
| Unordered state              | No                      | No                                           | Yes                                      |
| Equivalence implies equality | Yes                     | No                                           | No                                       |
| Typical applications         | Integers and exact keys | Comparisons that ignore certain distinctions | Floating-point values and partial orders |

In this table, equality means equality according to the corresponding comparison relation. It does not necessarily mean bitwise equality or equality of every internal object representation.

## Using the `<=>` Operator

### Directly Comparing Values

The `<=>` operator can be used directly with values whose types support it.

```
#include <compare>
#include <iostream>

int main() {
    int x = 7;
    int y = 12;

    auto result = x <=> y;

    std::cout << std::boolalpha;
    std::cout << (result < 0) << '\n';
    std::cout << (result == 0) << '\n';
    std::cout << (result > 0) << '\n';
}
```

The output is `true`, `false`, and `false`, respectively.

In C++20, you should not rely on the result being a specific integer such as `-1` or `1`. The comparison contract concerns the meaning of the result and its relationship to zero, not a particular numeric representation.

### Comparison Results Versus Boolean Values

The `<=>` operator produces a comparison result, whereas `<`, `>`, `<=`, and `>=` produce Boolean values.

```
int a = 5;
int b = 8;

auto comparison = a <=> b;
bool isLess = a < b;
```

In this example, `comparison` represents the ordering relationship between the values, while `isLess` only indicates whether the first value is less than the second.

This distinction matters when designing APIs because the result of `<=>` is not intended to be used directly as a Boolean condition.

The following code is invalid:

```
int a = 5;
int b = 8;

if (a <=> b) {
}
```

The standard comparison result types do not implicitly convert to `bool`. To inspect the result, compare it with zero or with an appropriate named value of the corresponding comparison type.

### Comparing Results with Zero

A comparison result can be compared with zero.

```
#include <compare>

int main() {
    int a = 3;
    int b = 9;

    auto result = a <=> b;

    bool less = result < 0;
    bool greater = result > 0;
    bool equivalent = result == 0;
}
```

This approach is suitable for most general-purpose use cases.

When the exact comparison state matters, you can also compare the result with named values of its result type. This is particularly useful when the code needs to distinguish between `equivalent` and `equal`, or explicitly handle `unordered`.

## Automatically Rewriting Comparison Operators

### Understanding Rewritten Comparison Operators

One of the most important features associated with `<=>` in C++20 is operator rewriting.

When a suitable `<=>` operator is defined, the compiler can use it under the language's rewriting rules to support many relational expressions.

For example:

```
#include <compare>

struct Version {
    int major;
    int minor;
    int patch;

    auto operator<=>(const Version&) const = default;
};

int main() {
    Version v1{1, 2, 0};
    Version v2{1, 3, 0};

    bool result = v1 < v2;
}
```

In this example, there is no need to define `operator<` separately. The `<` expression can use the operator-rewriting rules associated with `<=>`.

These rules apply to relational operators such as `<`, `>`, `<=`, and `>=`. However, they should not be confused with automatic generation of every comparison operator.

### Rewriting Operators Versus Generating All Operators

Defining `operator<=>` does not, by itself, mean that `operator==` will always be generated automatically.

For ordinary classes, equality should be defined separately when needed, or declared using `= default`.

However, when `operator<=>` is defaulted, the language's rules also provide for an implicit corresponding `operator==`, subject to the requirements and restrictions of defaulted comparisons.

This distinction is important when designing comparison-capable classes in C++20.

### Using Defaulted Comparisons

Defaulted comparisons allow the compiler to generate comparisons between class members according to the standard rules.

```
#include <compare>
#include <string>

struct Person {
    std::string firstName;
    std::string lastName;
    int age;

    auto operator<=>(const Person&) const = default;
    bool operator==(const Person&) const = default;
};
```

In this example, members are compared in declaration order. If the comparison of the first member produces a result other than equivalence, the remaining members do not need to be examined to determine the ordering.

For equality, the members are checked according to the rules for defaulted equality comparisons.

The explicit definition of `operator==` makes the class interface clear. When a defaulted `<=>` is declared and the applicable requirements are satisfied, the compiler can also implicitly generate the corresponding `operator==`.

### Member Comparison Versus Memory Comparison

Defaulted comparisons perform semantic comparisons of class members. They are not equivalent to byte-by-byte comparison of object memory.

For example, comparing two objects containing `std::string` compares the strings according to their comparison operations, rather than comparing their addresses or internal memory representations directly.

This makes defaulted comparisons suitable for many value types.

However, if a class contains raw pointers, ownership identifiers, external resources, or cached data, defaulted comparison may not reflect the intended semantics of the design.

## Designing Custom Types with `<=>`

### Defaulted Comparisons for Simple Types

For classes whose ordering follows directly from the ordering of their members, using `= default` is usually the best choice.

```
#include <compare>
#include <string>

struct Product {
    int id;
    std::string name;
    double price;

    auto operator<=>(const Product&) const = default;
};
```

In this example, ordering is determined first by `id`, then by `name`, and finally by `price`.

Because `double` can represent NaN, the deduced comparison result type may be `std::partial_ordering`.

Furthermore, because defaulted comparisons follow member declaration order, the member order should reflect the intended semantic ordering.

If products should be compared only by identifier, defaulted comparison is not appropriate for this class because changing the name or price can also change the ordering result.

### Implementing Lexicographical Comparison Manually

Some classes require an ordering that differs from the declaration order of their internal members.

For example, a network address might be ordered first by protocol family and then by its numeric address.

In such cases, `operator<=>` can be implemented manually.

```
#include <compare>
#include <cstdint>

struct NetworkAddress {
    std::uint8_t family;
    std::uint32_t address;

    std::strong_ordering operator<=>(
        const NetworkAddress& other
    ) const {
        if (auto result = family <=> other.family;
            result != 0) {
            return result;
        }

        return address <=> other.address;
    }

    bool operator==(const NetworkAddress& other) const {
        return family == other.family &&
               address == other.address;
    }
};
```

In this implementation, the address family is compared first. If the families differ, that result determines the ordering. Otherwise, the numeric addresses are compared.

This pattern is useful for classes whose ordering consists of several prioritized criteria.

### Using `std::tie` in Older Approaches

Before `<=>` was introduced, a common technique for lexicographical comparison was to use `std::tie`.

```
#include <tuple>
#include <string>

struct User {
    std::string name;
    int id;

    bool operator<(const User& other) const {
        return std::tie(name, id) <
               std::tie(other.name, other.id);
    }
};
```

This technique remains valid and can still be appropriate when a design only requires a specific relational operator.

However, in C++20, `<=>` can simplify the implementation when a class needs a more complete and consistent comparison interface.

An important distinction is that `std::tie` only establishes the ordering of the selected members. It does not automatically generate other comparison operators.

## Rules for Defaulted Comparisons

### The Order of Non-Static Members

For defaulted comparisons, non-static data members are examined in declaration order.

For example:

```
#include <compare>

struct Coordinate {
    int x;
    int y;

    auto operator<=>(const Coordinate&) const = default;
};
```

In this class, `x` is compared first, followed by `y`.

Consequently, the point `(2, 100)` is considered less than `(3, 0)` because the first point has a smaller `x` coordinate.

This behavior differs from comparing geometric distances from the origin. If geometric ordering is required, the comparison logic must be designed manually.

### Comparing Base Classes

When a class inherits from a base class, a defaulted comparison first compares the base-class subobject and then compares the derived class's non-static data members in declaration order.

```
#include <compare>

struct Base {
    int id;

    auto operator<=>(const Base&) const = default;
};

struct Derived : Base {
    int priority;

    auto operator<=>(const Derived&) const = default;
};
```

In this example, ordering is determined first by `Base::id`, followed by `priority`.

This behavior matters when designing class hierarchies because the ordering of a derived class can depend on the comparison semantics of its base class.

### Limitations of Defaulted Comparisons

Defaulted comparisons are not valid for every class. For example, if members or base classes cannot provide the required comparison operations, the defaulted definition may be defined as deleted.

Similarly, members whose comparison semantics are undefined or incompatible with the class's requirements can make defaulted comparison inappropriate.

In such cases, consider whether a manual `<=>` implementation is more suitable or whether the class should have no general ordering at all.

## Differences Between `operator==` and `operator<=>`

### The Role of Equality in Type Design

The `operator==` operator checks whether two values are equal, whereas `operator<=>` determines their ordering relationship.

These concepts are related, but they are not necessarily identical.

For example, in a string-like class that performs case-insensitive comparison, two strings might be equivalent under the ordering while remaining different under exact equality.

When designing an API, define the semantic meaning of equality and determine whether it should agree with equivalence under the ordering.

### An Example of Independent Equality

Suppose a `User` class is ordered by identifier, but equality must consider all of its information.

```
#include <compare>
#include <string>

struct User {
    int id;
    std::string name;

    std::strong_ordering operator<=>(
        const User& other
    ) const {
        return id <=> other.id;
    }

    bool operator==(const User& other) const {
        return id == other.id &&
               name == other.name;
    }
};
```

In this design, two users with the same identifier but different names are equivalent under the ordering, but they are not equal according to `operator==`.

Such a design may be valid for specific applications, but it can cause problems for algorithms and data structures that depend on consistency between ordering and equality.

For example, ordinary sorting may place these two objects at the same ordering position even though they are unequal.

If this distinction is not intentional, equality and ordering should be designed around a consistent semantic model.

## Floating-Point Comparisons and NaN

### The Behavior of `double` Under Three-Way Comparison

An important consideration when designing comparisons is the behavior of floating-point values.

In C++20, three-way comparisons of floating-point types such as `float` and `double` use `std::partial_ordering`, because NaN prevents a total ordering from being established.

```
#include <compare>
#include <limits>

int main() {
    double a = 1.0;
    double b = std::numeric_limits<double>::quiet_NaN();

    auto result = a <=> b;

    bool unordered =
        result == std::partial_ordering::unordered;
}
```

In this example, the result is `unordered`.

Therefore, if your class contains floating-point values, do not assume without examination that its defaulted comparison produces `std::strong_ordering`.

### The Consequences of NaN for Sorting

Comparisons involving NaN can create problems when designing sorting algorithms and data structures that require a strict weak ordering.

For example, ordinary floating-point comparisons in the presence of NaN may fail to provide the ordering relationship expected by an algorithm.

If an application requires predictable sorting of values that may contain NaN, define an explicit policy. One option is to place all NaN values at the end of the ordering.

```
#include <compare>
#include <cmath>

std::strong_ordering compareDouble(double lhs, double rhs) {
    const bool lhsNan = std::isnan(lhs);
    const bool rhsNan = std::isnan(rhs);

    if (lhsNan && rhsNan) {
        return std::strong_ordering::equal;
    }

    if (lhsNan) {
        return std::strong_ordering::greater;
    }

    if (rhsNan) {
        return std::strong_ordering::less;
    }

    if (lhs < rhs) {
        return std::strong_ordering::less;
    }

    if (lhs > rhs) {
        return std::strong_ordering::greater;
    }

    return std::strong_ordering::equal;
}
```

This example treats NaN as greater than every non-NaN value and considers all NaN values equivalent under the ordering.

This policy is a design choice. It does not necessarily satisfy every application's requirements, including requirements to distinguish different NaN representations or to order floating-point values by their bit representations.

When special values are involved, the handling of positive and negative zero and other exceptional inputs should also be specified according to the application's requirements.

## Comparison with Standard Library Facilities

### Using `std::compare_three_way`

The standard library introduced the function object `std::compare_three_way` in C++20.

It is a callable object designed to perform three-way comparisons.

```
#include <compare>
#include <iostream>

int main() {
    std::compare_three_way compare;

    auto result = compare(10, 20);

    std::cout << (result < 0) << '\n';
}
```

This utility is useful in generic code and templates that need to accept a three-way comparison operation as a function object.

When designing templates, remember that `std::compare_three_way` can only be used when the argument types satisfy its requirements. Arbitrary user-defined types cannot necessarily be compared without suitable comparison operators.

### `std::compare_three_way` Versus `std::less`

The `std::less` function object performs a binary comparison and returns a `bool`. In contrast, `std::compare_three_way` is designed to produce a three-way comparison result.

```
#include <compare>
#include <functional>

int main() {
    std::less<> less;
    std::compare_three_way compare;

    bool isLess = less(10, 20);
    auto ordering = compare(10, 20);
}
```

The choice between these utilities depends on the application's needs.

If you only need to determine whether one value is less than another, `std::less` is sufficient. If you need a result that represents the ordering relationship, `std::compare_three_way` is more appropriate.

### Understanding `std::strong_order`

The C++20 standard library provides `std::strong_order` for requesting strong ordering for types that support the operation.

This function is not equivalent to `std::compare_three_way`. While the latter uses a type's supported three-way comparison, `std::strong_order` is designed to obtain a strong ordering.

```
#include <compare>

int main() {
    double a = 1.0;
    double b = 2.0;

    auto result = std::strong_order(a, b);
}
```

For floating-point types, strong ordering can handle distinctions involving values such as NaN according to its specified contract. This differs from ordinary `<=>` comparison, which can produce `unordered`.

Before using this utility, check its precise contract and requirements for the type involved. Not every custom type supports strong ordering automatically or without an appropriate design.

## The Role of `<=>` in Concepts and Generic Programming

### Using `std::three_way_comparable`

C++20 defines the `std::three_way_comparable` concept in the `<compare>` header.

This concept specifies whether a type satisfies the requirements expected for three-way comparison.

```
#include <compare>
#include <concepts>

template <typename T>
requires std::three_way_comparable<T>
bool isOrderedBefore(const T& lhs, const T& rhs) {
    return lhs < rhs;
}
```

In this example, the function can be used only with types that satisfy `std::three_way_comparable`.

This concept does not merely check whether an expression involving `<=>` exists. It also considers a set of syntactic and semantic requirements concerning comparison operations.

### `std::three_way_comparable` Versus `std::totally_ordered`

The `std::totally_ordered` concept is used for types that provide the usual comparison operations and satisfy their associated semantic requirements.

In contrast, `std::three_way_comparable` specifically concerns three-way comparison and its relationship with relational operators.

The `std::three_way_comparable_with` concept is also available for checking comparison compatibility between different types.

These concepts are useful when designing template libraries because they allow algorithms and APIs to express their requirements more precisely.

However, satisfying a concept does not merely mean that the relevant expressions compile. Its semantic requirements must also be met.

## Applications in Sorting and Data Structures

### Sorting Custom Types

A common application of `<=>` is designing types that need to be used with sorting algorithms or ordered collections.

```
#include <algorithm>
#include <compare>
#include <vector>

struct Task {
    int priority;
    int id;

    auto operator<=>(const Task&) const = default;
};

int main() {
    std::vector<Task> tasks{
        {3, 10},
        {1, 20},
        {2, 30}
    };

    std::sort(tasks.begin(), tasks.end());
}
```

In this example, ordering is determined first by `priority` and then by `id`.

Because both members are integer types, the defaulted comparison provides an appropriate strong ordering.

If tasks need to be sorted in descending priority order, you can supply a separate comparator to `std::sort` or adjust the ordering design to suit the requirements.

### Using `std::set` and `std::map`

Containers such as `std::set` and `std::map` require a suitable ordering relation. Their comparators determine where elements belong within the ordered structure.

Having `<=>` in a custom type does not mean that every container directly uses its result. These containers typically use comparators such as `std::less`.

If the `<` operator is available for the type through the rewriting rules associated with `<=>`, the type can often be used with the default comparator.

```
#include <compare>
#include <map>

struct Version {
    int major;
    int minor;

    auto operator<=>(const Version&) const = default;
};

int main() {
    std::map<Version, int> versions;

    versions[{1, 0}] = 100;
    versions[{2, 0}] = 200;
}
```

This example demonstrates how a type with a defaulted comparison can be used as a key in an ordered container.

An important consideration is that the comparator used by ordered containers must satisfy the requirements of a strict weak ordering. The presence of `<=>` alone does not guarantee that every comparison relationship involving a type is suitable for every application.

## Comparing `<=>` with Traditional Approaches

### Traditional Versus Modern Implementations

In traditional approaches, developers often had to implement the required comparison operators separately.

```
struct Number {
    int value;

    bool operator==(const Number& other) const {
        return value == other.value;
    }

    bool operator<(const Number& other) const {
        return value < other.value;
    }
};
```

In C++20, the same design can often be simplified.

```
#include <compare>

struct Number {
    int value;

    auto operator<=>(const Number&) const = default;
};
```

In the second version, comparison is based on the `value` member, and the compiler can support the required relational operators through the operator-rewriting rules.

Furthermore, a defaulted `<=>` allows the corresponding `operator==` to be generated implicitly, provided the applicable requirements are satisfied.

However, a manual implementation may still be more appropriate when a class has specialized comparison semantics.

### Manual Multi-Stage Comparison Versus `<=>`

In older approaches, developers often implemented lexicographical comparisons by checking each member separately and moving to the next member whenever the current members were equal.

The `<=>` operator provides a standardized interface for implementing the same logic.

This does not eliminate the need to design comparison semantics carefully. Instead, it makes consistent and less repetitive implementations possible.

## Limitations and Common Mistakes

### Assuming the Result Is a Specific Integer

A common mistake is assuming that `<=>` always returns exactly `-1`, `0`, or `1`.

The standard does not provide that guarantee. Instead, inspect the result by comparing it with zero or with named values of the appropriate comparison type.

### Assuming All Operators Are Generated

Another common mistake is assuming that defining `<=>` automatically generates every comparison operator in all circumstances.

In practice, operator rewriting and implicit generation of `operator==` are governed by specific language rules. If a class requires particular equality semantics, those semantics must be designed explicitly.

### Misunderstanding `auto` Return-Type Deduction

Using `auto` for the return type of `<=>` is appropriate in many cases, especially when the result type depends on the types of the members being compared.

However, if a public API must expose a specific result type, explicitly declaring `std::strong_ordering` or another comparison type may be more appropriate.

For example, if a class contains a floating-point member, automatic deduction may produce `std::partial_ordering`. In such cases, explicitly declaring a stronger result type may not be compatible with the actual comparison behavior.

### Comparing Members That Do Not Represent the Intended Semantics

Defaulted comparisons examine non-static members and base-class subobjects according to the standard rules. This does not necessarily correspond to what should be compared from the application's perspective.

For example, a class containing a stable identifier and cached data may need to compare only the stable identifier, not the cached value.

In such cases, a manual implementation is usually a better choice.

### Ignoring NaN

In classes containing floating-point values, ignoring the `unordered` state can lead to logical errors in comparisons and sorting.

If the type requires a total ordering, the policy for handling NaN must be specified explicitly.

### Confusing Ordering with Equality

Ordering and equality are related but distinct concepts. Two objects that are equivalent under an ordering are not necessarily equal in every semantic respect.

This distinction is particularly important for classes that implement case-insensitive comparison, key-based ordering, or comparison using only a subset of their members.

## Design Considerations for Professional Developers

### Choosing the Appropriate Result Type

The comparison result type should match the semantic contract of the class.

If all values must have a well-defined ordering and equivalence must be consistent with equality, `std::strong_ordering` is appropriate.

If the ordering is definite but multiple values can be equivalent under the comparison, `std::weak_ordering` is suitable.

If some values cannot be compared in an ordering relationship, `std::partial_ordering` is the appropriate choice.

### Designing for Semantics, Not Just Syntax

The fact that an `<=>` operator compiles does not guarantee that its ordering relation is correct.

When designing general-purpose types, consider whether the ordering relation:

- Is compatible with the requirements of the algorithms that consume it.
- Produces predictable behavior.
- Is consistent with `operator==` when required.
- Avoids unintended semantic changes when the class's internal members change.

### Compatibility with Older Language Versions

The `<=>` operator and the primary comparison result types belong to C++20. Code that uses these features requires a compiler with appropriate C++20 support and a corresponding standard library implementation.

If a library must remain compatible with C++17 or earlier versions, these features cannot be used without adapting the design.

For cross-platform projects, also distinguish compiler support for the language standard from standard library support for `<compare>` and its associated facilities.

## Summary

The three-way comparison operator `<=>` is an important C++20 feature for designing comparable types. It provides a standardized way to represent comparison results and, together with operator rewriting and defaulted comparisons, reduces duplicated code.

The three primary comparison types—`std::strong_ordering`, `std::weak_ordering`, and `std::partial_ordering`—are designed for different ordering semantics. The choice between them should be based on the type's semantic contract rather than implementation convenience alone.

For many value types, a defaulted `<=>` is an appropriate choice. However, when the intended ordering differs from the order or meaning of the internal members, a manual implementation is necessary.

Ultimately, using `<=>` correctly involves more than defining an operator. It requires understanding the relationship between ordering and equality, the behavior of special values, the requirements of standard algorithms, and the contracts of public APIs. Following these principles helps developers build types that are more reliable, readable, and compatible with modern C++.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>