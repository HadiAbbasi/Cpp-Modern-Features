<div align="center">

[🇺🇸 English](./init_statements.md) | [🇮🇷 فارسی](../../fa/cpp17/init_statements.md)

</div>

----

# Selection Statements with Initializer in C++17: Scoped Conditionals in `if` and `switch`

C++17 introduced **selection statements with initializer** (often referred to as *init-statements in `if` and `switch`*). This language feature extends the familiar syntax from traditional `for` loops—where a loop-scoped variable is declared inside the statement head—to `if` and `switch` constructs (`if (init; condition)` and `switch (init; condition)`). It solves variable scope leakage, simplifies RAII guard management, and improves code readability by binding temporary status results strictly to the conditional blocks that inspect them.

```cpp
#include <iostream>
#include <map>
#include <mutex>

std::map<int, std::string> g_registry;
std::mutex g_mutex;

int main() {
    // Before C++17: Variable leaks into the enclosing scope
    // auto it = g_registry.find(42);
    // if (it != g_registry.end()) { /* use it */ }
    // // 'it' is still alive here, polluting scope and risking reuse!

    // C++17: Scoped directly to the condition and branches
    if (auto it = g_registry.find(42); it != g_registry.end()) {
        std::cout << "Found: " << it->second << '\n';
    } else {
        std::cout << "Key not present; iterator still valid here!\n";
    } // 'it' is destroyed here!

    // RAII Lock pattern with scoped lifetime:
    if (std::lock_guard lock(g_mutex); !g_registry.empty()) {
        g_registry.clear();
    } // 'lock' releases mutex immediately upon exiting the if statement!
}
```

This guide covers why selection initializers exist, formal grammar and syntax, scope and lifetime rules, lifetime extension with `else`, combining init-statements with structured bindings, idiomatic patterns with `switch`, common pitfalls, and real-world best practices.

---

## Table of Contents

1. [Why Selection Statement Initializers Exist](#1-why-selection-statement-initializers-exist)
2. [What Was the Problem Before C++17?](#2-what-was-the-problem-before-c17)
3. [Grammar and Syntax Overview](#3-grammar-and-syntax-overview)
4. [How Scope and Lifetime Work](#4-how-scope-and-lifetime-work)
5. [The `else` Branch Behavior](#5-the-else-branch-behavior)
6. [Combining with Structured Bindings](#6-combining-with-structured-bindings)
7. [Scoped RAII Locks and Synchronization](#7-scoped-raii-locks-and-synchronization)
8. [Init-Statements in `switch`](#8-init-statements-in-switch)
9. [Interaction with `constexpr if` (`if constexpr`)](#9-interaction-with-constexpr-if-if-constexpr)
10. [Comparison with Legacy Alternatives](#10-comparison-with-legacy-alternatives)
11. [Common Pitfalls and Gotchas](#11-common-pitfalls-and-gotchas)
12. [Performance and Assembly Analysis](#12-performance-and-assembly-analysis)
13. [Best Practices](#13-best-practices)
14. [Evolution: C++20 and Beyond](#14-evolution-c20-and-beyond)
15. [Feature-Test Macro](#15-feature-test-macro)
16. [Quick Reference Card](#16-quick-reference-card)
17. [Complete Example](#17-complete-example)
18. [Exercises](#18-exercises)
19. [Conclusion](#19-conclusion)
20. [Contributors](#19-contributors)

---

## 1. Why Selection Statement Initializers Exist

In clean systems programming, **minimizing variable scope** is one of the most effective strategies to prevent logic bugs, accidental use-after-free, and unintended state mutation.

Before C++17, C++ already allowed variable declarations inside conditional expressions, but **only if the variable itself could be implicitly converted to `bool`**:

```cpp
if (Node* n = getNode()) { // OK: pointer converts to bool
    n->process();
}
```

However, real-world code often deals with:
- Functions returning status codes, error enums, or `std::pair` results.
- Iterators where testing validity requires an explicit check (`it != container.end()`).
- RAII guards (like `std::lock_guard` or transactions) that must stay alive across the condition and both branches without polluting the surrounding function body.

C++17 bridged the syntax gap between `for (init; cond; step)` and `if (init; cond)` / `switch (init; cond)`.

---

## 2. What Was the Problem Before C++17?

### Problem 1: Variable Pollution and Shadowing Bugs

```cpp
void handleClient() {
    auto status = readSocket();
    if (status == Status::Ok) {
        // ...
    }
    
    // ... 40 lines of code ...
    
    // Developer accidentally checks the old 'status' instead of calling readSocket again!
    if (status == Status::Timeout) { 
        // Bug! Reusing stale variable
    }
}
```

### Problem 2: The Ugly Artificial Block Workaround

To limit variable lifetime before C++17, developers were forced to wrap `if` statements inside standalone block scopes (`{ ... }`):

```cpp
{
    auto it = cache.find(key);
    if (it != cache.end()) {
        use(it->second);
    }
} // 'it' goes out of scope here
```

This added redundant indentation levels and cluttered business logic.

---

## 3. Grammar and Syntax Overview

The C++17 standard modifies the grammar of selection statements as follows:

```text
if ( init-statement condition ) statement
if ( init-statement condition ) statement else statement

switch ( init-statement condition ) statement
```

Where `init-statement` can be:
- An **expression statement** (e.g., `doSomething();`)
- A **simple declaration** (e.g., `auto x = compute();` or `auto [k, v] = entry;`)

### Syntax Forms

```cpp
// 1. Simple variable declaration
if (int val = query(); val > 0) { ... }

// 2. Structured binding
if (auto [status, msg] = sendPacket(); status != 200) { ... }

// 3. RAII Guard without explicit value check in condition
if (std::lock_guard lock(mtx); is_ready) { ... }

// 4. Switch statement with scoped calculation
switch (DeviceState state = queryState(); state.code()) {
    case Ready:   /* ... */ break;
    case Busy:    /* ... */ break;
    case Offline: /* ... */ break;
}
```

---

## 4. How Scope and Lifetime Work

The variable declared in the `init-statement`:
1. Is constructed **before** the condition is evaluated.
2. Is in scope inside the `condition`.
3. Is in scope inside the `true` branch statement.
4. Is in scope inside the `else` branch statement.
5. Is **destroyed** at the exact end of the entire selection statement (after the `true` or `else` branch completes).

 
---

## 5. The `else` Branch Behavior

A common source of confusion for newcomers is whether the initialized variable is available inside the `else` block.

> **Rule:** The variable initialized in an `if` statement remains in scope and fully valid throughout the entire `else` branch.

This is extremely beneficial when handling fallback or failure diagnostics:

```cpp
#include <system_error>

if (std::error_code ec = performIO(); !ec) {
    std::cout << "Operation succeeded.\n";
} else {
    // 'ec' is completely accessible here for logging!
    std::cerr << "Operation failed: " << ec.message() << " (Code: " << ec.value() << ")\n";
}
```

---

## 6. Combining with Structured Bindings

The synergy between **init-statements** and **structured bindings (C++17)** is one of the most powerful and idiomatic paradigms in modern C++.

### Idiomatic Map Insertion

`std::map::insert` returns a `std::pair<iterator, bool>`, where the `bool` indicates whether the element was newly inserted or already existed:

```cpp
#include <map>
#include <string>
#include <iostream>

std::map<std::string, int> user_scores;

void registerUser(const std::string& name, int score) {
    if (auto [it, inserted] = user_scores.insert({name, score}); inserted) {
        std::cout << "Successfully inserted " << it->first << " with score " << it->second << '\n';
    } else {
        std::cout << "User " << it->first << " already exists with score " << it->second << '\n';
    }
    // Neither 'it' nor 'inserted' pollute the enclosing function!
}
```

---

## 7. Scoped RAII Locks and Synchronization

In concurrent programming, minimizing lock contention is critical. Locks should be held for the smallest possible scope.

```cpp
#include <mutex>
#include <vector>

std::mutex queue_mutex;
std::vector<int> task_queue;

void processNextTask() {
    // Lock acquired -> checked -> manipulated -> released at end of if-block!
    if (std::lock_guard lock(queue_mutex); !task_queue.empty()) {
        int task = task_queue.back();
        task_queue.pop_back();
        // Do NOT call expensive heavyTaskProcessing(task) inside the lock!
    } // lock automatically unlocked here!

    // Rest of function executes without holding the mutex
}
```

Notice how `std::lock_guard lock(queue_mutex)` uses **Class Template Argument Deduction (CTAD)** alongside the **init-statement**, drastically cutting boilerplate.

---

## 8. Init-Statements in `switch`

Before C++17, declaring variables inside a `switch` head was limited to types that directly evaluated to integer or enumeration values. If you needed to extract a code from an object, the object had to live outside the switch.

### C++17 `switch` with Initializer

```cpp
#include <iostream>

struct HttpResponse {
    int status_code;
    std::string body;
};

HttpResponse fetchEndpoint();

void handleResponse() {
    switch (HttpResponse res = fetchEndpoint(); res.status_code) {
        case 200:
            std::cout << "Success: " << res.body << '\n';
            break;
        case 404:
            std::cerr << "Not Found\n";
            break;
        case 500:
            std::cerr << "Internal Server Error\n";
            break;
        default:
            std::cerr << "Unhandled code: " << res.status_code << '\n';
            break;
    } // 'res' is destroyed here!
}
```

---

## 9. Interaction with `constexpr if` (`if constexpr`)

Init-statements can also be combined with compile-time conditionals:

```cpp
template <typename T>
auto getMemoryFootprint(const T& obj) {
    if constexpr (auto size = sizeof(T); size > 8) {
        return size * 2; // Arbitrary logic for large objects
    } else {
        return size;
    }
}
```

The initializer statement is evaluated during compilation; if any variables are constructed within non-discarded branches, standard lifetime and constant evaluation rules apply.

---

## 10. Comparison with Legacy Alternatives

| Technique | Scope Cleanup | Visual Overhead | Allows Complex Conditions? | Supports Multiple Branches? |
|---|---|---|---|---|
| **C++17 `if (init; cond)`** | ✅ Automatic at statement end | Minimal & Idiomatic | ✅ Yes (`val != end()`) | ✅ Yes (`if` & `else`) |
| **C++98 Condition Decl (`if (T* p = ...)`)** | ✅ Automatic | Minimal | ❌ No (only bool/pointer test) | ✅ Yes |
| **Artificial Enclosing Block (`{ ... }`)** | ✅ Manual via braces | ⚠️ Extra indentation | ✅ Yes | ✅ Yes |
| **Function-level declaration** | ❌ Leaks into whole function | Boilerplate pollution | ✅ Yes | ❌ Risk of stale state |

---

## 11. Common Pitfalls and Gotchas

### 11.1 The Empty Init-Statement Semicolon Mistake

Writing a semicolon without declaring an initializer is valid syntax, but can look confusing or mask intent:

```cpp
if (; condition) { // Legal, but redundant. Identical to standard `if (condition)`
    // ...
}
```

### 11.2 Accidental Variable Shadowing

If a variable in the enclosing scope shares the name with the init-statement variable, the inner variable completely shadows the outer one:

```cpp
int id = 999;

void check() {
    if (int id = fetchId(); id > 0) {
        std::cout << id; // Prints fetched id, NOT 999
    }
    std::cout << id; // Prints 999
}
```
*Best Practice: Enable compiler warnings like `-Wshadow` to detect accidental name overlaps.*

### 11.3 Lifetime of Temporaries Bound to References

If you bind a temporary to a reference inside the init-statement, the temporary's lifetime is extended until the end of the entire `if` statement (including `else`):

```cpp
struct Packet {
    int code;
    ~Packet() { std::cout << "Destroyed\n"; }
};

Packet getPacket();

void test() {
    if (const Packet& pkt = getPacket(); pkt.code == 0) {
        std::cout << "Handling zero\n";
    } // Packet destructor runs here, NOT before the if body!
}
```

---

## 12. Performance and Assembly Analysis

There is **zero runtime overhead** associated with using selection initializers.

- The compiler generates identical machine code to an artificial block scope (`{ auto x = ...; if (x) ... }`).
- Destructors for stack objects are emitted at the closing brace of the selection construct.
- By scoping variables more tightly, you can help the compiler's register allocator reuse stack slots and registers earlier, frequently improving optimization pipelines and cache locality.

---

## 13. Best Practices

1. **Adopt as default for lookups:** Use `if (auto it = map.find(...); it != map.end())` whenever interacting with associative containers or searching algorithms.
2. **Pair with Structured Bindings:** Write `if (auto [ok, val] = fn(); ok)` to handle dual-output or status-payload idioms cleanly.
3. **Use for localized RAII locking:** Minimize mutex hold times by letting the `if` construct automatically trigger lock destruction.
4. **Do not overcomplicate conditions:** If an initialization requires multiple separate statements or complex helper logic, consider extracting it into a well-named helper function instead of packing too much inside the `if (...)` header.

---

## 14. Evolution: C++20 and Beyond

- **C++20 Range-based `for` with Initializer:**  
  C++20 extended the init-statement syntax to range-based for loops:
  ```cpp
  // C++20: Range-for with initializer
  for (auto items = getContainer(); auto& item : items) {
      // Prevents dangling reference bugs when iterating over temporaries!
  }
  ```

---

## 15. Feature-Test Macro

Unlike many other C++17 features, the ISO C++ standard does not define a dedicated feature-test macro (such as `__cpp_...`) for selection statements with initializers.

Availability is checked via the standard version macro:

```cpp
#if defined(__cplusplus) && __cplusplus >= 201703L
    // C++17 selection statements with initializer are supported
#endif
```

Direct compiler availability begins in:
- GCC 7.1+
- Clang 3.9+
- MSVC 2017 (19.10+) with `/std:c++17`

---

## 16. Quick Reference Card

| Pattern | C++17 Syntax Example |
|---|---|
| **Lookup in Map** | `if (auto it = m.find(key); it != m.end()) { /* use it->second */ }` |
| **Map Insert** | `if (auto [it, ok] = m.insert({k, v}); ok) { /* newly inserted */ }` |
| **Error Handling** | `if (std::error_code ec = write(); !ec) { /* ok */ } else { /* log ec */ }` |
| **Thread Locking** | `if (std::unique_lock lk(mtx); ready) { /* protected work */ }` |
| **Switch Extraction** | `switch (auto val = parse(); val.type()) { case Type::A: ... }` |

---

## 17. Complete Example

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
#include <optional>
#include <mutex>

enum class ErrorCode { None, NotFound, PermissionDenied };

struct QueryResult {
    ErrorCode error;
    std::string payload;
};

QueryResult mockDatabaseQuery(int id) {
    if (id == 42) {
        return {ErrorCode::None, "Secret Payload"};
    }
    return {ErrorCode::NotFound, ""};
}

int main() {
    std::cout << "=== 1. Scoped Map Insertion ===\n";
    std::unordered_map<std::string, int> accounts;

    if (auto [it, success] = accounts.emplace("admin", 1001); success) {
        std::cout << "Account created: " << it->first << " -> ID " << it->second << '\n';
    }

    // Attempt duplicate insertion
    if (auto [it, success] = accounts.emplace("admin", 9999); !success) {
        std::cout << "Conflict detected! Account " << it->first << " already exists.\n";
    }

    std::cout << "\n=== 2. Switch Statement with Initializer ===\n";
    switch (QueryResult res = mockDatabaseQuery(42); res.error) {
        case ErrorCode::None:
            std::cout << "Query Success: " << res.payload << '\n';
            break;
        case ErrorCode::NotFound:
            std::cerr << "Query Failed: Record not found!\n";
            break;
        case ErrorCode::PermissionDenied:
            std::cerr << "Query Failed: Access Denied!\n";
            break;
    } // 'res' is cleaned up immediately here

    std::cout << "\n=== 3. Scope Protection in Else Branch ===\n";
    if (QueryResult res = mockDatabaseQuery(10); res.error == ErrorCode::None) {
        std::cout << "Data: " << res.payload << '\n';
    } else {
        std::cout << "Else branch has safe access to res. Error code: " 
                  << static_cast<int>(res.error) << '\n';
    }

    return 0;
}
```

### Expected Output

```text
=== 1. Scoped Map Insertion ===
Account created: admin -> ID 1001
Conflict detected! Account admin already exists.

=== 2. Switch Statement with Initializer ===
Query Success: Secret Payload

=== 3. Scope Protection in Else Branch ===
Else branch has safe access to res. Error code: 1
```

---

## 18. Exercises

### Exercise 1 — Container Lookup Refactoring
Take a codebase using the old pattern:
```cpp
auto it = cache.find(session_id);
if (it != cache.end()) {
    return it->second;
}
return std::nullopt;
```
Refactor it into a concise, single C++17 `if` with initializer statement.

### Exercise 2 — Mutex Lifetime Verification
Write a test program with a custom `LoggingLock` class that prints `"Acquired"` in its constructor and `"Released"` in its destructor. Place `LoggingLock` inside an `if (LoggingLock lock; is_valid)` construct and observe the exact console output order relative to code inside and outside the `if` block.

### Exercise 3 — Structured Binding with Fallback Diagnostics
Write a function returning `std::pair<bool, std::string>`. Use `if (auto [ok, err] = validateInput(); ok)` and verify that `err` can be inspected inside the `else` block to output a descriptive error message.

### Exercise 4 — Refactoring a Complex `switch`
Write a function that parses a raw network packet header, extracts an enum opcode via a member function, and processes it inside a `switch (PacketHeader h = parse(buf); h.opcode())`. Ensure no packet object leaks outside the `switch`.

---

## 19. Conclusion

Selection statements with initializer in C++17 provide a clean, disciplined approach to control flow:
- They **eliminate scope pollution**, confining variables to where they are actually needed.
- They **simplify RAII and thread synchronization**, bounding lock durations directly to the conditional scope.
- They combine seamlessly with **Structured Bindings** and **CTAD** for modern, declarative C++ coding styles.
- They incur **zero runtime cost** while improving code safety and compiler optimization opportunities.

---

## 20. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |