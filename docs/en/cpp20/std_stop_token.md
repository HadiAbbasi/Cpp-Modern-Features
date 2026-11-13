<div align="center">

[🇺🇸 English](./std_stop_token.md) | [🇮🇷 فارسی](../../fa/cpp20/std_stop_token.md)

</div>

---

# A Comprehensive Guide to `std::stop_token` in C++20

## Introduction

Managing the lifecycle of threads and controlling how concurrent operations terminate are important challenges in C++ concurrent programming. In many applications, a thread may be responsible for a long-running task, such as processing data, receiving information over a network, executing a computational loop, or managing a background service.

In such situations, another part of the application may decide that the operation should be stopped. The challenge is how to communicate this request to the executing thread without abruptly interrupting it while it is in an unsafe state.

C++20 introduced the Stop Token family as a standardized solution to this problem. This mechanism is based on the concept of Cooperative Cancellation: one part of the application requests cancellation, and the executing part responds to that request at an appropriate point.

The Stop Token family consists of three main classes:

- `std::stop_source`: Responsible for issuing stop requests.
- `std::stop_token`: Responsible for observing the stop-request state.
- `std::stop_callback`: Responsible for executing a callback when a stop request is made.

This functionality is particularly important when used alongside `std::jthread` and cancellable APIs, making it a valuable part of modern concurrent application design.

## The Concept of Cooperative Cancellation

### Definition of Cooperative Cancellation

Cooperative Cancellation is a technique in which a stop request is sent from one part of an application to another, while the executing part retains control over when and how the operation terminates.

For example, suppose a thread is responsible for processing a collection of files. Another thread can issue a stop request, but the processing thread must check for that request at specific points during its execution.

After observing the request, the thread can:

- Complete the current processing step.
- Release the resources it owns.
- Restore its internal state to a valid condition.
- Exit the processing loop.
- Report the cancellation result to the caller.

Compared with forced termination, this approach is safer and easier to control.

### Cooperative Cancellation vs. Forced Termination

With forced termination, another thread attempts to stop the target thread without its cooperation. This approach can leave resources in an invalid state.

For example, the target thread might be interrupted while it is:

- Holding a Mutex.
- Modifying a shared data structure.
- Performing a transaction that has not yet been committed.
- Writing data to a file.
- Managing a Socket or another external resource.

In contrast, Cooperative Cancellation allows the thread to stop at a safe point in its execution.

Important: The Stop Token mechanism in C++20 provides Cooperative Cancellation. It does not provide a way to forcibly terminate a thread.

## Introducing the Stop Token Family

### Introducing `std::stop_source`

The `std::stop_source` class is responsible for managing and issuing stop requests.

It allows you to create a Stop State and obtain a Token associated with that state.

Its most important member functions are:

- `get_token()`: Returns a Token associated with the Stop State.
- `request_stop()`: Registers a stop request.
- `stop_requested()`: Checks whether a stop request has been registered.
- `stop_possible()`: Checks whether a stop request is possible.

Example:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token = source.get_token();

    std::cout << std::boolalpha;
    std::cout << token.stop_requested() << '\n';

    source.request_stop();

    std::cout << token.stop_requested() << '\n';
}
```

Output:

```
false
true
```

In this example, the stop request is issued through `source`, while its state can be observed through `token`.

### Introducing `std::stop_token`

The `std::stop_token` class is a copyable handle that allows you to observe the state of a stop request.

It does not issue stop requests directly, nor is it responsible for managing a thread's lifecycle.

Its main characteristics include:

- It supports copying and moving.
- It can be shared among multiple threads.
- It observes the associated Stop State.
- It is suitable for passing cancellation support to functions that are independent of a particular thread.
- Once a stop request has been registered, the Stop State cannot be reset to its initial state.

### Introducing `std::stop_callback`

The `std::stop_callback` class is used to register a Callable that will execute when a stop request is registered.

This functionality is particularly useful for waking up waiting operations or notifying other parts of an application.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    auto token = source.get_token();

    std::stop_callback callback(
        token,
        [] {
            std::cout << "Stop requested\n";
        }
    );

    source.request_stop();
}
```

Output:

```
Stop requested
```

## Internal Structure and the Stop State Concept

### Relationships Between Stop Token Classes

Internally, `std::stop_source`, `std::stop_token`, and `std::stop_callback` work with a shared state called the Stop State.

This state stores information about stop requests and registered callbacks.

The conceptual relationship between these objects is illustrated below:

```
             std::stop_source
                    |
                    |
                Stop State
                /        \
               /          \
      std::stop_token   std::stop_callback
              |
              |
          Worker Thread
```

Every Token obtained from a Stop Source is associated with the same Stop State.

Therefore, copying a Token does not create an independent stop state. All copies observe the same shared state.

### Stop State Ownership and Lifetime

The Stop State uses shared lifetime management.

It can remain alive as long as an associated `std::stop_source` or `std::stop_token` exists.

Therefore, destroying a `std::stop_source` object does not necessarily destroy the Stop State.

For example:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_token token;

    {
        std::stop_source source;

        token = source.get_token();
        source.request_stop();
    }

    std::cout << std::boolalpha;
    std::cout << token.stop_possible() << '\n';
    std::cout << token.stop_requested() << '\n';
}
```

Output:

```
true
true
```

In this example, after `source` is destroyed, the Token can still access the Stop State and observe the previously registered stop request.

Note: The lifetime of the Stop State is not necessarily the same as the lifetime of external objects used by a callback. If a callback holds a reference to an external object, the validity of that reference must be guaranteed independently.

## Creating and Obtaining Stop Tokens

### Obtaining a Token from a Stop Source

The most common way to obtain a Token is to call `get_token()`.

```
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token = source.get_token();
}
```

The `get_token()` function returns a Token associated with the Stop State owned by that Source.

You can pass this Token to different functions so that they can also check for stop requests.

### Obtaining a Token from `std::jthread`

One of the most important uses of `std::stop_token` is alongside `std::jthread`.

The `std::jthread` class was introduced in C++20 and provides useful features for managing thread lifetimes.

If the Callable associated with a `std::jthread` can accept a `std::stop_token` as its first argument, the Token is passed to it automatically.

```
#include <iostream>
#include <stop_token>
#include <thread>
#include <chrono>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Working...\n";

        std::this_thread::sleep_for(
            std::chrono::milliseconds(200)
        );
    }

    std::cout << "Worker stopped\n";
}

int main() {
    std::jthread thread(worker);

    std::this_thread::sleep_for(
        std::chrono::seconds(1)
    );

    thread.request_stop();
}
```

In this example:

- A thread is created using `std::jthread`.
- The Token associated with the thread is passed to the `worker` function.
- The Worker periodically checks the stop-request state.
- Calling `request_stop()` registers a stop request.
- The Worker exits the loop when it checks the state again.
- At the end, if the thread has not been joined, the `std::jthread` destructor requests a stop and then waits for the thread to finish.

Note: Sending a stop request and joining a thread are two different operations. The `request_stop()` function does not wait for the thread to finish.

### Passing Additional Arguments to a Callable

With `std::jthread`, the Token is passed as the first argument to the Callable, followed by any additional arguments.

```
#include <iostream>
#include <stop_token>
#include <thread>
#include <chrono>

int main() {
    std::jthread worker(
        [](std::stop_token token, int value) {
            while (!token.stop_requested()) {
                std::cout << value << '\n';

                std::this_thread::yield();
            }
        },
        42
    );

    std::this_thread::sleep_for(
        std::chrono::milliseconds(100)
    );

    worker.request_stop();
}
```

In this example, the value `42` is passed to the Lambda as its second argument.

### Creating a Stop Source Without a Stop State

In some situations, you may want a `std::stop_source` that does not create a Stop State at all.

You can use `std::nostopstate` for this purpose.

```
#include <stop_token>
#include <iostream>

int main() {
    std::stop_source source(std::nostopstate);

    std::stop_token token = source.get_token();

    std::cout << std::boolalpha;
    std::cout << token.stop_possible() << '\n';
    std::cout << token.stop_requested() << '\n';
}
```

Output:

```
false
false
```

This feature is useful for APIs that accept optional Tokens but do not need to create a Stop State in every situation.

## Exploring the Functions and Features of `std::stop_token`

### Checking `stop_requested()`

The `stop_requested()` function indicates whether a stop request has been registered for the associated Stop State.

Its signature is:

```
bool stop_requested() const noexcept;
```

The function returns `true` if a stop request has been registered and `false` otherwise.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    auto token = source.get_token();

    std::cout << std::boolalpha;

    std::cout << token.stop_requested() << '\n';

    source.request_stop();

    std::cout << token.stop_requested() << '\n';
}
```

Output:

```
false
true
```

This function is thread-safe and can be called concurrently from multiple threads.

Important: A return value of `false` only reflects the Token's state at the time of the call. A stop request may be registered immediately after the function returns.

Therefore, this function must not be used as a substitute for a Lock or another synchronization mechanism.

### Checking `stop_possible()`

The `stop_possible()` function indicates whether the Token can receive a stop request at all.

Its signature is:

```
bool stop_possible() const noexcept;
```

This function can return `false` in the following situations:

- The Token is empty.
- The Token is associated with a Stop State that has no active Stop Source and has not previously received a stop request.
- The Token was obtained from a Stop Source without a Stop State.

Example:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_token emptyToken;

    std::stop_source source;

    auto token = source.get_token();

    std::cout << std::boolalpha;

    std::cout << emptyToken.stop_possible() << '\n';
    std::cout << token.stop_possible() << '\n';
}
```

Output:

```
false
true
```

The distinction between this function and `stop_requested()` is important.

| State                               | `stop_possible()` | `stop_requested()` |
| ----------------------------------- | ----------------- | ------------------ |
| Empty Token                         | `false`           | `false`            |
| Active Token without a stop request | `true`            | `false`            |
| Active Token with a stop request    | `true`            | `true`             |
| Token without a Stop State          | `false`           | `false`            |

After a stop request has been registered, remaining Tokens can still observe its state, even if no active Stop Source exists anymore.

### Comparing Tokens with `operator==`

The `std::stop_token` class supports the equality comparison operator.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    auto token1 = source.get_token();
    auto token2 = source.get_token();

    std::stop_token emptyToken;

    std::cout << std::boolalpha;

    std::cout << (token1 == token2) << '\n';
    std::cout << (token1 == emptyToken) << '\n';
}
```

Output:

```
true
false
```

Two Tokens are equal if they are associated with the same Stop State or if both are empty.

This comparison does not check the stop-request state. It compares the Tokens' association with their Stop State.

### Copying and Moving Tokens

The `std::stop_token` class supports Copy Construction, Copy Assignment, Move Construction, and Move Assignment.

```
#include <stop_token>
#include <utility>

int main() {
    std::stop_source source;

    std::stop_token token1 = source.get_token();

    std::stop_token token2 = token1;

    std::stop_token token3 = std::move(token2);
}
```

Copying a Token creates a new handle to the same Stop State.

With a Move operation, the Token's association is transferred to the destination object. The source object remains valid after the move, but its value is unspecified, so no assumptions should be made about its association with the Stop State.

## Issuing Stop Requests with `std::stop_source`

### Using `request_stop()`

The `request_stop()` function is the primary way to issue a stop request.

Its signature is:

```
bool request_stop() noexcept;
```

This function attempts to register a stop request for the associated Stop State.

If the stop request is registered for the first time, the function returns `true`.

If a stop request has already been registered, the function returns `false`.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::cout << std::boolalpha;

    std::cout << source.request_stop() << '\n';
    std::cout << source.request_stop() << '\n';
}
```

Output:

```
true
false
```

A stop request can only be registered once. After it has been registered, the Stop State cannot be reset.

### Interpreting the Return Value of `request_stop()`

The return value indicates whether the current call registered the stop request for the first time.

A return value of `true` does not mean that the operation has finished.

For example:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    if (source.request_stop()) {
        std::cout << "Stop request accepted\n";
    } else {
        std::cout << "Stop was already requested\n";
    }
}
```

In this example, accepting a stop request is distinct from the Worker finishing its execution.

### Issuing a Stop Request Through `std::jthread`

The `std::jthread` class provides a `request_stop()` function that issues a stop request for the thread's internal Stop State.

```
#include <iostream>
#include <stop_token>
#include <thread>

int main() {
    std::jthread worker(
        [](std::stop_token token) {
            while (!token.stop_requested()) {
                std::this_thread::yield();
            }
        }
    );

    bool requested = worker.request_stop();

    std::cout << std::boolalpha << requested << '\n';
}
```

This function provides direct access to the Stop State associated with the thread.

### The Difference Between a Stop Request and Thread Completion

When designing concurrent applications, the following three concepts must be kept separate:

- Stop Request: A request to stop has been registered.
- Thread Completion: The executing function has finished.
- Thread Join: The thread has been joined, and its completion has been observed by another thread.

These concepts are not equivalent.

For example, a stop request may be registered while the Worker is still completing a long-running operation.

## Designing Cancellable Functions

### Separating Processing Logic from Threads

One of the main advantages of `std::stop_token` is that it is not tied to a specific thread.

You can pass a Token to any function that needs cancellation support.

```
#include <iostream>
#include <stop_token>
#include <thread>
#include <chrono>

bool processItems(
    std::stop_token token,
    int itemCount
) {
    for (int i = 0; i < itemCount; ++i) {
        if (token.stop_requested()) {
            return false;
        }

        std::cout << "Processing item " << i << '\n';

        std::this_thread::sleep_for(
            std::chrono::milliseconds(100)
        );
    }

    return true;
}

int main() {
    std::stop_source source;

    std::jthread worker(
        [&source] {
            bool completed = processItems(
                source.get_token(),
                100
            );

            std::cout << "Completed: "
                      << std::boolalpha
                      << completed << '\n';
        }
    );

    std::this_thread::sleep_for(
        std::chrono::milliseconds(350)
    );

    source.request_stop();
}
```

In this design, the `processItems` function has no direct dependency on `std::jthread`.

This improves the function's reusability and testability.

### Passing Tokens to Nested Functions

In large systems, an operation may consist of multiple functions.

If several processing stages support cancellation, the Token can be passed to those functions.

```
#include <stop_token>

bool validateData(std::stop_token token) {
    if (token.stop_requested()) {
        return false;
    }

    return true;
}

bool transformData(std::stop_token token) {
    if (!validateData(token)) {
        return false;
    }

    if (token.stop_requested()) {
        return false;
    }

    return true;
}

bool executeTask(std::stop_token token) {
    return transformData(token);
}
```

With this architecture, each stage can independently respond to a stop request.

However, it is better to pass the Token only to the parts of the application that actually need cancellation support.

### Designing an Optional Token

In some APIs, cancellation support is optional.

In such cases, an empty Token can be used as the default value.

```
#include <stop_token>
#include <iostream>

void execute(std::stop_token token = {}) {
    if (token.stop_requested()) {
        return;
    }

    std::cout << "Executing operation\n";
}

int main() {
    execute();

    std::stop_source source;

    execute(source.get_token());
}
```

With this design, callers can use the function without creating a Stop Source.

An empty Token has no Stop State, so no stop request can be registered for it.

## Using Stop Callbacks

### Understanding Callbacks During Cancellation

Periodically checking `stop_requested()` is not always sufficient.

For example, a thread may be waiting to receive data from another system. If the thread is blocked in an operation, it may take a long time to reach the next point where it checks the Token.

In such situations, `std::stop_callback` makes it possible to execute a Callable when a stop request is registered.

### Registering a Callback

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    auto token = source.get_token();

    std::stop_callback callback(
        token,
        [] {
            std::cout << "Cancellation event received\n";
        }
    );

    source.request_stop();
}
```

The callback executes when the stop request is registered.

### Callback Behavior Across Different Threads

An important detail about `std::stop_callback` is that the callback is not necessarily executed in the thread that registered it.

If a stop request is issued by a thread, the registered callbacks are normally executed in that same thread.

However, if a callback is created after a stop request has already been registered, it executes in the thread that constructs the `std::stop_callback` object.

Therefore:

- A callback may execute in the thread that issues the stop request.
- A callback may execute while the callback object is being constructed.
- A callback may execute concurrently with other application operations.

For this reason, callbacks should be short, predictable, and thread-safe.

### Registering a Callback After a Stop Request

If the Stop State has already received a stop request, the callback executes immediately when the `std::stop_callback` object is constructed.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    source.request_stop();

    std::stop_callback callback(
        source.get_token(),
        [] {
            std::cout << "Callback executed immediately\n";
        }
    );
}
```

This behavior is important because you must not assume that a callback will execute only after its constructor returns.

### Removing a Callback

A callback remains registered for as long as its `std::stop_callback` object exists.

Destroying this object unregisters the callback.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    {
        std::stop_callback callback(
            source.get_token(),
            [] {
                std::cout << "Callback executed\n";
            }
        );
    }

    source.request_stop();
}
```

In this example, the callback object is destroyed before the stop request is issued, so the callback does not execute.

### Concurrency Considerations in the Callback Destructor

The destructor of `std::stop_callback` must coordinate with any concurrent execution of the callback.

If the callback is executing in another thread, destroying the callback object may wait for that execution to finish.

Therefore, destroying a callback from within the callback itself can lead to a deadlock.

A deadlock may also occur if the callback holds a Lock while executing and the thread destroying the callback needs that same Lock to proceed.

Recommendation: Avoid long-running operations, blocking operations, or unnecessary dependencies on external Locks inside callbacks.

### Using a Callback to Wake Up a Thread

One important use of callbacks is to wake a thread that is waiting.

For example, suppose a thread is waiting to receive data from a Queue. If the Queue is empty, the thread may remain blocked in a waiting operation.

A callback can be used to send a notification in this situation.

```
#include <condition_variable>
#include <mutex>
#include <stop_token>
#include <thread>
#include <iostream>

int main() {
    std::mutex mutex;
    std::condition_variable condition;
    bool ready = false;

    std::jthread worker(
        [&](std::stop_token token) {
            std::stop_callback callback(
                token,
                [&] {
                    condition.notify_all();
                }
            );

            std::unique_lock<std::mutex> lock(mutex);

            condition.wait(lock, [&] {
                return ready || token.stop_requested();
            });

            if (token.stop_requested()) {
                std::cout << "Worker cancelled\n";
                return;
            }

            std::cout << "Worker resumed\n";
        }
    );

    worker.request_stop();
}
```

In this example, the callback wakes up the Condition Variable when a stop request is issued.

However, C++20 provides a more direct solution for this type of waiting operation, which is covered in the next section.

## Cancellable Waiting with `std::condition_variable_any`

### The Problem with Blocking Operations

In many concurrent applications, threads wait for data to arrive or for a condition to become true.

For example:

```
condition.wait(lock);
```

If a stop request is issued, this operation is not necessarily awakened automatically.

As a result, a stop request may have been registered while the thread remains blocked in the waiting operation.

To address this problem, C++20 added new overloads to `std::condition_variable_any` that directly support `std::stop_token`.

### Using the Cancellable `wait()` Overload

The following example shows a Worker that waits until data is ready or a stop request is received.

```
#include <condition_variable>
#include <mutex>
#include <stop_token>
#include <thread>
#include <iostream>
#include <chrono>

