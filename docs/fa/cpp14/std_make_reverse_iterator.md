<div align="center">

[🇺🇸 English](../../en/cpp14/std_make_reverse_iterator.md) | [🇮🇷 فارسی](./std_make_reverse_iterator.md)

</div>

---

# آموزش جامع `std::make_reverse_iterator` در ++C14

## مقدمه و معرفی قابلیت

در کتابخانه استاندارد ++C، پیمایش معکوس یک مجموعه از عناصر، در بسیاری از موقعیت‌ها کاربرد دارد؛ برای مثال، زمانی که لازم است عناصر یک `std::vector` از انتها به ابتدا پردازش شوند، آخرین رخداد یک مقدار پیدا شود یا عملیات خاصی روی عناصر انتهایی یک `std::string` انجام گیرد.

یکی از ابزارهای استاندارد برای این کار، تابع `std::make_reverse_iterator` است که از استاندارد ++C14 به کتابخانه استاندارد اضافه شد. این تابع یک Iterator معمولی را دریافت می‌کند و یک `std::reverse_iterator` می‌سازد که جهت پیمایش را معکوس می‌کند.

این قابلیت در Header استاندارد `<iterator>` تعریف شده است و به‌جای پیاده‌سازی دستی Iterator معکوس، ساخت آن را ساده‌تر و خواناتر می‌کند.

نمونه‌ای از کاربرد این تابع را در کد زیر مشاهده می‌کنید:

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    auto reverseIt = std::make_reverse_iterator(values.end());

    std::cout << *reverseIt << '\n';
}
```

خروجی برنامه به شکل زیر است:

```
50
```

دلیل این رفتار آن است که `values.end()` به موقعیت بعد از آخرین عنصر اشاره می‌کند و `std::make_reverse_iterator` با ایجاد یک Reverse Iterator، عنصر قبل از آن موقعیت را به‌عنوان عنصر فعلی در نظر می‌گیرد.

برای استفاده صحیح از این تابع، باید تفاوت میان موقعیت داخلی Reverse Iterator و عنصری که از طریق Dereference در دسترس قرار می‌دهد، به‌خوبی درک شود.

## مفهوم Reverse Iterator و نحوه عملکرد آن

### تفاوت جهت پیمایش معمولی و معکوس

در یک Container مانند `std::vector`، پیمایش عادی از ابتدای مجموعه به انتهای آن انجام می‌شود.

برای نمونه، Iteratorهای `begin()` و `end()` محدوده‌ای را مشخص می‌کنند که عنصر اول را شامل می‌شود، اما موقعیت `end()` را شامل نمی‌شود.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    for (auto it = values.begin(); it != values.end(); ++it) {
        std::cout << *it << ' ';
    }
}
```

خروجی برنامه:

```
10 20 30
```

در پیمایش معکوس، جهت حرکت برعکس می‌شود. برای این منظور، کتابخانه استاندارد نوع `std::reverse_iterator` را ارائه می‌کند.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto first = std::make_reverse_iterator(values.end());
    auto last = std::make_reverse_iterator(values.begin());

    for (auto it = first; it != last; ++it) {
        std::cout << *it << ' ';
    }
}
```

خروجی:

```
30 20 10
```

در این مثال، `first` به‌صورت معکوس از آخرین عنصر شروع می‌کند و `last` مرز انتهایی محدوده معکوس را مشخص می‌کند.

### رابطه میان Reverse Iterator و Iterator پایه

مهم‌ترین نکته در کار با `std::reverse_iterator` این است که Iterator پایه آن مستقیماً به عنصری که از طریق Dereference مشاهده می‌شود، اشاره نمی‌کند.

اگر `r` یک Reverse Iterator معتبر باشد، رابطه مفهومی زیر برقرار است:

```
*r == *std::prev(r.base())
```

این رابطه برای Reverse Iteratorهای قابل Dereference و در محدوده معتبر قابل استفاده است.

تابع عضو `base()`، Iterator پایه را برمی‌گرداند. اگر `r` یک Reverse Iterator ساخته‌شده از `values.end()` باشد، نتیجه `r.base()` برابر `values.end()` خواهد بود، نه Iterator آخرین عنصر.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto r = std::make_reverse_iterator(values.end());

    std::cout << *r << '\n';
    std::cout << *std::prev(r.base()) << '\n';
}
```

خروجی:

