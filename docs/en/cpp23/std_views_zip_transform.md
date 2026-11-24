<div align="center">

[🇺🇸 English](./std_views_zip_transform.md) | [🇮🇷 فارسی](../../fa/cpp23/std_views_zip_transform.md)

</div>

---

# **A Comprehensive Guide to&#x20;**`std::views::zip_transform`**&#x20;in C++23**

## **Introduction**

The Ranges library in C++20 introduced powerful capabilities for processing collections of data in a declarative and composable manner. These capabilities make it possible to perform operations such as filtering, transforming, and traversing data without writing repetitive loops or manually managing iterators.

Despite these improvements, one common programming requirement was not directly addressed: **processing multiple ranges simultaneously and applying a function to their corresponding elements**.

For example, suppose we have two arrays of numbers and want to add their corresponding elements. The traditional approach uses a loop to access both arrays simultaneously. With Ranges, we can also combine them using `std::views::zip` and then apply a transformation to produce the result.

The `std::views::zip_transform` facility in C++23 was designed specifically to simplify these scenarios.

It allows us to traverse multiple ranges simultaneously and invoke a callable on their corresponding elements without first creating a new container or manually storing intermediate results.

In simple terms, `std::views::zip_transform` combines two concepts:

- **Zip:** Combining corresponding elements from multiple ranges.
- **Transform:** Applying a function to those elements to produce a result.

This facility is part of the standard library and is provided as a Range Adaptor. Its standardization was accomplished through proposal `P2321R2`. In addition to `zip_transform`, this proposal introduced other related facilities, including `zip`, `adjacent`, and `adjacent_transform`.

## **Core Concept and How It Works**

### **What Is std::views::zip_transform?**

`std::views::zip_transform` is a Range Adaptor that accepts a callable and one or more ranges, then creates a new view.

If we represent the input function as \\(f\\) and the input ranges as \\(R_1\\), \\(R_2\\), up to \\(R_n\\), the element at position \\(i\\) in the output is:

\\[ V[i] = f(R_1[i], R_2[i], \ldots, R_n[i]) \\]

For example, consider the following two ranges:

```
std::vector<int> a{1, 2, 3, 4};
std::vector<int> b{10, 20, 30, 40};
```

If we want to add their corresponding elements:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4};
    std::vector<int> b{10, 20, 30, 40};

    auto result = std::views::zip_transform(
        [](int x, int y) {
            return x + y;
        },
        a,
        b
    );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

Output:

```
11 22 33 44
```

In this example, the Lambda function is invoked with the corresponding elements from both ranges at each position.

An important detail is that `result` is not a new container. It is a view that describes how to calculate the output elements.

### **The Difference Between a View and a Container**

To understand the behavior of `zip_transform` correctly, it is important to understand the difference between a View and a Container.

A container such as `std::vector` is generally responsible for storing and managing data. A view, on the other hand, typically provides a representation of existing data or a computation based on that data.

For example:

```
std::vector<int> values{1, 2, 3, 4};

auto transformed = values | std::views::transform(
    [](int value) {
        return value * 2;
    }
);
```

In this example, the new view calculates its elements when they are accessed.

`zip_transform` follows the same approach, except that it operates on multiple ranges instead of just one.

Consequently:

- No new container is created to store the results.
- Elements are calculated when the view's iterator is dereferenced.
- Computed results are not automatically cached in memory.
- If the results need to be reused, they can be materialized into a container.

### **Lazy Evaluation**

One of the most important characteristics of `zip_transform` is its use of Lazy Evaluation.

This means that creating the view does not execute the transformation function for every element.

For example:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4};

    auto result = std::views::zip_transform(
        [](int value) {
            std::cout << "Processing: " << value << '\n';
            return value * 10;
        },
        values
    );

    std::cout << "View created\n";

    auto it = result.begin();

    std::cout << *it << '\n';
}
```

Output:

```
View created
Processing: 1
10
```

In this example, the Lambda is executed only when the iterator associated with the view is dereferenced.

This behavior provides several advantages:

- Avoiding unnecessary computations.
- Reducing memory consumption.
- Combining multiple Range Adaptors without creating intermediate containers.
- Processing data incrementally.

However, lazy evaluation does not imply memoization. If an iterator is dereferenced multiple times, the transformation function may be executed again each time.

This is an important consideration when designing the callable.

## **Using std::views::zip_transform**

### **Required Header**

The `zip_transform` facility is defined in the following header:

```
#include <ranges>
```

For examples that use `std::vector`, the following header is also required:

```
#include <vector>
```

### **General Syntax**

The general form of this Range Adaptor is:

```
auto result = std::views::zip_transform(
    callable,
    range1,
    range2,
    range3
);
```

The parameters are:

- `callable`: A function, Lambda, Function Object, or any other valid callable object.
- `range1`, `range2`, ...: The input ranges.

At least one input range must be provided.

Additionally, all input ranges must satisfy the standard requirements for use with this view.

### **Using a Lambda**

The most common way to use `zip_transform` is to pass a Lambda.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> prices{100, 200, 300};
    std::vector<int> quantities{2, 3, 4};

    auto totals = std::views::zip_transform(
        [](int price, int quantity) {
            return price * quantity;
        },
        prices,
        quantities
    );

    for (auto total : totals) {
        std::cout << total << '\n';
    }
}
```

Output:

```
200
600
1200
```

In this example, the Lambda receives the price and quantity of each corresponding product and calculates its total cost.

### **Using a Function Object**

