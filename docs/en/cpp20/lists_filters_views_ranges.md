<div align="center">

[🇺🇸 English](./lists_filters_views_ranges.md) | [🇮🇷 فارسی](../../fa/cpp20/lists_filters_views_ranges.md)

</div>

---

# Introduction to Lists, Filters, Views, and Ranges in C++20

# C++20 Ranges Library

## Introduction

The Ranges library is one of the major features introduced in C++20. It makes working with collections of data in C++ simpler, more readable, and more flexible.

Before Ranges was introduced, many standard algorithms required two Iterators to specify a range: one Iterator marking the beginning and another marking the end. This approach is still valid, but in many cases it leads to repetitive code and increases the possibility of mistakes.

The Ranges library allows us to pass the Range or Container itself to many algorithms instead of explicitly providing two Iterators. In addition, it introduces features such as Views, Range Adaptors, Projections, and Function Composition.

To understand this library, we should first become familiar with the concepts of a Range and its relationship with Iterators.

## The Concept of a Range

In C++, a Range is a sequence of elements that can be traversed through Iterators.

For example, a `std::vector`, a `std::array`, a `std::list`, and even a subrange of a regular array can be used as Ranges.

```cpp
#include <iostream>
#include <vector>
#include <ranges>

int main()
{
    std::vector<int> numbers{10, 20, 30, 40, 50};

    for (int number : numbers)
    {
        std::cout << number << ' ';
    }
}
```

In the example above, `numbers` is a Range. The Range-based for loop traverses it from beginning to end and provides access to its elements.

An important point is that a Range is not necessarily a Container. A Range can simply be a traversable sequence that obtains its data from another source or even generates elements during traversal.

For example, a View can be a Range without owning the underlying data.

### The Relationship Between Ranges and Iterators

Iterators are the tools used to traverse elements. Depending on the capabilities they provide, different Iterator categories support different operations.

In C++20, the Iterator categories include:

- **Input Iterator:** Provides the ability to read elements while traversing in one direction.
- **Output Iterator:** Provides the ability to write elements.
- **Forward Iterator:** Supports forward traversal and repeated passes.
- **Bidirectional Iterator:** Supports both forward and backward traversal.
- **Random Access Iterator:** Supports random access and jumping to different positions.
- **Contiguous Iterator:** In addition to random-access capabilities, guarantees that elements are stored contiguously in memory.

These categories are also important when designing Ranges. A Range can provide different traversal capabilities. For example, one Range may support only forward traversal, while another may also allow random access to its elements.

### The Concept of a Sentinel

In earlier algorithm designs, the end of a range was usually represented by a specific Iterator. Ranges also introduce the concept of a Sentinel.

A Sentinel is an object that can indicate the end of traversal. Unlike an Iterator, it does not need to provide all the capabilities of an Iterator.

As a result, a Range can use two different types to represent its boundaries:

- An Iterator to mark the beginning of traversal.
- A Sentinel to determine when traversal has reached the end.

This flexibility is particularly useful for Ranges whose length is not known in advance or whose elements are generated incrementally.

## Range Algorithms

The C++ standard library provided algorithms even before C++20. These algorithms typically used two Iterators to specify the target range.

For example, `std::all_of()` checks whether all elements in a range satisfy a specified condition.

### The Traditional Approach to Using Algorithms

```cpp
#include <algorithm>
#include <iostream>

int main()
{
    int collection[]{2, 6, 8, 40, 64, 70};

    bool result = std::all_of(
        std::begin(collection),
        std::end(collection),
        [](int value)
        {
            return value % 2 == 0;
        }
    );

    std::cout << std::boolalpha << result << '\n';
}
```

With this approach, we must pass both the beginning and ending Iterators of the range to the algorithm.

### Using Range Algorithms

C++20 introduced new versions of many standard algorithms in the `std::ranges` namespace.

In many cases, these versions accept a Range directly.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> numbers{11, 2, 6, 4, 8, 3, 17, 9};

    auto odd = [](int value)
    {
        return value % 2 != 0;
    };

    bool result = std::ranges::all_of(numbers, odd);

    if (result)
    {
        std::cout << "All elements are odd\n";
    }
    else
    {
        std::cout << "Not all elements are odd\n";
    }
}
```

In this example, there is no need to pass two Iterators. The Container itself is passed to the algorithm.

However, the Ranges versions of many algorithms still support Iterator and Sentinel arguments. Passing a Range is not the only valid way to call them.

### The for_each Algorithm

The `std::ranges::for_each()` algorithm invokes a function for every element in a range.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    std::ranges::for_each(numbers, [](int& value)
    {
        value *= 2;
    });

    for (int value : numbers)
    {
        std::cout << value << ' ';
    }
}
```

Output:

```text
2 4 6 8 10
```

Because the Lambda parameter is a Reference, the changes are applied directly to the original Container elements.

If the parameter were passed by value, the changes would affect only a copy of each element.

### The sort Algorithm

The `std::ranges::sort()` algorithm sorts the elements of a sortable range.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> numbers{8, 3, 9, 1, 5, 2};

    std::ranges::sort(numbers);

    for (int value : numbers)
    {
        std::cout << value << ' ';
    }
}
```

Output:

```text
1 2 3 5 8 9
```

By default, sorting is performed in ascending order.

To sort in descending order, we can provide an appropriate Comparator:

```cpp
std::ranges::sort(numbers, std::greater<>{});
```

Keep in mind that `std::ranges::sort()` cannot be used with every Range. It requires a range that supports the capabilities needed for sorting, including random access and the ability to rearrange its elements.

### The find Algorithm

The `std::ranges::find()` algorithm searches a Range for a specified element.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> numbers{11, 2, 6, 4, 8, 3, 17, 9};

    auto position = std::ranges::find(numbers, 17);

    if (position != std::ranges::end(numbers))
    {
        std::cout << "Found: " << *position << '\n';
    }
    else
    {
        std::cout << "Not found\n";
    }
}
```

If the element is found, the algorithm returns an Iterator pointing to it. Otherwise, the returned value is equal to the end of the Range.

### The copy Algorithm and Output Iterators

Range algorithms are not limited to Containers. They can also be used to transfer elements to destinations such as Output Iterators.

For example:

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    std::ranges::copy(
        numbers,
        std::ostream_iterator<int>(std::cout, " ")
    );
}
```

In this example, the elements of the Vector are sent directly to standard output.

### Benefits of Range Algorithms

The main benefits of Range algorithms include:

- Reducing the need to pass beginning and ending Iterators in common algorithm calls.
- Improving code readability.
- Using Constraints to restrict the types of valid inputs.
- Supporting Projections.
- Providing better interoperability with Views and Range Adaptors.
- Reducing the possibility of mistakes when specifying ranges.

## Iterator-Based Algorithms and Constraints

Traditional C++ standard algorithms are generally designed around the requirements of Iterators. For example, one algorithm may require only an Input Iterator, while another may require a Random Access Iterator.

The Ranges versions support not only Ranges but also the Constraints introduced in C++20.

Constraints specify the capabilities a type must provide to be used with a particular algorithm.

For example, `std::ranges::sort()` requires a Range that satisfies the requirements for sorting. If a type does not meet these requirements, the compiler can diagnose the invalid call.

### The Difference Between Iterator Pair and Range Overloads

Many Ranges algorithms provide two forms of invocation:

```cpp
std::ranges::sort(numbers);
```

And:

```cpp
std::ranges::sort(numbers.begin(), numbers.end());
```

The first form accepts a Range, while the second accepts an Iterator and a Sentinel.

Therefore, Ranges do not completely replace Iterators. Instead, they provide a more flexible layer for working with ranges.

### Why Should We Usually Prefer std::ranges?

In modern C++ code, using the `std::ranges` versions is generally a good choice when the algorithm's requirements are compatible with the data type.

These versions support Ranges, Projections, and Constraints.

However, traditional algorithms remain valid. In existing projects, legacy libraries, or specific circumstances, using them may still be reasonable.

## Projections

One of the important features of Range algorithms is their support for Projections.

A Projection is a callable object that extracts a particular part of an element or provides another representation of it before the element is compared or examined.

In simple terms, instead of having an algorithm operate directly on an entire object, we can specify which property or part of that object should be used as the basis for the operation.

Suppose we have a collection of Point objects and want to sort them based on one of their coordinates.

### Defining the Point Class

```cpp
#include <cmath>
#include <iostream>

class Point
{
public:
    Point() = default;

    Point(double x, double y)
        : m_x(x), m_y(y)
    {
    }

    double length() const
    {
        return std::sqrt(m_x * m_x + m_y * m_y);
    }

    double m_x{};
    double m_y{};
};
```

### Sorting Without a Projection

If the comparison operator is defined for the Point class, we can use it directly for sorting.

```cpp
std::ranges::sort(points);
```

In this case, the algorithm uses its default Comparator, and the objects are compared according to the comparison operator defined for them.

### Using a Lambda as a Projection

Suppose we want to sort the points based on their `m_x` coordinate.

```cpp
std::ranges::sort(
    points,
    std::less<>{},
    [](const Point& point)
    {
        return point.m_x;
    }
);
```

This invocation has three parts:

- The target Range.
- A Comparator used to compare values.
- A Projection used to extract the value to be compared from each element.

The algorithm first applies the Projection to the elements and then compares the resulting values using the Comparator.

In this example, instead of comparing two Point objects, their `m_x` values are compared.

### Using a Pointer to Member as a Projection

If the required Projection simply accesses a public member, we can use a Pointer to Member instead of a Lambda.

```cpp
std::ranges::sort(
    points,
    std::less<>{},
    &Point::m_y
);
```

This code sorts the points based on their `m_y` values.

This approach is concise and readable, and it is a good choice when we only need to extract a public member.

### Projections in the for_each Algorithm

Projections are not limited to sorting. Algorithms such as `std::ranges::for_each()` also support them.

```cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <utility>
#include <vector>

int main()
{
    using Pair = std::pair<int, std::string>;

    std::vector<Pair> pairs{
        {1, "one"},
        {2, "two"},
        {3, "three"}
    };

    auto print = [](const auto& value)
    {
        std::cout << value << ' ';
    };

    std::ranges::for_each(
        pairs,
        print,
        &Pair::first
    );

    std::cout << '\n';

    std::ranges::for_each(
        pairs,
        print,
        &Pair::second
    );
}
```

In the first invocation, the Projection selects the `first` member. In the second, it selects the `second` member.

This allows the `print` function to receive only the extracted value instead of the entire Pair.

### The Difference Between Projection and Transform

Projection and Transform may appear similar, but they do not serve the same purpose.

A Projection is part of an algorithm's interface. It specifies which value the algorithm should consider when comparing or examining elements.

In contrast, `transform_view` creates a new View whose elements are obtained by applying a function to the elements of the original Range.

A Projection is generally used to define how an algorithm interprets its input data, whereas Transform represents a processing stage applied to a Range.

## Views and Range Adaptors

One of the most important parts of the Ranges library is the concept of a View.

A View is a type of Range that generally does not own the underlying data. Instead of storing an independent collection of data, it provides a particular representation or access pattern for existing data.

We can think of a View as a window placed over the original data, showing only a specific portion or representation of it.

Important characteristics of Views include:

- They generally do not own the underlying data.
- In many cases, they operate lazily.
- They are usually inexpensive to construct and copy.
- They can be passed to functions as parameters.
- They allow multiple Views to be combined into a data-processing pipeline.

### The Difference Between a View and a Container

A Container is generally responsible for storing and owning its elements.

For example, `std::vector` manages the memory required for its elements.

A View, on the other hand, generally provides only a way to access or observe the original data without storing that data itself.

```cpp
std::vector<int> numbers{1, 2, 3, 4, 5};

auto view = numbers | std::views::filter(
    [](int value)
    {
        return value % 2 == 0;
    }
);
```

In this example, a View is created that contains only the even elements. The original elements remain in the Vector, and the View simply exposes the elements that satisfy the condition.

### An Important Note About Ownership

The fact that a View is generally non-owning means that the lifetime of the underlying data matters.

```cpp
auto create_view()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    return numbers | std::views::filter(
        [](int value)
        {
            return value % 2 == 0;
        }
    );
}
```

This function demonstrates a potentially dangerous design. The resulting View depends on data that is destroyed when the function returns. Using the View afterward can result in undefined behavior.

Therefore, when storing a View, we must ensure that the underlying data remains valid for as long as the View is used.

### Lazy Evaluation

One of the fundamental characteristics of many Views is Lazy Evaluation.

This means that constructing a View does not necessarily execute all operations on its elements. Operations are generally performed when the View's elements are traversed or accessed.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    auto view = numbers | std::views::transform(
        [](int value)
        {
            std::cout << "Transforming " << value << '\n';
            return value * 10;
        }
    );

    std::cout << "View created\n";

    for (int value : view)
    {
        std::cout << value << '\n';
    }
}
```

Creating the View does not execute the Transform function for every element. The function is called as the View is traversed.

This behavior can help avoid creating temporary collections and performing unnecessary computations.

However, Lazy Evaluation does not mean that results are cached. In many Views, if an element is accessed multiple times, the transformation function may be executed multiple times.

## Types of Views and Range Adaptors

The C++20 standard library provides a collection of Views and Range Adaptors.

A Range Adaptor is a tool that accepts a Range and creates a new View based on it.

The following sections examine some of the most important ones.

### Filter View

`std::views::filter` exposes only the elements that satisfy a specified condition.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5, 6, 7, 8, 9};

    auto evens = numbers | std::views::filter(
        [](int value)
        {
            return value % 2 == 0;
        }
    );

    for (int value : evens)
    {
        std::cout << value << ' ';
    }
}
```

Output:

```text
2 4 6 8
```

In this example, the View exposes only even numbers, and no new elements are created in the original Vector.

The condition can also be defined separately:

```cpp
auto is_even = [](int value)
{
    return value % 2 == 0;
};

auto evens = numbers | std::views::filter(is_even);
```

### Filter View and Predicate Evaluation

The Predicate is invoked during element traversal to determine whether an element satisfies the condition.

Therefore, constructing the View does not mean that all elements are immediately filtered.

The Predicate used by `filter_view` must also be compatible with the View's expected behavior. In particular, it should not produce unpredictable results that depend on changing external state.

Modifying the underlying elements in a way that changes the Predicate's result can also violate the requirements for using a Filter View.

### Transform View

`std::views::transform` applies a function to each element and exposes the result of that function as an element of the View.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    auto transformed = numbers | std::views::transform(
        [](int value)
        {
            return value * 10;
        }
    );

    for (int value : transformed)
    {
        std::cout << value << ' ';
    }
}
```

Output:

```text
10 20 30 40 50
```

In this example, the original data is not modified. The View exposes the transformed values as it is traversed.

However, if the Transform function returns a Reference, it may also be possible to modify the original data through the View. Therefore, its behavior depends on the return type of the function.

### Take View

`std::views::take` exposes at most a specified number of elements from the beginning of a Range.

```cpp
std::vector<int> numbers{1, 2, 3, 4, 5, 6, 7};

auto first_five = numbers | std::views::take(5);
```

This View exposes only the first five elements.

If the Range contains fewer elements than the requested number, all available elements are exposed.

### Take While View

`std::views::take_while` exposes elements from the beginning of a Range as long as a Predicate remains satisfied.

```cpp
std::vector<int> numbers{1, 3, 5, 7, 8, 9, 11};

auto result = numbers | std::views::take_while(
    [](int value)
    {
        return value % 2 != 0;
    }
);
```

Output:

```text
1 3 5 7
```

As soon as the element `8` fails the condition, traversal of this View stops. Elements after it are not exposed, even if they satisfy the condition again.

This behavior differs from Filter. Filter examines all elements and exposes every element that satisfies the condition, whereas Take While considers only a prefix of the Range.

### Drop View

`std::views::drop` skips a specified number of elements at the beginning of a Range and exposes the remaining elements.

```cpp
std::vector<int> numbers{1, 2, 3, 4, 5, 6, 7};

auto result = numbers | std::views::drop(3);
```

Output:

```text
4 5 6 7
```

### Drop While View

`std::views::drop_while` skips the initial elements as long as a Predicate is satisfied.

```cpp
std::vector<int> numbers{1, 3, 5, 8, 9, 11};

auto result = numbers | std::views::drop_while(
    [](int value)
    {
        return value % 2 != 0;
    }
);
```

Output:

```text
8 9 11
```

The difference from Filter is that Drop While skips only the initial elements. Once it encounters the first element that does not satisfy the condition, it exposes all remaining elements without checking the condition again.

### Keys View and Values View

For Ranges such as Maps or collections of Pairs, we sometimes need only the keys or only the values.

`std::views::keys` creates a View of the keys in a Range.

`std::views::values` provides a View of the values.

```cpp
#include <iostream>
#include <map>
#include <ranges>
#include <string>

int main()
{
    std::map<int, std::string> students{
        {1, "Ali"},
        {2, "Sara"},
        {3, "Reza"}
    };

    auto keys = students | std::views::keys;
    auto values = students | std::views::values;

    for (const auto& key : keys)
    {
        std::cout << key << ' ';
    }

    std::cout << '\n';

    for (const auto& value : values)
    {
        std::cout << value << ' ';
    }
}
```

These Views do not create separate copies of the keys or values. Instead, they provide access to the relevant parts of the original elements.

## Composing Views and the Pipe Operator

One of the most attractive features of the Ranges library is the ability to combine multiple Views.

Suppose we have a collection of numbers and want to:

- Select only the even numbers.
- Calculate the square of each selected number.
- Finally, expose only the first five results.

We could define these steps separately, but the Pipe operator allows us to express them as a readable chain.

