<div align="center">

[🇺🇸 English](./std_counting_semaphore.md) | [🇮🇷 فارسی](../../fa/cpp20/std_counting_semaphore.md)

</div>

---

# A Comprehensive Guide to `std::counting_semaphore` in C++20

Introduced into the standard library in C++20, `std::counting_semaphore` is a counter-based synchronization primitive for threads. It is useful when a thread must wait until one or more permits become available—for example, when multiple threads need to access a resource pool with limited capacity, or when a producer needs to signal a consumer that data is ready.

Unlike `std::mutex`, which typically manages exclusive ownership of a critical section, a semaphore controls the number of available permits. This distinction makes semaphores useful for limiting concurrency and coordinating producer/consumer workflows, rather than only locking data.

## Introduction and Primary Use Cases

A semaphore is a synchronization mechanism that operates on a non-negative counter. The counter indicates how many permits are currently available:

- `acquire` takes one permit. If the counter is positive, it decrements the counter and proceeds; otherwise, the calling thread waits.
- `release` returns one or more permits and may wake waiting threads.
- `try_acquire` attempts to obtain a permit without waiting.
- Timed operations such as `try_acquire_for` and `try_acquire_until` provide bounded waiting.

For `std::counting_semaphore`, the counter does not necessarily represent the number of waiting threads or the number of physical resources. It models the number of permits that can be acquired. The application design defines what those permits mean.

## Standard Status and Required Header

This type is part of the C++20 standard library and is declared in the `<semaphore>` header. To use it, compile the project with C++20 support.

```cpp
#include <semaphore>
#include <thread>
#include <iostream>

int main() {
    std::counting_semaphore<3> permits{3};

    permits.acquire();
    std::cout << "Permit acquired\n";
    permits.release();
}
```

In this example, the counter starts with three permits. Acquiring one reduces the count to two, and releasing a permit increases it again.

Actual support also depends on the standard library and toolchain version. Selecting C++20 language mode alone does not guarantee that every library feature is available in an older implementation.

## Understanding the Type and Template Parameter

The general declaration is:

```cpp
template<std::ptrdiff_t LeastMaxValue = /* implementation-defined */>
class counting_semaphore;
```

The `LeastMaxValue` template parameter specifies the minimum maximum value that the implementation must support for the semaphore. It is not the initial permit count; the initial count is specified when the object is constructed.

```cpp
std::counting_semaphore<5> sem{3};
```

In this example, the initial count is three, while the template parameter is five. The implementation must support a counter capacity of at least five.

`std::counting_semaphore` is an alias for the appropriate specialization based on `LeastMaxValue`. The standard library also provides `std::binary_semaphore` as an alias for a semaphore whose supported capacity is at least one. This type is commonly used for patterns with two states: permit available or permit unavailable.

```cpp
std::binary_semaphore ready{0};
```

`std::binary_semaphore` is a semantically suitable name for a semaphore with zero or one permit, but it should not be confused with `std::mutex`. A semaphore does not impose thread ownership and is not a direct replacement for a mutex that provides exclusive data access.

## Core Operations

### Acquiring a Permit with `acquire`

The `acquire` function waits until a permit becomes available and then consumes one permit.

```cpp
#include <semaphore>
#include <thread>
#include <iostream>

int main() {
    std::counting_semaphore<2> sem{1};

    std::jthread worker([&] {
        sem.acquire();
        std::cout << "Worker acquired a permit\n";
        sem.release();
    });

    sem.acquire();
    std::cout << "Main acquired a permit\n";
    sem.release();
}
```

This program demonstrates two threads acquiring permits. The exact order of the messages is not guaranteed. In real applications, concurrent access to output or shared data should also be managed with an appropriate synchronization mechanism.

### Releasing Permits with `release`

The `release(update = 1)` function increments the counter by `update` and may wake waiting threads.

```cpp
std::counting_semaphore<4> sem{0};

// Make two permits available.
sem.release(2);
```

The `update` parameter must be positive. You must also ensure that the counter does not exceed its permitted maximum. Violating the counter limit results in undefined behavior, so `release` must not be called without a precise model of the permits.

Releasing a permit does not transfer ownership from a particular thread. By contrast, `std::mutex::unlock` must comply with the mutex's ownership rules.

### Non-Blocking Attempts with `try_acquire`

The `try_acquire` function immediately attempts to obtain a permit and returns a `bool` result.

```cpp
if (sem.try_acquire()) {
    // Use the resource protected by this permit.
    use_resource();
    sem.release();
} else {
    // No permit was acquired.
    do_other_work();
}
```

If the function returns `false`, no permit was acquired. Success does not guarantee fair ordering or thread prioritization. A call may also fail even when a permit appears to be available from the application's perspective, due to concurrency details; code must not rely on fairness or a particular order.

### Timed Waiting

`std::counting_semaphore` does not support the timed operations `try_acquire_for` and `try_acquire_until`; these operations are not part of `std::counting_semaphore`. The standard semaphore API includes `acquire`, `release`, and `try_acquire`. If timed waiting is required, use another mechanism, such as `std::condition_variable`, or design a timeout-based approach. The appropriate alternative depends on whether the program needs to evaluate a condition more complex than permit availability.

## Initial Count and Modeling Permits

The initial count is specified when the object is constructed and must be non-negative and within the supported range. An initial value of zero means no permits are available, so acquiring threads must wait until a permit is released.

```cpp
std::counting_semaphore<10> slots{10};
std::counting_semaphore<10> items{0};
```

In a producer/consumer queue with a capacity of ten, `slots` can model the number of empty slots, while `items` can model the number of ready items. However, a semaphore does not manage the queue's memory or data structure by itself; access to the queue must be made safe separately.

## Practical Example: Limiting Concurrent Operations

Suppose an application is allowed to process at most three requests concurrently. A semaphore with three permits can limit the number of active operations.

```cpp
#include <semaphore>
#include <thread>
#include <vector>
#include <iostream>

std::counting_semaphore<3> request_slots{3};

void process_request(int id) {
    request_slots.acquire();

    // The permit must be returned even if processing throws.
    try {
        std::cout << "Processing request " << id << '\n';
        // Perform the request work here.
    } catch (...) {
        request_slots.release();
        throw;
    }

    request_slots.release();
}

int main() {
    std::vector<std::jthread> workers;
    for (int i = 0; i < 8; ++i) {
        workers.emplace_back([i] {
            process_request(i);
        });
    }
}
```

This example shows how a semaphore can limit the number of concurrent operations. In production code, it is important to release the permit on every exit path. The code uses `try`/`catch` to illustrate this requirement; a safer approach is usually an RAII-based guard that releases the permit automatically on both normal and exceptional exits.

### Managing Permits with RAII

Because a semaphore does not automatically return a permit when a scope exits, you can create a small guard. The guard should only be constructed after a permit has actually been acquired.

```cpp
#include <semaphore>

class semaphore_guard {
public:
    explicit semaphore_guard(std::counting_semaphore<3>& sem)
        : sem_(sem) {
        sem_.acquire();
    }

    semaphore_guard(const semaphore_guard&) = delete;
    semaphore_guard& operator=(const semaphore_guard&) = delete;

    ~semaphore_guard() {
        sem_.release();
    }

private:
    std::counting_semaphore<3>& sem_;
};

std::counting_semaphore<3> slots{3};

void do_work() {
    semaphore_guard guard{slots};
    // The permit is returned when guard leaves scope.
    perform_work();
}
```

This guard is designed for blocking acquisition. If permit acquisition must be cancellable or support a timeout, design a guard API that matches that behavior. The semaphore must also outlive the guard; holding a reference to a semaphore that is destroyed first is a lifetime error.

## Practical Example: Producer/Consumer Synchronization

In the producer/consumer pattern, a semaphore can represent the number of ready items. The following simple example uses a queue protected by a mutex and a semaphore to signal the number of available items.

```cpp
#include <condition_variable>
#include <deque>
#include <mutex>
#include <semaphore>
#include <thread>
#include <iostream>

class work_queue {
public:
    void push(int value) {
        {
            std::lock_guard<std::mutex> lock(mutex_);
            queue_.push_back(value);
        }
        items_.release();
    }

    int pop() {
        items_.acquire();

        std::lock_guard<std::mutex> lock(mutex_);
        int value = queue_.front();
        queue_.pop_front();
        return value;
    }

private:
    std::mutex mutex_;
    std::deque<int> queue_;
    std::counting_semaphore<> items_{0};
};

int main() {
    work_queue queue;

    std::jthread consumer([&] {
        for (int i = 0; i < 3; ++i) {
            std::cout << queue.pop() << '\n';
        }
    });

    queue.push(10);
    queue.push(20);
    queue.push(30);
}
```

In this design, data is first added to the queue while holding the mutex, and then a permit is released. The consumer first acquires a permit and then removes an item from the queue. This ordering matters: releasing a permit before inserting the data could allow the consumer to acquire a permit when the queue does not yet contain the corresponding item.

This example is educational and does not implement a shutdown or queue-closing policy. A real system should provide a way to terminate consumers, handle errors, and prevent indefinite waiting during shutdown. A `condition_variable` may also be a suitable choice, especially if the consumer must check multiple conditions or the queue needs a more complex shutdown policy.

## Relationship to the Memory Model and Concurrency

Semaphore operations do more than change the counter: successful `release` operations and corresponding acquisitions can also be used for memory synchronization. A `release` operation synchronizes preceding memory effects with a successful acquisition that consumes the released permit. This allows data prepared before the release to become visible to another thread after it acquires the permit, provided the design relies on the standard synchronization relationship.

Nevertheless, a semaphore is not a replacement for a mutex when protecting a shared data structure. If multiple threads modify a `std::deque` concurrently, merely having a semaphore with a positive count does not necessarily make access to the queue safe. Shared data requires a complete synchronization protocol, usually combining a semaphore with a mutex or using a data structure that is itself thread-safe.

## Comparison with Mutexes and Condition Variables

| Feature | `std::counting_semaphore` | `std::mutex` | `std::condition_variable` |
|---|---|---|---|
| Core concept | Permit counter | Exclusive lock ownership | Waiting for a condition to change |
| Initial state | Initial permit count | Locked or unlocked | No permit counter |
| Waiting | Until a permit is acquired | Until the lock is acquired | Until notified and the predicate is checked again |
| Thread ownership | None | Yes | Used with a mutex |
| Typical use | Capacity limits, counting events | Protecting shared data | Waiting for a complex condition |
| Persistent notification | A released permit can remain available | Not applicable | A notification is not stored by itself |

Choose the mechanism based on the problem's model. A mutex is usually the natural choice for protecting a shared map. A semaphore is suitable for limiting the number of simultaneous connections to a service. A `condition_variable` often expresses the intent more clearly when waiting for multiple conditions on an object's state.

## Limitations and Important Design Considerations

### Preventing Excessive Releases

Every `release` must correspond to a permit modeled by the design. Repeated releases or releasing more permits than the defined capacity can move the counter outside its valid range. Make each acquisition and release path traceable in the design, and document the invariants governing the counter.

### No Fairness Guarantee

The standard does not guarantee that waiting threads acquire permits in exactly the order in which they arrived. Do not rely on a semaphore for FIFO ordering, prioritization, or guaranteed prevention of starvation unless another layer implements that policy.

### Deadlock Risk

If all permits are acquired and no execution path releases them, other threads may wait forever. Early errors, exceptions, multiple return paths, and shutdown must be accounted for in the release design. RAII can reduce the risk of forgetting to call `release`.

### Object Lifetime and Program Shutdown

The semaphore must remain alive until all operations that access it have finished. Destroying a semaphore while another thread is waiting on it or still using it is not a valid design. Stop and join the relevant threads before destroying the object, and design the shutdown path explicitly.

### Stopping and Cancelling Operations

`acquire` does not provide a cancellation or timeout interface. If the application must interrupt waiting during shutdown, design a separate protocol—for example, shutdown permits, an independent stop condition, or another mechanism that correctly wakes all waiters. Such approaches must ensure that artificial permits are not mistaken for real items.

### Performance and Implementation Details

An implementation may use operating-system facilities, hardware primitives, or a combination of them. The standard does not mandate a particular internal implementation. Therefore, do not make general assumptions about lock-free behavior, the exact cost of each operation, or whether a semaphore is always faster than a `condition_variable`; measure performance under the actual workload.

## Common Mistakes

- **Confusing a semaphore with a lock:** Holding a permit does not automatically protect every access to shared data.
- **Calling `release` without a corresponding permit:** This may violate the counter invariant and can result in undefined behavior.
- **Forgetting to release during exception handling:** Use RAII to manage the permit's lifetime.
- **Assuming FIFO behavior:** Do not assume a guaranteed order for waking or acquiring permits.
- **Releasing a permit before data is ready:** Update the shared state first, then release the permit.
- **Destroying a semaphore before its threads finish:** The synchronization object's lifetime must cover all its users.
- **Assuming timeout support:** The standard `std::counting_semaphore` API does not provide `try_acquire_for` or `try_acquire_until`.

## Choosing the Template Parameter and Counter Maximum

The `LeastMaxValue` parameter provides a guaranteed minimum for the counter's maximum; it is not necessarily the implementation's exact maximum. The `max()` static member reports the supported maximum. For portable designs, keep required values within the guaranteed range and do not depend on extra capacity provided by a particular library.

```cpp
#include <semaphore>
#include <iostream>

int main() {
    std::counting_semaphore<8> sem{4};
    std::cout << sem.max() << '\n';
}
```

The exact output is not necessarily eight. The returned value represents the maximum supported by that implementation, while the template parameter guarantees a minimum capacity.

## Comparison with Older Approaches

Before C++20, programs often implemented semaphore-like behavior using `std::mutex` together with `std::condition_variable`. This approach remains suitable for more complex conditions, but it leaves the counter, predicate, and waiting loop to be implemented by the programmer. `std::counting_semaphore` provides a direct, standardized abstraction for counting permits.

The following simplified example illustrates the general idea behind an older implementation. In real code, the predicate must be checked in the wait operation, and the counter must always be protected by the same mutex.

```cpp
std::mutex mutex;
std::condition_variable cv;
int permits = 0;

void acquire_legacy() {
    std::unique_lock<std::mutex> lock(mutex);
    cv.wait(lock, [] { return permits > 0; });
    --permits;
}

void release_legacy() {
    {
        std::lock_guard<std::mutex> lock(mutex);
        ++permits;
    }
    cv.notify_one();
}
```

This snippet is not a complete standalone implementation because it omits the headers and counter-limit handling. Its purpose is to show which responsibilities the standard semaphore abstraction reduces. A manual implementation must also protect against counter overflow, preserve invariants, and implement a shutdown protocol.

## Choosing the Right Tool for Real Projects

- For limiting concurrent operations or managing resources with a defined capacity, `std::counting_semaphore` is a suitable option.
- To signal the number of ready items in a queue, a semaphore can be used alongside a mutex and the queue data structure.
- For exclusive data protection, consider `std::mutex` or, where appropriate, `std::shared_mutex`.
- For waiting on a complex predicate, multiple conditions, or a timeout, `std::condition_variable` may be more suitable.
- For designs that support stopping, define the shutdown protocol and thread lifetimes from the beginning.
- For performance-sensitive code, benchmark on the target operating system, library, and workload rather than relying on general claims about speed.

## Summary

The C++20 `std::counting_semaphore` type manages the number of permits available for acquisition and is suitable for limiting concurrency and synchronizing threads. Its three primary operations are `acquire`, `release`, and `try_acquire`. Unlike a mutex, a semaphore does not have thread-based ownership and does not, by itself, guarantee safe access to a shared data structure.

Using this tool safely depends on modeling permits precisely, respecting the counter limit, ensuring reliable release, accounting for the lack of a fairness guarantee, and designing shutdown explicitly. When a problem is genuinely about permits or limited capacity, a semaphore is an appropriate abstraction. For locking data or waiting on complex conditions, other standard-library mechanisms may be a clearer choice.


---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>