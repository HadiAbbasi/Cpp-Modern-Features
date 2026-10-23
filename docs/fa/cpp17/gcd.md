<div align="center">

[🇺🇸 English](../../en/cpp17/gcd.md) | [🇮🇷 فارسی](./gcd.md)

</div>

---

# std:gcd

تابع **`std::gcd`** یکی از توابع ریاضی استاندارد و بسیار کاربردی است که در **C++17** به کتابخانه استاندارد اضافه شد. 

نام `gcd` مخفف **Greatest Common Divisor** یا همان **«بزرگ‌ترین مقسوم‌علیه مشترک» (ب.م.م)** است. این تابع بزرگ‌ترین عدد صحیحی را پیدا می‌کند که هر دو عدد ورودی را بدون باقی‌مانده تقسیم کند.


##  هدر و سینتکس (Header & Syntax)

برای استفاده از `std::gcd` باید هدر **`<numeric>`** را وارد کنید:

```cpp
#include <numeric>
```

 **سینتکس:**
```cpp
namespace std {
    template< class M, class N >
    constexpr std::common_type_t<M, N> gcd( M m, N n );
}
```

* **ورودی‌ها (`m` و `n`):** حتماً باید از نوع **اعداد صحیح (Integer types)** باشند (مانند `int`, `long`, `short`, `unsigned int`).
* **خروجی:** مقداری از نوع مشترک آن دو عدد برمی‌گرداند.
* **`constexpr`:** این تابع `constexpr` است، یعنی اگر ورودی‌ها در زمان کامپایل معلوم باشند، نتیجه در همان زمان کامپایل محاسبه می‌شود.


## مثال ساده و کاربردی

```cpp
#include <iostream>
#include <numeric>

int main() {
    int a = 24;
    int b = 36;

    // مقسوم‌علیه‌های ۲۴: 1, 2, 3, 4, 6, 8, 12, 24
    // مقسوم‌علیه‌های ۳۶: 1, 2, 3, 4, 6, 9, 12, 18, 36
    // بزرگ‌ترین مشترک بین آن‌ها: 12

    int result = std::gcd(a, b);

    std::cout << "GCD of " << a << " and " << b << " is: " << result << '\n';
    // خروجی: GCD of 24 and 36 is: 12
}
```

## رفتار با اعداد منفی و صفر (قوانین مهم)

1. **اعداد منفی:** خروجی `std::gcd` همیشه **نامنفی (مثبت یا صفر)** است، حتی اگر یکی یا هر دو عدد منفی باشند:
   ```cpp
   std::gcd(-24, 36);  // خروجی: 12
   std::gcd(-24, -36); // خروجی: 12
   ```

2. **عدد صفر:**
   * اگر یکی از اعداد صفر باشد، خروجی قدر مطلق عدد دیگر است:
     ```cpp
     std::gcd(0, 15); // خروجی: 15
     ```
   * اگر هر دو عدد صفر باشند، خروجی صفر است:
     ```cpp
     std::gcd(0, 0);  // خروجی: 0
     ```

## کاربرد عملی: ساده کردن کسرها (Simplifying Fractions)

یکی از متداول‌ترین کاربردهای ب.م.م، ساده کردن صورت و مخرج کسرهاست:

```cpp
#include <iostream>
#include <numeric>

struct Fraction {
    int num; // صورت
    int den; // مخرج

    void simplify() {
        int common = std::gcd(num, den);
        if (common != 0) {
            num /= common;
            den /= common;
        }
    }
};

int main() {
    Fraction f{42, 56}; // کسر 42/56

    f.simplify();

    std::cout << "Simplified: " << f.num << "/" << f.den << '\n';
    // خروجی: Simplified: 3/4
}
```

##  کاربرد در زمان کامپایل (`constexpr`)

چون این تابع `constexpr` است، می‌توانید از آن برای اندازه‌دهی به آرایه‌ها یا ثابت‌های زمان کامپایل استفاده کنید:

```cpp
#include <numeric>

constexpr int buffer_size = std::gcd(1024, 768); // در زمان کامپایل محاسبه می‌شود: 256
int buffer[buffer_size]; // آرایه‌ای با طول 256
```

## تابع مکمل: `std::lcm` (ک.م.م)

در کنار `std::gcd`، استاندارد C++17 تابع **`std::lcm`** (**Least Common Multiple** یا **ک.م.م - کوچک‌ترین مضرب مشترک**) را نیز در همان هدر `<numeric>` قرار داده است:

```cpp
#include <iostream>
#include <numeric>

int main() {
    int a = 4;
    int b = 6;

    std::cout << "GCD: " << std::gcd(a, b) << '\n'; // ب.م.م: 2
    std::cout << "LCM: " << std::lcm(a, b) << '\n'; // ک.م.م: 12
}
```

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>
