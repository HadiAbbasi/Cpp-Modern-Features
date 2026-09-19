<div align="right">

[🇺🇸 English](./atomic.md) | [🇮🇷 فارسی](../../fa/cpp11/bind.md)

</div>

# Atomic

The <atomic> header, introduced in C++11, provides atomic types and operations for safely accessing shared data in multithreaded programs. It helps prevent data races by ensuring that operations on shared variables are performed atomically.

- Provides lock-free synchronization for many primitive data types.
- Enables safe communication between multiple threads without explicit mutexes.

```cpp
std::atomic<type> variable_name;
```
**Parameters**

- data_type: Primitive type such as int, bool, char, etc.
- variable_name: Name of the atomic variable.

## Common Atomic Operations
The `<atomic>` header provides several operations for reading, modifying, and synchronizing atomic variables.
| Function | Description |
| :--- | :--- |
| `load()` | Reads the current value of the atomic variable. |
| `store()` | Stores a new value into the atomic variable. |
| `exchange()` | Replaces the current value and returns the previous value. |
| `fetch_add()` | Atomically adds a value and returns the old value. |
| `fetch_sub()` | Atomically subtracts a value and returns the old value. |
| `fetch_and()` | Performs an atomic bitwise AND operation. |
| `fetch_or()` | Performs an atomic bitwise OR operation. |
| `fetch_xor()` | Performs an atomic bitwise XOR operation. |
| `wait()` | Blocks until the atomic value changes. (C++20) |
| `notify_one()` | Wakes one waiting thread. (C++20) |
| `notify_all()` | Wakes all waiting threads. (C++20) |

### Example

- Two threads increment the same atomic counter.
- fetch_add() performs each increment atomically.
- The final result is correct because no data races occur.
```cpp
#include <atomic>
#include <iostream>
#include <thread>
using namespace std;

atomic<int> counter(0); // Atomic integer

void increment_counter(int id)
{
    for (int i = 0; i < 100000; ++i) {
        // Increment counter atomically
        counter.fetch_add(1);
    }
}

int main()
{
    thread t1(increment_counter, 1);
    thread t2(increment_counter, 2);

    t1.join();
    t2.join();

    cout << "Counter: " << counter.load() << std::endl;

    return 0;
}
```

## std::atomic_flag
std::atomic_flag is the simplest atomic type provided by the <atomic> header. It represents a lock-free boolean flag and is commonly used to implement lightweight synchronization mechanisms such as spinlocks. It provides the following operations:

| Function | Description |
| :--- | :--- |
| `test()` | Reads the current flag value. (C++20) |
| `test_and_set()` | Sets the flag and returns its previous value. |
| `clear()` | Resets the flag. |

### example
- test_and_set() atomically acquires the flag.
- Only one thread enters the critical section at a time.
- clear() releases the flag so another thread can proceed.

```cpp
#include <atomic>
#include <iostream>
#include <thread>

using namespace std;

atomic_flag flag = ATOMIC_FLAG_INIT;

void work(int id)
{
    while (flag.test_and_set(memory_order_acquire))
        ;

    cout << "Thread " << id << " is running\n";

    flag.clear(memory_order_release);
}

int main()
{
    thread t1(work, 1);
    thread t2(work, 2);

    t1.join();
    t2.join();

    return 0;
}
```


## Advantages of atomic

The <atomic> header offers several benefits:

- Prevents data races in multithreaded programs.
- Supports efficient lock-free programming on many platforms.
- Improves performance for lightweight shared data access.


## Limitation of atomin
Despite its advantages, <atomic> has some limitations:

- Works primarily with individual variables, not complex operations.
- Incorrect memory ordering can still introduce synchronization bugs.
- Complex shared resources often still require mutexes.


---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>
