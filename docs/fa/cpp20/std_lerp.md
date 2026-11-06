<div align="center">

[🇺🇸 English](../../en/cpp20/std_lerp.md) | [🇮🇷 فارسی](./std_lerp.md)

</div>

---

# معرفی تابع `std::lerp` در ++C20

## مقدمه

محاسبه مقدار بین دو نقطه یکی از عملیات‌های رایج در برنامه‌نویسی است. این عملیات در حوزه‌های مختلفی مانند Computer Graphics، Game Development، انیمیشن، پردازش سیگنال، شبیه‌سازی‌های فیزیکی، محاسبات عددی و رابط‌های کاربری کاربرد دارد.

یکی از روش‌های متداول برای محاسبه مقدار بین دو مقدار، تکنیکی به نام Linear Interpolation یا به اختصار `Lerp` است.

در نسخه ++C20، کتابخانه استاندارد تابع `std::lerp` را به‌عنوان یک ابزار استاندارد برای انجام Linear Interpolation معرفی کرد. این تابع در Header فایل `<cmath>` قرار دارد و علاوه بر محاسبه مقدار درون یک بازه، ویژگی‌های عددی مهمی مانند Monotonicity و جلوگیری از برخی مشکلات Overflow را در شرایط مشخص تضمین می‌کند.

پیش از معرفی این تابع، توسعه‌دهندگان معمولاً از فرمول‌های دستی برای Interpolation استفاده می‌کردند. این فرمول‌ها در بسیاری از شرایط ساده به‌درستی عمل می‌کنند، اما ممکن است در محاسبات Floating-Point با مشکلاتی مانند Overflow، از دست رفتن دقت و نقض Monotonicity مواجه شوند.

تابع `std::lerp` با در نظر گرفتن این ملاحظات طراحی شده است تا یک روش استاندارد، قابل‌استفاده مجدد و از نظر عددی قابل‌اعتمادتر برای محاسبه Linear Interpolation فراهم کند.

## مفهوم Linear Interpolation

### تعریف ریاضی

روش Linear Interpolation برای محاسبه مقداری بین دو مقدار مشخص استفاده می‌شود. فرض کنید دو مقدار `a` و `b` داشته باشیم و بخواهیم مقدار متناظر با موقعیت نسبی `t` را محاسبه کنیم.

فرمول ریاضی این عملیات به شکل زیر است:

[\
\operatorname{lerp}(a,b,t)=a+t(b-a)\
]

در این فرمول:

- `a`: مقدار شروع یا Start Value است.
- `b`: مقدار پایان یا End Value است.
- `t`: پارامتر Interpolation است که موقعیت نسبی را مشخص می‌کند.
- نتیجه: مقدار محاسبه‌شده بین دو مقدار یا خارج از محدوده آن‌ها است.

مهم‌ترین حالت‌های این فرمول عبارت‌اند از:

| مقدار `t` | نتیجه                      | مفهوم         |
| --------- | -------------------------- | ------------- |
| `0`       | `a`                        | مقدار شروع    |
| `0.25`    | 25 درصد مسیر از `a` به `b` | یک‌چهارم مسیر |
| `0.5`     | نقطه میانی `a` و `b`       | نصف مسیر      |
| `0.75`    | 75 درصد مسیر از `a` به `b` | سه‌چهارم مسیر |
| `1`       | `b`                        | مقدار پایان   |

برای مثال، اگر مقدار شروع برابر با `10`، مقدار پایان برابر با `30` و پارامتر Interpolation برابر با `0.5` باشد، نتیجه برابر با `20` خواهد بود.

[\
10+0.5(30-10)=20\
]

### مفهوم پارامتر Interpolation

پارامتر `t` مشخص می‌کند که چه نسبتی از فاصله بین دو مقدار باید طی شود.

نکته مهم این است که `t` الزاماً نباید بین صفر و یک باشد.

اگر `t` در بازه `[0, 1]` قرار داشته باشد، عملیات Interpolation داخل محدوده دو مقدار انجام می‌شود.

اما اگر `t` خارج از این بازه باشد، عملیات به حالتی به نام Extrapolation تبدیل می‌شود.

برای مثال:

[\
\operatorname{lerp}(10,30,1.5)=40\
]

در این مثال، محاسبه از نقطه پایان نیز فراتر می‌رود.

همچنین:

[\
\operatorname{lerp}(10,30,-0.5)=0\
]

در این حالت، نتیجه قبل از نقطه شروع قرار می‌گیرد.

بنابراین، Interpolation و Extrapolation از نظر فرمول ریاضی یکسان هستند و تفاوت آن‌ها در محدوده پارامتر `t` است.

## معرفی std::lerp

### محل تعریف و استاندارد موردنیاز

تابع `std::lerp` از ویژگی‌های Standard Library در استاندارد ++C20 است.

برای استفاده از این تابع باید Header زیر را اضافه کنید:

```
#include <cmath>
```

امضای اصلی تابع به شکل زیر است:

```
namespace std {

    template<class T>
    constexpr T lerp(T a, T b, T t) noexcept;

}
```

ویژگی‌های اصلی این تابع عبارت‌اند از:

- **استاندارد:** از ++C20 به بعد.
- **Header:** فایل `<cmath>`.
- **Namespace:** فضای نام `std`.
- **نوع تابع:** Function Template.
- **نوع خروجی:** همان نوع `T`.
- **نوع پارامترها:** سه مقدار از نوع یکسان `T`.
- **ویژگی constexpr:** امکان استفاده در Constant Expression در شرایط مجاز.
- **ویژگی noexcept:** تابع تضمین می‌کند که Exception پرتاب نمی‌کند.

نوع `T` باید یک Floating-Point Type باشد؛ بنابراین انواعی مانند `float`، `double` و `long double` برای استفاده از این تابع مناسب هستند.

### ساده‌ترین مثال

در مثال زیر از `std::lerp` برای محاسبه مقدار میانی دو عدد استفاده می‌کنیم:

```
#include <cmath>
#include <iostream>

int main() {
    double start = 10.0;
    double end = 30.0;
    double t = 0.5;

    double result = std::lerp(start, end, t);

    std::cout << result << '\n';
}
```

خروجی:

```
20
```

در این مثال، تابع مقدار میانی بازه `10.0` تا `30.0` را محاسبه می‌کند.

### بررسی مقادیر مختلف

برای مشاهده رفتار تابع در موقعیت‌های مختلف می‌توان از مثال زیر استفاده کرد:

```
#include <cmath>
#include <iomanip>
#include <iostream>

int main() {
    constexpr double start = 10.0;
    constexpr double end = 30.0;

    for (int i = 0; i <= 10; ++i) {
        double t = i / 10.0;

        double value = std::lerp(start, end, t);

        std::cout << std::fixed
                  << std::setprecision(2)
                  << "t = " << t
                  << ", value = " << value
                  << '\n';
    }
}
```

خروجی:

```
t = 0.00, value = 10.00
t = 0.10, value = 12.00
t = 0.20, value = 14.00
t = 0.30, value = 16.00
t = 0.40, value = 18.00
t = 0.50, value = 20.00
t = 0.60, value = 22.00
t = 0.70, value = 24.00
t = 0.80, value = 26.00
t = 0.90, value = 28.00
t = 1.00, value = 30.00
```

در این مثال، پارامتر `t` به‌تدریج افزایش پیدا می‌کند و نتیجه نیز به‌صورت خطی از مقدار شروع به مقدار پایان حرکت می‌کند.

## تفاوت std::lerp با فرمول دستی

### پیاده‌سازی معمول Linear Interpolation

پیش از ++C20، توسعه‌دهندگان معمولاً Linear Interpolation را به‌صورت دستی پیاده‌سازی می‌کردند.

فرمول متداول به شکل زیر است:

```
double lerp(double a, double b, double t) {
    return a + t * (b - a);
}
```

این پیاده‌سازی از نظر ریاضی صحیح است، اما از نظر محاسبات Floating-Point همیشه بهترین انتخاب نیست.

### مشکل Overflow

یکی از مشکلات مهم فرمول معمولی، احتمال Overflow هنگام محاسبه اختلاف دو مقدار است.

برای مثال:

```
#include <iostream>
#include <limits>

int main() {
    const double max =
        std::numeric_limits<double>::max();

    double a = -max;
    double b = max;
    double t = 0.5;

    double result = a + t * (b - a);

    std::cout << result << '\n';
}
```

از نظر ریاضی، نتیجه باید صفر باشد.

اما عبارت زیر:

```
b - a
```

در این مثال از محدوده قابل نمایش نوع `double` بزرگ‌تر است و ممکن است به Infinity تبدیل شود.

در نتیجه، فرمول دستی نمی‌تواند نتیجه مورد انتظار را تضمین کند.

در مقابل، `std::lerp` با در نظر گرفتن محدودیت‌های Floating-Point طراحی شده است و برای ورودی‌های متناهی `a` و `b`، در صورتی که `t` بین صفر و یک باشد، نتیجه متناهی را تضمین می‌کند.

بنابراین، استفاده از `std::lerp` در این شرایط انتخاب مناسب‌تری است.

### مشکل Monotonicity

ویژگی Monotonicity به این معنا است که با افزایش پارامتر `t`، نتیجه نباید در جهت مخالف حرکت کند.

برای مثال، اگر `a` کوچک‌تر از `b` باشد، انتظار داریم با افزایش `t`، مقدار خروجی نیز کاهش پیدا نکند.

در پیاده‌سازی‌های دستی، خطاهای گرد کردن Floating-Point ممکن است باعث شوند این ویژگی در بعضی شرایط حفظ نشود.

تابع `std::lerp` دارای تضمین Monotonicity برای ورودی‌های متناهی و شرایط استاندارد مشخص‌شده است.

به‌طور دقیق، اگر `a` و `b` متناهی باشند و `t` در بازه `[0, 1]` قرار داشته باشد، نتیجه نسبت به `t` رفتار یکنواخت دارد:

- اگر `a < b` باشد، نتیجه نسبت به `t` غیرکاهشی است.
- اگر `a > b` باشد، نتیجه نسبت به `t` غیر‌افزایشی است.

این ویژگی برای Animation، محاسبات عددی و سیستم‌هایی که به حرکت پیوسته و قابل‌پیش‌بینی نیاز دارند اهمیت زیادی دارد.

### مقایسه رفتار دو روش

| ویژگی                       | فرمول دستی                            | `std::lerp`                     |
| --------------------------- | ------------------------------------- | ------------------------------- |
| محاسبه Linear Interpolation | بله                                   | بله                             |
| استاندارد بودن              | پیاده‌سازی شخصی                       | ++C20                           |
| تضمین Endpointها            | وابسته به پیاده‌سازی                  | بله                             |
| تضمین Monotonicity          | خیر                                   | بله، در شرایط استاندارد         |
| جلوگیری از Overflow         | وابسته به پیاده‌سازی                  | تضمین نتیجه متناهی در بازه مشخص |
| امکان استفاده در constexpr  | وابسته به نسخه استاندارد و پیاده‌سازی | بله                             |
| عدم پرتاب Exception         | وابسته به تابع                        | بله                             |

نکته مهم این است که `std::lerp` جایگزین تمام الگوریتم‌های Interpolation نیست؛ بلکه برای محاسبه Linear Interpolation بین مقادیر Floating-Point طراحی شده است.

## ویژگی‌های عددی و تضمین‌های استاندارد

### تضمین Endpointها

یکی از ویژگی‌های مهم `std::lerp` این است که برای پارامترهای صفر و یک، مقادیر ابتدا و انتهای بازه را دقیقاً برمی‌گرداند.

```
#include <cassert>
#include <cmath>

int main() {
    constexpr double a = 10.5;
    constexpr double b = 99.75;

    static_assert(std::lerp(a, b, 0.0) == a);
    static_assert(std::lerp(a, b, 1.0) == b);
}
```

این تضمین از اهمیت زیادی برخوردار است؛ زیرا در پیاده‌سازی‌های دستی ممکن است محاسبات Floating-Point باعث ایجاد اختلاف ناخواسته در Endpointها شوند.

برای مثال، در Animation ممکن است انتظار داشته باشیم که در پایان زمان مشخص‌شده، مقدار دقیقاً برابر با Target Value باشد.

### حفظ مقدار در صورت یکسان بودن دو ورودی

اگر مقدار شروع و پایان یکسان باشند، نتیجه نیز باید همان مقدار باشد:

[\
\operatorname{lerp}(a,a,t)=a\
]

برای مثال:

```
#include <cmath>
#include <iostream>

int main() {
    double value = std::lerp(42.0, 42.0, 0.75);

    std::cout << value << '\n';
}
```

خروجی:

```
42
```

این رفتار حتی زمانی که `t` خارج از بازه `[0, 1]` باشد نیز برقرار است.

### تضمین Finite Result

یکی از مهم‌ترین ویژگی‌های عددی این تابع، تضمین نتیجه متناهی در شرایط مشخص است.

اگر دو مقدار `a` و `b` متناهی باشند و پارامتر `t` در محدوده زیر قرار داشته باشد:

[\
0 \leq t \leq 1\
]

نتیجه `std::lerp` نیز متناهی خواهد بود.

این ویژگی در محاسباتی که با مقادیر بسیار بزرگ یا کوچک سروکار دارند اهمیت زیادی دارد.

البته این تضمین به معنای دقت نامحدود یا حذف تمام خطاهای Floating-Point نیست. نتیجه همچنان در قالب نوع Floating-Point مقصد محاسبه و گرد می‌شود.

### ویژگی Symmetry

از نظر ریاضی، رابطه زیر برقرار است:

\operatorname{lerp}(b,a,1-t)\
]

این رابطه نشان می‌دهد که حرکت از `a` به `b` با پارامتر `t` از نظر ریاضی با حرکت از `b` به `a` با پارامتر `1-t` معادل است.

برای مثال:

```
#include <cmath>
#include <iostream>

int main() {
    double a = 10.0;
    double b = 50.0;
    double t = 0.25;

    double forward = std::lerp(a, b, t);
    double backward = std::lerp(b, a, 1.0 - t);

    std::cout << forward << '\n';
    std::cout << backward << '\n';
}
```

خروجی:

```
20
20
```

توجه داشته باشید که این رابطه در ریاضیات دقیق برقرار است، اما به دلیل تفاوت مسیرهای محاسباتی و Rounding در Floating-Point، نباید در تمام ورودی‌ها انتظار برابری Bit-by-Bit داشت.

### تضمین Monotonicity در بازه Interpolation

یکی از ویژگی‌های مهم `std::lerp`، حفظ ترتیب حرکت بین دو Endpoint است.

برای مثال، کد زیر را در نظر بگیرید:

```
#include <cassert>
#include <cmath>

int main() {
    double a = 100.0;
    double b = 200.0;

    double previous = std::lerp(a, b, 0.0);

    for (int i = 1; i <= 100; ++i) {
        double t = i / 100.0;

        double current = std::lerp(a, b, t);

        assert(current >= previous);

        previous = current;
    }
}
```

در این مثال، با افزایش `t`، مقدار خروجی نباید کاهش پیدا کند.

اگر جهت حرکت برعکس باشد، یعنی `a` بزرگ‌تر از `b` باشد، رفتار Monotonicity نیز معکوس خواهد بود.

## نحوه استفاده از constexpr

### استفاده در Constant Expression

تابع `std::lerp` از ++C20 به‌صورت `constexpr` تعریف شده است.

این ویژگی امکان استفاده از آن را در محاسبات زمان کامپایل فراهم می‌کند.

```
#include <cmath>

constexpr double start = 10.0;
constexpr double end = 50.0;

constexpr double middle =
    std::lerp(start, end, 0.5);

static_assert(middle == 30.0);
```

در این مثال، محاسبه مقدار میانی می‌تواند هنگام Compile شدن برنامه انجام شود.

استفاده از `constexpr` برای محاسبات ثابت، جدول‌های Lookup و تنظیمات Compile-Time مفید است.

### استفاده در constexpr Function

همچنین می‌توان `std::lerp` را داخل یک تابع `constexpr` استفاده کرد:

```
#include <cmath>

constexpr double interpolate(
    double start,
    double end,
    double progress
) {
    return std::lerp(start, end, progress);
}

static_assert(interpolate(0.0, 100.0, 0.25) == 25.0);
static_assert(interpolate(0.0, 100.0, 0.75) == 75.0);
```

البته برای استفاده در Constant Expression، تمام ورودی‌ها و عملیات انجام‌شده باید شرایط مربوط به Constant Expression را رعایت کنند.

## بررسی انواع داده و Type Deduction

### پشتیبانی از انواع Floating-Point

تابع `std::lerp` برای انواع Floating-Point طراحی شده است.

نمونه‌های معتبر:

```
#include <cmath>

int main() {
    float f = std::lerp(0.0f, 100.0f, 0.5f);

    double d = std::lerp(0.0, 100.0, 0.5);

    long double ld = std::lerp(
        0.0L,
        100.0L,
        0.5L
    );
}
```

در هر سه مثال، نوع پارامترها یکسان است.

### مشکل ترکیب انواع مختلف

از آنجا که `std::lerp` یک Function Template است، نوع `T` از روی آرگومان‌ها استنتاج می‌شود.

بنابراین، ترکیب مستقیم انواع مختلف ممکن است باعث شکست Template Argument Deduction شود.

برای مثال:

```
#include <cmath>

int main() {
    double result = std::lerp(
        0.0f,
        100.0,
        0.5
    );
}
```

این کد به دلیل تفاوت نوع آرگومان‌ها در Template Argument Deduction معتبر نیست.

برای حل این مشکل، بهتر است تمام ورودی‌ها را به یک نوع مشترک تبدیل کنید:

```
#include <cmath>

int main() {
    double start = 0.0f;
    double end = 100.0;
    double progress = 0.5f;

    double result = std::lerp(
        start,
        end,
        progress
    );
}
```

روش دیگر استفاده از `static_cast` است:

```
#include <cmath>

int main() {
    float start = 10.0f;
    double end = 100.0;
    float progress = 0.5f;

    double result = std::lerp(
        static_cast<double>(start),
        end,
        static_cast<double>(progress)
    );
}
```

در برنامه‌های بزرگ‌تر، انتخاب یک Floating-Point Type مشخص برای کل محاسبه معمولاً خوانایی و پیش‌بینی‌پذیری بیشتری ایجاد می‌کند.

### استفاده از انواع صحیح

تابع `std::lerp` برای مقادیر Integer طراحی نشده است.

برای مثال:

```
#include <cmath>

int main() {
    auto result = std::lerp(10, 20, 0);
}
```

این استفاده معتبر نیست؛ زیرا نوع استنتاج‌شده `int` است و نوع `T` باید Floating-Point باشد.

برای انجام Interpolation روی مقادیر صحیح، باید ابتدا محاسبه را در یک Floating-Point Type انجام دهید و سپس در صورت نیاز نتیجه را به نوع Integer تبدیل کنید:

```
#include <cmath>
#include <iostream>

int main() {
    int start = 10;
    int end = 20;

    double progress = 0.5;

    double interpolated = std::lerp(
        static_cast<double>(start),
        static_cast<double>(end),
        progress
    );

    int result = static_cast<int>(interpolated);

    std::cout << result << '\n';
}
```

خروجی:

```
15
```

توجه داشته باشید که تبدیل Floating-Point به Integer می‌تواند بخش اعشاری را حذف کند. این تبدیل به‌صورت خودکار به نزدیک‌ترین عدد صحیح گرد نمی‌شود.

## کاربردهای عملی std::lerp

### کاربرد در Animation

یکی از رایج‌ترین کاربردهای Linear Interpolation، حرکت دادن یک Object از موقعیت شروع به موقعیت پایان است.

فرض کنید یک Object باید طی دو ثانیه از موقعیت صفر به موقعیت صد حرکت کند.

در این شرایط می‌توان از `std::lerp` برای محاسبه موقعیت فعلی استفاده کرد:

```
#include <cmath>
#include <iostream>

int main() {
    constexpr double startPosition = 0.0;
    constexpr double endPosition = 100.0;

    constexpr double duration = 2.0;

    double elapsedTime = 0.75;

    double progress = elapsedTime / duration;

    double currentPosition = std::lerp(
        startPosition,
        endPosition,
        progress
    );

    std::cout << currentPosition << '\n';
}
```

خروجی:

```
37.5
```

در این مثال، پس از گذشت 0.75 ثانیه از یک Animation دوثانیه‌ای، مقدار `progress` برابر با `0.375` خواهد بود.

بنابراین، Object در 37.5 درصد مسیر قرار می‌گیرد.

نکته مهم این است که در یک سیستم Animation واقعی، معمولاً باید `progress` را به بازه `[0, 1]` محدود کرد:

```
#include <algorithm>
#include <cmath>

double calculatePosition(
    double start,
    double end,
    double elapsed,
    double duration
) {
    double progress = elapsed / duration;

    progress = std::clamp(progress, 0.0, 1.0);

    return std::lerp(start, end, progress);
}
```

این پیاده‌سازی از Extrapolation ناخواسته پس از پایان Animation جلوگیری می‌کند.

همچنین باید توجه داشت که `duration` نباید صفر باشد و ورودی‌های زمانی باید مقادیر معتبر داشته باشند.

### کاربرد در Game Development

در بازی‌ها، بسیاری از ویژگی‌های Objectها به‌صورت تدریجی تغییر می‌کنند.

برای مثال:

- موقعیت یک Character
- سرعت حرکت یک Object
- مقدار Health یا Energy
- شدت نور
- میزان Transparency
- مقدار Zoom دوربین
- حجم صدای یک منبع صوتی

برای محاسبه این مقادیر می‌توان از `std::lerp` استفاده کرد.

نمونه زیر شدت نور را بین دو مقدار تغییر می‌دهد:

```
#include <cmath>
#include <iostream>

int main() {
    double minimumIntensity = 0.2;
    double maximumIntensity = 1.0;

    double transitionProgress = 0.65;

    double intensity = std::lerp(
        minimumIntensity,
        maximumIntensity,
        transitionProgress
    );

    std::cout << intensity << '\n';
}
```

خروجی:

```
0.72
```

این روش برای پیاده‌سازی تغییرات خطی در سیستم‌های Real-Time مناسب است.

### کاربرد در تغییر رنگ

در سیستم‌های Graphics، تغییر تدریجی رنگ یکی از کاربردهای رایج Interpolation است.

برای مثال، فرض کنید بخواهیم رنگی را از قرمز به آبی تغییر دهیم.

هر رنگ RGB از سه Component تشکیل شده است:

- Red
- Green
- Blue

می‌توان هر Component را به‌صورت مستقل Interpolate کرد:

```
#include <cmath>
#include <iostream>

struct Color {
    double r;
    double g;
    double b;
};

Color interpolateColor(
    const Color& start,
    const Color& end,
    double t
) {
    return {
        std::lerp(start.r, end.r, t),
        std::lerp(start.g, end.g, t),
        std::lerp(start.b, end.b, t)
    };
}

int main() {
    Color red{1.0, 0.0, 0.0};
    Color blue{0.0, 0.0, 1.0};

    Color result = interpolateColor(red, blue, 0.5);

    std::cout << result.r << ' '
              << result.g << ' '
              << result.b << '\n';
}
```

خروجی:

```
0.5 0 0.5
```

در این مثال، رنگ حاصل از Interpolation خطی Componentهای RGB به دست می‌آید.

البته در کاربردهای حرفه‌ای Graphics، باید به Color Space نیز توجه کرد. Interpolation مستقیم در فضای RGB لزوماً از نظر ادراکی یا فیزیکی بهترین نتیجه را تولید نمی‌کند.

در برخی کاربردها، انجام Interpolation در Linear RGB یا فضاهای رنگی دیگر مناسب‌تر است.

### کاربرد در Camera Zoom

در سیستم‌های Camera، می‌توان مقدار Zoom را به‌صورت تدریجی تغییر داد.

```
#include <cmath>
#include <iostream>

int main() {
    double initialZoom = 1.0;
    double targetZoom = 2.5;

    double progress = 0.4;

    double currentZoom = std::lerp(
        initialZoom,
        targetZoom,
        progress
    );

    std::cout << currentZoom << '\n';
}
```

خروجی:

```
1.6
```

این روش برای پیاده‌سازی تغییر خطی Zoom در یک بازه زمانی مناسب است.

### کاربرد در پردازش سیگنال

در Digital Signal Processing، Interpolation می‌تواند برای تخمین مقدار سیگنال بین دو Sample استفاده شود.

فرض کنید دو Sample متوالی داشته باشیم:

[\
x_0=2.0\
]

[\
x_1=6.0\
]

برای تخمین مقدار در نقطه‌ای با موقعیت نسبی `0.25` می‌توان از رابطه زیر استفاده کرد:

[\
x=\operatorname{lerp}(x_0,x_1,0.25)\
]

نتیجه برابر با `3.0` خواهد بود.

این روش نمونه‌ای از Linear Interpolation در پردازش سیگنال است.

البته در سیستم‌هایی که به کیفیت بالاتر نیاز دارند، ممکن است روش‌هایی مانند Cubic Interpolation یا Spline Interpolation انتخاب مناسب‌تری باشند.

## تفاوت Interpolation و Easing

### مفهوم Linear Interpolation

تابع `std::lerp` به‌صورت خطی بین دو مقدار حرکت می‌کند.

به این معنا که اگر `t` با سرعت ثابت افزایش پیدا کند، مقدار خروجی نیز با سرعت ثابت تغییر خواهد کرد.

برای مثال:

```
#include <cmath>

double value = std::lerp(0.0, 100.0, 0.5);
```

در این مثال، مقدار خروجی دقیقاً در نصف مسیر قرار دارد.

### مفهوم Easing

در Animation، گاهی حرکت خطی نتیجه طبیعی و مطلوبی ایجاد نمی‌کند.

برای مثال، ممکن است بخواهیم:

- حرکت در ابتدا آهسته باشد.
- در میانه سریع‌تر شود.
- در انتها دوباره آهسته شود.

این رفتار با استفاده از Easing Functions ایجاد می‌شود.

نکته مهم این است که `std::lerp` به‌تنهایی Easing انجام نمی‌دهد؛ اما می‌توان نتیجه یک Easing Function را به‌عنوان پارامتر `t` به آن ارسال کرد.

برای مثال:

```
#include <cmath>

double easeInOut(double t) {
    return t * t * (3.0 - 2.0 * t);
}

int main() {
    double start = 0.0;
    double end = 100.0;

    double progress = 0.5;

    double easedProgress = easeInOut(progress);

    double value = std::lerp(
        start,
        end,
        easedProgress
    );
}
```

در این مثال، ابتدا مقدار `progress` با یک Easing Function تغییر می‌کند و سپس مقدار نهایی محاسبه می‌شود.

بنابراین، این دو مفهوم مسئولیت‌های متفاوتی دارند:

| ویژگی         | `std::lerp`                  | Easing Function                             |
| ------------- | ---------------------------- | ------------------------------------------- |
| هدف           | محاسبه مقدار بین دو Endpoint | تغییر سرعت پیشرفت                           |
| رفتار پیش‌فرض | خطی                          | وابسته به تابع                              |
| کاربرد        | Interpolation                | ایجاد حرکت طبیعی‌تر                         |
| ترکیب‌پذیری   | قابل ترکیب با Easing         | قابل استفاده به‌عنوان پارامتر Interpolation |

## تفاوت std::lerp و std::midpoint

در استاندارد ++C20 دو تابع مرتبط با محاسبات عددی به نام‌های `std::lerp` و `std::midpoint` در کتابخانه استاندارد وجود دارند.

با وجود ارتباط مفهومی این دو تابع، کاربرد آن‌ها متفاوت است.

### معرفی std::midpoint

تابع `std::midpoint` برای محاسبه نقطه میانی دو مقدار استفاده می‌شود.

این تابع نیز در Header فایل `<numeric>` تعریف شده است.

نمونه:

```
#include <iostream>
#include <numeric>

int main() {
    double a = 10.0;
    double b = 30.0;

    double middle = std::midpoint(a, b);

    std::cout << middle << '\n';
}
```

خروجی:

```
20
```

### تفاوت اصلی دو تابع

تابع `std::midpoint` همیشه نقطه میانی دو مقدار را محاسبه می‌کند، اما `std::lerp` امکان تعیین موقعیت دلخواه با پارامتر `t` را فراهم می‌کند.

از نظر ریاضی، رابطه زیر برقرار است:

\operatorname{lerp}(a,b,0.5)\
]

برای مقادیر Floating-Point، هر دو تابع می‌توانند برای محاسبه نقطه میانی استفاده شوند؛ با این حال، تضمین‌های استاندارد و جزئیات رفتار عددی آن‌ها یکسان نیست.

یکی از تفاوت‌های مهم این است که `std::midpoint` علاوه بر Floating-Point، از انواع Arithmetic دیگری مانند Integer نیز پشتیبانی می‌کند.

برای مقادیر صحیح، `std::midpoint` در حالت نتیجه غیرصحیح، به سمت آرگومان اول گرد می‌کند.

بنابراین:

- برای محاسبه نقطه میانی دو مقدار از `std::midpoint` استفاده کنید.
- برای محاسبه مقدار در موقعیت نسبی دلخواه از `std::lerp` استفاده کنید.

## تفاوت Linear Interpolation و سایر روش‌ها

### مقایسه با Cubic Interpolation

در Linear Interpolation، مقدار خروجی با استفاده از یک رابطه خطی بین دو Endpoint محاسبه می‌شود.

اما در Cubic Interpolation، معمولاً از یک تابع درجه سه برای محاسبه مقدار استفاده می‌شود.

این روش می‌تواند برای ایجاد تغییرات نرم‌تر و کنترل بیشتر روی شیب منحنی مناسب باشد.

کاربردهای رایج آن عبارت‌اند از:

- پردازش تصویر
- Animation
- پردازش سیگنال
- محاسبات هندسی

در مقایسه با Linear Interpolation، پیاده‌سازی Cubic Interpolation معمولاً پیچیده‌تر است و ممکن است به اطلاعات بیشتری مانند نقاط همسایه یا Tangentها نیاز داشته باشد.

### مقایسه با Spline Interpolation

روش Spline Interpolation برای ایجاد منحنی‌های نرم با استفاده از چندین نقطه کنترلی استفاده می‌شود.

این روش در مواردی مناسب است که حرکت یا تغییر مقدار نباید صرفاً در یک خط مستقیم انجام شود.

برای مثال، در مسیر حرکت Camera یا طراحی Animationهای پیچیده، استفاده از Spline می‌تواند کنترل بیشتری فراهم کند.

تابع `std::lerp` جایگزین Spline نیست، اما می‌تواند به‌عنوان بخشی از پیاده‌سازی الگوریتم‌های پیچیده‌تر مورد استفاده قرار گیرد.

## مدیریت Extrapolation

### رفتار خارج از محدوده صفر تا یک

یکی از نکات مهم درباره `std::lerp` این است که تابع به‌صورت پیش‌فرض مقدار `t` را به بازه `[0, 1]` محدود نمی‌کند.

بنابراین، ورودی‌های بزرگ‌تر از یک یا کوچک‌تر از صفر باعث Extrapolation می‌شوند.

```
#include <cmath>
#include <iostream>

int main() {
    double a = 10.0;
    double b = 20.0;

    std::cout << std::lerp(a, b, -1.0) << '\n';
    std::cout << std::lerp(a, b, 0.5) << '\n';
    std::cout << std::lerp(a, b, 2.0) << '\n';
}
```

خروجی:

```
0
15
30
```

این رفتار برای کاربردهایی مانند پیش‌بینی خطی و محاسباتی که به ادامه مسیر خارج از بازه نیاز دارند مفید است.

### محدود کردن مقدار t با std::clamp

اگر هدف فقط Interpolation داخل بازه باشد، می‌توان از `std::clamp` استفاده کرد.

این تابع از ++C17 در Header فایل `<algorithm>` در دسترس است.

```
#include <algorithm>
#include <cmath>
#include <iostream>

int main() {
    double a = 10.0;
    double b = 20.0;

    double t = 1.5;

    t = std::clamp(t, 0.0, 1.0);

    double result = std::lerp(a, b, t);

    std::cout << result << '\n';
}
```

خروجی:

```
20
```

در این مثال، مقدار اولیه `t` برابر با `1.5` است، اما پس از محدودسازی به `1.0` تبدیل می‌شود.

در نتیجه، تابع مقدار انتهایی را برمی‌گرداند.

توجه داشته باشید که `std::clamp` برای استفاده معمولی به مقادیر قابل‌مقایسه نیاز دارد. در صورت احتمال وجود NaN در ورودی، باید سیاست مدیریت آن را به‌صورت جداگانه مشخص کنید.

## مدیریت مقادیر NaN و Infinity

### رفتار با NaN

مقدار NaN یا Not a Number یکی از مقادیر ویژه Floating-Point است.

این مقدار معمولاً در نتیجه عملیات نامعتبر ریاضی یا دریافت داده نامعتبر ایجاد می‌شود.

برای مثال:

```
#include <cmath>
#include <limits>

int main() {
    double a = 10.0;
    double b = 20.0;

    double t = std::numeric_limits<double>::quiet_NaN();

    double result = std::lerp(a, b, t);
}
```

در چنین شرایطی، نباید انتظار داشت که نتیجه یک مقدار عددی معتبر و قابل‌استفاده برای منطق برنامه باشد.

رفتار مربوط به NaN را نباید با تضمین‌های Monotonicity یا Finite Result اشتباه گرفت؛ زیرا این تضمین‌ها برای ورودی‌های نامعتبر یا غیرمتناهی به همان شکل برقرار نیستند.

اگر ورودی‌ها از شبکه، فایل یا منابع خارجی دریافت می‌شوند، بهتر است قبل از انجام محاسبات اعتبارسنجی شوند.

برای این کار می‌توان از `std::isfinite` و `std::isnan` استفاده کرد:

```
#include <cmath>
#include <stdexcept>

double safeLerp(double a, double b, double t) {
    if (!std::isfinite(a) ||
        !std::isfinite(b) ||
        !std::isfinite(t)) {
        throw std::invalid_argument("Invalid interpolation input");
    }

    return std::lerp(a, b, t);
}
```

این پیاده‌سازی یک سیاست مشخص برای جلوگیری از ورود مقادیر غیرمتناهی و NaN به محاسبات ایجاد می‌کند.

### رفتار با Infinity

مقادیر Infinity نیز باید با دقت مدیریت شوند.

برای مثال:

```
#include <cmath>
#include <limits>

int main() {
    double infinity =
        std::numeric_limits<double>::infinity();

    double result = std::lerp(
        10.0,
        infinity,
        0.5
    );
}
```

در این شرایط، تضمین نتیجه متناهی که برای Endpointهای متناهی و `t` داخل بازه `[0, 1]` وجود دارد، قابل اعمال نیست.

بنابراین، اگر سیستم شما به نتیجه متناهی نیاز دارد، باید Endpointها را پیش از فراخوانی تابع اعتبارسنجی کنید.

## نکات مهم در محاسبات Floating-Point

### محدودیت دقت

با وجود تضمین‌های عددی `std::lerp`، این تابع همچنان با محدودیت‌های Floating-Point مواجه است.

انواعی مانند `float` و `double` فقط تعداد محدودی بیت برای نمایش مقدار دارند.

در نتیجه، بعضی مقادیر اعشاری به‌صورت دقیق قابل نمایش نیستند.

برای مثال:

```
#include <cmath>
#include <iostream>

int main() {
    double result = std::lerp(
        0.1,
        0.2,
        0.3
    );

    std::cout.precision(17);

    std::cout << result << '\n';
}
```

خروجی ممکن است به‌صورت زیر نمایش داده شود:

```
0.13
```

البته نمایش دقیق وابسته به مقدار واقعی ذخیره‌شده و نحوه فرمت کردن خروجی است.

در تست‌های Floating-Point بهتر است در صورت نیاز از Tolerance استفاده کنید:

```
#include <cmath>
#include <cassert>

int main() {
    double result = std::lerp(0.1, 0.2, 0.3);

    double expected = 0.13;

    double epsilon = 1e-12;

    assert(std::abs(result - expected) < epsilon);
}
```

توجه داشته باشید که انتخاب Tolerance باید متناسب با مقیاس مقادیر، نوع Floating-Point و نیازهای برنامه انجام شود.

### انتخاب بین float و double

انتخاب نوع داده به نیازهای برنامه بستگی دارد.

| نوع           | ویژگی                    | کاربرد مناسب                        |
| ------------- | ------------------------ | ----------------------------------- |
| `float`       | حافظه کمتر و دقت محدودتر | Graphics و برخی سیستم‌های Real-Time |
| `double`      | دقت بیشتر                | محاسبات عمومی و عددی                |
| `long double` | دقت وابسته به پیاده‌سازی | محاسباتی که به دقت بیشتر نیاز دارند |

در بسیاری از برنامه‌های عمومی، `double` انتخاب مناسبی است.

با این حال، در موتورهای بازی یا پردازش‌های حجیم ممکن است `float` به دلیل سازگاری با سایر APIها و کاهش مصرف حافظه مناسب‌تر باشد.

نکته مهم این است که `long double` الزاماً در تمام پلتفرم‌ها دقت بیشتری نسبت به `double` ندارد.

### جلوگیری از تقسیم بر صفر در محاسبه t

در کاربردهای واقعی، مقدار `t` معمولاً از تقسیم زمان سپری‌شده بر مدت زمان کل محاسبه می‌شود.

فرمول معمول به شکل زیر است:

[\
t=\frac{elapsedTime}{duration}\
]

اگر `duration` برابر صفر باشد، محاسبه می‌تواند نتیجه نامعتبر ایجاد کند.

بنابراین، پیش از محاسبه `t` باید مقدار Duration بررسی شود.

```
#include <algorithm>
#include <cmath>
#include <stdexcept>

double animate(
    double start,
    double end,
    double elapsedTime,
    double duration
) {
    if (!std::isfinite(duration) || duration <= 0.0) {
        throw std::invalid_argument("Invalid duration");
    }

    double progress = elapsedTime / duration;

    if (!std::isfinite(progress)) {
        throw std::invalid_argument("Invalid progress");
    }

    progress = std::clamp(progress, 0.0, 1.0);

    return std::lerp(start, end, progress);
}
```

این پیاده‌سازی علاوه بر کنترل Duration، از ورود Progress نامعتبر به محاسبه نیز جلوگیری می‌کند.

## اشتباهات رایج هنگام استفاده

### استفاده از Integer به‌جای Floating-Point

یکی از اشتباهات رایج، استفاده مستقیم از Integer برای پارامترهای `std::lerp` است.

```
std::lerp(10, 20, 0);
```

این کد به دلیل استنتاج نوع `int` برای Template معتبر نیست.

بهتر است از Literalهای Floating-Point استفاده شود:

```
std::lerp(10.0, 20.0, 0.5);
```

### انتظار محدود شدن خودکار t

بعضی توسعه‌دهندگان تصور می‌کنند `std::lerp` مقدار `t` را به بازه `[0, 1]` محدود می‌کند.

این تصور اشتباه است.

این تابع Extrapolation را نیز پشتیبانی می‌کند.

اگر محدودسازی لازم باشد، باید از `std::clamp` استفاده کنید.

### تصور حذف کامل خطای Floating-Point

تابع `std::lerp` برای بهبود رفتار عددی طراحی شده است، اما محاسبات آن همچنان تابع محدودیت‌های Floating-Point هستند.

بنابراین، نباید انتظار دقت ریاضی نامحدود داشت.

### استفاده برای Interpolation غیرخطی

تابع `std::lerp` برای Linear Interpolation طراحی شده است.

اگر هدف ایجاد حرکت Ease-In، Ease-Out، منحنی‌های Bézier یا Spline باشد، باید از روش مناسب همان مسئله استفاده کنید.

### استفاده بدون اعتبارسنجی ورودی‌ها

در برنامه‌هایی که ورودی‌ها از منابع خارجی دریافت می‌شوند، نباید فرض کرد که تمام مقادیر معتبر هستند.

به‌خصوص در سیستم‌های Real-Time، Graphics و Network Programming بهتر است وضعیت NaN، Infinity و مقادیر خارج از محدوده مشخص باشد.

## ملاحظات عملکردی و طراحی

### هزینه محاسباتی

تابع `std::lerp` یک عملیات عددی سبک است و معمولاً برای استفاده در حلقه‌های پردازشی و Animation مناسب است.

با این حال، استاندارد ++C هیچ تضمین مستقیمی درباره تعداد Instructionهای ماشین یا سرعت اجرای تابع ارائه نمی‌کند.

Compiler می‌تواند با توجه به Optimization، معماری پردازنده و نوع Floating-Point از روش‌های مختلفی برای تولید کد استفاده کند.

بنابراین، اگر عملکرد بخش Interpolation برای برنامه اهمیت زیادی دارد، باید آن را در محیط واقعی با Benchmark ارزیابی کرد.

### استفاده در حلقه‌های پردازشی

در برنامه‌هایی مانند Game Engine، ممکن است در هر Frame هزاران بار Interpolation انجام شود.

نمونه:

```
#include <cmath>
#include <vector>

int main() {
    std::vector<double> positions(1000);

    for (std::size_t i = 0; i < positions.size(); ++i) {
        double t = static_cast<double>(i) /
                   static_cast<double>(positions.size() - 1);

        positions[i] = std::lerp(
            0.0,
            100.0,
            t
        );
    }
}
```

در این مثال، برای هر عنصر یک مقدار بین صفر و صد محاسبه می‌شود.

در کاربرد واقعی، باید اندازه Container و نحوه تولید `t` نیز بررسی شود. برای مثال، اگر تعداد عناصر صفر یا یک باشد، محاسبه Progress نیازمند مدیریت جداگانه است.

### توجه به SIMD

تابع `std::lerp` به‌خودی‌خود یک SIMD API نیست.

اگرچه Compiler ممکن است برخی محاسبات را Vectorize کند، اما نباید فرض کرد که فراخوانی این تابع همیشه به دستورهای SIMD تبدیل می‌شود.

برای پردازش برداری صریح می‌توان از قابلیت‌های مناسب Compiler، Intrinsics یا کتابخانه‌های SIMD استفاده کرد.

انتخاب روش مناسب باید بر اساس نیاز عملکردی و قابلیت حمل کد انجام شود.

### ملاحظات Determinism

در سیستم‌هایی که به Deterministic Computation نیاز دارند، مانند برخی شبیه‌سازی‌ها و Multiplayer Game Engineها، باید به تفاوت‌های Floating-Point بین معماری‌ها و تنظیمات Compiler توجه کرد.

استفاده از `std::lerp` به‌تنهایی تضمین نمی‌کند که نتیجه محاسبات روی تمام پردازنده‌ها و Compilerها به‌صورت Bitwise Identical باشد.

اگر Determinism دقیق ضروری است، باید سیاست محاسبات عددی، نوع داده، تنظیمات Floating-Point و روش‌های تست به‌صورت یکپارچه طراحی شوند.

## تفاوت نسخه‌های استاندارد ++C

### وضعیت در ++C17 و نسخه‌های قدیمی‌تر

تابع `std::lerp` در ++C17 و نسخه‌های قدیمی‌تر وجود ندارد.

در این نسخه‌ها، توسعه‌دهندگان معمولاً باید از پیاده‌سازی شخصی یا کتابخانه‌های جانبی استفاده کنند.

نمونه:

```
double lerp(double a, double b, double t) {
    return a + t * (b - a);
}
```

این پیاده‌سازی برای کاربردهای ساده مناسب است، اما تضمین‌های عددی `std::lerp` را ندارد.

### معرفی در ++C20

در ++C20، تابع `std::lerp` به Standard Library اضافه شد.

مهم‌ترین ویژگی‌های آن عبارت‌اند از:

- ارائه API استاندارد برای Linear Interpolation.
- پشتیبانی از `constexpr`.
- تضمین `noexcept`.
- حفظ دقیق Endpointها.
- تضمین Monotonicity در شرایط مشخص.
- تضمین نتیجه متناهی برای Endpointهای متناهی و `t` در بازه `[0, 1]`.

### وضعیت در ++C23 و ++C26

تابع `std::lerp` از ++C20 در Standard Library وجود دارد و همچنان بخشی از استانداردهای جدیدتر است.

برای استفاده معمول از این تابع، نیازی به قابلیت اختصاصی ++C23 یا ++C26 وجود ندارد.

بنابراین، پروژه‌ای که از ++C20 پشتیبانی می‌کند می‌تواند از این API استفاده کند.

## تنظیم Compiler برای استفاده از std::lerp

### کامپایل با GCC

برای استفاده از قابلیت‌های ++C20 در GCC می‌توانید از دستور زیر استفاده کنید:

```
g++ -std=c++20 -Wall -Wextra -Wpedantic main.cpp -o main
```

### کامپایل با Clang

برای Clang نیز دستور زیر قابل استفاده است:

```
clang++ -std=c++20 -Wall -Wextra -Wpedantic main.cpp -o main
```

### کامپایل با MSVC

در Visual Studio می‌توانید استاندارد زبان را روی C++20 تنظیم کنید.

همچنین از گزینه زیر در خط فرمان MSVC استفاده می‌شود:

```
cl /std:c++20 /EHsc /W4 main.cpp
```

نکته مهم این است که پشتیبانی از یک استاندارد زبان و پشتیبانی از قابلیت‌های Standard Library ممکن است به نسخه Compiler و کتابخانه استاندارد مورد استفاده وابسته باشد.

در صورت بروز خطا، باید نسخه Compiler و وضعیت پشتیبانی Standard Library بررسی شود.

## مثال جامع کاربردی

### پیاده‌سازی یک Animation ساده

در مثال زیر یک کلاس کوچک برای مدیریت Linear Animation پیاده‌سازی می‌کنیم.

این کلاس دارای ویژگی‌های زیر است:

- مقدار شروع و پایان.
- مدت زمان Animation.
- زمان سپری‌شده.
- محاسبه Progress.
- محدود کردن Progress به بازه معتبر.
- محاسبه مقدار فعلی با `std::lerp`.
- تشخیص پایان Animation.

```
#include <algorithm>
#include <cmath>
#include <stdexcept>

class LinearAnimation {
public:
    LinearAnimation(
        double start,
        double end,
        double duration
    )
        : start_(start),
          end_(end),
          duration_(duration),
          elapsed_(0.0) {

        if (!std::isfinite(start_) ||
            !std::isfinite(end_) ||
            !std::isfinite(duration_) ||
            duration_ <= 0.0) {
            throw std::invalid_argument("Invalid animation parameters");
        }
    }

    void update(double deltaTime) {
        if (!std::isfinite(deltaTime) || deltaTime < 0.0) {
            throw std::invalid_argument("Invalid delta time");
        }

        elapsed_ = std::min(
            elapsed_ + deltaTime,
            duration_
        );
    }

    double progress() const noexcept {
        return elapsed_ / duration_;
    }

    double value() const noexcept {
        return std::lerp(
            start_,
            end_,
            progress()
        );
    }

    bool finished() const noexcept {
        return elapsed_ >= duration_;
    }

private:
    double start_;
    double end_;
    double duration_;
    double elapsed_;
};

int main() {
    LinearAnimation animation(0.0, 100.0, 2.0);

    animation.update(0.5);

    double currentValue = animation.value();

    bool isFinished = animation.finished();
}
```

در این پیاده‌سازی، هر بار که `update` فراخوانی می‌شود، زمان سپری‌شده افزایش پیدا می‌کند.

سپس تابع `value` با استفاده از Progress فعلی، مقدار جدید را محاسبه می‌کند.

به دلیل استفاده از `std::min`، زمان سپری‌شده از مدت زمان Animation بیشتر نمی‌شود.

این طراحی برای یک Animation ساده مناسب است؛ البته در یک سیستم حرفه‌ای ممکن است قابلیت‌هایی مانند Pause، Resume، Reverse، Loop و Easing نیز به آن اضافه شوند.

## نکات کلیدی برای توسعه‌دهندگان حرفه‌ای

- تابع `std::lerp` از ++C20 در Header فایل `<cmath>` در دسترس است.
- این تابع برای Linear Interpolation روی Floating-Point Values طراحی شده است.
- نوع تمام آرگومان‌ها باید با نوع Template Parameter سازگار باشد.
- مقدار `t` به‌صورت خودکار به بازه `[0, 1]` محدود نمی‌شود.
- برای محدودسازی Progress می‌توان از `std::clamp` استفاده کرد.
- استفاده از `std::lerp` به‌جای فرمول دستی می‌تواند از مشکلات مهمی مانند Overflow و نقض Monotonicity در شرایط مشخص جلوگیری کند.
- این تابع Endpointها را برای `t == 0` و `t == 1` دقیقاً حفظ می‌کند.
- استفاده از `constexpr` امکان محاسبه در Compile-Time را فراهم می‌کند.
- این تابع جایگزین Cubic Interpolation، Spline یا Easing Functions نیست.
- تضمین‌های عددی آن به معنی حذف تمام خطاهای Floating-Point نیست.
- در ورودی‌های نامعتبر، به‌خصوص NaN و Infinity، باید سیاست اعتبارسنجی مشخصی داشته باشید.
- برای محاسبه نقطه میانی دو مقدار، `std::midpoint` نیز از ++C20 در دسترس است.
- عملکرد واقعی تابع به Compiler، معماری پردازنده و Optimization وابسته است.
- در سیستم‌های Deterministic باید تفاوت‌های احتمالی محاسبات Floating-Point بین پلتفرم‌ها را در نظر گرفت.

## جمع‌بندی

تابع `std::lerp` یکی از قابلیت‌های کاربردی Standard Library در ++C20 است که برای انجام Linear Interpolation به شکلی استاندارد و قابل‌اعتماد طراحی شده است.

این تابع علاوه بر ساده‌سازی محاسبات بین دو مقدار، ویژگی‌های عددی مهمی مانند حفظ Endpointها، Monotonicity و جلوگیری از Overflow در شرایط مشخص را فراهم می‌کند.

استفاده از `std::lerp` در حوزه‌هایی مانند Game Development، Animation، Computer Graphics، پردازش سیگنال و شبیه‌سازی‌های عددی می‌تواند کد را خواناتر و قابل‌نگهداری‌تر کند.

برای توسعه‌دهندگان ++C، توصیه می‌شود در پروژه‌هایی که از ++C20 یا استانداردهای جدیدتر استفاده می‌کنند، به‌جای پیاده‌سازی معمول و بدون محافظت Linear Interpolation، از `std::lerp` استفاده شود؛ مگر اینکه نیازهای خاص پروژه به الگوریتم متفاوتی احتیاج داشته باشند.

در نهایت، باید به خاطر داشت که `std::lerp` ابزاری برای Interpolation خطی است، نه راه‌حلی عمومی برای تمام مسائل مربوط به تغییر تدریجی مقادیر. انتخاب صحیح الگوریتم، مدیریت ورودی‌ها و درک محدودیت‌های Floating-Point همچنان از مسئولیت‌های مهم توسعه‌دهنده هستند.

مقاله را کوتاه‌تر و متمرکزتر کنهشدارهای عددی را دقیق‌تر و یکدست کن

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>