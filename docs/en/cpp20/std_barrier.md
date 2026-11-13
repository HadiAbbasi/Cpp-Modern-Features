<div align="center">

[🇺🇸 English](./std_barrier.md) | [🇮🇷 فارسی](../../fa/cpp20/std_barrier.md)

</div>

---

# **Introduction to std::barrier in C++20**

## **Introduction**

In multithreaded programming, one of the major challenges is coordinating the execution of multiple threads. Many parallel algorithms require several threads to complete a particular stage of computation before all of them can proceed to the next stage.

For example, consider an image-processing algorithm divided among multiple threads. Each thread processes a portion of the image, but the next processing stage cannot begin until all threads have completed the current stage.

Tools such as `std::mutex` and `std::condition_variable` make it possible to implement this type of synchronization. However, implementing a barrier using these tools requires additional code, shared-state management, and careful handling of concurrency details.

C++20 introduced `std::barrier` to simplify this type of synchronization.

The `std::barrier` class is a phase-based synchronization primitive that allows a group of threads to wait at a specific synchronization point until all expected participants have arrived. Once a phase completes, all waiting threads can continue execution.

This feature is particularly useful for iterative parallel algorithms, staged data processing, scientific simulations, and computations that consist of multiple dependent phases.

## **What Is a Barrier?**

In concurrent programming, a barrier is a synchronization point that threads must reach before they can proceed. Each participating thread waits until all other expected participants have reached the same point.

Consider four threads that must execute an algorithm in three stages:

- Stage 1: Each thread processes a portion of the data.
- Stage 2: The results from Stage 1 are combined or examined.
- Stage 3: Further computations are performed using the results of Stage 2.

In this situation, starting Stage 2 before every thread has completed Stage 1 may result in incomplete data being used.

The behavior of a barrier can be illustrated as follows:

```
Thread 1 ───── Phase 1 ───── Wait ───── Phase 2 ─────
Thread 2 ───── Phase 1 ───── Wait ───── Phase 2 ─────
Thread 3 ───── Phase 1 ───── Wait ───── Phase 2 ─────
Thread 4 ───── Phase 1 ───── Wait ───── Phase 2 ─────
                                |
                         All threads arrive
                                |
                         Barrier completes
```

In this model, no thread can pass the barrier until all expected participants have arrived.

### **Barrier vs. Other Synchronization Primitives**

Each synchronization primitive is designed to solve a particular type of problem.

| Primitive                 | Primary purpose                                             |
| ------------------------- | ----------------------------------------------------------- |
| `std::mutex`              | Protecting shared data against concurrent access            |
| `std::condition_variable` | Waiting for a condition to become true                      |
| `std::latch`              | Waiting for a counter to reach zero                         |
| `std::barrier`            | Synchronizing multiple threads across successive phases     |
| `std::semaphore`          | Controlling the number of concurrent accesses to a resource |
| `std::atomic`             | Performing atomic operations on data                        |

The main difference between `std::barrier` and `std::latch` is that a barrier can be reused.

A `std::latch` cannot be reset or reused for another phase after its counter reaches zero. In contrast, `std::barrier` can begin another phase after the current phase completes.

## **Introduction to std::barrier**

The `std::barrier` class is defined in the following header:

```
#include <barrier>
```

It is part of the C++20 Standard Library and is designed to coordinate multiple participants.

Its general class template declaration is:

```
template<class CompletionFunction = /* implementation-defined */>
class barrier;
```

The template parameter specifies a completion function that executes when a phase completes.

For typical use cases, the default completion function is sufficient:

```
std::barrier barrier{4};
```

In this example, a barrier is created with four expected participants.

All four participants must arrive before the current phase can complete.

### **Class Operations**

The most important public operations provided by `std::barrier` are:

| Operation           | Description                                                                     |
| ------------------- | ------------------------------------------------------------------------------- |
| `arrive()`          | Registers a participant's arrival at the barrier                                |
| `wait()`            | Waits for the current phase to complete                                         |
| `arrive_and_wait()` | Registers an arrival and waits for the phase to complete                        |
| `arrive_and_drop()` | Registers an arrival and permanently removes the participant from future phases |
| `max()`             | Returns the maximum supported expected count                                    |

The class also provides the concepts of a completion function and a phase, which are explained in the following sections.

## **Understanding Phases in std::barrier**

One of the most important concepts when working with `std::barrier` is the phase.

A barrier consists of a sequence of consecutive phases. Each new phase begins after the previous phase has completed.

When a barrier is constructed, its initial number of expected participants is specified. This value is called the **expected count**.

For example:

```
std::barrier syncPoint{3};
```

In this example, the initial expected count is three.

Each time a thread arrives at the barrier, the number of remaining expected arrivals decreases.

When the expected count reaches zero:

- The current phase completes.
- The completion function executes, if one was provided.
- The expected count is adjusted for the next phase.
- Waiting threads are allowed to continue execution.

The expected count for future phases can also change through `arrive_and_drop()`.

### **The Lifecycle of a Phase**

The general lifecycle of a phase looks like this:

```
       Phase N begins
              |
              v
    Participants arrive
              |
              v
    Expected Count reaches 0
              |
              v
    Completion Function executes
              |
              v
       Phase N completes
              |
              v
       Phase N+1 begins
              |
              v
       Participants continue
```

An important detail is that phase completion is not determined by the number of calls to `wait()`. Instead, it is determined by the expected number of arrival operations and any changes to the expected count.

## **Creating a std::barrier**

To create a basic barrier, simply pass the expected number of participants to its constructor.

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 4;

    std::barrier syncPoint{threadCount};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &syncPoint] {
            std::cout << "Thread " << i << " is working\n";

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " passed the barrier\n";
        });
    }
}
```

In this example:

- Four threads are created.
- Each thread performs its initial operation.
- Each thread then reaches the barrier using `arrive_and_wait()`.
- No thread can pass the barrier until all expected participants have arrived.

This example uses `std::jthread`, introduced in C++20, which automatically joins the thread when its object is destroyed.

## **Understanding arrive()**

The `arrive()` function registers a participant's arrival at the barrier.

```
std::barrier syncPoint{3};

auto token = syncPoint.arrive();
```

When this function is called:

- The participant's arrival is registered.
- The expected count is decremented by one.
- The calling thread does not wait for the barrier to complete.
- An `arrival_token` is returned.

The return type is:

```
std::barrier<>::arrival_token
```

This token is used to associate the arrival operation with a subsequent wait operation.

### **Using arrive() Without Waiting Immediately**

Sometimes a thread needs to register its arrival but still has independent work to perform before waiting for the barrier.

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 3;

    std::barrier syncPoint{threadCount};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &syncPoint] {
            std::cout << "Thread " << i << " completed its work\n";

            auto token = syncPoint.arrive();

            std::cout << "Thread " << i
                      << " is performing independent work\n";

            syncPoint.wait(std::move(token));

            std::cout << "Thread " << i
                      << " passed the barrier\n";
        });
    }
}
```

In this example, each thread first registers its arrival and then performs independent work.

After completing that work, it waits for the phase to complete using the token associated with its arrival.

This pattern is useful when a thread can perform some independent operations without needing results from other threads.

### **Important Details About arrival_token**

The `arrival_token` type carries information associated with an arrival operation so that it can be used by `wait()`.

Important considerations include:

- A token must be used with the same barrier on which its associated arrival was registered.
- A token associated with one phase must not be used for another phase.
- Tokens must be transferred correctly because this type is designed for move operations.
- A token must not be reused for multiple independent wait operations.
- Using an invalid token or one associated with another barrier results in undefined behavior.

## **Understanding wait()**

The `wait()` function waits for the current phase to complete.

It accepts an `arrival_token`:

```
syncPoint.wait(std::move(token));
```

The `wait()` function does not decrement the expected count.

This distinction is important because the arrival operation has already been registered through `arrive()`.

In other words, `wait()` simply waits for the phase associated with the registered arrival to complete.

### **Difference Between arrive() and wait()**

| Feature                       | `arrive()` | `wait()` |
| ----------------------------- | ---------- | -------- |
| Registers an arrival          | Yes        | No       |
| Decrements the expected count | Yes        | No       |
| Waits for phase completion    | No         | Yes      |
| Returns a token               | Yes        | No       |

Using these functions separately provides more precise control over when a thread waits.

## **Understanding arrive_and_wait()**

The `arrive_and_wait()` function is the most common way to use `std::barrier`.

It combines two operations into a single logical operation:

- Registering the participant's arrival.
- Waiting until the current phase completes.

Example:

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 4;

    std::barrier syncPoint{threadCount};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &syncPoint] {
            std::cout << "Starting phase 1: " << i << '\n';

            syncPoint.arrive_and_wait();

            std::cout << "Starting phase 2: " << i << '\n';

            syncPoint.arrive_and_wait();

            std::cout << "Starting phase 3: " << i << '\n';
        });
    }
}
```

In this example, each thread executes three stages of computation.

The barrier ensures that no thread enters the second stage until all participants have completed the first stage.

The same guarantee applies when transitioning from the second stage to the third.

### **When Should You Use arrive_and_wait()?**

This function is the best choice when a thread has no independent work to perform after reaching the barrier.

Typical use cases include:

- Completing a stage of parallel processing.
- Waiting for all workers to finish their computations.
- Coordinating iterative algorithms.
- Synchronizing transitions between stages of a simulation.

If a thread needs to perform independent work after registering its arrival, combining `arrive()` and `wait()` provides greater flexibility.

## **Completion Functions**

One of the important features of `std::barrier` is the ability to define a completion function.

This function executes when all expected arrivals for the current phase have been registered.

The completion function runs before the phase completes and can be used to perform operations shared between consecutive phases.

For example, suppose multiple threads have independently computed portions of some data. At the end of the phase, their results must be combined.

This operation can be performed inside the completion function.

### **Defining a Completion Function**

The completion function must be a callable with no arguments that is nothrow-invocable.

For example:

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 4;

    auto completion = []() noexcept {
        std::cout << "All threads completed the phase\n";
    };

    std::barrier syncPoint{threadCount, completion};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &syncPoint] {
            std::cout << "Thread " << i << " is working\n";

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " continues execution\n";
        });
    }
}
```

In this example, the completion function executes when all four threads have arrived at the barrier.

An important detail is that the completion function is executed by one of the participating threads, not necessarily by a particular thread.

Therefore, you should not assume that a specific thread will always execute it.

### **Completion Function Restrictions**

Several considerations should be taken into account when designing a completion function:

- The function must be `noexcept`.
- It must not perform blocking operations that depend on threads waiting on the same barrier.
- It must not cause deadlocks.
- It should execute within a reasonable amount of time.
- Long-running or computationally expensive operations should generally be avoided.

Additionally, if the completion function is a member function or modifies shared state, the lifetime and synchronization of the associated data must be managed correctly.

### **Understanding the Completion Function Type**

In the `std::barrier` class template, the completion function type is specified as the template parameter.

When using the default completion function, the barrier can be created as follows:

```
std::barrier syncPoint{4};
```

When providing a custom completion function, its type can be deduced using Class Template Argument Deduction:

```
auto completion = []() noexcept {};

std::barrier syncPoint{4, completion};
```

This eliminates the need to explicitly specify the template argument.

## **Managing Shared Data with Completion Functions**

One important use of completion functions is aggregating results produced by different threads.

Suppose each thread processes a separate portion of an array and stores its computation result in a shared array.

At the end of each phase, the results must be combined.

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>
#include <array>

