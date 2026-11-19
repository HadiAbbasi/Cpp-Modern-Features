<div align="center">

[🇺🇸 English](./coroutines.md) | [🇮🇷 فارسی](../../fa/cpp20/coroutines.md)

</div>

---

# **Exploring Coroutines in C++**

## **Introduction**

The **Coroutines** feature in C++20 is an important language capability for writing computations that can be suspended and resumed. A Coroutine can be considered an extension of a Function: it is called like a regular function, but unlike an ordinary function, it can suspend execution at specific points, preserve the necessary state, and later resume from where it left off.

This feature is useful for building data streams, generating values incrementally, implementing Asynchronous Operations, managing long-running sequences, and simplifying code that is inherently multi-step.

An important point is that **a Coroutine is not a Thread by itself**. Suspending a Coroutine does not necessarily mean that it will execute on another Thread, and using `co_await` does not automatically make an operation Asynchronous. When and how execution resumes depends on the type of Coroutine, the Awaitable contract, and the Scheduler or library used in the system design.

## **Function vs. Coroutine**

### **How an Ordinary Function Behaves**

In an ordinary Function, execution begins at the function's entry point and terminates when it reaches `return` or the end of the function body. After the function returns, its local state is no longer available as the active state of that invocation.

For example:

```
int calculate() {
    int value = 10;
    value += 5;
    return value;
}
```

### **How a Coroutine Behaves**

A Coroutine can be suspended at specific points during execution, in addition to starting and finishing. When suspended, the state required to resume execution is preserved in a structure called the **Coroutine State**. Later, when the Coroutine is resumed, execution continues from the suspended point.

```
generator<int> generate_numbers() {
    co_yield 10;
    co_yield 20;
    co_yield 30;
}
```

In this example, each time the consumer requests the next value, the Coroutine executes until the next `co_yield`, provides the value to the consumer, and suspends. The `generator<T>` definition in this example is only a hypothetical type used to demonstrate the Generator concept; C++20 does not provide such a type in the Standard Library.

### **Comparing the Behavior of Both Function Types**

| Feature                                  | Ordinary Function              | Coroutine                                                   |
| ---------------------------------------- | ------------------------------ | ----------------------------------------------------------- |
| Can be called                            | Yes                            | Yes                                                         |
| Can return a result                      | Yes                            | Yes, through the mechanism defined by its Promise           |
| Can temporarily suspend execution        | No, not normally               | Yes                                                         |
| Can resume from a suspended point        | No                             | Yes                                                         |
| Preserves local state while suspended    | No, after the function returns | Yes, for variables that remain alive in the Coroutine Frame |
| Automatically executes on another Thread | No                             | No                                                          |

## **Main Use Cases for Coroutines**

### **Lazy Value Generation**

With Lazy evaluation, a computation is not performed until the consumer needs its result. A Generator can produce values one at a time without requiring all results to be stored in a Container in advance.

### **Simplifying Asynchronous Code**

In Asynchronous designs, an operation sometimes needs to continue after an I/O operation or another event has completed. With a suitable Awaitable, `co_await` can identify the point where execution should wait, avoiding the need to write long chains of Callbacks.

### **Modeling Data Streams**

Coroutine-based Generators can be used for incremental file reading, event processing, traversing generated data, and building Pipelines.

## **How the Compiler Recognizes a Coroutine**

Any function whose body contains one of the following three Keywords is considered a Coroutine by the language:

- `co_await`
- `co_yield`
- `co_return`

Simply changing a function's return type or naming it a Coroutine is not sufficient. The presence of one of these Keywords in the function body makes it subject to the Coroutine rules.

Conversely, not every function in C++ can be a Coroutine. Examples of functions that are not Coroutines or are not permitted to use these Keywords include `main`, Constructors, Destructors, and `constexpr` functions. For functions subject to other restrictions, such as `consteval` functions or certain special contexts, the rules of the relevant language version must also be considered.

## **The Three Main Coroutine Keywords**

### **Introducing co_yield**

The `co_yield` expression provides a value from the Coroutine to its consumer and usually suspends Coroutine execution. This Keyword is particularly important for building Generators.

```
generator<int> numbers() {
    co_yield 10;
    co_yield 20;
    co_yield 30;
}
```

Conceptually, the following expression:

```
co_yield value;
```

is approximately transformed into the following call on the Promise Type:

```
co_await promise.yield_value(value);
```

This transformation provides a useful mental model for understanding how `co_yield` behaves. The exact details depend on the Coroutine rules and the Promise Type.

In this example, `co_yield 10` does not mean that the Coroutine has finished. The Coroutine provides the value `10` and suspends. When the consumer requests the next value, execution continues from the statement immediately following that `co_yield`.

### **Introducing co_return**

The `co_return` expression terminates Coroutine execution. Depending on the design of the Promise Type, a final value may also be stored or delivered.

There are two main forms:

```
co_return;
```

and:

```
co_return result;
```

The first form uses the `return_void()` function in the Promise Type, while the second uses `return_value(result)`.

A Promise Type cannot provide both `return_void()` and `return_value()` for the same Coroutine. The selected function must be compatible with how the Coroutine terminates.

An important distinction is that `co_return` and `co_yield` do different things: the former terminates the Coroutine, while the latter provides a value and usually suspends execution so that it can continue later.

### **Introducing co_await**

The `co_await` expression is used to wait for an operation or state that is Awaitable. It may suspend the current Coroutine and allow it to be resumed once the awaited result becomes available.

```
auto result = co_await operation;
```

In this example, `operation` must be convertible to an Awaiter according to the language rules. Whether the Coroutine is actually suspended depends on the result of `await_ready()` and the behavior of `await_suspend()`.

`co_await` does not create a new Thread by itself, nor does it guarantee that execution will continue on another Thread. For example, an Awaitable may return its result immediately without suspending if the result is already available.

## **Coroutine Workflow**

### **General Execution Steps**

The workflow of a Coroutine can be summarized in several steps:

1. The compiler identifies the corresponding Promise Type based on the Coroutine's return type.
2. The required space for storing the Coroutine's state, known as the Coroutine Frame, is provided.
3. Coroutine parameters and the variables required for continuing execution are stored in the Frame.
4. The Promise object is constructed.
5. The `get_return_object()` function is called to create the Coroutine's return object.
6. `initial_suspend()` determines whether the Coroutine begins executing immediately after initialization or remains suspended initially.
7. When the Coroutine is resumed, its body begins executing.
8. At `co_await` or `co_yield` expressions, the Coroutine may suspend.
9. When resumed again, execution continues from the suspended point.
10. At the end, `final_suspend()` and the Frame lifetime management mechanism determine how the Coroutine's state is preserved or released.

### **Eager and Lazy Execution**

In the **Eager** model, a Coroutine begins executing its body after being called and stops at the first suspension point. This behavior is commonly achieved by implementing `initial_suspend()` to return `std::suspend_never`.

In the **Lazy** model, a Coroutine is suspended immediately after its initial setup. Its body does not execute until the consumer resumes it. This behavior is commonly implemented by returning `std::suspend_always` from `initial_suspend()`.

Therefore, Lazy execution is not an inherent property of every Coroutine. It results from the contract defined by the Promise Type and the selected return type.

## **Coroutine Internals**

C++20 introduced low-level Coroutine mechanisms into the language. These mechanisms allow libraries to build different Coroutine types, such as Task, Generator, and Asynchronous operations.

Three important components of this infrastructure are:

- **Promise Type**: Defines the Coroutine's behavior and contract.
- **Coroutine Handle**: Provides access to the Coroutine for management and allows it to be resumed or destroyed.
- **Awaiter**: Defines the suspension and resumption behavior of a `co_await` expression.

### **Understanding the Coroutine Frame**

The compiler creates a structure called the Coroutine Frame to preserve the state required to continue execution. This Frame typically contains information such as:

- Parameters required for continuing Coroutine execution.
- Local variables whose lifetimes extend across a suspension point.
- The Coroutine's execution state and the point from which it must resume.
- The Promise object and data required by the Coroutine mechanism.

It should not be assumed that all local variables are always stored in the Frame in the same way. The implementation details are determined by the compiler. Also, the Frame is not necessarily always allocated on the Heap; under certain conditions, when the compiler can prove its lifetime and requirements, it may optimize the allocation.

### **Introducing the Promise Type**

The Promise Type is a C++ type that defines the behavioral contract of a Coroutine. It is usually identified through the return type of the Coroutine and the rules of `std::coroutine_traits`.

The Promise determines:

- How the Coroutine's return object is created.
- How the Coroutine behaves when it starts and finishes.
- How the value supplied by `co_return` is handled.
- How the value supplied by `co_yield` is provided.
- How an unhandled Exception is received.

Important Promise Type functions include:

| Function                | Responsibility                                                       |
| ----------------------- | -------------------------------------------------------------------- |
| `get_return_object()`   | Creates the Coroutine's return object                                |
| `initial_suspend()`     | Determines the initial suspension behavior                           |
| `final_suspend()`       | Determines the Coroutine's behavior when its body finishes           |
| `unhandled_exception()` | Receives an Exception that was not handled inside the Coroutine body |
| `return_void()`         | Supports `co_return;`                                                |
| `return_value(value)`   | Supports `co_return value;`                                          |
| `yield_value(value)`    | Defines the behavior of `co_yield value;`                            |

These functions must be implemented according to the standard rules and the Coroutine's return type. Their exact signatures and return types matter; the table above describes only their conceptual responsibilities.

### **Introducing the Coroutine Handle**

The `std::coroutine_handle` type is defined in the standard `<coroutine>` Header. This Handle represents access to a Coroutine Frame and provides important management operations to the implementer of a Coroutine type.

Its commonly used operations include:

- `resume()` to continue the execution of a suspended Coroutine.
- `done()` to check whether the Coroutine has reached its final suspended state.
- `destroy()` to destroy the Coroutine Frame.
- `promise()` to access the corresponding Promise in Handles that have a Promise Type.

An important point is that a Handle does not automatically provide safe ownership. If a Coroutine's return type owns the Handle, it must define exactly when the Frame is destroyed. Destroying an invalid Handle, destroying the same Frame multiple times, or resuming a Coroutine that is not in a suitable state can result in undefined behavior.

### **Introducing Awaitable and Awaiter**

An object that can appear on the right-hand side of `co_await` is called an Awaitable. Ultimately, the language rules must be able to obtain an Awaiter from it.

An Awaiter is an object that provides the following three main operations:

| Function                | Role                                                                             |
| ----------------------- | -------------------------------------------------------------------------------- |
| `await_ready()`         | Determines whether the result is already available and suspension is unnecessary |
| `await_suspend(handle)` | Manages suspension or schedules continuation when needed                         |
| `await_resume()`        | Produces the result of the `co_await` expression after execution resumes         |

A simple Awaiter example:

```
#include <coroutine>

struct ImmediateAwaiter {
    bool await_ready() const noexcept {
        return true;
    }

    void await_suspend(std::coroutine_handle<>) const noexcept {
    }

    int await_resume() const noexcept {
        return 42;
    }
};
```

Using it in a Coroutine:

```
Task<int> read_value() {
    int value = co_await ImmediateAwaiter{};
    co_return value;
}
```

In this example, because `await_ready()` returns `true`, the Coroutine is not suspended, and `await_resume()` is called immediately. The `Task<int>` type in this snippet is only an example return type and must be defined separately by a library or custom implementation.

### **Difference Between Awaitable and Awaiter**

An Awaitable is an object that can be used in a `co_await` expression. An Awaiter is an object that provides the three functions `await_ready`, `await_suspend`, and `await_resume`.

Sometimes the Awaitable itself is the Awaiter. In other cases, the Awaitable produces an Awaiter through `operator co_await` or through the conversion rules associated with the Promise.

### **Implementing a Custom Awaitable**

Custom waiting behavior can be created by designing a Class or Struct and providing a suitable Awaiter. For example, in an Asynchronous system, a real Awaitable may pass the Handle to an Event Loop or Scheduler from `await_suspend()`, then resume it when the operation becomes ready.

In such a design, several points must be considered:

- A Handle must not be retained or resumed after its Coroutine has been destroyed.
- Scheduling operations must prevent unintended concurrent or reentrant resumption.
- The lifetime of data referenced by the Awaiter must remain valid until the operation completes.
- Errors from Asynchronous operations must be propagated through `await_resume()` or the library's error-handling contract.
- Thread-safety must be specified by the Scheduler and the Awaitable contract.

## **Important Details of co_await**

### **Conceptual Evaluation Order**

After the language's conversion rules have been applied, the `co_await expr` expression obtains an Awaiter. In general, its behavior is as follows:

1. An Awaiter is obtained from the Awaitable expression.
2. The `await_ready()` function is called.
3. If `await_ready()` returns `true`, the Coroutine is not suspended, and `await_resume()` is executed.
4. If it returns `false`, the Coroutine enters the suspension path, and `await_suspend()` is called.
5. After the Coroutine resumes, `await_resume()` is executed and determines the result of the `co_await` expression.

The return type of `await_suspend()` can affect control flow. This function can return `void`, `bool`, or a Coroutine Handle; each has a specific meaning under the standard rules. Therefore, `await_suspend()` should not always be assumed to be a `void` function.

### **The Behavior of await_suspend**

If `await_suspend()` returns `void`, the Coroutine remains suspended, and the Awaiter is responsible for providing a way to resume it later.

If it returns a `bool`, returning `false` means that the Coroutine continues execution without remaining suspended, while returning `true` means that it is suspended.

If it returns a `std::coroutine_handle`, control can be transferred to another Coroutine. This capability is useful when implementing certain control-transfer patterns.

The implementation of `await_suspend()` requires careful attention. A Scheduler or another Thread may resume the Coroutine immediately after the work has been scheduled. Therefore, access to Awaiter data after publishing the Handle must be designed with the possibility of concurrent resumption in mind.

## **Important Details of co_yield**

### **The Role of yield_value in the Promise**

The `co_yield value` expression is processed through `yield_value(value)` in the Promise Type. The return value of `yield_value` must be compatible with the rules of `co_await`, because conceptually, `co_yield` is transformed into an Await operation.

This design allows a Generator to determine how a value is provided to its consumer and how the Coroutine remains suspended afterward.

### **Difference Between yield and return**

`co_yield` is used to provide an intermediate value while allowing execution to continue later. `co_return` is used to terminate the Coroutine. In a typical Generator, the consumer may receive several values before the Coroutine eventually finishes.

## **Important Details of co_return and Exception Handling**

### **Normal Coroutine Termination**

When execution reaches `co_return` or passes the end of the Coroutine body, the termination path is executed according to the Promise Type. For Coroutines that provide `return_void()`, reaching the end of the body can be equivalent to `co_return;`.

### **Exception Handling**

If an Exception escapes the Coroutine body without being handled inside it, the Promise's `unhandled_exception()` function is called. This function can store the Exception, propagate it to the consumer, or perform another action according to the Coroutine type's contract.

For example, a Task can store a `std::exception_ptr` in its Promise and rethrow it later from `await_resume()` using `std::rethrow_exception`.

### **Difference Between Execution Completion and Frame Deallocation**

The completion of a Coroutine body does not necessarily mean that the Coroutine Frame is immediately deallocated. Usually, the Coroutine first reaches the `final_suspend()` point. The Handle owner or return type must decide when to destroy the Frame according to its contract.

This separation is important because it allows the consumer to retrieve the final result or Exception from the Promise.

## **Building a Custom Generator**

### **The Goal of Implementing a Generator**

A Generator is a type of Coroutine that produces values incrementally. Each time the consumer requests the next value, the producer executes only until the next `co_yield` point.

Next, we will implement a simple educational Generator for `int` values. This example is designed to demonstrate the relationship between Promise, Handle, Iterator, and `co_yield`. It is not intended to replace a complete, production-ready library.

### **Implementing a Simple Generator**

```
#include <coroutine>
#include <exception>
#include <iterator>
#include <utility>

class Generator {
public:
    struct promise_type;
    using Handle = std::coroutine_handle<promise_type>;

    struct promise_type {
        int current_value{};

        Generator get_return_object() noexcept {
            return Generator{Handle::from_promise(*this)};
        }

        std::suspend_always initial_suspend() const noexcept {
            return {};
        }

        std::suspend_always final_suspend() const noexcept {
            return {};
        }

        std::suspend_always yield_value(int value) noexcept {
            current_value = value;
            return {};
        }

        void return_void() const noexcept {
        }

        void unhandled_exception() {
            exception = std::current_exception();
        }

        std::exception_ptr exception;
    };

    class iterator {
    public:
        using iterator_category = std::input_iterator_tag;
        using value_type = int;
        using difference_type = std::ptrdiff_t;
        using pointer = const int*;
        using reference = const int&;

        iterator() noexcept = default;

        explicit iterator(Handle handle) noexcept
            : handle_(handle) {
        }

        reference operator*() const noexcept {
            return handle_.promise().current_value;
        }

        pointer operator->() const noexcept {
            return &handle_.promise().current_value;
        }

        iterator& operator++() {
            handle_.resume();

            if (handle_.done()) {
                auto exception = handle_.promise().exception;
                if (exception) {
                    std::rethrow_exception(exception);
                }
            }

            return *this;
        }

        void operator++(int) {
            ++(*this);
        }

        friend bool operator==(const iterator& it,
                               std::default_sentinel_t) noexcept {
            return !it.handle_ || it.handle_.done();
        }

    private:
        Handle handle_{};
    };

    explicit Generator(Handle handle) noexcept
        : handle_(handle) {
    }

    Generator(const Generator&) = delete;
    Generator& operator=(const Generator&) = delete;

    Generator(Generator&& other) noexcept
        : handle_(std::exchange(other.handle_, {})) {
    }

    Generator& operator=(Generator&& other) noexcept {
        if (this != &other) {
            if (handle_) {
                handle_.destroy();
            }

            handle_ = std::exchange(other.handle_, {});
        }

        return *this;
    }

    ~Generator() {
        if (handle_) {
            handle_.destroy();
        }
    }

    iterator begin() {
        if (handle_) {
            handle_.resume();

            if (handle_.done()) {
                auto exception = handle_.promise().exception;
                if (exception) {
                    std::rethrow_exception(exception);
                }

                return iterator{};
            }
        }

        return iterator{handle_};
    }

    std::default_sentinel_t end() const noexcept {
        return {};
    }

private:
    Handle handle_{};
};

Generator generate_numbers() {
    co_yield 10;
    co_yield 20;
    co_yield 30;
}

#include <iostream>

int main() {
    for (int value : generate_numbers()) {
        std::cout << value << '\n';
    }
}
```

### **Analyzing the Generator Components**

In this implementation, `promise_type` defines the Coroutine's contract. The `get_return_object()` function creates a `Generator` object and provides it with the Handle associated with the Coroutine.

The `initial_suspend()` function uses `std::suspend_always`. Therefore, the Coroutine does not execute when it is created; it begins executing when `begin()` is first requested.

The `yield_value()` function stores the generated value in `current_value` and suspends Coroutine execution by returning `std::suspend_always`.

The Iterator calls `resume()` to ask the Coroutine to produce the next value. The Dereference operator provides the currently stored value from the Promise to the consumer.

The `Generator` type owns the Handle. For this reason, Copy is disabled and Move is implemented to transfer ownership. The Destructor also releases the Frame.

### **Limitations of the Educational Implementation**

This Generator is intentionally kept simple and is not sufficient for general-purpose use in a professional library. Its limitations include:

- It generates only `int` values and is not a Template.
- Its Iterator is a simple Input Iterator and does not cover all conceptual details and requirements of a general-purpose Range.
- It is not designed for multiple simultaneous Iterators or repeated traversal.
- Its Exception contract and behavior after completion are minimal.
- It is not designed for multi-Threaded use or control transfer between Schedulers.
- It does not support References, Move-only Values, custom Allocators, or Cancellation.

In a production implementation, Handle ownership, Iterator behavior, Exception handling, Reentrancy, Thread-safety, Frame allocation, and Cancellation must be designed and tested carefully.

## **Using Library Generators**

### **Standard Library Support Across Versions**

In C++20, the language provides the Coroutine infrastructure, but it does not provide a general-purpose standard type such as `std::generator` for direct use by programmers.

In C++23, `std::generator` was added to the Standard Library. This type is defined in the following Header:

```
#include <generator>
```

Therefore, the claim that a standard Generator was expected to be added in C++23 now refers to a standardized feature. However, practical availability depends on the Standard Library implementation and the level of support provided by the Toolchain.

### **Using std::generator in C++23**

A simple example:

```
#include <generator>
#include <iostream>

std::generator<int> generate_numbers() {
    co_yield 10;
    co_yield 20;
    co_yield 30;
}

int main() {
    for (int value : generate_numbers()) {
        std::cout << value << '\n';
    }
}
```

In this example, `std::generator<int>` is a Coroutine-based Range that produces values lazily. There is no need to store all values in a `std::vector` before traversal begins.

### **When Is std::generator a Suitable Choice?**

`std::generator` is suitable for sequential value generation, building Lazy Ranges, traversing computed data, and simplifying code that would otherwise require complex manually implemented Iterators.

However, `std::generator` is not a general-purpose Asynchronous Programming tool by itself. The standard Generator is designed to produce values in a traversal sequence. Asynchronous operations require a suitable Task or another Abstraction provided by a library and Scheduler.

## **Using Third-party Libraries**

### **Why Use Existing Libraries?**

Building a custom Coroutine type is technically possible, but implementing it correctly is more difficult than simply writing a few Promise functions. Managing Frame lifetime, Exceptions, Handles, suspension and resumption, Cancellation, and Thread-safety can introduce considerable complexity.

For this reason, real-world applications should generally use existing Standard Library capabilities or established libraries first. A custom Coroutine type should be implemented only when there is a specific need for it.

### **Examples of Libraries**

- **cppcoro**: A collection of Coroutine Abstractions such as Task, Generator, and related utilities. This project is known as `cppcoro`; its maintenance status, active branch, and compatibility with the intended Compiler should be checked before choosing it.
- **Coro**: A project hosted at `https://github.com/Quuxplusone/coro`, which is also referenced in the source slides. Its API and support status should be checked in the project's own documentation and repository.
- **Coroutine-based Asynchronous Libraries**: Some Frameworks and networking or I/O libraries provide their own Task and Awaitable Abstractions. Their selection should be based on their Scheduler, Cancellation model, Executor support, and Thread-safety contract.

### **Library Selection Considerations**

Before selecting a Coroutine library, consider the following:

- Compatibility with the Compiler and standard version being used.
- Maintenance status and documentation quality.
- Ownership model and lifetime of the Coroutine Frame.
- Exception propagation mechanism.
- Support for Cancellation and Timeout.
- Scheduler and Executor model.
- Thread-safety behavior and the ability to resume from different Threads.
- Integration with the I/O libraries used in the project.
- License terms and the cost of maintaining the dependency.

### **Difference Between Generator and Task**

A Generator is generally used to produce multiple values over time and provide them as a Range. A Task generally represents an operation that eventually produces a result or an Exception and may be used for Asynchronous operations.

These two Abstractions do not have the same purpose. A Generator is not necessarily Asynchronous, and a Task does not necessarily produce multiple sequential values.

## **Limitations and Design Considerations**

### **A Coroutine Is Not Necessarily Asynchronous**

A Coroutine can execute entirely on the same Thread as its caller. Suspension and resumption are control-flow capabilities; Asynchronous behavior depends on cooperation with an Awaitable, Scheduler, or I/O infrastructure.

### **A Coroutine Is Not Necessarily Faster Than a Function**

Coroutines are designed to simplify control flow and state management, not to guarantee increased performance. Creating and managing a Coroutine Frame, allocating memory, transferring control, and resuming execution can all have costs.

### **Variable Lifetime**

Variables that must remain alive after a Coroutine is suspended need to be preserved in the Coroutine's state. However, if a Coroutine refers to external data through a Pointer, Reference, or `std::string_view`, the Coroutine Frame does not guarantee the lifetime of that external data.

### **Ownership and Destruction**

Every design must clearly define which object owns the Coroutine Handle and when the Frame is released. A raw Handle should not be shared between multiple Owners without a clearly defined ownership contract.

### **Thread-related Risks**

Coroutines are not inherently Thread-safe. If an Awaitable resumes a Coroutine on another Thread, access to shared data and the state of involved objects must follow C++ concurrency rules.

### **Cancellation**

The C++20 Coroutine standard does not provide a complete, general-purpose Cancellation model. Cancellation must be designed by higher-level Abstractions, Awaitables, and the Scheduler. Simply destroying a Coroutine is not always equivalent to safely cancelling an external operation. For example, an I/O operation may still be running inside the operating system or a library.

### **Exception Safety**

An Exception can propagate from the Coroutine body to `unhandled_exception()`. The Coroutine type must define a clear policy for storing, propagating, or transforming Exceptions. In Asynchronous designs, it must also be clear where and on which Thread an Exception is rethrown or reported.

## **Common Mistakes**

- Assuming that `co_await` always causes a Coroutine to suspend.
- Assuming that every Coroutine executes on a separate Thread.
- Assuming that C++20 provides a ready-to-use `std::generator`.
- Confusing `co_yield` with `co_return`.
- Failing to release the Coroutine Frame or destroying it at the wrong time.
- Resuming a Coroutine that has already completed or whose Handle is invalid.
- Keeping a Reference to data that is destroyed before the next resumption.
- Assuming that the Promise Type is the same as the Coroutine Handle.
- Assuming that every Awaitable object necessarily performs an Asynchronous operation.
- Implementing a general-purpose Coroutine type without thoroughly testing lifetime, Exception handling, and ownership.

## **Differences Between Language Features and Standard Library Features**

| Feature                                        | Standard Version       | Description                                                                      |
| ---------------------------------------------- | ---------------------- | -------------------------------------------------------------------------------- |
| Coroutine Language Support                     | C++20                  | Includes `co_await`, `co_yield`, `co_return`, and Coroutine transformation rules |
| `std::coroutine_handle`                        | C++20                  | A low-level tool for managing the Coroutine Frame                                |
| `std::suspend_always` and `std::suspend_never` | C++20                  | Ready-to-use Awaiters for suspending or not suspending execution                 |
| `std::generator`                               | C++23                  | A standard Generator for Lazy value generation                                   |
| General-purpose standard Task                  | Not available in C++20 | A library or custom Abstraction is usually used                                  |

## **Summary**

Coroutines in C++20 make it possible to suspend and resume function execution. They provide the foundation for building Generators, Tasks, and Asynchronous Abstractions.

The three main Keywords are `co_await`, which waits for an Awaitable; `co_yield`, which provides a value and usually suspends execution in Generators; and `co_return`, which terminates a Coroutine.

At the infrastructure level, the Promise Type defines the Coroutine's contract, the Coroutine Handle provides control over its Frame, and the Awaiter specifies the suspension and resumption behavior of `co_await`.

For everyday use, it is generally better to rely on existing, well-tested Abstractions whenever possible. In C++23, `std::generator` can be used for Lazy value generation. However, for Tasks and Asynchronous I/O, a suitable Abstraction from the library or Framework being used must still be selected.

## **🤝 Contributions**

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>