int main() {
    std::mutex mutex;
    std::condition_variable_any condition;

    bool ready = false;

    std::jthread worker(
        [&](std::stop_token token) {
            std::unique_lock<std::mutex> lock(mutex);

            bool completed = condition.wait(
                lock,
                token,
                [&] {
                    return ready;
                }
            );

            if (!completed) {
                std::cout << "Operation cancelled\n";
                return;
            }

            std::cout << "Data is ready\n";
        }
    );

    std::this_thread::sleep_for(
        std::chrono::milliseconds(500)
    );

    worker.request_stop();
}
```

In this example, the Worker waits until the Predicate becomes true or a stop request is received.

If a stop request is received, the waiting operation wakes up and returns `false`.

### Understanding the `wait()` Signature

One of the overloads of this function has the following form:

```
template<class Lock, class Predicate>
bool wait(
    Lock& lock,
    std::stop_token stoken,
    Predicate pred
);
```

Its behavior is as follows:

- If the Predicate is satisfied, the function returns `true`.
- If a stop request is observed before the Predicate becomes satisfied, the function returns `false`.
- The Lock is released while the thread waits.
- The Lock is acquired again before the function returns.

An important detail is that the Predicate takes precedence over the stop state. If the Predicate is satisfied during the final check, the function returns `true`, even if a stop request has also been registered.

Therefore, the function's return value must be interpreted according to the application's intended policy.

### Using `wait_for()` and `wait_until()`

In addition to `wait()`, `std::condition_variable_any` in C++20 supports timed waiting overloads that accept a Stop Token.

These overloads include:

```
wait_until(lock, token, abs_time, predicate);
```

and:

```
wait_for(lock, token, rel_time, predicate);
```

These functions allow three conditions to be combined:

- The Predicate becomes satisfied.
- A stop request is received.
- The waiting period expires.

Example:

```
#include <condition_variable>
#include <mutex>
#include <stop_token>
#include <thread>
#include <chrono>
#include <iostream>

int main() {
    std::mutex mutex;
    std::condition_variable_any condition;

    bool ready = false;

    std::jthread worker(
        [&](std::stop_token token) {
            std::unique_lock<std::mutex> lock(mutex);

            bool completed = condition.wait_for(
                lock,
                token,
                std::chrono::seconds(5),
                [&] {
                    return ready;
                }
            );

            if (completed) {
                std::cout << "Operation completed\n";
            } else {
                std::cout << "Cancelled or timed out\n";
            }
        }
    );

    worker.join();
}
```

In this example, a return value of `false` does not, by itself, indicate whether the wait ended because of a timeout or a stop request.

If the application needs to distinguish between these causes, it must also check the Token state and account for its timing policy.

## Differences Between `std::condition_variable` and `std::condition_variable_any`

Both tools are used to coordinate threads and wait until a condition becomes satisfied. However, they differ in the types of Locks they support and their support for `std::stop_token`.

### Differences in Lock Types

The `std::condition_variable` class works only with `std::unique_lock<std::mutex>`. This restriction allows its implementation to use optimizations specific to `std::mutex`.

In contrast, `std::condition_variable_any` works with different Lock types, provided that the Lock type supplies the required operations, such as `lock()` and `unlock()`.

For example, `std::condition_variable_any` can be used with `std::shared_mutex` or custom Lock types.

### Differences in Stop Token Support

The most important difference for cancellation-aware programming is that, in C++20, `std::condition_variable_any` provides waiting overloads that directly accept a `std::stop_token`.

In contrast, `std::condition_variable` does not provide these overloads.

Example using `std::condition_variable_any`:

```
#include <condition_variable>
#include <mutex>
#include <stop_token>
#include <thread>
#include <iostream>

int main() {
    std::mutex mutex;
    std::condition_variable_any cv;
    bool ready = false;

    std::jthread worker([&](std::stop_token token) {
        std::unique_lock<std::mutex> lock(mutex);

        bool completed = cv.wait(lock, token, [&] {
            return ready;
        });

        if (!completed) {
            std::cout << "Operation cancelled\n";
            return;
        }

        std::cout << "Condition satisfied\n";
    });

    worker.request_stop();
}
```

In this example, the thread waits until the `ready` condition becomes true or a stop request is received. If the wait ends because of a stop request while the condition remains unsatisfied, `wait()` returns `false`.

### Comparison Table

| Feature                         | `condition_variable`         | `condition_variable_any`         |
| ------------------------------- | ---------------------------- | -------------------------------- |
| Lock type                       | Only `unique_lock<mutex>`    | Compatible Lock types            |
| Direct `stop_token` support     | No                           | Yes                              |
| Compatibility with custom Locks | No                           | Yes                              |
| Implementation overhead         | Usually lower                | May be higher                    |
| Common use case                 | Waiting with a regular Mutex | Flexible and cancellable waiting |

Therefore, if you need cancellable waiting with `std::stop_token`, `std::condition_variable_any` is the natural choice. If you do not need this functionality and use only `std::mutex`, `std::condition_variable` is generally the simpler and less expensive option.

## Design Patterns for Cancellation

Using `std::stop_token` correctly involves more than simply calling `request_stop()`. API design and thread architecture should be compatible with the concept of Cooperative Cancellation from the beginning.

### Designing Cancellable Functions

Functions that perform long-running or stoppable operations should ideally accept a `std::stop_token` as one of their parameters.

For example:

```
#include <stop_token>
#include <chrono>
#include <thread>

void process_data(std::stop_token token) {
    while (!token.stop_requested()) {
        perform_one_step();

        std::this_thread::sleep_for(
            std::chrono::milliseconds(100)
        );
    }
}
```

With this pattern, the operation is divided into smaller steps, and the function can check for a stop request after each step.

This approach offers several advantages:

- The function does not depend on a particular thread-management mechanism.
- The same function can be used by different threads.
- The operation can stop gradually and in a controlled manner.
- The function can release the resources it owns before returning.

In larger projects, it is better to check the Stop Token at specific, predictable points, such as after completing each processing stage or before starting an expensive operation.

### Supporting an Optional Stop Token

Sometimes a function needs to be usable both with and without cancellation support.

In such cases, `std::stop_token` can be given a default value:

```
#include <stop_token>

