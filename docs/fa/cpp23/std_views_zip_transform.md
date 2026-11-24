<div align="center">

[🇺🇸 English](../../en/cpp23/std_views_zip_transform.md) | [🇮🇷 فارسی](./std_views_zip_transform.md)

</div>

---

# **معرفی و بررسی جامع&#x20;**`std::views::zip_transform`**&#x20;در ++C23**

## **مقدمه**

کتابخانه استاندارد Ranges در ++C20 امکانات قدرتمندی برای پردازش مجموعه‌ای از داده‌ها به شیوه‌ای Declarative و ترکیبی معرفی کرد. با استفاده از این امکانات، می‌توان عملیات مختلفی مانند فیلتر کردن، تبدیل کردن و پیمایش داده‌ها را بدون نیاز به نوشتن حلقه‌های تکراری و مدیریت دستی Iteratorها انجام داد.

با وجود این قابلیت‌ها، یکی از نیازهای رایج در برنامه‌نویسی همچنان به شکل مستقیم پوشش داده نشده بود: **پردازش هم‌زمان چند Range و اعمال یک تابع روی عناصر متناظر آن‌ها**.

برای مثال، فرض کنید دو آرایه از اعداد داریم و می‌خواهیم عناصر متناظر آن‌ها را با یکدیگر جمع کنیم. روش سنتی استفاده از یک حلقه و دسترسی هم‌زمان به دو آرایه است. در روش مبتنی بر Ranges نیز می‌توان ابتدا آن‌ها را با `std::views::zip` ترکیب کرد و سپس با یک عملیات Transform نتیجه را تولید کرد.

قابلیت `std::views::zip_transform` در ++C23 دقیقاً برای ساده‌سازی چنین سناریوهایی طراحی شده است.

این قابلیت به ما اجازه می‌دهد چند Range را به‌صورت هم‌زمان پیمایش کنیم و یک Callable را روی عناصر متناظر آن‌ها اجرا کنیم؛ بدون اینکه مجبور باشیم ابتدا یک Container جدید ایجاد کنیم یا نتیجه هر مرحله را به‌صورت موقت ذخیره کنیم.

به بیان ساده، `std::views::zip_transform` ترکیبی از دو مفهوم زیر است:

- **Zip:** ترکیب عناصر متناظر چند Range.
- **Transform:** اعمال یک تابع روی عناصر و تولید نتیجه.

این قابلیت بخشی از Standard Library است و در قالب یک Range Adaptor ارائه می‌شود. استانداردسازی آن از طریق پیشنهاد `P2321R2` انجام شده است. این پیشنهاد علاوه بر `zip_transform`، قابلیت‌های مرتبط دیگری مانند `zip`، `adjacent` و `adjacent_transform` را نیز معرفی می‌کند.

## **مفهوم اصلی و نحوه عملکرد**

### **تعریف std::views::zip_transform**

نوع `std::views::zip_transform` یک Range Adaptor است که یک Callable و یک یا چند Range دریافت می‌کند و یک View جدید می‌سازد.

اگر تابع ورودی را \\(f\\) و Rangeهای ورودی را \\(R_1\\)، \\(R_2\\) تا \\(R_n\\) در نظر بگیریم، عنصر شماره \\(i\\) در خروجی برابر است با:

\\[ V[i] = f(R_1[i], R_2[i], \ldots, R_n[i]) \\]

برای مثال، اگر دو Range زیر را داشته باشیم:

```
std::vector<int> a{1, 2, 3, 4};
std::vector<int> b{10, 20, 30, 40};
```

و بخواهیم عناصر متناظر آن‌ها را جمع کنیم:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4};
    std::vector<int> b{10, 20, 30, 40};

    auto result = std::views::zip_transform(
        [](int x, int y) {
            return x + y;
        },
        a,
        b
    );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
11 22 33 44
```

در این مثال، تابع Lambda برای هر موقعیت از عناصر دو Range فراخوانی می‌شود.

نکته مهم این است که `result` یک Container جدید نیست؛ بلکه یک View است که نحوه محاسبه عناصر خروجی را تعریف می‌کند.

### **تفاوت View و Container**

برای درک صحیح رفتار `zip_transform` باید تفاوت View و Container را بشناسیم.

یک Container مانند `std::vector` معمولاً مسئول نگهداری و مدیریت داده‌هاست. در مقابل، یک View معمولاً نمایی از داده‌های موجود یا یک محاسبه مبتنی بر آن‌ها ارائه می‌کند.

برای مثال:

```
std::vector<int> values{1, 2, 3, 4};

auto transformed = values | std::views::transform(
    [](int value) {
        return value * 2;
    }
);
```

در این مثال، View جدید عناصر را هنگام دسترسی محاسبه می‌کند.

رفتار `zip_transform` نیز از همین رویکرد پیروی می‌کند؛ با این تفاوت که به‌جای یک Range، چند Range را دریافت می‌کند.

بنابراین:

- هیچ Container جدیدی برای ذخیره نتایج ایجاد نمی‌شود.
- عناصر هنگام Dereference کردن Iterator محاسبه می‌شوند.
- نتیجه محاسبه به‌صورت خودکار در حافظه ذخیره نمی‌شود.
- برای استفاده مجدد از نتایج، می‌توان آن‌ها را در یک Container Materialize کرد.

### **ارزیابی تنبل یا Lazy Evaluation**

یکی از مهم‌ترین ویژگی‌های `zip_transform`، استفاده از Lazy Evaluation است.

به این معنا که ایجاد View باعث اجرای تابع Transform برای تمام عناصر نمی‌شود.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4};

    auto result = std::views::zip_transform(
        [](int value) {
            std::cout << "Processing: " << value << '\n';
            return value * 10;
        },
        values
    );

    std::cout << "View created\n";

    auto it = result.begin();

    std::cout << *it << '\n';
}
```

خروجی:

```
View created
Processing: 1
10
```

در این مثال، تنها زمانی که Iterator مربوط به View Dereference می‌شود، تابع Lambda اجرا خواهد شد.

این رفتار چند مزیت مهم دارد:

- جلوگیری از محاسبات غیرضروری.
- کاهش مصرف حافظه.
- امکان ترکیب چند Range Adaptor بدون ایجاد Containerهای موقت.
- امکان پردازش داده‌ها به‌صورت تدریجی.

البته Lazy بودن به معنای Memoization نیست. اگر یک Iterator چند بار Dereference شود، تابع Transform ممکن است در هر بار دوباره اجرا شود.

این موضوع در زمان طراحی Callable اهمیت زیادی دارد.

## **نحوه استفاده از std::views::zip_transform**

### **هدر موردنیاز**

قابلیت `zip_transform` در هدر زیر تعریف شده است:

```
#include <ranges>
```

برای مثال‌هایی که از `std::vector` استفاده می‌کنند، هدر زیر نیز لازم است:

```
#include <vector>
```

### **امضای کلی**

ساختار کلی فراخوانی این Range Adaptor به شکل زیر است:

```
auto result = std::views::zip_transform(
    callable,
    range1,
    range2,
    range3
);
```

پارامترها عبارت‌اند از:

- `callable`: تابع، Lambda، Function Object یا هر شیء قابل فراخوانی معتبر.
- `range1`, `range2`, ...: مجموعه Rangeهای ورودی.

تعداد Rangeهای ورودی باید حداقل یک عدد باشد.

همچنین تمام Rangeهای ورودی باید شرایط موردنیاز استاندارد برای استفاده در این View را داشته باشند.

### **استفاده از Lambda**

رایج‌ترین روش استفاده از `zip_transform`، ارسال یک Lambda است.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> prices{100, 200, 300};
    std::vector<int> quantities{2, 3, 4};

    auto totals = std::views::zip_transform(
        [](int price, int quantity) {
            return price * quantity;
        },
        prices,
        quantities
    );

    for (auto total : totals) {
        std::cout << total << '\n';
    }
}
```

خروجی:

```
200
600
1200
```

در این مثال، تابع Lambda قیمت و تعداد متناظر هر محصول را دریافت می‌کند و مبلغ آن را محاسبه می‌کند.

### **استفاده از Function Object**

استفاده از Lambda تنها روش ممکن نیست. می‌توان از Function Object نیز استفاده کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

struct Multiply {
    constexpr int operator()(int a, int b) const {
        return a * b;
    }
};

int main() {
    std::vector<int> a{2, 3, 4};
    std::vector<int> b{5, 6, 7};

    auto result = std::views::zip_transform(
        Multiply{},
        a,
        b
    );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
10 18 28
```

استفاده از Function Object در شرایطی مفید است که:

- منطق محاسباتی پیچیده باشد.
- به State مشخصی برای انجام محاسبات نیاز داشته باشیم.
- بخواهیم یک Callable را در بخش‌های مختلف برنامه مجدداً استفاده کنیم.
- بخواهیم رفتار تابع را با Templateها یا Overloadها توسعه دهیم.

### **استفاده از توابع استاندارد**

می‌توان از Function Objectهای موجود در Standard Library نیز استفاده کرد.

برای مثال، برای جمع کردن عناصر دو Range می‌توان از `std::plus` استفاده کرد:

```
#include <functional>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
11 22 33
```

در این مثال، `std::plus<>` یک Function Object با Template Argument Deduction است که نوع عملیات را بر اساس آرگومان‌های دریافتی مشخص می‌کند.

برای عملیات‌هایی مانند جمع، تفریق، ضرب و مقایسه، استفاده از Function Objectهای استاندارد می‌تواند کد را ساده‌تر و خواناتر کند.

## **ارتباط با std::views::zip**

### **معرفی zip**

برای درک بهتر `zip_transform` باید ابتدا با `std::views::zip` آشنا باشیم.

قابلیت `std::views::zip` نیز در ++C23 معرفی شده است و چند Range را به یک Range ترکیبی تبدیل می‌کند.

هر عنصر خروجی شامل عناصر متناظر Rangeهای ورودی است.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{1, 2, 3};
    std::vector<char> letters{'A', 'B', 'C'};

    auto zipped = std::views::zip(numbers, letters);

    for (auto [number, letter] : zipped) {
        std::cout << number << ' ' << letter << '\n';
    }
}
```

خروجی:

```
1 A
2 B
3 C
```

در اینجا، هر عنصر خروجی یک ساختار Tuple-like است که مقادیر متناظر دو Range را در اختیار قرار می‌دهد.

### **تفاوت zip و zip_transform**

تفاوت اصلی این دو قابلیت در نوع خروجی آن‌هاست.

| ویژگی                | `std::views::zip`         | `std::views::zip_transform`   |
| -------------------- | ------------------------- | ----------------------------- |
| تعداد ورودی‌ها       | یک یا چند Range           | یک Callable و یک یا چند Range |
| خروجی                | مجموعه‌ای از Tuple-likeها | نتیجه اجرای Callable          |
| اجرای تابع           | ندارد                     | هنگام Dereference             |
| Lazy Evaluation      | بله                       | بله                           |
| تخصیص Container جدید | خیر                       | خیر                           |
| کاربرد اصلی          | پیمایش هم‌زمان            | پردازش هم‌زمان                |

برای مثال، کد زیر با `zip` عناصر دو Range را ترکیب می‌کند:

```
auto zipped = std::views::zip(a, b);
```

اما کد زیر همان عناصر را مستقیماً جمع می‌کند:

```
auto transformed = std::views::zip_transform(
    std::plus<>{},
    a,
    b
);
```

از نظر مفهومی، می‌توان رفتار `zip_transform` را مشابه اعمال `std::views::transform` روی خروجی `std::views::zip` دانست؛ اما این دو عبارت از نظر نوع View و جزئیات پیاده‌سازی الزاماً یکسان نیستند.

در واقع، `zip_transform` برای این سناریو به‌صورت اختصاصی در Standard Library طراحی شده است.

### **ترکیب دستی zip و transform**

پیش از معرفی `zip_transform`، می‌توانستیم عملیات مشابهی را با ترکیب دو Range Adaptor انجام دهیم.

```
#include <functional>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    auto result =
        std::views::zip(a, b)
        | std::views::transform(
            [](const auto& values) {
                return std::get<0>(values)
                     + std::get<1>(values);
            }
        );
}
```

این روش معتبر است، اما زمانی که تعداد Rangeها افزایش پیدا می‌کند، دسترسی به عناصر Tuple-like و نوشتن کد می‌تواند پیچیده‌تر شود.

در مقابل، `zip_transform` مستقیماً عناصر را به Callable ارسال می‌کند:

```
auto result = std::views::zip_transform(
    std::plus<>{},
    a,
    b
);
```

این روش:

- کوتاه‌تر است.
- بیان مستقیم‌تری از هدف برنامه دارد.
- نیازی به استخراج عناصر از Tuple ندارد.
- برای تعداد بیشتری از Rangeها مقیاس‌پذیرتر است.

## **ساختار داخلی و الزامات استاندارد**

### **معرفی zip_transform_view**

نوع View واقعی که توسط `std::views::zip_transform` تولید می‌شود، در Namespace زیر قرار دارد:

```
std::ranges::zip_transform_view
```

این کلاس Template، مسئول نگهداری Callable و Viewهای ورودی است.

نمای ساده‌شده تعریف آن به شکل زیر است:

```
namespace std::ranges {

    template<
        std::move_constructible F,
        std::ranges::input_range... Views
    >
    requires
        (std::ranges::view<Views> && ...) &&
        (sizeof...(Views) > 0) &&
        std::is_object_v<F> &&
        std::regular_invocable<
            F&,
            std::ranges::range_reference_t<Views>...
        > &&
        requires {
            typename std::invoke_result_t<
                F&,
                std::ranges::range_reference_t<Views>...
            >;
        }
    class zip_transform_view;

}
```

این کد صرفاً نمایی آموزشی از شرایط Template است و تعریف کامل استاندارد محسوب نمی‌شود.

در استاندارد واقعی، شرط مربوط به معتبر بودن Reference نوع خروجی نیز با مفهوم داخلی `can-reference` بیان می‌شود.

### **شرط move_constructible**

Callable باید شرایط Concept مربوط به `std::move_constructible` را داشته باشد.

این موضوع به این معناست که شیء Callable باید قابلیت انتقال به داخل View را داشته باشد.

برای مثال:

```
auto callable = [](int value) {
    return value * 2;
};

auto result = std::views::zip_transform(
    std::move(callable),
    values
);
```

البته برای Lambdaهای معمولی، انتقال Callable معمولاً بدون مشکل انجام می‌شود.

نکته مهم این است که استاندارد الزام نمی‌کند Callable حتماً Copyable باشد؛ شرط اصلی آن `move_constructible` است.

### **شرط input_range**

هر Range ورودی باید Concept مربوط به `std::ranges::input_range` را مدل‌سازی کند.

این Concept مشخص می‌کند که Range حداقل قابلیت پیمایش ورودی را دارد.

بنابراین استفاده از انواع مختلف Range امکان‌پذیر است، از جمله:

- `std::vector`
- `std::list`
- `std::array`
- `std::span`
- `std::ranges::subrange`
- برخی Viewهای استاندارد
- Rangeهای سفارشی که شرایط لازم را داشته باشند.

لازم نیست تمام Rangeها از نوع Container باشند.

### **شرط view**

نوع Viewهای داخلی باید Concept مربوط به `std::ranges::view` را مدل‌سازی کند.

نکته مهم این است که در فراخوانی عمومی `std::views::zip_transform` می‌توان Container یا سایر `viewable_range`ها را ارسال کرد.

Standard Library در صورت نیاز آن‌ها را به View مناسب تبدیل می‌کند.

برای مثال:

```
std::vector<int> values{1, 2, 3};

auto result = std::views::zip_transform(
    [](int value) {
        return value * 3;
    },
    values
);
```

در این مثال، نیازی نیست ابتدا `values` را به یک View تبدیل کنیم.

### **شرط regular_invocable**

یکی از مهم‌ترین محدودیت‌های Callable، شرط `std::regular_invocable` است.

این Concept از نظر ساختاری شبیه `std::invocable` است، اما الزامات معنایی بیشتری دارد.

به‌طور خاص، عملیات فراخوانی باید از نظر معنایی Equality-Preserving باشد.

این موضوع برای Viewهایی که محاسبات را هنگام Dereference انجام می‌دهند اهمیت زیادی دارد.

برای مثال، تابع زیر از نظر ظاهری یک Callable معتبر است:

```
int counter = 0;

auto callable = [&counter](int value) {
    ++counter;
    return value + counter;
};
```

اما رفتار آن با هر بار فراخوانی تغییر می‌کند.

چنین Callableای الزامات معنایی `regular_invocable` را رعایت نمی‌کند.

نکته مهم این است که Compiler معمولاً نمی‌تواند تمام الزامات معنایی این Concept را به‌صورت خودکار بررسی کند. بنابراین ممکن است چنین کدی کامپایل شود، اما همچنان رفتار آن با الزامات استاندارد سازگار نباشد.

در نتیجه، بهتر است Callable مورد استفاده در `zip_transform` تا حد امکان Pure و بدون Side Effect باشد.

### **شرط can-reference**

نوع بازگشتی Callable نیز باید شرایط خاصی داشته باشد.

استاندارد از مفهوم داخلی `can-reference` استفاده می‌کند.

این مفهوم به‌طور خلاصه بررسی می‌کند که آیا می‌توان برای نوع خروجی، یک Reference معتبر ایجاد کرد یا خیر.

به همین دلیل، خروجی‌هایی مانند `void` برای `zip_transform` معتبر نیستند.

برای مثال:

```
auto invalid = std::views::zip_transform(
    [](int value) {
        std::cout << value;
    },
    values
);
```

این کد معتبر نیست؛ زیرا Lambda مقدار `void` برمی‌گرداند.

در مقابل، کد زیر معتبر است:

```
auto valid = std::views::zip_transform(
    [](int value) {
        return value * 2;
    },
    values
);
```

بنابراین `zip_transform` برای تولید یک Range از نتایج طراحی شده است، نه صرفاً اجرای یک عملیات جانبی روی عناصر.

## **رفتار Iterator و ویژگی‌های Range**

### **قابلیت‌های Iterator**

نوع Iterator مربوط به `zip_transform_view` از ویژگی‌های Iteratorهای داخلی `zip_view` پیروی می‌کند.

به‌طور کلی، اگر View ترکیبی داخلی یک قابلیت Range را داشته باشد، `zip_transform_view` نیز می‌تواند همان قابلیت را ارائه کند.

مهم‌ترین Conceptهای قابل پشتیبانی عبارت‌اند از:

- `input_range`
- `forward_range`
- `bidirectional_range`
- `random_access_range`
- `common_range`
- `sized_range`

برای مثال، اگر تمام Rangeهای ورودی قابلیت Random Access داشته باشند، View خروجی نیز می‌تواند یک `random_access_range` باشد.

نمونه:

```
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{4, 5, 6};

    auto result = std::views::zip_transform(
        [](int x, int y) {
            return x + y;
        },
        a,
        b
    );

    static_assert(
        std::ranges::random_access_range<
            decltype(result)
        >
    );

    auto value = result[1];
}
```

در این مثال، چون هر دو ورودی از نوع `std::vector` هستند، View خروجی نیز قابلیت Random Access دارد.

### **دسترسی تصادفی به عناصر**

در صورت پشتیبانی Random Access، می‌توان از عملگر Index استفاده کرد.

```
auto result = std::views::zip_transform(
    std::plus<>{},
    a,
    b
);

int value = result[2];
```

در این حالت، عناصر متناظر در موقعیت سوم دو Range دریافت می‌شوند و تابع جمع روی آن‌ها اجرا می‌شود.

این قابلیت برای الگوریتم‌هایی که نیاز به دسترسی مستقیم به موقعیت‌های مختلف دارند مفید است.

### **پیمایش معکوس**

اگر تمام شرایط لازم برای Bidirectional Range برقرار باشند، می‌توان View خروجی را از انتها به ابتدا پیمایش کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );

    for (auto it = result.rbegin();
         it != result.rend();
         ++it) {
        std::cout << *it << ' ';
    }
}
```

خروجی:

```
33 22 11
```

قابلیت `rbegin` و `rend` از طریق امکانات Range و `view_interface` و با توجه به ویژگی‌های View داخلی قابل استفاده است.

### **ویژگی Sized Range**

اگر View داخلی `zip_view` از نوع `sized_range` باشد، View خروجی نیز قابلیت محاسبه اندازه را خواهد داشت.

برای `zip_transform`، اندازه خروجی برابر با کمترین اندازه بین Rangeهای ورودی است.

\\[ Size(V) = \min(Size(R_1), Size(R_2), \ldots, Size(R_n)) \\]

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4, 5};
    std::vector<int> b{10, 20, 30};

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );

    std::cout << std::ranges::size(result);
}
```

خروجی:

```
3
```

در این مثال، اندازه Range اول پنج و اندازه Range دوم سه است. بنابراین View خروجی فقط سه عنصر دارد.

## **مدیریت Rangeهای با اندازه متفاوت**

### **رفتار در صورت اختلاف اندازه**

یکی از نکات مهم `zip_transform`، رفتار آن هنگام دریافت Rangeهایی با اندازه متفاوت است.

این قابلیت مانند `std::views::zip` از کوتاه‌ترین Range پیروی می‌کند.

به عبارت دیگر، عملیات Transform زمانی پایان می‌یابد که حداقل یکی از Rangeها به انتهای خود برسد.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4, 5};
    std::vector<int> b{10, 20, 30};

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
11 22 33
```

چهارمین و پنجمین عنصر Range اول پردازش نمی‌شوند.

این رفتار از نظر طراحی مشابه `zip` در بسیاری از زبان‌های برنامه‌نویسی است.

### **چرا کوتاه‌ترین Range ملاک قرار می‌گیرد؟**

فرض کنید دو آرایه با اندازه‌های متفاوت داشته باشیم و بخواهیم عناصر متناظر آن‌ها را جمع کنیم.

اگر پیمایش تا انتهای Range بزرگ‌تر ادامه پیدا کند، ممکن است به عناصر نامعتبر Range کوچک‌تر دسترسی پیدا کنیم.

برای جلوگیری از چنین وضعیتی، استاندارد رفتار را به‌گونه‌ای تعریف می‌کند که پیمایش در کوتاه‌ترین Range متوقف شود.

### **بررسی برابری اندازه‌ها**

نکته مهم این است که `zip_transform` الزام نمی‌کند تمام Rangeها اندازه یکسانی داشته باشند.

اگر منطق برنامه نیازمند پردازش تمام عناصر باشد، باید اندازه‌ها را جداگانه بررسی کنیم.

```
#include <cassert>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    assert(std::ranges::size(a) ==
           std::ranges::size(b));

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );
}
```

این موضوع به‌خصوص در پردازش داده‌های مالی، علمی، آماری و داده‌های جدولی اهمیت دارد؛ زیرا ممکن است اختلاف اندازه‌ها نشانه وجود خطا در داده‌های ورودی باشد.

## **مدیریت Referenceها و Value Categoryها**

### **دسترسی به عناصر اصلی**

یکی از ویژگی‌های مهم `zip_transform` این است که تابع ورودی، عناصر Rangeها را بر اساس نوع Reference مربوط به هر Range دریافت می‌کند.

