<div align="right">

[🇺🇸 English](../../en/cpp11/static_assert.md) | [🇮🇷 فارسی](./static_assert.md)

</div>



# `static_assert` در ++C

`static_assert` یک **مکانیزم بررسی (Assertion) در زمان کامپایل** است که از استاندارد **C++11** معرفی شد. این دستور یک شرط را هنگام کامپایل بررسی می‌کند و اگر شرط برقرار نباشد، کامپایلر با نمایش یک پیام خطای سفارشی، از کامپایل برنامه جلوگیری می‌کند.

## مزایا

- بررسی صحت شرایط در **زمان کامپایل**.
- نمایش پیام خطای دلخواه در صورت برقرار نبودن شرط.
- شناسایی خطاهای برنامه‌نویسی قبل از اجرای برنامه.

---

# ساختار (Syntax)

```cpp
static_assert(constant_expression, "error message");
```

### اجزای دستور

- **`constant_expression`**: یک عبارت ثابت (Compile-Time Expression) که مقدار آن باید `true` یا `false` باشد.
- **`"error message"`**: پیامی که در صورت نادرست بودن شرط نمایش داده می‌شود.

---

# مثال ساده

```cpp
#include <iostream>
using namespace std;

int main()
{
    static_assert(sizeof(int) >= 4,
                  "int must be at least 4 bytes");

    cout << "Compilation successful";

    return 0;
}
```

### توضیح

کامپایلر اندازه‌ی نوع `int` را بررسی می‌کند. اگر اندازه‌ی آن کمتر از ۴ بایت باشد، برنامه کامپایل نمی‌شود و پیام زیر نمایش داده می‌شود:

```text
int must be at least 4 bytes
```

---

# بررسی در زمان کامپایل قبل از C++11

قبل از معرفی `static_assert` معمولاً از دستور پیش‌پردازنده‌ی `#error` استفاده می‌شد.

```cpp
#include <iostream>
using namespace std;

#if !defined(__mbrcode)
#error "mbrCode hasn't been defined yet"
#endif

int main()
{
    return 0;
}
```

### محدودیت `#error`

`#error` فقط وجود یا عدم وجود ماکروها را بررسی می‌کند و **نمی‌تواند** عباراتی مانند موارد زیر را ارزیابی کند:

- `sizeof()`
- پارامترهای Template
- ویژگی‌های نوع (`type traits`)
- محاسبات زمان کامپایل

به همین دلیل `static_assert` جایگزین قدرتمندتری محسوب می‌شود.

---

# استفاده از `static_assert` در Template

یکی از رایج‌ترین کاربردهای `static_assert` بررسی صحت پارامترهای Template است.

```cpp
#include <iostream>
using namespace std;

template <class T, int Size>
class Vector
{
    static_assert(Size > 3,
                  "Vector size is too small!");

    T values[Size];
};

int main()
{
    Vector<int, 4> four;      // صحیح
    Vector<short, 2> two;     // خطای کامپایل

    return 0;
}
```

### توضیح

شرط زیر بررسی می‌شود:

```cpp
Size > 3
```

اگر اندازه‌ی بردار کمتر از ۴ باشد، کامپایلر پیام زیر را نمایش می‌دهد:

```text
Vector size is too small!
```

---

# مزایای `static_assert`

- بررسی فرضیات برنامه در زمان کامپایل
- تولید پیام‌های خطای واضح و قابل شخصی‌سازی
- پشتیبانی از Templateها
- قابلیت استفاده با `sizeof`
- سازگار با Type Traits و پارامترهای Template
- جلوگیری از بسیاری از خطاها قبل از اجرای برنامه

---

# محدوده (Scope) استفاده از `static_assert`

دستور `static_assert` را می‌توان در بخش‌های مختلف برنامه استفاده کرد.

## ۱. محدوده Namespace

در این حالت، شرط برای کل فایل بررسی می‌شود.

```cpp
#include <iostream>
using namespace std;

static_assert(sizeof(void*) == 8,
              "64-bit architecture required");

int main()
{
    cout << "Assertion passed";

    return 0;
}
```

### کاربرد

مثلاً اطمینان از اینکه برنامه فقط روی سیستم ۶۴ بیتی کامپایل شود.

---

## ۲. محدوده کلاس (Class Scope)

می‌توان داخل کلاس یا Template قرار داد تا محدودیت‌هایی روی تعریف کلاس اعمال شود.

```cpp
template <class T, int Size>
class Vector
{
    static_assert(Size > 3,
                  "Vector size is too small");

    T values[Size];
};
```

در این مثال، هیچ شیئی از `Vector` با اندازه‌ی کمتر از ۴ ساخته نخواهد شد.

---

## ۳. محدوده بلاک یا تابع (Block Scope)

داخل توابع نیز می‌توان از آن استفاده کرد.

```cpp
template <typename T, int N>
void func()
{
    static_assert(N >= 0,
                  "Array size cannot be negative");

    T arr[N];
}
```

در اینجا، اگر مقدار `N` منفی باشد، تابع اصلاً کامپایل نخواهد شد.

---

# عبارات نامعتبر در `static_assert`

عبارت داخل `static_assert` باید **در زمان کامپایل قابل ارزیابی** باشد.

### مثال نادرست

```cpp
int main()
{
    static_assert(1 / 0,
                  "never shows up!");

    return 0;
}
```

### دلیل خطا

عبارت `1 / 0` یک عبارت ثابت معتبر نیست و در زمان کامپایل باعث خطا می‌شود؛ بنابراین برنامه قبل از نمایش پیام سفارشی متوقف خواهد شد.

---

# تفاوت `static_assert` و `assert`

| ویژگی | `static_assert` | `assert` |
|--------|-----------------|----------|
| زمان بررسی | زمان کامپایل | زمان اجرا |
| نیاز به اجرای برنامه | ❌ | ✅ |
| شرط باید ثابت باشد | ✅ | ❌ |
| مناسب برای Template | ✅ | ❌ |
| جلوگیری از کامپایل | ✅ | ❌ |

### مثال `assert`

```cpp
#include <cassert>

int main()
{
    int x = 5;

    assert(x > 0);   // هنگام اجرا بررسی می‌شود
}
```

در حالی که:

```cpp
static_assert(sizeof(int) == 4,
              "Invalid int size");
```

قبل از اجرای برنامه بررسی می‌شود.

---

# چه زمانی از `static_assert` استفاده کنیم؟

از `static_assert` در موارد زیر استفاده کنید:

- بررسی اندازه‌ی انواع داده (`sizeof`)
- محدود کردن پارامترهای Template
- بررسی ویژگی‌های نوع (`std::is_same` و سایر Type Traits)
- اطمینان از معماری یا ویژگی‌های کامپایلر
- جلوگیری از تعریف کلاس یا تابع با پارامترهای نامعتبر

---

# جمع‌بندی

- `static_assert` از **C++11** معرفی شده است.
- شرط را در **زمان کامپایل** بررسی می‌کند.
- در صورت نادرست بودن شرط، کامپایل متوقف شده و پیام خطای دلخواه نمایش داده می‌شود.
- نسبت به `#error` بسیار قدرتمندتر است، زیرا می‌تواند عبارات ثابت، `sizeof`، Type Traits و پارامترهای Template را ارزیابی کند.
- برای نوشتن کدهای **ایمن، جنریک (Generic)** و **قابل نگهداری** یکی از ابزارهای مهم ++C محسوب می‌شود.

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | m.roodsarabi76@gmail.com | | [mbr](https://t.me/ad1mi2n) |

</div>