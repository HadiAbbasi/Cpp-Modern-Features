<div align="right">

[🇺🇸 English](./[[no_unique_address]].md) | [🇮🇷 فارسی](../../fa/cpp20/[[no_unique_address]].md)

</div>

---

# Understanding `[[no_unique_address]]` in C++20

## Introduction

The `[[no_unique_address]]` attribute is a C++20 language feature designed to optimize object layout. It allows the implementation to place a non-static data member in memory without requiring that member to have a unique address relative to other objects, enabling more efficient memory layouts.

This feature is particularly useful when a class contains members with very little or no data, especially empty objects. Such objects can increase a class's size because of object identity requirements, alignment, and member layout rules, even when they do not store any meaningful state.

Examples include classes that use `std::allocator`, template policies, comparison function objects, behavioral customization objects, and similar types.

The key point is that `[[no_unique_address]]` does not reduce a member's size to zero. Instead, it relaxes the requirement that the member have a unique address, allowing the implementation to make better use of available storage.

## Understanding Unique Address Requirements in C++

### The Behavior of Empty Objects

In C++, a class type with no non-static data members is generally an empty class. However, a complete object of an empty class type cannot have zero size.

```
#include <iostream>

struct Empty {};

int main() {
    std::cout << sizeof(Empty) << '\n';
}
```

On typical implementations, the output is `1`.

This restriction follows from the language's object-size and identity rules. Every complete object must have a nonzero size so that objects of the same type in an array can occupy distinct addresses.

For example, two independent `Empty` objects cannot occupy the same address because the language must preserve the ability to distinguish independent objects of the same type.

### The Problem with Empty Class Members

Consider a class that contains an integer and a member of an empty type:

```
struct Empty {};

struct Regular {
    int value;
    Empty tag;
};
```

The exact size of `Regular` depends on the implementation, alignment requirements, and member layout. On a typical implementation, its size may be `8` bytes:

- The `value` member occupies four bytes.
- The `tag` member typically requires at least one byte.
- The remaining space may be used for padding.

These values are not guaranteed by the standard; the size of `int` and layout details are implementation-dependent.

If `tag` exists only to represent a policy or behavior in the class design, allocating separate storage for it may be unnecessary.

The `[[no_unique_address]]` attribute was introduced to reduce this kind of overhead.

## Introducing the `[[no_unique_address]]` Attribute

### Definition and Syntax

The `[[no_unique_address]]` attribute is a standard language attribute introduced in C++20.

It is applied to a non-static data member declaration:

```
struct Empty {};

struct Optimized {
    int value;
    [[no_unique_address]] Empty tag;
};
```

In this example, the compiler is permitted to place `tag` so that its storage overlaps with storage occupied by `value` or other parts of the object layout, where the language rules allow it.

As a result, the size of `Optimized` may be as small as `sizeof(int)`, although this is not guaranteed.

### What the Attribute Actually Means

This attribute permits the implementation to disregard the usual unique-address requirement for the designated member, subject to the language's rules.

More specifically, the implementation may:

- Overlap the storage of an empty member with storage occupied by another member.
- Make use of padding in the class layout.
- Overlap certain non-empty members with storage belonging to other parts of the object, where permitted.
- Reduce the final class size without changing the logical meaning of its members.

However, the attribute does not permit the compiler to arbitrarily eliminate an object's identity, change its lifetime, or invalidate otherwise legal access to it.

### The Difference Between Storage Overlap and Member Elimination

Using `[[no_unique_address]]` does not remove a member from the class.

```
struct Empty {};

struct Example {
    int value;
    [[no_unique_address]] Empty tag;
};
```

The `tag` member remains a real data member. It can be initialized, accessed, and used in operations on the object.

```
Example object{42, Empty{}};

object.tag = Empty{};
```

This code is valid regardless of whether the compiler allocates separate storage for `tag`.

Therefore, it is important to distinguish between the logical existence of a member and the physical storage allocated to it.

## Examining Memory Layout with Practical Examples

### Comparing Ordinary and Optimized Layouts

The following example compares two classes with the same structure:

```
#include <iostream>

struct Empty {};

struct WithoutOptimization {
    int value;
    Empty tag;
};

struct WithOptimization {
    int value;
    [[no_unique_address]] Empty tag;
};

int main() {
    std::cout << sizeof(WithoutOptimization) << '\n';
    std::cout << sizeof(WithOptimization) << '\n';
}
```

On many typical implementations, the output may look like this:

```
8
4
```

These values are illustrative examples of possible implementation behavior. They are not guaranteed across all compilers or ABIs.

