<div align="center">

[🇺🇸 English](../../en/cpp14/std_tuple_element_t.md) | [🇮🇷 فارسی](./std_tuple_element_t.md)

</div>

---

# آموزش جامع `std::tuple_element_t` در ++C14

## معرفی قابلیت و جایگاه آن در کتابخانه استاندارد

قابلیت `std::tuple_element_t` در استاندارد ++C14 معرفی شده است و امکان دسترسی به نوع دادهٔ عنصر مشخصی از یک ساختار چندعضوی را در زمان کامپایل فراهم می‌کند. این ابزار بخشی از کتابخانهٔ استاندارد و زیرمجموعهٔ قابلیت‌های مرتبط با `std::tuple` است و برای کار با `std::tuple`، `std::pair` و `std::array` کاربرد دارد.

در برنامه‌نویسی Generic و طراحی Templateها، گاهی لازم است نوع یک عضو را بدون ساختن شیء، بدون دسترسی به مقدار آن و صرفاً با استفاده از اطلاعات نوعی استخراج کنیم. برای مثال، ممکن است بخواهیم نوع عضو دوم یک `std::tuple` را برای تعریف یک متغیر، تعیین نوع خروجی یک تابع یا ایجاد یک Template دیگر به کار ببریم.

ویژگی مهم این ابزار آن است که نتیجهٔ آن یک نوع داده در زمان کامپایل است، نه مقداری که در زمان اجرا محاسبه شود.

نمونهٔ سادهٔ زیر نحوهٔ استفاده از این قابلیت را نشان می‌دهد:

```
#include <iostream>
#include <string>
#include <tuple>
#include <type_traits>

int main() {
    using Data = std::tuple<int, double, std::string>;

    using FirstType = std::tuple_element_t<0, Data>;
    using SecondType = std::tuple_element_t<1, Data>;
    using ThirdType = std::tuple_element_t<2, Data>;

    static_assert(std::is_same_v<FirstType, int>);
    static_assert(std::is_same_v<SecondType, double>);
    static_assert(std::is_same_v<ThirdType, std::string>);

    FirstType number = 42;
    SecondType value = 3.14;
    ThirdType text = "Hello";

    std::cout << number << '\n';
    std::cout << value << '\n';
    std::cout << text << '\n';
}
```

در این مثال، سه نوع متفاوت از یک `std::tuple` استخراج شده‌اند. هیچ شیء `tuple` برای انجام این استخراج لازم نیست و تمام بررسی‌های مربوط به نوع‌ها در زمان کامپایل انجام می‌شوند.

## مفهوم اصلی و سازوکار استخراج نوع

### آشنایی با `std::tuple_element`

نوع `std::tuple_element` یک `class template` در فضای نام `std` است که نوع عنصر مشخص‌شده با یک اندیس را از یک نوع پشتیبانی‌شده استخراج می‌کند.

شکل کلی آن به صورت زیر است:

```
template<std::size_t I, class T>
struct tuple_element;
```

در این تعریف:

- پارامتر `I` اندیس عنصر موردنظر است و از نوع `std::size_t` استفاده می‌کند.
- پارامتر `T` نوع ساختاری است که عنصرهای آن بررسی می‌شوند.
- عضو `type` در ساختار `std::tuple_element<I, T>` نوع عنصر انتخاب‌شده را مشخص می‌کند.

برای نمونه، عبارت زیر نوع عضو سوم یک `tuple` را استخراج می‌کند:

```
using TupleType = std::tuple<int, float, long>;
using ElementType = std::tuple_element<2, TupleType>::type;
```

در استاندارد ++C14، می‌توان همین عبارت را به شکل کوتاه‌تر زیر نوشت:

```
using ElementType = std::tuple_element_t<2, TupleType>;
```

### تفاوت `std::tuple_element` و `std::tuple_element_t`

نوع `std::tuple_element` یک `class template` است، درحالی‌که `std::tuple_element_t` یک alias template محسوب می‌شود.

تعریف مفهومی alias در استاندارد ++C14 به این شکل است:

```
template<std::size_t I, class T>
using tuple_element_t = typename tuple_element<I, T>::type;
```

این تعریف نشان می‌دهد که `std::tuple_element_t` قابلیت جدیدی برای استخراج نوع ایجاد نمی‌کند؛ بلکه دسترسی به عضو `type` را ساده‌تر می‌سازد.

| ویژگی           | `std::tuple_element`   | `std::tuple_element_t`                       |
| --------------- | ---------------------- | -------------------------------------------- |
| نوع تعریف       | `class template`       | `alias template`                             |
| دسترسی به نوع   | از طریق `::type`       | مستقیم                                       |
| نسخهٔ استاندارد | ++C11                  | ++C14                                        |
| کاربرد رایج     | دسترسی به ساختار Trait | تعریف نوع با `using` و استفاده در Templateها |

در کدهای جدید، استفاده از `std::tuple_element_t` معمولاً خوانایی بیشتری دارد. بااین‌حال، `std::tuple_element` همچنان برای درک ساختار Type Traitها و کار با سازوکارهای قدیمی‌تر مفید است.

### تفاوت نوع عنصر و مقدار عنصر

نکتهٔ اساسی این است که `std::tuple_element_t` نوع عنصر را استخراج می‌کند، نه مقدار آن را.

برای مثال، در کد زیر مقدار عضو اول برابر با `42` است، اما خروجی `std::tuple_element_t` نوع `int` خواهد بود:

```
#include <tuple>
#include <type_traits>

int main() {
    using Data = std::tuple<int, double>;

    Data values{42, 3.14};

    using FirstType = std::tuple_element_t<0, Data>;

    static_assert(std::is_same_v<FirstType, int>);

    (void)values;
}
```

در این مثال، شیء `values` برای استخراج نوع ضروری نیست. حتی اگر این شیء اصلاً وجود نداشته باشد، نوع عضو اول همچنان از تعریف `Data` قابل استخراج است.

## کاربردهای استاندارد و نوع‌های پشتیبانی‌شده

### استفاده با `std::tuple`

ساختار `std::tuple` می‌تواند چند مقدار با نوع‌های متفاوت را در یک نوع مرکب نگهداری کند. این ویژگی آن را برای بازگرداندن چند مقدار از یک تابع، ساختارهای داده‌ای عمومی و انتقال مجموعه‌ای از مقادیر ناهمگون مناسب می‌کند.

برای مثال:

```
#include <tuple>
#include <type_traits>

using Record = std::tuple<int, double, const char*>;

using IdType = std::tuple_element_t<0, Record>;
using ScoreType = std::tuple_element_t<1, Record>;
using NameType = std::tuple_element_t<2, Record>;

static_assert(std::is_same_v<IdType, int>);
static_assert(std::is_same_v<ScoreType, double>);
static_assert(std::is_same_v<NameType, const char*>);
```

در این نمونه، هر نوع با استفاده از اندیس متناظر با جایگاه آن در `tuple` استخراج شده است. ترتیب نوع‌ها اهمیت دارد؛ تغییر ترتیب پارامترهای `std::tuple` می‌تواند نتیجهٔ استخراج را نیز تغییر دهد.

### استفاده با `std::pair`

ساختار `std::pair` دو عضو با نوع‌های مستقل دارد و از `std::tuple_element` نیز پشتیبانی می‌کند.

```
#include <string>
#include <tuple>
#include <type_traits>
#include <utility>

using Entry = std::pair<int, std::string>;

using KeyType = std::tuple_element_t<0, Entry>;
using ValueType = std::tuple_element_t<1, Entry>;

static_assert(std::is_same_v<KeyType, int>);
static_assert(std::is_same_v<ValueType, std::string>);
```

این کاربرد به‌خصوص هنگام طراحی الگوریتم‌های عمومی مفید است؛ الگوریتمی که با یک جفت کلید و مقدار کار می‌کند، می‌تواند نوع هر عضو را بدون وابستگی به مقادیر واقعی استخراج کند.

### استفاده با `std::array`

ساختار `std::array` مجموعه‌ای از عنصرها با یک نوع مشترک است. بااین‌حال، نوع عنصر آن نیز از طریق `std::tuple_element` قابل استخراج است.

```
#include <array>
#include <tuple>
#include <type_traits>

using Numbers = std::array<int, 5>;

using ElementType = std::tuple_element_t<0, Numbers>;

static_assert(std::is_same_v<ElementType, int>);
```

تفاوت مهم این است که همهٔ عنصرهای `std::array` یک نوع دارند؛ بنابراین، برای یک آرایهٔ معتبر با تعداد عنصر مثبت، استخراج نوع از هر اندیس معتبر همان نوع عنصر را نتیجه می‌دهد.

