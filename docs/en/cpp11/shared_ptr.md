<div align="right">

[🇺🇸 English](./shared_ptr.md) | [🇮🇷 فارسی](../../fa/cpp11/shared_ptr.md)

</div>

---

# Complete Guide to `std::shared_ptr` in C++11 — Shared Ownership, Reference Counting, and Memory Management

## Table of Contents

1. [Introduction](#introduction)
2. [What Was the Problem?](#what-was-the-problem)
3. [What Problem Does shared_ptr Solve?](#what-problem-does-shared_ptr-solve)
4. [What Is the Reference Count?](#what-is-the-reference-count)
5. [How Can We Observe the Reference Count?](#how-can-we-observe-the-reference-count)
6. [Creating a shared_ptr](#creating-a-shared_ptr)
7. [Why Is make_shared Preferred?](#why-is-make_shared-preferred)
8. [Using shared_ptr in a Setter](#using-shared_ptr-in-a-setter)
9. [Why Can Passing shared_ptr by Value Be a Good Design?](#why-can-passing-shared_ptr-by-value-be-a-good-design)
10. [Should We Pass shared_ptr by const Reference?](#should-we-pass-shared_ptr-by-const-reference)
11. [Using shared_ptr in a Getter](#using-shared_ptr-in-a-getter)
12. [Why Can Returning shared_ptr from a Getter Be Appropriate?](#why-can-returning-shared_ptr-from-a-getter-be-appropriate)
13. [Should a Getter Return shared_ptr&?](#should-a-getter-return-shared_ptr)
14. [Does Copying shared_ptr Copy the Object?](#does-copying-shared_ptr-copy-the-object)
15. [How to Duplicate a shared_ptr](#how-to-duplicate-a-shared_ptr)
16. [How to Create a Real Duplicate of the Object](#how-to-create-a-real-duplicate-of-the-object)
17. [Accessing the Object Inside shared_ptr](#accessing-the-object-inside-shared_ptr)
18. [Accessing the Object Through a Raw Pointer](#accessing-the-object-through-a-raw-pointer)
19. [Risks of Using a Raw Pointer from get()](#risks-of-using-a-raw-pointer-from-get)
20. [get() Does Not Transfer Ownership](#get-does-not-transfer-ownership)
21. [A Dangerous Mistake with Raw Pointers](#a-dangerous-mistake-with-raw-pointers)
22. [Can Accessing the Object Cause a Memory Leak?](#can-accessing-the-object-cause-a-memory-leak)
23. [The Main shared_ptr Problem: Ownership Cycles](#the-main-shared_ptr-problem-ownership-cycles)
24. [Why Can the Reference Count Fail to Reach Zero?](#why-can-the-reference-count-fail-to-reach-zero)
25. [Why Does weak_ptr Exist?](#why-does-weak_ptr-exist)
26. [shared_ptr vs weak_ptr](#shared_ptr-vs-weak_ptr)
27. [Using shared_ptr in a Class](#using-shared_ptr-in-a-class)
28. [Should a Getter Return shared_ptr?](#should-a-getter-return-shared_ptr)
29. [What If We Only Need Access to the Object?](#what-if-we-only-need-access-to-the-object)
30. [shared_ptr and Move](#shared_ptr-and-move)
31. [Copy vs Move with shared_ptr](#copy-vs-move-with-shared_ptr)
32. [reset() with shared_ptr](#reset-with-shared_ptr)
33. [Can shared_ptr Be Empty?](#can-shared_ptr-be-empty)
34. [shared_ptr and Multithreading](#shared_ptr-and-multithreading)
35. [shared_ptr and Destructors](#shared_ptr-and-destructors)
36. [A Complete Class Example](#a-complete-class-example)
37. [Returning shared_ptr from a Function](#returning-shared_ptr-from-a-function)
38. [When Should We Avoid shared_ptr?](#when-should-we-avoid-shared_ptr)
39. [shared_ptr vs unique_ptr](#shared_ptr-vs-unique_ptr)
40. [A Simple Rule for Choosing Smart Pointers](#a-simple-rule-for-choosing-smart-pointers)
41. [Important shared_ptr Rules](#important-shared_ptr-rules)
42. [Final Summary](#final-summary)

---

# Introduction

The concept of `std::shared_ptr` is one of the most important tools for memory management in modern C++. It was introduced with the C++11 standard.

The main purpose of `shared_ptr` is to manage situations where **multiple parts of a program need to share ownership of the same dynamically allocated object**.

The key difference from `unique_ptr` is that `unique_ptr` is designed for unique ownership, while `shared_ptr` is designed for shared ownership.

In simple terms, if several `shared_ptr` objects point to the same object, that object remains alive as long as **at least one owning `shared_ptr` still exists**.

---

# What Was the Problem?

Before smart pointers became widely used, dynamic memory management in C++ was commonly performed with `new` and `delete`.

A simple example is:

```cpp
MyClass* ptr = new MyClass();

// Use ptr

delete ptr;
```

The problem is that the programmer has to determine exactly when the object is no longer needed and manually call `delete`.

The problem becomes more complicated when multiple parts of a program need to use the same object.

Consider the following:

```cpp
MyClass* ptr = new MyClass();

useInModuleA(ptr);
useInModuleB(ptr);
useInModuleC(ptr);

delete ptr;
```

The important question here is:

> Who is responsible for calling `delete`?

If `ModuleA` assumes that it owns the object and deletes it, `ModuleB` and `ModuleC` may later access an invalid object.

If none of them takes responsibility for deleting the object, the memory may leak.

---

# What Problem Does shared_ptr Solve?

The concept of `shared_ptr` was introduced to handle situations where **multiple parts of a program genuinely need shared ownership of the same object**.

For example:

```cpp
auto ptr1 = std::make_shared<MyClass>();

auto ptr2 = ptr1;
auto ptr3 = ptr1;
```

Now all three `shared_ptr` objects point to the same object and participate in its ownership.

If `ptr1` is destroyed, the object is not deleted because `ptr2` and `ptr3` are still owners.

For example:

```cpp
ptr1.reset();
```

This only removes `ptr1` from the ownership relationship.

The object is destroyed only when the **last owning `shared_ptr`** releases ownership.

---

# What Is the Reference Count?

One of the central concepts behind `shared_ptr` is the **reference count**, also commonly called the **strong reference count** or **use count**.

Consider:

```cpp
auto p1 = std::make_shared<MyClass>();
```

At this point, there is one owning `shared_ptr`.

Now:

```cpp
auto p2 = p1;
```

A second owner has been created.

And:

```cpp
auto p3 = p1;
```

A third owner has been created.

Conceptually, the relationship looks like this:

```text
p1 ──┐
     │
p2 ──┼──> MyClass
     │
p3 ──┘

Owners = 3
```

When one of the `shared_ptr` objects is destroyed, the reference count is reduced.

For example, after `p2` is destroyed:

```text
p1 ──┐
     │
p3 ──┘──> MyClass

Owners = 2
```

When the number of owners reaches zero, the managed object is destroyed.

---

# How Can We Observe the Reference Count?

The current number of owning `shared_ptr` instances can be observed using `use_count()`.

For example:

```cpp
auto p1 = std::make_shared<MyClass>();

std::cout << p1.use_count();
```

Normally, the result is `1`.

After:

```cpp
auto p2 = p1;

std::cout << p1.use_count();
```

The count is normally `2`.

After:

```cpp
auto p3 = p1;

std::cout << p1.use_count();
```

The count is normally `3`.

The important point is that `use_count()` is useful for observation and debugging, but it should generally not be the foundation of important program logic.

---

# An Important Note About use_count()

The concept of `use_count()` should not normally be used to implement fragile logic such as:

```cpp
if (ptr.use_count() == 1)
{
    // Assume that I am the only owner
}
```

The problem is that ownership can change as other `shared_ptr` instances are created or destroyed.

This becomes even more important in multithreaded programs, where another thread may change the ownership state.

The better approach is to design the ownership contract explicitly instead of relying on the current value of `use_count()`.

---

# Creating a shared_ptr

In C++11, it is possible to construct a `shared_ptr` directly from `new`:

```cpp
std::shared_ptr<MyClass> ptr(new MyClass());
```

However, the preferred approach is usually `std::make_shared`:

```cpp
auto ptr = std::make_shared<MyClass>();
```

The important point is that `make_shared` has been available since C++11.

It makes the code cleaner and, in typical implementations, can also make memory allocation more efficient.

---

# Why Is make_shared Preferred?

A `shared_ptr` usually needs some additional information besides the pointer itself.

This information is stored in an implementation-managed structure commonly called the **control block**.

Conceptually, it may contain information such as:

```text
        Control Block
      ┌───────────────┐
      │ strong count  │
      │ weak count    │
      │ deleter       │
      └───────┬───────┘
              │
              ▼
           Object
```

The exact layout is implementation-dependent.

A typical implementation of `make_shared` can allocate the control block and the object in a closely related or combined allocation.

This can reduce the number of separate memory allocations.

The exact implementation details should not be relied upon, because the C++ standard does not require a particular internal layout.

---

# Using shared_ptr in a Setter

If a class is designed to maintain shared ownership of an object, a setter can receive a `shared_ptr`.

A clear design is to receive it by value:

```cpp
class Owner
{
private:
    std::shared_ptr<MyClass> value;

public:
    void setValue(std::shared_ptr<MyClass> ptr)
    {
        value = std::move(ptr);
    }
};
```

The caller can then provide an existing `shared_ptr`:

```cpp
auto obj = std::make_shared<MyClass>();

owner.setValue(obj);
```

Now both `obj` and `owner.value` participate in the shared ownership relationship.

The reference count increases accordingly.

---

# Why Can Passing shared_ptr by Value Be a Good Design?

Receiving a `shared_ptr` by value can clearly communicate that the function may keep a copy and therefore participate in ownership.

For example:

```cpp
void setValue(std::shared_ptr<MyClass> ptr)
{
    value = std::move(ptr);
}
```

The caller can write:

```cpp
owner.setValue(obj);
```

The important point is that this is different from `unique_ptr`.

With `unique_ptr`, copying is prohibited because ownership must remain unique.

With `shared_ptr`, copying is part of the intended ownership model.

---

# Should We Pass shared_ptr by const Reference?

The answer depends on what the function actually needs.

If a function only needs temporary access to the `shared_ptr` itself and does not need to create or retain another owner, it can receive:

```cpp
void process(const std::shared_ptr<MyClass>& ptr);
```

However, if the function is going to store the `shared_ptr` or otherwise create another shared owner, receiving it by value is often clearer:

```cpp
void setValue(std::shared_ptr<MyClass> ptr);
```

The important design principle is:

> The parameter type should communicate the ownership semantics of the function.

---

# Using shared_ptr in a Getter

This is an important area where `shared_ptr` differs from `unique_ptr`.

Because `shared_ptr` is specifically designed for shared ownership, returning a `shared_ptr` by value from a getter can be perfectly reasonable.

For example:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

The caller can write:

```cpp
auto obj = owner.getValue();
```

The important point is that `obj` is now another owner of the same object.

---

# Why Can Returning shared_ptr from a Getter Be Appropriate?

Returning a `shared_ptr` is appropriate when the API intends to allow the caller to **keep the object alive independently of the owning class**.

For example:

```cpp
auto obj = owner.getValue();
```

If `owner` is destroyed afterward, `obj` can still keep the managed object alive.

This is one of the fundamental reasons why returning `shared_ptr` can make sense.

---

# Should a Getter Return shared_ptr&?

In most designs, exposing a reference to the internal `shared_ptr` is unnecessary.

For example:

```cpp
std::shared_ptr<MyClass>& getValue();
```

This exposes the internal ownership mechanism of the class.

The caller could then write:

```cpp
owner.getValue().reset();
```

Now the caller can directly modify the ownership state maintained by the class.

A cleaner API is often:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

The caller receives a new `shared_ptr` that shares ownership, while the class keeps control of its own member.

---

# Does Copying shared_ptr Copy the Object?

This is one of the most important concepts to understand.

Consider:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = p1;
```

The `MyClass` object is **not copied**.

Instead, another `shared_ptr` is created that points to the same object.

Conceptually:

```text
p1 ──┐
     │
p2 ──┼──> MyClass
     │
     └── Control Block
```

Both `p1` and `p2` therefore refer to the **same object**.

---

# How to Duplicate a shared_ptr

If by "duplicate" we mean creating another `shared_ptr` that shares ownership of the same object, simply copy it:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = p1;
```

Now `p2` is another owner of the same object.

The reference count will normally become `2`:

```cpp
std::cout << p1.use_count();
```

The important point is that this is a **duplicate of the smart pointer ownership**, not a duplicate of the object.

---

# How to Create a Real Duplicate of the Object

If we actually want a completely independent object, copying the `shared_ptr` is not enough.

For example:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = std::make_shared<MyClass>(*p1);
```

Now there are two separate objects:

```text
p1 ──> MyClass #1

p2 ──> MyClass #2
```

The exact behavior depends on whether `MyClass` supports copying and what its copy constructor does.

The important point is that these objects no longer share the same managed instance.

---

# Accessing the Object Inside shared_ptr

A `shared_ptr` behaves similarly to a pointer when accessing the managed object.

For example:

```cpp
(*ptr).doSomething();
```

The more common syntax is:

```cpp
ptr->doSomething();
```

Both expressions access the object managed by the `shared_ptr`.

---

# Accessing the Object Through a Raw Pointer

A raw pointer can be obtained using `get()`:

```cpp
MyClass* raw = ptr.get();
```

The important point is that `raw` is **non-owning**.

The `shared_ptr` remains responsible for managing the lifetime of the object.

The raw pointer can be used with APIs that require raw pointers:

```cpp
if (raw)
{
    raw->doSomething();
}
```

---

# Risks of Using a Raw Pointer from get()

The most important risk is that the lifetime of the raw pointer depends on the lifetime of the managed object.

Consider:

```cpp
auto ptr = std::make_shared<MyClass>();

MyClass* raw = ptr.get();

ptr.reset();

raw->doSomething(); // Dangerous
```

If `ptr` was the last owner, the object has already been destroyed.

The `raw` pointer is now a **dangling pointer**.

Using it can result in undefined behavior.

---

# get() Does Not Transfer Ownership

The `get()` function only returns the address of the managed object.

For example:

```cpp
MyClass* raw = ptr.get();
```

This does not mean that `raw` has become an owner.

The following must therefore never be done:

```cpp
delete raw;
```

The `shared_ptr` still considers itself responsible for the object.

Later, it may attempt to delete the same object, which can result in undefined behavior.

---

# A Dangerous Mistake with Raw Pointers

One of the most dangerous mistakes is constructing multiple independent `shared_ptr` objects from the same raw pointer.

For example:

```cpp
MyClass* raw = new MyClass();

std::shared_ptr<MyClass> p1(raw);
std::shared_ptr<MyClass> p2(raw);
```

This creates two independent control blocks.

Conceptually:

```text
p1 ──> Control Block #1 ──> MyClass

p2 ──> Control Block #2 ──> MyClass
```

Both control blocks believe that they own the same object.

Eventually, both may attempt to delete it.

The result can be double deletion and undefined behavior.

The correct approach is to copy an existing `shared_ptr`:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = p1;
```

---

# Can Accessing the Object Cause a Memory Leak?

Calling `get()` itself does not cause a memory leak.

The following merely creates a non-owning raw pointer:

```cpp
auto ptr = std::make_shared<MyClass>();

MyClass* raw = ptr.get();
```

The more serious ownership-related problem with `shared_ptr` is usually not `get()` itself.

One of the most important problems is **cyclic ownership**.

---

# The Main shared_ptr Problem: Ownership Cycles

A `shared_ptr` keeps its object alive as long as its strong reference count is greater than zero.

This creates a problem if objects own each other in a cycle.

Consider:

```cpp
class B;

class A
{
public:
    std::shared_ptr<B> b;
};

class B
{
public:
    std::shared_ptr<A> a;
};
```

Now:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();

a->b = b;
b->a = a;
```

Conceptually, the ownership looks like this:

```text
a ──> A ──shared_ptr──> B
       ▲                │
       │                │
       └──shared_ptr────┘
```

`A` owns `B`.

At the same time, `B` owns `A`.

Even after the local variables `a` and `b` go out of scope, the objects can remain alive because they still own each other.

The result is a memory leak.

---

# Why Can the Reference Count Fail to Reach Zero?

The important point is that the reference counting mechanism itself is not broken.

It is doing exactly what it was designed to do.

The problem is the ownership model.

`A` says:

> I still own `B`.

And `B` says:

> I still own `A`.

As a result, neither ownership count can naturally reach zero.

This is called a **reference cycle** or **ownership cycle**.

This is one of the main situations where `weak_ptr` becomes useful.

---

# Why Does weak_ptr Exist?

A `weak_ptr` refers to an object managed by `shared_ptr`, but it **does not own that object**.

For example:

```cpp
std::weak_ptr<MyClass> weak = shared;
```

The important point is that creating a `weak_ptr` does not increase the strong ownership count.

This allows us to represent relationships that need access to an object but should not keep it alive.

For the previous example, we can change one side of the relationship:

```cpp
class B
{
public:
    std::weak_ptr<A> a;
};
```

Now `B` observes `A` without owning it.

The ownership cycle is broken.

The objects can therefore be destroyed when the real owners release them.

A deeper discussion of `weak_ptr`, control blocks, and cyclic ownership can be covered separately.

---

# shared_ptr vs weak_ptr

A `shared_ptr` means:

> I am an owner of this object.

A `weak_ptr` means:

> I can observe this object, but I do not own it.

A `weak_ptr` normally needs to be converted temporarily into a `shared_ptr` before accessing the object.

For example:

```cpp
if (auto ptr = weak.lock())
{
    ptr->doSomething();
}
```

If the object is still alive, `lock()` returns a valid `shared_ptr`.

If the object has already been destroyed, `lock()` returns an empty `shared_ptr`.

---

# Using shared_ptr in a Class

A common use of `shared_ptr` is to make a class participate in shared ownership.

For example:

```cpp
class Car
{
private:
    std::shared_ptr<Engine> engine;

public:
    explicit Car(std::shared_ptr<Engine> engine)
        : engine(std::move(engine))
    {
    }

    std::shared_ptr<Engine> getEngine() const
    {
        return engine;
    }
};
```

Now `Car` is one of the owners of the `Engine`.

The caller can use it like this:

```cpp
auto engine = std::make_shared<Engine>();

Car car(engine);

auto engine2 = car.getEngine();
```

Now multiple `shared_ptr` instances refer to the same `Engine`.

---

# Should a Getter Return shared_ptr?

If the intention is to allow the caller to keep the object alive independently of the class, returning a `shared_ptr` by value is completely reasonable.

For example:

```cpp
std::shared_ptr<Engine> getEngine() const
{
    return engine;
}
```

The caller receives another owner:

```cpp
auto engine2 = car.getEngine();
```

The reference count increases accordingly.

The important point is that this is intentional shared ownership.

---

# What If We Only Need Access to the Object?

If the caller only needs temporary access and should not become an owner, returning a `shared_ptr` may be unnecessary.

For a non-null object, a reference may be a better API:

```cpp
const Engine& getEngine() const
{
    return *engine;
}
```

This gives access without creating another owner.

If the object may be absent, a non-owning raw pointer can be appropriate:

```cpp
const Engine* getEngine() const
{
    return engine.get();
}
```

The important principle is to choose the return type based on the intended ownership and lifetime contract.

---

# shared_ptr and Move

`shared_ptr`, like `unique_ptr`, supports move operations.

For example:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = std::move(p1);
```

The state of `p1` is moved into `p2`.

After the operation, `p1` is empty and `p2` owns the object.

The important distinction is that moving a `shared_ptr` does not create an additional owner.

---

# Copy vs Move with shared_ptr

Copying a `shared_ptr` creates another owner:

```cpp
auto p2 = p1;
```

The strong reference count increases.

Moving a `shared_ptr` transfers the smart pointer's ownership state:

```cpp
auto p2 = std::move(p1);
```

The source `shared_ptr` becomes empty.

The important point is that move generally avoids the need to increment the reference count because no additional owner is being created.

---

# reset() with shared_ptr

The `reset()` function releases the ownership held by a particular `shared_ptr`.

For example:

```cpp
ptr.reset();
```

After this operation, `ptr` no longer owns the object.

If it was the last owner, the object is destroyed.

Consider:

```cpp
auto p1 = std::make_shared<MyClass>();
auto p2 = p1;

p1.reset();
```

The object remains alive because `p2` is still an owner.

If we then execute:

```cpp
p2.reset();
```

`p2` releases the final ownership and the object can be destroyed.

---

# reset() with a New Object

`reset()` can also make a `shared_ptr` own another object:

```cpp
ptr.reset(new MyClass());
```

If `ptr` was the last owner of the previous object, the previous object is released.

For creating a new object, however, this is usually clearer:

```cpp
ptr = std::make_shared<MyClass>();
```

---

# Can shared_ptr Be Empty?

Yes.

An empty `shared_ptr` can be created like this:

```cpp
std::shared_ptr<MyClass> ptr;
```

Its state can be checked with:

```cpp
if (!ptr)
{
    // ptr is empty
}
```

Or:

```cpp
if (ptr == nullptr)
{
    // ptr is empty
}
```

An empty `shared_ptr` does not own an object.

---

# shared_ptr and Multithreading

An important distinction must be made between **thread safety of the shared ownership mechanism** and **thread safety of the managed object**.

The standard provides important thread-safety guarantees for independent `shared_ptr` objects that share ownership.

However, this does not mean that the object itself is thread-safe.

For example:

```cpp
ptr->value++;
```

If multiple threads modify `value` simultaneously, `shared_ptr` does not automatically protect that data.

The smart pointer manages lifetime and ownership.

It does not automatically synchronize access to the managed object.

If the object itself is shared between threads, appropriate synchronization mechanisms may still be required.

---

# shared_ptr and Destructors

One major advantage of `shared_ptr` is that manual `delete` is normally unnecessary.

For example:

```cpp
class Manager
{
private:
    std::shared_ptr<MyClass> value;
};
```

When a `Manager` object is destroyed, its `shared_ptr` member is destroyed as well.

If that `shared_ptr` is the last owner, the managed object is automatically destroyed.

This is another example of the RAII principle.

---

# A Complete Class Example

A class can be designed like this:

```cpp
class Manager
{
private:
    std::shared_ptr<MyClass> value;

public:
    explicit Manager(std::shared_ptr<MyClass> value)
        : value(std::move(value))
    {
    }

    void setValue(std::shared_ptr<MyClass> newValue)
    {
        value = std::move(newValue);
    }

    std::shared_ptr<MyClass> getValue() const
    {
        return value;
    }

    const MyClass* getRawValue() const
    {
        return value.get();
    }

    void resetValue()
    {
        value.reset();
    }
};
```

This class clearly exposes several different ownership operations.

`setValue()` accepts shared ownership.

`getValue()` returns another owning `shared_ptr`.

`getRawValue()` provides non-owning access.

`resetValue()` releases the ownership held by the class.

The ownership contract is therefore visible in the API.

---

# Returning shared_ptr from a Function

A function can return a `shared_ptr` to transfer or share ownership with its caller.

For example:

```cpp
std::shared_ptr<MyClass> createObject()
{
    return std::make_shared<MyClass>();
}
```

The caller can write:

```cpp
auto obj = createObject();
```

The caller now owns the returned object through `shared_ptr`.

The important point is that this is different from returning a raw pointer allocated with `new`.

The caller does not need to remember to call `delete`.

---

# A Note About Returning shared_ptr

Some developers worry that returning a `shared_ptr` means that the entire object is copied.

That is incorrect.

For example:

```cpp
return value;
```

does not copy the managed object.

It creates or transfers another smart-pointer ownership state as appropriate.

The object itself remains the same object.

---

# When Should We Avoid shared_ptr?

We should not use `shared_ptr` everywhere simply because it makes memory management easier.

`shared_ptr` introduces additional ownership machinery and usually has more overhead than `unique_ptr`.

If there is only one clear owner:

```cpp
std::unique_ptr<MyClass>
```

is often a better choice.

If several independent parts of the program genuinely need to own the object:

```cpp
std::shared_ptr<MyClass>
```

may be appropriate.

If we only need temporary access:

```cpp
MyClass&
```

or:

```cpp
MyClass*
```

may be a simpler and more accurate design.

---

# shared_ptr vs unique_ptr

The most important difference is the ownership model.

| Feature             | `unique_ptr`    | `shared_ptr`    |
| ------------------- | --------------- | --------------- |
| Ownership           | Unique          | Shared          |
| Copy                | No              | Yes             |
| Move                | Yes             | Yes             |
| Reference counting  | No              | Yes             |
| Multiple owners     | No              | Yes             |
| Management overhead | Lower           | Higher          |
| Ownership cycles    | Not applicable  | Possible        |
| Need for `weak_ptr` | Usually no      | Sometimes       |
| Main use case       | One clear owner | Multiple owners |

The simple rule is:

> If ownership can be clearly assigned to one entity, prefer `unique_ptr`.

If multiple independent entities genuinely need to keep the object alive, `shared_ptr` may be the correct choice.

---

# A Simple Comparison

A useful mental model is to think of `unique_ptr` as a single ownership key.

Only one entity owns the key.

A `shared_ptr` can be thought of as a system in which several entities hold their own ownership handles to the same resource.

As long as at least one ownership handle remains, the resource stays alive.

A `weak_ptr`, on the other hand, is like someone who knows where the resource is but does not own it.

---

# A Simple Rule for Choosing Smart Pointers

A practical rule can be summarized as follows:

```text
Unique ownership:
std::unique_ptr<T>

Shared ownership:
std::shared_ptr<T>

Non-owning, non-null access:
T&

Non-owning, nullable access:
T*

Non-owning observation of an object managed by shared_ptr:
std::weak_ptr<T>
```

This is not an absolute rule for every possible design, but it is an excellent starting point when designing C++ APIs.

---

# Important shared_ptr Rules

The `shared_ptr` ownership model is based on reference counting.

Copying a `shared_ptr` does not copy the managed object; it creates another owner of the same object.

`std::make_shared<T>()` is the preferred way to create a `shared_ptr` in C++11.

`get()` returns a non-owning raw pointer.

`reset()` releases the ownership held by a particular `shared_ptr`.

`shared_ptr` supports both copy and move operations.

`shared_ptr` can create cyclic ownership if used incorrectly.

`weak_ptr` can be used to create non-owning relationships and break ownership cycles.

A raw pointer obtained from `get()` must not be deleted manually.

We must not create multiple independent `shared_ptr` objects from the same raw pointer.

`use_count()` is useful for observation and debugging, but it should generally not be used as the foundation of application logic.

`shared_ptr` manages object lifetime; it does not automatically make the managed object thread-safe.

---

# Final Summary

The central idea behind `std::shared_ptr` is:

> **Multiple owners can share the same object, and the object remains alive until the last owning `shared_ptr` releases it.**

This behavior is implemented through reference counting and a control block associated with the shared ownership relationship.

Creating a `shared_ptr`:

```cpp
auto ptr = std::make_shared<MyClass>();
```

Creating another owner:

```cpp
auto other = ptr;
```

Moving the ownership state:

```cpp
auto other = std::move(ptr);
```

Accessing the managed object:

```cpp
ptr->doSomething();
```

Obtaining a non-owning raw pointer:

```cpp
MyClass* raw = ptr.get();
```

Releasing ownership held by one `shared_ptr`:

```cpp
ptr.reset();
```

Observing the current strong reference count:

```cpp
ptr.use_count();
```

Returning shared ownership from a getter:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

Returning non-owning access instead:

```cpp
MyClass* getValue()
{
    return value.get();
}
```

The most important point is that `shared_ptr` was not created merely to eliminate the need for writing `delete`.

Its real purpose is to explicitly represent **shared ownership and shared lifetime**.

If there is only one clear owner, `unique_ptr` is usually simpler, cheaper, and more expressive.

If multiple independent parts of a program genuinely need to keep the same object alive, `shared_ptr` can be the appropriate ownership mechanism.

If we only need temporary access and do not need ownership, a reference or non-owning raw pointer may be more appropriate.

Finally, if `shared_ptr` ownership relationships form a cycle, reference counting cannot reach zero because the objects continue to own each other.

This is where `weak_ptr` becomes important: it allows us to represent a relationship to an object **without becoming one of its owners**.

The most important question when choosing `shared_ptr` is therefore not:

> **"Can I use shared_ptr here?"**

but rather:

> **"Do multiple independent entities genuinely need shared ownership of this object?"**

If the answer is yes, `shared_ptr` may be the right tool.

If the answer is no, `unique_ptr`, a reference, or a non-owning raw pointer may provide a simpler and more precise design.

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>