<div align="center">

[🇺🇸 English](./std_latch.md) | [🇮🇷 فارسی](../../fa/cpp20/std_latch.md)

</div>

---

# **A Comprehensive Guide to&#x20;**`std::latch`**&#x20;in C++20**

## **Introduction**

In multithreaded applications, it is sometimes necessary for one or more threads to wait before continuing execution until a specific number of independent operations have completed. For example, several threads may load separate pieces of data, while the main thread must wait until all the data is ready before starting the next processing stage.

This type of synchronization can be implemented using `std::mutex`, `std::condition_variable`, or atomic counters. However, these approaches often require repetitive code and manual management of synchronization state.

C++20 introduced `std::latch` as a specialized synchronization primitive for implementing this coordination pattern.

The `std::latch` class is a single-use synchronization primitive initialized with a counter. Each thread or operation can report the completion of its contribution by decrementing the counter. Waiting threads are released when the counter reaches zero.

## **Concept and Design Purpose**

### **The Problem Solved by a Latch**

Consider an application that executes several independent tasks concurrently, where continuing execution depends on all of them completing.

This pattern consists of the following steps:

- Several operations are started independently.
- Each operation reports the completion of its contribution.
- One or more threads wait for the counter to reach zero.
- Once the final contribution is completed, all waiting threads are allowed to continue.

This pattern is useful in scenarios such as:

- Parallel initialization of application resources
- Concurrent loading of multiple files
- Starting several internal services
- Executing stages of a pipeline
- Collecting results from multiple independent tasks
- Coordinating the main thread with multiple worker threads

### **Single-Use Nature**

The most important design characteristic of `std::latch` is that its counter cannot be incremented or reset after reaching zero.

Therefore, this class is designed for a specific synchronization point during its lifetime, rather than for repeatedly coordinating multiple synchronization cycles.

For example, if several threads need to report that their required resources have been initialized, `std::latch` is an appropriate choice.

However, if the same threads need to synchronize with each other across multiple iterations, `std::barrier` is generally a more suitable option.

## **Header and Namespace**

To use this feature, include the following standard header:

```
#include <latch>
```

The `latch` type is defined in the `std` namespace:

```
std::latch ready{3};
```

This feature is part of the C++20 Standard Library and is not a compiler-specific extension.

However, actual support depends on the version of the compiler and its accompanying standard library implementation.

## **API Overview**

### **Constructor**

The primary constructor has the following signature:

```
explicit latch(std::ptrdiff_t expected);
```

The `expected` parameter specifies the initial counter value.

It determines how many contributions must be reported before the latch reaches its completion state.

The `expected` value must be non-negative and must not exceed `std::latch::max()`.

A value of zero is also valid. In this case, the latch is already ready when constructed, and waiting operations return without blocking.

### **The max Function**

The static `max` function returns the maximum supported counter value:

```
static constexpr std::ptrdiff_t max() noexcept;
```

This value is implementation-defined, so applications should not assume a fixed, portable value across all platforms.

### **The count_down Function**

The `count_down` function decrements the counter:

```
void count_down(std::ptrdiff_t update = 1) noexcept;
```

The default value of `update` is one.

This function is used to report the completion of an operation's contribution. It does not wait for the counter to reach zero.

The `update` value must be non-negative and must not exceed the current counter value.

Violating the preconditions of this operation results in undefined behavior.

An `update` value of zero does not decrement the counter and is not useful for reporting the completion of a contribution.

An important point is that `count_down` is not a waiting operation. If decrementing the counter causes it to reach zero, the calling thread can continue immediately, while other threads waiting on the latch are also released.

### **The wait Function**

The `wait` function blocks until the counter reaches zero:

```
void wait() const;
```

This function does not modify the counter.

If the counter is already zero, the function returns without waiting.

Multiple threads can wait on the same latch concurrently. When the counter reaches zero, all of them are allowed to proceed.

### **The try_wait Function**

The `try_wait` function checks the latch state without performing a blocking wait:

```
bool try_wait() const noexcept;
```

It returns `true` if the counter is zero and `false` otherwise.

This function is suitable for checking whether a latch has completed, but it should not be treated as a replacement for a properly designed waiting protocol.

Repeatedly calling `try_wait` inside a busy-wait loop can unnecessarily consume CPU resources.

If a thread needs to wait until an operation completes, `wait` is generally the simpler and more efficient choice.

### **The arrive_and_wait Function**

The `arrive_and_wait` function reports a contribution and then waits until the counter reaches zero:

```
void arrive_and_wait(std::ptrdiff_t update = 1);
```

Conceptually, this function is equivalent to calling `count_down(update)` followed by `wait()`.

The `update` value must be positive and must not exceed the current counter value.

This operation is useful when the calling thread is itself one of the participants in reaching the synchronization point and must not continue until the remaining contributions have been reported.

## **Memory Model and Synchronization Guarantees**

### **Memory Ordering**

`std::latch` is more than a simple counter. It also provides standard synchronization guarantees for coordinating memory access between threads.

Operations performed by participating threads before decrementing the counter can become visible to threads that successfully return from `wait` after the counter reaches zero, through the synchronization relationship established by the latch.

In practical terms, if each thread prepares some data and then reports its contribution using `count_down`, a waiting thread can access the prepared results after `wait` returns, provided that the data is otherwise used without data races.

This guarantee does not eliminate the need for proper shared-data ownership and access design.

If multiple threads concurrently write to the same non-atomic object, or another thread accesses that object independently of the latch synchronization protocol, a data race may still occur.

### **Completion Point**

When a decrement operation causes the counter to reach zero, the latch enters its completed state.

The return of `wait` calls that observe this state participates in the synchronization relationship with the completion operation.

In application design, it is best to call `count_down` only after the actual work has finished and all required data has been prepared, rather than at the beginning of an operation or before its results have been stored.

### **Difference Between Completion and Thread Termination**

A latch reaching zero does not mean that all participating threads have terminated.

It only indicates that the expected number of contributions has been reported.

A thread may still execute additional code after calling `count_down`.

If complete thread termination must also be guaranteed, an appropriate thread-lifetime management mechanism, such as `join`, must be used.

## **Practical Examples**

### **Waiting for Multiple Workers to Complete**

In the following example, three threads each prepare part of the data. The main thread waits until all three operations have completed:

```
#include <array>
#include <iostream>
#include <latch>
#include <thread>

int main() {
    constexpr int workerCount = 3;

    std::latch ready{workerCount};
    std::array<int, workerCount> results{};

    std::array<std::jthread, workerCount> workers;

    for (int i = 0; i < workerCount; ++i) {
        workers[i] = std::jthread([&, i] {
            results[i] = (i + 1) * 10;
            ready.count_down();
        });
    }

    ready.wait();

    int total = 0;
    for (int value : results) {
        total += value;
    }

    std::cout << "Total: " << total << '\n';
}
```

In this example, each thread writes only to its own array element, so the writes to independent elements do not interfere with each other.

The main thread collects the results after `wait` returns.

Using `std::jthread`, introduced in C++20, also simplifies thread-lifetime management.

If a `jthread` is still joinable when its destructor runs, the destructor first requests stop and then joins the thread.

In this example, all worker threads have completed their work before `main` exits.

### **Including the Main Thread as a Participant**

Sometimes the main thread also performs an independent task and must participate in the synchronization point alongside the workers.

In this case, `arrive_and_wait` can be used:

```
#include <iostream>
#include <latch>
#include <thread>

int main() {
    std::latch phaseReady{3};

    std::jthread first([&] {
        // Perform the first preparation task
        phaseReady.arrive_and_wait();

        // Continue after all participants arrive
        std::cout << "First worker continues\n";
    });

    std::jthread second([&] {
        // Perform the second preparation task
        phaseReady.arrive_and_wait();

        // Continue after all participants arrive
        std::cout << "Second worker continues\n";
    });

    // Perform the main thread preparation task
    phaseReady.arrive_and_wait();

    std::cout << "Main thread continues\n";
}
```

In this example, the counter starts at three:

- One contribution belongs to the first worker thread.
- One contribution belongs to the second worker thread.
- One contribution belongs to the main thread.

Each participant reports its contribution after completing its task and waits for the remaining participants to arrive.

### **Checking Completion Without Blocking**

Sometimes an application wants to proceed with the next stage if an operation has already completed, while performing other work if it has not:

```
#include <iostream>
#include <latch>
#include <thread>

int main() {
    std::latch completed{1};

    std::jthread worker([&] {
        // Perform some work
        completed.count_down();
    });

    if (completed.try_wait()) {
        std::cout << "Work is already complete\n";
    } else {
        std::cout << "Work is still running\n";
        completed.wait();
    }

    std::cout << "Continue execution\n";
}
```

The result of `try_wait` only represents the state of the latch at the time of the check.

If it returns `false`, the operation may complete immediately afterward. Therefore, this result must not be interpreted as a guarantee that the latch will remain incomplete.

### **Parallel Resource Initialization**

One suitable application of a latch is waiting for multiple independent resources to become ready.

For example, an application can load configuration, data tables, and required files concurrently before starting its main processing stage:

```
#include <latch>
#include <thread>

struct ApplicationState {
    int configurationVersion = 0;
    int dataVersion = 0;
    int cacheVersion = 0;
};

int main() {
    ApplicationState state;
    std::latch initialized{3};

    std::jthread configLoader([&] {
        state.configurationVersion = 1;
        initialized.count_down();
    });

    std::jthread dataLoader([&] {
        state.dataVersion = 2;
        initialized.count_down();
    });

    std::jthread cacheBuilder([&] {
        state.cacheVersion = 3;
        initialized.count_down();
    });

    initialized.wait();

    // All initialization tasks have reported completion.
    // The application can now use the prepared state.
}
```

