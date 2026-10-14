<div align="center">

[🇺🇸 English](./shared_lock.md) | [🇮🇷 فارسی](../../fa/cpp14/shared_lock.md)

</div>

---

# A Comprehensive Guide to `std::shared_lock` in C++14

## Introduction and Purpose of `std::shared_lock`

`std::shared_lock` is a Standard Library utility for managing shared locks in multithreaded programs, available since C++14. It allows a programmer to manage a mutex that supports shared locking using RAII (Resource Acquisition Is Initialization). This means that lock ownership is associated with the lifetime of an object, and the lock is released automatically when the object is destroyed.

The primary use of this class is to manage concurrent access to data that multiple threads are allowed to read, while ensuring that modifications are performed under an exclusive lock.

For example, in a cache, multiple threads may search for a key simultaneously while another thread occasionally inserts new information or removes existing entries. In such situations, a shared lock can allow multiple read operations to proceed concurrently without compromising the safety of write operations.

An important distinction is that `std::shared_lock` is not itself a mutex. It is a utility for managing lock ownership. It is commonly used with `std::shared_mutex`, which was introduced in C++17. In C++14, this behavior requires a shared mutex that satisfies the relevant Standard Library named requirements, such as `std::shared_timed_mutex`.

## Understanding Shared and Exclusive Locks

### The Difference Between Concurrent Reading and Writing

In multithreaded software design, two types of access are important:

- Shared access: Multiple threads can hold shared locks on the same mutex simultaneously.
- Exclusive access: Only one thread can hold an exclusive lock at a time, and exclusive ownership is incompatible with both shared and other exclusive ownership.

This mechanism enables the implementation of the Reader-Writer Lock pattern, in which readers can operate concurrently, but a writer must wait for conflicting access to finish.

The following table illustrates the typical behavior of a shared mutex:

| Current state    | Shared lock request | Exclusive lock request |
| ---------------- | ------------------- | ---------------------- |
| No owners        | Can proceed         | Can proceed            |
| One reader       | Usually can proceed | Must wait              |
| Multiple readers | Usually can proceed | Must wait              |
| One writer       | Must wait           | Must wait              |

The word "usually" is important because scheduling policies, fairness between readers and writers, and starvation prevention depend on the mutex implementation.

### Comparing `std::shared_lock` and `std::unique_lock`

Both `std::shared_lock` and `std::unique_lock` are RAII utilities for managing locks, but they manage different types of ownership.

| Feature                        | `std::shared_lock` | `std::unique_lock` |
| ------------------------------ | ------------------ | ------------------ |
| Ownership type                 | Shared             | Exclusive          |
| Can coexist with other readers | Yes                | No                 |
| Suitable for read operations   | Yes                | Yes                |
| Suitable for modifying data    | No, not by itself  | Yes                |
| Deferred locking               | Yes                | Yes                |
| Move ownership                 | Yes                | Yes                |
| Automatic release              | Yes                | Yes                |

Note that the name `unique_lock` does not mean that the lock object itself must be unique. It indicates that the object manages exclusive ownership of the mutex.

### Comparing `std::shared_lock` with `std::lock_guard`

`std::lock_guard` is a simpler RAII utility. It acquires a lock when constructed and releases it when destroyed, but it does not provide features such as ownership transfer, deferred locking, and explicit ownership-state management, which are available in `std::unique_lock` and `std::shared_lock`.

Additionally, `std::lock_guard` normally acquires an exclusive lock, so it is not a direct replacement for `std::shared_lock` when shared locking is required.

## Choosing a Suitable Mutex in C++14

### Using `std::shared_timed_mutex`

In C++14, `std::shared_timed_mutex` is a standard choice for implementing shared locking. In addition to exclusive locking operations, it provides shared locking and timed lock-acquisition attempts.

To use it, include the `<shared_mutex>` header.

```
#include <shared_mutex>

std::shared_timed_mutex mutex;
```

In this version of the standard, the following code is also valid and automatically acquires and releases a shared lock:

```
#include <iostream>
#include <shared_mutex>

class Configuration {
public:
    int getValue() const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        return value_;
    }

    void setValue(int value) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        value_ = value;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    int value_ = 0;
};
```

In this example, `mutex_` is declared `mutable` so that a `const` member function can lock it. The getter uses shared ownership, while the setter uses exclusive ownership.

Note that `std::unique_lock` requires the `<mutex>` header. The complete, self-contained version of this example is:

```
#include <iostream>
#include <mutex>
#include <shared_mutex>

class Configuration {
public:
    int getValue() const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        return value_;
    }

    void setValue(int value) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        value_ = value;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    int value_ = 0;
};

int main() {
    Configuration config;

    config.setValue(42);
    std::cout << config.getValue() << '\n';
}
```

C++17 introduced `std::shared_mutex`. This type supports shared and exclusive locking but does not provide timed locking operations. Therefore, when using `std::shared_lock` in C++14, choose a mutex type appropriate for that standard version.

## Constructing a Lock and Acquiring Ownership

`std::shared_lock` provides several ways to manage when a lock is acquired. The appropriate approach depends on whether the operation must acquire the lock immediately, acquire it later, or proceed without waiting if the mutex is busy.

### Acquiring a Lock Immediately

The simplest approach is to pass a reference to the mutex to the constructor:

```
std::shared_timed_mutex mutex;

void readData() {
    std::shared_lock<std::shared_timed_mutex> lock(mutex);

    // Access shared data here.
}
```

In this case, the constructor calls `lock_shared()` on the mutex. If another thread holds an exclusive lock, the call waits until shared ownership becomes available.

When `readData()` exits, the `lock` object is destroyed, and the shared lock is released if the object owns it.

### Deferred Lock Acquisition

Sometimes, an object that manages a lock must be created before the lock is actually acquired. The `std::defer_lock` tag supports this:

```
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void readData() {
    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::defer_lock
    );

    // Perform preparatory work here.

    lock.lock();

    // Access shared data here.
}
```

In this example, constructing the object does not acquire a lock. Acquisition occurs only when `lock()` is called.

This approach is useful when the program's control flow requires locking at a specific point. However, avoid performing lengthy operations or processing data that depends on the protected state before acquiring the lock.

### Attempting to Acquire a Lock Without Waiting

If the program must not wait for a lock, it can use `std::try_to_lock`:

```
#include <iostream>
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void tryReadData() {
    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::try_to_lock
    );

    if (lock.owns_lock()) {
        std::cout << "Lock acquired\n";
        // Access shared data here.
    } else {
        std::cout << "Lock unavailable\n";
    }
}
```

With this approach, the constructor attempts to acquire a shared lock and returns without waiting for the mutex to become available. Success can be checked using `owns_lock()`.

A failed attempt does not necessarily mean that a writer is currently active. The failure may also depend on the mutex's implementation and the current contention between threads.

### Acquiring a Lock with a Timeout

An important capability of `std::shared_lock` in C++14 is support for timed lock acquisition. This is useful for applications that cannot wait indefinitely.

The following example uses a timed constructor:

```
#include <chrono>
#include <iostream>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void readWithTimeout() {
    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::chrono::milliseconds(100)
    );

    if (lock.owns_lock()) {
        std::cout << "Lock acquired\n";
        // Access shared data here.
    } else {
        std::cout << "Timeout or lock acquisition failure\n";
    }
}
```

The `std::chrono_literals` namespace and suffixes such as `100ms` are available in C++14. However, this example uses `std::chrono::milliseconds(100)` for straightforward compatibility across different environments.

If you prefer duration suffixes, the following version is also valid:

```
#include <chrono>
#include <mutex>
#include <shared_mutex>

using namespace std::chrono_literals;

std::shared_timed_mutex mutex;

void readWithTimeout() {
    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, 100ms
    );
}
```

