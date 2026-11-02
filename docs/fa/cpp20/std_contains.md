<div align="center">

[🇺🇸 English](../../en/cpp20/std_contains.md) | [🇮🇷 فارسی](./std_contains.md)

</div>

---

# معرفی std::contains در ++C20

## مقدمه

در برنامه‌نویسی ++C، بررسی وجود یک مقدار در یک مجموعه از عملیات‌های رایج است. برای مثال، ممکن است لازم باشد پیش از درج یک شناسه از تکراری نبودن آن مطمئن شویم، وجود یک کلید در `std::map` را بررسی کنیم یا مشخص کنیم یک مقدار قبلاً در مجموعه‌ای از مقادیر ثبت شده است یا نه.

در نسخه‌های پیشین زبان، این کار معمولاً با تابع عضو `find()` و مقایسه نتیجه با `end()` انجام می‌شد. استاندارد ++C20 تابع عضو `contains()` را برای containerهای انجمنی استاندارد اضافه کرد تا بیان این قصد ساده‌تر و خواناتر باشد.

نکته مهم این است که `std::contains` نام یک تابع آزاد عمومی در فضای نام `std` نیست. منظور معمولاً تابع عضو `contains()` در containerهایی مانند `std::set`، `std::map` و نسخه‌های unordered آن‌ها است. همچنین قابلیت مشابهی برای `std::string` و الگوریتم‌های ranges در استانداردهای بعدی اضافه شده است که باید از قابلیت ++C20 تفکیک شود.

## مفهوم و هدف std::contains

تابع عضو `contains()` مشخص می‌کند آیا container دارای عنصری با کلید موردنظر است یا خیر. خروجی آن از نوع `bool` است:

- اگر کلید موردنظر در container وجود داشته باشد، مقدار `true` برگردانده می‌شود.
- اگر کلید وجود نداشته باشد، مقدار `false` برگردانده می‌شود.

این تابع برای زمانی طراحی شده است که فقط پاسخ پرسش «آیا این کلید وجود دارد؟» اهمیت دارد و نیازی به دسترسی به iterator عنصر پیدا‌شده نیست.

### نمونه ساده

```cpp
#include <iostream>
#include <set>

int main() {
    const std::set<int> numbers{10, 20, 30, 40};

    std::cout << std::boolalpha;
    std::cout << numbers.contains(20) << '\n';
    std::cout << numbers.contains(25) << '\n';
}
```

خروجی:

```text
true
false
```

در این مثال، تابع `contains()` مستقیماً نتیجه جست‌وجو را به شکل یک مقدار منطقی برمی‌گرداند.

## containerهای پشتیبانی‌شده در ++C20

قابلیت `contains()` در ++C20 برای containerهای انجمنی استاندارد ارائه شده است.

| container | نوع جست‌وجو | پیچیدگی معمول |
|---|---|---|
| `std::set` | کلید یکتا و مرتب | `O(log n)` |
| `std::multiset` | کلید مرتب با امکان تکرار | `O(log n)` |
| `std::map` | کلید یکتا و مرتب | `O(log n)` |
| `std::multimap` | کلید مرتب با امکان تکرار | `O(log n)` |
| `std::unordered_set` | کلید یکتا و بدون ترتیب | میانگین `O(1)`، بدترین حالت `O(n)` |
| `std::unordered_multiset` | کلید تکراری و بدون ترتیب | میانگین `O(1)`، بدترین حالت `O(n)` |
| `std::unordered_map` | کلید یکتا و بدون ترتیب | میانگین `O(1)`، بدترین حالت `O(n)` |
| `std::unordered_multimap` | کلید تکراری و بدون ترتیب | میانگین `O(1)`، بدترین حالت `O(n)` |

در جدول بالا، `n` تعداد عناصر container است. پیچیدگی containerهای ordered از ساختار درختی متوازن آن‌ها ناشی می‌شود؛ در مقابل، containerهای unordered از hashing استفاده می‌کنند و عملکرد آن‌ها به کیفیت توزیع hash و تعداد collisionها وابسته است.

### containerهایی که این تابع را ندارند

