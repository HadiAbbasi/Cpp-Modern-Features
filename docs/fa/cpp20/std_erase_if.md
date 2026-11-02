<div align="center">

[🇺🇸 English](../../en/cpp20/std_erase_if.md) | [🇮🇷 فارسی](./std_erase_if.md)

</div>

---

# راهنمای جامع `std::erase_if` در ++C20

## مقدمه

قابلیت `std::erase_if` یکی از امکانات کتابخانه استاندارد در ++C20 است که حذف عناصر یک container را بر اساس یک شرط، ساده‌تر و خواناتر می‌کند. پیش از معرفی این قابلیت، حذف شرطی عناصر از بسیاری از containerها به ترکیب الگوریتم‌هایی مانند `std::remove_if` و عملیات `erase` نیاز داشت. این الگو برای بعضی containerها مناسب بود، اما برای همه آن‌ها قابل استفاده نبود و تفاوت‌های رفتاری میان containerها باعث می‌شد کد حذف عناصر پیچیده‌تر از حد لازم شود.

تابع `std::erase_if` یک رابط غیرعضوی و یکپارچه برای حذف شرطی عناصر فراهم می‌کند. این تابع جزئیات متناسب با نوع container را پنهان می‌کند؛ برای مثال، برای `std::vector` از الگوی Remove-Erase استفاده می‌کند، در حالی که برای containerهای associative از سازوکار حذف اختصاصی همان container بهره می‌برد.

این قابلیت بخشی از **Standard Library** در استاندارد ++C20 است، نه یک قابلیت زبانی و نه یک compiler extension. برای استفاده از آن باید استاندارد کتابخانه و کامپایلر شما از ++C20 پشتیبانی کنند.

## مسئله‌ای که این قابلیت حل می‌کند

فرض کنید مجموعه‌ای از اعداد دارید و می‌خواهید تمام مقادیر منفی را حذف کنید. در ++C17 و نسخه‌های قدیمی‌تر، برای `std::vector` معمولاً چنین کدی نوشته می‌شد:

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, -2, 3, -4, 5};

    auto newEnd = std::remove_if(
        values.begin(),
        values.end(),
        [](int value) {
            return value < 0;
        }
    );

    values.erase(newEnd, values.end());
}
```

الگوریتم `std::remove_if` به‌تنهایی اندازه container را تغییر نمی‌دهد. این الگوریتم عناصر باقی‌ماندنی را به ابتدای محدوده منتقل می‌کند و یک iterator به انتهای منطقی محدوده جدید برمی‌گرداند؛ عناصر انتهایی همچنان داخل container هستند و باید با `erase` حذف شوند.

در ++C20 همین عملیات برای `std::vector` به شکل زیر نوشته می‌شود:

```cpp
#include <vector>

int main() {
    std::vector<int> values{1, -2, 3, -4, 5};

    std::erase_if(values, [](int value) {
        return value < 0;
    });
}
```

در این نسخه، هم شرط حذف و هم تغییر اندازه container در یک فراخوانی انجام می‌شود. در نتیجه احتمال فراموش کردن مرحله `erase` کمتر می‌شود و کد هدف خود را مستقیم‌تر بیان می‌کند.

## معرفی رابط استاندارد

تابع `std::erase_if` یک function template غیرعضوی است که برای containerهای پشتیبانی‌شده overloadهای جداگانه دارد. شکل دقیق declaration به نوع container وابسته است؛ بنابراین نباید آن را یک template واحد با یک امضای کاملاً یکسان برای همه containerها تصور کرد.

الگوی مفهومی فراخوانی چنین است:

```cpp
std::erase_if(container, predicate);
```

- آرگومان اول، container موردنظر است.
- آرگومان دوم، یک predicate است که برای هر عنصر بررسی می‌شود.
- اگر predicate مقدار `true` برگرداند، عنصر مربوطه حذف می‌شود.
- مقدار بازگشتی، تعداد عناصر حذف‌شده از نوع `container.size_type` است.

برای مثال:

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6};

    const auto removed = std::erase_if(values, [](int value) {
        return value % 2 == 0;
    });

    std::cout << "Removed: " << removed << '\n';
    std::cout << "Remaining:";

    for (int value : values) {
        std::cout << ' ' << value;
    }

    std::cout << '\n';
}
```

