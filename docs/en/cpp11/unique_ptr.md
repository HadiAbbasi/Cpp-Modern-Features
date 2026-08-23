<div align="center">

[🇺🇸 English](./unique_ptr.md) | [🇮🇷 فارسی](../../fa/cpp11/unique_ptr.md)

</div>

----

# A Complete Guide to `std::unique_ptr` in C++11: Unique Ownership and Smart Memory Management

## Table of Contents

- [Introduction](#introduction)
- [What Was the Problem Before C++11?](#what-was-the-problem-before-c11)
- [Exceptions Made the Problem Worse](#exceptions-made-the-problem-worse)
- [What Is the Solution?](#what-is-the-solution)
- [What Is unique_ptr?](#what-is-unique_ptr)
- [Why Is It Named unique?](#why-is-it-named-unique)
- [Transferring Ownership with move](#transferring-ownership-with-move)
- [Why Is There No copy but move?](#why-is-there-no-copy-but-move)
- [What Is reset?](#what-is-reset)
- [reset with a New Object](#reset-with-a-new-object)
- [Can a unique_ptr Be Empty?](#can-a-unique_ptr-be-empty)
- [Using unique_ptr in a setter](#using-unique_ptr-in-a-setter)
- [Why Taking unique_ptr by Value Can Be Good Design](#why-taking-unique_ptr-by-value-can-be-good-design)
- [What Should a getter for unique_ptr Look Like?](#what-should-a-getter-for-unique_ptr-look-like)
- [What If We Just Want to Give Access to the Object?](#what-if-we-just-want-to-give-access-to-the-object)
- [A Better getter for an Object That Always Exists](#a-better-getter-for-an-object-that-always-exists)
- [getter for Read-Only Access](#getter-for-read-only-access)
- [Is Returning a raw pointer from a getter Dangerous?](#is-returning-a-raw-pointer-from-a-getter-dangerous)
- [Does get Cause a Memory Leak?](#does-get-cause-a-memory-leak)
- [How Can release Cause a Memory Leak?](#how-can-release-cause-a-memory-leak)
- [Don't Confuse get and release](#dont-confuse-get-and-release)
- [Passing unique_ptr to a Function](#passing-unique_ptr-to-a-function)
- [What If the Function Just Wants to Use the Object?](#what-if-the-function-just-wants-to-use-the-object)
- [Using unique_ptr in a Class](#using-unique_ptr-in-a-class)
- [unique_ptr and the destructor](#unique_ptr-and-the-destructor)
- [Transferring Ownership Between Classes](#transferring-ownership-between-classes)
- [Can We Return a unique_ptr from a getter?](#can-we-return-a-unique_ptr-from-a-getter)
- [The Right getter Based on Your Needs](#the-right-getter-based-on-your-needs)
- [Why Shouldn't We Expose unique_ptr from a Class?](#why-shouldnt-we-expose-unique_ptr-from-a-class)
- [unique_ptr and const](#unique_ptr-and-const)
- [unique_ptr and polymorphism](#unique_ptr-and-polymorphism)
- [unique_ptr for Arrays](#unique_ptr-for-arrays)
- [Does unique_ptr Itself Consume Much Memory?](#does-unique_ptr-itself-consume-much-memory)
- [What Is a custom deleter?](#what-is-a-custom-deleter)
- [A Very Important Note: make_unique](#a-very-important-note-make_unique)
- [A Common Mistake: Creating Multiple unique_ptrs from One raw pointer](#a-common-mistake-creating-multiple-unique_ptrs-from-one-raw-pointer)
- [Common Mistake: Deleting the Result of get](#common-mistake-deleting-the-result-of-get)
- [Common Mistake: Holding a raw pointer for a Long Time](#common-mistake-holding-a-raw-pointer-for-a-long-time)
- [When Should We Use unique_ptr?](#when-should-we-use-unique_ptr)
- [Differences Between unique_ptr and shared_ptr](#differences-between-unique_ptr-and-shared_ptr)
- [An Important Rule in Modern C++ Design](#an-important-rule-in-modern-c-design)
- [A Complete Example](#a-complete-example)
- [Final Summary](#final-summary)
- [Contributions](#contributions)

---

## Introduction

`std::unique_ptr` is one of the most important memory management tools in modern C++, introduced into the language with the C++11 standard.

`unique_ptr` is designed for situations where an object in dynamic memory has exactly **one specific owner**.

The core behavior of this pointer type is very simple: as long as the `unique_ptr` is alive, it is considered the owner of the object, and when it is destroyed, it automatically frees and destroys the object it owns.

---

## What Was the Problem Before C++11?

Memory management in old C++ was usually done with `new` and `delete`.

The following statement is a simple example of this approach:

```cpp
MyClass* ptr = new MyClass();

// use ptr

delete ptr;
```

The main problem with this approach is that the programmer bears the full responsibility for freeing the memory.

If the following statement is forgotten, it causes a memory leak:

```cpp
MyClass* ptr = new MyClass();

// use ptr

// delete ptr; has been forgotten
```

Another case that becomes problematic is the existence of multiple exit paths from a function.

Consider the following:

```cpp
void process()
{
    MyClass* ptr = new MyClass();

    if (someCondition())
        return;

    doSomething();

    delete ptr;
}
```

The problem in this example is that if `someCondition()` returns `true`, the function exits before reaching `delete`.

The result is that the memory allocated with `new` is not freed, causing a Memory Leak.

---

## Exceptions Made the Problem Worse

Another major problem is the existence of exceptions.

Take a look at the following:

```cpp
void process()
{
    MyClass* ptr = new MyClass();

    doSomething(); // may throw an Exception

    delete ptr;
}
```

The problem is that if `doSomething()` throws an exception, the normal execution of the function is interrupted and `delete ptr` may never be executed.

The result can be a memory leak.

---

# What Is the Solution?

RAII is one of the very important principles of C++.

RAII stands for `Resource Acquisition Is Initialization`.

The idea of RAII is to tie the ownership of a resource to the lifetime of an object.

To put it more simply:

> When the owning object is created, it acquires the resource, and when the object is destroyed, it releases the resource.

`std::unique_ptr` works based on exactly this principle.

---

# What Is unique_ptr?

`std::unique_ptr<T>` is a smart pointer that owns an object of type `T`.

The following statement creates a `unique_ptr`:

```cpp
#include <memory>

std::unique_ptr<MyClass> ptr(new MyClass());
```

However, the recommended approach in C++14 and later is to use `std::make_unique`:

```cpp
auto ptr = std::make_unique<MyClass>();
```

The important point is that the `unique_ptr` itself is responsible for `delete`-ing the object.

The following ensures that when `ptr` goes out of scope, the object is freed as well:

```cpp
void process()
{
    auto ptr = std::make_unique<MyClass>();

    // use ptr
}
```

The point of this code is that we no longer need to write `delete`.

---

# Why Is It Named unique?

The word `unique` refers to unique ownership. The following is not allowed:

```cpp
auto ptr1 = std::make_unique<MyClass>();

auto ptr2 = ptr1; // Error
```

The reason for this error is that if two `unique_ptr`s own one object, it is not clear which one should free it.

---

# Transferring Ownership with move

A `unique_ptr` cannot be copied, but it can be transferred.

The following transfers ownership from `ptr1` to `ptr2`:

```cpp
auto ptr1 = std::make_unique<MyClass>();

auto ptr2 = std::move(ptr1);
```

The important point is that after this operation, `ptr2` owns the object.

The state of `ptr1` is such that it no longer owns the object, and its value will usually be `nullptr`.

The following is suitable for checking that:

```cpp
if (ptr1 == nullptr)
{
    // ptr1 no longer owns the object
}
```

---

# Why Is There No copy but move?

The concept of copy is that after copying, both objects are independent and valid.

The following statement has no valid meaning for `unique_ptr` and causes a compiler error:

```cpp
auto ptr2 = ptr1;
```

move is different; in a move, ownership is transferred from one object to another.

The standard form is:

```cpp
auto ptr2 = std::move(ptr1);
```

---

# What Is reset?

The `reset()` method is used to change ownership or to free the current object.

The following frees the current object:

```cpp
ptr.reset();
```

The meaning of this operation is that if `ptr` owns an object, that object is destroyed and `ptr` becomes empty.

Checking its state looks like this:

```cpp
if (!ptr)
{
    // ptr is empty
}
```

---

# reset with a New Object

The `reset` method can also give ownership of a new object to the `unique_ptr`.

A sample statement looks like this:

```cpp
ptr.reset(new MyClass());
```

The meaning of this code is that if `ptr` previously owned another object, the old object is freed first, and then ownership of the new object is given to `ptr`.

The better approach in modern C++ is to use `make_unique` whenever possible.

The more suitable statement for the initial creation is:

```cpp
auto ptr = std::make_unique<MyClass>();
```

---

# Can a unique_ptr Be Empty?

A `unique_ptr` may, at any moment, own no object at all.

The following creates an empty `unique_ptr`:

```cpp
std::unique_ptr<MyClass> ptr;
```

Checking whether it is empty is also simple:

```cpp
if (!ptr)
{
    // ptr is empty
}
```

---

# Using unique_ptr in a setter

A setter method for receiving a `unique_ptr` must be designed with ownership in mind.

A common and explicit way to transfer ownership is to receive the `unique_ptr` as `std::unique_ptr&&`, but the correct way to define a setter is to take it By Value! Like this:

```cpp
class Owner
{
private:
    std::unique_ptr<MyClass> value;

public:
    void setValue(std::unique_ptr<MyClass> ptr)
    {
        value = std::move(ptr);
    }
};
```

The meaning of this design is that the caller must explicitly transfer ownership:

```cpp
auto obj = std::make_unique<MyClass>();

owner.setValue(std::move(obj));
```

The important point is that after this transfer, `owner` will own the object and `obj` will no longer own it.

---

# Why Taking unique_ptr by Value Can Be Good Design

Taking a `unique_ptr` by value in a setter has the advantage that the function clearly states:

> This function takes ownership of an object.

So a proper setter looks like this:

```cpp
void setValue(std::unique_ptr<MyClass> ptr)
{
    value = std::move(ptr);
}
```

Another point is that if the caller wants to transfer ownership, they are forced to write `std::move`.

Therefore the following statement is completely explicit:

```cpp
owner.setValue(std::move(obj));
```

---

# What Should a getter for unique_ptr Look Like?

The getter method is one of the important parts of class design.

The important point is that we should not expose the `unique_ptr` itself directly to the caller merely because the class member is a `unique_ptr`.

The following is usually not a good choice:

```cpp
std::unique_ptr<MyClass>& getValue()
{
    return value;
}
```

The problem with this design is that the caller can change the class's internal ownership.

For example, the following could happen:

```cpp
owner.getValue().reset();
```

The meaning of this operation is that the caller was able to destroy the object owned by the class.

---

# What If We Just Want to Give Access to the Object?

The more appropriate approach is to **keep ownership inside the class and only provide access to the object.**

The following is a good choice for a nullable member:

```cpp
MyClass* getValue()
{
    return value.get();
}
```

The `get()` function can return the raw pointer related to the object, but it does not transfer ownership.

The caller's code can look like this:

```cpp
MyClass* ptr = owner.getValue();

if (ptr)
{
    ptr->doSomething();
}
```

What matters is that the caller does not own `ptr` and must not free it with `delete`.

---

# A Better getter for an Object That Always Exists

If the class guarantees that the object always exists, using a reference can be a better design. A sample statement looks like this:

```cpp
MyClass& getValue()
{
    return *value;
}
```

Using it is also simple:

```cpp
owner.getValue().doSomething();
```

The advantage of this approach is that the caller does not deal with `nullptr`.

The important condition is that `value` must truly always be valid.

---

# getter for Read-Only Access

If we do not want the caller to be able to modify the internal object, `const` is a suitable option.

A sample statement looks like this:

```cpp
const MyClass& getValue() const
{
    return *value;
}
```

The meaning of this design is that the caller can read the object, but cannot modify it.

---

# Is Returning a raw pointer from a getter Dangerous?

A raw pointer is not a problem in and of itself.

The problem arises when the caller assumes that the raw pointer owns the object. The following dangerous statement must never be done:

```cpp
MyClass* ptr = owner.getValue();

delete ptr;
```

Here the problem is that `ptr` does not own the object, and the `unique_ptr` still believes it owns that object.

The result can be a `double delete` — meaning the heap object being deleted twice — and undefined behavior.

---

# A Greater Danger of raw pointers: dangling pointers

Another danger arises when we hold on to a raw pointer and the main owner of the object is destroyed. A sample statement:

```cpp
MyClass* ptr = owner.getValue();

owner.reset();

ptr->doSomething(); // dangerous
```

Here, after `reset`, the object has most likely been deleted, and `ptr` now points to memory that no longer belongs to that object. Such a pointer is called a `dangling pointer`.

---

# Does get Cause a Memory Leak?

What matters is that `get()` itself does not cause a memory leak. The following only hands a non-owning raw pointer to the caller:

```cpp
MyClass* ptr = value.get();
```

A memory leak usually happens when we detach ownership from the `unique_ptr` in an improper way:

```cpp
MyClass* ptr = value.release();
```

The `release()` function is completely different from `get()`. `get()` says:

> Just give me the address, keep the ownership.

`release()` says:

> Relinquish ownership and give me the raw pointer.

---

# How Can release Cause a Memory Leak?

After `release()`, the `unique_ptr` no longer owns the object. Therefore the following is dangerous:

```cpp
auto ptr = std::make_unique<MyClass>();

MyClass* raw = ptr.release();

// Raw is now being used

// delete raw has been forgotten
```

In this example, since the `unique_ptr` no longer owns the object and `delete raw` has not been performed either, the memory leaks.

If you are forced to use `release()`, you must make the responsibility of the new ownership clear:

```cpp
MyClass* raw = ptr.release();

delete raw;
```

The important recommendation is that we use `release()` only when we really want to transfer ownership to a legacy API or another system.

---

# Don't Confuse get and release

The difference between these two functions is very important.

| Function    | Ownership transferred?         | Returns raw pointer? |
| ----------- | ------------------------------: | -------------------: |
| `get()`     | No                             | Yes                  |
| `release()` | Yes, it leaves the unique_ptr  | Yes                  |
| `reset()`   | The previous ownership is freed | No                   |

A brief summary of these:

```cpp
ptr.get();      // observe only
ptr.release();  // relinquish ownership
ptr.reset();    // delete the current object
```

---

# Passing unique_ptr to a Function

If a function must receive ownership of an object, we transfer the `unique_ptr` to it.

A sample statement:

```cpp
void process(std::unique_ptr<MyClass> obj)
{
    obj->doSomething();
}
```

Calling it looks like this:

```cpp
auto obj = std::make_unique<MyClass>();

process(std::move(obj));
```

In this operation, after this call, the function has become the owner of the object.

---

# What If the Function Just Wants to Use the Object?

If the function is not supposed to take ownership, it is better not to receive a `unique_ptr` at all.

The suitable statement for an object that must exist:

```cpp
void process(const MyClass& obj)
{
    obj.doSomething();
}
```

Calling it with a `unique_ptr` is also simple:

```cpp
process(*obj);
```

If the existence of the object is optional, a non-owning raw pointer can be received:

```cpp
void process(const MyClass* obj)
{
    if (obj)
    {
        obj->doSomething();
    }
}
```

This design makes the function's API state precisely whether it receives ownership or merely uses the object.

---

# Using unique_ptr in a Class

One of the most common uses of `unique_ptr` is holding a private member in a class.

A sample statement:

```cpp
class Car
{
private:
    std::unique_ptr<Engine> engine;

public:
    Car()
        : engine(std::make_unique<Engine>())
    {
    }
};
```

In this example, `Car` owns the `Engine`.

Also, when a `Car` is destroyed, the `Engine` is automatically freed as well.

---

# unique_ptr and the destructor

One of the important advantages of this design is that we usually do not need a manual destructor.

The following is enough:

```cpp
class Car
{
private:
    std::unique_ptr<Engine> engine;
};
```

When `Car` is destroyed, the destructor of the `unique_ptr` is executed and frees the `Engine`.

The following illustrates this principle:

```text
Car
 └── unique_ptr
      └── Engine
```

Ownership in this structure is completely clear.

---

# Transferring Ownership Between Classes

One of the advantages of `unique_ptr` is that ownership can be transferred between objects.

A sample statement:

```cpp
class A
{
public:
    std::unique_ptr<MyClass> create()
    {
        return std::make_unique<MyClass>();
    }
};
```

The caller can receive ownership of the result:

```cpp
std::unique_ptr<MyClass> obj = a.create();
```

Here the transfer of ownership is completely explicit and there is no need for a manual `delete`.

---

# Can We Return a unique_ptr from a getter?

Yes, but we must keep in mind that this usually means **transferring ownership**.

A sample statement:

```cpp
std::unique_ptr<MyClass> takeValue()
{
    return std::move(value);
}
```

The name `takeValue` is deliberately chosen to show that the caller receives ownership.

The usage statement looks like this:

```cpp
auto obj = owner.takeValue();
```

After this operation, the class no longer owns the object.

---

# The Right getter Based on Your Needs

We can have a simple rule for designing getters:

```text
Read-only:
const T&

Read and modify without transferring ownership:
T&

Optional access:
T*

Optional and read-only access:
const T*

Transferring ownership:
std::unique_ptr<T>

Direct access to the unique_ptr itself:
std::unique_ptr<T>&
```

The final choice must be based on the API contract, not merely on the type of the internal member.

---

# Why Shouldn't We Expose unique_ptr from a Class?

If the class owns a resource, it is better to keep ownership inside that class as much as possible.

The unsuitable statement is something like this:

```cpp
std::unique_ptr<MyClass>& getValue();
```

This API exposes the class's internal memory-management details to the user.

The better design statement is:

```cpp
MyClass* getValue();
```

or, if it is not nullable:

```cpp
MyClass& getValue();
```

In this case the class says:

> I own the object, but I allow you to use it.

---

# unique_ptr and const

The `const` keyword can be used at several different levels.

The following statement means the object itself is not modifiable:

```cpp
std::unique_ptr<const MyClass> ptr;
```

The following statement is different from the one above:

```cpp
const std::unique_ptr<MyClass> ptr;
```

In the second case, the `unique_ptr` itself cannot be modified or transferred, but the `MyClass` object may be modifiable.

This difference is one of the important points of C++, and these two must not be confused with each other.

---

# unique_ptr and polymorphism

`unique_ptr` is also very well suited for holding polymorphic objects.

A sample statement:

```cpp
class Base
{
public:
    virtual ~Base() = default;
};

class Derived : public Base
{
};
```

Now we can write:

```cpp
std::unique_ptr<Base> obj =
    std::make_unique<Derived>();
```

The very important point is that if a derived object is going to be deleted through `Base`, the destructor of the base class must absolutely be `virtual`.

The suitable statement is:

```cpp
virtual ~Base() = default;
```

---

# unique_ptr for Arrays

`unique_ptr` also has a specialization for arrays.

A sample statement:

```cpp
std::unique_ptr<int[]> values =
    std::make_unique<int[]>(10);
```

Upon destruction, `unique_ptr` frees the array in the appropriate way.

The element access statement is also just like a normal array:

```cpp
values[0] = 10;
values[1] = 20;
```

In modern code it is usually better to use `std::array` for fixed-size arrays and `std::vector` for variable sizes, unless we really need dynamic ownership in this form.

---

# Does unique_ptr Itself Consume Much Memory?

Under normal circumstances, `unique_ptr<T>` is conceptually almost the size of a raw pointer.

The important point is that a `unique_ptr` is a small object that manages ownership and the responsibility of releasing.

If a custom deleter is used, the size of the `unique_ptr` may be different.

---

# What Is a custom deleter?

`unique_ptr` is not only for `delete`-ing objects created with `new`.

We can specify how the resource is released:

```cpp
struct FileDeleter
{
    void operator()(FILE* file) const
    {
        if (file)
            fclose(file);
    }
};

using FilePtr = std::unique_ptr<FILE, FileDeleter>;
```

This capability makes `unique_ptr` useful for resources other than memory as well; such as file handles or resources that must be freed with a specific function. In this approach, by defining a Functor operator, we were able to specify how the File should be released!

---

# A Very Important Note: make_unique

In C++14, the `std::make_unique` function was added.

The recommended statement is:

```cpp
auto ptr = std::make_unique<MyClass>();
```

The advantage of this approach is that it is both simpler and is considered the preferred choice for constructing a `unique_ptr` in modern C++ code.

If the project is truly C++11, `make_unique` does not exist in the standard library, and one can use `std::unique_ptr<T>(new T(...))` or define a suitable helper for C++11.

---

# A Common Mistake: Creating Multiple unique_ptrs from One raw pointer

We must not give one raw pointer to multiple `unique_ptr`s.

The very dangerous statement:

```cpp
MyClass* raw = new MyClass();

std::unique_ptr<MyClass> p1(raw);
std::unique_ptr<MyClass> p2(raw);
```

Now there are two independent owners, both of which think they must delete the same object in the end.

The result can be a `double delete` and undefined behavior.

---

# Common Mistake: Deleting the Result of get

The result of `get()` does not belong to the caller.

The following statement is wrong:

```cpp
auto ptr = std::make_unique<MyClass>();

MyClass* raw = ptr.get();

delete raw;
```

The `ptr` pointer still owns the same object and will later try to delete it again.

---

# Common Mistake: Holding a raw pointer for a Long Time

The raw pointer obtained from `get()` is valid only as long as the object owned by the `unique_ptr` exists.

The dangerous statement:

```cpp
MyClass* raw = ptr.get();

ptr.reset();

// raw is no longer valid
```

Therefore, if the lifetime of the raw pointer becomes longer than the main owner's, there is a possibility of a `dangling pointer`.

---

# When Should We Use unique_ptr?

When an object has exactly one owner, `unique_ptr` is usually a very good choice.

The common use cases are:

```text
Ownership of a member inside a class
Creating objects in a factory
Transferring ownership between functions
Managing dynamic resources
Managing resources with a custom deleter
Implementing polymorphism
```

If we need multiple independent owners, we must turn to `std::shared_ptr`.

If shared ownership is not really needed, using `shared_ptr` merely for convenience is usually not a good choice.

---

# Differences Between unique_ptr and shared_ptr

The main difference between these two lies in the ownership model.

| Feature           | `unique_ptr`     | `shared_ptr`    |
| ----------------- | ---------------- | --------------- |
| Ownership         | Unique           | Shared          |
| Copy              | No               | Yes             |
| Move              | Yes              | Yes             |
| Management cost   | Low              | Higher          |
| Ownership control | Simple and clear | More complex    |
| Primary use       | One owner        | Multiple owners |

A simple rule is:

> If you don't know which one to use, first check whether you really need shared ownership or not.

---

# An Important Rule in Modern C++ Design

One of the very good rules is:

> Express ownership with a smart pointer, and express non-owning access with a reference or a raw pointer.

A sample statement:

```cpp
std::unique_ptr<Engine> engine;
```

The meaning of this member is that `Car` owns the `Engine`.

The following:

```cpp
Engine* getEngine();
```

The meaning of this getter is that the caller only has access to the `Engine` and does not own it.

The following:

```cpp
std::unique_ptr<Engine> takeEngine();
```

The meaning of this API is that the caller takes over ownership.

---

# A Complete Example

A real class can be designed as follows:

```cpp
class Car
{
private:
    std::unique_ptr<Engine> engine;

public:
    Car()
        : engine(std::make_unique<Engine>())
    {
    }

    Engine* getEngine()
    {
        return engine.get();
    }

    const Engine* getEngine() const
    {
        return engine.get();
    }

    void setEngine(std::unique_ptr<Engine> newEngine)
    {
        engine = std::move(newEngine);
    }

    std::unique_ptr<Engine> takeEngine()
    {
        return std::move(engine);
    }

    void resetEngine()
    {
        engine.reset();
    }
};
```

This design provides several types of behavior in a completely explicit manner.

The `getEngine()` statement only provides access.

The `setEngine()` statement receives new ownership.

The `takeEngine()` statement transfers ownership.

The `resetEngine()` statement frees the resource.

As a result, the class's API is very transparent in terms of ownership.

---

# Final Summary

`std::unique_ptr` is a solution for expressing **unique ownership of a resource**.

The important point is that `unique_ptr` is not merely a raw pointer with a different name.

Its core concept is **managing the ownership and lifetime of the resource**.

The key statements we must remember are these:

```cpp
auto ptr = std::make_unique<T>();
```

In the statement above, an object is created and its ownership is given to `ptr`.

```cpp
ptr.get();
```

In the statement above, we only received a non-owning raw pointer.

```cpp
ptr.reset();
```

In the statement above, we freed the current object and emptied the `unique_ptr`.

```cpp
ptr.reset(new T());
```

In the statement above, we freed the current object and became the owner of a new object; although for the initial creation, `make_unique` is preferred.

```cpp
ptr.release();
```

In the statement above, we removed ownership from the `unique_ptr` and handed over the raw pointer.

```cpp
auto other = std::move(ptr);
```

In the statement above, we transferred ownership from `ptr` to `other`.

```cpp
delete ptr.get();
```

In the statement above, we violated the `unique_ptr`'s ownership and it may lead to undefined behavior.

The final point is that when using `unique_ptr`, we must always ask ourselves one question:

> **Who owns this object?**

If the answer to this question is "only this class or this variable", `unique_ptr` is usually a suitable choice.

If the answer is "multiple entities own it jointly", we should consider another ownership model such as `shared_ptr`.

If nobody owns it and we only want temporary access to an existing object, a reference or a non-owning raw pointer is usually the more suitable choice.

In the end, the most important benefit of `unique_ptr` is not that we are spared from writing `delete`; the main benefit is that **we express resource ownership explicitly, understandably, and automatically in the structure of the program.**

---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>