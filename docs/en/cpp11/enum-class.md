<div align="center">

[🇺🇸 English](./enum-class.md) | [🇮🇷 فارسی](../../fa/cpp11/enum-class.md)

</div>

---

# Introduction to `enum class` in C++

The `enum class` feature is one of C++'s mechanisms for defining a limited set of named values. It is a type-safe and modern form of the traditional `enum` and has been available since C++11. This document focuses on defining and using `enum class`, as well as its `enum struct` form.

## The Concept of Enumeration

An `enumeration` is useful when a value should only be one of several predefined options. Instead of using raw integers, these options can be represented by meaningful names.

For example, if the state of a request can only be `Pending`, `Running`, or `Completed`, an enumeration can represent this constraint directly in the data model.

```cpp
enum Status {
    Pending,
    Running,
    Completed
};
```

In this example, enumeration values are generally associated with integer values, and traditional enumerations can be implicitly converted to an integer type in certain contexts.

This behavior is one of the main reasons `enum class` was introduced in C++11, because traditional enumerations had limitations in terms of type safety and name collisions.

## Introduction to enum class

An `enum class` is a scoped enumeration with stronger type safety.

```cpp
enum class Status {
    Pending,
    Running,
    Completed
};
```

Values must now be accessed through the enumeration's name:

```cpp
Status status = Status::Pending;
```

This prevents names such as `Pending` and `Completed` from being introduced directly into the surrounding scope.

As a result, `enum class` is generally a better choice for modern C++ code unless the specific behavior of a traditional `enum` is intentionally required.

## Differences Between enum class and Traditional enum

The most important differences between `enum class` and traditional `enum` are scoping and implicit conversions.

With a traditional enumeration:

```cpp
enum Color {
    Red,
    Green,
    Blue
};

Color color = Red;
```

With an `enum class`:

```cpp
enum class Color {
    Red,
    Green,
    Blue
};

Color color = Color::Red;
```

The values of an `enum class` belong to the enumeration's scope. Therefore, `Color::Red` must be written explicitly.

This also prevents name collisions:

```cpp
enum class TrafficLight {
    Red,
    Yellow,
    Green
};

enum class Color {
    Red,
    Green,
    Blue
};
```

Both enumerations can contain a value named `Red` because those names belong to different scopes.

## The Type-Safety Advantage

One of the most important advantages of `enum class` is that it prevents implicit conversion to integer types.

The following is not valid with an `enum class`:

```cpp
enum class Status {
    Pending,
    Completed
};

Status status = Status::Pending;

// int value = status;
```

An explicit conversion is required:

```cpp
int value = static_cast<int>(status);
```

This restriction prevents mistakes where an enumeration is unintentionally used in arithmetic operations, comparisons, or other APIs that expect an integer.

In contrast, a traditional enumeration can be implicitly converted to an integer type in many contexts:

```cpp
enum Status {
    Pending,
    Completed
};

Status status = Pending;

int value = status;
```

Therefore, `enum class` is generally a safer choice for modeling domain values.

## The Difference in Scoping

The values of an `enum class` belong to the scope of the enumeration itself.

```cpp
enum class Direction {
    North,
    South,
    East,
    West
};

Direction direction = Direction::North;
```

Writing `North` by itself is not valid:

```cpp
// Direction direction = North;
```

This scoping is particularly important in large projects because many enumerations can contain similarly named values without causing collisions.

## Specifying the Underlying Type

By default, the implementation chooses the underlying type of an enumeration. When necessary, the underlying type can be specified explicitly.

```cpp
enum class Status : std::uint8_t {
    Pending,
    Running,
    Completed
};
```

This generally requires `<cstdint>`:

```cpp
#include <cstdint>

enum class Status : std::uint8_t {
    Pending,
    Running,
    Completed
};
```

Specifying the underlying type is useful when the size of the data, memory layout, serialization, or compatibility with a particular interface matters.

However, explicitly choosing `std::uint8_t` solely to make a variable smaller is not always necessary. In ordinary application code, the underlying type should generally only be specified when there is a real requirement for it.