A Lambda is not the only available option. We can also use a Function Object.

```
#include <iostream>
#include <ranges>
#include <vector>

struct Multiply {
    constexpr int operator()(int a, int b) const {
        return a * b;
    }
};

int main() {
    std::vector<int> a{2, 3, 4};
    std::vector<int> b{5, 6, 7};

    auto result = std::views::zip_transform(
        Multiply{},
        a,
        b
    );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

Output:

```
10 18 28
```

Using a Function Object is useful when:

- The computational logic is complex.
- The operation requires explicit internal state.
- We want to reuse a callable in different parts of the program.
- We want to extend its behavior using templates or overloading.

### **Using Standard Library Functions**

We can also use Function Objects provided by the standard library.

For example, we can use `std::plus` to add elements from two ranges:

```
#include <functional>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

Output:

```
11 22 33
```

In this example, `std::plus<>` is a Function Object that uses template argument deduction to determine the operation's argument types.

For operations such as addition, subtraction, multiplication, and comparison, standard Function Objects can make the code simpler and more readable.

## **Relationship with std::views::zip**

### **Introducing zip**

To understand `zip_transform` better, we should first examine `std::views::zip`.

`std::views::zip` was also introduced in C++23. It combines multiple ranges into a single composite range.

Each output element contains the corresponding elements from the input ranges.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{1, 2, 3};
    std::vector<char> letters{'A', 'B', 'C'};

    auto zipped = std::views::zip(numbers, letters);

    for (auto [number, letter] : zipped) {
        std::cout << number << ' ' << letter << '\n';
    }
}
```

Output:

```
1 A
2 B
3 C
```

Here, each output element is a tuple-like object containing the corresponding values from both ranges.

### **Differences Between zip and zip_transform**

The primary difference between these two facilities is the type of their output.

| Feature                 | `std::views::zip`             | `std::views::zip_transform`       |
| ----------------------- | ----------------------------- | --------------------------------- |
| Input parameters        | One or more ranges            | A callable and one or more ranges |
| Output                  | A range of tuple-like objects | Results of invoking the callable  |
| Function execution      | None                          | On dereference                    |
| Lazy evaluation         | Yes                           | Yes                               |
| Creates a new container | No                            | No                                |
| Primary purpose         | Simultaneous traversal        | Simultaneous processing           |

For example, the following code combines elements from two ranges using `zip`:

```
auto zipped = std::views::zip(a, b);
```

However, the following code directly adds the corresponding elements:

```
auto transformed = std::views::zip_transform(
    std::plus<>{},
    a,
    b
);
```

Conceptually, the behavior of `zip_transform` is similar to applying `std::views::transform` to the output of `std::views::zip`. However, these two expressions are not necessarily identical in terms of their view types or implementation details.

In fact, `zip_transform` is specifically provided by the standard library for this use case.

### **Manually Combining zip and transform**

Before `zip_transform` was introduced, we could implement similar functionality by combining two Range Adaptors.

```
#include <functional>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    auto result =
        std::views::zip(a, b)
        | std::views::transform(
            [](const auto& values) {
                return std::get<0>(values)
                     + std::get<1>(values);
            }
        );
}
```

This approach is valid, but extracting elements from tuple-like objects can make the code more complicated, especially when the number of input ranges increases.

By contrast, `zip_transform` passes the elements directly to the callable:

```
auto result = std::views::zip_transform(
    std::plus<>{},
    a,
    b
);
```

This approach:

- Is shorter.
- Expresses the intended operation more directly.
- Eliminates the need to extract elements from a tuple.
- Scales more naturally to a larger number of input ranges.

## **Internal Structure and Standard Requirements**

### **Introducing zip_transform_view**

The actual view type produced by `std::views::zip_transform` is located in the following namespace:

```
std::ranges::zip_transform_view
```

This class template is responsible for storing the callable and the input views.

A simplified representation of its template constraints looks like this:

```
namespace std::ranges {

    template<
        std::move_constructible F,
        std::ranges::input_range... Views
    >
    requires
        (std::ranges::view<Views> && ...) &&
        (sizeof...(Views) > 0) &&
        std::is_object_v<F> &&
        std::regular_invocable<
            F&,
            std::ranges::range_reference_t<Views>...
        > &&
        requires {
            typename std::invoke_result_t<
                F&,
                std::ranges::range_reference_t<Views>...
            >;
        }
    class zip_transform_view;

}
```

This code is intended only as an educational representation of the template requirements and is not the complete standard definition.

In the actual standard, the requirement that the result type be a valid referenceable type is also expressed using the internal `can-reference` concept.

### **The move_constructible Requirement**

The callable must satisfy the `std::move_constructible` concept.

This means that the callable object must be capable of being moved into the view.

For example:

```
auto callable = [](int value) {
    return value * 2;
};

auto result = std::views::zip_transform(
    std::move(callable),
    values
);
```

For ordinary Lambdas, moving the callable is generally straightforward.

An important detail is that the standard does not require the callable to be Copyable. The primary requirement is that it be `move_constructible`.

### **The input_range Requirement**

Each input range must model the `std::ranges::input_range` concept.

This concept specifies that a range provides at least the capabilities required for input iteration.

Consequently, many different range types can be used, including:

- `std::vector`
- `std::list`
- `std::array`
- `std::span`
- `std::ranges::subrange`
- Various standard views
- Custom ranges that satisfy the required constraints

The input ranges do not have to be containers.

### **The view Requirement**

The internal view types must model the `std::ranges::view` concept.

An important detail is that the public `std::views::zip_transform` adaptor can accept containers and other `viewable_range` types.

When necessary, the standard library converts them into suitable views.

For example:

```
std::vector<int> values{1, 2, 3};

auto result = std::views::zip_transform(
    [](int value) {
        return value * 3;
    },
    values
);
```

In this example, there is no need to convert `values` into a view manually.

### **The regular_invocable Requirement**

One of the most important callable requirements is `std::regular_invocable`.

Structurally, this concept is similar to `std::invocable`, but it has additional semantic requirements.

In particular, the invocation operation must be equality-preserving in the semantic sense.

This matters for views that perform computations when their iterators are dereferenced.

For example, the following function is syntactically a valid callable:

```
int counter = 0;

auto callable = [&counter](int value) {
    ++counter;
    return value + counter;
};
```

However, its behavior changes with every invocation.

Such a callable does not satisfy the semantic requirements of `regular_invocable`.

It is important to note that a compiler generally cannot automatically verify all the semantic requirements of this concept. Therefore, code like this may compile while still failing to satisfy the standard's requirements.

As a result, callables used with `zip_transform` should ideally be Pure and free of side effects.

### **The can-reference Requirement**

The callable's return type must also satisfy specific requirements.

The standard uses an internal concept called `can-reference`.

In simplified terms, this concept checks whether a valid reference can be formed to the result type.

For this reason, outputs such as `void` are not valid for `zip_transform`.

For example:

```
auto invalid = std::views::zip_transform(
    [](int value) {
        std::cout << value;
    },
    values
);
```

This code is invalid because the Lambda returns `void`.

By contrast, the following code is valid:

```
auto valid = std::views::zip_transform(
    [](int value) {
        return value * 2;
    },
    values
);
```

Therefore, `zip_transform` is designed to produce a range of results, not merely to execute side-effecting operations on elements.

## **Iterator Behavior and Range Properties**

### **Iterator Capabilities**

The iterator type associated with `zip_transform_view` derives its capabilities from the internal `zip_view` iterator.

In general, if the underlying composite view supports a particular range capability, `zip_transform_view` can provide that capability as well.

The most important supported concepts include:

- `input_range`
- `forward_range`
- `bidirectional_range`
- `random_access_range`
- `common_range`
- `sized_range`

For example, if all input ranges support random access, the resulting view can also be a `random_access_range`.

Example:

```
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{4, 5, 6};

    auto result = std::views::zip_transform(
        [](int x, int y) {
            return x + y;
        },
        a,
        b
    );

    static_assert(
        std::ranges::random_access_range<
            decltype(result)
        >
    );

    auto value = result[1];
}
```

In this example, both inputs are `std::vector` objects, so the resulting view also supports random access.

### **Random Access to Elements**

When random access is supported, the indexing operator can be used.

```
auto result = std::views::zip_transform(
    std::plus<>{},
    a,
    b
);

int value = result[2];
```

In this case, the corresponding elements at the third position in both ranges are retrieved and passed to the addition function.

This capability is useful for algorithms that require direct access to different positions.

### **Reverse Iteration**

If all the necessary conditions for a bidirectional range are satisfied, the output view can be traversed in reverse.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );

    for (auto it = result.rbegin();
         it != result.rend();
         ++it) {
        std::cout << *it << ' ';
    }
}
```

Output:

```
33 22 11
```

The availability of `rbegin` and `rend` follows from the range facilities and `view_interface`, subject to the capabilities of the underlying view.

### **The Sized Range Property**

If the underlying `zip_view` models `sized_range`, the resulting view also supports size calculation.

For `zip_transform`, the output size is equal to the smallest size among the input ranges.

\\[ Size(V) = \min(Size(R_1), Size(R_2), \ldots, Size(R_n)) \\]

For example:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4, 5};
    std::vector<int> b{10, 20, 30};

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );

    std::cout << std::ranges::size(result);
}
```

Output:

```
3
```

In this example, the first range contains five elements, while the second contains three. Therefore, the resulting view contains only three elements.

## **Handling Ranges with Different Sizes**

### **Behavior When Sizes Differ**

One important aspect of `zip_transform` is its behavior when given ranges of different sizes.

Like `std::views::zip`, it follows the shortest-range rule.

In other words, iteration ends when at least one of the input ranges reaches its end.

For example:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4, 5};
    std::vector<int> b{10, 20, 30};

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

Output:

```
11 22 33
```

The fourth and fifth elements of the first range are not processed.

This behavior is similar to the `zip` operation found in many programming languages.

### **Why Is the Shortest Range Used?**

Suppose we have two arrays of different sizes and want to add their corresponding elements.

If iteration continued until the larger range reached its end, we might attempt to access invalid elements in the smaller range.

To avoid this situation, the standard defines the behavior so that iteration stops at the shortest range.

### **Checking Whether Ranges Have Equal Sizes**

An important detail is that `zip_transform` does not require all input ranges to have the same size.

If the program's logic requires processing every element, their sizes should be checked separately.

```
#include <cassert>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    assert(std::ranges::size(a) ==
           std::ranges::size(b));

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );
}
```

This is particularly important when processing financial, scientific, statistical, or tabular data, where different input sizes may indicate an error in the input data.

## **Managing References and Value Categories**

### **Accessing the Original Elements**

An important characteristic of `zip_transform` is that the callable receives elements according to the reference types associated with their respective ranges.

For example, when a `std::vector<int>` is passed to the view, its reference type is generally `int&`.

Consequently, the original elements can be modified without copying them.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto result = std::views::zip_transform(
        [](int& value) -> int& {
            return value;
        },
        values
    );

    for (int& value : result) {
        value *= 10;
    }

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

Output:

```
10 20 30
```

In this example, the Lambda returns a reference to the original element.

Therefore, modifying an element through the resulting view also modifies the original container.

### **Value Return Types Versus Reference Return Types**

The callable's return type plays an important role in the behavior of the view.

If the callable returns a value, the output element represents the result of the computation.

```
auto result = std::views::zip_transform(
    [](int& value) {
        return value * 2;
    },
    values
);
```

However, if the callable returns a reference, the view can provide direct access to the original data.

```
auto result = std::views::zip_transform(
    [](int& value) -> int& {
        return value;
    },
    values
);
```

This distinction is important for memory management, performance, and the ability to modify data.

### **Using const References**

If we do not intend to modify the data, it is generally preferable to use const references in the callable.

```
auto result = std::views::zip_transform(
    [](const int& a, const int& b) {
        return a + b;
    },
    first,
    second
);
```

This approach is suitable for read-only computations.

It also avoids unnecessary copies when working with large elements.

### **Using Proxy References**

In C++, not all containers necessarily provide ordinary references to their elements.

For example, `std::vector<bool>` has a specialized implementation in which element access is performed through proxy references.

Therefore, we should not always assume that the reference type of a range is exactly `T&`.

To write more generic callables, we can use `auto`:

```
auto result = std::views::zip_transform(
    [](auto&& value) {
        return !static_cast<bool>(value);
    },
    flags
);
```

This approach is useful for supporting different reference types.

However, using `auto&&` alone does not solve every issue associated with proxy references. The function's logic must still be compatible with the operations and conversions supported by the input type.

## **Advanced Capabilities and Practical Techniques**

### **Using Three or More Ranges**

One of the main advantages of `zip_transform` is its ability to accept an arbitrary number of ranges.

For example, suppose we want to calculate the areas of several rectangles.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> widths{2.0, 3.0, 4.0};
    std::vector<double> heights{5.0, 6.0, 7.0};

    std::vector<double> scales{1.0, 2.0, 0.5};

    auto areas = std::views::zip_transform(
        [](double width, double height, double scale) {
            return width * height * scale;
        },
        widths,
        heights,
        scales
    );

    for (double area : areas) {
        std::cout << area << ' ';
    }
}
```

Output:

```
10 36 14
```

In this example, each output element is calculated using three corresponding values.

This capability is useful in:

- Geometric calculations.
- Multidimensional data processing.
- Financial calculations.
- Signal processing.
- Matrix computations.
- Combining multiple features from a dataset.

### **Combining with std::views::filter**

`zip_transform` can be combined with other Range Adaptors.

For example, suppose we want to calculate the sums of corresponding elements from two ranges and then retain only the results greater than a specified value.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 10, 20, 30, 40};
    std::vector<int> b{2, 5, 10, 15, 20};

    auto result =
        std::views::zip_transform(
            [](int x, int y) {
                return x + y;
            },
            a,
            b
        )
        | std::views::filter(
            [](int value) {
                return value >= 30;
            }
        );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

Output:

```
30 45 60
```

In this example, the corresponding elements are added first, and the filter condition is then applied to each result.

An important detail is that both operations are lazy. Creating the views does not process all elements immediately.

### **Combining with std::views::take**

We can use `std::views::take` to limit the number of elements processed.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4, 5, 6};
    std::vector<int> b{10, 20, 30, 40, 50, 60};

    auto result =
        std::views::zip_transform(
            std::plus<>{},
            a,
            b
        )
        | std::views::take(3);

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

Output:

```
11 22 33
```

This approach is useful when we only need the initial portion of the data.

### **Combining with std::views::drop**

If we want to skip some initial elements, we can use `std::views::drop`.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4, 5};
    std::vector<int> b{10, 20, 30, 40, 50};

    auto result =
        std::views::zip_transform(
            std::plus<>{},
            a,
            b
        )
        | std::views::drop(2);

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

Output:

```
33 44 55
```

In this example, the first two results are skipped.

### **Combining with std::views::enumerate**

The `std::views::enumerate` facility was also introduced in C++23.

It provides a way to iterate over elements together with their indices.

We can use it to process the results of `zip_transform` while also accessing each element's position.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{10, 20, 30};
    std::vector<int> b{1, 2, 3};

    auto transformed = std::views::zip_transform(
        std::minus<>{},
        a,
        b
    );

    for (auto [index, value] :
         std::views::enumerate(transformed)) {
        std::cout << index << ": " << value << '\n';
    }
}
```

Output:

```
0: 9
1: 18
2: 27
```

This combination is useful for:

- Processing indexed data.
- Generating reports.
- Identifying invalid elements and their positions.
- Algorithms that require both an element's value and its position.

### **Converting the Result into a Container**

Since `zip_transform` produces a view, we must convert it into a container if we need to store the results permanently.

In C++23, we can use `std::ranges::to`.

```
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );

    auto output = result
        | std::ranges::to<std::vector>();
}
```

In this example, the view is converted into a `std::vector`.

The difference between the following two approaches is important:

```
auto view = std::views::zip_transform(
    std::plus<>{},
    a,
    b
);
```

And:

```
auto container = view
    | std::ranges::to<std::vector>();
