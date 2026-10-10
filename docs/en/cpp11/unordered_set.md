<div align="center">

[🇺🇸 English](./unordered_set.md) | [🇮🇷 فارسی](../../en/cpp11/unordered_set.md)

</div>

---

# A Comprehensive Guide to `std::unordered_set` in C++

## Introduction to the Unordered Hash Set

`std::unordered_set` is an STL container introduced in C++11 for storing a collection of unique values. It uses a hash table to provide average constant-time complexity, \\(O(1)\\), for insertion, lookup, and deletion.

Unlike `std::set`, which maintains its elements in a sorted order, `std::unordered_set` does not guarantee any particular iteration order. Instead, its primary focus is on fast lookup and membership testing.

For example, this container can be a suitable choice for storing processed identifiers, unique words, user IDs, or any collection of values that must not contain duplicates.

```
#include <iostream>
#include <string>
#include <unordered_set>

int main() {
    std::unordered_set<std::string> languages;

    languages.insert("C++");
    languages.insert("Python");
    languages.insert("Rust");
    languages.insert("C++");

    std::cout << "Number of unique languages: "
              << languages.size() << '\n';

    for (const auto& language : languages) {
        std::cout << language << '\n';
    }
}
```

In this example, the string `"C++"` is stored only once because `std::unordered_set` does not allow duplicate elements.

An important detail is that the order in which the elements are printed is unspecified. It may differ between executions, standard library implementations, or changes to the container's internal structure.

## Core Concepts and Characteristics

### Fundamental Properties

The most important characteristics of `std::unordered_set` are:

- Unique elements: Each element is stored only once according to the container's `KeyEqual` equivalence relation.
- Fast lookup: Lookup has average constant-time complexity, \\(O(1)\\).
- No guaranteed iteration order: Elements are not maintained in ascending order or insertion order.
- Automatic memory management: Memory allocation and deallocation are managed by the container and its allocator.
- Support for custom types: User-defined types can be stored by providing a suitable hash function and equality predicate.
- Iterator support: Elements can be traversed using standard iterators.
- No direct index-based access: Unlike `std::vector`, this container does not provide indexed access.

### Uniqueness Does Not Imply Ordering

In `std::unordered_set`, unique elements are not necessarily ordered. For example, the following collection of integers:

```
std::unordered_set<int> values = {40, 10, 30, 20};
```

may be traversed in an order different from both the insertion order and the numerical order.

If elements must remain sorted, `std::set` is generally a more appropriate choice.

If insertion order must be preserved, use a different data structure or combine multiple containers.

## Comparing `std::unordered_set` and `std::set`

Both containers store unique elements, but their internal mechanisms and performance characteristics differ.

| Property                          | `std::unordered_set` | `std::set`                       |
| --------------------------------- | -------------------- | -------------------------------- |
| Typical internal structure        | Hash table           | Balanced search tree             |
| Element order                     | No guaranteed order  | Sorted according to a comparator |
| Average lookup complexity         | \\(O(1)\\)           | \\(O(\log n)\\)                  |
| Worst-case lookup complexity      | \\(O(n)\\)           | \\(O(\log n)\\)                  |
| Insertion and deletion            | Average \\(O(1)\\)   | \\(O(\log n)\\)                  |
| Supports `lower_bound`            | No                   | Yes                              |
| Requires a hash function          | Yes                  | No                               |
| Requires an ordering comparator   | No                   | Yes                              |
| Sensitivity to hash quality       | High                 | None                             |
| Supports ordered range operations | No                   | Yes                              |

The complexity figures describe the expected behavior and standard complexity guarantees. The actual internal implementation may vary between standard library implementations.

### When Is `std::unordered_set` Preferable?

This container is generally appropriate when:

- The primary goal is to test whether a value belongs to a collection.
- Element ordering is irrelevant.
- Insertion, deletion, and lookup are frequent operations.
- A suitable hash function is available for the key type.
- The additional memory required by a hash table is acceptable.

### When Is `std::set` a Better Choice?

`std::set` may be preferable under the following conditions:

- Elements must always remain sorted.
- You need access to the smallest or largest element.
- Operations such as `lower_bound` and `upper_bound` are required.
- Ordered traversal or range-based processing is important.
- You need logarithmic worst-case complexity for the primary operations.

Actual performance should still be measured using representative data and access patterns. Cache locality, hashing costs, and memory allocation can change the results in practice.

## Creating and Initializing a Container

