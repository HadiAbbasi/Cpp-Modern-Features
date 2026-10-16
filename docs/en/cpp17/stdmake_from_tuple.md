<div align="center">

[🇺🇸 English](./std_make_from_tuple.md) | [🇮🇷 فارسی](../../fa/cpp17/std_make_from_tuple.md)

</div>

----

# `std::make_from_tuple` in C++17: Constructing Objects from Tuple-Like Containers

Template metaprogramming and generic library development frequently require unpacking heterogeneously typed collections into callable targets or constructors. While C++17 introduced `std::apply` to unpack a tuple into a standard callable function or functor, invoking an object's **constructor** with tuple elements remained cumbersome, requiring explicit index sequences (`std::index_sequence`) and manual pack expansions.

To complete this reflection and unpacking toolkit, C++17 introduced **`std::make_from_tuple`** in `<tuple>`. It allows you to construct any type `T` using the elements of a tuple-like container as arguments to its matching constructor, eliminating handwritten index sequence boilerplate.

```cpp
#include <iostream>
#include <tuple>
#include <string>

struct ConnectionConfig {
    std::string host;
    int port;
    bool use_tls;

    ConnectionConfig(std::string h, int p, bool tls)
        : host(std::move(h)), port(p), use_tls(tls) {}
};

int main() {
    auto params = std::make_tuple("127.0.0.1", 5432, true);

    // C++17: Directly instantiate ConnectionConfig from tuple elements
    ConnectionConfig cfg = std::make_from_tuple<ConnectionConfig>(std::move(params));

    std::cout << "Connecting to " << cfg.host << ":" << cfg.port
              << " [TLS: " << (cfg.use_tls ? "enabled" : "disabled") << "]\n";
}
```

---

## Table of Contents

1. [Why `std::make_from_tuple` Exists](#1-why-stdmake_from_tuple-exists)
2. [What Was the Problem Before C++17?](#2-what-was-the-problem-before-c17)
3. [Formal Declaration and Specification](#3-formal-declaration-and-specification)
4. [How It Works: Reference Implementation](#4-how-it-works-reference-implementation)
5. [`std::apply` vs. `std::make_from_tuple`](#5-stdapply-vs-stdmake_from_tuple)
6. [Supported "Tuple-Like" Types](#6-supported-tuple-like-types)
7. [Value Categories and Perfect Forwarding](#7-value-categories-and-perfect-forwarding)
8. [Real-World Architectural Patterns](#8-real-world-architectural-patterns)
   - 8.1 [Generic Factory and Object Pooling](#81-generic-factory-and-object-pooling)
   - 8.2 [Deserialization and Data Ingestion Pipelines](#82-deserialization-and-data-ingestion-pipelines)
   - 8.3 [Delayed Task Execution and Thread Pools](#83-delayed-task-execution-and-thread-pools)
   - 8.4 [Dependency Injection Containers](#84-dependency-injection-containers)
9. [Common Pitfalls and Gotchas](#9-common-pitfalls-and-gotchas)
10. [Performance, Inlining, and Zero-Cost Guarantees](#10-performance-inlining-and-zero-cost-guarantees)
11. [Best Practices](#11-best-practices)
12. [Evolution: C++20 and Aggregate Initialization](#12-evolution-c20-and-aggregate-initialization)
13. [Feature-Test Macro and Compiler Support](#13-feature-test-macro-and-compiler-support)
14. [Quick Reference Card](#14-quick-reference-card)
15. [Complete Executable Example](#15-complete-executable-example)
16. [Exercises](#16-exercises)
17. [Conclusion](#17-conclusion)
18. [Contributors](#18-contributors)

---

## 1. Why `std::make_from_tuple` Exists

In modern C++, components often separate **argument storage** from **object instantiation**. Examples include:
- Deserializing parameters from a network payload into a tuple, then instantiating a target domain model.
- Queuing asynchronous construction tasks in worker thread pools.
- Generic factory classes storing cached argument sets.

While `std::apply(fn, tup)` unpacks a tuple to invoke a function, a constructor is **not** a callable function pointer. You cannot write `std::apply(T::T, tup)`. 

Before C++17, every engineer or library had to reinvent an `index_sequence` helper function to expand the tuple elements into `T(std::get<I>(t)...)`. `std::make_from_tuple` standardizes this idiom into a single standard library primitive.

---

## 2. What Was the Problem Before C++17?

### The C++11/14 Idiom: The `index_sequence` Boilerplate

In C++14, constructing an object from a tuple required writing a private detail namespace with helper functions using `std::index_sequence`:

```cpp
#include <tuple>
#include <utility>

namespace detail {
    template <typename T, typename Tuple, std::size_t... Is>
    constexpr T make_from_tuple_impl(Tuple&& tup, std::index_sequence<Is...>) {
        // Direct initialization via pack expansion
        return T(std::get<Is>(std::forward<Tuple>(tup))...);
    }
}

template <typename T, typename Tuple>
constexpr T legacy_make_from_tuple(Tuple&& tup) {
    return detail::make_from_tuple_impl<T>(
        std::forward<Tuple>(tup),
        std::make_index_sequence<std::tuple_size_v<std::decay_t<Tuple>>>{}
    );
}
```

This pattern resulted in:
1. **Redundant code replication** across libraries and projects.
2. **Subtle perfect forwarding bugs**, such as forgetting `std::forward<Tuple>` inside `std::get<Is>`.
3. **Cluttered call stacks** and longer compilation times during diagnostic traces.

---

## 3. Formal Declaration and Specification

Defined in header `<tuple>`:

```cpp
namespace std {
    template <class T, class Tuple>
    constexpr T make_from_tuple(Tuple&& t);
}
```

### Constraints and Semantics:
- **Direct Initialization:** Constructs an object of type `T` using direct-initialization syntax: `T(std::get<I>(std::forward<Tuple>(t))...)`.
- **Return Value:** Returns the constructed instance of `T`. With C++17 Guaranteed Copy Elision, if the return expression is a prvalue, no temporary copy or move is performed.
- **`constexpr` Support:** Can be evaluated at compile time if both `std::get` on the tuple and `T`'s constructor are `constexpr`-friendly.

---

## 4. How It Works: Reference Implementation

The internal specification in ISO C++17 ([tuple.apply]) is conceptualized as:

```cpp
namespace std {
    namespace detail {
        template <class T, class Tuple, std::size_t... I>
        constexpr T make_from_tuple_impl(Tuple&& t, std::index_sequence<I...>) {
            return T(std::get<I>(std::forward<Tuple>(t))...);
        }
    }

    template <class T, class Tuple>
    constexpr T make_from_tuple(Tuple&& t) {
        return detail::make_from_tuple_impl<T>(
            std::forward<Tuple>(t),
            std::make_index_sequence<std::tuple_size_v<std::remove_reference_t<Tuple>>>{}
        );
    }
}
```

Because it uses `std::forward<Tuple>(t)`, whether `std::get<I>` yields an lvalue reference or an rvalue reference depends entirely on whether `t` was passed as an lvalue or an rvalue.

---

## 5. `std::apply` vs. `std::make_from_tuple`

Both utilities unpack tuples, but they target different language mechanisms:

| Feature | `std::apply` | `std::make_from_tuple` |
|---|---|---|
| **Target** | Callable entity (`Callable`: lambda, function, functor, member pointer) | Type to construct (`T`) |
| **Invocation Syntax** | `std::invoke(std::forward<F>(f), std::get<I>(t)...)` | `T(std::get<I>(t)...)` |
| **Return Type** | Deduced return type of calling `f(...)` | Exactly specified type `T` |
| **Type Specification** | `std::apply(fn, tuple)` (Target deduced) | `std::make_from_tuple<T>(tuple)` (Target explicit) |
| **Works with Constructors?** | No (constructors are not callable objects) | Yes (specifically designed for them) |

---

## 6. Supported "Tuple-Like" Types

`std::make_from_tuple` does not just accept `std::tuple`. Any type supporting the standard **Tuple Protocol** works seamlessly:
1. `std::tuple_size<std::remove_cvref_t<Tuple>>::value`
2. `std::tuple_element<I, std::remove_cvref_t<Tuple>>::type`
3. `std::get<I>(t)` available via ADL or standard namespace.

### Examples of Valid Tuple-Like Sources:

```cpp
#include <tuple>
#include <utility>
#include <array>

struct Coordinate {
    int x, y;
    Coordinate(int a, int b) : x(a), y(b) {}
};

// 1. std::pair
std::pair<int, int> p{10, 20};
auto c1 = std::make_from_tuple<Coordinate>(p);

// 2. std::array
std::array<int, 2> arr{30, 40};
auto c2 = std::make_from_tuple<Coordinate>(arr);

// 3. std::tuple
std::tuple<int, int> tup{50, 60};
auto c3 = std::make_from_tuple<Coordinate>(tup);
```

---

## 7. Value Categories and Perfect Forwarding

`std::make_from_tuple` uses perfect forwarding on the tuple argument:

```cpp
#include <memory>
#include <tuple>

struct Session {
    std::unique_ptr<int> ptr;
    std::string name;

    Session(std::unique_ptr<int> p, std::string n)
        : ptr(std::move(p)), name(std::move(n)) {}
};

int main() {
    auto args = std::make_tuple(std::make_unique<int>(42), std::string("Worker-1"));

    // PASSING AS RVALUE (std::move):
    // std::get<0>(std::move(args)) yields std::unique_ptr<int>&& (rvalue reference)
    // Moving move-only types works seamlessly!
    Session s = std::make_from_tuple<Session>(std::move(args));
}
```

- If passed an **lvalue tuple** (`t`), `std::get<I>(t)` yields an lvalue reference (elements are copied).
- If passed an **rvalue tuple** (`std::move(t)`), `std::get<I>(std::move(t))` yields an rvalue reference (elements are moved).

---

## 8. Real-World Architectural Patterns

### 8.1 Generic Factory and Object Pooling

When writing an object factory that caches creation parameters or delays instantiation:

```cpp
#include <memory>
#include <tuple>

template <typename Base, typename Derived, typename... Args>
class ComponentFactory {
public:
    explicit ComponentFactory(Args... args) 
        : cached_args_(std::make_tuple(std::move(args)...)) {}

    std::unique_ptr<Base> create() const {
        // Instantiate Derived using copy of cached arguments
        return std::make_unique<Derived>(
            std::make_from_tuple<Derived>(cached_args_)
        );
    }

private:
    std::tuple<Args...> cached_args_;
};
```

### 8.2 Deserialization and Data Ingestion Pipelines

When processing delimited records or binary streams:

```cpp
#include <iostream>
#include <tuple>
#include <string>

struct UserRecord {
    uint64_t id;
    std::string username;
    double balance;

    UserRecord(uint64_t i, std::string u, double b)
        : id(i), username(std::move(u)), balance(b) {}
};

// Generic pipeline step
template <typename T, typename TupleSource>
T parseAndBuild(TupleSource&& raw_tuple) {
    // Perform validation, sanitization, then construct
    return std::make_from_tuple<T>(std::forward<TupleSource>(raw_tuple));
}
```

### 8.3 Delayed Task Execution and Thread Pools

Decoupling the capture of task constructor arguments from execution time:

```cpp
#include <tuple>
#include <functional>

struct DatabaseWorker {
    DatabaseWorker(std::string conn_str, int max_retries, int timeout_sec);
    void run();
};

template <typename WorkerType, typename... Args>
auto prepareWorkerTask(Args&&... args) {
    // Capture arguments into tuple
    auto stored_args = std::make_tuple(std::forward<Args>(args)...);

    // Return lazy deferred callable
    return [params = std::move(stored_args)]() mutable {
        auto worker = std::make_from_tuple<WorkerType>(std::move(params));
        worker.run();
    };
}
```

### 8.4 Dependency Injection Containers

Dependency injection frameworks use tuples to resolve registered dependency signatures and inject them directly into service constructors:

```cpp
template <typename Service, typename... Dependencies>
struct ServiceDescriptor {
    std::tuple<Dependencies...> resolved_deps;

    Service createService() {
        return std::make_from_tuple<Service>(std::move(resolved_deps));
    }
};
```

---

## 9. Common Pitfalls and Gotchas

### 9.1 C++17 Aggregate Initialization Limitation

In **C++17**, `std::make_from_tuple` performs direct-initialization (`T(args...)`). Therefore, it **cannot** initialize C++17 aggregates that lack explicit constructors!

```cpp
// Plain Aggregate (No user-defined constructor)
struct Point {
    int x;
    int y;
};

int main() {
    auto args = std::make_tuple(10, 20);

    // COMPILE ERROR in C++17!
    // Point has no constructor Point(int, int); it is an aggregate.
    // auto pt = std::make_from_tuple<Point>(args); 
}
```

> **Note on C++20:** C++20 introduced parenthesized initialization of aggregates (`P0960R3`), making `std::make_from_tuple` work with aggregates out-of-the-box. In C++17, you must define an explicit constructor `Point(int x, int y) : x(x), y(y) {}` for this to compile.

### 9.2 Implicit Conversions and Ambiguous Overloads

If `T` possesses overloaded constructors, `std::make_from_tuple` follows standard overload resolution. Ambiguity will result in compiler errors:

```cpp
struct Ambiguous {
    Ambiguous(int, double);
    Ambiguous(double, int);
};

auto t = std::make_tuple(5, 5);
// Error: Ambiguous call (int vs double conversions are equivalent)
// auto obj = std::make_from_tuple<Ambiguous>(t);
```

### 9.3 Explicit Constructors

`std::make_from_tuple` uses direct-initialization (`T(...)`), which means **`explicit` constructors are fully supported and callable**:

```cpp
struct SafeID {
    explicit SafeID(int raw_id) : id(raw_id) {}
    int id;
};

auto t = std::make_tuple(1001);
// Works perfectly because direct-initialization considers explicit constructors
SafeID safe = std::make_from_tuple<SafeID>(t); 
```

---

## 10. Performance, Inlining, and Zero-Cost Guarantees

- **Zero Runtime Overhead:** Modern optimizing compilers (GCC, Clang, MSVC) inline the helper functions (`make_from_tuple_impl`) completely.
- **Assembly Generation:** The compiler optimizes `std::make_from_tuple` directly into the exact same machine code as writing `T(arg0, arg1, ...)`.
- **Guaranteed Copy Elision (C++17):** Returning the object constructed by `std::make_from_tuple<T>(...)` performs direct initialization into the destination storage without invoking copy or move constructors.

---

## 11. Best Practices

1. **Move rvalue tuples:** Always use `std::move(tuple)` when the tuple contains move-only types (`std::unique_ptr`, `std::thread`, sockets) or large containers to avoid accidental copies.
2. **Prefer over manual index sequences:** Replace any custom `index_sequence` constructor helpers with `std::make_from_tuple`.
3. **Beware of aggregates in C++17:** Add a user-defined constructor to aggregate structs if targeting C++17, or verify support in C++20.
4. **Use `constexpr` where possible:** Leverage `std::make_from_tuple` inside `constexpr` factory contexts for compile-time configuration validation.

---

## 12. Evolution: C++20 and Beyond

### 1. Aggregate Initialization (P0960R3)
Starting in **C++20**, parenthesized initialization `T(arg1, arg2)` is valid for aggregate types. Consequently, in C++20:
```cpp
struct Aggregate { int a; double b; };
// Perfectly valid in C++20!
auto agg = std::make_from_tuple<Aggregate>(std::make_tuple(1, 2.5));
```

### 2. Expanded Tuple-Like Concepts
C++20/C++23 integrate modern concepts (`std::ranges::subrange`, structured binding types), expanding the ecosystem of objects that can be unpacked with `std::make_from_tuple`.

---

## 13. Feature-Test Macro and Compiler Support

Header: `<tuple>`

```cpp
#if defined(__cpp_lib_make_from_tuple) && __cpp_lib_make_from_tuple >= 201606L
    // std::make_from_tuple is available
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
#include <tuple>

// Target Class
struct Widget {
    Widget(int a, double b, std::string c);
};

// 1. Unpacking std::tuple
auto t1 = std::make_tuple(1, 2.0, "three");
auto w1 = std::make_from_tuple<Widget>(t1);

// 2. Unpacking with Moves (Zero-copy)
auto w2 = std::make_from_tuple<Widget>(std::move(t1));

// 3. Unpacking std::pair
struct Point2D { Point2D(int x, int y); };
std::pair<int, int> coords{10, 20};
auto p = std::make_from_tuple<Point2D>(coords);

// 4. Unpacking std::array
std::array<int, 2> arr{100, 200};
auto p2 = std::make_from_tuple<Point2D>(arr);
```

---

## 15. Complete Executable Example

```cpp
#include <iostream>
#include <string>
#include <tuple>
#include <memory>
#include <vector>

// Domain Entity
class DatabaseSession {
public:
    DatabaseSession(std::string host, int port, std::string user, bool ssl)
        : host_(std::move(host)), port_(port), user_(std::move(user)), ssl_(ssl) {
        std::cout << "[Init] DatabaseSession established -> " 
                  << user_ << "@" << host_ << ":" << port_ 
                  << " [SSL: " << std::boolalpha << ssl_ << "]\n";
    }

    void query(const std::string& sql) const {
        std::cout << "[Query] " << user_ << "@" << host_ << " -> " << sql << '\n';
    }

private:
    std::string host_;
    int port_;
    std::string user_;
    bool ssl_;
};

// Generic Factory Function
template <typename Target, typename ArgTuple>
Target createService(ArgTuple&& args) {
    return std::make_from_tuple<Target>(std::forward<ArgTuple>(args));
}

int main() {
    std::cout << "=== Demonstrating std::make_from_tuple in C++17 ===\n\n";

    // 1. Pack construction arguments into a tuple
    auto session_params = std::make_tuple(
        std::string("cluster-node-01.internal"),
        5432,
        std::string("admin_svc"),
        true
    );

    // 2. Construct session using make_from_tuple
    std::cout << "Step 1: Constructing session directly from tuple...\n";
    DatabaseSession session = std::make_from_tuple<DatabaseSession>(session_params);
    session.query("SELECT * FROM audit_logs;");

    // 3. Using generic factory wrapper with perfect forwarding
    std::cout << "\nStep 2: Generic factory construction...\n";
    auto session2 = createService<DatabaseSession>(
        std::make_tuple("read-replica.internal", 5433, "readonly_usr", false)
    );
    session2.query("SELECT count(*) FROM metrics;");

    // 4. Using std::pair with make_from_tuple
    std::cout << "\nStep 3: Pair to 2D Dimension Object...\n";
    struct Dimension {
        int width;
        int height;
        Dimension(int w, int h) : width(w), height(h) {}
    };

    std::pair<int, int> resolution{1920, 1080};
    Dimension dim = std::make_from_tuple<Dimension>(resolution);
    std::cout << "Resolution: " << dim.width << "x" << dim.height << '\n';

    return 0;
}
```

### Expected Output

```text
=== Demonstrating std::make_from_tuple in C++17 ===

Step 1: Constructing session directly from tuple...
[Init] DatabaseSession established -> admin_svc@cluster-node-01.internal:5432 [SSL: true]
[Query] admin_svc@cluster-node-01.internal -> SELECT * FROM audit_logs;

Step 2: Generic factory construction...
[Init] DatabaseSession established -> readonly_usr@read-replica.internal:5433 [SSL: false]
[Query] readonly_usr@read-replica.internal -> SELECT count(*) FROM metrics;

Step 3: Pair to 2D Dimension Object...
Resolution: 1920x1080
```

---

## 16. Exercises

### Exercise 1 — Delayed Object Factory
Write a template class `DeferredInstance<T, ArgsTuple>` that takes a tuple of arguments during initialization, stores them, and provides a method `T instantiate()` that constructs and returns `T` on demand using `std::make_from_tuple`.

### Exercise 2 — Move-Only Dependency Instantiation
Construct a class `Pipeline` whose constructor accepts a `std::unique_ptr<std::string>` and a `std::unique_ptr<int>`. Store these two smart pointers in a `std::tuple`, and construct `Pipeline` using `std::make_from_tuple(std::move(args))`. Confirm that no compiler errors occur with move-only objects.

### Exercise 3 — Handling Aggregates in C++17
Define a struct `Point3D { double x; double y; double z; };`. Show what happens when passing a `std::tuple<double, double, double>` to `std::make_from_tuple<Point3D>` in C++17 mode. Fix the issue by providing the appropriate constructor.

### Exercise 4 — Compile-Time Verification (`constexpr`)
Write a simple struct `Rectangle { int width; int height; int area; constexpr Rectangle(int w, int h) : width(w), height(h), area(w * h) {} };`. Verify that `constexpr auto r = std::make_from_tuple<Rectangle>(std::make_tuple(4, 5));` compiles and can be evaluated at compile time in a `static_assert(r.area == 20)`.

---

## 17. Conclusion

`std::make_from_tuple` solves a long-standing omission in the C++ standard library:
- It **bridges the gap** between tuple-like data structures and class constructors.
- It **eliminates custom template metaprogramming**, removing handwritten `index_sequence` machinery.
- It supports **perfect forwarding** and `constexpr` evaluation with **zero runtime overhead**.
- Together with `std::apply`, it provides a complete toolkit for unpacking collections into both functions and objects.

---

## 18. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |[Ordikhani](ordikhanifateme@gmail.com) |  | [Ordikhani](@OrdikhaniFateme) |