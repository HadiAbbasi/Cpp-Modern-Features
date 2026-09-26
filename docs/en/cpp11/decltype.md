<div align="right">

[🇺🇸 English](./decltype.md) | [🇮🇷 فارسی](../../fa/cpp11/decltype.md)

</div>

# decltype
decltype is a keyword introduced in C++11 that infers the type of a given expression. Although it can be used instead of auto, it is mainly used for return types.

**The decltype keyword determines the type of an expression at compile time and uses that type for variable declarations.**

>**Why use decltype?**
When declaring types that are difficult or impossible to declare using standard notation, like lambda-related types or types that depend on template parameters


```cpp
int sum(int a, int b)
{
  using ret = int;
  ret x = a + b;
  return x;
}

```
use dceltype:

```cpp
int sum(int a, int b)
{
  using ret = decltype(a + b);
  ret x = a + b;
  return x;
}
```
The compiler evaluates the types of a + b and infers that the type of the resulting expression is int, and aliases ret as an int.

## Variable declaration using decltype

### char,int variable

```cpp
#include <iostream>
int main(){
    char c;
    decltype(c) c1 = 10;
    std::cout << c1;                     //10
    std::cout << typeid(c1).name();      //char

    int a;
    decltype(a) b = 10;
    std::cout << typeid(b).name();      //int
}
```

### Lambda

```cpp
#include <iostream>
int main(){
    auto f = [](int a, int b)->int {
        return a*b;
    };
    decltype(f) c = f;
    std::cout << typeid(c).name();                //int
    std::cout << c(2,3);                          //6
}
```

### Template Variable

```cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}

template <typename T, typename U>
auto add1(T a, U b) -> decltype(a + b) {
    return a + b;
}
int main() {
    decltype(add(2, 3)) a = 1;
    decltype(add(2.1, 3.1)) b = 2.3;
    cout << typeid(a).name() << "\n";       //int
    cout << a + b << "\n";                  //3.3

    decltype(add1(2, 1.1)) c = 3;
    cout<<<< typeid(c).name() << "\n";       //double
}
```
## decltype auto

auto does not deduces the type correctly, decltype(auto) does.

### cv-qualifiers(const, volatile)
auto does not keep cv qualifiers
``` cpp
int main() {
    const int a = 10;
    auto a1 = a;
    cout << typeid(a1).name() << "\n";      //int
}
```
decltype(auto) keeps the cv qualifiers

```cpp
int main() {
    const int a = 10;
    decltype(auto) a1 = 2;
    if (is_same <int&, decltype(d)>::value) {
        cout << "yes";                            // yes
    }
    cout << typeid(a1).name() << "\n";      //const int
}
```
### int&& (R value reference)

```cpp
int main() {
    int&& a = 0;    //R value reference
    auto a1 = move(a);
    cout << typeid(a1).name() << "\n";      //int
    if (is_same <decltype(a1), int&&>::value) {
        cout << "yes";
    } else {
        cout << "No";
    }
}
/*
Output:
No
*/
```

```cpp
int main() {
    int &&a = 0;
    decltype(auto) a2 = move(a);
    cout << typeid(a1).name() << "\n";      //int &&
    if (is_same <decltype(a2), int&&>::value)
        cout << "yes";
    else
        cout << "no";
}
/*
Output:
yes
*/
```

### Function Return type

auto does not keep function return type

```cpp
auto fun (int &i) { // Function return type is int&, but auto returned int
    return i;
}
int main() {
    int a = 1;
    if (is_same <decltype(fun(a)), int&>::value) {
        cout << "yes" << endl;
    } else {
        cout << "no" << endl;
    }
}
/*
Output:
No
*/
```

```cpp
decltype(auto) fun1 (int &i) {
    return i;
}
int main() {
    auto a2 = fun1(a);
    if (is_same <decltype(a2), int&>::value) {
        cout << "yes";
    }
}
/*
Output:
yes
*/
```



---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>