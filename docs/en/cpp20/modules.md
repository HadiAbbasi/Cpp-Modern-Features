<div align="center">

[🇺🇸 English](./modules.md) | [🇮🇷 فارسی](../../fa/cpp20/modules.md)

</div>

---

# Comprehensive Guide to Modules in C++20

The **Modules** feature is one of the most fundamental structural changes in the C++ language since its inception, introduced in the **C++20** standard. The primary goal of this feature is to replace the traditional, costly textual model (`#include`) with a modular, isolated, and semantic-based compilation model.

## Why Do We Need Modules and What Problem Do They Solve?

In the traditional header file model, the `#include` preprocessor directive simply copies the raw text content of one file into another. In large projects, this mechanism creates several severe problems:

* **Drastic Increase in Compilation Time:** The compiler is forced to repeatedly process and parse header files (such as `<vector>` or `<iostream>`) in every translation unit that includes them.
* **Macro Leakage:** Macros defined in a header file leak into all subsequent files and can inadvertently alter variable or function names.
* **Dependency on Inclusion Order:** Changing the order of `#include` lines can trigger compilation errors or alter program behavior.
* **Violation of the ODR (One Definition Rule):** Redefining entities across different header files easily leads to linker errors.
* **Lack of True Encapsulation:** Hiding implementation details in header files requires workarounds like the PIMPL idiom or `detail` namespaces, while the compiler still processes the entire code.

**Modules** solve these issues by converting the module interface into a precompiled binary representation known as a **Binary Module Interface (BMI)**. Macros do not cross module boundaries, include order does not matter, and each module is compiled only once.

## Basic Concepts and Structure of a Module

Every module in C++20 has a unique name and is defined using new contextual keywords such as `export`, `import`, and `module`.

### Core Parts of a Module Unit

Every module file is structurally divided into specific sections whose order is mandated by the standard:

1. **Global Module Fragment:** Begins with the `;module` directive at the top of the file. This is where traditional `#include` directives for legacy libraries or OS headers must be placed so they do not merge into the module.
2. **Module Declaration:** Starts with `;export module ModuleName` or `;module ModuleName`, establishing the primary module boundary.
3. **Module Purview:** Contains `import` directives, definitions, implementations, and exported symbols (`export`).
4. **Private Module Fragment:** An optional section at the very end of the file starting with `;module :private`, housing strictly private implementation details.

### Single Export vs. Block Export

To make functions, classes, variables, or namespaces available to module consumers, the `export` keyword is used. This can be done in two ways:

* **Single Export:** Placing `export` before each declaration individually.
* **Block Export:** Wrapping multiple declarations inside an `export { ... }` block, which improves readability and avoids repeating the `export` keyword.

The following example demonstrates a module interface unit using both approaches:

```cpp
// math_utils.ixx
module; // Global module fragment
#include <cmath>

export module math_utils; // Module declaration

// Single export
export int add(int a, int b) {
    return a + b;
}

// Block export
export {
    double calculate_hypotenuse(double a, double b) {
        return std::hypot(a, b);
    }

    struct Point2D {
        double x{};
        double y{};
    };
}

// Internal helper function (not exported)
double internal_scale(double val) {
    return val * 1.5;
}

```

In the consuming file (such as `main.cpp`), you simply import the module using the `import` directive:

```cpp
// main.cpp
import <iostream>;
import math_utils;

int main() {
    int sum = add(3, 4);
    Point2D p{3.0, 4.0};
    double dist = calculate_hypotenuse(p.x, p.y);
    
    // Error: internal_scale is not visible here
    // double s = internal_scale(10.0);
    
    std::cout << "Sum: " << sum << ", Distance: " << dist << '\n';
    return 0;
}

```

## Separating the Module Interface from Implementation

In real-world projects, placing all implementation code inside the module interface file causes any minor change in internal function logic to trigger a re-compilation of all dependent files. To prevent this, the module interface is separated from its implementation.

### Separation in the Same File

For smaller projects where you prefer keeping all module code in a single file while still separating the interface from the implementation, you can place declarations within the `export` section and provide their implementations further down in the same file (without `export`) or inside the **Private Module Fragment**.

The **Private Module Fragment**, denoted by `;module :private`, informs the compiler that any code following this line has zero impact on the public module interface. Consequently, modifications to code placed after `;module :private` will not trigger re-compilation of module consumers.

```cpp
// logger.ixx
module;
#include <string_view>
#include <iostream>

export module logger;

export class Logger {
public:
    explicit Logger(std::string_view prefix);
    void log(std::string_view message) const;

private:
    std::string_view prefix_;
};

// Everything below this line is completely private to this translation unit
module :private;

Logger::Logger(std::string_view prefix) : prefix_(prefix) {}

void Logger::log(std::string_view message) const {
    std::cout << "[" << prefix_ << "] " << message << '\n';
}

```

An important rule is that the **Private Module Fragment** is only permitted when the entire module consists of a single file; if the module uses separate implementation files or partitions, using `;module :private` results in a compilation error.

### Separation Across Different Files

The standard approach for medium and large projects is separating the **Module Interface Unit** from the **Module Implementation Unit**.

* **Module Interface Unit:** Contains the `;export module Name` directive and is typically saved with extensions like `.ixx` (in MSVC) or `.cppm` (in Clang and GCC).
* **Module Implementation Unit:** Contains the `;module Name` directive (without the `export` keyword) and is typically saved with the standard `.cpp` extension.

The implementation file automatically imports its corresponding module, granting it full access to all declarations and private parts of classes defined in the module interface.

The module interface file is written as follows:

```cpp
// bank_account.ixx
module;
#include <string>

export module bank_account;

export class BankAccount {
public:
    BankAccount(std::string owner, double initial_balance);
    void deposit(double amount);
    bool withdraw(double amount);
    double balance() const noexcept;

private:
    std::string owner_;
    double balance_;
};

```

The module implementation resides in a separate `.cpp` file:

```cpp
// bank_account.cpp
module;
#include <stdexcept>

module bank_account; // Implementation unit for bank_account

BankAccount::BankAccount(std::string owner, double initial_balance)
    : owner_(std::move(owner)), balance_(initial_balance) {
    if (balance_ < 0.0) {
        throw std::invalid_argument("Initial balance cannot be negative");
    }
}

void BankAccount::deposit(double amount) {
    if (amount > 0.0) {
        balance_ += amount;
    }
}

bool BankAccount::withdraw(double amount) {
    if (amount > 0.0 && balance_ >= amount) {
        balance_ -= amount;
        return true;
    }
    return false;
}

double BankAccount::balance() const noexcept {
    return balance_;
}

```

### Multiple Implementation Files

A module can have only **one** Primary Module Interface Unit, but there is no restriction on the number of implementation files. If a module contains multiple large classes or subsystems, you can place the implementation of each part in a separate `.cpp` file, starting each with the `;module ModuleName` directive.

Suppose the `network` module exports two classes, `TcpSocket` and `UdpSocket`, in its interface:

```cpp
// network.ixx
export module network;

export class TcpSocket {
public:
    void connect();
};

export class UdpSocket {
public:
    void bind();
};

```

The implementation of these two classes can be cleanly split across two separate files:

```cpp
// tcp_socket.cpp
module;
#include <iostream>

module network;

void TcpSocket::connect() {
    std::cout << "TCP Socket connected.\n";
}

```

```cpp
// udp_socket.cpp
module;
#include <iostream>

module network;

void UdpSocket::bind() {
    std::cout << "UDP Socket bound.\n";
}

```

## Structuring Large Modules and Interface Aggregation

In large libraries and frameworks, placing all interfaces in a single file is impractical. C++20 provides two mechanisms for partitioning modules: **Sub-Modules** and **Module Partitions**.

### Sub-Modules and Multiple Interface Files

In standard C++, the dot (`.`) character in a module name carries no special hierarchical meaning to the compiler and is simply part of the module name identifier. However, developers use dots to establish logical **Sub-Modules** (such as `graphics.shapes` and `graphics.colors`).

Each sub-module is a fully independent module with its own interface file. To spare end-users from importing every small module individually, an aggregating module is created that re-exports all sub-modules using `export import`.

### Single and Batch Export Import

The `export import` directive both loads the target module into the current module and exposes all of its exported symbols to consumers of the current module.

The following example demonstrates defining multiple interface files as sub-modules and aggregating them:

```cpp
// graphics_shapes.ixx
export module graphics.shapes;

export struct Circle {
    double radius;
};

export struct Rectangle {
    double width;
    double height;
};

```

```cpp
// graphics_colors.ixx
export module graphics.colors;

export struct RGB {
    unsigned char r, g, b;
};

```

In the main `graphics` module interface file, sub-modules can be exported individually or via an `export` block:

```cpp
// graphics.ixx
export module graphics;

// Separated export import
export import graphics.shapes;

// Batch export import using block export
export {
    import graphics.colors;
    // Other modules can also be added here
}

```

Now, a user writing a single line `import graphics;` gains access to `Circle`, `Rectangle`, and `RGB` structures.

### Module Partitions

Partitions allow you to physically divide a single module across multiple files without creating independent modules for the outside world. Partition names are specified using a colon (`:`), such as `engine:physics` or `engine:audio`.

The key difference between **Module Partitions** and **Sub-Modules** is that partitions cannot be directly imported from outside the module; consumers cannot write `;import engine:physics`. Partitions are visible only within the parent module itself, and the primary module must aggregate them.

Partitions are divided into two categories:

1. **Module Interface Partition:** Defined with `;export module ModuleName:PartitionName`, exporting symbols that must be re-exported by the primary module (`;export import :PartitionName`).
2. **Internal Module Partition:** Defined with `;module ModuleName:PartitionName` (without `export`), used exclusively for internal consumption by other parts of the same module (`;import :PartitionName`).

The following example shows a combination of an interface partition and an internal partition within the `engine` module:

```cpp
// engine_core.ixx (Internal Module Partition - NOT exported)
module engine:core;

struct InternalState {
    int tick_count = 0;
};

InternalState& get_state() {
    static InternalState state;
    return state;
}

```

```cpp
// engine_physics.ixx (Module Interface Partition)
export module engine:physics;

import :core; // Importing internal partition within the same module

export void update_physics() {
    auto& state = get_state();
    state.tick_count++;
}

```

```cpp
// engine.ixx (Primary Module Interface Unit)
export module engine;

// Re-exporting the interface partition is mandatory if it has exported entities
export import :physics;

// Internal partition :core is NOT re-exported, keeping it completely hidden

```

### Using Private Internal Modules Inside Exported Modules

When designing large libraries, parts of the code often need to be shared across multiple module files while remaining completely hidden from external consumers. Two standard patterns achieve this:

* **Using an Internal Partition within a Single Module:** As demonstrated previously, defining a partition without `export` (e.g., `;module engine:core`) guarantees that no external module can access it, while all files belonging to the `engine` module can import it.
* **Using a Helper Module with a Regular (Non-Exported) Import:** If you have two separate modules, the exported module (`public_api`) can import the internal module (`internal_helpers`) using a simple `;import internal_helpers` directive (without `export`). Consequently, functions and classes of `internal_helpers` are usable within `public_api`'s implementation while remaining invisible to programs importing `public_api`.

```cpp
// internal_crypto.ixx
export module internal_crypto;

export unsigned int compute_hash(const char* data) {
    unsigned int hash = 5381;
    while (*data) {
        hash = ((hash << 5) + hash) + static_cast<unsigned char>(*data++);
    }
    return hash;
}

```

```cpp
// auth_service.ixx
module;
#include <string>

export module auth_service;

import internal_crypto; // Imported privately, NOT re-exported

export class AuthService {
public:
    bool verify_token(const std::string& token) const {
        // Uses internal_crypto internally
        return compute_hash(token.c_str()) != 0;
    }
};

```

In this scenario, code executing `import auth_service;` sees only `AuthService`, while the `compute_hash` identifier remains completely invisible.

## Combining Modules with Namespaces

The **Modules** feature does not replace `namespace`; it complements it. Modules serve as physical and logical units of compilation and encapsulation, whereas namespaces prevent name collisions at the semantic level.

Crucially, **a module does not create a new namespace**. If you export a function within the `my_lib` module without a namespace, it resides in the global namespace. Therefore, exported code inside a module should always be wrapped in an appropriate `namespace`.

```cpp
// geometry.ixx
export module geometry;

// Exporting an entire namespace block
export namespace geom {
    constexpr double pi = 3.141592653589793;

    double circle_area(double radius) {
        return pi * radius * radius;
    }

    namespace details {
        // Even inside a namespace, items not marked export remain module-local
        // unless the enclosing namespace block itself is exported!
    }
}

```

Note that exporting an entire namespace block via `export namespace geom { ... }` exports **all** entities within that block. If part of the namespace should remain private, avoid exporting the whole block and instead explicitly mark individual entities with `export`:

```cpp
// geometry_selective.ixx
export module geometry_selective;

namespace geom {
    // Not exported: invisible outside the module
    constexpr double secret_factor = 0.42;

    // Exported: visible to importers
    export double compute(double x) {
        return x * secret_factor;
    }
}

```

## Implementing Custom Class Templates in Modules

Templates in C++ require their complete definitions to be available to the compiler due to compile-time instantiation mechanics. In the traditional header model, you had to place all template code in `.h` files, exposing private helper functions and slowing down compilation.

In C++20 modules, class and function templates can be defined in the module interface file (`export`), while non-template helper functions or internal types utilized by the template do not need to be exported. When instantiating a template in consumer code, the compiler accesses those non-exported functions via information stored in the module's binary file (BMI)—a concept known as reachability.

```cpp
// custom_container.ixx
module;
#include <cstddef>
#include <stdexcept>
#include <utility>

export module custom_container;

// Helper function NOT exported, yet reachable when FixedBuffer is instantiated
void throw_out_of_range() {
    throw std::out_of_range("Index out of bounds in FixedBuffer");
}

export template <typename T, std::size_t Capacity>
class FixedBuffer {
public:
    constexpr std::size_t capacity() const noexcept {
        return Capacity;
    }

    void push_back(T value) {
        if (size_ >= Capacity) {
            throw_out_of_range(); // Calls non-exported helper
        }
        data_[size_++] = std::move(value);
    }

    const T& operator[](std::size_t index) const {
        if (index >= size_) {
            throw_out_of_range();
        }
        return data_[index];
    }

    std::size_t size() const noexcept {
        return size_;
    }

private:
    T data_[Capacity]{};
    std::size_t size_ = 0;
};

```

Consuming this class template is structured as follows:

```cpp
// main.cpp
import <iostream>;
import custom_container;

int main() {
    FixedBuffer<int, 5> buffer;
    buffer.push_back(10);
    buffer.push_back(20);

    std::cout << "First element: " << buffer[0] << '\n';
    // Error: throw_out_of_range is not visible to the caller
    // throw_out_of_range();
    return 0;
}

```

## Key Differences Between Visibility and Reachability

One of the most subtle and vital concepts in the C++20 modules standard is the distinction between **Visibility** and **Reachability**. Understanding this difference is essential for avoiding puzzling compilation errors.

* **Visibility:** Determines whether an identifier's name can be found via name lookup in consumer code. Only entities marked with `export` are visible.
* **Reachability:** Determines whether the compiler is permitted to utilize a type's semantic properties (such as memory size, methods, or data members), even if the name of that type is not visible in consumer code.

This distinction enables you to write an exported function that returns an object of a **non-exported** type! Consumers cannot write that type's name directly (because it is not visible), but they can receive the object using the `auto` keyword and invoke its public methods (because its type is reachable).

```cpp
// session_manager.ixx
module;
#include <string>

export module session_manager;

// NOT exported: name 'SessionToken' is NOT visible outside this module
class SessionToken {
public:
    explicit SessionToken(std::string id) : id_(std::move(id)) {}
    
    const std::string& id() const noexcept {
        return id_;
    }

private:
    std::string id_;
};

// Exported factory function returning a non-exported type
export SessionToken create_session(const std::string& user) {
    return SessionToken("TOKEN_" + user);
}

```

In the consuming file, the compiler behaves as follows:

```cpp
// main.cpp
import <iostream>;
import session_manager;

int main() {
    // Valid: SessionToken is reachable, so 'auto' deduces its type
    // and calling .id() is completely legal.
    auto token = create_session("ali");
    std::cout << token.id() << '\n';

    // Invalid (Compiler Error): 'SessionToken' is not visible by name!
    // SessionToken explicit_token("123");
    
    return 0;
}

```

## Using Regular .cpp Files Alongside Modules

Migrating a large project to modules all at once is virtually impossible. C++20 is designed so that traditional `.cpp` and `.h` files can interoperate seamlessly with modules:

* **Calling modules from traditional `.cpp` files:** Any regular `.cpp` file (even if it is not part of any module) can consume modules via `;import MyModule`.
* **Using traditional `.h` files within modules:** Placing `#include` directives within the **Global Module Fragment** (between `;module` and `;export module Name`) allows modules to leverage legacy libraries (such as C libraries, OS interfaces, or third-party non-module libraries).
* **Header Units:** C++20 allows importing standard or legacy header files directly using `;import <vector>` or `;import "legacy_header.h"`. This converts the header into a binary unit offering the speed benefits of modules, though unlike true modules, macros defined within them still leak into importing code.

The following example demonstrates combining legacy libraries and modules:

```cpp
// legacy_wrapper.ixx
module; // Start of Global Module Fragment
#include <cstdio>
#include <cstdlib>

export module legacy_wrapper; // End of Global Module Fragment

export void print_formatted(const char* text) {
    std::printf("Legacy Output: %s\n", text);
}

```

## Do's and Don'ts in Modules

Adhering to the following guidelines when designing and developing module-based projects prevents architectural faults and build system issues:

| Category | Do's | Don'ts |
| --- | --- | --- |
| **File Structure** | Always place all `#include` directives inside the **Global Module Fragment** (before `export module`). | Never write `#include` directives after the `export module` declaration; doing so transfers ownership of header symbols to the module and triggers ODR linking errors. |
| **Macro Management** | Use exported `constexpr` and `inline` constructs for constants and small functions. | Do not expect `#define` macros to propagate to other files via `import`; macros stop at module boundaries. |
| **Namespaces** | Always place exported entities within a designated `namespace`. | Never write `import` directives inside a `namespace` block; `import` statements must always reside at the global scope of a file. |
| **Code Separation** | Keep the module interface clean and concise by moving implementations to `.cpp` files or the `module :private` section. | Do not use `module :private` in multi-file projects; this feature is restricted to single-file modules. |
| **Dependencies** | Design module dependencies as a Directed Acyclic Graph (DAG). | Avoid circular dependencies between modules (e.g., module A imports module B and vice versa); this is prohibited by the standard. |
| **Inline Functions** | Write the body of exported `inline` functions directly inside the module interface file. | Do not place the body of an `export inline` function in a separate implementation (`.cpp`) file, as its definition must be available to all module consumers. |

## Limitations and Practical Build Tool Considerations

Although modules are standardized linguistically in C++20, tool and build system support (such as CMake, Ninja, MSBuild, GCC, Clang, and MSVC) involves unique complexities:

* **Compilation Order of Dependencies:** Unlike the `#include` model where `.cpp` files could compile completely independently and in parallel, module interface files (`.ixx` or `.cppm`) **must** be compiled before any file that imports them in order to generate the BMI binary file.
* **CMake Support:** To automatically manage module dependencies in real-world projects, using **CMake 3.28 or higher** alongside the **Ninja** generator (or Visual Studio) is recommended to handle dependency scanning automatically.
* **Standard Library Module (`import std;`):** Importing the entire standard library via a simple `;import std` directive was officially introduced in the **C++23** standard (though modern compilers provide it as an extension in C++20 mode). Under standard C++20, header units like `;import <vector>` are used for the standard library.

# **C++20 Modules**

## **Introduction**

Modules are one of the most significant architectural additions introduced in C++20. They provide a modern mechanism for organizing, sharing, and compiling code in C++ projects.

For many years, the primary mechanism for sharing declarations between different Translation Units was the use of Header files and the `#include` directive. Although this approach is still widely used, it introduces several problems:

- **Increased compilation time:** Header contents are processed again by the Preprocessor in every Translation Unit.
- **Unnecessary dependencies:** Including a Header may indirectly include many other Headers through transitive dependencies.
- **Macro-related problems:** Macros do not have a well-defined visibility boundary and can affect code outside their intended scope.
- **ODR-related problems:** Managing definitions in Header files, particularly in large projects, requires considerable care.
- **Implementation detail exposure:** Consumers of a Library often gain access to declarations and dependencies they do not need to see.
- **Include-order dependencies:** In some projects, the order in which Headers are included can affect whether compilation succeeds.

Modules introduce a new model for sharing code and help address many of these problems.

Instead of processing a Header's contents again in every Translation Unit, a Module is compiled independently. The compiler makes the resulting information available to importing Translation Units through a compiler-specific binary representation, commonly called a **BMI (Built Module Interface)**.

As a result, consumers can use the `import` directive instead of `#include`.

### **A Simple Example**

To understand the difference between these two approaches, let's first examine a simple Module.

**File: Math.cppm**

```
export module Math;

export int add(int a, int b)
{
    return a + b;
}
```

**File: main.cpp**

```
import Math;

#include <iostream>

int main()
{
    std::cout << add(10, 20) << '\n';
}
```

In this example:

- `export module Math;` declares a Module named `Math`.
- `export` specifies that the declaration of the `add` function is part of the Module's public Interface.
- `import Math;` makes the Module's exported declarations available.

Notice that the consumer does not need to see the contents of `Math.cppm`. It only needs the compiled Module Interface.

## **Understanding Modules in C++**

In C++20, a Module is a collection of Translation Units organized under a common name.

Each Translation Unit belonging to a Module can access declarations and definitions associated with that same Module.

A Module can consist of one or multiple files.

For example:

```
Math
 ├── Math.cppm
 ├── Math.cpp
 └── MathInternal.cpp
```

In this structure, all three files can belong to the Module named `Math`.

However, simply placing files in the same directory or giving them similar names does not make them part of the same Module. This relationship must be explicitly established through the appropriate Module Declaration in each Translation Unit.

### **Types of Module Units**

In C++20, the units that make up a Module fall into several categories:

| Type                            | Description                                                      |
| ------------------------------- | ---------------------------------------------------------------- |
| Primary Module Interface Unit   | The main public Interface of a Module                            |
| Module Interface Partition      | An independently defined part of the Module Interface            |
| Module Implementation Unit      | A unit used to implement Module functionality                    |
| Module Implementation Partition | An internal part of a Module used to organize its implementation |

Every named Module must have exactly one Primary Module Interface Unit.

This Interface serves as the entry point for external consumers.

## **Introduction to the Main Module Syntax**

Before examining more advanced structures, let's become familiar with the main Module-related syntax.

| Syntax                | Purpose                                    |
| --------------------- | ------------------------------------------ |
| `export module M;`    | Defines a Primary Module Interface         |
| `module M;`           | Defines a Module Implementation Unit       |
| `export module M:P;`  | Defines an Interface Partition             |
| `module M:P;`         | Defines an Implementation Partition        |
| `import M;`           | Imports a Module                           |
| `import :P;`          | Imports a Partition of the current Module  |
| `export import M;`    | Imports and re-exports a Module            |
| `export import :P;`   | Imports and exports an Interface Partition |
| `export declaration;` | Exports a declaration                      |
| `export { ... }`      | Exports a group of declarations            |
| `module;`             | Starts the Global Module Fragment          |
| `module : private;`   | Starts the Private Module Fragment         |

The following sections examine how each of these constructs works.

## **Understanding Export and Import**

The two fundamental keywords in the Module system are `export` and `import`.

### **Using Export**

The `export` keyword specifies that a declaration is part of the Module's Interface and can be made available to external consumers.

By default, declarations defined within a named Module are available only within that Module and are not automatically exposed to consumers.

For example:

```
export module Math;

int internalAdd(int a, int b)
{
    return a + b;
}

export int add(int a, int b)
{
    return internalAdd(a, b);
}
```

In this example:

- The `add` function is exported.
- The `internalAdd` function is not exported.

Therefore, consumers can use `add`, but they cannot directly call `internalAdd`.

**File: main.cpp**

```
import Math;

int main()
{
    int result = add(10, 20);

    // Error: internalAdd is not exported
    // int value = internalAdd(10, 20);
}
```

It is important to understand that not exporting a declaration does not mean that it does not exist during compilation or linking. This distinction is related to the concepts of visibility and reachability, which will be discussed later.

### **Using Export Blocks**

To export multiple declarations, you can use an Export Block instead of repeating the `export` keyword for every declaration.

```
export module Math;

export
{
    int add(int a, int b)
    {
        return a + b;
    }

    int subtract(int a, int b)
    {
        return a - b;
    }

    int multiply(int a, int b)
    {
        return a * b;
    }
}
```

In this example, all three functions become part of the Module's public Interface.

Export Blocks are useful when several related declarations need to be grouped together as part of the public Interface.

An Export Block is simply a syntactic mechanism for grouping declarations. It does not introduce behavior different from exporting each declaration individually.

### **Using Import**

To use declarations exported by a Module, use the `import` directive.

```
import Math;
import Logger;
import Database;

int main()
{
    auto result = add(10, 20);

    return 0;
}
```

An Import Declaration must appear at namespace scope and, within a Module Unit, after the Module Declaration and before other ordinary declarations.

### **Importing Multiple Modules Separately or as a Group**

C++20 does not provide a dedicated syntax for importing multiple Modules through a single Import Declaration.

The conventional approach is to write separate Import Declarations:

```
import Math;
import Logger;
import Database;
```

Although these imports can appear next to one another, each remains an independent Import Declaration.

You can also re-export imported Modules through a Module Interface. This allows consumers to access their exported declarations without importing those dependencies directly.

```
export module Application;

export import Math;
export import Logger;
```

Consumers can now import only the Application Module:

```
import Application;

int main()
{
    auto result = add(10, 20);

    return 0;
}
```

In this example, exporting the imports makes the public Interfaces of `Math` and `Logger` available through `Application`.

This capability is particularly useful when designing a Facade Module.

## **Implementing a Module in a Single File**

The simplest way to use Modules is to place the Interface and Implementation in the same file.

In this approach, public declarations and their corresponding definitions are written in a single Translation Unit.

**File: Math.cppm**

```
export module Math;

export int add(int a, int b)
{
    return a + b;
}

export int subtract(int a, int b)
{
    return a - b;
}

int internalCalculation(int value)
{
    return value * 2;
}
```

**File: main.cpp**

```
import Math;

#include <iostream>

int main()
{
    std::cout << add(10, 20) << '\n';
    std::cout << subtract(20, 10) << '\n';
}
```

In this structure, all Module components are contained in one file.

### **Advantages**

- Simple project structure.
- No need to manage multiple files for small Modules.
- Suitable for simple Utility Modules.
- Useful for prototypes and small projects.

### **Disadvantages**

- The file can become large as the Module grows.
- Interface and Implementation become mixed together.
- Maintenance and development become more difficult in complex projects.
- Changes to definitions in the same file require recompiling the Module Interface.

For Modules with complex implementations, separating the Interface and Implementation is usually a better choice.

## **Separating the Interface and Implementation into Different Files**

One important feature of Modules is the ability to separate the public Interface from its Implementation.

Architecturally, this resembles the traditional separation between Header and Source files, although the mechanism connecting the files is different.

In this approach, the Interface is placed in one file, while the corresponding definitions are placed in one or more other files.

### **Defining the Interface**

**File: Math.cppm**

```
export module Math;

export int add(int a, int b);

export int subtract(int a, int b);
```

This file contains only the public declarations.

### **Defining the Implementation**

**File: Math.cpp**

```
module Math;

int add(int a, int b)
{
    return a + b;
}

int subtract(int a, int b)
{
    return a - b;
}
```

Notice the following declaration:

```
module Math;
```

This specifies that the current file is a Module Implementation Unit belonging to the Module named `Math`.

Consequently, the definitions in this file are associated with declarations belonging to the same Module.

### **Using the Module**

**File: main.cpp**

```
import Math;

#include <iostream>

int main()
{
    std::cout << add(10, 20) << '\n';
    std::cout << subtract(20, 10) << '\n';
}
```

In this structure, consumers need only the Interface and do not need to know where the Implementation is located.

### **An Important Difference Between Interface and Implementation**

The Interface file uses:

```
export module Math;
```

However, the Implementation file uses:

```
module Math;
```

The presence or absence of the `export` keyword in the Module Declaration is fundamental.

| Feature                                | Interface Unit        | Implementation Unit |
| -------------------------------------- | --------------------- | ------------------- |
| Module declaration                     | `export module Math;` | `module Math;`      |
| Exporting declarations                 | Allowed               | Not allowed         |
| Importing other Modules                | Allowed               | Allowed             |
| Containing definitions                 | Allowed               | Allowed             |
| Accessing internal Module declarations | Allowed               | Allowed             |

An Implementation Unit can use declarations exported by the Interface, but it cannot introduce a new public Interface for the same Module.

## **Using Multiple Implementation Files**

A Module can have multiple Implementation Units.

This capability is particularly useful for projects where different parts of the implementation have separate responsibilities.

Suppose we have a Module named `Database` responsible for several operations, such as managing connections, executing queries, and handling transactions.

We can organize the project as follows:

```
Database/
│
├── Database.cppm
│
├── Connection.cpp
├── Query.cpp
├── Transaction.cpp
│
└── main.cpp
```

**File: Database.cppm**

```
export module Database;

export class Connection
{
public:
    void connect();
};

export class Query
{
public:
    void execute();
};

export class Transaction
{
public:
    void begin();
    void commit();
};
```

**File: Connection.cpp**

```
module Database;

void Connection::connect()
{
}
```

**File: Query.cpp**

```
module Database;

void Query::execute()
{
}
```

**File: Transaction.cpp**

```
module Database;

void Transaction::begin()
{
}

void Transaction::commit()
{
}
```

In this structure, all Implementation files belong to the same Module.

Therefore, they can define the implementations of declarations belonging to that Module in separate files.

### **An Important Note About Multiple Implementation Units**

Having multiple Implementation files for a Module is perfectly valid.

However, each Module must still have only one Primary Module Interface Unit.

Also, definitions of the same entity must not be duplicated across multiple Translation Units in a way that violates the ODR.

For example, defining the same ordinary function in multiple Implementation Units with conflicting definitions does not become valid simply because those files belong to the same Module.

## **Managing Multiple Interface Files**

In large projects, a Module's public Interface may become too extensive to maintain all declarations in a single file.

To address this, C++20 provides a feature called **Module Interface Partitions**.

This feature allows a Module's Interface to be divided into multiple independent files.

Each Partition can have a specific responsibility, while all Partitions remain part of the same named Module.

For example:

```
Graphics/
│
├── Graphics.cppm
│
├── Graphics-Renderer.cppm
├── Graphics-Texture.cppm
├── Graphics-Shader.cppm
│
├── Renderer.cpp
├── Texture.cpp
└── Shader.cpp
```

In this structure:

- `Graphics.cppm` is the primary Module Interface.
- `Graphics-Renderer.cppm` contains the Interface related to Renderer.
- `Graphics-Texture.cppm` contains the Interface related to Texture.
- `Graphics-Shader.cppm` contains the Interface related to Shader.

The important point is that these files are not independent Modules. They are Interface Partitions of a single Module.

The following sections explain how to define and use these Partitions in detail.

## **Understanding Sub-Modules and Their Difference from Module Partitions**

One topic that can cause confusion when designing a Module structure is the concept of Sub-Modules.

C++20 does not provide a separate mechanism called Sub-Modules that represents a hierarchical structure within a Module.

For example, we can define two Modules with the following names:

```
export module Graphics;
```

```
export module Graphics.Renderer;
```

However, the dot in the second Module's name does not automatically make `Graphics.Renderer` part of `Graphics`.

These are two independent named Modules.

If we want to organize different parts of a single Module, we should use Module Partitions.

For example:

```
export module Graphics:Renderer;
```

Here, `Renderer` is a Partition of the primary Module `Graphics`.

The differences can be summarized as follows:

| Feature                                       | Independent Named Module | Module Partition    |
| --------------------------------------------- | ------------------------ | ------------------- |
| Example                                       | `Graphics.Renderer`      | `Graphics:Renderer` |
| Belongs to a primary Module                   | No                       | Yes                 |
| Shares Module ownership                       | No                       | Yes                 |
| Can be imported directly from outside         | Yes                      | No                  |
| Requires its own Primary Interface            | Yes                      | No                  |
| Can be re-exported through the primary Module | Using `export import`    | Yes                 |

Therefore, Module Partitions are generally the better choice when dividing a large Module into internal components.

## **Introduction to Module Interface Partitions**

A Module Interface Partition is a part of a Module's public Interface defined in a separate file.

