<div align="right">

[🇺🇸 English](./thread.md) | [🇮🇷 فارسی](../../fa/cpp11/thread.md)

</div>

# async
>It’s one of those C++ tools that feels small at first, but once you start using it, your code becomes cleaner, and naturally asynchronous. Whether you’re loading resources in the background, offloading heavy calculations, or simply keeping your UI responsive, std::async offers a clean, high-level way to execute work without blocking the main flow of your program.

>Asynchronous programming has become a fundamental part of modern application development, especially with the need to execute long-running tasks without blocking the user interface or halting the program. In languages like JavaScript, the Async/Await pattern is used to simplify writing asynchronous code, making it appear synchronous. But how can this pattern be achieved in a language like C++? In this article, we will explore how to represent asynchronous programming in C++ using Async/Await and how this approach can be useful in certain scenarios.


## why async
>Asynchronous programming allows long-running tasks (such as network requests or file access) to be executed without blocking the main program. Instead of waiting for the task to complete, the program can continue executing other tasks and then return to process the result once the asynchronous task is finished.

>Let’s imagine a simple scenario, you’re building a weather application that shows real-time conditions. The app retrieves all its data from a third-party API. Now, if you make this API call inside your main thread (your event loop), everything else in the application must wait until that request finishes and the duration of that wait is completely unknown. During that time, the UI freezes, user interactions lag, and the whole app feels unresponsive.

>This is where std::async steps in as a clean and elegant solution. Instead of blocking the main thread, we can offload the API call to a separate asynchronous task. The main thread keeps running smoothly, handling UI updates and user interactions, while the background task quietly fetches the weather data.

Let’s look at a very simple weather application. The app retrieves real-time weather data from a third-party API. But here’s the catch: API calls are slow and unpredictable. To simulate this, I added an artificial 4-second delay in the code. Here, I have used an open-source, header-only library to make the HTTP request .

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


---
## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>