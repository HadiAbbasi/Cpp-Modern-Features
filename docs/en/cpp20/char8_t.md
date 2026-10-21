<div align="center">

[🇺🇸 English](./char8_t.md) | [🇮🇷 فارسی](../../fa/cpp20/char8_t.md)

</div>

---

# Introduction to `char8_t` in C++20

The `char8_t` type was introduced in C++20 to represent code units in UTF-8 encoded text. It distinguishes UTF-8 text data from general `char` data at the language's type-system level, allowing developers to manage UTF-8 text more explicitly in APIs, libraries, and data structures.

## The Purpose of `char8_t`

Before C++20, programmers commonly used `char` and `std::string` to store UTF-8 data. Although this approach was widespread, the type system itself did not indicate whether a string contained UTF-8 text, text encoded using another encoding, or binary data.

For example, a string containing ASCII text is not distinguishable by type from a string containing text with another encoding:

```
#include <string>

int main() {
    std::string text = "Hello";
}
```

In this example, `std::string` simply stores a sequence of `char` objects and provides no guarantee about the text's encoding.

C++20 introduced the distinct `char8_t` type to separate UTF-8 code units from the general-purpose `char` type. This distinction allows the compiler to detect certain accidental combinations of incompatible text types at compile time.

```
#include <string>

int main() {
    std::u8string text = u8"Hello";
}
```

In this example, the `u8` prefix identifies the string literal as UTF-8 encoded, and `std::u8string` stores its code units.

## Type Definition and Language Characteristics

Since C++20, `char8_t` has been a distinct unsigned integer type designed to store a single UTF-8 code unit. It is a separate fundamental type, not merely an alias for `char`.

Its important characteristics include:

- Introduction: The type is available starting with C++20.
- Size: `char8_t` occupies exactly one C++ byte, meaning `sizeof(char8_t) == 1`.
- Distinct type: It is not the same type as `char`, and there is no implicit conversion between them.
- Primary purpose: Representing UTF-8 code units, not complete Unicode characters.
- Arrays and strings: It can be used to construct character arrays, pointers, and standard string types.

The `char8_t` type is also distinct from `unsigned char`, although both can be useful when working with byte-sized values. The appropriate choice depends on the meaning of the data and the API's contract.

## UTF-8 String Literals

In C++, the `u8` prefix is used to define UTF-8 string literals. The type of these literals underwent an important change in C++20.

```
auto text = u8"Hello";
```

In this example, the type of `text` is `const char8_t*`. A UTF-8 string literal is an array of `const char8_t`, which ordinarily decays to a pointer to its first element when used in an expression.

### Differences Between C++ Standard Versions

| Standard        | Element type of `u8` string literals |
| --------------- | ------------------------------------ |
| C++11 and C++14 | `char`                               |
| C++17           | `char`                               |
| C++20 and later | `char8_t`                            |

This change is important for backward compatibility. Code that used `u8"Hello"` as a `const char*` in C++17 may no longer compile when migrated to C++20.

### Different String Literal Types

C++ provides several prefixes for different character encodings:

| Prefix    | Element type in C++20 | Purpose                                                               |
| --------- | --------------------- | --------------------------------------------------------------------- |
| No prefix | `char`                | Ordinary string literals                                              |
| `u8`      | `char8_t`             | UTF-8                                                                 |
| `u`       | `char16_t`            | UTF-16                                                                |
| `U`       | `char32_t`            | UTF-32                                                                |
| `L`       | `wchar_t`             | Wide-character type, whose representation is implementation-dependent |

Importantly, the `u8` prefix alone does not guarantee that the source file uses the correct encoding or that incoming data is valid UTF-8. Source-file encoding and the interpretation of its characters are also governed by the standard's rules and implementation settings.

## `std::u8string` and Related Types

C++20 provides standard string types corresponding to `char8_t`.

| Type                              | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| `std::u8string`                   | A string containing `char8_t` elements                       |
| `std::u8string_view`              | A non-owning view of a sequence of `char8_t` elements        |
| `std::basic_string<char8_t>`      | The generic string template instantiated with `char8_t`      |
| `std::basic_string_view<char8_t>` | The generic string-view template instantiated with `char8_t` |

These types are available through the standard `<string>` and `<string_view>` headers.

```
#include <iostream>
#include <string>
#include <string_view>

int main() {
    std::u8string text = u8"Hello";
    std::u8string_view view = text;

    std::cout << "Code units: " << view.size() << '\n';
}
```

In this example, `text` owns the memory, whereas `view` provides a non-owning view of its data. The string's size is measured in `char8_t` elements, not in Unicode characters.

Also, `std::cout` prints only the fixed ASCII text in this example. Ordinary `char` output streams do not directly support printing a `char8_t` string in the same way.

### String Length Versus Character Count

For UTF-8 strings, `size()` returns the number of code units. This value is not necessarily equal to the number of Unicode code points or user-perceived characters.

```
#include <iostream>
#include <string>

int main() {
    std::u8string text = u8"é";

    std::cout << text.size() << '\n';
}
```

In this example, assuming the source literal contains the Unicode character `U+00E9`, its UTF-8 encoding consists of two code units. Therefore, `size()` returns `2`.

However, visually similar text can also be represented using a combination of `e` and a combining accent mark. In that case, the number of code units differs even though the text may look identical.

Counting Unicode code points or user-perceived characters requires Unicode-aware algorithms; `std::u8string` does not provide this functionality by itself.

## Type Conversions and Compatibility with Legacy APIs

One of the most important differences between `char8_t` and `char` is that there is no implicit conversion between these types.

The following code is invalid in C++20:

```
#include <string>

int main() {
    std::string text = u8"Hello";
}
```

The reason is that `u8"Hello"` is an array of `const char8_t`, whereas `std::string` stores `char` elements.

To resolve this issue, determine the actual type of data expected by the API.

### Explicit Conversion for Byte-Oriented APIs

If a legacy API only needs a sequence of bytes, the byte representation of UTF-8 text can be passed to it. This is appropriate when the API consumes raw bytes and does not perform unintended encoding conversions.

```
#include <cstddef>
#include <iostream>
#include <string>

void writeBytes(const char* data, std::size_t size) {
    std::cout.write(data, static_cast<std::streamsize>(size));
}

int main() {
    std::u8string text = u8"Hello";

    writeBytes(
        reinterpret_cast<const char*>(text.data()),
        text.size()
    );
}
```

In this example, `reinterpret_cast` changes the pointer type; it does not convert the string's contents to another encoding. The size is passed explicitly so that the consumer does not depend on a terminating `'\0'` character.

This approach is appropriate when the API's contract is byte-oriented. If the API expects text in another specific encoding, converting the pointer type is insufficient; the encoding must be converted correctly.

### Important Considerations When Converting Pointers

Using `reinterpret_cast` does not perform a real conversion from UTF-8 to another encoding. This technique should not be used to conceal a semantic mismatch between APIs.

When designing a new API, make the expected data type explicit. If the API operates on UTF-8, `std::u8string_view` or `std::u8string` can provide a clearer contract.

## Interaction with `std::string_view`

The `std::string_view` type provides a non-owning view of strings containing `char` elements. It is not a direct substitute for `std::u8string_view`.

```
#include <string_view>

int main() {
    std::u8string_view utf8 = u8"Hello";

    // std::string_view text = utf8;
}
```

The commented-out line cannot be enabled directly because the element types of the two views differ. An explicit conversion or the construction of a compatible view of another type would be required.

For UTF-8 APIs, the following function signature is more appropriate:

```
#include <string_view>

void processUtf8(std::u8string_view text) {
    // Process UTF-8 code units.
}

int main() {
    processUtf8(u8"Hello");
}
```

At the type level, this API specifies that its input consists of `char8_t` code units. However, validating the actual UTF-8 encoding still requires examining the input contents.

Because `std::u8string_view` does not own memory, the referenced data must remain alive for as long as the view is used. Retaining a pointer-like view after the lifetime of the underlying string ends can result in undefined behavior.

## UTF-8 Character Literals

In C++20, the `u8` prefix also applies to character literals and is associated with `char8_t`.

```
#include <type_traits>

int main() {
    constexpr char8_t ch = u8'A';

    static_assert(std::is_same_v<decltype(ch), const char8_t>);
}
```

In this example, `u8'A'` is a UTF-8 character literal whose type is `char8_t`.

A UTF-8 character literal is designed to represent a single code unit. Therefore, you should not assume that every Unicode character can be represented by a single `u8'...'` character literal, regardless of how many UTF-8 code units it requires.

For text requiring multiple code units, use a UTF-8 string literal such as `u8"..."`.

## Applications in Modern API Design

One of the most important applications of `char8_t` is establishing a clear distinction between UTF-8 text and other kinds of data.

Suppose a library provides a function for processing text. If the function accepts `std::string_view`, the input type alone does not indicate whether the data is UTF-8.

```
#include <string_view>

void processText(std::string_view text) {
    // The encoding is defined by the API contract.
}
```

In this case, the API contract must document the expected encoding.

By contrast, using a dedicated UTF-8 type also expresses the contract in the function signature:

```
#include <string_view>

void processUtf8(std::u8string_view text) {
    // The input is intended to contain UTF-8 code units.
}
```

This design can help prevent common mistakes, particularly in projects that work with UTF-8 text, binary data, and strings based on `char` at the same time.

However, the type alone does not guarantee that the encoding is valid. If data comes from a network, file, or untrusted source, validation must be part of the processing workflow.

## Applications in Files and Networks

UTF-8 data commonly comes from files, web APIs, databases, and network protocols. In these contexts, distinguishing text from raw bytes is particularly important.

When data arrives as a byte sequence, the source's contract must establish whether it is actually UTF-8. The data can then be transferred into an appropriate representation.

For example, if an internal API only stores byte sequences, `std::vector<std::byte>` may be appropriate. If the data is processed as UTF-8 text, `std::u8string` can express its meaning more clearly.

These types serve different purposes and should not be treated as interchangeable merely because both can represent byte-oriented data.

### Important Data Integrity Considerations

- The `char8_t` type does not guarantee valid UTF-8.
- A `'\0'` character within the data can cause some null-terminated string APIs to stop processing prematurely.
- UTF-8 text can contain multi-unit characters, so splitting a string at arbitrary byte boundaries can corrupt its encoding.
- Validation of external input should be performed at the appropriate layer of the application.
- When a protocol operates on raw bytes, the data contract should be defined independently of the text contract.

## Interaction with `std::filesystem`

In C++20, the interaction between `char8_t` and the filesystem library is also important. The `std::filesystem::path` type is designed independently of the string type used to construct it, but the encoding of input and output strings still requires careful handling.

For example, in C++20, `u8string()` returns a `std::u8string`:

```
#include <filesystem>
#include <string>

int main() {
    std::filesystem::path path = "example.txt";

    std::u8string utf8Path = path.u8string();
}
```

This behavior differs from earlier versions of the standard. Before C++20, `u8string()` returned `std::string`.

Consequently, code that previously assigned the result of `u8string()` directly to `std::string` may require changes when migrating to C++20.

It is also important to recognize that filesystem path representations and the encodings used by operating systems can differ between implementations. You should not assume that every operating system represents filesystem paths using UTF-8 merely because a C++ string type is involved.

## Comparing `char8_t` with `wchar_t`

The `wchar_t` type is designed for wide characters, and its size and encoding depend on the implementation. By contrast, `char8_t` was introduced in C++20 specifically for UTF-8 code units.

These types serve different purposes:

- `char8_t` is designed for UTF-8 data.
- `wchar_t` is associated with the implementation's wide-character facilities.
- Using `wchar_t` does not, by itself, guarantee that text is stored as UTF-16 or UTF-32.
- Converting between `char8_t` and `wchar_t` may require an actual encoding conversion.

For this reason, `char8_t` is a logical choice when the data contract explicitly specifies UTF-8. The choice of `wchar_t`, on the other hand, should depend on the requirements of the API and runtime environment.

## Limitations and Common Mistakes

### Assuming String Types Are Interchangeable

`std::u8string` is not the same type as `std::string`. This distinction in C++20 is intentional and should not be bypassed through implicit conversions or superficial changes to the apparent type.

### Assuming Code Unit Count Equals Character Count

The `size()` function on `std::u8string` returns the number of code units. Counting code points or grapheme clusters requires appropriate Unicode-aware algorithms.

### Assuming UTF-8 Is Automatically Valid

Having a `char8_t` type or a `std::u8string` does not guarantee that all contained data is valid. Invalid data can still be introduced through external input or byte-level operations.

### Using Pointer Conversion Instead of Encoding Conversion

Converting a `char8_t*` to `char*` does not convert the contents to another encoding. Such conversions are justifiable only in specific scenarios involving byte-oriented APIs.

### Ignoring the Differences Between C++17 and C++20

One common migration problem is the change in the type of `u8` string literals from `char` to `char8_t`. This change can affect overload resolution, string assignments, and function signatures.

### Ignoring Null-Termination Assumptions

C-style strings commonly use `'\0'` as a terminator, but byte sequences can also contain zero-valued bytes internally. To transfer data safely, its length should be passed explicitly.

## Compatibility and Migration to C++20

When migrating a project from C++17 to C++20, review every location that uses `u8` literals.

The following areas are particularly likely to require attention:

- Functions that accept `const char*` parameters.
- Variables that store `u8` literal values in `std::string`.
- Overloads that distinguish between `char` and `char8_t`.
- APIs that pass strings to external libraries.
- Code that obtains filesystem paths through `std::filesystem::path::u8string()`.

The compiler must support C++20 and `char8_t` to use these features. The appropriate language standard must also be enabled in the project's build configuration.

For example, GCC and Clang generally support the following option:

```
g++ -std=c++20 main.cpp -o main
```

In MSVC, the language standard can be enabled using the appropriate compiler option:

```
cl /std:c++20 /EHsc main.cpp
```

However, complete support for the standard library and related features depends on the compiler version and development environment.

## Summary

The `char8_t` type was introduced in C++20 to represent UTF-8 code units and distinguish them from general-purpose `char` data. By establishing a clear boundary in the language's type system, it makes text API design more explicit and can prevent certain mistakes caused by unintentionally mixing incompatible data types.

In modern projects, `std::u8string` and `std::u8string_view` are appropriate choices for data whose contract specifies UTF-8. However, differences in type behavior must be considered when interacting with legacy APIs, processing raw bytes, validating encodings, and handling filesystem paths.

The most important point is that `char8_t` makes the data type explicit, but it does not, by itself, guarantee valid UTF-8, determine the number of characters in a string, or ensure compatibility with every external API.


---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>