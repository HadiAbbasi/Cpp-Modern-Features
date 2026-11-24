<div align="center">

[🇺🇸 English](../../en/cpp23/unreachable.md) | [🇮🇷 فارسی](./unreachable.md)

</div>

#  std::unreachable


تابع **`std::unreachable()`** یکی از قابلیت‌های کاربردی و بهینه‌سازی در **C++23** است که در هدر `<utility>` تعریف شده است.

این تابع به کامپایلر اعلام می‌کند که **«اجرای برنامه هرگز نباید به این نقطه از کد برسد»**. به عبارت دیگر، رسیدن جریان اجرای برنامه به این خط، یک **رفتار تعریف‌نشده (Undefined Behavior یا UB)** محسوب می‌شود.

---

## چرا به `std::unreachable()` نیاز داریم؟

در نسخه‌های قبلی C++، برای این که به کامپایلر بگوییم یک نقطه غیرقابل دسترس است، از دستورات وابسته به کامپایلر (Vendor-specific) استفاده می‌کردیم:
- در **GCC / Clang:** `__builtin_unreachable()`
- در **MSVC:** `__assume(false)`

در C++23، `std::unreachable()` یک استاندارد رسمی و قابل حمل (Portable) برای همین کار ارائه داده است.

---

## کاربردهای اصلی

### الف) بهینه‌سازی ساختار `switch` (حذف چک‌های اضافی)
فرض کنید یک `enum` دارید و مطمئن هستید مقادیر آن محدود به موارد مشخصی است:

```cpp
#include <utility>
#include <iostream>

enum class Direction {
    North,
    South,
    East,
    West
};

const char* get_direction_name(Direction dir) {
    switch (dir) {
        case Direction::North: return "North";
        case Direction::South: return "South";
        case Direction::East:  return "East";
        case Direction::West:  return "West";
    }
    // به کامپایلر می‌گوییم هرگز مقداری غیر از ۴ حالت بالا وارد نمی‌شود
    std::unreachable();
}
```

**مزیت:**
کامپایلر دیگر کد اضافی یا شاخه پیش‌فرض (`default: ...`) برای پرتاب خطا یا مقدار بازگشتی ساختگی تولید نمی‌کند و هشدارهایی مثل *"control reaches end of non-void function"* خاموش می‌شوند.

---

### ب) بهینه‌سازی الگوریتم‌ها و شروط ریاضی
وقتی به صورت منطقی می‌دانید شرایط خاصی محال است اتفاق بیفتد:

```cpp
#include <utility>

int divide_by_power_of_two(int x, int p) {
    if (p <= 0 || p > 30) {
        std::unreachable(); // فرض قطعی: توان همیشه بین ۱ تا ۳۰ است
    }
    return x / (1 << p);
}
```
کامپایلر با این فرض، می‌تواند بهینه‌سازی‌های عمیق‌تری در سطح اسمبلی (مانند برداری‌سازی یا ساده‌سازی محاسبات) انجام دهد.

---

##  تفاوت با `assert` چیست؟

| ویژگی | `assert(...)` | `std::unreachable()` |
| :--- | :--- | :--- |
| **هدف اصلی** | خطایابی و دیباگ (Sanity check) | بهینه‌سازی کامپایلر (Compiler Hint) |
| **رفتار در حالت Release** | معمولاً غیرفعال می‌شود (`-DNDEBUG`) | همیشه فعال است و کامپایلر بر پایه آن کد را بهینه می‌کند |
| **رفتار در صورت وقوع** | برنامه با پیام خطا متوقف (abort) می‌شود | ایجاد **Undefined Behavior** (امکان کرش، تخریب حافظه، یا رفتارهای عجیب) |

> 💡 **نکته طلایی برای توسعه امن:**
> در محیط دیباگ می‌توانید از ترکیب هر دو استفاده کنید تا قبل از تولید نسخه نهایی از عدم رسیدن به آن نقطه مطمئن شوید:
> ```cpp
> #include <cassert>
> #include <utility>
> 
> void do_something(int x) {
>     if (x < 0) {
>         assert(false && "x must be non-negative");
>         std::unreachable();
>     }
>     // ...
> }
> ```


## جمع‌بندی
- **هدر:** `#include <utility>`
- **نقش:** راهنمای صریح به کامپایلر برای حذف شاخه‌های ناممکن و بهینه‌سازی حجم و سرعت کد.
- **هشدار:** فقط زمانی استفاده کنید که ۱۰۰٪ مطمئن هستید مسیر اجرا به آنجا نخواهد رسید، چرا که اجرای آن مساوی با Undefined Behavior است.

--- 
## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>