Each thread writes to a different member of `state`.

After `wait` returns, the main thread can access the prepared state.

In a real application, loading errors must also be handled because a latch does not provide an error-reporting mechanism by itself.

## **Design Considerations and Usage Patterns**

### **Determining the Number of Participants**

The initial counter value must match the actual number of expected contributions.

If the initial value is greater than the number of contributions that will actually be reported, waiting threads may remain blocked indefinitely.

If it is smaller, the latch may reach zero before all operations have genuinely completed.

It is best to calculate the number of tasks from a single source and use that same value both when constructing the latch and when creating the tasks.

Avoid scattering hardcoded counter values across different parts of the application.

### **Reporting Each Contribution Exactly Once**

Each contribution must be reported exactly once according to the designed protocol.

If an operation calls `count_down` twice for the same contribution, the counter may be decremented too far, resulting in undefined behavior.

If an execution path never reports its contribution, waiting may never finish.

When handling multiple execution paths, including exceptions and early returns, ensure that the contribution-completion protocol is preserved.

If necessary, a custom RAII guard can be designed to ensure that a contribution is reported when an execution path exits. However, care must be taken to avoid reporting a contribution too early or more than once.

### **Error Handling and Cancellation**

`std::latch` does not provide built-in mechanisms for exceptions, cancellation, or timeouts.

If one task fails and does not report its contribution, waiting threads may remain blocked indefinitely.

A robust design should record each task's result in an appropriate data structure and ensure that the counting protocol is completed even when a task fails.

The waiting thread can then inspect the recorded errors after `wait` returns.

If timeout support is required, `std::latch` does not provide a time-based waiting API.

In such situations, a different design using `std::condition_variable` or another suitable synchronization mechanism may be necessary.

### **Object Lifetime and Ownership**

The latch object must remain alive until all operations that access it have finished.

Destroying a latch while another thread is still accessing it through `count_down`, `wait`, or a related operation is invalid and can result in undefined behavior.

To avoid dangling references, place the latch in a scope whose lifetime exceeds that of all participating threads.

Before destroying the object, ensure that thread access has ended using `join` or an appropriate ownership mechanism.

## **Limitations and Common Mistakes**

### **Attempting to Reuse a Latch**

Once the counter reaches zero, it cannot be incremented again.

Therefore, using the same latch for multiple consecutive stages, such as repeated processing rounds, is incorrect.

Use `std::barrier` or another synchronization primitive appropriate to the problem when repeated coordination is required.

### **Initializing with an Incorrect Counter Value**

If the counter is greater than the actual number of reported contributions, waiting will never finish.

If it is smaller, waiting threads may be released before all tasks have genuinely completed.

Design the number of participants carefully.

### **Decrementing Beyond the Remaining Count**

The total decrement must not exceed the current counter value.

This condition must be guaranteed by application logic. `std::latch` does not throw an exception or produce a recoverable error for an invalid decrement.

### **Incorrect Assumptions About join**

The return of `wait` does not mean that worker threads have terminated.

It only indicates that all expected contributions have been reported.

Use `join` when complete thread termination must be guaranteed.

### **Unnecessary Polling**

A loop that repeatedly calls `try_wait` can increase CPU consumption.

If the thread does not need to perform other work while waiting, prefer `wait`.

### **Waiting for a Participant That Never Executes**

If the counter includes a task that never reports its contribution because of a scheduling issue, thread-creation failure, or a particular execution path, waiting may continue indefinitely.

The synchronization protocol must account for failure paths and tasks that are never executed.

### **Assuming a Specific Thread Execution Order**

`std::latch` does not guarantee that threads return from `wait` in a particular order or execute in a specific sequence after being released.

Thread scheduling is controlled by the operating system and the runtime implementation.

## **Comparison with Related Synchronization Primitives**

### **Difference Between std::latch and std::barrier**

Both tools are used for thread coordination, but their design goals are different:

| Feature             | `std::latch`                           | `std::barrier`                                      |
| ------------------- | -------------------------------------- | --------------------------------------------------- |
| Introduced in       | C++20                                  | C++20                                               |
| Nature              | Single-use                             | Reusable across multiple phases                     |
| Counter             | Can only be decremented                | Participant count is managed for each phase         |
| Participant waiting | `wait` or `arrive_and_wait`            | `arrive_and_wait` and related operations            |
| Primary use case    | Waiting for a set of tasks to complete | Repeated synchronization across multiple phases     |
| Counter reset       | Not supported                          | Each phase is managed through the barrier mechanism |

For example, if several threads only need to initialize resources once before the application continues, a latch is simpler.

