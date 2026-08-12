<div align="right">

[🇺🇸 English](./std_move.md) | [🇮🇷 فارسی](../../fa/cpp11/std_move.md)

</div>

---

# Move Semantics, std::move, and the Concepts of lvalue and rvalue in C++

**Introduced in: C++11

## Introduction

Move Semantics is one of the most important features introduced in C++11. It allows the ownership of an object's resources to be transferred instead of performing an expensive copy operation whenever possible. As a result, programs can achieve better performance and lower memory usage.
To fully understand Move Semantics, it is essential to first become familiar with concepts such as lvalue, rvalue, and rvalue references.
---

## What Are lvalue and rvalue?

Simply put:

- **`lvalues`:** Expressions that refer to an object with an identifiable location in memory. They have a persistent identity and can generally be accessed again later.
- **`rvalues`:** Temporary values or objects that typically exist only within a single expression and have a short lifetime.

Example:

```cpp
int x = 10;
```

in this example:

- variable `x` is a `lvalue`
- data `10` is a `rvalue`

for example:

```cpp
x = 20; // correct
```

Here, x is an lvalue because it is an object with a well-defined location in memory.

```cpp
10 = x; // wrong
```

On the other hand, 10 is an rvalue because it is just a literal value and does not have an assignable storage location.

another example:

```cpp
int a = 3;
int b = a;
```

in this part:

- variable `a` is a `lvalue`!
- variable `b` is a `lvalue`!
- data `3` is a `rvalue`!

---

## result of function

If a function returns its result by value, the returned expression is typically an `rvalue`:

```cpp
int foo()
{
    return 5;
}
```

in this case:

```cpp
foo();
```

is An `rvalue` because the function returns a temporary value.
However, if a function returns a reference, the result can be an `lvalue`:

```cpp
int x = 0;

int& foo()
{
    return x;
}
```

in this case:

```cpp
foo();
```

is An `lvalue` because it refers to a real object.

---

## A Simple Way to recognize the Difference

for example:

```cpp
int x = 5;
```

If you can take the address of an expression, it is usually an `lvalue`:

```cpp
&x; //is correct
```

but:

```cpp
&(x + 1); // wrong
```

because `x + 1` is a temp value!

> Note: Modern C++ has a more precise value category system consisting of `lvalue`, `prvalue`, and `xvalue`.
> For an introductory understanding of Move Semantics, the simple distinction between `lvalue` and `rvalue` is sufficient. However, keep in mind that `std::move(x)` specifically produces an `xvalue`.

---

## Why Does This Distinction Exist?

compilers need to know:

- Will this object be used again later?
- Or is it just a temporary value that can be safely discarded?

This information is essential for optimization.

---

## what was wrong before C++11?

suppose that:

```cpp
std::string s = "Hello";
```

And suppose we have a function that takes its parameter by value:

```cpp
void print(std::string str) {}
```

if we call:

```cpp
print(s);
```

A copy of `s` is created to construct `str`.

Conceptually:

```text
s
 ↓ copy
str
```

now if we send a temp value:

```cpp
print("Hello");
```

Again, a temporary object could be constructed and then copied, even though that temporary object was not going to be used later. This wastes memory and time.

> In C++11 and later, by using move semantics as well as copy elision, many of these copies are eliminated or reduced.

---

## what is the idea of Move Semantics?

the idea is easy yo know:

> If the object is temporary or we no longer need it, instead of copying its resources, transfer its resources.

A conceptual example for a class that has dynamic memory:

```text
Copy:

A ------copy------> B

A -> data
B -> copy of data
```

this means we need 2 different memories!
but Move:

```text
Move:

A ----move----> B

B -> data
A -> resource released / empty / safe state
```

In a move, usually only the pointer or the ownership of the resource is transferred, and the actual data is not copied

> Note: after a move, the source object is not necessarily `nullptr` or empty. For many library classes, the standard says the source object must be "valid but unspecified".

---

## Rvalue Reference

there is a new reference type in C++11

```cpp
&&
```

we call this a **rvalue reference**

example:

```cpp
void foo(std::string&& s) {}
```

This function can accept rvalues.

for example:

```cpp
foo(std::string("Hello")); // correct
```

or:

```cpp
foo("Hello"); // correct
```

In the second case, a temporary `std::string` is constructed from `"Hello"` and passed to `foo`.

but:

```cpp
std::string s = "Hello";
foo(s); // wrong
```

Because `s` is an `lvalue` and cannot be bound directly to `std::string&&`.

For this, you can write:

```cpp
foo(std::move(s)); // correct
```

---

## Important note about the `std::string&&` parameter

Inside the function body, the parameter `s` itself is an `lvalue`, because it has a name and its address can be taken.

example:

```cpp
void foo(std::string&& s)
{
    //variable s in this part is an lvalue!
    // even though its type is std::string&&.

    std::string local = std::move(s); // request a move from s
}
```

That is, if you want to move from an rvalue reference inside a function, you usually have to use `std::move` again.

---

## what is the `std::move`?

suppose that:

```cpp
std::string a = "Hello";
```

if we have:

```cpp
std::move(a)
```

This statement alone does not move anything.

`std::move` is actually a cast that says:

> Treat this expression like an rvalue.

More precisely, `std::move` returns an rvalue reference to the object:

```cpp
std::string b = std::move(a);
```

In this case, if there is a move constructor taking a `std::string`, the move operation may be performed.

After this operation:

```text
b -> resource moved from a
a -> valid but unspecified
```

That is, `a` is still a valid object, but its exact value is unspecified and you should not rely on its previous contents.

> Important note: `std::move` only provides the "possibility" of a move. If no suitable move constructor or move assignment exists, a copy may be performed again.

---

## Move Constructor

Suppose we have a class that manages a dynamic resource:

```cpp
class Buffer
{
    int* data_ = nullptr;
    std::size_t size_ = 0;

public:
    ~Buffer()
    {
        delete[] data_;
    }

    // Move Constructor
    Buffer(Buffer&& other) noexcept
        : data_(other.data_), size_(other.size_)
    {
        other.data_ = nullptr;
        other.size_ = 0;
    }

    // Move Assignment Operator
    Buffer& operator=(Buffer&& other) noexcept
    {
        if (this != &other)
        {
            delete[] data_;

            data_ = other.data_;
            size_ = other.size_;

            other.data_ = nullptr;
            other.size_ = 0;
        }

        return *this;
    }
};
```

when we do:

```cpp
Buffer a;
Buffer b = a; // Copy Constructor
```

If the copy constructor is defined, the copy operation is performed.but:

```cpp
Buffer b = std::move(a); // Move Constructor
```

In this case, if a move constructor exists, the resources of `a` are transferred to `b`.

---

##  Move Assignment operations

The same idea exists for the assignment operator as well.

Copy Assignment:

```cpp
a = b;
```

Move Assignment:

```cpp
a = std::move(b);
```

In Move Assignment, the resources of the destination object are freed or replaced, and the resources of the source object are transferred to the destination.

---

## why the Move is faster?

Suppose we have a class that manages a large buffer:

```cpp
class Buffer
{
    int* data_;
};
```

If the buffer size is, say, 10,000,000 numbers, copy means:

1. allocating new memory
2. Copying 10 million numbers

But move can just transfer a few pointers or internal values:

```cpp
data_ = other.data_;
other.data_ = nullptr;
```

In such a case, the complexity of the operation is reduced from about `O(n)` to `O(1)`.

> Note: for small or trivially copyable objects, copy may be fast enough and move may not provide a significant benefit.

---

## a real example

```cpp
std::vector<int> v1(1000000);
std::vector<int> v2 = std::move(v1);
```

without move:

- allocating new memory
- Copying 1,000,000 numbers

with move:

- move the internal pointer
- move the size
- move the capacity

In typical implementations, this operation is very fast.

after move:

- The array `v2` now holds the previous data of `v1`.
- The array `v1` is left in a valid but unspecified state.

> Important note: Move Semantics is only safe when the source object is no longer needed. That is why C++ automatically selects the move operation only for rvalues or for expressions that have been converted to rvalues with `std::move`.

---

## Types of Parameters

| Parameter Type|Example|Description|
|---|---|---|
| Pass by value|void foo(std::string s);|The parameter is initialized by copy or move. For lvalues, usually a copy occurs, and for rvalues, usually a move or copy elision occurs.|
| Reference to non-const lvalue|void foo(std::string& s);|Accepts only non-const lvalues.|
| Const reference|void foo(const std::string& s);|Accepts both lvalues and rvalues, but the object cannot be modified through it.|
| Reference to rvalue|void foo(std::string&& s);|Accepts rvalues and is the foundation of Move Semantics.|

