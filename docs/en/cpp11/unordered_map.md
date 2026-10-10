<div align="center">

[🇺🇸 English](./unordered_map.md) | [🇮🇷 فارسی](../../en/cpp11/unordered_map.md)

</div>

---

# A Comprehensive Guide to `std::unordered_map` in C++11

## Introduction

`std::unordered_map` is an important container in the C++ Standard Library designed to store and retrieve data by key. Introduced in C++11, this container provides fast access to values associated with keys.

For example, if we want to store user names and other information by their IDs, we can use `std::unordered_map`. Each key maps to at most one value, and keys must be unique.

The main characteristics of this container include:

- Average-case \\(O(1)\\) time complexity for lookup, insertion, and deletion.
- Uses a hash table to organize data.
- Does not guarantee element ordering.
- Supports custom key types through appropriate hash functions and equality predicates.
- Allows the initial capacity and load factor to be configured to control hash table behavior.
- Supports various key and value types, including user-defined types.

Despite these advantages, `std::unordered_map` is not always the best choice. Its suitability depends on the application's requirements, data access patterns, hash computation costs, and memory constraints.

## Introducing the Data Structure

### Mapping Keys to Values

The `std::unordered_map` type is defined in the standard `<unordered_map>` header and belongs to the `std` namespace.

Its general template declaration is as follows:

```
std::unordered_map<Key, T, Hash, KeyEqual, Allocator>
```

The template parameters are:

| Parameter   | Description                                        |
| ----------- | -------------------------------------------------- |
| `Key`       | The type of the keys                               |
| `T`         | The type of the stored values                      |
| `Hash`      | The hash function type used for keys               |
| `KeyEqual`  | The key equality comparison type                   |
| `Allocator` | The allocator used to allocate memory for elements |

The last three parameters are optional and do not need to be specified in typical use cases.

For example, the following code creates a mapping between numeric IDs and user names:

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<int, std::string> users;

    users[101] = "Alice";
    users[102] = "Bob";
    users[103] = "Charlie";

    std::cout << users[102] << '\n';
}
```

The output is:

```
Bob
```

In this example, the keys are of type `int`, and the values are of type `std::string`.

### Comparing `std::unordered_map` with `std::map`

Both containers map keys to values, but they organize their data differently.

`std::map` is typically implemented using a balanced tree and keeps elements ordered by key. In contrast, `std::unordered_map` uses a hash table and does not guarantee any particular iteration order.

| Feature                         | `std::unordered_map`  | `std::map`                          |
| ------------------------------- | --------------------- | ----------------------------------- |
| Typical underlying structure    | Hash table            | Balanced tree                       |
| Average lookup complexity       | \\(O(1)\\)            | \\(O(\log n)\\)                     |
| Worst-case lookup complexity    | \\(O(n)\\)            | \\(O(\log n)\\)                     |
| Key ordering                    | Not guaranteed        | Sorted                              |
| Direct range queries by key     | Not supported         | Supported                           |
| Requires a hash function        | Yes                   | No                                  |
| Requires an ordering comparison | No                    | Yes                                 |
| Typical use case                | Fast key-based lookup | Ordered traversal and range queries |

The exact internal implementation of these containers depends on the standard library implementation. The table summarizes their standard complexity guarantees and typical characteristics.

Rule of thumb: If key ordering is irrelevant and fast key-based lookup is the priority, `std::unordered_map` is a suitable choice. If you need sorted traversal, the minimum or maximum key, or range queries, `std::map` is generally more appropriate.

## How a Hash Table Works

### Understanding Hash Functions

A hash function converts a key into a numeric value of type `std::size_t`. The container uses this value to determine the element's likely location in the hash table.

For example, a hash function for integers can be defined as follows:

```
#include <cstddef>

