<div align="center">

[🇺🇸 English](./std_invoke.md) | [🇮🇷 فارسی](../../fa/cpp17/std_invoke.md)

</div>

----

# `std::invoke` in C++17: Unified Invocation for All Callable Entities

In generic programming, template libraries must frequently invoke callable targets provided by callers. However, C++ has historically had disjoint and incompatible call syntaxes depending on what is being invoked: free functions and functors use `f(args...)`, pointers to member functions require `(obj.*ptr)(args...)` or `(ptr_to_obj->*ptr)(args...)`, and pointers to data members use `obj.*ptr` or `ptr_to_obj->*ptr`.

Prior to C++17, writing generic code that accepted *any* callable entity required cumbersome SFINAE overloads, wrapping targets in heavyweight abstractions like `std::function`, or relying on internal compiler-specific helpers.

C++17 standardized **`std::invoke`** in `<functional>`, exposing the ISO *INVOKE* pseudo-expression directly to library writers and developers. With `std::invoke`, any callable—free function, lambda, functor, member function pointer, or member data pointer—is invoked using one uniform syntax with zero runtime overhead.

```cpp
#include <iostream>
#include <functional>
#include <string>

struct Account {
    std::string owner;
    double balance;

    void deposit(double amount) { balance += amount; }
    double get_balance() const { return balance; }
};

void print_greeting(const std::string& name) {
    std::cout << "Hello, " << name << "!\n";
}

int main() {
    Account acc{"Alice", 1000.0};

    // 1. Invoking a free function
    std::invoke(print_greeting, acc.owner);

    // 2. Invoking a member function on an object reference
    std::invoke(&Account::deposit, acc, 250.0);

    // 3. Invoking a member function on a pointer
    std::cout << "Balance: " << std::invoke(&Account::get_balance, &acc) << '\n';

    // 4. Accessing a data member via pointer-to-member
    std::invoke(&Account::balance, acc) += 50.0;
    std::cout << "Updated Balance: " << acc.balance << '\n';
}
```

---

## Table of Contents

1. [Why `std::invoke` Exists](#1-why-stdinvoke-exists)
2. [What Was the Problem Before C++17?](#2-what-was-the-problem-before-c17)
3. [The Formal *INVOKE* Concept](#3-the-formal-invoke-concept)
4. [Syntax and Reference Implementation](#4-syntax-and-reference-implementation)
5. [Dissecting All 5 Callable Categories](#5-dissecting-all-5-callable-categories)
   - 5.1 [Free Functions, Lambdas, and Functors](#51-free-functions-lambdas-and-functors)
   - 5.2 [Member Function Pointers on References](#52-member-function-pointers-on-references)
   - 5.3 [Member Function Pointers on Pointers and Smart Pointers](#53-member-function-pointers-on-pointers-and-smart-pointers)
   - 5.4 [Member Data Pointers on References](#54-member-data-pointers-on-references)
   - 5.5 [Member Data Pointers on Pointers and Smart Pointers](#55-member-data-pointers-on-pointers-and-smart-pointers)
6. [Companion Type Traits: `std::invoke_result` & `std::is_invocable`](#6-companion-type-traits-stdinvoke_result--stdis_invocable)
7. [Real-World Architectural Patterns](#7-real-world-architectural-patterns)
   - 7.1 [Generic Thread Pools and Task Executors](#71-generic-thread-pools-and-task-executors)
   - 7.2 [Higher-Order Algorithms (`map`, `filter`, `pipeline`)](#72-higher-order-algorithms-map-filter-pipeline)
   - 7.3 [Transparent Event Busses and Observers](#73-transparent-event-busses-and-observers)
   - 7.4 [Performance Profilers and Method Interceptors](#74-performance-profilers-and-method-interceptors)
8. [Comparison: `std::invoke` vs. `std::function` vs. `std::bind`](#8-comparison-stdinvoke-vs-stdfunction-vs-stdbind)
9. [Common Pitfalls and Gotchas](#9-common-pitfalls-and-gotchas)
10. [Performance, Assembly Generation, and Zero-Cost Guarantees](#10-performance-assembly-generation-and-zero-cost-guarantees)
11. [Best Practices](#11-best-practices)
12. [Evolution: C++20 `std::invoke_r` and Beyond](#12-evolution-c20-stdinvoke_r-and-beyond)
13. [Feature-Test Macro and Compiler Support](#13-feature-test-macro-and-compiler-support)
14. [Quick Reference Card](#14-quick-reference-card)
15. [Complete Executable Example](#15-complete-executable-example)
16. [Exercises](#16-exercises)
17. [Conclusion](#17-conclusion)
18. [Contributors](#18-contributors)

---

## 1. Why `std::invoke` Exists

In modern generic C++, templates often receive a "callable" along with arguments. Consider standard library facilities like:
- `std::thread(f, args...)`
- `std::async(f, args...)`
- `std::reference_wrapper::operator()`
- `std::apply(f, tuple)`

Every one of these facilities needed to execute *any* valid C++ callable entity. Yet prior to C++17, the language definition used a theoretical pseudo-expression called *INVOKE(f, t1, t2, ..., tN)* inside the normative standard text, but gave developers no standard library function to call it directly.

`std::invoke` exposed this core language machinery to everyone.

---

## 2. What Was the Problem Before C++17?

### The Incompatible Syntax Divide

In generic templates, simple function call syntax fails for member pointers:

```cpp
template <typename Callable, typename... Args>
decltype(auto) execute(Callable&& func, Args&&... args) {
    // Problem: If func is a pointer-to-member-function or pointer-to-data-member,
    // func(...) is a syntax error!
    return std::forward<Callable>(func)(std::forward<Args>(args)...); 
}

struct Node {
    int value;
    int get_value() const { return value; }
};

int main() {
    Node n{42};
    execute([](int x) { return x * 2; }, 10); // OK

    // COMPILE ERRORS before C++17 without custom overloads:
    // execute(&Node::get_value, n); 
    // execute(&Node::value, n);
}
```

### The Pre-C++17 Workaround: Custom Metaprogramming

Before C++17, developers had to write SFINAE-heavy overloads (or private helpers like `std::__invoke`) to test whether `Callable` was a member pointer or a regular function, and whether the first argument was an object, a reference, a raw pointer, or a smart pointer.

---

## 3. The Formal *INVOKE* Concept

The ISO C++ standard specifies that `INVOKE(f, t1, t2, ..., tN)` evaluates according to the following ranked hierarchy:

1. **Member Function Pointer on Object/Reference:** If `f` is a pointer to a member function of class `T`, and `std::is_base_of_v<T, std::decay_t<decltype(t1)>>` is `true`:
   $$\text{INVOKE}(f, t_1, t_2, \dots, t_N) \equiv (t_1.*f)(t_2, \dots, t_N)$$
2. **Member Function Pointer on `std::reference_wrapper`:** If `t1` is a specialization of `std::reference_wrapper`:
   $$\text{INVOKE}(f, t_1, t_2, \dots, t_N) \equiv (t_1.\text{get}().*f)(t_2, \dots, t_N)$$
3. **Member Function Pointer on Pointer/Smart Pointer:** Otherwise (e.g., raw pointer, `std::unique_ptr`, `std::shared_ptr`):
   $$\text{INVOKE}(f, t_1, t_2, \dots, t_N) \equiv ((*t_1).*f)(t_2, \dots, t_N)$$
4. **Member Data Pointer on Object/Reference:** If $N = 1$, `f` is a pointer to member data of class `T`, and `t1` is of class `T` (or derived):
   $$\text{INVOKE}(f, t_1) \equiv t_1.*f$$
5. **Member Data Pointer on `std::reference_wrapper`:**
   $$\text{INVOKE}(f, t_1) \equiv t_1.\text{get}().*f$$
6. **Member Data Pointer on Pointer/Smart Pointer:**
   $$\text{INVOKE}(f, t_1) \equiv (*t_1).*f$$
7. **All Other Callables:** If none of the above match:
   $$\text{INVOKE}(f, t_1, t_2, \dots, t_N) \equiv f(t_1, t_2, \dots, t_N)$$

---

## 4. Syntax and Reference Implementation

Defined in header `<functional>`:

```cpp
namespace std {
    template <class F, class... Args>
    constexpr std::invoke_result_t<F, Args...> invoke(F&& f, Args&&... args) 
        noexcept(std::is_nothrow_invocable_v<F, Args...>);
}
```

### Conceptual Reference Implementation (C++17 with `if constexpr`):

```cpp
#include <type_traits>
#include <utility>
#include <functional>

namespace detail {
    template <class Base, class T, class Derived, class... Args>
    constexpr decltype(auto) invoke_mempfun(T Base::* pmf, Derived&& ref, Args&&... args) {
        if constexpr (std::is_base_of_v<Base, std::decay_t<Derived>>) {
            return (std::forward<Derived>(ref).*pmf)(std::forward<Args>(args)...);
        } else if constexpr (requires { ref.get(); }) { // e.g. reference_wrapper
            return (ref.get().*pmf)(std::forward<Args>(args)...);
        } else {
            return ((*std::forward<Derived>(ref)).*pmf)(std::forward<Args>(args)...);
        }
    }

    template <class Base, class T, class Derived>
    constexpr decltype(auto) invoke_memdata(T Base::* pmd, Derived&& ref) {
        if constexpr (std::is_base_of_v<Base, std::decay_t<Derived>>) {
            return std::forward<Derived>(ref).*pmd;
        } else if constexpr (requires { ref.get(); }) {
            return ref.get().*pmd;
        } else {
            return (*std::forward<Derived>(ref)).*pmd;
        }
    }
}

template <class F, class... Args>
constexpr decltype(auto) custom_invoke(F&& f, Args&&... args) {
    if constexpr (std::is_member_function_pointer_v<std::decay_t<F>>) {
        return detail::invoke_mempfun(f, std::forward<Args>(args)...);
    } else if constexpr (std::is_member_object_pointer_v<std::decay_t<F>>) {
        return detail::invoke_memdata(f, std::forward<Args>(args)...);
    } else {
        return std::forward<F>(f)(std::forward<Args>(args)...);
    }
}
```

---

## 5. Dissecting All 5 Callable Categories

### 5.1 Free Functions, Lambdas, and Functors

Direct execution via regular `()` operator:

```cpp
auto lambda = [](int a, int b) { return a + b; };
int res = std::invoke(lambda, 10, 20); // res = 30
```

### 5.2 Member Function Pointers on References

Invokes member function directly on the target instance or reference:

```cpp
struct Worker {
    int compute(int x) { return x * 10; }
};

Worker w;
int val = std::invoke(&Worker::compute, w, 5); // w.compute(5) -> 50
```

### 5.3 Member Function Pointers on Pointers and Smart Pointers

Dereferences the pointer automatically before invoking the member function:

```cpp
Worker* raw_ptr = &w;
int val1 = std::invoke(&Worker::compute, raw_ptr, 5); // (raw_ptr->compute)(5)

auto smart_ptr = std::make_unique<Worker>();
int val2 = std::invoke(&Worker::compute, smart_ptr, 5); // ((*smart_ptr).compute)(5)
```

### 5.4 Member Data Pointers on References

Accesses and returns a reference to the member variable:

```cpp
struct Config {
    int port;
};

Config cfg{8080};
// Returns int& (lvalue reference), so you can read AND write!
std::invoke(&Config::port, cfg) = 9090; 
std::cout << cfg.port; // 9090
```

### 5.5 Member Data Pointers on Pointers and Smart Pointers

Dereferences the target pointer and accesses the member variable:

```cpp
auto smart_cfg = std::make_shared<Config>(Config{443});
int p = std::invoke(&Config::port, smart_cfg); // p = 443
```

---

## 6. Companion Type Traits: `std::invoke_result` & `std::is_invocable`

C++17 deprecated `std::result_of` and replaced it with a unified suite of invocation traits matching `std::invoke`:

### Querying Return Types: `std::invoke_result`

```cpp
#include <type_traits>

struct Parser {
    bool parse(const std::string&);
};

// Obtains the return type of invoking &Parser::parse with (Parser&, std::string)
using RetType = std::invoke_result_t<decltype(&Parser::parse), Parser&, std::string>;
static_assert(std::is_same_v<RetType, bool>);
```

### Constraining Templates with Invocability Checks

```cpp
// Check if callable with specific argument types
static_assert(std::is_invocable_v<decltype(&Parser::parse), Parser*, std::string>);

// Check if callable AND the return type converts to a given type (e.g. bool)
static_assert(std::is_invocable_r_v<bool, decltype(&Parser::parse), Parser*, std::string>);

// Check for noexcept invocability
static_assert(!std::is_nothrow_invocable_v<decltype(&Parser::parse), Parser*, std::string>);
```

---

## 7. Real-World Architectural Patterns

### 7.1 Generic Thread Pools and Task Executors

A robust task runner accepts functions, lambdas, or member pointers with arbitrary arguments:

```cpp
#include <functional>
#include <future>
#include <memory>
#include <utility>

class TaskQueue {
public:
    template <typename F, typename... Args>
    auto submit(F&& f, Args&&... args) {
        using ReturnType = std::invoke_result_t<F, Args...>;

        auto task = std::make_shared<std::packaged_task<ReturnType()>>(
            [func = std::forward<F>(f), 
             tup = std::make_tuple(std::forward<Args>(args)...)]() mutable {
                return std::apply([&func](auto&&... unpacked) {
                    return std::invoke(func, std::forward<decltype(unpacked)>(unpacked)...);
                }, tup);
            }
        );

        std::future<ReturnType> fut = task->get_future();
        // Enqueue task for background worker...
        (*task)(); 
        return fut;
    }
};
```

### 7.2 Higher-Order Algorithms (`map`, `filter`, `pipeline`)

Creating algorithms that accept any callable extractor:

```cpp
#include <vector>
#include <algorithm>
#include <functional>

template <typename Range, typename Projection>
auto project(const Range& items, Projection proj) {
    using ResultType = std::decay_t<std::invoke_result_t<Projection, decltype(*std::begin(items))>>;
    std::vector<ResultType> results;
    results.reserve(std::size(items));

    for (const auto& item : items) {
        results.push_back(std::invoke(proj, item));
    }
    return results;
}

struct Employee {
    int id;
    std::string name;
};

// Usage:
std::vector<Employee> team{{1, "John"}, {2, "Jane"}};
auto names = project(team, &Employee::name);     // Uses member data pointer
auto ids   = project(team, [](const auto& e) { return e.id; }); // Uses lambda
```

### 7.3 Transparent Event Busses and Observers

```cpp
#include <vector>
#include <functional>

template <typename EventType>
class EventBus {
public:
    template <typename Callable>
    void subscribe(Callable&& cb) {
        listeners_.emplace_back(std::forward<Callable>(cb));
    }

    void publish(const EventType& ev) {
        for (auto& fn : listeners_) {
            std::invoke(fn, ev);
        }
    }

private:
    std::vector<std::function<void(const EventType&)>> listeners_;
};
```

### 7.4 Performance Profilers and Method Interceptors

```cpp
#include <chrono>
#include <iostream>
#include <functional>

template <typename F, typename... Args>
decltype(auto) profile(const char* label, F&& f, Args&&... args) {
    auto start = std::chrono::steady_clock::now();
    
    // std::invoke ensures uniform invocation regardless of target type
    decltype(auto) result = std::invoke(std::forward<F>(f), std::forward<Args>(args)...);

    auto end = std::chrono::steady_clock::now();
    std::cout << "[" << label << "] Elapsed: " 
              << std::chrono::duration_cast<std::chrono::microseconds>(end - start).count() 
              << " us\n";

    return result;
}
```

---

## 8. Comparison: `std::invoke` vs. `std::function` vs. `std::bind`

| Feature | `std::invoke` (C++17) | `std::function` (C++11) | `std::bind` (C++11) |
|---|---|---|---|
| **Mechanism** | Compile-time function template | Type-erased runtime object wrapper | Partial application binder object |
| **Allocation Cost** | Zero (no heap memory) | Potential heap allocation (if target > SBO) | Zero (stores arguments in object) |
| **Virtual Call Cost**| Zero (fully inlinable) | Indirect call through vtable/function pointer | Zero (templated `operator()`) |
| **Callable Form** | Direct invocation primitive | Stores and delays invocation | Binds args, returns callable object |
| **Modern Recommendation** | **Primary choice** for generic templates | Use when runtime type erasure is required | **Avoid**; prefer lambdas |

---

## 9. Common Pitfalls and Gotchas

### 9.1 Ambiguity with Overloaded Functions

Passing an overloaded function name directly to `std::invoke` causes deduction failure:

```cpp
void logMsg(int code);
void logMsg(const std::string& msg);

int main() {
    // COMPILE ERROR: Cannot deduce template arguments for overloaded function
    // std::invoke(logMsg, 404);

    // FIX 1: Explicit cast to specific signature
    std::invoke(static_cast<void(*)(int)>(logMsg), 404);

    // FIX 2: Wrap in a generic lambda (Best practice)
    std::invoke([](auto&&... args) { logMsg(std::forward<decltype(args)>(args)...); }, 404);
}
```

### 9.2 Member Data Pointer Returns Lvalue Reference

When calling `std::invoke` on a member data pointer with an lvalue object, the return type is an **lvalue reference** to the member, not a copy:

```cpp
struct Sensor { int reading; };

Sensor s{10};
decltype(auto) val = std::invoke(&Sensor::reading, s); // val is int&
val = 99; // Modifies s.reading directly!
```
*Note:* If you intend to take a copy, use `auto val = ...` rather than `decltype(auto)`.

### 9.3 Invocability on Temporary Rvalues

Accessing member functions on rvalue objects requires checking ref-qualifiers:

```cpp
struct Resource {
    void process() & { std::cout << "lvalue\n"; }
    void process() && { std::cout << "rvalue\n"; }
};

int main() {
    std::invoke(&Resource::process, Resource{}); // Calls rvalue overload: process() &&
}
```

---

## 10. Performance, Assembly Generation, and Zero-Cost Guarantees

- **Full Inlining:** `std::invoke` introduces no function pointer jumps or dynamic dispatches. Under optimizations (`-O2` / `-O3`), compilers inline the internal dispatches completely.
- **Assembly Parity:**
  ```cpp
  // Hand-written member access:
  int a = obj.value;

  // std::invoke member access:
  int b = std::invoke(&MyClass::value, obj);
  ```
  Both generate the **exact same machine instruction** (e.g., `mov eax, DWORD PTR [rdi + offset]`).

---

## 11. Best Practices

1. **Use `std::invoke` inside generic templates:** Whenever accepting a template parameter `Callable&& f` with `Args&&... args`, use `std::invoke(std::forward<Callable>(f), std::forward<Args>(args)...)` instead of `f(args...)`.
2. **Pair with `std::invoke_result_t`:** Use `std::invoke_result_t<F, Args...>` or `decltype(auto)` to preserve precise value categories (references and constness).
3. **Guard with `std::is_invocable_v`:** Constrain templates using SFINAE (`std::enable_if_t`) or C++20 concepts (`std::invocable`) to provide clear compiler error diagnostics.
4. **Prefer generic lambdas over `std::bind`:** If you need to bind arguments before invoking, prefer C++14 generic lambdas over `std::bind`.

---

## 12. Evolution: C++20 `std::invoke_r` and Beyond

### C++20: `std::invoke_r` (P1065R4 / P2136R3)
C++20 added `std::invoke_r<R>(f, args...)` in header `<functional>`, which invokes the callable and explicitly converts the result to return type `R` (including `void`):

```cpp
// C++20: Force explicit conversion to double
double d = std::invoke_r<double>([](int a) { return a * 2; }, 5); // 10.0

// C++20: Explicitly discard non-void return type cleanly
std::invoke_r<void>([](int a) { return a * 2; }, 5);
```

### C++20: Concepts Integration
C++20 introduced the `<concepts>` header with standard concepts matching `std::invoke`:
- `std::invocable<F, Args...>`
- `std::regular_invocable<F, Args...>`
- `std::predicate<F, Args...>`

---

## 13. Feature-Test Macro and Compiler Support

Header: `<functional>`

```cpp
#if defined(__cpp_lib_invoke) && __cpp_lib_invoke >= 201411L
    // std::invoke is supported
#endif
```

### Compiler Support Table

| Compiler | Version | Standard Flag |
|---|---|---|
| **GCC** | 7.1+ | `-std=c++17` |
| **Clang** | 4.0+ | `-std=c++17` |
| **MSVC** | 2017 (15.3+) | `/std:c++17` |
| **Apple Clang** | 9.0+ | `-std=c++17` |

---

## 14. Quick Reference Card

```cpp
#include <functional>

struct S {
    int data = 100;
    int method(int x) const { return data + x; }
};

S s;
S* ptr = &s;

// 1. Regular functions / lambdas
std::invoke([](int x) { return x * 2; }, 10);        // 20

// 2. Member function with reference
std::invoke(&S::method, s, 5);                       // s.method(5) -> 105

// 3. Member function with pointer
std::invoke(&S::method, ptr, 10);                    // ptr->method(10) -> 110

// 4. Member data pointer (Read)
int val = std::invoke(&S::data, s);                  // s.data -> 100

// 5. Member data pointer (Write)
std::invoke(&S::data, s) = 200;                      // s.data = 200

// 6. Member data pointer with pointer
int val2 = std::invoke(&S::data, ptr);               // ptr->data -> 200
```

---

## 15. Complete Executable Example

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <functional>
#include <memory>

// Domain Entity
class DeviceSession {
public:
    DeviceSession(std::string id, int battery_pct)
        : device_id_(std::move(id)), battery_level_(battery_pct) {}

    // Member function
    void ping(const std::string& server) const {
        std::cout << "[Device " << device_id_ << "] Pinging " << server 
                  << " (Battery: " << battery_level_ << "%)\n";
    }

    // Member variable
    int battery_level_;

private:
    std::string device_id_;
};

// Generic Dispatcher supporting any callable target
template <typename Target, typename Callable, typename... Args>
void dispatchAction(Target&& target, Callable&& action, Args&&... args) {
    std::cout << "-> Dispatching through std::invoke...\n";
    std::invoke(std::forward<Callable>(action), 
                std::forward<Target>(target), 
                std::forward<Args>(args)...);
}

int main() {
    std::cout << "=== Demonstrating std::invoke in C++17 ===\n\n";

    DeviceSession dev{"DEV-9801", 85};
    auto smart_dev = std::make_unique<DeviceSession>("DEV-SMART-02", 92);

    // 1. Member function invocation via reference
    std::cout << "Test 1: Member Function with Object Reference\n";
    dispatchAction(dev, &DeviceSession::ping, "192.168.1.1");

    // 2. Member function invocation via smart pointer
    std::cout << "\nTest 2: Member Function with std::unique_ptr\n";
    dispatchAction(smart_dev, &DeviceSession::ping, "gateway.iot.internal");

    // 3. Member data pointer read and write
    std::cout << "\nTest 3: Reading and Mutating Member Data via std::invoke\n";
    std::cout << "Current battery: " << std::invoke(&DeviceSession::battery_level_, dev) << "%\n";
    
    // Mutate member directly using returned lvalue reference
    std::invoke(&DeviceSession::battery_level_, dev) = 100;
    std::cout << "Fully charged:   " << std::invoke(&DeviceSession::battery_level_, dev) << "%\n";

    // 4. Invoking lambda
    std::cout << "\nTest 4: Generic Lambda Invocation\n";
    auto logger = [](const DeviceSession& d, const std::string& msg) {
        std::cout << "[Custom Logger] Battery remaining: " << d.battery_level_ << "% | Msg: " << msg << '\n';
    };
    dispatchAction(dev, logger, "Telemetry routine passed");

    return 0;
}
```

### Expected Output

```text
=== Demonstrating std::invoke in C++17 ===

Test 1: Member Function with Object Reference
-> Dispatching through std::invoke...
[Device DEV-9801] Pinging 192.168.1.1 (Battery: 85%)

Test 2: Member Function with std::unique_ptr
-> Dispatching through std::invoke...
[Device DEV-SMART-02] Pinging gateway.iot.internal (Battery: 92%)

Test 3: Reading and Mutating Member Data via std::invoke
Current battery: 85%
Fully charged:   100%

Test 4: Generic Lambda Invocation
-> Dispatching through std::invoke...
[Custom Logger] Battery remaining: 100% | Msg: Telemetry routine passed
```

---

## 16. Exercises

### Exercise 1 — Implementing Generic Callback Runner
Write a template class `CallbackManager` that stores a callable entity and allows executing it with arguments provided via a `run(...)` method. Ensure it supports both free functions and member function pointers with an object reference.

### Exercise 2 — Transparent Map-Filter Container
Write a generic algorithm `transform_if` taking an input vector, a predicate, and a projection callable. Use `std::invoke` for both the predicate and projection so callers can pass member function pointers or member data pointers directly (e.g., extracting an attribute where an object's boolean flag is true).

### Exercise 3 — Handling Reference Wrappers
Instantiate an object `struct Counter { int count = 0; void inc() { ++count; } };`. Create a `std::reference_wrapper<Counter> ref_wrap = std::ref(c);`. Verify that `std::invoke(&Counter::inc, ref_wrap);` increments `c.count`.

### Exercise 4 — Compile-Time Invocability Verification
Write a template function `safe_call(Callable&& f, Args&&... args)` that uses `if constexpr` with `std::is_invocable_v<Callable, Args...>` to invoke the function if valid, or prints an error message and returns a default value if not valid.

---

## 17. Conclusion

`std::invoke` is a foundational component of modern C++ generic engineering:
- It **bridges the historical syntactical gap** between function objects, member functions, and data members.
- It provides a **uniform invocation primitive** with **zero runtime overhead**.
- Together with `std::invoke_result` and `std::is_invocable`, it allows writing generic libraries that are flexible, expressive, and robust.

---

## 18. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |[Ordikhani](ordikhanifateme@gmail.com) |  | [Ordikhani](@OrdikhaniFateme) |