## Explicit Enumerator Values

Specific values can be assigned to enumeration members.

```cpp
enum class ErrorCode {
    None = 0,
    NotFound = 404,
    PermissionDenied = 403,
    InternalError = 500
};
```

Each enumerator then has its explicitly specified value.

Subsequent values can also continue from the previous value:

```cpp
enum class Priority {
    Low = 1,
    Medium,
    High
};
```

In this example, `Medium` has the value `2` and `High` has the value `3`.

Completely independent values can also be defined:

```cpp
enum class HttpStatus {
    Ok = 200,
    BadRequest = 400,
    NotFound = 404,
    InternalServerError = 500
};
```

## What Is enum struct?

The `enum struct` form also exists in C++ and has the same semantics as `enum class`.

```cpp
enum struct Status {
    Pending,
    Running,
    Completed
};
```

This is semantically equivalent to:

```cpp
enum class Status {
    Pending,
    Running,
    Completed
};
```

Both are scoped enumerations and provide the same type-safety rules.

Therefore, `enum struct` can mostly be considered an alternative syntax for `enum class`.

In most projects, `enum class` is more common and is generally more recognizable to C++ developers. However, choosing between the two does not change their semantics.

## Using enum class with switch

A common use of `enum class` is with `switch`.

```cpp
enum class Status {
    Pending,
    Running,
    Completed,
    Failed
};

void handleStatus(Status status) {
    switch (status) {
        case Status::Pending:
            break;

        case Status::Running:
            break;

        case Status::Completed:
            break;

        case Status::Failed:
            break;
    }
}
```

Using `enum class` in such cases makes the possible options explicit.

When a new enumerator is added, appropriate compiler warnings can help identify `switch` statements that do not handle the new case. For projects where exhaustiveness is important, configuring compiler warnings appropriately is highly useful.

## Default Values of enum class

When an `enum class` variable is default-initialized, its value is not necessarily one of the defined enumerators.

```cpp
enum class Status {
    Pending,
    Completed
};

Status status;
```

For an automatic local variable, `status` does not have a usable, determined value in this situation.

When a valid initial value matters, it is better to initialize the variable explicitly:

```cpp
Status status = Status::Pending;
```

It should also be noted that, according to the language rules, an enumeration can represent values for which no named enumerator exists. Therefore, having an `enum class` type does not necessarily mean that every possible value is a valid, named enumerator.

## Values Outside the Defined Enumerators

An `enum class` value can also be created through an explicit conversion from a value that does not correspond to any defined enumerator.

```cpp
enum class Status {
    Pending = 0,
    Completed = 1
};

Status status = static_cast<Status>(42);
```

Here, `status` has an enumeration type, but `42` does not correspond to a named enumerator.

This matters when working with external data, serialization, files, or network protocols.

When data comes from outside the program, it should not be assumed that simply converting it to an `enum class` makes the value valid.

## Validating Values

When an enumeration value comes from an external source, it is better to validate it first.

For example:

```cpp
enum class Status : std::uint8_t {
    Pending = 0,
    Running = 1,
    Completed = 2
};

bool isValidStatus(std::uint8_t value) {
    switch (value) {
        case 0:
        case 1:
        case 2:
            return true;
        default:
            return false;
    }
}
```

Only valid values can then be converted to `Status`.

```cpp
std::uint8_t rawValue = 1;

if (isValidStatus(rawValue)) {
    Status status = static_cast<Status>(rawValue);
}
```

The validation strategy should match the actual domain, especially when an enumeration uses sparse values.

## Comparing enum class Values

Two values of the same `enum class` type can be compared directly.

```cpp
enum class Status {
    Pending,
    Completed
};

Status status = Status::Pending;

if (status == Status::Pending) {
    // Handle pending status
}
```

However, it cannot be directly compared with an integer:

```cpp
// if (status == 0) {
// }
```

If necessary, an explicit conversion can be used:

```cpp
if (static_cast<int>(status) == 0) {
    // Handle zero
}
```

In domain-level code, however, it is generally better to compare the enumeration with its named enumerator rather than with its numeric value.

## Using enum class in APIs

Using `enum class` in function interfaces makes API contracts clearer.

```cpp
enum class LogLevel {
    Debug,
    Info,
    Warning,
    Error
};

void setLogLevel(LogLevel level);
```

The call site is also self-explanatory:

```cpp
setLogLevel(LogLevel::Warning);
```

By contrast, an API such as the following communicates less information about the permitted values:

```cpp
void setLogLevel(int level);
```

Using `enum class` allows the compiler to enforce part of the API contract as well.

## enum class and Overloads

Because each `enum class` is a distinct type, overloads can be defined for different enumerations.

```cpp
enum class Color {
    Red,
    Blue
};

enum class Direction {
    North,
    South
};

void process(Color value);
void process(Direction value);
```

This provides stronger type safety than using several equivalent `int` parameters.

## enum class and Bit Flags

One advanced use of enumerations is representing a set of flags.

For example:

```cpp
enum class Permission : unsigned int {
    None    = 0,
    Read    = 1 << 0,
    Write   = 1 << 1,
    Execute = 1 << 2
};
```

However, unlike traditional enumerations, bitwise operators are not automatically provided for `enum class`.

Therefore, the following is not available by default:

```cpp
// auto permissions = Permission::Read | Permission::Write;
```

If such an API is actually required, the appropriate operators can be defined in a controlled way:

```cpp
#include <type_traits>

enum class Permission : unsigned int {
    None    = 0,
    Read    = 1 << 0,
    Write   = 1 << 1,
    Execute = 1 << 2
};

constexpr Permission operator|(Permission lhs, Permission rhs) {
    using Underlying = std::underlying_type_t<Permission>;

    return static_cast<Permission>(
        static_cast<Underlying>(lhs) |
        static_cast<Underlying>(rhs)
    );
}
```

The flags can then be combined:

```cpp
auto permissions = Permission::Read | Permission::Write;
```

In large projects, such operators should only be defined for enumerations that are genuinely designed to represent bitmasks. An `enum class` should not be treated as a bitmask merely because it has an underlying integer type.

## Designing enum class as a Bitmask

When an enumeration is designed as a bitmask, this intention should be clear in the type's design.

```cpp
enum class FileAccess : unsigned int {
    None    = 0,
    Read    = 1u << 0,
    Write   = 1u << 1,
    Execute = 1u << 2
};
```

In this model, each independent value should generally correspond to a separate bit.

Combined values can also be defined as enumerators:

```cpp
enum class FileAccess : unsigned int {
    None       = 0,
    Read       = 1u << 0,
    Write      = 1u << 1,
    Execute    = 1u << 2,
    ReadWrite  = Read | Write
};
```

However, this form requires care because the initializer expressions are subject to the type-safety rules of the enumeration. In more complex designs, it is better to define the required bitwise operators and helper functions explicitly.

## Using the Underlying Type

The underlying type of an enumeration can be specified using integer types:

```cpp
enum class State : std::uint8_t {
    Idle,
    Running,
    Stopped
};
```

This can be useful for serialization or data structures with a specified layout.

However, the underlying type should not be treated as a way to bypass type safety.

The underlying type can be obtained with `std::underlying_type_t`:

```cpp
#include <type_traits>

enum class Status : std::uint8_t {
    Pending,
    Completed
};

using StatusValue = std::underlying_type_t<Status>;
```

In this example, `StatusValue` is the underlying type of `Status`.

## Converting enum class to an Integer

An explicit conversion can be performed with `static_cast`:

```cpp
enum class Status {
    Pending = 10,
    Completed = 20
};

Status status = Status::Completed;

int value = static_cast<int>(status);
```

This conversion should be intentional because converting an enumeration to an integer removes part of the type abstraction.

If the purpose is serialization, it is generally better to perform this conversion in a dedicated layer rather than scattering it throughout the application.

## Converting an Integer to enum class

The reverse conversion is also possible with `static_cast`:

```cpp
int value = 20;

Status status = static_cast<Status>(value);
```

However, this conversion does not validate the value.

Therefore, if `value` comes from a user, file, network, or external system, it should be validated before use.

## enum class and constexpr

Enumeration values can also be used in compile-time contexts.

```cpp
enum class Status {
    Pending,
    Running,
    Completed
};

constexpr Status defaultStatus = Status::Pending;
```

They can also be used in many contexts that require constant expressions:

```cpp
enum class BufferSize {
    Small = 128,
    Large = 1024
};

std::array<char, static_cast<std::size_t>(BufferSize::Small)> buffer{};
```

Notice that an explicit conversion is used to obtain the numeric size.

## enum class and Templates

Because `enum class` defines a real type, it can be used with templates.

```cpp
enum class Color {
    Red,
    Green,
    Blue
};

template <Color Value>
struct ColorTag {
    static constexpr Color value = Value;
};

using RedTag = ColorTag<Color::Red>;
```

This can be useful in compile-time design, policy-based design, and metaprogramming.

## enum class and Specialization

Enumerations can be used as template arguments, allowing different specializations for different enumerator values.

```cpp
enum class Operation {
    Read,
    Write
};

template <Operation>
struct OperationTraits;

template <>
struct OperationTraits<Operation::Read> {
    static constexpr bool modifiesData = false;
};

template <>
struct OperationTraits<Operation::Write> {
    static constexpr bool modifiesData = true;
};
```

In this pattern, `enum class` becomes part of the type-level design.

## enum class and class

Although `enum class` contains the keyword `class`, it should not be confused with an ordinary `class`.

```cpp
enum class Status {
    Pending,
    Completed
};
```

This declaration defines an enumeration, not a class containing member functions and data members.

The `class` keyword in this syntax primarily indicates that the enumeration is scoped.

The same characteristic is present in:

```cpp
enum struct Status {
    Pending,
    Completed
};
```

## enum class and enum struct

The following two definitions are semantically equivalent:

```cpp
enum class Status {
    Pending,
    Completed
};
```

and:

```cpp
enum struct Status {
    Pending,
    Completed
};
```

Both are:

* scoped;
* type-safe;
* not implicitly convertible to integers;
* scoped with respect to their enumerators;
* capable of specifying an underlying type explicitly.

The main difference is syntax and convention.

For consistency within a codebase, a team should generally adopt a clear convention. In most C++ projects, `enum class` is the more common choice.

## enum class and Forward Declarations

When the underlying type is specified, an enumeration can be forward-declared.

```cpp
enum class Status : std::uint8_t;
```

It can then be defined later:

```cpp
enum class Status : std::uint8_t {
    Pending,
    Completed
};
```

Specifying the underlying type is important in the forward declaration.

This technique can be useful in large headers for reducing coupling.

## enum class in Headers

When an enumeration is part of a public API, its definition is generally placed in a header.

```cpp
#pragma once

enum class ConnectionState {
    Disconnected,
    Connecting,
    Connected
};
```

Other parts of the program can then use the same type.

For types shared between multiple components, appropriate naming and scoping are important. Placing an enumeration in an appropriate namespace also helps prevent collisions.

## Using a Namespace

An enumeration can be placed inside a namespace:

```cpp
namespace network {

enum class ConnectionState {
    Disconnected,
    Connecting,
    Connected
};

}
```

It can then be used as follows:

```cpp
network::ConnectionState state =
    network::ConnectionState::Connected;
```

Combining namespaces with scoped enumerations is an effective way to prevent namespace pollution in large projects.

## enum class Naming

Enumeration names should represent their domain.

For example:

```cpp
enum class ConnectionState {
    Disconnected,
    Connecting,
    Connected
};
```

This is generally better than overly generic names such as:

```cpp
// Avoid overly generic names.
enum class State {
    Start,
    Stop
};
```

Of course, the appropriate name depends on the actual domain and scope of the project.

With `enum class`, there is no need to use names such as `StateConnected` or `StateDisconnected` merely to avoid collisions because enumerators are scoped.

## Choosing enum class or Traditional enum

In modern C++ code, `enum class` is generally the better default choice.

Using `enum class` is usually recommended when:

* type safety matters;
* the values model a specific domain;
* enumerators should not be introduced into the surrounding scope;
* implicit conversion to integers is undesirable;
* the API should be self-documenting.

A traditional `enum` may still be required in legacy code, existing APIs, or situations where implicit conversions are intentionally part of the design.

However, a traditional `enum` should not be chosen merely because its syntax is shorter.

## Limitations of enum class

Despite its advantages, `enum class` does not solve every design problem.

One limitation is that an enumeration does not automatically provide a useful string representation.

For example, from:

```cpp
enum class Status {
    Pending,
    Completed
};
```

you cannot expect `Status::Pending` to automatically become `"Pending"` through a standard language mechanism.

If logging, UI, or serialization requires textual names, an appropriate mapping must be designed.

```cpp
std::string_view toString(Status status) {
    switch (status) {
        case Status::Pending:
            return "Pending";
        case Status::Completed:
            return "Completed";
    }

    return "Unknown";
}
```

This approach also allows behavior for invalid values to be handled explicitly.

## Designing enum class to String Conversion

A conversion function can provide a suitable API for logging or serialization:

```cpp
#include <string_view>

enum class Status {
    Pending,
    Running,
    Completed
};

constexpr std::string_view toString(Status status) {
    switch (status) {
        case Status::Pending:
            return "Pending";
        case Status::Running:
            return "Running";
        case Status::Completed:
            return "Completed";
    }

    return "Unknown";
}
```

Keeping this mapping in one defined location is preferable to scattering similar `switch` statements throughout the codebase.

## Common Mistakes

A common mistake is attempting to use an enumerator without its scope:

```cpp
enum class Color {
    Red,
    Blue
};

// Color color = Red;
```

The correct form is:

```cpp
Color color = Color::Red;
```

Another common mistake is expecting implicit conversion to an integer:

```cpp
enum class Status {
    Pending,
    Completed
};

// int value = Status::Pending;
```

An explicit conversion is required when necessary:

```cpp
int value = static_cast<int>(Status::Pending);
```

Another common issue occurs during serialization, when code assumes that every integer is valid after conversion:

```cpp
int value = readValue();
Status status = static_cast<Status>(value);
```

This code does not perform validation by itself.

## Important Considerations for switch

When using `switch` with an `enum class`, the possible states should be considered deliberately.

```cpp
switch (status) {
    case Status::Pending:
        break;

    case Status::Running:
        break;

    case Status::Completed:
        break;
}
```

Appropriate compiler warnings can help identify missing cases.

However, compiler warnings should not be treated as runtime validation, because an enumeration value may originate from external data or from an invalid cast.

## enum class and External Data

One of the most important design considerations arises when an enumeration value comes from outside the program.

For example, if a protocol defines the values `0`, `1`, and `2` for different states, the boundary between the external representation and the internal type should be clearly defined.

```cpp
enum class Status : std::uint8_t {
    Pending = 0,
    Running = 1,
    Completed = 2
};
```

The parsing layer can validate the raw value and then convert it to the internal type.

This separation keeps the internal parts of the application less dependent on the external representation.

## enum class and Serialization

Specifying an underlying type and explicit values can be useful for stable protocols:

```cpp
enum class MessageType : std::uint8_t {
    Request = 1,
    Response = 2,
    Error = 3
};
```

However, specifying the underlying type alone does not guarantee portable serialization.

Binary protocols may also require considerations such as endianness, exact data size, versioning, and validation.

Therefore, `enum class` is only one part of a serialization design.

## enum class and ABI

When an enumeration is used in a public ABI, changing its underlying type or values can have compatibility implications.

For example, if an enumeration value is part of a binary protocol or stable interface, changing those values may break existing programs.

In such situations, the underlying type and important values should be designed deliberately, and changes should follow the project's versioning and compatibility policy.

## enum class and Memory

The size of an enumeration object depends on its underlying type and implementation unless the underlying type is explicitly specified.

If the exact size matters for layout or serialization, specify it explicitly:

