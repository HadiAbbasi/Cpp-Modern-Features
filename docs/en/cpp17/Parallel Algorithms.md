<div align="center">

[🇺🇸 English](./parallel_algorithms.md) | [🇮🇷 فارسی](../../fa/cpp17/parallel_algorithms.md)

</div>

----

# std::Parallel Algorithms in C++17

Prior to C++17, the C++ Standard Library algorithms were strictly sequential. Developers needing parallelization were forced to rely on platform-specific APIs (pthreads, Win32 threads), heavy frameworks (Intel TBB, OpenMP), or external libraries like Boost.Compute.

C++17 introduced **Parallel Algorithms** via the `<algorithm>` and `<numeric>` headers, supported by execution policies defined in the `<execution>` header. This allows developers to transparently parallelize standard algorithms by passing an "execution policy" as the first argument, abstracting away thread management, load balancing, and hardware-specific vectorization.

```cpp
#include <vector>
#include <algorithm>
#include <execution>

void parallel_sort(std::vector<int>& v) {
    // Transparently parallelize sort using multithreading
    std::sort(std::execution::par, v.begin(), v.end());
}
```

---

## Table of Contents

1. [Introduction to Execution Policies](#1-introduction-to-execution-policies)
2. [The Standard Execution Policies (C++17)](#2-the-standard-execution-policies-c17)
3. [Algorithm Iterator Requirements](#3-algorithm-iterator-requirements)
4. [New C++17 Numeric Algorithms](#4-new-c17-numeric-algorithms)
5. [Exception Handling and `std::terminate`](#5-exception-handling-and-stdterminate)
6. [Data Races and Thread Safety](#6-data-races-and-thread-safety)
7. [Performance Considerations](#7-performance-considerations)
8. [Implementation Backends](#8-implementation-backends)
9. [Example: Transforming Data in Parallel](#9-example-transforming-data-in-parallel)
10. [Feature-Test Macro and Compiler Support](#10-feature-test-macro-and-compiler-support)
11. [Quick Reference Card](#11-quick-reference-card)
12. [Pitfalls and Gotchas](#12-pitfalls-and-gotchas)
13. [Conclusion](#13-conclusion)
14. [Contributors](#14-contributors)

---

## 1. Introduction to Execution Policies

Execution policies allow you to specify *how* an algorithm should execute. The policy is passed as the first parameter to algorithms that support parallel overloads. The header `<execution>` provides the tags required to control the execution strategy.

---

## 2. The Standard Execution Policies (C++17)

In C++17, the standard introduced three execution policies. Note that `std::execution::unseq` (SIMD-only) was added in C++20, not C++17.

| Policy | Semantic | Description |
|---|---|---|
| `std::execution::seq` | **Sequential** | No parallelism; same behavior as non-policy algorithms. |
| `std::execution::par` | **Parallel** | Parallel execution allowed using multiple threads. |
| `std::execution::par_unseq` | **Parallel & Unsequenced** | Multithreading + vectorization allowed. |

- **`par`**: Ideal for compute-heavy tasks where thread management overhead is offset by the workload size.
- **`par_unseq`**: The "maximum performance" policy, allowing the compiler to use both multi-threading and SIMD vectorization.

---

## 3. Algorithm Iterator Requirements

Standard sequential STL algorithms often accept `InputIterator`, but parallel algorithms require **`ForwardIterator`** as the minimum requirement.

- **Minimum Standard Requirement:** `ForwardIterator` (the range must support multiple passes).
- **Optimization Hint:** While `ForwardIterator` is sufficient for algorithms like `std::for_each`, algorithms that involve partitioning (like `std::sort`) perform significantly better with **`RandomAccessIterator`**. Providing a non-random access iterator to a parallel sort may lead to suboptimal performance.

---

## 4. New C++17 Numeric Algorithms

Because traditional algorithms like `std::accumulate` enforce strict left-to-right ordering, they are inherently sequential and cannot be parallelized. C++17 introduces new numeric algorithms specifically designed for parallel execution by relaxing the ordering requirement (often requiring operations to be commutative/associative):

| Algorithm | Description | Parallel Equivalent of |
|---|---|---|
| `std::reduce` | Reduces a range to a single value. | `std::accumulate` |
| `std::transform_reduce` | Transforms input, then reduces the result. | `std::inner_product` |
| `std::inclusive_scan` | Prefix sum (includes current element). | `std::partial_sum` |
| `std::exclusive_scan` | Prefix sum (excludes current element). | N/A |

---

## 5. Exception Handling and `std::terminate`

Parallel algorithms handle exceptions with strict rules:
1. **User-thrown exceptions:** If the lambda or function object passed to a parallel algorithm throws an exception, **`std::terminate()`** is called immediately. There is no mechanism to catch these exceptions within the algorithm context.
2. **Library-thrown exceptions:** If an implementation cannot perform an internal allocation required for the parallel algorithm (e.g., allocating a thread pool or buffer), it may throw `std::bad_alloc`. This is the only standard exception that may propagate out of the algorithm call.

---

## 6. Data Races and Thread Safety

User code passed to parallel algorithms must be **thread-safe** with respect to shared state.

- **Non-interference:** Concurrent access to distinct elements of a container is safe.
- **Race conditions:** Modifying shared global state from within the parallel lambda without synchronization is **undefined behavior**. Even *with* synchronization (e.g., `std::mutex`), severe performance degradation occurs due to contention.

---

## 7. Performance Considerations

Parallelism is not "free." Overhead includes:
- **Task Splitting:** Dividing the range into chunks.
- **Thread Synchronization:** Coordinating between workers.
- **False Sharing:** Modifying variables on the same cache line from different threads, causing cache coherency traffic.

**Rule of Thumb:** Only use `par` or `par_unseq` for large datasets. For small containers (e.g., $N < 1000$), the overhead of thread management often exceeds the sequential execution time.

---

## 8. Implementation Backends

C++17 defines the interface, but the backend is implementation-defined:
- **GCC (libstdc++):** Typically requires `libtbb` (Intel Threading Building Blocks) or OpenMP.
- **Clang/LLVM:** Also relies on TBB or OpenMP runtimes.
- **MSVC:** Uses internal concurrency runtime (ConcRT).

You must often explicitly link the backend library (e.g., `-ltbb`) during compilation.

---

## 9. Example: Transforming Data in Parallel

```cpp
#include <algorithm>
#include <execution>
#include <vector>
#include <cmath>

void compute_squares(std::vector<double>& data) {
    // Parallel execution policy + Vectorization
    std::transform(std::execution::par_unseq, 
                   data.begin(), data.end(), 
                   data.begin(), 
                   [](double x) { return std::pow(x, 2.0); });
}
```

---

## 10. Feature-Test Macro and Compiler Support

Header: `<version>` or `<algorithm>`

```cpp
#if defined(__cpp_lib_parallel_algorithm) && __cpp_lib_parallel_algorithm >= 201603L
    // Parallel algorithms are supported
#endif
```

### Compiler Support Table

| Compiler | Backend Required | Standard Flag |
|---|---|---|
| **GCC 9+** | `libtbb` | `-std=c++17 -ltbb` |
| **Clang 7+** | `libtbb` | `-std=c++17 -ltbb` |
| **MSVC 2017+** | Native | `/std:c++17` |

---

## 11. Quick Reference Card

```cpp
#include <algorithm>
#include <execution>
#include <numeric>

// 1. Parallel reduction
std::reduce(std::execution::par, v.begin(), v.end());

// 2. Multithreaded transform
std::transform(std::execution::par, in.begin(), in.end(), out.begin(), func);

// 3. Inclusive scan
std::inclusive_scan(std::execution::par, in.begin(), in.end(), out.begin());
```

---

## 12. Pitfalls and Gotchas

1. **Over-parallelizing:** Using `std::execution::par` on trivially small operations or small datasets.
2. **Global State:** Lambdas modifying non-thread-local global variables are unsafe.
3. **Synchronization Bottlenecks:** Using `std::mutex` inside a parallel lambda essentially serializes the algorithm, negating all performance gains.
4. **False Sharing:** Modifying adjacent memory locations concurrently, forcing cache lines to bounce between CPU cores.

---

## 13. Conclusion

C++17 Parallel Algorithms bring high-performance computing constructs into the standard STL. By using `std::execution` policies, developers can scale algorithms across multi-core systems with minimal boilerplate. However, developers must take responsibility for data races, exception safety, and careful selection of workload sizes to ensure meaningful performance gains.

---

## 14. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |