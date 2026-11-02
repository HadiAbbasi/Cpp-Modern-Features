<div align="center">

[🇺🇸 English](./std_contains.md) | [🇮🇷 فارسی](../../fa/cpp20/std_contains.md)

</div>

---

# Introduction to `std::contains` in C++20

## Introduction

Checking whether a value exists in a collection is one of the most common operations in C++ programming. For example, you may need to verify that an identifier is not duplicated before inserting it, check whether a key exists in a `std::map`, or determine whether a value has already been recorded in a collection.

In earlier versions of the language, this was typically done using the `find()` member function and comparing its result with `end()`. C++20 introduced the `contains()` member function for standard associative containers, making this intent easier to express and improving code readability.

An important distinction is that `std::contains` is not the name of a general-purpose free function in the `std` namespace. It usually refers to the `contains()` member function provided by containers such as `std::set`, `std::map`, and their unordered counterparts. Similar functionality was also introduced for `std::string` and ranges algorithms in later standards, and these features should be distinguished from the C++20 functionality.

## The Purpose and Concept of std::contains

The `contains()` member function determines whether a container contains an element with the specified key. It returns a value of type `bool`:

- If the specified key exists in the container, the function returns `true`.
- If the key does not exist, the function returns `false`.

This function is intended for situations where you only need the answer to the question "Does this key exist?" and do not need access to the iterator of the matching element.

### A Simple Example

```
#include <iostream>
#include <set>

int main() {
    const std::set<int> numbers{10, 20, 30, 40};

    std::cout << std::boolalpha;
    std::cout << numbers.contains(20) << '\n';
    std::cout << numbers.contains(25) << '\n';
}
```

Output:

```
true
false
```

In this example, `contains()` directly returns the search result as a Boolean value.

## Containers Supported in C++20

C++20 provides the `contains()` member function for standard associative containers.

| Container                 | Search characteristics                 | Typical complexity                |
| ------------------------- | -------------------------------------- | --------------------------------- |
| `std::set`                | Unique, ordered keys                   | `O(log n)`                        |
| `std::multiset`           | Ordered keys with duplicates allowed   | `O(log n)`                        |
| `std::map`                | Unique, ordered keys                   | `O(log n)`                        |
| `std::multimap`           | Ordered keys with duplicates allowed   | `O(log n)`                        |
| `std::unordered_set`      | Unique, unordered keys                 | Average `O(1)`, worst-case `O(n)` |
| `std::unordered_multiset` | Unordered keys with duplicates allowed | Average `O(1)`, worst-case `O(n)` |
| `std::unordered_map`      | Unique, unordered keys                 | Average `O(1)`, worst-case `O(n)` |
| `std::unordered_multimap` | Unordered keys with duplicates allowed | Average `O(1)`, worst-case `O(n)` |

In this table, `n` represents the number of elements in the container. The complexity of ordered containers comes from their balanced tree structures. In contrast, unordered containers use hashing, and their performance depends on hash distribution quality and the number of collisions.

### Containers That Do Not Provide This Function

C++20 did not add the `contains()` member function to every standard container. For example, the following sequence containers do not provide such a member function:

- `std::vector`
- `std::array`
- `std::deque`
- `std::list`
- `std::forward_list`

For these containers, you can use `std::find()` or, in C++20, `std::ranges::find()`. The `std::ranges::contains()` algorithm belongs to C++23, not C++20.

## Usage and Function Overloads

The conceptual function signature for associative containers is:

```
bool contains(const key_type& key) const;
```

This signature indicates that the function:

- Accepts the key to search for.
- Does not modify the container and is `const`.
- Returns only a `bool` value.
- Does not provide the caller with an iterator or reference to the matching element.

When heterogeneous lookup is enabled, an additional templated overload is also available, as explained later.

### Checking for a Key in std::map

In `std::map`, the search is performed using the key, not the value stored in the key-value pair.

```
#include <iostream>
#include <map>
#include <string>

int main() {
    const std::map<int, std::string> users{
        {100, "Alice"},
        {200, "Bob"},
        {300, "Charlie"}
    };

    if (users.contains(200)) {
        std::cout << "User exists\n";
    }
}
```

In this example, the number `200` is checked as a key. The `contains()` function does not return the value associated with that key; it only reports whether the key exists.

### Checking for a Value in std::set

In `std::set`, each element itself acts as a key. Therefore, the search is performed directly on the value.

```
#include <iostream>
#include <set>

int main() {
    const std::set<int> allowedPorts{80, 443, 8080};

    const int requestedPort = 443;

    if (allowedPorts.contains(requestedPort)) {
        std::cout << "Port is allowed\n";
    }
}
```

### Checking for a Key in Unordered Containers

Unordered containers provide the same interface, but their search operations are based on hashing.

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    const std::unordered_map<std::string, int> inventory{
        {"keyboard", 12},
        {"mouse", 25},
        {"monitor", 8}
    };

    if (inventory.contains("mouse")) {
        std::cout << "Item exists\n";
    }
}
```

The order of elements in `std::unordered_map` is not guaranteed. However, this does not affect the semantics of `contains()`.

## Comparing contains and find

Before C++20, the common pattern for checking whether an element existed was:

```
if (container.find(key) != container.end()) {
    // The key exists
}
```

In C++20, the same intent can be expressed as follows:

```
if (container.contains(key)) {
    // The key exists
}
```

Both approaches are valid for checking whether a key exists, but their purposes and return values are different.

| Feature                                             | `contains()`  | `find()`                  |
| --------------------------------------------------- | ------------- | ------------------------- |
| Return value                                        | `bool`        | Iterator                  |
| Directly expresses element existence                | Yes           | By comparing with `end()` |
| Provides access to the found element                | No            | Yes                       |
| Suitable for checking existence only                | Very suitable | Valid, but less direct    |
| Suitable for reading or modifying the found element | No            | Yes                       |

### When Is contains the Better Choice?

Use `contains()` when you only need the Boolean result of a search. Examples include:

- Validating whether an identifier exists in a set of permitted identifiers.
- Preventing a key from being processed more than once.
- Checking whether a configuration entry or option exists in a `std::map`.
- Checking whether an element belongs to a collection of values.

### When Is find More Appropriate?

If you need to access the element after searching, `find()` is the more appropriate choice.

```
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<int, std::string> users{
        {100, "Alice"},
        {200, "Bob"}
    };

    const auto it = users.find(200);

    if (it != users.end()) {
        std::cout << it->second << '\n';
    }
}
```

In this example, using `contains()` alone would not be sufficient because another search would be necessary to retrieve the associated value.

### Avoiding Duplicate Searches

The following pattern is generally not a good choice:

```
if (users.contains(id)) {
    const auto it = users.find(id);
    std::cout << it->second << '\n';
}
```

In this case, the key is searched for twice. If your goal is to access the element, call `find()` once and check the resulting iterator.

## Heterogeneous Lookup

One important capability of associative containers is the ability to search using a type different from `key_type`, provided that the container's comparator or hash and equality functions are configured to support it. This capability is known as heterogeneous lookup.

For example, suppose the keys of a `std::set` are of type `std::string`, but you want to search using a `std::string_view`. Under normal circumstances, constructing a temporary `std::string` might be necessary. A transparent comparator can eliminate this unnecessary conversion.

### Using std::less

with set and map

For ordered containers, the comparator must support heterogeneous comparisons. Using `std::less<>` instead of `std::less<std::string>` makes this possible.

```
#include <iostream>
#include <set>
#include <string>
#include <string_view>

int main() {
    const std::set<std::string, std::less<>> names{
        "Alice",
        "Bob",
        "Charlie"
    };

    const std::string_view query = "Bob";

    std::cout << std::boolalpha;
    std::cout << names.contains(query) << '\n';
}
```

In this example, the search is performed using `std::string_view`, even though the set's key type is `std::string`.

### Using Transparent Hash and Equality with Unordered Containers

For unordered containers, heterogeneous lookup requires transparent hash and equality functors. Both must be compatible with the key type and the lookup type and must satisfy the corresponding hashing and equality requirements.

```
#include <cstddef>
#include <functional>
#include <iostream>
#include <string>
#include <string_view>
#include <unordered_set>

struct TransparentHash {
    using is_transparent = void;

    std::size_t operator()(std::string_view value) const noexcept {
        return std::hash<std::string_view>{}(value);
    }
};

struct TransparentEqual {
    using is_transparent = void;

    bool operator()(std::string_view lhs,
                    std::string_view rhs) const noexcept {
        return lhs == rhs;
    }
};

int main() {
    const std::unordered_set<std::string, TransparentHash, TransparentEqual>
        names{"Alice", "Bob", "Charlie"};

    const std::string_view query = "Charlie";

    std::cout << std::boolalpha;
    std::cout << names.contains(query) << '\n';
}
```

In this example, both functors accept their arguments as `std::string_view`. Therefore, they can be used with both `std::string` keys and a `std::string_view` lookup value.

An important point is that defining `is_transparent` alone is not sufficient. The comparison or hashing operation must actually be valid for the types being used and must maintain consistent behavior for equivalent keys.

### Benefits of Heterogeneous Lookup

This approach can provide several benefits in frequently executed search operations:

- Avoiding the construction of temporary objects of the key type.
- Reducing memory allocations when ordinary conversions require allocation.
- Reducing string conversion costs.
- Supporting searches using `std::string_view` and other compatible types.

However, performance improvements occur only when the lookup type and comparator or hash function genuinely support a conversion-free lookup path. Simply using `contains()` does not guarantee that all temporary-object costs will be eliminated.

## Differences Between Ordered and Unordered Containers

The name and calling convention of `contains()` are identical in both container families, but the choice of container still affects the overall performance of the application.

### Behavior of Ordered Containers

In `std::set` and `std::map`, elements are maintained in sorted order according to the comparator. Searching typically has logarithmic complexity:

\\[ O(\log n) \\]

In addition to predictable search performance, these containers preserve ordering and support operations such as `lower_bound()` and `upper_bound()`.

### Behavior of Unordered Containers

In `std::unordered_set` and `std::unordered_map`, searching is performed using a hash table. The average complexity is typically constant:

\\[ O(1) \\]

However, in the worst case, a large number of collisions can result in linear complexity:

\\[ O(n) \\]

Therefore, choosing an unordered container solely because its average search complexity is `O(1)` is not always the better decision. Ordering requirements, hash quality, memory overhead, access patterns, and worst-case behavior should also be considered.

## Practical Applications

### Input Validation

To verify that an input value belongs to a set of permitted values, `contains()` provides a readable solution.

```
#include <iostream>
#include <set>

int main() {
    const std::set<int> supportedVersions{1, 2, 3};

    const int requestedVersion = 2;

    if (!supportedVersions.contains(requestedVersion)) {
        std::cerr << "Unsupported version\n";
        return 1;
    }

    std::cout << "Version accepted\n";
}
```

### Preventing Duplicate Processing

When processing data, you can store already-processed identifiers in a set.

```
#include <iostream>
#include <string>
#include <unordered_set>
#include <vector>

int main() {
    const std::vector<std::string> incomingIds{
        "A-100", "B-200", "A-100", "C-300"
    };

    std::unordered_set<std::string> processed;

    for (const auto& id : incomingIds) {
        if (processed.contains(id)) {
            continue;
        }

        processed.insert(id);
        std::cout << "Processing " << id << '\n';
    }
}
```

In this example, `contains()` checks whether an identifier already exists, while `insert()` records it. These are two separate operations. In a multithreaded application, this pattern does not by itself provide an atomic "check and insert" operation.

### Checking Whether Configuration Entries Exist

In applications where configuration settings are stored in a map, you can check whether a key exists before accessing it. However, if you intend to read the value immediately after checking, it is generally better to use `find()` directly to avoid repeating the search.

## Important Considerations and Common Mistakes

### Assuming std::contains Is a Free Function

In C++20, associative containers provide `container.contains(key)` as a member function. Writing `std::contains(container, key)` is not valid in this standard.

### Calling contains on a vector

The `std::vector` container does not provide a `contains()` member function in C++20. To search for an element in a vector, you can use the appropriate algorithms.

```
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    const std::vector<int> values{10, 20, 30};

    const bool exists =
        std::ranges::find(values, 20) != values.end();

    std::cout << std::boolalpha << exists << '\n';
}
```

The `std::ranges::find()` algorithm has been available since C++20. The classic `std::find()` algorithm has also been available since earlier standard versions.

### Confusing Key Existence with Value Existence

In `std::map` and `std::unordered_map`, the argument passed to `contains()` is compared against the key, not the value stored in the map. To search for a value, you must iterate over the elements or choose a more suitable data structure.

### Assuming contains Returns the Element

The return value of `contains()` is only a `bool`. If you need an iterator, reference, or the element's value, use `find()`.

### Checking and Then Inserting in a Multithreaded Environment

Calling `contains()` and `insert()` separately does not constitute an atomic operation. If multiple threads access a shared data structure concurrently, appropriate synchronization is required. Ordinary standard containers are not designed to support concurrent access involving modifications without synchronization.

### Ignoring Hashing and Comparison Costs

An average complexity of `O(1)` for unordered containers does not mean that the operation always has negligible, constant real-world cost. The hash function, key comparisons, collisions, and rehashing can all affect actual performance.

## Differences from Similar Features in Later Standards

The name `contains` is used in several parts of the standard library, but not all of these features belong to C++20.

### The contains Member Function in std::string

The `std::string::contains()` member function was introduced in C++23, not C++20. It checks whether a string contains the specified substring or character.

```
#include <iostream>
#include <string>

int main() {
    const std::string text = "modern cpp";

    std::cout << std::boolalpha;
    std::cout << text.contains("cpp") << '\n';
}
```

For code that must compile with C++20, you can use `find()` instead:

```
#include <iostream>
#include <string>

int main() {
    const std::string text = "modern cpp";

    const bool found = text.find("cpp") != std::string::npos;

    std::cout << std::boolalpha << found << '\n';
}
```

### The std::ranges::contains Algorithm

The `std::ranges::contains()` algorithm was also standardized in C++23. It checks whether a value exists in a range and, unlike the member function provided by associative containers, is not limited to `set` and `map`.

```
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    const std::vector<int> values{10, 20, 30};

    std::cout << std::boolalpha;
    std::cout << std::ranges::contains(values, 20) << '\n';
}
```

Therefore, when using `std::ranges::contains()` or `std::string::contains()`, you must consider the C++23 or later language mode and the level of support provided by the standard library implementation.

## Compiler Compatibility and Standard Configuration

To use the `contains()` member function in associative containers, both the compiler and the standard library implementation must support C++20. Enabling C++20 mode in the compiler is necessary; however, complete support also depends on the version of the standard library.

For example, with GCC, you can typically use the following option:

```
g++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o app
```

With Clang, a similar standard option is available:

```
clang++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o app
```

With MSVC, you can enable C++20 mode using the appropriate option:

```
cl /std:c++20 /EHsc main.cpp
```

These options specify the language mode, but they do not guarantee that every standard library feature has been implemented in every installed version. For real-world projects, you should also check the versions of the compiler and standard library.

## Design Considerations and Choosing the Right Approach

The choice between `contains()`, `find()`, and search algorithms should depend on what you intend to do after the search.

| Application requirement                                       | Recommended approach                   |
| ------------------------------------------------------------- | -------------------------------------- |
| Only checking whether a key exists in a `set` or `map`        | `contains()`                           |
| Checking whether a key exists and then accessing the element  | `find()`                               |
| Searching for a value in a `vector` or general range in C++20 | `std::find()` or `std::ranges::find()` |
| Checking for a substring in `std::string` with C++20          | `find()`                               |
| Checking for a substring in `std::string` with C++23          | `std::string::contains()`              |
| Checking for a value in a range with C++23                    | `std::ranges::contains()`              |

When designing APIs, a function that only needs a Boolean answer should preferably use `contains()`. However, if the found element will subsequently be used, it is better to retain the iterator returned by `find()`. This avoids duplicate searches and makes the code's intent clearer.

## Summary

The `contains()` member function in C++20 provides a direct and readable way to check whether a key exists in associative containers. This functionality is available in `std::set`, `std::map`, their `multiset` and `multimap` variants, and their unordered counterparts. It returns a value of type `bool`.

The most important practical points are:

- When checking only whether a key exists in an associative container, `contains()` is generally more readable than the `find() != end()` pattern.
- If you need the matching element, choose `find()` to avoid performing the search again.
- Search complexity is `O(log n)` for ordered containers and, on average, `O(1)` for unordered containers, with a worst-case complexity of `O(n)`.
- Heterogeneous lookup can be used through transparent comparators, hash functions, and equality predicates.
- `std::vector` does not provide a `contains()` member function in C++20.
- `std::string::contains()` and `std::ranges::contains()` are C++23 features and should not be confused with the associative container member function introduced in C++20.

Ultimately, `contains()` is a useful tool for clearly expressing membership checks in code, provided that the container type, the need to access the matching element, and the actual search costs are taken into account during design.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>