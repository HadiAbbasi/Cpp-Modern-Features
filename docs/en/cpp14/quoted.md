<div align="center">

[🇺🇸 English](./quoted.md) | [🇮🇷 فارسی](../../fa/cpp14/quoted.md)

</div>

---

# A Comprehensive Guide to `std::quoted` in C++14

## Introduction to `std::quoted`

Introduced in C++14, `std::quoted` is a standard library feature for formatting strings during input and output operations. It allows a string to be represented with a specified delimiter at the beginning and end, while escaping occurrences of the delimiter and a designated escape character when necessary.

Its primary purpose is to simplify reading and writing strings that may contain spaces, quotation marks, or other characters that require special handling in a particular output format.

For example, when a string contains multiple words, the ordinary `operator<<` writes it without quotation marks. Using `std::quoted`, however, makes the string's boundaries explicit.

```
#include <iomanip>
#include <iostream>
#include <string>

int main() {
    std::string text = "Hello World";

    std::cout << text << '\n';
    std::cout << std::quoted(text) << '\n';
}
```

The output is:

```
Hello World
"Hello World"
```

In this example, `std::quoted` produces a proxy object that provides special behavior when used with input and output streams.

Importantly, `std::quoted` is a standard library feature introduced in C++14. It is neither a language keyword nor a compiler extension.

## Why `std::quoted` Was Introduced

In many applications, strings must be stored or transmitted in a format that allows them to be reconstructed correctly later. For example, a program might need to store a string in a text file or display it in a diagnostic report.

The ordinary approach to printing a string is straightforward:

```
#include <iostream>
#include <string>

int main() {
    std::string text = "Alice Smith";

    std::cout << text << '\n';
}
```

Output:

```
Alice Smith
```

This output is suitable for displaying text to a user. However, when multiple strings are stored in the same line, identifying their boundaries is not always straightforward. In formats that use whitespace as a separator, the output may become ambiguous.

Consider the following two values:

```
Alice Smith
Bob Jones
```

Without a clearly defined format, it may be unclear whether each line contains one string or multiple separate values.

Using `std::quoted` makes the boundaries more explicit:

```
#include <iomanip>
#include <iostream>
#include <string>

int main() {
    std::string first = "Alice Smith";
    std::string second = "Bob Jones";

    std::cout << std::quoted(first) << '\n';
    std::cout << std::quoted(second) << '\n';
}
```

Output:

```
"Alice Smith"
"Bob Jones"
```

This approach is useful for simple text formats, basic configuration files, and diagnostic tools. However, using `std::quoted` does not automatically make the output a standard data format such as JSON.

## Required Header

To use `std::quoted`, include the `<iomanip>` header.

```
#include <iomanip>
```

The `<iomanip>` header provides facilities for controlling input and output formatting. The `std::quoted` function is defined in this header.

Here is a complete example:

```
#include <iomanip>
#include <iostream>
#include <string>

int main() {
    const std::string message = "C++ streams";

    std::cout << std::quoted(message) << '\n';
}
```

In this program, `<string>` provides `std::string`, while `<iostream>` provides access to `std::cout`.

## Function Signatures and Parameters

In C++14, `std::quoted` provides multiple overloads. Its two primary use cases are formatting strings for output and extracting strings from input.

The main overloads can be represented conceptually as follows:

```
std::quoted(const charT* s, charT delim = charT('"'),
            charT escape = charT('\\'));

std::quoted(const basic_string<charT, traits, Allocator>& s,
            charT delim = charT('"'),
            charT escape = charT('\\'));

std::quoted(basic_string<charT, traits, Allocator>& s,
            charT delim = charT('"'),
            charT escape = charT('\\'));
```

These declarations illustrate the general API and its parameters. The standard's exact declarations use proxy types as return types and include additional template details, so the snippets above should not be treated as verbatim declarations of every standard overload.

The parameters are:

- `s`: The string to format or extract.
- `delim`: The delimiter placed at the beginning and end of the string. Its default value is `"`.
- `escape`: The character used to escape occurrences of `delim` and `escape` itself. Its default value is `\`.

For example, the delimiter can be changed to another character:

```
#include <iomanip>
#include <iostream>
#include <string>

int main() {
    std::string text = "Hello";

    std::cout << std::quoted(text, '\'') << '\n';
}
```

Output:

```
'Hello'
```

In this example, a single quotation mark is used as the delimiter.

## Understanding the Proxy Object

The `std::quoted` function generally does not modify the original string. Instead, it returns a proxy object that stores the information needed to format the string.

This proxy behaves differently depending on how it is used with a stream:

- With `operator<<`, it writes the string with delimiters and the necessary escape characters.
- With `operator>>`, it extracts a string according to the special rules of `std::quoted`.

For example:

```
#include <iomanip>
#include <iostream>
#include <string>

int main() {
    std::string text = "She said \"Hello\"";

    std::cout << std::quoted(text) << '\n';
}
```

Output:

```
"She said \"Hello\""
```

Here, the quotation marks inside the string are escaped with the `\` character.

An important distinction is that this behavior depends on the input and output overloads. The proxy does not convert the string into an independent data type intended for persistent storage.

## Output Behavior with `operator<<`

The primary use of `std::quoted` is formatting strings when writing them to a stream.

Ordinary string output writes only the string's contents:

```
#include <iostream>
#include <string>

int main() {
    std::string text = "one two";

    std::cout << text << '\n';
}
```

Output:

```
one two
```

By contrast, `std::quoted` writes delimiters around the string and escapes the relevant characters:

```
#include <iomanip>
#include <iostream>
#include <string>

int main() {
    std::string text = "one \"two\" three";

    std::cout << std::quoted(text) << '\n';
}
```

Output:

```
"one \"two\" three"
```

The output behavior can be summarized as follows:

- An occurrence of `delim` inside the string is prefixed with `escape`.
- An occurrence of `escape` itself is also prefixed with `escape`.
- The delimiter `delim` is written at the beginning and end of the output.

An important limitation is that `std::quoted` does not convert every control character into a printable textual representation. For example, you should not expect newline and tab characters to be automatically converted into the textual escape sequences `\n` and `\t`.

## Input Behavior with `operator>>`

The `std::quoted` facility can also read strings enclosed in delimiters.

When extracting from a stream, if the next character matches `delim`, input processing enters quoted-string mode. The opening delimiter is consumed, and the string is read until the closing delimiter is encountered.

If a character matching `escape` is encountered, the following character is treated as part of the string, provided that another character is available in the stream.

For example:

```
#include <iomanip>
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::istringstream input("\"Hello World\"");

    std::string text;
    input >> std::quoted(text);

    std::cout << text << '\n';
}
```

Output:

```
Hello World
```

Unlike ordinary extraction with `operator>>`, this operation preserves whitespace inside the delimiters.

With ordinary string extraction, whitespace usually marks the end of the extracted string:

```
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::istringstream input("Hello World");

    std::string text;
    input >> text;

    std::cout << text << '\n';
}
```

Output:

```
Hello
```

Using `std::quoted` allows a string containing multiple words to be extracted in a single operation.

## Input Behavior for Unquoted Strings

If the first input character does not match `delim`, `std::quoted` does not process the input as a quoted string. Instead, extraction proceeds like ordinary string extraction, stopping when a character classified as whitespace by `std::isspace` in the stream's locale is encountered.

For example:

```
#include <iomanip>
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::istringstream input("Hello World");

    std::string text;
    input >> std::quoted(text);

    std::cout << text << '\n';
}
```

Output:

```
Hello
```

Therefore, `std::quoted` does not guarantee that an unquoted input string will be extracted in its entirety.

When processing input that may contain whitespace, the data format must define a clear convention, and the sender and receiver must agree on how delimiters are used.

## Understanding Escape Characters

Escape handling is one of the most important details of `std::quoted`.

By default, the delimiter is `"` and the escape character is `\`. Consequently, two cases must be handled when formatting the output:

- An embedded quotation mark must be escaped.
- A backslash must also be escaped.

For example:

```
#include <iomanip>
#include <iostream>
#include <string>

int main() {
    std::string text = "Path: C:\\temp\\file.txt";

    std::cout << std::quoted(text) << '\n';
}
```

Output:

```
"Path: C:\\temp\\file.txt"
```

In this example, the string stored in memory contains one backslash at each intended position. During formatting, each backslash is represented by two backslashes.

The distinction between a string's actual contents and its textual representation is important. Escape sequences in C++ source code are governed by the language's string-literal rules, whereas the escape sequences produced by `std::quoted` are part of stream formatting.

## Custom Delimiters and Escape Characters

One useful feature of `std::quoted` is the ability to specify a custom delimiter and escape character.

```
#include <iomanip>
#include <iostream>
#include <string>

int main() {
    std::string text = "It's a 'quoted' value";

    std::cout << std::quoted(text, '\'', '\\') << '\n';
}
```

Output:

```
'It\'s a \'quoted\' value'
```

In this example, a single quotation mark serves as the delimiter, and a backslash is used to escape it.

The choice of delimiter should take the string's contents into account. If the delimiter appears frequently in the text, the output may contain many escape characters. This does not inherently make the formatting incorrect, but it can reduce readability.

The input and output conventions must also match. If a string is written using a custom delimiter, the same delimiter and escape character should be supplied when reading it back.

## Reading and Writing with a Shared Convention

When storing and restoring strings, the output format must be compatible with the input format.

The following example writes a string to a `std::stringstream` and then extracts it:

```
#include <iomanip>
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::stringstream stream;

    std::string original = "A string with spaces and \"quotes\"";
    stream << std::quoted(original);

    std::string restored;
    stream >> std::quoted(restored);

    std::cout << std::boolalpha
              << (original == restored) << '\n';
}
```

Output:

```
true
```

This example demonstrates that `std::quoted` formatting and extraction are compatible for this string.

However, this facility alone does not define a complete data-storage protocol. For example, if several fields are written consecutively, the format must also specify field separators and how invalid input is handled.

## Handling Multiple Strings in a Stream

`std::quoted` can be combined with other input and output operations.

```
#include <iomanip>
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::stringstream stream;

    stream << std::quoted(std::string("Alice Smith"))
           << ' '
           << std::quoted(std::string("Developer"));

    std::string name;
    std::string role;

    stream >> std::quoted(name) >> std::quoted(role);

    std::cout << name << '\n';
    std::cout << role << '\n';
}
```

Output:

```
Alice Smith
Developer
```

In this example, the space between the two values is part of the data format, and each string is extracted according to the rules of `std::quoted`.

The separator between fields must still be managed. `std::quoted` is responsible only for formatting an individual string; it does not define the structure of the entire record.

## Comparing `std::quoted`, `std::string`, and `operator<<`

The `std::string` type stores a string, and `operator<<` ordinarily writes its contents to a stream. In contrast, `std::quoted` changes how a string is represented or extracted during a stream operation.

| Feature                                | `std::string`    | Ordinary `operator<<` | `std::quoted`                 |
| -------------------------------------- | ---------------- | --------------------- | ----------------------------- |
| Stores data                            | Yes              | No                    | No                            |
| Writes a string                        | Through a stream | Yes                   | Yes, using quoted formatting  |
| Preserves whitespace during extraction | Not by itself    | Not applicable        | Yes, when the input is quoted |
| Escapes the delimiter                  | No               | No                    | Yes                           |
| Modifies the original string           | No               | No                    | No                            |

The choice depends on the application's needs. For ordinary text output, `operator<<` is sufficient. For a simple text format in which string boundaries matter, `std::quoted` is a suitable option.

## Comparing `std::quoted` with JSON and Other Data Formats

The `std::quoted` facility should not be confused with serializers for standardized data formats.

JSON, for example, defines specific rules for representing strings, escaping control characters, and structuring data. `std::quoted` implements only its own limited rules for formatting an individual string in a stream.

Its important limitations include:

- It does not convert all control characters into textual escape sequences.
- It does not validate UTF-8.
- It does not define object or array structures.
- It does not implement the complete JSON specification.
- It does not validate input against a complete application-level protocol.

Therefore, a suitable JSON library should be used to generate or process JSON. Using `std::quoted` as a replacement for a standards-compliant serializer may produce data that does not conform to the intended format.

## Comparing `std::quoted` with `std::format`

Introduced in C++20, `std::format` provides general-purpose formatting of values. It serves a broader purpose than `std::quoted` and is designed to construct formatted strings using a specified format string.

In contrast, `std::quoted`, introduced in C++14, primarily handles quoted-string representation through input and output streams.

When the goal is to format numbers and strings in a general-purpose way, `std::format` may be a suitable choice. When a string must be written or read through a stream using the specific rules of `std::quoted`, that facility directly addresses the requirement.

These facilities are not complete substitutes for one another. The choice should depend on the data-format contract and the application's requirements.

## Important Limitations When Extracting Strings

If the input begins with a quotation delimiter but does not contain the corresponding closing delimiter, extraction does not complete successfully, and the stream enters an error state.

For example:

```
#include <iomanip>
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::istringstream input("\"unfinished string");

    std::string text;
    input >> std::quoted(text);

    if (!input) {
        std::cout << "Input error\n";
    }
}
```

Output:

```
Input error
```

In real applications, checking the stream state after extraction is important. You should not assume that every input can be processed successfully by `std::quoted`.

Another important case occurs when the escape character appears at the end of the input without a following character. Since the escape character is used to consume the next character, extraction cannot complete successfully in this situation.

## Managing Stream State and Invalid Input

The `std::quoted` facility follows the usual stream-state rules. Therefore, the stream state should be checked after extraction.

```
#include <iomanip>
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::istringstream input("\"valid value\"");

    std::string value;

    if (input >> std::quoted(value)) {
        std::cout << "Parsed: " << value << '\n';
    } else {
        std::cout << "Failed to parse input\n";
    }
}
```

This pattern is appropriate for programs that receive input from files, networks, or users.

For untrusted input, checking whether extraction succeeded is not enough. Depending on the application's requirements, the program may also need to enforce string-size limits, define an error-handling policy, and perform higher-level validation.

## Overloads and String Types

The `std::quoted` facility is not limited to `std::string`. Its overloads also support strings based on `basic_string` and character pointers.

The `basic_string` family supports character types such as `char` and `wchar_t`. However, the stream type must be compatible with the character type as well.

For example, a program using wide characters should use an appropriate string type and stream. A string based on `wchar_t` cannot simply be combined with `std::cout` without considering the character types involved.

For the character-pointer overload, the input must be a valid null-terminated string. An invalid pointer or a string that is not properly terminated can result in undefined behavior.

By contrast, `std::string` overloads obtain the string length from the string object and are not limited to the first null character. They can therefore handle strings containing embedded null characters, although displaying such strings in a textual format may require additional conventions.

## Temporary Objects and Object Lifetime

Because `std::quoted` returns a proxy object, using it directly in a stream expression is generally straightforward and safe.

```
#include <iomanip>
#include <iostream>
#include <string>

int main() {
    std::cout << std::quoted(std::string("temporary string")) << '\n';
}
```

In this expression, the temporary string remains alive until the end of the full expression, allowing the output operation to complete.

However, retaining a proxy object for later use can be problematic, particularly if it refers to a string whose lifetime has ended. For straightforward operations, it is preferable to use `std::quoted` directly in the input or output expression.

## Unicode Strings

The `std::quoted` facility is not a general-purpose Unicode serialization tool. It applies a limited set of escaping rules based on the character type and the values of `delim` and `escape`.

For UTF-8 strings stored in `std::string`, formatting operates on character units, which represent bytes in UTF-8. The facility does not validate UTF-8 and does not process Unicode characters according to their semantic meaning or code-point count.

Consequently, applications that handle multilingual data must manage encoding validity and conversion policies separately. Using `std::quoted` alone does not guarantee encoding correctness.

## Common Mistakes When Using `std::quoted`

Several common mistakes can lead to unexpected output or incorrect input processing.

### Forgetting the Required Header

The `<iomanip>` header must be included to use `std::quoted`. Including `<iostream>` alone does not guarantee that this facility is available.

### Expecting Every Special Character to Be Escaped

The `std::quoted` facility handles the delimiter and escape character, but it is not a general-purpose system for converting every control character into a textual representation.

### Using Different Input and Output Conventions

If a custom delimiter is used when writing a string but the default delimiter is used when reading it, the data may not be restored as expected.

### Ignoring Extraction Errors

If the delimiters are incomplete or an escape sequence is incomplete, extraction may fail. Checking the stream state should be part of input-processing code.

### Using `std::quoted` to Generate JSON

The rules of `std::quoted` are not equivalent to the complete JSON specification. A suitable library should be used to generate valid JSON.

### Assuming Whitespace Is Always Preserved

Whitespace is preserved when a quoted string is extracted. However, if the input does not begin with the delimiter, extraction behaves like ordinary string extraction and stops at the first whitespace character.

## Practical Design Recommendations

In professional projects, the use of `std::quoted` should follow a clearly defined convention.

- For displaying strings in logs and diagnostic output, this facility can improve readability.
- For storing strings in a simple text format, define delimiter rules and a policy for handling invalid input first.
- For exchanging data between independent applications, use a format with precisely documented rules.
- For untrusted input, enforce size limits and handle errors at the application level.
- For Unicode strings, validate encoding separately from stream formatting.
- For structured serialization, a specialized library for the intended format is generally a more reliable choice.

## Summary

Introduced in C++14, `std::quoted` provides a standard and straightforward way to write and read quoted strings through streams. It supports custom delimiters and escape characters and preserves whitespace inside quoted strings during extraction.

Its primary applications include output formatting, processing simple text files, and implementing basic conventions for storing and restoring strings.

However, `std::quoted` is not a general-purpose serializer and cannot replace formats such as JSON or specialized serialization libraries. For professional use, it is important to understand the input and output conventions, stream state, escape behavior, object lifetime, and encoding limitations.

---

## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>