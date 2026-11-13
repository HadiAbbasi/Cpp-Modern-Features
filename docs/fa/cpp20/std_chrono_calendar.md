<div align="center">

[🇺🇸 English](../../en/cpp20/std_chrono_calendar.md) | [🇮🇷 فارسی](./std_chrono_calendar.md)

</div>

---

# راهنمای جامع Calendar در `std::chrono` در ++C20

## مقدمه

کتابخانه `std::chrono` در استاندارد ++C20 مجموعه‌ای از ابزارهای استاندارد
برای نمایش و محاسبه زمان، مدت‌زمان، تاریخ‌های تقویمی و در سطحی پیشرفته‌تر،
زمان‌های وابسته به منطقه زمانی را ارائه می‌کند. بخش Calendar این کتابخانه
امکان نمایش اجزای یک تاریخ میلادی، اعتبارسنجی آن‌ها، انجام محاسبات ماهانه
و سالانه و تبدیل تاریخ به یک نمایش عددی مبتنی بر روز را فراهم می‌سازد.

پیش از ++C20، امکانات استاندارد برای کار با تاریخ تقویمی محدود بود.
نوع‌هایی مانند `std::time_t` و ساختار `std::tm` بیشتر حول زمان تقویمی C و
توابعی مانند `std::localtime` و `std::gmtime` طراحی شده بودند. این APIها
برای محاسبات دقیق تقویمی، ایمنی در برابر هم‌زمانی و مدیریت صریح منطقه
زمانی محدودیت‌هایی داشتند. کتابخانه `std::chrono` در ++C20 مدل نوع‌امن‌تر و
قابل ترکیب‌تری ارائه می‌دهد.

این قابلیت‌ها در هدر `<chrono>` و فضای نام `std::chrono` تعریف شده‌اند.
برای استفاده از آن‌ها به پشتیبانی کتابخانه استاندارد از ++C20 نیاز دارید؛
فعال‌کردن حالت زبانی ++C20 به‌تنهایی تضمین نمی‌کند که تمام اجزای کتابخانه،
به‌خصوص Time Zone Database، در پیاده‌سازی مورد استفاده موجود باشند.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, February, 29d};

    if (date.ok()) {
        std::cout << date << '\n';
    }
}
```

## تفاوت تاریخ تقویمی و زمان نقطه‌ای

درک مدل داده‌ای `std::chrono` مهم‌ترین قدم برای استفاده صحیح از Calendar
است. تاریخ تقویمی و یک نقطه روی خط زمان دو مفهوم مرتبط، اما متفاوت‌اند.

### تاریخ تقویمی

تاریخ تقویمی از مؤلفه‌هایی مانند سال، ماه و روز تشکیل می‌شود. برای نمونه،
تاریخ `2024-02-29` یک تاریخ در تقویم میلادی است. این تاریخ به‌خودی‌خود
ساعت، دقیقه، ثانیه یا منطقه زمانی مشخصی ندارد.

نوع `year_month_day` دقیقاً برای چنین مدلی طراحی شده است. این نوع
مؤلفه‌های تاریخ را نگه می‌دارد و امکان بررسی اعتبار آن‌ها را فراهم می‌کند.

### زمان نقطه‌ای

یک `time_point` نقطه‌ای نسبت به Epoch یک Clock است. برای نمونه،
`sys_time` زمانی را نسبت به Epoch ساعت سیستم نمایش می‌دهد. یک زمان نقطه‌ای
می‌تواند دقتی مانند ثانیه، میلی‌ثانیه یا نانوثانیه داشته باشد.

تبدیل یک تاریخ به `sys_days` آن را به نقطه‌ای با دقت روز روی خط زمان
سیستم تبدیل می‌کند. این تبدیل به معنی تعیین ساعت محلی یا منطقه زمانی
نیست؛ تنها یک نمایش روزمحور از تاریخ به دست می‌دهد.

### مدت‌زمان

نوع `duration` فاصله زمانی را نشان می‌دهد، نه یک تاریخ یا لحظه مشخص.
انواعی مانند `days`، `months` و `years` در محاسبات Calendar نقش دارند،
اما معنای آن‌ها یکسان نیست. روز یک واحد با طول ثابت در این مدل است، در
حالی که ماه و سال واحدهای تقویمی با طول متغیر هستند.

## اجزای اصلی Calendar

تمام انواع Calendar در ++C20 در فضای نام `std::chrono` قرار دارند و در
هدر `<chrono>` تعریف شده‌اند.

### نوع `day`

نوع `day` یک روز از ماه را نمایش می‌دهد. مقدار آن معمولاً بین ۱ تا ۳۱ است،
اما سازنده آن الزاماً مقدار را به محدوده معتبر محدود نمی‌کند.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr day first{1};
    constexpr day thirty_first{31};
    constexpr day invalid{0};

    std::cout << first.ok() << '\n';
    std::cout << thirty_first.ok() << '\n';
    std::cout << invalid.ok() << '\n';
}
```

تابع عضو `ok()` بررسی می‌کند که مقدار `day` در محدوده عمومی معتبر باشد.
این بررسی نمی‌گوید که آن روز در یک ماه خاص وجود دارد؛ برای مثال `day{31}`
از نظر نوع `day` معتبر است، اما ۳۱ آوریل یک تاریخ معتبر نیست.

### نوع `month`

نوع `month` ماهی از سال میلادی را نمایش می‌دهد. مقادیر معتبر آن از ژانویه
تا دسامبر و به‌ترتیب از ۱ تا ۱۲ هستند.

کتابخانه ثابت‌های نام‌گذاری‌شده‌ای برای ماه‌ها ارائه می‌کند:

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr month m1 = January;
    constexpr month m2 = February;
    constexpr month m3 = December;

    static_assert(m1.ok());
    static_assert(m2.ok());
    static_assert(m3.ok());
}
```

مقداردهی عددی نیز ممکن است:

``` cpp
constexpr std::chrono::month march{3};
```

عملگرهای افزایش و کاهش و همچنین جمع و تفریق با `months` برای پیمایش
ماه‌ها وجود دارند. محاسبات ماهانه در سطح Calendar انجام می‌شوند و نباید
آن‌ها را با افزودن تعداد ثابتی روز اشتباه گرفت.

### نوع `year`

نوع `year` سال تقویم میلادی را نمایش می‌دهد. محدوده قابل نمایش آن در
تعریف استاندارد محدود است؛ در ++C20 مقدارهای سال در بازه
`[-32767, 32767]` قابل نمایش‌اند. این نوع برای نمایش سال میلادی است و
به‌تنهایی نوعی برای تقویم‌های دیگر، مانند هجری شمسی یا قمری، محسوب نمی‌شود.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr year y{2024};
    static_assert(y.ok());

    constexpr auto next = y + years{1};
    static_assert(next == year{2025});
}
```

پسوندهای عددی مانند `2024y` برای ساخت مقدار `year` در ++C20 ارائه
شده‌اند.

### نوع `weekday`

نوع `weekday` روز هفته را در چرخه هفت‌روزه میلادی نمایش می‌دهد. ثابت‌های
`Sunday`، `Monday`، `Tuesday`، `Wednesday`، `Thursday`، `Friday` و
`Saturday` در دسترس هستند.

در نمایش عددی این نوع، یکشنبه مقدار صفر و شنبه مقدار شش دارد. این
قرارداد با برخی APIها که دوشنبه را روز اول هفته در نظر می‌گیرند متفاوت
است؛ بنابراین هنگام تبدیل داده از سیستم‌های دیگر باید قرارداد روز اول
هفته را صریح بررسی کنید.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr weekday wd{Monday};

    std::cout << wd.c_encoding() << '\n';
    std::cout << wd.iso_encoding() << '\n';
}
```

تابع `c_encoding()` قرارداد عددی مبتنی بر یکشنبه صفر را برمی‌گرداند. تابع
`iso_encoding()` قرارداد ISO را به کار می‌گیرد که در آن دوشنبه یک و
یکشنبه هفت است.

### نوع `weekday_indexed`

نوع `weekday_indexed` یک روز هفته و شماره وقوع آن در ماه را نگه می‌دارد.
برای مثال، «دومین سه‌شنبه ماه» با این مدل قابل نمایش است.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr weekday_indexed second_tuesday = Tuesday[2];
    static_assert(second_tuesday.ok());
}
```

اندیس وقوع از ۱ تا ۵ است. معتبر بودن این ترکیب به معنای وجود قطعی آن
وقوع در هر ماه خاص نیست؛ این نوع به‌تنهایی هنوز ماه و سال را مشخص نمی‌کند.

### نوع `weekday_last`

نوع `weekday_last` آخرین وقوع یک روز هفته در ماه را نمایش می‌دهد. برای
نمونه، «آخرین جمعه ماه» را می‌توان با `Friday[last]` ساخت.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr weekday_last final_friday = Friday[last];
    static_assert(final_friday.ok());
}
```

### نوع `last`

ثابت `last` از نوع `last_spec` برای بیان آخرین روز ماه یا آخرین وقوع یک
روز هفته در ماه استفاده می‌شود. این ثابت به‌تنهایی یک تاریخ نیست؛ در ترکیب
با انواع Calendar معنا پیدا می‌کند.

## ساخت تاریخ با عملگر اسلش

یکی از ویژگی‌های کاربردی Calendar عملگر `/` است. این عملگر امکان ساخت
ترکیب‌های تقویمی را با ترتیبی خوانا فراهم می‌کند.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr auto date1 = 2024y / February / 29d;
    constexpr auto date2 = April / 2025;
    constexpr auto date3 = June / 15;
}
```

نوع عبارت حاصل با توجه به اجزای واردشده تعیین می‌شود. برای مثال، ترکیب
سال و ماه یک `year_month` می‌سازد و ترکیب ماه و روز یک `month_day`
می‌سازد. وقتی هر سه جزء حاضر باشند، نتیجه `year_month_day` است.

در کدهای واقعی بهتر است نوع حاصل را با `auto` یا نوع مشخصی مانند
`year_month_day` نگه دارید و برای جلوگیری از ابهام، از مقادیر دارای نوع
استفاده کنید. استفاده از اعداد خام در برخی ترکیب‌ها مجاز است، اما
پسوندهای `y` و `d` و ثابت‌های نام‌گذاری‌شده خوانایی و وضوح نوع را افزایش
می‌دهند.

## انواع ترکیبی تاریخ

### نوع `month_day`

نوع `month_day` یک ماه و یک روز را نگه می‌دارد، بدون آنکه سال مشخصی داشته
باشد. این مدل برای تاریخ‌هایی مانند «روز استقلال» یا «روز اول ژانویه» که
سال آن‌ها در الگو ثابت نیست مفید است.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr month_day new_year = January / 1;
    constexpr month_day halloween = October / 31;

    static_assert(new_year.ok());
    static_assert(halloween.ok());
}
```

تابع `ok()` در `month_day` می‌تواند بررسی کند که روز در محدوده قابل قبول
ماه قرار دارد، اما چون سال مشخص نیست، برای ماه فوریه نمی‌تواند به‌تنهایی
مشخص کند که روز ۲۹ در سال موردنظر معتبر است یا خیر.

### نوع `month_day_last`

نوع `month_day_last` آخرین روز یک ماه را نمایش می‌دهد، بدون نیاز به
دانستن تعداد روزهای آن ماه در یک سال خاص.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr month_day_last end_of_february = February / last;
}
```

برای به‌دست‌آوردن تاریخ واقعی، باید سال نیز مشخص شود و از
`year_month_day_last` استفاده کنید.

### نوع `year_month`

نوع `year_month` یک سال و ماه مشخص را نگه می‌دارد. این نوع برای محاسبات
ماهانه مانند حرکت به ماه بعد یا ماه قبل مفید است.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr year_month current = 2024y / December;
    constexpr year_month next = current + months{1};

    static_assert(next == 2025y / January);
}
```

### نوع `year_month_day`

نوع `year_month_day` متداول‌ترین نوع برای نمایش یک تاریخ کامل میلادی است.
این نوع سه مؤلفه سال، ماه و روز را نگه می‌دارد.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr year_month_day leap_day{2024y, February, 29d};
    static_assert(leap_day.ok());

    constexpr year_month_day invalid_date{2023y, February, 29d};
    static_assert(!invalid_date.ok());
}
```

این نوع می‌تواند تاریخ نامعتبر را نیز در مؤلفه‌های خود نگه دارد. بنابراین
ساخت موفق شیء به معنای معتبر بودن تاریخ نیست. همیشه در مرزهای ورودی،
داده‌های خارجی یا پس از محاسباتی که ممکن است روز را از محدوده ماه خارج
کنند، `ok()` را بررسی کنید.

### نوع `year_month_day_last`

نوع `year_month_day_last` آخرین روز یک ماه مشخص در یک سال مشخص را نمایش
می‌دهد. این نوع برای محاسبه انتهای ماه بدون hard-code کردن تعداد روزها
مناسب است.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr year_month_day_last end_of_february{
        2024y, February / last
    };

    const year_month_day date{end_of_february};

    std::cout << date << '\n';
}
```

تبدیل به `year_month_day` تاریخ واقعی آخر ماه را در اختیار قرار می‌دهد؛
برای سال کبیسه، آخرین روز فوریه ۲۹ و در سال عادی ۲۸ خواهد بود.

### نوع `month_weekday`

نوع `month_weekday` ماه و یک وقوع مشخص از روز هفته را نگه می‌دارد، اما
سال ندارد. برای مثال، «سومین پنجشنبه ماه نوامبر» در قالب یک الگوی ماهانه
قابل نمایش است.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr month_weekday thanksgiving_pattern =
        November / Thursday[4];
}
```

این الگو به‌تنهایی یک تاریخ قطعی نیست؛ برای تعیین روز واقعی باید سال نیز
به آن افزوده شود.

### نوع `month_weekday_last`

نوع `month_weekday_last` ماه و آخرین وقوع یک روز هفته در آن ماه را مشخص
می‌کند. این نوع نیز بدون سال است.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr month_weekday_last last_monday_pattern =
        February / Monday[last];
}
```

### نوع `year_month_weekday`

نوع `year_month_weekday` سال، ماه و n‌امین وقوع یک روز هفته را مشخص
می‌کند. این نوع برای قواعدی مانند «دومین دوشنبه اکتبر» مناسب است.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr year_month_weekday date_pattern =
        October / Monday[2] / 2024y;

    const sys_days point = date_pattern;
    const year_month_day date{point};

    std::cout << date << '\n';
}
```

این مدل برای محاسبه تعطیلات شناور و برنامه‌های دوره‌ای مفید است.

### نوع `year_month_weekday_last`

نوع `year_month_weekday_last` سال، ماه و آخرین وقوع یک روز هفته را مشخص
می‌کند. برای نمونه، «آخرین جمعه نوامبر ۲۰۲۴» با این نوع قابل نمایش است.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr year_month_weekday_last pattern =
        Friday[last] / November / 2024y;

    const year_month_day date{sys_days{pattern}};
    std::cout << date << '\n';
}
```

## اعتبارسنجی تاریخ

### بررسی اعتبار با `ok()`

انواع Calendar که مقادیر تقویمی نگه می‌دارند، تابع عضو `ok()` دارند. این
تابع برای بررسی اعتبار مقدار یا ترکیب مربوط به همان نوع استفاده می‌شود.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, February, 30d};

    if (!date.ok()) {
        std::cout << "Invalid date\n";
    }
}
```

اعتبارسنجی باید متناسب با نوع داده انجام شود. برای مثال، `day{31}` از
نظر محدوده نوع `day` معتبر است، اما `April / 31` معتبر نیست. همچنین
`month_day` فاقد سال است و نمی‌تواند اعتبار روز ۲۹ فوریه را برای یک سال
خاص تعیین کند.

### تفاوت مقدار نامعتبر و مقدار نرمال‌شده

نوع `year_month_day` هنگام ساخت، مؤلفه‌های ورودی را الزاماً اصلاح نمی‌کند.
برای نمونه، تاریخ ۳۱ آوریل می‌تواند در شیء ذخیره شود، ولی `ok()` مقدار
false برمی‌گرداند.

در صورت معتبر بودن سال و ماه، تبدیل چنین مقداری به `sys_days` یا
`local_days` بر اساس قواعد تبدیل استاندارد می‌تواند روز را به‌صورت حسابی
نسبت به روز اول ماه تفسیر کند. این رفتار با رد کردن ورودی یا clamp کردن
آن به آخر ماه متفاوت است.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day input{2024y, April, 31d};

    const sys_days normalized_point{input};
    const year_month_day normalized_date{normalized_point};

    std::cout << input.ok() << '\n';
    std::cout << normalized_date << '\n';
}
```

در این مثال، روز ۳۱ آوریل به روز بعد از ۳۰ آوریل تبدیل می‌شود؛ یعنی اول
مه. اگر سیاست برنامه این است که ورودی نامعتبر رد شود، باید پیش از تبدیل
`ok()` را بررسی کنید. تبدیل تقویمی را جایگزین اعتبارسنجی ورودی نکنید.

### محدودیت محدوده سال

نوع `year` محدوده عددی مشخصی دارد. در کاربردهایی مانند پردازش تاریخ‌های
تاریخی بسیار دور، داده‌های نجومی یا ورودی‌های نامحدود، باید محدوده قابل
نمایش نوع را در طراحی API در نظر بگیرید.

## تبدیل تاریخ به نمایش روزمحور

### نوع `sys_days`

نوع `sys_days` نام مستعار یک `time_point` مبتنی بر `system_clock` با دقت
روز است. مقدار آن تعداد روزهای سپری‌شده از Epoch ساعت سیستم، یعنی
۱۹۷۰-۰۱-۰۱، را نمایش می‌دهد.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr year_month_day date{2024y, May, 20d};
    constexpr sys_days point{date};

    std::cout << point.time_since_epoch().count() << '\n';
}
```

