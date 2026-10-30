<div align="center">

[🇺🇸 English](./data.md) | [🇮🇷 فارسی](../../fa/cpp17/data.md)

</div>

# std::empty

**What is `std::empty`?**

The **`std::empty`** function is another very useful utility function added alongside `std::size` and `std::data` in **C++17** to the `<iterator>` header. It checks whether a container, raw array, or initializer list is empty, returning a `bool` (`true` or `false`).


## Why was it introduced?

Before C++17, checking for emptiness was inconsistent:
* For standard containers (like `vector`, `string`, `list`), you called `c.empty()`.
* For C-style raw arrays (like `int arr[5]`), there was no `.empty()` member function (and a normal array with a fixed size is never empty).
* For `std::initializer_list`, there was no `.empty()` method, so you had to rely on `il.size() == 0`.

`std::empty` provides a **uniform interface** for all these structures, eliminating the need for conditional checks or different approaches inside **Generic / Template code**.


## 2. Definition and Signatures

To use it, you need to include the `<iterator>` header:
```cpp
#include <iterator>
```

It provides three main overloads:

1. **For Containers:**
   ```cpp
   template <class C>
   [[nodiscard]] constexpr auto empty(const C& c) -> decltype(c.empty());
   ```
   Invokes the `.empty()` member function of any supporting container.

2. **For C-Style Raw Arrays:**
   ```cpp
   template <class T, std::size_t N>
   [[nodiscard]] constexpr bool empty(const T (&array)[N]) noexcept {
       return false;
   }
   ```
   *Fun fact:* Because C-style raw arrays have a fixed, non-zero size, this function always returns **`false`**.

3. **For Initializer Lists (`std::initializer_list`):**
   ```cpp
   template <class E>
   [[nodiscard]] constexpr bool empty(std::initializer_list<E> il) noexcept {
       return il.size() == 0;
   }
   ```

*(The `[[nodiscard]]` attribute was officially added in C++20 to warn the developer if the return value of checking emptiness is ignored).*


## Code Examples

### A) Basic Usage

```cpp
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    std::vector<int> numbers;

    // Check if the vector is empty
    if (std::empty(numbers)) {
        std::cout << "Vector is empty!\n"; // This will print
    }

    numbers.push_back(42);

    if (!std::empty(numbers)) {
        std::cout << "Vector is NOT empty!\n";
    }

    // C-style raw array
    int arr[] = {1, 2, 3};
    std::cout << std::boolalpha << std::empty(arr) << "\n"; // Always false

    return 0;
}
```


### B) Usage in Generic Code (Templates)

The true power of `std::empty` shines when writing functions that accept any kind of container or collection:

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <iterator>

template <typename Container>
void printFirstElement(const Container& c) {
    // We use std::empty(c) instead of c.empty()
    if (std::empty(c)) {
        std::cout << "Container is empty, cannot print first element.\n";
        return;
    }

    std::cout << "First element: " << *std::begin(c) << "\n";
}

int main() {
    std::vector<int> empty_vec;
    std::string text = "Hello C++17";
    int raw_arr[] = {100, 200, 300};

    printFirstElement(empty_vec); // Output: Container is empty...
    printFirstElement(text);      // Output: First element: H
    printFirstElement(raw_arr);   // Output: First element: 100

    return 0;
}
```


##  Why use `std::empty(c)` instead of `c.size() == 0`?

This is a best practice in modern C++:
1. **Performance ($O(1)$ guarantee):** For certain containers or custom data structures, calculating `.size()` might take $O(N)$ time, whereas `.empty()` is guaranteed to run in constant time $O(1)$.
2. **Code Readability:** It clearly expresses your intent. Reading `std::empty(c)` is much more explicit and natural than comparing a size against zero.


## Summary Table
| Feature | Description |
| :--- | :--- |
| **Standard** | C++17 |
| **Header** | `<iterator>` |
| **Purpose** | Check if a container, raw array, or initializer list is empty (returns `bool`) |
| **Main Benefit** | Unifies emptiness checks across STL containers, raw C-arrays, and templates |
---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>