```
30
30
```

این تفاوت، طراحی اصلی Reverse Iterator را توضیح می‌دهد: موقعیت داخلی آن مرز میان دو عنصر است و عنصر قابل دسترسی، عنصر بلافاصله قبل از آن مرز محسوب می‌شود.

### دلیل استفاده از Iterator پایه به‌عنوان مرز

استفاده از Iterator پایه باعث می‌شود محدوده‌های معکوس با محدوده‌های عادی هماهنگ باشند.

برای مثال، اگر یک Container شامل پنج عنصر باشد، محدوده عادی آن به شکل زیر تعریف می‌شود:

```
[values.begin(), values.end())
```

محدوده معکوس متناظر به شکل زیر است:

```
[std::make_reverse_iterator(values.end()),
 std::make_reverse_iterator(values.begin()))
```

این محدوده همان عناصر را در جهت مخالف پیمایش می‌کند، بدون اینکه لازم باشد برای عنصر اول یا آخر، یک Iterator ویژه و خارج از قرارداد معمول Container تعریف شود.

## معرفی تابع `std::make_reverse_iterator`

### امضای تابع و Header موردنیاز

تابع `std::make_reverse_iterator` در Header استاندارد `<iterator>` قرار دارد.

شکل کلی استفاده از آن به این صورت است:

```
#include <iterator>

auto reverseIt = std::make_reverse_iterator(it);
```

در اینجا، `it` یک Iterator است و نوع خروجی تابع بر اساس نوع آن تعیین می‌شود.

از نظر مفهومی، امضای استاندارد این تابع در ++C14 به شکل زیر است:

```
template<class Iterator>
constexpr std::reverse_iterator<Iterator>
make_reverse_iterator(Iterator i);
```

این امضا شکل ساده‌شده‌ای از اعلان تابع است؛ جزئیات کامل اعلان استاندارد، از جمله ویژگی‌های مربوط به پارامترها و محدودیت‌های نوع، در متن استاندارد تعریف می‌شوند.

ویژگی‌های اصلی تابع عبارت‌اند از:

- از استاندارد ++C14 در دسترس است.
- در Namespace استاندارد `std` قرار دارد.
- نوع Iterator پایه را از آرگومان دریافت می‌کند.
- یک `std::reverse_iterator` متناظر می‌سازد.
- برای استفاده از آن باید Header استاندارد `<iterator>` را وارد کرد.

### نحوه تعیین نوع خروجی

نوع خروجی تابع به نوع Iterator ورودی وابسته است.

برای مثال، اگر Iterator ورودی از نوع `std::vector<int>::iterator` باشد، نوع خروجی به شکل زیر خواهد بود:

```
std::reverse_iterator<std::vector<int>::iterator>
```

اگر Iterator ورودی از نوع `std::vector<int>::const_iterator` باشد، نوع خروجی به شکل زیر خواهد بود:

```
std::reverse_iterator<std::vector<int>::const_iterator>
```

استفاده از `auto` معمولاً خوانایی کد را افزایش می‌دهد و از تکرار نام‌های طولانی انواع Iterator جلوگیری می‌کند.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto mutableReverse =
        std::make_reverse_iterator(values.end());

    auto constReverse =
        std::make_reverse_iterator(values.cend());

    *mutableReverse = 30;

    // *constReverse = 40; // Compilation error
}
```

در این مثال، `mutableReverse` امکان تغییر عناصر را فراهم می‌کند، اما `constReverse` برای خواندن عناصر مناسب است و اجازه تغییر آن‌ها را نمی‌دهد.

## تفاوت `std::make_reverse_iterator` با سازنده مستقیم

### مقایسه روش‌های ساخت Reverse Iterator

پیش از ++C14، برنامه‌نویسان می‌توانستند `std::reverse_iterator` را مستقیماً از طریق سازنده آن ایجاد کنند.

این روش همچنان معتبر است:

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    std::reverse_iterator<std::vector<int>::iterator> r(
        values.end()
    );
}
```

