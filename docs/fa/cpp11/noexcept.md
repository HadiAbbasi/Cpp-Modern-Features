<div align="center">

[🇺🇸 English](../../en/cpp11/noexcept.md) | [🇮🇷 فارسی](./noexcept.md)

</div>

---

# راهنمای جامع noexcept در ++C

قابلیت `noexcept` در ++C مکانیزمی برای اعلام این موضوع است که یک تابع انتظار نمی‌رود exception پرتاب کند. این قابلیت از ++C11 وارد زبان شد و امروزه بخش مهمی از طراحی API، مدیریت exception، بهینه‌سازی Move Semantics و Generic Programming محسوب می‌شود.

نکته مهم این است که `noexcept` صرفاً یک «قول» برای خواننده کد یا یک hint برای compiler نیست؛ در زبان ++C بخشی از semantics تابع است. اگر exception از تابعی خارج شود که مشخصات `noexcept` آن اجازه چنین خروجی‌ای را نمی‌دهد، برنامه وارد فرآیند termination می‌شود.

---

## معرفی noexcept

مشخصات `noexcept` را می‌توان در declaration یا definition یک تابع قرار داد:

```cpp
void process() noexcept;
```

در این حالت تابع `process` اعلام می‌کند که exception نباید از آن خارج شود.

همچنین می‌توان یک expression را به `noexcept` داد:

```cpp
void process() noexcept(noexcept(do_work()));
```

در این حالت وضعیت `noexcept` تابع به نتیجه expression داخلی وابسته است.

قابلیت `noexcept` از نظر مفهومی دو کاربرد اصلی دارد:

* اعلام exception-safety contract برای یک تابع
* پرس‌وجو درباره اینکه آیا یک expression ممکن است exception پرتاب کند

این دو کاربرد با یکدیگر مرتبط‌اند، اما باید از هم تفکیک شوند.

---

## دلیل وجود noexcept

قبل از ++C11، زبان ++C از exception specifications قدیمی مانند `throw()` و `throw(Type)` پشتیبانی می‌کرد. این مکانیزم‌ها مشکلات طراحی و کاربردی متعددی داشتند و در نهایت deprecated شدند.

قابلیت `noexcept` با هدف ارائه یک مدل ساده‌تر و قابل اتکاتر طراحی شد.

مزایای اصلی `noexcept` شامل موارد زیر است:

* مشخص کردن contract مربوط به exception
* امکان تصمیم‌گیری بهتر در Generic Programming
* فعال کردن بعضی بهینه‌سازی‌های compiler
* امکان انتخاب Move Constructor به‌جای Copy Constructor در بسیاری از containerهای استاندارد
* فراهم کردن type-level information درباره exception behavior از ++C17
* ساده‌تر شدن reasoning درباره API و ownership
* جلوگیری از انتشار exception در مرزهایی که انتشار آن‌ها منطقی نیست

با این حال، استفاده از `noexcept` نباید صرفاً با هدف «سریع‌تر کردن برنامه» انجام شود. ابتدا باید contract تابع را درست تعریف کرد.

---

## مفهوم exception specification

عبارت `noexcept` بخشی از exception specification یک تابع است.

برای نمونه:

```cpp
void f() noexcept;
void g() noexcept(false);
```

تابع `f` یک non-throwing exception specification دارد، در حالی که `g` صراحتاً اجازه پرتاب exception را می‌دهد.

در واقع `noexcept(false)` از نظر semantics تقریباً معادل تابعی است که هیچ `noexcept` صریحی ندارد:

```cpp
void f();
void g() noexcept(false);
```

هر دو تابع می‌توانند exception را به caller منتقل کنند.

بنابراین نباید تصور کرد که `noexcept(false)` به معنای «تابع حتماً exception پرتاب می‌کند» است. این عبارت فقط می‌گوید که تابع از نظر language contract، non-throwing نیست.

---

## نحوه عملکرد noexcept در زمان اجرا

مهم‌ترین ویژگی `noexcept` این است که اگر exception از یک تابع non-throwing خارج شود، ++C آن را به caller منتقل نمی‌کند.

برای مثال:

```cpp
#include <stdexcept>

void process() noexcept {
    throw std::runtime_error("failure");
}
```

در این حالت exception از `process` خارج می‌شود و runtime باید `std::terminate` را فراخوانی کند.

یعنی این کد رفتار زیر را ندارد:

```cpp
try {
    process();
} catch (...) {
    // This is not reached
}
```

exception نمی‌تواند از مرز `noexcept` عبور کند.

نکته مهم این است که «تابع `noexcept` هیچ‌گاه نمی‌تواند exception ایجاد کند» گزاره دقیقی نیست. ممکن است exception درون تابع ایجاد شود؛ مسئله این است که exception نباید از تابع خارج شود.

برای نمونه:

```cpp
void process() noexcept {
    try {
        perform_operation();
    } catch (...) {
        recover_from_error();
    }
}
```

در این حالت exception ممکن است در `perform_operation` ایجاد شود، اما قبل از خروج از `process` مدیریت شده است.

---

## تفاوت noexcept و std::terminate

تابعی که با `noexcept` مشخص شده است، در صورت خروج exception باعث فراخوانی `std::terminate` می‌شود.

برای مثال:

```cpp
#include <exception>
#include <stdexcept>

void process() noexcept {
    throw std::runtime_error("unexpected failure");
}
```

در این شرایط، `std::terminate` فراخوانی می‌شود.

مکانیزم termination را می‌توان با `std::set_terminate` شخصی‌سازی کرد:

```cpp
#include <exception>
#include <cstdlib>

void terminate_handler() noexcept {
    std::abort();
}

int main() {
    std::set_terminate(terminate_handler);
}
```

البته تغییر terminate handler معمولاً راه‌حل مدیریت خطای application-level نیست. هدف اصلی `noexcept` این است که failureهای ناسازگار با contract تابع را به‌عنوان یک failure جدی مشخص کند.

---

## تفاوت noexcept با try/catch

قابلیت `noexcept` جایگزین `try/catch` نیست.

بلکه این دو مکانیزم وظایف متفاوتی دارند:

* `try/catch` برای مدیریت exception است.
* `noexcept` برای مشخص کردن مرز انتشار exception است.

برای مثال:

```cpp
void operation() noexcept {
    try {
        risky_operation();
    } catch (...) {
        handle_failure();
    }
}
```

در این مثال تابع non-throwing باقی می‌ماند، زیرا exception قبل از خروج از آن مدیریت شده است.

در مقابل:

```cpp
void operation() noexcept {
    risky_operation();
}
```

اگر `risky_operation` exception پرتاب کند و exception مدیریت نشود، termination رخ می‌دهد.

---

## معرفی noexcept operator

علاوه بر exception specification، کلمه `noexcept` می‌تواند به‌عنوان operator نیز استفاده شود.

برای مثال:

```cpp
static_assert(noexcept(42));
```

