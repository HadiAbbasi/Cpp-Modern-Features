<div align="center">

[🇺🇸 English](./std_stop_source.md) | [🇮🇷 فارسی](../../fa/cpp20/std_stop_source.md)

</div>

---

# **Comprehensive Introduction to&#x20;**`std::stop_source`**&#x20;in C++20**

## **Introduction**

Managing the termination of concurrent operations is one of the major challenges in concurrent and multithreaded programming. In many applications, one thread performs a long-running task while another thread needs to be able to request that the operation stop under specific conditions.

For example, an application might have:

- A thread downloading a large file while the user cancels the download.
- A thread processing data while the application needs to request that it stop before shutdown.
- Several threads performing a shared operation concurrently, all of which need to be notified when a stop is requested.
- A running task that must be stopped in a controlled manner when its time limit expires.

In older versions of C++, the standard library did not provide a general-purpose, unified mechanism for sending cooperative stop requests to threads. Developers typically had to implement this functionality themselves using shared variables, `std::atomic`, `std::mutex`, and `std::condition_variable`.

C++20 addressed much of this problem by introducing Cooperative Cancellation.

This mechanism consists of three primary types:

- `std::stop_source`
- `std::stop_token`
- `std::stop_callback`

The `std::stop_source` type is responsible for creating and sending stop requests. The `std::stop_token` type allows receiving code to check the status of a stop request, while `std::stop_callback` makes it possible to execute a callback when a stop request is registered.

Among these components, `std::stop_source` plays the central role in managing and propagating stop requests.

## **The Concept of Cooperative Cancellation**

### **What Is Cooperative Cancellation?**

Cooperative Cancellation means that one part of a program requests that an operation stop, but another part—the code performing the operation—is responsible for checking the request and deciding when and how to stop.

In this model, a stop request does not mean that a thread stops immediately.

For example, suppose a thread performs a long-running processing operation:

```
#include <chrono>
#include <iostream>
#include <thread>

void process() {
    for (int i = 0; i < 100; ++i) {
        std::cout << "Processing: " << i << '\n';

        std::this_thread::sleep_for(
            std::chrono::milliseconds(100)
        );
    }
}

int main() {
    std::jthread worker(process);
}
```

In this example, processing continues until the loop finishes.

If another thread wants to stop this operation, simply sending a stop request is not enough. The executing code must be able to receive and check that request.

In C++20, this communication is established through `std::stop_source` and `std::stop_token`.

### **Cooperative Cancellation vs. Forced Termination**

When designing multithreaded systems, it is important to distinguish between two concepts:

- **Cooperative Cancellation:** A thread checks its stop status at specific points and terminates in a controlled manner when it receives a request.
- **Forced Termination:** A thread's execution is stopped without the cooperation of its internal code.

The Standard Stop Token mechanism in C++20 uses the first model.

This design has several important advantages:

- It allows resources to be released correctly.
- Destructor execution and cleanup operations remain under the program's control.
- The likelihood of creating inconsistent states in shared data is reduced.
- Termination logic can be designed to suit the application's requirements.

An important point is that `std::stop_source` provides no mechanism for directly killing a thread.

## **Introducing&#x20;**`std::stop_source`

### **Definition and Primary Role**

`std::stop_source` is a C++20 Standard Library class that provides a way to create, manage, and send stop requests.

This class owns a shared Stop State. Different parts of a program can access the same Stop State through `std::stop_token` and check the status of a stop request.

The overall communication structure looks like this:

```
                 std::stop_source
                        |
                        |
                 request_stop()
                        |
                        v
                 Shared Stop State
                        |
             +----------+----------+
             |          |          |
             v          v          v
          Token 1    Token 2    Token 3
             |          |          |
             v          v          v
          Thread 1  Thread 2  Thread 3
```

In this structure, a single `std::stop_source` can propagate a stop request to multiple `std::stop_token` objects.

All tokens connected to the same shared Stop State observe the same stop-request status.

### **Required Header**

To use `std::stop_source`, include the following header:

```
#include <stop_token>
```

This header provides the types and functionality associated with Stop Tokens.

### **Namespace**

The `std::stop_source` type belongs to the `std` namespace.

```
#include <stop_token>

int main() {
    std::stop_source source;
}
```

### **Relationship with Other Stop Token Components**

The main components of this mechanism have different responsibilities:

| Type                 | Responsibility                                                  |
| -------------------- | --------------------------------------------------------------- |
| `std::stop_source`   | Creates and sends stop requests                                 |
| `std::stop_token`    | Observes and checks stop status                                 |
| `std::stop_callback` | Executes a callback when a stop is requested                    |
| `std::nostopstate_t` | Specifies construction of a Stop Source without a Stop State    |
| `std::nostopstate`   | Constant object used for construction without a Stop State      |
| `std::jthread`       | Manages a thread with built-in cooperative cancellation support |

Understanding this separation is important when designing concurrent systems correctly.

## **The Concept of Stop State**

### **Internal Structure of a Stop State**

Each `std::stop_source` normally refers to a shared Stop State.

A Stop State is an internal synchronization structure that stores information related to stop requests and registered callbacks.

Its most important information includes:

- Whether a stop request has been made.
- Whether the Stop State is valid.
- The tokens connected to it.
- The callbacks registered to receive stop requests.

Stop State is an internal concept in the design of the standard library. It should not be confused with a publicly accessible class.

### **Multiple Tokens Connected to One Stop State**

When multiple tokens are obtained from the same Stop Source, they all connect to the same shared Stop State.

Consequently, if a stop request is sent through any Source connected to that Stop State, all associated tokens observe the requested stop status.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token1 = source.get_token();
    std::stop_token token2 = source.get_token();

    source.request_stop();

    std::cout << token1.stop_requested() << '\n';
    std::cout << token2.stop_requested() << '\n';
}
```

Output:

```
1
1
```

In this example, both tokens are connected to the same Stop State, so both can observe the stop request.

### **Stop State Lifetime**

The lifetime of a Stop State is independent of the lifetime of any particular `std::stop_source` object.

The state remains alive as long as at least one connected Source or Token remains.

Therefore, if a `std::stop_source` object is destroyed while a `std::stop_token` is still connected to its Stop State, the token can continue to observe the stop-request status.

This behavior is important when designing systems in which ownership of stop requests is distributed across different parts of a program.

## **Exploring the Constructors of&#x20;**`std::stop_source`

The `std::stop_source` class provides several constructors, each designed for particular situations.

### **Default Constructor**

The default constructor creates a valid Stop Source with a new Stop State.

```
#include <stop_token>

int main() {
    std::stop_source source;
}
```

After the object is constructed, no stop request has been registered yet.

Therefore:

```
#include <cassert>
#include <stop_token>

int main() {
    std::stop_source source;

    assert(source.stop_possible());
    assert(!source.stop_requested());
}
```

In this example:

- `stop_possible()` indicates whether a stop request can be made.
- `stop_requested()` indicates whether a stop request has already been registered.

### **Constructor with&#x20;**`std::nostopstate`

Sometimes it is necessary to create a Stop Source that does not have a valid Stop State.

For this purpose, you can use `std::nostopstate`.

```
#include <cassert>
#include <stop_token>

int main() {
    std::stop_source source{std::nostopstate};

    assert(!source.stop_possible());
    assert(!source.stop_requested());
}
```

In this case:

- No Stop State is created.
- No usable Token can be obtained from this Source.
- A stop request cannot be made.

This feature is useful when an object needs to be created in an empty state and connected to a Stop State at a later stage.

However, in many designs, using the default constructor or a simple assignment is more straightforward and readable.

### **Copy Constructor**

The `std::stop_source` class supports Copy Construction.

When a Stop Source is copied, a new Stop State is not created. Instead, the new object connects to the same Stop State as the original object.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source1;

    std::stop_source source2 = source1;

    source2.request_stop();

    std::cout << source1.stop_requested() << '\n';
}
```

Output:

```
1
```

The important point is that copying a `std::stop_source` does not create an independent copy of the stop status.

Both objects use the same shared state.

### **Move Constructor**

This class also supports Move Construction.

```
#include <iostream>
#include <stop_token>
#include <utility>

int main() {
    std::stop_source source1;

    std::stop_source source2 = std::move(source1);

    std::cout << source2.stop_possible() << '\n';
    std::cout << source1.stop_possible() << '\n';
}
```

Output:

```
1
0
```

After Move Construction, the destination object takes over the previous connection to the Stop State, and the source object is left without a Stop State.

As a result, you should not assume that the source object remains a valid Stop Source after ownership has been transferred.

## **Exploring the Member Functions of&#x20;**`std::stop_source`

The `std::stop_source` class provides several important member functions for checking stop status and managing the Stop State.

### **The&#x20;**`get_token()`**&#x20;Function**

The `get_token()` function returns a `std::stop_token` associated with the current Stop State.

Its conceptual signature is:

```
std::stop_token get_token() const noexcept;
```

Example:

```
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token = source.get_token();
}
```

Important characteristics of this function include:

- The returned Token is connected to the shared Stop State.
- Calling it does not create a new stop request.
- It can be called on both valid and invalid Sources.
- If no valid Stop State exists, the returned Token will not support stop requests.

This function is one of the most important ways to transfer the ability to observe stop requests to other threads.

### **The&#x20;**`request_stop()`**&#x20;Function**

The `request_stop()` function registers a stop request for the shared Stop State.

Its signature is:

```
bool request_stop() noexcept;
```

Example:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    bool first = source.request_stop();
    bool second = source.request_stop();

    std::cout << first << '\n';
    std::cout << second << '\n';
}
```

Output:

```
1
0
```

The return value indicates whether this call registered the stop request for the first time.

Its behavior is as follows:

- If the Stop State is valid and no stop request has been registered, it returns `true`.
- If a stop request has already been registered, it returns `false`.
- If no Stop State exists, it returns `false`.

An important point is that `request_stop()` is idempotent with respect to the final state of the Stop State. Sending multiple stop requests does not create multiple independent stop states.

Once a stop request is registered, the Stop State transitions to the requested-stop state and does not return to its initial state.

### **Executing Callbacks When a Stop Is Requested**

One important behavior of `request_stop()` is that it executes callbacks registered with the same Stop State.

For example:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token = source.get_token();

    std::stop_callback callback(
        token,
        [] {
            std::cout << "Stop callback executed\n";
        }
    );

    source.request_stop();
}
```

Output:

```
Stop callback executed
```

An important point is that callbacks may execute in the same thread that calls `request_stop()`.

Therefore, callbacks should not perform expensive operations, blocking operations, or operations that could cause a deadlock.

Also, callbacks registered with the Stop State may be involved in races between multiple threads. However, the Stop State mechanism prevents a particular registered callback from being executed more than once.

### **The&#x20;**`stop_requested()`**&#x20;Function**

This function checks whether a stop request has been registered.

Its signature is:

```
bool stop_requested() const noexcept;
```

Example:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::cout << source.stop_requested() << '\n';

    source.request_stop();

    std::cout << source.stop_requested() << '\n';
}
```

Output:

```
0
1
```

This function behaves similarly for Sources and Tokens when observing stop status.

If a stop request has been registered, it returns `true`.

Otherwise, it returns `false`.

### **The&#x20;**`stop_possible()`**&#x20;Function**

The `stop_possible()` function determines whether the object is connected to a valid Stop State.

Its signature is:

```
bool stop_possible() const noexcept;
```

Example:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source1;
    std::stop_source source2{std::nostopstate};

    std::cout << source1.stop_possible() << '\n';
    std::cout << source2.stop_possible() << '\n';
}
```

Output:

```
1
0
```

This function is particularly useful for distinguishing valid Sources from Sources without a Stop State.

Its difference from `stop_requested()` is important:

| Function           | Meaning                                                      |
| ------------------ | ------------------------------------------------------------ |
| `stop_possible()`  | Is there a valid Stop State, and can a stop request be made? |
| `stop_requested()` | Has a stop request already been registered?                  |

`stop_possible()` may return `true` while `stop_requested()` returns `false`.

This is the normal state of an ordinary Stop Source before a stop request is made.

## **Exploring the Assignment Operators**

### **Copy Assignment Operator**

The `std::stop_source` class supports Copy Assignment.

In this case, the destination object connects to the same Stop State as the source object.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source1;
    std::stop_source source2;

    source2 = source1;

    source2.request_stop();

    std::cout << source1.stop_requested() << '\n';
}
```

Output:

```
1
```

After assignment, both Sources are connected to the same shared Stop State.

If the destination object was connected to another Stop State before assignment, its previous connection is released.

### **Move Assignment Operator**

Move Assignment is also used to transfer a connection to a Stop State.

```
#include <iostream>
#include <stop_token>
#include <utility>

int main() {
    std::stop_source source1;
    std::stop_source source2;

    source2 = std::move(source1);

    std::cout << source2.stop_possible() << '\n';
    std::cout << source1.stop_possible() << '\n';
}
```

Output:

```
1
0
```

This operation is useful for transferring Source ownership in designs based on Move Semantics.

### **The&#x20;**`swap()`**&#x20;Function**

The `swap()` function allows the Stop States of two Sources to be exchanged.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source1;
    std::stop_source source2{std::nostopstate};

    source1.swap(source2);

    std::cout << source1.stop_possible() << '\n';
    std::cout << source2.stop_possible() << '\n';
}
```

Output:

```
0
1
```

This function only exchanges the connections of the two objects; it does not modify the internal state of the Stop States themselves.

## **Comparing&#x20;**`std::stop_source`**&#x20;and&#x20;**`std::stop_token`

These two types are closely related, but they do not have the same responsibilities.

### **Difference in Responsibilities**

The `std::stop_source` type is designed to send stop requests.

In contrast, `std::stop_token` is used to observe and check stop status.

Typically, the Source is held by the part of the program responsible for control, while the Token is passed to the executing thread or task.

```
#include <iostream>
#include <stop_token>
#include <thread>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Working...\n";

        std::this_thread::sleep_for(
            std::chrono::milliseconds(100)
        );
    }

    std::cout << "Worker stopped\n";
}

int main() {
    std::stop_source source;

    std::thread thread(worker, source.get_token());

    std::this_thread::sleep_for(
        std::chrono::milliseconds(500)
    );

    source.request_stop();

    thread.join();
}
```

In this example:

- The main thread owns the Stop Source.
- The worker thread receives only a Token.
- The worker thread checks the stop status in its loop.
- The main thread requests a cooperative termination by sending a stop request.

This pattern provides a useful separation between control logic and execution logic.

### **Differences in Capabilities**

| Capability                          | `std::stop_source` | `std::stop_token`            |
| ----------------------------------- | ------------------ | ---------------------------- |
| Creates a Stop State                | Yes                | No                           |
| Sends a stop request                | Yes                | No                           |
| Checks stop requests                | Yes                | Yes                          |
| Checks whether stopping is possible | Yes                | Yes                          |
| Creates a Token                     | Yes                | No                           |
| Registers a callback                | No                 | Through `std::stop_callback` |
| Copyable                            | Yes                | Yes                          |
| Connected to a shared Stop State    | Yes                | Yes                          |

This separation means that code that only needs to detect whether an operation should stop does not need access to the ability to send stop requests.

## **Using&#x20;**`std::stop_source`**&#x20;with&#x20;**`std::thread`

### **Designing Cooperative Cancellation for&#x20;**`std::thread`

In C++20, `std::stop_source` and `std::stop_token` can be used to manage the termination of threads created with `std::thread`.

Unlike `std::jthread`, the `std::thread` class does not automatically pass a Stop Token to the executing function.

Therefore, the Token must be passed manually.

```
#include <chrono>
#include <iostream>
#include <stop_token>
#include <thread>

void worker(std::stop_token token) {
    int counter = 0;

    while (!token.stop_requested()) {
        std::cout << "Counter: " << counter++ << '\n';

        std::this_thread::sleep_for(
            std::chrono::milliseconds(200)
        );
    }

    std::cout << "Worker finished\n";
}

int main() {
    std::stop_source source;

    std::thread workerThread(
        worker,
        source.get_token()
    );

    std::this_thread::sleep_for(
        std::chrono::seconds(2)
    );

    source.request_stop();

    workerThread.join();
}
```

This example demonstrates one of the primary use cases of `std::stop_source`.

### **Why Is&#x20;**`join()`**&#x20;Necessary?**

In the previous example, after requesting a stop, the main thread must wait for the child thread to finish executing.

The `request_stop()` function only registers a stop request. It does not guarantee that the thread will terminate immediately.

Therefore, using `join()` to synchronize thread termination is important.

If a `std::thread` object is destroyed while it is still joinable, the program calls `std::terminate`.

For this reason, proper thread lifetime management must be designed independently of the mechanism used to send stop requests.

## **Using&#x20;**`std::stop_source`**&#x20;with&#x20;**`std::jthread`

### **Introducing&#x20;**`std::jthread`

The `std::jthread` class was introduced in C++20 and provides more advanced thread lifetime management features than `std::thread`.

Its most important features include:

- Built-in support for `std::stop_source`.
- Automatic delivery of a `std::stop_token` to the executing function when its signature supports it.
- Automatic stop requests when the thread object is destroyed.
- Automatic joining when the thread object is destroyed.

For these reasons, `std::jthread` is a suitable choice for many modern C++20 applications.

### **Managing Cancellation with an Automatic Stop Token**

In the following example, `std::jthread` automatically passes a Stop Token to the executing function.

```
#include <chrono>
#include <iostream>
#include <thread>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Worker is running\n";

        std::this_thread::sleep_for(
            std::chrono::milliseconds(200)
        );
    }

    std::cout << "Worker stopped\n";
}

int main() {
    std::jthread thread(worker);

    std::this_thread::sleep_for(
        std::chrono::seconds(2)
    );

    thread.request_stop();
}
```

In this example, there is no need to create a `std::stop_source` manually because `std::jthread` already has its own internal Stop Source.

The `request_stop()` function provided by `std::jthread` uses the same Stop Token mechanism.

### **Using an Independent Stop Source Alongside&#x20;**`std::jthread`

Sometimes a controller outside the `std::jthread` object needs to send a stop request.

In such cases, an independent `std::stop_source` can be created, and its Token can be passed to the executing function.

```
#include <chrono>
#include <iostream>
#include <stop_token>
#include <thread>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Processing task\n";

        std::this_thread::sleep_for(
            std::chrono::milliseconds(200)
        );
    }

    std::cout << "Task stopped\n";
}

int main() {
    std::stop_source source;

    std::jthread thread(
        worker,
        source.get_token()
    );

    std::this_thread::sleep_for(
        std::chrono::seconds(2)
    );

    source.request_stop();
}
```

In this case, the stop request is sent through the independent Source.

An important point is that `std::jthread` still has its own internal Stop Source. However, the executing function in this example receives the Token associated with the independent Source.

Therefore, an automatic stop request from `std::jthread` will not necessarily stop the executing function, because that function is using a different Token.

This is an important design consideration when combining an independent `std::stop_source` with `std::jthread`.

## **Exploring&#x20;**`std::stop_callback`

### **The Role of Callbacks in Cancellation**

Sometimes, repeatedly checking the stop status in a loop is not appropriate.

For example, a thread might be waiting to receive data, an event, or the release of a resource.

In such situations, `std::stop_callback` makes it possible to execute a function when a stop request is registered.

This class is also part of the C++20 Stop Token mechanism.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token = source.get_token();

    std::stop_callback callback(
        token,
        [] {
            std::cout << "Cancellation requested\n";
        }
    );

    source.request_stop();
}
```

In this example, the callback executes when the stop request is registered.

### **Callback Behavior When Registered After a Stop Request**

If a callback is created after a stop request has already been registered, it executes immediately in the thread constructing the callback.

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

This behavior is important because you should not assume that a callback will only execute in the future, after it has been constructed.

### **Behavior During Callback Destruction**

If a stop request has not yet been registered, destroying a `std::stop_callback` object unregisters the callback.

However, if the callback is already executing, destruction must be coordinated with the completion of its execution.

As a result, the destructor of this type may block under certain circumstances.

Also, if a callback is executing in the same thread that is destroying its callback object, and it directly or indirectly causes that same object to be destroyed, the standard's restrictions concerning callback execution and lifetime must be taken into account.

When designing callbacks, avoid complicated dependencies and cycles of function calls.

### **Important Callback Design Considerations**

Callbacks may execute in the thread that requests the stop.

As a result:

- Avoid performing long-running operations inside callbacks.
- Avoid acquiring locks that might already be held by the requesting thread.
- Avoid blocking waits inside callbacks.
- Avoid accessing objects that might be destroyed before the callback executes.
- For expensive cleanup operations, it is better to signal an event and let the responsible thread perform the cleanup.

## **Using&#x20;**`std::stop_source`**&#x20;to Manage Multiple Threads**

### **Stopping Multiple Threads with One Stop Source**

One important use case for `std::stop_source` is sending a shared stop request to multiple threads.

Since multiple tokens can connect to the same Stop State, a single Source can be shared among several threads.

```
#include <chrono>
#include <iostream>
#include <stop_token>
#include <thread>
#include <vector>

void worker(
    int id,
    std::stop_token token
) {
    while (!token.stop_requested()) {
        std::cout << "Worker " << id << " running\n";

        std::this_thread::sleep_for(
            std::chrono::milliseconds(200)
        );
    }

    std::cout << "Worker " << id << " stopped\n";
}

int main() {
    std::stop_source source;

    std::vector<std::thread> workers;

    for (int i = 0; i < 4; ++i) {
        workers.emplace_back(
            worker,
            i,
            source.get_token()
        );
    }

    std::this_thread::sleep_for(
        std::chrono::seconds(2)
    );

    source.request_stop();

    for (auto& thread : workers) {
        thread.join();
    }
}
```

In this example, four threads perform an operation concurrently.

When `request_stop()` is called, all threads observe the same stop request.

### **Advantages of Using a Shared Stop Source**

This approach has several advantages:

- There is no need to maintain a separate stop variable for each thread.
- Stop management is centralized.
- Adding new threads becomes simpler.
- Shared shutdown mechanisms can be implemented.
- Communication between the controller and worker threads becomes simpler.

However, sharing a Stop Source does not mean that all threads share their execution state. Each thread still needs its own stopping and cleanup logic.

## **Using&#x20;**`std::stop_source`**&#x20;in Asynchronous Systems**

### **Applications in Task Management**

In systems where tasks execute asynchronously, one part of the application is usually responsible for scheduling or managing tasks, while another part performs the actual work.

In such situations, `std::stop_source` can be used to propagate cancellation requests.

For example, an image-processing task might consist of several stages:

- Receiving image data.
- Decoding the data.
- Applying filters.
- Saving the output.

If the user cancels the operation, the task can check the Token's status at the boundaries between stages.

```
#include <iostream>
#include <stop_token>
#include <vector>

void processImage(
    const std::vector<int>& pixels,
    std::stop_token token
) {
    for (std::size_t i = 0; i < pixels.size(); ++i) {
        if (token.stop_requested()) {
            std::cout << "Image processing cancelled\n";
            return;
        }

        // Process the current pixel
    }

    std::cout << "Image processing completed\n";
}

int main() {
    std::stop_source source;

    std::vector<int> pixels(1000000, 255);

    processImage(pixels, source.get_token());
}
```

In this example, the processing operation can receive a stop request.

However, for the example to be genuinely asynchronous, the function must execute in a separate thread or through an executor.

### **Differences from&#x20;**`std::future`**&#x20;and&#x20;**`std::async`

The `std::stop_source` mechanism is not a direct replacement for `std::future` or `std::async`.

These tools have different responsibilities:

| Feature            | Purpose                                                            |
| ------------------ | ------------------------------------------------------------------ |
| `std::stop_source` | Sends a stop request                                               |
| `std::stop_token`  | Observes a stop request                                            |
| `std::future`      | Retrieves the result of an asynchronous operation                  |
| `std::promise`     | Produces and sets a result for a Future                            |
| `std::async`       | Executes an operation using a defined asynchronous execution model |

In C++20, `std::future` and `std::async` do not generally accept the Stop Token mechanism.

Therefore, if a task needs to support cancellation, the Stop Token must be passed as part of the task's own design.

Also, registering a stop request does not automatically cancel a `std::async` operation.

## **Exploring Thread Safety**

### **Safety of Concurrent Calls**

The classes associated with Stop Tokens are designed for use in multithreaded environments.

In particular, operations involving the Stop State have the internal synchronization mechanisms needed to coordinate concurrent access.

For example, multiple threads can call `request_stop()` concurrently on Sources connected to the same Stop State.

```
#include <iostream>
#include <stop_token>
#include <thread>

int main() {
    std::stop_source source;

    std::thread thread1([&source] {
        std::cout << source.request_stop() << '\n';
    });

    std::thread thread2([&source] {
        std::cout << source.request_stop() << '\n';
    });

    thread1.join();
    thread2.join();
}
```

In this example, only one call can register the first stop request.

Consequently, one call returns `true` and the other returns `false`.

The output order of the two threads is not deterministic.

### **Thread Safety Is Different from Data-Race Safety**

The fact that the Stop State is thread-safe does not mean that all data used alongside it is also thread-safe.

For example:

```
#include <stop_token>

int sharedCounter = 0;

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        ++sharedCounter;
    }
}
```

In this example, if multiple threads execute `worker` concurrently, simultaneous access to `sharedCounter` can cause a data race.

The Stop Token mechanism does not protect against such accesses.

To manage shared data correctly, use tools such as:

- `std::mutex`
- `std::atomic`
- `std::shared_mutex`
- Other synchronization mechanisms appropriate to the problem

### **Using Multiple Shared Stop Sources**

If multiple `std::stop_source` objects are connected to the same Stop State, all of them can send stop requests.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source1;
    std::stop_source source2 = source1;

    source2.request_stop();

    std::cout << source1.stop_requested() << '\n';
}
```

Output:

```
1
```

In this case, copying the Source does not create two independent cancellation channels. Both Sources are connected to the same Stop State.

## **Memory Model and Memory Ordering**

### **Synchronization Guarantees**

The Stop Token mechanism is more than a simple Boolean variable.

Operations involving the Stop State must be implemented in a thread-safe manner, and their behavior follows the rules of the standard memory model.

In the standard, registering a stop request and observing it through a Token have defined synchronization guarantees.

In particular, registering a stop request through the Stop State operations is synchronized with observing that request in other threads.

This allows a program to use Stop Tokens as a mechanism for propagating stop requests.

However, you should not assume that every kind of access to shared data is automatically safe simply because a Stop Token is being used.

If data is modified concurrently without an appropriate synchronization mechanism, a data race can still occur.

### **Why Isn't a Stop Token a Replacement for an Atomic?**

A `std::atomic<bool>` can represent a simple stop flag.

However, Stop Tokens provide additional capabilities:

- Management of the lifetime of the stop state.
- Sharing the state among multiple Sources and Tokens.
- Callback registration.
- Integration with `std::jthread`.
- Support for waiting mechanisms that work with Stop Tokens.

Therefore, the choice between a Stop Token and an Atomic should be based on the application's actual requirements.

## **Using Stop Tokens in Waiting Operations**

### **The Problem with Waiting Loops**

One common problem in concurrent programs is waiting for data to arrive or for an event to occur.

For example:

```
while (!token.stop_requested()) {
    if (hasData()) {
        processData();
    }
}
```

This pattern creates a Busy Loop and may consume a significant amount of CPU time.

Using `sleep_for` is not always a suitable solution either, because a stop request may not be detected immediately.

### **Using&#x20;**`condition_variable_any`

In C++20, `std::condition_variable_any` provides support for waiting with a Stop Token.

One of its `wait` overloads accepts a Stop Token.

Example:

```
#include <chrono>
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <stop_token>
#include <thread>

int main() {
    std::mutex mutex;
    std::condition_variable_any cv;

    bool ready = false;

    std::jthread worker(
        [&](std::stop_token token) {
            std::unique_lock lock(mutex);

            bool completed = cv.wait(
                lock,
                token,
                [&] {
                    return ready;
                }
            );

            if (completed) {
                std::cout << "Work is ready\n";
            } else {
                std::cout << "Waiting cancelled\n";
            }
        }
    );

    std::this_thread::sleep_for(
        std::chrono::seconds(1)
    );

    worker.request_stop();
}
```

In this example, the thread can receive a stop request even while it is waiting.

An important point is that this capability belongs to `std::condition_variable_any`. It should not be confused with `std::condition_variable`.

### **Difference Between the Predicate and the Stop Request**

The `wait` function that accepts a Stop Token involves two different conditions:

- The Predicate determines whether the expected event has occurred.
- The Stop Token determines whether the waiting operation should be cancelled because a stop request has been made.

The function's return value indicates whether the Predicate is ultimately satisfied.

If a stop request is registered, the wait can finish, but this does not necessarily mean that the Predicate has become true.

Also, if the Predicate becomes true at the same time as a stop request is made, the result of the function should be interpreted by checking the final Predicate state.

## **Using a Stop Source to Design Shutdown**

### **Managing Controlled Application Shutdown**

One important use of `std::stop_source` is managing shutdown in applications where several threads are continuously active.

For example, a server might have multiple workers processing incoming requests.

During shutdown, the application needs to:

- Prevent new work from being accepted.
- Send stop requests to the workers.
- Allow ongoing operations to terminate in a controlled manner whenever possible.
- Release resources.
- Wait for the threads to finish.

A simple example:

```
#include <chrono>
#include <iostream>
#include <stop_token>
#include <thread>
#include <vector>

class WorkerPool {
private:
    std::stop_source stopSource;
    std::vector<std::thread> workers;

public:
    void start(int count) {
        for (int i = 0; i < count; ++i) {
            workers.emplace_back(
                [this, i](std::stop_token token) {
                    while (!token.stop_requested()) {
                        std::cout
                            << "Worker "
                            << i
                            << " processing\n";

                        std::this_thread::sleep_for(
                            std::chrono::milliseconds(200)
                        );
                    }

                    std::cout
                        << "Worker "
                        << i
                        << " stopped\n";
                },
                stopSource.get_token()
            );
        }
    }

    void shutdown() {
        stopSource.request_stop();

        for (auto& worker : workers) {
            if (worker.joinable()) {
                worker.join();
            }
        }
    }

    ~WorkerPool() {
        shutdown();
    }
};

int main() {
    WorkerPool pool;

    pool.start(4);

    std::this_thread::sleep_for(
        std::chrono::seconds(2)
    );

    pool.shutdown();
}
```

In this example, a shared Stop Source sends a stop request to all workers.

The `shutdown()` function first registers the stop request and then waits for all threads to finish.

**Design note:** A production implementation should also handle concurrent calls to `start()` and `shutdown()`, and access to the thread container should be protected by appropriate synchronization. It should also prevent `shutdown()` from being called by one of the workers in the same pool, because joining the current thread is not valid.

## **Limitations of&#x20;**`std::stop_source`

### **Immediate Termination Is Not Guaranteed**

The most important limitation of this mechanism is that a stop request is only a logical signal.

If a thread does not check for the stop request, it may continue executing indefinitely.

For example:

```
#include <stop_token>

void worker(std::stop_token token) {
    while (true) {
        // Long-running operation without cancellation checks
    }
}
```

In this example, registering a stop request has no effect on the loop.

Therefore, operations that support cancellation must be designed from the beginning with specific points at which the stop status is checked.

### **Blocking Operations May Not Be Cancellable**

Suppose a thread is inside a blocking function waiting to receive data from an external API.

If that API does not provide a way to cancel the wait, simply sending a stop request will not wake the thread.

To solve this problem, use approaches such as:

- Waiting APIs that support Stop Tokens.
- `std::condition_variable_any`.
- Notification mechanisms.
- Bounded timeouts.
- Cancellable APIs provided by the operating system or the library in use.

### **A Stop State Cannot Be Reset**

Once a stop request has been registered, the Stop State does not return to its initial state.

Therefore, a new Stop Source must be created to begin a new cancellation cycle.

```
#include <stop_token>

int main() {
    std::stop_source source1;

    source1.request_stop();

    std::stop_source source2;
}
```

In this example, the second Source has an independent Stop State.

### **Callback Management Requires Care**

Because callbacks execute as part of the stop-request process, they can increase the time required for `request_stop()` to return.

Therefore, if a callback performs expensive or blocking operations, the requesting thread may also remain occupied for a significant amount of time.

This is particularly important in applications where latency matters.

### **Stopping an Operation Does Not Mean Rolling It Back**

A stop request simply informs the application that an operation should stop.

This mechanism does not guarantee that previous changes will be reversed.

For example, if a task has already stored some data in a database, requesting a stop does not automatically roll back the transaction.

Transaction management, cleanup, and consistency remain the application's responsibility.

## **Common Mistakes When Using&#x20;**`std::stop_source`

### **Mistaking a Stop Request for Forced Thread Termination**

A common mistake is assuming that calling `request_stop()` immediately terminates a thread.

In reality, this function only registers a stop request.

The thread must receive and process the request itself.

### **Using Independent Sources by Mistake**

In the following example, two independent Stop Sources are created:

```
#include <stop_token>

int main() {
    std::stop_source source1;
    std::stop_source source2;

    source1.request_stop();

    std::stop_token token = source2.get_token();
}
```

In this example, the stop request sent through `source1` has no effect on the Token associated with `source2`.

If the goal is to share the stop status, the Source must be copied, or the tokens must be obtained from the same shared Source.

### **Incorrectly Assuming That Copies Are Independent**

Developers sometimes assume that copying a `std::stop_source` creates an independent Stop State.

This assumption is incorrect.

Copying only creates another connection to the same Stop State.

To create an independent stop state, construct a new Stop Source using the default constructor.

### **Ignoring the Moved-From State**

After transferring a Source with `std::move`, you should not assume that the original object remains connected to its previous Stop State.

The source object is left without a Stop State.

Therefore, any subsequent use must take its new state into account.

### **Performing Expensive Operations Inside Callbacks**

Since a callback may execute in the thread that requests the stop, performing expensive operations inside it can introduce unexpected delays.

It is better for a callback to perform only lightweight notification.

### **Ignoring Object Lifetimes**

If a callback refers to objects that are destroyed before the callback executes, dangling references or undefined behavior may result.

To prevent this problem, the lifetimes of objects used by callbacks must be managed correctly.

## **Comparing&#x20;**`std::stop_source`**&#x20;with Traditional Approaches**

In versions before C++20, stopping a thread typically involved shared variables, `std::atomic<bool>`, combinations of `std::mutex` and `std::condition_variable`, or operating-system-specific mechanisms.

The main problem with these approaches was that programmers had to design the cancellation protocol themselves: maintaining the stop status, notifying other threads when it changed, and handling data races and the wake-up of waiting threads.

`std::stop_source` and `std::stop_token` provide a standardized mechanism for **Cooperative Cancellation** in C++20. In this model, one part of the program requests a stop, while other parts respond by checking a Token or registering a callback. Termination is not forced; the executing code must cooperate with the stop request.

### **1. Comparison with&#x20;**`std::atomic<bool>`

A common and straightforward traditional approach is to use an atomic variable as a stop flag:

```
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<bool> stopRequested{false};

void worker()
{
    while (!stopRequested.load())
    {
        // Perform a unit of work
    }

    std::cout << "Worker stopped\n";
}

int main()
{
    std::thread t(worker);

    // Request the worker to stop
    stopRequested.store(true);

    t.join();
}
```

This approach is simple, but it has several limitations:

- You must manage the atomic variable manually.
- With multiple workers, you generally need to share the flag correctly among all of them.
- There is no built-in standardized mechanism for registering a callback when a stop is requested.
- If a thread is waiting inside a blocking operation, changing the flag does not necessarily wake it.

The same example using `std::stop_source` and `std::stop_token`:

```
#include <stop_token>
#include <thread>
#include <iostream>

void worker(std::stop_token token)
{
    while (!token.stop_requested())
    {
        // Perform a unit of work
    }

    std::cout << "Worker stopped\n";
}

int main()
{
    std::stop_source source;

    std::thread t(worker, source.get_token());

    // Request cooperative cancellation
    source.request_stop();

    t.join();
}
```

The key advantage of this design is the separation of responsibilities:

- `std::stop_source` is responsible for requesting a stop.
- `std::stop_token` is provided to the Worker so it can observe the stop status.
- Any number of Tokens obtained from the same Source observe the same stop status.

**Note:** `std::stop_source` is not a complete replacement for `std::atomic<bool>`. If you only need a general-purpose atomic flag and do not need cooperative cancellation features, an Atomic may still be appropriate.

### **2. Comparison with&#x20;**`std::mutex`**&#x20;and&#x20;**`std::condition_variable`

In some applications, a worker waits for data or an event instead of continuously executing work. In this situation, changing a stop flag alone is not enough; the thread must also be woken from its waiting state.

The traditional approach usually combines a Mutex, a Condition Variable, and a stop flag:

```
#include <mutex>
#include <condition_variable>
#include <thread>

std::mutex mtx;
std::condition_variable cv;
bool stopRequested = false;

void worker()
{
    std::unique_lock<std::mutex> lock(mtx);

    cv.wait(lock, [] {
        return stopRequested;
    });

    // Continue shutdown
}

int main()
{
    std::thread t(worker);

    {
        std::lock_guard<std::mutex> lock(mtx);
        stopRequested = true;
    }

    cv.notify_all();

    t.join();
}
```

Several points must be considered when using this approach:

- The flag must be modified while holding the same Mutex.
- After changing the state, you must wake the waiting threads.
- The waiting condition should be checked using a Predicate to handle spurious wakeups correctly.
- If there are several types of events, managing the waiting conditions becomes more complicated.

In C++20, `std::condition_variable_any` can be used to associate a wait operation with a Stop Token:

```
#include <condition_variable>
#include <mutex>
#include <stop_token>
#include <thread>
#include <iostream>

std::mutex mtx;
std::condition_variable_any cv;

void worker(std::stop_token token)
{
    std::unique_lock<std::mutex> lock(mtx);

    cv.wait(lock, token, [] {
        return false;
    });

    if (token.stop_requested())
    {
        std::cout << "Cancellation received\n";
    }
}

int main()
{
    std::jthread t(worker);

    // Request stop and wake the waiting worker
    t.request_stop();
}
```

In this example, the wait operation is integrated with the Stop Token. When a stop is requested, the wait can be cancelled, so there is no need to maintain a separate flag and manually call `notify_all()` for this purpose.

However, this capability does not mean that `condition_variable` is completely replaced. Predicates and notifications are still used when waiting for ordinary events.

### **3. Comparison with&#x20;**`pthread_cancel`

In POSIX environments, the `pthread_cancel()` function is available to request the cancellation of a thread. This approach differs significantly from C++20 cooperative cancellation.

`pthread_cancel()` can cancel the target thread at specific points in its execution, and its behavior depends on the cancellation type (Deferred or Asynchronous Cancellation).

In contrast, `std::stop_source`:

- Does not forcibly terminate a thread.
- Only registers a stop request.
- Allows the worker code to respond at safe and appropriate points.
- Does not depend on operating-system-specific mechanisms.

Forced thread cancellation may leave Mutexes locked, resources unreleased, or data in an inconsistent state if it occurs at an unsuitable point. The cooperative model in C++20 is better suited to controlled thread lifetime management.

| Feature                     | `std::atomic<bool>` | `std::mutex` and `std::condition_variable` | `std::stop_source`                                          |
| --------------------------- | ------------------- | ------------------------------------------ | ----------------------------------------------------------- |
| Checking stop status        | With `load()`       | By checking a flag                         | With `stop_requested()`                                     |
| Registering a stop callback | Not available       | Must be implemented manually               | With `std::stop_callback`                                   |
| Cancelling a blocking wait  | Not by itself       | With `notify_one()` or `notify_all()`      | With compatible facilities such as `condition_variable_any` |
| Forced thread termination   | No                  | No                                         | No                                                          |
| Standard C++ support        | Since C++11         | Since C++11                                | Since C++20                                                 |

### **5. Advanced Design Considerations with&#x20;**`std::stop_source`

#### **5.1. Hierarchical Cancellation**

Large applications may contain several subsystems, each with multiple workers.

For example:

- A server has multiple connections.
- Each connection runs several concurrent operations.
- When the server closes, all related operations must stop.

