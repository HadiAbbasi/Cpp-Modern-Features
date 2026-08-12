# `std::shared_mutex` in C++17

## Table of Contents

- [Introduction](#introduction)
- [Required Header](#required-header)
- [How it Works](#how-it-works)
- [Exclusive vs. Shared Access](#exclusive-vs-shared-access)
- [Basic Usage](#basic-usage)
- [Using `std::shared_lock` and `std::unique_lock`](#using-stdshared_lock-and-stdunique_lock)
- [Performance Consideration](#performance-consideration)
- [Common Errors](#common-errors)
- [Complete Example](#complete-example)
- [Exercises](#exercises)
- [Conclusion](#conclusion)
- [Contributors](#contributors)

---

## Introduction

`std::shared_mutex` is a synchronization primitive added in C++17 that allows for shared access to a resource. It implements the "Readers-Writer Lock" pattern, which is ideal for scenarios where multiple threads need to read data simultaneously, but only one thread needs to write to it.

Unlike `std::mutex`, which is strictly exclusive, `std::shared_mutex` supports two levels of access.

---

## Required Header

To use `std::shared_mutex`, include the `<shared_mutex>` header:

```cpp
#include <shared_mutex>
```

---

## How it Works

`std::shared_mutex` allows:
1.  **Shared Access (Reader Lock):** Multiple threads can acquire the lock simultaneously for reading.
2.  **Exclusive Access (Writer Lock):** Only one thread can acquire the lock for writing; all other readers and writers are blocked.

This effectively prevents race conditions where one thread writes while another is reading.

---

## Exclusive vs. Shared Access

- **Writer (Exclusive):** Use `std::unique_lock<std::shared_mutex>` (or `lock()`).
  - Blocks other writers.
  - Blocks other readers.
- **Reader (Shared):** Use `std::shared_lock<std::shared_mutex>` (or `lock_shared()`).
  - Blocks writers.
  - Allows other readers.

---

## Basic Usage

### Writer (Exclusive Access)
Use `std::unique_lock` when you need to modify the shared resource:

```cpp
std::shared_mutex rw_mutex;

void write_data() {
    std::unique_lock lock(rw_mutex); // Exclusive access
    // ... update shared data ...
}
```

### Reader (Shared Access)
Use `std::shared_lock` when you only need to read the shared resource:

```cpp
void read_data() {
    std::shared_lock lock(rw_mutex); // Shared access
    // ... access shared data ...
}
```

---

## Performance Consideration

Use `std::shared_mutex` when the read-to-write ratio is high. If your application writes just as frequently as it reads, the overhead of the shared lock mechanism might make a standard `std::mutex` faster.

---

## Common Errors

1. **Forgetting `shared_lock`:** Accidentally using `unique_lock` for reading defeats the purpose, causing unnecessary contention.
2. **Deadlocks:** Even with a shared mutex, if you try to recursively lock a `shared_mutex` or perform complex multi-lock acquisitions, you can still trigger deadlocks.
3. **Write Starvation:** Depending on the implementation, excessive readers might starve writers. Most modern C++ implementations provide fairness, but don't rely on it for critical timing logic.

---

## Complete Example

```cpp
#include <iostream>
#include <shared_mutex>
#include <vector>
#include <thread>

class ThreadSafeCounter {
    mutable std::shared_mutex rw_mutex;
    int value = 0;

public:
    int get() const {
        std::shared_lock lock(rw_mutex); // Readers can enter concurrently
        return value;
    }

    void increment() {
        std::unique_lock lock(rw_mutex); // Exclusive: only one writer
        value++;
    }
};

int main() {
    ThreadSafeCounter counter;
    
    // Simulate reader/writer threads
    std::thread reader1([&]() { std::cout << "Read: " << counter.get() << '
'; });
    std::thread writer1([&]() { counter.increment(); });
    
    reader1.join();
    writer1.join();
}
```

---

## Exercises

1. Implement a thread-safe cache using `std::shared_mutex` where multiple threads can search for items (read), but only one thread can add/update items (write).
2. Measure the performance difference between a `std::mutex` and a `std::shared_mutex` in a system with 90% reads and 10% writes.

---

## Conclusion

`std::shared_mutex` is a powerful tool for optimizing high-read concurrency in C++. By correctly utilizing `std::shared_lock` and `std::unique_lock`, you can significantly improve the throughput of your multi-threaded applications.

---

## 🤝 Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |  |  |  |
