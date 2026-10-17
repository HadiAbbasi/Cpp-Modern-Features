<div align="center">

[🇺🇸 English](./aggregate_classes_with_default_member_initializers.md) | [🇮🇷 فارسی](../../fa/cpp14/aggregate_classes_with_default_member_initializers.md)

</div>

---

# Aggregate Classes with Default Member Initializers in C++14

## Introduction

The feature `Aggregate Classes with Default Member Initializers` concerns the interaction between two important C++ concepts: `Aggregate` and `Default Member Initializers`, which were introduced in C++11. Both concepts are useful independently, but their combination behaves differently across language standards, particularly C++14 and C++17. Understanding these differences is essential when designing simple, maintainable classes.

An `Aggregate` is a type whose members can be initialized using `Aggregate Initialization` without requiring user-defined constructors. The `Default Member Initializers` feature, on the other hand, allows default values for class members to be specified directly where those members are declared.

The important point is that, in C++14, the presence of `Default Member Initializers` prevents a class from qualifying as an `Aggregate`. This restriction changed in C++17, making the distinction between these two versions important when writing portable, standards-compliant code.

## The Concept of Aggregates in C++

In C++, an `Aggregate` is a type that satisfies specific requirements for aggregate initialization. This feature allows developers to initialize an object's members without writing a custom constructor.

For example, consider the following structure:

```
#include <iostream>

struct Point {
    int x;
    int y;
};

int main() {
    Point p{10, 20};

    std::cout << p.x << '\n';
    std::cout << p.y << '\n';
}
```

In this example, `Point` is an `Aggregate`, and the expression `Point p{10, 20};` initializes its members in declaration order.

The main characteristics of this approach are:

- No user-defined constructor is required.
- Members can be initialized directly using a braced initializer list.
- Initialization follows the order of the members eligible for aggregate initialization.
- If some values are omitted from the initializer list, the applicable initialization rules determine how the remaining members are initialized.

Here, an initializer list refers to the list of values in the initialization expression. This concept should not be confused with the standard library class `std::initializer_list`.

## The Concept of Default Member Initializers

The `Default Member Initializers` feature was introduced in C++11. It allows a default initial value to be specified directly in the declaration of each non-`static` data member.

For example:

```
struct Configuration {
    int timeout = 30;
    bool enabled = true;
    double threshold = 0.75;
};
```

In this definition, the members `timeout`, `enabled`, and `threshold` have default member initializers.

Consequently, when an object is created without explicitly providing different initial values for these members, their declared initial values are used:

```
Configuration config;
```

In this case, the members will have values of `30`, `true`, and `0.75`, respectively.

Different values can also be provided when creating an object:

```
Configuration config{60, false, 0.9};
```

In this example, the explicitly supplied values replace the default member initializers.

However, the exact initialization behavior depends on whether the type qualifies as an `Aggregate` under the language standard being used.

## Behavioral Differences Between C++11, C++14, and C++17

To understand this subject correctly, we must examine how the definition of an `Aggregate` differs across language versions.

### Behavior in C++11

Under the C++11 standard, the presence of `Default Member Initializers` prevented a class from qualifying as an `Aggregate`.

Therefore, the following definition did not represent an `Aggregate`:

```
struct Point {
    int x = 0;
    int y = 0;
};
```

This restriction meant that the type could not be initialized using `Aggregate Initialization` in the same way as a simple structure without default member initializers.

### Behavior in C++14

The same restriction applied in C++14.

Therefore, a class containing `Default Member Initializers` did not qualify as an `Aggregate` under this standard.

For example:

```
struct Settings {
    int retries = 3;
    bool verbose = false;
};
```

In C++14, `Settings` is not an `Aggregate`.

Consequently, the following expression is invalid under this standard because no suitable constructor is available:

```
Settings settings{5, true};
```

It is also important not to assume that writing `{}` or omitting initializers automatically removes this restriction. The underlying issue is the definition of an `Aggregate` in the applicable language standard.

### The Important Change in C++17

In C++17, the restriction related to `Default Member Initializers` was removed from the definition of an `Aggregate`.

Consequently, the following class can qualify as an `Aggregate`:

```
struct Settings {
    int retries = 3;
    bool verbose = false;
};
```

The following expression is now valid:

```
Settings settings{5, true};
```

In this case, `5` and `true` are assigned to the members `retries` and `verbose`, respectively.

It is also possible to initialize only some members:

```
Settings settings{5};
```

In this example, `retries` is `5`, and `verbose` is `false`.

Therefore, support for `Aggregate` types with `Default Member Initializers` is a C++17 feature, not a C++14 feature.

## A Practical Example in C++14

Suppose we want to define a structure for storing user information:

```
struct User {
    int id;
    bool active;
};
```

This structure is an `Aggregate` in C++14 and can be initialized as follows:

```
User user{101, true};
```

However, if we want to provide default values for its members, we must account for the standard's restrictions:

```
struct User {
    int id = 0;
    bool active = true;
};
```

In C++14, this definition no longer represents an `Aggregate`.

Consequently, if we want to use direct list initialization with a list of values, we must choose another approach.

## Suitable Approaches for C++14

If a project must remain compatible with C++14, there are several practical approaches. The appropriate choice depends on whether preserving `Aggregate Initialization` is more important than specifying default values directly in member declarations.

### Approach One: Remove Default Member Initializers

If the primary goal is to preserve `Aggregate` status, the default member initializers can be removed:

```
struct User {
    int id;
    bool active;
};

int main() {
    User user{0, true};
}
```

With this approach, the initial values are specified when the object is created.

Its main advantage is simplicity and compatibility with C++14. However, responsibility for providing the initial values is transferred to the code that creates the object.

### Approach Two: Define a Constructor

If default values are important, a constructor can be defined:

```
struct User {
    int id;
    bool active;

    User(int id = 0, bool active = true)
        : id(id), active(active) {}
};
```

Objects can now be created with or without arguments:

```
User first;
User second{101, false};
```

In this approach, the default values are defined through the constructor's default arguments.

However, defining a constructor means that the class no longer qualifies as an `Aggregate`. This approach is therefore an alternative based on constructor initialization, rather than a way to preserve all the properties of an `Aggregate`.

### Approach Three: Use a Factory Function

Sometimes, it is preferable to create objects through a function so that object creation logic remains centralized:

```
struct User {
    int id;
    bool active;
};

User makeDefaultUser() {
    return User{0, true};
}

int main() {
    User user = makeDefaultUser();
}
```

This approach is useful in projects that aim to keep object creation rules in separate functions.

However, a factory function does not, by itself, make it possible to define `Default Member Initializers` while bypassing the language's rules.

## Differences Between Initialization and Default Values

One important consideration when using this feature is understanding the difference between default member initializers and values explicitly supplied when creating an object.

Suppose we have the following structure in C++17:

```
struct Server {
    int port = 8080;
    bool secure = false;
    int timeout = 30;
};
```

We can now create objects in several ways:

```
Server first{};
Server second{443};
Server third{443, true};
Server fourth{443, true, 60};
```

The results are as follows:

| Expression                     | `port` | `secure` | `timeout` |
| ------------------------------ | ------ | -------- | --------- |
| `Server first{}`               | 8080   | `false`  | 30        |
| `Server second{443}`           | 443    | `false`  | 30        |
| `Server third{443, true}`      | 443    | `true`   | 30        |
| `Server fourth{443, true, 60}` | 443    | `true`   | 60        |

This behavior demonstrates that, during `Aggregate Initialization`, explicitly supplied values initialize their corresponding members, while omitted members use their default member initializers.

In contrast, in C++14, a class with `Default Member Initializers` does not qualify as an `Aggregate`. Therefore, these rules must not be applied to that language version without accounting for the difference.

## Design Considerations and Common Mistakes

When designing simple classes, several considerations are particularly important:

- Standard compatibility: If a project is compiled using C++14, do not assume that a class with `Default Member Initializers` still qualifies as an `Aggregate`.
- Constructors versus default member initializers: Defining a default value for a member is not equivalent to defining a constructor with default arguments.
- Member order: In `Aggregate Initialization`, values are assigned according to the order of eligible members. Changing the member declaration order can therefore change the meaning of existing initialization expressions.
- Standard version changes: Code that is valid in C++17 may fail to compile in C++14. Consequently, the target language standard should be explicitly specified in the project's build configuration.
- API compatibility: If a public structure is initialized using positional aggregate initialization, adding or reordering members can affect existing client code.

## Summary

The `Default Member Initializers` feature allows default initial values to be specified for class members and has been available since C++11. However, in C++14, the presence of these initializers prevents a class from qualifying as an `Aggregate`.

This restriction was removed in C++17. As a result, newer language versions allow developers to specify default member initializers while still benefiting from `Aggregate Initialization`.

For C++14 projects, the choice between a simple structure without `Default Member Initializers`, a constructor, and a factory function should depend on the design requirements. The most important principle is to follow the rules of the target language standard and avoid assuming that C++17 behavior also applies to C++14.

---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>