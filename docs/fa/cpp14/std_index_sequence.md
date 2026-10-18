<div align="center">

[🇺🇸 English](../../en/cpp14/std_index_sequence.md) | [🇮🇷 فارسی](./std_index_sequence.md)

</div>

---

# آموزش جامع `std::index_sequence` در ++C14

## مقدمه

یکی از قابلیت‌های مهم Template Metaprogramming در کتابخانه استاندارد ++C14، خانوادهٔ `std::integer_sequence` و به‌ویژه نوع `std::index_sequence` است. این ابزارها امکان نمایش مجموعه‌ای از مقادیر صحیح به‌صورت یک `type` در زمان کامپایل را فراهم می‌کنند و به توسعه‌دهنده اجازه می‌دهند عملیاتی را که به اندیس‌های متوالی نیاز دارند، بدون تولید دستی کد برای هر اندیس پیاده‌سازی کند.

برای مثال، فرض کنید یک `std::tuple` شامل چند مقدار با انواع متفاوت دارید و می‌خواهید همهٔ عناصر آن را به آرگومان‌های یک تابع دیگر تبدیل کنید. در زمان اجرا می‌توان عناصر را با یک حلقه پیمایش کرد، اما برای دسترسی مستقل به هر عنصر با استفاده از `std::get`، معمولاً اندیس باید در زمان کامپایل مشخص باشد. اینجا دقیقاً یکی از کاربردهای اصلی `std::index_sequence` نمایان می‌شود.

این قابلیت در استاندارد ++C14 معرفی شده است و در کنار امکاناتی مانند Variadic Templates، Parameter Pack Expansion و `std::tuple`، ابزار قدرتمندی برای نوشتن کدهای عمومی و قابل استفادهٔ مجدد محسوب می‌شود.

## مفهوم `std::integer_sequence`

نوع `std::integer_sequence` یک قالب کلاس (Class Template) است که مجموعه‌ای از مقادیر صحیح را به‌صورت پارامترهای غیرنوعی (Non-type Template Parameters) نگهداری می‌کند.

تعریف مفهومی آن را می‌توان به شکل زیر نمایش داد:

```
template<class T, T... Values>
class integer_sequence;
```

این قطعه صرفاً شکل مفهومی تعریف قالب را نشان می‌دهد؛ برای استفادهٔ واقعی باید نوع استاندارد `std::integer_sequence` را از هدر `<utility>` دریافت کنید.

در این قالب:

- پارامتر `T` نوع مقادیر صحیح را مشخص می‌کند.
- بستهٔ پارامتر `Values...` مجموعه‌ای از مقادیر ثابت زمان کامپایل را تشکیل می‌دهد.
- خود `integer_sequence` یک مجموعهٔ نوعی از مقادیر است، نه یک آرایهٔ زمان اجرا.

برای نمونه، نوع زیر دنباله‌ای شامل سه مقدار صحیح `10`، `20` و `30` را نمایش می‌دهد:

```
#include <utility>

using Values = std::integer_sequence<int, 10, 20, 30>;
```

در این مثال، `Values` نام یک نوع است. این تعریف به‌خودی‌خود هیچ آرایه‌ای ایجاد نمی‌کند و سه مقدار نیز به‌عنوان سه عضو داده‌ای در یک شیء ذخیره نمی‌شوند. اطلاعات دنباله در پارامترهای قالب نوع قرار گرفته‌اند.

### بررسی قابلیت‌های اصلی

قالب `std::integer_sequence` دو عضو مهم برای بررسی اندازه و دسترسی به مقادیر دارد:

```
#include <utility>

using Values = std::integer_sequence<int, 10, 20, 30>;

static_assert(Values::size() == 3);
static_assert(Values::value_type{} == 0);
```

مثال بالا یک نکتهٔ مهم دارد: عبارت دوم روش درستی برای بررسی نوع نیست و صرفاً یک شیء از نوع `int` با مقدار صفر ایجاد می‌کند. برای بررسی نوع، باید از ابزارهایی مانند `std::is_same` استفاده کرد.

نسخهٔ صحیح و کامل‌تر به شکل زیر است:

```
#include <type_traits>
#include <utility>

using Values = std::integer_sequence<int, 10, 20, 30>;

static_assert(Values::size() == 3);
static_assert(
    std::is_same<Values::value_type, int>::value
);
```

عضو `value_type` نوع مقادیر دنباله را مشخص می‌کند و تابع `size()` تعداد مقادیر آن را به‌صورت یک مقدار ثابت زمان کامپایل برمی‌گرداند.

## معرفی خانوادهٔ `std::integer_sequence`

استاندارد ++C14 سه ابزار اصلی برای ساخت و استفاده از دنباله‌های اعداد صحیح معرفی می‌کند که هرکدام هدف مشخصی دارند.

| ابزار                        | کاربرد                                                           |
| ---------------------------- | ---------------------------------------------------------------- |
| `std::integer_sequence`      | نمایش دنباله‌ای از مقادیر صحیح با نوع مشخص                       |
| `std::make_integer_sequence` | تولید خودکار دنباله‌ای از مقادیر صحیح                            |
| `std::index_sequence`        | نمایش دنباله‌ای از اندیس‌های `std::size_t`                       |
| `std::make_index_sequence`   | تولید دنباله‌ای از اندیس‌های متوالی از صفر                       |
| `std::index_sequence_for`    | تولید دنباله‌ای متناسب با تعداد انواع موجود در یک Parameter Pack |

این ابزارها در هدر استاندارد `<utility>` تعریف شده‌اند.

## تفاوت `std::index_sequence` با `std::integer_sequence`

نوع `std::index_sequence` در واقع یک Alias Template برای `std::integer_sequence<std::size_t, Indices...>` است.

شکل مفهومی تعریف آن به این صورت است:

```
template<std::size_t... Indices>
using index_sequence =
    std::integer_sequence<std::size_t, Indices...>;
```

بنابراین، `std::index_sequence` قابلیت مستقلی از `std::integer_sequence` ندارد؛ بلکه نسخه‌ای تخصص‌یافته برای نمایش اندیس‌هاست.

برای مثال، دو تعریف زیر از نظر نوع معادل‌اند:

```
#include <cstddef>
#include <utility>

using A = std::index_sequence<0, 1, 2, 3>;

using B = std::integer_sequence<
    std::size_t, 0, 1, 2, 3
>;

static_assert(std::is_same<A, B>::value);
```

برای کامپایل این مثال باید هدر `<type_traits>` نیز اضافه شود، زیرا `std::is_same` در آن تعریف شده است.

استفاده از `std::index_sequence` زمانی مناسب‌تر است که مقادیر صرفاً برای شماره‌گذاری، دسترسی به عناصر یا بازکردن یک Parameter Pack استفاده می‌شوند. در مقابل، زمانی که دنباله به نوع صحیح دیگری نیاز دارد، `std::integer_sequence` انتخاب عمومی‌تری است.

## نحوهٔ تولید دنباله با `std::make_index_sequence`

قالب `std::make_index_sequence` یکی از مهم‌ترین اجزای این خانواده است. این قالب با دریافت یک عدد صحیح غیرمنفی در زمان کامپایل، نوعی تولید می‌کند که اندیس‌های متوالی از صفر تا یک واحد کمتر از آن عدد را در خود دارد.

شکل استفاده از آن به‌صورت زیر است:

```
std::make_index_sequence<N>
```

برای مثال، عبارت زیر معادل نوع `std::index_sequence<0, 1, 2, 3, 4>` است:

```
#include <utility>

using Indices = std::make_index_sequence<5>;
```

قاعدهٔ تولید دنباله به شکل زیر است:

\\[ \operatorname{make\\\_index\\\_sequence}\langle N\rangle = \operatorname{index\\\_sequence}\langle 0,1,\ldots,N-1\rangle \\]

در این رابطه، اگر \\(N=0\\) باشد، نتیجه دنبالهٔ خالی `std::index_sequence<>` است.

### بررسی چند نمونه

```
#include <type_traits>
#include <utility>

using Empty = std::make_index_sequence<0>;
using One   = std::make_index_sequence<1>;
using Three = std::make_index_sequence<3>;

static_assert(
    std::is_same<Empty, std::index_sequence<>>::value
);

static_assert(
    std::is_same<One, std::index_sequence<0>>::value
);

static_assert(
    std::is_same<
        Three,
        std::index_sequence<0, 1, 2>
    >::value
);
```