int main() {
    constexpr int threadCount = 4;

    std::array<int, threadCount> partialResults{};
    int totalResult = 0;

    auto completion = [&]() noexcept {
        totalResult = 0;

        for (int value : partialResults) {
            totalResult += value;
        }

        std::cout << "Total result: " << totalResult << '\n';
    };

    std::barrier syncPoint{threadCount, completion};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &partialResults, &syncPoint] {
            partialResults[i] = (i + 1) * 10;

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " sees total: " << totalResult << '\n';
        });
    }
}
```

In this example:

- Each thread writes its own value to the array.
- All threads arrive at the barrier.
- The completion function calculates the sum of all values.
- After the phase completes, threads can continue and observe the final result.

Since each thread writes to a different array element, their write operations do not conflict.

Additionally, the completion function executes after all expected arrivals have been registered, and phase completion provides the necessary memory synchronization for subsequent operations to observe the results.

## **Understanding arrive_and_drop()**

Sometimes a thread no longer needs to participate in future phases after completing its work.

For example, in a parallel algorithm, some workers may finish processing a portion of the data and no longer need to participate in synchronization.

For these situations, `std::barrier` provides the `arrive_and_drop()` function.

This function performs two operations:

- Registers the calling thread's arrival for the current phase.
- Decrements the expected count for all future phases by one.

Example:

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    std::barrier syncPoint{3};

    std::vector<std::jthread> threads;

    for (int i = 0; i < 3; ++i) {
        threads.emplace_back([i, &syncPoint] {
            std::cout << "Thread " << i << " started\n";

            if (i == 2) {
                syncPoint.arrive_and_drop();
                std::cout << "Thread 2 dropped\n";
                return;
            }

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " passed the first phase\n";

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " passed the second phase\n";
        });
    }
}
```

In this example, thread number two registers its arrival and then permanently leaves the barrier.

As a result:

- The first phase still requires three arrivals.
- Future phases require only two arrivals.

### **Difference Between arrive_and_drop() and arrive()**

The main difference between these two functions is their effect on future phases.

| Feature                                         | `arrive()` | `arrive_and_drop()` |
| ----------------------------------------------- | ---------- | ------------------- |
| Registers an arrival for the current phase      | Yes        | Yes                 |
| Decrements the current phase's expected count   | Yes        | Yes                 |
| Decrements the expected count for future phases | No         | Yes                 |
| Can participate in the next phase               | Yes        | No                  |

An important detail is that `arrive_and_drop()` does not immediately remove the thread from the current phase. Its arrival still contributes to completing that phase.

### **Use Cases for arrive_and_drop()**

This function is appropriate when the number of participating threads decreases during execution.

Examples include:

- Algorithms in which some workers leave after reaching a final result.
- Processing systems where fewer participants are needed in each subsequent stage.
- Systems in which workers no longer need synchronization after completing a specific task.

However, this function is not suitable for temporarily leaving a barrier and rejoining it later.

## **Understanding Expected Count and Arrival Operations**

Correctly understanding the expected count is essential for avoiding deadlocks and invalid behavior.

Suppose a barrier is created with an initial expected count of four:

```
std::barrier syncPoint{4};
```

At the beginning of the phase, the expected count is four.

Each arrival operation decrements this count.

| Operation      | Remaining expected count |
| -------------- | ------------------------ |
| Initial value  | 4                        |
| First arrival  | 3                        |
| Second arrival | 2                        |
| Third arrival  | 1                        |
| Fourth arrival | 0                        |

When the count reaches zero, the phase completes.

### **Important Considerations About Arrival Counts**

Each participant must perform exactly the number of arrival operations allocated to it.

If a barrier expects four arrivals but only three threads reach it, the phase cannot complete.

Similarly, if a thread performs more arrival operations than its allocated share, the counter may enter an invalid state, resulting in undefined behavior.

Therefore, the number of arrival operations must be consistent with the algorithm's design.

## **Understanding max()**

The `max()` function returns the maximum supported value for the expected count.

Its signature is:

```
static constexpr std::ptrdiff_t max() noexcept;
```

This function is static, so it does not require an instance of `std::barrier`.

Example:

```
#include <barrier>
#include <iostream>

int main() {
    std::cout << std::barrier<>::max() << '\n';
}
```

The returned value is implementation-dependent. You should not assume that all compilers and platforms provide the same value.

