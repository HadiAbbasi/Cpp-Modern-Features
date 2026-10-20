<div align="center">

[🇺🇸 English](./std_make_reverse_iterator.md) | [🇮🇷 فارسی](../../fa/cpp14/std_make_reverse_iterator.md)

</div>

---

# A Comprehensive Guide to `std::make_reverse_iterator` in C++14

## Introduction

In the C++ standard library, traversing a collection in reverse is useful in many situations. For example, you might need to process a `std::vector` from its last element to its first, find the last occurrence of a value, or perform an operation on the trailing elements of a `std::string`.

One of the standard tools for this purpose is `std::make_reverse_iterator`, which was introduced in C++14. This function takes an ordinary Iterator and creates a `std::reverse_iterator` that reverses the direction of traversal.

The function is defined in the standard `<iterator>` header and simplifies the creation of reverse iterators without requiring you to construct them manually.

The following example demonstrates its use:

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    auto reverseIt = std::make_reverse_iterator(values.end());

    std::cout << *reverseIt << '\n';
}
```

The output is:

```
50
```

This happens because `values.end()` refers to the position immediately after the last element. `std::make_reverse_iterator` creates a reverse iterator that treats the element immediately before that position as its current element.

To use this function correctly, it is essential to understand the difference between a reverse iterator's underlying position and the element accessed through dereferencing.

## Understanding Reverse Iterators and How They Work

### The Difference Between Forward and Reverse Traversal

In a container such as `std::vector`, ordinary traversal proceeds from the beginning of the collection toward its end.

For example, the `begin()` and `end()` iterators define a range that includes the first element but excludes the position represented by `end()`.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    for (auto it = values.begin(); it != values.end(); ++it) {
        std::cout << *it << ' ';
    }
}
```

The output is:

```
10 20 30
```

Reverse traversal proceeds in the opposite direction. The standard library provides `std::reverse_iterator` for this purpose.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto first = std::make_reverse_iterator(values.end());
    auto last = std::make_reverse_iterator(values.begin());

    for (auto it = first; it != last; ++it) {
        std::cout << *it << ' ';
    }
}
```

The output is:

```
30 20 10
```

In this example, `first` begins at the last element when traversed in reverse, while `last` marks the end of the reverse range.

### The Relationship Between a Reverse Iterator and Its Base Iterator

The most important detail when working with `std::reverse_iterator` is that its base iterator does not directly point to the element accessed through dereferencing.

For a valid, dereferenceable reverse iterator `r`, the following conceptual relationship holds:

```
*r == *std::prev(r.base())
```

The member function `base()` returns the underlying iterator. If `r` is a reverse iterator constructed from `values.end()`, then `r.base()` equals `values.end()`, not an iterator to the last element.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto r = std::make_reverse_iterator(values.end());

    std::cout << *r << '\n';
    std::cout << *std::prev(r.base()) << '\n';
}
```

The output is:

```
30
30
```

This difference reflects the fundamental design of reverse iterators: their underlying position represents a boundary between elements, and the accessible element is the one immediately before that boundary.

### Why the Base Iterator Represents a Boundary

Using the base iterator as a boundary allows reverse ranges to remain consistent with ordinary iterator ranges.

For example, a container containing five elements has the following ordinary range:

```
[values.begin(), values.end())
```

The corresponding reverse range is:

```
[std::make_reverse_iterator(values.end()),
 std::make_reverse_iterator(values.begin()))
```

This range traverses the same elements in the opposite direction without requiring a special iterator outside the container's normal range conventions.

## Introducing `std::make_reverse_iterator`

### Function Signature and Required Header

The function `std::make_reverse_iterator` is provided by the standard `<iterator>` header.

Its usage is straightforward:

```
#include <iterator>

auto reverseIt = std::make_reverse_iterator(it);
```

Here, `it` is an iterator, and the function's return type is determined by the type of that iterator.

Conceptually, the function's C++14 signature is:

```
template<class Iterator>
constexpr std::reverse_iterator<Iterator>
make_reverse_iterator(Iterator i);
```

This is a simplified representation of the standard declaration. The complete standard declaration includes details such as parameter qualifications and type requirements.

The function has the following characteristics:

- It has been available since C++14.
- It belongs to the `std` namespace.
- It deduces the underlying iterator type from its argument.
- It constructs the corresponding `std::reverse_iterator`.
- It requires the standard `<iterator>` header.

### How the Return Type Is Determined

The function's return type depends on the type of the input iterator.

For example, if the input iterator has type `std::vector<int>::iterator`, the return type is:

```
std::reverse_iterator<std::vector<int>::iterator>
```

If the input iterator has type `std::vector<int>::const_iterator`, the return type is:

```
std::reverse_iterator<std::vector<int>::const_iterator>
```

Using `auto` generally improves readability and avoids repeating long iterator type names.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto mutableReverse =
        std::make_reverse_iterator(values.end());

    auto constReverse =
        std::make_reverse_iterator(values.cend());

    *mutableReverse = 30;

    // *constReverse = 40; // Compilation error
}
```

In this example, `mutableReverse` allows elements to be modified, whereas `constReverse` provides read-only access and does not permit modifying the elements.

## Comparing `std::make_reverse_iterator` with Direct Construction

### Comparing the Two Construction Methods

Before C++14, programmers could construct `std::reverse_iterator` directly using its constructor.

This approach remains valid:

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    std::reverse_iterator<std::vector<int>::iterator> r(
        values.end()
    );
}
```

Since C++14, the same operation can be expressed more concisely:

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto r = std::make_reverse_iterator(values.end());
}
```

The primary difference is readability and type deduction, not traversal direction or iterator semantics.

| Feature                         | Direct construction                      | `std::make_reverse_iterator`                        |
| ------------------------------- | ---------------------------------------- | --------------------------------------------------- |
| Standard version                | Available before C++14                   | C++14                                               |
| Need to write the complete type | Yes, when specifying the type explicitly | No                                                  |
| Type deduction                  | Possible with `auto` or other techniques | The template parameter is deduced from the argument |
| Traversal direction             | Reverse                                  | Reverse                                             |
| Required header                 | `<iterator>`                             | `<iterator>`                                        |

For modern C++ code, `std::make_reverse_iterator` is a suitable choice, particularly when the underlying iterator type is long or complicated.

However, if the exact reverse iterator type is part of a function signature or public interface, explicitly specifying the type may improve clarity.

## Requirements for the Underlying Iterator

### Capabilities Required for Reverse Traversal

Not every ordinary iterator can be converted into a reverse iterator without considering its capabilities.

`std::reverse_iterator` requires an underlying iterator that supports the operations needed for reverse traversal. Specifically, a standard reverse iterator requires an iterator that meets the `BidirectionalIterator` requirements or provides stronger capabilities.

In practice, this means the underlying iterator must support moving backward.

Containers such as the following generally provide suitable iterators:

- `std::vector`
- `std::deque`
- `std::list`
- `std::array`
- `std::string`

By contrast, the iterators of `std::forward_list`, as well as input and output iterators that support only one-way traversal, are not suitable for ordinary use with `std::reverse_iterator`.

For example, `std::vector` provides random-access iterators, so its iterators support all the operations required by reverse iterators.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto r = std::make_reverse_iterator(values.end());

    ++r;
    --r;

    auto next = r + 1;
}
```

In this example, arithmetic operations are available because the underlying iterator supports random access.

The important point is that support for additional operations, such as `r + n`, depends on the capabilities of the underlying iterator. Using `std::make_reverse_iterator` does not imply that every reverse iterator supports random access.

## Traversing Ranges with the Function

### Traversing an Entire Container in Reverse

The most common use of the function is to construct two boundaries for reverse traversal of a container.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    auto first = std::make_reverse_iterator(values.end());
    auto last = std::make_reverse_iterator(values.begin());

    for (auto it = first; it != last; ++it) {
        std::cout << *it << ' ';
    }
}
```

The output is:

```
40 30 20 10
```

This approach is useful when an algorithm or function expects two iterators and you want to reverse the traversal direction.

### Reverse Traversal with a Range-Based `for` Loop

If you only need to traverse a standard container in reverse, using `rbegin()` and `rend()` is usually simpler.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    for (auto it = values.rbegin(); it != values.rend(); ++it) {
        std::cout << *it << ' ';
    }
}
```

The output is:

```
40 30 20 10
```

In this case, there is no need to construct reverse iterators manually because the container already provides the reverse traversal boundaries.

However, `std::make_reverse_iterator` becomes useful when you need to reverse an arbitrary range or provide iterators to a function in reverse order.

### Reversing Part of a Range

Suppose you need to process only the last three elements of a `std::vector`, in reverse order.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50, 60};

    auto first = std::make_reverse_iterator(values.end());
    auto last = std::make_reverse_iterator(values.end() - 3);

    for (auto it = first; it != last; ++it) {
        std::cout << *it << ' ';
    }
}
```

The output is:

```
60 50 40
```

In this example, the reverse range's ending boundary is constructed from the iterator corresponding to the position three elements before the end.

This technique is useful for processing a container's trailing elements without changing the actual order of its elements.

Note that `values.end() - 3` requires random-access iterator operations and is therefore suitable for containers such as `std::vector`. For containers that provide only bidirectional iterators, use operations supported by that iterator type and verify that the position is valid before moving backward.

## Practical and Professional Applications

### Finding the Last Occurrence of a Value

Sometimes you need to find the last occurrence of a value in a range. Reverse traversal makes it possible to search from the end.

```
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{5, 8, 3, 8, 2, 8, 1};

    auto reverseIt = std::find(
        values.rbegin(),
        values.rend(),
        8
    );

    if (reverseIt != values.rend()) {
        auto forwardIt = reverseIt.base();
        --forwardIt;

        std::cout << *forwardIt << '\n';
        std::cout << std::distance(values.begin(), forwardIt) << '\n';
    }
}
```

The output is:

```
8
5
```

In this example, `std::find` traverses the collection backward and finds the first `8` encountered in reverse order. This is the last occurrence of `8` in the ordinary order.

The important detail is converting the result from a reverse iterator to an ordinary iterator. The value returned by `reverseIt.base()` refers to the position immediately after the found element in the ordinary order. Therefore, moving backward once is necessary to reach the element itself.

This conversion is valid only when the reverse iterator refers to an actual element. If the search result equals `values.rend()`, you must not dereference its base iterator or decrement it to obtain a supposed match.

### Processing Elements Until a Condition Is Met

Some algorithms need to traverse a collection backward until a particular condition is encountered.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 0, 5, 6};

    auto it = std::make_reverse_iterator(values.end());
    auto stop = std::make_reverse_iterator(values.begin());

    while (it != stop && *it != 0) {
        std::cout << *it << ' ';
        ++it;
    }
}
```

The output is:

```
6 5
```

In this example, reverse traversal stops before reaching the value zero.

The order of the conditions matters: first, the code checks that the iterator has not reached the end boundary, and only then does it dereference the iterator.

### Using Standard Algorithms

Many standard algorithms operate on iterators rather than depending on a particular container. Consequently, reverse iterators can change their traversal direction without requiring a different algorithm.

```
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    auto count = std::count_if(
        std::make_reverse_iterator(values.end()),
        std::make_reverse_iterator(values.begin()),
        [](int value) {
            return value % 2 == 1;
        }
    );

    std::cout << count << '\n';
}
```

The output is:

```
3
```

In this example, `std::count_if` respects the reverse traversal order. However, because the operation counts matching elements, the result is independent of the traversal direction.

This demonstrates that reverse iterators are useful not only for displaying elements in reverse order, but also for reusing generic algorithms with a different traversal direction.

## The Relationship with `rbegin()` and `rend()`

### Practical Differences Between the Two Approaches

Many standard containers provide `rbegin()` and `rend()`, which directly return suitable reverse iterators.

For example:

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto a = values.rbegin();
    auto b = std::make_reverse_iterator(values.end());
}
```

For this container, `a` and `b` are equivalent in both type and position.

However, the two approaches are not always interchangeable:

- Use `rbegin()` and `rend()` for straightforward reverse traversal of an entire container.
- Use `std::make_reverse_iterator` when you have the boundaries of an arbitrary range.
- In generic code, you might have access only to the beginning and ending iterators of a range, without a `rbegin()` function.
- To reverse only part of a range, you can convert its boundaries into reverse iterators.

Therefore, the choice should depend on the data structure and the algorithm's requirements.

## Converting Between Reverse and Ordinary Iterators

### Using `base()` Correctly

The member function `base()` returns the underlying iterator. It is used to convert a reverse iterator's position into the corresponding position in ordinary traversal.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    auto r = std::make_reverse_iterator(values.end());
    auto base = r.base();

    std::cout << *r << '\n';
    std::cout << (base == values.end()) << '\n';
}
```

The output is:

```
40
1
```

In this example, the reverse iterator provides access to the element `40`, but its base iterator equals `values.end()`.

If you need the ordinary iterator corresponding to the element `40`, you can move backward once:

```
auto forwardIt = r.base();
--forwardIt;
```

This operation must be performed only when the base iterator is positioned after `begin()`. Decrementing `begin()` or otherwise using an invalid position results in undefined behavior.

### Distinguishing Iterator Positions from Elements

When converting between iterators, it is important to distinguish between:

- The position represented by the base iterator.
- The element accessible through the reverse iterator.

This distinction matters when removing elements, inserting elements, calculating distances, and passing algorithm results to other parts of a program.

For example, if a reverse iterator finds the last element of a `std::vector`, using `base()` directly as an iterator to that element is incorrect. To reach the element itself, you must move backward once.

## Limitations and Safety Considerations

### Iterator Validity and Container Lifetime

Constructing a reverse iterator does not, by itself, guarantee that its underlying iterator is valid.

If the underlying iterator is invalid, the resulting reverse iterator cannot be used validly either.

For example, operations such as increasing the capacity of a `std::vector` can relocate its storage and invalidate existing iterators.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto r = std::make_reverse_iterator(values.end());

    values.push_back(4);

    // r may be invalid after reallocation.
}
```

In this example, adding an element may cause reallocation. If reallocation occurs, using `r` is invalid.

Even when reallocation does not occur, you must follow the iterator invalidation rules for the relevant container and operation, because those rules differ among container types.

### The Danger of Dereferencing an End Iterator

The ending boundary of a reverse range, such as `values.rend()`, does not provide access to an element.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto first = std::make_reverse_iterator(values.end());
    auto last = std::make_reverse_iterator(values.begin());

    if (first != last) {
        int value = *first;
        (void)value;
    }
}
```

Checking the boundary before dereferencing makes the code safe for empty ranges as well.

By contrast, dereferencing the reverse iterator equal to `last` ordinarily results in undefined behavior.

### Lack of Support for Singly Linked Containers

One-way iterators, such as those provided by `std::forward_list`, do not support moving backward.

Because reverse iterators must be able to move in the opposite direction, using such iterators with `std::make_reverse_iterator` is not a suitable standard approach.

Depending on the requirements, alternative approaches include:

- Copying the elements into a more suitable container.
- Storing the elements in a data structure that supports bidirectional traversal.
- Using an algorithm that produces the desired result without reverse traversal.

The appropriate choice depends on memory overhead, time complexity, and the program's actual requirements.

## C++14 and Other Standard Versions

### Differences Between C++11 and C++14

The `std::reverse_iterator` type existed before C++14, but the helper function `std::make_reverse_iterator` was introduced in C++14.

Consequently, C++11 code can construct a reverse iterator directly using the `std::reverse_iterator` constructor.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    std::reverse_iterator<std::vector<int>::iterator> r(
        values.end()
    );
}
```

In C++14, the helper function avoids writing the complete type:

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto r = std::make_reverse_iterator(values.end());
}
```

This change improves the standard library interface without changing the fundamental semantics of reverse iterators.

### Differences from Ranges in C++20

C++20 introduced broader facilities for working with ranges, including `std::ranges` and standard views.

For example, `std::views::reverse` can expose a range in reverse order without requiring you to construct reverse iterators manually.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    for (int value : values | std::views::reverse) {
        std::cout << value << ' ';
    }
}
```

The output is:

```
30 20 10
```

This capability belongs to C++20 and should not be confused with `std::make_reverse_iterator`, which has been available since C++14.

For projects based on C++14 or C++17, `std::make_reverse_iterator` remains a standard and appropriate way to reverse the traversal of iterator-based ranges.

## Common Mistakes When Using Reverse Iterators

### Confusing `base()` with the Iterator to the Current Element

One of the most common mistakes is assuming that `base()` directly points to the element accessible through the reverse iterator.

In reality, `base()` returns the corresponding boundary position. To reach the current element, you generally need to move backward once, provided that doing so is valid.

### Using Inconsistent Range Boundaries

If the beginning and ending reverse iterators are constructed from the wrong boundaries, the code may traverse a different range than intended.

To reverse an ordinary range `[first, last)`, construct the corresponding reverse boundaries as follows:

```
auto reverseFirst = std::make_reverse_iterator(last);
auto reverseLast = std::make_reverse_iterator(first);
```

The boundaries must be reversed because traversal begins at the end of the ordinary range and proceeds toward its beginning.

### Assuming Random-Access Capabilities

The existence of a reverse iterator does not imply support for arbitrary arithmetic operations.

Operations such as `r + n` or `r - n` are available only when the underlying iterator supports the required capabilities.

Therefore, generic algorithms should not use random-access operations without checking the requirements of the iterator type.

### Modifying a Container During Traversal

If an operation invalidates the underlying iterator during traversal, the reverse iterator becomes invalid as well.

To avoid this problem, follow the container's iterator invalidation rules and, when necessary, obtain new iterators after modifying the container's structure.

## Choosing the Appropriate Approach

| Programming requirement                            | Recommended approach                              |
| -------------------------------------------------- | ------------------------------------------------- |
| Traverse an entire container from end to beginning | `rbegin()` and `rend()`                           |
| Reverse an arbitrary iterator-based range          | `std::make_reverse_iterator`                      |
| Construct a reverse iterator in C++11              | Direct construction of `std::reverse_iterator`    |
| Find the last occurrence of a value                | Use a search algorithm with reverse iterators     |
| Work with standard ranges in C++20                 | Consider `std::views::reverse`                    |
| Reverse traversal with a one-way iterator          | Choose an alternative algorithm or data structure |

This table provides practical guidance. The final choice should account for the project's C++ standard version, iterator capabilities, memory overhead, and algorithmic requirements.

## Summary

The `std::make_reverse_iterator` function is a useful C++14 standard library utility that simplifies reverse iterator construction and allows standard algorithms to operate in reverse traversal order.

The most important points to remember are:

- The function is defined in the standard `<iterator>` header.
- The underlying iterator must support bidirectional traversal or provide stronger capabilities.
- For reverse traversal of an entire container, `rbegin()` and `rend()` are usually simpler.
- For reversing an arbitrary iterator-based range, `std::make_reverse_iterator` is useful.
- The member function `base()` returns the underlying iterator position, not an iterator directly pointing to the accessible element.
- The validity of a reverse iterator depends on the validity of its underlying iterator and the lifetime of the container.
- C++20 provides additional options for reverse range traversal, including `std::views::reverse`.

Ultimately, understanding the relationship between a reverse iterator and its underlying iterator is the key to using `std::make_reverse_iterator` correctly and safely. This understanding allows developers to reuse existing algorithms for reverse traversal without changing the container's actual element order, resulting in more generic, readable, and maintainable code.

---

## 🤝 Contributions

<div align="center">

| GitHub                                      | LinkedIn                                                           | Email                                                  | Site                           | Telegram                               |
| ------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------ | -------------------------------------- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](mailto:hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>