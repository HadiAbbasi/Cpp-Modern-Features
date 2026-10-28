<div align="center">

[🇺🇸 English](./std_jthread.md) | [🇮🇷 فارسی](../../fa/cpp20/std_jthread.md)

</div>

---

# A Comprehensive Guide to `std::jthread` in C++20

`std::jthread` is one of the most important concurrency features introduced in C++20. It provides automatic and safer thread management through RAII and built-in support for cooperative cancellation using `std::stop_token`.

In practice, `std::jthread` addresses two major problems:

- Automatic thread cleanup through RAII.
- Cooperative cancellation through `stop_token`.

It is not merely a replacement for `std::thread`; it is an improved alternative that eliminates many common threading errors and makes concurrent code easier to maintain.

## Introduction

Since `std::thread` was introduced in C++11, developers have been able to use threads through the standard library. However, `std::thread` places an important responsibility on the programmer:

Before a thread object is destroyed, either `join()` or `detach()` must be called.

Otherwise, the program terminates by calling `std::terminate`.

The following example demonstrates this problem:

```
#include <thread>

int main() {
    std::thread t([]{});

    // Destructor calls std::terminate
}
```

This behavior means that certain bugs become apparent only at runtime.

To address this issue, C++20 introduced `std::jthread`. When a `std::jthread` object is destroyed, it automatically requests cancellation and then joins the thread.

## Understanding RAII for Thread Management

The design of `std::jthread` is based on RAII (Resource Acquisition Is Initialization).

RAII means that:

- A resource is acquired when an object is constructed.
- The resource is released when the object is destroyed.

Just as `std::lock_guard` releases a lock automatically, `std::jthread` manages thread cleanup automatically.

The following table compares their behavior:

| Feature                                     | `std::thread` | `std::jthread`                                    |
| ------------------------------------------- | ------------- | ------------------------------------------------- |
| Manual `join()` required for cleanup        | Yes           | No                                                |
| Risk of `std::terminate` during destruction | Yes           | No, provided the destructor can complete normally |
| RAII-based thread cleanup                   | No            | Yes                                               |
| Built-in cooperative cancellation           | No            | Yes                                               |

## Introducing `std::jthread`

To use `std::jthread`, include the following header:

```
#include <thread>
```

Here is the simplest example:

```
#include <iostream>
#include <thread>

int main() {
    std::jthread worker([]{
        std::cout << "Worker running\n";
    });
}
```

When the scope ends:

- The destructor requests cancellation.
- The destructor calls `join()` if the thread is still joinable.
- The program waits for the thread to finish.

## Creating a `std::jthread`

### Using a Lambda

```
std::jthread worker([]{
    // work
});
```

### Using a Regular Function

```
void task() {
}

std::jthread worker(task);
```

### Passing Arguments

```
void print(int value) {
}

std::jthread worker(print, 42);
```

## Understanding Destructor Behavior

The most important difference between `std::jthread` and `std::thread` is the behavior of their destructors.

Conceptually, the `std::jthread` destructor behaves like this:

```
if (joinable()) {
    request_stop();
    join();
}
```

This means that if the thread is still joinable:

- A stop request is issued.
- The destructor waits for the thread to finish by calling `join()`.

This behavior makes thread cleanup safer, but it does not guarantee that the thread will finish promptly. The worker must cooperate with the stop request, and the destructor can block indefinitely if the thread never finishes.

## Understanding Cooperative Cancellation

One of the most important capabilities of `std::jthread` is its cooperative cancellation mechanism.

A thread is not forcibly terminated when a stop request is issued. Instead:

- A stop request is sent.
- The thread decides when and how to respond to the request.

This design avoids many of the dangerous problems associated with forcibly terminating a thread while it is accessing shared resources or holding locks.

## Introducing `std::stop_token`

When a thread function accepts `std::stop_token` as its first parameter, `std::jthread` automatically supplies a token to it.

For example:

```
#include <chrono>
#include <iostream>
#include <thread>

using namespace std::chrono_literals;

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Working...\n";
        std::this_thread::sleep_for(300ms);
    }

    std::cout << "Stopping\n";
}

int main() {
    std::jthread t(worker);

    std::this_thread::sleep_for(1s);
}
```

When `main()` exits:

- A stop request is issued.
- The loop terminates after the worker observes the request.
- The thread finishes, allowing the destructor to complete its `join()`.

The token is a lightweight handle to shared stop state. Copies of a token associated with the same stop state observe the same stop request.

## Requesting Cancellation Manually

In addition to automatic cancellation during destruction, you can explicitly request a stop.

```
t.request_stop();
```

A complete example:

```
#include <chrono>
#include <iostream>
#include <thread>

using namespace std::chrono_literals;

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Running\n";
        std::this_thread::sleep_for(200ms);
    }
}

int main() {
    std::jthread t(worker);

    std::this_thread::sleep_for(1s);

    t.request_stop();
}
```

The call to `request_stop()` does not wait for the thread to finish. It requests cancellation; the thread must respond to that request itself.

The request is persistent: once the stop state has been requested to stop, subsequent requests cannot reset it to the non-stopped state.

## Checking the Stop State

The `stop_requested()` member function checks whether a stop request has been made.

For example:

```
while (!token.stop_requested()) {
    // work
}
```

This check is useful in loops that perform recurring work and need to stop gracefully.

## Understanding `stop_requested()` and `stop_possible()`

The `std::stop_token` class provides two important member functions.

| Function           | Purpose                                                                                   |
| ------------------ | ----------------------------------------------------------------------------------------- |
| `stop_requested()` | Checks whether a stop request has been issued.                                            |
| `stop_possible()`  | Checks whether the token is associated with a stop state that can receive a stop request. |

For example:

```
if (token.stop_possible()) {
}
```

For a token obtained from a normal `std::jthread`, `stop_possible()` is generally `true`, because the thread has an associated stop state. A default-constructed token, by contrast, has no associated stop state.

## Introducing `std::stop_source`

The `std::stop_source` class represents the source of stop requests.

For example:

```
std::stop_source source;
auto token = source.get_token();

source.request_stop();
```

The relationship between these components is:

```
stop_source
     |
     v
 stop_token
     |
     v
worker thread
```

With `std::jthread`, this relationship is managed automatically. The thread object provides access to its associated stop state through `get_stop_source()` and `get_stop_token()`.

A separate `std::stop_source` is useful when the component requesting cancellation needs to be independent of the thread object.

## Introducing `std::stop_callback`

Sometimes a thread needs to perform an action when a stop request arrives. `std::stop_callback` provides a way to register a callback for that event.

For example:

```
#include <iostream>
#include <stop_token>

void worker(std::stop_token token) {
    std::stop_callback callback(token, [] {
        std::cout << "Stop callback\n";
    });

    while (!token.stop_requested()) {
    }
}
```

Important characteristics include:

- The callback is invoked when a stop request is successfully initiated.
- If a callback is registered after the stop state has already been requested to stop, it is invoked during registration.
- Callback execution is synchronous with the operation that triggers it, or with registration if the request already exists.
- Callbacks can be useful for notifying waiting threads or triggering other cancellation-related actions.

Because callbacks can execute on the thread that requests cancellation, they should be short, thread-safe, and designed to avoid deadlocks. A callback should not assume that it runs on the worker thread.

## Understanding `join()` and `detach()`

Both `join()` and `detach()` remain available in `std::jthread`.

### Using `join()`

```
worker.join();
```

This waits for the associated thread to finish. After a successful `join()`, the thread object is no longer joinable.

Calling `join()` manually does not automatically request cancellation. If the worker needs a stop request to finish, request it explicitly before joining.

### Using `detach()`

```
worker.detach();
```

After detaching:

- The thread continues independently of the thread object.
- The object no longer manages the lifetime of the running thread.
- The destructor will not join the detached thread.

Detaching also means that the destructor cannot use the thread object's stop state to request cancellation. If cancellation is required, the application needs another way to manage the relevant stop state and coordinate the worker's lifetime.

Detached threads require particular care because they may outlive objects or resources they access.

## Transferring Ownership

Like `std::thread`, `std::jthread` is move-only.

For example:

```
std::jthread t1([]{});

std::jthread t2 = std::move(t1);
```

Copying is not allowed:

```
std::jthread t2 = t1; // Error
```

Moving transfers ownership of the associated thread and its management state. The source object becomes non-joinable after a successful move.

One important detail is that move assignment to an already joinable `std::jthread` first requests cancellation and joins its currently managed thread before taking ownership of the new one. Consequently, move assignment can block.

## Checking Thread State

