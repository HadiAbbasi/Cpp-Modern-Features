<div align="right">

[🇺🇸 English](./thread_local.md) | [🇮🇷 فارسی](../../fa/cpp11/thread_local.md)

</div>

# thread_local
Thread local storage (TLS) is a feature introduced in C++ 11 that allows each thread in a multi-threaded program to have its own separate instance of a variable. In simple words, we can say that each thread can have its own independent instance of a variable. Each thread can access and modify its own copy of the variable without interfering with other threads.

 When you declare a variable as thread_local, every thread in your program gets its own independent copy of that variable. No sharing. No need of synchronization. Each thread sees only its own instance.

```cpp
thread_local int counter = 0;
void do_work() {
    ++counter;  // Thread A increments its own counter.
                // Thread B increments its own counter.
                // They never see each other's value.
}
```
thread_local works best when the data is mutable, reused, and thread-confined. If all three are true, it's probably the right tool.



## Properties of Thread Local Storage (TLS)

- **Lifetime**: The lifetime of a TLS variable begins when it is initialized and ends when the thread terminates.
- **Visibility**: TLS variables have visibility at the thread level.
- **Scope**: TLS variables have scope depending on where they are declared


## How It Differs From Other Storage Durations

| Storage Duration | Lifetime | Shared Across Threads? | Thread-Safe? |
|---|---|---|---|
| **automatic** (local variable) | Scope (stack) | No | Yes (private) |
| **static** | Program lifetime | **YES** (single copy) | **NO** (races) |
| **dynamic** (new/delete) | Manual (new → delete) | Depends on who holds ptr | Depends |
| **thread_local** | Thread lifetime (1st access → exit) | No (per-thread) | Yes (private) |


thread_local gives you the persistence of static (survives across function calls) with the isolation of a local variable (no other thread can touch it).



## Key Characteristics of thread_local

### One instance per thread, created on first access
```cpp
thread_local std::vector<int> cache;

void process() {
    // First time this thread calls process(): cache is constructed.
    // Second time: cache already exists. Same instance. No construction.
    cache.push_back(42);
}
```

### Destroyed when the owning thread exits.
When a thread terminates, all its thread_local variables are destroyed — destructors run, memory is freed. You don't manage the lifetime manually.

### No synchronization needed.
Since no two threads ever touch the same instance, you never need a mutex, an atomic, or any locking around a thread_local variable. The isolation is guaranteed by the language.

### Access is fast — typically one CPU instruction on x86–64

On Linux/x86–64, the OS maintains a Thread Local Storage (TLS) block per thread, accessible through the %fs segment register. Accessing a thread_local variable compiles down to something like:
```cpp
mov rax, fs:[offset] ; one instruction, no syscall, no lock
```

This is roughly the same cost as accessing a global variable — essentially free.


## Why use it?

```cpp
// Per-thread ID
thread_local int thread_id = -1;

// Per-thread error state (like errno)
thread_local std::string last_error;

// Per-thread random generator
thread_local std::mt19937 rng(std::random_device{}());
```

These are fine. But they’re small. The real power of thread_local shows up when you use it to cache expensive mutable objects in hot paths — objects that would otherwise be allocated and destroyed millions of times, or shared behind a mutex.


## Examples of thread_local Storage

**Examples1**

```cpp
#include <iostream>
#include <thread>
using namespace std;

thread_local int counter = 0;

void increment_counter()
{
    counter++;
    cout << "Thread " << this_thread::get_id()
         << " counter = " << counter << endl;
}

int main()
{
    thread t1(increment_counter);
    thread t2(increment_counter);
    t1.join();
    t2.join();
    return 0;
}

```
**Examples**
```cpp
#include <iostream>
#include <thread>

using namespace std;

class Singleton {
public:
    static Singleton& getInstance()
    {
        thread_local Singleton instance;
        return instance;
    }

    void printMessage()
    {
        cout << "Hello from thread "
             << this_thread::get_id() << endl;
    }

private:
    Singleton() = default;
};

void workerThread()
{
    Singleton::getInstance().printMessage();
}

int main()
{
    thread t1(workerThread);
    thread t2(workerThread);

    t1.join();
    t2.join();
    return 0;
}
```
## Static Thread Local Storage
It allows each thread to have its own separate instance of a static variable. Similar to TLS, each thread can access and modify its own copy of the static variable without affecting other threads but with static storage duration, which means the variable persists across multiple invocations of the function within the same thread.


```cpp
#include <iostream>
#include <thread>

using namespace std;

void thread_func()
{
    // Static thread-local variable
    static thread_local int stls_variable = 0;
    // Increment the variable
    stls_variable += 1;

    cout << "Thread ID: " << this_thread::get_id()
              << ", Variable: " << stls_variable
              << endl;
}

int main()
{
    thread t1(thread_func);
    thread t2(thread_func);

    t1.join();
    t2.join();

    return 0;
}
```


## Rules and limitations

- thread_local can not be used with function declarations or definitions. It can be used only for data declarations and definitions with static storage duration.
- If we declare any local variable with specifier thread_local, it is implicitly static. thread_local and static thread_local are equivalent if no other storage class is provided.
- We must use the thread_local specifier for both the declaration and the definition of a thread-local object. This applies to the case where the declaration and definition occur in the same file or separate files.

## Conclusion

Thread local storage (TLS) in C++ allows each thread to have its own separate instance of a variable. TLS variables have their own lifetime, visibility, and scope within each thread. TLS can be used to maintain thread-specific data without interfering with other threads.

---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>

