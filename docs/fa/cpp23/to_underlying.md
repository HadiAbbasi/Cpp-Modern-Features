<div align="center">

[🇺🇸 English](../../en/cpp23/to_underlying.md) | [🇮🇷 فارسی](./to_underlying.md)

</div>

# std::to_underlying

تابع **`std::to_underlying()`** یکی از امکانات بسیار کاربردی و تمیز اضافه شده در **C++23** است که در هدر `<utility>` قرار دارد.

وظیفه این تابع ساده اما بسیار پرکاربرد است: **تبدیل یک مقدار از نوع `enum` به نوع داده‌ی زیرین آن (Underlying Type) به شکلی ایمن، خوانا و بدون نیاز به کست‌های طولانی.**



## مشکل قبلی چی بود؟ (قبل از C++23)

در C++11 و بعد از آن، با معرفی `enum class` (Scoped Enums)، نوع‌های شمارشی ایمن‌تر شدند، اما تبدیل آن‌ها به مقادیر عددی زیرین کمی طولانی و نازیبا بود.

قبلاً معمولاً یکی از دو روش زیر استفاده می‌شد:

```cpp
enum class Status : uint8_t {
    OK = 0,
    NotFound = 404,
    Error = 500
};

// روش ۱: static_cast مستقیم (باید نوع زیرین را دستی تکرار کنید)
auto val1 = static_cast<uint8_t>(Status::OK);

// روش ۲: استفاده از std::underlying_type_t (خوانایی بسیار پایین)
auto val2 = static_cast<std::underlying_type_t<Status>>(Status::OK);
```

**معایب روش‌های قدیمی:**
- **تکرار کد و خطا:** اگر نوع پایه `enum` را بعدها از `uint8_t` به `uint16_t` تغییر می‌دادید، در `static_cast<uint8_t>` ممکن بود مقادیر truncate یا ناقص شوند.
- **طولانی بودن:** استفاده از `std::underlying_type_t` خط کدهای شلوغ و خسته‌کننده‌ای ایجاد می‌کرد.



## راه‌حل مدرن در C++23: `std::to_underlying`

در C++23، به سادگی می‌نویسید:

```cpp
#include <iostream>
#include <utility>
#include <cstdint>

enum class ErrorCode : uint16_t {
    Success = 0,
    Timeout = 1001,
    AccessDenied = 1002
};

int main() {
    ErrorCode code = ErrorCode::Timeout;

    // تبدیل تمیز و خودکار به نوع uint16_t
    auto raw_value = std::to_underlying(code);

    std::cout << "Error raw number: " << raw_value << '\n';
    std::cout << "Size of type: " << sizeof(raw_value) << " bytes\n"; // خروجی: 2 بایت
}
```

## پیاده‌سازی داخلی آن چگونه است؟

پیاده‌سازی آن در کتابخانه استاندارد در اصل یک تابع کمکی (Helper) ساده و `constexpr` است، شبیه به این:

```cpp
template <typename Enum>
constexpr std::underlying_type_t<Enum> to_underlying(Enum e) noexcept {
    return static_cast<std::underlying_type_t<Enum>>(e);
}
```

ویژگی‌های کلیدی این تابع:
- **`constexpr` بودن:** کاملاً در زمان کامپایل قابل ارزیابی است.
- **`noexcept` بودن:** هیچ استثنایی (Exception) پرتاب نمی‌کند.
- **بررسی نوع (Type Safety):** فقط برای ورودی‌هایی که واقعاً از نوع `enum` یا `enum class` هستند کار می‌کند؛ اگر چیز دیگری به آن پاس بدهید، خطای زمان کامپایل دریافت می‌کنید.


## کاربردهای رایج

### الف) کار با پرچم‌های بیتی (Bitwise Flags)
```cpp
enum class Permissions : uint8_t {
    Read    = 1 << 0,
    Write   = 1 << 1,
    Execute = 1 << 2
};

uint8_t mask = std::to_underlying(Permissions::Read) | std::to_underlying(Permissions::Write);
```

### ب) سریال‌سازی (Serialization) و ارسال روی شبکه
هنگام ذخیره در فایل، نوشتن در بافر شبکه یا ارسال به API‌های C که نیاز به اعداد صحیح خام (Raw Integers) دارند.


## خلاصه
- **هدر:** `#include <utility>`
- **نقش:** تبدیل یک `enum` یا `enum class` به نوع عددیِ زیربنایی‌اش بدون نیاز به مشخص کردن دستی نوع.
- **مزیت:** خوانایی بهتر، جلوگیری از خطاهای ناشی از تغییر نوع `enum` و کاملاً `constexpr`.

--- 
## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>