### Composing Views

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector<int> numbers{
        1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12
    };

    auto result = numbers
        | std::views::filter(
            [](int value)
            {
                return value % 2 == 0;
            }
        )
        | std::views::transform(
            [](int value)
            {
                return value * value;
            }
        )
        | std::views::take(5);

    for (int value : result)
    {
        std::cout << value << ' ';
    }
}
```

Output:

```text
4 16 36 64 100
```

In this chain:

1. Filter selects only the even numbers.
2. Transform calculates the square of each number.
3. Take exposes only the first five results.

An important point is that the processing stages in this example are evaluated lazily. We do not need to create a new Vector containing the even numbers, then another Vector containing their squares, and finally a separate collection containing the first five elements.

### Understanding the Pipe Operator

The `|` operator in the Ranges library connects a Range to a Range Adaptor.

Conceptually:

```cpp
range | adaptor
```

This means that the Range is passed to the Adaptor to create a new View.

Multiple Adaptors can be combined in the same way:

```cpp
range
    | adaptor1
    | adaptor2
    | adaptor3;
```

This structure is readable as a data-processing pipeline: each stage receives the output of the previous stage.

### Composing Adaptors Without a Range

Sometimes, we can compose Adaptors with one another in advance and then apply the resulting composition to a Range.

```cpp
auto pipeline =
    std::views::filter(
        [](int value)
        {
            return value % 2 == 0;
        }
    )
    | std::views::transform(
        [](int value)
        {
            return value * value;
        }
    );

std::vector<int> numbers{1, 2, 3, 4, 5, 6};

auto result = numbers | pipeline;
```

In this example, a chain of Adaptors is created first and applied to a Range later.

This capability is useful when building reusable Pipelines.

### The Difference Between Composition and Immediate Execution

Composing Views does not necessarily mean that all stages are executed immediately.

In many cases, only the processing structure is constructed, and computations are performed as the elements are traversed.

For this reason, a View can be traversed multiple times. However, keep in mind that the result of each traversal may depend on the state of the underlying data and the behavior of the functions being used.

If a Transform function has Side Effects, traversing the View again may execute those Side Effects again.

## Range Factories

So far, we have examined Views created from existing Ranges.

However, C++20 provides tools that can generate Ranges without requiring an initial Container.

These tools are known as Range Factories.

One of the most important examples is `std::views::iota`.

### Iota View

`std::views::iota` generates a sequence of values.

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto numbers = std::views::iota(1, 10);

    for (int value : numbers)
    {
        std::cout << value << ' ';
    }
}
```

Output:

```text
1 2 3 4 5 6 7 8 9
```

The starting value is included in the Range, but the ending value is excluded.

Therefore, `iota(1, 10)` generates the numbers from 1 through 9.

### A Sequence Without a Specified End

If we provide only the starting value to `iota`, it generates an unbounded sequence.

```cpp
auto numbers = std::views::iota(1);
```

This Range continues indefinitely.

To use it, we must define an end condition or impose a limit.

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto numbers = std::views::iota(1)
        | std::views::take(10);

    for (int value : numbers)
    {
        std::cout << value << ' ';
    }
}
```

Output:

```text
1 2 3 4 5 6 7 8 9 10
```

### Using Iota in a Pipeline

Since Iota produces a Range, it can be combined with other Adaptors.

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto result = std::views::iota(1, 20)
        | std::views::filter(
            [](int value)
            {
                return value % 3 == 0;
            }
        )
        | std::views::transform(
            [](int value)
            {
                return value * value;
            }
        );

    for (int value : result)
    {
        std::cout << value << ' ';
    }
}
```

In this example, the numbers from 1 through 19 are generated first. The multiples of 3 are then selected, and their squares are exposed.

### Applications of Iota

`iota` is useful in several situations:

- Generating sequences of numbers.
- Creating numeric ranges for loops.
- Generating test data.
- Building data-processing Pipelines.
- Replacing some counter-based loops.
- Combining with Filter and Transform.

Keep in mind that using Iota to generate an unbounded sequence requires proper management of the traversal's end condition. If such a Range is traversed in a loop without a limit, the loop will never terminate.

## Additional Considerations and Limitations of Ranges

The Ranges library provides powerful capabilities, but using it correctly requires understanding several important points.

### Not All Ranges Can Be Sorted

Not every Range supports sorting.

For example, a View that supports only forward traversal does not necessarily satisfy the requirements of `std::ranges::sort`.

Some Views may also expose References or Proxy References that are incompatible with the requirements of a particular algorithm.

Therefore, being traversable does not mean that a Range can be used with every algorithm.

### Views Do Not Necessarily Own Their Data

A View generally depends on the underlying data. If the original data is destroyed, the View may become invalid as well.

When designing APIs that accept or return Views, the lifetime of the underlying data must be managed carefully.

### Modifying the Original Data Can Change the View's Result

Because a View generally represents the original data, modifying the elements of the Container can affect the result of traversing the View.

For example, if an element's value changes in a Filter View, that element may no longer satisfy the Predicate.

In such situations, the Iterator validity requirements and the rules associated with each View type must be considered.

### Lazy Evaluation Is Not Always Better

Lazy Evaluation can prevent the creation of temporary collections, but it is not always the fastest choice.

In some situations, recomputing a Transform during multiple traversals can be expensive.

If the result of a costly computation is going to be used multiple times, materializing the result in a Container may sometimes be a better choice.

### The Difference Between a View and a Materialized Result

Suppose we want to calculate the squares of the numbers in a Vector.

Using a Transform View:

```cpp
auto squares = numbers
    | std::views::transform(
        [](int value)
        {
            return value * value;
        }
    );
```

In this case, the result remains a View, and the calculations are performed during traversal.

However, if we want to store the results in an independent Vector:

```cpp
std::vector<int> squares;

std::ranges::transform(
    numbers,
    std::back_inserter(squares),
    [](int value)
    {
        return value * value;
    }
);
```

In this case, the results are calculated and stored in a new Vector.

The choice between these approaches depends on the requirements of the application:

- If we only need to observe or process elements incrementally, a View is suitable.
- If we need independent, storable data that can be reused, materializing the result in a Container is more appropriate.

### Common Mistakes When Using Ranges

- Using a View after the underlying data has been destroyed.
- Assuming that constructing a View immediately executes its computations.
- Assuming that all Views provide the same traversal capabilities.
- Using an algorithm such as Sort with an unsuitable Range.
- Using a Predicate with Side Effects or unstable behavior in Filter.
- Assuming that a Transform View caches its results.
- Assuming that Filter and Take While behave identically.
- Forgetting that the ending value in Iota is excluded from the output.

## Summary

The Ranges library in C++20 extends the way we work with collections of data through Ranges, new algorithms, Views, and Range Adaptors.

The most important concepts in this library include:

- **Range:** A sequence of traversable elements.
- **Range Algorithms:** Versions of standard algorithms that work with Ranges and Constraints.
- **Projection:** Specifies which part of each element an algorithm should use when performing an operation.
- **View:** A representation of data that generally does not own the underlying data.
- **Range Adaptor:** A tool for creating a View from a Range.
- **Filter:** Selects elements that satisfy a condition.
- **Transform:** Transforms elements during traversal.
- **Take and Drop:** Limit or skip a portion of the elements.
- **Pipe Operator:** Combines multiple processing stages.
- **Iota:** Generates a sequence of values without requiring an initial Container.

Ultimately, Ranges is not merely a shorter way to write loops. It introduces a different approach to designing data-processing pipelines, in which individual processing stages can be defined independently and then combined.

When designing modern C++ code, understanding the differences between Containers, Iterators, Ranges, and Views is essential. With a clear understanding of these concepts, we can use the capabilities of Ranges effectively and write code that is more readable, composable, and, in many cases, more efficient.


# Comprehensive Guide to the Ranges Library in C++20

The Ranges library in the C++20 standard is one of the most foundational shifts to occur in C++ since the introduction of the Standard Template Library (STL). By combining Concepts with modern language features, it redesigns the traditional model of working with containers, iterators, and algorithms to make code safer, more readable, more composable, and more efficient.

In the classic STL model (from C++98 through C++17), the three core components—Containers, Iterators, and Algorithms—operated separately, and connecting a container to an algorithm always required passing a pair of begin and end iterators manually. Despite its flexibility, this design had serious drawbacks, including verbose code, the possibility of passing mismatched iterators from two different containers, opaque template error messages at compile time, and the lack of a standard mechanism for chaining operations over data without allocating intermediate memory. The `std::ranges` library in the `<ranges>` header and its corresponding algorithms in the `<algorithm>` header address these challenges at their root.

---

## Core Ranges Concepts and the Iterator Hierarchy in C++20

In C++20, the concept of a Range applies to any object over which elements can be iterated; more precisely, any type on which `std::ranges::begin()` and `std::ranges::end()` can be called is considered a Range. This definition includes all standard containers such as `std::vector`, `std::list`, and `std::map`, raw C-style arrays, `std::string_view` strings, and `std::span` ranges.

### The Sentinel Concept in the Ranges Library

One of the most important architectural changes in C++20 is decoupling the type of the end iterator from the begin iterator. In the traditional STL, algorithms expected `begin` and `end` to have the exact same type. In C++20, the end of a range can be a **Sentinel**—a distinct type that only specifies the termination condition for iteration (for example, the null terminator `\0` in C-style strings or a counter in infinite ranges). The iterator and the sentinel only need to be comparable with `==`.

### Iterator and Range Hierarchy

Using language Concepts, the C++20 library elevates the classification of iterators and ranges from legacy iterator tags to compile-time checks. This hierarchy is ordered from the weakest traversal capability to the strongest memory access capability:

| Hierarchy Level | Iterator Concept | Range Concept | Key Capability and Example Container |
| --- | --- | --- | --- |
| Input | `std::input_iterator` | `std::ranges::input_range` | Single-pass reading and forward traversal (`std::istream_iterator`) |
| Output | `std::output_iterator` | `std::ranges::output_range` | Single-pass writing and forward traversal (`std::ostream_iterator`) |
| Forward | `std::forward_iterator` | `std::ranges::forward_range` | Multi-pass forward traversal (`std::forward_list`) |
| Bidirectional | `std::bidirectional_iterator` | `std::ranges::bidirectional_range` | Forward and backward movement with `++` and `--` (`std::list`, `std::map`) |
| Random Access | `std::random_access_iterator` | `std::ranges::random_access_range` | Constant-time $O(1)$ jumps and indexing (`std::deque`) |
| Contiguous | `std::contiguous_iterator` | `std::ranges::contiguous_range` | Elements stored in a contiguous block of memory (`std::vector`, `std::array`) |

Each lower level in the table above also satisfies all capabilities of the higher levels; for example, every `contiguous_range` is automatically a `random_access_range` and a `bidirectional_range` as well. Introducing `contiguous_range` in C++20 made it possible for algorithms and low-level code to rely safely on physical memory contiguity (such as for SIMD optimizations or `memcpy`).

---

## Range Algorithms in the std::ranges Namespace

Traditional standard algorithms in the `std` namespace require explicitly passing two iterators, whereas their modern counterparts in the `std::ranges` namespace (introduced in C++20) can accept the entire container or range directly. Technically, these algorithms are implemented as **Niebloids** (special function objects in the standard library) to prevent interference with Argument-Dependent Lookup (ADL) and to produce much clearer compiler diagnostics.

To compare the traditional approach with the modern approach, examining the `all_of` algorithm in both forms is illustrative:

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{2, 6, 8, 40, 64, 70};

    // Legacy STL approach using iterator pairs (C++11/17)
    bool legacy_all_even = std::all_of(
        std::begin(numbers), std::end(numbers),
        [](int i) { return i % 2 == 0; }
    );

    // Modern C++20 ranges approach passing the container directly
    auto is_even = [](int i) { return i % 2 == 0; };
    bool ranges_all_even = std::ranges::all_of(numbers, is_even);

    if (legacy_all_even && ranges_all_even) {
        std::cout << "All numbers are even.\n";
    }
}

```

Passing the container directly to `std::ranges::all_of()` not only eliminates repeating the variable name `numbers` in `begin` and `end`, but also reduces the chance of bugs caused by accidentally passing `v1.begin()` and `v2.end()`.

### Commonly Used Algorithms in std::ranges and Stream Output

Many widely used algorithms—such as `std::ranges::for_each()`, `std::ranges::sort()`, `std::ranges::find_if()`, and `std::ranges::copy()`—have been rewritten. An important detail regarding search algorithms in `std::ranges` is preventing **Dangling Iterators**: if you pass a temporary container (an rvalue) to `std::ranges::find()`, the return type is `std::ranges::dangling` rather than an invalidated iterator, so attempting to dereference it causes a compile-time error.

In addition, combining `std::ranges::copy()` with `std::ostream_iterator` provides a clean, loop-free way to print the contents of a range directly to an output stream:

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{11, 2, 6, 4, 8, 3, 17, 9};

    // Multiply each element in-place by 2
    std::ranges::for_each(numbers, [](int& n) { n *= 2; });

    // Sort the entire container directly
    std::ranges::sort(numbers);

    // Search for the first element greater than 20
    auto it = std::ranges::find_if(numbers, [](int n) { return n > 20; });
    if (it != std::ranges::end(numbers)) {
        std::cout << "First element > 20: " << *it << '\n';
    }

    // Stream elements directly to standard output on the fly
    std::cout << "Sorted elements: ";
    std::ranges::copy(numbers, std::ostream_iterator<int>(std::cout, " "));
    std::cout << '\n';
}

```

---

## Constrained Iterator Pair Algorithms

Every algorithm in the `std::ranges` namespace also provides a second overload that takes a start iterator and an end sentinel instead of a full container. This overload is designed for cases where you want to run an algorithm on only part of a container (a subrange) or on structures where only iterators are available.

The critical difference between these `std::ranges` overloads and traditional `std` algorithms is that they are **constrained with Concepts**. As a result, compiler error messages are far more accurate and understandable when an iterator type does not meet an algorithm's requirements.

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <list>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> vec{11, 2, 6, 4, 8, 3, 17, 9};

    // Using constrained iterator-pair overload on a subrange
    std::ranges::sort(vec.begin(), vec.begin() + 4);

    std::ranges::copy(vec.begin(), vec.end(),
                      std::ostream_iterator<int>(std::cout, " "));
    std::cout << '\n';

    // std::list provides bidirectional iterators, NOT random access iterators
    std::list<int> linked_list{11, 2, 6, 4, 8};

    // Compilation error with a clear concept constraint message:
    // std::ranges::sort requires std::random_access_iterator!
    // std::ranges::sort(linked_list.begin(), linked_list.end());
}

```

If traditional `std::sort()` had been used instead of `std::ranges::sort()` on `std::list` in the code above, the compiler would have produced a long, cryptic error about a missing `-` operator on list iterators, whereas `std::ranges::sort()` immediately reports that the `std::random_access_iterator` constraint is not satisfied. For this reason, in C++20 and later standards, `std::ranges` algorithms should generally be preferred over legacy `std` algorithms even when working with iterator pairs.

---

## Projections in Range Algorithms

One of the most practical innovations in the `std::ranges` library is **Projection**. A projection is a unary function or callable that an algorithm applies to each element before passing the result to the comparator or predicate, mapping the element to the desired value or attribute.

In C++17 and earlier, if you wanted to sort an array of `Student` objects by age, you had to write a custom comparator that extracted the `m_age` member from two objects. With a projection, the comparison logic (such as default `std::less`) stays untouched and only the target field is specified. Beyond lambdas, thanks to `std::invoke()` under the hood in the standard library, you can also pass **pointers to data members** or **pointers to member functions** directly as projections.

```cpp
#include <algorithm>
#include <cmath>
#include <compare>
#include <iostream>
#include <ranges>
#include <string>
#include <utility>
#include <vector>

struct Point {
    double m_x{};
    double m_y{};

    double length() const {
        return std::hypot(m_x, m_y);
    }

    // Default ordering based on Euclidean distance from origin
    auto operator<=>(const Point& other) const {
        return length() <=> other.length();
    }
    bool operator==(const Point& other) const {
        return length() == other.length();
    }
};

int main() {
    std::vector<Point> points{{10.0, 90.0}, {30.0, 70.0}, {20.0, 80.0}};

    // 1. Default sort: uses Point::operator<=> (sorts by distance)
    std::ranges::sort(points);

    // 2. Sort using a lambda projection on m_x
    std::ranges::sort(points, std::ranges::less{}, [](const Point& p) {
        return p.m_x;
    });

    // 3. Sort using direct pointer-to-member projection on m_y
    std::ranges::sort(points, std::ranges::less{}, &Point::m_y);

    for (const auto& p : points) {
        std::cout << "Point(" << p.m_x << ", " << p.m_y << ")\n";
    }

    // 4. Using projections with std::ranges::for_each on pairs
    using Pair = std::pair<int, std::string>;
    std::vector<Pair> pairs{{1, "one"}, {2, "two"}, {3, "three"}};

    auto print_val = [](const auto& val) { std::cout << val << ' '; };

    std::cout << "Keys: ";
    std::ranges::for_each(pairs, print_val, &Pair::first);

    std::cout << "\nValues: ";
    std::ranges::for_each(pairs, print_val, &Pair::second);
    std::cout << '\n';
}

```

Projections make code significantly more concise, readable, and free of duplicated comparison logic. They are supported across most `std::ranges` algorithms, including `sort`, `find`, `count_if`, `lower_bound`, `min_element`, and `for_each`.

---

## Views and Range Adaptors

While algorithms execute operations immediately (eagerly), **Views** provide a mechanism for lazy evaluation and non-owning operations over ranges.

A View is a lightweight range that does not own the underlying data; instead, it acts like a window or lens that alters how elements of another range are viewed and traversed.

### Key Characteristics of Views

* **Non-owning:** A View does not copy or store data; it merely holds a reference or view over the source range along with the filtering or transformation logic.
* **Lazy evaluation:** Constructing a View performs no computation or traversal (constant time $O(1)$). Computations happen on the fly per element only when the consumer iterates over the View (for example, in a `for` loop or a `copy` algorithm).
* **Cheap to copy:** According to the C++20 standard, copying, moving, and destroying any View must take constant time $O(1)$, so Views can be safely passed around by value.

### Differences Between View Classes and Range Adaptors

The standard library provides two namespaces for working with Views:

* The **`std::ranges`** namespace contains the primary View class templates, such as `std::ranges::filter_view`, `std::ranges::transform_view`, and `std::ranges::take_view`.
* The **`std::views`** namespace (an alias for `std::ranges::views`) contains **Range Adaptors** or factory-like function objects, such as `std::views::filter`, `std::views::transform`, and `std::views::take`.

An important distinction is that Range Adaptors support the pipe operator (`|`) for chaining and perform smart adaptations in certain cases—for example, `std::views::all` returns a View directly if the input is already a View, whereas constructing a class template directly can sometimes encounter template argument deduction issues or constructor constraints. In practical code, adaptors in the `std::views` namespace are standard.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

void print_view(auto view) {
    // Lazy evaluation happens during iteration here
    for (const auto& item : view) {
        std::cout << item << ' ';
    }
    std::cout << '\n';
}

int main() {
    std::vector<int> numbers{1, 11, 23, 131, 2, 3, 4, 5, 6, 7, 8, 9};

    // filter: keeps elements satisfying the predicate
    auto evens = std::views::filter(numbers, [](int i) { return i % 2 == 0; });
    std::cout << "Evens: ";
    print_view(evens);

    // transform: projects each element to a new value on the fly
    auto multiplied = std::views::transform(numbers, [](int i) { return i * 10; });
    std::cout << "Multiplied: ";
    print_view(multiplied);

    // take: takes the first N elements
    std::cout << "First 5: ";
    print_view(std::views::take(numbers, 5));

    // take_while: takes elements as long as the predicate holds true, then stops
    std::cout << "Take while odd: ";
    print_view(std::views::take_while(numbers, [](int i) { return i % 2 != 0; }));

    // drop: skips the first N elements
    std::cout << "Drop first 5: ";
    print_view(std::views::drop(numbers, 5));

    // drop_while: skips elements while the predicate holds true, then yields the rest
    std::cout << "Drop while odd: ";
    print_view(std::views::drop_while(numbers, [](int i) { return i % 2 != 0; }));
}

```

A subtle difference between `filter` and `take_while` (or `drop_while`) is that `filter` evaluates the predicate across the range and yields every matching element, whereas `take_while` stops iteration completely as soon as it encounters the first element that fails the predicate.

---

## View Composition and the Pipe Operator

The real power of the Ranges library emerges when you connect multiple Views together to build a data processing pipeline. In the traditional approach, filtering even numbers, squaring them, and taking the first three results required multiple `for` loops or creating several temporary `std::vector` containers in memory.

With Views, you can compose these operations without allocating temporary memory for intermediate collections. Composition can be written in two ways: nested function calls (raw function composition) and the pipe operator (`|`).

### Comparing Nested Function Composition and the Pipe Operator

With nested function calls, code reads from the inside out, which reduces readability in longer pipelines. The `|` operator, overloaded for Range Adaptors, expresses data flow naturally from left to right:

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <map>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<int> vi{1, 2, 3, 4, 5, 6, 7, 8, 9};
    auto is_even = [](int n) { return n % 2 == 0; };
    auto square = [](int n) { return n * n; };

    // 1. Raw function composition (reads inside-out)
    auto nested_view = std::views::transform(std::views::filter(vi, is_even), square);

    // 2. Pipe operator composition (reads naturally left-to-right)
    auto piped_view = vi
                    | std::views::filter(is_even)
                    | std::views::transform(square);

    for (int val : piped_view) {
        std::cout << val << ' '; // Outputs: 4 16 36 64
    }
    std::cout << '\n';

    // Working with associative containers using keys, values, and reverse
    std::map<std::string, unsigned int> classroom{
        {"John", 11}, {"Mary", 17}, {"Steve", 15}, {"Lucy", 14}, {"Ariel", 12}
    };

    auto before_M = [](const std::string& name) {
        return !name.empty() && static_cast<unsigned char>(name[0]) < static_cast<unsigned char>('M');
    };

    std::cout << "Names before M: ";
    std::ranges::copy(
        classroom | std::views::keys | std::views::filter(before_M),
        std::ostream_iterator<std::string>(std::cout, " ")
    );

    std::cout << "\nAll names in reverse: ";
    std::ranges::copy(
        classroom | std::views::keys | std::views::reverse,
        std::ostream_iterator<std::string>(std::cout, " ")
    );
    std::cout << '\n';
}

```

### Structural Constraints When Composing Views

When chaining adaptors, you must respect the iterator constraints required by each adaptor. For example, `std::views::reverse` requires a `std::ranges::bidirectional_range` so it can move backward from the end. If you used `std::unordered_map` instead of `std::map` in the example above, applying `std::views::reverse` would fail to compile because `std::unordered_map` is only a `forward_range` and does not provide reverse iterators.

---

## Range Factories

All the Views examined so far required an existing container or range as their data source. **Range Factories**, by contrast, are functions in the `std::views` namespace that generate a View from scratch without needing an initial container.

The most important standard factories in C++20 include:

* **`std::views::iota`:** Generates an ascending sequence of values. Given only a starting value, it creates an **infinite range** that produces values lazily on demand. Given an upper bound as well, it creates the half-open range `[start, end)`.
* **`std::views::empty<T>`:** Produces an empty View of type `T` with no elements.
* **`std::views::single(val)`:** Creates a View containing a single element with the value `val`.
* **`std::views::istream<T>(stream)`:** Lazily extracts elements of type `T` from an input stream such as `std::cin` or `std::istringstream`.

```cpp
#include <iostream>
#include <ranges>
#include <sstream>

