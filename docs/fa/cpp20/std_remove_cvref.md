<div align="center">

[🇺🇸 English](../../en/cpp20/std_remove_cvref.md) | [🇮🇷 فارسی](./std_remove_cvref.md)

</div>

---

# معرفی `std::remove_cvref` در ++C20

## مقدمه

در برنامه‌نویسی Generic در ++C، گاهی لازم است نوعی را که از یک پارامتر
template یا یک عبارت به دست آمده است، بدون توجه به `const`، `volatile` و
reference بودن آن بررسی کنیم. برای مثال، ممکن است یک تابع template
بخواهد تشخیص دهد که آرگومان از نوع `Widget` است؛ چه آرگومان به‌صورت
`Widget`، چه `const Widget&` و چه `Widget&&` دریافت شده باشد.

قابلیت `std::remove_cvref` که از استاندارد ++C20 به کتابخانه استاندارد
اضافه شده است، دقیقاً برای این هدف طراحی شده است. این type trait، ابتدا
reference را حذف می‌کند و سپس qualifierهای سطح بالای `const` و `volatile`
را از نوع حاصل برمی‌دارد.

این ابزار کوچک، در کدهای مبتنی بر template، `type traits`، `concepts` و
`perfect forwarding` کاربرد زیادی دارد؛ اما با `std::decay` یکسان نیست و
جایگزین مناسبی برای آن در همه موقعیت‌ها محسوب نمی‌شود.

## معرفی و تعریف استاندارد

هدر موردنیاز برای استفاده از این قابلیت، `<type_traits>` است:

``` cpp
#include <type_traits>
```

کتابخانه استاندارد دو شکل مرتبط ارائه می‌کند:

-   `std::remove_cvref<T>` یک class template است که نوع تبدیل‌شده را در
    عضو `type` قرار می‌دهد.
-   `std::remove_cvref_t<T>` یک alias template است که مستقیماً به همان
    نوع تبدیل‌شده اشاره می‌کند.

برای نمونه:

``` cpp
#include <type_traits>

static_assert(std::is_same_v<
    std::remove_cvref<const int&>::type,
    int
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<volatile int&&>,
    int
>);
```

در بیشتر کدهای روزمره، شکل کوتاه‌تر یعنی `std::remove_cvref_t` خواناتر
است. شکل class template زمانی مفید است که به ساختار یک type trait یا عضو
`type` آن نیاز داشته باشیم.

## دلیل وجود این قابلیت

### مسئله حذف reference و cv-qualifierها

در templateها، نوعی که از پارامتر deduce می‌شود الزاماً همان نوع پایه‌ای
نیست که از نظر معنایی برای منطق برنامه اهمیت دارد. برای مثال، در یک تابع
با forwarding reference، آرگومان‌های متفاوت می‌توانند نوع‌های متفاوتی برای
پارامتر template ایجاد کنند:

``` cpp
template <typename T>
void inspect(T&& value) {
    // T depends on the argument's value category and cv-qualification.
}
```

اگر تابع با یک lvalue از نوع `const int` فراخوانی شود، `T` به
`const int&` تبدیل می‌شود؛ در نتیجه نوع پارامتر `T&&` پس از reference
collapsing به `const int&` خواهد بود. اگر همان تابع با یک rvalue از نوع
`int` فراخوانی شود، `T` برابر `int` است و نوع پارامتر `int&&` خواهد بود.

گاهی منطق تابع به این تفاوت‌ها وابسته نیست و فقط می‌خواهد نوع پایه را
بررسی کند. در چنین شرایطی، `std::remove_cvref_t<T>` راهی استاندارد و
صریح برای نرمال‌سازی نوع فراهم می‌کند.

### رابطه با نیازهای Generic Programming

در Generic Programming، یک الگوریتم ممکن است با انواع مختلفی از
آرگومان‌ها کار کند، اما تصمیم‌گیری آن بر اساس نوع بدون reference و
qualifierهای سطح بالا انجام شود. حذف این مشخصه‌ها می‌تواند نوشتن شرط‌های
`static_assert`، انتخاب overload و constraintهای مبتنی بر type را ساده‌تر
کند.

نکته مهم این است که حذف cv و reference فقط برای بررسی نوع است؛ این تبدیل
نه شیء جدیدی می‌سازد، نه از شیء اصلی `const` را حذف می‌کند و نه رفتار زمان
اجرا را تغییر می‌دهد.

## نحوه کار و ترتیب تبدیل

رفتار `std::remove_cvref<T>` از نظر مفهومی معادل ترکیب دو type trait
است:

``` cpp
std::remove_cv_t<std::remove_reference_t<T>>
```

ابتدا `std::remove_reference_t`، reference را حذف می‌کند؛ سپس
`std::remove_cv_t`، qualifierهای cv را از نوع حاصل حذف می‌کند.

ترتیب این دو مرحله مهم است. برای مثال، در نوع `const int&` نمی‌توان
مستقیماً انتظار داشت که حذف cv روی خود reference، `const` متعلق به نوع
ارجاع‌شده را حذف کند. ابتدا باید reference حذف شود تا نوع `const int` به
دست آید؛ سپس `const` از آن حذف می‌شود.

جدول زیر نتیجه چند تبدیل متداول را نشان می‌دهد:

  نوع ورودی `T`          نوع حاصل از `std::remove_cvref_t<T>`
  ---------------------- --------------------------------------
  `int`                  `int`
  `const int`            `int`
  `volatile int`         `int`
  `const volatile int`   `int`
  `int&`                 `int`
  `const int&`           `int`
  `volatile int&&`       `int`
  `const int*`           `const int*`
  `int* const`           `int*`
  `const int* const&`    `const int*`

## تفاوت cv سطح بالا و cv داخلی

یکی از نکات کلیدی در استفاده از این type trait، تفاوت بین top-level
cv-qualifier و qualifierهای موجود در بخش‌های داخلی یک نوع مرکب است.

برای نمونه، در `const int*`، خود pointer به `const` اشاره می‌کند؛ اما
pointer از نظر top-level دارای `const` نیست. در مقابل، در `int* const`،
خود pointer دارای `const` است.

بنابراین:

``` cpp
#include <type_traits>

static_assert(std::is_same_v<
    std::remove_cvref_t<const int*>,
    const int*
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<int* const>,
    int*
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<const int* const&>,
    const int*
>);
```

در مثال اول، `const` متعلق به نوع pointee است و باقی می‌ماند. در مثال
دوم، `const` متعلق به خود pointer است و حذف می‌شود. در مثال سوم، ابتدا
reference و سپس top-level `const` مربوط به pointer حذف می‌شود؛ اما
`const` مربوط به `int` حفظ می‌شود.

این رفتار برای انواع مرکب مانند pointerها، member pointerها و بعضی
ترکیب‌های دیگر نیز اهمیت دارد: `remove_cvref` فقط cv-qualification سطح
بالای نوع نهایی را حذف می‌کند، نه تمام qualifierهای موجود در ساختار نوع
را.

## تفاوت با `std::decay`

`std::remove_cvref_t` و `std::decay_t` هر دو در بسیاری از موارد، نوع
بدون reference و cv را تولید می‌کنند؛ اما هدف و دامنه تبدیل آن‌ها متفاوت
است.

نوع `std::decay_t<T>` علاوه بر حذف reference و cv، تبدیل‌های مشابه
parameter adjustment در انتقال آرگومان به تابع را نیز اعمال می‌کند:

-   اگر نوع پس از حذف reference، یک array باشد، به pointer به اولین عنصر
    تبدیل می‌شود.
-   اگر نوع پس از حذف reference، یک function type باشد، به function
    pointer تبدیل می‌شود.
-   در سایر موارد، cv-qualifierهای سطح بالا حذف می‌شوند.

در مقابل، `std::remove_cvref_t<T>` ساختار array و function type را حفظ
می‌کند.