```

In the first case, the results are provided lazily. In the second case, the results are stored in memory.

## **Practical Applications in Professional Programming**

### **Calculating the Sum of Two Data Sets**

One of the primary applications of `zip_transform` is performing element-wise computations.

For example, in a data processing system, we can add two collections of corresponding values.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> actual{10.5, 20.0, 30.5};
    std::vector<double> adjustment{1.5, 2.0, 3.5};

    auto adjusted = std::views::zip_transform(
        std::plus<>{},
        actual,
        adjustment
    );

    for (double value : adjusted) {
        std::cout << value << ' ';
    }
}
```

Output:

```
12 22 34
```

This pattern is useful in statistical, financial, and scientific data processing.

### **Calculating the Difference Between Two Data Sets**

To compare two collections of data, we can calculate the differences between corresponding elements.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> expected{100, 200, 300};
    std::vector<double> actual{95, 210, 290};

    auto differences = std::views::zip_transform(
        std::minus<>{},
        actual,
        expected
    );

    for (double difference : differences) {
        std::cout << difference << ' ';
    }
}
```

Output:

```
-5 10 -10
```

This approach can be used to calculate prediction errors in data processing systems and prediction algorithms.

### **Calculating the Dot Product**

An important application of this facility is implementing part of vector computations.

The dot product of two vectors is the sum of the products of their corresponding elements:

\\[ A \cdot B = \sum\_{i=0}^{n-1} A_i B_i \\]

We can implement this operation using `zip_transform` and `std::ranges::fold_left`.

```
#include <functional>
#include <iostream>
#include <numeric>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4};
    std::vector<int> b{5, 6, 7, 8};

    auto products = std::views::zip_transform(
        std::multiplies<>{},
        a,
        b
    );

    int dotProduct = std::ranges::fold_left(
        products,
        0,
        std::plus<>{}
    );

    std::cout << dotProduct;
}
```

Output:

```
70
```

In this example:

- `zip_transform` produces the products of corresponding elements.
- `fold_left` combines all the results using addition.

This approach performs the operation lazily without creating an intermediate container for the products.

However, numerical applications that require high precision must account for overflow, floating-point accuracy, and the selected data types.

### **Calculating Euclidean Distance**

For two vectors, we can calculate the sum of the squared differences between corresponding elements.

The Euclidean distance formula is:

\\[ d(A,B) = \sqrt{ \sum\_{i=0}^{n-1} (A_i-B_i)^2 } \\]

Implementation:

```
#include <cmath>
#include <functional>
#include <iostream>
#include <numeric>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> a{1.0, 2.0, 3.0};
    std::vector<double> b{4.0, 6.0, 3.0};

    auto squaredDifferences =
        std::views::zip_transform(
            [](double x, double y) {
                const double difference = x - y;
                return difference * difference;
            },
            a,
            b
        );

    double sum = std::ranges::fold_left(
        squaredDifferences,
        0.0,
        std::plus<>{}
    );

    double distance = std::sqrt(sum);

    std::cout << distance;
}
```

Output:

```
5
```

This pattern is useful in machine learning algorithms, signal processing, and geometric computations.

### **Calculating Weighted Values**

Suppose we have a collection of values and their corresponding weights.

We can use `zip_transform` to calculate the product of each value and its corresponding weight.

```
#include <iostream>
#include <numeric>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> values{10, 20, 30};
    std::vector<double> weights{0.2, 0.3, 0.5};

    auto weightedValues =
        std::views::zip_transform(
            [](double value, double weight) {
                return value * weight;
            },
            values,
            weights
        );

    double result = std::ranges::fold_left(
        weightedValues,
        0.0,
        std::plus<>{}
    );

    std::cout << result;
}
```

Output:

```
23
```

This approach can be used in statistical calculations and scoring systems.

### **Processing Structured Data Simultaneously**

Suppose user information is stored in several separate containers.

```
std::vector<std::string> names;
std::vector<int> ages;
std::vector<double> scores;
```

With `zip_transform`, we can combine corresponding data without creating an intermediate container.

```
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> names{
        "Alice", "Bob", "Charlie"
    };

    std::vector<int> ages{
        25, 30, 35
    };

    std::vector<double> scores{
        85.5, 90.0, 78.5
    };

    auto descriptions =
        std::views::zip_transform(
            [](const std::string& name,
               int age,
               double score) {
                return name + " (" +
                    std::to_string(age) +
                    "): " +
                    std::to_string(score);
            },
            names,
            ages,
            scores
        );

    for (const auto& description : descriptions) {
        std::cout << description << '\n';
    }
}
```

Output:

```
Alice (25): 85.500000
Bob (30): 90.000000
Charlie (35): 78.500000
```

This pattern is useful when data is stored in separate containers for architectural or performance reasons.

However, if these values conceptually always form a single entity, using a container of structs may be a better design.

## **Comparison with Older Approaches**

### **Comparison with a Traditional for Loop**

Before `zip_transform` was introduced, the most common approach for processing multiple collections was to use manually written loops.

```
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    std::vector<int> result;

    result.reserve(a.size());

    for (std::size_t i = 0;
         i < a.size();
         ++i) {
        result.push_back(a[i] + b[i]);
    }
}
```

This approach is completely valid, but it differs from `zip_transform` in several ways.

| Feature                              | Manual Loop                 | zip_transform |
| ------------------------------------ | --------------------------- | ------------- |
| Manual index management              | Yes                         | No            |
| Creates an output container          | Usually                     | No            |
| Lazy evaluation                      | No                          | Yes           |
| Composition with views               | More limited                | Very suitable |
| Direct control over execution flow   | Greater                     | More limited  |
| Support for non-random-access ranges | Requires suitable iterators | Yes           |

Manual loops are more suitable when the processing logic involves complex execution-flow control, error handling, conditional termination, or multiple state-dependent stages.

By contrast, `zip_transform` is a good choice for direct and independent computations on corresponding elements.

### **Comparison with std::transform**

`std::transform` has been available in the standard library since earlier C++ versions and can transform elements from one or more ranges.

For example, its multi-input overload can perform an operation similar to the following:

```
#include <algorithm>
#include <functional>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    std::vector<int> result(a.size());

    std::transform(
        a.begin(),
        a.end(),
        b.begin(),
        result.begin(),
        std::plus<>{}
    );
}
```

This approach can be perfectly suitable in terms of performance.

However, an important difference is that `std::transform` writes its output into a destination range. Therefore, suitable storage must be provided for the results.

By contrast, `zip_transform` creates a lazy view and defers computation until its elements are accessed.

Consequently:

- For immediately producing results in a container, `std::transform` is a suitable option.
- For building pipelines based on Ranges, `zip_transform` is generally a better choice.
- For lazy processing and composition with other adaptors, `zip_transform` offers additional advantages.

### **Comparison with std::views::transform**

The `std::views::transform` facility accepts a single range and invokes a callable on each of its elements.

By contrast, `zip_transform` accepts multiple ranges.

Example using `transform`:

```
auto result = values | std::views::transform(
    [](int value) {
        return value * 2;
    }
);
```

Example using `zip_transform`:

```
auto result = std::views::zip_transform(
    std::multiplies<>{},
    values,
    multipliers
);
```

In summary:

- `transform` is suitable for transforming elements of a single range.
- `zip_transform` is suitable for simultaneously transforming elements from multiple ranges.

### **Comparison with Range-v3**

Before the Zip facilities were standardized in C++23, the Range-v3 library provided similar functionality.

This library was one of the major influences on the design of Ranges in the standard library.

With the introduction of `std::views::zip_transform`, developers can use this functionality without depending on an external library.

However, different versions of Range-v3 may differ from the standard implementation in API details, output types, or additional capabilities.

Therefore, we should not assume that every implementation detail or behavior is identical between the two libraries.

## **Performance Considerations and Time Complexity**

### **Computational Cost**

In the typical case, if \\(n\\) represents the number of elements in the output view, fully traversing it requires \\(n\\) invocations of the callable.

Therefore, the overall time complexity is:

\\[ O(n) \\]

This complexity accounts only for iteration and callable invocation.

If the function itself performs complex operations, its cost must also be included.

For example, if each callable invocation has a complexity of \\(O(k)\\), the overall complexity is approximately:

\\[ O(nk) \\]

### **Memory Cost**

Since `zip_transform` does not automatically store its results, it generally does not require additional memory proportional to the number of elements.

The view itself typically requires constant storage relative to the number of elements and storage proportional to the number of input ranges.

Conceptually:

\\[ O(m) \\]

where \\(m\\) is the number of input ranges.

This represents the memory required to maintain the view's state and associated iterators. It does not include the memory consumed by the original containers.

### **The Importance of Side Effects**

Because computation takes place during dereferencing, we should not assume that each element is calculated exactly once.

For example:

```
auto result = std::views::zip_transform(
    [](int value) {
        std::cout << "Called\n";
        return value * 2;
    },
    values
);

