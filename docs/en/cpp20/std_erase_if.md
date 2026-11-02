<div align="center">

[🇺🇸 English](./std_erase_if.md) | [🇮🇷 فارسی](../../fa/cpp20/std_erase_if.md)

</div>

---

# Comprehensive Guide to `std::erase_if` in C++20

## Introduction

`std::erase_if` is a C++20 standard library feature that makes it easier and clearer to remove elements from a container based on a condition. Before its introduction, conditionally removing elements from many containers required combining algorithms such as `std::remove_if` with an `erase` operation. This pattern was suitable for some containers, but it could not be used with all of them. Differences in container behavior also made element-removal code more complicated than necessary.

The `std::erase_if` function provides a unified non-member interface for conditional element removal. It hides the container-specific details: for example, it uses the Remove-Erase idiom for `std::vector`, while associative containers use their own element-removal mechanisms.

This feature is part of the **C++20 Standard Library**, not a language feature or a compiler extension. To use it, both your compiler and its standard library implementation must support C++20.

## The Problem It Solves

Suppose you have a collection of numbers and want to remove all negative values. In C++17 and earlier, the usual approach for `std::vector` was:

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, -2, 3, -4, 5};

    auto newEnd = std::remove_if(
        values.begin(),
        values.end(),
        [](int value) {
            return value < 0;
        }
    );

    values.erase(newEnd, values.end());
}
```

The `std::remove_if` algorithm does not change the container's size by itself. It moves the elements to be retained toward the beginning of the range and returns an iterator to the new logical end. The trailing elements are still part of the container and must be removed using `erase`.

In C++20, the same operation for `std::vector` can be written as follows:

```cpp
#include <vector>

int main() {
    std::vector<int> values{1, -2, 3, -4, 5};

    std::erase_if(values, [](int value) {
        return value < 0;
    });
}
```

In this version, both the removal condition and the container-size adjustment are handled by a single call. This reduces the chance of forgetting the `erase` step and expresses the intent more directly.

## Introducing the Standard Interface

`std::erase_if` is a non-member function template with separate overloads for supported container types. Its exact declaration depends on the container type, so it should not be thought of as one template with an identical signature for every container.

The conceptual calling pattern is:

```cpp
std::erase_if(container, predicate);
```

- The first argument is the container to operate on.
- The second argument is a predicate that is evaluated for each element.
- If the predicate returns `true`, the corresponding element is removed.
- The return value is the number of elements removed, with the container's `size_type`.

For example:

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6};

    const auto removed = std::erase_if(values, [](int value) {
        return value % 2 == 0;
    });

    std::cout << "Removed: " << removed << '\n';
    std::cout << "Remaining:";

    for (int value : values) {
        std::cout << ' ' << value;
    }

    std::cout << '\n';
}
```

Output:

```text
Removed: 3
Remaining: 1 3 5
```

## Predicate Behavior

### Meaning of the Return Value

The predicate decides whether a given element should be removed. Returning `true` means the element is removed; returning `false` means it is retained.

```cpp
#include <string>
#include <vector>

int main() {
    std::vector<std::string> names{
        "Alice",
        "",
        "Bob",
        "",
        "Charlie"
    };

    std::erase_if(names, [](const std::string& name) {
        return name.empty();
    });
}
```

In this example, empty strings are removed.

### Predicate Argument Type

It is generally preferable to write the predicate with a `const` reference parameter unless the type or specific logic requires otherwise. This avoids unnecessary copies of elements, especially when they are large objects.

```cpp
std::erase_if(records, [](const Record& record) {
    return record.isExpired();
});
```

For small, simple types such as `int`, passing by value is also usually appropriate:

```cpp
std::erase_if(values, [](int value) {
    return value < 0;
});
```

The argument type must be compatible with the container's `value_type` or with the type supplied to the predicate by the corresponding overload. For maps and multimaps, an element has the type `std::pair<const Key, T>`, so the predicate is commonly written with `const auto&` or `const std::pair<const Key, T>&`.

### Keeping the Predicate Pure and Stable

The predicate should provide a consistent and reliable decision for each element. Structurally modifying the container from inside the predicate, including inserting or removing elements from that same container, can invalidate iterators and interfere with the internal traversal, making the program's behavior invalid or unpredictable.

