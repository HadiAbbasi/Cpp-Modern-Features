<div align="center">

[🇺🇸 English](./std_atomic_std_shared_ptr.md) | [🇮🇷 فارسی](../../fa/cpp20/std_atomic_std_shared_ptr.md)

</div>

---

# A Comprehensive Guide to `std::atomic<std::shared_ptr<T>>` in C++20

## Introduction

In multithreaded programs, multiple threads sometimes need to access a shared object while another thread may replace the current version of that object. The type `std::shared_ptr<T>` provides shared ownership and object lifetime management, but the same `shared_ptr` variable cannot be read and written concurrently by multiple threads without synchronization.

Introduced in C++20, `std::atomic<std::shared_ptr<T>>` provides a standard way to atomically load and replace a `shared_ptr`. It is particularly useful for patterns such as publishing immutable configuration, swapping snapshots, replacing lookup tables, and managing data that is read more often than it is written.

The key point is that making the pointer variable atomic does not make the pointed-to object itself thread-safe. This type handles concurrent access to the `shared_ptr` slot and its ownership management; synchronizing changes to the object’s contents remains the responsibility of the program’s design.

## The Problem This Feature Solves

Suppose a global variable points to the application’s configuration. Several threads read it, while another thread occasionally publishes a new configuration. Concurrently reading and writing the same `shared_ptr` object without synchronization creates a data race in C++, resulting in undefined behavior.

```cpp
#include <memory>

struct Config {
    int timeout_ms;
};

std::shared_ptr<const Config> current_config;

// Unsafe if one thread writes current_config while another reads it.
void update_config() {
    current_config = std::make_shared<const Config>(Config{500});
}

void read_config() {
    auto snapshot = current_config;
    (void)snapshot;
}
```

The problem in this example is not whether the pointed-to object is a `shared_ptr`; the problem is that the `current_config` variable itself is read and written without synchronization. In a standard implementation, increments and decrements of the shared ownership control block are designed to support concurrent ownership operations safely. However, that does not permit one thread to read the same `shared_ptr` variable while another thread modifies it.

## Introducing the Atomic Type in C++20

In C++20, the following specialization of `std::atomic` is available:

```cpp
std::atomic<std::shared_ptr<T>>
```

To use it, include the `<atomic>` and `<memory>` headers. This specialization follows the general interface of atomic operations but is specifically defined for `std::shared_ptr`.

```cpp
#include <atomic>
#include <memory>

struct Config {
    int timeout_ms;
};

std::atomic<std::shared_ptr<const Config>> current_config{
    std::make_shared<const Config>(Config{250})
};
```

This declaration creates an atomic slot for a `shared_ptr` value. Operations such as `load`, `store`, `exchange`, and `compare_exchange` are performed atomically on that slot.

### Differences from the Older `atomic_load` and `atomic_store` Interface

Before C++20, the standard library provided special free functions such as `std::atomic_load`, `std::atomic_store`, `std::atomic_exchange`, and `compare_exchange` variants for working with `shared_ptr`. These functions were introduced in C++11 and deprecated in C++20; the `std::atomic<std::shared_ptr<T>>` specialization is their modern replacement. The older functions may remain available in the relevant standards, but for new C++20 code, the atomic specialization is generally the preferred interface.

## Core Operations

### Loading with `load`

The `load` operation atomically obtains a copy of the shared pointer. The returned copy retains shared ownership, so if the atomic value is later replaced with another object, the previous object remains alive as long as active owners still exist.

```cpp
auto snapshot = current_config.load();
```

A `std::memory_order` argument can also be supplied to `load`:

```cpp
auto snapshot = current_config.load(std::memory_order_acquire);
```

For straightforward code, the default ordering, `std::memory_order_seq_cst`, is valid and often easier to read. A weaker ordering should be chosen based on the data-publication model and a careful analysis of synchronization requirements, not merely in the hope of improving performance.

### Storing with `store`

The `store` operation atomically replaces the current value with a new `shared_ptr`:

```cpp
auto next = std::make_shared<const Config>(Config{750});
current_config.store(next);
```

