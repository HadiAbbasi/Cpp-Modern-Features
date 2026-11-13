<div align="center">

[🇺🇸 English](./deducing_this.md) | [🇮🇷 فارسی](../../fa/cpp23/deducing_this.md)

</div>

---

# **Deducing&#x20;**`this`**&#x20;in C++23**

## **Introduction**

**Deducing&#x20;**`this` is an important feature introduced in the C++23 standard to simplify Member Function design, reduce code duplication, improve `CRTP` implementations, and provide greater flexibility in Generic Programming.

Standardized through proposal **P0847R7 – Deducing this**, this feature allows programmers to explicitly declare a parameter that receives the current object in a Member Function.

Before C++23, the compiler implicitly provided an Object Parameter for every Non-static Member Function. This parameter could be accessed through the `this` keyword inside the function body, but programmers could not explicitly specify its type.

This limitation often required repetitive implementations, particularly in the following situations:

- Implementing multiple overloads for `const` and non-`const` Member Functions.
- Handling differences between Lvalue and Rvalue objects.
- Designing fluent or chainable APIs.
- Implementing `CRTP` patterns.
- Defining Recursive Lambdas.
- Designing Generic Member Functions whose behavior depends on the exact type of the object.

Deducing `this` addresses these limitations by allowing the Object Parameter to be declared explicitly, either with a specific type or through Template Argument Deduction.

With this feature, a Member Function can be designed to preserve the object's `const` qualification, reference category, and exact type without requiring multiple overloads.

## **Understanding the Object Parameter in C++**

### **How this Worked Before C++23**

In previous C++ standards, every Non-static Member Function had an Implicit Object Parameter.

Consider the following example:

```
class Widget {
public:
    void update(int value) {
        // Update the object
    }
};
```

Conceptually, the `update` function requires an object of type `Widget` on which to operate.

When the function is called as follows:

```
Widget widget;
widget.update(42);
```

The `widget` object is implicitly passed to the function.

Inside the function, the `this` keyword provides a pointer to that same object.

In a conventional Non-static Member Function, the compiler determines the type of `this` based on the class type and the function's qualifiers.

For example:

```
class Widget {
public:
    void modify() {
        // this has type Widget*
    }

    void inspect() const {
        // this has type const Widget*
    }
};
```

In `modify`, the type of `this` is `Widget*`, whereas in `inspect`, it is `const Widget*`.

The important limitation is that `this` is not an ordinary parameter, and programmers cannot directly declare its type.

### **Limitations of the Traditional Model**

Suppose we want to design a Getter that returns an appropriate reference to the current object.

In a professional API, we may want the function to behave differently depending on the object:

- For a mutable Lvalue object, return a regular reference.
- For a `const` Lvalue object, return a `const` reference.
- For an Rvalue object, return an Rvalue Reference.

Before C++23, this behavior typically required multiple overloads:

```
class Widget {
public:
    Widget& get() & {
        return *this;
    }

    const Widget& get() const & {
        return *this;
    }

    Widget&& get() && {
        return std::move(*this);
    }
};
```

This approach is perfectly valid and remains appropriate for many APIs. However, as the number of qualifiers and function variations increases, the amount of duplicated code can grow significantly.

Deducing `this` allows us to implement the same behavior with a single function.

## **Introducing Explicit Object Parameters**

### **The Basic Syntax**

In C++23, the first parameter of a Non-static Member Function can be explicitly declared using the `this` keyword.

This parameter is known as an **Explicit Object Parameter**.

Its general syntax looks like this:

```
class Widget {
public:
    void update(this Widget& self, int value) {
        // Use self to access the current object
    }
};
```

In this example:

- `this Widget&` declares the first parameter as the Object Parameter.
- `self` is the name chosen by the programmer to access the current object.
- `Widget&` specifies that the function accepts only an Lvalue of type `Widget`.

The function can still be called using the usual Member Function syntax:

```
Widget widget;
widget.update(42);
```

During this call, the compiler binds the `widget` object to the `self` parameter.

The function can also be called explicitly through the class name, providing the object as an argument:

```
Widget::update(widget, 42);
```

This calling form is useful when understanding how the Object Parameter is passed to the function.

### **Implicit vs. Explicit Object Parameters**