Additionally, the initial expected count must not exceed this limit.

## **Memory Synchronization and Memory Ordering**

One of the most important properties of `std::barrier` is the memory synchronization it provides between participating threads.

In multithreaded programs, simply bringing threads to a common synchronization point is not enough. Changes to shared data must also become correctly observable by other threads.

This is closely related to the *happens-before* relationship in the C++ Memory Model.

### **Synchronization Guarantees of a Barrier**

Under the C++20 standard, the completion step has specific synchronization relationships with arrival and wait operations.

In summary:

- Arrival operations performed by participants in the current phase are sequenced before the completion step.
- The completion step is sequenced before the return of wait operations associated with that phase.
- Operations performed by threads before reaching the barrier can become visible to other threads after the barrier, provided that the accesses are correctly ordered through this synchronization.

This property means that many scenarios do not require a mutex to transfer results between phases.

### **Example: Observing Shared Data**

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 3;

    int sharedData = 0;

    std::barrier syncPoint{threadCount};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &sharedData, &syncPoint] {
            if (i == 0) {
                sharedData = 100;
            }

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " reads: " << sharedData << '\n';
        });
    }
}
```

In this example, thread number zero modifies the shared variable.

After passing the barrier, the other threads can observe the updated value.

This is possible because of the memory synchronization provided by the barrier.

However, this guarantee does not make concurrent unsynchronized writes to the same data valid.

If multiple threads write to the same object concurrently, appropriate synchronization mechanisms such as mutexes or atomic operations are still required.

## **std::barrier vs. std::latch**

Both `std::barrier` and `std::latch` were introduced in C++20 and are used for thread synchronization.

However, they have different usage models.

### **Introduction to std::latch**

The `std::latch` class is a one-shot synchronization counter.

When its counter reaches zero, all waiting threads are released.

Once the counter reaches zero, it cannot be reset.

Example:

```
#include <latch>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 4;

    std::latch completion{threadCount};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &completion] {
            std::cout << "Thread " << i << " working\n";

            completion.count_down();

            completion.wait();

            std::cout << "All threads completed\n";
        });
    }
}
```

### **Feature Comparison**

| Feature                        | `std::barrier`                  | `std::latch`                                |
| ------------------------------ | ------------------------------- | ------------------------------------------- |
| Introduced in                  | C++20                           | C++20                                       |
| Reusable                       | Yes                             | No                                          |
| Supports multiple phases       | Yes                             | No                                          |
| Completion function            | Yes                             | No                                          |
| Can reduce future participants | Yes, through Drop               | No                                          |
| Primary use case               | Synchronizing successive stages | Waiting for a set of operations to complete |

### **Choosing the Right Primitive**

If the goal is to wait for a collection of independent operations to finish, `std::latch` is generally the simpler choice.

However, if threads must execute multiple stages of an algorithm in a coordinated manner, `std::barrier` is usually more appropriate.

## **std::barrier vs. std::condition_variable**

The `std::condition_variable` class has been available since C++11 and allows threads to wait until a condition becomes true.

It is more flexible than a barrier, but its synchronization logic is more complicated to manage.

Correctly using `std::condition_variable` typically requires:

- A mutex to protect shared state.
- A condition that must become true.
- Handling spurious wakeups.
- Correct management of waiting and notification states.

In contrast, `std::barrier` provides this synchronization behavior as a ready-to-use abstraction.

### **When Is condition_variable More Appropriate?**

If the number of participants is dynamic or threads must be released based on a changing condition, `std::condition_variable` provides greater flexibility.

Examples include:

- Waiting for a task to enter a queue.
- Waiting for a resource to become available.
- Producer-consumer synchronization.
- Waiting for a complex condition.

However, if the goal is simply to coordinate a fixed number of threads across multiple successive phases, `std::barrier` is generally more readable and less error-prone.

## **Practical Applications in Parallel Algorithms**

One of the most important applications of `std::barrier` is coordinating algorithms that consist of multiple consecutive computational stages.

### **Image Processing**

Suppose a large image is divided into several sections, with each thread processing one section.

Image processing may involve the following stages:

- Initial pixel calculations.
- Applying a filter.
- Combining results.
- Performing the next processing stage.

In such algorithms, some stages depend on the results produced by all threads.

Using a barrier ensures that threads wait for the current stage to finish before starting the next one.

### **Scientific Simulations**

In simulations such as particle simulations and physical system modeling, different parts of the system may be updated during each iteration.

In these situations, a barrier can be placed between iterations to ensure that all threads have completed the current state update before proceeding.

### **Iterative Algorithms**

Some algorithms, such as certain numerical computing methods, perform computations iteratively.

Each iteration may involve:

- Computing independent portions of the problem.
- Aggregating results.
- Checking a termination condition.
- Starting the next iteration.

A barrier is particularly useful for coordinating these stages.

## **Practical Example: A Multi-Phase Algorithm with Result Aggregation**

In the following example, four threads calculate their individual values over multiple iterations.

At the end of each iteration, the completion function calculates the sum of all values.

```
#include <array>
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 4;
    constexpr int iterations = 3;

    std::array<int, threadCount> results{};
    int total = 0;

    auto completion = [&]() noexcept {
        total = 0;

        for (int value : results) {
            total += value;
        }

        std::cout << "Iteration completed. Total = "
                  << total << '\n';
    };

    std::barrier syncPoint{threadCount, completion};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &results, &syncPoint] {
            for (int iteration = 0; iteration < iterations; ++iteration) {
                results[i] = (iteration + 1) * (i + 1);

                syncPoint.arrive_and_wait();
            }
        });
    }
}
```

In this program, each thread stores a new value in its own array element during every iteration.

After all threads reach the barrier, the completion function executes and calculates the sum of the results.

This process is repeated three times.

An important detail is that `total` is modified by the completion function, and threads can observe its updated value after passing the barrier.

## **Using std::barrier in Thread Pool Designs**

In some thread pool designs, a collection of workers may process tasks in multiple batches.

For example, a system might operate as follows:

```
Batch 1:
    Worker 1 processes tasks
    Worker 2 processes tasks
    Worker 3 processes tasks
              |
           Barrier
              |