Keep in mind that specifying a duration does not guarantee that the function will return at precisely that moment. Operating system scheduling and execution conditions may cause additional delays.

### Using an Absolute Deadline

When a specific deadline is required, `std::chrono::time_point` is often more appropriate. The timed constructor can accept an absolute time point:

```
#include <chrono>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void readUntilDeadline() {
    const auto deadline =
        std::chrono::steady_clock::now() +
        std::chrono::milliseconds(200);

    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, deadline
    );

    if (lock.owns_lock()) {
        // Access shared data here.
    }
}
```

In this example, `std::chrono::steady_clock` is suitable for measuring elapsed time because it is designed to advance steadily and is not affected by changes to the system's wall clock.

## Understanding the Main `std::shared_lock` Operations

In addition to its constructors and destructor, `std::shared_lock` provides operations for acquiring and releasing locks, transferring ownership, and checking the current ownership state.

| Method or operation | Purpose                                                 |
| ------------------- | ------------------------------------------------------- |
| `lock()`            | Acquires a shared lock, blocking if necessary           |
| `try_lock()`        | Attempts to acquire a lock without waiting              |
| `try_lock_for()`    | Attempts to acquire a lock for a specified duration     |
| `try_lock_until()`  | Attempts to acquire a lock until a deadline             |
| `unlock()`          | Releases the shared lock                                |
| `owns_lock()`       | Checks whether the object currently owns the lock       |
| `operator bool()`   | Checks lock ownership through implicit conversion       |
| `mutex()`           | Provides access to the associated mutex pointer         |
| `release()`         | Detaches the mutex from the object without unlocking it |
| `swap()`            | Exchanges the state of two lock objects                 |

Timed operations are valid only when the mutex type supports the required timed shared-locking operations. `std::shared_timed_mutex` satisfies this requirement.

### Manually Acquiring and Releasing a Lock

Although RAII is the recommended approach, it is sometimes necessary to release a lock before leaving its scope:

```
#include <iostream>
#include <shared_mutex>

std::shared_timed_mutex mutex;
int value = 42;

void readValue() {
    std::shared_lock<std::shared_timed_mutex> lock(mutex);

    const int localValue = value;

    lock.unlock();

    std::cout << localValue << '\n';
}
```

In this example, the required value is copied into a local variable before the lock is released. The output operation therefore does not need to hold the lock.

After `unlock()`, the `lock` object remains associated with the mutex but no longer owns the lock. If locking is needed again, `lock()` or `try_lock()` can be called.

Calling `unlock()` is valid only when the object actually owns the lock. Attempting to unlock a lock that the object does not own results in undefined behavior.

### Checking Ownership with `owns_lock()`

To manage different execution paths correctly, explicitly checking the lock's state is often useful:

```
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void process() {
    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::defer_lock
    );

    if (!lock.try_lock()) {
        return;
    }

    // The shared lock is owned here.
}
```

This pattern prevents the program from continuing into a critical section when lock acquisition has failed.

### Understanding the Difference Between `release()` and `unlock()`

These operations behave differently and should not be confused.

- `unlock()` releases shared ownership but leaves the lock object associated with the mutex.
- `release()` detaches the mutex pointer from the lock object without releasing the lock itself.

The following example demonstrates how `release()` transfers responsibility for managing the lock to the calling code:

```
#include <shared_mutex>

std::shared_timed_mutex mutex;

void transferResponsibility() {
    std::shared_lock<std::shared_timed_mutex> lock(mutex);

    std::shared_timed_mutex* rawMutex = lock.release();

    rawMutex->unlock_shared();
}
```

This example illustrates the behavior of `release()` only and is not recommended for ordinary application code because responsibility for releasing the lock has been transferred to the caller.