auto it = result.begin();

int first = *it;
int second = *it;
```

In this example, the `Called` message may be displayed twice.

This happens because repeated dereferencing does not necessarily reuse a previously calculated result.

Therefore, when designing a suitable callable, it is preferable to:

- Avoid modifying external state.
- Avoid unnecessary input/output operations.
- Keep computations deterministic whenever possible.
- Materialize the view into a container when result caching is required.

### **The Effect of Caching and Repeated Computations**

In some scenarios, repeatedly using a view may cause computations to be performed again.

For example:

```
auto result = std::views::zip_transform(
    expensiveCalculation,
    a,
    b
);
```

If several different algorithms operate on `result`, `expensiveCalculation` may be executed multiple times.

In such cases, if the computations are expensive, materializing the result may provide better performance.

```
auto cached = result
    | std::ranges::to<std::vector>();
```

After this operation, subsequent operations use the stored data instead of invoking the callable again.

## **Limitations and Important Considerations**

### **Lifetime of the Input Ranges**

One of the most important considerations when using views is lifetime management.

When a view is created from an existing container, it generally refers to the container's data instead of copying all its elements.

Therefore, the container must remain valid for as long as the view is used.

Dangerous example:

```
#include <ranges>
#include <vector>

auto createView() {
    std::vector<int> values{1, 2, 3};

    return std::views::zip_transform(
        [](int value) {
            return value * 2;
        },
        values
    );
}
```

In this example, the local container is destroyed when the function returns.

The returned view must no longer be used to access that container's elements.

A more appropriate example is:

```
#include <ranges>
#include <vector>

auto createView(std::vector<int>& values) {
    return std::views::zip_transform(
        [](int value) {
            return value * 2;
        },
        values
    );
}
```

In this case, responsibility for the lifetime of the original data belongs to the caller.

### **Using Temporary Ranges**

The standard Ranges library uses the `viewable_range` concept to manage temporary ranges.

For example, under certain conditions, passing a temporary container to a view is permitted. However, how its lifetime is managed depends on the range type and its ownership characteristics.

An important detail is that a resulting view does not necessarily copy the original data. It may preserve ownership of a temporary range through an owning view such as `owning_view`.

However, if a non-owning or reference-based range is provided, its lifetime remains important.

In complex applications, the ownership and lifetime of all input ranges should be designed explicitly.

### **Limitations Regarding Parallel Processing**

`zip_transform` does not guarantee that elements will be processed in parallel.

It is a view that provides lazy results and should not be confused with parallel algorithms.

If parallel processing is required, suitable facilities such as Parallel Algorithms or specialized libraries should be used.

Additionally, side effects in the callable can introduce Thread Safety problems.

### **No Guarantee That the Callable Runs Only Once**

As explained earlier, each dereference may invoke the callable again.

Therefore, using a callable with expensive operations or side effects can lead to behavior and performance that differ from initial expectations.

### **Input Range Restrictions**

Not every range is suitable for use with `zip_transform`.

Each range must satisfy the `input_range` requirements and be convertible into an appropriate view.

Additionally, if capabilities such as random access are required, all input ranges must have iterators with the necessary properties.

For example, combining a `std::vector` with an input-only range does not make the output a random-access range.

### **Return Type Restrictions**

The callable must produce a valid return type.

A `void` result is not permitted.

Additionally, if the output is a reference, the lifetime of the object it refers to must remain valid.

For example, returning a reference to a local variable is incorrect:

```
auto result = std::views::zip_transform(
    [](int value) -> const int& {
        int temporary = value * 2;
        return temporary;
    },
    values
);
```

This function returns a reference to a local variable that is destroyed when the Lambda finishes executing.

Consequently, accessing such a reference results in undefined behavior.

### **Const Considerations**

Whether a view can be used in a `const` context depends on the callable and the underlying views.

In general, using the `const` version requires:

- The underlying view to be a range when const-qualified.
- The callable to be invocable when const-qualified.
- The relevant `regular_invocable` requirements to be satisfied for the const version.

For example:

```
auto result = std::views::zip_transform(
    [](int value) {
        return value * 2;
    },
    values
);
```

This ordinary Lambda has a `const` call operator and, under suitable conditions, allows the view to be used in a const context.

However, callables with mutable internal state or types that can only be invoked in a non-const context may restrict this capability.

## **Special Behavior When Called Without a Range**

One lesser-known detail of the standard is that `std::views::zip_transform` can be invoked with only a callable.

For example:

```
auto empty = std::views::zip_transform(
    [] {
        return 42;
    }
);
```

In this case, instead of creating an ordinary `zip_transform_view`, an appropriate empty view is produced.

The requirements for this case are:

- The callable must be `move_constructible`.
- The callable must be `regular_invocable`.
- Its return type must be a valid Object Type.

Since no input ranges are provided, the resulting view contains no elements.

This behavior is part of the standard specification, although it has limited applications in everyday programming.

## **Comparison with Related C++23 Facilities**

The `zip_transform` facility is part of the broader set of new Ranges capabilities introduced in C++23.

Understanding related facilities helps us select the appropriate tool for each scenario.

| Facility                         | Purpose                                                           |
| -------------------------------- | ----------------------------------------------------------------- |
| `std::views::zip`                | Combines corresponding elements from multiple ranges              |
| `std::views::zip_transform`      | Invokes a callable on corresponding elements from multiple ranges |
| `std::views::adjacent`           | Produces groups of adjacent elements from a single range          |
| `std::views::adjacent_transform` | Invokes a callable on groups of adjacent elements                 |
| `std::views::transform`          | Transforms elements of a single range                             |
| `std::views::filter`             | Filters elements of a range                                       |
| `std::views::enumerate`          | Iterates over elements together with their indices                |

For example, if the goal is to calculate the average of two adjacent elements in a range, `adjacent_transform` is a more appropriate tool.

However, if the goal is to add elements from two independent ranges, `zip_transform` is the natural choice.

## **Compiler Support and Compatibility**

### **Standardization Status**

`std::views::zip_transform` is part of the C++23 standard library.

It was introduced through the following standardization proposal:

`P2321R2 - zip`

To use it, the compiler and its associated standard library must provide a suitable implementation of the C++23 Ranges facilities.

### **Checking the Feature Test Macro**

The following Feature Test Macro can be used to check library support:

```
__cpp_lib_ranges_zip
```

Example:

```
#include <iostream>
#include <ranges>

int main() {
#ifdef __cpp_lib_ranges_zip
    std::cout << "Zip ranges are supported\n";
#else
    std::cout << "Zip ranges are not supported\n";
#endif
}
```

The Feature Test Macro value introduced for this set of facilities is:

```
202110L
```

A more precise check can therefore be performed:

```
#include <ranges>

#if defined(__cpp_lib_ranges_zip) && \
    __cpp_lib_ranges_zip >= 202110L

// Zip-related C++23 facilities are available.

#else

#error "The standard library does not provide the required zip facilities."

#endif
```

An important detail is that this Feature Test Macro belongs to the standard library, not the language itself.

Consequently, simply enabling `-std=c++23` or `/std:c++23` does not guarantee that all `zip_transform` facilities have been implemented in the standard library version being used.

### **Configuring the Standard in GCC and Clang**

To use C++23 with GCC, the following command can be used:

```
g++ -std=c++23 -O2 -Wall -Wextra main.cpp -o main
```

With Clang, the following command can be used:

```
clang++ -std=c++23 -O2 -Wall -Wextra main.cpp -o main
```

Actual support depends on the compiler version and the associated standard library implementation.

### **Configuring the Standard in MSVC**

With Microsoft Visual C++, the following option can be used:

```
cl /std:c++latest /EHsc main.cpp
```

Alternatively, on versions that provide the required standard-mode option, the appropriate C++23 setting can be selected.

It is important to remember that MSVC language support and STL support are related but separate matters.

Therefore, in addition to selecting the language standard, it is advisable to check the Feature Test Macro.

## **Common Mistakes When Using zip_transform**

### **Assuming the Result Is a Container**

A common mistake is treating the output of `zip_transform` as equivalent to a `std::vector`.

```
auto result = std::views::zip_transform(
    std::plus<>{},
    a,
    b
);
```

In this example, `result` is a view, not a container.

Therefore, if the results need to be stored permanently, we must explicitly convert the view into a container.

### **Assuming Input Ranges Have Equal Sizes**

Some developers assume that all input ranges must have identical sizes.

This assumption is incorrect.

If the input ranges have different sizes, the resulting view contains as many elements as the shortest range.

If equal sizes are required by the program's logic, they must be checked separately.

### **Using a Callable with Side Effects**

Since the function may be invoked multiple times for the same element, using callables that modify external state can cause problems.

It is preferable to keep callables used with this view as Pure as possible.

### **Returning an Invalid Reference**

If the callable returns a reference to a local variable or an object with a short lifetime, accessing the output can result in undefined behavior.

When necessary, it is better to return a value or provide a reference to data with a sufficiently long lifetime.

### **Expecting Parallel Execution**

`zip_transform` performs lazy evaluation, but this does not mean that it processes elements in parallel.

Parallel execution requires other facilities.

### **Forgetting the Lifetime of Input Data**

Views based on external containers generally do not own the data in those containers.

If a container is destroyed, the view that depends on it may become invalid.

This is particularly important when returning views from functions or storing them as class members.

### **Using std::views::transform Instead of zip_transform**

Developers sometimes first create a `zip` view and then apply `transform` to its tuple-like elements.

This approach is valid, but when the goal is simply to pass corresponding elements to a function, `zip_transform` is generally simpler and more direct.

## **Design Considerations and Choosing the Right Tool**

The choice of `zip_transform` should depend on the actual requirements of the application.

It is a suitable choice when:

- We have multiple independent ranges.
- Elements at corresponding positions must be processed together.
- Each output element is a computed value.
- We do not need to store all results immediately.
- We want to build pipelines using Range Adaptors.
- The transformation logic can be expressed as an independent callable.

However, another tool may be more appropriate in the following situations:

- We need precise control over execution flow.
- Each element depends on complex shared state.
- Essential side effects are part of the computation.
- We need to calculate and store each result exactly once.
- Parallel processing is required.
- Different input sizes must produce an error rather than terminate iteration early.

In such situations, we can use traditional loops, `std::transform`, parallel algorithms, or a combined pipeline with materialization.

## **Final Summary**

The `std::views::zip_transform` facility is a useful addition to the Ranges library in C++23 that simplifies the simultaneous processing of multiple ranges.

By accepting a callable and multiple ranges, it produces a lazy view in which each element is the result of invoking the callable on the corresponding elements of the input ranges.

Its most important characteristics and considerations include:

- **Lazy Evaluation:** Results are calculated when elements are accessed.
- **No Intermediate Container Required:** Results are not automatically stored in memory.
- **Multiple Range Support:** An arbitrary number of ranges can be provided for simultaneous processing.
- **Reference Support:** Original elements can be modified when a valid reference is returned.
- **Composability:** It can be combined with facilities such as `filter`, `take`, `drop`, `enumerate`, and `fold_left`.
- **Shortest-Range Termination:** The output contains as many elements as the shortest input range.
- **Dependence on Input Iterators:** Capabilities such as random access and bidirectional iteration depend on the properties of the underlying ranges.
- **No Guarantee of Single Execution:** Each dereference may invoke the callable again.
- **Lifetime Management:** The original data must remain valid while the view is being used.

Overall, `std::views::zip_transform` is an excellent choice for building data-processing pipelines and performing element-wise computations in modern C++ applications.

When used correctly, it can eliminate many repetitive loops and intermediate containers, allowing data-processing logic to be expressed in a declarative, readable, and composable manner.

## **References and Further Reading**

- [cppreference - std::ranges::views::zip_transform](https://en.cppreference.com/w/cpp/ranges/zip_transform_view?utm_source=chatgpt.com)
- [WG21 P2321R2 - zip, zip_transform, adjacent and adjacent_transform](https://www.open-std.org/JTC1/SC22/WG21/docs/papers/2021/p2321r2.html?utm_source=chatgpt.com)
- [C++23 Working Draft - Zip Transform View](https://timsong-cpp.github.io/cppwp/std23/ranges?utm_source=chatgpt.com)
- [cppreference - C++23 Compiler Support](https://en.cppreference.com/w/cpp/compiler_support/23?utm_source=chatgpt.com)
- [cppreference - std::ranges::views::zip](https://en.cppreference.com/w/cpp/ranges/zip_view?utm_source=chatgpt.com)
- [cppreference - std::ranges::views::transform](https://en.cppreference.com/w/cpp/ranges/transform_view?utm_source=chatgpt.com)
- [cppreference - std::ranges::fold_left](https://en.cppreference.com/w/cpp/algorithm/ranges/fold_left?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>