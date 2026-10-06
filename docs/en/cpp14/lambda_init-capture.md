<div align="center">

[🇺🇸 English](./lambda_init-capture.md) | [🇮🇷 فارسی](../../fa/cpp14/lambda_init-capture.md)

</div>

---

# Understanding init-capture in C++14

## Introduction

The `init-capture` feature in C++14 is one of the most important additions to lambda expressions. It allows a closure object to define an initial value, or even a different type, for a captured entity.

Before C++14, lambda captures were primarily expressed as regular captures; for example, `[x]` captured an existing variable by copy, while `[&x]` captured the same variable by reference. This model was sufficient for many use cases, but it had a significant limitation for scenarios such as transferring ownership of a `std::unique_ptr` into a lambda.

The `init-capture` feature solves this problem by allowing syntax such as `[p = std::move(p)]`. In this case, the lambda has an internal member named `p` that is initialized from the expression on the right-hand side.

This feature is fundamental to important patterns such as move-capture, self-contained closures, lifetime management, and transferring ownership into callbacks.

## Defining init-capture

In a regular lambda, a capture usually refers to an existing variable:

```cpp
int value = 42;

auto lambda = [value]() {
    return value;
};
```

With `init-capture`, a new name is introduced inside the closure and initialized with an initializer:

```cpp
int value = 42;

auto lambda = [captured = value]() {
    return captured;
};
```

Here, `captured` is the name available inside the lambda body. It does not have to be the name of an external variable.

Conceptually, `[captured = value]` means:

> Create an internal closure state named `captured` and initialize it with `value`.

This is the most important property of `init-capture`: it allows the closure's internal state to be independent of the name, and even the lifetime, of an external local variable.

## Why init-capture Exists

One important limitation of regular captures was that a capture had to be associated with a variable from the surrounding scope.

For example, the following code cannot directly use regular copy capture for move-only ownership:

```cpp
std::unique_ptr<int> ptr = std::make_unique<int>(42);

auto lambda = [ptr]() {
    return *ptr;
};
```

Copying a `std::unique_ptr` is not allowed, so `[ptr]` cannot transfer ownership into the closure.

With `init-capture`, ownership can be transferred directly:

```cpp
std::unique_ptr<int> ptr = std::make_unique<int>(42);

auto lambda = [ptr = std::move(ptr)]() {
    return *ptr;
};
```

Here, `ptr` on the left is the name of the state inside the closure, while `std::move(ptr)` on the right is the expression used to initialize that state.

After the lambda is created, the external `ptr` is in a moved-from state and ownership resides inside the closure.

## Basic init-capture Syntax

The basic form of `init-capture` is:

```cpp
[identifier = initializer]
```

For example:

```cpp
int x = 10;

auto lambda = [value = x + 5]() {
    return value;
};
```

In this example, `value` will be `15`.

Multiple `init-capture`s can also be used at the same time:

```cpp
int width = 10;
int height = 20;

auto area = [w = width, h = height]() {
    return w * h;
};
```

The closure now has two internal pieces of state named `w` and `h`.

## Difference Between Regular Capture and init-capture

With regular capture, the capture name is the same as the external variable name:

```cpp
int value = 42;

auto lambda = [value]() {
    return value;
};
```

With `init-capture`, a different internal name can be chosen:

```cpp
int value = 42;

auto lambda = [capturedValue = value]() {
    return capturedValue;
};
```

This difference is not merely cosmetic. `init-capture` allows the initializer to be an arbitrary expression:

```cpp
int x = 10;
int y = 20;

auto lambda = [sum = x + y]() {
    return sum;
};
```

Here, `sum` is not an independent variable in the outer scope at all; its state is created when the closure is constructed.

## Determining the Type in init-capture

In the usual `init-capture` form, the type of the internal state is determined from the initializer, conceptually similar to initialization with `auto`.

For example:

```cpp
const int value = 42;

auto lambda = [x = value]() {
    return x;
};
```

The type of `x` is deduced from the initializer, and the `const` qualification of the external variable does not automatically make the corresponding closure member a `const` member.

This is important because `init-capture` should not be thought of merely as another spelling of `[value]`. In reality, the initializer determines how the internal closure state is created.

## Move-capture and Ownership Transfer

One of the most important uses of `init-capture` is transferring a move-only object into a lambda.

The classic example is `std::unique_ptr`:

```cpp
#include <iostream>
#include <memory>

int main() {
    auto ptr = std::make_unique<int>(42);

    auto lambda = [ptr = std::move(ptr)]() {
        std::cout << *ptr << '\n';
    };

    lambda();
}
```

In this example, the lambda owns the pointed-to object.

This pattern is particularly important for callbacks:

```cpp
#include <memory>
#include <thread>

void startAsync(std::unique_ptr<int> data) {
    std::thread worker(
        [data = std::move(data)]() {
            // Use the owned data here.
        }
    );

    worker.detach();
}
```

In such scenarios, ownership can move together with the callback, allowing the callback to retain the state it needs independently of its caller.

In real programs, however, using `detach()` requires careful consideration of lifetime and synchronization. `init-capture` itself provides no guarantees about thread safety or synchronization.

## Difference Between Move-capture and Reference Capture

These two patterns have completely different behavior:

```cpp
auto lambda1 = [value]() {
    return value;
};

auto lambda2 = [&value]() {
    return value;
};
```

In the first case, the closure has independent state. Changes to the external `value` after the closure is created do not affect its internal state.

In the second case, the closure refers to the external object, and that object's lifetime must remain valid for as long as the lambda is used.

`init-capture` can explicitly create independent ownership or state:

```cpp
auto lambda = [value = std::move(value)]() {
    return value;
};
```

This distinction is especially important when designing callbacks: if a callback needs to carry its state with it, value capture or `init-capture` often provides a clearer ownership model than reference capture.

## Creating Computed State for a Closure

Sometimes a lambda needs a computed value that should be calculated only once, when the closure is created.

For example:

```cpp
std::string makePrefix();

auto print = [prefix = makePrefix()]() {
    // Use the already-created prefix.
};
```

Here, `makePrefix()` is executed when the lambda is created, not every time the lambda is invoked.

This is useful for callbacks that need prebuilt configuration or state:

```cpp
int timeout = 100;

auto callback = [
    timeoutMs = timeout,
    message = std::string("request completed")
]() {
    // Use timeoutMs and message.
};
```

Such a closure receives all of its required state when it is constructed.

## Limiting State to What Is Actually Needed

`init-capture` can help avoid capturing a larger scope than necessary.

For example, instead of retaining a large object merely to access one small value:

```cpp
struct Config {
    int timeout;
    std::string host;
    std::string certificate;
};

Config config;

auto callback = [config]() {
    // Use only config.timeout.
};
```

only the required value can be captured:

```cpp
auto callback = [timeout = config.timeout]() {
    // Use timeout only.
};
```

This can make lifetime and ownership clearer and reduce the closure's coupling to larger objects.

However, if other state is genuinely required, the design should not be made artificially small merely for the sake of reducing the capture list.

## Renaming a Captured Value

A simple but useful application of `init-capture` is giving the internal state a different name:

```cpp
std::string requestId = "abc";

auto callback = [id = requestId]() {
    // Use id here.
};
```

This can be useful when the external variable's name is not appropriate for the context inside the lambda or when the closure should be less dependent on the external name.

A more complex expression can also be given a meaningful name:

```cpp
auto callback = [
    normalizedPath = normalizePath(inputPath)
]() {
    // Use normalizedPath here.
};
```

In this example, the closure depends on the final result rather than on the details of how that result was computed.

## Reference init-capture

`init-capture` is not limited to creating value-like state. It can also be used with references:

```cpp
int value = 42;

auto lambda = [&ref = value]() {
    ++ref;
};
```

Here, `ref` is an internal reference referring to the target object.

This form must be used carefully because the lifetime still depends on the original object.

For example:

```cpp
auto makeLambda() {
    int value = 42;

    return [&ref = value]() {
        return ref;
    };
}
```

This code has a lifetime problem. After `makeLambda` returns, the object corresponding to `value` no longer exists, and the lambda contains a dangling reference.

This problem does not disappear just because `init-capture` is being used.

## Difference Between Value init-capture and Reference init-capture

The following two patterns have completely different ownership and lifetime semantics:

```cpp
auto byValue = [copy = value]() {
    return copy;
};

auto byReference = [&ref = value]() {
    return ref;
};
```

In the first case, the closure has independent state.

In the second case, the closure refers to an external object.

When designing a callback, the following question should always be clear:

> Should the callback retain the object itself, or should it merely access an existing object for as long as that object remains valid?

If the first answer applies, value-based state is generally a better ownership model. If the second answer applies, a reference may be appropriate, but the lifetime must be guaranteed precisely.

## init-capture and `mutable`

By default, `operator()` of a lambda observes captured values as const. If a value-captured state needs to be modified, the lambda must be declared `mutable`:

```cpp
int value = 10;

auto counter = [value]() mutable {
    ++value;
    return value;
};

counter();
counter();
```

In this example, modifying `value` changes only the closure's internal state and does not change the external `value`.

The same applies to `init-capture`:

```cpp
auto counter = [count = 0]() mutable {
    return ++count;
};
```

This pattern is common for creating stateful closures.

An important point is that `mutable` does not make a closure thread-safe. If multiple threads access a stateful closure, synchronization is still the responsibility of the program.

## init-capture and `const`

If the initializer is a `const` object, the distinction between the original object and the internal state must be understood:

```cpp
const int value = 42;

auto lambda = [copy = value]() mutable {
    ++copy;
    return copy;
};
```

Here, `copy` can be modified because the closure contains an independent object whose type is deduced from the initializer.

By contrast, reference capture refers to the original object:

```cpp
const int value = 42;

auto lambda = [&value]() {
    return value;
};
```

This difference is one reason it is important to distinguish between capturing an object and capturing a reference to an object.

## init-capture and `this`

In C++14, a lambda can capture `this`:

```cpp
class Widget {
public:
    int value = 42;

    auto makeLambda() {
        return [this]() {
            return value;
        };
    }
};
```

However, capturing `this` in this form does not mean that the object is retained by value. The lambda contains a pointer to the object, and the lifetime of the object itself must remain valid while the lambda is used.

A common problem is:

```cpp
class Widget {
public:
    auto makeLambda() {
        return [this]() {
            return value();
        };
    }

    int value() const {
        return 42;
    }
};
```

If the `Widget` object is destroyed before the lambda is executed, the lambda contains a dangling pointer.

In C++14, an independent copy of the object can be created using `init-capture`:

```cpp
class Widget {
public:
    int value = 42;

    auto makeLambda() {
        return [self = *this]() {
            return self.value;
        };
    }
};
```

This is fundamentally different from `[this]`: `self` is an independent object stored inside the closure.

In C++17, direct value capture of the object with `[*this]` was also introduced:

```cpp
class Widget {
public:
    int value = 42;

    auto makeLambda() {
        return [*this]() {
            return value;
        };
    }
};
```

Therefore, in C++17 and later, `[*this]` is generally a more direct way to express the same intent; in C++14, `[self = *this]` can be used.

## Limitations of Copying `*this`

Creating a copy of an object with `[self = *this]` or `[*this]` is not always an appropriate choice.

If the class has large state, every closure can contain an independent copy of that state. Also, if the class contains resources with special semantics, copying the object may not be possible at all.

For example, an object containing a `std::unique_ptr` is not copyable by default:

```cpp
#include <memory>

class Widget {
    std::unique_ptr<int> data;

public:
    auto makeLambda() {
        return [self = *this]() {
            return *self.data;
        };
    }
};
```

This code cannot compile because `*this` is being copied, unless the class is made copyable.

Therefore, value-capturing `*this` should be used only when the object's copy semantics are appropriate.

## init-capture and Move-only Objects

`init-capture` can be used with any move-enabled type, not only `std::unique_ptr`.

For example:

```cpp
#include <fstream>

auto file = std::make_unique<std::ofstream>("output.txt");

auto writer = [file = std::move(file)]() {
    *file << "hello\n";
};
```

Custom types can also be moved:

```cpp
class Connection {
public:
    Connection(Connection&&) = default;
    Connection(const Connection&) = delete;

    void send() {
    }
};

Connection connection;

auto callback = [connection = std::move(connection)]() mutable {
    connection.send();
};
```

In this pattern, the closure becomes the owner or holder of the transferred state.

## init-capture and Perfect Forwarding

In template code, `init-capture` can be used to convert an input into appropriate closure state, but it is important to distinguish this from perfect forwarding.

For example:

```cpp
template <typename T>
auto makeHandler(T&& value) {
    return [stored = std::forward<T>(value)]() mutable {
        return stored;
    };
}
```

Here, `std::forward<T>` determines how the object is transferred into the closure, and `stored` becomes the closure's final state.

This pattern is useful for generic factories because it can accept both lvalues and rvalues with appropriate semantics.

In such code, the copying or moving of the object, as well as the cost of constructing the closure, should be considered.

## init-capture in Generic Lambdas

Generic lambdas have been available since C++14, and combining them with `init-capture` provides a powerful mechanism:

```cpp
auto makePrinter = [prefix = std::string("value: ")](auto value) {
    std::cout << prefix << value << '\n';
};
```

Here, `prefix` is determined when the closure is created, while the type of `value` is determined on each invocation.

This combination is useful for creating small, type-safe, stateful function objects.

## init-capture for Type Conversion

The initializer can also be used to perform a type conversion:

```cpp
double value = 42.5;

auto lambda = [integerValue = static_cast<int>(value)]() {
    return integerValue;
};
```

In this case, the closure stores only the result of the conversion.

This can prevent the original object from being captured and make the callback's required state explicit.

## init-capture and Lifetime

One of the most important professional aspects of `init-capture` is understanding lifetime.

In the following example:

```cpp
auto makeLambda() {
    std::string text = "hello";

    return [copy = text]() {
        return copy;
    };
}
```

the lambda is safe because `copy` is an independent object stored inside the closure.

This version, however, has a lifetime problem:

```cpp
auto makeLambda() {
    std::string text = "hello";

    return [&ref = text]() {
        return ref;
    };
}
```

In the second version, `ref` refers to a local object that is destroyed when the function returns.

Therefore, an important rule is:

> `init-capture` does not automatically make lifetime safe; the type of capture determines whether the closure owns its state or merely depends on another existing object.

## init-capture in Asynchronous Callbacks

In asynchronous callbacks, capture strategy becomes even more important because the callback may execute much later than the scope in which it was created.

A value-based pattern:

```cpp
void schedule() {
    std::string message = "done";

    enqueue([message = std::move(message)]() {
        process(message);
    });
}
```

Here, the callback owns its own state.

A reference-based pattern:

```cpp
void schedule() {
    std::string message = "done";

    enqueue([&message]() {
        process(message);
    });
}
```

If `enqueue` executes the callback after `schedule` has returned, the reference may be dangling.

For asynchronous callbacks, the lifetime contract and execution timing of the callback API must be understood precisely. No capture syntax can replace correct lifetime design.

## init-capture for Self-contained Closures

One important benefit of this feature is creating closures with minimal dependence on the surrounding scope:

```cpp
auto makeMultiplier(int factor) {
    return [factor](int value) {
        return value * factor;
    };
}
```

The same idea can be used with more complex initializers:

```cpp
auto makeMultiplier(int factor) {
    return [factorValue = std::make_shared<int>(factor)](int value) {
        return value * (*factorValue);
    };
}
```

In the second version, ownership of a `shared_ptr` is transferred into the closure.

Using `shared_ptr` should have a clear purpose; it should not be used merely to make lifetime management easier without considering the actual ownership model.

## Difference Between init-capture and `std::bind`

Older code may use `std::bind` to retain arguments:

```cpp
auto handler = std::bind(process, std::move(resource));
```

In many callback scenarios, `init-capture` is more readable and provides more explicit control:

```cpp
auto handler = [resource = std::move(resource)]() {
    process(resource);
};
```

The lambda explicitly shows what state is stored, and its body precisely defines what happens during invocation.

This does not mean that `std::bind` is a deprecated language feature; the choice should be based on the API, readability, and actual requirements.

## Difference Between init-capture and a Local Variable

These two pieces of code may appear similar:

```cpp
auto value = calculate();

auto lambda = [value]() {
    return value;
};
```

and:

```cpp
auto lambda = [value = calculate()]() {
    return value;
};
```

The second version defines the state required by the closure directly at the point where the lambda is created.

This can make the scope smaller and the intent clearer, especially when the value is needed only by the lambda.

The second version also avoids leaving an unnecessary local variable in the surrounding scope.

## Evaluation Order of Initializers

Each `init-capture` initializer is evaluated when the closure is created.

For example:

```cpp
int counter = 0;

auto lambda = [
    first = ++counter,
    second = ++counter
]() {
    return first + second;
};
```

In examples like this, behavior should not be inferred merely from the visual order of the code. In production code, initializers with side effects should generally be kept simple and independent.

The exact ordering and semantics of closure initialization are governed by the language standard, and complex behavior should not be based on unnecessary assumptions about evaluation order.

When the ordering of side effects matters, it is often clearer to make it explicit:

```cpp
int first = ++counter;
int second = ++counter;

auto lambda = [first, second]() {
    return first + second;
};
```

## init-capture and `std::move` with Reuse of the Original Variable

After moving an object into a closure, the external object should not be assumed to retain its previous state:

```cpp
std::unique_ptr<int> ptr = std::make_unique<int>(42);

auto lambda = [ptr = std::move(ptr)]() {
    return *ptr;
};
```

After this operation, the external `ptr` is in a moved-from state.

For `std::unique_ptr`, this normally means that it becomes null, but the general rule for all types is not that a moved-from object has some particular value. The move constructor or move assignment operator of the relevant type defines its post-move state.

Therefore, if the original object is still needed after the move, the design should be reconsidered.

## Capturing Large Objects

`init-capture` can store large objects inside a closure:

```cpp
std::vector<int> values = createValues();

auto callback = [values = std::move(values)]() {
    process(values);
};
```

This may be excellent from an ownership perspective, but it also increases the size of the closure.

If such callbacks are stored in large numbers of objects or are frequently copied or moved, the size of the closure and the cost of copying or moving its state should be considered.

Possible approaches include:

* retaining only the state that is actually necessary
* moving an object instead of copying it
* using an appropriate ownership abstraction
* avoiding unnecessary copies of the closure
* using a reference when lifetime and thread-safety guarantees genuinely make that appropriate

Reference capture should not be used merely for performance reasons; a dangling reference is usually much more expensive than a controlled copy.

## Exception Safety in Initializers

`init-capture` initializers execute when the closure is constructed and can throw exceptions:

```cpp
auto callback = [
    resource = createResource()
]() {
    use(resource);
};
```

If `createResource()` throws, the closure is not constructed.

Resource acquisition should therefore be designed with the RAII and exception-safety properties of the involved types in mind.

If multiple initializers have side effects that depend on one another, reasoning about failure and cleanup can become difficult. In such cases, constructing the state in a helper function or a separate object may provide a clearer design.

## init-capture and Later Standards

The basic `init-capture` feature was introduced in C++14 and remains valid in newer standards.

C++17 introduced direct value capture of `*this` with the syntax `[*this]`. This is useful when a lambda needs to contain a copy of the current object.

In C++20, lambda and template capabilities were extended further, including features such as pack expansion in captures for variadic scenarios.

Despite these additions, the basic `identifier = initializer` syntax remains one of the primary tools for constructing stateful closures.

## init-capture and Packs in C++20

In modern variadic code, `init-capture` can be combined with pack expansion. This is useful for transferring a set of objects into a closure.

A conceptual C++20 pattern can look like this:

```cpp
template <typename... Args>
auto makeCallback(Args&&... args) {
    return [...values = std::forward<Args>(args)]() {
        // Use the captured pack.
    };
}
```

This feature is useful when designing advanced generic utilities because a collection of arguments can be turned into closure state.

However, variadic code involving ownership requires additional care regarding copy/move semantics and lifetime.

## init-capture and `decltype`

A captured name can be used in the lambda body like an ordinary name in expressions:

```cpp
auto lambda = [value = 42]() {
    using T = decltype(value);
    return value;
};
```

However, `decltype` follows its normal rules for expressions and names, and the exact closure-member details should not be treated as a public interface.

In generic design, it is generally better to rely on the closure's behavior and interface rather than on assumptions about its internal layout.

## Capture Names and Shadowing

`init-capture` can choose a name that is also used in surrounding scopes:

```cpp
int value = 10;

auto lambda = [value = value + 1]() {
    return value;
};
```

In this example, the right-hand side of the initializer refers to the external `value`, while the left-hand side introduces the new closure state.

Although this syntax is valid and common, complex expressions can reduce readability. Using distinct names for more complicated captures can make debugging and code review easier:

```cpp
int value = 10;

auto lambda = [capturedValue = value + 1]() {
    return capturedValue;
};
```

## Common Mistakes with init-capture

A common mistake is assuming that every `init-capture` provides safe ownership. This is not true for reference init-captures.

Another common mistake is forgetting `mutable` when modifying internal state:

```cpp
auto counter = [count = 0]() {
    ++count; // Error: operator() is const by default.
    return count;
};
```

The correct version is:

```cpp
auto counter = [count = 0]() mutable {
    ++count;
    return count;
};
```

Another mistake is moving an object into a lambda and then using the external object without considering its moved-from state:

```cpp
auto ptr = std::make_unique<int>(42);

auto callback = [ptr = std::move(ptr)]() {
    return *ptr;
};

// Do not assume ptr still owns the object here.
```

It is also incorrect to assume that a lambda containing move-only state is compatible with every callable API.

## Lifetime-related Mistakes

The following pattern is dangerous:

```cpp
auto makeCallback() {
    int value = 42;

    return [&captured = value]() {
        return captured;
    };
}
```

The problem is not `init-capture`; the problem is that the closure stores a reference to a local object.

The value-based version is:

```cpp
auto makeCallback() {
    int value = 42;

    return [captured = value]() {
        return captured;
    };
}
```

In this version, the closure has an independent object.

The same analysis must be applied to every captured object in asynchronous callbacks.

## Mistakes with Mutable State

The following lambda modifies its internal state:

```cpp
auto counter = [count = 0]() mutable {
    return ++count;
};
```

However, if multiple copies of `counter` are created, each copy has its own independent state:

```cpp
auto counter1 = counter;
auto counter2 = counter;
```

Changes to `counter1` do not affect the state of `counter2`.

This behavior is important when designing stateful function objects. If the state must be shared among multiple consumers, explicit shared state should be designed, for example with an object managed by `std::shared_ptr` when that ownership model is appropriate.

## init-capture and Closure Copy and Move Semantics

A lambda expression creates a closure object, and its captures are part of the closure's state.

If the closure contains a move-only object, the closure's ability to be copied is affected as well.

For example:

```cpp
auto ptr = std::make_unique<int>(42);

auto lambda = [ptr = std::move(ptr)]() {
    return *ptr;
};
```

This closure contains move-only state and therefore cannot be used like a fully copyable closure.

This becomes important when passing lambdas to APIs. Some APIs copy callbacks, while others only move them.

If an API requires a CopyConstructible callable, a lambda containing a `std::unique_ptr` may not be compatible with it.

## Checking Copyability in API Design

Suppose an API copies its callback:

```cpp
template <typename Callback>
void registerCallback(Callback callback) {
    auto another = callback;
    // ...
}
```

If the callback contains move-only state, this code will fail.

By contrast, if an API is designed around move-only callables:

```cpp
template <typename Callback>
void registerCallback(Callback&& callback) {
    // Store or forward the callback according to the API contract.
}
```

it may be able to support a lambda containing move-only state.

Therefore, choosing `init-capture` is not merely a local decision inside the lambda; the contract of the callable consumer must also be considered.

## init-capture and `std::function`

In C++14, `std::function` is generally designed for targets that are CopyConstructible. As a result, a lambda that stores a `std::unique_ptr` through move-capture cannot be used directly as the ordinary target of a `std::function`.

For example:

```cpp
auto ptr = std::make_unique<int>(42);

auto lambda = [ptr = std::move(ptr)]() {
    return *ptr;
};

// std::function<int()> fn = std::move(lambda); // Not valid in C++14.
```

In newer standards, the semantics of callable type erasure and the restrictions of the selected wrapper must still be considered. If a callable needs to be move-only, the abstraction used to store it must actually support move-only callables.

This is an important reason to examine the destination API before choosing move-capture.

## init-capture and Recursive Lambdas

An advanced use case is creating closures that can access themselves. In C++14, one approach is using `std::function`:

```cpp
#include <functional>

std::function<int(int)> factorial;

factorial = [&factorial](int n) {
    if (n <= 1) {
        return 1;
    }

    return n * factorial(n - 1);
};
```

This works, but it introduces the overhead and type-erasure associated with `std::function`.

In newer standards, recursive generic lambda patterns can use a self-parameter and avoid `std::function` in many cases:

```cpp
auto factorial = [](auto&& self, int n) -> int {
    if (n <= 1) {
        return 1;
    }

    return n * self(self, n - 1);
};
```

Therefore, `init-capture` is an important tool, but it is not the only mechanism related to stateful or recursive lambdas.

## init-capture and Function Parameters

Sometimes a function parameter needs to be transferred into a callback:

```cpp
void submit(std::unique_ptr<Task> task) {
    enqueue([task = std::move(task)]() {
        task->run();
    });
}
```

This is one of the most natural uses of `init-capture` in asynchronous APIs.

In such designs, the ownership transfer should be documented. The caller should understand that it no longer owns the resource after calling `submit`.

## init-capture and Avoiding Dangling References

The following pattern is often safer than capturing a reference to a local object:

```cpp
std::string makeMessage();

auto makeCallback() {
    return [message = makeMessage()]() {
        process(message);
    };
}
```

Here, the callback carries the object with it.

However, if the object is very large or ownership needs to be shared among multiple consumers, another approach may be more appropriate.

The key point is that lifetime should be designed according to the required semantics, rather than based solely on the shortest syntax.

## Difference Between `init-capture` and `std::move`

`std::move` does not move anything by itself; it merely converts an expression into a form suitable for a move operation.

In this code:

```cpp
auto lambda = [ptr = std::move(ptr)]() {
};
```

the actual move is performed by initialization of the closure's `ptr` state.

It is therefore useful to keep the concepts separate:

* `std::move` performs a cast to an xvalue.
* `init-capture` creates the closure state.
* The move constructor or move assignment operator of the relevant type performs the actual transfer of resources or state.

This distinction is important for understanding the language semantics accurately.

## init-capture and Value Categories

In the following expression:

```cpp
auto lambda = [value = std::move(object)]() {
};
```

`std::move(object)` produces an xvalue. The closure state is then initialized from that expression.

If the initializer is an lvalue:

```cpp
auto lambda = [value = object]() {
};
```

the state is generally initialized by copying from the object.

If the initializer is a prvalue:

```cpp
auto lambda = [value = createObject()]() {
};
```

the result of the expression is used to initialize the state, with the language's applicable temporary materialization and copy-elision rules.

Thus, `init-capture` integrates naturally with C++ move semantics and value categories.

## init-capture and `decltype`

A captured name can be used in the lambda body like a normal name in expressions:

```cpp
auto lambda = [value = 42]() {
    using T = decltype(value);
    return value;
};
```

However, the semantics of `decltype` for captured names should be understood according to the normal `decltype` rules. The exact closure-member representation is an implementation detail and should not be relied upon for naming or layout assumptions.

In generic code, it is generally better to rely on the closure's behavior and interface rather than assumptions about its internal layout.

## Capture Names and Shadowing

`init-capture` can choose a name that is also used in an enclosing scope:

```cpp
int value = 10;

auto lambda = [value = value + 1]() {
    return value;
};
```

In this example, the right-hand side of the initializer refers to the external `value`, while the left-hand side introduces the new closure state.

Although this syntax is legal and common, complex expressions can reduce readability. Using distinct names for complicated captures often makes debugging and code review easier:

```cpp
int value = 10;

auto lambda = [capturedValue = value + 1]() {
    return capturedValue;
};
```

## Common Mistakes in Using init-capture

A common mistake is assuming that every `init-capture` transfers safe ownership. This assumption is incorrect for reference init-captures.

Another common mistake is forgetting `mutable` when modifying internal state:

```cpp
auto counter = [count = 0]() {
    ++count; // Error: operator() is const by default.
    return count;
};
```

The correct version is:

```cpp
auto counter = [count = 0]() mutable {
    ++count;
    return count;
};
```

Another mistake is moving an object into a closure and then using the original object without considering its moved-from state:

```cpp
auto ptr = std::make_unique<int>(42);

auto callback = [ptr = std::move(ptr)]() {
    return *ptr;
};

// Do not assume ptr still owns the object here.
```

It is also incorrect to assume that a lambda containing move-only state is compatible with every callable API.

## Lifetime-related Mistakes

The following pattern is dangerous:

```cpp
auto makeCallback() {
    int value = 42;

    return [&captured = value]() {
        return captured;
    };
}
```

The problem is not `init-capture`; the problem is that the closure stores a reference to a local object.

The value-based version is:

```cpp
auto makeCallback() {
    int value = 42;

    return [captured = value]() {
        return captured;
    };
}
```

In this version, the closure has its own independent object.

The same lifetime analysis should be applied to every captured object in asynchronous callbacks.

## Performance and Closure Size

Each `init-capture` generally adds state to the closure object. Therefore, this code:

```cpp
auto callback = [
    largeObject = createLargeObject(),
    cache = createLargeCache()
]() {
    use(largeObject, cache);
};
```

may produce a large closure.

If many such callbacks are stored or if they are frequently copied or moved, the size of the state and the cost of copying or moving the closure may become significant.

Depending on the problem, appropriate approaches may include:

* retaining only the state that is actually required
* moving objects instead of copying them
* using an appropriate ownership abstraction
* avoiding unnecessary copies of the closure
* using a reference when lifetime and thread-safety are genuinely guaranteed

Reference capture should not be used merely for performance. A dangling reference is usually a much more serious problem than a controlled copy.

## Exception Safety in Initializers

`init-capture` initializers execute when the closure is constructed and may throw exceptions:

```cpp
auto callback = [
    resource = createResource()
]() {
    use(resource);
};
```

If `createResource()` throws, the closure is not constructed.

Resource acquisition should therefore be designed with the RAII and exception-safety guarantees of the involved types in mind.

If multiple initializers contain side effects that depend on one another, reasoning about failure and cleanup can become difficult. In such situations, constructing the state in a helper function or a separate object may provide a clearer design.

## init-capture and Later Standards

The basic `init-capture` feature was introduced in C++14 and remains valid in newer standards.

C++17 introduced direct value capture of `*this` using the syntax `[*this]`. This is useful when a lambda needs to contain a copy of the current object.

C++20 further expanded lambda and template capabilities, including features such as pack expansion in captures for variadic scenarios.

Despite these developments, the basic `identifier = initializer` syntax remains one of the primary tools for constructing stateful closures.

## init-capture and Packs in C++20

In modern variadic code, `init-capture` can be combined with pack expansion. This is useful for transferring a collection of objects into a closure.

A conceptual C++20 pattern can look like this:

```cpp
template <typename... Args>
auto makeCallback(Args&&... args) {
    return [...values = std::forward<Args>(args)]() {
        // Use the captured pack.
    };
}
```

This feature is useful when designing advanced generic utilities because a set of arguments can be turned into closure state.

However, variadic code involving ownership requires additional care regarding copy/move semantics and lifetime.

## init-capture and `decltype`

A captured name can be used in the lambda body like an ordinary name in expressions:

```cpp
auto lambda = [value = 42]() {
    using T = decltype(value);
    return value;
};
```

However, `decltype` follows its normal rules for expressions and names, and the exact details of the closure's internal members should not be treated as a public interface.

In generic design, it is generally better to rely on the closure's behavior and interface rather than assumptions about its internal layout.

## Capture Names and Shadowing

`init-capture` can choose a name that is also used in an enclosing scope:

```cpp
int value = 10;

auto lambda = [value = value + 1]() {
    return value;
};
```

In this example, the right-hand side of the initializer refers to the external `value`, while the left-hand side introduces the new closure state.

Although this syntax is legal and common, complex expressions can reduce readability. Using distinct names for more complicated captures can make debugging and code review easier:

```cpp
int value = 10;

auto lambda = [capturedValue = value + 1]() {
    return capturedValue;
};
```

## Common Mistakes When Using init-capture

A common mistake is assuming that every `init-capture` provides safe ownership. This is not true for reference init-captures.

Another common mistake is forgetting `mutable` when modifying internal state:

```cpp
auto counter = [count = 0]() {
    ++count; // Error: operator() is const by default.
    return count;
};
```

The correct version is:

```cpp
auto counter = [count = 0]() mutable {
    ++count;
    return count;
};
```

Another mistake is moving an object into a lambda and then using the external object without considering its moved-from state:

```cpp
auto ptr = std::make_unique<int>(42);

auto callback = [ptr = std::move(ptr)]() {
    return *ptr;
};

// Do not assume ptr still owns the object here.
```

It is also incorrect to assume that a lambda containing move-only state is compatible with every callable API.

## Common Lifetime Mistakes

The following pattern is dangerous:

```cpp
auto makeCallback() {
    int value = 42;

    return [&captured = value]() {
        return captured;
    };
}
```

The problem is not `init-capture`; the problem is that the closure stores a reference to a local object.

The value-based version is:

```cpp
auto makeCallback() {
    int value = 42;

    return [captured = value]() {
        return captured;
    };
}
```

In this version, the closure has an independent object.

For asynchronous callbacks, the same lifetime analysis must be applied to every captured object.

## Mutable State Mistakes

The following lambda modifies its internal state:

```cpp
auto counter = [count = 0]() mutable {
    return ++count;
};
```

However, if multiple copies of `counter` are created, each copy has its own independent state:

```cpp
auto counter1 = counter;
auto counter2 = counter;
```

Changes to `counter1` do not affect the state of `counter2`.

This behavior is important when designing stateful function objects. If the state must be shared among multiple consumers, explicit shared state should be designed, for example with an object managed by `std::shared_ptr` when that ownership model is appropriate.

## init-capture and Closure Copy and Move Semantics

A lambda expression creates a closure object, and its captures are part of the closure's state.

If the closure contains a move-only object, the closure's ability to be copied is affected as well.

For example:

```cpp
auto ptr = std::make_unique<int>(42);

auto lambda = [ptr = std::move(ptr)]() {
    return *ptr;
};
```

This closure contains move-only state and therefore cannot be used like a fully copyable closure.

This is important when passing lambdas to APIs. Some APIs copy callbacks, while others only move them.

If an API requires a CopyConstructible callable, a lambda containing a `std::unique_ptr` may not be compatible with it.

## Checking Copyability in API Design

Suppose an API copies its callback:

```cpp
template <typename Callback>
void registerCallback(Callback callback) {
    auto another = callback;
    // ...
}
```

If the callback contains move-only state, this code will fail.

By contrast, if an API is designed around move-only callables:

```cpp
template <typename Callback>
void registerCallback(Callback&& callback) {
    // Store or forward the callback according to the API contract.
}
```

it may be able to support a lambda containing move-only state.

Therefore, choosing `init-capture` is not merely a local decision inside the lambda; the callable consumer's contract must also be considered.

## init-capture and `std::function`

In C++14, `std::function` is generally designed for targets that are CopyConstructible. As a result, a lambda that stores a `std::unique_ptr` through move-capture cannot be used directly as the ordinary target of a `std::function`.

For example:

```cpp
auto ptr = std::make_unique<int>(42);

auto lambda = [ptr = std::move(ptr)]() {
    return *ptr;
};

// std::function<int()> fn = std::move(lambda); // Not valid in C++14.
```

In newer standards, the semantics of callable type erasure and the restrictions of the selected wrapper must still be considered. If a callable needs to be move-only, the abstraction used to store it must actually support move-only callables.

This is an important reason to examine the destination API before choosing move-capture.

## init-capture and Recursive Lambdas

An advanced use case is creating closures that can access themselves. In C++14, one approach is using `std::function`:

```cpp
#include <functional>

std::function<int(int)> factorial;

factorial = [&factorial](int n) {
    if (n <= 1) {
        return 1;
    }

    return n * factorial(n - 1);
};
```

This works, but it introduces the overhead and type erasure associated with `std::function`.

In newer standards, recursive generic lambda patterns can use a self-parameter and avoid `std::function` in many cases:

```cpp
auto factorial = [](auto&& self, int n) -> int {
    if (n <= 1) {
        return 1;
    }

    return n * self(self, n - 1);
};
```

Therefore, `init-capture` is an important tool, but it is not the only mechanism related to stateful or recursive lambdas.

## init-capture and Function Parameters

Sometimes a function parameter needs to be transferred into a callback:

```cpp
void submit(std::unique_ptr<Task> task) {
    enqueue([task = std::move(task)]() {
        task->run();
    });
}
```

This is one of the most natural uses of `init-capture` in asynchronous APIs.

In such designs, the ownership transfer should be documented. The caller should understand that it no longer owns the resource after calling `submit`.

## init-capture and Avoiding Dangling References

The following pattern is often safer than capturing a reference to a local object:

```cpp
std::string makeMessage();

auto makeCallback() {
    return [message = makeMessage()]() {
        process(message);
    };
}
```

Here, the callback carries the object with it.

However, if the object is very large or ownership needs to be shared among multiple consumers, another approach may be more appropriate.

The key point is that lifetime should be designed according to the required semantics rather than based solely on the shortest syntax.

## Difference Between `init-capture` and `std::move`

`std::move` does not move anything by itself; it merely converts an expression into a form suitable for a move operation.

In this code:

```cpp
auto lambda = [ptr = std::move(ptr)]() {
};
```

the actual move is performed by initialization of the closure's `ptr` state.

It is therefore useful to keep the concepts separate:

* `std::move` performs a cast to an xvalue.
* `init-capture` creates the closure state.
* The move constructor or move assignment operator of the relevant type performs the actual transfer of resources or state.

This distinction is important for accurately understanding the language semantics.

## init-capture and Value Categories

In the following expression:

```cpp
auto lambda = [value = std::move(object)]() {
};
```

`std::move(object)` produces an xvalue. The closure state is then initialized from that expression.

If the initializer is an lvalue:

```cpp
auto lambda = [value = object]() {
};
```

the state is generally initialized by copying from the object.

If the initializer is a prvalue:

```cpp
auto lambda = [value = createObject()]() {
};
```

the result of the expression is used to initialize the state, with the applicable temporary materialization and copy-elision rules.

Therefore, `init-capture` integrates naturally with C++ move semantics and value categories.

## init-capture and `decltype`

A captured name can be used in the lambda body like a normal name in expressions:

```cpp
auto lambda = [value = 42]() {
    using T = decltype(value);
    return value;
};
```

However, the semantics of `decltype` for captured names should be understood according to the normal `decltype` rules. The exact closure-member representation is an implementation detail and should not be relied upon for naming or layout assumptions.

In generic code, it is generally better to rely on the closure's behavior and interface rather than assumptions about its internal layout.

## Capture Names and Shadowing

`init-capture` can choose a name that is also used in surrounding scopes:

```cpp
int value = 10;

auto lambda = [value = value + 1]() {
    return value;
};
```

In this example, the right-hand side of the initializer refers to the external `value`, while the left-hand side introduces the new closure state.

Although this syntax is legal and common, complex expressions can reduce readability. Using distinct names for complicated captures can make debugging and code review easier:

```cpp
int value = 10;

auto lambda = [capturedValue = value + 1]() {
    return capturedValue;
};
```

## Common Mistakes When Using init-capture

A common mistake is assuming that every `init-capture` transfers safe ownership. This assumption is incorrect for reference init-captures.

Another common mistake is forgetting `mutable` when modifying internal state:

```cpp
auto counter = [count = 0]() {
    ++count; // Error: operator() is const by default.
    return count;
};
```

The correct version is:

```cpp
auto counter = [count = 0]() mutable {
    ++count;
    return count;
};
```

Another mistake is moving an object into a lambda and then using the external object without considering its moved-from state:

```cpp
auto ptr = std::make_unique<int>(42);

auto callback = [ptr = std::move(ptr)]() {
    return *ptr;
};

// Do not assume ptr still owns the object here.
```

It is also incorrect to assume that a lambda containing move-only state is compatible with every callable API.

## Common Lifetime Mistakes

The following pattern is dangerous:

```cpp
auto makeCallback() {
    int value = 42;

    return [&captured = value]() {
        return captured;
    };
}
```

The problem is not `init-capture`; the problem is that the closure stores a reference to a local object.

The value-based version is:

```cpp
auto makeCallback() {
    int value = 42;

    return [captured = value]() {
        return captured;
    };
}
```

In this version, the closure has its own independent object.

The same lifetime analysis must be applied to every captured object in asynchronous callbacks.

## Performance and Closure Size

Each `init-capture` generally adds state to the closure object. Therefore, this code:

```cpp
auto callback = [
    largeObject = createLargeObject(),
    cache = createLargeCache()
]() {
    use(largeObject, cache);
};
```

may produce a large closure.

If such callbacks are stored in large numbers or are frequently copied or moved, the size of the state and the cost of copying or moving the closure may become significant.

Depending on the problem, appropriate approaches may include:

* retaining only the state that is actually necessary
* moving objects instead of copying them
* using an appropriate ownership abstraction
* avoiding unnecessary copies of the closure
* using a reference when lifetime and thread-safety guarantees genuinely make that appropriate

Reference capture should not be used merely for performance reasons; a dangling reference is usually a much more serious problem than a controlled copy.

## Recommended Pattern for Resource-owning Callbacks

A common and clear C++14 pattern is:

```cpp
void submit(std::unique_ptr<Job> job) {
    enqueue([job = std::move(job)]() mutable {
        job->run();
    });
}
```

Here, ownership is transferred to the callback.

If `enqueue` does not support move-only callables or does not provide suitable storage semantics, this design may not be compatible with its API. Therefore, the implementation and lifetime contract of `enqueue` must be considered at the same time.

## Recommended Pattern for Computed State

If state is needed only by the callback, it can be constructed directly in the capture list:

```cpp
auto callback = [
    endpoint = normalizeEndpoint(config.endpoint),
    timeout = config.timeout
]() {
    connect(endpoint, timeout);
};
```

This style generally avoids unnecessary local variables and makes the callback's dependencies visible in one place.

## Recommended Pattern for Storing a Copy of an Object

In C++14, `init-capture` can be used to create a copy of the current object:

```cpp
class Session {
public:
    int id = 42;

    auto makeCallback() const {
        return [session = *this]() {
            return session.id;
        };
    }
};
```

This is appropriate when copying the object has the desired semantics and the object does not contain non-copyable resources.

In C++17, `[*this]` can express the same intent more directly:

```cpp
class Session {
public:
    int id = 42;

    auto makeCallback() const {
        return [*this]() {
            return id;
        };
    }
};
```

## Main Limitations

Despite its broad usefulness, `init-capture` has several important limitations.

First, its syntax does not automatically solve lifetime problems. A reference init-capture can still become dangling.

Second, capturing large objects by value can make the closure large.

Third, move-capture can make a closure move-only, making it incompatible with APIs that copy their callable objects.

Fourth, capturing `*this` by value can introduce the cost of copying the object and is usable only when the class has suitable copy semantics.

Fifth, `init-capture` does not replace proper ownership, synchronization, or exception-safety design.

## Summary

`init-capture` is one of the key lambda features introduced in C++14. It allows a closure's internal state to be created with an arbitrary initializer.

Its basic form is:

```cpp
auto lambda = [name = initializer]() {
    // Use name here.
};
```

Its most important applications include:

* transferring ownership with move-capture
* storing computed state
* renaming internal state
* creating self-contained closures
* managing callback lifetimes more explicitly
* storing move-only objects
* creating stateful lambdas
* combining with generic lambdas and template code

In C++14, syntax such as `[ptr = std::move(ptr)]` provides a standard and readable way to transfer a resource into a closure.

The most important professional principle is not to separate capture syntax from ownership and lifetime. When using `init-capture`, it should be clear whether the closure creates an independent copy, moves an object, or merely stores a reference. The closure's copyability and the contract of the API consuming the callback should also be considered.

In practice, `init-capture` is most valuable when it defines the lambda's required state explicitly, keeps that state minimal, gives it clear ownership semantics, and matches the actual lifetime of the callback.

---

## 🤝 Contributions

<div align="center">

| GitHub                                      | LinkedIn                                                           | Email                                                  | Site                           | Telegram                               |
| ------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------ | -------------------------------------- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](mailto:hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>
