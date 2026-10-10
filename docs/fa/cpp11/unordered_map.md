<div align="center">

[🇺🇸 English](../../en/cpp11/unordered_map.md) | [🇮🇷 فارسی](./unordered_map.md)

</div>

---

# راهنمای جامع `std::unordered_map` در ++C11

## مقدمه

ساختار دادهٔ `std::unordered_map` یکی از Containerهای مهم در Standard Library زبان ++C است که برای ذخیره‌سازی و بازیابی داده‌ها بر اساس کلید (Key) طراحی شده است. این Container از استاندارد ++C11 معرفی شد و امکان دسترسی سریع به مقادیر مرتبط با کلیدها را در اختیار برنامه‌نویس قرار می‌دهد.

برای مثال، اگر بخواهیم نام و مشخصات کاربران را بر اساس شناسهٔ آن‌ها ذخیره کنیم، می‌توانیم از `std::unordered_map` استفاده کنیم. در این ساختار، هر کلید به حداکثر یک مقدار نگاشت می‌شود و کلیدها باید یکتا باشند.

مهم‌ترین ویژگی‌های این Container عبارت‌اند از:

- دسترسی متوسط با پیچیدگی زمانی \(O(1)\) برای جست‌وجو، درج و حذف.
- استفاده از Hash Table برای سازمان‌دهی داده‌ها.
- عدم تضمین ترتیب عناصر.
- پشتیبانی از کلیدهای سفارشی با تعریف Hash Function و Equality Predicate مناسب.
- امکان تنظیم ظرفیت اولیه و Load Factor برای کنترل رفتار Hash Table.
- پشتیبانی از انواع مختلف کلید و مقدار، از جمله انواع سفارشی.

با وجود این مزایا، استفاده از `std::unordered_map` همیشه بهترین انتخاب نیست. انتخاب صحیح آن به نیازهای برنامه، الگوی دسترسی به داده‌ها، هزینهٔ محاسبهٔ Hash و محدودیت‌های حافظه بستگی دارد.

## معرفی ساختار داده

### مفهوم نگاشت کلید به مقدار

نوع `std::unordered_map` در Header استاندارد `<unordered_map>` تعریف شده است و در Namespace استاندارد `std` قرار دارد.

ساختار کلی تعریف آن به شکل زیر است:

```cpp
std::unordered_map<Key, T, Hash, KeyEqual, Allocator>
```

پارامترهای قالب (Template Parameters) این Container عبارت‌اند از:

| پارامتر | توضیح |
|---|---|
| `Key` | نوع کلیدها |
| `T` | نوع مقادیر ذخیره‌شده |
| `Hash` | نوع تابع Hash برای کلیدها |
| `KeyEqual` | نوع تابع مقایسهٔ برابری کلیدها |
| `Allocator` | تخصیص‌دهندهٔ حافظه برای عناصر |

سه پارامتر آخر اختیاری هستند و در حالت معمول نیازی به مشخص کردن آن‌ها نیست.

برای نمونه، کد زیر یک نگاشت بین شناسهٔ عددی و نام کاربران ایجاد می‌کند:

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<int, std::string> users;

    users[101] = "Alice";
    users[102] = "Bob";
    users[103] = "Charlie";

    std::cout << users[102] << '\n';
}
```

خروجی برنامه به شکل زیر است:

```text
Bob
```

در این مثال، کلیدها از نوع `int` و مقادیر از نوع `std::string` هستند.

### تفاوت با `std::map`

هر دو Container برای نگاشت کلید به مقدار استفاده می‌شوند، اما نحوهٔ سازمان‌دهی داده‌ها در آن‌ها متفاوت است.

نوع `std::map` معمولاً با یک درخت متوازن پیاده‌سازی می‌شود و عناصر را بر اساس ترتیب کلید مرتب نگه می‌دارد؛ در مقابل، `std::unordered_map` از Hash Table استفاده می‌کند و ترتیب پیمایش عناصر را تضمین نمی‌کند.

| ویژگی | `std::unordered_map` | `std::map` |
|---|---|---|
| ساختار متداول | Hash Table | درخت متوازن |
| پیچیدگی متوسط جست‌وجو | \(O(1)\) | \(O(\log n)\) |
| پیچیدگی بدترین حالت جست‌وجو | \(O(n)\) | \(O(\log n)\) |
| ترتیب کلیدها | تضمین نمی‌شود | مرتب‌شده |
| دسترسی بر اساس بازهٔ کلیدها | پشتیبانی مستقیم ندارد | پشتیبانی می‌شود |
| نیاز به Hash Function | بله | خیر |
| نیاز به مقایسهٔ مرتب‌سازی | خیر | بله |
| کاربرد شاخص | جست‌وجوی سریع بر اساس کلید | جست‌وجوی مرتب و Range Query |

پیاده‌سازی داخلی دقیق این Containerها وابسته به کتابخانهٔ استاندارد است؛ جدول بالا رفتار و پیچیدگی استاندارد آن‌ها را در حالت معمول خلاصه می‌کند.

**قاعدهٔ انتخاب:** اگر ترتیب کلیدها اهمیت ندارد و جست‌وجوی سریع بر اساس کلید اولویت دارد، `std::unordered_map` گزینهٔ مناسبی است. اگر به پیمایش مرتب، کمینه و بیشینهٔ کلید یا جست‌وجوی بازه‌ای نیاز دارید، `std::map` معمولاً انتخاب مناسب‌تری خواهد بود.

## سازوکار داخلی Hash Table

### مفهوم Hash Function

تابع Hash، کلید را به یک مقدار عددی از نوع `std::size_t` تبدیل می‌کند. Container از این مقدار برای تعیین محل احتمالی ذخیره‌سازی عنصر در Hash Table استفاده می‌کند.

برای مثال، یک تابع Hash برای اعداد صحیح می‌تواند به شکل زیر باشد:

```cpp
#include <cstddef>

