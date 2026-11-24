<div align="center">

[🇺🇸 English](../../en/cpp23/std_ranges_improvements.md) | [🇮🇷 فارسی](./std_ranges_improvements.md)

</div>

---

# بهبودهای کتابخانه `std::ranges` در ++C23

کتابخانه `std::ranges` در ++C20 معرفی شد تا پردازش مجموعه‌داده‌ها را با استفاده از Range، View، Concept و الگوریتم‌های مبتنی بر Range ساده‌تر کند. استاندارد ++C23 این کتابخانه را با Adaptorهای جدید، الگوریتم‌های تجمیع و جست‌وجو، امکانات بهتر برای Move و دسترسی فقط‌خواندنی و سازوکار توسعه Adaptorهای سفارشی گسترش داده است.

این تغییرات عمدتاً در کتابخانه استاندارد هستند؛ بنابراین برای استفاده از آن‌ها باید هم کامپایلر و هم پیاده‌سازی کتابخانه استاندارد، قابلیت‌های مربوط به ++C23 را پشتیبانی کنند.

## آشنایی با مفاهیم پایه

### مفهوم Range

یک `Range` نوعی است که امکان دریافت `begin` و `end` آن وجود دارد. Range الزاماً Container نیست؛ برای نمونه، `std::views::iota` می‌تواند دنباله‌ای از اعداد را بدون ذخیره‌سازی همه عناصر تولید کند.

```cpp
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};
    static_assert(std::ranges::range<decltype(values)>);
}
```

### مفهوم View

یک `View` نوعی Range سبک است که معمولاً برای مشاهده یا تبدیل Range دیگری استفاده می‌شود. بسیاری از Viewها مالک داده اصلی نیستند و به‌جای ساخت Container جدید، نحوه پیمایش داده را تغییر می‌دهند.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6};

    auto result = values
        | std::views::filter([](int x) { return x % 2 == 0; })
        | std::views::transform([](int x) { return x * x; });

    for (int value : result) {
        std::cout << value << ' ';
    }
}
```

خروجی برابر `4 16 36` است.

### مفهوم Lazy Evaluation

بسیاری از Viewها Lazy هستند؛ یعنی عملیات فیلتر و تبدیل هنگام ساخت Pipeline اجرا نمی‌شوند و در زمان پیمایش عناصر ارزیابی می‌شوند. این رفتار می‌تواند از ساخت داده‌های میانی جلوگیری کند، اما به معنی سریع‌تر بودن قطعی نیست. Predicate یا Transformer ممکن است چند بار فراخوانی شود و هزینه آن در هر پیمایش دوباره پرداخت شود.

### مالکیت و طول عمر

Viewهای ساخته‌شده از Lvalue یک Container معمولاً به همان Container ارجاع می‌دهند. بنابراین Container باید تا پایان استفاده از View زنده بماند. در مقابل، `viewable_range` و سازوکارهای `views::all` برای مدیریت تفاوت Lvalue و Rvalue به کار می‌روند؛ اما نباید فرض کرد هر View مالک داده‌ای است که به آن اشاره می‌کند.

## Range Adaptorهای جدید در ++C23

### تقسیم داده با `views::chunk`

قابلیت `std::views::chunk(n)` یک Range را به بخش‌های متوالی و غیرهم‌پوشان با اندازه حداکثر `n` تقسیم می‌کند. آخرین بخش می‌تواند کوچک‌تر باشد.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6, 7, 8};

    for (auto part : values | std::views::chunk(3)) {
        for (int value : part) {
            std::cout << value << ' ';
        }
        std::cout << '\n';
    }
}
```

خروجی شامل بخش‌های `1 2 3`، `4 5 6` و `7 8` است.

کاربردها شامل پردازش Batch، تقسیم بسته‌های داده و پردازش بلوکی است. مقدار اندازه باید مثبت باشد. `chunk` عناصر را به‌صورت گروه‌های View ارائه می‌کند و به‌خودی‌خود Container جدیدی از داده‌ها نمی‌سازد.

### پنجره‌های هم‌پوشان با `views::slide`