### Creating an Empty Set

To use `std::unordered_set`, include its header:

```
#include <unordered_set>
```

You can then define a set of integers:

```
std::unordered_set<int> numbers;
```

The set is initially empty, and elements can be added using insertion operations.

### Initializing with an Element List

C++11 supports initialization using an initializer list:

```
std::unordered_set<int> numbers = {1, 2, 3, 4, 5};
```

Direct-list initialization is also supported:

```
std::unordered_set<int> numbers{1, 2, 3, 4, 5};
```

If duplicate values are provided, only one instance of each value is stored:

```
std::unordered_set<int> numbers = {1, 2, 2, 3, 3, 3};
```

After initialization, `numbers.size()` is equal to `3`.

### Initializing from an Iterator Range

You can initialize a set from elements in another container:

```
#include <iostream>
#include <unordered_set>
#include <vector>

int main() {
    std::vector<int> values = {1, 2, 3, 2, 4, 1};

    std::unordered_set<int> uniqueValues(
        values.begin(),
        values.end()
    );

    std::cout << uniqueValues.size() << '\n';
}
```

This technique is useful for removing duplicate values from a collection.

Building a unique set this way does not preserve the original element order.

### Constructing with an Initial Bucket Count

You can specify the initial number of buckets when constructing the set:

```
std::unordered_set<int> numbers(100);
```

The argument `100` requests an initial bucket count; it does not specify the number of elements the container can store without limitation.

This distinction matters because the number of buckets differs from the number of elements, and the container can resize itself as it grows.

## Inserting Elements

### Using `insert`

The `insert` function adds an element to the set:

```
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values;

    values.insert(10);
    values.insert(20);
    values.insert(30);
    values.insert(20);

    std::cout << values.size() << '\n';
}
```

The second insertion of `20` does not create another element.

### Checking the Insertion Result

In C++11, the single-element overload of `insert` returns a `std::pair` containing an iterator and a Boolean value.

The first member identifies the inserted element or the equivalent element already present. The second indicates whether insertion actually took place.

```
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20};

    auto first = values.insert(30);
    auto second = values.insert(20);

    std::cout << std::boolalpha;
    std::cout << first.second << '\n';
    std::cout << second.second << '\n';
}
```

Output:

```
true
false
```

This capability is useful for detecting duplicate identifiers or preventing a value from being processed more than once.

### Using `emplace`

The `emplace` function constructs an element from the supplied arguments at an appropriate location within the container:

```
#include <iostream>
#include <string>
#include <unordered_set>

int main() {
    std::unordered_set<std::string> names;

    auto result = names.emplace(10, 'A');

    std::cout << result.first->size() << '\n';
    std::cout << result.second << '\n';
}
```

In this example, the string constructor creates a string containing ten `A` characters.

Using `emplace` is not necessarily faster than using `insert`. Its benefits depend on the element type, the available constructors, and how the object would otherwise be copied, moved, or constructed.

### Inserting Multiple Elements

You can insert a range of elements using the range overload of `insert`:

```
#include <unordered_set>
#include <vector>

int main() {
    std::unordered_set<int> values = {1, 2};

    std::vector<int> additional = {2, 3, 4, 5};

    values.insert(additional.begin(), additional.end());
}
```

New elements are inserted, while duplicates are ignored.

## Membership Testing and Lookup

### Using `find`

The `find` function searches for an element. If the element does not exist, it returns `end()`.

```
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto it = values.find(20);

    if (it != values.end()) {
        std::cout << "Found: " << *it << '\n';
    }
}
```

This approach is useful when you need both to check whether an element exists and to obtain an iterator to it.

### Using `contains` in Newer Standards

The `contains` function was added to `std::unordered_set` in C++20. It returns a Boolean indicating whether the specified element exists.

```
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    bool exists = values.contains(20);
}
```

This API is not available in C++11. If your project must remain compatible with C++11, use `find` or `count` instead.

### Using `count`

The `count` function returns the number of elements equivalent to the specified key:

```
std::unordered_set<int> values = {10, 20, 30};

if (values.count(20) != 0) {
    // The element exists.
}
```

Because elements in `std::unordered_set` are unique, `count` can return only `0` or `1`.

For readability, comparing `find` with `end()` is often a good choice in C++11 code. However, using `count` is also valid.

### Looking Up Keys Using a Different Type

In C++11, the arguments to `find`, `count`, and `equal_range` must be compatible with the key type. The original standard API does not provide general heterogeneous lookup.

