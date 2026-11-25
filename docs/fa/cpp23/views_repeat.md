
<div align="center">

[🇺🇸 English](../../en/cpp23/views_repeat.md) | [🇮🇷 فارسی](./views_repeat.md)

</div>

# std::views::repeat

`std::views::repeat` یکی از قابلیت‌های کتابخانه Ranges در استاندارد **C++23** است که برای تولید یک دنباله از عناصر تکراری استفاده می‌شود.

به زبان ساده، یک مقدار به آن می‌دهیم و این View همان مقدار را به تعداد مشخص یا به‌صورت نامحدود تولید می‌کند.

##  سینتکس

```cpp
#include <ranges>

std::views::repeat(value);
std::views::repeat(value, count);
```

دو حالت دارد:

1. بدون `count`: یک Range نامحدود تولید می‌کند.
2. با `count`: یک Range محدود با تعداد مشخصی عنصر تولید می‌کند.

##  مثال ساده: تکرار محدود

```cpp
#include <iostream>
#include <ranges>

int main()
{
    for (int x : std::views::repeat(10, 5))
    {
        std::cout << x << ' ';
    }
}
```

**خروجی:**

```txt
10 10 10 10 10
```

در این مثال عدد `10` پنج بار تولید می‌شود.

##  تکرار نامحدود (Infinite Range)

اگر آرگومان دوم را مشخص نکنیم، یک Range نامحدود ایجاد می‌شود.

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto r = std::views::repeat(42);

    for (int x : r | std::views::take(5))
    {
        std::cout << x << ' ';
    }
}
```

خروجی:

```txt
42 42 42 42 42
```

**نکته مهم:** چون Range نامحدود است، برای پیمایش آن معمولاً باید از `take` یا یک محدودکننده دیگر استفاده کنیم.

##  ترکیب با سایر Viewها

یکی از مزیت‌های اصلی `views::repeat` قابلیت ترکیب آن با سایر Viewهای کتابخانه Ranges است.

مثلاً ترکیب با `transform`:

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto r =
        std::views::repeat(5, 4)
        | std::views::transform([](int x)
        {
            return x * 2;
        });

    for (int x : r)
        std::cout << x << ' ';
}
```

**خروجی:**

```
10 10 10 10
```

مثال دیگر، ترکیب با `filter`:

```cpp
auto r =
    std::views::repeat(7, 10)
    | std::views::filter([](int x)
    {
        return x % 2 == 0;
    });
```

در این مثال هیچ عنصری باقی نمی‌ماند، چون تمام عناصر برابر با ۷ هستند.

##  استفاده برای تولید رشته

این View فقط برای اعداد نیست و می‌توان از انواع دیگر نیز استفاده کرد.

```cpp
#include <iostream>
#include <ranges>
#include <string>

int main()
{
    for (const auto& s : std::views::repeat(std::string{"Hello"}, 3))
    {
        std::cout << s << '\n';
    }
}
```

**خروجی:**

```
Hello
Hello
Hello
```

##  ویژگی‌های فنی و کارایی

| ویژگی        | مقدار                 |
| ------------ | --------------------- |
| استاندارد    | C++23                 |
| هدر          | `<ranges>`            |
| نوع View     | `repeat_view`         |
| نوع Range    | `random_access_range` |
| حالت محدود   | `sized_range`         |
| حالت نامحدود | Infinite Range        |
| پیچیدگی ساخت | \\(O(1)\\)            |
| تولید عناصر  | Lazy                  |

**چند نکته مهم:**

- عناصر از قبل در حافظه ساخته نمی‌شوند.
- `repeat_view` مقدار موردنظر را نگهداری می‌کند.
- می‌توان با Iterator به عناصر دسترسی تصادفی داشت.
- در حالت محدود، `size()` تعداد عناصر را برمی‌گرداند.

##  مقایسه با روش‌های قدیمی‌تر

قبل از C++23 برای تولید عناصر تکراری معمولاً از `vector` یا حلقه استفاده می‌کردیم.

روش قدیمی:

```cpp
std::vector<int> v(5, 10);
```

روش C++23:

```cpp
auto v = std::views::repeat(10, 5);
```

تفاوت مهم این است که روش دوم یک View تنبل (Lazy View) ایجاد می‌کند و لزوماً حافظه‌ای برای پنج عنصر جداگانه اختصاص نمی‌دهد.

##  بررسی پشتیبانی کامپایلر

برای بررسی پشتیبانی از این قابلیت:

```cpp
#ifdef __cpp_lib_ranges_repeat

    // std::views::repeat is supported

#endif
```

مقدار Feature Test Macro:

```cpp
__cpp_lib_ranges_repeat == 202207L
```



**یک نکته جالب:** تفاوت اصلی `std::views::iota` و `std::views::repeat` این است:

```
std::views::iota(1, 6);   // 1, 2, 3, 4, 5

std::views::repeat(1, 5); // 1, 1, 1, 1, 1
```

اولی دنباله‌ای افزایشی تولید می‌کند، دومی یک مقدار ثابت را تکرار می‌کند.

--- 
## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>