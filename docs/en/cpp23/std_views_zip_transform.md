<div align="center">

[🇺🇸 English](./std_views_zip_transform.md) | [🇮🇷 فارسی](../../fa/cpp23/std_views_zip_transform.md)

</div>

---

# **آموزش جامع&#x20;**`std::views::adjacent_transform`**&#x20;در ++C23**

## **مقدمه**

کتابخانه Ranges در ++C20 و ++C23 امکانات قدرتمندی برای پردازش دنباله‌ای از داده‌ها به سبک Declarative Programming فراهم کرده است. یکی از قابلیت‌های کاربردی این مجموعه، امکان پردازش هم‌زمان چند عنصر مجاور در یک Range و تولید نتیجه‌ای جدید از آن‌ها است.

در ++C23، قابلیت `std::views::adjacent_transform` به مجموعه Standard Views اضافه شده است تا بتوان عملیات‌هایی را که به چند عنصر متوالی نیاز دارند، بدون نوشتن حلقه‌های دستی و مدیریت مستقیم Iteratorها انجام داد.

برای مثال، فرض کنید دنباله‌ای از اعداد در اختیار داریم و می‌خواهیم اختلاف هر دو عنصر متوالی را محاسبه کنیم:

```
Input:  10, 15, 21, 30, 42
Output:  5,  6,  9, 12
```

در روش‌های سنتی، معمولاً باید دو Iterator یا دو اندیس را به‌صورت هم‌زمان مدیریت کنیم. اما با استفاده از `adjacent_transform` می‌توان این عملیات را مستقیماً به‌صورت یک View تعریف کرد.

این قابلیت به‌خصوص در پردازش داده‌های متوالی، محاسبات عددی، تحلیل تغییرات، پردازش سیگنال، بررسی پنجره‌های لغزان و الگوریتم‌هایی که به همسایگی عناصر وابسته هستند، کاربرد دارد.

## **معرفی قابلیت adjacent_transform**

قابلیت `std::views::adjacent_transform` بخشی از کتابخانه استاندارد Ranges در ++C23 است.

این قابلیت یک View ایجاد می‌کند که روی مجموعه‌ای از عناصر متوالی و مجاور یک Range حرکت می‌کند و برای هر گروه از عناصر، یک Callable را اجرا می‌کند.

تفاوت اصلی آن با الگوریتم‌هایی مانند `std::ranges::transform` این است که تابع Transform در اینجا به‌جای دریافت یک عنصر، مجموعه‌ای از عناصر مجاور را دریافت می‌کند.

برای نمونه، اگر اندازه پنجره برابر با ۳ باشد، عناصر ورودی به شکل زیر پردازش می‌شوند:

```
Input:
A, B, C, D, E

Windows:
(A, B, C)
(B, C, D)
(C, D, E)
```

در هر مرحله، سه عنصر متوالی به تابع موردنظر ارسال می‌شوند.

بنابراین، تعداد عناصر خروجی برای یک Range با اندازه `N` و پنجره با اندازه `K` برابر است با:

\\[ \max(N-K+1, 0) \\]

این قابلیت از نظر مفهومی ترکیبی از دو عملیات است:

- استخراج گروه‌های متوالی با اندازه ثابت.
- اعمال یک تابع روی هر گروه و تولید نتیجه.

## **محل قرارگیری در استاندارد**

قابلیت `std::views::adjacent_transform` در استاندارد ++C23 معرفی شده است.

این قابلیت با استفاده از مکانیزم‌های جدید Ranges طراحی شده و به خانواده `std::views` تعلق دارد.

برای استفاده از آن باید Header زیر را اضافه کنید:

```
#include <ranges>
```

همچنین برای استفاده از سایر امکانات موردنیاز برنامه، Headerهای مربوطه مانند `<vector>`، `<array>` و `<iostream>` را نیز اضافه کنید.

نکته مهم این است که `adjacent_transform` یک قابلیت استاندارد کتابخانه است، نه یک Keyword جدید در زبان ++C و نه یک Compiler Extension.

وجود پشتیبانی از ++C23 در Compiler به‌تنهایی تضمین نمی‌کند که تمام اجزای این قابلیت در نسخه مورد استفاده پیاده‌سازی شده باشند.

## **نحوه استفاده از adjacent_transform**

ساختار کلی استفاده از این قابلیت به شکل زیر است:

```
std::views::adjacent_transform<N>(function)
```

در این ساختار:

- پارامتر `N` اندازه هر پنجره یا تعداد عناصر مجاور مورد پردازش را مشخص می‌کند.
- پارامتر `function` تابعی است که روی عناصر هر پنجره اجرا می‌شود.
- نتیجه، یک `adjacent_transform_view` است که عناصر خروجی را به‌صورت Lazy تولید می‌کند.

برای مثال، اگر `N` برابر با ۲ باشد، تابع در هر مرحله دو عنصر متوالی دریافت می‌کند.

اگر `N` برابر با ۳ باشد، تابع در هر مرحله سه عنصر متوالی دریافت می‌کند.

نکته مهم این است که `N` یک Template Argument از نوع عدد صحیح در زمان کامپایل است و نمی‌توان اندازه پنجره را در زمان اجرا به‌صورت پویا تغییر داد.

### **مثال ساده با دو عنصر مجاور**

در مثال زیر اختلاف دو عنصر متوالی محاسبه می‌شود:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{10, 15, 21, 30, 42};

    auto differences = std::views::adjacent_transform<2>(
        [](int first, int second) {
            return second - first;
        }
    )(numbers);

    for (int value : differences) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
5 6 9 12
```

در این مثال، تابع Lambda دو پارامتر دریافت می‌کند و اختلاف عنصر دوم و اول را برمی‌گرداند.

مهم‌ترین نکته این است که تابع برای هر جفت عنصر مجاور اجرا می‌شود، نه برای تمام ترکیب‌های ممکن از دو عنصر.

### **مثال با سه عنصر مجاور**

در مثال بعدی، میانگین سه عنصر متوالی محاسبه می‌شود:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{10, 20, 30, 40, 50};

    auto averages = std::views::adjacent_transform<3>(
        [](int first, int second, int third) {
            return (first + second + third) / 3.0;
        }
    )(numbers);

    for (double value : averages) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
20 30 40
```

در این مثال، پنجره‌های پردازش به شکل زیر هستند:

```
(10, 20, 30)
(20, 30, 40)
(30, 40, 50)
```

هر پنجره به‌صورت مستقل به تابع Lambda ارسال می‌شود.

## **نحوه استفاده به‌صورت Pipeline**

یکی از مزیت‌های اصلی Ranges، قابلیت ترکیب Viewها با استفاده از Pipe Operator است.

در این روش می‌توان `adjacent_transform` را مستقیماً در یک Pipeline قرار داد:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{2, 4, 6, 8, 10};

    auto result = numbers
        | std::views::adjacent_transform<2>(
            [](int a, int b) {
                return a * b;
            }
        );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
8 24 48 80
```

در این مثال، ابتدا Range ورودی به `adjacent_transform` داده می‌شود و سپس تابع Lambda روی جفت‌های مجاور اجرا می‌شود.

### **دو شیوه فراخوانی Range Adaptor**

این قابلیت از دو شیوه اصلی استفاده پشتیبانی می‌کند.

**روش اول: ارسال مستقیم Range و تابع**

```
auto result = std::views::adjacent_transform<2>(
    numbers,
    [](int a, int b) {
        return a + b;
    }
);
```

**روش دوم: ساخت Range Adaptor Closure و استفاده از Pipe**

```
auto addAdjacent = std::views::adjacent_transform<2>(
    [](int a, int b) {
        return a + b;
    }
);

auto result = numbers | addAdjacent;
```

روش دوم زمانی مفید است که بخواهیم یک Transformation را تعریف کنیم و آن را روی چند Range مختلف به کار ببریم.

## **معرفی pairwise_transform**

کتابخانه استاندارد یک نام مستعار کاربردی برای حالت پردازش دو عنصر مجاور ارائه می‌کند:

```
std::views::pairwise_transform
```

این نام معادل زیر است:

```
std::views::adjacent_transform<2>
```

بنابراین، در مواردی که هدف پردازش جفت‌های متوالی است، می‌توان به‌جای مشخص کردن عدد ۲ از `pairwise_transform` استفاده کرد.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{3, 5, 8, 12, 17};

    auto result = values
        | std::views::pairwise_transform(
            [](int current, int next) {
                return next - current;
            }
        );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
2 3 4 5
```

استفاده از `pairwise_transform` خوانایی کد را افزایش می‌دهد، زیرا هدف پردازش جفت‌های مجاور را به‌صورت مستقیم بیان می‌کند.

## **مفهوم Sliding Window**

یکی از مفاهیم مهم برای درک رفتار `adjacent_transform`، مفهوم Sliding Window است.

در این روش، یک پنجره با اندازه ثابت روی عناصر Range حرکت می‌کند. در هر مرحله، پنجره یک عنصر به جلو منتقل می‌شود و عناصر جدیدی را در بر می‌گیرد.

برای مثال، اگر ورودی شامل عناصر زیر باشد:

```
10, 20, 30, 40, 50, 60
```

و اندازه پنجره برابر با ۳ باشد، پنجره‌ها به شکل زیر ساخته می‌شوند:

```
Step 1: 10, 20, 30
Step 2:     20, 30, 40
Step 3:         30, 40, 50
Step 4:             40, 50, 60
```

در هر مرحله، تنها یک عنصر به موقعیت بعدی منتقل می‌شود و دو عنصر دیگر با پنجره قبلی مشترک هستند.

این رفتار باعث می‌شود که عناصر میانی در چند پنجره مختلف حضور داشته باشند.

### **تفاوت Sliding Window با Chunking**

مفهوم Sliding Window نباید با تقسیم Range به بخش‌های مستقل یا Chunking اشتباه گرفته شود.

برای مقایسه، فرض کنید یک Range با شش عنصر داریم و اندازه هر گروه برابر با ۳ است.

در حالت Sliding Window:

```
(10, 20, 30)
(20, 30, 40)
(30, 40, 50)
(40, 50, 60)
```

اما در حالت Chunking:

```
(10, 20, 30)
(40, 50, 60)
```

در نتیجه:

- در Sliding Window، پنجره‌ها هم‌پوشانی دارند.
- در Chunking، هر عنصر فقط در یک گروه قرار می‌گیرد.
- در `adjacent_transform`، پردازش بر اساس پنجره‌های هم‌پوشان انجام می‌شود.

اگر هدف پردازش گروه‌های مستقل و بدون هم‌پوشانی باشد، `std::views::chunk` انتخاب مناسب‌تری است.

## **بررسی اندازه Range خروجی**

یکی از ویژگی‌های مهم `adjacent_transform`، اندازه خروجی آن است.

اگر اندازه Range ورودی برابر با `N` و اندازه پنجره برابر با `K` باشد، اندازه Range خروجی به شکل زیر محاسبه می‌شود:

\\[ S\_{\text{output}} = \begin{cases} S\_{\text{input}}-K+1 & S\_{\text{input}}\geq K\\\ 0 & S\_{\text{input}}\<K \end{cases} \\]

برای مثال:

| اندازه ورودی | اندازه پنجره | اندازه خروجی |
| ------------ | ------------ | ------------ |
| 5            | 2            | 4            |
| 5            | 3            | 3            |
| 5            | 4            | 2            |
| 5            | 5            | 1            |
| 5            | 6            | 0            |
| 2            | 3            | 0            |

بنابراین، اگر تعداد عناصر ورودی کمتر از اندازه پنجره باشد، هیچ خروجی‌ای تولید نمی‌شود.

### **مثال برای Range کوچک‌تر از پنجره**

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20};

    auto result = values
        | std::views::adjacent_transform<3>(
            [](int a, int b, int c) {
                return a + b + c;
            }
        );

    std::cout << std::ranges::distance(result);
}
```

خروجی:

```
0
```

در این مثال، چون تنها دو عنصر وجود دارد و پنجره به سه عنصر نیاز دارد، View خروجی خالی است.

نکته مهم این است که در این شرایط تابع Lambda اجرا نمی‌شود.

## **مفهوم Lazy Evaluation**

یکی از مهم‌ترین ویژگی‌های `adjacent_transform`، استفاده از Lazy Evaluation است.

این قابلیت به این معناست که هنگام ساخت View، تمام نتایج محاسبه نمی‌شوند.

برای مثال:

```
auto result = values
    | std::views::adjacent_transform<2>(
        [](int a, int b) {
            return a + b;
        }
    );
```

در این مرحله فقط یک View ساخته شده است و هنوز نیازی به محاسبه تمام مقادیر خروجی وجود ندارد.

محاسبه هر مقدار زمانی انجام می‌شود که Iterator مربوط به View آن مقدار را Dereference کند.

برای مثال:

```
for (int value : result) {
    std::cout << value << '\n';
}
```

در این مرحله، مقادیر موردنیاز به‌صورت تدریجی محاسبه می‌شوند.

### **مزایای Lazy Evaluation**

استفاده از این روش مزایای متعددی دارد:

- جلوگیری از تخصیص حافظه برای یک Container خروجی.
- امکان ترکیب چند Transformation بدون ساخت نتایج میانی.
- پردازش تدریجی داده‌ها.
- امکان استفاده از View در Pipelineهای پیچیده.
- محاسبه تنها مقادیری که واقعاً موردنیاز هستند.

### **نکته مهم درباره محاسبه مجدد**

از آنجا که `adjacent_transform` یک View تنبل است، نباید فرض کنیم که نتیجه هر پنجره الزاماً فقط یک‌بار محاسبه می‌شود.

اگر یک Iterator چندین بار Dereference شود، تابع Transformation ممکن است چندین بار برای همان پنجره اجرا شود.

بنابراین، بهتر است تابع مورد استفاده تا حد امکان Pure Function باشد؛ یعنی برای ورودی یکسان، نتیجه یکسان تولید کند و به Side Effect وابسته نباشد.

برای مثال، تابع زیر انتخاب مناسبی است:

```
[](int a, int b) {
    return a + b;
}
```

اما تابعی که در هر بار اجرا یک شمارنده خارجی را تغییر می‌دهد، ممکن است رفتار غیرمنتظره‌ای ایجاد کند.

## **الزامات و محدودیت‌های Type System**

قابلیت `adjacent_transform` برای Range و Callable ورودی، محدودیت‌های مشخصی دارد.

شناخت این محدودیت‌ها برای استفاده حرفه‌ای از آن ضروری است.

### **الزام Forward Range**

یکی از مهم‌ترین الزامات این قابلیت، پشتیبانی Range ورودی از مفهوم `std::ranges::forward_range` است.

به بیان ساده، Range باید امکان پیمایش چندباره و نگهداری Iteratorهای معتبر در چارچوب الزامات Forward Iterator را فراهم کند.

به همین دلیل، هر Rangeای که فقط مفهوم `input_range` را پیاده‌سازی کرده باشد، الزاماً برای `adjacent_transform` مناسب نیست.

برای مثال، یک Stream مبتنی بر `std::istream_view` معمولاً فقط Input Range است و نمی‌توان آن را مستقیماً به‌عنوان ورودی این View استفاده کرد.

```
#include <ranges>
#include <sstream>

int main() {
    std::istringstream stream{"10 20 30 40"};

    auto values = std::ranges::istream_view<int>(stream);

    // This does not satisfy the forward_range requirement.
    // auto result = values
    //     | std::views::adjacent_transform<2>(
    //         [](int a, int b) { return a + b; }
    //     );
}
```

برای استفاده از داده‌های چنین Streamهایی، می‌توان ابتدا آن‌ها را در یک Container ذخیره کرد:

```
#include <iostream>
#include <ranges>
#include <sstream>
#include <vector>

int main() {
    std::istringstream stream{"10 20 30 40"};

    std::vector<int> values;

    for (int value : std::ranges::istream_view<int>(stream)) {
        values.push_back(value);
    }

    auto result = values
        | std::views::adjacent_transform<2>(
            [](int a, int b) {
                return b - a;
            }
        );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
10 10 10
```

در این روش، هزینه ذخیره‌سازی داده‌ها در Container وجود دارد، اما در مقابل یک Range قابل پیمایش مجدد در اختیار خواهیم داشت.

### **الزام N بزرگ‌تر از صفر**

پارامتر اندازه پنجره باید در حالت معمول استفاده از `adjacent_transform_view` بزرگ‌تر از صفر باشد.

به‌صورت مفهومی:

```
std::views::adjacent_transform<1>
std::views::adjacent_transform<2>
std::views::adjacent_transform<3>
```

همگی حالت‌های معتبر هستند.

اما مقدار صفر با محدودیت‌های خاصی در Standard تعریف شده و نباید آن را یک پنجره معمولی در نظر گرفت.

طبق مشخصات استاندارد، برای Forward Range، حالت صفر معادل `views::zip_transform` است. همچنین محدودیت‌های مربوط به نسخه اصلاح‌شده استاندارد باید در نظر گرفته شوند.

در طراحی معمول برنامه، اگر هدف تبدیل تک‌تک عناصر است، استفاده از `std::views::transform` انتخاب واضح‌تر و مناسب‌تری خواهد بود.

### **الزامات Callable**

تابع ورودی باید بتواند دقیقاً `N` آرگومان دریافت کند.

برای نمونه، در حالت زیر:

```
std::views::adjacent_transform<3>(
    [](int a, int b, int c) {
        return a + b + c;
    }
);
```

تابع باید با سه Reference مربوط به عناصر ورودی قابل فراخوانی باشد.

از نظر مفهومی، استاندارد از Constraintهای `std::regular_invocable` برای بررسی Callable استفاده می‌کند.

همچنین نوع خروجی تابع نباید `void` باشد و باید قابلیت استفاده به‌عنوان Reference یا Value را داشته باشد.

نکته مهم این است که پارامترهای تابع الزاماً نباید از نوع `int` باشند. تابع می‌تواند از انواع مناسب دیگری مانند Reference، `const` Reference یا Generic Parameter استفاده کند.

برای مثال:

```
auto result = values
    | std::views::adjacent_transform<2>(
        [](const auto& first, const auto& second) {
            return first < second;
        }
    );
```

این روش به‌خصوص برای Rangeهایی که عناصر آن‌ها از انواع پیچیده تشکیل شده‌اند، مفید است.

## **بررسی نحوه پیاده‌سازی و ساختار داخلی**

برای درک بهتر رفتار این قابلیت، بررسی ساختار داخلی `std::ranges::adjacent_transform_view` مفید است.

این Class Template در Namespace زیر تعریف شده است:

```
std::ranges::adjacent_transform_view
```

ساختار کلی آن به شکل زیر است:

```
namespace std::ranges {

    template<
        forward_range V,
        std::move_constructible F,
        std::size_t N
    >
    requires
        view<V> &&
        (N > 0) &&
        std::is_object_v<F> &&
        std::regular_invocable<
            F&,
            std::ranges::range_reference_t<V>
        > &&
        std::invocable<F&>
    class adjacent_transform_view;

}
```

کد بالا صرفاً نمایشی از ساختار Template است و Declaration کامل استاندارد محسوب نمی‌شود؛ زیرا برای `N` آرگومان، نوع Reference متناظر باید به‌صورت تکراری در Constraint قرار گیرد.

محدودیت‌های اصلی Template عبارت‌اند از:

- `V` باید یک `view` و در عین حال یک `forward_range` باشد.
- نوع `F` باید قابلیت Move Construction داشته باشد.
- مقدار `N` باید بزرگ‌تر از صفر باشد.
- نوع `F` باید یک Object Type باشد.
- تابع باید با `N` آرگومان از نوع Reference عناصر Range قابل فراخوانی باشد.
- نوع خروجی تابع باید قابل استفاده باشد و `void` نباشد.

### **ارتباط با adjacent_view**

یکی از نکات جالب پیاده‌سازی این قابلیت، استفاده داخلی از `std::ranges::adjacent_view` است.

در واقع، `adjacent_transform_view` از یک View داخلی برای مدیریت پنجره‌های مجاور استفاده می‌کند.

به‌صورت مفهومی، ساختار پردازش چنین است:

```
Original Range
      |
      v
adjacent_view<N>
      |
      v
Adjacent Elements
      |
      v
Transformation Function
      |
      v
Result View
```

برای مثال، اگر ورودی شامل پنج عنصر و اندازه پنجره برابر با ۳ باشد، ابتدا پنجره‌های مجاور ایجاد می‌شوند و سپس تابع Transformation روی هر پنجره اجرا می‌شود.

نکته مهم این است که این ساختار مفهومی نباید با ساخت یک Container شامل تمام پنجره‌ها اشتباه گرفته شود. Viewها برای ارائه دسترسی تنبل به داده‌ها طراحی شده‌اند.

## **نحوه مدیریت Callable**

تابع ورودی در `adjacent_transform` به‌عنوان بخشی از وضعیت View نگهداری می‌شود.

این موضوع چند نکته مهم دارد.

### **امکان استفاده از Lambda**

یکی از رایج‌ترین روش‌های استفاده از این قابلیت، Lambda Expression است:

```
auto result = values
    | std::views::adjacent_transform<2>(
        [](int a, int b) {
            return a * b;
        }
    );
```

### **استفاده از Function Object**

در برنامه‌های بزرگ‌تر می‌توان منطق Transformation را در یک Function Object قرار داد:

```
#include <iostream>
#include <ranges>
#include <vector>

struct Difference {
    int operator()(int first, int second) const {
        return second - first;
    }
};

int main() {
    std::vector<int> values{10, 15, 25, 40, 60};

    auto result = values
        | std::views::adjacent_transform<2>(Difference{});

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
5 10 15 20
```

استفاده از Function Object زمانی مناسب است که منطق پردازش پیچیده باشد یا قرار باشد در چند قسمت برنامه استفاده شود.

### **استفاده از Generic Lambda**

در صورت نیاز به پشتیبانی از انواع مختلف داده‌ها، می‌توان از Generic Lambda استفاده کرد:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> values{
        1.5, 2.5, 4.0, 7.0, 11.5
    };

    auto result = values
        | std::views::adjacent_transform<2>(
            [](const auto& a, const auto& b) {
                return b - a;
            }
        );

    for (auto value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
1 1.5 3 4.5
```

این روش انعطاف بیشتری در طراحی Generic Code فراهم می‌کند.

## **کاربردهای عملی adjacent_transform**

این قابلیت در بسیاری از الگوریتم‌هایی که به مقایسه یا پردازش عناصر مجاور نیاز دارند، مفید است.

### **محاسبه اختلاف عناصر متوالی**

یکی از کاربردهای رایج، محاسبه تغییرات بین مقادیر متوالی است.

فرض کنید داده‌های مربوط به دمای یک محیط در اختیار داریم:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> temperatures{
        18.5, 19.0, 20.5, 20.0, 22.5
    };

    auto changes = temperatures
        | std::views::pairwise_transform(
            [](double previous, double current) {
                return current - previous;
            }
        );

    for (double change : changes) {
        std::cout << change << '\n';
    }
}
```

خروجی:

```
0.5
1.5
-0.5
2.5
```

این روش برای تحلیل تغییرات زمانی، بررسی روند داده‌ها و محاسبه نرخ تغییر در فاصله‌های متوالی کاربرد دارد.

### **محاسبه Moving Average**

یکی دیگر از کاربردهای مهم، محاسبه Moving Average با پنجره ثابت است.

در مثال زیر، میانگین سه عنصر متوالی محاسبه می‌شود:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> values{
        10, 20, 30, 40, 50, 60
    };

    auto movingAverage = values
        | std::views::adjacent_transform<3>(
            [](double a, double b, double c) {
                return (a + b + c) / 3.0;
            }
        );

    for (double value : movingAverage) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
20 30 40 50
```

در این مثال، هر خروجی میانگین سه مقدار متوالی را نشان می‌دهد.

البته اگر اندازه پنجره در زمان اجرا تغییر کند یا بخواهیم Moving Average را روی پنجره‌های بسیار بزرگ و متغیر محاسبه کنیم، ممکن است استفاده از الگوریتم اختصاصی با نگهداری مجموع لغزان کارآمدتر باشد.

### **بررسی صعودی بودن عناصر متوالی**

می‌توان از این قابلیت برای بررسی رابطه بین عناصر مجاور نیز استفاده کرد.

برای مثال، بررسی می‌کنیم که آیا عنصر بعدی از عنصر قبلی بزرگ‌تر است یا خیر:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        2, 5, 8, 4, 9
    };

    auto comparisons = values
        | std::views::pairwise_transform(
            [](int previous, int current) {
                return previous < current;
            }
        );

    for (bool result : comparisons) {
        std::cout << std::boolalpha << result << ' ';
    }
}
```

خروجی:

```
true true false true
```

این روش برای شناسایی تغییر جهت روند، تشخیص نقاط نزولی و بررسی ترتیب نسبی داده‌ها مفید است.

### **محاسبه مشتق تقریبی**

در محاسبات عددی می‌توان از اختلاف مقادیر متوالی برای تقریب مشتق استفاده کرد.

اگر دو مجموعه شامل مقادیر `x` و `y` داشته باشیم، می‌توان نرخ تغییر `y` نسبت به `x` را برای نقاط متوالی محاسبه کرد:

\\[ \frac{\Delta y}{\Delta x} = \frac{y\_{i+1}-y_i}{x\_{i+1}-x_i} \\]

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

struct Point {
    double x;
    double y;
};

int main() {
    std::vector<Point> points{
        {0.0, 0.0},
        {1.0, 2.0},
        {2.0, 5.0},
        {4.0, 9.0}
    };

    auto slopes = points
        | std::views::pairwise_transform(
            [](const Point& first, const Point& second) {
                return (second.y - first.y)
                     / (second.x - first.x);
            }
        );

    for (double slope : slopes) {
        std::cout << slope << ' ';
    }
}
```

خروجی:

```
2 3 2
```

در این مثال، شیب بین نقاط متوالی محاسبه می‌شود.

توجه داشته باشید که اگر مختصات `x` دو نقطه برابر باشند، تقسیم بر صفر رخ می‌دهد. در کاربردهای واقعی باید این حالت مدیریت شود.

### **محاسبه حاصل‌ضرب چند عنصر متوالی**

برای پردازش‌های عددی می‌توان از پنجره‌های بزرگ‌تر نیز استفاده کرد:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        1, 2, 3, 4, 5
    };

    auto products = values
        | std::views::adjacent_transform<3>(
            [](int a, int b, int c) {
                return a * b * c;
            }
        );

    for (int value : products) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
6 24 60
```

این الگو برای محاسبات مبتنی بر گروه‌های متوالی با اندازه ثابت مناسب است.

## **ترکیب adjacent_transform با سایر Views**

یکی از مزایای مهم Ranges، قابلیت ترکیب Viewهای مختلف در یک Pipeline است.

`adjacent_transform` نیز می‌تواند با Viewهای دیگر ترکیب شود.

### **ترکیب با filter**

فرض کنید فقط اختلاف‌های مثبت بین عناصر متوالی موردنیاز باشند.

می‌توان ابتدا اختلاف‌ها را محاسبه کرد و سپس فقط مقادیر مثبت را نگه داشت:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        10, 15, 12, 20, 18, 30
    };

    auto result = values
        | std::views::pairwise_transform(
            [](int a, int b) {
                return b - a;
            }
        )
        | std::views::filter(
            [](int difference) {
                return difference > 0;
            }
        );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
5 8 12
```

در این Pipeline ابتدا Transformation انجام می‌شود و سپس نتیجه‌ها براساس Predicate فیلتر می‌شوند.

### **ترکیب با transform**

می‌توان نتیجه `adjacent_transform` را دوباره با `transform` پردازش کرد.

برای مثال، اختلاف مطلق بین عناصر متوالی را محاسبه می‌کنیم:

```
#include <cstdlib>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        10, 15, 12, 20
    };

    auto result = values
        | std::views::pairwise_transform(
            [](int a, int b) {
                return b - a;
            }
        )
        | std::views::transform(
            [](int difference) {
                return std::abs(difference);
            }
        );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
5 3 8
```

این روش امکان جداسازی مراحل مختلف پردازش را فراهم می‌کند.

### **ترکیب با take**

اگر فقط چند نتیجه اول موردنیاز باشند، می‌توان از `std::views::take` استفاده کرد:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        1, 3, 6, 10, 15, 21, 28
    };

    auto result = values
        | std::views::pairwise_transform(
            [](int a, int b) {
                return b - a;
            }
        )
        | std::views::take(3);

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
2 3 4
```

به دلیل Lazy Evaluation، تنها نتایج موردنیاز برای پیمایش تولید می‌شوند.

البته این موضوع به معنای تضمین عدم اجرای مجدد تابع در صورت Dereference مجدد Iterator نیست.

### **تبدیل نتیجه به Container**

اگر به‌جای View به یک Container واقعی نیاز داشته باشیم، می‌توان نتیجه را Materialize کرد.

در ++C23، قابلیت `std::ranges::to` برای تبدیل Range به Container در دسترس است:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        10, 20, 30, 40, 50
    };

    auto result = values
        | std::views::pairwise_transform(
            [](int a, int b) {
                return b - a;
            }
        );

    auto output = std::ranges::to<std::vector>(result);

    for (int value : output) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
10 10 10 10
```

در این حالت، نتایج در یک `std::vector` ذخیره می‌شوند و دیگر وابسته به محاسبه Lazy در View نیستند.

این روش زمانی مناسب است که قرار باشد خروجی چندین بار استفاده شود یا نیاز به نگهداری مستقل نتایج داشته باشیم.

## **تفاوت adjacent_transform با قابلیت‌های مشابه**

در کتابخانه Ranges چند قابلیت وجود دارند که از نظر ظاهری یا کاربردی به `adjacent_transform` نزدیک هستند؛ اما هرکدام هدف متفاوتی دارند.

شناخت این تفاوت‌ها باعث می‌شود ابزار مناسب‌تری برای طراحی الگوریتم انتخاب کنیم.

### **مقایسه با std::views::transform**

قابلیت `std::views::transform` برای اعمال یک تابع روی تک‌تک عناصر Range طراحی شده است.

در مقابل، `adjacent_transform` یک تابع را روی چند عنصر متوالی اجرا می‌کند.

| ویژگی                       | `transform`   | `adjacent_transform` |
| --------------------------- | ------------- | -------------------- |
| تعداد عناصر ورودی تابع      | یک عنصر       | تعداد ثابت N         |
| نیاز به Forward Range       | خیر           | بله                  |
| پشتیبانی از Lazy Evaluation | بله           | بله                  |
| پردازش عناصر مجاور          | خیر           | بله                  |
| هم‌پوشانی پنجره‌ها          | موضوعیت ندارد | بله                  |

برای مثال:

```
auto single = values
    | std::views::transform(
        [](int value) {
            return value * 2;
        }
    );

auto adjacent = values
    | std::views::adjacent_transform<2>(
        [](int first, int second) {
            return first + second;
        }
    );
```

در View اول، هر عنصر به‌صورت مستقل پردازش می‌شود؛ اما در View دوم، هر بار دو عنصر متوالی در اختیار تابع قرار می‌گیرند.

### **مقایسه با std::views::adjacent**

قابلیت `std::views::adjacent` نیز برای استخراج گروه‌های متوالی از عناصر طراحی شده است.

تفاوت اصلی این دو قابلیت در خروجی آن‌ها است.

- `adjacent` پنجره‌های مجاور را به شکل Tuple از Referenceها ارائه می‌کند.
- `adjacent_transform` تابعی را روی هر پنجره اجرا می‌کند و نتیجه آن تابع را ارائه می‌دهد.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    auto windows = values | std::views::adjacent<2>;

    for (auto [first, second] : windows) {
        std::cout << first + second << ' ';
    }
}
```

خروجی:

```
30 50 70
```

همین عملیات را می‌توان با `adjacent_transform` به شکل مستقیم‌تری انجام داد:

```
auto result = values
    | std::views::adjacent_transform<2>(
        [](int first, int second) {
            return first + second;
        }
    );
```

بنابراین، اگر به خود پنجره‌ها و عناصر داخل آن‌ها نیاز داشته باشیم، `adjacent` انتخاب مناسب‌تری است.

اما اگر فقط نتیجه یک محاسبه روی هر پنجره موردنیاز باشد، `adjacent_transform` انتخاب طبیعی‌تری خواهد بود.

### **مقایسه با std::views::slide**

قابلیت `std::views::slide` نیز پنجره‌های هم‌پوشان تولید می‌کند.

تفاوت مهم این است که `slide` خود پنجره‌ها را به‌عنوان خروجی ارائه می‌دهد، در حالی که `adjacent_transform` نتیجه اجرای یک تابع روی پنجره‌ها را تولید می‌کند.

برای مثال:

```
auto windows = values | std::views::slide(3);
```

در این حالت، هر عنصر خروجی یک View از سه عنصر متوالی است.

اما در مثال زیر، میانگین هر پنجره محاسبه می‌شود:

```
auto averages = values
    | std::views::adjacent_transform<3>(
        [](int a, int b, int c) {
            return (a + b + c) / 3.0;
        }
    );
```

به‌صورت خلاصه:

| قابلیت               | عملکرد                                |
| -------------------- | ------------------------------------- |
| `adjacent`           | تولید Tuple از عناصر مجاور            |
| `slide`              | تولید پنجره‌های مجاور به شکل Subrange |
| `adjacent_transform` | محاسبه نتیجه روی عناصر مجاور          |

اگر نیاز به پردازش دلخواه روی یک پنجره با اندازه مشخص دارید، `slide` نیز می‌تواند همراه با `transform` استفاده شود.

### **مقایسه با std::views::zip_transform**

قابلیت `std::views::zip_transform` برای پردازش عناصر متناظر از چند Range مختلف طراحی شده است.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> first{1, 2, 3};
    std::vector<int> second{10, 20, 30};

    auto result = std::views::zip_transform(
        [](int a, int b) {
            return a + b;
        },
        first,
        second
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

در این مثال، دو Range مختلف به‌صورت هم‌زمان پیمایش می‌شوند.

در مقابل، `adjacent_transform` چند عنصر متوالی را از یک Range واحد دریافت می‌کند.

این تفاوت از نظر طراحی الگوریتم اهمیت زیادی دارد:

- برای پردازش عناصر متناظر چند Range، از `zip_transform` استفاده کنید.
- برای پردازش عناصر متوالی یک Range، از `adjacent_transform` استفاده کنید.

## **بررسی ویژگی‌های Iterator**

برای استفاده حرفه‌ای از این View، باید به ویژگی‌های Iterator آن نیز توجه داشته باشیم.

### **نوع Iterator**

Iteratorهای `adjacent_transform_view` دسترسی به نتایج محاسبه‌شده توسط Callable را فراهم می‌کنند.

این Iteratorها برای پیمایش تنبل نتایج طراحی شده‌اند و رفتار آن‌ها از Iteratorهای View داخلی و الزامات Transformation تأثیر می‌گیرد.

نکته مهم این است که نباید بدون بررسی Constraintها و ویژگی‌های Iterator، فرض کنیم تمام قابلیت‌های Iterator مربوط به Container اصلی عیناً در View خروجی نیز وجود دارند.

برای نمونه، Forward Range بودن ورودی به این معنا نیست که خروجی حتماً همان قابلیت‌های تمام دسته‌های Iterator ورودی را حفظ می‌کند.

### **دسترسی به اندازه با size**

اگر Range داخلی از نوع `sized_range` باشد، View خروجی نیز امکان دسترسی به اندازه خود را فراهم می‌کند.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        10, 20, 30, 40, 50
    };

    auto result = values
        | std::views::adjacent_transform<3>(
            [](int a, int b, int c) {
                return a + b + c;
            }
        );

    std::cout << result.size();
}
```

خروجی:

```
3
```

در این مثال، اندازه خروجی بدون پیمایش تمام عناصر قابل محاسبه است.

البته وجود تابع `size()` در همه شرایط تضمین نشده است و به `sized_range` بودن Range داخلی بستگی دارد.

### **دسترسی به Range اصلی با base**

کلاس `adjacent_transform_view` تابع عضوی `base` را برای دسترسی به View اصلی ارائه می‌کند.

این تابع در دو حالت قابل استفاده است:

- روی شیء `const` یا Lvalue، در صورت Copy Constructible بودن View اصلی، یک کپی از آن برمی‌گرداند.
- روی Rvalue، View اصلی را با Move استخراج می‌کند.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4};

    auto result = values
        | std::views::pairwise_transform(
            [](int a, int b) {
                return a + b;
            }
        );

    auto originalView = result.base();

    for (int value : originalView) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
1 2 3 4
```

توجه داشته باشید که `base()` لزوماً یک Container مستقل برنمی‌گرداند؛ بلکه View پایه را ارائه می‌کند.

## **نکات مربوط به Lifetime و مدیریت حافظه**

یکی از مهم‌ترین نکات در استفاده از Ranges، توجه به طول عمر Range اصلی است.

در مثال زیر:

```
std::vector<int> values{10, 20, 30, 40};

auto result = values
    | std::views::pairwise_transform(
        [](int a, int b) {
            return a + b;
        }
    );
```

View خروجی به Range اصلی وابسته است و عناصر را از همان Container می‌خواند.

بنابراین، اگر Container اصلی از بین برود، استفاده از View می‌تواند باعث Undefined Behavior شود.

برای مثال، الگوی زیر خطرناک است:

```
#include <ranges>
#include <vector>

auto createView() {
    std::vector<int> values{1, 2, 3, 4};

    return values
        | std::views::pairwise_transform(
            [](int a, int b) {
                return a + b;
            }
        );
}

int main() {
    auto result = createView();

    // The underlying vector has already been destroyed.
    // Iterating over result causes undefined behavior.
}
```

علت این مشکل آن است که View معمولاً مالک داده‌های Range اصلی نیست.

در مقابل، اگر Range ورودی یک Rvalue باشد، مکانیزم `views::all` می‌تواند آن را در یک Owning View نگهداری کند؛ در نتیجه طول عمر داده‌ها با طول عمر View مرتبط خواهد شد.

با این حال، باید توجه داشت که Viewهای پایه و Referenceهای خارجی همچنان می‌توانند محدودیت‌های Lifetime خود را داشته باشند.

### **نکته مربوط به تغییر داده‌های اصلی**

اگر عناصر Container اصلی تغییر کنند، View ممکن است نتایج متفاوتی تولید کند.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto result = values
        | std::views::pairwise_transform(
            [](int a, int b) {
                return a + b;
            }
        );

    values[1] = 100;

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
110 130
```