If the lock is not released, other threads may wait indefinitely. Incorrect use of the detached pointer can also result in undefined behavior.

## Managing Ownership and Move Semantics

### Why `std::shared_lock` Is Move-Only

`std::shared_lock` cannot be copied, but it supports move construction and move assignment. This design prevents two independent objects from claiming ownership of the same lock without coordination.

The following example demonstrates transferring lock ownership:

```
#include <mutex>
#include <shared_mutex>
#include <utility>

std::shared_timed_mutex mutex;

void transferLock() {
    std::shared_lock<std::shared_timed_mutex> first(mutex);

    std::shared_lock<std::shared_timed_mutex> second(
        std::move(first)
    );

    // second owns the shared lock.
    // first no longer owns the lock.
}
```

After the transfer, `second` owns the lock and releases it when it leaves scope. `first` no longer owns the lock and can be destroyed or reused for another locking operation if needed.

Using `std::move` does not itself release a lock; it enables the move operation to be selected.

### Transferring Ownership Between Functions

Moving a lock into another function can be useful when the scope responsible for managing the lock must be transferred from one function to another.

```
#include <mutex>
#include <shared_mutex>
#include <utility>

using ReadLock = std::shared_lock<std::shared_timed_mutex>;

std::shared_timed_mutex mutex;

void consumeLock(ReadLock lock) {
    // The function owns the lock for its execution.
}

void caller() {
    ReadLock lock(mutex);

    consumeLock(std::move(lock));

    // The local lock no longer owns the mutex.
}
```

This pattern is useful when the receiving function genuinely needs to own the lock. If the goal is simply to protect data while a function executes, it is usually simpler for that function to acquire the lock itself.

### Transferring Ownership Through Assignment

Move assignment is also available. If the destination already owns another lock, its previous ownership is released before the source state is transferred.

```
#include <mutex>
#include <shared_mutex>
#include <utility>

std::shared_timed_mutex firstMutex;
std::shared_timed_mutex secondMutex;

void moveAssign() {
    std::shared_lock<std::shared_timed_mutex> first(firstMutex);

    std::shared_lock<std::shared_timed_mutex> second(
        secondMutex
    );

    second = std::move(first);
}
```

In this example, move assignment releases the lock previously owned by `second` and transfers ownership of the lock held by `first` to it.

Because this operation can release one lock and transfer another, lock acquisition and release order are important in designs involving multiple mutexes.

## Understanding Specialized Constructors

### Using `std::adopt_lock`

The `std::adopt_lock` tag is intended for cases where the current thread has already acquired a shared lock on a mutex and wants to transfer responsibility for managing that lock to a `std::shared_lock` object.

```
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void adoptExistingLock() {
    mutex.lock_shared();

    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::adopt_lock
    );

    // The lock object now manages the existing shared lock.
}
```

In this example, the constructor does not acquire another lock. Instead, it assumes that the current thread already owns a shared lock.

This approach requires care. If the ownership precondition is not satisfied, behavior is undefined. For this reason, directly acquiring the lock through the ordinary `std::shared_lock` constructor is safer in most applications.

### Using the Constructor Without a Mutex

It is also possible to create an empty lock object:

```
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void deferredInitialization() {
    std::shared_lock<std::shared_timed_mutex> lock;

    lock = std::shared_lock<std::shared_timed_mutex>(
        mutex, std::defer_lock
    );

    lock.lock();
}
```

An empty lock object has neither an associated mutex nor lock ownership. This capability is useful for dynamic locking strategies and structures in which the mutex to use is not known in advance.

## Practical Application: Designing a Cache

### Implementing a Cache with Concurrent Reads

A common use case for shared locking is protecting data structures that are read much more frequently than they are modified.

The following example implements a simple cache using `std::unordered_map`. Multiple threads can search for keys concurrently, while insertion and removal operations use an exclusive lock.

