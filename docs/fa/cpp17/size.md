<div align="center">

[🇺🇸 English](../../en/cpp17/size.md) | [🇮🇷 فارسی](./size.md)

</div>

# std::size


تابع **`std::size`** یکی دیگر از توابع کمکی بسیار کاربردی است که در **C++17** به هدر `<iterator>` اضافه شد. این تابع تعداد عناصر یک کانتینر یا یک آرایه خام را بازمی‌گرداند.


## مشکل قبلی و دلیل معرفی `std::size`

پیش از C++17 برای به دست آوردن تعداد عناصر:
* در کانتینرهای استاندارد (مانند `vector`, `string`, `map`): از متد `c.size()` استفاده می‌شد.
* در آرایه‌های خام به سبک C (مانند `int arr[10]`): از ترفند قدیمی C یعنی `sizeof(arr) / sizeof(arr[0])` استفاده می‌شد که خطاپذیر بود (مثلاً اگر آرایه به عنوان اشاره‌گر به تابعی پاس داده می‌شد، نتیجه نادرست می‌داد).

این دوگانگی باعث می‌شد در **توابع جنریک (Template)** نتوانید به سادگی سایز هر دو نوع را با یک ساختار مشترک به دست آورید. `std::size` این مشکل را حل کرد.


## تعریف و امضای تابع

برای استفاده باید هدر `<iterator>` را اضافه کنید:
```cpp
#include <iterator>
```

پیاده‌سازی آن دو نسخه اصلی دارد:

1. **برای کانتینرها:**
   ```cpp
   template <class C>
   constexpr auto size(const C& c) -> decltype(c.size());
   ```
   هر کانتینری که متد `.size()` داشته باشد را پشتیبانی می‌کند.

2. **برای آرایه‌های خام:**
   ```cpp
   template <class T, std::size_t N>
   constexpr std::size_t size(const T (&array)[N]) noexcept;
   ```
   به صورت مستقیم تعداد عناصر (`N`) را بدون نیاز به محاسبه دستی برمی‌گرداند.


##  مثال‌های کاربردی

### الف) استفاده پایه

```cpp
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    // ۱. آرایه خام C
    int raw_arr[] = {10, 20, 30, 40, 50};
    std::cout << "Raw array size: " << std::size(raw_arr) << "\n"; // خروجی: 5

    // ۲. وکتور استاندارد
    std::vector<std::string> names = {"Ali", "Reza", "Sara"};
    std::cout << "Vector size: " << std::size(names) << "\n";       // خروجی: 3

    return 0;
}
```


### کاربرد در کدهای جنریک (Templates)

```cpp
#include <iostream>
#include <vector>
#include <array>
#include <iterator>

template <typename T>
void printCount(const T& collection) {
    // برای هر نوع کالکشن یا آرایه‌ای کار می‌کند
    std::cout << "Element count: " << std::size(collection) << "\n";
}

int main() {
    int arr[] = {1, 2, 3};
    std::vector<int> vec = {10, 20, 30, 40};
    std::array<int, 2> std_arr = {100, 200};

    printCount(arr);      // 3
    printCount(vec);      // 4
    printCount(std_arr);  // 2

    return 0;
}
```


##  جلوگیری از خطای رایج زوال آرایه (Array Decay)

در C اگر یک آرایه را به تابعی بفرستید، آرایه به اشاره‌گر تبدیل می‌شود (`decay`) و فرمول `sizeof` اشتباه کار می‌کند. اما `std::size` **امن** است: اگر اشتباهاً یک اشاره‌گر خام به آن پاس بدهید، کد اصلاً **کامپایل نمی‌شود**:

```cpp
void badFunction(int* ptr) {
    // خطا در زمان کامپایل: pointer متد size ندارد و آرایه با سایز مشخص نیست
    // auto s = std::size(ptr); 
}
```


## کاربرد در زمان کامپایل (`constexpr`)

برای آرایه‌ها و `std::array`، تابع `std::size` به صورت `constexpr` اجرا می‌شود، بنابراین می‌توانید از آن برای تعریف آرایه‌های دیگر یا ثابت‌ها استفاده کنید:

```cpp
constexpr int arr1[] = {1, 2, 3, 4};
constexpr auto len = std::size(arr1); // محاسبه در زمان کامپایل

int arr2[len]; // کاملاً معتبر
```


## نکته جانبی: `std::ssize` در C++20
یکی از مشکلات `std::size` این است که خروجی آن عدد بدون علامت (`unsigned` مانند `size_t`) است که در مقایسه‌ها با `int`های علامت‌دار ممکن است هشدار (Warning) بدهد. به همین دلیل در **C++20** تابع کمکی دیگری به نام **`std::ssize`** اضافه شد که سایز را به صورت عدد علامت‌دار (`signed`) برمی‌گرداند.


### خلاصه
| ویژگی | توضیح |
| :--- | :--- |
| **استاندارد** | C++17 |
| **هدر** | `<iterator>` |
| **کاربرد** | دریافت تعداد عناصر کانتینرها و آرایه‌های سبک C |
| **امنیت** | جلوگیری از کامپایل در صورت ارسال Pointer به جای آرایه |
| **مزیت** | ایجاد رابط یکسان برای کدهای Template |

---

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>