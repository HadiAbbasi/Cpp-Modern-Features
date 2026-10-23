<div align="center">

[🇺🇸 English](../../en/cpp17/byte.md) | [🇮🇷 فارسی](./byte.md)

</div>

---

# std::byte

نوع داده‌ی **`std::byte`** یکی از ویژگی‌های مهم و کاربردی معرفی‌شده در **C++17** است که برای نشان دادن **یک بایت از داده‌های خام حافظه (Raw Memory)** طراحی شده است.

در ادامه، چرایی معرفی آن، نحوه‌ی استفاده و مثال‌های عملی را به صورت کامل بررسی می‌کنیم.

---

##  چرا به `std::byte` نیاز داشتیم؟

پیش از C++17، برنامه‌نویسان برای کار با بایت‌های خام حافظه معمولاً از انواع زیر استفاده می‌کردند:
* `char`
* `unsigned char`
* `uint8_t`

**مشکل کجاست؟**
تمام این نوع‌ها در اصل نوع‌های **عددی (Arithmetic)** یا **کاراکتری** هستند:
1. `char` برای نمایش کاراکترهای متنی طراحی شده است (`'A'`, `'b'`).
2. `unsigned char` و `uint8_t` نوع عددی بدون علامت هستند؛ یعنی کامپایلر به شما اجازه می‌دهد عملگرهایی مانند جمع (`+`)، تفریق (`-`) یا ضرب (`*`) را رویشان انجام دهید.

اما **یک بایت حافظه خام نه کاراکتر است و نه عدد**؛ بلکه فقط مجموعه‌ای از بیت‌ها (Bit pattern) است. مفهوم ریاضیاتی مثل "ضرب دو بایت حافظه در هم" بی‌معنا است.

برای حل این مشکل، `std::byte` معرفی شد تا **امنیت نوع (Type-safety)** را تضمین کند:
* **نمی‌توانید** با آن عملیات ریاضی (`+`, `-`, `*`) انجام دهید.
* **نمی‌توانید** بدون تبدیل صریح آن را به کاراکتر یا عدد نسبت دهید.
* **تنها عملیات مجاز روی آن:** عملیات بیتی (Bitwise operations) مانند `&`, `|`, `^`, `~`, `<<`, `>>` است.

---

## ۲. نحوه تعریف و هدر (Header)

برای استفاده از `std::byte` باید هدر `<cstddef>` را اضافه کنید:

```cpp
#include <cstddef>
```

درون استاندارد، `std::byte` به شکل یک `enum class` پیاده‌سازی شده است:
```cpp
enum class byte : unsigned char {};
```

---

## ۳. مقداردهی و استفاده پایه

چون `std::byte` یک `enum class` بدون تبدیل ضمنی (Implicit conversion) است، مقداردهی اولیه به شیوه‌های خاصی انجام می‌شود:

```cpp
#include <iostream>
#include <cstddef>

int main() {
    // مقداردهی با مقدار مستقیم درون {}
    std::byte b1{0};
    std::byte b2{0xFF}; // ۲۵۵ در مبنای ۱۶
    std::byte b3{0b1010'0101}; // C++14 binary literal

    // خطا! تبدیل خودکار از int مجاز نیست:
    // std::byte b4 = 10; 

    // تبدیل صریح (Explicit Cast) از مقادیر عددی:
    int x = 42;
    std::byte b5 = static_cast<std::byte>(x);
}
```

## ۴. عملیات مجاز (Bitwise Operations)

روی `std::byte` می‌توانید تمام عملیات‌های منطقی بیتی را انجام دهید:

```cpp
#include <iostream>
#include <cstddef>

int main() {
    std::byte b1{0b0000'1111};
    std::byte b2{0b1111'0000};

    // عملیات OR بیتی
    std::byte b_or = b1 | b2; // 0b1111'1111

    // عملیات AND بیتی
    std::byte b_and = b1 & b2; // 0b0000'0000

    // عملیات XOR بیتی
    std::byte b_xor = b1 ^ b2;

    // شیفت بیتی (Shift)
    std::byte shifted = b1 << 2; // 0b0011'1100
}
```


## تبدیل به مقدار عددی با `std::to_integer`

اگر بخواهید مقدار یک `std::byte` را چاپ کنید یا مقدار عددی آن را بخوانید، باید از تابع کمکی `std::to_integer` استفاده کنید:

```cpp
#include <iostream>
#include <cstddef>

int main() {
    std::byte b{0x2A}; // عدد 42 در مبنای 16

    // تبدیل به int یا هر نوع عدد صحیح دیگر
    int val = std::to_integer<int>(b);

    std::cout << "Integer value: " << val << '\n'; // خروجی: 42
}
```

##  کاربرد عملی: ایجاد یک بافر حافظه خام (Raw Buffer)

یکی از رایج‌ترین سناریوها، خواندن داده از سوکت شبکه، فایل باینری یا دستکاری مستقیم حافظه است:

```cpp
#include <iostream>
#include <cstddef>
#include <vector>

void process_raw_data(const std::byte* data, std::size_t size) {
    for (std::size_t i = 0; i < size; ++i) {
        // خواندن و تبدیل بایت‌ها
        unsigned int byte_val = std::to_integer<unsigned int>(data[i]);
        std::cout << "Byte[" << i << "] = 0x" << std::hex << byte_val << '\n';
    }
}

int main() {
    // بافری از بایت‌های خام
    std::vector<std::byte> buffer = {
        std::byte{0x01},
        std::byte{0xFA},
        std::byte{0x10}
    };

    process_raw_data(buffer.data(), buffer.size());
}
```

## خلاصه‌ی مهم

| ویژگی | `char` / `unsigned char` | `std::byte` |
| :--- | :--- | :--- |
| **هدف اصلی** | کاراکتر متنی / عدد بدون علامت | بایت حافظه خام (Bit pattern) |
| **عملیات ریاضی (`+`, `-`)** | بله (مجاز) | **خیر** (خطای زمان کامپایل) |
| **عملیات بیتی (`&`, `|`, `<<`)** | بله | **بله** |
| **تبدیل خودکار از عدد** | بله | **خیر** (نیاز به Cast صریح) |
| **هدر استاندارد** | داخلی (Built-in) | `<cstddef>` |
---

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>
