<div align="center">

[🇺🇸 English](./if constexpr.md) | [🇮🇷 فارسی](../../fa/cpp17/if constexpr.md)

</div>

---

 
# `if constexpr` in C++17

> **Quick Summary:** `if constexpr` is a compile-time branch selection mechanism introduced in C++17 that evaluates a condition during template instantiation. The discarded branch is suppressed from code generation, bypassing type-checking requirements for invalid operations on mismatched types without triggering template substitution failures.

---

## Table of Contents
1. [The Motivation: What Problem Does It Solve?](#1-the-motivation-what-problem-does-it-solve)
2. [Mental Model & Compiler Mechanics](#2-mental-model--compiler-mechanics)
3. [Syntax, Header, and Standard Requirements](#3-syntax-header-and-standard-requirements)
4. [Core Mechanics and Supported Scenarios](#4-core-mechanics-and-supported-scenarios)
5. [API Design Contracts & Best Practices](#5-api-design-contracts--best-practices)
6. [Type Deduction, Qualifiers, and Lifetime Rules](#6-type-deduction-qualifiers-and-lifetime-rules)
7. [Evolution Across Standards (C++17 to C++26)](#7-evolution-across-standards-c17-to-c26)
8. [Performance & Zero-Cost Abstraction Analysis](#8-performance--zero-cost-abstraction-analysis)
9. [Common Pitfalls, Antipatterns, and Gotchas](#9-common-pitfalls-antipatterns-and-gotchas)
10. [Comparison with Alternatives](#10-comparison-with-alternatives)
11. [Quick Reference & Cheat Sheet](#11-quick-reference--cheat-sheet)
12. [End-to-End Real-World Example](#12-end-to-end-real-world-example)
13. [Exercises & Practical Challenges](#13-exercises--practical-challenges)
14. [Contributors & Feedback](#14-contributors--feedback)

---

## 1. The Motivation: What Problem Does It Solve?

### 1.1 The Pre-Feature World (Historical Context)
Prior to C++17, implementing compile-time conditional logic required heavy template metaprogramming machinery: **SFINAE** (`std::enable_if_t`), **Tag Dispatching**, or **Explicit/Partial Template Specialization**.

A standard runtime `if` statement evaluates branches at runtime. Even if the condition relies on a compile-time constant expression, the compiler must parse, type-check, and semantically validate **both branches** for every instantiation:

```cpp
// Pre-C++17 Problem: Runtime 'if' with compile-time intent
template <typename T>
void serialize(const T& val) {
    if (std::is_pointer_v<T>) {
        std::cout << *val; // FAILS to compile if T is an int!
                           // Compiler rejects dereferencing an int even though
                           // this branch is theoretically unreachable at runtime.
    } else {
        std::cout << val;
    }
}
```

To work around this limitation before C++17, developers had to split logic into multiple overloaded functions using `std::enable_if_t`:

```cpp
// Pre-C++17 Workaround: Verbose SFINAE Boilerplate
template <typename T>
std::enable_if_t<std::is_pointer<T>::value> serialize_impl(T val) {
    std::cout << *val;
}

template <typename T>
std::enable_if_t<!std::is_pointer<T>::value> serialize_impl(T val) {
    std::cout << val;
}
```
*Downsides:* Severe syntactic bloat, long compilation times, obfuscated call stacks, and cryptic compiler diagnostics upon failure.

### 1.2 The Solution
`if constexpr` enables conditional code inclusion directly inside function bodies. The branch not taken is a **discarded statement**:

```cpp
// C++17 Approach: Clean, readable, and localized
template <typename T>
void serialize(const T& val) {
    if constexpr (std::is_pointer_v<T>) {
        std::cout << *val; // Safely discarded when T is not a pointer
    } else {
        std::cout << val;
    }
}
```

---

## 2. Mental Model & Compiler Mechanics

### 2.1 What the Compiler Actually Does
`if constexpr` operates entirely during the **instantiation phase** of a template:

1. **Condition Evaluation:** The condition is converted to a `bool` via `constexpr` context rules. It **must** be a value-dependent core constant expression.
2. **Discarded Statement Pruning:** The unselected branch becomes a *discarded statement*.
3. **Template Instantiation Suppression:** Entities referenced *only* inside the discarded branch are not instantiated. Operations requiring specific type traits (such as `*val` on a non-pointer) are skipped during the second phase of two-phase lookup.
4. **Syntax Checking Remains Active:** Discarded statements must still be syntactically valid in phase one. You cannot write raw gibberish or tokens that break basic parsing.

```cpp
// Conceptual Transformation:
template <typename T>
void process(T x) {
    if constexpr (sizeof(T) == 4) {
        handle32(x);
    } else {
        handleOther(x);
    }
}

// When instantiated with T = double (sizeof == 8):
// The compiler effectively produces:
void process(double x) {
    handleOther(x); // handle32(x) is completely eliminated from the AST.
}
```

### 2.2 Guarantees & Invariants
* **Zero Binary Bloat:** The discarded branch leaves no traces in the generated assembly; no branch instructions (`jmp`, `je`) are emitted.
* **Scope Independence:** Variable scopes inside `if constexpr` blocks behave identically to runtime scopes. Variables defined inside the block are destroyed at the end of the block.
* **Return Type Deduction:** When using `auto` as a return type, multiple `return` statements in different branches do not conflict if discarded branches are pruned.

---

## 3. Syntax, Header, and Standard Requirements

| Requirement | Value / Detail |
| :--- | :--- |
| **Standard Introduced** | C++17 |
| **Required Header** | None (Core Language Keyword combination) |
| **Key Supporting Headers** | `<type_traits>`, `<utility>` |
| **Feature-Test Macro** | `__cpp_if_constexpr` (Introduced: `201606L`, updated: `201806L`) |

---

## 4. Core Mechanics and Supported Scenarios

### 4.1 Basic Usage: Type-Dependent Logic
Branching based on characteristics of types using `<type_traits>`:

```cpp
#include <type_traits>
#include <string>

template <typename T>
auto stringify(T value) {
    if constexpr (std::is_same_v<T, std::string>) {
        return value;
    } else if constexpr (std::is_arithmetic_v<T>) {
        return std::to_string(value);
    } else {
        return std::string("[unsupported type]");
    }
}
```

### 4.2 Handling Init-Statements (C++17)
Similar to runtime `if`, `if constexpr` supports initializers:

```cpp
template <typename T>
void check_container(const T& container) {
    if constexpr (auto it = container.begin(); std::is_same_v<decltype(it), typename T::iterator>) {
        // 'it' is available here at compile time and runtime
    }
}
```

### 4.3 Recursive Variadic Unpacking Without Base Case Overloads
Before C++17, variadic template recursion required an overloaded base-case function to terminate recursion. `if constexpr` removes this requirement:

```cpp
template <typename First, typename... Rest>
void print_all(First&& first, Rest&&... rest) {
    std::cout << first << " ";
    if constexpr (sizeof...(rest) > 0) {
        print_all(std::forward<Rest>(rest)...); // Recursion stops without base overload!
    }
}
```

### 4.4 Member Detection via Expression Validity
Pairing `if constexpr` with expression detection (e.g., `requires` expressions in C++20 or `std::is_detected` / void_t idioms in C++17):

```cpp
template <typename T>
void clear_buffer(T& buf) {
    if constexpr (requires { buf.clear(); }) {
        buf.clear(); // Invoked if method exists
    } else {
        buf = T{};   // Fallback assignment
    }
}
```

---

## 5. API Design Contracts & Best Practices

### 5.1 Function Parameters & Inlining
* Use `if constexpr` inside universal reference forwarding functions to dispatch without losing value category information:

```cpp
template <typename T>
void dispatch(T&& arg) {
    if constexpr (std::is_rvalue_reference_v<decltype(arg)>) {
        sink(std::move(arg));
    } else {
        inspect(arg);
    }
}
```

### 5.2 Dynamic Return Type Deduction
`if constexpr` allows a single function template to return completely different types depending on template arguments:

```cpp
template <typename T>
auto get_representation(T val) {
    if constexpr (sizeof(T) <= 4) {
        return static_cast<int32_t>(val); // Return type deduced as int32_t
    } else {
        return static_cast<int64_t>(val); // Return type deduced as int64_t
    }
}
```
> **Contract Rule:** If `if constexpr` were replaced with runtime `if`, the above code would fail to compile because auto deduction would encounter conflicting types (`int32_t` vs `int64_t`).

### 5.3 Class Design: Encapsulating Polymorphic Strategy Without Virtual Tables
Eliminate vtable overhead by embedding compile-time strategy selection directly inside class templates:

```cpp
enum class StoragePolicy { Stack, Heap };

template <typename T, StoragePolicy Policy>
class FlatBuffer {
    // Member storage selection
    std::conditional_t<Policy == StoragePolicy::Stack, std::array<T, 1024>, std::vector<T>> data;

public:
    void reset() {
        if constexpr (Policy == StoragePolicy::Stack) {
            data.fill(T{});
        } else {
            data.clear();
            data.shrink_to_fit();
        }
    }
};
```

---

## 6. Type Deduction, Qualifiers, and Lifetime Rules

### 6.1 Dependent vs. Non-Dependent Conditions
* **Dependent Condition:** The condition depends on a template parameter (e.g., `std::is_integral_v<T>`). Discarded branches are not fully instantiated.
* **Non-Dependent Condition:** The condition does not depend on a template parameter (e.g., `sizeof(int) == 4`).
  * **Critical Note:** Even inside a template, if the condition is non-dependent, the compiler **will** check and instantiate the discarded branch!

### 6.2 Scope of Variables inside Initializers
Variables declared in the init-statement of `if constexpr` exist throughout both the `then` and `else` blocks:

```cpp
if constexpr (constexpr int x = compute(); x > 10) {
    // x is valid and constexpr here
} else {
    // x is ALSO valid and constexpr here
}
```

---

## 7. Evolution Across Standards (C++17 to C++26)

| Standard | Changes / Additions | Details / Paper |
| :--- | :--- | :--- |
| **C++17** | Core Feature Introduction | [P0292R2](https://wg21.link/p0292r2) - Foundation of `if constexpr`. |
| **C++20** | Constrained Template Integration | Seamless synergy with Concepts and `requires` clauses. |
| **C++23** | Relaxation on non-dependent template entities | Minor core language defect fixes regarding expression evaluation. |
| **C++26 (Proposed)** | Pack Expansion in `if constexpr` | Proposals exploring fold-like branching patterns over parameter packs. |

---

## 8. Performance & Zero-Cost Abstraction Analysis

### 8.1 Runtime Overhead
* **Exact Zero Cost:** No execution cycles are consumed.
* **Instruction Cache Friendly:** Unused code is completely omitted from the binary, resulting in smaller translation units and reduced instruction cache pressure.

### 8.2 Assembly / Compiler Optimization Perspective
Consider the following comparison on Compiler Explorer (x86-64 Clang/GCC with `-O2`):

```cpp
template <bool OptimizeForSpeed>
int process(int x) {
    if constexpr (OptimizeForSpeed) {
        return x << 2; // Shift
    } else {
        return x / 2; // Division
    }
}

template int process<true>(int);
```

**Emitted Assembly:**
```assembly
process<true>(int):
        lea     eax, [4*rdi]
        ret
```
The condition evaluation, the alternative branch (`x / 2`), and all branching labels (`jmp`) are absent in the generated assembly.

---

## 9. Common Pitfalls, Antipatterns, and Gotchas

### Gotcha 1: The `static_assert(false)` Trap
You want to issue a compile-time error if no branch matches, but writing `static_assert(false)` breaks compilation immediately, even if the branch is discarded!

```cpp
// BROKEN: static_assert(false) triggers on phase-1 parsing!
template <typename T>
void bad_dispatch(T val) {
    if constexpr (std::is_integral_v<T>) {
        // ...
    } else {
        static_assert(false, "T must be integral!"); // ALWAYS FAILS!
    }
}
```
**Fix:** Tie the assertion condition to the template parameter `T` so evaluation is deferred to instantiation time:

```cpp
// Helper dependent false trait
template <typename> inline constexpr bool dependent_false_v = false;

template <typename T>
void good_dispatch(T val) {
    if constexpr (std::is_integral_v<T>) {
        // ...
    } else {
        static_assert(dependent_false_v<T>, "T must be integral!"); // Compiles fine!
    }
}
```

### Gotcha 2: Non-Dependent Expressions Still Cause Syntax Errors
`if constexpr` only suppresses semantic checks on type-dependent operations:

```cpp
template <typename T>
void check() {
    if constexpr (false) {
        int a = "incompatible type"; // COMPILE ERROR: ill-formed even in discarded branch!
    }
}
```

### Gotcha 3: Variable Shadowing and Lifetime Disappointment
A runtime condition cannot be evaluated using `if constexpr`:

```cpp
void run(int runtime_val) {
    // ERROR: runtime_val is not a constant expression!
    if constexpr (runtime_val > 5) {
        // ...
    }
}
```

---

## 10. Comparison with Alternatives

| Technique | Readability | Compile-Time Cost | Constraints | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **`if constexpr` (C++17)** | High (Inline, sequential) | Low | Requires C++17; branches must share outer scope | Internal algorithm dispatching |
| **C++20 Concepts / Requires** | Highest (Declarative) | Very Low | Requires C++20 | Public API overload constraints |
| **SFINAE (`std::enable_if_t`)**| Poor (Heavily signature-polluted) | High (Template substitution overhead) | Works in C++11 | Legacy API backwards compatibility |
| **Tag Dispatching** | Moderate (Splits logic into private helpers) | Moderate | Requires helper structs | Standard library iterator dispatching |

---

## 11. Quick Reference & Cheat Sheet

```
+---------------------------------------------------------------------------------+
|                                 QUICK RULES                                     |
| 1. The condition inside `if constexpr (...)` MUST evaluate to a constant bool.  |
| 2. Use `dependent_false_v<T>` when writing a fallback `static_assert`.          |
| 3. Both branches must contain syntactically valid code (token/grammar check).   |
| 4. Different branches can return different types if return type is `auto`.      |
+---------------------------------------------------------------------------------+
```

| Desired Behavior | Idiomatic Modern Syntax |
| :--- | :--- |
| Check pointer type | `if constexpr (std::is_pointer_v<T>)` |
| Terminate parameter pack | `if constexpr (sizeof...(Args) > 0)` |
| Constrain based on method existence (C++20) | `if constexpr (requires(T obj) { obj.flush(); })` |
| Catch unhandled types at compile-time | `else { static_assert(dependent_false_v<T>); }` |

---

## 12. End-to-End Real-World Example

### Scenario: High-Performance Universal Packet Serializer
A network engine that handles raw binary memory serialization. It handles raw POD data via fast memory copying, strings via sized prepending, and container types through custom iteration—all unified in a single zero-overhead function template.

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <cstring>
#include <type_traits>

// Dependent false helper for compile-time assertions
template <typename>
inline constexpr bool dependent_false_v = false;

class PacketBuffer {
public:
    template <typename T>
    void write(const T& value) {
        using DecayedT = std::decay_t<T>;

        if constexpr (std::is_trivially_copyable_v<DecayedT> && !std::is_pointer_v<DecayedT>) {
            // Case 1: Fast raw memcpy for POD / arithmetic structures
            const uint8_t* bytePtr = reinterpret_cast<const uint8_t*>(&value);
            buffer_.insert(buffer_.end(), bytePtr, bytePtr + sizeof(DecayedT));
            std::cout << "[POD written: " << sizeof(DecayedT) << " bytes]\n";
        }
        else if constexpr (std::is_same_v<DecayedT, std::string>) {
            // Case 2: String serialization with size-prefix
            uint32_t length = static_cast<uint32_t>(value.size());
            write(length); // Recursive compile-time call for the length prefix
            buffer_.insert(buffer_.end(), value.begin(), value.end());
            std::cout << "[String written: " << length << " characters]\n";
        }
        else if constexpr (requires { value.begin(); value.end(); }) {
            // Case 3: Container types (e.g. std::vector, std::list)
            uint32_t count = static_cast<uint32_t>(value.size());
            write(count);
            for (const auto& elem : value) {
                write(elem); // Recursive element dispatch
            }
            std::cout << "[Container written: " << count << " elements]\n";
        }
        else {
            // Case 4: Compile-time safety guard for unsupported types
            static_assert(dependent_false_v<T>, "Type is not serializable by PacketBuffer!");
        }
    }

    size_t size() const { return buffer_.size(); }

private:
    std::vector<uint8_t> buffer_;
};

int main() {
    PacketBuffer packet;

    // 1. Primitive serialization
    int id = 4096;
    packet.write(id);

    // 2. String serialization
    std::string message = "C++17 Engine";
    packet.write(message);

    // 3. Nested container serialization
    std::vector<double> coordinates = { 3.1415, 2.7182, 1.4142 };
    packet.write(coordinates);

    std::cout << "Total payload size: " << packet.size() << " bytes.\\n";
    return 0;
}
```

---

## 13. Exercises & Practical Challenges

### Exercise 1: Warm-up (SFINAE Refactoring)
**Task:** Refactor the following legacy SFINAE-based getter function into a single, clean function using `if constexpr`.

```cpp
template <typename T>
std::enable_if_t<std::is_pointer<T>::value, std::remove_pointer_t<T>>
fetch_value(T ptr) { return *ptr; }

template <typename T>
std::enable_if_t<!std::is_pointer<T>::value, T>
fetch_value(T val) { return val; }
```

### Exercise 2: Intermediate (JSON-Like Variant Parser)
**Task:** Write a function `print_variant` that accepts a `std::variant<int, double, std::string>` and uses `std::visit` combined with a generic lambda containing `if constexpr` to format strings differently (e.g., wrap strings in quotes, format doubles to 2 decimal places, print ints directly).

### Exercise 3: Corner-Case Debugging (Find the Bug)
**Task:** Identify why this code fails to compile and provide the idiomatic modern fix:

```cpp
template <typename T>
void trigger(T val) {
    if constexpr (sizeof(T) == 4) {
        val.custom_32bit_flush();
    } else if constexpr (sizeof(T) == 8) {
        val.custom_64bit_flush();
    } else {
        static_assert(false, "Unsupported size!");
    }
}
```

### Exercise 4: Architectural Design (Compile-Time Device Driver)
**Task:** Implement a template class `DeviceDriver<Architecture>` where `Architecture` is an enum (`x86`, `ARM`, `RISCV`). Using `if constexpr`, implement a `send_command(uint32_t cmd)` method that generates the proper register access assembly/mock instructions for that architecture without incurring any runtime dispatch overhead.

---

## 14. Contributors & Feedback

* **Standard Coverage:** C++17, C++20, C++23, C++26
* **Maintained By:** Modern C++ Documentation Initiative
* **Issues & Contributions:** Submit pull requests or discussions targeting modern template metaprogramming modules.

## 15. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  | [Ordikhani](mailto:ordikhanifateme@gmail.com) |  | [@OrdikhaniFateme](https://t.me/OrdikhaniFateme) |