خروجی:

```text
Removed: 3
Remaining: 1 3 5
```

## رفتار predicate

### معنای مقدار بازگشتی

predicate باید برای یک عنصر تصمیم بگیرد که آیا آن عنصر باید حذف شود یا خیر. مقدار `true` به معنی حذف و مقدار `false` به معنی نگه‌داشتن عنصر است.

```cpp
#include <string>
#include <vector>

int main() {
    std::vector<std::string> names{
        "Alice",
        "",
        "Bob",
        "",
        "Charlie"
    };

    std::erase_if(names, [](const std::string& name) {
        return name.empty();
    });
}
```

در این مثال، رشته‌های خالی حذف می‌شوند.

### نوع آرگومان predicate

بهتر است predicate را با پارامتر `const` reference بنویسید، مگر آنکه نوع یا منطق خاصی نیاز دیگری ایجاد کند. این روش از کپی غیرضروری عناصر، به‌خصوص برای objectهای بزرگ، جلوگیری می‌کند.

```cpp
std::erase_if(records, [](const Record& record) {
    return record.isExpired();
});
```

برای نوع‌های کوچک و ساده مانند `int`، دریافت مقدار نیز معمولاً مناسب است:

```cpp
std::erase_if(values, [](int value) {
    return value < 0;
});
```

نوع دقیق آرگومان باید با `value_type` یا نوعی که overload مربوطه به predicate ارائه می‌کند سازگار باشد. برای mapها و multimapها، عنصر از نوع `std::pair<const Key, T>` است؛ بنابراین معمولاً predicate با `const auto&` یا `const std::pair<const Key, T>&` نوشته می‌شود.

### خالص و پایدار بودن predicate

predicate باید تصمیمی سازگار و قابل اتکا برای عناصر ارائه کند. تغییر دادن container درون predicate، از جمله افزودن یا حذف عنصر از همان container، می‌تواند iteratorها و وضعیت پیمایش داخلی را نامعتبر کند و رفتار برنامه را نامعتبر یا غیرقابل پیش‌بینی سازد.

همچنین بهتر است predicate بر اساس وضعیت خارجیِ در حال تغییر تصمیم نگیرد. از side effectهای غیرضروری مانند تغییر شمارنده‌های مشترک، تغییر عناصر دیگر یا وابستگی به ترتیب ارزیابی اجتناب کنید. جزئیات تعداد و ترتیب فراخوانی‌ها را نباید بیش از آنچه قرارداد الگوریتم و overload مربوطه تضمین می‌کند فرض کرد.

## پشتیبانی containerها و تفاوت پیاده‌سازی

### containerهای ترتیبی مبتنی بر آرایه

برای `std::vector` و `std::deque`، حذف شرطی با جابه‌جایی عناصر باقی‌مانده و سپس کوچک کردن container انجام می‌شود. این رفتار از نظر مفهومی همان Remove-Erase Idiom است.

```cpp
#include <deque>
#include <vector>

int main() {
    std::vector<int> numbers{1, 2, 3, 4, 5};
    std::deque<int> queue{1, 2, 3, 4, 5};

    const auto removedFromVector = std::erase_if(
        numbers,
        [](int value) { return value > 3; }
    );

    const auto removedFromDeque = std::erase_if(
        queue,
        [](int value) { return value > 3; }
    );
}
```

در این containerها، ترتیب نسبی عناصر باقی‌مانده حفظ می‌شود. هزینه عملیات می‌تواند شامل بررسی تمام عناصر و جابه‌جایی عناصر باقی‌مانده باشد؛ بنابراین معمولاً پیچیدگی زمانی خطی نسبت به اندازه container دارد.

### containerهای لیستی

برای `std::list` و `std::forward_list`، عملیات حذف با سازوکارهای عضوی خود container انجام می‌شود. این containerها قابلیت حذف node را بدون جابه‌جایی مقدار عناصر دیگر دارند.

