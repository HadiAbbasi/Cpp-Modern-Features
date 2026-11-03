<div align="center">

[🇺🇸 English](./std_make_unique_for_overwrite.md) | [🇮🇷 فارسی](../../fa/cpp20/std_make_unique_for_overwrite.md)

</div>

---

# Introduction to `std::make_unique_for_overwrite` in C++20

## Introduction

The `std::make_unique_for_overwrite` facility was added to the standard library in C++20. It provides a way to create an object or array with exclusive ownership through `std::unique_ptr`, with one important difference: the created object is default-initialized rather than value-initialized.

Although this difference may seem small, it can matter in practice for buffers that will be completely overwritten immediately by an input operation or computation. For example, when receiving data from a file, network, or low-level API, it may be unnecessary to zero-initialize every element of an array before writing the incoming data.

However, this function is not a tool for creating “uninitialized memory” in the raw-storage sense. It still creates an object or array of a specific type, begins its lifetime, and transfers ownership to a `std::unique_ptr`. Understanding the exact initialization rules is essential for using it safely.

## Function Overview and Signature

The function is declared in the `<memory>` header and has been available since C++20. It has two primary overloads:

```cpp
template<class T>
std::unique_ptr<T> make_unique_for_overwrite();

template<class T>
std::unique_ptr<T> make_unique_for_overwrite(std::size_t size);
```

- The overload without arguments can be used only for non-array types.
- The overload that takes the `size` argument can be used only for arrays of unknown bound, such as `T[]`.
- The corresponding overload for arrays of known bound, such as `T[10]`, is intentionally deleted.

In both cases, the return type is a `std::unique_ptr`, so object lifetime and resource release are managed automatically.

## Understanding Default-initialization

To understand the function's behavior, it is important to distinguish among `default-initialization`, `value-initialization`, and `zero-initialization`. These are C++ language rules, not features specific to `make_unique_for_overwrite`.

### Initialization of Scalar Types

With `default-initialization`, no initial value is written for scalar types such as `int`, `double`, pointers, and `bool`. As a result, the program has no usable initial value to read.

```cpp
int* values = new int[4];

values[0] = 10;
values[1] = 20;
values[2] = 30;
values[3] = 40;
```

In this example, the array elements are not initialized before the assignments. Reading them before writing valid values is not permitted and cannot be relied upon.

By contrast, `value-initialization` produces zero values for these types:

```cpp
int* values = new int[4]();

```

Therefore, the key difference for an array of `int` is that `new int[n]` does not zero-initialize its elements, whereas `new int[n]()` does.

### Behavior for Class Types

For class types, `default-initialization` invokes the default constructor. If the default constructor is implicitly defined, the rules governing member initialization also determine the state of the object's members.

```cpp
#include <memory>

struct Record {
    int id;
    Record() : id(42) {}
};

int main() {
    auto record = std::make_unique_for_overwrite<Record>();
    return record->id;
}
```

In this example, the `Record` constructor is called, and `id` is set to 42. Therefore, the name `for_overwrite` does not mean that the class constructor is ignored.

However, if the default constructor does not initialize scalar members, those members will not have specified values either:

```cpp
struct Record {
    int id;
    double score;
};

int main() {
    auto record = std::make_unique_for_overwrite<Record>();
    record->id = 7;
    record->score = 98.5;
}
```

In this example, `id` and `score` must be assigned values before they are read.

### A Note About const

Scalar types qualified with `const` cannot be default-initialized in this way. Consequently, constructing an object such as `std::make_unique_for_overwrite<const int>()` is not valid. If an object must be `const`, use a mechanism that provides its value during construction.

## Comparison with std::make_unique

The `std::make_unique` function has been available since C++14. For a non-array type, it constructs an object using an expression equivalent to `new T(args...)`. When called without arguments, the object is value-initialized.

By contrast, `std::make_unique_for_overwrite` uses an expression equivalent to `new T`, which performs default-initialization.

