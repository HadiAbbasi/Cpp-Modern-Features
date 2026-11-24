<div align="center">

[🇺🇸 English](../../en/cpp23/std_views_zip.md) | [🇮🇷 فارسی](./std_views_zip.md)

</div>

---

# **معرفی&#x20;**`std::views::zip`**&#x20;در ++C23**

## **مقدمه**

در برنامه‌نویسی ++C، گاهی لازم است چند مجموعه داده را به‌صورت هم‌زمان پیمایش کنیم؛ به‌گونه‌ای که عنصر اول هر مجموعه با عنصر اول مجموعه‌های دیگر، عنصر دوم با عنصر دوم و به همین ترتیب با یکدیگر مرتبط شوند.

برای مثال، فرض کنید شناسه‌ها، نام‌ها و امتیازهای کاربران در سه `std::vector` جداگانه ذخیره شده‌اند. اگر بخواهیم اطلاعات هر کاربر را پردازش کنیم، باید عناصر متناظر این سه مجموعه را هم‌زمان بخوانیم.

پیش از ++C23، این کار معمولاً با استفاده از حلقه‌های مبتنی بر index، مدیریت چند iterator یا کتابخانه‌های جانبی انجام می‌شد.

قابلیت `std::views::zip` که در ++C23 به کتابخانه استاندارد اضافه شده است، این الگو را به‌شکل استاندارد و سازگار با `Ranges` فراهم می‌کند.

این قابلیت چند `range` را دریافت می‌کند و یک `view` تنبل می‌سازد که هر عنصر آن شامل عناصر متناظر rangeهای ورودی است.

از مهم‌ترین کاربردهای این قابلیت می‌توان به موارد زیر اشاره کرد:

- پیمایش هم‌زمان چند container
- پردازش داده‌های ذخیره‌شده در ساختارهای موازی
- ترکیب ستون‌های مختلف یک ساختار داده
- پردازش مختصات و بردارها
- همگام‌سازی چند جریان داده
- جایگزینی حلقه‌های indexمحور
- استفاده در pipelineهای مبتنی بر `Ranges`

## **معرفی مفهوم Zip**

### **رفتار اصلی**

تابع `std::views::zip` چند range را دریافت می‌کند و range جدیدی می‌سازد که هر عنصر آن از عناصر هم‌موقعیت rangeهای ورودی تشکیل شده است.

برای درک بهتر، مثال زیر را بررسی کنید:

```
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

int main() {
    std::vector<int> ids{10, 20, 30};

    std::vector<std::string> names{
        "Alice",
        "Bob",
        "Charlie"
    };

    for (auto [id, name] : std::views::zip(ids, names)) {
        std::cout << id << ": " << name << '\n';
    }
}
```

خروجی:

```
10: Alice
20: Bob
30: Charlie
```

در این مثال، دو مجموعه `ids` و `names` به تابع `std::views::zip` ارسال شده‌اند.

در هر مرحله از پیمایش، یک زوج از عناصر متناظر این مجموعه‌ها در اختیار حلقه قرار می‌گیرد.

به‌صورت مفهومی، عناصر خروجی ساختاری مشابه `std::tuple` دارند:

```
std::tuple<int&, std::string&>
```

البته نوع دقیق reference و value به نوع rangeهای ورودی وابسته است.

### **دلیل اضافه شدن این قابلیت**

در روش سنتی، برای پیمایش هم‌زمان چند مجموعه معمولاً از حلقه indexمحور استفاده می‌شود:

```
#include <iostream>
#include <string>
#include <vector>

int main() {
    std::vector<int> ids{10, 20, 30};

    std::vector<std::string> names{
        "Alice",
        "Bob",
        "Charlie"
    };

    for (std::size_t i = 0; i < ids.size(); ++i) {
        std::cout << ids[i] << ": "
                  << names[i] << '\n';
    }
}
```

این روش برای مثال‌های ساده قابل استفاده است، اما چند مشکل دارد:

- برنامه‌نویس باید index را به‌صورت دستی مدیریت کند.
- اندازه مجموعه‌ها باید به‌درستی مدیریت شود.
- احتمال دسترسی خارج از محدوده وجود دارد.
- اضافه شدن مجموعه‌های جدید، پیچیدگی حلقه را افزایش می‌دهد.
- ارتباط میان عناصر متناظر در کد به‌صورت صریح بیان نمی‌شود.
- ترکیب این روش با pipelineهای مبتنی بر `Ranges` دشوارتر است.

قابلیت `std::views::zip` این مشکلات را با ارائه یک abstraction استاندارد برطرف می‌کند.

در نتیجه، به‌جای مدیریت چند index یا iterator، می‌توان مستقیماً روی مجموعه‌ای از عناصر متناظر پیمایش کرد.

## **نحوه استفاده از std::views::zip**

### **هدر موردنیاز**

برای استفاده از این قابلیت باید هدر زیر را وارد برنامه کنید:

```
#include <ranges>
```

این قابلیت بخشی از کتابخانه استاندارد ++C23 است و به compiler و standard library دارای پشتیبانی مناسب از ++C23 نیاز دارد.

### **ارسال چند range**

تابع `std::views::zip` می‌تواند چند range مختلف را دریافت کند.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{1, 2, 3};

    std::vector<char> letters{
        'A',
        'B',
        'C'
    };

    std::vector<double> values{
        1.5,
        2.5,
        3.5
    };

    for (auto [number, letter, value] :
         std::views::zip(numbers, letters, values)) {

        std::cout << number << ' '
                  << letter << ' '
                  << value << '\n';
    }
}
```

خروجی:

```
1 A 1.5
2 B 2.5
3 C 3.5
```

در این مثال، هر عنصر خروجی شامل سه مقدار است.

تعداد rangeهای ورودی محدود به دو مجموعه نیست و می‌توان چندین range را در یک عملیات zip ترکیب کرد.

### **استفاده با انواع مختلف range**

یکی از ویژگی‌های مهم `std::views::zip` این است که ورودی‌ها مجبور نیستند از یک نوع container باشند.

برای مثال، می‌توان یک `std::vector` را با یک `std::array` ترکیب کرد.

```
#include <array>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{
        10, 20, 30
    };

    std::array<char, 3> letters{
        'A', 'B', 'C'
    };

    for (auto [number, letter] :
         std::views::zip(numbers, letters)) {

        std::cout << number << ' '
                  << letter << '\n';
    }
}
```

خروجی:

```
10 A
20 B
30 C
```

مهم این است که rangeهای ورودی الزامات موردنیاز `std::views::zip` را برآورده کنند؛ یکسان بودن نوع آن‌ها الزامی نیست.

## **رفتار طول و پایان پیمایش**

### **کوتاه‌ترین range تعیین‌کننده است**

یکی از مهم‌ترین ویژگی‌های `std::views::zip` نحوه مدیریت rangeهایی با اندازه متفاوت است.

طول خروجی zip برابر با طول کوتاه‌ترین range ورودی خواهد بود.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 4, 5
    };

    std::vector<char> letters{
        'A', 'B', 'C'
    };

    for (auto [number, letter] :
         std::views::zip(numbers, letters)) {

        std::cout << number << ' '
                  << letter << '\n';
    }
}
```

خروجی:

```
1 A
2 B
3 C
```

با وجود اینکه `numbers` پنج عنصر دارد، خروجی فقط سه عنصر خواهد داشت.

دلیل این رفتار آن است که پیمایش zip به محض رسیدن هرکدام از rangeهای ورودی به انتهای خود متوقف می‌شود.

به عبارت دیگر، طول خروجی برابر است با:

\\[ N\_{\text{zip}} = \min(N_1,N_2,\ldots,N_k) \\]

که در آن:

- \\(N_i\\) طول range شماره \\(i\\) است.
- \\(k\\) تعداد rangeهای ورودی است.

### **مدیریت اختلاف اندازه‌ها**

رفتار کوتاه‌ترین range در بسیاری از کاربردها مناسب است، اما همیشه رفتار مطلوبی نیست.

فرض کنید دو مجموعه شناسه و قیمت باید دقیقاً تعداد عناصر یکسانی داشته باشند.

اگر یکی از مجموعه‌ها ناقص باشد، ادامه پردازش بدون بررسی اندازه‌ها ممکن است باعث حذف خاموش بخشی از داده‌ها شود.

در چنین شرایطی بهتر است اندازه‌ها را پیش از zip کردن بررسی کنیم.

```
#include <cassert>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{
        1, 2, 3
    };

    std::vector<int> weights{
        10, 20, 30
    };

    assert(values.size() == weights.size());

    for (auto [value, weight] :
         std::views::zip(values, weights)) {

        // Process the corresponding values.
    }
}
```

در برنامه‌های production بهتر است به‌جای اتکای صرف به `assert`، در صورت نیاز خطای ورودی به‌شکل مناسبی مدیریت شود؛ زیرا `assert` در buildهایی که `NDEBUG` تعریف شده باشد ممکن است غیرفعال شود.

## **بررسی نوع عناصر خروجی**

### **ساختار tuple-like**

هر عنصر حاصل از پیمایش `zip_view` یک ساختار tuple-like است که عناصر متناظر rangeهای ورودی را در اختیار قرار می‌دهد.

برای مثال، اگر دو range شامل `int` و `double` باشند، نوع value حاصل از zip از نظر مفهومی مشابه زیر خواهد بود:

```
std::tuple<int, double>
```

اما نوع reference حاصل از dereference کردن iterator می‌تواند مشابه زیر باشد:

```
std::tuple<int&, double&>
```

این تفاوت اهمیت زیادی دارد؛ زیرا خروجی zip معمولاً به عناصر اصلی rangeها ارجاع می‌دهد و الزاماً کپی مستقلی از آن‌ها ایجاد نمی‌کند.

### **استفاده از Structured Binding**

یکی از کاربردی‌ترین روش‌های دریافت عناصر خروجی، استفاده از `Structured Binding` است.

این قابلیت از ++C17 در زبان ++C وجود دارد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{
        10, 20, 30
    };

    std::vector<int> values{
        100, 200, 300
    };

    for (auto [number, value] :
         std::views::zip(numbers, values)) {

        std::cout << number << ' '
                  << value << '\n';
    }
}
```

در هر دور حلقه، مقادیر متناظر در متغیرهای `number` و `value` قرار می‌گیرند.

### **تفاوت auto و auto&&**

در کدهای معمولی، انتخاب نوع متغیر حلقه اهمیت دارد.

برای نمونه:

```
for (auto [a, b] : std::views::zip(x, y)) {
    // Process elements.
}
```

در این حالت، شیء tuple-like حاصل از iterator در متغیر محلی ذخیره می‌شود.

با توجه به اینکه اعضای tuple معمولاً reference هستند، کپی شدن خود tuple لزوماً باعث کپی شدن داده‌های اصلی نمی‌شود.

برای generic code و کار با referenceهای حاصل از rangeها، استفاده از `auto&&` معمولاً انتخاب انعطاف‌پذیرتری است:

```
for (auto&& [a, b] : std::views::zip(x, y)) {
    // Process elements.
}
```

این روش امکان حفظ رفتار referenceای نتیجه پیمایش را فراهم می‌کند.

### **ساخت کپی مستقل از عناصر**

اگر هدف ساخت کپی مستقل از مقادیر باشد، باید این کار را به‌صورت صریح انجام داد.

برای مثال:

```
#include <ranges>
#include <tuple>
#include <vector>

int main() {
    std::vector<int> first{
        1, 2, 3
    };

    std::vector<int> second{
        4, 5, 6
    };

    for (auto&& item :
         std::views::zip(first, second)) {

        auto [a, b] = std::make_tuple(
            std::get<0>(item),
            std::get<1>(item)
        );

        // a and b are independent values.
    }
}
```

در این مثال، `std::make_tuple` مقادیر را به‌صورت value ذخیره می‌کند.

بنابراین تغییر متغیرهای `a` و `b` تأثیری روی containerهای اصلی ندارد.

## **تغییر عناصر از طریق Zip**

### **امکان تغییر داده‌های اصلی**

از آنجا که عناصر خروجی zip معمولاً reference هستند، در صورت قابل‌نوشتن بودن rangeهای ورودی می‌توان داده‌های اصلی را از طریق zip تغییر داد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3
    };

    std::vector<int> increments{
        10, 20, 30
    };

    for (auto&& [number, increment] :
         std::views::zip(numbers, increments)) {

        number += increment;
    }

    for (int number : numbers) {
        std::cout << number << ' ';
    }
}
```

خروجی:

```
11 22 33
```

در این مثال، تغییر `number` مستقیماً روی عنصر متناظر در `numbers` اعمال می‌شود.

این ویژگی برای پردازش هم‌زمان چند مجموعه بسیار کاربردی است.

### **استفاده برای به‌روزرسانی چند مجموعه**

فرض کنید دو مجموعه شامل مختصات X و Y هستند و می‌خواهیم هر دو را تغییر دهیم.

```
#include <ranges>
#include <vector>

int main() {
    std::vector<double> x{
        1.0, 2.0, 3.0
    };

    std::vector<double> y{
        4.0, 5.0, 6.0
    };

    for (auto&& [px, py] : std::views::zip(x, y)) {
        px *= 2.0;
        py *= 2.0;
    }
}
```

در این مثال، تمام مختصات با استفاده از یک حلقه به‌روزرسانی می‌شوند.

## **بررسی قابلیت‌های Iterator**

### **دسته‌بندی Iterator**

قابلیت‌های iterator مربوط به `zip_view` به قابلیت‌های rangeهای ورودی وابسته است.

به‌صورت کلی، `zip_view` حداقل یک `input_range` است.

قابلیت‌های قوی‌تر زمانی در دسترس هستند که تمام rangeهای ورودی مفهوم مربوطه را مدل کنند.

| قابلیت                | شرط                                     |
| --------------------- | --------------------------------------- |
| `input_range`         | تمام ورودی‌ها input range باشند         |
| `forward_range`       | تمام ورودی‌ها forward range باشند       |
| `bidirectional_range` | تمام ورودی‌ها bidirectional range باشند |
| `random_access_range` | تمام ورودی‌ها random access range باشند |
| `sized_range`         | تمام ورودی‌ها sized range باشند         |

بنابراین نمی‌توان فرض کرد که هر خروجی `std::views::zip` قابلیت دسترسی تصادفی یا پیمایش معکوس دارد.

### **قابلیت پیمایش معکوس**

اگر تمام rangeهای ورودی `bidirectional_range` باشند، iterator خروجی نیز قابلیت پیمایش معکوس خواهد داشت.

برای مثال:

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3
    };

    std::vector<int> values{
        10, 20, 30
    };

    auto zipped = std::views::zip(numbers, values);

    auto it = zipped.end();

    --it;

    auto [number, value] = *it;

    std::cout << number << ' '
              << value << '\n';
}
```

خروجی:

```
3 30
```

در این مثال، هر دو ورودی از نوع `std::vector` هستند و قابلیت پیمایش معکوس دارند.

### **قابلیت Random Access**

اگر تمام rangeهای ورودی `random_access_range` باشند، خروجی zip نیز این مفهوم را مدل می‌کند.

در نتیجه می‌توان از قابلیت‌هایی مانند دسترسی indexمحور استفاده کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> ids{
        10, 20, 30
    };

    std::vector<char> codes{
        'A', 'B', 'C'
    };

    auto zipped = std::views::zip(ids, codes);

    auto [id, code] = zipped[1];

    std::cout << id << ' '
              << code << '\n';
}
```

خروجی:

```
20 B
```

در این مثال، دسترسی تصادفی به عنصر دوم خروجی انجام شده است.

این قابلیت برای rangeهایی که تمام ورودی‌های آن‌ها random access هستند مفید است.

### **مفهوم Sized Range**

اگر تمام rangeهای ورودی `sized_range` باشند، خروجی zip نیز `sized_range` خواهد بود.

در این شرایط می‌توان اندازه خروجی را با استفاده از `size` به دست آورد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> first{
        1, 2, 3, 4
    };

    std::vector<int> second{
        10, 20
    };

    auto zipped = std::views::zip(first, second);

    std::cout << zipped.size() << '\n';
}
```

خروجی:

```
2
```

توجه کنید که مقدار بازگشتی برابر با کمترین اندازه ورودی‌ها است.

### **مفهوم Common Range**

خروجی `zip_view` همیشه الزاماً `common_range` نیست.

در یک `common_range`، نوع iterator و sentinel انتهایی یکسان است.

اما در `zip_view` ممکن است نوع iterator و sentinel متفاوت باشد.

در استاندارد، شرایط مشخصی برای common بودن خروجی در نظر گرفته شده است؛ از جمله حالت‌هایی که تمام rangeهای ورودی common هستند و همچنین برخی ترکیب‌های خاص از random access و sized range.

در generic code بهتر است به‌جای فرض یکسان بودن iterator و sentinel، از abstractionهای استاندارد مبتنی بر `range` استفاده شود.

## **استفاده از Zip در Pipeline**

یکی از مهم‌ترین مزیت‌های `std::views::zip` سازگاری آن با سایر range adaptorهای استاندارد است.

به کمک این قابلیت می‌توان چند عملیات را در قالب یک pipeline ترکیب کرد.

### **ترکیب با Filter**

می‌توان یک range را با استفاده از `views::filter` فیلتر کرد و سپس آن را با range دیگری ترکیب کرد.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> numbers{
        1, 2, 3, 4, 5
    };

    std::vector<int> factors{
        10, 20, 30, 40, 50
    };

    auto filtered = numbers
        | std::views::filter([](int value) {
              return value > 2;
          });

    for (auto [number, factor] :
         std::views::zip(filtered, factors)) {

        std::cout << number << ' '
                  << factor << '\n';
    }
}
```

خروجی:

```
3 10
4 20
5 30
```

نکته بسیار مهم این است که zip عناصر را بر اساس ترتیب پیمایش rangeها جفت می‌کند، نه بر اساس index اصلی container.

در این مثال، اولین عنصر خروجی `filtered` با اولین عنصر `factors` جفت می‌شود.

بنابراین مقدار `3` با `10` ترکیب شده است، نه با `30`.

این رفتار یکی از مهم‌ترین نکاتی است که هنگام ترکیب `filter` و `zip` باید در نظر گرفته شود.

### **ترکیب با Transform**

قابلیت `views::transform` برای تبدیل عناصر یک range استفاده می‌شود.

می‌توان خروجی zip را با transform ترکیب کرد تا روی عناصر متناظر عملیات محاسباتی انجام شود.

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> prices{
        100, 200, 300
    };

    std::vector<int> quantities{
        2, 3, 4
    };

    auto totals = std::views::zip(prices, quantities)
        | std::views::transform([](auto pair) {
              auto [price, quantity] = pair;
              return price * quantity;
          });

    for (int total : totals) {
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

در این مثال، حاصل‌ضرب قیمت و تعداد هر محصول محاسبه می‌شود.

از آنجا که `views::transform` نیز یک view تنبل تولید می‌کند، محاسبات هنگام پیمایش انجام می‌شوند.

## **معرفی std::views::zip_transform**

در ++C23 علاوه بر `std::views::zip`، قابلیت دیگری به نام `std::views::zip_transform` نیز معرفی شده است.

این قابلیت برای زمانی طراحی شده که بخواهیم مستقیماً تابعی را روی عناصر متناظر چند range اعمال کنیم.

تفاوت اصلی این دو قابلیت به نوع خروجی آن‌ها مربوط می‌شود:

- `std::views::zip` یک tuple-like object از عناصر متناظر تولید می‌کند.
- `std::views::zip_transform` تابع مشخص‌شده را روی عناصر متناظر اعمال می‌کند و نتیجه آن تابع را تولید می‌کند.

### **مثال استفاده**

```
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> first{
        1, 2, 3
    };

    std::vector<int> second{
        10, 20, 30
    };

    auto sums = std::views::zip_transform(
        [](int x, int y) {
            return x + y;
        },
        first,
        second
    );

    for (int sum : sums) {
        std::cout << sum << ' ';
    }
}
```

خروجی:

```
11 22 33
```

در این مثال، تابع lambda مستقیماً دو عنصر متناظر را دریافت می‌کند.

### **چه زمانی از Zip Transform استفاده کنیم؟**

اگر هدف فقط محاسبه یک نتیجه از چند range باشد، `zip_transform` معمولاً انتخاب مناسب‌تری است.

برای مثال، در موارد زیر می‌توان از آن استفاده کرد:

- محاسبه مجموع دو آرایه
- محاسبه حاصل‌ضرب عناصر متناظر
- ترکیب داده‌های مالی
- محاسبه فاصله بین نقاط
- تولید خروجی بر اساس چند ورودی مستقل

اما اگر لازم باشد به عناصر ورودی به‌صورت جداگانه دسترسی داشته باشیم یا آن‌ها را تغییر دهیم، `std::views::zip` انعطاف بیشتری در اختیارمان قرار می‌دهد.

## **تفاوت Zip با روش‌های دیگر**

### **مقایسه با حلقه Index-Based**

روش indexمحور برای زمانی مناسب است که خود index بخشی از منطق برنامه باشد.

برای مثال، اگر بخواهیم عنصر موقعیت پنجم را با عنصر موقعیت دهم یک مجموعه مقایسه کنیم، حلقه indexمحور ممکن است انتخاب مناسبی باشد.

اما برای پیمایش هم‌زمان عناصر متناظر چند مجموعه، `std::views::zip` معمولاً خواناتر و ساده‌تر است.

### **مقایسه با std::pair و std::tuple**

ساختارهای `std::pair` و `std::tuple` برای نگهداری چند مقدار در یک شیء استفاده می‌شوند.

در مقابل، `std::views::zip` یک range جدید از چند range موجود تولید می‌کند.

تفاوت مهم این است که zip به‌صورت lazy عمل می‌کند و برای ایجاد خروجی، لزوماً container جدیدی نمی‌سازد.

برای مثال:

```
auto zipped = std::views::zip(first, second);
```

در این حالت، داده‌های اصلی همچنان در containerهای اولیه قرار دارند.

### **مقایسه با std::views::adjacent**

در ++C23 قابلیت دیگری به نام `std::views::adjacent` نیز وجود دارد.

این قابلیت برای تولید گروه‌هایی از عناصر متوالی یک range واحد طراحی شده است.

برای مثال، فرض کنید range زیر را داریم:

```
1 2 3 4 5
```

استفاده از `adjacent<2>` می‌تواند گروه‌های زیر را تولید کند:

```
(1, 2)
(2, 3)
(3, 4)
(4, 5)
```

اما `zip` برای ترکیب چند range مستقل استفاده می‌شود.

بنابراین این دو قابلیت اهداف متفاوتی دارند.

## **مدیریت عمر و مالکیت Rangeها**

یکی از موضوعات مهم هنگام استفاده از `std::views::zip`، مدیریت طول عمر rangeهای ورودی است.

### **مالکیت Containerهای اصلی**

وقتی یک container به‌صورت lvalue به `std::views::zip` ارسال می‌شود، zip معمولاً مالک آن container نمی‌شود.

در این حالت، view حاصل به داده‌های container اصلی وابسته است.

برای مثال:

```
#include <ranges>
#include <vector>

int main() {
    std::vector<int> first{
        1, 2, 3
    };

    std::vector<int> second{
        4, 5, 6
    };

    auto zipped = std::views::zip(first, second);
}
```

در این مثال، containerها باید تا زمانی که `zipped` استفاده می‌شود معتبر و زنده باشند.

### **خطر Dangling View**

مثال زیر را در نظر بگیرید:

```
#include <ranges>
#include <vector>

auto make_bad_view() {
    std::vector<int> first{
        1, 2, 3
    };

    std::vector<int> second{
        4, 5, 6
    };

    return std::views::zip(first, second);
}
```

در این مثال، هر دو vector هنگام خروج از تابع از بین می‌روند.

بنابراین view بازگشتی به اشیایی ارجاع می‌دهد که دیگر عمر ندارند.

استفاده از این view می‌تواند باعث رفتار تعریف‌نشده شود.

در چنین شرایطی باید مالکیت داده‌ها یا طول عمر containerها به‌درستی مدیریت شود.

### **استفاده از Temporary Rangeها**

قواعد `viewable_range` و `views::all` تعیین می‌کنند که یک range چگونه می‌تواند وارد view شود.

در ++C20 و نسخه‌های بعدی، `views::all` می‌تواند برای برخی rangeهای موقت، یک `owning_view` ایجاد کند.

برای مثال:

```
#include <ranges>
#include <vector>

int main() {
    auto zipped = std::views::zip(
        std::vector<int>{1, 2, 3},
        std::vector<int>{4, 5, 6}
    );

    for (auto [a, b] : zipped) {
        // Both temporary vectors are owned by the resulting views.
    }
}
```

در این مثال، rangeهای موقت از بین نمی‌روند؛ زیرا viewهای حاصل مالک آن‌ها هستند.

با این حال، نباید تصور کرد که zip همیشه مالک تمام ورودی‌ها است.

رفتار دقیق به نوع ورودی و value category آن وابسته است.

### **مفهوم Borrowed Range**

`zip_view` در صورتی `borrowed_range` محسوب می‌شود که تمام viewهای underlying آن این مفهوم را مدل کنند.

این ویژگی به اعتبار iteratorها پس از نابودی خود view مربوط است.

نکته مهم این است که `borrowed_range` بودن یک view به معنای زنده ماندن containerهایی که به آن‌ها ارجاع داده شده نیست.

بنابراین borrowed بودن نباید با مالکیت یا مدیریت طول عمر داده‌ها اشتباه گرفته شود.

## **بررسی عملیات پیشرفته Iterator**

### **عملیات iter_swap**

iterator مربوط به `zip_view` از عملیات `iter_swap` پشتیبانی می‌کند؛ البته با رعایت محدودیت‌های مفهومی و قابلیت‌های rangeهای underlying.

این عملیات برای جابه‌جایی هماهنگ عناصر متناظر در rangeهای ورودی کاربرد دارد.

به بیان ساده، اگر دو موقعیت از range حاصل zip جابه‌جا شوند، عناصر متناظر در تمام rangeهای underlying نیز به‌صورت هماهنگ جابه‌جا می‌شوند.

این رفتار برای سازگاری با الگوریتم‌های استانداردی که عناصر range را جابه‌جا می‌کنند اهمیت دارد.

### **عملیات iter_move**

عملیات `iter_move` نیز برای iterator مربوط به zip تعریف شده است.

هدف آن فراهم کردن رفتار مناسب move برای عناصر متناظر rangeهای underlying است.

این موضوع در پیاده‌سازی الگوریتم‌های generic و کار با proxy referenceها اهمیت دارد.

در برنامه‌نویسی روزمره معمولاً نیازی به فراخوانی مستقیم این عملیات نیست؛ اما آشنایی با آن برای درک رفتار iteratorهای zip مفید است.

### **محدودیت‌های عملیات Iterator**

نباید تصور کرد که تمام عملیات iterator در همه خروجی‌های zip قابل استفاده هستند.

برای مثال:

- عملیات decrement به bidirectional بودن تمام ورودی‌ها وابسته است.
- عملیات دسترسی تصادفی به random access بودن تمام ورودی‌ها وابسته است.
- عملیات مربوط به اندازه به sized بودن تمام ورودی‌ها وابسته است.
- قابلیت نوشتن به reference type و قابلیت نوشتن rangeهای ورودی وابسته است.

این محدودیت‌ها در طراحی templateهای generic اهمیت زیادی دارند.

## **نکات مهم در کدنویسی حرفه‌ای**

### **توجه به Filter و ترتیب عناصر**

هنگام ترکیب `filter` با zip باید به ترتیب عناصر توجه ویژه‌ای داشت.

zip هیچ اطلاعی از index اصلی عناصر ندارد و فقط ترتیب پیمایش rangeها را ملاک قرار می‌دهد.

بنابراین اگر ارتباط داده‌ها بر اساس index اصلی است، فیلتر کردن مستقل یکی از مجموعه‌ها ممکن است ارتباط مورد انتظار را از بین ببرد.

### **توجه به Invalidation**

`zip_view` تضمین مستقلی برای اعتبار iteratorهای containerهای زیرین ایجاد نمی‌کند.

اگر عملیاتی روی container اصلی باعث invalidation شدن iteratorها شود، iteratorهای مربوط به zip نیز ممکن است نامعتبر شوند.

برای مثال، افزودن عنصر به `std::vector` ممکن است باعث reallocation شود.

در این شرایط iteratorهای قبلی آن vector معتبر نخواهند بود.

بنابراین نباید هنگام نگهداری iteratorهای zip، قواعد invalidation مربوط به containerهای اصلی را نادیده گرفت.

### **توجه به Const**

const بودن خود view و const بودن عناصر underlying دو موضوع متفاوت هستند.

قابلیت پیمایش نسخه const یک zip به این بستگی دارد که rangeهای underlying در حالت const نیز الزامات مربوط به `range` را برآورده کنند.

همچنین const بودن view الزاماً به این معنا نیست که داده‌های اصلی در تمام شرایط غیرقابل تغییر هستند.

برای تشخیص رفتار واقعی باید نوع reference عناصر و نوع rangeهای ورودی بررسی شود.

### **توجه به تفاوت Reference و Value Type**

در برنامه‌نویسی generic، تفاوت میان دو مفهوم زیر اهمیت زیادی دارد:

- `range_reference_t`
- `range_value_t`

اولی نوعی است که هنگام پیمایش به‌عنوان reference عنصر دریافت می‌شود.

دومی نوع value مربوط به عنصر range است.

در zip این تفاوت اهمیت بیشتری دارد؛ زیرا نوع reference خروجی معمولاً شامل چند reference در قالب یک ساختار tuple-like است.

### **پرهیز از نگهداری Viewهای وابسته به داده موقت**

اگر یک view به containerهای محلی یا اشیایی با طول عمر محدود وابسته باشد، نگهداری آن پس از پایان عمر داده‌های اصلی خطرناک است.

در طراحی APIهایی که view برمی‌گردانند، باید به مالکیت و طول عمر ورودی‌ها توجه ویژه داشت.

## **محدودیت‌ها و موارد نامناسب**

با وجود کاربردهای فراوان، `std::views::zip` برای تمام مسائل ترکیب داده مناسب نیست.

موارد زیر از مهم‌ترین محدودیت‌های آن هستند:

- اگر لازم باشد تمام عناصر تمام rangeها، حتی پس از پایان کوتاه‌ترین range، پردازش شوند، zip انتخاب مناسبی نیست.
- اگر اندازه تمام ورودی‌ها باید برابر باشد، باید اعتبارسنجی جداگانه انجام شود.
- اگر به خروجی مستقل و مالک نیاز دارید، باید داده‌ها را در یک container مناسب ذخیره کنید.
- اگر ارتباط عناصر بر اساس شناسه یا کلید است، zip ابزار مناسبی نیست.
- اگر ترتیب پیمایش rangeها متفاوت است، zip صرفاً بر اساس ترتیب موجود جفت‌سازی می‌کند.
- اگر rangeهای ورودی طول عمر مناسبی نداشته باشند، خروجی zip نیز ممکن است نامعتبر شود.
- اگر rangeهای ورودی قابلیت‌های متفاوتی داشته باشند، خروجی تنها قابلیت‌های مشترک و قابل پشتیبانی آن‌ها را خواهد داشت.

برای مثال، اگر دو مجموعه باید بر اساس شناسه کاربر با یکدیگر ترکیب شوند، استفاده از `zip` به‌تنهایی کافی نیست.

در چنین شرایطی باید از الگوریتم‌های مبتنی بر کلید، `std::map`، `std::unordered_map` یا روش‌های مناسب دیگر استفاده کرد.

## **بررسی سازگاری کامپایلر و کتابخانه استاندارد**

### **نسخه استاندارد موردنیاز**

قابلیت `std::views::zip` بخشی از کتابخانه استاندارد ++C23 است.

برای استفاده از آن باید هدر زیر در دسترس باشد:

```
#include <ranges>
```

نکته مهم این است که فعال کردن حالت ++C23 در compiler به‌تنهایی تضمین نمی‌کند که کتابخانه استاندارد مورد استفاده تمام قابلیت‌های این نسخه را پیاده‌سازی کرده باشد.

بنابراین باید پشتیبانی compiler و standard library به‌صورت جداگانه بررسی شود.

### **بررسی Feature-Test Macro**

برای تشخیص پشتیبانی از قابلیت‌های zip می‌توان از macro استاندارد زیر استفاده کرد:

```
#include <ranges>

#if defined(__cpp_lib_ranges_zip) && \
    __cpp_lib_ranges_zip >= 202110L

    // std::views::zip is available.

#else

    #error "C++23 zip views are not available."

#endif
```

مقدار feature-test macro مربوط به این قابلیت برابر با `202110L` است.

این روش برای تشخیص قابلیت‌های کتابخانه استاندارد معمولاً دقیق‌تر از بررسی صرف شماره نسخه compiler است.

## **مثال کاربردی کامل**

در مثال زیر، سه range شامل شناسه، نام و امتیاز کاربران با یکدیگر ترکیب می‌شوند.

همچنین پیش از پردازش، اندازه مجموعه‌ها بررسی می‌شود تا در صورت وجود داده ناقص، پردازش به‌صورت خاموش ادامه پیدا نکند.

```
#include <iostream>
#include <ranges>
#include <stdexcept>
#include <string>
#include <vector>

int main() {
    std::vector<int> ids{
        101, 102, 103
    };

    std::vector<std::string> names{
        "Nina",
        "Omid",
        "Sara"
    };

    std::vector<double> scores{
        18.5,
        16.0,
        19.25
    };

    if (ids.size() != names.size() ||
        ids.size() != scores.size()) {

        throw std::runtime_error(
            "Input ranges must have equal sizes"
        );
    }

    for (auto&& [id, name, score] :
         std::views::zip(ids, names, scores)) {

        std::cout << id << " | "
                  << name << " | "
                  << score << '\n';
    }
}
```

خروجی:

```
101 | Nina | 18.5
102 | Omid | 16
103 | Sara | 19.25
```

این الگو برای موارد زیر کاربرد دارد:

- پردازش ستون‌های یک جدول
- نگهداری داده‌ها در چند container موازی
- ترکیب شناسه و اطلاعات مربوط به آن
- پردازش اطلاعات کاربران
- پردازش داده‌های علمی و محاسباتی

## **توصیه‌های طراحی و استفاده**

برای استفاده حرفه‌ای از `std::views::zip` بهتر است نکات زیر رعایت شوند:

- برای پیمایش ساده چند range متناظر، از zip به‌جای حلقه‌های indexمحور غیرضروری استفاده کنید.
- اگر اندازه rangeها باید برابر باشد، پیش از پردازش آن را بررسی کنید.
- برای تغییر داده‌های اصلی، از bindingهای مناسب reference مانند `auto&&` استفاده کنید.
- هنگام ترکیب zip با filter، به تغییر ترتیب و موقعیت عناصر توجه کنید.
- اگر فقط به نتیجه محاسبه روی عناصر متناظر نیاز دارید، `zip_transform` را نیز بررسی کنید.
- در APIهای مبتنی بر view، طول عمر و مالکیت rangeهای ورودی را به‌دقت مدیریت کنید.
- در templateهای generic، قابلیت‌های واقعی iterator را با Concepts بررسی کنید.
- از فرض `random_access_range` یا `common_range` بودن تمام خروجی‌های zip خودداری کنید.
- برای ارتباط داده‌ها بر اساس کلید، از ساختارهای داده و الگوریتم‌های مناسب‌تر استفاده کنید.

## **جمع‌بندی**

قابلیت `std::views::zip` در ++C23 راهکاری استاندارد، lazy و ترکیب‌پذیر برای پیمایش هم‌زمان چند range فراهم می‌کند.

هر عنصر خروجی از عناصر هم‌موقعیت rangeهای ورودی ساخته می‌شود و طول خروجی به کوتاه‌ترین range محدود خواهد بود.

از آنجا که عناصر خروجی معمولاً reference هستند، می‌توان در شرایط مناسب داده‌های اصلی را مستقیماً تغییر داد؛ بنابراین نباید zip را با ساخت یک container جدید اشتباه گرفت.

مهم‌ترین نکات حرفه‌ای هنگام استفاده از این قابلیت عبارت‌اند از:

- مدیریت اختلاف اندازه rangeها
- شناخت رفتار referenceها و value typeها
- توجه به دسته‌بندی iterator
- مدیریت طول عمر و مالکیت داده‌ها
- توجه به اثر filter بر ترتیب عناصر
- رعایت قواعد invalidation
- انتخاب مناسب میان `zip` و `zip_transform`

در نهایت، `std::views::zip` یکی از قابلیت‌های کاربردی ++C23 برای نوشتن کدهای خواناتر، قابل ترکیب‌تر و سازگار با رویکرد مدرن `Ranges` است.

## **منابع و مراجع**

برای مطالعه جزئیات استاندارد و بررسی دقیق‌تر رفتار این قابلیت، منابع زیر پیشنهاد می‌شوند:

- [استاندارد ++C23 - بخش Zip View](https://timsong-cpp.github.io/cppwp/std23/range.zip?utm_source=chatgpt.com)
- [cppreference - std::ranges::zip_view](https://en.cppreference.com/w/cpp/ranges/zip_view?utm_source=chatgpt.com)
- [cppreference - zip_view::iterator](https://en.cppreference.com/w/cpp/ranges/zip_view/iterator?utm_source=chatgpt.com)
- [cppreference - std::views::zip_transform](https://en.cppreference.com/w/cpp/ranges/zip_transform_view?utm_source=chatgpt.com)
- [cppreference - C++ Ranges Library](https://en.cppreference.com/w/cpp/ranges?utm_source=chatgpt.com)
- [WG21 - P2321R2: zip](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2321r2.html?utm_source=chatgpt.com)

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>