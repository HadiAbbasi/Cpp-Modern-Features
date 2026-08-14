<div align="right">

[🇺🇸 English](./auto.md) | [🇮🇷 فارسی](../../fa/cpp11/auto.md)

</div>
---

# auto type

> **Introduced in:** C++11

## Introduction

The `auto` keyword allows the compiler to automatically deduce the type of a variable from its initializer. This feature reduces boilerplate code, improves readability, and eliminates the need to write long or complex type declarations.

> **Note:** `auto` performs **compile-time type deduction**. It is **not** a dynamically typed variable.

---

## Syntax

A variable declared with `auto` **must be initialized at the point of declaration**, since the compiler uses the initializer to determine its type.

```cpp
auto p = persons[1];

p.IncIncome(100);

auto income = p.GetIncome();
```

In this example, the compiler deduces the actual types of both `p` and `income`.

---

## Using References

By default, `auto` creates a copy of the assigned value.

If you want the variable to refer to the original object, use `&`.

```cpp
auto& p = persons[1];

p.IncIncome(100);

auto income = p.GetIncome();
```

In this case, `p` becomes a reference to the element stored in `persons`, so any modification affects the original object.

---

## A Common Use Case: Iterators

One of the most common uses of `auto` is simplifying long type declarations such as STL iterators.

### Before C++11

```cpp
std::vector<int> vec = {2, 3, 4, 5};

for (std::vector<int>::iterator it = vec.begin();
     it != vec.end();
     ++it)
{
    m_vec.push_back(*it);
}
```

The iterator type is verbose and can reduce code readability.

### Since C++11

```cpp
std::vector<int> vec = {2, 3, 4, 5};

for (auto it = vec.begin();
     it != vec.end();
     ++it)
{
    m_vec.push_back(*it);
}
```

The compiler automatically replaces `auto` with the correct iterator type during compilation.

---

## Basic Examples

```cpp
auto a = 6;          // int

auto b = 9.6;        // double

auto c = a;          // int

auto str = "Hello";  // const char*

auto flag = true;    // bool
```

---

## Advantages

- Reduces boilerplate code.
- Improves readability.
- Eliminates long type declarations.
- Works well with templates and STL iterators.
- Makes generic programming cleaner and easier.

---

## Important Notes

An `auto` variable **must** have an initializer.

```cpp
auto value = 10;   // OK

auto value;        // Error
```

By default, `auto` deduces a value type.

```cpp
auto x = obj;
```

To preserve a reference, use `&`.

```cpp
auto& x = obj;
```

To preserve a constant reference, use `const auto&`.

```cpp
const auto& x = obj;
```

---

## Best Practices

Use `auto` when:

- Working with STL iterators.
- Using lambda expressions.
- Writing generic or template-based code.
- Dealing with long or complex type names.
- The type is obvious from the initializer.

Avoid using `auto` when the deduced type is not obvious to the reader, as it may reduce code clarity.

---

## Summary

The `auto` keyword, introduced in C++11, significantly improves code readability by allowing the compiler to deduce variable types automatically. It is particularly useful when working with templates, iterators, lambda expressions, and complex type declarations, while introducing **no runtime overhead** because type deduction happens entirely at compile time.

---
## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>