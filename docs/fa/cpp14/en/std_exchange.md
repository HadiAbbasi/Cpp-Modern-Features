<div align="center">

[🇺🇸 English](./std_exchange.md) | [🇮🇷 فارسی](../../fa/cpp14/std_exchange.md)

</div>

---

# Introduction to `std::exchange` in C++14

`std::exchange` is a useful utility introduced in the C++14 standard library for replacing an object's value with a new value while returning its previous value. It is particularly useful for resource management, move semantics, ownership transfer, and resetting object state.

The core idea is simple: retrieve the object's current value, assign a new value to it, and return the previous value.

For example, suppose you have an integer variable whose value you want to replace with zero while preserving its previous value. `std::exchange` allows you to perform both operations in a single expression.

```
#include <iostream>
#include <utility>

int main() {
    int value = 42;

    int oldValue = std::exchange(value, 0);

    std::cout << "Old value: " << oldValue << '\n';
    std::cout << "New value: " << value << '\n';
}
```

The output is:

```
Old value: 42
New value: 0
```

In this example, the previous value of `value` is stored in `oldValue`, and `value` is updated to zero.

## Introducing `std::exchange`

`std::exchange` is declared in the standard `<utility>` header and has been available since C++14.

Its general signature is:

```
template<class T, class U = T>
constexpr T exchange(T& obj, U&& newValue)
    noexcept(/* implementation-dependent condition */);
```

This signature illustrates the function's general structure. The actual standard declaration specifies a condition for `noexcept` based on the relevant types and operations; it is not an arbitrary implementation choice.

The parameters are:

- `obj`: A reference to the object whose value will be replaced.
- `newValue`: The new value to assign to the object after obtaining its previous value.

The return type is `T`, so the function returns the object's previous value as an object of type `T`.

### How the Function Works

The function's conceptual behavior can be illustrated with the following code:

```
template<class T, class U = T>
constexpr T exchange_like(T& obj, U&& newValue) {
    T oldValue = std::move(obj);
    obj = std::forward<U>(newValue);
    return oldValue;
}
```

This code is an educational illustration of the function's behavior, not a complete replacement for the standard implementation.

In this implementation, the object's previous value is first moved into `oldValue` using `std::move`. The new value is then assigned to the original object using `std::forward`, and the previous value is returned.

An important point is that `std::move` does not move anything by itself. It enables overload resolution to select a move operation when one is available. The actual result depends on the constructors and assignment operators provided by the type.

## Comparing `std::exchange` with Ordinary Assignment

With ordinary assignment, a new value is assigned to a variable, but the previous value is not automatically returned.

```
int value = 10;

value = 20;
```

After this code executes, `value` is `20`, and its previous value is no longer directly available.

In contrast, `std::exchange` allows both operations to be performed together:

```
int value = 10;

int oldValue = std::exchange(value, 20);
```

Now, `oldValue` is `10`, and `value` is `20`.

This distinction becomes more important when the previous value represents a resource, an internal state, or ownership of an object.

## Understanding Template Parameters and Perfect Forwarding

One important feature of `std::exchange` is its use of the template parameter `U` and the forwarding reference `U&&`.

This design allows the new value to come from different sources, such as a temporary value, a constant variable, or an existing object.

```
#include <string>
#include <utility>

int main() {
    std::string value = "old";

    std::string previous = std::exchange(value, "new");

    std::string another = "replacement";
    std::string oldValue = std::exchange(value, another);
}
```

In the example above, the string literal `"new"` is used as the new value. In the second call, the value of `another` is assigned to `value`.

Because `newValue` has type `U&&` and `U` is deduced automatically, the function can use perfect forwarding.

In the conceptual implementation, `std::forward<U>(newValue)` preserves the original value category of the argument. This helps select the appropriate assignment operation.

However, using `std::exchange` does not guarantee that replacement will always occur without copying. The actual behavior depends on the types involved, their assignment operations, and the supplied argument.

## Comparing `std::exchange` with `std::move`

Both `std::exchange` and `std::move` are useful in scenarios involving move semantics, but they serve different purposes.

`std::move` converts an expression into an xvalue, enabling move operations to be selected when the type supports them. By itself, it does not modify the original variable.

In contrast, `std::exchange` returns an object's previous value and attempts to assign a new value to that object.

