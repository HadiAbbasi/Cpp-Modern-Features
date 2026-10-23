<div align="right">

[🇺🇸 English](./unique_lock.md) | [🇮🇷 فارسی](../../fa/cpp11/unique_lock.md)

</div>

---

# unique_lock 

The **`std::unique_lock`** class in **C++11** (defined in the `<mutex>` header) is an advanced **RAII**-based wrapper for managing `std::mutex` locks.

If `std::lock_guard` is considered a "simple and rigid" lock manager, `std::unique_lock` is its "flexible and feature-rich" counterpart.

---

## Why Do We Need `std::unique_lock` When `std::lock_guard` Exists?

With `std::lock_guard`, you have no control over when the lock is acquired or released within the scope, you cannot defer locking, and you cannot transfer it between functions.

`std::unique_lock` adds the following capabilities:
1. **Manual locking and unlocking (`lock()` and `unlock()`)** mid-scope.
2. **Deferred locking** (delaying the lock acquisition).
3. **Try-lock** capabilities (attempting to lock without blocking or with a timeout).
4. **Move semantics** (transferable ownership).
5. **Full integration with `std::condition_variable`** (mandatory for its use).

---

## Basic Syntax

In its simplest form, just like `std::lock_guard`, it automatically acquires the lock in the constructor and releases it in the destructor:

```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;

void printMessage(int id) {
    // The lock is acquired upon creation and released at the end of the scope
    std::unique_lock<std::mutex> lock(mtx);
    std::cout << "Thread " << id << " is working\n";
}
```

---

## Locking Strategies

When constructing a `std::unique_lock`, you can specify its behavior by passing standard tag arguments:

### A) `std::defer_lock` (Deferred Locking)
The mutex is taken under management but **not locked immediately**. You can lock it manually later:

```cpp
std::mutex mtx;

void task() {
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
    
    // Tasks that do not require a lock...
    
    lock.lock(); // Now the lock is acquired
    // Critical Section operations
    lock.unlock(); // Manual release if needed
}
```

### B) `std::try_to_lock` (Non-Blocking Try)
Attempts to acquire the lock; if the mutex is held by another thread, it does **not** block:

```cpp
void tryTask() {
    std::unique_lock<std::mutex> lock(mtx, std::try_to_lock);

    if (lock.owns_lock()) {
        std::cout << "Lock acquired! Processing data...\n";
    } else {
        std::cout << "Mutex is busy, doing something else...\n";
    }
}
```

### C) `std::adopt_lock` (Adopting an Existing Lock)
Assumes the calling thread has **already locked** the mutex and only takes on the responsibility of releasing it:

```cpp
mtx.lock(); // Manual lock
std::unique_lock<std::mutex> lock(mtx, std::adopt_lock);
// At the end of the scope, 'lock' will release the mutex
```

---

##  Key Features of `std::unique_lock`

### 1. Manual `unlock` and `lock`
Allows you to unlock and relock multiple times within the scope, useful for shortening lock times in loops or during heavy computations:

```cpp
std::unique_lock<std::mutex> lock(mtx);

// Shared data access
shared_data.process();

lock.unlock(); // Release lock for heavy, non-critical computation
heavyNonCriticalComputation();

lock.lock(); // Re-acquire lock
shared_data.finish();
```

### 2. Move-Only Semantics
You cannot copy a `std::unique_lock`, but you can transfer its ownership using `std::move`:

```cpp
std::unique_lock<std::mutex> getLock() {
    std::unique_lock<std::mutex> lock(mtx);
    // Preparation work
    return lock; // Returns with automatic move semantics
}

void useLock() {
    std::unique_lock<std::mutex> lock = getLock(); // Ownership transferred here
    // The lock is still active!
} // Lock is released here
```

---

## Critical Use Case: Using with `std::condition_variable`

One of the primary reasons to use `std::unique_lock` is its integration with `std::condition_variable`. The `wait` method must temporarily release the lock while the thread sleeps and re-acquire it upon waking up—a task `std::lock_guard` cannot perform:

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void workerThread() {
    std::unique_lock<std::mutex> lock(mtx);
    
    // The thread waits until 'ready' becomes true.
    // 'cv' automatically unlocks the mutex while waiting and re-locks it upon waking.
    cv.wait(lock, [] { return ready; });

    std::cout << "Worker is doing work...\n";
}

void publisherThread() {
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one(); // Wake up the worker
}

int main() {
    std::thread t1(workerThread);
    std::thread t2(publisherThread);

    t2.join();
    t1.join();
}
```

## Quick Comparison: `lock_guard` vs. `unique_lock`

| Feature | `std::lock_guard` | `std::unique_lock` |
| :--- | :--- | :--- |
| **Simplicity & Speed** | Very simple, lightweight, and fast (Zero-overhead) | Slightly heavier due to maintaining lock state |
| **Manual `unlock` capability** | ❌ No | ✅ Yes |
| **Move Ownership** | ❌ No | ✅ Yes |
| **Works with `condition_variable`** | ❌ No | ✅ Yes (mandatory) |
| **Deferred Locking** | ❌ No | ✅ Yes |

---

## Summary: Comparison with `std::lock_guard`

* The performance overhead of `std::unique_lock` compared to `std::lock_guard` is negligible (due to storing a boolean flag to track lock status).
* **Conclusion:**
  * If you only need a simple lock for the entire scope $\rightarrow$ Use **`std::lock_guard`**.
  * If you need **`std::condition_variable`**, **deferred locking**, **non-blocking tries**, or **manual unlocks** $\rightarrow$ Always use **`std::unique_lock`**.


---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>