عبارت زیر یک constant expression از نوع `bool` تولید می‌کند:

```cpp
noexcept(expression)
```

اگر compiler تشخیص دهد که expression نمی‌تواند exception را به caller منتقل کند، نتیجه `true` خواهد بود؛ در غیر این صورت نتیجه `false` است.

برای نمونه:

```cpp
void safe_operation() noexcept;
void risky_operation();

static_assert(noexcept(safe_operation()));
static_assert(!noexcept(risky_operation()));
```

این operator در Generic Programming بسیار مهم است.

---

## بررسی noexcept برای expressionها

نتیجه `noexcept(...)` به expression مربوط است، نه صرفاً به اسم تابع.

برای مثال:

```cpp
void f() noexcept;

struct Widget {
    void run() noexcept;
};

Widget widget;

static_assert(noexcept(f()));
static_assert(noexcept(widget.run()));
```

همچنین constructor، destructor، conversion و operatorها نیز می‌توانند در چنین بررسی‌هایی قرار بگیرند:

```cpp
struct Widget {
    Widget() noexcept = default;
    ~Widget() noexcept = default;
};

static_assert(noexcept(Widget{}));
```

این ویژگی امکان نوشتن templateهایی را فراهم می‌کند که behavior خود را بر اساس exception-safety عملیات داخلی تنظیم می‌کنند.

---

## کاربرد conditional noexcept

یکی از مهم‌ترین الگوهای مدرن در ++C، استفاده از conditional `noexcept` است.

برای مثال:

```cpp
template <typename T>
void swap_values(T& a, T& b) noexcept(noexcept(T(std::move(a)))) {
    T temp(std::move(a));
    a = std::move(b);
    b = std::move(temp);
}
```

در این مثال وضعیت `noexcept` تابع بر اساس operation داخلی تعیین می‌شود.

این الگو برای generic code بسیار مهم است، زیرا نویسنده template نباید بدون دلیل فرض کند که تمام typeها non-throwing هستند.

---

## کاربرد noexcept در Move Constructor

یکی از مهم‌ترین کاربردهای عملی `noexcept` در Move Semantics است.

فرض کنید یک type دارای Move Constructor باشد:

```cpp
class Buffer {
public:
    Buffer(Buffer&& other) noexcept;
};
```

در چنین شرایطی containerهایی مانند `std::vector` می‌توانند در بسیاری از عملیات‌های خود از move به‌شکل مؤثرتری استفاده کنند.

دلیل این موضوع exception guarantee است. هنگام رشد `std::vector`، container ممکن است مجبور شود عناصر قبلی را به storage جدید منتقل کند. اگر move operation ممکن است exception پرتاب کند، صرفاً move کردن ممکن است وضعیت قبلی را از بین ببرد و بازگرداندن state قبلی دشوار شود.

به همین دلیل standard library در موارد مناسب ترجیح می‌دهد از move operation غیرقابل‌پرتاب استفاده کند.

نمونه ساده:

```cpp
#include <vector>

class Widget {
public:
    Widget() = default;

    Widget(const Widget&) = default;

    Widget(Widget&&) noexcept = default;
};

int main() {
    std::vector<Widget> widgets;

    widgets.emplace_back();
    widgets.emplace_back();
}
```

در اینجا `noexcept` بودن Move Constructor یک property مهم برای تعامل type با standard library است.

---

## تفاوت Copy و Move هنگام رشد vector

اگر type دارای Copy Constructor و Move Constructor باشد، انتخاب بین آن‌ها ممکن است به `noexcept` بودن Move Constructor وابسته باشد.

برای مثال:

```cpp
class Widget {
public:
    Widget(const Widget&);
    Widget(Widget&&) noexcept;
};
```

این وضعیت معمولاً برای containerها بسیار مطلوب است.

اما اگر Move Constructor این‌گونه باشد:

```cpp
class Widget {
public:
    Widget(const Widget&);
    Widget(Widget&&);
};
```

یعنی Move Constructor non-throwing بودن خود را تضمین نکند، standard library ممکن است در شرایطی به Copy Constructor متوسل شود تا exception guarantee مناسب حفظ شود.

این رفتار یکی از دلایل مهمی است که چرا Move Constructorهای واقعاً non-throwing بهتر است `noexcept` باشند.

---

## معرفی std::move_if_noexcept

کتابخانه استاندارد برای همین سناریو الگویی تحت عنوان `std::move_if_noexcept` دارد.

این utility در `<utility>` تعریف شده است:

```cpp
#include <utility>

auto value = std::move_if_noexcept(object);
```

این utility در شرایط مناسب یک rvalue برای move کردن تولید می‌کند و در شرایطی که move ممکن است exception ایجاد کند و copy موجود باشد، می‌تواند به copy preference بدهد.

این مکانیزم به‌ویژه در implementationهای generic و containerها اهمیت دارد.

---

## مدیریت implicit noexcept در special member functionها

بعضی special member functionها ممکن است به‌صورت implicit دارای exception specification باشند.

برای مثال:

```cpp
struct Point {
    int x;
    int y;
};
```

در اینجا compiler می‌تواند exception specification مربوط به عملیات‌هایی مانند default constructor، copy constructor، move constructor و destructor را بر اساس memberها و base classها تعیین کند.

این ویژگی برای طراحی typeهای composable بسیار مهم است.

اگر تمام اجزای یک type non-throwing باشند، عملیات implicit مربوط به آن type نیز می‌تواند non-throwing باشد.

---

## بررسی noexcept در typeهای مرکب

در Generic Programming بهتر است به‌جای فرض کردن behavior typeها، آن را به‌صورت compile-time بررسی کنیم.

برای مثال:

```cpp
template <typename T>
void process(T& value) noexcept(noexcept(value.reset())) {
    value.reset();
}
```

در اینجا اگر `reset` برای type موردنظر non-throwing باشد، `process` نیز non-throwing خواهد بود.

این الگو باعث می‌شود exception specification یک wrapper با operation داخلی هماهنگ باقی بماند.

---

## استفاده از type traits

کتابخانه استاندارد type traitهایی برای بررسی properties مرتبط با exception behavior فراهم می‌کند.

برای مثال:

```cpp
#include <type_traits>

static_assert(std::is_nothrow_move_constructible_v<int>);
```

همچنین traitهای مرتبط دیگری مانند موارد زیر وجود دارند:

```cpp
std::is_nothrow_constructible
std::is_nothrow_assignable
std::is_nothrow_copy_constructible
std::is_nothrow_move_constructible
std::is_nothrow_copy_assignable
std::is_nothrow_move_assignable
std::is_nothrow_swappable
```

این traitها بخشی از standard library هستند و با `noexcept` operator مکمل یکدیگر محسوب می‌شوند.

---

## کاربرد در Generic Programming

یکی از بهترین کاربردهای `noexcept` در template code، انتقال exception guarantee از عملیات داخلی به API عمومی است.

برای مثال:

```cpp
template <typename T>
void reset(T& object) noexcept(noexcept(object.reset())) {
    object.reset();
}
```

این طراحی به‌جای hard-code کردن رفتار exception، آن را به type واقعی وابسته می‌کند.

در library design این رویکرد معمولاً بهتر از نوشتن مواردی مانند زیر است:

```cpp
template <typename T>
void reset(T& object) noexcept {
    object.reset();
}
```

زیرا نسخه دوم ممکن است contract نادرستی ایجاد کند.

اگر `object.reset()` بتواند exception پرتاب کند، نسخه دوم باعث termination خواهد شد.

---

## تفاوت noexcept(true) و noexcept

این دو شکل از نظر معنایی یکسان‌اند:

```cpp
void f() noexcept;
void g() noexcept(true);
```

هر دو تابع non-throwing هستند.

با این حال، شکل ساده‌تر معمولاً خواناتر است:

```cpp
void f() noexcept;
```

استفاده از `noexcept(true)` زمانی می‌تواند مفید باشد که exception specification به‌صورت generic یا conditional نوشته می‌شود:

```cpp
template <typename T>
void process(T& value) noexcept(noexcept(value.process()));
```

---

## تفاوت noexcept(false) و حذف noexcept

این دو از نظر exception specification عملاً یک contract throwing ایجاد می‌کنند:

```cpp
void f();
void g() noexcept(false);
```

تابع بدون `noexcept` معمولاً شکل طبیعی‌تر و خواناتری دارد.

بنابراین استفاده از `noexcept(false)` معمولاً زمانی ارزش دارد که بخشی از یک expression شرطی یا declaration generic باشد.

---

## محدودیت‌های noexcept

قابلیت `noexcept` تضمین نمی‌کند که هیچ failure یا errorی در تابع رخ ندهد.

برای مثال:

```cpp
void process() noexcept {
    allocate_resource();
}
```

حتی اگر تابع `noexcept` باشد، ممکن است `allocate_resource` با failure روبه‌رو شود.

اگر failure به شکل exception نمایان شود و مدیریت نشود، termination رخ خواهد داد.

بنابراین `noexcept` به معنای موارد زیر نیست:

* تابع همیشه موفق است.
* تابع هیچ خطایی ندارد.
* تابع هیچ operation خطرناکی انجام نمی‌دهد.
* compiler تضمین می‌کند تابع سریع‌تر اجرا شود.
* هیچ exceptionای در هیچ نقطه‌ای از تابع ایجاد نمی‌شود.

`noexcept` فقط contract مربوط به خروج exception از تابع را مشخص می‌کند.

---

## بررسی تأثیر noexcept بر بهینه‌سازی

یکی از انگیزه‌های اولیه برای استفاده از exception specificationهای non-throwing، امکان optimization است؛ اما نباید `noexcept` را به‌عنوان optimization directive در نظر گرفت.

ممکن است compiler از اطلاعات `noexcept` برای تولید code بهتر استفاده کند، اما استاندارد ++C تضمین نمی‌کند که اضافه کردن `noexcept` الزاماً باعث سریع‌تر شدن تابع شود.

بنابراین تصمیم اصلی باید بر اساس correctness و API contract باشد.

قاعده مناسب این است:

> ابتدا مشخص کنید آیا تابع واقعاً باید non-throwing باشد؛ سپس `noexcept` را اعمال کنید.

نه اینکه:

> `noexcept` را اضافه کنید تا compiler سریع‌تر code تولید کند.

---

## مدیریت عملیات‌هایی که واقعاً exception نمی‌پرانند

توابعی که ذاتاً operationهای non-throwing انجام می‌دهند، کاندیدای خوبی برای `noexcept` هستند.

نمونه رایج، swap برای typeهای مناسب است:

```cpp
class Buffer {
public:
    void swap(Buffer& other) noexcept {
        // Swap internal resources
    }
};
```

همچنین مواردی مانند destructor، move operationهای ساده و عملیات روی pointerها یا primitiveها اغلب می‌توانند non-throwing باشند.

اما باید implementation واقعی را بررسی کرد.

---

## اهمیت noexcept برای Destructor

Destructorها نقش ویژه‌ای دارند.

در طراحی مدرن ++C، destructor نباید exception را از خود خارج کند.

برای مثال:

```cpp
class Resource {
public:
    ~Resource() noexcept {
        release_resource();
    }
};
```

اگر destructor هنگام stack unwinding exception پرتاب کند، وضعیت بسیار خطرناکی ایجاد می‌شود؛ به‌خصوص اگر exception دیگری در حال unwind شدن باشد.

در چنین شرایطی termination رخ می‌دهد.

به همین دلیل، destructor باید exception را در داخل خود مدیریت کند یا از ابتدا طوری طراحی شود که عملیات آن non-throwing باشد.

---

## نکته مهم درباره Destructorهای implicit

اگر destructor به‌صورت implicit تولید شود، exception specification آن بر اساس subobjectها تعیین می‌شود.

برای مثال:

```cpp
struct Resource {
    Resource() = default;
    ~Resource() = default;
};
```

اگر destructor اعضای type موردنظر non-throwing باشند، destructor ضمنی نیز می‌تواند non-throwing باشد.

این behavior یکی از مزایای composition در طراحی مدرن ++C است.

---

## noexcept و Override در توابع virtual

در inheritance، exception specification تابع overriding باید با contract تابع پایه سازگار باشد.

برای مثال:

```cpp
struct Base {
    virtual void process() noexcept;
};

struct Derived : Base {
    void process() noexcept override;
};
```

اما این طراحی مشکل‌دار است:

```cpp
struct Base {
    virtual void process() noexcept;
};

struct Derived : Base {
    void process() override;
};
```

تابع `Derived::process` نمی‌تواند contract non-throwing تابع پایه را ضعیف‌تر کند.

به بیان دیگر، اگر تابع پایه `noexcept` باشد، override باید نیز با آن contract سازگار باشد.

---

## تفاوت virtual throwing و non-throwing

اگر تابع پایه throwing باشد، override می‌تواند non-throwing باشد:

```cpp
struct Base {
    virtual void process();
};

struct Derived : Base {
    void process() noexcept override;
};
```

این طراحی مجاز است، زیرا تابع derived محدودیت بیشتری روی exception behavior اعمال می‌کند.

اما برعکس آن مجاز نیست:

```cpp
struct Base {
    virtual void process() noexcept;
};

struct Derived : Base {
    void process() override;
};
```

زیرا derived نمی‌تواند contract پایه را تضعیف کند.

---

## تغییرات noexcept در ++C17

یکی از تغییرات مهم ++C17 این است که exception specification بخشی از function type محسوب می‌شود.

برای نمونه:

```cpp
using SafeFunction = void() noexcept;
using RegularFunction = void();
```

این دو function type یکسان نیستند.

این تفاوت در function pointerها نیز اهمیت دارد:

```cpp
void safe_function() noexcept;

void (*ptr)() noexcept = safe_function;
```

در ++C17 و نسخه‌های بعدی، این property بخشی از type system محسوب می‌شود.

---

## تبدیل function pointerها

یک pointer به تابع non-throwing می‌تواند در شرایط مشخصی به pointer به تابع potentially-throwing تبدیل شود:

```cpp
void safe_function() noexcept;

void (*regular_ptr)() = safe_function;
```

اما تبدیل معکوس مجاز نیست:

```cpp
void regular_function();

void (*safe_ptr)() noexcept = regular_function;
```

این تبدیل منطقی است، زیرا caller نمی‌تواند یک potentially-throwing function را به‌عنوان تابعی که هرگز exception خارج نمی‌کند، استفاده کند.

---

## noexcept در Lambdaها

Lambdaها نیز می‌توانند `noexcept` باشند:

```cpp
auto operation = []() noexcept {
    // Non-throwing operation
};
```

همچنین می‌توان exception specification را conditional کرد:

```cpp
auto operation = [](auto& value) noexcept(noexcept(value.process())) {
    value.process();
};
```

این قابلیت در Generic Lambdaهای مدرن بسیار مفید است.

---

## noexcept در Function Objectها

برای function object نیز می‌توان `operator()` را non-throwing کرد:

```cpp
struct Processor {
    void operator()() noexcept {
        // Process data
    }
};
```

این موضوع می‌تواند هنگام استفاده از object در generic algorithmها و utilityهای مختلف اهمیت داشته باشد.

---

## noexcept در Move Assignment

`noexcept` فقط برای Move Constructor مهم نیست.

Move Assignment نیز ممکن است non-throwing باشد:

```cpp
class Buffer {
public:
    Buffer& operator=(Buffer&& other) noexcept {
        // Transfer ownership
        return *this;
    }
};
```

برای typeهای resource-owning، non-throwing بودن move operations معمولاً property ارزشمندی است.

---

## noexcept در swap

برای typeهای قابل swap، non-throwing بودن `swap` نیز مهم است.

یک الگوی رایج:

```cpp
class Widget {
public:
    void swap(Widget& other) noexcept {
        // Swap state
    }
};

void swap(Widget& a, Widget& b) noexcept {
    a.swap(b);
}
```

در Generic Programming می‌توان exception behavior مربوط به swap را نیز بررسی کرد:

```cpp
static_assert(std::is_nothrow_swappable_v<Widget>);
```

این property می‌تواند در طراحی algorithmها و containerهای generic مؤثر باشد.

---

## استفاده از noexcept در Concepts و requires

در ++C20 می‌توان `noexcept` را با Concepts و `requires` ترکیب کرد.

برای مثال:

```cpp
template <typename T>
concept NothrowResettable =
    requires(T& value) {
        { value.reset() } noexcept;
    };

template <NothrowResettable T>
void reset(T& value) noexcept {
    value.reset();
}
```

در اینجا constraint مشخص می‌کند که operation موردنظر باید وجود داشته باشد و non-throwing باشد.

این روش از بسیاری از traitهای دستی خواناتر است و با مدل Concepts در ++C20 سازگاری خوبی دارد.

---

## تفاوت noexcept و Contracts

قابلیت `noexcept` نباید با contractهای عمومی تابع اشتباه گرفته شود.

برای مثال:

```cpp
void transfer(Resource& resource) noexcept;
```

این declaration درباره exception behavior صحبت می‌کند، نه درباره اینکه آیا `resource` معتبر است یا آیا preconditionهای تابع رعایت شده‌اند.

به‌طور مفهومی:

* `noexcept` درباره انتشار exception است.
* precondition درباره شرایط معتبر برای فراخوانی تابع است.
* postcondition درباره وضعیت مورد انتظار پس از اجرای تابع است.

این مفاهیم ممکن است در یک API کنار هم قرار بگیرند، اما یکسان نیستند.

---

## استفاده از noexcept در API Design

در طراحی library، اضافه کردن `noexcept` یک تصمیم قراردادی است.

برای API عمومی بهتر است تابع را `noexcept` کنید اگر:

* واقعاً نباید exception از آن خارج شود.
* implementation می‌تواند این contract را حفظ کند.
* caller از non-throwing بودن آن سود می‌برد.
* تغییر state یا resource management به شکل قابل اطمینان انجام می‌شود.
* عدم انتشار exception بخشی از semantics طبیعی operation است.

نمونه مناسب:

```cpp
class Handle {
public:
    Handle(Handle&& other) noexcept;
    Handle& operator=(Handle&& other) noexcept;
    ~Handle() noexcept;
};
```

این نوع API برای resource-owning typeها رایج و مفید است.

---

## زمان‌هایی که نباید noexcept استفاده شود

نباید صرفاً برای «محکم‌تر» یا «سریع‌تر» نشان دادن API، `noexcept` اضافه کرد.

برای مثال اگر تابع واقعاً می‌تواند failure را با exception به caller منتقل کند:

```cpp
void load_file(const std::string& path);
```

اضافه کردن `noexcept` بدون مدیریت exception داخلی می‌تواند contract خطرناکی ایجاد کند:

```cpp
void load_file(const std::string& path) noexcept {
    // Potentially throwing operations
}
```

اگر allocation، parsing یا I/O در این تابع exception پرتاب کند و مدیریت نشود، نتیجه termination خواهد بود.

---

## اهمیت exception safety در طراحی

هنگام تصمیم‌گیری درباره `noexcept` باید exception-safety guarantee را در نظر گرفت.

سه مفهوم رایج عبارت‌اند از:

* **No-throw guarantee**: operation exception را از خود خارج نمی‌کند.
* **Strong guarantee**: در صورت failure، state observable برنامه بدون تغییر باقی می‌ماند.
* **Basic guarantee**: invariantهای object حفظ می‌شوند و resource leak رخ نمی‌دهد.

`noexcept` مستقیماً بیانگر no-throw guarantee است، اما strong و basic guarantee مفاهیم گسترده‌تری هستند.

برای مثال:

```cpp
void commit() noexcept;
```

این declaration می‌تواند بیانگر no-throw بودن `commit` باشد، اما به‌تنهایی نمی‌گوید operation چه state transitionای انجام می‌دهد.

---

## ارتباط noexcept با RAII

در طراحی RAII، destructor باید بتواند resource را بدون خارج کردن exception آزاد کند.

برای نمونه:

```cpp
class File {
public:
    ~File() noexcept {
        close_file();
    }

private:
    void close_file() noexcept {
        // Release operating-system resource
    }
};
```

اگر cleanup واقعاً ممکن است failure ایجاد کند، طراحی باید مشخص کند که این failure چگونه مدیریت می‌شود.

در بسیاری از موارد، گزارش failure از destructor مناسب نیست و باید operationهایی مانند `close` یا `commit` به‌صورت صریح توسط caller انجام شوند.

---

## الگوی Explicit Commit

در resource management، یکی از الگوهای مفید این است که operation حساس قبل از destruction انجام شود.

برای مثال:

```cpp
class Transaction {
public:
    void commit();
    ~Transaction() noexcept;
};
```

در این طراحی، `commit` می‌تواند exception پرتاب کند، اما destructor فقط cleanup را انجام می‌دهد.

این الگو معمولاً بهتر از آن است که destructor را مجبور کنیم failureهای business-level را گزارش کند.

---

## noexcept و Allocation

وجود `noexcept` جلوی exceptionهای allocation را به‌صورت خودکار نمی‌گیرد.

برای مثال:

```cpp
void process() noexcept {
    std::vector<int> values(1000000);
}
```

اگر allocation باعث `std::bad_alloc` شود و exception مدیریت نشود، چون exception نمی‌تواند از `process` خارج شود، termination رخ می‌دهد.

بنابراین نباید فرض کرد `noexcept` باعث می‌شود allocationها non-throwing شوند.

---

## استفاده از nothrow allocation

کتابخانه استاندارد شکل دیگری از allocation را نیز فراهم می‌کند:

```cpp
int* value = new (std::nothrow) int(42);
```

در این حالت allocation failure با `nullptr` گزارش می‌شود، نه با `std::bad_alloc`.

اما این موضوع مستقل از `noexcept` است.

به بیان دقیق‌تر، `noexcept` exception specification یک تابع است، در حالی که `std::nothrow` رفتار یک allocation operation را تغییر می‌دهد.

---

## تفاوت noexcept با std::nothrow

این دو مفهوم گاهی با یکدیگر اشتباه گرفته می‌شوند.

`noexcept`:

```cpp
void process() noexcept;
```

رفتار exception specification یک تابع را مشخص می‌کند.

`std::nothrow`:

```cpp
new (std::nothrow) Widget;
```

نحوه گزارش failure در allocation را تغییر می‌دهد.

بنابراین این دو جایگزین یکدیگر نیستند.

---

## noexcept و Constructor

Constructor نیز می‌تواند `noexcept` باشد:

```cpp
class Point {
public:
    Point(int x, int y) noexcept
        : x_(x), y_(y) {}

private:
    int x_;
    int y_;
};
```

اما باید دقت کرد که initializerها نیز باید با این contract سازگار باشند.

اگر construction یکی از memberها exception پرتاب کند، exception نمی‌تواند از constructor خارج شود و termination رخ خواهد داد.

---

## noexcept و Conversion Operator

Conversion operator نیز می‌تواند non-throwing باشد:

```cpp
class Identifier {
public:
    operator int() const noexcept {
        return value_;
    }

private:
    int value_;
};
```

این property در generic code و overload resolution می‌تواند مفید باشد.

---

## noexcept و Defaulted Functions

برای defaulted special member functionها می‌توان `noexcept` صریح نیز تعیین کرد:

```cpp
struct Widget {
    Widget(Widget&&) noexcept = default;
};
```

در این حالت نویسنده API contract صریحی ارائه می‌کند.

با این حال، اگر اعضای type چنین operationای را پشتیبانی نکنند، compiler ممکن است نتواند definition موردنظر را به شکل سازگار تولید کند.

به همین دلیل، بهتر است قبل از تعیین `noexcept` صریح، properties اعضا و base classها بررسی شوند.

---

## noexcept و Deleted Functions

قابلیت `noexcept` را می‌توان همراه declarationهای deleted نیز مشاهده کرد:

```cpp
struct Widget {
    Widget(const Widget&) = delete;
    Widget(Widget&&) noexcept = default;
};
```

این الگو برای typeهایی که باید move-only باشند بسیار رایج است.

---

## noexcept و Function Pointer

در ++C17، تفاوت function typeهای throwing و non-throwing در function pointerها نیز قابل مشاهده است.

برای مثال:

```cpp
using SafeFunction = void (*)() noexcept;
using RegularFunction = void (*)();

void safe_function() noexcept;

SafeFunction safe = safe_function;
RegularFunction regular = safe_function;
```

این تفاوت می‌تواند در طراحی callback APIها اهمیت داشته باشد.

---

## noexcept و std::function

در استفاده از `std::function` باید توجه داشت که نوع `std::function` به شکل معمول exception specification `noexcept` را به‌عنوان بخشی از signature عمومی خود encode نمی‌کند.

بنابراین نباید فرض کرد:

```cpp
std::function<void()> callback;
```

به‌معنای داشتن یک callback ذاتاً non-throwing است.

اگر contract مربوط به non-throwing بودن callback برای طراحی API اهمیت دارد، می‌توان از templateها، Concepts، function pointerهای مناسب یا typeهای اختصاصی استفاده کرد.

---

## noexcept در Algorithmها

بعضی algorithmها و utilityهای standard library می‌توانند از properties نوع مورد استفاده خود بهره ببرند.

به همین دلیل، داشتن move constructor یا swap غیرقابل‌پرتاب می‌تواند behavior کلی generic algorithm را بهتر کند.

برای نمونه، طراحی type به‌شکل زیر معمولاً مطلوب است:

```cpp
class Widget {
public:
    Widget(Widget&&) noexcept;
    Widget& operator=(Widget&&) noexcept;

    void swap(Widget&) noexcept;
};
```

این ویژگی‌ها type را برای استفاده در containerها و algorithmهای استاندارد مناسب‌تر می‌کنند.

---

## noexcept و Value Categories

در expression زیر:

```cpp
noexcept(std::move(value))
```

باید توجه کرد که خود `std::move` معمولاً non-throwing است، اما operation بعدی ممکن است چنین نباشد.

برای مثال:

```cpp
noexcept(std::move(value))
```

با این مورد تفاوت دارد:

```cpp
noexcept(T(std::move(value)))
```

عبارت دوم construction واقعی را نیز در نظر می‌گیرد.

بنابراین هنگام استفاده از `noexcept` operator باید دقیقاً expression موردنظر را بررسی کرد، نه فقط یک operation مقدماتی مانند `std::move`.

---

## noexcept و Overload Resolution

وجود `noexcept` معمولاً معیار مستقیم overload resolution به معنای «این overload همیشه ترجیح داده می‌شود» نیست.

برای مثال، صرفاً non-throwing بودن یک function overload باعث نمی‌شود compiler همیشه آن overload را انتخاب کند.

با این حال، از ++C17 به بعد exception specification بخشی از function type است و در type compatibility و برخی contextهای مرتبط اثر دارد.

بنابراین باید بین «جزء type بودن» و «معیار انتخاب overload بودن» تفاوت قائل شد.

---

## نکات مهم درباره noexcept در Templateها

در templateها، این طراحی خطرناک است:

```cpp
template <typename T>
void process(T& value) noexcept {
    value.process();
}
```

اگر `T::process` بتواند exception پرتاب کند، wrapper contract نادرستی دارد.

طراحی بهتر:

```cpp
template <typename T>
void process(T& value) noexcept(noexcept(value.process())) {
    value.process();
}
```

در این حالت wrapper behavior را به operation واقعی وابسته می‌کند.

این الگو یکی از مهم‌ترین کاربردهای conditional `noexcept` در library code است.

---

## پیاده‌سازی Conditional noexcept در Generic Code

برای یک wrapper کامل‌تر می‌توان از `std::forward` نیز استفاده کرد:

```cpp
#include <utility>

template <typename T>
void invoke_process(T&& value)
    noexcept(noexcept(std::forward<T>(value).process()))
{
    std::forward<T>(value).process();
}
```

این طراحی دو property مهم را حفظ می‌کند:

* value category آرگومان
* exception specification operation داخلی

این الگو در utilityهای generic مدرن بسیار رایج است.

---

## نکات مربوط به Perfect Forwarding

در generic code، `noexcept` باید با همان expressionای بررسی شود که واقعاً اجرا خواهد شد.

برای مثال، این دو الزاماً equivalent نیستند:

```cpp
noexcept(value.process())
```

و:

```cpp
noexcept(std::forward<T>(value).process())
```

اگر overloadهای مختلف برای lvalue و rvalue وجود داشته باشند، ممکن است operation انتخاب‌شده متفاوت باشد.

بنابراین در Perfect Forwarding، بهتر است `noexcept` expression دقیقاً expression اجرایی را منعکس کند.

---

## خطای رایج در استفاده بیش از حد از noexcept

یکی از اشتباهات رایج این است که توسعه‌دهنده تصور کند هر تابع کوچک باید `noexcept` باشد.

برای مثال:

```cpp
std::string normalize(std::string value) noexcept;
```

اگر implementation ممکن است allocation انجام دهد، این declaration می‌تواند بسیار خطرناک باشد.

در چنین حالتی exception ممکن است طبیعی‌ترین روش گزارش failure باشد.

`noexcept` باید بر اساس semantics و contract انتخاب شود، نه بر اساس اندازه یا سادگی ظاهری تابع.

---

## خطای رایج در Move Constructor

یکی از مهم‌ترین اشتباه‌ها، فراموش کردن `noexcept` برای Move Constructorهایی است که واقعاً non-throwing هستند.

برای مثال:

```cpp
class Buffer {
public:
    Buffer(Buffer&& other);
};
```

اگر انتقال ownership صرفاً شامل pointer و metadata باشد و هیچ operation throwکننده‌ای وجود نداشته باشد، بهتر است contract صریح باشد:

```cpp
class Buffer {
public:
    Buffer(Buffer&& other) noexcept;
};
```

این کار می‌تواند interaction type با standard library را بهبود دهد.

---

## خطای رایج در noexcept دروغین

اشتباه جدی‌تر، اعلام `noexcept` برای تابعی است که واقعاً می‌تواند exception را بدون مدیریت از خود خارج کند:

```cpp
void process() noexcept {
    operation_that_can_throw();
}
```

این طراحی exception را «حل» نمی‌کند؛ فقط failure را از exception propagation به termination تبدیل می‌کند.

بنابراین `noexcept` نباید برای پنهان کردن exception behavior استفاده شود.

---

## خطای رایج در Destructor

این طراحی بسیار خطرناک است:

```cpp
class Resource {
public:
    ~Resource() noexcept {
        risky_cleanup();
    }
};
```

اگر `risky_cleanup` exception پرتاب کند، termination رخ خواهد داد.

راهکار بهتر این است که cleanup destructor ذاتاً non-throwing باشد یا failureهای قابل مدیریت قبل از destructor و در یک operation صریح مدیریت شوند.

---

## خطای رایج در Catch کردن exception پس از noexcept

این تصور اشتباه است:

```cpp
void process() noexcept {
    throw std::runtime_error("error");
}

int main() {
    try {
        process();
    } catch (...) {
        // Expected recovery
    }
}
```

exception قبل از رسیدن به caller باعث termination می‌شود.

اگر recovery در سطح caller لازم است، تابع نباید exception را از طریق `noexcept` مسدود کند، مگر اینکه خودش exception را مدیریت و به شکل دیگری گزارش کند.

---

## خطای رایج در استفاده از noexcept به‌عنوان optimization hint

این تصور که:

```cpp
void calculate() noexcept;
```

حتماً از:

```cpp
void calculate();
```

سریع‌تر اجرا می‌شود، صحیح نیست.

ممکن است compiler از information مربوط به non-throwing بودن استفاده کند، اما `noexcept` یک performance annotation ساده نیست.

تصمیم باید بر اساس correctness و contract باشد.

---

## خطای رایج در نادیده گرفتن ABI

در libraryهای بزرگ، تغییر exception specification می‌تواند روی compatibility و function type اثر بگذارد، به‌خصوص از ++C17 به بعد.

برای مثال، در interface عمومی یک library، تغییر بین:

```cpp
void process();
```

و:

```cpp
void process() noexcept;
```

ممکن است صرفاً یک تغییر cosmetic نباشد.

تأثیر دقیق ABI به compiler، platform، ABI implementation و نحوه استفاده از symbolها بستگی دارد؛ بنابراین نباید درباره سازگاری binary بدون بررسی toolchain مربوطه فرضی کلی ارائه کرد.

---

## مقایسه noexcept با throw()

در ++C++11، `throw()` معنای non-throwing specification داشت، اما این syntax بعدها deprecated شد.

کد قدیمی ممکن است شامل موارد زیر باشد:

```cpp
void process() throw();
```

در کد مدرن بهتر است از:

```cpp
void process() noexcept;
```

استفاده شود.

در استانداردهای جدید، exception specificationهای dynamic مانند:

```cpp
void process() throw(std::runtime_error);
```

دیگر بخشی از روش مدرن طراحی ++C نیستند و نباید در کد جدید استفاده شوند.

---

## تفاوت noexcept با dynamic exception specification

مدل قدیمی می‌توانست نوع exceptionهای مجاز را مشخص کند:

```cpp
void process() throw(std::runtime_error);
```

این مدل با `noexcept` متفاوت است.

`noexcept` اساساً روی این سؤال تمرکز می‌کند:

> آیا exception می‌تواند از این تابع خارج شود؟

و نه:

> کدام نوع exceptionها اجازه خروج دارند؟

این ساده‌سازی یکی از مزایای مهم طراحی جدید است.

---

## وضعیت noexcept در استانداردهای مختلف

قابلیت `noexcept` در ++C11 معرفی شد.

در ++C14، استفاده عمومی از آن ادامه پیدا کرد و به یکی از ابزارهای مهم exception specification تبدیل شد.

در ++C17، exception specification به‌عنوان بخشی از function type اهمیت type-system بیشتری پیدا کرد.

در ++C20، `noexcept` در کنار Concepts، requires-expressions و Generic Programming مدرن کاربرد بیشتری پیدا کرد.

در ++C23 و ++C26 نیز `noexcept` همچنان بخشی از مدل استاندارد exception handling زبان است و syntax بنیادی آن تغییر نکرده است.

---

## noexcept و constexpr

قابلیت `constexpr` و `noexcept` دو property مستقل هستند و می‌توانند کنار هم استفاده شوند:

```cpp
constexpr int square(int value) noexcept {
    return value * value;
}
```

`constexpr` درباره امکان evaluation در compile time صحبت می‌کند، در حالی که `noexcept` درباره exception propagation صحبت می‌کند.

وجود یکی به‌صورت خودکار به معنای وجود دیگری نیست.

---

## noexcept و consteval

همین تفکیک در مورد `consteval` نیز برقرار است:

```cpp
consteval int square(int value) noexcept {
    return value * value;
}
```

`consteval` الزام compile-time evaluation را مشخص می‌کند، در حالی که `noexcept` exception behavior را مشخص می‌کند.

---

## noexcept و Coroutineها

در ++C20، coroutine functionها نیز می‌توانند `noexcept` باشند.

برای مثال:

```cpp
Task process() noexcept {
    co_await perform_operation();
}
```

اما در coroutineها باید semantics مربوط به `promise_type`، exception handling و coroutine machinery را جداگانه در نظر گرفت.

`noexcept` به این معنا نیست که هیچ failureای در coroutine وجود ندارد؛ بلکه exception نباید از contract تعیین‌شده برای coroutine عبور کند.

به‌خصوص در طراحی coroutineهای library-level، باید مشخص شود exception چگونه به `promise_type` یا مکانیزم error reporting منتقل می‌شود.

---

## noexcept و Asynchronous Programming

در APIهای asynchronous، non-throwing بودن operation الزاماً به معنای عدم وجود failure نیست.

برای مثال، یک API می‌تواند:

```cpp
Future<Result> start_operation() noexcept;
```

باشد و failure را از طریق `Result` یا state مربوط به asynchronous operation گزارش کند.

این طراحی در شرایطی مناسب است که failure بخشی از نتیجه طبیعی operation باشد و نباید از مسیر exception propagation عبور کند.

اما انتخاب این مدل باید بر اساس semantics API انجام شود.

---

## طراحی API با Result به‌جای Exception

در بعضی سیستم‌ها، به‌خصوص codeهای low-level یا latency-sensitive، ممکن است تصمیم گرفته شود failureها از طریق result type منتقل شوند:

```cpp
Result process() noexcept;
```

در این حالت `noexcept` می‌تواند کاملاً طبیعی باشد.

اما نباید `noexcept` را به‌تنهایی دلیل استفاده از Result type دانست. این دو تصمیم مستقل هستند:

* انتخاب error model
* تعیین exception specification

---

## مرزهای مناسب برای noexcept

یکی از کاربردهای خوب `noexcept` تعیین مرزهایی است که exception نباید از آن‌ها عبور کند.

نمونه‌های رایج:

* destructorها
* move constructorهای واقعاً non-throwing
* move assignmentهای واقعاً non-throwing
* swapهای non-throwing
* عملیات cleanup
* callbackهای مشخصاً non-throwing
* utilityهای low-level که contract آن‌ها no-throw است

در مقابل، عملیات‌هایی مانند parsing، allocation، I/O و resource acquisition اغلب باید با دقت بیشتری بررسی شوند.

---

## نکات مهم برای Library Authorها

برای نویسنده یک library عمومی، `noexcept` بخشی از API contract است.

پیش از اضافه کردن آن باید این پرسش‌ها بررسی شوند:

* آیا تمام operationهای داخلی با contract سازگارند؟
* آیا memberها و base classها non-throwing هستند؟
* آیا تغییر state در صورت failure قابل مدیریت است؟
* آیا caller از no-throw بودن operation سود می‌برد؟
* آیا این property برای move semantics اهمیت دارد؟
* آیا تغییر exception specification روی API یا ABI اثر می‌گذارد؟
* آیا generic wrapper باید conditional `noexcept` باشد؟

پاسخ دادن به این پرسش‌ها معمولاً بهتر از استفاده مکانیکی از `noexcept` است.

---

## الگوی پیشنهادی برای Typeهای Resource-Owning

یک type که ownership یک resource را مدیریت می‌کند، اغلب باید تا حد امکان move-friendly و non-throwing طراحی شود:

```cpp
class Resource {
public:
    Resource(Resource&& other) noexcept;
    Resource& operator=(Resource&& other) noexcept;

    Resource(const Resource&) = delete;
    Resource& operator=(const Resource&) = delete;

    ~Resource() noexcept;
};
```

این طراحی برای handleها، pointerهای مالک، file descriptor wrapperها و بسیاری از resource abstractionها مناسب است؛ البته implementation واقعی باید contract را تضمین کند.

---

## الگوی پیشنهادی برای Wrapperهای Generic

برای wrapperهای generic، conditional `noexcept` معمولاً انتخاب مناسبی است:

```cpp
template <typename T>
auto transform(T&& value)
    noexcept(noexcept(std::forward<T>(value).transform()))
{
    return std::forward<T>(value).transform();
}
```

این طراحی اجازه می‌دهد wrapper بدون تحمیل contract اشتباه، behavior type داخلی را حفظ کند.

---

## بررسی noexcept در تست‌های Compile-Time

می‌توان contract موردنظر را با `static_assert` آزمایش کرد:

```cpp
#include <type_traits>

static_assert(std::is_nothrow_move_constructible_v<Widget>);
static_assert(std::is_nothrow_move_assignable_v<Widget>);
static_assert(noexcept(std::declval<Widget&>().swap(
    std::declval<Widget&>()
)));
```

این نوع تست‌ها برای library code بسیار مفید هستند، زیرا تغییر ناخواسته exception specification را در زمان compile آشکار می‌کنند.

---

## بررسی noexcept با declval

تابع `std::declval` امکان بررسی expressionهای مربوط به typeهایی را فراهم می‌کند که object واقعی آن‌ها را نمی‌سازیم:

```cpp
#include <utility>

template <typename T>
concept NothrowMovable =
    requires {
        requires noexcept(T(std::declval<T&&>()));
    };
```

در کد مدرن ++C20 می‌توان این مدل را با Concepts ترکیب کرد.

---

## تفاوت noexcept و std::is_nothrow_move_constructible

این دو mechanism نتایج نزدیک ولی کاربرد متفاوتی دارند.

عبارت:

```cpp
noexcept(T(std::declval<T&&>()))
```

مستقیماً یک expression مشخص را بررسی می‌کند.

در حالی که:

```cpp
std::is_nothrow_move_constructible_v<T>
```

یک type trait سطح بالاتر است که property مربوط به move construction را بیان می‌کند.

در Generic Programming، traitها معمولاً برای بیان intent خواناتر هستند، در حالی که `noexcept(...)` برای بررسی دقیق expression بسیار قدرتمند است.

---

## نکات طراحی درباره Strong Guarantee

گاهی non-throwing کردن یک operation اشتباه است، زیرا exception می‌تواند بخشی ضروری از مکانیزم rollback باشد.

فرض کنید تابعی state پیچیده‌ای را تغییر می‌دهد:

```cpp
void update() noexcept;
```

اگر implementation برای حفظ consistency مجبور باشد operationهای throwکننده را انجام دهد، `noexcept` می‌تواند مسئله را پنهان کند و به termination منجر شود.

در چنین مواردی ممکن است طراحی مناسب‌تر این باشد:

```cpp
void update();
```

و ارائه strong exception guarantee.

بنابراین no-throw بودن همیشه بهتر از throwing بودن نیست.

---

## نکات طراحی درباره Error Handling

استفاده صحیح از `noexcept` وابسته به error model برنامه است.

اگر failure یک exceptional condition است، exception ممکن است مناسب باشد:

```cpp
void parse_document();
```

اگر failure یک نتیجه عادی operation است، ممکن است Result type مناسب‌تر باشد:

```cpp
Result parse_document() noexcept;
```

اگر operation اصولاً نباید fail کند، ممکن است `noexcept` منطقی باشد:

```cpp
void release_resource() noexcept;
```

این تصمیم‌ها باید از semantics برنامه ناشی شوند، نه از یک قاعده ساده مانند «هرچه `noexcept` بیشتر باشد، بهتر است».

---

## نکات مهم درباره Termination

زمانی که exception از `noexcept` خارج می‌شود، `std::terminate` فراخوانی می‌شود و stack unwinding معمولی تا caller ادامه پیدا نمی‌کند.

این مسئله پیامدهای جدی دارد:

* caller نمی‌تواند exception را catch کند.
* recovery معمولی ممکن نیست.
* termination handler اجرا می‌شود.
* process معمولاً در نهایت متوقف می‌شود.
* اگر diagnostic مناسب وجود نداشته باشد، علت failure ممکن است دشوار پیدا شود.

به همین دلیل استفاده نادرست از `noexcept` می‌تواند bug بسیار جدی ایجاد کند.

---

## راهنمای Debug کردن terminate ناشی از noexcept

اگر برنامه ناگهان با `std::terminate` متوقف می‌شود، یکی از مواردی که باید بررسی شود خروج exception از یک `noexcept` function است.

نمونه مشکوک:

```cpp
void process() noexcept {
    std::vector<int> values;
    values.resize(1000000);
}
```

در چنین شرایطی باید بررسی کرد که آیا operationهای داخلی می‌توانند exception پرتاب کنند.

همچنین بررسی stack trace و terminate handler می‌تواند برای پیدا کردن محل failure مفید باشد.

---

## نکات مهم برای Code Review

در review کد مربوط به `noexcept` بهتر است این موارد بررسی شوند:

* آیا `noexcept` واقعاً contract صحیحی است؟
* آیا implementation ممکن است exception را بدون catch کردن خارج کند؟
* آیا Move Constructor باید `noexcept` باشد؟
* آیا Move Assignment باید `noexcept` باشد؟
* آیا destructor non-throwing است؟
* آیا wrapper generic باید conditional `noexcept` داشته باشد؟
* آیا `noexcept` روی virtual override با base contract سازگار است؟
* آیا `noexcept` صرفاً برای optimization اضافه شده است؟
* آیا تغییر آن روی public API یا ABI اثر می‌گذارد؟
* آیا failure بهتر است با exception یا Result گزارش شود؟

---

## مقایسه الگوهای رایج

در طراحی مدرن می‌توان این الگوها را به‌صورت زیر مقایسه کرد:

| الگو                        | کاربرد اصلی                                        | توصیه                               |
| --------------------------- | -------------------------------------------------- | ----------------------------------- |
| `void f();`                 | تابعی که ممکن است exception را منتقل کند           | انتخاب پیش‌فرض برای عملیات throwing |
| `void f() noexcept;`        | تابعی که exception نباید از آن خارج شود            | فقط با contract واقعی               |
| `void f() noexcept(false);` | تابع potentially-throwing در expressionهای generic | معمولاً به‌صورت صریح لازم نیست      |
| `noexcept(expr)`            | بررسی compile-time یک expression                   | بسیار مناسب برای generic code       |
| `std::is_nothrow_*`         | بررسی property یک type                             | مناسب برای type-level constraints   |
| `std::move_if_noexcept`     | انتخاب move یا copy بر اساس exception safety       | مفید در generic/container code      |

---

## توصیه‌های عملی نهایی

برای استفاده حرفه‌ای از `noexcept` چند اصل بسیار مهم وجود دارد.

نخست، `noexcept` را بخشی از API contract بدانید، نه یک annotation تزئینی.

دوم، Move Constructor و Move Assignment را در typeهای resource-owning بررسی کنید؛ اگر واقعاً non-throwing هستند، `noexcept` معمولاً انتخاب مناسبی است.

سوم، destructorها را non-throwing نگه دارید و failureهای قابل گزارش را تا حد امکان به operationهای صریح مانند `commit` یا `close` منتقل کنید.

چهارم، در Generic Programming از conditional `noexcept` استفاده کنید تا wrapperها exception behavior واقعی operation داخلی را حفظ کنند.

پنجم، برای بررسی compile-time از `noexcept(expr)` و type traitهایی مانند `std::is_nothrow_move_constructible` استفاده کنید.

ششم، `noexcept` را صرفاً به امید optimization اضافه نکنید.

هفتم، اگر تابعی `noexcept` است، مطمئن شوید exceptionهای احتمالی درون آن یا مدیریت می‌شوند یا اصلاً امکان خروج ندارند.

هشتم، بین `noexcept` و سایر error-handling mechanismها مانند `try/catch`، Result type و `std::nothrow` تفاوت مفهومی قائل شوید.

---

## جمع‌بندی مفهوم noexcept

قابلیت `noexcept` یکی از اجزای مهم طراحی مدرن ++C است که contract مربوط به exception propagation را مشخص می‌کند.

مهم‌ترین نکته این است که `noexcept` به معنای «هیچ exceptionای در این تابع ایجاد نمی‌شود» نیست؛ بلکه به این معناست که exception نباید از مرز آن تابع عبور کند. در صورت نقض این contract، `std::terminate` فراخوانی می‌شود.

ارزش واقعی `noexcept` در ترکیب آن با بخش‌های دیگر زبان و standard library مشخص می‌شود. Move Semantics، `std::vector`، Generic Programming، Concepts، type traits، function types و RAII همگی می‌توانند از اطلاعات مربوط به non-throwing بودن عملیات استفاده کنند.

در نتیجه، رویکرد حرفه‌ای این نیست که `noexcept` را در همه‌جا اضافه کنیم؛ بلکه باید آن را جایی به کار ببریم که semantics تابع واقعاً no-throw باشد و این property برای caller یا generic infrastructure ارزش داشته باشد.


---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>