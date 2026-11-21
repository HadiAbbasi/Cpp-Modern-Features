<div align="center">

[🇺🇸 English](../../en/cpp23/byteswap.md) | [🇮🇷 فارسی](./byteswap.md)

</div>

# `std::byteswap`

استاندارد C++23 تابع `std::byteswap` را در هدر `<bit>` معرفی کرده است. این تابع روشی استاندارد، قابل‌حمل (Portable) و `constexpr` برای معکوس کردن ترتیب بایت‌های یک مقدار صحیح (Integer) فراهم می‌کند.

این قابلیت هنگام کار با فایل‌های باینری، پروتکل‌های شبکه (تبدیل بین Big-Endian و Little-Endian) و ارتباط با سخت‌افزار بسیار کاربردی است.

## **چرا به&#x20;**`std::byteswap`**&#x20;نیاز داریم؟**

معماری‌های مختلف کامپیوتر، داده‌های چندبایتی را به روش‌های متفاوتی در حافظه ذخیره می‌کنند. دو روش اصلی عبارت‌اند از:

- **Little-Endian:** بایت کم‌ارزش‌تر (Least Significant Byte) در پایین‌ترین آدرس حافظه ذخیره می‌شود. پردازنده‌های x86 و x64 از این روش استفاده می‌کنند.
- **Big-Endian:** بایت پرارزش‌تر (Most Significant Byte) در پایین‌ترین آدرس حافظه ذخیره می‌شود. این روش در ترتیب بایت استاندارد شبکه (Network Byte Order) استفاده می‌شود.

برای مثال، عدد زیر را در نظر بگیرید:

```
uint32_t value = 0x12345678;
```

ترتیب بایت‌های آن به شکل زیر است:

| نوع ترتیب     | ترتیب بایت‌ها |
| ------------- | ------------- |
| Little-Endian | `78 56 34 12` |
| Big-Endian    | `12 34 56 78` |

پیش از C++23، برنامه‌نویسان برای معکوس کردن ترتیب بایت‌ها مجبور بودند از توابع اختصاصی کامپایلر مانند `__builtin_bswap32` در GCC و Clang یا `_byteswap_ulong` در MSVC استفاده کنند یا خودشان عملیات شیفت بیتی را پیاده‌سازی کنند.

تابع `std::byteswap` این قابلیت را به‌صورت استاندارد وارد زبان C++ کرده است.

## سینتکس و هدر موردنیاز

برای استفاده از `std::byteswap` باید هدر `<bit>` را اضافه کنید:

```cpp
#include <bit>
```

امضای تابع در استاندارد C++23:

```cpp
template<class T>
constexpr T byteswap(T n) noexcept;
```

**ویژگی‌های مهم:**

- **نوع داده:** پارامتر `T` باید یک نوع صحیح (Integral Type) باشد؛ مانند `uint16_t`، `uint32_t`، `uint64_t` و انواع صحیح استاندارد دیگر.
- **محدودیت:** انواع ممیز شناور (Floating-Point) مانند `float` و `double` مستقیماً پشتیبانی نمی‌شوند.
- `constexpr`**:** امکان محاسبه در زمان کامپایل (Compile-Time) را فراهم می‌کند.
- `noexcept`**:** تابع هیچ استثنایی (Exception) پرتاب نمی‌کند.
- **محدودیت مهم:** نوع داده باید تعداد بایت‌های مشخصی داشته باشد و اندازه آن باید مضربی از یک بایت باشد؛ همچنین برای انواع صحیح دارای بیت‌های padding، رفتار تابع محدودیت‌هایی دارد.

## مثال کامل استفاده از&#x20;`std::byteswap`

در مثال زیر نحوه استفاده از این تابع برای اعداد ۱۶، ۳۲ و ۶۴ بیتی را مشاهده می‌کنیم.

همچنین از نمایش هگزادسیمال (Hexadecimal) برای مشاهده بهتر ترتیب بایت‌ها استفاده شده است.

```cpp
#include <bit>
#include <iostream>
#include <iomanip>
#include <cstdint>

int main() {

    // 1. Example: 32-bit Integer
    uint32_t original32 = 0x12345678;
    uint32_t swapped32 = std::byteswap(original32);

    std::cout << "--- 32-bit Integer ---" << std::endl;

    std::cout << "Original: 0x"
              << std::hex << std::setfill('0')
              << std::setw(8) << original32 << std::endl;

    std::cout << "Swapped:  0x"
              << std::setw(8) << swapped32 << std::endl;

    // Output: 0x78563412


    // 2. Example: 16-bit Integer
    uint16_t original16 = 0xABCD;
    uint16_t swapped16 = std::byteswap(original16);

    std::cout << "\n--- 16-bit Integer ---" << std::endl;

    std::cout << "Original: 0x"
              << std::setw(4) << original16 << std::endl;

    std::cout << "Swapped:  0x"
              << std::setw(4) << swapped16 << std::endl;

    // Output: 0xCDAB


    // 3. Compile-time Evaluation (constexpr)
    constexpr uint32_t compileTimeSwap =
        std::byteswap(0x01020304U);

    static_assert(compileTimeSwap == 0x04030201U);

    std::cout << "\nCompile-time byteswap verified successfully!"
              << std::endl;

    return 0;
}
```

**خروجی برنامه:**

```txt
--- 32-bit Integer ---
Original: 0x12345678
Swapped:  0x78563412

--- 16-bit Integer ---
Original: 0xabcd
Swapped:  0xcdab

Compile-time byteswap verified successfully!
```

همان‌طور که مشاهده می‌کنید، ترتیب بایت‌ها کاملاً معکوس شده است.

## کاربرد عملی: تبدیل Network Byte Order

در پروتکل‌های شبکه، ترتیب بایت‌ها معمولاً از استاندارد Big-Endian پیروی می‌کند که با نام Network Byte Order نیز شناخته می‌شود.

اگر سیستم شما Little-Endian باشد، برای تبدیل داده دریافتی از شبکه به ترتیب بایت میزبان (Host Byte Order)، باید بایت‌ها را معکوس کنید.

در C++23 می‌توانیم از `std::endian` و `std::byteswap` در کنار یکدیگر استفاده کنیم.

```cpp
#include <bit>
#include <iostream>
#include <cstdint>

uint32_t readNetworkData(uint32_t netValue) {

    // Check native system endianness
    if constexpr (std::endian::native == std::endian::little) {

        // Convert Big-Endian to Little-Endian
        return std::byteswap(netValue);

    } else {

        // No conversion needed
        return netValue;
    }
}

int main() {

    uint32_t packetData = 0x01020304;

    uint32_t hostValue = readNetworkData(packetData);

    std::cout << "Host value: 0x"
              << std::hex << hostValue << std::endl;

    return 0;
}
```

**نکته مهم:** در این مثال فرض شده است که مقدار دریافتی شبکه قبلاً به‌صورت عددی در متغیر `netValue` قرار گرفته و ترتیب بایت‌های آن با ترتیب بایت میزبان تفسیر شده است. در کد واقعی شبکه، باید هنگام خواندن بایت‌ها از بافر نیز دقت کنید تا تبدیل دوبار انجام نشود.

همچنین `std::endian::native` می‌تواند در برخی معماری‌ها مقداری غیر از `little` یا `big` داشته باشد؛ بنابراین برای کدهای قابل‌حمل‌تر باید این حالت نیز در نظر گرفته شود.

## تفاوت&#x20;`std::byteswap`&#x20;با&#x20;`std::endian`

این دو قابلیت در C++20 و C++23 مکمل یکدیگر هستند:

| قابلیت          | کاربرد                             |
| --------------- | ---------------------------------- |
| `std::endian`   | تشخیص ترتیب بایت‌های سیستم         |
| `std::byteswap` | معکوس کردن ترتیب بایت‌های یک مقدار |

برای مثال:

```cpp
#include <bit>
#include <cstdint>

int main() {

    constexpr uint32_t value = 0x12345678;

    constexpr auto swapped = std::byteswap(value);

    static_assert(swapped == 0x78563412);

}
```

## نکات کلیدی

1. **هدر موردنیاز:** برای استفاده از `std::byteswap` باید `<bit>` را اضافه کنید.
2. **قابلیت حمل (Portability):** این تابع جایگزین استانداردی برای توابع اختصاصی کامپایلر مانند `__builtin_bswap32` و `_byteswap_ulong` است.
3. **کارایی (Performance):** کامپایلرها می‌توانند این تابع را به دستورالعمل‌های بهینه سخت‌افزاری مانند `BSWAP` در معماری x86 تبدیل کنند.
4. **پشتیبانی از&#x20;**`constexpr`**:** امکان اجرای عملیات در زمان کامپایل و استفاده از آن در `static_assert` وجود دارد.
5. **نوع داده:** این تابع برای انواع صحیح (Integer Types) طراحی شده است و از `float` و `double` مستقیماً پشتیبانی نمی‌کند.
6. **استاندارد:** این قابلیت از C++23 در دسترس است.

**جمع‌بندی:**

`std::byteswap` یکی از قابلیت‌های کوچک اما کاربردی C++23 است که عملیات معکوس کردن ترتیب بایت‌ها را ساده‌تر، خواناتر و قابل‌حمل‌تر می‌کند. اگر با فایل‌های باینری، پروتکل‌های شبکه یا داده‌های سطح پایین سروکار دارید، آشنایی با این تابع و ترکیب آن با `std::endian` می‌تواند کدنویسی شما را بسیار تمیزتر کند.

--- 
## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>