struct IntegerHash {
    std::size_t operator()(int value) const noexcept {
        return static_cast<std::size_t>(value);
    }
};
```

در این مثال، مقدار ورودی مستقیماً به `std::size_t` تبدیل می‌شود. این تابع صرفاً برای نمایش مفهوم Hash مناسب است و لزوماً برای هر نوع کلید یا هر الگوی داده‌ای انتخاب مناسبی نیست.

تابع Hash استاندارد برای انواع متداول، از جمله انواع صحیح و `std::string`، معمولاً بدون نیاز به تعریف دستی در دسترس است.

### مفهوم Bucket

در Hash Table، عناصر بر اساس Hash Value در بخش‌هایی به نام Bucket سازمان‌دهی می‌شوند. تابع Hash به تعیین Bucket مورد استفاده کمک می‌کند، اما مقدار Hash الزاماً شمارهٔ مستقیم Bucket نیست؛ کتابخانهٔ استاندارد می‌تواند از روش داخلی دیگری برای تعیین Bucket استفاده کند.

دو کلید متفاوت ممکن است به یک Bucket نگاشت شوند. این وضعیت Collision نام دارد.

برای مدیریت Collision، پیاده‌سازی Hash Table از سازوکار داخلی خود استفاده می‌کند. روش دقیق مدیریت Collision و سازمان‌دهی Bucketها به پیاده‌سازی کتابخانهٔ استاندارد وابسته است.

### مفهوم Load Factor

Load Factor نسبت تعداد عناصر ذخیره‌شده به تعداد Bucketها است:

\[
\text{Load Factor}=\frac{\text{Size}}{\text{Bucket Count}}
\]

اگر Container دارای ۱۰۰ عنصر و ۲۰۰ Bucket باشد، Load Factor برابر با ۰٫۵ خواهد بود.

افزایش Load Factor می‌تواند باعث افزایش احتمال برخورد و افزایش هزینهٔ جست‌وجو شود. به همین دلیل، `std::unordered_map` امکان کنترل Load Factor و تعداد Bucketها را فراهم می‌کند.

### مفهوم Rehash

وقتی تعداد عناصر افزایش می‌یابد یا شرایط ظرفیت Hash Table تغییر می‌کند، Container ممکن است عملیات Rehash را انجام دهد. در این عملیات، عناصر بر اساس ساختار جدید Bucketها دوباره سازمان‌دهی می‌شوند.

عملیات Rehash می‌تواند پرهزینه باشد، زیرا ممکن است نیازمند تخصیص حافظه و پردازش مجدد تعداد زیادی عنصر باشد.

در نتیجه، اگر تعداد تقریبی عناصر از قبل مشخص است، بهتر است ظرفیت موردنیاز را پیش از درج گستردهٔ داده‌ها تنظیم کنید.

## روش ایجاد و مقداردهی اولیه

### ایجاد یک Container خالی

ساده‌ترین روش ساخت Container به شکل زیر است:

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores;
}
```

در این مثال، کلید از نوع `std::string` و مقدار از نوع `int` است.

### مقداردهی اولیه با فهرست عناصر

از C++11 می‌توان برای مقداردهی اولیه از Initializer List استفاده کرد:

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87},
        {"Charlie", 91}
    };
}
```

در این روش، هر عنصر شامل یک کلید و مقدار متناظر آن است.

اگر کلیدی بیش از یک بار در Initializer List ظاهر شود، نمی‌توان به‌طور قابل‌حمل به یک مقدار مشخص برای انتخاب میان عناصر تکراری اتکا کرد. برای داده‌هایی که یکتایی کلیدها در آن‌ها تضمین نشده است، بهتر است ابتدا سیاست رفع تکرار را مشخص کنید یا از درج کنترل‌شده استفاده کنید.

### ایجاد با ظرفیت اولیه

برای کاهش احتمال Rehash می‌توان ظرفیت تقریبی موردنیاز را پیش از درج عناصر مشخص کرد:

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores;

    scores.reserve(1000);

    for (int i = 0; i < 1000; ++i) {
        scores.emplace("user_" + std::to_string(i), i);
    }
}
```

تابع `reserve` تعداد Bucketهای موردنیاز را بر اساس تعداد عنصر هدف و `max_load_factor` تنظیم می‌کند.

این تابع تضمین نمی‌کند که دقیقاً به تعداد مشخصی Bucket دسترسی داشته باشید؛ کتابخانهٔ استاندارد می‌تواند تعداد دیگری را انتخاب کند.

همچنین، `reserve` به معنای رزرو دقیق حافظه برای تمام رشته‌ها و اشیای ذخیره‌شده نیست؛ تخصیص حافظهٔ خود عناصر و داده‌های پویا همچنان به نوع و اندازهٔ آن‌ها بستگی دارد.

## عملیات اصلی روی عناصر

### دسترسی با عملگر `operator[]`

عملگر `[]` برای دسترسی به مقدار یک کلید استفاده می‌شود:

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> inventory;

    inventory["apple"] = 10;
    inventory["banana"] = 20;

    std::cout << inventory["apple"] << '\n';
}
```

اگر کلید وجود نداشته باشد، `operator[]` یک عنصر جدید با همان کلید ایجاد می‌کند و مقدار آن را به‌صورت Value Initialization مقداردهی می‌کند.

برای مثال، در یک `std::unordered_map<std::string, int>`، مقدار اولیهٔ یک کلید جدید برابر با صفر خواهد بود.

این رفتار هنگام خواندن داده‌ها اهمیت زیادی دارد. کد زیر فقط مقدار را نمی‌خواند؛ در صورت نبود کلید، عنصر جدیدی نیز ایجاد می‌کند:

```cpp
int value = inventory["orange"];
```

اگر هدف صرفاً بررسی وجود کلید یا خواندن مقدار آن است، از `find` یا `at` استفاده کنید.

### دسترسی کنترل‌شده با `at`

تابع `at` فقط برای کلیدهای موجود مقدار را برمی‌گرداند:

```cpp
#include <iostream>
#include <stdexcept>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> inventory{
        {"apple", 10},
        {"banana", 20}
    };

    try {
        std::cout << inventory.at("apple") << '\n';
        std::cout << inventory.at("orange") << '\n';
    } catch (const std::out_of_range&) {
        std::cout << "Key not found\n";
    }
}
```

اگر کلید وجود نداشته باشد، `at` استثنای `std::out_of_range` پرتاب می‌کند.

این تابع برای دسترسی‌ای مناسب است که نبود کلید در آن یک وضعیت خطا محسوب می‌شود.

### بررسی وجود کلید با `find`

تابع `find` یک Iterator به عنصر پیدا‌شده برمی‌گرداند. اگر کلید موجود نباشد، مقدار بازگشتی برابر با `end()` است.

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87}
    };

    auto it = scores.find("Bob");

    if (it != scores.end()) {
        std::cout << it->first << ": " << it->second << '\n';
    }
}
```

این روش زمانی مناسب است که علاوه بر بررسی وجود کلید، به مقدار یا خود عنصر نیز نیاز دارید.

### بررسی وجود کلید با `contains`

در استاندارد C++20 تابع `contains` به Containerهای Associative و Unordered Associative اضافه شد.

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87}
    };

    if (scores.contains("Alice")) {
        // Key exists
    }
}
```

این تابع در C++11 وجود ندارد. برای سازگاری با C++11 باید از `find` استفاده کنید:

```cpp
if (scores.find("Alice") != scores.end()) {
    // Key exists
}
```

### افزودن عنصر با `insert`

تابع `insert` عنصر جدیدی را درج می‌کند، مشروط بر اینکه کلید آن از قبل موجود نباشد.

```cpp
#include <iostream>
#include <string>
#include <unordered_map>
#include <utility>

int main() {
    std::unordered_map<std::string, int> scores;

    auto result = scores.insert(std::make_pair("Alice", 95));

    if (result.second) {
        std::cout << "Inserted successfully\n";
    } else {
        std::cout << "Key already exists\n";
    }
}
```

مقدار بازگشتی `insert` در این حالت یک `std::pair` است:

- عضو `first` یک Iterator به عنصر موجود یا عنصر درج‌شده است.
- عضو `second` مشخص می‌کند که درج انجام شده است یا خیر.

اگر کلید تکراری باشد، مقدار قبلی جایگزین نمی‌شود.

### افزودن عنصر با `emplace`

تابع `emplace` امکان ساخت عنصر در محل موردنیاز را فراهم می‌کند:

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores;

    scores.emplace("Alice", 95);
    scores.emplace("Bob", 87);
}
```

این تابع می‌تواند از ساخت اشیای موقت غیرضروری جلوگیری کند، اما تضمین نمی‌کند که در همهٔ شرایط سریع‌تر از `insert` باشد. هزینهٔ Hash کردن کلید، بررسی تکراری بودن و تخصیص حافظه نیز باید در نظر گرفته شود.

### افزودن یا به‌روزرسانی با `insert_or_assign`

تابع `insert_or_assign` از C++17 در دسترس است. اگر کلید وجود نداشته باشد، عنصر را درج می‌کند؛ در غیر این صورت، مقدار موجود را جایگزین می‌کند.

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores;

    scores.insert_or_assign("Alice", 95);
    scores.insert_or_assign("Alice", 98);
}
```

در این مثال، مقدار نهایی کلید `Alice` برابر با ۹۸ است.

در C++11 باید این رفتار را با بررسی وجود کلید یا استفاده از `operator[]` و سیاست مناسب پیاده‌سازی کنید.

### درج شرطی با `try_emplace`

تابع `try_emplace` از C++17 معرفی شده است. این تابع تنها زمانی مقدار جدید را می‌سازد که کلید وجود نداشته باشد.

```cpp
#include <memory>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, std::unique_ptr<int>> values;

    values.try_emplace("first", new int(42));
}
```

در این مثال، ساخت `unique_ptr` با انتقال مالکیت انجام می‌شود؛ با این حال، عبارت `new int(42)` پیش از فراخوانی تابع ارزیابی می‌شود. بنابراین این مثال به‌تنهایی از تخصیص حافظه در صورت وجود کلید جلوگیری نمی‌کند.

برای بهره‌گیری از ساخت کنترل‌شده‌تر، می‌توان از `std::make_unique` در C++14 استفاده کرد، اما توجه داشته باشید که آرگومان‌های فراخوانی تابع همچنان پیش از ورود به تابع ارزیابی می‌شوند.

مزیت اصلی `try_emplace` این است که اگر کلید موجود باشد، آرگومان‌های ارسال‌شده برای ساخت شیء مقصد به سازندهٔ مقدار جدید منتقل نمی‌شوند.

## به‌روزرسانی و حذف عناصر

### تغییر مقدار موجود

برای به‌روزرسانی مقدار یک کلید می‌توان از `operator[]` استفاده کرد:

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95}
    };

    scores["Alice"] = 98;
}
```

در این مثال، چون کلید وجود دارد، مقدار آن تغییر می‌کند.

اگر وجود کلید تضمین نشده باشد، باید به این نکته توجه کرد که `operator[]` ممکن است عنصر جدید ایجاد کند.

### حذف عنصر با `erase`

