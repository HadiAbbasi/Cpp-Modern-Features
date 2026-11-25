<div align="center">

[🇺🇸 English](./std_views_pairwise.md) | [🇮🇷 فارسی](../../fa/cpp23/std_views_pairwise.md)

</div>

---

# Introduction to `std::views::pairwise` in C++23

## Introduction

Many algorithms need to examine each element in a range together with the element that follows it. Examples include calculating the difference between consecutive values, detecting changes in a sequence, examining consecutive edges in a path, or creating pairs of adjacent data elements. A traditional implementation of this pattern usually requires iterators, an end condition, and manual iterator advancement.

The `std::views::pairwise` facility in the C++23 standard library expresses this pattern as a `view`. Starting at each position, it produces a pair containing the current element and the next element, making them available to the consumer without copying the elements.

## Feature Overview and Standardization

`std::views::pairwise` is defined in the `std::views` namespace in the `<ranges>` header and has been available since C++23. It is effectively an alias for `std::views::adjacent<2>`, so its behavior and constraints come from `adjacent_view`.

```cpp
#include <ranges>
#include <vector>

int main() {
    std::vector values{10, 20, 30, 40};

    for (auto [current, next] : values | std::views::pairwise) {
        // Use current and next
    }
}
```

The example produces the pairs `(10, 20)`, `(20, 30)`, and `(30, 40)`. For a range containing `n` elements, the number of pairs is `n - 1` when `n >= 2`. Otherwise, the output is empty.

## The Concept of an Adjacent Window

`pairwise` is an example of a sliding window with a fixed size of two. If the elements of the input range are represented by `x[0]` through `x[n-1]`, the output consists of the following pairs:

```text
(x[0], x[1])
(x[1], x[2])
(x[2], x[3])
...
(x[n-2], x[n-1])
```

Each pair shares one element with the preceding pair. Consequently, this view produces consecutive, overlapping windows rather than independent, non-overlapping pairs.

Each output element is a two-element `std::tuple` whose components refer to elements in the original range. Therefore, constructing the pairs does not normally copy the element values. The exact reference type and `const` behavior depend on the type of the input range.

## How to Use the Feature

### Direct Usage

The simplest way to use the feature is to apply the adaptor to a range:

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector values{3, 8, 12, 19};

    for (auto pair : values | std::views::pairwise) {
        std::cout << std::get<0>(pair)
                  << " -> "
                  << std::get<1>(pair)
                  << '\n';
    }
}
```

In this example, the pipe operator passes the range on the left to the adaptor on the right. You can also use the direct-call form, `std::views::pairwise(values)`.

### Using Structured Bindings

Because each output element is tuple-like, structured bindings can be used to give names to its two components:

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector values{2, 5, 9, 14};

    for (auto [left, right] : values | std::views::pairwise) {
        std::cout << left << ", " << right << '\n';
    }
}
```

In this example, the variables `left` and `right` are initialized from the tuple's components. Although the tuple components refer to the original elements, the view itself does not copy those elements. However, the types of the structured-binding variables and the use of `auto` or `auto&` in the loop affect whether local variables are copies or references. If you intend to modify the original elements, choose reference semantics deliberately.

### Using the Direct-Call Form

The direct-call form is useful when combining the adaptor with intermediate variables or APIs where pipe syntax is not convenient:

```cpp
auto adjacent_pairs = std::views::pairwise(values);

for (auto [a, b] : adjacent_pairs) {
    // Process the adjacent elements
}
```

This expression creates a view; it does not, by itself, iterate over the elements or store them in a new container.

## Input Range Requirements

One important constraint of `std::views::pairwise` is that the input range must model `std::ranges::forward_range`. A range that models only `input_range`, such as some streams or ranges that can be traversed only once, is therefore insufficient.

This requirement follows from the nature of the algorithm: producing overlapping windows requires the implementation to retain the necessary iterators and access adjacent elements reliably. A `forward_range` provides the minimum capabilities required.

The input must also be a `viewable_range` so that the adaptor can convert it into a suitable view. This does not mean that the user must always construct a view in advance; containers such as `std::vector` can also be used directly.

## Reference Semantics and Modifying Elements

The tuple components in the output refer to elements in the original range. Therefore, if the range is mutable, the pairs can be used to modify the original elements:

```cpp
#include <ranges>
#include <vector>

int main() {
    std::vector values{1, 2, 3, 4};

    for (auto pair : values | std::views::pairwise) {
        std::get<0>(pair) *= 10;
    }
}
```

In this example, the first element of each pair is modified, which also modifies the corresponding element in `values`. Note that middle elements can appear in two consecutive pairs, so a modification to an element may also be observed during the next iteration.

For read-only access, ordinary local variables are sufficient. When modifying elements, make your intent clear through reference types and structured bindings, and avoid assuming that each pair is an independent copy of the data.

## Combining with Other Range Adaptors

The main strength of `pairwise` becomes apparent when it is combined with other adaptors. For example, you can filter the data first and then create adjacent pairs:

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector values{1, 2, 3, 4, 5, 6, 7, 8};

    auto even_values = values
        | std::views::filter([](int value) {
              return value % 2 == 0;
          });

    for (auto [a, b] : even_values | std::views::pairwise) {
        std::cout << a << " - " << b << '\n';
    }
}
```

In this example, adjacency is defined relative to the sequence produced by `filter`, not relative to consecutive indices in the original container. The resulting pairs are therefore `(2, 4)`, `(4, 6)`, and `(6, 8)`.

You can also use `transform` to convert the result of each pair:

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector values{4, 9, 15, 22};

    auto differences = values
        | std::views::pairwise
        | std::views::transform([](auto pair) {
              return std::get<1>(pair) - std::get<0>(pair);
          });

    for (int difference : differences) {
        std::cout << difference << '\n';
    }
}
```

The output of this example consists of the differences between consecutive elements: `5`, `6`, and `7`. The pipeline remains lazy; each difference is calculated during iteration.

## Practical Applications

### Calculating Differences Between Consecutive Elements

A common application is calculating the change between two consecutive samples. This pattern is useful in time-series analysis, measurements, and numerical data processing:

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector temperatures{18, 20, 19, 23, 25};

    auto changes = temperatures
        | std::views::pairwise
        | std::views::transform([](auto values) {
              return std::get<1>(values) - std::get<0>(values);
          });

    for (int change : changes) {
        std::cout << change << ' ';
    }
}
```

### Checking Whether a Sequence Is Strictly Increasing

You can determine whether a sequence is strictly increasing by examining all adjacent pairs:

```cpp
#include <ranges>
#include <vector>

bool is_strictly_increasing(const std::vector<int>& values) {
    for (auto [a, b] : values | std::views::pairwise) {
        if (a >= b) {
            return false;
        }
    }

    return true;
}
```

For a range containing zero or one element, the loop does not execute and the function returns `true`. This is consistent with the usual definition of strict increase for such a sequence.

### Traversing Consecutive Edges in a Path

If the points in a path are stored in order, each adjacent pair represents an edge. This approach is useful for calculating the lengths of path segments or checking connections between points:

```cpp
#include <cmath>
#include <iostream>
#include <ranges>
#include <utility>
#include <vector>

int main() {
    using Point = std::pair<double, double>;

    std::vector<Point> path{
        {0.0, 0.0},
        {3.0, 4.0},
        {6.0, 4.0}
    };

    for (auto [from, to] : path | std::views::pairwise) {
        const double dx = to.first - from.first;
        const double dy = to.second - from.second;
        const double length = std::hypot(dx, dy);

        std::cout << length << '\n';
    }
}
```

## Comparison with Related Approaches

### Comparison with an Iterator-Based Loop

A manual implementation typically maintains two iterators: one for the current element and another for the next element. Both iterators are then advanced until the second reaches the end. This approach is still valid and can be appropriate when more precise control over traversal is required.

The advantage of `pairwise` is that it expresses the programmer's intent directly and reduces errors involving end conditions and iterator advancement. On the other hand, a manual loop may be more suitable for single-pass inputs or algorithms with unusual traversal logic.

### Comparison with `std::views::adjacent`

The `std::views::adjacent<N>` adaptor generalizes the same idea and produces windows with a fixed size of `N`. The name `std::views::pairwise` is exactly equivalent to `std::views::adjacent<2>`.

```cpp
auto pairs = values | std::views::adjacent<2>;
auto triples = values | std::views::adjacent<3>;
```

In both cases, the output consists of tuples of references to adjacent elements. The window size is determined at compile time through a template parameter.

### Comparison with `std::views::slide`

The `std::views::slide` adaptor also creates sliding windows, but each output element is a range rather than a tuple of references. The window size is supplied as an argument to `slide`.

Therefore, if you need a general-purpose window of elements that can be traversed internally, `slide` is usually more natural. If you specifically need a fixed number of adjacent elements in a tuple-like structure, `adjacent` or `pairwise` is a better fit.

### Comparison with `std::views::zip`

The `std::views::zip` adaptor combines elements at corresponding positions from multiple independent ranges. In contrast, `pairwise` combines two adjacent elements from a single range. The two tools therefore address different problems.

## Traversal Properties and Performance

The output of `adjacent_view` is always a `forward_range`. If the underlying view has stronger capabilities, the resulting view can preserve them, including `bidirectional_range`, `random_access_range`, and `sized_range`.

For a sized range containing `n` elements, the output size is `n - 1` when `n >= 2`, and zero when the input contains fewer than two elements. This view does not store the pairs in a separate container. As a result, its construction cost is usually negligible, and the main cost comes from iteration and the operations the consumer performs on each pair.

However, this does not mean that every adaptor combination necessarily has constant-time operations or random access. The final capabilities depend on the underlying range and the adaptors in the pipeline. Also, if you perform an expensive operation in `transform`, that operation is executed when the elements are dereferenced and consumed.

## Data Lifetime and Safety Considerations

Because the output elements refer to the underlying range, the original data must outlive any use of the view and its iterators. Keeping the view alive does not, by itself, guarantee that memory owned by an external container remains valid.

Operations that invalidate iterators of the underlying container can also invalidate subsequent use of the view. For example, modifying a `std::vector` in a way that causes reallocation may invalidate the references and iterators used by the view.

The `borrowed_range` property of `adjacent_view` follows that of its underlying view. This property does not imply ownership of the data; it only concerns the validity of iterators that remain after the view object is destroyed, subject to the properties of the underlying range. It should therefore not be treated as a substitute for proper container lifetime management.

## Limitations and Common Mistakes

- The input must be a `forward_range`; therefore, not every stream or single-pass `input_range` can be passed directly to `pairwise`.
- An empty or single-element range produces no pairs. Code that assumes at least one pair exists without checking the range size may have incorrect logic.
- The pairs overlap; a middle element appears in two consecutive pairs. Modifying it in one iteration may affect what is observed in the next.
- An output pair is not an independent copy of two values; its components refer to the original elements.
- Adjacency is defined after applying preceding adaptors. For example, `filter | pairwise` produces adjacent pairs from the filtered output.
- The view is lazy; chained transformations are not necessarily executed until iteration takes place.
- Do not use the view after the underlying range has been destroyed or its iterators have been invalidated.

## Checking Library Support and Feature Availability

To detect standard library support, you can use the feature-test macro associated with the zip and adjacent facilities. A value of `202110L` for `__cpp_lib_ranges_zip` indicates support for `adjacent_view` and related facilities in C++23.

```cpp
#include <ranges>

#ifndef __cpp_lib_ranges_zip
#error "The standard library does not provide C++23 ranges zip/adjacent support"
#elif __cpp_lib_ranges_zip < 202110L
#error "Insufficient ranges zip/adjacent support"
#endif

int main() {
    // Use std::views::pairwise
}
```

Simply selecting C++23 compilation mode does not guarantee that the installed standard library version implements all the facilities in this area. Compiler support and standard library support are related but separate matters; different versions of development toolchains may differ in the completeness of their implementations.

## Design Considerations for Professional Code

When using `pairwise`, first determine which sequence should define adjacency. The order of adaptors has semantic significance: filtering before `pairwise` is not equivalent to filtering after pairs have been created.

If you only need to perform a calculation on each pair, combining `pairwise` with `transform` usually produces clear, expressive code. If the goal is simply to determine whether a pair with a particular property exists, standard algorithms or a loop that returns early may be clearer than a long adaptor pipeline.

When returning a view from a function in an API, carefully consider the lifetime of the underlying range. Returning a view that refers to a local container is an example of a dangling-reference error. In such cases, ownership of the data, the type of view, and the lifetime of the result must be designed correctly.

## Summary

The `std::views::pairwise` facility in C++23 provides a standard, lazy, and readable way to traverse overlapping pairs of adjacent elements in a range. It is particularly useful for calculating consecutive differences, checking ordering, analyzing changes, and processing path edges.

The most important considerations when using it are the `forward_range` requirement, the reference semantics of the output tuples, the overlap between pairs, and the dependence of the view's lifetime on the underlying data. For windows containing more than two elements, consider `std::views::adjacent<N>`. For windows whose elements are themselves range-like, `std::views::slide` is the more relevant option.

## References and Further Reading

- [cppreference: `std::ranges::views::adjacent` and `std::views::pairwise`](https://en.cppreference.com/w/cpp/ranges/adjacent_view)
- [cppreference: Ranges library](https://en.cppreference.com/w/cpp/ranges)
- [cppreference: Feature-test macros](https://en.cppreference.com/w/cpp/feature_test)
- [C++ standard, `range.adjacent` section](https://www.iso.org/standard/83626.html)


---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>