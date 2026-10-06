<div align="center">

[🇺🇸 English](./noexcept.md) | [🇮🇷 فارسی](../../fa/cpp11/noexcept.md)

</div>

---

# Comprehensive Guide to `noexcept` in C++

The `noexcept` feature in C++ is a mechanism for declaring that a function is not expected to throw an exception. Introduced in C++11, it is now an important part of API design, exception handling, Move Semantics, and Generic Programming.

An important point is that `noexcept` is not merely a “promise” to the reader or a hint to the compiler. In C++, it is part of the function's semantics. If an exception exits a function whose `noexcept` specification does not allow such an exception to escape, the program enters termination.

---

## Introduction to `noexcept`

The `noexcept` specification can be placed on a function declaration or definition:

```cpp
void process() noexcept;
```

Here, the function `process` declares that an exception should not escape from it.

An expression can also be supplied to `noexcept`:

```cpp
void process() noexcept(noexcept(do_work()));
```

In this case, the function's `noexcept` status depends on the result of the inner expression.

Conceptually, `noexcept` has two primary uses:

* Declaring the exception-safety contract of a function
* Querying whether an expression may throw an exception

These two uses are related, but they should be distinguished from each other.

---

## Why `noexcept` Exists

Before C++11, C++ supported older exception specifications such as `throw()` and `throw(Type)`. These mechanisms had several design and usability problems and were eventually deprecated.

The `noexcept` mechanism was designed to provide a simpler and more reliable model.

Its main benefits include:

* Explicitly specifying an exception contract
* Enabling better decisions in Generic Programming
* Allowing certain compiler optimizations
* Allowing standard containers to prefer Move Construction over Copy Construction in many cases
* Providing type-level information about exception behavior since C++17
* Simplifying reasoning about APIs and ownership
* Preventing exceptions from crossing boundaries where doing so is inappropriate

However, `noexcept` should not be used merely to make a program “faster.” The function's contract should be designed correctly first.

---

## Understanding Exception Specifications

The `noexcept` specification is part of a function's exception specification.

For example:

```cpp
void f() noexcept;
void g() noexcept(false);
```

The function `f` has a non-throwing exception specification, while `g` explicitly allows exceptions to propagate.

In practice, `noexcept(false)` represents a potentially-throwing function, much like a function with no `noexcept` specification:

```cpp
void f();
void g() noexcept(false);
```

Both functions may propagate exceptions to their callers.

Therefore, it is incorrect to interpret `noexcept(false)` as meaning that the function will definitely throw an exception. It only means that the function does not have a non-throwing contract.

---

## How `noexcept` Works at Runtime

The most important characteristic of `noexcept` is that an exception cannot propagate through a non-throwing function.

For example:

```cpp
#include <stdexcept>

void process() noexcept {
    throw std::runtime_error("failure");
}
```

Here, the exception attempts to escape from `process`, so the runtime calls `std::terminate`.

This does not behave as follows:

```cpp
try {
    process();
} catch (...) {
    // This is not reached
}
```

The exception cannot cross the `noexcept` boundary.

An important distinction is that saying “a `noexcept` function can never create an exception” is inaccurate. An exception may be created inside the function; the requirement is that it must not escape from the function.

For example:

```cpp
void process() noexcept {
    try {
        perform_operation();
    } catch (...) {
        recover_from_error();
    }
}
```

An exception may occur inside `perform_operation`, but it is handled before `process` returns.

---

## `noexcept` and `std::terminate`

When an exception escapes a function declared `noexcept`, `std::terminate` is called.

For example:

```cpp
#include <exception>
#include <stdexcept>

void process() noexcept {
    throw std::runtime_error("unexpected failure");
}
```

In this situation, `std::terminate` is invoked.

The termination mechanism can be customized with `std::set_terminate`:

```cpp
#include <exception>
#include <cstdlib>

void terminate_handler() noexcept {
    std::abort();
}

int main() {
    std::set_terminate(terminate_handler);
}
```

However, changing the terminate handler is generally not an application-level error-handling strategy. The primary purpose of `noexcept` is to make failures that violate the function's contract explicit as serious failures.

---

## `noexcept` vs. `try`/`catch`

`noexcept` is not a replacement for `try`/`catch`.

These mechanisms serve different purposes:

* `try`/`catch` is used to handle exceptions.
* `noexcept` specifies the boundary beyond which exceptions cannot propagate.

For example:

```cpp
void operation() noexcept {
    try {
        risky_operation();
    } catch (...) {
        handle_failure();
    }
}
```

The function remains non-throwing because the exception is handled before the function exits.

In contrast:

```cpp
void operation() noexcept {
    risky_operation();
}
```

If `risky_operation` throws and the exception is not handled, termination occurs.

---

## Introduction to the `noexcept` Operator

In addition to being an exception specification, `noexcept` can also be used as an operator.

For example:

```cpp
static_assert(noexcept(42));
```

The expression

```cpp
noexcept(expression)
```

produces a constant expression of type `bool`.

