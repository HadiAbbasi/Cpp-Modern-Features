<div align="center">

[🇺🇸 English](./final.md) | [🇮🇷 فارسی](../../fa/cpp11/final.md)

</div>

---

# Introduction to `final` in C++11

In C++11, the `final` specifier is used to impose restrictions on inheritance and overriding virtual functions. It is a language feature and does not depend on the Standard Library.

The `final` specifier has two primary uses: a `class` can be marked as the final class in an inheritance hierarchy, or a `virtual function` can be specified so that derived classes can no longer override it.

Proper use of `final` can make class design contracts more explicit and prevent unintended inheritance or overriding.

## The Concept of `final` in C++11

In C++11, `final` is a **contextual keyword**; unlike ordinary keywords, it has a special meaning only in specific contexts.

Its two primary uses are:

* Preventing a `virtual function` from being overridden
* Preventing other classes from deriving from a `class`

A simple example looks like this:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() final;
};
```

In this example, `process()` in `Derived` is the final override in this inheritance hierarchy. Therefore, another class cannot override it again.

The second use applies to classes:

```cpp
class FinalClass final {
public:
    void process();
};
```

In this case, no other class can derive from `FinalClass`.

## Why `final` Exists

Inheritance is an important feature of C++, but inheritance is not always desirable. Sometimes a class is designed so that overriding a particular function or deriving from the class could violate internal invariants, API contracts, or expected behavior.

For example, suppose a class has an algorithm where the order in which several steps are executed is important:

```cpp
class Processor {
public:
    virtual void start();
    virtual void execute();
    virtual void finish();
};
```

The class design may require `finish` to no longer be customizable. In that case, `final` can be used:

```cpp
class SafeProcessor : public Processor {
public:
    void finish() final;
};
```

The compiler will then reject any subsequent attempt to override `finish`.

Instead of relying on documentation or informal agreements, this approach records the design restriction directly in the type system and the rules of the C++ language.

## Preventing Overrides with `final`

To prevent a `virtual function` from being overridden, `final` must appear after the function declaration or definition:

```cpp
class Base {
public:
    virtual void run();
};

class Derived : public Base {
public:
    void run() final;
};
```

In this example, `run` remains a virtual function, but it becomes the final allowed override in the inheritance hierarchy.

If another class derives from `Derived`, it cannot override `run`:

```cpp
class MoreDerived : public Derived {
public:
    void run() override;
};
```

The code above must be rejected by the compiler because the directly inherited version of `run` is marked `final`.

## Combining `final` and `override`

Combining `override` and `final` is one of the most common and useful applications of these specifiers:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() override final;
};
```

Two independent contracts are expressed here:

* `override` guarantees that `process` actually overrides a virtual function from the base class.
* `final` guarantees that derived classes can no longer override that override.

Using both keywords is often more precise than using `final` alone, because if the function signature is incorrect, `override` exposes the error.

For example:

```cpp
class Base {
public:
    virtual void process(int value);
};

class Derived : public Base {
public:
    void process() final;
};
```

This code is invalid because `process()` has a different signature and does not override the base function.

As a result, `override` provides an important additional layer of compiler checking.

## Difference Between `final` and `override`

These two specifiers serve different purposes.

| Feature                                     | `override` | `final`                       |
| ------------------------------------------- | ---------- | ----------------------------- |
| Checks whether a function overrides another | Yes        | Yes, when used on an override |
| Prevents further overrides                  | No         | Yes                           |
| Prevents inheritance                        | No         | Yes, for a class              |
| Introduced in                               | C++11      | C++11                         |

Conceptually, `override` is about the **correctness of an overriding relationship**, while `final` is about **ending the ability to change that relationship**.

The following example demonstrates both concepts:

```cpp
class Base {
public:
    virtual void draw();
};

class Button : public Base {
public:
    void draw() override final;
};
```

Here, `Button` is allowed to override `draw`, but further derived classes are not allowed to provide another override.

## Preventing Inheritance with `final`

To prevent other classes from deriving from a class, place `final` after the class name:

```cpp
class NetworkConnection final {
public:
    void connect();
};
```

The following code is now invalid:

```cpp
class SecureConnection : public NetworkConnection {
};
```

The compiler must diagnose an attempt to derive from a `final` class.

This usage is useful when a class is not designed for inheritance or when creating subclasses would be invalid from a design perspective.

## `final class` vs. a Class Without Virtual Functions

A class that has no virtual functions is not necessarily a `final` class.

For example:

```cpp
class Configuration {
public:
    void load();
};
```

Another class can still derive from it:

```cpp
class ApplicationConfiguration : public Configuration {
};
```

If the design goal is to prohibit all subclasses, the restriction must be stated explicitly:

```cpp
class Configuration final {
public:
    void load();
};
```

Therefore, the absence of `virtual` does not by itself prohibit inheritance.

## Effect of `final` on Dynamic Dispatch

Using `final` does not change the basic behavior of virtual dispatch; it restricts the possibility of further overrides.

For example:

```cpp
class Base {
public:
    virtual void run();
};

class Derived : public Base {
public:
    void run() final;
};
```

If a reference or pointer to `Base` refers to a `Derived` object, dynamic dispatch still takes place:

```cpp
Base* object = new Derived;
object->run();
```

`Derived::run` will be executed.

Therefore, `final` does not mean that the virtual function is automatically converted into a non-virtual function in the language semantics.

## Effect of `final` on Performance

`final` sometimes appears in discussions about optimization because, under certain circumstances, a compiler can use the limited set of possible overrides for optimization.

For example:

```cpp
class Base {
public:
    virtual void run();
};

class Derived final : public Base {
public:
    void run() override;
};
```

If the compiler can determine that the runtime type belongs to a limited set of possible types, it may be able to devirtualize the call.

However, `final` should not be viewed primarily as an optimization keyword.

The primary purpose of `final` is **to express design constraints and prevent unintended overriding or inheritance**.

Any performance improvement depends on the compiler, optimization level, program context, and the information available to the compiler.

## Difference Between `final` and Making a Function Non-Virtual

Suppose a function is declared as follows:

```cpp
class Base {
public:
    void process();
};
```

This function is not virtual, so a derived class cannot override it in the polymorphic sense.

However, a derived class can still declare a function with the same name:

```cpp
class Derived : public Base {
public:
    void process();
};
```

This is **not overriding**; it is name hiding.

In contrast, in the following code:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() final;
};
```

`Derived::process` really does override the function, and `final` explicitly prohibits further overrides.

This distinction is important for correctly understanding `final`.

## Using `final` in a Multi-Level Hierarchy

The restriction imposed by `final` applies from the point where it is declared downward in the inheritance hierarchy.

For example:

```cpp
class A {
public:
    virtual void process();
};

class B : public A {
public:
    void process() override;
};

class C : public B {
public:
    void process() final;
};

class D : public C {
public:
    void process() override;
};
```

The code above is invalid because `C::process` is marked `final`.

However, if `final` is used only at a particular level, classes before that point remain subject to the normal inheritance rules.

## Using `final` on a Destructor

From a syntax perspective, a destructor can also be marked `final` because a destructor can be virtual:

```cpp
class Base {
public:
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    ~Derived() final = default;
};
```

In this case, classes derived from `Derived` cannot define their own destructor as an override.

This is a highly specialized use and is usually unnecessary for general-purpose class design.

If the goal is simply to prevent inheritance, marking the class itself as `final` is generally clearer:

```cpp
class Derived final {
public:
    ~Derived() = default;
};
```

Therefore, `final` on a destructor should be used for a specific purpose rather than simply as an alternative to a `final` class.

## Using `final` in Polymorphic APIs

In the design of polymorphic APIs, `final` can be used to define the boundaries of an inheritance hierarchy.

For example:

```cpp
class Document {
public:
    virtual ~Document() = default;
    virtual void save() = 0;
    virtual void validate() = 0;
};

class PdfDocument final : public Document {
public:
    void save() override;
    void validate() override;
};
```

In this design, `Document` is intended for inheritance, while `PdfDocument` is provided as a final implementation.

This approach is appropriate when an API has multiple implementations but some implementations should not be specialized further.

## Using `final` in Interfaces

Interfaces generally should not be marked `final`, because their purpose is to allow other classes to implement them through inheritance.

For example:

```cpp
class Renderer {
public:
    virtual ~Renderer() = default;
    virtual void render() = 0;
};
```

Marking `Renderer` as `final` would conflict with that design:

```cpp
class Renderer final {
public:
    virtual void render() = 0;
};
```

In contrast, a concrete implementation of an interface can be `final`:

```cpp
class OpenGLRenderer final : public Renderer {
public:
    void render() override;
};
```

Therefore, `final` should be used according to the role of the class in the architecture rather than mechanically.

## Using `final` to Preserve Invariants

One important reason to use `final` is when correctness depends on a particular implementation.

For example:

```cpp
class Transaction {
public:
    virtual ~Transaction() = default;
    virtual void commit();
    virtual void rollback();
};

class AtomicTransaction final : public Transaction {
public:
    void commit() override;
    void rollback() override;
};
```

If the internal logic of `AtomicTransaction` assumes that no subclass can alter the behavior of `commit` or `rollback`, `final` can enforce that contract.

In such cases, `final` is part of the correctness design rather than merely a syntactic or performance-related feature.

## Common Mistake: Using `final` Without `virtual`

A common mistake is to use `final` when there is no virtual function to override.

For example:

```cpp
class Base {
public:
    void process();
};

class Derived : public Base {
public:
    void process() final;
};
```

This code is invalid because `Derived::process` cannot override a non-virtual function.

If the intention is to use `final` on a function, that function must participate in a valid virtual override chain.

Using `override` as well can make the intent clearer:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() override final;
};
```

## Common Mistake: The Ordering of Specifiers

Both `override` and `final` can be used in a function declaration:

```cpp
void process() override final;
```

or:

```cpp
void process() final override;
```

Both orders are valid for these two specifiers.

However, adopting a consistent style such as `override final` usually improves readability and makes the codebase more uniform.

The important point is that `override` and `final` must appear after the function declarator:

```cpp
void process() override final;
```

They should not be placed in a position where they could be interpreted as part of the function's type or name.

## Common Mistake: Treating `final` as an Access Modifier

`final` has nothing to do with `public`, `protected`, or `private`.

For example:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
private:
    void process() final;
};
```

Here, `process` is both `private` and `final`.

These are independent properties:

* Access control determines which code can access a member.
* `final` determines whether further overriding is allowed.

## Common Mistake: Using `final` Instead of Composition

`final` should not be used as a way to compensate for an inappropriate inheritance hierarchy.

If a class derives from another class merely to reuse implementation, composition may be a more appropriate design.

Inheritance expresses an «is-a» relationship, while composition generally models a «has-a» relationship.

Using `final` cannot turn an inappropriate inheritance hierarchy into an appropriate one.

If a subclass has no meaningful semantic relationship with its base class, the design should be reconsidered before adding `final`.

## Difference Between `final` and `private`

Older techniques sometimes used private constructors or other access-control mechanisms to prevent inheritance.

In C++11, if the actual goal is to explicitly prohibit inheritance, `final` is generally a clearer way to express that intent:

```cpp
class Utility final {
public:
    static void process();
};
```

By contrast, a `private` constructor addresses a different concern and can affect how objects are created.

Therefore, these mechanisms should not be considered interchangeable.

## Difference Between `final` and Older Techniques for Preventing Inheritance

Before C++11, techniques such as private constructors, private destructors, or specialized design patterns were sometimes used to prevent inheritance.

These approaches usually expressed the intent indirectly.

In C++11, the intent can be expressed directly:

```cpp
class NonExtendable final {
};
```

The primary advantage is clarity and compile-time diagnostics.

`final` also communicates to both the compiler and other developers that preventing inheritance is an intentional design decision.

## Difference Between `final` and `sealed` in Other Languages

Some other programming languages provide concepts such as `sealed` for preventing inheritance.

In C++11, the standard keyword used for this purpose is `final`.

Therefore, the term `sealed class` may be useful when explaining the concept, but the standard C++11 syntax is:

```cpp
class Example final {
};
```

In C++, `final` can be used not only on classes but also on virtual functions.

## Limitations of `final`

The `final` feature is meaningful only in the contexts defined by the C++ standard.

It cannot:

* restrict object creation.
* control member access.
* provide thread safety.
* make objects immutable.
* manage object lifetime.
* prevent composition.
* guarantee class correctness by itself.

In other words, `final` is a focused tool for controlling inheritance and virtual overriding and should not be assigned responsibilities beyond that scope.

## ABI and Library Design Considerations

In public libraries, using `final` can become part of the API contract.

If a class is published as `final`, library users cannot derive from it. Therefore, adding or removing `final` can affect the capabilities available to library users.

For example:

```cpp
class LibraryComponent final {
public:
    virtual ~LibraryComponent() = default;
    virtual void process();
};
```

This API effectively tells users that the class is the endpoint of the inheritance hierarchy.

Therefore, `final` should be used carefully in public APIs and with consideration for its restrictive effect.

## Design Considerations for Large Projects

In large projects, `final` should be used according to actual design contracts rather than simply adding more restrictions.

If a class is not intended for inheritance, using `final` can make that intent explicit:

```cpp
class JsonParser final {
public:
    void parse();
};
```

However, if there is a legitimate possibility that the class needs to be extended through inheritance, using `final` may unnecessarily reduce API flexibility.

A useful rule is to use `final` when **prohibiting extension is part of the design**.

## Using `final` with an Abstract Base Class

A common pattern is to provide an abstract base class for extension and make specific implementations final:

```cpp
class Storage {
public:
    virtual ~Storage() = default;
    virtual void read() = 0;
    virtual void write() = 0;
};

class FileStorage final : public Storage {
public:
    void read() override;
    void write() override;
};
```

In this design, `Storage` is the extension point, while `FileStorage` is considered a final implementation.

This pattern is frequently useful in architectures where dependency inversion and polymorphism are important.

## Using `final` in Templates

`final` can also be used on a class template:

```cpp
template <typename T>
class Container final {
public:
    void process(const T& value);
};
```

Every specialization produced from this template is a `final` class.

Therefore, being a template does not change the inheritance restriction imposed by `final`.

A specialization can still be defined normally, but the inheritance restriction remains determined by the final declaration of the resulting type.

## Using `final` with Multiple Inheritance

The rules for `final` also apply to multiple inheritance.

For example:

```cpp
class A {
public:
    virtual void process();
};

class B {
public:
    virtual void process();
};

class C : public A, public B {
public:
    void process() override final;
};
```

In this case, `C::process` can provide the final override for the virtual functions inherited from the base classes, provided their signatures and override relationships are compatible.

If another class derives from `C`, it can no longer override `process`:

```cpp
class D : public C {
public:
    void process() override;
};
```

This code is invalid.

## Important Notes About `final` and Virtual Destructors

If a class is intended to be polymorphic, its destructor should generally be virtual:

```cpp
class Base {
public:
    virtual ~Base() = default;
};
```

This issue is independent of `final`.

If a class is a final implementation, the class itself can be marked `final`:

```cpp
class Derived final : public Base {
public:
    ~Derived() override = default;
};
```

Here, `override` on the destructor indicates that the base destructor is virtual.

For polymorphic designs, this pattern is often clear in terms of intent.

## Important Notes About the Name `final`

Because `final` is a contextual keyword, C++ allows it to be used as an identifier in certain contexts, provided the relevant grammar does not interpret it as the specifier.

Nevertheless, using names such as `final` in new code is generally not a good choice because it can reduce readability and create conceptual ambiguity with the standard meaning of `final`.

Clear, non-ambiguous identifiers are preferable.

## Differences Between C++ Standard Versions

The `final` feature was introduced in **C++11**.

It remains part of the language in later standards, including C++14, C++17, C++20, and C++23.

Therefore, code using `final` does not inherently require C++14, C++17, or a later standard; `final` has been available since C++11.

The standard `final` specifier for this purpose did not exist in C++98 or C++03.

## Comparison with Designs Without `final`

Without `final`, a hierarchy may look like this:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() override;
};

class MoreDerived : public Derived {
public:
    void process() override;
};
```

In this structure, every level can change the behavior of the virtual function.

If the design requires `Derived` to be the final implementation, it can be written as follows:

```cpp
class Base {
public:
    virtual void process();
};

class Derived final : public Base {
public:
    void process() override;
};
```

In this case:

```cpp
class MoreDerived : public Derived {
};
```

is invalid.

If only the function should be final while the rest of the class should remain available for inheritance, `final` should be placed on the function:

```cpp
class Derived : public Base {
public:
    void process() override final;
};
```

This distinction is one of the most important design decisions when using `final`.

## Important Points for Code Review

When reviewing code that uses `final`, it is useful to ask several questions:

* Should the class really be non-derivable?
* Should the function really not be overridden by further subclasses?
* Is this restriction part of the class contract?
* Should `override` also be used?
* Is `final` being used to compensate for a deeper design problem?
* Does this restriction have an undesirable effect on the public API?
* Would composition be a better alternative to inheritance?
* Is `final` being added merely for speculative optimization?

If the answers show that the restriction is not actually part of the design, `final` may not be the appropriate choice.

## Recommended Pattern for Virtual Functions

For a class that overrides a function and should provide the final override, the following pattern is generally clear and explicit:

```cpp
class Base {
public:
    virtual ~Base() = default;
    virtual void process();
};

class Derived : public Base {
public:
    void process() override final;
};
```

In this pattern, the compiler checks both that the function is an override and that no further override is permitted.

## Recommended Pattern for a Final Class

For a class that should not have subclasses, the following simple pattern is sufficient:

```cpp
class JsonParser final {
public:
    void parse();
};
```

There is no need to mark every member function with `final` merely to indicate that the class itself is final.

`final` on the class already prohibits inheritance.

## Common Compiler Diagnostics

If a `final` function is overridden again, the compiler must reject the program.

For example:

```cpp
class Base {
public:
    virtual void run();
};

class Derived : public Base {
public:
    void run() final;
};

class MoreDerived : public Derived {
public:
    void run() override;
};
```

In this example, `MoreDerived::run` cannot override `Derived::run`.

Likewise, attempting to derive from a `final` class:

```cpp
class Base final {
};

class Derived : public Base {
};
```

must also be rejected during compilation.

The exact diagnostic message varies between implementations and is not part of the standard; what the standard requires is that the program be ill-formed.

## Difference Between the Standard and Compiler Extensions

`final` is a standard language feature introduced in C++11 and is not a compiler extension.

Therefore, conforming compilers that support C++11 must implement its rules.

By contrast, details such as the exact wording of diagnostic messages or optimizations performed as a result of `final` are implementation-specific.

For example, code should not depend on a particular compiler's error message.

## Practical Recommendations

For modern C++ design, several simple guidelines can be followed:

* Use `final` on a virtual function to prevent further overriding.
* Use `final` on a class to prevent inheritance.
* Prefer using `override` together with `final` for actual overrides.
* Do not use `final` merely based on assumptions about performance.
* Consider the restrictive effect of `final` when designing public APIs.
* If inheritance is not conceptually appropriate, consider composition first.
* Use `final` to express a real design decision rather than as decoration.
* During code review, consider the distinction between `override`, `final`, and non-virtual functions.

## Final Summary

In C++11, `final` is a standard tool for restricting polymorphism and inheritance.

It has two primary uses: it can prevent a `virtual function` from being overridden further, and it can prevent a `class` from being derived from.

Using `override` together with `final` often communicates the intent of the code clearly:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() override final;
};
```

Likewise, inheritance can be completely prohibited with the following pattern:

```cpp
class Component final {
public:
    void process();
};
```

The key point is that `final` is primarily a **tool for expressing and enforcing a design contract**, rather than an optimization feature. It should be used when the class design genuinely requires an implementation or inheritance hierarchy to end at a specific point.

---

## 🤝 Contributors

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>