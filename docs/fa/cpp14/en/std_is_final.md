<div align="center">

[🇺🇸 English](./std_is_final.md) | [🇮🇷 فارسی](../../fa/cpp14/std_is_final.md)

</div>

---

# A Comprehensive Guide to `std::is_final` in C++14

## Introduction to `std::is_final`

`std::is_final` is a type trait in the C++ Standard Library that determines whether a type is declared with the `final` specifier, preventing further inheritance from that class. It was introduced in C++14.

The `final` specifier itself was introduced in C++11 to prevent a class or `struct` from being used as a base class. However, C++11 did not provide a standard library type trait for checking this property at compile time. C++14 addressed this need by introducing `std::is_final`.

This trait is defined in the `<type_traits>` header and can be used for:

- Checking type properties at compile time.
- Implementing template metaprogramming techniques.
- Enforcing inheritance-related constraints in generic libraries.
- Developing general-purpose tools for analyzing types and class hierarchies.
- Performing compile-time checks to validate design contracts.

An important distinction is that `std::is_final` only reports whether a type is final. It does not itself prevent inheritance or modify the definition of a type.

## Understanding the `final` Specifier

The `final` specifier was introduced in C++11. It can be used to prevent inheritance from a class or to prevent a virtual function from being overridden.

### Preventing Inheritance from a Class

To prevent a class from being used as a base class, place `final` after the class name.

```
class Base {
public:
    virtual void execute() {}
};

class FinalClass final : public Base {
public:
    void execute() override {}
};
```

In this example, `FinalClass` can inherit from `Base`, but no other class can derive from `FinalClass`.

The following definition is invalid and must be rejected by the compiler:

```
class InvalidDerived : public FinalClass {
};
```

### Preventing a Virtual Function from Being Overridden

The `final` specifier is not limited to class declarations. It can also be applied to virtual functions.

```
class Base {
public:
    virtual void execute() {}
};

class Derived : public Base {
public:
    void execute() final {}
};
```

In this example, `Derived` can inherit from `Base`, but classes derived from `Derived` cannot override `execute` again.

```
class FurtherDerived : public Derived {
public:
    void execute() override {}
};
```

The definition of `FurtherDerived` is invalid because the virtual function `execute` was declared `final` in its base class.

Key point: Applying `final` to a virtual function is different from applying it to a class. `std::is_final` checks whether a type is final, not whether any of its virtual functions are final.

## Introduction to `std::is_final`

`std::is_final` is a unary type trait that determines whether a type is a final class type.

Its conceptual interface is:

```
template<class T>
struct is_final;
```

This is a conceptual representation of the standard interface, not an actual implementation of the library type trait.

To use this trait, include the following header:

```
#include <type_traits>
```

The result is available through the `value` member, whose type is `bool`.

```
#include <type_traits>

class OrdinaryClass {
};

class FinalClass final {
};

static_assert(!std::is_final<OrdinaryClass>::value,
              "OrdinaryClass is not final");

static_assert(std::is_final<FinalClass>::value,
              "FinalClass is final");
```

In this example, both checks are performed at compile time. If either condition does not match the corresponding `static_assert`, the program fails to compile.

### Interpreting the Result

The result of `std::is_final<T>::value` can be interpreted as follows:

| Result  | Meaning                                                                      |
| ------- | ---------------------------------------------------------------------------- |
| `true`  | `T` is a final class type.                                                   |
| `false` | `T` is not final or is not a class type covered by the trait's intended use. |

Consequently, a result of `false` does not necessarily mean that a type can be used as a base class. For example, numeric types such as `int` and pointer types also produce `false`, even though they cannot be used as base classes.

## Checking Different Types

To use `std::is_final` effectively, it is important to understand how the trait behaves with different types.

### Checking Ordinary and Final Classes

```
#include <type_traits>

class Base {
};

class Derived : public Base {
};

class Sealed final {
};

static_assert(!std::is_final<Base>::value, "");
static_assert(!std::is_final<Derived>::value, "");
static_assert(std::is_final<Sealed>::value, "");
```

In this example, `Derived` is not considered final merely because it inherits from another class. Whether a type is final depends on its own declaration, not on the status of its base classes or derived classes.

### Checking `struct` Types

In terms of inheritance rules, `struct` and `class` behave identically with respect to the `final` specifier. Their main differences concern the default access levels of members and inheritance.

```
#include <type_traits>

struct OpenStruct {
};

struct ClosedStruct final {
};

static_assert(!std::is_final<OpenStruct>::value, "");
static_assert(std::is_final<ClosedStruct>::value, "");
```

Therefore, `std::is_final` can be used with either form of class definition.

### Checking Non-Class Types

This trait is intended to check whether a type is a final class type. For non-class types, the result is `false`.

```
#include <type_traits>

static_assert(!std::is_final<int>::value, "");
static_assert(!std::is_final<double>::value, "");
static_assert(!std::is_final<int*>::value, "");
static_assert(!std::is_final<int&>::value, "");
static_assert(!std::is_final<void>::value, "");
```

This behavior is useful when implementing generic utilities because applying the trait to different types does not necessarily result in a compilation error.

### Checking `const` and `volatile` Types

The `const` and `volatile` qualifiers do not change whether a class is final. The trait reports the final status of the underlying class type even when these qualifiers are present.

```
#include <type_traits>

class FinalType final {
};

static_assert(std::is_final<FinalType>::value, "");
static_assert(std::is_final<const FinalType>::value, "");
static_assert(std::is_final<volatile FinalType>::value, "");
static_assert(std::is_final<const volatile FinalType>::value, "");
```

This behavior is consistent with the purpose of `std::is_final`, since `const` and `volatile` do not change a class's final status.

## Comparing `std::is_final` with `std::is_base_of`

The standard traits `std::is_final` and `std::is_base_of` serve different purposes and should not be used interchangeably.

`std::is_final<T>` checks whether `T` is final. In contrast, `std::is_base_of<Base, Derived>` checks the inheritance relationship between two types.

```
#include <type_traits>

class Base {
};

class Derived : public Base {
};

class FinalDerived final : public Base {
};

static_assert(std::is_base_of<Base, Derived>::value, "");
static_assert(std::is_base_of<Base, FinalDerived>::value, "");

static_assert(!std::is_final<Derived>::value, "");
static_assert(std::is_final<FinalDerived>::value, "");
```

In this example, both derived classes inherit from `Base`, but only `FinalDerived` is final.

An important consideration is that `std::is_base_of` alone does not guarantee that a particular type can be used as a base class in every context. Access control, ambiguous inheritance, and restrictions imposed by `final` can affect whether an actual inheritance declaration is valid.

## Comparing `std::is_final` with `std::is_abstract`

Another standard trait, `std::is_abstract`, determines whether a class type is abstract.

These two properties are independent:

- A `final` class is not necessarily abstract.
- An abstract class is not necessarily `final`.
- A class can be both abstract and `final`.

The following example illustrates this distinction:

```
#include <type_traits>

class AbstractBase {
public:
    virtual void execute() = 0;
};

class ConcreteFinal final : public AbstractBase {
public:
    void execute() override {}
};

static_assert(std::is_abstract<AbstractBase>::value, "");
static_assert(!std::is_final<AbstractBase>::value, "");

static_assert(!std::is_abstract<ConcreteFinal>::value, "");
static_assert(std::is_final<ConcreteFinal>::value, "");
```

In this example, `AbstractBase` is an abstract, non-final class, while `ConcreteFinal` is a concrete, final class.

## Understanding Standard Support Across C++ Versions

The availability of `std::is_final` differs across C++ standard versions.

| C++ standard | Status                                                                                                                                       |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| C++11        | The `final` specifier was introduced, but `std::is_final` was not available in the Standard Library.                                         |
| C++14        | `std::is_final` was added to `<type_traits>`.                                                                                                |
| C++17        | Inline `constexpr` variable templates and related improvements, including `std::is_final_v`, became available.                               |
| C++20        | Features such as Concepts and `consteval` provided additional ways to express compile-time requirements; `std::is_final` remained available. |
| C++23        | The trait remained available without any major change to its fundamental semantics.                                                          |

For projects that must remain compatible with C++14, `std::is_final<T>::value` is the standard, portable approach.

### Introducing `std::is_final_v` in C++17

Starting with C++17, many type traits gained variable-template helpers with the `_v` suffix. These make trait expressions more readable.

```
#include <type_traits>

class FinalClass final {
};

static_assert(std::is_final_v<FinalClass>, "");
```

In C++14, the form above is not available in the Standard Library. Instead, use:

```
#include <type_traits>

class FinalClass final {
};

static_assert(std::is_final<FinalClass>::value, "");
```

This distinction matters when maintaining libraries that support multiple C++ standard versions.

## Practical Applications in Template Metaprogramming

One important use of `std::is_final` is in template metaprogramming. These techniques allow decisions about types and implementation paths to be made at compile time.

### Selecting an Implementation Based on Whether a Type Is Final

Suppose a general-purpose library has two processing paths for different types. In C++14, `std::enable_if` can enable an overload specifically for final classes.

```
#include <type_traits>
#include <iostream>

template<class T>
typename std::enable_if<std::is_final<T>::value, void>::type
process(const T&) {
    std::cout << "Final type\n";
}

template<class T>
typename std::enable_if<!std::is_final<T>::value, void>::type
process(const T&) {
    std::cout << "Non-final or non-class type\n";
}

class FinalType final {
};

class OrdinaryType {
};

int main() {
    process(FinalType{});
    process(OrdinaryType{});
    process(42);
}
```

In this example, `std::enable_if` enables or disables overloads at compile time. The first function is selected for final types, while the second is selected for other types.

This example is technically valid, but in real-world designs, finality is not always an appropriate criterion for choosing an algorithm. If the behavioral difference depends on a type's capabilities, contract, or supported operations, a dedicated trait or an appropriate interface will usually result in a clearer design.

### Using Compile-Time Assertions

Some libraries may require a particular type to be final as part of their design contract. In such cases, `static_assert` can enforce the requirement.

```
#include <type_traits>

template<class T>
class Registry {
    static_assert(std::is_final<T>::value,
                  "T must be a final class type");

public:
    void registerObject() {
    }
};

class FinalObject final {
};

Registry<FinalObject> registry;
```

In this example, `FinalObject` satisfies the required condition.

However, the diagnostic message is somewhat broad because `std::is_final<T>::value` is also `false` for non-class types. In a public API, separate checks can provide more precise diagnostic messages.

## Limitations of `std::is_final`

Despite its simplicity, `std::is_final` has several limitations that should be considered when designing libraries and generic utilities.

### It Does Not Check Whether Functions Are Final

`std::is_final<T>` checks only whether the type itself is final. It does not determine whether any virtual function within the class is declared `final`.

For example, the following class has a final virtual function, but the class itself is not final:

```
class Base {
public:
    virtual void execute() final {}
};

class Derived : public Base {
};
```

The definition of `Derived` is valid because the restriction applies only to overriding `execute`.

### It Does Not Determine Whether Inheritance Is Valid in Every Context

A result of `false` does not mean that inheritance from the type is permitted. This is particularly important for non-class types and incomplete types.

Furthermore, `std::is_final` does not examine an actual inheritance relationship. It reports the type's final status rather than validating a particular inheritance declaration.

### It Does Not Replace the Language's Inheritance Rules

Even if a trait check returns `false`, that result does not guarantee that every inheritance declaration involving the type is valid. Language rules concerning access control, type completeness, ambiguous base classes, and other inheritance restrictions still apply.

### It Depends on the Type's Definition

Finality is part of a type's definition. If a library declares a type with `final`, using that type as a base class is invalid, regardless of whether the consuming code uses this trait.

Consequently, `std::is_final` is useful for inspecting type properties, but it does not replace sound class-hierarchy design or the language's inheritance rules.

## Important Considerations for Incomplete Types

One important detail when working with type traits is their behavior with incomplete types.

In C++, a class can be introduced using a forward declaration before its full definition is available:

```
class ForwardDeclared;
```

At this point, the compiler does not yet have enough information to perform many checks that depend on the class's complete definition.

The standard imposes restrictions on using `std::is_final` with incomplete types. It should not be assumed that this trait produces a reliable result for every incomplete type. Applying it to an incomplete class type when completing that type could change the result violates the standard's requirements and can result in undefined behavior.

Therefore, if a template design needs to check whether a type is final, it is best to perform the check after the class definition is complete.

The following example illustrates an appropriate usage pattern:

```
#include <type_traits>

class ForwardDeclared;

class FinalType final {
};

static_assert(std::is_final<FinalType>::value, "");
```

In this example, the trait is applied only to the complete type `FinalType`.

The important point is that a forward declaration alone does not guarantee that every type trait associated with a class can be used with that type.

## Design Guidelines and Best Practices

The following principles can help improve the design of APIs and libraries that use `std::is_final`.

- Use the trait to inspect an actual type property: Use `std::is_final` when a class's final status directly affects correctness or implementation selection.
- Avoid unnecessarily coupling logic to `final`: A final class is not inherently faster, safer, or more suitable for a particular operation.
- Consider the project's C++ standard version: For C++14 code, `std::is_final<T>::value` is the portable choice. The `_v` form requires C++17 or later.
- Keep independent properties separate: Use the appropriate trait to check abstractness, inheritance relationships, or other type properties.
- Ensure that the type is complete: Perform checks for class finality after the class definition is complete.
- Provide clear diagnostics: In public libraries, compile-time assertions should describe the expected contract precisely to make debugging easier.
- Avoid assumptions about compilers: Use the standard trait rather than relying on compiler-specific behavior or non-standard extensions.

## Final Summary

`std::is_final` was introduced in C++14 to make it possible to check whether a type is declared `final` at compile time. It is provided in the `<type_traits>` header, and its result is accessible through the `value` member, which has type `bool`.

Its primary applications include template metaprogramming, compile-time validation of design contracts, implementation selection based on type properties, and the development of general-purpose libraries.

However, `std::is_final` checks only whether the type itself is final. It does not guarantee that inheritance is valid in every context, nor does it determine whether virtual functions are final.

For code compatible with C++14, `std::is_final<T>::value` is the standard approach. In C++17 and later, `std::is_final_v<T>` provides a more concise alternative.

---

## 🤝 Contributions

<div align="center">

| GitHub                                      | LinkedIn                                                           | Email                                                  | Site                           | Telegram                               |
| ------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------ | -------------------------------------- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](mailto:hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>