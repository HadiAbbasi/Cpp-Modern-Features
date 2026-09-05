<div align="right">

[🇺🇸 English](./emplace.md) | [🇮🇷 فارسی](../../fa/cpp11/emplace.md)

</div>

# std::emplace / emplace_back

##  std::emplace
In C++, vector emplace() is used to insert elements at the given position in a vector by constructing it in-place within the vector, rather than creating a temporary object and then moving or copying it into the vector.

Let's take a quick look at a simple example that uses vector emplace() method:

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> v = {1, 5, 8};
  
  	// Inserting 6 at index 2
    v.emplace(v.begin() + 2, 6);

    for (auto i : v)
        cout << i << " ";
    return 0;
}
```
### syntax 
The vector emplace() is the member method of std::vector defined inside <vector> header file.
```cpp

    v.emplace(pos, args...);
```
**Parameters**

- pos: Iterator to the position where element is to be inserted.
- args...: Arguments forwarded to the constructor of the vector's element type.

**Return Value**

- Returns an iterator to the inserted element.

### Examples of vector emplace

The following examples demonstrates the use of vector emplace() function for different insertion position and number of elements:

#### Insert an Element at the End of Vector

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> v = {1, 4, 5};

    // Insert element 6 at the end
    v.emplace(v.end(), 6);

    for (auto i : v)
        cout << i << " ";
    return 0;
}
```
We can also use the vector `emplace_back()` method for insert an element at the end of vector.

#### Insert Element at Specific Position in a Vector of Custom Data Type
```cpp
#include <bits/stdc++.h>
using namespace std;

// Custom Type
class A {
  public:
  int a, b, c;
  A(int x = 0, int y = 0, int z = 0):
    a(x), b(y), c(z) {}
};

int main() {
    vector<A> v = {{1, 4, 5}};

    // Insert an element at index 1
    v.emplace(v.begin() + 1, 2, 3, 6);

    for (auto i : v)
        cout << i.a << " " << i.b << " "
      		<< i.c << endl;
    return 0;
}
```
**Explanation:** The argument we pass to the vector emplace() method are forwarded to the constructor of the vector's element type. So, the arguments 2, 3, and 6 are passed to the constructor of A which then creates an object in-place to insert it into the vector.

### Difference Between vector emplace() and vector insert()
Both vector emplace() and vector insert() are used to insert elements into the vector, but they differ from each other in how they operate.

| Feature | `vector::emplace()` | `vector::insert()` |
| :--- | :--- | :--- |
| **Purpose** | Insert a single element in the vector at given position. | Insert a single or multiple elements in a vector at given position. |
| **Temporary Object** | Avoids creating a temporary object, directly constructs the element in place. | Requires an existing object, which either already exists or is created for insertion. |
| **Efficiency** | More efficient for complex types since it avoids copies/moves. | Less efficient with complex types due to extra moves/copies. |
| **Syntax** | `v.emplace(pos, args...)` | `v.insert(pos, val)`<br>`v.insert(pos, n, val)`<br>`v.insert(pos, first, last)` |


## emplace_back
In C++, the vector emplace_back() is a built-in method used to insert an element at the end of the vector by constructing it in-place. It means that the element is created directly in the memory allocated to the vector avoiding unnecessary copies or moves.

Let’s take a quick look at a simple example that illustrates vector emplace_back() method:

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> v;

    // Inserting elements
    v.emplace_back(1);
    v.emplace_back(9);
    v.emplace_back(5);

    for (auto i : v)
        cout << i << " ";
    return 0;
}
```
### syntax
The vector emplace_back() is the member method of std::vector class defined inside <vector> header file.
```cpp
    v.emplace_back(val);
```
**Parameters**

- **val**: Value to be added. It is forwarded to the constructor of the type of vector.

**Return Value**

- Until C++ 17, this function didn't used to return any value, but now, it returns the reference to the inserted value.

### Examples

The following examples demonstrates the use of vector emplace_back() function for different scenarios:

#### Inserting Elements in Vector of Strings
```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<string> v = {"Hi", "Geeks!"};

    // Inserting element at the end
    v.emplace_back("Welcome");

    for (auto i : v)
        cout << i << " ";
    return 0;
}
```
#### Inserting Elements in a Vector of Custom Type
```cpp
#include <bits/stdc++.h>
using namespace std;

// Class that notifies when copied
class A {
  public:
  int a;
  A(int x = 0) {
      a = x;
  }
  A(const A& other) {
    	a = other.a;
    	cout << a << "'s CC called\n";
  }
};

int main() {
    vector<A> v;
  
  	v.reserve(5);
    
  	// Inserting element using emplace_back()
    v.emplace_back(1);
  	v.emplace_back(9);
  	
  	// Inserting element using push_back()
  	v.push_back(5);
  
    return 0;
}

```
From the above example, we can confirm that vector emplace_back() doesn't create extra copies but vector push_back() does.


### Vector emplace_back() vs push_back()

Both functions are used to insert elements at the back of vector but they differs in the way of insertion. Following table lists the primary differences of the vector emplace_back() and vector push_back() in C++:

| Feature | `vector::emplace_back()` | `vector::push_back()` |
| :--- | :--- | :--- |
| **Mechanism** | Constructs the element in-place directly in the vector. | Creates an object to be copied/moved first and then passes it to this function. |
| **Efficiency** | More efficient as it avoids unnecessary copying or moving. | Less efficient due to copying or moving the object. |
| **Syntax** | `v.emplace_back(args...)` | `v.push_back(val)` |

---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>