For example, if a set stores `std::string` values and you want to search using a `const char*`, constructing a temporary `std::string` may be necessary.

```
#include <string>
#include <unordered_set>

int main() {
    std::unordered_set<std::string> words = {
        "apple",
        "orange",
        "banana"
    };

    const char* query = "apple";

    auto it = words.find(std::string(query));
}
```

Heterogeneous lookup using transparent hash and equality functors was expanded in later standards. C++20 standardized support for heterogeneous lookup operations in `std::unordered_set`, provided the hash and equality types support the required transparent operations.

## Removing Elements and Clearing the Container

### Removing an Element with `erase`

The `erase` function can remove an element by key:

```
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    std::size_t removed = values.erase(20);
}
```

If the element exists, the return value is `1`; otherwise, it is `0`.

### Removing an Element by Iterator

If you already have an iterator to an element, you can remove that element directly:

```
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto it = values.find(20);

    if (it != values.end()) {
        values.erase(it);
    }
}
```

In C++11, the iterator overload of `erase` returns an iterator to the next element. This makes it useful for safely removing elements while traversing the container.

### Removing Elements During Iteration

The following pattern safely removes even numbers from a set:

```
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {
        1, 2, 3, 4, 5, 6
    };

    for (auto it = values.begin(); it != values.end();) {
        if (*it % 2 == 0) {
            it = values.erase(it);
        } else {
            ++it;
        }
    }
}
```

The important point is that after an element has been erased, you must not use the iterator that referred to it.

You should also avoid assuming that the same `erase` pattern applies to every container without checking its iterator invalidation rules.

### Removing All Elements

The `clear` function removes all elements from the set:

```
std::unordered_set<int> values = {1, 2, 3};

values.clear();
```

After this call, the container's size is zero. However, you should not assume that its allocated memory or bucket count must also become zero.

If your goal is to release excess memory, you must also consider bucket management and the container's allocation strategy.

## The Internal Structure of a Hash Table

### Understanding the Hash Function

`std::unordered_set` uses a hash function to determine where a key should be stored within its internal hash table.

The function converts a key into a hash value of type `std::size_t`. The container then uses this value and the number of buckets to select an appropriate bucket for the element.

At a conceptual level, lookup works as follows:

- Compute the hash of the key.
- Determine the corresponding bucket.
- Examine the elements in that bucket using the equality predicate.
- Return the matching element or report that it does not exist.

This is a conceptual model; the standard does not specify the exact internal implementation of the hash table.

### Understanding Buckets

A hash table consists of a collection of buckets. Multiple elements may occupy the same bucket, so hash collisions are a normal occurrence.

You can inspect the number of buckets using `bucket_count`:

```
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {
        10, 20, 30, 40, 50
    };

    std::cout << values.size() << '\n';
    std::cout << values.bucket_count() << '\n';
    std::cout << values.load_factor() << '\n';
}
```

The exact bucket count and load factor in this example depend on the container's state and the standard library implementation.

### Hash Collisions and Their Performance Impact

When several keys occupy the same bucket, the container must use the equality predicate to identify the correct key.

If the hash function has poor distribution and many elements accumulate in the same bucket, lookup performance can deviate significantly from the expected average behavior.

In the worst case, lookup, insertion, and deletion can have linear complexity, \\(O(n)\\).

For this reason, choosing an appropriate hash function for user-defined types is important.

## Time Complexity and Performance Characteristics

### Complexity of Common Operations

The following table summarizes the typical complexity and important standard guarantees:

| Operation    | Average complexity | Worst-case complexity |
| ------------ | ------------------ | --------------------- |
| `find`       | \\(O(1)\\)         | \\(O(n)\\)            |
| `insert`     | \\(O(1)\\)         | \\(O(n)\\)            |
| `erase(key)` | \\(O(1)\\)         | \\(O(n)\\)            |
| `count`      | \\(O(1)\\)         | \\(O(n)\\)            |
| `size`       | \\(O(1)\\)         | \\(O(1)\\)            |
| `clear`      | \\(O(n)\\)         | \\(O(n)\\)            |

This table covers common container operations. For operations involving multiple input elements, ranges, or groups of erased elements, the exact complexity also depends on the number of elements processed.

### Why Average Complexity Matters

Average \\(O(1)\\) complexity does not mean that every operation takes exactly constant time. This behavior depends on suitable hash distribution and typical usage conditions.

Excessive collisions or rehashing can increase the cost of an operation.

Therefore, in systems with strict response-time requirements, you should benchmark representative data distributions, lookup patterns, and memory behavior.

### Memory Overhead

Compared with some other containers, `std::unordered_set` typically requires additional memory for buckets and the internal hash-table structure.

This overhead can be significant for large collections or collections containing small elements.

If memory is limited, or if the elements are very small and numerous, it may be useful to compare `std::unordered_set` with `std::set`, a sorted `std::vector`, or a specialized data structure.

## Capacity Management and Rehashing

### Understanding the Load Factor

The load factor is the ratio of the number of elements to the number of buckets:

\\[ \text{load factor} = \frac{\text{size}}{\text{bucket count}} \\]

The `load_factor` function returns the current ratio.

```
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values;

    for (int i = 0; i < 1000; ++i) {
        values.insert(i);
    }

    std::cout << values.load_factor() << '\n';
    std::cout << values.max_load_factor() << '\n';
}
```

A lower load factor can reduce the likelihood of excessive collisions, but it generally comes at the cost of increased memory consumption.

### Configuring `max_load_factor`

The `max_load_factor` function retrieves or sets the maximum load factor used by the container.

```
#include <unordered_set>

int main() {
    std::unordered_set<int> values;

    values.max_load_factor(0.7f);

    for (int i = 0; i < 1000; ++i) {
        values.insert(i);
    }
}
```

The value `0.7f` is only an example and is not necessarily optimal for every application.

Changing the maximum load factor may cause a subsequent operation to trigger rehashing. For reliable performance tuning, measure this setting alongside element count, memory consumption, and operation timings.

### Reserving Capacity with `reserve`

If you know approximately how many elements the container will hold, using `reserve` can reduce the number of rehash operations:

```
#include <unordered_set>

int main() {
    std::unordered_set<int> values;

    values.reserve(10000);

    for (int i = 0; i < 10000; ++i) {
        values.insert(i);
    }
}
```

The `reserve` function adjusts the bucket capacity based on the expected number of elements and `max_load_factor`. It does not guarantee that the container will have exactly that number of buckets.

For applications that insert many elements incrementally, reserving capacity can reduce rehashing overhead.

### Directly Configuring the Bucket Count with `rehash`

The `rehash` function requests a minimum number of buckets:

```
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {1, 2, 3};

    values.rehash(100);
}
```

The resulting bucket count may be greater than the requested value. The final count also depends on the load-factor requirements and the implementation.

The key distinction is that `reserve` works in terms of the expected number of elements, whereas `rehash` operates directly on the number of buckets.

### The Effect of Rehashing on Iterators and References

Rehashing invalidates existing iterators into `std::unordered_set`. Therefore, you must not use iterators obtained before a rehash operation afterward.

However, references and pointers to elements are not invalidated merely because rehashing occurs. This distinction is important when working with hash containers.

Insertion can also invalidate existing iterators if it triggers rehashing. If you need to retain iterators across insertions, account for the possibility of rehashing in your design.

Erasing an element invalidates the iterator referring to that element, but it does not invalidate iterators to other elements merely because of that erasure.

## Traversing Elements and Using Iterators

### Using a Range-Based `for` Loop

The simplest way to traverse the set is with a range-based `for` loop:

```
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    for (const auto& value : values) {
        std::cout << value << '\n';
    }
}
```

Using `const auto&` avoids unnecessary copies of the elements.

Because `std::unordered_set` is designed to store unique keys, you cannot modify an element through an iterator. Directly changing a key could make its hash inconsistent with its location in the container.

### Traversing with Iterators

When more precise control is required, you can use iterators:

```
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    for (auto it = values.begin(); it != values.end(); ++it) {
        std::cout << *it << '\n';
    }
}
```

The `begin` and `end` functions return iterators to the beginning and end of the range, respectively.

The `cbegin` and `cend` variants are also available for read-only traversal.

### No Guaranteed Iteration Order

Even if you insert elements in sorted order, you cannot assume that iteration will follow that order.

If deterministic output is required, copy the elements into another container, such as `std::vector`, and sort them.

```
#include <algorithm>
#include <iostream>
#include <unordered_set>
#include <vector>

int main() {
    std::unordered_set<int> values = {40, 10, 30, 20};

    std::vector<int> sortedValues(
        values.begin(),
        values.end()
    );

    std::sort(sortedValues.begin(), sortedValues.end());

    for (int value : sortedValues) {
        std::cout << value << ' ';
    }
}
```