تابع عضو `contains()` در ++C20 به همه containerهای استاندارد اضافه نشده است. برای نمونه، containerهای ترتیبی زیر چنین تابع عضوی ندارند:

- `std::vector`
- `std::array`
- `std::deque`
- `std::list`
- `std::forward_list`

برای این containerها می‌توان از `std::find()` یا در ++C20 از `std::ranges::find()` استفاده کرد. الگوریتم `std::ranges::contains()` متعلق به ++C23 است، نه ++C20.

## نحوه استفاده و سربارهای تابع

امضای مفهومی تابع برای containerهای انجمنی به این شکل است:

```cpp
bool contains(const key_type& key) const;
```

این امضا نشان می‌دهد که تابع:

- کلید مورد جست‌وجو را دریافت می‌کند.
- وضعیت container را تغییر نمی‌دهد و `const` است.
- فقط یک مقدار `bool` برمی‌گرداند.
- iterator یا reference عنصر را در اختیار فراخواننده قرار نمی‌دهد.

در صورت فعال بودن heterogeneous lookup، سربار قالبی دیگری نیز در دسترس خواهد بود که در ادامه توضیح داده می‌شود.

### بررسی وجود کلید در std::map

در `std::map`، جست‌وجو بر اساس کلید انجام می‌شود، نه مقدار ذخیره‌شده در زوج کلید و مقدار.

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    const std::map<int, std::string> users{
        {100, "Alice"},
        {200, "Bob"},
        {300, "Charlie"}
    };

    if (users.contains(200)) {
        std::cout << "User exists\n";
    }
}
```

در این مثال، عدد `200` به‌عنوان کلید بررسی می‌شود. تابع `contains()` مقدار متناظر با کلید را برنمی‌گرداند؛ فقط وجود یا نبود آن کلید را گزارش می‌کند.

### بررسی وجود مقدار در std::set

در `std::set`، خود عنصر نقش کلید را دارد؛ بنابراین جست‌وجو مستقیماً روی مقدار انجام می‌شود.

```cpp
#include <iostream>
#include <set>

int main() {
    const std::set<int> allowedPorts{80, 443, 8080};

    const int requestedPort = 443;

    if (allowedPorts.contains(requestedPort)) {
        std::cout << "Port is allowed\n";
    }
}
```

### بررسی وجود کلید در containerهای unordered

در containerهای unordered نیز رابط کاربری یکسان است، اما عملکرد جست‌وجو بر پایه hash است.

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    const std::unordered_map<std::string, int> inventory{
        {"keyboard", 12},
        {"mouse", 25},
        {"monitor", 8}
    };

    if (inventory.contains("mouse")) {
        std::cout << "Item exists\n";
    }
}
```

ترتیب عناصر در `std::unordered_map` تضمین نشده است؛ با این حال، این موضوع روی معنای `contains()` اثری ندارد.

## مقایسه contains با find

پیش از ++C20، الگوی رایج بررسی وجود عنصر چنین بود:

```cpp
if (container.find(key) != container.end()) {
    // The key exists
}
```

در ++C20 می‌توان همان قصد را به شکل زیر نوشت:

```cpp
if (container.contains(key)) {
    // The key exists
}
```

هر دو روش برای بررسی وجود کلید معتبرند، اما هدف و خروجی آن‌ها یکسان نیست.

| ویژگی | `contains()` | `find()` |
|---|---|---|
| خروجی | `bool` | iterator |
| بیان مستقیم وجود عنصر | بله | با مقایسه با `end()` |
| دسترسی به عنصر پیدا‌شده | خیر | بله |
| مناسب برای بررسی صرف وجود | بسیار مناسب | معتبر، اما غیرمستقیم‌تر |
| مناسب برای خواندن یا تغییر عنصر پیدا‌شده | خیر | بله |

### چه زمانی contains انتخاب بهتری است؟

زمانی از `contains()` استفاده کنید که فقط به نتیجه منطقی جست‌وجو نیاز دارید؛ برای مثال:

- اعتبارسنجی وجود یک شناسه در مجموعه شناسه‌های مجاز
- جلوگیری از پردازش دوباره یک کلید
- بررسی وجود تنظیمات یا گزینه‌ای در یک `std::map`
- کنترل عضویت یک عنصر در مجموعه‌ای از مقادیر

### چه زمانی find مناسب‌تر است؟

اگر پس از جست‌وجو لازم است به عنصر دسترسی داشته باشید، `find()` انتخاب مناسب‌تری است.

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<int, std::string> users{
        {100, "Alice"},
        {200, "Bob"}
    };

    const auto it = users.find(200);

    if (it != users.end()) {
        std::cout << it->second << '\n';
    }
}
```

در این نمونه، استفاده از `contains()` به‌تنهایی کافی نیست؛ زیرا برای دریافت مقدار باید جست‌وجوی دیگری انجام شود.

### پرهیز از دو بار جست‌وجو

الگوی زیر معمولاً انتخاب خوبی نیست:

```cpp
if (users.contains(id)) {
    const auto it = users.find(id);
    std::cout << it->second << '\n';
}
```

در این حالت، کلید دو بار جست‌وجو می‌شود. اگر هدف دسترسی به عنصر است، یک بار `find()` فراخوانی کنید و iterator را بررسی کنید.

## جست‌وجوی ناهمگون یا Heterogeneous Lookup

یکی از قابلیت‌های مهم containerهای انجمنی، امکان جست‌وجو با نوعی متفاوت از `key_type` است؛ به شرط آن‌که comparator یا توابع hash و برابری container برای این کار پیکربندی شده باشند. این قابلیت با عنوان `Heterogeneous Lookup` شناخته می‌شود.

برای مثال، فرض کنید کلیدهای یک `std::set` از نوع `std::string` هستند، اما می‌خواهید با یک `std::string_view` جست‌وجو کنید. در حالت معمول، ممکن است ساخت یک `std::string` موقت لازم شود. با comparator شفاف می‌توان از این تبدیل غیرضروری جلوگیری کرد.

### استفاده از std::less<> در set و map

در containerهای ordered، comparator باید از مقایسه ناهمگون پشتیبانی کند. استفاده از `std::less<>` به‌جای `std::less<std::string>` این امکان را فراهم می‌کند.

```cpp
#include <iostream>
#include <set>
#include <string>
#include <string_view>

int main() {
    const std::set<std::string, std::less<>> names{
        "Alice",
        "Bob",
        "Charlie"
    };

    const std::string_view query = "Bob";

    std::cout << std::boolalpha;
    std::cout << names.contains(query) << '\n';
}
```

در این مثال، جست‌وجو با `std::string_view` انجام می‌شود، در حالی که نوع کلیدهای مجموعه `std::string` است.

### استفاده از transparent hash و equality در unordered containerها

در containerهای unordered، پشتیبانی از جست‌وجوی ناهمگون به hash و predicate برابری شفاف نیاز دارد. هر دو باید برای نوع کلید و نوع جست‌وجو سازگار باشند و قراردادهای hash و برابری را رعایت کنند.

```cpp
#include <cstddef>
#include <functional>
#include <iostream>
#include <string>
#include <string_view>
#include <unordered_set>

struct TransparentHash {
    using is_transparent = void;

    std::size_t operator()(std::string_view value) const noexcept {
        return std::hash<std::string_view>{}(value);
    }
};

struct TransparentEqual {
    using is_transparent = void;

    bool operator()(std::string_view lhs,
                    std::string_view rhs) const noexcept {
        return lhs == rhs;
    }
};

int main() {
    const std::unordered_set<std::string, TransparentHash, TransparentEqual>
        names{"Alice", "Bob", "Charlie"};

    const std::string_view query = "Charlie";

    std::cout << std::boolalpha;
    std::cout << names.contains(query) << '\n';
}
```

در این نمونه، هر دو functor ورودی را به شکل `std::string_view` می‌پذیرند؛ بنابراین هم برای کلیدهای `std::string` و هم برای عبارت جست‌وجوی `std::string_view` قابل استفاده‌اند.

نکته مهم این است که تعریف `is_transparent` به‌تنهایی کافی نیست. عملیات مقایسه یا hash باید واقعاً برای انواع مورد استفاده معتبر باشد و برای کلیدهای معادل، قرارداد یکسانی داشته باشد.

### مزایای Heterogeneous Lookup

این روش می‌تواند در سناریوهای پرتکرار مزایای زیر را داشته باشد:

- جلوگیری از ساخت شیء موقت از نوع کلید
- کاهش تخصیص حافظه در مواردی که تبدیل معمول به تخصیص نیاز دارد
- کاهش هزینه تبدیل رشته‌ها
- امکان جست‌وجو با `std::string_view` یا انواع سازگار دیگر

با این حال، کاهش هزینه فقط زمانی رخ می‌دهد که نوع جست‌وجو و comparator یا hash واقعاً از مسیر بدون تبدیل پشتیبانی کنند. نباید صرف استفاده از `contains()` را تضمین‌کننده حذف تمام هزینه‌های موقت دانست.

## تفاوت containerهای ordered و unordered

نام و شیوه فراخوانی `contains()` در هر دو خانواده یکسان است، اما انتخاب container همچنان بر عملکرد کلی برنامه اثر می‌گذارد.

### رفتار در containerهای ordered

در `std::set` و `std::map`، عناصر بر اساس comparator مرتب نگهداری می‌شوند. جست‌وجو معمولاً پیچیدگی لگاریتمی دارد:

\[
O(\log n)
\]

مزیت این containerها علاوه بر جست‌وجوی قابل پیش‌بینی، حفظ ترتیب و پشتیبانی از عملیات‌هایی مانند `lower_bound()` و `upper_bound()` است.

### رفتار در containerهای unordered

در `std::unordered_set` و `std::unordered_map`، جست‌وجو از hash table استفاده می‌کند. پیچیدگی متوسط آن معمولاً ثابت است:

\[
O(1)
\]

اما در بدترین حالت، به دلیل collisionهای زیاد، پیچیدگی خطی خواهد بود:

\[
O(n)
\]

بنابراین، انتخاب unordered صرفاً به دلیل متوسط `O(1)` بودن جست‌وجو همیشه تصمیم بهتری نیست. نیاز به ترتیب، کیفیت hash، هزینه حافظه، الگوی دسترسی و رفتار بدترین حالت نیز باید بررسی شوند.

## کاربردهای عملی

### اعتبارسنجی ورودی

برای کنترل این‌که یک مقدار ورودی در مجموعه مقادیر مجاز قرار دارد، `contains()` خوانایی خوبی دارد.

```cpp
#include <iostream>
#include <set>

int main() {
    const std::set<int> supportedVersions{1, 2, 3};

    const int requestedVersion = 2;

    if (!supportedVersions.contains(requestedVersion)) {
        std::cerr << "Unsupported version\n";
        return 1;
    }

    std::cout << "Version accepted\n";
}
```

### جلوگیری از پردازش تکراری

در پردازش داده‌ها می‌توان شناسه‌های پردازش‌شده را در یک مجموعه نگهداری کرد.

```cpp
#include <iostream>
#include <string>
#include <unordered_set>
#include <vector>

int main() {
    const std::vector<std::string> incomingIds{
        "A-100", "B-200", "A-100", "C-300"
    };

    std::unordered_set<std::string> processed;

    for (const auto& id : incomingIds) {
        if (processed.contains(id)) {
            continue;
        }

        processed.insert(id);
        std::cout << "Processing " << id << '\n';
    }
}
```

در این مثال، `contains()` وجود شناسه را بررسی می‌کند و `insert()` آن را ثبت می‌کند. این دو عملیات جداگانه‌اند؛ در برنامه چندنخی، این الگو به‌تنهایی عملیات اتمیک «بررسی و درج» ایجاد نمی‌کند.

### بررسی وجود تنظیمات

در برنامه‌هایی که تنظیمات به شکل map نگهداری می‌شوند، می‌توان پیش از دسترسی به یک کلید، وجود آن را بررسی کرد. با این حال، اگر بلافاصله پس از بررسی قرار است مقدار خوانده شود، معمولاً بهتر است مستقیماً از `find()` استفاده شود تا جست‌وجو تکرار نشود.

## نکات مهم و خطاهای رایج

### تصور وجود std::contains به‌عنوان تابع آزاد

در ++C20، برای containerهای انجمنی از تابع عضو `container.contains(key)` استفاده می‌شود. نوشتن `std::contains(container, key)` در این استاندارد صحیح نیست.

### استفاده از contains روی vector

تابع عضو `contains()` برای `std::vector` در ++C20 وجود ندارد. برای جست‌وجوی عنصر در یک vector می‌توان از الگوریتم‌های مناسب استفاده کرد.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    const std::vector<int> values{10, 20, 30};

    const bool exists =
        std::ranges::find(values, 20) != values.end();

    std::cout << std::boolalpha << exists << '\n';
}
```

الگوریتم `std::ranges::find()` از ++C20 در دسترس است. نسخه کلاسیک `std::find()` نیز از استانداردهای قدیمی‌تر وجود داشته است.

### اشتباه گرفتن وجود کلید با وجود مقدار

در `std::map` و `std::unordered_map`، آرگومان `contains()` با کلید مقایسه می‌شود، نه با مقدار ذخیره‌شده در map. برای جست‌وجوی مقدار باید روی عناصر پیمایش انجام دهید یا ساختار داده متناسب‌تری انتخاب کنید.

### فرض کردن اینکه contains عنصر را برمی‌گرداند

خروجی `contains()` فقط `bool` است. اگر به iterator، reference یا مقدار عنصر نیاز دارید، از `find()` استفاده کنید.

### انجام بررسی و سپس درج در محیط چندنخی

فراخوانی جداگانه `contains()` و `insert()` یک عملیات اتمیک نیست. اگر چند thread به‌صورت هم‌زمان به یک ساختار داده مشترک دسترسی دارند، باید همگام‌سازی مناسب انجام شود. containerهای استاندارد معمولی نیز برای دسترسی هم‌زمان همراه با تغییرات بدون synchronization طراحی نشده‌اند.

### نادیده گرفتن هزینه hash و مقایسه

پیچیدگی `O(1)` متوسط در containerهای unordered به معنی هزینه ثابت و ناچیز در تمام شرایط نیست. تابع hash، مقایسه کلیدها، collisionها و rehash می‌توانند بر عملکرد واقعی اثر بگذارند.

## تفاوت با قابلیت‌های مشابه در استانداردهای بعدی

نام `contains` در چند بخش کتابخانه استاندارد استفاده شده است؛ اما همه آن‌ها متعلق به ++C20 نیستند.

### تابع contains در std::string

تابع عضو `std::string::contains()` در ++C23 اضافه شده است، نه ++C20. این تابع بررسی می‌کند آیا رشته شامل زیررشته یا نویسه موردنظر است یا خیر.

```cpp
#include <iostream>
#include <string>

int main() {
    const std::string text = "modern cpp";

    std::cout << std::boolalpha;
    std::cout << text.contains("cpp") << '\n';
}
```

برای کدی که باید با ++C20 کامپایل شود، می‌توان از `find()` استفاده کرد:

```cpp
#include <iostream>
#include <string>

int main() {
    const std::string text = "modern cpp";

    const bool found = text.find("cpp") != std::string::npos;

    std::cout << std::boolalpha << found << '\n';
}
```

### الگوریتم std::ranges::contains

الگوریتم `std::ranges::contains()` نیز در ++C23 استاندارد شده است. این الگوریتم برای بررسی وجود مقدار در یک range به کار می‌رود و برخلاف تابع عضو containerهای انجمنی، محدود به `set` و `map` نیست.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    const std::vector<int> values{10, 20, 30};

    std::cout << std::boolalpha;
    std::cout << std::ranges::contains(values, 20) << '\n';
}
```

بنابراین، هنگام استفاده از `std::ranges::contains()` یا `std::string::contains()` باید حالت استاندارد ++C23 یا جدیدتر و پشتیبانی کتابخانه استاندارد مورد استفاده را در نظر گرفت.

## سازگاری کامپایلر و تنظیم استاندارد

برای استفاده از تابع عضو `contains()` در containerهای انجمنی، باید کامپایلر و کتابخانه استاندارد مورد استفاده از ++C20 پشتیبانی کنند. فعال کردن حالت ++C20 در کامپایلر ضروری است؛ با این حال، پشتیبانی کامل به نسخه کتابخانه استاندارد نیز وابسته است.

برای نمونه، در GCC معمولاً می‌توان از گزینه زیر استفاده کرد:

```bash
g++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o app
```

در Clang نیز گزینه استاندارد مشابه است:

```bash
clang++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o app
```

در MSVC می‌توان حالت استاندارد ++C20 را با گزینه مناسب فعال کرد:

```bat
cl /std:c++20 /EHsc main.cpp
```

این گزینه‌ها حالت زبان را مشخص می‌کنند، اما به‌تنهایی تضمین نمی‌کنند که تمام قابلیت‌های کتابخانه استاندارد در هر نسخه نصب‌شده پیاده‌سازی شده باشند. برای پروژه‌های واقعی، نسخه کامپایلر و کتابخانه استاندارد را نیز بررسی کنید.

## ملاحظات طراحی و انتخاب روش مناسب

انتخاب میان `contains()`، `find()` و الگوریتم‌های جست‌وجو باید بر اساس کاری باشد که پس از جست‌وجو انجام می‌دهید.

| نیاز برنامه | روش پیشنهادی |
|---|---|
| فقط بررسی وجود کلید در `set` یا `map` | `contains()` |
| بررسی وجود کلید و سپس دسترسی به عنصر | `find()` |
| جست‌وجوی مقدار در `vector` یا range عمومی در ++C20 | `std::find()` یا `std::ranges::find()` |
| بررسی زیررشته در `std::string` با ++C20 | `find()` |
| بررسی زیررشته با `std::string` در ++C23 | `std::string::contains()` |
| بررسی مقدار در range با ++C23 | `std::ranges::contains()` |

در طراحی APIها نیز بهتر است تابعی که فقط به پاسخ منطقی نیاز دارد، از `contains()` استفاده کند؛ اما اگر قرار است عنصر پیدا‌شده مصرف شود، بهتر است iterator حاصل از `find()` حفظ شود. این انتخاب از جست‌وجوی تکراری جلوگیری می‌کند و قصد کد را روشن نگه می‌دارد.

## جمع‌بندی

تابع عضو `contains()` در ++C20 راهی مستقیم و خوانا برای بررسی وجود کلید در containerهای انجمنی فراهم می‌کند. این قابلیت برای `std::set`، `std::map` و نسخه‌های `multiset`، `multimap` و unordered آن‌ها در دسترس است و نتیجه‌ای از نوع `bool` برمی‌گرداند.

مهم‌ترین نکات کاربردی عبارت‌اند از:

- برای بررسی صرف وجود کلید در containerهای انجمنی، `contains()` معمولاً خواناتر از الگوی `find() != end()` است.
- اگر به عنصر پیدا‌شده نیاز دارید، `find()` را انتخاب کنید تا جست‌وجو دوباره انجام نشود.
- پیچیدگی جست‌وجو در containerهای ordered برابر `O(log n)` و در containerهای unordered به‌طور متوسط `O(1)` و در بدترین حالت `O(n)` است.
- با استفاده از comparator، hash و equality شفاف می‌توان از Heterogeneous Lookup بهره برد.
- `std::vector` در ++C20 تابع عضو `contains()` ندارد.
- `std::string::contains()` و `std::ranges::contains()` قابلیت‌های ++C23 هستند و نباید با تابع عضو containerهای انجمنی در ++C20 اشتباه گرفته شوند.

در نتیجه، `contains()` ابزار مناسبی برای بیان روشن قصد برنامه در بررسی عضویت است؛ به شرط آن‌که نوع container، نیاز به دسترسی به عنصر و هزینه واقعی جست‌وجو در طراحی لحاظ شوند.


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>