
<div align="center">

[🇺🇸 English](./weak_ptr.md) | [🇮🇷 فارسی](../../fa/cpp11/weak_ptr.md)

</div>

---

# Using `std::weak_ptr` in C++11 — A Complete and Simple Guide to Understanding Ownership, `shared_ptr` Cycles, and Preventing Memory Leaks

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. What Was the Problem Before `weak_ptr`?](#2-what-was-the-problem-before-weak_ptr)
- [3. How Does `shared_ptr` Manage Ownership?](#3-how-does-shared_ptr-manage-ownership)
- [4. The Main Problem with `shared_ptr`: Ownership Cycles and Memory Leaks](#4-the-main-problem-with-shared_ptr-ownership-cycles-and-memory-leaks)
- [5. What Exactly Is `weak_ptr`?](#5-what-exactly-is-weak_ptr)
- [6. Fundamental Difference Between `shared_ptr` and `weak_ptr`](#6-fundamental-difference-between-shared_ptr-and-weak_ptr)
- [7. How Does `weak_ptr` Break Ownership Cycles?](#7-how-does-weak_ptr-break-ownership-cycles)
- [8. Creating and Passing `weak_ptr` in a Setter](#8-creating-and-passing-weak_ptr-in-a-setter)
- [9. Returning `weak_ptr` from a Getter](#9-returning-weak_ptr-from-a-getter)
- [10. Copy, Move, and `reset()` in `weak_ptr`](#10-copy-move-and-reset-in-weak_ptr)
- [11. Checking Emptiness and `expired()`](#11-checking-emptiness-and-expired)
- [12. Converting `weak_ptr` to `shared_ptr` Using `lock()`](#12-converting-weak_ptr-to-shared_ptr-using-lock)
- [13. Accessing Object Members via `weak_ptr`](#13-accessing-object-members-via-weak_ptr)
- [14. `weak_ptr` and Raw Pointers](#14-weak_ptr-and-raw-pointers)
- [15. Using `weak_ptr` in Classes and the Parent/Child Pattern](#15-using-weak_ptr-in-classes-and-the-parentchild-pattern)
- [16. The Observer Pattern](#16-the-observer-pattern)
- [17. Full Example of a Broken Cycle and Fixing It with `weak_ptr`](#17-full-example-of-a-broken-cycle-and-fixing-it-with-weak_ptr)
- [18. `use_count()`, Strong Count, Weak Count, and Control Block](#18-use_count-strong-count-weak-count-and-control-block)
- [19. Differences Between `weak_ptr` and `shared_ptr`](#19-differences-between-weak_ptr-and-shared_ptr)
- [20. Differences Between `weak_ptr` and `unique_ptr`](#20-differences-between-weak_ptr-and-unique_ptr)
- [21. When Should or Shouldn't We Use `weak_ptr`?](#21-when-should-or-shouldnt-we-use-weak_ptr)
- [22. Common Mistakes](#22-common-mistakes)
- [23. A Complete and More Realistic Example](#23-a-complete-and-more-realistic-example)
- [24. Final Summary](#24-final-summary)

---

# 1. Introduction

The `std::weak_ptr` tool is one of the most important parts of memory management in C++11[cite: 1].

The phrase `weak_ptr` gains meaning alongside `shared_ptr` and came into existence to solve a problem that is extremely important in the shared ownership model of `shared_ptr`[cite: 1].

If we want to state its core function in one sentence, it is:

> **`weak_ptr` can observe an object without owning that object.**[cite: 1]

The statement above might sound simple at first, but this exact feature solves one of the most important problems of `shared_ptr`[cite: 1].

The problem in question is the **Ownership Cycle**[cite: 1].

If we understand the ownership cycle correctly, we have almost understood the main reason for the existence of `weak_ptr` as well[cite: 1].

---

# 2. What Was the Problem Before `weak_ptr`?

To understand `weak_ptr`, it is better to first know why `shared_ptr` came into existence in the first place[cite: 1].

The `shared_ptr` pointer is meant for times when several different parts of the program need to own an object[cite: 1].

For example:

```cpp
auto person = std::make_shared<Person>();

auto a = person;
auto b = person;

```

Here we have three `shared_ptr` instances:

```text
person ──┐
         │
a ───────┼──> Person
         │
b ───────┘

```

What is important here is that all three `shared_ptr` instances own the exact same `Person`.

The Reference Count property also tracks the number of active owners.

```text
person + a + b = 3 owners

```

When one of them goes away:

```text
person + a = 2 owners

```

When another one goes away as well:

```text
person = 1 owner

```

And finally, when the last owner goes away:

```text
0 owners

```

The object is also Destroyed.

This behavior is very good.

However, there is a very important problem.

---

# 3. How Does `shared_ptr` Manage Ownership?

The `shared_ptr` pointer usually uses an internal structure called a **Control Block**.

Its simplified representation can look something like this:

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

The Strong Count indicates the number of `shared_ptr` owners.

The important point is that as long as Strong Count has not reached zero, the object is usually not Destroyed.

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

And when:

```cpp
p2.reset();

```

is executed:

```text
Strong Count = 1

```

Finally:

```cpp
p3.reset();

```

causes the last owner to also go away.

As a result, the object can be Destroyed.

---

# 4. The Main Problem with `shared_ptr`: Ownership Cycles and Memory Leaks

The important point is that Reference Counting works very well when the ownerships of shared pointers do not form a cycle.

To understand the problem, suppose we have two classes:

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

The concept of the code above is that `A` owns `B` and `B` also owns `A`. Now:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();

a->b = b;
b->a = a;

```

Now the ownership structure looks like this:

```text
             ┌───────────────┐
             │               │
             │               ▼
      ┌───────────┐      ┌───────────┐
      │  Object A │ ────▶│  Object B │
      └───────────┘      └───────────┘
             ▲               │
             │               │
             └───────────────┘

```

The more precise statement is:

```text
A owns B
B owns A

```

This is an **Ownership Cycle**.

---

The most important point of this section is right here.

Suppose local variables `a` and `b` go out of Scope.

We might imagine:

```text
a destroyed
b destroyed

therefore:

A destroyed
B destroyed

```

However, such a thing does not necessarily happen. Why?

The reason is that `A` still holds a `shared_ptr` to `B`.

And `B` also still holds a `shared_ptr` to `A`.

Meaning:

```text
A ──shared_ptr──> B
▲                │
│                │
└──shared_ptr────┘

```

Therefore:

```text
A has 1 owner
B has 1 owner

```

Even if no other code from the outside owns them.

The key meaning is:

> Each object is kept alive by the other object.
> 
> 

As a result:

```text
A → B → A → B → A → ...

```

The cycle is never broken.

A very important point is that **Reference Counting is not broken**.

Here, Reference Counting is doing exactly what it was designed to do.

Each `shared_ptr` says:

> I still own this object.
> 
> 

And since `A` owns `B` and `B` owns `A`, neither counter can ever reach zero.

This is the exact problem that `weak_ptr` was created to solve.

---

An important point is that `weak_ptr` itself does not generally cure Memory Leaks.

Its main job is to prevent **Ownership Cycles** in `shared_ptr` structures.

Suppose:

```cpp
A ──shared_ptr──> B
B ──shared_ptr──> A

```

A cycle is created.

If we convert one of the relationships to:

```cpp
B ──weak_ptr──> A

```

Then:

```text
A ──shared_ptr──> B
B ──weak_ptr────> A

```

`B` no longer causes `A` to stay alive.

As a result, when the real owner of `A` goes away, `A` can be Destroyed.

This is precisely where `weak_ptr` can prevent Memory Leaks.

---

# 5. What Exactly Is `weak_ptr`?

The `weak_ptr` pointer is a Smart Pointer that can **point to an object managed by a `shared_ptr` without owning it**.

The key phrase:

> **Observe, but do not own.**
> 

Meaning:

> Observe, but do not be the owner.
> 
> 

For example:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

```

Here:

```text
shared ──owns──> int

weak ──observes──> int

```

The important point is that creating `weak` does not increase the Strong Count.

If:

```cpp
shared.use_count()

```

was equal to `1` before creating `weak`, the Strong Count will still be `1` after creating `weak`.

---

This is the most important feature of `weak_ptr`.

The following statement:

```cpp
std::weak_ptr<MyClass> weak;

```

does not mean that `weak` owns `MyClass`.

The only difference is that `weak` has a non-owning connection to an Object.

For this reason, `weak_ptr` does not increase the Strong Reference Count.

---

# 6. Fundamental Difference Between `shared_ptr` and `weak_ptr`

A simple statement to understand the difference between the two is this:

```text
shared_ptr:
"I own the object."

weak_ptr:
"I know about the object, but I do not own it."

```

The `shared_ptr` pointer keeps the Object alive.

The `weak_ptr` pointer does not keep the Object alive.

For instance:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

shared.reset();

```

Now, if `shared` was the last owner, `int` is Destroyed.

However, `weak` might still exist.

It just can no longer access a live Object.

---

# 7. How Does `weak_ptr` Break Ownership Cycles?

The main point is that in an ownership cycle, at least one of the ownership relationships must be non-owning.

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

The important point here is that `A` owns `B`.

However, `B` only observes `A`.

As a result:

```text
A owns B
B observes A

```

There is no longer an ownership cycle.

---

# 8. Creating and Passing `weak_ptr` in a Setter

Creating a `weak_ptr` is usually done from a `shared_ptr`:

```cpp
auto shared = std::make_shared<MyClass>();

std::weak_ptr<MyClass> weak = shared;

```

Now `weak` only connects to that same Control Block.

Also, `weak_ptr` cannot independently manage an Object with `new`.

For example, this design usually makes no sense:

```cpp
std::weak_ptr<MyClass> weak(new MyClass());

```

because `weak_ptr` is fundamentally designed to observe Objects whose Ownership is managed by `shared_ptr`.

---

If a class is supposed to have a non-owning relationship with an Object, the Setter can receive a `weak_ptr`.

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

Now:

```cpp
auto object = std::make_shared<MyClass>();

Observer observer;

observer.setValue(object);

```

Here, `observer` has not become the owner of `object`.

If:

```cpp
object.reset();

```

is executed and `object` is the last owner, the Object will be Destroyed.

---

# 9. Returning `weak_ptr` from a Getter

If a class maintains a non-owning connection, we can return it with a Getter:

```cpp
std::weak_ptr<MyClass> getValue() const
{
    return value;
}

```

The important point is that returning `weak_ptr` does not convert it to Ownership.
The Caller can receive a copy of `weak_ptr`.

---

In many designs:

```cpp
std::weak_ptr<MyClass> getValue() const
{
    return value;
}

```

is an appropriate choice.

This API tells the Caller:

> This Object might exist, but I do not own it and I cannot extend its lifetime simply by returning this value.
> 
> 

This topic is very important in class design.

If the Getter instead returned:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value.lock();
}

```

the Caller would receive a new owner.

Therefore, these two Getters are completely different from a design perspective.

---

# 10. Copy, Move, and `reset()` in `weak_ptr`

The `weak_ptr` pointer, like other regular Objects, is copyable. For example:

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

The important point is that by copying `weak_ptr`, the Object is not copied.

Even new Ownership is not created.

Only a new Observer is created.

---

The `weak_ptr` pointer supports Copy and Move.

Copy:

```cpp
auto w2 = w1;

```

creates a new Observer.

Move:

```cpp
auto w2 = std::move(w1);

```

transfers the state of `w1` to `w2`.

After the Move, `w1` will usually be Empty.

---

The `reset()` method severs the connection of `weak_ptr` with the Control Block. For example:

```cpp
std::weak_ptr<MyClass> weak = shared;

weak.reset();

```

Now `weak` no longer observes the Object.

Here, `reset()` on `weak_ptr` does not cause the Object to be Destroyed.

Because `weak_ptr` is not the owner of the Object.

If:

```cpp
shared.use_count() == 1

```

executing:

```cpp
weak.reset();

```

has no effect on the lifetime of the Object.

The Object is still managed by `shared`.

---

# 11. Checking Emptiness and `expired()`

To check if `weak_ptr` has no connection to the Object, we can use:

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

However, there is a very important point.

`expired()` alone is usually not suitable for making a final decision about accessing the Object.

---

The phrase `expired()` determines whether the Object observed by `weak_ptr` is still alive or not. For example:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

std::cout << weak.expired();

```

As long as `shared` keeps the Object alive, the result will be `false`.

Later:

```cpp
shared.reset();

```

If `shared` was the last owner:

```cpp
weak.expired()

```

will become `true`.

The `expired()` method only checks the status at a single instant.

---

# 12. Converting `weak_ptr` to `shared_ptr` Using `lock()`

This section is one of the most important parts of `weak_ptr`.

The `weak_ptr` pointer does not directly allow the use of:

```cpp
weak->someFunction();

```

First, we must convert it to a `shared_ptr` using `lock()`:

```cpp
auto shared = weak.lock();

```

If the Object is still alive, `shared` will be a valid `shared_ptr`.

If the Object has been destroyed, `shared` will be empty.

For example:

```cpp
if (auto shared = weak.lock())
{
    shared->doSomething();
}

```

This is one of the most important usage patterns of `weak_ptr`.

---

The `lock()` method does not just check if the Object is alive. Rather, if the Object is alive, it creates a `shared_ptr` that temporarily preserves Ownership. This topic is extremely important. Suppose:

```cpp
if (!weak.expired())
{
    // ...
}

```

and then later we want to use the Object.

In multithreaded environments or even more complex designs, the Object might be Destroyed between checking and using.

For this reason, the better pattern is:

```cpp
if (auto shared = weak.lock())
{
    shared->doSomething();
}

```

The concept of this code is:

> If the Object is still alive, create a temporary owner and then use the Object.
> 
> 

This approach is much safer.

---

What `lock()` does is atomically check the **Control Block** to see if the target object is still alive; and if it is alive, it creates a new `shared_ptr` in that same operation to acquire ownership of the object.

The importance of this behavior becomes clear when another `shared_ptr` exists simultaneously as the **last owner of the object** and intends to `reset()` it at that exact moment. In such conditions, `weak_ptr::lock()` and `reset()` settle atomically which state holds true:

* If `lock()` succeeds first, a new `shared_ptr` is created and **Reference Count goes from 1 to 2**. Then, if the previous `shared_ptr` is `reset()`, the Reference Count decreases to 1 and the object remains alive.


* If `reset()` happens first and as a result the Reference Count reaches zero, the object is destroyed and `lock()` returns an empty `shared_ptr` (`nullptr`).



Therefore, the important point is that `lock()` does not merely **check for the existence of the object** first and then, in a separate step, construct the `shared_ptr`. Such an approach could create a Race Condition:

```text
Does object exist? → Yes
                     ↓
               [Another Thread]
                     ↓
                   reset()
                     ↓
                 object destroyed
                     ↓
          Now access object ❌

```

Instead, `lock()` performs the **"check object liveness + acquire ownership"** operation atomically. As a result, it either acquires new ownership before the last owner is `reset()` and keeps the object alive, or if the last owner has already relinquished its ownership, `lock()` encounters an empty result.

Simply put, `weak_ptr::lock()` guarantees that there is no exploitable time window between **"the object is still alive"** and **"I acquired its ownership."**

---

# 13. Accessing Object Members via `weak_ptr`

The `weak_ptr` pointer itself does not have `operator->` for direct access to the Object.

Therefore, this code is not correct:

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

And for Members:

```cpp
if (auto ptr = weak.lock())
{
    std::cout << ptr->value;
}

```

---

No.

This decision is entirely intentional.

The reason is that the Object might no longer exist.

If `weak_ptr` allowed:

```cpp
weak->doSomething();

```

the programmer might use the Object without paying attention to its Lifetime.

The `lock()` method forces the programmer to first check the Lifetime status and, if possible, create a temporary `shared_ptr`.

---

# 14. `weak_ptr` and Raw Pointers

Directly, `weak_ptr` does not have a function like:

```cpp
get()

```

that returns a Raw Pointer to the Object.

This design is also intentional.

To reach the Object, one must first perform:

```cpp
auto shared = weak.lock();

```

Then:

```cpp
MyClass* raw = shared.get();

```

will be available to us.

---

The dangerous scenario is taking ownership of Lifetime management ourselves after obtaining the Raw Pointer.

For example:

```cpp
auto shared = weak.lock();

if (shared)
{
    MyClass* raw = shared.get();

    raw->doSomething();
}

```

In this example there is no problem, because `shared` is alive until the end of the Scope.

However, if we write inside a main block:

```cpp
MyClass* raw = weak.lock().get();

```

a very serious problem occurs.

The `lock()` function created a Temporary `shared_ptr`.

Perhaps before the end of this code block execution and after getting a Raw Pointer, the main object on the Heap gets destroyed, and after its destruction, without being aware of this event, at the end of this block of code and after the main object is Deleted, we might try to access the Deleted object via this Raw Pointer and face a runtime error due to a Dangling Pointer!

As a result, the Raw Pointer might become Dangling.

Therefore, this is very dangerous:

```cpp
MyClass* raw = weak.lock().get();

```

Instead:

```cpp
if (auto shared = weak.lock())
{
    MyClass* raw = shared.get();

    raw->doSomething();
}

```

is safer.

---

# 15. Using `weak_ptr` in Classes and the Parent/Child Pattern

A very common example is holding the Parent inside the Child.

Suppose:

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

Here `Child` knows who its Parent is.

However, Child does not own Parent.

This prevents cycle creation.

---

The Parent/Child concept (Composition Pattern) is one of the best examples for understanding `weak_ptr`.

Suppose:

```text
Parent
   │
   │ owns
   ▼
Child

```

If Child also holds Parent using `shared_ptr`:

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

a cycle is created.

Usually, the correct method is:

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

In this state, Parent owns Child.

However, Child only observes Parent.

---

# 16. The Observer Pattern

Another important use case of `weak_ptr` is the Observer Pattern.

Suppose we have a main Object:

```cpp
auto subject = std::make_shared<Subject>();

```

and several Observers that need to know when Subject exists.

If Observers hold `shared_ptr`, they might unintentionally keep Subject alive.

Instead:

```cpp
std::weak_ptr<Subject> subject;

```

is more suitable.

The concept of Observer is:

> I want to access the Object if it exists, but I should not cause it to stay alive.
> 
> 

---

# 17. Full Example of a Broken Cycle and Fixing It with `weak_ptr`

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

The apparent intention is that with the end of the Scope, both Objects should be Destroyed.

However, such a thing does not happen.

The reason is:

```text
A owns B
B owns A

```

As a result, each Object is kept alive by the other.

---

To fix it, we make one of the relationships non-owning:

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

The Ownership Cycle concept is eliminated.

When Scope ends:

```text
a destroyed
b destroyed

```

As a result, the Strong Count related to the Objects can reach zero.

The Objects are also Destroyed.

---

# 18. `use_count()`, Strong Count, Weak Count, and Control Block

The `weak_ptr` also has the function:

```cpp
use_count()

```

For example:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

std::cout << weak.use_count();

```

The displayed value is the number of **Strong Owners**.

Meaning the number of `shared_ptr` instances that own the Object.

Now here, the `weak_ptr` pointer does not add to this number.

---

The Control Block concept usually has two important counting concepts:

```text
Strong Count
Weak Count

```

The Strong Count shows the number of real owners, i.e., `shared_ptr` instances.

The Weak Count relates to the existence of `weak_ptr` Observers and managing the Control Block itself.

These two Counts should not be confused with each other.

The most important rule for the Object is:

```text
Strong Count == 0

```

can cause the Object to be Destroyed.

However, the Control Block might still remain due to the existence of `weak_ptr`.

---

The Control Block concept is an internal structure that `shared_ptr` and `weak_ptr` use to manage Ownership.

Simplified representation:

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

The important point is that `weak_ptr` does not directly own the Object.

Rather, it connects to the Control Block associated with the Ownership of that Object.

For this reason, it can determine whether the Object is still alive or not.

---

This question is very important.

Suppose:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

```

Now:

```text
Strong Count = 1

```

and one `weak_ptr` also exists.

If:

```cpp
shared.reset();

```

is executed:

```text
Strong Count = 0

```

The Object is Destroyed.

However, the Control Block might still remain so `weak` can determine its state.

So:

```text
Object lifetime

```

and:

```text
Control Block lifetime

```

are two completely different concepts.

This is one of the important points in understanding the internals of `weak_ptr`.

---

# 19. Differences Between `weak_ptr` and `shared_ptr`

The table below shows the main differences:

| Feature | `shared_ptr` | `weak_ptr` |
| --- | --- | --- |
| Owns the Object? | Yes | No |
| Increases Strong Count? | Yes | No |
| Keeps the Object alive? | Yes | No |
| Has Copy? | Yes | Yes |
| Has Move? | Yes | Yes |
| Has `reset()`? | Yes | Yes |
| Has direct `get()`? | Yes | No |
| Has `operator->`? | Yes | No |
| Has `lock()`? | No | Yes |
| Has `expired()`? | No | Yes |
| Suitable for ownership cycles? | Dangerous | Suitable for breaking cycles |
| Creates Ownership? | Yes | No |

---

# 20. Differences Between `weak_ptr` and `unique_ptr`

The `unique_ptr` and `weak_ptr` pointers are different even in terms of Ownership philosophy.

The `unique_ptr` pointer means:

> I am the sole owner of this Object.
> 
> 

The `weak_ptr` pointer means:

> I do not own this Object at all; I just want to observe it.
> 
> 

Therefore:

```cpp
std::unique_ptr<MyClass>

```

is for **Unique Ownership**.

However:

```cpp
std::weak_ptr<MyClass>

```

is for **Non-owning Observation** of an Object managed by `shared_ptr`.

---

# 21. When Should or Shouldn't We Use `weak_ptr`?

1- When we want to observe an Object but do not want to extend its Lifetime, `weak_ptr` is a suitable choice.

2- When relationships between Objects might create a cycle, one of the Ownership relationships can be modeled with `weak_ptr`.

3- In the Observer Pattern, Parent/Child Relationship, and Caches, `weak_ptr` can be an appropriate choice.

The general concept is:

> Wherever a relationship exists, Ownership does not necessarily exist.
> 
> 

This sentence is one of the best ways to understand `weak_ptr`.

---

We must know that we should not use `weak_ptr` everywhere just because it is a powerful tool.

If a class really needs to own the Object, `weak_ptr` is not the right choice.

For example:

```cpp
class Car
{
    std::weak_ptr<Engine> engine;
};

```

If Car cannot function without Engine and needs to keep Engine alive, `shared_ptr` or even `unique_ptr` is likely a better choice.

The `weak_ptr` pointer makes sense when non-ownership is part of the design.

---

# 22. Common Mistakes

One common mistake is thinking that `weak_ptr` keeps the Object alive.

Another mistake is directly dereferencing `weak_ptr`:

```cpp
weak->foo();

```

This is not possible.

The correct form:

```cpp
if (auto ptr = weak.lock())
{
    ptr->foo();
}

```

Another mistake is using this expression:

```cpp
if (!weak.expired())
{
    auto ptr = weak.lock();
}

```

In most cases, this check is redundant.

The better form:

```cpp
if (auto ptr = weak.lock())
{
    ptr->foo();
}

```

Another dangerous mistake is getting a Raw Pointer from a Temporary:

```cpp
auto raw = weak.lock().get();

```

This Raw Pointer can immediately become Dangling.

The safer form:

```cpp
if (auto ptr = weak.lock())
{
    auto raw = ptr.get();

    raw->foo();
}

```

Of course, in many cases we do not need a Raw Pointer at all and it is better to directly use:

```cpp
ptr->foo();

```

---

# 23. A Complete and More Realistic Example

The code below shows a common Parent/Child design:

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

The Ownership relationship here looks like this:

```text
Parent ──shared_ptr──> Child
   ▲
   │
   │ weak_ptr
   │
 Child

```

In this design, Child does not keep Parent alive.

---

# 24. Final Summary

The `shared_ptr` pointer was created for **Shared Ownership**.

The `weak_ptr` pointer was created for when we want to communicate with an Object, but do not want to own it.

The main problem arises when several `shared_ptr` instances keep each other alive cyclically.

The statement:

```text
A owns B
B owns A

```

creates an Ownership Cycle.

The result of this cycle is that Reference Count never reaches zero.

The `weak_ptr` pointer can make one of these relationships non-owning:

```text
A owns B
B observes A

```

As a result, the cycle is broken.

Creating `weak_ptr`:

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

Checking for expiration:

```cpp
weak.expired();

```

Safe conversion to `shared_ptr`:

```cpp
auto shared = weak.lock();

```

Accessing the Object:

```cpp
if (auto shared = weak.lock())
{
    shared->doSomething();
}

```

Accessing Raw Pointer:

```cpp
if (auto shared = weak.lock())
{
    MyClass* raw = shared.get();
}

```

An important point is that `weak_ptr` does not directly have `get()`, and to access the Object we must first obtain a valid `shared_ptr` using `lock()`.

Another important point is that `weak_ptr` does not own the Object and therefore cannot keep the Object alive.

---

Smart Pointers can be remembered with four simple questions in mind:

```text
Who owns the object?
Who shares ownership?
Who only observes it?
Can ownership form a cycle?

```

The answers are usually as follows:

```text
One owner
    → unique_ptr

Multiple owners
    → shared_ptr

Non-owning observer of shared ownership
    → weak_ptr

```

The final point is that `weak_ptr` is not a "weaker" version of `shared_ptr`.

The word `weak` in its name does not mean it has fewer capabilities.

The real meaning of this name is:

> **Weak Ownership — or more precisely, the absence of Strong Ownership.**
> 

`weak_ptr` intentionally does not keep the Object alive.

And that is precisely why it can break `shared_ptr` ownership cycles.

If a programmer remembers only one sentence from this tutorial, it should be this:

> **`shared_ptr` says "as long as I exist, the Object must stay alive"; `weak_ptr` says "if the Object is still alive, let me see it, but my presence should not prevent its destruction."**
> 

---

## 🤝 Contributions

| GitHub | LinkedIn | Email | Site | Telegram |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://www.google.com/search?q=https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.google.com/search?q=https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](https://www.google.com/search?q=mailto%3Ahadi.abbasi.programmer%40gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](https://www.google.com/search?q=https://t.me/Hadi_Abbasi_Programmer) |