برای مثال، اگر یک `std::vector<int>` به تابع ارسال شود، نوع Reference آن معمولاً `int&` است.

بنابراین می‌توان عناصر اصلی را بدون کپی کردن تغییر داد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto result = std::views::zip_transform(
        [](int& value) -> int& {
            return value;
        },
        values
    );

    for (int& value : result) {
        value *= 10;
    }

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
10 20 30
```

در این مثال، تابع Lambda یک Reference به عنصر اصلی برمی‌گرداند.

بنابراین تغییر عناصر خروجی، داده‌های Container اصلی را نیز تغییر می‌دهد.

### **تفاوت خروجی Value و Reference**

نوع بازگشتی Callable نقش مهمی در رفتار View دارد.

اگر Callable یک Value برگرداند، عنصر خروجی نتیجه محاسبه خواهد بود.

```
auto result = std::views::zip_transform(
    [](int& value) {
        return value * 2;
    },
    values
);
```

اما اگر Callable یک Reference برگرداند، View می‌تواند مستقیماً به داده اصلی دسترسی داشته باشد.

```
auto result = std::views::zip_transform(
    [](int& value) -> int& {
        return value;
    },
    values
);
```

این تفاوت برای مدیریت حافظه، عملکرد و امکان تغییر داده‌ها بسیار مهم است.

### **استفاده از const Reference**

اگر قصد تغییر داده‌ها را نداریم، بهتر است Callable را با Reference ثابت طراحی کنیم.

```
auto result = std::views::zip_transform(
    [](const int& a, const int& b) {
        return a + b;
    },
    first,
    second
);
```

این روش برای محاسبات Read-only مناسب است.

همچنین از کپی شدن غیرضروری عناصر بزرگ جلوگیری می‌کند.

### **استفاده از Proxy Reference**

در ++C، همه Containerها الزاماً Reference معمولی به عناصر خود ارائه نمی‌کنند.

برای مثال، `std::vector<bool>` یک پیاده‌سازی تخصصی دارد که دسترسی به عناصر آن از طریق Proxy Reference انجام می‌شود.

بنابراین نباید همیشه فرض کنیم که نوع Reference هر Range دقیقاً `T&` است.

برای نوشتن Callableهای عمومی‌تر می‌توان از `auto` استفاده کرد:

```
auto result = std::views::zip_transform(
    [](auto&& value) {
        return !static_cast<bool>(value);
    },
    flags
);
```

این روش برای سازگاری با انواع مختلف Reference مفید است.

البته استفاده از `auto&&` به‌تنهایی تمام مسائل مربوط به Proxy Referenceها را حل نمی‌کند؛ منطق تابع همچنان باید با عملیات و تبدیل‌های معتبر نوع ورودی سازگار باشد.

## **قابلیت‌های پیشرفته و تکنیک‌های کاربردی**

### **استفاده از سه یا چند Range**

یکی از مزیت‌های اصلی `zip_transform` امکان دریافت تعداد دلخواهی Range است.

برای مثال، فرض کنید بخواهیم مساحت تعدادی مستطیل را محاسبه کنیم.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> widths{2.0, 3.0, 4.0};
    std::vector<double> heights{5.0, 6.0, 7.0};

    std::vector<double> scales{1.0, 2.0, 0.5};

    auto areas = std::views::zip_transform(
        [](double width, double height, double scale) {
            return width * height * scale;
        },
        widths,
        heights,
        scales
    );

    for (double area : areas) {
        std::cout << area << ' ';
    }
}
```

خروجی:

```
10 36 14
```

در این مثال، هر عنصر خروجی بر اساس سه مقدار متناظر محاسبه می‌شود.

این ویژگی در موارد زیر کاربرد دارد:

- محاسبات هندسی.
- پردازش داده‌های چندبعدی.
- محاسبات مالی.
- پردازش سیگنال.
- محاسبات ماتریسی.
- ترکیب چند ویژگی از یک Dataset.

### **ترکیب با std::views::filter**

می‌توان `zip_transform` را با سایر Range Adaptorها ترکیب کرد.

برای مثال، فرض کنید بخواهیم ابتدا مجموع عناصر دو Range را محاسبه کنیم و سپس فقط نتایجی را نگه داریم که از یک مقدار مشخص بزرگ‌تر هستند.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 10, 20, 30, 40};
    std::vector<int> b{2, 5, 10, 15, 20};

    auto result =
        std::views::zip_transform(
            [](int x, int y) {
                return x + y;
            },
            a,
            b
        )
        | std::views::filter(
            [](int value) {
                return value >= 30;
            }
        );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
30 45 60
```

در این مثال، ابتدا مجموع عناصر متناظر محاسبه می‌شود و سپس شرط Filter روی نتیجه اعمال می‌شود.

نکته مهم این است که هر دو عملیات Lazy هستند و ایجاد Viewها به‌تنهایی باعث پردازش تمام عناصر نمی‌شود.

### **ترکیب با std::views::take**

با استفاده از `std::views::take` می‌توان تعداد عناصر موردنیاز را محدود کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4, 5, 6};
    std::vector<int> b{10, 20, 30, 40, 50, 60};

    auto result =
        std::views::zip_transform(
            std::plus<>{},
            a,
            b
        )
        | std::views::take(3);

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
11 22 33
```

این روش زمانی مفید است که فقط به بخش ابتدایی داده‌ها نیاز داشته باشیم.

### **ترکیب با std::views::drop**

اگر بخواهیم چند عنصر ابتدایی را نادیده بگیریم، می‌توان از `std::views::drop` استفاده کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4, 5};
    std::vector<int> b{10, 20, 30, 40, 50};

    auto result =
        std::views::zip_transform(
            std::plus<>{},
            a,
            b
        )
        | std::views::drop(2);

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
33 44 55
```

در این مثال، دو نتیجه ابتدایی کنار گذاشته می‌شوند.

### **ترکیب با std::views::enumerate**

قابلیت `std::views::enumerate` نیز در ++C23 معرفی شده است.

این View امکان پیمایش عناصر همراه با Index آن‌ها را فراهم می‌کند.

می‌توان از آن برای پردازش نتایج `zip_transform` همراه با موقعیت هر عنصر استفاده کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{10, 20, 30};
    std::vector<int> b{1, 2, 3};

    auto transformed = std::views::zip_transform(
        std::minus<>{},
        a,
        b
    );

    for (auto [index, value] :
         std::views::enumerate(transformed)) {
        std::cout << index << ": " << value << '\n';
    }
}
```

خروجی:

```
0: 9
1: 18
2: 27
```

این ترکیب برای موارد زیر مفید است:

- پردازش داده‌های دارای Index.
- تولید گزارش.
- بررسی موقعیت عناصر نامعتبر.
- الگوریتم‌هایی که به مقدار و موقعیت آن نیاز دارند.

### **تبدیل نتیجه به Container**

از آنجا که `zip_transform` یک View تولید می‌کند، در صورت نیاز به ذخیره دائمی نتایج باید آن‌ها را به یک Container تبدیل کنیم.

در ++C23 می‌توان از `std::ranges::to` استفاده کرد.

```
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    auto result = std::views::zip_transform(
        std::plus<>{},
        a,
        b
    );

    auto output = result
        | std::ranges::to<std::vector>();
}
```

در این مثال، View به یک `std::vector` تبدیل می‌شود.

تفاوت مهم این دو حالت:

```
auto view = std::views::zip_transform(
    std::plus<>{},
    a,
    b
);
```

و:

```
auto container = view
    | std::ranges::to<std::vector>();
```

در حالت اول، نتیجه محاسباتی به‌صورت Lazy ارائه می‌شود؛ اما در حالت دوم، نتایج در حافظه ذخیره می‌شوند.

## **کاربردهای عملی در برنامه‌نویسی حرفه‌ای**

### **محاسبه مجموع دو مجموعه داده**

یکی از کاربردهای اصلی `zip_transform` انجام محاسبات Element-wise است.

