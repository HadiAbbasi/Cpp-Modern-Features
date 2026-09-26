<div align="right">

[🇺🇸 English](./range_base.md) | [🇮🇷 فارسی](../../fa/cpp11/range_base.md.md)

</div>

# range_base for
The range-based for loop, introduced in C++11, provides a simple and readable way to iterate over all elements of a range. It automatically traverses arrays, strings, and STL containers without requiring explicit indices or iterators.

- Simplifies iteration over arrays, strings, and containers.
- Eliminates the need to manage loop counters and iterators manually.
- Improves code readability and reduces the chances of iteration-related errors. 

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5};

    // Iterating through vector
    for (int i : v) {
        cout << i << " ";
    }
  
    return 0;
}
```
The range-based for loop automatically visits each element of the vector. There is no need to specify the container size or use iterators explicitly.

**syntax**
```cpp
for (declaration: range) {
    // statements
}
```
**where**
- `declaration:` Variable used to represent each element of the range.
- `range:` The array, string, container, or any iterable object being traversed.


## Working of Range-Based for Loop

The working of a range-based for loop is as follows:

1. The loop obtains the beginning and ending positions of the specified range.
2. The first element is assigned to the loop variable.
3. The loop body is executed using that element.
4. The loop variable is updated with the next element in the range.
5. Steps 3 and 4 repeat until all elements have been processed.
6. Control exits the loop after the last element is visited.


## Examples of Range Based for Loop
The below example demonstrates the use of range based for loop in our C++ programs:

### Iterate over an Array

The following program prints all elements of an array using a range-based for loop.
```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[] = {1, 2, 3, 4, 5};
  	
  	// Range based for loop to iterate over array
  	// and i is used to represent each element 
    for (int i : arr) {
        cout << i << " ";
    }

    return 0;
}
```

### Iterate over a Map

Since each element of a map is stored as a key-value pair, the loop variable must represent a pair object. Using auto allows the compiler to determine the appropriate type automatically.

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    map<int, char> m = {{1, 'A'}, {2, 'B'}, {3, 'C'},
                  {4, 'D'}, {5, 'E'}};
  	
  	// Range based for loop to iterate over array
  	// and i is used to represent each element 
    for (auto p: m) {
        cout << p.first << ": " << p.second << endl;
    }

    return 0;
}
```
## Advantages of Range-Based for Loop

The range-based for loop offers several benefits over traditional iteration methods, making code simpler, safer, and easier to read.

- Provides cleaner and more readable syntax.
- Eliminates the need for explicit loop counters and iterators.
- Reduces the possibility of index-related errors.
- Works seamlessly with arrays, strings, and STL containers.
- Makes code easier to maintain and understand.
---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>