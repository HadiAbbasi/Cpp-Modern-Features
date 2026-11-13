<div align="center">

[🇺🇸 English](./nodiscard.md) | [🇮🇷 فارسی](../../fa/cpp17/nodiscard.md)

</div>

----

# `[[nodiscard]]` in C++17 and C++20

Introduced in **C++17** via proposal **P0188R1** ("Wording for `[[nodiscard]]` attribute") and significantly expanded in **C++20** via proposal **P1301R4** ("`[[nodiscard("should have a reason")]]`") and **P1771R1** ("`[[nodiscard]]` for constructors"), `[[nodiscard]]` is a standard attribute that encourages compilers to emit a diagnostic warning when a function's return value (or an instance of an annotated type) is discarded.

In safety-critical, systems, and concurrent programming, ignoring return values is among the leading causes of critical security vulnerabilities, resource leaks, silent logic errors, and concurrency deadlocks. `[[nodiscard]]` enforces compile-time semantic contracts directly in API interfaces without incurring runtime overhead.

```cpp
#include <system_error>

[[nodiscard]] std::error_code transmit_telemetry(const uint8_t* payload, size_t len);

void process() {
    uint8_t frame[64]{};
    // Compile-Time Warning / Error: Discarding return value of function declared with 'nodiscard'
    transmit_telemetry(frame, sizeof(frame)); 
}
```

---

## Table of Contents

1. [Motivation & Software Safety](#1-motivation--software-safety)
2. [C++17 Core Mechanics: Function vs. Type Level](#2-c17-core-mechanics-function-vs-type-level)
3. [C++20 Evolutions: String Explanations & Constructors](#3-c20-evolutions-string-explanations--constructors)
4. [Critical Systems Use Cases & Idiomatic Patterns](#4-critical-systems-use-cases--idiomatic-patterns)
5. [The `std::unique_lock` / `std::lock_guard` Pitfall](#5-the-stdunique_lock--stdlock_guard-pitfall)
6. [Intentional Discard Idiom: `std::ignore` and `(void)`](#6-intentional-discard-idiom-stdignore-and-void)
7. [Standard Library Retrofitting](#7-standard-library-retrofitting)
8. [Feature-Test Macros & Clang/GCC Attributes](#8-feature-test-macros--clanggcc-attributes)
9. [Pitfalls and Anti-Patterns](#9-pitfalls-and-anti-patterns)
10. [Contributors](#10-contributors)

---

## 1. Motivation & Software Safety

Traditionally, C and early C++ relied on compiler-specific extensions like GCC/Clang's `__attribute__((warn_unused_result))` or MSVC's `_Check_return_`. The absence of a standard attribute led to fragmented, non-portable codebases.

Functions frequently return status codes, error tokens, allocated resource handles, or query results whose disposal indicates either a programming error or an unhandled failure state:

- **Resource Leaks**: Functions returning newly allocated memory, file descriptors, or socket handles (e.g., `allocate()`, `open_file()`).
- **Silent Failures**: Functions returning status enums or error codes (e.g., `try_lock()`, `validate_checksum()`).
- **Purity & Side-Effect Fallacies**: Functions that compute a value without altering system state (e.g., `std::vector::empty()`, `std::clamp()`). A common bug is writing `vec.empty()` expecting it to erase elements like `vec.clear()`.

`[[nodiscard]]` establishes a standard contract across all conforming compilers to diagnose these issues at compile time with zero binary or runtime cost.

---

## 2. C++17 Core Mechanics: Function vs. Type Level

In C++17, `[[nodiscard]]` can be applied at two granularities: **function declarations** and **type definitions** (classes, structs, and enums).

### A. Function-Level Annotation
When placed on a function or lambda declaration, any direct call to that function that discards the returned value triggers a compiler warning:

```cpp
[[nodiscard]] int calculate_crc32(const void* data, size_t len);

void run() {
    calculate_crc32(nullptr, 0); // Warning: return value ignored
}
```

### B. Type-Level Annotation (Class / Struct / Enum)
When placed on a class, struct, or enum definition, **any** function returning that type by value inherits the `[[nodiscard]]` requirement, even if the function itself is not annotated:

```cpp
enum class [[nodiscard]] Status {
    Success,
    Timeout,
    Corrupted
};

// Even without [[nodiscard]] on verify_packet, the return type enforces it:
Status verify_packet();

void inspect() {
    verify_packet(); // Warning: ignoring return value of type 'Status' declared with 'nodiscard'
}
```

```text
+-----------------------------------------------------------------------------------------+
|                                  [[nodiscard]] Scope                                    |
+-----------------------------------------------------------------------------------------+
| Target                    | Trigger Condition                                           |
+---------------------------+-------------------------------------------------------------+
| Function Declaration      | When this specific function is called and value is dropped. |
| Class / Struct Definition | When ANY function returns this type by value and is dropped.|
| Enum Definition           | When ANY function returns this enum and is dropped.         |
+-----------------------------------------------------------------------------------------+
```

---

## 3. C++20 Evolutions: String Explanations & Constructors

C++20 introduced two major extensions that elevate `[[nodiscard]]` to a full-fledged API design tool.

### A. Explanatory Reason String (P1301R4)
Similar to `static_assert`, developers can now communicate the reasoning and actionable fix directly in compiler diagnostics:

```cpp
[[nodiscard("Ignoring the token leads to memory leaks; pass it to release_token().")]]
uint64_t acquire_token();

[[nodiscard("Did you mean vec.clear()? empty() only checks if the container is empty.")]]
bool empty() const noexcept;
```

**Compiler Output (Clang/GCC):**
```text
warning: ignoring return value of function declared with 'nodiscard' attribute: 
Ignoring the token leads to memory leaks; pass it to release_token(). [-Wunused-result]
```

### B. Constructor Annotation (P1771R1)
In C++17, `[[nodiscard]]` on constructors was technically ill-formed or undefined. C++20 officially standardizes `[[nodiscard]]` for constructors:

```cpp
class ScopedConnection {
public:
    [[nodiscard("ScopedConnection must be bound to a variable to keep the link alive.")]]
    explicit ScopedConnection(int socket_id);
    ~ScopedConnection();
};

void run(int sock) {
    ScopedConnection(sock); // Warning: temporary destroyed immediately; link closed!
}
```

---

## 4. Critical Systems Use Cases & Idiomatic Patterns

### Use Case 1: Pure Query Functions
Pure functions that inspect state without modifying the object must never have their output dropped:

```cpp
template <typename T>
class RingBuffer {
public:
    [[nodiscard]] bool full() const noexcept;
    [[nodiscard]] bool empty() const noexcept;
    [[nodiscard]] size_t capacity() const noexcept;
};
```

### Use Case 2: Factory & Allocation Methods
Returning ownership via raw or smart pointers:

```cpp
class MemoryPool {
public:
    [[nodiscard]] void* allocate(size_t bytes);
};
```

### Use Case 3: Thread Synchronization & Non-Blocking Locks
Attempting to acquire a lock conditionally:

```cpp
class SpinLock {
public:
    [[nodiscard]] bool try_lock() noexcept;
    void unlock() noexcept;
};

void execute(SpinLock& lock) {
    // If try_lock() fails and return is ignored, the thread proceeds into data races!
    lock.try_lock(); 
}
```

---

## 5. The `std::unique_lock` / `std::lock_guard` Pitfall

One of the most insidious bugs in concurrent C++ occurs when naming a RAII lock guard without a variable identifier:

```cpp
#include <mutex>

std::mutex mtx;
int shared_resource = 0;

void bad_worker() {
    // BUG: Missing variable name! Constructs a temporary std::lock_guard, 
    // immediately destructs it, and unlocks mtx before accessing shared_resource!
    std::lock_guard<std::mutex>(mtx); 

    shared_resource++; // CRITICAL RACE CONDITION: Unprotected access!
}
```

In C++20, RAII types such as `std::lock_guard`, `std::unique_lock`, and `std::scoped_lock` have their constructors annotated with `[[nodiscard]]`:

```cpp
template <class Mutex>
class [[nodiscard]] lock_guard {
    // ...
};
```

If an engineer attempts `std::lock_guard<std::mutex>(mtx);`, the compiler rejects or warns about the dropped temporary instance immediately.

---

## 6. Intentional Discard Idiom: `std::ignore` and `(void)`

There are rare cases where an engineer intentionally chooses to ignore a `[[nodiscard]]` value after verifying that doing so is sound. 

### Method A: Cast to `void` (Standard C++ Language Rule)
The C++ standard mandates that an explicit cast to `void` discards the expression and **suppresses** the `[[nodiscard]]` warning:

```cpp
[[nodiscard]] int sync_flush();

void teardown() {
    (void)sync_flush(); // Standard suppression: no compiler warning generated
}
```

### Method B: `std::ignore` Assignment
When working with structured bindings or `std::tuple`, `std::ignore` can explicitly consume unwanted values:

```cpp
#include <tuple>

[[nodiscard]] std::pair<int, bool> insert_key(int key);

void process() {
    std::ignore = insert_key(42); // Explicitly ignored via assignment
}
```

---

## 7. Standard Library Retrofitting

Over successive revisions (C++17, C++20, C++23), the ISO C++ committee has systematically retrofitted `[[nodiscard]]` across `<algorithm>`, `<memory>`, `<vector>`, `<future>`, and `<mutex>`:

| Function / Type | Header | Motivation |
|---|---|---|
| `std::async` | `<future>` | Dropping the returned `std::future` makes the call synchronous (blocking on destruction). |
| `std::allocate` | `<memory>` | Discarding returns immediately causes unrecoverable memory leaks. |
| `std::empty` | `<iterator>` | Common confusion with `container.clear()`. |
| `std::launder` | `<new>` | Purity; does not modify the pointer in-place. |
| `std::remove` / `std::unique` | `<algorithm>` | They do not erase elements; discarding the returned iterator causes truncation bugs. |
| `std::lock_guard` / `std::scoped_lock` | `<mutex>` | Prevents creating nameless, immediately destroyed temporary locks. |

---

## 8. Feature-Test Macros & Clang/GCC Attributes

The presence of `[[nodiscard]]` and its enhancements can be verified portably using `__has_cpp_attribute`:

```cpp
#include <iostream>

#if defined(__has_cpp_attribute)
    #if __has_cpp_attribute(nodiscard) >= 201907L
        // C++20: nodiscard with reason string and constructor support
        #define SAFE_NODISCARD(reason) [[nodiscard(reason)]]
    #elif __has_cpp_attribute(nodiscard) >= 201603L
        // C++17: basic nodiscard
        #define SAFE_NODISCARD(reason) [[nodiscard]]
    #else
        #define SAFE_NODISCARD(reason)
    #endif
#else
    #define SAFE_NODISCARD(reason)
#endif

SAFE_NODISCARD("Return status must be inspected")
int secure_init();
```

---

## 9. Pitfalls and Anti-Patterns

### 1. Overusing on Fluent / Method Chaining Interfaces
Applying `[[nodiscard]]` to methods designed for chaining (like builders) can generate redundant noise if callers frequently end chains early:

```cpp
class QueryBuilder {
public:
    // BAD: Caller cannot run q.where("x=1"); without capturing or casting
    [[nodiscard]] QueryBuilder& where(const std::string& clause);
};
```

### 2. Discard Suppression in Macro Expansions
Be cautious when wrapping `[[nodiscard]]` calls inside macros that might expand to nothing in release modes (e.g., inside `assert()`):

```cpp
[[nodiscard]] int calculate_critical_hash();

// CAVEAT: In NDEBUG builds, assert expands to ((void)0), 
// causing calculate_critical_hash() to NOT execute at all!
assert(calculate_critical_hash() != 0); 
```

### 3. Slicing with Temporary Reference Returns
If a function returns a reference (`const T&`) annotated with `[[nodiscard]]`, ignoring it triggers a warning, but saving it to a dangling reference or a sliced value requires independent vigilance.

---

## 10. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |