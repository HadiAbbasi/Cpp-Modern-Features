<div align="right">

[🇺🇸 English](./weak_ptr.md) | [🇮🇷 فارسی](../../fa/cpp11/weak_ptr.md)

</div>

---
# `std::weak_ptr` in C++11 — A Complete and Simple Guide to Ownership, `shared_ptr` Cycles, and Preventing Memory Leaks

## Table of Contents

1. Introduction
2. What problem existed before `weak_ptr`?
3. How does `shared_ptr` manage ownership?
4. The main problem with `shared_ptr`: Ownership Cycles
5. Why can't the Reference Counter reach zero?
6. What exactly is `weak_ptr`?
7. The fundamental difference between `shared_ptr` and `weak_ptr`
8. Does `weak_ptr` own the object?
9. How does `weak_ptr` break ownership cycles?
10. Creating a `weak_ptr`
11. Putting a `weak_ptr` into a Setter
12. Getting a `weak_ptr` from a Getter
13. Should a Getter return `weak_ptr`?
14. How to make a copy of a `weak_ptr`
15. Copy and Move with `weak_ptr`
16. `reset()` with `weak_ptr`
17. Checking whether a `weak_ptr` is empty
18. What is `expired()`?
19. Converting `weak_ptr` to `shared_ptr` with `lock()`
20. Why is `lock()` important?
21. Accessing the underlying object after `lock()`
22. Accessing object members through `weak_ptr`
23. Does `weak_ptr` have `operator->`?
24. Does `weak_ptr` provide a Raw Pointer?
25. The dangers of using a Raw Pointer
26. How can `weak_ptr` prevent a Memory Leak?
27. Using `weak_ptr` inside a class
28. The common Parent/Child pattern
29. The Observer pattern
30. A complete example of a broken cycle with `shared_ptr`
31. Fixing the example with `weak_ptr`
32. `use_count()` in `weak_ptr`
33. Strong Count vs. Weak Count
34. What is the Control Block?
35. Why doesn't `weak_ptr` keep the object alive?
36. `weak_ptr` vs. `shared_ptr`
37. `weak_ptr` vs. `unique_ptr`
38. When should we use `weak_ptr`?
39. When should we not use `weak_ptr`?
40. Common mistakes
41. Final summary

---

# 1. Introduction

`std::weak_ptr` is one of the most important parts of C++11's smart-pointer system.

The concept of `weak_ptr` makes sense together with `shared_ptr`, and it was introduced to solve an important problem that can occur with the shared-ownership model of `shared_ptr`.

The main idea can be expressed in one sentence:

> **A `weak_ptr` can observe an object without owning that object.**

This statement may look simple at first, but this exact property solves one of the most important problems associated with `shared_ptr`.

That problem is called an **Ownership Cycle**.

If we understand ownership cycles correctly, we have already understood the primary reason why `weak_ptr` exists.

---

# 2. What problem existed before `weak_ptr`?

To understand `weak_ptr`, it is useful to first understand why `shared_ptr` exists in the first place.

The purpose of `shared_ptr` is to handle situations where multiple parts of a program need to share ownership of the same object.

For example:

```cpp
auto person = std::make_shared<Person>();

auto a = person;
auto b = person;
```

Now we have three `shared_ptr` objects:

```text
person ──┐
         │
a ───────┼──> Person
         │
b ───────┘
```

The important point is that all three `shared_ptr` objects own the same `Person`.

The Reference Count keeps track of the number of active owners.

For example:

```text
person + a + b = 3 owners
```

When one of them is destroyed:

```text
person + a = 2 owners
```

When another one is destroyed:

```text
person = 1 owner
```

Eventually, when the last owner disappears:

```text
0 owners
```

the object can be destroyed.

This behavior is very useful.

However, there is one important problem.

---

# 3. How does `shared_ptr` manage ownership?

`shared_ptr` typically uses an internal structure called a **Control Block**.

A simplified representation looks like this:

```text
              Control Block
          ┌───────────────────┐
          │ Strong Count      │
          │ Weak Count        │
          │ Deleter           │
          │ Other information │
          └─────────┬─────────┘
                    │
                    ▼
                 Object
```

The Strong Count represents the number of actual owners, meaning the number of `shared_ptr` instances that own the object.

The important concept is that as long as the Strong Count has not reached zero, the object normally remains alive.

For example:

```cpp
auto p1 = std::make_shared<int>(42);
auto p2 = p1;
auto p3 = p2;
```

Now:

```text
Strong Count = 3
```

When:

```cpp
p1.reset();
```

is executed:

```text
Strong Count = 2
```

Then:

```cpp
p2.reset();
```

results in:

```text
Strong Count = 1
```

Finally:

```cpp
p3.reset();
```

removes the last owner.

At that point, the object can be destroyed.

---

# 4. The main problem with `shared_ptr`: Ownership Cycles

The important point is that Reference Counting works very well as long as ownership relationships do not form a cycle.

To understand the problem, imagine that we have two classes:

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

The meaning of these classes is that `A` owns `B`, while `B` also owns `A`.

Now:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();

a->b = b;
b->a = a;
```

The ownership relationship now looks like this:

```text
        ┌──────────────┐
        │              │
        ▼              │
      Object A      Object B
        │              │
        │              │
        └──────────────┘
```

More precisely:

```text
A owns B
B owns A
```

This is an **Ownership Cycle**.

---

# 5. Why can't the Reference Counter reach zero?

This is one of the most important concepts in the entire subject.

Suppose the local variables `a` and `b` go out of scope.

We might expect:

```text
a destroyed
b destroyed

therefore:

A destroyed
B destroyed
```

But that does not necessarily happen.

The reason is that `A` still contains a `shared_ptr` to `B`.

And `B` still contains a `shared_ptr` to `A`.

Therefore:

```text
A has 1 owner
B has 1 owner
```

even though there are no longer any external owners.

The key concept is:

> Each object is keeping the other object alive.

Therefore:

```text
A → B → A → B → A → ...
```

The cycle never gets broken.

It is important to understand that **Reference Counting itself has not failed**.

The Reference Counting mechanism is doing exactly what it was designed to do.

Each `shared_ptr` is effectively saying:

> I am still an owner of this object.

Since `A` owns `B`, and `B` owns `A`, neither count can reach zero.

This is exactly the problem that `weak_ptr` was designed to solve.

---

# 6. What exactly is `weak_ptr`?

A `weak_ptr` is a smart pointer that can point to an object managed by `shared_ptr` **without owning that object**.

The key phrase is:

> **Observe, but do not own.**

For example:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;
```

The relationship is:

```text
shared ──owns──> int

weak ──observes──> int
```

The important point is that creating `weak` does not increase the Strong Count.

If:

```cpp
shared.use_count()
```

was `1` before creating `weak`, the Strong Count is still `1` afterward.

---

# 7. The fundamental difference between `shared_ptr` and `weak_ptr`

A simple way to understand the difference is:

```text
shared_ptr:
"I own the object."

weak_ptr:
"I know about the object, but I do not own it."
```

A `shared_ptr` helps keep the object alive.

A `weak_ptr` does not keep the object alive.

For example:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

shared.reset();
```

If `shared` was the last owner, the `int` object is destroyed.

However, `weak` may still exist.

It simply can no longer access a live object.

---

# 8. Does `weak_ptr` own the object?

No.

This is the most important characteristic of `weak_ptr`.

The following:

```cpp
std::weak_ptr<MyClass> weak;
```

does not mean that `weak` owns a `MyClass`.

It simply means that `weak` has a non-owning relationship with an object.

Therefore, `weak_ptr` does not increase the Strong Reference Count.

---

# 9. How does `weak_ptr` break ownership cycles?

The key idea is that in an ownership cycle, at least one relationship should be non-owning.

For example, instead of:

```cpp
class B
{
public:
    std::shared_ptr<A> a;
};
```

we can write:

```cpp
class B
{
public:
    std::weak_ptr<A> a;
};
```

Now:

```text
A ──shared_ptr──> B
B ──weak_ptr────> A
```

The important point is that `A` owns `B`.

But `B` only observes `A`.

Therefore:

```text
A owns B
B observes A
```

There is no longer an ownership cycle.

---

# 10. Creating a `weak_ptr`

A `weak_ptr` is normally created from a `shared_ptr`:

```cpp
auto shared = std::make_shared<MyClass>();

std::weak_ptr<MyClass> weak = shared;
```

The important point is that `weak` connects to the same Control Block.

A `weak_ptr` is not designed to independently manage an object created with `new`.

For example, this is generally not a meaningful design:

```cpp
std::weak_ptr<MyClass> weak(new MyClass());
```

The reason is that `weak_ptr` is specifically designed to observe objects whose ownership is managed by `shared_ptr`.

---

# 11. Putting a `weak_ptr` into a Setter

If a class needs to maintain a non-owning relationship with an object, its Setter can accept a `weak_ptr`.

For example:

```cpp
class Observer
{
private:
    std::weak_ptr<MyClass> value;

public:
    void setValue(std::weak_ptr<MyClass> ptr)
    {
        value = std::move(ptr);
    }
};
```

Then:

```cpp
auto object = std::make_shared<MyClass>();

Observer observer;

observer.setValue(object);
```

The important concept is that `observer` does not become an owner of `object`.

If:

```cpp
object.reset();
```

is executed and `object` was the last owner, the object will be destroyed.

---

# 12. Getting a `weak_ptr` from a Getter

If a class stores a non-owning relationship, we can return it through a Getter:

```cpp
std::weak_ptr<MyClass> getValue() const
{
    return value;
}
```

The important point is that returning a `weak_ptr` does not turn it into ownership.

The caller receives a copy of the `weak_ptr`.

---

# 13. Should a Getter return `weak_ptr`?

In many designs:

```cpp
std::weak_ptr<MyClass> getValue() const
{
    return value;
}
```

is a good choice.

The meaning of this API is essentially:

> This object may still exist, but I do not own it, and simply receiving this value should not extend its lifetime.

This is an important concept in class design.

If the Getter instead returns:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value.lock();
}
```

the caller receives a new owner.

Therefore, these two Getter designs have completely different semantics.

---

# 14. How to make a copy of a `weak_ptr`

A `weak_ptr`, like many normal C++ objects, can be copied.

For example:

```cpp
auto shared = std::make_shared<MyClass>();

std::weak_ptr<MyClass> w1 = shared;
std::weak_ptr<MyClass> w2 = w1;
```

Now:

```text
w1 ──┐
     ├──> same Control Block
w2 ──┘
```

The important point is that copying the `weak_ptr` does not copy the object.

It does not create another owner either.

It simply creates another observer.

---

# 15. Copy and Move with `weak_ptr`

A `weak_ptr` supports both Copy and Move operations.

Copy:

```cpp
auto w2 = w1;
```

creates another observer.

Move:

```cpp
auto w2 = std::move(w1);
```

transfers the `weak_ptr` state to `w2`.

After the Move, `w1` will normally be empty.

---

# 16. `reset()` with `weak_ptr`

The `reset()` function disconnects the `weak_ptr` from its Control Block.

For example:

```cpp
std::weak_ptr<MyClass> weak = shared;

weak.reset();
```

Now `weak` no longer observes the object.

The important point is that calling `reset()` on a `weak_ptr` does not destroy the object.

A `weak_ptr` does not own the object in the first place.

If:

```cpp
shared.use_count() == 1
```

then:

```cpp
weak.reset();
```

has no effect on the object's lifetime.

The object is still managed by `shared`.

---

# 17. Checking Whether a `weak_ptr` Is Empty

To check whether a `weak_ptr` currently refers to a Control Block and whether its observed object is still alive, we commonly use:

```cpp
weak.expired()
```

For example:

```cpp
std::weak_ptr<MyClass> weak;

if (weak.expired())
{
    // No live object is available through weak
}
```

However, there is an important detail.

Using `expired()` alone is generally not the best way to make the final decision about accessing the object.

---

# 18. What is `expired()`?

`expired()` tells us whether the object observed by the `weak_ptr` is no longer alive.

For example:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

std::cout << weak.expired();
```

As long as `shared` keeps the object alive, the result is `false`.

Then:

```cpp
shared.reset();
```

If `shared` was the last owner:

```cpp
weak.expired()
```

will return `true`.

The important point is that `expired()` represents the state at a particular moment.

---

# 19. Converting `weak_ptr` to `shared_ptr` with `lock()`

This is one of the most important parts of `weak_ptr`.

A `weak_ptr` does not allow direct access like:

```cpp
weak->someFunction();
```

Instead, we first convert it to a `shared_ptr` using `lock()`:

```cpp
auto shared = weak.lock();
```

If the object is still alive, `shared` will contain a valid `shared_ptr`.

If the object has already been destroyed, `shared` will be empty.

A very common pattern is:

```cpp
if (auto shared = weak.lock())
{
    shared->doSomething();
}
```

This is one of the most important usage patterns for `weak_ptr`.

---

# 20. Why is `lock()` important?

The important concept behind `lock()` is that it does not merely check whether the object is alive.

If the object is alive, it creates a `shared_ptr` that temporarily maintains ownership of the object.

This is very important.

Suppose we write:

```cpp
if (!weak.expired())
{
    // ...
}
```

and then access the object later.

In multithreaded environments, or even in more complex designs, the object could potentially be destroyed between the check and the actual use.

Therefore, this pattern is generally better:

```cpp
if (auto shared = weak.lock())
{
    shared->doSomething();
}
```

The meaning is:

> If the object is still alive, create a temporary owner and then use the object.

This is much safer.

---

# 21. Accessing the Underlying Object After `lock()`

The `lock()` function returns a `shared_ptr`.

Therefore, we can use all the usual capabilities of `shared_ptr`:

```cpp
if (auto shared = weak.lock())
{
    shared->value = 100;
}
```

Or:

```cpp
if (auto shared = weak.lock())
{
    shared->doSomething();
}
```

Conceptually:

```text
weak_ptr
   │
   │ lock()
   ▼
shared_ptr
   │
   ▼
Object
```

---

# 22. Accessing Object Members Through `weak_ptr`

The important point is that `weak_ptr` itself does not provide `operator->` for direct access to the object.

Therefore, this is not valid:

```cpp
weak->doSomething();
```

Instead:

```cpp
if (auto ptr = weak.lock())
{
    ptr->doSomething();
}
```

And for a data member:

```cpp
if (auto ptr = weak.lock())
{
    std::cout << ptr->value;
}
```

---

# 23. Does `weak_ptr` Have `operator->`?

No.

This is intentional.

The reason is that the object may no longer exist.

If `weak_ptr` allowed:

```cpp
weak->doSomething();
```

a programmer might accidentally use an object without considering its lifetime.

The `lock()` mechanism forces the programmer to explicitly obtain a `shared_ptr` and therefore acknowledge the object's lifetime before accessing it.

---

# 24. Does `weak_ptr` Provide a Raw Pointer?

`weak_ptr` does not directly provide a function such as:

```cpp
get()
```

that returns a Raw Pointer to the object.

This is also intentional.

To access the object, we first do:

```cpp
auto shared = weak.lock();
```

Then:

```cpp
MyClass* raw = shared.get();
```

is available.

---

# 25. The Dangers of Using a Raw Pointer

The dangerous part is that once you have a Raw Pointer, you are no longer using the smart pointer itself to express the lifetime relationship.

For example:

```cpp
auto shared = weak.lock();

if (shared)
{
    MyClass* raw = shared.get();

    raw->doSomething();
}
```

There is no problem here because `shared` remains alive until the end of the scope.

However, this is dangerous:

```cpp
MyClass* raw = weak.lock().get();
```

The reason is that `lock()` creates a temporary `shared_ptr`.

That temporary is destroyed at the end of the full expression.

Therefore, the Raw Pointer can immediately become dangling.

So this is dangerous:

```cpp
MyClass* raw = weak.lock().get();
```

A safer version is:

```cpp
if (auto shared = weak.lock())
{
    MyClass* raw = shared.get();

    raw->doSomething();
}
```

However, in many cases we do not need a Raw Pointer at all.

It is usually better to simply write:

```cpp
shared->doSomething();
```

---

# 26. How Can `weak_ptr` Prevent a Memory Leak?

The important point is that `weak_ptr` does not generally solve every kind of Memory Leak.

Its main purpose here is to prevent **ownership cycles** created by `shared_ptr`.

Suppose we have:

```text
A ──shared_ptr──> B
B ──shared_ptr──> A
```

An ownership cycle is created.

If we change one relationship to:

```text
B ──weak_ptr────> A
```

we get:

```text
A ──shared_ptr──> B
B ──weak_ptr────> A
```

The ownership cycle has been removed.

Therefore, when the real owner of `A` disappears, `A` can be destroyed.

This is exactly how `weak_ptr` can prevent a Memory Leak caused by a `shared_ptr` ownership cycle.

---

# 27. Using `weak_ptr` Inside a Class

A common example is a Child storing a reference to its Parent.

For example:

```cpp
class Parent;

class Child
{
private:
    std::weak_ptr<Parent> parent;

public:
    void setParent(std::shared_ptr<Parent> p)
    {
        parent = p;
    }

    std::shared_ptr<Parent> getParent() const
    {
        return parent.lock();
    }
};
```

The meaning of this class is that `Child` knows who its Parent is.

But Child does not own Parent.

This prevents the creation of an ownership cycle.

---

# 28. The Common Parent/Child Pattern

Parent/Child is one of the best examples for understanding `weak_ptr`.

Suppose:

```text
Parent
   │
   │ owns
   ▼
Child
```

If Child also stores the Parent using `shared_ptr`:

```text
Parent
   │
   │ shared_ptr
   ▼
Child
   │
   │ shared_ptr
   ▼
Parent
```

we have a cycle.

The typical design is:

```text
Parent
   │
   │ shared_ptr
   ▼
Child
   │
   │ weak_ptr
   ▼
Parent
```

In this design, Parent owns Child.

Child only observes Parent.

---

# 29. The Observer Pattern

Another important use case for `weak_ptr` is the **Observer Pattern**.

Suppose we have a main object:

```cpp
auto subject = std::make_shared<Subject>();
```

and several observers that need to know whether the Subject still exists.

If the observers store `shared_ptr`, they may unintentionally keep the Subject alive.

Instead:

```cpp
std::weak_ptr<Subject> subject;
```

is often more appropriate.

The idea of an Observer is:

> I want to access the object if it still exists, but I must not keep it alive.

This is exactly the type of relationship that `weak_ptr` is designed to represent.

---

# 30. A Complete Example of a Broken Cycle with `shared_ptr`

The following is one of the most important examples for understanding `weak_ptr`:

```cpp
#include <iostream>
#include <memory>

class B;

class A
{
public:
    std::shared_ptr<B> b;

    ~A()
    {
        std::cout << "A destroyed\n";
    }
};

class B
{
public:
    std::shared_ptr<A> a;

    ~B()
    {
        std::cout << "B destroyed\n";
    }
};
```

Then:

```cpp
{
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();

    a->b = b;
    b->a = a;
}
```

The apparent expectation is that both objects should be destroyed when the scope ends.

But that does not happen.

The reason is:

```text
A owns B
B owns A
```

Therefore, each object keeps the other alive.

---

# 31. Fixing the Example with `weak_ptr`

To fix the problem, we make one relationship non-owning:

```cpp
class B
{
public:
    std::weak_ptr<A> a;

    ~B()
    {
        std::cout << "B destroyed\n";
    }
};
```

Now:

```text
A ──shared_ptr──> B
B ──weak_ptr────> A
```

The Ownership Cycle has been removed.

When the scope ends:

```text
a destroyed
b destroyed
```

the Strong Counts can reach zero.

The objects can therefore be destroyed.

---

# 32. `use_count()` in `weak_ptr`

A `weak_ptr` also provides:

```cpp
use_count()
```

For example:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

std::cout << weak.use_count();
```

The returned value represents the number of **Strong Owners**.

In other words, it tells us how many `shared_ptr` objects currently own the object.

The important point is that the `weak_ptr` itself does not increase this number.

---

# 33. Strong Count vs. Weak Count

A Control Block generally involves two important counting concepts:

```text
Strong Count
Weak Count
```

The Strong Count represents the number of actual owners, meaning `shared_ptr` instances.

The Weak Count is associated with the existence of `weak_ptr` observers and with keeping the Control Block itself alive.

These two counts should not be confused.

The most important rule regarding the object itself is:

```text
Strong Count == 0
```

At that point, the managed object can be destroyed.

However, the Control Block may remain alive because `weak_ptr` instances may still exist.

---

# 34. What Is the Control Block?

The Control Block is an internal structure used by `shared_ptr` and `weak_ptr` to manage ownership and lifetime information.

A simplified representation is:

```text
             Control Block
       ┌───────────────────────┐
       │ Strong Count          │
       │ Weak Count            │
       │ Deleter               │
       │ Other metadata        │
       └───────────┬───────────┘
                   │
                   ▼
                 Object
```

The important point is that `weak_ptr` does not directly own the object.

Instead, it is associated with the Control Block that manages the object's ownership.

This is how it can determine whether the object is still alive.

---

# 35. Why Doesn't `weak_ptr` Keep the Object Alive?

This is a very important question.

Suppose:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;
```

Now:

```text
Strong Count = 1
```

and a `weak_ptr` also exists.

If:

```cpp
shared.reset();
```

is executed:

```text
Strong Count = 0
```

The object is destroyed.

However, the Control Block may remain alive so that `weak` can determine the object's state.

Therefore:

```text
Object lifetime
```

and:

```text
Control Block lifetime
```

are two different concepts.

This is an important detail when understanding the internal behavior of `weak_ptr`.

---

# 36. `weak_ptr` vs. `shared_ptr`

The following table summarizes the main differences:

| Feature                                 | `shared_ptr`              | `weak_ptr` |
| --------------------------------------- | ------------------------- | ---------- |
| Owns the object?                        | Yes                       | No         |
| Increases Strong Count?                 | Yes                       | No         |
| Keeps object alive?                     | Yes                       | No         |
| Supports Copy?                          | Yes                       | Yes        |
| Supports Move?                          | Yes                       | Yes        |
| Has `reset()`?                          | Yes                       | Yes        |
| Has direct `get()`?                     | Yes                       | No         |
| Has `operator->`?                       | Yes                       | No         |
| Has `lock()`?                           | No                        | Yes        |
| Has `expired()`?                        | No                        | Yes        |
| Suitable for breaking ownership cycles? | No, potentially dangerous | Yes        |
| Creates ownership?                      | Yes                       | No         |

---

# 37. `weak_ptr` vs. `unique_ptr`

`unique_ptr` and `weak_ptr` are fundamentally different in terms of ownership semantics.

The meaning of `unique_ptr` is:

> I am the sole owner of this object.

The meaning of `weak_ptr` is:

> I do not own this object at all; I only want to observe it.

Therefore:

```cpp
std::unique_ptr<MyClass>
```

represents **Unique Ownership**.

Whereas:

```cpp
std::weak_ptr<MyClass>
```

represents **Non-owning Observation** of an object managed by `shared_ptr`.

---

# 38. When Should We Use `weak_ptr`?

First, when we want to observe an object without extending its lifetime, `weak_ptr` is often appropriate.

Second, when relationships between objects can create ownership cycles, one of the ownership relationships can often be modeled using `weak_ptr`.

Third, `weak_ptr` is commonly useful in Observer Patterns, Parent/Child relationships, and certain caching designs.

The general principle is:

> **A relationship does not necessarily mean ownership.**

This is one of the best mental models for understanding `weak_ptr`.

---

# 39. When Should We Not Use `weak_ptr`?

The important point is that we should not use `weak_ptr` everywhere simply because it is a powerful tool.

If a class genuinely needs to own an object, `weak_ptr` is not the correct choice.

For example:

```cpp
class Car
{
    std::weak_ptr<Engine> engine;
};
```

If a `Car` cannot function without its `Engine` and must keep the `Engine` alive, then `shared_ptr` or even `unique_ptr` may be a better choice.

The concept of `weak_ptr` only makes sense when non-ownership is actually part of the design.

---

# 40. Common Mistakes

One common mistake is assuming that `weak_ptr` keeps an object alive.

Another mistake is trying to dereference a `weak_ptr` directly:

```cpp
weak->foo();
```

This is not possible.

The correct approach is:

```cpp
if (auto ptr = weak.lock())
{
    ptr->foo();
}
```

Another common mistake is unnecessarily writing:

```cpp
if (!weak.expired())
{
    auto ptr = weak.lock();
}
```

In most cases, this check is redundant.

A better pattern is:

```cpp
if (auto ptr = weak.lock())
{
    ptr->foo();
}
```

Another dangerous mistake is obtaining a Raw Pointer from a temporary:

```cpp
auto raw = weak.lock().get();
```

This Raw Pointer can immediately become dangling.

A safer version is:

```cpp
if (auto ptr = weak.lock())
{
    auto raw = ptr.get();

    raw->foo();
}
```

However, in many cases there is no need for a Raw Pointer at all.

It is usually better to simply use:

```cpp
ptr->foo();
```

---

# 41. A More Realistic Complete Example

The following example demonstrates a common Parent/Child design:

```cpp
#include <iostream>
#include <memory>

class Parent;

class Child
{
private:
    std::weak_ptr<Parent> parent;

public:
    void setParent(const std::shared_ptr<Parent>& p)
    {
        parent = p;
    }

    void printParent() const
    {
        if (auto p = parent.lock())
        {
            std::cout << "Parent is alive\n";

            p->hello();
        }
        else
        {
            std::cout << "Parent no longer exists\n";
        }
    }
};

class Parent
{
private:
    std::shared_ptr<Child> child;

public:
    void setChild(std::shared_ptr<Child> c)
    {
        child = std::move(c);
    }

    void hello() const
    {
        std::cout << "Hello from Parent\n";
    }
};
```

Then:

```cpp
auto parent = std::make_shared<Parent>();
auto child = std::make_shared<Child>();

parent->setChild(child);
child->setParent(parent);
```

The ownership relationship is:

```text
Parent ──shared_ptr──> Child
   ▲
   │
   │ weak_ptr
   │
 Child
```

The important point is that Child does not keep Parent alive.

---

# 42. An Important Point About Getters

A Getter involving `weak_ptr` should be designed carefully.

If we want the caller to merely observe the object:

```cpp
std::weak_ptr<MyClass> getObject() const
{
    return object;
}
```

is appropriate.

The caller can then decide whether to temporarily acquire ownership:

```cpp
if (auto object = getObject().lock())
{
    object->doSomething();
}
```

However, if we want the caller to be able to keep the object alive for some period of time, we can instead return:

```cpp
std::shared_ptr<MyClass> getObject() const
{
    return object.lock();
}
```

In this case, the Getter creates and returns a `shared_ptr`.

Therefore, the object remains alive as long as that returned `shared_ptr` exists.

---

# 43. Why Don't We Directly Convert `weak_ptr` to `shared_ptr`?

The following is not allowed:

```cpp
std::shared_ptr<MyClass> ptr = weak;
```

The reason is that the object may no longer exist.

Therefore, we must explicitly write:

```cpp
auto ptr = weak.lock();
```

Now we have two possible states.

First:

```cpp
if (ptr)
{
    // Object is alive
}
```

Second:

```cpp
if (!ptr)
{
    // Object has already been destroyed
}
```

This design makes the object's lifetime state explicit in the code.

---

# 44. How Can We Understand `lock()` in Very Simple Terms?

Imagine the following:

```text
weak_ptr:

"I have a reference to this person,
but I am not responsible for keeping them alive."

lock():