### پشتیبانی از `const` و `volatile`

در استاندارد ++C، نسخه‌های تخصصی `std::tuple_element` برای نوع‌های `const`، `volatile` و ترکیب آن‌ها با `tuple`، `pair` و `array` تعریف شده‌اند. در نتیجه، qualifierهای نوع اصلی در نوع استخراج‌شده حفظ می‌شوند.

```
#include <tuple>
#include <type_traits>

using Data = std::tuple<int, double>;

using MutableType = std::tuple_element_t<0, Data>;
using ConstType = std::tuple_element_t<0, const Data>;
using VolatileType = std::tuple_element_t<0, volatile Data>;
using ConstVolatileType = std::tuple_element_t<0, const volatile Data>;

static_assert(std::is_same_v<MutableType, int>);
static_assert(std::is_same_v<ConstType, const int>);
static_assert(std::is_same_v<VolatileType, volatile int>);
static_assert(std::is_same_v<ConstVolatileType, const volatile int>);
```

این رفتار برای Generic Code اهمیت زیادی دارد؛ زیرا نوع استخراج‌شده لزوماً همان نوع پایهٔ ذخیره‌شده در ساختار نیست و ممکن است qualifierهای سطح بالای نوع اصلی را نیز داشته باشد.

## قواعد اندیس‌گذاری و محدودیت‌های دسترسی

### اندیس‌گذاری از صفر

اندیس عنصرها در `std::tuple_element_t` از صفر آغاز می‌شود. بنابراین، برای یک `tuple` با \\(N\\) عنصر، اندیس معتبر باید در محدودهٔ زیر قرار داشته باشد:

\\[ 0 \le I < N \\]

برای نمونه، اگر یک `tuple` سه عنصر داشته باشد، اندیس‌های معتبر آن `0`، `1` و `2` هستند.

```
#include <tuple>
#include <type_traits>

using Data = std::tuple<int, double, char>;

static_assert(std::is_same_v<std::tuple_element_t<0, Data>, int>);
static_assert(std::is_same_v<std::tuple_element_t<1, Data>, double>);
static_assert(std::is_same_v<std::tuple_element_t<2, Data>, char>);
```

### رفتار در صورت استفاده از اندیس نامعتبر

استفاده از اندیسی که خارج از محدودهٔ عنصرهای `tuple`، `pair` یا `array` باشد، استفادهٔ معتبری از این Trait نیست و در استاندارد برای آن نتیجهٔ نوعی قابل اتکایی تعریف نشده است. در عمل، معمولاً کامپایلر خطا می‌دهد.

```
#include <tuple>

using Data = std::tuple<int, double>;

// Invalid: index 2 is outside the valid range [0, 2).
// using InvalidType = std::tuple_element_t<2, Data>;
```

در این نمونه، خطا در زمان کامپایل رخ می‌دهد، نه در زمان اجرا. برای جلوگیری از چنین خطاهایی، اگر اندیس از یک پارامتر Template به دست می‌آید، بهتر است اعتبار آن را با قیود Template یا `static_assert` بررسی کنید.

### رفتار در برابر `std::tuple<>`

یک `tuple` خالی هیچ عضوی ندارد؛ بنابراین هیچ اندیسی برای استخراج نوع عنصر آن معتبر نیست.

```
#include <tuple>

using Empty = std::tuple<>;

// Invalid: Empty has no elements.
// using ElementType = std::tuple_element_t<0, Empty>;
```

این محدودیت در طراحی Templateهای عمومی اهمیت دارد. اگر الگوریتم شما باید `tuple` خالی را نیز بپذیرد، نباید بدون بررسی تعداد عنصرها، `std::tuple_element_t<0, T>` را مستقیماً به کار ببرید.

## کاربرد در طراحی Templateهای عمومی

### استخراج نوع برای تعریف متغیر

یکی از کاربردهای عملی این ابزار، تعریف متغیری با نوعی است که از یک ساختار دیگر استخراج شده است. این روش از تکرار نام نوع‌ها جلوگیری می‌کند و وابستگی کد به جزئیات پیاده‌سازی را کاهش می‌دهد.

```
#include <string>
#include <tuple>

using UserRecord = std::tuple<int, std::string, double>;

using NameType = std::tuple_element_t<1, UserRecord>;

NameType userName = "Ali";
```

در این مثال، اگر نوع نام در تعریف `UserRecord` تغییر کند، نوع `NameType` نیز به‌صورت خودکار با تعریف جدید هماهنگ می‌شود.

این رویکرد در کتابخانه‌هایی که نوع‌های مرکب را به‌عنوان پارامتر دریافت می‌کنند، می‌تواند از تکرار و ناسازگاری نوع‌ها جلوگیری کند.

### استفاده در تعریف نوع خروجی Template

در برخی طراحی‌ها، نوع خروجی یک کلاس یا تابع عمومی به نوع یکی از اعضای ورودی وابسته است.

```
#include <tuple>
#include <type_traits>

template<class Tuple>
struct FirstElement {
    using type = std::tuple_element_t<0, Tuple>;
};

using Data = std::tuple<long, double, char>;
using Result = FirstElement<Data>::type;

static_assert(std::is_same_v<Result, long>);
```

در این مثال، `FirstElement` نوع عضو اول را استخراج و به‌عنوان `type` منتشر می‌کند. چنین الگویی در Type Traitهای سفارشی و طراحی زیرساخت‌های Generic کاربرد دارد.

البته اگر هدف فقط تعریف یک نام مستعار باشد، می‌توان از `std::tuple_element_t` مستقیماً استفاده کرد و نیازی به تعریف یک کلاس واسط نیست.

### استفاده در ترکیب با `std::conditional_t`

نوع استخراج‌شده را می‌توان با سایر ابزارهای Metaprogramming ترکیب کرد تا نوع نهایی بر اساس یک شرط زمان کامپایل انتخاب شود.

```
#include <tuple>
#include <type_traits>

using Data = std::tuple<int, double, char>;

using SecondType = std::tuple_element_t<1, Data>;

using SelectedType = std::conditional_t<
    std::is_floating_point_v<SecondType>,
    long,
    int
>;

static_assert(std::is_same_v<SelectedType, long>);
```

در این نمونه، ابتدا نوع عضو دوم استخراج می‌شود و سپس `std::conditional_t` بر اساس ویژگی آن، نوع `long` یا `int` را انتخاب می‌کند.

توجه داشته باشید که `std::conditional_t` به‌خودی‌خود تضمین نمی‌کند که تنها یکی از دو آرگومان نوعی آن قابل تشکیل باشد. اگر یکی از نوع‌ها نامعتبر باشد، ممکن است حتی در شاخه‌ای که انتخاب نمی‌شود نیز خطای کامپایل ایجاد شود.

### استفاده در `decltype` و استخراج نوع عبارت

ابزار `std::tuple_element_t` و عبارت `decltype` هر دو برای استخراج اطلاعات نوعی مفید هستند، اما هدف یکسانی ندارند.

- ابزار `std::tuple_element_t` نوع عنصر را از روی نوع ساختار و اندیس آن استخراج می‌کند.
- عبارت `decltype` نوع یک عبارت مشخص را استخراج می‌کند و نتیجهٔ آن می‌تواند به قواعد مقدارگذاری عبارت وابسته باشد.

برای نمونه:

```
#include <tuple>
#include <type_traits>
#include <utility>

using Data = std::tuple<int, double>;

using ElementType = std::tuple_element_t<0, Data>;

using LvalueReferenceType =
    decltype(std::get<0>(std::declval<Data&>()));

using ConstLvalueReferenceType =
    decltype(std::get<0>(std::declval<const Data&>()));

static_assert(std::is_same_v<ElementType, int>);
static_assert(std::is_same_v<LvalueReferenceType, int&>);
static_assert(std::is_same_v<ConstLvalueReferenceType, const int&>);
```

در این مثال، نوع عضو مستقل از عبارت دسترسی، `int` است؛ اما نوع عبارت دسترسی به عضو می‌تواند `int&` یا `const int&` باشد.

این تفاوت در طراحی APIهایی که باید رفتار Referenceها را حفظ کنند، بسیار مهم است.

## ارتباط با `std::get`

### تفاوت استخراج نوع و دسترسی به مقدار

ابزار `std::tuple_element_t` با تابع `std::get` ارتباط نزدیکی دارد، اما این دو جایگزین یکدیگر نیستند.

| ویژگی            | `std::tuple_element_t` | `std::get`                            |
| ---------------- | ---------------------- | ------------------------------------- |
| هدف              | استخراج نوع عنصر       | دسترسی به مقدار عنصر                  |
| زمان اصلی عملیات | زمان کامپایل           | دسترسی به شیء در زمان اجرا            |
| ورودی اصلی       | نوع ساختار و اندیس     | شیء ساختار و اندیس                    |
| نتیجه            | یک نوع داده            | یک عبارت با نوع و دستهٔ مقداری مشخص   |
| نیاز به شیء      | ندارد                  | برای دسترسی به مقدار به شیء نیاز دارد |

برای نمونه:

```
#include <tuple>
#include <type_traits>

int main() {
    std::tuple<int, double> data{10, 2.5};

    using FirstType = std::tuple_element_t<0, decltype(data)>;

    static_assert(std::is_same_v<FirstType, int>);

    int& first = std::get<0>(data);
    first = 20;
}
```

در این مثال، `std::tuple_element_t` نوع عضو را استخراج می‌کند؛ درحالی‌که `std::get` امکان خواندن یا تغییر مقدار عضو را فراهم می‌سازد.

### حفظ Referenceها هنگام دسترسی به عنصرها

در زمان طراحی Templateها باید میان نوع ذخیره‌شده در `tuple` و نوع عبارت حاصل از `std::get` تفاوت قائل شد.

تابع `std::get` برای نسخه‌های مختلف `tuple`، از جمله شیء قابل تغییر، شیء `const`، مقدار `rvalue` و مقدار `const rvalue`، overloadهای متفاوتی دارد. به همین دلیل، نوع عبارت حاصل از فراخوانی آن می‌تواند متفاوت باشد.

```
#include <tuple>
#include <type_traits>
#include <utility>

using Data = std::tuple<int, double>;

static_assert(
    std::is_same_v<
        std::tuple_element_t<0, Data>,
        int
    >
);

static_assert(
    std::is_same_v<
        decltype(std::get<0>(std::declval<Data&>())),
        int&
    >
);

static_assert(
    std::is_same_v<
        decltype(std::get<0>(std::declval<Data&&>())),
        int&&
    >
);
```

نکتهٔ طراحی این است که برای بررسی نوع ذخیره‌شدهٔ یک عضو، `std::tuple_element_t` انتخاب مناسبی است؛ اما اگر نوع Reference حاصل از دسترسی واقعی به آن عضو اهمیت دارد، باید از `decltype` همراه با عبارت دسترسی مناسب استفاده کرد.

## استفاده در Structured Bindings

### ارتباط با قابلیت Structured Bindings در ++C17

قابلیت Structured Bindings در استاندارد ++C17 معرفی شد. این قابلیت امکان تجزیهٔ یک شیء مرکب و نام‌گذاری مستقیم اعضای آن را فراهم می‌کند.

```
#include <string>
#include <tuple>

int main() {
    std::tuple<int, std::string, double> data{
        42,
        "Alice",
        98.5
    };

    auto [id, name, score] = data;

    (void)id;
    (void)name;
    (void)score;
}
```

در این مثال، کامپایلر از قواعد مربوط به Structured Bindings استفاده می‌کند تا اعضای `data` را در اختیار نام‌های `id`، `name` و `score` قرار دهد.

برای نوع‌های Tuple-like، سازوکار Structured Bindings به اطلاعاتی مانند `std::tuple_size`، `std::tuple_element` و تابع‌های `get` وابسته است. بنابراین، `std::tuple_element` یکی از اجزای مهم زیرساختی است که این قابلیت به آن متکی است.

### تفاوت استخراج نوع با تجزیهٔ ساختار

اگر تنها به نوع یکی از اعضا نیاز داشته باشید، استفاده از `std::tuple_element_t` ساده‌تر و مستقیم‌تر است. در مقابل، Structured Bindings زمانی مناسب‌اند که بخواهید به چند عضو یک شیء دسترسی داشته باشید.

برای مثال، اگر هدف فقط تعریف یک نام مستعار برای نوع عضو دوم باشد، نوشتن کدی مانند زیر کافی است:

```
#include <tuple>

using Data = std::tuple<int, double, char>;
using SecondType = std::tuple_element_t<1, Data>;
```

در این حالت، نیازی به ساختن یک شیء یا تعریف Structured Binding وجود ندارد.

## پیاده‌سازی سازوکار Tuple-like سفارشی

### نقش `std::tuple_element` در نوع‌های سفارشی

کتابخانهٔ استاندارد امکان تعریف برخی اجزای پروتکل Tuple-like را برای نوع‌های سفارشی فراهم می‌کند. در طراحی چنین نوع‌هایی، برنامه‌نویس می‌تواند برای نوع متعلق به برنامهٔ خود، تخصصی‌سازی‌های مناسب `std::tuple_size` و `std::tuple_element` را ارائه کند و تابع‌های `get` متناظر را تعریف کند.

این سازوکار به کدهای عمومی اجازه می‌دهد اطلاعات نوعی اعضای یک ساختار سفارشی را با روشی مشابه `std::tuple` استخراج کنند.

بااین‌حال، نباید فرض کرد که تخصصی‌سازی `std::tuple_element` به‌تنهایی یک نوع را به‌طور کامل Tuple-like می‌کند. برای پشتیبانی از Structured Bindings و الگوریتم‌های مبتنی بر پروتکل Tuple-like، اجزای لازم باید با یکدیگر سازگار باشند.

### نمونهٔ نوع سفارشی با دو عضو

نمونهٔ زیر یک ساختار ساده را تعریف می‌کند که می‌تواند در الگوی Tuple-like مورد استفاده قرار گیرد.

```
#include <cstddef>
#include <tuple>
#include <type_traits>
#include <utility>

struct Point {
    int x;
    double y;
};

namespace std {

template<>
struct tuple_size<Point>
    : integral_constant<size_t, 2> {};

template<>
struct tuple_element<0, Point> {
    using type = int;
};

template<>
struct tuple_element<1, Point> {
    using type = double;
};

}

template<std::size_t I>
decltype(auto) get(Point& point) {
    static_assert(I < 2);

    if constexpr (I == 0) {
        return (point.x);
    } else {
        return (point.y);
    }
}

template<std::size_t I>
decltype(auto) get(const Point& point) {
    static_assert(I < 2);

    if constexpr (I == 0) {
        return (point.x);
    } else {
        return (point.y);
    }
}

int main() {
    using XType = std::tuple_element_t<0, Point>;
    using YType = std::tuple_element_t<1, Point>;

    static_assert(std::is_same_v<XType, int>);
    static_assert(std::is_same_v<YType, double>);

    Point point{10, 20.5};

    auto [x, y] = point;

    (void)x;
    (void)y;
}
```

در این مثال، تخصصی‌سازی‌های `std::tuple_size` و `std::tuple_element` اطلاعات تعداد و نوع اعضا را فراهم می‌کنند. تابع‌های `get` نیز امکان دسترسی به اعضا را بر اساس اندیس فراهم می‌سازند.

استفاده از `if constexpr` در این کد به استاندارد ++C17 نیاز دارد. بنابراین، این نمونه برای نشان دادن ارتباط `std::tuple_element_t` با Tuple-likeها مناسب است، اما کل کد را نمی‌توان به‌عنوان نمونه‌ای سازگار با ++C14 کامپایل کرد.

همچنین، این پیاده‌سازی صرفاً برای نشان دادن ساختار کلی پروتکل است. برای یک نوع عمومی و حرفه‌ای، باید overloadهای مناسب برای Referenceهای مختلف، رفتار `const`، دسترسی به اعضای `rvalue` و جزئیات موردنیاز استاندارد پیاده‌سازی شوند. تخصصی‌سازی اعضای قالبی در فضای نام `std` نیز باید به مواردی محدود شود که استاندارد صراحتاً اجازه می‌دهد؛ در این مثال، نوع سفارشی `Point` متعلق به برنامه است.

## محدودیت‌ها و نکات طراحی

### محدودیت در استخراج نوع از دادهٔ زمان اجرا

ابزار `std::tuple_element_t` نمی‌تواند بر اساس یک اندیس متغیر در زمان اجرا، نوع متفاوتی را انتخاب کند.

برای مثال، اگر یک `tuple` دارای اعضایی با نوع‌های متفاوت باشد، نمی‌توان یک متغیر عددی زمان اجرا را مستقیماً به‌عنوان پارامتر نوعی `std::tuple_element_t` استفاده کرد؛ زیرا پارامتر اندیس باید یک مقدار ثابت قابل استفاده در زمان کامپایل باشد.

