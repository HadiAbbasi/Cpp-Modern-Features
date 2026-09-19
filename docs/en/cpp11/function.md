<div align="right">

[🇺🇸 English](./function.md) | [🇮🇷 فارسی](../../fa/cpp11/function.md)

</div>

# Function

Now imagine you’re building an event system — a button that stores an onClick callback. What type do you give that member variable? A function pointer? Then lambdas with captures won't work. A template parameter? Then every different callback creates a different button type. You want one uniform type to hold any callable, and you want a way to adapt callables that don't quite fit the expected signature by pre-filling some arguments.

**That’s `std::function` and [`std::bind`](./bind.md).**


## std::function

Think of `std::function` as a box that can hold anything callable with a matching signature.

![function](./assets/function.webp)

std::function in C++ is a template class used to store and invoke callable objects such as functions, lambda expressions, and functors. It provides a flexible and generic way to handle callable objects in C++ programs.

- Can wrap different types of callable objects in a single interface
- Commonly used in callbacks, event handling, and functional programming
- Improves code flexibility, reusability, and maintainability

```cpp
#include <iostream>
#include <functional>
using namespace std;

int add(int a, int b) {

    return a + b;
}

int main() {

    function<int(int, int)> f = add;

    cout << f(10, 20);

    return 0;
}
```
```cpp
#include <bits/stdc++.h>
using namespace std;

int f(int a, int b) {
  return a + b;
}

int main() {
  
  	// std::function wrapping traditional
  	// function
	function<int(int, int)> calc = f;
  	cout << "Sum: " << calc(8, 2) << endl;
    
  	// std::function wrapping a lambda
    // expression
    calc = [](int a, int b) { return a * b; };
  	cout << "Product: " << calc(8, 2);
    return 0;
}
```

### Define

To create a wrapper object, we first declare it using the following syntax:

```cpp
    std::function< rtype (atype...)> name();
```
**where**:

-**name** :Name of the wrapper.
-**atype**: Types of the arguments that function takes.
-**rtype**: Return type of the function you want to store



## Member Functions of std::function


| Function | Description |
|---|---|
| **swap()** | Swaps the wrapped callable of two `std::function` objects. |
| **operator bool** | Checks if the `std::function` contains a callable. |
| **operator ()** | Invoke the callable with the given arguments. |
| **target()** | Returns a pointer to the stored callable. If there is no callable stored, returns `nullptr`. |
| **target_type()** | Returns the [`typeid`](https://www.geeksforgeeks.org/cpp/typeid-operator-in-c-with-examples/) of the callable. If no callable is stored, it returns `typeid(void)`. |

## Applications of std::functions in C++
Apart from the applications shown in the above examples, std::function can also be used for:

- **Callbacks**: Used for event-driven systems where functions are passed and called later when an event occurs.
- **Function Wrapping and Higher-Order Functions**: Allows passing functions as parameters and returning them.
- **Stateful Callbacks**: Enables callbacks with preserved states, making it easier to manage state without global variables.
- **Replacing Function Pointers**: Provides a safer and more flexible alternative to traditional function pointers, supporting multiple callable types.

## Examples of std::function

### Passing std::function as Argument (Callback)
```cpp
#include <bits/stdc++.h>
using namespace std;

// Functions for simple math operations
int add(int a, int b) {
    return a + b;
}
int sub(int a, int b) {
    return a - b;
}
int mul(int a, int b) {
    return a * b;
}
int divs(int a, int b) {
    return a / b;
}

// Using std::function as parameter
void func(int a, int b, 
         function<int(int, int)> calc) {
  
    int res = calc(a, b);
    cout << "Result: " << res << endl;
}

int main() {
    
  	// Calling all the arithmetic functions
    func(8, 2, add);
    func(8, 2, sub);
    func(8, 2, mul);
    func(8, 2, divs);
    return 0;
}
```
### Wrapping Member Functions of a Class
We can also wrap a member function of a class using std::function object but we have to add one extra argument as shown in the below program.

```cpp
#include <bits/stdc++.h>
using namespace std;

class C {
public:
    int f(int a, int b) {
        return a * b;
    }
};

int main() {
    C c;
    
    // Wrapping member function of C
    function<int(C&, int, int)> calc = &C::f;
    
    // Call the member function using function
  	if (calc)
    	cout << "Product: " << calc(c, 4, 5);
  	else
      	cout << "No Callable Assigned";
    return 0;
}
```

### Composing Two Functions into One

```cpp
#include <bits/stdc++.h>
using namespace std;

// Composed function
function<int(int)> cf(function<int(int)> f1,
                        function<int(int)> f2) {
  
  	// Returning a lambda expression that
  	// in turn returns a function
    return [f1, f2](int x) {
      
      	// Apply f1 first, then f2
        return f2(f1(x)); 
    };
}

int main() {
    auto add = [](int x) { return x + 2; };
    auto mul = [](int x) { return x * 3; };

    function<int(int)> calc = cf(add, mul);
    cout << calc(4);
    return 0;
}
```
In this example, we composted two lambda functions add() and mul() into one function cf() which takes two function wrappers and returns another function wrapper. Then we created a wrapper for this using this composed function and use it to perform the addition and multiplication operator at one call.

---

**NOTE**:std::function has hidden cost so be careful to use it in your hotpath.
Calls through std::function are indirect (type-erased), so they cannot be inlined. It may also allocate on the heap if the callable doesn’t fit its small internal buffer (small object optimization).

---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>