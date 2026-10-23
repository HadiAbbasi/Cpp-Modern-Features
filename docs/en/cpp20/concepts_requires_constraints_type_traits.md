<div align="center">

[🇺🇸 English](./concepts_requires_constraints_type_traits.md) | [🇮🇷 فارسی](../../fa/cpp20/concepts_requires_constraints_type_traits.md)

</div>

---

# A Comprehensive Guide to Concepts, Constraints, and requires in C++20

## Introduction and Motivation

C++20 introduced `Concepts`, the `requires` expression, and the `Constraints` system, significantly improving how templates are defined and used. These features make it possible to express requirements on types and operations directly as part of an API's contract, rather than leaving them in documentation or allowing them to surface only through complicated compiler errors.

Before C++20, programmers commonly used techniques such as `SFINAE`, `std::enable_if`, `Type Traits`, and `static_assert` to constrain templates. These techniques remain useful in certain situations, but they often make declarations more complicated, increase maintenance costs, and produce less transparent diagnostics.

With the newer features, developers can specify exactly which properties a template parameter must provide. For example, they can require that a type supports comparison, that two values can be added, or that a type provides the operations necessary to traverse a range.

The three central terms are closely related, but they do not mean the same thing:

- Concepts: A mechanism for naming and defining sets of type and operational requirements for templates.
- Constraints: Logical rules that determine whether a template or overload is eligible for use.
- requires: A language keyword used in two constructs, `requires-expression` and `requires-clause`, to express or apply requirements.

Understanding the distinctions between these concepts is essential for designing public APIs, template libraries, generic algorithms, and systems with multiple overloads.

## Understanding Constraints and Their Evaluation

### Defining Constraints

In C++20, a `Constraint` is a condition that the compiler can evaluate to determine whether a template or function template is eligible for use. A constraint typically depends on type properties, the validity of an expression, the existence of a member, or a logical relationship.

For example, suppose a function should only accept types for which the addition operator is valid:

```
#include <concepts>

template <typename T>
concept Addable = requires(T a, T b) {
    a + b;
};

template <typename T>
requires Addable<T>
T add(T a, T b) {
    return a + b;
}
```

Here, the `Addable` concept requires the expression `a + b` to be valid for type `T`. The `add` function is eligible for use only when `Addable<T>` is satisfied.

An important detail is that the expression `a + b` does not necessarily produce a result of type `T`. Its result may have a different type because the `Addable` concept does not constrain the result type.

### Constraints Versus Runtime Validation

Constraints are checked at compile time and do not replace runtime input validation.

For example, whether two values can be compared can be checked at compile time. However, whether a number is positive or an index falls within an allowed range usually depends on runtime data.

```
#include <concepts>
#include <stdexcept>

template <std::integral T>
T divide(T numerator, T denominator) {
    if (denominator == 0) {
        throw std::invalid_argument("Division by zero");
    }

    return numerator / denominator;
}
```

Here, `std::integral` restricts the type to integral types, but the denominator must still be checked at runtime. Furthermore, because of integer division rules, this function is intended for integral values; floating-point division requires a different design.

### Constraints and Template Instantiation

When a template is used, the compiler must determine whether its requirements are satisfied for the specified types. If a constraint is not satisfied, the corresponding candidate is generally excluded from the set of eligible overloads.

This behavior is particularly important in `Overload Resolution`, where multiple implementations can share a function name but have different constraints that depend on the arguments' properties.

However, `Constraints` do not guarantee that every operation in a function body is valid. The constraints must match the implementation's actual requirements. If the body depends on an operation that is not expressed in the constraints and is invalid for the selected type, compilation can still fail.

## Introducing requires-expressions

### Structure and Primary Purpose

A `requires-expression` is one of the main features introduced in C++20 for checking whether a collection of expressions and requirements is valid. It typically produces a compile-time `bool` result and can be used directly in a `Concept` definition or a template constraint.

Its general structure is:

```
requires (parameter-list) {
    requirement-sequence
}
```

The parameter list is optional. Its parameters exist only to express and check requirements; they do not need to be instantiated as ordinary runtime objects.

For example:

```
template <typename T>
concept HasSize = requires(const T& value) {
    value.size();
};
```

This concept checks whether `value.size()` is valid for a const object of type `T`. It does not check the return type, time complexity, or actual semantics of the function.

### Types of Requirements in a requires-expression

There are four primary kinds of requirements in this construct:

- `Simple requirement`
- `Type requirement`
- `Compound requirement`
- `Nested requirement`

Each checks a different aspect of a type or expression's validity.

#### Simple Requirements

The simplest requirement checks whether an expression is valid. It does not explicitly constrain the expression's result or type.

```
template <typename T>
concept Incrementable = requires(T value) {
    ++value;
    value++;
};
```

In this example, both prefix and postfix increment expressions must be valid. If either expression is unavailable for type `T`, the concept is not satisfied.

