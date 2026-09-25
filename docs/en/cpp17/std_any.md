<div align="center">

[🇺🇸 English](./std_any.md) | [🇮🇷 فارسی](../../fa/c++17/std_any.md)

</div>

---

# `std::any` in C++17 and Beyond — Complete Deep-Dive Guide

`std::any` is a type-safe container introduced in C++17 that can store a single value of almost any copy-constructible type. Its stored type can change dynamically at runtime while retaining complete type safety, value semantics, and automatic lifetime management without requiring common base classes or inheritance hierarchies.

```cpp
#include <any>
#include <iostream>
#include <string>

int main() {
    std::any value = 42;
    value = std::string("Hello, Modern C++");
    value = 3.14159;

    if (const auto* val = std::any_cast<double>(&value)) {
        std::cout << "Stored double: " << *val << '\n';
    }
}
```

This guide covers the conceptual model of `std::any`, its underlying type-erasure and Small Object Optimization (SOO) mechanisms, safe type inspection, reference and pointer extraction, lifetime management, performance characteristics, and interactions with modern C++ standards.

---

## Table of Contents

1. [Why `std::any` Exists](#1-why-stdany-exists)
2. [Required Header & Compilation](#2-required-header--compilation)
3. [Syntax Overview & Core States](#3-syntax-overview--core-states)
4. [How `std::any` Works: Type Erasure & Mechanics](#4-how-stdany-works-type-erasure--mechanics)
5. [Memory Layout & Small Object Optimization (SOO)](#5-memory-layout--small-object-optimization-soo)
6. [Type Constraints & Storable Types](#6-type-constraints--storable-types)
7. [Checking State and Inspecting Types](#7-checking-state-and-inspecting-types)
8. [Value Extraction with `std::any_cast`](#8-value-extraction-with-stdany_cast)
9. [Reference Extraction (Avoiding Copies)](#9-reference-extraction-avoiding-copies)
10. [Pointer Form of `std::any_cast` (Safe Non-throwing Inspection)](#10-pointer-form-of-stdany_cast-safe-non-throwing-inspection)
11. [Creating Objects with `emplace` and `std::make_any`](#11-creating-objects-with-emplace-and-stdmake_any)
12. [Resetting, Swapping, and Lifetime Management](#12-resetting-swapping-and-lifetime-management)
13. [Copy, Move, and Value Categories](#13-copy-move-and-value-categories)
14. [`std::any` vs. `void*`](#14-stdany-vs-void)
15. [`std::any` vs. `std::variant` vs. `std::optional`](#15-stdany-vs-stdvariant-vs-stdoptional)
16. [When to Use `std::any`](#16-when-to-use-stdany)
17. [Common Pitfalls and Gotchas](#17-common-pitfalls-and-gotchas)
18. [Performance Considerations](#18-performance-considerations)
19. [Best Practices](#19-best-practices)
20. [`std::any` in C++20 and Beyond](#20-stdany-in-c20-and-beyond)
21. [Feature-Test Macro](#21-feature-test-macro)
22. [Quick Reference Card](#22-quick-reference-card)
23. [Complete Example](#23-complete-example)
24. [Exercises & Solutions](#24-exercises--solutions)
25. [Conclusion](#25-conclusion)
26. [Contributors](#26-contributors)

---

## 1. Why `std::any` Exists

C++ is a statically typed language where types are determined and checked at compile time. However, certain domains require storing and passing values whose types cannot be predicted ahead of time:

- Plugin architectures and extensible module payloads
- Heterogeneous message brokers and event buses
- Dynamic configuration bags and runtime property systems
- Reflection and dynamic language bridges (Python/Lua bindings)

### Before C++17: The `void*` Approach

```cpp
// Legacy, unsafe approach
void* data = new int(42);

// Undefined Behavior: Reinterpreting int as double without warning
double* bad_ptr = static_cast<double*>(data); 

delete static_cast<int*>(data); // Manual lifetime management prone to leaks
```

### Before C++17: Polymorphic Base Classes

```cpp
// Intrusive approach
struct ObjectBase {
    virtual ~ObjectBase() = default;
};

template <typename T>
struct BoxedObject : ObjectBase {
    T value;
    explicit BoxedObject(T v) : value(std::move(v)) {}
};

std::unique_ptr<ObjectBase> obj = std::make_unique<BoxedObject<int>>(42);
```

### Problems with Older Approaches

- **`void*`** provides zero type safety, causes silent bugs on mismatched casts, and requires manual allocation and deallocation.
- **Base-class polymorphism** is intrusive, mandates dynamic allocation for every scalar, introduces virtual function table (vtable) overhead, and prevents value semantics.
- **`boost::any`** filled this gap historically, prompting the C++ committee to standardize `std::any` in C++17 as a native, optimized solution.

---

## 2. Required Header & Compilation

`std::any` is defined in the standard `<any>` header:

```cpp
#include <any>
```

### Compiler Commands

- **GCC / Clang:**
  ```bash
  g++ -std=c++17 -Wall -Wextra -O2 main.cpp -o app
  ```
- **MSVC:**
  ```powershell
  cl /std:c++17 /EHsc /W4 /O2 main.cpp
  ```

---

## 3. Syntax Overview & Core States

An `std::any` object operates in one of two states:

1. **Empty / Disengaged:** Contains no value (`has_value() == false`).
2. **Engaged:** Holds an instance of a concrete, copy-constructible type `T` (`has_value() == true`).

```cpp
#include <any>
#include <string>

// 1. Default constructor: Empty state
std::any a;

// 2. Direct initialization (deduces type)
std::any b = 42;                         // Holds int
std::any c = std::string("Modern C++");  // Holds std::string

// 3. Dynamic reassignment (type can change at runtime)
a = 3.14159;                             // Now holds double
a = std::string("Type mutated");         // Destroys double, now holds std::string
```

---

## 4. How `std::any` Works: Type Erasure & Mechanics

`std::any` implements non-intrusive **Type Erasure**. It strips the concrete compile-time type while packaging runtime type information alongside operations to manage object lifetimes.



### Internal Implementation Model

Internally, implementations generally use a function pointer (the *manager function*) that handles operations based on an internal action enum:

```cpp
// Conceptual Model of Internal Type Erasure Mechanics
enum class AnyAction { Destroy, Copy, Move, GetTypeInfo, GetPointer };

using AnyManager = void* (*)(AnyAction, void* src, void* dst);

template <typename T>
void* any_manager(AnyAction action, void* src, void* dst) {
    switch (action) {
        case AnyAction::Destroy:
            static_cast<T*>(src)->~T();
            return nullptr;
        case AnyAction::Copy:
            new (dst) T(*static_cast<const T*>(src));
            return nullptr;
        case AnyAction::GetTypeInfo:
            return const_cast<void*>(static_cast<const void*>(&typeid(T)));
        case AnyAction::GetPointer:
            return src;
        default:
            return nullptr;
    }
}
```

---

## 5. Memory Layout & Small Object Optimization (SOO)

To avoid dynamic heap allocations for small types (like `int`, `double`, small structs, or small function objects), standard library implementations apply **Small Object Optimization (SOO)**.


### Standard Guarantee vs. Implementation Details

- The C++ standard **does not mandate** an exact inline buffer size.
- Typical inline buffer sizes:
  - **libstdc++ (GCC):** Typically fits objects up to `sizeof(void*) * 2` (16 bytes on 64-bit systems) with `alignof(T) <= alignof(max_align_t)`.
  - **MSVC STL:** Fits objects up to 3 pointers worth of data (24 bytes on 64-bit systems).
  - **libc++ (Clang):** Similar 3-word inline capacity.
- **Nothrow Requirement:** Implementations generally enable SOO only if `std::is_nothrow_move_constructible_v<T>` is `true`.

```cpp
#include <any>
#include <iostream>

struct Small { char data[16]; };
struct Large { char data[256]; };

int main() {
    // Both instances have identical sizeof on the stack
    std::cout << "sizeof(std::any): " << sizeof(std::any) << " bytes\n";
    // Usually 32 bytes or 64 bytes depending on implementation & padding
}
```

---

## 6. Type Constraints & Storable Types

### Requirement: `std::is_copy_constructible_v<T>`

`std::any` requires that the stored type decay to a copy-constructible type (`std::is_copy_constructible_v<std::decay_t<T>> == true`).

```cpp
#include <any>
#include <memory>

// OK: Copyable
std::any a = 10;
std::any b = std::make_shared<int>(42);

// ERROR: std::unique_ptr is move-only (Not CopyConstructible)
// std::any c = std::make_unique<int>(42); // Static assertion failure
```

### Handling Move-Only Types

If you must store move-only objects, wrap them in `std::shared_ptr` or build a custom move-only type-erased wrapper:

```cpp
#include <any>
#include <memory>

struct MoveOnlyPayload {
    std::unique_ptr<int[]> buffer;
};

// Workaround using shared ownership
std::any wrapped = std::make_shared<MoveOnlyPayload>();
```

---

## 7. Checking State and Inspecting Types

### Checking Existence: `has_value()`

```cpp
std::any val;
if (!val.has_value()) {
    std::cout << "Empty\n";
}

val = 100;
if (val.has_value()) {
    std::cout << "Engaged\n";
}
```

### Runtime Type Inspection: `type()`

The member function `type()` returns the `const std::type_info&` of the held object. If the container is empty, it returns `typeid(void)`.

```cpp
#include <any>
#include <iostream>
#include <typeinfo>

std::any a = 42;
if (a.type() == typeid(int)) {
    std::cout << "Contains an int\n";
}

std::any empty;
if (empty.type() == typeid(void)) {
    std::cout << "Contains nothing (void)\n";
}
```

> **Warning:** Never use `a.type().name()` for conditional control flow. `name()` returns compiler-mangled, implementation-defined strings (`i` on GCC/Clang, `int` on MSVC). Always compare `type()` against `typeid(T)`.

---

## 8. Value Extraction with `std::any_cast`

`std::any_cast<T>` extracts the underlying value. It performs an exact type match comparison.

```cpp
std::any value = 42;

// Exact match: OK
int number = std::any_cast<int>(value);
```

### Exact Match Requirement (No Conversions)

`std::any` does not perform implicit numeric or polymorphic conversions:

```cpp
std::any value = 42; // Stored as int

try {
    // Fails: int will NOT convert to double or short automatically
    double d = std::any_cast<double>(value);
} catch (const std::bad_any_cast& e) {
    std::cerr << "Cast failed: " << e.what() << '\n';
}
```

---

## 9. Reference Extraction (Avoiding Copies)

By default, `std::any_cast<T>(any_obj)` makes a copy of the contained object. To avoid expensive copies or to modify the contained value in place, cast to a reference type (`T&` or `const T&`).

```cpp
#include <any>
#include <iostream>
#include <string>

int main() {
    std::any value = std::string("Deep Dive");

    // 1. Read-only access without copying
    const auto& read_ref = std::any_cast<const std::string&>(value);
    std::cout << "Length: " << read_ref.length() << '\n';

    // 2. In-place modification
    auto& mutable_ref = std::any_cast<std::string&>(value);
    mutable_ref += " in Modern C++";

    // 3. Move out of std::any
    std::string drained = std::any_cast<std::string>(std::move(value));
    // value now holds an empty std::string (moved-from state)
}
```

---

## 10. Pointer Form of `std::any_cast` (Safe Non-throwing Inspection)

Passing a pointer to `std::any` into `std::any_cast` returns a pointer to the stored element if the type matches, or `nullptr` if it fails or if the `std::any` is empty. **This form never throws exceptions.**

```cpp
#include <any>
#include <iostream>
#include <string>

void process_data(const std::any& data) {
    if (const auto* i = std::any_cast<int>(&data)) {
        std::cout << "Found int: " << *i << '\n';
    } else if (const auto* s = std::any_cast<std::string>(&data)) {
        std::cout << "Found string: " << *s << '\n';
    } else if (const auto* d = std::any_cast<double>(&data)) {
        std::cout << "Found double: " << *d << '\n';
    } else {
        std::cout << "Unknown or empty payload\n";
    }
}
```

```cpp
// Modifiable Pointer Form
std::any value = 100;
if (auto* ptr = std::any_cast<int>(&value)) {
    *ptr = 200; // Directly updates the stored value
}
```

---

## 11. Creating Objects with `emplace` and `std::make_any`

### `emplace<T>(args...)`

Constructs an object of type `T` in place directly inside the `std::any` container, destroying any previously held value.

```cpp
#include <any>
#include <vector>
#include <string>

struct ServerNode {
    std::string hostname;
    int port;
    std::vector<int> routes;

    ServerNode(std::string h, int p, std::vector<int> r)
        : hostname(std::move(h)), port(p), routes(std::move(r)) {}
};

int main() {
    std::any node;
    // In-place construction: no temporary ServerNode copies
    node.emplace<ServerNode>("127.0.0.1", 8080, std::vector<int>{1, 2, 3});
}
```

### `std::make_any<T>(args...)`

Helper factory function equivalent to `std::make_shared` or `std::make_unique`:

```cpp
auto text = std::make_any<std::string>(10, 'A'); // Holds "AAAAAAAAAA"
auto node = std::make_any<ServerNode>("localhost", 443, std::vector<int>{80, 443});
```

---

## 12. Resetting, Swapping, and Lifetime Management

### Resetting Storage

`reset()` destroys the contained object and returns `std::any` to the disengaged state.

```cpp
std::any a = 42;
a.reset();
assert(a.has_value() == false);
```

### Swapping Containers

Member `swap` and non-member `std::swap` exchange contents without reallocating underlying payloads (by exchanging internal buffers and function pointers):

```cpp
std::any a = 42;
std::any b = std::string("Swap Target");

a.swap(b);
// a now holds std::string("Swap Target"), b holds int(42)
```

---

## 13. Copy, Move, and Value Categories

Copying an `std::any` performs a deep copy of the contained object via the erased type's copy constructor. Moving transfers ownership of the storage and manager pointer without copying the underlying object payload.

```cpp
#include <any>
#include <iostream>
#include <string>

struct Tracker {
    std::string tag;
    Tracker(std::string t) : tag(std::move(t)) {}
    Tracker(const Tracker& o) : tag(o.tag) { std::cout << "Copied\n"; }
    Tracker(Tracker&& o) noexcept : tag(std::move(o.tag)) { std::cout << "Moved\n"; }
};

int main() {
    std::any a = Tracker("Primary");
    
    std::any b = a;            // Outputs: "Copied"
    std::any c = std::move(a); // No copy. Fast pointer/manager transfer.
}
```

---

## 14. `std::any` vs. `void*`

| Feature | `void*` | `std::any` |
|---|---|---|
| **Type Safety** | None (unsafe casts) | Strict (verified by `std::type_info`) |
| **Lifetime Management** | Manual (`new` / `delete`) | Automatic (RAII via type-erased destructor) |
| **Copy Semantics** | Shallow pointer copy | Deep copy of contained value |
| **Type Mismatch** | Undefined Behavior | Throws `std::bad_any_cast` or returns `nullptr` |
| **Small Object Optimization** | Not applicable | Yes (implementation-dependent) |
| **Recommended Use** | Low-level OS/C interop | Modern application architecture |

---

## 15. `std::any` vs. `std::variant` vs. `std::optional`

| Feature | `std::optional<T>` | `std::variant<Ts...>` | `std::any` |
|---|---|---|---|
| **Type Set** | Single type $T$ (or empty) | Closed, compile-time list | Open-ended, dynamic |
| **Dispatch Mechanism** | Direct check (`bool()`) | Static visitor (`std::visit`) | Dynamic downcast (`any_cast`) |
| **Dynamic Allocation** | Never | Never | Possible (for large types) |
| **Compile-time Check** | Exhaustive | Exhaustive | None (resolved at runtime) |
| **Performance** | Zero overhead | Direct stack/index lookup | Type check & potential heap indirection |
| **Best Use Case** | Missing/Nullable value | Finite state machines, algebraic types | Extensible plugins, metadata bags |

```cpp
// When to use which:
std::optional<int> maybe_port;                          // Value might not exist
std::variant<int, std::string, double> closed_payload;   // Fixed set of types known at compile time
std::any open_payload;                                   // Completely extensible payload
```

---

## 16. When to Use `std::any`

`std::any` is ideal when the type universe cannot be closed at compile time:

1. **Plugin Architecture & Extensions:** Passing opaque payloads between decoupled modules without shared inheritance.
2. **Dynamic Property Systems / Entity-Component Bags:** Attaching arbitrary runtime properties to game entities or UI nodes.
3. **Generic Event Hubs:** Distributing messages across decoupled application layers.
4. **Interpreters and FFI Bridges:** Exposing dynamic variables between C++ and script runtimes (Lua, Python, JS).

```cpp
// Generic Property Bag Example
#include <any>
#include <string>
#include <unordered_map>

class PropertyBag {
public:
    template <typename T>
    void set(const std::string& key, T&& val) {
        storage_[key] = std::forward<T>(val);
    }

    template <typename T>
    const T* get(const std::string& key) const {
        auto it = storage_.find(key);
        if (it == storage_.end()) return nullptr;
        return std::any_cast<T>(&it->second);
    }

private:
    std::unordered_map<std::string, std::any> storage_;
};
```

---

## 17. Common Pitfalls and Gotchas

### 17.1 String Literal Decay to `const char*`

Assigning a string literal stores `const char*`, not `std::string`:

```cpp
std::any val = "Hello World"; // Stored type is const char*

// CRASH: Throws std::bad_any_cast
// std::string str = std::any_cast<std::string>(val);

// CORRECT:
const char* raw = std::any_cast<const char*>(val);
// OR store std::string explicitly:
std::any str_val = std::string("Hello World");
```

### 17.2 Array-to-Pointer Decay

C-style arrays decay to raw pointers:

```cpp
int arr[5] = {1, 2, 3, 4, 5};
std::any val = arr; // Stored type is int* (decayed)

// std::any_cast<int[5]>(val); // FAILS
int* ptr = std::any_cast<int*>(val); // SUCCEEDS
```

### 17.3 Unintended Copies from Value Casts

```cpp
std::any val = std::string(1000, 'X');

// SLOW: Performs an expensive copy of the 1000-char string
std::string copied = std::any_cast<std::string>(val);

// FAST: Zero-copy const reference
const auto& ref = std::any_cast<const std::string&>(val);
```

### 17.4 Passing `std::any` by Value

```cpp
void process(std::any payload);        // BAD: Deep copies the entire any container
void process(const std::any& payload); // GOOD: Passes reference
```

### 17.5 Attempting Implicit Numeric Conversions

```cpp
std::any val = 10; // Holds int
// Throws std::bad_any_cast:
// double d = std::any_cast<double>(val); 

// Correct explicit cast:
double d = static_cast<double>(std::any_cast<int>(val));
```

---

## 18. Performance Considerations

1. **Small Object Optimization (SOO):**
   - Keep frequently used types within the SOO threshold (typically $\le 16\text{--}24$ bytes with `noexcept` move constructor) to avoid heap allocation overhead.
2. **Runtime Overhead:**
   - Accessing data via `std::any_cast` requires a `typeid` equality comparison. While fast, it cannot be inlined or devirtualized as aggressively as `std::variant`.
3. **Prefer `std::variant` for Closed Sets:**
   - If the set of types is known at compile time, `std::variant` eliminates dynamic allocations and avoids `typeid` comparison overhead through index-based jump tables.

---

## 19. Best Practices

1. **Prefer the pointer form of `std::any_cast`** for non-exceptional branching logic.
2. **Use reference casts (`const T&` or `T&`)** to prevent accidental deep copies.
3. **Use `emplace<T>()` or `std::make_any<T>()`** to construct complex objects directly inside the container.
4. **Explicitly instantiate types for literals** (e.g., `using namespace std::string_literals; ""s`).
5. **Ensure move constructors are `noexcept`** so implementations can leverage Small Object Optimization.
6. **Do not use `std::any` as a replacement for clean polymorphic class hierarchies** or when `std::variant` can express the domain.

---

## 20. `std::any` in C++20 and Beyond

- **C++20 Constexpr Enhancements:** While `std::any` is heavily reliant on dynamic type information and storage managers, standard library implementations have incrementally made non-allocating sub-operations `constexpr`-friendly where technically viable.
- **Move-Only Erased Types (`std::move_only_any` proposals):** The C++ committee continues to evaluate proposals for non-copyable type-erasure wrappers to support move-only types like `std::unique_ptr`.
- **Reflection (Future C++):** Static and dynamic reflection will provide unified accessors that interoperate cleanly with type-erased structures.

---

## 21. Feature-Test Macro

The feature-test macro for `std::any` is:

```cpp
#include <any>

#if defined(__cpp_lib_any) && __cpp_lib_any >= 201606L
// Full C++17 std::any support is active
#endif
```

---

## 22. Quick Reference Card

| Operation | Syntax | Semantics |
|---|---|---|
| **Empty Check** | `a.has_value()` | Returns `true` if an object is held; `false` otherwise |
| **Type Query** | `a.type()` | Returns `const std::type_info&` (or `typeid(void)` if empty) |
| **Value Extraction** | `std::any_cast<T>(a)` | Returns a copy of the value; throws `std::bad_any_cast` on mismatch |
| **Reference Extraction**| `std::any_cast<T&>(a)` | Returns a reference without copying; throws on mismatch |
| **Pointer Extraction**  | `std::any_cast<T>(&a)` | Returns `T*` or `nullptr` (never throws) |
| **In-place Construct** | `a.emplace<T>(args...)`| Constructs `T` directly; replaces previous content |
| **Factory Creation**   | `std::make_any<T>(args...)`| Constructs an `std::any` containing `T` |
| **Clear**              | `a.reset()` | Destroys contained object and enters empty state |
| **Swap**               | `a.swap(b)` | Exchanges contents of two containers |

---

## 23. Complete Example

The following program demonstrates type-safe storage, dynamic dispatch using the pointer cast idiom, in-place construction, mutation via references, and moving elements:

```cpp
#include <any>
#include <iostream>
#include <string>
#include <vector>
#include <unordered_map>

struct SystemEvent {
    std::string source;
    int priority;
};

void dispatch_event(const std::string& key, const std::any& payload) {
    std::cout << "Event [" << key << "]: ";

    if (!payload.has_value()) {
        std::cout << "<EMPTY>\n";
        return;
    }

    if (const auto* val = std::any_cast<int>(&payload)) {
        std::cout << "(int) " << *val << '\n';
    } else if (const auto* text = std::any_cast<std::string>(&payload)) {
        std::cout << "(string) \"" << *text << "\"\n";
    } else if (const auto* flag = std::any_cast<bool>(&payload)) {
        std::cout << "(bool) " << std::boolalpha << *flag << '\n';
    } else if (const auto* ev = std::any_cast<SystemEvent>(&payload)) {
        std::cout << "(SystemEvent) { Source: " << ev->source 
                  << ", Priority: " << ev->priority << " }\n";
    } else {
        std::cout << "Unhandled type: " << payload.type().name() << '\n';
    }
}

int main() {
    std::unordered_map<std::string, std::any> event_bus;

    // 1. Storing primitive values
    event_bus["packet_count"] = 1024;
    event_bus["network_down"] = false;
    event_bus["gateway_ip"] = std::string("192.168.1.1");

    // 2. In-place construction of custom structures
    event_bus["alert"].emplace<SystemEvent>(SystemEvent{"Firewall", 1});

    // 3. Inspecting and dispatching events
    for (const auto& [name, data] : event_bus) {
        dispatch_event(name, data);
    }

    // 4. In-place modification through reference cast
    auto& count_ref = std::any_cast<int&>(event_bus["packet_count"]);
    count_ref += 512;
    std::cout << "\nUpdated packet count: " 
              << std::any_cast<int>(event_bus["packet_count"]) << '\n';

    // 5. Non-throwing cast check
    if (auto* ptr = std::any_cast<double>(&event_bus["packet_count"])) {
        std::cout << "Double value: " << *ptr << '\n';
    } else {
        std::cout << "Verified: 'packet_count' is not a double.\n";
    }

    // 6. Resetting value
    event_bus["alert"].reset();
    dispatch_event("alert", event_bus["alert"]);

    return 0;
}
```

---

## 24. Exercises & Solutions

### Exercise 1: Heterogeneous Configuration Manager
**Task:** Build an application configuration container using `std::unordered_map<std::string, std::any>`. Add entries for `"host"` (`std::string`), `"port"` (`int`), and `"tls"` (`bool`). Read the `"port"` safely using pointer-form `any_cast`.

#### Solution:
```cpp
#include <any>
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, std::any> config;
    config["host"] = std::string("127.0.0.1");
    config["port"] = 8080;
    config["tls"] = true;

    if (const auto* port = std::any_cast<int>(&config["port"])) {
        std::cout << "Configured Port: " << *port << '\n';
    } else {
        std::cerr << "Invalid port configuration\n";
    }
}
```

---

### Exercise 2: Implementing a Non-throwing `try_get` Helper
**Task:** Implement two template helper functions:
1. `try_get_ptr<T>(const std::any&)` returning `const T*`.
2. `try_get_opt<T>(const std::any&)` returning `std::optional<T>`.

#### Solution:
```cpp
#include <any>
#include <optional>
#include <iostream>
#include <string>

template <typename T>
const T* try_get_ptr(const std::any& a) noexcept {
    return std::any_cast<T>(&a);
}

template <typename T>
std::optional<T> try_get_opt(const std::any& a) {
    if (const auto* ptr = std::any_cast<T>(&a)) {
        return *ptr;
    }
    return std::nullopt;
}

int main() {
    std::any val = std::string("C++17 Any");

    if (auto ptr = try_get_ptr<std::string>(val)) {
        std::cout << "Pointer extracted: " << *ptr << '\n';
    }

    auto opt = try_get_opt<std::string>(val);
    if (opt) {
        std::cout << "Optional extracted: " << *opt << '\n';
    }
}
```

---

### Exercise 3: Move-Only Wrapper Strategy
**Task:** `std::any` fails to compile with `std::unique_ptr`. Demonstrate how to store move-only heap resources inside `std::any` using shared ownership semantics without breaking type safety.

#### Solution:
```cpp
#include <any>
#include <iostream>
#include <memory>

struct BigResource {
    int id{42};
};

int main() {
    // std::any bad = std::make_unique<BigResource>(); // Compile Error
    
    // Solution: Wrap in std::shared_ptr to satisfy CopyConstructible requirement
    std::any safe_resource = std::make_shared<BigResource>();

    if (const auto* ptr = std::any_cast<std::shared_ptr<BigResource>>(&safe_resource)) {
        std::cout << "Extracted Resource ID: " << (*ptr)->id << '\n';
    }
}
```

---

## 25. Conclusion

`std::any` is a standardized, type-safe, and non-intrusive container designed for open-ended type management in C++17.

### Summary Checklist

- **Strict Type Safety:** Retains exact runtime type metadata (`std::type_info`) and prohibits implicit conversions.
- **Value Semantics & RAII:** Automatically invokes destructors, deep-copies held values, and supports zero-cost moves.
- **Pointer `any_cast`:** Idiomatic pattern for non-throwing type checks and branch-based inspections.
- **Performance Aware:** Utilizes Small Object Optimization (SOO) to eliminate heap allocations for small, nothrow-movable types.
- **Architectural Fit:** Choose `std::any` for open-ended extensibility (plugins, event buses, dynamic properties). Prefer `std::variant` when the set of types is known at compile time.

---

## 26. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  | [Ordikhani](mailto:ordikhanifateme@gmail.com) |  | [@OrdikhaniFateme](https://t.me/OrdikhaniFateme) |
