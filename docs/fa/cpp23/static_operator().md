<div align="center">

[🇺🇸 English](../../en/cpp23/static_operator().md) | [🇮🇷 فارسی](./static_operator().md)

</div>

---

# قابلیت Static operator() در ++C23

## مقدمه

یکی از قابلیت‌های اضافه‌شده به استاندارد ++C23، امکان تعریف `operator()` به‌صورت `static` است. این قابلیت که با عنوان **Static Call Operator** شناخته می‌شود، به برنامه‌نویسان اجازه می‌دهد Function Call Operator را بدون نیاز به یک implicit object parameter تعریف کنند.

در نسخه‌های قبلی ++C، عملگر `operator()` همواره یک non-static member function بود. به همین دلیل، فراخوانی آن مستلزم وجود یک object از کلاس بود و تابع به‌صورت ضمنی به object جاری از طریق `this` دسترسی داشت.

اما در ++C23 می‌توان این عملگر را به‌صورت static تعریف کرد:

```
#include <iostream>

struct Calculator {
    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    Calculator calc;

    std::cout << calc(10, 20) << '\n';
    std::cout << Calculator{}(5, 7) << '\n';
}
```

در این مثال، عملگر `operator()` هیچ وابستگی‌ای به وضعیت یک instance ندارد و صرفاً دو ورودی را دریافت کرده و نتیجه را محاسبه می‌کند.

این قابلیت به‌ویژه در طراحی Stateless Function Objects، پیاده‌سازی Generic Callable Objects، استفاده از Lambda Expressions و کاهش وابستگی به object instanceها کاربرد دارد.

## مفهوم Static Call Operator

### تعریف operator() در ++C

عملگر `operator()` یکی از قابلیت‌های Operator Overloading در ++C است که امکان فراخوانی یک object مانند یک تابع معمولی را فراهم می‌کند.

برای مثال:

```
#include <iostream>

struct Printer {
    void operator()(int value) const {
        std::cout << value << '\n';
    }
};

int main() {
    Printer printer;

    printer(42);
}
```

در این مثال، عبارت زیر:

```
printer(42);
```

از نظر مفهومی معادل فراخوانی زیر است:

```
printer.operator()(42);
```

در واقع، کلاس `Printer` یک Function Object یا Functor ایجاد می‌کند.

در حالت معمول، عملگر `operator()` یک non-static member function است؛ بنابراین هر بار که فراخوانی می‌شود، object مربوطه به‌عنوان implicit object parameter در اختیار تابع قرار می‌گیرد.

### تغییر ایجادشده در ++C23

استاندارد ++C23 این محدودیت را برطرف کرده و اجازه می‌دهد عملگر `operator()` به‌صورت static تعریف شود.

```
struct Utility {
    static void operator()(int value) {
        // Process the value
    }
};
```

تفاوت اصلی این است که تابع دیگر به یک object مشخص وابسته نیست.

در نتیجه، موارد زیر درباره آن صدق می‌کنند:

- تابع دارای implicit `this` pointer نیست.
- تابع نمی‌تواند به non-static memberهای کلاس دسترسی مستقیم داشته باشد.
- فراخوانی تابع نیازی به وضعیت داخلی یک instance ندارد.
- تابع می‌تواند از طریق object یا با استفاده از class scope فراخوانی شود.
- تابع نمی‌تواند از cv-qualifier یا ref-qualifier مخصوص member functionها استفاده کند.

## دلیل اضافه‌شدن این قابلیت

### مشکل Functorهای بدون وضعیت

در بسیاری از طراحی‌های ++C، از Functorها برای پیاده‌سازی رفتارهای قابل فراخوانی استفاده می‌شود.

برای مثال:

```
struct Square {
    int operator()(int value) const {
        return value * value;
    }
};
```

کلاس `Square` هیچ stateای ندارد. تنها وظیفه آن دریافت یک ورودی و محاسبه مربع آن است.

با وجود این، عملگر `operator()` همچنان یک non-static member function است.

بنابراین برای استفاده از آن باید یک object ایجاد شود:

```
Square square;

int result = square(5);
```

در چنین شرایطی، وجود implicit object parameter از نظر منطقی ضرورتی ندارد؛ زیرا رفتار تابع به هیچ‌یک از memberهای instance وابسته نیست.

نسخه static این مسئله را برطرف می‌کند:

```
struct Square {
    static int operator()(int value) {
        return value * value;
    }
};

int main() {
    int result = Square{}(5);
}
```

### بهبود طراحی Stateless Function Objectها

یکی از اهداف اصلی این قابلیت، امکان بیان مستقیم این مفهوم در سطح زبان است که یک Function Object می‌تواند فاقد وابستگی به instance باشد.

این موضوع به بهبود طراحی کلاس‌هایی کمک می‌کند که صرفاً نقش یک Callable Interface را ایفا می‌کنند.

همچنین در برخی سناریوها، می‌توان از ایجاد وابستگی غیرضروری به object state جلوگیری کرد.

البته باید توجه داشت که static بودن عملگر به‌تنهایی تضمین‌کننده بهبود Performance نیست. کامپایلرها معمولاً قادر به بهینه‌سازی Functorهای بدون وضعیت نیز هستند.

## نحوه تعریف Static operator()

### ساختار کلی

برای تعریف Static Call Operator کافی است کلمه کلیدی `static` را قبل از `operator()` قرار دهید.

```
struct MyFunction {
    static ReturnType operator()(Parameters...);
};
```

یک نمونه عملی:

```
struct Multiplier {
    static int operator()(int a, int b) {
        return a * b;
    }
};
```

در این تعریف:

- `static` مشخص می‌کند که عملگر یک static member function است.
- `operator()` مشخص‌کننده Function Call Operator است.
- پارامترها ورودی‌های فراخوانی تابع هستند.
- مقدار بازگشتی تابع می‌تواند هر نوع معتبر مورد پشتیبانی زبان باشد.

### فراخوانی از طریق Object

مانند Function Objectهای معمولی، می‌توان Static Call Operator را از طریق یک object فراخوانی کرد.

```
#include <iostream>

struct Adder {
    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    Adder adder;

    std::cout << adder(10, 20) << '\n';
}
```

نکته مهم این است که وجود object در این روش صرفاً بخشی از syntax فراخوانی است و عملگر برای اجرای خود به instance آن وابسته نیست.

### فراخوانی با Class Scope

از آنجا که عملگر static است، می‌توان آن را از طریق نام کلاس نیز فراخوانی کرد:

```
#include <iostream>

struct Adder {
    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    int result = Adder::operator()(10, 20);

    std::cout << result << '\n';
}
```

این روش برای مواقعی مناسب است که قصد دارید صریحاً static بودن تابع و عدم وابستگی آن به instance را نشان دهید.

توجه کنید که عبارت `Adder::operator()` یک فراخوانی عادی تابع static است و نباید آن را با syntax معمول فراخوانی Functor اشتباه گرفت.

## تفاوت Static و Non-Static operator()

### بررسی رفتار دو نوع عملگر

مهم‌ترین تفاوت این دو نوع عملگر در وجود یا نبود implicit object parameter است.

در حالت non-static:

```
struct NormalFunctor {
    int factor = 10;

    int operator()(int value) const {
        return value * factor;
    }
};
```

عملگر به object جاری وابسته است و می‌تواند به memberهای non-static دسترسی داشته باشد.

اما در حالت static:

```
struct StaticFunctor {
    static int operator()(int value) {
        return value * 10;
    }
};
```

عملگر به instance وابسته نیست و نمی‌تواند مستقیماً از `this` یا memberهای non-static استفاده کند.

### مقایسه ویژگی‌ها

| ویژگی                               | Non-Static operator() | Static operator() |
| ----------------------------------- | --------------------- | ----------------- |
| وجود implicit object parameter      | دارد                  | ندارد             |
| دسترسی مستقیم به `this`             | دارد                  | ندارد             |
| دسترسی مستقیم به non-static members | دارد                  | ندارد             |
| دسترسی به static members            | دارد                  | دارد              |
| امکان استفاده از cv-qualifier       | دارد                  | ندارد             |
| امکان استفاده از ref-qualifier      | دارد                  | ندارد             |
| امکان virtual بودن                  | در شرایط مجاز دارد    | ندارد             |
| امکان فراخوانی از طریق object       | دارد                  | دارد              |
| امکان فراخوانی از طریق class scope  | ندارد                 | دارد              |
| نیاز به object برای اجرای رفتار     | دارد                  | ندارد             |

### مقایسه از نظر طراحی

زمانی که رفتار Functor به وضعیت داخلی object وابسته است، استفاده از non-static operator() انتخاب مناسبی است.

اما اگر عملگر صرفاً یک عملیات مستقل انجام می‌دهد و هیچ نیازی به instance state ندارد، نسخه static می‌تواند طراحی واضح‌تری ارائه دهد.

برای مثال:

```
struct StatefulMultiplier {
    int factor;

    explicit StatefulMultiplier(int value)
        : factor(value) {}

    int operator()(int value) const {
        return value * factor;
    }
};

struct StatelessMultiplier {
    static int operator()(int value) {
        return value * 10;
    }
};
```

کلاس اول برای انجام محاسبات به state داخلی وابسته است، در حالی که کلاس دوم یک رفتار ثابت و مستقل از instance ارائه می‌دهد.

## تفاوت Static operator() با Static Member Function

هر دو قابلیت از کلمه کلیدی `static` استفاده می‌کنند، اما Static Call Operator از نظر syntax و هدف کاربردی با یک static member function معمولی متفاوت است.

### تابع عضو Static معمولی

```
struct Math {
    static int square(int value) {
        return value * value;
    }
};
```

فراخوانی این تابع به شکل زیر است:

```
int result = Math::square(5);
```

### عملگر فراخوانی Static

```
struct MathFunctor {
    static int operator()(int value) {
        return value * value;
    }
};
```

فراخوانی آن به دو شکل امکان‌پذیر است:

```
int first = MathFunctor{}(5);

int second = MathFunctor::operator()(5);
```

تفاوت مهم در این است که `operator()` اجازه می‌دهد کلاس مانند یک Callable Object رفتار کند.

بنابراین Static Call Operator می‌تواند در APIهایی که از Callable Objects استفاده می‌کنند، جایگزین مناسب‌تری برای نام‌گذاری صریح توابع باشد.

## ارتباط با Lambda Expressions

یکی از مهم‌ترین کاربردهای مرتبط با Static Call Operator در ++C23، قابلیت Static Lambda است.

در نسخه‌های قبلی ++C، Lambda Expression بدون capture معمولاً به یک Closure Object تبدیل می‌شود که دارای یک `operator()` است.

برای مثال:

```
auto square = [](int value) {
    return value * value;
};
```

در این حالت، کامپایلر یک Closure Type با عملگر فراخوانی مناسب تولید می‌کند.

استاندارد ++C23 امکان اضافه‌کردن `static` به Lambda Declarator را فراهم کرده است.

### معرفی Static Lambda

```
auto square = [](int value) static {
    return value * value;
};

int result = square(5);
```

در این مثال، `operator()` مربوط به Closure Type به‌صورت static تعریف می‌شود.

این قابلیت برای Lambdaهایی مفید است که هیچ captureای ندارند و رفتار آن‌ها کاملاً مستقل از Closure Object است.

### محدودیت Capture در Static Lambda

Static Lambda نمی‌تواند از capture برای نگهداری وضعیت بیرونی استفاده کند.

برای مثال، تعریف زیر معتبر نیست:

```
int factor = 10;

auto multiply = [factor](int value) static {
    return value * factor;
};
```

دلیل آن این است که Static Call Operator به implicit object parameter دسترسی ندارد و بنابراین نمی‌تواند از state ذخیره‌شده در Closure Object استفاده کند.

برای چنین سناریویی باید از Lambda معمولی استفاده کنید:

```
int factor = 10;

auto multiply = [factor](int value) {
    return value * factor;
};
```

### Static Lambda بدون Capture

نمونه زیر یک Static Lambda معتبر است:

```
auto add = [](int a, int b) static {
    return a + b;
};

int result = add(10, 20);
```

همچنین می‌توان از آن برای تعریف Generic Lambda استفاده کرد.

## کاربرد Static در Generic Lambda

یکی از کاربردهای مفید این قابلیت، ترکیب Static Lambda با Template Parameters است.

### تعریف Generic Static Lambda

```
#include <concepts>

auto add = []<typename T>(T a, T b) static
    requires std::integral<T>
{
    return a + b;
};

int main() {
    auto result1 = add(10, 20);
    auto result2 = add(30L, 40L);
}
```

در این مثال:

- Lambda از Template Parameter استفاده می‌کند.
- عملگر فراخوانی Closure Type به‌صورت static تعریف شده است.
- با استفاده از `requires` فقط انواع integral مجاز هستند.

این روش برای طراحی عملیات عمومی و بدون state مناسب است.

### ترکیب با Concepts

استفاده از Concepts می‌تواند محدودیت‌های ورودی Static Lambda را به شکل واضح‌تری مشخص کند.

```
#include <concepts>
#include <type_traits>

auto equal = []<typename T>(const T& a, const T& b) static
    requires std::equality_comparable<T>
{
    return a == b;
};

int main() {
    static_assert(equal(10, 10));
    static_assert(!equal(10, 20));
}
```

در این مثال، Generic Static Lambda فقط برای نوع‌هایی قابل استفاده است که مفهوم `std::equality_comparable` را برآورده کنند.

### نکته درباره Generic Static Lambda

Static بودن عملگر به معنای غیرممکن بودن استفاده از Template نیست.

عملگر همچنان می‌تواند:

- Function Template باشد.
- از Template Parameterها استفاده کند.
- دارای Parameter Pack باشد.
- از Concepts و Constraints استفاده کند.
- دارای Return Type وابسته به Template Parameter باشد.

بنابراین این قابلیت محدودیتی برای Generic Programming ایجاد نمی‌کند.

## کاربرد در Standard Library و Callable Objects

در کتابخانه استاندارد ++C، مفهوم Callable Object نقش مهمی در الگوریتم‌ها و ابزارهای عمومی دارد.

بسیاری از الگوریتم‌ها می‌توانند یک تابع، Function Pointer، Lambda یا Functor دریافت کنند.

Static Call Operator می‌تواند برای طراحی Functorهای بدون state مفید باشد.

### استفاده با std::sort

```
#include <algorithm>
#include <vector>

struct Ascending {
    static bool operator()(int a, int b) {
        return a < b;
    }
};

int main() {
    std::vector<int> values{5, 2, 8, 1, 9};

    std::sort(values.begin(), values.end(), Ascending{});
}
```

در این مثال، `Ascending` یک Comparator بدون state است.

از آنجا که عملگر static است، برای اجرای مقایسه‌ها نیازی به وضعیت یک instance وجود ندارد.

البته الگوریتم `std::sort` همچنان یک Callable Object دریافت می‌کند و در این مثال از یک instance خالی از `Ascending` استفاده شده است.

### استفاده با std::for_each

```
#include <algorithm>
#include <iostream>
#include <vector>

struct Printer {
    static void operator()(int value) {
        std::cout << value << ' ';
    }
};

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    std::for_each(
        values.begin(),
        values.end(),
        Printer{}
    );
}
```

این Functor برای پردازش هر عنصر به state داخلی احتیاج ندارد.

با این حال، اگر قصد داشته باشید وضعیت مشترکی مانند مجموع اعداد را نگهداری کنید، Static Call Operator به‌تنهایی راهکار مناسبی نیست و باید از state خارجی یا Functor غیر static استفاده کنید.

## ارتباط با Function Pointer

یکی دیگر از موضوعات مهم در طراحی Static Call Operator، تفاوت آن با Function Pointer است.

Static Member Function مانند سایر توابع static می‌تواند به Function Pointer معمولی تبدیل شود.

برای مثال:

```
#include <iostream>

struct Calculator {
    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    using FunctionPointer = int(*)(int, int);

    FunctionPointer function =
        &Calculator::operator();

    std::cout << function(10, 20) << '\n';
}
```

در این مثال، آدرس عملگر static به یک Function Pointer معمولی اختصاص داده شده است.

در مقابل، آدرس یک non-static member function از نوع Member Function Pointer است و به شکل متفاوتی تعریف می‌شود.

### مقایسه انواع Pointer

```
struct Normal {
    int operator()(int value) const {
        return value;
    }
};

struct Static {
    static int operator()(int value) {
        return value;
    }
};

using StaticFunctionPointer = int(*)(int);

using MemberFunctionPointer =
    int (Normal::*)(int) const;
```

در این مثال:

- `StaticFunctionPointer` یک pointer به تابع معمولی است.
- `MemberFunctionPointer` یک pointer به member function است.

این تفاوت در طراحی APIهایی که Function Pointer دریافت می‌کنند اهمیت دارد.

### محدودیت مهم در Function Pointer

Static بودن عملگر به این معنی نیست که خود object نیز به Function Pointer تبدیل می‌شود.

برای مثال:

```
Static object;

int (*function)(int) = &Static::operator();
```

در اینجا pointer به خود تابع static اشاره می‌کند، نه به object از نوع `Static`.

بنابراین اگر API به یک Callable Object نیاز داشته باشد، باید تفاوت بین Function Pointer و Functor را در نظر بگیرید.

## کاربرد در طراحی Stateless Policies

در طراحی Generic Components، گاهی لازم است رفتار یک کلاس از طریق یک Policy Type مشخص شود.

Static Call Operator می‌تواند برای تعریف Policyهای بدون state استفاده شود.

برای مثال:

```
#include <iostream>

struct AdditionPolicy {
    static int apply(int a, int b) {
        return a + b;
    }
};

struct MultiplicationPolicy {
    static int apply(int a, int b) {
        return a * b;
    }
};

template<typename Policy>
struct Calculator {
    int operator()(int a, int b) const {
        return Policy::apply(a, b);
    }
};

int main() {
    Calculator<AdditionPolicy> add;
    Calculator<MultiplicationPolicy> multiply;

    std::cout << add(10, 20) << '\n';
    std::cout << multiply(10, 20) << '\n';
}
```

در این مثال از static member function برای پیاده‌سازی Policy استفاده شده است.

می‌توان همین ایده را با Static Call Operator نیز پیاده‌سازی کرد:

```
struct AdditionPolicy {
    static int operator()(int a, int b) {
        return a + b;
    }
};

template<typename Policy>
struct Calculator {
    int operator()(int a, int b) const {
        return Policy{}(a, b);
    }
};
```

این طراحی زمانی مفید است که بخواهید Policyها نیز به‌عنوان Callable Object قابل استفاده باشند.

نکته مهم این است که Static Call Operator در اینجا یک امکان زبانی برای طراحی API است و الزاماً به معنای بهبود Performance نسبت به یک static function معمولی نیست.

## ارتباط با Template Metaprogramming

Static Call Operator می‌تواند در طراحی کلاس‌های Template و ابزارهای Generic Programming نیز استفاده شود.

به‌خصوص زمانی که یک نوع صرفاً مسئول ارائه یک عملیات بدون state است، استفاده از آن می‌تواند رابط کاربری کلاس را ساده‌تر کند.

### تعریف یک عملیات عمومی

```
#include <type_traits>

template<typename T>
struct Identity {
    static constexpr T operator()(T value) noexcept {
        return value;
    }
};

int main() {
    constexpr int value = Identity<int>{}(42);

    static_assert(value == 42);
}
```

در این مثال، Static Call Operator یک تابع `constexpr` است.

به همین دلیل می‌توان آن را در Constant Evaluation استفاده کرد.

### ترکیب با constexpr

Static Call Operator می‌تواند با ویژگی‌های معمول تابع‌های ++C مانند موارد زیر ترکیب شود:

- `constexpr`
- `consteval`
- `noexcept`
- `inline`
- `template`
- `requires`

برای مثال:

```
struct Square {
    static constexpr int operator()(int value) noexcept {
        return value * value;
    }
};

constexpr int result = Square{}(5);

static_assert(result == 25);
```

در این مثال، تابع در زمان کامپایل قابل ارزیابی است.

توجه کنید که static بودن و constexpr بودن دو ویژگی مستقل هستند. اولی به عدم وابستگی تابع به object اشاره دارد و دومی به امکان Constant Evaluation.

## قواعد زبان و محدودیت‌ها

استفاده از Static Call Operator تابع قواعد مشخصی در زبان ++C است.

شناخت این محدودیت‌ها برای جلوگیری از خطاهای طراحی و کامپایل ضروری است.

### عدم وجود this Pointer

Static Member Function دارای implicit `this` pointer نیست.

بنابراین کد زیر معتبر نیست:

```
struct Example {
    int value = 10;

    static int operator()() {
        return value;
    }
};
```

زیرا `value` یک non-static data member است و برای دسترسی به آن باید object مشخصی وجود داشته باشد.

روش صحیح در صورتی که مقدار موردنظر ثابت و مشترک باشد:

```
struct Example {
    inline static int value = 10;

    static int operator()() {
        return value;
    }
};
```

### عدم امکان دسترسی مستقیم به Non-Static Members

برای دسترسی به non-static member باید یک object مشخص در اختیار داشته باشید.

برای مثال:

```
struct Example {
    int value = 10;

    static int operator()(const Example& object) {
        return object.value;
    }
};
```

این کد معتبر است؛ زیرا object به‌صورت صریح از طریق پارامتر دریافت می‌شود.

در واقع، Static Call Operator می‌تواند روی داده‌های یک object عملیات انجام دهد، اما به‌صورت ضمنی به object جاری دسترسی ندارد.

### عدم امکان استفاده از cv-qualifier

در تعریف non-static member function می‌توان از `const` استفاده کرد:

```
struct Normal {
    int operator()(int value) const {
        return value;
    }
};
```

اما برای Static Call Operator نمی‌توان از cv-qualifier مربوط به member function استفاده کرد.

بنابراین تعریف زیر معتبر نیست:

```
struct Invalid {
    static int operator()(int value) const {
        return value;
    }
};
```

دلیل این محدودیت آن است که تابع static به object جاری وابسته نیست تا const بودن آن معنا داشته باشد.

### عدم امکان استفاده از Ref-Qualifier

در member functionهای معمولی می‌توان با استفاده از `&` و `&&` رفتار تابع را بر اساس Value Category مربوط به object محدود کرد.

برای مثال:

```
struct Normal {
    void operator()() & {
    }

    void operator()() && {
    }
};
```

اما Static Call Operator نمی‌تواند از ref-qualifier استفاده کند؛ زیرا implicit object parameter ندارد.

### عدم امکان Virtual بودن

یک static member function نمی‌تواند virtual باشد.

در نتیجه، Static Call Operator نیز نمی‌تواند virtual تعریف شود.

برای مثال:

```
struct Invalid {
    virtual static void operator()() {
    }
};
```

تعریف بالا نامعتبر است.

اگر به Polymorphic Dispatch نیاز دارید، باید از non-static virtual `operator()` استفاده کنید.

### عدم امکان Override

از آنجا که Static Member Function نمی‌تواند virtual باشد، Static Call Operator نیز قابلیت Override کردن یک عملگر virtual از کلاس پایه را ندارد.

این محدودیت در طراحی Interfaceهای مبتنی بر Inheritance اهمیت دارد.

## تفاوت با Explicit Object Parameter

یکی از قابلیت‌های مهم ++C23، معرفی Explicit Object Parameter است که با نام‌های Deducing This و Explicit Object Member Function نیز شناخته می‌شود.

این قابلیت از نظر مفهومی با Static Call Operator ارتباط دارد، اما یکسان نیست.

### مفهوم Explicit Object Parameter

در روش معمول، object جاری به‌صورت ضمنی به member function منتقل می‌شود.

اما Explicit Object Parameter اجازه می‌دهد نوع و ویژگی‌های object به‌صورت صریح در فهرست پارامترها مشخص شوند.

برای مثال:

```
struct Example {
    int value = 10;

    int operator()(this const Example& self) {
        return self.value;
    }
};
```

در این مثال، `self` یک Explicit Object Parameter است.

این تابع همچنان یک non-static member function محسوب می‌شود، اما object parameter آن به‌صورت صریح تعریف شده است.

### تفاوت اصلی دو قابلیت

| ویژگی                                          | Static operator()                     | Explicit Object Parameter                        |
| ---------------------------------------------- | ------------------------------------- | ------------------------------------------------ |
| وجود object parameter                          | ندارد                                 | دارد                                             |
| دسترسی به state مربوط به object                | فقط از طریق پارامترهای صریح دیگر      | از طریق object parameter                         |
| وجود implicit `this`                           | ندارد                                 | ندارد                                            |
| نوع member function                            | Static                                | Non-Static                                       |
| امکان virtual بودن                             | ندارد                                 | ندارد                                            |
| امکان overload بر اساس object type             | محدود به قواعد Static Member Function | امکان استفاده از نوع‌های متفاوت object parameter |
| قابلیت استفاده از `const` روی object parameter | ندارد                                 | دارد                                             |

### چه زمانی از Explicit Object Parameter استفاده کنیم؟

اگر هدف شما دسترسی به object جاری با کنترل دقیق روی نوع، constness یا Value Category آن است، Explicit Object Parameter گزینه مناسب‌تری خواهد بود.

برای مثال:

```
struct Example {
    int value = 10;

    int& get(this Example& self) {
        return self.value;
    }

    const int& get(this const Example& self) {
        return self.value;
    }
};
```

در مقابل، اگر هیچ نیازی به object parameter ندارید، Static Call Operator انتخاب ساده‌تری است.

## نکات مهم درباره Overloading

مانند سایر Functionها، عملگر `operator()` نیز می‌تواند Overload شود.

Static Call Operator نیز از این قاعده مستثنا نیست.

### Overloading بر اساس تعداد پارامترها

```
struct Calculator {
    static int operator()(int value) {
        return value * value;
    }

    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    int first = Calculator{}(5);
    int second = Calculator{}(10, 20);
}
```

در این مثال، کامپایلر بر اساس تعداد و نوع آرگومان‌ها، overload مناسب را انتخاب می‌کند.

### ترکیب Static و Non-Static Overload

استاندارد ++C23 اجازه می‌دهد overloadهای static و non-static برای `operator()` در یک کلاس وجود داشته باشند، مشروط بر اینکه قواعد Overload Resolution و تفاوت امضاها رعایت شوند.

برای مثال:

```
struct Example {
    int operator()(int value) const {
        return value;
    }

    static int operator()(double value) {
        return static_cast<int>(value * 2);
    }
};

int main() {
    Example object;

    int first = object(10);
    int second = object(2.5);
}
```

در این مثال:

- آرگومان `int` به overload غیر static هدایت می‌شود.
- آرگومان `double` به overload static هدایت می‌شود.

نکته مهم این است که فراخوانی از طریق object به‌تنهایی مشخص نمی‌کند کدام نوع عملگر انتخاب شود؛ نوع و تعداد آرگومان‌ها و قواعد Overload Resolution تعیین‌کننده هستند.

### محدودیت Overload بر اساس Static بودن

Static بودن به‌تنهایی یک معیار مستقل برای تشخیص overload محسوب نمی‌شود.

بنابراین نمی‌توان دو تابع را صرفاً به این دلیل که یکی static و دیگری non-static است و سایر پارامترهای آن‌ها یکسان هستند، به‌عنوان overloadهای مستقل تعریف کرد.

## نکات مهم درباره Operator Overloading

Static Call Operator همچنان تابع قواعد عمومی Operator Overloading است.

بنابراین برخی محدودیت‌های عمومی عملگرهای ++C همچنان برقرار هستند.

### عملگر باید Member Function باشد

عملگر `operator()` یک عملگر خاص است و باید به‌عنوان member function تعریف شود.

بنابراین برخلاف برخی عملگرهای دیگر، نمی‌توان آن را به‌عنوان یک non-member یا free function مستقل تعریف کرد.

### تعداد و نوع پارامترها

تعداد پارامترهای `operator()` می‌تواند متغیر باشد.

همچنین می‌توان از Template Parameterها و Variadic Templates استفاده کرد.

برای مثال:

```
#include <iostream>

struct Printer {
    template<typename... Args>
    static void operator()(Args&&... args) {
        ((std::cout << args << ' '), ...);
    }
};

int main() {
    Printer{}(10, 20, 30, "Hello");
}
```

این مثال یک Variadic Static Call Operator را نشان می‌دهد.

### امکان استفاده از Return Typeهای مختلف

Static Call Operator محدود به نوع بازگشتی خاصی نیست.

می‌توان از انواع مختلف مانند موارد زیر استفاده کرد:

- `void`
- انواع عددی
- Referenceها
- Pointerها
- انواع Template
- انواع User-Defined

برای مثال:

```
struct Factory {
    static auto operator()(int value) {
        return value * 2.5;
    }
};
```

در این مثال از Return Type Deduction استفاده شده است.

## کاربردهای عملی

### طراحی Utility Functorها

برای کلاس‌هایی که صرفاً یک عملیات مستقل انجام می‌دهند، Static Call Operator می‌تواند رابط ساده‌ای فراهم کند.

```
struct IsPositive {
    static bool operator()(int value) {
        return value > 0;
    }
};

int main() {
    bool result = IsPositive{}(10);
}
```

این نوع Functor برای طراحی عملیات بدون state مناسب است.

### طراحی Comparatorهای بدون State

در الگوریتم‌هایی مانند `std::sort`، Comparatorهای بدون state یکی از کاربردهای طبیعی این قابلیت هستند.

```
#include <algorithm>
#include <vector>

struct Compare {
    static bool operator()(int a, int b) {
        return a > b;
    }
};

int main() {
    std::vector<int> values{3, 1, 5, 2, 4};

    std::sort(values.begin(), values.end(), Compare{});
}
```

این Comparator ترتیب نزولی ایجاد می‌کند.

### طراحی Stateless Function Adapter

می‌توان از این قابلیت برای ایجاد کلاس‌هایی استفاده کرد که یک عملیات مشخص را به‌عنوان Callable Object ارائه می‌کنند.

```
struct AbsoluteValue {
    static constexpr int operator()(int value) noexcept {
        return value < 0 ? -value : value;
    }
};
```

البته این پیاده‌سازی برای `INT_MIN` مشکل Overflow دارد و برای کاربرد عمومی باید نوع داده و محدودیت‌های عددی در نظر گرفته شوند.

### طراحی الگوریتم‌های Compile-Time

با ترکیب `static` و `constexpr` می‌توان Callable Typeهایی طراحی کرد که در زمان کامپایل نیز قابل استفاده باشند.

```
struct Factorial {
    static constexpr unsigned long long operator()(unsigned int n) {
        return n <= 1 ? 1 : n * operator()(n - 1);
    }
};

static_assert(Factorial{}(5) == 120);
```

این مثال از یک تابع constexpr بازگشتی استفاده می‌کند.

البته محدودیت‌های عددی نوع `unsigned long long` همچنان برقرار هستند و برای ورودی‌های بزرگ ممکن است Overflow رخ دهد.

## اشتباهات رایج

### تصور اینکه Static بودن باعث حذف نیاز به Callable Object می‌شود

یکی از اشتباهات رایج این است که تصور شود پس از تعریف Static Call Operator دیگر نیازی به ساخت object نیست.

در واقع، برای استفاده از syntax معمول Functor همچنان می‌توان یک object ساخت:

```
struct Example {
    static void operator()() {
    }
};

int main() {
    Example{}();
}
```

البته برای فراخوانی مستقیم عملگر می‌توان از class scope نیز استفاده کرد:

```
Example::operator()();
```

بنابراین وجود object در syntax معمول یک الزام برای اجرای تابع نیست.

### تلاش برای دسترسی به this

از آنجا که عملگر static است، استفاده از `this` در بدنه آن مجاز نیست.

```
struct Example {
    static void operator()() {
        // Invalid: static member functions have no this pointer
        // this->process();
    }

    void process() {
    }
};
```

برای رفع این مشکل باید object موردنظر را به‌صورت صریح به تابع منتقل کنید یا عملگر را non-static تعریف کنید.

### استفاده از Static برای Stateful Functor

اگر Functor به اطلاعات داخلی نیاز دارد، static کردن عملگر معمولاً انتخاب مناسبی نیست.

برای مثال:

```
struct Counter {
    int count = 0;

    void operator()() {
        ++count;
    }
};
```

در این حالت، استفاده از non-static `operator()` ضروری است؛ زیرا رفتار تابع به state مربوط به object وابسته است.

### تصور اینکه Static همیشه سریع‌تر است

Static Call Operator لزوماً سریع‌تر از Non-Static Call Operator نیست.

کامپایلر می‌تواند Functorهای معمولی بدون state را نیز بهینه‌سازی کند.

بنابراین انتخاب بین این دو باید بر اساس طراحی و معناشناسی برنامه انجام شود، نه صرفاً فرض بهبود Performance.

### اشتباه گرفتن Static Lambda با Lambda بدون Capture

هر Lambda بدون Capture الزاماً Static Lambda نیست.

برای مثال:

```
auto normal = [](int value) {
    return value * 2;
};

auto staticLambda = [](int value) static {
    return value * 2;
};
```

هر دو Lambda فاقد capture هستند، اما در دومی عملگر فراخوانی به‌صورت صریح static تعریف شده است.

این تفاوت از نظر ویژگی‌های Closure Type اهمیت دارد.

## سازگاری با نسخه‌های مختلف استاندارد

Static Call Operator در استاندارد ++C23 معرفی شده است.

بنابراین کدهایی که از این قابلیت استفاده می‌کنند، به یک Compiler با پشتیبانی مناسب از ++C23 نیاز دارند.

| استاندارد | پشتیبانی                  |
| --------- | ------------------------- |
| ++C11     | پشتیبانی نمی‌شود          |
| ++C14     | پشتیبانی نمی‌شود          |
| ++C17     | پشتیبانی نمی‌شود          |
| ++C20     | پشتیبانی نمی‌شود          |
| ++C23     | معرفی و استاندارد شده است |

### Feature-Test Macro

برای بررسی پشتیبانی Compiler می‌توان از Feature-Test Macro زیر استفاده کرد:

```
#ifdef __cpp_static_call_operator
    // Static call operator support is available
#endif
```

مقدار استاندارد این Macro برای قابلیت موردنظر برابر است با:

```
202207L
```

بنابراین می‌توان بررسی دقیق‌تری انجام داد:

```
#if defined(__cpp_static_call_operator) && \
    __cpp_static_call_operator >= 202207L

    #define HAS_STATIC_CALL_OPERATOR 1

#else

    #define HAS_STATIC_CALL_OPERATOR 0

#endif
```

توجه کنید که فعال‌کردن گزینه استاندارد ++C23 در Compiler به‌تنهایی تضمین نمی‌کند که تمام قابلیت‌های این استاندارد پیاده‌سازی شده باشند.

پشتیبانی واقعی به نسخه Compiler و میزان تکمیل پیاده‌سازی آن بستگی دارد.

## مقایسه با روش‌های قدیمی‌تر

پیش از ++C23 نیز امکان طراحی Functorهای بدون state وجود داشت.

بنابراین Static Call Operator یک قابلیت کاملاً جدید برای انجام عملیات بدون state نیست؛ بلکه راهی استاندارد برای بیان مستقیم‌تر این مفهوم است.

### استفاده از Non-Static Functor

```
struct OldStyle {
    int operator()(int value) const {
        return value * 2;
    }
};
```

این روش در استانداردهای قدیمی‌تر نیز معتبر است و همچنان برای Functorهای دارای state یا نیازمند object semantics کاربرد دارد.

### استفاده از Static Member Function معمولی

```
struct Utility {
    static int execute(int value) {
        return value * 2;
    }
};
```

این روش برای فراخوانی مستقیم تابع مناسب است، اما رابط Callable Object ارائه نمی‌کند.

### استفاده از Function Pointer

```
int doubleValue(int value) {
    return value * 2;
}

int main() {
    int (*function)(int) = doubleValue;

    int result = function(10);
}
```

Function Pointer برای APIهایی که به یک تابع مستقل نیاز دارند گزینه مناسبی است.

### استفاده از Static operator()

```
struct StaticFunctor {
    static int operator()(int value) {
        return value * 2;
    }
};

int main() {
    int result = StaticFunctor{}(10);
}
```

این روش برای مواقعی مناسب است که می‌خواهید یک Type قابل فراخوانی داشته باشید، اما رفتار آن به instance state وابسته نباشد.

## ملاحظات طراحی و Performance

### آیا Static Call Operator باعث کاهش حافظه می‌شود؟

Static Call Operator به‌خودی‌خود اندازه object را کاهش نمی‌دهد.

برای مثال:

```
struct Example {
    static void operator()() {
    }
};
```

این کلاس همچنان یک کلاس معمولی است و می‌توان از آن object ایجاد کرد.

با این حال، اگر کلاس هیچ data member غیر static نداشته باشد، طبق قواعد زبان اندازه object آن حداقل یک بایت خواهد بود.

بنابراین static بودن عملگر به معنی Zero-Size بودن object نیست.

### آیا می‌توان از Empty Base Optimization استفاده کرد؟

بله. Static Call Operator می‌تواند در کلاس‌های خالی مورد استفاده قرار گیرد و چنین کلاس‌هایی همچنان می‌توانند از Empty Base Optimization بهره ببرند.

برای مثال:

```
struct Policy {
    static int operator()(int value) {
        return value * 2;
    }
};

struct Container : Policy {
    int data = 10;
};
```

کلاس `Policy` یک Empty Class است و قواعد مربوط به Empty Base Optimization مستقل از static بودن عملگر آن هستند.

### آیا Static Call Operator برای Thread Safety مناسب‌تر است؟

Static بودن عملگر به‌تنهایی تضمین‌کننده Thread Safety نیست.

اگر عملگر از shared mutable state استفاده کند، همچنان احتمال Data Race وجود دارد.

برای مثال:

```
struct Counter {
    inline static int count = 0;

    static void operator()() {
        ++count;
    }
};
```

در صورت فراخوانی هم‌زمان از چند Thread، دسترسی‌های بدون Synchronization به `count` می‌توانند باعث Data Race شوند.

بنابراین Static بودن تابع را نباید با Thread Safety اشتباه گرفت.

## راهنمای انتخاب بین Static و Non-Static

برای انتخاب صحیح، ابتدا باید مشخص کنید که رفتار موردنظر واقعاً به object state وابسته است یا خیر.

| شرایط                                             | انتخاب پیشنهادی                  |
| ------------------------------------------------- | -------------------------------- |
| رفتار کاملاً مستقل از instance است                | Static operator()                |
| رفتار به memberهای object وابسته است              | Non-Static operator()            |
| نیاز به Capture در Lambda وجود دارد               | Lambda معمولی                    |
| Lambda بدون Capture و Static بودن صریح مطلوب است  | Static Lambda                    |
| نیاز به Polymorphic Dispatch وجود دارد            | Virtual Non-Static operator()    |
| نیاز به دسترسی کنترل‌شده به object جاری وجود دارد | Explicit Object Parameter        |
| API به Function Pointer معمولی نیاز دارد          | Static Function یا Free Function |
| نیاز به نگهداری وضعیت بین فراخوانی‌ها وجود دارد   | Stateful Functor                 |

نکته مهم این است که Static Call Operator جایگزین تمام Functorهای معمولی نیست. این قابلیت صرفاً انتخاب دیگری برای طراحی Callable Typeهایی است که به object state وابستگی ندارند.

## جمع‌بندی

قابلیت Static `operator()` که در استاندارد ++C23 معرفی شده است، امکان تعریف Function Call Operator بدون implicit object parameter را فراهم می‌کند.

این قابلیت برای طراحی Stateless Functorها، Generic Callable Objects و Static Lambdaها کاربرد دارد.

مهم‌ترین مزایای آن عبارت‌اند از:

- امکان تعریف Callable Typeهای مستقل از instance state.
- حذف وابستگی به implicit `this` pointer.
- امکان فراخوانی مستقیم عملگر از طریق Class Scope.
- امکان استفاده از Static Lambda در ++C23.
- پشتیبانی از Generic Programming، Concepts و Templateها.
- امکان تبدیل عملگر static به Function Pointer معمولی.

در مقابل، محدودیت‌های مهم آن شامل موارد زیر است:

- عدم دسترسی مستقیم به non-static memberها.
- عدم وجود `this` pointer.
- عدم امکان استفاده از cv-qualifier و ref-qualifier.
- عدم امکان virtual بودن و Override شدن.
- عدم امکان استفاده از capture در Static Lambda.

در نهایت، بهترین رویکرد این است که از Static Call Operator زمانی استفاده کنید که رفتار Callable Object واقعاً مستقل از وضعیت instance باشد.

اگر رفتار تابع به وضعیت object وابسته است یا به قابلیت‌هایی مانند Polymorphism و Ref-Qualification نیاز دارید، Non-Static Call Operator همچنان انتخاب مناسب‌تری خواهد بود.

این قابلیت را باید به‌عنوان یک ابزار تکمیلی برای طراحی بهتر Callable Objects در ++C23 در نظر گرفت، نه جایگزینی عمومی برای Functorهای سنتی.

مطالب تکراری را ادغام کنیک مسیر آموزشی منسجم بسازادعاهای فنی را دقیق‌تر کن

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>