The important point is that each expression must be valid according to the language's syntax and semantic rules, but it is not necessarily executed at runtime. Here, the expressions are used to check the capabilities of a type.

#### Type Requirements

A type requirement checks whether a name denotes a valid type. This is particularly useful when a type is a nested member of another type.

```
template <typename T>
concept HasValueType = requires {
    typename T::value_type;
};
```

In this example, `T` must have a member named `value_type` that can be interpreted as a type.

This requirement only checks that the type name is valid. It does not guarantee that the resulting type satisfies the requirements of a particular algorithm.

#### Compound Requirements

A compound requirement is useful when an expression must be valid and additional properties must be checked, such as its result type or whether it can throw an exception.

Its general structure is:

```
requires {
    { expression } noexcept -> Concept;
}
```

Both `noexcept` and `-> Concept` are optional. The `noexcept` specifier requires the expression not to be potentially throwing. The `-> Concept` portion applies a concept to the expression's result type.

For example:

```
#include <concepts>

template <typename T>
concept IntegerLikeResult = requires(const T& value) {
    { value.getValue() } -> std::integral;
};
```

Here, `value.getValue()` must be valid, and its result type must satisfy `std::integral`.

An important detail is that, in a compound requirement, the concept following `->` is applied to `decltype((expression))`. The expression's type is therefore determined using the rules of `decltype` with double parentheses. Reference and `const` qualifications can affect the result.

For example, if `getValue` returns a reference to an integral value, the result does not necessarily satisfy `std::integral`, because that concept does not accept a reference type as an integral type.

#### Nested Requirements

A nested requirement expresses another logical condition inside a `requires-expression`. The condition must be introduced with `requires`.

```
#include <concepts>

template <typename T>
concept SmallIntegral = requires {
    requires std::integral<T>;
    requires (sizeof(T) <= 4);
};
```

In this example, `T` must be an integral type whose size is no greater than four bytes.

Note that type sizes are implementation-dependent. The condition `sizeof(T) <= 4` does not necessarily restrict the type to a specific bit width on every platform. Furthermore, `sizeof` reports the size of an object in units of the implementation-defined byte, which is not necessarily an eight-bit unit.

### Using Parameters in requires-expressions

Parameters inside a `requires-expression` parameter list help express requirements that depend on particular types.

```
template <typename T>
concept ComparableWithSelf = requires(const T& a, const T& b) {
    a == b;
    a != b;
};
```

These parameters are local declarations used for checking requirements. They cannot be used like ordinary variables in a function body.

If a `requires-expression` appears in a non-templated context and its requirements are invalid from the outset, compilation can fail rather than simply producing `false`. Therefore, not every `requires` expression can turn arbitrary syntax or semantic errors into an unsuccessful logical condition.

## Understanding requires-clauses and Applying Constraints

### The General Structure of a requires-clause

Unlike a `requires-expression`, a `requires-clause` applies constraints to a template declaration.

The clause can appear after the template parameter list or after the function parameter list.

```
#include <concepts>

template <typename T>
requires std::integral<T>
T twice(T value) {
    return value + value;
}
```

Here, `requires std::integral<T>` is a `requires-clause` that restricts the function to integral types.

The same constraint can be expressed more concisely with a concept:

```
#include <concepts>

template <std::integral T>
T twice(T value) {
    return value + value;
}
```

These declarations express equivalent type constraints. The second form uses a `Constrained Template Parameter`, while the first explicitly writes a `requires-clause`.

### Comparing the Placement of requires-clauses

In C++20, a `requires-clause` can appear after the template parameter list or after the function parameter list.

```
#include <concepts>

template <typename T>
requires std::integral<T>
T first(T value) {
    return value;
}

template <typename T>
T second(T value)
    requires std::integral<T>
{
    return value;
}
```

Both functions have the same constraint. Choosing where to write it is primarily a matter of readability and declaration structure.

For templates with more complex constraints, keeping the requirements organized and close to the relevant declaration makes the function's contract easier to understand.

### Using requires to Constrain Classes

Constraints are not limited to functions. They can also be applied to `Class Template` declarations.

```
#include <concepts>

template <std::integral T>
class Counter {
public:
    explicit Counter(T initial) : value_(initial) {}

    T value() const {
        return value_;
    }

private:
    T value_;
};
```

In this example, instantiating `Counter` with a non-integral type is not permitted.

This approach is useful when designing classes that require specific properties from their data types. However, restricting a type to a standard category is not always sufficient. Additional requirements, such as incrementability, convertibility, or support for specific operations, may also need to be checked.

## Introducing Concepts and Defining Them

### Understanding Concepts in C++20

A `Concept` is a named set of constraints that specifies the properties a type must satisfy to be used in a particular templated context.

Concepts are defined with the `concept` keyword, and their definitions must produce a valid logical condition.

```
#include <concepts>

template <typename T>
concept Arithmetic = std::integral<T> || std::floating_point<T>;
```