If the same threads need to collect results during every iteration and then proceed to the next iteration together, a barrier is generally more appropriate.

### **Difference Between std::latch and std::condition_variable**

`std::condition_variable` is a more general-purpose synchronization tool for waiting on a predicate.

The programmer must manage the mutex, shared state, and predicate-checking loop.

In contrast, a latch provides a specific counter and a straightforward purpose, requiring less code for the pattern of waiting until N contributions have been reported.

If the waiting condition is complex, the state must remain changeable, a timeout is required, or multiple conditions can trigger a wake-up, `condition_variable` provides greater flexibility.

### **Difference Between std::latch and std::atomic**

An atomic counter allows concurrent counting, but it is not, by itself, a synchronization mechanism for waiting threads.

Manually implementing a latch using atomic operations generally requires managing the waiting loop, notifications, and memory ordering.

Starting with C++20, `std::atomic::wait` and notification operations can be used in certain scenarios. However, a latch provides a higher-level abstraction for counting contributions and waiting until the counter reaches zero.

The choice between them depends on the exact synchronization protocol required.

### **Difference Between std::latch and std::future/std::promise**

`std::future` and `std::promise` are suitable for transferring the result or exception of an asynchronous operation to a consumer.

A latch only coordinates the completion of a specified number of contributions; it does not transfer the result of each task.

If individual operation results must be retrieved, futures may be a more appropriate choice.

If the only requirement is to report that several independent tasks have completed, a latch can provide a simpler solution.

## **Implementation and Portability Considerations**

### **Compiler and Standard Library Support**

Using C++20 mode in a compiler does not automatically guarantee that every C++20 Standard Library feature is fully implemented in the installed version.

Support for `std::latch` depends on the standard library implementation, such as libstdc++, libc++, or Microsoft STL, and its version.

To enable C++20 mode, the appropriate compiler option usually needs to be specified. For example:

```
g++ -std=c++20 -pthread main.cpp -o app
```

```
clang++ -std=c++20 -pthread main.cpp -o app
```

```
cl /std:c++20 /EHsc main.cpp
```

The exact options and any additional configuration requirements depend on the compiler, operating system, and toolchain.

On many POSIX systems, the `-pthread` option is used with GCC and Clang to enable thread support.

### **Checking the Feature-Test Macro**

The standard feature-test macro can be used to check whether the standard library supports latch:

```
#include <version>

#if defined(__cpp_lib_latch) && __cpp_lib_latch >= 201907L
    // std::latch is available
#else
    #error "This standard library does not provide std::latch"
#endif
```

The recognized value of `__cpp_lib_latch` is `201907L`.

This macro reports library feature availability, not merely the selected language standard version.

### **Runtime Cost and Primitive Selection**

The standard does not guarantee that a latch implementation is lock-free or that it uses a particular hardware primitive.

Internal implementation details, such as the use of atomics, futexes, or native operating-system mechanisms, are implementation-specific.

In practice, `std::latch` is generally simpler and less error-prone than manually implementing a counting and waiting protocol.

However, for highly latency-sensitive code paths, the behavior of the target implementation should be measured rather than making assumptions about performance based solely on the abstraction's name.

## **Professional Design Recommendations**

- Prefer `std::latch` over manually implemented counter-and-wait solutions for single-use synchronization points.
- Derive the initial counter from the actual number of tasks to keep the counter consistent with the execution protocol.
- Report each contribution exactly once and only after the corresponding task has genuinely completed.
- Account for exceptions, cancellation, thread-creation failures, and tasks that are never executed.
- Partition shared data so that no data races occur before the synchronization point.
- Keep the latch alive longer than all threads that use it.
- Choose a more suitable mechanism, such as `std::condition_variable`, when timed waiting or complex conditions are required.
- Consider `std::barrier` for repeated synchronization across multiple phases.
- Consider `std::future` and `std::promise` when individual asynchronous results and exceptions must be retrieved.
- Use `join` or an equivalent thread-lifetime management mechanism in addition to the latch when thread termination must be guaranteed.

## **Summary**

The `std::latch` class, introduced in the C++20 Standard Library, is a simple and effective synchronization primitive for coordinating a group of concurrent operations exactly once.

It operates using an initial counter. Participants report their contributions through `count_down`, while waiting threads remain blocked in `wait` until the counter reaches zero.

The main advantage of a latch is that it provides a common synchronization pattern without requiring programmers to manually manage mutexes, predicates, and counters.

On the other hand, its single-use nature, lack of timeout support, and absence of built-in error-handling or cancellation mechanisms are important limitations.

Consequently, `std::latch` is an appropriate choice when several independent tasks must report completion once and the continuation of the application depends on all of them finishing.

For multi-phase synchronization, complex conditional waiting, or retrieving individual operation results, other Standard Library synchronization mechanisms should be considered.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>