```
#include <mutex>
#include <optional>
#include <shared_mutex>
#include <string>
#include <unordered_map>

class Cache {
public:
    std::optional<std::string> get(
        const std::string& key
    ) const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);

        const auto it = data_.find(key);

        if (it == data_.end()) {
            return std::nullopt;
        }

        return it->second;
    }

    void set(std::string key, std::string value) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);

        data_[std::move(key)] = std::move(value);
    }

    bool erase(const std::string& key) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);

        return data_.erase(key) != 0;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    std::unordered_map<std::string, std::string> data_;
};
```

This example illustrates several important design principles:

- Lookup operations use shared locking, allowing multiple threads to call `find()` on the map concurrently.
- Insertion and removal operations use exclusive locking, preventing modifications to the map's internal structure from conflicting with concurrent lookups.
- The `get` method returns an independent copy of the value rather than a reference to data stored internally in the cache.
- The `get` method uses `std::optional`, which was introduced in C++17. Consequently, this version requires C++17 or later rather than C++14.

For a C++14-compatible version, the lookup result can be communicated through an output parameter and a `bool` return value:

```
#include <mutex>
#include <shared_mutex>
#include <string>
#include <unordered_map>
#include <utility>

class Cache {
public:
    bool get(const std::string& key, std::string& result) const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);

        const auto it = data_.find(key);

        if (it == data_.end()) {
            return false;
        }

        result = it->second;
        return true;
    }

    void set(std::string key, std::string value) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);

        data_[std::move(key)] = std::move(value);
    }

private:
    mutable std::shared_timed_mutex mutex_;
    std::unordered_map<std::string, std::string> data_;
};
```

This version uses only features available in C++14.

### Why Returning a Reference Can Be Dangerous

If a cache returns a reference or pointer to its internal data and releases the lock before the caller uses it, another thread may modify or erase that data.

For example, having a lock inside `get` does not make a returned reference safe by itself:

```
const std::string* getUnsafe(const std::string& key) const {
    std::shared_lock<std::shared_timed_mutex> lock(mutex_);

    const auto it = data_.find(key);

    if (it == data_.end()) {
        return nullptr;
    }

    return &it->second;
}
```

This function is an example of an unsafe design. When the function returns, the lock is released. Using the returned pointer without another protection mechanism may therefore conflict with concurrent modification or deletion of the element.

For cache implementations, consider one of the following approaches:

- Return an independent copy of the data.
- Use an owning handle that controls the lifetime of the lock.
- Use a data structure with an explicitly defined ownership and lifetime policy.
- Design the API so that the required operation is performed while the lock is held.

The right choice depends on copying costs, value sizes, and performance requirements.

## Important Considerations for Thread Safety

### Protecting All Conflicting Accesses

To prevent data races, all conflicting accesses to a shared object must use a consistent synchronization mechanism.

If one thread reads data under a `std::shared_lock`, but another thread modifies the same data without a lock, the presence of the first thread's lock is not sufficient to ensure safety.

```
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;
int value = 0;

void safeRead() {
    std::shared_lock<std::shared_timed_mutex> lock(mutex);
    const int localValue = value;
}

void safeWrite() {
    std::unique_lock<std::shared_timed_mutex> lock(mutex);
    value = 42;
}
```

In this example, both reading and writing use the same mutex, and conflicting accesses to `value` are properly synchronized.

By contrast, if `safeWrite` modifies the value directly while another thread is reading it, a data race may occur.

### Using Locks in `const` Member Functions

In many classes, `const` member functions do not modify the object's logical state, but they still require synchronization to read shared data safely.

Declaring the mutex as `mutable` is a common pattern in these cases:

```
#include <shared_mutex>

class Counter {
public:
    int get() const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        return value_;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    int value_ = 0;
};
```

Using `mutable` does not mean that protected data can be modified without a lock. It simply allows the mutex member to be managed from a `const` member function.

### Limitations of Recursive Lock Acquisition

Standard shared mutexes, including `std::shared_timed_mutex`, are not recursive. A thread must not reacquire shared or exclusive ownership of the same mutex without respecting the mutex's ownership rules.

For example, if one method acquires a shared lock and then calls another method that attempts to lock the same mutex again, the program may deadlock or exhibit undefined behavior.

```
#include <mutex>
#include <shared_mutex>

class Data {
public:
    void outer() {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        inner();
    }

    void inner() {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        // Access data here.
    }

private:
    std::shared_timed_mutex mutex_;
};
```

This example demonstrates a problematic design because both `outer()` and `inner()` attempt to acquire a shared lock on the same mutex. Lock ownership should be clearly defined at the API level. For example, an internal method can be designed to assume that the lock has already been acquired rather than acquiring another lock itself.

## Using Condition Variables

### Waiting for a State Change

In some applications, a thread must wait until a condition becomes true, such as waiting for a queue to become nonempty.

`std::condition_variable` requires a `std::unique_lock<std::mutex>` and does not work directly with `std::shared_lock`. However, `std::condition_variable_any` can work with compatible lock types, including `std::shared_lock`.

The following example demonstrates waiting under a shared lock until a condition becomes true:

```
#include <condition_variable>
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;
std::condition_variable_any condition;
bool ready = false;

void waitForReady() {
    std::shared_lock<std::shared_timed_mutex> lock(mutex);

    condition.wait(lock, [] {
        return ready;
    });

    // The predicate is true while the shared lock is held.
}

void setReady() {
    {
        std::unique_lock<std::shared_timed_mutex> lock(mutex);
        ready = true;
    }

    condition.notify_all();
}
```

In this example, `wait()` releases the lock while waiting and reacquires it before returning. Using a predicate also ensures that spurious wakeups are handled correctly.

An important consideration is that `std::condition_variable_any` may introduce additional overhead and complexity compared with `std::condition_variable`. Therefore, if exclusive locking is sufficient, `std::condition_variable` with `std::unique_lock` is usually the simpler choice.

## Common Problems and Solutions

### Holding a Lock for Too Long

Shared locking allows readers to operate concurrently, but writers must still wait until all conflicting shared ownership has been released.

If a thread holds a lock during a lengthy operation, a network request, or input/output, writers may experience substantial delays.

A suitable approach is to copy the required data while holding the lock and then release it before continuing. This is safe only when copying the data and proceeding without the lock preserve the required semantics.

### Assuming Shared Locking Is Always Faster

Using a shared lock does not necessarily improve performance. A shared mutex may incur additional costs for tracking readers and coordinating access with writers compared with a simple `std::mutex`.

If the critical section is very short or the number of concurrent readers is small, a regular mutex may perform better.

The best choice should be determined through measurements in the actual target environment rather than by assuming that more concurrent readers automatically mean better performance.

### Writer Starvation

Depending on the implementation, a continuous stream of readers may cause an exclusive lock request to experience prolonged delays. The standard does not specify a universal, portable fairness policy between readers and writers.

If writer response time is important, measure the behavior of the mutex in the target environment and consider alternative scheduling policies or designs when necessary.

### Attempting to Upgrade a Lock

`std::shared_lock` does not provide an operation for directly converting shared ownership into exclusive ownership.

If a thread wants to modify data after reading it, it must not simply retain a shared lock and then attempt to acquire an exclusive lock on the same mutex. Doing so may result in deadlock or undefined behavior.

The usual pattern is to release the shared lock and then acquire an exclusive lock. However, this creates a window during which the data may change. Consequently, the relevant condition must be checked again after the exclusive lock has been acquired.

### Forgetting the Mutex Lifetime

A `std::shared_lock` object retains an internal pointer or association with its mutex. The mutex must therefore remain valid throughout the lifetime of the lock object.

Destroying a mutex while a thread still owns it, or while a lock-management object still depends on it, can result in undefined behavior.