The C++ standard specifies the required semantics, but in many cases it leaves the precise layout of class members and the amount of padding to the implementation.

### Inspecting Size and Alignment

To examine the result in a real environment, use `sizeof` and `alignof`:

```
#include <iostream>

struct Empty {};

struct Example {
    char prefix;
    [[no_unique_address]] Empty tag;
    int value;
};

int main() {
    std::cout << "sizeof(Example): " << sizeof(Example) << '\n';
    std::cout << "alignof(Example): " << alignof(Example) << '\n';
}
```

In this example, member alignment and potential padding can affect the result.

Importantly, `[[no_unique_address]]` does not directly reduce the value of `alignof`. The member type's alignment requirements remain part of the layout rules; the attribute only relaxes the unique-address requirement for the designated member.

## Understanding Empty Objects and Object Identity

### The Difference Between Complete Objects and Subobjects

One of the most important concepts for understanding this attribute is the distinction between a complete object and a subobject.

A complete object of an empty class type cannot have zero size. In contrast, an empty-class subobject may overlap with other parts of the layout under the applicable rules.

For example:

```
struct Empty {};

struct Container {
    [[no_unique_address]] Empty first;
    [[no_unique_address]] Empty second;
};
```

In this structure, the compiler may use the same storage for both members because they have the same type and the attribute relaxes the unique-address requirement for each of them.

However, such overlap is not mandatory, and the exact size of `Container` should not be calculated merely from the number of members.

### Preserving the Identity of Members of the Same Type

Using `[[no_unique_address]]` does not make two independent objects identical.

```
struct Empty {};

struct Pair {
    [[no_unique_address]] Empty first;
    [[no_unique_address]] Empty second;
};
```

In this class, `first` and `second` remain two independent members. However, within the language rules, the compiler may choose their addresses to be either the same or different.

Consequently, code should not assume a fixed relationship between their addresses.

In software design, whenever member identity or address is part of a program's contract, this behavior requires careful consideration.

## Important Applications in Class Design

### Optimizing Policy-Based Classes

One of the most common applications of this feature is Policy-Based Design.

In this pattern, a class's behavior is configurable through types such as comparison function objects, allocators, resource-management policies, or validation policies.

Many of these types can be empty because they define behavior without storing data.

```
#include <functional>
#include <memory>

template <typename T,
          typename Allocator = std::allocator<T>,
          typename Compare = std::less<T>>
struct Storage {
    [[no_unique_address]] Allocator allocator;
    [[no_unique_address]] Compare compare;

    T value;
};
```

In this design, if `Allocator` and `Compare` are suitable types and the implementation can exploit storage overlap, the memory overhead of these members can be reduced.

However, an empty type does not guarantee that the final class will be smaller. For example, a type may contain data members, or other layout constraints may prevent the implementation from taking full advantage of the attribute.

### Reducing Wrapper Overhead

Wrapper classes often store a primary value alongside additional information.

For example, a wrapper may hold a value and a conversion or validation policy:

```
template <typename T, typename Validator>
struct ValidatedValue {
    T value;
    [[no_unique_address]] Validator validator;
};
```

If `Validator` is an empty type, the implementation may be able to reuse available storage in the class layout.

This technique can be useful in general-purpose libraries, small value types, and classes instantiated in large numbers.

### Optimizing Function Objects

Function objects are commonly used with algorithms and data structures. A function object may provide specialized behavior without storing any data.

```
struct Increment {
    int operator()(int value) const {
        return value + 1;
    }
};

template <typename Function>
struct FunctionHolder {
    [[no_unique_address]] Function function;
};
```

In this example, `Increment` is an empty type. Using the attribute allows the implementation to reduce the storage allocated to `function`.

This approach is particularly valuable in more complex designs that store several policies or function objects simultaneously.

## Relationship to Empty Base Optimization

### Introducing Empty Base Optimization

Before C++20, a common technique for reducing the overhead of empty objects was to use private inheritance or the Empty Base Optimization pattern, commonly known as EBO.

```
struct Empty {};

struct BaseOptimized : private Empty {
    int value;
};
```

On typical implementations, the empty base subobject may occupy no additional storage, allowing the class size to be approximately equal to the size of `value`.

However, the exact result still depends on the standard's rules and the implementation.

### Comparing EBO with `[[no_unique_address]]`

EBO uses inheritance to exploit the properties of empty types, whereas `[[no_unique_address]]` is applied directly to a data member.