In this example, `Arithmetic` covers integral types and standard floating-point types.

Contrary to a common assumption, this concept does not cover every type that can be used for mathematical calculations. For example, a custom complex-number type may support arithmetic operations without automatically satisfying either `std::integral` or `std::floating_point`.

### Using Standard Concepts

The C++20 standard library provides a collection of general-purpose concepts in the `<concepts>` header. They are designed for use in templated APIs and are usually clearer and more standardized than repeatedly defining equivalent constraints from scratch.

| Standard concept             | Purpose                                                                |
| ---------------------------- | ---------------------------------------------------------------------- |
| `std::same_as`               | Checks whether two types are identical                                 |
| `std::derived_from`          | Checks for a public, unambiguous inheritance relationship              |
| `std::convertible_to`        | Checks the expected convertibility requirements                        |
| `std::integral`              | Checks integral types                                                  |
| `std::floating_point`        | Checks standard floating-point types                                   |
| `std::signed_integral`       | Checks signed integral types                                           |
| `std::unsigned_integral`     | Checks unsigned integral types                                         |
| `std::constructible_from`    | Checks whether a type can be constructed from specified arguments      |
| `std::default_initializable` | Checks default-initialization requirements                             |
| `std::copy_constructible`    | Checks copy-construction requirements                                  |
| `std::move_constructible`    | Checks move-construction-related requirements                          |
| `std::assignable_from`       | Checks assignment requirements                                         |
| `std::destructible`          | Checks destruction requirements                                        |
| `std::invocable`             | Checks whether an object can be invoked with specified arguments       |
| `std::predicate`             | Checks the requirements of a predicate                                 |
| `std::regular`               | Checks a set of common requirements for regular types                  |
| `std::semiregular`           | Checks basic requirements for default-initializable and copyable types |
| `std::equality_comparable`   | Checks equality-comparison requirements                                |
| `std::totally_ordered`       | Checks total-ordering requirements                                     |
| `std::ranges::range`         | Checks the basic requirements of a range                               |

The appropriate header must be included to use these concepts. All concepts listed here belong to C++20, although some concepts associated with `Ranges` are defined in the `std::ranges` namespace.

### Distinguishing Similar Concepts

Some standard concepts have similar names but express different contracts.

For example, `std::convertible_to<From, To>` does not merely mean that an expression can be converted to `To`. It also checks requirements involving implicit conversion and `static_cast`, along with specified semantic expectations.

Similarly, `std::derived_from<Derived, Base>` is more restrictive than `std::is_base_of_v`. It checks for public, unambiguous inheritance, so the results can differ in cases such as private inheritance.

Likewise, `std::move_constructible<T>` does not mean that the type must have a dedicated constructor with the exact signature `T(T&&)`. A type might also be constructible from an rvalue through a constructor taking `const T&` and still satisfy the concept.

### Designing Custom Concepts

In real projects, standard concepts do not always cover every requirement of an API. In such cases, a custom concept can express the actual contract directly.

```
#include <concepts>

template <typename T>
concept NumericAccumulator = requires(T a, T b) {
    { a + b } -> std::same_as<T>;
    { a += b } -> std::same_as<T&>;
    { T{} } -> std::same_as<T>;
};
```

In this example, the type must support addition and compound addition assignment, with the results matching the expected types.

However, this concept does not guarantee that addition behaves correctly according to mathematics, avoids overflow, or preserves the expected properties of an accumulator. Such semantic properties generally need to be specified through documentation, type design, and appropriate tests.

### Combining Concepts and Reusing Requirements

One of the main advantages of concepts is that simpler concepts can be combined to build higher-level concepts.

```
#include <concepts>

template <typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template <typename T>
concept Additive = requires(T a, T b) {
    { a + b } -> std::same_as<T>;
};

template <typename T>
concept NumericAdditive = Numeric<T> && Additive<T>;
```

In this example, `NumericAdditive` accepts only types that satisfy both `Numeric` and `Additive`.

This approach is particularly useful in large libraries because shared requirements can be defined once and reused across multiple algorithms and classes.

However, concepts should not be split into excessively small units without a real need. Many scattered concepts that provide neither meaningful reuse nor semantic value can reduce readability.

## Named Concepts Versus Raw Constraints

### Using Named Concepts

When a constraint has an independent meaning in the API design or is used in several places, it is often best expressed as a named concept.

```
#include <concepts>
#include <cstddef>

template <typename T>
concept StringLike = requires(const T& value) {
    { value.data() } -> std::convertible_to<const char*>;
    { value.size() } -> std::convertible_to<std::size_t>;
};
```

The `StringLike` concept introduces a recognizable contract. Using it in function signatures communicates the purpose of the constraint more clearly than a collection of scattered expressions.

However, names such as `StringLike` must match the actual requirements. This concept does not necessarily guarantee all the properties of a string, such as null termination, memory ownership, or support for a particular encoding.

### When Raw Constraints Are Appropriate

If a constraint is used only once and has no independent semantic meaning, a direct condition may be more appropriate.

```
template <typename T>
requires (sizeof(T) <= 16)
void processSmallObject(const T& value) {
    // Implementation
}
```

Here, the size requirement is written directly where it is used.

The condition should still match the function's purpose. For example, a small type is not necessarily inexpensive to copy or faster to process.

## Understanding Overload Resolution with Constraints

### Restricting Function Candidates

One important application of constraints is defining multiple overloads with the same name but different contracts.

```
#include <concepts>
#include <string>

template <std::integral T>
std::string describe(T value) {
    return "integral";
}

template <std::floating_point T>
std::string describe(T value) {
    return "floating point";
}
```

In this example, calls with integral types select the first overload, while calls with standard floating-point types select the second.

```
auto a = describe(42);
auto b = describe(3.14);
```

If a type satisfies neither constraint, neither overload is eligible unless another overload with a suitable constraint is available.

### Prioritizing Overloads with Subsumption

`Subsumption` determines how logical relationships between constraints affect overload selection. When one constraint subsumes another according to the standard's rules, this relationship can help the compiler identify the more constrained candidate.

```
#include <concepts>
#include <string>

template <typename T>
requires std::integral<T>
std::string classify(T) {
    return "integral";
}

template <typename T>
requires std::signed_integral<T>
std::string classify(T) {
    return "signed integral";
}
```

In this example, `std::signed_integral` includes the `std::integral` constraint. Therefore, for a signed integral type, the second overload is more constrained and is selected.

```
auto a = classify(10);
auto b = classify(10u);
```

The first call returns `signed integral`, while the second returns `integral`.

The key point is that the compiler does not determine subsumption merely by simplifying logical expressions as a person might perform ordinary algebra. The standard defines specific rules involving `Atomic Constraints`, their equivalence, and the logical structure of constraints.

### Constraints That Appear Equivalent

Two conditions can be logically equivalent without necessarily being equivalent for subsumption.

```
#include <concepts>

template <typename T>
concept IsIntegral = std::integral<T>;

template <typename T>
requires IsIntegral<T>
void handle(T) {}

template <typename T>
requires std::integral<T>
void handle(T) {}
```

This example should not be treated as a valid overload design for ordinary use. Despite their similar meanings, the two constraints do not necessarily establish the subsumption relationship required for unique selection, and a call may be ambiguous.

This behavior is related to the identity of `Atomic Constraint` objects. Their identity is not determined solely by logical equivalence or identical-looking text; the source expression and the rules for mapping template parameters also matter.

To avoid these problems, define shared requirements in a single concept and use that concept consistently in all overloads that need a specific priority relationship.

### Named Concepts and API Design

Reusing a shared concept improves more than readability; it can also affect overload resolution.

```
#include <concepts>

template <typename T>
concept IntegralType = std::integral<T>;

template <IntegralType T>
void process(T) {}

template <IntegralType T>
requires std::signed_integral<T>
void process(T) {}
```

In this design, the second overload has the first overload's constraints plus an additional requirement that the type be signed. For signed integral types, the second overload is more constrained.

When developing public libraries, design relationships between overloads deliberately and test them with different types, especially when several combined constraints or custom concepts are involved.

## Understanding Atomic Constraints and Constraint Normalization

### Atomic Constraints

An `Atomic Constraint` is the smallest unit that directly participates in the subsumption rules. It generally consists of a logical expression that depends on template parameters and the associated parameter mapping.

For example, consider the following condition:

```
template <typename T>
concept SmallAndIntegral =
    std::integral<T> && (sizeof(T) <= 4);
```

The final constraint combines two logical conditions. During `Constraint Normalization`, the structure is transformed into a form that allows relationships between its atomic components to be examined.

This process is one reason consistent reuse of named concepts matters when designing overloads.

### Constraint Normalization

`Constraint Normalization` is the process of transforming constraints into a normalized logical structure according to the standard's rules. When concepts and composite constraints are evaluated, this process takes into account the expansion of other constraints and the mapping of template parameters.

This process is not equivalent to ordinary simplification of logical expressions. For example, two expressions may be mathematically equivalent but differ in their atomic structure and subsumption relationships.

In practical library design, avoid relying on superficial logical simplifications. Instead, define shared constraints as standard or custom concepts and reuse the same definitions in related declarations.

### Combining Constraints Logically

Constraints can be combined using the logical operators `&&` and `||`.

```
#include <concepts>

template <typename T>
concept IntegerOrFloatingPoint =
    std::integral<T> || std::floating_point<T>;

template <typename T>
concept SignedOrSmall =
    std::signed_integral<T> || (sizeof(T) <= 2);
```

In the first example, a type must be either integral or a standard floating-point type.

In the second example, a type must either be signed integral or have a size of at most two bytes. This condition alone does not guarantee that the type is numeric, signed, or suitable for arithmetic operations.

Because constraints are evaluated according to language rules, the ordering and dependency structure of conditions can matter when checking expression validity. In particular, requirements should be written to avoid checking operations whose prerequisites are not satisfied.

## Applying Constraints to Generic Types

### Checking Callability

Generic algorithms sometimes need to verify that an object can be invoked with specified arguments. The standard library provides `std::invocable` for this purpose.

```
#include <concepts>
#include <utility>

template <typename F, typename... Args>
requires std::invocable<F, Args...>
decltype(auto) invokeCallable(F&& callable, Args&&... args) {
    return std::forward<F>(callable)(
        std::forward<Args>(args)...
    );
}
```

Here, the constraint checks whether `callable` can be invoked with the specified argument types.

`std::forward` preserves the value categories of the inputs when forwarding them to the final invocation. However, this implementation is not a complete replacement for `std::invoke`, which also supports pointers to member functions and pointers to data members, in addition to ordinary callable objects.

### Checking Construction and Copying

When designing generic classes, concepts such as `std::constructible_from`, `std::copy_constructible`, and `std::movable` can express requirements related to constructing and moving objects.

```
#include <concepts>

template <std::copy_constructible T>
T duplicate(const T& value) {
    return value;
}
```

This function can only be used with types that satisfy the requirements of `std::copy_constructible`.

Standard concepts sometimes go beyond a simple check for the existence of a constructor. They can express collections of requirements involving conversions, assignments, and semantic properties. Therefore, select a concept based on the intended contract, not merely because its name resembles the desired operation.

### Designing Range-Based Algorithms

In C++20, the `Ranges` library uses concepts to express the requirements of algorithms and ranges. This allows algorithms to accept input types based on their required capabilities rather than requiring a specific pointer or iterator type.

```
#include <concepts>
#include <ranges>

template <std::ranges::input_range R>
void inspectRange(R&& range) {
    for (auto&& element : range) {
        (void)element;
    }
}
```

Here, the parameter must satisfy `std::ranges::input_range`, which expresses the basic requirements for traversing the input through the Range interface.

An important detail is that `input_range` does not necessarily imply support for multiple independent traversals, random access, or element modification. If an algorithm needs stronger capabilities, it should use a more appropriate concept, such as `forward_range`, `bidirectional_range`, or `random_access_range`.

### Using Iterator Concepts

Iterator concepts in C++20 express the differences between traversal capabilities explicitly.

| Concept                       | Purpose                                       |
| ----------------------------- | --------------------------------------------- |
| `std::input_iterator`         | Reading input while traversing                |
| `std::output_iterator`        | Writing output                                |
| `std::forward_iterator`       | Repeated forward traversal                    |
| `std::bidirectional_iterator` | Forward and backward traversal                |
| `std::random_access_iterator` | Random access and distance-related operations |
| `std::contiguous_iterator`    | Contiguous ordering of elements in memory     |

Choose these concepts according to the algorithm's actual requirements. For example, an algorithm that reads data once from beginning to end should not unnecessarily require a `random_access_iterator`, because doing so would exclude otherwise suitable iterators.

### Concepts Related to Containers

The standard library's `std::ranges::range` concept checks whether the required beginning and ending operations are valid for an object.

However, this concept alone does not guarantee that the object is a complete Container, owns the memory occupied by its elements, or supports operations such as insertion and removal.

When designing algorithms, express the specific capabilities they need rather than requiring a particular Container type. This approach makes it easier to support arrays, `std::vector`, `std::array`, views, and compatible custom types.

## Comparing Concepts with SFINAE and Type Traits

### The Traditional Approach to Constraining Templates

Before C++20, one common technique for constraining templates was `SFINAE`, short for Substitution Failure Is Not An Error. It refers to situations in which a failure during template argument substitution does not immediately produce an error but instead removes the corresponding template candidate from consideration.

For example, in C++17, a function could be constrained using `std::enable_if_t`:

```
#include <type_traits>

template <typename T,
          std::enable_if_t<std::is_integral_v<T>, int> = 0>
T doubleValue(T value) {
    return value + value;
}
```

This approach is valid, but compared with concepts, it expresses the constraint less directly.

The same goal can be expressed in C++20 as follows:

```
#include <concepts>

template <std::integral T>
T doubleValue(T value) {
    return value + value;
}
```

The second version is more readable and communicates the purpose of the constraint directly.

### Key Differences Between the Approaches

| Feature                            | `SFINAE`                                                 | `Concepts` and `Constraints`                                   |
| ---------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------- |
| Declaration readability            | Often more complicated                                   | Usually clearer                                                |
| Expressing multiple requirements   | Often requires combining several techniques              | Direct support for logical constraints                         |
| Overload selection                 | Based on SFINAE and overload resolution rules            | Also uses subsumption rules in addition to overload resolution |
| Error messages                     | Often harder to understand                               | Usually more closely related to the failed constraint          |
| Semantic requirements              | Often based on types and expressions that can be checked | Makes type and operational contracts easier to express         |
| Compatibility with older standards | Suitable for C++11 and later                             | Requires C++20                                                 |
| Standard library usage             | Still widely applicable                                  | Used extensively in newer APIs and `Ranges`                    |