"If they are still alive,
give me a shared_ptr temporarily."
```

If the person no longer exists:

```text
lock() → empty shared_ptr
```

If the person is still alive:

```text
lock() → valid shared_ptr
```

This is one of the simplest mental models for understanding `weak_ptr`.

---

# 45. Can `weak_ptr` Itself Cause a Memory Leak?

Normally, no.

A `weak_ptr` does not keep the object alive.

It may cause the **Control Block** to remain alive longer, but that is not the same as leaking the managed object.

The important distinction is:

```text
Strong Count → Object lifetime
Weak Count   → Control Block lifetime
```

Therefore, having a `weak_ptr` remain after the object has been destroyed is completely normal.

When the last `weak_ptr` also disappears, the Control Block can eventually be released.

---

# 46. A Subtle Point About `use_count()`

The following:

```cpp
weak.use_count()
```

returns the number of Strong Owners.

For example:

```cpp
auto p = std::make_shared<int>(10);

std::weak_ptr<int> w = p;

std::cout << w.use_count();
```

will normally print:

```text
1
```

The important point is that:

```cpp
w.use_count()
```

does **not** return the number of `weak_ptr` objects.

To determine whether the object is still alive, it is generally better to use:

```cpp
w.expired()
```

or, preferably:

```cpp
if (auto p = w.lock())
```

---

# 47. `weak_ptr` in One Sentence

If we wanted to summarize the entire article in one sentence:

> **`std::weak_ptr` is a non-owning smart pointer that allows us to observe an object managed by `shared_ptr` without increasing the Strong Reference Count and therefore without extending the object's lifetime.**

The more important concept is that this property allows us to break ownership cycles.

---

# 48. Final Summary

The purpose of `shared_ptr` is **Shared Ownership**.

The purpose of `weak_ptr` is to establish a relationship with an object without owning it.

The main problem occurs when multiple `shared_ptr` objects keep each other alive in a cycle.

For example:

```text
A owns B
B owns A
```

creates an Ownership Cycle.

The result is that the Reference Count can never reach zero.

A `weak_ptr` can make one of these relationships non-owning:

```text
A owns B
B observes A
```

This breaks the cycle.

Creating a `weak_ptr`:

```cpp
auto shared = std::make_shared<MyClass>();

std::weak_ptr<MyClass> weak = shared;
```

Copying it:

```cpp
auto weak2 = weak;
```

Moving it:

```cpp
auto weak2 = std::move(weak);
```

Resetting it:

```cpp
weak.reset();
```

Checking whether the object has expired:

```cpp
weak.expired();
```

Safely converting it to a `shared_ptr`:

```cpp
auto shared = weak.lock();
```

Accessing the object:

```cpp
if (auto shared = weak.lock())
{
    shared->doSomething();
}
```

Accessing a Raw Pointer:

```cpp
if (auto shared = weak.lock())
{
    MyClass* raw = shared.get();
}
```

The important point is that `weak_ptr` does not directly provide `get()`, and we must first call `lock()` to obtain a valid `shared_ptr`.

Another important point is that `weak_ptr` does not own the object and therefore cannot keep it alive.

---

# Final Conclusion

The concept of smart pointers can be remembered through four simple questions:

```text
Who owns the object?
Who shares ownership?
Who only observes it?
Can ownership form a cycle?
```

The answers are generally:

```text
One owner
    → unique_ptr

Multiple owners
    → shared_ptr

Non-owning observer of shared ownership
    → weak_ptr
```

The final idea is that `weak_ptr` is not simply a "weaker" version of `shared_ptr`.

The word **weak** does not mean that it is less capable.

It refers to **Weak Ownership**—or, more precisely, the absence of Strong Ownership.

`weak_ptr` intentionally does not keep the object alive.

And that is exactly why it can break ownership cycles created by `shared_ptr`.

If a programmer remembers only one sentence from this article, it should be this:

> **`shared_ptr` says, "As long as I exist, the object should remain alive." `weak_ptr` says, "If the object is still alive, let me see and use it, but my existence should never prevent the object from being destroyed."**

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>