```
#include <cstddef>
#include <tuple>

using Data = std::tuple<int, double, char>;

template<std::size_t I>
using ElementType = std::tuple_element_t<I, Data>;

using FirstType = ElementType<0>;
using ThirdType = ElementType<2>;
```

در این نمونه، اندیس در هر بار استفاده مشخص است و کامپایلر می‌تواند نوع متناظر را تعیین کند.

اگر انتخاب عضو باید بر اساس ورودی زمان اجرا انجام شود، معمولاً باید از یک طراحی متفاوت مانند `std::variant`، توابع جداگانه یا یک لایهٔ Dispatch استفاده کرد؛ انتخاب روش مناسب به نوع مسئله بستگی دارد.

### تفاوت با `std::remove_cvref_t`

قالب `std::tuple_element_t` به‌طور خودکار تمام qualifierهای نوع ورودی را حذف نمی‌کند. اگر هدف استخراج نوع عضو بدون `const`، `volatile` و Reference باشد، باید از ابزار مناسب استفاده کرد.

نوع `std::remove_cvref_t` در ++C20 معرفی شده است و در ++C14 در دسترس نیست. برای سازگاری با ++C14 می‌توان از ترکیب `std::remove_reference_t` و `std::remove_cv_t` استفاده کرد.

```
#include <tuple>
#include <type_traits>

using Data = std::tuple<int, double>;

using ElementType = std::tuple_element_t<0, const Data>;

using NormalizedType = std::remove_cv_t<
    std::remove_reference_t<ElementType>
>;

static_assert(std::is_same_v<ElementType, const int>);
static_assert(std::is_same_v<NormalizedType, int>);
```

در این مثال، `ElementType` به دلیل `const` بودن نوع `Data`، نوع `const int` است؛ اما `NormalizedType` پس از حذف qualifierها، `int` خواهد بود.

البته اگر Reference یا qualifier بخشی از قرارداد نوعی API شما باشد، حذف آن‌ها ممکن است اطلاعات مهمی را از بین ببرد. بنابراین، نرمال‌سازی نوع باید هدفمند انجام شود.

### تفاوت با `std::decay_t`

نوع `std::decay_t` نیز برای نرمال‌سازی نوع‌ها استفاده می‌شود، اما دامنهٔ تبدیل‌های آن با `std::remove_cv_t` و `std::remove_reference_t` یکسان نیست.

قالب `std::decay_t` علاوه بر حذف Reference و qualifierهای سطح بالا، تبدیل‌هایی مشابه تبدیل آرایه به Pointer و تبدیل نوع تابع به Pointer به تابع را نیز اعمال می‌کند.

در نتیجه، اگر تنها هدف حذف `const` و `volatile` یا Reference باشد، انتخاب `std::decay_t` ممکن است بیش از نیاز شما نوع را تغییر دهد.

## اشتباهات رایج و روش جلوگیری از آن‌ها

### اشتباه در انتخاب اندیس

یکی از رایج‌ترین خطاها، فرض کردن اندیس‌گذاری از یک به‌جای صفر است. در `std::tuple_element_t`، اندیس `0` همیشه به اولین عضو اشاره می‌کند.

برای جلوگیری از خطا، در کدهای عمومی بهتر است ترتیب اعضا مستند باشد یا به‌جای اندیس‌های پراکنده، نام‌های مستعار معنادار تعریف شوند.

### اشتباه در فرض یکسان بودن نوع عضو و نوع دسترسی

نوع استخراج‌شده از `std::tuple_element_t` لزوماً با نوع عبارت حاصل از `std::get` یکسان نیست. اگر تابع شما باید Reference برگرداند، استفاده از نوع عنصر به‌تنهایی برای تعیین نوع خروجی کافی نیست.

در چنین شرایطی، باید نوع عبارت دسترسی را با `decltype` بررسی کنید و در صورت نیاز، دستهٔ مقدار و qualifierهای آن را حفظ کنید.

### اشتباه در استفاده از ویژگی‌های استانداردهای جدیدتر

قالب `std::tuple_element_t` متعلق به ++C14 است؛ اما ابزارهایی مانند `std::is_same_v` در ++C17 معرفی شده‌اند. همچنین، `std::remove_cvref_t` به ++C20 تعلق دارد.

بنابراین، استفاده از این ابزارها در یک پروژه با هدف کامپایل ++C14، بدون جایگزین مناسب، باعث خطای کامپایل خواهد شد.

برای پروژه‌های ++C14 می‌توان از `std::is_same` به همراه `::value` و از ترکیب `std::remove_cv` و `std::remove_reference` استفاده کرد.

### اشتباه در تخصصی‌سازی فضای نام `std`

تخصصی‌سازی قالب‌های استاندارد برای نوع‌های متعلق به برنامه در موارد مشخص مجاز است، اما افزودن اعضا یا overloadهای دلخواه به فضای نام `std` مجاز نیست، مگر آنکه استاندارد صراحتاً چنین کاری را اجازه داده باشد.

هنگام توسعهٔ یک نوع Tuple-like سفارشی، باید تنها تخصصی‌سازی‌های مجاز را اضافه کرد و از تعریف نام‌های جدید و دلخواه در فضای نام `std` خودداری کرد.

## سازگاری با استانداردهای مختلف ++C

قالب `std::tuple_element_t` در ++C14 به کتابخانهٔ استاندارد اضافه شد و از آن زمان همچنان بخشی از کتابخانهٔ استاندارد است.

| نسخهٔ استاندارد       | وضعیت و نکتهٔ مرتبط                                                                                                                        |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| ++C11                 | `std::tuple_element` در دسترس است؛ برای دسترسی به نوع باید از `::type` استفاده کرد.                                                        |
| ++C14                 | `std::tuple_element_t` معرفی شد و نوشتن کدهای نوع‌محور ساده‌تر شد.                                                                         |
| ++C17                 | قابلیت Structured Bindings معرفی شد و استفاده از اطلاعات Tuple-like برای تجزیهٔ ساختارها گسترش یافت.                                       |
| ++C20                 | ابزارهایی مانند `std::remove_cvref_t` برای نرمال‌سازی نوع در دسترس قرار گرفتند.                                                            |
| ++C23 و نسخه‌های بعدی | `std::tuple_element_t` همچنان برای استخراج نوع عنصر کاربرد دارد؛ جزئیات پروتکل‌ها و قابلیت‌های جدید باید بر اساس استاندارد هدف بررسی شوند. |

برای استفاده از این قابلیت در ++C14، معمولاً کافی است هدر `<tuple>` را وارد کنید. برای کار با `std::pair` نیز هدر `<utility>` مرتبط است، هرچند `<tuple>` برای استفاده از خود `std::tuple_element_t` لازم خواهد بود.

## جمع‌بندی نهایی

قالب `std::tuple_element_t` ابزاری کوچک اما مهم در کتابخانهٔ استاندارد ++C14 است که استخراج نوع عنصر از ساختارهای چندعضوی را در زمان کامپایل امکان‌پذیر می‌کند.

مهم‌ترین نکات کاربردی آن عبارت‌اند از:

- این قالب یک نوع داده را استخراج می‌کند و مقدار عنصر را در اختیار قرار نمی‌دهد.
- استفاده از آن به شیء موجود در زمان اجرا نیاز ندارد.
- اندیس‌گذاری از صفر آغاز می‌شود و اندیس باید در محدودهٔ معتبر قرار داشته باشد.
- این ابزار با ساختارهایی مانند `std::tuple`، `std::pair` و `std::array` قابل استفاده است.
- qualifierهای سطح بالای نوع ساختار، مانند `const` و `volatile`، می‌توانند بر نوع استخراج‌شده تأثیر بگذارند.
- برای تشخیص نوع Reference حاصل از دسترسی به عضو، معمولاً باید از `decltype` همراه با `std::get` استفاده کرد.
- این قالب یکی از اجزای مهم سازوکارهای Generic Programming، Type Traitها و پروتکل Tuple-like است.

در طراحی کدهای حرفه‌ای، بهترین زمان برای استفاده از `std::tuple_element_t` هنگامی است که نوع یک عضو از قبل مشخص است و به استخراج آن در زمان کامپایل نیاز دارید. در مقابل، اگر هدف دسترسی به مقدار، مدیریت Referenceها یا انتخاب عضو بر اساس ورودی زمان اجرا باشد، باید از ابزارهای مناسب همان مسئله استفاده کنید.

---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>