int main() {
    // 1. Bounded iota: generates numbers from 1 up to 9 (10 is excluded)
    std::cout << "Bounded iota: ";
    for (int i : std::views::iota(1, 10)) {
        std::cout << i << ' ';
    }
    std::cout << '\n';

    // 2. Infinite iota combined with take and transform
    std::cout << "First 5 squares of even numbers: ";
    auto infinite_squares = std::views::iota(1)
                          | std::views::filter([](int n) { return n % 2 == 0; })
                          | std::views::transform([](int n) { return n * n; })
                          | std::views::take(5);

    for (int val : infinite_squares) {
        std::cout << val << ' '; // Outputs: 4 16 36 64 100
    }
    std::cout << '\n';

    // 3. istream_view: lazily parsing integers from a string stream
    std::istringstream input{"10 20 30 40 50"};
    std::cout << "Parsed from stream (doubled): ";
    for (int val : std::views::istream<int>(input) | std::views::transform([](int x) { return x * 2; })) {
        std::cout << val << ' ';
    }
    std::cout << '\n';
}

```

Using `std::views::iota(1)` by itself in a `for` loop without a stopping condition or an adaptor such as `std::views::take` results in an infinite loop (or integer overflow), so pairing infinite factories with bounding adaptors is a standard and essential pattern.

---

## Advanced Notes, Limitations, and Common Pitfalls

Using the Ranges library effectively requires awareness of implementation details and subtle traps that can lead to undefined behavior or degraded performance.

### Underlying Data Lifetime and Dangling Views

Because a View does not own its data, the lifetime of the source data must exceed the lifetime of the View. If a temporary container (an rvalue) is fed directly into a View pipeline and the result is stored in a variable, the View would refer to destroyed memory:

```cpp
#include <ranges>
#include <vector>

std::vector<int> get_temp_vector() {
    return {1, 2, 3, 4, 5};
}

void danger_example() {
    // UNDEFINED BEHAVIOR: get_temp_vector() is destroyed at the end of this statement!
    // std::views::all only accepts lvalues or safe viewable_ranges (like std::span or std::string_view).
    // Note: C++20 prevents binding an owning container rvalue directly to most views!
    // auto bad_view = get_temp_vector() | std::views::filter([](int x) { return x > 2; });
}

```

To help prevent this mistake, the C++20 standard defines the `std::ranges::viewable_range` concept, which disallows piping a temporary owning container (such as a temporary `std::vector`) directly into adaptors. However, if you create a local container (an lvalue) inside a function and return a View over it, you will still encounter a dangling reference.

### Begin Caching and Why Views Should Not Be const

Certain Views—such as `std::ranges::filter_view` and `std::ranges::drop_while_view`—**cache** the result of their initial search inside the View object so that subsequent calls to `begin()` guarantee amortized $O(1)$ time complexity. Because of this caching, `begin()` on these Views is non-`const`.

As a practical consequence, **you cannot iterate over a `const filter_view**` or pass it to a function as `const auto&`:

```cpp
#include <iostream>
#include <ranges>
#include <vector>

// Fails to compile if v is filter_view because filter_view::begin() is non-const!
// void print_broken(const auto& v) {
//     for (auto x : v) std::cout << x;
// }

// Correct way: pass views by value (they are O(1) cheap to copy) or by forwarding reference
void print_correct(auto v) {
    for (const auto& x : v) {
        std::cout << x << ' ';
    }
    std::cout << '\n';
}

```

Therefore, a golden rule when working with Views in C++20 is to **pass them by value or by forwarding reference (`auto&&`)** and avoid marking View objects `const`.

### Mutating Container Data While Iterating a View

If you mutate the underlying container (for example, inserting or erasing elements in a `std::vector`) while iterating over a `filter_view`, the cached state inside `filter_view` can become invalidated, leading to undefined behavior. Likewise, mutating values relied upon by a filter predicate or mutating non-reference return values in `std::views::transform` can disrupt element ordering and cached iterators.

### Lack of a Direct View-to-Container Conversion in C++20

In C++20, there is no built-in helper to directly materialize a View pipeline into a new container such as `std::vector`; developers typically rely on `std::ranges::copy()` with `std::back_inserter` (especially when a `filter_view` is present, where iterator and sentinel types differ). This limitation was resolved in **C++23** with the introduction of `std::ranges::to()`:

```cpp
#include <algorithm>
#include <iterator>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> source{1, 2, 3, 4, 5, 6};
    auto pipeline = source
                  | std::views::filter([](int n) { return n % 2 == 0; })
                  | std::views::transform([](int n) { return n * 10; });

    // C++20 approach to materialize a filtered view into a vector
    std::vector<int> result_cpp20;
    std::ranges::copy(pipeline, std::back_inserter(result_cpp20));

    // C++23 approach (available in modern compilers with -std=c++23)
    // auto result_cpp23 = pipeline | std::ranges::to<std::vector>();
}

```

### When to Use and When Not to Use Ranges and Views

* **When to use:** Whenever you need composable data pipelines, filtering, element transformations, subrange iteration, or sorting and searching by object fields using projections. Views excel at reducing memory usage, avoiding temporary allocations, and improving readability in declarative code.
* **When not to use:** In performance-critical hot loops with deeply nested chains of `filter_view` and `transform_view` where complex lambdas prevent the compiler from fully inlining the pipeline; in such cases, a straightforward `for` loop or low-level vectorized code may generate more optimal machine code. In addition, for parallel execution policies (`std::execution::par` and related policies introduced in C++17), `std::ranges` algorithms in C++20 do not accept an `ExecutionPolicy` parameter, so `std` algorithms are still required for parallel execution.


---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>