<div align="right">

[🇺🇸 English](./future.md) | [🇮🇷 فارسی](../../fa/cpp11/async.md)

</div>

# future

## std::future
`std::future` is a template class in C++ representing the result (value or exception) of an asynchronous operation that will become available in the future. It is used to obtain the result of a task executed in a separate thread or via `std::async`.

### feature

- **Asynchrony**: Allows performing an operation without blocking the current thread.
- **Getting the result**: The `get()` method allows retrieving the result. If the result is not ready, get() will block the current thread until it is ready or an exception is thrown.
- **Data transfer**: It is linked with a `std::promise` object, which provides a way to set the result or exception.
- **State**: `std::future` has a state indicating whether the result is ready. You can check this with wait() or wait_for().
- **Single retrieval**: The `get()` method can only be called once for each `std::future` object.
- **Result type**: It is typed with the return type of the asynchronous operation.

## What is std::future in C++
In C++, the std::future is a class template used to receive the result of an asynchronously executed task that is not computed yet i.e. future value. Asynchronous operations in C++ are managed using std::async, std::packaged_task, or std::promise, which returns a std::future object. We can use this std::future object to verify, await, or obtain the outcome of the operation.

It is to be noted that we can receive the result of the computation only once in the program as the state of the future is reset after getting the result.

## syntax std::future
The syntax to define a std::future object is pretty straightforward:
```cpp
std::future<type> name;
```
where:
- **name**: name of the future object.
- **type**: type of the data that is to be recieved.

## Member Function
The std::future class consists of the various member functions providing basic functionality. Some of the common ones are:

| S. No. | Function | Description |
|---:|---|---|
| 1 | **`get()`** | Function to get the value received. |
| 2 | **`wait()`** | This function tells the compiler to wait for the process to be done. |
| 3 | **`wait_for()`** | This function tells the compiler to wait for the specified time duration for the process to be finished. |
| 4 | **`wait_until()`** | This function tells the compiler to wait till the specified time duration for the process to be finished. |
| 5 | **`valid()`** | This function checks if the `future()` object has a shared state, i.e., whether it is valid to perform the `get()` operation. |

## Examples

### Using std::future to Print the Value Returned by Asynchronous Task
```cpp
#include <chrono>
#include <future>
#include <iostream>
using namespace std;

int returnTwo() { return 2; }

int main()
{
    future<int> f = async(launch::async, returnTwo);
    cout << f.get();
    return 0;
}
```

### Trying to Get the Value Multiple Times from std::future
```cpp
#include <chrono>
#include <future>
#include <iostream>
using namespace std;

int returnTwo() { return 2; }

int main()
{
    future<int> f = async(launch::async, returnTwo);
    cout << f.get();
    cout << f.get();
    return 0;
}
```
**NOTE**:Error
>terminate called after throwing an instance of 'std::future_error'what():  std::future_error: No associated state
The solution to the above problem is to use the future::valid() function to check the state of the future object before using get() method.

### Avoid the No Associate State Error using valid()

```cpp
// C++ Program to illustrate the use of std::future
#include <chrono>
#include <future>
#include <iostream>
using namespace std;

int returnTwo() { return 2; }

int main()
{
    future<int> f = async(launch::async, returnTwo);

    if (f.valid()) {
        cout << f.get() << endl;
    }
    else {
        cout << "Invalid State, Please create another Task"
             << endl;
    }
    if (f.valid()) {
        cout << f.get() << endl;
    }
    else {
        cout << "Invalid State, Please create another Task"
             << endl;
    }

    return 0;
}
```
Apart from the `async()` method, we can also get future objects using `std::packaged_task()` and `std::promise()`. The below example demonstrates the use of `std::future` with `std::promise`.
### Using std::future with std::promise

```cpp
#include <chrono>
#include <future>
#include <iostream>
using namespace std;

void foo(promise<int> p) { p.set_value(25); }

int main()
{
    promise<int> p;
    future<int> f = p.get_future();
    thread t(foo, move(p));

    t.join();
    cout << f.get();

    return 0;
}
```


## Conclusion
The std::future provides the programmers with the method for thread communication in asynchronous programming in C++ in an easy way. It is especially helpful in cases where we need to do some tasks in the background and need the result of that task in the main process.

---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>