<div align="right">

[🇺🇸 English](./mutex.md) | [🇮🇷 فارسی](../../fa/cpp11/mutex.md)

</div>


# Mutex 

> **The mutex class is a synchronization primitive that can be used to protect shared data from being simultaneously accessed by multiple threads.**
> - Allows exclusive access to critical sections.
> - Ensures predictable behavior while accessing shared data.

**A mutex can only be in two states: locked or unlocked.**

- **Once a thread locks a mutex:**
- Other threads attempting to lock the same mutex are blocked.
- Only the thread that initially locked the mutex has the ability to unlock it.

- **This allows to protect regions of code.**

- **Typical mutex workflow:**
- Create and initialize a mutex variable
- Several threads attempt to lock the mutex
- Only one succeeds and that thread owns the mutex
- The owner thread performs some set of actions
- The owner unlocks the mutex
- Another thread acquires the mutex and repeats the process
- The mutext should be destroyed at the end.

![mutex](./assets/mutex.png)

## API for Cpp11
>`#include <mutex>`
> - Include the header file with mutex interface 

> `void mutex.lock()`
> - Locks a mutex; blocks if another thread has locked this mutex and owns it.

>  `void mutex.unlock()`
> - Unlocks mutex; after unlocking, other threads get a chance to lock the mutex.

> `bool mutex.try_lock()`
> - Tries to lock the mutex. Returns immediately. On successful lock acquisition returns true, otherwise returns false

![lock](./assets/lock.png)

## Unique lock - API
The mutexes are encapsulated by unique_lock classes, that simplify the usage, e.g. they automatically unlock the held mutex during their destruction (exceptions).

- `unique_lock unique_lock(mutex_type& m)`
Takes mutex m and and locks it
- `unique_lock unique_lock(mutex_type& m, std::defer_lock_t t)`
 Takes mutex m and and keeps it unlocked
- `unique_lock.lock()`
Locks the unique_lock
- `unique_lock.unlock()`
Unlocks the unique_lock