The following syntax is used to define one:

```
export module ModuleName:PartitionName;
```

The presence of `export` at the beginning of the Module Declaration indicates that this Partition is an Interface Unit.

### **A Practical Example**

Suppose we want to divide a Graphics Module into two parts: Renderer and Texture.

**File: Graphics-Renderer.cppm**

```
export module Graphics:Renderer;

export class Renderer
{
public:
    void render();
};
```

**File: Graphics-Texture.cppm**

```
export module Graphics:Texture;

export class Texture
{
public:
    void load();
};
```

We now have two independent Interface Partitions.

However, for external consumers to access these capabilities through the primary Module, we must introduce them in the Primary Interface.

**File: Graphics.cppm**

```
export module Graphics;

export import :Renderer;
export import :Texture;
```

In this example:

- The primary Module `Graphics` is defined.
- The Renderer Partition is imported and re-exported.
- The Texture Partition is also imported and re-exported.

Consumers can now import only the primary Module.

**File: main.cpp**

```
import Graphics;

int main()
{
    Renderer renderer;
    Texture texture;

    renderer.render();
    texture.load();
}
```

Consumers do not need to know the names of the Partitions.

### **An Important Note About Importing Partitions**

Module Partitions can only be imported by Translation Units belonging to the same named Module.

Therefore, the following statement is not valid in an external Translation Unit:

```
import Graphics:Renderer;
```

However, it can be used inside the Module itself:

```
export module Graphics;

import :Renderer;
```

Also note that an Interface Partition must be exported, directly or indirectly, by the Primary Interface for its Interface to become available through importing the primary Module.

## **Difference Between Interface Partitions and Implementation Partitions**

In addition to Interface Partitions, you can define Partitions used exclusively for organizing the internal Implementation.

The main difference is whether the `export` keyword is present.

### **Defining an Interface Partition**

```
export module Graphics:Renderer;

export class Renderer
{
public:
    void render();
};
```

This Partition can contain public declarations.

### **Defining an Implementation Partition**

```
module Graphics:Internal;

void initializeRenderer()
{
}
```

This Partition is part of the Module's internal implementation.

To use it in the Primary Interface or other units belonging to the same Module, it can be imported.

**File: Graphics.cppm**

```
export module Graphics;

import :Internal;

export void initializeGraphics()
{
    initializeRenderer();
}
```

Since `initializeRenderer` is not exported, external consumers cannot access it.

### **An Important Detail**

In Module Partitions, declarations and definitions from an imported Partition are reachable within the importing units of the same Module, even if they are not exported.

This creates an important distinction between Partitions and independent named Modules.

In practice, Partitions share the ownership of their parent Module.

Therefore:

- Internal Partitions can contain shared implementation details.
- The Primary Interface can access their internal declarations.
- External consumers cannot directly import internal Partitions.
- Non-exported declarations do not become part of the public Interface merely because the primary Module is imported.

This capability is particularly useful for building a powerful internal structure while keeping the public Interface small.

## **Using Re-Imports and Export Import**

One important feature of Modules is the ability to export an Import.

This allows us to expose the Interfaces of multiple Modules or Partitions through a single Module.

For example, suppose we have three independent Modules:

```
export module Math;
```

```
export module Geometry;
```

```
export module Physics;
```

We can now create a higher-level Module.

**File: Engine.cppm**

```
export module Engine;

export import Math;
export import Geometry;
export import Physics;
```

In this case, any consumer importing `Engine` also gains access to the declarations exported by these three Modules.

**File: main.cpp**

```
import Engine;

int main()
{
    auto result = add(10, 20);

    return 0;
}
```

This pattern is suitable for designing Facade Modules.

### **Difference Between Import and Export Import**

| Statement             | Behavior                                                                          |
| --------------------- | --------------------------------------------------------------------------------- |
| `import Math;`        | Makes the Module's capabilities available in the current Translation Unit         |
| `export import Math;` | Imports the Module and re-exports its public Interface through the current Module |

Note that Export Import does not transfer ownership of declarations to the new Module.

Each declaration remains associated with its original Module.

## **Using Classes in Modules**

Using Classes in Modules is not fundamentally different from using them in traditional C++.

The main difference is that instead of placing declarations in a Header, we define or declare them in a Module Interface.

### **Defining a Complete Class in the Interface**

```
export module Person;

export class Person
{
public:
    Person(const char* name);

    void print() const;

private:
    const char* name_;
};
```

**File: Person.cpp**

```
module Person;

#include <iostream>

Person::Person(const char* name)
    : name_(name)
{
}

void Person::print() const
{
    std::cout << name_ << '\n';
}
```

**File: main.cpp**

```
import Person;

int main()
{
    Person person("Hadi");

    person.print();
}
```

In this example, the Class is declared in the Interface, while its member function definitions are placed in the Implementation file.

### **Exporting a Class**

To make a Class available to external consumers, we can export the Class itself.

```
export module User;

export class User
{
public:
    void login();
    void logout();

private:
    int id_;
};
```

Exporting a Class makes its public declaration and accessible members part of the exported Interface.

However, exporting a Class does not make all its internal details publicly accessible.

For example, `private` members remain subject to the usual Access Control rules.

## **Using Class Templates in Modules**

One important aspect of using Modules is managing Templates.

In traditional C++, Template definitions are usually placed in Header files because the compiler needs their definitions to instantiate them.

The same principle applies to Modules.

If we export a Class Template, consumers must have access to the definitions required for its instantiation.

### **Defining a Class Template in the Interface**

**File: Container.cppm**

```
export module Container;

export template <typename T>
class Container
{
public:
    explicit Container(T value)
        : value_(value)
    {
    }

    T get() const
    {
        return value_;
    }

private:
    T value_;
};
```

**File: main.cpp**

```
import Container;

#include <iostream>

int main()
{
    Container<int> numbers(100);
    Container<double> values(25.5);

    std::cout << numbers.get() << '\n';
    std::cout << values.get() << '\n';
}
```

In this example, the compiler has access to the complete Class Template definition and can instantiate the required specializations.

### **Separating Template Definitions**

Suppose we want to separate the public Template declarations from their definitions.

We can use an Interface Partition.

**File: Container-Template.cppm**

```
export module Container:Template;

export template <typename T>
class Container
{
public:
    explicit Container(T value);

    T get() const;

private:
    T value_;
};

template <typename T>
Container<T>::Container(T value)
    : value_(value)
{
}

template <typename T>
T Container<T>::get() const
{
    return value_;
}
```

**File: Container.cppm**

```
export module Container;

export import :Template;
```

In this example, the Template definitions are placed in an Interface Partition.

Consumers importing `Container` gain access to the Interface and the definitions required for Template instantiation.

### **Can Template Definitions Be Placed in a Regular cpp File?**

In general, you cannot simply place a Template definition in an Implementation Unit and expect external consumers to instantiate it for arbitrary Types.

For example, if the following definition exists only in an Implementation Unit:

```
module Container;

template <typename T>
T calculate(T value)
{
    return value * 2;
}
```

External consumers do not have access to the definition required to instantiate this Template.

However, there is an alternative: Explicit Instantiation.

For example:

```
export module Container;

export template <typename T>
T calculate(T value);
```

**File: Container.cpp**

```
module Container;

template <typename T>
T calculate(T value)
{
    return value * 2;
}

template int calculate<int>(int);
template double calculate<double>(double);
```

With this design, only the explicitly specified specializations are instantiated.

This approach is suitable when the supported Types are known and limited.

However, if consumers need to instantiate the Template for arbitrary Types, it is generally better to make the required definitions available in the Interface or an accessible Interface Partition.

### **An Architectural Note About Templates**

For public Templates, it is generally preferable not to separate the Interface from the definitions required for instantiation unless there is a specific reason to do so.

Modules do not magically eliminate the need for Template definitions during instantiation.

## **Using Namespaces in Modules**

Modules and Namespaces are independent mechanisms with different purposes.

- Namespaces organize names and help prevent name collisions.
- Modules organize Translation Units and control how declarations are exposed between them.

Therefore, Namespaces can be used inside Modules.

### **Defining a Namespace in a Module**

```
export module Math;

export namespace Mathematics
{
    int add(int a, int b)
    {
        return a + b;
    }

    int subtract(int a, int b)
    {
        return a - b;
    }
}
```

In this example, the `Mathematics` Namespace and its declarations are exported.

**File: main.cpp**

```
import Math;

int main()
{
    auto result = Mathematics::add(10, 20);

    return 0;
}
```

### **Exporting Specific Namespace Members**

You do not always need to export every member of a Namespace.

For example:

```
export module Math;

namespace Mathematics
{
    export int add(int a, int b)
    {
        return a + b;
    }

    int internalCalculation(int value)
    {
        return value * 2;
    }
}
```

In this structure, the `Mathematics` Namespace is accessible through the declaration of `add` in the exported Interface, but `internalCalculation` is not exported.

Consumers can use `add`, but they cannot directly access the internal function.

### **Exporting an Entire Namespace**

If we want to export all declarations within a Namespace, we can use the following approach:

```
export module Math;

export namespace Mathematics
{
    int add(int a, int b)
    {
        return a + b;
    }

    int subtract(int a, int b)
    {
        return a - b;
    }
}
```

In this case, all declarations within the Namespace definition that are subject to the Export rules become part of the public Interface.

### **Using Internal Namespaces**

We can also define Namespaces used exclusively for internal implementation.

```
export module Math;

namespace Internal
{
    int normalize(int value)
    {
        return value < 0 ? 0 : value;
    }
}

export int calculate(int value)
{
    return Internal::normalize(value) * 2;
}
```

In this example, the internal Namespace is not part of the public Interface, and consumers do not need direct access to it.

## **Understanding Visibility and Reachability in Modules**

One of the most important and, at the same time, complex aspects of the Module system is the distinction between Visibility and Reachability.

To understand Modules correctly, we should not simply treat Export as equivalent to Public and non-Export as equivalent to Private.

These concepts are related, but they are not identical.

### **Understanding Visibility**

Visibility determines whether a declaration can be seen and used by Name Lookup in a particular context.

For example:

```
export module Math;

export int add(int a, int b);

int internalAdd(int a, int b);
```

In this example, an external consumer can see the declaration of `add`, but the declaration of `internalAdd` is not made available through the public Interface.

### **Understanding Reachability**

Reachability is a broader concept than Visibility.

In C++, Reachability determines whether a declaration or definition is accessible at a particular point according to the language rules.

This is particularly important for:

- Template Instantiation
- Name Lookup
- ODR
- Class definitions
- Exported declarations
- Module Partitions

A declaration may not be directly visible to Name Lookup but may still be reachable.

For this reason, these two concepts should not be treated as interchangeable.

### **Difference Between Visibility, Reachability, and Access Control**

Another concept that must be distinguished from these two is Access Control.

Access Control is the familiar mechanism involving `public`, `protected`, and `private` in Classes.

For example:

```
export module User;

export class User
{
public:
    void login();

private:
    int password_;
};
```

In this example:

- The `User` Class is exported.
- The declaration of `login` is part of the public Interface.
- The `password_` member remains private.

Therefore, even when a Class is accessible to consumers, Access Control continues to enforce its own restrictions.

### **Comparing These Concepts**

| Concept        | Responsibility                                                                 |
| -------------- | ------------------------------------------------------------------------------ |
| Export         | Exposes a declaration through the public Interface                             |
| Visibility     | Determines whether a declaration can be seen by Name Lookup                    |
| Reachability   | Determines whether a declaration is accessible according to the language rules |
| Access Control | Controls access to Class members                                               |
| Linkage        | Determines how an entity is associated across Translation Units                |

These concepts must be considered separately when designing Modules.

## **Understanding Linkage in Modules**

In traditional C++, we generally work with two main types of Linkage:

- Internal Linkage
- External Linkage

With the introduction of Modules, another type was added:

**Module Linkage**

In simple terms, declarations defined within a named Module and subject to Module Linkage can be associated across Translation Units belonging to that same Module without becoming publicly available entities with External Linkage in other Modules.

For example:

**File: Math.cppm**

```
export module Math;

int internalValue = 100;

export int getValue()
{
    return internalValue;
}
```

**File: Math.cpp**

```
module Math;

int calculate()
{
    return internalValue * 2;
}
```

In this example, `internalValue` is not directly exported, but Translation Units belonging to the `Math` Module can access it.

This is one of the important differences between the Module model and the traditional Header-based approach.

## **Using cpp Files Alongside Modules**

One common question when migrating from the traditional Header/Source architecture to Modules is whether regular `.cpp` files can still be used.

The answer is yes.

Modules do not require a particular file extension.

Extensions such as the following are simply common conventions used by tools and build systems:

| Extension | Common Usage                           |
| --------- | -------------------------------------- |
| `.cppm`   | Module Interface                       |
| `.ixx`    | Module Interface in the MSVC ecosystem |
| `.cpp`    | Implementation                         |
| `.cc`     | Implementation                         |
| `.cxx`    | Implementation                         |

The important point is that the compiler and Build System must know the role of each file.

### **Using cpp Files for Implementation**

**File: Logger.cppm**

```
export module Logger;

export class Logger
{
public:
    void log(const char* message);
};
```

**File: Logger.cpp**

```
module Logger;

#include <iostream>

void Logger::log(const char* message)
{
    std::cout << message << '\n';
}
```

In this example, the `.cpp` file is recognized as a Module Implementation Unit.

### **Using Regular cpp Files Alongside Modules**

It is also possible to have files that are not Module Units at all.

For example:

**File: Utility.cpp**

```
#include <iostream>

void printMessage()
{
    std::cout << "Hello\n";
}
```

This file is a regular Translation Unit and does not belong to any particular Module.

If we want to use its declarations from a Module, we must design an appropriate mechanism for connecting them, such as using a Header or another compatible approach.

The important point is that simply placing a `.cpp` file alongside Module files does not make it part of the corresponding Module.

To belong to a named Module, it must contain the appropriate Module Declaration.

## **Introduction to the Global Module Fragment**

Sometimes, we still need traditional Headers when working with Modules.

For example, an older Library may only be available through a Header, or we may need macros defined in an existing Header.

In such situations, we can use a feature called the **Global Module Fragment**.

The following declaration starts the Global Module Fragment:

```
module;
```

It must appear before the Module Declaration.

### **A Practical Example**

```
module;

#include <vector>
#include <string>

#define MAX_USERS 100

export module UserManager;

export class UserManager
{
public:
    void addUser(const std::string& name);

private:
    std::vector<std::string> users_;
};
```

In this example:

- The Global Module Fragment is started.
- The required Headers are included in this section.
- The primary Module is then declared.
- The `UserManager` Class is exported.

### **Why Does the Global Module Fragment Exist?**

Under normal circumstances, declarations from traditional Headers included within a named Module may have Ownership and Linkage characteristics that do not match the expectations of traditional Libraries.

The Global Module Fragment provides a way to use existing Headers without making their declarations part of the named Module in the usual way.

Macros defined in this section can also be used later in the same Translation Unit.

However, macros are not made available to consumers through Module Import.

### **An Important Note**

The Global Module Fragment should not be considered a permanent replacement for proper Module design.

It is primarily useful for compatibility with legacy code and dependencies that still rely on the Header mechanism.

## **Introduction to the Private Module Fragment**

One of the less familiar features of Modules in C++20 is the **Private Module Fragment**.

It is introduced using the following declaration:

```
module : private;
```

The Private Module Fragment is a section of the Primary Module Interface Unit that appears after the public portion.

Declarations and definitions in this section are not accessible to external consumers of the Module.

### **A Practical Example**

**File: Math.cppm**

```
export module Math;

export int add(int a, int b);

module : private;

int internalCalculation(int value)
{
    return value * 2;
}

int add(int a, int b)
{
    return internalCalculation(a + b);
}
```

In this example:

- The `add` function is declared in the public section of the Module.
- The Private Module Fragment is started.
- The `internalCalculation` function is defined in the private section.
- The definition of `add` is also placed in this section.

Consumers can still call `add`, but they cannot access the internal function.

**File: main.cpp**

```
import Math;

int main()
{
    return add(10, 20);
}
```

### **An Important Limitation of the Private Module Fragment**

If a Module Unit contains a Private Module Fragment, that Module must consist of only that single Module Unit.

In other words, you cannot define multiple independent Implementation Units or Partitions for the same Module alongside the unit containing the Private Module Fragment.

Therefore, this feature is more suitable for small, single-file Modules.

### **Difference Between the Private Module Fragment and an Implementation Unit**

| Feature                                      | Private Module Fragment                     | Implementation Unit                             |
| -------------------------------------------- | ------------------------------------------- | ----------------------------------------------- |
| Location                                     | At the end of the Primary Interface Unit    | Separate Translation Unit                       |
| Requires a separate file                     | No                                          | Yes                                             |
| Supports multiple Implementation files       | No                                          | Yes                                             |
| Supports dividing the Module into Partitions | No                                          | Yes                                             |
| Purpose                                      | Separates private code within the same file | Separates Implementation into independent files |

For large projects, using separate Implementation Units is generally more flexible.

## **Using an Internal Module Within an Exported Module**

One important application of Module Partitions is creating internal components that external consumers cannot directly access.

Suppose we have a Module named `Engine` containing both public and internal functionality.

We want to:

- Make the public Engine Interface available to consumers.
- Place the internal Implementation in a separate file.
- Prevent internal declarations from being directly available to other Modules.
- Allow consumers to access only the public API.

We can achieve this using an Internal Module Partition.

### **Defining an Internal Partition**

**File: Engine-Internal.cppm**

```
module Engine:Internal;

void initializeSubsystem()
{
}

int calculateInternalValue()
{
    return 100;
}
```

Notice that this file does not use `export module`.

Therefore, it is an Internal Module Partition.

### **Using the Internal Partition**

**File: Engine.cppm**

```
export module Engine;

import :Internal;

export class Engine
{
public:
    void initialize()
    {
        initializeSubsystem();
    }

    int getValue() const
    {
        return calculateInternalValue();
    }
};
```

In this example:

- The primary `Engine` Module is declared.
- The internal Partition is imported.
- The public `Engine` Class is exported.
- Public member functions use internal functionality.

### **Using the Module from an External Consumer**

**File: main.cpp**

```
import Engine;

int main()
{
    Engine engine;

    engine.initialize();

    return engine.getValue();
}
```

The consumer imports only the primary Module.

The following statement is not allowed in an external Translation Unit:

```
import Engine:Internal;
```

This is because Partitions can only be imported from within their own named Module.

### **An Important Architectural Note**

In this example, the internal Partition is part of the primary Module, not an independent named Module.

For this reason, its declarations are available to Translation Units belonging to the same Module.

However, because the Partition is not publicly exported, external consumers cannot directly access its Interface.

This approach is useful for:

- Internal Helpers
- Internal algorithms
- Implementation details
- Internal Subsystems
- Declarations shared between Implementation Units
- Reducing the size of the public Interface

**Note:** This approach should not be confused with the Private Module Fragment. Internal Module Partitions support multiple files and a modular internal structure, whereas the Private Module Fragment is restricted to a single Module Unit.

## **Comparing Headers and Modules**

One of the main goals of Modules is to reduce the problems associated with traditional Header files.

However, Modules and Headers are not complete replacements for one another. They have different characteristics.

| Feature                   | Headers and Include               | Modules and Import                        |
| ------------------------- | --------------------------------- | ----------------------------------------- |
| Sharing mechanism         | Textual Inclusion                 | Module Import                             |
| Reprocessing declarations | Usually in every Translation Unit | Uses BMI                                  |
| Macros                    | Can be propagated through Include | Not exported by default                   |
| Interface control         | Depends on Header structure       | Controlled through Export                 |
| Implementation separation | Usually through cpp files         | Through Implementation Units              |
| Dependency management     | Primarily based on Include        | Based on Module Dependencies              |
| Include order             | Can be significant                | Dependencies are specified through Import |
| Legacy code compatibility | Extensive                         | May require compatibility mechanisms      |

### **Advantages of Modules**

**Reduced Compilation Time**

One of the main advantages of Modules is that the compiler can use compiled Interface information instead of processing all Header declarations again in every Translation Unit.

However, the actual performance improvement depends on the project structure, dependencies, compiler, and Build System.

**Prevention of Macro Propagation**

Macros defined in a named Module are not automatically transferred to consumers.

This helps prevent many problems related to name collisions and Preprocessor pollution.

**Better Control Over the Public Interface**

Using `export`, we can specify which declarations should be available to consumers.

**Reduced Unnecessary Dependencies**

Instead of exposing all the details contained in Headers, consumers receive only the Module's accessible Interface.

### **Limitations of Modules**

Despite their advantages, Modules also have limitations:

- They require adequate compiler support.
- They require a Build System compatible with Module Dependencies.
- Build configuration can become more complex.
- Compatibility with some older Libraries may be limited.
- BMI files must be managed.
- BMI generation and storage differ between compilers.
- Gradual migration can be difficult in some legacy projects.

Modules also do not necessarily improve compilation speed in every project. If a project has few dependencies and lightweight Headers, the performance improvement may be negligible.

## **Understanding BMI and How Modules Are Compiled**

One of the fundamental differences between Modules and Headers is how they are processed by the compiler.

In the traditional approach, a Header file is directly inserted into a Translation Unit by the Preprocessor.

With Modules, however, the compiler must first process the Module Interface and generate a BMI.

A **BMI (Built Module Interface)** contains compiled information that the compiler needs to use the Module.

For example, the general process can be represented as follows:

```
              Math.cppm
                  |
                  |
              Compiler
                  |
                  |
             Math BMI
                  |
                  |
        +---------+---------+
        |                   |
        |                   |
      main.cpp          Engine.cpp
        |                   |
        +---------+---------+
                  |
                  |
              Compiler
                  |
                  |
             Object Files
                  |
                  |
                Linker
                  |
                  |
              Executable
```

In this model, the Interface is compiled first to generate the BMI.

Files importing the Module can then be compiled using that BMI.

### **Is BMI a Standard and Portable File Format?**

No.

The C++ standard does not define a common binary format for BMI files.

Therefore:

- The BMI format is compiler-specific.
- A BMI generated by one compiler is not necessarily usable by another compiler.
- Even different versions of the same compiler may not have complete BMI compatibility.
- Compilation settings, target platforms, and other Toolchain characteristics can affect BMI compatibility.

Consequently, BMI files should not be treated like standard Object files or Libraries that can be freely shared between different Toolchains.

### **Dependency Management**

In projects using Modules, the Build System must determine compilation order based on Import relationships.

For example, if the `Engine` Module imports the `Math` Module, the BMI for Math must be generated first.

Engine can then be compiled.

This makes Build Systems with proper Module dependency support increasingly important.

## **Using Modules with CMake**

For practical use of Modules in real projects, it is preferable to use Build Systems that can manage Module Dependencies.

One suitable option is CMake.

CMake introduced support for C++ Modules in version 3.28.

In these versions, Modules can be managed through a dedicated File Set named `CXX_MODULES`.

### **Project Structure**

```
MyProject/
│
├── CMakeLists.txt
│
├── Math.cppm
├── Math.cpp
│
└── main.cpp
```

### **CMakeLists.txt**

```
cmake_minimum_required(VERSION 3.28)

project(MyProject LANGUAGES CXX)

add_executable(MyProject
    main.cpp
    Math.cpp
)

target_sources(MyProject
    PUBLIC
    FILE_SET CXX_MODULES
    TYPE CXX_MODULES
    FILES
        Math.cppm
)

target_compile_features(MyProject PRIVATE cxx_std_20)
```

In this example:

- `Math.cppm` is declared as a Module Interface.
- `Math.cpp` is included in the Target as an Implementation file.
- C++20 is enabled.
- CMake can analyze Module Dependencies to determine the correct compilation order.

### **An Important Note About CMake**

Simply using CMake version 3.28 or later does not guarantee complete support for every Module feature.

Support depends on:

- Compiler version
- Selected Generator
- Build Tool version
- Dependency Scanning capabilities
- Current CMake limitations

For example, Generators such as Ninja and Visual Studio provide Module Scanning support, but their capabilities are not identical.

Therefore, it is advisable to verify complete Toolchain compatibility before choosing a project structure.

## **Compiler Support**

Modules were standardized as part of C++20, but practical compiler and tool support has continued to evolve.

A few important points:

- **MSVC:** Provides good Module support and commonly uses the `.ixx` extension for Interfaces.
- **Clang:** Provides support for Standard C++ Modules, but the exact compiler and Build System versions must be checked.
- **GCC:** Module support has improved in newer versions, and the availability of different features can vary between releases.

It is also important to distinguish support for Modules themselves from support for Standard Library Modules.

For example, the following capability:

```
import std;
```

was standardized as part of Standard Library Modules in C++23. It should not be considered a universally available feature across all C++20 Toolchains.

Even with newer language standards, practical support depends on the compiler and Standard Library implementation.

## **Using Header Units**

In addition to named Modules, Modules introduce another concept called **Header Units**.

A Header Unit provides a mechanism for importing existing Headers using the Module Import mechanism.

For example:

```
import <iostream>;
import <vector>;

int main()
{
    std::vector<int> values{10, 20, 30};
}
```

In this example, Headers are imported using `import` instead of `#include`.

### **Difference Between Header Units and Named Modules**

| Feature                           | Header Unit                           | Named Module                 |
| --------------------------------- | ------------------------------------- | ---------------------------- |
| Source                            | Existing Header                       | Module Interface             |
| Requires a Module declaration     | No                                    | Yes                          |
| Explicitly exporting declarations | No                                    | Yes                          |
| Macros                            | Can be made available to the importer | Not normally propagated      |
| Purpose                           | Compatibility with existing Headers   | Designing modular Interfaces |

Header Units are useful for gradual migration away from traditional Headers.

However, they should not be considered completely equivalent to Named Modules.

## **Combining Modules and Traditional Headers**

In real-world projects, there is usually no need to convert all existing code to Modules at once.

Both approaches can be used together.

For example:

```
import Math;

#include <iostream>
#include <vector>

int main()
{
    std::vector<int> values{10, 20, 30};

    for (int value : values)
    {
        std::cout << add(value, 10) << '\n';
    }
}
```

In this example, the Math Module is imported, while Standard Library Headers are included using `#include`.

This approach is useful for projects migrating gradually.

## **Important Module Design Considerations**

Using Modules correctly involves more than learning the syntax of `export` and `import`.

To design an appropriate architecture, we must consider Interface organization, dependencies, and internal implementation details.

### **Design Small Interfaces**

Export only declarations that genuinely belong to the public API.

Exporting every internal declaration increases consumers' dependencies on implementation details.

### **Separate the Implementation**

For large Modules, it is preferable to place member function definitions and implementation logic in separate Implementation files.

This helps ensure that changes to the Implementation do not unnecessarily affect the public Interface.

### **Use Partitions Appropriately**

Use Partitions to divide a large Module into logically organized components.

For example:

```
Database
 ├── Database:Connection
 ├── Database:Query
 ├── Database:Transaction
 └── Database:Internal
```

This structure is suitable for internal organization because all components still belong to the same Module.

### **Avoid Dependency Cycles**

Circular dependencies between Modules can cause serious problems in the project structure and compilation process.

Dependencies should ideally be designed as a Directed Acyclic Graph, or DAG.

For example:

```
Core
 |
 +------+
 |      |
Math   Logger
 |      |
 +------+
    |
  Engine
    |
Application
```

In this structure, dependencies have a clear direction, avoiding cycles.

### **Use Facade Modules**

In large projects, you can create a primary Module that re-exports the Interfaces of several other Modules.

This simplifies the API for consumers.

### **Avoid Macro Propagation**

One advantage of Modules is that macros are not automatically propagated from a Module to its consumers.

Therefore, new designs should avoid relying on public macros when possible. Consider using `constexpr`, Templates, Functions, and other language features instead.

### **Pay Attention to Templates**

For public Templates, the definitions required for instantiation must be accessible to consumers.

Placing definitions only in Implementation Units without considering Explicit Instantiation is generally not a suitable choice.

## **Common Mistakes When Using Modules**

This section examines several common mistakes.

### **Assuming That Exporting a Module Exports All Its Declarations**

The following declaration:

```
export module Math;
```

does not mean that every declaration in the file is exported.

Only declarations that satisfy the Export rules become available through the public Interface.

### **Assuming That Modules With Similar Names Are the Same Module**

For example:

```
export module Graphics;
```

and:

```
export module Graphics.Renderer;
```

are two independent Modules.

A dot in a Module name does not establish an ownership relationship or hierarchy.

### **Using the Wrong Module Declaration**

The primary Interface uses:

```
export module Math;
```

while the Implementation uses:

```
module Math;
```

Confusing these two can result in an incorrectly defined Module structure.

### **Placing Imports After Ordinary Declarations**

Within a Module Unit, Import Declarations must appear after the Module Declaration and before other ordinary declarations.

### **Attempting to Import a Partition From Outside Its Module**

The following statement is not allowed in an external Translation Unit:

```
import Graphics:Internal;
```

Partitions can only be imported from within their own named Module.

### **Assuming Modules Completely Replace Headers**

Modules reduce many of the problems associated with Headers, but Headers may still be required for compatibility with older Libraries, macros, and certain APIs.

### **Placing Template Definitions Only in Implementations**

This can prevent consumers from instantiating a Template for the Types they need.

### **Ignoring Build Dependencies**

Compilation order matters when using Modules.

If a Module's BMI is not generated before its consumers are compiled, the Build can fail.

Therefore, Dependency management must be handled by a compatible Build System.

## **When Should You Use Modules?**

Using Modules is not necessarily the best choice for every project.

To make an appropriate decision, consider the project size, dependency structure, Toolchain, and architecture.

| Project Conditions                    | Recommendation                                       |
| ------------------------------------- | ---------------------------------------------------- |
| New and large project                 | Modules are a suitable choice                        |
| New public Library                    | Modules can provide an appropriate Interface         |
| Project with very large Headers       | Gradual migration may be beneficial                  |
| Legacy project with many dependencies | Verify Toolchain compatibility first                 |
| Small project with a few files        | Traditional Headers may be sufficient                |
| Project using an older compiler       | Headers may be the more practical choice             |
| Project with extensive Templates      | Modules can be used with appropriate Template design |

### **Recommendations for Large Projects**

For professional projects, it is advisable to begin by converting a relatively simple, independent Module from the Header/Source structure to Modules.

Then evaluate the following:

- Compilation time
- Build complexity
- Compiler compatibility
- BMI management
- Interface dependencies
- Library distribution
- Compatibility with legacy code

After evaluating the results, the migration can be extended to other parts of the project.

## **Summary**

Modules are one of the most important architectural features introduced in C++20. They provide a new model for organizing code, controlling Interfaces, and sharing declarations between Translation Units.

The main capabilities of this system include:

- **Module Interface:** Defines a Module's public API.
- **Module Implementation:** Separates definitions from the Interface.
- **Export and Import:** Controls how Module functionality is exposed and consumed.
- **Export Blocks:** Group public declarations.
- **Multiple Implementation Units:** Divide implementation across multiple files.
- **Module Interface Partitions:** Divide a large Module Interface into multiple components.
- **Internal Module Partitions:** Organize internal and non-public components.
- **Export Import:** Expose a Module's Interface through another Module.
- **Private Module Fragment:** Creates a private section within a single-file Module.
- **BMI:** Uses compiled Interface information for Module Import.

The most important principle in professional Module design is to establish a clear boundary between the public Interface and internal implementation details.

It is also essential to remember that named Modules, Namespaces, and Module Partitions are three different concepts, each serving a distinct purpose in C++ architecture.

Ultimately, Modules are not simply a replacement for `#include`. They represent a significant change in C++ compilation, dependency management, and API design.

## **References**

The following resources are recommended for further study and for examining the details of the standard:

- [cppreference - Modules (C++20)](https://en.cppreference.com/w/cpp/language/modules?utm_source=chatgpt.com) — A comprehensive reference covering syntax, Export, Import, Partitions, and language rules.
- [Clang - Standard C++ Modules](https://clang.llvm.org/docs/StandardCPlusPlusModules.html?utm_source=chatgpt.com) — An in-depth examination of Module architecture, Reachability, and its implementation in Clang.
- [Microsoft Learn - Overview of Modules in C++](https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp?utm_source=chatgpt.com) — Practical examples of Interfaces, Implementations, and Partitions.
- [CMake - C++ Modules](https://cmake.org/cmake/help/latest/manual/cmake-cxxmodules.7.html?utm_source=chatgpt.com) — A guide to dependency management, BMI, and Build System support for Modules.
- [ISO C++20 Working Draft - N4860](https://isocpp.org/files/papers/N4860.pdf?utm_source=chatgpt.com) — The C++20 Working Draft for examining the precise language rules.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>