نکتهٔ مهم این است که عدد ورودی، تعداد اندیس‌ها را مشخص می‌کند، نه آخرین اندیس را. به همین دلیل، `std::make_index_sequence<3>` سه اندیس `0`، `1` و `2` تولید می‌کند و اندیس `3` در خروجی وجود ندارد.

## تولید دنباله با `std::make_integer_sequence`

قالب `std::make_integer_sequence` همان ایده را با یک نوع صحیح دلخواه پیاده‌سازی می‌کند.

شکل استفاده از آن چنین است:

```
std::make_integer_sequence<T, N>
```

در اینجا `T` نوع مقادیر دنباله و `N` تعداد مقادیر تولیدشده را مشخص می‌کند.

برای نمونه:

```
#include <type_traits>
#include <utility>

using Sequence =
    std::make_integer_sequence<int, 5>;

using Expected =
    std::integer_sequence<int, 0, 1, 2, 3, 4>;

static_assert(
    std::is_same<Sequence, Expected>::value
);
```

این قالب برای تولید دنباله‌های عمومی از مقادیر صحیح مناسب است، اما باید توجه داشت که دنبالهٔ تولیدشده از صفر شروع می‌شود و نوع مقادیر آن `T` است.

برای مثال، با `T = unsigned int`، مقادیر دنباله نیز از نوع `unsigned int` خواهند بود.

### تفاوت تولید دنباله با تعریف دستی

تولید دستی دنباله برای تعداد اندیس کم ممکن است ساده به نظر برسد:

```
using Indices = std::index_sequence<0, 1, 2, 3, 4>;
```

اما وقتی تعداد عناصر از اندازهٔ یک آرایه، یک Tuple یا یک Parameter Pack به دست می‌آید، تعریف دستی دیگر راهکار مناسبی نیست.

در چنین شرایطی، استفاده از `std::make_index_sequence` باعث می‌شود تعداد اندیس‌ها مستقیماً از ساختار داده استخراج شود و با تغییر تعداد عناصر، کد همچنان معتبر بماند.

## کاربرد `std::index_sequence_for`

قالب `std::index_sequence_for` برای زمانی طراحی شده است که به تعداد انواع موجود در یک Parameter Pack نیاز دارید.

شکل مفهومی آن به این صورت است:

```
template<class... Types>
using index_sequence_for =
    std::make_index_sequence<sizeof...(Types)>;
```

در اینجا `sizeof...(Types)` تعداد انواع موجود در بستهٔ پارامتر را مشخص می‌کند.

برای مثال:

```
#include <type_traits>
#include <utility>

using Indices =
    std::index_sequence_for<int, double, char>;

using Expected =
    std::index_sequence<0, 1, 2>;

static_assert(
    std::is_same<Indices, Expected>::value
);
```

این قالب به‌خصوص در توابعی مفید است که هم‌زمان با یک Parameter Pack از انواع، به یک Parameter Pack متناظر از اندیس‌ها نیاز دارند.

نکتهٔ مهم این است که `std::index_sequence_for` تعداد انواع را می‌شمارد؛ مقادیر یا محتوای این انواع را بررسی نمی‌کند.

## مفهوم Parameter Pack Expansion

برای درک کاربرد واقعی `std::index_sequence`، ابتدا باید با سازوکار Parameter Pack Expansion آشنا باشید.

در قالب‌های Variadic، یک Parameter Pack می‌تواند چندین پارامتر را در خود جای دهد. با استفاده از الگوی مناسب، کامپایلر این بسته را به چند عبارت مجزا گسترش می‌دهد.

برای مثال:

```
template<class... Types>
void process(Types... values);
```

تابع بالا می‌تواند تعداد متغیری از آرگومان‌ها را دریافت کند. با این حال، وقتی به دسترسی مستقل به عناصر یک Tuple نیاز داشته باشیم، صرف داشتن Parameter Pack کافی نیست؛ زیرا دسترسی به `std::get<I>` به اندیس `I` در زمان کامپایل نیاز دارد.

در چنین شرایطی، `std::index_sequence` یک بستهٔ اندیس تولید می‌کند و آن بسته را به یک تابع کمکی می‌فرستد تا گسترش اندیس‌ها انجام شود.

برای مثال، یک تابع کمکی می‌تواند از الگویی شبیه به این استفاده کند:

```
template<std::size_t... Indices>
void helper(std::index_sequence<Indices...>);
```

هنگامی که تابع با `std::index_sequence<0, 1, 2>` نمونه‌سازی شود، بستهٔ `Indices...` شامل سه مقدار `0`، `1` و `2` خواهد بود.

این مقادیر سپس می‌توانند در یک Expansion مانند `std::get<Indices>(tuple)...` به کار بروند و سه عبارت مجزا تولید کنند.

## کاربرد عملی در پیمایش `std::tuple`

یکی از رایج‌ترین کاربردهای `std::index_sequence`، دسترسی به تمام عناصر یک Tuple با استفاده از Parameter Pack Expansion است.

در ++C14، می‌توان تابعی نوشت که تمام عناصر یک Tuple را به‌صورت جداگانه به `std::cout` ارسال کند.

```
#include <iostream>
#include <tuple>
#include <utility>

template<class Tuple, std::size_t... Indices>
void printTupleImpl(
    const Tuple& tuple,
    std::index_sequence<Indices...>
) {
    ((std::cout << std::get<Indices>(tuple) << ' '), ...);
}

template<class... Types>
void printTuple(const std::tuple<Types...>& tuple) {
    printTupleImpl(
        tuple,
        std::index_sequence_for<Types...>{}
    );
}

int main() {
    auto values = std::make_tuple(10, 3.14, 'A');

    printTuple(values);
}
```

توجه: عبارت Fold Expression استفاده‌شده در این مثال، یعنی `(..., ...)`، از ویژگی‌های ++C17 است؛ بنابراین این کد در شکل فعلی، نمونه‌ای از راهکار ++C17 است و نباید به‌عنوان کد قابل کامپایل در ++C14 معرفی شود.

در ++C14، باید گسترش بسته را با روشی سازگار با Variadic Templates، مانند یک Initializer List یا یک سازوکار کمکی دیگر، پیاده‌سازی کرد.

نسخهٔ سازگار با ++C14 به این شکل است:

```
#include <iostream>
#include <tuple>
#include <utility>

template<class Tuple, std::size_t... Indices>
void printTupleImpl(
    const Tuple& tuple,
    std::index_sequence<Indices...>
) {
    using Expand = int[];
    (void)Expand{
        0,
        ((std::cout << std::get<Indices>(tuple) << ' '), 0)...
    };
}

template<class... Types>
void printTuple(const std::tuple<Types...>& tuple) {
    printTupleImpl(
        tuple,
        std::index_sequence_for<Types...>{}
    );
}

int main() {
    auto values = std::make_tuple(10, 3.14, 'A');

    printTuple(values);
}
```

در این پیاده‌سازی، عبارت `((std::cout << std::get<Indices>(tuple) << ' '), 0)...` برای هر اندیس یک عبارت ایجاد می‌کند. آرایهٔ کمکی نیز راهی برای واداشتن کامپایلر به ارزیابی این عبارت‌ها در ++C14 فراهم می‌کند.

این روش یکی از الگوهای شناخته‌شدهٔ پیش از Fold Expressions است. در ++C17، Fold Expressions معمولاً خواناتر هستند و برای چنین کاربردهایی انتخاب بهتری محسوب می‌شوند.

## کاربرد پیشرفته در پیاده‌سازی تابعی مشابه `std::apply`

تابع استاندارد `std::apply` در ++C17 معرفی شده است و یک تابع قابل فراخوانی را با عناصر یک Tuple اجرا می‌کند. پیش از معرفی این قابلیت، می‌شد با استفاده از `std::index_sequence` رفتار مشابهی را در ++C14 پیاده‌سازی کرد.

فرض کنید تابعی داریم که سه آرگومان با انواع متفاوت دریافت می‌کند:

```
#include <iostream>

void calculate(int a, double b, char c) {
    std::cout << a << ", " << b << ", " << c << '\n';
}
```

هدف این است که عناصر Tuple را مستقیماً به این تابع ارسال کنیم؛ بدون اینکه لازم باشد نوع یا تعداد آرگومان‌ها را از قبل در تابع کمکی ثابت کنیم.

### پیاده‌سازی نسخهٔ عمومی در ++C14

