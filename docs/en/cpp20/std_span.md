<div align="center">

[🇺🇸 English](./std_span.md) | [🇮🇷 فارسی](../../fa/cpp20/std_span.md)

</div>

---

# A Comprehensive Guide to `std::span` in C++20

## Introduction and Design Goals

`std::span` is an important component of the C++20 Standard Library designed to represent a non-owning view over a contiguous sequence of elements in memory. It allows a program to access a collection of elements without copying the data or transferring ownership.

In many C++ APIs, functions receive a pointer and an element count rather than an entire collection. Although this approach is simple, it can introduce errors, such as mismatches between an array's length and the specified element count, incorrect size arguments, and interfaces that are difficult to reuse.

Consider the following traditional API:

```
#include <cstddef>
#include <iostream>

void printValues(const int* data, std::size_t size) {
    for (std::size_t i = 0; i < size; ++i) {
        std::cout << data[i] << ' ';
    }

    std::cout << '\n';
}
```

This function requires two independent arguments. The compiler cannot guarantee that `size` matches the number of elements accessible through `data`.

In C++20, the same interface can be simplified with `std::span`:

```
#include <iostream>
#include <span>

void printValues(std::span<const int> values) {
    for (int value : values) {
        std::cout << value << ' ';
    }

    std::cout << '\n';
}
```

With this design, the pointer to the data and the number of elements are represented together as a single view.

The main benefits of this approach include:

- Avoiding unnecessary element copies.
- Keeping the data address and element count together.
- Supporting C-style arrays, `std::array`, `std::vector`, and many other compatible containers.
- Providing a common interface for algorithms that operate on contiguous sequences.
- Supporting bounds-aware operations in some contexts without introducing ownership or memory-allocation overhead.

However, `std::span` does not own its data and does not guarantee that the underlying elements remain alive. This is the most important principle to understand when using it correctly.

## Understanding the Structure of `std::span`

### What Is a Non-Owning View?

A non-owning view is an object that provides access to existing data without being responsible for creating, storing, or releasing that data.

For example, suppose a `std::vector<int>` contains five elements. A `std::span<int>` can represent all or part of those elements. Changes made through the span affect the original memory.

```
#include <iostream>
#include <span>
#include <vector>

int main() {
    std::vector<int> numbers{10, 20, 30, 40, 50};

    std::span<int> values{numbers};

    values[1] = 200;

    std::cout << numbers[1] << '\n';
}
```

Output:

```
200
```

In this example, no independent copy of the elements is created. The `values` variable is simply a view of the memory owned by `numbers`.

### The Conceptual Components of a Span

A span can be conceptually described using two components:

- Pointer: The address of the first element.
- Extent: The number of elements accessible through the view.

This is a conceptual description. The C++ standard does not require an implementation to use exactly two data members with this structure.

There are two principal forms of extent:

- Dynamic extent: The size is determined at runtime.
- Static extent: The size is part of the type at compile time.

For example:

```
#include <span>

std::span<int> dynamicValues;
std::span<int, 4> fixedValues;
```

In this example, `dynamicValues` has no fixed extent encoded in its type, whereas `fixedValues` represents a view with a fixed extent of four elements.

### Required Header and Language Standard

`std::span` was introduced in C++20 and is declared in the following header:

```
#include <span>
```

Using it requires a compiler and standard library implementation that provide the necessary C++20 support.

For example, with environments using GCC or Clang, you may need to specify the language standard explicitly:

```
g++ -std=c++20 main.cpp -o main
```

Or:

```
clang++ -std=c++20 main.cpp -o main
```

Specifying the language standard alone does not guarantee that every required feature is implemented in the installed version of the standard library.

## Understanding Dynamic and Static Extent

### Dynamic Extent

With dynamic extent, the number of elements is not part of the type and is determined at runtime.

```
#include <iostream>
#include <span>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    std::span<int> view{values};

    std::cout << view.size() << '\n';
}
```

Output:

```
5
```

The type `std::span<int>` can represent views with different sizes. It is therefore a common choice for general-purpose interfaces and functions that accept variable-length input.

### Static Extent

With static extent, the size is part of the type's template parameters.

```
#include <iostream>
#include <span>

void processFour(std::span<int, 4> values) {
    values[0] = 100;

    std::cout << values.size() << '\n';
}

int main() {
    int data[4]{1, 2, 3, 4};

    processFour(data);
}
```

Output:

```
4
```

Static extent is appropriate when the number of elements is fixed by the API's design, such as in functions that always receive a four-dimensional vector or a block with a specified size.

Static extent can provide additional information to the type system, but it does not guarantee that the underlying data is valid or that out-of-bounds access cannot occur.

### Comparing the Two Extent Types

| Feature                      | Dynamic Extent                                     | Static Extent                            |
| ---------------------------- | -------------------------------------------------- | ---------------------------------------- |
| Example type                 | `std::span<int>`                                   | `std::span<int, 4>`                      |
| Size encoded in the type     | No                                                 | Yes                                      |
| Flexibility                  | Greater                                            | Restricted to a fixed size               |
| Suitable for                 | General-purpose APIs and variable-length sequences | Data with a fixed size                   |
| Runtime size storage         | Typically required                                 | Not required for the fixed extent itself |
| Conversion to dynamic extent | Supported                                          | Supported                                |

An important point is that static extent is not necessarily more efficient in every implementation. Its primary advantage is expressing a size constraint in the type system and providing more information to the compiler and the reader.

## Constructing a Span

### Constructing from a C-Style Array

One of the simplest ways to construct a span is directly from an array whose size is known.

```
#include <span>

int main() {
    int values[3]{10, 20, 30};

    std::span<int> allValues{values};
    std::span<int, 3> fixedValues{values};
}
```

Because the array's size is known at compile time, the appropriate constructor can use that information when constructing the span.

### Constructing from `std::array`

`std::array` is designed for arrays with a fixed size and is directly compatible with `std::span`.

```
#include <array>
#include <span>

int main() {
    std::array<int, 4> values{1, 2, 3, 4};

    std::span<int> view{values};
    std::span<int, 4> fixedView{values};
}
```

Because `std::array` owns its elements, the lifetime of the span depends on the lifetime of the array.

### Constructing from `std::vector`

Spans can also be constructed from contiguous containers such as `std::vector`.

```
#include <span>
#include <vector>

int main() {
    std::vector<int> values{5, 10, 15, 20};

    std::span<int> view{values};
}
```

In this case, the span does not own the vector's memory. If an operation such as `push_back` causes the vector to reallocate its storage, existing pointers may become invalid, and the span may no longer refer to valid elements.

### Constructing from a Pointer and Size

When working with a legacy API, an external library, or a specific memory region, the available information may consist only of a pointer and an element count.

```
#include <cstddef>
#include <span>

void process(std::span<const int> values);

int main() {
    int data[5]{1, 2, 3, 4, 5};

    std::span<const int> firstFour{data, std::size_t{4}};

    process(firstFour);
}
```

With this approach, the programmer is responsible for ensuring that the pointer is valid and that the specified number of elements is accessible.

If the size comes from an untrusted source or external input, constructing a span does not automatically validate it.

### Constructing from Two Iterators or a Sentinel

C++20 also provides constructors based on an iterator and a sentinel for suitable iterator and sentinel types.

```
#include <array>
#include <span>

int main() {
    std::array<int, 5> values{10, 20, 30, 40, 50};

    std::span<int> view{values.begin(), values.end()};
}
```

This approach is suitable for ranges whose iterators meet the required constraints and whose elements occupy contiguous memory.

Not every range can be converted to a span. For example, `std::list` does not store its elements contiguously, so it cannot be converted directly into a span.

### Constructing an Empty Span

A span with dynamic extent can be constructed as an empty view.

