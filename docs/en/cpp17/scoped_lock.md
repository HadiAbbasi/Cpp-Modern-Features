<div align="center">

[🇺🇸 English](./scoped_lock.md) | [🇮🇷 فارسی](../../fa/cpp17/scoped_lock.md)

</div>

----

# `std::scoped_lock` in C++17

Multithreaded concurrent applications rely on mutual exclusion primitives to protect shared data against race conditions. However, acquiring multiple locks across concurrent operations introduces the risk of distributed deadlocks (specifically *circular wait* conditions) and verbose RAII boilerplate.

Prior to C++17, concurrent code requiring multiple locks had to combine `std::lock` with `std::unique_lock` or `std::lock_guard` using `std::adopt_lock`:
1. **Multi-statement ceremony:** Locking and RAII ownership were separated across multiple lines, increasing cognitive load and bug potential.
2. **Exception safety hazards:** If an exception occurred between the creation of guards or during non-RAII setup, mutexes could remain locked indefinitely.
3. **Template verbosity:** Single-lock wrappers (`std::lock_guard<std::mutex>`) required explicit template arguments before Class Template Argument Deduction (CTAD) in C++17.

C++17 introduced `<mutex>` with `std::scoped_lock`: a variadic RAII lock manager that acquires **zero or more mutexes simultaneously** using a deadlock-avoidance algorithm (equivalent to `std::lock`) and releases them upon scope exit.

```cpp
#include <mutex>
#include <thread>
#include <iostream>

struct BankAccount {
    std::mutex mtx;
    int balance{100};
};

void transfer(BankAccount& from, BankAccount& to, int amount) {
    // Deadlock-free multi-lock acquisition via CTAD
    std::scoped_lock lock(from.mtx, to.mtx);

    from.balance -= amount;
    to.balance += amount;
} // Both mutexes automatically unlocked here
```

---

## Table of Contents

1. [Why `std::scoped_lock` Was Introduced](#1-why-stdscoped_lock-was-introduced)
2. [Class Template Definition and Variadic Parameters](#2-class-template-definition-and-variadic-parameters)
3. [Deadlock Avoidance Algorithm](#3-deadlock-avoidance-algorithm)
4. [Class Template Argument Deduction (CTAD)](#4-class-template-argument-deduction-ctad)
5. [Degenerate Cases: Single-Mutex and Zero-Mutex](#5-degenerate-cases-single-mutex-and-zero-mutex)
6. [Adopting Already-Locked Mutexes (`std::adopt_lock`)](#6-adopting-already-locked-mutexes-stdadopt_lock)
7. [Exception Safety and Invariants](#7-exception-safety-and-invariants)
8. [Real-World Architectural Patterns](#8-real-world-architectural-patterns)
   - 8.1 [Thread-Safe Account Balance Transfer](#81-thread-safe-account-balance-transfer)
   - 8.2 [Multi-Resource Transaction Engine](#82-multi-resource-transaction-engine)
   - 8.3 [Concurrent Graph Edge Swapping](#83-concurrent-graph-edge-swapping)
9. [Comparison: `std::scoped_lock` vs `std::lock_guard` vs `std::unique_lock`](#9-comparison-stdscoped_lock-vs-stdlock_guard-vs-stdunique_lock)
10. [Common Pitfalls and Gotchas](#10-common-pitfalls-and-gotchas)
11. [Performance and Inlining Considerations](#11-performance-and-inlining-considerations)
12. [Best Practices](#12-best-practices)
13. [Feature-Test Macro and Compiler Support](#13-feature-test-macro-and-compiler-support)
14. [Quick Reference Card](#14-quick-reference-card)
15. [Complete Executable Example](#15-complete-executable-example)
16. [Exercises](#16-exercises)
17. [Conclusion](#17-conclusion)
18. [Contributors](#18-contributors)

---

## 1. Why `std::scoped_lock` Was Introduced

Acquiring multiple mutexes across different threads in inconsistent order is a primary cause of deadlocks:

```cpp
// THREAD 1:                  // THREAD 2:
std::lock_guard<std::mutex> l1(mtxA);     std::lock_guard<std::mutex> l2(mtxB);
std::lock_guard<std::mutex> l2(mtxB);     std::lock_guard<std::mutex> l1(mtxA);
// Potential DEADLOCK!
```

In C++11, developers avoided this using `std::lock` paired with `std::adopt_lock`:

```cpp
// C++11 verbose approach
std::lock(mtxA, mtxB);
std::lock_guard<std::mutex> lockA(mtxA, std::adopt_lock);
std::lock_guard<std::mutex> lockB(mtxB, std::adopt_lock);
```

While safe, this idiom required three distinct statements, invited copy-paste errors (such as forgetting `std::adopt_lock`), and made code noisy.

`std::scoped_lock` combines both actions into a single atomic RAII declaration:

```cpp
// C++17 modern approach
std::scoped_lock lock(mtxA, mtxB);
```

---

## 2. Class Template Definition and Variadic Parameters

`std::scoped_lock` is declared in `<mutex>` as a variadic template:

```cpp
namespace std {
    template <class... MutexTypes>
    class scoped_lock {
    public:
        // Acquire all mutexes using deadlock-avoidance algorithm
        explicit scoped_lock(MutexTypes&... m);

        // Adopt already-locked mutexes
        scoped_lock(adopt_lock_t, MutexTypes&... m);

        // Unlock all mutexes
        ~scoped_lock();

        scoped_lock(const scoped_lock&) = delete;
        scoped_lock& operator=(const scoped_lock&) = delete;
    };
}
```

- With **one mutex**, its type must satisfy the *Cpp17BasicLockable* requirements (providing `.lock()` and `.unlock()`).
- With **two or more mutexes**, **every** mutex type must satisfy the *Cpp17Lockable* requirements, which additionally requires a `.try_lock()` method to facilitate deadlock-avoidance logic.
- Mutex types can be heterogeneous: you can pass `std::mutex`, `std::recursive_mutex`, `std::shared_mutex`, or custom lockable types into the same `std::scoped_lock`.

---

## 3. Deadlock Avoidance Algorithm

When instantiated with **two or more mutexes**, `std::scoped_lock` delegates acquisition to `std::lock(m1, m2, ..., mn)`.

The standard requires a deadlock-avoiding acquisition but leaves the specific sequence of `lock()`, `try_lock()`, and `unlock()` calls implementation-defined. If any acquisition call throws an exception, all mutexes acquired during that call are released before the exception propagates. Implementations typically use a "try-and-back-off" algorithm to avoid circular hold-and-wait conditions.

---

## 4. Class Template Argument Deduction (CTAD)

In C++17, Class Template Argument Deduction eliminates explicit template specifications for lock guards:

```cpp
std::mutex m1;
std::shared_mutex m2;

// C++11/14 syntax:
// std::scoped_lock<std::mutex, std::shared_mutex> lock(m1, m2);

// C++17 CTAD deduction:
std::scoped_lock lock(m1, m2); // Deduced as std::scoped_lock<std::mutex, std::shared_mutex>
```

A deduction guide is provided by the standard library:

```cpp
template <class... MutexTypes>
scoped_lock(MutexTypes&...) -> scoped_lock<MutexTypes...>;
```

---

## 5. Degenerate Cases: Single-Mutex and Zero-Mutex

### Single-Mutex Specialization
When instantiated with exactly one mutex, the implementation is required by the standard to call `mtx.lock()` directly, bypassing the multi-mutex deadlock-avoidance loop. It functions as a direct, drop-in replacement for `std::lock_guard`.

### Zero-Mutex Specialization (`scoped_lock<>`)
The template allows zero arguments. This performs no-op construction and destruction, which is useful in generic variadic template expansions where the parameter pack of mutexes may expand to empty.

---

## 6. Adopting Already-Locked Mutexes (`std::adopt_lock`)

If mutexes are already acquired prior to creating the guard:

```cpp
std::mutex mtx1, mtx2;

// Manually locked earlier
std::lock(mtx1, mtx2);

// Transfer ownership to RAII guard
std::scoped_lock lock(std::adopt_lock, mtx1, mtx2);
// Mutexes will be unlocked automatically on scope exit
```

---

## 7. Exception Safety and Invariants

`std::scoped_lock` guarantees strong exception safety:
- If an exception is thrown while locking any mutex in the pack, all mutexes that were already acquired during the call are unlocked before the exception propagates out.
- The destructor is `noexcept` (provided `unlock()` does not throw, which is an invariant for all standard-compliant mutexes).

---

## 8. Real-World Architectural Patterns

### 8.1 Thread-Safe Account Balance Transfer

```cpp
#include <mutex>

class Account {
public:
    int id;
    double balance;
    mutable std::mutex mtx;

    Account(int id, double b) : id(id), balance(b) {}
};

bool transferFunds(Account& src, Account& dst, double amount) {
    if (&src == &dst) return false;

    // Deadlock-free locking
    std::scoped_lock lock(src.mtx, dst.mtx);

    if (src.balance >= amount) {
        src.balance -= amount;
        dst.balance += amount;
        return true;
    }
    return false;
}
```

### 8.2 Multi-Resource Transaction Engine

```cpp
#include <mutex>
#include <shared_mutex>
#include <string>

struct DatabaseShard {
    std::shared_mutex rw_mtx;
    std::string data;
};

void syncShards(DatabaseShard& primary, DatabaseShard& replica) {
    // Heterogeneous locks: both acquired in exclusive mode
    std::scoped_lock lock(primary.rw_mtx, replica.rw_mtx);
    replica.data = primary.data;
}
```

### 8.3 Concurrent Graph Edge Swapping

```cpp
#include <mutex>
#include <vector>

struct Node {
    std::mutex mtx;
    std::vector<int> edges;
};

void exchangeEdges(Node& a, Node& b) {
    std::scoped_lock guard(a.mtx, b.mtx);
    std::swap(a.edges, b.edges);
}
```

---

## 9. Comparison: `std::scoped_lock` vs `std::lock_guard` vs `std::unique_lock`

| Feature | `std::lock_guard` | `std::unique_lock` | `std::scoped_lock` |
|---|---|---|---|
| **Standard** | C++11 | C++11 | **C++17** |
| **Number of Mutexes** | Exactly 1 | Exactly 1 | **Variadic ($0, 1, 2, \dots$)** |
| **Deadlock Avoidance** | No | Optional (`std::defer_lock`) | **Automatic (for $\ge 2$ mutexes)** |
| **Manual Lock/Unlock** | No | Yes (`.lock()`, `.unlock()`) | **No** |
| **Condition Variable** | No | Yes (`std::condition_variable`) | **No** |
| **Movable** | No | Yes | **No** |
| **Overhead vs Direct Lock** | Zero | Small (stores bool flag) | **Zero (single) / Minimal (multi)** |

---

## 10. Common Pitfalls and Gotchas

### 10.1 Passing the Same Non-Recursive Mutex Multiple Times
Passing the same `std::mutex` instance more than once to `std::scoped_lock` results in **undefined behavior** (typically deadlock or system error):
```cpp
std::mutex m;
std::scoped_lock lock(m, m); // UNDEFINED BEHAVIOR!
```

### 10.2 Inability to Use with `std::condition_variable`
`std::condition_variable::wait` requires a `std::unique_lock<std::mutex>&`. You cannot pass `std::scoped_lock` to condition variables.

---

## 11. Performance and Inlining Considerations

- **Single Mutex:** Compilers inline `std::scoped_lock<Mutex>` down to the exact assembly instructions of `mtx.lock()` and `mtx.unlock()`. It has **identical runtime performance** to `std::lock_guard`.
- **Multiple Mutexes:** Compilers instantiate the standard library's `std::lock` algorithm, optimizing trivial cases where feasible.

---

## 12. Best Practices

1. **Replace `std::lock_guard` with `std::scoped_lock`:** In C++17 and later, prefer `std::scoped_lock` everywhere for standard RAII locking.
2. **Leverage CTAD:** Do not write explicit template arguments (e.g., prefer `std::scoped_lock lock(m)` over `std::scoped_lock<std::mutex> lock(m)`).
3. **Keep Lock Scopes Minimal:** Restrict the lifetime of `scoped_lock` instances to the minimum critical section required.
4. **Use `std::unique_lock` only when needed:** Reserve `std::unique_lock` for explicit manual unlocking, deferred locking, or condition variables.

---

## 13. Feature-Test Macro and Compiler Support

Header: `<mutex>`

```cpp
#if defined(__cpp_lib_scoped_lock) && __cpp_lib_scoped_lock >= 201703L
    // std::scoped_lock available
#endif
```

### Compiler Support Table

| Compiler | Version Support | Standard Flag |
|---|---|---|
| **GCC** | 7.1+ | `-std=c++17` |
| **Clang** | 5.0+ | `-std=c++17` |
| **MSVC** | 2017 (15.7+) | `/std:c++17` |

---

## 14. Quick Reference Card

```cpp
#include <mutex>

std::mutex m1, m2;

// 1. Single mutex lock (CTAD)
std::scoped_lock lock1(m1);

// 2. Multi-mutex deadlock-free lock (CTAD)
std::scoped_lock lock2(m1, m2);

// 3. Adopt existing locks
std::lock(m1, m2);
std::scoped_lock lock3(std::adopt_lock, m1, m2);
```

---

## 15. Complete Executable Example

```cpp
#include <iostream>
#include <mutex>
#include <thread>
#include <vector>

struct SafeAccount {
    int id;
    double balance;
    mutable std::mutex mtx;

    SafeAccount(int id, double bal) : id(id), balance(bal) {}
};

void transfer(SafeAccount& from, SafeAccount& to, double amount) {
    if (&from == &to) return;

    // Simultaneous deadlock-free lock acquisition
    std::scoped_lock lock(from.mtx, to.mtx);

    if (from.balance >= amount) {
        from.balance -= amount;
        to.balance += amount;
    }
}

int main() {
    SafeAccount acc1(1, 1000.0);
    SafeAccount acc2(2, 1000.0);

    std::vector<std::thread> workers;

    for (int i = 0; i < 50; ++i) {
        workers.emplace_back(transfer, std::ref(acc1), std::ref(acc2), 10.0);
        workers.emplace_back(transfer, std::ref(acc2), std::ref(acc1), 10.0);
    }

    for (auto& t : workers) {
        t.join();
    }

    std::scoped_lock final_check(acc1.mtx, acc2.mtx);
    std::cout << "Account 1 balance: " << acc1.balance << '\n';
    std::cout << "Account 2 balance: " << acc2.balance << '\n';
    std::cout << "Total balance: " << (acc1.balance + acc2.balance) << '\n';

    return 0;
}
```

### Expected Output

```text
Account 1 balance: 1000
Account 2 balance: 1000
Total balance: 2000
```

---

## 16. Exercises

### Exercise 1 — Dining Philosophers Without Deadlock
Implement the classical Dining Philosophers problem where 5 philosophers attempt to acquire their adjacent left and right forks (`std::mutex`). Use `std::scoped_lock` to guarantee deadlock-free execution without starvation.

### Exercise 2 — Multi-Container Atomic Transfer
Create a templated container transfer function that atomically moves an element from `std::vector<T>` in Bucket A to `std::vector<T>` in Bucket B, where each Bucket has its own `std::mutex`.

### Exercise 3 — Custom Lockable Compatibility
Write a lightweight custom spinlock class implementing `.lock()` and `.unlock()`. Verify that it seamlessly instantiates with `std::scoped_lock` alongside a standard `std::mutex`.

---

## 17. Conclusion

`std::scoped_lock` in C++17 provides:
- **Deadlock Avoidance:** Automates the multi-lock acquisition algorithm cleanly.
- **Syntactic Conciseness:** Replaces multi-line `std::lock` + `std::adopt_lock` ceremonies with a single line.
- **Universal Guard:** Serves as a zero-overhead replacement for `std::lock_guard` when used with a single mutex via CTAD.

---

## 18. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |