# Fold Expressions in C++17

## Table of Contents

- [Introduction](#introduction)
- [Why Fold Expressions Matter](#why-fold-expressions-matter)
- [Required Headers and Feature-Test Macro](#required-headers-and-feature-test-macro)
- [Core Terminology](#core-terminology)
- [Syntax](#syntax)
- [The 32 Permitted Operators](#the-32-permitted-operators)
- [The Four Fold Forms](#the-four-fold-forms)
  - [Unary Right Fold](#unary-right-fold)
  - [Unary Left Fold](#unary-left-fold)
  - [Binary Right Fold](#binary-right-fold)
  - [Binary Left Fold](#binary-left-fold)
- [Precedence and Parenthesization Rules](#precedence-and-parenthesization-rules)
- [Expansion Mechanics](#expansion-mechanics)
- [Empty Parameter Packs](#empty-parameter-packs)
- [Evaluation Order and Operator Semantics](#evaluation-order-and-operator-semantics)
- [Practical Idioms](#practical-idioms)
- [Complete C++17 Example](#complete-c17-example)
- [Common Mistakes](#common-mistakes)
- [Exercises](#exercises)
- [Conclusion](#conclusion)
- [References](#references)
- [Contributors](#contributors)

---

## Introduction

A **fold expression** is a C++17 language feature that reduces a parameter pack over a binary operator. It replaces much of the recursive boilerplate traditionally used with variadic templates and makes the intended reduction direction explicit.

For example, a logical “all” operation can be written as:

```cpp
template <typename... Args>
bool all(Args... args) {
    return (... && args);
}
```

For `all(true, true, true, false)`, the expression is instantiated as:

```cpp
return ((true && true) && true) && false;
```

The parentheses surrounding a fold expression are part of the required syntax; they are not optional decoration.

## Why Fold Expressions Matter

Before C++17, a variadic operation commonly required a base-case overload and a recursive overload:

```cpp
int add() { return 0; }

template <typename T, typename... Rest>
int add(T first, Rest... rest) {
    return first + add(rest...);
}
```

A binary left fold expresses the same intent directly:

```cpp
template <typename... Args>
auto add(Args... args) {
    return (0 + ... + args);
}
```

Fold expressions generally provide:

- less code than recursive expansion;
- explicit left-to-right or right-to-left grouping;
- natural support for side-effecting comma folds;
- useful constraints and checks over every element of a pack;
- ordinary operator semantics after expansion, including conversions and overload resolution.

A fold is not automatically a loop with a particular runtime performance profile. The compiler forms an expression tree, and the chosen operator determines evaluation, sequencing, associativity, and type behavior.

## Required Headers and Feature-Test Macro

Fold expressions are a core-language feature, so no special header is required. Examples in this document use headers such as:

```cpp
#include <concepts>
#include <cstdint>
#include <iostream>
#include <limits>
#include <type_traits>
#include <utility>
#include <vector>
```

The standard feature-test macro is:

| Macro | Value | Standard | Feature |
|---|---:|---|---|
| `__cpp_fold_expressions` | `201603L` | C++17 | Fold expressions |
| `__cpp_fold_expressions` | `202406L` | C++26 | Ordering of constraints involving fold expressions |

The exact C++26 behavior is beyond the scope of this C++17 guide, but the macro value can be checked conditionally when writing portable headers.

## Core Terminology

### Parameter pack

A parameter pack is a template parameter or function parameter containing zero or more elements. In a fold, `pack` means an expression containing an **unexpanded** pack, such as `args`, `std::forward<Args>(args)`, or `std::numeric_limits<Ts>::max()`.

### Pack expansion

When a template is instantiated, the pack pattern is replaced by one expression per pack element. If the pack contains `E1, E2, ..., EN`, the fold rules below describe the resulting expression tree.

### Initializer (`init`)

In a binary fold, `init` is a single expression that does not contain an unexpanded pack. It supplies the identity-like starting or ending value for the reduction.

### Operator (`op`)

`op` is one of the 32 binary operators permitted by the fold-expression grammar. In a binary fold, the two appearances of `op` must be the same operator token.

### Cast-expression requirement

The `pack` and `init` operands must be cast-expressions: at top level they cannot contain an operator with precedence lower than cast. Parenthesize a compound subexpression when necessary.

## Syntax

The four grammar forms are:

```text
(pack op ...)
(... op pack)
(pack op ... op init)
(init op ... op pack)
```

Their names and meanings are:

| Form | Name | Shape |
|---|---|---|
| `(pack op ...)` | Unary right fold | `E1 op (E2 op (... op EN))` |
| `(... op pack)` | Unary left fold | `((E1 op E2) op ...) op EN` |
| `(pack op ... op init)` | Binary right fold | `E1 op (... op (EN op init))` |
| `(init op ... op pack)` | Binary left fold | `(((init op E1) op E2) ...) op EN` |

The opening and closing parentheses are mandatory in every form.

## The 32 Permitted Operators

The fold grammar permits exactly these binary operators:

| Category | Operators |
|---|---|
| Arithmetic | `+`, `-`, `*`, `/`, `%` |
| Bitwise | `^`, `&`, `\|` |
| Shift | `<<`, `>>` |
| Relational | `<`, `>`, `<=`, `>=` |
| Equality | `==`, `!=` |
| Logical | `&&`, `\|\|` |
| Assignment | `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `^=`, `&=`, `\|=`, `<<=`, `>>=` |
| Member pointer | `.*`, `->*` |
| Other | `,` |

That is 5 + 3 + 2 + 4 + 2 + 2 + 11 + 2 + 1 = 32 operators. The spaceship operator `<=>` is **not** part of the C++17 fold-expression operator list.

In a binary fold, these are invalid because the two operators differ:

```cpp
// return (args + ... * 0); // error: '+' and '*' must match
```

## The Four Fold Forms

Assume the pack expands to `E1, E2, ..., EN`.

### Unary Right Fold

Syntax:

```cpp
(pack op ...)
```

Expansion:

```text
(E1 op (E2 op (... op (EN-1 op EN))))
```

Example:

```cpp
template <typename... Args>
auto divide_right(Args... args) {
    return (args / ...);
}

// divide_right(100, 2, 5)
// 100 / (2 / 5) == 250
```

The right fold preserves right-nested grouping. This is important for non-associative or non-commutative operators.

### Unary Left Fold

Syntax:

```cpp
(... op pack)
```

Expansion:

```text
(((E1 op E2) op ...) op EN)
```

Example:

```cpp
template <typename... Args>
auto divide_left(Args... args) {
    return (... / args);
}

// divide_left(100, 2, 5)
// (100 / 2) / 5 == 10
```

The common “all arguments are true” helper uses a unary left fold:

```cpp
template <typename... Args>
bool all(Args... args) {
    return (... && args);
}
```

### Binary Right Fold

Syntax:

```cpp
(pack op ... op init)
```

Expansion:

```text
E1 op (... op (EN op init))
```

Example:

```cpp
template <typename... Args>
auto subtract_from_end(Args... args) {
    return (args - ... - 100);
}

// subtract_from_end(20, 5, 2)
// 20 - (5 - (2 - 100)) == 113
```

For a non-empty pack, the initializer appears at the far right of the expression tree.

### Binary Left Fold

Syntax:

```cpp
(init op ... op pack)
```

Expansion:

```text
(((init op E1) op E2) ...) op EN
```

Example:

```cpp
template <typename... Args>
auto sum_from_100(Args... args) {
    return (100 + ... + args);
}

// sum_from_100(1, 2, 3) == ((100 + 1) + 2) + 3 == 106
```

With an empty pack, a binary fold evaluates to `init` (subject to the usual validity of that expression).

## Precedence and Parenthesization Rules

### The fold itself must be parenthesized

```cpp
return (... + args); // correct
// return ... + args; // invalid syntax
```

### `pack` and `init` must be cast-expressions

A top-level operator with precedence lower than cast must be parenthesized:

```cpp
template <typename... Args>
int sum_with_six(Args&&... args) {
    // return (args + ... + 1 * 2);   // error: init // correct a cast-expression
    return (args + ... + (1 * 2));   // correct
}
```

The same rule applies to the pack pattern:

```cpp
template <typename... Args>
auto scaled_sum(Args... args) {
    // return ((args + 1) * ...); // the pattern is not the intended fold shape
    return (((args + 1)) + ...); // unary right fold over the parenthesized pattern
}
```

Parentheses also clarify the desired grouping inside a pattern:

```cpp
template <typename... Args>
auto sum_of_doubles(Args... args) {
    return ((args * 2) + ...); // (args1 * 2) + ((args2 * 2) + ...)
}
```

### Do not confuse operator precedence with fold direction

A fold's direction controls the parentheses inserted around the pack elements. It does not change the precedence table of C++. For overloaded operators, the resulting grouped calls still follow normal overload resolution.

## Expansion Mechanics

For `N` pack elements, instantiation expands the four forms as follows:

1. **Unary right fold** `(E op ...)` becomes `(E1 op (... op (EN-1 op EN)))`.
2. **Unary left fold** `(... op E)` becomes `(((E1 op E2) op ...) op EN)`.
3. **Binary right fold** `(E op ... op I)` becomes `(E1 op (... op (EN-1 op (EN op I))))`.
4. **Binary left fold** `(I op ... op E)` becomes `((((I op E1) op E2) op ...) op EN)`.

Here `N` is the number of elements in the pack expansion. The apparent `EN-1` and `EN` notation means the final two elements, not subtraction in the C++ source.

For example:

```cpp
template <typename... Args>
bool all(Args... args) {
    return (... && args);
}

// all(true, true, true, false)
// ((true && true) && true) && false
```

For `N == 1`, unary folds reduce to the one element itself, while binary folds combine the initializer and that element once:

```text
(E op ...)       -> E1
(... op E)       -> E1
(E op ... op I)  -> (E1 op I)
(I op ... op E)  -> (I op E1)
```

## Empty Parameter Packs

A pack can contain zero elements. Empty-pack behavior is deliberately limited for unary folds because most operators do not have one universally correct identity value.

### Unary folds: only three operators are allowed

| Unary operator | Empty-pack result |
|---|---|
| `&&` | `true` |
| `\|\|` | `false` |
| `,` | `void()` |

Examples:

```cpp
template <typename... Args>
bool all_true(Args... args) {
    return (... && args); // all_true() is true
}

template <typename... Args>
bool any_true(Args... args) {
    return (... || args); // any_true() is false
}

template <typename... Args>
void evaluate_all(Args&&... args) {
    (static_cast<void>(std::forward<Args>(args)), ...);
    // evaluate_all() is valid; the result is void()
}
```

A unary fold such as `(... + args)` or `(... * args)` is ill-formed when instantiated with an empty pack:

```cpp
template <typename... Args>
auto product(Args... args) {
    return (... * args); // valid only when the pack is non-empty
}
```

### Binary folds: the initializer handles the empty case

For either binary form, an empty pack evaluates to `init`:

```cpp
template <typename... Args>
auto sum(Args... args) {
    return (0 + ... + args);
}

static_assert(sum() == 0);
static_assert(sum(1, 2, 3) == 6);
```

Choose an initializer that is a genuine identity for the operator and the intended type domain. For example, `0` is appropriate for addition, but not necessarily for a custom overloaded operator.

## Evaluation Order and Operator Semantics

The fold determines grouping, not a universal evaluation policy for every operator.

- `&&`, `||`, and the built-in comma operator provide sequencing/short-circuit behavior according to their ordinary C++ rules.
- With `&&` and `||`, later operands may not be evaluated after the result is known.
- The comma fold is a common way to perform an action once for every element, and the built-in comma operator sequences the evaluations.
- For arithmetic, bitwise, comparison, and assignment operators, preserve normal C++ rules concerning side effects, conversions, sequencing, and undefined behavior.
- A fold using overloaded `operator,`, `operator&&`, or `operator||` may not have the same built-in sequencing or short-circuit properties. Prefer explicit casts or carefully constrained APIs when those guarantees matter.
- Left and right folds can produce different values, types, or overload selections when the operator is non-associative, non-commutative, stateful, or overloaded.

For example, addition commonly gives the same mathematical result in either direction, but subtraction and division do not:

```cpp
// (... / args)  -> ((a / b) / c)
// (args / ...)   -> (a / (b / c))
```

## Practical Idioms

### 1. Print a variadic argument list

```cpp
template <typename... Args>
void print(Args&&... args) {
    (std::cout << ... << std::forward<Args>(args)) << '\\n';
}
```

This is a binary-looking stream chain implemented as a unary left fold. The pack pattern is `std::cout << std::forward<Args>(args)`, and the expression is parenthesized before the final newline.

### 2. Check a constraint for every type/value

```cpp
template <typename T, typename... Args>
void push_back_all(std::vector<T>& destination, Args&&... args) {
    static_assert((std::is_constructible_v<T, Args&&> && ...));
    (destination.push_back(std::forward<Args>(args)), ...);
}
```

The first fold checks every argument; the second executes one `push_back` per argument.

### 3. Execute an expression once per pack element

The comma operator is useful when the individual expression returns a value that should be discarded:

```cpp
template <typename... Args>
void call_each(Args&&... args) {
    (static_cast<void>(use(std::forward<Args>(args))), ...);
}
```

`static_cast<void>` helps ensure that the built-in comma operator is selected rather than an accidental overloaded comma operator.

### 4. Build a stream or visitor chain

Any type that supports the selected operator can participate:

```cpp
template <typename... Args>
auto concatenate(std::string initial, Args&&... args) {
    return (std::move(initial) + ... + std::forward<Args>(args));
}
```

The initializer establishes the result type and makes the empty call well-defined, provided the operation is valid.

### 5. Fold an index sequence

An index sequence can supply a pack whose values are not themselves important. The pack expansion simply repeats a lambda call:

```cpp
template <class T, std::size_t... I>
constexpr T byte_swap_impl(T value, std::index_sequence<I...>) {
    T result{};
    ([&] {
        (void)I;
        // transform value and result for one byte
    }(), ...);
    return result;
}
```

This idiom is useful for unrolling a fixed number of operations without recursive templates.

### 6. Fold member-pointer application

`.*` and `->*` are among the permitted operators, although useful expressions need careful type design:

```cpp
struct Point { int x; };

int Point::* member = &Point::x;
Point p{42};
int value = p.*member;
```

A member-pointer fold is possible when each intermediate expression remains a valid operand for the next member-pointer operation; it is less common than `&&`, `||`, `,`, arithmetic, or stream folds.

## Complete C++17 Example

The following program demonstrates stream output, a type constraint, a comma fold, an integer-sequence fold, empty-pack identities, and a byte-swap implementation. It is self-contained and uses only C++17 facilities except for the abbreviated function-template syntax, which is intentionally avoided.

```cpp
#include <climits>
#include <cstdint>
#include <iostream>
#include <limits>
#include <type_traits>
#include <utility>
#include <vector>

// Fold a stream insertion chain.
template <typename... Args>
void printer(Args&&... args) {
    (std::cout << ... << std::forward<Args>(args)) << '\\n';
}

// Fold an expression that uses a type pack directly.
template <typename... Ts>
void print_limits() {
    ((std::cout << +std::numeric_limits<Ts>::max() << ' '), ...) << '\\n';
}

// Check constructibility and execute one insertion per argument.
template <typename T, typename... Args>
void push_back_vec(std::vector<T>& v, Args&&... args) {
    static_assert((std::is_constructible<T, Args&&>::value && ...),
                  "every argument must construct T");
    (v.push_back(std::forward<Args>(args)), ...);
}

// Repeat a byte transformation sizeof(T) times.
template <class T, std::size_t... I>
constexpr T bswap_impl(T value, std::index_sequence<I...>) {
    T low_byte_mask = static_cast<unsigned char>(-1);
    T result{};
    ([&] {
        (void)I;
        result <<= CHAR_BIT;
        result |= value & low_byte_mask;
        value >>= CHAR_BIT;
    }(), ...);
    return result;
}

template <class T>
constexpr T bswap(T value) {
    static_assert(std::is_unsigned<T>::value,
                  "bswap requires an unsigned integer type");
    return bswap_impl(value, std::make_index_sequence<sizeof(T)>{});
}

template <typename... Args>
bool all_true(Args... args) {
    return (... && args);
}

template <typename... Args>
bool any_true(Args... args) {
    return (... || args);
}

template <typename... Args>
int sum(Args... args) {
    return (0 + ... + args);
}

int main() {
    printer(1, 2, 3, "abc");
    print_limits<std::uint8_t, std::uint16_t, std::uint32_t>();

    std::vector<int> values;
    push_back_vec(values, 6, 2, 45, 12);
    push_back_vec(values, 1, 2, 9);
    for (int value : values)
        std::cout << value << ' ';
    std::cout << '\\n';

    std::cout << std::boolalpha
              << "all_true() = " << all_true() << '\\n'
              << "any_true() = " << any_true() << '\\n'
              << "sum() = " << sum() << '\\n';

    static_assert(bswap<std::uint16_t>(0x1234u) == 0x3412u,
                  "16-bit byte swap failed");
    static_assert(bswap<std::uint64_t>(0x0123456789abcdefULL) ==
                      0xefcdab8967452301ULL,
                  "64-bit byte swap failed");
}
```

Expected output (the exact width of the integer-limit values depends on the implementation):

```text
123abc
255 65535 4294967295
6 2 45 12 1 2 9
all_true() = true
any_true() = false
sum() = 0
```

Compile with a C++17 compiler:

```text
g++ -std=c++17 -Wall -Wextra -pedantic fold_expressions.cpp -o fold_expressions
./fold_expressions
```

## Common Mistakes

### Calling a unary arithmetic fold with no arguments

```cpp
// (... * args) is ill-formed when Args... is empty.
```

Use a binary fold with a suitable initializer, or constrain the function to require at least one argument.

### Reversing left and right syntax

`(... - args)` and `(args - ...)` are different trees. Verify the expansion before using subtraction, division, shifts, assignments, or stateful overloaded operators.

### Forgetting required parentheses

```cpp
// return ... && args; // invalid
return (... && args); // valid
```

### Using a non-cast-expression pattern or initializer

```cpp
// (args + ... + 1 * 2)   // invalid at the top level
(args + ... + (1 * 2))   // valid
```

### Assuming all operators have a useful identity

A binary fold accepts an initializer syntactically, but that initializer must be semantically appropriate. `0` is not a universal identity, and an empty binary fold still has to form a valid result expression.

### Losing value categories

When forwarding function arguments, use `std::forward<Args>(args)` in the fold. Passing `args` directly can turn rvalues into lvalues and can change overload resolution.

### Expecting short-circuiting from overloaded operators

Built-in `&&` and `||` short-circuit. User-defined overloads are function calls and do not provide the same guarantee. Constrain or cast operands when built-in semantics are required.

### Miscounting the operator set

The C++17 list contains 32 operators. `<=>` is not one of them. In a binary fold, both operator tokens must match.

## Exercises

1. Implement `all_true` and `any_true` using unary folds. Add `static_assert` checks for empty and non-empty packs.
2. Implement `sum` using a binary left fold and `product` using a binary left fold. Choose appropriate initial values and document the supported types.
3. Write `divide_left` and `divide_right`, then test them with `(100, 2, 5)` and explain the different results.
4. Implement `print_with_separator(separator, args...)` using a comma fold. Ensure that no separator is printed after the final argument.
5. Write `count_if<Predicate>(args...)`, where a fold over `+` counts how many arguments satisfy a compile-time predicate.
6. Implement `push_back_all` for `std::vector<T>`, checking constructibility with a fold before performing insertions.
7. Write a comma-fold function that invokes `f` for each argument and forwards each argument exactly once.
8. Rewrite a recursive variadic `min` using a binary fold. Explain why a suitable initializer is needed for the empty-pack case.
9. Create a fold over `<<` that prints a heterogeneous pack, preserving lvalue/rvalue categories where relevant.
10. Experiment with a user-defined `operator&&` and compare it with built-in `&&`. Explain why short-circuit assumptions can become invalid.
11. Use `std::index_sequence` and a comma fold to reverse the bytes of an unsigned integer. Verify the result with `static_assert`.
12. Inspect the generated grouping for a custom non-associative operator and describe when a left fold and a right fold are observably different.

## Conclusion

Fold expressions are the direct C++17 notation for reducing a parameter pack over one of 32 permitted binary operators. The four forms are distinguished by whether the pack and initializer are present and by whether the ellipsis is on the left or right of the pack. Left and right folds create different parenthesization, binary folds provide an initializer, and unary folds have special empty-pack identities only for `&&`, `||`, and `,`.

Use explicit parentheses around compound pack patterns and initializers, preserve value categories with forwarding, and remember that ordinary operator semantics still apply after expansion. With these rules in mind, folds make constraints, stream chains, repeated actions, and fixed-size unrolling concise without hiding the underlying expression tree.

## References

- [cppreference — Fold expressions](https://en.cppreference.com/w/cpp/language/fold): grammar, expansion rules, empty-pack identities, and operator list.
- [cppreference — Parameter pack](https://en.cppreference.com/w/cpp/language/parameter_pack): pack declarations and pack expansion rules.
- ISO/IEC 14882:2017, C++ standard, sections covering fold expressions and template parameter packs.

## Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |  |  |  |
