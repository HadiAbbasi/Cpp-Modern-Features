<div align="right">

[🇺🇸 English](./thread.md) | [🇮🇷 فارسی](../../fa/cpp11/thread.md)

</div>

# Chrono

 >provides classes and functions for measuring time, representing durations, and working with clocks. It offers a portable and precise way to perform time-related operations in C++ programs.
>- Provides clocks, durations, and time points for time measurement.
>- Supports high-precision and platform-independent time operati

## Which Clock Should You Use
- `std::chrono::steady_clock:` ALWAYS use this for measuring elapsed time (stopwatch). It is monotonic, meaning it can never go backwards (even if the OS changes the system time).
- `std::chrono::system_clock:` Use this for "Wall Clock" time (e.g., getting the current date/time to display to a user). It syncs with the system time and can jump forward/backward.
- `std::chrono::high_resolution_clock:` Usually just an alias for one of the above. Stick to steady_clock for robustness.

```cpp
#include <chrono>
#include <iostream>

using namespace std;
using namespace std::chrono;

int main() {
    auto start = high_resolution_clock::now();

    // Code to measure
    for (int i = 0; i < 1000000; i++);

    auto end = high_resolution_clock::now();

    auto duration = duration_cast<microseconds>(end - start);

    cout << "Execution Time: "
         << duration.count()
         << " microseconds";
}
```
**Explanation:** The program records the start and end time of a code segment and calculates the elapsed time using the <chrono> library.

## Main Components of <chrono>
The <chrono> library is built around three fundamental concepts:

### Duration
A duration represents the amount of time between two events. It stores both the numeric value and the unit of time.

- Represented using std::chrono::duration.
- Can store seconds, milliseconds, microseconds, nanoseconds, and more.
- Supports arithmetic operations such as addition and subtraction.
- The count() function returns the stored value. 

```cpp
#include <chrono>
#include <iostream>
using namespace std;
using namespace std::chrono;

int main()
{
    seconds s(5);
    milliseconds ms = duration_cast<milliseconds>(s);

    cout << "Seconds: " << s.count() << endl;
    cout << "Milliseconds: " << ms.count() << endl;

    return 0;
}
```

### clock
A clock provides the current time and acts as the reference for measuring durations and creating time points. C++ provides three standard clock types:

| Clock Description | Description |
|---|---|
| `system_clock` | Represents the system's real-time clock. |
| `steady_clock` | Monotonic clock that is never adjusted and is ideal for measuring elapsed time. |
| `high_resolution_clock` | Provides the smallest available tick period for high-precision measurements. |


### TimePoint 
A time point represents a specific moment in time relative to the epoch of a clock.

- Represented using std::chrono::time_point.
- Obtained using the now() function of a clock.
- Can be subtracted to calculate elapsed time.
- Frequently used for benchmarking program execution

```cpp
#include <chrono>
#include <iostream>
#include <thread>
using namespace std;
using namespace std::chrono;

int main()
{
    time_point<steady_clock> start = steady_clock::now();

    // Simulate some work
    this_thread::sleep_for(seconds(2));

    time_point<steady_clock> end = steady_clock::now();

    auto elapsed = duration_cast<milliseconds>(end - start);

    cout << "Elapsed Time: "
         << elapsed.count() << " ms";

    return 0;
}
```
 ### Common Duration Types

The <chrono> library provides several predefined duration types.

 | Duration Type | Time Unit |
|---|---|
| `nanoseconds` | 10⁻⁹ second |
| `microseconds` | 10⁻⁶ second |
| `milliseconds` | 10⁻³ second |
| `seconds` | 1 second |
| `minutes` | 60 seconds |
| `hours` | 60 minutes |


## Example 

### Measuring Execution Time (Stopwatch)
```cpp
#include <iostream>
#include <chrono>
#include <thread>

int main() {
    // 1. Start the timer
    auto start = std::chrono::steady_clock::now();

    // ... Do some work (simulated here by sleeping) ...
    std::this_thread::sleep_for(std::chrono::milliseconds(500));

    // 2. Stop the timer
    auto end = std::chrono::steady_clock::now();

    // 3. Calculate duration
    // You must cast it to the unit you want (e.g., milliseconds, microseconds)
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << "Time elapsed: " << duration.count() << " ms" << std::endl;

    return 0;
}
```
### Getting Current Date/Time
```cpp
#include <iostream>
#include <chrono>
#include <ctime>   // For std::time_t

int main() {
    // 1. Get current time point
    auto now = std::chrono::system_clock::now();

    // 2. Convert to time_t (C-style time) to print it easily
    std::time_t currentTime = std::chrono::system_clock::to_time_t(now);

    // 3. Print
    // ctime converts time_t to a string like "Wed Dec 31 17:00:00 2025\n"
    std::cout << "Current time: " << std::ctime(&currentTime); 

    return 0;
}
```


## Advantages of <chrono>

The <chrono> library provides several benefits:

- Works consistently across different platforms.
- Separates clocks, durations, and time points for better flexibility.
- Simplifies benchmarking and performance analysis. 

## Limitations of <chrono>

Despite its advantages, <chrono> has some limitations:

- Clock precision depends on the underlying operating system.
- Different clocks serve different purposes and should be chosen carefully.
- Some advanced time-zone features require newer C++ standards.

---
## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>