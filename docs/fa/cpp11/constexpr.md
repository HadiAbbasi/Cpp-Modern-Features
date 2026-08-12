<div align="right">

[🇺🇸 English](../../en/cpp11/constexpr.md) | [🇮🇷 فارسی](./constexpr.md)

</div>

---
# آموزش `constexpr` در C++؛ از مفاهیم پایه تا کاربردهای پیشرفته

## فهرست مطالب

* [مقدمه؛ `constexpr` چیست؟](#مقدمه-constexpr-چیست)
* [نیاز به `constexpr` از کجا به وجود آمد؟](#نیاز-به-constexpr-از-کجا-به-وجود-آمد)
* [تفاوت محاسبه در زمان کامپایل و زمان اجرا](#تفاوت-محاسبه-در-زمان-کامپایل-و-زمان-اجرا)
* [متغیر `constexpr` چیست؟](#متغیر-constexpr-چیست)
* [آیا مقدار `constexpr` را می‌توان تغییر داد؟](#آیا-مقدار-constexpr-را-می‌توان-تغییر-داد)
* [اگر برای `constexpr` مقدار اولیه تعیین نکنیم چه می‌شود؟](#اگر-برای-constexpr-مقدار-اولیه-تعیین-نکنیم-چه-می‌شود)
* [آیا می‌توان تابع `constexpr` تعریف کرد؟](#آیا-می‌توان-تابع-constexpr-تعریف-کرد)
* [آیا ورودی تابع `constexpr` باید حتماً `constexpr` باشد؟](#آیا-ورودی-تابع-constexpr-باید-حتماً-constexpr-باشد)
* [مزیت استفاده از `constexpr` در ورودی تابع](#مزیت-استفاده-از-constexpr-در-ورودی-تابع)
* [تفاوت `const` و `constexpr`](#تفاوت-const-و-constexpr)
* [تفاوت `constexpr` و `consteval`](#تفاوت-constexpr-و-consteval)
* [تفاوت `constexpr` و `constinit`](#تفاوت-constexpr-و-constinit)
* [مقایسه کامل `const`، `constexpr`، `consteval` و `constinit`](#مقایسه-کامل)
* [مثال ترکیبی؛ کنار هم قرار دادن مفاهیم](#مثال-ترکیبی)
* [مثال پیشرفته؛ انتخاب زمان محاسبه](#مثال-پیشرفته)
* [نکات مهم و اشتباهات رایج](#نکات-مهم-و-اشتباهات-رایج)
* [جمع‌بندی](#جمع‌بندی)

---

## مقدمه؛ `constexpr` چیست؟

`constexpr` یکی از قابلیت‌های مهم زبان C++ است که به برنامه‌نویس اجازه می‌دهد مشخص کند یک مقدار یا یک تابع **می‌تواند در زمان کامپایل محاسبه شود**.

عبارت «می‌تواند» در این تعریف بسیار مهم است، زیرا `constexpr` لزوماً به این معنی نیست که هر بار استفاده از آن حتماً در زمان کامپایل اجرا می‌شود.

مثلاً کد زیر را در نظر بگیرید:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

تابع `square` یک تابع `constexpr` است.

عبارت زیر می‌تواند در زمان کامپایل محاسبه شود:

```cpp
constexpr int x = square(10);
```

در این حالت کامپایلر می‌تواند مقدار `100` را هنگام کامپایل محاسبه کند.

اما همین تابع می‌تواند با مقداری که فقط در زمان اجرا مشخص می‌شود نیز فراخوانی شود:

```cpp
int n;
std::cin >> n;

int result = square(n);
```

در این حالت مقدار `n` هنگام کامپایل مشخص نیست، بنابراین محاسبه در زمان اجرا انجام می‌شود.

پس می‌توان `constexpr` را به‌صورت ساده این‌گونه در نظر گرفت:

> `constexpr` به کامپایلر اجازه می‌دهد یک مقدار یا محاسبه را در زمان کامپایل انجام دهد، هر زمان که شرایط لازم وجود داشته باشد.

---

## نیاز به `constexpr` از کجا به وجود آمد؟

زبان C++ از ابتدا تلاش می‌کرد بین **کارایی بالا** و **انعطاف‌پذیری** تعادل برقرار کند.

در بسیاری از برنامه‌ها بعضی محاسبات هستند که نتیجه آن‌ها از قبل قابل تعیین است.

مثلاً فرض کنید چنین تابعی داریم:

```cpp
int square(int x)
{
    return x * x;
}
```

اگر بنویسیم:

```cpp
int x = square(10);
```

از دید منطقی کامپایلر می‌داند که نتیجه `100` است.

اما زبان باید مشخص می‌کرد که آیا چنین تابعی اجازه دارد در contextهایی استفاده شود که مقدار باید در زمان کامپایل مشخص باشد یا خیر.

مثلاً اندازه آرایه یا بعضی پارامترهای template به مقادیر compile-time نیاز دارند.

در نسخه‌های قدیمی C++ امکانات محدودی برای بیان این مفهوم وجود داشت.

قابلیت `constexpr` در C++11 معرفی شد تا برنامه‌نویس بتواند صراحتاً مشخص کند که یک مقدار یا تابع **قابلیت محاسبه در زمان کامپایل** دارد.

---

## اگر `constexpr` وجود نداشت چه مشکلی به وجود می‌آمد؟

نبودن `constexpr` باعث نمی‌شد C++ نتواند هیچ محاسبه‌ای را در زمان کامپایل انجام دهد.

کامپایلرها سال‌ها قبل از `constexpr` نیز بهینه‌سازی‌هایی مانند **constant folding** انجام می‌دادند.

مثلاً کامپایلر ممکن است چنین عبارتی را:

```cpp
int x = 10 * 20;
```

به‌صورت مؤثر به مقدار `200` تبدیل کند.

اما مشکل اصلی این بود که برنامه‌نویس نمی‌توانست به‌شکل استاندارد و قابل اتکا به زبان بگوید:

> «این مقدار یا تابع از نظر معنایی باید قابلیت ارزیابی در زمان کامپایل داشته باشد.»

تفاوت بسیار مهم همین‌جاست.

`constexpr` فقط یک درخواست برای بهینه‌سازی نیست؛ بلکه بخشی از **قواعد معنایی زبان C++** است.

---

## تفاوت محاسبه در زمان کامپایل و زمان اجرا

برای درک `constexpr` باید ابتدا تفاوت دو مفهوم را بشناسیم.

محاسبه‌ای که در **زمان کامپایل** انجام می‌شود، قبل از اجرای برنامه صورت می‌گیرد.

محاسبه‌ای که در **زمان اجرا** انجام می‌شود، هنگام اجرای برنامه توسط CPU انجام می‌شود.

مثلاً:

```cpp
constexpr int value = 10 * 20;
```

در این مثال مقدار `200` از همان زمان کامپایل مشخص است.

اما در مثال زیر مقدار فقط هنگام اجرای برنامه مشخص می‌شود:

```cpp
int x;
std::cin >> x;

int value = x * 20;
```

در اینجا کامپایلر نمی‌تواند مقدار `x` را از قبل بداند.

---

## متغیر `constexpr` چیست؟

متغیر `constexpr` متغیری است که مقدار اولیه آن باید یک **constant expression** معتبر باشد.

مثلاً:

```cpp
constexpr int max_users = 100;
```

در اینجا مقدار `100` در زمان کامپایل مشخص است.

مثال دیگر:

```cpp
constexpr int x = 10;
constexpr int y = x * 2;
```

در اینجا مقدار `y` نیز در زمان کامپایل قابل محاسبه است.

مثال زیر نیز معتبر است:

```cpp
constexpr int square(int x)
{
    return x * x;
}

constexpr int result = square(5);
```

کامپایلر می‌تواند مقدار `result` را برابر `25` قرار دهد.

---

## آیا مقدار `constexpr` را می‌توان تغییر داد؟

خیر.

متغیر `constexpr` قابل تغییر نیست.

مثلاً:

```cpp
constexpr int x = 10;

x = 20; // خطا
```

دلیل این رفتار این است که یک متغیر `constexpr` عملاً یک **constant** است.

به‌عبارت دقیق‌تر، یک متغیر `constexpr` دارای ویژگی `const` نیز هست.

بنابراین:

```cpp
constexpr int x = 10;
```

از نظر constness عملاً مانند یک مقدار ثابت رفتار می‌کند.

---

## اگر برای `constexpr` مقدار اولیه تعیین نکنیم چه می‌شود؟

متغیر `constexpr` باید در همان زمان تعریف مقداردهی شود.

بنابراین کد زیر صحیح نیست:

```cpp
constexpr int x; // خطا
```

کامپایلر نمی‌تواند متغیر `constexpr` را بدون initializer بپذیرد.

کد صحیح به شکل زیر است:

```cpp
constexpr int x = 10;
```

این موضوع یکی از تفاوت‌های مهم `constexpr` با یک متغیر عادی است.

حتی `const` نیز برای یک شیء عادی معمولاً باید هنگام تعریف مقداردهی شود:

```cpp
const int x = 10;
```

اما نکته مهم این است که `constexpr` علاوه بر ثابت بودن، الزام قوی‌تری درباره **قابل محاسبه بودن مقدار در زمان کامپایل** دارد.

---

## آیا می‌توان متغیر یا تابع `constexpr` تعریف کرد؟

بله.

`constexpr` می‌تواند برای چند نوع entity در C++ استفاده شود که مهم‌ترین آن‌ها متغیرها و توابع هستند.

مثلاً برای متغیر:

```cpp
constexpr int size = 100;
```

و برای تابع:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

همچنین `constexpr` در طراحی کلاس‌ها و constructorها نیز کاربرد دارد:

```cpp
class Point
{
public:
    constexpr Point(int x, int y)
        : x_(x), y_(y)
    {
    }

    constexpr int x() const
    {
        return x_;
    }

    constexpr int y() const
    {
        return y_;
    }

private:
    int x_;
    int y_;
};
```

اکنون می‌توان یک شیء را نیز در contextهای compile-time ساخت:

```cpp
constexpr Point p(10, 20);

static_assert(p.x() == 10);
static_assert(p.y() == 20);
```

---

## آیا تابع `constexpr` حتماً در زمان کامپایل اجرا می‌شود؟

خیر.

این یکی از مهم‌ترین نکات `constexpr` است.

فرض کنید تابع زیر را داریم:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

اگر ورودی compile-time باشد:

```cpp
constexpr int a = square(10);
```

محاسبه می‌تواند در زمان کامپایل انجام شود.

اما اگر ورودی runtime باشد:

```cpp
int n;

std::cin >> n;

int a = square(n);
```

همان تابع می‌تواند در زمان اجرا اجرا شود.

پس `constexpr` به این معنی نیست که تابع **مجبور است همیشه compile-time اجرا شود**.

معنای دقیق‌تر این است:

> تابع `constexpr` باید طوری تعریف شده باشد که در صورت فراهم بودن شرایط، بتواند به‌عنوان یک constant expression ارزیابی شود.

---

## آیا ورودی تابع `constexpr` باید حتماً `constexpr` باشد؟

خیر.

این تصور یکی از اشتباهات رایج درباره `constexpr` است.

تابع زیر را داریم:

```cpp
constexpr int multiply(int a, int b)
{
    return a * b;
}
```

می‌توانیم با مقادیر `constexpr` از آن استفاده کنیم:

```cpp
constexpr int a = 10;
constexpr int b = 20;

constexpr int result = multiply(a, b);
```

در این حالت نتیجه نیز compile-time است.

اما می‌توانیم از متغیرهای معمولی نیز استفاده کنیم:

```cpp
int a = 10;
int b = 20;

int result = multiply(a, b);
```

این کد کاملاً معتبر است.

حتی اگر مقدار متغیر در runtime تغییر کند، تابع همچنان قابل استفاده است:

```cpp
int a;

std::cin >> a;

int result = multiply(a, 20);
```

در این حالت `multiply` در runtime اجرا می‌شود.

---

## مزیت استفاده از `constexpr` در ورودی تابع

اگر ورودی تابع از قبل مشخص باشد، استفاده از `constexpr` می‌تواند امکان محاسبه در زمان کامپایل را فراهم کند.

مثلاً:

```cpp
constexpr int cube(int x)
{
    return x * x * x;
}

constexpr int value = cube(4);
```

کامپایلر می‌تواند نتیجه را از قبل محاسبه کند:

```text
value = 64
```

این قابلیت در مواردی مانند موارد زیر اهمیت زیادی دارد:

* محاسبات ریاضی ثابت
* اندازه آرایه‌ها
* پارامترهای template
* `static_assert`
* ساختارهای داده ثابت
* تولید lookup table در زمان کامپایل
* محاسبات متادیتا
* برنامه‌نویسی template
* بعضی کاربردهای embedded و سیستم‌های حساس به performance

---

## اگر ورودی `constexpr` نباشد چه می‌شود؟

اگر ورودی در runtime مشخص شود، دیگر نتیجه نمی‌تواند به‌عنوان constant expression استفاده شود.

مثلاً:

```cpp
constexpr int square(int x)
{
    return x * x;
}

int n;
std::cin >> n;

constexpr int result = square(n); // خطا
```

دلیل خطا این است که `n` در زمان کامپایل مقدار مشخصی ندارد.

اما اگر `constexpr` را از نتیجه برداریم:

```cpp
int result = square(n);
```

کد کاملاً صحیح است.

پس مشکل، `constexpr` بودن تابع نیست.

مشکل این است که **constant expression بودن نتیجه به ورودی‌های آن وابسته است**.

---

## یک مثال مهم؛ یک تابع، دو نوع استفاده

تابع زیر را در نظر بگیرید:

```cpp
constexpr int factorial(int n)
{
    if (n <= 1)
        return 1;

    return n * factorial(n - 1);
}
```

اکنون می‌توانیم دو نوع استفاده داشته باشیم.

استفاده اول در زمان کامپایل:

```cpp
constexpr int a = factorial(5);
```

استفاده دوم در زمان اجرا:

```cpp
int n;

std::cin >> n;

int b = factorial(n);
```

بنابراین یک تابع `constexpr` می‌تواند هم **compile-time** و هم **runtime** باشد.

---

## تفاوت `const` و `constexpr`

مهم‌ترین تفاوت این دو مفهوم در این است که `const` درباره **قابل تغییر نبودن** صحبت می‌کند، در حالی که `constexpr` درباره **قابل ارزیابی بودن در زمان کامپایل** نیز الزام ایجاد می‌کند.

مثلاً:

```cpp
int get_value()
{
    return 42;
}

const int a = get_value();
```

این کد می‌تواند کاملاً معتبر باشد.

مقدار `a` بعد از مقداردهی قابل تغییر نیست، اما الزاماً یک constant expression نیست.

اما:

```cpp
constexpr int b = 42;
```

مقدار `b` باید در زمان کامپایل قابل محاسبه باشد.

می‌توان این تفاوت را این‌گونه خلاصه کرد:

```text
const      → بعد از مقداردهی قابل تغییر نیست
constexpr  → مقدار باید compile-time قابل محاسبه باشد
```

در نتیجه:

```cpp
const int a = 10;
constexpr int b = 10;
```

هر دو تغییرناپذیر هستند، اما `b` تضمین قوی‌تری درباره compile-time بودن مقدار دارد.

---

## یک مثال برای تفاوت واقعی `const` و `constexpr`

تابع زیر فقط در runtime اجرا می‌شود:

```cpp
int get_number()
{
    return 42;
}
```

اکنون:

```cpp
const int a = get_number();
```

مقدار `a` ثابت است، اما `get_number()` برای تولید آن باید اجرا شود.

اما این کد:

```cpp
constexpr int b = get_number();
```

معتبر نیست، زیرا `get_number()` یک تابع معمولی است و نمی‌تواند در این context به‌عنوان constant expression استفاده شود.

اگر تابع را `constexpr` کنیم:

```cpp
constexpr int get_number()
{
    return 42;
}
```

اکنون:

```cpp
constexpr int b = get_number();
```

معتبر است.

---

## تفاوت `constexpr` و `consteval`

تفاوت `constexpr` و `consteval` بسیار مهم است.

تابع `constexpr` **می‌تواند** در زمان کامپایل اجرا شود.

تابع `consteval` **باید** در زمان کامپایل اجرا شود.

مثلاً:

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

این تابع هم compile-time و هم runtime قابل استفاده است.

اما:

```cpp
consteval int square(int x)
{
    return x * x;
}
```

فراخوانی این تابع باید نتیجه‌ای داشته باشد که در زمان کامپایل قابل محاسبه باشد.

مثلاً:

```cpp
constexpr int a = square(10);
```

معتبر است.

اما:

```cpp
int n;

std::cin >> n;

int a = square(n); // خطا
```

نام `consteval` به این دلیل انتخاب شده است که مفهوم **constant evaluation اجباری** را بیان می‌کند.

---

## تفاوت مفهومی `constexpr` و `consteval`

می‌توان این دو را این‌گونه به خاطر سپرد:

```text
constexpr → اگر امکانش باشد، compile-time
consteval → حتماً compile-time
```

در نتیجه اگر می‌خواهیم یک API به‌گونه‌ای طراحی شود که **هیچ‌وقت runtime اجرا نشود**، `consteval` ابزار مناسب‌تری است.

مثلاً:

```cpp
consteval int square(int x)
{
    return x * x;
}
```

این ویژگی می‌تواند برای ساخت ابزارهای compile-time بسیار قدرتمند باشد.

---

## تفاوت `constexpr` و `constinit`

`constinit` مفهوم متفاوتی دارد.

`constinit` برای متغیرهایی با **static storage duration** یا **thread storage duration** استفاده می‌شود و هدف اصلی آن اطمینان از **constant initialization** است.

مثلاً:

```cpp
constinit int global_value = 100;
```

این متغیر همچنان می‌تواند تغییر کند:

```cpp
global_value = 200;
```

بنابراین:

```cpp
constinit int x = 10;
x = 20; // معتبر
```

اما:

```cpp
constexpr int x = 10;
x = 20; // خطا
```

پس `constinit` به معنی immutable بودن نیست.

---

## چرا `constinit` به وجود آمد؟

یکی از مسائل مهم در C++ مسئله **ترتیب initialization اشیای global** در translation unitهای مختلف است.

فرض کنید چند متغیر global داریم که initialization آن‌ها به یکدیگر وابسته است.

در بعضی شرایط ممکن است initialization یک متغیر قبل یا بعد از متغیر دیگری اتفاق بیفتد و اصطلاحاً با مشکلاتی مانند **Static Initialization Order Fiasco** مواجه شویم.

`constinit` کمک می‌کند مطمئن شویم متغیر موردنظر به‌صورت **constant initialization** مقداردهی می‌شود.

مثلاً:

```cpp
constinit int value = 42;
```

در اینجا هدف اصلی `constinit` این نیست که `value` را ثابت کند.

هدف این است که initialization آن در مرحله مناسب و به‌صورت constant initialization انجام شود.

---

## تفاوت مهم `constexpr` و `constinit`

مثال زیر را ببینید:

```cpp
constexpr int a = 10;
```

در این حالت `a` هم compile-time constant است و هم قابل تغییر نیست.

اما:

```cpp
constinit int b = 10;
```

در این حالت `b` می‌تواند تغییر کند:

```cpp
b = 20;
```

بنابراین این دو keyword هدف یکسانی ندارند.

```text
constexpr → constant value / constant expression
constinit  → constant initialization
```

---

## مقایسه کامل

| ویژگی                         | `const`                 | `constexpr`             | `consteval`                      | `constinit`                 |
| ----------------------------- | ----------------------- | ----------------------- | -------------------------------- | --------------------------- |
| جلوگیری از تغییر مقدار        | بله                     | بله                     | برای تابع مطرح است               | خیر                         |
| الزام compile-time بودن مقدار | خیر                     | بله                     | بله                              | الزام برای initialization   |
| برای متغیر                    | بله                     | بله                     | خیر                              | بله                         |
| برای تابع                     | خیر                     | بله                     | بله                              | خیر                         |
| اجرای runtime                 | بله                     | بله، در صورت نیاز       | خیر                              | مربوط به initialization است |
| مقداردهی اولیه لازم           | برای object معمولاً بله | بله                     | تابع باید تعریف مناسب داشته باشد | بله                         |
| هدف اصلی                      | تغییرناپذیری            | compile-time evaluation | اجبار به compile-time evaluation | constant initialization     |

---

## یک مثال ترکیبی

اکنون چهار مفهوم را کنار هم قرار می‌دهیم:

```cpp
constexpr int compile_time_value = 10;

const int runtime_constant = get_value();

consteval int force_compile_time(int x)
{
    return x * 2;
}

constinit int global_value = 100;
```

در این مثال هر keyword یک نقش متفاوت دارد.

متغیر `compile_time_value` یک مقدار ثابت compile-time است.

متغیر `runtime_constant` بعد از مقداردهی قابل تغییر نیست، اما مقدار آن الزاماً compile-time نیست.

تابع `force_compile_time` باید در compile-time قابل ارزیابی شود.

متغیر `global_value` باید constant initialization داشته باشد، ولی خودش immutable نیست.

---

## مثال پیشرفته؛ ترکیب `constexpr` با `static_assert`

یکی از کاربردهای بسیار مهم `constexpr` استفاده همراه با `static_assert` است.

مثلاً:

```cpp
constexpr int square(int x)
{
    return x * x;
}

constexpr int value = square(5);

static_assert(value == 25);
```

در اینجا `static_assert` نیز در زمان کامپایل بررسی می‌شود.

اگر مقدار اشتباه باشد:

```cpp
static_assert(value == 30);
```

کامپایل با خطا مواجه می‌شود.

این ویژگی برای ساخت نرم‌افزارهایی که باید برخی شروط آن‌ها در compile-time بررسی شود بسیار مفید است.

---

## مثال پیشرفته؛ استفاده از `constexpr` در template

یکی از کاربردهای قدرتمند `constexpr` در کنار templateها است.

مثلاً:

```cpp
constexpr int square(int x)
{
    return x * x;
}

template<int Size>
class Buffer
{
    char data[Size];
};

Buffer<square(4)> buffer;
```

در اینجا `square(4)` باید به مقداری تبدیل شود که در زمان کامپایل مشخص باشد.

بنابراین نتیجه `square(4)` برابر `16` می‌شود و کلاس عملاً به شکل زیر instantiate خواهد شد:

```cpp
Buffer<16> buffer;
```

این مثال نشان می‌دهد که `constexpr` فقط یک ابزار برای سریع‌تر کردن کد نیست.

این قابلیت می‌تواند بخشی از **ساختار برنامه در زمان کامپایل** باشد.

---

## مثال پیشرفته؛ محاسبه آرایه در زمان کامپایل

فرض کنید می‌خواهیم یک جدول ثابت تولید کنیم.

```cpp
constexpr int square(int x)
{
    return x * x;
}

constexpr int table_size = 10;

constexpr auto create_table()
{
    std::array<int, table_size> result{};

    for (int i = 0; i < table_size; ++i)
        result[i] = square(i);

    return result;
}

constexpr auto table = create_table();
```

در استانداردهای جدید C++ قابلیت‌های `constexpr` بسیار گسترده‌تر شده‌اند و بسیاری از عملیات‌هایی که در نسخه‌های قدیمی فقط در runtime ممکن بودند، اکنون می‌توانند در constant evaluation انجام شوند.

البته قابلیت دقیق یک تابع `constexpr` به نسخه استاندارد C++ نیز وابسته است.

---

## مثال پیشرفته؛ `constexpr` برای اشیاء

`constexpr` فقط برای `int` و انواع ساده نیست.

اگر نوعی قابلیت لازم برای constant evaluation را داشته باشد، می‌توان objectهای آن را نیز به‌صورت compile-time ساخت.

مثلاً:

```cpp
struct Point
{
    int x;
    int y;

    constexpr int distance_squared() const
    {
        return x * x + y * y;
    }
};

constexpr Point p{3, 4};

constexpr int distance = p.distance_squared();

static_assert(distance == 25);
```

در این مثال خود `Point` نیز در زمان کامپایل ساخته شده است.

---

## آیا `constexpr` همیشه باعث سریع‌تر شدن برنامه می‌شود؟

نه لزوماً.

هدف اصلی `constexpr` این نیست که صرفاً runtime performance را افزایش دهد.

در بسیاری از موارد کامپایلر بدون `constexpr` نیز می‌تواند محاسبات ثابت را بهینه کند.

ارزش واقعی `constexpr` بیشتر در این است که بتوانیم **محاسبات را بخشی از compile-time computation برنامه کنیم**.

این موضوع می‌تواند باعث کاهش کار runtime شود، اما مهم‌تر از آن می‌تواند امکان استفاده از نتیجه در contextهایی را فراهم کند که به constant expression نیاز دارند.

بنابراین این دو جمله را نباید یکی دانست:

```text
constexpr = بهینه‌سازی
```

و:

```text
constexpr = قابلیت بیان و الزام compile-time evaluation
```

جمله دوم دقیق‌تر است.

---

## نکات مهم و اشتباهات رایج

### اشتباه اول؛ تصور اینکه `constexpr` همیشه در compile-time اجرا می‌شود

تابع زیر:

```cpp
constexpr int add(int a, int b)
{
    return a + b;
}
```

می‌تواند با ورودی runtime نیز استفاده شود:

```cpp
int a;
std::cin >> a;

int result = add(a, 10);
```

پس `constexpr` به‌تنهایی runtime execution را ممنوع نمی‌کند.

---

### اشتباه دوم؛ تصور اینکه ورودی تابع باید `constexpr` باشد

کد زیر کاملاً معتبر است:

```cpp
constexpr int add(int a, int b)
{
    return a + b;
}

int x = 10;
int y = 20;

int result = add(x, y);
```

در اینجا خود تابع `constexpr` است، اما آرگومان‌ها `constexpr` نیستند.

---

### اشتباه سوم؛ تصور اینکه `const` همان `constexpr` است

کد زیر:

```cpp
const int x = get_value();
```

فقط می‌گوید `x` بعد از مقداردهی تغییر نکند.

اما:

```cpp
constexpr int x = get_value();
```

علاوه بر ثابت بودن، نیاز دارد که `get_value()` بتواند در constant expression ارزیابی شود.

---

### اشتباه چهارم؛ تصور اینکه `consteval` همان `constexpr` است

تابع `constexpr` می‌تواند runtime اجرا شود.

تابع `consteval` باید در compile-time ارزیابی شود.

مثلاً:

```cpp
constexpr int f(int x)
{
    return x * 2;
}
```

همراه با:

```cpp
int x = 10;
int y = f(x);
```

مجاز است.

اما:

```cpp
consteval int f(int x)
{
    return x * 2;
}
```

در چنین استفاده‌ای نمی‌تواند runtime باشد.

---

### اشتباه پنجم؛ تصور اینکه `constinit` یعنی مقدار ثابت است

کد زیر کاملاً معتبر است:

```cpp
constinit int counter = 0;

counter++;
counter++;
```

در اینجا `counter` قابل تغییر است.

بنابراین `constinit` را نباید با `const` یا `constexpr` یکی دانست.

---

## یک مدل ذهنی ساده برای حفظ تفاوت‌ها

برای به خاطر سپردن این چهار keyword می‌توان چهار سؤال متفاوت مطرح کرد.

اگر سؤال این باشد:

> آیا بعد از مقداردهی می‌توانم این مقدار را تغییر بدهم؟

پاسخ مرتبط با:

```cpp
const
```

است.

اگر سؤال این باشد:

> آیا این مقدار باید یک constant expression باشد؟

پاسخ مرتبط با:

```cpp
constexpr
```

است.

اگر سؤال این باشد:

> آیا این تابع حتماً باید در compile-time اجرا شود؟

پاسخ مرتبط با:

```cpp
consteval
```

است.

اگر سؤال این باشد:

> آیا initialization این متغیر global/static باید constant initialization باشد؟

پاسخ مرتبط با:

```cpp
constinit
```

است.

---

## یک مثال نهایی برای مقایسه چهار مفهوم

فرض کنید چنین کدی داریم:

```cpp
int runtime_value()
{
    return 42;
}

constexpr int compile_time_value()
{
    return 42;
}

const int a = runtime_value();

constexpr int b = compile_time_value();

consteval int double_value(int x)
{
    return x * 2;
}

constinit int global_counter = 0;
```

در این مثال `a` یک مقدار ثابت است، اما تولید مقدار آن می‌تواند در runtime اتفاق افتاده باشد.

متغیر `b` یک مقدار compile-time ثابت است.

تابع `double_value` مجبور است هنگام فراخوانی در constant evaluation انجام شود.

متغیر `global_counter` نیز با constant initialization مقداردهی می‌شود، اما خودش قابل تغییر است.

---

## جمع‌بندی

`constexpr` یکی از ابزارهای کلیدی C++ مدرن برای انتقال بخشی از محاسبات از runtime به compile-time است.

متغیر `constexpr` باید مقدار اولیه‌ای داشته باشد که در زمان کامپایل قابل ارزیابی باشد و بعد از مقداردهی نیز قابل تغییر نیست.

تابع `constexpr` می‌تواند در compile-time یا runtime اجرا شود و این موضوع به context و ورودی‌های آن بستگی دارد.

تابع `consteval` یک قدم جلوتر می‌رود و compile-time evaluation را اجباری می‌کند.

کلیدواژه `constinit` نیز مسئله متفاوتی را حل می‌کند و روی **نحوه initialization متغیرهای static یا thread storage duration** تمرکز دارد، نه روی immutable بودن متغیر.

در نتیجه می‌توان این چهار مفهوم را در یک خط به خاطر سپرد:

```text
const      → تغییر نده
constexpr  → در صورت امکان/لزوم، compile-time
consteval  → حتماً compile-time
constinit  → initialization را compile-time و constant انجام بده
```

مهم‌ترین نکته نیز این است که `constexpr` را صرفاً یک **optimization hint** در نظر نگیریم.

`constexpr` یک قابلیت زبانی برای بیان این موضوع است که یک مقدار، تابع یا عملیات **قابلیت constant evaluation** دارد و همین قابلیت در C++ مدرن پایه بسیاری از تکنیک‌های template metaprogramming، `static_assert`، تولید داده در زمان کامپایل و طراحی APIهای compile-time است.

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>