If the compiler determines that the expression cannot propagate an exception to its caller, the result is `true`; otherwise, the result is `false`.

For example:

```cpp
void safe_operation() noexcept;
void risky_operation();

static_assert(noexcept(safe_operation()));
static_assert(!noexcept(risky_operation()));
```

This operator is particularly important in Generic Programming.

---

## Checking `noexcept` for Expressions

The result of `noexcept(...)` applies to the expression itself, not merely to the name of a function.

For example:

```cpp
void f() noexcept;

struct Widget {
    void run() noexcept;
};

Widget widget;

static_assert(noexcept(f()));
static_assert(noexcept(widget.run()));
```

Constructors, destructors, conversions, and operators can also be examined in this way:

```cpp
struct Widget {
    Widget() noexcept = default;
    ~Widget() noexcept = default;
};

static_assert(noexcept(Widget{}));
```

This capability makes it possible to write templates whose behavior depends on the exception properties of the operations they use.

---

## Conditional `noexcept`

One of the most important modern C++ patterns is conditional `noexcept`.

For example:

```cpp
template <typename T>
void swap_values(T& a, T& b) noexcept(noexcept(T(std::move(a)))) {
    T temp(std::move(a));
    a = std::move(b);
    b = std::move(temp);
}
```

Here, the `noexcept` status of the function depends on an internal operation.

This pattern is especially important in generic code because a template should not assume that every type is non-throwing.

---

## `noexcept` and Move Constructors

One of the most important practical uses of `noexcept` is with Move Semantics.

Suppose a type has a Move Constructor:

```cpp
class Buffer {
public:
    Buffer(Buffer&& other) noexcept;
};
```

In this situation, containers such as `std::vector` can use move operations more effectively in many of their operations.

The reason is related to exception guarantees. When `std::vector` grows, it may need to move existing elements to new storage. If moving an element can throw, simply moving the elements may destroy the original state and make restoring the previous state difficult.

For this reason, the standard library prefers a non-throwing move operation when appropriate.

A simple example:

```cpp
#include <vector>

class Widget {
public:
    Widget() = default;

    Widget(const Widget&) = default;

    Widget(Widget&&) noexcept = default;
};

int main() {
    std::vector<Widget> widgets;

    widgets.emplace_back();
    widgets.emplace_back();
}
```

Here, the `noexcept` status of the Move Constructor is an important property for how the type interacts with the standard library.

---

## Copy vs. Move When a Vector Grows

If a type has both a Copy Constructor and a Move Constructor, the choice between them can depend on whether the Move Constructor is `noexcept`.

For example:

```cpp
class Widget {
public:
    Widget(const Widget&);
    Widget(Widget&&) noexcept;
};
```

This is generally desirable for interaction with containers.

But if the Move Constructor is written as:

```cpp
class Widget {
public:
    Widget(const Widget&);
    Widget(Widget&&);
};
```

the Move Constructor does not guarantee that it is non-throwing. In such cases, the standard library may use the Copy Constructor in situations where doing so is necessary to preserve the appropriate exception guarantee.

This is one of the important reasons why Move Constructors that are genuinely non-throwing should usually be declared `noexcept`.

---

## Introduction to `std::move_if_noexcept`

The standard library provides `std::move_if_noexcept` for this type of scenario.

It is defined in `<utility>`:

```cpp
#include <utility>

auto value = std::move_if_noexcept(object);
```

Under the appropriate conditions, this utility produces an rvalue suitable for moving and, when moving may throw while copying is available, can prefer copying instead.

This mechanism is particularly useful in generic implementations and containers.

---

## Implicit `noexcept` in Special Member Functions

Some special member functions can have implicitly generated exception specifications.

For example:

```cpp
struct Point {
    int x;
    int y;
};
```

Here, the compiler can determine the exception specifications of operations such as the default constructor, Copy Constructor, Move Constructor, and destructor based on their members and base classes.

This behavior is important for designing composable types.

If all components of a type are non-throwing, the corresponding implicitly generated operation can also be non-throwing.

---

## `noexcept` in Composite Types

In Generic Programming, it is better to inspect the properties of a type than to assume them.

For example:

```cpp
template <typename T>
void process(T& value) noexcept(noexcept(value.reset())) {
    value.reset();
}
```

If `reset()` is non-throwing for the target type, `process` is also non-throwing.

This pattern keeps a wrapper's exception specification aligned with the operation it invokes.

---

## Type Traits

The standard library provides type traits for inspecting properties related to exception behavior.

For example:

```cpp
#include <type_traits>

static_assert(std::is_nothrow_move_constructible_v<int>);
```

Other related traits include:

```cpp
std::is_nothrow_constructible
std::is_nothrow_assignable
std::is_nothrow_copy_constructible
std::is_nothrow_move_constructible
std::is_nothrow_copy_assignable
std::is_nothrow_move_assignable
std::is_nothrow_swappable
```

These traits are part of the standard library and complement the `noexcept` operator.

---

## `noexcept` in Generic Programming