This approach is useful when the primary data structure must support fast lookup but the final output requires a specific order.

## Custom Hash Functions and Equality Predicates

### Using the Default Hash Function

The standard library provides suitable specializations of `std::hash` for many common types.

For example, `std::unordered_set<int>` can normally be used without defining a custom hash function.

```
std::unordered_set<int> identifiers;
std::unordered_set<std::string> names;
```

For user-defined types, you must ensure that the hash and equality operations required by the container are available.

### Defining a Hash Function for a Custom Type

Suppose we want to create a set of two-dimensional coordinates:

```
#include <cstddef>
#include <functional>
#include <unordered_set>

struct Point {
    int x;
    int y;

    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

struct PointHash {
    std::size_t operator()(const Point& point) const {
        const std::size_t hx = std::hash<int>{}(point.x);
        const std::size_t hy = std::hash<int>{}(point.y);

        return hx ^ (hy + 0x9e3779b9U + (hx << 6) + (hx >> 2));
    }
};

int main() {
    std::unordered_set<Point, PointHash> points;

    points.insert({1, 2});
    points.insert({3, 4});
    points.insert({1, 2});
}
```

In this example, points with identical coordinates are considered equivalent, so only one instance of each point remains in the set.

The hash-combination formula is a simple example and is not necessarily the best choice for every application. In performance-sensitive or security-sensitive projects, evaluate hash distribution and behavior using representative data.

### The Hash and Equality Contract

One of the most important rules of `std::unordered_set` is:

If two keys are equivalent according to the equality predicate, their hash values must be identical.

Mathematically:

\\[ \operatorname{KeyEqual}(a,b)=\text{true} \Rightarrow \operatorname{Hash}(a)=\operatorname{Hash}(b) \\]

The reverse implication is not required. Two keys with the same hash value are not necessarily equivalent.

Violating this contract can cause lookup and uniqueness behavior to become inconsistent with the program's expectations.

### Defining a Custom Equality Predicate

You can pass custom hash and equality types to the container independently:

```
#include <cstddef>
#include <functional>
#include <unordered_set>

struct CaseInsensitiveHash {
    std::size_t operator()(int value) const {
        return std::hash<int>{}(value);
    }
};

struct CaseInsensitiveEqual {
    bool operator()(int lhs, int rhs) const {
        return lhs == rhs;
    }
};

int main() {
    std::unordered_set<
        int,
        CaseInsensitiveHash,
        CaseInsensitiveEqual
    > values;

    values.insert(10);
    values.insert(20);
}
```

The names of these types are chosen only to illustrate the template structure. In this example, they operate on integers and implement ordinary equality.

For case-insensitive strings, both the hash function and equality predicate must follow the same policy. If the equality predicate considers two strings equivalent but their hash values differ, the container's contract is violated.

Unicode, normalization, and locale rules should not be treated as equivalent to simply converting ASCII characters to lowercase.

## Specializing `std::hash` for a Custom Type

One way to use a custom hash function is to define a separate functor, as shown above. Another approach is to specialize `std::hash` for a user-defined type.

```
#include <cstddef>
#include <functional>
#include <unordered_set>

struct UserId {
    int value;

    bool operator==(const UserId& other) const {
        return value == other.value;
    }
};

namespace std {
    template <>
    struct hash<UserId> {
        std::size_t operator()(const UserId& id) const {
            return std::hash<int>{}(id.value);
        }
    };
}

int main() {
    std::unordered_set<UserId> users;

    users.insert({1});
    users.insert({2});
}
```

Specializing `std::hash` for a user-defined type is valid when the standard's requirements are followed. However, arbitrary specializations for standard-library types or modifications to the standard library's defined behavior are not permitted.

Using a separate functor usually provides greater flexibility because multiple hash policies can be defined for the same type.

## Using Complex Structures as Keys

### Using `std::pair`

For a set of pairs, you must provide a suitable hash function because C++11 does not provide a general standard specialization of `std::hash<std::pair<...>>`.

```
#include <cstddef>
#include <functional>
#include <unordered_set>
#include <utility>

struct PairHash {
    std::size_t operator()(
        const std::pair<int, int>& value
    ) const {
        const std::size_t h1 = std::hash<int>{}(value.first);
        const std::size_t h2 = std::hash<int>{}(value.second);

        return h1 ^ (h2 + 0x9e3779b9U + (h1 << 6) + (h1 >> 2));
    }
};

int main() {
    std::unordered_set<
        std::pair<int, int>,
        PairHash
    > edges;

    edges.insert({1, 2});
    edges.insert({2, 3});
    edges.insert({1, 2});
}
```

