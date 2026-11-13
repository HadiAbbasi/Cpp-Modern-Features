<div align="center">

[🇺🇸 English](./maybe_unused.md) | [🇮🇷 فارسی](../../fa/cpp17/maybe_unused.md)

</div>

----

# `[[maybe_unused]]` in C++17

Introduced in **C++17** via proposal **P0212R1** ("Wording for `[[maybe_unused]]` attribute") and refined in **C++20** via proposal **P1061R2** / **P1091R3** (extending support to structured bindings), `[[maybe_unused]]` is a standard attribute that instructs the compiler to suppress diagnostic warnings regarding unused entities.

Modern high-reliability codebases treat warnings as errors (`-Wall -Wextra -Werror` / `/W4 /WX`). However, specific architectural patterns—such as conditional compilation (`#ifdef`), debug-only assertions, generic template meta-programming, and interface conformance—frequently introduce entities that are intentionally unreferenced in particular build configurations. `[[maybe_unused]]` provides an expressive, standard-compliant mechanism to signal this intent without runtime overhead or preprocessor pollution.

```cpp
#include <cassert>

void dispatch_event([[maybe_unused]] int channel_id, const void* payload) {
    // channel_id is only referenced in Debug mode; Release builds (-DNDEBUG) strip assert(),
    // but [[maybe_unused]] prevents "unused parameter" warnings/errors under -Werror.
    assert(channel_id >= 0 && channel_id < 32);
    
    // Process payload...
}
```

---

## Table of Contents

1. [Motivation & The Problem with Legacy Solutions](#1-motivation--the-problem-with-legacy-solutions)
2. [Language Mechanics & Allowed Placement Targets](#2-language-mechanics--allowed-placement-targets)
3. [Core Systems Use Cases](#3-core-systems-use-cases)
   - [A. Debug-Only Variables & Assertions (`NDEBUG`)](#a-debug-only-variables--assertions-ndebug)
   - [B. Conditional Compilation & Target Hardware](#b-conditional-compilation--target-hardware)
   - [C. Template Metaprogramming & Tag Dispatching](#c-template-metaprogramming--tag-dispatching)
   - [D. Interface Conformance & Callback Signatures](#d-interface-conformance--callback-signatures)
4. [Structured Bindings Integration (C++17 vs. C++20)](#4-structured-bindings-integration-c17-vs-c20)
5. [Disassembly & Assembly Analysis: Zero-Cost Guarantee](#5-disassembly--assembly-analysis-zero-cost-guarantee)
6. [Feature-Test Macros & Portability](#6-feature-test-macros--portability)
7. [Pitfalls, Edge Cases, and Clean Code Guidelines](#7-pitfalls-edge-cases-and-clean-code-guidelines)
8. [Contributors](#8-contributors)

---

## 1. Motivation & The Problem with Legacy Solutions

Before C++17, suppressing unused entity warnings across different compilers required non-portable extensions or brittle preprocessor tricks:

### Historical Workarounds:
```cpp
// 1. GCC / Clang attribute
void legacy_gcc(__attribute__((unused)) int x);

// 2. MSVC __pragma
void legacy_msvc(int x) { __pragma(warning(suppress: 4100)) }

// 3. The cast-to-void macro hack
#define UNUSED(x) ((void)(x))
void legacy_cast(int x) { UNUSED(x); }

// 4. Omitting parameter names (only works for function parameters, not variables)
void legacy_omit(int /*x*/) {}
```

### Why Legacy Hacks Fall Short:
- **Portability**: Vendor-specific attributes tie codebases to specific toolchains.
- **Scope Limitations**: Omitting parameter names cannot solve unused local variables, unreferenced class members, static utility functions, or variables used only in preprocessor branches.
- **Readability**: Scattering `(void)var;` throughout function bodies adds visual clutter and pollutes the control flow.

`[[maybe_unused]]` replaces all of these idioms with a unified, standard C++ syntax applicable to functions, variables, parameters, types, and enumerators.

---

## 2. Language Mechanics & Allowed Placement Targets

The standard permits `[[maybe_unused]]` on a wide variety of declarative contexts:

```text
+-------------------------------------------------------------------------------------------------+
|                                [[maybe_unused]] Placement Matrix                                |
+-------------------------------------------------------------------------------------------------+
| Entity                | Example Declaration                                                     |
+-----------------------+-------------------------------------------------------------------------+
| Local Variable        | [[maybe_unused]] int result = perform_audit();                          |
| Function Parameter    | void handler([[maybe_unused]] int event_flags);                         |
| Function Declaration  | [[maybe_unused]] static void internal_probe();                          |
| Class / Struct        | struct [[maybe_unused]] LegacyHeader {};                                |
| Non-Static Data Member| struct Packet { [[maybe_unused]] uint32_t reserved; };                  |
| Static Data Member    | [[maybe_unused]] static constexpr uint32_t kMagic = 0xDEADBEEF;         |
| Enumeration & Enum Val| enum class [[maybe_unused]] State { Active, [[maybe_unused]] Stale };   |
| Typedef / Using Alias | [[maybe_unused]] using Handle = void*;                                  |
| Structured Bindings   | auto [status, [[maybe_unused]] bytes] = socket.recv(); // (C++20)       |
+-------------------------------------------------------------------------------------------------+
```

### Important Syntactic Nuance:
Placing `[[maybe_unused]]` on an entity does **not** force it to be unused. If the entity is actually accessed, the attribute is simply ignored by the compiler without warnings.

---

## 3. Core Systems Use Cases

### A. Debug-Only Variables & Assertions (`NDEBUG`)
In release builds (`-O3 -DNDEBUG`), the standard macro `assert(expr)` expands to `((void)0)`. Any variable evaluated exclusively inside an assertion will trigger an `unused-variable` compiler error under `-Werror`:

```cpp
#include <cassert>

bool verify_checksum(const void* data, size_t len);

void process_packet(const void* payload, size_t size) {
    [[maybe_unused]] bool is_valid = verify_checksum(payload, size);
    
    // In Release: assert disappears -> 'is_valid' is never read!
    // Without [[maybe_unused]], CI builds fail on unused-variable.
    assert(is_valid); 
}
```

---

### B. Conditional Compilation & Target Hardware
Embedded, systems, and cross-platform drivers maintain platform-dependent execution branches. Declaring a variable that is only read on specific architectures causes warnings on others:

```cpp
void flush_pipeline(void* mmio_register) {
    [[maybe_unused]] auto* reg_ptr = static_cast<volatile uint32_t*>(mmio_register);

#if defined(PLATFORM_X86_64)
    _mm_sfence();
#elif defined(PLATFORM_ARM64)
    asm volatile("dmb ish" ::: "memory");
#elif defined(PLATFORM_EMBEDDED_SOC)
    *reg_ptr = 0x1; // Only used on bare-metal SoC builds
#endif
}
```

---

### C. Template Metaprogramming & Tag Dispatching
In generic template code, tag parameters are often passed by value purely for type resolution via argument-dependent lookup (ADL) or function overloading:

```cpp
template <typename T>
struct SerializerTag {};

template <typename T>
void serialize_impl(const T& value, [[maybe_unused]] SerializerTag<T> tag) {
    // 'tag' exists solely for overload resolution; it has no runtime data.
    serialize_raw(value);
}
```

---

### D. Interface Conformance & Callback Signatures
When implementing abstract interfaces, function pointer hooks, or signal handlers, the function signature is dictated by the API framework. If an implementation does not need a particular parameter, `[[maybe_unused]]` avoids compiler noise while keeping documentation intact:

```cpp
struct IOHandler {
    static void on_read_complete([[maybe_unused]] int fd, 
                                 [[maybe_unused]] size_t bytes_transferred,
                                 const char* buffer) {
        // This handler forwards buffer directly to an in-memory queue,
        // ignoring descriptor ID and length (null-terminated stream).
        log_stream(buffer);
    }
};
```

---

## 4. Structured Bindings Integration (C++17 vs. C++20)

In C++17, structured bindings could only be annotated at the whole-declaration level, not individual decomposed elements:

```cpp
// C++17: Attributes apply to the entire structured binding declaration
[[maybe_unused]] auto [fd, status, bytes_read] = read_network_stream();
```
*Limitation in C++17:* If you only wanted to suppress warnings for one specific variable while making sure you didn't accidentally forget the others, you could not place `[[maybe_unused]]` directly onto `[fd]`.

### C++20 Enhancement (P1061R2 / P1091R3):
C++20 permits applying attributes to individual elements inside the identifier pack:

```cpp
// C++20: Per-element attribute assignment
auto [fd, [[maybe_unused]] status, bytes_read] = read_network_stream();
// 'fd' and 'bytes_read' MUST be used; 'status' may remain unused without warnings!
```

---

## 5. Disassembly & Assembly Analysis: Zero-Cost Guarantee

Attributes in C++ are purely compile-time syntactic directives. They generate **no instructions, no stack allocations, and no metadata** in the resulting binary.

Let us compare the code generation under GCC 13 (`-O2`):

```cpp
int calculate_with_unused(int x, [[maybe_unused]] int y) {
    return x * 2;
}

int calculate_standard(int x, int /*y*/) {
    return x * 2;
}
```

### Disassembly (x86-64):
```assembly
calculate_with_unused(int, int):
    lea     eax, [rdi+rdi]
    ret

calculate_standard(int, int):
    lea     eax, [rdi+rdi]
    ret
```

Both functions emit identical machine code. The attribute is consumed and discarded by the compiler front-end parser and warning analyzer.

---

## 6. Feature-Test Macros & Portability

The presence of `[[maybe_unused]]` across toolchains can be queried using `__has_cpp_attribute`:

```cpp
#include <iostream>

#if defined(__has_cpp_attribute)
    #if __has_cpp_attribute(maybe_unused) >= 201603L
        #define SAFE_MAYBE_UNUSED [[maybe_unused]]
    #else
        #define SAFE_MAYBE_UNUSED
    #endif
#else
    #define SAFE_MAYBE_UNUSED
#endif

SAFE_MAYBE_UNUSED static void internal_telemetry() {
    // Suppresses warning if unreferenced in certain translation units
}
```

---

## 7. Pitfalls, Edge Cases, and Clean Code Guidelines

### 1. Masking True Bugs and Dead Code
Do not indiscriminately spray `[[maybe_unused]]` across codebases to silence warnings quickly. Unused variables are often symptoms of uncompleted logic, typos (e.g., modifying `val` instead of `val2`), or forgotten return values:

```cpp
// BAD: Silencing a genuine bug
[[maybe_unused]] auto res = write_to_database(); 
// Res should probably be checked for network failure!
```

### 2. Static Functions in Translation Units
If a `static` non-inline function is unreferenced in a translation unit, marking it `[[maybe_unused]]` prevents `-Wunused-function` warnings. However, if the function is completely obsolete, prefer deleting it or using anonymous namespaces cleanly rather than keeping dead code around indefinitely.

### 3. Side Effects in Unused Declarations
Remember that declaring a variable `[[maybe_unused]]` still constructs and destructs the object:

```cpp
struct RAIIProbe {
    RAIIProbe() { log("Resource acquired"); }
    ~RAIIProbe() { log("Resource released"); }
};

void run() {
    [[maybe_unused]] RAIIProbe probe; // The constructor and destructor STILL execute!
}
```
If you intend to completely remove runtime costs in specific builds, rely on `#ifdef` or `if constexpr` rather than solely `[[maybe_unused]]`.

---

## 8. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |