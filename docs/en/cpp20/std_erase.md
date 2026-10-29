<div align="center">

[🇺🇸 English](./std_erase.md) | [🇮🇷 فارسی](../../fa/cpp20/std_erase.md)

</div>

---

# Introduction to `std::erase` in C++20

## Introduction

Managing elements in containers is one of the most common operations in C++ programming. In many situations, it is necessary to remove one or more elements from a container based on a specific value or condition.

Before C++20, removing elements from containers such as `std::vector` typically required combining the `std::remove` algorithm with the member function `erase`. This technique, known as the Erase-Remove Idiom, was fully standard and widely used. However, it was not particularly intuitive for beginners and could sometimes lead to mistakes when working with the API.

C++20 introduced the non-member functions `std::erase` and `std::erase_if` to simplify this process.

These functions allow elements to be removed directly based on a value or predicate, hiding the details of the removal algorithm and the subsequent call to the container's member function `erase`.

For example, in C++17, removing all elements equal to a specific value from a `std::vector` required the following code:

```
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 2, 4, 2, 5};

    values.erase(
        std::remove(values.begin(), values.end(), 2),
        values.end()
    );
}
```

Starting with C++20, the same operation can be performed with a single function call:

```
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 2, 4, 2, 5};

    std::erase(values, 2);
}
```

This change not only reduces code size but also improves readability and maintainability.

## Introduction to std::erase

The `std::erase` function is a non-member function provided by the C++20 Standard Library.

Its primary purpose is to remove all elements from a container that are equal to a specified value.

### General Syntax

The general usage of this function is:

```
std::erase(container, value);
```

In this structure:

- `container` refers to the target container.
- `value` specifies the value whose matching elements should be removed.
- The return value is the number of elements removed.

### Function Signature

For containers such as `std::vector`, the general function overload has the following structure:

```
template<class T, class Allocator, class U = T>
constexpr typename std::vector<T, Allocator>::size_type
erase(std::vector<T, Allocator>& container, const U& value);
```

This signature indicates that:

- The container is passed as a non-const reference because its contents are modified.
- The target value is passed as `const U&`.
- The return type is `size_type`.
- The type `U` may differ from the container's element type, provided that the required comparison is valid.

The exact signatures of the overloads may differ between container types, but their general behavior is consistent.

### Return Value

The `std::erase` function returns the number of elements that were actually removed.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 2, 4, 2};

    const auto removedCount = std::erase(values, 2);

    std::cout << "Removed: " << removedCount << '\n';
    std::cout << "Remaining: " << values.size() << '\n';
}
```

Output:

```
Removed: 3
Remaining: 3
```

If the specified value does not exist in the container, the return value is zero, and the container remains unchanged.

## Containers Supported by std::erase

The `std::erase` function is provided for a specific set of Standard Library containers.

It should not be considered a universal API for every container type.

### Support for Sequence Containers

In C++20, `std::erase` is provided for the following containers:

| Container               | std::erase Support |
| ----------------------- | ------------------ |
| `std::vector`           | Yes                |
| `std::deque`            | Yes                |
| `std::list`             | Yes                |
| `std::forward_list`     | Yes                |
| `std::basic_string`     | Yes                |
| `std::array`            | No                 |
| `std::span`             | No                 |
| `std::initializer_list` | No                 |

To use this function, include the header associated with the target container.

For example:

```
#include <vector>
#include <deque>
#include <list>
#include <forward_list>
#include <string>
```

### Why Is std::array Not Supported?

The `std::array` type has a fixed size, and removing an element in a way that reduces the container's actual size is incompatible with its design.

For this reason, `std::erase` is not provided for `std::array`.

If the goal is to logically remove elements from a `std::array`, techniques such as moving valid elements toward the beginning of the array and maintaining the number of active elements can be used.

### Why Are std::map and std::set Not Directly Supported?

The `std::erase` function is not provided for associative and unordered associative containers, including:

- `std::set`
- `std::multiset`
- `std::map`
- `std::multimap`
- `std::unordered_set`
- `std::unordered_multiset`
- `std::unordered_map`
- `std::unordered_multimap`

Instead, C++20 introduced `std::erase_if` for removing elements from these containers based on a predicate.

The differences between these two functions are explained in the following sections.

## How std::erase Works

The internal behavior of `std::erase` is not necessarily identical for every container.

Its implementation depends on the underlying data structure and the API provided by each container.

### Behavior in std::vector and std::deque

For containers such as `std::vector` and `std::deque`, elements are removed using an approach similar to the `std::remove` algorithm, followed by removal of the trailing elements from the container.

The `std::remove` algorithm logically removes matching elements from the specified range, but it does not reduce the container's actual size.

To understand this behavior, consider the following vector:

```
std::vector<int> values{10, 20, 30, 20, 40, 20, 50};
```

After executing the following algorithm:

```
auto newEnd = std::remove(
    values.begin(),
    values.end(),
    20
);
```

The elements equal to `20` are moved out of the initial logical range, and the remaining elements are placed at the beginning of the range.

At this point, the vector still contains seven elements.

```
values.erase(newEnd, values.end());
```

The call above removes the trailing elements and reduces the container's size to four.

The `std::erase` function in C++20 provides both operations through a single API.

### Behavior in std::list and std::forward_list

These two containers provide dedicated member functions for removing elements based on a value.

Consequently, the implementation of `std::erase` for these containers can directly use their native removal capabilities.

For example:

```
#include <iostream>
#include <list>