برای مثال، در یک سیستم پردازش داده می‌توان دو مجموعه مقدار را با یکدیگر جمع کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> actual{10.5, 20.0, 30.5};
    std::vector<double> adjustment{1.5, 2.0, 3.5};

    auto adjusted = std::views::zip_transform(
        std::plus<>{},
        actual,
        adjustment
    );

    for (double value : adjusted) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
12 22 34
```

این الگو در پردازش داده‌های آماری، مالی و علمی کاربرد زیادی دارد.

### **محاسبه اختلاف دو مجموعه**

برای مقایسه دو مجموعه داده می‌توان اختلاف عناصر متناظر را محاسبه کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> expected{100, 200, 300};
    std::vector<double> actual{95, 210, 290};

    auto differences = std::views::zip_transform(
        std::minus<>{},
        actual,
        expected
    );

    for (double difference : differences) {
        std::cout << difference << ' ';
    }
}
```

خروجی:

```
-5 10 -10
```

این روش می‌تواند برای محاسبه خطا در سیستم‌های پردازش داده و الگوریتم‌های پیش‌بینی استفاده شود.

### **محاسبه Dot Product**

یکی از کاربردهای مهم این قابلیت، پیاده‌سازی بخشی از محاسبات برداری است.

Dot Product دو بردار برابر است با مجموع حاصل‌ضرب عناصر متناظر آن‌ها:

\\[ A \cdot B = \sum\_{i=0}^{n-1} A_i B_i \\]

برای پیاده‌سازی این عملیات می‌توان از `zip_transform` و `std::ranges::fold_left` استفاده کرد.

```
#include <functional>
#include <iostream>
#include <numeric>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3, 4};
    std::vector<int> b{5, 6, 7, 8};

    auto products = std::views::zip_transform(
        std::multiplies<>{},
        a,
        b
    );

    int dotProduct = std::ranges::fold_left(
        products,
        0,
        std::plus<>{}
    );

    std::cout << dotProduct;
}
```

خروجی:

```
70
```

در این مثال:

- `zip_transform` حاصل‌ضرب عناصر متناظر را تولید می‌کند.
- `fold_left` تمام نتایج را با یک عملیات جمع ترکیب می‌کند.

این روش بدون ایجاد یک Container موقت برای حاصل‌ضرب‌ها، عملیات را به‌صورت Lazy انجام می‌دهد.

البته در برنامه‌های عددی حساس باید به Overflow، دقت محاسبات اعشاری و نوع داده مورد استفاده توجه کرد.

### **محاسبه فاصله اقلیدسی**

برای دو بردار می‌توان مجموع مربع اختلاف عناصر متناظر را محاسبه کرد.

فرمول فاصله اقلیدسی:

\\[ d(A,B) = \sqrt{ \sum\_{i=0}^{n-1} (A_i-B_i)^2 } \\]

پیاده‌سازی:

```
#include <cmath>
#include <functional>
#include <iostream>
#include <numeric>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> a{1.0, 2.0, 3.0};
    std::vector<double> b{4.0, 6.0, 3.0};

    auto squaredDifferences =
        std::views::zip_transform(
            [](double x, double y) {
                const double difference = x - y;
                return difference * difference;
            },
            a,
            b
        );

    double sum = std::ranges::fold_left(
        squaredDifferences,
        0.0,
        std::plus<>{}
    );

    double distance = std::sqrt(sum);

    std::cout << distance;
}
```

خروجی:

```
5
```

این الگو در الگوریتم‌های Machine Learning، پردازش سیگنال و محاسبات هندسی کاربرد دارد.

### **محاسبه وزن‌دار داده‌ها**

فرض کنید مجموعه‌ای از مقادیر و وزن‌های متناظر آن‌ها را داریم.

می‌توان با استفاده از `zip_transform` حاصل‌ضرب هر مقدار در وزن مربوطه را محاسبه کرد.

```
#include <iostream>
#include <numeric>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> values{10, 20, 30};
    std::vector<double> weights{0.2, 0.3, 0.5};

    auto weightedValues =
        std::views::zip_transform(
            [](double value, double weight) {
                return value * weight;
            },
            values,
            weights
        );

    double result = std::ranges::fold_left(
        weightedValues,
        0.0,
        std::plus<>{}
    );

    std::cout << result;
}
```

خروجی:

```
23
```

این روش در محاسبات آماری و سیستم‌های امتیازدهی قابل استفاده است.

### **پردازش هم‌زمان داده‌های ساختاریافته**

فرض کنید اطلاعات کاربران در چند Container جداگانه نگهداری می‌شود.

```
std::vector<std::string> names;
std::vector<int> ages;
std::vector<double> scores;
```

با استفاده از `zip_transform` می‌توان داده‌های متناظر را بدون ساختن یک Container واسط ترکیب کرد.

```
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> names{
        "Alice", "Bob", "Charlie"
    };

    std::vector<int> ages{
        25, 30, 35
    };

    std::vector<double> scores{
        85.5, 90.0, 78.5
    };

    auto descriptions =
        std::views::zip_transform(
            [](const std::string& name,
               int age,
               double score) {
                return name + " (" +
                    std::to_string(age) +
                    "): " +
                    std::to_string(score);
            },
            names,
            ages,
            scores
        );

    for (const auto& description : descriptions) {
        std::cout << description << '\n';
    }
}
```

خروجی:

```
Alice (25): 85.500000
Bob (30): 90.000000
Charlie (35): 78.500000
```

این الگو زمانی مفید است که داده‌ها به دلایل معماری یا عملکردی در Containerهای جداگانه ذخیره شده باشند.

با این حال، اگر این داده‌ها از نظر مفهومی همیشه یک موجودیت واحد را تشکیل می‌دهند، استفاده از یک Container از Structها ممکن است طراحی مناسب‌تری باشد.

## **بررسی تفاوت با روش‌های قدیمی‌تر**

### **مقایسه با حلقه for معمولی**

قبل از معرفی `zip_transform`، رایج‌ترین روش برای پردازش چند مجموعه استفاده از حلقه‌های دستی بود.

```
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    std::vector<int> result;

    result.reserve(a.size());

    for (std::size_t i = 0;
         i < a.size();
         ++i) {
        result.push_back(a[i] + b[i]);
    }
}
```

این روش کاملاً معتبر است، اما در مقایسه با `zip_transform` چند تفاوت دارد.

| ویژگی                                       | حلقه دستی              | zip_transform |
| ------------------------------------------- | ---------------------- | ------------- |
| نیاز به مدیریت Index                        | بله                    | خیر           |
| ایجاد Container خروجی                       | معمولاً بله            | خیر           |
| Lazy Evaluation                             | خیر                    | بله           |
| قابلیت ترکیب با Views                       | محدودتر                | بسیار مناسب   |
| کنترل مستقیم جریان اجرا                     | بیشتر                  | کمتر          |
| امکان استفاده از Rangeهای غیر Random Access | نیازمند Iterator مناسب | بله           |

حلقه دستی زمانی مناسب‌تر است که منطق پردازش شامل کنترل پیچیده جریان اجرا، مدیریت خطا، توقف شرطی یا چند مرحله وابسته به State باشد.

در مقابل، `zip_transform` برای محاسبات مستقیم و مستقل روی عناصر متناظر انتخاب مناسبی است.

### **مقایسه با std::transform**

قابلیت `std::transform` از نسخه‌های قدیمی‌تر Standard Library وجود داشته و برای تبدیل عناصر یک یا چند محدوده استفاده می‌شود.

برای مثال، با استفاده از Overload چند ورودی آن می‌توان عملیاتی مشابه انجام داد:

```
#include <algorithm>
#include <functional>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    std::vector<int> result(a.size());

    std::transform(
        a.begin(),
        a.end(),
        b.begin(),
        result.begin(),
        std::plus<>{}
    );
}
```

این روش از نظر عملکردی می‌تواند کاملاً مناسب باشد.

اما تفاوت مهم آن با `zip_transform` این است که `std::transform` خروجی را در محدوده مقصد می‌نویسد و به همین دلیل باید فضای ذخیره‌سازی مناسبی برای نتیجه فراهم شود.

در مقابل، `zip_transform` یک View تنبل تولید می‌کند و محاسبه را به زمان دسترسی به عناصر موکول می‌کند.

بنابراین:

- برای تولید فوری نتایج در یک Container، `std::transform` گزینه مناسبی است.
- برای ساخت Pipelineهای مبتنی بر Ranges، `zip_transform` انتخاب مناسب‌تری است.
- برای پردازش Lazy و ترکیب با سایر Adaptors، `zip_transform` مزیت بیشتری دارد.

### **مقایسه با std::views::transform**

قابلیت `std::views::transform` یک Range را دریافت می‌کند و یک Callable را روی هر عنصر آن اجرا می‌کند.

در مقابل، `zip_transform` چند Range را دریافت می‌کند.

مثال با `transform`:

```
auto result = values | std::views::transform(
    [](int value) {
        return value * 2;
    }
);
```

مثال با `zip_transform`:

```
auto result = std::views::zip_transform(
    std::multiplies<>{},
    values,
    multipliers
);
```

به‌طور خلاصه:

- `transform` برای تبدیل عناصر یک Range مناسب است.
- `zip_transform` برای تبدیل هم‌زمان عناصر چند Range مناسب است.

### **مقایسه با Range-v3**

پیش از استاندارد شدن قابلیت‌های Zip در ++C23، کتابخانه Range-v3 امکانات مشابهی را ارائه می‌کرد.

این کتابخانه یکی از منابع مهم طراحی Ranges در Standard Library بوده است.

با معرفی `std::views::zip_transform`، توسعه‌دهندگان می‌توانند بدون وابستگی به کتابخانه خارجی از این قابلیت استفاده کنند.

البته ممکن است نسخه‌های مختلف Range-v3 از نظر جزئیات API، نوع خروجی یا قابلیت‌های جانبی تفاوت‌هایی با نسخه استاندارد داشته باشند.

بنابراین نباید فرض کرد که تمام جزئیات پیاده‌سازی یا رفتار دو کتابخانه کاملاً یکسان است.

## **نکات مهم عملکردی و پیچیدگی زمانی**

### **هزینه محاسباتی**

در حالت معمول، اگر \\(n\\) تعداد عناصر View خروجی باشد، پیمایش کامل آن به \\(n\\) بار اجرای Callable نیاز دارد.

بنابراین پیچیدگی زمانی کلی برابر است با:

\\[ O(n) \\]

البته این پیچیدگی فقط هزینه پیمایش و فراخوانی Callable را در نظر می‌گیرد.

اگر خود تابع عملیات پیچیده‌ای انجام دهد، هزینه آن نیز باید به محاسبه اضافه شود.

برای مثال، اگر هر فراخوانی Callable دارای پیچیدگی \\(O(k)\\) باشد، پیچیدگی کل تقریباً برابر خواهد بود با:

\\[ O(nk) \\]

### **هزینه حافظه**

از آنجا که `zip_transform` نتیجه‌ها را به‌صورت خودکار ذخیره نمی‌کند، معمولاً به حافظه اضافی متناسب با تعداد عناصر نیاز ندارد.

پیچیدگی حافظه خود View معمولاً ثابت و متناسب با تعداد Rangeهای ورودی است.

به‌صورت مفهومی:

\\[ O(m) \\]

که در آن \\(m\\) تعداد Rangeهای ورودی است.

این مقدار شامل حافظه موردنیاز برای نگهداری وضعیت View و Iteratorهای مرتبط می‌شود و حافظه‌ای را که خود Containerهای ورودی مصرف می‌کنند، شامل نمی‌شود.

### **اهمیت Side Effectها**

از آنجا که محاسبه عناصر هنگام Dereference انجام می‌شود، نباید فرض کنیم که هر عنصر دقیقاً یک بار محاسبه خواهد شد.

برای مثال:

```
auto result = std::views::zip_transform(
    [](int value) {
        std::cout << "Called\n";
        return value * 2;
    },
    values
);

auto it = result.begin();

int first = *it;
int second = *it;
```

در این مثال، ممکن است پیام `Called` دو بار نمایش داده شود.

این موضوع به این دلیل است که Dereference مجدد الزاماً از نتیجه قبلی استفاده نمی‌کند.

بنابراین برای طراحی Callable مناسب بهتر است:

- از تغییر State خارجی اجتناب شود.
- از عملیات ورودی و خروجی غیرضروری پرهیز شود.
- محاسبات تا حد امکان Deterministic باشند.
- در صورت نیاز به ذخیره نتایج، View به Container تبدیل شود.

### **تأثیر Cache و تکرار محاسبات**

در بعضی سناریوها، استفاده مکرر از یک View ممکن است باعث انجام دوباره محاسبات شود.

برای مثال:

```
auto result = std::views::zip_transform(
    expensiveCalculation,
    a,
    b
);
```

اگر چند الگوریتم مختلف روی `result` اجرا شوند، ممکن است محاسبات `expensiveCalculation` چندین بار تکرار شوند.

در چنین شرایطی، اگر هزینه محاسبات بالا باشد، Materialize کردن نتیجه می‌تواند عملکرد بهتری داشته باشد.

```
auto cached = result
    | std::ranges::to<std::vector>();
```

پس از این مرحله، عملیات بعدی روی `cached` به‌جای اجرای دوباره Callable از داده‌های ذخیره‌شده استفاده می‌کنند.

## **محدودیت‌ها و مواردی که باید در نظر گرفت**

### **وابستگی به عمر Rangeهای ورودی**

یکی از مهم‌ترین نکات هنگام استفاده از Viewها، مدیریت Lifetime است.

اگر View از یک Container موجود ساخته شود، معمولاً View به‌جای کپی کردن تمام عناصر، از داده‌های همان Container استفاده می‌کند.

بنابراین Container باید تا زمان استفاده از View معتبر باقی بماند.

مثال خطرناک:

```
#include <ranges>
#include <vector>

auto createView() {
    std::vector<int> values{1, 2, 3};

    return std::views::zip_transform(
        [](int value) {
            return value * 2;
        },
        values
    );
}
```

در این مثال، Container محلی پس از خروج از تابع از بین می‌رود.

View بازگشتی دیگر نباید برای دسترسی به داده‌های آن Container استفاده شود.

نمونه صحیح‌تر:

```
#include <ranges>
#include <vector>

auto createView(std::vector<int>& values) {
    return std::views::zip_transform(
        [](int value) {
            return value * 2;
        },
        values
    );
}
```

در این حالت، مسئولیت Lifetime داده‌های اصلی بر عهده فراخواننده است.

### **استفاده از Temporary Rangeها**

استاندارد Ranges برای مدیریت Rangeهای موقت از مفهوم `viewable_range` استفاده می‌کند.

برای مثال، در برخی شرایط ارسال یک Temporary Container به View مجاز است، اما نحوه مدیریت Lifetime آن به نوع Range و مالکیت آن بستگی دارد.

باید توجه داشت که View تولیدشده الزاماً داده‌های اصلی را کپی نمی‌کند و ممکن است مالکیت یک Range موقت را از طریق یک View مالک مانند `owning_view` حفظ کند.

با این حال، اگر یک Range غیرمالک یا Reference-based در اختیار View قرار گیرد، Lifetime آن همچنان اهمیت دارد.

در برنامه‌های پیچیده، بهتر است مالکیت داده‌ها و Lifetime تمام Rangeهای ورودی به‌صورت صریح طراحی شوند.

### **محدودیت در پردازش هم‌زمان**

خود `zip_transform` تضمین نمی‌کند که عناصر به‌صورت Parallel پردازش شوند.

این قابلیت یک View برای ارائه نتایج تنبل است و نباید آن را با الگوریتم‌های Parallel اشتباه گرفت.

اگر به پردازش موازی نیاز داریم، باید از ابزارهای مناسب مانند Parallel Algorithms یا کتابخانه‌های تخصصی استفاده کنیم.

همچنین وجود Side Effect در Callable می‌تواند باعث مشکلات Thread Safety شود.

### **عدم تضمین اجرای فقط یک‌باره Callable**

همان‌طور که قبلاً توضیح داده شد، هر بار Dereference می‌تواند باعث اجرای مجدد Callable شود.

بنابراین اگر تابع دارای هزینه بالا یا Side Effect باشد، ممکن است عملکرد و رفتار برنامه با انتظار اولیه متفاوت باشد.

### **محدودیت‌های انواع ورودی**

تمام Rangeها برای استفاده در `zip_transform` مناسب نیستند.

هر Range باید شرایط `input_range` و تبدیل شدن به View را داشته باشد.

همچنین اگر به قابلیت‌هایی مانند Random Access نیاز داشته باشیم، باید تمام Rangeهای ورودی از نظر ویژگی‌های Iterator شرایط مناسب را داشته باشند.

برای مثال، ترکیب یک `std::vector` با یک Range صرفاً Input، باعث نمی‌شود خروجی حتماً Random Access باشد.

### **محدودیت‌های نوع خروجی**

Callable باید نوع خروجی معتبری تولید کند.

خروجی `void` قابل استفاده نیست.

همچنین اگر خروجی Reference باشد، باید Lifetime شیئی که Reference به آن اشاره می‌کند معتبر باقی بماند.

برای مثال، بازگرداندن Reference به یک متغیر محلی اشتباه است:

```
auto result = std::views::zip_transform(
    [](int value) -> const int& {
        int temporary = value * 2;
        return temporary;
    },
    values
);
```

این تابع Reference به یک متغیر محلی برمی‌گرداند که پس از پایان اجرای Lambda از بین می‌رود.

بنابراین استفاده از چنین Referenceای رفتار تعریف‌نشده ایجاد می‌کند.

### **ملاحظات مربوط به Const**

قابلیت استفاده از View در حالت `const` به شرایط Callable و Viewهای داخلی وابسته است.

به‌طور کلی، برای استفاده از نسخه `const` باید:

- View داخلی امکان Range بودن در حالت Const را داشته باشد.
- Callable در حالت Const قابل فراخوانی باشد.
- شرایط مربوط به `regular_invocable` برای نسخه Const برقرار باشند.

برای مثال:

```
auto result = std::views::zip_transform(
    [](int value) {
        return value * 2;
    },
    values
);
```

این Lambda معمولی دارای `operator() const` است و در بسیاری از شرایط امکان استفاده از View در حالت Const را فراهم می‌کند.

اما Callableهای دارای State قابل تغییر یا انواعی که فقط در حالت غیر Const قابل فراخوانی هستند، ممکن است این قابلیت را محدود کنند.

## **رفتار خاص در فراخوانی بدون Range**

یکی از جزئیات کمتر شناخته‌شده استاندارد، امکان فراخوانی `std::views::zip_transform` تنها با یک Callable است.

برای مثال:

```
auto empty = std::views::zip_transform(
    [] {
        return 42;
    }
);
```

در این حالت، به‌جای ساخت یک `zip_transform_view` معمولی، یک Empty View از نوع مناسب تولید می‌شود.

شرایط این حالت عبارت‌اند از:

- Callable باید `move_constructible` باشد.
- Callable باید `regular_invocable` باشد.
- نوع خروجی آن باید یک Object Type معتبر باشد.

از آنجا که هیچ Range ورودی وجود ندارد، خروجی نیز هیچ عنصری نخواهد داشت.

این رفتار بخشی از مشخصات استاندارد است، اما در برنامه‌نویسی روزمره کاربرد محدودی دارد.

## **مقایسه با قابلیت‌های مرتبط در ++C23**

قابلیت‌های `zip_transform` بخشی از مجموعه امکانات جدید Ranges در ++C23 است.

شناخت قابلیت‌های نزدیک به آن کمک می‌کند ابزار مناسب‌تری برای هر سناریو انتخاب کنیم.

| قابلیت                           | کاربرد                                    |
| -------------------------------- | ----------------------------------------- |
| `std::views::zip`                | ترکیب عناصر متناظر چند Range              |
| `std::views::zip_transform`      | اجرای Callable روی عناصر متناظر چند Range |
| `std::views::adjacent`           | تولید گروه‌هایی از عناصر مجاور یک Range   |
| `std::views::adjacent_transform` | اجرای Callable روی گروه‌های عناصر مجاور   |
| `std::views::transform`          | تبدیل عناصر یک Range                      |
| `std::views::filter`             | فیلتر کردن عناصر یک Range                 |
| `std::views::enumerate`          | پیمایش همراه با Index                     |

برای مثال، اگر هدف محاسبه میانگین دو عنصر مجاور در یک Range باشد، `adjacent_transform` ابزار مناسب‌تری است.

اما اگر هدف جمع کردن عناصر دو Range مستقل باشد، `zip_transform` انتخاب طبیعی‌تری خواهد بود.

## **پشتیبانی کامپایلرها و سازگاری**

### **وضعیت استاندارد**

قابلیت `std::views::zip_transform` بخشی از Standard Library در ++C23 است.

این قابلیت از طریق پیشنهاد استانداردسازی زیر معرفی شده است:

`P2321R2 - zip`

برای استفاده از آن باید کامپایلر و کتابخانه استاندارد مورد استفاده، پیاده‌سازی مناسبی از قابلیت‌های Ranges مربوط به ++C23 داشته باشند.

### **بررسی Feature Test Macro**

برای بررسی پشتیبانی کتابخانه استاندارد می‌توان از Feature Test Macro زیر استفاده کرد:

```
__cpp_lib_ranges_zip
```

نمونه:

```
#include <iostream>
#include <ranges>

int main() {
#ifdef __cpp_lib_ranges_zip
    std::cout << "Zip ranges are supported\n";
#else
    std::cout << "Zip ranges are not supported\n";
#endif
}
```

مقدار Feature Test Macro معرفی‌شده برای این مجموعه قابلیت‌ها برابر است با:

```
202110L
```

بنابراین می‌توان بررسی دقیق‌تری انجام داد:

```
#include <ranges>

#if defined(__cpp_lib_ranges_zip) && \
    __cpp_lib_ranges_zip >= 202110L

// Zip-related C++23 facilities are available.

#else

#error "The standard library does not provide the required zip facilities."

#endif
```

نکته مهم این است که Feature Test Macro مربوط به کتابخانه استاندارد است، نه قابلیت زبان.

در نتیجه، صرفاً فعال کردن `-std=c++23` یا `/std:c++23` تضمین نمی‌کند که تمام قابلیت‌های `zip_transform` در نسخه کتابخانه استاندارد مورد استفاده پیاده‌سازی شده باشند.

### **تنظیم استاندارد در GCC و Clang**

برای استفاده از ++C23 در GCC می‌توان از دستور زیر استفاده کرد:

```
g++ -std=c++23 -O2 -Wall -Wextra main.cpp -o main
```

در Clang نیز می‌توان از دستور زیر استفاده کرد:

```
clang++ -std=c++23 -O2 -Wall -Wextra main.cpp -o main
```

پشتیبانی واقعی به نسخه Compiler و Standard Library متصل به آن وابسته است.

### **تنظیم استاندارد در MSVC**

در Microsoft Visual C++ می‌توان از گزینه زیر استفاده کرد:

```
cl /std:c++latest /EHsc main.cpp
```

یا در نسخه‌هایی که گزینه استاندارد موردنظر را ارائه می‌کنند، از تنظیمات مناسب استاندارد ++C23 استفاده کرد.

باید توجه داشت که پشتیبانی MSVC از زبان و پشتیبانی STL آن دو موضوع مرتبط اما مستقل هستند.

بنابراین بهتر است علاوه بر انتخاب استاندارد، Feature Test Macro نیز بررسی شود.

## **اشتباهات رایج هنگام استفاده**

### **تصور اینکه خروجی یک Container است**

یکی از اشتباهات رایج این است که خروجی `zip_transform` را با یک `std::vector` یکسان در نظر بگیریم.

```
auto result = std::views::zip_transform(
    std::plus<>{},
    a,
    b
);
```

در این مثال، `result` یک View است، نه یک Container.

بنابراین اگر به ذخیره دائمی نتایج نیاز داریم باید از عملیات تبدیل به Container استفاده کنیم.

### **فرض برابر بودن اندازه Rangeها**

برخی توسعه‌دهندگان تصور می‌کنند که تمام Rangeهای ورودی باید اندازه یکسانی داشته باشند.

این تصور صحیح نیست.

اگر اندازه Rangeها متفاوت باشد، View خروجی به‌اندازه کوتاه‌ترین Range خواهد بود.

اگر برابر بودن اندازه‌ها برای منطق برنامه ضروری است، باید این شرط را جداگانه بررسی کنیم.

### **استفاده از Callable دارای Side Effect**

از آنجا که تابع ممکن است چند بار برای یک عنصر اجرا شود، استفاده از Callableهایی که State خارجی را تغییر می‌دهند می‌تواند مشکل‌ساز باشد.

بهتر است Callableهای مورد استفاده در این View تا حد امکان Pure باشند.

### **بازگرداندن Reference نامعتبر**

اگر Callable یک Reference به متغیر محلی یا شیئی با Lifetime کوتاه برگرداند، دسترسی به خروجی می‌تواند به رفتار تعریف‌نشده منجر شود.

در صورت نیاز، بهتر است مقدار را برگردانیم یا از داده‌ای با Lifetime معتبر Reference ارائه کنیم.

### **انتظار اجرای موازی**

`zip_transform` عملیات را به‌صورت Lazy انجام می‌دهد، اما این موضوع به معنای Parallel Execution نیست.

برای اجرای موازی باید از امکانات دیگری استفاده کنیم.

### **فراموش کردن Lifetime داده‌ها**

Viewهای مبتنی بر Containerهای خارجی معمولاً مالک داده‌های آن Containerها نیستند.

اگر Container از بین برود، View وابسته به آن نیز ممکن است نامعتبر شود.

این موضوع هنگام بازگرداندن View از تابع یا نگهداری آن در اعضای کلاس اهمیت ویژه‌ای دارد.

### **استفاده از std::views::transform به‌جای zip_transform**

گاهی توسعه‌دهندگان ابتدا `zip` را تولید می‌کنند و سپس روی Tupleهای آن `transform` انجام می‌دهند.

این روش معتبر است، اما اگر هدف تنها ارسال عناصر متناظر به یک تابع باشد، `zip_transform` معمولاً ساده‌تر و مستقیم‌تر است.

## **ملاحظات طراحی و انتخاب ابزار مناسب**

انتخاب `zip_transform` باید بر اساس نیاز واقعی برنامه انجام شود.

این قابلیت زمانی انتخاب مناسبی است که:

- چند Range مستقل داشته باشیم.
- عناصر موقعیت‌های متناظر باید با هم پردازش شوند.
- خروجی هر مرحله یک مقدار محاسبه‌شده باشد.
- نیازی به ذخیره فوری تمام نتایج نداشته باشیم.
- بخواهیم از ترکیب Range Adaptorها استفاده کنیم.
- منطق تبدیل را بتوانیم به یک Callable مستقل تبدیل کنیم.

اما در شرایط زیر ممکن است ابزار دیگری مناسب‌تر باشد:

- به کنترل دقیق جریان اجرا نیاز داشته باشیم.
- هر عنصر به State پیچیده و مشترکی وابسته باشد.
- محاسبات دارای Side Effectهای ضروری باشند.
- بخواهیم نتایج دقیقاً یک بار محاسبه و ذخیره شوند.
- به پردازش موازی نیاز داشته باشیم.
- اختلاف اندازه Rangeها باید باعث خطا شود، نه توقف زودتر پیمایش.

در چنین شرایطی می‌توان از حلقه‌های معمولی، `std::transform`، الگوریتم‌های Parallel یا یک Pipeline ترکیبی همراه با Materialization استفاده کرد.

## **جمع‌بندی نهایی**

قابلیت `std::views::zip_transform` یکی از امکانات کاربردی Ranges در ++C23 است که پردازش هم‌زمان چند Range را ساده‌تر و خواناتر می‌کند.

این قابلیت با دریافت یک Callable و چند Range، یک View تنبل تولید می‌کند که هر عنصر آن نتیجه اجرای Callable روی عناصر متناظر Rangeهای ورودی است.

مهم‌ترین ویژگی‌ها و نکات این قابلیت عبارت‌اند از:

- **Lazy Evaluation:** محاسبه نتایج هنگام دسترسی به عناصر انجام می‌شود.
- **عدم نیاز به Container موقت:** نتایج به‌صورت خودکار در حافظه ذخیره نمی‌شوند.
- **پشتیبانی از چند Range:** می‌توان تعداد دلخواهی Range را برای پردازش هم‌زمان ارسال کرد.
- **پشتیبانی از Referenceها:** امکان تغییر مستقیم عناصر اصلی در صورت بازگرداندن Reference معتبر وجود دارد.
- **ترکیب‌پذیری:** می‌توان آن را با قابلیت‌هایی مانند `filter`، `take`، `drop`، `enumerate` و `fold_left` ترکیب کرد.
- **توقف در کوتاه‌ترین Range:** تعداد عناصر خروجی برابر با کمترین طول Rangeهای ورودی است.
- **وابستگی به Iteratorهای ورودی:** قابلیت‌هایی مانند Random Access و Bidirectional بودن از ویژگی‌های Rangeهای داخلی تأثیر می‌گیرند.
- **عدم تضمین اجرای یک‌باره:** هر بار Dereference می‌تواند Callable را دوباره اجرا کند.
- **مدیریت Lifetime:** اعتبار داده‌های اصلی باید در طول استفاده از View حفظ شود.

در مجموع، `std::views::zip_transform` برای ساخت Pipelineهای پردازش داده و انجام محاسبات Element-wise در برنامه‌های مدرن ++C انتخاب بسیار مناسبی است.

با استفاده صحیح از این قابلیت می‌توان بسیاری از حلقه‌های تکراری و Containerهای موقت را حذف کرد و منطق پردازش داده را به شکلی Declarative، خوانا و قابل ترکیب پیاده‌سازی کرد.

## **منابع و مراجع**

- [cppreference - std::ranges::views::zip_transform](https://en.cppreference.com/w/cpp/ranges/zip_transform_view?utm_source=chatgpt.com)
- [WG21 P2321R2 - zip, zip_transform, adjacent and adjacent_transform](https://www.open-std.org/JTC1/SC22/WG21/docs/papers/2021/p2321r2.html?utm_source=chatgpt.com)
- [C++23 Working Draft - Zip Transform View](https://timsong-cpp.github.io/cppwp/std23/ranges?utm_source=chatgpt.com)
- [cppreference - C++23 Compiler Support](https://en.cppreference.com/w/cpp/compiler_support/23?utm_source=chatgpt.com)
- [cppreference - std::ranges::views::zip](https://en.cppreference.com/w/cpp/ranges/zip_view?utm_source=chatgpt.com)
- [cppreference - std::ranges::views::transform](https://en.cppreference.com/w/cpp/ranges/transform_view?utm_source=chatgpt.com)
- [cppreference - std::ranges::fold_left](https://en.cppreference.com/w/cpp/algorithm/ranges/fold_left?utm_source=chatgpt.com)

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>