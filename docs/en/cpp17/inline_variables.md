-

<div align="center">

[🇺🇸 English](./inline_variables.md) | [🇮🇷 فارسی](../../fa/cpp17/inline_variables.md)

</div>

----

# `inline` Variables in C++17: Header-Only Constants and Static Class Members Without ODR Violations

`inline` variables are a fundamental language feature introduced in C++17 that allows variables to be defined directly in header files without causing **One Definition Rule (ODR)** violations at link time. Prior to C++17, only functions could be declared `inline`. With C++17, global constants, namespace-level configurations, and `static` class members can finally be declared and defined entirely inside header files with a single, unified memory address across all translation units.

```cpp
// config.hpp
#pragma once
#include <string_view>

// Valid in C++17: can be included in 100+ translation units without linker errors!
inline constexpr std::string_view kAppVersion = "2.4.1";

struct ServerConfig {
    // Defined inside the class body, no separate .cpp definition required!
    inline static int default_port = 8080;
};
```

This guide covers the formal definition of ODR, why `inline` variables exist, the exact compilation/link failure mechanics they solve, differences between `static`, `extern`, and `inline`, memory layout, template variables, header-only library design, and common pitfalls.

---

## Table of Contents

1. [What is the One Definition Rule (ODR)?](#1-what-is-the-one-definition-rule-odr)
2. [How Was ODR Violated Before C++17?](#2-how-was-odr-violated-before-c17)
3. [The Workarounds Before C++17 (And Why They Hurt)](#3-the-workarounds-before-c17-and-why-they-hurt)
4. [Syntax Overview](#4-syntax-overview)
5. [What `inline` Actually Means for a Variable](#5-what-inline-actually-means-for-a-variable)
6. [`inline static` Class Members](#6-inline-static-class-members)
7. [`constexpr` vs. `inline`: The Implicit Rule](#7-constexpr-vs-inline-the-implicit-rule)
8. [Linkage, Address Equivalence, and ODR](#8-linkage-address-equivalence-and-odr)
9. [`inline` Variables vs. `static` at Namespace Scope](#9-inline-variables-vs-static-at-namespace-scope)
10. [`inline` Variables with Templates](#10-inline-variables-with-templates)
11. [Header-Only Library Architecture](#11-header-only-library-architecture)
12. [Interaction with Other C++17 Features](#12-interaction-with-other-c17-features)
13. [Common Pitfalls and Gotchas](#13-common-pitfalls-and-gotchas)
14. [Performance and Binary Size Considerations](#14-performance-and-binary-size-considerations)
15. [Best Practices](#15-best-practices)
16. [Feature-Test Macro](#16-feature-test-macro)
17. [Quick Reference Card](#17-quick-reference-card)
18. [Complete Example](#18-complete-example)
19. [Exercises](#19-exercises)
20. [Conclusion](#20-conclusion)
21. [Contributors](#21-contributors)

---

## 1. What is the One Definition Rule (ODR)?

The **One Definition Rule (ODR)** is a foundational rule in the ISO C++ standard (`[basic.def.odr]`) that governs how entities can be defined across translation units (TUs).

It consists of three core tenets:

1. **Within a single Translation Unit:** No variable, function, class type, enumeration, or template can have more than one definition.
2. **Across the entire Program (for non-inline entities with external linkage):** An object or non-inline function with external linkage must have **exactly one definition in the entire program**. If more than one definition exists across different translation units, the program is ill-formed, resulting in link-time duplicate symbol errors.
3. **Across the entire Program (for inline entities and templates):** An `inline` function, `inline` variable, class type, or template can be defined in multiple translation units, provided that:
   - Each definition appears in a separate translation unit.
   - Each definition consists of the **identical sequence of tokens**.
   - All definitions have identical semantics (meaning they do not resolve names differently).

If an entity violates rule #2, the linker cannot decide which symbol definition to bind to references, terminating the build process with a multiple definition error.

---

## 2. How Was ODR Violated Before C++17?

Prior to C++17, defining global variables or static class members directly in headers inevitably triggered linker collisions or forced bad engineering trade-offs.

### The Underlying Compilation & Link Mechanics

When a header file is `#include`d into multiple `.cpp` files:
1. The preprocessor literally pastes the contents of the header into each source file.
2. Each source file is compiled **independently** into an isolated translation unit (`.o` / `.obj`).
3. If the header contains a variable definition with external linkage, that variable symbol is emitted as an exported global symbol into every object file.
4. During the **link stage**, the linker gathers all object files into the final executable. Seeing multiple identical non-weak global symbols, the linker halts.

---

### Scenario 1: Defining a Global Configuration Variable in a Header

```cpp
// config.hpp
#pragma once
#include <string>

// A definition with external linkage
int max_users = 100;
std::string global_endpoint = "https://api.example.com";
```

```cpp
// auth.cpp
#include "config.hpp"
// Emits symbol `max_users` into auth.o
```

```cpp
// network.cpp
#include "config.hpp"
// Emits symbol `max_users` into network.o
```

At link time:
```text
duplicate symbol: max_users in:
    auth.o
    network.o
ld: 1 duplicate symbol for architecture x86_64
clang: error: linker command failed with exit code 1
```

---

### Scenario 2: Defining a Static Class Member

```cpp
// Window.hpp
#pragma once

class Window {
public:
    static int default_width; // Declaration only
};
```

In pre-C++17, you **could not** initialize `default_width` inside the class body unless it was `const integral` or `constexpr literal`. For any non-trivial or mutable type, you were forced to create a separate translation unit (`Window.cpp`):

```cpp
// Window.cpp
#include "Window.hpp"

int Window::default_width = 1920; // Definition must live in exactly one .cpp
```

If you attempted to define it in `Window.hpp` outside the class:
```cpp
// Window.hpp
class Window {
public:
    static int default_width;
};
int Window::default_width = 1920; // ODR VIOLATION if Window.hpp is included twice!
```

This completely broke header-only designs and forced library authors to distribute both `.hpp` and compiled `.cpp` files.

---

### Scenario 3: ODR-used `static constexpr` Class Members

Prior to C++17, writing:
```cpp
struct Math {
    static constexpr int factor = 42;
};
```
worked fine **unless** you took its address or bound it to an lvalue reference (`const int& r = Math::factor;`). If you took its address, it became *ODR-used*, and the compiler complained about a missing out-of-line definition in a `.cpp` file!

```cpp
void print_ref(const int& n);

int main() {
    print_ref(Math::factor); // Pre-C++17 Linker Error: undefined reference to 'Math::factor'
}
```

---

## 3. The Workarounds Before C++17 (And Why They Hurt)

Before C++17, developers used several clumsy workarounds to place variables in headers:

### Workaround 1: `static` in header (Silent Bug & Memory Waste)

```cpp
// config.hpp
static const std::string app_name = "MyApp";
```
- **The Problem:** `static` at namespace scope gives the variable **internal linkage**.
- Every single `.cpp` file that includes `config.hpp` gets its **own independent copy** of `app_name`.
- If the type is non-trivial (like `std::string` or `std::vector`), each translation unit runs constructors and destructors on program startup and exit.
- `&app_name` yields different addresses in different translation units!

### Workaround 2: The Inline Function / Meyer's Singleton Hack

```cpp
// config.hpp
inline const std::string& get_app_name() {
    static const std::string name = "MyApp";
    return name;
}
```
- **The Problem:** Works around the linker, but adds call overhead, parentheses syntax `get_app_name()`, and forces deferred runtime initialization.

### Workaround 3: The Dummy Template Hack

```cpp
// config.hpp
template <typename T = void>
struct ConfigHelper {
    static int port;
};

template <typename T>
int ConfigHelper<T>::port = 8080;

using Config = ConfigHelper<>;
```
- **The Problem:** Linkers already merged template variables across translation units. Programmers abused this loophole, introducing absurd boilerplate just to place an integer in a header.

---

## 4. Syntax Overview

The syntax is as simple as adding the `inline` specifier to a variable declaration:

```text
inline decl-specifier-seq init-declarator ;
```

### Common Declarations

```cpp
// 1. Global / Namespace-scope variable
inline int global_counter = 0;

// 2. Global constant object
inline const std::string default_user = "guest";

// 3. Static data member of a class
struct CacheSettings {
    inline static std::size_t max_entries = 1000;
};

// 4. Combined with constexpr
inline constexpr double PI = 3.14159265358979323846;
```

---

## 5. What `inline` Actually Means for a Variable

Just like an `inline` function, an `inline` variable:

1. **Can be defined in multiple translation units**, provided that each definition is identical (same tokens and semantics).
2. **Has external linkage** (unless explicitly marked `static`).
3. **Has a single, unique memory address:** The linker resolves all references across all object files to one canonical memory location (typically using COMDAT / weak symbols).
4. **Is initialized exactly once:** Static initialization happens once at program startup.

```cpp
// math_constants.hpp
#pragma once

inline double G = 9.80665;
```

If `physics.cpp` and `simulation.cpp` both use `G`:
```cpp
// In physics.cpp:
&G == 0x00007ff8

// In simulation.cpp:
&G == 0x00007ff8 // Exactly the same address!
```

---

## 6. `inline static` Class Members

The most immediate practical benefit of `inline` variables is simplifying static data members inside classes.

### Before C++17

```cpp
// Database.hpp
class Database {
    static std::string connection_string; // declaration
};

// Database.cpp
#include "Database.hpp"
std::string Database::connection_string = "postgresql://localhost:5432/app"; // required definition
```

### In C++17

```cpp
// Database.hpp
#pragma once
#include <string>

class Database {
public:
    inline static std::string connection_string = "postgresql://localhost:5432/app"; // OK!
};
```

No `.cpp` file is needed. The class remains 100% self-contained inside the header.

---

## 7. `constexpr` vs. `inline`: The Implicit Rule

In C++17, there is a very important rule regarding `constexpr` and `static` class members:

> **A `constexpr` static data member of a class is implicitly an `inline` variable.**

### What does this mean?

In C++17, because `static constexpr` class members are implicitly `inline`:

```cpp
struct Math {
    static constexpr int factor = 42; // Implicitly 'inline static constexpr'
};

void printRef(const int& n);

int main() {
    printRef(Math::factor); // Valid in C++17 without any out-of-line definition in .cpp!
}
```

> **Warning for Namespace-Scope Variables:**  
> A namespace-scope `constexpr` variable is **not** implicitly `inline`.  
> At namespace scope, `constexpr` defaults to **internal linkage** (just like `const`).  
> If you want a namespace-scope constant to have a single canonical address across all translation units, you **must explicitly declare it `inline constexpr`**:
> ```cpp
> inline constexpr int global_limit = 500; // Explicit inline required for global scope
> ```

---

## 8. Linkage, Address Equivalence, and ODR

| Specifier | Linkage | Address Across Translation Units | Can Be in Header? |
|---|---|---|---|
| `int x = 10;` | External | Linker Error (ODR conflict) | ❌ No |
| `static int x = 10;` | Internal | Different in each `.cpp` | ⚠️ Yes (creates duplicates) |
| `const int x = 10;` | Internal (in C++) | Different in each `.cpp` | ⚠️ Yes (optimized if literal) |
| `inline int x = 10;` | External | **Identical across all `.cpp`** | ✅ Yes |
| `inline constexpr int x = 10;` | External | **Identical across all `.cpp`** | ✅ Yes |

### Verifying Address Equivalence

```cpp
// common.hpp
#pragma once
inline int shared_flag = 42;
void print_addr_from_a();

// a.cpp
#include "common.hpp"
#include <iostream>
void print_addr_from_a() {
    std::cout << "a.cpp address: " << &shared_flag << '\n';
}

// main.cpp
#include "common.hpp"
#include <iostream>
int main() {
    std::cout << "main.cpp address: " << &shared_flag << '\n';
    print_addr_from_a();
}
```

**Output:**
```text
main.cpp address: 0x104b2c018
a.cpp address:    0x104b2c018
```
Both translation units access the exact same memory cell.

---

## 9. `inline` Variables vs. `static` at Namespace Scope

It is crucial not to confuse `static` and `inline` at namespace scope:

```cpp
// WARNING: Antipattern in modern C++
static std::string g_service_url = "https://service.local";
```

Why is `static` bad in header files?
1. **Bloat:** If 50 `.cpp` files include this header, 50 distinct `std::string` objects are instantiated, constructed, and destructed.
2. **Broken Logic:** Modifying `g_service_url` in one `.cpp` does **not** change it in another `.cpp`. Each file mutates its own private copy.

```cpp
// CORRECT in C++17
inline std::string g_service_url = "https://service.local";
```
Here, only one `std::string` is created, and modifying it in one translation unit is immediately visible in all others.

---

## 10. `inline` Variables with Templates

Variable templates were introduced in C++14. By default, variable templates already had inline-like linkage rules (the linker merged instantiations).

However, explicit template specializations did **not**!

```cpp
// types.hpp
template <typename T>
constexpr std::size_t type_weight = sizeof(T);

// Explicit specialization before C++17 had to go to a .cpp file!
// In C++17, mark it 'inline':
template <>
inline constexpr std::size_t type_weight<void> = 0;
```

Without `inline`, specializing a variable template in a header file caused multiple-definition errors across translation units.

---

## 11. Header-Only Library Architecture

`inline` variables are one of the biggest enablers for clean header-only library design in modern C++.

### Idiomatic Modern C++ Header Structure

```cpp
// LoggerLib.hpp
#pragma once
#include <string>
#include <mutex>
#include <iostream>

namespace LoggerLib {

    // Single configuration object across entire program
    struct Settings {
        inline static bool verbose = false;
        inline static std::string log_file = "app.log";
    };

    // Global synchronization mutex in header without ODR violation
    inline std::mutex logger_mutex;

    inline void log(const std::string& msg) {
        std::lock_guard<std::mutex> lock(logger_mutex);
        if (Settings::verbose) {
            std::cout << "[VERBOSE] " << msg << '\n';
        } else {
            std::cout << msg << '\n';
        }
    }
}
```

Any project can drop `LoggerLib.hpp` into its source tree without creating a companion `.cpp` file or setting up static library targets in CMake.

---

## 12. Interaction with Other C++17 Features

### 12.1 `inline constexpr std::string_view`

In C++17, `std::string_view` pairs exceptionally well with `inline constexpr` for global string constants:

```cpp
namespace System {
    inline constexpr std::string_view kOSName = "Linux";
    inline constexpr std::string_view kArch = "x86_64";
}
```
Zero allocations, compile-time string evaluation, and a single global definition.

### 12.2 Structured Bindings and `inline`

Structured bindings **cannot** be declared `inline`:

```cpp
// ERROR: Ill-formed
// inline auto [x, y] = std::pair{1, 2};
```

Structured bindings must be defined within a valid scope without the `inline` specifier.

---

## 13. Common Pitfalls and Gotchas

### 13.1 Static Initialization Order Fiasco (Still Applies!)

`inline` variables fix the **link-time multiple definition problem**, but they **do not** fix the **static initialization order fiasco**.

If an `inline` variable in `fileA.hpp` depends on the initial value of an `inline` variable in `fileB.hpp`:

```cpp
// a.hpp
inline int A = 10;

// b.hpp
#include "a.hpp"
inline int B = A * 2; // Order between A and B across different TUs is undefined!
```

> **Rule:** If order of initialization between global objects matters, continue using Meyer's Singleton (function-local static variables) or `constexpr`.

### 13.2 Forgetting `inline` with Non-Const Objects

```cpp
// globals.hpp
int current_session_id = 0; // ODR violation when included in >1 file!
```
Always verify that mutable global state in headers is marked `inline`.

### 13.3 Header Inconsistencies

All translation units must see the **exact same definition** of an `inline` variable. If macro definitions change the value across different files, you introduce undefined behavior (UB):

```cpp
// config.hpp
#ifdef DEBUG_MODE
inline int timeout = 1000;
#else
inline int timeout = 500; // UB if different .cpp files define DEBUG_MODE differently!
#endif
```

---

## 14. Performance and Binary Size Considerations

1. **Smaller Binaries:** Replacing namespace-scope `static const` with `inline constexpr` reduces binary size because the compiler no longer generates duplicate object instances for every translation unit.
2. **Zero Runtime Overhead:** For primitive and `constexpr` literal types, compilers treat `inline` variables as immediate values or place them in read-only data segments (`.rodata`).
3. **Link-Time Merging:** The linker uses COMDAT sections to deduplicate the symbol, resulting in virtually no link-time performance penalty.

---

## 15. Best Practices

1. **Use `inline static` for class-level constants and singletons** instead of splitting declarations across `.hpp` and definitions in `.cpp`.
2. **Use `inline constexpr` for namespace-level constants** to guarantee external linkage and address unification.
3. **Prefer `std::string_view` over `std::string`** when defining string constants as `inline constexpr`.
4. **Never put non-inline mutable globals in headers.**
5. **Beware the Static Initialization Order Fiasco** when `inline` variables have complex inter-dependencies.

---

## 16. Feature-Test Macro

The feature-test macro for `inline` variables is:

```cpp
__cpp_inline_variables
```

Standard value:
- `201606L` — C++17 `inline` variables support.

```cpp
#if defined(__cpp_inline_variables) && __cpp_inline_variables >= 201606L
    #define HAS_INLINE_VARIABLES 1
#else
    #define HAS_INLINE_VARIABLES 0
#endif
```

---

## 17. Quick Reference Card

| Context | Before C++17 | C++17 Idiomatic Style |
|---|---|---|
| Class static member | Declared in `.hpp`, defined in `.cpp` | `inline static int port = 8080;` inside class |
| Global constant string | `static const char*` (replicated) | `inline constexpr std::string_view name = "App";` |
| Class `static constexpr` | Required `.cpp` definition if ODR-used | Implicitly `inline`, never needs `.cpp` definition |
| Global mutable object | Required `extern` + single `.cpp` def | `inline std::string g_path = "/";` |
| Explicit template specialization | Required `.cpp` definition | `template<> inline constexpr int v<void> = 0;` |

---

## 18. Complete Example

```cpp
#include <iostream>
#include <string_view>
#include <vector>

// 1. Header-only namespace constants
namespace AppConfig {
    inline constexpr std::string_view kAppName = "TelemetryCore";
    inline constexpr int kMaxConnections = 128;
    inline int runtime_active_connections = 0; // Shared across all TUs
}

// 2. Class with self-contained static data members
class SystemMonitor {
public:
    inline static const std::string kSubsystemName = "NetworkMonitor";
    inline static std::vector<int> registered_nodes{1, 2, 3}; // Non-trivial member
    
    // Static constexpr is implicitly inline
    static constexpr double kSamplingIntervalSec = 0.5;

    static void printStatus() {
        std::cout << "Subsystem: " << kSubsystemName << '\n';
        std::cout << "Nodes: " << registered_nodes.size() << '\n';
        std::cout << "Sampling: " << kSamplingIntervalSec << "s\n";
    }
};

int main() {
    std::cout << "App: " << AppConfig::kAppName << '\n';
    std::cout << "Max Conns: " << AppConfig::kMaxConnections << '\n';

    AppConfig::runtime_active_connections += 5;
    std::cout << "Active: " << AppConfig::runtime_active_connections << '\n';

    SystemMonitor::printStatus();

    // Verify address safety (taking address of static constexpr)
    const double* interval_ptr = &SystemMonitor::kSamplingIntervalSec;
    std::cout << "Sampling address valid: " << (interval_ptr != nullptr) << '\n';

    return 0;
}
```

---

## 19. Exercises

### Exercise 1 — Eliminating the `.cpp` file
Take an existing class that declares `static std::string server_url;` and requires a `.cpp` file to initialize it. Rewrite it in C++17 using `inline static` and remove the `.cpp` file entirely.

### Exercise 2 — Verifying Address Identity
Create two translation units (`unit1.cpp` and `unit2.cpp`) that include a common header containing:
1. `static int var_static = 1;`
2. `inline int var_inline = 1;`  
Print the pointers `&var_static` and `&var_inline` from both files. Explain why one pair differs while the other is identical.

### Exercise 3 — Implicit `inline` Verification
Define a struct with a `static constexpr int code = 100;`. Pass it by reference to a function `void check(const int& ref);`. Compile in C++14 (which might require a definition at link time depending on flags) and compare with C++17.

### Exercise 4 — Refactoring String Constants
Convert a set of `#define APP_VERSION "1.0"` and `static const char* APP_AUTHOR = "Alice"` declarations into type-safe, idiomatic C++17 `inline constexpr std::string_view` variables.

---

## 20. Conclusion

`inline` variables in C++17 resolve one of the longest-standing architectural nuisances in C++ development:
- They allow **true header-only libraries** without workaround hacks or duplicate object instantiations.
- `static` class data members can finally be initialized **inline in the class body**.
- `static constexpr` members of classes are **implicitly inline**, eliminating linker errors when taking their address.
- When used with `inline constexpr std::string_view`, they provide zero-cost, type-safe global constants.

---

## 21. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |[Ordikhani](ordikhanifateme@gmail.com) |  | [Ordikhani](@OrdikhaniFateme) |