<div align="center">

[🇺🇸 English](./std_expected.md) | [🇮🇷 فارسی](../../fa/cpp23/std_expected.md)

</div>

---

# **A Comprehensive Guide to std::expected in C++23**

## **Introduction**

Error handling is one of the most important aspects of designing reliable software. C++ provides several approaches to error handling, including exceptions, special return values, pointers, `std::optional`, and error-code-based patterns.

Each approach has its own advantages and limitations. For example, exceptions separate normal execution from error-handling paths, but they are not always the best choice for APIs where failure is considered a normal part of the operation's contract.

On the other hand, using error codes together with output parameters passed by reference or pointer can complicate function interfaces and increase the likelihood of mistakes.

To address these problems, C++23 introduced `std::expected`.

This type provides a way to represent the result of an operation in which both the successful value and error information are part of the function's return type.

Using `std::expected`, developers can:

- Represent the successful result of an operation.
- Store error information in a type-safe manner.
- Handle predictable failures without using exceptions.
- Propagate errors through chains of operations.
- Express function contracts through the type system.
- Combine successful operations and error handling using Monadic Operations.

This guide explores the internal structure, usage, primary operations, advanced capabilities, limitations, and practical applications of this type.

## **Introduction to std::expected**

`std::expected` is a standard library type introduced in C++23. It is available through the following header:

```
#include <expected>
```

Its primary template declaration is:

```
template<class T, class E>
class expected;
```

In this declaration:

- `T` represents the type of the value returned when an operation succeeds.
- `E` represents the type of the error information stored when an operation fails.

Therefore, an instance of `std::expected<T, E>` is always in one of two states:

- **Value State:** Contains a valid value of type `T`.
- **Error State:** Contains an error value of type `E`.

For example:

```
std::expected<int, std::string>
```

This type can hold either an integer or an error message represented by `std::string`.

An important property is that `std::expected` is always in one of these two states. Unlike some traditional union-based designs, it does not have a valueless state.

### **Why std::expected Was Introduced**

Suppose we have a function that converts a string into an integer.

This operation can fail for several reasons:

- The input is empty.
- The string contains invalid characters.
- The input value is outside the range of the destination type.

One traditional approach is to use a special return value:

```
int parseNumber(const std::string& input) {
    if (input.empty()) {
        return -1;
    }

    return std::stoi(input);
}
```

This approach has several problems:

- `-1` might be a valid result.
- The return type does not communicate the reason for failure.
- Callers must know about an undocumented function contract.
- Supporting multiple error categories becomes difficult.

Another approach is to use an output parameter:

```
bool parseNumber(
    const std::string& input,
    int& result
);
```

With this approach, callers must manage both the Boolean return value and the output parameter.

Using `std::expected` allows us to express the function's contract more precisely:

```
std::expected<int, std::string>
parseNumber(const std::string& input);
```

The return type now clearly indicates that the function either produces an integer or returns error information.

## **Difference Between std::expected and std::optional**

Both `std::expected` and `std::optional` represent results that may not contain the desired value. However, their design goals are different.

`std::optional<T>` was introduced in C++17.

It represents two possible states:

- It contains a value.
- It does not contain a value.

For example:

```
std::optional<int> findUserId();
```

This function might return a valid identifier or indicate that no identifier was found.

However, `std::optional` does not provide information about why the value is unavailable.

In contrast:

```
std::expected<int, UserError> findUserId();
```

can provide both the successful result and the reason for failure.

| Feature                               | std::optional | std::expected |
| ------------------------------------- | ------------- | ------------- |
| Introduced in                         | C++17         | C++23         |
| Stores a successful value             | Yes           | Yes           |
| Stores error information              | No            | Yes           |
| Number of states                      | Two           | Two           |
| Suitable for descriptive errors       | Limited       | Yes           |
| Monadic operations                    | Yes           | Yes           |
| Stores Value and Error simultaneously | No            | No            |

### **When to Use std::optional**

Use `std::optional` when the absence of a value is itself a valid state and there is no need to explain why the value is unavailable.

For example:

```
std::optional<User> findUser(UserId id);
```

If the user does not exist, returning `std::nullopt` is perfectly meaningful.

### **When to Use std::expected**

Use `std::expected` when a failure requires additional information.

For example:

```
enum class DatabaseError {
    connection_failed,
    query_failed,
    permission_denied
};

std::expected<User, DatabaseError>
loadUser(UserId id);
```

With this design, callers can make different decisions depending on the error type.

## **Internal State Representation**

`std::expected<T, E>` uses internal storage to hold one of two possible values.

Its behavior is similar to a discriminated union: in addition to the stored value, the active state is explicitly tracked.

For example:

```
std::expected<int, std::string> result = 42;
```

In this state, the object contains the value `42` of type `int`.

Consider the following example:

```
std::expected<int, std::string> result =
    std::unexpected(std::string{"Invalid input"});
```

The object is now in the Error State, and the error message is stored inside it.

Several important properties follow from this design:

- At any given time, only one of the two values, `T` or `E`, is active.
- The storage required for the values is part of the object itself.
- Using `std::expected` does not inherently require dynamic memory allocation.
- The object's size depends on the implementation, the sizes of `T` and `E`, and their alignment requirements.
- The internal storage does not normally use the heap, although the stored types themselves may allocate dynamic memory.

Therefore, it is incorrect to assume that using `std::expected` is always allocation-free. This also depends on the types stored inside it.

## **Defining an Error Type**

One of the most important design decisions when working with `std::expected` is choosing an appropriate type for `E`.

The error type should provide enough information to identify and handle failures.

### **Using enum class**

For a limited and well-defined set of errors, `enum class` is usually a good choice.

```
enum class ParseError {
    empty_input,
    invalid_character,
    out_of_range
};

std::expected<int, ParseError>
parseInteger(const std::string& input);
```

Advantages include:

- Low storage overhead.
- A clearly defined set of errors.
- The ability to use `switch` statements.
- No dynamic allocation required for the error value itself.
- Suitability for APIs with a limited and known set of error conditions.

### **Using a Custom Structure**

When an error requires additional information, a custom structure can be defined.

```
#include <string>

struct ParseError {
    int errorCode;
    std::size_t position;
    std::string message;
};
```

Now, the error can contain additional information beyond its category.

For example:

```
std::expected<int, ParseError>
parseInteger(const std::string& input);
```

This design is useful for parsers, file readers, and systems that need detailed error reporting.

### **Using std::string**

For small applications or internal APIs, using `std::string` as the error type is perfectly valid.

```
std::expected<int, std::string>
parseInteger(const std::string& input);
```

However, for professional projects, dedicated error types are generally preferable.

Comparing strings to identify error categories is fragile, and changing an error message can unintentionally affect application logic.

### **Designing Hierarchical Error Types**

Large systems may contain multiple subsystems with different error categories.

For example:

```
enum class FileError {
    not_found,
    permission_denied,
    read_failed
};

enum class ParseError {
    invalid_format,
    unexpected_token
};
```

Each subsystem can define its own error type, while higher-level components can convert these errors into a more general representation.

This approach reduces unnecessary dependencies between subsystems.

## **Creating a Successful Value**

To construct a `std::expected` in the successful state, simply provide the desired value.

```
#include <expected>

std::expected<int, int> getValue() {
    return 42;
}
```

In this example, the value `42` is stored in the Value state.

The return type can also be initialized directly:

```
std::expected<std::string, int> getName() {
    return std::string{"Alice"};
}
```

### **Using std::in_place**

The `std::in_place` tag can be used to construct the successful value directly inside the storage of `expected`.

```
#include <expected>
#include <string>

std::expected<std::string, int> result{
    std::in_place,
    5,
    'A'
};
```

In this example, a `std::string` containing five `A` characters is constructed directly.

This is useful when the destination value requires multiple constructor arguments.

It can also avoid unnecessary temporary objects.

However, in many situations, the compiler can eliminate temporary objects through copy elision.

## **Creating an Error State**

To create an error state, use `std::unexpected`.

This type is a wrapper around an error value.

Its general form is:

```
std::unexpected<ErrorType>
```

For example:

```
#include <expected>

enum class MathError {
    division_by_zero
};

std::expected<double, MathError>
divide(double a, double b) {
    if (b == 0.0) {
        return std::unexpected(
            MathError::division_by_zero
        );
    }

    return a / b;
}
```

In this example, the function returns an error when the denominator is zero.

It is important to understand that `std::unexpected` is not a replacement for `std::expected`. It is a wrapper used to construct and propagate an error state.

### **Using std::unexpect**

Another way to construct an error state directly is to use the `std::unexpect` tag.

```
#include <expected>
#include <string>

std::expected<int, std::string> result{
    std::unexpect,
    "Invalid input"
};
```

This constructs the error value directly inside the `expected` object.

The practical difference is:

- `std::unexpected` creates a wrapper around an error value.
- `std::unexpect` tells the `expected` constructor to construct an error value directly.

`std::unexpect` can be useful when direct construction of the error value is important.

## **Checking the State with has_value**

To determine whether an object contains a successful value or an error, use `has_value()`.

```
if (result.has_value()) {
    // Success
} else {
    // Failure
}
```

This function returns a Boolean value.

The object itself can also be used directly in a conditional expression:

```
if (result) {
    // Success
}
```

This works because `std::expected` provides an explicit conversion to `bool`.

This is a common and convenient way to check its state.

However, checking the state does not extract the stored value.

## **Accessing the Successful Value**

There are several ways to access the stored value.

Each has different behavior when the object is in the Error State.

### **Using value()**

The `value()` function returns the successful value.

```
auto result = getValue();

if (result.has_value()) {
    int value = result.value();
}
```

An important detail is that calling `value()` when the object is in the Error State throws an exception of type:

```
std::bad_expected_access<E>
```

This exception also provides access to the stored error information.

```
try {
    int value = result.value();
} catch (const std::bad_expected_access<int>& ex) {
    int error = ex.error();
}
```

Therefore, calling `value()` without first ensuring that the object contains a value may introduce exception handling into the execution path.

### **Using operator\***

The dereference operator can also be used to access the successful value.

```
int value = *result;
```

When a value is present, this provides direct and inexpensive access.

However, unlike `value()`, using this operator when the object is in the Error State results in undefined behavior.

Therefore, use it only after checking the object's state or when the presence of a value is guaranteed by the program's logic.

### **Using operator->**

The `operator->` member function provides access to the members of the stored object.

```
std::expected<std::string, int> result = "Hello";

if (result) {
    auto length = result->size();
}
```

This operator is also valid only when the object is in the Value State.

### **Using value_or()**

The `value_or()` function allows you to specify a default value.

```
std::expected<int, std::string> result =
    std::unexpected("Calculation failed");

int value = result.value_or(0);
```

In this example, the result is in the Error State, so the function returns `0`.

This function is useful when providing a fallback value is logically appropriate.

However, it is important to understand that `value_or()` discards the error information.

Also, its default-value argument is evaluated when the function is called. Therefore, the fallback expression is evaluated even when the `expected` object contains a successful value.

