# ============================================
# EDITABLE INPUT
# DOCUMENT GENERATION INSTRUCTIONS
# ============================================
Create a comprehensive Persian educational Markdown document (.md) about the following subject:

MySubject

ADDITIONAL TOPICS AND REQUIREMENTS:
- Topic 1
- Topic 2
- Topic 3
- Topic 4

The document will be published on GitHub and is intended for C++ developers. Follow all of the rules below carefully.

# 1. Language and Writing Style

- Write the entire document in Persian.
- Use clear, natural, educational, and technically accurate Persian.
- Keep technical terms in English whenever translating them would make their meaning less precise or less familiar to C++ developers.
- Write for a professional C++ developer, while explaining concepts clearly enough that the reader can follow the subject from its fundamentals to advanced practical usage.
- Explain not only what a feature is, but also:
  - why it exists
  - what problem it solves
  - how it works
  - when it should be used
  - when it should not be used
  - practical use cases
  - limitations
  - edge cases
  - common mistakes
  - important design considerations
- Prefer modern C++ practices over outdated techniques.
- Do not mention these instructions in the generated document.


# 2. Technical Accuracy

- Use modern and standard C++ terminology.
- Clearly distinguish between:
  - C++ language features
  - standard library features
  - compiler extensions
  - implementation-specific behavior
- If a feature belongs to a particular C++ standard version, explicitly mention the relevant version, such as C++11, C++14, C++17, C++20, C++23, or C++26.
- Do not invent language features, APIs, syntax, standardization details, or compiler behavior.
- Code examples must use valid and technically correct C++ syntax.
- When relevant, explain important differences between language versions and older approaches.


# 3. Comprehensive Professional Coverage

The document must provide comprehensive coverage of the subject and include all important concepts, features, capabilities, related mechanisms, practical techniques, limitations, edge cases, and important details that a professional C++ developer should know.

Do not limit the document to a basic introduction or a few simple examples.

Cover the subject from a professional and practical perspective, including closely related features when they are relevant to understanding or using the subject effectively.

However, avoid unnecessary repetition and redundant explanations.

- Do NOT repeat the same concept multiple times unless there is a meaningful reason.
- Do NOT restate information that has already been clearly explained.
- Do NOT repeat the same example merely to explain the same concept again.
- If several features share the same underlying concept, explain that concept once and then focus on their differences.
- Prefer adding new and useful information over repeating previously explained information.
- Each section should contribute new knowledge, practical insight, an important distinction, or a useful example.
- Keep the document comprehensive without making it unnecessarily verbose.

The goal is to create documentation that is comprehensive enough for a professional C++ developer while remaining concise, focused, and free of redundant repetition.


# 4. Additional Topics and Requirements

In addition to the main subject, cover all relevant topics, concepts, features, or requirements listed in:

ADDITIONAL TOPICS AND REQUIREMENTS

These are explicit requirements and MUST be covered when they are relevant to the main subject.

Integrate these topics naturally into the document structure rather than simply adding them as an unrelated section.

If an additional topic is already naturally covered by another section, do not repeat the explanation. Instead, make sure the existing section adequately covers the requested topic.

If an additional topic is not relevant to the main subject, do not force it into the document.


# 5. Document Structure

- Start the document directly with the main `#` heading.
- Do NOT include a Table of Contents (TOC).
- Do NOT create a manually written list of links to sections.
- Do NOT create GitHub anchor links for navigation.
- Use Markdown headings (`#`, `##`, `###`, etc.) to organize the content naturally.
- Use sections such as introduction, concepts, usage, examples, practical applications, pitfalls, limitations, comparisons, and summary when they are relevant.
- Adapt the structure to the actual subject instead of blindly following a fixed structure.


# 6. Persian Markdown Heading Rules

Because the document is written in Persian (RTL) and will be rendered on GitHub, all Markdown headings must be written in a way that ensures correct RTL rendering.

Every Markdown heading (`#`, `##`, `###`, etc.) MUST start with at least one meaningful Persian word or Persian phrase.

Never start a heading with:

- an English word
- an English technical term
- a C++ keyword
- a function name
- a class name
- a type name
- a namespace
- a number
- a symbol
- an abbreviation

If the natural title starts with an English technical term, add a short, natural, contextually appropriate Persian prefix before it.

The Persian prefix must be meaningful and related to the actual content of the heading. Do not use meaningless filler words merely to satisfy this rule.

For example, instead of:

## Concepts in C++20

write:

## مفاهیم Concepts در ++C

Instead of:

## std::expected

write:

## معرفی std::expected

Instead of:

## Move Semantics

write:

## مفهوم Move Semantics

Choose the Persian prefix according to the actual meaning of the heading.

Suitable prefixes may include:

- معرفی
- مفهوم
- بررسی
- نحوه استفاده از
- کاربرد
- مثال
- مقایسه
- تفاوت
- مدیریت
- پیاده‌سازی
- نکات مهم
- محدودیت‌ها


# 7. Heading Numbering

NEVER add numerical prefixes, counters, or section numbers to headings.

Do NOT write:

## 1- مقدمه
## 2- معرفی
## 3- مثال

Do NOT write:

## 1. مقدمه
## 2. معرفی
## 3. مثال

Do NOT use section numbering such as:

- 1
- 1.1
- 1.2
- 2
- 2.1
- 2.2

Instead, write:

## مقدمه

## معرفی

## مثال

The Markdown heading hierarchy (`#`, `##`, `###`, ...) is sufficient to organize the document.


# 8. Persian RTL Context at the Beginning of New Lines

Whenever a new paragraph, sentence, list item, blockquote, note, or other block of Persian prose starts on a new line, it MUST NOT begin with an English word, English technical term, C++ keyword, function name, class name, type name, namespace, number, symbol, or abbreviation.

If the intended Persian text would naturally begin with an English term, add a short, natural, and contextually relevant Persian word or phrase before it.

This Persian prefix is required to establish the correct RTL context and prevent GitHub from rendering the line as LTR or left-aligned.

For example, instead of:

Concepts در ++C به ...

write:

مفهوم Concepts در ++C به ...

Instead of:

std::expected برای مدیریت ...

write:

نوع std::expected برای مدیریت ...

Instead of:

Move Semantics یکی از ...

write:

مفهوم Move Semantics یکی از ...

Choose the Persian prefix naturally according to the meaning and context of the sentence.

Do not use meaningless filler words merely to satisfy this rule.

This rule applies to:

- normal paragraphs
- individual sentences
- list items
- blockquotes
- explanatory text
- notes
- descriptions

This rule does NOT apply to:

- fenced code blocks
- actual source code
- shell commands
- URLs
- filenames
- lines intentionally written entirely in English
- technical identifiers when they are part of actual code


# 9. C++ RTL Rendering Rule

Because `C++` can be rendered incorrectly when embedded in Persian RTL prose on GitHub:

Whenever `C++` appears inside normal Persian prose, ALWAYS write it as:

++C

For example:

در ++C می‌توان از این قابلیت استفاده کرد.

instead of:

در C++ می‌توان از این قابلیت استفاده کرد.

Apply this replacement EVERY TIME `C++` appears in Persian prose.

Do NOT apply this replacement inside:

- fenced code blocks
- actual source code
- compiler commands
- filenames
- URLs
- technical identifiers
- quoted code

In those contexts, preserve the original `C++` spelling exactly.


# 10. Function Names with Parentheses in Persian Prose

When a function name appears together with `()` inside Persian prose, write the parentheses BEFORE the function name in the Markdown source.

For example, write:

()MyFunction

instead of:

MyFunction()

This is necessary to ensure that the function name and its parentheses are rendered correctly in Persian RTL text on GitHub.

For example:

از تابع `()std::move` برای انتقال مقدار استفاده می‌شود.

Do NOT write:

از تابع `std::move()` برای انتقال مقدار استفاده می‌شود.

This rule applies to function names appearing inside Persian prose, including inline code.

IMPORTANT:

- Apply this RTL-safe representation only to Persian prose.
- Never apply it inside fenced code blocks.
- Never modify actual C++ source code.
- Preserve the exact spelling, capitalization, namespace, and qualification of the function name.
- In actual C++ code, always use standard C++ syntax such as `MyFunction()`.


# 11. Code Blocks

All C++ source code must be placed inside fenced Markdown code blocks using the `cpp` language identifier.

For example:

```cpp
#include <iostream>

int main() {
    std::cout << "Hello\n";
}
````

Never apply Persian RTL transformations to the contents of code blocks.

Inside code blocks:

* Keep `C++` exactly as `C++`.
* Keep function calls exactly as `MyFunction()`.
* Keep namespaces and identifiers unchanged.
* Keep valid C++ syntax.
* Do not reverse parentheses.
* Do not alter keywords or technical identifiers for RTL purposes.

# 12. Comments Inside Code Snippets

NEVER use Persian words or Persian sentences inside code snippets.

All comments written inside C++ code blocks MUST be written entirely in English.

Do NOT mix Persian and English inside code comments, because mixed RTL/LTR text can render incorrectly and make the code difficult to read on GitHub.

Do NOT write:

```cpp
// این تابع مقدار را محاسبه می‌کند
auto result = calculateValue();
```

Do NOT write:

```cpp
// محاسبه مقدار نهایی
auto result = calculateValue();
```

Instead, always write comments entirely in English:

```cpp
// Calculate the final value
auto result = calculateValue();
```

This applies to:

* `//` single-line comments
* `/* ... */` block comments
* comments inside C++ code snippets
* explanatory comments attached to code examples

All comments inside code snippets must use English only.

Persian explanations must be written outside the code block, as normal Persian prose.

For example:

توجه کنید که تابع `()calculateValue` مقدار نهایی را محاسبه می‌کند.

```cpp
// Calculate the final value
auto result = calculateValue();
```

No Persian text should appear anywhere inside a code snippet.

# 13. Final Output Requirements

* Return ONLY the final Markdown document.
* Do NOT wrap the entire document in a code block.
* Do NOT provide explanations before or after the document.
* Do NOT mention these instructions.
* Do NOT include a Table of Contents.
* Do NOT number headings.
* Ensure every heading starts with meaningful Persian text.
* Ensure every new line of Persian prose has an appropriate Persian RTL context before any English technical term.
* Ensure every `C++` appearing in Persian prose is written as `++C`.
* Ensure function names with `()` appearing in Persian prose use the RTL-safe form `()FunctionName`.
* Never apply Persian RTL transformations to actual C++ source code.
* Never use Persian text inside C++ code comments.
* Ensure the final document is comprehensive, professional, technically accurate, focused, and free of unnecessary repetition.
