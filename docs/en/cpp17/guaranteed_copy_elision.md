<div align="center">

[🇺🇸 English](./guaranteed_copy_elision.md) | [🇮🇷 فارسی](../../fa/cpp17/guaranteed_copy_elision.md)

</div>

----

# Guaranteed Copy Elision in C++17: Mandatory Copy/Move Omission and Value Categories

Guaranteed copy elision is a major language-level feature introduced in C++17 that transforms copy and move omission from a mere compiler optimization into a **mandatory, core language guarantee**. Even more fundamentally, it changes the semantic definition of prvalues (pure rvalues), allowing developers to return and pass **immovable, non-copyable types** by value directly into their destination storage with zero overhead.

```cpp
#include <iostream>

struct Immovable {
    Immovable() = default;
    // Both copy and move constructors are completely deleted!
    Immovable(const Immovable&) = delete;
    Immovable(Immovable&&) = delete;
};

Immovable makeImmovable() {
    return Immovable{}; // In C++14: Compile error! In C++17: Guaranteed to work!
}

int main() {
    // Constructed directly in 'obj' with ZERO copies and ZERO moves
    Immovable obj = makeImmovable();
}
```

This guide covers the history of copy elision, what C++17 formally changed, how prvalues work under the hood, URVO vs. NRVO, working with immovable types, common pitfalls, and real-world design implications.

---

## Table of Contents

