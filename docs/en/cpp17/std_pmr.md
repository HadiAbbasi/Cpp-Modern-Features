<div align="center">

[🇺🇸 English](./pmr.md) | [🇮🇷 فارسی](../../fa/cpp17/pmr.md)

</div>

----

# `std::pmr` (Polymorphic Memory Resources) in C++17

The `<memory_resource>` header, introduced in C++17, fundamentally alters how C++ containers manage dynamic memory. Traditionally, C++ containers (like `std::vector`, `std::list`, `std::map`) have a hard-coded allocator type, which is part of their template signature (e.g., `std::vector<T, std::allocator<T>>`).

`std::pmr` introduces **Polymorphic Memory Resources**, allowing a container to change its allocation strategy at *runtime* without changing its type.

```cpp
#include <vector>
#include <memory_resource>

// std::pmr::vector is an alias for std::vector with a polymorphic allocator
std::pmr::vector<int> vec{1, 2, 3}; 

// You can swap memory resources at runtime
char buffer[1024];
std::pmr::monotonic_buffer_resource pool{buffer, sizeof(buffer)};
vec.get_allocator() = &pool; // Uses the stack buffer for allocations
```

---

## Table of Contents

1. [The Problem with Standard Allocators](#1-the-problem-with-standard-allocators)
2. [How `std::pmr` Works](#2-how-pmr-works)
3. [Memory Resource Types](#3-memory-resource-types)
4. [Container Aliases (`std::pmr::...`)](#4-container-aliases)
5. [Performance Considerations](#5-performance-considerations)
6. [Best Practices](#6-best-practices)
7. [Quick Reference](#7-quick-reference)
8. [Pitfalls and Gotchas](#8-pitfalls-and-gotchas)
9. [Contributors](#9-contributors)

---

## 1. The Problem with Standard Allocators

Standard allocators (`std::allocator`) are template parameters. This leads to two primary issues:
1. **Type Bloat:** `std::vector<int, A>` and `std::vector<int, B>` are distinct, incompatible types. They cannot be stored in the same container or passed to the same functions without templates.
2. **Compile-Time Binding:** You must know your allocation strategy at compile time. If you want to switch from a heap-based allocator to a stack-based arena allocator, you are forced to recompile or rewrite your types.

---

## 2. How `std::pmr` Works

`std::pmr` decouples the *interface* of the allocator from the *implementation* using type erasure.

*   **`std::pmr::memory_resource`**: An abstract base class defining the interface (`do_allocate`, `do_deallocate`, `do_is_equal`).
*   **`std::pmr::polymorphic_allocator<T>`**: A concrete allocator that holds a pointer to a `memory_resource`. This is the allocator used by all `pmr` containers.

Because the `polymorphic_allocator` simply wraps a pointer to the base class, all `pmr` containers of the same type (e.g., `std::pmr::vector<int>`) are compatible, regardless of the *actual* memory strategy used at runtime.

---

## 3. Memory Resource Types

C++17 provides several standard implementations of `memory_resource`:

*   **`std::pmr::new_delete_resource()`**: The default. Uses global `new` and `delete`.
*   **`std::pmr::null_memory_resource()`**: Always throws `std::bad_alloc`. Useful for testing or disabling allocations.
*   **`std::pmr::monotonic_buffer_resource`**: Extremely fast. It allocates from a pre-supplied buffer. It does *not* deallocate individual blocks; instead, it deallocates everything at once when the resource object is destroyed.
*   **`std::pmr::synchronized_pool_resource` / `unsynchronized_pool_resource`**: Manages a pool of fixed-size chunks, good for reducing fragmentation when performing many small allocations.

---

## 4. Container Aliases (`std::pmr::...`)

The standard library provides aliases for common containers to simplify usage:

```cpp
namespace std::pmr {
    template <typename T> using vector = std::vector<T, polymorphic_allocator<T>>;
    template <typename T> using list = std::list<T, polymorphic_allocator<T>>;
    template <typename K, typename V> using map = std::map<K, V, std::less<K>, polymorphic_allocator<std::pair<const K, V>>>;
    // ... and so on for string, deque, set, unordered_map, etc.
}
```

---

## 5. Performance Considerations

*   **Indirection:** `pmr` containers introduce a minor overhead due to the virtual function call on every allocation/deallocation.
*   **Cache Locality:** Using `monotonic_buffer_resource` can provide significant performance gains by keeping allocated objects contiguous in memory and avoiding expensive system calls to the OS heap manager.
*   **Pool Resources:** Pool allocators are superior when the application creates and destroys many objects of similar size, significantly reducing heap fragmentation.

---

## 6. Best Practices

1. **Prefer `pmr` for Arenas:** If you have an "arena" of memory (e.g., a buffer that lives for the duration of a request), use `std::pmr::monotonic_buffer_resource`.
2. **Don't over-abstract:** If you do not need runtime-polymorphic memory, stick to `std::allocator` or custom compile-time allocators to avoid the virtual function overhead.
3. **Use the aliases:** Always use the `std::pmr::...` aliases instead of manually typing the long template specializations.

---

## 7. Quick Reference

| Feature | Description |
|---|---|
| **Header** | `<memory_resource>` |
| **Base Class** | `std::pmr::memory_resource` |
| **Allocator** | `std::pmr::polymorphic_allocator<T>` |
| **Performance** | O(1) virtual call overhead per allocation |
| **Compatibility** | All `std::pmr` containers of type `T` are compatible |

---

## 8. Pitfalls and Gotchas

1. **Resource Lifetime:** The `memory_resource` object must outlive all containers using it. If a `vector` tries to deallocate memory using a destroyed resource, the program will crash (undefined behavior).
2. **Virtual Overhead:** Do not use `pmr` containers in performance-critical inner loops where a single `malloc`/`free` might have been faster than a virtual function call.
3. **Implicit Default:** If you don't provide a resource, `std::pmr` containers default to `std::pmr::new_delete_resource()`. This is usually safe but be aware it uses the global heap.

---

## 9. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |