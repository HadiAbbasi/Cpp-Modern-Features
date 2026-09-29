<div align="center">

[🇺🇸 English](./make_unique.md) | [🇮🇷 فارسی](../../fa/cpp14/make_unique.md)

</div>

---

## Table of Contents

* [`std::make_unique` چیست؟](#what-is-stdmake_unique)
* [Why Do We Need `make_unique`?](#why-do-we-need-make_unique)
* [The Relationship Between `make_unique` and the Heap](#the-relationship-between-make_unique-and-the-heap)
* [Difference Between `new` and `make_unique`](#difference-between-new-and-make_unique)
* [General Structure of `make_unique`](#general-structure-of-make_unique)
* [Simple Example](#simple-example)
* [Why Is `make_unique` Better Than Raw `new`?](#why-is-make_unique-better-than-raw-new)
* [Memory Leak Problem with `new`](#memory-leak-problem-with-new)
* [More Important Advantage: Exception Safety](#more-important-advantage-exception-safety)
* [Difference Between `make_unique` and `unique_ptr(new ...)`](#difference-between-make_unique-and-unique_ptrnew-)
* [Ownership Transfer](#ownership-transfer)
* [Using It in Functions](#using-it-in-functions)
* [Using It as a Return Value](#using-it-as-a-return-value)
* [Creating an Object with Constructor Arguments](#creating-an-object-with-constructor-arguments)
* [Creating an Array with `make_unique`](#creating-an-array-with-make_unique)
* [What Is `make_unique_for_overwrite`?](#what-is-make_unique_for_overwrite)
* [Difference Between `make_unique` and `make_unique_for_overwrite`](#difference-between-make_unique-and-make_unique_for_overwrite)
* [Accessing the Object](#accessing-the-object)
* [Important `unique_ptr` Functions](#important-unique_ptr-functions)
* [Is `make_unique` Faster Than `new`?](#is-make_unique-faster-than-new)
* [Does `make_unique` Perform an Allocation?](#does-make_unique-perform-an-allocation)
* [`make_unique` and `shared_ptr`](#make_unique-and-shared_ptr)
* [When Should You Not Use `make_unique`?](#when-should-you-not-use-make_unique)
* [Common Mistakes](#common-mistakes)
* [Recommended Pattern in Modern C++](#recommended-pattern-in-modern-c)
* [The Most Important Difference Between `new` and `make_unique`](#the-most-important-difference-between-new-and-make_unique)
* [Complete Example](#complete-example)
* [An Important Mental Model](#an-important-mental-model)
* [Summary](#summary)
* [Contributions](#contributions)

---

# `std::make_unique` in C++: A Safe and Modern Way to Create `unique_ptr`

# What Is `std::make_unique`?

`std::make_unique` is a Function Template in the C++ Standard Library that was added to the language in C++14.

Its main purpose is very simple:

> It creates an Object, places it in Dynamic Storage, and stores the result inside a `std::unique_ptr`.

The function is available in the following Header:

```cpp
#include <memory>
```

For example:

```cpp
auto user = std::make_unique<User>();
```

Here:

```text
Heap
 └── User Object
       ↑
       │
   unique_ptr
```

The Object is dynamically created and `unique_ptr` becomes its owner.

According to the standard, for ordinary non-array types, `make_unique` is conceptually close to constructing a `unique_ptr` with `new`.

---

# Why Do We Need `make_unique`?

Before C++14, a `unique_ptr` was commonly created like this:

```cpp
std::unique_ptr<User> user(new User());
```

This code is technically correct, but it has some drawbacks.

### Exception Safety

One important advantage of `make_unique` is that modern C++ aims to place memory management directly under RAII and avoid direct use of `new` whenever possible.

The `make_unique` function was specifically designed for this pattern.

One of the main motivations behind adding it to C++ was improved safety in the presence of Exceptions.

---

# The Relationship Between `make_unique` and the Heap

`make_unique` is a standard and safer way to create an Object in Dynamic Storage and place its ownership inside a `unique_ptr`.

For example:

```cpp
auto p = std::make_unique<int>(42);
```

Conceptually, the following happens:

```text
Dynamic Storage
┌───────────────┐
│      42       │
└───────────────┘
        ↑
        │
        │ pointer
        │
┌───────────────┐
│ unique_ptr<int>│
└───────────────┘
```

When `p` goes out of Scope, the Object is destroyed and its memory is released.

Therefore, `unique_ptr` owns the Object and uses its Deleter to release the Object when it is destroyed.

---

# Difference Between `new` and `make_unique`

It is important to distinguish between these two concepts:

```cpp
new User();
```

and:

```cpp
std::make_unique<User>();
```

`new` is a **new-expression**.

Its job is to:

1. Allocate memory
2. Construct the Object
3. Return a Raw Pointer

For example:

```cpp
User* p = new User();
```

The result is:

```text
p ────────► User
```

However, the responsibility for releasing the Object also belongs to the programmer:

```cpp
delete p;
```

---

In contrast:

```cpp
auto p = std::make_unique<User>();
```

The result is:

```text
p
│
└──── unique_ptr ─────► User
```

When `p` goes out of Scope:

```text
unique_ptr destructor
        ↓
     delete User
```

Therefore, no manual `delete` is required.

If the Raw Pointer returned by `new` is lost, a Memory Leak can occur. `unique_ptr` is designed to avoid this kind of manual memory management.

---

# General Structure of `make_unique`

For an ordinary Object:

```cpp
std::make_unique<T>(arguments...);
```

For example:

```cpp
auto p = std::make_unique<int>(10);
```

or:

```cpp
auto user = std::make_unique<User>("Hadi", 42);
```

The Arguments are passed directly to the class Constructor.

For example:

```cpp
class User
{
public:
    User(std::string name, int age)
        : name(std::move(name)),
          age(age)
    {
    }

private:
    std::string name;
    int age;
};
```

Then:

```cpp
auto user =
    std::make_unique<User>("Ali", 30);
```

Conceptually, this is approximately equivalent to:

```cpp
std::unique_ptr<User>(
    new User("Ali", 30)
);
```

The standard also specifies this general concept for the overload used with ordinary Objects.

---

# Simple Example

```cpp
#include <iostream>
#include <memory>

class Person
{
public:
    Person()
    {
        std::cout << "Person created\n";
    }

    ~Person()
    {
        std::cout << "Person destroyed\n";
    }
};

int main()
{
    auto person = std::make_unique<Person>();

    std::cout << "Using person...\n";
}
```

The sequence of events is:

```text
main()
  │
  ├── make_unique
  │      │
  │      └── Create Person
  │
  ├── Use Person
  │
  └── Scope ends
           │
           └── unique_ptr destructor
                    │
                    └── Person destructor
```

Therefore, we should not write:

```cpp
delete person.get();
```

---

# Why Is `make_unique` Better Than Raw `new`?

Compare:

```cpp
auto user = new User();
```

with:

```cpp
auto user = std::make_unique<User>();
```

### With `new`

Ownership must be managed manually:

```cpp
User* user = new User();

// ...

delete user;
```

If `delete` is accidentally skipped during program execution:

```cpp
return;
```

or an Exception occurs:

```cpp
throw ...;
```

the `delete` operation may never execute.

---

### With `make_unique`

```cpp
auto user = std::make_unique<User>();
```

Lifetime management is delegated to Scope.

This is RAII:

```text
Scope starts
    ↓
Object is created
    ↓
Use Object
    ↓
Scope ends
    ↓
unique_ptr destructor
    ↓
Object destroyed
    ↓
Memory released
```

---

# Memory Leak Problem with `new`

For example:

```cpp
void foo()
{
    User* user = new User();

    doSomething();

    delete user;
}
```

As long as `doSomething()` does not suddenly throw an `Exception`, everything may work correctly.

But:

```cpp
void foo()
{
    User* user = new User();

    doSomething(); // may throw

    delete user;
}
```

If `doSomething()` suddenly throws an `Exception`, execution stops at that point and:

```cpp
delete user;
```

will not execute.

As a result:

```text
User
 ↓
memory leak
```

However:

```cpp
void foo()
{
    auto user = std::make_unique<User>();

    doSomething();
}
```

Even if an Exception occurs, Stack Unwinding causes the `unique_ptr` to be destroyed and the Object will be released.

---

# More Important Advantage: Exception Safety

This is one of the most important reasons to use `make_unique`.

Suppose we have a function:

```cpp
void process(
    std::unique_ptr<A> a,
    std::unique_ptr<B> b
);
```

We can write:

```cpp
process(
    std::make_unique<A>(),
    std::make_unique<B>()
);
```

Each Object is under RAII management from the beginning.

In contrast, using `new` directly in complex Expressions can be more dangerous in the presence of Exceptions.

Reducing such risks was one of the main motivations behind `make_unique`.

---

# Difference Between `make_unique` and `unique_ptr(new ...)`

These two:

```cpp
auto p =
    std::make_unique<User>();
```

and:

```cpp
std::unique_ptr<User> p(
    new User()
);
```

are very similar in terms of the Object that is created.

The concept of `make_unique` for a non-array type can even be approximately imagined as:

```cpp
return std::unique_ptr<T>(
    new T(std::forward<Args>(args)...)
);
```

This is also the general concept shown in the standard documentation.

However, `make_unique`:

* Is more readable
* Does not repeat the Type
* Places Ownership inside the Smart Pointer from the beginning
* Provides better Exception Safety for Temporary Objects and more complex Expressions

Therefore, in modern C++:

```cpp
std::make_unique<T>()
```

is generally preferred.

---

# Ownership Transfer

`unique_ptr` means:

> There is only one owner.

For example:

```cpp
auto p1 = std::make_unique<User>();
```

We cannot write:

```cpp
auto p2 = p1; // ERROR
```

because copying a `unique_ptr` is not allowed.

However, Ownership can be transferred:

```cpp
auto p2 = std::move(p1);
```

After that:

```text
Before:

p1 ─────► User


After:

p1 ─────► nullptr

p2 ─────► User
```

This feature is very important for modeling exclusive ownership.

`unique_ptr` is Moveable but not Copyable.

---

# Using It in Functions

For example:

```cpp
void process(std::unique_ptr<User> user)
{
    user->run();
}
```

Call it with:

```cpp
process(
    std::make_unique<User>()
);
```

Ownership is transferred to the function.

When `process` finishes, `user` goes out of Scope and the Object is destroyed.

This pattern is especially suitable when the function **takes ownership of the Object**.

---

# Using It as a Return Value

One very useful application is:

```cpp
std::unique_ptr<User> createUser()
{
    return std::make_unique<User>();
}
```

Usage:

```cpp
auto user = createUser();
```

Here, Ownership is transferred to the Caller.

No manual `delete` is required.

This is a common pattern in modern C++ API design.

---

# Creating an Object with Constructor Arguments

Suppose we have:

```cpp
class Product
{
public:
    Product(
        int id,
        std::string name,
        double price
    )
        : id(id),
          name(std::move(name)),
          price(price)
    {
    }

private:
    int id;
    std::string name;
    double price;
};
```

We can write:

```cpp
auto product =
    std::make_unique<Product>(
        10,
        "Laptop",
        1500.0
    );
```

The Arguments:

```cpp
10
"Laptop"
1500.0
```

are passed directly to the Constructor.

---

# Creating an Array with `make_unique`

`make_unique` is not limited to ordinary Objects.

There is also an overload for Arrays with a runtime size:

```cpp
auto data =
    std::make_unique<int[]>(10);
```

This creates:

```text
10 × int
```

in Dynamic Storage.

Then:

```cpp
data[0] = 10;
data[1] = 20;
data[2] = 30;
```

and:

```cpp
std::cout << data[2];
```

can be used.

For Dynamic Arrays, `unique_ptr<T[]>` provides `operator[]`.

An important point:

```cpp
std::make_unique<int[]>(10);
```

is valid.

However:

```cpp
std::make_unique<int[10]>();
```

is not supported by `make_unique`; the overload for Arrays with a fixed Bound is deleted.

---

# What Is `make_unique_for_overwrite`?

Since C++20, another function has been added:

```cpp
std::make_unique_for_overwrite
```

For example:

```cpp
auto p =
    std::make_unique_for_overwrite<int>();
```

or:

```cpp
auto buffer =
    std::make_unique_for_overwrite<char[]>(1024);
```

This function is designed for cases where the Object's initial value will immediately be overwritten by the program.

---

# Difference Between `make_unique` and `make_unique_for_overwrite`

Compare:

```cpp
auto p =
    std::make_unique<int>();
```

with:

```cpp
auto p =
    std::make_unique_for_overwrite<int>();
```

The first one **Value-Initializes** the Object.

For an `int`, this means the value will be:

```cpp
0
```

However, the `for_overwrite` version uses Default Initialization and does not initialize types such as `int` to zero.

Therefore:

```cpp
std::make_unique<int>();
```

is suitable when you want:

```text
int = 0
```

while:

```cpp
std::make_unique_for_overwrite<int>();
```

is suitable when the value will immediately be overwritten:

```cpp
auto p =
    std::make_unique_for_overwrite<int>();

*p = 42;
```

It is also useful for Buffers:

```cpp
auto buffer =
    std::make_unique_for_overwrite<char[]>(1024);

// Fill the buffer immediately
read(fd, buffer.get(), 1024);
```

`make_unique_for_overwrite` was added in C++20.

---

# Accessing the Object

For example:

```cpp
auto user =
    std::make_unique<User>();
```

There are two main ways to access the Object:

```cpp
(*user).setName("Ali");
```

or:

```cpp
user->setName("Ali");
```

The operator:

```cpp
->
```

accesses the Object owned by the `unique_ptr`.

---

# Important `unique_ptr` Functions

`make_unique` itself does not have many methods because it is a **factory function** and its result is a `unique_ptr`.

Therefore, the main functionality available after creation comes from `unique_ptr`.

The most important ones are:

```text
get()
release()
reset()
swap()
get_deleter()
operator bool
operator*
operator->
operator[]
```

Modern C++ also provides comparison capabilities and other functionality.

---

# Is `make_unique` Faster Than `new`?

This is an important point:

> `make_unique` is not inherently an Optimization technique for making Allocation faster.

For example:

```cpp
std::make_unique<User>();
```

compared with:

```cpp
std::unique_ptr<User>(
    new User()
);
```

does not necessarily result in a faster Allocation.

The main reason for `make_unique` is Exception Safety and better Ownership design, not an optimization similar to `make_shared`.

---

# Does `make_unique` Perform an Allocation?

For an ordinary Object, conceptually:

```cpp
auto p =
    std::make_unique<User>();
```

creates an Object in Dynamic Storage.

Unlike `make_shared`, `make_unique` does not have a separate Control Block mechanism.

This distinction is important.

### `unique_ptr`

```text
unique_ptr
    │
    └────► Object
```

### `shared_ptr`

Conceptually, something similar to:

```text
shared_ptr ───► Control Block
                    │
                    └──► Object
```

Therefore, `make_unique` should not be assumed to have the same Allocation and Performance characteristics as `make_shared`.

---

# `make_unique` and `shared_ptr`

For exclusive ownership:

```cpp
auto p =
    std::make_unique<User>();
```

For shared ownership:

```cpp
auto p =
    std::make_shared<User>();
```

The main difference:

```text
unique_ptr
    ↓
one owner
```

In contrast:

```text
shared_ptr
    ↓
multiple owners
```

In modern C++ design, if Shared Ownership is not actually required, `unique_ptr` is generally a simpler and more suitable choice.

---

# When Should You Not Use `make_unique`?

`make_unique` is very useful, but it is not suitable for every scenario.

### 1. The Object Does Not Need to Be Dynamic

If the Object's Lifetime is limited to the Scope:

```cpp
User user;
```

is generally preferable to:

```cpp
auto user =
    std::make_unique<User>();
```

Do not choose the Heap simply because it is "modern".

---

### 2. Ownership Is Not Needed

If you only want to point to an existing Object:

```cpp
User user;

User* p = &user;
```

there is no need to create:

```cpp
std::unique_ptr<User>
```

---

### 3. A Custom Deleter Is Required

In this case, `make_unique` is not directly suitable for creating a `unique_ptr` with a Custom Deleter.

---

### 4. Shared Ownership Is Required

If multiple parts of the program genuinely need to own the Object:

```cpp
std::shared_ptr
```

may be more appropriate.

---

# Common Mistakes

## Mistake 1: Deleting the Object Manually

Incorrect:

```cpp
auto p =
    std::make_unique<User>();

delete p.get();
```

because `p` still owns the Object.

---

## Mistake 2: Copying

Incorrect:

```cpp
auto a =
    std::make_unique<User>();

auto b = a;
```

`unique_ptr` is not Copyable.

If you need to transfer Ownership:

```cpp
auto b =
    std::move(a);
```

---

## Mistake 3: Unnecessary Use of `get()`

For example:

```cpp
auto user =
    std::make_unique<User>();

User* p = user.get();

p->run();
```

If a Raw Pointer is not required, this is simpler:

```cpp
user->run();
```

---

## Mistake 4: Unnecessary Use of the Heap

This:

```cpp
auto user =
    std::make_unique<User>();
```

is not always better than:

```cpp
User user;
```

If Dynamic Ownership is not required, an ordinary Object with automatic storage duration is often simpler.

---

## Mistake 5: Using `release()` Without a Plan for Ownership

For example:

```cpp
auto p =
    std::make_unique<User>();

User* raw = p.release();
```

From this point:

```text
unique_ptr
     └── nullptr

raw
     └────► User
```

The `unique_ptr` is no longer responsible for the Object.

Therefore, `release()` should be used deliberately.

---

# Recommended Pattern in Modern C++

For creating an Object with exclusive ownership:

```cpp
auto object =
    std::make_unique<MyClass>();
```

For a Constructor:

```cpp
auto object =
    std::make_unique<MyClass>(
        arg1,
        arg2
    );
```

For returning Ownership:

```cpp
std::unique_ptr<MyClass>
create()
{
    return std::make_unique<MyClass>();
}
```

For transferring Ownership:

```cpp
auto b = std::move(a);
```

For accessing the Object:

```cpp
a->method();
```

For checking whether the pointer owns an Object:

```cpp
if (a)
{
    a->method();
}
```

And in ordinary cases, direct use of:

```cpp
new
delete
```

is avoided.

This approach is consistent with the philosophy of RAII and the standard Smart Pointers provided by C++.

---

# The Most Important Difference Between `new` and `make_unique`

Suppose we have two overloaded functions with different parameter types:

```cpp
void process(User* user);                    // Raw Pointer
void process(std::unique_ptr<User> user);    // unique_ptr
```

If we call it with `new`:

```cpp
process(new User());
```

the overload that accepts a Raw Pointer, `User*`, is selected.

However, if we call it with `make_unique`:

```cpp
process(std::make_unique<User>());
```

the `std::unique_ptr<User>` overload is selected.

**Key point:** The type of the argument expression determines which overload is selected.

---

# Complete Example

```cpp
#include <iostream>
#include <memory>
#include <string>

class User
{
public:
    User(
        std::string name,
        int age
    )
        : name(std::move(name)),
          age(age)
    {
        std::cout << "User created\n";
    }

    ~User()
    {
        std::cout << "User destroyed\n";
    }

    void print() const
    {
        std::cout
            << name
            << " - "
            << age
            << '\n';
    }

private:
    std::string name;
    int age;
};

std::unique_ptr<User> createUser()
{
    return std::make_unique<User>(
        "Ali",
        30
    );
}

int main()
{
    auto user = createUser();

    if (user)
    {
        user->print();
    }

    auto another =
        std::make_unique<User>(
            "Reza",
            25
        );

    user.swap(another);
}
```

In this example:

```cpp
std::make_unique<User>()
```

creates the Object.

```cpp
std::unique_ptr<User>
```

manages its Ownership.

```cpp
user->print();
```

accesses the Object.

```cpp
user.swap(another);
```

swaps Ownership between the two `unique_ptr` objects.

Finally, when the Scope ends, the Objects are automatically destroyed.

---

# An Important Mental Model

If we want to summarize the entire topic with a mental model:

### The Old Manual Approach

```text
new
 │
 ▼
Object
 │
 ▼
Raw Pointer
 │
 │  must be managed manually
 │
 ▼
delete
```

Most of the responsibility for Lifetime management is placed on the programmer.

---

### The Modern C++ Approach

```text
make_unique<T>()
       │
       ▼
Dynamic Object
       ▲
       │
       │ ownership
       │
unique_ptr
       │
       ▼
Scope ends
       │
       ▼
automatic destruction
```

In other words:

> **`make_unique` is the standard and clean way to create an Object with exclusive ownership in Dynamic Storage.**

---

# Summary

`std::make_unique` should not be considered merely a "shorter replacement for `new`."

Its main concept is:

```text
Object Creation
      +
Ownership
      +
RAII
      +
Exception Safety
```

in a single pattern.

Therefore:

```cpp
std::unique_ptr<User>(
    new User(...)
);
```

is technically valid and the Object will still be automatically managed by `unique_ptr`.

However, in modern C++, it is generally preferred to use:

```cpp
std::make_unique<User>(...);
```

The important point is that `make_unique` does not necessarily provide **better Performance** than `unique_ptr(new T)`.

Its main benefits are:

* Safety
* Clearer Ownership
* Better Readability
* Fewer Memory Management Errors

It is also important to distinguish between these three concepts:

```cpp
new T
```

means:

> Create the Object in Dynamic Storage and return a Raw Pointer.

```cpp
std::unique_ptr<T>(new T)
```

means:

> Create the Object using `new` and transfer its Ownership to a `unique_ptr`.

```cpp
std::make_unique<T>()
```

means:

> Create the Object and place it into a `unique_ptr` with the standard RAII-based approach.

Therefore, in most modern C++ code, when **exclusive ownership and Dynamic Lifetime** are required, the natural choice is:

```cpp
auto object = std::make_unique<T>(...);
```

---

## 🤝 Contributions

<div align="center">

| GitHub                                      | LinkedIn                                                           | Email                                                  | Site                           | Telegram                               |
| ------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------ | -------------------------------------- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](mailto:hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>
