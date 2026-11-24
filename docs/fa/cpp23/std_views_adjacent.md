<div align="center">

[🇺🇸 English](../../en/cpp23/std_views_adjacent.md) | [🇮🇷 فارسی](./std_views_adjacent.md)

</div>

---

# **معرفی و بررسی کامل&#x20;**`std::views::adjacent`**&#x20;در ++C23**

## **مقدمه**

یکی از قابلیت‌های کاربردی کتابخانه `Ranges` در استاندارد ++C23، قابلیت `std::views::adjacent` است که امکان پردازش عناصر مجاور یک Range را به شکلی ساده، خوانا و بدون نیاز به ایجاد container جدید فراهم می‌کند.

در بسیاری از الگوریتم‌ها، لازم است چند عنصر متوالی از یک مجموعه را به‌صورت هم‌زمان بررسی کنیم. برای مثال:

- مقایسه هر عنصر با عنصر بعدی.
- محاسبه اختلاف بین مقادیر متوالی.
- بررسی تغییرات یک دنباله از داده‌ها.
- شناسایی الگوهای متوالی در یک مجموعه.
- پردازش پنجره‌های لغزان (Sliding Windows).
- بررسی سه عنصر متوالی برای تشخیص الگوهای خاص.
- محاسبه مشتق تقریبی از داده‌های نمونه‌برداری‌شده.
- تحلیل تغییرات قیمت، دما، سرعت یا سایر داده‌های زمانی.

پیش از ++C23، برای پیاده‌سازی این عملیات معمولاً از Iteratorها، حلقه‌های دستی، الگوریتم‌های سفارشی یا کتابخانه‌های جانبی استفاده می‌شد.

قابلیت `std::views::adjacent` این مسئله را با استفاده از یک Range Adaptor حل می‌کند. این قابلیت یک View ایجاد می‌کند که در هر موقعیت، تعداد مشخصی از عناصر متوالی Range اصلی را در اختیار برنامه‌نویس قرار می‌دهد.

نکته مهم این است که خروجی این قابلیت یک مجموعه جدید از داده‌های کپی‌شده نیست؛ بلکه یک View از عناصر موجود در Range اصلی است.

## **آشنایی با مفهوم Adjacent Elements**

### **عناصر مجاور چیستند؟**

فرض کنید یک `std::vector` شامل مقادیر زیر داریم:

```
std::vector<int> values{10, 20, 30, 40, 50};
```

اگر بخواهیم هر دو عنصر مجاور را در کنار یکدیگر بررسی کنیم، پنجره‌های موردنظر به شکل زیر خواهند بود:

```
(10, 20)
(20, 30)
(30, 40)
(40, 50)
```

در این حالت، هر پنجره شامل دو عنصر متوالی است و با حرکت به موقعیت بعدی، پنجره یک عنصر جلو می‌رود.

اگر به‌جای دو عنصر، سه عنصر متوالی را در نظر بگیریم، خروجی به شکل زیر خواهد بود:

```
(10, 20, 30)
(20, 30, 40)
(30, 40, 50)
```

بنابراین، مفهوم اصلی Adjacent Elements بر اساس ایجاد پنجره‌های هم‌پوشان (Overlapping Windows) از عناصر یک Range است.

### **تفاوت پنجره‌های هم‌پوشان با گروه‌بندی معمولی**

توجه داشته باشید که پنجره‌های حاصل از `adjacent` با گروه‌بندی معمولی عناصر تفاوت دارند.

برای مثال، گروه‌بندی سه‌تایی به شکل زیر است:

```
(10, 20, 30)
(40, 50, ...)
```

اما پنجره‌های مجاور سه‌تایی به شکل زیر هستند:

```
(10, 20, 30)
(20, 30, 40)
(30, 40, 50)
```

در حالت دوم، عناصر بین پنجره‌های متوالی مشترک هستند.

این ویژگی برای الگوریتم‌هایی که به ارتباط میان عناصر متوالی نیاز دارند، اهمیت زیادی دارد.

## **معرفی std::views::adjacent**

### **ساختار کلی**

قابلیت `std::views::adjacent` در Header زیر تعریف شده است:

```
#include <ranges>
```

ساختار استفاده از آن به شکل زیر است:

```
std::views::adjacent<N>(range);
```

در این ساختار:

- پارامتر `N` تعداد عناصر هر پنجره را مشخص می‌کند.
- پارامتر `range` مجموعه یا View ورودی است.
- خروجی یک View از پنجره‌های متوالی ایجادشده از Range اصلی است.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    for (auto window : values | std::views::adjacent<2>) {
        std::cout
            << std::get<0>(window) << ", "
            << std::get<1>(window) << '\n';
    }
}
```

خروجی:

```
10, 20
20, 30
30, 40
40, 50
```

در این مثال، `adjacent<2>` دو عنصر متوالی را در هر پنجره قرار می‌دهد.

### **بررسی رفتار در موقعیت‌های مختلف**

فرض کنید Range ورودی دارای پنج عنصر باشد.

| عبارت         | تعداد عناصر هر پنجره | تعداد پنجره‌ها |
| ------------- | -------------------- | -------------- |
| `adjacent<1>` | 1                    | 5              |
| `adjacent<2>` | 2                    | 4              |
| `adjacent<3>` | 3                    | 3              |
| `adjacent<4>` | 4                    | 2              |
| `adjacent<5>` | 5                    | 1              |
| `adjacent<6>` | 6                    | 0              |

به‌طور کلی، اگر تعداد عناصر Range ورودی را با `S` و تعداد عناصر هر پنجره را با `N` نشان دهیم، تعداد عناصر View خروجی برابر است با:

\\[ \text{size} = \begin{cases} S-N+1 & S \geq N\\\ 0 & S < N \end{cases} \\]

این رابطه یکی از مهم‌ترین ویژگی‌های `std::views::adjacent` است.

## **نحوه دسترسی به عناصر هر پنجره**

### **استفاده از std::get**

هر عنصر خروجی `adjacent` یک `std::tuple` شامل عناصر پنجره است.

بنابراین، می‌توان با استفاده از `std::get` به عناصر مختلف دسترسی داشت.

```
#include <iostream>
#include <ranges>
#include <tuple>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    for (auto window : values | std::views::adjacent<3>) {
        int first = std::get<0>(window);
        int second = std::get<1>(window);
        int third = std::get<2>(window);

        std::cout
            << first << ' '
            << second << ' '
            << third << '\n';
    }
}
```

خروجی:

```
1 2 3
2 3 4
3 4 5
```

نکته مهم این است که اندیس‌های `std::get` در زمان کامپایل مشخص می‌شوند.

در نتیجه، برخلاف containerهای معمولی نمی‌توان با یک اندیس متغیر به اعضای tuple دسترسی پیدا کرد.

### **استفاده از Structured Bindings**

در ++C17، قابلیت Structured Bindings معرفی شد که برای دریافت عناصر `tuple` بسیار مناسب است.

به‌جای استفاده مکرر از `std::get` می‌توان از ساختار زیر استفاده کرد:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    for (auto [first, second] : values | std::views::adjacent<2>) {
        std::cout << first << " -> " << second << '\n';
    }
}
```

خروجی:

```
10 -> 20
20 -> 30
30 -> 40
40 -> 50
```

این روش برای پنجره‌هایی با تعداد عناصر ثابت، خوانایی بسیار خوبی دارد.

### **انتخاب صحیح نوع متغیر در Structured Bindings**

یکی از نکات مهم هنگام استفاده از `adjacent`، انتخاب نوع مناسب برای متغیر حلقه است.

به‌طور معمول، سه حالت زیر قابل استفاده هستند:

```
for (auto [a, b] : range | std::views::adjacent<2>) {
    // ...
}
```

```
for (auto& [a, b] : range | std::views::adjacent<2>) {
    // ...
}
```

```
for (auto&& [a, b] : range | std::views::adjacent<2>) {
    // ...
}
```

تفاوت این حالت‌ها به نحوه نگهداری و دسترسی به شیء tuple حاصل از Dereference مربوط است.

برای کاربردهای عمومی، `auto&&` انتخاب مناسبی است؛ زیرا امکان برخورداری از رفتار Reference و Proxy Reference را فراهم می‌کند.

نکته مهم‌تر این است که Reference بودن عناصر داخل tuple با Reference بودن خود tuple تفاوت دارد. حتی اگر شیء tuple در حلقه کپی شود، عناصر آن می‌توانند همچنان به عناصر Range اصلی ارجاع دهند.

## **مفهوم Reference در خروجی adjacent**

### **آیا عناصر کپی می‌شوند؟**

یکی از مهم‌ترین ویژگی‌های `std::views::adjacent` این است که عناصر پنجره‌ها به‌صورت Reference یا Reference-like از Range اصلی ارائه می‌شوند.

به عبارت دیگر، این قابلیت برای ایجاد هر پنجره نیازی به کپی کردن مقادیر اصلی ندارد.

برای بررسی این موضوع، مثال زیر را در نظر بگیرید:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    for (auto&& [first, second] : values | std::views::adjacent<2>) {
        first *= 2;
        second += 5;
    }

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

خروجی:

```
20 45 70 45
```

تغییرات مستقیماً روی عناصر `vector` اعمال شده‌اند.

علت این رفتار آن است که `adjacent` مقادیر را به‌صورت مستقل کپی نمی‌کند.

### **تفاوت auto و auto& در خروجی**

در مثال زیر:

```
for (auto [first, second] : values | std::views::adjacent<2>) {
    first = 100;
}
```

با وجود استفاده از `auto`، تغییرات می‌توانند روی Range اصلی اعمال شوند.

دلیل آن است که کپی شدن tuple به معنای کپی شدن عناصر Reference شده توسط آن نیست.

البته این رفتار به نوع Reference حاصل از Range اصلی وابسته است.

برای مثال، اگر Range از یک View تبدیلی استفاده کند که مقادیر را به‌صورت Value تولید می‌کند، رفتار می‌تواند متفاوت باشد.

### **بررسی خروجی برای Rangeهای مختلف**

نوع عناصر موجود در tuple خروجی به نوع Reference ارائه‌شده توسط Range اصلی بستگی دارد.

برای مثال:

- در یک `std::vector<int>` معمولی، عناصر خروجی قابلیت تغییر دارند.
- در یک `std::span<const int>`، عناصر فقط قابل خواندن هستند.
- در یک View که عناصر را به‌صورت Value تولید می‌کند، خروجی الزاماً Reference مستقیم به container اصلی نیست.
- در یک View دارای Proxy Reference، رفتار وابسته به همان Proxy است.

بنابراین، نباید فرض کرد که هر عنصر خروجی `adjacent` الزاماً یک `T&` معمولی است.

## **معرفی std::views::pairwise**

یکی از Aliasهای کاربردی مرتبط با `adjacent` در ++C23، قابلیت `std::views::pairwise` است.

این قابلیت معادل زیر است:

```
std::views::adjacent<2>
```

به عبارت دیگر، `pairwise` همیشه دو عنصر مجاور را در هر پنجره قرار می‌دهد.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 3, 6, 10, 15};

    for (auto [previous, current] : values | std::views::pairwise) {
        std::cout << previous << " -> " << current << '\n';
    }
}
```

خروجی:

```
1 -> 3
3 -> 6
6 -> 10
10 -> 15
```

### **چه زمانی از pairwise استفاده کنیم؟**

استفاده از `pairwise` زمانی مناسب است که الگوریتم به‌طور مشخص به دو عنصر مجاور نیاز دارد.

برای مثال:

- محاسبه اختلاف دو مقدار متوالی.
- بررسی صعودی یا نزولی بودن داده‌ها.
- مقایسه دو عنصر متوالی.
- پردازش Edgeهای متوالی در یک مسیر.
- محاسبه تغییرات قیمت در بازه‌های زمانی.

در مقابل، زمانی که تعداد عناصر پنجره متغیر یا بیشتر از دو است، بهتر است از `adjacent<N>` استفاده شود.

## **محدودیت‌های Range ورودی**

### **الزام Forward Range**

یکی از محدودیت‌های اصلی `std::views::adjacent` این است که Range ورودی باید مفهوم `std::ranges::forward_range` را مدل‌سازی کند.

بنابراین، این قابلیت برخلاف برخی Range Adaptorهای دیگر، مستقیماً از یک Input Range معمولی پشتیبانی نمی‌کند.

برای درک بهتر، تفاوت چند مفهوم مهم را بررسی می‌کنیم.

| مفهوم                 | ویژگی اصلی                                     |
| --------------------- | ---------------------------------------------- |
| `input_range`         | امکان پیمایش و خواندن عناصر                    |
| `forward_range`       | امکان پیمایش چندباره و ایجاد Iteratorهای مستقل |
| `bidirectional_range` | امکان حرکت رو به عقب                           |
| `random_access_range` | امکان دسترسی تصادفی به عناصر                   |

هر Forward Range الزاماً Bidirectional یا Random Access نیست.

اما هر Bidirectional Range و Random Access Range، Forward Range نیز محسوب می‌شود.

### **چرا adjacent به Forward Range نیاز دارد؟**

برای ایجاد پنجره‌ای شامل `N` عنصر، قابلیت `adjacent` باید بتواند چند Iterator را به‌صورت هم‌زمان مدیریت کند.

برای مثال، در `adjacent<3>` هر موقعیت خروجی به سه عنصر متوالی اشاره می‌کند.

پیاده‌سازی باید بتواند Iteratorهای مربوط به این سه موقعیت را نگهداری کند و هنگام حرکت به پنجره بعدی، آن‌ها را به‌روزرسانی کند.

این طراحی به قابلیت کپی‌پذیری و پیمایش مجدد Iterator نیاز دارد.

در یک Input Range معمولی، ممکن است Iteratorها Single-pass باشند؛ یعنی پس از حرکت دادن یک Iterator، وضعیت پیمایش قبلی دیگر قابل استفاده نباشد.

به همین دلیل، Forward Range حداقل نیازمندی این قابلیت است.

### **استفاده از Input Range**

کد زیر برای Rangeهایی که فقط `input_range` هستند، قابل استفاده نیست:

```
auto result = input_range | std::views::adjacent<2>;
```

اگر منبع داده فقط یک‌بار قابل خواندن باشد، باید ابتدا راهکار مناسبی برای ذخیره‌سازی یا تبدیل آن به یک Forward Range در نظر گرفت.

برای مثال، می‌توان داده‌ها را در یک container ذخیره کرد:

```
#include <ranges>
#include <vector>

int main() {
    auto input = std::views::iota(0)
        | std::views::take(5);

    std::vector<int> values(input.begin(), input.end());

    auto pairs = values | std::views::adjacent<2>;
}
```

در این مثال، داده‌های ورودی در یک `vector` ذخیره می‌شوند و سپس عملیات `adjacent` روی آن انجام می‌شود.

البته اگر داده‌ها از ابتدا در یک Forward Range قرار دارند، این مرحله اضافه ضرورتی ندارد.

## **بررسی نحوه کارکرد داخلی adjacent_view**

### **معرفی adjacent_view**

قابلیت `std::views::adjacent` در نهایت از کلاس زیر استفاده می‌کند:

```
std::ranges::adjacent_view<V, N>
```

در این تعریف:

- `V` نوع View پایه است.
- `N` تعداد عناصر هر پنجره است.

ساختار مفهومی تعریف این کلاس به شکل زیر است:

```
template<std::ranges::forward_range V, std::size_t N>
    requires std::ranges::view<V> && (N > 0)
class adjacent_view;
```

این تعریف نشان می‌دهد که `N` یک پارامتر Template در زمان کامپایل است.

بنابراین، مقدار آن باید در زمان کامپایل مشخص باشد.

### **نقش Iteratorهای داخلی**

در پیاده‌سازی `adjacent_view`، یک Iterator منفرد برای نمایش پنجره استفاده نمی‌شود.

بلکه Iterator داخلی این View، مجموعه‌ای از `N` Iterator را نگهداری می‌کند.

از نظر مفهومی:

```
std::array<iterator_type, N> current;
```

هر Iterator به یکی از عناصر پنجره جاری اشاره می‌کند.

برای مثال، در `adjacent<3>` وضعیت Iteratorها به شکل زیر خواهد بود:

```
Window 1:
Iterator 1 -> Element 1
Iterator 2 -> Element 2
Iterator 3 -> Element 3

Window 2:
Iterator 1 -> Element 2
Iterator 2 -> Element 3
Iterator 3 -> Element 4
```

با حرکت به موقعیت بعدی، تمام Iteratorها یک گام جلو می‌روند.

این طراحی باعث می‌شود نیازی به ساخت یک container جدید برای هر پنجره وجود نداشته باشد.

### **رفتار Dereference**

عملگر Dereference در Iterator داخلی، یک tuple از عناصر پنجره جاری ایجاد می‌کند.

از نظر مفهومی، خروجی به شکل زیر است:

```
std::tuple<
    reference_type,
    reference_type,
    reference_type
>
```

برای `adjacent<3>`.

البته نوع دقیق عناصر به نوع Reference حاصل از Iteratorهای Range پایه بستگی دارد.

### **رفتار Increment**

هنگامی که Iterator خروجی یک موقعیت جلو می‌رود، Iteratorهای داخلی نیز جلو می‌روند.

برای مثال:

```
Before increment:

[10, 20, 30]

After increment:

[20, 30, 40]
```

در نتیجه، پنجره بعدی با استفاده از Iteratorهای موجود ایجاد می‌شود.

این ساختار از ایجاد و کپی غیرضروری عناصر اصلی جلوگیری می‌کند.

## **بررسی قابلیت‌های Iterator**

یکی از نکات مهم برای توسعه‌دهندگان حرفه‌ای، بررسی قابلیت‌های Iterator خروجی `adjacent_view` است.

این قابلیت همیشه یک `forward_range` تولید می‌کند و در صورت پشتیبانی Range پایه، برخی قابلیت‌های قوی‌تر را نیز حفظ می‌کند.

### **پشتیبانی از Bidirectional Range**

اگر Range پایه یک `bidirectional_range` باشد، View حاصل نیز قابلیت حرکت به عقب را خواهد داشت.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    auto windows = values | std::views::adjacent<2>;

    auto it = windows.end();

    --it;

    auto [first, second] = *it;

    std::cout << first << ' ' << second;
}
```

خروجی:

```
4 5
```

در این مثال، Iterator از انتهای View یک موقعیت به عقب حرکت می‌کند.

### **پشتیبانی از Random Access Range**

اگر Range پایه از `random_access_range` پشتیبانی کند، View خروجی نیز این قابلیت را خواهد داشت.

بنابراین، عملیات‌هایی مانند موارد زیر امکان‌پذیر هستند:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    auto windows = values | std::views::adjacent<2>;

    auto it = windows.begin();

    it += 2;

    auto [first, second] = *it;

    std::cout << first << ' ' << second;
}
```

خروجی:

```
30 40
```

این ویژگی برای الگوریتم‌هایی که نیازمند دسترسی تصادفی به پنجره‌ها هستند، مفید است.

### **نکته مهم درباره iterator_category**

در مشخصات استاندارد، `iterator_category` مربوط به Iterator داخلی `adjacent_view` برابر `std::input_iterator_tag` است.

در عین حال، قابلیت پیمایش واقعی Iterator با استفاده از `iterator_concept` مشخص می‌شود.

این دو مفهوم یکسان نیستند.

به‌طور خاص، `iterator_concept` می‌تواند بر اساس قابلیت‌های Range پایه، از نوع Forward، Bidirectional یا Random Access باشد.

این طراحی برای سازگاری با Iteratorهای قدیمی و در عین حال پشتیبانی از مفهوم‌های جدید Iterator در Ranges در نظر گرفته شده است.

بنابراین، برای بررسی قابلیت واقعی Iterator بهتر است از مفهوم‌های استاندارد مانند موارد زیر استفاده کنید:

```
std::ranges::forward_range
std::ranges::bidirectional_range
std::ranges::random_access_range
```

نباید صرفاً با بررسی `iterator_category` درباره قابلیت‌های Iterator خروجی نتیجه‌گیری کرد.

## **بررسی رفتار size**

### **محاسبه تعداد پنجره‌ها**

اگر Range پایه یک `sized_range` باشد، View خروجی نیز یک `sized_range` خواهد بود.

تعداد عناصر با استفاده از رابطه زیر محاسبه می‌شود:

\\[ \max(0, S-N+1) \\]

که در آن:

- `S` تعداد عناصر Range اصلی است.
- `N` اندازه پنجره است.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    auto windows = values | std::views::adjacent<3>;

    std::cout << windows.size();
}
```

خروجی:

```
3
```

### **بررسی Rangeهای خالی و کوچک**

در صورت کوچک‌تر بودن اندازه Range از اندازه پنجره، خروجی خالی خواهد بود.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2};

    auto windows = values | std::views::adjacent<3>;

    std::cout << std::ranges::empty(windows);
}
```

خروجی:

```
1
```

این رفتار به‌صورت طبیعی از تعریف `adjacent` حاصل می‌شود و نیازی به بررسی دستی اندازه Range قبل از استفاده نیست.

## **بررسی پارامتر N و مقدار صفر**

### **آیا adjacent معتبر است؟**

یکی از جزئیات مهم این قابلیت، رفتار آن در زمان استفاده از مقدار صفر برای `N` است.

در استاندارد ++C23، عبارت زیر برای یک Forward Range معتبر است:

```
auto result = range | std::views::adjacent<0>;
```

در این حالت، نتیجه یک View خالی از نوع مرتبط با `std::tuple<>` است.

اما این رفتار نباید به‌عنوان ایجاد پنجره‌هایی با صفر عنصر در نظر گرفته شود.

در واقع، `views::adjacent<0>` یک حالت ویژه در تعریف Range Adaptor است و به‌جای ایجاد `adjacent_view`، یک Empty View برمی‌گرداند.

### **رفتار adjacent با Input Range**

نکته مهم این است که حتی حالت صفر نیز الزام Forward Range را نادیده نمی‌گیرد.

طبق اصلاح استانداردی LWG 4098، استفاده از `views::adjacent<0>` برای Input Rangeهایی که Forward Range نیستند، مجاز نیست.

بنابراین، نباید از مقدار صفر برای دور زدن محدودیت‌های `adjacent` استفاده کرد.

### **انتخاب مقدار مناسب N**

در کاربردهای معمول، مقدار `N` باید بر اساس تعداد عناصر موردنیاز الگوریتم انتخاب شود.

| مقدار N        | کاربرد                                   |
| -------------- | ---------------------------------------- |
| `1`            | ایجاد پنجره‌های تک‌عضوی                  |
| `2`            | مقایسه عناصر مجاور                       |
| `3`            | بررسی الگوهای سه‌عنصری                   |
| `4`            | تحلیل پنجره‌های چهارتایی                 |
| مقادیر بزرگ‌تر | پردازش پنجره‌های با ابعاد ثابت و بزرگ‌تر |

## **ترکیب adjacent با سایر Range Adaptorها**

یکی از مزیت‌های اصلی `std::views::adjacent` امکان ترکیب آن با سایر قابلیت‌های کتابخانه Ranges است.

این قابلیت به‌تنهایی فقط پنجره‌ها را ایجاد می‌کند؛ اما با ترکیب سایر Viewها می‌توان عملیات پیچیده‌تری را با کد ساده‌تر پیاده‌سازی کرد.

### **ترکیب با filter**

فرض کنید بخواهیم فقط زوج‌های مجاوری را انتخاب کنیم که مقدار عنصر دوم آن‌ها بزرگ‌تر از عنصر اول باشد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 15, 30, 25, 40};

    auto increasing_pairs =
        values
        | std::views::adjacent<2>
        | std::views::filter([](const auto& pair) {
            return std::get<0>(pair) < std::get<1>(pair);
        });

    for (auto [first, second] : increasing_pairs) {
        std::cout << first << " -> " << second << '\n';
    }
}
```

خروجی:

```
10 -> 20
15 -> 30
25 -> 40
```

در این مثال:

- `adjacent<2>` زوج‌های مجاور را ایجاد می‌کند.
- `filter` فقط زوج‌هایی را نگه می‌دارد که شرط موردنظر را داشته باشند.

این ترکیب بدون ایجاد یک container واسط انجام می‌شود.

### **ترکیب با transform**

در بسیاری از موارد، هدف فقط بررسی پنجره‌ها نیست؛ بلکه می‌خواهیم از هر پنجره یک مقدار جدید محاسبه کنیم.

برای مثال، می‌توان اختلاف دو عنصر متوالی را محاسبه کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 15, 25, 40, 60};

    auto differences =
        values
        | std::views::adjacent<2>
        | std::views::transform([](const auto& pair) {
            return std::get<1>(pair) - std::get<0>(pair);
        });

    for (int difference : differences) {
        std::cout << difference << ' ';
    }
}
```

خروجی:

```
5 10 15 20
```

در این حالت، خروجی نهایی یک View از مقادیر محاسبه‌شده است.

نکته مهم این است که `transform` نیز به‌صورت Lazy اجرا می‌شود.

بنابراین، تا زمانی که عناصر خروجی پیمایش نشوند، عملیات محاسبه اختلاف انجام نخواهد شد.

### **ترکیب با take**

گاهی فقط به تعداد محدودی از پنجره‌های ابتدایی نیاز داریم.

در این شرایط می‌توان از `std::views::take` استفاده کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6, 7};

    auto windows =
        values
        | std::views::adjacent<3>
        | std::views::take(2);

    for (auto [a, b, c] : windows) {
        std::cout << a << ' ' << b << ' ' << c << '\n';
    }
}
```

خروجی:

```
1 2 3
2 3 4
```

### **ترکیب با drop**

برای نادیده گرفتن تعدادی از پنجره‌های ابتدایی می‌توان از `std::views::drop` استفاده کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6};

    auto windows =
        values
        | std::views::adjacent<2>
        | std::views::drop(2);

    for (auto [a, b] : windows) {
        std::cout << a << ' ' << b << '\n';
    }
}
```

خروجی:

```
3 4
4 5
5 6
```

### **ترکیب با iota**

قابلیت `std::views::iota` برای ایجاد دنباله‌های عددی مفید است.

از آنجا که Range تولیدشده توسط `iota` در حالت متداول یک Forward Range یا Random Access Range است، می‌توان آن را مستقیماً با `adjacent` ترکیب کرد.

```
#include <iostream>
#include <ranges>

int main() {
    auto numbers = std::views::iota(1, 6);

    for (auto [first, second] : numbers | std::views::adjacent<2>) {
        std::cout << first << ", " << second << '\n';
    }
}
```

خروجی:

```
1, 2
2, 3
3, 4
4, 5
```

## **معرفی std::views::adjacent_transform**

یکی از قابلیت‌های نزدیک و بسیار مهم به `std::views::adjacent`، قابلیت `std::views::adjacent_transform` است.

این قابلیت نیز در ++C23 معرفی شده است.

تفاوت اصلی آن با `adjacent` در این است که به‌جای تولید tuple از عناصر مجاور، یک تابع را روی عناصر هر پنجره اجرا می‌کند و نتیجه تابع را در View خروجی قرار می‌دهد.

ساختار کلی آن به شکل زیر است:

```
std::views::adjacent_transform<N>(function);
```

### **مقایسه با adjacent و transform**

فرض کنید بخواهیم اختلاف دو عنصر متوالی را محاسبه کنیم.

با `adjacent` و `transform`:

```
auto differences =
    values
    | std::views::adjacent<2>
    | std::views::transform([](const auto& pair) {
        return std::get<1>(pair) - std::get<0>(pair);
    });
```

با `adjacent_transform`:

```
auto differences =
    values
    | std::views::adjacent_transform<2>(
        [](int first, int second) {
            return second - first;
        }
    );
```

روش دوم مستقیماً تابع را با عناصر پنجره فراخوانی می‌کند.

در نتیجه، نیازی به دریافت و پردازش tuple در تابع تبدیل وجود ندارد.

### **مثال عملی adjacent_transform**

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 15, 25, 40, 60};

    auto differences =
        values
        | std::views::adjacent_transform<2>(
            [](int first, int second) {
                return second - first;
            }
        );

    for (int difference : differences) {
        std::cout << difference << ' ';
    }
}
```

خروجی:

```
5 10 15 20
```

### **چه زمانی adjacent_transform مناسب‌تر است؟**

استفاده از `adjacent_transform` در شرایط زیر مناسب‌تر است:

- زمانی که به نتیجه محاسبه‌شده هر پنجره نیاز داریم.
- زمانی که نمی‌خواهیم tupleها را در کد مدیریت کنیم.
- زمانی که هدف اصلی یک Transformation روی عناصر مجاور است.
- زمانی که خوانایی Pipeline اهمیت دارد.

در مقابل، اگر لازم است خود عناصر پنجره را بررسی کنیم، تغییر دهیم یا چندین عملیات متفاوت روی آن‌ها انجام دهیم، `adjacent` انعطاف‌پذیری بیشتری فراهم می‌کند.

## **مقایسه adjacent با slide**

یکی دیگر از قابلیت‌های مرتبط با پردازش پنجره‌ها، `std::views::slide` است.

این قابلیت نیز در ++C23 معرفی شده است.

هر دو قابلیت برای ایجاد پنجره‌های لغزان استفاده می‌شوند؛ اما ساختار خروجی آن‌ها متفاوت است.

### **تفاوت ساختاری**

قابلیت `adjacent` در هر موقعیت، یک `tuple` از عناصر متوالی ایجاد می‌کند.

اما `slide` در هر موقعیت، یک View شامل عناصر پنجره را ارائه می‌دهد.

برای مثال، با استفاده از `adjacent<3>`:

```
auto windows = values | std::views::adjacent<3>;
```

خروجی مفهومی:

```
tuple(10, 20, 30)
tuple(20, 30, 40)
tuple(30, 40, 50)
```

اما با استفاده از `slide(3)`:

```
auto windows = values | std::views::slide(3);
```

خروجی مفهومی:

```
view(10, 20, 30)
view(20, 30, 40)
view(30, 40, 50)
```

### **جدول مقایسه**

| ویژگی                          | adjacent                          | slide                                |
| ------------------------------ | --------------------------------- | ------------------------------------ |
| استاندارد                      | ++C23                             | ++C23                                |
| نوع خروجی هر پنجره             | `tuple`                           | View                                 |
| تعداد عناصر هر پنجره           | ثابت در Template                  | ثابت در زمان اجرا                    |
| دسترسی به عناصر                | `std::get` یا Structured Bindings | Range Interface                      |
| تغییر عناصر اصلی               | در صورت Reference بودن عناصر      | از طریق View و Referenceهای آن       |
| استفاده در Structured Bindings | مستقیم                            | به شکل معمول مستقیم نیست             |
| کاربرد اصلی                    | پردازش چند عنصر مجاور             | پردازش پنجره‌های لغزان به‌صورت Range |

### **انتخاب بین adjacent و slide**

اگر تعداد عناصر پنجره از قبل مشخص است و می‌خواهیم به عناصر آن مانند یک مجموعه ثابت دسترسی داشته باشیم، `adjacent` انتخاب مناسبی است.

اما اگر می‌خواهیم هر پنجره را مانند یک Range مستقل پردازش کنیم، `slide` گزینه مناسب‌تری خواهد بود.

برای مثال، استفاده از `slide` برای محاسبه میانگین یک پنجره چندعنصری می‌تواند خواناتر باشد:

```
#include <iostream>
#include <numeric>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    auto windows = values | std::views::slide(3);

    for (auto window : windows) {
        int sum = std::accumulate(window.begin(), window.end(), 0);

        std::cout << sum / 3 << ' ';
    }
}
```

خروجی:

```
20 30 40
```

## **مقایسه adjacent با chunk**

قابلیت `std::views::chunk` نیز در ++C23 معرفی شده است.

این قابلیت عناصر Range را به گروه‌های متوالی تقسیم می‌کند.

تفاوت مهم `chunk` و `adjacent` در هم‌پوشانی پنجره‌ها است.

### **مثال مقایسه**

فرض کنید داده‌های ورودی به شکل زیر باشند:

```
std::vector<int> values{1, 2, 3, 4, 5, 6};
```

با استفاده از `adjacent<3>`:

```
(1, 2, 3)
(2, 3, 4)
(3, 4, 5)
(4, 5, 6)
```

اما با استفاده از `chunk(3)`:

```
(1, 2, 3)
(4, 5, 6)
```

در `chunk`، عناصر بین گروه‌های متوالی مشترک نیستند.

در مقابل، `adjacent` پنجره‌های هم‌پوشان ایجاد می‌کند.

### **جدول مقایسه**

| ویژگی               | adjacent          | chunk                        |
| ------------------- | ----------------- | ---------------------------- |
| هم‌پوشانی پنجره‌ها  | دارد              | ندارد                        |
| تعداد عناصر هر گروه | N                 | معمولاً N، به‌جز گروه پایانی |
| حرکت پنجره          | یک عنصر           | یک گروه                      |
| کاربرد اصلی         | تحلیل عناصر مجاور | تقسیم Range به گروه‌ها       |

بنابراین، برای پردازش عناصر متوالی که ارتباط بین پنجره‌ها اهمیت دارد، `adjacent` مناسب‌تر است.

برای تقسیم یک مجموعه به بخش‌های مستقل، بهتر است از `chunk` استفاده شود.

## **بررسی پیچیدگی زمانی و حافظه**

### **پیچیدگی زمانی**

از آنجا که `adjacent` یک View است، هنگام ایجاد آن تمام پنجره‌ها محاسبه و ذخیره نمی‌شوند.

ایجاد View معمولاً دارای پیچیدگی ثابت است؛ زیرا فقط وضعیت Range پایه و اطلاعات لازم برای پیمایش نگهداری می‌شود.

عملیات Dereference یک پنجره شامل دسترسی به `N` عنصر است.

بنابراین، هزینه Dereference از نظر تعداد عناصر پنجره متناسب با `N` است.

برای پیمایش کامل یک Range شامل `S` عنصر:

\\[ O((S-N+1)\times N) \\]

برای `N` ثابت، پیچیدگی کلی پیمایش از نظر تعداد عملیات دسترسی به عناصر، خطی نسبت به اندازه Range است:

\\[ O(S) \\]

البته هزینه واقعی عملیات انجام‌شده روی عناصر پنجره به الگوریتمی که برنامه‌نویس پیاده‌سازی می‌کند بستگی دارد.

### **پیچیدگی حافظه**

یکی از مزیت‌های مهم `adjacent` عدم نیاز به ایجاد container جدید برای ذخیره پنجره‌ها است.

Iterator داخلی به‌صورت مفهومی `N` Iterator را نگهداری می‌کند.

بنابراین، فضای اضافی موردنیاز برای وضعیت Iterator متناسب با `N` است:

\\[ O(N) \\]

این مقدار مستقل از تعداد عناصر Range اصلی است.

در نتیجه، حتی برای Rangeهای بزرگ نیز نیازی به تخصیص حافظه برای ذخیره تمام پنجره‌ها وجود ندارد.

### **نکته مهم درباره Lazy Evaluation**

عملیات `adjacent` از مدل Lazy Evaluation پیروی می‌کند.

یعنی ایجاد View به معنای محاسبه و ذخیره تمام پنجره‌ها نیست.

پنجره‌ها هنگام پیمایش و Dereference شدن Iterator در اختیار برنامه قرار می‌گیرند.

برای مثال:

```
auto windows = values | std::views::adjacent<2>;
```

در این مرحله، فقط View ساخته شده است.

تا زمانی که `windows` پیمایش نشود، پنجره‌های خروجی به‌صورت کامل تولید و مصرف نمی‌شوند.

## **کاربردهای عملی std::views::adjacent**

### **محاسبه اختلاف عناصر متوالی**

یکی از رایج‌ترین کاربردهای `adjacent` محاسبه اختلاف بین مقادیر متوالی است.

این کاربرد در تحلیل داده‌های زمانی، محاسبه تغییرات قیمت و پردازش اطلاعات حسگرها اهمیت زیادی دارد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> temperatures{
        20.5, 21.2, 22.8, 22.1, 24.0
    };

    auto changes =
        temperatures
        | std::views::adjacent_transform<2>(
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
0.7
1.6
-0.7
1.9
```

در این مثال، تغییر دما بین دو اندازه‌گیری متوالی محاسبه می‌شود.

### **بررسی مرتب بودن داده‌ها**

یکی دیگر از کاربردهای مهم، بررسی صعودی بودن یک دنباله است.

```
#include <algorithm>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 3, 5, 7, 9};

    bool strictly_increasing = std::ranges::all_of(
        values | std::views::adjacent<2>,
        [](const auto& pair) {
            return std::get<0>(pair) < std::get<1>(pair);
        }
    );

    std::cout << std::boolalpha << strictly_increasing;
}
```

خروجی:

```
true
```

در این الگوریتم، تمام زوج‌های متوالی بررسی می‌شوند.

اگر حتی یک زوج شرط صعودی بودن را نقض کند، نتیجه `false` خواهد بود.

نکته مهم این است که `all_of` روی Range خالی مقدار `true` برمی‌گرداند.

بنابراین، برای یک Range با کمتر از دو عنصر نیز نتیجه این الگوریتم `true` خواهد بود.

اگر از نظر منطق برنامه، دنباله باید حداقل دو عنصر داشته باشد، لازم است این شرط جداگانه بررسی شود.

### **تشخیص نقاط تغییر جهت**

فرض کنید یک مجموعه از مقادیر داریم و می‌خواهیم نقاطی را شناسایی کنیم که مقدار آن‌ها نسبت به دو همسایه تغییر جهت می‌دهد.

برای این کار به سه عنصر متوالی نیاز داریم.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        1, 3, 5, 4, 2, 6, 8, 7
    };

    auto turning_points =
        values
        | std::views::adjacent_transform<3>(
            [](int previous, int current, int next) {
                return (current > previous && current > next)
                    || (current < previous && current < next);
            }
        );

    int index = 1;

    for (bool is_turning_point : turning_points) {
        if (is_turning_point) {
            std::cout << "Turning point at index "
                      << index << '\n';
        }

        ++index;
    }
}
```

خروجی:

```
Turning point at index 2
Turning point at index 4
Turning point at index 6
```

در این مثال، هر پنجره سه‌عضوی برای تشخیص یک نقطه بیشینه یا کمینه محلی بررسی می‌شود.

### **بررسی پیوستگی داده‌ها**

در برخی الگوریتم‌ها لازم است اختلاف بین دو مقدار متوالی از یک حد مشخص بیشتر نباشد.

برای مثال، در داده‌های اندازه‌گیری‌شده از یک حسگر می‌توان پیوستگی تغییرات را بررسی کرد.

```
#include <algorithm>
#include <cmath>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<double> readings{
        10.0, 10.5, 10.8, 11.0, 15.5, 15.8
    };

    constexpr double max_change = 2.0;

    bool valid = std::ranges::all_of(
        readings | std::views::adjacent<2>,
        [](const auto& pair) {
            return std::abs(
                std::get<1>(pair) - std::get<0>(pair)
            ) <= max_change;
        }
    );

    std::cout << std::boolalpha << valid;
}
```

خروجی:

```
false
```

زیرا اختلاف بین `11.0` و `15.5` از مقدار مجاز بیشتر است.

### **پردازش مسیرها و مختصات**

در الگوریتم‌های گرافیکی و محاسبات هندسی، گاهی لازم است هر دو نقطه متوالی یک مسیر را بررسی کنیم.

برای مثال:

```
#include <cmath>
#include <iostream>
#include <ranges>
#include <vector>

struct Point {
    double x;
    double y;
};

int main() {
    std::vector<Point> path{
        {0.0, 0.0},
        {3.0, 4.0},
        {6.0, 8.0},
        {9.0, 8.0}
    };

    auto distances =
        path
        | std::views::adjacent_transform<2>(
            [](const Point& first, const Point& second) {
                const double dx = second.x - first.x;
                const double dy = second.y - first.y;

                return std::sqrt(dx * dx + dy * dy);
            }
        );

    for (double distance : distances) {
        std::cout << distance << '\n';
    }
}
```

خروجی:

```
5
5
3
```

در این مثال، طول هر Segment از مسیر محاسبه می‌شود.

### **بررسی الگوهای سه‌عنصری**

یکی دیگر از کاربردهای `adjacent` تشخیص الگوهای خاص در دنباله‌ها است.

برای مثال، فرض کنید می‌خواهیم توالی‌هایی را پیدا کنیم که مقدار عنصر میانی از هر دو همسایه بزرگ‌تر باشد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        2, 5, 3, 8, 4, 9, 6
    };

    auto peaks =
        values
        | std::views::adjacent<3>
        | std::views::filter([](const auto& window) {
            return std::get<1>(window) > std::get<0>(window)
                && std::get<1>(window) > std::get<2>(window);
        });

    for (auto [left, middle, right] : peaks) {
        std::cout << middle << '\n';
    }
}
```

خروجی:

```
5
8
9
```

این روش برای پردازش سیگنال‌ها، تحلیل سری‌های زمانی و شناسایی الگوهای محلی کاربرد دارد.

## **مدیریت Lifetime و طول عمر Range**

یکی از مهم‌ترین موضوعات هنگام استفاده از Ranges، مدیریت صحیح طول عمر اشیای مورد استفاده است.

از آنجا که `adjacent` یک View است، معمولاً داده‌ها را مالکیت نمی‌کند؛ بلکه به Range پایه وابسته است.

### **استفاده از lvalue Range**

در مثال زیر:

```
std::vector<int> values{1, 2, 3, 4};

auto windows = values | std::views::adjacent<2>;
```

متغیر `windows` به داده‌های موجود در `values` وابسته است.

بنابراین، تا زمانی که View مورد استفاده قرار می‌گیرد، باید طول عمر `values` معتبر باشد.

### **استفاده از temporary Range**

یکی از مزیت‌های Ranges این است که می‌توان از برخی temporary Rangeها نیز استفاده کرد.

برای مثال:

```
#include <ranges>

int main() {
    auto windows =
        std::views::iota(1, 10)
        | std::views::adjacent<2>;

    for (auto [first, second] : windows) {
        // Process adjacent values
    }
}
```

در این مثال، `iota_view` یک View است که به‌صورت مستقل قابل نگهداری است.

در نتیجه، زنجیره Viewها بدون نیاز به یک container واسط ساخته می‌شود.

### **خطر Dangling Reference**

اگر یک View به حافظه‌ای وابسته باشد که طول عمر آن تمام شده است، استفاده از View می‌تواند به رفتار تعریف‌نشده منجر شود.

برای مثال، الگوی زیر خطرناک است:

```
#include <ranges>
#include <vector>

int main() {
    auto windows = [] {
        std::vector<int> values{1, 2, 3, 4};

        return values | std::views::adjacent<2>;
    }();

    // The underlying vector no longer exists.
}
```

در این مثال، `values` با پایان اجرای Lambda از بین می‌رود.

اما View خروجی تلاش می‌کند به عناصر همان `vector` دسترسی داشته باشد.

بنابراین، استفاده از `windows` پس از بازگشت Lambda معتبر نیست.

نکته مهم این است که `views::all` می‌تواند برای Rangeهای مختلف، رفتار متفاوتی در نگهداری مالکیت داشته باشد.

برای یک lvalue container معمولی، View معمولاً یک Reference-like Wrapper مانند `ref_view` نگهداری می‌کند.

برای یک rvalue container مناسب، ممکن است مالکیت از طریق `owning_view` مدیریت شود.

با این حال، باید طول عمر Range پایه و نحوه ایجاد View را همیشه در نظر گرفت.

## **محدودیت‌ها و نکات مهم در استفاده**

### **تغییر اندازه container اصلی**

اگر View از یک container مانند `std::vector` ساخته شده باشد، عملیات‌هایی که باعث جابه‌جایی حافظه یا نامعتبر شدن Iteratorها می‌شوند، می‌توانند View را غیرقابل استفاده کنند.

برای مثال:

```
std::vector<int> values{1, 2, 3, 4};

auto windows = values | std::views::adjacent<2>;

values.push_back(5);
```

اگر عملیات `push_back` باعث Reallocation شود، Iteratorهای داخلی View ممکن است نامعتبر شوند.

بنابراین، نباید تصور کرد که Viewها مستقل از تغییرات ساختاری container اصلی هستند.

### **تغییر عناصر Range هنگام پیمایش**

تغییر مقدار عناصر معمولاً در صورتی امکان‌پذیر است که Range پایه عناصر قابل تغییر ارائه دهد.

اما تغییر ساختار container، حذف عناصر یا عملیات‌هایی که Iteratorها را نامعتبر می‌کنند، مسئله متفاوتی است.

برای مثال، تغییر مقدار یک عنصر در `std::vector` معمولاً Iteratorها را نامعتبر نمی‌کند؛ اما حذف یا درج عناصر ممکن است چنین اثری داشته باشد.

### **هزینه تابع Transformation**

اگر از `adjacent_transform` استفاده می‌کنید، باید توجه داشته باشید که تابع Transformation هنگام ارزیابی عناصر خروجی اجرا می‌شود.

اگر یک عنصر خروجی چند بار Dereference شود، نباید بدون بررسی قرارداد الگوریتم، فرض کنید تابع دقیقاً یک بار اجرا خواهد شد.

بنابراین، بهتر است تابع Transformation تا حد امکان Pure و بدون Side Effect باشد.

### **استفاده با Rangeهای نامحدود**

یکی از ویژگی‌های مفید `adjacent` امکان استفاده از آن با برخی Infinite Rangeها است.

برای مثال:

```
#include <iostream>
#include <ranges>

int main() {
    auto numbers = std::views::iota(1)
        | std::views::adjacent<3>
        | std::views::take(5);

    for (auto [a, b, c] : numbers) {
        std::cout << a << ' '
                  << b << ' '
                  << c << '\n';
    }
}
```

خروجی:

```
1 2 3
2 3 4
3 4 5
4 5 6
5 6 7
```

در این مثال، `iota` یک دنباله نامحدود تولید می‌کند.

با ترکیب `adjacent` و `take` فقط پنج پنجره ابتدایی پردازش می‌شوند.

نکته مهم این است که برای استفاده از Range نامحدود باید مطمئن شویم که عملیات‌های بعدی نیز امکان خاتمه پیمایش را فراهم می‌کنند.

### **استفاده از N بزرگ**

از آنجا که `N` یک پارامتر Template است، اندازه پنجره در زمان کامپایل مشخص می‌شود.

همچنین، Iterator داخلی به تعداد `N` Iterator نیاز دارد.

بنابراین، برای پنجره‌های بسیار بزرگ باید هزینه نگهداری وضعیت Iterator و هزینه پردازش هر پنجره را در نظر گرفت.

اگر اندازه پنجره باید در زمان اجرا تغییر کند، `adjacent` انتخاب مناسبی نیست و بهتر است قابلیت‌هایی مانند `slide` بررسی شوند.

## **اشتباهات رایج هنگام استفاده**

### **تصور اینکه adjacent یک container جدید ایجاد می‌کند**

یکی از اشتباهات رایج این است که خروجی `adjacent` را یک مجموعه مستقل از داده‌ها در نظر بگیریم.

در واقع، این قابلیت یک View ایجاد می‌کند و داده‌ها معمولاً در Range اصلی باقی می‌مانند.

بنابراین، تغییرات روی عناصر Reference شده می‌توانند روی داده‌های اصلی اثر بگذارند.

### **تصور اینکه N در زمان اجرا قابل تغییر است**

عبارت زیر معتبر نیست:

```
std::size_t window_size = 3;

auto windows = values | std::views::adjacent<window_size>;
```

زیرا `window_size` یک متغیر معمولی است و مقدار آن به‌عنوان پارامتر Template در زمان کامپایل شناخته نمی‌شود.

برای این قابلیت باید از یک مقدار ثابت در زمان کامپایل استفاده کرد:

```
constexpr std::size_t window_size = 3;

auto windows = values | std::views::adjacent<window_size>;
```

### **تصور اینکه خروجی یک pair است**

در `adjacent<2>`، خروجی از نوع `std::tuple` است، نه الزاماً `std::pair`.

بنابراین، برای دسترسی به عناصر باید از `std::get` یا Structured Bindings استفاده شود.

این موضوع زمانی اهمیت دارد که بخواهیم نوع دقیق خروجی را در Templateها بررسی کنیم.

### **استفاده از Input Range**

استفاده از یک Range که فقط `input_range` است، معتبر نیست.

برای استفاده از `adjacent` باید حداقل شرایط `forward_range` برقرار باشد.

### **استفاده از View پس از نابودی Range اصلی**

اگر View به یک container محلی وابسته باشد، پس از نابودی آن container نباید از View استفاده کرد.

این مسئله یکی از مهم‌ترین نکات مربوط به Lifetime در طراحی مبتنی بر Ranges است.

### **فرض اینکه پیمایش View همیشه رایگان است**

اگرچه ساخت View معمولاً هزینه بسیار کمی دارد، اما پیمایش آن رایگان نیست.

هر پنجره باید Dereference شود و عناصر آن در اختیار الگوریتم قرار گیرند.

همچنین، هزینه عملیات انجام‌شده روی هر پنجره به کد برنامه‌نویس وابسته است.

## **بررسی سازگاری با استانداردها و کامپایلرها**

### **نسخه استاندارد**

قابلیت‌های زیر در استاندارد ++C23 معرفی شده‌اند:

- `std::views::adjacent`
- `std::views::pairwise`
- `std::ranges::adjacent_view`
- `std::views::adjacent_transform`
- `std::views::pairwise_transform`
- `std::ranges::adjacent_transform_view`

بنابراین، برای استفاده استاندارد از این قابلیت‌ها باید از کامپایلری استفاده کرد که پیاده‌سازی مربوط به ++C23 را ارائه می‌دهد.

### **فعال‌سازی C++23**

برای فعال‌سازی استاندارد ++C23 می‌توان از دستورات زیر استفاده کرد.

**GCC:**

```
g++ -std=c++23 main.cpp -o main
```

**Clang:**

```
clang++ -std=c++23 main.cpp -o main
```

**MSVC:**

```
cl /std:c++latest /EHsc main.cpp
```

نکته مهم این است که فعال کردن استاندارد ++C23 به‌تنهایی تضمین نمی‌کند که تمام قابلیت‌های آن در نسخه مورد استفاده کامپایلر و Standard Library پیاده‌سازی شده باشند.

پشتیبانی واقعی به نسخه کامپایلر و کتابخانه استاندارد آن وابسته است.

برای بررسی وضعیت پشتیبانی بهتر است مستندات پیاده‌سازی مورد استفاده بررسی شوند.

### **بررسی مقدار Feature-Test Macro**

برای بررسی پشتیبانی از `adjacent` می‌توان از Feature-Test Macro زیر استفاده کرد:

```
#include <ranges>

#ifndef __cpp_lib_ranges_zip
#error "Required ranges library features are unavailable"
#endif
```

البته `__cpp_lib_ranges_zip` فقط یک بررسی کلی برای مجموعه‌ای از قابلیت‌های مرتبط با Ranges مانند `zip` است و به‌تنهایی آزمون اختصاصی پشتیبانی از `adjacent` محسوب نمی‌شود.

برای بررسی اختصاصی‌تر، می‌توان از Compile-time Test استفاده کرد:

```
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto windows = values | std::views::adjacent<2>;

    static_assert(std::ranges::forward_range<decltype(windows)>);
    static_assert(std::ranges::sized_range<decltype(windows)>);
}
```

این روش مشخص می‌کند که قابلیت موردنظر در محیط کامپایل فعلی قابل استفاده است و ویژگی‌های مورد انتظار را دارد.

## **بررسی تفاوت با روش‌های قدیمی‌تر**

### **استفاده از حلقه‌های Iterator**

پیش از معرفی `adjacent`، یکی از روش‌های متداول استفاده از دو Iterator بود.

برای مثال:

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    for (auto first = values.begin();
         first != values.end() && std::next(first) != values.end();
         ++first) {

        auto second = std::next(first);

        std::cout << *first << " -> " << *second << '\n';
    }
}
```

این روش برای پردازش دو عنصر مجاور مناسب است.

اما برای پنجره‌های بزرگ‌تر، مدیریت Iteratorها پیچیده‌تر می‌شود.

برای مثال، در یک پنجره پنج‌عنصری باید چندین Iterator را مدیریت کنیم و مراقب باشیم از انتهای Range عبور نکنیم.

در مقابل، `adjacent` این منطق را به شکل یک Range Adaptor استاندارد ارائه می‌دهد.

### **استفاده از حلقه‌های اندیسی**

روش دیگری که در کدهای قدیمی مشاهده می‌شود، استفاده از اندیس است:

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    for (std::size_t i = 0; i + 1 < values.size(); ++i) {
        std::cout << values[i] << " -> "
                  << values[i + 1] << '\n';
    }
}
```

این روش برای containerهایی که از Random Access پشتیبانی می‌کنند، ساده است.

اما برای Rangeهایی که دسترسی اندیسی ندارند، قابل استفاده نیست.

همچنین، کد به نوع container وابستگی بیشتری پیدا می‌کند.

در مقابل، `adjacent` بر اساس مفاهیم Ranges طراحی شده و با Forward Rangeهای مختلف کار می‌کند.

### **استفاده از std::adjacent_find**

در ++C98، الگوریتم `std::adjacent_find` معرفی شده است.

این الگوریتم برای پیدا کردن اولین دو عنصر مجاور که با یکدیگر برابر هستند یا Predicate مشخصی را برآورده می‌کنند، استفاده می‌شود.

اما هدف آن با `std::views::adjacent` متفاوت است.

برای مثال:

```
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 3, 4, 5};

    auto it = std::adjacent_find(values.begin(), values.end());

    if (it != values.end()) {
        std::cout << *it;
    }
}
```

خروجی:

```
3
```

در این مثال، فقط اولین زوج عناصر مجاور برابر پیدا می‌شود.

اما `adjacent` تمام پنجره‌های متوالی را در قالب یک View ارائه می‌کند.

بنابراین، این دو قابلیت جایگزین مستقیم یکدیگر نیستند.

## **نکات طراحی برای توسعه‌دهندگان حرفه‌ای**

### **انتخاب بین adjacent و adjacent_transform**

در طراحی Pipelineهای مبتنی بر Ranges، بهتر است ابتدا مشخص کنیم که آیا به خود پنجره نیاز داریم یا فقط نتیجه پردازش آن.

اگر هدف فقط محاسبه یک مقدار از هر پنجره است، `adjacent_transform` معمولاً طراحی مستقیم‌تری ارائه می‌دهد.

اما اگر به عناصر پنجره در چند مرحله مختلف نیاز داریم، `adjacent` انعطاف‌پذیرتر است.

### **توجه به نوع Reference**

هنگام نوشتن Generic Code بهتر است فرض نکنیم که تمام عناصر Range از نوع Reference معمولی هستند.

برخی Viewها ممکن است Proxy Reference یا Value تولید کنند.

بنابراین، استفاده از `auto&&` و Generic Lambdaها در بسیاری از سناریوها انتخاب مناسب‌تری است.

### **جلوگیری از Side Effect در Viewهای Lazy**

از آنجا که Viewها Lazy هستند، نباید تابع‌های Transformation را بدون در نظر گرفتن زمان و تعداد دفعات ارزیابی، با Side Effectهای حساس طراحی کرد.

برای مثال، انجام عملیات I/O، تغییر وضعیت Global یا افزایش شمارنده‌های خارجی در Lambdaهای Pipeline می‌تواند باعث رفتار غیرمنتظره در صورت پیمایش چندباره شود.

بهتر است عملیات اصلی Viewها تا حد امکان Pure باشند.

### **توجه به قابلیت‌های Range پایه**

قدرت Iteratorهای خروجی `adjacent` به قابلیت‌های Range پایه وابسته است.

اگر Range اصلی Random Access باشد، امکان استفاده از دسترسی تصادفی به پنجره‌ها وجود دارد.

اما اگر Range اصلی فقط Forward باشد، نباید انتظار عملیات Random Access از View خروجی داشته باشیم.

### **توجه به مالکیت داده‌ها**

در طراحی APIهایی که Viewها را به‌عنوان خروجی برمی‌گردانند، باید مشخص باشد که View به چه داده‌هایی وابسته است.

اگر View به یک container محلی ارجاع دهد، بازگرداندن آن می‌تواند باعث Dangling Reference شود.

این مسئله فقط مختص `adjacent` نیست و در تمام طراحی‌های مبتنی بر `std::views` اهمیت دارد.

## **جمع‌بندی**

قابلیت `std::views::adjacent` یکی از ابزارهای مهم کتابخانه Ranges در استاندارد ++C23 است که پردازش چند عنصر متوالی را ساده‌تر و خواناتر می‌کند.

این قابلیت با ایجاد یک View از پنجره‌های هم‌پوشان، امکان دسترسی هم‌زمان به تعداد مشخصی از عناصر یک Range را فراهم می‌کند.

مهم‌ترین ویژگی‌ها و نکات آن عبارت‌اند از:

- قابلیت `adjacent` از استاندارد ++C23 در دسترس است.
- تعداد عناصر هر پنجره با پارامتر Template به نام `N` مشخص می‌شود.
- هر پنجره شامل یک `std::tuple` از عناصر متوالی است.
- عناصر خروجی معمولاً به عناصر Range اصلی ارجاع می‌دهند.
- این قابلیت حداقل به یک `forward_range` نیاز دارد.
- اندازه خروجی در صورت Sized بودن Range پایه قابل محاسبه است.
- View خروجی در صورت پشتیبانی Range پایه می‌تواند Bidirectional یا Random Access باشد.
- مقدار `N` باید در زمان کامپایل مشخص باشد.
- قابلیت `std::views::pairwise` معادل `std::views::adjacent<2>` است.
- قابلیت `std::views::adjacent_transform` برای پردازش مستقیم پنجره‌ها مناسب است.
- قابلیت `std::views::slide` زمانی مناسب‌تر است که هر پنجره باید به‌صورت یک Range مستقل پردازش شود.
- قابلیت `std::views::chunk` برای تقسیم Range به گروه‌های غیرهم‌پوشان استفاده می‌شود.
- از آنجا که `adjacent` یک View است، برای مدیریت Lifetime و اعتبار Iteratorهای Range پایه باید دقت کرد.

در مجموع، اگر در یک الگوریتم نیاز به بررسی چند عنصر متوالی دارید، `std::views::adjacent` یکی از انتخاب‌های استاندارد و مناسب در ++C مدرن است.

## **منابع و مراجع**

برای مطالعه بیشتر و بررسی جزئیات استاندارد می‌توانید از منابع زیر استفاده کنید:

- [cppreference - std::ranges::adjacent_view](https://en.cppreference.com/w/cpp/ranges/adjacent_view?utm_source=chatgpt.com)
- [C++ Working Draft - Adjacent View](https://eel.is/c++draft/range.adjacent?utm_source=chatgpt.com)
- [C++ Working Draft - Adjacent Transform View](https://eel.is/c++draft/range.adjacent.transform?utm_source=chatgpt.com)
- [C++ Working Draft - Range Adaptors](https://eel.is/c++draft/range.adaptors?utm_source=chatgpt.com)
- [cppreference - Range Adaptors](https://en.cppreference.com/w/cpp/ranges.html?utm_source=chatgpt.com)
- [cppreference - std::views::slide](https://en.cppreference.com/w/cpp/ranges/slide_view?utm_source=chatgpt.com)
- [cppreference - std::views::chunk](https://en.cppreference.com/w/cpp/ranges/chunk_view?utm_source=chatgpt.com)
- [cppreference - std::views::adjacent_transform](https://en.cppreference.com/w/cpp/ranges/adjacent_transform_view?utm_source=chatgpt.com)

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [HadiAbbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>