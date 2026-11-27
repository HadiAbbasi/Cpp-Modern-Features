<div align="center">

[🇺🇸 English](../../en/cpp23/unreachable.md) | [🇮🇷 فارسی](./unreachable.md)

</div>

# std::range::find_last

تابع `std::ranges::find_last` در استاندارد **C++23** معرفی شده است و برای پیدا کردن **آخرین عنصر مطابق با یک مقدار مشخص در یک محدوده (Range)** استفاده می‌شود.

این تابع از ابتدای محدوده تا انتهای آن جست‌وجو می‌کند و آخرین عنصر منطبق را پیدا می‌کند.

##  سینتکس پایه

```cpp
#include <algorithm>
#include <vector>

auto result = std::ranges::find_last(range, value);
```

مقدار بازگشتی یک شیء از نوع `std::ranges::subrange` است که شامل دو iterator می‌شود:

- `begin()` : اشاره‌گر به آخرین عنصر پیدا شده.
- `end()` : اشاره‌گر به یک خانه بعد از عنصر پیدا شده.

اگر هیچ عنصر منطبقی پیدا نشود، هر دو iterator به انتهای محدوده اشاره می‌کنند.

##  مثال ساده

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v = {1, 2, 3, 2, 4, 2, 5};

    auto result = std::ranges::find_last(v, 2);

    if (!result.empty())
        std::cout << *result.begin(); // 2
}
```

در این مثال، آخرین عدد `2` در اندیس `5` قرار دارد.

**نکته:** تابع `find_last` یک iterator ساده برنمی‌گرداند، بلکه یک `subrange` برمی‌گرداند.

##  استفاده از find_last_if

اگر بخواهیم به‌جای مقایسه با یک مقدار مشخص، از یک شرط استفاده کنیم، می‌توانیم از `find_last_if` استفاده کنیم.

```cpp
std::vector<int> v = {1, 3, 5, 2, 7, 4, 9, 6};

auto result = std::ranges::find_last_if(
    v,
    [](int x) { return x % 2 == 0; }
);

if (!result.empty())
    std::cout << *result.begin(); // 6
```

در این مثال، آخرین عدد زوج پیدا می‌شود که برابر با `6` است.

##  استفاده از Projection

یکی از قابلیت‌های کاربردی الگوریتم‌های `ranges`، امکان استفاده از Projection است.

فرض کنید ساختار زیر را داریم:

```cpp
#include <algorithm>
#include <string>
#include <vector>

struct Person {
    std::string name;
    int age;
};

int main()
{
    std::vector<Person> people = {
        {"Alice", 25},
        {"Bob", 30},
        {"Charlie", 25},
        {"David", 40}
    };

    auto result = std::ranges::find_last(
        people,
        25,
        &Person::age
    );

    if (!result.empty())
        std::cout << result.begin()->name; // Charlie
}
```

در اینجا:

```
&Person::age
```

به الگوریتم می‌گوید به‌جای مقایسه کل شیء `Person`، فقط فیلد `age` را برای مقایسه در نظر بگیرد.

بنابراین آخرین شخصی که سن او ۲۵ است، یعنی `Charlie`، پیدا می‌شود.

##  تفاوت find_last با الگوریتم‌های مشابه

| الگوریتم               | کاربرد                                       |
| ---------------------- | -------------------------------------------- |
| `ranges::find`         | پیدا کردن اولین عنصر منطبق                   |
| `ranges::find_last`    | پیدا کردن آخرین عنصر منطبق                   |
| `ranges::find_if`      | پیدا کردن اولین عنصر مطابق با یک شرط         |
| `ranges::find_last_if` | پیدا کردن آخرین عنصر مطابق با یک شرط         |
| `ranges::find_end`     | پیدا کردن آخرین وقوع یک دنباله (Subsequence) |

##  پیچیدگی زمانی و کارایی

- حداقل به یک `forward_range` نیاز دارد.
- پیچیدگی زمانی: \\(O(N)\\)
- حداکثر N بار مقایسه انجام می‌دهد.
- پیاده‌سازی می‌تواند با توجه به نوع iterator بهینه‌سازی‌های بیشتری داشته باشد.

##  بررسی پشتیبانی کامپایلر

برای بررسی پشتیبانی کتابخانه استاندارد از این قابلیت می‌توان از Feature Test Macro استفاده کرد:

```cpp
#if defined(__cpp_lib_ranges_find_last) && \
    __cpp_lib_ranges_find_last >= 202207L

    // find_last is supported

#endif
```

## نکته مهم درباره مقدار بازگشتی

شاید مهم‌ترین نکته درباره `find_last` این باشد که مقدار بازگشتی آن یک `subrange` از آخرین عنصر پیدا شده تا انتهای محدوده است.

مثلاً اگر محدوده به شکل زیر باشد:

```cpp
std::vector<int> v = {1, 2, 3, 2, 4, 2, 5};
```

و آخرین `2` پیدا شود، `result` محدوده زیر را نشان می‌دهد:

```txt
2, 5
^  ^
|  |
begin
   end
```

یعنی علاوه بر دسترسی به آخرین عنصر پیدا شده، می‌توان عناصر بعد از آن را نیز پیمایش کرد.


--- 
## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>