``` cpp
#include <type_traits>

using Array = const int[4];
using Function = void(double);

static_assert(std::is_same_v<
    std::remove_cvref_t<Array&>,
    const int[4]
>);

static_assert(std::is_same_v<
    std::decay_t<Array&>,
    const int*
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<Function&>,
    void(double)
>);

static_assert(std::is_same_v<
    std::decay_t<Function&>,
    void(*)(double)
>);
```

### انتخاب ابزار مناسب

  -----------------------------------------------------------------------
  نیاز                                ابزار مناسب
  ----------------------------------- -----------------------------------
  حذف reference و cv سطح بالا، با حفظ `std::remove_cvref_t`
  ساختار array و function             

  شبیه‌سازی نوع پارامتر پس از decay در `std::decay_t`
  انتقال آرگومان به تابع              

  حذف صرفاً reference                  `std::remove_reference_t`

  حذف صرفاً cv-qualifierهای سطح بالا   `std::remove_cv_t`
  -----------------------------------------------------------------------

به‌عنوان یک قاعده عملی، اگر هدف فقط یکسان‌سازی نوع برای مقایسه یا
constraint باشد و نباید array به pointer یا function به function pointer
تبدیل شود، معمولاً `std::remove_cvref_t` انتخاب دقیق‌تری است. اگر واقعاً به
semantics مربوط به decay نیاز دارید، از `std::decay_t` استفاده کنید.

## استفاده در templateها و تابع‌های عمومی

### نرمال‌سازی نوع deduced شده

یک تابع عمومی ممکن است بخواهد بر اساس نوع پایه آرگومان، رفتار متفاوتی
داشته باشد؛ بدون آنکه تفاوت lvalue و rvalue یا `const` بودن آرگومان روی
تشخیص نوع اثر بگذارد.

``` cpp
#include <type_traits>

struct Message {};

template <typename T>
void process(T&& value) {
    using ValueType = std::remove_cvref_t<T>;

    static_assert(std::is_same_v<ValueType, Message>);

    // Process the message without changing the original object's type.
}
```

در این مثال، فراخوانی تابع با `Message`، `Message&`، `const Message&` یا
`Message&&` می‌تواند به یک `ValueType` یکسان منتهی شود؛ البته تنها در
صورتی که deduction نوع `T` با آن آرگومان‌ها سازگار باشد.

### استفاده در انتخاب رفتار با `if constexpr`

در صورت نیاز به شاخه‌بندی کامپایل‌تایم، می‌توان نوع نرمال‌شده را با
`if constexpr` بررسی کرد:

``` cpp
#include <type_traits>

struct Text {};
struct Binary {};

template <typename T>
void handle(T&& value) {
    using U = std::remove_cvref_t<T>;

    if constexpr (std::is_same_v<U, Text>) {
        // Handle text data.
    } else if constexpr (std::is_same_v<U, Binary>) {
        // Handle binary data.
    } else {
        // Handle other types.
    }
}
```

این الگو برای templateهایی مفید است که رفتارشان به دسته نوع آرگومان
وابسته است. با این حال، اگر هدف صرفاً محدود کردن مجموعه نوع‌های مجاز باشد،
در ++C20 استفاده از `concepts` اغلب خواناتر است.

## کاربرد در Concepts و Constraints در ++C20

یکی از کاربردهای مهم `std::remove_cvref_t`، تعریف constraintهایی است که
باید انواع مختلف reference و cv از یک نوع را بپذیرند.

برای مثال، می‌توان concept زیر را تعریف کرد:

``` cpp
#include <concepts>
#include <type_traits>

struct Message {};

template <typename T>
concept MessageLike =
    std::same_as<std::remove_cvref_t<T>, Message>;

template <MessageLike T>
void send(T&& message) {
    // Send the message.
}
```

در این تعریف، نوع‌هایی مانند `Message`، `Message&` و `const Message&` از
نظر constraint به یک نوع نرمال‌شده تبدیل می‌شوند.