تابع `erase` امکان حذف عناصر را فراهم می‌کند:

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87},
        {"Charlie", 91}
    };

    scores.erase("Bob");
}
```

در C++11، فراخوانی `erase(key)` تعداد عناصر حذف‌شده را برمی‌گرداند. این مقدار برای کلیدهای یکتا معمولاً صفر یا یک است.

در C++20، تابع `erase_if` برای حذف شرطی عناصر از Containerهای استاندارد مربوطه اضافه شد. در C++11 می‌توان از الگوی Iterator مناسب استفاده کرد.

### حذف شرطی با Iterator

کد زیر عناصر دارای مقدار کمتر از ۹۰ را حذف می‌کند:

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87},
        {"Charlie", 91},
        {"David", 78}
    };

    for (auto it = scores.begin(); it != scores.end();) {
        if (it->second < 90) {
            it = scores.erase(it);
        } else {
            ++it;
        }
    }
}
```

در این الگو، Iterator بازگشتی از `erase` برای ادامهٔ پیمایش استفاده می‌شود. این روش از دسترسی به Iterator حذف‌شده جلوگیری می‌کند.

### پاک‌کردن تمام عناصر

برای حذف تمام عناصر، می‌توان از `clear` استفاده کرد:

```cpp
scores.clear();
```

این تابع عناصر را حذف می‌کند، اما الزاماً تعداد Bucketها را کاهش نمی‌دهد. بنابراین اگر هدف کاهش ظرفیت حافظه باشد، باید رفتار پیاده‌سازی و نیاز واقعی برنامه را جداگانه بررسی کنید.

## پیمایش عناصر

### پیمایش با Range-based for

برای پیمایش تمام عناصر، Range-based for معمولاً خواناترین گزینه است:

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87},
        {"Charlie", 91}
    };

    for (const auto& entry : scores) {
        std::cout << entry.first << ": " << entry.second << '\n';
    }
}
```

هر عنصر در این Container از نوعی مشابه `std::pair<const Key, T>` است. بنابراین کلید عنصر از طریق Iterator قابل تغییر نیست، اما مقدار آن در شرایط مناسب قابل تغییر است.

### پیمایش با Iterator

برای کنترل دقیق‌تر نحوهٔ پیمایش می‌توان از Iterator استفاده کرد:

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scores{
        {"Alice", 95},
        {"Bob", 87}
    };

    for (auto it = scores.begin(); it != scores.end(); ++it) {
        std::cout << it->first << ": " << it->second << '\n';
    }
}
```

در این مثال، Iterator امکان دسترسی مستقیم به کلید و مقدار را فراهم می‌کند.

### نبود ترتیب تضمین‌شده

ترتیب پیمایش عناصر در `std::unordered_map` تضمین نمی‌شود. حتی اگر در چند اجرای متوالی عناصر با ترتیب یکسانی دیده شوند، نباید برنامه را به آن وابسته کرد.

ترتیب پیمایش ممکن است با درج یا حذف عناصر، Rehash، تغییر پیاده‌سازی کتابخانه یا تغییر نسخهٔ آن متفاوت شود.

اگر خروجی مرتب نیاز دارید، می‌توانید از `std::map` استفاده کنید یا عناصر را به Container دیگری منتقل کرده و جداگانه مرتب کنید.

## مدیریت ظرفیت و عملکرد

### تنظیم ظرفیت با `reserve`

تابع `reserve` برای آماده‌سازی Container جهت ذخیرهٔ تعداد مشخصی عنصر استفاده می‌شود:

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> counts;

    counts.reserve(10000);

    for (int i = 0; i < 10000; ++i) {
        counts.emplace("item_" + std::to_string(i), i);
    }
}
```

اگر تعداد عناصر نهایی تقریباً مشخص باشد، این کار می‌تواند هزینهٔ Rehashهای مکرر را کاهش دهد.

با این حال، `reserve` به‌تنهایی تضمین‌کنندهٔ عملکرد سریع نیست؛ کیفیت Hash Function و نحوهٔ توزیع کلیدها نیز اهمیت دارند.

### تنظیم Load Factor با `max_load_factor`

تابع `max_load_factor` حداکثر Load Factor موردنظر را تعیین می‌کند:

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> counts;

    counts.max_load_factor(0.7f);
    counts.reserve(10000);
}
```

در این مثال، ابتدا حد موردنظر تعیین می‌شود و سپس ظرفیت رزرو می‌شود.

ترتیب این دو عملیات اهمیت دارد؛ زیرا مقدار Load Factor هنگام محاسبهٔ ظرفیت لازم توسط `reserve` استفاده می‌شود.

انتخاب Load Factor کوچک‌تر می‌تواند تعداد Bucketها و مصرف حافظه را افزایش دهد. در مقابل، Load Factor بزرگ‌تر ممکن است مصرف حافظه را کاهش دهد، اما هزینهٔ Collision و جست‌وجو را افزایش دهد.

مقدار مناسب به پیاده‌سازی، الگوی کلیدها، محدودیت حافظه و معیارهای عملکرد بستگی دارد.

### بررسی تعداد Bucketها

برای مشاهدهٔ وضعیت Hash Table می‌توان از توابع زیر استفاده کرد:

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> counts;

    counts.reserve(100);

    for (int i = 0; i < 100; ++i) {
        counts.emplace("key_" + std::to_string(i), i);
    }

    std::cout << "Size: " << counts.size() << '\n';
    std::cout << "Buckets: " << counts.bucket_count() << '\n';
    std::cout << "Load factor: " << counts.load_factor() << '\n';
    std::cout << "Maximum load factor: "
              << counts.max_load_factor() << '\n';
}
```

این توابع برای تحلیل رفتار Container مفید هستند، اما نباید انتظار داشت که تعداد Bucketها دقیقاً برابر مقدار مشخصی باشد.

### بررسی وضعیت یک Bucket

برای بررسی نحوهٔ توزیع عناصر می‌توان از `bucket` و `bucket_size` استفاده کرد:

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> values{
        {"apple", 1},
        {"banana", 2},
        {"orange", 3}
    };

    auto index = values.bucket("apple");

    std::cout << "Bucket index: " << index << '\n';
    std::cout << "Bucket size: "
              << values.bucket_size(index) << '\n';
}
```

این قابلیت برای تحلیل Collisionها و بررسی توزیع عناصر مفید است. با این حال، نتایج به پیاده‌سازی و Hash Function مورد استفاده وابسته‌اند.

## طراحی Hash Function سفارشی

### تعریف Hash برای یک نوع سفارشی

برای استفاده از نوع سفارشی به‌عنوان کلید، معمولاً باید Hash Function و تابع برابری مناسب تعریف شوند.

فرض کنید می‌خواهیم اطلاعات افراد را با استفاده از یک شناسهٔ ترکیبی ذخیره کنیم:

```cpp
#include <cstddef>
#include <functional>
#include <string>
#include <unordered_map>

struct UserKey {
    int id;
    std::string region;

    bool operator==(const UserKey& other) const {
        return id == other.id && region == other.region;
    }
};

struct UserKeyHash {
    std::size_t operator()(const UserKey& key) const {
        std::size_t h1 = std::hash<int>{}(key.id);
        std::size_t h2 = std::hash<std::string>{}(key.region);

        return h1 ^ (h2 + 0x9e3779b9u + (h1 << 6) + (h1 >> 2));
    }
};

int main() {
    std::unordered_map<UserKey, std::string, UserKeyHash> users;

    users.emplace(UserKey{1, "EU"}, "Alice");
    users.emplace(UserKey{2, "US"}, "Bob");
}
```

در این مثال، ساختار `UserKey` دارای دو فیلد است و برابری آن بر اساس هر دو فیلد تعیین می‌شود.

تابع Hash نیز Hash هر فیلد را محاسبه و ترکیب می‌کند. فرمول ترکیب Hash در این مثال یک الگوی ساده و متداول است، اما بهترین انتخاب برای تمام داده‌ها محسوب نمی‌شود.

### رابطهٔ میان Hash و برابری

یکی از مهم‌ترین الزامات `std::unordered_map` این است که اگر دو کلید برابر باشند، Hash آن‌ها نیز باید برابر باشد:

\[
a == b \implies hash(a) = hash(b)
\]

عکس این رابطه الزامی نیست؛ یعنی برابر بودن Hash دو کلید به معنای برابر بودن خود کلیدها نیست.

اگر تابع Hash و تابع برابری با یکدیگر سازگار نباشند، رفتار Container با الزامات استاندارد سازگار نخواهد بود.

### ویژگی‌های مهم Hash Function

یک Hash Function مناسب باید:

- برای کلید یکسان، در طول عمر مربوطه رفتار سازگار داشته باشد.
- برای کلیدهای برابر، مقدار Hash یکسان تولید کند.
- توزیع مناسبی برای کلیدهای مورد انتظار داشته باشد.
- هزینهٔ محاسباتی قابل‌قبولی داشته باشد.
- از وابستگی به وضعیت متغیری که نتیجهٔ Hash را تغییر می‌دهد، اجتناب کند.

توجه کنید که یک Hash Function سریع اما با توزیع ضعیف می‌تواند عملکرد کلی Hash Table را به‌شدت کاهش دهد.

### استفاده از تابع برابری سفارشی

گاهی برابری کلیدها با `operator==` پیش‌فرض مطابقت ندارد. برای مثال، ممکن است بخواهیم رشته‌ها بدون توجه به بزرگی و کوچکی حروف برابر در نظر گرفته شوند.

در چنین شرایطی باید هم Hash Function و هم تابع برابری را مطابق یک سیاست یکسان تعریف کنید. اگر تابع برابری دو رشته را برابر بداند، Hash آن دو رشته نیز باید برابر باشد.

استفاده از مقایسهٔ بدون حساسیت به حروف همراه با Hash معمولی رشته، بدون اطمینان از سازگاری این دو تابع، اشتباه است.

## استفاده از کلیدهای رشته‌ای و حساسیت به حروف

### رفتار پیش‌فرض رشته‌ها

در حالت پیش‌فرض، مقایسهٔ رشته‌های `std::string` به‌صورت حساس به حروف انجام می‌شود.

بنابراین کلیدهای زیر متفاوت هستند:

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> values;

    values["Admin"] = 1;
    values["admin"] = 2;

    std::cout << values.size() << '\n';
}
```

خروجی برابر با `2` خواهد بود.

اگر برنامه به سیاست‌هایی مانند Case-insensitive Lookup نیاز دارد، باید سیاست نرمال‌سازی یا توابع Hash و برابری سفارشی را به‌صورت هماهنگ طراحی کند.

در بسیاری از برنامه‌ها، نرمال‌سازی کلید هنگام ورود داده‌ها راهکار ساده‌تری است؛ البته نرمال‌سازی Unicode، به‌ویژه در زبان‌های مختلف، به سیاست مشخص و پیاده‌سازی دقیق نیاز دارد.

## مدیریت Iteratorها و ارجاع‌ها

### اعتبار Iteratorها پس از درج

یکی از نکات مهم در کار با `std::unordered_map`، اعتبار Iteratorها پس از درج عنصر است.

اگر درج عنصر باعث Rehash شود، Iteratorهای مربوط به عناصر Container نامعتبر می‌شوند.

برای مثال، نگهداری Iterator در بخشی از برنامه و سپس انجام عملیات درج بدون توجه به احتمال Rehash می‌تواند باعث بروز خطا شود.

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> values{
        {"a", 1},
        {"b", 2}
    };

    auto it = values.find("a");

    values.reserve(1000);

    // Do not use the old iterator after a possible rehash.
}
```

پس از `reserve` نباید فرض کرد Iterator قبلی همچنان معتبر است، زیرا عملیات ممکن است باعث Rehash شود.

### اعتبار Pointer و Reference