قابلیت `std::views::slide(n)` پنجره‌های متوالی با اندازه `n` می‌سازد. برخلاف `chunk`، پنجره‌ها هم‌پوشانی دارند و هر بار یک موقعیت جلو می‌روند.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    for (auto window : values | std::views::slide(3)) {
        for (int value : window) {
            std::cout << value << ' ';
        }
        std::cout << '\n';
    }
}
```

خروجی پنجره‌های `1 2 3`، `2 3 4` و `3 4 5` است. اگر تعداد عناصر کمتر از اندازه پنجره باشد، پنجره کاملی تولید نمی‌شود.

این قابلیت برای Moving Average، تحلیل سری زمانی و الگوریتم‌های مبتنی بر همسایگی مفید است.

### گروه‌بندی با `views::chunk_by`

قابلیت `std::views::chunk_by(predicate)` عناصر مجاور را تا زمانی که Predicate رابطه آن‌ها را تأیید کند، در یک گروه قرار می‌دهد.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 7, 8, 10, 11, 12};

    auto groups = values | std::views::chunk_by(
        [](int left, int right) {
            return right - left == 1;
        }
    );

    for (auto group : groups) {
        for (int value : group) {
            std::cout << value << ' ';
        }
        std::cout << '\n';
    }
}
```

گروه‌ها به‌ترتیب `1 2 3`، `7 8` و `10 11 12` هستند. Predicate رابطه بین دو عنصر مجاور را بررسی می‌کند، نه عضویت مستقل هر عنصر در یک دسته. برای نتیجه قابل‌پیش‌بینی، Predicate باید رفتاری پایدار داشته باشد و نباید به تغییر وضعیت خارجی وابسته باشد.

### پیمایش فاصله‌دار با `views::stride`

قابلیت `std::views::stride(n)` عناصر Range را با گام مثبت `n` پیمایش می‌کند.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50, 60, 70};

    for (int value : values | std::views::stride(2)) {
        std::cout << value << ' ';
    }
}
```

خروجی `10 30 50 70` است. این قابلیت برای نمونه‌برداری و انتخاب عناصر با فاصله ثابت کاربرد دارد. گام باید بزرگ‌تر از صفر باشد.

### پیمایش همراه با Index با `views::enumerate`

قابلیت `std::views::enumerate` هر عنصر را همراه با Index آن ارائه می‌کند. نتیجه به شکل Tuple-like است و با Structured Binding قابل بازکردن است.

```cpp
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> names{"Alice", "Bob", "Charlie"};

    for (auto [index, name] : names | std::views::enumerate) {
        std::cout << index << ": " << name << '\n';
    }
}
```

Index از صفر آغاز می‌شود. این قابلیت جایگزین شمارنده دستی در بسیاری از حلقه‌ها است. نوع Index را در کد عمومی بهتر است بدون نیاز واقعی، به یک نوع عددی خاص محدود نکنیم.

### ترکیب Rangeها با `views::zip`

قابلیت `std::views::zip` چند Range را به‌صورت هم‌زمان پیمایش می‌کند و عناصر متناظر را در قالب Tuple-like ارائه می‌دهد.

```cpp
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> names{"Alice", "Bob", "Charlie"};
    std::vector<int> ages{25, 30, 35};

    for (auto [name, age] : std::views::zip(names, ages)) {
        std::cout << name << ": " << age << '\n';
    }
}
```

پیمایش تا پایان کوتاه‌ترین Range ادامه پیدا می‌کند؛ بنابراین اگر اندازه ورودی‌ها برابر نباشد، دنباله باقی‌مانده Range بلندتر در خروجی دیده نمی‌شود.

### تبدیل هم‌زمان با `views::zip_transform`

قابلیت `std::views::zip_transform(function, ranges...)` عناصر متناظر چند Range را به تابع می‌دهد و نتیجه تابع را به‌عنوان Range خروجی ارائه می‌کند.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{10, 20, 30};

    auto sums = std::views::zip_transform(
        [](int x, int y) { return x + y; },
        a,
        b
    );

    for (int value : sums) {
        std::cout << value << ' ';
    }
}
```

خروجی `11 22 33` است. این Adaptor زمانی مناسب است که نتیجه محاسبه مهم باشد و نیازی به نگهداری Tupleهای عناصر ورودی نباشد.

### دسترسی به عناصر مجاور با `views::adjacent`

قابلیت `std::views::adjacent<N>` پنجره‌های هم‌پوشان با اندازه ثابت `N` تولید می‌کند. برخلاف `slide`، اندازه در زمان کامپایل مشخص می‌شود.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    for (auto [left, right] : values | std::views::adjacent<2>) {
        std::cout << left << ", " << right << '\n';
    }
}
```

این قابلیت برای مقایسه عناصر مجاور و محاسبات مبتنی بر چند مقدار متوالی مناسب است.

### تبدیل عناصر مجاور با `views::adjacent_transform`

قابلیت `std::views::adjacent_transform<N>(function)` تابع را روی هر گروه مجاور با اندازه ثابت اجرا می‌کند.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};

    auto differences = values
        | std::views::adjacent_transform<2>(
            [](int left, int right) {
                return right - left;
            }
        );

    for (int value : differences) {
        std::cout << value << ' ';
    }
}
```

خروجی `10 10 10 10` است. این قابلیت معادل ترکیب مفهومی پنجره‌های مجاور و Transform است و برای محاسبه اختلاف، نرخ تغییر و ویژگی‌های محلی کاربرد دارد.

### حاصل‌ضرب دکارتی با `views::cartesian_product`

قابلیت `std::views::cartesian_product` تمام ترکیب‌های ممکن عناصر چند Range را تولید می‌کند.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{1, 2, 3};
    std::vector<char> letters{'A', 'B'};

    for (auto [number, letter] :
         std::views::cartesian_product(numbers, letters)) {
        std::cout << number << letter << '\n';
    }
}
```

تعداد ترکیب‌ها برابر حاصل‌ضرب اندازه Rangeهای ورودی است:

\[
N = n_1 \times n_2 \times \cdots \times n_k
\]

این قابلیت برای تولید فضای جست‌وجو و تست ترکیبیاتی مفید است؛ اما رشد تعداد ترکیب‌ها می‌تواند بسیار سریع باشد. Lazy بودن مانع انفجار تعداد عناصر در پیمایش کامل نمی‌شود.

### اتصال Rangeهای داخلی با `views::join_with`

قابلیت `std::views::join_with(separator)` عناصر Rangeهای داخلی را Flatten می‌کند و Separator را میان آن‌ها قرار می‌دهد.

```cpp
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> words{"Modern", "C++", "Ranges"};

    for (char ch : words | std::views::join_with(' ')) {
        std::cout << ch;
    }
}
```

خروجی `Modern C++ Ranges` است. Separator می‌تواند یک مقدار یا Range مناسب باشد. این قابلیت برای ساخت متن و اتصال ساختارهای تو‌در‌تو کاربرد دارد.

### تولید مقادیر تکراری با `views::repeat`

قابلیت `std::views::repeat(value)` یک Range تکرارشونده از مقدار مشخص ایجاد می‌کند. نسخه دارای تعداد، دنباله محدود می‌سازد و نسخه بدون تعداد، نامحدود است.

```cpp
#include <iostream>
#include <ranges>

int main() {
    auto values = std::views::repeat(42, 5);

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

برای نسخه نامحدود، مصرف‌کننده باید حدی مانند `take` تعیین کند:

```cpp
auto values = std::views::repeat(42) | std::views::take(5);
```

این قابلیت برای تولید داده آزمایشی، مقدار ثابت و دنباله‌های تکراری مناسب است.

## کنترل نحوه دسترسی به عناصر

### مشاهده عناصر به‌صورت Rvalue با `views::as_rvalue`

قابلیت `std::views::as_rvalue` عناصر Range را به‌صورت Rvalue در اختیار مصرف‌کننده قرار می‌دهد. این کار می‌تواند به الگوریتم‌های بعدی اجازه دهد عناصر را Move کنند.

```cpp
#include <ranges>
#include <string>
#include <utility>
#include <vector>

int main() {
    std::vector<std::string> source{"Alice", "Bob"};
    std::vector<std::string> destination;

    destination.reserve(source.size());

    for (auto&& value : source | std::views::as_rvalue) {
        destination.push_back(std::move(value));
    }
}
```

خود `as_rvalue` انتقال را انجام نمی‌دهد؛ تنها Value Category عناصر را تغییر می‌دهد. انتقال واقعی به عملیات مصرف‌کننده وابسته است. پس از Move، شیء مبدأ همچنان معتبر است، اما مقدار آن مشخص نیست.

### مشاهده عناصر به‌صورت Const با `views::as_const`

قابلیت `std::views::as_const` یک View فقط‌خواندنی از عناصر ارائه می‌کند.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    for (const int& value : values | std::views::as_const) {
        std::cout << value << ' ';
    }
}
```

این قابلیت برای APIهایی مفید است که باید دسترسی خواندنی را از طریق Range ارائه دهند. بااین‌حال، `as_const` داده اصلی را تغییرناپذیر نمی‌کند؛ کد دیگری که Reference غیر Const به Container دارد همچنان می‌تواند آن را تغییر دهد.

## الگوریتم‌های جدید std::ranges در ++C23

### بررسی وجود مقدار با `ranges::contains`

الگوریتم `std::ranges::contains` بررسی می‌کند که آیا مقدار موردنظر در Range وجود دارد یا خیر.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    bool found = std::ranges::contains(values, 30);
    std::cout << std::boolalpha << found;
}
```

این الگوریتم در بسیاری از موارد جایگزین خواناتری برای ترکیب `find` و مقایسه با `end` است.

### بررسی زیرRange متوالی با `ranges::contains_subrange`

الگوریتم `std::ranges::contains_subrange` بررسی می‌کند که آیا یک الگوی متوالی در Range وجود دارد یا خیر.

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6};
    std::vector<int> pattern{3, 4, 5};

    bool found = std::ranges::contains_subrange(values, pattern);
}
```

منظور از زیرRange، عناصر متوالی است؛ نه صرفاً وجود تک‌تک عناصر الگو در موقعیت‌های مختلف.

### جست‌وجوی آخرین تطبیق با `find_last`

استاندارد ++C23 سه الگوریتم `std::ranges::find_last`، `std::ranges::find_last_if` و `std::ranges::find_last_if_not` را اضافه کرده است.

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 2, 4, 2, 5};

    auto result = std::ranges::find_last(values, 2);

    if (result.found) {
        int lastValue = *result.begin();
    }
}
```

این الگوریتم‌ها برخلاف `find` که اولین تطبیق را پیدا می‌کند، آخرین تطبیق را جست‌وجو می‌کنند. نتیجه از نوع Subrange است و موقعیت بخش تطبیق را در اختیار می‌گذارد.

### تجمیع عناصر با الگوریتم‌های Fold

در ++C23 الگوریتم‌های زیر برای Fold به `std::ranges` اضافه شده‌اند:

- `fold_left`
- `fold_left_first`
- `fold_right`
- `fold_right_last`
- `fold_left_with_iter`
- `fold_left_first_with_iter`

الگوریتم `fold_left` عناصر را از چپ به راست با مقدار اولیه ترکیب می‌کند.

```cpp
#include <iostream>
#include <numeric>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    int result = std::ranges::fold_left(
        values,
        0,
        [](int accumulator, int value) {
            return accumulator + value;
        }
    );

    std::cout << result;
}
```

نتیجه برابر ۱۵ است. از نظر مفهومی، محاسبه به شکل `(((((0 + 1) + 2) + 3) + 4) + 5)` انجام می‌شود.

الگوریتم `fold_right` ترتیب پردازش را از راست به چپ می‌کند. این تفاوت برای عملیات غیرجابجایی مانند تفریق یا ساخت رشته اهمیت دارد. `fold_left_first` و `fold_right_last` مقدار اولیه جداگانه نمی‌گیرند و از یکی از عناصر Range به‌عنوان شروع استفاده می‌کنند؛ برای Range خالی، نتیجه اختیاری خالی است.

دو الگوریتم دارای پسوند `with_iter` علاوه بر نتیجه، Iterator موقعیت پردازش را نیز برمی‌گردانند.

### مقداردهی Range با `ranges::iota`

الگوریتم `std::ranges::iota` عناصر یک Range موجود را با مقادیر متوالی مقداردهی می‌کند.

```cpp
#include <numeric>
#include <vector>

int main() {
    std::vector<int> values(5);
    std::ranges::iota(values, 10);
}
```

مقادیر Range برابر `10, 11, 12, 13, 14` می‌شوند. این الگوریتم را نباید با `std::views::iota` اشتباه گرفت: اولی Range موجود را مقداردهی می‌کند و دومی یک Range تولیدی ایجاد می‌کند.

### جابه‌جایی عناصر با `shift_left` و `shift_right`

الگوریتم‌های `std::ranges::shift_left` و `std::ranges::shift_right` عناصر را درون Range به چپ یا راست جابه‌جا می‌کنند.

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    auto newEnd = std::ranges::shift_left(values, 2);
}
```

پس از انتقال، بخش معتبر جدید شامل `3, 4, 5` است. Iterator بازگشتی پایان بخش معتبر جدید را مشخص می‌کند. عناصر خارج از این بخش را نباید داده‌های معتبر با مقدار معنادار فرض کرد. این الگوریتم‌ها برای Rangeهای قابل‌نوشتن با الزامات Iterator مناسب طراحی شده‌اند.

## توسعه Range Adaptorهای سفارشی

### استفاده از `range_adaptor_closure`

در ++C23 کلاس پایه `std::ranges::range_adaptor_closure` برای ساخت Adaptorهای سفارشی سازگار با Pipe اضافه شده است.

```cpp
#include <iostream>
#include <ranges>
#include <utility>
#include <vector>

struct MultiplyByTwo
    : std::ranges::range_adaptor_closure<MultiplyByTwo> {

    template <std::ranges::viewable_range R>
    auto operator()(R&& range) const {
        return std::views::all(std::forward<R>(range))
            | std::views::transform([](int value) {
                return value * 2;
            });
    }
};

int main() {
    std::vector<int> values{1, 2, 3};

    for (int value : values | MultiplyByTwo{}) {
        std::cout << value << ' ';
    }
}
```

این سازوکار برای Libraryهایی مفید است که می‌خواهند عملیات اختصاصی خود را مانند Adaptorهای استاندارد در Pipeline ترکیب کنند. در طراحی Adaptor عمومی باید به `viewable_range`، Forwarding Reference، نوع عناصر و طول عمر Range توجه کرد.

### بهبود `views::join` برای Rangeهای موقت

با تغییرات P2328، محدودیت‌های `join_view` برای Rangeهای داخلی Prvalue کاهش یافتند. برای نمونه، خروجی یک Transform که یک Container موقت تولید می‌کند، می‌تواند با `join` تخت شود.

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    auto flattened = values
        | std::views::transform([](int value) {
            return std::vector<int>{value, value * 10};
        })
        | std::views::join;

    for (int value : flattened) {
        std::cout << value << ' ';
    }
}
```

خروجی `1 10 2 20 3 30` است. این بهبود ترکیب Transform و Join را برای تولید و تخت‌سازی Rangeهای داخلی ساده‌تر می‌کند.

### بهبود Projectionها

در ++C23 محدودیت‌های برخی Projectionها در الگوریتم‌های Ranges با P2609 کاهش یافته‌اند. Projection تابعی است که الگوریتم از طریق آن ویژگی موردنظر را از هر عنصر استخراج می‌کند.

```cpp
#include <algorithm>
#include <string>
#include <vector>

struct User {
    int id;
    std::string name;
};

int main() {
    std::vector<User> users{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"}
    };

    auto result = std::ranges::find(users, 2, &User::id);
}
```

در این مثال، جست‌وجو روی فیلد `id` انجام می‌شود و نیازی به تعریف Comparator اختصاصی نیست.

## تفاوت قابلیت‌های مشابه

| قابلیت | رفتار اصلی | هم‌پوشانی |
|---|---|---|
| `chunk(n)` | تقسیم به بخش‌های با اندازه حداکثر n | خیر |
| `slide(n)` | پنجره‌های متوالی با اندازه n | بله |
| `chunk_by(pred)` | گروه‌بندی بر اساس رابطه عناصر مجاور | وابسته به Predicate |
| `stride(n)` | انتخاب عناصر با گام n | کاربرد متفاوت |
| `adjacent<N>` | پنجره هم‌پوشان با اندازه ثابت زمان کامپایل | بله |
| `zip` | ترکیب عناصر متناظر چند Range | وابسته به ورودی‌ها |
| `cartesian_product` | تولید تمام ترکیب‌های ممکن | ترکیب کامل |

## نکات عملکردی و محدودیت‌ها

### Lazy بودن به معنی رایگان بودن نیست

Viewها معمولاً از ساخت Containerهای میانی جلوگیری می‌کنند، اما هر بار پیمایش می‌تواند Predicateها و Transformerها را دوباره اجرا کند. اگر نتیجه محاسبه گران‌قیمت است و چند بار مصرف می‌شود، Materialize کردن نتیجه در یک Container ممکن است مناسب‌تر باشد.

### طول عمر داده اصلی را مدیریت کنید

View غیرمالک که به Container اشاره می‌کند، نباید پس از نابودی Container استفاده شود. همچنین تغییر ساختار Container ممکن است Iteratorهای View را نامعتبر کند. قواعد Invalidation همان Container اصلی همچنان اهمیت دارند.

### تفاوت بین Reference و Value را در نظر بگیرید

خروجی Transform ممکن است Reference یا Value تولید کند. انتخاب نوع خروجی روی امکان تغییر داده، Copy و Move تأثیر می‌گذارد. برای داده‌های بزرگ، از کپی ناخواسته جلوگیری کنید و برای انتقال آگاهانه منابع از `as_rvalue` و عملیات مصرف‌کننده مناسب استفاده کنید.

### Predicate و Transformer باید رفتار مناسبی داشته باشند

توابعی که در Viewها استفاده می‌شوند ممکن است در زمان‌های متفاوت و بیش از یک بار فراخوانی شوند. اتکا به ترتیب دقیق فراخوانی یا تغییر وضعیت خارجی می‌تواند رفتار برنامه را شکننده کند. در صورت امکان، توابع خالص و بدون Side Effect انتخاب کنید.

### هزینه ترکیب چند Adaptor را بررسی کنید

Pipeline طولانی همیشه بهتر از حلقه ساده نیست. در مسیرهای حساس به کارایی، Benchmark انجام دهید و نوع Iterator، تعداد فراخوانی‌ها، Cache Locality و هزینه Projectionها را بررسی کنید.

### اندازه خروجی را دست‌کم نگیرید

به‌خصوص در `cartesian_product`، تعداد عناصر خروجی حاصل‌ضرب اندازه ورودی‌ها است. در Rangeهای نامحدود یا بسیار بزرگ، باید مصرف‌کننده محدودکننده‌ای مانند `take` در نظر گرفته شود.

## تشخیص پشتیبانی کتابخانه استاندارد

پشتیبانی از زبان ++C23 به‌تنهایی تضمین نمی‌کند تمام امکانات کتابخانه استاندارد نیز پیاده‌سازی شده باشند. نسخه کامپایلر و نسخه کتابخانه استاندارد را جداگانه بررسی کنید.

برای برخی قابلیت‌ها Feature-test Macro در `<version>` یا هدرهای مرتبط تعریف شده است. از جمله:

| Macro | قابلیت |
|---|---|
| `__cpp_lib_ranges_as_const` | `as_const` |
| `__cpp_lib_ranges_as_rvalue` | `as_rvalue` |
| `__cpp_lib_ranges_cartesian_product` | `cartesian_product` |
| `__cpp_lib_ranges_chunk` | `chunk` و `slide` |
| `__cpp_lib_ranges_chunk_by` | `chunk_by` |
| `__cpp_lib_ranges_enumerate` | `enumerate` |
| `__cpp_lib_ranges_join_with` | `join_with` |
| `__cpp_lib_ranges_repeat` | `repeat` |
| `__cpp_lib_ranges_stride` | `stride` |
| `__cpp_lib_ranges_zip` | `zip`، `zip_transform`، `adjacent` و `adjacent_transform` |
| `__cpp_lib_ranges_to_container` | `ranges::to` |

مقدار Macroها را از مستندات پیاده‌سازی مورد استفاده بررسی کنید؛ وجود یک Macro یا قابلیت در نسخه‌ای از کتابخانه، به معنی پشتیبانی یکسان در تمام کامپایلرها نیست.

## راهنمای انتخاب قابلیت مناسب

- برای تقسیم داده به Batchهای ثابت از `chunk` استفاده کنید.
- برای پنجره‌های هم‌پوشان با اندازه زمان اجرا از `slide` استفاده کنید.
- برای گروه‌بندی بر اساس رابطه عناصر مجاور از `chunk_by` استفاده کنید.
- برای انتخاب عناصر با فاصله ثابت از `stride` استفاده کنید.
- برای دریافت Index و مقدار از `enumerate` استفاده کنید.
- برای پیمایش هم‌زمان چند دنباله از `zip` استفاده کنید.
- برای محاسبه مستقیم از چند Range از `zip_transform` استفاده کنید.
- برای ترکیب‌های چندگانه از `cartesian_product` استفاده کنید.
- برای اتصال Rangeهای داخلی همراه Separator از `join_with` استفاده کنید.
- برای انتقال عناصر از `as_rvalue` و عملیات Move مناسب استفاده کنید.
- برای دسترسی خواندنی از `as_const` استفاده کنید.
- برای تجمیع داده‌ها از الگوریتم‌های Fold استفاده کنید.

## جمع‌بندی

استاندارد ++C23 کتابخانه `std::ranges` را از یک چارچوب مناسب برای فیلتر و تبدیل داده‌ها به مجموعه‌ای بسیار کامل‌تر از ابزارهای پردازش دنباله‌ها تبدیل کرده است.

قابلیت‌هایی مانند `chunk`، `slide` و `chunk_by` تقسیم و گروه‌بندی داده‌ها را ساده می‌کنند. `zip`، `zip_transform` و `cartesian_product` پردازش چند Range را تسهیل می‌کنند. `enumerate` و `stride` کنترل پیمایش را بهبود می‌دهند و `as_rvalue` و `as_const` کنترل دقیق‌تری بر نحوه دسترسی به عناصر فراهم می‌کنند.

در کنار این موارد، الگوریتم‌های Fold، جست‌وجوی آخرین تطبیق، بررسی وجود مقدار و جابه‌جایی عناصر، نیاز به حلقه‌های دستی را کاهش می‌دهند.

بااین‌حال، استفاده حرفه‌ای از Ranges نیازمند توجه به طول عمر داده‌ها، مالکیت Viewها، Value Category، هزینه ارزیابی مجدد و پیچیدگی Iteratorها است. بهترین رویکرد آن است که از Pipelineهای خوانا استفاده شود، اما در نقاط حساس به کارایی، رفتار واقعی کد با Benchmark و تحلیل دقیق بررسی شود.

## منابع و مراجع

- [cppreference: C++23](https://en.cppreference.com/w/cpp/23)
- [cppreference: Ranges library](https://en.cppreference.com/w/cpp/ranges)
- [cppreference: Header `<ranges>`](https://en.cppreference.com/w/cpp/header/ranges)
- [WG21 P2442: Windowing range adaptors](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2442r1.html)
- [WG21 P2443: `views::chunk_by`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2443r1.html)
- [WG21 P2321: zip](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2321r2.html)
- [WG21 P2164: `views::enumerate`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2164r9.html)

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>