struct IntegerHash {
    std::size_t operator()(int value) const noexcept {
        return static_cast<std::size_t>(value);
    }
};
```

In this example, the input value is converted directly to `std::size_t`. This function is suitable for illustrating the concept of hashing, but it is not necessarily the best choice for every key type or data distribution.

Standard hash functions for common types, including integral types and `std::string`, are generally available without requiring a custom implementation.

### Understanding Buckets

In a hash table, elements are organized into groups called buckets. The hash function helps determine which bucket is used, but the hash value is not necessarily the bucket index itself. The standard library implementation may use an additional internal mapping to determine the bucket.

Two different keys may map to the same bucket. This situation is called a collision.

Hash table implementations handle collisions using their own internal mechanisms. The precise collision-handling strategy and bucket organization depend on the standard library implementation.

### Understanding the Load Factor

The load factor is the ratio of the number of stored elements to the number of buckets:

\\[ \text{Load Factor}=\frac{\text{Size}}{\text{Bucket Count}} \\]

If a container has 100 elements and 200 buckets, its load factor is 0.5.

A higher load factor can increase the likelihood of collisions and the cost of lookup. For this reason, `std::unordered_map` provides mechanisms for controlling the load factor and the number of buckets.

### Understanding Rehashing

As the number of elements grows or the hash table's capacity requirements change, the container may perform a rehash. During this operation, elements are reorganized according to the new bucket structure.

Rehashing can be expensive because it may require memory allocation and processing many existing elements.

Consequently, if the approximate number of elements is known in advance, it is often beneficial to configure the required capacity before inserting a large amount of data.

## Creating and Initializing a Container

### Creating an Empty Container

The simplest way to create a container is:

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores;
}
```

In this example, the keys are of type `std::string`, and the values are of type `int`.

### Initializing with an Element List

C++11 supports initializing a container using an initializer list:

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87},
        {"Charlie", 91}
    };
}
```

Each element in this example consists of a key and its corresponding value.

If a key appears more than once in an initializer list, portable code should not rely on a particular value being selected from the duplicate entries. When input data may contain duplicate keys, define an explicit duplicate-handling policy or use controlled insertion.

### Creating a Container with an Initial Capacity

To reduce the likelihood of rehashing, you can reserve capacity for an approximate number of elements before inserting them:

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores;

    scores.reserve(1000);

    for (int i = 0; i < 1000; ++i) {
        scores.emplace("user_" + std::to_string(i), i);
    }
}
```

The `reserve()` function adjusts the number of buckets based on the requested number of elements and the current `max_load_factor()`.

It does not guarantee an exact bucket count. The standard library implementation may choose a different number.

Also, `reserve()` does not mean that memory for every string and stored object is allocated in advance. Allocations for the elements themselves and their dynamically managed data still depend on their types and sizes.

## Core Element Operations

### Accessing Elements with `operator[]`

The subscript operator, `[]`, provides access to the value associated with a key:

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> inventory;

    inventory["apple"] = 10;
    inventory["banana"] = 20;

    std::cout << inventory["apple"] << '\n';
}
```

If the key does not exist, `operator[]` inserts a new element with that key and value-initializes its mapped value.

For example, in a `std::unordered_map<std::string, int>`, a newly inserted key receives an initial value of zero.

This behavior is important when reading data. The following statement does not merely read a value; it also inserts an element if the key is missing:

```
int value = inventory["orange"];
```

If you only want to check whether a key exists or read its value, use `find()` or `at()` instead.

### Checked Access with `at()`

The `at()` function accesses an existing key without inserting a new element:

```
#include <iostream>
#include <stdexcept>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> inventory{
        {"apple", 10},
        {"banana", 20}
    };

    try {
        std::cout << inventory.at("apple") << '\n';
        std::cout << inventory.at("orange") << '\n';
    } catch (const std::out_of_range&) {
        std::cout << "Key not found\n";
    }
}
```

If the key does not exist, `at()` throws `std::out_of_range`.

This function is appropriate when a missing key should be treated as an error.

### Checking for a Key with `find()`

The `find()` function returns an iterator to the matching element. If the key does not exist, it returns `end()`.

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87}
    };

    auto it = scores.find("Bob");

    if (it != scores.end()) {
        std::cout << it->first << ": " << it->second << '\n';
    }
}
```

This approach is useful when you need to check whether a key exists and access the corresponding element or value.

### Checking for a Key with `contains()`

