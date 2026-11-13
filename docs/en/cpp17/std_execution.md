<div align="center">

[🇺🇸 English](./execution.md) | [🇮🇷 فارسی](../../fa/cpp17/execution.md)

</div>

----

# `std::execution` in C++17

The header `<execution>` was introduced in C++17 to establish the foundational policy taxonomy for parallel and vectorized standard algorithms. Rather than creating distinct function signatures for every algorithm variation (e.g., `std::sort_par` or `std::transform_simd`), the ISO C++ standard library adopted a policy-based architecture.

By passing an **execution policy** from the `std::execution` namespace as the first argument to standard library algorithms, developers explicitly specify concurrency constraints, vectorization safety, and thread-scheduling models.

```cpp
#include <vector>
#include <algorithm>
#include <execution>

std::vector<int> data(5'000'000);

// Dispatches algorithm execution across multi-core workers and SIMD pipelines
std::sort(std::execution::par_unseq, data.begin(), data.end());
```

---

## Table of Contents

1. [Architectural Role of `<execution>`](#1-architectural-role-of-execution)
2. [Standard Execution Policy Types and Instances](#2-standard-execution-policy-types-and-instances)
3. [Formal Concurrency and Ordering Semantics](#3-formal-concurrency-and-ordering-semantics)
4. [Vectorization-Unsafe Code and Vector-Deadlocks](#4-vectorization-unsafe-code-and-vector-deadlocks)
5. [Type Traits and Concept Support](#5-type-traits-and-concept-support)
6. [Implementation-Defined and Custom Policies](#6-implementation-defined-and-custom-policies)
7. [Evolution Across Standards (C++17, C++20, and C++26)](#7-evolution-across-standards-c17-c20-and-c26)
8. [Feature-Test Macros and Toolchain Integration](#8-feature-test-macros-and-toolchain-integration)
9. [Quick Reference Summary](#9-quick-reference-summary)
10. [Pitfalls and Gotchas](#10-pitfalls-and-gotchas)
11. [Conclusion](#11-conclusion)
12. [Contributors](#12-contributors)

---

## 1. Architectural Role of `<execution>`

The `<execution>` header does not contain algorithm implementations; it establishes the formal types and semantic rules that guide parallel execution overloads across `<algorithm>` and `<numeric>`.

When an algorithm taking an execution policy is called:
1. **Policy Tag Dispatch:** The compiler matches overloads of the form:
   ```cpp
   template<class ExecutionPolicy, class ForwardIt>
   void for_each(ExecutionPolicy&& exec, ForwardIt first, ForwardIt last, ...);
   ```
2. **Execution Contract:** The policy serves as a developer-certified contract. It promises the runtime that user-supplied invocables (lambdas, functors) conform to specific thread-safety and reentrancy guarantees.
3. **Hardware Dispatch:** The runtime maps the work across worker pools, hardware threads, or SIMD execution lanes depending on the policy tag and backend runtime.

---

## 2. Standard Execution Policy Types and Instances

In C++17, `<execution>` defines three standard policy classes and three corresponding global constexpr instances:

```cpp
namespace std {
namespace execution {
    // 1. Sequenced Policy
    class sequenced_policy;
    inline constexpr sequenced_policy seq{};

    // 2. Parallel Policy
    class parallel_policy;
    inline constexpr parallel_policy par{};

    // 3. Parallel and Unsequenced Policy
    class parallel_unsequenced_policy;
    inline constexpr parallel_unsequenced_policy par_unseq{};
}
}
```

### Policy Taxonomy

| Policy Class | Global Object | Concurrency Model | Target Architecture |
|---|---|---|---|
| `std::execution::sequenced_policy` | `std::execution::seq` | Strictly sequential on caller thread | Single-thread CPU |
| `std::execution::parallel_policy` | `std::execution::par` | Multi-threaded work distribution | Multi-core CPU worker pool |
| `std::execution::parallel_unsequenced_policy` | `std::execution::par_unseq` | Multi-threaded + SIMD vectorization | Multi-core CPU + Vector units (AVX, NEON) |

---

## 3. Formal Concurrency and Ordering Semantics

The ISO standard formally characterizes each execution policy by specifying how elemental steps (*element access functions*) may be interleaved:

### 1. `std::execution::seq`
- The invocations of element access functions are **sequenced** on the calling thread.
- Guarantees strict left-to-right evaluation order consistent with traditional sequential program execution.
- No data races are possible from algorithm execution itself.

### 2. `std::execution::par`
- The invocations of element access functions are **indeterminately sequenced** across multiple threads.
- Invocations within a single thread remain sequenced with respect to each other.
- **Contract:** Invocables must not modify shared memory without synchronization. Traditional synchronization primitives (`std::mutex`, `std::atomic`) are valid, though lock contention degrades throughput.

### 3. `std::execution::par_unseq`
- The invocations of element access functions are **unsequenced** with respect to each other and may execute across arbitrary threads.
- Elemental steps may be interleaved on the *same* physical thread (e.g., vectorized execution via SIMD lanes).
- **Contract:** Invocables must be **vectorization-safe**. Operations cannot rely on completion of previous instructions before the next instruction starts on the same thread.

---

## 4. Vectorization-Unsafe Code and Vector-Deadlocks

A critical distinction between `std::execution::par` and `std::execution::par_unseq` is that `par_unseq` allows elemental steps to execute concurrently on the same thread of execution. Consequently, code inside a `par_unseq` algorithm must adhere to strict reentrancy constraints.

### Prohibited Operations in `par_unseq`:
1. **Mutual Exclusion Primitives (`std::mutex`, `std::shared_mutex`):**
   If thread $T_1$ acquires a mutex during one elemental step and is interrupted by another SIMD lane on the same thread that attempts to acquire the same mutex, a **vector-deadlock** occurs.
2. **Dynamic Memory Allocation (`malloc`, `new`, `free`, `delete`):**
   Standard heap allocators acquire internal thread-synchronization locks that are not vector-safe.
3. **Non-Lock-Free Atomics:**
   Only atomics where `std::atomic<T>::is_lock_free()` returns `true` are legal.
4. **Thread-Local Storage Access (`thread_local`):**
   State may not persist deterministically across interleaved SIMD operations.

```cpp
#include <vector>
#include <algorithm>
#include <execution>
#include <mutex>

std::mutex mtx;

void unsafe_vector_operation(std::vector<int>& vec) {
    // UNDEFINED BEHAVIOR: Mutex locking inside par_unseq causes deadlock
    std::for_each(std::execution::par_unseq, vec.begin(), vec.end(), [&](int& n) {
        std::lock_guard<std::mutex> lock(mtx); // Deadlock hazard!
        n *= 2;
    });
}
```

---

## 5. Type Traits and Concept Support

The `<execution>` header provides meta-programming traits to inspect whether a type qualifies as a standard or implementation-defined execution policy:

### `std::is_execution_policy<T>`
Checks whether `T` is an execution policy type.

```cpp
#include <execution>
#include <type_traits>

static_assert(std::is_execution_policy<std::execution::parallel_policy>::value);
static_assert(std::is_execution_policy_v<std::execution::sequenced_policy>);
static_assert(std::is_execution_policy_v<std::execution::parallel_unsequenced_policy>);
static_assert(!std::is_execution_policy_v<int>);
```

### Writing Generic Policy Overloads
Library authors use `std::is_execution_policy_v` with SFINAE or C++20 concepts to implement custom policy-aware algorithms:

```cpp
#include <execution>
#include <type_traits>

template <typename Policy, typename Iter,
          std::enable_if_t<std::is_execution_policy_v<std::decay_t<Policy>>, int> = 0>
void parallel_pipeline(Policy&& policy, Iter first, Iter last) {
    // Policy-aware generic routing
}
```

---

## 6. Implementation-Defined and Custom Policies

The standard permits vendors and hardware architectures to extend `std::execution` with vendor-specific policies:
- **SYCL / DPC++:** Exposes device execution policies.
- **NVIDIA thrust / CUDA:** Provides device execution spaces.
- **HPX:** Provides asynchronous and fiber-based execution policies.

Any standard-conforming execution policy implementation must specialize `std::is_execution_policy<T>` to yield `true_type`.

---

## 7. Evolution Across Standards (C++17, C++20, and C++26)

```
┌────────────────────────────────────────────────────────────────────────┐
│ C++17: Initial Parallel Algorithms Policy Taxonomy                     │
│        • std::execution::seq, par, par_unseq                           │
├────────────────────────────────────────────────────────────────────────┤
│ C++20: Vectorized Single-Threaded Execution (P1001R2)                  │
│        • std::execution::unseq                                         │
├────────────────────────────────────────────────────────────────────────┤
│ C++26: Senders/Receivers Asynchronous Execution Framework (P2300)      │
│        • Expands std::execution to include generic async pipelines     │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Feature-Test Macros and Toolchain Integration

The execution policy mechanism is governed by standard feature-test macros. Note that the `<version>` header is a C++20 feature; in C++17, you must include the specific feature header (e.g., `<execution>` or `<algorithm>`) directly to check these macros.

```cpp
#include <execution> // Include feature header directly in C++17

#if defined(__cpp_lib_execution) && __cpp_lib_execution >= 201603L
    // <execution> policy tags and type traits available
#endif

#if defined(__cpp_lib_parallel_algorithm) && __cpp_lib_parallel_algorithm >= 201603L
    // Parallel algorithm overloads are implemented
#endif
```

### Compiler and Runtime Requirements

| Compiler / Standard Library | Backend Engine | Compilation & Linker Flags |
|---|---|---|
| **GCC 9+ (libstdc++)** | Intel oneTBB | `-std=c++17 -ltbb` |
| **Clang 9+ (libc++)** | Intel oneTBB | `-std=c++17 -ltbb` |
| **MSVC 2017+ (MSVC STL)** | Windows ConcRT | `/std:c++17` (native) |

---

## 9. Quick Reference Summary

| Execution Policy | Multithreaded | Vectorized | Mutex Synchronization | Exception Model |
|---|---|---|---|---|
| `std::execution::seq` | No | No | Yes | `std::terminate()` / `std::bad_alloc` |
| `std::execution::par` | Yes | No | Yes | `std::terminate()` / `std::bad_alloc` |
| `std::execution::par_unseq` | Yes | Yes | **No (Deadlock)** | `std::terminate()` / `std::bad_alloc` |

---

## 10. Pitfalls and Gotchas

1. **Exception Safety:** If the user-provided code (e.g., lambda/functor) throws an exception, `std::terminate()` is invoked immediately, regardless of the execution policy used. If the algorithm itself fails to allocate internal resources, the implementation may propagate `std::bad_alloc` out of the algorithm call. This is the only standard-compliant way for an exception to escape the parallel algorithm interface.
2. **Dynamic Allocations in `par_unseq`:** Allocating memory inside `par_unseq` risks deadlocks due to non-reentrant heap locks.
3. **Lock Contention:** Using `std::mutex` inside parallel algorithms often causes severe serialization, negating performance benefits.
4. **Linker Errors:** Failure to link the backend (e.g., `-ltbb` on Linux) results in unresolved symbols (`tbb::task_arena`).

---

## 11. Conclusion

The `<execution>` header in C++17 formalized policy-driven parallelism in the C++ Standard Library. By decoupling algorithm interfaces from execution mechanics, C++ achieves cross-platform parallel and vectorized acceleration while maintaining portable, modern syntax.

---

## 12. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |