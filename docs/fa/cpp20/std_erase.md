<div align="center">

[🇺🇸 English](../../en/cpp20/std_erase.md) | [🇮🇷 فارسی](./std_erase.md)

</div>

---

# معرفی std::erase در ++C20

## مقدمه

مدیریت عناصر در Containerها یکی از عملیات‌های رایج در برنامه‌نویسی ++C است. در بسیاری از مواقع لازم است یک یا چند عنصر را بر اساس مقدار مشخص یا یک شرط خاص از یک Container حذف کنیم.

پیش از استاندارد ++C20، حذف عناصر از Containerهایی مانند `std::vector` معمولاً به ترکیب الگوریتم `std::remove` و تابع عضو `erase` نیاز داشت. این روش که با نام **Erase-Remove Idiom** شناخته می‌شود، اگرچه کاملاً استاندارد و کاربردی بود، اما برای توسعه‌دهندگان تازه‌کار چندان شهودی نبود و گاهی باعث اشتباه در استفاده از API می‌شد.

استاندارد ++C20 با معرفی توابع غیرعضوی `std::erase` و `std::erase_if` این فرایند را ساده‌تر کرد.

این توابع امکان حذف مستقیم عناصر بر اساس مقدار یا Predicate را فراهم می‌کنند و جزئیات مربوط به الگوریتم حذف و فراخوانی تابع عضو `erase` را از دید کاربر پنهان می‌کنند.

برای مثال، در ++C17 حذف تمام عناصر برابر با مقدار مشخص از یک `std::vector` به شکل زیر انجام می‌شد:

```
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 2, 4, 2, 5};

    values.erase(
        std::remove(values.begin(), values.end(), 2),
        values.end()
    );
}
```

اما از ++C20 می‌توان همین عملیات را با یک فراخوانی انجام داد:

```
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 2, 4, 2, 5};

    std::erase(values, 2);
}
```

این تغییر علاوه بر کاهش حجم کد، خوانایی و قابلیت نگهداری آن را نیز افزایش می‌دهد.

## معرفی std::erase

تابع `std::erase` یک Non-member Function در Standard Library است که از استاندارد ++C20 به بعد در دسترس قرار دارد.

هدف اصلی این تابع حذف تمام عناصر یک Container است که با یک مقدار مشخص برابر هستند.

### ساختار کلی

ساختار مفهومی استفاده از این تابع به شکل زیر است:

```
std::erase(container, value);
```

در این ساختار:

- `container` به Container موردنظر اشاره می‌کند.
- `value` مقداری است که باید تمام عناصر برابر با آن حذف شوند.
- مقدار بازگشتی تعداد عناصر حذف‌شده است.

### امضای تابع

برای Containerهایی مانند `std::vector`، ساختار کلی Overload تابع به شکل زیر است:

```
template<class T, class Allocator, class U = T>
constexpr typename std::vector<T, Allocator>::size_type
erase(std::vector<T, Allocator>& container, const U& value);
```

این امضا نشان می‌دهد که:

- Container باید به‌صورت Non-const Reference دریافت شود؛ زیرا محتوای آن تغییر می‌کند.
- مقدار موردنظر از طریق `const U&` دریافت می‌شود.
- نوع بازگشتی `size_type` است.
- پارامتر `U` می‌تواند با نوع عناصر Container متفاوت باشد، مشروط بر اینکه مقایسه موردنیاز معتبر باشد.

امضای دقیق Overloadها برای انواع مختلف Containerها ممکن است متفاوت باشد، اما رفتار عمومی تابع یکسان است.

### مقدار بازگشتی

