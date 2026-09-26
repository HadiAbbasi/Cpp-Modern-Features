<div align="center">

[🇺🇸 English](./shared_mutex.md) | [🇮🇷 فارسی](../../fa/cpp17/shared_mutex.md)

</div>

----

# A Complete Guide to `std::shared_mutex` in C++17: Readers-Writer Concurrency and Synchronization

## Table of Contents

- [Introduction](#introduction)
- [What Was the Problem Before `std::shared_mutex`?](#what-was-the-problem-before-stdshared_mutex)
- [The Readers-Writer Problem](#the-readers-writer-problem)
- [What Is the Solution?](#what-is-the-solution)
- [Required Header and History (C++14 vs C++17)](#required-header-and-history-c14-vs-c17)
- [What Is `std::shared_mutex`?](#what-is-stdshared_mutex)
- [Exclusive vs. Shared Ownership](#exclusive-vs-shared-ownership)
- [The RAII Approach: `std::unique_lock` and `std::shared_lock`](#the-raii-approach-stdunique_lock-and-stdshared_lock)
- [Why is `mutable` Essential in Class Design?](#why-is-mutable-essential-in-class-design)
- [Writing a Thread-Safe Getter (Reader)](#writing-a-thread-safe-getter-reader)
- [Writing a Thread-Safe Setter (Writer)](#writing-a-thread-safe-setter-writer)
- [Low-Level Interface (Manual Locking)](#low-level-interface-manual-locking)
  - [Manual Exclusive Locking: `lock` and `unlock`](#manual-exclusive-locking-lock-and-unlock)
  - [Manual Shared Locking: `lock_shared` and `unlock_shared`](#manual-shared-locking-lock_shared-and-unlock_shared)
- [Non-Blocking Locking: `try_lock` and `try_lock_shared`](#non-blocking-locking-try_lock-and-try_lock_shared)
- [Difference Between `std::shared_mutex` and `std::shared_timed_mutex`](#difference-between-stdshared_mutex-and-stdshared_timed_mutex)
- [Differences Between `std::mutex` and `std::shared_mutex`](#differences-between-stdmutex-and-stdshared_mutex)
- [Performance Considerations: When Is It Actually Faster?](#performance-considerations-when-is-it-actually-faster)
- [Common Mistakes and Pitfalls](#common-mistakes-and-pitfalls)
  - [Common Mistake: Using `unique_lock` Everywhere](#common-mistake-using-unique_lock-everywhere)
  - [Common Mistake: Manual `unlock` Missing on Exceptions](#common-mistake-manual-unlock-missing-on-exceptions)
  - [Common Mistake: Lock Upgrade/Downgrade Pitfalls](#common-mistake-lock-upgradedowngrade-pitfalls)
  - [Common Mistake: Recursive Locking (Deadlock)](#common-mistake-recursive-locking-deadlock)
  - [Common Mistake: Reader/Writer Starvation](#common-mistake-readerwriter-starvation)
- [When Should We Use `std::shared_mutex`?](#when-should-we-use-stdshared_mutex)
- [A Complete Production-Ready Example: Thread-Safe LRU/Cache](#a-complete-production-ready-example-thread-safe-lrucache)
- [Final Summary](#final-summary)
- [🤝 Contributors](#-contributors)

---

## Introduction

Concurrent programming and multithreading are essential for building high-performance modern applications. However, sharing data between multiple threads introduces the risk of data races, state inconsistency, and undefined behavior.

Standard synchronization primitives like `std::mutex` provide safety by enforcing strict mutual exclusion: **only one thread can access the data at any time**. 

While safe, this creates a severe performance bottleneck when multiple threads only want to **read** the shared state concurrently without modifying it. 

To solve this, modern C++ introduces `std::shared_mutex` in the C++17 standard, implementing the classic **Readers-Writer Lock (RW Lock)** pattern.

---

## What Was the Problem Before `std::shared_mutex`?

Consider a service where 100 threads are constantly reading configuration data or querying a cache, while only 1 thread occasionally updates that data.

Using a standard `std::mutex`:

```cpp
#include <mutex>

class ConfigRegistry {
private:
    std::mutex mtx;
    ConfigData config;

public:
    ConfigData getConfig() {
        std::lock_guard<std::mutex> lock(mtx);
        return config; // Thread 2 is blocked even if Thread 1 is only reading!
    }

    void updateConfig(const ConfigData& newConfig) {
        std::lock_guard<std::mutex> lock(mtx);
        config = newConfig;
    }
};
```

The problem with this approach is that `std::mutex` treats **reads** and **writes** identically:
- Reading threads are serialized and forced to wait in a queue behind other reading threads.
- CPU cores sit idle waiting for lock acquisition, destroying read throughput and scalability.

---

## The Readers-Writer Problem

The fundamental nature of concurrency states:
1. **Read-Read Concurrency:** Multiple threads reading the same memory location simultaneously is completely safe and free of data races.
2. **Read-Write Concurrency:** A thread writing while another thread reads causes data corruption and undefined behavior.
3. **Write-Write Concurrency:** Multiple threads writing concurrently corrupts data.

The goal of a Readers-Writer lock is to allow **unlimited concurrent readers** while ensuring **strictly exclusive access for writers**.

---

## What Is the Solution?

The solution is an asymmetric synchronization primitive with two distinct access tiers:
- **Shared Access (Reader Lock):** Acquired concurrently by multiple threads as long as no thread holds the exclusive lock.
- **Exclusive Access (Writer Lock):** Acquired by exactly one thread, completely blocking all other readers and writers until released.

`std::shared_mutex` provides this mechanism natively with standard C++ semantics and zero OS-dependent API calls.

---

## Required Header and History (C++14 vs C++17)

To use `std::shared_mutex`, include `<shared_mutex>`:

```cpp
#include <shared_mutex>
```

### Evolution Note:
- **C++14:** Introduced `std::shared_timed_mutex`, which supports shared locking with timeouts (`try_lock_for`, `try_lock_until`), but carried additional runtime and memory overhead.
- **C++17:** Introduced `std::shared_mutex` as a leaner, faster, non-timed alternative. If you don't need timeout operations, `std::shared_mutex` is always preferred.

---

## What Is `std::shared_mutex`?

`std::shared_mutex` is a synchronization primitive that models shared-ownership locking.

```cpp
#include <shared_mutex>

std::shared_mutex rw_mtx;
```

It is a non-copyable, non-movable class designed to protect shared state across thread boundaries.

---

## Exclusive vs. Shared Ownership

| Ownership Mode | Intended Operation | Acquiring Wrapper | Concurrency Rule |
| :--- | :--- | :--- | :--- |
| **Exclusive (Write)** | Modifying shared state | `std::unique_lock` / `std::lock_guard` | **1 Writer ONLY** (0 Readers, 0 Other Writers) |
| **Shared (Read)** | Observing shared state | `std::shared_lock` | **N Readers** concurrently (0 Writers) |

```text
                  ┌──────────────────────┐
                  │  std::shared_mutex   │
                  └──────────┬───────────┘
                             │
            ┌────────────────┴────────────────┐
            ▼                                 ▼
   Exclusive Access (Write)           Shared Access (Read)
 ┌───────────────────────────┐      ┌───────────────────────────┐
 │   std::unique_lock        │      │   std::shared_lock        │
 │   - Blocks all readers    │      │   - Allows other readers  │
 │   - Blocks all writers    │      │   - Blocks all writers    │
 └───────────────────────────┘      └───────────────────────────┘
```

---

## The RAII Approach: `std::unique_lock` and `std::shared_lock`

Just as with `std::unique_ptr` and memory, manual lock management in C++ is error-prone. Always manage `std::shared_mutex` through RAII guard objects.

C++ standard library provides two main lock wrappers for this:

1. **`std::shared_lock` (Shared Guard):** Calls `lock_shared()` upon construction and `unlock_shared()` upon destruction.
2. **`std::unique_lock` (Exclusive Guard):** Calls `lock()` upon construction and `unlock()` upon destruction.
*(Note: In C++17, `std::lock_guard` can also be used for exclusive locking).*

```cpp
std::shared_mutex rw_mtx;

void reader() {
    std::shared_lock<std::shared_mutex> lock(rw_mtx); // RAII Shared Lock
    // Read shared data safely...
} // Automatically unlocks lock_shared() when exiting scope

void writer() {
    std::unique_lock<std::shared_mutex> lock(rw_mtx); // RAII Exclusive Lock
    // Modify shared data safely...
} // Automatically unlocks lock() when exiting scope
```

---

## Why is `mutable` Essential in Class Design?

In idiomatic C++, getter functions that do not alter the logical state of an object must be marked `const`.

However, acquiring a shared lock on a mutex internally modifies the state of the mutex (e.g., incrementing an internal reader counter). If the mutex is not declared `mutable`, calling `std::shared_lock lock(rw_mtx)` inside a `const` member function will cause a compile error.

```cpp
class SafeCounter {
private:
    mutable std::shared_mutex rw_mtx; // Must be mutable for const methods!
    int value = 0;

public:
    // const method guarantees logical immutability to caller
    int getValue() const {
        std::shared_lock lock(rw_mtx); // Modifies rw_mtx internally, allowed via mutable
        return value;
    }
};
```

---

## Writing a Thread-Safe Getter (Reader)

A proper getter method should:
- Be marked `const`.
- Use `std::shared_lock`.
- Return either by value (for small primitive/copyable types) or under careful synchronization.

```cpp
std::string get(const std::string& key) const {
    std::shared_lock lock(rw_mtx);
    auto it = map.find(key);
    if (it != map.end()) {
        return it->second;
    }
    return {};
}
```

---

## Writing a Thread-Safe Setter (Writer)

A setter or modifier method must:
- Use `std::unique_lock` (or `std::lock_guard`).
- Keep the critical section as short as possible to minimize blocking concurrent readers.

```cpp
void set(const std::string& key, const std::string& val) {
    std::unique_lock lock(rw_mtx);
    map[key] = val;
}
```

---

## Low-Level Interface (Manual Locking)

While RAII guards should always be your default choice, `std::shared_mutex` exposes manual locking primitives for specialized low-level control.

### Manual Exclusive Locking: `lock` and `unlock`

```cpp
rw_mtx.lock();
// Exclusive critical section (modify shared data)
rw_mtx.unlock();
```

### Manual Shared Locking: `lock_shared` and `unlock_shared`

```cpp
rw_mtx.lock_shared();
// Shared critical section (read shared data)
rw_mtx.unlock_shared();
```

---

## Non-Blocking Locking: `try_lock` and `try_lock_shared`

If a thread cannot afford to block while waiting for a lock, it can use the non-blocking polling variants:

```cpp
// Try acquiring exclusive lock
if (rw_mtx.try_lock()) {
    // Got exclusive access
    rw_mtx.unlock();
} else {
    // Mutex is held by either a writer or one/more readers
}

// Try acquiring shared lock
if (rw_mtx.try_lock_shared()) {
    // Got shared access alongside other readers
    rw_mtx.unlock_shared();
} else {
    // Mutex is currently held by an exclusive writer
}
```

---

## Difference Between `std::shared_mutex` and `std::shared_timed_mutex`

| Feature | `std::shared_mutex` (C++17) | `std::shared_timed_mutex` (C++14) |
| :--- | :--- | :--- |
| **Standard** | C++17 | C++14 |
| **Timeout Support** | ❌ No (`lock`, `try_lock`) | ✅ Yes (`try_lock_for`, `try_lock_until`) |
| **Memory Footprint** | Smaller | Larger |
| **Performance Overhead** | Lower (optimal for standard RW lock) | Higher (due to timer tracking) |
| **Recommended Default** | **Yes** (unless timeouts are strictly needed) | Only when timed locks are required |

---

## Differences Between `std::mutex` and `std::shared_mutex`

| Property | `std::mutex` | `std::shared_mutex` |
| :--- | :--- | :--- |
| **Access Pattern** | Strictly Exclusive | Shared (N Readers) OR Exclusive (1 Writer) |
| **Lock Guard Required** | `std::lock_guard` / `std::unique_lock` | `std::shared_lock` (Read) / `std::unique_lock` (Write) |
| **Locking Overhead** | Very low (single atomic/futex) | Slightly higher (maintains reader count state) |
| **Ideal Workload** | Write-heavy, balanced, or short-lived sections | Read-heavy workloads (>80-90% reads) |

---

## Performance Considerations: When Is It Actually Faster?

`std::shared_mutex` is not a magic speedup for every multithreaded application. Acquiring a shared lock involves atomic increments and cache-line invalidation across CPU cores.

### The Rule of Thumb:
- **High Read-to-Write Ratio (e.g., 90% Reads, 10% Writes):** `std::shared_mutex` offers massive throughput gains over `std::mutex` because read threads run truly in parallel without waiting.
- **Write-Heavy or 50/50 Ratio:** Standard `std::mutex` will almost always outperform `std::shared_mutex` due to the lower overhead of a plain exclusive mutex.
- **Extremely Short Read Operations:** If the read operation is trivial (e.g., reading a single integer), `std::atomic<T>` is vastly superior to both `std::mutex` and `std::shared_mutex`.

---

## Common Mistakes and Pitfalls

### Common Mistake: Using `unique_lock` Everywhere

```cpp
// ❌ INCORRECT: Reader using unique_lock ruins concurrency
int get() const {
    std::unique_lock lock(rw_mtx); 
    return data;
}

// ✅ CORRECT: Reader uses shared_lock
int get() const {
    std::shared_lock lock(rw_mtx);
    return data;
}
```

### Common Mistake: Manual `unlock` Missing on Exceptions

```cpp
// ❌ DANGEROUS: Exception causes permanent deadlock
void process() {
    rw_mtx.lock();
    doSomethingRisky(); // If this throws, unlock() is never called!
    rw_mtx.unlock();
}

// ✅ CORRECT: RAII guarantees release during stack unwinding
void process() {
    std::unique_lock lock(rw_mtx);
    doSomethingRisky();
}
```

### Common Mistake: Lock Upgrade/Downgrade Pitfalls

Standard C++ `std::shared_mutex` **does not support atomic lock upgrading** (converting a `shared_lock` to a `unique_lock` in-place).

```cpp
// ❌ WRONG / UNDEFINED BEHAVIOR: Attempting to upgrade while holding shared_lock
void updateValue() {
    std::shared_lock read_lock(rw_mtx);
    if (needsUpdate()) {
        std::unique_lock write_lock(rw_mtx); // DEADLOCK! Thread waits for itself to release shared_lock
        applyUpdate();
    }
}

// ✅ CORRECT: Release shared lock first, then acquire exclusive lock
void updateValue() {
    {
        std::shared_lock read_lock(rw_mtx);
        if (!needsUpdate()) return;
    } // read_lock released here

    std::unique_lock write_lock(rw_mtx);
    // Double-check condition in case another thread updated it in the gap
    if (needsUpdate()) {
        applyUpdate();
    }
}
```

### Common Mistake: Recursive Locking (Deadlock)

`std::shared_mutex` is **not recursive**. If a thread holding a shared or exclusive lock tries to lock it again on the same thread, undefined behavior (typically an immediate deadlock) occurs.

### Common Mistake: Reader/Writer Starvation

- If new readers arrive continuously, writers might starve and wait indefinitely.
- Most OS-level C++ standard library implementations (such as glibc/pthreads and MSVC STL) implement fair scheduling or writer-preference policies to prevent writer starvation, but you should avoid holding reader locks across long-running I/O operations.

---

## When Should We Use `std::shared_mutex`?

Use `std::shared_mutex` when all of the following are true:
1. Data is shared across multiple concurrent threads.
2. Read operations occur far more frequently than write operations (>80% reads).
3. The read operations take enough time that concurrent execution outweighs the mutex's atomic bookkeeping overhead.
4. The resource is too complex to be modeled as simple `std::atomic` variables.

---

## A Complete Production-Ready Example: Thread-Safe LRU/Cache

Here is a complete, real-world example demonstrating a thread-safe in-memory cache using `std::shared_mutex`:

```cpp
#include <iostream>
#include <string>
#include <unordered_map>
#include <optional>
#include <shared_mutex>
#include <thread>
#include <vector>

template <typename Key, typename Value>
class ThreadSafeCache {
private:
    mutable std::shared_mutex rw_mtx;
    std::unordered_map<Key, Value> storage;

public:
    ThreadSafeCache() = default;

    // Concurrent read access (Multiple threads can read simultaneously)
    std::optional<Value> get(const Key& key) const {
        std::shared_lock<std::shared_mutex> lock(rw_mtx);
        
        auto it = storage.find(key);
        if (it != storage.end()) {
            return it->second;
        }
        return std::nullopt;
    }

    // Exclusive write access (Only one writer at a time, blocking readers)
    void put(const Key& key, const Value& value) {
        std::unique_lock<std::shared_mutex> lock(rw_mtx);
        storage[key] = value;
    }

    // Check key existence (Concurrent read)
    bool contains(const Key& key) const {
        std::shared_lock<std::shared_mutex> lock(rw_mtx);
        return storage.find(key) != storage.end();
    }

    // Remove key (Exclusive write)
    bool erase(const Key& key) {
        std::unique_lock<std::shared_mutex> lock(rw_mtx);
        return storage.erase(key) > 0;
    }

    // Clear entire cache (Exclusive write)
    void clear() {
        std::unique_lock<std::shared_mutex> lock(rw_mtx);
        storage.clear();
    }

    // Get current size (Concurrent read)
    std::size_t size() const {
        std::shared_lock<std::shared_mutex> lock(rw_mtx);
        return storage.size();
    }
};

int main() {
    ThreadSafeCache<std::string, std::string> sessionCache;

    // Prepopulate
    sessionCache.put("user_101", "session_token_abc");
    sessionCache.put("user_102", "session_token_xyz");

    // Spawn multiple reader threads
    std::vector<std::thread> readers;
    for (int i = 0; i < 5; ++i) {
        readers.emplace_back([&sessionCache, i]() {
            auto val = sessionCache.get("user_101");
            if (val) {
                std::cout << "[Reader " << i << "] Found token: " << *val << "\n";
            }
        });
    }

    // Spawn a writer thread
    std::thread writer([&sessionCache]() {
        sessionCache.put("user_103", "session_token_new");
        std::cout << "[Writer] Added new session for user_103\n";
    });

    for (auto& t : readers) {
        t.join();
    }
    writer.join();

    std::cout << "Final Cache Size: " << sessionCache.size() << "\n";

    return 0;
}
```

---

## Final Summary

`std::shared_mutex` is the standard modern C++ approach for resolving contentions in **read-heavy concurrent systems**.

The key statements to remember:

```cpp
// 1. Shared ownership (Readers)
std::shared_lock<std::shared_mutex> lock(rw_mtx);
```
> Allows unlimited simultaneous readers. Blocks writers.

```cpp
// 2. Exclusive ownership (Writers)
std::unique_lock<std::shared_mutex> lock(rw_mtx);
```
> Enforces single-thread exclusive access. Blocks all readers and writers.

```cpp
// 3. Always declare inside class as mutable
mutable std::shared_mutex rw_mtx;
```
> Enables safe usage inside `const` member functions.

Always ask yourself before choosing a mutex:
> **"What is the read-to-write ratio of this critical section?"**
- If reads dominate: Use `std::shared_mutex` with `std::shared_lock`.
- If writes dominate or sections are trivial: Use `std::mutex` with `std::lock_guard` / `std::unique_lock` or `std::atomic`.

---

## 🤝 Contributors


| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |[Ordikhani](ordikhanifateme@gmail.com) |  | [Ordikhani](@OrdikhaniFateme) |





