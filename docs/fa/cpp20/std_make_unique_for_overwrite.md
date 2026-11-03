<div align="center">

[🇺🇸 English](../../en/cpp20/std_make_unique_for_overwrite.md) | [🇮🇷 فارسی](./std_make_unique_for_overwrite.md)

</div>

---

# آشنایی با `std::make_unique_for_overwrite` در ++C20

## مقدمه

قابلیت `std::make_unique_for_overwrite` از ++C20 به کتابخانه استاندارد اضافه شده است و راهی برای ساخت یک شیء یا آرایه با مالکیت انحصاری در قالب `std::unique_ptr` فراهم می‌کند؛ با این تفاوت مهم که شیء ساخته‌شده به‌جای `value-initialization`، با `default-initialization` مقداردهی اولیه می‌شود.

این تفاوت در ظاهر کوچک، در عمل می‌تواند برای بافرهایی که قرار است بلافاصله توسط یک عملیات ورودی یا محاسباتی کاملاً بازنویسی شوند مهم باشد. برای نمونه، هنگام دریافت داده از فایل، شبکه یا یک API سطح پایین، ممکن است نیازی به صفر کردن تمام عناصر یک آرایه پیش از نوشتن داده‌ها وجود نداشته باشد.

بااین‌حال، این تابع ابزار «ساخت حافظه بدون مقداردهی» به معنای خام نیست. این تابع همچنان یک شیء یا آرایه از نوع مشخص ایجاد می‌کند، طول عمر آن را آغاز می‌کند و مالکیت آن را به `std::unique_ptr` می‌سپارد. درک دقیق قواعد مقداردهی اولیه برای استفاده ایمن از آن ضروری است.

## معرفی و امضای تابع

این تابع در هدر `<memory>` تعریف شده و از ++C20 در دسترس است. دو overload اصلی دارد:

```cpp
template<class T>
std::unique_ptr<T> make_unique_for_overwrite();

template<class T>
std::unique_ptr<T> make_unique_for_overwrite(std::size_t size);
```

- overload بدون آرگومان فقط برای نوع‌های غیرآرایه‌ای قابل استفاده است.
- overload دارای آرگومان `size` فقط برای آرایه‌های با bound نامشخص، مانند `T[]`، قابل استفاده است.
- overload متناظر برای آرایه‌های با bound مشخص، مانند `T[10]`، عمداً حذف‌شده (deleted) است.

در هر دو حالت، نوع بازگشتی یک `std::unique_ptr` است؛ بنابراین مدیریت طول عمر و آزادسازی منبع به‌صورت خودکار انجام می‌شود.

## مفهوم Default-initialization

برای فهم رفتار تابع باید میان سه مفهوم `default-initialization`، `value-initialization` و `zero-initialization` تمایز قائل شد. این‌ها قواعد زبان ++C هستند، نه قابلیت‌های اختصاصی `make_unique_for_overwrite`.

### تفاوت مقداردهی اولیه برای نوع‌های scalar

در مقداردهی از نوع `default-initialization`، برای نوع‌های scalar مانند `int`، `double`، اشاره‌گرها و `bool` مقدار اولیه‌ای نوشته نمی‌شود. در نتیجه، مقدار قابل استفاده‌ای برای خواندن در اختیار برنامه قرار ندارد.

```cpp
int* values = new int[4];

values[0] = 10;
values[1] = 20;
values[2] = 30;
values[3] = 40;
```

در مثال بالا عناصر آرایه پیش از انتساب، مقداردهی نشده‌اند. خواندن آن‌ها پیش از نوشتن مقدار معتبر، رفتار مجاز و قابل اتکایی نیست.

در مقابل، `value-initialization` برای چنین نوع‌هایی مقدار صفر تولید می‌کند:

```cpp
int* values = new int[4]();

```

بنابراین، تفاوت اصلی در آرایه‌ای از `int` این است که `new int[n]` عناصر را صفر نمی‌کند، اما `new int[n]()` آن‌ها را صفر می‌کند.

