<div align="center">

[🇺🇸 English](./std_begin__std_end.md) | [🇮🇷 فارسی](../../en/cpp11/std_begin__std_end.md)

</div>

---

# Understanding `std::begin` and `std::end` in C++11

## Introduction

The `std::begin` and `std::end` functions are important Standard Library utilities introduced in C++11 that provide a uniform way to access the beginning and end of data ranges. They allow developers to write generic code for iterating over elements, searching, sorting, and processing data without needing to know the implementation details of a particular container.

Their importance becomes especially apparent when an algorithm needs to work with different data types, such as built-in arrays, `std::vector`, `std::array`, and other standard containers.

For example, the following code is valid in C++11:

```
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values = {10, 20, 30, 40};

    auto first = std::begin(values);
    auto last = std::end(values);

    std::for_each(first, last, [](int value) {
        std::cout << value << ' ';
    });

    std::cout << '\n';
}
```

The program produces the following output:

```
10 20 30 40
```

In this example, `std::begin` returns an iterator to the first element, while `std::end` returns an iterator to the position immediately after the last element.

An important distinction is that `std::end` does not point to the last element. Instead, it represents the position at which iteration must stop.

## Understanding Iterators and Data Ranges

### The Iterator Concept

An iterator in C++ provides a standardized way to access elements in a collection of data. Depending on its type, an iterator may support reading, modifying, traversing, or accessing elements.

Consider the following container:

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers = {10, 20, 30};

    auto it = numbers.begin();

    std::cout << *it << '\n';

    ++it;

    std::cout << *it << '\n';
}
```

The output is:

```
10
20
```

In this program, the `begin()` member function of `std::vector` returns an iterator to the first element. The `*` operator accesses the value of the referenced element, and the `++` operator advances the iterator to the next position.

### Understanding Half-Open Ranges

Standard C++ algorithms generally represent ranges using the half-open interval notation:

```
[first, last)
```

This notation means that the range includes the element referenced by `first` but excludes the position represented by `last`.

For example, if a `std::vector` contains four elements, the range defined by `begin()` and `end()` includes all four elements.

This design provides several benefits:

- The number of elements in a range can be obtained from the difference between compatible iterators when their iterator category supports that operation.
- An empty range is represented by two equal iterators.
- Adjacent ranges are easy to combine because the end of one range can equal the beginning of the next.
- Algorithms can use a consistent termination condition without requiring special handling for the last element.

A typical iteration pattern for a valid array or container is:

```
for (auto it = first; it != last; ++it) {
    // Process the current element
}
```

The condition `it != last` is checked before accessing the current element. Consequently, the loop never dereferences the iterator representing the end position.

## Introducing `std::begin`

### Purpose and Behavior

The `std::begin` function was introduced in C++11 in the standard `<iterator>` header. It returns an iterator to the beginning of a range.

Its general usage is:

```
auto first = std::begin(container);
```

The actual type of `first` is deduced from the function's return type. Using `auto` eliminates the need to explicitly specify the iterator type.

The `std::begin` function can be used with standard containers, built-in arrays, and other types supported by suitable overloads.

### Differences Between `std::begin` and `begin()`

The difference between these approaches lies in where the function is defined and how it is called.

Using a container member function:

```
auto first = values.begin();
```

Using the standard free function:

```
auto first = std::begin(values);
```

The first approach directly calls the container's member function. The second uses a standard free function that provides overloads for several different data types.

This distinction is particularly important for built-in arrays, which do not have a `begin()` member function but can be handled by `std::begin`.

### Working with `std::vector`

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values = {5, 10, 15};

    auto first = std::begin(values);

    std::cout << *first << '\n';
}
```

The output is:

```
5
```

In this case, `std::begin(values)` uses the corresponding `values.begin()` member function and returns the iterator associated with the first element.

### Working with Built-In Arrays

```
#include <iostream>
#include <iterator>

int main() {
    int values[] = {100, 200, 300};

    auto first = std::begin(values);

    std::cout << *first << '\n';
}
```

The output is:

```
100
```

In this example, the built-in array has no member functions. Nevertheless, `std::begin` returns a pointer to its first element.

For a built-in array, the return type is typically `T*`, where `T` is the element type.

## Introducing `std::end`

### Purpose and Behavior

The `std::end` function was also introduced in C++11 in the standard `<iterator>` header. It returns an iterator to the position immediately after the last element of a range.

Its general usage is:

```
auto last = std::end(container);
```

This iterator is used to define the endpoint of a range in loops and standard algorithms.

### Understanding the End Position

```
#include <iostream>
#include <iterator>

int main() {
    int values[] = {10, 20, 30};

    auto first = std::begin(values);
    auto last = std::end(values);

    while (first != last) {
        std::cout << *first << ' ';
        ++first;
    }

    std::cout << '\n';
}
```

The output is:

```
10 20 30
```

In this example, `last` represents the position after the element `30`. Therefore, the loop condition prevents the end iterator from being dereferenced.

### Why the End Position Follows the Last Element

In the standard iterator model, the end position is a boundary rather than an element that can be read.

For an array containing three elements, incrementing `first` three times moves it to `last`.

By contrast, attempting to read `*last` in this situation results in undefined behavior.

This rule is not limited to arrays. For standard container iterators, the end iterator must not be assumed to be dereferenceable either.

## Understanding the Standard Implementation

### Behavior with Standard Containers

For standard container types, the overloads of `std::begin` and `std::end` use the corresponding member functions.

The general idea can be illustrated as follows:

```
template <class Container>
auto begin(Container& container) -> decltype(container.begin());

template <class Container>
auto end(Container& container) -> decltype(container.end());
```

This code is only a conceptual illustration, not the actual Standard Library implementation. The real implementation may contain additional details, and the standard free functions belong to the `std` namespace.

The important point is that the iterator type is derived from the container's member function. Consequently, if a container uses a custom iterator type, the standard functions can preserve that type.

### Behavior with Built-In Arrays

Built-in arrays do not have member functions. To support them, dedicated array overloads are provided.

For example, the idea of returning a pointer to the beginning of an array can be illustrated in C++11 as follows:

```
template <class T, std::size_t N>
T* begin(T (&array)[N]) noexcept;
```

This is also a conceptual illustration of the function signature, not a definition that should replace the standard function.

The parameter is declared as a reference to an array. This allows the array's size to be preserved through the template parameter `N`.

As a result, `std::begin` can work with the array's size known at compile time without allowing the array to decay into a pointer.

### Behavior with Empty Ranges

A zero-length built-in array is not a valid standard array in C++11. However, containers such as `std::vector` can be empty.

For an empty container, `begin()` and `end()` are equal, and a conventional iteration loop executes zero times.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values;

    auto first = std::begin(values);
    auto last = std::end(values);

    if (first == last) {
        std::cout << "The range is empty\n";
    }
}
```

The output is:

```
The range is empty
```

This behavior allows algorithms that operate on standard ranges to handle many empty inputs without additional special-case conditions.

## Working with `const` and Mutable Iterators

### Behavior with Non-`const` Objects

When a non-`const` container is passed to `std::begin`, the function typically returns the container's mutable iterator.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values = {1, 2, 3};

    auto first = std::begin(values);

    *first = 100;
}
```

In this example, the first element changes from `1` to `100`.

### Behavior with `const` Objects

When a container is `const`, the corresponding function returns an iterator that provides constant access.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    const std::vector<int> values = {1, 2, 3};

    auto first = std::begin(values);

    std::cout << *first << '\n';
}
```

The output is:

```
1
```

In this situation, the iterator does not allow the element to be modified through it. Attempting to change the value through this iterator would make the program ill-formed.

### Comparing `cbegin` and `cend`

C++11 also provides `std::cbegin` and `std::cend`, which are used to access ranges through constant iterators.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values = {10, 20, 30};

    auto first = std::cbegin(values);
    auto last = std::cend(values);

    for (auto it = first; it != last; ++it) {
        // Read-only access
        int value = *it;
        (void)value;
    }
}
```

In this example, even though `values` is non-`const`, the iterator type prevents elements from being modified through the iterator.

Using `cbegin` and `cend` is useful when the intention to avoid modifying elements should be explicit.

However, a constant iterator does not necessarily make the underlying data completely immutable. The restriction applies to the access provided through that iterator.

## Using `std::begin` and `std::end` with Standard Algorithms

### Searching with `std::find`

A common application of these functions is passing ranges to Standard Library algorithms.

```
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values = {4, 8, 12, 16};

    auto it = std::find(
        std::begin(values),
        std::end(values),
        12
    );

    if (it != std::end(values)) {
        std::cout << "Value found: " << *it << '\n';
    }
}
```

The output is:

```
Value found: 12
```

The `std::find` function searches from the beginning of the range up to, but not including, the end position. If the requested element is not found, it returns the end iterator.

In this example, `std::end(values)` is used to determine whether the search succeeded.

### Sorting with `std::sort`

For containers such as `std::vector`, which provide suitable random-access iterators, `std::sort` can be used.

```
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values = {9, 2, 7, 1, 5};

    std::sort(std::begin(values), std::end(values));

    for (auto it = std::begin(values);
         it != std::end(values);
         ++it) {
        std::cout << *it << ' ';
    }

    std::cout << '\n';
}
```

The output is:

```
1 2 5 7 9
```

An important point is that having `std::begin` and `std::end` available does not mean every container can be sorted using `std::sort`.

The `std::sort` algorithm requires random-access iterators. Therefore, the bidirectional iterators provided by `std::list` are not suitable for `std::sort`. To sort a `std::list`, use its member function `sort()` instead.

### Calculating the Sum of Elements

Data can also be processed using `std::accumulate`, which is declared in the `<numeric>` header.

```
#include <iostream>
#include <iterator>
#include <numeric>
#include <vector>

int main() {
    std::vector<int> values = {10, 20, 30, 40};

    int total = std::accumulate(
        std::begin(values),
        std::end(values),
        0
    );

    std::cout << total << '\n';
}
```

The output is:

```
100
```

This approach reduces the algorithm's dependence on a particular container type. The algorithm only needs a compatible range of iterators.

## Using Built-In Arrays in Generic Functions

### Preserving Array Size

One of the most important advantages of `std::begin` and `std::end` is that they allow built-in arrays to be processed without unintentionally converting them to pointers.

For example, the following code passes an array to a function by reference:

```
#include <iostream>
#include <iterator>

template <class T, std::size_t N>
void printArray(const T (&values)[N]) {
    for (auto it = std::begin(values);
         it != std::end(values);
         ++it) {
        std::cout << *it << ' ';
    }

    std::cout << '\n';
}

int main() {
    int numbers[] = {3, 6, 9, 12};

    printArray(numbers);
}
```

The output is:

```
3 6 9 12
```

In this example, the array size is preserved through the template parameter `N`, allowing the function to iterate over the entire array.

This pattern is useful for generic functions because, unlike parameters that cause an array to decay into a pointer, it preserves the array's size at compile time.

### Limitations of Array-to-Pointer Decay

Consider a function with the following parameter:

```
void process(int values[]);
```

The parameter `values` is actually of type `int*`. Consequently, the size of the original array cannot be determined from this parameter.

In this situation, `std::end(values)` cannot recover the original array's size because `values` is no longer an array; it is a pointer.

When processing built-in arrays, prefer a reference-to-array parameter or another mechanism that explicitly preserves or communicates the range's size whenever possible.

## Working with Different Container Types

### Using `std::array`

The `std::array` type, introduced in C++11, is a fixed-size container that provides a standard iterator interface.

```
#include <iostream>
#include <iterator>
#include <array>

int main() {
    std::array<int, 4> values = {10, 20, 30, 40};

    auto first = std::begin(values);
    auto last = std::end(values);

    while (first != last) {
        std::cout << *first << ' ';
        ++first;
    }

    std::cout << '\n';
}
```

The output is:

```
10 20 30 40
```

Using `std::begin` and `std::end` here allows the code to depend on the general iterator interface rather than on implementation details specific to `std::array`.

### Using `std::string`

Standard strings also provide iterators and can be used with these functions.

```
#include <iostream>
#include <iterator>
#include <string>

int main() {
    std::string text = "Hello";

    for (auto it = std::begin(text);
         it != std::end(text);
         ++it) {
        std::cout << *it << ' ';
    }

    std::cout << '\n';
}
```

The output is:

```
H e l l o
```

In this example, iteration operates on `char` elements. For UTF-8 strings, however, each `char` is not necessarily a complete Unicode character. Therefore, this iteration pattern does not by itself guarantee traversal by Unicode code point or user-perceived character.

### Using `std::list`

The `std::list` container also provides iterators, but its iterators do not support random access.

```
#include <iostream>
#include <iterator>
#include <list>

int main() {
    std::list<int> values = {10, 20, 30};

    for (auto it = std::begin(values);
         it != std::end(values);
         ++it) {
        std::cout << *it << ' ';
    }

    std::cout << '\n';
}
```

The output is:

```
10 20 30
```

This example demonstrates that the standard functions can provide ranges for containers with different iterator types without restricting the iterator type to pointers.

## Understanding ADL and Custom `begin` and `end` Functions

### The Argument-Dependent Lookup Mechanism

Argument-Dependent Lookup (ADL) in C++ allows unqualified function calls to consider functions declared in namespaces associated with the argument types.

This mechanism is important when designing generic code because some custom types define free `begin` and `end` functions in their own namespaces rather than providing member functions.

For example, consider the following custom type:

```
namespace custom {

struct Range {
    int* first;
    int* last;
};

int* begin(Range& range) {
    return range.first;
}

int* end(Range& range) {
    return range.last;
}

}
```

The functions associated with this type can be called using ADL:

```
#include <iostream>

namespace custom {

struct Range {
    int* first;
    int* last;
};

int* begin(Range& range) {
    return range.first;
}

int* end(Range& range) {
    return range.last;
}

}

int main() {
    int values[] = {10, 20, 30};

    custom::Range range{values, values + 3};

    using std::begin;
    using std::end;

    for (auto it = begin(range);
         it != end(range);
         ++it) {
        std::cout << *it << ' ';
    }

    std::cout << '\n';
}
```

The output is:

```
10 20 30
```

In this example, the expressions `begin(range)` and `end(range)` can find the functions declared in the `custom` namespace through ADL.

It is important to note that `custom::begin` and `custom::end` in this example are custom free functions, not part of the Standard Library.

### Differences Between `std::begin` and the ADL Pattern

A direct call to `std::begin(range)` considers overloads in the `std` namespace. It does not, by itself, select the custom free function `custom::begin` as an overload.

For this reason, the following pattern is often preferable in generic code that must support both standard and custom types:

```
using std::begin;
using std::end;

auto first = begin(range);
auto last = end(range);
```

This pattern makes the standard overloads available while allowing ADL to find functions associated with the argument type.

This distinction can be decisive when designing general-purpose APIs. However, for ordinary code that works exclusively with standard containers, directly using `std::begin` and `std::end` is straightforward and clear.

## Important Considerations for Iterator and Range Validity

### Iterator Validity

Correct use of `std::begin` and `std::end` requires a valid iterator range. These functions do not guarantee that the returned iterators remain valid throughout the program's execution.

For example, operations on `std::vector`, such as increasing its capacity through `reserve` or adding elements under certain conditions, may invalidate existing iterators.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values = {1, 2, 3};

    auto first = std::begin(values);
    auto last = std::end(values);

    values.push_back(4);

    // Do not assume that first and last are still valid.
}
```

In this example, adding an element to `std::vector` may cause memory reallocation. If reallocation occurs, the previous iterators become invalid. Even if reallocation does not occur, the previous end iterator is no longer the end of the updated range because the container's size has changed.

Therefore, after operations that may invalidate iterators, their validity must be checked according to the rules of the specific container.

### Using Compatible Beginning and End Iterators

The beginning and end iterators must belong to the same valid and compatible range.

Comparing iterators from unrelated containers or calculating the distance between them is not valid unless the standard explicitly permits the operation in the relevant circumstances.

For example, do not use the beginning iterator of one `std::vector` and the end iterator of another `std::vector` to define a single range.

### Avoiding Dereferencing the End Iterator

The position returned by `std::end` is a boundary and must not be treated as an actual element.

The correct iteration pattern is:

```
for (auto it = std::begin(values);
     it != std::end(values);
     ++it) {
    // Process the current element
}
```

By contrast, directly accessing `*std::end(values)` for an ordinary range is invalid and can result in undefined behavior.

### Considering Object Lifetime

An iterator generally depends on the lifetime of its container. Once the container is destroyed, its iterators must no longer be used to access its elements.

The same principle applies to local arrays: pointers obtained from `std::begin` and `std::end` cannot be used after the array's lifetime has ended.

## Important Differences Between C++ Standards

### C++98 and C++03

In standards older than C++11, the standard free functions `std::begin` and `std::end` were not available in the form introduced by C++11.

However, many standard containers already provided the member functions `begin()` and `end()`. Consequently, code that calls these member functions directly can also be valid under these older standards.

### C++11

C++11 introduced the free functions `std::begin` and `std::end`, providing a unified way to work with built-in arrays and standard containers.

The `std::array` type and the `std::cbegin` and `std::cend` functions are also important related features introduced in this standard.

### C++14 and Later Standards

In C++14 and subsequent standards, `std::begin` and `std::end` remain valid and widely used utilities.

However, newer capabilities for working with ranges have been added to the language and Standard Library. For example, C++20 introduced the Ranges framework and facilities such as `std::ranges::begin` and `std::ranges::end`.

These newer utilities provide additional capabilities for generic programming and working with diverse ranges, but they are not mandatory replacements for every use of `std::begin` and `std::end`.

## Comparing `std::begin` with `std::ranges::begin`

### Design Differences

In C++11, `std::begin` and `std::end` provide a way to obtain iterators from containers and built-in arrays.

In C++20, `std::ranges::begin` and `std::ranges::end` were introduced as part of the Ranges framework. These functions follow the concepts and constraints associated with the Ranges model and form part of the Standard Library's newer approach to generic range processing.

The following example demonstrates their use:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values = {10, 20, 30};

    auto first = std::ranges::begin(values);
    auto last = std::ranges::end(values);

    while (first != last) {
        std::cout << *first << ' ';
        ++first;
    }

    std::cout << '\n';
}
```

This example requires C++20 or later and cannot be compiled as C++11 code.

The main distinction is that Ranges facilities are designed to work with the standard Range and Iterator concepts and offer capabilities beyond the simpler free-function model introduced in C++11.

### When to Use Each Approach

For projects that must remain compatible with C++11, `std::begin` and `std::end` are natural choices.

For C++20 or later projects that use the Ranges framework, `std::ranges::begin` and `std::ranges::end` may be more suitable, particularly when taking advantage of the newer Standard Library's type constraints and capabilities.

The final choice should reflect the project's target standard, the type of range being used, and the requirements of the API design.

## Common Mistakes When Using These Functions

### Assuming `end` Points to the Last Element

This is incorrect. `std::end` returns the position immediately after the last element, which is generally not dereferenceable.

### Using `std::end` with a Raw Pointer

When a raw pointer is passed to `std::begin` or `std::end`, the size of the original array cannot be inferred.

A raw pointer stores the address of an object or memory location but does not, by itself, describe the size of a range.

### Using `std::sort` with Every Container Type

Having iterators to the beginning and end does not guarantee that `std::sort` can be used. This algorithm requires random-access iterators.

Therefore, in addition to verifying that a range exists, check the iterator requirements of the selected algorithm.

### Forgetting That Data May Be `const`

The iterator type returned by `std::begin` depends on the type of the argument. For `const` containers, the function returns an iterator that provides constant access, preventing element modification through that iterator.

### Assuming Iterators Remain Valid

Modifying a container's structure may invalidate its iterators. Do not reuse previously stored iterators without checking the container's iterator-invalidation rules.

## Design Recommendations for Professional Code

### Prefer Standard Ranges

When implementing generic algorithms, consider using iterator ranges rather than relying on container-specific indexing, provided the chosen approach satisfies the algorithm's requirements.

This reduces the algorithm's dependence on a particular data type and improves reusability.

### Use `auto` for Iterators

Iterator types can differ across containers. Using `auto` often improves readability and avoids unnecessary dependencies on the exact iterator type.

### Use Standard Algorithms

Instead of manually implementing common operations such as searching, sorting, or accumulating values, prefer the corresponding Standard Library algorithms when they meet the requirements.

### Consider Operation Costs and Complexity

The `std::begin` and `std::end` functions themselves typically perform simple operations. However, the cost of traversal and of the algorithms that use them depends on the container type and iterator characteristics.

For example, traversing a `std::vector` differs from traversing a `std::list` in terms of access patterns and memory characteristics. Similarly, providing the beginning and end of a range does not make algorithms requiring random access applicable to every container.

## Summary

The `std::begin` and `std::end` functions are important C++11 Standard Library utilities that provide a unified way to access ranges across different containers and built-in arrays.

The key points are:

- `std::begin` returns an iterator to the beginning of a range.
- `std::end` returns an iterator to the position immediately after the last element.
- These functions provide a common iteration pattern for containers and built-in arrays.
- Half-open ranges simplify generic algorithm design and the handling of empty ranges.
- The resulting iterator type depends on the data type and its `const` qualification.
- These functions do not guarantee iterator validity, compatibility with every algorithm, or iterator stability after container modifications.
- In generic code that supports custom types, combining `using std::begin` and `using std::end` with unqualified calls often enables ADL.
- In C++20 and later, Ranges facilities provide more advanced options for designing range-based APIs.

Overall, using `std::begin` and `std::end` correctly helps produce more generic, readable code that aligns with the design principles of the Standard Library. A thorough understanding of their behavior is an important part of professional C++ development.

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>