```
#include <functional>
#include <iostream>
#include <tuple>
#include <type_traits>
#include <utility>

template<class Function, class Tuple, std::size_t... Indices>
auto applyImpl(
    Function&& function,
    Tuple&& tuple,
    std::index_sequence<Indices...>
)
    -> decltype(
        std::forward<Function>(function)(
            std::get<Indices>(std::forward<Tuple>(tuple))...
        )
    )
{
    return std::forward<Function>(function)(
        std::get<Indices>(std::forward<Tuple>(tuple))...
    );
}

template<class Function, class Tuple>
auto apply(Function&& function, Tuple&& tuple)
    -> decltype(
        applyImpl(
            std::forward<Function>(function),
            std::forward<Tuple>(tuple),
            std::make_index_sequence<
                std::tuple_size<
                    typename std::remove_reference<Tuple>::type
                >::value
            >{}
        )
    )
{
    return applyImpl(
        std::forward<Function>(function),
        std::forward<Tuple>(tuple),
        std::make_index_sequence<
            std::tuple_size<
                typename std::remove_reference<Tuple>::type
            >::value
        >{}
    );
}

void calculate(int a, double b, char c) {
    std::cout << a << ", " << b << ", " << c << '\n';
}

int main() {
    auto values = std::make_tuple(10, 3.14, 'A');

    apply(calculate, values);
}
```

در این پیاده‌سازی، چند مفهوم مهم با یکدیگر ترکیب شده‌اند:

- تابع `apply` تعداد عناصر Tuple را با استفاده از `std::tuple_size` به دست می‌آورد.
- قالب `std::make_index_sequence` دنبالهٔ اندیس‌های موردنیاز را تولید می‌کند.
- تابع `applyImpl` بستهٔ اندیس‌ها را با `std::get` گسترش می‌دهد.
- استفاده از `std::forward` امکان حفظ دسته‌بندی مقدار (Value Category) آرگومان‌ها را فراهم می‌کند.
- عبارت `decltype` نوع بازگشتی را متناسب با نتیجهٔ فراخوانی تابع تعیین می‌کند.

نکتهٔ طراحی این است که این کد برای آموزش الگوی پیاده‌سازی مناسب است؛ اما برای استفادهٔ عمومی در یک کتابخانهٔ حرفه‌ای، باید مواردی مانند Tupleهای خالی، توابع دارای خروجی `void`، سازگاری با آرگومان‌های ارجاعی و محدودیت‌های فراخوانی تابع نیز به‌دقت بررسی شوند. همچنین نام `apply` در فضای نام سراسری انتخاب مناسبی برای یک کتابخانهٔ عمومی نیست، زیرا از ++C17 به بعد `std::apply` وجود دارد.

در پروژه‌ای که از ++C17 یا نسخه‌های جدیدتر استفاده می‌کند، در حالت عادی باید تابع استاندارد `std::apply` را ترجیح داد.

## کاربرد در ساخت اشیا از روی آرایه

یکی دیگر از کاربردهای `std::index_sequence`، تبدیل عناصر یک آرایه به آرگومان‌های یک سازنده یا تابع است.

برای مثال، فرض کنید یک کلاس با سه پارامتر سازنده داریم:

```
class Point {
public:
    Point(int x, int y, int z)
        : x_(x), y_(y), z_(z) {}

private:
    int x_;
    int y_;
    int z_;
};
```

اگر مقادیر از قبل در یک `std::array` ذخیره شده باشند، می‌توان با استفاده از اندیس‌های زمان کامپایل آن‌ها را به سازنده ارسال کرد.

```
#include <array>
#include <cstddef>
#include <utility>

class Point {
public:
    Point(int x, int y, int z)
        : x_(x), y_(y), z_(z) {}

private:
    int x_;
    int y_;
    int z_;
};

template<class T, std::size_t Size, std::size_t... Indices>
Point makePointImpl(
    const std::array<T, Size>& values,
    std::index_sequence<Indices...>
) {
    return Point(values[Indices]...);
}

template<class T, std::size_t Size>
Point makePoint(const std::array<T, Size>& values) {
    static_assert(Size == 3, "Point requires exactly three values");

    return makePointImpl(
        values,
        std::make_index_sequence<Size>{}
    );
}
```

این نمونه نشان می‌دهد که `std::index_sequence` چگونه می‌تواند یک ساختار دادهٔ دارای اندیس را به یک بستهٔ آرگومان تبدیل کند.

بااین‌حال، نوع بازگشتی `Point` در این مثال ثابت است و تعداد عناصر نیز باید دقیقاً سه باشد. برای طراحی عمومی‌تر، می‌توان نوع کلاس مقصد و تعداد آرگومان‌های سازنده را نیز به‌صورت پارامتری تعریف کرد.

## کاربرد در اجرای یک عملیات برای هر اندیس

گاهی هدف نه دسترسی به Tuple، بلکه اجرای یک عملیات برای مجموعه‌ای از اندیس‌هاست. برای مثال، ممکن است بخواهید تابعی را برای هر اندیس از صفر تا `N - 1` فراخوانی کنید.

در ++C14 می‌توان از الگوی Initializer List برای ایجاد ترتیب مشخص ارزیابی استفاده کرد:

```
#include <cstddef>
#include <iostream>
#include <utility>

template<std::size_t... Indices>
void printIndices(std::index_sequence<Indices...>) {
    using Expand = int[];

    (void)Expand{
        0,
        ((std::cout << Indices << '\n'), 0)...
    };
}

int main() {
    printIndices(std::make_index_sequence<5>{});
}
```

خروجی این برنامه چنین است:

```
0
1
2
3
4
```

در این مثال، هر مقدار از بستهٔ `Indices...` در یک عبارت مستقل گسترش داده می‌شود.

این تکنیک به‌خصوص برای مواردی مفید است که در زمان کامپایل مجموعه‌ای از عملیات مشابه تولید می‌شوند، اما هر عملیات باید با یک اندیس متفاوت اجرا شود.

البته اگر عملیات موردنظر صرفاً یک حلقهٔ معمولی روی داده‌های زمان اجرا باشد، استفاده از یک حلقهٔ عادی ساده‌تر و خواناتر است.

## نحوهٔ استفاده از `std::index_sequence` در Template Metaprogramming

در Template Metaprogramming، انواع می‌توانند حامل اطلاعاتی باشند که در زمان کامپایل بررسی و پردازش می‌شوند.

برای نمونه، می‌توان یک قالب تابع نوشت که تعداد اندیس‌های یک دنباله را گزارش کند:

```
#include <cstddef>
#include <iostream>
#include <utility>

template<std::size_t... Indices>
void reportSize(std::index_sequence<Indices...>) {
    std::cout << sizeof...(Indices) << '\n';
}

int main() {
    reportSize(std::make_index_sequence<8>{});
}
```

در اینجا `sizeof...(Indices)` تعداد اعضای بستهٔ پارامتر را در زمان کامپایل محاسبه می‌کند. در نتیجه، تابع برای دنباله‌ای با هشت اندیس، عدد `8` را چاپ می‌کند.

این مثال ساده است، اما همین الگو می‌تواند در ساخت قالب‌های پیچیده‌تر برای Tupleها، کلاس‌های عمومی، تبدیل ساختار داده‌ها و پیاده‌سازی الگوریتم‌های مبتنی بر تعداد پارامترها استفاده شود.

## پیاده‌سازی مفهومی `std::make_index_sequence`

از نظر استاندارد، توسعه‌دهنده ملزم نیست الگوریتم داخلی تولید دنباله را بداند. بااین‌حال، شناخت روش‌های رایج پیاده‌سازی به درک بهتر هزینهٔ کامپایل و طراحی قالب‌های عمومی کمک می‌کند.

یک روش آموزشی، تولید بازگشتی دنباله است. در این روش، برای رسیدن به دنباله‌ای با طول `N`، ابتدا دنباله‌ای با طول `N - 1` ساخته می‌شود و سپس اندیس جدید به آن اضافه می‌شود.

نسخهٔ ساده‌شدهٔ زیر صرفاً برای توضیح مفهوم است و جایگزین کامل پیاده‌سازی استاندارد نیست:

```
#include <cstddef>

template<std::size_t... Indices>
struct IndexSequence {};

template<std::size_t N, std::size_t... Indices>
struct MakeIndexSequenceImpl
    : MakeIndexSequenceImpl<N - 1, N - 1, Indices...> {};

template<std::size_t... Indices>
struct MakeIndexSequenceImpl<0, Indices...> {
    using type = IndexSequence<Indices...>;
};

template<std::size_t N>
using MakeIndexSequence =
    typename MakeIndexSequenceImpl<N>::type;
```

برای نمونه، با درخواست `MakeIndexSequence<3>`، مراحل بازگشتی به تولید مقادیر `0`، `1` و `2` منجر می‌شوند.

این پیاده‌سازی از نظر ساختار، یک روش بازگشتی ساده برای تولید دنباله است؛ اما نباید نتیجه گرفت که کتابخانهٔ استاندارد الزاماً از همین الگوریتم استفاده می‌کند. پیاده‌سازی‌های واقعی می‌توانند از روش‌های بهینه‌تر مبتنی بر تقسیم و غلبه یا قابلیت‌های داخلی کامپایلر بهره ببرند.

همچنین این نمونه برای ورودی‌های نامعتبر، مانند برخی مقادیر خارج از دامنهٔ قابل نمایش در نوع مورد استفاده، کنترل جامع ندارد. هدف آن تنها نمایش ایدهٔ تولید دنباله در زمان کامپایل است.

## نکات مهم دربارهٔ هزینهٔ کامپایل

استفاده از `std::index_sequence` به‌خودی‌خود باعث اجرای یک حلقه در زمان اجرا نمی‌شود. دنباله در قالب نوعی نمایش داده می‌شود و کامپایلر از آن برای تولید نمونه‌های لازم از قالب‌ها و گسترش بسته‌ها استفاده می‌کند.

بااین‌حال، برای دنباله‌های بسیار بزرگ، پیچیدگی پیاده‌سازی قالب‌ها و تعداد نمونه‌سازی‌ها می‌تواند زمان کامپایل و مصرف حافظهٔ کامپایلر را افزایش دهد.

نکتهٔ مهم این است که هزینهٔ واقعی به ساختار قالب‌های مصرف‌کننده، الگوریتم تولید دنباله و پیاده‌سازی کامپایلر بستگی دارد. بنابراین نباید بدون اندازه‌گیری نتیجه گرفت که همهٔ کاربردهای `std::index_sequence` هزینهٔ یکسانی دارند.

برای پروژه‌های بزرگ، بهتر است از تولید چندبارهٔ دنباله‌های مشابه در ساختارهای غیرضروری خودداری شود و در صورت مشاهدهٔ مشکل، زمان کامپایل با ابزارهای مناسب اندازه‌گیری شود.

## محدودیت‌ها و خطاهای رایج

### اشتباه گرفتن تعداد عناصر با آخرین اندیس

قالب `std::make_index_sequence<N>` دنباله‌ای با `N` عضو تولید می‌کند، نه دنباله‌ای که آخرین عضو آن `N` باشد.

برای مثال، `std::make_index_sequence<4>` معادل `std::index_sequence<0, 1, 2, 3>` است.

### استفاده از اندیس خارج از محدوده

تولید دنباله به معنای تضمین معتبر بودن اندیس برای هر ساختار داده نیست. اگر از دنباله‌ای بزرگ‌تر از تعداد عناصر یک آرایه یا Tuple استفاده کنید، دسترسی حاصل ممکن است نامعتبر باشد یا باعث خطای کامپایل شود.

بنابراین، تعداد اندیس‌ها باید با اندازهٔ ساختار داده هماهنگ باشد.

### فراموش کردن حالت دنبالهٔ خالی

در ++C14، `std::make_index_sequence<0>` یک دنبالهٔ خالی معتبر تولید می‌کند. بااین‌حال، کد مصرف‌کننده باید بتواند بستهٔ پارامتر خالی را نیز مدیریت کند.

برخی پیاده‌سازی‌های مبتنی بر Initializer List یا توابع کمکی ممکن است به دلیل فرض‌های نادرست دربارهٔ تعداد عناصر، برای این حالت به اصلاح نیاز داشته باشند.

### تصور اینکه دنباله خودش داده نگهداری می‌کند

نوع `std::index_sequence` یک آرایه یا مجموعهٔ زمان اجرا نیست. این نوع صرفاً مجموعه‌ای از مقادیر صحیح در پارامترهای قالب است. استفاده از آن برای دسترسی به داده‌ها به یک ساختار دادهٔ واقعی یا عملیاتی که آن مقادیر را مصرف کند نیاز دارد.

### استفادهٔ غیرضروری در کدهای ساده

اگر هدف صرفاً پیمایش عناصر یک آرایهٔ معمولی در زمان اجرا باشد، حلقهٔ `for` یا Range-based `for` اغلب انتخاب ساده‌تر و مناسب‌تری است.

`std::index_sequence` زمانی ارزش بیشتری دارد که به اندیس‌های ثابت زمان کامپایل، گسترش Parameter Pack یا تولید مجموعه‌ای از عبارت‌های متفاوت نیاز داشته باشید.

## مقایسهٔ `std::index_sequence` با امکانات نسخه‌های جدیدتر

قابلیت‌های جدیدتر زبان ++C برخی کاربردهای متداول `std::index_sequence` را ساده‌تر کرده‌اند، اما این ابزار همچنان برای تولید و انتقال بسته‌های اندیس کاربرد دارد.

| قابلیت                                | نسخهٔ استاندارد | تفاوت اصلی                                |
| ------------------------------------- | --------------- | ----------------------------------------- |
| `std::integer_sequence` و خانوادهٔ آن | ++C14           | نمایش و تولید دنباله‌های اعداد صحیح       |
| `std::apply`                          | ++C17           | ارسال عناصر Tuple به یک تابع              |
| Fold Expressions                      | ++C17           | ساده‌سازی اجرای عملیات روی Parameter Pack |
| Concepts                              | ++C20           | محدودسازی و مستندسازی بهتر قالب‌های عمومی |

برای مثال، در ++C17، بسیاری از کاربردهای ارسال عناصر Tuple را می‌توان با `std::apply` انجام داد و در بسیاری از عملیات روی بسته‌ها نیز Fold Expressions جایگزین الگوهای قدیمی‌تر مبتنی بر Initializer List می‌شوند.

بااین‌حال، این قابلیت‌ها جایگزین کامل `std::index_sequence` نیستند؛ زیرا خود `std::apply` نیز برای پیاده‌سازی‌های متداول به نوعی سازوکار دسترسی به اندیس‌های Tuple نیاز دارد.

## جمع‌بندی

نوع `std::index_sequence` یکی از ابزارهای پایه‌ای و مفید برای برنامه‌نویسی قالبی در ++C14 است. این ابزار مجموعه‌ای از اندیس‌های ثابت زمان کامپایل را نمایش می‌دهد و امکان گسترش یک Parameter Pack را در موقعیت‌هایی فراهم می‌کند که به اندیس‌های مستقل نیاز دارند.

مهم‌ترین نکاتی که باید به خاطر بسپارید عبارت‌اند از:

- نوع `std::index_sequence` از خانوادهٔ `std::integer_sequence` است و اندیس‌ها را با نوع `std::size_t` نمایش می‌دهد.
- قالب `std::make_index_sequence<N>` دنباله‌ای شامل `N` اندیس از صفر تا `N - 1` تولید می‌کند.
- قالب `std::index_sequence_for<Types...>` تعداد انواع موجود در یک Parameter Pack را به تعداد اندیس‌های دنباله تبدیل می‌کند.
- کاربرد مهم این ابزار در دسترسی به عناصر Tuple، گسترش بسته‌ها و ارسال آرگومان‌های متغیر به توابع و سازنده‌هاست.
- در ++C14، الگوهای مبتنی بر Initializer List راهی رایج برای گسترش بسته‌ها و ترتیب‌دهی ارزیابی عبارت‌ها هستند.
- در ++C17 و نسخه‌های جدیدتر، قابلیت‌هایی مانند `std::apply` و Fold Expressions بسیاری از این الگوها را خواناتر می‌کنند.
- استفاده از `std::index_sequence` زمانی ارزشمند است که ساختار مسئله واقعاً به اندیس‌های زمان کامپایل نیاز داشته باشد؛ نه صرفاً برای جایگزین کردن حلقه‌های معمولی.

در نهایت، بهترین راه برای تسلط بر این قابلیت، درک ارتباط آن با Variadic Templates و Parameter Pack Expansion است. وقتی این ارتباط روشن شود، `std::index_sequence` دیگر یک ابزار پیچیده و مرموز به نظر نمی‌رسد، بلکه راهکاری طبیعی برای تبدیل مجموعه‌ای از اندیس‌های ثابت زمان کامپایل به عبارت‌های قابل استفاده در قالب‌های عمومی خواهد بود.

---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>