<div align="center">

[🇺🇸 English](./std_views_adjacent.md) | [🇮🇷 فارسی](../../fa/cpp23/std_views_adjacent.md)

</div>

---

# **Introduction to&#x20;**`std::views::adjacent`**&#x20;in C++23**

## **Introduction**

One of the useful features introduced in the C++23 `Ranges` library is `std::views::adjacent`, which makes it possible to process adjacent elements of a Range in a simple, readable way without creating a new container.

Many algorithms need to examine multiple consecutive elements simultaneously. For example:

- Comparing each element with the next one.
- Calculating differences between consecutive values.
- Detecting changes in a sequence of data.
- Identifying consecutive patterns in a collection.
- Processing sliding windows.
- Examining three consecutive elements to detect specific patterns.
- Approximating derivatives from sampled data.
- Analyzing changes in prices, temperatures, speeds, or other time-series data.

Before C++23, these operations were typically implemented using iterators, manual loops, custom algorithms, or third-party libraries.

`std::views::adjacent` solves this problem by providing a Range Adaptor. It creates a View that exposes a specified number of consecutive elements from the underlying Range at each position.

An important detail is that the result is not a new collection of copied elements. Instead, it is a View over the elements of the original Range.

## **Understanding Adjacent Elements**

### **What Are Adjacent Elements?**

Suppose we have a `std::vector` containing the following values:

```
std::vector<int> values{10, 20, 30, 40, 50};
```

If we want to examine every pair of adjacent elements, the resulting windows are:

```
(10, 20)
(20, 30)
(30, 40)
(40, 50)
```

Each window contains two consecutive elements. When moving to the next position, the window advances by one element.

If we consider three consecutive elements instead, the resulting windows are:

```
(10, 20, 30)
(20, 30, 40)
(30, 40, 50)
```

Therefore, the central idea behind adjacent elements is the creation of overlapping windows over a Range.

### **Overlapping Windows vs. Regular Grouping**

It is important to understand that the windows produced by `adjacent` differ from ordinary grouping.

For example, grouping elements into groups of three produces something conceptually similar to:

```
(10, 20, 30)
(40, 50, ...)
```

However, adjacent three-element windows produce:

```
(10, 20, 30)
(20, 30, 40)
(30, 40, 50)
```

In the second case, consecutive windows share elements.

This behavior is particularly useful for algorithms that need to examine relationships between neighboring elements.

## **Introduction to std::views::adjacent**

### **Basic Syntax**

`std::views::adjacent` is defined in the following header:

```
#include <ranges>
```

Its general usage is:

```
std::views::adjacent<N>(range);
```

Here:

- `N` specifies the number of elements in each window.
- `range` is the input collection or View.
- The result is a View containing consecutive windows from the original Range.

For example:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    for (auto window : values | std::views::adjacent<2>) {
        std::cout
            << std::get<0>(window) << ", "
            << std::get<1>(window) << '\n';
    }
}
```

Output:

```
10, 20
20, 30
30, 40
40, 50
```

In this example, `adjacent<2>` creates windows containing two consecutive elements.

### **Behavior for Different Window Sizes**

Suppose the input Range contains five elements.

| Expression    | Elements per window | Number of windows |
| ------------- | ------------------- | ----------------- |
| `adjacent<1>` | 1                   | 5                 |
| `adjacent<2>` | 2                   | 4                 |
| `adjacent<3>` | 3                   | 3                 |
| `adjacent<4>` | 4                   | 2                 |
| `adjacent<5>` | 5                   | 1                 |
| `adjacent<6>` | 6                   | 0                 |

In general, if the input Range contains `S` elements and each window contains `N` elements, the output size is:

\\[ \text{size} = \begin{cases} S-N+1 & S \geq N\\\ 0 & S < N \end{cases} \\]

This is one of the most important properties of `std::views::adjacent`.

## **Accessing Elements in Each Window**

### **Using std::get**

Each element produced by `adjacent` is a `std::tuple` containing the elements of the current window.

Therefore, individual elements can be accessed using `std::get`.

```
#include <iostream>
#include <ranges>
#include <tuple>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    for (auto window : values | std::views::adjacent<3>) {
        int first = std::get<0>(window);
        int second = std::get<1>(window);
        int third = std::get<2>(window);

        std::cout
            << first << ' '
            << second << ' '
            << third << '\n';
    }
}
```

Output:

```
1 2 3
2 3 4
3 4 5
```

An important detail is that the indices passed to `std::get` must be known at compile time.

Consequently, unlike ordinary containers, tuple elements cannot be accessed using a runtime index.

### **Using Structured Bindings**

Structured Bindings were introduced in C++17 and provide a convenient way to access tuple elements.

Instead of repeatedly calling `std::get`, we can use the following syntax:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    for (auto [first, second] : values | std::views::adjacent<2>) {
        std::cout << first << " -> " << second << '\n';
    }
}
```

Output:

```
10 -> 20
20 -> 30
30 -> 40
40 -> 50
```

This approach provides excellent readability when working with windows containing a fixed number of elements.

### **Choosing the Appropriate Variable Type in Structured Bindings**

An important consideration when using `adjacent` is choosing the appropriate type for the loop variable.

The following three forms are generally available:

```
for (auto [a, b] : range | std::views::adjacent<2>) {
    // ...
}
```

```
for (auto& [a, b] : range | std::views::adjacent<2>) {
    // ...
}
```

```
for (auto&& [a, b] : range | std::views::adjacent<2>) {
    // ...
}
```

The differences concern how the tuple object returned by dereferencing the iterator is stored and accessed.

For general-purpose code, `auto&&` is a suitable choice because it accommodates both reference and proxy-reference behavior.

A more important distinction is that the reference semantics of the tuple's elements are independent of whether the tuple object itself is a reference.

Even when the tuple object is copied inside the loop, its elements can still refer to elements in the original Range.

## **Understanding References in adjacent's Output**

### **Are Elements Copied?**

One of the most important properties of `std::views::adjacent` is that window elements are exposed as references or reference-like values from the underlying Range.

In other words, this feature does not need to copy the original values to create each window.

Consider the following example:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    for (auto&& [first, second] : values | std::views::adjacent<2>) {
        first *= 2;
        second += 5;
    }

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

Output:

```
20 45 70 45
```

The modifications are applied directly to the elements of the original `vector`.

This happens because `adjacent` does not independently copy the underlying elements.

### **Differences Between auto and auto& in the Output**

Consider the following example:

```
for (auto [first, second] : values | std::views::adjacent<2>) {
    first = 100;
}
```

Even though `auto` is used, the modifications can affect the original Range.

This is because copying the tuple does not necessarily copy the elements referenced by that tuple.

However, this behavior depends on the reference type provided by the underlying Range.

For example, if the Range uses a transformation View that produces values, its output may not be a direct reference to the original container.

### **Output Behavior for Different Range Types**

The types of the elements in the resulting tuple depend on the reference type provided by the underlying Range.

For example:

- With an ordinary `std::vector<int>`, the output elements are modifiable.
- With a `std::span<const int>`, the elements are read-only.
- With a View that produces values, the output is not necessarily a direct reference to the original container.
- With a View that uses proxy references, behavior depends on the corresponding proxy type.

Therefore, it is incorrect to assume that every element returned by `adjacent` is necessarily an ordinary `T&`.

## **Introduction to std::views::pairwise**

One useful alias related to `adjacent` was introduced in C++23: `std::views::pairwise`.

It is equivalent to:

```
std::views::adjacent<2>
```

In other words, `pairwise` always creates windows containing two adjacent elements.

For example:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 3, 6, 10, 15};

    for (auto [previous, current] : values | std::views::pairwise) {
        std::cout << previous << " -> " << current << '\n';
    }
}
```

Output:

```
1 -> 3
3 -> 6
6 -> 10
10 -> 15
```

### **When Should You Use pairwise?**

`pairwise` is appropriate when an algorithm specifically needs two adjacent elements.

Typical use cases include:

- Calculating differences between consecutive values.
- Checking whether consecutive elements are increasing or decreasing.
- Comparing neighboring elements.
- Processing consecutive edges in a path.
- Calculating price changes over consecutive time intervals.

When the number of elements per window is variable or greater than two, `adjacent<N>` is generally the better choice.

## **Input Range Requirements**

### **The Forward Range Requirement**

One of the main limitations of `std::views::adjacent` is that its input Range must model `std::ranges::forward_range`.

Consequently, unlike some other Range Adaptors, it cannot directly operate on an ordinary Input Range that does not satisfy the Forward Range requirements.

To understand this restriction, consider the differences between several important Range concepts.

| Concept               | Main characteristic                                   |
| --------------------- | ----------------------------------------------------- |
| `input_range`         | Supports reading and traversing elements              |
| `forward_range`       | Supports repeated traversal and independent iterators |
| `bidirectional_range` | Supports moving backward                              |
| `random_access_range` | Supports random access to elements                    |

Every Forward Range is also an Input Range.

However, a Forward Range is not necessarily Bidirectional or Random Access.

Every Bidirectional Range and Random Access Range is also a Forward Range.

### **Why Does adjacent Require a Forward Range?**

To create a window containing `N` elements, `adjacent` must be able to manage multiple iterators simultaneously.

For example, with `adjacent<3>`, each output position represents three consecutive elements.

The implementation must maintain the iterators corresponding to these three positions and update them when advancing to the next window.

This design requires iterator copyability and the ability to traverse the Range repeatedly.

An ordinary Input Range may provide only single-pass iterators. Moving an iterator forward can invalidate the ability to reuse its previous traversal state.

This is why a Forward Range is the minimum requirement.

### **Using an Input Range**

The following expression is not valid for a Range that models only `input_range`:

```
auto result = input_range | std::views::adjacent<2>;
```

If the data source can only be read once, an appropriate storage or conversion strategy must be considered.

For example, the input data can first be stored in a container:

```
#include <ranges>
#include <vector>

int main() {
    auto input = std::views::iota(0)
        | std::views::take(5);

    std::vector<int> values(input.begin(), input.end());

    auto pairs = values | std::views::adjacent<2>;
}
```

In this example, the input data is stored in a `vector`, after which `adjacent` is applied to it.

However, this additional step is unnecessary when the data is already available through a Forward Range.

## **Understanding the Internal Implementation of adjacent_view**

### **Introduction to adjacent_view**

The `std::views::adjacent` facility is ultimately implemented using the following class:

```
std::ranges::adjacent_view<V, N>
```

Here:

- `V` is the underlying View type.
- `N` is the number of elements in each window.

Conceptually, its definition looks like this:

```
template<std::ranges::forward_range V, std::size_t N>
    requires std::ranges::view<V> && (N > 0)
class adjacent_view;
```

This definition demonstrates that `N` is a compile-time Template parameter.

Therefore, its value must be known at compile time.

### **The Role of Internal Iterators**

The internal iterator of `adjacent_view` does not use a single iterator to represent a window.

Instead, it maintains a collection of `N` iterators.

Conceptually:

```
std::array<iterator_type, N> current;
```

Each iterator points to one of the elements in the current window.

For example, with `adjacent<3>`, the iterator state looks like this:

```
Window 1:
Iterator 1 -> Element 1
Iterator 2 -> Element 2
Iterator 3 -> Element 3

Window 2:
Iterator 1 -> Element 2
Iterator 2 -> Element 3
Iterator 3 -> Element 4
```

When advancing to the next position, all internal iterators move forward by one step.

This design avoids creating a new container for each window.

### **Dereference Behavior**

Dereferencing the internal iterator produces a tuple containing the elements of the current window.

Conceptually, the result has a structure similar to:

```
std::tuple<
    reference_type,
    reference_type,
    reference_type
>
```

For `adjacent<3>`.

The exact types of the elements depend on the reference types provided by the underlying Range iterators.

### **Increment Behavior**

When the output iterator advances by one position, its internal iterators also advance.

For example:

```
Before increment:

[10, 20, 30]

After increment:

[20, 30, 40]
```

As a result, the next window is obtained using the updated iterators.

This design avoids unnecessary allocation and copying of the original elements.

## **Iterator Capabilities**

One important consideration for professional developers is understanding the iterator capabilities of `adjacent_view`.

This facility always produces a `forward_range` and preserves stronger traversal capabilities when supported by the underlying Range.

### **Bidirectional Range Support**

If the underlying Range is a `bidirectional_range`, the resulting View also supports moving backward.

For example:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    auto windows = values | std::views::adjacent<2>;

    auto it = windows.end();

    --it;

    auto [first, second] = *it;

    std::cout << first << ' ' << second;
}
```

Output:

```
4 5
```

In this example, the iterator moves backward one position from the end of the View.

### **Random Access Range Support**

If the underlying Range models `random_access_range`, the resulting View also supports random access.

Consequently, operations such as the following are available:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    auto windows = values | std::views::adjacent<2>;

    auto it = windows.begin();

    it += 2;

    auto [first, second] = *it;

    std::cout << first << ' ' << second;
}
```

Output:

```
30 40
```

This capability is useful for algorithms that require random access to windows.

### **An Important Note About iterator_category**

According to the standard specification, the `iterator_category` of the iterator used by `adjacent_view` is `std::input_iterator_tag`.

At the same time, the iterator's actual traversal capabilities are described by its `iterator_concept`.

These two concepts are not identical.

In particular, `iterator_concept` can be Forward, Bidirectional, or Random Access, depending on the capabilities of the underlying Range.

This design provides compatibility with older iterators while supporting the newer iterator concepts introduced by Ranges.

Therefore, it is better to use standard concepts such as the following to check actual iterator capabilities:

```
std::ranges::forward_range
std::ranges::bidirectional_range
std::ranges::random_access_range
```

You should not draw conclusions about the actual capabilities of the output iterator solely by examining `iterator_category`.

## **Understanding size Behavior**

### **Calculating the Number of Windows**

If the underlying Range models `sized_range`, the resulting View also models `sized_range`.

The number of elements is calculated using:

\\[ \max(0, S-N+1) \\]

Where:

- `S` is the number of elements in the original Range.
- `N` is the window size.

For example:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    auto windows = values | std::views::adjacent<3>;

    std::cout << windows.size();
}
```

Output:

```
3
```

### **Empty and Small Ranges**

If the underlying Range contains fewer elements than the requested window size, the output is empty.

For example:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2};

    auto windows = values | std::views::adjacent<3>;

    std::cout << std::ranges::empty(windows);
}
```

Output:

```
1
```

This behavior follows naturally from the definition of `adjacent`, so there is no need to manually check the Range size before using it.

## **Understanding the N Parameter and Zero**

### **Is adjacent Valid?**

An important detail of this feature is its behavior when `N` is zero.

In C++23, the following expression is valid for a Forward Range:

```
auto result = range | std::views::adjacent<0>;
```

In this case, the result is an empty View associated with `std::tuple<>`.

However, this behavior should not be interpreted as creating windows containing zero elements.

Instead, `views::adjacent<0>` is a special case in the Range Adaptor definition. Rather than creating an `adjacent_view`, it returns an Empty View.

### **adjacent and Input Ranges**

An important detail is that even the zero-sized special case does not bypass the Forward Range requirement.

According to the standard defect resolution LWG 4098, using `views::adjacent<0>` with an Input Range that does not model `forward_range` is not permitted.

Therefore, zero should not be used to bypass the requirements of `adjacent`.

### **Choosing an Appropriate N Value**

In typical applications, `N` should be selected according to the number of elements required by the algorithm.

| N             | Application                                            |
| ------------- | ------------------------------------------------------ |
| `1`           | Creating single-element windows                        |
| `2`           | Comparing adjacent elements                            |
| `3`           | Examining three-element patterns                       |
| `4`           | Processing four-element windows                        |
| Larger values | Processing fixed-size windows containing more elements |

## **Combining adjacent with Other Range Adaptors**

One of the main advantages of `std::views::adjacent` is its ability to compose with other features of the Ranges library.

By itself, this feature only creates windows. Combining it with other Views makes it possible to implement more complex operations using simpler code.

### **Combining with filter**

Suppose we want to select only adjacent pairs in which the second element is greater than the first.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 15, 30, 25, 40};

    auto increasing_pairs =
        values
        | std::views::adjacent<2>
        | std::views::filter([](const auto& pair) {
            return std::get<0>(pair) < std::get<1>(pair);
        });

    for (auto [first, second] : increasing_pairs) {
        std::cout << first << " -> " << second << '\n';
    }
}
```

