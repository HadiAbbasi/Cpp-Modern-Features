<div align="center">

[🇺🇸 English](./std_shared_timed_mutex.md) | [🇮🇷 فارسی](../../en/cpp14/std_shared_timed_mutex.md)

</div>

---

# Introduction to `std::shared_timed_mutex` in C++14

## Introduction

In multithreaded programming, one of the main challenges is managing concurrent access to shared data. When multiple threads modify the same variable or data structure simultaneously, or when one thread modifies data while other threads are reading it, a race condition may occur.

To manage these situations, the C++ standard library provides various synchronization tools. One of them is `std::shared_timed_mutex`, a class introduced in C++14.

This class is a type of mutex that supports both exclusive locking and shared locking, as well as timed attempts to acquire a lock.

Its key feature is that multiple threads can hold shared locks simultaneously, while normally only one thread can hold an exclusive lock. An exclusive lock is incompatible with shared locks, so a thread cannot acquire an exclusive lock while shared locks remain active.

This capability is useful in systems where read operations significantly outnumber write operations, such as caches, configuration tables, internal registries, and data structures that are frequently accessed by multiple threads for reading.

## Introducing the Shared Mutex Concept

### The Difference Between Exclusive and Shared Locks

With a conventional mutex such as `std::mutex`, only one thread can hold the lock at a time. This behavior is essential for protecting write operations, but it can be restrictive when multiple threads only need to read data.

In contrast, a shared mutex supports two primary ownership modes:

- Exclusive ownership: Only one thread can hold the lock exclusively. This mode is typically used to modify shared data.
- Shared ownership: Multiple threads can hold shared locks simultaneously. This mode is typically used to read shared data.

The ownership rules can be summarized as follows:

| Current Lock State        | Acquire Shared Lock                             | Acquire Exclusive Lock                          |
| ------------------------- | ----------------------------------------------- | ----------------------------------------------- |
| Unlocked                  | Allowed                                         | Allowed                                         |
| One or more shared owners | Allowed, subject to the implementation's policy | Not allowed until all shared locks are released |
| Exclusive owner           | Not allowed                                     | Not allowed                                     |

The important point is that an exclusive lock cannot coexist with either a shared lock or another exclusive lock.

The C++ standard also does not guarantee that waiting threads will always acquire the lock in the order in which they arrived. Therefore, you should not rely on implementation-specific behavior to guarantee fairness or eliminate starvation.

### Why Do We Need a Shared Mutex?

Suppose a multithreaded service stores user information in a shared data structure. Hundreds of threads may read the information, while only a small number of threads modify it occasionally.

If `std::mutex` is used for every operation, even independent read operations must acquire the lock exclusively. This can reduce parallelism.

With `std::shared_timed_mutex`, multiple threads can read information simultaneously, while write operations remain protected by exclusive locking.

However, this design is not always faster. The internal overhead of a shared mutex may exceed that of a conventional mutex, and its benefits depend on the access pattern, number of threads, hardware, and lock-holding duration.

## Introducing the Main Capabilities of the Class

The `std::shared_timed_mutex` class is defined in the standard `<shared_mutex>` header.

The following example demonstrates its basic definition and usage:

```
#include <shared_mutex>
#include <iostream>

int main() {
    std::shared_timed_mutex mutex;

    mutex.lock();

    std::cout << "Exclusive lock acquired\n";

    mutex.unlock();

    return 0;
}
```

In this example, the main thread acquires an exclusive lock and releases it after performing the operation.

In real-world applications, it is generally better to use RAII utilities such as `std::unique_lock` and `std::shared_lock` instead of managing locks manually.

## Comparing `std::shared_timed_mutex` and `std::shared_mutex`

Both classes support shared and exclusive ownership, but there is an important difference in their capabilities.

`std::shared_timed_mutex` has been available since C++14 and supports timed operations in addition to ordinary locking operations.

`std::shared_mutex` was introduced in C++17 and provides a simpler interface for shared and exclusive locking, but it does not have dedicated timed-locking operations.

| Feature                            | `std::shared_timed_mutex` | `std::shared_mutex` |
| ---------------------------------- | ------------------------- | ------------------- |
| Standard version                   | C++14                     | C++17               |
| Exclusive locking                  | Yes                       | Yes                 |
| Shared locking                     | Yes                       | Yes                 |
| Timed locking                      | Yes                       | No                  |
| Compatible with `std::unique_lock` | Yes                       | Yes                 |
| Compatible with `std::shared_lock` | Yes                       | Yes                 |
| Supports `try_lock_for()`          | Yes                       | No                  |
| Supports `try_lock_until()`        | Yes                       | No                  |

If an application needs timeouts when acquiring locks, `std::shared_timed_mutex` is an appropriate choice. Otherwise, in C++17 and later, `std::shared_mutex` may be a simpler option.

## Understanding Locking Operations

The `std::shared_timed_mutex` class provides several groups of operations for controlling access.

### Exclusive Locking Operations

Exclusive operations are used when a thread needs to modify data or requires exclusive access.

| Function           | Purpose                                                                         |
| ------------------ | ------------------------------------------------------------------------------- |
| `lock()`           | Wait until an exclusive lock can be acquired                                    |
| `try_lock()`       | Attempt to acquire an exclusive lock without waiting for it to become available |
| `unlock()`         | Release an exclusive lock                                                       |
| `try_lock_for()`   | Attempt to acquire an exclusive lock for a specified duration                   |
| `try_lock_until()` | Attempt to acquire an exclusive lock until a specified time                     |

The `lock()` function may block until the lock becomes available. The `try_lock()` function attempts to acquire exclusive ownership without waiting for the lock to be released.

If `try_lock()` fails, it returns `false`. If it succeeds, it returns `true`.

### Shared Locking Operations

Shared operations are designed to allow multiple threads to access shared data concurrently.

| Function                  | Purpose                                                                     |
| ------------------------- | --------------------------------------------------------------------------- |
| `lock_shared()`           | Wait until a shared lock can be acquired                                    |
| `try_lock_shared()`       | Attempt to acquire a shared lock without waiting for it to become available |
| `unlock_shared()`         | Release a shared lock                                                       |
| `try_lock_shared_for()`   | Attempt to acquire a shared lock for a specified duration                   |
| `try_lock_shared_until()` | Attempt to acquire a shared lock until a specified time                     |

In shared mode, multiple threads can hold the lock simultaneously, provided that no other thread holds an exclusive lock and the implementation's constraints permit the acquisition.

## Using Exclusive Locking with RAII

Directly calling `lock()` and `unlock()` can leave a lock unreleased if an exception occurs or a function exits early.

To avoid this problem, you can use `std::unique_lock`.

```
#include <shared_mutex>
#include <mutex>
#include <iostream>

class Counter {
public:
    void increment() {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        ++value_;
    }

    int get() {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        return value_;
    }

private:
    std::shared_timed_mutex mutex_;
    int value_ = 0;
};

int main() {
    Counter counter;

    counter.increment();
    counter.increment();

    std::cout << counter.get() << '\n';
}
```

In this example, `std::unique_lock` acquires the exclusive lock when it is constructed and releases it automatically when the object's lifetime ends.

Although `get()` only reads the data, this example uses exclusive locking. The implementation can be improved by using shared locking instead, as demonstrated in the next section.

## Using Shared Locking with `std::shared_lock`

The `std::shared_lock` class was introduced in C++14 and provides RAII management for shared locks.

It requires a mutex that supports shared locking, including `std::shared_timed_mutex`.

The following example uses a shared lock to read data:

```
#include <shared_mutex>
#include <mutex>
#include <iostream>

class Counter {
public:
    void increment() {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        ++value_;
    }

    int get() const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        return value_;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    int value_ = 0;
};

int main() {
    Counter counter;

    counter.increment();
    counter.increment();

    std::cout << counter.get() << '\n';
}
```

In this version, `get()` uses `std::shared_lock`, allowing it to execute concurrently with other readers.

The `mutable` keyword is necessary for the mutex because `get()` is a `const` member function, but synchronization requires modifying the mutex's internal state.

An important point is that shared locking is safe only when the protected data is not modified in an incompatible way while the lock is held. In this example, all modifications to `value_` take place under an exclusive lock.

## Using Timed Locking

One of the important capabilities of `std::shared_timed_mutex` is the ability to attempt to acquire a lock with a time limit.

This capability is useful in systems that should not allow a thread to wait indefinitely to acquire a lock.

### Using `try_lock_for()`

The `try_lock_for()` function accepts a relative duration. If the lock can be acquired within that duration, it returns `true`; otherwise, it returns `false`.

The following example uses a timed exclusive lock:

```
#include <shared_mutex>
#include <mutex>
#include <chrono>
#include <iostream>

int main() {
    std::shared_timed_mutex mutex;

    if (mutex.try_lock_for(std::chrono::milliseconds(100))) {
        std::cout << "Lock acquired\n";
        mutex.unlock();
    } else {
        std::cout << "Lock acquisition timed out\n";
    }

    return 0;
}
```

In this example, the thread attempts to acquire the exclusive lock for approximately 100 milliseconds. The actual waiting time may depend on system scheduling and implementation details.

To avoid forgetting to release the lock, it is preferable to use `std::unique_lock` with `std::defer_lock`.

```
#include <shared_mutex>
#include <mutex>
#include <chrono>
#include <iostream>

int main() {
    std::shared_timed_mutex mutex;

    std::unique_lock<std::shared_timed_mutex> lock(
        mutex, std::defer_lock
    );

    if (lock.try_lock_for(std::chrono::milliseconds(100))) {
        std::cout << "Lock acquired\n";
    } else {
        std::cout << "Lock acquisition timed out\n";
    }

    return 0;
}
```

In this version, the `lock` object initially does not own the mutex. If `try_lock_for()` succeeds, it acquires ownership of the lock, which is then released automatically when the object's lifetime ends.

### Using `try_lock_until()`

The `try_lock_until()` function accepts an absolute time point rather than a relative duration. This time point is typically created using `std::chrono::steady_clock` or another appropriate clock.

```
#include <shared_mutex>
#include <mutex>
#include <chrono>
#include <iostream>

int main() {
    std::shared_timed_mutex mutex;

    const auto deadline =
        std::chrono::steady_clock::now() +
        std::chrono::milliseconds(200);

    std::unique_lock<std::shared_timed_mutex> lock(
        mutex, std::defer_lock
    );

    if (lock.try_lock_until(deadline)) {
        std::cout << "Lock acquired\n";
    } else {
        std::cout << "Deadline reached\n";
    }

    return 0;
}
```

Using `std::chrono::steady_clock` to calculate a deadline is generally appropriate because this clock is designed to measure elapsed time and does not move backward or forward when the system clock changes.

### Using Timed Shared Locking

Timed operations are also available for shared locks.

```
#include <shared_mutex>
#include <mutex>
#include <chrono>
#include <iostream>

int main() {
    std::shared_timed_mutex mutex;

    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::defer_lock
    );

    if (lock.try_lock_for(std::chrono::milliseconds(100))) {
        std::cout << "Shared lock acquired\n";
    } else {
        std::cout << "Shared lock unavailable\n";
    }

    return 0;
}
```

In this example, the thread attempts to acquire a shared lock. If another thread holds an exclusive lock, or the lock cannot be acquired within the specified duration, the operation fails.

## Applications in Read-Heavy Data Structures

One of the most common applications of a shared mutex is protecting data structures in which read operations significantly outnumber write operations.

For example, a registry can store information that multiple threads read while updates occur only occasionally.

```
#include <shared_mutex>
#include <mutex>
#include <unordered_map>
#include <string>
#include <optional>

class UserRegistry {
public:
    void setUser(int id, const std::string& name) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        users_[id] = name;
    }

    std::optional<std::string> getUser(int id) const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);

        const auto it = users_.find(id);
        if (it == users_.end()) {
            return std::nullopt;
        }

        return it->second;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    std::unordered_map<int, std::string> users_;
};
```

This example uses `std::optional`, which has been available since C++17. Therefore, the code is not fully compatible with C++14, even though the main `std::shared_timed_mutex` class is available in C++14.

For a C++14 implementation, the lookup result can instead be represented using a `bool` return value and an output parameter:

```
#include <shared_mutex>
#include <mutex>
#include <unordered_map>
#include <string>

class UserRegistry {
public:
    void setUser(int id, const std::string& name) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        users_[id] = name;
    }

    bool getUser(int id, std::string& result) const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);

        const auto it = users_.find(id);
        if (it == users_.end()) {
            return false;
        }

        result = it->second;
        return true;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    std::unordered_map<int, std::string> users_;
};
```

In this design, the read operation accesses the internal data while holding a shared lock and returns a copy of the result to the caller. Consequently, the caller does not need to retain the internal lock to use the returned value.

## Important Limitations and Considerations

### Starvation

In some implementations, writer threads may have to wait for a long time because new readers continue to acquire shared locks. Depending on the implementation's locking policy, the reverse situation may also occur.

The C++ standard does not provide a general guarantee of fair scheduling or starvation prevention. Therefore, systems that require predictable latency should evaluate the behavior of their target implementation and their scheduling requirements separately.

### Deadlocks

As with other mutexes, incorrect use of `std::shared_timed_mutex` can lead to deadlocks.

For example, if a thread acquires a shared lock and then attempts to acquire an exclusive lock on the same mutex without releasing its shared ownership, it may wait for a condition that it is itself preventing from occurring.

In general, you should not assume that this mutex supports a direct, atomic upgrade from shared ownership to exclusive ownership.

### Data Lifetime Management

Locking protects data only when all relevant accesses follow the same synchronization contract.

If a pointer or reference to internal data is returned to the caller and used after the lock has been released, subsequent access may occur without protection. In such a situation, the existence of a mutex alone does not guarantee memory safety.

### Performance Overhead

A shared mutex is not necessarily faster than `std::mutex`. The overhead of managing multiple shared owners, lock contention, and scheduling behavior can make a conventional mutex perform better under some workloads.

Therefore, the choice should be based on actual measurements, the read/write access pattern, the duration of lock ownership, and the number of threads.

## Comparing `std::shared_timed_mutex` with `std::mutex`

The `std::mutex` class is designed for exclusive synchronization and has a simpler interface. In contrast, `std::shared_timed_mutex` also supports shared locking and timed operations.

| Criterion                     | `std::mutex`                 | `std::shared_timed_mutex` |
| ----------------------------- | ---------------------------- | ------------------------- |
| Exclusive locking             | Yes                          | Yes                       |
| Shared locking                | No                           | Yes                       |
| Timed operations              | No                           | Yes                       |
| Usage complexity              | Lower                        | Higher                    |
| Concurrent reading            | Requires exclusive ownership | Supported                 |
| Suitability for all workloads | Depends on requirements      | Depends on requirements   |

If a data structure is frequently modified or its protected sections are very short, `std::mutex` may be the more suitable option. If multiple threads frequently read data and writes are relatively rare, evaluating a shared mutex is reasonable.

## Practical Guidelines for Professional Use

- Use `std::unique_lock` or, for simpler cases, `std::lock_guard` to manage ordinary exclusive locking.
- Use `std::shared_lock` to manage shared locking.
- For timeouts, use the mutex's timed operations or the corresponding methods provided by `std::unique_lock` and `std::shared_lock`.
- Avoid holding locks during I/O operations, network calls, or lengthy processing unless necessary.
- To prevent deadlocks, establish a consistent lock acquisition order throughout the application.
- Run benchmarks that reflect the application's actual workload before deciding to use a shared mutex.
- Instead of relying on assumptions about fairness, evaluate scheduling behavior and the system's latency requirements.

## Summary

The `std::shared_timed_mutex` class, introduced in C++14, is a synchronization tool for managing access to shared data. It supports both exclusive and shared locking, along with the ability to attempt lock acquisition with a time limit.

This class can be useful for data structures with many concurrent readers and relatively few writers. However, its suitability should be evaluated based on the application's actual requirements, synchronization overhead, and measured performance.

In modern C++ applications, RAII utilities such as `std::unique_lock` and `std::shared_lock` help manage lock lifetimes correctly and reduce the risk of errors caused by locks not being released.

---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>