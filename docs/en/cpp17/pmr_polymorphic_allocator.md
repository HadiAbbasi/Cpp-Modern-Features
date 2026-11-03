

<div align="center">

[🇺🇸 English](./polymorphic_allocator.md) | [🇮🇷 فارسی](../../fa/cpp17/polymorphic_allocator.md)

</div>

----

# Polymorphic Memory Resources (PMR) & `std::pmr::polymorphic_allocator` in C++17

Custom memory allocation is one of the most powerful performance tools in high-throughput C++ applications (such as game engines, quantitative trading systems, embedded robotics, and real-time network pipelines). Prior to C++17, standard library allocators were parameterized as part of the container's static type: `std::vector<int, MyCustomAlloc>` is fundamentally an incompatible, distinct type from `std::vector<int, std::allocator<int>>`. This caused interface fragmentation, template code bloat, and viral propagation across public APIs.

C++17 resolved this architectural dilemma by introducing **Polymorphic Memory Resources (PMR)** in header `<memory_resource>`. By separating the **allocation strategy** (handled at runtime through virtual dispatch on `std::pmr::memory_resource`) from the **allocator interface** (`std::pmr::polymorphic_allocator<T>`), containers share identical types regardless of their underlying memory backing (stack, pool, monotonic arena, or heap).

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <array>
#include <memory_resource>

int main() {
    // 1. Stack buffer arena (Zero heap allocation!)
    std::array<std::byte, 1024> stack_buf;
    std::pmr::monotonic_buffer_resource mem_pool(
        stack_buf.data(), stack_buf.size(), std::pmr::null_memory_resource()
    );

    // 2. Vector using the polymorphic allocator
    std::pmr::vector<std::pmr::string> logs(&mem_pool);

    logs.emplace_back("System initialization complete.");
    logs.emplace_back("Listening on port 8080...");

    for (const auto& log : logs) {
        std::cout << "[LOG] " << log << '\n';
    }

    return 0;
}
```

---

## Table of Contents

1. [Why PMR Exists: The Static Allocator Dilemma](#1-why-pmr-exists-the-static-allocator-dilemma)
2. [What Was the Problem Before C++17?](#2-what-was-the-problem-before-c17)
3. [Architecture and Core Abstractions](#3-architecture-and-core-abstractions)
   - 3.1 [`std::pmr::memory_resource`](#31-stdpmrmemory_resource)
   - 3.2 [`std::pmr::polymorphic_allocator<T>`](#32-stdpmrpolymorphic_allocatort)
4. [Standard Concrete Memory Resources](#4-standard-concrete-memory-resources)
   - 4.1 [`std::pmr::monotonic_buffer_resource`](#41-stdpmrmonotonic_buffer_resource)
   - 4.2 [`std::pmr::synchronized_pool_resource`](#42-stdpmrsynchronized_pool_resource)
   - 4.3 [`std::pmr::unsynchronized_pool_resource`](#43-stdpmrunsynchronized_pool_resource)
   - 4.4 Global Resources: `new_delete_resource` & `null_memory_resource`
5. [Standard Container Aliases (`std::pmr::*`)](#5-standard-container-aliases-stdpmr)
6. [Nested Allocator Propagation and Transitive Memory](#6-nested-allocator-propagation-and-transitive-memory)
7. [Writing a Custom `memory_resource`](#7-writing-a-custom-memory_resource)
8. [Real-World Architectural Patterns](#8-real-world-architectural-patterns)
   - 8.1 [Per-Frame / Per-Request Scratch Arenas](#81-per-frame--per-request-scratch-arenas)
   - 8.2 [Thread-Local High-Speed Allocation](#82-thread-local-high-speed-allocation)
   - 8.3 [Hard Real-Time Latency Containment](#83-hard-real-time-latency-containment)
   - 8.4 [Memory Profiling, Tracing, and Leak Detection](#84-memory-profiling-tracing-and-leak-detection)
9. [Common Pitfalls and Gotchas](#9-common-pitfalls-and-gotchas)
10. [Performance, Inlining, and Cache Locality](#10-performance-inlining-and-cache-locality)
11. [Best Practices](#11-best-practices)
12. [Evolution: C++20 Enhancements](#12-evolution-c20-enhancements)
13. [Feature-Test Macro and Compiler Support](#13-feature-test-macro-and-compiler-support)
14. [Quick Reference Card](#14-quick-reference-card)
15. [Complete Executable Example](#15-complete-executable-example)
16. [Exercises](#16-exercises)
17. [Conclusion](#17-conclusion)
18. [Contributors](#18-contributors)

---

## 1. Why PMR Exists: The Static Allocator Dilemma

Standard C++ allocators (since C++98) were designed with compile-time customization in mind:

```cpp
template <class T, class Allocator = std::allocator<T>>
class vector;
```

While this achieved zero-overhead inlining of simple `new`/`delete` calls, it had a fatal design flaw: **allocator identity is baked into container identity**.

```cpp
using HeapVector  = std::vector<int, std::allocator<int>>;
using StackVector = std::vector<int, MyCustomStackAllocator<int>>;

// IMPOSSIBLE: Function cannot accept StackVector without template bloat
void process(const HeapVector& vec); 
```

Polymorphic Memory Resources decouple these responsibilities:
- `std::pmr::memory_resource`: An abstract base class providing dynamic memory operations through virtual functions (`do_allocate`, `do_deallocate`).
- `std::pmr::polymorphic_allocator<T>`: A thin wrapper satisfying the C++ allocator concept, delegating all operations to a pointer to `memory_resource`.
- Result: **All PMR containers of type `T` share the identical static type `std::pmr::vector<T>`**, regardless of whether their memory originates from the stack, an arena, or a locked-memory pool.

---

## 2. What Was the Problem Before C++17?

### Problem 1: Interface Pollution and ABI Fracturing

Prior to C++17, if a low-latency library used a fast custom arena allocator, every consumer API had to be templated:

```cpp
// Pre-C++17: Either force templates everywhere or force standard heap allocations
template <typename Alloc>
void parsePayload(const std::vector<uint8_t, Alloc>& bytes);
```

Header interfaces became infectious, compilation slowed, and shared libraries (`.so` / `.dll`) could not easily expose standard container boundaries with non-default allocators.

### Problem 2: Allocator Propagation in Composed Objects

In classic C++, if you had a `std::vector<std::string>`, passing a custom allocator to the outer vector did not cleanly propagate to the inner strings without manual, error-prone `std::scoped_allocator_adaptor` setup. Inner elements continued to make separate heap allocations via default `std::allocator`.

---

## 3. Architecture and Core Abstractions

PMR separates the mechanism of allocation from the type of the container:

```text
┌────────────────────────────────────────────────────────┐
│             std::pmr::vector<T>                        │
│   (Static container type identical across all scopes)  │
└───────────────────────────┬────────────────────────────┘
                            │ uses
┌───────────────────────────▼────────────────────────────┐
│      std::pmr::polymorphic_allocator<T>                │
│    (Holds pointer: memory_resource*)                   │
└───────────────────────────┬────────────────────────────┘
                            │ delegates via vtable
┌───────────────────────────▼────────────────────────────┐
│         std::pmr::memory_resource (Base)               │
│   do_allocate() / do_deallocate() / do_is_equal()      │
└───────▲───────────────────▲─────────────────────▲──────┘
        │                   │                     │
┌───────┴───────┐   ┌───────┴────────┐    ┌───────┴──────┐
│  monotonic    │   │ synchronized   │    │  custom      │
│  buffer_res   │   │ pool_resource  │    │  resource    │
└───────────────┘   └────────────────┘    └──────────────┘
```

### 3.1 `std::pmr::memory_resource`

The abstract base class defines three primary virtual methods using the Non-Virtual Interface (NVI) idiom:

```cpp
namespace std::pmr {
    class memory_resource {
    public:
        virtual ~memory_resource() = default;

        void* allocate(size_t bytes, size_t alignment = alignof(max_align_t)) {
            return do_allocate(bytes, alignment);
        }

        void deallocate(void* p, size_t bytes, size_t alignment = alignof(max_align_t)) {
            do_deallocate(p, bytes, alignment);
        }

        bool is_equal(const memory_resource& other) const noexcept {
            return do_is_equal(other);
        }

    protected:
        virtual void* do_allocate(size_t bytes, size_t alignment) = 0;
        virtual void  do_deallocate(void* p, size_t bytes, size_t alignment) = 0;
        virtual bool  do_is_equal(const memory_resource& other) const noexcept = 0;
    };
}
```

### 3.2 `std::pmr::polymorphic_allocator<T>`

`polymorphic_allocator<T>` holds a single pointer: `memory_resource* res_`.
- Default constructor points to `std::pmr::get_default_resource()`.
- Copying an allocator copies only the pointer to the resource.
- `allocate(n)` forwards to `res_->allocate(n * sizeof(T), alignof(T))`.
- It implements the scoped allocator model automatically: it passes itself down to any elements constructed within the container if they support PMR.

---

## 4. Standard Concrete Memory Resources

C++17 includes ready-to-use, highly optimized memory resource implementations:

### 4.1 `std::pmr::monotonic_buffer_resource`

Fast arena / bump allocator. Memory is allocated sequentially from a pre-allocated buffer or upstream resource.
- **Deallocations are no-ops:** Calling `deallocate` does nothing.
- **Fast bulk release:** All memory is released at once when the `monotonic_buffer_resource` destructs.
- **Zero synchronization overhead:** Unsynchronized, optimal for single-threaded frames or scratchpads.

```cpp
std::array<std::byte, 4096> buffer;
std::pmr::monotonic_buffer_resource pool(
    buffer.data(), buffer.size(), std::pmr::new_delete_resource()
);
```

### 4.2 `std::pmr::synchronized_pool_resource`

A thread-safe pool allocator (slab allocator) consisting of multiple pools tuned for different allocation sizes (chunks).
- Reduces fragmentation and overhead for frequent allocations of small, equal-sized objects.
- Protected by internal mutexes for multi-threaded safety.

### 4.3 `std::pmr::unsynchronized_pool_resource`

Identical to `synchronized_pool_resource`, but with **zero locking overhead**. Designed for thread-confined workloads.

### 4.4 Global Resources: `new_delete_resource` & `null_memory_resource`

```cpp
// Uses standard global operator new and operator delete
std::pmr::memory_resource* r1 = std::pmr::new_delete_resource();

// Always throws std::bad_alloc if allocation is attempted.
// Used as upstream fallback to guarantee no heap allocations occur!
std::pmr::memory_resource* r2 = std::pmr::null_memory_resource();

// Gets or replaces the process-wide default PMR resource
std::pmr::memory_resource* current_def = std::pmr::get_default_resource();
std::pmr::set_default_resource(r1);
```

---

## 5. Standard Container Aliases (`std::pmr::*`)

For every standard container in `<vector>`, `<string>`, `<map>`, `<unordered_map>`, etc., C++17 provides a standard alias inside the `std::pmr` namespace:

| Standard Type | PMR Equivalent Alias | Full Equivalent Definition |
|---|---|---|
| `std::string` | `std::pmr::string` | `std::basic_string<char, std::char_traits<char>, std::pmr::polymorphic_allocator<char>>` |
| `std::vector<T>` | `std::pmr::vector<T>` | `std::vector<T, std::pmr::polymorphic_allocator<T>>` |
| `std::map<K, V>` | `std::pmr::map<K, V>` | `std::map<K, V, std::less<K>, std::pmr::polymorphic_allocator<std::pair<const K, V>>>` |
| `std::unordered_map<K, V>` | `std::pmr::unordered_map<K, V>` | `std::unordered_map<K, V, ..., std::pmr::polymorphic_allocator<...>>` |

---

## 6. Nested Allocator Propagation and Transitive Memory

One of PMR's greatest features is **scoped transitive propagation**. When you put a `std::pmr::string` inside a `std::pmr::vector`, the vector automatically propagates its own `memory_resource*` into each inserted string.

```cpp
std::array<std::byte, 8192> buffer;
std::pmr::monotonic_buffer_resource arena(buffer.data(), buffer.size());

// Vector uses the arena
std::pmr::vector<std::pmr::string> list(&arena);

// Both vector dynamic array AND internal string contents reside in the arena!
list.emplace_back("A long string exceeding small string optimization (SSO) capacity!");
```

In pre-C++17 code, strings inside vectors defaulted to global `operator new`. With PMR, the entire object graph remains confined to the designated memory resource.

---

## 7. Writing a Custom `memory_resource`

Creating custom allocators requires only subclassing `std::pmr::memory_resource` and overriding its 3 protected virtual methods:

```cpp
#include <iostream>
#include <memory_resource>

class DiagnosticMemoryResource : public std::pmr::memory_resource {
public:
    explicit DiagnosticMemoryResource(std::pmr::memory_resource* upstream = std::pmr::new_delete_resource())
        : upstream_(upstream) {}

    size_t total_allocated() const noexcept { return total_allocated_; }

protected:
    void* do_allocate(size_t bytes, size_t alignment) override {
        total_allocated_ += bytes;
        std::cout << "[ALLOC] " << bytes << " bytes (align: " << alignment << ")\n";
        return upstream_->allocate(bytes, alignment);
    }

    void do_deallocate(void* p, size_t bytes, size_t alignment) override {
        std::cout << "[DEALLOC] " << bytes << " bytes\n";
        upstream_->deallocate(p, bytes, alignment);
    }

    bool do_is_equal(const std::pmr::memory_resource& other) const noexcept override {
        return this == &other;
    }

private:
    std::pmr::memory_resource* upstream_;
    size_t total_allocated_ = 0;
};
```

---

## 8. Real-World Architectural Patterns

### 8.1 Per-Frame / Per-Request Scratch Arenas

In game engines or web servers, allocate a bump allocator at the start of a frame/request and reset it at the end:

```cpp
void processRequest(const uint8_t* payload, size_t len) {
    std::array<std::byte, 64 * 1024> scratch_pad;
    // Upstream null_memory_resource ensures no accidental heap calls
    std::pmr::monotonic_buffer_resource request_mem(
        scratch_pad.data(), scratch_pad.size(), std::pmr::null_memory_resource()
    );

    std::pmr::vector<std::pmr::string> tokens(&request_mem);
    // Parse, filter, process tokens without ANY dynamic heap overhead...
} // Everything in scratch_pad is trivially abandoned/reset when leaving scope
```

### 8.2 Thread-Local High-Speed Allocation

Avoid cross-thread contention by binding unsynchronized pools to thread-local storage:

```cpp
thread_local std::array<std::byte, 1024 * 1024> tl_buffer;
thread_local std::pmr::monotonic_buffer_resource tl_arena(
    tl_buffer.data(), tl_buffer.size()
);

void performThreadTask() {
    std::pmr::vector<int> numbers(&tl_arena);
    // Runs with maximum L1/L2 cache locality and no locks
}
```

### 8.3 Hard Real-Time Latency Containment

To guarantee deterministic execution time, configure `null_memory_resource` as the upstream resource:

```cpp
std::array<std::byte, 32768> fixed_stack;
std::pmr::monotonic_buffer_resource deterministic_pool(
    fixed_stack.data(), fixed_stack.size(), std::pmr::null_memory_resource()
);

std::pmr::vector<TelemetryPacket> packets(&deterministic_pool);
// If telemetry bursts beyond 32KB, std::bad_alloc is thrown immediately
// instead of causing unpredictable kernel page faults via heap expansion!
```

### 8.4 Memory Profiling, Tracing, and Leak Detection

Wrap production resources with counting or tracking resources to inspect memory pressure during unit tests and profiling runs.

---

## 9. Common Pitfalls and Gotchas

### 9.1 Lifetime of the Memory Resource

**The most common bug:** The `memory_resource` must strictly outlive every container using it!

```cpp
std::pmr::vector<int> createVector() {
    std::array<std::byte, 512> stack_buf;
    std::pmr::monotonic_buffer_resource res(stack_buf.data(), stack_buf.size());
    
    std::pmr::vector<int> v(&res);
    v.push_back(10);
    return v; // UNDEFINED BEHAVIOR! 'v' outlives 'res' and 'stack_buf'
}
```

### 9.2 Mixing Standard String with PMR String

Assigning or converting between `std::string` and `std::pmr::string` requires an explicit copy because they are distinct types:

```cpp
std::string classic = "Hello";
// std::pmr::string pmr_str = classic; // Compile error!
std::pmr::string pmr_str(classic.begin(), classic.end(), &my_res); // OK
```

### 9.3 Move Semantics Across Different Resources

When moving a PMR container into another, if both containers have **different** memory resources (`!alloc1.resource()->is_equal(*alloc2.resource())`), the container cannot simply steal pointer ownership. It must **copy** individual elements into the destination's memory resource!

```cpp
std::pmr::monotonic_buffer_resource resA;
std::pmr::monotonic_buffer_resource resB;

std::pmr::vector<int> v1(&resA);
v1.assign({1, 2, 3, 4, 5});

std::pmr::vector<int> v2(&resB);
v2 = std::move(v1); // Performs ELEMENT-BY-ELEMENT COPY, NOT an O(1) pointer swap!
```

---

## 10. Performance, Inlining, and Cache Locality

| Metric | Classic `std::allocator` | `std::pmr::monotonic_buffer_resource` | `std::pmr::unsynchronized_pool_resource` |
|---|---|---|---|
| **Allocation Cost** | Kernel syscall / malloc logic ($\approx 30\text{–}100\text{ ns}$) | Pointer bump ($\approx 1\text{–}3\text{ ns}$) | Linked pool pop ($\approx 5\text{–}10\text{ ns}$) |
| **Deallocation Cost** | Heap free list bookkeeping | **0 ns** (noop) | Linked pool push |
| **Virtual Dispatch** | None | 1 indirect call per buffer growth | 1 indirect call per buffer growth |
| **Cache Locality** | Random/fragmented across heap | Contiguous / high L1/L2 cache hits | High for matching block size bins |

*Note on Virtual Call Overhead:* The virtual call in `std::pmr` happens **only when the container reallocates its dynamic buffer** (e.g. geometric resizing in vector), not on every read/write or element lookup. The cost of a single virtual call during buffer reallocation is negligible compared to a heap system call.

---

## 11. Best Practices

1. **Use `std::pmr` container aliases:** Always write `std::pmr::vector<T>` instead of `std::vector<T, std::pmr::polymorphic_allocator<T>>`.
2. **Chain upstream resources:** Always give temporary resources (like `monotonic_buffer_resource`) a meaningful upstream resource (either `new_delete_resource()` or `null_memory_resource()`).
3. **Guard lifetimes meticulously:** Ensure memory pools outlive all consumers. For functions returning data, pass the target allocator in as an argument.
4. **Use `null_memory_resource` for embedded/safety-critical code:** To guarantee zero dynamic allocation, use `null_memory_resource()` as your upstream handler.
5. **Be aware of move costs across resources:** Moving between containers with distinct memory resources results in deep copies.

---

## 12. Evolution: C++20 Enhancements

C++20 extended and simplified PMR usage:
- **`std::pmr::polymorphic_allocator<std::byte>` as default:** `polymorphic_allocator<>` can now be used without a template parameter (defaults to `std::byte`).
- **`allocate_object`, `deallocate_object`, `new_object`, `delete_object`:** Direct type-safe object creation helper methods were added to `polymorphic_allocator`:
  ```cpp
  // C++20: Directly construct an object in PMR memory
  auto alloc = std::pmr::polymorphic_allocator<>{&my_resource};
  MyType* obj = alloc.new_object<MyType>(arg1, arg2);
  alloc.delete_object(obj);
  ```

---

## 13. Feature-Test Macro and Compiler Support

Header: `<memory_resource>`

```cpp
#if defined(__cpp_lib_memory_resource) && __cpp_lib_memory_resource >= 201603L
    // Full PMR support available
#endif
```

### Compiler Support Table

| Compiler | Version | Standard Flag | Notes |
|---|---|---|---|
| **GCC** | 9.1+ | `-std=c++17` | Full libstdc++ PMR support |
| **Clang** | 16.0+ | `-std=c++17` | Full libc++ support (`<memory_resource>`) |
| **MSVC** | 2017 (15.8+) | `/std:c++17` | Native MSVC STL support |
| **Apple Clang** | 15.0+ | `-std=c++17` | Uses updated libc++ |

---

## 14. Quick Reference Card

```cpp
#include <memory_resource>
#include <vector>
#include <string>

// 1. Stack Arena Setup
std::array<std::byte, 2048> stack_mem;
std::pmr::monotonic_buffer_resource mem_arena(stack_mem.data(), stack_mem.size());

// 2. Container Instantiation
std::pmr::vector<int> v(&mem_arena);
std::pmr::string s("High performance PMR string", &mem_arena);

// 3. Pool Allocators (Thread-confined)
std::pmr::unsynchronized_pool_resource pool;
std::pmr::vector<double> v_pooled(&pool);

// 4. Guaranteed Non-Allocating Boundary
std::pmr::monotonic_buffer_resource strict_arena(
    stack_mem.data(), stack_mem.size(), std::pmr::null_memory_resource()
);
```

---

## 15. Complete Executable Example

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <array>
#include <chrono>
#include <memory_resource>

// Custom tracking resource to monitor allocations
class TrackingResource : public std::pmr::memory_resource {
public:
    explicit TrackingResource(std::pmr::memory_resource* upstream)
        : upstream_(upstream) {}

    size_t bytes_allocated() const noexcept { return bytes_allocated_; }
    size_t allocation_count() const noexcept { return allocation_count_; }

protected:
    void* do_allocate(size_t bytes, size_t alignment) override {
        bytes_allocated_ += bytes;
        ++allocation_count_;
        return upstream_->allocate(bytes, alignment);
    }

    void do_deallocate(void* p, size_t bytes, size_t alignment) override {
        upstream_->deallocate(p, bytes, alignment);
    }

    bool do_is_equal(const std::pmr::memory_resource& other) const noexcept override {
        return this == &other;
    }

private:
    std::pmr::memory_resource* upstream_;
    size_t bytes_allocated_ = 0;
    size_t allocation_count_ = 0;
};

void run_benchmark() {
    std::cout << "=== Demonstrating std::pmr::polymorphic_allocator in C++17 ===\n\n";

    TrackingResource tracker(std::pmr::new_delete_resource());

    std::cout << "--- 1. Monotonic Stack Buffer Arena ---\n";
    {
        std::array<std::byte, 4096> stack_buffer;
        
        // Pass tracker as upstream to catch overflow beyond 4KB
        std::pmr::monotonic_buffer_resource arena(
            stack_buffer.data(), stack_buffer.size(), &tracker
        );

        // All vectors share standard type: std::pmr::vector
        std::pmr::vector<std::pmr::string> names(&arena);

        names.emplace_back("Alpha Node");
        names.emplace_back("Beta Transponder");
        names.emplace_back("Gamma Sensor Array Exceeding SSO Buffer Size");

        std::cout << "Entries stored in stack-backed PMR vector:\n";
        for (const auto& item : names) {
            std::cout << "  - " << item << '\n';
        }

        std::cout << "Heap fallback allocation count: " << tracker.allocation_count() << '\n';
        std::cout << "Heap fallback bytes allocated: " << tracker.bytes_allocated() << '\n';
    }

    std::cout << "\n--- 2. Transparent API (Uniform Container Type) ---\n";
    auto inspect_container = [](const std::pmr::vector<int>& vec, const char* label) {
        std::cout << label << " size: " << vec.size() 
                  << " | Elements: ";
        for (int x : vec) std::cout << x << ' ';
        std::cout << '\n';
    };

    // Vector A backed by heap
    std::pmr::vector<int> heap_vec(std::pmr::new_delete_resource());
    heap_vec.assign({10, 20, 30});

    // Vector B backed by local pool
    std::pmr::unsynchronized_pool_resource pool;
    std::pmr::vector<int> pool_vec(&pool);
    pool_vec.assign({100, 200, 300});

    // Both pass into the EXACT same non-templated function signature!
    inspect_container(heap_vec, "Heap-backed vector");
    inspect_container(pool_vec, "Pool-backed vector");
}

int main() {
    run_benchmark();
    return 0;
}
```

### Expected Output

```text
=== Demonstrating std::pmr::polymorphic_allocator in C++17 ===

--- 1. Monotonic Stack Buffer Arena ---
Entries stored in stack-backed PMR vector:
  - Alpha Node
  - Beta Transponder
  - Gamma Sensor Array Exceeding SSO Buffer Size
Heap fallback allocation count: 0
Heap fallback bytes allocated: 0

--- 2. Transparent API (Uniform Container Type) ---
Heap-backed vector size: 3 | Elements: 10 20 30 
Pool-backed vector size: 3 | Elements: 100 200 300 
```

---

## 16. Exercises

### Exercise 1 — Strict Zero-Allocation Arena
Set up a `std::pmr::monotonic_buffer_resource` using a 512-byte stack array backed by `std::pmr::null_memory_resource()`. Push elements into a `std::pmr::vector<std::pmr::string>` until it exceeds the buffer. Catch and verify the resulting `std::bad_alloc` exception.

### Exercise 2 — ABI Container Interoperability
Write a function `void exportData(const std::pmr::vector<std::pmr::string>& data)` that accepts any vector without knowing its memory layout. In `main()`, instantiate one vector with a stack arena and another with a `synchronized_pool_resource`. Verify that both can be passed directly to `exportData` without template instantiation.

### Exercise 3 — Custom Block-Tracking Memory Resource
Write a custom `memory_resource` that logs every allocation and deallocation with pointer addresses, requested sizes, and alignments. Verify that nested containers (e.g. `std::pmr::vector<std::pmr::string>`) correctly allocate both vector capacity and string memory through your resource.

### Exercise 4 — Cross-Resource Move Evaluation
Construct two `std::pmr::vector<int>` objects with two separate `std::pmr::monotonic_buffer_resource` instances. Insert elements into the first. Use `std::move` to assign the first to the second. Print the address of the underlying data pointers before and after the move to verify whether pointer stealing occurred or elements were copied.

---

## 17. Conclusion

`std::pmr::polymorphic_allocator` and Polymorphic Memory Resources represent one of the most important architectural improvements in modern C++:
- They **eliminate the static allocator type epidemic**, unifying container types across library and translation boundaries.
- They provide **blazing-fast scratchpad memory allocation** through ready-made monotonic and pool resources.
- They ensure **scoped transitive allocator propagation** across nested types like strings and nested vectors.
- They maintain **C++'s zero-cost abstraction philosophy**, adding virtual dispatch only on rare dynamic buffer growths while leaving element access at full speed.

---

## 18. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |[Ordikhani](ordikhanifateme@gmail.com) |  | [Ordikhani](@OrdikhaniFateme) |