int main() {
    std::list<int> values{1, 2, 3, 2, 4, 2};

    const auto removedCount = std::erase(values, 2);

    std::cout << removedCount << '\n';
}
```

In this case, elements equal to `2` are removed from the linked list.

An important distinction is that removing a node from `std::list` does not move the remaining elements. Iterators and references to elements that were not removed remain valid.

### Behavior in std::basic_string

The `std::erase` function is also provided for `std::basic_string`.

This makes it possible to remove all characters equal to a specified value.

```
#include <iostream>
#include <string>

int main() {
    std::string text = "programming";

    const auto removedCount = std::erase(text, 'm');

    std::cout << text << '\n';
    std::cout << removedCount << '\n';
}
```

Output:

```
prograing
2
```

In this example, all occurrences of the character `m` are removed.

Note that this operation compares character values rather than Unicode characters or grapheme clusters.

## Introduction to std::erase_if

The `std::erase_if` function is another important C++20 feature introduced alongside `std::erase`.

The primary difference between these functions is how they determine which elements should be removed:

- `std::erase` removes elements equal to a specified value.
- `std::erase_if` removes elements for which a predicate returns `true`.

### General Syntax

```
std::erase_if(container, predicate);
```

In this structure:

- `container` is the target container.
- `predicate` is a callable object used to evaluate the elements.

### Basic Example

Suppose we want to remove all even numbers from a vector.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6, 7, 8};

    const auto removedCount = std::erase_if(
        values,
        [](int value) {
            return value % 2 == 0;
        }
    );

    for (const int value : values) {
        std::cout << value << ' ';
    }

    std::cout << "\nRemoved: " << removedCount << '\n';
}
```

Output:

```
1 3 5 7
Removed: 4
```

In this example, a lambda expression checks whether each element is even.

Any element for which the predicate returns `true` is removed.

### Predicate Requirements

The predicate must be callable with the container's element type and must produce a result convertible to `bool`.

The predicate should also represent a valid and stable condition throughout the removal operation.

In particular, it is recommended that predicates:

- Do not depend on unpredictable external state that changes during execution.
- Do not modify the container's elements.
- Avoid unnecessary side effects.

These considerations are particularly important when using lambdas that capture external variables.

## Differences Between std::erase and std::erase_if

These two functions serve different purposes and have different usage patterns.

| Feature                   | std::erase                 | std::erase_if              |
| ------------------------- | -------------------------- | -------------------------- |
| Removal criterion         | A specified value          | A predicate                |
| Removes multiple elements | Yes                        | Yes                        |
| Accepts a lambda          | No                         | Yes                        |
| Supports vector           | Yes                        | Yes                        |
| Supports list             | Yes                        | Yes                        |
| Supports map              | No                         | Yes                        |
| Return value              | Number of removed elements | Number of removed elements |
| Introduced in             | C++20                      | C++20                      |

### When to Use std::erase

When the goal is to remove all elements equal to a specific value, `std::erase` provides the simplest and most readable solution.

```
std::vector<int> values{1, 2, 3, 2, 4};

std::erase(values, 2);
```

