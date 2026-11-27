<div align="center">

[🇺🇸 English](../../en/cpp23/ranges_to.md) | [🇮🇷 فارسی](./ranges_to.md)

</div>


# std::ranges::to

تابع `std::ranges::to` یکی از قابلیت‌های استاندارد C++23 است که برای **تبدیل یک Range به یک Container** استفاده می‌شود.

به زبان ساده، این تابع یک Range مانند `std::views::iota` یا ترکیبی از چند View را دریافت می‌کند و عناصر آن را به یک Container مانند `std::vector`، `std::list` یا `std::set` تبدیل می‌کند.

یکی از مهم‌ترین مزایای آن، ساده‌تر و خواناتر کردن تبدیل Viewها به Containerها است.

##  سینتکس پایه

```cpp
#include <ranges>

auto container = std::ranges::to<Container>(range);
```

برای مثال:

```cpp
std::vector<int> v =
    std::ranges::to<std::vector<int>>(some_range);
```

همچنین می‌توان از سینتکس Pipe استفاده کرد:

```cpp
auto v = some_range | std::ranges::to<std::vector>();
```

در روش دوم، نیازی نیست تمام آرگومان‌های قالب (Template Arguments) را مشخص کنیم.

##  مثال ساده: تبدیل یک View به Vector

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    auto r = std::views::iota(1, 6);

    auto v = std::ranges::to<std::vector<int>>(r);

    for (int x : v)
        std::cout << x << ' ';
}
```

**خروجی:**

```txt
1 2 3 4 5
```

در این مثال:

- `std::views::iota(1, 6)` یک Range شامل اعداد ۱ تا ۵ تولید می‌کند.
- `std::ranges::to<std::vector<int>>` این Range را به یک `std::vector<int>` تبدیل می‌کند.

برخلاف View، بردار (Vector) حاصل مالک عناصر خودش است.

##  استفاده از سینتکس Pipe

یکی از قابلیت‌های بسیار کاربردی `std::ranges::to`، امکان استفاده از آن در زنجیره پردازش Ranges است.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    auto v = std::views::iota(1, 11)
           | std::views::filter([](int x)
             {
                 return x % 2 == 0;
             })
           | std::ranges::to<std::vector>();

    for (int x : v)
        std::cout << x << ' ';
}
```

**خروجی:**

```txt
2 4 6 8 10
```

این مثال سه مرحله دارد:

1. اعداد ۱ تا ۱۰ تولید می‌شوند.
2. اعداد فرد حذف می‌شوند.
3. View حاصل به یک Vector تبدیل می‌شود.

نکته جالب این است که نیازی به پیمایش دستی View و اضافه کردن عناصر با `push_back` نداریم.

##  تبدیل به Containerهای مختلف

تابع `std::ranges::to` محدود به `std::vector` نیست.

برای مثال، تبدیل به `std::list`:

```cpp
#include <list>
#include <ranges>

int main()
{
    auto r = std::views::iota(1, 6);

    auto values = std::ranges::to<std::list<int>>(r);
}
```

تبدیل به `std::set`:

```cpp 
#include <ranges>
#include <set>

int main()
{
    auto r = std::views::iota(1, 6);

    auto values = std::ranges::to<std::set<int>>(r);
}
```

Containerهای حاصل:

```txt
list: 1, 2, 3, 4, 5
set:  1, 2, 3, 4, 5
```

توجه داشته باشید که `std::set` به‌صورت خودکار عناصر را مرتب می‌کند و از ذخیره عناصر تکراری جلوگیری می‌کند.

##  ترکیب با transform**

می‌توان `to` را با `transform` ترکیب کرد تا عناصر تغییر داده شوند و سپس در یک Container ذخیره شوند.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    auto v = std::views::iota(1, 6)
           | std::views::transform([](int x)
             {
                 return x * x;
             })
           | std::ranges::to<std::vector>();

    for (int x : v)
        std::cout << x << ' ';
}
```

**خروجی:**

```txt
1 4 9 16 25
```

در این مثال، عملیات تبدیل به‌صورت Lazy انجام می‌شود و `std::ranges::to` عناصر حاصل را در یک Vector ذخیره می‌کند.

##  استفاده برای تبدیل به String

می‌توان از این تابع برای تبدیل یک Range از کاراکترها به `std::string` نیز استفاده کرد.

```cpp
#include <iostream>
#include <ranges>
#include <string>

int main()
{
    auto r = std::views::iota('A', 'F');

    auto s = std::ranges::to<std::string>(r);

    std::cout << s;
}
```

**خروجی:**

```txt
ABCDE
```

این قابلیت هنگام کار با Viewهای کاراکتری بسیار کاربردی است.

##  تبدیل Range به Container با نوع عناصر متفاوت

`std::ranges::to` می‌تواند در صورت پشتیبانی Container مقصد از ساخت یا درج عناصر، تبدیل نوع عناصر را نیز انجام دهد.

برای مثال:

```cpp
#include <ranges>
#include <vector>

int main()
{
    auto r = std::views::iota(1, 6);

    auto v = std::ranges::to<std::vector<double>>(r);
}
```

Vector حاصل شامل مقادیر زیر خواهد بود:

```txt
1.0, 2.0, 3.0, 4.0, 5.0
```

##  ویژگی‌های فنی

| ویژگی              | توضیحات                       |
| ------------------ | ----------------------------- |
| استاندارد          | C++23                         |
| هدر                | `<ranges>`                    |
| نام تابع           | `std::ranges::to`             |
| کاربرد             | تبدیل Range به Container      |
| پشتیبانی از Viewها | بله                           |
| پشتیبانی از Pipe   | بله                           |
| تبدیل نوع عناصر    | در صورت امکان ساخت عناصر مقصد |
| خروجی              | یک Container مالک عناصر       |

چند نکته مهم:

- `std::ranges::to` یک ابزار تبدیل است، نه یک View.
- Container حاصل مالک عناصر خودش است.
- می‌تواند با Viewهای Lazy کار کند.
- قابلیت ترکیب با `filter`، `transform`، `take` و سایر Range Adaptorها را دارد.
- Container مقصد باید شرایط لازم برای ساخت و درج عناصر را داشته باشد.

##  مقایسه با روش‌های قدیمی‌تر

قبل از C++23، برای تبدیل یک Range به Container معمولاً باید عناصر را به‌صورت دستی درج می‌کردیم.

روش قدیمی:

```cpp
std::vector<int> v;

for (int x : std::views::iota(1, 6))
{
    v.push_back(x);
}
```

روش C++23:

```cpp
auto v = std::views::iota(1, 6)
       | std::ranges::to<std::vector>();
```

روش جدید کوتاه‌تر، خواناتر و سازگارتر با زنجیره پردازش Ranges است.

## تفاوت مهم بین std::views و std::ranges::to

به مثال زیر توجه کنید:

```cpp
auto r = std::views::iota(1, 6);

auto v = r | std::ranges::to<std::vector>();
```

تفاوت این دو مفهوم:

| `std::views`               | `std::ranges::to`                  |
| -------------------------- | ---------------------------------- |
| یک View ایجاد می‌کند       | یک Container ایجاد می‌کند          |
| معمولاً Lazy است           | عناصر را در Container ذخیره می‌کند |
| لزوماً مالک عناصر نیست     | مالک عناصر خودش است                |
| مناسب ترکیب عملیات پردازشی | مناسب ذخیره نتایج                  |

**نکته کلیدی:**

از `std::views` برای تعریف چگونگی پردازش داده‌ها استفاده می‌کنیم و از `std::ranges::to` زمانی استفاده می‌کنیم که بخواهیم نتیجه پردازش را در یک Container واقعی ذخیره کنیم.

--- 
## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>