void process_data(
    std::stop_token token = {}
) {
    while (!token.stop_requested()) {
        perform_one_step();
    }
}
```

The default value `{}` creates a Token without a Stop State. As a result, `stop_requested()` always returns `false` for that Token.

This design allows the function to be called without passing a Token:

```
process_data();
```

A real Token can also be passed:

```
std::stop_source source;

process_data(source.get_token());

source.request_stop();
```

However, because the function call in this example is synchronous, the stop request must come from another thread, or the application must be structured so that a stop request can be issued while the function is running.

### Using Callbacks to Release Resources

Sometimes a thread is waiting inside an external API or a blocking operation and cannot continuously check `stop_requested()`.

In such situations, `std::stop_callback` can be used to notify the blocking mechanism.

```
#include <stop_token>
#include <mutex>
#include <condition_variable>

void worker_function(std::stop_token token) {
    std::mutex mutex;
    std::condition_variable_any cv;
    bool ready = false;

    std::stop_callback callback(token, [&] {
        cv.notify_all();
    });

    std::unique_lock<std::mutex> lock(mutex);

    cv.wait(lock, [&] {
        return ready || token.stop_requested();
    });
}
```

In this example, the callback executes when a stop request is issued and wakes the waiting thread.

However, this example could also use the Stop Token-aware overload of `condition_variable_any`, which is generally the simpler choice. A callback is more useful when another API or multiple waiting mechanisms need to be notified at the same time.

## Implementing a Cancellable Thread Pool

One important application of `std::stop_token` is designing a Thread Pool.

In a typical Thread Pool, several Workers retrieve Tasks from a shared Queue. When the Pool shuts down, the Workers must be able to leave their waiting state, and running operations must have an opportunity to terminate in a controlled manner.

The following example shows a simple implementation of this pattern:

```
#include <condition_variable>
#include <functional>
#include <mutex>
#include <queue>
#include <thread>
#include <vector>
#include <stop_token>

class ThreadPool {
public:
    explicit ThreadPool(std::size_t count) {
        for (std::size_t i = 0; i < count; ++i) {
            workers_.emplace_back(
                [this](std::stop_token token) {
                    worker_loop(token);
                }
            );
        }
    }

    void submit(std::function<void()> task) {
        {
            std::lock_guard<std::mutex> lock(mutex_);
            tasks_.push(std::move(task));
        }

        cv_.notify_one();
    }

    ~ThreadPool() {
        for (auto& worker : workers_) {
            worker.request_stop();
        }

        cv_.notify_all();
    }

private:
    void worker_loop(std::stop_token token) {
        while (true) {
            std::function<void()> task;

            {
                std::unique_lock<std::mutex> lock(mutex_);

                cv_.wait(lock, token, [&] {
                    return !tasks_.empty();
                });

                if (token.stop_requested()) {
                    return;
                }

                if (tasks_.empty()) {
                    continue;
                }

                task = std::move(tasks_.front());
                tasks_.pop();
            }

            task();
        }
    }

    std::mutex mutex_;
    std::condition_variable_any cv_;
    std::queue<std::function<void()>> tasks_;
    std::vector<std::jthread> workers_;
};
```

This implementation is intended as an educational example and has several important characteristics:

- Workers are created using `std::jthread`, and each receives its own Token.
- The Task Queue is protected by a Mutex.
- `condition_variable_any` allows a Worker to wake up in response to a Stop Request.
- When the Thread Pool is destroyed, stop requests are issued to its Workers.
- After the destructor finishes, the `workers_` members are destroyed, and the `std::jthread` destructors perform the Join operations.

In this example, a Worker exits its loop when it receives a stop request. Therefore, Tasks that are still in the Queue may never be executed.

Also, if a Task runs for a long time and does not support cancellation itself, requesting a stop for the Worker cannot stop that Task. To support cancellation of running Tasks, the Tasks themselves must also receive a `std::stop_token`.

### Immediate Shutdown vs. Graceful Shutdown

When designing a Thread Pool, you must decide what should happen to the remaining Tasks during Shutdown.

Two common policies are:

- Immediate Shutdown: Workers exit their loops as soon as possible, and Tasks that have not started may be discarded.
- Graceful Shutdown: New Tasks are no longer accepted first. Workers then execute all remaining Tasks before finally terminating.

These behaviors are not equivalent. You should not assume that simply using `request_stop()` guarantees that all Tasks will be executed.

A graceful Shutdown usually requires a separate state, such as `accepting_tasks_` or `shutdown_`, and Worker exit logic designed around the Queue's state and the chosen Shutdown policy.

## Comparing Stop Tokens with Atomic Flags

Before `std::stop_token` was introduced, one common way to stop a thread was to use `std::atomic<bool>`.

For example:

```
#include <atomic>
#include <thread>

std::atomic<bool> stop_requested = false;

void worker() {
    while (!stop_requested.load()) {
        perform_one_step();
    }
}

int main() {
    std::thread thread(worker);

    stop_requested.store(true);

    thread.join();
}
```

This approach is simple and remains useful in many scenarios. However, `std::stop_token` provides additional capabilities.

| Feature                                           | `atomic<bool>`               | `stop_token`            |
| ------------------------------------------------- | ---------------------------- | ----------------------- |
| Issuing a stop request                            | By changing the Atomic value | With `request_stop()`   |
| Observing the stop state                          | With `load()`                | With `stop_requested()` |
| Callback support                                  | No                           | Yes                     |
| Sharing state among multiple Tokens               | Requires manual design       | Built in                |
| Integration with `jthread`                        | Manual                       | Direct                  |
| Cancellable waiting with `condition_variable_any` | No                           | Yes                     |
| Stop State management                             | Manual                       | Managed by the library  |

### When Is an Atomic Flag More Appropriate?

If you only need a simple flag to coordinate two threads, using `std::atomic<bool>` may be sufficient.

However, if you need multiple consumers, callbacks, integration with `std::jthread`, or cancellable waiting, `std::stop_token` provides a more complete design.

It is important to understand that Stop Tokens are not a replacement for every use of Atomic types. These tools serve different purposes.

## Memory Model and Thread Safety

One important characteristic of `std::stop_token` is that it is designed for concurrent use by multiple threads.

Multiple threads can share the same Token and observe the stop state.

For example:

```
#include <stop_token>
#include <thread>
#include <vector>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        perform_one_step();
    }
}

int main() {
    std::stop_source source;
    std::vector<std::jthread> workers;

    auto token = source.get_token();

    for (int i = 0; i < 4; ++i) {
        workers.emplace_back([token] {
            worker(token);
        });
    }

    source.request_stop();
}
```

In this example, four Workers receive the same Stop Token. When `request_stop()` is called, all of them can observe the stop request.

### Concurrency Guarantees

The Stop Token mechanism is designed for concurrent use by multiple threads. However, this does not mean that all data accessed by callbacks or Workers is automatically thread-safe.

For example, if multiple callbacks access an ordinary variable concurrently and at least one of them modifies it, a Data Race may occur.

Therefore:

- Use a Mutex or Atomic type for shared data.
- Do not assume that executing a callback automatically protects all application data.
- Do not rely on callback execution order.
- Use appropriate Synchronization Primitives to coordinate threads.

In C++, stop requests and callback execution have concurrency guarantees defined by the standard. Nevertheless, Stop Tokens are not a replacement for Mutexes, Atomic types, or other synchronization tools.

## Limitations and Important Considerations

Despite the useful features of `std::stop_token`, it has limitations that must be considered when designing multithreaded systems.

### It Does Not Forcefully Terminate Threads

A stop request is only a request. If a thread does not check it, the thread may continue executing.

For example, the following loop is not cancellable:

```
void worker(std::stop_token token) {
    while (true) {
        perform_one_step();
    }
}
```

Even if a Stop Request is issued, the function never checks its state.

### Blocking Operations Are Not Automatically Cancelled

If a thread is inside a blocking function, issuing a stop request does not necessarily cause that function to return.

For example, a network call or file I/O operation may remain blocked until another event occurs.

To address this problem, you must use APIs that support cancellation or use callbacks and wake-up mechanisms to make it possible for the thread to leave its waiting state.

### Callbacks Can Cause Deadlocks

Callbacks associated with `std::stop_callback` execute synchronously in the thread that issues the stop request.

Therefore, if a callback attempts to acquire a Lock that the requesting thread already holds, a deadlock may occur.

Callbacks should also be short and inexpensive, and unnecessary blocking operations should be avoided.

### A Stop Request Is Not Equivalent to Thread Completion

Issuing a stop request does not mean that the thread has finished.

To ensure that a thread has completed, use `join()` or rely on the managed lifetime behavior of `std::jthread`.

### Stop Tokens Are Not Sufficient for Resource Management

A Stop Token alone does not guarantee that resources such as files, Sockets, Mutexes, or memory will be released.

Resources must be released through appropriate design and RAII.

## Common Mistakes

There are several common mistakes when using Stop Tokens that should be avoided.

### Assuming That a Thread Stops Immediately

Calling `request_stop()` only registers a stop request. The thread must check it at appropriate points or be waiting in a cancellable operation.

### Forgetting to Join

In applications that use `std::thread`, a stop request is not a substitute for `join()`.

### Performing Expensive Operations in Callbacks

Since a callback executes synchronously in the thread that issues the stop request, performing expensive or blocking operations inside it can cause delays and concurrency problems.

### Using References with Invalid Lifetimes

If a callback accesses a variable through a Reference, that variable must remain valid for the entire lifetime of the callback.

### Assuming a Callback Execution Order

The C++ standard does not guarantee the execution order of multiple callbacks registered with the same Stop State. Therefore, you should not rely on that order to perform operations in stages.

### Assuming Tasks Are Automatically Cancelled

Stop Tokens do not automatically stop Tasks in a Queue or operations that are already running. Each Task must have its own appropriate cancellation mechanism.

## Best Practices

The following principles are recommended for safer use of `std::stop_token`:

- Divide long-running functions into smaller steps.
- Check `stop_requested()` at appropriate points.
- Use the `condition_variable_any` overloads for cancellable waiting.
- Keep callbacks short and fast, and avoid unnecessary blocking operations inside them.
- Prefer `std::jthread` for new threads unless there is a specific reason to use `std::thread`.
- Decide on an immediate or graceful Shutdown policy when designing a Thread Pool.
- Pass Stop Tokens to long-running Tasks when those Tasks need cancellation support.
- Manage resources with RAII.
- Use appropriate Mutexes or Atomic types for shared data.
- Do not depend on callback execution order.
- Distinguish between a stop request, operation completion, and Join.

## Complete Example: A Cancellable Worker with a Task Queue

The following example implements a Worker that retrieves Tasks from a shared Queue and can be stopped using `std::stop_token` and `std::condition_variable_any`.

```
#include <condition_variable>
#include <functional>
#include <mutex>
#include <queue>
#include <stop_token>
#include <thread>
#include <iostream>

class Worker {
public:
    void submit(std::function<void()> task) {
        {
            std::lock_guard<std::mutex> lock(mutex_);
            tasks_.push(std::move(task));
        }

        cv_.notify_one();
    }

    void run(std::stop_token token) {
        while (true) {
            std::function<void()> task;

            {
                std::unique_lock<std::mutex> lock(mutex_);

                cv_.wait(lock, token, [&] {
                    return !tasks_.empty();
                });

                if (token.stop_requested()) {
                    return;
                }

                if (tasks_.empty()) {
                    continue;
                }

                task = std::move(tasks_.front());
                tasks_.pop();
            }

            task();
        }
    }

private:
    std::mutex mutex_;
    std::condition_variable_any cv_;
    std::queue<std::function<void()>> tasks_;
};

int main() {
    Worker worker;

    worker.submit([] {
        std::cout << "Task 1\n";
    });

    worker.submit([] {
        std::cout << "Task 2\n";
    });

    std::jthread thread([&](std::stop_token token) {
        worker.run(token);
    });

    thread.request_stop();
}
```

In this example:

- The `Worker` class maintains a shared Queue for storing Tasks.
- The `submit()` function adds a Task to the Queue.
- The `run()` function waits until a Task arrives or a stop request is received.
- `condition_variable_any` makes it possible to cancel the wait using a Stop Token.
- Calling `request_stop()` wakes the Worker from its waiting operation.
- When `std::jthread` is destroyed, the Join operation is performed.

This is an educational implementation. In a real system, you must decide what should happen to the remaining Tasks when the Worker stops and whether currently executing Tasks must also support cancellation.

Also, because `thread` is created after `worker` in this example, `thread` is destroyed and joined first when `main()` exits. The `worker` is destroyed afterward. As a result, the Worker remains valid throughout the thread's execution.

## Compiler and Standard Compatibility

The `std::stop_token` features were introduced in C++20.

To use them, the compiler and standard library in use must provide the required C++20 support.

The main headers are:

| Header                 | Purpose                                                   |
| ---------------------- | --------------------------------------------------------- |
| `<stop_token>`         | Defines `stop_source`, `stop_token`, and `stop_callback`  |
| `<thread>`             | Defines `std::jthread`                                    |
| `<condition_variable>` | Defines `condition_variable` and `condition_variable_any` |
| `<mutex>`              | Defines Mutex and Lock types                              |
| `<chrono>`             | Provides timing and Timeout utilities                     |

You can use a Feature Test Macro to check for standard library support:

```
#include <version>

#if defined(__cpp_lib_jthread) && __cpp_lib_jthread >= 201911L
    // C++20 jthread and stop token support is available.
#else
    #error "Required C++20 thread support is unavailable"
#endif
```

With GCC, you can typically use the following command-line option:

```
g++ -std=c++20 main.cpp -pthread
```

With Clang:

```
clang++ -std=c++20 main.cpp -pthread
```

With MSVC:

```
cl /std:c++20 /EHsc main.cpp
```

Actual support depends on the compiler version and the standard library version. Therefore, enabling C++20 mode alone does not guarantee that all library features are available.

## Conclusion

`std::stop_token` is an important C++20 feature for designing multithreaded systems that support Cooperative Cancellation.

It separates the act of requesting a stop from the actual execution of a thread. With `std::stop_source`, `std::stop_callback`, and `std::jthread`, thread lifetimes can be managed more effectively.

The most important points to remember are:

- `std::stop_source` is used to issue stop requests.
- `std::stop_token` makes it possible to observe the stop state.
- `std::stop_callback` is used to respond to stop requests.
- `std::jthread` simplifies thread management and automatically requests a stop when destroyed.
- `std::condition_variable_any` supports cancellable waiting with Stop Tokens.
- A stop request does not forcibly terminate a thread.
- To ensure that a thread has finished, use Join and manage its lifetime correctly.

Ultimately, the most important point is that cancellation should be part of the system's design, not a feature added at the end of implementation. When APIs, Tasks, and waiting mechanisms are designed around Cooperative Cancellation from the beginning, it becomes easier to build multithreaded applications that are predictable, safe, and maintainable.


---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>