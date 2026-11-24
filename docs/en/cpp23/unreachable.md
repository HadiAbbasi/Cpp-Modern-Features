<div align="center">

[🇺🇸 English](./unreachable.md) | [🇮🇷 فارسی](../../fa/cpp23/unreachable.md)

</div>

#  std::unreachable

The **`std::unreachable()`** function, introduced in **C++23** and defined in the `<utility>` header, is a powerful optimization tool. 

It informs the compiler that **"execution should never reach this point of the code."** If execution *does* reach a call to `std::unreachable()`, the behavior is **Undefined Behavior (UB)**.


## Why Do We Need `std::unreachable()`?

Prior to C++23, to tell the compiler that a path was unreachable, developers had to rely on compiler-specific extensions:
- **GCC / Clang:** `__builtin_unreachable()`
- **MSVC:** `__assume(false)`

C++23 standardizes this mechanism, making code portable across different compilers.


##  Main Use Cases

### A) Optimizing `switch` Statements (Removing Redundant Checks)
Suppose you have an `enum` and you are certain that all valid cases are covered:

```cpp
#include <utility>
#include <iostream>

enum class Direction {
    North,
    South,
    East,
    West
};

const char* get_direction_name(Direction dir) {
    switch (dir) {
        case Direction::North: return "North";
        case Direction::South: return "South";
        case Direction::East:  return "East";
        case Direction::West:  return "West";
    }
    // Tells the compiler no other values are possible
    std::unreachable();
}
```

**Benefit:**
The compiler doesn't need to generate a fallback branch, default warning handlers, or dummy return statements, resulting in cleaner and more efficient assembly code.



### B) Mathematical and Algorithmic Optimizations
When you logically know certain inputs or states are impossible:

```cpp
#include <utility>

int divide_by_power_of_two(int x, int p) {
    if (p <= 0 || p > 30) {
        std::unreachable(); // Guarantee: 'p' is always between 1 and 30
    }
    return x / (1 << p);
}
```
With this hint, the compiler can perform deeper low-level optimizations.


##  Difference from `assert`

| Feature | `assert(...)` | `std::unreachable()` |
| :--- | :--- | :--- |
| **Primary Purpose** | Debugging and sanity checking | Compiler optimization hint |
| **Behavior in Release Mode** | Usually disabled (when `-DNDEBUG` is defined) | Always active; the compiler relies on it for optimization |
| **On Violation** | Terminate the program safely (Abort) | Results in **Undefined Behavior** (can cause crashes, corruption, or unpredictable results) |

> 💡 **Best Practice:**
> In debug builds, you can combine both to catch logic bugs early, while letting the compiler optimize safely:
> ```cpp
> #include <cassert>
> #include <utility>
> 
> void do_something(int x) {
>     if (x < 0) {
>         assert(false && "x must be non-negative");
>         std::unreachable();
>     }
>     // ...
> }
> ```


## Summary
- **Header:** `#include <utility>`
- **Role:** Explicitly guides the compiler to prune impossible branches and optimize performance/size.
- **Warning:** Use it **only** when you are 100% certain execution can never reach that line; otherwise, you risk Undefined Behavior.


---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>