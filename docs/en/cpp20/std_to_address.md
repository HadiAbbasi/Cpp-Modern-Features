<div align="center">

[🇺🇸 English](./std_to_address.md) | [🇮🇷 فارسی](../../fa/cpp20/std_to_address.md)

</div>

---

# A Comprehensive Guide to `std::to_address` in C++20

## Introduction

In C++20, the `std::to_address` function was introduced in the `<memory>` header as a standard utility for obtaining the raw address of a memory location or object from a pointer-like type. Its significance is not limited to converting a pointer type to `T*`; its primary purpose is to provide a standard, extensible way to work with custom pointer types, allocators, and contiguous iterators.

In C++, not every type that behaves like a pointer is necessarily of type `T*`. For example, an allocator may return its own pointer type. Such a type, commonly called a *fancy pointer*, may store information such as an offset relative to shared memory, a handle, or metadata. In these situations, generic code must not assume that it can use `operator*` or `operator->` directly to obtain a raw address.

`std::to_address` hides this difference and, when the pointer-like type is implemented correctly, provides the corresponding raw address without requiring a reference to be formed through dereferencing.

## The Problem This Function Solves

### Obtaining an Address Versus Dereferencing

A common way to obtain the address of an existing object is to use `std::addressof(*p)`. This first dereferences the pointer and then takes the address of the resulting object. However, this pattern is not valid in every memory state.

For example, an allocator may have reserved storage in which no object has been constructed yet. In this situation, the storage has an address, but dereferencing the pointer to form a reference to a live, valid object is not permitted. `std::to_address` is designed for such situations as well: it can obtain the storage address without forming a reference through `*p`.

```cpp
#include <memory>

int main() {
    std::allocator<int> alloc;

    int* p = alloc.allocate(4);

    // The storage is allocated, but no int object has been constructed yet.
    int* raw = std::to_address(p);

    alloc.deallocate(p, 4);
}
```

In the example above, `std::allocator<int>::allocate` provides the required memory, but it does not, by itself, initialize and create four `int` objects. Calling `std::to_address` to obtain the raw address does not require dereferencing the pointer.

### Support for Fancy Pointers

When designing allocators, the `pointer` type is not necessarily `T*`. A custom pointer type can have a different internal representation and may provide its own customization mechanism as the only valid way to access the raw address.

Without a standard customization point, generic libraries would have to implement separate logic for every custom pointer type or depend on its implementation details. `std::to_address` addresses this need through a mechanism based on `std::pointer_traits` and `operator->`.

## Introducing the Standard Interface

### Header and Function Signatures

The function is defined in the `<memory>` header and has been available since C++20.

```cpp
#include <memory>

template<class T>
constexpr T* to_address(T* p) noexcept;

template<class Ptr>
constexpr auto to_address(const Ptr& p) noexcept;
```

There are two primary overloads:

- The first overload is for raw pointers such as `int*`, `const Widget*`, and `void*`.
- The second overload is designed for pointer-like types and types such as fancy pointers that can provide a raw address through customization or `operator->`.

The return type of the overload for pointer-like types is deduced from the expression used to obtain the address. In typical use, the result is a raw pointer to the element type.

### Behavior of the Raw-Pointer Overload

For an argument of type `T*`, the function returns the input pointer unchanged. Therefore, this call performs no additional conversion or dereferencing.

```cpp
#include <memory>
#include <type_traits>

int main() {
    int value = 42;
    int* p = &value;

    int* raw = std::to_address(p);

    static_assert(std::is_same_v<decltype(raw), int*>);
    return raw == &value ? 0 : 1;
}
```

The type `T` must not be a function type. Consequently, using this function with function pointers is ill-formed; the function is specified for addresses of objects and their storage, not for converting function pointers.

### Behavior of the Pointer-Like-Type Overload

For the second overload, the standard specifies two paths:

- If the following expression is valid, that path is used:

  `std::pointer_traits<Ptr>::to_address(p)`

- Otherwise, the function recursively uses the following path:

  `std::to_address(p.operator->())`

Therefore, providing a member named `operator->` is not the only way to support the function. A suitable specialization of `std::pointer_traits` can customize address extraction.

## The `std::pointer_traits` Mechanism

### The Role of Pointer Traits

The `std::pointer_traits` class template has existed since C++11 and is designed to provide standard information about pointer-like types. It can provide information such as `element_type`, `difference_type`, and the rebound pointer template represented by `rebind`.

In C++20, the optional `to_address` member was also made available for programmer-defined specializations. `std::to_address` checks for this member as a customization.

### Implementing a Custom Specialization

Suppose a custom pointer type stores the raw address in an internal member and does not want to, or cannot, provide a conventional `operator->`. A specialization of `std::pointer_traits` can be defined.

```cpp
#include <cstddef>
#include <memory>

template<class T>
class offset_like_pointer {
public:
    using element_type = T;

    explicit offset_like_pointer(T* ptr) noexcept : ptr_(ptr) {}

private:
    T* ptr_;

    template<class>
    friend struct std::pointer_traits;
};

namespace std {
    template<class T>
    struct pointer_traits<offset_like_pointer<T>> {
        using pointer = offset_like_pointer<T>;
        using element_type = T;
        using difference_type = ptrdiff_t;

        template<class U>
        using rebind = offset_like_pointer<U>;

        static element_type* to_address(pointer p) noexcept {
            return p.ptr_;
        }
    };
}

int main() {
    int value = 10;
    offset_like_pointer<int> p{&value};

    int* raw = std::to_address(p);
    return *raw == 10 ? 0 : 1;
}
```

This example is only a simple representation intended to explain customization. A real offset-based pointer usually also needs to manage conversion rules, memory lifetime, and constraints related to the relevant segment or mapping. In addition, programmer-defined specializations of standard templates must be defined in accordance with the permitted standard rules. Adding a specialization to `std` for a program-defined type is allowed within this framework; changing the behavior of standard types is not.

### Choosing Between `pointer_traits::to_address` and `operator->`

Having both paths provides greater design flexibility:

- If the pointer type requires a special conversion mechanism, `pointer_traits::to_address` is the more suitable choice.
- If `operator->` correctly provides a raw address or another pointer-like type, the fallback path is sufficient.
- If both paths are available and the expression `pointer_traits<Ptr>::to_address(p)` is valid, the `pointer_traits` path takes precedence.

A custom implementation should be `noexcept` and return a raw pointer that genuinely refers to the location represented by the pointer-like object. Returning an invalid address or an address with different semantics violates the function's expected contract.

## Relationship to Iterators

### Support for Contiguous Iterators

One important use of `std::to_address` in C++20 is working with iterators that satisfy the `std::contiguous_iterator` concept. This concept is more restrictive than random-access iteration and guarantees that the elements of a range are stored contiguously in memory.

Common examples of contiguous iterators include raw pointers and `std::vector` iterators under normal conditions. By contrast, `std::deque` iterators are not contiguous because their elements may be stored in multiple separate blocks.

```cpp
#include <iterator>
#include <memory>
#include <type_traits>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto it = values.begin();

    static_assert(std::contiguous_iterator<decltype(it)>);

    int* raw = std::to_address(it);

    return raw[1] == 20 ? 0 : 1;
}
```

The `std::contiguous_iterator` concept specifies requirements beyond random-access iteration, including requirements on the reference type and the relationship between the addresses of consecutive elements. It guarantees that the address of the next element is consistent with arithmetic on the corresponding pointer. This guarantee is what makes converting the iterator to a pointer valid.

### Why Not Every Iterator Can Be Converted to a Raw Address

Random-access capability does not imply contiguity. An iterator may support `it + n`, iterator subtraction, and indexed access while its elements are not stored in a single contiguous block of memory.

For example, `std::deque` iterators are generally random-access, but they are not contiguous. Converting such an iterator to a raw pointer and then assuming that `raw + n` reaches the corresponding element is incorrect. Therefore, `std::to_address` should not be used with every arbitrary iterator; its use is appropriate for contiguous iterators and pointer-like types that satisfy the relevant contract.

### The Past-the-End Iterator

Under the contiguous-iterator model, `std::to_address` can also have meaning for an iterator that refers to the past-the-end position of a range, even though that iterator cannot be dereferenced. However, the pointer obtained for the past-the-end position is suitable only for representing the range boundary and for permitted pointer-arithmetic operations; it must not be dereferenced.

When designing algorithms, distinguish between the address at the beginning of a range, the address at its past-the-end position, and the address of an accessible object.

## Applications in Allocators and Memory Management

### Obtaining an Address for Object Construction

One important motivation for this API is simplifying allocator-generic code. The type `allocator_traits<A>::pointer` may be a raw pointer or a fancy pointer. Generic code can use `std::to_address` to access the raw storage address.

```cpp
#include <memory>
#include <new>

template<class Alloc>
auto create_one(Alloc& alloc) {
    using traits = std::allocator_traits<Alloc>;
    using value_type = typename traits::value_type;

    auto p = traits::allocate(alloc, 1);
    value_type* raw = std::to_address(p);

    try {
        traits::construct(alloc, raw);
    } catch (...) {
        traits::deallocate(alloc, p, 1);
        throw;
    }

    return p;
}

template<class Alloc>
void destroy_one(Alloc& alloc,
                 typename std::allocator_traits<Alloc>::pointer p) {
    using traits = std::allocator_traits<Alloc>;
    auto* raw = std::to_address(p);

    traits::destroy(alloc, raw);
    traits::deallocate(alloc, p, 1);
}
```

In this pattern, the allocator's original handle or pointer is preserved for `deallocate`; the raw address is used only for operations such as object construction and destruction. This distinction matters because a custom allocator may require its own pointer type for deallocation.

### The Difference Between Storage and Object Lifetime

Obtaining the address of storage does not start an object's lifetime. `std::to_address` does not allocate memory, execute a constructor, create an object, or end an object's lifetime.

An object must be created using an appropriate mechanism, such as `allocator_traits::construct` or construction and lifetime facilities suited to the design. To end an object's lifetime, its destructor or the corresponding mechanism must be invoked. Finally, the memory must be released using the same allocator and the rules corresponding to the original allocation.

## Differences from Similar Facilities

### Comparison with `std::addressof`

The `std::addressof` function obtains the address of an object or function that is accessible through a reference, and it avoids any overloaded `operator&`. However, using it with a pointer usually requires dereferencing the pointer first:

```cpp
auto* raw = std::addressof(*p);
```

By contrast, `std::to_address(p)` extracts the address directly from a pointer-like object and does not require forming a reference through dereferencing. This makes it suitable for allocated storage that does not yet contain a constructed, live object.

### Comparison with `operator->`

The `operator->` operator is part of a class's interface and can lead to a raw pointer or another pointer-like proxy type. Calling it directly makes code dependent on the details of the pointer type.

`std::to_address` provides a higher-level standard abstraction: it first checks for the `pointer_traits` customization and then follows the `operator->` chain if that customization is unavailable. Therefore, it is generally a better choice for generic code.

### Comparison with `std::pointer_traits<Ptr>::pointer_to`

The `pointer_to` member of `std::pointer_traits` creates the corresponding pointer-like object from a reference to an object. The optional `to_address` member provides the reverse direction for types that implement it.

These two functions are not necessarily available as a complete pair for every pointer-like type. `to_address` is an optional member of the specialization, and you must not assume that every type providing `pointer_to` also provides a custom `to_address`.

## Important Properties and Limitations

### The `noexcept` Guarantee

Both standard overloads of `std::to_address` are declared `noexcept`. Consequently, address extraction, as exposed by this interface, must not propagate an exception to the caller. If a custom implementation performs an operation that may throw, that design is incompatible with the `noexcept` contract. If an exception escapes a `noexcept` function, the program encounters `std::terminate`.

### Function-Pointer Limitation

The raw-pointer overload does not permit a function type. Therefore, `std::to_address` is not a general-purpose tool for converting every kind of pointer to an address and must not be used with function pointers.

### Requirement for a Valid Specialization

In the fancy-pointer path, if instantiating `std::pointer_traits<Ptr>` is itself invalid, the program may encounter a hard compile-time error. This situation is not necessarily converted into a simple `false` result in the immediate context of a requires-expression. Therefore, do not assume that an arbitrary type can be passed to `std::to_address` without having an appropriate pointer-like structure.

### A Raw Address Does Not Transfer Ownership

The raw pointer returned by `std::to_address` does not acquire ownership of the memory or object. Its validity depends on the lifetime, allocation state, mapping, and invalidation rules of the original pointer-like object.

If a container or allocator moves the memory, releases it, or changes the corresponding mapping, a previously obtained raw pointer may become invalid. Therefore, do not retain it longer than the original pointer remains valid unless the relevant type's contract explicitly guarantees that doing so is safe.

### The Function Performs No General Validation

This function does not check whether the pointer refers to live memory, whether dereferencing it is permitted, whether the accessible range is valid, or whether the expected object has been constructed. Responsibility for these guarantees belongs to the pointer type, allocator, and calling code.

## Common Usage Mistakes

- **Confusing random access with contiguity:** Supporting iterator addition and subtraction does not guarantee that elements are stored contiguously in memory.
- **Using a raw pointer for deallocation:** If an allocator has its own pointer type, pass the original pointer to `deallocate`; the raw pointer returned by `std::to_address` is not a general replacement.
- **Assuming an object exists after allocation:** Allocating storage does not, by itself, construct an object.
- **Keeping a raw pointer for too long:** A raw pointer may become invalid after deallocation, relocation, or a change in mapping.
- **Using the function with function pointers:** This use is incompatible with the requirements of the raw-pointer overload.
- **Writing an incompatible customization:** A custom function must return the address corresponding to the pointer and satisfy the `noexcept` contract.

## Design Considerations for Library Authors

### Designing a Fancy Pointer

If you are designing a fancy pointer, first determine whether `operator->` can correctly and reliably provide the raw address or whether your type needs a dedicated customization. For pointers such as offset pointers, whose address depends on a base address or the current mapping, conversion must occur at the time of the call and account for the current valid memory state.

### Preserving the Original Pointer Type

In allocator-aware APIs, preserve the original pointer type for ownership and deallocation operations. Extract a raw pointer only at boundaries where a lower-level API genuinely requires one, such as placement construction or interaction with an API based on `T*`.

### Limiting Dependence on Raw Addresses

In portable designs, preserve the pointer abstraction as long as possible. Converting to a raw pointer too early can eliminate benefits such as relocatability, shared-memory addressing, or access control.

### Using the Appropriate Concept

If a template genuinely requires contiguous memory, express its constraint using `std::contiguous_iterator`, not merely `std::random_access_iterator`. This makes the algorithm's requirements explicit and prevents it from accepting iterators that support only random access.

## Standard Versions and Compatibility

The `std::to_address` function has been part of the standard library since C++20. The `std::pointer_traits` class template has existed since C++11, but the optional `pointer_traits::to_address` member was also introduced in C++20. The `std::contiguous_iterator` concept likewise belongs to C++20.

The following feature-test macro can be used to detect standard-library support:

```cpp
#include <memory>

#if defined(__cpp_lib_to_address) && __cpp_lib_to_address >= 201711L
// std::to_address is available.
#endif
```

The feature-test macro value for this functionality is `201711L`. The presence of this macro indicates standard-library support and should not be confused with frontend support for the C++20 language mode alone; the version of the standard library being used also matters.

In projects that must be built with standards earlier than C++20, `std::to_address` is unavailable. In such projects, you can write a helper compatible with older language versions for a limited, known set of pointer types. However, implementing a general equivalent is not straightforward, especially when custom fancy pointers or different iterator types are involved. It is preferable to use a dedicated, documented helper and avoid assuming that every iterator with an `operator->` is necessarily contiguous.

## Complete Practical Example

The following example shows a generic function that accepts an allocator, allocates storage for one element, obtains its raw address, and constructs the object using the allocator's construction mechanism. If construction fails, the storage is released.

```cpp
#include <memory>
#include <utility>

template<class Alloc, class... Args>
auto allocate_and_construct(Alloc& alloc, Args&&... args) {
    using traits = std::allocator_traits<Alloc>;
    using pointer = typename traits::pointer;
    using value_type = typename traits::value_type;

    pointer p = traits::allocate(alloc, 1);
    value_type* raw = std::to_address(p);

    try {
        traits::construct(alloc, raw, std::forward<Args>(args)...);
    } catch (...) {
        traits::deallocate(alloc, p, 1);
        throw;
    }

    return p;
}

template<class Alloc>
void destroy_and_deallocate(
    Alloc& alloc,
    typename std::allocator_traits<Alloc>::pointer p) {

    using traits = std::allocator_traits<Alloc>;
    using value_type = typename traits::value_type;

    value_type* raw = std::to_address(p);

    traits::destroy(alloc, raw);
    traits::deallocate(alloc, p, 1);
}
```

This example is an educational pattern for a single object. In production code, you should also consider allocator behavior in the event of failure, the number of elements, alignment, ownership rules, and the exception-safety requirements of the overall container or abstraction.

## Summary

The `std::to_address` function in C++20 provides a standard way to convert a raw pointer or pointer-like type to its corresponding raw address. It extracts the address without dereferencing the pointer, which makes it particularly important in allocator-generic code and in situations where storage has been allocated but does not yet contain a live object.

For fancy pointers, the function first checks for the optional customization `std::pointer_traits<Ptr>::to_address` and, if it is unavailable, follows the `operator->` chain. The function can also be used with iterators that genuinely model the `std::contiguous_iterator` concept; random-access capability alone is not sufficient.

As a practical rule, whenever an API genuinely requires a raw pointer and its input may be a standard or custom pointer-like type, consider `std::to_address` as the standard conversion point. At the same time, manage ownership, object lifetime, memory validity, and the pointer type required for deallocation separately.

## References

- [cppreference: `std::to_address`](https://en.cppreference.com/w/cpp/memory/to_address)
- [cppreference: `std::pointer_traits`](https://en.cppreference.com/w/cpp/memory/pointer_traits)
- [cppreference: `std::pointer_traits::to_address`](https://en.cppreference.com/w/cpp/memory/pointer_traits/to_address)
- [Working Draft: Pointer conversion](https://eel.is/c++draft/pointer.conversion)
- [Working Draft: Pointer traits](https://eel.is/c++draft/pointer.traits)
- [Working Draft: `contiguous_iterator`](https://eel.is/c++draft/iterator.concept.contiguous)
- [WG21 P3349R1: Converting contiguous iterators to pointers](https://www.open-std.org/JTC1/SC22/WG21/docs/papers/2025/p3349r1.html)


---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>