Output:

```
10 -> 20
15 -> 30
25 -> 40
```

In this example:

- `adjacent<2>` creates pairs of adjacent elements.
- `filter` retains only pairs that satisfy the specified condition.

This composition does not require an intermediate container.

### **Combining with transform**

In many cases, the goal is not simply to inspect windows but to calculate a new value from each window.

For example, we can calculate the difference between two consecutive elements.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 15, 25, 40, 60};

    auto differences =
        values
        | std::views::adjacent<2>
        | std::views::transform([](const auto& pair) {
            return std::get<1>(pair) - std::get<0>(pair);
        });

    for (int difference : differences) {
        std::cout << difference << ' ';
    }
}
```

Output:

```
5 10 15 20
```

In this case, the final output is a View of calculated values.

An important detail is that `transform` also uses lazy evaluation.

Consequently, the difference calculation is not performed until the output elements are traversed.

### **Combining with take**

Sometimes only a limited number of the initial windows are needed.

In this situation, `std::views::take` can be used.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6, 7};

    auto windows =
        values
        | std::views::adjacent<3>
        | std::views::take(2);

    for (auto [a, b, c] : windows) {
        std::cout << a << ' ' << b << ' ' << c << '\n';
    }
}
```

Output:

```
1 2 3
2 3 4
```

### **Combining with drop**

To skip a specified number of initial windows, use `std::views::drop`.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6};

    auto windows =
        values
        | std::views::adjacent<2>
        | std::views::drop(2);

    for (auto [a, b] : windows) {
        std::cout << a << ' ' << b << '\n';
    }
}
```

Output:

```
3 4
4 5
5 6
```

### **Combining with iota**

`std::views::iota` is useful for generating numeric sequences.

Since a typical `iota_view` models at least a Forward Range, and commonly a Random Access Range, it can be combined directly with `adjacent`.

```
#include <iostream>
#include <ranges>

int main() {
    auto numbers = std::views::iota(1, 6);

    for (auto [first, second] : numbers | std::views::adjacent<2>) {
        std::cout << first << ", " << second << '\n';
    }
}
```

Output:

```
1, 2
2, 3
3, 4
4, 5
```

## **Introduction to std::views::adjacent_transform**

An important related feature is `std::views::adjacent_transform`, which was also introduced in C++23.

It also operates on consecutive windows, but instead of producing a tuple containing the adjacent elements, it invokes a function on each window and places the function's result in the output View.

Its general syntax is:

```
std::views::adjacent_transform<N>(function);
```

### **Comparing adjacent with adjacent_transform**

Suppose we want to calculate the difference between two consecutive elements.

Using `adjacent` and `transform`:

```
auto differences =
    values
    | std::views::adjacent<2>
    | std::views::transform([](const auto& pair) {
        return std::get<1>(pair) - std::get<0>(pair);
    });
```

Using `adjacent_transform`:

```
auto differences =
    values
    | std::views::adjacent_transform<2>(
        [](int first, int second) {
            return second - first;
        }
    );
```

The second approach invokes the function directly with the elements of each window.

As a result, there is no need to receive and process a tuple inside the transformation function.

### **Practical adjacent_transform Example**

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 15, 25, 40, 60};

    auto differences =
        values
        | std::views::adjacent_transform<2>(
            [](int first, int second) {
                return second - first;
            }
        );

    for (int difference : differences) {
        std::cout << difference << ' ';
    }
}
```

Output:

```
5 10 15 20
```

### **When Is adjacent_transform More Appropriate?**

`adjacent_transform` is generally more appropriate in the following situations:

- When only the calculated result of each window is needed.
- When tuple handling is unnecessary.
- When the primary goal is transforming adjacent elements.
- When readability of the Range pipeline is important.

In contrast, if the actual window elements need to be inspected, modified, or used in several different operations, `adjacent` provides greater flexibility.

## **Comparing adjacent with slide**

Another feature related to window processing is `std::views::slide`.

This feature was also introduced in C++23.

Both features create sliding windows, but their output structures are different.

### **Structural Differences**

With `adjacent`, each position produces a `tuple` containing consecutive elements.

With `slide`, each position produces a View containing the elements of the corresponding window.

For example, with `adjacent<3>`:

```
auto windows = values | std::views::adjacent<3>;
```

The conceptual output is:

```
tuple(10, 20, 30)
tuple(20, 30, 40)
tuple(30, 40, 50)
```

With `slide(3)`:

```
auto windows = values | std::views::slide(3);
```

The conceptual output is:

```
view(10, 20, 30)
view(20, 30, 40)
view(30, 40, 50)
```

### **Comparison Table**

| Feature                       | adjacent                                       | slide                                        |
| ----------------------------- | ---------------------------------------------- | -------------------------------------------- |
| Standard                      | C++23                                          | C++23                                        |
| Type of each window           | `tuple`                                        | View                                         |
| Number of elements per window | Fixed at compile time                          | Specified at runtime                         |
| Element access                | `std::get` or Structured Bindings              | Range interface                              |
| Modifying original elements   | Possible when elements are references          | Possible through the View and its references |
| Direct Structured Bindings    | Yes                                            | Not directly in the usual form               |
| Primary use case              | Processing a fixed number of adjacent elements | Processing sliding windows as Ranges         |

### **Choosing Between adjacent and slide**

If the number of elements per window is known in advance and we want to access them as a fixed collection, `adjacent` is a suitable choice.

However, if each window needs to be processed as an independent Range, `slide` is generally more appropriate.

For example, using `slide` to calculate the average of a multi-element window can be more readable:

```
#include <iostream>
#include <numeric>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    auto windows = values | std::views::slide(3);

    for (auto window : windows) {
        int sum = std::accumulate(window.begin(), window.end(), 0);

        std::cout << sum / 3 << ' ';
    }
}
```

Output:

```
20 30 40
```

## **Comparing adjacent with chunk**

`std::views::chunk` is another feature introduced in C++23.

It divides the elements of a Range into consecutive groups.

The main difference between `chunk` and `adjacent` is whether consecutive windows overlap.

### **Comparison Example**

Suppose the input data is:

```
std::vector<int> values{1, 2, 3, 4, 5, 6};
```

Using `adjacent<3>`:

```
(1, 2, 3)
(2, 3, 4)
(3, 4, 5)
(4, 5, 6)
```

Using `chunk(3)`:

```
(1, 2, 3)
(4, 5, 6)
```

With `chunk`, elements are not shared between consecutive groups.

In contrast, `adjacent` creates overlapping windows.

### **Comparison Table**

| Feature                      | adjacent                    | chunk                                      |
| ---------------------------- | --------------------------- | ------------------------------------------ |
| Overlapping windows          | Yes                         | No                                         |
| Number of elements per group | N                           | Usually N, except possibly the final group |
| Window advancement           | One element                 | One group                                  |
| Primary use case             | Analyzing adjacent elements | Dividing a Range into groups               |

Therefore, `adjacent` is more appropriate when processing consecutive elements whose relationships across windows matter.

For dividing a collection into independent groups, `chunk` is the better choice.

## **Time and Space Complexity**

### **Time Complexity**

Since `adjacent` is a View, creating it does not calculate and store all windows.

View construction generally has constant complexity because it only stores the underlying Range and the information required for traversal.

Dereferencing a window involves accessing `N` elements.

Therefore, the dereference cost is proportional to `N`.

For traversing a complete Range containing `S` elements:

\\[ O((S-N+1)\times N) \\]

For a fixed `N`, the overall traversal complexity, measured in terms of element accesses, is linear in the size of the Range:

\\[ O(S) \\]

However, the actual cost of operations performed on the elements depends on the algorithm implemented by the programmer.

### **Space Complexity**

One of the main advantages of `adjacent` is that it does not need to create a new container to store the windows.

Its internal iterator conceptually maintains `N` iterators.

Therefore, the additional storage required for iterator state is proportional to `N`:

\\[ O(N) \\]

This amount is independent of the number of elements in the original Range.

Consequently, even for large Ranges, there is no need to allocate memory to store every window.

### **An Important Note About Lazy Evaluation**

`adjacent` follows the lazy evaluation model.

Creating a View does not mean that all windows are immediately calculated and stored.

Windows become available when the iterator is traversed and dereferenced.

For example:

```
auto windows = values | std::views::adjacent<2>;
```

At this point, only the View has been constructed.

The output windows are not fully generated and consumed until `windows` is traversed.

## **Practical Applications of std::views::adjacent**

### **Calculating Differences Between Consecutive Elements**

One of the most common applications of `adjacent` is calculating differences between consecutive values.

This is useful in time-series analysis, price change calculations, and sensor data processing.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> temperatures{
        20.5, 21.2, 22.8, 22.1, 24.0
    };

    auto changes =
        temperatures
        | std::views::adjacent_transform<2>(
            [](double previous, double current) {
                return current - previous;
            }
        );

    for (double change : changes) {
        std::cout << change << '\n';
    }
}
```

Output:

```
0.7
1.6
-0.7
1.9
```

In this example, the temperature change between consecutive measurements is calculated.

### **Checking Whether Data Is Sorted**

Another important application is checking whether a sequence is strictly increasing.

```
#include <algorithm>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 3, 5, 7, 9};

    bool strictly_increasing = std::ranges::all_of(
        values | std::views::adjacent<2>,
        [](const auto& pair) {
            return std::get<0>(pair) < std::get<1>(pair);
        }
    );

    std::cout << std::boolalpha << strictly_increasing;
}
```

Output:

```
true
```

In this algorithm, every consecutive pair is examined.

If even one pair violates the increasing-order condition, the result is `false`.

An important detail is that `all_of` returns `true` for an empty Range.

Therefore, this algorithm also returns `true` for a Range containing fewer than two elements.

If the application logic requires the sequence to contain at least two elements, that condition must be checked separately.

### **Detecting Turning Points**

Suppose we have a collection of values and want to identify positions where a value changes direction relative to its two neighbors.

This requires examining three consecutive elements.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        1, 3, 5, 4, 2, 6, 8, 7
    };

    auto turning_points =
        values
        | std::views::adjacent_transform<3>(
            [](int previous, int current, int next) {
                return (current > previous && current > next)
                    || (current < previous && current < next);
            }
        );

    int index = 1;

    for (bool is_turning_point : turning_points) {
        if (is_turning_point) {
            std::cout << "Turning point at index "
                      << index << '\n';
        }

        ++index;
    }
}
```

Output:

```
Turning point at index 2
Turning point at index 4
Turning point at index 6
```

In this example, each three-element window is examined to identify a local maximum or minimum.

### **Checking Data Continuity**

Some algorithms require the difference between consecutive values to remain below a specified threshold.

For example, measurements from a sensor can be checked for excessively large changes.

```
#include <algorithm>
#include <cmath>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> readings{
        10.0, 10.5, 10.8, 11.0, 15.5, 15.8
    };

    constexpr double max_change = 2.0;

    bool valid = std::ranges::all_of(
        readings | std::views::adjacent<2>,
        [](const auto& pair) {
            return std::abs(
                std::get<1>(pair) - std::get<0>(pair)
            ) <= max_change;
        }
    );

    std::cout << std::boolalpha << valid;
}
```

Output:

```
false
```

This is because the difference between `11.0` and `15.5` exceeds the permitted threshold.

### **Processing Paths and Coordinates**

In graphics and geometric algorithms, it is sometimes necessary to examine every pair of consecutive points in a path.

For example:

```
#include <cmath>
#include <iostream>
#include <ranges>
#include <vector>

struct Point {
    double x;
    double y;
};

int main() {
    std::vector<Point> path{
        {0.0, 0.0},
        {3.0, 4.0},
        {6.0, 8.0},
        {9.0, 8.0}
    };

    auto distances =
        path
        | std::views::adjacent_transform<2>(
            [](const Point& first, const Point& second) {
                const double dx = second.x - first.x;
                const double dy = second.y - first.y;

                return std::sqrt(dx * dx + dy * dy);
            }
        );

    for (double distance : distances) {
        std::cout << distance << '\n';
    }
}
```

Output:

```
5
5
3
```

In this example, the length of each path segment is calculated.

### **Detecting Three-Element Patterns**

Another application of `adjacent` is identifying specific patterns in sequences.

For example, suppose we want to find sequences in which the middle element is greater than both neighboring elements.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        2, 5, 3, 8, 4, 9, 6
    };

    auto peaks =
        values
        | std::views::adjacent<3>
        | std::views::filter([](const auto& window) {
            return std::get<1>(window) > std::get<0>(window)
                && std::get<1>(window) > std::get<2>(window);
        });

    for (auto [left, middle, right] : peaks) {
        std::cout << middle << '\n';
    }
}
```

Output:

```
5
8
9
```

This approach is useful for signal processing, time-series analysis, and local pattern detection.

## **Managing Range Lifetime**

One of the most important topics when working with Ranges is correctly managing the lifetime of the objects being used.

Since `adjacent` is a View, it generally does not own its underlying data. Instead, it depends on the lifetime of the underlying Range.

### **Using an lvalue Range**

Consider the following example:

```
std::vector<int> values{1, 2, 3, 4};

auto windows = values | std::views::adjacent<2>;
```

The `windows` variable depends on the data stored in `values`.

Therefore, `values` must remain alive for as long as the View is being used.

### **Using a Temporary Range**

One advantage of Ranges is that some temporary Ranges can also be used.

For example:

```
#include <ranges>

int main() {
    auto windows =
        std::views::iota(1, 10)
        | std::views::adjacent<2>;

    for (auto [first, second] : windows) {
        // Process adjacent values
    }
}
```

In this example, `iota_view` is a self-contained View that can be stored independently.

Consequently, the View pipeline can be constructed without requiring an intermediate container.

### **The Danger of Dangling References**

If a View depends on memory whose lifetime has ended, using that View can result in undefined behavior.

For example, the following pattern is dangerous:

```
#include <ranges>
#include <vector>

int main() {
    auto windows = [] {
        std::vector<int> values{1, 2, 3, 4};

        return values | std::views::adjacent<2>;
    }();

    // The underlying vector no longer exists.
}
```

In this example, `values` is destroyed when the Lambda finishes executing.

However, the returned View attempts to access elements of that `vector`.

Consequently, using `windows` after the Lambda returns is invalid.

An important detail is that `views::all` can behave differently depending on the type and value category of the underlying Range.

For an ordinary lvalue container, the resulting View typically stores a reference-like wrapper such as `ref_view`.

For a suitable rvalue container, ownership may instead be managed through `owning_view`.

Nevertheless, the lifetime of the underlying Range and the way the View is constructed must always be considered.

## **Limitations and Important Considerations**

### **Modifying the Size of the Underlying Container**

If a View is created from a container such as `std::vector`, operations that relocate memory or invalidate iterators can make the View unusable.

For example:

```
std::vector<int> values{1, 2, 3, 4};

auto windows = values | std::views::adjacent<2>;

values.push_back(5);
```

If `push_back` causes reallocation, the iterators maintained by the View may become invalid.

Therefore, Views should not be assumed to remain independent of structural changes to their underlying containers.

### **Modifying Range Elements During Traversal**

Modifying element values is generally possible when the underlying Range provides mutable elements.

However, modifying the container's structure, erasing elements, or performing operations that invalidate iterators is a different matter.

For example, changing an element's value in a `std::vector` generally does not invalidate its iterators, whereas inserting or erasing elements may do so.

### **Transformation Function Costs**

When using `adjacent_transform`, remember that the transformation function is evaluated when output elements are accessed.

If an output element is dereferenced multiple times, you should not assume that the function is necessarily invoked exactly once without considering the algorithm's evaluation behavior.

Therefore, transformation functions should ideally be pure and free of side effects.

### **Using Unbounded Ranges**

One useful feature of `adjacent` is that it can be used with certain infinite Ranges.

For example:

```
#include <iostream>
#include <ranges>

int main() {
    auto numbers = std::views::iota(1)
        | std::views::adjacent<3>
        | std::views::take(5);

    for (auto [a, b, c] : numbers) {
        std::cout << a << ' '
                  << b << ' '
                  << c << '\n';
    }
}
```

Output:

```
1 2 3
2 3 4
3 4 5
4 5 6
5 6 7
```

In this example, `iota` generates an unbounded sequence.

By combining `adjacent` with `take`, only the first five windows are processed.

An important consideration is that operations applied to an unbounded Range must provide a way to terminate traversal when necessary.

### **Using a Large N**

Since `N` is a Template parameter, the window size is determined at compile time.

The internal iterator also requires maintaining `N` iterators.

Therefore, when using very large windows, the cost of maintaining iterator state and processing each window should be considered.

If the window size needs to change at runtime, `adjacent` is not an appropriate choice. Features such as `slide` should be considered instead.

## **Common Mistakes**

### **Assuming adjacent Creates a New Container**

A common mistake is treating the output of `adjacent` as an independent collection of data.

In reality, it creates a View, and the underlying data generally remains in the original Range.

Consequently, modifications to referenced elements can affect the original data.

### **Assuming N Can Be Determined at Runtime**

The following expression is invalid:

```
std::size_t window_size = 3;

auto windows = values | std::views::adjacent<window_size>;
```

This is because `window_size` is an ordinary variable whose value is not available as a Template argument at compile time.

Instead, use a compile-time constant:

```
constexpr std::size_t window_size = 3;

auto windows = values | std::views::adjacent<window_size>;
```

### **Assuming the Output Is a pair**

With `adjacent<2>`, the output is a `std::tuple`, not necessarily a `std::pair`.

Therefore, elements should be accessed using `std::get` or Structured Bindings.

This distinction becomes important when examining the exact output type in Template code.

### **Using an Input Range**

A Range that models only `input_range` cannot be used directly.

The input must satisfy at least the `forward_range` requirements.

### **Using a View After the Underlying Range Is Destroyed**

If a View depends on a local container, it must not be used after that container has been destroyed.

This is one of the most important lifetime considerations when designing code with Ranges.

### **Assuming View Traversal Is Always Free**

Although constructing a View generally has a very low cost, traversing it is not free.

Every window must be dereferenced, and its elements must be made available to the algorithm.

Furthermore, the cost of operations performed on each window depends on the implementation.

## **Compiler and Standard Compatibility**

### **Standard Version**

The following features were introduced in C++23:

- `std::views::adjacent`
- `std::views::pairwise`
- `std::ranges::adjacent_view`
- `std::views::adjacent_transform`
- `std::views::pairwise_transform`
- `std::ranges::adjacent_transform_view`

Therefore, using these features requires a compiler and standard library implementation that supports the corresponding C++23 functionality.

### **Enabling C++23**

The following commands can be used to enable C++23 mode.

**GCC:**

```
g++ -std=c++23 main.cpp -o main
```

**Clang:**

```
clang++ -std=c++23 main.cpp -o main
```

**MSVC:**

```
cl /std:c++latest /EHsc main.cpp
```

An important detail is that enabling C++23 mode does not guarantee that every C++23 feature has been implemented in the selected compiler and standard library version.

Actual support depends on the compiler and its standard library implementation.

The documentation for the specific implementation should be consulted to verify feature availability.

### **Checking the Feature-Test Macro**

The following Feature-Test Macro can be used to check support for `adjacent`:

```
#include <ranges>

#ifndef __cpp_lib_ranges_zip
#error "Required ranges library features are unavailable"
#endif
```

However, `__cpp_lib_ranges_zip` provides a broader check for a group of related Ranges features, including `zip`. It is not an `adjacent`-specific feature test.

For a more direct check, a compile-time test can be used:

```
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto windows = values | std::views::adjacent<2>;

    static_assert(std::ranges::forward_range<decltype(windows)>);
    static_assert(std::ranges::sized_range<decltype(windows)>);
}
```

This approach verifies that the feature is available in the current compilation environment and that the resulting View satisfies the expected concepts.

## **Comparing adjacent with Older Approaches**

### **Using Iterator Loops**

Before `adjacent` was introduced, one common approach was to use two iterators.

For example:

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    for (auto first = values.begin();
         first != values.end() && std::next(first) != values.end();
         ++first) {

        auto second = std::next(first);

        std::cout << *first << " -> " << *second << '\n';
    }
}
```

This approach is suitable for processing two adjacent elements.

However, managing iterators becomes more complicated when working with larger windows.

For example, a five-element window requires maintaining several iterators and ensuring that none of them advances beyond the end of the Range.

In contrast, `adjacent` provides this functionality as a standard Range Adaptor.

### **Using Index-Based Loops**

Another common approach in older code is using indices:

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    for (std::size_t i = 0; i + 1 < values.size(); ++i) {
        std::cout << values[i] << " -> "
                  << values[i + 1] << '\n';
    }
}
```

This approach is simple for containers that support random access.

However, it cannot be used directly with Ranges that do not provide indexed access.

It also introduces a stronger dependency on the underlying container type.

In contrast, `adjacent` is designed around Range concepts and works with different types of Forward Ranges.

### **Using std::adjacent_find**

The `std::adjacent_find` algorithm was introduced in C++98.

It is used to find the first pair of adjacent elements that are equal or satisfy a specified Predicate.

However, its purpose differs from that of `std::views::adjacent`.

For example:

```
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 3, 4, 5};

    auto it = std::adjacent_find(values.begin(), values.end());

    if (it != values.end()) {
        std::cout << *it;
    }
}
```

Output:

```
3
```

In this example, only the first pair of equal adjacent elements is found.

In contrast, `adjacent` exposes all consecutive windows through a View.

Therefore, these two features are not direct replacements for one another.

## **Design Considerations for Professional Developers**

### **Choosing Between adjacent and adjacent_transform**

When designing Range-based pipelines, first determine whether the actual window is needed or whether only its calculated result is required.

If the goal is simply to calculate one value from each window, `adjacent_transform` generally provides a more direct implementation.

However, if the window elements are needed in multiple processing stages, `adjacent` offers greater flexibility.

### **Paying Attention to Reference Types**

When writing Generic Code, avoid assuming that every Range element is an ordinary reference.

Some Views may produce proxy references or values.

Therefore, using `auto&&` and Generic Lambdas is often a more suitable approach.

### **Avoiding Side Effects in Lazy Views**

Since Views use lazy evaluation, transformation functions should not contain sensitive side effects without considering when and how often they may be evaluated.

For example, performing I/O, modifying global state, or incrementing external counters inside pipeline Lambdas can lead to unexpected behavior when the pipeline is traversed multiple times.

View operations should ideally remain pure.

### **Considering the Capabilities of the Underlying Range**

The iterator capabilities of `adjacent` depend on the capabilities of the underlying Range.

If the original Range is Random Access, random access to windows is available.

However, if the original Range is only Forward, random access operations should not be expected from the resulting View.

### **Considering Data Ownership**

When designing APIs that return Views, it is important to understand which data the View depends on.

If a View refers to a local container, returning it can result in dangling references.

This issue is not specific to `adjacent`; it is important in all designs based on `std::views`.

## **Summary**

`std::views::adjacent` is an important feature of the C++23 Ranges library that simplifies processing multiple consecutive elements.

By creating a View of overlapping windows, it provides simultaneous access to a specified number of consecutive elements from a Range.

Its most important characteristics and considerations include:

- `adjacent` was introduced in C++23.
- The number of elements in each window is specified through the Template parameter `N`.
- Each window contains a `std::tuple` of consecutive elements.
- Output elements generally refer to elements in the original Range.
- The feature requires at least a `forward_range`.
- The output size can be calculated when the underlying Range is sized.
- Depending on the capabilities of the underlying Range, the resulting View can support Bidirectional or Random Access traversal.
- The value of `N` must be known at compile time.
- `std::views::pairwise` is equivalent to `std::views::adjacent<2>`.
- `std::views::adjacent_transform` is suitable for directly transforming windows.
- `std::views::slide` is more appropriate when each window needs to be processed as an independent Range.
- `std::views::chunk` is used to divide a Range into non-overlapping groups.
- Since `adjacent` is a View, care must be taken to manage the lifetime of the underlying Range and the validity of its iterators.

Overall, when an algorithm needs to examine multiple consecutive elements, `std::views::adjacent` is a useful standard feature for modern C++ development.

## **References and Further Reading**

For additional information and details about the standard, consult the following resources:

- [cppreference - std::ranges::adjacent_view](https://en.cppreference.com/w/cpp/ranges/adjacent_view?utm_source=chatgpt.com)
- [C++ Working Draft - Adjacent View](https://eel.is/c++draft/range.adjacent?utm_source=chatgpt.com)
- [C++ Working Draft - Adjacent Transform View](https://eel.is/c++draft/range.adjacent.transform?utm_source=chatgpt.com)
- [C++ Working Draft - Range Adaptors](https://eel.is/c++draft/range.adaptors?utm_source=chatgpt.com)
- [cppreference - Range Adaptors](https://en.cppreference.com/w/cpp/ranges.html?utm_source=chatgpt.com)
- [cppreference - std::views::slide](https://en.cppreference.com/w/cpp/ranges/slide_view?utm_source=chatgpt.com)
- [cppreference - std::views::chunk](https://en.cppreference.com/w/cpp/ranges/chunk_view?utm_source=chatgpt.com)
- [cppreference - std::views::adjacent_transform](https://en.cppreference.com/w/cpp/ranges/adjacent_transform_view?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>