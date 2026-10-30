<div align="center">

[🇺🇸 English](../../en/cpp17/data.md) | [🇮🇷 فارسی](./data.md)

</div>


# std::data
تابع **`std::data`** یکی از توابع کمکی کاربردی و ساده است که در **C++17** به کتابخانه استاندارد (هدر `<iterator>`) اضافه شد. این تابع یک اشاره‌گر (`pointer`) مستقیم به بلوک حافظه پیوسته (contiguous memory) مربوط به یک کانتینر یا آرایه برمی‌گرداند.


## هدف از معرفی `std::data` چیست؟

قبل از C++17 برای گرفتن اشاره‌گر به اولین خانه داده‌ها:
* برای کانتینرهایی مثل `std::vector` یا `std::string` باید می‌نوشتید: `v.data()`
* برای آرایه‌های خام به سبک C (مثل `int arr[5]`) باید از نام آرایه یا `&arr[0]` استفاده می‌کردید.

این تفاوت باعث می‌شد هنگام نوشتن **کدهای جنریک (Templates)** نتوانید به یک روش یکسان با هر دو کار کنید. `std::data` مانند `std::size`، `std::begin` و `std::end` یک رابط واحد (Uniform Interface) برای هر دوی این موارد فراهم می‌کند.

###  نحوه کار و تعریف

برای استفاده از آن نیاز به افزودن هدر زیر دارید:
```cpp
#include <iterator>
```

`std::data` به دو شکل تعریف شده است:

1. **برای کانتینرها (Containers):**
   ```cpp
   template <class C>
   constexpr auto data(C& c) -> decltype(c.data());
   ```
   کافیست کانتینر شما متد عضوی به نام `.data()` داشته باشد (مانند `std::vector`, `std::array`, `std::string`, `std::string_view`, `std::span`).

2. **برای آرایه‌های خام به سبک C (C-style Arrays):**
   ```cpp
   template <class T, std::size_t N>
   constexpr T* data(T (&array)[N]) noexcept;
   ```
   برای آرایه‌های خام، آدرس خانه اول (`array`) را بازمی‌گرداند.

---

## مثال‌های کاربردی

### الف) استفاده ساده با کانتینرها و آرایه‌ها

```cpp
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    // ۱. آرایه خام C
    int raw_arr[] = {10, 20, 30};
    int* ptr1 = std::data(raw_arr);
    std::cout << "Raw array first element: " << *ptr1 << "\n"; // خروجی: 10

    // ۲. وکتور
    std::vector<int> vec = {100, 200, 300};
    int* ptr2 = std::data(vec);
    std::cout << "Vector first element: " << *ptr2 << "\n";       // خروجی: 100

    // ۳. تغییر مقدار از طریق اشاره‌گر
    *ptr2 = 999;
    std::cout << "Modified vec[0]: " << vec[0] << "\n";          // خروجی: 999

    return 0;
}
```


### ب) کاربرد در برنامه‌نویسی جنریک (Generic / Template Code)

فرض کنید می‌خواهید تابعی بنویسید که داده‌ها را به یک API سطح پایین یا C (مانند توابع سیستم‌عامل، OpenGL یا کتابخانه‌های سوکت) ارسال کند:

```cpp
#include <iostream>
#include <vector>
#include <iterator>

// تابعی که داده‌ها را به صورت بافر C پردازش می‌کند
template <typename Container>
void sendToC_API(const Container& c) {
    // std::data و std::size هر دو در C++17 اضافه شدند
    const auto* buffer = std::data(c);
    auto length = std::size(c);

    std::cout << "Sending " << length << " items starting at address: " << buffer << "\n";
}

int main() {
    int c_arr[] = {1, 2, 3, 4};
    std::vector<double> v = {1.1, 2.2, 3.3};

    // این تابع جنریک هم با آرایه معمولی کار می‌کند و هم با وکتور
    sendToC_API(c_arr);
    sendToC_API(v);

    return 0;
}
```


## با چه کانتینرهایی کار می‌کند؟

`std::data` فقط با ساختارهایی کار می‌کند که داده‌های آن‌ها در حافظه **پیوسته (Contiguous)** قرار دارند و متد `.data()` دارند:
* ✅ کار می‌کند: `std::vector`, `std::array`, `std::string`, `std::string_view` و آرایه‌های سبک C (`T[]`).
* ❌ کار نمی‌کند: `std::list`, `std::deque`, `std::set`, `std::map` (زیرا این ساختارها داده‌های پیوسته ندارند و متد `data()` ندارند).


## نسخه `constexpr`

تابع `std::data` به‌صورت `constexpr` تعریف شده است، به این معنی که در زمان کامپایل (Compile-time) هم قابل استفاده است:

```cpp
constexpr int arr[] = {1, 2, 3};
constexpr const int* ptr = std::data(arr); // کاملاً در زمان کامپایل اجرا می‌شود
```

### خلاصه
* **کجا؟** هدر `<iterator>` در **C++17**.
* **چه کاری می‌کند؟** آدرس حافظه اولین عنصر (اشاره‌گر خام) را برمی‌گرداند.
* **مزیت اصلی:** یکسان‌سازی کد برای آرایه‌های خام (`T[]`) و کانتینرهای استاندارد (`std::vector`, `std::array` و ...)، به‌ویژه داخل توابع Template.

---

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>