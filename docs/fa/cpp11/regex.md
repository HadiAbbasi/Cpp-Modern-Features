<div align="right">

[🇺🇸 English](../../en/cpp11/regex.md) | [🇮🇷 فارسی](./regex.md)

</div>

# عبارات باقاعده (Regular Expressions)

![regex](./assets/regex.webp)

در این مطلب ابتدا نگاهی سریع به نحو (Syntax) عبارات باقاعده می‌اندازیم و سپس به برخی قابلیت‌هایی اشاره می‌کنیم که در کتابخانه‌ی استاندارد C++ پشتیبانی نمی‌شوند. البته کتابخانه‌های شخص ثالثی مانند **Boost** امکانات بیشتری ارائه می‌دهند، اما ترجیح من این است که تا حد امکان از کتابخانه‌ی استاندارد استفاده کنم تا کدها قابل حمل (Cross-Platform) باقی بمانند.

با این مقدمه، بیایید با یک مرور سریع بر نحو عبارات باقاعده، بر اساس استاندارد **ECMAScript** که C++ از آن پشتیبانی می‌کند، شروع کنیم.

---

## Cheat Sheet — نحو الگوهای Regex

### کلاس‌های کاراکتری (Character Classes)

| نحو | توضیح |
|---|---|
| `.` | هر کاراکتری به‌جز خط جدید |
| `[abc]` | یکی از کاراکترهای `a`، `b` یا `c` |
| `[^abc]` | هر کاراکتری به‌جز `a`، `b` و `c` |
| `[a-z]` | حروف کوچک انگلیسی |
| `[0-9]` | اعداد بین `0` تا `9` |
| `\w` | کاراکتر کلمه: `[a-zA-Z0-9_]` |
| `\W` | کاراکتری که جزو Word Character نیست |
| `\d` | یک رقم بین `0` تا `9` |
| `\D` | هر کاراکتری که رقم نیست |
| `\s` | کاراکتر فاصله: Space، Tab یا Newline |
| `\S` | کاراکتری که Whitespace نیست |

> **نکته:** در C++ بهتر است از **Raw String**ها مانند `R"(pattern)"` استفاده کنید تا مجبور نباشید Backslashها را دوباره Escape کنید.

---

### تکرارکننده‌ها (Quantifiers)

| نحو | توضیح | مثال |
|---|---|---|
| `*` | صفر بار یا بیشتر | `a*` → `""`، `a`، `aa`، `aaa`، ... |
| `?` | صفر یا یک بار (اختیاری) | `a?` → `""` یا `a` |
| `+` | یک بار یا بیشتر | `a+` → `a`، `aa`، `aaa`، ... |
| `{n}` | دقیقاً `n` بار | `a{3}` → `aaa` |
| `{n,}` | حداقل `n` بار | `a{2,}` → `aa`، `aaa`، ... |
| `{n,m}` | بین `n` و `m` بار | `a{2,3}` → `aa`، `aaa` |

---

### لنگرها و مرزها (Anchors & Boundaries)

| نماد | توضیح |
|---|---|
| `^` | ابتدای رشته |
| `$` | انتهای رشته |
| `\b` | مرز یک کلمه |
| `\B` | جایی که مرز کلمه نیست |

---

### گروه‌بندی و انتخاب (Grouping & Alternation)

| نحو | توضیح |
|---|---|
| `(abc)` | گروه Capture کننده |
| `(?:abc)` | گروه Non-Capturing |
| `a\|b` | یا `a` یا `b` را مطابقت می‌دهد |

> **توجه:** پشتیبانی از برخی ساختارهای Regex در `std::regex` به نحو انتخاب‌شده وابسته است؛ بنابراین قبل از استفاده از قابلیت‌هایی مثل گروه‌های Non-Capturing، باید سازگاری آن‌ها با ECMAScript موردنظر C++ را بررسی کنید.

---

# `std::regex`

عبارات باقاعده به شما اجازه می‌دهند رشته‌ها را بر اساس **الگو (Pattern)** جست‌وجو، تطبیق و دستکاری کنید؛ نه صرفاً بر اساس کاراکترهای ثابت.

از Regex می‌توان برای کارهای مختلفی استفاده کرد، از جمله:

- اعتبارسنجی ایمیل یا شماره تلفن
- استخراج کلمات یا اعداد از یک متن
- جایگزینی الگوهای خاص در متن
- پردازش Logها
- پردازش فایل‌های Configuration
- پردازش حتی بخشی از کدهای برنامه

برای شروع، هدرهای زیر را اضافه کنید:

```cpp
#include <regex>
#include <string>
#include <iostream>
```

---

## انواع و توابع اصلی `<regex>`

| نوع / تابع | کاربرد |
|---|---|
| `std::regex` | نگهداری الگوی Regular Expression |
| `std::smatch` | نگهداری نتایج Match برای `std::string` |
| `std::cmatch` | نگهداری نتایج Match برای رشته‌های C-style مانند `const char*` |
| `std::regex_match` | بررسی می‌کند که آیا کل رشته با الگو مطابقت دارد |
| `std::regex_search` | به دنبال یک تطبیق در بخشی از رشته می‌گردد |
| `std::regex_replace` | بخش‌های Match شده را با متن جدید جایگزین می‌کند |

---

# `regex_match` در برابر `regex_search`

این دو تابع شباهت زیادی دارند، اما یک تفاوت مهم بین آن‌ها وجود دارد:

- `regex_search` به دنبال یک Match **در هر بخشی از رشته** می‌گردد.
- `regex_match` بررسی می‌کند که **کل رشته** با الگو مطابقت داشته باشد.

### مثال: `regex_search`

در این مثال، شماره تلفن می‌تواند در هر جای متن قرار داشته باشد:

```cpp
std::string text = "Call me at 555-1234";

std::regex pattern(R"(\d{3}-\d{4})");

std::smatch match;

if (std::regex_search(text, match, pattern)) {
    std::cout << "Phone found: " << match[0] << "\n";
}
```

خروجی:

```text
Phone found: 555-1234
```

---

### مثال: `regex_match`

در اینجا کل رشته باید با الگو مطابقت داشته باشد:

```cpp
std::string input = "abc123";

std::regex pattern(R"([a-z]+[0-9]+)");

if (std::regex_match(input, pattern)) {
    std::cout << "Valid format\n";
}
```

---

# `regex_replace`

از `regex_replace` برای جایگزین کردن بخش‌هایی از متن که با یک الگو Match می‌شوند استفاده می‌کنیم.

برای مثال، می‌توانیم تگ‌های HTML را حذف کنیم:

```cpp
std::string html = "<b>Bold</b>";

std::regex tag(R"(<.*?>)");

std::string plain = std::regex_replace(html, tag, "");

std::cout << plain;
```

خروجی:

```text
Bold
```

---

# `std::regex_iterator`

`std::regex_iterator` زمانی مفید است که بخواهیم اطلاعات دقیق‌تری درباره‌ی Matchها و Sub-Matchها به دست بیاوریم.

برای مثال:

```cpp
const std::string input = "ABC:1->   PQR:2;;;   XYZ:3<<<";

const std::regex r(R"((\w+):(\d+))");

const std::vector<std::smatch> matches{
    std::sregex_iterator{input.begin(), input.end(), r},
    std::sregex_iterator{}
};

assert(
    matches[0].str(0) == "ABC:1" &&
    matches[0].str(1) == "ABC" &&
    matches[0].str(2) == "1"
);

assert(
    matches[1].str(0) == "PQR:2" &&
    matches[1].str(1) == "PQR" &&
    matches[1].str(2) == "2"
);

assert(
    matches[2].str(0) == "XYZ:3" &&
    matches[2].str(1) == "XYZ" &&
    matches[2].str(2) == "3"
);
```

در C++11 محدودیتی وجود داشت که نمی‌شد `std::regex_iterator` را با یک Regex موقت (Temporary Regex Object) استفاده کرد. این محدودیت در C++14 با اضافه شدن Overload مربوطه برطرف شد.

---

# `std::regex_token_iterator`

`std::regex_token_iterator` ابزاری است که در بسیاری از سناریوهای عملی می‌تواند بسیار کاربردی باشد.

تفاوت اصلی بین `std::regex_iterator` و `std::regex_token_iterator` این است:

- `std::regex_iterator` به **نتایج Match** اشاره می‌کند.
- `std::regex_token_iterator` به **Sub-Matchها (Capture Groupها)** دسترسی می‌دهد.
- هر Iterator در `std::regex_token_iterator` یک نتیجه‌ی Token/Match را در اختیار شما قرار می‌دهد.

مثلاً:

```cpp
const std::string input = "ABC:1->   PQR:2;;;   XYZ:3<<<";

const std::regex r(R"((\w+):(\d+))");
```

## دریافت کل Match

با استفاده از Token شماره‌ی `0`، کل Regex Match شده دریافت می‌شود:

```cpp
const std::vector<std::string> full_match{
    std::sregex_token_iterator{input.begin(), input.end(), r, 0},
    std::sregex_token_iterator{}
};

assert(
    (full_match == std::vector<std::string>{
        "ABC:1",
        "PQR:2",
        "XYZ:3"
    })
);
```

---

## دریافت Capture Group اول

با استفاده از Token شماره‌ی `1`، اولین Capture Group دریافت می‌شود:

```cpp
const std::vector<std::string> capture_group_1{
    std::sregex_token_iterator{input.begin(), input.end(), r, 1},
    std::sregex_token_iterator{}
};

assert(
    (capture_group_1 == std::vector<std::string>{
        "ABC",
        "PQR",
        "XYZ"
    })
);
```

---

## دریافت Capture Group دوم

با استفاده از Token شماره‌ی `2`، دومین Capture Group دریافت می‌شود:

```cpp
const std::vector<std::string> capture_group_2{
    std::sregex_token_iterator{input.begin(), input.end(), r, 2},
    std::sregex_token_iterator{}
};

assert(
    (capture_group_2 == std::vector<std::string>{
        "1",
        "2",
        "3"
    })
);
```

---

# Match معکوس با `std::regex_token_iterator`

یک قابلیت جالب دیگر این است که می‌توانیم به جای بخش‌های Match شده، قسمت‌هایی را دریافت کنیم که **Match نشده‌اند**.

برای این کار از Token برابر `-1` استفاده می‌کنیم:

```cpp
const std::string input = "ABC:1->   PQR:2;;;   XYZ:3<<<";

const std::regex r(R"((\w+):(\d+))");

const std::vector<std::string> inverted{
    std::sregex_token_iterator{
        input.begin(),
        input.end(),
        r,
        -1
    },
    std::sregex_token_iterator{}
};

assert(
    (inverted == std::vector<std::string>{
        "",
        "->   ",
        ";;;   ",
        "<<<"
    })
);
```

در اینجا:

```cpp
-1
```

به معنی **بخش‌هایی از رشته است که با Regex مطابقت ندارند**.

---

# مثال‌های واقعی

## اعتبارسنجی ایمیل

می‌توانیم از Regex برای بررسی یک فرمت ساده‌ی ایمیل استفاده کنیم:

```cpp
std::regex email(
    R"((\w+)(\.\w+)@\w+\.\w+)"
);

std::string s = "user.name@example.com";

std::smatch m;

if (std::regex_match(s, m, email)) {
    std::cout << "Valid email!\n";
}
```

> **نکته:** Regex برای اعتبارسنجی ایمیل معمولاً فقط یک بررسی ساده‌ی فرمت انجام می‌دهد و نمی‌تواند تمام قواعد ایمیل‌های واقعی را پوشش دهد.

---

## استخراج تمام کلمات از یک جمله

می‌توانیم با `std::sregex_iterator` تمام کلمات یک جمله را پیدا کنیم:

```cpp
std::string sentence = "C++ is powerful and fast";

std::regex word(R"(\w+)");

auto begin = std::sregex_iterator(
    sentence.begin(),
    sentence.end(),
    word
);

auto end = std::sregex_iterator();

for (auto it = begin; it != end; ++it) {
    std::cout << "Word: " << it->str() << "\n";
}
```

خروجی:

```text
Word: C
Word: is
Word: powerful
Word: and
Word: fast
```

توجه کنید که `\w+` در این مثال علامت `+` را بخشی از کلمه در نظر نمی‌گیرد.

---

## جایگزینی URLها با یک Placeholder

می‌توانیم URLهای موجود در متن را با یک عبارت جایگزین کنیم:

```cpp
std::string input =
    "Visit http://example.com now!";

std::regex url(
    R"(https?://\S+)"
);

std::string clean =
    std::regex_replace(input, url, "[link]");

std::cout << clean;
```

خروجی:

```text
Visit [link] now!
```

---

# مدیریت خطاها

اگر الگوی Regex نامعتبر باشد، ساختن `std::regex` می‌تواند باعث پرتاب شدن `std::regex_error` شود.

برای مدیریت این خطا می‌توانیم از `try/catch` استفاده کنیم:

```cpp
try {
    std::regex broken("[A-Z");
}
catch (const std::regex_error& e) {
    std::cerr << "Regex error: "
              << e.what()
              << '\n';
}
```

در این مثال، `]` پایانی وجود ندارد، بنابراین Regex نامعتبر است.

---

# استفاده از Flagهای Regex

هنگام ساختن Regex می‌توانیم Syntax یا Flagهای مختلفی را مشخص کنیم.

برای مثال، `icase` باعث می‌شود حروف بزرگ و کوچک تفاوتی نداشته باشند:

```cpp
std::regex pattern(
    "hello",
    std::regex_constants::icase
);
```

بنابراین این Regex می‌تواند موارد زیر را Match کند:

```text
hello
Hello
HELLO
HeLLo
```

---

## Syntaxها و Flagهای مهم

| Flag / Syntax | توضیح |
|---|---|
| `std::regex::ECMAScript` | Syntax پیش‌فرض، مشابه JavaScript |
| `std::regex::basic` | POSIX Basic Regular Expression |
| `std::regex::extended` | POSIX Extended Regular Expression |
| `std::regex_constants::icase` | عدم حساسیت به حروف بزرگ و کوچک |

> **نکته:** `ECMAScript` Syntax پیش‌فرض `std::regex` است.

---

# جمع‌بندی

کتابخانه‌ی استاندارد C++ ابزارهای مناسبی برای کار با Regular Expression در اختیار ما قرار می‌دهد.

مهم‌ترین ابزارهایی که باید به خاطر داشته باشیم عبارت‌اند از:

| ابزار | کاربرد |
|---|---|
| `std::regex` | تعریف و نگهداری Regex |
| `std::regex_match` | بررسی تطبیق کل رشته |
| `std::regex_search` | پیدا کردن Match در بخشی از رشته |
| `std::regex_replace` | جایگزینی Matchها |
| `std::regex_iterator` | پیمایش Matchها |
| `std::regex_token_iterator` | پیمایش Matchها یا Capture Groupها |
| `std::smatch` | نگهداری نتایج Match برای `std::string` |
| `std::regex_error` | مدیریت Regexهای نامعتبر |

اگر با C++ کار می‌کنید، دانستن تفاوت **`regex_match` و `regex_search`** و همچنین نحوه‌ی استفاده از **`regex_iterator` و `regex_token_iterator`** بخش مهمی از کار با `std::regex` است.

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | m.roodsarabi76@gmail.com | | [mbr](https://t.me/ad1mi2n) |

</div>