<div align="right">

[🇺🇸 English](../../en/cpp20/constinit.md) | [🇮🇷 فارسی](./constinit.md)

</div>

---
# آموزش constinit در سی‌پلاس‌پلاس؛ کنترل مقداردهی اولیه در زمان کامپایل

## فهرست مطالب

* [مقدمه](#مقدمه)
* [constinit چیست؟](#constinit-چیست)
* [چرا constinit به وجود آمد؟](#چرا-constinit-به-وجود-آمد)
* [مشکل مقداردهی اولیه پویا چیست؟](#مشکل-مقداردهی-اولیه-پویا-چیست)
* [نحوه استفاده از constinit](#نحوه-استفاده-از-constinit)
* [آیا مقدار constinit را می‌توان تغییر داد؟](#آیا-مقدار-constinit-را-می‌توان-تغییر-داد)
* [اگر برای constinit مقدار اولیه ندهیم چه می‌شود؟](#اگر-برای-constinit-مقدار-اولیه-ندهیم-چه-می‌شود)
* [تفاوت constinit با const](#تفاوت-constinit-با-const)
* [تفاوت constinit با constexpr](#تفاوت-constinit-با-constexpr)
* [تفاوت constinit با consteval](#تفاوت-constinit-با-consteval)
* [مقایسه نهایی const و constexpr و consteval و constinit](#مقایسه-نهایی-const-و-constexpr-و-consteval-و-constinit)
* [مثال‌های کاربردی](#مثال‌های-کاربردی)
* [یک مثال پیشرفته با static initialization](#یک-مثال-پیشرفته-با-static-initialization)
* [محدودیت‌های constinit](#محدودیت‌های-constinit)
* [اگر constinit وجود نداشت چه مشکلی داشتیم؟](#اگر-constinit-وجود-نداشت-چه-مشکلی-داشتیم)
* [چه زمانی از constinit استفاده کنیم؟](#چه-زمانی-از-constinit-استفاده-کنیم)
* [جمع‌بندی](#جمع‌بندی)

---

# مقدمه

کلمه `constinit` یکی از قابلیت‌های اضافه‌شده به زبان سی‌پلاس‌پلاس در استاندارد **C++20** است که برای کنترل نحوه مقداردهی اولیه متغیرهایی با **static storage duration** یا **thread storage duration** طراحی شده است.

مفهوم اصلی `constinit` بسیار ساده است:

> `constinit` به کامپایلر می‌گوید که متغیر موردنظر **حتماً باید به‌صورت constant initialization مقداردهی اولیه شود**.

نکته مهم این است که `constinit` برخلاف نامش، متغیر را `const` نمی‌کند.

یعنی متغیری مانند نمونه زیر بعد از مقداردهی اولیه همچنان می‌تواند تغییر کند:

```cpp
constinit int counter = 10;

counter = 20;
counter = 30;
```

در این مثال مقدار اولیه `counter` باید به‌شکل ثابت و در مرحله مناسب initialization مشخص شود، اما خود `counter` یک متغیر قابل‌تغییر باقی می‌ماند.

---

# constinit چیست؟

کلمه `constinit` یک **specifier** در زبان سی‌پلاس‌پلاس است که روی متغیرهایی با طول عمر ایستا یا نخی اعمال می‌شود.

نمونه ساده زیر را ببینید:

```cpp
constinit int value = 42;
```

در اینجا کامپایلر موظف است بررسی کند که `value` دارای **constant initializer** باشد.

مثال زیر نیز معتبر است:

```cpp
constinit int value = 10 + 20;
```

زیرا مقدار اولیه `30` می‌تواند به‌صورت ثابت محاسبه شود.

اما هدف `constinit` فقط این نیست که کامپایلر یک عبارت را در زمان کامپایل محاسبه کند.

هدف اصلی این است که مطمئن شویم متغیر موردنظر دچار **dynamic initialization** نمی‌شود.

---

# چرا constinit به وجود آمد؟

مسئله اصلی `constinit` به رفتار مقداردهی اولیه متغیرهای دارای **static storage duration** مربوط می‌شود.

متغیرهای سراسری و متغیرهای `static` می‌توانند در فرایند راه‌اندازی برنامه مقداردهی اولیه شوند.

این مقداردهی اولیه در حالت کلی می‌تواند به دو شکل مهم انجام شود:

```text
constant initialization
dynamic initialization
```

مقداردهی اولیه ثابت معمولاً می‌تواند قبل از شروع اجرای عادی برنامه انجام شود.

مقداردهی اولیه پویا ممکن است به اجرای کد در زمان startup برنامه نیاز داشته باشد.

مشکل زمانی ایجاد می‌شود که برنامه‌نویس انتظار داشته باشد یک متغیر حتماً قبل از شروع سایر بخش‌های برنامه مقدار مشخصی داشته باشد، اما مقداردهی آن به‌صورت dynamic initialization انجام شود.

---

# مشکل مقداردهی اولیه پویا چیست؟

فرض کنید دو فایل سورس مستقل داریم.

فایل اول چنین چیزی دارد:

```cpp
int getValue();

int globalValue = getValue();
```

فایل دوم نیز چنین چیزی دارد:

```cpp
int otherValue = globalValue;
```

در چنین شرایطی ممکن است ترتیب مقداردهی اولیه متغیرهای سراسری در واحدهای ترجمه مختلف باعث ایجاد مشکل شود.

مسئله معروفی که در این زمینه مطرح می‌شود **Static Initialization Order Fiasco** است.

یعنی ممکن است یک متغیر سراسری قبل از اینکه متغیر دیگری که به آن وابسته است مقداردهی شود، استفاده شود.

مثلاً تصور کنید:

```cpp
// file_a.cpp

int getNumber()
{
    return 42;
}

int number = getNumber();
```

و در فایل دیگری داشته باشیم:

```cpp
// file_b.cpp

extern int number;

int value = number;
```

در اینجا `number` برای مقداردهی به `value` استفاده شده است.

اگر `number` نیازمند dynamic initialization باشد، ترتیب initialization بین واحدهای ترجمه مختلف می‌تواند به مسئله تبدیل شود.

---

# راه‌حل قدیمی چیست؟

یکی از راه‌حل‌های رایج این بود که متغیر را `constexpr` کنیم.

مثلاً:

```cpp
constexpr int number = 42;
```

این روش بسیار خوب است، اما یک تفاوت مهم وجود دارد.

متغیر `constexpr` باید یک **constant** باشد.

یعنی نمی‌توان بعداً مقدار آن را تغییر داد:

```cpp
constexpr int number = 42;

// خطای کامپایل
number = 100;
```

گاهی ما دقیقاً چنین چیزی نمی‌خواهیم.

گاهی نیاز داریم مقدار اولیه متغیر حتماً به‌شکل ثابت تعیین شود، اما خود متغیر بعداً قابل تغییر باشد.

در چنین شرایطی `constinit` ابزار مناسبی است.

---

# نحوه استفاده از constinit

ساده‌ترین شکل استفاده از `constinit` به‌صورت زیر است:

```cpp
constinit int value = 42;
```

در اینجا `value` یک متغیر معمولی است.

یعنی این کد کاملاً معتبر است:

```cpp
constinit int value = 42;

int main()
{
    value = 100;
}
```

بعد از اجرای این برنامه مقدار `value` برابر `100` خواهد بود.

نکته مهم این است که `constinit` فقط **نحوه initialization اولیه** را محدود می‌کند، نه نحوه استفاده از متغیر بعد از initialization را.

---

# مقدار اولیه باید چه ویژگی‌ای داشته باشد؟

مقدار اولیه یک متغیر `constinit` باید امکان **constant initialization** داشته باشد.

مثلاً کد زیر معتبر است:

```cpp
constinit int x = 10;
```

عبارت زیر نیز معتبر است:

```cpp
constinit int x = 10 * 20;
```

اما عبارت زیر مشکل دارد:

```cpp
int getValue();

constinit int x = getValue();
```

در این حالت مقدار `getValue()` باید هنگام اجرای برنامه محاسبه شود و نمی‌تواند به‌عنوان constant initializer مورد استفاده قرار بگیرد.

در نتیجه کامپایلر باید برنامه را رد کند.

---

# آیا مقدار constinit را می‌توان تغییر داد؟

بله.

این یکی از مهم‌ترین تفاوت‌های `constinit` با `const` و `constexpr` است.

کد زیر کاملاً معتبر است:

```cpp
constinit int counter = 0;

void increment()
{
    ++counter;
}
```

در اینجا `counter` ثابت نیست.

فقط تضمین شده است که initialization اولیه آن به‌شکل constant initialization انجام شود.

بنابراین می‌توانیم بگوییم:

```text
constinit
    initialization را محدود می‌کند

const
    امکان تغییر مقدار را محدود می‌کند

constexpr
    هم constant expression بودن مقدار را دنبال می‌کند
    و هم متغیر را غیرقابل‌تغییر می‌کند
```

---

# اگر برای constinit مقدار اولیه ندهیم چه می‌شود؟

این بخش کمی ظریف‌تر است.

کدی مانند زیر را در نظر بگیرید:

```cpp
constinit int value;
```

در این حالت `value` می‌تواند به‌صورت zero-initialization مقداردهی شود و مقدار اولیه آن `0` خواهد بود.

بنابراین در چنین موردی نداشتن initializer لزوماً به معنی خطای کامپایل نیست.

مثلاً:

```cpp
constinit int value;

int main()
{
    // مقدار value برابر صفر است
}
```

نکته مهم این است که `constinit` همچنان باید بتواند الزام constant initialization را برآورده کند.

از طرف دیگر، اگر declaration از نوع `extern` باشد، می‌توان declaration را بدون initializer نوشت و تعریف واقعی متغیر را در جای دیگری قرار داد.

مثلاً:

```cpp
// header.hpp

extern constinit int counter;
```

و سپس:

```cpp
// source.cpp

constinit int counter = 100;
```

این الگو برای متغیرهای سراسری قابل استفاده است.

---

# تفاوت constinit با const

کلمات `const` و `constinit` از نظر ظاهری شبیه یکدیگر هستند، اما هدف کاملاً متفاوتی دارند.

کلمه `const` می‌گوید:

> این شیء بعد از initialization از طریق این نام قابل تغییر نیست.

مثال:

```cpp
const int value = 42;
```

در این حالت:

```cpp
value = 100;
```

غیرمجاز است.

کلمه `constinit` می‌گوید:

> این متغیر باید با constant initialization مقداردهی شود.

مثال:

```cpp
constinit int value = 42;
```

در اینجا:

```cpp
value = 100;
```

کاملاً مجاز است.

بنابراین می‌توانیم تفاوت اصلی را این‌گونه خلاصه کنیم:

| ویژگی                             | `const`                 | `constinit` |
| --------------------------------- | ----------------------- | ----------- |
| غیرقابل تغییر کردن متغیر          | بله                     | خیر         |
| اجبار به constant initialization  | خیر                     | بله         |
| مناسب برای متغیرهای static/thread | محدود به این موضوع نیست | بله         |
| اضافه‌شده در C++20                | خیر                     | بله         |
| جلوگیری از dynamic initialization | هدف اصلی نیست           | بله         |

---

# تفاوت constinit با constexpr

تفاوت `constinit` و `constexpr` از مهم‌ترین قسمت‌های این بحث است.

کلمه `constexpr` برای بیان این موضوع استفاده می‌شود که یک مقدار یا تابع بتواند در **constant expression** استفاده شود.

مثلاً:

```cpp
constexpr int size = 100;
```

این مقدار می‌تواند در موقعیت‌هایی که constant expression لازم است استفاده شود.

مثلاً:

```cpp
int array[size];
```

در مقابل:

```cpp
constinit int size = 100;
```

هدف اصلی این declaration این نیست که `size` را به یک constant expression قابل استفاده در همه contextها تبدیل کند.

همچنین `constinit` باعث `const` شدن متغیر نمی‌شود.

بنابراین:

```cpp
constexpr int a = 10;
```

و:

```cpp
constinit int b = 10;
```

رفتار یکسانی ندارند.

متغیر `a` قابل تغییر نیست:

```cpp
// خطا
a = 20;
```

اما `b` قابل تغییر است:

```cpp
b = 20;
```

---

# تفاوت constinit با consteval

کلمه `consteval` نیز با `constinit` تفاوت بنیادی دارد.

کلمه `consteval` برای تعریف **immediate function** استفاده می‌شود.

مثلاً:

```cpp
consteval int square(int x)
{
    return x * x;
}
```

فراخوانی چنین تابعی باید در context مناسب به‌صورت compile-time ارزیابی شود:

```cpp
constexpr int value = square(10);
```

بنابراین:

```text
consteval
    درباره تابع است.

constexpr
    درباره قابلیت ارزیابی در زمان کامپایل است.

constinit
    درباره initialization یک متغیر static یا thread-local است.

const
    درباره غیرقابل‌تغییر بودن شیء است.
```

این چهار کلمه را نباید صرفاً به خاطر شباهت اسمی در یک دسته قرار داد.

هرکدام مسئله متفاوتی را حل می‌کنند.

---

# مقایسه نهایی const و constexpr و consteval و constinit

جدول زیر یک تصویر کلی از تفاوت این چهار مفهوم ارائه می‌دهد:

| ویژگی                             |       `const` |        `constexpr` | `consteval` | `constinit` |
| --------------------------------- | ------------: | -----------------: | ----------: | ----------: |
| مربوط به متغیر                    |           بله |                بله |         خیر |         بله |
| مربوط به تابع                     |           خیر |                بله |         بله |         خیر |
| متغیر را غیرقابل تغییر می‌کند     |           بله |                بله |           — |         خیر |
| اجرای اجباری در زمان کامپایل      |           خیر | در contextهای لازم |         بله |         خیر |
| جلوگیری از dynamic initialization | هدف اصلی نیست |     در موارد مناسب |           — |         بله |
| نیازمند static/thread storage     |           خیر |                خیر |           — |         بله |
| معرفی در C++20                    |           خیر |                خیر |       C++20 |       C++20 |

---

# مثال‌های کاربردی

فرض کنید یک متغیر global داریم که مقدار اولیه آن از یک عبارت ثابت به دست می‌آید:

```cpp
constinit int maxConnections = 100;
```

حالا در زمان اجرای برنامه می‌توانیم آن را تغییر دهیم:

```cpp
void configure()
{
    maxConnections = 200;
}
```

این دقیقاً یکی از کاربردهای مناسب `constinit` است.

---

# مثال ساده با تابع

حال فرض کنید مقدار اولیه از تابعی به دست می‌آید:

```cpp
int getDefaultValue()
{
    return 100;
}

constinit int value = getDefaultValue();
```

این کد معتبر نیست.

علت این است که `getDefaultValue()` یک تابع معمولی است و فراخوانی آن نمی‌تواند به‌عنوان constant initializer استفاده شود.

اگر هدف ما این باشد که تابع هنگام کامپایل اجرا شود، می‌توانیم در شرایط مناسب از `constexpr` استفاده کنیم:

```cpp
constexpr int getDefaultValue()
{
    return 100;
}

constinit int value = getDefaultValue();
```

در این حالت مقداردهی `value` می‌تواند constant initialization باشد.

نکته ظریف این مثال این است که `constexpr` و `constinit` می‌توانند در کنار یکدیگر استفاده شوند، اما نقش یکسانی ندارند.

کلمه `constexpr` درباره قابلیت ارزیابی تابع در زمان کامپایل است، در حالی که `constinit` روی خود متغیر تضمین می‌کند که initialization آن از نوع موردنظر باشد.

---

# مثال پیشرفته با متغیر thread_local

کلمه `constinit` فقط برای متغیرهای global معمولی نیست.

متغیرهای `thread_local` نیز می‌توانند از آن استفاده کنند:

```cpp
constinit thread_local int threadCounter = 0;
```

در اینجا هر thread نسخه مخصوص خودش از `threadCounter` را دارد.

مقدار اولیه این متغیر نیز باید constant initialization داشته باشد.

سپس هر thread می‌تواند مقدار خودش را تغییر دهد:

```cpp
++threadCounter;
```

---

# مثال پیشرفته با static member

متغیرهای static member نیز می‌توانند در طراحی‌های مناسب از `constinit` استفاده کنند.

مثلاً:

```cpp
struct Config
{
    static inline int timeout;
};

constinit int Config::timeout = 30;
```

با این حال در کدهای مدرن C++ باید به تفاوت بین `inline static` و تعریف‌های جداگانه static member نیز توجه کرد.

نکته اصلی این است که `constinit` زمانی معنا پیدا می‌کند که متغیر موردنظر storage duration مناسب داشته باشد.

---

# مثال مهم؛ جلوگیری از dynamic initialization

فرض کنید تابعی داریم که در زمان اجرای برنامه مقداری را تولید می‌کند:

```cpp
int loadConfiguration()
{
    // خواندن تنظیمات
    return 100;
}

int configuration = loadConfiguration();
```

در اینجا `configuration` ممکن است به dynamic initialization نیاز داشته باشد.

اگر برنامه‌نویس انتظار داشته باشد این متغیر حتماً قبل از شروع execution عادی برنامه مقداردهی شده باشد، بهتر است بررسی کند آیا مقدار واقعاً می‌تواند constant initialization باشد یا خیر.

اگر تابع قابل ارزیابی در زمان کامپایل باشد:

```cpp
consteval int defaultConfiguration()
{
    return 100;
}
```

می‌توان نوشت:

```cpp
constinit int configuration = defaultConfiguration();
```

در این حالت هم `consteval` و هم `constinit` نقش مشخص خودشان را ایفا می‌کنند.

کلمه `consteval` تضمین می‌کند که تابع در زمان کامپایل ارزیابی شود.

کلمه `constinit` تضمین می‌کند که متغیر دارای constant initialization باشد.

---

# یک مثال پیشرفته با static initialization

فرض کنید یک کتابخانه چنین متغیری دارد:

```cpp
// config.hpp

extern constinit int globalTimeout;
```

و در فایل پیاده‌سازی داریم:

```cpp
// config.cpp

constinit int globalTimeout = 30;
```

اکنون سایر بخش‌های برنامه می‌توانند از آن استفاده کنند:

```cpp
#include "config.hpp"

void resetTimeout()
{
    globalTimeout = 30;
}
```

مزیت این الگو این است که قرارداد مهمی درباره initialization متغیر در سطح declaration بیان شده است.

یعنی `constinit` فقط یک optimization hint نیست.

کلمه `constinit` یک الزام زبانی است.

اگر initializer نتواند constant initialization باشد، کامپایلر باید برنامه را رد کند.

---

# constinit فقط یک optimization نیست

یکی از اشتباهات رایج این است که تصور کنیم:

> `constinit` فقط به کامپایلر می‌گوید کد را سریع‌تر اجرا کن.

این برداشت دقیق نیست.

کلمه `constinit` یک **compile-time requirement** ایجاد می‌کند.

مثلاً:

```cpp
int getValue();

constinit int value = getValue();
```

کامپایلر نباید این کد را صرفاً به این دلیل که شاید بتواند somehow آن را بهینه کند قبول کند.

مسئله این است که initializer باید شرایط constant initialization را داشته باشد.

اگر نداشته باشد، برنامه ill-formed است.

---

# اگر constinit وجود نداشت چه مشکلی داشتیم؟

قبل از C++20، زبان ابزاری با معنای دقیق `constinit` نداشت که به‌صورت مستقیم بگوید:

> این متغیر mutable باشد، اما initialization اولیه آن حتماً constant initialization باشد.

می‌توانستیم از `constexpr` استفاده کنیم، اما `constexpr` متغیر را immutable می‌کند.

می‌توانستیم از `const` استفاده کنیم، اما `const` به‌تنهایی چنین تضمینی درباره initialization موردنظر ما ایجاد نمی‌کند.

می‌توانستیم از تکنیک‌های مختلف طراحی برای جلوگیری از initialization order problem استفاده کنیم، اما زبان ابزار مستقیمی برای بیان این قرارداد نداشت.

بنابراین `constinit` یک شکاف مشخص را پر می‌کند.

این ویژگی به برنامه‌نویس اجازه می‌دهد **قابل‌تغییر بودن متغیر** را از **نحوه initialization اولیه آن** جدا کند.

---

# یک مثال برای درک تفاوت اصلی

چهار declaration زیر را کنار هم قرار دهید:

```cpp
const int a = 10;

constexpr int b = 10;

constinit int c = 10;

consteval int getValue()
{
    return 10;
}
```

اکنون هرکدام مفهوم متفاوتی دارند.

متغیر `a` بعد از initialization قابل تغییر نیست.

متغیر `b` نیز ثابت است و مقدار آن می‌تواند در constant expression استفاده شود.

متغیر `c` قابل تغییر است، اما initialization آن باید constant initialization باشد.

تابع `getValue` نیز یک immediate function است و فراخوانی آن باید در زمان کامپایل ارزیابی شود.

بنابراین شباهت اسمی این کلمات نباید باعث شود آن‌ها را جایگزین یکدیگر بدانیم.

---

# محدودیت‌های constinit

کلمه `constinit` برای هر متغیری قابل استفاده نیست.

متغیر باید دارای **static storage duration** یا **thread storage duration** باشد.

بنابراین استفاده‌ای مانند نمونه زیر مناسب نیست:

```cpp
void function()
{
    constinit int value = 10;
}
```

متغیر محلی معمولی چنین storage durationای ندارد.

در مقابل، این نمونه از نظر storage duration مناسب است:

```cpp
constinit int globalValue = 10;
```

همچنین نمونه زیر نیز می‌تواند مناسب باشد:

```cpp
thread_local constinit int threadValue = 10;
```

نکته مهم دیگر این است که `constinit` خودش متغیر را constant expression نمی‌کند.

مثلاً نباید تصور کنیم:

```cpp
constinit int value = 10;
```

همیشه در هر جایی معادل این است:

```cpp
constexpr int value = 10;
```

این دو declaration از نظر معنای زبان متفاوت هستند.

---

# یک نکته مهم درباره constinit و constexpr

کلمه `constexpr` در تعریف متغیر معمولاً باعث می‌شود متغیر `const` باشد.

مثلاً:

```cpp
constexpr int value = 10;
```

در مقابل:

```cpp
constinit int value = 10;
```

متغیر را mutable نگه می‌دارد.

پس اگر نیاز ما این باشد که:

> مقدار هرگز تغییر نکند و مقدار در compile-time قابل استفاده باشد.

معمولاً `constexpr` انتخاب طبیعی‌تری است.

اگر نیاز ما این باشد که:

> مقدار اولیه حتماً constant initialization باشد، اما متغیر بعداً قابل تغییر بماند.

در این حالت `constinit` ابزار مناسب‌تری است.

---

# یک نکته مهم درباره constinit و const

ترکیب این دو نیز امکان‌پذیر است:

```cpp
constinit const int value = 42;
```

در اینجا دو قرارداد مختلف داریم.

کلمه `const` می‌گوید متغیر بعد از initialization قابل تغییر نیست.

کلمه `constinit` می‌گوید initialization باید constant initialization باشد.

در نتیجه:

```cpp
constinit const int value = 42;
```

از نظر مفهومی چیزی شبیه این قرارداد را بیان می‌کند:

> مقدار اولیه ثابت باشد و خود متغیر نیز بعداً قابل تغییر نباشد.

البته در بسیاری از کاربردها `constexpr` انتخاب ساده‌تر و بیانگرتری برای چنین سناریویی است.

---

# چه زمانی از constinit استفاده کنیم؟

استفاده از `constinit` زمانی ارزشمند است که متغیری با static یا thread storage duration داریم و می‌خواهیم یک invariant مهم درباره initialization آن ایجاد کنیم.

سناریوی مناسب معمولاً چنین ویژگی‌هایی دارد:

```text
متغیر global یا thread_local است
        +
مقدار اولیه باید constant initialization باشد
        +
خود متغیر باید بعداً قابل تغییر باشد
```

در چنین شرایطی `constinit` گزینه بسیار خوبی است.

سناریوی زیر نیز مناسب نیست:

```text
من فقط یک مقدار ثابت می‌خواهم
```

در چنین شرایطی معمولاً `constexpr` انتخاب بهتری است.

سناریوی دیگری که مناسب نیست:

```text
من فقط می‌خواهم مقدار متغیر بعداً تغییر نکند
```

در چنین شرایطی `const` ممکن است کافی باشد.

سناریوی دیگری که داریم:

```text
من می‌خواهم یک تابع حتماً در زمان کامپایل اجرا شود
```

در اینجا باید سراغ `consteval` برویم.

---

# جمع‌بندی

کلمه `constinit` یکی از ابزارهای مهم C++20 برای کنترل initialization متغیرهای دارای static یا thread storage duration است.

هدف اصلی آن **جلوگیری از dynamic initialization ناخواسته** و ایجاد یک الزام صریح برای constant initialization است.

مهم‌ترین نکته این است که `constinit` به معنی `const` بودن متغیر نیست.

مثلاً:

```cpp
constinit int counter = 0;

counter++;
```

کد بالا کاملاً معتبر است.

در مقابل، `constexpr` برای اشیایی مناسب است که باید در قالب constant expression قابل استفاده باشند و متغیر `constexpr` نیز immutable است.

کلمه `consteval` نیز اساساً موضوع متفاوتی دارد و برای immediate functionها استفاده می‌شود.

در یک نگاه نهایی می‌توان این چهار مفهوم را این‌گونه به خاطر سپرد:

```text
const
    «بعداً تغییرم نده»

constexpr
    «مقدارم را می‌توان در compile-time استفاده کرد»

consteval
    «این تابع باید در compile-time اجرا شود»

constinit
    «initialization من باید constant initialization باشد»
```

پس اگر با یک متغیر global یا `thread_local` روبه‌رو شدید که **باید مقداردهی اولیه آن به‌صورت constant initialization انجام شود، اما خود متغیر باید بعداً قابل تغییر باشد**، `constinit` دقیقاً برای چنین نیازی طراحی شده است.

🤝

## مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>