The default memory ordering for `store`, `std::memory_order_seq_cst`, is valid. `std::memory_order_acquire` and `std::memory_order_acq_rel` cannot be used with `store`; those orderings are not valid for a store-only operation.

### Replacing and Retrieving the Previous Value with `exchange`

The `exchange` operation atomically installs a new value and returns the previous value:

```cpp
auto next = std::make_shared<const Config>(Config{1000});
auto previous = current_config.exchange(next);
```

This operation is useful when the program wants to retain or record the previous version as well as publish the new one, or when it needs to make decisions about releasing the old resources after other users have finished with them. `exchange` itself does not guarantee that no other thread is still using the previous value; the returned pointer and previously loaded copies may remain alive.

### Conditional Updates with `compare_exchange`

The `compare_exchange_weak` and `compare_exchange_strong` operations compare the atomic value with an expected value. If the comparison succeeds, the new value is installed. If it fails, the `expected` variable is updated with the current value.

```cpp
auto expected = current_config.load();
auto desired = std::make_shared<const Config>(Config{1500});

if (current_config.compare_exchange_strong(expected, desired)) {
    // The replacement succeeded.
} else {
    // expected now contains the value observed by the failed operation.
}
```

The comparison of `shared_ptr` values in these operations follows the rules specified for this specialization, involving ownership and the stored pointer. Do not confuse this with comparing the contents of the pointed-to objects. Consequently, two objects with identical fields are not necessarily equivalent for this comparison.

The `weak` version may fail spuriously even when the value has not actually changed, which is why it is commonly used in retry loops. The `strong` version does not fail spuriously, although `weak` may be more suitable for some patterns.

## A Complete Practical Example: Publishing Immutable Configuration

A straightforward use case is to build a new version of data in one thread and publish it for readers. In this pattern, readers obtain a local snapshot and then work with that snapshot without locking it. Using `const` helps ensure that the data is not modified after publication.

```cpp
#include <atomic>
#include <iostream>
#include <memory>
#include <thread>

struct Config {
    int timeout_ms;
    int retry_count;
};

std::atomic<std::shared_ptr<const Config>> config{
    std::make_shared<const Config>(Config{200, 3})
};

void reader() {
    auto snapshot = config.load(std::memory_order_acquire);

    std::cout << "timeout=" << snapshot->timeout_ms
              << ", retries=" << snapshot->retry_count << '\n';
}

void writer() {
    auto next = std::make_shared<const Config>(Config{800, 5});
    config.store(std::move(next), std::memory_order_release);
}

int main() {
    std::thread r1(reader);
    std::thread w(writer);
    std::thread r2(reader);

    r1.join();
    w.join();
    r2.join();
}
```

In this example, each reader sees either the old version or the new version. The reader holds a local `shared_ptr` that keeps that snapshot alive for the duration of its use. If the store uses `memory_order_release` and a successful load using `memory_order_acquire` observes the published value, writes performed before publication can become visible to the reader through the `synchronizes-with` relationship and, consequently, `happens-before`.

Note that this program does not guarantee a fixed order for the printed lines. Also, `std::cout` is used here for demonstration; output order is not a measure of the order in which configuration versions are published.

## The Memory Model and Operation Ordering

### Default Ordering

Like other atomic operations, operations on this type use `std::memory_order_seq_cst` when no memory-order argument is supplied. This ordering provides a single order consistent with the sequential-consistency rules for the relevant operations, but it does not replace correct synchronization for access to non-atomic data.

### Using `release` and `acquire`

In a data-publication pattern, the writer typically finishes constructing an object and then publishes it with `store(memory_order_release)`. The reader obtains the value with `load(memory_order_acquire)`. If the load observes the published value, operations preceding the release store are ordered before operations following the acquire load.

```cpp
auto next = std::make_shared<const Config>(Config{600, 4});
config.store(std::move(next), std::memory_order_release);

auto snapshot = config.load(std::memory_order_acquire);
```

In a real example, these two lines would usually execute in separate threads. If executed in the same function and in this order, they are illustrative, but do not by themselves demonstrate a concurrent scenario.

### Restrictions on `memory_order`

The allowed memory orderings differ between operations:

- `load` cannot use `std::memory_order_release` or `std::memory_order_acq_rel`.
- `store` cannot use `std::memory_order_acquire` or `std::memory_order_acq_rel`.
- `exchange` and `compare_exchange` operations can accept stronger orderings such as `acq_rel`, subject to the restrictions on success and failure orderings for `compare_exchange`.
- The failure ordering for `compare_exchange` cannot be `release` or `acq_rel`, and it must not be stronger than the success ordering.

In performance-sensitive code, choose memory orderings only after identifying the required relationships between data. Making a `shared_ptr` atomic does not make every other variable in the program safe.

## Lifetime and Ownership Guarantees

### Keeping the Pointed-to Object Alive

When `load` returns a `shared_ptr`, the returned value retains shared ownership. If a writer immediately replaces the atomic value, the reader’s snapshot can still keep the old object alive.

```cpp
auto old_snapshot = config.load();
config.store(std::make_shared<const Config>(Config{900, 2}));

// old_snapshot still keeps the old Config alive.
std::cout << old_snapshot->timeout_ms << '\n';
```

This behavior is useful for accessing immutable snapshots without locking. However, if a reader retains a snapshot for a long time, memory used by old versions may not be released until that snapshot is destroyed.

### Thread Safety of the Ownership Counter Versus the Pointed-to Object

Thread-safe management of the shared ownership counter is different from thread safety of the object’s data. This pattern can be appropriate:

```cpp
std::atomic<std::shared_ptr<Config>> current;

void reader() {
    auto p = current.load();
    if (p) {
        // Reading is safe only if no concurrent unsynchronized write
        // modifies the same fields of *p.
        int value = p->timeout_ms;
        (void)value;
    }
}
```

If another thread modifies `p->timeout_ms` concurrently, the atomic `shared_ptr` does not protect that field. For a mutable object, use a mutex, separate atomic members, or another valid synchronization protocol. In snapshot-based designs, the data is usually kept immutable after publication.

### Be Careful with Ownership Cycles

`std::shared_ptr` does not automatically collect ownership cycles. If an object directly or indirectly creates shared ownership that leads back to itself, its reference count may never reach zero. Using `std::weak_ptr` for relationships that should not create ownership can solve this problem. Making the `shared_ptr` slot atomic does not change this behavior.

## Are the Operations Lock-Free?

The existence of `std::atomic<std::shared_ptr<T>>` does not mean that all its operations are necessarily lock-free. The standard-library implementation may use an internal lock or another mechanism to guarantee atomicity. This specialization does not provide an `is_lock_free` member function, so one should not infer that `atomic<shared_ptr>` operations are lock-free from `std::atomic_is_lock_free` or from assumptions about simpler atomic types.

In practice, `shared_ptr` operations require ownership-counter management and may cost more than swapping a raw pointer. If predictable latency or very high performance is essential, benchmark on the target library and platform, taking into account allocation, reference-count increments and decrements, and object destruction.

## Comparison with `std::mutex`

Two common approaches are `std::atomic<std::shared_ptr<T>>` and protecting an ordinary `shared_ptr` with `std::mutex`. Neither is universally better.

| Criterion | `std::atomic<std::shared_ptr<T>>` | `std::mutex` with `shared_ptr` |
|---|---|---|
| Atomicity of the pointer slot | Provided by the specialization's operations | Provided by locking around all relevant accesses |
| Reading a snapshot | Usually straightforward with `load` | Requires following the locking protocol |
| Internal locking | May exist in the implementation | Explicit and visible lock |
| Working with related variables | Not sufficient by itself | Can protect multiple pieces of data under one lock |
| Suitable for | Publishing a single value or snapshot | Multi-variable invariants and compound operations |
| Snapshot lifetime | Preserved by copying the `shared_ptr` | Can be copied under the lock and used after unlocking |