You can create an independent Stop Source for each level and use callbacks to propagate a stop request from a higher level to a lower one.

```
#include <stop_token>
#include <thread>
#include <iostream>

int main()
{
    std::stop_source serverSource;
    std::stop_source connectionSource;

    std::stop_callback connectionCallback(
        serverSource.get_token(),
        [&connectionSource] {
            connectionSource.request_stop();
        }
    );

    std::jthread worker(
        [token = connectionSource.get_token()](std::stop_token)
        {
            while (!token.stop_requested())
            {
                // Process connection work
            }

            std::cout << "Connection worker stopped\n";
        }
    );

    // Stop the server and propagate cancellation
    serverSource.request_stop();
}
```

In this design, stopping the server propagates the request to the connection, and the worker observes it through the connection's Token.

An important point is that `std::stop_source` does not automatically establish parent-child relationships. Stop-request propagation must be implemented explicitly.

#### **5.2. Combining Multiple Stop Sources**

Sometimes an operation must stop when any one of several events occurs:

- The user cancels the operation.
- The server is shutting down.
- The operation's deadline expires.

`std::stop_token` does not directly support combining multiple Tokens. However, an aggregate Token can be created using callbacks.

```
#include <stop_token>
#include <thread>

int main()
{
    std::stop_source userSource;
    std::stop_source serverSource;
    std::stop_source combinedSource;

    std::stop_callback userCallback(
        userSource.get_token(),
        [&combinedSource] {
            combinedSource.request_stop();
        }
    );

    std::stop_callback serverCallback(
        serverSource.get_token(),
        [&combinedSource] {
            combinedSource.request_stop();
        }
    );

    // Either source can stop the combined operation
    serverSource.request_stop();

    return combinedSource.stop_requested() ? 0 : 1;
}
```

In this example, a stop request from either of the two Sources causes the combined Source to stop.

This approach is useful in systems such as Task Schedulers, servers, and processing pipelines.

#### **5.3. Races During Callback Registration**

One important feature of `std::stop_callback` is its behavior under concurrent conditions.

Suppose one thread is executing `request_stop()` while another thread attempts to register a callback at the same time.

The standard behavior is as follows:

- If the callback is registered before the stop request, it is executed when the stop is requested.
- If the stop has already been requested, the callback executes immediately in the thread registering it.
- A particular callback registration is executed at most once.

Therefore, you should not assume that a callback will always execute in the thread requesting the stop.

Also, a callback may execute during its registration. For this reason, avoid registering callbacks while holding sensitive locks.

#### **5.4. Avoiding Deadlocks and Reentrancy**

Callbacks can execute in the thread requesting the stop. Therefore, if a callback locks a Mutex, you must ensure that the same Mutex is not already locked by the requesting thread.

An unsuitable example:

```
#include <mutex>
#include <stop_token>

std::mutex mtx;

void example(std::stop_token token)
{
    std::lock_guard<std::mutex> lock(mtx);

    std::stop_callback callback(
        token,
        [] {
            std::lock_guard<std::mutex> innerLock(mtx);

            // Perform cleanup
        }
    );

    // Other operations
}
```

If the Token has already been stopped, the callback executes during registration. As a result, attempting to lock the same non-recursive Mutex again can cause a deadlock.

Recommended approaches include:

- Register callbacks outside the scope of sensitive locks.
- Keep callback operations short and predictable.
- Avoid calling complex functions or code that might re-enter the same execution path.
- If necessary, delegate cleanup operations to another worker.

### **6. Reviewing the API and Standard Specifications**

The `std::stop_source` class is defined in the following header:

```
#include <stop_token>
```

Its main members include:

| Member                         | Purpose                                    |
| ------------------------------ | ------------------------------------------ |
| `std::stop_source()`           | Creates a new Source                       |
| `std::stop_source(token)`      | Constructs a Source from an existing Token |
| `get_token()`                  | Retrieves a Token                          |
| `stop_requested()`             | Checks whether a stop has been requested   |
| `stop_possible()`              | Checks whether a stop request is possible  |
| `request_stop()`               | Requests a stop                            |
| `stop_source::stop_possible()` | Checks whether a usable Stop State exists  |
| `swap()`                       | Exchanges the states of two Sources        |
| `operator bool()`              | Checks whether the Source is valid         |

Two important concepts must not be confused:

`stop_possible()`

Determines whether the Source has a usable Stop State through which a stop request can be made.

`stop_requested()`

Determines whether a stop request has already been registered.

Therefore, the following state is possible:

```
source.stop_possible() == true
source.stop_requested() == false
```

This means that a stop request is possible, but no stop has been requested yet.

Also, if no Stop State exists, the associated Token cannot observe a stop request.

### **7. Compiler Compatibility**

To use `std::stop_source`, both the compiler and the standard library must support C++20.

Example compilation with GCC:

```
g++ -std=c++20 -pthread main.cpp -o app
```

Example compilation with Clang:

```
clang++ -std=c++20 -pthread main.cpp -o app
```

Note: Enabling the `-std=c++20` option alone is not sufficient. The version of the standard library must also implement this feature.

### **8. Summary**

`std::stop_source` is an important component of the cooperative cancellation model introduced in C++20. It allows an application to propagate stop requests in a standardized and thread-safe manner without forcibly terminating threads.

Its most important advantages include:

1. Separating the responsibility for requesting a stop from the responsibility for observing it.
2. Sharing one stop status among multiple threads.
3. Supporting callbacks through `std::stop_callback`.
4. Propagating stop requests across multiple subsystems.
5. Integrating with `std::jthread` and certain waiting mechanisms in C++20.
6. Reducing the need to implement cancellation protocols manually.

Ultimately, remember that `std::stop_source` is a tool for **requesting a stop**, not for **forcibly terminating a thread**.

Correct design still requires workers to check for stop requests at appropriate points, release their resources, and use a well-defined method to exit their functions or threads.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>