<div align="center">

[🇺🇸 English](./static_operator().md) | [🇮🇷 فارسی](../../fa/cpp23/static_operator().md)

</div>

---

# **Static&#x20;**`operator()`**&#x20;in C++23**

## **Introduction**

One of the features introduced in the C++23 standard is the ability to define `operator()` as a `static` member function. This feature, known as the **Static Call Operator**, allows developers to define a Function Call Operator without requiring an implicit object parameter.

In previous versions of C++, `operator()` was always a non-static member function. Consequently, invoking it required an object of the class, and the function implicitly had access to the current object through `this`.

C++23 removes this restriction and allows `operator()` to be declared as `static`.

```
#include <iostream>

struct Calculator {
    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    Calculator calc;

    std::cout << calc(10, 20) << '\n';
    std::cout << Calculator{}(5, 7) << '\n';
}
```

In this example, the `operator()` function has no dependency on instance state. It simply receives two arguments and calculates their sum.

This feature is particularly useful when designing stateless function objects, generic callable objects, and lambda expressions, as well as when reducing unnecessary dependencies on object instances.

## **Understanding the Static Call Operator**

### **Defining operator() in C++**

The `operator()` function is one of the features of operator overloading in C++ that allows an object to be invoked like an ordinary function.

For example:

```
#include <iostream>

struct Printer {
    void operator()(int value) const {
        std::cout << value << '\n';
    }
};

int main() {
    Printer printer;

    printer(42);
}
```

In this example, the following expression:

```
printer(42);
```

is conceptually equivalent to:

```
printer.operator()(42);
```

The `Printer` class effectively defines a Function Object, commonly called a Functor.

Under the traditional model, `operator()` is a non-static member function. Therefore, whenever it is invoked, the associated object is passed implicitly as the implicit object parameter.

### **Changes Introduced in C++23**

The C++23 standard removes this restriction by allowing `operator()` to be declared as a static member function.

```
struct Utility {
    static void operator()(int value) {
        // Process the value
    }
};
```

The main difference is that the function is no longer associated with a particular object instance.

Consequently:

- The function has no implicit `this` pointer.
- The function cannot directly access non-static members of the class.
- The function does not depend on instance state for its execution.
- The function can be invoked through either an object or the class scope.
- The function cannot use member-function-specific cv-qualifiers or ref-qualifiers.

## **Why Was This Feature Introduced?**

### **The Problem with Stateless Functors**

In many C++ designs, Functors are used to implement callable behavior.

For example:

```
struct Square {
    int operator()(int value) const {
        return value * value;
    }
};
```

The `Square` class has no state. Its only responsibility is to receive an input and calculate its square.

Despite this, its `operator()` is still a non-static member function.

Consequently, an object must be created before the function can be invoked:

```
Square square;

int result = square(5);
```

In this situation, the implicit object parameter serves no meaningful purpose because the function's behavior does not depend on any instance members.

The static version addresses this issue:

```
struct Square {
    static int operator()(int value) {
        return value * value;
    }
};

int main() {
    int result = Square{}(5);
}
```

### **Improving Stateless Function Object Design**

One of the main goals of this feature is to allow developers to express directly at the language level that a Function Object can operate without depending on an instance.

This improves the design of classes whose sole purpose is to provide a callable interface.

It can also eliminate unnecessary dependencies on object instances in certain scenarios.

However, it is important to understand that declaring an operator as `static` does not automatically improve performance. Compilers can generally optimize stateless traditional Functors as well.

## **Defining and Using Static operator()**

### **General Syntax**

To define a Static Call Operator, place the `static` keyword before `operator()`.

```
struct MyFunction {
    static ReturnType operator()(Parameters...);
};
```

A practical example:

```
struct Multiplier {
    static int operator()(int a, int b) {
        return a * b;
    }
};
```

In this declaration:

- `static` specifies that the operator is a static member function.
- `operator()` identifies the Function Call Operator.
- The parameters represent the arguments passed to the call.
- The return type can be any valid type supported by the language.

### **Calling Through an Object**

As with traditional Function Objects, a Static Call Operator can be invoked through an object.

```
#include <iostream>

struct Adder {
    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    Adder adder;

    std::cout << adder(10, 20) << '\n';
}
```

The important point is that the object's presence in this syntax is only part of the function-call expression. The operator itself does not depend on the instance for its execution.

### **Calling Through Class Scope**

Since the operator is static, it can also be invoked through the class name:

```
#include <iostream>

struct Adder {
    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    int result = Adder::operator()(10, 20);

    std::cout << result << '\n';
}
```

This approach is useful when you want to explicitly communicate that the function is static and does not depend on an instance.

Note that `Adder::operator()` is an ordinary static member function call. It should not be confused with the usual Functor invocation syntax.

## **Static vs. Non-Static operator()**

### **Comparing Their Behavior**

The most important difference between these two operators is whether an implicit object parameter exists.

With a non-static operator:

```
struct NormalFunctor {
    int factor = 10;

    int operator()(int value) const {
        return value * factor;
    }
};
```

The operator depends on the current object and can directly access its non-static members.

With a static operator:

```
struct StaticFunctor {
    static int operator()(int value) {
        return value * 10;
    }
};
```

The operator does not depend on an instance and cannot directly access `this` or non-static members.

### **Feature Comparison**

| Feature                             | Non-Static `operator()`   | Static `operator()` |
| ----------------------------------- | ------------------------- | ------------------- |
| Implicit object parameter           | Yes                       | No                  |
| Direct access to `this`             | Yes                       | No                  |
| Direct access to non-static members | Yes                       | No                  |
| Access to static members            | Yes                       | Yes                 |
| cv-qualifiers                       | Supported                 | Not supported       |
| ref-qualifiers                      | Supported                 | Not supported       |
| Virtual function support            | Supported where permitted | Not supported       |
| Invocation through an object        | Yes                       | Yes                 |
| Invocation through class scope      | No                        | Yes                 |
| Requires an object for its behavior | Yes                       | No                  |

### **Design Considerations**

When an operator's behavior depends on the internal state of an object, a non-static `operator()` is the appropriate choice.

However, when the operator performs an independent operation and does not need instance state, the static version can express the design more clearly.

For example:

```
struct StatefulMultiplier {
    int factor;

    explicit StatefulMultiplier(int value)
        : factor(value) {}

    int operator()(int value) const {
        return value * factor;
    }
};

struct StatelessMultiplier {
    static int operator()(int value) {
        return value * 10;
    }
};
```

The first class depends on internal state to perform calculations, whereas the second provides behavior that is independent of any instance.

## **Static operator() vs. Static Member Functions**

Both features use the `static` keyword, but a Static Call Operator differs from an ordinary static member function in syntax and intended usage.

### **Ordinary Static Member Function**

```
struct Math {
    static int square(int value) {
        return value * value;
    }
};
```

The function is invoked as follows:

```
int result = Math::square(5);
```

### **Static Call Operator**

```
struct MathFunctor {
    static int operator()(int value) {
        return value * value;
    }
};
```

It can be invoked in two ways:

```
int first = MathFunctor{}(5);

int second = MathFunctor::operator()(5);
```

The important distinction is that `operator()` allows a class to behave like a Callable Object.

Therefore, a Static Call Operator can be a more suitable alternative to explicitly named functions in APIs that use Callable Objects.

## **Relationship with Lambda Expressions**

One of the most important related applications of Static Call Operator in C++23 is the Static Lambda feature.

In previous versions of C++, a lambda expression without captures typically produces a Closure Object containing an `operator()`.

For example:

```
auto square = [](int value) {
    return value * value;
};
```

In this case, the compiler generates a Closure Type with an appropriate call operator.

C++23 allows the `static` keyword to be added to the Lambda Declarator.

### **Introducing Static Lambdas**

```
auto square = [](int value) static {
    return value * value;
};

int result = square(5);
```

In this example, the `operator()` of the Closure Type is declared as static.

This feature is useful for lambdas that have no captures and whose behavior is entirely independent of the Closure Object.

### **Capture Restrictions in Static Lambdas**

A Static Lambda cannot use captures to store external state.

For example, the following definition is invalid:

```
int factor = 10;

auto multiply = [factor](int value) static {
    return value * factor;
};
```

The reason is that a Static Call Operator has no implicit object parameter and therefore cannot access state stored in the Closure Object.

For this scenario, use an ordinary lambda instead:

```
int factor = 10;

auto multiply = [factor](int value) {
    return value * factor;
};
```

### **Captureless Static Lambdas**

The following is a valid Static Lambda:

```
auto add = [](int a, int b) static {
    return a + b;
};

int result = add(10, 20);
```

Static Lambdas can also be used to define Generic Lambdas.

## **Using Static with Generic Lambdas**

One useful application of this feature is combining Static Lambdas with template parameters.

### **Defining a Generic Static Lambda**

```
#include <concepts>

auto add = []<typename T>(T a, T b) static
    requires std::integral<T>
{
    return a + b;
};

int main() {
    auto result1 = add(10, 20);
    auto result2 = add(30L, 40L);
}
```

In this example:

- The lambda uses a template parameter.
- The Closure Type's call operator is declared as static.
- The `requires` clause restricts the accepted types to integral types.

This approach is suitable for implementing generic, stateless operations.

### **Combining Static Lambdas with Concepts**

Concepts can make the input requirements of a Generic Static Lambda more explicit.

```
#include <concepts>
#include <type_traits>

auto equal = []<typename T>(const T& a, const T& b) static
    requires std::equality_comparable<T>
{
    return a == b;
};

int main() {
    static_assert(equal(10, 10));
    static_assert(!equal(10, 20));
}
```

In this example, the Generic Static Lambda can only be used with types that satisfy the `std::equality_comparable` concept.

### **Important Considerations for Generic Static Lambdas**

Declaring the operator as static does not prevent the use of templates.

The operator can still:

- Be a function template.
- Use template parameters.
- Use parameter packs.
- Use Concepts and Constraints.
- Have a return type dependent on template parameters.

Therefore, this feature does not impose restrictions on Generic Programming.

## **Applications in the Standard Library and Callable Objects**

The concept of Callable Objects plays an important role in the C++ standard library.

Many algorithms can accept a function, Function Pointer, Lambda, or Functor.

Static Call Operator can be useful when designing stateless Functors for these scenarios.

### **Using with std::sort**

```
#include <algorithm>
#include <vector>

struct Ascending {
    static bool operator()(int a, int b) {
        return a < b;
    }
};

int main() {
    std::vector<int> values{5, 2, 8, 1, 9};

    std::sort(values.begin(), values.end(), Ascending{});
}
```

In this example, `Ascending` is a stateless Comparator.

Since its operator is static, performing comparisons does not require any instance state.

However, the `std::sort` algorithm still receives a Callable Object. In this example, an empty `Ascending` instance is passed to it.

### **Using with std::for_each**

```
#include <algorithm>
#include <iostream>
#include <vector>

struct Printer {
    static void operator()(int value) {
        std::cout << value << ' ';
    }
};

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    std::for_each(
        values.begin(),
        values.end(),
        Printer{}
    );
}
```

This Functor does not need internal state to process each element.

However, if you need to maintain shared state, such as the sum of all numbers, Static Call Operator alone is not an appropriate solution. You should instead use external state or a non-static Functor.

## **Relationship with Function Pointers**

Another important consideration when designing Static Call Operators is their relationship with Function Pointers.

A static member function can be converted to an ordinary Function Pointer, just like other static functions.

For example:

```
#include <iostream>

struct Calculator {
    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    using FunctionPointer = int(*)(int, int);

    FunctionPointer function =
        &Calculator::operator();

    std::cout << function(10, 20) << '\n';
}
```

In this example, the address of the static operator is assigned to an ordinary Function Pointer.

In contrast, the address of a non-static member function has the distinct type of a Member Function Pointer and must be declared differently.

### **Comparing Pointer Types**

```
struct Normal {
    int operator()(int value) const {
        return value;
    }
};

struct Static {
    static int operator()(int value) {
        return value;
    }
};

using StaticFunctionPointer = int(*)(int);

using MemberFunctionPointer =
    int (Normal::*)(int) const;
```

In this example:

- `StaticFunctionPointer` is a pointer to an ordinary function.
- `MemberFunctionPointer` is a pointer to a member function.

This distinction is important when designing APIs that accept Function Pointers.

### **An Important Function Pointer Limitation**

Declaring the operator as static does not mean that the object itself can be converted into a Function Pointer.

For example:

```
Static object;

int (*function)(int) = &Static::operator();
```

Here, the pointer refers to the static function itself, not to an object of type `Static`.

Therefore, if an API requires a Callable Object, you must consider the distinction between Function Pointers and Functors.

## **Applications in Stateless Policy Design**

In Generic Component design, a Policy Type is sometimes used to specify the behavior of a class.

Static Call Operator can be useful for implementing policies that have no state.

For example:

```
#include <iostream>

struct AdditionPolicy {
    static int apply(int a, int b) {
        return a + b;
    }
};

struct MultiplicationPolicy {
    static int apply(int a, int b) {
        return a * b;
    }
};

template<typename Policy>
struct Calculator {
    int operator()(int a, int b) const {
        return Policy::apply(a, b);
    }
};

int main() {
    Calculator<AdditionPolicy> add;
    Calculator<MultiplicationPolicy> multiply;

    std::cout << add(10, 20) << '\n';
    std::cout << multiply(10, 20) << '\n';
}
```

In this example, a static member function is used to implement the Policy.

The same idea can be implemented using Static Call Operator:

```
struct AdditionPolicy {
    static int operator()(int a, int b) {
        return a + b;
    }
};

template<typename Policy>
struct Calculator {
    int operator()(int a, int b) const {
        return Policy{}(a, b);
    }
};
```

This design is useful when you want Policy Types to also be usable as Callable Objects.

The important point is that Static Call Operator provides a language-level design option here. It does not necessarily offer better performance than an ordinary static function.

## **Applications in Template Metaprogramming**

Static Call Operator can also be used when designing class templates and Generic Programming utilities.

In particular, it is useful when a type is responsible for providing an operation that does not depend on instance state.

### **Defining a Generic Operation**

```
#include <type_traits>

template<typename T>
struct Identity {
    static constexpr T operator()(T value) noexcept {
        return value;
    }
};

int main() {
    constexpr int value = Identity<int>{}(42);

    static_assert(value == 42);
}
```

In this example, Static Call Operator is a `constexpr` function.

As a result, it can be evaluated during Constant Evaluation.

### **Combining with constexpr**

Static Call Operator can be combined with the usual C++ function features, including:

- `constexpr`
- `consteval`
- `noexcept`
- `inline`
- `template`
- `requires`

For example:

```
struct Square {
    static constexpr int operator()(int value) noexcept {
        return value * value;
    }
};

constexpr int result = Square{}(5);

static_assert(result == 25);
```

In this example, the function can be evaluated at compile time.

Note that `static` and `constexpr` are independent properties. The former indicates that the function does not depend on an object, while the latter specifies that the function can participate in Constant Evaluation.

## **Language Rules and Restrictions**

Using Static Call Operator is subject to specific C++ language rules.

Understanding these restrictions is essential for avoiding compilation errors and design problems.

### **No this Pointer**

A static member function does not have an implicit `this` pointer.

Therefore, the following code is invalid:

```
struct Example {
    int value = 10;

    static int operator()() {
        return value;
    }
};
```

This is invalid because `value` is a non-static data member, and accessing it requires a specific object.

If the intended value is constant or shared across all instances, the following approach is valid:

```
struct Example {
    inline static int value = 10;

    static int operator()() {
        return value;
    }
};
```

### **No Direct Access to Non-Static Members**

To access a non-static member, an explicit object must be available.

For example:

```
struct Example {
    int value = 10;

    static int operator()(const Example& object) {
        return object.value;
    }
};
```

This code is valid because the object is explicitly passed as a function parameter.

In other words, a Static Call Operator can operate on an object's data, but it has no implicit access to a current object.

### **No cv-Qualifiers**

Non-static member functions can use `const`:

```
struct Normal {
    int operator()(int value) const {
        return value;
    }
};
```

However, cv-qualifiers for member functions cannot be applied to a Static Call Operator.

Therefore, the following declaration is invalid:

```
struct Invalid {
    static int operator()(int value) const {
        return value;
    }
};
```

The reason for this restriction is that a static function is not associated with a current object, so qualifying that object as const has no meaning.

### **No Ref-Qualifiers**

Ordinary member functions can use `&` and `&&` to constrain their behavior based on the value category of the object.

For example:

```
struct Normal {
    void operator()() & {
    }

    void operator()() && {
    }
};
```

However, Static Call Operator cannot use ref-qualifiers because it has no implicit object parameter.

### **Cannot Be Virtual**

A static member function cannot be virtual.

Consequently, Static Call Operator cannot be declared as virtual either.

For example:

```
struct Invalid {
    virtual static void operator()() {
    }
};
```

This declaration is invalid.

If you need polymorphic dispatch, you must use a non-static virtual `operator()`.

### **Cannot Be Overridden**

Since a static member function cannot be virtual, a Static Call Operator cannot override a virtual operator declared in a base class.

This restriction is important when designing inheritance-based interfaces.

## **Static operator() vs. Explicit Object Parameters**

One of the important features introduced in C++23 is Explicit Object Parameters, also known as Deducing This and Explicit Object Member Functions.

This feature is conceptually related to Static Call Operator, but the two are not equivalent.

### **Understanding Explicit Object Parameters**

Under the traditional model, the current object is passed implicitly to a member function.

Explicit Object Parameters allow the type and properties of that object to be specified explicitly in the parameter list.

For example:

```
struct Example {
    int value = 10;

    int operator()(this const Example& self) {
        return self.value;
    }
};
```

In this example, `self` is an Explicit Object Parameter.

The function is still a non-static member function, but its object parameter is declared explicitly.

### **Key Differences**

| Feature                                 | Static `operator()`                     | Explicit Object Parameter                 |
| --------------------------------------- | --------------------------------------- | ----------------------------------------- |
| Object parameter                        | None                                    | Present                                   |
| Access to object state                  | Only through other explicit parameters  | Through the object parameter              |
| Implicit `this`                         | No                                      | No                                        |
| Member function category                | Static                                  | Non-static                                |
| Can be virtual                          | No                                      | No                                        |
| Overloading based on object type        | Limited by static member function rules | Supports different object parameter types |
| Can use `const` on the object parameter | No                                      | Yes                                       |

### **When Should You Use Explicit Object Parameters?**

If your goal is to access the current object while controlling its type, constness, or value category, Explicit Object Parameters are a more appropriate choice.

For example:

```
struct Example {
    int value = 10;

    int& get(this Example& self) {
        return self.value;
    }

    const int& get(this const Example& self) {
        return self.value;
    }
};
```

In contrast, if you do not need an object parameter at all, Static Call Operator is the simpler option.

## **Important Overloading Considerations**

Like other functions, `operator()` can be overloaded.

Static Call Operator follows the same general overloading principles.

### **Overloading by Parameter Count**

```
struct Calculator {
    static int operator()(int value) {
        return value * value;
    }

    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    int first = Calculator{}(5);
    int second = Calculator{}(10, 20);
}
```

In this example, the compiler selects the appropriate overload based on the number and types of the arguments.

### **Combining Static and Non-Static Overloads**

C++23 allows static and non-static `operator()` overloads to coexist in the same class, provided that the applicable overloading and overload resolution rules are satisfied.

For example:

```
struct Example {
    int operator()(int value) const {
        return value;
    }

    static int operator()(double value) {
        return static_cast<int>(value * 2);
    }
};

int main() {
    Example object;

    int first = object(10);
    int second = object(2.5);
}
```

In this example:

- The `int` argument selects the non-static overload.
- The `double` argument selects the static overload.

The important point is that invoking an operator through an object does not, by itself, determine whether the static or non-static overload is selected. The argument types, parameter counts, and overload resolution rules determine the result.

### **Overloading Cannot Differ Only by Static**

The presence or absence of `static` alone is not an independent criterion for distinguishing overloaded functions.

Therefore, two functions cannot be declared as separate overloads solely because one is static and the other is non-static when their other parameters are identical.

## **Important Operator Overloading Rules**

Static Call Operator remains subject to the general rules of operator overloading in C++.

Consequently, several restrictions that apply to operators in general still apply.

### **The Operator Must Be a Member Function**

`operator()` is a special operator and must be defined as a member function.

Unlike certain other operators, it cannot be defined as an independent non-member or free function.

### **Parameter Count and Types**

The number of parameters accepted by `operator()` can vary.

Template parameters and Variadic Templates can also be used.

For example:

```
#include <iostream>

struct Printer {
    template<typename... Args>
    static void operator()(Args&&... args) {
        ((std::cout << args << ' '), ...);
    }
};

int main() {
    Printer{}(10, 20, 30, "Hello");
}
```

This example demonstrates a Variadic Static Call Operator.

### **Different Return Types**

Static Call Operator is not restricted to a particular return type.

It can return various types, including:

- `void`
- Numeric types
- References
- Pointers
- Template-dependent types
- User-defined types

For example:

```
struct Factory {
    static auto operator()(int value) {
        return value * 2.5;
    }
};
```

This example uses return type deduction.

## **Practical Applications**

### **Designing Utility Functors**

For classes that perform an independent operation, Static Call Operator can provide a simple interface.

```
struct IsPositive {
    static bool operator()(int value) {
        return value > 0;
    }
};

int main() {
    bool result = IsPositive{}(10);
}
```

This type of Functor is suitable for stateless operations.

### **Designing Stateless Comparators**

Stateless Comparators are a natural application of this feature in algorithms such as `std::sort`.

```
#include <algorithm>
#include <vector>

struct Compare {
    static bool operator()(int a, int b) {
        return a > b;
    }
};

int main() {
    std::vector<int> values{3, 1, 5, 2, 4};

    std::sort(values.begin(), values.end(), Compare{});
}
```

This Comparator produces descending order.

### **Designing Stateless Function Adapters**

This feature can be used to create classes that expose a specific operation through a Callable Object interface.

```
struct AbsoluteValue {
    static constexpr int operator()(int value) noexcept {
        return value < 0 ? -value : value;
    }
};
```

However, this implementation has an overflow problem for `INT_MIN`. A generic implementation must account for the data type and its numeric limitations.

### **Designing Compile-Time Algorithms**

Combining `static` with `constexpr` allows you to design Callable Types that can also be used during compile time.

```
struct Factorial {
    static constexpr unsigned long long operator()(unsigned int n) {
        return n <= 1 ? 1 : n * operator()(n - 1);
    }
};

static_assert(Factorial{}(5) == 120);
```

This example uses a recursive constexpr function.

However, the numeric limitations of `unsigned long long` still apply, and sufficiently large inputs can cause overflow.

## **Common Mistakes**

### **Assuming Static Eliminates the Need for Callable Objects**

A common misconception is that defining a Static Call Operator eliminates the need to create an object.

In practice, an object can still be created to use the usual Functor invocation syntax:

```
struct Example {
    static void operator()() {
    }
};

int main() {
    Example{}();
}
```

However, the operator can also be invoked directly through class scope:

```
Example::operator()();
```

Therefore, an object is not required for the execution of the function itself when using class-scope invocation.

### **Attempting to Access this**

Since the operator is static, using `this` inside its body is not permitted.

```
struct Example {
    static void operator()() {
        // Invalid: static member functions have no this pointer
        // this->process();
    }

    void process() {
    }
};
```

To address this issue, you must explicitly pass the object to the function or define the operator as non-static.

### **Using Static for Stateful Functors**

If a Functor needs access to internal state, making its operator static is generally not an appropriate choice.

For example:

```
struct Counter {
    int count = 0;

    void operator()() {
        ++count;
    }
};
```

In this case, a non-static `operator()` is necessary because the function's behavior depends on the object's state.

### **Assuming Static Is Always Faster**

Static Call Operator is not necessarily faster than a non-static Call Operator.

Compilers can also optimize traditional stateless Functors.

Therefore, the choice between these two approaches should be based on design and semantics rather than an assumed performance improvement.

### **Confusing Static Lambdas with Captureless Lambdas**

Not every captureless Lambda is necessarily a Static Lambda.

For example:

```
auto normal = [](int value) {
    return value * 2;
};

auto staticLambda = [](int value) static {
    return value * 2;
};
```

Both lambdas have no captures, but the second explicitly declares its call operator as static.

This distinction is relevant to the properties of the generated Closure Type.

## **Compatibility with Different C++ Standard Versions**

Static Call Operator was introduced in the C++23 standard.

Therefore, code using this feature requires a compiler with appropriate C++23 support.

| Standard | Support                     |
| -------- | --------------------------- |
| C++11    | Not supported               |
| C++14    | Not supported               |
| C++17    | Not supported               |
| C++20    | Not supported               |
| C++23    | Introduced and standardized |

### **Feature-Test Macro**

You can use the following Feature-Test Macro to check compiler support:

```
#ifdef __cpp_static_call_operator
    // Static call operator support is available
#endif
```

The standardized value of this macro for the feature is:

```
202207L
```

A more precise check can be implemented as follows:

```
#if defined(__cpp_static_call_operator) && \
    __cpp_static_call_operator >= 202207L

    #define HAS_STATIC_CALL_OPERATOR 1

#else

    #define HAS_STATIC_CALL_OPERATOR 0

#endif
```

Note that enabling the C++23 language mode in a compiler does not guarantee that every feature of the standard has been implemented.

Actual support depends on the compiler version and the completeness of its implementation.

## **Comparison with Earlier Approaches**

Stateless Functors could already be implemented before C++23.

Therefore, Static Call Operator does not introduce an entirely new way to perform stateless operations. Instead, it provides a standardized way to express this design more directly.

### **Using a Non-Static Functor**

```
struct OldStyle {
    int operator()(int value) const {
        return value * 2;
    }
};
```

This approach is valid in older C++ standards and remains useful for Functors that have state or require object semantics.

### **Using an Ordinary Static Member Function**

```
struct Utility {
    static int execute(int value) {
        return value * 2;
    }
};
```

This approach is suitable for direct function invocation, but it does not provide a Callable Object interface.

### **Using a Function Pointer**

```
int doubleValue(int value) {
    return value * 2;
}

int main() {
    int (*function)(int) = doubleValue;

    int result = function(10);
}
```

Function Pointers are suitable when an API requires an independent function.

### **Using Static operator()**

```
struct StaticFunctor {
    static int operator()(int value) {
        return value * 2;
    }
};

int main() {
    int result = StaticFunctor{}(10);
}
```

This approach is suitable when you want a Callable Type whose behavior does not depend on instance state.

## **Design and Performance Considerations**

### **Does Static Call Operator Reduce Memory Usage?**

Static Call Operator does not, by itself, reduce the size of an object.

For example:

```
struct Example {
    static void operator()() {
    }
};
```

This is still an ordinary class, and objects of this class can be created.

However, if a class has no non-static data members, its object size must still be at least one byte under the language rules.

Therefore, declaring the operator as static does not make the object zero-sized.

### **Can It Be Used with Empty Base Optimization?**

Yes. Static Call Operator can be used in empty classes, and such classes can still benefit from Empty Base Optimization.

For example:

```
struct Policy {
    static int operator()(int value) {
        return value * 2;
    }
};

struct Container : Policy {
    int data = 10;
};
```

The `Policy` class is an Empty Class, and the rules governing Empty Base Optimization apply independently of whether its call operator is static.

### **Is Static Call Operator More Suitable for Thread Safety?**

Declaring an operator as static does not automatically make it thread-safe.

If the operator accesses shared mutable state, Data Races can still occur.

For example:

```
struct Counter {
    inline static int count = 0;

    static void operator()() {
        ++count;
    }
};
```

If multiple threads invoke this operator concurrently, unsynchronized access to `count` can cause a Data Race.

Therefore, static member functions should not be confused with thread-safe functions.

## **Choosing Between Static and Non-Static**

To make the right choice, first determine whether the intended behavior actually depends on object state.

| Scenario                                                     | Recommended Choice               |
| ------------------------------------------------------------ | -------------------------------- |
| Behavior is completely independent of instance state         | Static `operator()`              |
| Behavior depends on object members                           | Non-Static `operator()`          |
| Captures are required in a Lambda                            | Ordinary Lambda                  |
| Captureless Lambda with explicit static semantics is desired | Static Lambda                    |
| Polymorphic dispatch is required                             | Virtual Non-Static `operator()`  |
| Controlled access to the current object is required          | Explicit Object Parameter        |
| An API requires an ordinary Function Pointer                 | Static Function or Free Function |
| State must be maintained between invocations                 | Stateful Functor                 |

The important point is that Static Call Operator is not a replacement for all traditional Functors. It is simply another option for designing Callable Types that do not depend on object state.

## **Summary**

Static `operator()` was introduced in the C++23 standard and makes it possible to define a Function Call Operator without an implicit object parameter.

This feature is useful for designing stateless Functors, Generic Callable Objects, and Static Lambdas.

Its main advantages include:

- The ability to define Callable Types that are independent of instance state.
- Elimination of the implicit `this` pointer dependency.
- Direct invocation of the operator through class scope.
- Support for Static Lambdas in C++23.
- Compatibility with Generic Programming, Concepts, and Templates.
- The ability to convert the static operator into an ordinary Function Pointer.

Its main limitations include:

- No direct access to non-static members.
- No `this` pointer.
- No cv-qualifiers or ref-qualifiers.
- No virtual functions or overriding.
- No captures in Static Lambdas.

Ultimately, Static Call Operator should be used when the behavior of a Callable Object is genuinely independent of instance state.

If the function depends on object state or requires capabilities such as polymorphism and ref-qualification, a Non-Static Call Operator remains the more appropriate choice.

This feature should be viewed as an additional tool for designing better Callable Objects in C++23, not as a general replacement for traditional Functors.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>