Several member functions are useful when managing a thread:

| Function            | Purpose                                                           |
| ------------------- | ----------------------------------------------------------------- |
| `joinable()`        | Checks whether the object represents a thread that can be joined. |
| `get_id()`          | Returns the thread identifier.                                    |
| `join()`            | Waits for the associated thread to finish.                        |
| `detach()`          | Detaches the thread from the object.                              |
| `request_stop()`    | Requests cooperative cancellation.                                |
| `get_stop_token()`  | Returns a token associated with the thread's stop state.          |
| `get_stop_source()` | Returns a stop source associated with the thread's stop state.    |

For example:

```
if (worker.joinable()) {
}
```

A joinable thread is not necessarily executing at that exact moment. The thread may already have completed but still require `join()` or `detach()` to release its association with the thread object.

## Using a Lambda with `std::stop_token`

A common pattern is to pass a lambda that accepts `std::stop_token`:

```
std::jthread worker([](std::stop_token token) {
    while (!token.stop_requested()) {
    }
});
```

For automatic token injection, the callable must be invocable with the stop token followed by any arguments supplied to the `std::jthread` constructor. The token is passed as the first argument, before the explicitly supplied arguments.

If the callable does not accept a stop token, `std::jthread` can invoke it without one, just as a regular thread function would be invoked.

## Using `std::jthread` in Classes

One of the most useful applications of `std::jthread` is managing a worker thread as a class member.

For example:

```
#include <chrono>
#include <thread>

using namespace std::chrono_literals;

class BackgroundWorker {
    std::jthread thread_;

public:
    BackgroundWorker()
        : thread_([](std::stop_token token) {
            while (!token.stop_requested()) {
                std::this_thread::sleep_for(100ms);
            }
        })
    {
    }
};
```

The primary advantage is that the class does not need a custom destructor merely to call `join()`.

However, automatic joining does not eliminate every lifetime concern. Class members are destroyed in reverse order of declaration, and the destructor body executes before member destruction begins. If a worker accesses other members of the class, those members must remain valid until the worker has finished.

In particular, a worker that accesses members declared after `thread_` can be problematic because those members are destroyed before `thread_` is destroyed and joined.

When a worker accesses the owning object, an explicit shutdown strategy or careful member ordering may be necessary.

## Comparing `std::jthread` with `std::thread`

### Key Differences

| Feature                                     | `std::thread`         | `std::jthread`                                         |
| ------------------------------------------- | --------------------- | ------------------------------------------------------ |
| RAII-based thread cleanup                   | No                    | Yes                                                    |
| Automatic `join()` during destruction       | No                    | Yes, if joinable                                       |
| Built-in `stop_token` support               | No                    | Yes                                                    |
| Cooperative cancellation support            | No built-in mechanism | Yes                                                    |
| Risk of `std::terminate` during destruction | Yes, if joinable      | Not from an unhandled joinable-thread destructor alone |

### Comparing the Syntax

With `std::thread`:

```
std::thread t(worker);
t.join();
```

With `std::jthread`:

```
std::jthread t(worker);
```

The second version is shorter and provides automatic cleanup. However, the difference is more than syntactic: `std::jthread` also provides a standardized cancellation mechanism.

## Comparing `std::jthread` with Older Cancellation Techniques

Before C++20, a common approach was to use a shared `std::atomic<bool>` variable to communicate a stop request.

For example:

```
#include <atomic>

std::atomic<bool> stop = false;

while (!stop.load()) {
}
```

This approach can work, but the application must manage the shared state, communicate it to the worker, and ensure that its lifetime and access are correct.

With C++20, a `std::stop_token` provides a standardized alternative:

```
while (!token.stop_requested()) {
}
```

The stop-token mechanism integrates with `std::jthread`, `std::stop_source`, `std::stop_callback`, and cancellation-aware waiting facilities. It does not eliminate the need to design synchronization correctly, but it provides a consistent cancellation interface.

## Integrating with `condition_variable`

A common challenge is stopping a thread that is blocked while waiting on a condition variable.

### Using `std::condition_variable` with `std::stop_callback`

A stop callback can notify a condition variable when cancellation is requested.

For example:

```
#include <condition_variable>
#include <mutex>
#include <stop_token>

void worker(std::stop_token token) {
    std::mutex mutex;
    std::condition_variable cv;

    std::unique_lock lock(mutex);

    std::stop_callback callback(token, [&] {
        cv.notify_all();
    });

    while (!token.stop_requested()) {
        cv.wait(lock);
    }
}
```

Without a notification mechanism, a thread waiting indefinitely on a condition variable may never wake up to observe a stop request.

This example illustrates the basic approach, but a production implementation should also use a predicate representing its actual work or shutdown conditions. A plain `wait()` can wake spuriously, and notifications can occur before a thread begins waiting. A correct design must ensure that its shared state and waiting protocol prevent missed work or indefinite waiting.

### Using `std::condition_variable_any`

C++20 provides stop-token-aware waiting through `std::condition_variable_any`.

For example:

```
#include <condition_variable>
#include <mutex>
#include <stop_token>

void worker(std::stop_token token) {
    std::mutex mutex;
    std::condition_variable_any cv;

    std::unique_lock lock(mutex);

    cv.wait(lock, token, [] {
        return false;
    });
}
```

The overload accepts a stop token and a predicate. It waits until the predicate becomes true or a stop request is observed. The function returns the predicate's final result.

In this example, the predicate is always false, so the wait can finish only because of a stop request. The example demonstrates the API; a practical worker would normally use a predicate that checks for available work or another meaningful condition.

This stop-aware overload is particularly useful for worker threads that need to wait efficiently without repeatedly polling a flag.

## Handling Sleep Operations

The standard `std::this_thread::sleep_for()` function is not directly cancellable through `std::stop_token`.

Consider the following example:

```
while (!token.stop_requested()) {
    std::this_thread::sleep_for(std::chrono::seconds(10));
}
```

If cancellation is requested immediately after the thread begins sleeping, the worker may not observe it until the sleep finishes.

Possible approaches include:

- Using shorter sleep intervals.
- Replacing the sleep with a condition-variable-based wait.
- Using `std::condition_variable_any` when stop-aware waiting is appropriate.

For periodic tasks, the right choice depends on the required timing precision and cancellation responsiveness.

## Managing Exceptions

Exception handling remains important when using `std::jthread`.

If an exception escapes the top-level thread function, `std::terminate` is called. The thread object's automatic joining behavior does not catch or propagate exceptions from the worker.

A worker can handle exceptions internally:

```
void worker(std::stop_token token) {
    try {
        while (!token.stop_requested()) {
            // Perform work
        }
    }
    catch (...) {
        // Handle exception
    }
}
```

In production code, catching exceptions should be accompanied by an appropriate error-reporting or recovery strategy. If the main thread needs to observe worker failures, the application must explicitly communicate them, for example through an exception pointer, a result object, or a synchronized error channel.

## Important Edge Cases

### Stop Requested Before the Worker Begins Its Work

A stop request may arrive before the worker starts processing its main task.

The worker should check the token before beginning work:

```
if (token.stop_requested()) {
    return;
}
```

This is particularly important when the thread performs initialization or acquires resources before entering its main loop.

### Cancellation During a Long-Running Operation

If the worker is executing a long operation that cannot be interrupted, `stop_token` alone cannot terminate that operation.

Examples include:

- Blocking I/O operations that do not support cancellation.
- System calls that cannot be interrupted through the application's chosen mechanism.
- Long computations that do not check the stop state.

In such cases, use APIs that support cancellation or interruption where available, or divide the computation into smaller units that periodically check the token.

### Using a Thread After `detach()`

After a thread is detached:

- The thread object no longer manages the running thread.
- Its destructor no longer joins that thread.
- The thread object's automatic stop-request behavior no longer applies to the detached thread.

If cancellation is needed after detaching, the application must retain and manage an appropriate cancellation mechanism independently. It must also ensure that all resources accessed by the detached worker remain valid until the worker finishes.

## Common Mistakes

### Forgetting to Check `stop_requested()`

A worker that ignores cancellation can prevent its owning `std::jthread` from being destroyed promptly.

Incorrect:

```
while (true) {
}
```

Correct:

```
while (!token.stop_requested()) {
}
```

The corrected loop is responsive to cancellation, although a busy loop like this is usually inefficient and should be replaced with meaningful work or an appropriate waiting mechanism.

### Using Long Sleep Intervals

An unnecessarily long sleep can delay shutdown:

```
std::this_thread::sleep_for(std::chrono::minutes(1));
```