در ++C14 می‌توان همین کار را به شکل خلاصه‌تری انجام داد:

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto r = std::make_reverse_iterator(values.end());
}
```

تفاوت اصلی در خوانایی و استنتاج نوع است، نه در جهت پیمایش یا معنای Iterator ساخته‌شده.

| ویژگی                  | سازنده مستقیم                      | `std::make_reverse_iterator`       |
| ---------------------- | ---------------------------------- | ---------------------------------- |
| نسخه استاندارد         | پیش از ++C14 نیز قابل استفاده بود  | ++C14                              |
| نیاز به نوشتن نوع کامل | در صورت استفاده از نوع صریح، بله   | خیر                                |
| استنتاج نوع خروجی      | با `auto` یا روش‌های دیگر ممکن است | نوع قالب از آرگومان استنتاج می‌شود |
| جهت پیمایش             | معکوس                              | معکوس                              |
| نیاز به Header         | `<iterator>`                       | `<iterator>`                       |

برای کدهای مدرن، استفاده از `std::make_reverse_iterator` انتخاب مناسبی است؛ به‌ویژه زمانی که نوع دقیق Iterator طولانی یا پیچیده باشد.

البته اگر نوع دقیق Reverse Iterator بخشی از امضای یک تابع یا یک Interface عمومی باشد، نوشتن نوع صریح ممکن است خوانایی یا وضوح بیشتری ایجاد کند.

## الزامات Iterator ورودی

### قابلیت‌های لازم برای پیمایش معکوس

هر Iterator معمولی را نمی‌توان بدون توجه به قابلیت‌های آن به Reverse Iterator تبدیل کرد.

نوع `std::reverse_iterator` به Iterator پایه‌ای نیاز دارد که از عملیات موردنیاز برای پیمایش معکوس پشتیبانی کند. به‌طور مشخص، Reverse Iterator استاندارد برای کارکرد صحیح به Iterator با قابلیت‌های `BidirectionalIterator` یا قوی‌تر نیاز دارد.

در عمل، این موضوع به معنی پشتیبانی از عملیات حرکت به عقب در Iterator پایه است.

Containerهایی مانند موارد زیر معمولاً Iteratorهای مناسبی ارائه می‌کنند:

- `std::vector`
- `std::deque`
- `std::list`
- `std::array`
- `std::string`

در مقابل، Iteratorهای `std::forward_list` و Iteratorهای خروجی یا ورودی یک‌طرفه، به‌تنهایی برای استفاده معمول از `std::reverse_iterator` مناسب نیستند.

برای مثال، `std::vector` از Random Access Iterator استفاده می‌کند؛ بنابراین تمام عملیات موردنیاز Reverse Iterator را پشتیبانی می‌کند.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto r = std::make_reverse_iterator(values.end());

    ++r;
    --r;

    auto next = r + 1;
}
```

در این مثال، علاوه بر حرکت معکوس، عملیات حسابی نیز به‌دلیل قابلیت Random Access Iterator پایه امکان‌پذیر است.

نکته مهم این است که پشتیبانی از عملیات اضافی مانند `r + n` به قابلیت Iterator پایه وابسته است. وجود `std::make_reverse_iterator` به این معنی نیست که تمام Reverse Iteratorها الزاماً Random Access هستند.

## پیمایش محدوده‌ها با استفاده از تابع

### پیمایش کامل از انتها به ابتدا

رایج‌ترین کاربرد تابع، ساخت دو مرز برای پیمایش معکوس یک Container است.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    auto first = std::make_reverse_iterator(values.end());
    auto last = std::make_reverse_iterator(values.begin());

    for (auto it = first; it != last; ++it) {
        std::cout << *it << ' ';
    }
}
```

خروجی:

```
40 30 20 10
```

این روش زمانی مفید است که یک Algorithm یا تابع عمومی به دو Iterator نیاز داشته باشد و بخواهید جهت پیمایش را معکوس کنید.

### پیمایش معکوس با Range-based for

اگر فقط قصد پیمایش معکوس یک Container استاندارد را دارید، استفاده از `rbegin()` و `rend()` معمولاً ساده‌تر است.

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    for (auto it = values.rbegin(); it != values.rend(); ++it) {
        std::cout << *it << ' ';
    }
}
```

خروجی:

```
40 30 20 10
```

در این مورد، نیازی به ساخت Reverse Iterator به‌صورت دستی نیست، زیرا Container خودش مرزهای پیمایش معکوس را فراهم می‌کند.

با این حال، `std::make_reverse_iterator` هنگامی اهمیت پیدا می‌کند که بخواهید یک محدوده دلخواه را معکوس کنید یا Iteratorهای موجود را به شکل معکوس در اختیار یک تابع قرار دهید.

### معکوس کردن بخشی از یک محدوده

فرض کنید فقط لازم است سه عنصر انتهایی یک `std::vector` را در جهت معکوس پردازش کنید.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50, 60};

    auto first = std::make_reverse_iterator(values.end());
    auto last = std::make_reverse_iterator(values.end() - 3);

    for (auto it = first; it != last; ++it) {
        std::cout << *it << ' ';
    }
}
```

خروجی:

```
60 50 40
```

در این مثال، مرز انتهایی معکوس از روی Iterator متناظر با موقعیت قبل از سه عنصر آخر ساخته شده است.

این تکنیک برای پردازش بخش انتهایی یک Container بدون تغییر ترتیب واقعی عناصر آن مفید است.

باید توجه داشت که عبارت `values.end() - 3` به دلیل نیاز به Random Access فقط برای Iteratorهای مناسب، مانند Iteratorهای `std::vector`، قابل استفاده است. برای Containerهایی که فقط Bidirectional Iterator دارند، باید از عملیات مناسب همان نوع استفاده کرد و پیش از عقب رفتن، معتبر بودن موقعیت را بررسی کرد.

## کاربردهای عملی و حرفه‌ای

### پیدا کردن آخرین رخداد یک مقدار

گاهی لازم است آخرین رخداد یک مقدار در یک محدوده پیدا شود. پیمایش معکوس امکان جست‌وجو از انتهای مجموعه را فراهم می‌کند.

```
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{5, 8, 3, 8, 2, 8, 1};

    auto reverseIt = std::find(
        values.rbegin(),
        values.rend(),
        8
    );

    if (reverseIt != values.rend()) {
        auto forwardIt = reverseIt.base();
        --forwardIt;

        std::cout << *forwardIt << '\n';
        std::cout << std::distance(values.begin(), forwardIt) << '\n';
    }
}
```

خروجی:

```
8
5
```

در این مثال، `std::find` از آخرین عنصر به سمت ابتدا حرکت می‌کند و اولین `8` را در جهت معکوس پیدا می‌کند؛ این عنصر همان آخرین رخداد `8` در ترتیب عادی است.

نکته مهم در این کاربرد، تبدیل نتیجه Reverse Iterator به Iterator عادی است. مقدار `reverseIt.base()` به موقعیت بعد از عنصر پیدا‌شده در ترتیب عادی اشاره می‌کند؛ بنابراین برای رسیدن به خود عنصر باید یک بار به عقب حرکت کرد.

این تبدیل فقط هنگامی معتبر است که Reverse Iterator به عنصری واقعی اشاره کند. اگر نتیجه جست‌وجو برابر `values.rend()` باشد، نباید روی `base()` آن عمل Dereference یا کاهش انجام شود.

### پردازش عناصر تا رسیدن به شرط

در برخی الگوریتم‌ها لازم است از انتهای مجموعه حرکت کنیم تا به یک شرط خاص برسیم.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 0, 5, 6};

    auto it = std::make_reverse_iterator(values.end());
    auto stop = std::make_reverse_iterator(values.begin());

    while (it != stop && *it != 0) {
        std::cout << *it << ' ';
        ++it;
    }
}
```

خروجی:

```
6 5
```

در این مثال، پیمایش معکوس قبل از رسیدن به مقدار صفر متوقف می‌شود.

ترتیب بررسی شرط‌ها اهمیت دارد: ابتدا باید بررسی شود که Iterator به مرز انتهایی نرسیده باشد و تنها پس از آن می‌توان عنصر را Dereference کرد.

### استفاده با Algorithmهای استاندارد

بسیاری از Algorithmهای استاندارد به‌جای وابستگی به یک Container خاص، با Iteratorها کار می‌کنند. در نتیجه، می‌توان Reverse Iterator را برای تغییر جهت پیمایش به آن‌ها داد.

```
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    auto count = std::count_if(
        std::make_reverse_iterator(values.end()),
        std::make_reverse_iterator(values.begin()),
        [](int value) {
            return value % 2 == 1;
        }
    );

    std::cout << count << '\n';
}
```

خروجی:

```
3
```

در این مثال، `std::count_if` ترتیب پیمایش معکوس را رعایت می‌کند؛ با این حال، چون هدف شمارش است، نتیجه مستقل از جهت پیمایش خواهد بود.

این موضوع نشان می‌دهد که معکوس کردن Iteratorها فقط برای نمایش معکوس نیست، بلکه راهی برای استفاده مجدد از Algorithmهای عمومی در جهت پیمایش متفاوت نیز محسوب می‌شود.

## رابطه با `rbegin()` و `rend()`

### تفاوت کاربردی دو روش

در بسیاری از Containerهای استاندارد، توابع `rbegin()` و `rend()` مستقیماً Reverse Iterator مناسب را برمی‌گردانند.

برای مثال:

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto a = values.rbegin();
    auto b = std::make_reverse_iterator(values.end());
}
```

برای این Container، `a` و `b` از نظر موقعیت و نوع Iterator معادل‌اند.

با این حال، این دو روش همیشه جایگزین یکدیگر نیستند:

- استفاده از `rbegin()` و `rend()` برای پیمایش معکوس کامل Container طبیعی‌تر است.
- استفاده از `std::make_reverse_iterator` زمانی مفید است که مرزهای یک محدوده دلخواه در اختیار باشند.
- در کدهای Generic، ممکن است تنها Iteratorهای ابتدا و انتهای یک محدوده در دسترس باشند و تابع `rbegin()` وجود نداشته باشد.
- برای معکوس کردن یک بخش از محدوده، می‌توان از تبدیل مرزهای آن به Reverse Iterator استفاده کرد.

بنابراین، انتخاب روش باید بر اساس ساختار داده و هدف الگوریتم انجام شود.

## تبدیل میان Iterator معکوس و عادی

### استفاده صحیح از `base()`

تابع عضو `base()`، Iterator پایه را برمی‌گرداند. این تابع برای تبدیل موقعیت Reverse Iterator به موقعیت متناظر در پیمایش عادی استفاده می‌شود.

```
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    auto r = std::make_reverse_iterator(values.end());
    auto base = r.base();

    std::cout << *r << '\n';
    std::cout << (base == values.end()) << '\n';
}
```

خروجی:

```
40
1
```

در این مثال، Reverse Iterator به عنصر `40` دسترسی دارد، اما Iterator پایه برابر `values.end()` است.

اگر لازم باشد Iterator عادی متناظر با عنصر `40` را به دست آوریم، می‌توانیم یک گام به عقب حرکت کنیم:

```
auto forwardIt = r.base();
--forwardIt;
```

این عملیات باید فقط در شرایطی انجام شود که Iterator پایه پس از `begin()` قرار داشته باشد. کاهش `begin()` یا استفاده از موقعیت نامعتبر، رفتار تعریف‌نشده ایجاد می‌کند.

### تفاوت موقعیت و عنصر در تبدیل Iteratorها

در تبدیل Iteratorها باید میان این دو مفهوم تفاوت قائل شد:

- موقعیت Iterator پایه
- عنصر قابل دسترسی از طریق Reverse Iterator

این تفاوت در حذف عناصر، درج عناصر، محاسبه فاصله‌ها و انتقال نتیجه الگوریتم‌ها به کدهای دیگر اهمیت زیادی دارد.

برای مثال، اگر یک Reverse Iterator آخرین عنصر یک `std::vector` را پیدا کرده باشد، استفاده مستقیم از `base()` به‌عنوان Iterator همان عنصر اشتباه است. برای رسیدن به عنصر باید یک گام به عقب حرکت کرد.

## محدودیت‌ها و نکات مربوط به ایمنی

### اعتبار Iterator و طول عمر Container

ساخت یک Reverse Iterator به‌خودی‌خود اعتبار Iterator پایه را تضمین نمی‌کند.

اگر Iterator پایه نامعتبر باشد، Reverse Iterator ساخته‌شده نیز برای استفاده معتبر نیست.

برای نمونه، عملیات‌هایی مانند افزایش ظرفیت یک `std::vector` ممکن است باعث جابه‌جایی حافظه و نامعتبر شدن Iteratorهای قبلی شوند.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto r = std::make_reverse_iterator(values.end());

    values.push_back(4);

    // r may be invalid after reallocation.
}
```

در این مثال، افزودن عنصر جدید ممکن است باعث Reallocation شود. اگر چنین اتفاقی رخ دهد، استفاده از `r` معتبر نیست.

حتی در مواردی که Reallocation رخ نمی‌دهد، باید قواعد اعتبار Iterator همان Container و همان عملیات را بررسی کرد؛ زیرا رفتار اعتبارسنجی برای تمام Containerها یکسان نیست.

### خطر Dereference کردن مرز انتهایی

مرز انتهایی محدوده معکوس، مانند `values.rend()`، به هیچ عنصر قابل دسترسی اشاره نمی‌کند.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto first = std::make_reverse_iterator(values.end());
    auto last = std::make_reverse_iterator(values.begin());

    if (first != last) {
        int value = *first;
        (void)value;
    }
}
```

بررسی مرز قبل از Dereference باعث می‌شود کد برای محدوده‌های خالی نیز قابل استفاده باشد.

در مقابل، Dereference کردن Reverse Iterator برابر با `last`، در شرایط معمول، رفتار تعریف‌نشده دارد.

### پشتیبانی نکردن از Containerهای یک‌طرفه

Iteratorهای یک‌طرفه مانند Iteratorهای `std::forward_list` امکان حرکت به عقب را فراهم نمی‌کنند.

از آنجا که Reverse Iterator باید بتواند در جهت مخالف حرکت کند، استفاده از چنین Iteratorهایی با `std::make_reverse_iterator` راهکار استاندارد و مناسبی نیست.

در این شرایط، بسته به نیاز می‌توان از روش‌های دیگری استفاده کرد:

- کپی کردن عناصر در یک Container مناسب‌تر
- ذخیره عناصر در ساختاری با قابلیت پیمایش دوطرفه
- استفاده از الگوریتمی که بدون نیاز به پیمایش معکوس، نتیجه موردنظر را تولید کند

انتخاب روش مناسب به هزینه حافظه، پیچیدگی زمانی و نیازهای واقعی برنامه وابسته است.

## نکات مربوط به ++C14 و نسخه‌های دیگر استاندارد

### تفاوت ++C11 و ++C14

قابلیت `std::reverse_iterator` پیش از ++C14 نیز در کتابخانه استاندارد وجود داشت، اما تابع کمکی `std::make_reverse_iterator` در ++C14 اضافه شد.

در نتیجه، برای کدهای ++C11 می‌توان از سازنده مستقیم `std::reverse_iterator` استفاده کرد.

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    std::reverse_iterator<std::vector<int>::iterator> r(
        values.end()
    );
}
```

در ++C14 می‌توان به‌جای نوشتن نوع کامل، از تابع کمکی استفاده کرد:

```
#include <iterator>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto r = std::make_reverse_iterator(values.end());
}
```

این تغییر، یک بهبود در رابط کتابخانه استاندارد است و به معنی تغییر در مفهوم Reverse Iterator نیست.

### تفاوت با قابلیت‌های Ranges در ++C20

استاندارد ++C20 قابلیت‌های گسترده‌تری برای کار با Rangeها معرفی کرد؛ از جمله `std::ranges` و Viewهای استاندارد.

برای مثال، می‌توان با استفاده از `std::views::reverse` یک Range را به‌صورت معکوس مشاهده کرد، بدون اینکه لازم باشد مرزهای Reverse Iterator را به‌صورت دستی ایجاد کنیم.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    for (int value : values | std::views::reverse) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
30 20 10
```

این قابلیت به ++C20 تعلق دارد و نباید آن را با `std::make_reverse_iterator` که از ++C14 در دسترس است، اشتباه گرفت.

برای پروژه‌هایی که بر اساس ++C14 یا ++C17 ساخته شده‌اند، استفاده از `std::make_reverse_iterator` همچنان راهکاری استاندارد و مناسب برای معکوس کردن محدوده‌های مبتنی بر Iterator است.

## خطاهای رایج در استفاده از Reverse Iterator

### اشتباه گرفتن `base()` با Iterator عنصر فعلی

یکی از رایج‌ترین خطاها این است که تصور شود `base()` مستقیماً به عنصر قابل دسترسی از طریق Reverse Iterator اشاره می‌کند.

در واقع، `base()` موقعیت مرزی متناظر را برمی‌گرداند. برای دسترسی به عنصر فعلی، معمولاً باید از Iterator پایه یک گام به عقب رفت؛ البته فقط زمانی که این حرکت معتبر باشد.

### استفاده از مرزهای ناهماهنگ

اگر Reverse Iterator آغاز و پایان از مرزهای اشتباه ساخته شوند، ممکن است محدوده‌ای متفاوت از محدوده موردنظر پیمایش شود.

برای معکوس کردن یک محدوده عادی `[first, last)`، مرزهای معکوس متناظر به ترتیب زیر هستند:

```
auto reverseFirst = std::make_reverse_iterator(last);
auto reverseLast = std::make_reverse_iterator(first);
```

ترتیب مرزها باید برعکس شود؛ زیرا پیمایش از انتهای محدوده عادی شروع می‌شود و به ابتدای آن می‌رسد.

### فرض کردن قابلیت Random Access

وجود Reverse Iterator به معنی پشتیبانی از عملیات حسابی دلخواه نیست.

عملیات‌هایی مانند `r + n` یا `r - n` تنها زمانی در دسترس‌اند که Iterator پایه قابلیت‌های لازم را داشته باشد.

بنابراین، در الگوریتم‌های Generic نباید بدون بررسی الزامات نوع Iterator، عملیات مختص Random Access را استفاده کرد.

### تغییر ساختار Container در حین پیمایش

اگر در طول پیمایش، عملیاتی انجام شود که Iterator پایه را نامعتبر کند، Reverse Iterator نیز دیگر قابل استفاده نخواهد بود.

برای جلوگیری از این مشکل، باید قواعد اعتبار Iterator مربوط به Container را رعایت کرد و در صورت لزوم، Iteratorهای جدید را پس از عملیات تغییر ساختار به دست آورد.

## راهنمای انتخاب روش مناسب

| نیاز برنامه                                | روش پیشنهادی                                    |
| ------------------------------------------ | ----------------------------------------------- |
| پیمایش کامل یک Container از انتها به ابتدا | `rbegin()` و `rend()`                           |
| معکوس کردن یک محدوده مبتنی بر Iterator     | `std::make_reverse_iterator`                    |
| ساخت Reverse Iterator در ++C11             | سازنده مستقیم `std::reverse_iterator`           |
| پیدا کردن آخرین رخداد یک مقدار             | استفاده از الگوریتم جست‌وجو با Reverse Iterator |
| کار با Rangeهای استاندارد در ++C20         | بررسی `std::views::reverse`                     |
| پیمایش معکوس روی Iterator یک‌طرفه          | انتخاب الگوریتم یا ساختار داده جایگزین          |

این جدول یک راهنمای عملی است؛ انتخاب نهایی باید با توجه به نسخه استاندارد پروژه، قابلیت Iteratorها، هزینه حافظه و نیازهای الگوریتم انجام شود.

## جمع‌بندی نهایی

تابع `std::make_reverse_iterator` یکی از ابزارهای مفید کتابخانه استاندارد ++C14 است که ساخت Reverse Iterator را ساده می‌کند و امکان استفاده از الگوریتم‌های استاندارد در جهت پیمایش معکوس را فراهم می‌سازد.

مهم‌ترین نکاتی که هنگام استفاده از این قابلیت باید به خاطر داشت عبارت‌اند از:

- تابع در Header استاندارد `<iterator>` تعریف شده است.
- Iterator پایه باید قابلیت پیمایش دوطرفه یا قوی‌تر داشته باشد.
- برای پیمایش کامل یک Container، معمولاً `rbegin()` و `rend()` ساده‌تر هستند.
- برای معکوس کردن یک محدوده دلخواه، می‌توان از `std::make_reverse_iterator` استفاده کرد.
- تابع عضو `base()` موقعیت Iterator پایه را برمی‌گرداند، نه Iterator مستقیم عنصر قابل مشاهده.
- اعتبار Reverse Iterator به اعتبار Iterator پایه و طول عمر Container وابسته است.
- در ++C20، قابلیت‌هایی مانند `std::views::reverse` گزینه‌های دیگری برای پیمایش معکوس Rangeها ارائه می‌کنند.

در مجموع، درک رابطه میان Reverse Iterator و Iterator پایه، کلید استفاده ایمن و صحیح از `std::make_reverse_iterator` است. این درک به برنامه‌نویس اجازه می‌دهد بدون تغییر ساختار یا ترتیب واقعی عناصر، الگوریتم‌های موجود را برای پیمایش معکوس محدوده‌ها به کار بگیرد و کدهایی عمومی‌تر، خواناتر و قابل نگهداری‌تر بنویسد.

---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>