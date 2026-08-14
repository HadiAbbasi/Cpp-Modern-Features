<div align="right">

[🇺🇸 English](./unique_ptr.md) | [🇮🇷 فارسی](../../fa/cpp11/unique_ptr.md)

</div>

---

# Complete Guide to `std::unique_ptr` in C++11 — Unique Ownership and Smart Memory Management

## Table of Contents

- [Introduction](#introduction)
- [What Was the Problem Before C++11?](#what-was-the-problem-before-c11)
- [Exceptions Made the Problem More Serious](#exceptions-made-the-problem-more-serious)
- [What Is the Solution?](#what-is-the-solution)
- [What Is unique_ptr?](#what-is-unique_ptr)
- [Why Is It Called unique?](#why-is-it-called-unique)
- [Transferring Ownership with move](#transferring-ownership-with-move)
- [Why Can We Move a unique_ptr but Not Copy It?](#why-can-we-move-a-unique_ptr-but-not-copy-it)
- [What Is reset()?](#what-is-reset)
- [reset() with a New Object](#reset-with-a-new-object)
- [Can a unique_ptr Be Empty?](#can-a-unique_ptr-be-empty)
- [Using unique_ptr in a Setter](#using-unique_ptr-in-a-setter)
- [Why Can Passing unique_ptr by Value Be a Good Design?](#why-can-passing-unique_ptr-by-value-be-a-good-design)
- [How Should a Getter for unique_ptr Be Designed?](#how-should-a-getter-for-unique_ptr-be-designed)
- [What If We Only Want to Give Access to the Object?](#what-if-we-only-want-to-give-access-to-the-object)
- [A Better Getter When the Object Always Exists](#a-better-getter-when-the-object-always-exists)
- [A Read-Only Getter](#a-read-only-getter)
- [Is Returning a Raw Pointer from a Getter Dangerous?](#is-returning-a-raw-pointer-from-a-getter-dangerous)
- [Can get() Cause a Memory Leak?](#can-get-cause-a-memory-leak)
- [How Can release() Cause a Memory Leak?](#how-can-release-cause-a-memory-leak)
- [Do Not Confuse get() and release()](#do-not-confuse-get-and-release)
- [Passing unique_ptr to a Function](#passing-unique_ptr-to-a-function)
- [What If the Function Only Wants to Use the Object?](#what-if-the-function-only-wants-to-use-the-object)
- [Using unique_ptr as a Class Member](#using-unique_ptr-as-a-class-member)
- [unique_ptr and the Destructor](#unique_ptr-and-the-destructor)
- [Transferring Ownership Between Classes](#transferring-ownership-between-classes)
- [Can We Return a unique_ptr from a Getter?](#can-we-return-a-unique_ptr-from-a-getter)
- [Choosing the Right Getter](#choosing-the-right-getter)
- [Why Should We Usually Avoid Exposing the unique_ptr Itself?](#why-should-we-usually-avoid-exposing-the-unique_ptr-itself)
- [unique_ptr and const](#unique_ptr-and-const)
- [unique_ptr and Polymorphism](#unique_ptr-and-polymorphism)
- [unique_ptr for Arrays](#unique_ptr-for-arrays)
- [Does unique_ptr Consume a Lot of Memory?](#does-unique_ptr-consume-a-lot-of-memory)
- [What Is a Custom Deleter?](#what-is-a-custom-deleter)
- [An Important Note About make_unique](#an-important-note-about-make_unique)
- [A Common Mistake: Creating Multiple unique_ptr Objects from One Raw Pointer](#a-common-mistake-creating-multiple-unique_ptr-objects-from-one-raw-pointer)
- [A Common Mistake: Deleting the Result of get()](#a-common-mistake-deleting-the-result-of-get)
- [A Common Mistake: Keeping a Raw Pointer for Too Long](#a-common-mistake-keeping-a-raw-pointer-for-too-long)
- [When Should We Use unique_ptr?](#when-should-we-use-unique_ptr)
- [unique_ptr vs shared_ptr](#unique_ptr-vs-shared_ptr)
- [An Important Rule for Modern C++ Design](#an-important-rule-for-modern-c-design)
- [A Complete Example](#a-complete-example)
- [Final Summary](#final-summary)
- [Contributors](#contributors)

## Introduction

The concept of `std::unique_ptr` is one of the most important tools for memory management in modern C++. It was introduced with the C++11 standard.

The term `unique_ptr` is designed for situations where a dynamically allocated object has exactly **one clear owner**.

The core idea is simple: as long as a `unique_ptr` is alive, it owns the object it points to. When the `unique_ptr` is destroyed, the owned object is automatically released as well.

---

## What Was the Problem Before C++11?

The concept of manual memory management in older versions of C++ was commonly based on `new` and `delete`.

The following is a simple example:

```cpp
MyClass* ptr = new MyClass();

// Use ptr

delete ptr;
```

The main problem with this approach is that the programmer is fully responsible for releasing the allocated memory.

The following code causes a memory leak if `delete` is forgotten:

```cpp
MyClass* ptr = new MyClass();

// Use ptr

// delete ptr; forgotten
```

Another problem occurs when a function has multiple exit paths.

Consider the following example:

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

The problem here is that if `someCondition()` returns `true`, the function exits before reaching `delete`.

As a result, the dynamically allocated memory is never released.

---

# Exceptions Made the Problem More Serious

Another important concept is exception handling.

Consider the following code:

```cpp
void process()
{
    MyClass* ptr = new MyClass();

    doSomething(); // May throw an exception

    delete ptr;
}
```

The problem is that if `doSomething()` throws an exception, normal execution stops and `delete ptr` may never be executed.

The result can be a memory leak.

---

# What Is the Solution?

RAII is one of the most important principles in C++.

The term RAII stands for `Resource Acquisition Is Initialization`.

The basic idea of RAII is to associate ownership of a resource with the lifetime of an object.

In simpler terms:

> When the object that owns a resource is created, it acquires the resource, and when the object is destroyed, it releases the resource.

The concept of `std::unique_ptr` is built around this principle.

---

# What Is unique_ptr?

`std::unique_ptr<T>` is a smart pointer that owns an object of type `T`.

The following creates a `unique_ptr`:

```cpp
#include <memory>

std::unique_ptr<MyClass> ptr(new MyClass());
```

However, in C++14 and later, the recommended approach is to use `std::make_unique`:

```cpp
auto ptr = std::make_unique<MyClass>();
```

The important concept is that the `unique_ptr` is responsible for deleting the object it owns.

For example:

```cpp
void process()
{
    auto ptr = std::make_unique<MyClass>();

    // Use ptr
}
```

When `ptr` goes out of scope, the object is automatically released.

There is no need to write `delete`.

---

# Why Is It Called unique?

The word `unique` refers to unique ownership.

The following code is not allowed:

```cpp
auto ptr1 = std::make_unique<MyClass>();

auto ptr2 = ptr1; // Error
```

The reason is that if two `unique_ptr` objects owned the same object, it would be unclear which one should delete it.

---

# Transferring Ownership with move

A `unique_ptr` cannot be copied, but it can be moved.

The following transfers ownership from `ptr1` to `ptr2`:

```cpp
auto ptr1 = std::make_unique<MyClass>();

auto ptr2 = std::move(ptr1);
```

After this operation, `ptr2` owns the object.

The important point is that `ptr1` no longer owns the object and is typically empty, meaning it contains `nullptr`.

You can check its state as follows:

```cpp
if (ptr1 == nullptr)
{
    // ptr1 no longer owns an object
}
```

---

# Why Can We Move a unique_ptr but Not Copy It?

Copying means that after the operation, both objects should remain valid and independent.

The following therefore does not make sense for `unique_ptr`:

```cpp
auto ptr2 = ptr1;
```

Moving is different because ownership is transferred from one object to another.

The standard operation is:

```cpp
auto ptr2 = std::move(ptr1);
```

---

# What Is reset()?

The `reset()` function is used to change ownership or release the currently owned object.

The following releases the current object:

```cpp
ptr.reset();
```

If `ptr` owns an object, that object is deleted and `ptr` becomes empty.

You can check its state as follows:

```cpp
if (!ptr)
{
    // ptr is empty
}
```

---

# reset() with a New Object

`reset()` can also make the `unique_ptr` take ownership of another object.

For example:

```cpp
ptr.reset(new MyClass());
```

If `ptr` already owns another object, the old object is first released and then ownership of the new object is assigned to `ptr`.

For modern C++, however, `make_unique` is generally preferred for object creation.

The preferred initialization syntax is:

```cpp
auto ptr = std::make_unique<MyClass>();
```

---

# Can a unique_ptr Be Empty?

A `unique_ptr` can exist without owning an object.

The following creates an empty `unique_ptr`:

```cpp
std::unique_ptr<MyClass> ptr;
```

You can check whether it is empty with:

```cpp
if (!ptr)
{
    // ptr is empty
}
```

---

# Using unique_ptr in a Setter

A setter that receives a `unique_ptr` should be designed according to the ownership semantics.

A common and clear approach is to receive the `unique_ptr` by value:

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

The idea is that the caller explicitly transfers ownership:

```cpp
auto obj = std::make_unique<MyClass>();

owner.setValue(std::move(obj));
```

After this transfer, `owner` owns the object and `obj` no longer owns it.

---

# Why Can Passing unique_ptr by Value Be a Good Design?

Receiving a `unique_ptr` by value clearly communicates that the function is going to receive ownership of an object.

For example:

```cpp
void setValue(std::unique_ptr<MyClass> ptr)
{
    value = std::move(ptr);
}
```

The caller must explicitly use `std::move` when transferring ownership:

```cpp
owner.setValue(std::move(obj));
```

This makes the ownership transfer explicit and easy to understand.

---

# How Should a Getter for unique_ptr Be Designed?

A getter is one of the most important parts of API design.

Just because a class internally contains a `unique_ptr` does not mean that the class should expose that `unique_ptr` directly.

The following is usually not a good design:

```cpp
std::unique_ptr<MyClass>& getValue()
{
    return value;
}
```

The problem is that the caller can now manipulate the ownership of the class.

For example:

```cpp
owner.getValue().reset();
```

The caller has now been given the ability to destroy the object owned by the class.

---

# What If We Only Want to Give Access to the Object?

A better approach is usually to **keep ownership inside the class and expose only access to the object**.

For a nullable object, a suitable getter is:

```cpp
MyClass* getValue()
{
    return value.get();
}
```

The `get()` function returns a raw pointer to the object without transferring ownership.

The caller can use it like this:

```cpp
MyClass* ptr = owner.getValue();

if (ptr)
{
    ptr->doSomething();
}
```

The important point is that the caller does not own `ptr` and must not delete it.

---

# A Better Getter When the Object Always Exists

If the class guarantees that the object always exists, returning a reference can be a better design.

For example:

```cpp
MyClass& getValue()
{
    return *value;
}
```

The caller can then use it as follows:

```cpp
owner.getValue().doSomething();
```

The advantage is that the caller does not have to deal with `nullptr`.

The important requirement is that `value` must actually be valid whenever the getter is called.

---

# A Read-Only Getter

If the caller should not be able to modify the internal object, `const` is useful.

For example:

```cpp
const MyClass& getValue() const
{
    return *value;
}
```

This design allows the caller to read the object but prevents modification through the returned reference.

---

# Is Returning a Raw Pointer from a Getter Dangerous?

A raw pointer itself is not necessarily a problem.

The problem occurs when the caller incorrectly assumes that the raw pointer represents ownership.

The following is dangerous:

```cpp
MyClass* ptr = owner.getValue();

delete ptr;
```

The problem is that `ptr` does not own the object. The `unique_ptr` still owns it.

As a result, the `unique_ptr` may later attempt to delete an object that has already been deleted.

This can lead to a double delete and undefined behavior.

---

# Can get() Cause a Memory Leak?

The `get()` function itself does not cause a memory leak.

The following simply creates a non-owning raw pointer:

```cpp
MyClass* ptr = value.get();
```

A memory leak usually occurs when ownership is incorrectly separated from the `unique_ptr`.

The important function here is `release()`:

```cpp
MyClass* ptr = value.release();
```

`release()` is completely different from `get()`.

`get()` means:

> Give me the address, but keep ownership.

`release()` means:

> Give up ownership and give me the raw pointer.

---

# How Can release() Cause a Memory Leak?

After calling `release()`, the `unique_ptr` no longer owns the object.

The following code is therefore dangerous:

```cpp
auto ptr = std::make_unique<MyClass>();

MyClass* raw = ptr.release();

// Use raw

// delete raw was forgotten
```

Because the `unique_ptr` is no longer the owner and `delete raw` was not performed, the allocated object leaks.

If `release()` is absolutely necessary, ownership must be transferred to another owner:

```cpp
MyClass* raw = ptr.release();

delete raw;
```

In practice, `release()` should generally only be used when ownership really needs to be transferred to an older API or another system that expects a raw owning pointer.

---

# Do Not Confuse get() and release()

The difference between these two functions is extremely important.

| Function    |                 Transfers ownership? | Returns raw pointer? |
| ----------- | -----------------------------------: | -------------------: |
| `get()`     |                                   No |                  Yes |
| `release()` | Yes, ownership leaves the unique_ptr |                  Yes |
| `reset()`   |          Releases the current object |                   No |

The short version is:

```cpp
ptr.get();      // Access only
ptr.release();  // Release ownership
ptr.reset();    // Delete the current object
```

---

# Passing unique_ptr to a Function

If a function needs to take ownership of an object, we can pass the `unique_ptr` to it.

For example:

```cpp
void process(std::unique_ptr<MyClass> obj)
{
    obj->doSomething();
}
```

The caller can transfer ownership like this:

```cpp
auto obj = std::make_unique<MyClass>();

process(std::move(obj));
```

After the call, the function owns the object.

---

# What If the Function Only Wants to Use the Object?

If a function does not need ownership, it usually should not receive a `unique_ptr`.

For an object that must exist, a reference is often more appropriate:

```cpp
void process(const MyClass& obj)
{
    obj.doSomething();
}
```

A `unique_ptr` can be passed like this:

```cpp
process(*obj);
```

If the object is optional, a non-owning raw pointer can be used:

```cpp
void process(const MyClass* obj)
{
    if (obj)
    {
        obj->doSomething();
    }
}
```

The key idea is that the function's interface should clearly communicate whether it receives ownership or merely accesses an existing object.

---

# Using unique_ptr as a Class Member

One of the most common uses of `unique_ptr` is to store it as a private class member.

For example:

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

This means that `Car` owns the `Engine`.

When a `Car` object is destroyed, the `Engine` is automatically released as well.

---

# unique_ptr and the Destructor

One major advantage of this design is that we usually do not need to write a custom destructor.

For example:

```cpp
class Car
{
private:
    std::unique_ptr<Engine> engine;
};
```

When `Car` is destroyed, the destructor of `unique_ptr` automatically releases the `Engine`.

The ownership relationship can be visualized like this:

```text
Car
 └── unique_ptr
      └── Engine
```

The ownership is explicit and easy to understand.

---

# Transferring Ownership Between Classes

One useful feature of `unique_ptr` is that ownership can be transferred between objects.

For example:

```cpp
class Factory
{
public:
    std::unique_ptr<MyClass> create()
    {
        return std::make_unique<MyClass>();
    }
};
```

The caller can receive ownership:

```cpp
std::unique_ptr<MyClass> obj = factory.create();
```

The ownership relationship is explicit and no manual `delete` is required.

---

# Can We Return a unique_ptr from a Getter?

Yes, but we must understand that doing so generally means **transferring ownership**.

For example:

```cpp
std::unique_ptr<MyClass> takeValue()
{
    return std::move(value);
}
```

The name `takeValue` is intentionally chosen because it communicates that the caller takes ownership.

The caller can use it like this:

```cpp
auto obj = owner.takeValue();
```

After this operation, the class no longer owns the object.

---

# Choosing the Right Getter

A simple rule for designing accessors is:

```text
Read-only access:
const T&

Read/write access without ownership transfer:
T&

Optional access:
T*

Optional read-only access:
const T*

Ownership transfer:
std::unique_ptr<T>

Direct access to the unique_ptr itself:
std::unique_ptr<T>&
```

The final choice should be based on the API's ownership contract, not merely on the type of the internal member.

---

# Why Should We Usually Avoid Exposing the unique_ptr Itself?

If a class owns a resource, it is generally better to keep ownership inside that class.

The following exposes too much of the class's internal memory-management design:

```cpp
std::unique_ptr<MyClass>& getValue();
```

A better design might be:

```cpp
MyClass* getValue();
```

Or, when the object is guaranteed to exist:

```cpp
MyClass& getValue();
```

The class can then communicate:

> I own this object, but I allow you to use it.

---

# unique_ptr and const

`const` can be applied at different levels.

The following means that the pointed-to object is const:

```cpp
std::unique_ptr<const MyClass> ptr;
```

The following is different:

```cpp
const std::unique_ptr<MyClass> ptr;
```

Here, the `unique_ptr` itself is const, while the `MyClass` object can still be modified.

This distinction is important and should not be overlooked.

---

# unique_ptr and Polymorphism

`unique_ptr` is also very useful for managing polymorphic objects.

Consider:

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

We can now write:

```cpp
std::unique_ptr<Base> obj =
    std::make_unique<Derived>();
```

The important point is that if a derived object is going to be deleted through a `Base` pointer, the base class should normally have a virtual destructor.

For example:

```cpp
virtual ~Base() = default;
```

Without a proper virtual destructor, deleting the derived object through the base type can result in undefined behavior.

---

# unique_ptr for Arrays

`unique_ptr` also has a specialization for arrays.

For example:

```cpp
std::unique_ptr<int[]> values =
    std::make_unique<int[]>(10);
```

The `unique_ptr` will correctly release the array when it is destroyed.

Elements can be accessed like a normal array:

```cpp
values[0] = 10;
values[1] = 20;
```

In modern C++, however, `std::array` is generally preferable for fixed-size arrays, while `std::vector` is usually preferable for dynamically sized collections, unless there is a specific reason to use `unique_ptr<T[]>`.

---

# Does unique_ptr Consume a Lot of Memory?

In the common case, a `unique_ptr<T>` is conceptually similar in size to a raw pointer.

The important point is that `unique_ptr` is a small object that provides ownership semantics and automatic cleanup.

If a custom deleter is used, however, the size of the `unique_ptr` can be different depending on the deleter type.

---

# What Is a Custom Deleter?

`unique_ptr` is not limited to objects that are released using `delete`.

We can specify how a resource should be released.

For example:

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

This capability makes `unique_ptr` useful for resources other than ordinary dynamically allocated memory, such as file handles or resources that must be released using a specific function.

---

# An Important Note About make_unique

`std::make_unique` was introduced in C++14.

The preferred modern syntax is:

```cpp
auto ptr = std::make_unique<MyClass>();
```

It is simpler and is generally the preferred way to create a `unique_ptr` in modern C++.

If the project specifically uses C++11, `make_unique` is not part of the standard library.

In that case, the following is possible:

```cpp
std::unique_ptr<MyClass> ptr(new MyClass());
```

Alternatively, a suitable `make_unique` helper can be implemented for a C++11 project.

---

# A Common Mistake: Creating Multiple unique_ptr Objects from One Raw Pointer

We must never give the same raw pointer to multiple independent `unique_ptr` objects.

The following is extremely dangerous:

```cpp
MyClass* raw = new MyClass();

std::unique_ptr<MyClass> p1(raw);
std::unique_ptr<MyClass> p2(raw);
```

Now there are two independent owners that both believe they are responsible for deleting the same object.

The result can be a double delete and undefined behavior.

---

# A Common Mistake: Deleting the Result of get()

The pointer returned by `get()` does not belong to the caller.

The following is incorrect:

```cpp
auto ptr = std::make_unique<MyClass>();

MyClass* raw = ptr.get();

delete raw;
```

The problem is that `ptr` still owns the object and will later attempt to delete it.

---

# A Common Mistake: Keeping a Raw Pointer for Too Long

A raw pointer returned by `get()` is valid only while the object owned by the `unique_ptr` remains alive.

The following is dangerous:

```cpp
MyClass* raw = ptr.get();

ptr.reset();

// raw is now potentially dangling
```

After `reset()`, the object may have been deleted and `raw` may point to invalid memory.

This is known as a **dangling pointer**.

---

# When Should We Use unique_ptr?

`unique_ptr` is usually an excellent choice when exactly one entity owns an object.

Common use cases include:

```text
Owning a class member
Creating objects in factories
Transferring ownership between functions
Managing dynamically allocated resources
Managing resources with custom deleters
Implementing polymorphism
```

If multiple independent owners are genuinely required, `std::shared_ptr` may be appropriate.

If shared ownership is not actually necessary, using `shared_ptr` simply for convenience is generally not a good design choice.

---

# unique_ptr vs shared_ptr

The main difference is the ownership model.

| Feature             | `unique_ptr`        | `shared_ptr`    |
| ------------------- | ------------------- | --------------- |
| Ownership           | Unique              | Shared          |
| Copy                | No                  | Yes             |
| Move                | Yes                 | Yes             |
| Management overhead | Low                 | Higher          |
| Ownership model     | Simple and explicit | More complex    |
| Main use case       | Single owner        | Multiple owners |

A useful rule is:

> If you are unsure which one to use, first ask whether shared ownership is genuinely necessary.

---

# An Important Rule for Modern C++ Design

One useful design principle is:

> Express ownership with smart pointers, and express non-owning access with references or raw pointers.

For example:

```cpp
std::unique_ptr<Engine> engine;
```

This means that `Car` owns the `Engine`.

The following:

```cpp
Engine* getEngine();
```

means that the caller receives access to the `Engine` but does not become its owner.

The following:

```cpp
std::unique_ptr<Engine> takeEngine();
```

means that the caller receives ownership.

---

# A Complete Example

A real class can be designed like this:

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

This design provides several clearly defined behaviors.

`getEngine()` provides access without transferring ownership.

`setEngine()` receives ownership.

`takeEngine()` transfers ownership.

`resetEngine()` releases the owned resource.

As a result, the ownership contract of the class is clear.

---

# Final Summary

`std::unique_ptr` is a tool for expressing **unique ownership of a resource**.

The important point is that `unique_ptr` is not simply a raw pointer with a different name.

Its main purpose is **ownership management and lifetime management**.

The key operations to remember are:

```cpp
auto ptr = std::make_unique<T>();
```

This creates an object and gives ownership to `ptr`.

```cpp
ptr.get();
```

This returns a non-owning raw pointer to the object.

```cpp
ptr.reset();
```

This releases the currently owned object and makes the `unique_ptr` empty.

```cpp
ptr.reset(new T());
```

This releases the current object and makes the `unique_ptr` own a new object. For initial creation, however, `make_unique` is generally preferred.

```cpp
ptr.release();
```

This removes ownership from the `unique_ptr` and returns the raw pointer.

```cpp
auto other = std::move(ptr);
```

This transfers ownership from `ptr` to `other`.

```cpp
delete ptr.get();
```

This violates the ownership model of `unique_ptr` and can lead to undefined behavior.

The most important question to ask when using `unique_ptr` is:

> **Who owns this object?**

If the answer is "exactly this class or this variable," `unique_ptr` is usually an appropriate choice.

If the answer is "multiple independent entities share ownership," a different ownership model such as `shared_ptr` may be appropriate.

If nobody should own the object and we only need temporary access to an existing object, a reference or non-owning raw pointer is often more appropriate.

The key benefit of `unique_ptr` is therefore not simply that we do not have to write `delete`.

Its real benefit is that **ownership and lifetime are expressed explicitly, safely, and automatically in the structure of the program.**

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>