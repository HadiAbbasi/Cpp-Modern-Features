<div align="right">

[🇺🇸 English](./promise.md) | [🇮🇷 فارسی](../../fa/cpp11/promise.md)

</div>

# Promise
In C++ multithreading, a thread is the basic unit that can be executed within a process and to communicate two or more threads with each other, std::promise in conjunction with std::future can be used.
```text
+-------------------+                          +------------------+
| std::promise<T>   |                          | std::future<T>   |
| (Worker/Producer) |                          | (Main/Consumer)  |
+---------+---------+                          +--------+---------+
          |                                             |
          |  set_value(data)                            |  get() (blocks)
          v                                             v
    +---------------------------------------------------------+
    |                    Shared State (Heap)                  |
    |  - Value / Exception storage (std::aligned_storage/any) |
    |  - Mutex & Condition Variable (internal sync)           |
    |  - Ready / Abandoned Flags                              |
    +---------------------------------------------------------+

```

## What is std:: promise in C++

std::promise is a class template that is used with std::future class and it promises to set the value of the std::future object that can be accessed in another thread. The main functionality of this method is to provide a method for one thread to fulfill a promise (set a value or an exception), and another thread to retrieve that value or exception at a later point in time thereby defining the future references.

It is generally helpful in producer-consumer type problems.

## How to Use std::promise
To use std::promise include the <future> header and follow the below steps:

- 1. Create a std::promise Object: Define an object of type promise that will hold the value that will be set in the future.
- 2. Create a std::future Object: Create a future object that will be associated with our promise using the get_future() member function and it will be used to access the value once it's set.
- 3. Set the Value in the std::promise: From a producer thread, call promise by setting the corresponding values with the promise we can set a value using set_value()or set exception thrown using set_exception().
- 4. Retrieving Value from the std::future: Use std::future to get a value from a consumer thread using the get() function of future to get the values associated or get the error associated.

## Example
The below example demonstrates the use of "std::promise" and "std::future" to communicate between threads.

```cpp
#include <future>
#include <iostream>
#include <stdexcept>
#include <thread>

using namespace std;
void RetrieveValue(promise<int>& result)
{
    try {
        int ans = 21095022;
        result.set_value(ans);
    }
    catch (...) {
        result.set_exception(current_exception());
    }
}

int main()
{
    promise<int> myPromise;
    future<int> myFuture = myPromise.get_future();
    thread computationThread(RetrieveValue, ref(myPromise));
    try {
        int result = myFuture.get();
        cout << "Result: " << result << endl;
    }
    catch (const exception& e) {
        cerr << "Exception is: " << e.what() << endl;
    }
    computationThread.join();

    return 0;
}
```

## Low-Level and Performance Considerations

### Move-Only Semantics
std::promise is not copyable, but it is movable:

```cpp
std::promise<int> p1;

// Error: copying is not allowed
// std::promise<int> p2 = p1;

// Correct: transfer ownership
std::promise<int> p2 = std::move(p1);

```
This guarantees that there is only one active producer responsible for satisfying the shared state.

After moving from p1, it no longer owns a shared state. Calling operations such as p1.set_value() will throw std::future_error with:
```cpp
std::future_errc::no_state
```

### `get_future()` Can Be Called Only Once

Each std::promise has exactly one associated std::future. Therefore, get_future() can be called only once:
```cpp
std::promise<int> promise;
std::future<int> future1 = promise.get_future();
std::future<int> future2 = promise.get_future();
```
The error code is:
```cpp
std::future_errc::future_already_retrieved

```
If multiple consumers need to observe the same result, convert the future to `std::shared_future`:
```cpp
std::promise<int> promise;

std::shared_future<int> sharedFuture =
    promise.get_future().share();

```
A std::shared_future is copyable, and multiple threads can safely wait for and read the same result.

### A Promise Can Be Satisfied Only Once

A promise can receive either:
- one value through set or(), or
- one exception through set_exception().

It cannot be satisfied more than once:
```cpp
std::promise<int> promise;
auto future = promise.get_future();

promise.set_value(42);

// Throws std::future_error
promise.set_value(100);

```
The same rule applies when mixing set_value() and set_exception():

```cpp
promise.set_value(42);

// Also throws std::future_error
promise.set_exception(
    std::make_exception_ptr(
        std::runtime_error("Something failed")
    )
);

```
The corresponding error code is:

```cpp
std::future_errc::promise_already_satisfied

```

## Conclusion

In conclusion, concurrent programming can be done in C++ using "std::promise" in C++. This along with "std::future" is part of the broader C++11/14 concurrency features. This is a useful and simple method that helps in data transfer and also allows us to handle exceptions.

---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>