It is also preferable for the predicate not to make decisions based on external state that changes during the operation. Avoid unnecessary side effects such as modifying shared counters, changing other elements, or depending on the order of evaluation. Do not assume more about the number or order of predicate invocations than is guaranteed by the contract of the relevant algorithm or overload.

## Supported Containers and Implementation Differences

### Array-Based Sequence Containers

For `std::vector` and `std::deque`, conditional removal is performed by moving the elements that are retained and then shrinking the container. Conceptually, this is the same as the Remove-Erase idiom.

```cpp
#include <deque>
#include <vector>

int main() {
    std::vector<int> numbers{1, 2, 3, 4, 5};
    std::deque<int> queue{1, 2, 3, 4, 5};

    const auto removedFromVector = std::erase_if(
        numbers,
        [](int value) { return value > 3; }
    );

    const auto removedFromDeque = std::erase_if(
        queue,
        [](int value) { return value > 3; }
    );
}
```

For these containers, the relative order of the remaining elements is preserved. The operation may involve examining every element and moving retained elements, so its time complexity is generally linear in the container's size.

### List Containers

For `std::list` and `std::forward_list`, removal uses the containers' own member mechanisms. These containers can remove nodes without moving the values stored in other elements.

```cpp
#include <forward_list>
#include <list>

int main() {
    std::list<int> values{1, 2, 3, 4, 5};
    std::forward_list<int> linkedValues{1, 2, 3, 4, 5};

    std::erase_if(values, [](int value) {
        return value % 2 == 0;
    });

    std::erase_if(linkedValues, [](int value) {
        return value % 2 == 0;
    });
}
```

This distinction matters: `std::erase_if` provides a consistent interface, but it does not necessarily run the same internal algorithm for every container.

### Ordered Associative Containers

For `std::set`, `std::multiset`, `std::map`, and `std::multimap`, elements are stored according to ordering rules in a tree-based structure. The `std::remove_if` algorithm is not suitable for these containers because their keys and structure cannot be changed through ordinary element movement.

`std::erase_if` removes matching elements using the container's own removal operation:

```cpp
#include <map>
#include <set>
#include <string>

int main() {
    std::set<int> ids{10, 11, 12, 13, 14};

    std::erase_if(ids, [](int id) {
        return id < 12;
    });

    std::map<int, std::string> users{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"}
    };

    std::erase_if(users, [](const auto& entry) {
        return entry.second == "Bob";
    });
}
```

In the map example, the predicate receives a key-value pair. The `first` member is the key, and the `second` member is the value. A map's key is `const`, so it cannot be modified through this operation.

### Unordered Associative Containers

Dedicated overloads are also available for `std::unordered_set`, `std::unordered_multiset`, `std::unordered_map`, and `std::unordered_multimap`.

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 90},
        {"Bob", 45},
        {"Charlie", 72}
    };

    const auto removed = std::erase_if(
        scores,
        [](const auto& entry) {
            return entry.second < 60;
        }
    );
}
```

The iteration order of unordered containers is not guaranteed. Therefore, do not make your program depend on a fixed traversal order or a particular order in which elements are examined.

### Standard Strings

Starting with C++20, the non-member `std::erase_if` function is also available for `std::basic_string`. This is useful for removing characters selected by a predicate.

```cpp
#include <cctype>
#include <string>

int main() {
    std::string text = "C++20 is powerful!";

    std::erase_if(text, [](unsigned char ch) {
        return std::isspace(ch);
    });
}
```

When using the `<cctype>` family of functions such as `std::is...`, converting the argument to `unsigned char` is important. Passing a negative `char` value to these functions, other than `EOF`, can result in undefined behavior.

## Practical Examples

### Removing Invalid Data

```cpp
#include <vector>

struct Measurement {
    double value;
    bool valid;
};

int main() {
    std::vector<Measurement> measurements{
        {12.5, true},
        {-1.0, false},
        {18.2, true},
        {0.0, false}
    };

    std::erase_if(measurements, [](const Measurement& item) {
        return !item.valid;
    });
}
```

In this pattern, the predicate expresses the validation logic clearly and keeps it separate from the container-management logic.

### Removing Expired Records

```cpp
#include <chrono>
#include <vector>

struct Session {
    std::chrono::system_clock::time_point expiresAt;
};

int main() {
    using Clock = std::chrono::system_clock;

    std::vector<Session> sessions;
    const auto now = Clock::now();

    std::erase_if(sessions, [now](const Session& session) {
        return session.expiresAt <= now;
    });
}
```

In this example, the current time is calculated once and captured by the lambda. As a result, every element is evaluated against the same time reference.

### Removing Entries Based on a Criterion

```cpp
#include <string>
#include <vector>

struct User {
    int id;
    std::string name;
    bool active;
};

int main() {
    std::vector<User> users{
        {1, "Alice", true},
        {2, "Bob", false},
        {3, "Charlie", true}
    };

    std::erase_if(users, [](const User& user) {
        return !user.active;
    });
}
```

This example removes inactive users. If the goal is to remove duplicates, a rule for choosing which representative to retain must also be defined; `std::erase_if` does not perform deduplication by itself.

### Removing Entries from a Map Based on Their Values

```cpp
#include <map>
#include <string>

int main() {
    std::map<std::string, int> inventory{
        {"apple", 10},
        {"banana", 0},
        {"orange", 7}
    };

    std::erase_if(inventory, [](const auto& item) {
        return item.second == 0;
    });
}
```

If the goal is to remove a specific key rather than conditionally remove multiple elements, the member `erase(key)` operation is usually simpler and more appropriate.

## Comparison with Older Approaches

### Comparison with the Remove-Erase Idiom

In C++11, C++14, and C++17, the common approach for `vector` and `deque` was to combine `std::remove_if` with `erase`. In C++20, `std::erase_if` combines these two steps behind a single interface.

| Feature | `std::remove_if` with `erase` | `std::erase_if` |
|---|---|---|
| Standard version | Available in earlier standards | C++20 |
| Changes container size | `remove_if` alone does not | Yes |
| Readability | Two steps | One call |
| Support for map and set | The general algorithmic pattern is not applicable | Has dedicated overloads |
| Return value | Iterator to the logical end | Number of removed elements |

The `std::remove_if` algorithm is still useful, for example, when working with a range or a custom container that has no corresponding `std::erase_if` overload, or when an iterator to the logical end is needed.

### Comparison with `container.erase`

The member `erase` function is generally used to remove an element by iterator, a range, or a key. In contrast, `std::erase_if` expresses removal based on a predicate.

```cpp
#include <map>

int main() {
    std::map<int, int> values{
        {1, 10},
        {2, 20},
        {3, 30}
    };

    values.erase(2);

    std::erase_if(values, [](const auto& item) {
        return item.second > 20;
    });
}
```

For removing a specific key from a map, `erase(key)` is more direct. For removing all elements that match a condition, `std::erase_if` is more readable.

## Time Complexity and Runtime Cost

The exact complexity depends on the container type and the corresponding standard overload:

- For sequence containers such as `vector` and `deque`, examining elements and moving retained elements has a linear cost relative to the number of elements.
- For `list` and `forward_list`, traversal is linear, and matching nodes are removed using the containers' member operations.
- For ordered associative containers, traversal is required, and the cost of removing each element depends on the container's removal operation.
- For unordered containers, linear traversal is required, and removal costs depend on the hash-table behavior and the cost of removing elements.

Therefore, the fact that the function has the same name does not mean its cost is identical for every container. If performance is critical, benchmark with the actual container type, number of elements removed, predicate cost, and memory-access pattern.

## Iterator and Reference Validity

Invalidation behavior depends on the container, and `std::erase_if` does not introduce a separate rule that makes it uniform. In practice, follow the invalidation rules of the removal operation used by the corresponding container.

- For `std::vector`, removing elements can invalidate iterators and references at or after the first removed element; iterators referring to the old end are also invalidated.
- For `std::deque`, invalidation rules depend on the location and form of the erase operation. Consult the contract of the container.
- For `std::list` and `std::forward_list`, iterators and references to erased elements are invalidated, while those referring to remaining elements stay valid.
- For ordered associative containers, iterators and references to erased elements are invalidated, while those referring to other elements remain valid.
- For unordered containers, iterators and references to erased elements are invalidated. Iterator rules should be checked against the container's contract and the relevant operation.

If you hold an iterator or reference to an element that might be removed, do not use it after calling `std::erase_if` without first establishing that it remains valid.

## Limitations and Design Considerations

### Unsupported Custom Containers

The existence of `std::erase_if` does not mean that every container type or custom range is automatically supported. The function has overloads for specific standard library container types. For a custom container, use its own appropriate API or, where possible, implement a suitable algorithmic pattern.

### Element Lifetime and Resource Ownership

Removing an element does not merely make it disappear from the container; the destructor of the erased element is also invoked as part of the object's lifetime rules. If an element manages a resource such as memory, a file handle, or a lock, the effects of removal should be considered in the type's design.

### Expensive Predicates

If the predicate performs expensive work such as network access, database access, or complex computation, the predicate itself may dominate the total cost. In such cases, consider preparing the required data before removal or computing the removal criterion more efficiently.

### Removing All Elements

To remove all elements, you could write a predicate that always returns `true`, but in most cases `clear()` is simpler and more explicit.

```cpp
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    values.clear();
}
```

### Removing No Elements

If the predicate returns `false` for every element, the container's contents remain unchanged and the return value is zero. However, in performance-sensitive code, consider the unnecessary predicate evaluations and traversal cost.

### Modifying the Container During Traversal

Do not structurally modify the target container from inside the predicate. For example, calling `push_back` or `erase` on the same container from within the predicate can disrupt the traversal. If other elements need to be inserted, perform that work in a separate step.

## Common Mistakes

- **Getting the predicate's meaning backward:** Returning `true` means remove the element, not retain it.
- **Using `std::remove_if` with map or set:** This algorithm is not appropriate for containers whose elements cannot be modified and moved through their iterators.
- **Assuming `std::remove_if` changes the container's size:** The algorithm does not physically shrink the container by itself, whereas `std::erase_if` performs the actual removal.
- **Using an old iterator after removal:** Iterator validity depends on the container type and the location of the removal.
- **Modifying the container inside the predicate:** This can invalidate the internal traversal.
- **Assuming a fixed order in unordered containers:** The order of elements in these containers is not guaranteed.
- **Ignoring the element type in a map:** A map element is a `pair` with a `const` key, so the predicate must be compatible with that type.
- **Ignoring the return value:** It reports the number of removed elements and can be useful for logging, statistics, or control flow.

## Standard Version and Compatibility Notes

`std::erase_if` was introduced in the C++20 standard library. Merely enabling C++20 language features is not sufficient; the selected standard library implementation must also provide the corresponding declaration and implementation.

For example, with GCC or Clang, you can select the language standard using an appropriate option:

```bash
g++ -std=c++20 main.cpp -o app
clang++ -std=c++20 main.cpp -o app
```

Actual support depends on the compiler version and the standard library version. In Clang-based environments, the selected standard library implementation may be decisive. Therefore, if you encounter an error indicating that `std::erase_if` is unavailable, check the compiler version, standard library version, and build configuration.

Include the header corresponding to the container you use, such as `<vector>`, `<map>`, `<unordered_map>`, `<list>`, or `<string>`. Older algorithms such as `std::remove_if` are declared in `<algorithm>`, but to use the `std::erase_if` overloads, the header for the relevant container type must be available.

## Choosing the Right Approach

- **For conditional removal from supported standard containers:** Use `std::erase_if`.
- **For removing a specific key from a map or set:** `container.erase(key)` is usually the more direct choice.
- **For removing a specific iterator or range:** Use the appropriate `erase` overload.
- **For a custom range or container without a standard overload:** Use the type's own API or an appropriate algorithm such as `std::remove_if` combined with an erase operation.
- **For clearing an entire container:** Use `clear()`.

## Summary

`std::erase_if` in C++20 provides a standard, readable interface for conditionally removing elements from commonly used standard library containers. Its main advantage is that it hides the differences between container-specific removal implementations and eliminates the need to write different removal patterns manually.

To use this feature effectively in professional code, understand the predicate's meaning, the element type in associative containers, removal costs, iterator invalidation, and the limitations of the available overloads. For conditional removal from supported standard containers, this function is usually a good default choice. However, for removal by key, removal of a specific range, or clearing an entire container, the corresponding member operation may be clearer and more efficient.


---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>