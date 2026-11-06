# ============================================
# INPUT
# ============================================

The Persian Markdown document provided after this prompt is the source document.

Translate and convert it into a professional English educational Markdown document.

The final output must be a complete, polished, technically accurate English Markdown document suitable for publication on GitHub.


# 1. Primary Objective

Translate the provided Persian document into natural, professional, technically accurate English.

Preserve the author's original meaning, technical explanations, examples, structure, and intent.

This is a technical documentation translation, not a literal word-for-word translation.

Use natural English appropriate for professional C++ developers.

Do NOT simplify or omit technical information merely because the original document is detailed.

Do NOT add new technical information that is not present in the source document unless a minimal grammatical adjustment is necessary to make the English text coherent.


# 2. Preserve the Markdown Structure

Preserve the original Markdown structure as much as possible.

Keep:

- the main `#` heading
- `##`, `###`, and deeper headings
- paragraphs
- lists
- numbered lists when they exist as actual content
- tables
- blockquotes
- emphasis
- inline code
- links
- code blocks
- code block language identifiers

Do not add a Table of Contents.

Do not introduce new section numbering.

Do not reorganize the document unless a structural change is necessary to produce natural English.


# 3. Remove Persian RTL Workarounds

The source document was originally written in Persian and contains special formatting adjustments that were necessary only because Persian is an RTL language and the document was rendered on GitHub.

These RTL workarounds MUST be converted back to their normal English/technical form in the final document.

The final document is English and therefore MUST NOT retain Persian RTL formatting workarounds.


# 4. Restore C++ Spelling

In the Persian document, `C++` may have been intentionally written as:

`++C`

because of Persian RTL rendering requirements.

When converting the document to English, ALWAYS restore:

`++C`

back to:

`C++`

For example:

Persian source:

در ++C می‌توان از این قابلیت استفاده کرد.

English output:

This feature can be used in C++.

This restoration must be applied throughout the document whenever `++C` represents the C++ programming language.

IMPORTANT:

Do NOT modify actual C++ source code incorrectly.

If a code block already contains the standard spelling:

```cpp
C++
````

preserve it exactly.

The RTL workaround must only be reversed where it was used in Persian prose or headings.

# 5. Restore Function Names and Parentheses

The Persian document may contain function names written in an RTL-safe form such as:

`()MyFunction`

instead of the normal:

`MyFunction()`

This was done only to prevent incorrect rendering inside Persian RTL prose.

When translating the document into English, ALWAYS restore these function names to their normal form:

`MyFunction()`

For example, if the Persian source contains:

از تابع `()std::move` برای انتقال مقدار استفاده می‌شود.

the English output must contain:

The `std::move()` function is used to move the value.

Do NOT preserve the RTL-safe form `()std::move` in the English document.

Restore:

`()MyFunction`

to:

`MyFunction()`

whenever it represents a function name followed by parentheses.

IMPORTANT:

Never alter actual C++ syntax inside code blocks.

For example:

```cpp
auto result = std::move(value);
```

must remain exactly valid C++ code.

# 6. English Markdown Heading Rules

The final document is entirely in English.

Therefore, do NOT apply any Persian RTL heading rules.

English headings may naturally begin with:

* English words
* technical terms
* C++ terminology
* function names
* class names
* types
* namespaces

For example, a Persian heading such as:

## معرفی std::expected در ++C

should become:

## Introduction to std::expected in C++

Do NOT add artificial Persian prefixes to English headings.

Do NOT add unnecessary words merely to influence text direction.

Do NOT number headings unless the source document explicitly contains meaningful numerical content that is part of the heading rather than a formatting workaround.

# 7. New-Line RTL Workarounds

The Persian source may contain artificial Persian prefixes before English technical terms because a Persian line was not allowed to start with an English word.

These prefixes were added only to establish an RTL context and prevent GitHub from rendering the line as LTR.

When translating into English, remove such artificial RTL prefixes when they are no longer semantically necessary.

For example, if the source contains:

مفهوم Move Semantics یکی از ...

translate naturally as:

Move Semantics is one of ...

Do NOT preserve `مفهوم` as an unnecessary literal word if it was only used as an RTL rendering prefix.

However, if the Persian word carries real semantic meaning in context, translate its meaning normally.

For example:

مفهوم Move Semantics

may become:

The Concept of Move Semantics

or:

Move Semantics

depending on the actual context and meaning.

Use judgment to distinguish between:

1. a meaningful part of the sentence
2. an artificial Persian RTL prefix added only for rendering purposes

# 8. Comments Inside Code

All comments inside C++ code snippets must remain in English.

If the source document already contains English comments inside code blocks, preserve them.

If any Persian comment somehow exists inside a code block, translate it into clear, natural English.

Never introduce Persian text into code comments.

For example:

```cpp
// محاسبه مقدار نهایی
auto result = calculateValue();
```

must become:

```cpp
// Calculate the final value
auto result = calculateValue();
```

Do NOT translate actual code identifiers, keywords, APIs, namespaces, or syntax merely because they look like English words.

Only translate human-language comments and explanatory text.

# 9. Preserve Technical Identifiers

Preserve technical identifiers exactly.

Do NOT translate or rename:

* function names
* class names
* struct names
* enum names
* variable names
* namespace names
* template parameters
* type names
* standard library names
* API names
* compiler flags
* command names
* filenames
* URLs
* library names

For example:

`std::expected`

must remain:

`std::expected`

and:

`std::move()`

must remain:

`std::move()`.

# 10. Code Must Remain Valid

Treat all code blocks as actual source code.

Do NOT apply translation or stylistic changes to C++ syntax.

Preserve:

* keywords
* operators
* punctuation
* braces
* parentheses
* namespaces
* identifiers
* templates
* string literals
* compiler directives
* code structure

Only translate comments inside code blocks when necessary.

The resulting code must remain valid C++.

# 11. Technical Translation Quality

Use terminology familiar to professional C++ developers.

Prefer established English technical terminology rather than awkward literal translations.

For example, use standard terms such as:

* move semantics
* value category
* lvalue
* rvalue
* perfect forwarding
* type deduction
* compile-time
* runtime
* exception safety
* undefined behavior
* standard library
* template instantiation
* constraints
* concepts

Do not translate established C++ terminology into unnatural English equivalents.

Preserve distinctions between similar technical concepts.

# 12. Do Not Over-Edit

The Persian document has already been reviewed and edited by a human.

Therefore:

* Preserve the author's organization.
* Preserve the author's explanations.
* Preserve examples unless they contain a genuine technical or grammatical problem.
* Do not rewrite the document unnecessarily.
* Do not remove information.
* Do not add unrelated information.
* Do not introduce new sections merely because they might be useful.
* Do not change the technical scope of the document.

The goal is to convert the reviewed Persian document into an equivalent professional English document, not to redesign its content.

# 13. Final Consistency Check

Before producing the final output, internally verify that:

* The entire document is in English.
* No Persian prose remains.
* No Persian RTL formatting workaround remains.
* `++C` used as a Persian RTL workaround has been restored to `C++`.
* `()FunctionName` used as a Persian RTL workaround has been restored to `FunctionName()`.
* English headings do not contain artificial Persian prefixes.
* No unnecessary heading numbering has been introduced.
* No Table of Contents has been added.
* C++ code remains valid.
* Function calls inside code use normal syntax such as `MyFunction()`.
* Technical identifiers remain unchanged.
* Code comments are entirely in English.
* Markdown formatting is preserved.
* The final document reads naturally to a professional C++ developer.

# 14. Final Output

Return ONLY the final English Markdown document.

Do NOT provide:

* translation notes
* explanations about the translation
* a summary of changes
* comments about RTL conversion
* comments about the source document
* a preface
* a postscript

The output must be ready to save directly as a `.md` file and publish on GitHub.

# SOURCE DOCUMENT

The Persian Markdown document to be translated follows below:

[PASTE THE REVIEWED PERSIAN MARKDOWN DOCUMENT HERE]