این رفتار یکی از نتایج مستقیم Lazy Evaluation است.

View داده‌ها را به‌صورت مستقل کپی نکرده است و هنگام محاسبه نتایج، مقادیر فعلی عناصر Range اصلی را می‌خواند.

البته تغییر اندازه Container یا عملیاتی که Iteratorهای آن را نامعتبر می‌کنند، می‌تواند باعث Invalid شدن View شود.

## **بررسی کارایی و پیچیدگی زمانی**

از نظر کارایی، `adjacent_transform` برای بسیاری از عملیات مبتنی بر پنجره‌های هم‌پوشان گزینه مناسبی است.

اگر تعداد عناصر ورودی برابر با `S` و اندازه پنجره برابر با `N` باشد، تعداد پنجره‌های قابل پردازش حداکثر برابر با `S - N + 1` خواهد بود.

در نتیجه، اگر اجرای تابع Transformation دارای پیچیدگی ثابت باشد، پیمایش کامل View دارای پیچیدگی زمانی خطی نسبت به اندازه ورودی است:

\\[ O(S) \\]

البته پیچیدگی واقعی به تابع Transformation نیز وابسته است.

### **آیا حافظه اضافی تخصیص داده می‌شود؟**

خود `adjacent_transform` برای نگهداری تمام نتایج خروجی به Container جدیدی نیاز ندارد.

این View معمولاً وضعیت مربوط به Callable و View پایه را نگهداری می‌کند و نتایج را هنگام Dereference تولید می‌کند.

بنابراین، در مقایسه با ساخت یک `std::vector` جدید، از تخصیص حافظه برای کل خروجی جلوگیری می‌شود.

با این حال، اگر Callable خودش حافظه تخصیص دهد یا نتیجه به Container تبدیل شود، هزینه‌های مربوط به آن عملیات همچنان وجود خواهند داشت.

### **آیا عناصر مجاور دوباره خوانده می‌شوند؟**

در پنجره‌های هم‌پوشان، برخی عناصر در چند مرحله مورد استفاده قرار می‌گیرند.

برای مثال:

```
(A, B, C)
(B, C, D)
(C, D, E)
```

عناصر `B` و `C` در چند پنجره حضور دارند.

این موضوع به معنای آن نیست که تمام عناصر ورودی در هر مرحله کپی می‌شوند. پیاده‌سازی از Iteratorها و Referenceهای مربوط به عناصر Range استفاده می‌کند.

با این حال، تابع Transformation در هر Dereference می‌تواند دوباره اجرا شود و بنابراین هزینه آن باید در تحلیل کارایی لحاظ شود.

### **اهمیت هزینه Callable**

فرض کنید تابع Transformation یک محاسبه سنگین انجام می‌دهد.

در چنین شرایطی، اگر یک Iterator چندین بار Dereference شود، احتمال تکرار محاسبه وجود دارد.

برای جلوگیری از محاسبه‌های غیرضروری می‌توان نتایج را در یک Container ذخیره کرد:

```
auto result = values
    | std::views::adjacent_transform<3>(
        [](int a, int b, int c) {
            return a + b + c;
        }
    );

auto cachedResults = std::ranges::to<std::vector>(result);
```

در این حالت، Transformation هنگام ساخت Container اجرا می‌شود و نتایج برای استفاده‌های بعدی ذخیره خواهند شد.

## **نکات مهم و خطاهای رایج**

در استفاده از `adjacent_transform` چند اشتباه متداول وجود دارد که بهتر است از ابتدا به آن‌ها توجه شود.

### **اشتباه در تعداد پارامترهای Lambda**

تعداد پارامترهای تابع باید با اندازه پنجره مطابقت داشته باشد.

برای مثال، کد زیر معتبر نیست:

```
auto result = values
    | std::views::adjacent_transform<3>(
        [](int a, int b) {
            return a + b;
        }
    );
```

زیرا اندازه پنجره ۳ است، اما تابع تنها دو آرگومان دریافت می‌کند.

برای رفع مشکل:

```
auto result = values
    | std::views::adjacent_transform<3>(
        [](int a, int b, int c) {
            return a + b + c;
        }
    );
```

### **استفاده از Input Range**

همان‌طور که توضیح داده شد، ورودی باید `forward_range` باشد.

بنابراین، استفاده مستقیم از Rangeهایی که فقط قابلیت Input Iterator دارند، امکان‌پذیر نیست.

در چنین شرایطی باید داده‌ها ابتدا به یک ساختار مناسب منتقل شوند.

### **فرض اجرای یک‌باره Callable**

نباید فرض کنیم که تابع Transformation فقط یک‌بار برای هر پنجره اجرا می‌شود.

به دلیل Lazy Evaluation، ممکن است با Dereference مجدد یک Iterator، محاسبه دوباره انجام شود.

بنابراین، از توابع دارای Side Effect غیرضروری اجتناب کنید.

### **نادیده گرفتن Lifetime**

View معمولاً مالک Container اصلی نیست.

اگر Container از بین برود یا Iteratorهای آن نامعتبر شوند، استفاده از View ممکن است باعث Undefined Behavior شود.

### **استفاده نامناسب از پنجره‌های بزرگ**

اگر اندازه پنجره بسیار بزرگ باشد، ممکن است نوشتن Lambda با تعداد زیادی پارامتر خوانایی کد را کاهش دهد.

در چنین شرایطی می‌توان از `std::views::slide` همراه با `std::views::transform` استفاده کرد تا پنجره به‌صورت یک Range در اختیار تابع قرار بگیرد.

این روش برای الگوریتم‌هایی که نیاز دارند روی عناصر پنجره با Iterator یا Range کار کنند، انعطاف بیشتری دارد.

### **نادیده گرفتن محدودیت‌های نوع خروجی**

نوع خروجی Callable باید قابل استفاده باشد.

برای مثال، تابعی که `void` برمی‌گرداند، برای این View مناسب نیست:

```
auto result = values
    | std::views::pairwise_transform(
        [](int a, int b) {
            std::cout << a + b;
        }
    );
```

به‌جای آن، مقدار موردنظر را برگردانید:

```
auto result = values
    | std::views::pairwise_transform(
        [](int a, int b) {
            return a + b;
        }
    );
```

## **نکات پیشرفته در طراحی**

### **انتخاب بین View و Container**

یکی از تصمیم‌های مهم در طراحی برنامه این است که آیا خروجی باید به‌صورت View باقی بماند یا به Container تبدیل شود.

استفاده از View مناسب است اگر:

- داده‌ها فقط برای پیمایش موردنیاز باشند.
- بخواهیم چند Transformation را ترکیب کنیم.
- نیازی به نگهداری مستقل نتایج نداشته باشیم.
- بخواهیم از تخصیص حافظه برای خروجی جلوگیری کنیم.

استفاده از Container مناسب است اگر:

- خروجی چندین بار استفاده می‌شود.
- هزینه Transformation زیاد است.
- نتایج باید مستقل از Range اصلی باشند.
- نیاز به ذخیره‌سازی، تغییر یا انتقال نتایج وجود دارد.

### **انتخاب اندازه پنجره**

اندازه پنجره باید براساس ماهیت مسئله انتخاب شود.

برای مثال:

| اندازه پنجره | کاربرد معمول                                     |
| ------------ | ------------------------------------------------ |
| 1            | تبدیل تک‌عنصری؛ معمولاً `transform` مناسب‌تر است |
| 2            | مقایسه دو عنصر متوالی و محاسبه اختلاف            |
| 3            | Moving Average سه‌عنصری و محاسبات محلی           |
| 4 یا بیشتر   | پردازش الگوهای متوالی با اندازه ثابت             |

نکته مهم این است که اندازه پنجره در زمان کامپایل تعیین می‌شود.

اگر الگوریتم به پنجره‌ای با اندازه متغیر در زمان اجرا نیاز داشته باشد، باید از روش دیگری مانند `slide` یا پیاده‌سازی اختصاصی استفاده کرد.

### **استفاده از const View**

کلاس `adjacent_transform_view` امکان ارائه `begin()` و `end()` برای حالت `const` را نیز دارد؛ اما این قابلیت به شرایط خاصی وابسته است.

از جمله:

- View داخلی باید بتواند در حالت `const` به‌عنوان Range استفاده شود.
- Callable باید برای Referenceهای عناصر در حالت `const` قابل فراخوانی باشد.

بنابراین، نباید تصور کرد که هر View قابل پیمایش در حالت غیر `const`، الزاماً در حالت `const` نیز قابل پیمایش است.

در طراحی Callableهای قابل استفاده مجدد، بهتر است رفتار `const` نیز در نظر گرفته شود.

## **پشتیبانی Compilerها**

از آنجا که `std::views::adjacent_transform` بخشی از ++C23 است، برای استفاده از آن باید Compiler و Standard Library مورد استفاده از این قابلیت پشتیبانی کنند.

نکته مهم این است که پشتیبانی از ++C23 در سطح Compiler و Library دو موضوع مرتبط اما متمایز هستند.

برای مثال، فعال کردن Standard زیر به‌تنهایی تضمین‌کننده پیاده‌سازی تمام امکانات موردنیاز نیست:

```
-std=c++23
```

در پروژه‌های مختلف بهتر است پشتیبانی قابلیت موردنظر را با یک برنامه آزمایشی بررسی کنید.

### **بررسی قابلیت با یک برنامه ساده**

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4};

    auto result = values
        | std::views::pairwise_transform(
            [](int a, int b) {
                return a + b;
            }
        );

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی موردانتظار:

```
3 5 7
```

در صورت بروز خطای Compilation، ابتدا نسخه Compiler، نسخه Standard Library و میزان پشتیبانی آن‌ها از Ranges در ++C23 را بررسی کنید.

## **جمع‌بندی**

قابلیت `std::views::adjacent_transform` یکی از امکانات کاربردی Ranges در ++C23 است که پردازش عناصر متوالی و مجاور را ساده‌تر می‌کند.

این قابلیت به توسعه‌دهندگان اجازه می‌دهد بدون مدیریت دستی چند Iterator، عملیات مبتنی بر Sliding Window را به شکل Declarative پیاده‌سازی کنند.

مهم‌ترین ویژگی‌های آن عبارت‌اند از:

- دریافت تعداد ثابتی از عناصر مجاور در هر مرحله.
- اجرای یک Callable روی هر پنجره.
- تولید Lazy View به‌جای Container جدید.
- پشتیبانی از Pipelineهای مبتنی بر Ranges.
- امکان استفاده از پنجره‌های با اندازه متفاوت در زمان کامپایل.
- ارائه نام مستعار `std::views::pairwise_transform` برای پردازش دو عنصر مجاور.
- نیاز به `forward_range` برای Range ورودی.
- امکان ترکیب با Viewهایی مانند `filter`، `transform` و `take`.

از نظر انتخاب ابزار نیز باید به تفاوت قابلیت‌های مشابه توجه داشت:

- برای تبدیل مستقل هر عنصر، `std::views::transform` مناسب است.
- برای استخراج پنجره‌های مجاور به شکل Tuple، از `std::views::adjacent` استفاده کنید.
- برای دریافت پنجره‌های هم‌پوشان به شکل Subrange، `std::views::slide` انتخاب مناسبی است.
- برای پردازش عناصر متناظر چند Range، از `std::views::zip_transform` استفاده کنید.
- برای محاسبه مستقیم روی عناصر متوالی یک Range، `std::views::adjacent_transform` انتخاب مناسبی است.

در مجموع، `adjacent_transform` ابزاری مناسب برای نوشتن الگوریتم‌های مدرن، خوانا و قابل ترکیب در ++C23 است؛ به‌خصوص زمانی که منطق پردازش به رابطه میان عناصر متوالی وابسته باشد.

## **منابع و مراجع**

برای مطالعه جزئیات استاندارد، Constraintها، ساختار Iteratorها و نمونه‌های بیشتر می‌توانید از منابع زیر استفاده کنید:

- [cppreference – std::views::adjacent_transform](https://en.cppreference.com/w/cpp/ranges/adjacent_transform_view?utm_source=chatgpt.com)
- [C++ Working Draft – Adjacent Transform View](https://eel.is/c++draft/range.adjacent.transform?utm_source=chatgpt.com)
- [C++ Working Draft – Adjacent View](https://eel.is/c++draft/range.adjacent?utm_source=chatgpt.com)
- [C++ Working Draft – Slide View](https://eel.is/c++draft/range.slide?utm_source=chatgpt.com)
- [cppreference – C++ Ranges Library](https://en.cppreference.com/w/cpp/ranges?utm_source=chatgpt.com)

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>