### رفتار برای نوع‌های class

برای نوع‌های class، `default-initialization` سازنده پیش‌فرض را اجرا می‌کند؛ اگر سازنده پیش‌فرض به‌طور ضمنی تعریف شده باشد، قواعد مقداردهی اعضا نیز تعیین می‌کنند اعضای شیء چه وضعیتی داشته باشند.

```cpp
#include <memory>

struct Record {
    int id;
    Record() : id(42) {}
};

int main() {
    auto record = std::make_unique_for_overwrite<Record>();
    return record->id;
}
```

در این مثال، سازنده `Record` اجرا می‌شود و `id` برابر ۴۲ خواهد بود. پس نام `for_overwrite` به این معنا نیست که سازنده کلاس نادیده گرفته می‌شود.

اما اگر سازنده پیش‌فرض اعضای scalar را مقداردهی نکند، آن اعضا نیز مقدار مشخصی نخواهند داشت:

```cpp
struct Record {
    int id;
    double score;
};

int main() {
    auto record = std::make_unique_for_overwrite<Record>();
    record->id = 7;
    record->score = 98.5;
}
```

در این مثال باید پیش از خواندن `id` یا `score`، مقدار آن‌ها نوشته شود.

### یک نکته درباره const

نوع‌های scalar از نوع `const` را نمی‌توان به این شکل default-initialize کرد؛ در نتیجه ساخت نمونه‌ای مانند `std::make_unique_for_overwrite<const int>()` معتبر نیست. برای شیئی که باید `const` باشد، باید سازوکاری انتخاب شود که مقدار آن را هنگام ساخت فراهم کند.

## مقایسه با std::make_unique

تابع `std::make_unique` از ++C14 در دسترس است. برای نوع غیرآرایه‌ای، ساخت شیء را با expression معادل `new T(args...)` انجام می‌دهد. در حالت بدون آرگومان، این شیء با `value-initialization` ساخته می‌شود.

در مقابل، `std::make_unique_for_overwrite` از expression معادل `new T` استفاده می‌کند و در نتیجه `default-initialization` انجام می‌شود.

برای آرایه‌ها نیز تفاوت مشابه است:

```cpp
#include <cstddef>
#include <memory>

int main() {
    constexpr std::size_t count = 1024;

    auto initialized = std::make_unique<int[]>(count);
    auto for_overwrite = std::make_unique_for_overwrite<int[]>(count);

    // initialized elements are zero-initialized
    // for_overwrite elements are not initialized
}
```

در نمونه بالا، عناصر `initialized` صفر هستند؛ اما عناصر `for_overwrite` تا پیش از نوشتن مقدار، قابل خواندن نیستند.

| ویژگی | `std::make_unique<T>()` | `std::make_unique_for_overwrite<T>()` |
|---|---|---|
| نوع مقداردهی برای شیء بدون آرگومان | `value-initialization` | `default-initialization` |
| نوع مقداردهی عناصر آرایه | `value-initialization` | `default-initialization` |
| مقدار اولیه نوع scalar | صفر در حالت value-initialization | مقدار مشخصی تضمین نمی‌شود |
| اجرای سازنده پیش‌فرض class | بله | بله |
| مناسب برای بافرهایی که فوراً بازنویسی می‌شوند | ممکن است هزینه مقداردهی غیرضروری داشته باشد | معمولاً انتخاب مناسب‌تری است |

این جدول درباره overloadهای بدون آرگومان و آرایه‌ای است. اگر برای `std::make_unique<T>(args...)` آرگومان سازنده ارسال شود، همان آرگومان‌ها برای ساخت شیء استفاده می‌شوند و مقایسه دیگر صرفاً مقایسه دو نوع مقداردهی بدون آرگومان نیست.

## نحوه ساخت شیء غیرآرایه‌ای

برای یک نوع غیرآرایه‌ای، نوع `T` را مشخص می‌کنیم و تابع یک شیء از همان نوع می‌سازد:

```cpp
#include <memory>

struct ConnectionState {
    int status;
    ConnectionState() : status(1) {}
};

int main() {
    auto state =
        std::make_unique_for_overwrite<ConnectionState>();

    return state->status;
}
```

تابع برای ساخت شیء از سازنده پیش‌فرض استفاده می‌کند. اگر این سازنده وجود نداشته باشد یا قابل دسترسی نباشد، فراخوانی نیز معتبر نخواهد بود.

این overload آرگومان سازنده دریافت نمی‌کند. اگر لازم است هنگام ساخت، آرگومان‌هایی به سازنده ارسال شوند، باید از `std::make_unique<T>(args...)` استفاده کرد.

## نحوه ساخت آرایه پویا

برای ساخت آرایه‌ای با اندازه زمان اجرا، نوع آرایه را با bound نامشخص می‌نویسیم و اندازه را به تابع می‌دهیم:

```cpp
#include <cstddef>
#include <memory>

int main() {
    const std::size_t count = 4096;

    auto buffer =
        std::make_unique_for_overwrite<std::byte[]>(count);

    // The buffer is ready to receive bytes.
}
```

نوع `buffer` در این مثال `std::unique_ptr<std::byte[]>` است. حافظه برای `count` عنصر تخصیص داده می‌شود و عناصر با `default-initialization` ساخته می‌شوند.

برای نوع‌های scalar مانند `std::byte`، این کار مقدار اولیه صفر ایجاد نمی‌کند. بنابراین باید قبل از خواندن هر عنصر، آن را با داده معتبر پر کرد.

### آرایه‌های چندبعدی

این الگو برای آرایه‌های چندبعدی با اولین بُعد نامشخص نیز قابل استفاده است:

```cpp
#include <memory>

int main() {
    constexpr std::size_t rows = 100;

    auto matrix =
        std::make_unique_for_overwrite<int[][3]>(rows);

    matrix[0][0] = 1;
    matrix[0][1] = 2;
    matrix[0][2] = 3;
}
```

در این مثال، تعداد سطرها در زمان اجرا تعیین می‌شود و هر سطر سه عنصر `int` دارد. مانند آرایه یک‌بعدی، عناصر scalar به‌صورت خودکار صفر نمی‌شوند.

## کاربردهای عملی

### دریافت مستقیم داده از ورودی

یکی از کاربردهای مناسب، ساخت بافری است که قرار است یک API ورودی تمام یا بخشی از آن را پر کند:

```cpp
#include <cstddef>
#include <memory>
#include <istream>

bool read_data(std::istream& input,
               std::size_t count,
               std::unique_ptr<char[]>& destination) {
    auto buffer =
        std::make_unique_for_overwrite<char[]>(count);

    input.read(buffer.get(),
               static_cast<std::streamsize>(count));

    if (!input) {
        return false;
    }

    destination = std::move(buffer);
    return true;
}
```

در این الگو، بافر ابتدا به‌صورت محلی ساخته می‌شود و فقط پس از موفقیت عملیات ورودی به خروجی منتقل می‌شود. اگر خواندن شکست بخورد، `unique_ptr` محلی با خروج از scope حافظه را آزاد می‌کند.

این نمونه فرض می‌کند اندازه ورودی در محدوده قابل نمایش با `std::streamsize` است و جریان دقیقاً به تعداد مورد انتظار بایت می‌خواند. در کد production باید تبدیل اندازه و سیاست مدیریت خواندن ناقص نیز به‌صورت صریح بررسی شود.

### آماده‌سازی بافر برای APIهای C

گاهی یک API مبتنی بر C یک pointer و ظرفیت بافر دریافت می‌کند و خودش داده را در آن می‌نویسد:

```cpp
#include <cstddef>
#include <memory>

int external_api(char* output, std::size_t capacity);

int main() {
    constexpr std::size_t capacity = 2048;

    auto buffer =
        std::make_unique_for_overwrite<char[]>(capacity);

    const int written = external_api(buffer.get(), capacity);

    if (written >= 0) {
        // Process only the number of bytes reported by the API.
    }
}
```

در این سناریو، صفر کردن اولیه بافر ممکن است غیرضروری باشد؛ البته فقط زمانی که قرارداد API تضمین کند بخش مورد استفاده را پیش از خواندن مقداردهی می‌کند.

### ساخت بافرهای بزرگ

برای بافرهای بزرگ، حذف مقداردهی اولیه می‌تواند از نوشتن غیرضروری روی حافظه جلوگیری کند. این موضوع ممکن است در مسیرهای پردازش داده، ورودی/خروجی و پردازش تصویر مفید باشد.

بااین‌حال، نباید فرض کرد که این تابع همیشه سریع‌تر است. نتیجه به نوع داده، allocator، کتابخانه استاندارد، سیستم‌عامل، اندازه تخصیص و الگوی دسترسی حافظه وابسته است. در صورت اهمیت کارایی، باید با benchmark واقعی اندازه‌گیری شود.

## نکات ایمنی و خطاهای رایج

### خواندن پیش از نوشتن

مهم‌ترین خطر، خواندن عنصری است که هنوز مقدار معتبر دریافت نکرده است:

```cpp
#include <memory>

int main() {
    auto values =
        std::make_unique_for_overwrite<int[]>(10);

    int first = values[0];
}
```

در این مثال، `values[0]` پیش از مقداردهی خوانده می‌شود. این کار مجاز نیست و در نسخه‌های مختلف استاندارد نیز نباید به رفتار یا مقدار خاصی برای آن تکیه کرد.

الگوی صحیح این است که ابتدا تمام عناصر موردنیاز مقداردهی شوند:

```cpp
#include <memory>

int main() {
    auto values =
        std::make_unique_for_overwrite<int[]>(10);

    for (int i = 0; i < 10; ++i) {
        values[i] = i * 2;
    }

    const int first = values[0];
}
```

### مقداردهی ناقص بافر

اگر یک عملیات ورودی فقط بخشی از بافر را پر کند، فقط همان بخش را می‌توان پردازش کرد. ظرفیت تخصیص‌یافته با تعداد بایت‌های واقعاً نوشته‌شده یکسان نیست.

برای مثال، اگر ظرفیت بافر ۱۰۰۰ بایت باشد اما API فقط ۳۰۰ بایت بنویسد، خواندن ۷۰۰ بایت باقی‌مانده معتبر نیست. بهتر است تعداد واقعی عناصر نوشته‌شده جداگانه نگهداری شود.

### اشتباه گرفتن default-initialization با مقدار پیش‌فرض

نام `for_overwrite` ممکن است این تصور را ایجاد کند که تمام داده‌ها به شکلی امن با مقدار پیش‌فرض پر می‌شوند. چنین برداشتی درست نیست. برای نوع‌های scalar، مقدار اولیه تضمین‌شده‌ای وجود ندارد؛ و برای classها نیز رفتار به سازنده پیش‌فرض و قواعد مقداردهی اعضا وابسته است.

### فرض صفر بودن حافظه تازه تخصیص‌یافته

نباید به این دلیل که یک سیستم‌عامل یا allocator در یک اجرای خاص صفحات حافظه را صفر تحویل داده است، روی صفر بودن عناصر حساب کرد. رفتار قابل اتکا باید از قواعد زبان و کتابخانه استاندارد نتیجه شود، نه از مشاهده اتفاقی یک محیط اجرایی.

### انتخاب نوع آرایه اشتباه

برای آرایه پویا باید نوعی با bound نامشخص استفاده شود:

```cpp
auto valid =
    std::make_unique_for_overwrite<int[]>(100);
```

اما نوع آرایه با bound ثابت برای این تابع پشتیبانی نمی‌شود:

```cpp
// This call is intentionally invalid.
auto invalid =
    std::make_unique_for_overwrite<int[100]>();
```

برای آرایه با اندازه ثابت، می‌توان از `std::array` یا یک شیء مناسب دیگر استفاده کرد.

## محدودیت‌ها و جزئیات استاندارد

### محدودیت overloadها

تابع فقط دو شکل اصلی دارد: ساخت یک شیء غیرآرایه‌ای بدون آرگومان، یا ساخت آرایه‌ای با bound نامشخص با دریافت اندازه. برای ساخت شیء با آرگومان‌های سازنده، `std::make_unique` انتخاب مناسب است.

همچنین ساخت آرایه‌ای با bound مشخص از طریق overload حذف‌شده ممنوع شده است. این طراحی با الگوی `std::make_unique` هم‌راستا است و از برخی خطاهای استفاده از آرایه جلوگیری می‌کند.

### مدیریت استثناها و مالکیت

اگر تخصیص حافظه شکست بخورد، ممکن است `std::bad_alloc` پرتاب شود. در ساخت نوع class نیز اگر سازنده استثنا پرتاب کند، عملیات ساخت کامل نمی‌شود و منبع تخصیص‌یافته باید طبق قواعد `new-expression` آزاد شود.

پس از موفقیت، شیء تحت مالکیت انحصاری `std::unique_ptr` قرار دارد. با خروج pointer از scope، حافظه آزاد می‌شود؛ مگر اینکه مالکیت با عملیاتی مانند `std::move` به pointer دیگری منتقل شده باشد.

```cpp
#include <memory>

int main() {
    auto first =
        std::make_unique_for_overwrite<int[]>(256);

    auto second = std::move(first);

    // first is empty; second owns the array.
}
```

### تفاوت با تخصیص خام حافظه

این تابع جایگزین عمومی `operator new`، `std::malloc` یا ابزارهای مدیریت حافظه خام نیست. `make_unique_for_overwrite` یک object یا array با نوع مشخص ایجاد می‌کند و قواعد ساخت و تخریب همان نوع را رعایت می‌کند.

اگر مسئله به تخصیص storage خام، کنترل دقیق alignment، مدیریت دستی lifetime یا استفاده از حافظه از پیش تخصیص‌یافته مربوط باشد، ابزارهایی مانند `std::allocator`، `std::construct_at` و امکانات مرتبط با lifetime ممکن است مناسب‌تر باشند. انتخاب آن‌ها نیازمند بررسی دقیق قواعد object lifetime است.

### تفاوت با std::make_shared_for_overwrite

از ++C20 تابع‌های `std::make_shared_for_overwrite` و `std::allocate_shared_for_overwrite` نیز برای ساخت اشیاء با مالکیت اشتراکی و `default-initialization` ارائه شده‌اند.

تفاوت اصلی در مدل مالکیت است:

- `std::make_unique_for_overwrite` مالکیت انحصاری را با `std::unique_ptr` فراهم می‌کند.
- `std::make_shared_for_overwrite` مالکیت اشتراکی را با `std::shared_ptr` فراهم می‌کند.
- `std::allocate_shared_for_overwrite` امکان استفاده از allocator را در مسیر ساخت `shared_ptr` فراهم می‌کند.

اگر واقعاً به مالکیت اشتراکی نیاز ندارید، استفاده از `shared_ptr` صرفاً برای ساخت بافر معمولاً انتخاب مناسبی نیست.

### ویژگی constexpr در نسخه‌های جدیدتر

این تابع از ++C20 معرفی شده است؛ اما `constexpr` شدن overloadهای آن از ++C23 است. بنابراین نباید قابلیت `constexpr` را به نسخه اولیه ++C20 نسبت داد.

## بررسی پشتیبانی کتابخانه استاندارد

برای بررسی وجود این قابلیت در کتابخانه استاندارد، می‌توان از feature-test macro زیر استفاده کرد:

```cpp
#include <memory>

#if defined(__cpp_lib_smart_ptr_for_overwrite) && \
    __cpp_lib_smart_ptr_for_overwrite >= 202002L

// std::make_unique_for_overwrite is available.

#else

#error "The standard library does not provide smart pointer overwrite support"

#endif
```

مقدار استاندارد این macro برابر `202002L` است. این macro پشتیبانی کتابخانه استاندارد از قابلیت‌های smart pointer مربوط به overwrite را نشان می‌دهد؛ بنابراین برای بررسی قابلیت کتابخانه مناسب‌تر از اتکا به نام کامپایلر به‌تنهایی است.

برای استفاده عملی، باید هم زبان و هم کتابخانه استاندارد با قابلیت موردنیاز پیکربندی شده باشند. برای نمونه، انتخاب حالت ++C20 در کامپایلر به‌تنهایی تضمین نمی‌کند که نسخه کتابخانه استاندارد نصب‌شده تمام قابلیت‌های لازم را پیاده‌سازی کرده باشد.

## راهنمای انتخاب در طراحی نرم‌افزار

انتخاب میان `std::make_unique` و `std::make_unique_for_overwrite` باید بر اساس قرارداد مقداردهی داده انجام شود، نه صرفاً بر اساس امید به افزایش سرعت.

- اگر مقدار صفر یا مقداردهی اولیه معمول برای عناصر لازم است، از `std::make_unique` استفاده کنید.
- اگر تمام عناصر بلافاصله توسط کد قابل اعتماد مقداردهی می‌شوند و پیش از آن خوانده نخواهند شد، `std::make_unique_for_overwrite` می‌تواند مناسب باشد.
- اگر فقط بخشی از بافر نوشته می‌شود، طول بخش معتبر را جداگانه نگهداری کنید.
- اگر نوع موردنظر class است، سازنده پیش‌فرض و مقداردهی اعضای آن را بررسی کنید.
- اگر آرگومان سازنده لازم دارید، از overloadهای معمول `std::make_unique` استفاده کنید.
- اگر مالکیت اشتراکی واقعاً لازم است، خانواده `std::make_shared_for_overwrite` را بررسی کنید.

در کدهای حساس، می‌توان مرز میان بخش مقداردهی‌نشده و بخش معتبر را با یک متغیر اندازه یا نوع wrapper مشخص کرد. این کار احتمال خواندن ناخواسته داده مقداردهی‌نشده را کاهش می‌دهد.

## جمع‌بندی

تابع `std::make_unique_for_overwrite` در ++C20 راهی استاندارد و مبتنی بر RAII برای ساخت شیء یا آرایه با مالکیت انحصاری فراهم می‌کند، درحالی‌که به‌جای `value-initialization` از `default-initialization` استفاده می‌کند.

مزیت اصلی آن حذف مقداردهی اولیه‌ای است که در بعضی مسیرها بلافاصله با داده واقعی جایگزین می‌شود. در مقابل، مسئولیت اطمینان از مقداردهی کامل پیش از خواندن داده بر عهده برنامه‌نویس است.

قاعده عملی ساده است: تنها زمانی از این تابع استفاده کنید که بتوانید ثابت کنید هر بخش از شیء یا بافر پیش از اولین خواندن، مقدار معتبر دریافت می‌کند. در غیر این صورت، `std::make_unique` انتخاب امن‌تر و روشن‌تری خواهد بود.

## منابع و مراجع

- [Working Draft: C++ Standard — Unique pointer creation](https://eel.is/c++draft/unique.ptr.create)
- [cppreference — std::make_unique and std::make_unique_for_overwrite](https://en.cppreference.com/w/cpp/memory/unique_ptr/make_unique)
- [cppreference — Default-initialization](https://en.cppreference.com/w/cpp/language/default_initialization)
- [WG21 P1973R1 — Rename “_default_init” Functions](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p1973r1.pdf)


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>