```cpp
#include <forward_list>
#include <list>

int main() {
    std::list<int> values{1, 2, 3, 4, 5};
    std::forward_list<int> linkedValues{1, 2, 3, 4, 5};

    std::erase_if(values, [](int value) {
        return value % 2 == 0;
    });

    std::erase_if(linkedValues, [](int value) {
        return value % 2 == 0;
    });
}
```

این تفاوت مهم است: `std::erase_if` رابط یکسانی ارائه می‌دهد، اما الزاماً یک الگوریتم داخلی یکسان برای همه containerها اجرا نمی‌کند.

### containerهای associative مرتب

برای `std::set`، `std::multiset`، `std::map` و `std::multimap`، عناصر با قواعد ordering و ساختار درختی نگهداری می‌شوند. الگوریتم `std::remove_if` برای این containerها مناسب نیست؛ زیرا کلیدها و ساختار container را نمی‌توان با جابه‌جایی معمول عناصر تغییر داد.

تابع `std::erase_if` عناصر منطبق را با عملیات حذف خود container پاک می‌کند:

```cpp
#include <map>
#include <set>
#include <string>

int main() {
    std::set<int> ids{10, 11, 12, 13, 14};

    std::erase_if(ids, [](int id) {
        return id < 12;
    });

    std::map<int, std::string> users{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"}
    };

    std::erase_if(users, [](const auto& entry) {
        return entry.second == "Bob";
    });
}
```

در مثال map، predicate یک جفت کلید و مقدار را دریافت می‌کند. عضو `first` کلید و عضو `second` مقدار است. کلید map از نوع `const` است و از طریق این عملیات نمی‌توان آن را تغییر داد.

### containerهای associative نامرتب

برای `std::unordered_set`، `std::unordered_multiset`، `std::unordered_map` و `std::unordered_multimap` نیز overloadهای اختصاصی وجود دارد.

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 90},
        {"Bob", 45},
        {"Charlie", 72}
    };

    const auto removed = std::erase_if(
        scores,
        [](const auto& entry) {
            return entry.second < 60;
        }
    );
}
```

ترتیب پیمایش containerهای unordered تضمین‌شده نیست. بنابراین نباید خروجی پیمایش یا ترتیب بررسی عناصر را به یک ترتیب ثابت وابسته کنید.

### رشته‌های استاندارد

از ++C20، برای `std::basic_string` نیز تابع غیرعضوی `std::erase_if` در دسترس است. این قابلیت برای حذف characterهایی که predicate آن‌ها را مشخص می‌کند مفید است.

```cpp
#include <cctype>
#include <string>

int main() {
    std::string text = "C++20 is powerful!";

    std::erase_if(text, [](unsigned char ch) {
        return std::isspace(ch);
    });
}
```

در استفاده از توابع خانواده `std::is...` از `<cctype>`، تبدیل مقدار به `unsigned char` اهمیت دارد؛ ارسال `char` منفی به این توابع، به‌جز مقدار `EOF`، می‌تواند رفتار تعریف‌نشده ایجاد کند.

## مثال‌های کاربردی

### حذف داده‌های نامعتبر

```cpp
#include <vector>

struct Measurement {
    double value;
    bool valid;
};

int main() {
    std::vector<Measurement> measurements{
        {12.5, true},
        {-1.0, false},
        {18.2, true},
        {0.0, false}
    };

    std::erase_if(measurements, [](const Measurement& item) {
        return !item.valid;
    });
}
```

در این الگو، predicate منطق اعتبارسنجی را به شکل خوانا از منطق نگهداری container جدا می‌کند.

### حذف رکوردهای منقضی‌شده

```cpp
#include <chrono>
#include <vector>

struct Session {
    std::chrono::system_clock::time_point expiresAt;
};

int main() {
    using Clock = std::chrono::system_clock;

    std::vector<Session> sessions;
    const auto now = Clock::now();

    std::erase_if(sessions, [now](const Session& session) {
        return session.expiresAt <= now;
    });
}
```

در این مثال، زمان جاری یک بار محاسبه و در lambda ذخیره شده است. در نتیجه تمام عناصر با یک مرجع زمانی یکسان ارزیابی می‌شوند.

### حذف ورودی‌های تکراری بر اساس یک معیار

```cpp
#include <string>
#include <vector>

struct User {
    int id;
    std::string name;
    bool active;
};

int main() {
    std::vector<User> users{
        {1, "Alice", true},
        {2, "Bob", false},
        {3, "Charlie", true}
    };

    std::erase_if(users, [](const User& user) {
        return !user.active;
    });
}
```

این مثال حذف کاربرهای غیرفعال را نشان می‌دهد. اگر هدف حذف duplicateها باشد، باید معیار نگه‌داشتن یک نماینده از هر گروه نیز مشخص شود؛ `std::erase_if` به‌تنهایی عملیات deduplication انجام نمی‌دهد.

### حذف بر اساس کلید در map

```cpp
#include <map>
#include <string>

int main() {
    std::map<std::string, int> inventory{
        {"apple", 10},
        {"banana", 0},
        {"orange", 7}
    };

    std::erase_if(inventory, [](const auto& item) {
        return item.second == 0;
    });
}
```

اگر هدف حذف یک کلید مشخص باشد و نه حذف شرطی چند عنصر، عملیات عضوی `erase(key)` معمولاً ساده‌تر و مناسب‌تر است.

## مقایسه با روش‌های قدیمی‌تر

### مقایسه با Remove-Erase Idiom

در ++C11، ++C14 و ++C17، روش رایج برای `vector` و `deque` ترکیب `std::remove_if` و `erase` بود. در ++C20، `std::erase_if` این دو مرحله را در یک رابط جمع می‌کند.

| ویژگی | `std::remove_if` همراه `erase` | `std::erase_if` |
|---|---|---|
| نسخه استاندارد | الگوریتم قدیمی‌تر | ++C20 |
| تغییر اندازه container | `remove_if` به‌تنهایی خیر | بله |
| خوانایی | دو مرحله | یک فراخوانی |
| پشتیبانی از map و set | الگوی عمومی قابل استفاده نیست | overload اختصاصی دارد |
| مقدار بازگشتی | iterator انتهای منطقی | تعداد حذف‌شده‌ها |

الگوریتم `std::remove_if` همچنان مفید است؛ برای مثال زمانی که با یک محدوده یا container سفارشی کار می‌کنید که overload متناظر `std::erase_if` ندارد، یا زمانی که به iterator انتهای منطقی نیاز دارید.

### مقایسه با `container.erase`

تابع عضوی `erase` معمولاً برای حذف بر اساس iterator، محدوده یا کلید استفاده می‌شود. در مقابل، `std::erase_if` حذف بر اساس predicate را بیان می‌کند.

```cpp
#include <map>

int main() {
    std::map<int, int> values{
        {1, 10},
        {2, 20},
        {3, 30}
    };

    values.erase(2);

    std::erase_if(values, [](const auto& item) {
        return item.second > 20;
    });
}
```

برای حذف یک کلید مشخص در map، `erase(key)` مستقیم‌تر است. برای حذف تمام عناصر مطابق یک شرط، `std::erase_if` خواناتر است.

## پیچیدگی زمانی و هزینه اجرا

پیچیدگی دقیق به نوع container و overload استاندارد مربوطه وابسته است:

- برای containerهای ترتیبی مانند `vector` و `deque`، بررسی عناصر و جابه‌جایی عناصر باقی‌مانده هزینه‌ای خطی نسبت به تعداد عناصر دارد.
- برای `list` و `forward_list`، پیمایش خطی است و حذف nodeهای منطبق از عملیات عضوی container استفاده می‌کند.
- برای containerهای associative مرتب، پیمایش عناصر لازم است و هزینه حذف هر عنصر تابع هزینه عملیات حذف همان container است.
- برای containerهای unordered، پیمایش خطی لازم است و هزینه حذف به رفتار hash table و هزینه حذف عناصر وابسته است.

در نتیجه، نباید صرفاً از نام یکسان تابع نتیجه گرفت که هزینه آن در تمام containerها یکسان است. اگر performance حساس است، نوع container، تعداد عناصر حذف‌شده، هزینه predicate و الگوی دسترسی به حافظه را در benchmark واقعی بررسی کنید.

## اعتبار iteratorها و referenceها

رفتار invalidation به container بستگی دارد و `std::erase_if` قاعده مستقلی برای یکسان‌سازی آن ایجاد نمی‌کند؛ در عمل باید قواعد invalidation عملیات حذف همان container را در نظر گرفت.

- در `std::vector`، حذف عناصر می‌تواند iteratorها و referenceهای عناصر از محل اولین عنصر حذف‌شده به بعد را نامعتبر کند؛ iteratorهای مربوط به انتهای قبلی نیز معتبر نمی‌مانند.
- در `std::deque`، قواعد invalidation به محل و شکل عملیات حذف وابسته است و باید به قرارداد همان container رجوع کرد.
- در `std::list` و `std::forward_list`، iteratorها و referenceهای عناصر حذف‌شده نامعتبر می‌شوند، اما عناصر باقی‌مانده اعتبار خود را حفظ می‌کنند.
- در containerهای associative مرتب، iteratorها و referenceهای عناصر حذف‌شده نامعتبر می‌شوند و عناصر دیگر معتبر می‌مانند.
- در containerهای unordered، iteratorها و referenceهای عناصر حذف‌شده نامعتبر می‌شوند؛ قواعد iteratorها را باید مطابق قرارداد container و عملیات مربوطه بررسی کرد.

اگر iterator یا referenceای به عنصری دارید که ممکن است حذف شود، پس از فراخوانی `std::erase_if` نباید بدون بررسی اعتبار آن از آن استفاده کنید.

## محدودیت‌ها و نکات طراحی

### پشتیبانی نشدن containerهای دلخواه

وجود `std::erase_if` به معنی پشتیبانی خودکار از هر نوع container یا range سفارشی نیست. این تابع برای انواع مشخصی از containerهای کتابخانه استاندارد overload دارد. برای container سفارشی باید API مناسب خود آن نوع را به کار ببرید یا در صورت امکان الگوی الگوریتمی متناسبی پیاده‌سازی کنید.

### تغییر اندازه و مالکیت عناصر

حذف عنصر فقط به معنی ناپدید شدن آن از container نیست؛ destructor عنصر حذف‌شده نیز در چارچوب قواعد عمر object اجرا می‌شود. اگر عنصر منبعی مانند حافظه، file handle یا lock را مدیریت می‌کند، اثر حذف باید در طراحی نوع در نظر گرفته شده باشد.

### predicate پرهزینه

اگر predicate عملیات سنگینی مانند جست‌وجوی شبکه‌ای، دسترسی به پایگاه داده یا محاسبات پیچیده انجام دهد، هزینه اصلی ممکن است از خود predicate ناشی شود. در چنین شرایطی بهتر است داده‌های لازم پیش از حذف آماده شوند یا معیار حذف به شکلی کارآمدتر محاسبه شود.

### حذف همه عناصر

برای حذف همه عناصر می‌توان predicateای نوشت که همیشه `true` برگرداند، اما در اغلب موارد `clear()` ساده‌تر و صریح‌تر است.

```cpp
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3};

    values.clear();
}
```

### حذف هیچ عنصر

اگر predicate برای تمام عناصر `false` برگرداند، container از نظر محتوایی تغییری نمی‌کند و مقدار بازگشتی صفر خواهد بود. با این حال، در کد حساس به performance بهتر است predicate و هزینه پیمایش غیرضروری را در نظر بگیرید.

### تغییر container در زمان پیمایش

درون predicate، container هدف را به‌صورت ساختاری تغییر ندهید. برای مثال، فراخوانی `push_back` یا `erase` روی همان container از داخل predicate می‌تواند روند پیمایش را مختل کند. اگر لازم است عناصر دیگری اضافه شوند، این کار را در مرحله‌ای جداگانه انجام دهید.

## خطاهای رایج

- **اشتباه گرفتن جهت predicate:** مقدار `true` به معنی حذف است، نه نگه‌داشتن عنصر.
- **استفاده از `std::remove_if` برای map یا set:** این الگوریتم برای containerهایی که عنصرهایشان از طریق iterator قابل تغییر و جابه‌جایی نیستند، الگوی مناسبی نیست.
- **فرض تغییر نکردن اندازه با `std::remove_if`:** این الگوریتم به‌تنهایی container را کوچک نمی‌کند؛ اما `std::erase_if` عملیات حذف واقعی را انجام می‌دهد.
- **استفاده از iterator قدیمی پس از حذف:** اعتبار iteratorها به نوع container و محل حذف وابسته است.
- **تغییر container در predicate:** این کار می‌تواند اعتبار پیمایش داخلی را از بین ببرد.
- **فرض ترتیب ثابت در unordered containerها:** ترتیب عناصر این containerها تضمین‌شده نیست.
- **نادیده گرفتن نوع عنصر map:** عنصر map یک `pair` با کلید `const` است؛ predicate باید با این نوع سازگار باشد.
- **نادیده گرفتن مقدار بازگشتی:** این مقدار تعداد عناصر حذف‌شده را گزارش می‌کند و می‌تواند برای logging، آمار یا کنترل جریان مفید باشد.

## نکات مربوط به نسخه استاندارد و سازگاری

تابع `std::erase_if` از ++C20 در کتابخانه استاندارد معرفی شده است. فعال کردن صرف قابلیت‌های زبانی ++C20 کافی نیست؛ کتابخانه استاندارد مورد استفاده نیز باید declaration و پیاده‌سازی مربوطه را ارائه کند.

برای نمونه، در GCC یا Clang می‌توانید استاندارد را با گزینه مناسب فعال کنید:

```bash
g++ -std=c++20 main.cpp -o app
clang++ -std=c++20 main.cpp -o app
```

پشتیبانی واقعی به نسخه compiler و نسخه standard library وابسته است. در محیط‌های مبتنی بر Clang ممکن است نسخه کتابخانه استاندارد انتخاب‌شده تعیین‌کننده باشد؛ بنابراین در صورت مشاهده خطای نبودن `std::erase_if`، نسخه compiler، کتابخانه استاندارد و تنظیمات build را بررسی کنید.

برای استفاده از هر container، header مربوط به آن را include کنید؛ برای مثال `<vector>`، `<map>`، `<unordered_map>`، `<list>` یا `<string>`. الگوریتم‌های قدیمی مانند `std::remove_if` در `<algorithm>` قرار دارند، اما برای استفاده از overloadهای `std::erase_if` باید header مربوط به نوع container در دسترس باشد.

## راهنمای انتخاب روش مناسب

- **برای حذف شرطی از containerهای استاندارد پشتیبانی‌شده:** از `std::erase_if` استفاده کنید.
- **برای حذف یک کلید مشخص از map یا set:** معمولاً `container.erase(key)` انتخاب مستقیم‌تری است.
- **برای حذف یک iterator یا محدوده مشخص:** از overload مناسب `erase` استفاده کنید.
- **برای range یا container سفارشی بدون overload استاندارد:** از API خود نوع یا الگوریتم مناسب مانند `std::remove_if` همراه با عملیات حذف استفاده کنید.
- **برای خالی کردن کامل container:** از `clear()` استفاده کنید.

## جمع‌بندی

قابلیت `std::erase_if` در ++C20 یک رابط استاندارد، خوانا و مناسب برای حذف شرطی عناصر از containerهای رایج کتابخانه استاندارد است. مزیت اصلی آن پنهان کردن تفاوت‌های پیاده‌سازی حذف میان containerها و حذف نیاز به نوشتن دستی الگوهای متفاوت است.

برای استفاده حرفه‌ای از این قابلیت، باید معنای predicate، نوع عنصر در containerهای associative، هزینه حذف، invalidation iteratorها و محدودیت overloadهای موجود را بشناسید. در صورت نیاز به حذف شرطی از containerهای استاندارد پشتیبانی‌شده، این تابع معمولاً انتخاب پیش‌فرض مناسبی است؛ اما برای حذف بر اساس کلید، حذف یک محدوده مشخص یا خالی کردن کامل container، عملیات عضوی مربوطه ممکن است روشن‌تر و کارآمدتر باشد.


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>