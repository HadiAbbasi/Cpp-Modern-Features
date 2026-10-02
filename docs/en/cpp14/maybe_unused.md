<div align="center">

[🇺🇸 English](./maybe_unused.md) | [🇮🇷 فارسی](../../fa/cpp14/maybe_unused.md)

</div>

---

# Introduction to `[[maybe_unused]]` in C++

The `[[maybe_unused]]` attribute is one of the **standard attributes** of the C++ language. It has been available since C++17 and is used to indicate that an entity may intentionally remain unused. Its primary purpose is to prevent compiler warnings about unused entities without requiring the code structure to be changed merely to silence a warning.

This feature is part of the **C++ language**, not the Standard Library, so no special header is required to use it.

```cpp
[[maybe_unused]]
int debug_value = 42;
```

In this example, if `debug_value` is not used anywhere in the program, the compiler can suppress the warning associated with the unused variable.

## Understanding Unused Entity Warnings

Warnings about unused entities usually appear when the compiler determines that a name, variable, function, parameter, or other entity has been declared but is ultimately not used.

For example:

```cpp
void process(int value)
{
    // The parameter is intentionally unused.
}
```

Depending on the compiler warnings that are enabled, `value` may produce a warning.

Such warnings are generally useful because a genuinely unused variable can indicate a bug, incomplete refactoring, or a design mistake. However, not every unused entity is a mistake.

For example, a parameter may exist solely to maintain compatibility with an interface:

```cpp
class Handler
{
public:
    void on_event(int event_id)
    {
        // The interface requires the parameter.
    }
};
```

In such situations, `[[maybe_unused]]` allows the programmer to explicitly express that the unused status is intentional.

## Using `[[maybe_unused]]`

The basic syntax for the attribute is:

```cpp
[[maybe_unused]] declaration;
```

This attribute can be used with several kinds of declarations and has been part of the C++ language standard since C++17.

A simple example for a variable is:

```cpp
[[maybe_unused]] int value = 42;
```

The attribute can also be placed after the variable name:

```cpp
int value [[maybe_unused]] = 42;
```

Both forms serve the same purpose, although a codebase should preferably adopt a consistent convention for readability.

## Using the Attribute with Variables

The most common use of `[[maybe_unused]]` is for local variables and objects.

```cpp
void initialize()
{
    [[maybe_unused]] const auto configuration = load_configuration();
}
```

This can be useful when a value is used in some build configurations but not others.

A more common scenario occurs when code differs between debug and release builds:

```cpp
void process()
{
    [[maybe_unused]] const auto result = calculate_result();

#ifdef ENABLE_LOGGING
    log_result(result);
#endif
}
```

In builds where `ENABLE_LOGGING` is not enabled, `result` may be unused.

However, the attribute should not be used as an excuse to keep variables that are genuinely no longer needed.

## Using the Attribute with Function Parameters

One of the most important uses of this attribute is for function parameters.

```cpp
void callback([[maybe_unused]] int event_id)
{
    notify_observer();
}
```

Here, `event_id` may be part of a callback interface even though the current implementation does not need it.

This technique is particularly useful for interface implementations, callbacks, framework APIs, and template code.

Multiple parameters can be annotated independently:

```cpp
void callback(
    [[maybe_unused]] int event_id,
    [[maybe_unused]] int flags)
{
    notify_observer();
}
```

Applying the attribute only to parameters that are actually unused generally makes the intent clearer.

## Using the Attribute with Functions

A function declaration itself can also be marked with `[[maybe_unused]]`:

```cpp
[[maybe_unused]] void debug_dump()
{
    // Debug-only implementation.
}
```

This can be useful when the existence of a function depends on configuration, build mode, or platform.

For example:

```cpp
[[maybe_unused]] void trace_message(const char* message)
{
    write_trace(message);
}
```

If this function has no call sites in a particular build, the attribute can suppress the corresponding unused-function warning.

However, if the function is genuinely no longer used and there is no architectural reason to keep it, removing it is generally better than using `[[maybe_unused]]`.

## Using the Attribute with Types

The attribute can also be applied to certain type declarations.

```cpp
struct [[maybe_unused]] DebugState
{
    int counter;
};
```

If the type is used only in some configurations, this can prevent warnings about the unused type.

The same capability applies to `enum` declarations:

```cpp
enum [[maybe_unused]] class LogLevel
{
    info,
    warning,
    error
};
```

In such cases, the attribute applies to the entity itself rather than to its members.

## Using the Attribute with Enumerators

An individual enumerator can also be marked with `[[maybe_unused]]`.

```cpp
enum class Status
{
    success,
    failure,
    legacy [[maybe_unused]]
};
```

This can be useful when a set of enumerators must be retained for ABI compatibility, protocol compatibility, or an external interface even though one value is not used by the current implementation.

Note that an unused enumerator and an unused enum are two different concepts.

## Using the Attribute with `typedef` and Aliases

In C++17, `[[maybe_unused]]` can also be applied to `typedef` and alias declarations.

```cpp
using FileHandle [[maybe_unused]] = int;
```

This can be useful in generic code, platform abstractions, and headers that are used across multiple configurations.

## Using the Attribute with Data Members

The attribute can be applied to non-static data members as well:

```cpp
struct State
{
    int active;

    [[maybe_unused]] int debug_counter;
};
```

This is less common than using it for local variables, but it can be appropriate for types whose members are retained for other reasons, such as layout or interface compatibility.

Static data members can also be annotated:

```cpp
struct Statistics
{
    [[maybe_unused]] static int debug_counter;
};
```

## Using the Attribute with Structured Bindings

`[[maybe_unused]]` can be used with structured bindings:

```cpp
#include <utility>

void process()
{
    [[maybe_unused]] auto [width, height] = std::pair{1920, 1080};
}
```

The attribute can also be applied to individual bindings:

```cpp
#include <utility>

void process()
{
    auto [width, height [[maybe_unused]]] = std::pair{1920, 1080};
}
```

Standard support for `[[maybe_unused]]` with structured bindings in C++17 was clarified through a defect report, so current documentation considers this capability part of C++17.

## Comparing `[[maybe_unused]]` with Removing a Variable

If a variable genuinely has no purpose, the first question should not be how to suppress its warning.

For example:

```cpp
void process()
{
    [[maybe_unused]] int unused_value = expensive_calculation();
}
```

This code may produce no warning, but `expensive_calculation()` is still executed.

The attribute does not remove the initialization or prevent the expression from being evaluated.

If the value is genuinely unnecessary, removing it may be the correct solution:

```cpp
void process()
{
    perform_processing();
}
```

This distinction is important: `[[maybe_unused]]` concerns **diagnostics**, not runtime behavior or optimization.

## Comparing `[[maybe_unused]]` with a Cast to `void`

Before C++17, a common way to suppress a warning about an unused parameter was to cast it to `void`:

```cpp
void callback(int event_id)
{
    static_cast<void>(event_id);
}
```

This technique is still valid, but `[[maybe_unused]]` expresses the intent more directly:

```cpp
void callback([[maybe_unused]] int event_id)
{
}
```

The second form tells the reader, directly in the declaration, that the parameter may intentionally be unused.

By contrast, `static_cast<void>(event_id)` creates an actual expression in the function body and is more of a workaround for the diagnostic.

For modern C++ code, when the sole goal is to express intentional unused status, `[[maybe_unused]]` generally communicates the intent more clearly.

## Comparing `[[maybe_unused]]` with `std::ignore`

In some code, `std::ignore` is used to discard a value:

```cpp
#include <tuple>

void process()
{
    auto [value, status] = get_result();
    std::ignore = status;
}
```

However, `std::ignore` was primarily designed for use with `std::tie` and has other related uses.

If the goal is simply to indicate that a named variable is intentionally unused, `[[maybe_unused]]` communicates the intent more directly:

```cpp
void process()
{
    [[maybe_unused]] auto [value, status] = get_result();
}
```

By contrast, when the intent is specifically to ignore a value during unpacking, `std::ignore` may be the more appropriate choice.

## Comparing `[[maybe_unused]]` with Ignoring a Return Value

A common mistake is to assume that `[[maybe_unused]]` is a general solution for discarded return values.

Suppose a function is declared with `[[nodiscard]]`:

```cpp
[[nodiscard]] bool save();

void process()
{
    save();
}
```