A shorter interval may improve responsiveness:

```
std::this_thread::sleep_for(std::chrono::milliseconds(100));
```

However, shorter intervals increase wake-ups and do not provide immediate cancellation. For interruptible waiting, a condition-variable-based design is often preferable.

### Allowing Exceptions to Escape the Worker

An uncaught exception escaping the thread function causes `std::terminate`. Handle exceptions inside the worker or communicate failures through an explicit error-reporting mechanism.

### Using `detach()` Without a Clear Lifetime Strategy

Detaching removes the automatic lifetime coordination provided by joining. Unless there is a clear reason to detach, retaining ownership through `std::jthread` and allowing it to join is generally easier to reason about.

## Design Considerations

When designing concurrent systems with `std::jthread`, consider the following practices:

- Accept `std::stop_token` in worker functions that need cooperative cancellation.
- Check the stop state at appropriate points in the work loop.
- Divide long computations into smaller units when cancellation responsiveness matters.
- Use RAII to manage resources acquired by the worker.
- Use `std::condition_variable_any` for stop-aware waiting when appropriate.
- Use `std::stop_callback` to notify or wake operations that otherwise cannot observe a stop request.
- Ensure that a worker cannot outlive the objects and resources it accesses.
- Avoid holding locks or performing long blocking operations in cancellation callbacks.
- Design shutdown so that the worker can finish without depending on actions that cannot occur until after the destructor returns.

These practices help make shutdown behavior predictable and reduce the risk of deadlocks and lifetime errors.

## Implementing a Cancellable Worker

The following example demonstrates a practical worker managed by `std::jthread`:

```
#include <chrono>
#include <iostream>
#include <thread>

using namespace std::chrono_literals;

class Worker {
    std::jthread thread_;

public:
    Worker()
        : thread_([](std::stop_token token) {
            while (!token.stop_requested()) {
                std::cout << "Processing\n";
                std::this_thread::sleep_for(200ms);
            }

            std::cout << "Worker stopped\n";
        })
    {
    }
};

int main() {
    Worker worker;

    std::this_thread::sleep_for(1s);
}
```

When `main()` exits:

- The `Worker` object is destroyed.
- Its `std::jthread` member requests cancellation.
- The loop eventually observes the request and terminates.
- The thread is joined automatically.

Because the worker uses `sleep_for()`, shutdown may be delayed until the current sleep finishes. In this example, that delay is bounded by the sleep interval, assuming the worker continues executing normally.

## Performance and Overhead

In terms of performance, `std::jthread` is generally comparable to `std::thread` for the underlying thread execution.

Additional overhead can arise from:

- Maintaining the shared stop state.
- Managing `std::stop_source` and `std::stop_token` objects.
- Registering and invoking stop callbacks.

The actual cost depends on the implementation and usage pattern. For most applications, these costs are small compared with the benefits of safer thread management. Performance-sensitive systems should still benchmark their own workloads rather than assume the overhead is always negligible.

## When to Use `std::jthread`

`std::jthread` is particularly suitable for:

- Background services.
- Worker threads.
- Periodic processing tasks.
- Classes that own a background thread.
- Applications that benefit from RAII-based thread cleanup.
- Systems that require controlled shutdown and cooperative cancellation.

Other approaches may be more appropriate when:

- A detached-thread design is an explicit requirement.
- An external API owns the thread's lifetime and imposes different management requirements.
- The operation cannot respond to cooperative cancellation and requires a different interruption mechanism.
- The application needs a task pool, executor, or higher-level concurrency abstraction rather than individually managed threads.

## Summary

`std::jthread` is an important addition to concurrency in C++20. By combining automatic thread management with standardized cooperative cancellation, it addresses several common problems associated with `std::thread`.

Its main advantages include:

- Automatic joining through RAII.
- Avoiding `std::terminate` caused solely by destroying a still-joinable thread object.
- Standardized support for `std::stop_token`.
- Cooperative cancellation without forcibly terminating a running thread.
- Integration with `std::condition_variable_any` and `std::stop_callback`.
- Simpler lifetime management for classes that own worker threads.

For modern C++20 and later applications, `std::jthread` is often a better choice than `std::thread` when the application needs an owned thread with predictable cleanup and cooperative shutdown. However, safe destruction still depends on the worker being able to finish, shared resources remaining valid, and synchronization being designed correctly.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>