In class design, it is usually best to declare the mutex as a member of the object containing the protected data. This simplifies lifetime management.

## Comparing Alternative Approaches

### Choosing Between `std::mutex` and a Shared Mutex

| Feature                            | `std::mutex`              | `std::shared_timed_mutex`                     |
| ---------------------------------- | ------------------------- | --------------------------------------------- |
| Exclusive ownership                | Yes                       | Yes                                           |
| Shared ownership                   | No                        | Yes                                           |
| Concurrent readers                 | No                        | Yes                                           |
| Standard timed-locking support     | No                        | Yes                                           |
| Potential synchronization overhead | Usually lower             | May be higher                                 |
| Suitable for                       | General critical sections | Workloads with substantial concurrent reading |

If only one thread should access the data at a time, `std::mutex` is usually a suitable and simpler option.

If there are many readers and the read operations do not modify shared state, shared locking may be beneficial. However, its performance advantages should be measured.

### Comparing `std::shared_mutex` in C++17

`std::shared_mutex` was introduced in C++17 and supports shared and exclusive ownership. Unlike `std::shared_timed_mutex`, it does not provide timed locking operations.

Consequently, choosing between these types depends on the application's requirements and the target C++ standard:

- In C++14, `std::shared_timed_mutex` is a suitable standard mutex for using `std::shared_lock` with shared locking.
- In C++17 and later, `std::shared_mutex` is also a suitable option when timed locking is unnecessary.
- If the application requires a specific fairness policy, writer priority, or scheduling behavior, those requirements must be evaluated separately. The mutex type's name alone does not guarantee them.

## Compilation and Compatibility

To compile a program that uses `std::shared_lock` and `std::shared_timed_mutex`, the compiler and Standard Library must support C++14 and the threading facilities.

For example, in a compatible GCC environment, use the following command:

```
g++ -std=c++14 -pthread main.cpp -o main
```

The `-std=c++14` option selects the language standard, while `-pthread` enables the required thread support in environments where it is applicable.

Other environments may use different mechanisms to enable thread support, so check the project configuration and compiler documentation as well.

## Final Summary

`std::shared_lock` is a C++14 utility for managing shared mutex ownership. Through RAII, ownership transfer, and multiple lock-acquisition strategies, it simplifies synchronization management.

The following principles are particularly important when using this class professionally:

- Use a compatible mutex type, such as `std::shared_timed_mutex`, in C++14.
- Use shared locks for concurrent reading and exclusive locks for modifying data.
- Ensure that all conflicting data accesses use a consistent synchronization mechanism.
- Avoid recursively acquiring a non-recursive mutex and attempting to upgrade a shared lock directly.
- Ensure that the mutex outlives every lock-management object that depends on it.
- Carefully design the lifetime and access guarantees of APIs that return references or pointers to internal data.
- Benchmark shared locking under realistic workloads before assuming that it improves performance.

Ultimately, `std::shared_lock` provides the greatest benefit when an application genuinely benefits from concurrent readers and its locking strategy, data lifetime, and writer behavior are all designed correctly.

## Further Reading

- [cppreference documentation for ](https://en.cppreference.com/w/cpp/thread/shared_lock)[`std::shared_lock`](https://en.cppreference.com/w/cpp/thread/shared_lock)
- [cppreference documentation for ](https://en.cppreference.com/w/cpp/thread/shared_timed_mutex)[`std::shared_timed_mutex`](https://en.cppreference.com/w/cpp/thread/shared_timed_mutex)
- [cppreference documentation for the ](https://en.cppreference.com/w/cpp/header/shared_mutex)[`<shared_mutex>`](https://en.cppreference.com/w/cpp/header/shared_mutex)[ header](https://en.cppreference.com/w/cpp/header/shared_mutex)
- [C++ standard draft: Thread Support Library](https://eel.is/c++draft/thread)

---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>