Batch 2:
    Worker 1 processes tasks
    Worker 2 processes tasks
    Worker 3 processes tasks
              |
           Barrier
              |
Batch 3
```

In this design, a barrier can ensure that all workers finish the previous batch before starting the next one.

However, using a barrier in a general-purpose thread pool is not always appropriate.

If tasks are independent and the number of active workers may change, a queue, condition variable, or task scheduler generally provides greater flexibility.

## **Limitations of std::barrier**

Despite its usefulness, `std::barrier` is not suitable for every synchronization problem.

### **Fixed Participant Structure**

The initial number of participants is specified when the barrier is constructed.

Under normal operation, this number cannot be increased during execution.

The only standard mechanism for reducing the number of expected participants in future phases is `arrive_and_drop()`.

There is no mechanism for adding new participants to an existing barrier.

### **Dependency on All Participants**

Completing each phase depends on receiving arrivals from all expected participants.

If a thread:

- Stops executing.
- Enters a deadlock.
- Fails to perform its arrival operation.
- Exits early because of an error.

Other threads may wait indefinitely.

### **No Timeout Support for Wait**

The `wait()` function does not accept a timeout parameter.

Therefore, it cannot directly specify a maximum waiting duration.

If timeout support is required, other synchronization primitives, such as `std::condition_variable`, may be more appropriate.

### **No Guarantee About Thread Execution Order**

A barrier only coordinates phase completion.

It does not guarantee any particular execution order for threads after they pass the barrier.

Therefore, you cannot assume that thread number one will always resume before thread number two.

### **No Fairness Guarantee**

The standard does not guarantee that threads will receive CPU time in a particular order or fairly after a barrier completes.

Thread scheduling remains under the control of the operating system and thread scheduler.

## **Common Mistakes When Using std::barrier**

### **Not All Threads Reach the Barrier**

One of the most common mistakes is incorrectly designing thread execution paths.

For example:

```
if (someCondition) {
    syncPoint.arrive_and_wait();
}
```

If some threads enter this condition while others do not, the barrier may never complete.

When designing a barrier, ensure that all expected participants follow valid arrival paths.

### **Performing Too Many Arrival Operations**

Each participant must respect its allocated number of arrival operations.

Calling `arrive()` or `arrive_and_wait()` more times than permitted may violate the barrier's contract and result in undefined behavior.

### **Confusing wait() with arrive()**

The `wait()` function does not decrement the expected count.

Therefore, if you expect a thread to register its arrival by calling `wait()`, the design is incorrect.

Use `arrive()` or `arrive_and_wait()` to register an arrival.

### **Incorrect Use of arrival_token**

A token must be compatible with the arrival operation and phase for which it was created.

Reusing or misusing an invalid token can result in undefined behavior.

### **Performing Blocking Operations in the Completion Function**

The completion function is part of the phase-completion process.

If it waits for threads that depend on its completion to continue, a deadlock may occur.

### **Destroying the Barrier Before Its Use Is Complete**

The lifetime of the barrier must extend beyond all threads that access it.

If the barrier is destroyed before the relevant thread operations have completed, the program's behavior is undefined.

For this reason, thread joining and the lifetimes of shared objects must be managed correctly.

## **Advanced Design Considerations**

### **Using std::barrier with std::jthread**

The `std::jthread` class was introduced in C++20 and simplifies thread lifetime management.

It automatically joins the thread when its object is destroyed.

Using it alongside `std::barrier` can make program structure simpler.

However, note that the `std::jthread` destructor first calls `request_stop()` and then joins the thread.

Consequently, if your threads do not use stop tokens, requesting a stop does not necessarily cause them to exit while waiting on a barrier.

### **Coordinating Stop Tokens and Barriers**

One important limitation of `std::barrier` is that its `wait()` function does not directly support `std::stop_token`.

Therefore, requesting a stop on a `std::jthread` does not automatically release a thread waiting at a barrier.

When designing interruptible systems, you must also account for how threads will exit their barrier synchronization.

For example, `arrive_and_drop()` can be used by threads that decide not to participate in future phases. However, this operation must still be coordinated with the expected participant count.

### **Avoiding Deadlocks in Completion Functions**

To prevent deadlocks, completion functions should generally perform short, non-blocking operations.

Suitable operations include:

- Aggregating computation results.
- Updating shared state.
- Calculating values needed by the next phase.
- Recording simple statistics.

In contrast, the following operations are generally poor choices:

- Waiting for other threads.
- Waiting on another barrier with cyclic dependencies.
- Performing lengthy I/O operations.
- Waiting for responses from external services.

### **Using Multiple Barriers**

In some complex algorithms, multiple barriers may be required to coordinate different groups of threads.

For example:

```
Group A:
    Worker 1
    Worker 2
    Worker 3
        |
     Barrier A
        |
        v
Group B:
    Worker 4
    Worker 5
    Worker 6
        |
     Barrier B
        |
        v
    Next phase
```

In these designs, dependencies between barriers must be examined carefully.

If one thread waits at the first barrier for another thread that is itself waiting at the second barrier, a deadlock may occur.

### **Using std::barrier in Heterogeneous Environments**

In applications where computations are distributed between CPUs and accelerators such as GPUs, `std::barrier` alone is not sufficient to synchronize all resources.

This class is designed for threads participating in the standard C++ execution model. It does not provide direct synchronization guarantees for GPU operations or external devices.

In such environments, synchronization mechanisms provided by the relevant accelerator runtime must also be used.

## **Performance Considerations**

The performance of `std::barrier` depends on several factors:

- Number of participants.
- Number of phases.
- Standard library implementation.
- Number of processor cores.
- Thread scheduler behavior.
- Amount of work performed by each thread.
- Differences in the time threads take to reach the barrier.

### **Waiting Overhead**

If one thread reaches the barrier significantly earlier than the others, it must wait.

During this time, processing resources may be allocated to other threads.

As a result, the overall algorithm's performance is generally influenced by its slowest participant.

### **Importance of Load Balancing**

Suppose four threads perform the following tasks:

```
Thread 1: 10 ms
Thread 2: 12 ms
Thread 3: 11 ms
Thread 4: 80 ms
```

In this situation, the first three threads must wait until the fourth thread finishes.

Therefore, even if the barrier itself is implemented efficiently, poor load balancing can significantly reduce performance.

### **Completion Function Overhead**

The completion function is part of the phase-completion process.

Consequently, performing expensive operations inside it affects all threads waiting for the phase to complete.

Completion functions should be kept as short as possible.

### **Busy Waiting and Blocking**

The standard does not prescribe a specific internal waiting algorithm for `std::barrier`.

An implementation may use a combination of atomic operations, waiting mechanisms, and operating-system facilities.

Therefore, you should not assume that `std::barrier` is implemented in exactly the same way across all compilers and platforms.

If performance is important to your application, benchmark the actual implementation used in your environment.

## **std::barrier vs. std::atomic**

The `std::atomic` class is designed to perform atomic operations on data.

In contrast, `std::barrier` is a synchronization primitive that coordinates a group of threads.

For example, an atomic counter can be used to count completed operations. However, implementing a complete reusable barrier with an atomic counter requires additional logic for phase management, waiting, and releasing threads.

Therefore:

- `std::atomic` is suitable for managing atomic data operations.
- `std::barrier` is suitable for coordinating multiple threads across successive phases.

Both primitives can be used together in the same application.

## **Compiler Support**

`std::barrier` is part of the C++20 Standard Library.

However, actual support depends on the compiler version and the standard library implementation being used.

When using this feature, you should check:

- Compiler version.
- `libstdc++` or `libc++` version in GCC- and Clang-based environments.
- Microsoft Standard Library version when using MSVC.
- Whether C++20 is enabled in the build configuration.

Common commands for enabling C++20 are shown below.

**GCC:**

```
g++ -std=c++20 -pthread main.cpp -o app
```

**Clang:**

```
clang++ -std=c++20 -pthread main.cpp -o app
```

**MSVC:**

```
cl /std:c++20 /EHsc main.cpp
```

An important detail is that enabling the C++20 language standard does not, by itself, guarantee that every Standard Library feature is supported by the installed library version.

For more precise checks, you can use the Compiler Feature Test Macro.

### **Checking the Feature Test Macro**

The following macro is provided for `std::barrier`:

```
__cpp_lib_barrier
```

This macro is available when the Standard Library supports the corresponding feature.

Example:

```
#include <barrier>
#include <iostream>

int main() {
#ifdef __cpp_lib_barrier
    std::cout << "std::barrier is supported\n";
#else
    std::cout << "std::barrier is not supported\n";
#endif
}
```

The standardized value of this feature test macro is `201907L`.

This macro is more appropriate for checking Standard Library support than relying solely on the compiler version.

## **Summary**

The `std::barrier` class is an important synchronization primitive introduced in the C++20 Standard Library.

It is designed to coordinate multiple threads across successive phases of a parallel algorithm.

Its most important capabilities include:

- Synchronizing multiple threads within each phase.
- Supporting multiple consecutive phases.
- Registering arrivals without immediately waiting through `arrive()`.
- Waiting for phase completion through `wait()`.
- Combining arrival and waiting through `arrive_and_wait()`.
- Permanently removing participants from future phases through `arrive_and_drop()`.
- Executing a completion function when a phase completes.
- Providing the memory synchronization necessary for transferring results between threads.

When choosing this primitive, consider the fixed participant structure, the possibility that a thread may fail to reach the barrier, the lifetime of shared objects, and the overhead associated with waiting.

In general, if your algorithm consists of multiple computational stages and threads must synchronize at the end of each stage, `std::barrier` is one of the most suitable and readable synchronization tools available in C++20.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>