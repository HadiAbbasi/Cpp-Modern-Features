<div align="center">

[🇺🇸 English](./std_atomic_std_weak_ptr.md) | [🇮🇷 فارسی](../../fa/cpp20/std_atomic_std_weak_ptr.md)

</div>

---

# A Comprehensive Guide to `std::atomic<std::weak_ptr<T>>` in C++20

## Introduction

In multithreaded programs, multiple threads sometimes need to refer to a shared object without each observer extending the object's lifetime. In such designs, `std::weak_ptr<T>` is an important tool: it is a weak reference to an object whose ownership is managed by `std::shared_ptr<T>`, and it does not, by itself, prevent the object from being destroyed.

However, using an ordinary `std::weak_ptr<T>` across multiple threads is not automatically safe if at least one thread reads or modifies the same `weak_ptr` variable concurrently. C++20 introduced the atomic specialization `std::atomic<std::weak_ptr<T>>` to perform atomic access to and modification of the weak pointer value itself.

This type manages concurrency on the pointer variable. Safely operating on the pointed-to object, synchronizing changes to that object, and managing ownership at the application level remain responsibilities of your design.

## Understanding `std::weak_ptr` and Shared Ownership

### Weak Pointers Versus Owning Pointers

The type `std::shared_ptr<T>` manages shared ownership of an object using a reference count. The object remains alive as long as at least one shared owner exists. By contrast, `std::weak_ptr<T>` does not increase shared ownership; it only allows you to check for, or attempt to obtain access to, the object.

To use the object, a `weak_ptr` is usually converted to a `shared_ptr` with `lock()`. If the object is still alive, the result is a valid `shared_ptr`, and ownership is maintained for the lifetime of that `shared_ptr`. If the object has already been destroyed, the result is empty.

```
#include <memory>
#include <iostream>

int main() {
    auto owner = std::make_shared<int>(42);
    std::weak_ptr<int> observer = owner;

    if (auto locked = observer.lock()) {
        std::cout << *locked << '\n';
    }
}
```

In this example, `observer` does not guarantee that the object remains alive. Only `locked`, if successful, provides temporary ownership.

### The Race on the `weak_ptr` Variable Itself

The standard allows different threads to access the pointed-to objects and the shared control blocks of `shared_ptr` and `weak_ptr` instances according to the library's rules. However, this is not the same as safe concurrent access to a single smart-pointer variable. If one thread writes to a particular `weak_ptr` variable while another thread reads that same variable concurrently, a data race occurs unless appropriate synchronization is used.

The following example illustrates an invalid pattern:

```
#include <memory>

std::weak_ptr<int> shared_observer;

// Thread A:
shared_observer = std::make_shared<int>(7);

// Thread B:
auto local = shared_observer.lock();
```

The fact that operations on the control block's internal counters are thread-safe does not make concurrent access to the `shared_observer` variable itself safe. A mutex or `std::atomic<std::weak_ptr<T>>` can be used for this scenario.

## Introducing `std::atomic<std::weak_ptr<T>>`

### Purpose and Standard Version

The specialization `std::atomic<std::weak_ptr<T>>` was introduced in the C++20 standard. It allows a `weak_ptr` value to be atomically loaded, replaced, exchanged, or compared and replaced. This lets threads read and modify a shared weak pointer without a data race on that variable.

The required header is `<memory>`. The type is declared in the `std` namespace and is a specialization of the `std::atomic` class template.

```
#include <atomic>
#include <memory>

std::atomic<std::weak_ptr<int>> observer;
```

Here, `observer` is an atomic variable whose value is a `weak_ptr<int>`. This is different from making the `int` object itself atomic: only operations on the pointer value are atomic.

### What Does This Type Not Guarantee?

Making the pointer variable atomic does not make every operation related to the target object atomic. Specifically:

- This type synchronizes reads and modifications of the stored `weak_ptr` value.
- It does not keep the target object alive. To do that, `lock()` must succeed, and the resulting local `shared_ptr` must be retained.
- It does not make concurrent access to mutable members of the target object safe.
- It is not necessarily lock-free.
- It does not guarantee that several separate operations on the atomic variable form a single transaction.

## Construction and Initialization

### Default Construction

Default construction places an empty `weak_ptr` inside the atomic object:

```
#include <atomic>
#include <memory>

std::atomic<std::weak_ptr<int>> current;
```

In this state, calling `load()` returns an empty `weak_ptr`.

### Initializing from an Existing `weak_ptr`

The atomic object can be constructed with an initial `weak_ptr` value:

```
#include <atomic>
#include <memory>

auto owner = std::make_shared<int>(10);
std::weak_ptr<int> weak = owner;

std::atomic<std::weak_ptr<int>> current{weak};
```

