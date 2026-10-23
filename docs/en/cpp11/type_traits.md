<div align="right">

[🇺🇸 English](./type_traits.md) | [🇮🇷 فارسی](../../fa/cpp11/type_traits.md)

</div>

---

# type_traits
The **`type_traits`** library in **C++11** (located in the `<type_traits>` header) is one of the most powerful tools for **Template Metaprogramming**.

In simple terms, `type_traits` is a collection of templates that allows the compiler at **compile-time** to:

1. Inspect or query the properties of a data type (Type Inspection / Querying).
2. Modify or transform data types (Type Transformation).
3. Change code behavior or impose specific conditions based on data type properties (such as SFINAE and `static_assert`).

## Main Categories of Type Traits in C++11

The templates and structures of `type_traits` are generally divided into three main categories:

### A) Type Queries / Checks
These structures contain a static field named `value` that returns a `bool` value.

* **Checking the nature of a data type:**
  * `std::is_integral<T>::value`: Is it an integer? (`int`, `char`, `long`, etc.)
  * `std::is_floating_point<T>::value`: Is it a floating-point number? (`float`, `double`)
  * `std::is_pointer<T>::value`: Is it a pointer? (`int*`)
  * `std::is_reference<T>::value`: Is it a reference?
  * `std::is_array<T>::value`: Is it a raw C-style array?
  * `std::is_class<T>::value`: Is it a `class` or `struct`?

* **Checking type traits/properties:**
  * `std::is_const<T>::value`: Is it `const`?
  * `std::is_signed<T>::value`: Is it signed?
  * `std::is_default_constructible<T>::value`: Does it have a default constructor?
  * `std::is_copy_constructible<T>::value`: Is it copy-constructible?
  * `std::is_move_constructible<T>::value`: Is it move-constructible?

* **Checking relationships between two types:**
  * `std::is_same<T, U>::value`: Are they exactly identical?
  * `std::is_base_of<Base, Derived>::value`: Is one derived from the other?
  * `std::is_convertible<From, To>::value`: Can the first type be implicitly converted to the second type?

---

### B) Type Transformations
These structures process a type and store the resulting type in a member named `type`:

* `std::remove_const<T>::type`: Removes `const` (e.g., `const int` becomes `int`).
* `std::remove_reference<T>::type`: Removes references (e.g., `int&` or `int&&` becomes `int`).
* `std::decay<T>::type`: Simulates type transformation when passing an argument to a function by value (removes references, converts arrays to pointers, etc.).
* `std::add_pointer<T>::type`: Adds a pointer to the type.


## Practical Examples in C++11

### Example 1: Compile-Time Validation with `static_assert`
Suppose you want to write a function that accepts only integer types such as int and long:
```cpp
#include <iostream>
#include <type_traits>

template <typename T>
T add_integers_only(T a, T b) {
    // If T is not an integer type, compilation stops with this message.
    static_assert(std::is_integral<T>::value,
                  "T must be an integral type!");
    return a + b;
}

int main() {
    std::cout << add_integers_only(10, 20) << "\n"; // OK
}
```
### Example 2: Type Transformation with remove_reference
Sometimes you need to obtain a type without reference qualifiers:
```cpp 
#include <iostream>
#include <type_traits>

template <typename T>
void print_clean_type(T&& arg) {
    // The type without a reference
    using CleanType = typename std::remove_reference<T>::type;

    // Now CleanType is the type with its reference removed.
    std::cout << "Is pointer? "
              << std::is_pointer<CleanType>::value << "\n";
}

```
### Example 3: Compile-Time Overload Selection with std::enable_if (SFINAE)

One important use of type_traits in C++11 is SFINAE, which can enable or disable function overloads depending on a type’s properties:

```cpp
#include <iostream>
#include <type_traits>

// Overload 1: available only for floating-point types
template <typename T>
typename std::enable_if<
    std::is_floating_point<T>::value
>::type
printValue(T val) {
    std::cout << "Floating-point value: " << val << "\n";
}

// Overload 2: available only for integral types
template <typename T>
typename std::enable_if<
    std::is_integral<T>::value
>::type
printValue(T val) {
    std::cout << "Integer value: " << val << "\n";
}

int main() {
    printValue(42);      // Calls overload 2
    printValue(3.1415);  // Calls overload 1
}
```
### Example 4: Conditional Type Selection with std::conditional
std::conditional is like the ternary operator (? :), but it selects between types at compile time:
```cpp
#include <iostream>
#include <type_traits>

// If T is larger than 4 bytes, select double; otherwise, select float.
template <typename T>
struct SmartTypeSelector {
    using Type = typename std::conditional<
        (sizeof(T) > 4),
        double,
        float
    >::type;
};

int main() {
    SmartTypeSelector<char>::Type a = 1.5f;   // Type is float
    SmartTypeSelector<double>::Type b = 2.5;  // Usually, Type is double
}
```

## An Important Note: C++11 vs. C++14 and C++17
    
In C++11, the syntax is a little more verbose:
To access a trait’s value, use ::value:
```cpp
  std::is_same<A, B>::value
```

To access a transformed type, use typename and ::type:
```cpp
  typename std::decay<T>::type
```

C++14 added shorthand aliases ending in _t, such as std::decay_t<T>. C++17 added variable templates ending in _v, such as std::is_same_v<A, B>. These are convenient shorthand forms; the underlying type-trait concepts were already available in C++11.


## Summary
The `type_traits` library in C++11 allows you to:

1. **Inspect type properties** (`is_pointer`, `is_integral`, `is_same`, etc.).
2. **Enforce compile-time conditions** (`static_assert`).
3. **Transform types** (`remove_reference`, `decay`, etc.).
4. **Conditionally enable function signatures** (`enable_if` for SFINAE).

---

## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>