<div align="right">

[🇺🇸 English](./ref.md) | [🇮🇷 فارسی](../../fa/cpp11/ref.md)

</div>

# std::ref
The ‍‍‍‍‍‍‍‍‍‍‍‍‍`std::ref` is a powerful utility in C++ that allows you to pass references to objects in contexts where copies are typically expected, such as with function templates of standard algorithms.

## What problem does it solve ?
The `std::ref` is particularly useful when you want to pass a callback function that takes a reference as an argument, especially in scenarios where the callback mechanism or the function invocation system (e.g., STL algorithms, threads, or std::bind) would otherwise pass the argument by value.

Many template functions and classes in modern C++ (such as `std::thread`, `std::bind`, and `std::make_pair`) receive and store their arguments by value (pass-by-value).

Even if your target function expects an argument by reference (&), these constructs (like `std::thread` or `std::bind`) will create a copy of the variable in the intermediate layer. Consequently:

- Compilation Errors: If your object is non-copyable (e.g., `std::unique_ptr` or `std::mutex`), you will get a compilation error.
- Modification of Copies: If you modify the variable, changes are applied to the copy rather than the original object.
- Pointer vs. Reference Confusion: Using the standard address-of operator (&) in these contexts results in a raw pointer, not a true reference.

This is where std::ref comes in. It wraps your variable into a small, copyable object (a reference_wrapper) that behaves like a reference, allowing you to bypass the default pass-by-value behavior of these templates.


## applications of std::ref

### 1. Passing by Reference to std::thread (Most common use case)
The std::thread constructor copies and stores arguments by value. If you want a thread to modify your original variable, you must use std::ref.
```cpp
#include <iostream>
#include <thread>
#include <functional> // Required for std::ref

void increment(int& n) {
    n++;
}

int main() {
    int x = 10;
    std::thread t(increment, std::ref(x));
    t.join();

    std::cout << x << '\n'; // Prints: 11
}
```
### 2. Usage in std::bind
By default, std::bind copies the arguments it binds.
```cpp
#include <iostream>
#include <functional>

void update(int& n) {
    n += 5;
}

int main() {
    int val = 20;

    // Without std::ref, 'val' would be copied, and the original variable would remain unchanged.
    auto bound_fn = std::bind(update, std::ref(val));
    bound_fn();

    std::cout << val << '\n'; // Prints: 25
}
```
### 3. Storing References in STL Containers (e.g., std::vector)
In C++, you cannot create a vector of references directly (std::vector<int&> causes a compilation error because references are not objects and cannot be copied or moved). However, you can create a vector of reference_wrapper objects:

```cpp
#include <iostream>
#include <vector>
#include <functional>

int main() {
    int a = 1, b = 2, c = 3;
    std::vector<std::reference_wrapper<int>> vec = { std::ref(a), std::ref(b), std::ref(c) };

    for (int& item : vec) {
        item *= 10;
    }

    std::cout << a << ", " << b << ", " << c << '\n'; // 10, 20, 30
}
```

## Difference between std::ref and std::cref
- std::ref(x): Used to pass a mutable reference (T&).
- std::cref(x): Used to pass a read-only (const) reference (const T&). This is useful for avoiding the copying of large objects in functions that only need to read data.


## Summary
- std::ref is a tool used to force templates and functions that default to copying objects to instead hold and use a reference. Its most common applications are when working with threads (std::thread) and binders (std::bind).

---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>