نمایش روزمحور برای مقایسه تاریخ‌ها، مرتب‌سازی، پیمایش روزها و انجام
محاسبات مبتنی بر تعداد روز مناسب است.

### نوع `local_days`

نوع `local_days` نیز یک `time_point` با دقت روز است، اما بر پایه
`local_t` تعریف شده و به‌جای خط زمان سیستم، برای زمان محلی بدون تعیین
منطقه زمانی به کار می‌رود.

تفاوت `sys_days` و `local_days` ظریف اما مهم است. هر دو می‌توانند یک
تاریخ تقویمی را در مقیاس روز نمایش دهند، ولی معنا و Clock پایه آن‌ها
متفاوت است. برای تبدیل تاریخ تقویمی به یک نقطه روی خط زمان سیستم معمولاً
از `sys_days` استفاده می‌شود. برای مدل‌سازی تاریخ/زمان محلی که هنوز به
منطقه زمانی نگاشت نشده، `local_days` مناسب است.

### تبدیل دوطرفه با `year_month_day`

تبدیل بین `year_month_day` و `sys_days` برای تغییر بین نمایش مؤلفه‌ای و
نمایش روزمحور استفاده می‌شود.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, July, 4d};

    const sys_days serial_date{date};
    const year_month_day recovered{serial_date};

    std::cout << recovered << '\n';
}
```

برای یک تاریخ معتبر، تبدیل به `sys_days` و بازگشت به `year_month_day`
همان تاریخ را بازسازی می‌کند. تبدیل معکوس برای مقادیر نامعتبر تابع قواعد
خاص خود است؛ بنابراین نباید بدون بررسی اعتبار، آن را راهی برای اصلاح
خودکار داده فرض کرد.

## محاسبات تقویمی

### افزودن و کاستن روز

برای افزودن یا کاستن تعداد روز، ابتدا تاریخ را به `sys_days` تبدیل کنید.
سپس `days` را به نقطه روزمحور اضافه یا از آن کم کنید.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day start{2024y, March, 1d};
    const sys_days start_point{start};

    const year_month_day after_ten_days{start_point + days{10}};

    std::cout << after_ten_days << '\n';
}
```

این روش از قواعد ماه و سال عبور می‌کند و به‌درستی طول ماه‌ها و سال کبیسه را
در نظر می‌گیرد.

### افزودن و کاستن ماه

برای محاسبات ماهانه، از `months` استفاده کنید. انواع `year_month` و
`year_month_day` عملگرهای لازم برای حرکت ماهانه را ارائه می‌کنند.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month current{2024y, January};
    const year_month next_month = current + months{1};

    std::cout << next_month << '\n';
}
```

رفتار افزودن ماه به یک تاریخ دارای روز باید با دقت بررسی شود. برای مثال،
افزودن یک ماه به ۳۱ ژانویه لزوماً به معنی آخر فوریه نیست؛ ممکن است نتیجه
میانی شامل روزی باشد که در ماه مقصد وجود ندارد. در چنین شرایطی `ok()` را
بررسی کنید یا سیاست موردنظر را با `year_month_day_last` پیاده‌سازی کنید.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    year_month_day date{2024y, January, 31d};
    date += months{1};

    std::cout << date.ok() << '\n';
}
```

اگر نیاز تجاری شما «همان شماره روز در ماه بعد» است، ممکن است نتیجه
نامعتبر باشد. اگر نیاز «آخرین روز ماه بعد در صورت نبودن شماره روز» است،
باید این سیاست را صریح پیاده‌سازی کنید.

### افزودن و کاستن سال

برای حرکت سالانه از `years` استفاده می‌شود. مسئله مهم در اینجا تاریخ ۲۹
فوریه است. افزودن یک سال به تاریخ ۲۹ فوریه ممکن است تاریخ نامعتبری در
سال مقصد ایجاد کند.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    year_month_day leap_day{2024y, February, 29d};
    leap_day += years{1};

    std::cout << leap_day.ok() << '\n';
}
```

در سامانه‌های مالی، رزرو یا اشتراک، مشخص کنید که سالگرد یک تاریخ کبیسه در
سال غیرکبیسه باید در ۲۸ فوریه، اول مارس یا طبق قاعده دیگری در نظر گرفته
شود. کتابخانه نمی‌تواند سیاست کسب‌وکار را حدس بزند.

### محاسبه آخرین روز ماه

با ترکیب `year_month_day_last` و `last` می‌توان آخرین روز هر ماه را بدون
جدول دستی تعداد روزها محاسبه کرد.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day_last end_of_month{
        2025y, February / last
    };

    const year_month_day date{end_of_month};

    std::cout << date << '\n';
}
```

این روش برای محاسبه پایان ماه، سررسیدهای ماهانه و گزارش‌های مالی کاربرد
دارد.

### محاسبه اولین روز ماه بعد

یک راه روشن برای یافتن اولین روز ماه بعد، کار با `year_month` است.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month current{2024y, December};
    const year_month next = current + months{1};
    const year_month_day first_day{next / 1};

    std::cout << first_day << '\n';
}
```

### محاسبه n‌امین روز هفته در ماه

برای یافتن n‌امین وقوع یک روز هفته، از `weekday_indexed` و
`year_month_weekday` استفاده کنید.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_weekday second_monday =
        Monday[2] / February / 2025y;

    const year_month_day date{sys_days{second_monday}};

    std::cout << date << '\n';
}
```

این الگو برای تعطیلات شناور، جلسات دوره‌ای و قواعد تقویمی مانند «دومین
دوشنبه ماه» مفید است.

### محاسبه آخرین روز هفته در ماه

برای یافتن آخرین وقوع یک روز هفته، از `weekday_last` استفاده کنید.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_weekday_last last_friday =
        Friday[last] / November / 2025y;

    const year_month_day date{sys_days{last_friday}};

    std::cout << date << '\n';
}
```

### محاسبه اختلاف دو تاریخ

برای محاسبه فاصله روزی بین دو تاریخ، هر دو را به `sys_days` تبدیل و از
هم کم کنید.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day first{2024y, January, 1d};
    const year_month_day second{2024y, December, 31d};

    const days difference = sys_days{second} - sys_days{first};

    std::cout << difference.count() << '\n';
}
```

این اختلاف تعداد مرزهای روزی بین دو تاریخ را نشان می‌دهد. اگر منظور از
«تعداد روزهای شامل ابتدا و انتها» شمارش inclusive باشد، ممکن است لازم
باشد یک روز به نتیجه اضافه کنید. تعریف دامنه بازه را در منطق برنامه شفاف
کنید.

## محاسبات روز هفته

### استخراج روز هفته از تاریخ

پس از تبدیل تاریخ به `sys_days`، می‌توان `weekday` متناظر را ساخت.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, January, 1d};
    const weekday day_of_week{sys_days{date}};

    std::cout << day_of_week << '\n';
}
```

### مقایسه روزهای هفته

مقادیر `weekday` قابل مقایسه‌اند. برای شرط‌هایی مانند آخر هفته یا روز
کاری، بهتر است قرارداد روزهای کاری را در برنامه تعریف کنید؛ استاندارد
به‌تنهایی تعیین نمی‌کند که کدام روزها در یک کشور یا سازمان تعطیل هستند.

``` cpp
#include <chrono>

bool is_weekend(std::chrono::weekday wd) {
    using namespace std::chrono;
    return wd == Saturday || wd == Sunday;
}
```

این تابع فقط یک فرض رایج درباره آخر هفته را پیاده‌سازی می‌کند. برای
سامانه‌های بین‌المللی، روزهای کاری و تعطیلات باید بر اساس locale یا قواعد
کسب‌وکار پیکربندی شوند.

### تفاوت کدگذاری C و ISO

در `weekday`، مقدار `c_encoding()` یکشنبه را صفر در نظر می‌گیرد، در حالی
که `iso_encoding()` دوشنبه را یک و یکشنبه را هفت در نظر می‌گیرد. هنگام
تبادل داده با APIهای دیگر، این تفاوت می‌تواند باعث جابه‌جایی روز هفته شود.

## دریافت تاریخ جاری

برای استخراج تاریخ جاری سیستم، از `system_clock::now()` استفاده کنید و
آن را به دقت روز گرد کنید.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto now = system_clock::now();
    const auto today = floor<days>(now);
    const year_month_day date{today};

    std::cout << date << '\n';
}
```

استفاده از `floor<days>` از خطای تبدیل نامناسب جلوگیری می‌کند. برای
زمان‌های پیش از Epoch، تفاوت `floor` با تبدیل ساده به عدد صحیح می‌تواند
مهم باشد؛ `floor` به سمت منفی بی‌نهایت گرد می‌کند، نه به سمت صفر.

نکته مهم این است که `system_clock` زمان سیستم را نمایش می‌دهد و این نمونه
تاریخ متناظر با مقیاس ساعت سیستم را استخراج می‌کند. برای تاریخ محلی
کاربر، باید منطقه زمانی را در نظر بگیرید؛ ساعت سیستم الزاماً با منطقه
زمانی موردنظر برنامه یکسان نیست.

## ترکیب تاریخ و ساعت روز

Calendar تنها بخش تاریخ را مدل می‌کند. برای نمایش ساعت روز، ++C20 نوع
`hh_mm_ss` را معرفی کرده است. این نوع یک `duration` را به ساعت، دقیقه،
ثانیه و بخش کسری تقسیم می‌کند.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr auto elapsed = 13h + 42min + 18s;
    const hh_mm_ss time_of_day{elapsed};

    std::cout << time_of_day << '\n';
}
```

`hh_mm_ss` یک نمایش تجزیه‌شده از مدت‌زمان است و به‌تنهایی یک تاریخ یا زمان
منطقه‌دار ایجاد نمی‌کند. برای ساخت زمان محلی کامل، می‌توان `local_days` را
با مدت‌زمان سپری‌شده از نیمه‌شب ترکیب کرد.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    const local_days date = local_days{days{20000}};
    const local_time<seconds> local_datetime =
        date + 13h + 42min + 18s;
}
```

در این مدل هنوز منطقه زمانی مشخص نشده است. تبدیل این زمان محلی به زمان
سیستم نیازمند قواعد منطقه زمانی است.

## منطقه‌های زمانی و زمان محلی

### تفاوت `local_time` و `sys_time`

نوع `sys_time` زمانی روی خط زمان سیستم را نمایش می‌دهد. این زمان مستقل از
نمایش محلی منطقه‌های زمانی است. نوع `local_time` یک زمان محلی را نمایش
می‌دهد که هنوز به منطقه زمانی خاصی متصل نشده است.

برای مثال، `2025-10-26 02:30` در یک منطقه زمانی ممکن است مبهم باشد؛ چون
ساعت در پایان Daylight Saving Time یک بار تکرار می‌شود. در بهار نیز ممکن
است بعضی ساعت‌های محلی اصلاً وجود نداشته باشند.

### معرفی `time_zone` و `zoned_time`

استاندارد ++C20 اجزایی برای کار با Time Zone Database معرفی می‌کند؛ از
جمله `time_zone`، `time_zone_link`، `tzdb`، `tzdb_list`، `zoned_time`،
`sys_info` و `local_info`.

نوع `zoned_time` یک زمان نقطه‌ای را همراه با منطقه زمانی نگه می‌دارد و
برای تبدیل بین زمان سیستم و نمایش محلی کاربرد دارد.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Europe/Paris");
    const zoned_time local{zone, system_clock::now()};

    std::cout << local << '\n';
}
```

در این مثال، شناسه منطقه زمانی باید در Time Zone Database پیاده‌سازی
موجود باشد. دسترسی به پایگاه داده و پشتیبانی از APIهای منطقه زمانی در
کتابخانه‌های استاندارد مختلف ممکن است متفاوت باشد؛ قابلیت زبانی ++C20
به‌تنهایی وجود داده‌های منطقه زمانی را تضمین نمی‌کند.

### تبدیل زمان محلی به زمان سیستم

تبدیل از زمان سیستم به زمان محلی با منطقه زمانی مشخص، یکتا است. اما
تبدیل از زمان محلی به زمان سیستم ممکن است مبهم یا ناممکن باشد.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Europe/Paris");
    const local_time<minutes> local =
        local_days{year{2025}/October/26} + 2h + 30min;

    const local_info info = zone->get_info(local);

    if (info.result == local_info::unique) {
        const zoned_time result{zone, local};
        std::cout << result << '\n';
    } else if (info.result == local_info::ambiguous) {
        std::cout << "Ambiguous local time\n";
    } else {
        std::cout << "Nonexistent local time\n";
    }
}
```

در زمان مبهم، برنامه باید تصمیم بگیرد کدام رخداد را انتخاب کند. در زمان
ناموجود نیز باید سیاستی مانند رد ورودی یا جابه‌جایی به زمان معتبر بعدی
تعریف شود. سازنده‌های `zoned_time` امکان استفاده از سیاست‌های
`choose::earliest` و `choose::latest` را برای حل ابهام فراهم می‌کنند.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Europe/Paris");
    const local_time<minutes> local =
        local_days{year{2025}/October/26} + 2h + 30min;

    const zoned_time resolved{zone, local, choose::earliest};
}
```

سیاست انتخاب تنها برای ابهام به کار می‌رود؛ برای زمان ناموجود، انتخاب
earliest یا latest مسئله را به‌صورت عمومی حل نمی‌کند. رفتار مورد انتظار
برنامه را صریح تعریف کنید.

### دریافت اطلاعات منطقه زمانی

تابع `get_info` می‌تواند اطلاعات مربوط به یک `sys_time` یا `local_time`
را از منطقه زمانی دریافت کند. اطلاعاتی مانند offset از UTC، نام اختصاری
منطقه و بازه اعتبار آن در `sys_info` یا `local_info` ارائه می‌شود.

این اطلاعات برای نمایش offset، تشخیص تغییر ساعت تابستانی و تحلیل زمان‌های
محلی مفید است. نباید فرض کنید offset یک منطقه زمانی در تمام سال یا در
تمام سال‌های تاریخی ثابت است.

### به‌روزرسانی پایگاه داده منطقه زمانی

قواعد منطقه‌های زمانی ممکن است با تصمیم‌های قانونی تغییر کنند. Time Zone
Database نسخه‌ای از داده‌ها را در اختیار کتابخانه قرار می‌دهد و APIهایی
برای دسترسی به آن دارد. سامانه‌هایی که به صحت زمان‌های آینده یا تاریخی
وابسته‌اند باید سیاست به‌روزرسانی داده‌های منطقه زمانی را در نظر بگیرند.

## قالب‌بندی و تجزیه تاریخ

### خروجی با `operator<<`

انواع Calendar از عملگر درج در stream پشتیبانی می‌کنند. این روش برای
نمایش سریع و عیب‌یابی مفید است.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, August, 17d};
    std::cout << date << '\n';
}
```

### قالب‌بندی با `std::format`

++C20 پشتیبانی قالب‌بندی برای انواع زمانی را از طریق `std::format` و
formatterهای مربوط به `chrono` معرفی می‌کند. با این حال، پیاده‌سازی
کتابخانه استاندارد ممکن است در نسخه‌های مختلف، پشتیبانی متفاوتی داشته
باشد.

``` cpp
#include <chrono>
#include <format>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, August, 17d};

    std::cout << std::format("{:%Y-%m-%d}", date) << '\n';
}
```

قالب‌های رایج شامل `%Y` برای سال، `%m` برای ماه و `%d` برای روز هستند.
برای استفاده از formatterهای `chrono`، مستندات پیاده‌سازی و نسخه
استاندارد کتابخانه را بررسی کنید.

### تجزیه ورودی با `std::chrono::from_stream`

در ++C20 امکان تجزیه مقادیر تقویمی از stream با `from_stream` وجود دارد.
این API از مشخص‌کننده‌های قالب‌بندی زمانی استفاده می‌کند.

``` cpp
#include <chrono>
#include <iostream>
#include <sstream>

int main() {
    using namespace std::chrono;

    std::istringstream input{"2024-02-29"};
    year_month_day date;

    from_stream(input, "%F", date);

    if (!input.fail() && date.ok()) {
        std::cout << date << '\n';
    } else {
        std::cout << "Invalid input\n";
    }
}
```

برای ورودی خارجی، تنها موفقیت parsing کافی نیست؛ اعتبار تقویمی را نیز با
`ok()` بررسی کنید. همچنین قالب ورودی باید با قرارداد داده‌ای برنامه
هماهنگ باشد. برای نمونه، `%F` قالب تاریخ ISO با ترتیب سال-ماه-روز است.

## نمونه کاربردی: محاسبه بازه یک ماه

در گزارش‌گیری و پرس‌وجوهای پایگاه داده، اغلب به ابتدای ماه و ابتدای ماه
بعد نیاز دارید. استفاده از بازه نیمه‌باز `[start, end)` معمولاً از خطاهای
مربوط به آخرین لحظه روز جلوگیری می‌کند.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month month_to_report{2024y, February};

    const sys_days start{month_to_report / 1};
    const sys_days end{(month_to_report + months{1}) / 1};

    std::cout << start << '\n';
    std::cout << end << '\n';
}
```

در این مدل، تمام زمان‌هایی که از ابتدای ماه شروع می‌شوند و پیش از ابتدای
ماه بعد قرار دارند، متعلق به بازه گزارش هستند. برای داده‌های timestamp،
مرزهای روز باید با منطقه زمانی موردنظر ساخته و سپس به زمان سیستم تبدیل
شوند؛ استفاده مستقیم از UTC برای گزارش محلی ممکن است رکوردهای نزدیک
نیمه‌شب را در ماه یا روز اشتباه قرار دهد.

## نمونه کاربردی: ساخت تقویم روزهای یک ماه

برای پیمایش روزهای یک ماه، تبدیل ابتدا و انتهای ماه به `sys_days` راهی
ساده و مطمئن است.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month selected_month{2024y, February};
    const sys_days first{selected_month / 1};
    const sys_days next_month{(selected_month + months{1}) / 1};

    for (sys_days current = first; current < next_month; current += days{1}) {
        const year_month_day date{current};
        const weekday wd{current};

        std::cout << date << " weekday=" << wd << '\n';
    }
}
```

این روش به‌طور طبیعی تعداد روزهای ماه و سال کبیسه را مدیریت می‌کند و از
ساخت دستی فهرست روزها جلوگیری می‌کند.

## نمونه کاربردی: محاسبه تاریخ سررسید ماهانه

در برنامه‌های مالی، افزودن یک ماه به تاریخ سررسید ممکن است با سیاست خاصی
همراه باشد. نمونه زیر سیاست «اگر روز در ماه مقصد وجود نداشت، آخرین روز
ماه مقصد را انتخاب کن» را پیاده‌سازی می‌کند.

``` cpp
#include <chrono>
#include <iostream>

std::chrono::year_month_day add_month_clamped(
    std::chrono::year_month_day date,
    std::chrono::months delta
) {
    using namespace std::chrono;

    const year_month target_month = date.year() / date.month() + delta;
    const year_month_day candidate{target_month / date.day()};

    if (candidate.ok()) {
        return candidate;
    }

    return year_month_day{target_month / last};
}

int main() {
    using namespace std::chrono;

    const year_month_day due_date{2024y, January, 31d};
    const auto next_due_date = add_month_clamped(due_date, months{1});

    std::cout << next_due_date << '\n';
}
```

این تابع یک سیاست نمونه است، نه رفتار پیش‌فرض همه محاسبات ماهانه. در
سیستم‌های مالی، قرارداد کسب‌وکار ممکن است سیاست دیگری مانند حفظ فاصله روزی
یا انتقال به روز کاری بعد را لازم بداند.

## تفاوت Calendar با APIهای قدیمی

### مقایسه با `std::tm`

ساختار `std::tm` از API زمان C می‌آید و فیلدهایی برای سال، ماه، روز،
ساعت، دقیقه و ثانیه دارد. در این ساختار، سال نسبت به ۱۹۰۰ و ماه از صفر
شماره‌گذاری می‌شود؛ این قراردادها مستعد خطای off-by-one هستند.

در مقابل، `year_month_day` از مقادیر نوع‌دار استفاده می‌کند و ماه‌ها را با
مقادیر ۱ تا ۱۲ نمایش می‌دهد. همچنین انواع Calendar برای اعتبارسنجی و
محاسبات تقویمی طراحی شده‌اند.

### مقایسه با `std::chrono::system_clock::to_time_t`

تابع `to_time_t` یک زمان نقطه‌ای را به `time_t` تبدیل می‌کند. این تبدیل
برای تعامل با APIهای قدیمی سیستم‌عامل یا C مفید است، اما برای محاسبات
Calendar مدرن انتخاب اصلی نیست. بهتر است تا حد امکان در بخش داخلی برنامه
از انواع `std::chrono` استفاده و تنها در مرز سازگاری تبدیل انجام دهید.

### مقایسه با کتابخانه‌های تاریخ خارجی

کتابخانه‌های خارجی ممکن است تقویم‌های غیرمیلادی، parsing پیشرفته،
localeهای گسترده یا امکانات منطقه زمانی خاصی ارائه کنند. Calendar
استاندارد ++C20 بر تقویم میلادی متکی است و جایگزین کامل کتابخانه‌های
تخصصی بین‌المللی‌سازی یا تقویم‌های چندگانه نیست.

## محدودیت‌ها و نکات طراحی

### تمرکز بر تقویم میلادی

انواع `year`، `month`، `day` و انواع ترکیبی Calendar برای تقویم میلادی
طراحی شده‌اند. برای تقویم هجری شمسی، قمری، عبری یا تقویم‌های دیگر، به مدل
یا کتابخانه تخصصی نیاز دارید. تبدیل تقویم‌ها باید با قواعد معتبر و مشخص
انجام شود.

### نبود مفهوم تعطیلات و روز کاری

کتابخانه روز هفته و تاریخ را محاسبه می‌کند، اما فهرست تعطیلات رسمی،
تعطیلات سازمانی، روزهای کاری بانک‌ها یا قواعد منطقه‌ای را نگه نمی‌دارد. این
اطلاعات باید در لایه دامنه برنامه مدیریت شوند.

### تفاوت روز تقویمی و ۲۴ ساعت

افزودن `days{1}` به یک زمان روی خط زمان سیستم به معنی افزودن دقیقاً ۲۴
ساعت است. اما افزودن یک روز تقویمی به زمان محلی در منطقه‌ای که تغییر ساعت
تابستانی دارد ممکن است به فاصله‌ای برابر با ۲۳ یا ۲۵ ساعت روی خط زمان
سیستم منجر شود.

برای محاسبات زمان‌بندی محلی، ابتدا مشخص کنید مسئله شما «۲۴ ساعت بعد» است
یا «همان ساعت در روز تقویمی بعد». این دو نیاز متفاوت‌اند.

### سیاست صریح برای تاریخ‌های نامعتبر

تاریخ نامعتبر را بی‌صدا normalize نکنید، مگر اینکه این رفتار بخشی از
قرارداد API باشد. در ورودی‌های کاربر، فایل‌ها و پیام‌های شبکه معمولاً رد
کردن مقدار نامعتبر و گزارش خطا انتخاب امن‌تری است.

### سیاست صریح برای ابهام‌های منطقه زمانی

در زمان‌های تکرارشده یا حذف‌شده به‌علت تغییر ساعت رسمی، برنامه باید رفتار
مشخصی داشته باشد. انتخاب اولین یا آخرین رخداد، رد مقدار یا انتقال به
زمان معتبر بعدی، تصمیم‌های دامنه‌ای هستند.

### تفاوت پیاده‌سازی‌ها

وجود ویژگی در استاندارد با دسترس‌بودن کامل آن در نسخه مشخص کامپایلر و
کتابخانه استاندارد یکسان نیست. امکانات Calendar، formatting و Time Zone
Database ممکن است در نسخه‌های متفاوت GCC/libstdc++، Clang/libc++ و MSVC
STL وضعیت پشتیبانی متفاوتی داشته باشند. برای پروژه واقعی، یک برنامه کوچک
آزمایشی و ماتریس CI روی toolchainهای هدف داشته باشید.

## خطاهای رایج

-   **فرض معتبر بودن تاریخ پس از ساخت:** ساخت `year_month_day` به‌تنهایی
    اعتبار تاریخ را تضمین نمی‌کند؛ `ok()` را بررسی کنید.
-   **اشتباه گرفتن `months` با تعداد روز ثابت:** طول ماه‌ها یکسان نیست؛
    برای محاسبه ماهانه از `months` استفاده کنید.
-   **افزودن ۲۴ ساعت به‌جای یک روز محلی:** تغییر ساعت تابستانی می‌تواند
    طول روز محلی را تغییر دهد.
-   **فرض یکسان بودن شماره‌گذاری روز هفته:** قرارداد `c_encoding()` با
    `iso_encoding()` متفاوت است.
-   **نادیده گرفتن سال کبیسه:** تاریخ ۲۹ فوریه و محاسبات سالانه نیازمند
    سیاست مشخص هستند.
-   **فرض وجود Time Zone Database در هر پیاده‌سازی:** پشتیبانی کتابخانه و
    داده‌های منطقه زمانی را در toolchain هدف بررسی کنید.
-   **تبدیل خودکار تاریخ نامعتبر به تاریخ معتبر:** این تبدیل ممکن است
    تاریخ را به ماه بعد ببرد و با سیاست clamp مورد انتظار متفاوت باشد.
-   **استفاده از تاریخ UTC برای گزارش محلی:** بازه روز یا ماه محلی باید
    بر اساس منطقه زمانی هدف محاسبه شود.

## راهنمای انتخاب نوع مناسب

  نیاز                                نوع یا ابزار پیشنهادی
  ----------------------------------- ----------------------------------------
  نمایش سال                           `year`
  نمایش ماه                           `month`
  نمایش روز ماه                       `day`
  نمایش روز هفته                      `weekday`
  نمایش تاریخ کامل                    `year_month_day`
  نمایش ماه و سال                     `year_month`
  نمایش آخرین روز ماه                 `year_month_day_last`
  نمایش n‌امین روز هفته ماه            `year_month_weekday`
  نمایش آخرین روز هفته ماه            `year_month_weekday_last`
  محاسبه تعداد روزها                  `sys_days` و `days`
  نمایش تاریخ محلی بدون منطقه زمانی   `local_days` یا `local_time`
  نمایش زمان با منطقه زمانی           `zoned_time`
  شکستن مدت‌زمان به اجزای ساعت         `hh_mm_ss`
  قالب‌بندی تاریخ و زمان               `std::format` با formatterهای `chrono`
  تجزیه تاریخ از stream               `std::chrono::from_stream`

## جمع‌بندی

بخش Calendar در `std::chrono` از ++C20 یک مدل استاندارد، نوع‌امن و قابل
ترکیب برای تاریخ‌های میلادی فراهم می‌کند. انواع مؤلفه‌ای مانند
`year_month_day` برای نمایش و اعتبارسنجی تاریخ مناسب‌اند؛ انواع الگو
مانند `year_month_weekday` برای قواعد تقویمی شناور به کار می‌روند؛ و
`sys_days` امکان محاسبات روزمحور دقیق را فراهم می‌کند.

برای استفاده حرفه‌ای، سه تصمیم را از یکدیگر جدا نگه دارید: اعتبار تاریخ،
سیاست محاسبات ماهانه و سالانه، و تبدیل زمان محلی به زمان سیستم. همچنین
به خاطر داشته باشید که Calendar استاندارد تقویم میلادی را مدل می‌کند و
منطق تعطیلات، روزهای کاری و سیاست‌های منطقه‌ای باید در لایه‌های دیگر برنامه
پیاده‌سازی شوند.

## منابع تکمیلی

-   [cppreference: Date and time
    library](https://en.cppreference.com/w/cpp/chrono)
-   [cppreference: Header
    `<chrono>`](https://en.cppreference.com/w/cpp/header/chrono)
-   [cppreference:
    `std::chrono::year_month_day`](https://en.cppreference.com/w/cpp/chrono/year_month_day)
-   [cppreference:
    `std::chrono::year_month_day_last`](https://en.cppreference.com/w/cpp/chrono/year_month_day_last)
-   [Microsoft Learn:
    `year_month_day`](https://learn.microsoft.com/en-us/cpp/standard-library/year-month-day-class)
-   [Working Draft, Standard for Programming Language
    C++](https://eel.is/c++draft/time)


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>