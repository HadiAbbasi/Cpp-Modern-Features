<div align="right">

[🇺🇸 English](../../en/cpp11/type_traits.md) | [🇮🇷 فارسی](./type_traits.md)

</div>
کتابخانه **`type_traits`** در **C++11** (که در هدر `<type_traits>` قرار دارد) یکی از قدرتمندترین ابزارها برای **برنامه‌نویسی متا (Template Metaprogramming)** است.

به زبان ساده، `type_traits` مجموعه‌ای از الگوها (Templates) است که به کامپایلر اجازه می‌دهد در **زمان کامپایل (Compile-time)** کارهای زیر را انجام دهد:

۱. بازرسی یا استعلام ویژگی‌های یک نوع داده (Type Inspection / Querying).
۲. تغییر یا تبدیل انواع داده (Type Transformation).
۳. تغییر رفتار کد یا اعمال شرایط خاص بر اساس ویژگی‌های نوع داده (مانند SFINAE و `static_assert`).

---

## دسته‌های اصلی Type Traits در C++11

الگوها و ساختارهای `type_traits` به‌طور کلی به سه دسته اصلی تقسیم می‌شوند:

### الف) پرس‌وجوها / بررسی‌های نوع (Type Queries / Checks)
این ساختارها حاوی یک فیلد استاتیک به نام `value` هستند که یک مقدار منطقی (`bool`) برمی‌گرداند.

* **بررسی ماهیت نوع داده:**
  * `std::is_integral<T>::value`: آیا این نوع، عددی صحیح (Integer) است؟ (`int`, `char`, `long` و غیره)
  * `std::is_floating_point<T>::value`: آیا این نوع، ممیز شناور (Floating-point) است؟ (`float`, `double`)
  * `std::is_pointer<T>::value`: آیا این نوع یک اشاره‌گر (Pointer) است؟ (`int*`)
  * `std::is_reference<T>::value`: آیا این نوع یک ارجاع (Reference) است؟
  * `std::is_array<T>::value`: آیا این یک آرایه خام سبک C است؟
  * `std::is_class<T>::value`: آیا این یک `class` یا `struct` است؟

* **بررسی ویژگی‌ها / صفات نوع (Type Traits/Properties):**
  * `std::is_const<T>::value`: آیا نوع داده `const` است؟
  * `std::is_signed<T>::value`: آیا علامت‌دار (Signed) است؟
  * `std::is_default_constructible<T>::value`: آیا سازنده پیش‌فرض (Default Constructor) دارد؟
  * `std::is_copy_constructible<T>::value`: آیا قابلیت ساخت کپی (Copy-constructible) دارد؟
  * `std::is_move_constructible<T>::value`: آیا قابلیت انتقال (Move-constructible) دارد؟

* **بررسی روابط بین دو نوع:**
  * `std::is_same<T, U>::value`: آیا دو نوع کاملاً یکسان هستند؟
  * `std::is_base_of<Base, Derived>::value`: آیا یکی از دیگری مشتق شده است؟
  * `std::is_convertible<From, To>::value`: آیا نوع اول می‌تواند به‌صورت ضمنی به نوع دوم تبدیل شود؟

---

### ب) تبدیل‌های نوع (Type Transformations)
این ساختارها یک نوع داده را پردازش کرده و نوع حاصل را در عضو متغیری به نام `type` ذخیره می‌کنند:

* `std::remove_const<T>::type`: کلمه کلیدی `const` را حذف می‌کند (مثلاً `const int` به `int` تبدیل می‌شود).
* `std::remove_reference<T>::type`: ارجاعات را حذف می‌کند (مثلاً `int&` یا `int&&` به `int` تبدیل می‌شود).
* `std::decay<T>::type`: تبدیل نوع هنگام ارسال آرگومان به تابع با مقدار (Pass by value) را شبیه‌سازی می‌کند (ارجاعات را حذف می‌کند، آرایه‌ها را به اشاره‌گر تبدیل می‌کند و غیره).
* `std::add_pointer<T>::type`: یک اشاره‌گر به نوع داده اضافه می‌کند.

---

## نمونه‌های کاربردی در C++11

### نمونه ۱: اعتبارسنجی در زمان کامپایل با `static_assert`
فرض کنید می‌خواهید تابعی بنویسید که فقط انواع صحیح مانند `int` و `long` را بپذیرد:

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
T add_integers_only(T a, T b) {
    // اگر T یک نوع صحیح نباشد، کامپایل با این پیام متوقف می‌شود.
    static_assert(std::is_integral<T>::value,
                  "T must be an integral type!");
    return a + b;
}

int main() {
    std::cout << add_integers_only(10, 20) << "\n"; // درست و بدون مشکل
}
```

### نمونه ۲: تبدیل نوع با `remove_reference`
گاهی اوقات لازم است نوعی را بدون نشانگرهای ارجاع دریافت کنید:

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void print_clean_type(T&& arg) {
    // نوع داده بدون ارجاع
    using CleanType = typename std::remove_reference<T>::type;

    // حالا CleanType همان نوع اولیه است که ارجاع از آن حذف شده است.
    std::cout << "Is pointer? "
              << std::is_pointer<CleanType>::value << "\n";
}
```

### نمونه ۳: انتخاب سربارگذاری (Overload) در زمان کامپایل با `std::enable_if` (مکانیسم SFINAE)
یکی از کاربردهای مهم `type_traits` در C++11 استفاده از SFINAE است که می‌تواند سربارگذاری‌های تابع را بسته به ویژگی‌های نوع داده فعال یا غیرفعال کند:

```cpp
#include <iostream>
#include <type_traits>

// سربارگذاری ۱: فقط برای انواع ممیز شناور (Floating-point) در دسترس است
template <typename T>
typename std::enable_if<
    std::is_floating_point<T>::value
>::type
printValue(T val) {
    std::cout << "Floating-point value: " << val << "\n";
}

// سربارگذاری ۲: فقط برای انواع صحیح (Integral) در دسترس است
template <typename T>
typename std::enable_if<
    std::is_integral<T>::value
>::type
printValue(T val) {
    std::cout << "Integer value: " << val << "\n";
}

int main() {
    printValue(42);      // سربارگذاری شماره ۲ را فراخوانی می‌کند
    printValue(3.1415);  // سربارگذاری شماره ۱ را فراخوانی می‌کند
}
```

### نمونه ۴: انتخاب شرطی نوع داده با `std::conditional`
قالب `std::conditional` شبیه عملگر شرطی سه‌تایی (`? :`) است، با این تفاوت که انتخاب نوع را در زمان کامپایل انجام می‌دهد:

```cpp
#include <iostream>
#include <type_traits>

// اگر اندازه T بیشتر از ۴ بایت باشد، double را انتخاب کن؛ در غیر این صورت float را انتخاب کن.
template <typename T>
struct SmartTypeSelector {
    using Type = typename std::conditional<
        (sizeof(T) > 4),
        double,
        float
    >::type;
};

int main() {
    SmartTypeSelector<char>::Type a = 1.5f;   // نوع داده از نوع float است
    SmartTypeSelector<double>::Type b = 2.5;  // معمولاً نوع داده از نوع double است
}
```

---

## یک نکته مهم: مقایسه C++11 با C++14 و C++17

در C++11، نحو (Syntax) کمی طولانی‌تر و وابسته‌تر به جزئیات است:
* برای دسترسی به مقدار یک ویژگی (Trait)، از `::value` استفاده می‌شود:
  ```cpp
  std::is_same<A, B>::value
  ```
* برای دسترسی به نوع تبدیل‌شده، از کلمات کلیدی `typename` و `::type` استفاده می‌شود:
  ```cpp
  typename std::decay<T>::type
  ```

در نسخه **C++14** نام مستعار کوتاه‌تری با پسوند `_t` اضافه شد، مانند `std::decay_t<T>`. همچنین نسخه **C++17** الگوهای متغیری با پسوند `_v` را معرفی کرد، مانند `std::is_same_v<A, B>`. این موارد شکل‌های مختصری برای راحتی کار هستند؛ مفاهیم اصلی `type_traits` از همان C++11 در دسترس بوده‌اند.

---

## خلاصه
کتابخانه `type_traits` در C++11 به شما اجازه می‌دهد که:

۱. **ویژگی‌های نوع داده را بازرسی کنید** (`is_pointer`, `is_integral`, `is_same` و غیره).
۲. **شرایط زمان کامپایل را اعمال کنید** (`static_assert`).
۳. **انواع داده را تبدیل کنید** (`remove_reference`, `decay` و غیره).
۴. **امضاهای تابع را به‌صورت شرطی فعال کنید** (`enable_if` برای مکانیزم SFINAE).

---

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | mb.roodsarabi76@gmail.com | | [mbr](https://t.me/ad1mi2n) |

</div>