Initialization should be completed before the object is published to other threads. Atomic operations after construction do not, by themselves, publish an object that is still being initialized before initialization is complete.

### Copy and Assignment Restrictions

Objects of type `std::atomic` are not copyable. Therefore, you cannot copy the atomic variable itself like an ordinary `weak_ptr`, or replace it with another atomic object. Read or write its value using the appropriate atomic operations.

```
std::weak_ptr<int> snapshot = current.load();
```

## Core Operations

### Loading with `load()`

The `load()` operation atomically returns a copy of the stored `weak_ptr` value:

```
auto snapshot = current.load();
```

Its optional parameter is `std::memory_order`. The default ordering is `std::memory_order_seq_cst`. If the loaded value is intended to provide access to other data prepared by the publishing thread, the synchronization relationship and memory ordering must be designed carefully; using an atomic alone does not create every desired producer-consumer relationship.

### Storing with `store()`

The `store()` operation replaces the atomic value with a new value:

```
std::weak_ptr<int> next = std::make_shared<int>(20);
current.store(next);
```

The `store()` operation supports `std::memory_order_relaxed`, `std::memory_order_release`, and `std::memory_order_seq_cst`. Using `std::memory_order_acquire` or `std::memory_order_acq_rel` with `store()` is invalid.

### Exchanging with `exchange()`

The `exchange()` operation stores a new value and returns the previous value:

```
std::weak_ptr<int> replacement = std::make_shared<int>(30);
auto previous = current.exchange(replacement);
```

This pattern is useful when you also need to inspect the previous value. The memory-order parameter for this operation must be a valid ordering for a read-modify-write operation.

### Comparing and Replacing with `compare_exchange_weak()` and `compare_exchange_strong()`

A compare-exchange operation compares the current value with an expected value. If they compare equal, it stores the new value; otherwise, it writes the current value to the `expected` argument.

```
std::weak_ptr<int> expected = current.load();
std::weak_ptr<int> desired = std::make_shared<int>(40);

if (current.compare_exchange_strong(expected, desired)) {
    // The replacement succeeded.
} else {
    // expected now contains the value observed by the failed operation.
}
```

The `compare_exchange_weak()` function may fail spuriously even when the comparison would otherwise have succeeded. For this reason, it is commonly used in a loop. The `compare_exchange_strong()` function does not fail spuriously, but it can still fail if the actual value differs from `expected`.

When comparing `weak_ptr` instances, account for the standard's rules for comparing pointer values and ownership. Do not compare two `weak_ptr` objects solely on the assumption that they “seem to point to the same object.” In identity-sensitive patterns, the distinction between the stored pointer value and shared ownership matters. Also, an object whose lifetime has ended may no longer be accessible even though its control block remains while weak references exist.

### Memory Ordering

The `std::memory_order` parameters specify how atomic operations are ordered relative to other memory operations across threads. The default ordering for many operations is `std::memory_order_seq_cst`. Weaker options may reduce cost, but they are correct only when the algorithm needs no stronger guarantees.

- `std::memory_order_relaxed` preserves atomicity, but by itself does not establish a synchronizes-with relationship for publishing other data.
- `std::memory_order_release` is used to publish preceding changes in the writing thread.
- `std::memory_order_acquire` is used to observe changes published by a corresponding release operation.
- `std::memory_order_acq_rel` is typically used for read-modify-write operations.
- `std::memory_order_seq_cst` participates in a single total order of sequentially consistent operations, in addition to acquire/release rules.

Do not use `release` or `acq_rel` with `load()`. Do not use `consume`, `acquire`, or `acq_rel` with `store()`. The success and failure orderings of compare-exchange overloads also have different restrictions: the failure ordering cannot be `release` or `acq_rel`, and it must not be stronger than the success ordering.

If you do not have a clear reason to choose a weaker ordering, start with the default ordering and optimize only after measuring and analyzing the algorithm.

## Turning an Atomic Value into an Object You Can Use

### Using `load()` Followed by `lock()`

A common pattern is to atomically load a `weak_ptr` and then attempt to lock it:

```
#include <atomic>
#include <memory>
#include <iostream>

std::atomic<std::weak_ptr<int>> current;

void consume() {
    auto weak = current.load();

    if (auto strong = weak.lock()) {
        std::cout << *strong << '\n';
    }
}
```

In this pattern, `load()` takes a snapshot of the pointer value. Then `lock()` attempts to obtain temporary ownership. If the original ownership has already ended, `lock()` returns an empty value, and the program must handle that case.

### Why Not Check Only `expired()`?

Checking `expired()` and then calling `lock()` as two separate operations does not create an atomic check-and-use operation. The object's state may change between those calls. Prefer using the result of `lock()` directly:

```
if (auto strong = weak.lock()) {
    // Use the object while strong remains alive.
}
```

### Retaining a Local `shared_ptr`

After `lock()` succeeds, keep the local `shared_ptr` alive for as long as you need the object. If you convert it to a raw pointer and release the local ownership too early, the object may be destroyed while you are still using it.

## Waiting for and Notifying Changes with `wait()` and `notify_*()`

The atomic smart-pointer specialization in C++20 also supports atomic wait and notify operations. These can be useful when threads should wait for the atomic value to change instead of repeatedly polling it.

The following simple example shows a thread loading the observed value and waiting for it to change:

```
#include <atomic>
#include <memory>

std::atomic<std::weak_ptr<int>> current;

void wait_for_change() {
    auto old_value = current.load();

    current.wait(old_value);

    auto new_value = current.load();
    (void)new_value;
}

void publish(std::weak_ptr<int> next) {
    current.store(next);
    current.notify_all();
}
```

The `wait(old_value)` operation blocks until the atomic value differs from the comparison value; an implementation may check the value again to avoid spurious wake-ups. After waking, the program should load the value again and inspect the state. `notify_one()` wakes one waiting thread, and `notify_all()` wakes all waiting threads, but notification itself does not change the value.

In a real design, ensure that changing the value and notifying waiters are coordinated by the program's logic. If a waiting thread could miss a change after loading the old value but before beginning `wait()`, the wait operation with the old value must recheck the value according to the atomic contract. Even so, keep the condition loop and notification protocol simple and explicit.

## Practical Example: Safely Publishing a Replaceable Object

Suppose a service tracks the current version of an object, and consumer threads are only observers. The primary object is owned by a `shared_ptr` elsewhere in the program, while the service publishes an atomic `weak_ptr`.

```
#include <atomic>
#include <memory>
#include <string>
#include <iostream>

struct Config {
    std::string endpoint;
};

class ConfigRegistry {
public:
    void publish(const std::shared_ptr<Config>& config) {
        current_.store(config, std::memory_order_release);
    }

    std::shared_ptr<Config> acquire() const {
        auto weak = current_.load(std::memory_order_acquire);
        return weak.lock();
    }

private:
    std::atomic<std::weak_ptr<Config>> current_;
};

int main() {
    ConfigRegistry registry;

    auto config = std::make_shared<Config>();
    config->endpoint = "https://example.invalid";
    registry.publish(config);

    if (auto active = registry.acquire()) {
        std::cout << active->endpoint << '\n';
    }
}
```

In this example, `ConfigRegistry` does not own `Config`. Ownership must remain elsewhere in the program; if all owning `shared_ptr` instances are destroyed, `acquire()` may return an empty `shared_ptr`.

An important point is that acquire/release ordering on the atomic variable does not, by itself, make it safe to modify `Config` concurrently after publication without synchronization. Treat this example as a pattern for publishing a pointer to an object that is not modified incompatibly after publication, or whose modifications are synchronized separately.

## Appropriate Use Cases

### Observer and Callback Registries

In systems where one component needs to refer to another object without controlling its lifetime, an atomic `weak_ptr` can allow multiple threads to read or replace the current observer.

### Publishing the Current Version of a Resource

For a configuration, state, or resource whose ownership is managed elsewhere, an atomic weak reference can be published. This design is useful when observers should not extend the resource's lifetime by retaining a reference.

### Breaking Ownership Cycles

`weak_ptr` is suitable for breaking cycles of `shared_ptr` ownership. Making it atomic can make concurrent access to the observer variable safe, but the type does not automatically detect or resolve ownership cycles.

## When It Is Not a Good Choice

### When Consumers Must Keep the Object Alive

If a consumer needs to guarantee that the object remains alive until an operation completes, it is often better to publish a `shared_ptr` atomically or explicitly transfer ownership to the consumer. Using a `weak_ptr` deliberately allows the object to disappear before the consumer accesses it.

### When All Access Is Already Protected by a Mutex

If the `weak_ptr` variable is already read and written under a shared mutex, replacing it with an atomic may add complexity without a measurable benefit. Choose the approach based on actual requirements and measurements.

### When Several Values Must Remain Consistent

A single atomic cannot automatically update several independent variables transactionally. If a pointer and several other fields must form a consistent snapshot, use an immutable object published through a single pointer, a mutex, or an appropriate synchronization protocol.

## Important Lifetime and Identity Considerations

### Atomicity Does Not Create Strong Ownership

The stored value is a `weak_ptr` and does not provide strong ownership. The atomic variable itself does not guarantee that the target object remains alive after loading. Only a successful `lock()` and retaining the resulting `shared_ptr` provide that guarantee for as long as the ownership is held.

