<div align="center">

[🇺🇸 English](./designated_initializers.md) | [🇮🇷 فارسی](../../fa/cpp20/designated_initializers.md)

</div>

---

# Designated Initializers in C++20

## Introduction

Designated Initializers are a useful C++20 language feature that allows aggregate members to be initialized by specifying their names.

Before this feature was introduced, initializing the members of a `struct` was typically done according to the order in which the members were declared. As a result, programmers had to remember the order of the fields, and readers had to refer to the structure definition to understand which values corresponded to which members.

Consider the following structure:

```
struct ServerConfig {
    int port;
    bool use_tls;
    int timeout_ms;
};
```

With traditional aggregate initialization, the object is initialized as follows:

```
ServerConfig config{443, true, 5000};
```

Although this code is valid, understanding the meaning of each value requires knowing the declaration order of the members in `ServerConfig`.

Designated Initializers solve this problem by allowing member names to be specified explicitly:

```
ServerConfig config{
    .port = 443,
    .use_tls = true,
    .timeout_ms = 5000
};
```

Now, the relationship between each value and its corresponding member is immediately clear.

This feature is particularly useful for data structures containing many members, application configuration objects, messages, and data records.

An important point is that Designated Initializers are a C++20 language feature and should not be considered identical to the similar feature available in the C language. C++ has specific restrictions that will be discussed throughout this document.

## Understanding the Problem This Feature Solves

### Ambiguity in Positional Initialization

With traditional aggregate initialization, values are assigned to members according to their declaration order:

```
struct Window {
    int width;
    int height;
    bool resizable;
    int refresh_rate;
};

Window window{1920, 1080, true, 144};
```

Understanding this statement requires examining the definition of `Window`.

The problem becomes more significant when multiple members have the same type:

```
struct NetworkConfig {
    int server_port;
    int client_port;
    int timeout;
};

NetworkConfig config{8080, 9090, 3000};
```

In this example, accidentally swapping the first two values can introduce a logical error without producing any compiler diagnostics.

### Improving Code Readability

With Designated Initializers, the purpose of each initialization becomes explicit:

```
NetworkConfig config{
    .server_port = 8080,
    .client_port = 9090,
    .timeout = 3000
};
```

This approach provides several advantages:

- The destination of each value is explicit.
- Code is easier to understand without repeatedly consulting the structure definition.
- The likelihood of mistakes caused by incorrect value ordering is reduced.
- Future changes to the data structure are easier to review.

### Selective Member Initialization

Another important advantage is the ability to initialize only the members that are needed.

For example:

```
struct ApplicationConfig {
    int max_connections = 100;
    int timeout = 30;
    bool enable_logging = false;
    bool enable_cache = true;
};

ApplicationConfig config{
    .timeout = 60,
    .enable_logging = true
};
```

In this example, only two members are explicitly initialized. The remaining members use the default values defined in the structure.

This feature is particularly useful when creating configuration objects with many optional settings.

## The Main Requirement: The Type Must Be an Aggregate

The most important restriction of Designated Initializers is that they can only be used with aggregates.

Therefore, before using this feature, we must determine whether the target type is actually an aggregate.

### Aggregate Definition in C++20

Under the C++20 standard, a class must satisfy the following conditions to qualify as an aggregate:

- It has no user-declared or inherited constructors.
- It has no non-static data members that are `private` or `protected`.
- It has no `virtual` member functions.
- It has no `virtual`, `private`, or `protected` base classes.

The `struct` and `class` keywords do not fundamentally change these rules. Their primary difference is the default access level of their members.

The following structure is a valid aggregate:

```
struct Point {
    int x;
    int y;
};

Point point{
    .x = 10,
    .y = 20
};
```

However, the following structure is not an aggregate:

```
struct Point {
    int x;
    int y;

    Point(int x_value, int y_value)
        : x{x_value}, y{y_value} {}
};
```

This is because it has a user-defined constructor.

Consequently, the following initialization is invalid:

```
// Invalid: Point is not an aggregate.
Point point{
    .x = 10,
    .y = 20
};
```

### The Effect of Default Member Initializers

In C++20, having `default member initializers` does not prevent a class from being an aggregate.

For example:

```
struct UserConfig {
    int max_attempts = 5;
    int timeout = 30;
    bool enabled = true;
};

UserConfig config{
    .timeout = 60
};
```

This structure is still an aggregate, and Designated Initializers can be used with it.

This is important when considering compatibility with different C++ standard versions:

| Standard | Default Member Initializers                               |
| -------- | --------------------------------------------------------- |
| C++11    | Their presence prevented a class from being an aggregate. |
| C++14    | This restriction was removed.                             |
| C++17    | They remain permitted.                                    |
| C++20    | They remain permitted.                                    |

### Aggregate Types vs. Classes with Constructors

Designated Initializers are not a general replacement for constructors.

A constructor can be responsible for several tasks:

- Input validation
- Maintaining class invariants
- Resource management
- Executing required logic during object construction
- Preventing the creation of invalid states

In contrast, Designated Initializers simply initialize the members of an aggregate.

Therefore, if constructing an object requires validation logic or resource management, using a constructor or factory function is generally a more appropriate choice.

## Understanding the Syntax and Basic Rules

### General Syntax

The general syntax of Designated Initializers is:

```
Type object{
    .member1 = value1,
    .member2 = value2,
    .member3 = value3
};
```

The following form can also be used:

```
Type object = {
    .member1 = value1,
    .member2 = value2
};
```

In both cases, the member name is specified using a leading dot (`.`), followed by an assignment operator (`=`) and the corresponding value.

### Using Different Expressions as Values

The value assigned to each member can be a constant, a variable, or any valid C++ expression:

```
#include <string>

struct Employee {
    int id;
    std::string name;
    double salary;
    bool active;
};

int employee_id = 1001;
double base_salary = 4500.0;

Employee employee{
    .id = employee_id,
    .name = "Alice",
    .salary = base_salary * 1.2,
    .active = true
};
```

In this example, member values are obtained from different expressions.

### Declaration Order Must Be Preserved

One of the most important rules of Designated Initializers in C++20 is that designated members must appear in the same order in which they are declared in the class.

For example:

```
struct Product {
    int id;
    double price;
    int quantity;
    bool available;
};
```

The following initialization is valid:

```
Product product{
    .id = 10,
    .quantity = 5,
    .available = true
};
```

This is because the order of the selected members is consistent with their declaration order.

However, the following example is invalid:

```
// Invalid: designated initializers are out of declaration order.
Product product{
    .quantity = 5,
    .id = 10,
    .available = true
};
```

Even though all the member names are correct, the incorrect ordering causes a compilation error.

An important distinction is that omitting some members is allowed, but changing the order of the selected members is not.

### Only Direct Member Names Are Allowed

In C++20, each designator must refer to a direct non-static data member of the aggregate being initialized.

For example:

```
struct Address {
    int house_number;
    int postal_code;
};

struct Person {
    int id;
    Address address;
};
```

The following syntax is invalid:

```
// Invalid: nested member designators are not supported.
Person person{
    .id = 10,
    .address.postal_code = 12345
};
```

This is because `postal_code` is not a direct member of `Person`.

The correct approach is to initialize the nested member separately:

```
Person person{
    .id = 10,
    .address = {
        .house_number = 25,
        .postal_code = 12345
    }
};
```

### Duplicate Designators Are Not Allowed

Each member can appear at most once in a list of Designated Initializers.

```
struct Item {
    int id;
    int quantity;
};
```

The following example is invalid:

```
// Invalid: id is designated more than once.
Item item{
    .id = 10,
    .id = 20,
    .quantity = 5
};
```

To avoid this situation, each member should either be specified exactly once or be omitted so that its default initialization rules apply.

### Positional and Designated Initialization Cannot Be Mixed

In C++20, positional initializers and Designated Initializers cannot be combined within the same initializer list.

For example:

```
struct Color {
    int red;
    int green;
    int blue;
};
```

The following expression is invalid:

```
// Invalid: positional and designated initialization are mixed.
Color color{
    255,
    .green = 128,
    .blue = 64
};
```

You must choose either positional initialization or designated initialization.

## Understanding the Behavior of Uninitialized Members

One important aspect of using Designated Initializers is understanding what happens to members that are not explicitly specified in the initializer list.

This behavior follows the rules of `Aggregate Initialization`.

### Members with Default Member Initializers

If a member has a default value and is not specified in the Designated Initializers list, its default value is used.

```
struct ServerSettings {
    int port = 8080;
    int timeout = 30;
    int max_connections = 100;
    bool enable_ssl = false;
};

ServerSettings settings{
    .port = 443,
    .enable_ssl = true
};
```

The resulting member values are:

| Member            | Final Value |
| ----------------- | ----------- |
| `port`            | 443         |
| `timeout`         | 30          |
| `max_connections` | 100         |
| `enable_ssl`      | true        |

In this example, the default values of omitted members are preserved.

### Members Without Default Initializers

If a member is neither explicitly specified in the initializer list nor provided with a `default member initializer`, it is initialized with an empty initializer list according to the rules of Aggregate Initialization.

For example:

```
struct Statistics {
    int requests;
    double response_time;
    bool successful;
};

Statistics stats{
    .requests = 100
};
```

The resulting member values are:

- `requests` is `100`.
- `response_time` is `0.0`.
- `successful` is `false`.

These results apply to the scalar members shown above, but they should not be generalized to every type.

For example, a reference member must be bound to a valid object and cannot be left without valid initialization.

### Explicit Values Take Precedence Over Default Values

If a member has a default initializer and is also explicitly specified in Designated Initializers, the explicitly provided value is used.

```
struct RetryPolicy {
    int attempts = 3;
    int delay_ms = 100;
};

RetryPolicy policy{
    .attempts = 10
};
```

In this example:

- `attempts` is `10`.
- `delay_ms` remains `100`.

## Understanding Nested Member Initialization

One important application of this feature is constructing hierarchical data structures.

Suppose an application's configuration consists of several sections:

```
struct DatabaseConfig {
    std::string host;
    int port;
};

struct LoggingConfig {
    bool enabled;
    int level;
};

struct ApplicationConfig {
    DatabaseConfig database;
    LoggingConfig logging;
};
```

The main structure and its nested members can be initialized together:

```
ApplicationConfig config{
    .database = {
        .host = "localhost",
        .port = 5432
    },
    .logging = {
        .enabled = true,
        .level = 2
    }
};
```

In this example, each nested member is initialized independently.

It is important to understand that Designated Initializers do not support chaining nested member names. Each level must be initialized through an initializer associated with its direct member.

### Advantages of Nested Structures

This approach provides several benefits in projects with extensive configuration requirements:

- Logical separation of configuration settings
- Reduced risk of mistakes caused by incorrect value ordering
- Improved readability of complex data structures
- Ability to use default values at every level
- Easier creation of different configuration instances

However, excessive nesting can reduce readability. In such situations, using intermediate variables or factory functions may provide a clearer design.

## Using Designated Initializers with Union

Designated Initializers can also be used with `union` types.

In a union, all members share the same memory, and only one member can be active at a time.

For example:

```
union Value {
    int integer;
    double real;
    bool boolean;
};
```

Designated Initializers allow you to specify which member should be initialized:

```
Value value{
    .real = 3.14
};
```

In this example, `real` becomes the active union member.

### Union Initialization Restrictions

Multiple union members cannot be explicitly initialized in the same initializer list.

The following example is invalid:

```
// Invalid: only one union member can be explicitly initialized.
Value value{
    .integer = 10,
    .real = 2.5
};
```

Additionally, if a union has a `default member initializer` for one of its members, the default initialization rules for that union must be considered when no member is explicitly specified.

In the absence of such an initializer, the first union member is initialized with an empty initializer list.

### Using Anonymous Unions

This feature is also useful with `anonymous union` members:

```
struct Token {
    bool is_number;

    union {
        int number;
        char symbol;
    };
};
```

An initialization example is:

```
Token token{
    .is_number = true,
    .number = 42
};
```

In this example, `number` becomes the active member of the anonymous union.

Only one member of the anonymous union should be selected in each initialization.

## Understanding the Relationship with Aggregate Initialization

Designated Initializers are part of the `Aggregate Initialization` mechanism in C++.

To understand this feature more clearly, it is useful to compare it with other initialization techniques.

### Comparison with Positional Aggregate Initialization

With the traditional approach, initialization follows member declaration order:

```
struct Rectangle {
    int width;
    int height;
};

Rectangle a{100, 200};
```

With designated initialization, the destination of each value is explicitly specified:

```
Rectangle b{
    .width = 100,
    .height = 200
};
```

Both approaches produce the same result.

The primary difference is readability and how explicitly the programmer communicates their intent.

Positional initialization remains suitable for small and simple structures, while designated initialization is generally a better choice for structures with many members.

### Comparison with Constructors

Constructors are designed to control the object construction process.

For example:

```
#include <stdexcept>

class Port {
public:
    explicit Port(int value)
        : value_{value} {
        if (value < 1 || value > 65535) {
            throw std::out_of_range("Invalid port");
        }
    }

    int get() const {
        return value_;
    }

private:
    int value_;
};
```

In this example, the constructor validates the allowed port range.

This class is not an aggregate because it has a constructor and a `private` data member.

Consequently, Designated Initializers cannot be used to construct it.

This demonstrates why the choice between aggregates and constructors should be based on the responsibilities and design of the type, rather than simply on which syntax is shorter.

### Comparison with std::initializer_list

`std::initializer_list` is a C++ standard library type that allows a function or constructor to receive a list of values of the same type.

For example:

```
#include <initializer_list>

class NumberCollection {
public:
    NumberCollection(std::initializer_list<int> values)
        : values_{values} {}

private:
    std::initializer_list<int> values_;
};
```

Designated Initializers, however, are a language-level syntactic feature that identifies aggregate members by name.

Therefore, these two concepts have different purposes and behaviors.

## Differences Between C and C++20 Designated Initializers

The C language has supported Designated Initializers since earlier versions.

However, the rules governing this feature in C differ from those in C++20.

The most important differences are:

| Feature                                         | C         | C++20         |
| ----------------------------------------------- | --------- | ------------- |
| Designating structure members by name           | Supported | Supported     |
| Arbitrary designator ordering                   | Supported | Not allowed   |
| Nested designators                              | Supported | Not supported |
| Array index designators                         | Supported | Not supported |
| Mixing positional and designated initialization | Supported | Not allowed   |
| Designating direct aggregate members            | Supported | Supported     |

### Differences in Member Ordering

In C, members can be designated in an order different from their declaration order.

In C++20, however, the order of Designators must follow the declaration order of the members.

### Differences in Array Initialization

C supports index designators:

```
int values[5] = {
    [2] = 42
};
```

This syntax is not supported in C++20.

Arrays in C++ must be initialized using the standard Aggregate Initialization rules.

### Differences in Nested Members

C allows chained designators for nested members.

In C++20, you must first designate the direct member and then initialize its nested members through a separate initializer.

These differences are particularly important when porting C code to C++.

## Understanding the Relationship with Base Classes

In C++20, having a base class does not automatically prevent a derived class from being an aggregate.

However, the base class must satisfy the restrictions imposed by the aggregate definition.

For example:

```
struct Base {
    int id;
};

struct Derived : Base {
    int value;
};
```

`Derived` can be an aggregate.

However, Designated Initializers in C++20 can only designate direct data members of the class being initialized. They cannot directly designate members inherited from a base class.

To initialize the base-class subobject, the rules of Aggregate Initialization must be followed:

```
Derived object{
    {100},
    .value = 200
};
```

This example uses positional initialization for the base-class subobject and designated initialization for the direct member. However, this combination is not permitted within a single initializer list.

Therefore, the example above is invalid, and an initialization style consistent with C++20 rules must be selected.

A simple and portable approach is to use positional initialization for the entire aggregate:

```
Derived object{
    {100},
    200
};
```

This is an important detail: C++20 does not allow positional and designated initialization to be combined within the same initializer when initializing the base-class subobject and derived-class members.

## Understanding Narrowing Conversions

Designated Initializers follow the rules of `List Initialization`.

Consequently, narrowing conversions are not allowed.

For example:

```
struct Measurement {
    int count;
};
```

The following initialization is invalid:

```
// Invalid: floating-point to integer narrowing conversion.
Measurement measurement{
    .count = 3.7
};
```

This is because converting a floating-point value to an integer can result in information loss.

If such a conversion is genuinely required, it should be performed explicitly and with appropriate checks for the input range and precision.

For example:

```
double input = 3.7;

int converted = static_cast<int>(input);

Measurement measurement{
    .count = converted
};
```

In this case, the conversion is performed before initialization.

However, using `static_cast` does not automatically make an input value valid from a logical or application perspective. Responsibility for validating the value still belongs to the programmer.

## Understanding Evaluation Order and Side Effects

During Aggregate Initialization, aggregate members are initialized according to the order of their elements.

Consequently, the expressions used to initialize those members are evaluated in a predictable order.

For example:

```
#include <iostream>

int next_value() {
    static int value = 0;
    return ++value;
}

struct Pair {
    int first;
    int second;
};

int main() {
    Pair pair{
        .first = next_value(),
        .second = next_value()
    };

    std::cout << pair.first << '\n';
    std::cout << pair.second << '\n';
}
```

Output:

```
1
2
```

First, `first` is initialized, followed by `second`.

Nevertheless, it is generally better to avoid complex expressions with side effects in initializers, as they can make code harder to understand and maintain.

## Practical Applications of Designated Initializers

### Creating Configuration Objects

One of the best applications of this feature is creating configuration objects.

```
#include <string>

struct DatabaseConfig {
    std::string host = "localhost";
    int port = 5432;
    int pool_size = 10;
    bool enable_tls = false;
};

DatabaseConfig config{
    .host = "database.internal",
    .port = 5433,
    .enable_tls = true
};
```

In this example, the required settings are explicitly specified, while the remaining members use their default values.

This pattern is particularly suitable for application settings, service connections, and resource configuration.

### Creating Test Data

In Unit Testing, we often need to create instances of a data structure with different states.

Designated Initializers make this process simpler:

```
#include <string>

struct User {
    int id;
    std::string name;
    bool active = true;
};

User active_user{
    .id = 1,
    .name = "Alice"
};

User inactive_user{
    .id = 2,
    .name = "Bob",
    .active = false
};
```

In this example, the purpose of each instance is immediately clear.

Additionally, changing the member declaration order does not silently change which values are assigned to which members, because each destination is explicitly identified.

### Creating Messages and Data Records

For structures that simply transfer data between different parts of an application, this feature can improve readability.

```
struct Message {
    int id;
    int priority;
    bool encrypted;
    int payload_size;
};

Message message{
    .id = 100,
    .priority = 2,
    .encrypted = true,
    .payload_size = 1024
};
```

However, if this structure is used for serialization or network communication, its memory layout should not be treated as a serialization format without an explicit design.

Issues such as `padding`, `endianness`, and differences between compiler implementations must be handled separately.

### Creating Customized Default Values

This feature also makes it possible to create multiple instances with different configurations:

```
struct ThreadPoolConfig {
    int worker_count = 4;
    int queue_size = 100;
    bool enable_monitoring = false;
};

ThreadPoolConfig development{
    .worker_count = 2,
    .enable_monitoring = true
};

ThreadPoolConfig production{
    .worker_count = 16,
    .queue_size = 1000
};
```

In this example, each environment overrides only the settings that differ from the default configuration.

## Important Limitations

### Constructor-Related Restrictions

Any user-declared constructor can prevent a class from being an aggregate.

This also includes constructors declared with `= default` or `= delete`, because they are still considered user-declared.

For example:

```
struct Configuration {
    int value;

    Configuration() = default;
};
```

This class is not an aggregate, so Designated Initializers cannot be used with it.

### Access Control Restrictions

Having a non-static `private` or `protected` data member prevents a class from being an aggregate.

```
class User {
private:
    int id;

public:
    std::string name;
};
```

This class is not an aggregate.

This restriction is related to the principles of Encapsulation and prevents Designated Initializers from being used as a way to bypass member access restrictions.

### Restrictions Related to Virtual Functions

A class containing a `virtual` member function is not an aggregate.

```
struct Base {
    virtual void execute() {}
};
```

Consequently, Designated Initializers cannot be used with this class.

### Restrictions Related to Static Data Members

Static data members are not part of an aggregate object's subobjects and cannot be initialized using Designated Initializers.

For example:

```
struct Config {
    int port;
    static int default_port;
};
```

The following expression is invalid:

```
// Invalid: static data members cannot be designated.
Config config{
    .port = 8080,
    .default_port = 443
};
```

The `default_port` member must be managed according to the normal rules for static members.

### Inheritance-Related Restrictions

If a base class is `virtual`, `private`, or `protected`, the derived class is not an aggregate.

Additionally, Designated Initializers in C++20 do not allow direct designation of base-class members.

## Common Mistakes

### Assuming Member Order Is Optional

One common mistake is assuming that because members are specified by name, their order does not matter.

This assumption is incorrect in C++20.

The order of Designators must match the declaration order of the members.

### Assuming the Feature Works with Every Class

Designated Initializers are only available for aggregates.

Therefore, constructors, virtual functions, and non-public data members can prevent a class from using this feature.

### Using C Language Syntax

The rules for Designated Initializers in C and C++20 are different.

Consequently, directly transferring C code into C++ without reviewing the target language rules can result in compilation errors.

### Ignoring Default Member Initializers

If a member has a default initializer, omitting it from the initialization list causes that default value to be used.

Programmers should account for this behavior when designing configurations and default data states.

### Overusing Aggregates

Not every class should be converted into an aggregate merely to take advantage of Designated Initializers.

If a class is responsible for maintaining invariants, validating input, or managing resources, constructors may be a more appropriate choice.

### Ignoring API Stability

When users of a library rely on Designated Initializers, the names of its public members become part of the library's usage interface.

As a result:

- Renaming members can break consumer code.
- Removing a member may introduce incompatibilities.
- Adding a constructor can cause the type to stop being an aggregate.
- Changing member access levels can remove the ability to use designated initialization.

Therefore, using this feature in public APIs requires considering the maintenance cost of future changes.

## Design Considerations

### When to Use Designated Initializers

Designated Initializers are generally a good choice in the following situations:

- The data structure contains multiple members.
- Several members have the same type.
- Many members have default values.
- Code readability is particularly important.
- Object construction does not require complex logic.
- The type is a simple Data Transfer Object.

### When Not to Use Designated Initializers

Under the following conditions, constructors or factory functions are generally preferable:

- Object construction requires validation across multiple members.
- The class must maintain specific invariants.
- Data members need to remain `private`.
- Object creation requires resource management.
- Internal implementation details should not become part of the public API.
- Object construction involves complex logic.

### Maintaining a Consistent Style

In a professional project, it is preferable to establish a clear and consistent initialization style for each data type.

For example, if a configuration structure is constructed using Designated Initializers throughout different parts of a project, maintaining the same approach in similar situations is generally beneficial.

Inconsistent use of different initialization styles can reduce readability, even when all the expressions are valid C++.

## Differences Across C++ Standard Versions

Designated Initializers were introduced in C++20, but Aggregate Initialization has undergone several important changes across C++ standard versions.

Some relevant changes include:

| Standard | Related Change                                                          |
| -------- | ----------------------------------------------------------------------- |
| C++11    | Introduction of modern List Initialization and Aggregate Initialization |
| C++14    | Aggregate classes became permitted to have Default Member Initializers  |
| C++17    | The definition of aggregates was expanded to support base classes       |
| C++20    | Introduction of Designated Initializers                                 |
| C++23    | Changes and clarifications related to Aggregate Initialization          |

An important point is that support for a feature in a newer standard does not mean that all similar syntax from the C language is supported.

When developing portable libraries, the exact rules of the target C++ standard must be followed.

## Compiler Support

Using Designated Initializers requires a compiler operating in C++20 mode or a later standard mode.

### Compiling with GCC

```
g++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o main
```

### Compiling with Clang

```
clang++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o main
```

### Compiling with MSVC

```
cl /std:c++20 /W4 /EHsc main.cpp
```

Exact support depends on the compiler version and its level of standard implementation.

For professional projects, it is recommended to test the targeted compiler versions through the CI process and avoid relying on compiler-specific extensions as if they were standard C++ behavior.

## Summary

Designated Initializers in C++20 provide a readable and practical way to initialize aggregate members by specifying their names.

This feature addresses problems such as dependence on member declaration order, difficulty identifying the destination of individual values, and reduced readability in larger data structures.

Its main advantages include:

- Explicit identification of each initialization target
- Improved code readability
- Reduced errors caused by incorrect value ordering
- Selective member initialization
- Effective use of Default Member Initializers
- Simpler construction of configuration objects and data records

However, its limitations must also be clearly understood:

- It can only be used with aggregates.
- The order of Designators must match the declaration order of the members.
- Only direct non-static data members can be designated.
- Nested initialization must be performed through the corresponding direct member.
- Positional and designated initialization cannot be mixed.
- Its rules differ from those of Designated Initializers in C.
- It is not a replacement for constructors responsible for validation and maintaining invariants.

Ultimately, Designated Initializers are a valuable tool for designing and implementing modern C++ code, provided they are used in appropriate situations and with a thorough understanding of the Aggregate Initialization rules.

## References

The following resources are useful for studying the standard details and exploring additional examples:

- [cppreference — Aggregate initialization](https://en.cppreference.com/w/cpp/language/aggregate_initialization?utm_source=chatgpt.com)
- [cppreference — Aggregate](https://en.cppreference.com/w/cpp/language/aggregate?utm_source=chatgpt.com)
- [cppreference — List initialization](https://en.cppreference.com/w/cpp/language/list_initialization?utm_source=chatgpt.com)
- [WG21 N4810 — C++20 Working Draft](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/n4810.pdf?utm_source=chatgpt.com)
- [ISO C++ — C++ Standards](https://isocpp.org/std/the-standard?utm_source=chatgpt.com)
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>