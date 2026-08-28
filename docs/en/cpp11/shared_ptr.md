<div align="center">

[🇺🇸 English](./shared_ptr.md) | [🇮🇷 فارسی](../../fa/cpp11/shared_ptr.md)

</div>

---

# A Complete Guide to `std::shared_ptr` in C++11: Shared Ownership, Reference Counting, and the Difference from `unique_ptr`

## Table of Contents

- [Introduction](#introduction)
- [What Was the Main Problem?](#what-was-the-main-problem)
- [What Problem Does shared_ptr Solve?](#what-problem-does-shared_ptr-solve)
- [What Is the Internal Counter of shared_ptr?](#what-is-the-internal-counter-of-shared_ptr)
- [How Can the Counter Be Observed?](#how-can-the-counter-be-observed)
- [An Important Note About use_count()](#an-important-note-about-use_count)
- [Creating a shared_ptr](#creating-a-shared_ptr)
- [Why Is make_shared Better?](#why-is-make_shared-better)
- [Using shared_ptr in a Setter](#using-shared_ptr-in-a-setter)
- [Why Should a Setter Take shared_ptr by Value?](#why-should-a-setter-take-shared_ptr-by-value)
- [Should We Take shared_ptr by const Reference?](#should-we-take-shared_ptr-by-const-reference)
- [Using shared_ptr in a Getter](#using-shared_ptr-in-a-getter)
- [Why Can a Getter of Type shared_ptr Be Appropriate?](#why-can-a-getter-of-type-shared_ptr-be-appropriate)
- [Should a Getter Return shared_ptr&?](#should-a-getter-return-shared_ptr)
- [Does Copying a shared_ptr Mean Copying the Object?](#does-copying-a-shared_ptr-mean-copying-the-object)
- [How Do We Get a Duplicate of a shared_ptr?](#how-do-we-get-a-duplicate-of-a-shared_ptr)
- [What If We Want a Real Duplicate of the Object Itself?](#what-if-we-want-a-real-duplicate-of-the-object-itself)
- [Accessing the Internal Value of shared_ptr](#accessing-the-internal-value-of-shared_ptr)
- [Accessing the Object Through a Raw Pointer](#accessing-the-object-through-a-raw-pointer)
- [Risks of Using a Raw Pointer from get()](#risks-of-using-a-raw-pointer-from-get)
- [The get() Method Does Not Transfer Ownership](#the-get-method-does-not-transfer-ownership)
- [A Very Dangerous Mistake with Raw Pointers](#a-very-dangerous-mistake-with-raw-pointers)
- [Can Accessing the Inside of shared_ptr Cause a Memory Leak?](#can-accessing-the-inside-of-shared_ptr-cause-a-memory-leak)
- [The Main Problem of shared_ptr: Ownership Cycles](#the-main-problem-of-shared_ptr-ownership-cycles)
- [Why Doesn't the shared_ptr Counter Work Well in This Case?](#why-doesnt-the-shared_ptr-counter-work-well-in-this-case)
- [Why Does weak_ptr Exist?](#why-does-weak_ptr-exist)
- [What Is the Difference Between shared_ptr and weak_ptr?](#what-is-the-difference-between-shared_ptr-and-weak_ptr)
- [shared_ptr in a Class](#shared_ptr-in-a-class)
- [Is It Better for a Getter to Return shared_ptr?](#is-it-better-for-a-getter-to-return-shared_ptr)
- [What If We Only Want to Give Access to the Object?](#what-if-we-only-want-to-give-access-to-the-object)
- [An Important Difference: Getters with unique_ptr](#an-important-difference-getters-with-unique_ptr)
- [shared_ptr and Transferring with move](#shared_ptr-and-transferring-with-move)
- [The Difference Between copy and move in shared_ptr](#the-difference-between-copy-and-move-in-shared_ptr)
- [The reset Operation in shared_ptr](#the-reset-operation-in-shared_ptr)
- [The reset Operation with a New Object](#the-reset-operation-with-a-new-object)
- [Can a shared_ptr Be Made Empty?](#can-a-shared_ptr-be-made-empty)
- [shared_ptr and Multiple Threads](#shared_ptr-and-multiple-threads)
- [shared_ptr and the Destructor](#shared_ptr-and-the-destructor)
- [A Complete Example of Class Design](#a-complete-example-of-class-design)
- [A Note About Returning shared_ptr](#a-note-about-returning-shared_ptr)
- [When Is Using shared_ptr Not an Appropriate Choice?](#when-is-using-shared_ptr-not-an-appropriate-choice)
- [The Difference Between shared_ptr and unique_ptr](#the-difference-between-shared_ptr-and-unique_ptr)
- [A Simple Comparison](#a-simple-comparison)
- [When Is shared_ptr the Right Choice?](#when-is-shared_ptr-the-right-choice)
- [When Is unique_ptr Better?](#when-is-unique_ptr-better)
- [A Golden Rule for Choosing a Smart Pointer](#a-golden-rule-for-choosing-a-smart-pointer)
- [Important Notes About shared_ptr](#important-notes-about-shared_ptr)
- [Final Summary](#final-summary)
- [Contributors](#-contributors)

---

## Introduction

The `std::shared_ptr` pointer is one of the most important memory management tools in modern C++, introduced into the language with the C++11 standard.

The main role of `shared_ptr` becomes significant when an object lives in dynamic memory and **more than one part of the program must own that object**.

The `unique_ptr` pointer is designed for unique ownership, but `shared_ptr` is used when ownership must be divided among several `shared_ptr` instances in a shared manner.

To put it more simply: if several `shared_ptr` objects point to one object, the object remains alive as long as **the last owning `shared_ptr` still exists**.

---

# What Was the Main Problem?

Before smart pointers came into widespread use, manual memory management was based on `new` and `delete`.

Consider the following simple statement:

```cpp
MyClass* ptr = new MyClass();

// use ptr

delete ptr;
```

The problem with this approach is that we must determine exactly when the object is no longer needed and call `delete` at that exact moment.

The problem becomes more serious when several different parts of the program need the same object.

Imagine the following:

```cpp
MyClass* ptr = new MyClass();

useInModuleA(ptr);
useInModuleB(ptr);
useInModuleC(ptr);

delete ptr;
```

The important question here is: who should perform the `delete`?

If `ModuleA` assumes it owns the object and deletes it, `ModuleB` and `ModuleC` can no longer use it with confidence.

If none of them accepts the responsibility for deleting it, the memory will leak.

---

# What Problem Does shared_ptr Solve?

The `shared_ptr` pointer was designed for exactly this scenario; that is, when **the ownership of an object must be shared among several parts of the program**.

The following statement creates several owners for one object:

```cpp
auto ptr1 = std::make_shared<MyClass>();

auto ptr2 = ptr1;
auto ptr3 = ptr1;
```

Now all three `shared_ptr` objects point to the same object and share in its ownership.

When `ptr1` is destroyed, the object is not deleted, because `ptr2` and `ptr3` still own it.

The following statement also does not cause the object to be deleted:

```cpp
ptr1.reset();
```

The object is deleted only when **the last owner** is also destroyed or releases its ownership.

---

# What Is the Internal Counter of shared_ptr?

To manage this shared ownership, the `shared_ptr` pointer uses a concept called the **reference count**.

Consider the following statement:

```cpp
auto p1 = std::make_shared<MyClass>();
```

At this moment, there is one owner of the object.

The following statement:

```cpp
auto p2 = p1;
```

increases the number of owners.

And this statement:

```cpp
auto p3 = p1;
```

adds yet another owner of the same object.

We can depict the situation conceptually like this:

```text
p1 ──┐
     │
p2 ──┼──> MyClass
     │
p3 ──┘

Owners = 3
```

When one of the `shared_ptr` objects is destroyed, the counter decreases.

For example, if `p2` is destroyed:

```text
p1 ──┐
     │
p3 ──┘──> MyClass

Owners = 2
```

When the number of owners reaches zero, the object is deleted.

---

# How Can the Counter Be Observed?

We can observe the number of owners that currently exist using `use_count()`.

A sample statement:

```cpp
auto p1 = std::make_shared<MyClass>();

std::cout << p1.use_count();
```

After `p1` is created, the value is normally `1`.

The following statement:

```cpp
auto p2 = p1;

std::cout << p1.use_count();
```

Now the value will be `2`.

The following statement:

```cpp
auto p3 = p1;

std::cout << p1.use_count();
```

Now the value will be `3`.

The `use_count()` method is useful for observation and debugging, but we must not design the main logic of the program based on its value.

---

# An Important Note About use_count()

The `use_count()` method is not a suitable tool for asking the question "Am I the only owner?".

It is better to design the program based on an ownership contract rather than constantly checking the counter.

Also, in multithreaded environments, the value of `use_count()` may no longer be the same immediately after it is read.

Therefore, it is better to use `use_count()` mostly for observation, diagnostics, and debugging — not for sensitive design decisions.

---

# Creating a shared_ptr

In C++11, we can directly use `new` to instantiate objects on the Heap and have a shared pointer refer to them:

```cpp
std::shared_ptr<MyClass> ptr(new MyClass());
```

The preferred approach is to use `std::make_shared`:

```cpp
auto ptr = std::make_shared<MyClass>();
```

The `make_shared` function is both more readable and can usually perform the memory allocation more efficiently.

The important point is that if you are using C++11, `make_shared` has been part of the standard from the very beginning.

---

# Why Is make_shared Better?

Under normal circumstances, `make_shared` can place the memory for the object and the control block in a single allocation.

Its conceptual representation looks like this:

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

This structure can generally reduce the number of required allocations.

The exact layout details depend on the implementation of the standard library; therefore, we must not rely on a precise, fixed internal layout of `shared_ptr`.

---

# Using shared_ptr in a Setter

If a class accepts shared ownership, the `shared_ptr` can usually be received by value.

A sample statement:

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

The caller can set an existing `shared_ptr`:

```cpp
auto obj = std::make_shared<MyClass>();

owner.setValue(obj);
```

In this case, both `obj` and `value` own the object, and the counter increases.

---

# Why Should a Setter Take shared_ptr by Value?

Receiving a `shared_ptr` by value can express shared ownership very clearly.

The following statement:

```cpp
void setValue(std::shared_ptr<MyClass> ptr)
{
    value = std::move(ptr);
}
```

says that this function receives a `shared_ptr` and, if needed, keeps shared ownership of it.

If the caller has something like this:

```cpp
auto obj = std::make_shared<MyClass>();

owner.setValue(obj);
```

After the call, both `obj` and `owner.value` own the object.

---

# Should We Take shared_ptr by const Reference?

It depends on the function's contract.

If the function only wants to use the `shared_ptr` and is not going to create new ownership, the following can be used:

```cpp
void process(const std::shared_ptr<MyClass>& ptr);
```

If the function is going to store the `shared_ptr` or create shared ownership, receiving it by value is usually a clearer design:

```cpp
void setValue(std::shared_ptr<MyClass> ptr);
```

This difference is important, because the parameter type must communicate the ownership contract to the reader of the code.

---

# Using shared_ptr in a Getter

This part has a very important difference from `unique_ptr`.

Because `shared_ptr` is designed for shared ownership, returning a `shared_ptr` from a getter is completely logical in many APIs.

For example:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

The caller now receives a new `shared_ptr` that shares ownership of the class's internal object.

The usage statement:

```cpp
auto obj = owner.getValue();
```

Now `obj` is also one of the owners of the object.

---

# Why Can a Getter of Type shared_ptr Be Appropriate?

If the purpose of the getter is for the caller to be able to **maintain shared ownership and the lifetime of the object**, returning a `shared_ptr` is a suitable choice.

Consider the following statement:

```cpp
std::shared_ptr<MyClass> obj = owner.getValue();
```

If `owner` is destroyed later, `obj` can still keep the object alive.

This is precisely one of the fundamental differences between a getter for `shared_ptr` and a getter for `unique_ptr`.

---

# Should a Getter Return shared_ptr&?

Usually, it is better to avoid returning a reference to the internal `shared_ptr`, unless we have a specific reason to do so.

The following statement:

```cpp
std::shared_ptr<MyClass>& getValue();
```

exposes the details of the class's internal ownership management to the caller.

The caller could then do something like this:

```cpp
owner.getValue().reset();
```

Now the caller can modify the class's internal ownership.

In many designs, it is better to return the `shared_ptr` itself by value instead:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

This creates a copy of the `shared_ptr` itself, not a copy of the object it owns.

---

# Does Copying a shared_ptr Mean Copying the Object?

This is one of the most important points about `shared_ptr`.

The following statement:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = p1;
```

does not copy `MyClass`.

It only creates another `shared_ptr` that points to the same object.

Its visual representation:

```text
p1 ──┐
     │
p2 ──┼──> MyClass
     │
     └── Control Block
```

Therefore, both `p1` and `p2` point to **the same object**.

---

# How Do We Get a Duplicate of a shared_ptr?

If by "getting a duplicate" we mean creating another `shared_ptr` for the same object, it is enough to copy it.

A sample statement:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = p1;
```

Now `p2` is a new owner of the same object.

The counter statement:

```cpp
std::cout << p1.use_count();
```

It should now normally display the value `2`.

---

# What If We Want a Real Duplicate of the Object Itself?

Copying a `shared_ptr` is completely different from cloning the object.

The following statement:

```cpp
auto p2 = p1;
```

does not create a new object here.

If we truly want an independent object, we must copy the object itself — provided that the type in question is copyable.

A sample statement:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = std::make_shared<MyClass>(*p1);
```

Now `p1` and `p2` have two different objects.

The visual representation:

```text
p1 ──> MyClass #1

p2 ──> MyClass #2
```

These two objects share no ownership with each other.

---

# Accessing the Internal Value of shared_ptr

If by "internal value" we mean the object owned by the `shared_ptr`, the `*` operator can be used.

A sample statement:

```cpp
(*ptr).doSomething();
```

The more common approach is to use the `->` operator:

```cpp
ptr->doSomething();
```

These two are equivalent in terms of accessing the object.

---

# Accessing the Object Through a Raw Pointer

To obtain a non-owning raw pointer, `get()` can be used.

A sample statement:

```cpp
MyClass* raw = ptr.get();
```

The `raw` pointer does not own the object.

The `shared_ptr` still owns the object and remains responsible for freeing it.

The usage statement:

```cpp
if (raw)
{
    raw->doSomething();
}
```

This approach is useful for interacting with APIs that receive raw pointers.

---

# Risks of Using a Raw Pointer from get()

The most important risk is that the lifetime of the raw pointer depends on the main owner.

The dangerous statement:

```cpp
auto ptr = std::make_shared<MyClass>();

MyClass* raw = ptr.get();

ptr.reset();

raw->doSomething(); // dangerous
```

After `reset()`, if `ptr` was the last owner, the object has already been deleted.

The `raw` pointer is now a dangling pointer.

---

# The get() Method Does Not Transfer Ownership

The `get()` method only returns the address of the object.

The following statement:

```cpp
MyClass* raw = ptr.get();
```

does not in any way mean that `raw` has become the owner of the object.

So, after obtaining the raw pointer, you must not delete it as follows; instead, leave the deletion of the object to the smart pointer!:

```cpp
delete raw;
```

This can cause a double deletion, because the `shared_ptr` still owns the object and will eventually try to delete it.

---

# A Very Dangerous Mistake with Raw Pointers

One of the most dangerous mistakes is constructing several independent `shared_ptr` objects from one shared raw pointer.

The following must never be done:

```cpp
MyClass* raw = new MyClass();

std::shared_ptr<MyClass> p1(raw);
std::shared_ptr<MyClass> p2(raw);
```

The `p1` and `p2` objects create two independent control blocks.

The visual representation:

```text
p1 ──> Control Block #1 ──> MyClass

p2 ──> Control Block #2 ──> MyClass
```

Both control blocks believe they own the object.

Eventually, both may try to delete the object, and the result will be undefined behavior.

If we need to create multiple `shared_ptr` objects, we must copy them from an existing `shared_ptr`:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = p1;
```

---

# Can Accessing the Inside of shared_ptr Cause a Memory Leak?

`get()` itself does not cause a memory leak.

The main problem arises when we manage lifetime and ownership incorrectly through a raw pointer.

The dangerous statement:

```cpp
auto ptr = std::make_shared<MyClass>();

MyClass* raw = ptr.get();

ptr.reset();

// raw is now being kept, but the object no longer exists
```

This example leads more to a dangling pointer than to a memory leak.

The more serious memory leak disaster in `shared_ptr` usually arises from an **ownership cycle**.

---

# The Main Problem of shared_ptr: Ownership Cycles

The `shared_ptr` pointer does not delete the object until the reference count reaches zero.

If objects own each other in a cycle, the counter may never reach zero.

A classic example:

```cpp
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

If we write:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();

a->b = b;
b->a = a;
```

The ownership structure now looks like this:

```text
a ──> A ──shared_ptr──> B
       ▲                │
       │                │
       └──shared_ptr────┘
```

`A` owns `B`, and `B` in turn owns `A`.

Even if `a` and `b` go out of scope, this cycle may prevent the reference count of each object from reaching zero.

The result is that the objects are never freed.

---

# Why Doesn't the shared_ptr Counter Work Well in This Case?

The very important point is that **the reference counting mechanism itself is not broken**.

The counter works exactly according to the rules of ownership.

The problem is that our ownership model has created a cycle.

`A` says:

> I still own `B`.

And `B` says:

> I still own `A`.

As a result, neither of them can naturally reach zero.

This is exactly the problem that `std::weak_ptr` was designed to partly solve.

---

# Why Does weak_ptr Exist?

The `weak_ptr` pointer refers to an object managed by a `shared_ptr`, but it **does not own that object**.

A sample statement:

```cpp
std::weak_ptr<MyClass> weak = shared;
```

This does not increase the reference count related to the shared ownership.

Therefore, we can have a relationship that only observes the object, without creating ownership over its lifetime.

In the previous structure, we can change one of the relationships as follows:

```cpp
class B
{
public:
    std::weak_ptr<A> a;
};
```

Now `B` no longer owns `A`.

As a result, the ownership cycle is broken, and the objects can be freed at the appropriate time.

Deeper details of `weak_ptr` and the control block can be covered in a separate tutorial.

---

# What Is the Difference Between shared_ptr and weak_ptr?

The `shared_ptr` pointer is an owner.

The statement:

```cpp
std::shared_ptr<MyClass>
```

means:

> I share in the ownership of the object.

The statement:

```cpp
std::weak_ptr<MyClass>
```

means:

> I only observe the object; I do not own it.

To access the object, a `weak_ptr` usually must first be temporarily converted into a `shared_ptr`.

A sample statement:

```cpp
if (auto ptr = weak.lock())
{
    ptr->doSomething();
}
```

If the object is still alive, `lock()` creates a valid `shared_ptr`.

If the object has already been destroyed, `lock()` returns an empty `shared_ptr`.

---

# shared_ptr in a Class

One of the common uses of `shared_ptr` is for a class to hold shared ownership of an object.

A sample statement:

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

Usage:

```cpp
auto engine = std::make_shared<Engine>();

Car car(engine);

auto engine2 = car.getEngine();
```

Now several `shared_ptr` objects point to the same `Engine`, and all of them share in its ownership.

---

# Is It Better for a Getter to Return shared_ptr?

If the goal is for the caller to be able to maintain the object's lifetime independently of the class, then yes — returning a `shared_ptr` by value is completely logical.

The statement:

```cpp
std::shared_ptr<Engine> getEngine() const
{
    return engine;
}
```

This getter creates a lightweight copy of the smart pointer itself; it does not copy the object.

The result is a new owner, which increases the reference count.

---

# What If We Only Want to Give Access to the Object?

If the caller only needs the object for a short time and must not create new ownership, perhaps we should not return a `shared_ptr` at all.

For a non-modifiable object:

```cpp
const Engine& getEngine() const
{
    return *engine;
}
```

This API gives the caller access, but it does not create new ownership.

If the object being returned can be null, a non-owning const raw pointer is used:

```cpp
const Engine* getEngine() const
{
    return engine.get();
}
```

The choice among these approaches must be made based on the lifetime and ownership contract.

---

# An Important Difference: Getters with unique_ptr

With `unique_ptr`, we usually do not want to hand the class's internal ownership over to the caller so easily, because the ownership must remain unique.

With `shared_ptr`, handing over a copy of the smart pointer is usually not a problem, because its design is fundamentally based on shared ownership.

As a result, this design is completely natural:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

The caller receives a new owner, and the class also remains an owner.

---

# shared_ptr and Transferring with move

Like `unique_ptr`, the `shared_ptr` pointer supports move.

A sample statement:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = std::move(p1);
```

The ownership of `p1` is transferred to `p2`.

After the move, `p1` is empty and `p2` holds the ownership.

---

# The Difference Between copy and move in shared_ptr

The copy command creates a new owner.

The statement:

```cpp
auto p2 = p1;
```

The reference count variable increases.

The move operation transfers the ownership of the `shared_ptr`.

The statement:

```cpp
auto p2 = std::move(p1);
```

In this case, the reference count does not increase; the ownership state of the `shared_ptr` itself is transferred.

Therefore, the choice between copy and move must be based on whether we want to create a new owner or merely transfer the same ownership.

---

# The reset Operation in shared_ptr

The `reset()` method is used to release the current ownership.

The statement:

```cpp
ptr.reset();
```

The `ptr` pointer no longer owns the object.

If `ptr` was the last owner, the object is deleted as well.

A sample statement:

```cpp
auto p1 = std::make_shared<MyClass>();
auto p2 = p1;

p1.reset();
```

After `reset()`, the object created on the Heap is still alive, because `p2` still owns it.

If later:

```cpp
p2.reset();
```

The `p2` pointer also releases the final ownership, and the object will be freed.

---

# The reset Operation with a New Object

The `reset()` method can also attach the `shared_ptr` to a new object.

A sample statement:

```cpp
ptr.reset(new MyClass());
```

If `ptr` was previously the last owner of the old object, the old object is freed and `ptr` becomes the owner of the new object.

For creating a new object, using `make_shared` is still preferred:

```cpp
ptr = std::make_shared<MyClass>();
```

---

# Can a shared_ptr Be Made Empty?

Yes!

The statement:

```cpp
std::shared_ptr<MyClass> ptr;
```

creates an empty `shared_ptr`.

Checking whether the shared_ptr is null:

```cpp
if (!ptr)
{
    // ptr is empty
}
```

`nullptr` can also be used:

```cpp
if (ptr == nullptr)
{
    // Empty
}
```

---

# shared_ptr and Multiple Threads

One of the important points is that we must distinguish between **the thread safety of the control block** and **the thread safety of the object**.

Ownership operations on `shared_ptr` objects are thread-safe under certain specific standard conditions, but this does not mean that the owned object itself is thread-safe.

If two threads do this at the same time:

```cpp
ptr->value++;
```

The `shared_ptr` pointer does not prevent a data race on `value`.

Here, the smart pointer is responsible for managing lifetime and ownership, not for the internal synchronization of the object.

---

# shared_ptr and the Destructor

One of the main advantages of `shared_ptr` is that we normally do not need a manual `delete`.

The statement:

```cpp
class Manager
{
private:
    std::shared_ptr<MyClass> value;
};
```

When `Manager` is destroyed, the class's `shared_ptr` member is destroyed as well.

If that `shared_ptr` is the last owner, the object is freed too.

---

# A Complete Example of Class Design

We can design a class as follows:

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

This class manages shared ownership clearly.

The `setValue()` method receives new shared ownership.

The `getValue()` method returns a new `shared_ptr` for shared ownership.

The `getRawValue()` method only gives non-owning access.

The `resetValue()` method releases the internal ownership.

---

# A Note About Returning shared_ptr

Some programmers worry that returning a `shared_ptr` causes a heavy copy of the object.

This assumption is wrong.

The statement:

```cpp
return value;
```

does not copy the owned object.

Only the smart pointer itself and its related ownership information are managed.

In modern code, the compiler can also optimize many move and copy operations.

---

# When Is Using shared_ptr Not an Appropriate Choice?

We must not use `shared_ptr` everywhere merely because it makes memory management easier.

The `shared_ptr` pointer has more overhead and complexity than `unique_ptr`.

So if we have only one owner:

```cpp
std::unique_ptr<MyClass>
```

is usually the better choice.

If several parts genuinely must own the object:

```cpp
std::shared_ptr<MyClass>
```

can be an appropriate choice.

If we only need access:

```cpp
MyClass&
```

or:

```cpp
MyClass*
```

may be a simpler and more precise choice.

---

# The Difference Between shared_ptr and unique_ptr

The most important difference between these two is the **ownership model**.

| Feature                      | `unique_ptr`       | `shared_ptr`                |
| ---------------------------- | ------------------ | --------------------------- |
| Ownership type               | Unique             | Shared                      |
| Copy                         | No                 | Yes                         |
| Move                         | Yes                | Yes                         |
| Reference Count              | No                 | Yes                         |
| Multiple ownership           | No                 | Yes                         |
| Management overhead          | Lower              | Higher                      |
| Risk of ownership cycles     | No                 | Yes                         |
| Possible need for `weak_ptr` | Usually no         | Yes                         |
| Main use case                | One specific owner | Multiple independent owners |

If we can limit ownership to one specific entity, `unique_ptr` is usually the better choice.

If several entities must be able to maintain the object's lifetime independently of one another, `shared_ptr` is more suitable.

---

# A Simple Comparison

The `unique_ptr` pointer can be thought of as a single key.

Only one person holds the key.

The `shared_ptr` pointer is like a system in which several people each hold a copy of the ownership rights to a resource.

As long as at least one person is still an owner, the resource remains.

The `weak_ptr` pointer is like someone who only knows the address of the resource but does not own it.

---

# When Is shared_ptr the Right Choice?

Using `shared_ptr` is logical when several independent parts of the program must be able to hold the object and maintain its lifetime.

### Appropriate Cases for Using `std::shared_ptr`

The `std::shared_ptr` pointer is appropriate when **the ownership of an object is genuinely shared among several independent parts of the program** and we want the object to remain alive until the last owner is gone.

Some common examples:

* **Sharing an Object Among Multiple Components:** several components need one shared object, and each of them can own it during its own lifetime.
* **In Factories:** when a Factory creates an Object and wants to transfer its ownership to the Caller while also allowing multiple owners.
* **In Graphs and Complex Structures:** in structures where multiple Objects can refer to another Object and its ownership is divided among several parts.
* **Caches:** when objects in a Cache are simultaneously used by other parts of the program and must not be destroyed immediately just because they were removed from the Cache.
* **Resources with Multiple Independent Consumers:** when several independent parts of the program need one Resource and none of them alone is fully responsible for its lifetime.

**Important note:** the mere fact that several parts use one Object does not mean we must use `std::shared_ptr`. First, we must determine **who owns the Object and when it should be destroyed**. If there is one specific owner, `std::unique_ptr` is usually the more appropriate choice, and the other parts can access the Object merely through a non-owning Reference or Pointer.

---

# When Is unique_ptr Better?

If the ownership is specific and unique, it is better to use `unique_ptr` as much as possible.

A sample statement:

```cpp
class Car
{
private:
    std::unique_ptr<Engine> engine;
};
```

The meaning of this design is very clear:

> `Car` owns the `Engine`.

If the `Engine` is not supposed to be owned by any other part, converting it to `shared_ptr` only adds extra complexity.

---

# A Golden Rule for Choosing a Smart Pointer

We can have a simple rule:

Unique ownership:
unique_ptr

Shared ownership:
shared_ptr

Non-owning access, non-nullable:
reference

Non-owning access, nullable:
raw pointer

Non-owning access to an object managed by shared_ptr,
with the need to check whether the object is alive:
weak_ptr

This rule is not absolute, but it is a very good starting point for API design.

---

# Important Notes About shared_ptr

The `shared_ptr` pointer manages ownership with reference counting.

Copying a `shared_ptr` does not copy the object; it only creates shared ownership.

Using the following is the recommended way to create a `shared_ptr` in C++11:

`std::make_shared<T>()`

The `get()` method only returns a non-owning raw pointer.

The `reset()` method releases the `shared_ptr`'s ownership, and if it is the last owner, the object is freed as well.

The `release()` method does not exist in `shared_ptr`; that function belongs to `unique_ptr`.

The `shared_ptr` pointer supports both copy and move.

The `unique_ptr` pointer supports move but not copy.

The `shared_ptr` pointer can create ownership cycles if designed incorrectly.

The `weak_ptr` pointer is very important for creating non-owning relationships and breaking such cycles.

The raw pointer obtained from `get()` must not be freed with `delete`.

We must not create several independent `shared_ptr` objects from a single raw pointer.

---

# Final Summary

The core concept of `std::shared_ptr` is:

> **Multiple owners can jointly own one object, and the object remains alive as long as the last owner still exists.**

This behavior is implemented using reference counting.

The statement for creating a `shared_ptr`:

```cpp
auto ptr = std::make_shared<MyClass>();
```

Creating a new owner:

```cpp
auto other = ptr;
```

Transferring the state of a `shared_ptr`:

```cpp
auto other = std::move(ptr);
```

Accessing the object:

```cpp
ptr->doSomething();
```

Obtaining a non-owning raw pointer:

```cpp
MyClass* raw = ptr.get();
```

Releasing ownership:

```cpp
ptr.reset();
```

Observing the number of owners:

```cpp
ptr.use_count();
```

Obtaining shared ownership from a getter:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

Obtaining only non-owning access:

```cpp
MyClass* getValue()
{
    return value.get();
}
```

The very important point is that `shared_ptr` was created to solve the problem of **shared ownership**, not merely to eliminate the need to write `delete`.

If only one party holds the ownership, `unique_ptr` is usually the better and simpler choice.

If several independent parts of the program must be able to maintain the lifetime of an object, `shared_ptr` can be an appropriate choice.

If we only want to access the object and do not own it, it is better not to transfer ownership with `shared_ptr` at all.

And finally, if the `shared_ptr` ownership relationships turn into a cycle, reference counting never reaches zero — and this is where `weak_ptr` enters the design to create a **non-owning** relationship.

Therefore, the most important question when using `shared_ptr` is not "Can I use it here?" but rather:

> **Do there really exist multiple independent owners for this object?**

If the answer to this question is yes, `shared_ptr` can be a suitable tool for expressing this ownership model; otherwise, `unique_ptr` or even a non-owning reference/raw pointer often provides a simpler and more precise design.

---
## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>