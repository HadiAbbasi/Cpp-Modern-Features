<div align="center">

[🇺🇸 English](../../en/cpp17/empty.md) | [🇮🇷 فارسی](./empty.md)

</div>

# std::empty

تابع **`std::empty`** یکی دیگر از توابع کمکی بسیار کاربردی است که همراه با `std::size` و `std::data` در **C++17** به هدر `<iterator>` اضافه شد. این تابع بررسی می‌کند که آیا یک کانتینر، آرایه یا لیست مقداردهی اولیه (`initializer_list`) خالی است یا خیر و خروجی `bool` (`true` یا `false`) می‌دهد.


## هدف و فلسفه وجودی `std::empty`

قبل از C++17 برای بررسی خالی بودن:
* در کانتینرهای استاندارد (مانند `vector`, `string`, `list`) می‌نوشتیم: `c.empty()`
* در آرایه‌های سبک C (مانند `int arr[5]`) متدی به نام `empty()` وجود نداشت (و اصولاً یک آرایه معمولی با اندازه مشخص هرگز خالی نیست).
* در `std::initializer_list` متد `.empty()` وجود نداشت و باید از `il.size() == 0` استفاده می‌شد.

تابع `std::empty` یک **رابط مشترک (Uniform Interface)** برای همه این ساختارها ایجاد می‌کند تا در **توابع جنریک (Templates)** نیازی به کدهای شرطی مختلف نداشته باشید.



## تعریف و امضای تابع

برای استفاده باید هدر `<iterator>` را فراخوانی کنید:
```cpp
#include <iterator>
```

پیاده‌سازی آن دارای سه نسخه اصلی است:

1. **برای کانتینرها (Containers):**
   ```cpp
   template <class C>
   [[nodiscard]] constexpr auto empty(const C& c) -> decltype(c.empty());
   ```
   هر کانتینری که متد `.empty()` داشته باشد را صدا می‌زند.

2. **برای آرایه‌های خام به سبک C (C-style Arrays):**
   ```cpp
   template <class T, std::size_t N>
   [[nodiscard]] constexpr bool empty(const T (&array)[N]) noexcept {
       return false;
   }
   ```
   *نکته جالب:* چون آرایه‌های خام به سبک C اندازه ثابت و غیرصفر دارند، این تابع همیشه مقدار **`false`** برمی‌گرداند.

3. **برای لیست‌های مقداردهی (`std::initializer_list`):**
   ```cpp
   template <class E>
   [[nodiscard]] constexpr bool empty(std::initializer_list<E> il) noexcept {
       return il.size() == 0;
   }
   ```

*(ویژگی `[[nodiscard]]` در C++20 اضافه شد تا اگر خروجی بررسی خالی بودن را نادیده بگیرید، کامپایلر اخطار دهد).*


## مثال‌های کاربردی

### الف) استفاده پایه

```cpp
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    std::vector<int> numbers;

    // بررسی وکتور خالی
    if (std::empty(numbers)) {
        std::cout << "Vector is empty!\n"; // این چاپ می‌شود
    }

    numbers.push_back(42);

    if (!std::empty(numbers)) {
        std::cout << "Vector is NOT empty!\n";
    }

    // آرایه معمولی C
    int arr[] = {1, 2, 3};
    std::cout << std::boolalpha << std::empty(arr) << "\n"; // همیشه false است

    return 0;
}
```


### ب) استفاده در کدهای جنریک (Templates)

قدرت اصلی `std::empty` زمانی مشخص می‌شود که تابعی می‌نویسید که ورودی آن می‌تواند هر نوع کانتینری باشد:

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <iterator>

template <typename Container>
void printFirstElement(const Container& c) {
    // به جای c.empty() از std::empty(c) استفاده می‌کنیم
    if (std::empty(c)) {
        std::cout << "Container is empty, cannot print first element.\n";
        return;
    }

    std::cout << "First element: " << *std::begin(c) << "\n";
}

int main() {
    std::vector<int> empty_vec;
    std::string text = "Hello C++17";
    int raw_arr[] = {100, 200, 300};

    printFirstElement(empty_vec); // خروجی: Container is empty...
    printFirstElement(text);      // خروجی: First element: H
    printFirstElement(raw_arr);   // خروجی: First element: 100

    return 0;
}
```


## چرا به جای `c.size() == 0` باید از `std::empty(c)` استفاده کرد؟

این یک اصل مهم در مدرن C++ است:
1. **پیچیدگی زمانی (Performance):** برای برخی کانتینرها (مثل `std::list` در C++ قدیمی یا کانتینرهای پیاده‌سازی شده سفارشی)، محاسبه `size()` ممکن است از مرتبه $O(N)$ زمان ببرد، در حالی که `empty()` همیشه در زمان ثابت $O(1)$ اجرا می‌شود.
2. **خوانایی کد:** بیان صریح هدف (Intent)؛ خواندن `std::empty(c)` خواناتر و گویاتر از مقایسه با صفر است.


## خلاصه
* **استاندارد:** C++17
* **هدر:** `<iterator>`
* **وظیفه:** بررسی خالی بودن ساختار داده با خروجی بولی (`bool`).
* **مزیت:** هماهنگی بین کانتینرهای STL، آرایه‌های خام C و `initializer_list` در برنامه‌نویسی Generic.
---

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>