This technique is useful for storing unique graph edges or coordinate pairs.

### Using Structures with Multiple Fields

If an object contains several fields, you must determine which fields define its logical identity as a key.

For example, if a user's ID alone determines uniqueness, the hash and equality operations should both be based on that ID rather than using the ID for hashing and the ID plus name for equality.

When designing composite keys, consistency between hashing and equality is more important than the complexity of the hash-combination formula.

## Limitations on Modifying Elements and Mutable Keys

### Why Are Keys Read-Only?

In `std::unordered_set`, an ordinary iterator refers to an element of type `const value_type`. This design prevents direct modification of a key that might change its hash and leave it in the wrong bucket.

For example, the following code is invalid:

```
std::unordered_set<int> values = {10, 20, 30};

auto it = values.find(20);

// *it = 25; // Invalid: elements are immutable through iterators.
```

If you want to change a key, you must erase it and insert a new value:

```
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto it = values.find(20);

    if (it != values.end()) {
        values.erase(it);
        values.insert(25);
    }
}
```

### Extracting and Modifying a Key with Node Handles in C++17

C++17 introduced node handles, which allow you to extract an element, modify its key, and insert it again.

```
#include <unordered_set>
#include <utility>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto node = values.extract(20);

    if (!node.empty()) {
        node.value() = 25;
        values.insert(std::move(node));
    }
}
```

This capability is not available in C++11. Projects targeting C++11 must use the erase-and-reinsert approach.

If the new key already exists in the set, insertion of the node handle may fail. In that case, check the insertion result and manage the remaining node if necessary.

## Object Lifetime and Iterator Invalidation

### Operations That Invalidate Iterators

Iterator validity in `std::unordered_set` must be handled carefully.

| Operation                                                          | Effect on iterators                                |
| ------------------------------------------------------------------ | -------------------------------------------------- |
| Insertion without rehashing                                        | Existing iterators remain valid                    |
| Insertion that causes rehashing                                    | Existing iterators are invalidated                 |
| Calling `rehash`                                                   | Existing iterators are invalidated                 |
| Erasing an element                                                 | The iterator to that element is invalidated        |
| Calling `clear`                                                    | Iterators to the previous elements are invalidated |
| Illegally changing hash or equality behavior while keys are stored | Can violate the container's requirements           |

References and pointers to elements are not invalidated merely because rehashing changes the internal bucket structure. However, erasing an element ends that object's lifetime and invalidates references and pointers to it.

To avoid errors, retain an iterator only while you can account for operations that might invalidate it.

### An Incorrect Example Using an Old Iterator

The following code can cause problems:

```
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto it = values.begin();

    values.rehash(values.bucket_count() + 100);

    // Using the old iterator is invalid.
    // int value = *it;
}
```

After rehashing, you must obtain a new iterator from the container.

## Differences Between `clear`, `rehash`, and `swap`

### Clearing Elements Versus Reducing Capacity

The `clear` function removes elements but does not necessarily release the bucket storage.

The `rehash` function changes the bucket structure and can be used to request a smaller or larger bucket count. However, the resulting count must satisfy the container's requirements.

The `swap` function exchanges two containers:

```
#include <unordered_set>

int main() {
    std::unordered_set<int> first = {1, 2, 3};
    std::unordered_set<int> second = {10, 20};

    first.swap(second);
}
```

After `swap`, the two sets have exchanged their elements.

When using `swap`, account for allocator requirements and the standard's container-specific behavior. If you already hold iterators or references, check their validity according to the operation's rules.

## Using `equal_range`

The `equal_range` function returns a range containing the elements equivalent to a specified key.

Because `std::unordered_set` stores unique elements, the returned range contains either zero or one element.

```
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto range = values.equal_range(20);

    if (range.first != range.second) {
        int value = *range.first;
    }
}
```

This function is less commonly used than `find` in typical `std::unordered_set` code, but it can be useful in generic code that works with different associative containers.

## Inspecting Buckets and Diagnosing Performance

### Inspecting the Bucket for an Element

The `bucket` function returns the bucket index associated with a key:

```
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30, 40};

    for (int value : values) {
        std::cout << value << ": "
                  << values.bucket(value) << '\n';
    }
}
```

Bucket indices are implementation details and should not be used as part of an application's business logic.

### Counting Elements in Each Bucket

The `bucket_size` function returns the number of elements in a particular bucket:

```
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {
        10, 20, 30, 40, 50
    };

    for (std::size_t i = 0; i < values.bucket_count(); ++i) {
        std::cout << "Bucket " << i << ": "
                  << values.bucket_size(i) << '\n';
    }
}
```

These tools are useful for diagnosing poor hash distribution during performance testing.

However, application logic should not depend on bucket indices or the order of elements within a bucket.

## Security Considerations for Hash Functions

### Weak Hash Functions and Adversarial Input

In services that accept user-provided input, a weak or predictable hash function can increase the risk of excessive collisions and severe performance degradation.

Under certain conditions, an attacker may construct inputs that cause many elements to accumulate in a small number of buckets. As a result, operations that are usually fast may approach linear time.

To reduce this risk:

- Use an appropriate hash function for the key type.
- Limit the size and number of untrusted inputs.
- For security-sensitive data, choose a hashing strategy appropriate to the threat model.
- Test the container with adversarial or highly skewed data.
- Do not assume that the default hash function in every standard library implementation is resistant to hash-flooding attacks.

If attack-resistant hashing is required, select an algorithm based on the security requirements, support for randomized keys, and threat model.

## Removing Duplicate Values with `std::unordered_set`

One common application of this container is deduplicating data.

```
#include <iostream>
#include <unordered_set>
#include <vector>

int main() {
    std::vector<int> input = {
        5, 2, 5, 3, 2, 7, 3, 8
    };

    std::unordered_set<int> unique(
        input.begin(),
        input.end()
    );

    std::cout << unique.size() << '\n';
}
```

This technique is useful when duplicates must be removed and element order does not matter.

If the original order of the first occurrence of each element must be preserved, constructing an `unordered_set` directly is insufficient. In that case, use a set for membership testing and a `std::vector` to maintain the output order.

```
#include <iostream>
#include <unordered_set>
#include <vector>

int main() {
    std::vector<int> input = {
        5, 2, 5, 3, 2, 7, 3, 8
    };

    std::unordered_set<int> seen;
    std::vector<int> uniqueInOrder;

    for (int value : input) {
        if (seen.insert(value).second) {
            uniqueInOrder.push_back(value);
        }
    }

    for (int value : uniqueInOrder) {
        std::cout << value << ' ';
    }
}
```

This approach preserves the first occurrence of each value while providing average constant-time membership checks.

## Practical Applications in Real Projects

### Preventing Duplicate Processing of Identifiers

When processing large files, messages, or network requests, you may need to ensure that each identifier is processed only once.

Keeping previously observed identifiers in a `std::unordered_set` allows you to check whether an identifier has already been encountered before processing it.

### Detecting Unique Words

For text analysis, counting distinct words or checking whether a word exists, a set of strings can be useful.

However, real-world text processing also requires an explicit policy for case sensitivity, Unicode, whitespace, and normalization.

### Implementing a Set of Permissions

In an authorization system, a collection of permitted capabilities or identifiers can be stored in a set. Membership checks are generally straightforward and fast.

However, container performance alone is not sufficient for security-sensitive design. Validation policies, concurrency, and data updates also matter.

### Preventing Repeated Visits to Graph Nodes

In graph traversal algorithms such as BFS and DFS, a set of visited node identifiers can prevent nodes from being processed repeatedly.

In this application, `std::unordered_set` is particularly useful when node identifiers are sparse or nonsequential.

## Comparing `std::unordered_set` with `std::vector` and Other Alternatives

### Comparing with `std::vector`

Checking whether a value exists in a `std::vector` generally requires linear search unless the vector is sorted or another strategy is used.

By contrast, `std::unordered_set` is designed for average constant-time membership checks.

However, for small collections, `std::vector` may be faster because it uses contiguous storage and benefits from cache locality.

The choice between these structures should therefore depend on collection size, lookup frequency, insertion and deletion costs, and benchmark results.

### Comparing with `std::unordered_map`

If each key must be associated with a separate value, `std::unordered_map` is usually the more appropriate choice.

By contrast, `std::unordered_set` stores only keys and is suitable for membership checks or collections of unique values.

For example, if only user IDs matter, a set is sufficient. If each ID must be associated with a name or status, a map is generally more appropriate.

### Comparing with `std::unordered_multiset`

Another container, `std::unordered_multiset`, allows multiple equivalent elements.

