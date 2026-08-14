<div align="right">

[   English](./list-initialization.md) | [🇮🇷 فارسی](../../fa/cpp11/list-initialization.md)

</div>

# std::initializer_list in C++
>In C++, the std::initializer_list is a class template that allows us to initialize a lightweight object with a list of values. An initializer list is used to set values to variables, arrays, classes, functions, constructors of classes, and standard containers like vectors in a convenient way.

## cpp11
>The std::initializer_list class template was added in C++ 11 and contains many built-in functions to perform various operations with the initializer list. It provides member functions like a `size()`, `begin()`, `end()`, and constructor to construct, iterate, and access elements of the initializer list.

To use `initializer_list`, you need to include the `<initializer_list>` header in your C++ program.
```cpp
initializer_list<T> name_of_list= { };
```
- Braced Initializer is used to construct the object for initializer_list.
- It is generally implemented as a wrapper over arrays.
- Unlike standard containers like vectors, copying the object of the initializer list does not copy the entire elements to the copied list. But both the original and copied object of the initializer list contain the same elements.

### Examples
```cpp
#include <initializer_list>
#include <iostream>
using namespace std;
int main()
{
    initializer_list<int> num = { 2, 4, 6, 8, 10, 12 };
    cout << "Numbers in the list are: ";
    for (int it : num) {
        cout << it << " ";
    }
    return 0;
}
```
```cpp
// C++ program to illustrate the use of initializer_list in
// object construction
#include <iostream>
using namespace std;
#include <initializer_list>

// array type container constructed using initializer list
template <typename T> class MyContainer {
public:
    // Constructor taking initializer_list as a parameter
    MyContainer(initializer_list<T> values)
        : list(values)
    {
    }

    // Function to print all elements
    void printList() const
    {
        for (const T& value : list) {
            cout << value << " ";
        }
        cout << endl;
    }

private:
    initializer_list<T> list;
};

// diver code
int main()
{
    // Creating an instance of MyContainer with
    // initializer_list of int type
    MyContainer<int> intContainer = { 1, 2, 3, 4, 5 };
    cout << "Elements of Integer type are: ";
    intContainer.printList();
    cout << endl;

    // Creating an instance of MyContainer with
    // initializer_list of double type
    cout << "Elements of double type are: ";
    MyContainer<double> doubleContainer
        = { 1.1, 2.2, 3.3, 4.4, 5.5 };
    doubleContainer.printList();
    cout << endl;

    return 0;
}
```
**NOTE:** "std::initializer_list" should be used for immediate consumption, such as iterating in a constructor or passing to a function. It must not be stored as a data member because it does not own its storage and its lifetime is limited to the full expression in which it is created.

### Member Functions of std::initializer_list
```cpp 
#include <initializer_list>
#include <iostream>

int main() {
    std::initializer_list<int> numbers{10, 20, 30, 40};

    // 1) begin(): اشاره‌گر به اولین عنصر
    const int* first = numbers.begin();
    std::cout << "begin(): " << *first << '\n';

    // 2) end(): اشاره‌گر به یک خانه بعد از آخرین عنصر
    const int* afterLast = numbers.end();

    // برای دسترسی به آخرین عنصر، یک واحد به عقب برمی‌گردیم
    std::cout << "last element: " << *(afterLast - 1) << '\n';

    // 3) size(): تعداد عناصر
    std::cout << "size(): " << numbers.size() << '\n';

    // 4) empty(): بررسی خالی بودن
    if (numbers.empty()) {
        std::cout << "The list is empty.\n";
    } else {
        std::cout << "The list is not empty.\n";
    }

    // 5) data():
    // std::initializer_list در C++11 تابع data() ندارد.
    // begin() عملاً اشاره‌گر به آرایهٔ داخلی را می‌دهد.
    const int* data = numbers.begin();

    std::cout << "underlying data: ";
    for (std::size_t i{0}; i < numbers.size(); ++i) {
        std::cout << data[i] << ' ';
    }

    std::cout << '\n';

    return 0;
}
```

| S. No. | Function Name | Description | Code Example |
|---:|---|---|---|
| 1 | `begin()` | Returns a pointer (iterator) to the first element of the `initializer_list`. | `const int* first = numbers.begin();` |
| 2 | `end()` | Returns a pointer (iterator) to **one position after the last element**. | `const int* afterLast = numbers.end();` |
| 3 | `size()` | Returns the number of elements in the initializer list. | `std::size_t count = numbers.size();` |
| 4 | `empty()` | Returns `true` if the initializer list is empty; otherwise, returns `false`. | `bool isEmpty = numbers.empty();` |
| 5 | `data()` | `std::initializer_list` does **not** provide a `data()` member function in C++11. Use `begin()` to get a pointer to its un


## Applications of initializer_list
Apart from the construction of objects, initializer list can be used in the following cases:
### Variable Function Parameters
```cpp
#include <iostream>
using namespace std;
#include <initializer_list>

void myFunction(initializer_list<int> myList)
{
    cout << "Size of myList: " << myList.size();
    cout << "\n";
    cout << "Elements of myList: ";
    for (int value : myList) {

        // Print value at each iteration
        cout << value << " ";
    }
}

int main()
{
    myFunction({ 1, 2, 3, 4, 5 });

    return 0;
}
```
### Store Data in Contigious Memory
The elements of the initializer_list container can be used to store data as it is a lightweight container.
```cpp
#include <iostream>
using namespace std;
#include <initializer_list>

int main() {
    initializer_list<int> list = {1, 2, 3, 4, 5};
    for (int value : list) {
        cout << value << " ";
    }
    cout << endl;
    return 0;
}
```
### Initialize Standard Containers
The initializer_list can be used for initializing the standard containers with a List of Elements like vectors.

```cpp
// C++ program to demonstratethe use of initializer_list as
// return type.

#include <initializer_list>
#include <iostream>
#include <vector>
using namespace std;

initializer_list<int> getNumbers()
{
    return { 1, 2, 3, 4, 5 };
}

int main()
{
    auto num = getNumbers();
    // Use the generated numbers
    for (auto it : num) {
        cout << it << " ";
    }
    return 0;
}
```
### Initializer Lists as Return Types
The initializer_list can be used as a return type to return lists from any function. It allows the function to return multiple values.
```cpp
#include <initializer_list>
#include <iostream>
#include <vector>
using namespace std;

initializer_list<int> getNumbers()
{
    return { 1, 2, 3, 4, 5 };
}

int main()
{
    auto num = getNumbers();
    for (auto it : num) {
        cout << it << " ";
    }
    return 0;
}
```
## Limitations of initializer_list
**The initializer lists also have some limitations associated with it:**

- **Size cannot be changed:** The size of initializer_list is fixed at compile time. It does not have a dynamic nature as a standard container such as a vector. The size of the initializer cannot be changed once it has been created.
- **Cannot access the elements randomly:** initializer_list supports only forward iteration. We cannot access the desired or random element using the index as standard containers.
 - **Immutable elements:** The elements within an initializer_list are immutable. Once the list is created, the values cannot be modified. Any attempt to modify the elements through the iterator or by any other means will result in a compilation error.
## 🤝 Contributors



<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>