<div align="center">

[🇺🇸 English](./non_type_template_parameter_auto.md) | [🇮🇷 فارسی](../../fa/cpp17/non_type_template_parameter_auto.md)

</div>

----

# Non-Type Template Parameters with `auto` in C++17: Deducing Value Types at Compile Time

Non-Type Template Parameters (NTTP) have been part of C++ since its inception, enabling templates to accept compile-time values such as integers, pointers, and enumerations instead of just types. However, prior to C++17, every non-type parameter required an explicitly spelled-out type (e.g., `template <int N>`). This forced developers to write cumbersome workarounds—such as pairing an explicit type template argument with the value, or relying on macro generators—whenever a generic value of arbitrary type had to be accepted.

C++17 resolved this by allowing **`auto` in Non-Type Template Parameters** (`template <auto V>`). The compiler deduces the exact type of the compile-time value directly from the argument provided at instantiation.

```cpp
#include <iostream>

// C++17: The type of 'V' is deduced at compile time!
template <auto V>
void printConstant() {
    std::cout << "Value: " << V << ", Type size: " << sizeof(V) << " bytes\n";
}

int main() {
    printConstant<10>();          // Deduced as: int
    printConstant<10000000000LL>();// Deduced as: long long
    printConstant<'c'>();         // Deduced as: char
    printConstant<true>();        // Deduced as: bool
}
```

This guide explores the historical limitations of NTTPs before C++17, formal deduction mechanics, partial specializations, variadic `auto` packs, combinations with `decltype(auto)`, interactions with modern metaprogramming, common pitfalls, and real-world architectures.

---

## Table of Contents