If duplicate values must be preserved, `std::unordered_multiset` is appropriate. If each value must appear only once, `std::unordered_set` is the correct choice.

## Differences Between C++ Standard Versions

### Features Available in C++11

`std::unordered_set` was introduced in C++11. Its core capabilities include insertion, deletion, lookup, iterators, custom hash functions, bucket management, and load-factor configuration.

### Features Added in C++17

C++17 introduced features such as node handles and operations for transferring nodes between compatible containers.

These capabilities are useful for extracting an element, modifying its key, and inserting it again.

### Features Added in C++20

C++20 added the `contains` function for membership testing. It also standardized heterogeneous lookup for the primary lookup operations when transparent hash and equality functors are used.

For example, with an appropriately designed `std::unordered_set<std::string>`, a compatible implementation can search using `std::string_view` without constructing a temporary `std::string`.

```
#include <functional>
#include <string>
#include <string_view>
#include <unordered_set>

struct TransparentHash {
    using is_transparent = void;

    std::size_t operator()(std::string_view value) const {
        return std::hash<std::string_view>{}(value);
    }
};

struct TransparentEqual {
    using is_transparent = void;

    bool operator()(
        std::string_view lhs,
        std::string_view rhs
    ) const {
        return lhs == rhs;
    }
};

int main() {
    std::unordered_set<
        std::string,
        TransparentHash,
        TransparentEqual
    > words = {"apple", "banana"};

    std::string_view query = "apple";

    auto it = words.find(query);
}
```

This example requires C++20. Defining `is_transparent` alone is not sufficient: the hash and equality functors must support the required argument types, and their consistency requirements must be preserved.

## Common Mistakes and How to Avoid Them

### Assuming a Stable Element Order

Do not rely on the iteration order of `std::unordered_set` for application logic or deterministic tests.

If a specific order is required, use an ordered container or sort the output.

### Using an Iterator After Rehashing

Any operation that may trigger rehashing must be considered when retaining iterators.

After rehashing, existing iterators are invalid and must be reacquired from the container.

### Defining an Inconsistent Hash and Equality Predicate

If two keys are equivalent according to the equality predicate, their hash values must be identical.

Violating this requirement can make the container behave inconsistently with the program's logical expectations.

### Modifying Key State

If hashing or equality depends on mutable external state, changing that state while a key remains in the container can violate the container's requirements.

Prefer hash and equality functions based on stable key properties.

### Assuming Constant Complexity in Every Situation

Average \\(O(1)\\) complexity does not guarantee constant time for every individual operation. Excessive collisions or rehashing can increase operation costs.

### Using `unordered_set` for Ordered Data

If an algorithm needs the smallest element, the largest element, or ordered ranges, `std::set` or a sorted container may be more appropriate.

### Using `emplace` Unnecessarily

`emplace` is not always faster than `insert`. If the element already exists as an object or constructing it has no particular cost, `insert` may be simpler and equally efficient.

## Best Practices for Professional Code

- Design custom hash and equality operations as a consistent pair.
- Consider using `reserve` when the expected element count is known.
- Pay attention to iterator validity during insertion, deletion, and rehashing.
- Avoid depending on iteration order or bucket indices.
- Measure memory usage and hash distribution in performance-sensitive applications.
- Consider resource limits and hash-flooding risks when processing untrusted input.
- If your project must remain compatible with C++11, do not use newer APIs such as `contains` or node handles.
- If keys must be modified, use erase-and-reinsert in C++11 or node handles in C++17.
- In multithreaded applications, manage concurrent access according to the standard library's thread-safety rules. Concurrent modifying operations must not run without appropriate synchronization.
- Benchmark with representative data and access patterns before choosing between `unordered_set`, `set`, and `vector`.

## Summary

`std::unordered_set` in C++11 is a suitable tool for storing unique elements and performing fast lookup, insertion, and deletion. It uses a hash table and provides average constant-time complexity, \\(O(1)\\), but does not guarantee element order and can exhibit linear-time behavior in the worst case.

Choosing this container depends on the application's requirements. If element order is irrelevant and membership checks are frequent, `std::unordered_set` can be an excellent choice. For ordered data, range-based operations, or requirements for logarithmic worst-case complexity, `std::set` is generally more appropriate.

Ultimately, professional and reliable use of this container requires understanding hash functions, equality contracts, capacity management, rehashing, iterator validity, and the differences between APIs introduced in various C++ standard versions.

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>