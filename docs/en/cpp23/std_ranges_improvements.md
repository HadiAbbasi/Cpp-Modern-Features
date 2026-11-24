<div align="center">

[🇺🇸 English](./std_ranges_improvements.md) | [🇮🇷 فارسی](../../fa/cpp23/std_ranges_improvements.md)

</div>

---

# Improvements to the `std::ranges` Library in C++23

The `std::ranges` library was introduced in C++20 to simplify data processing through ranges, views, concepts, and range-based algorithms. The C++23 standard extends this library with new adaptors, aggregation and search algorithms, improved support for moving and read-only access, and a mechanism for extending the library with custom adaptors.

These changes are primarily part of the standard library. Therefore, using them requires both the compiler and its standard library implementation to support the relevant C++23 features.

## Introduction to the Core Concepts

### The Range Concept

A `Range` is a type for which `begin` and `end` can be obtained. A range does not necessarily have to be a container. For example, `std::views::iota` can generate a sequence of numbers without storing all its elements.

```cpp
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};
    static_assert(std::ranges::range<decltype(values)>);
}
```

### The View Concept

A `View` is a lightweight range type typically used to observe or transform another range. Many views do not own the underlying data; instead of creating a new container, they change how the data is traversed.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6};

    auto result = values
        | std::views::filter([](int x) { return x % 2 == 0; })
        | std::views::transform([](int x) { return x * x; });

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

The output is `4 16 36`.

### Lazy Evaluation

Many views are lazy: filtering and transformation operations are not performed when the pipeline is constructed; they are evaluated as the elements are traversed. This behavior can avoid creating intermediate data, but it does not guarantee faster execution. A predicate or transformer may be invoked multiple times, and its cost is incurred again on each traversal.

### Ownership and Lifetime

Views created from an lvalue container usually refer to that same container. Therefore, the container must remain alive for as long as the view is used. By contrast, `viewable_range` and mechanisms such as `views::all` help handle the differences between lvalues and rvalues. However, you should not assume that every view owns the data it refers to.

## New Range Adaptors in C++23

### Splitting Data with `views::chunk`

`std::views::chunk(n)` divides a range into consecutive, non-overlapping chunks of at most `n` elements. The final chunk may be smaller.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6, 7, 8};

    for (auto part : values | std::views::chunk(3)) {
        for (int value : part) {
            std::cout << value << ' ';
        }
        std::cout << '\n';
    }
}
```

The output consists of the chunks `1 2 3`, `4 5 6`, and `7 8`.

Use cases include batch processing, splitting data packets, and block-based processing. The chunk size must be positive. `chunk` exposes the elements as groups of views; by itself, it does not create a new data container.

### Overlapping Windows with `views::slide`

`std::views::slide(n)` creates consecutive windows of size `n`. Unlike `chunk`, the windows overlap and advance by one position at a time.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    for (auto window : values | std::views::slide(3)) {
        for (int value : window) {
            std::cout << value << ' ';
        }
        std::cout << '\n';
    }
}
```

The output windows are `1 2 3`, `2 3 4`, and `3 4 5`. If the number of elements is smaller than the window size, no complete window is produced.

This adaptor is useful for moving averages, time-series analysis, and neighborhood-based algorithms.

### Grouping with `views::chunk_by`

`std::views::chunk_by(predicate)` groups adjacent elements together for as long as the predicate confirms the relationship between them.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 7, 8, 10, 11, 12};

    auto groups = values | std::views::chunk_by(
        [](int left, int right) {
            return right - left == 1;
        }
    );

    for (auto group : groups) {
        for (int value : group) {
            std::cout << value << ' ';
        }
        std::cout << '\n';
    }
}
```

The groups are `1 2 3`, `7 8`, and `10 11 12`, in that order. The predicate checks the relationship between two adjacent elements, not whether each element independently belongs to a category. For predictable results, the predicate should behave consistently and should not depend on changes to external state.

### Strided Traversal with `views::stride`

`std::views::stride(n)` traverses the elements of a range using a positive step size of `n`.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50, 60, 70};

    for (int value : values | std::views::stride(2)) {
        std::cout << value << ' ';
    }
}
```

