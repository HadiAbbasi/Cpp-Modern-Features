<div align="right">

[🇺🇸 English](./visit.md) | [🇮🇷 فارسی](../../fa/cpp11/visit.md)

</div>

# std::variant & std::visit
Picture this: you’re coding away, and suddenly, your program needs to juggle different kinds of data — sometimes an integer, other times a string, maybe even a custom object. In the olden days of C++, the `union` type would be your go-to. It's like a box that can hold any one of its predefined types. But there's a catch: unions are a bit reckless. They don't keep track of what's currently inside, setting the stage for potential mishaps if you accidentally try to access the wrong type.


## std::variant
Fast forward to Modern C++, and we have std::variant, the type-safe champion. It's similar to a union, but way smarter. Imagine it as a box with multiple, labeled compartments, each tailored to hold a specific type. The magic is that std::variant always remembers which compartment is currently occupied. This makes it a far safer and more reliable choice for managing data that can take on different forms.

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    std::variant<int, std::string> data;

    data = 42; 
    std::cout << std::get<int>(data) << std::endl; // Output: 42 (Correct access)
    data = "Hello, world!"; 
    return 0;
}
```
### Type Safety in Action:
The beauty of std::variant lies in its type safety. If you attempt to access the variant with the wrong type using std::get, you'll encounter either:

- A compile-time error, preventing the issue before your program even runs
- A std::bad_variant_access exception at runtime, allowing you to handle the error gracefully

This robust error handling ensures that your code remains stable and predictable, even when dealing with data that can change its type.


## std::visit
So, you have your trusty std::variant, holding data that can change its type. But what if you want to perform different actions based on the current type inside? That's where std::visit steps in. Think of it as a multi-tool, equipped to handle all the diverse types within your std::variant.

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    std::variant<int, std::string> data = 42; // Initialize variant with an int

    auto visitor = [](auto&& arg) -> std::string { 
        using T = std::decay_t<decltype(arg)>; // Get the underlying type of 'arg'

        if constexpr (std::is_same_v<T, int>) {
            return "It's an int: " + std::to_string(arg); // Convert int to string and return
        } 
        else if constexpr (std::is_same_v<T, std::string>) {
            return "It's a string: " + arg; // Concatenate and return
        } 
    };

    std::string result = std::visit(visitor, data); 
    std::cout << result << std::endl; // Output: It's an int: 42

    return 0;
}
```

Here, std::visit takes a "visitor" function (our lambda) and the std::variant. The visitor is then applied to the current value residing within the std::variant, allowing you to customize the behavior for each possible type.

### The Return Value of std::visit
The return value of std::visit is like a chameleon - it adapts to what your visitor function returns. Each "case" within your visitor (the if constexpr blocks) can return a different type. std::visit intelligently figures out the common type that all these cases could potentially return and uses that as its own return type.

##  applications of std::visit

**1. Implementing the Visitor Pattern and Static Polymorphism**
Instead of relying on inheritance, virtual base classes, and raw pointers, you can store different types of data within an std::variant. This allows you to define behavior via std::visit without the runtime overhead of vtable lookups associated with virtual functions. This is often referred to as “Static Polymorphism” because the dispatching logic is resolved based on the type at compile-time (or through type-switch generation).

**2. Event Handling and State Machines**
std::visit is excellent for processing events or transitions in a state machine. If you model various events (e.g., MouseClick, KeyPress, WindowResize) as members of a variant, std::visit allows you to cleanly route each event to its corresponding handler. This keeps your event-loop logic decoupled and type-safe.

**3. Interpreters, Compilers, and Abstract Syntax Trees (ASTs)**
This is perhaps the most common use case. When building an AST, your nodes are often different types (e.g., NumberNode, BinaryOpNode, VariableNode). By wrapping these in an std::variant, std::visit provides a concise way to traverse, evaluate, or optimize the tree, ensuring that you handle every possible node type defined in your grammar.

## Example

### Example 1: Using Generic Lambdas (The Simplest Case)
If the operation you want to perform is identical for all types held within the variant (for example, simply printing the value), you can use a generic lambda with auto.

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    std::variant<int, double, std::string> data = "Hello C++";

    // The generic lambda (auto) is compatible with any type held by the variant
    std::visit([](const auto& val) {
        std::cout << "Value: " << val << '\n';
    }, data);

    data = 42;
    std::visit([](const auto& val) {
        std::cout << "Value: " << val << '\n';
    }, data);
}

```
### Example 2: The Overloaded Pattern (Handling Types Differently)

There is a very popular pattern called overloaded that allows you to write a separate lambda for each specific type held by the variant.

```cpp
#include <iostream>
#include <variant>
#include <string>

// Helper structure to combine multiple lambdas into one object
template<class... Ts> struct overloaded : Ts... { using Ts::operator()...; };
template<class... Ts> overloaded(Ts...) -> overloaded<Ts...>; // Deduction guide for C++17

int main() {
    std::variant<int, double, std::string> data = 3.14;

    std::visit(overloaded {
        [](int arg) { std::cout << "Integer: " << arg << '\n'; },
        [](double arg) { std::cout << "Double: " << arg << '\n'; },
        [](const std::string& arg) { std::cout << "String: " << arg << '\n'; }
    }, data);
}

```
**The great advantage of this method:** If you add a new type to your variant but forget to handle it in your overloaded block, the code will not compile. This forces you to handle all cases at compile-time, which is much safer than if/else chains.

### Example 3: Multi-variant Dispatch
You can pass two or more variants to std::visit simultaneously. This is very useful for performing matrix-like operations or combining logic between multiple variant objects.

```cpp
#include <iostream>
#include <variant>

int main() {
    std::variant<int, double> a = 10;
    std::variant<int, double> b = 2.5;

    // Performing an operation on two variants simultaneously
    std::visit([](auto x, auto y) {
        std::cout << "Sum: " << (x + y) << '\n';
    }, a, b);
}
```
## Summary of Advantages
- Type Safety: Ensures full type safety, preventing runtime errors associated with invalid type access.
- High Performance: Typically implemented using compile-time jump tables, it avoids the overhead associated with virtual function calls (vtable) or dynamic heap allocations.
- Cleaner Code: Replaces complex and deeply nested switch or if-else blocks with a more readable and declarative structure.


---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>