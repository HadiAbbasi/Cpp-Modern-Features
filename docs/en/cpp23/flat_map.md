<div align="center">

[🇺🇸 English](./flat_map.md) | [🇮🇷 فارسی](../../fa/cpp23/flat_map.md)

</div>

# std::flat_map
C++23 introduces a new header, `<flat_map>`, for storing key-value pairs in a more cache-friendly and efficient manner. It provides an alternative to traditional associative containers like std::map and std::unordered_map.

std::flat_map is a container introduced in C++23 for storing key-value pairs. It combines features of std::map and std::vector:

- Stores elements in sorted order based on keys
- Uses contiguous memory layout for better cache performance
- Provides efficient access and faster traversal



To use it, include the `<flat_map>` header and declare it with the required key and value types.
```cpp
std::flat_map < keyType, valueType > name;
```

## `std::flat_map` Initialization
We can initialize a flat_map container in C++ using the following methods:

### Using the Initializer List
We can initialize std::flat_map by proving key and value pairs in the initializer list.

```cpp
std::flat_map<int, string> object_name { {3, "three"}, {1, "one"}, {4, "four"} };
```

### Using the Range Constructor
We can use the range constructor to initialize the std::flat_map object with key and value pairs by specifying the range using the starting iterator and ending iterator.

```cpp
flat_map <keyType, valueType > object_name(startingIterator, endingIterator);
```

### Using the Assignment Operator
We can simply initialize the flat_map object object with another already existing flat_map object using an assignment operator.

```cpp
std::flat_map <keyType, valueType > object_name;
object_name = prev_object;
```

### Using the Move Assignment Operator
We can initialize a std::flat_map using the move assignment operator. This way of initializing a std::flat_map is memory efficient because the assigned object simply points to the existing memory location where the data is stored without creating unnecessary copies of data.

```cpp
std::flat_map <keyType, valueType > object_name;
object_name = move(prev_object);
```


## Commonly Used Methods

### Iterators
Iterators are used to traverse the container. The following function returns iterators to the flat_map containers:

- **begin()** - Returns an iterator pointing to the first element in the flat_map.
- **end()** - Returns an iterator pointing to the position just after the last element in the flat_map.

```cpp
#include <flat_map>
#include <iostream>
using namespace std;

int main()
{
    flat_map<int, string> myMap = { { 1, "Geeks" },
                                    { 2, "for" },
                                    { 3, "Geeks" },
                                    { 4, "Coding" } };

    // Iterate over the map using begin() and end()
    cout << "All key-value pairs in the map:" << endl;
    for (auto it = myMap.begin(); it != myMap.end(); ++it) {
        cout << "Key: " << it->first
             << ", Value: " << it->second << endl;
    }

    return 0;
}
```
**output**

```txt
All key-value pairs in the map:
Key: 1, Value: Geeks
Key: 2, Value: for
Key: 3, Value: Geeks
Key: 4, Value: Coding
```

### Modifiers
- pair insert(keyvalue, mapvalue): Adds a new element to the map.
- erase(iterator position): Removes the element at the position pointed by the iterator.
- erase(const key_type& g): Removes the key value ‘g’ from the map.
- clear() - Removes all the elements from the map.


```cpp

#include <flat_map>
#include <iostream>
using namespace std;

int main()
{
    flat_map<int, string> myMap;

    // Insert key-value pairs using insert()
    myMap.insert(make_pair(1, "Geeks"));
    myMap.insert(make_pair(2, "for"));
    myMap.insert(make_pair(3, "Geeks"));
    myMap.insert(make_pair(4, "Coding"));

    // Print the map before erasing
    cout << "Map before erasing:" << endl;
    for (const auto& pair : myMap) {
        cout << "Key: " << pair.first
             << ", Value: " << pair.second << endl;
    }

    // Erase an element by key using erase()
    myMap.erase(3);

    // Print the map after erasing by key
    cout << "Map after erasing by key:" << endl;
    for (const auto& pair : myMap) {
        cout << "Key: " << pair.first
             << ", Value: " << pair.second << endl;
    }

    // Clear the map using clear()
    myMap.clear();

    // Check if the map is empty after clearing
    if (myMap.empty()) {
        cout << "The map is empty after clearing." << endl;
    }

    return 0;
}
```

**output**

```txt
Map before erasing:
Key: 1, Value: Geeks
Key: 2, Value: for
Key: 3, Value: Geeks
Key: 4, Value: Coding
Map after erasing by key:
Key: 1, Value: Geeks
Key: 2, Value: for
Key: 4, Value: Coding
The map is empty after clearing.
```

### Size

- size() - Returns the number of elements in the flat_map.
- max_size() - Returns the maximum number of elements that the flat_map can hold.
- empty() - Returns whether the flat_map is empty.

```cpp
#include <flat_map>
#include <iostream>
using namespace std;

int main()
{
    flat_map<int, string> myMap
        = { { 1, "Geeks" }, { 2, "for" }, { 3, "Geeks" } };

    // Get the size of the map using size()
    cout << "Size of the map: " << myMap.size() << endl;

    // Get the maximum possible size of the map using
    // max_size()
    cout << "Maximum possible size of the map: "
         << myMap.max_size() << endl;

    // Check if the map is empty using empty()
    if (myMap.empty()) {
        cout << "The map is empty." << endl;
    }
    else {
        cout << "The map is not empty." << endl;
    }

    return 0;
}
```

**output**

```txt
Size of the map: 3
Maximum possible size of the map: 256204778801521550
The map is not empty.
```


### Map operations

- **bool contains(const key_type& g) const** : Returns true if there is an element present with key value 'g' in the flat_map, else it returns false.
- **find(const key_type & g)**: Returns an iterator pointing to an element with the key value 'g', or end() if such an element is not present in the flat_map.
- **count(const key_type & g) const** : Returns the number of elements with key value 'g'.

```cpp
#include <flat_map>
#include <iostream>

int main()
{
    flat_map<int, string> myMap;

    myMap.insert(make_pair(1, "Geeks"));
    myMap.insert(make_pair(2, "for"));
    myMap.insert(make_pair(3, "Geeks"));

    // Check if a key exists using contains()
    if (myMap.contains(2)) {
        cout << "Key 2 exists in the map." << endl;
    }

    // Find an element by key using find()
    auto it = myMap.find(3);
    if (it != myMap.end()) {
        cout << "Value at key 3: " << it->second << endl;
    }

    // Count the number of elements with a specific key
    // using count()
    int key = 2;
    int numOccurrences = myMap.count(key);
    cout << "Number of occurrences of key " << key << ": "
         << numOccurrences << endl;

    return 0;
}
```
**Output**

```txt
Key 2 exists in the map.
Value at key 3: Geeks
Number of occurrences of key 2: 1
```

## Time Complexity

The usual complexity of the operations below is as follows; the standard may specify different details for particular overloads:

| Operation | Typical complexity |
|---|---:|
| Lookup such as `find`, `contains`, `lower_bound` | `O(log n)` |
| Traversing the entire container | `O(n)` |
| Inserting or erasing one element | Usually `O(n)`, due to shifting elements in the underlying sequences |
| `clear()` | `O(n)` in the general case, to destroy/release the elements |

Binary search on sorted data is fast, but insertion and erasure should not generally be considered `O(log n)`: finding the position may take logarithmic time, while preserving order in contiguous storage usually requires linear element movement.

## Comparison with Other Containers

| Feature | `std::flat_map` | `std::map` | `std::unordered_map` |
|---|---|---|---|
| Key order | Sorted | Sorted | No ordering guarantee |
| Typical structure | Two contiguous sequences | Balanced tree | Hash table |
| Lookup | `O(log n)` | `O(log n)` | Average `O(1)`, worst-case `O(n)` |
| Insertion/erasure | Usually `O(n)` | `O(log n)` | Average `O(1)`, with possible rehashing |
| Key strength | Good locality and traversal performance | Ordered updates with generally more stable iterators | Fast average lookup, no ordering |

These complexity figures are summaries; comparison costs, memory allocation, and data size also affect real-world performance.

## Advantages

- One of the key benefits of flat_map is its ability to provide fast search, insertion, and deletion operations while maintaining the sorted order.
- It offers logarithmic time complexity for these operations, making it appropriate for programs that require efficient key-value storage and retrieval.
- The contiguous memory format of flat_map allows for cache-friendly access patterns, in addition to improving performance.
- Used in dictionary applications to quickly map words to their definitions.
- Useful in leaderboards to manage scores and retrieve or update rankings efficiently.


---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>