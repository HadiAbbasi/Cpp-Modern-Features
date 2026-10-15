<div align="center">

[🇺🇸 English](./ctad.md) | [🇮🇷 فارسی](../../fa/cpp17/ctad.md)

</div>

----

# Class Template Argument Deduction (CTAD) in C++17: Deducing Types Without Redundant Boilerplate

Class Template Argument Deduction (CTAD) is a cornerstone language feature introduced in C++17 that allows the compiler to deduce template parameters for class templates directly from constructor arguments. Prior to C++17, template argument deduction was available only for function templates, forcing developers to write repetitive type arguments or rely on helper factory functions like `std::make_pair`, `std::make_tuple`, and `std::lock_guard` wrappers.

```cpp
#include <mutex>
#include <tuple>
#include <vector>

std::mutex mtx;

int main() {
    // Before C++17: Explicit types required everywhere
    std::pair<int, double> p1(42, 3.14);
    std::lock_guard<std::mutex> lock1(mtx);
    std::vector<int> v1 = {1, 2, 3};

    // C++17: CTAD infers the template types automatically
    std::pair p2(42, 3.14);                 // Deduces std::pair<int, double>
    std::lock_guard lock2(mtx);             // Deduces std::lock_guard<std::mutex>
    std::vector v2 = {1, 2, 3};             // Deduces std::vector<int>
    std::tuple t("Alice", 100, 3.14f);      // Deduces std::tuple<const char*, int, float>
}
```

This guide covers why CTAD exists, how implicit deduction guides work, how to author custom explicit deduction guides, aggregate deduction nuances, container pitfalls, interactions with aliases, and practical best practices.

---

## Table of Contents