This matters when the fallback expression is expensive.

## **Accessing Error Information**

Use the `error()` function to retrieve the stored error value.

```
if (!result) {
    auto error = result.error();
}
```

This function returns the error information.

It is important to call `error()` only when the object is in the Error State.

Calling it when the object is in the Value State violates the function's precondition.

Therefore, always check the object's state before using it.

### **Difference Between value() and error()**

The `value()` and `error()` functions provide access to different states.

| Function     | Valid state  | Result                               |
| ------------ | ------------ | ------------------------------------ |
| `value()`    | Value State  | Value of type `T`                    |
| `error()`    | Error State  | Value of type `E`                    |
| `value_or()` | Either state | Value of type `T` or a default value |

## **Error Handling in Functions**

One of the most important applications of `std::expected` is designing functions that can encounter predictable failures.

To understand this, consider an implementation of a string-to-integer conversion function.

```
#include <charconv>
#include <expected>
#include <string_view>
#include <system_error>

enum class ParseError {
    empty_input,
    invalid_input,
    out_of_range
};

std::expected<int, ParseError>
parseInteger(std::string_view input) {
    if (input.empty()) {
        return std::unexpected(
            ParseError::empty_input
        );
    }

    int value{};

    const char* begin = input.data();
    const char* end = begin + input.size();

    auto [ptr, ec] = std::from_chars(
        begin,
        end,
        value
    );

    if (ec == std::errc::result_out_of_range) {
        return std::unexpected(
            ParseError::out_of_range
        );
    }

    if (ec != std::errc{} || ptr != end) {
        return std::unexpected(
            ParseError::invalid_input
        );
    }

    return value;
}
```

This implementation uses `std::from_chars`.

This function is designed for converting strings into numbers and, unlike some older approaches, does not throw exceptions.

Its result can also be combined with `std::expected` to provide a type-safe contract for the higher-level function.

An important detail is that `std::from_chars` does not necessarily consume the entire input. For this reason, the example also checks the value of `ptr`.

For example, the input `123abc` should not be accepted as the number `123`.

## **Using std::expected**

Many functions perform operations that do not need to return a value on success.

Examples include:

- Saving information to a file.
- Deleting a database record.
- Sending a message.
- Validating an object.
- Performing a network operation.

In these situations, the following specialization can be used:

```
std::expected<void, E>
```

This type represents only two states:

- The operation succeeded.
- The operation failed.

For example:

```
#include <expected>
#include <string>

std::expected<void, std::string>
saveFile() {
    bool success = true;

    if (!success) {
        return std::unexpected(
            "Unable to save file"
        );
    }

    return {};
}
```

In this example, `return {};` constructs a successful `std::expected<void, std::string>`.

The result can be checked using the usual approach:

```
auto result = saveFile();

if (!result) {
    // Handle failure
}
```

In the `void` specialization, `value()` is still available, but it does not return a meaningful value.

Also, `operator*` and `operator->` are not provided by this specialization.

## **Chaining Operations with Monadic Operations**

One of the important features of `std::expected` in C++23 is its set of Monadic Operations.

These operations make it possible to combine multiple functions without repeatedly writing conditional error checks.

The four primary functions are:

- `and_then()`
- `transform()`
- `or_else()`
- `transform_error()`

These capabilities were added to C++23 based on proposal P2505R5.

### **Understanding and_then()**

The `and_then()` function executes a callback only when the object contains a successful value.

If the object is in the Error State, the callback is not executed, and the error is propagated.

Suppose we have two functions:

```
std::expected<int, ParseError>
parseInteger(std::string_view input);

std::expected<double, MathError>
calculateSomething(int value);
```

To combine these functions, their error types must be compatible.

In the simplest case, if both functions use the same error type, they can be chained as follows:

```
auto result = parseInteger("42")
    .and_then(calculateSomething);
```

In this example:

- If string conversion succeeds, the second function is executed.
- If string conversion fails, the second function is not executed.
- The error is propagated without requiring additional conditional statements.

The callback passed to `and_then()` must return a compatible `std::expected`.

More precisely, this operation is designed to combine functions that can themselves fail.

### **Practical and_then() Example**

```
#include <expected>
#include <string>

enum class Error {
    invalid_input,
    invalid_operation
};

std::expected<int, Error>
parseValue(const std::string& input) {
    if (input.empty()) {
        return std::unexpected(
            Error::invalid_input
        );
    }

    return 10;
}

std::expected<int, Error>
multiplyByTwo(int value) {
    if (value < 0) {
        return std::unexpected(
            Error::invalid_operation
        );
    }

    return value * 2;
}

int main() {
    auto result = parseValue("123")
        .and_then(multiplyByTwo);

    if (result) {
        int value = *result;
    }
}
```

This approach keeps the successful execution path in a chain while automatically propagating errors.

### **Understanding transform()**

Use `transform()` when you want to modify a successful value but the new operation cannot independently fail.

For example:

```
std::expected<int, Error> result = 10;

auto transformed = result.transform(
    [](int value) {
        return value * 2;
    }
);
```

The result is a new `std::expected<int, Error>`.

If the original object contains a successful value, the callback is executed and its result is stored in the new `expected`.

If the original object contains an error, that error is propagated unchanged.

The main difference between `transform()` and `and_then()` is the callback's return type:

- `transform()` expects the callback to return an ordinary value.
- `and_then()` expects the callback to return a `std::expected`.

This distinction is important when designing operation chains.

### **Practical transform() Example**

```
#include <expected>
#include <string>

enum class Error {
    invalid_input
};

std::expected<int, Error>
parseValue(const std::string& input) {
    if (input.empty()) {
        return std::unexpected(Error::invalid_input);
    }

    return 21;
}

int main() {
    auto result = parseValue("21")
        .transform([](int value) {
            return value * 2;
        });

    if (result) {
        int value = *result;
    }
}
```

In this example, string conversion can fail, but multiplying an integer by two is an ordinary successful operation.

Therefore, `transform()` is the appropriate choice.

### **Understanding or_else()**

The `or_else()` function executes when the object is in the Error State.

It can be used to recover from an error, perform a fallback operation, or provide an alternative execution path.

For example:

```
auto result = getConfiguration()
    .or_else([](const ConfigError& error) {
        return loadDefaultConfiguration();
    });
```

If retrieving the configuration succeeds, the callback is not executed.

If the operation fails, the fallback function is executed.

An important requirement is that the callback passed to `or_else()` must return a compatible `std::expected`. Its Value type must remain compatible with the original Value type.

The resulting Error type must also satisfy the type requirements of the resulting operation.

### **Understanding transform_error()**

The `transform_error()` function transforms the Error Value without changing the successful Value.

For example:

```
enum class InternalError {
    invalid_input,
    timeout
};

enum class PublicError {
    bad_request,
    service_unavailable
};

std::expected<int, InternalError> internalOperation();

std::expected<int, PublicError>
publicOperation() {
    return internalOperation()
        .transform_error(
            [](InternalError error) {
                switch (error) {
                    case InternalError::invalid_input:
                        return PublicError::bad_request;

                    case InternalError::timeout:
                        return PublicError::service_unavailable;
                }

                return PublicError::bad_request;
            }
        );
}
```

In this example, internal errors are converted into error types suitable for a public API.

This approach is particularly useful in layered architectures.

### **Comparison of Monadic Operations**

| Operation           | When the callback executes | Callback return type | Purpose                   |
| ------------------- | -------------------------- | -------------------- | ------------------------- |
| `and_then()`        | On success                 | `expected`           | Chain fallible operations |
| `transform()`       | On success                 | Ordinary value       | Transform the Value       |
| `or_else()`         | On failure                 | `expected`           | Recovery or fallback      |
| `transform_error()` | On failure                 | Error Value          | Transform the Error type  |

## **Handling Multiple Fallible Operations**

In real-world projects, an operation usually consists of several stages.

For example, processing a request might involve:

- Receiving information.
- Validating the input.
- Transforming the data.
- Performing the main operation.
- Saving the result.

If each stage can fail, combining `std::expected` with Monadic Operations can simplify the program's structure.

For example:

```
#include <expected>
#include <string>

enum class Error {
    invalid_input,
    invalid_format,
    processing_failed
};

std::expected<std::string, Error>
readInput();

std::expected<int, Error>
parseInput(const std::string& input);

std::expected<int, Error>
processValue(int value);

std::expected<std::string, Error>
formatResult(int value);

std::expected<std::string, Error>
execute() {
    return readInput()
        .and_then(parseInput)
        .and_then(processValue)
        .and_then(formatResult);
}
```

In this structure, each stage is responsible for its own operation.

If any stage fails, the subsequent stages are not executed, and the error is propagated through the chain.

This design reduces nesting and eliminates much of the repetitive code required for checking errors.

## **Chaining Operations with Different Error Types**

One important consideration when designing operation chains is that the error types must be identical or otherwise compatible.

Suppose the first function has the following return type:

```
std::expected<int, ParseError>
```

And the second function has this return type:

```
std::expected<double, DatabaseError>
```

These functions cannot be chained directly using `and_then()` because their error types are incompatible.

There are several ways to address this problem.

### **Approach One: Use a Shared Error Type**

A general error type can be defined for multiple subsystems.

```
enum class ApplicationError {
    parse_failed,
    database_failed,
    network_failed
};
```

Each function can then convert its specific errors into this shared type.

This approach is suitable for systems with a limited set of error categories.

### **Approach Two: Use std::variant**

If you want to preserve the original types of different errors, you can use `std::variant`.

```
#include <expected>
#include <variant>

using ApplicationError = std::variant<
    ParseError,
    DatabaseError,
    NetworkError
>;
```

Now, all functions can use a common error type while preserving the original error alternatives.

### **Approach Three: Convert Errors at Subsystem Boundaries**

Another approach is to keep internal error types independent and convert them only when crossing a subsystem boundary.

`transform_error()` is a suitable tool for this purpose.

This approach often preserves subsystem independence more effectively in large architectures.

## **Constructors and Assignment Operators**

`std::expected` provides several constructors for creating Value and Error states.

Understanding them is important for professional use.

### **Default Constructor**

When `T` is default-constructible, an `expected` object can be created without explicitly providing an initial value.

```
std::expected<int, std::string> result;
```

In this case, the default value of the `int` type is constructed.

Therefore, the result is a successful `expected` containing zero.

The default constructor is also available for `std::expected<void, E>` and creates a successful state.

### **Copy Constructor**

When the stored types satisfy the required copy-construction constraints, an `expected` object can be copied.

```
auto first = std::expected<int, std::string>{42};

auto second = first;
```

In this example, the successful value is copied.

If the original object is in the Error State, its Error Value is copied instead.

### **Move Constructor**

Move Construction can be used to transfer resources from one object to another.

```
auto first =
    std::expected<std::string, std::string>{
        "Hello"
    };

auto second = std::move(first);
```

When the stored types support Move Construction, this operation allows their resources to be transferred.

The source object remains a valid `expected` object. It does not automatically become empty.

The exact state of its stored value after the move depends on the move behavior of the corresponding type.

### **Using std::in_place**

As discussed earlier, this tag is used to construct the Value directly.

```
std::expected<std::string, int> result{
    std::in_place,
    5,
    'X'
};
```

### **Using std::unexpect**

This tag can be used to construct the Error Value directly.

```
std::expected<std::string, int> result{
    std::unexpect,
    404
};
```

### **Assignment Operators**

`std::expected` also supports assignment operations.

```
std::expected<int, std::string> result = 10;

result = 20;

result = std::unexpected(
    std::string{"Failed"}
);
```

In this example, the object initially contains a successful value, then its value is changed, and finally it transitions into the Error State.

The availability of assignment operations depends on the required capabilities of the stored types `T` and `E`.

## **Replacing the Value with emplace()**

The `emplace()` function replaces the existing Value with a newly constructed one.

```
std::expected<std::string, int> result = "Hello";

result.emplace(5, 'A');
```

The stored value is now the string `AAAAA`.

This function is useful when constructing a new Value directly inside the internal storage.

If the object is in the Error State, the error value is replaced and the new Value is constructed.

An important consideration is that if construction of the new Value throws an exception, the `expected` object must still remain in one of its two valid states: Value or Error.

In such situations, the exception-safety guarantees of the operation and the behavior of the stored types should be considered according to the standard specification.

## **Swapping States with swap()**

The `swap()` function allows the contents of two `expected` objects to be exchanged.

```
std::expected<int, std::string> first = 10;

std::expected<int, std::string> second =
    std::unexpected("Error");

first.swap(second);
```

After this operation:

- `first` contains the Error Value.
- `second` contains the value `10`.

This function is designed to exchange states and contents. Its availability depends on the Swappable requirements of the stored types `T` and `E`.

## **Using References and Pointers**

An important limitation of `std::expected` is that reference types cannot be used directly as `T` or `E`.

For example, the following declaration is invalid:

```
std::expected<int&, Error> result;
```

This is because `std::expected` is not designed to store references directly.

If a reference needs to be represented as a Value, `std::reference_wrapper` can be used.

```
#include <expected>
#include <functional>

int value = 42;

std::expected<std::reference_wrapper<int>, int>
getReference() {
    return std::ref(value);
}
```

In this example, the Value type is `std::reference_wrapper<int>`.

Keep in mind that `std::reference_wrapper` does not own the referenced object. The lifetime of the original object must still be managed by the program.

### **Using Pointers**

Pointers are valid Value types.

```
std::expected<int*, Error> result;
```

However, two different states must be distinguished:

- The `expected` object is in the Error State.
- The `expected` object is successful, but its stored pointer is `nullptr`.

These states are different from the perspective of the type system.

However, if `nullptr` already represents failure or the absence of a value, combining pointers with `expected` may introduce additional and potentially ambiguous states.

In such situations, it is worth reconsidering the API contract.

## **Combining std::expected and std::optional**

These two types can be used together.

For example, suppose a function can fail because of an error, but even when it succeeds, the requested data might not exist.

In this situation, the following return type can be used:

```
std::expected<std::optional<User>, DatabaseError>
findUser(UserId id);
```

This type represents three logical states:

- A database error occurred.
- The operation succeeded, but no user was found.
- The operation succeeded and a user was found.

This design is useful when the absence of a value has a different meaning from an operational failure.

However, when designing an API, consider whether all three states are actually necessary or whether a simpler contract would be sufficient.

## **Comparing std::expected with Exceptions**

Both approaches can communicate failures from a function to its caller, but they follow different design philosophies.

### **Exception-Based Error Handling**

With exception-based error handling, a function throws an exception when it fails.

```
int divide(int a, int b) {
    if (b == 0) {
        throw std::runtime_error(
            "Division by zero"
        );
    }

    return a / b;
}
```

The caller can use `try` and `catch` when necessary.

One advantage of this approach is that normal execution is separated from error-handling paths.

However, exception propagation is not normally visible in the function's signature.

### **Error Handling with std::expected**

With `std::expected`, the error is part of the function's return type.

```
std::expected<int, MathError>
divide(int a, int b);
```

The caller can determine from the signature that the function may fail.

### **Practical Comparison**

| Criterion                                | std::expected       | Exceptions               |
| ---------------------------------------- | ------------------- | ------------------------ |
| Errors visible in the function signature | Yes                 | Usually no               |
| Suitable for predictable failures        | Very suitable       | Also possible            |
| Requires checking the result             | Usually yes         | No                       |
| Error propagation through chains         | Yes                 | Through stack unwinding  |
| Requires designing an Error Type         | Yes                 | Depends on the design    |
| Cost on the successful path              | Usually predictable | Implementation-dependent |
| Suitable for exceptional failures        | Usually not         | Yes                      |

### **Is std::expected a Replacement for Exceptions?**

No.

These two mechanisms are designed for different scenarios.

`std::expected` is generally suitable for errors that:

- Are part of the normal behavior of the system.
- Are predictable.
- Allow the caller to make a specific decision.
- Should be represented in the API contract.

Exceptions are suitable for situations where:

- An operation cannot continue according to its contract.
- The current layer cannot handle the failure.
- The failure is genuinely exceptional.
- Using error return values would introduce unnecessary complexity.

Both approaches can also coexist in the same application.

For example, a function can report ordinary failures using `std::expected` while still allowing unexpected exceptions from internal operations to propagate.

An important point is that `std::expected` does not automatically convert exceptions thrown by internal functions into Error States.

## **Comparing std::expected with std::variant**

`std::variant` was introduced in C++17.

It allows an object to hold one of several specified types.

For example:

```
std::variant<int, std::string> result;
```

This type can contain either an integer or a string.

Structurally, `std::expected<T, E>` is similar to a two-alternative variant, but its semantics are more specific.

With `std::expected`:

- One type represents the successful result.
- The other type represents an error.
- Dedicated APIs are provided for error handling.
- Monadic Operations are available for success and failure flows.

In contrast, `std::variant` is designed for general-purpose representation of alternative types.

Therefore, if the goal is simply to store one of several independent types, `std::variant` is usually more appropriate.

However, if the goal is to represent the result of an operation that may fail, `std::expected` provides a more expressive abstraction.

## **Designing Professional APIs with std::expected**

Using `std::expected` correctly involves more than simply choosing it as a function's return type.

Error type design, subsystem boundaries, and responsibility for error handling are equally important.

### **Choosing an Appropriate Error Type**

The error type should contain enough information without becoming unnecessarily complicated.

For example, using a general string might be appropriate for small projects, but structured error types are usually better for public APIs.

```
struct NetworkError {
    int statusCode;
    std::string message;
};
```

### **Avoiding Leaks of Internal Implementation Details**

Suppose a library uses a specific database implementation internally.

If its public error type directly exposes the database's error types, unnecessary dependencies are introduced.

Internal errors should preferably be converted into independent error types at the library boundary.

### **Preserving Error Information**

Important information should not be lost when converting errors.

For example, if a database error contains an error code, query, and original message, converting it into a generic value such as `operation_failed` may discard information needed for debugging.

A general error type should preserve essential diagnostic information in addition to categorizing the failure.

### **Separating Logging from Error Handling**

Returning an Error Value does not automatically imply that the error should be logged.

In many architectures, a function should report the result, while a higher-level component decides whether logging is necessary.

This helps prevent the same error from being logged multiple times across different layers.

### **Designing Extensible Error Types**

When designing public libraries, remember that adding a new member to an `enum class` may affect consumers.

This is especially relevant when consumers use `switch` statements to handle error categories.

Therefore, the evolution policy for error types should be considered from the beginning.

## **Handling System Errors with std::error_code**

The C++ standard library provides facilities for handling system-related errors through the following header:

```
#include <system_error>
```

Two important types in this area are:

- `std::error_code`
- `std::error_condition`

These types are designed to represent standardized and system-specific errors.

They can be combined with `std::expected`.

```
#include <expected>
#include <system_error>

std::expected<int, std::error_code>
readData();
```

This design is suitable for APIs that interact with operating-system errors or low-level libraries.

An advantage of `std::error_code` is that it also identifies the associated error category.

Therefore, unlike a plain integer, it provides a way to identify the origin and meaning of an error.

## **constexpr Support**

One important feature of `std::expected` is its `constexpr` support across much of its API.

When the types `T` and `E` and the operations involved satisfy the relevant requirements, this allows `expected` operations to be evaluated at compile time.

For example:

```
#include <expected>

constexpr std::expected<int, int>
calculate(int value) {
    if (value < 0) {
        return std::unexpected(-1);
    }

    return value * 2;
}

constexpr auto result = calculate(10);

static_assert(result.has_value());
static_assert(*result == 20);
```

In this example, the result can be computed at compile time.

However, using `constexpr` depends on the restrictions associated with the stored types and the operations being performed.

## **Move Semantics Considerations**

`std::expected` supports Move Construction and Move Assignment.

This makes it suitable for storing non-copyable types or types that manage resources.

For example:

```
#include <expected>
#include <memory>

enum class Error {
    allocation_failed
};

std::expected<std::unique_ptr<int>, Error>
createValue() {
    return std::make_unique<int>(42);
}
```

In this example, the Value is of type `std::unique_ptr<int>`, which is not copyable.

Nevertheless, it can be stored and moved through `std::expected`.

Monadic operations also support different Value Categories.

In general, multiple overloads are available for Lvalues and Rvalues.

This allows them to pass the stored Value to callbacks using references or rvalue references when appropriate.

Therefore, when working with Move-Only types, it is important to understand the expression's value category and how the object is being transferred.

## **Reference Qualifiers and Value Categories**

Access operations and Monadic Operations in `std::expected` provide overloads for different `const` and Value Category combinations.

For example, `value()` has overloads for:

- Lvalues.
- Const Lvalues.
- Rvalues.
- Const Rvalues.

This allows correct use of Move Semantics.

For example:

```
std::expected<std::string, int> result = "Hello";

std::string value = std::move(result).value();
```

In this example, the Value is moved from the `expected` object into the new string.

However, remember that `std::move` does not perform a move by itself. It only converts the expression into an Rvalue, allowing an appropriate move constructor or assignment operator to perform the transfer.

Also, subsequent use of the moved-from object must take its post-move state into account.

## **Limitations and Type Requirements**

`std::expected` imposes several restrictions on the types `T` and `E`.

Understanding these restrictions is essential for avoiding compile-time errors.

### **No Direct Support for Reference Types**

Reference types cannot be used directly as either the Value or Error type.

```
std::expected<int&, Error> result;
```

This declaration is invalid.

A suitable wrapper must be used when storing a reference.

### **No Direct Support for Function Types**

Function types cannot be used directly as `T` or `E` either.

If necessary, use an appropriate pointer or wrapper type.

### **Restrictions on Certain Types**

The Value type `T` must not be one of the following types:

- `std::unexpected`
- `std::in_place_t`
- `std::unexpect_t`

Additionally, the types used must satisfy the relevant Object Type and Destructibility requirements.

### **Constructor and Assignment Requirements**

Not every operation is available for every type.

For example:

- The Copy Constructor depends on copy-construction capabilities.
- The Move Constructor depends on move-construction capabilities.
- Assignment depends on the assignment capabilities of the stored types.
- `emplace()` depends on the ability to construct a new Value.
- `swap()` depends on the Swappable requirements of the stored types.

For this reason, when working with complex custom types, the requirements of each operation should be examined carefully.

## **Common Mistakes When Using std::expected**

### **Accessing the Value Without Checking the State**

One of the most common mistakes is dereferencing an `expected` object without checking whether it contains a value.

The following code is unsafe:

```
std::expected<int, Error> result =
    std::unexpected(Error::invalid_input);

int value = *result;
```

In this situation, the program has undefined behavior.

The correct approach is:

```
if (result) {
    int value = *result;
}
```

### **Using value() in Every Situation**

Calling `value()` in the Error State throws an exception.

If the goal is to handle errors without exceptions, check the state first or use Monadic Operations.

### **Ignoring the Error Value**

Sometimes developers check only whether an operation succeeded and ignore the actual error information.

For example:

```
if (!result) {
    return;
}
```

This code might be logically valid, but in many applications, it discards important error information.

When necessary, the Error Value should be propagated to the appropriate layer.

### **Using Ambiguous Error Types**

Using types such as `int` or `std::string` for every error may initially seem simple.

However, as a project grows, identifying specific error categories becomes increasingly difficult.

Dedicated error types are generally a better choice.

### **Incorrectly Combining and_then() and transform()**

These operations have different behaviors.

If a callback returns `std::expected`, `and_then()` is generally the appropriate choice.

If a callback returns an ordinary value, `transform()` is appropriate.

Using them incorrectly can introduce nested structures or type-compatibility errors.

### **Using value_or() for Important Errors**

The `value_or()` function converts an error into a default value.

When a failure has operational significance, this may hide problems.

For example, returning zero when a calculation fails may produce an apparently valid but incorrect result.

### **Assuming Ownership of External Resources**

Using `std::expected` does not mean that all resources referenced by its Value or Error are automatically managed.

If the Value contains a pointer or a reference wrapper, the lifetime of the external resource must still be managed.

## **Performance Considerations**

In terms of performance, `std::expected` is generally a suitable choice for explicit error handling.

However, it should not be assumed to be completely free of overhead in every situation.

### **Memory Overhead**

Each `expected` object must have enough storage for one of the two types, `T` and `E`, along with information identifying the active state.

Consequently, its size is generally influenced by the larger stored type and the relevant alignment requirements.

### **Copy and Move Costs**

If `T` or `E` are large objects, copying an `expected` may be expensive.

In such situations, Move Semantics should be considered.

Using appropriately designed, lightweight error types can also help.

### **Error Handling Costs**

On the successful path, the operation generally involves constructing the Value and maintaining the associated state.

On the failure path, the Error Value is constructed or transferred.

This design can be suitable for applications where failure is a normal part of operation.

However, actual performance depends on the compiler, stored types, optimization settings, and usage patterns.

### **No Guaranteed Zero-Cost Behavior**

Using `std::expected` does not guarantee that it will always be faster than exceptions.

Performance comparisons should be based on realistic scenarios and benchmarks.

## **Compatibility with Different C++ Standard Versions**

`std::expected` is part of the C++23 standard library.

Using it requires both the compiler and the standard library implementation to support the feature.

Enabling C++23 does not automatically guarantee that every standard library feature is available in the selected environment.

### **Checking the Feature Test Macro**

The following macro can be used to check whether the feature is available:

```
#include <expected>
#include <version>

#if defined(__cpp_lib_expected)

#if __cpp_lib_expected >= 202202L
#define HAS_EXPECTED 1
#else
#define HAS_EXPECTED 0
#endif

#else
#define HAS_EXPECTED 0
#endif
```

The original Feature Test Macro value for `std::expected` is:

```
202202L
```

The Monadic Operations have a separate feature test value:

```
202211L
```

Therefore, if your code requires Monadic Operations, check the macro value separately.

```
#include <expected>
#include <version>

#if defined(__cpp_lib_expected) && \
    __cpp_lib_expected >= 202211L

#define HAS_EXPECTED_MONADIC 1

#else

#define HAS_EXPECTED_MONADIC 0

#endif
```

This approach is useful for libraries that need to support multiple toolchains.

### **Enabling C++23**

For GCC and Clang, the following option is commonly used:

```
-std=c++23
```

For Microsoft Visual C++, the following option can be used:

```
/std:c++latest
```

However, actual support depends on the compiler and standard library versions.

Some versions may require a newer toolchain.

Additionally, some implementations may provide certain standard library features earlier or later than others.

Therefore, actual compatibility should be verified by consulting the documentation for the specific version and checking the relevant Feature Test Macro.

## **Practical Applications of std::expected**

### **File I/O**

File operations commonly encounter errors such as missing files, permission failures, and read failures.

The result of such an operation can be represented as follows:

```
#include <expected>
#include <string>

enum class FileError {
    not_found,
    permission_denied,
    read_failed
};

std::expected<std::string, FileError>
readFile(const std::string& path);
```

This design allows callers to respond differently depending on the error category.

### **Database Operations**

Database operations such as retrieving, inserting, and deleting information can fail.

```
enum class DatabaseError {
    connection_failed,
    query_failed,
    transaction_failed
};

std::expected<User, DatabaseError>
loadUser(int userId);
```

This approach is particularly useful when failures are a predictable part of the system's behavior.

### **Network Programming**

Network applications commonly encounter errors such as timeouts, connection refusals, and protocol errors.

```
enum class NetworkError {
    timeout,
    connection_refused,
    protocol_error
};

std::expected<Response, NetworkError>
sendRequest(const Request& request);
```

This design allows different error categories to be handled by higher-level components.

### **Parser Implementations**

Parsers commonly encounter several types of errors.

For example:

- Invalid tokens.
- Incomplete structures.
- Unexpected end of input.
- Syntax errors.

A structured error type can provide both the error category and its location.

```
struct ParserError {
    std::size_t line;
    std::size_t column;
    std::string message;
};

std::expected<AstNode, ParserError>
parseSource(std::string_view source);
```

This design is particularly suitable for compilers, interpreters, and language-processing tools.

### **Configuration Management**

When loading configuration settings, invalid values, unknown keys, or incomplete files may cause failures.

```
enum class ConfigError {
    file_not_found,
    invalid_value,
    missing_required_key
};

std::expected<Config, ConfigError>
loadConfiguration();
```

This approach makes it possible to distinguish between different configuration errors.

### **Validation**

When validating data, several conditions may need to be checked, and each may have its own error category.

```
enum class ValidationError {
    invalid_email,
    weak_password,
    invalid_age
};

std::expected<User, ValidationError>
validateUser(const UserInput& input);
```

With this design, the function communicates both success or failure and the specific reason for failure.

## **Functional Programming with std::expected**

The Monadic Operations in `std::expected` make it possible to write code using a Functional Programming style.

Instead of manually controlling every stage, different operations can be composed through small, independent functions.

For example:

```
auto result = parseInput(input)
    .and_then(validateInput)
    .and_then(processInput)
    .transform(formatOutput)
    .transform_error(mapError);
```

In this structure:

- Each stage is responsible for a specific operation.
- Failures are propagated automatically.
- Successful operations are executed sequentially.
- Value and Error transformations remain independent.

However, excessively long chains can reduce readability.

In such situations, splitting the operations into several smaller functions may be preferable.

## **Thread Safety Considerations**

`std::expected` is not inherently thread-safe.

If multiple threads access the same shared object and at least one of them modifies its value, a data race may occur.

For example, sharing a mutable `expected` object between threads requires appropriate synchronization mechanisms.

Additionally, if the Value or Error contains shared resources, the safety of accessing those resources depends on the design of their respective types.

Therefore, using `std::expected` does not automatically provide thread safety.

## **Important Considerations for Public APIs**

When designing public libraries based on `std::expected`, several considerations are particularly important:

- The Error type should be an intentional part of the public API contract.
- Changing the Error type may introduce source incompatibility.
- Adding new error categories should be done with consumers in mind.
- Internal implementation details should not leak into public APIs unnecessarily.
- Documentation should specify the conditions under which each Error Value can occur.
- It should be clear which errors are recoverable.
- It should be clear which errors are propagated to higher layers.

Return-type compatibility is particularly important when designing stable APIs.

## **Comparing std::expected with Result in Other Languages**

The design pattern behind `std::expected` is not unique to C++.

Rust provides a type called `Result<T, E>`, which is conceptually very similar to `std::expected`.

Functional programming languages also use concepts such as `Either` to represent one of two alternative states.

The common principle behind these designs is explicitly representing successful results and failures in the type system.

However, the syntax, memory ownership rules, error propagation mechanisms, and language capabilities differ.

Therefore, conceptual similarity should not be interpreted as identical behavior.

## **Conclusion**

`std::expected` is an important addition to the C++23 standard library that provides explicit, type-safe, and composable error handling.

It allows developers to represent successful results and error information within a single well-defined type.

Its most important capabilities include:

- Storing a Value and an Error in a single type.
- Defining custom error types.
- Accessing results through `value()`, `error()`, and the relevant operators.
- Supporting `std::expected<void, E>`.
- Directly constructing values and errors using `std::in_place` and `std::unexpect`.
- Supporting Move Semantics.
- Supporting `constexpr`.
- Composing operations using `and_then()` and `transform()`.
- Recovering from or transforming errors using `or_else()` and `transform_error()`.
- Supporting the design of professional, layered APIs.

The most important consideration when using `std::expected` correctly is choosing it for failures that are part of an operation's contract and predictable behavior.

It is not a universal replacement for exceptions, `std::optional`, or `std::variant`. Instead, it is a specialized tool for representing the result of an operation that may fail.

In modern C++ projects, using `std::expected` appropriately can make APIs clearer, reduce reliance on exceptions for ordinary failures, improve testability, and simplify error propagation throughout an application.

## **References**

- [cppreference — std::expected](https://en.cppreference.com/w/cpp/utility/expected?utm_source=chatgpt.com)
- [cppreference — std::unexpected](https://en.cppreference.com/w/cpp/utility/expected/unexpected?utm_source=chatgpt.com)
- [cppreference — std::bad_expected_access](https://en.cppreference.com/w/cpp/utility/expected/bad_expected_access?utm_source=chatgpt.com)
- [WG21 P0323R12 — std::expected](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0323r12.html?utm_source=chatgpt.com)
- [WG21 P2505R5 — Monadic Functions for std::expected](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2505r5.html?utm_source=chatgpt.com)
- [WG21 P2549R1 — std::unexpected should have error() as member accessor](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2549r1.html?utm_source=chatgpt.com)
- [C++23 Standard Library Features](https://en.cppreference.com/w/cpp/23?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>