The output is `10 30 50 70`. This adaptor is useful for sampling and selecting elements at fixed intervals. The stride must be greater than zero.

### Traversing with Indices Using `views::enumerate`

`std::views::enumerate` provides each element together with its index. The result is tuple-like and can be unpacked using structured bindings.

```cpp
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> names{"Alice", "Bob", "Charlie"};

    for (auto [index, name] : names | std::views::enumerate) {
        std::cout << index << ": " << name << '\n';
    }
}
```

The index starts at zero. In many loops, this adaptor replaces the need for a manually maintained counter. In generic code, it is best not to constrain the index to a specific numeric type unless necessary.

### Combining Ranges with `views::zip`

`std::views::zip` traverses several ranges simultaneously and provides corresponding elements in a tuple-like form.

```cpp
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> names{"Alice", "Bob", "Charlie"};
    std::vector<int> ages{25, 30, 35};

    for (auto [name, age] : std::views::zip(names, ages)) {
        std::cout << name << ": " << age << '\n';
    }
}
```

Traversal continues until the shortest range ends. Therefore, if the input ranges have different sizes, the remaining elements of the longer ranges do not appear in the output.

### Transforming Multiple Ranges with `views::zip_transform`

`std::views::zip_transform(function, ranges...)` passes corresponding elements from multiple ranges to a function and exposes the function's results as an output range.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    auto sums = std::views::zip_transform(
        [](int x, int y) { return x + y; },
        a,
        b
    );

    for (int value : sums) {
        std::cout << value << ' ';
    }
}
```

The output is `11 22 33`. This adaptor is appropriate when the computed result matters and there is no need to retain tuples of the input elements.

### Accessing Adjacent Elements with `views::adjacent`

`std::views::adjacent<N>` produces overlapping windows of a fixed size `N`. Unlike `slide`, the size is specified at compile time.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    for (auto [left, right] : values | std::views::adjacent<2>) {
        std::cout << left << ", " << right << '\n';
    }
}
```

This adaptor is suitable for comparing adjacent elements and performing calculations based on several consecutive values.

### Transforming Adjacent Elements with `views::adjacent_transform`

`std::views::adjacent_transform<N>(function)` applies a function to each adjacent group of a fixed size.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    auto differences = values
        | std::views::adjacent_transform<2>(
            [](int left, int right) {
                return right - left;
            }
        );

    for (int value : differences) {
        std::cout << value << ' ';
    }
}
```

The output is `10 10 10 10`. Conceptually, this adaptor combines adjacent windows and transformation. It is useful for calculating differences, rates of change, and local properties.

### Cartesian Products with `views::cartesian_product`

`std::views::cartesian_product` generates all possible combinations of elements from multiple ranges.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{1, 2, 3};
    std::vector<char> letters{'A', 'B'};

    for (auto [number, letter] :
         std::views::cartesian_product(numbers, letters)) {
        std::cout << number << letter << '\n';
    }
}
```

The number of combinations is the product of the sizes of the input ranges:

\[
N = n_1 \times n_2 \times \cdots \times n_k
\]

This adaptor is useful for generating search spaces and combinatorial tests. However, the number of combinations can grow very quickly. Laziness does not prevent an explosion in the number of elements when the entire range is traversed.

### Joining Inner Ranges with `views::join_with`

`std::views::join_with(separator)` flattens the elements of inner ranges and places a separator between them.

```cpp
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> words{"Modern", "C++", "Ranges"};

    for (char ch : words | std::views::join_with(' ')) {
        std::cout << ch;
    }
}
```

The output is `Modern C++ Ranges`. The separator can be a suitable value or range. This adaptor is useful for constructing text and joining nested structures.

### Producing Repeated Values with `views::repeat`

`std::views::repeat(value)` creates a repeating range of a specified value. The version that takes a count creates a finite sequence, while the version without a count creates an unbounded sequence.

```cpp
#include <iostream>
#include <ranges>

int main() {
    auto values = std::views::repeat(42, 5);

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

For the unbounded version, the consumer should specify a limit such as `take`:

```cpp
auto values = std::views::repeat(42) | std::views::take(5);
```

This adaptor is suitable for generating test data, constant values, and repeating sequences.

## Controlling How Elements Are Accessed

### Viewing Elements as Rvalues with `views::as_rvalue`

`std::views::as_rvalue` exposes the elements of a range as rvalues. This can allow subsequent algorithms to move the elements.

```cpp
#include <ranges>
#include <string>
#include <utility>
#include <vector>

int main() {
    std::vector<std::string> source{"Alice", "Bob"};
    std::vector<std::string> destination;

    destination.reserve(source.size());

    for (auto&& value : source | std::views::as_rvalue) {
        destination.push_back(std::move(value));
    }
}
```

`as_rvalue` does not perform the move itself; it only changes the value category of the elements. The actual move depends on the consuming operation. After being moved from, the source object remains valid, but its value is unspecified.

### Viewing Elements as Const with `views::as_const`

`std::views::as_const` provides a read-only view of the elements.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    for (const int& value : values | std::views::as_const) {
        std::cout << value << ' ';
    }
}
```

This adaptor is useful for APIs that need to expose read-only access through a range. However, `as_const` does not make the underlying data immutable: other code holding a non-const reference to the container can still modify it.

## New `std::ranges` Algorithms in C++23

### Checking for a Value with `ranges::contains`

The `std::ranges::contains` algorithm checks whether a specified value exists in a range.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    bool found = std::ranges::contains(values, 30);
    std::cout << std::boolalpha << found;
}
```

In many cases, this algorithm is a more readable alternative to combining `find` with a comparison against `end`.

### Checking for a Consecutive Subrange with `ranges::contains_subrange`

The `std::ranges::contains_subrange` algorithm checks whether a consecutive pattern exists in a range.

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6};
    std::vector<int> pattern{3, 4, 5};

    bool found = std::ranges::contains_subrange(values, pattern);
}
```

A subrange here means a sequence of consecutive elements, not merely the presence of each pattern element at different positions.

### Searching for the Last Match with `find_last`

The C++23 standard adds three algorithms: `std::ranges::find_last`, `std::ranges::find_last_if`, and `std::ranges::find_last_if_not`.

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 2, 4, 2, 5};

    auto result = std::ranges::find_last(values, 2);

    if (result.found) {
        int lastValue = *result.begin();
    }
}
```

Unlike `find`, which locates the first match, these algorithms search for the last match. The result is a subrange that provides the position of the matching portion.

### Aggregating Elements with Fold Algorithms

C++23 adds the following fold algorithms to `std::ranges`:

- `fold_left`
- `fold_left_first`
- `fold_right`
- `fold_right_last`
- `fold_left_with_iter`
- `fold_left_first_with_iter`

The `fold_left` algorithm combines elements with an initial value from left to right.

```cpp
#include <iostream>
#include <numeric>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    int result = std::ranges::fold_left(
        values,
        0,
        [](int accumulator, int value) {
            return accumulator + value;
        }
    );

    std::cout << result;
}
```

The result is 15. Conceptually, the calculation is performed as `(((((0 + 1) + 2) + 3) + 4) + 5)`.

The `fold_right` algorithm processes elements in the opposite direction, from right to left. This distinction matters for non-commutative operations such as subtraction or string construction. `fold_left_first` and `fold_right_last` do not take a separate initial value; they use one of the range's elements as the starting value. For an empty range, the result is empty.

The two algorithms with the `with_iter` suffix also return the iterator indicating the processing position, in addition to the result.

### Filling a Range with `ranges::iota`

The `std::ranges::iota` algorithm fills the elements of an existing range with consecutive values.

```cpp
#include <numeric>
#include <vector>

int main() {
    std::vector<int> values(5);
    std::ranges::iota(values, 10);
}
```

The range's values become `10, 11, 12, 13, 14`. Do not confuse this algorithm with `std::views::iota`: the former fills an existing range, while the latter creates a generated range.

### Shifting Elements with `shift_left` and `shift_right`

The `std::ranges::shift_left` and `std::ranges::shift_right` algorithms move elements to the left or right within a range.

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    auto newEnd = std::ranges::shift_left(values, 2);
}
```

After the shift, the new valid portion contains `3, 4, 5`. The returned iterator identifies the end of the new valid portion. Elements outside this portion should not be assumed to be valid data with meaningful values. These algorithms are designed for writable ranges that meet the required iterator constraints.

## Extending the Range Adaptors

### Using `range_adaptor_closure`

C++23 adds the base class `std::ranges::range_adaptor_closure` for creating custom adaptors that are compatible with the pipe syntax.

```cpp
#include <iostream>
#include <ranges>
#include <utility>
#include <vector>

struct MultiplyByTwo
    : std::ranges::range_adaptor_closure<MultiplyByTwo> {

    template <std::ranges::viewable_range R>
    auto operator()(R&& range) const {
        return std::views::all(std::forward<R>(range))
            | std::views::transform([](int value) {
                return value * 2;
            });
    }
};

int main() {
    std::vector<int> values{1, 2, 3};

    for (int value : values | MultiplyByTwo{}) {
        std::cout << value << ' ';
    }
}
```

This mechanism is useful for libraries that want to compose their own operations in pipelines, just like standard adaptors. When designing a generic adaptor, consider `viewable_range`, forwarding references, element types, and range lifetime.

### Improved `views::join` Support for Temporary Ranges

With the P2328 changes, the restrictions on `join_view` for inner ranges that are prvalues are reduced. For example, the output of a transform that produces a temporary container can be flattened with `join`.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto flattened = values
        | std::views::transform([](int value) {
            return std::vector<int>{value, value * 10};
        })
        | std::views::join;

    for (int value : flattened) {
        std::cout << value << ' ';
    }
}
```

The output is `1 10 2 20 3 30`. This improvement makes it easier to combine transform and join when generating and flattening inner ranges.

### Improved Projections

In C++23, P2609 relaxes some restrictions on projections in ranges algorithms. A projection is a function through which an algorithm extracts the property it needs from each element.

```cpp
#include <algorithm>
#include <string>
#include <vector>

struct User {
    int id;
    std::string name;
};

int main() {
    std::vector<User> users{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"}
    };

    auto result = std::ranges::find(users, 2, &User::id);
}
```

In this example, the search is performed on the `id` field, without the need to define a custom comparator.

## Differences Between Similar Features

| Feature | Main behavior | Overlap |
|---|---|---|
| `chunk(n)` | Splits into chunks of at most n elements | No |
| `slide(n)` | Consecutive windows of size n | Yes |
| `chunk_by(pred)` | Groups according to the relationship between adjacent elements | Depends on the predicate |
| `stride(n)` | Selects elements at intervals of n | Different use case |
| `adjacent<N>` | Overlapping window with a compile-time fixed size | Yes |
| `zip` | Combines corresponding elements from multiple ranges | Depends on the inputs |
| `cartesian_product` | Generates all possible combinations | Full combination |

## Performance Considerations and Limitations

### Laziness Does Not Mean Zero Cost

Views usually avoid creating intermediate containers, but each traversal may execute predicates and transformers again. If an expensive result is consumed multiple times, materializing it in a container may be more appropriate.

### Manage the Lifetime of the Underlying Data

A non-owning view that refers to a container must not be used after that container has been destroyed. In addition, structural changes to a container may invalidate the view's iterators. The invalidation rules of the original container still matter.

### Consider the Difference Between References and Values

A transform may produce either a reference or a value. The choice of output type affects whether the data can be modified, copied, or moved. For large data, avoid unintended copies; for deliberate resource transfers, use `as_rvalue` together with an appropriate consuming operation.

### Predicates and Transformers Should Behave Appropriately

Functions used in views may be invoked at different times and more than once. Relying on an exact invocation order or modifying external state can make a program fragile. Prefer pure functions without side effects whenever possible.

### Evaluate the Cost of Combining Multiple Adaptors

A long pipeline is not always better than a simple loop. In performance-sensitive code, benchmark the implementation and examine iterator types, invocation counts, cache locality, and projection costs.

### Do Not Underestimate the Output Size

In particular, with `cartesian_product`, the number of output elements is the product of the input sizes. For unbounded or very large ranges, consider using a limiting consumer such as `take`.

## Detecting Standard Library Support

C++23 language support alone does not guarantee that every standard library feature has been implemented. Check the compiler version and the standard library implementation version separately.

For some features, feature-test macros are defined in `<version>` or related headers. These include:

| Macro | Feature |
|---|---|
| `__cpp_lib_ranges_as_const` | `as_const` |
| `__cpp_lib_ranges_as_rvalue` | `as_rvalue` |
| `__cpp_lib_ranges_cartesian_product` | `cartesian_product` |
| `__cpp_lib_ranges_chunk` | `chunk` and `slide` |
| `__cpp_lib_ranges_chunk_by` | `chunk_by` |
| `__cpp_lib_ranges_enumerate` | `enumerate` |
| `__cpp_lib_ranges_join_with` | `join_with` |
| `__cpp_lib_ranges_repeat` | `repeat` |
| `__cpp_lib_ranges_stride` | `stride` |
| `__cpp_lib_ranges_zip` | `zip`, `zip_transform`, `adjacent`, and `adjacent_transform` |
| `__cpp_lib_ranges_to_container` | `ranges::to` |

Check the macro values in the documentation for the implementation you use. The presence of a macro or feature in one library version does not mean that all compilers provide the same level of support.

## Choosing the Right Feature

- Use `chunk` to divide data into fixed-size batches.
- Use `slide` for overlapping windows whose size is determined at runtime.
- Use `chunk_by` to group elements according to the relationship between adjacent elements.
- Use `stride` to select elements at fixed intervals.
- Use `enumerate` to obtain indices together with values.
- Use `zip` to traverse multiple sequences simultaneously.
- Use `zip_transform` to calculate results directly from multiple ranges.
- Use `cartesian_product` to generate combinations across multiple ranges.
- Use `join_with` to join inner ranges with a separator.
- Use `as_rvalue` together with an appropriate move operation to move elements.
- Use `as_const` for read-only access.
- Use fold algorithms to aggregate data.

## Summary

The C++23 standard has evolved the `std::ranges` library from a framework suitable for filtering and transforming data into a much more complete collection of sequence-processing tools.

Features such as `chunk`, `slide`, and `chunk_by` simplify data splitting and grouping. `zip`, `zip_transform`, and `cartesian_product` make it easier to process multiple ranges. `enumerate` and `stride` improve traversal control, while `as_rvalue` and `as_const` provide more precise control over how elements are accessed.

In addition, fold algorithms, last-match search, value-existence checks, and element-shifting operations reduce the need for manual loops.

However, professional use of ranges requires attention to data lifetime, view ownership, value categories, repeated evaluation costs, and iterator complexity. The best approach is to use readable pipelines while examining actual behavior through benchmarking and careful analysis in performance-sensitive parts of the code.

## References

- [cppreference: C++23](https://en.cppreference.com/w/cpp/23)
- [cppreference: Ranges library](https://en.cppreference.com/w/cpp/ranges)
- [cppreference: Header `<ranges>`](https://en.cppreference.com/w/cpp/header/ranges)
- [WG21 P2442: Windowing range adaptors](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2442r1.html)
- [WG21 P2443: `views::chunk_by`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2443r1.html)
- [WG21 P2321: zip](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2321r2.html)
- [WG21 P2164: `views::enumerate`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2164r9.html)


---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>