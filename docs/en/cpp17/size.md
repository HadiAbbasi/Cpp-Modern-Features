<div align="center">

[🇺🇸 English](./size.md) | [🇮🇷 فارسی](../../fa/cpp17/size.md)

</div>

# std::size

**What is `std::size`?**

The **`std::size`** function is another very useful utility function added to the `<iterator>` header in **C++17**. It returns the number of elements in a container or a raw array.


##  Why was it introduced?

Before C++17, getting the number of elements required different approaches:
* For standard containers (like `vector`, `string`, `map`): you used `c.size()`.
* For C-style raw arrays (like `int arr[10]`): you used the old C trick `sizeof(arr) / sizeof(arr[0])`, which is error-prone (for instance, if the array decays to a pointer when passed to a function, it yields an incorrect result).

This inconsistency made writing **Generic / Template code** difficult because you couldn't use a single unified way to get the size of both types. `std::size` solved this problem.


##  Definition and Signatures

To use it, you need to include the `<iterator>` header:
```cpp
#include <iterator>
```

It provides two main overloads:

1. **For Containers:**
   ```cpp
   template <class C>
   constexpr auto size(const C& c) -> decltype(c.size());
   ```
   Supports any container that has a `.size()` member function.

2. **For C-Style Raw Arrays:**
   ```cpp
   template <class T, std::size_t N>
   constexpr std::size_t size(const T (&array)[N]) noexcept;
   ```
   Directly returns the number of elements (`N`) without manual calculation.


## Code Examples

### A) Basic Usage

```cpp
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    // 1. C-style raw array
    int raw_arr[] = {10, 20, 30, 40, 50};
    std::cout << "Raw array size: " << std::size(raw_arr) << "\n"; // Output: 5

    // 2. Standard vector
    std::vector<std::string> names = {"Ali", "Reza", "Sara"};
    std::cout << "Vector size: " << std::size(names) << "\n";       // Output: 3

    return 0;
}
```


### B) Usage in Generic Code (Templates)

```cpp
#include <iostream>
#include <vector>
#include <array>
#include <iterator>

template <typename T>
void printCount(const T& collection) {
    // Works with any collection or raw array type
    std::cout << "Element count: " << std::size(collection) << "\n";
}

int main() {
    int arr[] = {1, 2, 3};
    std::vector<int> vec = {10, 20, 30, 40};
    std::array<int, 2> std_arr = {100, 200};

    printCount(arr);      // 3
    printCount(vec);      // 4
    printCount(std_arr);  // 2

    return 0;
}
```


## Preventing Array Decay Errors

In C/C++, if you pass an array to a function, it "decays" into a pointer, and the old `sizeof` trick breaks. However, `std::size` is **safe**: if you mistakenly pass a raw pointer instead of an array, the code **fails to compile** rather than producing silent, incorrect results:

```cpp
void badFunction(int* ptr) {
    // Compile-time error: raw pointer has no .size() member and 
    // the array bounds are unknown.
    // auto s = std::size(ptr); 
}
```


## Compile-time Support (`constexpr`)

For raw arrays and `std::array`, `std::size` is evaluated as `constexpr`, meaning you can use it at compile time to define other arrays or constants:

```cpp
constexpr int arr1[] = {1, 2, 3, 4};
constexpr auto len = std::size(arr1); // Computed at compile-time

int arr2[len]; // Perfectly valid
```


## Bonus: `std::ssize` in C++20
One limitation of `std::size` is that its return type is unsigned (`size_t`), which can trigger compiler warnings when compared against signed integers (`int`). To address this, **C++20** introduced **`std::ssize`**, which returns the size as a **signed** integer type.


# Summary Table
| Feature | Description |
| :--- | :--- |
| **Standard** | C++17 |
| **Header** | `<iterator>` |
| **Purpose** | Get the number of elements in containers and raw arrays |
| **Safety** | Prevents compilation if a raw pointer is passed instead of an array |
| **Main Benefit** | Provides a uniform interface for Template code |
---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>