همچنین می‌توان از این ابزار در یک constraint مستقیم استفاده کرد:

``` cpp
#include <concepts>
#include <type_traits>

template <typename T>
    requires std::same_as<std::remove_cvref_t<T>, int>
void consume(T&& value) {
    // Consume an integer-like argument.
}
```

استفاده از `std::remove_cvref_t` در constraint باید آگاهانه باشد. اگر
`const` بودن یا reference بودن بخشی از قرارداد API است، حذف آن‌ها می‌تواند
constraint را بیش از حد گسترده کند و نوعی را بپذیرد که نباید پذیرفته
شود.

## کاربرد در Type Traits و انتخاب نوع

در پیاده‌سازی type traitهای سفارشی، گاهی لازم است یک شرط مستقل از cv و
reference بودن نوع باشد. برای مثال:

``` cpp
#include <type_traits>

template <typename T>
struct is_integer_like
    : std::is_integral<std::remove_cvref_t<T>> {};

static_assert(is_integer_like<int>::value);
static_assert(is_integer_like<const int&>::value);
static_assert(is_integer_like<volatile long&&>::value);
```

این trait نمونه‌ای ساده از ترکیب type traitهای استاندارد است. در پروژه
واقعی، پیش از ساختن trait سفارشی بررسی کنید که آیا trait استاندارد یا
concept مناسب‌تری وجود دارد یا خیر.

## کاربرد در Perfect Forwarding

در تابع‌های دارای perfect forwarding، پارامتر template اطلاعاتی مانند
lvalue یا rvalue بودن و cv-qualification آرگومان را حفظ می‌کند. این
اطلاعات برای forward کردن صحیح آرگومان ضروری است؛ اما برای بعضی تصمیم‌های
کامپایل‌تایم، نوع نرمال‌شده کاربردی‌تر است.

``` cpp
#include <type_traits>
#include <utility>

template <typename T>
decltype(auto) forward_to_target(T&& value) {
    using U = std::remove_cvref_t<T>;

    static_assert(std::is_class_v<U>);

    return target(std::forward<T>(value));
}
```

در این الگو، `T` برای `std::forward<T>` دست‌نخورده باقی می‌ماند؛ زیرا حذف
reference از آن می‌تواند value category آرگومان را از بین ببرد. نوع `U`
فقط برای بررسی کامپایل‌تایم استفاده می‌شود.

نکته طراحی مهم این است که `remove_cvref` را جایگزین `std::forward`
نکنید. اولی ابزار تبدیل نوع در compile time است؛ دومی ابزاری برای حفظ و
انتقال value category در یک عبارت است.

## تفاوت با `std::remove_reference` و `std::remove_cv`

این سه ابزار به هم مرتبط‌اند، اما مسئولیت یکسانی ندارند:

-   `std::remove_reference_t<T>` فقط `&` یا `&&` را حذف می‌کند.
-   `std::remove_cv_t<T>` فقط `const` و `volatile` سطح بالای نوع را حذف
    می‌کند.
-   `std::remove_cvref_t<T>` هر دو عملیات را به ترتیب مناسب انجام می‌دهد.

اگر فقط یکی از این ویژگی‌ها باید حفظ شود، استفاده از trait اختصاصی دقیق‌تر
است. برای مثال، در کدی که باید `const` بودن نوع را تشخیص دهد، نرمال‌سازی
با `remove_cvref` پیش از بررسی می‌تواند اطلاعات موردنیاز را از بین ببرد.

## پیاده‌سازی مفهومی و سازگاری با استانداردهای قدیمی‌تر

قابلیت `std::remove_cvref` بخشی از کتابخانه استاندارد ++C20 است.
بنابراین در محیطی که کتابخانه استاندارد یا حالت زبان ++C20 را پشتیبانی
نمی‌کند، نباید وجود آن را فرض کرد.

برای پروژه‌های ++C11، ++C14 یا ++C17 می‌توان معادل آن را با ترکیب traitهای
قدیمی‌تر نوشت:

``` cpp
#include <type_traits>

template <typename T>
using remove_cvref_t =
    std::remove_cv_t<std::remove_reference_t<T>>;
```

این alias از traitهای استاندارد موجود در نسخه‌های قدیمی‌تر استفاده می‌کند.
اگر این نام در یک namespace عمومی کتابخانه تعریف می‌شود، بهتر است از
نام‌گذاری‌ای استفاده شود که با نام‌های آینده یا اجزای دیگر کتابخانه تداخل
نداشته باشد.

برای تشخیص پشتیبانی کتابخانه استاندارد، می‌توان از feature-test macro
استفاده کرد:

``` cpp
#include <type_traits>

#if defined(__cpp_lib_remove_cvref) && \
    __cpp_lib_remove_cvref >= 201711L
// std::remove_cvref_t is available.
#endif
```

مقدار feature-test macro مربوط به این قابلیت `201711L` است. این macro
قابلیت کتابخانه استاندارد را گزارش می‌کند و با صرفاً انتخاب یک گزینه زبان
در compiler یکسان نیست؛ نسخه و پیکربندی standard library نیز اهمیت دارد.

## نکات مربوط به Array و Function Types

از آنجا که `remove_cvref` برخلاف `decay` تبدیل array-to-pointer و
function-to-pointer انجام نمی‌دهد، در generic codeهایی که با array یا
function type سروکار دارند باید این تفاوت را در نظر گرفت.

``` cpp
#include <type_traits>

using ArrayType = int[3];
using FunctionType = int(char);

static_assert(std::is_same_v<
    std::remove_cvref_t<ArrayType&>,
    int[3]
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<FunctionType&>,
    int(char)
>);
```

این ویژگی برای حفظ اطلاعات نوعی مفید است؛ برای مثال، زمانی که اندازه
array یا ساختار function type بخشی از تصمیم کامپایل‌تایم است. در مقابل،
اگر API به pointer نیاز دارد، باید تبدیل مناسب را به‌صورت صریح انجام دهید
یا از `std::decay_t` استفاده کنید.

## نکات مربوط به Pointerها و Member Pointerها

حذف cv در این ابزار به top-level type محدود است. بنابراین، qualifierهای
متعلق به نوعی که pointer به آن اشاره می‌کند حذف نمی‌شوند. همین اصل درباره
ساختارهای نوعی پیچیده‌تر نیز برقرار است.

``` cpp
#include <type_traits>

struct Object {
    int member;
};

using Pointer = int* const&;
using MemberPointer = int Object::* const&;

static_assert(std::is_same_v<
    std::remove_cvref_t<Pointer>,
    int*
>);

static_assert(std::is_same_v<
    std::remove_cvref_t<MemberPointer>,
    int Object::*
>);
```

در مثال دوم، `const` متعلق به خود member pointer است و پس از حذف
reference برداشته می‌شود؛ نوع عضو و کلاس مالک بدون تغییر باقی می‌مانند.

## نکات مربوط به Incomplete Types

عملیات `remove_cvref` به اندازه یا layout نوع وابسته نیست و برای انجام
تبدیل نوع، به کامل بودن class type نیاز ندارد. بنابراین معمولاً می‌توان آن
را روی reference یا cv-qualified reference به یک incomplete class نیز به
کار برد.

``` cpp
#include <type_traits>

class ForwardDeclared;

using Ref = const ForwardDeclared&;
using Result = std::remove_cvref_t<Ref>;

static_assert(std::is_same_v<Result, ForwardDeclared>);
```

این موضوع به معنای مجاز بودن هر عملیات دیگری روی incomplete type نیست.
برای مثال، traitهایی که به ویژگی‌های کامل نوع وابسته‌اند ممکن است
محدودیت‌های جداگانه‌ای داشته باشند.

## خطاهای رایج و سوءبرداشت‌ها

### تصور حذف همه cv-qualifierها

این ابزار qualifierهای سطح بالای نوع را حذف می‌کند، نه تمام `const` و
`volatile`های موجود در ساختار نوع را. برای مثال، `const int*` پس از
نرمال‌سازی همچنان `const int*` است.

### استفاده به جای `std::decay_t`

اگر کد به pointer شدن array یا function نیاز دارد، `remove_cvref_t` چنین
تبدیلی انجام نمی‌دهد. انتخاب اشتباه این دو trait می‌تواند باعث ناسازگاری
نوع در template یا overload resolution شود.

### استفاده برای تغییر شیء در زمان اجرا

`std::remove_cvref_t` یک type transformation در compile time است. این
ابزار شیء را تغییر نمی‌دهد، `const` را از یک object واقعی حذف نمی‌کند و
مجوز نوشتن در حافظه‌ای که از طریق یک `const` object قابل دسترسی است ایجاد
نمی‌کند.

### استفاده پیش از Perfect Forwarding

اگر نوع deduced شده را برای `std::forward` نرمال‌سازی کنید، ممکن است
اطلاعات لازم برای حفظ lvalue/rvalue بودن آرگومان را از دست بدهید.
پارامتر اصلی `T` را برای forwarding نگه دارید و از نوع نرمال‌شده فقط برای
بررسی یا انتخاب رفتار استفاده کنید.

### حذف اطلاعاتی که بخشی از قرارداد API است

گاهی تفاوت `const T&` با `T&` یا `T&&` برای semantics تابع مهم است.
نرمال‌سازی زودهنگام می‌تواند این تفاوت را پنهان کند. پیش از استفاده، مشخص
کنید تصمیم شما باید بر اساس نوع دقیق آرگومان باشد یا نوع پایه آن.

## ملاحظات طراحی و کارایی

`std::remove_cvref` در compile time عمل می‌کند و object یا allocation
زمان اجرا تولید نمی‌کند. استفاده از آن به‌خودی‌خود هزینه runtime ندارد؛ با
این حال، مانند هر template metaprogramming دیگری، پیچیدگی بیش از حد در
زنجیره traitها می‌تواند خوانایی و زمان کامپایل را تحت تأثیر قرار دهد.

برای طراحی APIهای عمومی، بهتر است مرز میان دو مفهوم روشن باشد:

-   نوع دقیق deduced شده که ممکن است اطلاعات cv و reference را حمل کند.
-   نوع نرمال‌شده‌ای که برای مقایسه، dispatch یا constraint استفاده می‌شود.

همچنین بهتر است نرمال‌سازی را در نزدیک‌ترین نقطه به تصمیمی انجام دهید که
به آن نیاز دارد، نه اینکه نوع اصلی را در تمام مسیرهای پیاده‌سازی با نسخه
نرمال‌شده جایگزین کنید.

## جمع‌بندی

`std::remove_cvref` یک type trait استاندارد در کتابخانه ++C20 است که
ابتدا reference و سپس cv-qualifierهای سطح بالای نوع را حذف می‌کند. شکل
alias آن، یعنی `std::remove_cvref_t`، در کدهای generic و template معمولاً
خواناتر است.

مهم‌ترین تفاوت آن با `std::decay_t` این است که `remove_cvref` ساختار
array و function type را حفظ می‌کند، در حالی که `decay` آن‌ها را به
pointer تبدیل می‌کند. این ابزار برای type comparison، نوشتن `concepts`،
ساخت traitهای سفارشی و تصمیم‌گیری کامپایل‌تایم بسیار مفید است؛ اما نباید
برای تغییر object در زمان اجرا یا جایگزینی `std::forward` به کار رود.

قاعده عملی این است: هرگاه هدف، بررسی نوع پایه بدون cv و reference باشد و
حفظ ساختار نوع اهمیت داشته باشد، `std::remove_cvref_t` انتخاب مناسبی
است. هرگاه به semantics مربوط به decay نیاز دارید، `std::decay_t` را
انتخاب کنید و هرگاه cv یا reference بخشی از قرارداد تابع است، نوع دقیق
را حفظ کنید.


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>