
<div align="center">

[🇺🇸 English](./weak_from_this.md) | [🇮🇷 فارسی](../../fa/cpp17/weak_from_this.md)

</div>

----

# `std::weak_from_this` in C++17

Introduced in **C++17** via proposal **P0033R1** ("Re-enabling `shared_from_this`"), `std::weak_from_this` is a non-throwing member function added to the class template `std::enable_shared_from_this<T>`. It allows an instance to obtain a non-owning `std::weak_ptr<T>` (or `std::weak_ptr<const T>`) observing its own lifetime without throwing exceptions and without inadvertently extending its lifetime.

Prior to C++17, an object attempting to produce a `std::weak_ptr` to itself had to resort to `std::weak_ptr<T>(shared_from_this())`, which was both computationally wasteful (atomic refcount increment followed immediately by a decrement) and exceptionally hazardous: calling `shared_from_this()` when no owning `std::shared_ptr` existed unconditionally threw `std::bad_weak_ptr`.

```cpp
#include <memory>
#include <iostream>
#include <functional>

class EventBusNode : public std::enable_shared_from_this<EventBusNode> {
public:
    // C++17: Safe, zero-refcount-increment non-owning handle generation
    [[nodiscard]] std::weak_ptr<EventBusNode> get_weak_handle() noexcept {
        return weak_from_this();
    }

    [[nodiscard]] std::weak_ptr<const EventBusNode> get_weak_handle() const noexcept {
        return weak_from_this();
    }
};
```

---

## Table of Contents

1. [Historical Context & Motivation: The P0033R1 Proposal](#1-historical-context--motivation-the-p0033r1-proposal)
2. [Internal Mechanics: The Control Block and Weak Pointer Layout](#2-internal-mechanics-the-control-block-and-weak-pointer-layout)
3. [The Atomic Cost: `weak_from_this()` vs `std::weak_ptr(shared_from_this())`](#3-the-atomic-cost-weak_from_this-vs-stdweak_ptrshared_from_this)
4. [Exception Safety Guarantees & Lifecycle Boundary Conditions](#4-exception-safety-guarantees--lifecycle-boundary-conditions)
5. [Real-World Design Patterns](#5-real-world-design-patterns)
   - [Pattern A: Asynchronous I/O Watchdogs & Weak Lambdas](#pattern-a-asynchronous-io-watchdogs--weak-lambdas)
   - [Pattern B: Cache Invalidation Without Cyclic Retain Loops](#pattern-b-cache-invalidation-without-cyclic-retain-loops)
6. [Const-Correctness and Signature Overloads](#6-const-correctness-and-signature-overloads)
7. [Comparison Matrix](#7-comparison-matrix)
8. [Feature-Test Macro](#8-feature-test-macro)
9. [Pitfalls, Undefined Behaviors, and Edge Cases](#9-pitfalls-undefined-behaviors-and-edge-cases)
10. [Contributors](#10-contributors)

---

## 1. Historical Context & Motivation: The P0033R1 Proposal

In C++11 and C++14, `std::enable_shared_from_this<T>` provided only `shared_from_this()`. Under the hood, however, `std::enable_shared_from_this<T>` was implemented by storing a private mutable `std::weak_ptr<T>` inside the base class subobject.

To export a `std::weak_ptr` to the outside world, developers were forced to write:
```cpp
// Pre-C++17 Workaround:
std::weak_ptr<T> w = this->shared_from_this();
```
This idiom possessed three major architectural flaws:
1. **Unnecessary Atomic Overhead**: It invoked `shared_from_this()`, which atomized an increment on `use_count` (strong reference counter), constructed a temporary `std::shared_ptr`, constructed the `std::weak_ptr` (atomically incrementing the weak reference counter), and then destroyed the temporary `std::shared_ptr` (atomically decrementing `use_count`). On multi-socket x86 and ARM platforms, two atomic read-modify-write operations on contested cache lines represent substantial microarchitectural latency.
2. **Mandatory Exception Handling**: If the object was constructed on the stack, within an aggregate, or was during constructor execution, `shared_from_this()` threw `std::bad_weak_ptr`. The developer had to wrap the code in a `try...catch` block simply to test whether an owning instance existed.
3. **Ambiguity During Destruction**: In destructors, calling `shared_from_this()` is undefined behavior or throws `std::bad_weak_ptr`, making teardown notification in complex object graphs exceedingly brittle.

Proposal **P0033R1** eliminated these issues by directly exposing a read-only copy of the internal `weak_ptr` already living inside the base class.

---

## 2. Internal Mechanics: The Control Block and Weak Pointer Layout

To understand why `weak_from_this()` is so efficient, consider the physical memory layout of a standard control block and smart pointers:

```text
                  +---------------------------------------+
                  |            Control Block              |
                  +---------------------------------------+
                  | strong_count (use_count) : atomic<32> |
                  | weak_count               : atomic<32> |
                  | allocator / deleter pointers          |
                  +---------------------------------------+
                                     ^
                                     |
+--------------------------+         |
| std::weak_ptr<T>         |---------+
|   T*         _M_ptr      |---------> [ Managed Object Subobject ]
|   _Sp_counted_base* _M_pi|           [ (Derived : enable_shared) ]
+--------------------------+           [                           ]
                                       [ std::weak_ptr<T> _M_weak  ] ---+
                                       +---------------------------+    |
                                                     ^                  |
                                                     +------------------+
                                                 (points back to control block)
```

1. When an object derived from `std::enable_shared_from_this<T>` is passed to `std::make_shared<T>()` or `std::shared_ptr<T>(new T())`, the runtime initializes the internal private member (e.g., `_M_weak_this` in libstdc++/libc++).
2. The internal `_M_weak_this` observes the exact same control block that manages the object.
3. Calling `weak_from_this()` simply invokes the copy constructor of `std::weak_ptr`, which:
   - Copies the raw pointer `_M_ptr`.
   - Atomically increments **only** the `weak_count` field in the control block.
   - **Never touches the strong reference counter (`use_count`)**.

---

## 3. The Atomic Cost: `weak_from_this()` vs `std::weak_ptr(shared_from_this())`

The performance difference can be seen directly in the generated assembly on modern architectures:

### Pre-C++17 Workaround:
```cpp
// 1. Atomic increment strong_count (e.g., LOCK XADD)
// 2. Memory barrier / acquire semantics
// 3. Atomic increment weak_count (LOCK INC)
// 4. Atomic decrement strong_count (LOCK XADD)
// 5. Release / acquire fence check if strong_count reached 0
std::weak_ptr<T> w = shared_from_this();
```

### C++17 `weak_from_this()`:
```cpp
// 1. Atomic increment weak_count (LOCK INC on x86, LDREX/STREX or LDADD on ARM64)
// Done!
std::weak_ptr<T> w = weak_from_this();
```

In high-throughput event dispatchers, network multiplexers, or game engine scene graphs where handles are frequently passed to job queues, this optimization completely bypasses ping-ponging the cache line containing `use_count`.

---

## 4. Exception Safety Guarantees & Lifecycle Boundary Conditions

`std::weak_from_this` is declared **`noexcept`**:

```cpp
std::weak_ptr<T> weak_from_this() noexcept;
std::weak_ptr<const T> weak_from_this() const noexcept;
```

### Lifecycle Boundaries:

| State of Object | `shared_from_this()` | `weak_from_this()` |
|:---|:---|:---|
| **Allocated on Stack or Raw Heap (`new T`)** | **Throws** `std::bad_weak_ptr` | **Returns empty** `weak_ptr` (expired) |
| **Inside Constructor (`T::T()`)** | **Throws** `std::bad_weak_ptr` | **Returns empty** `weak_ptr` (expired) |
| **Fully constructed & owned by `shared_ptr`** | Returns valid `std::shared_ptr<T>` | Returns valid `std::weak_ptr<T>` |
| **Inside Destructor (`T::~T()`)** | **Throws** or **UB** (`use_count == 0`) | **Returns expired** `std::weak_ptr<T>` |

Because `weak_from_this()` never throws, it allows for branch-based validation rather than expensive exception handling:

```cpp
void Session:::try_register(Dispatcher&(SessionData& d) noexcept {
    auto weak = weak_from_this();
    if (!weak.expired()) {
        d.register_session(std::move(weak));
    } else {
        // Handle unmanaged/stack-allocated scenario cleanly without try/catch
    }
}
```

---

## 5. Real-World Design Patterns

### Pattern A: Asynchronous I/O Watchdogs & Weak Lambdas

When capturing an instance pointer inside an asynchronous callback (such as Boost.ASIO, libuv, or a thread pool), capturing `shared_from_this()` creates a **cyclic or extended lifetime bug**—the object will never be destroyed if the callback remains enqueued indefinitely. 

Capturing `weak_from_this()` solves this elegantly:

```cpp
#include <iostream>
#include <memory>
#include <thread>
#include <chrono>

class AsyncWorker : public std::enable_shared_from_this<AsyncWorker> {
public:
    void trigger_background_task() {
        // Capture weak pointer inside lambda via C++17 weak_from_this()
        std::thread([weak = weak_from_this()]() {
            std::this_thread::sleep_for(std::chrono::milliseconds(200));

            // Upgrade weak_ptr to shared_ptr to execute critical section
            if (auto self = weak.lock()) {
                self->on_task_completed();
            } else {
                std::cout << "AsyncWorker destroyed before task completed; gracefully aborting.\n";
            }
        }).detach();
    }

private:
    void on_task_completed() {
        std::cout << "AsyncWorker processing result on address: " << this << "\n";
    }
};

int main() {
    {
        auto worker = std::make_shared<AsyncWorker>();
        worker->trigger_background_task();
        // worker scope ends immediately here
    }
    // Give async task time to expire
    std::this_thread::sleep_for(std::chrono::milliseconds(300));
    return 0;
}
```

---

### Pattern B: Cache Invalidation Without Cyclic Retain Loops

In global or subsystem registries (e.g., texture managers, session brokers), keeping strong pointers in the cache prevents memory reclamation:

```cpp
#include <memory>
#include <unordered_map>
#include <string>

class Resource;

class ResourceCache {
public:
    void track(const std::string& id, std::weak_ptr<Resource> handle) {
        m_registry[id] = std::move(handle);
    }

    std::shared_ptr<Resource> fetch(const std::string& id) {
        auto it = m_registry.find(id);
        if (it != m_registry.end()) {
            return it->second.lock(); // Returns nullptr if already reclaimed
        }
        return nullptr;
    }

private:
    std::unordered_map<std::string, std::weak_ptr<Resource>> m_registry;
};

class Resource : public std::enable_shared_from_this<Resource> {
public:
    Resource(std::string name, ResourceCache& cache) 
        : m_name(std::move(name)), m_cache(cache) {}

    void register_self() {
        // No risk of retain cycle: Cache only holds a weak reference!
        m_cache.track(m_name, weak_from_this());
    }

private:
    std::string m_name;
    ResourceCache& m_cache;
};
```

---

## 6. Const-Correctness and Signature Overloads

`std::enable_shared_from_this` defines two precise overloads in C++17:

```cpp
template<class T>
class enable_shared_from_this {
public:
    std::shared_ptr<T>        shared_from_this();
    std::shared_ptr<const T>  shared_from_this() const;

    // Added in C++17:
    std::weak_ptr<T>          weak_from_this() noexcept;
    std::weak_ptr<const T>    weak_from_this() const noexcept;
};
```

This ensures that inspecting an immutable or read-only object via `const weak_from_this()` correctly yields a `std::weak_ptr<const T>`, preventing accidental mutability through downstream `lock()` calls.

---

## 7. Comparison Matrix

| Attribute | `shared_from_this()` | `weak_from_this()` |
|:---|:---|:---|
| **Standard** | C++11 | **C++17 (P0033R1)** |
| **Signature** | `shared_ptr<T>()` / `shared_ptr<const T>() const` | `weak_ptr<T>() noexcept` / `weak_ptr<const T>() const noexcept` |
| **Exception Safety** | Can throw `std::bad_weak_ptr` | Guaranteed `noexcept` |
| **Performance Overhead** | Increments & decrements atomic strong count | Increments only atomic weak count |
| **Safety in Constructors** | UB / Throws exception | Returns empty `weak_ptr` |
| **Safety on Raw Stack** | Throws exception | Returns empty `weak_ptr` |
| **Intended Role** | Shared Ownership Propagation | Observation, Callbacks, Caches, Cycle-Breaking |

---

## 8. Feature-Test Macro

Support for `std::weak_from_this` is coupled with general standard library enhancements for smart pointers. While P0033R1 formally upgraded the type, you can verify compiler support using the library feature macro or standard C++17 language checks:

```cpp
#include <memory>

#if defined(__cpp_lib_enable_shared_from_this) && __cpp_lib_enable_shared_from_this >= 201603L
    // Explicit weak_from_this capability guaranteed
#elif __cplusplus >= 201703L
    // General C++17 environment
#endif
```

---

## 9. Pitfalls, Undefined Behaviors, and Edge Cases

### 1. The Multiple Inheritance & Slicing Trap
If a derived class inherits multiple times from `std::enable_shared_from_this` (either directly or via multiple diamond bases), calls to `weak_from_this()` become ambiguous and fail at compile time:

```cpp
struct BaseA : std::enable_shared_from_this<BaseA> {};
struct BaseB : std::enable_shared_from_this<BaseB> {};

struct Derived : BaseA, BaseB {
    void fail() {
        // COMPILE ERROR: Ambiguous call to weak_from_this()
        // auto w = weak_from_this(); 
        
        // Must explicitly disambiguate base subobject:
        auto wA = BaseA::weak_from_this();
    }
};
```

### 2. Private Inheritance Breaks Internal Initialization
`enable_shared_from_this` **must be inherited publicly**:
```cpp
// INCORRECT:
class BadNode : std::enable_shared_from_this<BadNode> { /* ... */ };

// CORRECT:
class GoodNode : public std::enable_shared_from_this<GoodNode> { /* ... */ };
```
If private or protected inheritance is used, `std::shared_ptr` constructors cannot access the base class's internal weak pointer hooks via ADL/template deduction, leaving the internal weak pointer **uninitialized forever**. Any subsequent call to `weak_from_this()` will return an empty pointer!

### 3. Calling inside Constructor
Inside a constructor, the `std::shared_ptr` that will eventually own the object has **not yet completed its own construction**. Therefore, the internal `weak_ptr` is not yet bound:
```cpp
class Widget : public std::enable_shared_from_this<Widget> {
public:
    Widget() {
        auto w = weak_from_this();
        assert(w.expired()); // ALWAYS TRUE inside constructor!
    }
};
```
*Fix:* Defer registration logic to a dedicated post-construction factory method (e.g., `create()`).

### 4. Memory Retention by Weak References
Even if an object is destroyed (`use_count == 0`), the **control block itself is not freed from memory until `weak_count == 0`**. If thousands of `weak_ptr` instances returned by `weak_from_this()` are kept alive across long-lived containers, the control block memory (and any contiguous memory allocated via `std::make_shared`) will remain occupied.

---

## 10. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |
```