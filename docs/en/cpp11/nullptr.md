<div align="right">

[🇺🇸 English](./nullptr.md) | [🇮🇷 فارسی](../../fa/cpp11/nullptr.md)

</div>
---

# nullptr type

 > **define:** In C++, the concept of a null pointer is used to represent a pointer that does not point to any valid memory address.

## From NULL and 0 to nullptr
In older versions of C++ (prior to the C++11 standard) and the C language, two methods were used to define a null pointer:

```cpp
int* ptr=0;
```
```cpp
 int* ptr=NULL;
```
 > **problem:** The biggest problem was that 0 or NULL are of the Integer data type, not the pointer type. This caused ambiguity and hard-to-debug issues during function overloading.

```cpp
#include <iostream>

void foo(int x) {
    std::cout << "Function with int called\n";
}

void foo(int* ptr) {
    std::cout << "Function with pointer called\n";
}

int main() {
    foo(NULL); 
}

```

### nullptr in c++11
Starting with the C++11 standard, the `nullptr` keyword was introduced to resolve this issue once and for all.

- **Independent data type:** `nullptr` is a literal of type `std::nullptr_t`.
- **Type safety:** This value automatically converts to pointer types but does not convert to numeric types (such as `int` or `char`).

```cpp
foo(nullptr);
```

```cpp
if (myPtr != nullptr) {
    *myPtr = 10;
} else {
}
```
```cpp
int* myPtr = nullptr; 

```

## summery
| Feature | `nullptr` (C++11) | `NULL` / `0` |
| :--- | :--- | :--- |
| **Data Type** | `std::nullptr_t` | Integral (`int` or cast to `void*` in C) |
| **Type Safety** | Highly Type-Safe | Weak (Prone to overload resolution issues) |
| **Recommended Standard** | Modern C++ (C++11 and newer) | Legacy C++ and C |

---
## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>