تابع `std::erase` تعداد عناصری را که واقعاً حذف شده‌اند برمی‌گرداند.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 2, 4, 2};

    const auto removedCount = std::erase(values, 2);

    std::cout << "Removed: " << removedCount << '\n';
    std::cout << "Remaining: " << values.size() << '\n';
}
```

خروجی:

```
Removed: 3
Remaining: 3
```

اگر مقدار موردنظر در Container وجود نداشته باشد، مقدار بازگشتی صفر خواهد بود و Container بدون تغییر باقی می‌ماند.

## Containerهای پشتیبانی‌شده توسط std::erase

تابع `std::erase` برای مجموعه مشخصی از Containerهای Standard Library ارائه شده است.

این تابع را نباید با یک API عمومی برای تمام انواع Containerها اشتباه گرفت.

### پشتیبانی از Sequence Containerها

در استاندارد ++C20، تابع `std::erase` برای Containerهای زیر ارائه شده است:

| Container               | پشتیبانی از std::erase |
| ----------------------- | ---------------------- |
| `std::vector`           | بله                    |
| `std::deque`            | بله                    |
| `std::list`             | بله                    |
| `std::forward_list`     | بله                    |
| `std::basic_string`     | بله                    |
| `std::array`            | خیر                    |
| `std::span`             | خیر                    |
| `std::initializer_list` | خیر                    |

برای استفاده از این تابع باید Header مربوط به Container موردنظر را Include کنید.

برای مثال:

```
#include <vector>
#include <deque>
#include <list>
#include <forward_list>
#include <string>
```

### چرا std::array پشتیبانی نمی‌شود؟

نوع `std::array` دارای اندازه ثابت است و عملیات حذف عنصر که باعث کاهش تعداد عناصر آن شود، با طراحی این Container سازگار نیست.

به همین دلیل تابع `std::erase` برای `std::array` ارائه نشده است.

اگر هدف حذف منطقی عناصر از یک `std::array` باشد، می‌توان از روش‌هایی مانند انتقال عناصر معتبر به بخش ابتدایی آرایه و نگهداری تعداد عناصر فعال استفاده کرد.

### چرا std::map و std::set مستقیماً پشتیبانی نمی‌شوند؟

برای Containerهای Associative و Unordered Associative، مانند موارد زیر:

- `std::set`
- `std::multiset`
- `std::map`
- `std::multimap`
- `std::unordered_set`
- `std::unordered_multiset`
- `std::unordered_map`
- `std::unordered_multimap`

تابع `std::erase` ارائه نشده است.

در عوض، استاندارد ++C20 تابع `std::erase_if` را برای حذف عناصر بر اساس Predicate در این Containerها معرفی کرده است.

در ادامه تفاوت این دو تابع بررسی خواهد شد.

## نحوه عملکرد std::erase

رفتار داخلی `std::erase` برای همه Containerها الزاماً یکسان پیاده‌سازی نمی‌شود.

این موضوع به ساختار داده و API مربوط به هر Container بستگی دارد.

### عملکرد در std::vector و std::deque

در Containerهایی مانند `std::vector` و `std::deque`، حذف عناصر با استفاده از الگوریتمی مشابه `std::remove` و سپس حذف بخش انتهایی Container انجام می‌شود.

الگوریتم `std::remove` عناصر موردنظر را از نظر منطقی حذف می‌کند، اما اندازه واقعی Container را کاهش نمی‌دهد.

برای درک بهتر، فرض کنید Vector زیر را داریم:

```
std::vector<int> values{10, 20, 30, 20, 40, 20, 50};
```

با اجرای الگوریتم زیر:

```
auto newEnd = std::remove(
    values.begin(),
    values.end(),
    20
);
```

عناصر برابر با `20` از محدوده منطقی ابتدایی حذف می‌شوند و عناصر باقی‌مانده در ابتدای محدوده قرار می‌گیرند.

در این مرحله، اندازه واقعی Vector هنوز هفت است.

```
values.erase(newEnd, values.end());
```

فراخوانی بالا عناصر اضافی انتهای Vector را حذف می‌کند و اندازه Container را به چهار کاهش می‌دهد.

تابع `std::erase` در ++C20 همین عملیات را در یک API واحد ارائه می‌کند.

### عملکرد در std::list و std::forward_list

این دو Container دارای Member Function اختصاصی برای حذف عناصر بر اساس مقدار هستند.

در نتیجه، پیاده‌سازی `std::erase` برای آن‌ها می‌تواند مستقیماً از قابلیت حذف خود Container استفاده کند.

برای مثال:

```
#include <iostream>
#include <list>

int main() {
    std::list<int> values{1, 2, 3, 2, 4, 2};

    const auto removedCount = std::erase(values, 2);

    std::cout << removedCount << '\n';
}
```

در این حالت، عناصر برابر با مقدار `2` از ساختار Linked List حذف می‌شوند.

نکته مهم این است که حذف یک Node در `std::list` باعث جابه‌جایی عناصر دیگر نمی‌شود و Iteratorها و Referenceهای مربوط به عناصر حذف‌نشده معتبر باقی می‌مانند.

### عملکرد در std::basic_string

تابع `std::erase` برای `std::basic_string` نیز ارائه شده است.

بنابراین می‌توان تمام کاراکترهای برابر با یک مقدار مشخص را حذف کرد.

```
#include <iostream>
#include <string>

int main() {
    std::string text = "programming";

    const auto removedCount = std::erase(text, 'm');

    std::cout << text << '\n';
    std::cout << removedCount << '\n';
}
```

خروجی:

```
prograing
2
```

در این مثال، تمام کاراکترهای `m` حذف شده‌اند.

توجه کنید که این عملیات بر اساس مقدار Character انجام می‌شود، نه بر اساس Unicode Character یا Grapheme Cluster.

## معرفی std::erase_if

تابع `std::erase_if` یکی دیگر از قابلیت‌های مهم ++C20 است که در کنار `std::erase` معرفی شد.

تفاوت اصلی این دو تابع در نحوه تعیین عناصر قابل حذف است:

- تابع `std::erase` عناصر برابر با یک مقدار مشخص را حذف می‌کند.
- تابع `std::erase_if` عناصری را حذف می‌کند که یک Predicate برای آن‌ها مقدار `true` برگرداند.

### ساختار کلی

```
std::erase_if(container, predicate);
```

در این ساختار:

- `container` همان Container موردنظر است.
- `predicate` یک Callable Object است که برای بررسی عناصر استفاده می‌شود.

### مثال ساده

فرض کنید می‌خواهیم تمام اعداد زوج یک Vector را حذف کنیم.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6, 7, 8};

    const auto removedCount = std::erase_if(
        values,
        [](int value) {
            return value % 2 == 0;
        }
    );

    for (const int value : values) {
        std::cout << value << ' ';
    }

    std::cout << "\nRemoved: " << removedCount << '\n';
}
```

خروجی:

```
1 3 5 7
Removed: 4
```

در این مثال، Lambda Expression برای بررسی زوج بودن عناصر استفاده شده است.

هر عنصری که Predicate برای آن `true` برگرداند، حذف خواهد شد.

### شرایط Predicate

تابع Predicate باید برای نوع عنصر Container قابل فراخوانی باشد و نتیجه‌ای قابل تبدیل به `bool` تولید کند.

همچنین Predicate باید از نظر منطقی یک شرط معتبر و پایدار برای عملیات حذف باشد.

بهتر است Predicate در طول اجرای الگوریتم:

- به وضعیت خارجی متغیر و غیرقابل پیش‌بینی وابسته نباشد.
- عناصر Container را تغییر ندهد.
- باعث ایجاد Side Effectهای غیرضروری نشود.

این نکات به‌خصوص هنگام استفاده از Lambdaهایی که متغیرهای خارجی را Capture می‌کنند اهمیت دارند.

## تفاوت std::erase و std::erase_if

این دو تابع از نظر هدف و نحوه استفاده متفاوت هستند.

| ویژگی              | std::erase          | std::erase_if       |
| ------------------ | ------------------- | ------------------- |
| مبنای حذف          | مقدار مشخص          | Predicate           |
| حذف چند عنصر       | بله                 | بله                 |
| دریافت Lambda      | خیر                 | بله                 |
| پشتیبانی از vector | بله                 | بله                 |
| پشتیبانی از list   | بله                 | بله                 |
| پشتیبانی از map    | خیر                 | بله                 |
| مقدار بازگشتی      | تعداد عناصر حذف‌شده | تعداد عناصر حذف‌شده |
| معرفی در استاندارد | ++C20               | ++C20               |

### چه زمانی از std::erase استفاده کنیم؟

زمانی که هدف حذف تمام عناصر برابر با یک مقدار مشخص است، استفاده از `std::erase` انتخاب ساده‌تر و خواناتری خواهد بود.

```
std::vector<int> values{1, 2, 3, 2, 4};

std::erase(values, 2);
```

### چه زمانی از std::erase_if استفاده کنیم؟

زمانی که حذف عناصر به یک شرط، مقایسه پیچیده یا منطق خاص وابسته است، `std::erase_if` گزینه مناسب‌تری خواهد بود.

```
std::vector<int> values{10, 15, 20, 25, 30};

std::erase_if(values, [](int value) {
    return value > 20;
});
```

همچنین برای حذف عناصر از Associative Containerها باید از `std::erase_if` استفاده کرد.

## استفاده از std::erase در Containerهای مختلف

### حذف عناصر از std::vector

نوع `std::vector` یکی از رایج‌ترین موارد استفاده از `std::erase` است.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers{
        10, 20, 30, 20, 40, 20, 50
    };

    std::erase(numbers, 20);

    for (const auto number : numbers) {
        std::cout << number << ' ';
    }
}
```

خروجی:

```
10 30 40 50
```

در این مثال، تمام عناصر برابر با `20` حذف شده‌اند.

### حذف عناصر از std::deque

تابع `std::erase` برای `std::deque` نیز قابل استفاده است.

```
#include <deque>
#include <iostream>

int main() {
    std::deque<int> numbers{
        1, 2, 3, 2, 4, 2, 5
    };

    const auto removedCount = std::erase(numbers, 2);

    std::cout << "Removed: " << removedCount << '\n';

    for (const auto number : numbers) {
        std::cout << number << ' ';
    }
}
```

رفتار کلی حذف مشابه `std::vector` است، اما قواعد Invalidation مربوط به Iteratorها و Referenceها در `std::deque` متفاوت است.

### حذف عناصر از std::list

در `std::list`، حذف عناصر از طریق عملیات اختصاصی Linked List انجام می‌شود.

```
#include <iostream>
#include <list>

int main() {
    std::list<int> numbers{
        10, 20, 30, 20, 40, 20
    };

    const auto removedCount = std::erase(numbers, 20);

    std::cout << "Removed: " << removedCount << '\n';

    for (const auto number : numbers) {
        std::cout << number << ' ';
    }
}
```

خروجی:

```
Removed: 3
10 30 40
```

### حذف عناصر از std::forward_list

برای `std::forward_list` نیز می‌توان از همین API استفاده کرد.

```
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> numbers{
        1, 2, 3, 2, 4, 2
    };

    const auto removedCount = std::erase(numbers, 2);

    std::cout << "Removed: " << removedCount << '\n';

    for (const auto number : numbers) {
        std::cout << number << ' ';
    }
}
```

این قابلیت استفاده از یک API مشترک را برای دو نوع Linked List استاندارد فراهم می‌کند.

### حذف کاراکترها از std::string

یکی از کاربردهای عملی `std::erase` حذف کاراکترهای ناخواسته از String است.

```
#include <iostream>
#include <string>

int main() {
    std::string username = "user_name_123";

    std::erase(username, '_');

    std::cout << username << '\n';
}
```

خروجی:

```
username123
```

برای حذف چند نوع کاراکتر می‌توان از `std::erase_if` استفاده کرد:

```
#include <iostream>
#include <string>
#include <cctype>

int main() {
    std::string text = "C++20 Programming 123!";

    std::erase_if(text, [](unsigned char ch) {
        return std::isdigit(ch) || std::isspace(ch);
    });

    std::cout << text << '\n';
}
```

خروجی:

```
C++Programming!
```

در این مثال، اعداد و فاصله‌ها حذف شده‌اند.

نکته مهم این است که توابعی مانند `std::isdigit` و `std::isspace` در `<cctype>` برای مقادیری معتبر هستند که برابر با `EOF` باشند یا در محدوده قابل نمایش توسط `unsigned char` قرار بگیرند. به همین دلیل در مثال بالا پارامتر Lambda از نوع `unsigned char` است.

این روش برای پردازش Byteهای یک String مناسب است، اما برای تحلیل کامل متن Unicode کافی نیست.

## استفاده از std::erase_if در Associative Containerها

یکی از مهم‌ترین مزیت‌های `std::erase_if` نسبت به `std::erase`، پشتیبانی آن از Associative Containerها است.

### حذف عناصر از std::set

فرض کنید یک Set از اعداد داریم و می‌خواهیم تمام اعداد کوچک‌تر از یک مقدار مشخص را حذف کنیم.

```
#include <iostream>
#include <set>

int main() {
    std::set<int> numbers{
        1, 2, 3, 4, 5, 6, 7
    };

    const auto removedCount = std::erase_if(
        numbers,
        [](int value) {
            return value < 4;
        }
    );

    std::cout << "Removed: " << removedCount << '\n';

    for (const auto value : numbers) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
Removed: 3
4 5 6 7
```

### حذف عناصر از std::map

در `std::map`، هر عنصر یک `std::pair<const Key, T>` است.

بنابراین Predicate به یک عنصر از نوع Pair دسترسی دارد و می‌تواند هم Key و هم Value را بررسی کند.

```
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> scores{
        {"Alice", 90},
        {"Bob", 45},
        {"Charlie", 70},
        {"David", 30}
    };

    const auto removedCount = std::erase_if(
        scores,
        [](const auto& entry) {
            return entry.second < 50;
        }
    );

    std::cout << "Removed: " << removedCount << '\n';

    for (const auto& [name, score] : scores) {
        std::cout << name << ": " << score << '\n';
    }
}
```

خروجی:

```
Removed: 2
Alice: 90
Charlie: 70
```

در این مثال، دانش‌آموزانی که امتیاز آن‌ها کمتر از `50` است حذف می‌شوند.

استفاده از `const auto&` در Lambda باعث جلوگیری از کپی غیرضروری عناصر می‌شود.

### حذف عناصر از std::unordered_map

این تابع برای Unordered Associative Containerها نیز قابل استفاده است.

```
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> inventory{
        {"Laptop", 10},
        {"Mouse", 0},
        {"Keyboard", 5},
        {"Monitor", 0}
    };

    const auto removedCount = std::erase_if(
        inventory,
        [](const auto& item) {
            return item.second == 0;
        }
    );

    std::cout << "Removed: " << removedCount << '\n';
}
```

در این مثال، تمام محصولاتی که موجودی آن‌ها صفر است حذف می‌شوند.

توجه کنید که ترتیب عناصر در `std::unordered_map` تضمین‌شده نیست.

## مقایسه با Erase-Remove Idiom

یکی از مهم‌ترین دلایل معرفی `std::erase`، ساده‌سازی الگوی قدیمی Erase-Remove Idiom است.

این الگو پیش از ++C20 برای حذف عناصر از Containerهایی استفاده می‌شد که الگوریتم‌های Standard Library با آن‌ها کار می‌کردند.

### روش قدیمی در ++C17

```
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 2, 4, 2, 5
    };

    numbers.erase(
        std::remove(
            numbers.begin(),
            numbers.end(),
            2
        ),
        numbers.end()
    );
}
```

این روش از دو مرحله تشکیل شده است:

- الگوریتم `std::remove` عناصر نامطلوب را از محدوده منطقی ابتدایی کنار می‌زند.
- تابع عضو `erase` بخش اضافی انتهای Container را حذف می‌کند.

### روش مدرن در ++C20

```
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 2, 4, 2, 5
    };

    std::erase(numbers, 2);
}
```

در روش جدید، جزئیات پیاده‌سازی از کد فراخواننده پنهان شده‌اند.

### مقایسه برای Predicate

در روش قدیمی، حذف عناصر بر اساس شرط با `std::remove_if` انجام می‌شد:

```
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 4, 5, 6
    };

    numbers.erase(
        std::remove_if(
            numbers.begin(),
            numbers.end(),
            [](int value) {
                return value % 2 == 0;
            }
        ),
        numbers.end()
    );
}
```

در ++C20 می‌توان همین عملیات را با کد زیر انجام داد:

```
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 4, 5, 6
    };

    std::erase_if(numbers, [](int value) {
        return value % 2 == 0;
    });
}
```

### مزایای روش جدید

استفاده از `std::erase` و `std::erase_if` مزایای زیر را دارد:

- کاهش پیچیدگی کد
- خوانایی بیشتر
- جلوگیری از اشتباه در استفاده از Iteratorها
- یکسان‌سازی API حذف برای Containerهای مختلف
- امکان دریافت مستقیم تعداد عناصر حذف‌شده
- کاهش وابستگی کد فراخواننده به جزئیات الگوریتم حذف

با این حال، Erase-Remove Idiom همچنان برای استانداردهای قدیمی‌تر و الگوریتم‌هایی که به محدوده Iterator نیاز دارند، کاربرد دارد.

## تفاوت std::erase با تابع عضو erase

وجود تابع غیرعضوی `std::erase` به معنی حذف یا جایگزینی تابع عضو `erase` نیست.

این دو API اهداف متفاوتی دارند.

### تابع عضو erase

تابع عضو `erase` برای حذف عناصر بر اساس Iterator، Range یا Key استفاده می‌شود؛ امضاهای دقیق آن به نوع Container بستگی دارند.

برای مثال:

```
#include <vector>

int main() {
    std::vector<int> numbers{
        10, 20, 30, 40, 50
    };

    numbers.erase(numbers.begin() + 2);
}
```

در این مثال، عنصر موجود در موقعیت سوم حذف می‌شود.

### تابع غیرعضوی std::erase

در مقابل، `std::erase` برای حذف تمام عناصر برابر با یک مقدار مشخص طراحی شده است.

```
#include <vector>

int main() {
    std::vector<int> numbers{
        10, 20, 30, 20, 40
    };

    std::erase(numbers, 20);
}
```

### چه زمانی از کدام استفاده کنیم؟

| نیاز                             | انتخاب مناسب            |
| -------------------------------- | ----------------------- |
| حذف عنصر در یک موقعیت مشخص       | Member Function `erase` |
| حذف یک محدوده از Iteratorها      | Member Function `erase` |
| حذف تمام عناصر برابر با یک مقدار | `std::erase`            |
| حذف عناصر بر اساس Predicate      | `std::erase_if`         |
| حذف عناصر از map بر اساس شرط     | `std::erase_if`         |

بنابراین این APIها مکمل یکدیگر هستند و جایگزین کامل یکدیگر محسوب نمی‌شوند.

## پیچیدگی زمانی و کارایی

هزینه زمانی عملیات حذف به ساختار داده Container وابسته است.

### پیچیدگی std::erase در Containerهای مختلف

| Container           | پیچیدگی کلی |
| ------------------- | ----------- |
| `std::vector`       | خطی         |
| `std::deque`        | خطی         |
| `std::list`         | خطی         |
| `std::forward_list` | خطی         |
| `std::basic_string` | خطی         |

در این جدول، منظور از پیچیدگی خطی، رشد هزینه با تعداد عناصر Container است.

### پیچیدگی در std::vector

برای `std::vector`، ابتدا عناصر بررسی می‌شوند و سپس عناصر باقی‌مانده در صورت نیاز جابه‌جا می‌شوند.

اگر تعداد عناصر اولیه `N` باشد، پیچیدگی زمانی کلی `O(N)` است.

همچنین تعداد جابه‌جایی‌ها و Assignmentهای موردنیاز به تعداد عناصر حذف‌شده و موقعیت آن‌ها بستگی دارد.

### پیچیدگی در std::list

در `std::list`، برای یافتن عناصر موردنظر باید Nodeها بررسی شوند.

بنابراین جست‌وجوی عناصر موردنظر پیچیدگی خطی دارد.

اما حذف هر Node پیدا‌شده، بدون نیاز به جابه‌جایی عناصر دیگر انجام می‌شود.

### پیچیدگی در Associative Containerها

برای `std::erase_if` در Associative Containerها، عناصر باید بررسی شوند.

بنابراین عملیات بررسی Predicate در مجموع متناسب با تعداد عناصر Container است.

حذف عناصر درختی یا Hash-based نیز هزینه خاص ساختار داده مربوطه را دارد.

به همین دلیل هزینه دقیق عملیات به نوع Container و تعداد عناصر حذف‌شده بستگی خواهد داشت.

## مدیریت Iteratorها و Referenceها

یکی از مهم‌ترین نکات هنگام حذف عناصر از Containerها، توجه به Iterator Invalidation است.

حذف عنصر ممکن است باعث نامعتبر شدن Iteratorها، Pointerها و Referenceهایی شود که قبلاً به عناصر Container اشاره می‌کردند.

قواعد دقیق این رفتار به نوع Container وابسته است.

### رفتار در std::vector

در `std::vector`، حذف عناصر باعث تغییر موقعیت عناصر بعدی می‌شود.

Iteratorها و Referenceهایی که به عناصر قبل از اولین عنصر حذف‌شده اشاره می‌کنند معتبر باقی می‌مانند.

اما Iteratorها و Referenceهای مربوط به موقعیت حذف‌شده و عناصر بعد از آن نامعتبر می‌شوند.

همچنین Iterator مربوط به `end()` قبلی نیز نامعتبر خواهد شد.

برای مثال:

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{
        10, 20, 30, 40, 50
    };

    auto iterator = values.begin();

    std::erase(values, 30);

    std::cout << *iterator << '\n';
}
```

در این مثال، Iterator به اولین عنصر اشاره می‌کند و چون عنصر موردنظر قبل از آن حذف نشده است، Iterator معتبر باقی می‌ماند.

اما نباید Iterator مربوط به موقعیت حذف‌شده یا بعد از آن را پس از عملیات حذف استفاده کرد.

### رفتار در std::list

در `std::list`، حذف یک Node فقط Iteratorها و Referenceهای مربوط به Node حذف‌شده را نامعتبر می‌کند.

Iteratorهای عناصر باقی‌مانده معتبر هستند.

### رفتار در std::forward_list

در `std::forward_list` نیز حذف Node باعث نامعتبر شدن Iteratorهای مربوط به Nodeهای حذف‌شده می‌شود، اما Iteratorهای Nodeهای دیگر معتبر باقی می‌مانند.

### رفتار در std::deque

قواعد Invalidation در `std::deque` پیچیده‌تر هستند و به موقعیت عناصر حذف‌شده بستگی دارند.

به‌خصوص در حذف عناصری که در بخش‌های ابتدایی، انتهایی یا میانی قرار دارند، رفتار Iteratorها و Referenceها متفاوت است.

بنابراین در صورت نگهداری Iterator یا Reference به عناصر `std::deque`، باید قواعد استاندارد مربوط به `erase` این Container را بررسی کرد.

### رفتار در std::map و std::set

در Associative Containerهایی مانند `std::map` و `std::set`، حذف یک عنصر فقط Iteratorها و Referenceهای مربوط به عناصر حذف‌شده را نامعتبر می‌کند.

Iteratorهای عناصر باقی‌مانده معتبر می‌مانند.

این ویژگی برای حذف انتخابی عناصر از ساختارهای درختی بسیار مفید است.

## نکات مهم درباره Predicate

استفاده از Lambda در `std::erase_if` ساده به نظر می‌رسد، اما رعایت چند نکته برای جلوگیری از رفتار نامطلوب ضروری است.

### عدم تغییر عناصر داخل Predicate

بهتر است Predicate صرفاً وظیفه بررسی شرط را انجام دهد.

برای مثال:

```
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 4, 5
    };

    std::erase_if(numbers, [](const int& value) {
        return value < 3;
    });
}
```

در این مثال، Predicate تنها مقدار عنصر را بررسی می‌کند.

تغییر عناصر Container از داخل Predicate می‌تواند باعث ایجاد رفتار نامطلوب و نقض الزامات الگوریتم شود.

### عدم وابستگی به ترتیب فراخوانی

نباید فرض کنید که Predicate در ترتیب خاصی فراخوانی می‌شود، مگر آنکه چنین ترتیبی صراحتاً توسط قرارداد عملیات تضمین شده باشد.

طراحی Predicate باید مستقل از ترتیب فراخوانی آن باشد.

### استفاده از Capture در Lambda

در مواردی که شرط حذف به یک مقدار خارجی وابسته است، می‌توان از Capture استفاده کرد.

```
#include <vector>

int main() {
    std::vector<int> numbers{
        10, 20, 30, 40, 50
    };

    const int threshold = 30;

    std::erase_if(numbers, [threshold](int value) {
        return value > threshold;
    });
}
```

در این مثال، تمام اعداد بزرگ‌تر از `threshold` حذف می‌شوند.

استفاده از Capture با Value باعث می‌شود Lambda یک نسخه از مقدار موردنظر را نگهداری کند.

اگر از Reference Capture استفاده می‌کنید، باید طول عمر متغیر خارجی و معتبر بودن Reference را در نظر بگیرید.

## محدودیت‌ها و مواردی که باید در نظر گرفت

با وجود سادگی و کاربردی بودن این توابع، استفاده از آن‌ها محدودیت‌هایی نیز دارد.

### تغییر اندازه Container

توابع `std::erase` و `std::erase_if` اندازه Container را تغییر می‌دهند.

بنابراین برای Containerهایی که اندازه ثابت دارند، مانند `std::array`، این APIها قابل استفاده نیستند.

### حذف بر اساس Equality

تابع `std::erase` بر اساس مقایسه برابری عمل می‌کند.

بنابراین نوع عناصر باید قابلیت مقایسه معتبر با مقدار ورودی را داشته باشد.

در صورت استفاده از Typeهای سفارشی، باید مطمئن شوید که عملیات مقایسه موردنیاز به‌درستی تعریف شده است.

### نبود Overload عمومی برای تمام Containerها

این توابع برای تمام انواع Rangeها، Viewها یا Containerهای سفارشی ارائه نشده‌اند.

برای مثال، نمی‌توان فرض کرد هر نوعی که دارای `begin()` و `end()` است، الزاماً با `std::erase` سازگار خواهد بود.

در صورت استفاده از Container سفارشی، ممکن است همچنان نیاز به پیاده‌سازی عملیات حذف اختصاصی یا استفاده از الگوریتم‌های استاندارد داشته باشید.

### عدم حذف مستقیم از Viewها

نوع `std::span` یا Viewهایی مانند `std::views::filter` مالک عناصر نیستند و عملیات حذف از ساختار داده اصلی را انجام نمی‌دهند.

در چنین شرایطی باید Container مالک داده را تغییر دهید، نه خود View را.

### هزینه جابه‌جایی عناصر

در Containerهایی مانند `std::vector`، حذف عناصر می‌تواند باعث جابه‌جایی عناصر باقی‌مانده شود.

بنابراین اگر حذف‌های مکرر و پراکنده در حجم بسیار زیادی از داده انجام می‌شوند، انتخاب نوع Container اهمیت زیادی دارد.

برای مثال، در سناریوهایی که حذف و درج مکرر عناصر در بخش‌های مختلف ساختار داده انجام می‌شود، ممکن است `std::list` یا ساختار داده دیگری انتخاب مناسب‌تری باشد؛ البته این انتخاب باید با توجه به الگوی دسترسی، Cache Locality و هزینه حافظه انجام شود.

## نکات مربوط به Typeهای سفارشی

تابع `std::erase` فقط محدود به Typeهای Primitive نیست و برای Typeهای سفارشی نیز قابل استفاده است.

شرط اصلی این است که مقایسه موردنیاز معتبر باشد.

### مثال با Class سفارشی

```
#include <iostream>
#include <string>
#include <vector>

class User {
public:
    int id;
    std::string name;

    friend bool operator==(const User& lhs, const User& rhs) {
        return lhs.id == rhs.id;
    }
};

int main() {
    std::vector<User> users{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"},
        {2, "David"}
    };

    const User target{2, ""};

    const auto removedCount = std::erase(users, target);

    std::cout << "Removed: " << removedCount << '\n';
}
```

در این مثال، عملگر `operator==` تنها فیلد `id` را مقایسه می‌کند.

بنابراین هر کاربری که شناسه آن برابر با شناسه کاربر هدف باشد حذف خواهد شد.

### نکته طراحی درباره operator==

تعریف `operator==` باید با معنای منطقی Equality در Type موردنظر سازگار باشد.

اگر هدف فقط حذف عناصر بر اساس یک فیلد خاص است و تعریف Equality عمومی برای Class مناسب نیست، استفاده از `std::erase_if` انتخاب بهتری خواهد بود.

```
#include <string>
#include <vector>

struct User {
    int id;
    std::string name;
};

int main() {
    std::vector<User> users{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"},
        {2, "David"}
    };

    std::erase_if(users, [](const User& user) {
        return user.id == 2;
    });
}
```

این روش از تغییر معنای Equality عمومی Type جلوگیری می‌کند.

## تعامل با Move Semantics و Typeهای غیرقابل کپی

عملیات حذف در Containerهایی مانند `std::vector` می‌تواند شامل Move Assignment یا Copy Assignment عناصر باقی‌مانده باشد.

بنابراین Type مورد استفاده باید الزامات عملیاتی موردنیاز Container و الگوریتم را برآورده کند.

برای مثال، Typeهای Move-only مانند `std::unique_ptr` نیز می‌توانند با `std::erase_if` استفاده شوند.

```
#include <memory>
#include <vector>

int main() {
    std::vector<std::unique_ptr<int>> values;

    values.push_back(std::make_unique<int>(10));
    values.push_back(std::make_unique<int>(20));
    values.push_back(std::make_unique<int>(30));

    std::erase_if(values, [](const auto& value) {
        return *value == 20;
    });
}
```

در این مثال، عنصر مربوط به مقدار `20` حذف می‌شود.

از آنجا که `std::unique_ptr` قابل کپی نیست، این مثال نشان می‌دهد که عملیات حذف به کپی‌پذیر بودن عناصر محدود نیست.

با این حال، اگر Type موردنظر نه Move Assignment و نه Copy Assignment موردنیاز عملیات را پشتیبانی کند، استفاده از `std::erase` یا `std::erase_if` برای برخی Containerها امکان‌پذیر نخواهد بود.

## تفاوت رفتار با std::remove

یکی از اشتباهات رایج، تصور یکسان بودن رفتار `std::remove` و `std::erase` است.

این دو تابع عملیات متفاوتی انجام می‌دهند.

### رفتار std::remove

تابع `std::remove`:

- عناصر موردنظر را از محدوده منطقی ابتدایی کنار می‌زند.
- اندازه واقعی Container را تغییر نمی‌دهد.
- یک Iterator به انتهای محدوده منطقی جدید برمی‌گرداند.

### رفتار std::erase

تابع `std::erase`:

- عناصر موردنظر را حذف می‌کند.
- اندازه واقعی Container را کاهش می‌دهد.
- تعداد عناصر حذف‌شده را برمی‌گرداند.

برای درک این تفاوت:

```
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 2, 4
    };

    auto newEnd = std::remove(
        numbers.begin(),
        numbers.end(),
        2
    );

    std::cout << "Size: " << numbers.size() << '\n';

    numbers.erase(newEnd, numbers.end());

    std::cout << "Size: " << numbers.size() << '\n';
}
```

خروجی:

```
Size: 5
Size: 3
```

در مقابل، با استفاده از `std::erase` نیازی به مدیریت این دو مرحله نیست.

## بررسی constexpr بودن

در استاندارد ++C20، بسیاری از عملیات مربوط به Containerهای استاندارد و الگوریتم‌های موردنیاز برای این قابلیت‌ها به‌صورت `constexpr` قابل استفاده شده‌اند.

Overloadهای استاندارد `std::erase` و `std::erase_if` که برای Containerهای مربوطه ارائه شده‌اند نیز در ++C20 دارای قابلیت `constexpr` هستند.

این ویژگی امکان استفاده از آن‌ها را در Contextهای Constant Evaluation فراهم می‌کند، البته به شرطی که تمام عملیات و داده‌های مورد استفاده با الزامات Constant Evaluation سازگار باشند.

برای مثال، صرفاً `constexpr` بودن تابع به این معنی نیست که هر Container یا هر Predicate در هر Contextی قابل استفاده خواهد بود.

همچنین پشتیبانی واقعی از Constant Evaluation به پیاده‌سازی Standard Library و استاندارد زبان مورد استفاده وابسته است.

## بررسی Feature-Test Macro

برای بررسی پشتیبانی کتابخانه استاندارد از قابلیت‌های حذف جدید می‌توان از Feature-Test Macro استفاده کرد.

ماکروی مرتبط عبارت است از:

```
__cpp_lib_erase_if
```

مقدار استاندارد این ماکرو برای قابلیت‌های مربوط به `erase_if` برابر است با:

```
202002L
```

برای بررسی پشتیبانی می‌توان از Header مربوط به `<version>` استفاده کرد:

```
#include <version>

#if defined(__cpp_lib_erase_if) && __cpp_lib_erase_if >= 202002L
    #define HAS_CPP20_ERASE_IF
#endif
```

توجه کنید که Feature-Test Macroهای کتابخانه استاندارد، پشتیبانی Standard Library را بررسی می‌کنند، نه صرفاً قابلیت‌های Frontend کامپایلر را.

همچنین باید بین پشتیبانی از زبان ++C20 و پشتیبانی از قابلیت‌های کتابخانه استاندارد آن تفاوت قائل شد.

ممکن است کامپایلر از Syntax استاندارد ++C20 پشتیبانی کند، اما نسخه Standard Library همراه آن هنوز تمام قابلیت‌های موردنظر را پیاده‌سازی نکرده باشد.

## اشتباهات رایج

### استفاده از std::erase برای std::map

کد زیر معتبر نیست:

```
#include <map>

int main() {
    std::map<int, int> values{
        {1, 10},
        {2, 20},
        {3, 30}
    };

    std::erase(values, 2);
}
```

برای حذف عناصر از `std::map` بر اساس شرط باید از `std::erase_if` استفاده کرد:

```
#include <map>

int main() {
    std::map<int, int> values{
        {1, 10},
        {2, 20},
        {3, 30}
    };

    std::erase_if(values, [](const auto& entry) {
        return entry.first == 2;
    });
}
```

### اشتباه گرفتن مقدار بازگشتی با اندازه جدید

مقدار بازگشتی `std::erase` تعداد عناصر حذف‌شده است، نه اندازه Container پس از حذف.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 2, 3};

    const auto result = std::erase(values, 2);

    std::cout << result << '\n';
    std::cout << values.size() << '\n';
}
```

خروجی:

```
2
2
```

### استفاده از Iterator نامعتبر

پس از حذف عناصر از `std::vector` نباید بدون بررسی اعتبار، Iteratorهای قبلی را مجدداً استفاده کرد.

بهتر است پس از انجام عملیات حذف، در صورت نیاز Iteratorهای جدید از Container دریافت شوند.

### استفاده از std::erase روی Container غیرقابل تغییر

از آنجا که این توابع محتوای Container را تغییر می‌دهند، نمی‌توان آن‌ها را مستقیماً روی Containerهای `const` اجرا کرد.

```
#include <vector>

int main() {
    const std::vector<int> values{1, 2, 3, 2};

    // This does not compile.
    // std::erase(values, 2);
}
```

### استفاده از std::remove بدون erase

استفاده از `std::remove` به‌تنهایی باعث کاهش اندازه `std::vector` نمی‌شود.

اگر هدف حذف واقعی عناصر از Container است، باید از Erase-Remove Idiom یا API جدید `std::erase` استفاده کرد.

## توصیه‌های طراحی و Best Practices

برای استفاده حرفه‌ای از قابلیت‌های حذف در ++C20 رعایت نکات زیر توصیه می‌شود:

- برای حذف تمام عناصر برابر با یک مقدار از `std::vector`، `std::deque`، `std::list`، `std::forward_list` و `std::string` از `std::erase` استفاده کنید.
- برای حذف عناصر بر اساس شرط از `std::erase_if` استفاده کنید.
- برای Associative Containerها مانند `std::map` و `std::set` از `std::erase_if` استفاده کنید.
- اگر فقط یک عنصر در موقعیت مشخصی باید حذف شود، تابع عضو `erase` انتخاب مناسب‌تری است.
- هنگام حذف از `std::vector` و `std::deque` به Invalidation مربوط به Iteratorها و Referenceها توجه کنید.
- در Lambdaهای مورد استفاده به‌عنوان Predicate، از Side Effectهای غیرضروری اجتناب کنید.
- در طراحی Typeهای سفارشی، تنها زمانی `operator==` را تعریف کنید که معنای عمومی و منطقی Equality برای Type مشخص باشد.
- در پروژه‌هایی که با استانداردهای قدیمی‌تر کار می‌کنند، از Erase-Remove Idiom استفاده کنید.
- در پروژه‌های ++C20 و جدیدتر، برای عملیات حذف متداول، APIهای جدید را به روش‌های قدیمی ترجیح دهید.
- برای Typeهای غیرقابل کپی، قابلیت Move Assignment و الزامات Container موردنظر را بررسی کنید.

## جمع‌بندی

قابلیت‌های `std::erase` و `std::erase_if` که در استاندارد ++C20 معرفی شدند، روش حذف عناصر از Containerهای استاندارد را ساده‌تر و خواناتر کرده‌اند.

تابع `std::erase` برای حذف تمام عناصر برابر با یک مقدار مشخص استفاده می‌شود، در حالی که `std::erase_if` امکان حذف عناصر بر اساس یک Predicate را فراهم می‌کند.

مهم‌ترین نکات این قابلیت‌ها عبارت‌اند از:

- حذف مستقیم عناصر بدون نیاز به نوشتن Erase-Remove Idiom
- پشتیبانی از Sequence Containerهای متداول
- پشتیبانی از Associative و Unordered Associative Containerها از طریق `std::erase_if`
- بازگرداندن تعداد عناصر حذف‌شده
- حفظ قواعد اختصاصی Invalidation هر Container
- کاهش پیچیدگی و افزایش خوانایی کد

در نهایت، استفاده از این دو API در پروژه‌های مبتنی بر ++C20 باعث می‌شود عملیات حذف عناصر به شکل واضح‌تر، استانداردتر و قابل نگهداری‌تری نوشته شود؛ بدون آنکه نیاز باشد توسعه‌دهنده در هر بار حذف، جزئیات الگوریتم و مدیریت Iteratorها را به‌صورت دستی پیاده‌سازی کند.

محتوا را کوتاه‌تر و متمرکزتر کنجدول تصمیم‌گیری کاربردی اضافه کنلحن و واژگان را یکدست کن

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>