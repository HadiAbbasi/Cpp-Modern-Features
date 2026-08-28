<div align="right">

[🇺🇸 English](./promise.md) | [🇮🇷 فارسی](../../fa/cpp11/promise.md)

</div>

# Promise
In C++ multithreading, a thread is the basic unit that can be executed within a process and to communicate two or more threads with each other, std::promise in conjunction with std::future can be used.

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
// C++ program to use std::promise and std::future to
// communicate between threads.
#include <future>
#include <iostream>
#include <stdexcept>
#include <thread>

using namespace std;

// Function to perform some computation and set the result
// in a promise
void RetrieveValue(promise<int>& result)
{
    try {
        int ans = 21095022;

        // Set the result in the promise
        result.set_value(ans);
    }
    catch (...) {
        // if an error occurs set the exception
        result.set_exception(current_exception());
    }
}

int main()
{
    // Step 1: Creating a std::promise object
    promise<int> myPromise;

    // Step 2: Associate a std::future with the promise
    future<int> myFuture = myPromise.get_future();

    // Step 3: Launching a thread to perform computation and
    // set the result in the promise
    thread computationThread(RetrieveValue, ref(myPromise));

    // Step 4: Retrieve the value or handle the exception in
    // the original thread
    try {
        int result = myFuture.get();
        cout << "Result: " << result << endl;
    }
    catch (const exception& e) {
        cerr << "Exception is: " << e.what() << endl;
    }

    // thread finishes
    computationThread.join();

    return 0;
}
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