### When to Use std::erase_if

When element removal depends on a condition, a complex comparison, or custom logic, `std::erase_if` is the more appropriate choice.

```
std::vector<int> values{10, 15, 20, 25, 30};

std::erase_if(values, [](int value) {
    return value > 20;
});
```

Additionally, `std::erase_if` should be used when removing elements from associative containers.

## Using std::erase with Different Containers

### Removing Elements from std::vector

The `std::vector` type is one of the most common use cases for `std::erase`.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers{
        10, 20, 30, 20, 40, 20, 50
    };

    std::erase(numbers, 20);

    for (const auto number : numbers) {
        std::cout << number << ' ';
    }
}
```

Output:

```
10 30 40 50
```

In this example, all elements equal to `20` are removed.

### Removing Elements from std::deque

The `std::erase` function can also be used with `std::deque`.

```
#include <deque>
#include <iostream>

int main() {
    std::deque<int> numbers{
        1, 2, 3, 2, 4, 2, 5
    };

    const auto removedCount = std::erase(numbers, 2);

    std::cout << "Removed: " << removedCount << '\n';

    for (const auto number : numbers) {
        std::cout << number << ' ';
    }
}
```

Its general removal behavior is similar to that of `std::vector`, but iterator and reference invalidation rules differ between these containers.

### Removing Elements from std::list

For `std::list`, elements are removed using the linked list's dedicated operations.

```
#include <iostream>
#include <list>

int main() {
    std::list<int> numbers{
        10, 20, 30, 20, 40, 20
    };

    const auto removedCount = std::erase(numbers, 20);

    std::cout << "Removed: " << removedCount << '\n';

    for (const auto number : numbers) {
        std::cout << number << ' ';
    }
}
```

Output:

```
Removed: 3
10 30 40
```

### Removing Elements from std::forward_list

The same API is available for `std::forward_list`.

```
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> numbers{
        1, 2, 3, 2, 4, 2
    };

    const auto removedCount = std::erase(numbers, 2);

    std::cout << "Removed: " << removedCount << '\n';

    for (const auto number : numbers) {
        std::cout << number << ' ';
    }
}
```

This feature provides a common API for removing elements from both standard linked-list container types.

### Removing Characters from std::string

One practical application of `std::erase` is removing unwanted characters from a string.

```
#include <iostream>
#include <string>

int main() {
    std::string username = "user_name_123";

    std::erase(username, '_');

    std::cout << username << '\n';
}
```

Output:

```
username123
```

To remove multiple types of characters, `std::erase_if` can be used:

```
#include <iostream>
#include <string>
#include <cctype>

int main() {
    std::string text = "C++20 Programming 123!";

    std::erase_if(text, [](unsigned char ch) {
        return std::isdigit(ch) || std::isspace(ch);
    });

    std::cout << text << '\n';
}
```

Output:

```
C++Programming!
```

In this example, digits and whitespace characters are removed.

An important detail is that functions such as `std::isdigit` and `std::isspace`, declared in `<cctype>`, accept values that are either equal to `EOF` or representable as `unsigned char`. This is why the lambda parameter in the example uses `unsigned char`.

This approach is suitable for processing the bytes of a string, but it is not sufficient for complete Unicode text processing.

## Using std::erase_if with Associative Containers

One of the most important advantages of `std::erase_if` over `std::erase` is its support for associative containers.

### Removing Elements from std::set

Suppose we have a set of numbers and want to remove all numbers smaller than a specific value.

```
#include <iostream>
#include <set>

int main() {
    std::set<int> numbers{
        1, 2, 3, 4, 5, 6, 7
    };

    const auto removedCount = std::erase_if(
        numbers,
        [](int value) {
            return value < 4;
        }
    );

    std::cout << "Removed: " << removedCount << '\n';

    for (const auto value : numbers) {
        std::cout << value << ' ';
    }
}
```

Output:

```
Removed: 3
4 5 6 7
```

### Removing Elements from std::map

In `std::map`, each element is a `std::pair<const Key, T>`.

Therefore, the predicate receives a Pair and can examine both its key and value.

```
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> scores{
        {"Alice", 90},
        {"Bob", 45},
        {"Charlie", 70},
        {"David", 30}
    };

    const auto removedCount = std::erase_if(
        scores,
        [](const auto& entry) {
            return entry.second < 50;
        }
    );

    std::cout << "Removed: " << removedCount << '\n';

    for (const auto& [name, score] : scores) {
        std::cout << name << ": " << score << '\n';
    }
}
```

Output:

```
Removed: 2
Alice: 90
Charlie: 70
```

In this example, entries with scores below `50` are removed.

Using `const auto&` in the lambda avoids unnecessary copies of container elements.

### Removing Elements from std::unordered_map

This function is also available for unordered associative containers.

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> inventory{
        {"Laptop", 10},
        {"Mouse", 0},
        {"Keyboard", 5},
        {"Monitor", 0}
    };

    const auto removedCount = std::erase_if(
        inventory,
        [](const auto& item) {
            return item.second == 0;
        }
    );

    std::cout << "Removed: " << removedCount << '\n';
}
```

In this example, all products with zero inventory are removed.

Note that the iteration order of `std::unordered_map` is not guaranteed.

## Comparison with the Erase-Remove Idiom

One of the main reasons `std::erase` was introduced was to simplify the traditional Erase-Remove Idiom.

Before C++20, this idiom was commonly used to remove elements from containers supported by the relevant Standard Library algorithms.

### The Traditional Approach in C++17

```
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 2, 4, 2, 5
    };

    numbers.erase(
        std::remove(
            numbers.begin(),
            numbers.end(),
            2
        ),
        numbers.end()
    );
}
```

This approach consists of two steps:

- The `std::remove` algorithm moves unwanted elements out of the logical range.
- The member function `erase` removes the trailing portion of the container.

### The Modern Approach in C++20

```
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 2, 4, 2, 5
    };

    std::erase(numbers, 2);
}
```

With the new approach, the implementation details are hidden from the calling code.

### Comparison for Predicate-Based Removal

In the traditional approach, removing elements based on a condition required `std::remove_if`:

```
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 4, 5, 6
    };

    numbers.erase(
        std::remove_if(
            numbers.begin(),
            numbers.end(),
            [](int value) {
                return value % 2 == 0;
            }
        ),
        numbers.end()
    );
}
```

In C++20, the same operation can be performed as follows:

```
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 4, 5, 6
    };

    std::erase_if(numbers, [](int value) {
        return value % 2 == 0;
    });
}
```

### Advantages of the Modern Approach

Using `std::erase` and `std::erase_if` provides several advantages:

- Less complex code.
- Improved readability.
- Reduced risk of mistakes when working with iterators.
- A consistent removal API across different containers.
- Direct access to the number of removed elements.
- Less dependence on the removal algorithm's implementation details in calling code.

However, the Erase-Remove Idiom remains useful when targeting older C++ standards and when working with algorithms that operate on iterator ranges.

## Differences Between std::erase and the Member Function erase

The introduction of the non-member function `std::erase` does not mean that the member function `erase` has been removed or replaced.

These APIs serve different purposes.

### The Member Function erase

The member function `erase` is used to remove elements based on iterators, ranges, or keys. Its exact overloads depend on the container type.

For example:

```
#include <vector>

int main() {
    std::vector<int> numbers{
        10, 20, 30, 40, 50
    };

    numbers.erase(numbers.begin() + 2);
}
```

In this example, the third element is removed.

### The Non-Member Function std::erase

In contrast, `std::erase` is designed to remove all elements equal to a specified value.

```
#include <vector>

int main() {
    std::vector<int> numbers{
        10, 20, 30, 20, 40
    };

    std::erase(numbers, 20);
}
```

### Which One Should You Use?

| Requirement                                     | Appropriate Choice      |
| ----------------------------------------------- | ----------------------- |
| Remove an element at a specific position        | Member function `erase` |
| Remove a range of elements using iterators      | Member function `erase` |
| Remove all elements equal to a specified value  | `std::erase`            |
| Remove elements based on a predicate            | `std::erase_if`         |
| Remove elements from a map based on a condition | `std::erase_if`         |

These APIs complement each other rather than serving as complete replacements for one another.

## Time Complexity and Performance

The computational cost of removal depends on the underlying container data structure.

### Complexity of std::erase Across Different Containers

| Container           | Overall Complexity |
| ------------------- | ------------------ |
| `std::vector`       | Linear             |
| `std::deque`        | Linear             |
| `std::list`         | Linear             |
| `std::forward_list` | Linear             |
| `std::basic_string` | Linear             |

In this table, linear complexity means that the cost grows proportionally to the number of elements in the container.

### Complexity for std::vector

For `std::vector`, the elements are examined first, and the remaining elements are moved when necessary.

If the initial number of elements is `N`, the overall time complexity is `O(N)`.

The number of required moves and assignments also depends on the number and positions of the removed elements.

### Complexity for std::list

In `std::list`, finding the elements to remove requires examining the nodes.

Therefore, the search has linear complexity.

However, removing each matching node does not require moving the remaining elements.

### Complexity for Associative Containers

For `std::erase_if` on associative containers, the elements must be examined.

Therefore, the overall predicate evaluation cost is proportional to the number of elements in the container.

Removing elements from tree-based or hash-based structures also incurs the costs associated with their respective data structures.

Consequently, the exact cost depends on the container type and the number of elements removed.

## Managing Iterators and References

One of the most important considerations when removing elements from containers is iterator invalidation.

Removing an element may invalidate iterators, pointers, and references that previously referred to elements in the container.

The exact rules depend on the container type.

### Behavior in std::vector

In `std::vector`, removing elements changes the positions of subsequent elements.

Iterators and references pointing to elements before the first removed element remain valid.

However, iterators and references associated with the erased position and all subsequent positions become invalid.

The previous `end()` iterator also becomes invalid.

For example:

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{
        10, 20, 30, 40, 50
    };

    auto iterator = values.begin();

    std::erase(values, 30);

    std::cout << *iterator << '\n';
}
```

In this example, the iterator points to the first element. Since no element before that position is removed, the iterator remains valid.

However, iterators associated with the erased position or subsequent positions must not be used after the removal operation.

### Behavior in std::list

In `std::list`, removing a node invalidates only iterators and references associated with that node.

Iterators to the remaining elements stay valid.

### Behavior in std::forward_list

In `std::forward_list`, removing a node also invalidates iterators associated with the removed nodes, while iterators to other nodes remain valid.

### Behavior in std::deque

Iterator invalidation rules for `std::deque` are more complicated and depend on the positions of the removed elements.

In particular, the behavior differs depending on whether elements are removed from the beginning, end, or middle of the container.

Therefore, when retaining iterators or references to elements in a `std::deque`, consult the standard invalidation rules associated with its `erase` operations.

### Behavior in std::map and std::set

In associative containers such as `std::map` and `std::set`, removing an element invalidates only iterators and references associated with the removed elements.

Iterators to the remaining elements remain valid.

This property is particularly useful when selectively removing elements from tree-based data structures.

## Important Considerations for Predicates

Using a lambda with `std::erase_if` is straightforward, but several considerations are important for avoiding undesirable behavior.

### Avoid Modifying Elements Inside the Predicate

A predicate should ideally be responsible only for evaluating the removal condition.

For example:

```
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 4, 5
    };

    std::erase_if(numbers, [](const int& value) {
        return value < 3;
    });
}
```

In this example, the predicate only examines each element.

Modifying container elements from inside the predicate can lead to undesirable behavior and violate the algorithm's requirements.

### Do Not Depend on Invocation Order

You should not assume that the predicate is invoked in a particular order unless that order is explicitly guaranteed by the operation's contract.

Predicates should be designed so that their behavior does not depend on invocation order.

### Using Captures in Lambdas

When the removal condition depends on an external value, a lambda capture can be used.

```
#include <vector>