### Object Destruction and Control-Block Destruction Are Different

When the last `shared_ptr` is destroyed, the managed object is destroyed. However, the control block may remain alive while weak references still exist. Therefore, a `weak_ptr` may still have a comparable ownership identity even though `lock()` can no longer provide access to the object.

### Do Not Bypass Ownership with Raw Pointers

After a successful `lock()`, using `strong.get()` may be necessary within the scope in which `strong` remains alive. However, storing that raw pointer for use after `strong`'s lifetime ends is unsafe. Doing so discards the lifetime guarantee provided by `shared_ptr`.

## Comparison with Alternatives

| Approach                          | Synchronization of the Pointer Value                  | Keeps the Object Alive                              | Main Point                                                                               |
| --------------------------------- | ----------------------------------------------------- | --------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Ordinary `std::weak_ptr<T>`       | No, not for concurrent access to the same variable    | No                                                  | Suitable for a non-owning observer in synchronized code                                  |
| `std::atomic<std::weak_ptr<T>>`   | Yes, for atomic operations on the value               | No                                                  | Suitable for thread-safe publication or replacement of an observer                       |
| `std::atomic<std::shared_ptr<T>>` | Yes                                                   | The loaded value provides strong ownership          | Suitable when consumers must keep the object alive                                       |
| `std::mutex` with `weak_ptr`      | Yes, if all accesses follow the same locking protocol | Only if `lock()` succeeds and ownership is retained | Flexible for multiple variables or complex invariants                                    |
| Atomic raw pointer                | Only the address value                                | No                                                  | Lifetime management and prevention of use-after-free remain the program's responsibility |

In C++11 and C++14, this atomic specialization was not available. A mutex could be used in earlier versions. Free atomic functions for `shared_ptr` were also available in earlier standards, but they are not a direct replacement for the `atomic<weak_ptr<T>>` specialization in C++20. When deciding which standard version to target, check the standard library and target implementation.

## Implementation Limitations and Costs

### There Is No Lock-Free Guarantee

An atomic API does not imply lock-free execution. An implementation of this specialization may use an internal lock. If latency or real-time behavior matters, check the target implementation's specifications and measure its actual behavior.

### Reference-Counting and Control-Block Costs

Operations on `weak_ptr` and `shared_ptr` may require updates to control-block counters. The exact cost depends on the standard library, architecture, and usage pattern. Avoid assumptions such as “atomic is always faster than a mutex” or “these operations are always lock-free.”

### Dependence on Library Support

The compiler must operate in C++20 mode, and the standard library in use must implement `std::atomic<std::weak_ptr<T>>` and the required members. The compiler version alone does not guarantee that every library feature is available or behaves identically on every platform.

## Common Mistakes

- Confusing control-block thread safety with variable thread safety: Multiple threads can work with distinct smart-pointer objects that share ownership, but concurrent access to one non-atomic variable still requires synchronization.
- Ignoring the result of `lock()`: The object may already have been destroyed; always handle an empty result.
- Using `expired()` as a guarantee: The state can change after the check; use the result of `lock()` as the deciding factor.
- Assuming the object stays alive because it is stored atomically: The stored value is weak and does not retain strong ownership.
- Using `memory_order_relaxed` without a reason: It preserves atomicity but does not guarantee the publication ordering of other data.
- Ignoring synchronization of the target object itself: Atomicity of the pointer does not make the object's internal fields atomic.
- Assuming lock-free behavior: Check the documentation and behavior of the implementation.
- Expecting one atomic to coordinate multiple variables: Multi-value invariants require a separate synchronization design.

## Design Recommendations

- First determine whether consumers need strong ownership or only a non-owning observer.
- If a consumer needs the object, load the value and call `lock()` immediately; then retain the resulting `shared_ptr` for the entire period of use.
- Use the default memory ordering until the algorithm's requirements are clear.
- If several values must remain consistent, bundle them into an immutable state and publish that state, or use a mutex.
- If performance matters, benchmark the actual usage pattern; do not infer lock-free behavior or speed from the name `atomic`.
- For mutable objects, define a separate synchronization contract for their internal members.

## Summary

In C++20, `std::atomic<std::weak_ptr<T>>` provides a standard way to atomically read and modify a `weak_ptr` value in a multithreaded environment. Its primary use is safely publishing or replacing a weak reference without having the atomic variable itself retain strong ownership of the target object.

The key is to keep three concerns separate: synchronizing the pointer value, guaranteeing the object's lifetime, and synchronizing the object's internal data. This atomic type solves only the first problem. `lock()` and retaining the resulting `shared_ptr` are needed for the second, while the third requires an independent concurrency design.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>