`Concepts` do not replace every use of `SFINAE`. Some traditional techniques remain useful in code targeting older language standards or in specialized designs. Concepts also interact with substitution and expression-validity rules during template processing.

### Comparing Concepts with Type Traits

`Type Traits` have been available in the standard library since C++11. They allow programmers to inspect properties such as whether a type is integral, whether it can be constructed, whether it derives from another type, and many other characteristics.

```
#include <type_traits>

template <typename T>
constexpr bool isInteger = std::is_integral_v<T>;
```

In this example, `isInteger` is a `constexpr` variable that makes the result of a type check available at compile time.

In C++20, the same property can be expressed as a concept:

```
#include <concepts>

template <typename T>
concept IntegerType = std::integral<T>;
```

The important distinction is that `Type Traits` generally provide information about a type as a value or a helper type, whereas `Concepts` are designed to express constraints directly in template declarations.

These tools complement each other. If you need information such as `std::is_trivially_copyable_v<T>`, you can continue to use `Type Traits`; there is no need to reimplement every property using `requires-expression`.

## Comparing Constraints with static_assert

### Different Roles in Compile-Time Validation

`static_assert` has been available since C++11 and is used to produce a compile-time error when a condition is not satisfied.

```
#include <type_traits>

template <typename T>
T identity(T value) {
    static_assert(std::is_integral_v<T>,
                  "T must be an integral type");

    return value;
}
```

In this example, the function produces an explicit error for a non-integral type, but the condition is not applied during overload selection.

By contrast, the following version attaches the constraint directly to the template declaration:

```
#include <concepts>

template <std::integral T>
T identity(T value) {
    return value;
}
```

In this case, calling the function with a non-integral type makes the constrained candidate ineligible.

### When Is static_assert Better?

`static_assert` remains useful for checking internal invariants, compile-time configuration, and conditions that should explicitly trigger an error.

For example, if an implementation is usable only on platforms where a particular type has a specified size, checking that condition inside the implementation may be appropriate.

By contrast, if the goal is to make a public function eligible for overload resolution only for certain categories of types, a `Concept` or `requires-clause` generally produces a better design.

## Understanding Dependencies and Validation Rules

### Expressions Dependent on Template Parameters

In templates, the validity of an expression may depend on the template arguments. For example, `value.size()` is valid only if the actual type of `value` provides such a member.

```
template <typename T>
concept HasSize = requires(const T& value) {
    value.size();
};
```

In this example, the expression's validity is checked for different types, and the result contributes to evaluating the concept.

However, dependent names must be distinguished from non-dependent names. Non-dependent names may be checked before a template is instantiated for a specific type. Therefore, `requires-expression` should not be treated as a mechanism for hiding every kind of syntax error or invalid expression.

### Managing the Order of Constraint Checks

In logical expressions, condition ordering can matter when avoiding the evaluation of an invalid dependent expression.

```
#include <concepts>

template <typename T>
concept HasNestedTypeAndValue = requires {
    typename T::value_type;
    requires std::integral<typename T::value_type>;
};
```

In this example, the existence of `T::value_type` is checked first, and the nested requirement depends on that type.

The structure of the conditions should make dependencies explicit. In particular, when combining custom concepts, avoid writing conditions that depend on nonexistent types or members without considering the language's rules for logical short-circuiting and template validity.

### Recursive Concepts and Mutual Dependencies

Defining concepts with recursive dependencies can interact poorly with the rules governing constraint definitions and normalization. Concepts are designed to compose requirements, but this does not mean that every circular dependency is valid.

In library design, base concepts should preferably be independent, with higher-level concepts built on top of them. This structure is easier to read and simplifies the analysis of logical relationships.

## Understanding the Semantic Limitations of Concepts

### Valid Expressions Versus Correct Behavior

One of the most important points about concepts is that many of their checks concern the validity of expressions rather than the semantic correctness of program behavior.

```
template <typename T>
concept AddableToSelf = requires(T a, T b) {
    a + b;
};
```

This concept only checks whether the addition expression is valid. It does not guarantee that the result corresponds to mathematical addition, that the operator behaves reasonably for particular inputs, or that the operation avoids overflow.

For example, a custom type could define `operator+` to perform string concatenation or another operation instead of mathematical addition. The expression could still be valid from the compiler's perspective.

### Semantic Requirements of Standard Concepts

Some standard concepts have semantic requirements in addition to syntactic checks. For example, `std::equality_comparable` and `std::totally_ordered` rely on contracts governing comparison relationships.

The compiler cannot generally prove all these contracts. Consequently, satisfying a standard concept may also require adhering to documented semantic requirements.

Therefore, passing the compiler's checks is not sufficient to prove that a custom type behaves correctly. Tests and documentation should also cover the expected behavior.

### Overly Restrictive Concepts

If an algorithm only requires the expression `a + b`, restricting its parameter to `std::integral` may be unnecessary. Custom classes and other numeric types may also provide this operation.

For example:

```
template <typename T>
requires requires(const T& a, const T& b) {
    a + b;
}
auto sum(const T& a, const T& b) {
    return a + b;
}
```

This version requires only that the addition expression be valid, allowing a wider range of types.

However, this flexibility must match the function's actual requirements. If the implementation depends on a specific result type, numeric behavior, or additional operations, those requirements should also be expressed.

## Guidelines for Writing and Organizing Constraints

### Choosing Among the Three Common Forms

In C++20, constraints can be expressed using a concept in a template parameter, a `requires-clause`, or, in some cases, a `requires-expression` directly in a constraint.

```
#include <concepts>

template <std::integral T>
T first(T value) {
    return value;
}

template <typename T>
requires std::integral<T>
T second(T value) {
    return value;
}

template <typename T>
T third(T value)
    requires requires(T x) {
        x + x;
    }
{
    return value;
}
```

Each form is suitable for a different purpose:

- The first form is appropriate when the constraint is simple and can be named directly.
- The second form makes the constraint explicit at the declaration.
- The third form expresses an operational requirement directly in the function's constraint, which can be useful for specialized conditions.

In library code, defining a custom `Concept` is generally the best choice when a condition is reused or has a meaningful semantic name.

### Avoiding Repeated Constraints

Repeating long conditions across multiple functions makes code harder to maintain. If a condition changes, every dependent declaration must remain consistent.

```
#include <concepts>

template <typename T>
concept IncrementableValue = requires(T value) {
    ++value;
};

template <IncrementableValue T>
void advanceOnce(T& value) {
    ++value;
}

template <IncrementableValue T>
void advanceTwice(T& value) {
    ++value;
    ++value;
}
```

Here, the requirement that a type support prefix increment is defined once and reused by two functions.

However, this concept checks only that the prefix increment expression is valid. If the algorithm also depends on the operator's result type or semantic properties, those requirements must be expressed explicitly.

### Avoiding Incompatible Constraints

When multiple overloads have overlapping constraints but no clear subsumption relationship, overload resolution may become ambiguous.

To prevent this problem, identify the supported type categories first and then design the relationships between the constraints. If one overload must take priority over another in a particular set of cases, express that relationship using compatible concepts and logical constraints.

## Practical Applications in Library Design

### Designing Numeric APIs

In computational libraries, concepts can distinguish between integral types, floating-point types, and custom numeric types. This makes it possible to detect invalid type usage earlier and provide diagnostics that are more relevant to the problem.

However, if an algorithm requires properties such as limited precision, specific overflow behavior, or division operations, the simple concepts `std::integral` and `std::floating_point` may not express all the necessary requirements.

### Designing Callable APIs

In systems that accept functions, lambdas, and callable objects, `std::invocable` and `std::predicate` can clarify the input contract.

For example, a sorting algorithm may require a comparison function that can be invoked with two elements and returns a result compatible with the predicate requirements. Using the appropriate concept in the template declaration can make comparator-related errors easier to detect.

### Designing Resource-Management Classes

For classes that own or transfer resources, concepts such as `std::movable` and `std::move_constructible` can help express the type requirements.

However, a type being movable is not the same as guaranteeing that its resource transfer is correct. Concepts alone do not ensure that a custom type remains in a semantically appropriate state after a move or releases its resources correctly.

### Designing Generic Algorithms

In generic algorithms, `Ranges` and iterator concepts help express the precise operations required by an algorithm. This avoids unnecessarily restricting the algorithm to a particular Container and makes it easier to support views, lazy ranges, and compatible custom types.

For example, an algorithm that needs only forward traversal should use `std::forward_iterator` or an appropriate Range concept rather than unnecessarily requiring random access.

## Understanding Error Messages and Debugging

### Interpreting Constraint Diagnostics

One advantage of concepts is that, in many cases, the compiler can identify which requirement was not satisfied. However, the format and detail of diagnostics depend on the compiler and its version.

For example:

```
#include <concepts>

template <std::integral T>
T square(T value) {
    return value * value;
}

int main() {
    auto result = square(3.14);
}
```

The argument `3.14` has type `double` and does not satisfy `std::integral`. Therefore, the constrained template is not eligible for this call.

To debug such errors, it is helpful to proceed in the following order:

- First, identify which overload or template is being considered for the call.
- Then, check whether the argument types satisfy the declared constraints.
- If a custom concept is involved, inspect the requirements inside its `requires-expression` individually.
- If multiple overloads exist, check subsumption relationships and possible ambiguities.
- Finally, verify that the function body does not depend on operations that are missing from the constraints.

### Using Development Tools

For developing concept-based libraries, recent versions of GCC, Clang, or MSVC are recommended. Although these compilers support C++20, the completeness of their implementations and the quality of their diagnostics can vary between versions.

Using the language-standard option to enable C++20 is also helpful:

```
g++ -std=c++20 -Wall -Wextra -Wpedantic main.cpp -o main
```

Clang can also use the `-std=c++20` option. These options belong to the compiler and are not part of the C++ language syntax.

## Distinguishing Language Features from Standard Library Features

### Understanding the Boundary Between Language and Library

When using concepts, it is important to distinguish between features of the language itself and facilities provided by the standard library.

| Feature                    | Category         | Standard version |
| -------------------------- | ---------------- | ---------------- |
| `concept` keyword          | Language feature | C++20            |
| `requires-expression`      | Language feature | C++20            |
| `requires-clause`          | Language feature | C++20            |
| `Constraint Normalization` | Language rules   | C++20            |
| `Constraint Subsumption`   | Language rules   | C++20            |
| `std::integral`            | Standard library | C++20            |
| `std::same_as`             | Standard library | C++20            |
| `std::invocable`           | Standard library | C++20            |
| `std::ranges::range`       | Standard library | C++20            |
| `std::is_integral_v`       | Standard library | C++17            |

This distinction matters because a compiler might support the `requires` syntax while the version of its standard library, or its implementation of a particular `Ranges` feature, has a different level of support than expected.

Furthermore, the `-std=c++20` option alone does not guarantee that every standard feature is implemented completely and without defects in every compiler version.

### Checking Compiler Support

In cross-platform projects, test the required features against the target compilers and standard libraries. When necessary, language feature-test macros such as `__cpp_concepts` can be used to check the level of support.

```
#ifndef __cpp_concepts
#error "This project requires C++20 Concepts support"
#endif
```

This check verifies that the macro is defined, but it does not by itself guarantee complete and defect-free support for every related feature.

For public projects, using a build system such as CMake and specifying the required language standard in the project configuration is a more suitable way to manage compiler requirements.

## Important Design Considerations

### Choosing Constraints That Match Actual Requirements

The best constraint is one that expresses exactly what the implementation needs. If an algorithm requires only one operation, restricting it to a specific type category may unnecessarily reduce flexibility.

Conversely, a constraint that is too weak can allow unsuitable types and cause complicated errors to emerge from the function body. The goal is to balance flexibility with an accurate description of the contract.

### Prefer Standard Concepts When Appropriate

If a standard concept accurately expresses the required contract, using it is generally a good choice. A custom concept is valuable when the project's requirements cannot be expressed precisely by existing concepts or when naming the contract provides meaningful semantic value.

For example, using `std::integral` is clearer and more consistent than redefining the standard requirement for integral types.

### Avoiding Unsupported Semantic Guarantees

A `Concept` can check whether an operation is valid, but it usually cannot prove every behavioral property. If an algorithm depends on characteristics such as total ordering, specific numeric behavior, or invariant preservation, those requirements should also be covered by documentation and tests.

### Designing Overloads Consistently

In APIs with multiple overloads, the relationships between constraints should be designed deliberately. Merely writing different `requires-clause` expressions does not guarantee that the compiler will select the intended overload when their constraints overlap.

Using shared concepts, avoiding unnecessary duplication, and testing edge cases can reduce ambiguity.

### Considering Compatibility with Earlier Standards

If a library must also support C++17 or earlier standards, C++20 language features cannot be used directly in every build configuration.

In such projects, `Type Traits` and `SFINAE` can provide implementations compatible with older standards, or different API implementations can be selected according to the target language standard. The appropriate approach depends on the library's users and support policy.

## Final Summary

`Concepts`, `Constraints`, and `requires` in C++20 provide tools for designing clearer templates, expressing contracts more precisely, and improving overload selection. They make it possible to specify an algorithm's or generic class's requirements directly in its declaration rather than leaving them hidden in its implementation.

The most important points are:

- A `Concept` names a set of type and operational requirements.
- A `Constraint` determines whether a template or overload is eligible for use.
- A `requires-expression` checks the validity of expressions and requirements.
- A `requires-clause` attaches constraints to a template declaration.
- `Constraint Normalization` and `Subsumption` play important roles in selecting constrained overloads.
- Standard library concepts are generally preferable to redefining common constraints.
- `Concepts` do not completely replace `SFINAE`, `Type Traits`, or `static_assert`; each retains its own use cases.
- The validity of an expression does not, by itself, guarantee the semantic correctness of a type's behavior.
- Constraints should match an algorithm's actual requirements to avoid unnecessarily restricting users or accepting unsuitable types.

Ultimately, using these features effectively requires more than learning the syntax of `requires`. Understanding logical relationships between constraints, knowing the semantic contracts of standard concepts, and deliberately designing overloads are essential to making professional use of C++20's template capabilities.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |
