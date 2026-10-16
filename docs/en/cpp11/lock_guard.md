<div align="right">

[🇺🇸 English](./lock_guard.md) | [🇮🇷 فارسی](../../fa/cpp11/lock_guard.md)

</div>

# lock_guard
The std::lock_guard class in C++11 (defined in the `<mutex>` header) is a simple and safe tool for managing mutex locks. It is based on the well-known RAII pattern, which stands for Resource Acquisition Is Initialization.

## What Is the Problem, and Why Do We Need `lock_guard`?
In multithreaded programming, we use `std::mutex` to prevent race conditions. The traditional manual locking approach looks like this:

```cpp
std::mutex mtx;

void unsafeFunction() {
    mtx.lock();
    // Perform critical-section operations
    mtx.unlock();
}
```
### Problems with the Manual Approach
- Forgetting to unlock: The programmer may forget to call unlock().
- Exceptions: If an exception is thrown inside the critical section, the function exits before reaching mtx.unlock(). As a result, the mutex remains locked indefinitely, which can lead to a deadlock.
- Early returns: If the function contains multiple return statements, unlock() must be called before each return. This approach is error-prone and difficult to maintain.

### Philosophy and How `std::lock_guard` Works

The RAII pattern in `std::lock_guard` works as follows:

* **Upon object construction (Constructor):** The lock is acquired automatically (`mtx.lock()`).
* **Upon object destruction (Destructor):** As soon as the object goes out of scope—even due to an early `return` or a thrown exception—the lock is guaranteed to be released (`mtx.unlock()`).

## Basic Structure and Syntax
```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx; // Shared mutex variable
int counter = 0;

void increment() {
    std::lock_guard<std::mutex> lock(mtx);
    
    ++counter;
    std::cout << "Counter: " << counter << " from thread: " 
              << std::this_thread::get_id() << "\n";

}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();
}

```
## Key Features of std::lock_guard in C++11
- **Non-copyable & Non-movable**:
You cannot copy a std::lock_guard or transfer it using std::move. The ownership of the lock is strictly tied to the scope in which it was created.

- **No manual lock() or unlock() functions**:
Unlike std::unique_lock, you cannot manually unlock or relock midway through. Everything is strictly bound to the lifetime of the object.

- **Zero-overhead**:
At runtime, it incurs no additional performance cost compared to manual calls to lock and unlock.

## Fine-Grained Scope Control Using {} Blocks
Sometimes you do not want an entire function to remain locked (so other threads aren’t blocked unnecessarily). You can narrow the scope of the std::lock_guard by introducing a simple pair of curly braces:

```cpp
void processData() {
    // Heavy tasks that do not require a lock
    doHeavyPreparation();

    {
        // The lock is acquired only for updating the shared variable
        std::lock_guard<std::mutex> lock(mtx);
        sharedList.push_back(result);
    } // <-- The lock is destroyed here and the mutex is released

    // Other tasks that still do not require a lock
    logCompletion();
}
```
## **Using** std::adopt_lock
If a mutex has already been locked by the current thread, it should not be locked again (as trying to lock a standard std::mutex twice causes a deadlock). By passing the std::adopt_lock argument to std::lock_guard, you tell it: “The lock is already acquired; your only job is to release it at the end of the scope!”

An important use case for this feature is preventing deadlocks when locking two mutexes simultaneously:

```cpp
#include <mutex>

std::mutex mtxA;
std::mutex mtxB;

void safeSwap() {
    // 1. Lock both mutexes simultaneously in a deadlock-free manner
    std::lock(mtxA, mtxB);

    // 2. Delegate the unlocking responsibility to RAII using adopt_lock
    std::lock_guard<std::mutex> lockA(mtxA, std::adopt_lock);
    std::lock_guard<std::mutex> lockB(mtxB, std::adopt_lock);

    // Perform critical operations on shared data
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


> **General Rule:** Always use **`std::lock_guard`** by default, unless you need manual unlocking or need to use `std::condition_variable`, in which case you should switch to **`std::unique_lock`**.
---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>