# `std::string_view` in C++17

## Table of Contents

- [Introduction](#introduction)
- [Required Header](#required-header)
- [Non-Owning Model](#non-owning-model)
- [Pointer + Length Representation](#pointer--length-representation)
- [Construction](#construction)
- [Core C++17 Operations](#core-c17-operations)
- [Substrings and Trimming](#substrings-and-trimming)
- [Search and Comparison](#search-and-comparison)
- [Character Specializations](#character-specializations)
- [String Literal `sv`](#string-literal-sv)
- [Non-Null-Terminated Data](#non-null-terminated-data)
- [Converting to `std::string`](#converting-to-stdstring)
- [Safe `data()` Usage](#safe-data-usage)
- [Lifetime and Dangling Views](#lifetime-and-dangling-views)
- [Practical Parser and API Use Cases](#practical-parser-and-api-use-cases)
- [Performance Caveats](#performance-caveats)
- [C++20 and C++23 Additions](#c20-and-c23-additions)
- [Common Errors](#common-errors)
- [Complete Example](#complete-example)
- [Exercises](#exercises)
- [Conclusion](#conclusion)
- [Contributors](#contributors)

---

## Introduction

`std::string_view` is a C++17 class template for a read-only view over a contiguous sequence of characters. It does not own the characters it refers to. Instead, it stores a pointer and a length, then lets you inspect that character range without copying it.

That makes it a good fit for parsing, tokenization, logging, slicing, and API design where the caller already owns the text and the callee only needs to read it.

The key idea is simple:

- `std::string` owns data
- `std::string_view` observes data

Because it is non-owning, `std::string_view` is cheap to copy and cheap to pass by value. The tradeoff is lifetime safety: the data must remain alive for as long as the view is used.

---

## Required Header

Include `<string_view>`.

```cpp
#include <string_view>
```

Common alias:

```cpp
namespace sv = std::string_view;
```

The main template is:

```cpp
template<class CharT, class Traits = std::char_traits<CharT>>
class basic_string_view;
```

The standard alias `std::string_view` means `std::basic_string_view<char>`.

---

## Non-Owning Model

A `std::string_view` does not allocate memory and does not copy the underlying text. It only keeps a view into an existing character sequence.

This is why it is often described as a "window" into text.

Example:

```cpp
#include <iostream>
#include <string>
#include <string_view>

int main() {
    std::string text = "hello world";
    std::string_view view = text;

    std::cout << view << '\n';
}
```

`view` refers to `text`; it does not own the characters. If `text` is modified in a way that keeps its buffer valid, the view may still observe those changes. If `text` is destroyed, the view becomes dangling.

---

## Pointer + Length Representation

Conceptually, `std::string_view` behaves like this:

```cpp
struct string_view_like {
    const char* ptr;
    std::size_t len;
};
```

The important point is that the length is stored explicitly. That is what lets `string_view` work with non-null-terminated data and with substrings that are only part of a larger buffer.

Example:

```cpp
#include <iostream>
#include <string_view>

int main() {
    std::string_view sv("hello world", 5);
    std::cout << sv << '\n'; // hello
}
```

No terminator is required at position 5. The view knows its own size.

---

## Construction

`std::string_view` can be constructed from many read-only text sources.

### From a string literal

```cpp
std::string_view sv1 = "hello";
```

### From a `std::string`

```cpp
std::string s = "hello";
std::string_view sv2 = s;
```

### From a character array with explicit length

```cpp
char buf[] = {'h', 'e', 'l', 'l', 'o'};
std::string_view sv3(buf, 5);
```

### From pointer plus length

```cpp
const char* p = "abcdef";
std::string_view sv4(p, 3); // "abc"
```

### From `std::string` data and size

```cpp
std::string s = "payload";
std::string_view sv5{s.data(), s.size()};
```

This is useful when you already have the pointer and length.

---

## Core C++17 Operations

`std::string_view` provides the common read-only operations you expect from text handling.

### Size and emptiness

```cpp
std::string_view sv = "hello";

sv.size();      // 5
sv.length();    // 5
sv.empty();     // false
sv.max_size();  // implementation-defined
```

### Element access

```cpp
sv[0];      // 'h'
sv.at(1);    // 'e', throws std::out_of_range if invalid
sv.front();  // 'h'
sv.back();   // 'o'
sv.data();   // pointer to the first character
```

### Iteration

```cpp
for (char ch : sv) {
    std::cout << ch << '\n';
}
```

You can also use `begin()`, `end()`, `cbegin()`, `cend()`, `rbegin()`, and `rend()`.

### Copying characters out

```cpp
#include <array>
#include <iostream>
#include <string_view>

int main() {
    std::string_view sv = "hello";
    std::array<char, 6> out{};

    sv.copy(out.data(), 5);
    std::cout << out.data() << '\n';
}
```

`copy()` copies characters into a destination buffer. It does not append a null terminator.

---

## Substrings and Trimming

`substr()`, `remove_prefix()`, and `remove_suffix()` are the key slicing tools.

### `substr()`

`substr(pos, count)` returns a new view over a subrange.

```cpp
std::string_view sv = "hello world";
std::string_view word = sv.substr(6, 5); // "world"
```

This is O(1) because it only adjusts pointer and length metadata.

### `remove_prefix()`

```cpp
std::string_view sv = "token=value";
sv.remove_prefix(6);
std::cout << sv << '\n'; // value
```

### `remove_suffix()`

```cpp
std::string_view sv = "value\r\n";
sv.remove_suffix(2);
std::cout << sv << '\n'; // value
```

Both trimming functions modify the view itself, not the underlying characters.

A common parser pattern is to move through a line with repeated prefix trimming:

```cpp
std::string_view line = "GET /index.html HTTP/1.1";

std::string_view method = line.substr(0, line.find(' '));
line.remove_prefix(method.size() + 1);

std::string_view path = line.substr(0, line.find(' '));
line.remove_prefix(path.size() + 1);

std::string_view version = line;
```

---

## Search and Comparison

`std::string_view` supports the important read-only search and comparison operations.

### Comparison

Use `compare()` for lexical comparison:

```cpp
std::string_view a = "abc";
std::string_view b = "abd";

int result = a.compare(b); // negative because "abc" < "abd"
```

The usual relational operators also work in C++17:

```cpp
a == b;
a != b;
a < b;
a <= b;
a > b;
a >= b;
```

### Searching

```cpp
std::string_view sv = "the quick brown fox";

sv.find("quick");
sv.rfind('o');
sv.find_first_of("aeiou");
sv.find_last_of("xyz");
sv.find_first_not_of(' ');
sv.find_last_not_of(' ');
```

All of these return `std::string_view::npos` when the search fails.

### Useful patterns

```cpp
std::string_view path = "/home/user/file.txt";
auto slash = path.rfind('/');
auto name = path.substr(slash + 1);
auto ext = name.rfind('.');
```

---

## Character Specializations

The standard library provides several aliases for different character types.

- `std::string_view` for `char`
- `std::wstring_view` for `wchar_t`
- `std::u16string_view` for `char16_t`
- `std::u32string_view` for `char32_t`
- `std::u8string_view` for `char8_t` in C++20

A generic function can take `std::basic_string_view<CharT>` when it should work across multiple character types.

Example:

```cpp
#include <iostream>
#include <string_view>

std::wstring_view drop_prefix(std::wstring_view sv) {
    if (sv.size() > 2) {
        sv.remove_prefix(2);
    }
    return sv;
}

int main() {
    std::wstring_view sv = L"abcdef";
    std::wcout << drop_prefix(sv) << L'\n';
}
```

---

## String Literal `sv`

C++17 adds the `"sv"` user-defined literal in `std::string_view_literals`.

```cpp
using namespace std::string_view_literals;

auto sv = "hello"sv;
```

This produces a `std::string_view` directly, which is cleaner than writing the type explicitly.

The literal is especially useful in comparisons and parser code:

```cpp
using namespace std::string_view_literals;

std::string_view a = "alpha"sv;
std::string_view b = "beta"sv;
```

---

## Non-Null-Terminated Data

`std::string_view` does not need a null terminator. That is one of its main advantages.

Example with raw data:

```cpp
#include <cstddef>
#include <iostream>
#include <string_view>

int main() {
    char raw[] = {'M', 'a', 'd', 'h', 'a', 'v'};
    std::string_view sv(raw, 6);

    std::cout << sv << '\n';
}
```

This is valid because the view knows the exact length.

That also means `sv.data()` is not automatically a C string. It is just a pointer to the first character.

---

## Converting to `std::string`

When you need ownership, create a `std::string` explicitly.

```cpp
#include <iostream>
#include <string>
#include <string_view>

void print_owned(std::string s) {
    std::cout << s << '\n';
}

int main() {
    std::string_view sv = "hello world";
    std::string owned = std::string(sv);
    print_owned(owned);
}
```

The conversion is explicit because ownership changes. `std::string_view` does not implicitly become `std::string` in all call contexts.

---

## Safe `data()` Usage

`data()` is safe when the consumer understands a pointer-plus-length contract.

Good uses:

```cpp
std::string_view sv = "hello";
std::cout.write(sv.data(), static_cast<std::streamsize>(sv.size()));
```

```cpp
#include <cstdio>
std::fwrite(sv.data(), 1, sv.size(), stdout);
```

Unsafe uses:

```cpp
std::puts(sv.data());   // only safe if sv is known to be null-terminated
std::strlen(sv.data()); // same issue
```

If a function requires a C string, first create an owning `std::string` and use `c_str()`:

```cpp
std::string s(sv);
std::puts(s.c_str());
```

---

## Lifetime and Dangling Views

This is the part that matters most in real code.

A `std::string_view` must not outlive the characters it points to.

### Safe case

```cpp
std::string text = "hello";
std::string_view sv = text;
std::cout << sv << '\n';
```

### Unsafe case with a temporary

```cpp
std::string_view sv = std::string("hello");
```

Here the temporary `std::string` is destroyed at the end of the statement, so `sv` dangles immediately.

### Unsafe return

```cpp
std::string_view bad_view() {
    std::string local = "temporary";
    return local;
}
```

That returned view refers to destroyed storage.

### Safe return patterns

Return an owning string:

```cpp
std::string good_owned() {
    return "temporary";
}
```

Or return a view into static storage:

```cpp
std::string_view good_view() {
    return "static text";
}
```

### Why this matters in APIs

If a function stores a `std::string_view` for later use, the caller must guarantee the backing data outlives that storage. For one-shot parsing, immediate inspection is fine. For asynchronous work, persistent storage, or cross-thread handoff, use `std::string` or another owning type.

---

## Practical Parser and API Use Cases

### Tokenizing a line

`std::string_view` is ideal for splitting a line into pieces without copying.

```cpp
#include <iostream>
#include <string_view>

int main() {
    std::string_view line = "name=Ordikhani;role=author";

    auto first_sep = line.find(';');
    auto first_part = line.substr(0, first_sep);
    auto second_part = line.substr(first_sep + 1);

    auto eq1 = first_part.find('=');
    auto eq2 = second_part.find('=');

    std::cout << first_part.substr(0, eq1) << " -> "
              << first_part.substr(eq1 + 1) << '\n';
    std::cout << second_part.substr(0, eq2) << " -> "
              << second_part.substr(eq2 + 1) << '\n';
}
```

### Accepting many input forms in one API

```cpp
#include <iostream>
#include <string_view>

void log_message(std::string_view msg) {
    std::cout << msg << '\n';
}

int main() {
    log_message("literal text");
    std::string text = "owned text";
    log_message(text);
    log_message(std::string_view{text});
}
```

This reduces overload clutter and keeps the call site simple.

### Parsing a command line

```cpp
std::string_view input = "copy source.txt target.txt";
auto space = input.find(' ');
auto cmd = input.substr(0, space);
input.remove_prefix(space + 1);
auto next = input.find(' ');
auto src = input.substr(0, next);
auto dst = input.substr(next + 1);
```

The parser walks the same buffer without copying tokens out.

---

## Performance Caveats

`std::string_view` is not magic. It is faster only when avoiding copies matters.

A few practical points:

- it is cheap to create and copy, but it does not own the data
- `substr()` on `std::string_view` is O(1), while `std::string::substr()` copies
- passing a `std::string_view` is often cheaper than passing a temporary `std::string`
- if you need ownership, a copy is unavoidable and `std::string` is the right tool

There is also a common misunderstanding about `std::string` and small string optimization. Short strings may already avoid heap allocation in many implementations, so the gain from `std::string_view` is not always about allocation alone. The bigger wins are usually reduced copying, simpler APIs, and cheaper substring handling.

Use `std::string_view` where it matches the problem. Do not use it as a replacement for ownership.

---

## C++20 and C++23 Additions

`std::string_view` gained several useful members after C++17.

### C++20

- `starts_with()`
- `ends_with()`
- `std::u8string_view`
- stronger constexpr iterator support
- ranges integration as a borrowed view

Example:

```cpp
#include <string_view>

int main() {
    std::string_view sv = "prefix_body_suffix";
    bool ok1 = sv.starts_with("prefix");
    bool ok2 = sv.ends_with("suffix");
    (void)ok1;
    (void)ok2;
}
```

### C++23

- `contains()`
- `basic_string_view` is formally required to be trivially copyable

Example:

```cpp
#include <string_view>

int main() {
    std::string_view sv = "alpha beta gamma";
    bool has_beta = sv.contains("beta");
    (void)has_beta;
}
```

These additions make the type a bit more convenient, but the core model stays the same.

---

## Common Errors

1. Returning a `std::string_view` to local storage. The view dangles as soon as the function returns.
2. Binding a view to a temporary `std::string`. The temporary dies at the end of the full expression.
3. Calling C APIs with `sv.data()` and assuming null termination.
4. Storing a view inside an object without proving the source data outlives that object.
5. Using `string_view` for code that needs to mutate characters.
6. Confusing `substr()` on `std::string_view` with `std::string::substr()`. One copies; the other does not.
7. Forgetting that `remove_prefix()` and `remove_suffix()` mutate the view itself.

---

## Complete Example

```cpp
#include <cstddef>
#include <iostream>
#include <string>
#include <string_view>
#include <vector>

struct KV {
    std::string_view key;
    std::string_view value;
};

std::vector<KV> parse_pairs(std::string_view input) {
    std::vector<KV> pairs;

    while (!input.empty()) {
        auto end = input.find_first_of(";\n");
        std::string_view item = input.substr(0, end);

        auto eq = item.find('=');
        if (eq != std::string_view::npos) {
            pairs.push_back({item.substr(0, eq), item.substr(eq + 1)});
        }

        if (end == std::string_view::npos) {
            break;
        }
        input.remove_prefix(end + 1);
    }

    return pairs;
}

int main() {
    std::string text = "name=Ali;role=admin;city=Tehran";
    std::vector<KV> pairs = parse_pairs(text);

    for (const auto& kv : pairs) {
        std::cout << kv.key << " = " << kv.value << '\n';
    }

    std::string_view sv = text;
    std::cout << "role present? " << std::boolalpha
              << (sv.find("role") != std::string_view::npos) << '\n';
}
```

This example stays within C++17 while still showing a realistic parser-style use of `std::string_view`.

---

## Exercises

1. Write a function that trims leading and trailing spaces from a `std::string_view` without allocating.
2. Parse a header line of the form `Key: Value` into two views.
3. Implement a CSV splitter that returns `std::vector<std::string_view>` for a single input line.
4. Replace a `const std::string&` parameter in a read-only API with `std::string_view` and compare the call sites.
5. Write a small benchmark that compares `std::string::substr()` and `std::string_view::substr()` in a loop.

---

## Conclusion

`std::string_view` is one of the most useful C++17 additions for text-heavy code. It gives you a cheap, read-only, non-owning view into character data, which makes parsing and API design cleaner and faster.

The rule to remember is simple: use `std::string_view` when you only need to read text and can guarantee lifetime. Use `std::string` when you need ownership.

---

## Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) |  |  |  |  |