If an operation requires several variables to remain mutually consistent, a mutex is often the simpler and clearer solution. If the problem is only to replace a pointer to immutable data, the atomic type can provide a suitable interface.

## Common Usage Patterns

### Publishing Configuration Snapshots

An application can build new configuration data outside the read path and publish it with a single `store` once it is ready. Readers call `load` once and use their local snapshots. This pattern is suitable when reads greatly outnumber writes and the data remains unchanged after publication.

### Replacing a Lookup Table or Cache Data

In a cache or registry, a new data structure can be fully prepared and then replace the previous version. Atomicity of the pointer slot prevents readers from observing a partially replaced `shared_ptr` value, but the construction and mutation of the data structure itself must be made safe separately.

### Publishing Replaceable Policies or Models

A service can replace a policy object, routing table, or read-only model with a new version. A reader that has obtained the previous version retains ownership until it finishes. This prevents premature destruction and reduces the need for manual lifetime coordination.

## Updating in a `compare_exchange` Loop

Sometimes the new value must be calculated based on the current version. In that case, a single `load` followed by `store` is not enough, because another writer may change the value between those operations. A compare-exchange loop can be used instead:

```cpp
#include <atomic>
#include <memory>

struct Settings {
    int limit;
};

std::atomic<std::shared_ptr<const Settings>> settings{
    std::make_shared<const Settings>(Settings{10})
};

void increase_limit(int delta) {
    auto expected = settings.load(std::memory_order_acquire);

    for (;;) {
        auto desired = std::make_shared<const Settings>(
            Settings{expected->limit + delta}
        );

        if (settings.compare_exchange_weak(
                expected,
                desired,
                std::memory_order_release,
                std::memory_order_acquire)) {
            break;
        }

        // On failure, expected is updated to the currently observed value.
    }
}
```

In this example, the desired value is constructed on each attempt using that attempt’s `expected` value. If the operation fails, `expected` is updated to the observed value, and the loop uses the newer value as its basis. The success ordering `release` and failure ordering `acquire` are compatible with the restrictions on `compare_exchange`.

This pattern is correct only if the computed operation can safely be retried after failure. Constructing a new object on every attempt may incur allocation costs. Also, if the computation has external side effects, a retry may execute those side effects multiple times; in that case, the design should be reconsidered.

## Subtleties of `compare_exchange`

### Comparing Pointers and Ownership

The compare-exchange operation for `shared_ptr` follows the comparison rules specified for the specialization, rather than comparing the contents of the pointed-to object. In older code or across different implementations, do not assume—without considering the target standard version—that comparing only the stored address captures the entire meaning of the comparison. In C++20, the rules for this specialization define how the expected value is compared, and library details may evolve in later standards. Consult the relevant standard when relying on edge cases.

### The ABA Problem

The ABA problem occurs when a value changes from A to B and then back to A, while an algorithm checks only the current value for equality. Shared ownership can keep objects alive in some scenarios, but it does not by itself provide a generation counter or a protocol that detects every intervening change. If correctness depends on knowing that no intermediate change occurred, use a generation counter, version identifier, or another design that makes such changes detectable.

### Failure and Updating `expected`

After a compare-exchange failure, `expected` is no longer necessarily the original value; it contains the current value observed by the failed operation. Forgetting this can cause recalculation based on an outdated snapshot and lead to logical errors.

## Handling `nullptr` and Empty Values

The atomic value can hold an empty `shared_ptr`. Therefore, as with any other `shared_ptr`, check that it is non-empty before dereferencing it.

```cpp
std::atomic<std::shared_ptr<const Config>> config;

auto snapshot = config.load();
if (snapshot) {
    std::cout << snapshot->timeout_ms << '\n';
}
```

Default initialization of a `std::atomic<std::shared_ptr<T>>` initializes it with an empty `shared_ptr`. If the program expects a value to always be present, it is better to establish that invariant during construction while also defining how an empty value should be handled during error paths or shutdown.

## Destruction and Destructor Side Effects

When an atomic operation changes the `shared_ptr` slot, the ownership count of the previous value may decrease. If this removes the last owner, object destruction and memory deallocation may occur on the same execution path that performed the operation, subject to the standard’s rules and implementation details. Therefore, do not place operations such as `store` or `exchange` in latency-sensitive paths without considering whether destructors, expensive resource release, or unpredictable callbacks might run there.

If the timing and location of resource release matter, the previous value can be retrieved with `exchange` and a separate release policy can be designed. However, copies retained by readers may still delay final destruction. Using `exchange` alone does not guarantee that the destructor runs at that exact point or on a particular thread.

## Construction and Initialization

An existing `shared_ptr` can be passed to the constructor for initialization:

```cpp
auto initial = std::make_shared<const Config>(Config{300});
std::atomic<std::shared_ptr<const Config>> config{initial};
```

It can also be initialized directly with a `shared_ptr` created in the same expression. The type `std::atomic<std::shared_ptr<T>>` is not copyable; an atomic object cannot be copied like an ordinary `shared_ptr`. To transfer a value between atomic slots, use load, store, or exchange operations.

## Limitations and When Not to Use It

- If several fields or objects must change together as one invariant, a single atomic slot is not sufficient.
- If the pointed-to object is modified concurrently after publication, separate synchronization is needed for its contents.
- If reads or writes are extremely frequent and the cost of `shared_ptr` reference counting is significant, benchmark the design and consider alternatives.
- If shared ownership is unnecessary and object lifetime is already guaranteed by another protocol, `shared_ptr` may introduce unnecessary overhead.
- If strict latency or lock-free behavior is required, do not assume this specialization provides such guarantees.
- If multiple writers make updates based on the previous value, a simple `load` followed by `store` can cause lost updates; use compare-exchange or a mutex in that case.

## Common Mistakes

### Confusing an Atomic Pointer with an Atomic Object

This type synchronizes operations on the `shared_ptr` slot only. Concurrently reading and writing a non-atomic field in the pointed-to object can still create a data race.

### Keeping a Raw Pointer After Ownership Ends

If a raw pointer or reference is obtained from a snapshot and its owning `shared_ptr` is destroyed, later access may result in a dangling pointer. Keep a local `shared_ptr` for as long as access is needed, or guarantee the lifetime through another valid mechanism.

### Assuming the Operations Are Lock-Free

The name `atomic` does not guarantee a lock-free implementation. Measure performance on the target platform.

### Using `store` for Updates Based on the Current Value

The pattern of `load`, compute, then `store` can lose updates when multiple writers are present. Use compare-exchange or a mutex for conditional updates or updates based on the previous value.

### Retaining Snapshots for Too Long

Shared ownership protects lifetime but can keep memory used by old versions allocated for an extended period. Account for snapshot retention time and data size in the memory design.

## Choosing an Approach

When choosing between alternatives, first identify what needs to be atomic:

- If only a pointer to an immutable snapshot needs to be replaced, `std::atomic<std::shared_ptr<const T>>` is a clear and practical option.
- If multiple pieces of data must change together, consider `std::mutex` or a more comprehensive synchronization design.
- If shared ownership is unnecessary, a raw pointer or another ownership model with a precise lifetime protocol may be more appropriate; however, removing `shared_ptr` without proving lifetime safety is dangerous.
- If performance matters, evaluate it using realistic benchmarks, data-race detection tools, and a workload similar to production.

## Summary

In C++20, `std::atomic<std::shared_ptr<T>>` provides atomic loading, storing, replacement, and compare-exchange operations for a `shared_ptr`. It is especially suitable for publishing immutable snapshots and replacing data that is read frequently and updated less often.

The most important design principle is to keep three concerns separate: atomicity of the pointer slot, the lifetime of the pointed-to object, and synchronization of access to the object’s contents. `std::atomic<std::shared_ptr<T>>` usefully supports the first two at the level of ownership and publication operations, but concurrent modifications within the object or multi-variable invariants require additional mechanisms. Finally, performance, destruction behavior, and the actual need for shared ownership should be evaluated for the target implementation and workload.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>