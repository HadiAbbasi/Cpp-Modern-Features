<div align="center">

[🇺🇸 English](../../en/cpp23/flat_map.md) | [🇮🇷 فارسی](./flat_map.md)

</div>

# `std::flat_map`

`std::flat_map` یک ظرف انجمنی مرتب‌شده در C++23 است که جفت‌های کلید و مقدار را نگه می‌دارد. برخلاف `std::map` که معمولاً ساختار درختی دارد، `std::flat_map` داده‌ها را با دو دنبالهٔ زیرینِ جداگانه برای کلیدها و مقادیر ذخیره می‌کند. این چیدمان می‌تواند locality حافظه و پیمایش را بهبود دهد، اما درج یا حذف معمولاً مستلزم جابه‌جایی عناصر است.

کلیدها یکتا هستند و به‌صورت مرتب، بر اساس مقایسه‌گر ظرف نگه‌داری می‌شوند. `std::flat_map` در فضای نام `std` قرار دارد و به هدر `<flat_map>` نیاز دارد. پشتیبانی از کتابخانهٔ C++23 در کامپایلر و کتابخانهٔ استانداردِ مورد استفاده باید موجود باشد.

## نحوهٔ استفاده

```cpp
#include <flat_map>
#include <string>

int main() {
    std::flat_map<int, std::string> words;
    words.emplace(1, "one");
    words.emplace(2, "two");
}
```

## روش‌های مقداردهی اولیه

### فهرست مقداردهی

```cpp
#include <flat_map>
#include <string>

int main() {
    std::flat_map<int, std::string> words{
        {3, "three"}, {1, "one"}, {4, "four"}
    };
    // ترتیب پیمایش بر اساس کلید است: 1, 3, 4
}
```

### ساخت از بازه

می‌توان ظرف را از یک بازهٔ iterator ساخت. در نمونهٔ زیر داده‌ها ابتدا در یک بردار قرار دارند:

```cpp
#include <flat_map>
#include <string>
#include <utility>
#include <vector>

int main() {
    std::vector<std::pair<int, std::string>> entries{
        {2, "two"}, {1, "one"}
    };
    std::flat_map<int, std::string> words(entries.begin(), entries.end());
}
```

### کپی و انتساب

```cpp
#include <flat_map>
#include <string>

int main() {
    std::flat_map<int, std::string> original{{1, "one"}};
    std::flat_map<int, std::string> copied = original;
    std::flat_map<int, std::string> assigned;
    assigned = original;
}
```

### انتساب انتقالی (move assignment)

```cpp
#include <flat_map>
#include <string>
#include <utility>

int main() {
    std::flat_map<int, std::string> source{{1, "one"}};
    std::flat_map<int, std::string> destination;
    destination = std::move(source);
}
```

انتقال منابع می‌تواند از کپی‌کردن عناصر پرهیز کند، اما تضمین نمی‌کند که ظرف مقصد صرفاً به همان حافظه اشاره کند؛ جزئیات و میزان انتقال بدون کپی به پیاده‌سازی، تخصیص‌دهنده و نوع‌های زیرین وابسته است. پس از انتقال، `source` معتبر است، ولی وضعیت محتوای آن نباید بدون بررسی فرض شود.

## Iteratorها

`begin()` و `end()` به‌ترتیب iterator ابتدای ظرف و iterator یک‌گام‌پس‌ازآخر را می‌دهند. همچنین گونه‌های معکوس مانند `rbegin()` و `rend()` در دسترس‌اند. پیمایش، ترتیب مرتب‌شدهٔ کلیدها را دنبال می‌کند.

```cpp
#include <flat_map>
#include <iostream>
#include <string>

int main() {
    std::flat_map<int, std::string> words{
        {1, "one"}, {2, "two"}, {3, "three"}
    };

    for (auto it = words.begin(); it != words.end(); ++it) {
        std::cout << it->first << ": " << it->second << '\n';
    }

    for (const auto& [key, value] : words) {
        std::cout << key << " => " << value << '\n';
    }
}
```

## Modifierها

از عملیات رایج می‌توان به `insert`، `emplace`، `try_emplace`، `insert_or_assign`، `erase` و `clear` اشاره کرد. درج کلیدی که از قبل وجود دارد، کلید تکراری ایجاد نمی‌کند؛ رفتار دقیق هر تابع مطابق امضای همان تابع است. `erase` می‌تواند با iterator، بازه یا کلید فراخوانی شود.

```cpp
#include <flat_map>
#include <iostream>
#include <string>
#include <utility>

int main() {
    std::flat_map<int, std::string> words;
    words.insert(std::pair<int, std::string>{1, "one"});
    words.emplace(2, "two");
    words.insert_or_assign(2, "TWO");

    words.erase(1);
    for (const auto& [key, value] : words) {
        std::cout << key << ": " << value << '\n';
    }
    words.clear();
}
```

## اندازه و وضعیت

- `size()` تعداد عناصر موجود را برمی‌گرداند.
- `empty()` مشخص می‌کند ظرف خالی است یا نه.
- `max_size()` بیشینهٔ نظری تعداد عناصر قابل نگه‌داری را گزارش می‌کند؛ مقدار آن به پیاده‌سازی و تخصیص‌دهنده وابسته است و عدد ثابتی برای همهٔ سامانه‌ها نیست.

## عملیات جست‌وجو

`find(key)` iterator عنصر متناظر را برمی‌گرداند یا در صورت نبودن کلید `end()` را. `contains(key)` وجود کلید را به‌شکل بولی می‌سنجد و `count(key)` برای کلید یکتا صفر یا یک است. `lower_bound` و `upper_bound` نیز برای یافتن مرزهای بازهٔ مرتب‌شده کاربرد دارند.

```cpp
#include <flat_map>
#include <iostream>
#include <string>

int main() {
    std::flat_map<int, std::string> words{
        {1, "one"}, {3, "three"}, {5, "five"}
    };

    if (words.contains(3)) {
        auto it = words.find(3);
        std::cout << it->second << '\n';
    }
    std::cout << words.count(4) << '\n';
}
```

## پیچیدگی زمانی

برای عملیات زیر، پیچیدگی معمول چنین است؛ جزئیات استاندارد ممکن است برای overloadهای خاص تفاوت داشته باشد:

| عملیات | پیچیدگی معمول |
|---|---:|
| جست‌وجو مانند `find`، `contains`، `lower_bound` | `O(log n)` |
| پیمایش کل ظرف | `O(n)` |
| درج یا حذف یک عنصر | معمولاً `O(n)` به‌دلیل جابه‌جایی عناصر در دنباله‌های زیرین |
| `clear()` | `O(n)` در حالت کلی، برای آزادسازی/تخریب عناصر |

جست‌وجوی دودویی روی دادهٔ مرتب‌شده سریع است، اما درج و حذف را نباید به‌طور کلی `O(log n)` دانست: پیدا‌کردن جایگاه می‌تواند لگاریتمی باشد، ولی حفظ ترتیب در ذخیره‌سازی پیوسته معمولاً جابه‌جایی خطی می‌خواهد.

## مقایسه با ظرف‌های دیگر

| ویژگی | `std::flat_map` | `std::map` | `std::unordered_map` |
|---|---|---|---|
| ترتیب کلیدها | مرتب | مرتب | تضمین‌شده نیست |
| ساختار متداول | دو دنبالهٔ پیوسته | درخت متوازن | جدول درهم‌سازی |
| جست‌وجو | `O(log n)` | `O(log n)` | میانگین `O(1)`، بدترین `O(n)` |
| درج/حذف | معمولاً `O(n)` | `O(log n)` | میانگین `O(1)`، با امکان بازدرهم‌سازی |
| ویژگی شاخص | locality و پیمایش مناسب | درج/حذف مرتب با iteratorهای پایدارتر | جست‌وجوی سریعِ میانگین، بدون ترتیب |

پیچیدگی‌های جدول خلاصه‌اند؛ هزینهٔ مقایسه، تخصیص حافظه و اندازهٔ داده نیز بر کارایی واقعی اثر می‌گذارند.

## مزایا و محدودیت‌ها

مزیت اصلی `std::flat_map` ذخیره‌سازی پیوسته‌تر و locality بهتر نسبت به ساختارهای گره‌ای متداول است. این ویژگی می‌تواند پیمایش و جست‌وجو را در عمل کارآمد کند، به‌ویژه وقتی مجموعه پس از ساخت کمتر تغییر می‌کند. کلیدها مرتب‌اند و عملیات بازه‌ای مانند `lower_bound` مفیدند.

در مقابل، درج و حذف معمولاً عناصر را جابه‌جا می‌کنند و بنابراین برای تغییرات مکرر یا مجموعه‌های بزرگِ پُرتغییر ممکن است انتخاب مناسبی نباشد. همچنین نباید بدون توجه به قواعد iteratorها و اثر اصلاح ظرف، iteratorها یا ارجاع‌های قبلی را معتبر فرض کرد. نتیجهٔ کارایی به نوع داده، allocator و پیاده‌سازی وابسته است.

در مجموع، `std::flat_map` معمولاً برای الگوی «خواندن زیاد، تغییر کم» گزینهٔ مناسبی است؛ `std::map` برای درج/حذف مرتب و نیازهای مرتبط با iteratorها، و `std::unordered_map` برای جست‌وجوی کلیدیِ بدون نیاز به ترتیب می‌تواند مناسب‌تر باشد.


--- 
## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>