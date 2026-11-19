<div align="center">

[🇺🇸 English](../../en/cpp20/lists_filters_views_ranges.md) | [🇮🇷 فارسی](./lists_filters_views_ranges.md)

</div>

---

# معرفی ابزار Lists,Filters,Views,Ranges در C++20

# کتابخانه Ranges در ++C20

## مقدمه

کتابخانه Ranges یکی از قابلیت‌های مهم استاندارد ++C20 است که شیوه کار با مجموعه‌ای از داده‌ها را در ++C ساده‌تر، خواناتر و انعطاف‌پذیرتر می‌کند.

پیش از معرفی Ranges، برای استفاده از بسیاری از الگوریتم‌های استاندارد باید محدوده داده‌ها را با دو Iterator مشخص می‌کردیم: یک Iterator برای ابتدای محدوده و یک Iterator برای انتهای آن. این روش همچنان معتبر است، اما در بسیاری از موارد باعث تکرار کد و افزایش احتمال اشتباه می‌شود.

کتابخانه Ranges این امکان را فراهم می‌کند که در بسیاری از الگوریتم‌ها، به‌جای ارسال دو Iterator، مستقیماً خود Range یا Container را ارسال کنیم. علاوه بر این، قابلیت‌هایی مانند Viewها، Range Adaptorها، Projectionها و Function Composition را در اختیارمان قرار می‌دهد.

برای درک بهتر این کتابخانه، ابتدا باید با مفهوم Range و ارتباط آن با Iterator آشنا شویم.

## مفهوم Range

در ++C، یک Range به محدوده‌ای از عناصر گفته می‌شود که بتوان آن‌ها را از طریق Iterator پیمایش کرد.

برای مثال، یک `std::vector`، یک `std::array`، یک `std::list` و حتی یک محدوده از عناصر یک آرایه معمولی می‌توانند به‌عنوان Range استفاده شوند.

```cpp
#include <iostream>
#include <vector>
#include <ranges>

int main()
{
    std::vector<int> numbers{10, 20, 30, 40, 50};

    for (int number : numbers)
    {
        std::cout << number << ' ';
    }
}
```

در مثال بالا، متغیر `numbers` یک Range است. حلقه Range-based for از ابتدا تا انتهای آن حرکت می‌کند و عناصر را در اختیار ما قرار می‌دهد.

نکته مهم این است که Range الزاماً یک Container نیست. یک Range می‌تواند فقط یک محدوده قابل پیمایش باشد که داده‌ها را از منبع دیگری دریافت می‌کند یا حتی عناصر را هنگام پیمایش تولید می‌کند.

برای مثال، یک View نیز می‌تواند Range باشد، بدون آنکه خودش مالک داده‌های اصلی باشد.

### ارتباط Range با Iterator

Iteratorها ابزار پیمایش عناصر هستند. هر Iterator بسته به قابلیت‌هایی که ارائه می‌دهد، می‌تواند عملیات مشخصی را پشتیبانی کند.

در استاندارد ++C20، دسته‌بندی Iteratorها شامل موارد زیر است:

- **Input Iterator:** امکان خواندن عناصر را در یک مسیر پیمایش فراهم می‌کند.
- **Output Iterator:** امکان نوشتن عناصر را فراهم می‌کند.
- **Forward Iterator:** امکان پیمایش رو به جلو و چندباره را فراهم می‌کند.
- **Bidirectional Iterator:** امکان حرکت به جلو و عقب را فراهم می‌کند.
- **Random Access Iterator:** امکان دسترسی تصادفی و پرش به موقعیت‌های مختلف را فراهم می‌کند.
- **Contiguous Iterator:** علاوه بر قابلیت‌های دسترسی تصادفی، تضمین می‌کند عناصر در حافظه به‌صورت پیوسته قرار گرفته‌اند.

این دسته‌بندی‌ها در طراحی Rangeها نیز اهمیت دارند. یک Range می‌تواند قابلیت‌های پیمایشی مختلفی داشته باشد؛ برای مثال، یک Range ممکن است فقط قابلیت پیمایش رو به جلو داشته باشد، در حالی که Range دیگری امکان دسترسی تصادفی به عناصر را نیز فراهم کند.

### مفهوم Sentinel

در طراحی قدیمی‌تر الگوریتم‌ها، معمولاً انتهای محدوده با یک Iterator مشخص می‌شد. در Ranges مفهوم Sentinel نیز مطرح است.

Sentinel شیئی است که می‌تواند پایان پیمایش را مشخص کند. برخلاف Iterator، لازم نیست Sentinel تمام قابلیت‌های Iterator را داشته باشد.

در نتیجه، یک Range می‌تواند از دو نوع متفاوت برای ابتدا و انتهای محدوده استفاده کند:

- Iterator برای شروع پیمایش
- Sentinel برای تشخیص پایان پیمایش

این انعطاف‌پذیری به‌خصوص برای Rangeهایی مفید است که طول آن‌ها از ابتدا مشخص نیست یا به‌صورت تدریجی تولید می‌شوند.

## الگوریتم‌های Ranges

کتابخانه الگوریتم‌های استاندارد ++C پیش از ++C20 نیز وجود داشت. این الگوریتم‌ها معمولاً با دریافت دو Iterator، محدوده موردنظر را مشخص می‌کردند.

برای مثال، الگوریتم `std::all_of` بررسی می‌کند که آیا تمام عناصر یک محدوده شرط مشخصی را برقرار می‌کنند یا خیر.

### روش قدیمی استفاده از الگوریتم‌ها

```cpp
#include <algorithm>
#include <iostream>

int main()
{
    int collection[]{2, 6, 8, 40, 64, 70};

    bool result = std::all_of(
        std::begin(collection),
        std::end(collection),
        [](int value)
        {
            return value % 2 == 0;
        }
    );

    std::cout << std::boolalpha << result << '\n';
}
```

در این روش باید Iterator ابتدا و انتهای محدوده را به الگوریتم ارسال کنیم.

### استفاده از الگوریتم‌های Ranges

در ++C20 نسخه‌های جدیدی از بسیاری از الگوریتم‌های استاندارد در فضای نام `std::ranges` ارائه شدند.

این نسخه‌ها در بسیاری از موارد مستقیماً یک Range را دریافت می‌کنند.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> numbers{11, 2, 6, 4, 8, 3, 17, 9};

    auto odd = [](int value)
    {
        return value % 2 != 0;
    };

    bool result = std::ranges::all_of(numbers, odd);

    if (result)
    {
        std::cout << "All elements are odd\n";
    }
    else
    {
        std::cout << "Not all elements are odd\n";
    }
}
```

در این مثال، دیگر نیازی به ارسال دو Iterator نیست و خود Container به الگوریتم داده شده است.

البته نسخه‌های Ranges بسیاری از الگوریتم‌ها همچنان امکان دریافت Iterator و Sentinel را نیز دارند. بنابراین، استفاده از Range تنها شکل قابل‌قبول فراخوانی آن‌ها نیست.

### الگوریتم for_each

الگوریتم `std::ranges::for_each` یک تابع را برای هر عنصر از محدوده اجرا می‌کند.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    std::ranges::for_each(numbers, [](int& value)
    {
        value *= 2;
    });

    for (int value : numbers)
    {
        std::cout << value << ' ';
    }
}
```

خروجی:

```text
2 4 6 8 10
```

از آنجا که Lambda در این مثال پارامتر را به‌صورت Reference دریافت می‌کند، تغییرات روی عناصر اصلی Container اعمال می‌شوند.

اگر پارامتر را به‌صورت مقدار دریافت می‌کردیم، تغییرات فقط روی کپی عنصر اعمال می‌شدند.

### الگوریتم sort

الگوریتم `std::ranges::sort` عناصر یک محدوده قابل مرتب‌سازی را مرتب می‌کند.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> numbers{8, 3, 9, 1, 5, 2};

    std::ranges::sort(numbers);

    for (int value : numbers)
    {
        std::cout << value << ' ';
    }
}
```

خروجی:

```text
1 2 3 5 8 9
```

به‌صورت پیش‌فرض، مرتب‌سازی با ترتیب صعودی انجام می‌شود.

برای مرتب‌سازی نزولی می‌توان Comparator مناسب ارسال کرد:

```cpp
std::ranges::sort(numbers, std::greater<>{});
```

توجه داشته باشید که `std::ranges::sort` برای هر Rangeای قابل استفاده نیست. این الگوریتم به محدوده‌ای نیاز دارد که قابلیت‌های لازم برای مرتب‌سازی، از جمله دسترسی تصادفی و امکان جابه‌جایی عناصر را داشته باشد.

### الگوریتم find

الگوریتم `std::ranges::find` یک عنصر مشخص را در Range جست‌وجو می‌کند.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> numbers{11, 2, 6, 4, 8, 3, 17, 9};

    auto position = std::ranges::find(numbers, 17);

    if (position != std::ranges::end(numbers))
    {
        std::cout << "Found: " << *position << '\n';
    }
    else
    {
        std::cout << "Not found\n";
    }
}
```