```
#include <string>
#include <utility>

int main() {
    std::string source = "hello";

    std::string destination = std::move(source);

    int value = 100;
    int oldValue = std::exchange(value, 0);
}
```

In this example, `std::move` enables a move operation from `source` to `destination`. The state of `source` afterward depends on the move operation provided by `std::string`.

By contrast, `std::exchange` returns the previous value of `value` and changes `value` to zero.

## Using `std::exchange` in a Move Constructor

One important application of `std::exchange` is transferring resource ownership in a move constructor.

Suppose a class owns a dynamically allocated resource. When ownership is transferred, the pointer must be obtained from the source object, and the source object must be returned to an empty state.

```
#include <utility>

class Resource {
public:
    explicit Resource(int* ptr = nullptr)
        : ptr_(ptr) {}

    Resource(const Resource&) = delete;
    Resource& operator=(const Resource&) = delete;

    Resource(Resource&& other) noexcept
        : ptr_(std::exchange(other.ptr_, nullptr)) {}

    ~Resource() {
        delete ptr_;
    }

private:
    int* ptr_;
};
```

In this example, when a new object is constructed from another object, the value of `other.ptr_` is transferred to `ptr_`, and the source object's pointer is set to `nullptr`.

This prevents both objects from believing they own the same resource.

However, this class is intended only as an educational example. In production code, `std::unique_ptr` is generally a safer and simpler choice for managing dynamically allocated resources.

## Using `std::exchange` in a Move Assignment Operator

Using `std::exchange` is not limited to move constructors. It can also be useful when implementing a move assignment operator.

```
#include <utility>

class Resource {
public:
    explicit Resource(int* ptr = nullptr)
        : ptr_(ptr) {}

    Resource(const Resource&) = delete;
    Resource& operator=(const Resource&) = delete;

    Resource(Resource&& other) noexcept
        : ptr_(std::exchange(other.ptr_, nullptr)) {}

    Resource& operator=(Resource&& other) noexcept {
        if (this != &other) {
            delete ptr_;
            ptr_ = std::exchange(other.ptr_, nullptr);
        }

        return *this;
    }

    ~Resource() {
        delete ptr_;
    }

private:
    int* ptr_;
};
```

In this implementation, the destination object's previous resource is released before ownership of the source object's resource is transferred.

The `this != &other` check is important because it prevents the resource from being incorrectly released during self-move assignment.

In professional designs, this pattern requires more careful consideration, especially when resource release, reallocation, or assignment operations can throw exceptions.

## Resetting Variable State

One simple and common use of `std::exchange` is resetting a variable to its initial state while preserving its previous value.

```
#include <utility>

int main() {
    bool active = true;

    bool wasActive = std::exchange(active, false);
}
```

In this example, the previous value of `active` is stored in `wasActive`, and `active` is set to `false`.

This pattern is useful for status flags, control variables, and recording previous states.

For example, when processing an event, a program can clear a pending-state flag and simultaneously determine whether an event was previously pending.

## Managing Queues and Events

`std::exchange` can be used to retrieve the contents of a variable and replace them with an empty value.

```
#include <string>
#include <utility>

int main() {
    std::string pendingMessage = "Process this message";

    std::string message = std::exchange(pendingMessage, {});
}
```

In this example, the contents of `pendingMessage` are transferred to `message`, and the original string is replaced with an empty string.

Using `{}` in this call allows the new value to be determined from the destination type, which is `std::string`.

This approach is suitable for transferring temporary contents, draining buffers, and moving data that is waiting to be processed. However, in multithreaded programs, concurrent access must also be managed to prevent data races.

## Working with Non-Copyable Types

`std::exchange` can be used with types that do not support copying, provided that the operations required to transfer the previous value, assign the new value, and return the result are valid.

For example, `std::unique_ptr` manages exclusive ownership of a resource and cannot be copied.

```
#include <memory>
#include <utility>

int main() {
    auto first = std::make_unique<int>(42);

    auto previous = std::exchange(
        first,
        std::make_unique<int>(100)
    );
}
```

After this code executes, `previous` owns the original value, `42`, while `first` owns the new value, `100`.

In this example, exclusive resource ownership is preserved, and no invalid copy of `std::unique_ptr` occurs.

## Understanding the Return Value and Move Semantics

The return type of `std::exchange` is `T`, so the function returns an object of type `T`.

In many cases, the previous value is returned through a move constructor or compiler optimizations. However, you should not assume that every call is necessarily free of cost.

If the type has an expensive move operation, or if its constructors and assignment operators have particular behaviors, the execution cost of the function will also be affected.

For this reason, when working with custom types, it is important to examine their move constructors, move assignment operators, and other related operations.

## Understanding `noexcept` and Exception Handling

An important consideration when using `std::exchange` is its behavior when exceptions occur.

The function's `noexcept` specification depends on the properties of the types involved and the operations required to replace the value. Therefore, you should not assume that using this function always guarantees that no exception can be thrown.

If assigning the new value throws an exception, the destination object might already have been modified before the operation fails. In such cases, the object's final state depends on the behavior of the assignment operation for the type involved.

Similarly, using `std::exchange` inside a move constructor declared `noexcept` requires care. If an internal operation can throw an exception, an incorrect `noexcept` declaration can cause `std::terminate` to be called at runtime.

For safe design, examine the exception specifications of the relevant types and determine `noexcept` specifications based on the actual behavior of the operations involved.

## Common Mistakes and Limitations

`std::exchange` is a powerful utility, but incorrect usage can lead to logical errors or resource management problems.

### Using an Inappropriate Reset Value

The new value must be compatible with the destination object's type, and the required assignment operation must be valid.

For example, when resetting a raw pointer to `nullptr`, ensure that the new state is consistent with the resource's ownership and lifetime contract.

### Assuming That Moving Is Free

Using `std::exchange` does not guarantee that the operation is cost-free. The actual cost depends on the data type and its defined operations.

### Ignoring the Source Object's State

After `std::exchange` is called, the original object is replaced with the new value. Therefore, subsequent code must not continue to rely on the object's previous value.

### Using It Incorrectly in Multithreaded Code

`std::exchange` is not atomic by itself. If multiple threads access the same shared variable, concurrent use without appropriate synchronization can cause data races.

For atomic variables, use operations appropriate to the atomic type and consider the relevant memory-ordering requirements.

## Comparing Alternative Approaches

Choosing between `std::exchange`, ordinary assignment, and `std::move` depends on the intended operation.

| Feature                    | `std::exchange`                           | Ordinary assignment                        | `std::move`                                 |
| -------------------------- | ----------------------------------------- | ------------------------------------------ | ------------------------------------------- |
| Replaces a value           | Yes                                       | Yes                                        | No, not by itself                           |
| Returns the previous value | Yes                                       | No                                         | No                                          |
| Transfers ownership        | When supported by the type and operations | Depends on the type and right-hand value   | Only enables move operations to be selected |
| Requires `<utility>`       | Yes                                       | No                                         | Yes                                         |
| Standard version           | C++14                                     | Available since the earliest C++ standards | C++11                                       |

In this table, ownership transfer refers to the behavior of the actual operations provided by the type, not an independent guarantee arising solely from the function's name.

## Design Considerations for Professional Developers

When designing APIs and custom classes, `std::exchange` is most useful when retrieving a previous value and resetting or replacing it are parts of the same logical operation.

For example, this function can make code more readable when implementing resource-managing classes, draining buffers, transferring internal state, and managing temporary values.

However, avoid overusing it in complex expressions. When an operation involves multiple side effects, exception handling, or dependent steps, splitting the code into several clear statements can improve readability and maintainability.

Additionally, when designing classes that own resources, RAII tools such as `std::unique_ptr` and `std::vector` are generally safer than manual memory management. `std::exchange` should support a sound ownership design rather than serve as a substitute for one.

## Summary

`std::exchange`, introduced into the standard library in C++14, provides a simple and effective way to replace an object's value while retrieving its previous value.

Its main applications include:

- Transferring resource ownership in move constructors and move assignment operators.
- Resetting flags and internal state.
- Draining variables that contain temporary data.
- Transferring values from non-copyable types that support the required move operations.
- Simplifying code that needs to retrieve a previous value and assign a new value in a single operation.

Ultimately, using `std::exchange` correctly requires an understanding of move semantics, assignment behavior, `noexcept` specifications, and the ownership contract of the type involved. When these considerations are handled appropriately, the function can make C++ code more readable, concise, and maintainable in many scenarios.

---

## 🤝 Contributions

<div align="center">

| GitHub                                      | LinkedIn                                                           | Email                                                  | Site                           | Telegram                               |
| ------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------ | -------------------------------------- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](mailto:hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>