The main difference between these two models is how the current object is declared and accessed.

| Feature                                | Implicit Object Parameter          | Explicit Object Parameter                                |
| -------------------------------------- | ---------------------------------- | -------------------------------------------------------- |
| Direct parameter declaration           | No                                 | Yes                                                      |
| Access to the current object           | Through `this`                     | Through a named parameter                                |
| Object Parameter type                  | Determined by the language         | Specified by the programmer or deduced through templates |
| Template Deduction for the object type | Limited to conventional mechanisms | Supported                                                |
| Usage in Lambdas                       | Through implicit capture or `this` | Through an Explicit Object Parameter                     |
| Reference Category control             | Through Ref-qualified overloads    | Through the parameter type or deduction                  |

### **Explicit Object Member Functions vs. Static Member Functions**

An important distinction is that an Explicit Object Member Function is not the same as a Static Member Function.

A Static Member Function has no Object Parameter and does not implicitly or explicitly depend on a current object.

An Explicit Object Member Function, however, is still a Non-static Member Function. Its first parameter receives the current object.

For example:

```
class Widget {
public:
    void regular(this Widget& self) {
        self.value = 10;
    }

    static void staticFunction() {
    }

private:
    int value = 0;
};
```

In this example, `regular` is an Explicit Object Member Function, whereas `staticFunction` is a Static Member Function.

## **Understanding Deducing this**

### **Using auto for Object Type Deduction**

The most important capability of this feature is the ability to use `auto` in an Explicit Object Parameter.

For example:

```
class Widget {
public:
    void inspect(this auto&& self) {
        // The type of self is deduced
    }
};
```

In this example, the compiler deduces the type of `self` based on the type and Value Category of the object on which the function is called.

This allows a single function to work with different object types without requiring separate overloads.

Consider the following example:

```
#include <iostream>
#include <type_traits>

class Widget {
public:
    void inspect(this auto&& self) {
        using SelfType = decltype(self);

        std::cout << std::boolalpha;
        std::cout << std::is_lvalue_reference_v<SelfType> << '\n';
        std::cout << std::is_const_v<
            std::remove_reference_t<SelfType>
        > << '\n';
    }
};

int main() {
    Widget widget;
    const Widget constWidget;

    widget.inspect();
    constWidget.inspect();
    Widget{}.inspect();
}
```

In this example, the type of `self` differs depending on the call:

| Object                           | Deduced type of `self` |
| -------------------------------- | ---------------------- |
| Lvalue of type `Widget`          | `Widget&`              |
| Lvalue of type `const Widget`    | `const Widget&`        |
| Rvalue of type `Widget`          | `Widget&&`             |
| Lvalue of type `volatile Widget` | `volatile Widget&`     |

The important detail is that `auto&&` follows the rules of Forwarding References in this context.

Consequently, the original object's qualifiers and Reference Category can be preserved.

### **auto&& vs. Widget&&**

Using `auto&&` is fundamentally different from declaring an ordinary Rvalue Reference.

For example:

```
class Widget {
public:
    void process(this Widget&& self) {
    }

    void inspect(this auto&& self) {
    }
};
```

The `process` function accepts only Rvalues of type `Widget`.

However, `inspect` can accept Lvalues and Rvalues, including `const` objects.

For this reason, `auto&&` is particularly useful in Generic Programming.

## **Understanding Qualifiers and Value Categories**

### **Handling const Qualification**

One important advantage of Deducing `this` is the ability to work with the object's type without writing separate overloads.

For example:

```
#include <iostream>
#include <type_traits>

class Widget {
public:
    void inspect(this auto&& self) {
        using ObjectType = std::remove_reference_t<
            decltype(self)
        >;

        if constexpr (std::is_const_v<ObjectType>) {
            std::cout << "Const object\n";
        } else {
            std::cout << "Mutable object\n";
        }
    }
};

int main() {
    Widget widget;
    const Widget constWidget;

    widget.inspect();
    constWidget.inspect();
}
```

In this implementation, `if constexpr` selects the function's behavior based on the object's const qualification.

This technique is useful when a shared algorithm must support both mutable and const versions of a class.

However, if the only requirement is to prevent modification of the object, a `const` function or conventional overloads may provide a simpler and more readable design.

### **Handling Lvalues and Rvalues**

Value Categories play an important role in modern API design.

With `auto&&`, we can determine whether the current object is an Lvalue or an Rvalue.

For example:

```
#include <iostream>
#include <type_traits>

class Widget {
public:
    void inspect(this auto&& self) {
        if constexpr (
            std::is_lvalue_reference_v<decltype(self)>
        ) {
            std::cout << "Lvalue object\n";
        } else {
            std::cout << "Rvalue object\n";
        }
    }
};

int main() {
    Widget widget;

    widget.inspect();
    Widget{}.inspect();
}
```

This capability is particularly useful when designing APIs that must distinguish between using existing resources and transferring resources.

### **Using decltype(auto)**

Alongside Deducing `this`, we can use `decltype(auto)` to preserve the exact return type.

For example:

```
class Widget {
public:
    int value = 42;

    decltype(auto) getValue(this auto&& self) {
        return (self.value);
    }
};
```

The parentheses around `self.value` are important in this example.

The rules of `decltype` determine the return type based on the Value Category of the expression.

Consequently:

- For an Lvalue, the result is an Lvalue Reference.
- For an Rvalue, the result is an Rvalue Reference.

The return type of this function can therefore be:

| Call                | Return type  |
| ------------------- | ------------ |
| On a regular Lvalue | `int&`       |
| On a Const Lvalue   | `const int&` |
| On a regular Rvalue | `int&&`      |

An important consideration is that returning a Reference to a member of a temporary object can be dangerous.

For example:

```
auto&& value = Widget{}.getValue();
```

After the full-expression ends, the temporary object is destroyed, leaving `value` as a dangling reference.

Therefore, preserving Value Categories does not automatically guarantee Lifetime safety.

## **Important Applications of Deducing this**

### **Reducing the Number of Overloads**

One of the primary applications of this feature is eliminating repetitive overloads.

Suppose a class needs a Getter that returns an appropriate Reference to the current object.

A traditional implementation looks like this:

```
class Widget {
public:
    Widget& get() & {
        return *this;
    }

    const Widget& get() const & {
        return *this;
    }

    Widget&& get() && {
        return std::move(*this);
    }

    const Widget&& get() const && {
        return std::move(*this);
    }
};
```

With Deducing `this`, these four overloads can be replaced with a single function:

```
class Widget {
public:
    auto&& get(this auto&& self) {
        return std::forward<decltype(self)>(self);
    }
};
```

In this design, `std::forward` preserves the object's type and Value Category.

This approach is particularly useful for classes with many Member Functions that would otherwise require separate `const` and Ref-qualified overloads.

### **Designing Fluent Interfaces**

In Fluent Interface design, we often want to chain multiple function calls.

One challenge with traditional implementations is the need for multiple overloads to preserve the object's Reference Category.

For example:

```
#include <string>
#include <utility>

class Builder {
public:
    Builder& setName(const char* name) & {
        name_ = name;
        return *this;
    }

    Builder&& setName(const char* name) && {
        name_ = name;
        return std::move(*this);
    }

private:
    std::string name_;
};
```

In C++23, this design can be simplified:

```
#include <string>
#include <utility>

class Builder {
public:
    auto&& setName(
        this auto&& self,
        std::string name
    ) {
        self.name_ = std::move(name);

        return std::forward<decltype(self)>(self);
    }

    auto&& setAge(
        this auto&& self,
        int age
    ) {
        self.age_ = age;

        return std::forward<decltype(self)>(self);
    }

private:
    std::string name_;
    int age_ = 0;
};
```

Both of the following forms are now supported:

```
int main() {
    Builder builder;

    builder.setName("Alice")
           .setAge(30);

    auto result = Builder{}
        .setName("Bob")
        .setAge(25);
}
```

In this implementation, the object's Reference Category is preserved throughout the call chain.

An important consideration is that when called on an Rvalue, the object remains temporary. Any Reference obtained from it must not be used after the object's Lifetime ends.

### **Implementing CRTP**

The `Curiously Recurring Template Pattern`, or CRTP, is an important Template Programming technique in C++.

In this pattern, a derived class is passed as a Template Argument to its base class.

For example:

```
template <typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived&>(*this).implementation();
    }
};

class Widget : public Base<Widget> {
public:
    void implementation() {
    }
};
```

One limitation of the traditional design is that the base class must use the `Derived` type to access the derived object.

Deducing `this` provides a similar implementation without requiring a class Template Parameter:

```
class Base {
public:
    void interface(this auto&& self) {
        self.implementation();
    }
};

class Widget : public Base {
public:
    void implementation() {
    }
};
```

In this example, the actual object type is deduced through the Explicit Object Parameter.

This allows the base class to access the derived class's API without having a Template Parameter.

This approach is sometimes described as a **CRTP-less Design**, or a design similar to CRTP without templates.

However, these two patterns are not completely equivalent.

With CRTP, the base class can use the derived type in various parts of its class definition. With Deducing `this`, the derived type is deduced when the function is used.

Therefore, Deducing `this` is not a universal replacement for every CRTP use case.

### **Implementing Generic Interfaces**

Another important application is designing interfaces whose behavior depends on the actual type of the object.

For example:

```
#include <iostream>

class Printable {
public:
    void print(this const auto& self) {
        self.printImpl();
    }
};

class User : public Printable {
public:
    void printImpl() const {
        std::cout << "User\n";
    }
};

class Product : public Printable {
public:
    void printImpl() const {
        std::cout << "Product\n";
    }
};
```

In this example, the `print` function in the base class is instantiated separately for each derived type.

Consequently, `self` has the actual object type and can invoke the appropriate function.

This approach provides a form of Static Polymorphism.

Note that it does not rely on Virtual Functions; dispatch is performed at compile time.

## **Using Deducing this in Lambdas**

### **The Problem with Traditional Recursive Lambdas**

One particularly interesting application of Deducing `this` is simplifying Recursive Lambdas.

In older versions of C++, writing a recursive Lambda typically required an intermediate variable or the use of `std::function`.

For example:

```
#include <functional>

int main() {
    std::function<int(int)> factorial;

    factorial = [&](int n) -> int {
        if (n <= 1) {
            return 1;
        }

        return n * factorial(n - 1);
    };

    return factorial(5);
}
```

This approach works, but using `std::function` can introduce additional overhead.

Potential costs include Type Erasure and possible Dynamic Allocation.

### **Using Explicit Object Parameters in Lambdas**

Starting with C++23, a Recursive Lambda can be defined directly using an Explicit Object Parameter.

For example:

```
#include <iostream>

int main() {
    auto factorial = [](this auto&& self, int n) -> int {
        if (n <= 1) {
            return 1;
        }

        return n * self(n - 1);
    };

    std::cout << factorial(5) << '\n';
}
```

In this example:

- The `self` parameter refers to the current Closure Object.
- The Lambda can recursively invoke itself through `self`.
- No `std::function` is required.
- The closure type is known at compile time.
- The compiler can potentially inline recursive calls.

This approach is generally simpler than using `std::function` and more efficient from a type-system perspective.

### **Recursive Lambda for Tree Traversal**

One practical application of Recursive Lambdas is traversing tree structures.

```
#include <iostream>
#include <memory>

struct Node {
    int value;

    std::unique_ptr<Node> left;
    std::unique_ptr<Node> right;
};

int main() {
    auto root = std::make_unique<Node>(
        Node{1, nullptr, nullptr}
    );

    root->left = std::make_unique<Node>(
        Node{2, nullptr, nullptr}
    );

    root->right = std::make_unique<Node>(
        Node{3, nullptr, nullptr}
    );

    auto traverse = [](this auto&& self, const Node* node) -> void {
        if (node == nullptr) {
            return;
        }

        std::cout << node->value << '\n';

        self(node->left.get());
        self(node->right.get());
    };

    traverse(root.get());
}
```

In this example, the Lambda recursively traverses a binary tree.

The important point is that recursion requires neither changing the variable's type nor introducing additional wrappers.

### **Using Stateful Lambdas**

Explicit Object Parameters can also be used with Lambdas that have captures.

For example:

```
#include <iostream>

int main() {
    int count = 0;

    auto counter = [&count](this auto&& self) {
        ++count;

        if (count < 5) {
            self();
        }
    };

    counter();

    std::cout << count << '\n';
}
```

In this example, the Lambda recursively invokes itself through `self` and modifies its captured state.

Note that an Explicit Object Parameter does not replace captures.

It simply provides explicit access to the Closure Object.

## **Important Differences from Existing Features**

### **CRTP vs. Deducing this**

Both approaches can be used to implement Static Polymorphism, but they have important differences.

| Feature                                                       | CRTP                        | Deducing this     |
| ------------------------------------------------------------- | --------------------------- | ----------------- |
| Requires a Template Parameter in the base class               | Yes                         | No                |
| Requires `static_cast` to access Derived                      | Usually                     | Usually not       |
| Access to the derived type                                    | Through a Template Argument | Through deduction |
| Can use the derived type throughout the base class definition | Yes                         | More limited      |
| Complexity of the base class definition                       | Higher                      | Lower             |
| Suitable for Generic Base Classes                             | Yes                         | In many cases     |

An important consideration is that Deducing `this` is not a complete replacement for CRTP.

For example, if a base class needs to use the derived type in Data Member declarations, Base Class declarations, or other Template declarations, CRTP remains an appropriate choice.

### **std::forward vs. Deducing this**

Deducing `this` is sometimes confused with `std::forward`.

These two features serve different purposes.

Deducing `this` obtains or deduces the type of the Object Parameter.

In contrast, `std::forward` is used to preserve Value Category when forwarding a Reference.

For example:

```
#include <utility>

class Widget {
public:
    void process(this auto&& self) {
        helper(std::forward<decltype(self)>(self));
    }

private:
    template <typename T>
    void helper(T&& value) {
    }
};
```

In this example, `self` provides the object's type, while `std::forward` passes it to another function with the same Reference Category.

Using `std::forward` is important when we need to preserve Forwarding Reference semantics.

### **std::move vs. std::forward**

The `std::move` function always produces an Rvalue Reference, but this does not mean that resources are automatically transferred.

In contrast, `std::forward` preserves the original type and Value Category.

Therefore, when implementing a function that must support both Lvalues and Rvalues, `std::forward` is generally the more appropriate choice.

For example:

```
#include <utility>

class Widget {
public:
    auto&& get(this auto&& self) {
        return std::forward<decltype(self)>(self);
    }
};
```

In this example, using `std::move` would cause even Lvalues to be returned as Rvalues.

## **Advanced Language Details**

### **Restrictions on Explicit Object Parameters**

Despite its flexibility, Explicit Object Parameters have specific language restrictions.

The most important restrictions include:

- An Explicit Object Parameter must be the first parameter of a Member Function.
- A function cannot have both an Explicit Object Parameter and an Implicit Object Parameter.
- A Member Function cannot have more than one Explicit Object Parameter.
- An Explicit Object Member Function cannot be `static`.
- An Explicit Object Member Function cannot be `virtual`.
- Conventional CV-Qualifiers and Ref-Qualifiers cannot be specified after the declarator of such a function.
- An Explicit Object Parameter cannot have a Default Argument.
- Explicit Object Parameters cannot be used for Constructors or Destructors.
- They cannot be used for Conversion Functions.

For example, the following declaration is invalid:

```
class Widget {
public:
    void process(this Widget& self) const;
};
```

This is because conventional Member Function CV-Qualifiers cannot be used in this form.

Instead, const qualification must be specified in the Explicit Object Parameter type:

```
class Widget {
public:
    void process(this const Widget& self);
};
```

Alternatively, `auto&&` can be used to support both mutable and const objects.

### **Virtual Function Restrictions**

An Explicit Object Member Function cannot be virtual.

For example, the following declaration is invalid:

```
class Base {
public:
    virtual void execute(this Base& self);
};
```

This restriction exists because Explicit Object Parameters follow a different model from conventional Virtual Dispatch.

For Dynamic Polymorphism, conventional Virtual Member Functions must still be used.

For example:

```
class Base {
public:
    virtual void execute() = 0;

    virtual ~Base() = default;
};
```

Therefore, Deducing `this` should not be considered a general replacement for Virtual Functions.

### **Differences in Function Pointer Types**

One important detail of this feature is the difference between the Pointer Types of conventional and Explicit Object Member Functions.

For conventional Member Functions, the address of the function has a Pointer-to-Member-Function type.

However, an Explicit Object Member Function has an ordinary function pointer type.

For example:

```
class Widget {
public:
    void regular(int value) {
    }

    void explicitObject(this Widget& self, int value) {
    }
};

int main() {
    void (Widget::*memberPointer)(int) = &Widget::regular;

    void (*functionPointer)(Widget&, int) =
        &Widget::explicitObject;
}
```

This distinction is important when working with Callbacks, Function Pointers, and APIs that use Member Function Pointers.

### **Calling Explicit Object Member Functions**

A function with an Explicit Object Parameter is still a Member Function and can be called using the conventional Member Function syntax.

For example:

```
class Widget {
public:
    void execute(this Widget& self, int value) {
    }
};

int main() {
    Widget widget;

    widget.execute(10);

    Widget::execute(widget, 20);
}
```

In the first form, the object is passed through the Member Function Call Syntax.

In the second form, the Object Argument is provided explicitly.

The important point is that the first parameter represents the same current object in both cases.

### **Accessing Class Members**

Inside an Explicit Object Member Function, the named parameter must be used to access the object's members.

For example:

```
class Widget {
public:
    void update(this Widget& self) {
        self.value = 100;
    }

private:
    int value = 0;
};
```

In this model, `self.value` is the correct way to access the object's member.

An Explicit Object Member Function cannot use `this` to obtain an Implicit Object Pointer in the same way as a conventional Member Function.

This is an important conceptual difference between the two models.

### **Using Explicit Template Parameters**

In addition to `auto`, an Explicit Object Parameter can be declared using a Template Parameter.

For example:

```
#include <type_traits>

class Widget {
public:
    template <typename Self>
    void inspect(this Self&& self) {
        static_assert(
            std::is_same_v<
                std::remove_cvref_t<Self>,
                Widget
            >
        );
    }
};
```

In this example, the object's type is deduced through the `Self` Template Parameter.

This approach is useful when we want to use the object type in other parts of the function in addition to receiving it.

For example, `Self` can be used with Constraints, Type Traits, or `if constexpr`.

### **Using Concepts**

Deducing `this` can be combined with Concepts, introduced in C++20.

For example:

```
#include <concepts>

class Widget {
public:
    template <typename Self>
        requires std::same_as<
            std::remove_cvref_t<Self>,
            Widget
        >
    void execute(this Self&& self) {
    }
};
```

In this example, the function is only available for objects whose underlying type is `Widget`.

This approach is useful for constraining Generic APIs and preventing unwanted instantiations.

However, the type restriction in this example is not particularly necessary because the function is already declared inside `Widget`. The same technique becomes more useful when designing general-purpose templates.

## **Important Design and Implementation Considerations**

### **Choosing the Appropriate Explicit Object Parameter Type**

The Explicit Object Parameter type should be selected according to the actual requirements of the API.

Some common patterns include:

| Declaration               | Use Case                                                 |
| ------------------------- | -------------------------------------------------------- |
| `this Widget& self`       | Accepting mutable Lvalues                                |
| `this const Widget& self` | Accepting const and non-const Lvalues and Rvalues        |
| `this Widget&& self`      | Accepting mutable Rvalues                                |
| `this auto& self`         | Deducing the object type while restricting it to Lvalues |
| `this auto&& self`        | Generic support for different Value Categories           |
| `this const auto& self`   | Read-only access to an object with type deduction        |

Using `auto&&` is not always the best decision.

If a function should only operate on Lvalues, `auto&` provides an appropriate restriction.

Similarly, if the function does not need Rvalues, accepting them may expose unintended API behavior.

### **Preserving Lifetime Safety**

One common mistake is returning a Reference to an object whose Lifetime may have ended.

For example:

```
class Widget {
public:
    auto&& get(this auto&& self) {
        return static_cast<decltype(self)&&>(self);
    }
};
```

This function preserves the object's Value Category. However, if it is called on a Temporary, the returned Reference becomes invalid after the Temporary's Lifetime ends.

Therefore, using `auto&&` and `decltype(auto)` requires a thorough understanding of Lifetime rules.

For APIs where returning a Reference to a Temporary should be prohibited, a more restrictive design can be used.

For example:

```
class Widget {
public:
    Widget& get() & {
        return *this;
    }

    const Widget& get() const & {
        return *this;
    }

    Widget&& get() && = delete;
};
```

With this design, calls on Rvalues are explicitly prohibited.

### **Avoiding Over-Generalization**

One advantage of Generic Programming is reducing code duplication. However, excessive use of generic mechanisms can reduce API readability.

For example, if a function only needs to read a value from an object, the following implementation may be sufficient:

```
class Widget {
public:
    int getValue() const {
        return value_;
    }

private:
    int value_ = 42;
};
```

In this case, using an Explicit Object Parameter does not provide a clear advantage.

Deducing `this` should be used when there is a genuine need for behavior that depends on the object's exact type, const qualification, or Value Category.

### **Understanding Access Control**

An Explicit Object Member Function is still a member of its class.

Therefore, the usual Access Control rules for accessing private and protected members continue to apply.

For example:

```
class Widget {
public:
    void update(this auto& self) {
        self.value_ = 10;
    }

private:
    int value_ = 0;
};
```

In this example, the function can access the class's private member.

Type deduction for `self` does not change Encapsulation or Access Control rules.

### **Understanding Template Instantiation**

When `auto` or a Template Parameter is used in an Explicit Object Parameter, the function can be instantiated for different object types.

As a result, multiple function specializations may be generated at compile time.

This is particularly important when the function body is complex or invokes other templates.

From a performance perspective, this model generally provides good opportunities for optimization and inlining. However, increasing the number of instantiations can increase Binary Size and Compilation Time.

## **Common Mistakes**

### **Assuming Deducing this Creates a Static Function**

Although an Explicit Object Member Function has an explicit Object Parameter, it is still a Non-static Member Function.

The difference concerns how the Object Parameter is declared, not whether the function belongs to the class.

### **Incorrectly Using this Inside the Function Body**

Inside an Explicit Object Member Function, the named parameter must be used to access the current object.

For example:

```
class Widget {
public:
    void update(this Widget& self) {
        self.value = 10;
    }

private:
    int value = 0;
};
```

This function should not be designed like a conventional Member Function that uses `this` to access the current object.

### **Forgetting std::forward**

When a parameter has type `auto&&`, using it directly in an expression generally treats it as an Lvalue.

To preserve its Value Category when forwarding it, `std::forward` must be used.

For example:

```
#include <utility>

class Widget {
public:
    void process(this auto&& self) {
        helper(std::forward<decltype(self)>(self));
    }

private:
    template <typename T>
    void helper(T&& value) {
    }
};
```

### **Assuming auto&& Is Always an Rvalue Reference**

Forwarding Reference rules also apply to Explicit Object Parameters.

Therefore, `auto&&` can become either an Lvalue Reference or an Rvalue Reference depending on the type of the object.

### **Ignoring Const Qualification**

If a function is defined as follows:

```
class Widget {
public:
    void modify(this auto&& self) {
        self.value = 10;
    }

private:
    int value = 0;
};
```

This function cannot be used with a `const Widget` object because `self` becomes a Const Reference in that case, and modifying non-`mutable` members is not allowed.

### **Confusing Explicit Object Parameters with Ordinary Parameters**

An Explicit Object Parameter differs from ordinary parameters in its position and language rules.

It must be the first parameter of the function and is responsible for receiving the current object.

Therefore, it cannot be placed arbitrarily within the Parameter List.

## **Comparing Different Implementation Approaches**

### **Traditional Getter vs. Deducing this**

To better understand the benefits of this feature, consider the following two implementations.

Traditional approach:

```
class Widget {
public:
    Widget& get() & {
        return *this;
    }

    const Widget& get() const & {
        return *this;
    }

    Widget&& get() && {
        return std::move(*this);
    }

    const Widget&& get() const && {
        return std::move(*this);
    }
};
```

Deducing `this` approach:

```
#include <utility>

class Widget {
public:
    auto&& get(this auto&& self) {
        return std::forward<decltype(self)>(self);
    }
};
```

Both approaches can provide equivalent behavior.

However, the second approach requires less code and is easier to maintain in larger classes.

The traditional approach is still entirely appropriate for APIs that require explicit overloads and clearly defined restrictions.

### **Recursive Lambda vs. std::function**

Traditional approach:

```
#include <functional>

std::function<int(int)> factorial;

factorial = [&](int n) {
    if (n <= 1) {
        return 1;
    }

    return n * factorial(n - 1);
};
```

Modern approach:

```
auto factorial = [](this auto&& self, int n) -> int {
    if (n <= 1) {
        return 1;
    }

    return n * self(n - 1);
};
```

The modern approach does not require `std::function`, and its type is known at compile time.

However, `std::function` remains useful when Type Erasure or storing callbacks with different types is required.

Therefore, Recursive Lambdas using Deducing `this` are not a replacement for every use case of `std::function`.

## **Compiler Support and Standardization**

### **Standardization Status**

Deducing `this` was added to the C++23 standard through proposal `P0847R7`.

This feature is part of the Core Language and does not require a dedicated Standard Library header.

However, related examples may require headers such as:

- `<utility>` for `std::forward` and `std::move`.
- `<type_traits>` for Type Traits.
- `<concepts>` for Concepts.
- `<functional>` for `std::function`.
- `<iostream>` for input and output operations.

### **Checking Compiler Support**

C++23 support can vary between compilers.

To use this feature, you need a compiler version that implements Explicit Object Parameters.

Example compilation commands:

```
g++ -std=c++23 main.cpp -o app
```

```
clang++ -std=c++23 main.cpp -o app
```

```
cl /std:c++latest /EHsc main.cpp
```

In Visual C++, the `c++latest` option enables the latest supported language features.

An important consideration is that enabling C++23 mode or `c++latest` does not guarantee that every feature of the standard has been implemented in the installed compiler version.

For accurate information, check the documentation for the specific compiler version you are using.

Also, distinguish between language support and IDE support. A compiler may support a feature even if the IDE's Code Analysis or Syntax Highlighting has not yet been updated.

## **Practical Applications in Professional Projects**

### **Designing General-Purpose Libraries**

When developing libraries intended for use by different developers, reducing the number of overloads can make APIs easier to maintain and extend.

For example, a Container or Wrapper can use Deducing `this` to implement shared operations without defining multiple versions.

### **Designing Fluent APIs and the Builder Pattern**

In the Builder Pattern, preserving the object's type and Value Category across chained functions is important.

Deducing `this` can reduce the number of overloads required for this purpose.

### **Developing Generic Programming Interfaces**

In Generic API design, the exact object type and its qualifiers may influence which behavior should be selected.

Deducing `this` provides direct access to this information.

### **Implementing Recursive Algorithms**

Recursive Lambdas using Deducing `this` can provide simpler implementations for algorithms such as:

- Tree traversal.
- Depth-First Search.
- Graph traversal and search.
- Recursive calculations.
- Processing nested structures.
- Evaluating Expression Trees.

### **Implementing Static Polymorphism**

In some designs, Deducing `this` can eliminate the need for CRTP.

This can reduce the complexity of base class structures, particularly when the only goal is to access the derived class's API through Member Functions.

## **Summary**

**Deducing&#x20;**`this` is an important C++23 feature that provides more direct control over how Member Functions receive their Object Parameter.

By introducing Explicit Object Parameters, it reduces the limitations of the traditional Implicit Object Parameter model and allows programmers to explicitly specify or deduce the type of the current object.

The most important benefits include:

- Reducing repetitive overloads for `const` and non-`const` Member Functions.
- Simplifying Lvalue and Rvalue handling.
- Preserving Reference Categories through `std::forward`.
- Simplifying Fluent Interfaces and the Builder Pattern.
- Reducing the complexity of some CRTP implementations.
- Enabling Recursive Lambdas without requiring `std::function`.
- Increasing flexibility in Generic Programming.
- Supporting Concepts and Type Traits for constraining Generic Member Functions.

However, this feature also has limitations.

Explicit Object Member Functions cannot be Virtual or Static, and their use is prohibited in Constructors, Destructors, and certain other special functions.

Additionally, incorrect use of `auto&&`, `std::forward`, and returned References can introduce unnecessary API complexity or Dangling References.

Ultimately, Deducing `this` is a powerful tool for simplifying Generic designs and reducing code duplication. However, its use should be driven by the actual requirements of the API.

For simple Member Functions, the traditional approach remains appropriate. In contrast, whenever preserving the exact type, const qualification, and Value Category of an object is important, Deducing `this` can provide a more readable, flexible, and modern solution.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>