The same distinction applies to arrays:

```cpp
#include <cstddef>
#include <memory>

int main() {
    constexpr std::size_t count = 1024;

    auto initialized = std::make_unique<int[]>(count);
    auto for_overwrite = std::make_unique_for_overwrite<int[]>(count);

    // initialized elements are zero-initialized
    // for_overwrite elements are not initialized
}
```

In this example, the elements of `initialized` are zero, whereas the elements of `for_overwrite` cannot be read until values have been written to them.

| Feature | `std::make_unique<T>()` | `std::make_unique_for_overwrite<T>()` |
|---|---|---|
| Initialization of an object with no arguments | `value-initialization` | `default-initialization` |
| Initialization of array elements | `value-initialization` | `default-initialization` |
| Initial value of a scalar type | Zero under value-initialization | No specific value is guaranteed |
| Invocation of a class's default constructor | Yes | Yes |
| Suitability for buffers that are immediately overwritten | May perform unnecessary initialization | Usually a more suitable choice |

This table concerns the no-argument and array overloads. If constructor arguments are passed to `std::make_unique<T>(args...)`, those arguments are used to construct the object, so the comparison is no longer simply between two forms of initialization without arguments.

## Creating a Non-array Object

For a non-array type, specify the type `T`, and the function creates an object of that type:

```cpp
#include <memory>

struct ConnectionState {
    int status;
    ConnectionState() : status(1) {}
};

int main() {
    auto state =
        std::make_unique_for_overwrite<ConnectionState>();

    return state->status;
}
```

The function uses the default constructor to create the object. If no such constructor exists or it is inaccessible, the call is not valid.

This overload does not accept constructor arguments. If arguments must be passed to the constructor during construction, use `std::make_unique<T>(args...)` instead.

## Creating a Dynamic Array

To create an array whose size is determined at runtime, specify an array type with an unknown bound and pass the size to the function:

```cpp
#include <cstddef>
#include <memory>

int main() {
    const std::size_t count = 4096;

    auto buffer =
        std::make_unique_for_overwrite<std::byte[]>(count);

    // The buffer is ready to receive bytes.
}
```

In this example, the type of `buffer` is `std::unique_ptr<std::byte[]>`. Storage is allocated for `count` elements, and the elements are default-initialized.

For scalar types such as `std::byte`, this does not produce an initial zero value. Therefore, each element must be populated with valid data before it is read.

### Multidimensional Arrays

This approach can also be used for multidimensional arrays whose first dimension has an unknown bound:

```cpp
#include <memory>

int main() {
    constexpr std::size_t rows = 100;

    auto matrix =
        std::make_unique_for_overwrite<int[][3]>(rows);

    matrix[0][0] = 1;
    matrix[0][1] = 2;
    matrix[0][2] = 3;
}
```

In this example, the number of rows is determined at runtime, and each row contains three `int` elements. As with a one-dimensional array, scalar elements are not automatically zero-initialized.

## Practical Use Cases

### Reading Data Directly from an Input Source

One suitable use case is creating a buffer that an input API is expected to fill, either completely or partially:

```cpp
#include <cstddef>
#include <memory>
#include <istream>

bool read_data(std::istream& input,
               std::size_t count,
               std::unique_ptr<char[]>& destination) {
    auto buffer =
        std::make_unique_for_overwrite<char[]>(count);

    input.read(buffer.get(),
               static_cast<std::streamsize>(count));

    if (!input) {
        return false;
    }

    destination = std::move(buffer);
    return true;
}
```

With this approach, the buffer is first created locally and transferred to the destination only after the input operation succeeds. If reading fails, the local `unique_ptr` releases the memory when it goes out of scope.

This example assumes that the input size is representable by `std::streamsize` and that the stream reads exactly the expected number of bytes. In production code, size conversion and the policy for handling partial reads should also be checked explicitly.

### Preparing a Buffer for C APIs

Some C-based APIs accept a pointer and a buffer capacity, then write data into that buffer themselves:

```cpp
#include <cstddef>
#include <memory>

int external_api(char* output, std::size_t capacity);

int main() {
    constexpr std::size_t capacity = 2048;

    auto buffer =
        std::make_unique_for_overwrite<char[]>(capacity);

    const int written = external_api(buffer.get(), capacity);

    if (written >= 0) {
        // Process only the number of bytes reported by the API.
    }
}
```

In this scenario, initially zeroing the buffer may be unnecessary, but only if the API contract guarantees that the portion being used is initialized before it is read.

### Allocating Large Buffers

For large buffers, omitting initial initialization can avoid unnecessary writes to memory. This may be useful in data-processing, I/O, and image-processing paths.

However, do not assume that this function is always faster. The result depends on the data type, allocator, standard library, operating system, allocation size, and memory-access pattern. If performance matters, measure it with a representative benchmark.

## Safety Considerations and Common Mistakes

### Reading Before Writing

The most important risk is reading an element that has not yet received a valid value:

```cpp
#include <memory>

int main() {
    auto values =
        std::make_unique_for_overwrite<int[]>(10);

    int first = values[0];
}
```

In this example, `values[0]` is read before it has been initialized. This is not permitted, and code must not rely on any particular behavior or value across standard versions.

The correct approach is to initialize all required elements first:

```cpp
#include <memory>

int main() {
    auto values =
        std::make_unique_for_overwrite<int[]>(10);

    for (int i = 0; i < 10; ++i) {
        values[i] = i * 2;
    }

    const int first = values[0];
}
```

### Partially Initialized Buffers

If an input operation fills only part of a buffer, only that portion can be processed. The allocated capacity is not the same as the number of bytes actually written.

For example, if a buffer has a capacity of 1,000 bytes but the API writes only 300 bytes, reading the remaining 700 bytes is not valid. It is better to track the actual number of elements written separately.

### Confusing Default-initialization with a Default Value

The name `for_overwrite` may suggest that all data is safely filled with a default value. That interpretation is incorrect. Scalar types have no guaranteed initial value, and the behavior for class types depends on the default constructor and member-initialization rules.

### Assuming Newly Allocated Memory Is Zeroed

Do not rely on elements being zero merely because an operating system or allocator happened to provide zeroed memory pages in a particular execution. Reliable behavior must follow from the language and standard-library rules, not from an incidental observation about a runtime environment.

### Choosing the Wrong Array Type

For a dynamic array, use an array type with an unknown bound:

```cpp
auto valid =
    std::make_unique_for_overwrite<int[]>(100);
```

An array type with a fixed bound is not supported by this function:

```cpp
// This call is intentionally invalid.
auto invalid =
    std::make_unique_for_overwrite<int[100]>();
```

For an array with a fixed size, `std::array` or another suitable object can be used.

## Limitations and Standard Details

### Overload Limitations

The function has only two primary forms: creating a non-array object without arguments, or creating an array of unknown bound by supplying its size. To construct an object with constructor arguments, `std::make_unique` is the appropriate choice.

Creating an array with a known bound through the corresponding deleted overload is also prohibited. This design follows the pattern of `std::make_unique` and helps prevent certain array-usage mistakes.

### Exception Handling and Ownership

If memory allocation fails, `std::bad_alloc` may be thrown. When constructing a class type, if its constructor throws an exception, construction does not complete, and the allocated resource is released according to the rules for a `new-expression`.

After successful construction, the object is exclusively owned by `std::unique_ptr`. The memory is released when the pointer goes out of scope, unless ownership has been transferred to another pointer using an operation such as `std::move`.

```cpp
#include <memory>

int main() {
    auto first =
        std::make_unique_for_overwrite<int[]>(256);

    auto second = std::move(first);

    // first is empty; second owns the array.
}
```

### Difference from Raw Memory Allocation

This function is not a general replacement for `operator new`, `std::malloc`, or raw-memory management tools. `make_unique_for_overwrite` creates an object or array of a specific type and follows that type's construction and destruction rules.

If the problem involves allocating raw storage, controlling alignment precisely, manually managing object lifetime, or using preallocated memory, tools such as `std::allocator`, `std::construct_at`, and related lifetime facilities may be more appropriate. Choosing them requires careful consideration of the object-lifetime rules.

### Difference from std::make_shared_for_overwrite

Since C++20, the functions `std::make_shared_for_overwrite` and `std::allocate_shared_for_overwrite` have also been available for creating objects with shared ownership and `default-initialization`.

The primary difference is the ownership model:

- `std::make_unique_for_overwrite` provides exclusive ownership through `std::unique_ptr`.
- `std::make_shared_for_overwrite` provides shared ownership through `std::shared_ptr`.
- `std::allocate_shared_for_overwrite` allows an allocator to be used when constructing a `shared_ptr`.

If shared ownership is not actually required, using `shared_ptr` merely to create a buffer is usually not an appropriate choice.

### constexpr Support in Later Versions

This function was introduced in C++20, but its overloads became `constexpr` in C++23. Therefore, `constexpr` support should not be attributed to the original C++20 version.

## Checking Standard Library Support

To check whether this feature is available in the standard library, you can use the following feature-test macro:

```cpp
#include <memory>

#if defined(__cpp_lib_smart_ptr_for_overwrite) && \
    __cpp_lib_smart_ptr_for_overwrite >= 202002L

// std::make_unique_for_overwrite is available.

#else

#error "The standard library does not provide smart pointer overwrite support"

#endif
```

The standardized value of this macro is `202002L`. It indicates that the standard library supports the smart-pointer overwrite facilities, making it more suitable for checking library support than relying on the compiler name alone.

For practical use, both the language mode and the standard library must be configured with the required support. For example, selecting C++20 mode in the compiler does not by itself guarantee that the installed standard library version implements all the required features.

## Choosing the Right Approach in Software Design

The choice between `std::make_unique` and `std::make_unique_for_overwrite` should be based on the data-initialization contract, not merely on the hope of improving performance.

- If the elements need to be zero-initialized or otherwise initialized normally, use `std::make_unique`.
- If all elements are immediately initialized by trusted code and will not be read beforehand, `std::make_unique_for_overwrite` may be appropriate.
- If only part of a buffer is written, track the valid portion's length separately.
- If the target type is a class, examine its default constructor and member initialization.
- If constructor arguments are required, use the regular `std::make_unique` overloads.
- If shared ownership is genuinely required, consider the `std::make_shared_for_overwrite` family.

In sensitive code, the boundary between the uninitialized portion and the valid portion can be made explicit with a size variable or a wrapper type. This reduces the likelihood of accidentally reading uninitialized data.

## Summary

The `std::make_unique_for_overwrite` function, introduced in C++20, provides a standard, RAII-based way to create an object or array with exclusive ownership while using `default-initialization` instead of `value-initialization`.

Its main advantage is avoiding initialization that would otherwise be immediately replaced by real data in certain code paths. In return, the programmer is responsible for ensuring that the data is fully initialized before it is read.

The practical rule is simple: use this function only when you can prove that every part of the object or buffer receives a valid value before its first read. Otherwise, `std::make_unique` is the safer and clearer choice.

## References

- [Working Draft: C++ Standard — Unique pointer creation](https://eel.is/c++draft/unique.ptr.create)
- [cppreference — std::make_unique and std::make_unique_for_overwrite](https://en.cppreference.com/w/cpp/memory/unique_ptr/make_unique)
- [cppreference — Default-initialization](https://en.cppreference.com/w/cpp/language/default_initialization)
- [WG21 P1973R1 — Rename “_default_init” Functions](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p1973r1.pdf)


---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>