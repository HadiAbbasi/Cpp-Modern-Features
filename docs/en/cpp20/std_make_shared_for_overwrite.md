<div align="center">

[🇺🇸 English](./std_make_shared_for_overwrite.md) | [🇮🇷 فارسی](../../fa/cpp20/std_make_shared_for_overwrite.md)

</div>

---

# `std::make_shared_for_overwrite` in C++20

## Introduction

The `std::make_shared_for_overwrite` facility was added to the family of `std::shared_ptr` creation utilities in the C++20 standard. Its primary purpose is to create an object or array in dynamic storage without performing zero-initialization or value-initialization.

This distinction matters when a program is going to overwrite the entire storage occupied by the object or array with valid data immediately after allocation. Examples include receiving data from a file or network, using a low-level API, or calling a buffer-filling function. In such situations, unnecessary initialization can introduce additional overhead.

However, omitting initialization places an important responsibility on the programmer: no value that has not yet been initialized must be read. Therefore, this function is a targeted optimization tool, not a general replacement for `std::make_shared`.

## Function Overview and Design Goal

The `std::make_shared_for_overwrite` function is declared in the standard `<memory>` header and has been available since C++20. It creates a `std::shared_ptr`, but default-initializes the object or array elements rather than value-initializing them.

The key difference is the kind of initialization performed:

- `std::make_shared<T>()` creates a non-array object using value-initialization.
- `std::make_shared_for_overwrite<T>()` creates a non-array object using default-initialization.
- The array overloads also default-initialize the array elements rather than value-initializing them.

This distinction is particularly noticeable for scalar types such as `int` and `double`. Value-initialization of an `int` typically initializes it to zero, whereas default-initialization does not initialize a scalar variable.

## Function Signatures and Overloads

The usage pattern for a non-array object is as follows:

```
template<class T>
std::shared_ptr<T> make_shared_for_overwrite();
```

For an array with a runtime-determined size, the following overload is available:

```
template<class T>
std::shared_ptr<T> make_shared_for_overwrite(std::size_t count);
```

You can also specify an array type with a fixed size:

```
template<class T>
std::shared_ptr<T> make_shared_for_overwrite();
```

In this case, the template argument can be a type such as `T[N]`. For an array with a runtime-determined size, the template argument is written as `T[]`, and the number of elements is passed as a function argument.

Common examples:

```
auto object = std::make_shared_for_overwrite<MyType>();

auto fixed_array =
    std::make_shared_for_overwrite<int[128]>();

auto dynamic_array =
    std::make_shared_for_overwrite<int[]>(128);
```

In the last example, the array size is determined at runtime. For a fixed-size array, the size is part of the type.

## Differences Between Initialization Forms

### Understanding Value-Initialization

With value-initialization, the C++ language rules determine how an object is initialized. For scalar types, the result is zero:

```
auto a = std::make_shared<int>();
auto b = std::make_shared<double>();

// Both objects are value-initialized.
```

In this example, the initial value of `*a` is zero, and the initial value of `*b` is `0.0`.

### Understanding Default-Initialization

With default-initialization, the behavior depends on the type of the object:

- For a scalar type such as `int`, no initialization is performed, and the object's value is indeterminate.
- For a class type, the appropriate constructor is executed.
- For an array, each element is default-initialized individually.

Therefore, describing this behavior as "no initialization" is not accurate for all types. The function still executes a constructor for class objects; what it omits is value-initialization and the implicit zero-initialization associated with it.

### Practical Comparison

```
#include <memory>

struct Record {
    int id;
    double score;

    Record() : id(0), score(0.0) {}
};

int main() {
    auto initialized = std::make_shared<int>();
    auto overwritten = std::make_shared_for_overwrite<int>();

    auto record = std::make_shared_for_overwrite<Record>();

    // initialized is zero.
    // overwritten must be assigned before it is read.
    // Record's constructor runs and initializes its members.
}
```

In this example, `initialized` has a value of zero. The value of `overwritten` must not be used until it has been written. By contrast, the `Record` constructor is executed and initializes its members.

## Behavior for Class and Struct Types

Using `std::make_shared_for_overwrite` does not bypass constructors. If the specified type is a class, the language's default-initialization rules apply, and its default constructor is executed.