One of the best uses of `noexcept` in template code is propagating the exception guarantee of an internal operation to a public API.

For example:

```cpp
template <typename T>
void reset(T& object) noexcept(noexcept(object.reset())) {
    object.reset();
}
```

This design avoids hard-coding the exception behavior of the function.

The following approach is potentially incorrect:

```cpp
template <typename T>
void reset(T& object) noexcept {
    object.reset();
}
```

The second version may impose an incorrect contract.

If `object.reset()` can throw, the second version will cause termination instead of allowing the exception to propagate.

---

## `noexcept(true)` vs. `noexcept`

These two forms are semantically equivalent:

```cpp
void f() noexcept;
void g() noexcept(true);
```

Both functions are non-throwing.

However, the shorter form is generally more readable:

```cpp
void f() noexcept;
```

The explicit `noexcept(true)` form can be useful when the exception specification is being expressed generically or conditionally:

```cpp
template <typename T>
void process(T& value) noexcept(noexcept(value.process()));
```

---

## `noexcept(false)` vs. Omitting `noexcept`

These two forms represent a potentially-throwing function:

```cpp
void f();
void g() noexcept(false);
```

A function without `noexcept` is generally the more natural and readable form.

Therefore, `noexcept(false)` is usually useful only when it is part of a conditional or generic expression.

---

## Limitations of `noexcept`

`noexcept` does not guarantee that no failure or error can occur inside a function.

For example:

```cpp
void process() noexcept {
    allocate_resource();
}
```

Even though the function is `noexcept`, `allocate_resource` may fail.

If that failure is represented by an exception and the exception is not handled, termination occurs.

Therefore, `noexcept` does not mean:

* The function always succeeds.
* The function cannot encounter errors.
* The function performs no potentially dangerous operations.
* The compiler guarantees that the function is faster.
* No exception can ever be created anywhere inside the function.

`noexcept` only specifies the contract governing exceptions escaping from the function.

---

## `noexcept` and Optimization

One motivation for non-throwing exception specifications is the possibility of optimization, but `noexcept` should not be treated as an optimization directive.

A compiler may use `noexcept` information to generate better code, but the C++ standard does not guarantee that adding `noexcept` will make a function faster.

The primary decision should therefore be based on correctness and the API contract.

A good rule is:

> First determine whether the function should genuinely be non-throwing; then apply `noexcept`.

Not:

> Add `noexcept` so that the compiler can make the code faster.

---

## Handling Operations That Truly Do Not Throw

Functions that naturally perform non-throwing operations are good candidates for `noexcept`.

A common example is `swap` for suitable types:

```cpp
class Buffer {
public:
    void swap(Buffer& other) noexcept {
        // Swap internal resources
    }
};
```

Move operations, destructors, and operations involving pointers or primitive types can also often be non-throwing.

However, the actual implementation must be examined.

---

## `noexcept` and Destructors

Destructors have a special role.

In modern C++, a destructor should not allow an exception to escape.

For example:

```cpp
class Resource {
public:
    ~Resource() noexcept {
        release_resource();
    }
};
```

If a destructor throws during stack unwinding, the situation can become extremely dangerous, especially when another exception is already being unwound.

In such circumstances, termination occurs.

For this reason, destructors should either handle exceptions internally or be designed so that their operations are inherently non-throwing.

---

## Implicit Destructor `noexcept`

If a destructor is implicitly generated, its exception specification is determined based on its subobjects.

For example:

```cpp
struct Resource {
    Resource() = default;
    ~Resource() = default;
};
```

If the destructors of the type's members are non-throwing, the implicitly generated destructor can also be non-throwing.

This is one of the benefits of composition in modern C++ design.

---

## `noexcept` and Virtual Overrides

In inheritance, the exception specification of an overriding function must be compatible with that of the base function.

For example:

```cpp
struct Base {
    virtual void process() noexcept;
};

struct Derived : Base {
    void process() noexcept override;
};
```

The following design is invalid:

```cpp
struct Base {
    virtual void process() noexcept;
};

struct Derived : Base {
    void process() override;
};
```

The derived function cannot weaken the non-throwing contract of the base function.

In other words, if a base function is `noexcept`, the override must provide a compatible non-throwing contract.

---

## Throwing vs. Non-Throwing Virtual Functions

If the base function is potentially throwing, the override may be non-throwing:

```cpp
struct Base {
    virtual void process();
};

struct Derived : Base {
    void process() noexcept override;
};
```

This is valid because the derived function imposes a stronger restriction on exception behavior.

The reverse is not allowed:

```cpp
struct Base {
    virtual void process() noexcept;
};

struct Derived : Base {
    void process() override;
};
```

A derived class cannot weaken the base class's contract.

---

## `noexcept` Changes in C++17

One important change in C++17 is that exception specifications became part of the function type.

For example:

```cpp
using SafeFunction = void() noexcept;
using RegularFunction = void();
```

These are different function types.

This distinction also matters for function pointers:

```cpp
void safe_function() noexcept;

void (*ptr)() noexcept = safe_function;
```

In C++17 and later, this property is part of the type system.

---

## Function Pointer Conversions

A pointer to a non-throwing function can, under the appropriate rules, be converted to a pointer to a potentially-throwing function:

```cpp
void safe_function() noexcept;

void (*regular_ptr)() = safe_function;
```

The reverse conversion is not allowed:

```cpp
void regular_function();

void (*safe_ptr)() noexcept = regular_function;
```

This is logical because a caller cannot safely treat a potentially-throwing function as a function that guarantees that no exception will escape.

---

## `noexcept` in Lambdas

Lambdas can also be declared `noexcept`:

```cpp
auto operation = []() noexcept {
    // Non-throwing operation
};
```

The exception specification can also be conditional:

```cpp
auto operation = [](auto& value) noexcept(noexcept(value.process())) {
    value.process();
};
```

This capability is particularly useful for Generic Lambdas.

---

## `noexcept` in Function Objects

A function object's `operator()` can also be non-throwing:

```cpp
struct Processor {
    void operator()() noexcept {
        // Process data
    }
};
```

This property can matter when the object is used with generic algorithms and other utilities.

---

## `noexcept` in Move Assignment

`noexcept` is not limited to Move Constructors.

Move Assignment can also be non-throwing:

```cpp
class Buffer {
public:
    Buffer& operator=(Buffer&& other) noexcept {
        // Transfer ownership
        return *this;
    }
};
```

For resource-owning types, non-throwing move operations are generally valuable properties.

---

## `noexcept` in `swap`

For swappable types, making `swap` non-throwing can also be important.

A common pattern is:

```cpp
class Widget {
public:
    void swap(Widget& other) noexcept {
        // Swap state
    }
};

void swap(Widget& a, Widget& b) noexcept {
    a.swap(b);
}
```

In Generic Programming, the exception behavior of swap can also be checked:

```cpp
static_assert(std::is_nothrow_swappable_v<Widget>);
```

This property can affect the design and implementation of generic algorithms and containers.

---

## `noexcept` with Concepts and `requires`

In C++20, `noexcept` can be combined with Concepts and `requires`.

For example:

```cpp
template <typename T>
concept NothrowResettable =
    requires(T& value) {
        { value.reset() } noexcept;
    };

template <NothrowResettable T>
void reset(T& value) noexcept {
    value.reset();
}
```

Here, the constraint specifies that the operation must exist and must be non-throwing.

This approach can be clearer than manually maintaining multiple traits and works naturally with the Concepts model introduced in C++20.

---

## `noexcept` vs. Contracts

`noexcept` should not be confused with general function contracts.

For example:

```cpp
void transfer(Resource& resource) noexcept;
```

This declaration describes exception behavior, not whether `resource` is valid or whether the function's preconditions have been satisfied.

Conceptually:

* `noexcept` concerns exception propagation.
* A precondition describes the conditions required for a valid call.
* A postcondition describes the expected state after execution.

These concepts can coexist in an API, but they are not the same thing.

---

## `noexcept` in API Design

In library design, making a function `noexcept` is a contractual decision.

A function is a good candidate for `noexcept` when:

* It genuinely should not allow an exception to escape.
* Its implementation can maintain that contract.
* Callers benefit from its non-throwing behavior.
* State changes or resource management can be performed reliably.
* Non-throwing behavior is part of the natural semantics of the operation.

For example:

```cpp
class Handle {
public:
    Handle(Handle&& other) noexcept;
    Handle& operator=(Handle&& other) noexcept;
    ~Handle() noexcept;
};
```

This kind of API is common and useful for resource-owning types.

---

## When Not to Use `noexcept`

Do not add `noexcept` merely to make an API appear more robust or faster.

For example, if a function can genuinely report failure through an exception:

```cpp
void load_file(const std::string& path);
```

adding `noexcept` without handling exceptions internally can create a dangerous contract:

```cpp
void load_file(const std::string& path) noexcept {
    // Potentially throwing operations
}
```

If allocation, parsing, or I/O causes an exception and it is not handled, the result will be termination.

---

## Exception Safety in Design

When deciding whether to use `noexcept`, exception-safety guarantees should be considered.

Three common concepts are:

* **No-throw guarantee**: The operation does not allow an exception to escape.
* **Strong guarantee**: If the operation fails, the observable state remains unchanged.
* **Basic guarantee**: Object invariants are preserved and no resource leaks occur.

`noexcept` directly expresses the no-throw guarantee, while the strong and basic guarantees are broader concepts.

For example:

```cpp
void commit() noexcept;
```

This declaration can express that `commit` does not allow exceptions to escape, but by itself it does not describe the state transition performed by the operation.

---

## `noexcept` and RAII

In RAII design, destructors should release resources without allowing exceptions to escape.

For example:

```cpp
class File {
public:
    ~File() noexcept {
        close_file();
    }

private:
    void close_file() noexcept {
        // Release operating-system resource
    }
};
```

If cleanup can genuinely fail, the design should determine how that failure is handled.

In many cases, reporting failure from a destructor is inappropriate, so explicit operations such as `close` or `commit` should be used when the caller needs to handle failure.

---

## The Explicit Commit Pattern

In resource management, a useful pattern is to perform important operations before destruction.

For example:

```cpp
class Transaction {
public:
    void commit();
    ~Transaction() noexcept;
};
```

Here, `commit` may throw an exception, while the destructor only performs cleanup.

This pattern is generally preferable to forcing a destructor to report business-level failures.

---

## `noexcept` and Allocation

Having `noexcept` does not automatically prevent allocation exceptions.

For example:

```cpp
void process() noexcept {
    std::vector<int> values(1000000);
}
```

If allocation throws `std::bad_alloc` and the exception is not handled, the `noexcept` boundary means termination occurs.

Therefore, `noexcept` does not make allocation operations non-throwing.

---

## `std::nothrow` Allocation

The standard library provides another form of allocation:

```cpp
int* value = new (std::nothrow) int(42);
```

Here, allocation failure is reported through `nullptr` instead of `std::bad_alloc`.

However, this mechanism is independent of `noexcept`.

More precisely, `noexcept` specifies the exception behavior of a function, while `std::nothrow` changes how a particular allocation operation reports failure.

---

## `noexcept` vs. `std::nothrow`

These concepts are sometimes confused.

`noexcept`:

```cpp
void process() noexcept;
```

specifies the exception behavior of a function.

`std::nothrow`:

```cpp
new (std::nothrow) Widget;
```

changes how allocation failure is reported.

They are therefore not alternatives to one another.

---

## `noexcept` and Constructors

Constructors can also be declared `noexcept`:

```cpp
class Point {
public:
    Point(int x, int y) noexcept
        : x_(x), y_(y) {}

private:
    int x_;
    int y_;
};
```

However, the initializer operations must also be compatible with this contract.

If construction of a member throws, the exception cannot escape from the constructor and termination occurs.

---

## `noexcept` and Conversion Operators

Conversion operators can also be non-throwing:

```cpp
class Identifier {
public:
    operator int() const noexcept {
        return value_;
    }

private:
    int value_;
};
```

This property can be useful in Generic Programming and overload resolution.

---

## `noexcept` and Defaulted Functions

Defaulted special member functions can explicitly specify `noexcept`:

```cpp
struct Widget {
    Widget(Widget&&) noexcept = default;
};
```

This makes the API contract explicit.

However, if the members or base classes do not support such an operation consistently, the compiler may be unable to generate the requested definition.

For this reason, the properties of members and base classes should be considered before explicitly specifying `noexcept`.

---

## `noexcept` and Deleted Functions

`noexcept` can also appear alongside deleted declarations:

```cpp
struct Widget {
    Widget(const Widget&) = delete;
    Widget(Widget&&) noexcept = default;
};
```

This is a common pattern for move-only types.

---

## `noexcept` and Function Pointers

Since C++17, the distinction between throwing and non-throwing function types is visible in function pointers.

For example:

```cpp
using SafeFunction = void (*)() noexcept;
using RegularFunction = void (*)();

void safe_function() noexcept;

SafeFunction safe = safe_function;
RegularFunction regular = safe_function;
```

This distinction can be important when designing callback APIs.

---

## `noexcept` and `std::function`

When using `std::function`, note that the usual `std::function` type does not encode a `noexcept` specification in its general signature.

Therefore, this:

```cpp
std::function<void()> callback;
```

should not be interpreted as meaning that the callback is inherently non-throwing.

If a non-throwing callback contract is important to an API, templates, suitable function pointers, Concepts, or dedicated function types can be used instead.

---

## `noexcept` in Algorithms

Some algorithms and standard library utilities can take advantage of the properties of the types they operate on.

For this reason, having a non-throwing Move Constructor or `swap` can improve how a type works with containers and algorithms.

For example, the following type design is generally desirable:

```cpp
class Widget {
public:
    Widget(Widget&&) noexcept;
    Widget& operator=(Widget&&) noexcept;

    void swap(Widget&) noexcept;
};
```

These properties make the type better suited for use with standard containers and algorithms.

---

## `noexcept` and Value Categories

Consider:

```cpp
noexcept(std::move(value))
```

The important point is that `std::move` itself is typically non-throwing, while the operation performed afterward may not be.

For example:

```cpp
noexcept(std::move(value))
```

is different from:

```cpp
noexcept(T(std::move(value)))
```

The second expression also considers the actual construction operation.

Therefore, when using the `noexcept` operator, the exact expression being evaluated must be considered rather than just a preliminary operation such as `std::move`.

---

## `noexcept` and Overload Resolution

`noexcept` is generally not a direct overload-resolution criterion in the sense that a non-throwing overload is automatically preferred over a throwing one.

For example, simply being non-throwing does not cause the compiler to always select one overload over another.

However, since C++17, exception specifications are part of function types and therefore affect type compatibility and related contexts.

It is important to distinguish between being part of the type and being an overload-selection criterion.

---

## `noexcept` in Templates

The following template design is dangerous:

```cpp
template <typename T>
void process(T& value) noexcept {
    value.process();
}
```

If `T::process` can throw, the wrapper has an incorrect contract.

A better design is:

```cpp
template <typename T>
void process(T& value) noexcept(noexcept(value.process())) {
    value.process();
}
```

Here, the wrapper's behavior depends on the actual operation.

This is one of the most important uses of conditional `noexcept` in library code.

---

## Implementing Conditional `noexcept` in Generic Code

A more complete wrapper can also use `std::forward`:

```cpp
#include <utility>

template <typename T>
void invoke_process(T&& value)
    noexcept(noexcept(std::forward<T>(value).process()))
{
    std::forward<T>(value).process();
}
```

This design preserves two important properties:

* The argument's value category
* The exception specification of the underlying operation

This pattern is common in modern generic utilities.

---

## `noexcept` and Perfect Forwarding

In generic code, the `noexcept` expression should inspect the same expression that will actually be executed.

For example, these are not necessarily equivalent:

```cpp
noexcept(value.process())
```

and:

```cpp
noexcept(std::forward<T>(value).process())
```

If different overloads exist for lvalues and rvalues, the selected operation may be different.

Therefore, in Perfect Forwarding code, the `noexcept` expression should accurately reflect the expression that will actually execute.

---

## Common Mistake: Overusing `noexcept`

One common mistake is assuming that every small function should be `noexcept`.

For example:

```cpp
std::string normalize(std::string value) noexcept;
```

If the implementation may allocate memory, this declaration can be dangerous.

In such cases, an exception may be the most natural way to report failure.

`noexcept` should be chosen based on semantics and the function's contract, not simply on the apparent size or simplicity of the function.

---

## Common Mistake: Forgetting `noexcept` on Move Constructors

Another important mistake is forgetting `noexcept` on Move Constructors that are genuinely non-throwing.

For example:

```cpp
class Buffer {
public:
    Buffer(Buffer&& other);
};
```

If ownership transfer consists only of pointers and metadata and does not involve operations that can throw, the contract should generally be explicit:

```cpp
class Buffer {
public:
    Buffer(Buffer&& other) noexcept;
};
```

This can improve the type's interaction with the standard library.

---

## Common Mistake: Lying with `noexcept`

A more serious mistake is declaring `noexcept` for a function that can actually allow an exception to escape:

```cpp
void process() noexcept {
    operation_that_can_throw();
}
```

This does not “solve” exception handling; it merely changes exception propagation into termination.

Therefore, `noexcept` should never be used to hide exception behavior.

---

## Common Mistake: Throwing from a Destructor

This design is highly dangerous:

```cpp
class Resource {
public:
    ~Resource() noexcept {
        risky_cleanup();
    }
};
```

If `risky_cleanup` throws, termination occurs.

A better design is to make destructor cleanup inherently non-throwing or move reportable failures into explicit operations that the caller can invoke and handle.

---

## Common Mistake: Catching an Exception After `noexcept`

This assumption is incorrect:

```cpp
void process() noexcept {
    throw std::runtime_error("error");
}

int main() {
    try {
        process();
    } catch (...) {
        // Expected recovery
    }
}
```

The exception causes termination before it can reach the caller.

If recovery at the caller level is required, the function should not block exception propagation with `noexcept`, unless it handles the exception internally and reports the failure through another mechanism.

---

## Common Mistake: Using `noexcept` as an Optimization Hint

It is incorrect to assume that:

```cpp
void calculate() noexcept;
```

will necessarily execute faster than:

```cpp
void calculate();
```

The compiler may use non-throwing information for optimization, but `noexcept` is not simply a performance annotation.

The decision should be based on correctness and the contract.

---

## Common Mistake: Ignoring ABI Considerations

In large libraries, changing an exception specification can affect compatibility and function types, especially since C++17.

For example, changing:

```cpp
void process();
```

to:

```cpp
void process() noexcept;
```

may not be merely a cosmetic change.

The exact ABI impact depends on the compiler, platform, ABI implementation, and how the symbols are used. Therefore, binary compatibility should not be assumed without checking the relevant toolchain.

---

## `noexcept` vs. `throw()`

In C++11, `throw()` represented a non-throwing specification, but this syntax was later deprecated.

Legacy code may contain:

```cpp
void process() throw();
```

Modern C++ code should use:

```cpp
void process() noexcept;
```

Dynamic exception specifications such as:

```cpp
void process() throw(std::runtime_error);
```

are no longer part of modern C++ design and should not be used in new code.

---

## `noexcept` vs. Dynamic Exception Specifications

The older model could specify the types of exceptions that were permitted:

```cpp
void process() throw(std::runtime_error);
```

This model is different from `noexcept`.

`noexcept` essentially focuses on the question:

> Can an exception escape from this function?

rather than:

> Which exception types are allowed to escape?

This simplification is one of the important advantages of the modern design.

---

## `noexcept` Across C++ Standard Versions

The `noexcept` feature was introduced in C++11.

In C++14, its general use continued and it became an important tool for expressing exception specifications.

In C++17, exception specifications became part of function types, giving `noexcept` greater importance in the type system.

In C++20, `noexcept` became even more useful alongside Concepts, `requires`-expressions, and modern Generic Programming.

In C++23 and C++26, `noexcept` remains part of the language's standard exception-handling model, and its fundamental syntax has not changed.

---

## `noexcept` and `constexpr`

`constexpr` and `noexcept` are independent properties and can be used together:

```cpp
constexpr int square(int value) noexcept {
    return value * value;
}
```

`constexpr` concerns the possibility of compile-time evaluation, while `noexcept` concerns exception propagation.

Having one does not automatically imply the other.

---

## `noexcept` and `consteval`

The same distinction applies to `consteval`:

```cpp
consteval int square(int value) noexcept {
    return value * value;
}
```

`consteval` requires compile-time evaluation, while `noexcept` specifies exception behavior.

---

## `noexcept` and Coroutines

In C++20, coroutine functions can also be declared `noexcept`.

For example:

```cpp
Task process() noexcept {
    co_await perform_operation();
}
```

However, coroutines require separate consideration of `promise_type`, exception handling, and the coroutine machinery.

`noexcept` does not mean that a coroutine has no possible failure; it means that exceptions must not escape according to the coroutine's specified contract.

In particular, library-level coroutine designs must determine how exceptions are transferred to `promise_type` or another error-reporting mechanism.

---

## `noexcept` and Asynchronous Programming

In asynchronous APIs, a non-throwing operation does not necessarily mean that no failure exists.

For example, an API can be designed as:

```cpp
Future<Result> start_operation() noexcept;
```

and report failure through `Result` or the state associated with the asynchronous operation.

This design can be appropriate when failure is considered a normal result of the operation rather than something that should propagate through exceptions.

However, this choice should be based on the semantics of the API.

---

## `noexcept` and Result-Based Error Handling

In some systems, particularly low-level or latency-sensitive code, failures may be represented through result types:

```cpp
Result process() noexcept;
```

In such a design, `noexcept` can be entirely natural.

However, `noexcept` alone is not a reason to use a Result type. These are separate design decisions:

* Choosing the error model
* Specifying the exception behavior

---

## Appropriate Boundaries for `noexcept`

One good use of `noexcept` is to define boundaries that exceptions must not cross.

Common examples include:

* Destructors
* Move Constructors that are genuinely non-throwing
* Move Assignment operators that are genuinely non-throwing
* Non-throwing `swap` operations
* Cleanup operations
* Explicitly non-throwing callbacks
* Low-level utilities whose contract is no-throw

By contrast, operations such as parsing, allocation, I/O, and resource acquisition often require more careful consideration.

---

## Important Considerations for Library Authors

For a public library, `noexcept` is part of the API contract.

Before adding it, consider:

* Are all internal operations compatible with the contract?
* Are the members and base classes non-throwing?
* Can state remain consistent if an operation fails?
* Does the caller benefit from the operation being non-throwing?
* Is the property important for Move Semantics?
* Could changing the exception specification affect the API or ABI?
* Should a generic wrapper use conditional `noexcept`?

Answering these questions is generally better than applying `noexcept` mechanically.

---

## Recommended Pattern for Resource-Owning Types

A type that owns a resource should often be designed to be as move-friendly and non-throwing as possible:

```cpp
class Resource {
public:
    Resource(Resource&& other) noexcept;
    Resource& operator=(Resource&& other) noexcept;

    Resource(const Resource&) = delete;
    Resource& operator=(const Resource&) = delete;

    ~Resource() noexcept;
};
```

This design is common for handles, owning pointers, file descriptor wrappers, and many other resource abstractions. The actual implementation must still satisfy the contract.

---

## Recommended Pattern for Generic Wrappers

For generic wrappers, conditional `noexcept` is often the appropriate choice:

```cpp
template <typename T>
auto transform(T&& value)
    noexcept(noexcept(std::forward<T>(value).transform()))
{
    return std::forward<T>(value).transform();
}
```

This allows the wrapper to preserve the actual exception behavior of the underlying type without imposing an incorrect contract.

---

## Compile-Time Testing of `noexcept`

The desired contract can be tested with `static_assert`:

```cpp
#include <type_traits>

static_assert(std::is_nothrow_move_constructible_v<Widget>);
static_assert(std::is_nothrow_move_assignable_v<Widget>);
static_assert(noexcept(std::declval<Widget&>().swap(
    std::declval<Widget&>()
)));
```

These kinds of tests are particularly useful in library code because they expose accidental changes to exception specifications at compile time.

---

## Checking `noexcept` with `std::declval`

`std::declval` makes it possible to inspect expressions involving types without constructing actual objects:

```cpp
#include <utility>

template <typename T>
concept NothrowMovable =
    requires {
        requires noexcept(T(std::declval<T&&>()));
    };
```

In modern C++20 code, this approach can be combined naturally with Concepts.

---

## `noexcept` and the Strong Guarantee

Sometimes making an operation `noexcept` is the wrong choice because an exception may be an essential part of its rollback mechanism.

Suppose a function modifies complex state:

```cpp
void update() noexcept;
```

If the implementation needs potentially throwing operations to preserve consistency, adding `noexcept` can hide the problem and lead to termination.

In such cases, a more appropriate design may be:

```cpp
void update();
```

with a strong exception guarantee.

Therefore, non-throwing is not always better than throwing.

---

## `noexcept` and Error Handling

Correct use of `noexcept` depends on the program's error model.

If failure represents an exceptional condition, an exception may be appropriate:

```cpp
void parse_document();
```

If failure is a normal result of the operation, a Result type may be more appropriate:

```cpp
Result parse_document() noexcept;
```

If the operation fundamentally should not fail, `noexcept` may be appropriate:

```cpp
void release_resource() noexcept;
```

These decisions should follow the semantics of the program rather than a simple rule such as “the more `noexcept`, the better.”

---

## Important Considerations About Termination

When an exception escapes a `noexcept` function, `std::terminate` is called, and normal propagation to the caller does not continue.

This has serious consequences:

* The caller cannot catch the exception.
* Normal recovery is not possible.
* The terminate handler is executed.
* The process will usually terminate.
* Without appropriate diagnostics, the cause of the failure can be difficult to determine.

For this reason, incorrect use of `noexcept` can create a very serious bug.

---

## Debugging Termination Caused by `noexcept`

If a program unexpectedly stops through `std::terminate`, one of the things to investigate is whether an exception escaped a `noexcept` function.

For example:

```cpp
void process() noexcept {
    std::vector<int> values;
    values.resize(1000000);
}
```

In such a case, the potentially throwing internal operations should be examined.

A stack trace and an appropriate terminate handler can also help identify where the failure occurred.

---

## Important Code Review Considerations

When reviewing code involving `noexcept`, consider the following:

* Is `noexcept` actually the correct contract?
* Could the implementation allow an exception to escape without handling it?
* Should the Move Constructor be `noexcept`?
* Should Move Assignment be `noexcept`?
* Is the destructor non-throwing?
* Should a generic wrapper use conditional `noexcept`?
* Is `noexcept` on a virtual override compatible with the base contract?
* Was `noexcept` added solely for optimization?
* Could changing it affect the public API or ABI?
* Should failure be reported through an exception or a Result type instead?

---

## Comparison of Common Patterns

The following patterns can be compared as follows:

| Pattern                     | Primary use                                              | Recommendation                                     |
| --------------------------- | -------------------------------------------------------- | -------------------------------------------------- |
| `void f();`                 | A function that may propagate exceptions                 | Default choice for potentially-throwing operations |
| `void f() noexcept;`        | A function that must not allow exceptions to escape      | Use only when the contract is genuinely valid      |
| `void f() noexcept(false);` | A potentially-throwing function in generic expressions   | Usually unnecessary explicitly                     |
| `noexcept(expr)`            | Compile-time inspection of an expression                 | Very useful for generic code                       |
| `std::is_nothrow_*`         | Inspecting a type property                               | Appropriate for type-level constraints             |
| `std::move_if_noexcept`     | Choosing between move and copy based on exception safety | Useful in generic and container code               |

---

## Practical Recommendations

For professional use of `noexcept`, several principles are especially important.

First, treat `noexcept` as part of the API contract rather than as a decorative annotation.

Second, examine Move Constructors and Move Assignment operators in resource-owning types. If they are genuinely non-throwing, `noexcept` is generally appropriate.

Third, keep destructors non-throwing and move reportable failures into explicit operations such as `commit` or `close`.

Fourth, use conditional `noexcept` in Generic Programming so that wrappers preserve the actual exception behavior of the operations they invoke.

Fifth, use `noexcept(expr)` and type traits such as `std::is_nothrow_move_constructible` for compile-time verification.

Sixth, do not add `noexcept` merely in the hope of improving performance.

Seventh, if a function is `noexcept`, make sure that potential exceptions are either handled inside it or cannot escape.

Eighth, distinguish `noexcept` from other error-handling mechanisms such as `try`/`catch`, Result types, and `std::nothrow`.

---

## Summary of `noexcept`

The `noexcept` feature is an important part of modern C++ that specifies a function's contract regarding exception propagation.

The key point is that `noexcept` does not mean “no exception can ever be created inside this function.” It means that an exception must not escape from the function. If that contract is violated, `std::terminate` is called.

The real value of `noexcept` becomes apparent when it is combined with other parts of the language and standard library. Move Semantics, `std::vector`, Generic Programming, Concepts, type traits, function types, and RAII can all make use of information about whether an operation is non-throwing.

Therefore, the professional approach is not to add `noexcept` everywhere. Instead, use it where the function's semantics genuinely provide a no-throw guarantee and where that property provides meaningful value to callers or generic infrastructure.

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>