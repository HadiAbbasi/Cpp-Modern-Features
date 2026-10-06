<div align="right">

[🇺🇸 English](./function.md) | [🇮🇷 فارسی](../../fa/cpp11/function.md)

</div>

In C++11 we got a new reference syntax: &&. You famously know it as an rvalue reference. But in certain contexts, T&& can act as a forwarding reference (also called a universal reference) - a pattern that can bind to both lvalues and rvalues, adapting depending on what you pass to it.

A universal reference or Forwarding reference is a type of reference pattern that can bind to both lvalues and rvalues — and adapts depending on what you pass to it.

Before we talk about universal references, let’s start with something familiar — passing objects to functions. We’ll look at a few common patterns that seem fine at first but quickly break down in real-world code. These problems will naturally guide us to the idea of universal references and why C++11 introduced them.


Lets say i have a function upload that takes a Photo object as parameter and do further operation using the photo. The operation can be just upload the photo or edit the photo and upload or something else.

```cpp
void upload(Photo& p) {
    std::cout << "upload(Photo&) called\n";
    // Some operation
}

int main() {
    Photo selfie = Photo{1,2};
    upload(selfie);
}
```

Ok great its working fine. Now lets say in main function the Photo object are generated on the fly without storing in some named variable so it would be like this:
```cpp
void upload(Photo& p) {
    std::cout << "upload(Photo&) called\n";
    // Some operation
}

int main() {
    Photo selfie = Photo{1,2};
    upload(selfie);
    upload(Photo{1,2});
}

```

**faile compiler**
`error: candidate function not viable: expects lvalue as 1st argument`

This will fail because Photo{1,2} is an rvalue (temporary), but Photo& only binds to lvalues! No issues we can fix this if we can write an overload of upload function that does the same thing but accepts a rvalue reference like below:

```cpp
void upload(Photo& p) {
    std::cout << "upload(Photo&) called\n";
}

void upload(Photo&& p) {
    std::cout << "upload(Photo&&) called\n";
}

int main() {
    Photo selfie = Photo{1,2};
    upload(selfie); // calls upload(Photo&)
    upload(Photo{1,2}); // // calls upload(Photo&&)
}
```
**This works — but now imagine:**

- You also want to support const Photo
- You want this to work for many types
- You want to forward arguments to another function (Covered in the later section of this article)

**We want one function that:**

- Accepts lvalues and rvalues
- Preserves their value category 
- Preserves constness
- Works for any type

To achieve all this C++11 introduced Forwarding Reference (Universal Reference).

## Universal(Forwarding) Reference

If you see T&& where T is a deduced type (like in a function template or auto), that’s a universal reference.

### Universal (Forwarding) Reference Rule:
&& is a universal (forwarding) reference only when:

1.  It appears in a template function parameter with type deduction:

```cpp
template<typename T>
void f(T&& x);   // universal reference
```

2. It appears in auto&&:

```cpp
auto&& x = expr; // universal reference
```
3. It appears in a variadic template parameter pack:

```cpp
template<typename... Args>
void f(Args&&... args); // universal references
```
In all other cases, && means a normal rvalue reference:

```cpp
void f(int&& x);          // NOT universal
template<typename T>
void f(std::vector<T>&&); // NOT universal
const T&& x;              // NOT universal

```



## std::forward

The term “Perfect Forwarding” means forwarding arguments perfectly between functions — that is, preserving ALL of the following properties:

1. Value category (lvalue vs rvalue)
    - Lvalues remain lvalues
    - Rvalues remain rvalues
2. Const-ness
    - const objects remain const
    - Non-const objects remain non-const
3. Type information
    - The exact type is preserved through template deduction


In C++, std::forward() is a template function used for achieving perfect forwarding of arguments to functions so that it's lvalue or rvalue is preserved. It basically forwards the argument while preserving the value type of it.

std::forward() was introduced in C++ 11 as the part of `<utility>` header file.

```cpp
std::forward<T> (arg)
```
```cpp

// C++ program to illustrate the use of
// std::forward() function
#include <iostream>
#include <utility>
using namespace std;

// Function that takes lvalue reference
void UtiltyFun(int& i) {
    cout << "Process lvalue: " << i << endl;
}

// Overload of above function but it takes rvalue
// reference
void UtiltyFun(int&& i) {
    cout << "Process rvlaue: " << i << endl;
}

// Template function for forwarding arguments
// to utlityFun() 
template <typename T>
void Fun(T&& arg) {
    UtiltyFun(forward<T>(arg));
}

int main() {
    int x = 10;
  	
  	// Passing lvalue
    Fun(x);
  
  	// Passing rvalue
    Fun(move(x)); 
}
```
Here, in function fun(), T&& is called as universal reference which means it can hold both type (lvalue and rvalue). By using std::forward(), it will check what type is coming in arg. Based on whether it is lvalue or rvalue, it will call the correct overloaded version of the utilityFun().

If we don't use std::forward, it will every time call lvalue version of utilityFun(int &i), as compiler won't check the type of arg and assumed that it is lvalue only, which we will see in below example.

## Above Example Without std::forward

```cpp
// C++ Program to illustrate what will happen if we
// don't use std::forward for argument passing
#include <iostream>
#include <utility>
using namespace std;

// Function with lvalue reference parameter
void utiltyFun(int& i) {
    cout << "Process lvalue: " << i << endl;
}

// Overload of above function with rvalue
// reference arameter
void utiltyFun(int&& i) {
    cout << "Process rvlaue: " << i << endl;
}

// Function that forwards argument
template <typename T>
void fun(T&& arg) {
    utiltyFun(arg);
}


int main() {
    int x = 10;
  
  	// Passing lvalue
    fun(x); 
  
  	// Passing rvalue
    fun(move(x)); 
}
```

## Things to Remember


**In the above program**

- std::move() function will cast its arguments unconditionally, means either if we pass lvalue or rvalue it will cast it to rvalue, it doesn't move anything.
- std::forward will cast its argument to rvalue only if argument coming is rvalue.
- std::move and std::forward won't do anything in runtime.


---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>