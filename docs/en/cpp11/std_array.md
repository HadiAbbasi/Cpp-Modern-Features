<div align="center">

[🇺🇸 English](./std_array.md) | [🇮🇷 فارسی](../../fa/cpp11/std_array.md)

</div>

---

# A Comprehensive Guide to `std::array` in C++11

## Introduction

In C++11, the C++ Standard Library introduced `std::array` to provide a standard Container for fixed-size arrays. Unlike built-in arrays, this type offers an interface compatible with many features of the Standard Library while avoiding significant additional management overhead.

Built-in arrays, such as `int values[5]`, have long been used to store a fixed number of elements. However, they have limitations when interacting with standard algorithms, passing size information to functions, and integrating with other Containers.

`std::array` addresses many of these limitations without allowing the array's size to change at runtime or dynamically allocating storage for its elements.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> values = {10, 20, 30, 40, 50};

    for (const int value : values) {
        std::cout << value << ' ';
    }

    std::cout << '\n';
}
```

The program produces the following output:

```
10 20 30 40 50
```

In this example, an array containing five `int` elements is created. The number of elements is part of the type, and the array's size cannot be changed after the array is defined.

## Structure and Core Features

### What Is std::array?

`std::array` is a class template defined in the standard `<array>` header.

Its general form is:

```
std::array<T, N>
```

In this declaration:

- `T` specifies the element type.
- `N` specifies the number of elements at compile time.
- The array's size cannot change after construction.
- Elements are stored contiguously in memory.
- The type provides the standard Container interface.

For example, `std::array<double, 10>` represents an array of ten `double` values.

```
#include <array>

int main() {
    std::array<int, 3> integers = {1, 2, 3};
    std::array<double, 4> measurements = {1.2, 2.4, 3.6, 4.8};
    std::array<char, 5> letters = {'H', 'e', 'l', 'l', 'o'};
}
```

### Fixed Size Does Not Mean Immutable Elements

An important distinction is that a fixed size does not mean the elements themselves are immutable.

```
#include <array>

int main() {
    std::array<int, 3> values = {10, 20, 30};

    values[0] = 100;
    values[1] = 200;
}
```

In this example, the array always contains three elements, but their values can change.

To prevent modification, you can use `const`:

```
#include <array>

int main() {
    const std::array<int, 3> values = {10, 20, 30};

    // values[0] = 100; // Compilation error
}
```

Also, a fixed-size array does not necessarily occupy a nonzero amount of element storage. An array containing zero elements is valid.

### The Role of the Size Parameter in the Type

In `std::array`, `N` is part of the type. Consequently, arrays with different sizes have different types.

```
#include <array>
#include <type_traits>

int main() {
    using Array3 = std::array<int, 3>;
    using Array5 = std::array<int, 5>;

    static_assert(!std::is_same<Array3, Array5>::value,
                  "The array types must be different");
}
```

This property makes it possible to inspect an array's size at compile time. However, it also means that APIs must account for the difference between arrays of different sizes.

## Defining and Initializing Arrays

### Initialization Methods

There are several common ways to initialize a `std::array`.

```
#include <array>

int main() {
    std::array<int, 4> first = {1, 2, 3, 4};

    std::array<int, 4> second{{1, 2, 3, 4}};

    std::array<int, 4> third = {};

    std::array<int, 4> fourth{};

    std::array<int, 4> fifth = {1, 2};
}
```

In this example:

- `first` is initialized with four explicit values.
- `second` uses an older nested-brace initialization form that appears in some C++11 code.
- `third` and `fourth` use value initialization, which initializes all their `int` elements to zero.
- `fifth` contains four elements. The first two are `1` and `2`, respectively, and the remaining elements are zero.

In modern code, `{}` or an explicit list of values generally provides readable initialization.

### Initialization Versus Leaving Elements Uninitialized

A common mistake is assuming that every `std::array` automatically initializes its elements to zero.

```
#include <array>

int main() {
    std::array<int, 3> initialized{};
    std::array<int, 3> uninitialized;
}
```

The `initialized` array is value-initialized, so all its elements are zero.

By contrast, `uninitialized` is a local variable with a nonzero size that is defined without an explicit initializer. Its `int` elements do not have reliable initial values, and reading them before assigning values is invalid.

For class types, the exact behavior depends on the initialization mechanism and the type's constructors.

Therefore, explicitly initializing arrays with `{}` or specified values is generally preferable in application code.

### Initializing with Fewer Values

If fewer initializers are provided than the array's size, the remaining elements are initialized from empty initializer clauses.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> values = {7, 8};

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

Output:

```
7 8 0 0 0
```

This behavior is useful for numeric types and many common applications. However, when using class types, consider the initialization semantics and the cost of initializing each remaining element.

### Zero-Sized Arrays

C++11 permits arrays with zero elements:

```
#include <array>

int main() {
    std::array<int, 0> values{};

    return static_cast<int>(values.size());
}
```

The array has a size of zero, so no elements are available for access.

Functions such as `begin()` and `end()` still provide a valid Container interface, and the resulting range is empty.

However, you must not use `front()`, `back()`, or indexed access to retrieve an element from such an array, because no element exists.

## Accessing Array Elements

### Access with the Subscript Operator

The `operator[]` subscript operator is one of the simplest ways to access elements.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 4> values = {10, 20, 30, 40};

    std::cout << values[0] << '\n';
    std::cout << values[2] << '\n';

    values[1] = 99;

    std::cout << values[1] << '\n';
}
```

Element indices start at zero. Therefore, for an array of size `N`, valid indices range from zero to `N - 1`.

Access through `operator[]` does not perform bounds checking. Using an invalid index results in undefined behavior.

Consequently, if an index is not guaranteed to be valid, you should use an appropriate bounds-checking mechanism.

### Bounds-Checked Access with at

The `at()` function checks the index before returning the requested element.

```
#include <array>
#include <iostream>
#include <stdexcept>

int main() {
    std::array<int, 3> values = {10, 20, 30};

    try {
        std::cout << values.at(1) << '\n';
        std::cout << values.at(5) << '\n';
    } catch (const std::out_of_range& error) {
        std::cout << "Index is out of range\n";
    }
}
```

In this example, accessing index `5` throws a `std::out_of_range` exception.

The main differences between the two approaches are:

- `operator[]` is appropriate when the index is already known to be valid.
- `at()` is useful when an index comes from external input or a calculation whose validity is uncertain.

`operator[]` is a common choice in performance-sensitive code, but omitting bounds checking should be based on an actual guarantee rather than an assumption.

### Accessing the First and Last Elements

The `front()` and `back()` functions access the first and last elements, respectively.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 4> values = {10, 20, 30, 40};

    std::cout << values.front() << '\n';
    std::cout << values.back() << '\n';

    values.front() = 100;
    values.back() = 400;
}
```

These functions improve readability when the intent is to access the first or last element rather than a particular numeric index.

Importantly, neither function can be used validly on an empty array.

### Accessing the Underlying Storage with data

The `data()` function returns a pointer to the first element of the array.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 3> values = {10, 20, 30};

    int* pointer = values.data();

    std::cout << pointer[0] << '\n';
    std::cout << pointer[1] << '\n';
    std::cout << pointer[2] << '\n';
}
```

This feature is useful when interacting with legacy C APIs or libraries that accept a pointer and an element count.

For example, if an API requires an `int*` and a length, you can pass `data()` and `size()`.

However, the returned pointer does not have ownership independent of the array. Also, for a zero-sized array, you must not assume that the returned pointer can be used to access an element.

## Managing Size and Capacity

### Checking the Size with size

The `size()` function returns the number of elements in the array.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> values{};

    std::cout << values.size() << '\n';
}
```

Output:

```
5
```

The return type of `size()` is `std::size_t`.

Because a `std::array` has a size determined at compile time, the returned value never changes for a given instance.

### Checking Whether the Array Is Empty with empty

The `empty()` function determines whether an array contains any elements.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 0> emptyArray{};
    std::array<int, 3> nonEmptyArray{};

    std::cout << std::boolalpha;
    std::cout << emptyArray.empty() << '\n';
    std::cout << nonEmptyArray.empty() << '\n';
}
```

Output:

```
true
false
```

This function is useful in generic code and improves readability by avoiding assumptions about the presence of elements.

### Understanding max_size

The `max_size()` function reports the maximum number of elements the instance can hold.

For `std::array`, the size is already specified by the type. Therefore, in typical standard-library implementations, `max_size()` returns the same value as `size()`.

This function is primarily useful in generic code that works with different Container types.

## Iterating Over Elements with Iterators

### Using begin and end

`std::array` supports standard iterators and can therefore be used with algorithms from the Standard Library.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> values = {1, 2, 3, 4, 5};

    for (auto iterator = values.begin();
         iterator != values.end();
         ++iterator) {
        std::cout << *iterator << ' ';
    }
}
```

The `begin()` and `end()` functions return an iterator to the beginning of the range and an iterator one position past the last element, respectively.

The iterator returned by `end()` must not be dereferenced. It is used to identify the end of the range.

### Reverse Iteration with reverse_iterator

The `rbegin()` and `rend()` functions support reverse iteration.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 4> values = {10, 20, 30, 40};

    for (auto iterator = values.rbegin();
         iterator != values.rend();
         ++iterator) {
        std::cout << *iterator << ' ';
    }
}
```

Output:

```
40 30 20 10
```

This approach enables reverse traversal without manually calculating indices.

### Range-Based for Loops

In many cases, a range-based `for` loop is the most readable choice.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 4> values = {10, 20, 30, 40};

    for (const int value : values) {
        std::cout << value << ' ';
    }
}
```

If the elements need to be modified, you can use a reference:

```
#include <array>

int main() {
    std::array<int, 4> values = {1, 2, 3, 4};

    for (int& value : values) {
        value *= 2;
    }
}
```

By contrast, using a local loop variable without a reference copies each element. This usually has negligible cost for simple types, but it may be inefficient for large class objects or unsuitable for non-copyable types.

## Using Standard Algorithms

### Sorting Elements with sort

Because `std::array` supports random-access iterators, it can be used with the `std::sort` algorithm.

```
#include <algorithm>
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> values = {50, 10, 40, 20, 30};

    std::sort(values.begin(), values.end());

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

Output:

```
10 20 30 40 50
```

The `std::sort` algorithm has been available in the Standard Library since C++98 and is not specific to `std::array`.

### Searching with find

The `std::find` algorithm searches a range for a value.

```
#include <algorithm>
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> values = {10, 20, 30, 40, 50};

    auto iterator = std::find(
        values.begin(),
        values.end(),
        30
    );

    if (iterator != values.end()) {
        std::cout << "Value found: " << *iterator << '\n';
    }
}
```

If the value is not found, the returned iterator equals `end()`.

### Calculating a Sum with accumulate

You can use `std::accumulate` to calculate the sum of the elements.

```
#include <array>
#include <iostream>
#include <numeric>

int main() {
    std::array<int, 4> values = {10, 20, 30, 40};

    const int total = std::accumulate(
        values.begin(),
        values.end(),
        0
    );

    std::cout << total << '\n';
}
```

Output:

```
100
```

An important detail is that the initial value's type in `std::accumulate` affects the type used for accumulation. For example, if the elements are `int` and the initial value is `0`, accumulation occurs using `int`. If the integer sum overflows its representable range, undefined behavior may result.

When the total might exceed the range of `int`, consider using a more appropriate type for the initial value.

## Comparing std::array with Built-In Arrays

### Structural Differences

Built-in arrays and `std::array` are both suitable for storing a fixed number of elements, but they differ in important aspects of their programming interfaces.

| Feature                                | Built-in array                       | `std::array`               |
| -------------------------------------- | ------------------------------------ | -------------------------- |
| Size determined at compile time        | Yes                                  | Yes                        |
| Resizable                              | No                                   | No                         |
| Contiguous element storage             | Yes                                  | Yes                        |
| `size()` member function               | No                                   | Yes                        |
| `at()` member function                 | No                                   | Yes                        |
| Direct support for standard iterators  | Through pointers or helper functions | Yes                        |
| Whole-array assignment with `=`        | Not directly supported               | Supported                  |
| Compatibility with standard algorithms | Through suitable pointers and ranges | Through standard iterators |
| Standard Container type                | No                                   | Yes                        |

### Assignment and Copying Differences

One important advantage of `std::array` is that it supports whole-array assignment.

```
#include <array>

int main() {
    std::array<int, 3> source = {1, 2, 3};
    std::array<int, 3> destination{};

    destination = source;
}
```

In this example, the elements of `source` are copied into `destination`.

By contrast, built-in arrays do not support direct whole-array assignment. Copying their elements requires a loop, `std::copy`, or another appropriate mechanism.

### Differences in Pointer Conversion

Built-in arrays decay to pointers to their first elements in many expressions. This behavior, known as array-to-pointer decay, can cause size information to be lost when passing arrays to certain APIs.

However, `std::array` does not automatically convert to a pointer.

```
#include <array>

void process(int* data, std::size_t count);

int main() {
    std::array<int, 3> values = {1, 2, 3};

    process(values.data(), values.size());
}
```

In this example, the pointer conversion is explicit, and the array size is passed separately.

## Comparing std::array with std::vector

### Differences in Size and Memory Allocation

`std::vector` is designed to store a sequence of elements whose size can change, whereas `std::array` has a fixed size.

| Feature                                 | `std::array`                          | `std::vector`                       |
| --------------------------------------- | ------------------------------------- | ----------------------------------- |
| Size                                    | Fixed                                 | Resizable                           |
| Number of elements                      | Determined at compile time            | Usually determined at runtime       |
| Memory management                       | Element storage is part of the object | Dynamically manages element storage |
| Supports `push_back`                    | No                                    | Yes                                 |
| Can increase its size                   | No                                    | Yes                                 |
| Suitable for a fixed number of elements | Very suitable                         | Suitable, but often unnecessary     |
| Random access                           | Yes                                   | Yes                                 |

With `std::array`, the elements are stored in storage belonging to the array object itself. With `std::vector`, the object typically maintains management information and dynamically manages its element storage.

However, the C++ Standard does not guarantee that a particular Container must reside on the stack or heap. Storage location depends on where the object is defined, how it is allocated, and implementation details.

### Choosing the Right Container for Different Applications

`std::array` is appropriate when the number of elements is fixed by the program's design, such as a 3-by-3 matrix, a collection of constant coefficients, or a data packet with a specified number of fields.

By contrast, if the number of elements depends on user input, a file, network data, or runtime calculations, `std::vector` is generally a better choice.

A fixed size alone is not sufficient reason to choose `std::array`. The size must be known at compile time, and the ownership model and lifetime requirements must also suit this choice.

## Passing std::array to Functions

### Passing an Array by Reference

Because the size is part of the `std::array` type, you can define a function that accepts an array of a specific size as part of its parameter type.

```
#include <array>
#include <iostream>

void printValues(const std::array<int, 3>& values) {
    for (int value : values) {
        std::cout << value << ' ';
    }

    std::cout << '\n';
}

int main() {
    std::array<int, 3> values = {10, 20, 30};

    printValues(values);
}
```

Passing a const reference avoids copying the array and prevents the function from modifying its elements.

The limitation is that this function accepts only an array of the exact type `std::array<int, 3>`.

### Accepting Different Sizes with a Function Template

If a function must accept arrays of different sizes, you can make the size a template parameter.

```
#include <array>
#include <iostream>

template <typename T, std::size_t N>
void printArray(const std::array<T, N>& values) {
    for (const T& value : values) {
        std::cout << value << ' ';
    }

    std::cout << '\n';
}

int main() {
    std::array<int, 3> integers = {1, 2, 3};
    std::array<double, 2> decimals = {1.5, 2.5};

    printArray(integers);
    printArray(decimals);
}
```

With this approach, the compiler deduces the element type and array size from the function argument.

This pattern is particularly useful when writing generic functions that work with different `std::array` types.

### Accepting Arrays Whose Size Is Not Known at Compile Time

Sometimes a function needs to operate on a range of elements regardless of the array's size. In such cases, tying the function's signature to a specific size may be inconvenient.

In C++20, `std::span` was introduced to represent a non-owning view of a contiguous range of elements.

```
#include <array>
#include <span>
#include <iostream>

void printValues(std::span<const int> values) {
    for (int value : values) {
        std::cout << value << ' ';
    }

    std::cout << '\n';
}

int main() {
    std::array<int, 3> first = {1, 2, 3};
    std::array<int, 5> second = {4, 5, 6, 7, 8};

    printValues(first);
    printValues(second);
}
```

This example requires C++20 or later and is not part of C++11.

For projects that must remain compatible with C++11, alternatives include function templates, pointers accompanied by lengths, or another suitable generic interface.

## Practical Applications of std::array

### Storing Coefficients and Fixed Values

For a collection of values whose number is known by design, `std::array` is a simple and predictable choice.

```
#include <array>
#include <iostream>

int main() {
    const std::array<double, 4> weights = {
        0.1, 0.2, 0.3, 0.4
    };

    double result = 0.0;

    for (std::size_t i = 0; i < weights.size(); ++i) {
        result += weights[i] * static_cast<double>(i + 1);
    }

    std::cout << result << '\n';
}
```

In this example, the number of coefficients is fixed, so changing the array's size at runtime is unnecessary.

### Representing a Three-Dimensional Vector

A three-dimensional vector can be represented by an array containing three values.

```
#include <array>
#include <iostream>

using Vector3 = std::array<double, 3>;

Vector3 add(const Vector3& first, const Vector3& second) {
    Vector3 result{};

    for (std::size_t i = 0; i < result.size(); ++i) {
        result[i] = first[i] + second[i];
    }

    return result;
}

int main() {
    const Vector3 first = {1.0, 2.0, 3.0};
    const Vector3 second = {4.0, 5.0, 6.0};

    const Vector3 result = add(first, second);

    std::cout << result[0] << ' '
              << result[1] << ' '
              << result[2] << '\n';
}
```

In this application, `std::array` manages ownership simply, and returning a result from a function is straightforward.

However, if the application requires complex vector operations, specialized mathematical semantics, or specific optimization capabilities, a dedicated type or mathematical library may offer a better design.

### Representing a Fixed-Dimension Matrix

For a small matrix with known dimensions, you can use a nested array.

```
#include <array>
#include <iostream>

int main() {
    std::array<std::array<int, 3>, 2> matrix = {{
        {{1, 2, 3}},
        {{4, 5, 6}}
    }};

    for (const auto& row : matrix) {
        for (const int value : row) {
            std::cout << value << ' ';
        }

        std::cout << '\n';
    }
}
```

This structure represents a matrix with two rows and three columns.

Each row is a `std::array<int, 3>` with its own type, and the complete matrix contains two elements of that row type.

### Using std::array for Protocols and Data Packets

In networking, binary files, and protocols with fixed-length fields, `std::array` can be suitable for storing fixed-size buffers.

```
#include <array>
#include <cstdint>

int main() {
    std::array<std::uint8_t, 16> identifier{};

    identifier[0] = 0x10;
    identifier[1] = 0x20;
}
```

This structure is suitable for storing sixteen bytes of data, but it does not by itself guarantee that its memory layout matches an external protocol or a particular binary format.

When designing binary protocols, byte order, serialization format, input validation, and the precise representation of data must be handled separately.

## Memory and Performance Considerations

### Contiguous Element Storage

One of the defining features of `std::array` is that its elements are stored contiguously in memory.

This allows traversal through a pointer to the first element and enables many algorithms to access elements efficiently.

However, actual performance depends on the element types, access patterns, data size, cache behavior, and the code generated by the compiler.

Therefore, using `std::array` does not guarantee that every operation will be faster than its equivalent using `std::vector`.

### The Cost of Copying an Array

Copying a `std::array` copies its elements.

```
#include <array>

int main() {
    std::array<int, 100> source{};

    std::array<int, 100> destination = source;
}
```

In this example, the destination array receives an independent copy of the source array's values.

For class-type elements, the cost of copying depends on the behavior of their copy constructors or assignment operators.

For large arrays, passing them by const reference usually avoids unnecessary copying.

### Large Arrays and Stack Space

Defining a large array as a local variable may consume a substantial amount of stack space.

```
#include <array>

int main() {
    std::array<int, 1000000> values{};
}
```

This example requires approximately four million bytes for its elements, assuming that each `int` occupies four bytes. The exact size is implementation-dependent.

If such an array is defined in an environment with a limited stack, execution may fail.

For large collections whose size is determined at runtime or that require dynamic allocation, `std::vector` is generally a more suitable choice. In specific cases, a `std::array` can also be stored in static storage or within another object with an appropriate lifetime.

### No Dynamic Allocation by the Array Itself

`std::array` does not use separate dynamic allocation for its own element storage in the way that `std::vector` typically does.

This does not mean its elements cannot own dynamic resources. For example, if the elements are `std::string` or `std::unique_ptr` objects, each element may manage its own resources.

It is therefore important to distinguish the storage occupied by the array's elements from the resources managed by the objects stored within it.

## Swapping and Assignment

### Swapping Contents with swap

The `swap()` member function exchanges the contents of two arrays of the same type.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 3> first = {1, 2, 3};
    std::array<int, 3> second = {4, 5, 6};

    first.swap(second);

    for (int value : first) {
        std::cout << value << ' ';
    }

    std::cout << '\n';
}
```

After this operation, the elements of the two arrays are exchanged.

Because the size is part of the array's type, arrays of different sizes cannot be swapped directly through this member function.

### swap Versus Swapping Pointers

For `std::array`, swapping contents does not mean exchanging pointers to owned data. The array stores its elements in storage belonging to the object itself.

Consequently, `swap()` for `std::array` should not be treated as equivalent to simply exchanging pointers or transferring ownership of storage, as can occur with some dynamic Containers.

## Comparison and Relational Operators

In C++11, `std::array` supports comparison operators, provided that the element type satisfies the necessary comparison requirements.

```
#include <array>
#include <iostream>

int main() {
    std::array<int, 3> first = {1, 2, 3};
    std::array<int, 3> second = {1, 2, 4};

    std::cout << std::boolalpha;
    std::cout << (first == second) << '\n';
    std::cout << (first < second) << '\n';
}
```

Output:

```
false
true
```

The `==` operator compares corresponding elements and returns `true` only if all corresponding elements are equal.

Ordering comparisons are lexicographical: elements are compared from the beginning, and the first pair of unequal elements determines the result.

This capability is useful for fixed-size keys and certain sorting applications. However, for custom element types, ensure that their comparison semantics match the intended behavior of the program.

## Using std::get and the Tuple Interface

### Accessing Elements with std::get

`std::array` supports the standard Tuple interface. Consequently, in C++11, you can use `std::get` to access an element at a compile-time index.

```
#include <array>
#include <iostream>
#include <tuple>

int main() {
    std::array<int, 3> values = {10, 20, 30};

    std::cout << std::get<0>(values) << '\n';
    std::cout << std::get<1>(values) << '\n';
    std::cout << std::get<2>(values) << '\n';
}
```

The index passed to `std::get` is a template argument and must be known at compile time.

This function is useful in generic code and algorithms that require type information at compile time.

If the requested index is out of range, the program fails to compile.

### Using tuple_size and tuple_element

In C++11, the standard type traits `std::tuple_size` and `std::tuple_element` can be used to extract structural information about an array at compile time.

```
#include <array>
#include <tuple>
#include <type_traits>

int main() {
    using ArrayType = std::array<double, 4>;

    static_assert(
        std::tuple_size<ArrayType>::value == 4,
        "Unexpected array size"
    );

    static_assert(
        std::is_same<
            std::tuple_element<0, ArrayType>::type,
            double
        >::value,
        "Unexpected element type"
    );
}
```

These capabilities are useful when designing templates and libraries that need access to element types and array sizes at compile time.

## Using std::to_array in Newer C++ Standards

### Creating an Array with std::to_array

The `std::to_array` function was introduced in C++20 and is not part of C++11.

It can create a `std::array` from a built-in array or an initializer list, deducing the element type and number of elements.

```
#include <array>
#include <iostream>

int main() {
    auto values = std::to_array({1, 2, 3, 4});

    std::cout << values.size() << '\n';
}
```

This feature is useful when you want to avoid repeating the element count manually in the type declaration.

However, if a project must be compiled with a compiler and standard library that support only C++11, use the usual `std::array` initialization techniques instead.

## Limitations and Common Mistakes

### Assuming Elements Are Zero-Initialized

Defining an array without an initializer does not always mean its elements are initialized to zero. To avoid reading values that have not been initialized, explicitly initialize local arrays.

### Accessing an Out-of-Range Index

Using `operator[]` with an invalid index results in undefined behavior. When the validity of an index is uncertain, using `at()` or checking the bounds manually is safer.

### Calling Functions on an Empty Array

`front()` and `back()` cannot be used on an array with zero elements. Ensure that an element exists before accessing either function.

### Confusing Fixed Size with Immutable Values

A fixed array size does not prevent changes to its elements. If modification must not be allowed, use `const` or an appropriate interface that restricts access.

### Passing Large Arrays by Value

Passing a large array by value can copy many elements. For functions that do not need to modify the array, passing it by const reference is generally preferable.

### Assuming the Array Can Be Resized

`std::array` does not support operations such as `push_back`, `resize`, or `clear`. If the sequence must change size, use a suitable Container such as `std::vector`.

### Assuming Arrays with the Same Element Type Have the Same Type

Arrays such as `std::array<int, 3>` and `std::array<int, 4>` are different types, even though their element types are identical. This distinction matters when defining function parameters and templates.

### Assuming a Specific Binary Layout

Contiguous element storage does not guarantee a portable serialization format or a particular binary representation. When exchanging data with other systems, define the data format and conversion rules explicitly.

## Design Considerations and Container Selection

### When to Choose std::array

Using `std::array` is generally appropriate when:

- The number of elements is known at compile time.
- The array's size must remain unchanged throughout the object's lifetime.
- Random access and contiguous memory are important.
- Compatibility with standard algorithms is required.
- Data ownership should be simple and predictable.

### When Not to Choose std::array

If the sequence's size must be determined at runtime, the number of elements changes frequently, or the data set is very large, another Container is often more appropriate.

Similarly, if the elements represent different semantic roles, a dedicated `struct` or class may provide a clearer and safer design.

For example, if an object contains a name, an age, and an active status, defining a dedicated type is generally better than storing the three values in an array and accessing them through numeric indices.

## Summary

`std::array` in C++11 provides a standard solution for storing a fixed-size collection of elements of the same type. It preserves the advantages of built-in arrays, such as contiguous storage and a fixed size, while providing features such as `size()`, `at()`, iterators, direct assignment, and compatibility with standard algorithms.

In professional software development, the choice of this type should be based on the application's actual requirements. If the data size is known in advance and must not change, `std::array` is often a simple, predictable, and suitable choice. If the size needs to change, `std::vector` is generally a better option.

Correct initialization, bounds checking, avoiding unnecessary copies, and understanding the distinction between fixed size and immutable elements all contribute to safer and more effective use of this Container.

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>