```
#include <span>

int main() {
    std::span<int> empty;

    return empty.empty() ? 0 : 1;
}
```

In contrast, a span with a positive static extent, such as `std::span<int, 4>`, cannot represent an empty view because its extent is part of the type.

## Accessing Elements and Size Information

### Using `size()` and `empty()`

The `size()` and `empty()` member functions are used to inspect the size of a view.

```
#include <iostream>
#include <span>
#include <vector>

int main() {
    std::vector<int> values{2, 4, 6};

    std::span<int> view{values};

    std::cout << view.size() << '\n';
    std::cout << view.empty() << '\n';
}
```

Output:

```
3
0
```

The `size()` member function returns the number of elements, while `empty()` indicates whether the number of elements is zero.

The return type of `size()` is `std::size_t`.

### Accessing Elements with `operator[]`

The `operator[]` operator provides direct access to individual elements.

```
#include <iostream>
#include <span>

int main() {
    int data[]{10, 20, 30};

    std::span<int> values{data};

    std::cout << values[1] << '\n';
}
```

Output:

```
20
```

In C++20, `operator[]` does not guarantee bounds checking. Accessing an invalid index results in undefined behavior.

Therefore, if an index comes from user input or untrusted data, validate it before accessing the element.

### Using `front()` and `back()`

The `front()` and `back()` member functions provide access to the first and last elements, respectively.

```
#include <iostream>
#include <span>

int main() {
    int data[]{3, 6, 9};

    std::span<int> values{data};

    std::cout << values.front() << '\n';
    std::cout << values.back() << '\n';
}
```

Output:

```
3
9
```

Calling either function on an empty span is invalid and can result in undefined behavior.

### Using `data()`

The `data()` member function returns a pointer to the first element.

```
#include <span>
#include <iostream>

void legacyFunction(const int* data, std::size_t size);

int main() {
    int values[]{1, 2, 3};

    std::span<const int> view{values};

    legacyFunction(view.data(), view.size());
}
```

This feature is useful when integrating modern APIs with legacy functions. However, once the pointer is extracted, the extent is no longer carried with it, and responsibility for respecting the bounds returns to the caller and the destination function.

### Iterating with a Range-Based `for` Loop

In many cases, iterating directly over a span is more readable.

```
#include <iostream>
#include <span>

int main() {
    int values[]{1, 2, 3, 4};

    std::span<int> view{values};

    for (int& value : view) {
        value *= 2;
    }

    for (int value : view) {
        std::cout << value << ' ';
    }
}
```

Output:

```
2 4 6 8
```

In this example, using `int&` allows the original elements to be modified. If the elements only need to be read, using a const-qualified element type is often more appropriate.

## Understanding `std::span<T>` and `std::span<const T>`

### Controlling Write Access

The type `std::span<T>` allows elements to be modified, provided the underlying data is itself mutable.

```
#include <span>

void modify(std::span<int> values) {
    if (!values.empty()) {
        values[0] = 42;
    }
}
```

By contrast, `std::span<const T>` provides read-only access to the elements.

```
#include <iostream>
#include <span>

void print(std::span<const int> values) {
    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

The second form is often a better choice for functions that only read data because it makes the function's intent explicit and prevents accidental modification through that view.

### Distinguishing a Const Span from a Span of Const Elements

The following declarations have different meanings:

```
std::span<int> values;
const std::span<int> fixedView;
```

In the first declaration, the span itself is mutable, and its elements can also be modified.

In the second declaration, the span object is const, but its elements remain mutable through the `int` element type.

To prevent element modification through the view, use `std::span<const int>`:

```
std::span<const int> readOnlyValues;
```

In this case, the span itself can be reassigned if it is not const, but its elements cannot be written through that span.

### Converting a Mutable Span to a Read-Only Span

Converting a mutable view to a read-only view is supported.

```
#include <span>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    std::span<int> writable{values};
    std::span<const int> readable{writable};
}
```

This conversion removes write access through the new view, but it does not make the underlying data itself immutable. Other code that has mutable access can still modify the elements.

## Using Span in API Design

### Replacing Pointer-and-Size Parameters

One of the most important uses of `std::span` is designing functions that operate on contiguous sequences of elements.

```
#include <span>
#include <numeric>

int sum(std::span<const int> values) {
    return std::accumulate(values.begin(), values.end(), 0);
}
```

The function can now be called with different kinds of data sources:

```
#include <array>
#include <vector>
#include <iostream>

int main() {
    int raw[]{1, 2, 3};

    std::array<int, 3> fixed{4, 5, 6};

    std::vector<int> dynamic{7, 8, 9};

    std::cout << sum(raw) << '\n';
    std::cout << sum(fixed) << '\n';
    std::cout << sum(dynamic) << '\n';
}
```

This example assumes that the definition of `sum` is available in the same translation unit or through an appropriate header.

This design reduces the need for separate overloads for arrays, vectors, and `std::array`.

### Using Span in Read-Only APIs

For functions that only analyze data, `std::span<const T>` is usually appropriate.

```
#include <span>
#include <algorithm>

bool containsZero(std::span<const int> values) {
    return std::find(values.begin(), values.end(), 0) != values.end();
}
```

The function can work with compatible data sources without copying their elements.

### Using Span in Mutating APIs

If a function needs to modify existing elements, it can accept a mutable span.

```
#include <span>

void clearValues(std::span<int> values) {
    for (int& value : values) {
        value = 0;
    }
}
```

This function modifies the elements without changing the size or ownership of the original container.

### Choosing Between Span and an Owning Container

When designing an API, distinguish between receiving a view and receiving an owning container.

| Requirement                                              | Appropriate choice                             |
| -------------------------------------------------------- | ---------------------------------------------- |
| Inspecting or processing existing elements               | `std::span<const T>`                           |
| Modifying existing elements without changing their count | `std::span<T>`                                 |
| Owning an independent copy of the data                   | `std::vector<T>` or another suitable container |
| Adding and removing elements                             | An owning container such as `std::vector<T>`   |
| Representing a sequence with a fixed extent in the type  | `std::span<T, N>`                              |
| Accessing non-contiguous elements                        | Another suitable container or view             |

Do not choose a span solely because it reduces the number of lines of code. If a function must retain data for later use, a span alone is not a safe choice because it does not manage the lifetime of the external data.

## Working with Subranges

One of the important features of `std::span` is the ability to create views of parts of a sequence without copying elements.

### Using `first()`

The `first()` member function creates a view containing a specified number of initial elements.

```
#include <iostream>
#include <span>

int main() {
    int values[]{10, 20, 30, 40, 50};

    std::span<int> view{values};

    auto firstThree = view.first<3>();

    for (int value : firstThree) {
        std::cout << value << ' ';
    }
}
```

Output:

```
10 20 30
```

There are two principal forms:

```
auto a = view.first<3>();
auto b = view.first(3);
```

In the first form, the element count is known at compile time, and the result has an appropriate static extent. In the second form, the count is determined at runtime, and the result has dynamic extent.

The requested count must not exceed the span's size. Violating this requirement in C++20 results in undefined behavior.

### Using `last()`

The `last()` member function creates a view of the final elements in the sequence.

```
#include <iostream>
#include <span>

int main() {
    int values[]{10, 20, 30, 40, 50};

    std::span<int> view{values};

    auto lastTwo = view.last<2>();

    for (int value : lastTwo) {
        std::cout << value << ' ';
    }
}
```

Output:

```
40 50
```

This function is useful for accessing a suffix of a sequence without allocating memory or copying elements.

### Using `subspan()`

The `subspan()` member function creates a view of an arbitrary section of a sequence.

```
#include <iostream>
#include <span>

int main() {
    int values[]{10, 20, 30, 40, 50};

    std::span<int> view{values};

    auto middle = view.subspan(1, 3);

    for (int value : middle) {
        std::cout << value << ' ';
    }
}
```

Output:

```
20 30 40
```

The first parameter specifies the starting offset, and the second specifies the number of elements.

You can also omit the element count to include all remaining elements:

```
auto suffix = view.subspan(2);
```

In this example, the resulting view contains the elements from index two through the end of the sequence.

If the offset or element count exceeds the valid range, undefined behavior may occur. For untrusted input, validate the range before constructing the view.

### Using `std::dynamic_extent`

In C++20, `std::dynamic_extent` represents dynamic extent in the type system.

```
#include <span>

static_assert(std::span<int>::extent == std::dynamic_extent);
static_assert(std::span<int, 4>::extent == 4);
```

This constant is useful when writing templates and inspecting type properties at compile time.

## Bounds Checking and Memory Safety

### How `std::span` Differs from Checked Access

The `std::span` type stores range information, but it does not automatically check every access.

For example, the following code is unsafe:

```
#include <span>

int main() {
    int values[]{1, 2, 3};

    std::span<int> view{values};

    return view[10];
}
```

Index ten is outside the range of three elements. In C++20, `operator[]` does not guarantee bounds checking, so this code has undefined behavior.

Explicit validation provides a way to control the program's behavior:

```
#include <span>
#include <stdexcept>

int readValue(std::span<const int> values, std::size_t index) {
    if (index >= values.size()) {
        throw std::out_of_range("Index out of range");
    }

    return values[index];
}
```

This approach is appropriate for APIs that operate on externally supplied indices.

### Comparing `std::span` with `std::vector::at()`

The `at()` member function of `std::vector` throws `std::out_of_range` when an index is outside the valid range.

However, `std::span` does not provide an `at()` member function in C++20.

Therefore, if every access must be bounds-checked, implement the checks explicitly or use an appropriate abstraction that provides this guarantee.

### The Role of Sanitizers

Tools such as AddressSanitizer can detect many memory-access errors at runtime.

For example, in compatible environments, you can use the following options:

```
g++ -std=c++20 -fsanitize=address,undefined -fno-omit-frame-pointer main.cpp -o main
```

These tools are useful for finding common errors, but they do not replace correct design and logical bounds validation.

## Data Lifetime and the Risk of Dangling Spans

### The Fundamental Lifetime Rule

Because a span does not own its data, its validity depends on the lifetime of the underlying elements.

For example, the following function returns an invalid span:

```
#include <span>
#include <vector>

std::span<const int> createValues() {
    std::vector<int> values{1, 2, 3};

    return std::span<const int>{values};
}
```

When the function returns, the local variable `values` is destroyed, and its memory is no longer valid for use through the span.

Consequently, using the returned span can result in undefined behavior.

### A Correct Way to Return Data

If a function must produce data that remains valid after the function returns, it should generally return ownership of that data to the caller.

```
#include <vector>

std::vector<int> createValues() {
    return {1, 2, 3};
}
```

The caller can then create a span from the valid vector:

```
#include <span>
#include <vector>

int main() {
    std::vector<int> values = createValues();

    std::span<const int> view{values};
}
```

This design makes ownership explicit, and the span's lifetime depends on the lifetime of `values`.

### The Danger of Constructing a Span from a Temporary

A common mistake is constructing a span from a temporary container and retaining it after the full expression has ended.

```
#include <span>
#include <vector>

int main() {
    std::span<const int> view{std::vector<int>{1, 2, 3}};

    return view[0];
}
```

In C++20, the ordinary span constructors are constrained to reject many unsafe conversions from temporary ranges. Consequently, this form of construction generally fails to compile in conforming implementations.

However, these restrictions do not solve every lifetime problem. If a program obtains an invalid pointer, reference, or span through another path, constructing a new span from that source does not necessarily reveal the problem.

The essential rule is that the underlying data must remain valid throughout the entire period in which the span is used.

### The Effect of Vector Reallocation

Even when a vector remains alive, structural changes can invalidate a span.

```
#include <span>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    std::span<int> view{values};

    values.push_back(4);

    // The span may be invalid if vector reallocation occurred.
}
```

If `push_back` causes reallocation, pointers to the previous vector elements become invalid.

Because the span refers to the same underlying memory, it may no longer point to valid elements.

If the vector has enough capacity to insert the new element, appending to the end generally does not invalidate pointers to existing elements. However, API designs should not assume that a span remains valid without considering the precise conditions.

### The Effect of Resizing and Erasing Elements

Operations such as `erase`, `clear`, and resizing a vector can affect the validity of existing spans.

Even when the memory address does not change, erasing or destroying elements can make access through a span invalid.

Therefore, span validity depends not only on whether the original container is still alive, but also on the container's invalidation rules and the state of its elements.

## Using Span in Classes and Asynchronous Functions

### Storing a Span as a Class Member

Using a span as a local variable is usually straightforward, but storing it as a class member requires more careful design.

```
#include <span>

class Processor {
public:
    explicit Processor(std::span<const int> values)
        : values_(values) {}

    int firstValue() const {
        return values_.empty() ? 0 : values_.front();
    }

private:
    std::span<const int> values_;
};
```

This class does not own the data. The caller must therefore ensure that the underlying data remains alive and valid for as long as the `Processor` object uses it.

For classes with long lifetimes or classes that retain data for future operations, storing a span may be inappropriate unless the lifetime contract is explicit.

### Using Span in Callbacks and Asynchronous Tasks

The risk of dangling spans is greater in asynchronous programs because a callback may execute at a different time from the lifetime of the original data.

For example, if a task executes after a function returns, a local span or a span referring to a temporary buffer may no longer be valid.

Appropriate alternatives can include:

- Transferring ownership of the data to the task.
- Keeping an owning container, such as a `std::vector`, in a shared object.
- Using an appropriate ownership mechanism, such as `std::shared_ptr`, with a clearly defined lifetime design.
- Restricting the span to a synchronous, guaranteed execution scope.

Using `std::span` alone provides no guarantees about thread safety, synchronization, or asynchronous lifetime management.

## Comparing Span with Pointers, Containers, and Other Views

### Comparing Span with a Pointer and Size

The traditional pointer-and-size pattern looks like this:

```
void process(const int* data, std::size_t size);
```

A span-based interface looks like this:

```
void process(std::span<const int> values);
```

The second interface groups the pointer and size into a single object and reduces the risk of mixing up argument order or separating the pointer from its associated size.

However, a span does not guarantee that the original pointer and size were valid. If a span is constructed using an invalid pointer or an incorrect size, the resulting view can still be invalid.

### Comparing Span with `std::vector`

The type `std::vector` is an owning container that manages the memory occupied by its elements.

In contrast, `std::span` is simply a view.

| Feature                                | `std::span`         | `std::vector`               |
| -------------------------------------- | ------------------- | --------------------------- |
| Memory ownership                       | No                  | Yes                         |
| Allocates memory for elements          | No                  | May allocate                |
| Can change size                        | No                  | Yes                         |
| Can add or remove elements             | No                  | Yes                         |
| Provides access to contiguous elements | Yes                 | Yes                         |
| Suitable for API input parameters      | Often very suitable | Depends on the requirements |
| Independent of the source's lifetime   | No                  | Yes, for its own owned data |

If the goal is simply to process existing elements, span is often a better choice for a function parameter. If the goal is to retain or construct an independent collection, an owning container is more appropriate.

### Comparing Span with `std::string_view`

The type `std::string_view` is also a non-owning view, but it is designed specifically for character sequences.

In contrast, `std::span` can represent contiguous sequences of elements of arbitrary types.

```
#include <span>
#include <string_view>

void processBytes(std::span<const unsigned char> bytes);

void processText(std::string_view text);
```

For text, `std::string_view` is usually the more natural interface because it provides string-oriented semantics and works with common string types.

For binary data, buffers, and numeric arrays, `std::span` is generally more appropriate.

Both types are non-owning, and the lifetime of their underlying data must be managed carefully.

### Comparing Span with `std::ranges::subrange`

In C++20, `std::ranges::subrange` was introduced to represent a range using an iterator and a sentinel.

Unlike span, a `subrange` is not necessarily restricted to contiguous elements in memory.

For example, a `subrange` can represent a portion of a list-based container, whereas `std::span` is not suitable for such a non-contiguous container.

Consequently, the choice between these two types depends on the data structure and the algorithm's requirements:

- Use `std::span` for contiguous sequences and direct memory access.
- Use `std::ranges::subrange` when a range is represented by an iterator and sentinel and does not necessarily need to be contiguous.

### Comparing Span with `std::mdspan`

In C++23, `std::mdspan` was introduced to represent multidimensional data.

An ordinary span represents a one-dimensional sequence of contiguous elements, whereas `std::mdspan` supports multidimensional indexing with different layout policies.

For example, when data is stored in memory as a two-dimensional array, `std::mdspan` can provide access through row and column indices.

Therefore, for a simple buffer of elements, span is usually sufficient. For modeling matrices and multidimensional data, `std::mdspan` may be a more precise choice.

## Practical Applications

### Processing Binary Buffers

In systems that receive data from files, networks, or hardware devices, a buffer and its size are often available together.

The type `std::span<const std::byte>` can represent this data without copying it.

```
#include <cstddef>
#include <span>

bool hasEnoughData(std::span<const std::byte> bytes,
                   std::size_t requiredSize) {
    return bytes.size() >= requiredSize;
}
```

This pattern is useful for validating a buffer's size before processing it.

However, span alone does not validate a file format, verify a protocol, or guarantee that the received data is safe. Those properties must be checked separately.

### Processing File or Network Subranges

If a buffer contains a header and a payload, you can use `first()` and `subspan()` to separate the sections without copying them.

```
#include <cstddef>
#include <span>

struct PacketParts {
    std::span<const std::byte> header;
    std::span<const std::byte> payload;
};

PacketParts splitPacket(std::span<const std::byte> packet,
                        std::size_t headerSize) {
    if (headerSize > packet.size()) {
        return {};
    }

    return {
        packet.first(headerSize),
        packet.subspan(headerSize)
    };
}
```

This example demonstrates how to divide a buffer into independent views without moving or copying the underlying data.

The validity of the returned views still depends on the lifetime and validity of the input buffer.

### Working with Numeric Data

In numerical algorithms, span can provide a common interface for data owned by different containers.

```
#include <numeric>
#include <span>

double average(std::span<const double> values) {
    if (values.empty()) {
        return 0.0;
    }

    const double total =
        std::accumulate(values.begin(), values.end(), 0.0);

    return total / static_cast<double>(values.size());
}
```

In this design, the function's behavior for empty input is explicit. In financial or scientific applications, returning zero may not be appropriate; throwing an exception or returning a result such as `std::optional` may be more suitable, depending on the API contract.

### Integrating Different Library Interfaces

Sometimes a legacy library uses pointers and sizes, while a newer part of the application is designed around span.

In such cases, a span can be adapted to the legacy interface:

```
#include <cstddef>
#include <span>

void oldApi(const unsigned char* data, std::size_t size);

void newApi(std::span<const unsigned char> bytes) {
    oldApi(bytes.data(), bytes.size());
}
```

This approach enables gradual API migration without requiring every legacy component to be rewritten at the same time.

## Performance and Runtime Cost

### The Cost of Constructing a Span

Constructing a span is generally inexpensive because it does not copy the data or allocate storage for the elements.

In many implementations, a span with dynamic extent uses information equivalent to a pointer and a size. However, the standard specifies the type's behavior and contract rather than requiring a particular memory layout.

### Effects on Inlining and Optimization

Because span is a lightweight view type, its use generally does not prevent common compiler optimizations.

A compiler may inline functions that accept spans or use static-extent information to simplify loops and element access.

However, using span is not guaranteed to be faster than using a pointer and size in every situation. Actual performance depends on the generated code, compiler, target architecture, and usage pattern.

For performance-critical decisions, benchmarking and examining the generated assembly are more reliable than guessing.

### Is Static Extent Always Faster?

Static extent provides additional compile-time information about the number of elements and may enable better optimization.

However, when the actual size varies at runtime or the API needs greater flexibility, dynamic extent is often the more appropriate design.

Choose static extent primarily because it reflects the actual data contract, not because every fixed-size type is assumed to be more efficient.

## Important Considerations for Professional Use

### Avoid Retaining Views Without a Clear Lifetime Contract

When designing an API, specify whether a span is valid only for the duration of a function call or whether it will be retained afterward.

If a view is stored in a class member, callback, or task, the lifetime responsibilities must be clearly defined.

### Avoid Using Span with Non-Contiguous Containers

The `std::span` type is designed for contiguous elements in memory. It is not suitable for representing a non-contiguous collection such as `std::list`.

If an algorithm only requires iteration and does not depend on contiguous memory, ranges and iterators often provide greater flexibility.

### Account for Invalidation

After structurally modifying an owning container, reconsider the validity of existing spans. Insertion, erasure, resizing, or memory reallocation can invalidate existing views.

### Prefer Read-Only Types When Possible

If a function is not intended to modify its elements, accepting `std::span<const T>` generally establishes a clearer and safer contract.

### Validate Sizes Before Creating Subranges

When using `first()`, `last()`, or `subspan()`, validate the size and offset before constructing the view, especially when these values originate from external data.

### Separate Ownership from Access

Span is a tool for accessing existing memory, not for ownership, resource management, or thread synchronization. If the problem requires those capabilities, use additional appropriate mechanisms.

## Important Limitations of `std::span`

Despite its broad usefulness, span has limitations that must be considered when designing a system:

- No ownership: Span is not responsible for releasing or retaining the underlying data.
- No automatic checking of every access: Using `operator[]` outside the valid range can result in undefined behavior.
- Requires contiguous memory: The elements must be accessible as a contiguous sequence.
- Cannot change the collection's size: Span cannot add elements to or remove elements from its underlying container.
- Depends on the source's lifetime: Destroying the original data or invalidating its address also invalidates the span.
- No thread-safety guarantee: Concurrent reads and writes through spans must follow the applicable synchronization rules.
- No independent ownership: An owning container is generally more appropriate when data must be retained for an extended period.

These limitations are not design flaws. They follow directly from span's purpose: providing a lightweight, non-owning view of an existing sequence.

## Conclusion

`std::span`, introduced in C++20, is a simple and effective tool for representing contiguous sequences of elements in memory. It allows developers to design APIs that accept a single view instead of separate pointer and size arguments.

Its most important applications include:

- Processing arrays, `std::array`, and `std::vector` through a common interface.
- Designing functions that do not need to own their input data.
- Splitting buffers into subranges without copying elements.
- Connecting modern APIs to legacy interfaces based on pointers and sizes.
- Expressing fixed or variable sizes through static and dynamic extents.

When choosing this type, keep several principles in mind: span does not own its data, its validity depends on the lifetime and state of the underlying memory, and it does not automatically make out-of-bounds access safe.

If a function only needs to read or modify existing data, `std::span` is often an excellent choice for API design. However, if the application requires independent ownership, resizing, long-term storage, or complex lifetime management, appropriate owning containers and ownership mechanisms should be used instead.

Ultimately, span provides the greatest value when it is combined with a clear design for ownership, lifetime, access bounds, and API contracts.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>