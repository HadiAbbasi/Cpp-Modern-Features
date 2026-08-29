<div align="right">

[🇺🇸 English](../../en/cpp11/regex.md) | [🇮🇷 فارسی](./regex.md)

</div>

# Regular Expression 
> Regular expressions (or regex in short) is a much-hated & underrated topic so far with Modern C++. But at the same time, correct use of regex can spare you writing many lines of code. If you have spent quite enough time in the industry. And not knowing regex then you are missing out on 20-30% productivity. In that case, I highly recommend you to learn regex, as it is one-time investment(something similar to learn once, write anywhere philosophy).

Regular expressions are powerful but also sometimes expensive and complicated machinery to work with text. When the interface of a std::string or the algorithms of the Standard Template Library can do the job, use them.  

**Use-Case for Regular Expressions**

- Check if a text matches a text pattern: `std::regex_match`
- Search for a text pattern in a text: `std::regex_search`
- Replace a text pattern with a text: `std::regex_replace`
- Iterate through all text patterns in a text: `std::regex_iterator` and `std::regex_token_iterator`

## Syntax

### Regular Expression Syntax

`.` match any 1 character except newline

`*` match 0 or more of previous pattern

`+` match 1 or more of previous pattern

`?` match 0 or 1 of previous pattern meaning pattern is optional

`{n}` match n of previous pattern

`{n,}` match at least n of previous pattern

`{n,m}` match at least n and at most m of previous pattern

`*? +? ?? {n,m}?` use lazy matching (? appears after repetition symbols)

`(...)` capture group

`(?:...)` non capturing group

`[...]` any character in the class (may contain ranges)

`[^...]` any character not in the class (may contain ranges) (^ must be first character after [)

`...|...` pattern before or pattern after

`^` start of line

`$` end of line

`\b` word boundary

`\B` not word boundary

### Character Classes
`\d` any digit 0–9

`\D` not a digit

`\s` whitespace characters such as space, tab, newline

`\S` not white space

`\w` letter, digit or underscore

`\W` not letter, digit or underscore

### Lookahead

`(?=...)` match only if followed by pattern
`(?!...)` match only if not followed by pattern

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | m.roodsarabi76@gmail.com | | [mbr](https://t.me/ad1mi2n) |

</div>