1. [Why `template <auto>` Exists](#1-why-template-auto-exists)
2. [What Was the Problem Before C++17?](#2-what-was-the-problem-before-c17)
3. [Grammar and Syntax Overview](#3-grammar-and-syntax-overview)
4. [Type Deduction Rules for NTTP `auto`](#4-type-deduction-rules-for-nttp-auto)
5. [Deducing Pointers, References, and Member Pointers](#5-deducing-pointers-references-and-member-pointers)
6. [Variadic `auto` Packs (`template <auto... Values>`)](#6-variadic-auto-packs-template-auto-values)
7. [`auto` vs. `decltype(auto)` in Template Parameters](#7-auto-vs-decltypeauto-in-template-parameters)
8. [Partial Specialization with `auto` NTTP](#8-partial-specialization-with-auto-nttp)
9. [Interaction with Class Template Argument Deduction (CTAD)](#9-interaction-with-class-template-argument-deduction-ctad)
10. [Real-World Use Cases](#10-real-world-use-cases)
    - 10.1 [Type-Safe Heterogeneous Compile-Time Constants](#101-type-safe-heterogeneous-compile-time-constants)
    - 10.2 [Universal Heterogeneous Value Lists (`value_list`)](#102-universal-heterogeneous-value-lists-value_list)
    - 10.3 [Compile-Time Member Reflection Proxies](#103-compile-time-member-reflection-proxies)
11. [Common Pitfalls and Gotchas](#11-common-pitfalls-and-gotchas)
12. [Performance and Assembly Analysis](#12-performance-and-assembly-analysis)
13. [Evolution: C++20 and Beyond (Structural Types and Floating Points)](#13-evolution-c20-and-beyond-structural-types-and-floating-points)
14. [Feature-Test Macro and Compiler Support](#14-feature-test-macro-and-compiler-support)
15. [Quick Reference Card](#15-quick-reference-card)
16. [Complete Example](#16-complete-example)
17. [Exercises](#17-exercises)
18. [Conclusion](#18-conclusion)
19. [Contributors](#19-contributors)

---

## 1. Why `template <auto>` Exists

In modern C++, template metaprogramming often treats compile-time constants (values) with the same flexibility as types. Before C++17, if a template needed to work with an arbitrary value, it also had to be given the **type** of that value, or rely on a standard wrapper type such as `std::integral_constant<T, Val>`:

```cpp
// C++11/C++14: Two parameters needed for a single constant!
template <typename T, T Value>
struct ConstantWrapper {
    static constexpr T value = Value;
};

ConstantWrapper<int, 42> c1;
ConstantWrapper<char, 'z'> c2;
```

This redundancy was contrary to the modern C++ philosophy of minimizing repetitive typing where the compiler has unambiguous information. In `ConstantWrapper<int, 42>`, specifying `int` is redundant because `42` is inherently typed as an `int`.

With C++17, `auto` parameter deduction closes this gap, allowing templates to declare value parameters whose types are inferred directly at the point of instantiation.

---

## 2. What Was the Problem Before C++17?

### Problem 1: Redundant Type Parameters

Consider writing a compile-time fixed-value logger or tag:

```cpp
// C++14
template <typename Type, Type Value>
void logCompileTimeValue() {
    std::cout << Value << '\n';
}

// Call site:
logCompileTimeValue<int, 100>();
logCompileTimeValue<unsigned long, 4000UL>();
```

Callers were forced to repeat the type `int` or `unsigned long`, even though literals carry their own types.

### Problem 2: Heterogeneous Compile-Time Value Lists

Before C++17, implementing a compile-time list holding constants of **differing types** (e.g., `42`, `'a'`, `true`) required complex nested typelists or multiple layers of `std::integral_constant`:

```cpp
// C++14 Workaround:
template <typename... Elements>
struct HeterogeneousList {};

using MyList = HeterogeneousList<
    std::integral_constant<int, 42>,
    std::integral_constant<char, 'A'>,
    std::integral_constant<bool, false>
>;
```

This was verbose, difficult to teach, and significantly increased compiler symbol table sizes and compile times.

---

## 3. Grammar and Syntax Overview

The syntax allows `auto`, references/pointers qualified with `auto`, and `decltype(auto)` anywhere an NTTP type is declared:

```text
template < auto VariableName >
template < const auto* PointerName >
template < const auto& ReferenceName >
template < auto... PackName >
template < decltype(auto) VariableName >
```

### Syntax Showcase

```cpp
// 1. Value parameter (by value)
template <auto N>
struct ValHolder {
    using ValueType = decltype(N);
    static constexpr ValueType value = N;
};

// 2. Reference to static object
template <const auto& Ref>
void readRef() {
    std::cout << Ref << '\n';
}

// 3. Pointer to function or member
template <auto FuncPtr>
struct CallbackDispatcher;

// 4. Variadic pack of arbitrary constants
template <auto... Values>
struct ValueSequence {};
```

---

## 4. Type Deduction Rules for NTTP `auto`

When an NTTP is declared with `auto`, type deduction follows the exact rules of **template argument deduction for functions** (which are closely aligned with standard `auto` variable deduction):

1. **Decay:** Array and function types decay to pointers.
2. **Top-level `const` and references are dropped** when deduced by value (`template <auto V>`).
3. If declared as `template <const auto& R>`, references and `const` qualifiers are preserved.

```cpp
template <auto V>
struct Inspector {
    using Type = decltype(V);
};

// Examples:
Inspector<42>::Type        // int
Inspector<42U>::Type       // unsigned int
Inspector<'A'>::Type       // char
Inspector<true>::Type      // bool

enum class Status { Ok, Err };
Inspector<Status::Ok>::Type // Status
```

### Which Types Are Allowed as NTTPs in C++17?

In C++17, an NTTP deduced via `auto` must still belong to the set of valid non-type template parameter types permitted by C++17:
- Integral types (`int`, `long`, `char`, `bool`, `size_t`, etc.)
- Enumeration types (`enum`, `enum class`)
- Pointers to objects or functions (with static/global linkage)
- References to objects or functions (with static/global linkage)
- Pointers to member functions or member objects
- `std::nullptr_t`

> ⚠️ **Important:** In C++17, floating-point types (`float`, `double`), class types (`std::string`, custom structs), and string literals are **not** permitted as NTTP values. (Floating-point and structural class types were later added in C++20).

---

## 5. Deducing Pointers, References, and Member Pointers

`auto` is not restricted to integers and enums; it cleanly deduces pointers, function pointers, and member pointers.

```cpp
#include <iostream>

struct Engine {
    void start() { std::cout << "Engine started\n"; }
    int rpm = 3000;
};

// Generic Member Invoker taking ANY member function pointer
template <auto MemberFunc>
struct MemberCaller;

template <typename ClassType, typename ReturnType, typename... Args, 
          ReturnType (ClassType::*Func)(Args...)>
struct MemberCaller<Func> {
    static void invoke(ClassType& instance, Args... args) {
        (instance.*Func)(args...);
    }
};

int main() {
    Engine e;
    MemberCaller<&Engine::start>::invoke(e); // Deduced MemberFunc = &Engine::start
}
```

---

## 6. Variadic `auto` Packs (`template <auto... Values>`)

One of the most transformative aspects of this feature is **heterogeneous variadic value packs**:

```cpp
#include <iostream>

template <auto... Values>
struct HeterogeneousValues {
    static void printAll() {
        // C++17 Fold Expression over comma operator
        ((std::cout << Values << " (type size: " << sizeof(decltype(Values)) << ")\n"), ...);
    }
};

int main() {
    using Mixed = HeterogeneousValues<42, 'Z', true, 1000000000ULL>;
    Mixed::printAll();
}
```

### Output

```text
42 (type size: 4)
Z (type size: 1)
1 (type size: 1)
1000000000 (type size: 8)
```

Notice that every single element in `Values...` can have a **different** type. In earlier versions of C++, representing this required multiple layers of template wrappers.

---

## 7. `auto` vs. `decltype(auto)` in Template Parameters

C++17 also permits `template <decltype(auto) V>`.

- `template <auto V>`: Deduces by value. Top-level `const` and reference qualifiers are stripped (decay rules apply).
- `template <decltype(auto) V>`: Preserves the exact declared type, including references and top-level `const`.

```cpp
static int g_number = 100;

template <auto V>
struct ValueTemplate {
    // decltype(V) is 'int' (decayed)
};

template <decltype(auto) V>
struct ExactTemplate {
    // If instantiated with a reference, decltype(V) is 'int&'
};

// Instantiations:
ValueTemplate<g_number> v;       // Error in C++17! 'g_number' is not constant/integral.
ExactTemplate<(g_number)> e;     // Valid: deduces int& (reference to static storage)
```

Use `decltype(auto)` when an NTTP must bind to references of static/global variables while preserving their reference semantics.

---

## 8. Partial Specialization with `auto` NTTP

You can partially specialize templates with `auto` parameters to filter or branch based on the deduced type:

```cpp
#include <iostream>

// Primary template: handles any non-type constant
template <auto Value>
struct ConstantCategory {
    static void identify() {
        std::cout << "General type: " << Value << '\n';
    }
};

// Partial specialization for bool
template <bool Flag>
struct ConstantCategory<Flag> {
    static void identify() {
        std::cout << "Boolean flag: " << std::boolalpha << Flag << '\n';
    }
};

// Partial specialization for char
template <char Character>
struct ConstantCategory<Character> {
    static void identify() {
        std::cout << "Character literal: '" << Character << "'\n";
    }
};

int main() {
    ConstantCategory<123>::identify();    // Primary template (int)
    ConstantCategory<true>::identify();   // Bool specialization
    ConstantCategory<'x'>::identify();    // Char specialization
}
```

---

## 9. Interaction with Class Template Argument Deduction (CTAD)

`template <auto>` pairs naturally with Class Template Argument Deduction (CTAD) introduced in C++17. When defining class templates, CTAD can deduce types from constructor arguments while `template <auto>` handles static configuration constants:

```cpp
template <auto Capacity, typename T>
class FixedRingBuffer {
    T data[Capacity];
    // ...
public:
    FixedRingBuffer(std::initializer_list<T> list) { /* ... */ }
};

// CTAD in action:
FixedRingBuffer<64> buffer = { 1, 2, 3 }; // Capacity deduced as int(64), T deduced as int
```

---

## 10. Real-World Use Cases

### 10.1 Type-Safe Heterogeneous Compile-Time Constants

In high-reliability embedded and systems code, physical units or configuration flags can be passed at compile time without risk of type mismatches:

```cpp
template <auto TargetFrequency>
class TimerModule {
public:
    using FrequencyType = decltype(TargetFrequency);

    static void init() {
        std::cout << "Clock initialized to " << TargetFrequency 
                  << " [Type: " << typeid(FrequencyType).name() << "]\n";
    }
};

TimerModule<16000000UL> cpuClock;  // unsigned long
TimerModule<400U> i2cBusClock;      // unsigned int
```

### 10.2 Universal Heterogeneous Value Lists (`value_list`)

A modern implementation of compile-time value sequences:

```cpp
template <auto... Elements>
struct value_list {
    static constexpr std::size_t size = sizeof...(Elements);
};

// Compile-time indexing
template <std::size_t Index, typename List>
struct get_element;

template <std::size_t Index, auto Head, auto... Tail>
struct get_element<Index, value_list<Head, Tail...>> 
    : get_element<Index - 1, value_list<Tail...>> {};

template <auto Head, auto... Tail>
struct get_element<0, value_list<Head, Tail...>> {
    static constexpr auto value = Head;
    using type = decltype(Head);
};

// Usage:
using MyConfig = value_list<8080, 'V', true>;
static_assert(get_element<0, MyConfig>::value == 8080);
static_assert(get_element<1, MyConfig>::value == 'V');
static_assert(get_element<2, MyConfig>::value == true);
```

### 10.3 Compile-Time Member Reflection Proxies

You can bind member pointers directly into static dispatcher tables without writing explicit types:

```cpp
#include <iostream>

struct PacketDecoder {
    int decodeV1(const char* buf) { return 1; }
    int decodeV2(const char* buf) { return 2; }
};

template <auto MethodPtr>
struct DispatchHandler;

template <typename Class, typename Ret, typename... Args, Ret (Class::*Method)(Args...)>
struct DispatchHandler<Method> {
    static Ret execute(Class& obj, Args&&... args) {
        return (obj.*Method)(std::forward<Args>(args)...);
    }
};

int main() {
    PacketDecoder decoder;
    int res = DispatchHandler<&PacketDecoder::decodeV2>::execute(decoder, "payload");
    std::cout << "Executed handler result: " << res << '\n';
}
```

---

## 11. Common Pitfalls and Gotchas

### 11.1 String Literals Are NOT Allowed in C++17

Developers often attempt to pass string literals to `template <auto>`:

```cpp
template <auto Str>
struct StringHolder {};

// ERROR in C++17:
// StringHolder<"hello"> bad; 
```
*Reason:* In C++17, string literals are arrays of `const char` with static storage duration, but with no external linkage or standard identity as template non-type arguments.

### 11.2 Floating-Point Values Are Forbidden in C++17

```cpp
template <auto V>
void setThreshold() {}

// ERROR in C++17:
// setThreshold<3.14159>(); 
```
*Reason:* C++17 retains the restriction that floating-point numbers cannot be template arguments due to precision representation and NaN/sign bit equality issues across architectures. *(This was relaxed in C++20 for structural types).*

### 11.3 Top-Level Decay of Modifiers

Remember that `template <auto V>` deduces values by decay. If you pass an array with external linkage, `V` becomes a pointer, not an array:

```cpp
extern const int g_numbers[4] = {1, 2, 3, 4};

template <auto X>
struct ArrayInspector {
    // decltype(X) is const int*, NOT const int[4]!
};

ArrayInspector<g_numbers> inspect;
```

---

## 12. Performance and Assembly Analysis

`template <auto>` is a pure compile-time deduction feature. It operates entirely during template instantiation and AST generation within the compiler frontend.

- **Generated Assembly:** The resulting binary instructions are completely identical to hand-written `template <SpecificType, Value>` instantiations.
- **Zero Runtime Overhead:** No runtime type information (RTTI) or indirection is generated.
- **Compile-Time Efficiency:** Eliminating wrapper structures such as `std::integral_constant<T, V>` reduces the number of template instantiations, cutting symbol table pressure and compiler memory consumption.

---

## 13. Evolution: C++20 and Beyond (Structural Types and Floating Points)

C++20 significantly expanded the power of NTTPs beyond C++17's initial boundaries:

1. **Floating-point types:** `template <auto V>` can accept `double` and `float` in C++20:
   ```cpp
   // Valid in C++20:
   template <auto V> struct Constant {};
   Constant<3.1415> pi;
   ```
2. **Structural Class Types:** In C++20, any class type where all base classes and non-static data members are public, non-mutable, and structural can be passed directly as an NTTP (enabling compile-time fixed strings).
3. **Concepts with NTTPs:** In C++20, NTTPs can be constrained directly:
   ```cpp
   template <std::integral auto N>
   void onlyIntegers() {}
   ```

---

## 14. Feature-Test Macro and Compiler Support

The feature-test macro for auto non-type template parameters is:

```cpp
#if defined(__cpp_nontype_template_parameter_auto) && __cpp_nontype_template_parameter_auto >= 201606L
    // C++17 auto NTTP supported!
#endif
```

### Compiler Support Table

| Compiler | Version | Flag |
|---|---|---|
| **GCC** | 7.1+ | `-std=c++17` |
| **Clang** | 4.0+ | `-std=c++17` |
| **MSVC** | 2017 (v19.12+) | `/std:c++17` |
| **Apple Clang** | 9.0+ | `-std=c++17` |

---

## 15. Quick Reference Card

| Syntax | What it deduces | Typical Usage |
|---|---|---|
| `template <auto V>` | Deduces the type of `V` by value (decayed) | Integers, enums, values |
| `template <const auto* P>` | Deduces pointer target type | Pointers to static data/functions |
| `template <const auto& R>` | Deduces reference target type | References to static globals |
| `template <auto... Vals>` | Variadic heterogeneous value pack | Compile-time tuple of constants |
| `template <decltype(auto) V>` | Exact declared type (preserves references) | Reference preservation without decay |

---

## 16. Complete Example

```cpp
#include <iostream>
#include <type_traits>

// 1. Heterogeneous constant printer using Fold Expressions
template <auto... Items>
void displaySequence() {
    std::cout << "Sequence (" << sizeof...(Items) << " items):\n";
    ( (std::cout << "  -> Value: " << Items 
                 << " | Size: " << sizeof(decltype(Items)) 
                 << " Byte(s)\n"), ... );
}

// 2. Type-agnostic compile-time wrapper
template <auto Value>
struct TaggedValue {
    using Type = decltype(Value);
    static constexpr Type data = Value;

    static void print() {
        std::cout << "TaggedValue = " << data << " [type size: " << sizeof(Type) << "]\n";
    }
};

// 3. Enum class support
enum class Subsystem : int {
    Network = 1,
    Storage = 2,
    Compute = 4
};

template <auto SubSys>
struct SubsystemController {
    static_assert(std::is_same_v<decltype(SubSys), Subsystem>, "Must pass a Subsystem enum!");
    
    void boot() {
        std::cout << "Booting subsystem code: " << static_cast<int>(SubSys) << '\n';
    }
};

int main() {
    std::cout << "=== 1. Generic Constants ===\n";
    TaggedValue<42>::print();
    TaggedValue<'M'>::print();
    TaggedValue<0xDEADBEEFULL>::print();

    std::cout << "\n=== 2. Heterogeneous Sequence ===\n";
    displaySequence<100, 'K', true, 999999999999LL>();

    std::cout << "\n=== 3. Strongly Typed Enums ===\n";
    SubsystemController<Subsystem::Storage> storageModule;
    storageModule.boot();

    return 0;
}
```

### Expected Output

```text
=== 1. Generic Constants ===
TaggedValue = 42 [type size: 4]
TaggedValue = M [type size: 1]
TaggedValue = 3735928559 [type size: 8]

=== 2. Heterogeneous Sequence ===
Sequence (4 items):
  -> Value: 100 | Size: 4 Byte(s)
  -> Value: K | Size: 1 Byte(s)
  -> Value: 1 | Size: 1 Byte(s)
  -> Value: 999999999999 | Size: 8 Byte(s)

=== 3. Strongly Typed Enums ===
Booting subsystem code: 2
```

---

## 17. Exercises

### Exercise 1 — Compile-Time Factorial Tag
Create a template `template <auto N> struct FactorialCalculator;` that computes $N!$ at compile-time. Use `static_assert` to ensure that `N` is an integral type and $N \ge 0$.

### Exercise 2 — Heterogeneous Fold Check
Write a template function `template <auto... Values> constexpr bool allPositive();` using C++17 fold expressions that returns `true` if all values in the pack are strictly greater than zero. Test it with mixed types: `allPositive<1, 5UL, 'A'>()`.

### Exercise 3 — Generic Function Pointer Invoker
Write a template `template <auto Func, auto... Args> struct Invoker;` that invokes the function pointer `Func` passing `Args...` and prints the output. Test it with standard functions like `std::abs` or custom math utilities.

### Exercise 4 — Compile-Time Bitmask Flag Aggregator
Implement an enum class `Flags : uint8_t { Read = 1, Write = 2, Exec = 4 };` and a template `template <auto... ActiveFlags> struct PermissionMask;` that calculates the bitwise OR of all active flags at compile-time and exposes it via a `static constexpr uint8_t mask` member.

---

## 18. Conclusion

`auto` in Non-Type Template Parameters represents an essential simplification of C++ metaprogramming:
- It eliminates the redundant requirement to pass both the type and the value (`<typename T, T v>`).
- It enables concise, natural **heterogeneous compile-time lists** and packs (`template <auto...>` ).
- It integrates seamlessly with **Fold Expressions**, **CTAD**, and `decltype`.
- It incurs **zero runtime penalty** while drastically reducing boilerplate in generic architectures.

---

## 19. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |[Ordikhani](ordikhanifateme@gmail.com) |  | [Ordikhani](@OrdikhaniFateme) |