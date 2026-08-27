<div align="right">

[🇺🇸 English](./thread.md) | [🇮🇷 فارسی](../../fa/cpp11/thread.md)

</div>

# async
>It’s one of those C++ tools that feels small at first, but once you start using it, your code becomes cleaner, and naturally asynchronous. Whether you’re loading resources in the background, offloading heavy calculations, or simply keeping your UI responsive, std::async offers a clean, high-level way to execute work without blocking the main flow of your program.

>Asynchronous programming has become a fundamental part of modern application development, especially with the need to execute long-running tasks without blocking the user interface or halting the program. In languages like JavaScript, the Async/Await pattern is used to simplify writing asynchronous code, making it appear synchronous. But how can this pattern be achieved in a language like C++? In this article, we will explore how to represent asynchronous programming in C++ using Async/Await and how this approach can be useful in certain scenarios.


## why async

Not every concurrency problem is "start a thread and manage it forever." A lot of tasks are closer to this:

- run some expensive work
- continue doing something else
- collect the answer later
- handle failure cleanly

That is exactly what futures are good at.
**Examples:**

- parsing a large config file while the UI continues
- loading assets in the background
- dispatching independent computations in a build tool
- running multiple independent benchmark stages and collecting results later

## Core Cpp Modern Async/Await
Modern C++ provides a small set of types around asynchronous result handling:

- `std::async`: starts asynchronous work and returns a future
- `std::future<T>`: represents a result that will become available later [Learn future](./future.md)
- `std::promise<T>: lets one thread produce a result for another thread
- `std::shared_future<T>`: allows multiple consumers to observe the same result

The main idea is simple: one side produces, another side waits or retrieves.

## std::async
The simplest entry point is std::async.
```cpp
#include <future>
#include <iostream>

int compute() {
    return 42;
}

int main() {
    std::future<int> result = std::async(std::launch::async, compute);
    std::cout << result.get() << '\n';
}
```
Important detail: std::async can use different launch policies.

- std::launch::async: run on a separate thread
- std::launch::deferred: delay execution until someone waits or calls get()
- default policy: the implementation may choose either

If you care about predictable behavior, specify the policy explicitly.

```cpp
auto future_value = std::async(std::launch::async, [] {
    return 10 * 20;
});
```
```cpp
#include <chrono>
#include <iostream>
#include <string>
#include <thread>
#include "httplib.h" 

std::string getWeatherData()
{
    httplib::Client cli("http://api.open-meteo.com");

    std::string apiBody;
    if (auto res = cli.Get("/v1/forecast?latitude=52.52&longitude=13.41&current=temperature_2m")) {
        apiBody = res->body;
    }

    std::this_thread::sleep_for(std::chrono::seconds(4));

    return apiBody;
}

int main() {

    while (true) {
        std::cout << "Fetching weather data....\n";

        std::string currentWeather = getWeatherData();

        std::cout << "Loading UI ...\n";
        std::cout << "\nCurrent weather is : " << currentWeather << "\n";

        std::this_thread::sleep_for(std::chrono::seconds(1));
    }

    return 0;
}
```
You can clearly see it: every cycle waits ~4 seconds before doing anything else. And in real applications, API calls can take even longer depending on network speed.

## Conclusion
`std::async` is a high-level thread abstraction tool in the C++ standard library that simplifies the implementation of asynchronous operations and makes code more concise. However, due to differences between compiler implementations, developers need to carefully consider these factors when using it to avoid potential issues. Special attention should be paid to `thread_local` variables and the returned `std::future`.

---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>
