<div align="center">

[🇺🇸 English](./make_shared.md) | [🇮🇷 فارسی](../../fa/cpp14/make_shared.md)

</div>

---

# Comprehensive Guide to `std::make_shared` in C++

## Introduction to `std::make_shared`

The `std::make_shared` function is an important Standard Library utility in C++ for creating Objects that are intended to be managed by `std::shared_ptr`.

It is provided by the following header:

```cpp
#include <memory>
```

The typical usage looks like this:

```cpp
auto user = std::make_shared<User>("Ali", 30);
```

In this example, `std::make_shared()` creates an Object of type `User` and returns a `std::shared_ptr<User>`.

An important point is that `std::make_shared` is not itself a Smart Pointer; it is a Function Template used to create a `std::shared_ptr`.

Simply put, the relationship can be viewed as follows:

```text
std::make_shared<T>(...)
        │
        ▼
std::shared_ptr<T>
        │
        ▼
      Object
```

The core functionality of `std::make_shared` has been available since C++11, while Array support and `std::make_shared_for_overwrite` were added in C++20. In C++26, the functions related to `make_shared` have also been specified as `constexpr`. ([Cppreference][1])

---

## The Concept of Shared Ownership

To properly understand `std::make_shared`, we first need to understand the concept of `shared ownership`.

Suppose we have an Object that needs to be owned by several different parts of a program:

```cpp
auto user = std::make_shared<User>("Ali", 30);

auto service = user;
auto repository = user;
```

There are now three `std::shared_ptr` objects managing the same Object:

```text
user ───────────────┐
                    │
service ────────────┼──────► User Object
                    │
repository ─────────┘
```

In this situation, the Object remains alive as long as at least one `shared_ptr` owns it.

If one of the Pointers is destroyed:

```cpp
service.reset();
```

the Object is still alive because other Pointers still own it.

If the last `shared_ptr` is also destroyed:

```cpp
user.reset();
repository.reset();
```

the Object is destroyed as well.

This ownership model is what `shared_ptr` is designed for.

---

## Ownership vs. Access

One of the most important points when using `shared_ptr` is that access to an Object is not the same as ownership of that Object.

Suppose:

```cpp
auto user = std::make_shared<User>();
```

If a function only needs temporary access to the User, it does not necessarily need to receive a `shared_ptr`.

For example, if the function only uses the Object without owning it, this design may be more appropriate:

```cpp
void printUser(const User& user);
```

Or, if nullable access is required:

```cpp
void printUser(const User* user);
```

In contrast, if the function genuinely needs to share ownership, then using:

```cpp
std::shared_ptr<User>
```

makes sense.

Therefore, `shared_ptr` should not be considered merely a safer Pointer than a Raw Pointer; its primary concept is **Shared Ownership**.

---

# `new` vs. `std::make_shared`

To understand the advantages of `make_shared`, let's first examine the traditional approach.

## Creating an Object with `new`

The direct approach is:

```cpp
User* user = new User("Ali", 30);
```

Here, we receive a Raw Pointer.

As a result, the program is responsible for managing the Lifetime:

```cpp
delete user;
```

If `delete` is forgotten, a Memory Leak occurs.

---

## Creating an Object with `shared_ptr`

To manage the Lifetime automatically, we can write:

```cpp
std::shared_ptr<User> user(
    new User("Ali", 30)
);
```

In this case, `shared_ptr` owns the Object and destroys it when the last Owner is gone.

However, there is an important point: normally, creating the Object and creating the Control Block require two separate Allocations. In contrast, `make_shared` normally places both in a single Allocation. ([Cppreference][1])

---

## Creating an Object with `make_shared`

The more modern approach is:

```cpp
auto user = std::make_shared<User>("Ali", 30);
```

Here:

* A `User` is created.
* A `shared_ptr<User>` is created.
* Ownership is managed by `shared_ptr` from the beginning.
* In common implementations, the Object and Control Block are provided by a single Allocation.

This approach is generally simpler, more readable, and more Allocation-efficient than `shared_ptr(new T(...))`. ([Cppreference][1])

---

# Internal Structure of `shared_ptr`

To understand `make_shared`, we need to understand the concept of a `Control Block`.

A `shared_ptr` can conceptually be viewed as having two parts:

```text
shared_ptr
├── pointer to managed object
└── pointer to control block
```

The Control Block generally stores information related to Ownership.

The exact structure of the Control Block is Implementation-defined, but it typically contains information such as Reference Count, Weak Count, Deleter, and the data required to manage the Lifetime.

Conceptually, the structure can be viewed as follows:

```text
┌─────────────────────────────┐
│       Control Block         │
│                             │
│   shared ownership count    │
│   weak ownership count      │
│   deleter / allocator       │
│   implementation data       │
└──────────────┬──────────────┘
               │
               ▼
        ┌──────────────┐
        │     User     │
        └──────────────┘
```

The exact Layout of this structure is not specified by the Standard and should not be relied upon as a property of a particular Implementation.

---

# Allocation Benefits of `make_shared`

One of the most important benefits of `make_shared` concerns how memory is allocated.

In the usual case:

```cpp
std::shared_ptr<User>(
    new User("Ali", 30)
);
```

there are normally two Allocations:

```text
Allocation #1
┌─────────────────┐
│     User        │
└─────────────────┘

Allocation #2
┌─────────────────┐
│  Control Block  │
└─────────────────┘
```

However:

```cpp
auto user = std::make_shared<User>("Ali", 30);
```

is commonly implemented using a single Allocation:

```text
Allocation #1
┌───────────────────────────────┐
│       Control Block           │
│                               │
│       User Object             │
└───────────────────────────────┘
```

The Standard does not make this an absolute requirement, but `make_shared` is commonly implemented this way, and known Implementations use this model. ([Cppreference][1])

---

# Performance Benefits of `make_shared`

Reducing the number of Allocations can provide practical benefits.

Each Allocation can involve costs such as:

* Allocator management
* Requesting memory from the Runtime
* Allocation-related Metadata
* Fragmentation
* Synchronization costs in some Allocators

Therefore, reducing the number of Allocations can be useful in programs that create a large number of small Objects.

However, the following statement is not precise:

> `make_shared` is always faster.

A more accurate statement is:

> `make_shared` usually provides an opportunity to reduce the number of Allocations and can therefore improve Allocation overhead and memory Locality.

The final Performance still depends on the type of Object, Object size, Allocator, Lifetime pattern, and how `weak_ptr` is used.

---

# The Concept of Memory Locality

Placing the Object and Control Block in a single Allocation can provide better memory Locality.

With separate Allocations:

```text
Memory:

[User Object]                     [Control Block]
     ↑                                  ↑
     └──────────── distance ───────────┘
```

With the usual `make_shared` implementation:

```text
Memory:

[Control Block | User Object]
```

As a result, data related to Object management is generally located within the same memory region.

This can be beneficial for Cache behavior and Memory Access Patterns, but the actual benefit depends entirely on the Application and Implementation.

---

# Exception Safety Benefits

Another important reason to use `make_shared` is that it simplifies Exception management.

Consider this code:

```cpp
process(
    std::shared_ptr<User>(new User()),
    createSomething()
);
```

In older standards, the order in which Function Arguments were evaluated could create a window for a Memory Leak in such a pattern.

For example, if `new User()` succeeded but another expression threw an Exception before the `shared_ptr` was constructed, the created Object could be left without an Owner.

In contrast:

```cpp
process(
    std::make_shared<User>(),
    createSomething()
);
```

the Object is created directly under the management of `shared_ptr`.

This difference was particularly important in standards before C++17. According to Standard Library documentation, the well-known `f(std::shared_ptr<int>(new int(42)), g())` scenario could result in a Leak under the relevant older-standard evaluation rules, whereas using `make_shared` avoids this problem. ([Cppreference][1])

---

# Passing Arguments to the Constructor

One of the important features of `make_shared` is that its Arguments are used to construct the Object.

Suppose:

```cpp
class User
{
public:
    User(std::string name, int age)
        : name_(std::move(name)),
          age_(age)
    {
    }

private:
    std::string name_;
    int age_;
};
```

We can now write:

```cpp
auto user = std::make_shared<User>("Ali", 30);
```

The Object is constructed in a way that is essentially equivalent to constructing `User` with the same Arguments.

Conceptually:

```cpp
User("Ali", 30)
```

As a result, `make_shared` can be used with different Constructors:

```cpp
auto a = std::make_shared<User>();

auto b = std::make_shared<User>("Ali");

auto c = std::make_shared<User>("Ali", 30);
```

Of course, each form is valid only if the corresponding Constructor exists in the class.

---

# Using `auto`

One of the most common Modern C++ patterns is using `auto`:

```cpp
auto user = std::make_shared<User>("Ali", 30);
```

The type of `user` in this example is:

```cpp
std::shared_ptr<User>
```

Therefore, explicitly specifying the type is also possible:

```cpp
std::shared_ptr<User> user =
    std::make_shared<User>("Ali", 30);
```

However, using `auto` generally makes the code shorter and easier to read.

---

# Using `make_shared` for Polymorphism

One important use of `shared_ptr` is managing Polymorphic Objects.

Suppose:

```cpp
class Animal
{
public:
    virtual ~Animal() = default;
    virtual void speak() = 0;
};
```

And the derived class:

```cpp
class Dog : public Animal
{
public:
    void speak() override
    {
    }
};
```

We can write:

```cpp
std::shared_ptr<Animal> animal =
    std::make_shared<Dog>();
```

Here, the Pointer type is:

```text
shared_ptr<Animal>
```

but the actual Object is:

```text
Dog
```

Therefore:

```cpp
animal->speak();
```

reaches the `Dog` implementation through Virtual Dispatch.

Having a Virtual Destructor in the Base Class is important for such designs:

```cpp
virtual ~Animal() = default;
```

---

# Using `enable_shared_from_this`

One important mechanism related to `shared_ptr` is:

```cpp
std::enable_shared_from_this
```

Suppose an Object needs to return a `shared_ptr` to itself.

A correct pattern can be:

```cpp
class User
    : public std::enable_shared_from_this<User>
{
public:
    std::shared_ptr<User> getShared()
    {
        return shared_from_this();
    }
};
```

We can then create the Object using `make_shared`:

```cpp
auto user = std::make_shared<User>();

auto another = user->getShared();
```

In this case, `another` and `user` share the same Ownership.

An important point is that we should not manually do the following for an Object that is already managed by a `shared_ptr`:

```cpp
std::shared_ptr<User>(this);
```

This can create an independent Control Block and ultimately lead to incorrect Lifetime management and even a Double Delete.

The `enable_shared_from_this` mechanism is provided to avoid such designs. `make_shared` is also compatible with this mechanism. ([Cppreference][1])

---

# Checking the Number of Owners with `use_count`

To observe the number of `shared_ptr` instances participating in Shared Ownership, we can use `use_count`.

For example:

```cpp
auto p1 = std::make_shared<int>(42);

std::cout << p1.use_count() << '\n';

auto p2 = p1;

std::cout << p1.use_count() << '\n';
```

Typical output:

```text
1
2
```

After:

```cpp
p2.reset();
```

the number of Owners becomes one again.

An important design point is that `use_count()` generally should not be used as the basis for core program logic because its value can change between observations in Concurrent programs.

---

# Accessing the Object with `get`

The `get` function returns the Raw Pointer associated with the Object:

```cpp
auto user = std::make_shared<User>();

User* raw = user.get();
```

Here, `raw` does not own the Object.

Therefore, we must not write:

```cpp
delete raw;
```

because the Lifetime is still managed by `shared_ptr`.

The `get` function is mainly useful when working with an older API that accepts only a Raw Pointer:

```cpp
void legacy_api(User* user);
```

In this case, we can write:

```cpp
legacy_api(user.get());
```

However, we must make sure that the API does not store the Pointer and use it later.

The `get()` function only provides access to the Stored Pointer; it does not transfer Ownership. ([Cppreference][2])

---

# Managing Lifetime with `reset`

`reset` can be used to detach a `shared_ptr` from its Object:

```cpp
auto user = std::make_shared<User>();

user.reset();
```

If `user` is the last Owner, the Object is destroyed.

However, if another Pointer exists:

```cpp
auto user = std::make_shared<User>();

auto another = user;

user.reset();
```

the Object remains alive because `another` still owns it.

---

# Copying and Moving `shared_ptr`

Copying a `shared_ptr` means adding another Owner:

```cpp
auto p1 = std::make_shared<User>();

auto p2 = p1;
```

Here:

```text
p1 ───────┐
          ├────► User
p2 ───────┘
```

However, `move` transfers the existing ownership state to another Pointer:

```cpp
auto p1 = std::make_shared<User>();

auto p2 = std::move(p1);
```

After this operation, `p2` owns the Object and `p1` is in a moved-from state and will typically be empty.

The difference can be summarized as:

```text
copy
 └── shared ownership

move
 └── transfer of ownership state
```

---

# Preventing Circular Ownership

One of the most important problems with `shared_ptr` is the creation of a Reference Cycle.

Suppose:

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

Then:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();

a->b = b;
b->a = a;
```

The structure becomes:

```text
A ───── shared_ptr ─────► B
▲                         │
└──────── shared_ptr ─────┘
```

Even if the external Owners `a` and `b` are destroyed, each Object is still owned by the other Object.

As a result, the Reference Count never reaches zero.

---

# Using `weak_ptr` to Solve the Cycle

For a relationship that does not represent ownership, `weak_ptr` should be used.

For example:

```cpp
class B
{
public:
    std::weak_ptr<A> a;
};
```

The structure is now:

```text
A ───── shared_ptr ─────► B
▲                         │
└───────── weak_ptr ──────┘
```

`weak_ptr` does not own the Object and does not increase the Shared Ownership Count.

To access the Object, we must first call `lock()`:

```cpp
if (auto owner = b->a.lock())
{
    owner->doSomething();
}
```

This pattern is very important for Observer Relationships, Parent/Child Relationships, and preventing Ownership Cycles.

---

# The Relationship Between `make_shared` and `weak_ptr`

Using `make_shared` introduces an important Trade-off involving `weak_ptr`.

Suppose:

```cpp
auto object = std::make_shared<LargeObject>();

std::weak_ptr<LargeObject> weak = object;

object.reset();
```

After the last `shared_ptr` is reset, the Lifetime of the `LargeObject` itself ends.

However, the Control Block may still be needed by the `weak_ptr`.

With `make_shared`, the Object and Control Block are typically placed in a single Allocation:

```text
┌────────────────────────────┐
│ Control Block              │
│                            │
│ LargeObject                │
└────────────────────────────┘
```

Therefore, if the Object is very large, the shared Allocation may not be completely released until the last `weak_ptr` is gone. This is one of the most important Trade-offs of `make_shared`. ([Cppreference][1])

In contrast:

```cpp
std::shared_ptr<LargeObject>(
    new LargeObject()
);
```

normally uses separate Allocations for the Object and Control Block.

As a result, after the Object is destroyed, the Object's memory can be released independently of the Control Block.

---

# Custom Deleter Limitation

One important difference between `make_shared` and the `shared_ptr` Constructors is that `make_shared` does not provide a way to specify a Custom Deleter.

For example, this code allows us to define a custom Deleter:

```cpp
auto ptr = std::shared_ptr<Resource>(
    resource,
    [](Resource* resource)
    {
        release_resource(resource);
    }
);
```

However, there is no equivalent with `make_shared`:

```cpp
// There is no custom deleter parameter here.
auto ptr = std::make_shared<Resource>();
```

Therefore, if a Resource requires a special Release mechanism, a `shared_ptr` Constructor can be a more suitable option. ([Cppreference][1])

---

# Private Constructor Limitation

Suppose the Constructor of a class is `private`:

```cpp
class SingletonLike
{
private:
    SingletonLike() = default;

public:
    static std::shared_ptr<SingletonLike> create();
};
```

We might expect this internal Factory to be able to use `make_shared` directly:

```cpp
std::shared_ptr<SingletonLike>
SingletonLike::create()
{
    return std::make_shared<SingletonLike>();
}
```

However, access to the Constructor occurs in the internal Context of `make_shared`, not in the Context of the `create` function.

As a result, `make_shared` cannot use the private Constructor merely because the Factory itself has access to that Constructor.

In contrast, direct construction with `new` from a Context where the Constructor is accessible can behave differently:

```cpp
std::shared_ptr<SingletonLike>(
    new SingletonLike()
);
```

This is one of the formal differences between the two approaches. ([Cppreference][1])

---

# Difference with a Class-Specific `operator new`

Another professional consideration is the behavior of a class-specific `operator new`.

Suppose a class provides custom Allocation behavior:

```cpp
class MyType
{
public:
    static void* operator new(std::size_t size);

    static void operator delete(void* ptr) noexcept;
};
```

With direct construction:

```cpp
auto ptr = std::shared_ptr<MyType>(
    new MyType()
);
```

the expression `new MyType()` can use the Class-Specific `operator new`.

However, `make_shared` uses its own internal Allocation mechanism, and the Standard does not make it equivalent to using the Class-Specific `operator new`.

Standard Library documentation also explicitly notes that `make_shared` uses `::new`, and its behavior in this respect can differ from `shared_ptr(new T(...))`. ([Cppreference][1])

Therefore, this difference should be taken seriously in systems with custom Memory Allocation.

---

# Array Support in `make_shared`

In older versions of C++, `make_shared` did not have Standard support for Arrays.

Starting with C++20, Array-related overloads were added to `make_shared`. ([Cppreference][1])

For a dynamically sized Array, we can write:

```cpp
auto values = std::make_shared<int[]>(100);
```

This creates an Array containing 100 elements.

For a fixed-size Array, we can write:

```cpp
auto values = std::make_shared<int[10]>();
```

It is also possible to initialize the Array with a specified value:

```cpp
auto values = std::make_shared<int[]>(100, 42);
```

In this case, the Array elements are initialized with the specified value. The Array overloads of `make_shared` are available starting with C++20. ([Cppreference][1])

---

# Bounded and Unbounded Arrays

In the following syntax:

```cpp
std::make_shared<int[]>(100);
```

the type:

```text
int[]
```

is an Unbounded Array Type, and its size is specified at Runtime.

In contrast:

```cpp
std::make_shared<int[10]>();
```

uses the type:

```text
int[10]
```

which is a Bounded Array Type.

These two cases activate different overloads of `make_shared`.

---

# Using `make_shared_for_overwrite`

C++20 introduced another function named `make_shared_for_overwrite`.

This function is useful when we want to create Storage and then write the appropriate values into it ourselves.

For a regular Object:

```cpp
auto object =
    std::make_shared_for_overwrite<MyType>();
```

For an Array:

```cpp
auto buffer =
    std::make_shared_for_overwrite<std::byte[]>(1024);
```

The main difference is that this API is designed for Default Initialization and does not guarantee a specified initial value for the Object or Array elements. ([Cppreference][1])

Therefore, we must not read from such an Object before properly initializing it.

---

# Appropriate Use of `make_shared_for_overwrite`

A suitable scenario for this API is a Buffer that is going to be immediately filled by an external API.

For example:

```cpp
auto buffer =
    std::make_shared_for_overwrite<std::byte[]>(4096);

read_data(buffer.get(), 4096);
```

In such a scenario, if `read_data` completely fills the Buffer before it is read, performing additional Initialization may be unnecessary.

However, `make_shared_for_overwrite` should be used deliberately because reading the Object or its elements before proper Initialization can cause problems.

The `make_shared_for_overwrite` capability is identified by the following Feature-Test Macro:

```cpp
__cpp_lib_smart_ptr_for_overwrite
```

Its standard value for C++20 is `202002L`. ([Cppreference][3])

---

# Using `allocate_shared`

If more control over Allocation is required, the Standard Library provides `allocate_shared`.

A simple usage looks like this:

```cpp
auto object =
    std::allocate_shared<MyType>(allocator);
```

The main difference is that we provide the desired Allocator to `allocate_shared`.

Conceptually:

```text
make_shared
    │
    └── Standard/default allocation strategy

allocate_shared
    │
    └── User-provided allocator
```

Like `make_shared`, `allocate_shared` also normally places the Object and Control Block in a single Allocation. ([Cppreference][4])

---

# Using `allocate_shared` with a Memory Pool

This capability is useful for systems that use a Memory Resource or Memory Pool.

For example, together with `std::pmr`, we can create an appropriate Allocator and then create Objects using `allocate_shared`.

A simple example:

```cpp
#include <memory>
#include <memory_resource>

class Node
{
public:
    explicit Node(int value)
        : value_(value)
    {
    }

private:
    int value_;
};

int main()
{
    std::byte buffer[4096];

    std::pmr::monotonic_buffer_resource resource(
        buffer,
        sizeof(buffer)
    );

    std::pmr::polymorphic_allocator<Node> allocator(
        &resource
    );

    auto node =
        std::allocate_shared<Node>(
            allocator,
            42
        );
}
```

In such a scenario, the Allocator plays a role not only in constructing the Object but also in managing the Allocation associated with the Control Block. ([Cppreference][4])

---

# Difference Between `make_shared` and `allocate_shared`

The difference between these two can be summarized as follows:

| Feature                    | `make_shared` | `allocate_shared` |
| -------------------------- | ------------- | ----------------- |
| Creates `shared_ptr`       | Yes           | Yes               |
| Creates Object             | Yes           | Yes               |
| Custom Allocator           | No            | Yes               |
| Usually one Allocation     | Yes           | Yes               |
| Independent Custom Deleter | No            | No                |
| Suitable for Memory Pool   | Limited       | Suitable          |
| Standard                   | C++11         | C++11             |

With `allocate_shared`, the Allocator is part of the Allocation and Lifetime management mechanism and is typically retained in the Control Block so that it can be used for Deallocation when Ownership has completely ended. ([Cppreference][4])

---

# Difference Between `make_shared` and `make_unique`

These two functions should not be compared only in terms of Syntax.

The primary difference concerns Ownership.

With:

```cpp
auto ptr = std::make_unique<User>();
```

there is only one Owner.

With:

```cpp
auto ptr = std::make_shared<User>();
```

there can be multiple Owners.

Therefore:

```text
make_unique
    │
    ▼
unique_ptr
    │
    ▼
Unique Ownership
```

In contrast:

```text
make_shared
    │
    ▼
shared_ptr
    │
    ▼
Shared Ownership
```

If Shared Ownership is not actually required, using `unique_ptr` or even a regular Object generally results in a simpler design.

---

# Choosing Between a Regular Object, `unique_ptr`, and `shared_ptr`

A suitable decision-making model can be viewed as follows:

```text
Does the Object need to be Dynamic?
        │
        ├── No
        │    └── Regular Object
        │
        └── Yes
             │
             ▼
      Are there multiple Owners?
             │
             ├── No
             │    └── unique_ptr / make_unique
             │
             └── Yes
                  └── shared_ptr / make_shared
```

This decision-making model is better than creating every Object with `shared_ptr`.

---

# The Common Mistake of Using `shared_ptr` for Everything

This code is not necessarily a good design:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();
auto c = std::make_shared<C>();
```

The fact that `shared_ptr` automatically manages memory is not, by itself, sufficient reason to use it.

If Object `B` is owned only by `A`, this design may be more appropriate:

```cpp
class A
{
private:
    std::unique_ptr<B> b_;
};
```

And if `B` is inherently part of `A`, an even better design may be:

```cpp
class A
{
private:
    B b_;
};
```

Therefore, Smart Pointers should be selected according to the Ownership Model, not merely to eliminate `delete`.

---

# Avoiding Multiple Control Blocks

One of the most dangerous mistakes when working with `shared_ptr` is creating multiple Control Blocks for the same Object.

For example:

```cpp
auto p1 = std::make_shared<User>();

User* raw = p1.get();

std::shared_ptr<User> p2(raw);
```

We now have two independent `shared_ptr` instances that may each believe they own the Object:

```text
Control Block #1 ──► User
Control Block #2 ──► User
```

This design can ultimately result in a Double Delete.

Therefore, we should not place a Raw Pointer obtained from an Object managed by `shared_ptr` into another `shared_ptr`.

---

# Proper Object Management with `enable_shared_from_this`

If an Object needs to create a `shared_ptr` referring to itself, the appropriate design is to use:

```cpp
std::enable_shared_from_this<T>
```

Example:

```cpp
class Connection
    : public std::enable_shared_from_this<Connection>
{
public:
    std::shared_ptr<Connection> self()
    {
        return shared_from_this();
    }
};
```

Create it with:

```cpp
auto connection =
    std::make_shared<Connection>();
```

Then:

```cpp
auto self =
    connection->self();
```

This causes `self` to share the existing Control Block rather than creating a new one.

---

# Important Note About Thread Safety

A common misunderstanding is that Thread Safety of certain `shared_ptr` operations means that the managed Object itself is Thread-Safe.

These are two completely different concepts.

For example:

```cpp
auto object = std::make_shared<Counter>();
```

Multiple Threads may be able to have independent copies of the `shared_ptr`.

However, this does not mean that:

```cpp
object->increment();
```

is necessarily Thread-Safe when called concurrently from multiple Threads.

In short:

```text
shared_ptr ownership safety
        ≠
managed object thread safety
```

If the Object itself has shared mutable State, appropriate Synchronization must be considered for the Object.

---

# Using `shared_ptr` in Containers

A common use of `make_shared` is creating a collection of Objects with Shared Ownership.

For example:

```cpp
std::vector<std::shared_ptr<User>> users;

users.push_back(
    std::make_shared<User>("Ali", 30)
);

users.push_back(
    std::make_shared<User>("Sara", 25)
);
```

Here, the Container owns the `shared_ptr` instances and therefore maintains the corresponding Owners of the Objects.

When an Object's Pointer is removed from the Container, if no other Owner exists, the Object is destroyed as well.

---

# An Important Note About `use_count`

Suppose we have:

```cpp
auto p = std::make_shared<User>();

auto p2 = p;
auto p3 = p;
```

We can expect:

```cpp
p.use_count() == 3
```

However, this observation should not be used for sensitive decisions in Concurrent programs.

For example, this design is fragile:

```cpp
if (p.use_count() == 1)
{
    // Assume that this is the only owner
}
```

Another Thread could immediately create a Copy of `p`.

Therefore, `use_count` is primarily an observation and Debugging tool rather than an Ownership design mechanism.

---

# Difference Between Object Lifetime and Control Block Lifetime

This concept is very important for professionally understanding `shared_ptr`.

There are two different Lifetimes:

```text
Object Lifetime
        │
        ▼
As long as shared ownership remains

Control Block Lifetime
        │
        ▼
Until both shared and weak ownership have ended
```

Simply:

```text
Last shared_ptr
       │
       ▼
Object destroyed
       │
       │
       ▼
Control Block may still remain
       │
       ▼
Last weak_ptr
       │
       ▼
Control Block destroyed
```

This distinction is exactly what makes the `make_shared` trade-off involving large Objects and long-lived `weak_ptr` instances important.

---

# `make_shared` vs. `new` from a Design Perspective

The expression:

```cpp
new User();
```

is responsible only for creating an Object in Dynamic Storage.

However:

```cpp
std::make_shared<User>();
```

introduces both the concept of Object creation and Shared Ownership into the design.

Therefore, these are not merely two different Syntax forms for the same operation.

We can think of them as:

```text
new
 └── Create Dynamic Object

make_shared
 └── Create Dynamic Object + Shared Ownership
```

---

# Difference Between `shared_ptr(new T)` and `make_shared`

The main comparison can be summarized as follows:

| Topic                                | `shared_ptr(new T(...))`          | `make_shared<T>(...)`           |
| ------------------------------------ | --------------------------------- | ------------------------------- |
| Shared Ownership                     | Yes                               | Yes                             |
| Automatic Lifetime management        | Yes                               | Yes                             |
| Typical Allocation                   | At least two                      | Usually one                     |
| Custom Deleter                       | Yes                               | No                              |
| Non-public Constructor               | May work in an accessible Context | Limited by `make_shared` access |
| Class-Specific `operator new`        | Uses the behavior of `new T`      | Different Allocation behavior   |
| Large Object + long-lived `weak_ptr` | Potential advantage               | Memory may be retained longer   |
| Readability                          | Lower                             | Higher                          |
| Common Modern C++ approach           | Usually no                        | Usually yes                     |

These differences are directly mentioned in Standard Library documentation as trade-offs of `make_shared`. ([Cppreference][1])

---

# When Is `make_shared` a Suitable Choice?

In the usual case, if Shared Ownership is genuinely required and there are no special constraints, `make_shared` is a natural way to create the Object.

Suitable scenarios include:

* Creating regular Objects with Shared Ownership.
* Creating small and medium-sized Objects.
* Creating a large number of Objects where reducing Allocations is beneficial.
* Using `enable_shared_from_this`.
* Creating Polymorphic Objects.
* Creating Objects that do not require a Custom Deleter.

---

# When Is `make_shared` Not a Suitable Choice?

Using `make_shared` may not be appropriate when:

* A Custom Deleter is required.
* The selected Constructor is not accessible to `make_shared`.
* Class-Specific Allocation behavior is important.
* The Object is very large and `weak_ptr` instances remain alive for a long time.
* A specific Allocator is required.
* Shared Ownership is not actually necessary.

When a specific Allocator is required, `allocate_shared` may be a more appropriate option. ([Cppreference][4])

---

# Important Considerations for Large Objects

Suppose an Object contains a very large Buffer:

```cpp
struct LargeObject
{
    std::array<std::byte, 100 * 1024 * 1024> buffer;
};
```

If such an Object is created with `make_shared`:

```cpp
auto object =
    std::make_shared<LargeObject>();
```

and then a `weak_ptr` remains alive for a long time:

```cpp
std::weak_ptr<LargeObject> observer = object;

object.reset();
```

the Object itself is destroyed, but the shared Allocation containing the Control Block may remain until the Lifetime of the last `weak_ptr` ends.

In such circumstances, separating the Object and Control Block Allocations can be advantageous.

This is one of the few situations in which directly using:

```cpp
std::shared_ptr<LargeObject>(
    new LargeObject()
);
```

may make more sense from a Memory Lifetime perspective. ([Cppreference][1])

---

# Checking Feature-Test Macros

Feature-Test Macros can be used to check capabilities related to `make_shared`.

For Array support in `make_shared`:

```cpp
__cpp_lib_shared_ptr_arrays
```

This capability is identified in C++20 by the following Feature-Test Macro value:

```cpp
201707L
```

For `make_shared_for_overwrite`, the following Macro exists:

```cpp
__cpp_lib_smart_ptr_for_overwrite
```

whose standard C++20 value is:

```cpp
202002L
```

([Cppreference][1])

---

# Feature Availability Across C++ Versions

| Feature                         | Standard |
| ------------------------------- | -------- |
| `std::make_shared`              | C++11    |
| `std::allocate_shared`          | C++11    |
| `std::make_unique`              | C++14    |
| `make_shared` Arrays            | C++20    |
| `make_shared_for_overwrite`     | C++20    |
| `allocate_shared_for_overwrite` | C++20    |
| `constexpr make_shared`         | C++26    |
| `constexpr allocate_shared`     | C++26    |

Information about the Standard Versions and Features above can be found in the Standard Library documentation. ([Cppreference][4])

---

# A Complete and Realistic Example

In this example, we have a simple User management system:

```cpp
#include <iostream>
#include <memory>
#include <string>

class User
{
public:
    User(std::string name, int age)
        : name_(std::move(name)),
          age_(age)
    {
        std::cout << "User created\n";
    }

    ~User()
    {
        std::cout << "User destroyed\n";
    }

    void print() const
    {
        std::cout << name_
                  << " - "
                  << age_
                  << '\n';
    }

private:
    std::string name_;
    int age_;
};

int main()
{
    auto user =
        std::make_shared<User>("Ali", 30);

    user->print();

    {
        auto another_owner = user;

        another_owner->print();

        std::cout
            << "Owners: "
            << user.use_count()
            << '\n';
    }

    std::cout
        << "Owners: "
        << user.use_count()
        << '\n';
}
```

In this program, there is initially one Owner.

Then, another Owner is created inside the inner Scope.

When `another_owner` leaves the Scope, that Owner is destroyed, but the Object is still managed by `user`.

Finally, when `main` ends, the last `shared_ptr` is destroyed and the `User` is also destroyed.

---

# An Example for Observing Lifetime

To observe Lifetime more clearly, we can use a Constructor and Destructor:

```cpp
#include <iostream>
#include <memory>

class Resource
{
public:
    Resource()
    {
        std::cout << "Resource constructed\n";
    }

    ~Resource()
    {
        std::cout << "Resource destroyed\n";
    }
};

int main()
{
    auto p1 = std::make_shared<Resource>();

    {
        auto p2 = p1;

        std::cout
            << p1.use_count()
            << '\n';
    }

    std::cout
        << p1.use_count()
        << '\n';
}
```

As long as `p1` exists, the Resource remains alive.

After `p2` leaves the Scope, the number of Owners decreases, but the Resource is still managed by `p1`.

At the end of `main`, the last Owner is destroyed and the Destructor is executed.

---

# Recommended Pattern for Modern C++

If Shared Ownership is genuinely required, the usual pattern is:

```cpp
auto object =
    std::make_shared<MyType>(
        constructor_arg1,
        constructor_arg2
    );
```

If we have only Unique Ownership:

```cpp
auto object =
    std::make_unique<MyType>(
        constructor_arg1,
        constructor_arg2
    );
```

And if Dynamic Allocation is not required at all:

```cpp
MyType object(
    constructor_arg1,
    constructor_arg2
);
```

These three cases should not be selected based only on Syntax; the Ownership and Lifetime model should be determined first.

---

# Common Mistakes When Using `make_shared`

## Creating a Second `shared_ptr` from `get`

This is dangerous:

```cpp
auto p1 = std::make_shared<User>();

auto p2 =
    std::shared_ptr<User>(p1.get());
```

These two Pointers do not necessarily share the same Control Block and can result in a Double Delete.

---

## Using `shared_ptr` Without Shared Ownership

This design:

```cpp
auto user = std::make_shared<User>();
```

is meaningful only when multiple parts of the program genuinely need to own the User.

If there is only one Owner, `unique_ptr` generally provides a clearer Ownership model.

---

## Creating Circular Ownership

This design is dangerous:

```cpp
class Parent;

class Child
{
public:
    std::shared_ptr<Parent> parent;
};
```

If the Parent also owns the Child through a `shared_ptr`, a Cycle can occur.

For a non-owning relationship, `weak_ptr` is generally more appropriate.

---

## Keeping a Raw Pointer Obtained from `get` for Too Long

This code:

```cpp
auto user = std::make_shared<User>();

User* raw = user.get();

user.reset();
```

leaves `raw` pointing to an Object whose Lifetime has already ended.

Therefore, a Raw Pointer obtained from `get()` must not be used beyond the Lifetime of its Owner.

---

## Using `use_count()` as Synchronization

This design is not appropriate:

```cpp
if (object.use_count() == 1)
{
    // Assume exclusive ownership
}
```

In a Concurrent environment, the Ownership state can change.

---

# `make_shared` Selection Checklist

Before using `make_shared`, consider these questions:

* Does the Object really need to be Dynamic?
* Are there multiple Owners?
* Is Shared Ownership actually part of the Design?
* Is a Custom Deleter unnecessary?
* Is the Constructor accessible to `make_shared`?
* Is the Object not extremely large?
* Are there no long-lived `weak_ptr` instances?
* Is Class-Specific Allocation behavior unimportant?
* Is a custom Allocator unnecessary?
* Will no Reference Cycle be created?

If the answers to these questions are consistent with your Design, `make_shared` is generally a suitable choice for creating a `shared_ptr`.

---

# Final Summary

`std::make_shared` is a Function Template in the Standard Library used to create an Object managed by `std::shared_ptr`.

Its typical usage is:

```cpp
auto ptr = std::make_shared<MyType>(args...);
```

Its most important advantage over:

```cpp
std::shared_ptr<MyType>(
    new MyType(args...)
);
```

is that common implementations place the Object and Control Block in a single Allocation, whereas constructing a `shared_ptr` from a Raw Pointer normally requires at least two Allocations. ([Cppreference][1])

This can reduce Allocation overhead and improve memory Locality.

On the other hand, `make_shared` also has limitations:

* It does not support a Custom Deleter.
* It has limitations with inaccessible Constructors.
* It can behave differently with a Class-Specific `operator new`.
* For very large Objects combined with long-lived `weak_ptr` instances, it can cause a larger Allocation to remain alive for longer.
* If a custom Allocator is required, `allocate_shared` is a more suitable option. ([Cppreference][4])

Important related capabilities include:

```text
std::make_shared
        │
        ├── std::shared_ptr
        │
        ├── std::weak_ptr
        │
        ├── std::enable_shared_from_this
        │
        ├── std::allocate_shared
        │
        ├── std::make_shared_for_overwrite
        │
        └── Array support
```

Ultimately, the most important principle is:

> Design ownership first, then choose the Smart Pointer.

If an Object has one Owner, `unique_ptr` or a regular Object is generally more appropriate.

If there are multiple genuine Owners, `shared_ptr` makes sense, and in such cases `make_shared` is generally the preferred way to create it.

If a relationship is observational and non-owning, `weak_ptr` may be the more appropriate tool.

Therefore, the real value of `make_shared` is not merely shortening this code:

```cpp
auto ptr = std::make_shared<MyType>();
```

It is the combination of **Shared Ownership, automatic Lifetime management, improved Exception Safety, and usually more efficient Allocation**. ([Cppreference][1])

[1]: https://en.cppreference.com/%20%20cpp/memory/shared_ptr/make_shared?utm_source=chatgpt.com "std::make_shared, std::make_shared_for_overwrite - cppreference.com"
[2]: https://en.cppreference.com/cpp/memory/shared_ptr/get?utm_source=chatgpt.com "std::shared_ptr<T>::get - cppreference.com"
[3]: https://en.cppreference.com/cpp/feature_test?utm_source=chatgpt.com "Feature testing (since C++20) - cppreference.com"
[4]: https://en.cppreference.com/cpp/memory/shared_ptr/allocate_shared?utm_source=chatgpt.com "std::allocate_shared, std::allocate_shared_for_overwrite - cppreference.com"

---

## 🤝 Contributions

<div align="center">

| GitHub                                      | LinkedIn                                                           | Email                                           | Site                           | Telegram                               |
| ------------------------------------------- | ------------------------------------------------------------------ | ----------------------------------------------- | ------------------------------ | -------------------------------------- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>