| Feature                                     | EBO                              | `[[no_unique_address]]`           |
| ------------------------------------------- | -------------------------------- | --------------------------------- |
| Mechanism                                   | Inheritance                      | Data member                       |
| Requires changing the inheritance structure | Yes                              | No                                |
| Readability with composition                | Sometimes less straightforward   | Usually better                    |
| Applicability to non-empty members          | Limited by inheritance rules     | Allows overlap in permitted cases |
| Language support                            | Available in older C++ standards | Introduced in C++20               |

The main advantage of `[[no_unique_address]]` is that it allows a class to use composition without converting a member into a base class, while still benefiting from some of the same storage optimizations.

### EBO Limitations and Subtle Differences

With EBO, the base type and inheritance structure affect the result. There are also restrictions on overlapping base subobjects of the same type.

For example, object identity rules do not permit two independent subobjects of the same type to occupy the same address arbitrarily in every situation.

The `[[no_unique_address]]` attribute does not eliminate all these restrictions; it provides another mechanism for optimizing object layout.

In modern projects, the choice between these approaches should be based on the design model, compatibility requirements, and actual layout measurements rather than general assumptions about object size.

## Relationship to `std::tuple` and `std::pair`

### Optimizing Composite Types

Types such as `std::tuple` and `std::pair` store multiple values of potentially different types. If some of those types are empty, reducing storage overhead can be beneficial.

For example:

```
#include <tuple>

struct Empty {};

using Data = std::tuple<int, Empty, Empty>;
```

A standard library implementation may use techniques such as EBO or `[[no_unique_address]]` to reduce the size of this structure.

However, the standard does not require all implementations to use a particular technique or produce identical object sizes.

### Library Behavior Versus Language Guarantees

It is important to distinguish between a language feature and a standard library implementation detail.

The `[[no_unique_address]]` attribute is a language feature, but its use in implementations of `std::tuple` or `std::pair` depends on the library's design and ABI constraints.

Therefore, the presence of this attribute in one implementation does not imply that all libraries will exhibit identical memory-layout behavior.

## Examining Non-Empty Members

### The Possibility of Overlapping Non-Empty Members

The name of this feature may suggest that only empty types can benefit from it. That interpretation is not accurate.

```
struct Data {
    int value;
};

struct Container {
    int id;
    [[no_unique_address]] Data data;
};
```

The `data` member is not empty, but the attribute can still permit the implementation to overlap its storage with other parts of the object, provided all language rules are satisfied.

For example, available padding or differences between the storage regions of members may affect the result.

However, this does not mean that two members containing independent values can be arbitrarily placed over the same memory region. The rules governing subobject overlap, size, alignment, and access to stored values still apply.

### Avoiding Assumptions Based on Size

When designing classes, it is better to inspect the actual size in the target environment rather than assume a fixed value.

```
#include <iostream>

struct Data {
    int value;
};

struct Container {
    int id;
    [[no_unique_address]] Data data;
};

int main() {
    std::cout << sizeof(Data) << '\n';
    std::cout << sizeof(Container) << '\n';
}
```

The output provides information about the current implementation, but it cannot be generalized to every architecture and compiler.

## Language Rules and Limitations

### An Attribute Rather Than an Executable Operation

The `[[no_unique_address]]` feature is an attribute, not a function, class, or executable statement.

It does not perform an independent operation at runtime or introduce direct runtime overhead in the form of a function call or algorithm.

Its primary effect is on object layout and the way the implementation allocates storage.

### Dependence on the C++ Standard Version

This attribute became part of the C++ language standard in C++20.

Using it requires a compiler and build configuration that support C++20.

If a compiler provides this feature as an extension in an older language mode, its behavior may be specific to that compiler. Support in an older mode should not automatically be treated as standard-conforming C++20 support.

### Differences in Compiler Support

Actual support depends on the compiler version, target platform, and language-standard settings.

For cross-platform projects, test the code with the primary compilers and all relevant target configurations. Consulting the documentation for the specific compiler version and using small tests involving `sizeof` and `alignof` can help identify implementation differences.

### No Guarantee of Reduced Object Size

The most important practical limitation is that using `[[no_unique_address]]` does not guarantee a smaller class.

The compiler is merely permitted to use a more efficient layout. Its final decision depends on the language rules and implementation.

Therefore, if reducing memory consumption is a critical requirement, the result must be measured and verified on the target environments.

## Important Considerations About Member Addresses

### Avoiding Dependencies on Member Addresses

When `[[no_unique_address]]` is used, code should not assume that the address of the designated member is always distinct from the addresses of other members.

```
struct Empty {};

struct Example {
    int value;
    [[no_unique_address]] Empty tag;
};
```

In this structure, comparing the address of `tag` with the addresses of other parts of the object may produce results that differ from simple expectations based on conventional member layout.

Consequently, do not use member addresses to infer the actual object size, determine logical identity, or establish ownership relationships unless the relevant type and layout rules have been carefully considered.

### The Difference Between Overlapping Storage and Aliasing

Overlapping storage does not make every kind of memory access valid.

Aliasing rules, object lifetimes, the actual object types, and the requirements for accessing stored values remain in force.

This attribute is not a general-purpose license to reinterpret memory through unrelated types or bypass the language's rules.

Therefore, it should not be considered a replacement for facilities such as `std::bit_cast`, introduced in C++20, or other standard techniques for handling object representations.

### Impact on ABI-Dependent Code

In many environments, class layout is part of the ABI. Changing the layout of a public class can affect binary compatibility, data exchange between modules, or the way compiled libraries are consumed.

For this reason, adding `[[no_unique_address]]` to members of classes that cross ABI boundaries requires careful consideration.

Even if a layout change produces a desirable size reduction with one compiler, it may introduce incompatibilities with existing binary consumers.

## Important Considerations for Library Design

### Using the Attribute in Small, Frequently Created Types

This feature is generally most valuable for classes that are instantiated in large numbers and where every byte of storage matters.

For example, nodes in data structures, small wrappers, policy-based types, and types used in custom containers may benefit from this capability.

However, reducing an object's size does not necessarily improve program performance. The actual effect should be measured while taking memory-access patterns into account.

### Considering Project Language-Standard Requirements

Libraries that must support multiple C++ versions may need alternative implementation paths for older and newer language standards.

For example, a library may use `[[no_unique_address]]` in C++20 and rely on alternative techniques, such as EBO, in older language modes.

Such a strategy should be designed around actual project requirements and must not depend on non-portable assumptions about compiler behavior.

### Avoiding Unnecessary Complexity

Applying this attribute to every data member does not necessarily improve a design.

If a type is not empty, the class is already sufficiently small, or its size has no meaningful impact on program behavior, the complexity introduced by layout differences and compatibility considerations may not be worthwhile.

A better approach is to design the class around its logical requirements first, then apply this feature where there is a practical reason to optimize storage.

## Common Mistakes

### Assuming That a Member's Size Becomes Zero

Using `[[no_unique_address]]` does not make `sizeof` for the member's type equal to zero. The type's own size remains governed by the language rules.

### Expecting Identical Results Across Compilers

Member layout, alignment, and padding may differ between implementations. Results from one compiler should not be generalized to other environments without testing.

### Using the Attribute to Change Program Semantics

This feature affects the implementation's freedom to arrange storage. It should not be used to eliminate a member's logical state or introduce new assumptions about object identity.

### Ignoring Binary Compatibility

Changing class layout can affect the ABI. This is especially important for shared libraries and projects that use components compiled with different versions of a library.

### Using the Attribute Without Measurement

If the goal is memory optimization, measuring `sizeof`, `alignof`, and, where necessary, actual memory consumption is essential. The mere presence of the attribute does not guarantee improved performance.

## Practical Guidelines for Choosing When to Use It

The following criteria can help determine whether `[[no_unique_address]]` is appropriate:

| Situation                                                                 | Recommendation                                   |
| ------------------------------------------------------------------------- | ------------------------------------------------ |
| The member has an empty type, and many instances of the class are created | Consider using the attribute                     |
| The class contains multiple policies or function objects                  | There may be a significant benefit               |
| The class is part of a public ABI                                         | Check compatibility before making changes        |
| The exact class size is a cross-platform contract                         | Do not rely on one implementation's result       |
| The data member plays an important role in program semantics              | Preserve the object's design and semantics first |
| The expected size reduction is only theoretical                           | Measure the actual result first                  |

## Summary

The `[[no_unique_address]]` attribute in C++20 is a tool for optimizing data-member layout and reducing storage overhead. It is particularly useful for classes containing empty members, policy-based designs, and small wrapper types.

Its main advantage is that it allows implementations to take advantage of available storage without requiring changes to the inheritance structure. As a result, developers can retain composition while benefiting from some of the same storage optimizations associated with Empty Base Optimization.

However, this attribute does not guarantee a smaller object, does not override object-identity or aliasing rules, and can affect the layout of ABI-sensitive classes.

The professional approach is to design the class around its logical requirements first, use the attribute where reducing storage overhead is worthwhile, and then verify the results through measurements and testing on the target compilers and configurations.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>