1. [Why Guaranteed Copy Elision Exists](#1-why-guaranteed-copy-elision-exists)
2. [What Was the Problem Before C++17?](#2-what-was-the-problem-before-c17)
3. [The Core Conceptual Shift: Re-defining Prvalues](#3-the-core-conceptual-shift-re-defining-prvalues)
4. [Where Copy Elision Is Guaranteed (Mandatory)](#4-where-copy-elision-is-guaranteed-mandatory)
5. [Where Copy Elision Remains an Optimization (NRVO)](#5-where-copy-elision-remains-an-optimization-nrvo)
6. [Immovable and Non-Copyable Types by Value](#6-immovable-and-non-copyable-types-by-value)
7. [Passing Arguments by Value](#7-passing-arguments-by-value)
8. [Factory Functions and Return-Value Optimization (RVO)](#8-factory-functions-and-return-value-optimization-rvo)
9. [Interaction with Other C++17 Features](#9-interaction-with-other-c17-features)
10. [Common Pitfalls and Gotchas](#10-common-pitfalls-and-gotchas)
11. [Performance Considerations](#11-performance-considerations)
12. [Best Practices](#12-best-practices)
13. [Evolution: C++20 and Beyond](#13-evolution-c20-and-beyond)
14. [Feature-Test Macro](#14-feature-test-macro)
15. [Quick Reference Card](#15-quick-reference-card)
16. [Complete Example](#16-complete-example)
17. [Exercises](#17-exercises)
18. [Conclusion](#18-conclusion)
19. [Contributors](#19-contributors)

---

## 1. Why Guaranteed Copy Elision Exists

Before C++17, whenever you returned an object by value or initialized an object with a temporary expression:
1. The compiler **tried** to optimize away the copy or move (using Return Value Optimization, or RVO).
2. However, the standard still required that a valid **copy or move constructor be accessible and available**, even if the compiler ended up never calling it!
3. Because the optimization was technically optional, language behavior was non-deterministic across different compilers or `-O0` debug builds.

C++17 removes this ambiguity entirely. Copy and move omission for prvalue expressions is no longer an optional optimization; it is a **syntactic and semantic guarantee**.

---

## 2. What Was the Problem Before C++17?

Prior to C++17, temporary objects were conceptually created first, and then moved or copied into their final destination.

### The Inconsistency Dilemma

Consider this simple factory pattern:

```cpp
struct Widget {
    Widget() = default;
    Widget(const Widget&) = delete; // Non-copyable
    Widget(Widget&&) = delete;      // Immovable
};

Widget makeWidget() {
    return Widget{}; // C++14 ERROR: call to deleted constructor of 'Widget'
}
```

Even though an optimizing compiler (`-O2` or `-O3`) would omit the copy/move instruction in assembly, the C++14 semantic check insisted:
> *"The type must have an accessible copy or move constructor to permit this return statement."*

As a result:
- Immovable objects (such as `std::mutex`, types containing mutexes, or atomic wrappers) could **never** be returned by value from a factory function.
- Developers were forced to use heap allocation with `std::unique_ptr`, output parameters `void make(Widget& out)`, or placement new.
- Compiling code with `-O0` could exhibit different constructor/destructor side-effect orders than compiling with `-O2`.

---

## 3. The Core Conceptual Shift: Re-defining Prvalues

To make guaranteed copy elision work cleanly, C++17 did not merely add an exception to constructor checks; it **redefined the core taxonomy of value categories**.

### The Pre-C++17 Model
- A prvalue was thought of as an actual temporary object in memory that needed to be copied or moved into its destination.

### The C++17 Model
- A **prvalue is not an object!**
- A prvalue is simply an **initializer recipe** (a set of instructions for initializing an object).
- An actual object is only materialized when the recipe is bound to a destination (a named variable, a reference, or a parameter). This process is formally called **Temporary Materialization**.

```cpp
Widget w = Widget{};
```
In C++17:
- `Widget{}` does not construct a temporary `Widget` and move it to `w`.
- `Widget{}` is just the initialization recipe.
- `w` is initialized directly in place. There was never any copy or move to omit in the first place!

---

## 4. Where Copy Elision Is Guaranteed (Mandatory)

Mandatory omission occurs in two primary scenarios:

### 4.1 Unnamed Return Value Optimization (URVO)

When a function returns a prvalue of the same class type as the function return type:

```cpp
struct Resource {
    Resource() { std::cout << "Constructed\n"; }
    Resource(const Resource&) { std::cout << "Copied\n"; }
    Resource(Resource&&) { std::cout << "Moved\n"; }
};

Resource create() {
    return Resource{}; // Prvalue return
}

int main() {
    Resource r = create(); // Guaranteed: zero copies, zero moves!
}
```

**Output:**
```text
Constructed
```
*(Notice: No copy or move constructor is ever invoked, even in `-O0` debug builds!)*

### 4.2 Direct Initialization from a Prvalue

When an object is initialized directly with a prvalue of the same type:

```cpp
Resource r = Resource{Resource{Resource{}}};
```
No matter how many nested prvalues are wrapped, exactly **one** object is constructed directly into `r`.

---

## 5. Where Copy Elision Remains an Optimization (NRVO)

It is crucial to understand where elision is **guaranteed** versus where it remains an **optional compiler optimization**.

### Named Return Value Optimization (NRVO) is NOT Guaranteed

When you return a named local variable, the compiler is allowed to elide the copy or move, but it is **not guaranteed**:

```cpp
Resource makeNamed() {
    Resource local;
    // ... complex initialization ...
    return local; // NRVO: local has a name!
}
```

Because `local` is an lvalue (it has a name and storage during the function execution):
1. Copy elision is **optional**. High optimization levels will usually elide it, but compilers are not required to do so.
2. Therefore, the class **must still have an accessible move or copy constructor**!
3. If `Resource` is immovable (`move = delete`), returning `local` will produce a compilation error.

### Guaranteed vs. Optional Comparison

| Scenario | Expression | C++17 Guarantee | Requires Copy/Move Ctor? |
|---|---|---|---|
| Returning a temporary (URVO) | `return Widget{};` | **Guaranteed** | ❌ No |
| Initializing from a temporary | `Widget w = Widget{};` | **Guaranteed** | ❌ No |
| Returning a named variable (NRVO) | `Widget w; return w;` | *Optional (Optimization)* | ✅ **Yes** |
| Returning based on branches | `if (c) return a; else return b;` | *Optional (Optimization)* | ✅ **Yes** |

---

## 6. Immovable and Non-Copyable Types by Value

The most practical breakthrough of guaranteed copy elision is the ability to write expressive, value-based APIs for immovable types.

### Example: Factory Returning an Immovable Type

```cpp
#include <mutex>
#include <string>

struct ThreadSafeLogger {
    std::mutex mtx; // std::mutex is immovable!
    std::string name;

    ThreadSafeLogger(std::string n) : name(std::move(n)) {}

    // Because std::mutex is non-copyable and non-movable, 
    // ThreadSafeLogger is also implicitly immovable!
    ThreadSafeLogger(const ThreadSafeLogger&) = delete;
    ThreadSafeLogger(ThreadSafeLogger&&) = delete;
};

// Valid in C++17!
ThreadSafeLogger makeLogger(std::string name) {
    return ThreadSafeLogger{std::move(name)}; // Direct prvalue return
}

int main() {
    // Constructed directly in logger's memory space:
    ThreadSafeLogger logger = makeLogger("AppLog");
}
```

Prior to C++17, the only ways to implement this were:
- Returning `std::unique_ptr<ThreadSafeLogger>` (costing a heap allocation).
- Passing by reference `void makeLogger(ThreadSafeLogger& out)`.

---

## 7. Passing Arguments by Value

Guaranteed copy elision also applies when passing prvalues as function arguments:

```cpp
void takeImmovable(Immovable arg) {
    // ...
}

int main() {
    // Immovable is initialized directly inside the function's parameter frame
    takeImmovable(Immovable{}); // Guaranteed: no copy, no move!
}
```

The prvalue is materialized directly into the parameter storage of `takeImmovable`.

---

## 8. Factory Functions and Return-Value Optimization (RVO)

Consider an aggregation or builder pattern:

```cpp
struct BigData {
    std::array<int, 1000000> buffer;

    BigData() = default;
    BigData(const BigData&) = delete;
    BigData(BigData&&) = delete;
};

BigData buildData() {
    return BigData{};
}

int main() {
    // 4 MB of memory initialized in-place on the stack, no copying or moving:
    BigData data = buildData();
}
```

The caller allocates the stack space for `data`, passes a hidden pointer to `buildData()`, and `buildData()` constructs `BigData` directly inside the caller's stack frame.

---

## 9. Interaction with Other C++17 Features

### 9.1 Structured Bindings

Guaranteed copy elision works seamlessly with structured bindings when decomposing prvalues:

```cpp
#include <tuple>

std::pair<int, Immovable> createPair() {
    return {1, Immovable{}};
}

int main() {
    // Direct initialization of pair in hidden object 'e'
    auto [id, immovable_item] = createPair();
}
```

### 9.2 `std::optional` in C++17

Notice that `std::optional<T>` still requires move/copy constructors if you try to return an optional containing an immovable type by value, unless using `std::in_place`:

```cpp
#include <optional>

// Requires std::in_place because std::optional owns internal storage
std::optional<Immovable> opt{std::in_place}; // OK!
```

---

## 10. Common Pitfalls and Gotchas

### 10.1 Accidentally Converting a Prvalue into an Lvalue / Xvalue

A very common mistake is using `std::move` when returning a temporary:

```cpp
// PESSIMIZATION: Breaks Guaranteed Copy Elision!
Widget badFactory() {
    return std::move(Widget{}); // DON'T DO THIS!
}
```
Why is this bad?
- `Widget{}` is a prvalue.
- `std::move(Widget{})` casts it to an **xvalue** (an rvalue reference)!
- Guaranteed copy elision **only applies to prvalues**, not xvalues!
- The compiler is now forced to invoke the move constructor. If `Widget` is immovable, compilation fails!

### 10.2 Mistaking NRVO for Guaranteed Elision

```cpp
Immovable riskyFactory(bool condition) {
    Immovable a{};
    Immovable b{};

    if (condition) {
        return a; // ERROR: 'a' is an lvalue (named variable).
                  // Requires copy or move constructor!
    }
    return b;
}
```
If your type is immovable, you **cannot** return named variables from different branches. You must construct the return value as a prvalue directly in the return statement:

```cpp
Immovable safeFactory(bool condition) {
    if (condition) {
        return Immovable{/* option A */}; // Guaranteed!
    }
    return Immovable{/* option B */};     // Guaranteed!
}
```

---

## 11. Performance Considerations

1. **Guaranteed Zero Overhead:** Even in debug mode (`-O0`) or on embedded targets where compiler optimizations are turned off, guaranteed copy elision executes zero copy/move instructions.
2. **Deterministic Side Effects:** In pre-C++17 code, constructor and destructor print statements or logging could vary between debug and release builds depending on whether the compiler elided copies. In C++17, constructor execution count is **strictly deterministic**.
3. **No Move Constructor Overhead:** Types that would otherwise require cheap moves (like `std::vector` copying 24 bytes of pointers) now incur zero overhead when constructed via prvalue factories.

---

## 12. Best Practices

1. **Return by value from factory functions.** Trust guaranteed copy elision for temporaries.
2. **Never write `return std::move(Temporary{});`**. It disables guaranteed copy elision and forces move constructor lookups.
3. **Never write `return std::move(local_var);`** unless `local_var` cannot be returned directly (e.g., returning a base class slice or converting types). For normal variables, standard return already treats local variables as rvalues and allows NRVO.
4. **Design immovable domain types freely.** You no longer need to wrap every mutex-holding class in a dynamic pointer (`std::unique_ptr`) just to return it from a factory.

---

## 13. Evolution: C++20 and Beyond

- **C++20 Simpler Implicit Move:** C++20 expanded rules so that even more expressions in return statements are automatically treated as rvalues when NRVO fails.
- **C++23 Deduced `this` and NRVO:** While proposals have been submitted to make NRVO mandatory under specific single-return-statement conditions, NRVO remains technically optional up to C++23, whereas **URVO prvalue elision remains permanently guaranteed**.

---

## 14. Feature-Test Macro

The feature-test macro for guaranteed copy elision is:

```cpp
__cpp_guaranteed_copy_elision
```

Standard value:
- `201606L` — C++17 Guaranteed Copy Elision support.

```cpp
#if defined(__cpp_guaranteed_copy_elision) && __cpp_guaranteed_copy_elision >= 201606L
    // Guaranteed copy elision is active
#endif
```

---

## 15. Quick Reference Card

| Scenario | Code | Elision Guarantee | Notes |
|---|---|---|---|
| Prvalue Return | `return T{};` | **Mandatory** | Immovable types allowed |
| Prvalue Init | `T x = T{};` | **Mandatory** | Exactly 1 ctor call |
| Named Return (NRVO) | `T x; return x;` | *Optional* | Move/Copy ctor required |
| `std::move` Return | `return std::move(T{});` | ❌ **Disabled** | Anti-pattern; forces move ctor |
| Prvalue Argument | `f(T{});` | **Mandatory** | Materialized directly in parameter |

---

## 16. Complete Example

```cpp
#include <iostream>
#include <string>

class HeavyResource {
private:
    std::string tag_;

public:
    explicit HeavyResource(std::string tag) : tag_(std::move(tag)) {
        std::cout << "[Constructor] HeavyResource: " << tag_ << '\n';
    }

    ~HeavyResource() {
        std::cout << "[Destructor] HeavyResource: " << tag_ << '\n';
    }

    // Completely delete both copy and move operations
    HeavyResource(const HeavyResource&) = delete;
    HeavyResource& operator=(const HeavyResource&) = delete;
    HeavyResource(HeavyResource&&) = delete;
    HeavyResource& operator=(HeavyResource&&) = delete;

    void execute() const {
        std::cout << "Executing task with: " << tag_ << '\n';
    }
};

// Factory returning immovable resource by value
HeavyResource createResource(const std::string& name) {
    return HeavyResource{name}; // Guaranteed copy elision!
}

// Function taking immovable resource by value
void processResource(HeavyResource res) {
    res.execute();
}

int main() {
    std::cout << "--- 1. Initializing directly from factory ---\n";
    HeavyResource res = createResource("DatabaseConnection");
    res.execute();

    std::cout << "\n--- 2. Passing directly by value to function ---\n";
    // Constructed directly in the parameter slot of processResource
    processResource(createResource("WorkerThread"));

    std::cout << "\n--- 3. Exiting main ---\n";
    return 0;
}
```

### Expected Output

```text
--- 1. Initializing directly from factory ---
[Constructor] HeavyResource: DatabaseConnection
Executing task with: DatabaseConnection

--- 2. Passing directly by value to function ---
[Constructor] HeavyResource: WorkerThread
Executing task with: WorkerThread
[Destructor] HeavyResource: WorkerThread

--- 3. Exiting main ---
[Destructor] HeavyResource: DatabaseConnection
```

*(Zero copy constructors and zero move constructors were called!)*

---

## 17. Exercises

### Exercise 1 — Immovable Factory Test
Define a class `NonMovable` with `= delete` for both copy and move constructors. Write a factory function that returns `NonMovable` by value. Compile the code using `-std=c++14` (observe the compiler error) and then with `-std=c++17` (observe successful compilation).

### Exercise 2 — The `std::move` Pessimization
Modify the factory function from Exercise 1 to return `std::move(NonMovable{})`. Try compiling with `-std=c++17`. Explain why the compiler rejects it even though guaranteed copy elision is supported.

### Exercise 3 — URVO vs. NRVO Verification
Write a test class with logging inside its default, copy, and move constructors. Compare:
1. `Test funcA() { return Test{}; }`
2. `Test funcB() { Test t; return t; }`  
Compile with `-O0` and observe whether `funcB` invokes a move constructor while `funcA` invokes none.

### Exercise 4 — Passing to Consumer Functions
Write a function `void consume(NonMovable nm)` and call it via `consume(createNonMovable())`. Verify at what point the destructor of `NonMovable` is invoked.

---

## 18. Conclusion

Guaranteed copy elision is much more than an optimization:
- It fundamentally refines the **value category model** of C++: prvalues are initialization recipes, not constructed temporaries.
- It enables **returning and passing immovable, non-copyable objects by value**.
- It guarantees **zero copies and zero moves** regardless of compiler flags or optimization levels.
- It eliminates the need for heap-allocated wrappers when implementing value-based factory patterns.

---

## 19. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |[Ordikhani](ordikhanifateme@gmail.com) |  | [Ordikhani](@OrdikhaniFateme) |