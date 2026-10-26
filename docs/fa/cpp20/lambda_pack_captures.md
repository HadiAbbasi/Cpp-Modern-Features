<div align="center">

[🇺🇸 English](../../en/cpp20/lambda_pack_captures.md) | [🇮🇷 فارسی](./lambda_pack_captures.md)

</div>

---

# راهنمای جامع Pack Captures در Lambdaهای ++C20

## مقدمه

قابلیت Pack Captures در Lambdaها یکی از امکانات معرفی‌شده در استاندارد ++C20 است که امکان Capture کردن تعداد متغیری از مقادیر را در یک Lambda Expression فراهم می‌کند. این قابلیت به‌ویژه هنگام کار با Variadic Templates، پارامترهای Template و توابعی که تعداد متغیری از آرگومان‌ها دریافت می‌کنند، کاربرد زیادی دارد.

پیش از ++C20، اگر می‌خواستیم یک Parameter Pack را مستقیماً داخل Capture List یک Lambda قرار دهیم، چنین کاری با Syntax استاندارد امکان‌پذیر نبود. برای حل این مشکل معمولاً به راهکارهای جایگزین، مانند ساختن یک Tuple یا استفاده از یک Helper Function، متوسل می‌شدیم.

با معرفی Pack Captures، می‌توانیم مقادیر موجود در یک Parameter Pack را مستقیماً در Capture List گسترش دهیم و از آن‌ها در بدنه Lambda استفاده کنیم.

برای مثال، در ++C20 می‌توانیم بنویسیم:

```
#include <iostream>
#include <utility>

template <typename... Args>
auto makeLambda(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        ((std::cout << values << ' '), ...);
        std::cout << '\n';
    };
}

int main()
{
    auto lambda = makeLambda(10, 20, 30);
    lambda();
}
```

خروجی برنامه:

```
10 20 30
```

در این مثال، هر آرگومان به یک عضو مستقل از Closure Object تبدیل می‌شود و Lambda می‌تواند بعداً بدون وابستگی به طول عمر متغیرهای محلی تابع `makeLambda`، به مقادیر Captureشده دسترسی داشته باشد.

البته نکته مهم این است که Pack Captures به‌خودی‌خود یک مکانیزم جدید برای ذخیره‌سازی یا مدیریت حافظه نیست؛ بلکه قابلیت جدیدی در Syntax زبان است که با مکانیزم‌های موجود در Lambda Capture و Parameter Pack ترکیب می‌شود.

## مفهوم Lambda Capture و نقش آن در Closure Object

برای درک صحیح Pack Captures، ابتدا باید بدانیم Capture در Lambda چگونه عمل می‌کند.

یک Lambda Expression در ++C، هنگام نیاز به نگهداری وضعیت، یک Closure Object ایجاد می‌کند. این شیء از نوعی اختصاصی و بدون نام است که کامپایلر برای Lambda تعریف می‌کند. متغیرهایی که در Capture List قرار می‌گیرند، به‌صورت وضعیت ذخیره‌شده در این شیء قابل دسترسی خواهند بود.

برای مثال:

```
int value = 42;

auto lambda = [value] {
    return value;
};
```

در این مثال، مقدار `value` به‌صورت مقداری در Closure Object ذخیره می‌شود. تغییر بعدی متغیر اصلی، مقدار Captureشده را تغییر نمی‌دهد.

در مقابل، با Capture به‌صورت Reference می‌توان به متغیر اصلی دسترسی داشت:

```
int value = 42;

auto lambda = [&value] {
    return value;
};

value = 100;

std::cout << lambda(); // 100
```

بنابراین، Capture تعیین می‌کند که Lambda چگونه به وضعیت موردنیاز خود دسترسی داشته باشد؛ اما Pack Captures این قابلیت را گسترش می‌دهد تا بتوانیم چندین مقدار را به‌صورت یک Parameter Pack در Capture List مدیریت کنیم.

## مفهوم Parameter Pack در ++C

قابلیت Variadic Templates از ++C11 معرفی شد و به ما اجازه می‌دهد Templateهایی بنویسیم که تعداد متغیری از پارامترها را دریافت می‌کنند.

برای مثال:

```
template <typename... Types>
void process(Types... values)
{
}
```

در اینجا `Types` یک Template Parameter Pack و `values` یک Function Parameter Pack است.

هنگام فراخوانی تابع به شکل زیر:

```
process(10, 3.14, 'A');
```

پارامترها به سه مقدار با نوع‌های متفاوت تبدیل می‌شوند:

- `int`
- `double`
- `char`

در بسیاری از موارد، لازم است این مقادیر را به یک Lambda منتقل کنیم تا پس از پایان اجرای تابع فعلی نیز قابل استفاده باشند. Pack Captures دقیقاً چنین امکانی را فراهم می‌کند.

## معرفی Syntax مربوط به Pack Captures

در ++C20 می‌توانیم از عملگر گسترش `...` در Capture List استفاده کنیم تا یک Parameter Pack را به مجموعه‌ای از Captureها تبدیل کنیم.

شکل کلی این قابلیت به‌صورت زیر است:

```
[...capturedPack = expressionPack] {
    // Use capturedPack...
}
```

در این Syntax، عبارت `expressionPack` باید قابلیت گسترش به مجموعه‌ای از Initializerها را داشته باشد. برای هر عنصر از Pack، یک مقدار Captureشده ساخته می‌شود.

نکته مهم این است که `capturedPack` در بدنه Lambda همچنان یک Pack است؛ بنابراین برای دسترسی به عناصر آن در یک Context نیازمند گسترش، باید از `...` استفاده کنیم.

برای نمونه:

```
#include <iostream>
#include <utility>

template <typename... Args>
auto makePrinter(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        ((std::cout << values << '\n'), ...);
    };
}

int main()
{
    auto printer = makePrinter(10, 20, 30);
    printer();
}
```

در این مثال، `values` یک Pack از مقادیر Captureشده است و Fold Expression تمام عناصر آن را به‌ترتیب چاپ می‌کند.

### تفاوت Pack Expansion و Pack Capture

این دو مفهوم به یکدیگر مرتبط هستند، اما یکسان نیستند.

- Pack Expansion مکانیزمی برای گسترش یک Parameter Pack در یک Context مجاز زبان است.
- Pack Capture از Pack Expansion در Capture List استفاده می‌کند تا چندین مقدار را در Closure Object ذخیره کند.

برای مثال، عبارت زیر یک Pack Expansion است:

```
printAll(values...);
```

اما عبارت زیر از Pack Expansion برای ایجاد Captureهای مستقل استفاده می‌کند:

```
[...captured = values] {
    // Use captured...
}
```

در مثال دوم، هر مقدار متناظر با یکی از عناصر `values` در وضعیت Closure Object ذخیره می‌شود.

## بررسی Init-Capture در Pack Captures

یکی از مهم‌ترین کاربردهای این قابلیت، ترکیب آن با Init-Capture است.

قابلیت Init-Capture از ++C14 معرفی شد و به ما اجازه می‌دهد متغیرهایی را مستقیماً در Capture List ایجاد کنیم؛ حتی اگر متغیری با همان نام در Scope بیرونی وجود نداشته باشد.

برای مثال:

```
auto lambda = [value = 42] {
    return value;
};
```

در ++C20 می‌توانیم همین ایده را برای یک Pack گسترش دهیم:

```
template <typename... Args>
auto makeLambda(Args... args)
{
    return [...values = args] {
        return sizeof...(values);
    };
}
```

در این مثال، هر عنصر از `args` به یک Capture مستقل با نام متناظر در Pack به نام `values` تبدیل می‌شود.

البته این نمونه پارامترها را به‌صورت مقداری دریافت می‌کند. برای جلوگیری از Copyهای غیرضروری و حفظ رفتار مناسب در مواجهه با مقادیر موقت، معمولاً استفاده از Forwarding Reference انتخاب بهتری است:

```
template <typename... Args>
auto makeLambda(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        return sizeof...(values);
    };
}
```

در این نسخه، نوع و Value Category آرگومان‌ها در هنگام ساخت Captureها لحاظ می‌شود؛ البته نتیجه نهایی به نوع واقعی هر آرگومان و قواعد Deduction وابسته است.

## مدیریت Copy و Move در Pack Captures

یکی از نکات مهم در استفاده حرفه‌ای از Pack Captures، درک دقیق رفتار Copy و Move است.

در عبارت زیر:

```
template <typename... Args>
auto captureByCopy(Args&&... args)
{
    return [...values = args] {
        // Use values...
    };
}
```

مقادیر `args` برای ساخت Captureها استفاده می‌شوند. با وجود اینکه `args` از نوع Forwarding Reference است، نام هر پارامتر یک Lvalue Expression محسوب می‌شود. بنابراین این روش به‌خودی‌خود Move را تضمین نمی‌کند.

اگر بخواهیم متناسب با نوع Deduceشده هر آرگومان از Forwarding استفاده کنیم، باید بنویسیم:

```
template <typename... Args>
auto captureWithForwarding(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        // Use values...
    };
}
```

در این حالت، اگر آرگومان اولیه یک Lvalue باشد، رفتار مناسب Copy یا ساخت از Lvalue اعمال می‌شود؛ اگر آرگومان یک Rvalue باشد، امکان Move فراهم خواهد شد.

البته Forwarding صرفاً تضمین می‌کند که Value Category به‌درستی منتقل شود؛ اینکه در نهایت Copy یا Move رخ دهد، به نوع مقصد و Constructorهای در دسترس بستگی دارد.

### استفاده از Move برای انتقال مالکیت

فرض کنید می‌خواهیم یک شیء غیرقابل Copy را به یک Lambda منتقل کنیم. در این حالت می‌توانیم از Pack Captures همراه با `std::move` یا Forwarding استفاده کنیم.

```
#include <memory>
#include <utility>

template <typename... Args>
auto makeTask(Args&&... args)
{
    return [...items = std::forward<Args>(args)]() mutable {
        // Work with the captured objects.
        ((items.reset()), ...);
    };
}

int main()
{
    auto task = makeTask(
        std::make_unique<int>(10),
        std::make_unique<int>(20)
    );

    task();
}
```

در این مثال، اشیای `std::unique_ptr` به داخل Closure Object منتقل می‌شوند. از آنجا که `unique_ptr` قابل Copy نیست، انتقال آن‌ها باید با Move انجام شود.

همچنین استفاده از `mutable` ضروری است، زیرا به‌صورت پیش‌فرض، عملگر فراخوانی Lambda با Captureهای مقداری، آن‌ها را از طریق یک `operator()` ثابت در دسترس قرار می‌دهد. با `mutable` می‌توان وضعیت Captureشده را در Lambda تغییر داد.

## بررسی Captureهای قابل تغییر و غیرقابل تغییر

در Lambdaهای معمولی، اگر Capture به‌صورت مقداری انجام شود، بدنه Lambda به‌طور پیش‌فرض اجازه تغییر مستقیم اعضای Captureشده را نمی‌دهد.

برای مثال:

```
int value = 10;

auto lambda = [value] {
    // value = 20; // Error
    return value;
};
```

با افزودن `mutable` می‌توان مقدار Captureشده را تغییر داد:

```
int value = 10;

auto lambda = [value]() mutable {
    value += 5;
    return value;
};
```

این موضوع در Pack Captures نیز برقرار است:

```
template <typename... Args>
auto makeIncrementer(Args... args)
{
    return [...values = args]() mutable {
        ((++values), ...);
    };
}
```

در این مثال، هر عنصر Captureشده قابل تغییر است و با هر بار فراخوانی Lambda، مقدار عناصر افزایش پیدا می‌کند.

باید توجه داشت که `mutable` مقدار متغیرهای اصلی را تغییر نمی‌دهد؛ زیرا مقادیر به Closure Object کپی شده‌اند. همچنین `mutable` صرفاً محدودیت `const` بودن عملگر فراخوانی را تغییر می‌دهد و به معنای Thread Safety نیست.

## تفاوت Capture به‌صورت مقداری و Reference در Pack Captures

یکی از نکات مهم این است که Pack Captures الزاماً به معنای کپی‌کردن تمام مقادیر نیست. نوع Initializer تعیین می‌کند که هر Capture چگونه ساخته شود.

برای مثال، می‌توانیم مقادیر را به‌صورت مستقل ذخیره کنیم:

```
template <typename... Args>
auto captureValues(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        // Use owned captured values.
    };
}
```

در این حالت، Closure Object مقادیر مستقلی از آرگومان‌ها نگهداری می‌کند؛ این مقادیر ممکن است با Copy یا Move ساخته شوند.

در مقابل، اگر بخواهیم Referenceها را Capture کنیم، می‌توانیم از `std::ref` و `std::reference_wrapper` یا سازوکاری مناسب برای حفظ Reference استفاده کنیم. بااین‌حال، باید مراقب طول عمر اشیای اصلی باشیم.

برای مثال:

```
#include <functional>
#include <iostream>
#include <utility>

template <typename... Args>
auto captureReferences(Args&... args)
{
    return [...refs = std::ref(args)] {
        ((std::cout << refs.get() << '\n'), ...);
    };
}

int main()
{
    int first = 10;
    int second = 20;

    auto lambda = captureReferences(first, second);

    first = 100;
    lambda();
}
```

خروجی:

```
100
20
```

در این مثال، `refs` شامل اشیای `std::reference_wrapper` است که به متغیرهای اصلی اشاره می‌کنند؛ بنابراین تغییر `first` در خروجی Lambda مشاهده می‌شود.

نکته مهم: استفاده از `std::ref` طول عمر شیء مرجع را افزایش نمی‌دهد. اگر Lambda پس از نابودی متغیر اصلی فراخوانی شود، دسترسی به آن معتبر نخواهد بود.

## بررسی Pack Captures با Lambdaهای Generic

از ++C14، Generic Lambdaها امکان استفاده از پارامترهای `auto` را در فهرست پارامترهای Lambda فراهم کردند. در ++C20 نیز می‌توانیم Pack Captures را با Generic Lambdaها ترکیب کنیم تا Lambdaهای انعطاف‌پذیرتری بسازیم.

برای مثال:

```
#include <iostream>
#include <utility>

auto makePrinter()
{
    return []<typename... Args>(Args&&... args) {
        ((std::cout << args << ' '), ...);
        std::cout << '\n';
    };
}

int main()
{
    auto printer = makePrinter();
    printer(10, 20.5, 'A');
}
```

در این مثال، از Pack Captures استفاده نشده است؛ بلکه یک Generic Lambda داریم که هنگام فراخوانی، پارامترهای متغیر دریافت می‌کند.

این تفاوت از نظر طراحی مهم است:

- Pack Captures برای نگهداری مقادیر در Closure Object استفاده می‌شود.
- Generic Lambda با Parameter Pack برای دریافت تعداد متغیری از آرگومان‌ها هنگام فراخوانی استفاده می‌شود.

اگر به هر دو قابلیت نیاز داشته باشیم، می‌توانیم آن‌ها را ترکیب کنیم:

```
#include <iostream>
#include <utility>

template <typename... Stored>
auto makeProcessor(Stored&&... stored)
{
    return [...values = std::forward<Stored>(stored)]
           <typename... Args>(Args&&... args) mutable {
        ((std::cout << values << ' '), ...);
        ((std::cout << args << ' '), ...);
        std::cout << '\n';
    };
}

int main()
{
    auto processor = makeProcessor(10, 20);
    processor(30, 40);
}
```

خروجی:

```
10 20 30 40
```

در این مثال، Lambda هم وضعیت ذخیره‌شده دارد و هم هنگام فراخوانی می‌تواند آرگومان‌های جدید دریافت کند.

## مدیریت Pack خالی و Fold Expression

از آنجا که Pack Captures معمولاً با Variadic Templates استفاده می‌شود، باید رفتار آن را در حالت خالی نیز در نظر بگیریم.

فرض کنید تابع زیر هیچ آرگومانی دریافت نکند:

```
template <typename... Args>
auto makeLambda(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        return sizeof...(values);
    };
}
```

فراخوانی زیر معتبر است:

```
auto lambda = makeLambda();
```

در این حالت، Pack به‌اندازه صفر عنصر دارد و Closure Object هیچ Captureای از این Pack نخواهد داشت. فراخوانی `lambda()` مقدار صفر را برمی‌گرداند.

اما اگر بدنه Lambda از Fold Expression استفاده کند، باید قواعد Fold Expression را نیز در نظر بگیریم.

برای مثال:

```
template <typename... Args>
auto makeSum(Args... args)
{
    return [...values = args] {
        return (values + ... + 0);
    };
}
```

عبارت `(values + ... + 0)` یک Right Fold با مقدار اولیه صفر است. بنابراین اگر Pack خالی باشد، نتیجه صفر خواهد بود.

در مقابل، Foldهای بدون مقدار اولیه برای Pack خالی فقط در برخی عملگرها نتیجه مشخصی دارند. برای عملگرهای `&&`، `||` و `,` قواعد ویژه‌ای وجود دارد؛ اما برای عملگرهایی مانند `+` نمی‌توان بدون بررسی قواعد زبان، نتیجه‌ای برای Pack خالی فرض کرد.

توصیه: در کدهای عمومی و کتابخانه‌ای، رفتار Pack خالی را صریحاً طراحی کنید؛ به‌خصوص اگر Lambda بخشی از یک API عمومی باشد.

## مقایسه Pack Captures با روش‌های پیش از ++C20

پیش از ++C20، راه مستقیمی برای گسترش یک Pack در Init-Capture List وجود نداشت. بااین‌حال، راهکارهایی برای رسیدن به نتیجه مشابه قابل استفاده بودند.

### روش استفاده از Tuple

یکی از راهکارهای رایج، ذخیره آرگومان‌ها در یک `std::tuple` بود.

```
#include <tuple>
#include <utility>

template <typename... Args>
auto makeLambda(Args&&... args)
{
    auto values = std::make_tuple(std::forward<Args>(args)...);

    return [values = std::move(values)] {
        return std::tuple_size_v<decltype(values)>;
    };
}
```

این روش در ++C17 قابل استفاده است و مقادیر را در یک شیء Tuple نگهداری می‌کند.

بااین‌حال، دسترسی به عناصر Tuple معمولاً به `std::get`، ابزارهای متاپروگرمینگ یا `std::apply` نیاز دارد. در مقابل، Pack Captures اجازه می‌دهد هر عنصر مستقیماً از طریق Pack موجود در Lambda استفاده شود.

### روش استفاده از Helper Function

در برخی طراحی‌ها، می‌توانستیم یک تابع کمکی بسازیم که آرگومان‌ها را دریافت کند و وضعیت موردنیاز Lambda را آماده سازد. این راهکار همچنان معتبر است و در بعضی موارد به دلیل جداسازی مسئولیت‌ها یا پیچیدگی خاص مالکیت، می‌تواند انتخاب مناسبی باشد.

بااین‌حال، برای Capture مستقیم یک Pack، قابلیت ++C20 معمولاً کد را ساده‌تر و خواناتر می‌کند.

## کاربردهای عملی Pack Captures

این قابلیت در سناریوهایی مفید است که یک تابع Variadic باید داده‌ها را دریافت کند و برای استفاده در آینده در یک Lambda نگهداری کند.

### ساخت Callback با وضعیت ذخیره‌شده

فرض کنید یک سیستم Event یا Callback داریم که باید هنگام ثبت Callback، آرگومان‌های لازم را ذخیره کند و بعداً از آن‌ها استفاده کند.

```
#include <iostream>
#include <utility>

template <typename Function, typename... Args>
auto bindArguments(Function&& function, Args&&... args)
{
    return [...values = std::forward<Args>(args),
            function = std::forward<Function>(function)]() mutable {
        return function(values...);
    };
}

int main()
{
    auto callback = bindArguments(
        [](int a, int b, int c) {
            return a + b + c;
        },
        10, 20, 30
    );

    std::cout << callback() << '\n';
}
```

خروجی:

```
60
```

در این مثال، مقادیر آرگومان‌ها و تابع موردنظر در Closure Object ذخیره می‌شوند. توجه داشته باشید که این نمونه برای آرگومان‌های ذخیره‌شده‌ای مناسب است که با پارامترهای تابع سازگار باشند؛ در طراحی عمومی‌تر باید رفتار Copy، Move و فراخوانی‌های تکراری نیز در نظر گرفته شود.

### ذخیره تنظیمات برای اجرای یک Task

در سیستم‌های Task، ممکن است لازم باشد مقادیر موردنیاز برای اجرای یک عملیات را ذخیره کنیم تا عملیات در زمان دیگری انجام شود.

```
#include <iostream>
#include <utility>

template <typename... Args>
auto makeTask(Args&&... args)
{
    return [...values = std::forward<Args>(args)] {
        ((std::cout << values << ' '), ...);
        std::cout << '\n';
    };
}

int main()
{
    auto task = makeTask(10, 20, 30);
    task();
}
```

در این نمونه، Task به متغیرهای محلی تابع سازنده وابسته نیست، زیرا مقادیر موردنیاز را در Closure Object نگهداری می‌کند.

البته اگر مقادیر ذخیره‌شده شامل Pointer، Reference یا Handle به منابع خارجی باشند، اعتبار و طول عمر آن منابع همچنان باید مدیریت شود.

## اشتباهات رایج در استفاده از Pack Captures

### فراموش‌کردن Forwarding

در یک تابع Template، استفاده از `args` به‌تنهایی معمولاً به معنای Forwarding نیست.

```
template <typename... Args>
auto makeLambda(Args&&... args)
{
    return [...values = args] {
        // Use values...
    };
}
```

در این مثال، Initializerها از Lvalue Expressionهای نام‌دار `args` ساخته می‌شوند. برای حفظ Value Category آرگومان‌های ورودی، معمولاً باید از `std::forward<Args>(args)` استفاده کنیم.

### اشتباه‌گرفتن Pack ذخیره‌شده با آرگومان‌های ورودی

پس از ساخت Lambda، Pack ذخیره‌شده بخشی از وضعیت Closure Object است. تغییر متغیرهای اولیه، Captureهای مقداری را تغییر نمی‌دهد؛ همچنین تغییر Captureها به‌طور خودکار متغیرهای اولیه را تغییر نخواهد داد.

### نادیده‌گرفتن طول عمر Referenceها

اگر Pack شامل Reference Wrapperها، Pointerها یا اشیای مشابه باشد، ذخیره‌شدن آن‌ها در Closure Object تضمینی برای معتبرماندن اشیای مقصد ایجاد نمی‌کند.

### استفاده نادرست از `mutable`

وجود `mutable` باعث می‌شود Captureهای مقداری در بدنه Lambda قابل تغییر باشند؛ اما این ویژگی به‌تنهایی مالکیت مشترک، همگام‌سازی Threadها یا ایمنی هم‌زمانی را تضمین نمی‌کند.

### تصور اینکه Pack Captures فقط برای Copy است

با استفاده از Init-Capture و ابزارهایی مانند `std::move` و `std::forward`، می‌توان مالکیت اشیا را نیز منتقل کرد. رفتار واقعی به نوع آرگومان‌ها، Initializerها و Constructorهای قابل استفاده وابسته است.

## محدودیت‌ها و ملاحظات طراحی

قابلیت Pack Captures با وجود کاربردهای فراوان، چند نکته مهم دارد:

- این قابلیت از ++C20 به بعد بخشی از زبان استاندارد است؛ برای استفاده از آن باید کامپایلر و تنظیمات پروژه از ++C20 پشتیبانی کنند.
- Pack Captures به‌تنهایی مشکل Lifetime، مالکیت یا مدیریت منابع خارجی را حل نمی‌کند.
- نوع هر Capture از Initializer متناظر با آن تعیین می‌شود؛ بنابراین در Templateهای عمومی باید به Reference، `const`، Copy و Move توجه کرد.
- اگر Lambda قرار است چند بار اجرا شود، باید مشخص کنیم وضعیت Captureشده در هر اجرا حفظ می‌شود، تغییر می‌کند یا مصرف می‌شود.
- در APIهای عمومی، باید رفتار Pack خالی و اشیای غیرقابل Copy نیز بررسی شود.

## جمع‌بندی نهایی

قابلیت Pack Captures در ++C20 راهکاری استاندارد و خوانا برای Capture کردن تعداد متغیری از مقادیر در Lambda Expressionها است. این ویژگی به‌خصوص در ترکیب با Variadic Templates، Forwarding Referenceها و Init-Capture کاربرد دارد.

با استفاده از این قابلیت، می‌توانیم وضعیت لازم برای Callbackها، Taskها و توابع تأخیری را مستقیماً در Closure Object ذخیره کنیم؛ بدون آنکه الزاماً به Tuple یا Helperهای اضافی نیاز داشته باشیم.

مهم‌ترین نکته در استفاده حرفه‌ای از Pack Captures، درک صحیح رابطه میان Pack Expansion، Value Category، Copy و Move، و همچنین طول عمر اشیای Captureشده است. تسلط بر این مفاهیم به ما اجازه می‌دهد Lambdaهای عمومی‌تر، امن‌تر و خواناتری در پروژه‌های مدرن ++C طراحی کنیم.

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>