The issue here concerns **ignoring a return value**, not an unused declaration.

If the intention is genuinely to discard the return value, the intent should be expressed appropriately for the API and situation.

For example:

```cpp
void process()
{
    static_cast<void>(save());
}
```

This expression explicitly indicates that discarding the result is intentional.

Therefore, `[[maybe_unused]]` should not be treated as a general replacement for handling `[[nodiscard]]`.

## Comparing `[[maybe_unused]]` with `[[nodiscard]]`

The `[[nodiscard]]` and `[[maybe_unused]]` attributes work in almost opposite directions.

The `[[nodiscard]]` attribute tells the compiler that **ignoring the result of an entity is important and should produce a diagnostic**.

```cpp
[[nodiscard]] bool commit();
```

By contrast, `[[maybe_unused]]` tells the compiler that **an entity being unused may be intentional and should not produce the corresponding unused warning**.

Therefore, these two attributes complement each other rather than replace each other.

## Limiting the Attribute to the Necessary Declaration

An important principle when using this attribute is to keep its scope as narrow as possible.

For example, if only one parameter is unused:

```cpp
void callback([[maybe_unused]] int event_id, int flags)
{
    use_flags(flags);
}
```

It is preferable to annotate only that parameter rather than marking the entire function without a specific reason.

This makes the intent more precise and allows genuine warnings to remain visible.

## Using the Attribute with Conditional Compilation

An important practical use of `[[maybe_unused]]` is in code built with `#if`, feature flags, or platform-specific compilation.

For example:

```cpp
void process()
{
    const int value = calculate_value();

#ifdef FEATURE_X
    use_feature_value(value);
#else
    (void)value;
#endif
}
```

In modern code, the declaration can instead be marked so that it does not generate a warning in different builds:

```cpp
void process()
{
    [[maybe_unused]] const int value = calculate_value();

#ifdef FEATURE_X
    use_feature_value(value);
#endif
}
```

This technique is particularly useful in headers and codebases that are built for multiple platforms or configurations.

## Using the Attribute in APIs and Interfaces

Suppose a framework interface passes a particular parameter to a callback, but the current implementation does not need it:

```cpp
class EventHandler
{
public:
    void handle(
        [[maybe_unused]] int event_id,
        const char* message)
    {
        print_message(message);
    }
};
```

Here, removing the parameter is not an option because the signature is part of the contract.

Using `[[maybe_unused]]` communicates that intent directly.

## Using the Attribute in Templates

Unused parameters or variables can also occur in template code when an entity is used only by some instantiations.

```cpp
template <typename T>
void process(T value)
{
    [[maybe_unused]] auto size = sizeof(T);

    use(value);
}
```

However, `[[maybe_unused]]` should not be used to hide poor template design.

If an entity is unused in most instantiations, the template's structure may need to be reconsidered.

## Using the Attribute in Debug and Release Builds

A common scenario involves a variable that is used only in a debug build.

```cpp
void process()
{
    auto result = calculate();

#ifndef NDEBUG
    assert(result.valid());
#endif

    submit(result);
}
```

In this example, `result` is still used in release builds, so `[[maybe_unused]]` is unnecessary.

However, if the variable exists only for an assertion:

```cpp
void process()
{
    [[maybe_unused]] const auto invariant = calculate_invariant();

#ifndef NDEBUG
    assert(invariant);
#endif
}
```

In a release build, the variable may become unused, and the attribute can explicitly express that situation.

However, it is important to consider whether a calculation that exists only for debugging introduces unnecessary runtime cost.

## Important Runtime Behavior Limitation

The `[[maybe_unused]]` attribute provides no guarantee that code, an object, or a function will be removed from the runtime program.

For example:

```cpp
[[maybe_unused]] auto value = expensive_operation();
```

The attribute does not mean that the compiler must eliminate `expensive_operation()`.

If the expression has side effects, removing it could change program semantics, and the compiler cannot simply eliminate it because of `[[maybe_unused]]`.

Therefore, two concepts must be distinguished:

* **Unused diagnostic**: an entity is not used.
* **Dead-code elimination**: the compiler determines that some code has no observable effect and can remove it.

`[[maybe_unused]]` concerns only the first concept.

## Limitations Regarding Real Bugs

One danger of excessive use of this attribute is hiding useful warnings.

Suppose a refactoring has been performed and a variable is no longer used:

```cpp
void process()
{
    [[maybe_unused]] auto result = calculate_result();

    send_default_response();
}
```

A developer might quickly add `[[maybe_unused]]` to eliminate the warning, even though `result` was supposed to be used by the new logic.

In such a situation, the attribute does not solve the underlying problem; it merely hides a useful warning.

A practical rule is to determine why an entity is unused first, and use the attribute only when the unused status is intentional.

## Differences Between Compilers

`[[maybe_unused]]` is a **standard attribute** and modern C++ compilers support it as part of the standard language.

However, exactly which warnings a compiler emits for unused entities is a detail of that compiler's diagnostic system.

For example, whether unused-entity warnings are enabled can depend on compiler warning flags.

Therefore, two concerns should be kept separate:

* The C++ standard defines the meaning of `[[maybe_unused]]`.
* The compiler determines which warnings are generated for unused entities and how those diagnostics are configured.

The attribute is a standard mechanism for expressing intent, while the diagnostic policy remains compiler-specific.

## Comparing Standard and Compiler-Specific Attributes

Before `[[maybe_unused]]` became standardized, different compilers provided their own mechanisms for this purpose.

For example, GNU toolchains provide compiler-specific attributes for similar use cases.

In modern portable C++ code, when the standard facility is available, using:

```cpp
[[maybe_unused]] int value = 42;
```

is generally preferable to compiler-specific alternatives.

Using a compiler extension may be reasonable in a project that deliberately depends on a specific compiler, but it should not be confused with a standard language feature.

## Differences Between C++ Standard Versions

The `[[maybe_unused]]` attribute was standardized in **C++17**.

In later C++ standards, the attribute remains part of the language.

Therefore, projects targeting C++17 or later can use it as a standard and portable mechanism.

For projects targeting standards before C++17, an approach compatible with the project's target standard and compiler must be used. For example, older codebases may use `static_cast<void>(value)` or compiler-specific macros.

## Changes Related to C++26

Newer standards expand the scope of attributes in certain cases. In C++26, `[[maybe_unused]]` can also be applied to unused labels.

For example:

```cpp
void process(bool condition)
{
    if (condition)
        goto done;

    [[maybe_unused]] done:
    return;
}
```

This can be useful in codebases that use labels and need to control diagnostics for unused labels.

However, the existence of this capability is not an endorsement of using `goto`; the feature is limited to handling unused-label diagnostics.

## Important Details About Attribute Placement

An important syntactic detail is that the attribute can appear in different positions within a declaration.

For example:

```cpp
[[maybe_unused]] int value = 42;
```

or:

```cpp
int value [[maybe_unused]] = 42;
```

Both forms are supported by the language, although the position must conform to the grammar of the entity being declared.

During code review, the chosen form should be consistent with the project's style guide.

## Important Considerations for Headers

In headers, `[[maybe_unused]]` can be useful for declarations that are not used in every translation unit.

For example:

```cpp
[[maybe_unused]] inline void debug_log()
{
    // Debug logging implementation.
}
```

This is particularly relevant for utility headers used across multiple configurations.

However, care must be taken not to use the attribute as a replacement for proper use of `inline`, `static`, internal linkage, or conditional compilation.

`[[maybe_unused]]` only controls diagnostics related to unused entities; it does not change linkage.

## Important Considerations for ODR and Linkage

The `[[maybe_unused]]` attribute does not directly change linkage or the One Definition Rule.

For example:

```cpp
[[maybe_unused]] void helper()
{
}
```

This declaration merely indicates that the entity may be unused.

If the goal is to prevent multiple definitions, control linkage, or manage visibility, the appropriate mechanism for that problem must be used.

Therefore, `[[maybe_unused]]` should not be confused with `static`, `inline`, or visibility attributes.

## Common Mistakes

One common mistake is using the attribute for every unused warning without first determining why the entity is unused.

Another mistake is assuming that the attribute causes a variable or function to be removed from the binary.

```cpp
[[maybe_unused]] auto result = expensive_operation();
```

This declaration can still perform initialization and produce side effects.

Another common mistake is applying the attribute to the wrong kind of entity. If the issue is a discarded return value, `[[maybe_unused]]` is not necessarily the appropriate solution.

Another important mistake is over-annotating declarations:

```cpp
[[maybe_unused]]
void process(
    [[maybe_unused]] int a,
    [[maybe_unused]] int b,
    [[maybe_unused]] int c)
{
}
```

If only `b` is genuinely unused, marking `a` and `c` as well provides no useful information about the intent.

## Choosing the Appropriate Approach

The appropriate approach depends first on the kind of problem involved.

If a variable may intentionally remain unused, `[[maybe_unused]]` is appropriate:

```cpp
[[maybe_unused]] int value = 42;
```

If a function parameter is required by an interface but is not needed by the implementation, applying the attribute to the parameter is appropriate:

```cpp
void callback([[maybe_unused]] int event_id)
{
}
```

If a function's return value is intentionally discarded, the intent should be expressed directly:

```cpp
static_cast<void>(save());
```

If a variable is genuinely no longer needed, removing it is generally better than suppressing its warning.

If a warning is caused by compiler-specific diagnostic policy, first determine whether the standard `[[maybe_unused]]` mechanism actually addresses the underlying requirement.

## A Practical Code Review Approach

During code review, a simple question can be asked about every use of `[[maybe_unused]]`:

> Is this entity being unused an intentional part of the design?

If the answer is yes, the attribute generally communicates the intent appropriately.

If the answer is no, the reason for the unused status should be investigated.

This approach turns `[[maybe_unused]]` into a tool for **documenting programmer intent**, rather than a tool for blindly suppressing warnings.

## A Comprehensive Example

The following example demonstrates several real-world uses together:

```cpp
#include <cassert>

enum class Status
{
    success,
    failure,
    legacy [[maybe_unused]]
};

struct [[maybe_unused]] DebugContext
{
    int counter{};
};

[[maybe_unused]] void trace_event(const char* message)
{
    // Emit diagnostic information.
}

void handle_event(
    [[maybe_unused]] int event_id,
    Status status)
{
    [[maybe_unused]] const bool valid =
        status == Status::success;

#ifndef NDEBUG
    assert(valid);
#endif
}
```

In this example, `[[maybe_unused]]` is used with several different entities, but each use has a specific reason:

* The `legacy` enumerator is retained for compatibility.
* `DebugContext` may be used only in certain configurations.
* The `trace_event` function may have no caller in a particular build.
* `event_id` is part of the callback contract.
* The `valid` variable is used in debug builds and may be unused in release builds.

This kind of usage is much better than placing the attribute on many entities without a clear reason.

## Important Guidelines for Modern Projects

For modern C++ projects, `[[maybe_unused]]` should be part of the diagnostic strategy rather than a replacement for diagnostics.

Keeping appropriate warnings enabled remains important. The purpose of the attribute is not to weaken warnings, but to distinguish **intentional unused entities** from **suspicious unused entities**.

It is also preferable to place the attribute as close as possible to the entity that may actually be unused. This makes the intent clear to both the compiler and other developers.

For library code, portability should also be considered, and the standard attribute should generally be preferred over compiler-specific extensions whenever possible.

## Final Summary

The `[[maybe_unused]]` attribute, introduced in C++17, is a standard and straightforward mechanism for indicating that an entity may intentionally remain unused. It can be applied to entities such as variables, parameters, functions, types, enumerators, data members, aliases, and structured bindings, and in C++26 its scope was extended to unused labels.

The key point is that `[[maybe_unused]]` **does not change program behavior**; it is intended to control diagnostics related to unused entities. Therefore, it should not be confused with optimization, dead-code elimination, `[[nodiscard]]`, `std::ignore`, or compiler-specific mechanisms for suppressing warnings.

The best use of this attribute is when an entity is intentionally unused, such as a required interface parameter, code that depends on build configuration, debug-only state, compatibility entities, or template code that does not need a particular entity in every case.

Conversely, if an entity is unused because of a bug or incomplete refactoring, using `[[maybe_unused]]` merely hides a useful warning. The primary value of this feature is therefore not simply suppressing warnings, but **expressing programmer intent clearly in the code**.


---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>