در مقابل، Rehash به‌خودی‌خود Pointerها و Referenceهای اشاره‌کننده به عناصر موجود را نامعتبر نمی‌کند. این تفاوت یکی از ویژگی‌های مهم Containerهای Unordered است.

با این حال، حذف عنصر باعث نامعتبر شدن Pointerها، Referenceها و Iteratorهای مربوط به همان عنصر می‌شود.

همچنین، اگر خود شیء ذخیره‌شده به‌نحوی تغییر کند که آدرس یا طول عمر آن را تحت تأثیر قرار دهد، باید قوانین همان نوع را نیز در نظر گرفت.

### تغییر کلیدهای عناصر

کلید هر عنصر در `std::unordered_map` به‌صورت `const` ذخیره می‌شود. این محدودیت برای حفظ سازگاری ساختار Hash Table ضروری است.

تغییر مستقیم کلید می‌تواند محل منطقی عنصر را با Hash فعلی ناسازگار کند؛ به همین دلیل، چنین تغییری از طریق Iterator مجاز نیست.

اگر نیاز به تغییر کلید دارید، روش متداول حذف عنصر و درج مجدد آن با کلید جدید است.

در C++17 قابلیت Node Handle امکان انتقال و تغییر کلید برخی عناصر را بدون نیاز به ساخت مجدد کامل شیء فراهم کرد.

## قابلیت‌های پیشرفته در نسخه‌های جدیدتر

### استخراج عنصر با `extract`

قابلیت Node Handle در C++17 معرفی شد. با استفاده از `extract` می‌توان یک عنصر را از Container خارج کرد، درحالی‌که Node آن برای درج مجدد یا تغییر کلید در دسترس باقی می‌ماند.

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> values{
        {"old_key", 42}
    };

    auto node = values.extract("old_key");

    if (!node.empty()) {
        node.key() = "new_key";
        values.insert(std::move(node));
    }
}
```

در این مثال، کلید Node Handle پس از استخراج تغییر می‌کند و سپس عنصر دوباره درج می‌شود.

این قابلیت برای انتقال عناصر بین Containerهای سازگار و تغییر کلید بدون کپی کردن کامل مقدار مفید است.

### انتقال عناصر با `merge`

از C++17 می‌توان از `merge` برای انتقال Nodeهای قابل‌انتقال بین Containerهای سازگار استفاده کرد:

```cpp
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> source{
        {"a", 1},
        {"b", 2}
    };

    std::unordered_map<std::string, int> destination{
        {"b", 20},
        {"c", 3}
    };

    destination.merge(source);
}
```

در این مثال، عنصر `a` به مقصد منتقل می‌شود؛ اما عنصر `b` به دلیل وجود کلید تکراری در مقصد، در `source` باقی می‌ماند.

انتقال Nodeها به سازگاری Containerها، از جمله الزامات مربوط به Allocator، وابسته است.

### جست‌وجوی Heterogeneous

در C++20 قابلیت Heterogeneous Lookup برای بسیاری از Containerهای Associative و Unordered Associative با پشتیبانی از توابع Hash و برابری Transparent استاندارد شد.

این قابلیت اجازه می‌دهد در شرایط مناسب، بدون ساخت شیء موقت از نوع کلید اصلی، جست‌وجو با نوع دیگری انجام شود.

برای نمونه، اگر کلیدها از نوع `std::string` باشند، امکان جست‌وجو با `std::string_view` می‌تواند از ساخت رشتهٔ موقت جلوگیری کند.

این قابلیت به تعریف Hash و Equality سازگار و Transparent نیاز دارد. در C++11 نمی‌توان وجود این APIها را فرض کرد.

## کاربردهای عملی

### شمارش تعداد تکرار عناصر

یکی از کاربردهای متداول `std::unordered_map` شمارش فراوانی عناصر است:

```cpp
#include <iostream>
#include <string>
#include <unordered_map>
#include <vector>

int main() {
    std::vector<std::string> words{
        "apple", "banana", "apple", "orange", "banana", "apple"
    };

    std::unordered_map<std::string, int> frequency;

    for (const auto& word : words) {
        ++frequency[word];
    }

    for (const auto& entry : frequency) {
        std::cout << entry.first << ": " << entry.second << '\n';
    }
}
```

در این مثال، `operator[]` در صورت نبود کلید، شمارنده را با مقدار صفر ایجاد می‌کند و سپس آن را افزایش می‌دهد.

ترتیب خروجی تضمین‌شده نیست.

### ساخت Cache

برای نگهداری نتیجهٔ محاسبات یا پاسخ درخواست‌ها می‌توان از این Container به‌عنوان Cache استفاده کرد.

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int calculate(int value) {
    return value * value;
}

int main() {
    std::unordered_map<int, int> cache;

    auto getValue = [&cache](int input) {
        auto it = cache.find(input);

        if (it != cache.end()) {
            return it->second;
        }

        int result = calculate(input);
        cache.emplace(input, result);

        return result;
    };

    std::cout << getValue(10) << '\n';
    std::cout << getValue(10) << '\n';
}
```

در این مثال، نتیجهٔ محاسبه برای ورودی تکراری دوباره محاسبه نمی‌شود.

البته این پیاده‌سازی ساده فاقد سیاست‌هایی مانند محدودیت اندازه، انقضای داده، مدیریت هم‌زمانی و حذف هوشمند عناصر قدیمی است. برای Cacheهای واقعی باید این نیازها جداگانه بررسی شوند.

### نگاشت شناسه به شیء

این Container برای نگاشت شناسه‌ها به داده‌های برنامه، مانند کاربران، Sessionها یا منابع داخلی، مناسب است.

```cpp
#include <string>
#include <unordered_map>

struct User {
    std::string name;
    int age;
};

int main() {
    std::unordered_map<int, User> users;

    users.emplace(101, User{"Alice", 30});
    users.emplace(102, User{"Bob", 25});
}
```

در این ساختار، شناسه به‌عنوان کلید عمل می‌کند و اطلاعات مربوط به هر کاربر در مقدار نگهداری می‌شود.

### ساخت Index برای جست‌وجوی سریع

در برنامه‌هایی که داده‌ها ابتدا به‌صورت مجموعه‌ای از اشیا ذخیره می‌شوند، می‌توان یک Index جداگانه با `std::unordered_map` ایجاد کرد تا دسترسی بر اساس شناسه سریع‌تر شود.

در چنین طراحی‌ای باید رابطهٔ میان دادهٔ اصلی و Index را حفظ کرد. اگر دادهٔ اصلی تغییر کند یا حذف شود، Index نیز باید به‌روزرسانی شود؛ در غیر این صورت ممکن است به اشیای ناموجود یا داده‌های قدیمی اشاره کند.

## محدودیت‌ها و خطاهای رایج

### فرض‌کردن پیچیدگی ثابت

پیچیدگی متوسط جست‌وجو در `std::unordered_map` برابر \(O(1)\) است، اما این به معنای تضمین زمان ثابت برای هر عملیات نیست.

در بدترین حالت، هزینهٔ جست‌وجو، درج یا حذف می‌تواند به \(O(n)\) برسد؛ برای مثال، زمانی که تعداد زیادی کلید در یک Bucket قرار بگیرند.

بنابراین، برای کاربردهای حساس به زمان پاسخ باید رفتار واقعی Container با داده‌های نماینده اندازه‌گیری شود.

### استفاده از Hash Function نامناسب

اگر Hash Function برای کلیدهای مورد انتظار توزیع مناسبی نداشته باشد، Collisionها افزایش می‌یابند و عملکرد کاهش پیدا می‌کند.

برای انواع سفارشی، انتخاب تابع Hash باید بر اساس ویژگی‌های واقعی داده انجام شود. استفاده از یک فرمول ساده بدون بررسی توزیع کلیدها ممکن است در داده‌های واقعی نتیجهٔ مطلوبی نداشته باشد.

### استفادهٔ ناخواسته از `operator[]`

یکی از اشتباهات رایج، استفاده از `operator[]` صرفاً برای بررسی وجود کلید است:

```cpp
int value = values["missing"];
```

این کد ممکن است عنصر جدیدی ایجاد کند. اگر ایجاد عنصر موردنظر نیست، باید از `find` استفاده شود.

### وابستگی به ترتیب پیمایش

نباید ترتیب پیمایش `std::unordered_map` را بخشی از قرارداد برنامه در نظر گرفت. این اشتباه می‌تواند باعث شکست تست‌ها یا تغییر خروجی میان نسخه‌های مختلف محیط اجرا شود.

اگر ترتیب اهمیت دارد، باید از ساختار دادهٔ مرتب یا مرتب‌سازی صریح خروجی استفاده کرد.

### تغییر وضعیت مؤثر بر Hash کلید

تابع Hash و تابع برابری باید در مدت حضور کلید در Container با یکدیگر سازگار باقی بمانند.

اگر Hash یا برابری کلید به دادهٔ متغیری وابسته باشد که در طول حضور کلید تغییر می‌کند، ممکن است Container نتواند کلید را مطابق انتظار پیدا کند.

به همین دلیل، Hash Function و Equality Predicate باید بر مبنای ویژگی‌های پایدار کلید طراحی شوند.

### فرض‌کردن ایمنی در محیط چندنخی

`std::unordered_map` به‌صورت خودکار یک Container Thread-safe برای دسترسی‌های هم‌زمان همراه با تغییر داده‌ها نیست.

اگر چند Thread به‌طور هم‌زمان عملیات نوشتن انجام دهند، یا یک Thread داده را تغییر دهد و Thread دیگری بدون هماهنگی به همان Container دسترسی داشته باشد، ممکن است Data Race رخ دهد.

برای استفادهٔ هم‌زمان باید از Synchronization مناسب، مانند Mutex، یا ساختارهای دادهٔ مخصوص Concurrent Access استفاده شود.

### بی‌توجهی به مصرف حافظه

در بسیاری از پیاده‌سازی‌ها، Hash Table علاوه بر حافظهٔ موردنیاز عناصر، برای Bucketها و ساختارهای مدیریتی نیز حافظه مصرف می‌کند.

بنابراین، `std::unordered_map` ممکن است از Containerهای دیگر حافظهٔ بیشتری مصرف کند، به‌خصوص زمانی که تعداد عناصر کم باشد یا Load Factor بسیار پایین تنظیم شده باشد.

در برنامه‌هایی با محدودیت جدی حافظه، اندازه‌گیری واقعی مصرف حافظه و مقایسه با گزینه‌های جایگزین ضروری است.

## نکات عملکردی برای برنامه‌های حرفه‌ای

### رزرو ظرفیت پیش از درج انبوه

اگر تعداد عناصر نهایی قابل‌پیش‌بینی است، استفاده از `reserve` پیش از درج انبوه معمولاً می‌تواند تعداد Rehashها را کاهش دهد.

با این حال، بهتر است ظرفیت موردنیاز بر اساس برآورد واقع‌بینانه انتخاب شود؛ رزرو بسیار بزرگ ممکن است مصرف حافظه را بی‌دلیل افزایش دهد.

### انتخاب نوع مناسب برای کلید

هزینهٔ Hash کردن کلید در عملکرد کلی Container مؤثر است. برای مثال، Hash کردن یک عدد صحیح معمولاً ارزان‌تر از Hash کردن یک رشتهٔ طولانی است.

در برنامه‌هایی که تعداد زیادی Lookup انجام می‌دهند، هزینهٔ ساخت کلید، کپی رشته‌ها و تخصیص حافظه نیز باید در ارزیابی عملکرد در نظر گرفته شود.

### پرهیز از کپی‌های غیرضروری

هنگام پیمایش عناصر، استفاده از Reference ثابت در مواردی که نیازی به تغییر مقدار ندارید، از کپی شدن عناصر جلوگیری می‌کند:

```cpp
for (const auto& entry : values) {
    // Process the existing element without copying it
}
```

در این مثال، عناصر Container کپی نمی‌شوند و داده‌ها به‌صورت فقط‌خواندنی در دسترس هستند.

### اندازه‌گیری به‌جای حدس

سرعت `std::unordered_map` به عوامل متعددی از جمله نوع کلید، کیفیت Hash Function، تعداد عناصر، Cache Locality، الگوی دسترسی و پیاده‌سازی کتابخانه بستگی دارد.

به همین دلیل، برای انتخاب نهایی باید Benchmark با داده‌ها و الگوهای دسترسی نزدیک به محیط واقعی انجام شود.

## مقایسه با گزینه‌های جایگزین

### مقایسه با `std::vector`

اگر تعداد عناصر کم است یا پیمایش ترتیبی اهمیت زیادی دارد، جست‌وجوی خطی روی `std::vector` ممکن است در عمل عملکرد رقابتی یا حتی بهتری داشته باشد.

دلیل این موضوع، پیوستگی حافظه و رفتار مناسب Cache در بسیاری از عملیات‌های ترتیبی است.

در مقابل، برای جست‌وجوی مکرر بر اساس کلید در مجموعه‌های بزرگ، `std::unordered_map` معمولاً گزینهٔ مناسب‌تری است.

### مقایسه با `std::unordered_set`

اگر فقط به ذخیره و بررسی وجود کلیدها نیاز دارید و هیچ مقداری به کلید متصل نیست، `std::unordered_set` انتخاب مناسب‌تری است.

این Container نیز بر اساس Hash Table کار می‌کند، اما به‌جای نگاشت کلید به مقدار، صرفاً مجموعه‌ای از کلیدهای یکتا را نگهداری می‌کند.

### مقایسه با `std::map`

اگر برنامه به مرتب‌سازی کلیدها، جست‌وجوی بازه‌ای یا دسترسی به اولین و آخرین کلید نیاز دارد، `std::map` مزیت‌های مهمی دارد.

در مقابل، اگر چنین نیازهایی وجود ندارد و Lookup سریع بر اساس کلید اولویت دارد، `std::unordered_map` می‌تواند انتخاب مناسب‌تری باشد.

## نکات سازگاری با استانداردهای مختلف

### قابلیت‌های موجود در C++11

قابلیت‌های اصلی `std::unordered_map` از C++11 در دسترس هستند، از جمله:

- ساخت و مقداردهی اولیهٔ Container.
- دسترسی با `operator[]` و `at`.
- جست‌وجو با `find`.
- درج با `insert` و `emplace`.
- حذف با `erase`.
- مدیریت ظرفیت با `reserve` و `rehash`.
- تنظیم و بررسی Load Factor.
- تعریف Hash Function و Equality Predicate سفارشی.
- پشتیبانی از Move Semantics و Initializer List.

برای استفاده از این قابلیت‌ها، پروژه باید با استاندارد C++11 یا استاندارد جدیدتری کامپایل شود.

### قابلیت‌های اضافه‌شده در C++17

استاندارد C++17 امکاناتی مانند `try_emplace`، `insert_or_assign`، Node Handle و `merge` را اضافه کرد.

این قابلیت‌ها برای کنترل بهتر ساخت عناصر، تغییر کلیدها و انتقال عناصر بین Containerها مفید هستند.

### قابلیت‌های اضافه‌شده در C++20

در C++20، تابع `contains` و قابلیت‌های استانداردشدهٔ Heterogeneous Lookup برای Containerهای مربوطه در دسترس قرار گرفتند.

در پروژه‌هایی که باید با C++11 سازگار باشند، نباید بدون بررسی نسخهٔ استاندارد از این APIها استفاده کرد.

## جمع‌بندی

ساختار `std::unordered_map` یک ابزار مهم در Standard Library زبان ++C برای نگاشت کلیدهای یکتا به مقادیر است. استفاده از Hash Table امکان جست‌وجو، درج و حذف با پیچیدگی متوسط \(O(1)\) را فراهم می‌کند، اما عملکرد واقعی به کیفیت Hash Function، توزیع کلیدها، Load Factor و شرایط اجرایی وابسته است.

برای استفادهٔ حرفه‌ای از این Container، نکات زیر اهمیت ویژه‌ای دارند:

- برای خواندن کلیدهای موجود، از `find` یا `at` استفاده کنید و به اثر جانبی احتمالی `operator[]` توجه داشته باشید.
- برای کاهش Rehashهای مکرر، در صورت مشخص بودن اندازهٔ نهایی از `reserve` استفاده کنید.
- برای کلیدهای سفارشی، سازگاری Hash Function و Equality Predicate را تضمین کنید.
- به نامعتبر شدن Iteratorها پس از Rehash و حذف عناصر توجه کنید.
- ترتیب پیمایش عناصر را تضمین‌شده فرض نکنید.
- برای کاربردهای حساس به عملکرد یا حافظه، Benchmark و اندازه‌گیری واقعی انجام دهید.
- در صورت نیاز به ترتیب کلیدها یا جست‌وجوی بازه‌ای، گزینه‌هایی مانند `std::map` را نیز بررسی کنید.

در نهایت، انتخاب `std::unordered_map` باید بر اساس نیاز واقعی برنامه انجام شود. این Container برای جست‌وجوی سریع بر اساس کلید بسیار مفید است، اما آشنایی با محدودیت‌ها و رفتارهای آن برای ساخت نرم‌افزارهای قابل‌اعتماد، کارآمد و قابل‌نگهداری ضروری است.

---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>