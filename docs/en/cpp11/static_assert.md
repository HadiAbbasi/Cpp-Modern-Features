<div align="right">

[🇺🇸 English](./static_assert.md) | [🇮🇷 فارسی](../../fa/cpp11/static_assert.md)

</div>

# Static_assert 
static_assert is a compile-time assertion mechanism introduced in C++11. It verifies conditions during compilation and generates an error if a condition is not satisfied.

- Displays custom error messages when an assertion fails.
- Helps catch programming errors before the program runs.

**Syntax**

```cpp
    static_assert(constant_expression, "error message");
```
**where:**

- constant_expression is a compile-time expression that evaluates to true or false.
- "error message" is displayed if the assertion fails.

```cpp
#include <iostream>
using namespace std;

int main()
{
    static_assert(sizeof(int) >= 4,
                  "int must be at least 4 bytes");

    cout << "Compilation successful";
    return 0;
}
```

## Compile-Time Assertions Before C++11
Before C++11, developers often used the #error directive to generate compilation errors. However, #error cannot evaluate expressions such as sizeof(), template parameters, or other compile-time computations.
```cpp
#include <iostream>
using namespace std;
#if !defined(__mbrcode)
#error "mbrCode hasn't been defined yet".
#endif
int main()
{
    return 0;
}
```
## Using static_assert with Templates
One of the most common uses of static_assert is validating template arguments.

```cpp
#include <iostream>
using namespace std;

template <class T, int Size>
class Vector {
    // Compile time assertion to check if
    // the size of the vector is greater than
    // 3 or not. If any vector is declared whose
    // size is less than 4, the assertion will fail
    static_assert(Size > 3, "Vector size is too small!");

    T m_values[Size];
};

int main()
{
    Vector<int, 4> four; // This will work
    Vector<short, 2> two; // This will fail

    return 0;
}
```
The assertion ensures that the template parameter Size is greater than 3. Since v2 uses a size of 2, compilation fails.

## Advantages of static_assert
The static_assert keyword provides several benefits over traditional compile-time checking techniques.

- Allows compile-time validation of program assumptions.
- Produces clear and customizable error messages.
- Works with templates and compile-time expressions.
- Can evaluate expressions involving sizeof, type traits, and template parameters.
- Helps detect common programming mistakes before execution.

## Using static_assert with Templates


---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>