An important point is that executing a constructor does not necessarily mean that all members are initialized:

```
#include <memory>

struct Packet {
    int size;
    int flags;

    Packet() : size(0) {}
};

int main() {
    auto packet = std::make_shared_for_overwrite<Packet>();

    // packet->size is initialized to zero.
    // packet->flags is not initialized by the constructor.
}
```

In this example, `size` is initialized, but `flags` is not. Consequently, `flags` must not be read unless a valid value has first been assigned to it.

This is especially important for classes whose default constructors do not initialize some of their members. When reviewing such code, all member-initialization paths should be examined.

## Creating Arrays

One important capability of this API is direct support for arrays. This is useful when creating buffers with shared ownership.

### Fixed-Size Arrays

```
#include <memory>

int main() {
    auto values =
        std::make_shared_for_overwrite<int[256]>();

    for (std::size_t i = 0; i < 256; ++i) {
        values[i] = static_cast<int>(i);
    }
}
```

The array elements have type `int`, and their default-initialization does not produce values. The loop writes every element before it is read.

### Runtime-Sized Arrays

```
#include <cstddef>
#include <memory>

int main() {
    const std::size_t count = 4096;

    auto buffer =
        std::make_shared_for_overwrite<std::byte[]>(count);

    for (std::size_t i = 0; i < count; ++i) {
        buffer[i] = std::byte{0};
    }
}
```

This example creates a dynamic array of `std::byte`. Each byte is initialized before it is used. In real code, data can be written through a data-receiving API or an I/O operation instead of using a loop.

### Multidimensional Arrays

Nested arrays follow the ordinary initialization rules as well. For example, a fixed-size two-dimensional array type can be specified as the template argument:

```
#include <memory>

int main() {
    auto matrix =
        std::make_shared_for_overwrite<int[16][16]>();

    for (int row = 0; row < 16; ++row) {
        for (int column = 0; column < 16; ++column) {
            matrix[row][column] = row + column;
        }
    }
}
```

All elements must be initialized before they are read. For large arrays, the element count and all write paths should be managed clearly and in a way that is easy to verify.

## Using the Function for Input and Output Buffers

One of the most suitable applications of this function is allocating storage that another operation is expected to fill, either completely or over a specified range.

For example, suppose an API writes data into a destination buffer and returns the number of bytes written:

```
#include <cstddef>
#include <memory>

std::size_t read_data(std::byte* destination,
                      std::size_t capacity);

int main() {
    constexpr std::size_t capacity = 8192;

    auto buffer =
        std::make_shared_for_overwrite<std::byte[]>(capacity);

    const std::size_t bytes_read =
        read_data(buffer.get(), capacity);

    // Only the first bytes_read bytes contain received data.
}
```

In this example, only the range `[0, bytes_read)` contains received data. Reading the remaining part of the buffer is not permitted unless it has been initialized separately.

When designing such APIs, the contract of the filling function should specify:

- How many bytes may be written.
- Whether part of the buffer may be modified if an error occurs.
- Whether the entire reported range is actually initialized.
- How the consumer obtains the length of the valid data.

### Managing a Buffer with Shared Ownership

If several parts of a program need to manage the buffer's lifetime through shared ownership, `std::shared_ptr` may be appropriate:

```
#include <cstddef>
#include <memory>

using ByteBuffer = std::shared_ptr<std::byte[]>;

ByteBuffer create_buffer(std::size_t size) {
    return std::make_shared_for_overwrite<std::byte[]>(size);
}
```

However, shared ownership does not, by itself, make concurrent access safe. If multiple threads access the buffer and at least one of them writes to it, appropriate synchronization is still required.

## Comparison with std::make_shared

The choice between these two functions should be based on the actual initialization requirements.

| Feature                                | `std::make_shared`                      | `std::make_shared_for_overwrite`                       |
| -------------------------------------- | --------------------------------------- | ------------------------------------------------------ |
| Standard                               | C++11                                   | C++20                                                  |
| Initialization of a non-array object   | Value-initialization                    | Default-initialization                                 |
| Initial value of a scalar type         | Typically zero                          | Indeterminate value                                    |
| Class constructor execution            | Yes                                     | Yes, as required by default-initialization rules       |
| Typical use                            | Creating an object that is ready to use | Creating an object or buffer for immediate overwriting |
| Risk of reading an uninitialized value | Lower                                   | Higher; requires explicit care                         |