int main() {
    std::vector<int> numbers{
        10, 20, 30, 40, 50
    };

    const int threshold = 30;

    std::erase_if(numbers, [threshold](int value) {
        return value > threshold;
    });
}
```

In this example, all numbers greater than `threshold` are removed.

Capturing by value causes the lambda to store a copy of the specified value.

When using reference captures, consider the lifetime of the external variable and ensure that the reference remains valid.

## Limitations and Considerations

Despite their simplicity and usefulness, these functions have some limitations.

### Container Size Modification

The `std::erase` and `std::erase_if` functions modify the size of the container.

Therefore, they cannot be used with fixed-size containers such as `std::array`.

### Removal Based on Equality

The `std::erase` function relies on equality comparison.

Consequently, the element type must support a valid comparison with the supplied value.

When using custom types, ensure that the required comparison operation is properly defined.

### No Generic Overload for Every Container

These functions are not provided for every Range, View, or custom container type.

For example, it cannot be assumed that every type providing `begin()` and `end()` is automatically compatible with `std::erase`.

When working with a custom container, you may still need to implement a dedicated removal operation or use standard algorithms.

### No Direct Removal from Views

Types such as `std::span` and views such as `std::views::filter` do not own their elements and cannot remove elements from the underlying data structure.

In these situations, modify the container that owns the data rather than the view itself.

### Element Movement Costs

In containers such as `std::vector`, removing elements may require moving the remaining elements.

Therefore, if frequent and scattered removals are performed on large collections of data, choosing the appropriate container type becomes important.

For example, in scenarios involving frequent insertions and removals at different positions, `std::list` or another data structure may be a more appropriate choice. However, this decision should account for the access pattern, cache locality, and memory overhead.

## Considerations for Custom Types

The `std::erase` function is not limited to primitive types and can also be used with custom types.

The main requirement is that the necessary comparison operation must be valid.

### Example with a Custom Class

```
#include <iostream>
#include <string>
#include <vector>

class User {
public:
    int id;
    std::string name;

    friend bool operator==(const User& lhs, const User& rhs) {
        return lhs.id == rhs.id;
    }
};

int main() {
    std::vector<User> users{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"},
        {2, "David"}
    };

    const User target{2, ""};

    const auto removedCount = std::erase(users, target);

    std::cout << "Removed: " << removedCount << '\n';
}
```

In this example, the `operator==` compares only the `id` field.

Consequently, every user whose ID matches the target user's ID is removed.

### Design Considerations for operator==

The definition of `operator==` should be consistent with the logical meaning of equality for the custom type.

If the goal is to remove elements based only on a particular field and defining a general equality operation is not appropriate, `std::erase_if` is a better choice.

```
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
        {3, "Charlie"},
        {2, "David"}
    };

    std::erase_if(users, [](const User& user) {
        return user.id == 2;
    });
}
```

This approach avoids changing the general equality semantics of the type.

## Interaction with Move Semantics and Non-Copyable Types

Removing elements from containers such as `std::vector` may involve move assignment or copy assignment of the remaining elements.

Therefore, the element type must satisfy the operational requirements of the container and the removal algorithm.

For example, move-only types such as `std::unique_ptr` can also be used with `std::erase_if`.

```
#include <memory>
#include <vector>

int main() {
    std::vector<std::unique_ptr<int>> values;

    values.push_back(std::make_unique<int>(10));
    values.push_back(std::make_unique<int>(20));
    values.push_back(std::make_unique<int>(30));

    std::erase_if(values, [](const auto& value) {
        return *value == 20;
    });
}
```

In this example, the element containing `20` is removed.

Since `std::unique_ptr` is not copyable, this example demonstrates that removal is not restricted to copyable element types.

However, if a type supports neither move assignment nor copy assignment as required by the operation, using `std::erase` or `std::erase_if` with certain containers may not be possible.

## Differences from std::remove

A common mistake is assuming that `std::remove` and `std::erase` perform the same operation.

These functions have different behaviors.

### Behavior of std::remove

The `std::remove` function:

- Moves elements that do not match the removal value toward the beginning of the specified range.
- Does not change the actual size of the container.
- Returns an iterator to the end of the new logical range.

### Behavior of std::erase

The `std::erase` function:

- Removes the matching elements.
- Reduces the actual size of the container.
- Returns the number of elements removed.

Consider the following example:

```
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 2, 4
    };

    auto newEnd = std::remove(
        numbers.begin(),
        numbers.end(),
        2
    );

    std::cout << "Size: " << numbers.size() << '\n';

    numbers.erase(newEnd, numbers.end());

    std::cout << "Size: " << numbers.size() << '\n';
}
```

Output:

```
Size: 5
Size: 3
```

In contrast, using `std::erase` eliminates the need to manage these two steps separately.

## Constexpr Support

In C++20, many operations involving standard containers and the algorithms required for these capabilities became usable in constant expressions.

The standard overloads of `std::erase` and `std::erase_if` provided for the relevant containers are also `constexpr` in C++20.

This capability makes it possible to use them in constant evaluation contexts, provided that all involved operations and data satisfy the requirements of constant evaluation.

For example, the fact that a function is `constexpr` does not mean that every container or predicate can be used in every context.

Actual constant evaluation support also depends on the Standard Library implementation and the C++ standard version being used.

## Feature-Test Macro

To check whether the standard library supports the new removal functionality, a feature-test macro can be used.

The relevant macro is:

```
__cpp_lib_erase_if
```

The standardized value of this macro for the `erase_if` functionality is:

```
202002L
```

The `<version>` header can be used to check for support:

```
#include <version>

#if defined(__cpp_lib_erase_if) && __cpp_lib_erase_if >= 202002L
    #define HAS_CPP20_ERASE_IF
#endif
```

Note that standard library feature-test macros indicate Standard Library support, not merely support for C++ language features in the compiler frontend.

It is also important to distinguish between C++20 language support and C++20 Standard Library support.

A compiler may support C++20 syntax while the version of its accompanying Standard Library has not yet implemented every required feature.

## Common Mistakes

### Using std::erase with std::map

The following code is invalid:

```
#include <map>

int main() {
    std::map<int, int> values{
        {1, 10},
        {2, 20},
        {3, 30}
    };

    std::erase(values, 2);
}
```

To remove elements from `std::map` based on a condition, use `std::erase_if`:

```
#include <map>

int main() {
    std::map<int, int> values{
        {1, 10},
        {2, 20},
        {3, 30}
    };

    std::erase_if(values, [](const auto& entry) {
        return entry.first == 2;
    });
}
```

### Confusing the Return Value with the New Size

The return value of `std::erase` is the number of removed elements, not the container's size after removal.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 2, 3};

    const auto result = std::erase(values, 2);

    std::cout << result << '\n';
    std::cout << values.size() << '\n';
}
```

Output:

```
2
2
```

### Using Invalid Iterators

After removing elements from `std::vector`, previously obtained iterators must not be reused without first verifying their validity.

If new iterators are required after removal, obtain them from the container again.

### Using std::erase with an Immutable Container

Since these functions modify the container's contents, they cannot be called directly on `const` containers.

```
#include <vector>

int main() {
    const std::vector<int> values{1, 2, 3, 2};

    // This does not compile.
    // std::erase(values, 2);
}
```

### Using std::remove Without erase

Using `std::remove` alone does not reduce the size of a `std::vector`.

If the goal is to actually remove elements from the container, use the Erase-Remove Idiom or the newer `std::erase` API.

## Design Recommendations and Best Practices

For professional C++20 development, the following practices are recommended:

- Use `std::erase` to remove all elements equal to a specified value from `std::vector`, `std::deque`, `std::list`, `std::forward_list`, and `std::string`.
- Use `std::erase_if` to remove elements based on a condition.
- Use `std::erase_if` for associative containers such as `std::map` and `std::set`.
- When only one element at a specific position needs to be removed, the member function `erase` is more appropriate.
- Pay attention to iterator and reference invalidation when removing elements from `std::vector` and `std::deque`.
- Avoid unnecessary side effects in lambdas used as predicates.
- When designing custom types, define `operator==` only when a general and meaningful equality relationship exists for the type.
- For projects targeting older C++ standards, use the Erase-Remove Idiom.
- In C++20 and newer projects, prefer the new APIs for common removal operations.
- For non-copyable types, verify move-assignment capabilities and the requirements of the target container.

## Summary

The `std::erase` and `std::erase_if` functions, introduced in C++20, simplify and improve the readability of element removal operations in standard containers.

The `std::erase` function removes all elements equal to a specified value, while `std::erase_if` makes it possible to remove elements based on a predicate.

The most important characteristics of these functions include:

- Direct element removal without manually writing the Erase-Remove Idiom.
- Support for commonly used sequence containers.
- Support for associative and unordered associative containers through `std::erase_if`.
- Returning the number of removed elements.
- Preserving the container-specific iterator invalidation rules.
- Reducing code complexity and improving readability.

Ultimately, using these two APIs in C++20 projects makes element removal clearer, more standardized, and easier to maintain, without requiring developers to manually implement the algorithm and iterator-management details for every removal operation.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>