C++20 introduced `contains()` for associative and unordered associative containers.

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87}
    };

    if (scores.contains("Alice")) {
        // Key exists
    }
}
```

This function is not available in C++11. To maintain C++11 compatibility, use `find()`:

```
if (scores.find("Alice") != scores.end()) {
    // Key exists
}
```

### Inserting Elements with `insert()`

The `insert()` function attempts to insert a new element, provided that its key does not already exist.

```
#include <iostream>
#include <string>
#include <unordered_map>
#include <utility>

int main() {
    std::unordered_map<std::string, int> scores;

    auto result = scores.insert(std::make_pair("Alice", 95));

    if (result.second) {
        std::cout << "Inserted successfully\n";
    } else {
        std::cout << "Key already exists\n";
    }
}
```

In this case, `insert()` returns a `std::pair`:

- The `first` member is an iterator to the existing or newly inserted element.
- The `second` member indicates whether insertion took place.

If the key already exists, its previous value is not replaced.

### Inserting Elements with `emplace()`

The `emplace()` function allows an element to be constructed in place:

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores;

    scores.emplace("Alice", 95);
    scores.emplace("Bob", 87);
}
```

This function can avoid unnecessary temporary objects, but it is not guaranteed to be faster than `insert()` in every situation. Hash computation, duplicate-key checks, and memory allocation also contribute to the overall cost.

### Inserting or Updating with `insert_or_assign()`

The `insert_or_assign()` function was introduced in C++17. It inserts an element if the key does not exist; otherwise, it replaces the existing value.

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores;

    scores.insert_or_assign("Alice", 95);
    scores.insert_or_assign("Alice", 98);
}
```

In this example, the final value associated with `Alice` is 98.

In C++11, this behavior must be implemented by checking whether the key exists or by using `operator[]` with an appropriate policy.

### Conditional Insertion with `try_emplace()`

The `try_emplace()` function was introduced in C++17. It constructs the new mapped value only if the key does not already exist.

```
#include <memory>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, std::unique_ptr<int>> values;

    values.try_emplace("first", new int(42));
}
```

In this example, ownership is transferred through `std::unique_ptr`. However, the expression `new int(42)` is evaluated before the function is called. Therefore, this example does not avoid the allocation when the key already exists.

For more controlled construction, `std::make_unique()` can be used in C++14, but function arguments are still evaluated before the function is entered.

The main advantage of `try_emplace()` is that, when the key already exists, the supplied arguments are not forwarded to the mapped value's constructor.

## Updating and Removing Elements

### Updating an Existing Value

You can update a key's value using `operator[]`:

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95}
    };

    scores["Alice"] = 98;
}
```

In this example, the value associated with `Alice` is updated because the key already exists.

If the key may not exist, remember that `operator[]` can insert a new element.

### Removing an Element with `erase()`

The `erase()` function removes elements from the container:

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87},
        {"Charlie", 91}
    };

    scores.erase("Bob");
}
```

In C++11, `erase(key)` returns the number of elements removed. For a container with unique keys, this value is normally either zero or one.

C++20 introduced `erase_if()` for conditional removal from applicable standard containers. In C++11, conditional removal can be implemented using an appropriate iterator pattern.

### Removing Elements Conditionally with Iterators

The following code removes elements whose values are less than 90:

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87},
        {"Charlie", 91},
        {"David", 78}
    };

    for (auto it = scores.begin(); it != scores.end();) {
        if (it->second < 90) {
            it = scores.erase(it);
        } else {
            ++it;
        }
    }
}
```

In this pattern, the iterator returned by `erase()` is used to continue traversal. This prevents access to an erased iterator.

### Clearing All Elements

Use `clear()` to remove every element:

```
scores.clear();
```

This function removes the elements but does not necessarily reduce the number of buckets. If reducing memory usage is the goal, the implementation's behavior and the application's actual requirements should be considered separately.

## Iterating Over Elements

### Iterating with a Range-Based `for` Loop

A range-based `for` loop is usually the most readable way to iterate over all elements:

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87},
        {"Charlie", 91}
    };

    for (const auto& entry : scores) {
        std::cout << entry.first << ": " << entry.second << '\n';
    }
}
```

Each element in this container has a type similar to `std::pair<const Key, T>`. Consequently, the key cannot be modified through an iterator, although the mapped value can be modified when appropriate.

### Iterating with Iterators

For more precise control over traversal, use iterators:

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87}
    };

    for (auto it = scores.begin(); it != scores.end(); ++it) {
        std::cout << it->first << ": " << it->second << '\n';
    }
}
```