For example, using `make_shared` is the natural choice in the following code:

```
auto counter = std::make_shared<int>();
++(*counter);
```

The initial value of `counter` is zero, so incrementing it is valid.

In the following example, however, using `make_shared_for_overwrite` may be appropriate because all elements are written before they are read:

```
auto samples =
    std::make_shared_for_overwrite<float[]>(sample_count);

fill_samples(samples.get(), sample_count);
process_samples(samples.get(), sample_count);
```

In this example, whether `process_samples` is valid depends on the contract of `fill_samples`: the filling function must initialize every element that the processing function reads.

## Comparison with Direct Memory Allocation

C++ provides other memory-allocation tools, including `new`, `std::unique_ptr`, and allocators. The appropriate choice depends on the ownership model and object-lifetime requirements.

### Comparison with new

The expression `new T` for a non-array object generally performs default-initialization. Therefore, in terms of the kind of initialization performed, it can be close to the behavior desired from `make_shared_for_overwrite`:

```
int* raw = new int;
```

However, this approach does not provide shared ownership or automatic lifetime management. Memory deallocation and exception handling must also be managed correctly.

By contrast, `std::make_shared_for_overwrite<T>()` returns a `shared_ptr<T>` and delegates ownership management to the standard library.

### Comparison with std::unique_ptr

If exclusive ownership is sufficient, `std::unique_ptr` is usually a simpler choice with a lower ownership-management overhead. For example:

```
#include <memory>

auto buffer =
    std::make_unique_for_overwrite<std::byte[]>(4096);
```

The `std::make_unique_for_overwrite` function was also introduced in C++20. It serves a similar initialization purpose but provides exclusive ownership.

As a design rule, do not use `shared_ptr` merely for convenience if you do not actually need multiple independent owners. The cost of a control block and reference-counting operations may be unnecessary.

### Comparison with Allocators

The allocator family provides greater control over memory allocation and object construction. In C++20, the `std::allocate_shared_for_overwrite` function is also available for creating a `shared_ptr` with a specified allocator.

This function is useful when a program uses a custom allocator, a memory resource, or a specific allocation policy. By contrast, `make_shared_for_overwrite` provides a simpler interface for ordinary cases where a custom allocator is not required.

## Relationship to std::allocate_shared_for_overwrite

The `std::allocate_shared_for_overwrite` function is the allocator-aware counterpart of this facility and was introduced in C++20. Its main difference from `make_shared_for_overwrite` is that it accepts an allocator as an argument.

Example usage:

```
#include <memory>

int main() {
    std::allocator<int> allocator;

    auto value =
        std::allocate_shared_for_overwrite<int>(allocator);

    *value = 42;
}
```

Choosing this function makes sense when the memory-allocation policy is part of the system's design. Otherwise, `make_shared_for_overwrite` is generally more readable.

## Lifetime and Ownership Considerations

The return value has type `std::shared_ptr`, so the usual shared-ownership rules still apply:

- The ownership count increases when a `shared_ptr` is copied.
- The object or array is destroyed when the last shared owner is gone.
- Using `std::weak_ptr` does not increase the ownership count.
- Reference-counting operations do not guarantee thread-safe access to the object itself.

An important implementation detail concerns the combined allocation of the object and the control block. Common implementations of `make_shared` and its related functions place the object and control block in a single allocation. This approach usually reduces the number of allocations. However, if a `weak_ptr` remains alive after the last `shared_ptr` has been destroyed, the memory allocated for the control block cannot be fully released until the last `weak_ptr` has also expired. In implementations that use a combined allocation, this can also delay the release of the object's storage.

This behavior matters for very large buffers. If the lifetime of `weak_ptr` instances may be significantly longer than that of the original owners, account for the resulting memory usage in your design.

## Limitations and Inappropriate Use Cases

Using this function is generally not a good choice in the following situations:

- When the program's logic requires an initial zero value or another specific default value.
- When some members or elements might not be initialized before they are read.
- When there are multiple error paths and it is difficult to guarantee that the entire buffer is initialized.
- When shared ownership is unnecessary and `std::unique_ptr` is more appropriate.
- When the goal is merely to shorten the code and there is no measurable cost associated with initialization.

This function is also not a tool for creating a raw object without executing its constructor. The ordinary object-construction rules still apply to class types.

## Important Notes About Indeterminate Values

For a scalar type created using default-initialization, its value must not be read before it has been written. Doing so can lead to undefined behavior; the precise standard rules concerning indeterminate values and their special cases depend on the type and context of use. The safe and reliable approach is to treat initialization as a prerequisite for every read.

For arrays, partial initialization must also be accompanied by tracking the length of the valid data. Having a buffer with a specified capacity does not mean that all of its elements contain valid data.

A useful pattern is to track capacity and the actual data size separately:

```
#include <cstddef>
#include <memory>

struct Buffer {
    std::shared_ptr<std::byte[]> storage;
    std::size_t capacity = 0;
    std::size_t size = 0;
};

Buffer make_buffer(std::size_t capacity) {
    return {
        std::make_shared_for_overwrite<std::byte[]>(capacity),
        capacity,
        0
    };
}
```

In this design, `size` should be increased only after data has been written successfully. The consumer should process only the valid range.

## Performance Considerations and Benchmarking

The potential benefit of this function is the removal of unnecessary initialization overhead. For large buffers, zeroing all memory and then overwriting it can consume memory bandwidth unnecessarily.

However, a performance improvement is not guaranteed. The result depends on several factors:

- The data type and allocation size.
- The behavior of the allocator and standard library.
- How the memory is used and the access pattern.
- Whether the subsequent function actually overwrites the entire range.
- Compiler optimizations and runtime characteristics.

Consequently, this function should be selected based on profiling and measurements under a real workload. If the data fills only part of the buffer, the potential benefit should be weighed against the complexity of managing the valid range.

## Common Mistakes

### Assuming That Elements Are Zero-Initialized

A common mistake is to assume that an array created this way is zero-initialized, just like an array created using `make_shared`. This assumption is incorrect for scalar elements.

### Reading a Buffer Before It Has Been Filled

Receiving a valid pointer from a `shared_ptr` does not mean that the contents of the memory have been initialized. A valid address and a valid stored value are two different concepts.

### Ignoring Partial Initialization

Some APIs fill only part of a buffer. Consuming the entire capacity instead of the actual data length can result in reading uninitialized values.

### Assuming That the Constructor Is Not Executed

This function does not eliminate the default constructor of a class. If the goal is to prevent constructor execution or manage object lifetimes manually, this API is not the appropriate tool.

### Choosing shared_ptr Without a Real Need

If a buffer has only one owner, using `make_unique_for_overwrite` is generally a more appropriate design. Shared ownership should reflect an actual requirement of the program.

## Selection Guide

The following rules can help with quick decisions:

| Program requirement                                                  | Recommended choice                                |
| -------------------------------------------------------------------- | ------------------------------------------------- |
| Creating an object with ordinary initialization                      | `std::make_shared`                                |
| Creating an object with shared ownership for immediate overwriting   | `std::make_shared_for_overwrite`                  |
| Creating a buffer with exclusive ownership for immediate overwriting | `std::make_unique_for_overwrite`                  |
| Creating an object with a custom allocator and shared ownership      | `std::allocate_shared_for_overwrite`              |
| Requiring a zero-initialized buffer                                  | `std::make_shared` with an appropriate array type |

## Summary

The `std::make_shared_for_overwrite` function, introduced in C++20, provides a standard way to create an object or array with shared ownership without value-initialization. This capability is particularly useful for buffers that are immediately filled with real data and for which initial zeroing would introduce unnecessary overhead.

The most important aspect of using this API is understanding the difference between default-initialization and value-initialization. For scalar types and array elements, no initial value is guaranteed; therefore, each value must be written before it is read. For class types, the default constructor is executed according to the language rules, and members that the constructor does not initialize may still have indeterminate values.

Ultimately, this function is an appropriate choice when the program requires shared ownership, all read paths occur only after valid initialization, and measurements show that eliminating initialization provides a meaningful performance benefit. In other cases, `std::make_shared` or an exclusive-ownership utility may be a simpler and safer choice.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>