---

## difference between Copy و Move

### Copy Constructor

```cpp
std::string b = a;
```

Here a new object is constructed by copying from `a`.

### Move Constructor

```cpp
std::string b = std::move(a);
```

Here a new object is constructed by transferring the resources of `a`, of course if a suitable move constructor exists.

### Copy Assignment Operator

```cpp
b = a;
```

the value of `a` is copied in `b`

### Move Assignment Operator

```cpp
b = std::move(a);
```

The resources of `a` are transferred to `b`, and usually `a` is left in a valid but unspecified state.

---

## Common Mistakes

### 1. Thinking that `std::move` necessarily moves

The `std::move` statement alone does not move any data. It only casts the expression into an rvalue form.

```cpp
std::move(a); // is a type of castings
```

The move operation happens when a suitable move constructor or move assignment is called.

---

### 2. Thinking that the moved object is necessarily empty or `nullptr`

after:
```cpp
std::string b = std::move(a);
```

We should not assume that `a` is necessarily empty. For many standard classes, the state of `a` is valid but unspecified.

You can usually destroy it or assign a new value to it, but you should not rely on its previous contents.

---

### 3. reuse of an object after move

After a move, you should not use the source object as before.

```cpp
std::string a = "Hello";
std::string b = std::move(a);

```

Using a after the move may be logically wrong,
مگر اینکه فقط عملیات‌های امن مثل assign جدید انجام شود.

---

### 4. using `std::move` on a `const` object

```cpp
const std::string a = "Hello";
std::string b = std::move(a);
```

In this case, a move usually does not occur because the move constructor typically takes a non-const std::string&&. As a result, a copy may be performed instead.

In other words, since the source object is expected to become invalid or be left in a moved-from state after a move operation, move constructors and move assignment operators typically take their parameters as non-const rvalue references (&&). Passing a const object prevents the move constructor or move assignment operator from being called in many cases, because a const object cannot be modified or moved from. As a result, the copy constructor or copy assignment operator may be selected instead.
---

### 5. using `return std::move(local)`

usually it's not recommended to do that:

```cpp
std::string make()
{
    std::string s = "Hello";
    return s; // Good
}
```

but the below line is not recommended:

```cpp
std::string make()
{
    std::string s = "Hello";
    return std::move(s); // not recommended
}
```

The reason is that `return std::move(s)` may prevent optimizations such as NRVO.

---

### 6. Forgetting `noexcept`

If your class has a move constructor or move assignment, it is better to declare them `noexcept`:

```cpp
Buffer(Buffer&& other) noexcept;
Buffer& operator=(Buffer&& other) noexcept;
```

Many containers, such as `std::vector`, use move during reallocation only if the move constructor is `noexcept` or certain conditions hold. Otherwise, a copy may be performed to preserve exception safety.
---

## Summary

| Concept|Description|
|---|---|
| lvalue|An expression with a distinct identity that can usually be referred to and whose address can be taken.|
| rvalue|A temporary value or object that usually has a short lifetime and from which a Move operation can be performed.|
| prvalue|A temporary value without identity; such as 5 or the result of a function that returns its value by value.|
| xvalue|An expiring value; such as the result of std::move(x).|
| &&|The rvalue reference operator, which is the foundation of Move Semantics.|
| std::move|A function that converts an expression to an rvalue reference and does not perform any Move operation by itself.|
| Move Constructor|A constructor that creates a new object by transferring resources from an existing object.|
| Move Assignment|An assignment operator that replaces the resources of the destination object with the resources of the source object, without performing an expensive Copy.|
| Copy Constructor|A constructor that creates a new object by copying an existing object.|
| Copy Assignment|An assignment operator that replaces the contents of the destination object with a copy from the source object.|
| Moved-from object|An object on which a Move operation has been performed; it is usually valid, but its value is unspecified.|

---

## References

- https://en.cppreference.com/w/cpp/utility/move
- https://en.cppreference.com/w/cpp/language/move_constructor
- https://en.cppreference.com/w/cpp/language/value_category

Note: This article has been prepared and edited using the explanations and rewriting done by ChatGPT (OpenAI) and Qwen (Alibaba).

## 🤝 Contributers

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>[new 15.md](../../../../../Users/Hadi/Desktop/new%2015.md)[new 15.md](../../../../../Users/Hadi/Desktop/new%2015.md)