اگر عنصر پیدا شود، Iterator آن برگردانده می‌شود. در غیر این صورت، مقدار بازگشتی با انتهای Range برابر خواهد بود.

### الگوریتم copy و خروجی جریان داده

الگوریتم‌های Ranges فقط برای Containerها کاربرد ندارند. می‌توان از آن‌ها برای انتقال عناصر به مقصدهایی مانند Output Iterator نیز استفاده کرد.

برای مثال:

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    std::ranges::copy(
        numbers,
        std::ostream_iterator<int>(std::cout, " ")
    );
}
```

در این مثال، عناصر Vector مستقیماً به خروجی استاندارد ارسال می‌شوند.

### مزایای الگوریتم‌های Ranges

مهم‌ترین مزایای الگوریتم‌های Ranges عبارت‌اند از:

- کاهش نیاز به ارسال Iterator ابتدا و انتها در فراخوانی‌های متداول
- خوانایی بهتر کد
- استفاده از Constraintها برای محدود کردن نوع ورودی‌های معتبر
- پشتیبانی از Projectionها
- هماهنگی بهتر با Viewها و Range Adaptorها
- کاهش احتمال اشتباه در تعیین محدوده

## الگوریتم‌های مبتنی بر Iterator و Constraintها

الگوریتم‌های قدیمی استاندارد ++C عموماً بر اساس نیازمندی‌های Iterator طراحی شده‌اند. برای مثال، یک الگوریتم ممکن است فقط به Input Iterator نیاز داشته باشد، در حالی که الگوریتم دیگری به Random Access Iterator احتیاج داشته باشد.

نسخه‌های Ranges علاوه بر پشتیبانی از Rangeها، از Constraintهای زبان ++C20 نیز بهره می‌برند.

Constraintها مشخص می‌کنند که یک نوع داده باید چه قابلیت‌هایی داشته باشد تا بتوان از آن در یک الگوریتم استفاده کرد.

برای نمونه، الگوریتم `std::ranges::sort` به Rangeای نیاز دارد که شرایط لازم برای مرتب‌سازی را برآورده کند. در نتیجه، اگر یک نوع داده این شرایط را نداشته باشد، کامپایلر می‌تواند فراخوانی نامعتبر را تشخیص دهد.

### تفاوت Iterator Pair و Range Overload

بسیاری از الگوریتم‌های Ranges دو شکل فراخوانی دارند:

```cpp
std::ranges::sort(numbers);
```

و:

```cpp
std::ranges::sort(numbers.begin(), numbers.end());
```

شکل اول یک Range دریافت می‌کند و شکل دوم یک Iterator و Sentinel دریافت می‌کند.

بنابراین، Ranges جایگزین کامل Iteratorها نیست؛ بلکه لایه‌ای انعطاف‌پذیرتر برای کار با محدوده‌ها فراهم می‌کند.

### چرا معمولاً بهتر است از std::ranges استفاده کنیم؟

در کدهای جدید، در صورت سازگاری نیازمندی‌های الگوریتم با نوع داده، استفاده از نسخه‌های `std::ranges` معمولاً انتخاب مناسبی است.

این نسخه‌ها علاوه بر پشتیبانی از Rangeها، امکان استفاده از Projectionها و Constraintها را فراهم می‌کنند.

البته الگوریتم‌های قدیمی همچنان معتبرند و در پروژه‌های موجود، کتابخانه‌های قدیمی یا شرایط خاص ممکن است استفاده از آن‌ها منطقی باشد.

## Projectionها

یکی از قابلیت‌های مهم الگوریتم‌های Ranges، پشتیبانی از Projection است.

Projection تابعی است که قبل از مقایسه یا بررسی یک عنصر، بخشی از آن را استخراج می‌کند یا نمایش دیگری از آن ارائه می‌دهد.

به بیان ساده، به‌جای آنکه الگوریتم مستقیماً با خود شیء کار کند، می‌توانیم مشخص کنیم کدام ویژگی یا بخش از شیء باید مبنای عملیات قرار بگیرد.

فرض کنید مجموعه‌ای از اشیای Point داریم و می‌خواهیم آن‌ها را بر اساس یکی از مختصاتشان مرتب کنیم.

### تعریف کلاس Point

```cpp
#include <cmath>
#include <iostream>

class Point
{
public:
    Point() = default;

    Point(double x, double y)
        : m_x(x), m_y(y)
    {
    }

    double length() const
    {
        return std::sqrt(m_x * m_x + m_y * m_y);
    }

    double m_x{};
    double m_y{};
};
```

### مرتب‌سازی بدون Projection

اگر عملگر مقایسه برای کلاس Point تعریف شده باشد، می‌توان مستقیماً از آن برای مرتب‌سازی استفاده کرد.

```cpp
std::ranges::sort(points);
```

در این حالت، الگوریتم از Comparator پیش‌فرض استفاده می‌کند و مقایسه اشیا بر اساس عملگر مقایسه تعریف‌شده برای آن‌ها انجام می‌شود.

### استفاده از Lambda به‌عنوان Projection

فرض کنیم می‌خواهیم نقاط را بر اساس مختصات `m_x` مرتب کنیم.

```cpp
std::ranges::sort(
    points,
    std::less<>{},
    [](const Point& point)
    {
        return point.m_x;
    }
);
```

در این فراخوانی سه بخش وجود دارد:

- Range موردنظر
- Comparator برای مقایسه مقادیر
- Projection برای استخراج مقدار مورد مقایسه از هر عنصر

الگوریتم ابتدا Projection را روی عناصر اعمال می‌کند و سپس مقادیر حاصل را با Comparator مقایسه می‌کند.

در این مثال، به‌جای مقایسه دو شیء Point، مقدار `m_x` آن‌ها با یکدیگر مقایسه می‌شود.

### استفاده از Pointer to Member به‌عنوان Projection

اگر Projection موردنظر صرفاً دسترسی به یک عضو عمومی باشد، می‌توان به‌جای Lambda از Pointer to Member استفاده کرد.

```cpp
std::ranges::sort(
    points,
    std::less<>{},
    &Point::m_y
);
```

این کد نقاط را بر اساس مقدار `m_y` مرتب می‌کند.

این روش کوتاه و خواناست و زمانی که فقط قصد استخراج یک عضو عمومی را داریم، انتخاب مناسبی است.

### Projection در الگوریتم for_each

Projection فقط برای مرتب‌سازی کاربرد ندارد. الگوریتم‌هایی مانند `std::ranges::for_each` نیز از آن پشتیبانی می‌کنند.

```cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <utility>
#include <vector>

int main()
{
    using Pair = std::pair<int, std::string>;

    std::vector<Pair> pairs{
        {1, "one"},
        {2, "two"},
        {3, "three"}
    };

    auto print = [](const auto& value)
    {
        std::cout << value << ' ';
    };

    std::ranges::for_each(
        pairs,
        print,
        &Pair::first
    );

    std::cout << '\n';

    std::ranges::for_each(
        pairs,
        print,
        &Pair::second
    );
}
```

در فراخوانی اول، Projection عضو `first` را انتخاب می‌کند و در فراخوانی دوم عضو `second` را.

به این ترتیب، تابع `print` به‌جای دریافت کل Pair، فقط مقدار استخراج‌شده را دریافت می‌کند.

### تفاوت Projection و Transform

Projection و Transform در ظاهر شبیه به یکدیگرند، اما نقش یکسانی ندارند.

Projection بخشی از قرارداد الگوریتم است و مشخص می‌کند الگوریتم هنگام مقایسه یا بررسی عناصر، چه مقداری را در نظر بگیرد.

در مقابل، `transform_view` یک View جدید تولید می‌کند که عناصر آن از اعمال یک تابع روی عناصر Range اصلی به دست می‌آیند.

Projection معمولاً برای تعیین نحوه تفسیر داده در یک الگوریتم استفاده می‌شود؛ اما Transform یک مرحله از پردازش Range را تشکیل می‌دهد.

## Viewها و Range Adaptorها

یکی از مهم‌ترین بخش‌های کتابخانه Ranges، مفهوم View است.

یک View نوعی Range است که معمولاً مالک داده‌های اصلی نیست و به‌جای نگهداری یک مجموعه داده مستقل، نمایی از داده‌های موجود را در اختیار ما قرار می‌دهد.

می‌توان View را مانند یک پنجره در نظر گرفت که روی داده‌های اصلی قرار می‌گیرد و فقط بخش یا شکل خاصی از آن‌ها را نمایش می‌دهد.

ویژگی‌های مهم Viewها:

- معمولاً مالک داده‌های اصلی نیستند.
- در بسیاری از موارد به‌صورت Lazy عمل می‌کنند.
- معمولاً ساخت و کپی کردن آن‌ها کم‌هزینه است.
- می‌توان آن‌ها را به‌عنوان پارامتر به توابع ارسال کرد.
- امکان ترکیب چند View برای ساخت یک جریان پردازش را فراهم می‌کنند.

### تفاوت View و Container

Container معمولاً مسئول نگهداری و مالکیت عناصر است.

برای مثال، `std::vector` حافظه لازم برای عناصر خود را مدیریت می‌کند.

اما View معمولاً فقط نحوه دسترسی یا مشاهده عناصر را تعریف می‌کند و داده‌های اصلی را در خود ذخیره نمی‌کند.

```cpp
std::vector<int> numbers{1, 2, 3, 4, 5};

auto view = numbers | std::views::filter(
    [](int value)
    {
        return value % 2 == 0;
    }
);
```

در این مثال، یک View از عناصر زوج ساخته شده است. عناصر اصلی همچنان در Vector قرار دارند و View فقط عناصر مطابق شرط را نمایش می‌دهد.

### نکته مهم درباره مالکیت

اینکه View معمولاً Non-owning است، به این معنی است که طول عمر داده اصلی اهمیت زیادی دارد.

```cpp
auto create_view()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    return numbers | std::views::filter(
        [](int value)
        {
            return value % 2 == 0;
        }
    );
}
```

این تابع نمونه‌ای از طراحی خطرناک است. View حاصل به داده‌هایی وابسته است که با پایان تابع از بین می‌روند. استفاده از آن پس از بازگشت تابع می‌تواند به رفتار تعریف‌نشده منجر شود.

بنابراین، هنگام نگهداری View باید مطمئن باشیم منبع داده تا پایان استفاده از View معتبر باقی می‌ماند.

### Lazy Evaluation

یکی از ویژگی‌های اساسی بسیاری از Viewها، Lazy Evaluation است.

یعنی هنگام ساخت View، لزوماً تمام عملیات روی عناصر اجرا نمی‌شوند. عملیات معمولاً زمانی انجام می‌شود که عناصر View پیمایش یا خوانده شوند.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    auto view = numbers | std::views::transform(
        [](int value)
        {
            std::cout << "Transforming " << value << '\n';
            return value * 10;
        }
    );

    std::cout << "View created\n";

    for (int value : view)
    {
        std::cout << value << '\n';
    }
}
```

ساخت View به‌تنهایی باعث اجرای تابع Transform برای تمام عناصر نمی‌شود. تابع هنگام پیمایش View اجرا می‌شود.

این ویژگی می‌تواند از ایجاد مجموعه‌های موقت و محاسبات غیرضروری جلوگیری کند.

البته Lazy بودن به معنای Cache شدن نتیجه نیست. در بسیاری از Viewها، اگر یک عنصر چند بار خوانده شود، تابع تبدیل ممکن است چند بار اجرا شود.

## انواع View و Range Adaptorها

کتابخانه استاندارد ++C20 مجموعه‌ای از Viewها و Range Adaptorها را فراهم می‌کند.

Range Adaptor ابزاری است که یک Range را دریافت می‌کند و View جدیدی بر اساس آن می‌سازد.

در ادامه مهم‌ترین موارد را بررسی می‌کنیم.

### Filter View

`std::views::filter` فقط عناصری را نمایش می‌دهد که شرط مشخص‌شده را برقرار کنند.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5, 6, 7, 8, 9};

    auto evens = numbers | std::views::filter(
        [](int value)
        {
            return value % 2 == 0;
        }
    );

    for (int value : evens)
    {
        std::cout << value << ' ';
    }
}
```

خروجی:

```text
2 4 6 8
```

در این مثال، View فقط اعداد زوج را نمایش می‌دهد و هیچ عنصر جدیدی در Vector ساخته نمی‌شود.

می‌توان شرط را در یک متغیر جداگانه نیز تعریف کرد:

```cpp
auto is_even = [](int value)
{
    return value % 2 == 0;
};

auto evens = numbers | std::views::filter(is_even);
```

### Filter View و زمان اجرای Predicate

تابع Predicate هنگام پیمایش عناصر برای تشخیص معتبر بودن آن‌ها فراخوانی می‌شود.

بنابراین، ساخت View به معنی فیلتر شدن فوری تمام عناصر نیست.

همچنین Predicate مورد استفاده در `filter_view` باید با رفتار مورد انتظار این View سازگار باشد. به‌خصوص نباید در هر بار فراخوانی، نتیجه‌ای غیرقابل پیش‌بینی و وابسته به تغییر وضعیت خارجی تولید کند.

تغییر دادن عناصر منبع به‌گونه‌ای که نتیجه Predicate را عوض کند نیز می‌تواند باعث نقض پیش‌شرط‌های استفاده از Filter View شود.

### Transform View

`std::views::transform` روی هر عنصر یک تابع اجرا می‌کند و نتیجه آن تابع را به‌عنوان عنصر View نمایش می‌دهد.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    auto transformed = numbers | std::views::transform(
        [](int value)
        {
            return value * 10;
        }
    );

    for (int value : transformed)
    {
        std::cout << value << ' ';
    }
}
```

خروجی:

```text
10 20 30 40 50
```

در این مثال، داده‌های اصلی تغییر نمی‌کنند. View مقادیر تبدیل‌شده را هنگام پیمایش در اختیار ما قرار می‌دهد.

البته اگر تابع Transform یک Reference برگرداند، امکان تغییر داده اصلی از طریق View نیز وجود دارد. بنابراین، رفتار آن به نوع مقدار بازگشتی تابع وابسته است.

### Take View

`std::views::take` حداکثر تعداد مشخصی از عناصر ابتدایی Range را نمایش می‌دهد.

```cpp
std::vector<int> numbers{1, 2, 3, 4, 5, 6, 7};

auto first_five = numbers | std::views::take(5);
```

این View فقط پنج عنصر ابتدایی را در اختیار مصرف‌کننده قرار می‌دهد.

اگر Range کمتر از تعداد درخواستی عنصر داشته باشد، تمام عناصر موجود نمایش داده می‌شوند.

### Take While View

`std::views::take_while` عناصر را از ابتدای Range نمایش می‌دهد تا زمانی که Predicate برقرار باشد.

```cpp
std::vector<int> numbers{1, 3, 5, 7, 8, 9, 11};

auto result = numbers | std::views::take_while(
    [](int value)
    {
        return value % 2 != 0;
    }
);
```

خروجی:

```text
1 3 5 7
```

به محض اینکه عنصر 8 با شرط سازگار نباشد، پیمایش این View متوقف می‌شود. عناصر بعد از آن، حتی اگر دوباره شرط را برقرار کنند، نمایش داده نمی‌شوند.

این رفتار با Filter متفاوت است. Filter تمام عناصر را بررسی می‌کند و هر عنصری را که شرط را برقرار کند نمایش می‌دهد؛ اما Take While فقط یک پیشوند از Range را در نظر می‌گیرد.

### Drop View

`std::views::drop` تعداد مشخصی از عناصر ابتدایی Range را نادیده می‌گیرد و بقیه را نمایش می‌دهد.

```cpp
std::vector<int> numbers{1, 2, 3, 4, 5, 6, 7};

auto result = numbers | std::views::drop(3);
```

خروجی:

```text
4 5 6 7
```

### Drop While View

`std::views::drop_while` عناصر ابتدایی را تا زمانی که Predicate برقرار باشد نادیده می‌گیرد.

```cpp
std::vector<int> numbers{1, 3, 5, 8, 9, 11};

auto result = numbers | std::views::drop_while(
    [](int value)
    {
        return value % 2 != 0;
    }
);
```

خروجی:

```text
8 9 11
```

تفاوت آن با Filter در این است که Drop While فقط عناصر ابتدایی را حذف می‌کند و پس از اولین عنصری که شرط را نقض کند، بقیه عناصر را بدون بررسی مجدد شرط نمایش می‌دهد.

### Keys View و Values View

برای Rangeهایی مانند Map یا مجموعه‌ای از Pairها، گاهی فقط به کلیدها یا فقط به مقادیر نیاز داریم.

`std::views::keys` نمایی از کلیدهای Range ایجاد می‌کند.

`std::views::values` نیز نمایی از مقادیر را ارائه می‌دهد.

```cpp
#include <iostream>
#include <map>
#include <ranges>
#include <string>

int main()
{
    std::map<int, std::string> students{
        {1, "Ali"},
        {2, "Sara"},
        {3, "Reza"}
    };

    auto keys = students | std::views::keys;
    auto values = students | std::views::values;

    for (const auto& key : keys)
    {
        std::cout << key << ' ';
    }

    std::cout << '\n';

    for (const auto& value : values)
    {
        std::cout << value << ' ';
    }
}
```

این Viewها کپی جداگانه‌ای از کلیدها یا مقادیر تولید نمی‌کنند؛ بلکه دسترسی به بخش موردنظر از عناصر اصلی را فراهم می‌کنند.

## ترکیب Viewها و عملگر Pipe

یکی از جذاب‌ترین قابلیت‌های کتابخانه Ranges، امکان ترکیب چند View با یکدیگر است.

فرض کنید مجموعه‌ای از اعداد داریم و می‌خواهیم:

- فقط اعداد زوج را انتخاب کنیم.
- سپس مربع هر عدد را محاسبه کنیم.
- در نهایت فقط پنج نتیجه اول را نمایش دهیم.

می‌توان این مراحل را به‌صورت جداگانه تعریف کرد؛ اما عملگر Pipe امکان نوشتن آن‌ها به‌صورت یک زنجیره خوانا را فراهم می‌کند.

### ترکیب Viewها

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector<int> numbers{
        1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12
    };

    auto result = numbers
        | std::views::filter(
            [](int value)
            {
                return value % 2 == 0;
            }
        )
        | std::views::transform(
            [](int value)
            {
                return value * value;
            }
        )
        | std::views::take(5);

    for (int value : result)
    {
        std::cout << value << ' ';
    }
}
```

خروجی:

```text
4 16 36 64 100
```

در این زنجیره:

1. Filter فقط اعداد زوج را انتخاب می‌کند.
2. Transform مربع هر عدد را محاسبه می‌کند.
3. Take فقط پنج نتیجه اول را نمایش می‌دهد.

نکته مهم این است که در این مثال، مراحل پردازش به‌صورت Lazy انجام می‌شوند. لازم نیست ابتدا یک Vector جدید از اعداد زوج بسازیم، سپس Vector دیگری از مربع آن‌ها ایجاد کنیم و در نهایت پنج عنصر اول را جدا کنیم.

### مفهوم عملگر Pipe

عملگر `|` در کتابخانه Ranges برای اتصال یک Range به یک Range Adaptor استفاده می‌شود.

به‌صورت مفهومی:

```cpp
range | adaptor
```

یعنی Range را به Adaptor می‌دهیم تا View جدیدی ساخته شود.

ترکیب چند Adaptor نیز به همین شکل انجام می‌شود:

```cpp
range
    | adaptor1
    | adaptor2
    | adaptor3;
```

این ساختار از نظر خوانایی شبیه یک جریان پردازش داده است؛ هر مرحله خروجی مرحله قبلی را دریافت می‌کند.

### ترکیب Adaptorها بدون Range

گاهی می‌توان خود Adaptorها را از قبل با یکدیگر ترکیب کرد و سپس آن ترکیب را روی یک Range اعمال نمود.

```cpp
auto pipeline =
    std::views::filter(
        [](int value)
        {
            return value % 2 == 0;
        }
    )
    | std::views::transform(
        [](int value)
        {
            return value * value;
        }
    );

std::vector<int> numbers{1, 2, 3, 4, 5, 6};

auto result = numbers | pipeline;
```

در این مثال، ابتدا یک زنجیره از Adaptorها ساخته شده است و بعداً روی Range اعمال می‌شود.

این قابلیت برای ساخت Pipelineهای قابل استفاده مجدد مفید است.

### تفاوت Composition و اجرای فوری

ترکیب Viewها لزوماً به معنی اجرای فوری تمام مراحل نیست.

در بسیاری از موارد، تنها ساختار پردازش ساخته می‌شود و محاسبات هنگام پیمایش انجام می‌شوند.

به همین دلیل، یک View می‌تواند چندین بار پیمایش شود؛ اما باید توجه داشت که نتیجه هر پیمایش ممکن است تابع وضعیت منبع داده و رفتار توابع استفاده‌شده باشد.

اگر تابع Transform دارای Side Effect باشد، پیمایش مجدد View ممکن است آن Side Effect را دوباره اجرا کند.

## Range Factoryها

تا اینجا Viewهایی را بررسی کردیم که از یک Range موجود ساخته می‌شدند.

اما در ++C20 ابزارهایی وجود دارند که می‌توانند Rangeهایی را بدون نیاز به یک Container اولیه تولید کنند.

به این ابزارها Range Factory گفته می‌شود.

یکی از مهم‌ترین نمونه‌های آن `std::views::iota` است.

### Iota View

`std::views::iota` یک دنباله از مقادیر را تولید می‌کند.

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto numbers = std::views::iota(1, 10);

    for (int value : numbers)
    {
        std::cout << value << ' ';
    }
}
```

خروجی:

```text
1 2 3 4 5 6 7 8 9
```

مقدار شروع در محدوده قرار می‌گیرد، اما مقدار پایان در آن قرار نمی‌گیرد.

بنابراین، `iota(1, 10)` اعداد 1 تا 9 را تولید می‌کند.

### دنباله بدون انتهای مشخص

اگر فقط مقدار شروع را به `iota` بدهیم، یک دنباله نامحدود تولید می‌شود.

```cpp
auto numbers = std::views::iota(1);
```

این Range به‌صورت نامحدود ادامه پیدا می‌کند.

برای استفاده از آن باید یک شرط پایان یا محدودیت در نظر بگیریم.

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto numbers = std::views::iota(1)
        | std::views::take(10);

    for (int value : numbers)
    {
        std::cout << value << ' ';
    }
}
```

خروجی:

```text
1 2 3 4 5 6 7 8 9 10
```

### استفاده از Iota در Pipeline

از آنجا که Iota یک Range است، می‌توان آن را با سایر Adaptorها ترکیب کرد.

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto result = std::views::iota(1, 20)
        | std::views::filter(
            [](int value)
            {
                return value % 3 == 0;
            }
        )
        | std::views::transform(
            [](int value)
            {
                return value * value;
            }
        );

    for (int value : result)
    {
        std::cout << value << ' ';
    }
}
```

در این مثال، ابتدا اعداد 1 تا 19 تولید می‌شوند، سپس مضرب‌های 3 انتخاب شده و مربع آن‌ها نمایش داده می‌شود.

### کاربردهای Iota

`iota` در موارد مختلفی کاربرد دارد:

- تولید دنباله اعداد
- ساخت محدوده‌های عددی برای حلقه‌ها
- تولید داده‌های آزمایشی
- ساخت Pipelineهای پردازشی
- جایگزینی بعضی حلقه‌های شمارنده‌ای
- ترکیب با Filter و Transform

توجه داشته باشید که استفاده از Iota برای تولید یک دنباله نامحدود نیازمند مدیریت درست پایان پیمایش است. اگر چنین Rangeای بدون محدودیت در یک حلقه پیمایش شود، حلقه پایان نخواهد یافت.

## نکات تکمیلی و محدودیت‌های Ranges

کتابخانه Ranges قابلیت‌های قدرتمندی در اختیار ما قرار می‌دهد، اما استفاده صحیح از آن نیازمند شناخت چند نکته مهم است.

### همه Rangeها قابل مرتب‌سازی نیستند

هر Range الزاماً قابلیت مرتب‌سازی ندارد.

برای مثال، یک View که فقط قابلیت پیمایش رو به جلو دارد، لزوماً شرایط موردنیاز `std::ranges::sort` را برآورده نمی‌کند.

همچنین بعضی Viewها ممکن است Reference یا Proxy Referenceهایی ارائه دهند که با نیازمندی‌های یک الگوریتم سازگار نباشند.

بنابراین، قابل پیمایش بودن یک Range به معنی قابل استفاده بودن آن در تمام الگوریتم‌ها نیست.

### Viewها لزوماً مالک داده نیستند

View معمولاً به داده اصلی وابسته است. اگر داده اصلی از بین برود، View نیز ممکن است نامعتبر شود.

در طراحی APIهایی که View دریافت می‌کنند یا برمی‌گردانند، باید طول عمر داده‌ها به‌دقت مدیریت شود.

### تغییر داده اصلی می‌تواند نتیجه View را تغییر دهد

چون View معمولاً نمایی از داده اصلی است، تغییر عناصر Container می‌تواند بر نتیجه پیمایش View تأثیر بگذارد.

برای مثال، اگر View از نوع Filter باشد و مقدار یک عنصر تغییر کند، ممکن است آن عنصر دیگر شرط Predicate را برقرار نکند.

در چنین شرایطی باید الزامات اعتبار Iteratorها و قواعد مربوط به هر نوع View را در نظر گرفت.

### Lazy Evaluation همیشه بهتر نیست

Lazy Evaluation می‌تواند از ساخت مجموعه‌های موقت جلوگیری کند؛ اما همیشه سریع‌ترین انتخاب نیست.

در برخی شرایط، محاسبه مجدد یک Transform هنگام پیمایش‌های متعدد می‌تواند هزینه‌بر باشد.

اگر نتیجه محاسبات سنگین قرار است چندین بار استفاده شود، گاهی Materialize کردن نتیجه در یک Container انتخاب مناسب‌تری است.

### تفاوت View و Materialized Result

فرض کنید می‌خواهیم مربع اعداد یک Vector را محاسبه کنیم.

با Transform View:

```cpp
auto squares = numbers
    | std::views::transform(
        [](int value)
        {
            return value * value;
        }
    );
```

در این حالت، نتیجه به‌صورت یک View باقی می‌ماند و هنگام پیمایش محاسبه می‌شود.

اما اگر بخواهیم نتایج را در یک Vector مستقل ذخیره کنیم:

```cpp
std::vector<int> squares;

std::ranges::transform(
    numbers,
    std::back_inserter(squares),
    [](int value)
    {
        return value * value;
    }
);
```

در این حالت، نتیجه محاسبه شده و در حافظه Vector جدید قرار می‌گیرد.

انتخاب بین این دو روش به نیاز برنامه بستگی دارد:

- اگر فقط به مشاهده یا پردازش تدریجی عناصر نیاز داریم، View مناسب است.
- اگر به داده مستقل، قابل ذخیره‌سازی و قابل استفاده مجدد نیاز داریم، Materialize کردن نتیجه در Container مناسب‌تر است.

### خطاهای رایج در استفاده از Ranges

- استفاده از View پس از نابودی داده اصلی
- تصور اینکه ساخت View حتماً محاسبات را اجرا می‌کند
- تصور اینکه تمام Viewها قابلیت پیمایش یکسانی دارند
- استفاده از الگوریتمی مانند Sort روی Range نامناسب
- استفاده از Predicate دارای Side Effect یا رفتار ناپایدار در Filter
- فرض اینکه Transform View نتیجه را Cache می‌کند
- تصور اینکه Filter و Take While رفتار یکسانی دارند
- فراموش کردن اینکه انتهای محدوده در Iota جزو خروجی نیست

## جمع‌بندی

کتابخانه Ranges در ++C20 شیوه کار با مجموعه داده‌ها را از طریق Rangeها، الگوریتم‌های جدید، Viewها و Range Adaptorها توسعه می‌دهد.

مهم‌ترین مفاهیم این کتابخانه عبارت‌اند از:

- **Range:** محدوده‌ای از عناصر قابل پیمایش
- **Range Algorithms:** نسخه‌هایی از الگوریتم‌های استاندارد که با Rangeها و Constraintها کار می‌کنند
- **Projection:** تعیین بخشی از هر عنصر که الگوریتم باید بر اساس آن عملیات را انجام دهد
- **View:** نمایی از داده‌ها که معمولاً مالک داده اصلی نیست
- **Range Adaptor:** ابزاری برای ساخت View از یک Range
- **Filter:** انتخاب عناصر مطابق یک شرط
- **Transform:** تبدیل عناصر هنگام پیمایش
- **Take و Drop:** محدود کردن یا رد کردن بخشی از عناصر
- **Pipe Operator:** ترکیب چند مرحله پردازش
- **Iota:** تولید دنباله‌ای از مقادیر بدون نیاز به Container اولیه

در نهایت، Ranges تنها راه کوتاه‌تری برای نوشتن حلقه‌ها نیست. این کتابخانه رویکردی متفاوت برای طراحی جریان‌های پردازش داده ارائه می‌دهد؛ رویکردی که در آن می‌توان مراحل مختلف پردازش را به‌صورت مستقل تعریف و سپس با یکدیگر ترکیب کرد.

در طراحی کدهای مدرن ++C، شناخت تفاوت Container، Iterator، Range و View اهمیت زیادی دارد. با درک این مفاهیم می‌توان از قابلیت‌های Ranges به‌شکل مؤثر استفاده کرد و کدهایی خواناتر، قابل ترکیب‌تر و در بسیاری از موارد کم‌هزینه‌تر نوشت.


# راهنمای جامع کتابخانه Ranges در ++C20

کتابخانه Ranges در استاندارد ++C20 یکی از بنیادین‌ترین تحولاتی است که از زمان معرفی کتابخانه استاندارد قالب‌ها (STL) در ++C رخ داده است. این کتابخانه با ترکیب قابلیت‌های Concepts و ویژگی‌های مدرن زبان، مدل سنتی کار با کانتینرها، تکرارگرها (Iterators) و الگوریتم‌ها را بازطراحی کرده است تا کدنویسی را ایمن‌تر، خواناتر، ترکیب‌پذیرتر (Composable) و بهینه‌تر سازد.

در مدل کلاسیک STL (از ++C98 تا ++C17)، سه جزء اصلی یعنی Containers، Iterators و Algorithms به صورت جداگانه عمل می‌کردند و ارتباط میان کانتینر و الگوریتم همواره نیازمند عبور دستی یک جفت تکرارگر آغاز و پایان بود. این طراحی با وجود انعطاف‌پذیری بالا، مشکلات جدی نظیر طولانی بودن کدها، امکان ارسال تکرارگرهای نامتجانس از دو کانتینر متفاوت، خطاهای مبهم قالب‌ها (Template Errors) در زمان کامپایل و نبود راهکاری استاندارد برای زنجیره‌سازی عملیات روی داده‌ها بدون تخصیص حافظه میانی داشت. کتابخانه `std::ranges` در هدر `<ranges>` و الگوریتم‌های متناظر آن در هدر `<algorithm>` این چالش‌ها را به صورت ریشه‌ای برطرف کرده‌اند.

---

## مفاهیم پایه Ranges و سلسله‌مراتب تکرارگرها در ++C20

مفهوم Range در ++C20 به هر شیئی اطلاق می‌شود که بتوان روی عناصر آن پیمایش کرد؛ به عبارت دقیق‌تر، هر نوعی که توابع `()std::ranges::begin` و `()std::ranges::end` روی آن قابل فراخوانی باشند، یک Range محسوب می‌شود. این تعریف شامل تمام کانتینرهای استاندارد مانند `std::vector`، `std::list`، `std::map`، آرایه‌های خام سبک C، رشته‌های `std::string_view` و بازه‌های `std::span` می‌شود.

### مفهوم Sentinel در کتابخانه Ranges

یکی از مهم‌ترین تغییرات معماری در ++C20، جداسازی نوع تکرارگر پایان از تکرارگر آغاز است. در STL سنتی، الگوریتم‌ها انتظار داشتند که `begin` و `end` دقیقاً از یک نوع واحد باشند. در ++C20، پایان یک بازه می‌تواند یک **Sentinel** باشد؛ یعنی نوعی متفاوت که تنها شرط توقف پیمایش را مشخص می‌کند (مثلاً کاراکتر پایان‌دهنده `\0` در رشته‌های سبک C یا یک شمارنده در بازه‌های نامتناهی). تکرارگر و Sentinel تنها باید با عملگر `==` قابل مقایسه باشند.

### سلسله‌مراتب Iterators و Ranges

کتابخانه ++C20 با استفاده از زبان Concepts، دسته‌بندی تکرارگرها و بازه‌ها را از سطح مبتنی بر تگ‌های قدیمی (Iterator Tags) به سطح بررسی‌های زمان کامپایل ارتقا داده است. این سلسله‌مراتب از ضعیف‌ترین قابلیت پیمایش تا قوی‌ترین دسترسی به حافظه مرتب شده است:

| سطح سلسله‌مراتب | کانسپت تکرارگر (Iterator) | کانسپت بازه (Range) | قابلیت کلیدی و نمونه کانتینر |
| --- | --- | --- | --- |
| ورودی (Input) | `std::input_iterator` | `std::ranges::input_range` | خواندن یک‌بار و پیمایش رو به جلو (`std::istream_iterator`) |
| خروجی (Output) | `std::output_iterator` | `std::ranges::output_range` | نوشتن یک‌بار و پیمایش رو به جلو (`std::ostream_iterator`) |
| رو به جلو (Forward) | `std::forward_iterator` | `std::ranges::forward_range` | پیمایش چندباره رو به جلو (`std::forward_list`) |
| دوسویه (Bidirectional) | `std::bidirectional_iterator` | `std::ranges::bidirectional_range` | حرکت به جلو و عقب با `++` و `--` (`std::list`، `std::map`) |
| دسترسی تصادفی (Random Access) | `std::random_access_iterator` | `std::ranges::random_access_range` | پرش با زمان ثابت $O(1)$ و اندیس‌گذاری (`std::deque`) |
| پیوسته (Contiguous) | `std::contiguous_iterator` | `std::ranges::contiguous_range` | ذخیره عناصر در بلوک حافظه پیوسته (`std::vector`، `std::array`) |

هر سطح پایینی در جدول فوق تمام قابلیت‌های سطوح بالاتر خود را نیز داراست؛ برای مثال، هر `contiguous_range` به صورت خودکار یک `random_access_range` و یک `bidirectional_range` نیز هست. معرفی `contiguous_range` در ++C20 این امکان را فراهم کرد که الگوریتم‌ها و کدهای سطح پایین بتوانند به صورت امن از پیوستگی فیزیکی حافظه (مانند بهینه‌سازی‌های SIMD یا `memcpy`) مطمئن شوند.

---

## الگوریتم‌های Range در فضای نام std::ranges

الگوریتم‌های استاندارد سنتی در فضای نام `std` نیازمند دریافت صریح دو تکرارگر هستند، در حالی که نسخه‌های مدرن آن‌ها در فضای نام `std::ranges` (معرفی‌شده در ++C20) می‌توانند کل کانتینر یا بازه را به صورت مستقیم بپذیرند. همچنین این الگوریتم‌ها از نظر فنی به صورت **Niebloids** (اشیای تابعی خاص در کتابخانه استاندارد) پیاده‌سازی شده‌اند تا از تداخل با مکانیزم Argument-Dependent Lookup (ADL) جلوگیری کرده و خطاهای کامپایل بسیار شفاف‌تری ارائه دهند.

برای مقایسه رویکرد سنتی و رویکرد مدرن، بررسی الگوریتم `all_of` در دو حالت زیر گویا است:

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{2, 6, 8, 40, 64, 70};

    // Legacy STL approach using iterator pairs (C++11/17)
    bool legacy_all_even = std::all_of(
        std::begin(numbers), std::end(numbers),
        [](int i) { return i % 2 == 0; }
    );

    // Modern C++20 ranges approach passing the container directly
    auto is_even = [](int i) { return i % 2 == 0; };
    bool ranges_all_even = std::ranges::all_of(numbers, is_even);

    if (legacy_all_even && ranges_all_even) {
        std::cout << "All numbers are even.\n";
    }
}

```

استفاده مستقیم از کانتینر در `()std::ranges::all_of` علاوه بر حذف تکرار نام متغیر `numbers` در `begin` و `end`، امکان رخ دادن باگ ناشی از ارسال `()v1.begin` و `()v2.end` را به صفر می‌رساند.

### الگوریتم‌های پرکاربرد در std::ranges و خروجی جریان‌ها

بسیاری از الگوریتم‌های پرکاربرد مانند `()std::ranges::for_each`، `()std::ranges::sort`، `()std::ranges::find_if` و `()std::ranges::copy` همگی بازنویسی شده‌اند. نکته بسیار مهم درباره الگوریتم‌های جستجو در `std::ranges`، جلوگیری از بروز **Dangling Iterators** است: اگر یک کانتینر موقت (Rvalue) را به `()std::ranges::find` بدهید، خروجی تابع به جای تکرارگر نامعتبر، مقدار `std::ranges::dangling` خواهد بود که تلاش برای Dereference کردن آن در زمان کامپایل خطا می‌دهد.

همچنین ترکیب `()std::ranges::copy` با `std::ostream_iterator` روشی بسیار تمیز و بدون حلقه صریح برای چاپ مستقیم محتوای یک بازه در جریان خروجی فراهم می‌کند:

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{11, 2, 6, 4, 8, 3, 17, 9};

    // Multiply each element in-place by 2
    std::ranges::for_each(numbers, [](int& n) { n *= 2; });

    // Sort the entire container directly
    std::ranges::sort(numbers);

    // Search for the first element greater than 20
    auto it = std::ranges::find_if(numbers, [](int n) { return n > 20; });
    if (it != std::ranges::end(numbers)) {
        std::cout << "First element > 20: " << *it << '\n';
    }

    // Stream elements directly to standard output on the fly
    std::cout << "Sorted elements: ";
    std::ranges::copy(numbers, std::ostream_iterator<int>(std::cout, " "));
    std::cout << '\n';
}

```

---

## الگوریتم‌های جفت تکرارگر مقید (Constrained Iterator Pair Algorithms)

تمام الگوریتم‌های موجود در فضای نام `std::ranges` دارای یک سربارگذاری (Overload) دوم نیز هستند که به جای کل کانتینر، یک تکرارگر آغاز و یک Sentinel پایان دریافت می‌کنند. این نسخه برای مواقعی طراحی شده است که قصد دارید الگوریتم را تنها روی بخشی از یک کانتینر (زیربازه) یا روی ساختارهایی اجرا کنید که فقط تکرارگرهای آن‌ها در دسترس است.

تفاوت حیاتی میان این سربارگذاری‌ها در `std::ranges` با الگوریتم‌های سنتی `std` در **مقید بودن آن‌ها با Concepts** است. این ویژگی باعث می‌شود پیام‌های خطای کامپایلر در صورت ناسازگاری نوع تکرارگر با نیاز الگوریتم، بسیار دقیق و قابل فهم باشند.

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <list>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> vec{11, 2, 6, 4, 8, 3, 17, 9};

    // Using constrained iterator-pair overload on a subrange
    std::ranges::sort(vec.begin(), vec.begin() + 4);

    std::ranges::copy(vec.begin(), vec.end(),
                      std::ostream_iterator<int>(std::cout, " "));
    std::cout << '\n';

    // std::list provides bidirectional iterators, NOT random access iterators
    std::list<int> linked_list{11, 2, 6, 4, 8};

    // Compilation error with a clear concept constraint message:
    // std::ranges::sort requires std::random_access_iterator!
    // std::ranges::sort(linked_list.begin(), linked_list.end());
}

```

اگر در کد بالا به جای `()std::ranges::sort` از `()std::sort` سنتی برای `std::list` استفاده می‌شد، کامپایلر خطای بسیار طولانی و پیچیده‌ای درباره نبود عملگر `-` روی تکرارگرهای لیست تولید می‌کرد؛ اما `()std::ranges::sort` بلافاصله اعلام می‌کند که قید `std::random_access_iterator` برآورده نشده است. به همین دلیل، در ++C20 و نسخه‌های بعدی همواره باید الگوریتم‌های `std::ranges` را حتی هنگام کار با جفت تکرارگرها به نسخه‌های قدیمی `std` ترجیح داد.

---

## قابلیت Projections در الگوریتم‌های Ranges

یکی از کاربردی‌ترین نوآوری‌های کتابخانه `std::ranges`، مفهوم **Projection** است. Projection یک تابع یا قابل‌فراخوانی (Callable) یک‌تایی است که الگوریتم قبل از ارسال هر عنصر به تابع مقایسه (Comparator) یا شرط (Predicate)، آن را روی عنصر اعمال می‌کند تا عنصر را به مقدار یا ویژگی مورد نظر تبدیل کند.

در ++C17 و قبل از آن، اگر می‌خواستید آرایه‌ای از اشیای `Student` را بر اساس سن مرتب کنید، مجبور بودید یک تابع مقایسه سفارشی بنویسید و در آن عضو `m_age` را از دو شیء استخراج کنید. با استفاده از Projection، منطق مقایسه (`std::less` پیش‌فرض) دست‌نخورده باقی می‌ماند و تنها فیلد هدف به الگوریتم معرفی می‌شود. علاوه بر توابع لامبدا، به لطف `()std::invoke` در پس‌زمینه کتابخانه استاندارد، می‌توان از **اشاره‌گر به اعضای داده (Member Data Pointers)** یا **اشاره‌گر به توابع عضو (Member Function Pointers)** نیز مستقیماً به عنوان Projection استفاده کرد.

```cpp
#include <algorithm>
#include <cmath>
#include <compare>
#include <iostream>
#include <ranges>
#include <string>
#include <utility>
#include <vector>

struct Point {
    double m_x{};
    double m_y{};

    double length() const {
        return std::hypot(m_x, m_y);
    }

    // Default ordering based on Euclidean distance from origin
    auto operator<=>(const Point& other) const {
        return length() <=> other.length();
    }
    bool operator==(const Point& other) const {
        return length() == other.length();
    }
};

int main() {
    std::vector<Point> points{{10.0, 90.0}, {30.0, 70.0}, {20.0, 80.0}};

    // 1. Default sort: uses Point::operator<=> (sorts by distance)
    std::ranges::sort(points);

    // 2. Sort using a lambda projection on m_x
    std::ranges::sort(points, std::ranges::less{}, [](const Point& p) {
        return p.m_x;
    });

    // 3. Sort using direct pointer-to-member projection on m_y
    std::ranges::sort(points, std::ranges::less{}, &Point::m_y);

    for (const auto& p : points) {
        std::cout << "Point(" << p.m_x << ", " << p.m_y << ")\n";
    }

    // 4. Using projections with std::ranges::for_each on pairs
    using Pair = std::pair<int, std::string>;
    std::vector<Pair> pairs{{1, "one"}, {2, "two"}, {3, "three"}};

    auto print_val = [](const auto& val) { std::cout << val << ' '; };

    std::cout << "Keys: ";
    std::ranges::for_each(pairs, print_val, &Pair::first);

    std::cout << "\nValues: ";
    std::ranges::for_each(pairs, print_val, &Pair::second);
    std::cout << '\n';
}

```

استفاده از Projection کد را به شدت مختصر، خوانا و عاری از تکرار منطق مقایسه می‌کند. این قابلیت در اکثر الگوریتم‌های `std::ranges` مانند `sort`، `find`، `count_if`، `lower_bound`، `min_element` و `for_each` پشتیبانی می‌شود.

---

## مفهوم Views و Range Adaptors

در حالی که الگوریتم‌ها عملیات را بلافاصله (Eagerly) اجرا می‌کنند، **Views** سازوکاری برای پردازش تنبل (Lazy Evaluation) و بدون مالکیت (Non-owning) روی بازه‌ها ارائه می‌دهند.

یک View یک بازه سبک‌وزن است که مالک داده‌های زیربنایی نیست؛ بلکه مانند یک پنجره یا لنز عمل می‌کند که نحوه مشاهده و پیمایش داده‌های یک بازه دیگر را تغییر می‌دهد.

### ویژگی‌های کلیدی Views

* **عدم مالکیت (Non-owning):** یک View داده‌ها را کپی یا ذخیره نمی‌کند، بلکه صرفاً ارجاعی به بازه اصلی و منطق تبدیل یا فیلتر را نگه می‌دارد.
* **ارزیابی تنبل (Lazy Evaluation):** ساختن یک View هیچ محاسبه یا پیمایشی انجام نمی‌دهد (پیچیدگی زمانی $O(1)$). محاسبات تنها زمانی که کاربر شروع به پیمایش روی View می‌کند (مثلاً در حلقه `for` یا الگوریتم `copy`) و به ازای هر عنصر در لحظه انجام می‌شوند.
* **هزینه کپی ناچیز:** طبق استاندارد ++C20، کپی کردن، جابه‌جا کردن (Move) و تخریب هر View باید در زمان ثابت $O(1)$ انجام شود؛ بنابراین می‌توان آن‌ها را با خیال راحت به صورت مقداری (By Value) به توابع ارسال کرد.

### تفاوت کلاس‌های View و Range Adaptors

در کتابخانه استاندارد دو فضای نام برای کار با Views وجود دارد:

* فضای نام **`std::ranges`** شامل کلاس‌های اصلی View مانند `std::ranges::filter_view`، `std::ranges::transform_view` و `std::ranges::take_view` است.
* فضای نام **`std::views`** (که نام مستعاری برای `std::ranges::views` است) شامل **Range Adaptors** یا اشیای سازنده‌ای مانند `std::views::filter`، `std::views::transform` و `std::views::take` است.

تفاوت مهم این دو در آن است که Range Adaptorها علاوه بر تولید شیء View مناسب، از عملگر لوله (`|`) برای زنجیره‌سازی پشتیبانی می‌کنند و در مواردی مانند `std::views::keys` یا `std::views::all` بهینه‌سازی‌های هوشمند انجام می‌دهند (برای مثال اگر بازه‌ای از قبل یک View باشد، `std::views::all` همان را برمی‌گرداند، در حالی که سازنده مستقیم کلاس ممکن است در برخی انواع با خطای کسر نوع قالب یا محدودیت‌های سازنده مواجه شود). به همین دلیل در کدنویسی عملی همواره از آداپتورهای فضای نام `std::views` استفاده می‌شود.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

void print_view(auto view) {
    // Lazy evaluation happens during iteration here
    for (const auto& item : view) {
        std::cout << item << ' ';
    }
    std::cout << '\n';
}

int main() {
    std::vector<int> numbers{1, 11, 23, 131, 2, 3, 4, 5, 6, 7, 8, 9};

    // filter: keeps elements satisfying the predicate
    auto evens = std::views::filter(numbers, [](int i) { return i % 2 == 0; });
    std::cout << "Evens: ";
    print_view(evens);

    // transform: projects each element to a new value on the fly
    auto multiplied = std::views::transform(numbers, [](int i) { return i * 10; });
    std::cout << "Multiplied: ";
    print_view(multiplied);

    // take: takes the first N elements
    std::cout << "First 5: ";
    print_view(std::views::take(numbers, 5));

    // take_while: takes elements as long as the predicate holds true, then stops
    std::cout << "Take while odd: ";
    print_view(std::views::take_while(numbers, [](int i) { return i % 2 != 0; }));

    // drop: skips the first N elements
    std::cout << "Drop first 5: ";
    print_view(std::views::drop(numbers, 5));

    // drop_while: skips elements while the predicate holds true, then yields the rest
    std::cout << "Drop while odd: ";
    print_view(std::views::drop_while(numbers, [](int i) { return i % 2 != 0; }));
}

```

نکته ظریف در تفاوت بین `filter` و `take_while` (یا `drop_while`) این است که `filter` تمام بازه را بررسی می‌کند و هر عنصری که شرط را داشته باشد برمی‌گرداند، اما `take_while` به محض برخورد با اولین عنصری که شرط را نقض کند، پیمایش را به طور کامل متوقف می‌کند.

---

## ترکیب Views و عملگر لوله (Pipe Operator)

قدرت واقعی کتابخانه Ranges زمانی آشکار می‌شود که چندین View را به یکدیگر متصل کنیم تا یک خط لوله پردازش داده (Data Pipeline) بسازیم. در رویکرد سنتی، برای فیلتر کردن اعداد زوج، به توان رساندن آن‌ها و برداشتن سه عنصر اول، باید چندین حلقه `for` می‌نوشتیم و یا چندین `std::vector` موقت در حافظه ایجاد می‌کردیم.

با استفاده از Viewها، می‌توان این عملیات را بدون تخصیص حتی یک بایت حافظه اضافی با هم ترکیب کرد. این ترکیب به دو روش امکان‌پذیر است: ترکیب تابعی تودرتو (Raw Function Composition) و استفاده از عملگر لوله (`|`).

### مقایسه ترکیب توابع تودرتو و عملگر |

در ترکیب توابع تودرتو، ترتیب خواندن کد از داخل به خارج است که در خطوط لوله طولانی خوانایی را به شدت کاهش می‌دهد. اما عملگر `|` که برای تمام Range Adaptorها سربارگذاری شده است، جریان داده را به صورت طبیعی از چپ به راست نشان می‌دهد:

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <map>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<int> vi{1, 2, 3, 4, 5, 6, 7, 8, 9};
    auto is_even = [](int n) { return n % 2 == 0; };
    auto square = [](int n) { return n * n; };

    // 1. Raw function composition (reads inside-out)
    auto nested_view = std::views::transform(std::views::filter(vi, is_even), square);

    // 2. Pipe operator composition (reads naturally left-to-right)
    auto piped_view = vi
                    | std::views::filter(is_even)
                    | std::views::transform(square);

    for (int val : piped_view) {
        std::cout << val << ' '; // Outputs: 4 16 36 64
    }
    std::cout << '\n';

    // Working with associative containers using keys, values, and reverse
    std::map<std::string, unsigned int> classroom{
        {"John", 11}, {"Mary", 17}, {"Steve", 15}, {"Lucy", 14}, {"Ariel", 12}
    };

    auto before_M = [](const std::string& name) {
        return !name.empty() && static_cast<unsigned char>(name[0]) < static_cast<unsigned char>('M');
    };

    std::cout << "Names before M: ";
    std::ranges::copy(
        classroom | std::views::keys | std::views::filter(before_M),
        std::ostream_iterator<std::string>(std::cout, " ")
    );

    std::cout << "\nAll names in reverse: ";
    std::ranges::copy(
        classroom | std::views::keys | std::views::reverse,
        std::ostream_iterator<std::string>(std::cout, " ")
    );
    std::cout << '\n';
}

```

### محدودیت‌های ساختاری در ترکیب Views

هنگام ترکیب آداپتورها، باید به قیدهای تکرارگر (Iterator Constraints) که هر آداپتور نیاز دارد توجه کرد. برای نمونه، آداپتور `std::views::reverse` نیازمند یک `std::ranges::bidirectional_range` است تا بتواند از انتها به ابتدا حرکت کند. اگر در مثال بالا به جای `std::map` از `std::unordered_map` استفاده می‌کردیم، اعمال `std::views::reverse` منجر به خطای کامپایل می‌شد؛ زیرا `std::unordered_map` تنها یک `forward_range` است و تکرارگر برگشتی ندارد.

---

## کارخانه‌های بازه (Range Factories)

تا اینجا تمام Viewهایی که بررسی کردیم نیازمند یک کانتینر یا بازه از پیش موجود به عنوان منبع داده بودند. اما **Range Factories** توابعی در فضای نام `std::views` هستند که بدون نیاز به هیچ کانتینر اولیه، یک View را از ابتدا (Out of the blue) تولید می‌کنند.

مهم‌ترین کارخانه‌های استاندارد در ++C20 عبارت‌اند از:

* **`std::views::iota`:** دنباله‌ای افزایشی از مقادیر را تولید می‌کند. اگر تنها مقدار شروع به آن داده شود، یک **بازه نامتناهی (Infinite Range)** ایجاد می‌کند که مقادیر را به صورت تنبل و در لحظه تقاضا تولید می‌نماید. اگر مقدار پایان نیز مشخص شود، بازه نیمه‌باز `[start, end)` را می‌سازد.
* **`std::views::empty<T>`:** یک View خالی از نوع `T` بدون هیچ عنصری تولید می‌کند.
* **`std::views::single(val)`:** یک View شامل دقیقاً یک عنصر با مقدار `val` می‌سازد.
* **`std::views::istream<T>(stream)`:** عناصر نوع `T` را به صورت تنبل از یک جریان ورودی مانند `std::cin` یا `std::istringstream` استخراج می‌کند.

```cpp
#include <iostream>
#include <ranges>
#include <sstream>

int main() {
    // 1. Bounded iota: generates numbers from 1 up to 9 (10 is excluded)
    std::cout << "Bounded iota: ";
    for (int i : std::views::iota(1, 10)) {
        std::cout << i << ' ';
    }
    std::cout << '\n';

    // 2. Infinite iota combined with take and transform
    std::cout << "First 5 squares of even numbers: ";
    auto infinite_squares = std::views::iota(1)
                          | std::views::filter([](int n) { return n % 2 == 0; })
                          | std::views::transform([](int n) { return n * n; })
                          | std::views::take(5);

    for (int val : infinite_squares) {
        std::cout << val << ' '; // Outputs: 4 16 36 64 100
    }
    std::cout << '\n';

    // 3. istream_view: lazily parsing integers from a string stream
    std::istringstream input{"10 20 30 40 50"};
    std::cout << "Parsed from stream (doubled): ";
    for (int val : std::views::istream<int>(input) | std::views::transform([](int x) { return x * 2; })) {
        std::cout << val << ' ';
    }
    std::cout << '\n';
}

```

استفاده از `std::views::iota(1)` به تنهایی در یک حلقه `for` بدون شرط توقف یا آداپتورهایی مانند `std::views::take` منجر به یک حلقه بی‌نهایت (یا سرریز عدد صحیح) خواهد شد؛ بنابراین ترکیب کارخانه‌های نامتناهی با آداپتورهای محدودکننده یک الگوی استاندارد و ضروری است.

---

## نکات پیشرفته، محدودیت‌ها و اشتباهات رایج

استفاده حرفه‌ای از کتابخانه Ranges نیازمند آگاهی از جزئیات پیاده‌سازی و دام‌های پنهانی است که می‌تواند منجر به رفتار تعریف‌نشده (Undefined Behavior) یا افت کارایی شود.

### طول عمر داده‌های زیربنایی و Dangling Views

از آنجا که یک View مالک داده‌های خود نیست، طول عمر داده‌های منبع باید از طول عمر View بیشتر باشد. اگر یک کانتینر موقت (Rvalue) را مستقیماً وارد یک خط لوله View کنید و نتیجه را در یک متغیر ذخیره نمایید، آن View به حافظه آزادشده اشاره خواهد کرد:

```cpp
#include <ranges>
#include <vector>

std::vector<int> get_temp_vector() {
    return {1, 2, 3, 4, 5};
}

void danger_example() {
    // UNDEFINED BEHAVIOR: get_temp_vector() is destroyed at the end of this statement!
    // std::views::all only accepts lvalues or safe viewable_ranges (like std::span or std::string_view).
    // Note: C++20 prevents binding an owning container rvalue directly to most views!
    // auto bad_view = get_temp_vector() | std::views::filter([](int x) { return x > 2; });
}

```

کمیته استاندارد ++C20 برای جلوگیری از این خطا، کانسپت `std::ranges::viewable_range` را طراحی کرده است که اجازه نمی‌دهد یک کانتینر مالک موقت (مانند `std::vector` موقت) مستقیماً به آداپتورها لوله شود. با این حال، اگر خودتان یک کانتینر محلی (Lvalue) بسازید و View آن را از تابع برگردانید، باز هم با مشکل Dangling Reference روبه‌رو خواهید شد.

### کش شدن تکرارگر آغاز (Begin Caching) و قانون عدم استفاده از const

برخی Viewها مانند `std::ranges::filter_view` و `std::ranges::drop_while_view` برای تضمین پیچیدگی زمانی سرشکن $O(1)$ هنگام فراخوانی مکرر تابع `()begin`، نتیجه اولین جستجوی خود را در داخل شیء View **کش (Cache)** می‌کنند. به همین دلیل، تابع `()begin` در این Viewها غیر `const` است.

نتیجه عملی این طراحی آن است که **شما نمی‌توانید روی یک `const filter_view` پیمایش کنید** یا آن را به صورت `const auto&` به یک تابع بفرستید:

```cpp
#include <iostream>
#include <ranges>
#include <vector>

// Fails to compile if v is filter_view because filter_view::begin() is non-const!
// void print_broken(const auto& v) {
//     for (auto x : v) std::cout << x;
// }

// Correct way: pass views by value (they are O(1) cheap to copy) or by forwarding reference
void print_correct(auto v) {
    for (const auto& x : v) {
        std::cout << x << ' ';
    }
    std::cout << '\n';
}

```

به همین دلیل، قانون طلایی کار با Viewها در ++C20 این است که آن‌ها را **همواره به صورت مقداری (By Value) یا ارجاع ارسالی (`auto&&`)** به توابع پاس دهید و از قرار دادن کلمه کلیدی `const` روی اشیای View خودداری کنید.

### تغییر داده‌های کانتینر هنگام پیمایش View

اگر هنگام پیمایش یک `filter_view` یا تغییر ساختار کانتینر اصلی (مانند اضافه یا حذف کردن عناصر در `std::vector`)، کش داخلی `filter_view` نامعتبر شود، رفتار برنامه تعریف‌نشده خواهد بود. علاوه بر این، اگر در `std::views::transform` مقداری که به صورت مقداری (By Value) برگردانده می‌شود را به صورت ارجاع تغییر دهید یا در `filter` مقداری را که برای بررسی شرط استفاده شده تغییر دهید، ترتیب عناصر و کش View دچار اختلال می‌شود.

### نبود ابزار تبدیل مستقیم View به کانتینر در ++C20

در استاندارد ++C20، هیچ تابع مستقیمی برای تبدیل نتیجه یک خط لوله View به یک کانتینر جدید (مانند `std::vector`) وجود ندارد و برنامه‌نویس مجبور است از تکرارگرهای `()view.begin` و `()view.end` در سازنده کانتینر استفاده کند (که البته در صورت وجود `filter_view` در خط لوله به دلیل عدم تطابق نوع تکرارگر و Sentinel در سازنده‌های قدیمی، با خطا مواجه می‌شود و نیاز به `std::ranges::copy` و `std::back_inserter` دارد). این محدودیت سرانجام در استاندارد **++C23** با معرفی تابع **`()std::ranges::to`** به طور کامل برطرف شد:

```cpp
#include <algorithm>
#include <iterator>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> source{1, 2, 3, 4, 5, 6};
    auto pipeline = source
                  | std::views::filter([](int n) { return n % 2 == 0; })
                  | std::views::transform([](int n) { return n * 10; });

    // C++20 approach to materialize a filtered view into a vector
    std::vector<int> result_cpp20;
    std::ranges::copy(pipeline, std::back_inserter(result_cpp20));

    // C++23 approach (available in modern compilers with -std=c++23)
    // auto result_cpp23 = pipeline | std::ranges::to<std::vector>();
}

```

### زمان استفاده و عدم استفاده از Ranges و Views

* **زمان مناسب برای استفاده:** هر زمان که نیاز به پردازش زنجیره‌ای داده‌ها، فیلتر کردن، نگاشت مقادیر، پیمایش بخش‌هایی از داده‌ها یا مرتب‌سازی و جستجو بر اساس ویژگی‌های خاص اشیا (با Projections) دارید. Views در کاهش مصرف حافظه، حذف تخصیص‌های موقت و افزایش خوانایی کد در برنامه‌نویسی اعلانی (Declarative) بی‌رقیب هستند.
* **زمان نامناسب برای استفاده:** در حلقه‌های بسیار حساس به کارایی (Hot Loops) که در آن‌ها زنجیره‌های عمیقی از چندین `filter_view` و `transform_view` تودرتو وجود دارد و کامپایلر به دلیل پیچیدگی توابع لامبدا قادر به Inline کردن کامل آن‌ها نیست؛ در چنین مواردی یک حلقه `for` ساده یا الگوریتم‌های برداری سطح پایین می‌تواند کد ماشین بهینه‌تری تولید کند. همچنین در پردازش‌های موازی (Parallel Execution Policies مانند `std::execution::par`) که در ++C17 معرفی شدند، الگوریتم‌های `std::ranges` در ++C20 هنوز از پارامتر `ExecutionPolicy` پشتیبانی نمی‌کنند و برای اجرای موازی همچنان باید از الگوریتم‌های فضای نام `std` استفاده کرد.


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>