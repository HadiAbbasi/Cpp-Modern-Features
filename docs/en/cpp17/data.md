<div align="center">

[🇺🇸 English](./data.md) | [🇮🇷 فارسی](../../fa/cpp17/data.md)

</div>

Here is the complete guide to **`std::data` in C++17**:

# std::data

**What is `std::data`?**

The **`std::data`** function (introduced in **C++17** in the `<iterator>` header) returns a raw pointer to the first element of a contiguous container or a C-style array. 


##  Why was it introduced?

Before C++17, getting a pointer to the underlying data was inconsistent:
* For containers like `std::vector` or `std::string`, you had to use: `v.data()`
* For C-style raw arrays (e.g., `int arr[5]`), you had to use the array name or `&arr[0]`.

This inconsistency made writing **Generic / Template code** difficult. `std::data` (along with `std::size`, `std::begin`, and `std::end`) provides a **uniform interface** for both containers and raw arrays.


## How to Use & Definitions

You need to include the `<iterator>` header:
```cpp
#include <iterator>
```

It has two primary overloads:

1. **For Containers:**
   ```cpp
   template <class C>
   constexpr auto data(C& c) -> decltype(c.data());
   ```
   Works with any container that has a `.data()` member function (e.g., `std::vector`, `std::array`, `std::string`, `std::string_view`, `std::span`).

2. **For C-Style Raw Arrays:**
   ```cpp
   template <class T, std::size_t N>
   constexpr T* data(T (&array)[N]) noexcept;
   ```
   Returns a pointer to the first element of a raw array.


## Code Examples

### A) Basic Usage with Containers and Raw Arrays

```cpp
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    // 1. C-style raw array
    int raw_arr[] = {10, 20, 30};
    int* ptr1 = std::data(raw_arr);
    std::cout << "Raw array first element: " << *ptr1 << "\n"; // Output: 10

    // 2. Vector
    std::vector<int> vec = {100, 200, 300};
    int* ptr2 = std::data(vec);
    std::cout << "Vector first element: " << *ptr2 << "\n";       // Output: 100

    // 3. Modifying through the pointer
    *ptr2 = 999;
    std::cout << "Modified vec[0]: " << vec[0] << "\n";          // Output: 999

    return 0;
}
```


### B) Usage in Generic / Template Code

Suppose you want to write a function that passes data to a low-level or C-style API (like OS functions, OpenGL, or socket libraries):

```cpp
#include <iostream>
#include <vector>
#include <iterator>

// A generic function processing data as a C-style buffer
template <typename Container>
void sendToC_API(const Container& c) {
    const auto* buffer = std::data(c);
    auto length = std::size(c);

    std::cout << "Sending " << length << " items starting at address: " << buffer << "\n";
}

int main() {
    int c_arr[] = {1, 2, 3, 4};
    std::vector<double> v = {1.1, 2.2, 3.3};

    // This generic function works seamlessly with both raw arrays and vectors
    sendToC_API(c_arr);
    sendToC_API(v);

    return 0;
}
```


##  What works and what doesn't?

`std::data` only works with data structures that have **contiguous memory** and a `.data()` method:
* ✅ **Supported:** `std::vector`, `std::array`, `std::string`, `std::string_view`, and C-style arrays (`T[]`).
* ❌ **Not Supported:** `std::list`, `std::deque`, `std::set`, `std::map` (because their elements are not stored contiguously and they lack a `.data()` method).

---

## `constexpr` Support

`std::data` is marked as `constexpr`, meaning it can be evaluated at **compile-time**:

```cpp
constexpr int arr[] = {1, 2, 3};
constexpr const int* ptr = std::data(arr); // Evaluated at compile-time
```

---

## Summary
* **Where?** Header `<iterator>` in **C++17**.
* **What does it do?** Returns a raw pointer to the first element.
* **Main Benefit:** Standardizes pointer retrieval across raw arrays (`T[]`) and standard containers (`std::vector`, `std::array`, etc.), especially useful inside Templates.

---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>