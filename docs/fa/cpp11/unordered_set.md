<div align="center">

[🇺🇸 English](../../en/cpp11/unordered_set.md) | [🇮🇷 فارسی](./unordered_set.md)

</div>

---

# آموزش جامع std::unordered_set در ++C

## مقدمه و آشنایی با ساختار مجموعه هش‌شده

ساختار `std::unordered_set` یکی از Containerهای استاندارد کتابخانه STL است که از استاندارد ++C11 در دسترس قرار گرفت. این ساختار برای نگهداری مجموعه‌ای از مقادیر یکتا طراحی شده است و با استفاده از سازوکار `Hash Table` امکان درج، جست‌وجو و حذف عناصر را با پیچیدگی زمانی متوسط \(O(1)\) فراهم می‌کند.

برخلاف `std::set` که عناصر را بر اساس ترتیب مقایسه‌ای مرتب نگه می‌دارد، این Container ترتیب مشخصی برای پیمایش عناصر تضمین نمی‌کند. در عوض، تمرکز اصلی آن بر عملیات سریع جست‌وجو و بررسی عضویت است.

برای مثال، اگر بخواهیم شناسه‌های پردازش‌شده، کلمات یکتا، شناسه کاربران یا مقادیر تکراری‌ناپذیر را نگهداری کنیم، این ساختار می‌تواند انتخاب مناسبی باشد.

```cpp
#include <iostream>
#include <string>
#include <unordered_set>

int main() {
    std::unordered_set<std::string> languages;

    languages.insert("C++");
    languages.insert("Python");
    languages.insert("Rust");
    languages.insert("C++");

    std::cout << "Number of unique languages: "
              << languages.size() << '\n';

    for (const auto& language : languages) {
        std::cout << language << '\n';
    }
}
```

در این مثال، رشته `"C++"` فقط یک بار در مجموعه ذخیره می‌شود؛ زیرا `std::unordered_set` مقادیر تکراری را نمی‌پذیرد.

نکته مهم این است که ترتیب چاپ عناصر در این مثال مشخص نیست و ممکن است بین اجراها، نسخه‌های مختلف کتابخانه استاندارد یا تغییرات ساختار داخلی Container متفاوت باشد.

## مفهوم و ویژگی‌های اصلی

### ویژگی‌های اساسی

مهم‌ترین ویژگی‌های `std::unordered_set` عبارت‌اند از:

- **یکتایی عناصر:** هر عنصر بر اساس تعریف `KeyEqual` فقط یک بار در مجموعه حضور دارد.
- **جست‌وجوی سریع:** عملیات جست‌وجو به‌طور متوسط پیچیدگی زمانی \(O(1)\) دارد.
- **عدم تضمین ترتیب:** عناصر بر اساس ترتیب صعودی یا ترتیب درج‌شدن نگهداری نمی‌شوند.
- **مدیریت خودکار حافظه:** تخصیص و آزادسازی حافظه توسط Container و Allocator مدیریت می‌شود.
- **امکان استفاده از نوع‌های سفارشی:** با تعریف Hash Function و تابع برابری مناسب می‌توان از انواع کاربرساخته نیز استفاده کرد.
- **پشتیبانی از Iterator:** امکان پیمایش عناصر با Iteratorهای استاندارد وجود دارد.
- **عدم دسترسی مستقیم به عنصر با Index:** این Container برخلاف `std::vector` دسترسی اندیسی ندارد.

### تفاوت مفهوم یکتایی با مرتب‌بودن

در `std::unordered_set`، یکتایی عناصر به معنای مرتب‌بودن آن‌ها نیست. برای مثال، مجموعه‌ای از اعداد زیر:

```cpp
std::unordered_set<int> values = {40, 10, 30, 20};
```

ممکن است هنگام پیمایش با ترتیبی متفاوت از ترتیب درج‌شدن یا ترتیب عددی نمایش داده شود.

اگر نیاز دارید عناصر به‌صورت مرتب نگهداری شوند، `std::set` معمولاً انتخاب مناسب‌تری است.

اگر ترتیب درج‌شدن عناصر اهمیت دارد، باید از ساختاری دیگر یا ترکیبی از چند Container استفاده کنید.

## تفاوت std::unordered_set و std::set

هر دو Container برای نگهداری عناصر یکتا استفاده می‌شوند، اما سازوکار داخلی و ویژگی‌های عملکردی آن‌ها متفاوت است.

| ویژگی | `std::unordered_set` | `std::set` |
|---|---|---|
| ساختار داخلی متداول | Hash Table | درخت جست‌وجوی متوازن |
| ترتیب عناصر | بدون ترتیب تضمین‌شده | مرتب بر اساس Comparator |
| پیچیدگی متوسط جست‌وجو | \(O(1)\) | \(O(\log n)\) |
| پیچیدگی بدترین حالت جست‌وجو | \(O(n)\) | \(O(\log n)\) |
| درج و حذف | متوسط \(O(1)\) | \(O(\log n)\) |
| پشتیبانی از `lower_bound` | ندارد | دارد |
| نیاز به Hash Function | بله | خیر |
| نیاز به Comparator مرتب‌سازی | خیر | بله |
| حساسیت به کیفیت Hash | زیاد | ندارد |
| پشتیبانی از محدوده‌های مرتب | خیر | بله |

در این جدول، پیچیدگی‌ها بیانگر رفتار استاندارد مورد انتظار هستند؛ پیاده‌سازی داخلی واقعی می‌تواند بین کتابخانه‌های استاندارد متفاوت باشد.

### چه زمانی std::unordered_set مناسب‌تر است؟

استفاده از این ساختار معمولاً زمانی مناسب است که:

- هدف اصلی، بررسی عضویت یک مقدار در مجموعه باشد.
- ترتیب عناصر اهمیتی نداشته باشد.
- عملیات درج، حذف و جست‌وجو پرتکرار باشند.
- Hash Function مناسبی برای نوع کلید وجود داشته باشد.
- حافظه اضافی موردنیاز Hash Table قابل‌قبول باشد.

### چه زمانی std::set انتخاب بهتری است؟

در شرایط زیر، `std::set` می‌تواند انتخاب مناسب‌تری باشد:

- عناصر باید همواره مرتب باشند.
- به کوچک‌ترین یا بزرگ‌ترین عنصر نیاز دارید.
- عملیات `lower_bound` و `upper_bound` لازم است.
- به پیمایش مرتب یا پردازش بازه‌ای نیاز دارید.
- می‌خواهید پیچیدگی بدترین حالت عملیات اصلی همچنان لگاریتمی باشد.

البته عملکرد واقعی باید با داده‌ها و الگوی دسترسی برنامه سنجیده شود. در برخی سناریوها، تفاوت Cache Locality، هزینه Hash کردن و تخصیص حافظه می‌تواند نتیجه را تغییر دهد.

## نحوه تعریف و مقداردهی اولیه

### تعریف مجموعه خالی

برای استفاده از `std::unordered_set` باید Header مربوط به آن را اضافه کنید:

```cpp
#include <unordered_set>
```

سپس می‌توانید مجموعه‌ای از اعداد صحیح تعریف کنید:

```cpp
std::unordered_set<int> numbers;
```

در این حالت، مجموعه خالی است و می‌توان عناصر را با عملیات درج به آن اضافه کرد.

### مقداردهی با فهرست عناصر

در ++C11 امکان استفاده از `Initializer List` وجود دارد:

```cpp
std::unordered_set<int> numbers = {1, 2, 3, 4, 5};
```

همچنین می‌توانید از مقداردهی مستقیم استفاده کنید:

```cpp
std::unordered_set<int> numbers{1, 2, 3, 4, 5};
```

اگر مقدار تکراری وجود داشته باشد، تنها یک نمونه از آن نگهداری می‌شود:

```cpp
std::unordered_set<int> numbers = {1, 2, 2, 3, 3, 3};
```

پس از مقداردهی، مقدار `numbers.size()` برابر با `3` خواهد بود.

### مقداردهی از محدوده Iteratorها

می‌توان عناصر موجود در یک Container دیگر را به مجموعه منتقل کرد:

```cpp
#include <iostream>
#include <unordered_set>
#include <vector>

int main() {
    std::vector<int> values = {1, 2, 3, 2, 4, 1};

    std::unordered_set<int> uniqueValues(
        values.begin(),
        values.end()
    );

    std::cout << uniqueValues.size() << '\n';
}
```

این روش برای حذف مقادیر تکراری از یک مجموعه داده کاربرد دارد.

توجه کنید که ساخت مجموعه یکتا به معنای حفظ ترتیب اولیه نیست.

### تعریف با ظرفیت اولیه

می‌توانید هنگام ساخت مجموعه، تعداد Bucketهای اولیه را مشخص کنید:

```cpp
std::unordered_set<int> numbers(100);
```

آرگومان `100` در این سازنده، تعداد Bucketهای درخواستی اولیه است؛ نه تعداد عناصری که مجموعه می‌تواند بدون محدودیت در خود نگه دارد.

این تفاوت مهم است، زیرا ظرفیت Bucketها با تعداد عناصر متفاوت است و Container می‌تواند در طول عمر خود تغییر اندازه دهد.

## نحوه درج عناصر

### استفاده از insert

تابع `insert` برای درج یک عنصر در مجموعه استفاده می‌شود:

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values;

    values.insert(10);
    values.insert(20);
    values.insert(30);
    values.insert(20);

    std::cout << values.size() << '\n';
}
```

در این مثال، درج دوم عدد `20` باعث ایجاد عنصر جدیدی نمی‌شود.

### بررسی نتیجه درج

در ++C11، فراخوانی `insert` برای یک عنصر منفرد، یک `std::pair` شامل Iterator و مقدار Boolean برمی‌گرداند.

عضو اول مشخص می‌کند عنصر موجود یا درج‌شده کجاست و عضو دوم نشان می‌دهد آیا درج واقعاً انجام شده است یا خیر.

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20};

    auto first = values.insert(30);
    auto second = values.insert(20);

    std::cout << std::boolalpha;
    std::cout << first.second << '\n';
    std::cout << second.second << '\n';
}
```

خروجی:

```text
true
false
```

این قابلیت برای تشخیص شناسه تکراری یا جلوگیری از پردازش مجدد یک مقدار بسیار مفید است.

### استفاده از emplace

تابع `emplace` عنصر را با استفاده از آرگومان‌های داده‌شده در محل مناسب Container می‌سازد:

```cpp
#include <iostream>
#include <string>
#include <unordered_set>

int main() {
    std::unordered_set<std::string> names;

    auto result = names.emplace(10, 'A');

    std::cout << result.first->size() << '\n';
    std::cout << result.second << '\n';
}
```

در این مثال، سازنده رشته برای ایجاد رشته‌ای شامل ده نویسه `A` استفاده می‌شود.

استفاده از `emplace` لزوماً همیشه سریع‌تر از `insert` نیست. مزیت آن به نوع عنصر، سازنده‌های موجود و نحوه انتقال یا ساخت شیء بستگی دارد.

### درج چند عنصر

برای درج مجموعه‌ای از عناصر می‌توان از `insert` با محدوده Iteratorها استفاده کرد:

```cpp
#include <unordered_set>
#include <vector>

int main() {
    std::unordered_set<int> values = {1, 2};

    std::vector<int> additional = {2, 3, 4, 5};

    values.insert(additional.begin(), additional.end());
}
```

عناصر جدید اضافه می‌شوند و مقادیر تکراری نادیده گرفته می‌شوند.

## بررسی عضویت و جست‌وجو

### استفاده از find

تابع `find` برای یافتن یک عنصر استفاده می‌شود. اگر عنصر وجود نداشته باشد، `end()` برگردانده می‌شود.

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto it = values.find(20);

    if (it != values.end()) {
        std::cout << "Found: " << *it << '\n';
    }
}
```

این روش زمانی مفید است که علاوه بر بررسی وجود عنصر، به Iterator آن نیز نیاز داشته باشید.

### استفاده از contains در نسخه‌های جدیدتر

تابع `contains` در استاندارد ++C20 به `std::unordered_set` اضافه شد. این تابع برای بررسی وجود یک عنصر، نتیجه Boolean برمی‌گرداند.

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    bool exists = values.contains(20);
}
```

این API در ++C11 وجود ندارد. بنابراین، اگر پروژه باید با ++C11 سازگار باشد، از `find` یا `count` استفاده کنید.

### استفاده از count

تابع `count` تعداد عناصر معادل با کلید موردنظر را برمی‌گرداند:

```cpp
std::unordered_set<int> values = {10, 20, 30};

if (values.count(20) != 0) {
    // The element exists.
}
```

از آنجا که عناصر در `std::unordered_set` یکتا هستند، مقدار بازگشتی `count` فقط می‌تواند `0` یا `1` باشد.

برای خوانایی بهتر، در کدهای ++C11 معمولاً مقایسه نتیجه `find` با `end()` انتخاب مناسبی است؛ بااین‌حال، استفاده از `count` نیز معتبر است.

### جست‌وجو با نوعی متفاوت از نوع کلید

در ++C11، آرگومان‌های `find`، `count` و `equal_range` باید با نوع کلید سازگار باشند و API استاندارد اولیه، جست‌وجوی عمومی بر اساس نوع متفاوت را فراهم نمی‌کند.

برای مثال، اگر مجموعه‌ای از `std::string` دارید و می‌خواهید با یک `const char*` جست‌وجو کنید، ممکن است ساخت یک `std::string` موقت لازم باشد.

```cpp
#include <string>
#include <unordered_set>

int main() {
    std::unordered_set<std::string> words = {
        "apple",
        "orange",
        "banana"
    };

    const char* query = "apple";

    auto it = words.find(std::string(query));
}
```

جست‌وجوی Heterogeneous با Hash و Equality شفاف در استانداردهای بعدی توسعه پیدا کرد و به پشتیبانی مناسب از `Hash` و `KeyEqual` شفاف نیاز دارد؛ در ++C20 این قابلیت برای عملیات اصلی جست‌وجوی `std::unordered_set` در دسترس استاندارد قرار گرفت.

## حذف عناصر و پاک‌سازی مجموعه

### حذف با استفاده از erase

تابع `erase` می‌تواند یک عنصر را با کلید آن حذف کند:

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    std::size_t removed = values.erase(20);
}
```

اگر عنصر وجود داشته باشد، مقدار بازگشتی `1` است؛ در غیر این صورت، مقدار `0` برگردانده می‌شود.

### حذف با Iterator

اگر Iterator یک عنصر را در اختیار دارید، می‌توانید همان عنصر را حذف کنید:

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto it = values.find(20);

    if (it != values.end()) {
        values.erase(it);
    }
}
```

در ++C11، نسخه Iterator-based تابع `erase`، Iterator عنصر بعدی را برمی‌گرداند. این ویژگی برای حذف امن عناصر هنگام پیمایش کاربرد دارد.

### حذف عناصر در حین پیمایش

روش زیر برای حذف عناصر زوج در یک مجموعه معتبر است:

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {
        1, 2, 3, 4, 5, 6
    };

    for (auto it = values.begin(); it != values.end();) {
        if (*it % 2 == 0) {
            it = values.erase(it);
        } else {
            ++it;
        }
    }
}
```

نکته مهم این است که پس از حذف یک عنصر، نباید Iterator مربوط به آن عنصر را دوباره استفاده کنید.

همچنین، نباید بدون بررسی قواعد اعتبار Iteratorها، از الگوی `erase` در یک Container دیگر به همان شکل استفاده کنید؛ رفتار هر Container متفاوت است.

### حذف تمام عناصر

تابع `clear` تمام عناصر مجموعه را حذف می‌کند:

```cpp
std::unordered_set<int> values = {1, 2, 3};

values.clear();
```

پس از اجرای این تابع، اندازه مجموعه صفر است. بااین‌حال، نباید فرض کنید که حافظه اختصاص‌یافته یا تعداد Bucketها نیز الزاماً به صفر می‌رسد.

اگر هدف آزادسازی حافظه اضافی است، باید ظرفیت Bucketها و سازوکار تخصیص حافظه را نیز در نظر بگیرید.

## ساختار داخلی Hash Table

### مفهوم Hash Function

ساختار `std::unordered_set` برای تعیین محل تقریبی نگهداری هر کلید از یک Hash Function استفاده می‌کند.

این تابع، کلید را به یک مقدار هش از نوع `std::size_t` تبدیل می‌کند. Container با استفاده از مقدار هش و تعداد Bucketها، Bucket مناسب را برای عنصر انتخاب می‌کند.

در ساده‌ترین شکل مفهومی، فرایند جست‌وجو چنین است:

- محاسبه Hash مربوط به کلید.
- تعیین Bucket مرتبط با Hash.
- بررسی عناصر موجود در آن Bucket با استفاده از تابع برابری.
- بازگرداندن عنصر موردنظر یا اعلام عدم وجود آن.

این توضیح، مدل مفهومی است و جزئیات پیاده‌سازی Hash Table در استاندارد مشخص نشده‌اند.

### مفهوم Bucket

هر Hash Table از مجموعه‌ای از Bucketها تشکیل شده است. چند عنصر ممکن است در یک Bucket قرار بگیرند؛ بنابراین، برخورد Hash یا Collision اتفاقی طبیعی است.

تعداد Bucketها را می‌توان با تابع `bucket_count` مشاهده کرد:

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {
        10, 20, 30, 40, 50
    };

    std::cout << values.size() << '\n';
    std::cout << values.bucket_count() << '\n';
    std::cout << values.load_factor() << '\n';
}
```

در این مثال، مقدار دقیق تعداد Bucketها و Load Factor به وضعیت Container و پیاده‌سازی کتابخانه استاندارد بستگی دارد.

### برخورد Hash و اثر آن بر عملکرد

وقتی چند کلید در یک Bucket قرار می‌گیرند، Container باید برای تشخیص کلید صحیح از تابع برابری استفاده کند.

اگر Hash Function کیفیت مناسبی نداشته باشد و تعداد زیادی از عناصر در یک Bucket جمع شوند، عملیات جست‌وجو می‌تواند از عملکرد متوسط مورد انتظار فاصله بگیرد.

در بدترین حالت، جست‌وجو، درج یا حذف می‌تواند پیچیدگی خطی \(O(n)\) داشته باشد.

به همین دلیل، انتخاب Hash Function مناسب برای نوع‌های سفارشی اهمیت زیادی دارد.

## پیچیدگی زمانی و رفتار عملکردی

### پیچیدگی عملیات اصلی

جدول زیر پیچیدگی‌های معمول و تضمین‌های مهم استاندارد را خلاصه می‌کند:

| عملیات | پیچیدگی متوسط | پیچیدگی بدترین حالت |
|---|---:|---:|
| `find` | \(O(1)\) | \(O(n)\) |
| `insert` | \(O(1)\) | \(O(n)\) |
| `erase(key)` | \(O(1)\) | \(O(n)\) |
| `count` | \(O(1)\) | \(O(n)\) |
| `size` | \(O(1)\) | \(O(1)\) |
| `clear` | \(O(n)\) | \(O(n)\) |

این جدول برای عملیات متداول Container ارائه شده است. در عملیات‌هایی که تعداد عناصر ورودی، محدوده حذف یا تعداد عناصر درج‌شده متفاوت است، پیچیدگی دقیق به تعداد عناصر پردازش‌شده نیز وابسته خواهد بود.

### چرا پیچیدگی متوسط مهم است؟

پیچیدگی متوسط \(O(1)\) به این معنا نیست که هر عملیات دقیقاً در زمان ثابت اجرا می‌شود. این نتیجه به توزیع مناسب Hashها و شرایط معمول استفاده وابسته است.

در برخی ورودی‌ها، Collisionهای زیاد یا Rehash می‌توانند هزینه عملیات را افزایش دهند.

بنابراین، در سامانه‌های حساس به زمان پاسخ، باید توزیع داده‌ها، الگوی جست‌وجو و رفتار حافظه را با Benchmark واقعی بررسی کرد.

### هزینه حافظه

در مقایسه با برخی Containerهای دیگر، `std::unordered_set` معمولاً به حافظه اضافی برای Bucketها و ساختارهای داخلی Hash Table نیاز دارد.

این هزینه می‌تواند برای مجموعه‌های بزرگ یا مجموعه‌هایی که عناصر کوچک دارند، قابل‌توجه باشد.

اگر حافظه محدود است یا عناصر بسیار کوچک و پرتعداد هستند، مقایسه با `std::set`، `std::vector` مرتب‌شده یا ساختارهای اختصاصی می‌تواند مفید باشد.

## مدیریت ظرفیت و Rehash

### مفهوم Load Factor

Load Factor نسبت تعداد عناصر به تعداد Bucketها است:

\[
\text{load factor} =
\frac{\text{size}}{\text{bucket count}}
\]

تابع `load_factor` مقدار فعلی این نسبت را برمی‌گرداند.

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values;

    for (int i = 0; i < 1000; ++i) {
        values.insert(i);
    }

    std::cout << values.load_factor() << '\n';
    std::cout << values.max_load_factor() << '\n';
}
```

Load Factor پایین‌تر می‌تواند احتمال برخوردهای زیاد را کاهش دهد، اما معمولاً با مصرف حافظه بیشتر همراه است.

### تنظیم max_load_factor

تابع `max_load_factor` برای مشاهده یا تنظیم حداکثر Load Factor مورد استفاده قرار می‌گیرد.

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values;

    values.max_load_factor(0.7f);

    for (int i = 0; i < 1000; ++i) {
        values.insert(i);
    }
}
```

مقدار `0.7f` در این مثال صرفاً یک انتخاب نمونه است و لزوماً برای تمام برنامه‌ها بهترین مقدار نیست.

پس از تغییر حداکثر Load Factor، ممکن است عملیات بعدی باعث Rehash شود. برای ارزیابی بهتر، این تنظیم را همراه با تعداد عناصر، مصرف حافظه و زمان عملیات اندازه‌گیری کنید.

### رزرو ظرفیت با reserve

اگر تعداد تقریبی عناصر از قبل مشخص است، استفاده از `reserve` می‌تواند تعداد دفعات Rehash را کاهش دهد:

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values;

    values.reserve(10000);

    for (int i = 0; i < 10000; ++i) {
        values.insert(i);
    }
}
```

تابع `reserve` ظرفیت Bucketها را بر اساس تعداد عناصر مورد انتظار و `max_load_factor` تنظیم می‌کند. این تابع تضمین نمی‌کند که Container دقیقاً به همان تعداد Bucket برسد.

برای برنامه‌هایی که تعداد زیادی عنصر را به‌صورت تدریجی وارد می‌کنند، رزرو ظرفیت می‌تواند به کاهش هزینه Rehash کمک کند.

### تنظیم مستقیم تعداد Bucketها با rehash

تابع `rehash` حداقل تعداد Bucket درخواستی را مشخص می‌کند:

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {1, 2, 3};

    values.rehash(100);
}
```

تابع `rehash` ممکن است تعداد Bucketهای بیشتری از مقدار درخواستی ایجاد کند. همچنین، اندازه نهایی به الزامات Load Factor و پیاده‌سازی Container وابسته است.

تفاوت مهم این است که `reserve` بر اساس تعداد عناصر مورد انتظار کار می‌کند، اما `rehash` مستقیماً با تعداد Bucketها سروکار دارد.

### اثر Rehash بر Iteratorها و Referenceها

در `std::unordered_set`، Rehash باعث نامعتبرشدن Iteratorهای موجود می‌شود. بنابراین، پس از Rehash نباید Iteratorهای قبلی را استفاده کنید.

بااین‌حال، Referenceها و Pointerهای اشاره‌کننده به عناصر صرفاً به دلیل Rehash نامعتبر نمی‌شوند. این تفاوت از نکات مهم کار با Hash Containerها است.

در عملیات درج نیز اگر Rehash رخ دهد، Iteratorهای قبلی ممکن است نامعتبر شوند. پس از درج عنصر، اگر به Iteratorهای قدیمی نیاز دارید، باید احتمال Rehash را در طراحی کد لحاظ کنید.

عملیات حذف نیز Iterator عنصر حذف‌شده را نامعتبر می‌کند، اما Iteratorهای مربوط به عناصر دیگر را صرفاً به دلیل همان حذف نامعتبر نمی‌کند.

## پیمایش عناصر و Iteratorها

### استفاده از Range-based for

ساده‌ترین روش پیمایش مجموعه، استفاده از حلقه Range-based است:

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    for (const auto& value : values) {
        std::cout << value << '\n';
    }
}
```

استفاده از `const auto&` از کپی‌شدن غیرضروری عناصر جلوگیری می‌کند.

از آنجا که `std::unordered_set` برای نگهداری کلیدهای یکتا طراحی شده است، نمی‌توانید از طریق Iterator مقدار عنصر را تغییر دهید. تغییر مستقیم کلید می‌تواند Hash و جایگاه منطقی عنصر را ناسازگار کند.

### پیمایش با Iterator

در مواردی که نیاز به کنترل دقیق‌تر دارید، می‌توانید از Iterator استفاده کنید:

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    for (auto it = values.begin(); it != values.end(); ++it) {
        std::cout << *it << '\n';
    }
}
```

توابع `begin` و `end` به‌ترتیب Iterator شروع و پایان محدوده را برمی‌گردانند.

نسخه‌های `cbegin` و `cend` نیز برای پیمایش فقط‌خواندنی در دسترس هستند.

### نبود ترتیب پیمایش تضمین‌شده

حتی اگر مجموعه‌ای با عناصر مرتب‌شده درج کنید، نمی‌توانید فرض کنید که پیمایش بر اساس همان ترتیب انجام می‌شود.

برای تولید خروجی قطعی، باید عناصر را در Container دیگری مانند `std::vector` کپی کنید و سپس مرتب‌سازی انجام دهید.

```cpp
#include <algorithm>
#include <iostream>
#include <unordered_set>
#include <vector>

int main() {
    std::unordered_set<int> values = {40, 10, 30, 20};

    std::vector<int> sortedValues(
        values.begin(),
        values.end()
    );

    std::sort(sortedValues.begin(), sortedValues.end());

    for (int value : sortedValues) {
        std::cout << value << ' ';
    }
}
```

این روش زمانی مفید است که ساختار اصلی باید جست‌وجوی سریع داشته باشد، اما خروجی نهایی به ترتیب مشخص نیاز دارد.

## Hash Function و تابع برابری سفارشی

### استفاده از Hash پیش‌فرض

کتابخانه استاندارد برای بسیاری از نوع‌های متداول، تخصصی‌سازی‌های مناسبی از `std::hash` فراهم می‌کند.

برای نمونه، `std::unordered_set<int>` به‌طور معمول بدون تعریف Hash سفارشی قابل استفاده است.

```cpp
std::unordered_set<int> identifiers;
std::unordered_set<std::string> names;
```

برای نوع‌های سفارشی، باید بررسی کنید که Hash Function و Equality موردنیاز Container در دسترس باشند.

### تعریف Hash برای نوع سفارشی

فرض کنید می‌خواهیم مجموعه‌ای از مختصات دوبعدی داشته باشیم:

```cpp
#include <cstddef>
#include <functional>
#include <unordered_set>

struct Point {
    int x;
    int y;

    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

struct PointHash {
    std::size_t operator()(const Point& point) const {
        const std::size_t hx = std::hash<int>{}(point.x);
        const std::size_t hy = std::hash<int>{}(point.y);

        return hx ^ (hy + 0x9e3779b9U + (hx << 6) + (hx >> 2));
    }
};

int main() {
    std::unordered_set<Point, PointHash> points;

    points.insert({1, 2});
    points.insert({3, 4});
    points.insert({1, 2});
}
```

در این مثال، دو نقطه با مختصات یکسان معادل شناخته می‌شوند و فقط یک عنصر در مجموعه باقی می‌ماند.

فرمول ترکیب Hash در این مثال یک نمونه ساده است و برای تمام کاربردها بهترین انتخاب نیست. در پروژه‌های حساس به عملکرد یا امنیت، باید کیفیت توزیع Hash و رفتار آن روی داده‌های واقعی بررسی شود.

### قرارداد مهم Hash و Equality

یکی از مهم‌ترین قواعد `std::unordered_set` این است:

اگر دو کلید طبق تابع برابری معادل باشند، Hash آن‌ها باید یکسان باشد.

به زبان ریاضی:

\[
\operatorname{KeyEqual}(a,b)=\text{true}
\Rightarrow
\operatorname{Hash}(a)=\operatorname{Hash}(b)
\]

اما برعکس این رابطه الزامی نیست؛ یعنی دو کلید با Hash یکسان لزوماً برابر نیستند.

اگر این قرارداد نقض شود، رفتار جست‌وجو و یکتایی مجموعه می‌تواند با انتظارات برنامه‌نویس ناسازگار شود.

### تعریف تابع برابری سفارشی

می‌توانید Hash و Equality را جداگانه به Container بدهید:

```cpp
#include <cstddef>
#include <unordered_set>

struct CaseInsensitiveHash {
    std::size_t operator()(int value) const {
        return std::hash<int>{}(value);
    }
};

struct CaseInsensitiveEqual {
    bool operator()(int lhs, int rhs) const {
        return lhs == rhs;
    }
};

int main() {
    std::unordered_set<
        int,
        CaseInsensitiveHash,
        CaseInsensitiveEqual
    > values;

    values.insert(10);
    values.insert(20);
}
```

نام این دو نوع صرفاً برای نمایش ساختار Template انتخاب شده است؛ در این مثال، عملیات روی اعداد صحیح و برابری عادی انجام می‌شود.

برای رشته‌های بدون حساسیت به بزرگی و کوچکی حروف، باید هم Hash و هم Equality را مطابق یک سیاست یکسان طراحی کنید. اگر Equality دو رشته را معادل بداند اما Hash آن‌ها متفاوت باشد، قرارداد Container نقض می‌شود.

همچنین، قواعد مربوط به Unicode، Normalization و Locale را نباید با تبدیل ساده ASCII به حروف کوچک یکسان دانست.

## تخصصی‌سازی std::hash برای نوع سفارشی

یکی از روش‌های استفاده از Hash سفارشی، تعریف Functor مجزا است که در بخش قبل مشاهده کردید. روش دیگر، تخصصی‌سازی `std::hash` برای نوع کاربرساخته است.

```cpp
#include <cstddef>
#include <functional>
#include <unordered_set>

struct UserId {
    int value;

    bool operator==(const UserId& other) const {
        return value == other.value;
    }
};

namespace std {
    template <>
    struct hash<UserId> {
        std::size_t operator()(const UserId& id) const {
            return std::hash<int>{}(id.value);
        }
    };
}

int main() {
    std::unordered_set<UserId> users;

    users.insert({1});
    users.insert({2});
}
```

تخصصی‌سازی `std::hash` برای نوع کاربرساخته، در صورت رعایت الزامات استاندارد، روشی معتبر است. بااین‌حال، افزودن تخصصی‌سازی‌های دلخواه برای نوع‌های استاندارد یا تغییر رفتار تعریف‌شده کتابخانه استاندارد مجاز نیست.

استفاده از Functor مجزا معمولاً انعطاف بیشتری ایجاد می‌کند، زیرا می‌توان چند سیاست Hash متفاوت برای یک نوع تعریف کرد.

## استفاده از ساختارهای پیچیده‌تر به‌عنوان کلید

### استفاده از std::pair

برای مجموعه‌ای از زوج‌ها، باید Hash مناسب تعریف کنید؛ زیرا در ++C11، تخصصی‌سازی عمومی استاندارد برای `std::hash<std::pair<...>>` ارائه نشده است.

```cpp
#include <cstddef>
#include <functional>
#include <unordered_set>
#include <utility>

struct PairHash {
    std::size_t operator()(
        const std::pair<int, int>& value
    ) const {
        const std::size_t h1 = std::hash<int>{}(value.first);
        const std::size_t h2 = std::hash<int>{}(value.second);

        return h1 ^ (h2 + 0x9e3779b9U + (h1 << 6) + (h1 >> 2));
    }
};

int main() {
    std::unordered_set<
        std::pair<int, int>,
        PairHash
    > edges;

    edges.insert({1, 2});
    edges.insert({2, 3});
    edges.insert({1, 2});
}
```

این روش برای نگهداری یال‌های یکتا در Graph یا زوج‌های مختصات کاربرد دارد.

### استفاده از ساختارهای چندفیلدی

اگر یک شیء چند فیلد دارد، باید مشخص کنید کدام فیلدها هویت منطقی کلید را تعیین می‌کنند.

برای مثال، اگر شناسه کاربر تنها معیار یکتایی است، نباید Hash بر اساس شناسه و Equality بر اساس شناسه به‌علاوه نام تعریف شود.

در طراحی کلیدهای مرکب، سازگاری Hash و Equality مهم‌تر از پیچیدگی فرمول Hash است.

## محدودیت تغییر عناصر و کلیدهای قابل‌تغییر

### چرا کلیدها فقط‌خواندنی هستند؟

در `std::unordered_set`، Iterator معمولی به عنصری از نوع `const value_type` اشاره می‌کند. دلیل این طراحی آن است که تغییر کلید ممکن است مقدار Hash را عوض کند و عنصر را در Bucket نامناسب قرار دهد.

برای نمونه، کد زیر معتبر نیست:

```cpp
std::unordered_set<int> values = {10, 20, 30};

auto it = values.find(20);

// *it = 25; // Invalid: elements are immutable through iterators.
```

اگر می‌خواهید کلید را تغییر دهید، باید آن را حذف کرده و نسخه جدید را درج کنید:

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto it = values.find(20);

    if (it != values.end()) {
        values.erase(it);
        values.insert(25);
    }
}
```

### استخراج و تغییر کلید با node handle در ++C17

در استاندارد ++C17، قابلیت Node Handle اضافه شد که امکان استخراج یک عنصر، تغییر کلید و درج مجدد آن را فراهم می‌کند.

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto node = values.extract(20);

    if (!node.empty()) {
        node.value() = 25;
        values.insert(std::move(node));
    }
}
```

این قابلیت در ++C11 وجود ندارد. بنابراین، برای پروژه‌های مبتنی بر ++C11 باید از روش حذف و درج مجدد استفاده کنید.

توجه کنید که اگر کلید جدید از قبل در مجموعه وجود داشته باشد، درج Node Handle ممکن است موفق نشود؛ در چنین حالتی باید نتیجه درج را بررسی کنید و در صورت نیاز Node باقی‌مانده را مدیریت کنید.

## مدیریت عمر اشیا و Iterator Invalidation

### عملیات‌هایی که Iteratorها را نامعتبر می‌کنند

رفتار Iteratorها در `std::unordered_set` باید با دقت در نظر گرفته شود.

| عملیات | اثر بر Iteratorها |
|---|---|
| درج عنصر بدون Rehash | Iteratorهای قبلی معتبر می‌مانند |
| درج عنصر همراه با Rehash | Iteratorهای قبلی نامعتبر می‌شوند |
| اجرای `rehash` | Iteratorهای قبلی نامعتبر می‌شوند |
| حذف یک عنصر | Iterator همان عنصر نامعتبر می‌شود |
| اجرای `clear` | Iteratorهای مربوط به عناصر قبلی نامعتبر می‌شوند |
| تغییر Hash یا Equality به‌صورت غیرمجاز در زمان استفاده | می‌تواند قراردادهای Container را نقض کند |

Referenceها و Pointerهای اشاره‌کننده به عناصر، در اثر Rehash صرفاً به دلیل جابه‌جایی داخلی Bucketها نامعتبر نمی‌شوند؛ اما حذف عنصر، عمر همان شیء را پایان می‌دهد و Referenceها و Pointerهای مربوط به آن را نامعتبر می‌کند.

برای جلوگیری از خطا، بهتر است Iterator را تنها تا زمانی نگه دارید که از عدم وقوع عملیات نامعتبرکننده مطمئن هستید.

### مثال نادرست با Iterator قدیمی

کد زیر می‌تواند مشکل‌ساز باشد:

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto it = values.begin();

    values.rehash(values.bucket_count() + 100);

    // Using the old iterator is invalid.
    // int value = *it;
}
```

پس از Rehash، باید Iterator جدیدی دریافت کنید.

## تفاوت clear، rehash و swap

### تفاوت پاک‌سازی عناصر با کاهش ظرفیت

تابع `clear` عناصر را حذف می‌کند، اما الزاماً Bucketها را آزاد نمی‌کند.

تابع `rehash` ساختار Bucketها را تغییر می‌دهد و ممکن است برای کاهش یا افزایش تعداد Bucketها استفاده شود؛ بااین‌حال، مقدار نهایی باید با محدودیت‌های Container سازگار باشد.

تابع `swap` دو Container را جابه‌جا می‌کند:

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> first = {1, 2, 3};
    std::unordered_set<int> second = {10, 20};

    first.swap(second);
}
```

پس از `swap`، عناصر دو مجموعه جابه‌جا شده‌اند.

در استفاده از `swap` باید الزامات مربوط به Allocatorها و رفتار استاندارد Container را در نظر بگیرید. همچنین، اگر Iterator یا Reference از قبل نگه داشته‌اید، باید قواعد اعتبار آن‌ها را مطابق عملیات انجام‌شده بررسی کنید.

## استفاده از equal_range

تابع `equal_range` برای کلید مشخص، محدوده‌ای از عناصر معادل را برمی‌گرداند.

در `std::unordered_set`، چون عناصر یکتا هستند، محدوده بازگشتی شامل صفر یا یک عنصر است.

```cpp
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30};

    auto range = values.equal_range(20);

    if (range.first != range.second) {
        int value = *range.first;
    }
}
```

این تابع در کاربردهای معمول `std::unordered_set` کمتر از `find` استفاده می‌شود، اما برای کدهای عمومی‌تر که با انواع مختلف Associative Container کار می‌کنند، می‌تواند مفید باشد.

## بررسی Bucketها و ابزارهای تشخیصی

### مشاهده Bucket مربوط به یک عنصر

تابع `bucket` شماره Bucket مرتبط با یک کلید را برمی‌گرداند:

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {10, 20, 30, 40};

    for (int value : values) {
        std::cout << value << ": "
                  << values.bucket(value) << '\n';
    }
}
```

شماره Bucketها جزئیات پیاده‌سازی هستند و نباید به‌عنوان بخشی از منطق تجاری برنامه در نظر گرفته شوند.

### شمارش عناصر هر Bucket

تابع `bucket_size` تعداد عناصر یک Bucket را برمی‌گرداند:

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> values = {
        10, 20, 30, 40, 50
    };

    for (std::size_t i = 0; i < values.bucket_count(); ++i) {
        std::cout << "Bucket " << i << ": "
                  << values.bucket_size(i) << '\n';
    }
}
```

این ابزارها برای تشخیص توزیع نامناسب Hash در آزمایش‌های عملکردی مفید هستند.

بااین‌حال، وابستگی منطق برنامه به شماره Bucketها یا ترتیب عناصر داخل Bucketها توصیه نمی‌شود.

## نکات امنیتی در طراحی Hash Function

### Hash Function ضعیف و حملات ورودی

در سرویس‌هایی که ورودی از کاربران دریافت می‌کنند، Hash Function ضعیف یا قابل‌پیش‌بینی می‌تواند خطر افزایش Collisionها و افت شدید عملکرد را ایجاد کند.

در برخی شرایط، مهاجم می‌تواند ورودی‌هایی بسازد که تعداد زیادی از آن‌ها در Bucketهای محدودی قرار بگیرند. در نتیجه، عملیات‌هایی که معمولاً سریع هستند ممکن است به زمان خطی نزدیک شوند.

برای کاهش ریسک:

- از Hash Function مناسب برای نوع کلید استفاده کنید.
- ورودی‌های غیرقابل‌اعتماد را از نظر حجم و تعداد محدود کنید.
- برای داده‌های حساس به امنیت، از راهکار Hash مناسب با مدل تهدید استفاده کنید.
- رفتار Container را با داده‌های خصمانه یا بسیار نامتوازن آزمایش کنید.
- فرض نکنید Hash Function پیش‌فرض تمام کتابخانه‌های استاندارد در برابر حملات Hash Flooding مقاوم است.

در صورت نیاز به Hash مقاوم در برابر حملات، انتخاب الگوریتم باید بر اساس الزامات امنیتی، پشتیبانی از کلید تصادفی و مدل تهدید انجام شود.

## استفاده از std::unordered_set برای حذف مقادیر تکراری

یکی از کاربردهای رایج این Container، یکتا‌سازی داده‌ها است.

```cpp
#include <iostream>
#include <unordered_set>
#include <vector>

int main() {
    std::vector<int> input = {
        5, 2, 5, 3, 2, 7, 3, 8
    };

    std::unordered_set<int> unique(
        input.begin(),
        input.end()
    );

    std::cout << unique.size() << '\n';
}
```

این روش برای حذف تکرارها زمانی مفید است که ترتیب عناصر اهمیت نداشته باشد.

اگر حفظ ترتیب اولین رخداد هر عنصر لازم باشد، ساختن مستقیم `unordered_set` کافی نیست. در آن شرایط می‌توان از یک مجموعه برای بررسی عضویت و یک `std::vector` برای نگهداری ترتیب خروجی استفاده کرد.

```cpp
#include <iostream>
#include <unordered_set>
#include <vector>

int main() {
    std::vector<int> input = {
        5, 2, 5, 3, 2, 7, 3, 8
    };

    std::unordered_set<int> seen;
    std::vector<int> uniqueInOrder;

    for (int value : input) {
        if (seen.insert(value).second) {
            uniqueInOrder.push_back(value);
        }
    }

    for (int value : uniqueInOrder) {
        std::cout << value << ' ';
    }
}
```

در این روش، ترتیب اولین رخداد هر مقدار حفظ می‌شود و عملیات بررسی عضویت به‌طور متوسط زمان ثابت دارد.

## کاربردهای عملی در پروژه‌های واقعی

### جلوگیری از پردازش دوباره شناسه‌ها

در پردازش فایل‌های بزرگ، پیام‌ها یا درخواست‌های شبکه، ممکن است لازم باشد هر شناسه فقط یک بار پردازش شود.

با نگهداری شناسه‌های مشاهده‌شده در `std::unordered_set` می‌توان پیش از پردازش، وجود شناسه را بررسی کرد.

### تشخیص کلمات یکتا

برای تحلیل متن، شمارش تعداد کلمات متفاوت یا بررسی حضور یک واژه می‌توان از مجموعه رشته‌ها استفاده کرد.

البته در پردازش متن واقعی، باید سیاست مربوط به حروف بزرگ و کوچک، Unicode، فاصله‌ها و Normalization نیز مشخص باشد.

### پیاده‌سازی مجموعه‌ای از مجوزها

در سامانه‌های مجوزدهی، مجموعه‌ای از قابلیت‌ها یا شناسه‌های مجاز را می‌توان نگهداری کرد. بررسی عضویت در این مجموعه معمولاً ساده و سریع است.

بااین‌حال، در طراحی امنیتی نباید صرفاً به عملکرد Container تکیه کرد؛ سیاست‌های اعتبارسنجی، هم‌زمانی و به‌روزرسانی داده‌ها نیز اهمیت دارند.

### جلوگیری از بازدید مجدد گره‌ها در Graph

در الگوریتم‌های پیمایش Graph مانند BFS یا DFS، مجموعه‌ای از شناسه گره‌های بازدیدشده می‌تواند از پردازش دوباره گره‌ها جلوگیری کند.

در این کاربرد، `std::unordered_set` به‌ویژه زمانی مفید است که شناسه گره‌ها پراکنده یا غیرمتوالی باشند.

## مقایسه با std::vector و ساختارهای جایگزین

### مقایسه با std::vector

برای بررسی وجود یک مقدار در `std::vector` معمولاً به جست‌وجوی خطی نیاز است، مگر اینکه ساختار داده مرتب شده باشد یا از روش دیگری استفاده شود.

در مقابل، `std::unordered_set` برای جست‌وجوی عضویت با عملکرد متوسط ثابت طراحی شده است.

بااین‌حال، برای مجموعه‌های کوچک، `std::vector` ممکن است سریع‌تر باشد؛ زیرا ساختار حافظه پیوسته دارد و از Locality مناسب Cache بهره می‌برد.

بنابراین، انتخاب بین این دو ساختار باید بر اساس اندازه داده‌ها، تعداد جست‌وجوها، هزینه درج و حذف و نتایج Benchmark انجام شود.

### مقایسه با std::unordered_map

اگر به ازای هر کلید باید یک مقدار جداگانه نگهداری شود، `std::unordered_map` انتخاب مناسب‌تری است.

در مقابل، `std::unordered_set` تنها کلیدها را نگهداری می‌کند و برای بررسی عضویت یا ذخیره مقادیر یکتا مناسب است.

برای مثال، اگر فقط شناسه کاربران اهمیت دارد، مجموعه کافی است؛ اما اگر به هر شناسه باید نام یا وضعیت متصل شود، Map معمولاً مناسب‌تر خواهد بود.

### مقایسه با std::unordered_multiset

Container دیگری به نام `std::unordered_multiset` امکان نگهداری چند عنصر معادل را فراهم می‌کند.

اگر تکرار عناصر باید حفظ شود، `unordered_multiset` مناسب است؛ اما اگر هر مقدار باید تنها یک بار وجود داشته باشد، `unordered_set` انتخاب درست‌تری است.

## تفاوت نسخه‌های استاندارد ++C

### قابلیت‌های موجود در ++C11

نوع `std::unordered_set` در ++C11 معرفی شد. قابلیت‌های اصلی آن شامل درج، حذف، جست‌وجو، Iteratorها، Hash سفارشی، مدیریت Bucketها و تنظیم Load Factor است.

### قابلیت‌های اضافه‌شده در ++C17

استاندارد ++C17 قابلیت‌هایی مانند Node Handle و عملیات انتقال گره بین Containerهای سازگار را اضافه کرد.

این امکانات برای استخراج یک عنصر، تغییر کلید و درج مجدد آن کاربرد دارند.

### قابلیت‌های اضافه‌شده در ++C20

استاندارد ++C20 تابع `contains` را برای بررسی عضویت اضافه کرد. همچنین، جست‌وجوی Heterogeneous برای عملیات اصلی در صورت استفاده از Hash و Equality شفاف پشتیبانی می‌شود.

برای نمونه، با طراحی مناسب `std::unordered_set<std::string>` می‌توان در محیط‌های سازگار، بدون ساخت رشته موقت با `std::string_view` جست‌وجو کرد.

```cpp
#include <functional>
#include <string>
#include <string_view>
#include <unordered_set>

struct TransparentHash {
    using is_transparent = void;

    std::size_t operator()(std::string_view value) const {
        return std::hash<std::string_view>{}(value);
    }
};

struct TransparentEqual {
    using is_transparent = void;

    bool operator()(
        std::string_view lhs,
        std::string_view rhs
    ) const {
        return lhs == rhs;
    }
};

int main() {
    std::unordered_set<
        std::string,
        TransparentHash,
        TransparentEqual
    > words = {"apple", "banana"};

    std::string_view query = "apple";

    auto it = words.find(query);
}
```

این مثال به ++C20 نیاز دارد. وجود `is_transparent` به‌تنهایی کافی نیست؛ Hash و Equality باید برای نوع‌های ورودی موردنیاز نیز قابل فراخوانی باشند و قرارداد سازگاری آن‌ها حفظ شود.

## خطاهای رایج و روش جلوگیری از آن‌ها

### فرض‌کردن ترتیب ثابت عناصر

نباید برای منطق برنامه یا تست‌های قطعی، به ترتیب پیمایش `std::unordered_set` وابسته شوید.

اگر ترتیب مشخصی لازم است، از Container مرتب‌شده یا مرحله مرتب‌سازی خروجی استفاده کنید.

### استفاده از Iterator پس از Rehash

هر عملیاتی که احتمال Rehash داشته باشد باید با توجه به Iteratorهای نگهداری‌شده بررسی شود.

در صورت Rehash، Iteratorهای قبلی نامعتبر می‌شوند و باید دوباره از Container Iterator دریافت کنید.

### تعریف Hash ناسازگار با Equality

اگر دو کلید از نظر Equality معادل باشند، Hash آن‌ها باید یکسان باشد.

نقض این قرارداد می‌تواند باعث شود رفتار مجموعه با انتظار منطقی شما سازگار نباشد.

### تغییر وضعیت داخلی کلید

اگر Hash یا Equality به داده‌های قابل‌تغییر خارجی وابسته باشند، تغییر آن داده‌ها در زمان حضور کلید در مجموعه می‌تواند قواعد Container را نقض کند.

بهتر است Hash و Equality بر اساس ویژگی‌های پایدار کلید طراحی شوند.

### فرض‌کردن پیچیدگی ثابت در تمام شرایط

پیچیدگی متوسط \(O(1)\) تضمین زمان ثابت برای تک‌تک عملیات نیست. Collisionهای زیاد یا Rehash می‌توانند هزینه عملیات را افزایش دهند.

### استفاده از unordered_set برای داده‌های مرتب

اگر الگوریتم به کوچک‌ترین عنصر، بزرگ‌ترین عنصر یا بازه مرتب نیاز دارد، `std::set` یا یک Container مرتب‌شده ممکن است انتخاب مناسب‌تری باشد.

### استفاده غیرضروری از emplace

تابع `emplace` همیشه سریع‌تر از `insert` نیست. اگر عنصر از قبل آماده است یا ساخت آن هزینه خاصی ندارد، `insert` ممکن است ساده‌تر و به همان اندازه کارآمد باشد.

## نکات مهم برای طراحی و نگهداری کد حرفه‌ای

- برای کلیدهای سفارشی، Hash و Equality را به‌صورت یک جفت سازگار طراحی کنید.
- اگر تعداد عناصر از قبل مشخص است، استفاده از `reserve` را بررسی کنید.
- به رفتار Iteratorها در عملیات درج، حذف و Rehash توجه کنید.
- از اتکا به ترتیب پیمایش یا شماره Bucketها خودداری کنید.
- در برنامه‌های حساس به عملکرد، مصرف حافظه و کیفیت توزیع Hash را اندازه‌گیری کنید.
- برای ورودی‌های غیرقابل‌اعتماد، محدودیت‌های منابع و ریسک Hash Flooding را در نظر بگیرید.
- در پروژه‌هایی که باید با ++C11 سازگار باشند، از APIهای جدیدتر مانند `contains` یا Node Handle استفاده نکنید.
- اگر کلیدها باید تغییر کنند، از روش حذف و درج مجدد در ++C11 یا Node Handle در ++C17 استفاده کنید.
- در پروژه‌های چندریسمانی، دسترسی هم‌زمان به Container را مطابق قواعد Thread Safety کتابخانه استاندارد مدیریت کنید؛ عملیات تغییردهنده را نباید بدون همگام‌سازی مناسب هم‌زمان اجرا کرد.
- برای تصمیم‌گیری میان `unordered_set`، `set` و `vector`، Benchmark را با داده‌ها و الگوی دسترسی واقعی انجام دهید.

## جمع‌بندی

ساختار `std::unordered_set` در ++C11 ابزاری مناسب برای نگهداری عناصر یکتا و انجام عملیات سریع جست‌وجو، درج و حذف است. این Container با استفاده از Hash Table کار می‌کند و در شرایط معمول، پیچیدگی زمانی متوسط \(O(1)\) دارد؛ اما ترتیب عناصر را تضمین نمی‌کند و در بدترین حالت ممکن است عملکرد خطی داشته باشد.

انتخاب درست این ساختار به نیاز برنامه بستگی دارد. اگر ترتیب عناصر اهمیت ندارد و بررسی عضویت پرتکرار است، `std::unordered_set` می‌تواند انتخاب بسیار مناسبی باشد. در مقابل، برای داده‌های مرتب، دسترسی بازه‌ای یا الزام به پیچیدگی لگاریتمی در بدترین حالت، `std::set` گزینه مناسب‌تری خواهد بود.

در نهایت، تسلط بر Hash Function، قرارداد Equality، مدیریت ظرفیت، Rehash، اعتبار Iteratorها و تفاوت APIهای استانداردهای مختلف، برای استفاده قابل‌اعتماد و حرفه‌ای از این Container ضروری است.

---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>