1. [Why CTAD Exists](#1-why-ctad-exists)
2. [What Was the Problem Before C++17?](#2-what-was-the-problem-before-c17)
3. [How CTAD Works Under the Hood](#3-how-ctad-works-under-the-hood)
4. [Implicit Deduction Guides](#4-implicit-deduction-guides)
5. [Explicit (User-Defined) Deduction Guides](#5-explicit-user-defined-deduction-guides)
6. [Deduction from Aggregate Initialization](#6-deduction-from-aggregate-initialization)
7. [Standard Library Deduction Guides](#7-standard-library-deduction-guides)
8. [CTAD with Iterators and Ranges](#8-ctad-with-iterators-and-ranges)
9. [Interaction with Alias Templates](#9-interaction-with-alias-templates)
10. [Common Pitfalls and Gotchas](#10-common-pitfalls-and-gotchas)
11. [Performance and Compilation Impact](#11-performance-and-compilation-impact)
12. [Best Practices](#12-best-practices)
13. [Evolution: C++20 and Beyond](#13-evolution-c20-and-beyond)
14. [Feature-Test Macro](#14-feature-test-macro)
15. [Quick Reference Card](#15-quick-reference-card)
16. [Complete Example](#16-complete-example)
17. [Exercises](#17-exercises)
18. [Conclusion](#18-conclusion)
19. [Contributors](#19-contributors)

---

## 1. Why CTAD Exists

Function templates in C++ have deduced their template arguments from function parameters since C++98:

```cpp
template <typename T>
void printVal(T val);

printVal(42); // Automatically deduces printVal<int>
```

However, class templates were arbitrarily excluded from this convenience for two decades. Whenever you constructed an object of a class template, you had to duplicate the type arguments, even when the constructor arguments already stated them unambiguously:

```cpp
std::lock_guard<std::mutex> lock(mtx); // Why repeat std::mutex?
```

CTAD eliminates this cognitive and syntactic overhead, making class template instantiation as clean and intuitive as function template invocation.

---

## 2. What Was the Problem Before C++17?

### The Proliferation of `make_*` Helper Functions

To avoid repeating verbose types, the C++ standard library and user code were saturated with helper factory functions:

```cpp
auto p = std::make_pair(10, "test");
auto t = std::make_tuple(1, 2.0, 'a');
auto sp = std::make_shared<Widget>(args);
```

While convenient, `make_*` factory functions introduced noticeable architectural problems:

1. **Inconsistency:** Some standard types had helpers (`std::make_pair`, `std::make_tuple`), while others did not (there was no `std::make_lock_guard` until `std::lock_guard` had to be used, or `std::make_unique` until C++14).
2. **Move/Copy Overhead:** Returning an object from a factory function before C++17 guaranteed copy elision could require moves or accessible move constructors.
3. **Extra Library Code:** Library authors had to write and maintain boilerplate factory functions for every class template they published.

With CTAD, `std::pair(10, "test")` replaces `std::make_pair(10, "test")` directly at the call site.

---

## 3. How CTAD Works Under the Hood

When the compiler encounters a template class name without angle brackets `<...>` during object definition:

```cpp
MyTemplate obj(arg1, arg2);
```

The compiler follows a structured 3-step deduction algorithm:

```text
1. Collect Candidate Guides
   ├── Implicit guides (synthesized automatically from each constructor)
   └── Explicit guides (user-defined via syntax: Class(Args...) -> Class<Types...>;)
          │
          ▼
2. Transform Guides to Hypothetical Function Templates
   Constructor `MyClass(T val)` becomes:
   template <typename T>
   auto __f(T val) -> MyClass<T>;
          │
          ▼
3. Perform Overload Resolution
   Run standard C++ template argument deduction & overload resolution on the set.
   └── Best viable candidate determines deduced class specialization!
```

If overload resolution succeeds and unambiguously selects a single specialization, the object is constructed with that deduced type. If multiple candidates conflict or no candidate matches, the compiler halts with a compilation error.

---

## 4. Implicit Deduction Guides

For every constructor defined in a class template, the compiler automatically generates a corresponding **implicit deduction guide**.

### Example: Simple Wrapper

```cpp
template <typename T>
class Wrapper {
private:
    T value_;
public:
    Wrapper(T val) : value_(val) {}
};

// Compiler automatically synthesizes:
// template <typename T>
// Wrapper(T val) -> Wrapper<T>;

int main() {
    Wrapper w1(100);       // Deduces Wrapper<int>
    Wrapper w2(3.14);      // Deduces Wrapper<double>
    Wrapper w3("hello");   // Deduces Wrapper<const char*>
}
```

### The Implicit Copy Guide

The compiler also automatically synthesizes an implicit guide for the copy/move constructor:

```cpp
// template <typename T>
// Wrapper(Wrapper<T>) -> Wrapper<T>;

Wrapper w1(42);
Wrapper w2(w1); // Deduces Wrapper<int>, NOT Wrapper<Wrapper<int>>!
```

---

## 5. Explicit (User-Defined) Deduction Guides

Implicit guides are not always sufficient. In many designs:
- A constructor takes parameters that do not directly match the class template parameters (e.g., iterators, raw buffers, conversion wrappers).
- You need type decay (converting arrays to pointers or string literals to `std::string_view`).
- You need conditional or SFINAE-based deduction.

An explicit deduction guide uses the following syntax:

```text
template <parameter-list>
Class-Name ( argument-list ) -> Deduced-Class-Specialization ;
```

Deduction guides must be defined in the **same namespace** and **same scope** as the class template, and must appear after the class template declaration.

### Example: Custom RingBuffer Guide

```cpp
#include <cstddef>

template <typename T, std::size_t N>
class FixedBuffer {
public:
    // Takes a pointer to an array
    FixedBuffer(const T* /*data*/) {}
};

// Explicit deduction guide:
// When initialized with a native array T[Size], deduce FixedBuffer<T, Size>
template <typename T, std::size_t Size>
FixedBuffer(const T(&)[Size]) -> FixedBuffer<T, Size>;

int main() {
    int raw_arr[] = {1, 2, 3, 4, 5};
    FixedBuffer buf(raw_arr); // Deduces FixedBuffer<int, 5>!
}
```

---

## 6. Deduction from Aggregate Initialization

In C++17, CTAD works primarily through **constructors**. If a class template is an **aggregate** (it has no user-declared constructors), C++17 **does not** automatically synthesize implicit deduction guides!

### C++17 Aggregate Limitation

```cpp
template <typename T, typename U>
struct PairAggregate {
    T first;
    U second;
};

// In pure C++17, this FAILS without an explicit guide:
// PairAggregate p{1, 2.5}; // Error in C++17! No constructors to deduce from.
```

To make aggregate types work with CTAD in C++17, you **must write an explicit deduction guide**:

```cpp
// Explicit guide for the C++17 aggregate:
template <typename T, typename U>
PairAggregate(T, U) -> PairAggregate<T, U>;

int main() {
    PairAggregate p{1, 2.5}; // OK in C++17 thanks to the explicit guide!
}
```

*(Note: C++20 later solved this limitation by adding automatic CTAD for aggregates; see Section 13).*

---

## 7. Standard Library Deduction Guides

C++17 added explicit deduction guides throughout the standard library:

### 7.1 `std::pair` and `std::tuple`

```cpp
std::pair p(1, 2.0); // std::pair<int, double>
```
The standard library provides guides that apply `std::decay_t`, converting array references and function pointers to their decayed equivalents.

### 7.2 `std::lock_guard` and `std::scoped_lock`

```cpp
std::mutex m1, m2;
std::scoped_lock lock(m1, m2); // std::scoped_lock<std::mutex, std::mutex>
```

### 7.3 `std::array`

`std::array` is an aggregate. C++17 includes a dedicated deduction guide:

```cpp
#include <array>

std::array arr = {1, 2, 3, 4}; // Deduces std::array<int, 4>
```

Under the hood in `<array>`:
```cpp
template <typename First, typename... Rest>
array(First, Rest...) -> array<First, 1 + sizeof...(Rest)>;
```

---

## 8. CTAD with Iterators and Ranges

One of the most practical applications of CTAD is constructing containers from iterator pairs without spelling out the element type:

```cpp
#include <vector>
#include <list>
#include <set>

int main() {
    std::list<int> numbers = {10, 20, 30, 40};

    // Before C++17:
    std::vector<int> v_old(numbers.begin(), numbers.end());

    // C++17 CTAD:
    std::vector v_new(numbers.begin(), numbers.end()); // Deduces std::vector<int>
    std::set s(numbers.begin(), numbers.end());        // Deduces std::set<int>
}
```

### How the Standard Library Implements This

Standard containers use deduction guides with `std::iterator_traits`:

```cpp
template <typename InputIterator>
vector(InputIterator, InputIterator) 
    -> vector<typename std::iterator_traits<InputIterator>::value_type>;
```

---

## 9. Interaction with Alias Templates

In C++17, **CTAD does not work through template aliases (`using`)**:

```cpp
template <typename T>
using StringMap = std::map<std::string, T>;

// Attempting CTAD in C++17:
// StringMap sm = { {"alice", 10}, {"bob", 20} }; // COMPILE ERROR in C++17!
```

The C++17 standard only defines deduction for class templates, not alias templates.

*(Note: C++20 introduced alias template argument deduction, allowing this syntax).*

---

## 10. Common Pitfalls and Gotchas

### 10.1 The `std::vector` Initializer List Trap: Braces vs. Parentheses

CTAD respects constructor overload rules strictly. The difference between `()` and `{}` can completely alter the deduced type:

```cpp
#include <vector>

std::vector v1{5, 2}; // Type: std::vector<int> with 2 elements: [5, 2]
std::vector v2(5, 2); // Type: std::vector<int> with 5 elements: [2, 2, 2, 2, 2]
```

Even more deceptive is nesting single containers:

```cpp
std::vector v{1, 2, 3}; // std::vector<int>

std::vector copy_v{v};  // Deduces std::vector<int> (copy constructor guide wins!)
                        // Does NOT deduce std::vector<std::vector<int>>!
```

If you explicitly want a vector of vectors containing one vector:
```cpp
std::vector<std::vector<int>> nested{v}; // Explicit type required!
```

### 10.2 String Literals: `const char*` vs. `std::string`

```cpp
#include <string>
#include <vector>
using namespace std::string_literals;

std::vector v1 = {"hello", "world"}; // Deduces std::vector<const char*>, NOT std::string!
std::vector v2 = {"hello"s, "world"s}; // Deduces std::vector<std::string>
```

Always use `s` or `sv` literals if you intend to store `std::string` or `std::string_view`.

### 10.3 Non-Deducible Contexts

If a constructor argument does not expose the template parameter in a deducible context, CTAD fails:

```cpp
template <typename T>
struct Identity { using type = T; };

template <typename T>
struct Holder {
    // T is in a non-deducible context via Identity<T>::type
    Holder(typename Identity<T>::type val) {}
};

// Holder h(42); // ERROR: Cannot deduce T!
```

**Fix:** Provide an explicit deduction guide:
```cpp
template <typename T>
Holder(T) -> Holder<T>;
```

---

## 11. Performance and Compilation Impact

- **Zero Runtime Cost:** CTAD occurs entirely at compile time during type resolution. There is zero runtime memory, CPU, or indirection penalty.
- **Negligible Compilation Impact:** Deduction guides simply introduce hypothetical function templates into normal overload resolution sets.

---

## 12. Best Practices

1. **Use CTAD to eliminate redundant typing**, especially with RAII locks (`std::scoped_lock`, `std::lock_guard`) and container iteration.
2. **Watch for array decay:** Remember that string literals deduce to `const char*` by default. Use `""s` or `""sv` when you need string types.
3. **Be explicit when initializing nested containers:** Do not rely on CTAD when initializing a container with another container of the same type (due to copy-guide priority).
4. **Always write explicit deduction guides for custom wrappers** that accept iterators, pointers, or custom handles.
5. **Mark explicit deduction guides `explicit`** if the constructor itself is `explicit` to prevent unintended implicit conversions.

---

## 13. Evolution: C++20 and Beyond

C++20 addressed the primary limitations of C++17 CTAD:

1. **Aggregate CTAD (P1816R0):** C++20 automatically synthesizes deduction guides for aggregates without requiring manual user guides.
2. **Alias Template CTAD (P1814R0):** C++20 allows CTAD to deduce template arguments through `using` alias templates:
   ```cpp
   // Valid in C++20:
   template <typename T>
   using Vec = std::vector<T>;
   Vec v = {1, 2, 3}; // Deduces Vec<int> -> std::vector<int>
   ```

---

## 14. Feature-Test Macro

The feature-test macro for class template argument deduction is:

```cpp
__cpp_deduction_guides
```

Standard values:
- `201606L` — C++17 Initial CTAD support.
- `201907L` — C++20 CTAD for aggregates and alias templates.

```cpp
#if defined(__cpp_deduction_guides) && __cpp_deduction_guides >= 201606L
    // C++17 CTAD is available
#endif
```

---

## 15. Quick Reference Card

| Scenario | Pre-C++17 | C++17 CTAD | Deduced Type |
|---|---|---|---|
| Pair creation | `std::make_pair(1, 2.5)` | `std::pair(1, 2.5)` | `std::pair<int, double>` |
| Mutex lock | `std::lock_guard<std::mutex> l(m);` | `std::lock_guard l(m);` | `std::lock_guard<std::mutex>` |
| Multi-lock | `std::lock(m1, m2); ...` | `std::scoped_lock l(m1, m2);` | `std::scoped_lock<std::mutex, std::mutex>` |
| Vector from iterators | `std::vector<int> v(it1, it2);` | `std::vector v(it1, it2);` | `std::vector<int>` |
| Array initialization | `std::array<int, 3> a{1, 2, 3};` | `std::array a{1, 2, 3};` | `std::array<int, 3>` |

---

## 16. Complete Example

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <mutex>
#include <type_traits>

// Custom generic container
template <typename T>
class DataPacket {
private:
    T payload_;
public:
    DataPacket(T payload) : payload_(std::move(payload)) {}

    void inspect() const {
        std::cout << "Payload: " << payload_ << " (size: " << sizeof(T) << " bytes)\n";
    }
};

// Custom class with iterator-based constructor
template <typename T>
class RangeStore {
private:
    std::vector<T> elements_;
public:
    template <typename It>
    RangeStore(It first, It last) : elements_(first, last) {}

    std::size_t size() const { return elements_.size(); }
};

// Explicit deduction guide for RangeStore
template <typename It>
RangeStore(It, It) -> RangeStore<typename std::iterator_traits<It>::value_type>;

int main() {
    std::cout << "=== 1. Implicit CTAD ===\n";
    DataPacket p1(42);                      // Deduces DataPacket<int>
    DataPacket p2(3.14159);                 // Deduces DataPacket<double>
    DataPacket p3(std::string("Secure"));   // Deduces DataPacket<std::string>

    p1.inspect();
    p2.inspect();
    p3.inspect();

    std::cout << "\n=== 2. Standard Library RAII Lock ===\n";
    std::mutex mtx;
    {
        std::lock_guard lock(mtx); // Deduces std::lock_guard<std::mutex>
        std::cout << "Mutex successfully locked via CTAD lock_guard\n";
    }

    std::cout << "\n=== 3. Explicit Guide with Iterators ===\n";
    std::vector source = {10, 20, 30, 40, 50}; // std::vector<int>
    RangeStore store(source.begin(), source.end()); // Deduces RangeStore<int>
    std::cout << "RangeStore initialized with " << store.size() << " elements\n";

    return 0;
}
```

### Expected Output

```text
=== 1. Implicit CTAD ===
Payload: 42 (size: 4 bytes)
Payload: 3.14159 (size: 8 bytes)
Payload: Secure (size: 32 bytes)

=== 2. Standard Library RAII Lock ===
Mutex successfully locked via CTAD lock_guard

=== 3. Explicit Guide with Iterators ===
RangeStore initialized with 5 elements
```

---

## 17. Exercises

### Exercise 1 — Basic Deduction Verification
Declare a `Point3D<T>` class template with constructor `Point3D(T x, T y, T z)`. Instantiate it with `Point3D pt(1.0f, 2.0f, 3.0f);` and print `typeid(pt).name()` to verify that `float` is deduced.

### Exercise 2 — Writing an Explicit Guide for Fixed Arrays
Create a class template `Matrix<T, Rows, Cols>` and write an explicit deduction guide so that passing a 2D native array `const T(&)[R][C]` automatically deduces `Matrix<T, R, C>`.

### Exercise 3 — The Copy Deduction Trap
Create a `std::vector<int> original = {1, 2, 3};`. Initialize `std::vector copy_attempt{original};`. Inspect its type using `std::is_same_v` to prove whether the deduced type is `std::vector<int>` or `std::vector<std::vector<int>>`.

### Exercise 4 — SFINAE-Constrained Guide
Write an explicit deduction guide for a `SmartBuffer<T>` class that only enables deduction if `std::is_integral_v<T>` is true (using `std::enable_if_t`).

---

## 18. Conclusion

Class Template Argument Deduction (CTAD) in C++17 brings class templates up to parity with function templates:
- It eliminates the need for factory functions like `std::make_pair` and `std::make_tuple`.
- It drastically simplifies RAII locking constructs (`std::lock_guard`, `std::scoped_lock`).
- It allows custom class templates to define **explicit deduction guides** for complex mappings like iterator ranges and raw arrays.
- It is a purely compile-time mechanism with zero runtime overhead.

---

## 19. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |[Ordikhani](ordikhanifateme@gmail.com) |  | [Ordikhani](@OrdikhaniFateme) |