```cpp
enum class PacketType : std::uint8_t {
    Data,
    Control,
    Error
};
```

In ordinary code, however, the underlying type should not be made artificially small without a real requirement.

Choosing a very small type may also result in integral promotions in some contexts and does not necessarily make every operation simpler or faster.

## enum class and Custom Operators

Custom operators can be defined for `enum class`, but they should match the actual semantics of the type.

For example, if a type genuinely represents a set of flags, defining `operator|` makes sense.

However, defining arithmetic operators such as `+` for a status enumeration is usually not an appropriate design:

```cpp
enum class Status {
    Pending,
    Running,
    Completed
};
```

This type represents a domain value rather than a number intended for arithmetic.

Therefore, type safety is not merely a compiler feature; it is also part of the semantic design of the type.

## Choosing an Appropriate Underlying Type

Several scenarios can be considered when choosing an underlying type.

In application code, the compiler can generally be allowed to choose it:

```cpp
enum class Status {
    Pending,
    Running,
    Completed
};
```

In protocols or structures with a specified layout, explicitly choosing the type can be appropriate:

```cpp
enum class Status : std::uint8_t {
    Pending = 0,
    Running = 1,
    Completed = 2
};
```

In either case, the purpose should be clear. Specifying an underlying type without a concrete requirement can introduce unnecessary complexity.

## enum class for API Design

For a new API, this design:

```cpp
enum class Mode {
    Read,
    Write
};

void open(Mode mode);
```

is generally safer than:

```cpp
enum Mode {
    Read,
    Write
};

void open(Mode mode);
```

The caller must use `Mode::Read` and `Mode::Write`, and the compiler can prevent many unintended conversions.

The explicit scope also makes the API more readable:

```cpp
open(Mode::Read);
```

This expression communicates more information about the type of the value.

## enum class in Domain Design

One of the best uses of `enum class` is modeling states and other finite domain categories.

```cpp
enum class OrderState {
    Created,
    Paid,
    Shipped,
    Delivered,
    Cancelled
};
```

The type system now prevents arbitrary `int` values from being implicitly used where an `OrderState` is expected.

This makes the domain model more precise and allows many common errors to be detected earlier.

## When enum class Is Not the Right Choice

If a domain is actually an open-ended set of values, an enumeration may not be the appropriate abstraction.

For example, if an application works with arbitrary numeric identifiers:

```cpp
using UserId = std::uint64_t;
```

using an enumeration for a user ID would be inappropriate because the set of possible values is not fixed and limited.

Likewise, if values require complex behavior, additional state, or multiple invariants, a domain-specific `class` or `struct` may be a better design.

## Best Practices

For modern C++ design, several practical guidelines are useful:

* Prefer `enum class` over `enum` for new enumerations in most cases.
* Choose enumeration names that accurately represent the domain.
* Create domain-specific types instead of using overly generic names.
* Specify the underlying type only when there is a concrete reason to do so.
* Design values deliberately when dealing with protocols and serialization.
* Validate external data before converting it to an enumeration.
* Clearly define the semantics of a type intended to be used as a bitmask.
* Add custom operators only when they match the semantics of the type.
* Avoid unnecessary conversions to integers.
* Keep string and serialization mappings in a clearly defined layer.
* Enable appropriate compiler warnings so that incomplete `switch` statements can be identified more easily.

## Summary

The `enum class` feature introduced in C++11 provides a type-safe and scoped way to define a limited set of named values. Unlike a traditional `enum`, its enumerators do not enter the surrounding scope, and it does not implicitly convert to an integer type.

The `enum struct` form is semantically equivalent to `enum class` and is primarily an alternative syntax.

For modern designs, `enum class` is a good choice for states, modes, categories, error codes, and other finite domain values. At the same time, when using it for serialization, protocols, bit flags, or external data, the underlying type, validation, and compatibility should be designed deliberately.

Ultimately, the main value of `enum class` is not simply naming a few values. It creates a distinct and constrained type that transfers part of the domain contract into the language's type system, making many classes of errors detectable before the program runs.


---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>