In this example, the iterator provides direct access to both the key and the mapped value.

### No Guaranteed Iteration Order

The iteration order of `std::unordered_map` is not guaranteed. Even if consecutive executions produce the same order, a program must not depend on that behavior.

The iteration order may change after insertions, deletions, or rehashing, and it may differ between standard library implementations or versions.

If sorted output is required, use `std::map` or copy the elements into another container and sort them explicitly.

## Managing Capacity and Performance

### Reserving Capacity with `reserve()`

The `reserve()` function prepares the container for a specified number of elements:

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> counts;

    counts.reserve(10000);

    for (int i = 0; i < 10000; ++i) {
        counts.emplace("item_" + std::to_string(i), i);
    }
}
```

When the final number of elements is approximately known, this can reduce the number of rehash operations.

However, `reserve()` alone does not guarantee fast performance. Hash function quality and key distribution also matter.

### Configuring the Load Factor with `max_load_factor()`

The `max_load_factor()` function sets the desired maximum load factor:

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> counts;

    counts.max_load_factor(0.7f);
    counts.reserve(10000);
}
```

In this example, the desired limit is set before capacity is reserved.

The order matters because `reserve()` uses the current maximum load factor when calculating the required capacity.

A smaller load factor can increase the number of buckets and memory consumption. Conversely, a larger load factor may reduce memory usage while increasing collision costs and lookup time.

The appropriate value depends on the implementation, key distribution, memory constraints, and performance requirements.

### Inspecting the Number of Buckets

The following functions can be used to inspect the hash table's state:

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> counts;

    counts.reserve(100);

    for (int i = 0; i < 100; ++i) {
        counts.emplace("key_" + std::to_string(i), i);
    }

    std::cout << "Size: " << counts.size() << '\n';
    std::cout << "Buckets: " << counts.bucket_count() << '\n';
    std::cout << "Load factor: " << counts.load_factor() << '\n';
    std::cout << "Maximum load factor: "
              << counts.max_load_factor() << '\n';
}
```

These functions are useful for analyzing container behavior, but the number of buckets should not be expected to match an exact value.

### Inspecting an Individual Bucket

The `bucket()` and `bucket_size()` functions can help examine how elements are distributed:

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> values{
        {"apple", 1},
        {"banana", 2},
        {"orange", 3}
    };

    auto index = values.bucket("apple");

    std::cout << "Bucket index: " << index << '\n';
    std::cout << "Bucket size: "
              << values.bucket_size(index) << '\n';
}
```

This functionality is useful for analyzing collisions and inspecting element distribution. However, the results depend on the implementation and the hash function in use.

## Designing Custom Hash Functions

### Defining a Hash Function for a Custom Type

To use a user-defined type as a key, you generally need to define a hash function and a suitable equality predicate.

Suppose we want to store user information using a composite identifier:

```
#include <cstddef>
#include <functional>
#include <string>
#include <unordered_map>

struct UserKey {
    int id;
    std::string region;

    bool operator==(const UserKey& other) const {
        return id == other.id && region == other.region;
    }
};

struct UserKeyHash {
    std::size_t operator()(const UserKey& key) const {
        std::size_t h1 = std::hash<int>{}(key.id);
        std::size_t h2 = std::hash<std::string>{}(key.region);

        return h1 ^ (h2 + 0x9e3779b9u + (h1 << 6) + (h1 >> 2));
    }
};

int main() {
    std::unordered_map<UserKey, std::string, UserKeyHash> users;

    users.emplace(UserKey{1, "EU"}, "Alice");
    users.emplace(UserKey{2, "US"}, "Bob");
}
```

In this example, `UserKey` contains two fields, and equality is determined by both fields.

The hash function computes the hash of each field and combines the results. The combination formula is a simple, commonly used pattern, but it is not necessarily the best choice for every dataset.

### The Relationship Between Hashing and Equality

One of the most important requirements of `std::unordered_map` is that equal keys must have equal hash values:

\\[ a == b \implies hash(a) = hash(b) \\]

The reverse implication is not required. Two keys having the same hash value does not mean that the keys themselves are equal.

If the hash function and equality predicate are inconsistent, the container will not satisfy the standard's requirements.

### Important Properties of a Hash Function

A suitable hash function should:

- Produce consistent results for the same key throughout the relevant lifetime.
- Produce the same hash value for equal keys.
- Provide a suitable distribution for the expected keys.
- Have an acceptable computational cost.
- Avoid depending on mutable state that changes the hash result.

A fast hash function with poor distribution can significantly reduce the overall performance of a hash table.

### Using a Custom Equality Predicate

Sometimes, key equality does not match the default behavior of `operator==`. For example, we may want strings to compare equal regardless of letter case.

In such cases, both the hash function and equality predicate must implement the same policy. If the equality predicate considers two strings equal, their hash values must also be equal.

Using a case-insensitive comparison together with the ordinary string hash function, without ensuring that the two are compatible, is incorrect.

## Using String Keys and Case Sensitivity

### Default String Behavior

By default, comparisons between `std::string` values are case-sensitive.

Consequently, the following keys are different:

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> values;

    values["Admin"] = 1;
    values["admin"] = 2;

    std::cout << values.size() << '\n';
}
```

The output is `2`.

If an application requires case-insensitive lookup, it must define a consistent normalization policy or implement compatible custom hash and equality functions.

In many applications, normalizing keys when data enters the system is a simpler solution. However, Unicode normalization, particularly across different languages, requires a well-defined policy and a careful implementation.

## Managing Iterators and References

### Iterator Validity After Insertion

An important consideration when using `std::unordered_map` is iterator validity after insertion.

If an insertion causes rehashing, iterators referring to elements in the container become invalid.

For example, retaining an iterator in one part of a program and then inserting elements without accounting for possible rehashing can lead to errors.

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> values{
        {"a", 1},
        {"b", 2}
    };

    auto it = values.find("a");

    values.reserve(1000);

    // Do not use the old iterator after a possible rehash.
}
```

After `reserve()`, do not assume that the previous iterator remains valid, because the operation may trigger rehashing.

### Pointer and Reference Validity

In contrast, rehashing by itself does not invalidate pointers or references to existing elements. This is an important property of unordered containers.

However, erasing an element invalidates pointers, references, and iterators referring to that particular element.

Additionally, if a stored object is modified in a way that affects its address or lifetime, the rules for that object type must also be considered.

### Modifying Element Keys

The key of each `std::unordered_map` element is stored as `const`. This restriction is necessary to preserve the consistency of the hash table.

Changing a key directly could make the element's logical location inconsistent with its hash value. For this reason, modifying the key through an iterator is not permitted.

If a key must be changed, the usual approach is to erase the element and insert it again with the new key.

C++17 introduced Node Handles, which make it possible to extract an element and modify its key without reconstructing the entire object.

## Advanced Features in Later C++ Standards

### Extracting Elements with `extract()`

Node Handles were introduced in C++17. The `extract()` function removes an element from the container while keeping its node available for reinsertion or key modification.

```
#include <string>
#include <unordered_map>
#include <utility>

int main() {
    std::unordered_map<std::string, int> values{
        {"old_key", 42}
    };

    auto node = values.extract("old_key");

    if (!node.empty()) {
        node.key() = "new_key";
        values.insert(std::move(node));
    }
}
```

In this example, the key is changed through the Node Handle, and the element is then inserted back into the container.

This feature is useful for transferring elements between compatible containers and changing keys without copying the entire stored value.

### Transferring Elements with `merge()`

C++17 introduced `merge()` for transferring eligible nodes between compatible containers:

```
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> source{
        {"a", 1},
        {"b", 2}
    };

    std::unordered_map<std::string, int> destination{
        {"b", 20},
        {"c", 3}
    };

    destination.merge(source);
}
```

In this example, element `a` is transferred to the destination. However, element `b` remains in `source` because the destination already contains that key.

Node transfer depends on container compatibility, including the applicable allocator requirements.

### Heterogeneous Lookup

In C++20, heterogeneous lookup was standardized for applicable associative and unordered associative containers when compatible transparent hash and equality functions are provided.

This feature allows lookup using a type different from the container's key type, potentially avoiding the construction of a temporary key object.

For example, when keys are stored as `std::string`, it may be possible to perform lookup using `std::string_view` without constructing a temporary string.

This capability requires compatible transparent hash and equality implementations. In C++11, these APIs must not be assumed to exist.

## Practical Applications

### Counting Element Frequencies

A common use of `std::unordered_map` is counting how frequently elements occur:

```
#include <iostream>
#include <string>
#include <unordered_map>
#include <vector>

int main() {
    std::vector<std::string> words{
        "apple", "banana", "apple", "orange", "banana", "apple"
    };

    std::unordered_map<std::string, int> frequency;

    for (const auto& word : words) {
        ++frequency[word];
    }

    for (const auto& entry : frequency) {
        std::cout << entry.first << ": " << entry.second << '\n';
    }
}
```

In this example, `operator[]` creates a counter initialized to zero if the key does not exist, and then increments it.

The output order is not guaranteed.

### Building a Cache

A cache can use this container to store computed results or request responses:

```
#include <iostream>
#include <unordered_map>

int calculate(int value) {
    return value * value;
}

int main() {
    std::unordered_map<int, int> cache;

    auto getValue = [&cache](int input) {
        auto it = cache.find(input);

        if (it != cache.end()) {
            return it->second;
        }

        int result = calculate(input);
        cache.emplace(input, result);

        return result;
    };

    std::cout << getValue(10) << '\n';
    std::cout << getValue(10) << '\n';
}
```

In this example, a repeated input does not trigger another computation.

However, this simple implementation does not include features such as a size limit, expiration, concurrency management, or eviction of old entries. Real-world caches must account for these requirements separately.

### Mapping IDs to Objects

This container is suitable for mapping identifiers to application data, such as users, sessions, or internal resources.

```
#include <string>
#include <unordered_map>

struct User {
    std::string name;
    int age;
};

int main() {
    std::unordered_map<int, User> users;

    users.emplace(101, User{"Alice", 30});
    users.emplace(102, User{"Bob", 25});
}
```

In this design, the identifier serves as the key, and each user's information is stored as the mapped value.

### Building an Index for Fast Lookup

When data is initially stored as a collection of objects, a separate index implemented with `std::unordered_map` can provide faster access by identifier.

In this design, the relationship between the original data and the index must be maintained. If an object is modified or deleted, the index must also be updated. Otherwise, it may refer to nonexistent objects or outdated data.

## Limitations and Common Mistakes

### Assuming Constant-Time Complexity

The average-case complexity of lookup in `std::unordered_map` is \\(O(1)\\), but this does not guarantee constant execution time for every operation.

In the worst case, lookup, insertion, or deletion can take \\(O(n)\\) time, such as when many keys end up in the same bucket.

Therefore, applications with strict response-time requirements should be tested with representative datasets.

### Using a Poor Hash Function

If a hash function distributes the expected keys poorly, collisions increase and performance deteriorates.

For custom types, the hash function should be designed around the actual properties of the data. Using a simple formula without examining the key distribution may produce poor results in real applications.

### Accidentally Using `operator[]`

A common mistake is using `operator[]` merely to check whether a key exists:

```
int value = values["missing"];
```

This code may insert a new element. If insertion is not intended, use `find()` instead.

### Depending on Iteration Order

The iteration order of `std::unordered_map` must not be treated as part of the application's contract. Doing so can cause tests to fail or output to change between execution environments.

If ordering matters, use an ordered data structure or explicitly sort the output.

### Changing State That Affects Key Hashing

The hash function and equality predicate must remain compatible while a key is stored in the container.

If either depends on mutable state that changes while the key is present, the container may no longer be able to locate the key as expected.

For this reason, hash functions and equality predicates should be based on stable key properties.

### Assuming Thread Safety

`std::unordered_map` is not automatically a thread-safe container for concurrent access involving modifications.

If multiple threads perform writes simultaneously, or one thread modifies the container while another accesses it without appropriate synchronization, a data race may occur.

Concurrent use requires appropriate synchronization, such as a mutex, or a data structure specifically designed for concurrent access.

### Ignoring Memory Consumption

In many implementations, a hash table consumes memory not only for the elements themselves but also for buckets and management structures.

Consequently, `std::unordered_map` may use more memory than alternative containers, particularly when the number of elements is small or the load factor is configured to be very low.

For applications with strict memory constraints, measuring actual memory consumption and comparing alternatives is essential.

## Performance Considerations for Professional Applications

### Reserving Capacity Before Bulk Insertion

If the final number of elements is predictable, using `reserve()` before bulk insertion can generally reduce repeated rehashing.

However, capacity should be chosen based on a realistic estimate. Reserving far more space than necessary can increase memory consumption without providing a meaningful benefit.

### Choosing an Appropriate Key Type

The cost of hashing a key affects the container's overall performance. For example, hashing an integer is generally less expensive than hashing a long string.

In applications that perform many lookups, the cost of constructing keys, copying strings, and allocating memory should also be considered.

### Avoiding Unnecessary Copies

When iterating over elements, using a constant reference when modification is unnecessary avoids copying elements:

```
for (const auto& entry : values) {
    // Process the existing element without copying it
}
```

In this example, the container's elements are not copied, and their data is accessed in a read-only manner.

### Measuring Instead of Guessing

The performance of `std::unordered_map` depends on several factors, including key type, hash function quality, element count, cache locality, access patterns, and the standard library implementation.

Therefore, the final choice should be based on benchmarks that reflect the application's real data and access patterns.

## Comparing Alternative Containers

### Comparing with `std::vector`

If the number of elements is small or sequential traversal is important, linear search through a `std::vector` may perform competitively or even outperform a hash table in practice.

This is partly due to contiguous memory storage and favorable cache behavior during sequential operations.

In contrast, `std::unordered_map` is generally more suitable for frequent key-based lookups in larger collections.

### Comparing with `std::unordered_set`

If you only need to store keys and check whether they exist, without associating values with them, `std::unordered_set` is a more appropriate choice.

This container also uses a hash table but stores a set of unique keys rather than mapping keys to values.

### Comparing with `std::map`

If an application needs sorted keys, range queries, or access to the smallest and largest keys, `std::map` offers important advantages.

In contrast, if those requirements do not exist and fast key-based lookup is the priority, `std::unordered_map` may be the better choice.

## Compatibility with Different C++ Standards

### Features Available in C++11

The core functionality of `std::unordered_map` has been available since C++11, including:

- Container construction and initialization.
- Access through `operator[]` and `at()`.
- Lookup with `find()`.
- Insertion with `insert()` and `emplace()`.
- Removal with `erase()`.
- Capacity management with `reserve()` and `rehash()`.
- Load factor configuration and inspection.
- Custom hash functions and equality predicates.
- Move semantics and initializer-list support.

Projects must be compiled using C++11 or a later standard to use these features.

### Features Introduced in C++17

C++17 added features such as `try_emplace()`, `insert_or_assign()`, Node Handles, and `merge()`.

These features provide better control over element construction, key modification, and element transfer between containers.

### Features Introduced in C++20

C++20 introduced `contains()` and standardized heterogeneous lookup for applicable containers.

Projects that must remain compatible with C++11 should not use these APIs without checking the required language standard.

## Summary

`std::unordered_map` is an important component of the C++ Standard Library for mapping unique keys to values. Its hash table implementation provides average-case \\(O(1)\\) complexity for lookup, insertion, and deletion, but actual performance depends on hash function quality, key distribution, load factor, and runtime conditions.

For professional use, the following considerations are particularly important:

- Use `find()` or `at()` when reading existing keys, and be aware that `operator[]` may insert an element as a side effect.
- When the final size is predictable, use `reserve()` to reduce repeated rehashing.
- For custom key types, ensure that the hash function and equality predicate are compatible.
- Account for iterator invalidation after rehashing and element deletion.
- Never assume that element iteration order is guaranteed.
- Benchmark and measure actual performance when execution time or memory usage is critical.
- Consider alternatives such as `std::map` when sorted keys or range queries are required.

Ultimately, choosing `std::unordered_map` should depend on the application's actual requirements. It is a powerful tool for fast key-based lookup, but understanding its limitations and behavior is essential for building reliable, efficient, and maintainable software.

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>