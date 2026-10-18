<div align="center">

[🇺🇸 English](../../en/cpp14/std_integer_sequence.md) | [🇮🇷 فارسی](./std_integer_sequence.md)

</div>

---

# آموزش جامع `std::integer_sequence` در ++C14

## مقدمه

قابلیت `std::integer_sequence` یکی از ابزارهای مهم Standard Library در استاندارد ++C14 است که برای نمایش مجموعه‌ای از مقادیر صحیح به‌صورت یک دنبالهٔ کاملاً مشخص در زمان کامپایل طراحی شده است. این ابزار به برنامه‌نویس اجازه می‌دهد مقادیر عددی را به‌صورت Template Parameter Pack نگه‌داری کند و در صورت نیاز، آن‌ها را به آرگومان‌های یک تابع یا عملیات دیگری در زمان کامپایل تبدیل کند.

اهمیت این قابلیت زمانی بیشتر مشخص می‌شود که بخواهیم با تعداد متغیری از مقادیر در سطح Templateها کار کنیم؛ برای مثال، اعضای یک `std::tuple` را به‌صورت جداگانه استخراج کنیم، یک Tuple را به آرگومان‌های تابع تبدیل کنیم یا عملیاتی را روی عناصر یک ساختار داده در زمان کامپایل گسترش دهیم.

پیش از معرفی این ابزار، باید به یک نکتهٔ مهم توجه کنیم: `std::integer_sequence` خودش یک Container نیست، عناصر را در حافظه نگه نمی‌دارد و الگوریتمی برای پیمایش مقادیر در زمان اجرا ارائه نمی‌کند. این نوع در واقع یک ساختار Type-Level است که مقادیر صحیح را به‌عنوان Non-Type Template Parameters در نوع خود رمزگذاری می‌کند.

این تفاوت، اساس درک صحیح کاربردهای آن است.

## مفهوم دنبالهٔ اعداد در زمان کامپایل

### معرفی `std::integer_sequence`

نوع `std::integer_sequence` در هدر استاندارد `<utility>` تعریف شده است و شکل کلی آن به این صورت است:

```
template<class T, T... Vals>
struct integer_sequence;
```

در این تعریف، پارامتر `T` نوع تمام مقادیر دنباله را مشخص می‌کند و `Vals...` یک Parameter Pack از مقادیر ثابت در زمان کامپایل است.

برای مثال، تعریف زیر یک نوع شامل دنباله‌ای از پنج عدد صحیح را ایجاد می‌کند:

```
#include <utility>

using Sequence = std::integer_sequence<int, 10, 20, 30, 40, 50>;
```

در این مثال، نوع `Sequence` اطلاعات زیر را در خود رمزگذاری می‌کند:

- نوع مقادیر برابر `int` است.
- تعداد مقادیر برابر پنج است.
- مقادیر دنباله به‌ترتیب برابر `10`، `20`، `30`، `40` و `50` هستند.

این اعداد به‌عنوان مقادیر Template در تعریف نوع حضور دارند؛ بنابراین برای استفاده از آن‌ها لازم نیست پنج متغیر مستقل در حافظه ایجاد کنیم.

### بررسی یک دنباله با مقادیر متفاوت

دنباله الزاماً نباید از صفر شروع شود یا اعداد متوالی داشته باشد.

```
#include <utility>

using A = std::integer_sequence<int, 3, 7, 11>;
using B = std::integer_sequence<int, -4, 0, 100>;
using C = std::integer_sequence<unsigned, 2, 4, 8, 16>;
```

در این مثال‌ها، هر نوع دنباله‌ای مستقل با مقادیر و نوع عددی مشخص خود را نشان می‌دهد.

نکتهٔ مهم این است که `std::integer_sequence` یک نوع مشخص را توصیف می‌کند، نه یک متغیر یا آرایه را. به همین دلیل، دو دنباله با مقادیر متفاوت، دو نوع متفاوت خواهند بود؛ حتی اگر تعداد عناصر یا نوع عددی آن‌ها یکسان باشد.

## آشنایی با اعضای استاندارد

### بررسی `value_type`

این نوع تو‌در‌تو، نوع مقادیر دنباله را مشخص می‌کند.

```
#include <type_traits>
#include <utility>

using Sequence = std::integer_sequence<long, 1, 2, 3>;

static_assert(
    std::is_same<Sequence::value_type, long>::value,
    "Unexpected value type"
);
```

در این مثال، نوع `value_type` برابر `long` است؛ زیرا نوعی که هنگام تعریف دنباله مشخص شده، `long` بوده است.

### بررسی `size()`

تابع `size()` تعداد مقادیر دنباله را در زمان کامپایل برمی‌گرداند. مقدار بازگشتی آن از نوع `std::size_t` است.

```
#include <cassert>
#include <utility>

using Sequence = std::integer_sequence<int, 10, 20, 30>;

static_assert(Sequence::size() == 3, "Unexpected sequence size");

int main() {
    assert(Sequence::size() == 3);
}
```

در این مثال، `static_assert` بررسی را در زمان کامپایل انجام می‌دهد. فراخوانی `size()` در `main` نیز از نظر زبان مجاز است، اما در این مورد نیازی به محاسبه در زمان اجرا وجود ندارد.

### بررسی دنبالهٔ خالی

یک دنباله می‌تواند هیچ مقداری نداشته باشد:

```
#include <utility>

using EmptySequence = std::integer_sequence<int>;

static_assert(
    EmptySequence::size() == 0,
    "Sequence must be empty"
);
```

دنبالهٔ خالی در پیاده‌سازی Templateهای عمومی اهمیت دارد؛ زیرا تعداد عناصر یک Parameter Pack ممکن است صفر باشد. طراحی صحیح کد باید این حالت را نیز پوشش دهد.

## تفاوت `std::integer_sequence` با `std::index_sequence`

### معرفی `std::index_sequence`

نوع `std::index_sequence` یک Alias Template استاندارد است که برای نمایش دنباله‌ای از اندیس‌های `std::size_t` طراحی شده است.

تعریف مفهومی آن به این صورت است:

```
template<std::size_t... Indices>
using index_sequence =
    std::integer_sequence<std::size_t, Indices...>;
```

بنابراین، دنبالهٔ زیر:

```
using Indices = std::index_sequence<0, 1, 2, 3>;
```

از نظر نوع معادل این دنباله است:

```
using Indices = std::integer_sequence<std::size_t, 0, 1, 2, 3>;
```

مزیت `std::index_sequence` این است که هدف استفاده از دنباله را روشن می‌کند: مقادیر آن اندیس هستند، نه لزوماً داده‌های عددی دلخواه.

### مقایسهٔ کاربردها

برای نمایش مقادیر عددی دلخواه، `std::integer_sequence` انتخاب طبیعی‌تری است:

```
using Values = std::integer_sequence<int, 10, 20, 30>;
```

اما برای دسترسی به عناصر یک Tuple، استفاده از `std::index_sequence` خواناتر است:

```
using Indices = std::index_sequence<0, 1, 2>;
```

هر دو نوع از یک سازوکار اساسی استفاده می‌کنند. تفاوت آن‌ها بیشتر در نوع مقادیر، هدف کاربرد و خوانایی کد است.

## تولید خودکار دنباله‌ها در زمان کامپایل

### معرفی `std::make_integer_sequence`

در بسیاری از موارد، نوشتن تمام مقادیر دنباله به‌صورت دستی منطقی نیست. تصور کنید بخواهیم دنباله‌ای شامل تمام اعداد از صفر تا \\(N-1\\) تولید کنیم. اگر مقدار \\(N\\) یک پارامتر Template باشد، تعداد عناصر دنباله نیز باید به‌صورت خودکار تعیین شود.

برای حل این مسئله، استاندارد ++C14 ابزار `std::make_integer_sequence` را ارائه می‌کند.

```
template<class T, T N>
using make_integer_sequence = /* implementation-defined */;
```

این Alias Template دنباله‌ای با نوع `T` تولید می‌کند که مقادیر آن از صفر تا \\(N-1\\) هستند.

برای مثال:

```
#include <utility>

using Sequence = std::make_integer_sequence<int, 5>;
```

نوع تولیدشده معادل این تعریف است:

```
using Sequence = std::integer_sequence<int, 0, 1, 2, 3, 4>;
```

در اینجا، تعداد عناصر پنج است؛ اما آخرین مقدار برابر چهار خواهد بود.

### بررسی حالت‌های مرزی

رفتار این ابزار در حالت‌های مرزی اهمیت زیادی دارد.

```
#include <utility>

using Empty = std::make_integer_sequence<int, 0>;
using Single = std::make_integer_sequence<int, 1>;
using Multiple = std::make_integer_sequence<int, 4>;
```

دنباله‌های تولیدشده به‌ترتیب عبارت‌اند از:

```
using Empty = std::integer_sequence<int>;
using Single = std::integer_sequence<int, 0>;
using Multiple = std::integer_sequence<int, 0, 1, 2, 3>;
```

در نتیجه، برای مقدار صفر یک دنبالهٔ خالی تولید می‌شود و برای مقدار یک، دنباله فقط شامل صفر است.

اما اگر مقدار `N` منفی باشد، استفاده از `std::make_integer_sequence` معتبر نیست و برنامه با توجه به الزامات استاندارد و محدودیت‌های پیاده‌سازی، با خطای کامپایل مواجه خواهد شد. بنابراین بهتر است در Templateهای عمومی، نامعتبر بودن مقادیر منفی را به‌عنوان یکی از محدودیت‌های ورودی در نظر بگیریم.

نکتهٔ دیگر این است که نوع `N` باید با الزامات Non-Type Template Parameter این Alias سازگار باشد. همچنین نوع `T` باید از انواع صحیح مجاز برای چنین پارامتری باشد.

### معرفی `std::make_index_sequence`

نوع `std::make_index_sequence` نسخهٔ تخصصی‌شده‌ای از ایدهٔ تولید خودکار دنباله برای اندیس‌ها است.

```
template<std::size_t N>
using make_index_sequence =
    std::make_integer_sequence<std::size_t, N>;
```

برای مثال:

```
#include <utility>

using Indices = std::make_index_sequence<4>;
```

نوع حاصل معادل این دنباله است:

```
using Indices = std::index_sequence<0, 1, 2, 3>;
```

این ابزار برای زمانی طراحی شده که تعداد عناصر را می‌دانیم، اما نمی‌خواهیم اندیس‌های آن‌ها را به‌صورت دستی بنویسیم.

### معرفی `std::index_sequence_for`

نوع `std::index_sequence_for` زمانی مفید است که بخواهیم دنباله‌ای از اندیس‌ها را بر اساس تعداد پارامترهای یک Parameter Pack تولید کنیم.

```
template<class... Types>
using index_sequence_for =
    std::make_index_sequence<sizeof...(Types)>;
```

برای مثال:

```
#include <utility>

using Indices = std::index_sequence_for<int, double, char>;
```

نوع حاصل معادل این دنباله است:

```
using Indices = std::index_sequence<0, 1, 2>;
```

تفاوت مهم این ابزار با `std::make_index_sequence` در نحوهٔ تعیین تعداد عناصر است:

- در `std::make_index_sequence` تعداد اندیس‌ها را مستقیماً مشخص می‌کنیم.
- در `std::index_sequence_for` تعداد اندیس‌ها از تعداد انواع موجود در Parameter Pack به دست می‌آید.

این ویژگی در پیاده‌سازی Templateهایی که با Tupleها یا مجموعه‌ای از انواع کار می‌کنند، بسیار کاربردی است.

## نحوهٔ استفاده از دنباله برای گسترش پارامترها

### مفهوم Parameter Pack Expansion

برای درک کاربرد اصلی `std::integer_sequence`، ابتدا باید با مفهوم Parameter Pack Expansion آشنا باشیم.

در ++C می‌توان یک مجموعه از پارامترهای Template را با استفاده از `...` گسترش داد. این قابلیت اجازه می‌دهد عملیات مشابهی روی تمام عناصر یک Parameter Pack انجام شود.

برای مثال:

```
#include <iostream>

template<int... Values>
void printValues() {
    ((std::cout << Values << ' '), ...);
}

int main() {
    printValues<10, 20, 30>();
}
```

این مثال از Fold Expression استفاده می‌کند که از استاندارد ++C17 در دسترس است. بنابراین برای کدی که باید با ++C14 سازگار باشد، باید از روش دیگری استفاده کنیم.

نکتهٔ اصلی این است که `std::integer_sequence` به‌تنهایی عملیات گسترش را انجام نمی‌دهد؛ بلکه یک Parameter Pack از مقادیر را در اختیار Template دیگری قرار می‌دهد تا آن Template بتواند عملیات موردنظر را انجام دهد.

### پیاده‌سازی گسترش پارامترها در ++C14

در ++C14 هنوز Fold Expression وجود ندارد. یکی از روش‌های رایج برای اجرای عملیاتی روی تمام عناصر یک Parameter Pack، استفاده از گسترش پارامترها در یک Initializer List است.

```
#include <iostream>

template<int... Values>
void printValues() {
    using Expand = int[];

    (void)Expand{0, (std::cout << Values << ' ', 0)...};
}

int main() {
    printValues<10, 20, 30>();
}
```

در این روش، عبارت داخل Initializer List برای هر مقدار از `Values...` گسترش می‌یابد.

عدد صفر ابتدایی باعث می‌شود حتی در حالت خالی بودن Parameter Pack نیز Initializer List معتبر باقی بماند. در این مثال، خروجی به این صورت خواهد بود:

```
10 20 30
```

البته برای کدهای عمومی باید به نوع عناصر، اثرات جانبی عملیات و الزامات ترتیب ارزیابی توجه کرد. در این الگو، عناصر Initializer List به‌ترتیب ارزیابی می‌شوند.

## کاربرد عملی برای دسترسی به عناصر `std::tuple`

### مشکل دسترسی به عناصر Tuple

نوع `std::tuple` می‌تواند مجموعه‌ای از مقادیر با انواع متفاوت را نگه‌داری کند. برای دسترسی به هر عنصر، معمولاً از `std::get` استفاده می‌کنیم.

```
#include <iostream>
#include <tuple>

int main() {
    auto values = std::make_tuple(10, 3.14, 'A');

    std::cout << std::get<0>(values) << '\n';
    std::cout << std::get<1>(values) << '\n';
    std::cout << std::get<2>(values) << '\n';
}
```

در این مثال، اندیس هر عنصر باید در زمان کامپایل مشخص باشد. به همین دلیل، نمی‌توان به‌سادگی از یک حلقهٔ معمولی با اندیس زمان اجرا برای فراخوانی `std::get<i>` استفاده کرد؛ زیرا پارامتر Template این تابع باید یک مقدار ثابت در زمان کامپایل باشد.

اینجاست که `std::index_sequence` کاربرد خود را نشان می‌دهد.

### پیاده‌سازی تابعی برای چاپ تمام عناصر Tuple

می‌توان یک دنبالهٔ اندیس تولید کرد و سپس آن را به یک تابع کمکی فرستاد.

```
#include <iostream>
#include <tuple>
#include <utility>

template<class Tuple, std::size_t... Indices>
void printTupleImpl(const Tuple& tuple,
                    std::index_sequence<Indices...>) {
    using Expand = int[];

    (void)Expand{
        0,
        (static_cast<void>(
            std::cout << std::get<Indices>(tuple) << ' '
        ), 0)...
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

خروجی برنامه به این صورت است:

```
10 3.14 A
```

در این پیاده‌سازی، چند مفهوم مهم در کنار یکدیگر قرار گرفته‌اند:

- تابع `printTuple` نوع عناصر Tuple را از طریق Template Deduction استخراج می‌کند.
- ابزار `std::index_sequence_for` دنباله‌ای متناسب با تعداد عناصر Tuple می‌سازد.
- پارامتر `std::index_sequence<Indices...>` مقادیر اندیس‌ها را به تابع کمکی منتقل می‌کند.
- عبارت `std::get<Indices>(tuple)...` برای تمام اندیس‌ها گسترش می‌یابد.

در نتیجه، به‌جای نوشتن یک فراخوانی جداگانه برای هر عنصر، می‌توان با یک Template عمومی تمام عناصر را پردازش کرد.

### بررسی Tuple خالی

یک پیاده‌سازی عمومی باید حالت Tuple خالی را نیز مدیریت کند.

```
#include <iostream>
#include <tuple>
#include <utility>

template<class Tuple, std::size_t... Indices>
void printTupleImpl(const Tuple& tuple,
                    std::index_sequence<Indices...>) {
    using Expand = int[];

    (void)Expand{
        0,
        (static_cast<void>(
            std::cout << std::get<Indices>(tuple) << ' '
        ), 0)...
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
    std::tuple<> empty;

    printTuple(empty);
}
```

در این مثال، `Indices...` خالی خواهد بود و هیچ فراخوانی برای `std::get` تولید نمی‌شود. همچنین وجود عنصر ابتدایی صفر باعث می‌شود Initializer List همچنان معتبر باشد.

این نمونه نشان می‌دهد که استفاده از دنباله‌های اندیس چگونه می‌تواند طراحی Templateها را نسبت به نوشتن کدهای جداگانه برای حالت‌های مختلف ساده‌تر کند.

## کاربرد عملی برای تبدیل Tuple به آرگومان‌های تابع

### مفهوم Tuple Unpacking

یکی از کاربردهای مهم `std::index_sequence` تبدیل عناصر یک Tuple به آرگومان‌های مستقل یک تابع است. این تکنیک معمولاً با نام Tuple Unpacking یا Tuple Expansion شناخته می‌شود.

برای مثال، فرض کنید تابعی داریم که سه آرگومان دریافت می‌کند:

```
int add(int a, int b, int c) {
    return a + b + c;
}
```

اکنون می‌خواهیم مقادیر ذخیره‌شده در یک Tuple را مستقیماً به این تابع ارسال کنیم.

### پیاده‌سازی تابع کمکی

در ++C14 می‌توان چنین قابلیتی را با ترکیب `std::index_sequence`، `std::get` و Parameter Pack Expansion پیاده‌سازی کرد.

```
#include <iostream>
#include <tuple>
#include <utility>

template<class Function, class Tuple, std::size_t... Indices>
auto applyImpl(Function&& function,
               Tuple&& tuple,
               std::index_sequence<Indices...>)
    -> decltype(
        std::forward<Function>(function)(
            std::get<Indices>(
                std::forward<Tuple>(tuple)
            )...
        )
    ) {
    return std::forward<Function>(function)(
        std::get<Indices>(
            std::forward<Tuple>(tuple)
        )...
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
                    typename std::decay<Tuple>::type
                >::value
            >{}
        )
    ) {
    return applyImpl(
        std::forward<Function>(function),
        std::forward<Tuple>(tuple),
        std::make_index_sequence<
            std::tuple_size<
                typename std::decay<Tuple>::type
            >::value
        >{}
    );
}

int add(int a, int b, int c) {
    return a + b + c;
}

int main() {
    auto values = std::make_tuple(10, 20, 30);

    std::cout << apply(add, values) << '\n';
}
```

خروجی برنامه برابر است با:

```
60
```

در این مثال، تعداد اندیس‌ها از `std::tuple_size` استخراج می‌شود و سپس `std::make_index_sequence` دنبالهٔ موردنیاز را تولید می‌کند.

تابع کمکی `applyImpl` نیز عناصر Tuple را با استفاده از `std::get` استخراج کرده و به‌صورت آرگومان‌های جداگانه به تابع مقصد می‌فرستد.

نکتهٔ مهم این است که این پیاده‌سازی آموزشی برای نمایش سازوکار اصلی نوشته شده است. در استاندارد ++C17، تابع استاندارد `std::apply` همین هدف عمومی را دنبال می‌کند و معمولاً باید به آن ترجیح داده شود.

### تفاوت با `std::apply`

تابع `std::apply` از استاندارد ++C17 در هدر `<tuple>` در دسترس است.

```
#include <iostream>
#include <tuple>

int add(int a, int b, int c) {
    return a + b + c;
}

int main() {
    auto values = std::make_tuple(10, 20, 30);

    std::cout << std::apply(add, values) << '\n';
}
```

اگر پروژه به ++C14 محدود باشد، می‌توان از پیاده‌سازی سفارشی استفاده کرد؛ اما در ++C17 و نسخه‌های جدیدتر، استفاده از ابزار استاندارد معمولاً خواناتر و کم‌خطاتر است.

## پیاده‌سازی الگوریتم‌های عمومی با دنباله‌های اعداد

### تولید آرایه از مقادیر دنباله

گاهی لازم است مقادیر یک دنبالهٔ کامپایل‌تایم را به یک ساختار داده در زمان اجرا تبدیل کنیم. برای نمونه، می‌توان یک `std::array` را با استفاده از مقادیر دنباله مقداردهی اولیه کرد.

```
#include <array>
#include <iostream>
#include <utility>

template<class T, T... Values>
constexpr auto toArray(std::integer_sequence<T, Values...>) {
    return std::array<T, sizeof...(Values)>{{Values...}};
}

int main() {
    constexpr auto values =
        toArray(std::integer_sequence<int, 10, 20, 30>{});

    static_assert(values.size() == 3, "Unexpected size");

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

خروجی برنامه به این صورت خواهد بود:

```
10 20 30
```

در این مثال، مقادیر Template مستقیماً برای مقداردهی اولیهٔ آرایه استفاده می‌شوند. تفاوت مهم این است که برخلاف خود `std::integer_sequence`، شیء `std::array` واقعاً عناصر را به‌عنوان داده نگه‌داری می‌کند.

### محاسبهٔ مجموع دنباله در زمان کامپایل

یکی دیگر از کاربردهای آموزشی، محاسبهٔ مجموع مقادیر یک دنباله است.

```
#include <utility>

template<class T, T... Values>
constexpr T sumSequence(std::integer_sequence<T, Values...>) {
    return (T{} + ... + Values);
}

int main() {
    constexpr auto result =
        sumSequence(std::integer_sequence<int, 1, 2, 3, 4>{});

    static_assert(result == 10, "Unexpected sum");
}
```

این مثال از Fold Expression استفاده می‌کند و به ++C17 نیاز دارد؛ بنابراین نباید آن را به‌عنوان پیاده‌سازی سازگار با ++C14 در نظر گرفت.

در ++C14 می‌توان همین عملیات را با Recursion در سطح Template یا با یک الگوی جایگزین مبتنی بر گسترش پارامترها پیاده‌سازی کرد.

نکتهٔ مهم‌تر این است که `std::integer_sequence` الزاماً به معنی انجام یک محاسبه در زمان کامپایل نیست. این نوع فقط مقادیر را در سطح نوع رمزگذاری می‌کند. اینکه عملیات روی آن‌ها واقعاً در زمان کامپایل انجام شود، به قواعد `constexpr`، نحوهٔ استفاده از نتیجه و الزامات ارزیابی ثابت بستگی دارد.

## مقایسهٔ روش‌های تولید دنباله

### تفاوت ابزارهای اصلی

| ابزار                        | کاربرد اصلی                            | نوع نتیجه                                            |
| ---------------------------- | -------------------------------------- | ---------------------------------------------------- |
| `std::integer_sequence`      | نمایش مقادیر صحیح دلخواه               | یک نوع دنباله                                        |
| `std::index_sequence`        | نمایش دنباله‌ای از اندیس‌ها            | Alias برای `std::integer_sequence<std::size_t, ...>` |
| `std::make_integer_sequence` | تولید خودکار اعداد از صفر تا \\(N-1\\) | یک نوع دنباله                                        |
| `std::make_index_sequence`   | تولید خودکار اندیس‌ها                  | یک نوع `std::index_sequence`                         |
| `std::index_sequence_for`    | تولید اندیس‌ها بر اساس تعداد انواع     | یک نوع `std::index_sequence`                         |

این ابزارها مکمل یکدیگر هستند و هرکدام نقش مشخصی دارند. معمولاً در طراحی Templateهای عمومی، ابزارهای تولید خودکار دنباله را همراه با نوع دنبالهٔ متناظر استفاده می‌کنیم.

## نکات مهم دربارهٔ کارایی و پیاده‌سازی

### هزینهٔ زمان اجرا

خود `std::integer_sequence` یک سازوکار Type-Level است و برای نگه‌داری دنباله، نیازی به ایجاد یک آرایهٔ زمان اجرا ندارد.

بااین‌حال، نباید نتیجه گرفت که هر کدی که از آن استفاده می‌کند الزاماً بدون هزینه است. اگر دنباله برای تولید فراخوانی‌های متعدد تابع، ایجاد اشیا یا اجرای عملیات زمان اجرا به کار رود، هزینهٔ آن عملیات همچنان وجود خواهد داشت.

برای مثال، استفاده از `std::index_sequence` برای گسترش دسترسی به عناصر Tuple می‌تواند کدهای تکراری را حذف کند، اما عملیات روی عناصر Tuple همچنان تابع هزینهٔ واقعی خود هستند.

### هزینهٔ زمان کامپایل

تولید دنباله‌های بزرگ ممکن است بر زمان کامپایل، حافظهٔ مصرفی کامپایلر و عمق Template Instantiation تأثیر بگذارد.

استاندارد، الگوریتم داخلی مشخصی برای تولید دنباله‌ها الزام نمی‌کند. به همین دلیل، جزئیات پیاده‌سازی ممکن است میان کتابخانه‌های استاندارد مختلف متفاوت باشد.

برخی پیاده‌سازی‌ها از الگوریتم‌های تقسیم‌و‌غلبه استفاده می‌کنند تا عمق بازگشت Templateها کاهش یابد. این موضوع به‌ویژه هنگام تولید دنباله‌های بزرگ اهمیت پیدا می‌کند.

در نتیجه، اگر پروژه به تولید تعداد بسیار زیادی اندیس در زمان کامپایل نیاز داشته باشد، بهتر است زمان کامپایل و مصرف منابع در کامپایلرهای هدف نیز اندازه‌گیری شود.

### تفاوت با حلقه‌های معمولی

حلقهٔ معمولی برای پیمایش عناصر در زمان اجرا مناسب است؛ اما اندیس‌های موردنیاز برای پارامترهای Template باید در زمان کامپایل مشخص باشند.

برای مثال، اگر بخواهیم `std::get<i>` را برای اندیس‌های یک Tuple فراخوانی کنیم، مقدار `i` نمی‌تواند صرفاً یک متغیر معمولی در حلقه باشد. در چنین مواردی، دنبالهٔ اندیس‌ها راهکاری مناسب برای تولید مجموعه‌ای از فراخوانی‌های مشخص در زمان کامپایل است.

این تفاوت به معنی برتری مطلق `std::integer_sequence` بر حلقه‌ها نیست. انتخاب درست به نوع مسئله بستگی دارد:

- برای پیمایش داده‌های معمولی در زمان اجرا، حلقه‌ها اغلب ساده‌تر هستند.
- برای گسترش Templateها و فراخوانی‌هایی که به مقادیر ثابت در زمان کامپایل نیاز دارند، دنباله‌های اندیس مناسب‌تر هستند.

## محدودیت‌ها و اشتباهات رایج

### استفاده از مقدار زمان اجرا به‌عنوان پارامتر Template

یکی از اشتباهات رایج، تلاش برای تولید دنباله بر اساس مقداری است که فقط در زمان اجرا مشخص می‌شود.

```
#include <utility>

int main() {
    int count = 5;

    // This is invalid because count is not a constant expression.
    // using Sequence = std::make_index_sequence<count>;
}
```

مقدار پارامتر `N` در `std::make_index_sequence<N>` باید در زمان کامپایل مشخص باشد. بنابراین، اگر تعداد عناصر فقط هنگام اجرای برنامه معلوم شود، نمی‌توان مستقیماً از این ابزار برای تولید دنباله‌ای با آن تعداد استفاده کرد.

در چنین شرایطی، معمولاً باید از ساختارهای داده و الگوریتم‌های زمان اجرا استفاده کنیم.

### اشتباه گرفتن نوع دنباله با شیء حاوی داده

دنبالهٔ زیر:

```
using Sequence = std::integer_sequence<int, 10, 20, 30>;
```

به معنی ایجاد یک آرایه با سه عنصر نیست. این تعریف فقط یک نوع را معرفی می‌کند.

می‌توان از این نوع برای استخراج مقادیر در زمان کامپایل، تولید آرایه یا گسترش پارامترها استفاده کرد؛ اما اگر هدف نگه‌داری داده‌های قابل پیمایش در زمان اجرا باشد، باید از ساختار داده‌ای مناسب استفاده کرد.

### نادیده گرفتن دنبالهٔ خالی

در Templateهای عمومی، تعداد عناصر ممکن است صفر باشد. اگر پیاده‌سازی فقط برای دنباله‌های غیرخالی طراحی شده باشد، احتمال بروز خطای کامپایل وجود دارد.

برای مثال، در الگوهای مبتنی بر Initializer List، افزودن یک عنصر ثابت ابتدایی می‌تواند از خالی شدن فهرست جلوگیری کند. البته در هر پیاده‌سازی باید معتبر بودن نوع عناصر و عبارت‌های تولیدشده نیز بررسی شود.

### فرض کردن الگوریتم داخلی مشخص

استاندارد مشخص می‌کند که `std::make_integer_sequence` چه نوع نتیجه‌ای باید تولید کند، اما الگوریتم داخلی تولید دنباله را الزام نمی‌کند.

بنابراین، نباید فرض کنیم که تمام کامپایلرها از Recursion خطی، تقسیم‌و‌غلبه یا یک روش خاص استفاده می‌کنند. اگر جزئیات پیاده‌سازی بر عملکرد پروژه تأثیر دارند، باید آن‌ها را در محیط هدف بررسی کنیم.

## مقایسه با قابلیت‌های استانداردهای جدیدتر

### نقش `std::integer_sequence` در ++C17

در استاندارد ++C17، ابزارهایی مانند `std::apply` بسیاری از کاربردهای رایج `std::index_sequence` را ساده‌تر کرده‌اند.

به‌جای پیاده‌سازی دستی Tuple Unpacking، می‌توان از تابع استاندارد استفاده کرد؛ اما این موضوع باعث حذف `std::integer_sequence` از کتابخانهٔ استاندارد نشده است.

این ابزار همچنان در پیاده‌سازی Templateهای عمومی، تولید دنباله‌های اندیس و طراحی سازوکارهای Type-Level کاربرد دارد.

### تفاوت با Fold Expression

قابلیت Fold Expression در ++C17 امکان انجام عملیات روی Parameter Pack را با کدی کوتاه‌تر فراهم می‌کند.

برای مثال، محاسبهٔ مجموع یک Parameter Pack در ++C17 می‌تواند به این صورت نوشته شود:

```
template<class... Values>
constexpr auto sum(Values... values) {
    return (values + ...);
}
```

این قابلیت در بسیاری از موارد نیاز به الگوهای قدیمی‌تر گسترش پارامترها را کاهش می‌دهد؛ اما جایگزین کامل `std::integer_sequence` نیست.

Fold Expression برای ترکیب عناصر یک Parameter Pack مناسب است، درحالی‌که `std::integer_sequence` برای نمایش دنباله‌ای از مقادیر ثابت در سطح نوع طراحی شده است. این دو ابزار می‌توانند در یک پیاده‌سازی واحد نیز به کار روند.

## جمع‌بندی نهایی

قابلیت `std::integer_sequence` در ++C14 راهکاری استاندارد برای نمایش دنباله‌ای از مقادیر صحیح در زمان کامپایل است. این ابزار به‌ویژه زمانی مفید است که تعداد یا مقادیر اندیس‌ها باید در Templateها مشخص باشند.

مهم‌ترین نکاتی که باید به خاطر داشته باشیم عبارت‌اند از:

- `std::integer_sequence` یک نوع Type-Level است، نه یک Container زمان اجرا.
- عضو `value_type` نوع مقادیر دنباله را مشخص می‌کند و `size()` تعداد عناصر آن را برمی‌گرداند.
- ابزار `std::make_integer_sequence` دنباله‌ای از صفر تا \\(N-1\\) تولید می‌کند.
- نوع `std::index_sequence` برای نمایش اندیس‌ها طراحی شده است.
- ابزار `std::make_index_sequence` تعداد اندیس‌ها را مستقیماً از یک مقدار ثابت دریافت می‌کند.
- ابزار `std::index_sequence_for` تعداد اندیس‌ها را از تعداد انواع یک Parameter Pack استخراج می‌کند.
- یکی از کاربردهای مهم این ابزارها، گسترش عناصر `std::tuple` و تبدیل آن‌ها به آرگومان‌های مستقل تابع است.
- استفاده از این ابزارها باید متناسب با نیازهای زمان کامپایل باشد؛ برای پیمایش عادی داده‌ها در زمان اجرا، حلقه‌ها و Containerهای معمولی اغلب انتخاب ساده‌تری هستند.
- استاندارد، الگوریتم داخلی تولید دنباله را مشخص نمی‌کند؛ بنابراین نباید به یک روش پیاده‌سازی خاص وابسته شد.

در مجموع، `std::integer_sequence` یکی از ابزارهای پایه‌ای برای درک و پیاده‌سازی تکنیک‌های پیشرفتهٔ Template Metaprogramming است. یادگیری درست آن، درک نحوهٔ گسترش Parameter Packها، طراحی Templateهای عمومی و استفاده از قابلیت‌های استاندارد کتابخانهٔ ++C را ساده‌تر می‌کند.

---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>