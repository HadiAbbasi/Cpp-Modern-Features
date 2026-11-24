<div align="center">

[🇺🇸 English](../../en/cpp23/std_format_improvements.md) | [🇮🇷 فارسی](./std_format_improvements.md)

</div>

---

# بهبودهای `std::format` در ++C23

## مقدمه

کتابخانهٔ Formatting که با استاندارد ++C20 معرفی شد، راهی نوع‌امن و قابل‌گسترش برای ساخت رشته‌های قالب‌بندی‌شده فراهم کرد. این کتابخانه در بسیاری از کاربردها جایگزین مناسبی برای خانوادهٔ `printf` و راهی مکمل برای جریان‌های ورودی/خروجی سنتی مانند `std::cout` است. بااین‌حال، نسخهٔ اولیهٔ آن محدودیت‌هایی داشت: قالب‌بندی مستقیم بسیاری از Containerها ممکن نبود، بعضی انواع کتابخانهٔ استاندارد Formatter نداشتند، رفتار عرض متن‌های Unicode و نویسهٔ Fill به‌اندازهٔ کافی دقیق نبود و استفاده از قالب‌های پویا و ساخت ابزارهای عمومی‌تر به امکانات بیشتری نیاز داشت.

استاندارد ++C23 این زیرساخت را در چند جهت گسترش داد: پشتیبانی استاندارد از قالب‌بندی Rangeها و Containerها، امکانات بهتر برای نوشتن Formatter سفارشی، اصلاحات مرتبط با Unicode و عرض نمایشی، بهبود Escape کردن رشته‌ها، و پشتیبانی از انواعی مانند شناسهٔ Thread و Stacktrace. همچنین تعدادی اصلاحیهٔ فنی و بهبود سازگاری به امکاناتی رسیدند که در ++C20 معرفی شده بودند.

نکتهٔ مهم این است که همهٔ مواردی که در نسخهٔ ++C23 دیده می‌شوند الزاماً یک قابلیت کاملاً تازه در خود زبان نیستند. `std::format` یک قابلیت **کتابخانهٔ استاندارد** است که از Header مربوط به `<format>` استفاده می‌کند. در نتیجه، فعال بودن گزینهٔ `-std=c++23` به‌تنهایی تضمین نمی‌کند که تمام اجزای آن در نسخهٔ کتابخانهٔ استاندارد نصب‌شده پیاده‌سازی شده باشند.

## مروری بر جایگاه `std::format`

### قابلیت‌های پایه در ++C20

قابلیت `std::format` از ++C20 آغاز شد و توابع اصلی آن در Header زیر قرار دارند:

```cpp
#include <format>
```

تابع `std::format` نتیجه را در یک `std::string` جدید تولید می‌کند:

```cpp
#include <format>
#include <iostream>
#include <string>

int main() {
    const std::string user = "Ava";
    const int score = 97;

    const auto message = std::format(
        "User: {}, score: {:03}",
        user,
        score
    );

    std::cout << message << '\n';
}
```

در این مثال، `{}` محل قرارگیری Argument است و `{:03}` عدد را با حداقل سه رقم و با صفر در سمت چپ قالب‌بندی می‌کند. ترتیب Argumentها به‌صورت پیش‌فرض با ترتیب Fieldها هماهنگ است؛ همچنین می‌توان از Index صریح مانند `{0}` و `{1}` استفاده کرد.

### خانوادهٔ توابع قالب‌بندی

علاوه بر `std::format`، چند تابع برای سناریوهای متفاوت وجود دارد:

| تابع | کاربرد |
|---|---|
| `std::format` | ساخت یک رشتهٔ جدید |
| `std::format_to` | نوشتن خروجی در یک Output Iterator |
| `std::format_to_n` | نوشتن حداکثر تعداد مشخصی از نویسه‌ها |
| `std::formatted_size` | محاسبهٔ تعداد نویسه‌های خروجی قالب‌بندی‌شده |
| `std::vformat` | قالب‌بندی با Argumentهای Type-erased |
| `std::vformat_to` | نسخهٔ Type-erased برای نوشتن در Output Iterator |

برای مثال، زمانی که مقصد یک Buffer یا Iterator است، `std::format_to` می‌تواند از ساخت رشتهٔ میانی جلوگیری کند:

```cpp
#include <format>
#include <iterator>
#include <string>

int main() {
    std::string output;
    output.reserve(64);

    std::format_to(
        std::back_inserter(output),
        "id={}, active={}",
        42,
        true
    );
}
```

این APIها از ++C20 هستند؛ اهمیت آن‌ها در بحث ++C23 این است که قابلیت‌های جدید Range Formatting و Formatter سفارشی با همین زیرساخت کار می‌کنند.

## مهم‌ترین تغییرات مرتبط با قالب‌بندی در ++C23

### قالب‌بندی مستقیم Rangeها

یکی از مهم‌ترین تغییرات ++C23، پشتیبانی از قالب‌بندی Rangeها است که در پیشنهاد استانداردسازی P2286R8 مطرح شد. پیش از این تغییر، برای چاپ یک `std::vector` یا بسیاری از Containerها معمولاً باید با حلقه عناصر را پیمایش می‌کردیم یا Formatter سفارشی می‌نوشتیم.

در ++C23، انواع Range قابل‌قالب‌بندیِ پشتیبانی‌شده می‌توانند مستقیماً به `std::format` داده شوند:

```cpp
#include <format>
#include <iostream>
#include <vector>

int main() {
    const std::vector<int> values{10, 20, 30, 40};

    std::cout << std::format("values = {}", values) << '\n';
}
```

نمایش پیش‌فرض یک Range معمولاً شامل جداکنندهٔ عناصر و براکت‌های مشخص‌کنندهٔ محدوده است. شکل دقیق نمایش به نوع Range و دسته‌بندی آن در قواعد کتابخانه وابسته است؛ بنابراین بهتر است برای خروجی‌ای که بخشی از یک Protocol یا فرمت پایدار ذخیره‌سازی است، به نمایش پیش‌فرض متکی نشوید.

#### انواع Range و دسته‌بندی قالب‌بندی

کتابخانه برای انتخاب شیوهٔ نمایش Rangeها از `std::format_kind` و `std::range_format` استفاده می‌کند. دسته‌بندی‌های مهم شامل موارد زیر هستند:

- `sequence`: نمایش دنباله‌ای از عناصر، مانند بسیاری از Containerهای ترتیبی.
- `map`: نمایش Rangeهای کلید/مقدار مانند Mapها.
- `set`: نمایش Rangeهای مجموعه‌ای.
- `string`: نمایش نوعی که باید به‌عنوان رشته در نظر گرفته شود، نه دنباله‌ای از نویسه‌های مستقل.
- `debug_string`: نمایش رشته با قواعد Debug و Escape مناسب.
- `disabled`: غیرفعال بودن قالب‌بندی عمومی Range برای نوع موردنظر.

این دسته‌بندی‌ها بخشی از سازوکار کتابخانه‌اند و برای کنترل یا سفارشی‌سازی Formatterهای Range اهمیت دارند. برای مثال، `std::string` نباید مانند یک Container عمومی از `char`ها چاپ شود؛ رفتار رشته‌ای آن باید حفظ شود.

#### نمونهٔ قالب‌بندی Containerهای مختلف

```cpp
#include <format>
#include <iostream>
#include <map>
#include <set>
#include <string>
#include <vector>

int main() {
    const std::vector<int> numbers{1, 2, 3};
    const std::set<int> unique_numbers{4, 5, 6};
    const std::map<std::string, int> ages{
        {"Ada", 36},
        {"Linus", 54}
    };

    std::cout << std::format("{}\n", numbers);
    std::cout << std::format("{}\n", unique_numbers);
    std::cout << std::format("{}\n", ages);
}
```

قالب‌بندی Rangeها برای Logging، ساخت پیام‌های Diagnostic، تست‌ها و نمایش وضعیت Containerها بسیار مفید است. بااین‌حال، نباید آن را جایگزین Serialization دانست؛ خروجی قالب‌بندی‌شده الزاماً قرارداد قابل‌حمل و نسخه‌پذیر برای تبادل داده نیست.

### کنترل قالب‌بندی Range با Format Specifier

قابلیت Range Formatting تنها به نمایش پیش‌فرض محدود نیست. `std::range_formatter` زیرساختی برای تعریف رفتار قالب‌بندی Range فراهم می‌کند و امکان تعیین مشخصه‌های مرتبط با عناصر، جداکننده و براکت‌ها را می‌دهد.

برای نمونه، می‌توان Formatter سفارشی یک نوع Range را بر پایهٔ `std::range_formatter` پیاده‌سازی کرد. نمونهٔ زیر یک Wrapper ساده را با قالب‌بندی مبتنی بر Range نشان می‌دهد:

```cpp
#include <format>
#include <string>
#include <vector>

struct Scores {
    std::vector<int> values;
};

template <>
struct std::formatter<Scores> : std::range_formatter<int> {
    auto format(const Scores& scores, std::format_context& ctx) const {
        return std::range_formatter<int>::format(scores.values, ctx);
    }
};

int main() {
    const Scores scores{{80, 90, 100}};
    const auto text = std::format("{}", scores);
}
```

این مثال از `std::range_formatter` برای استفادهٔ مجدد از منطق قالب‌بندی Range بهره می‌برد. در پروژهٔ واقعی، هنگام طراحی Formatter باید نوع Character و Context را نیز در نظر گرفت؛ برای پشتیبانی از `wchar_t` یا Character Typeهای دیگر، تخصصی‌سازی صرفاً برای `char` کافی نیست.

### بهبود قالب‌بندی پیش‌فرض Containerها

پیشنهاد P2585R1 با عنوان Improving default container formatting به تکمیل و اصلاح رفتار پیش‌فرض قالب‌بندی Containerها در کنار Range Formatting مربوط است. هدف این تغییر آن است که نمایش Containerهای استاندارد برای استفادهٔ روزمره و Diagnostic کاربردی‌تر و منسجم‌تر باشد.

نکتهٔ طراحی این است که نمایش پیش‌فرض برای خوانایی انسان بهینه شده است، نه برای تضمین یک Syntax مستقل و قابل‌تجزیه. اگر برنامه خروجی را در فایل، شبکه یا پایگاه داده ذخیره می‌کند و بعداً باید آن را Parse کند، باید یک Serialization Format مشخص مانند JSON یا یک قالب اختصاصی با قرارداد نسخه‌بندی‌شده انتخاب شود.

### مفهوم `std::formattable`

استاندارد ++C23 مفهوم `std::formattable` را معرفی می‌کند. این Concept مشخص می‌کند که آیا یک نوع برای Character Type مشخصی قابل قالب‌بندی است یا خیر.

```cpp
#include <concepts>
#include <format>
#include <string>

static_assert(std::formattable<int, char>);
static_assert(std::formattable<std::string, char>);
```

پارامتر دوم Character Type قالب‌بندی را مشخص می‌کند. بنابراین Formattable بودن یک نوع برای `char` الزاماً به معنای Formattable بودن آن برای `wchar_t` نیست.

این Concept برای نوشتن APIهای Generic مفید است؛ به‌جای آنکه خطاهای پیچیدهٔ مربوط به `std::formatter` در عمق Templateها ظاهر شوند، می‌توان محدودیت را در امضای تابع بیان کرد:

```cpp
#include <format>
#include <string>

template <typename T>
requires std::formattable<T, char>
std::string make_message(const T& value) {
    return std::format("value = {}", value);
}
```

در این نمونه، تابع فقط برای نوع‌هایی قابل استفاده است که برای قالب‌بندی Narrow Character با `char` مناسب باشند.

### بهبود پشتیبانی از نوع‌های غیر `const`-formattable

پیشنهاد P2418R2 محدودیت‌هایی را که مانع قالب‌بندی بعضی Objectهای غیرقابل‌کپی یا غیرقابل‌قالب‌بندی از طریق `const` بودند کاهش داد. این تغییر برای نوع‌هایی اهمیت دارد که عملیات مشاهده یا تولید نمایش آن‌ها ممکن است به دسترسی غیر `const` نیاز داشته باشد؛ Generatorها و برخی Rangeهای تک‌بارمصرف نمونه‌های مهمی از این گروه هستند.

پیش از این اصلاح، مدل قالب‌بندی در مواردی بیش از حد به دسترسی `const` وابسته بود. در نسخهٔ اصلاح‌شده، کتابخانه می‌تواند در صورت لزوم از شیء غیر `const` استفاده کند؛ البته این موضوع به قیود و نوع Range یا Formatter وابسته است و به معنای مجاز شدن تغییر دلخواه State در تمام Formatterها نیست.

این تغییر به‌صورت Defect Report نیز به ++C20 اعمال شده است؛ بنابراین ممکن است یک پیاده‌سازی در حالت ++C20 نیز آن را ارائه کند. برای تشخیص، باید مستندات و Feature-test Macro همان کتابخانه بررسی شود.

## بهبودهای مرتبط با رشته و Unicode

### اصلاح قواعد عرض نمایشی

یکی از نکات دشوار قالب‌بندی متن، تفاوت تعداد Code Unitها با عرضی است که متن روی Terminal یا محیط نمایشی اشغال می‌کند. برای مثال، یک نویسهٔ Unicode ممکن است در UTF-8 چند Byte داشته باشد، اما روی صفحه یک ستون یا حتی دو ستون اشغال کند. همچنین یک Grapheme Cluster ممکن است از چند Code Point تشکیل شده باشد.

پیشنهاد P2675R1 تخمین عرض در `std::format` را دقیق‌تر و آینده‌نگرانه‌تر می‌کند. این موضوع به‌ویژه برای Alignment و Padding اهمیت دارد؛ زیرا عرض فیلد نباید صرفاً با تعداد Byteهای رشته یکی فرض شود.

```cpp
#include <format>
#include <iostream>

int main() {
    std::cout << std::format("|{:^12}|\n", "hello");
}
```

در این مثال، رشته در فیلدی با عرض مشخص و به‌صورت Center-aligned قرار می‌گیرد. هنگام استفاده از متن Unicode، نتیجهٔ بصری به قواعد تخمین عرض استاندارد، Encoding و پشتیبانی پیاده‌سازی وابسته است.

باید توجه داشت که `std::format` یک موتور کامل Layout برای متن‌های پیچیده نیست. این کتابخانه جایگزین الگوریتم‌های پیشرفتهٔ صفحه‌آرایی، Line Breaking، BiDi یا شکل‌دهی متن زبان‌های طبیعی محسوب نمی‌شود.

### انعطاف بیشتر در Fill Character

در Syntax قالب‌بندی، نویسهٔ Fill برای پر کردن فضای خالی اطراف مقدار استفاده می‌شود. پیشرفت P2572R1 قواعد Fill Character را اصلاح کرد تا محدودیت‌های غیرضروری کاهش یابد و رفتار آن در حضور نویسه‌های Unicode روشن‌تر شود.

نمونهٔ سادهٔ Fill و Alignment:

```cpp
#include <format>
#include <iostream>

int main() {
    std::cout << std::format("{:*^10}\n", "C++");
    std::cout << std::format("{:.>10}\n", 42);
}
```

در مثال اول، `*` فضای خالی دو طرف رشته را پر می‌کند. در مثال دوم، نقطه برای پر کردن سمت چپ عدد استفاده می‌شود.

بااین‌حال، عرض نمایشی Fill Character همیشه با تعداد Code Unit آن برابر نیست. انتخاب نویسه‌های چندستونی یا Graphemeهای پیچیده ممکن است باعث شود ظاهر نهایی در Terminalهای مختلف یکسان نباشد. P2572R1 این حوزه را از نظر قواعد استاندارد روشن‌تر می‌کند، اما تفاوت Font، Terminal و محیط نمایش را از بین نمی‌برد.

### بهبود Escape کردن رشته‌ها

پیشنهاد P2713R1 امکانات Escape در `std::format` را بهبود می‌دهد. این قابلیت برای نمایش امن‌تر و قابل‌فهم‌تر رشته‌ها در خروجی Debug مفید است؛ به‌ویژه وقتی رشته شامل نویسه‌های کنترلی، کوتیشن یا نویسه‌هایی باشد که نمایش مستقیم آن‌ها خوانایی خروجی را مختل می‌کند.

در Format Specification مربوط به رشته‌ها، Presentation مربوط به Debug با `?` قابل استفاده است:

```cpp
#include <format>
#include <iostream>
#include <string>

int main() {
    const std::string text = "first line\nsecond line";

    std::cout << std::format("normal: {}\n", text);
    std::cout << std::format("debug:  {:?}\n", text);
}
```

قالب‌بندی عادی رشته، محتوای آن را همان‌طور که هست وارد خروجی می‌کند؛ درحالی‌که Debug Formatting می‌تواند نویسه‌های ویژه را به شکل Escape‌شده نشان دهد. این تفاوت در Logging و گزارش خطا اهمیت دارد، چون یک Newline یا Tab پنهان ممکن است ساختار لاگ را به‌هم بزند.

خروجی Debug را نباید بدون قرارداد مشخص، فرمت Serialization در نظر گرفت. هدف اصلی آن نمایش تشخیصی است و نه تضمین Round-trip برای تمام Encodingها و داده‌ها.

## گسترش نوع‌های قابل‌قالب‌بندی

### قالب‌بندی شناسهٔ Thread

پیشنهاد P2693R1 برای `std::thread::id` امکان استفادهٔ مستقیم از این نوع در `std::format` را فراهم می‌کند:

```cpp
#include <format>
#include <iostream>
#include <thread>

int main() {
    const std::thread::id id = std::this_thread::get_id();

    std::cout << std::format("current thread: {}\n", id);
}
```

این قابلیت در Logging چندنخی و Diagnosticهای مربوط به Concurrency مفید است. شناسهٔ Thread یک مقدار قابل‌نمایش برای شناسایی Thread در محدودهٔ اجرای برنامه است؛ نباید آن را شناسه‌ای پایدار میان اجراهای مختلف یا یک مقدار قابل‌حمل برای ذخیره‌سازی بلندمدت فرض کرد.

### قالب‌بندی Stacktrace

همان پیشنهاد، پشتیبانی قالب‌بندی از نوع‌های مرتبط با Stacktrace را نیز اضافه می‌کند. قابلیت Stacktrace در کتابخانهٔ استاندارد از ++C23 در Header مربوط به `<stacktrace>` ارائه شده است و برای ثبت مسیر فراخوانی‌ها در زمان وقوع خطا مفید است.

```cpp
#include <format>
#include <iostream>
#include <stacktrace>

int main() {
    const auto trace = std::stacktrace::current();

    std::cout << std::format("stack trace:\n{}\n", trace);
}
```

در محیط‌هایی که پشتیبانی کامل Stacktrace ندارند یا امکان جمع‌آوری اطلاعات Stack وجود ندارد، رفتار و جزئیات خروجی می‌تواند به پیاده‌سازی و تنظیمات Build وابسته باشد. همچنین Stacktrace ممکن است اطلاعات حساس داخلی برنامه، مسیر فایل‌ها یا نام Symbolها را افشا کند؛ بنابراین انتشار آن در Logهای عمومی یا پاسخ‌های شبکه باید با ملاحظات امنیتی انجام شود.

### معرفی `std::basic_format_string`

پیشنهاد P2508R1 نام عمومی `std::basic_format_string` را در معرض استفادهٔ برنامه‌نویس قرار داد. این نوع زیربنای Format Stringهای نوع‌امن است و امکان تعریف APIهایی را می‌دهد که Format String آن‌ها در زمان Compile بررسی شود.

```cpp
#include <format>
#include <string_view>

template <typename... Args>
std::string make_text(
    std::format_string<Args...> fmt,
    Args&&... args
) {
    return std::format(fmt, std::forward<Args>(args)...);
}

int main() {
    const auto text = make_text("id = {}, name = {}", 7, "Ada");
}
```

در این نمونه از Alias استاندارد `std::format_string` استفاده شده است که بر پایهٔ `std::basic_format_string` تعریف می‌شود. در نتیجه، ناسازگاری میان تعداد یا نوع Argumentها و Format Specifierها در حالت‌های قابل‌بررسی می‌تواند در زمان Compile تشخیص داده شود.

این قابلیت به‌ویژه برای ساخت Wrapperهای Logging و Formatting مهم است. اگر Format String در زمان اجرا از فایل یا ورودی کاربر دریافت شود، مسیر بررسی آن متفاوت است و نباید فرض کرد که هر رشتهٔ Runtime به همان شکل Compile-time بررسی می‌شود.

## اصلاحات و شفاف‌سازی‌های مرتبط با ++C20

### بررسی Compile-time قالب‌ها

پیشنهاد P2216R3 بررسی Format String را در زمان Compile تقویت کرد و کیفیت تشخیص خطاها را بهبود داد. این ویژگی در جدول Feature-test با مقدار `__cpp_lib_format == 202106L` شناخته می‌شود و به‌عنوان Defect Report به ++C20 نیز مرتبط است.

```cpp
#include <format>

int main() {
    const auto valid = std::format("value = {}", 42);

    // This format string is intentionally invalid.
    // A conforming implementation should diagnose the mismatch.
    // const auto invalid = std::format("{:d}", "text");
}
```

در حالت عادی، Format String ثابت و قابل ارزیابی در زمان Compile به‌صورت Type-safe بررسی می‌شود. البته جزئیات Diagnostic و میزان دقیق بررسی ممکن است میان Compilerها متفاوت باشد؛ آنچه استاندارد تضمین می‌کند قواعد معنایی و الزامات تشخیص است، نه متن یکسان پیام خطا.

### اصلاح Locale در Formatterهای Chrono

پیشنهاد P2372R3 رفتار Locale را در Formatterهای مرتبط با `std::chrono` اصلاح کرد. این تغییر به نحوهٔ اعمال Locale در قالب‌بندی تاریخ و زمان مربوط است و برای برنامه‌هایی اهمیت دارد که نمایش تاریخ، نام ماه‌ها یا سایر اطلاعات وابسته به Locale را تولید می‌کنند.

پیشنهاد P2419R2 نیز نحوهٔ برخورد با Encoding در قالب‌بندی Localized انواع Chrono را شفاف‌تر می‌کند. این اصلاحات در مرز میان قالب‌بندی متن و داده‌های تاریخ/زمان قرار می‌گیرند؛ در نتیجه هنگام استفاده از `L` در Format Specifier و انتخاب Locale باید Encoding خروجی و Character Type نیز در نظر گرفته شود.

نمونهٔ کلی قالب‌بندی زمان:

```cpp
#include <chrono>
#include <format>
#include <iostream>

int main() {
    const auto now = std::chrono::system_clock::now();

    std::cout << std::format("{:%Y-%m-%d %H:%M:%S}\n", now);
}
```

جزئیات Format Specifierهای Chrono از قابلیت‌های ++C20 هستند؛ اصلاحات ++C23 در این بخش بیشتر بر سازگاری Locale و شفافیت Encoding متمرکزند.

### کاهش پارامتردهی در `std::vformat_to`

بخش دیگری از P2216R3 به کاهش پارامتردهی غیرضروری در `std::vformat_to` مربوط است. این تغییر بیشتر ماهیت کتابخانه‌ای و زیرساختی دارد و معمولاً در کد روزمره مستقیماً دیده نمی‌شود؛ اما برای پیاده‌سازی‌های استاندارد و توسعه‌دهندگان APIهای مبتنی بر Type-erased Formatting اهمیت دارد.

## نوشتن Formatter سفارشی

### ساخت Formatter برای نوع کاربر

یکی از ویژگی‌های کلیدی Formatting Library، امکان تعریف `std::formatter` برای نوع‌های User-defined است. این قابلیت از ++C20 وجود داشت و در ++C23 با Range Formatting و Conceptهای جدید انعطاف بیشتری پیدا کرد.

```cpp
#include <format>
#include <string>

struct User {
    std::string name;
    int age;
};

template <>
struct std::formatter<User> : std::formatter<std::string> {
    auto format(const User& user, std::format_context& ctx) const {
        const auto text = std::format(
            "{} ({})",
            user.name,
            user.age
        );

        return std::formatter<std::string>::format(text, ctx);
    }
};

int main() {
    const User user{"Ada", 36};
    const auto text = std::format("{}", user);
}
```

در این الگو، Formatter نوع `User` از Formatter رشته استفاده می‌کند تا قواعد پایهٔ Alignment و Width را دوباره پیاده‌سازی نکند. برای یک Formatter عمومی‌تر، باید توجه داشت که `std::format_context` مشخصاً Context مربوط به `char` است. اگر پشتیبانی از Character Typeهای دیگر لازم باشد، بهتر است از الگوی Template متناسب با `CharT` و Context استفاده شود.

### طراحی Format Specifier اختصاصی

Formatter سفارشی می‌تواند Format Specifier مخصوص خود را تعریف کند. این کار معمولاً با پیاده‌سازی `parse` و `format` انجام می‌شود:

```cpp
#include <format>
#include <string_view>

struct Point {
    int x;
    int y;
};

template <>
struct std::formatter<Point> {
    bool compact = false;

    constexpr auto parse(std::format_parse_context& ctx) {
        auto it = ctx.begin();

        if (it != ctx.end() && *it == 'c') {
            compact = true;
            ++it;
        }

        if (it != ctx.end() && *it != '}') {
            throw std::format_error("invalid Point format");
        }

        return it;
    }

    auto format(const Point& point, std::format_context& ctx) const {
        if (compact) {
            return std::format_to(
                ctx.out(),
                "({}, {})",
                point.x,
                point.y
            );
        }

        return std::format_to(
            ctx.out(),
            "Point{{x={}, y={}}}",
            point.x,
            point.y
        );
    }
};

int main() {
    const Point p{10, 20};

    const auto normal = std::format("{}", p);
    const auto compact = std::format("{:c}", p);
}
```

این مثال نشان می‌دهد که Formatter چگونه می‌تواند یک Grammar کوچک برای Format Specifier خود تعریف کند. در کد تولیدی، باید رفتار `parse` برای ورودی خالی، پایان Field و ورودی نامعتبر دقیق باشد. همچنین Format String باید در زمان Compile تا حد ممکن قابل بررسی باقی بماند.

### استفاده از `std::range_formatter`

در مواردی که نوع سفارشی در اصل یک Range را نگه‌داری می‌کند، `std::range_formatter` می‌تواند بخش زیادی از کار را برعهده بگیرد. این کار از تکرار منطق جداکننده، براکت‌ها و Formatter عناصر جلوگیری می‌کند.

برای انتخاب این روش، ابتدا بررسی کنید که آیا نوع شما واقعاً باید از نظر معنایی مانند یک Range نمایش داده شود یا یک نمایش دامنه‌ای مستقل دارد. برای مثال، یک ماتریس، مسیر شبکه یا مجموعهٔ تراکنش‌ها ممکن است نیازمند قالب‌بندی‌ای باشد که صرفاً فهرست عناصر نباشد.

## کارایی و طراحی API

### انتخاب میان `format` و `format_to`

`std::format` یک رشتهٔ جدید می‌سازد؛ این رفتار برای بیشتر پیام‌ها خوانا و مناسب است. اگر مقصد از قبل یک Buffer یا Output Iterator باشد، `std::format_to` می‌تواند از ساخت رشتهٔ موقت جلوگیری کند.

```cpp
#include <format>
#include <iterator>
#include <string>

int main() {
    std::string log_line;
    log_line.reserve(128);

    std::format_to(
        std::back_inserter(log_line),
        "[{}] user={} status={}",
        1234,
        "Ada",
        200
    );
}
```

انتخاب `format_to` به‌خودی‌خود تضمین نمی‌کند که هیچ Allocationای رخ ندهد؛ مقصد، رشد Buffer و Formatterهای Argumentها همچنان می‌توانند حافظه تخصیص دهند. برای بهینه‌سازی واقعی باید مسیر کامل تولید خروجی Benchmark شود.

### بهینه‌سازی Non-locking Formatter

استاندارد ++C23 متغیر قالبی `std::enable_nonlocking_formatter_optimization` را معرفی می‌کند. این Trait به پیاده‌سازی اجازه می‌دهد برای نوع‌هایی که Formatter آن‌ها به سازوکار Locking نیاز ندارد، مسیر بهینه‌تری انتخاب کند.

این قابلیت یک ابزار برای پیاده‌سازی کتابخانهٔ استاندارد است و نباید به‌عنوان تضمین عمومی Thread-safety یا حذف تمام Lockها تفسیر شود. برنامه‌نویس باید فقط در صورت احراز دقیق الزامات استاندارد، تخصصی‌سازی مربوط به نوع خود را ارائه کند. اعلام نادرست این ویژگی می‌تواند رفتار برنامه را از الزامات پیاده‌سازی دور کند.

## تشخیص پشتیبانی کتابخانه و Compiler

### Feature-test Macroهای مهم

برای قابلیت‌های کتابخانه‌ای، بررسی Feature-test Macro از اتکا به شمارهٔ نسخهٔ Compiler دقیق‌تر است. Macroها در Headerهای مربوط یا `<version>` در دسترس هستند.

| Macro | مقدار | قابلیت |
|---|---:|---|
| `__cpp_lib_format` | `201907L` | قابلیت پایهٔ Text Formatting در ++C20 |
| `__cpp_lib_format` | `202106L` | Compile-time Format String Checks و اصلاحات مرتبط |
| `__cpp_lib_format` | `202110L` | اصلاحات Locale در Chrono و پشتیبانی از Non-const-formattable Types |
| `__cpp_lib_format` | `202207L` | نمایان شدن `std::basic_format_string` و شفاف‌سازی Encoding |
| `__cpp_lib_format_ranges` | `202207L` | قالب‌بندی Rangeها در ++C23 |
| `__cpp_lib_formatters` | `202302L` | Formatterهای `std::thread::id` و `std::stacktrace` در ++C23 |

برای نمونه، می‌توان وجود Range Formatting را به شکل زیر بررسی کرد:

```cpp
#include <format>

#if defined(__cpp_lib_format_ranges) && \
    __cpp_lib_format_ranges >= 202207L
    #define HAS_STD_FORMAT_RANGES 1
#else
    #define HAS_STD_FORMAT_RANGES 0
#endif

int main() {
    static_assert(HAS_STD_FORMAT_RANGES == 0 ||
                  HAS_STD_FORMAT_RANGES == 1);
}
```

این بررسی نشان می‌دهد که قابلیت در کتابخانه موجود است یا خیر؛ اما برای استفاده از Syntax و APIهای استاندارد ++C23 همچنان باید Compiler و کتابخانه با حالت زبانی مناسب پیکربندی شوند.

### چرا نسخهٔ Compiler کافی نیست؟

پشتیبانی از `std::format` به دو جزء وابسته است:

- **Compiler:** باید حالت زبانی و قابلیت‌های لازم ++C23 را بپذیرد.
- **Standard Library:** باید APIهای موردنیاز مانند Range Formatterها را پیاده‌سازی کرده باشد.

ممکن است یک نسخهٔ Compiler از بخش‌هایی از ++C23 پشتیبانی کند، اما کتابخانهٔ همراه آن هنوز بعضی قابلیت‌های Formatting را نداشته باشد. همچنین ممکن است همان کتابخانه در نسخهٔ جدیدتر، برخی اصلاحیه‌های استاندارد را حتی در حالت ++C20 نیز ارائه کند.

برای پروژه‌های چندپلتفرمی، بهتر است قابلیت‌های موردنیاز در CI روی ترکیب‌های واقعی Compiler و Standard Library آزمایش شوند.

## مقایسهٔ رفتار ++C20 و ++C23

| موضوع | ++C20 | ++C23 |
|---|---|---|
| `std::format` و خانوادهٔ اصلی توابع | معرفی شده | حفظ شده و اصلاح شده |
| بررسی Format String در Compile-time | در نسخهٔ اولیه محدودتر؛ با DR بهبود یافته | قواعد اصلاح‌شده و Diagnostic بهتر |
| قالب‌بندی عمومی Rangeها | پشتیبانی استاندارد عمومی وجود نداشت | با P2286R8 اضافه شده |
| `std::range_formatter` | وجود نداشت | اضافه شده |
| `std::formattable` | وجود نداشت | اضافه شده |
| Formatterهای `std::thread::id` و Stacktrace | پشتیبانی استاندارد عمومی نداشت | در P2693R1 استانداردسازی شده |
| قواعد Fill و تخمین عرض Unicode | محدودیت‌ها و ابهام‌هایی داشت | با P2572R1 و P2675R1 بهبود یافته |
| Debug Escaping رشته‌ها | محدودتر | با P2713R1 بهبود یافته |
| `std::basic_format_string` | در متن اولیه نام عمومی در دسترس نبود | با P2508R1 نمایان شده؛ به‌صورت DR نیز قابل اعمال است |

این جدول تفاوت نسخهٔ اولیهٔ استانداردها را نشان می‌دهد، نه وضعیت تمام کتابخانه‌های موجود. Defect Reportها ممکن است قابلیت یا اصلاحی را به حالت ++C20 یک پیاده‌سازی نیز منتقل کنند.

## محدودیت‌ها و خطاهای رایج

### اشتباه گرفتن Format String ثابت و پویا

Format Stringهای ثابت در APIهای نوع‌امن مانند `std::format` امکان بررسی در Compile-time دارند. اما رشته‌ای که از فایل، شبکه یا ورودی کاربر دریافت شده، یک رشتهٔ Runtime است و نباید بدون توجه به API و نسخهٔ استاندارد، همان رفتار را برای آن فرض کرد.

در ++C23، برای سناریوهای Format String پویا همچنان باید API مناسب و محدودیت‌های استاندارد مورد استفاده بررسی شود. API استاندارد `std::runtime_format` متعلق به ++C26 است، نه ++C23؛ بنابراین نباید آن را به‌عنوان قابلیت استاندارد ++C23 معرفی کرد. در نسخه‌های پیش از آن، `std::vformat` برای سناریوهای Type-erased و قالب‌های Runtime کاربرد دارد، اما باید Argumentها و طول عمر آن‌ها به‌درستی مدیریت شوند.

### فرض کردن پشتیبانی کامل همهٔ Rangeها

قابل‌قالب‌بندی بودن یک Range به Constraints و Formattable بودن عناصر آن وابسته است. هر نوعی که بتوان روی آن حلقه نوشت الزاماً با هر Character Type قابل‌قالب‌بندی نیست. همچنین برخی Rangeها ممکن است تک‌بارمصرف باشند یا برای پیمایش به State غیر `const` نیاز داشته باشند.

پیش از طراحی API عمومی، بهتر است قیود `std::formattable` و الزامات Range Formatting بررسی شوند.

### استفاده از خروجی پیش‌فرض برای Serialization

نمایش پیش‌فرض Containerها برای Debug و Logging طراحی شده است. نباید فرض کرد که این نمایش همیشه قابل Parse، مستقل از نسخه یا یکسان در همهٔ پیاده‌سازی‌هاست. برای ذخیره‌سازی پایدار، فرمت دادهٔ صریح و قرارداد سازگار با نسخه‌ها انتخاب کنید.

### نادیده گرفتن هزینهٔ قالب‌بندی

قالب‌بندی می‌تواند شامل Allocation، تبدیل Encoding، پیمایش Range و اجرای Formatterهای سفارشی باشد. برای مسیرهای پرتکرار مانند Logging در حلقه‌های حساس به کارایی، باید از ساخت رشته‌های غیرضروری پرهیز کرد، Buffer مناسب انتخاب کرد و Benchmark واقعی گرفت.

### نادیده گرفتن دادهٔ نامطمئن

اگر Format String از ورودی نامطمئن می‌آید، آن را با Format String ثابت برنامه یکی ندانید. Format String نامعتبر می‌تواند به خطای قالب‌بندی منجر شود و در طراحی Logging یا ابزارهای عمومی باید اعتبارسنجی و مدیریت خطا در نظر گرفته شود. همچنین محتوای Argumentها ممکن است شامل داده‌های حساس باشد؛ قالب‌بندی امن از نظر نوع، به‌تنهایی سیاست محرمانگی داده را تأمین نمی‌کند.

### فرض یکسان بودن نمایش Unicode در همهٔ Terminalها

استاندارد می‌تواند قواعد محاسبهٔ عرض و Fill را مشخص کند، اما Font، Terminal، Encoding محیط و پشتیبانی از Graphemeها بر ظاهر نهایی اثر می‌گذارند. برای رابط‌های متنی حساس به Layout، خروجی را در محیط‌های هدف آزمایش کنید.

## راهنمای انتخاب قابلیت مناسب

- برای ساخت یک رشتهٔ معمولی، `std::format` انتخاب ساده و خوانایی است.
- برای نوشتن مستقیم در Buffer یا Iterator، `std::format_to` را بررسی کنید.
- برای نمایش سریع Containerها در Log و تست، Range Formatting در ++C23 بسیار مفید است.
- برای ساخت API عمومی Generic، Constraint مبتنی بر `std::formattable` می‌تواند قرارداد تابع را روشن کند.
- برای نوع‌های دامنه‌ای، Formatter سفارشی بنویسید و در صورت مناسب بودن از Formatterهای موجود به‌عنوان پایه استفاده کنید.
- برای نمایش رشته‌های حاوی نویسه‌های کنترلی، Debug Formatting را در نظر بگیرید.
- برای دادهٔ پایدار و قابل‌تبادل، به‌جای خروجی پیش‌فرض Formatter از Serialization Format مشخص استفاده کنید.
- برای پروژه‌های چندکامپایلری، Feature-test Macro و آزمون Build واقعی را مبنا قرار دهید.

## جمع‌بندی

بهبودهای Formatting در ++C23 این کتابخانه را از یک ابزار ساخت رشته با پشتیبانی محدود از نوع‌ها، به زیرساختی کامل‌تر برای نمایش Rangeها و ساخت Formatterهای Generic تبدیل می‌کند. پشتیبانی از `std::formattable`، `std::range_formatter` و قالب‌بندی Containerها کدنویسی روزمره و Diagnosticها را ساده‌تر می‌سازد. اصلاحات Unicode، Fill Character و Debug Escaping نیز کیفیت نمایش متن را افزایش می‌دهند؛ درحالی‌که پشتیبانی از Thread ID و Stacktrace برای ابزارهای تشخیصی مفید است.

با وجود این پیشرفت‌ها، باید میان قالب‌بندی برای انسان و Serialization برای ماشین تفاوت قائل شد. همچنین پشتیبانی واقعی هر قابلیت به نسخهٔ Standard Library وابسته است، نه فقط گزینهٔ زبان Compiler. استفاده از Feature-test Macroها، آزمایش روی Toolchainهای هدف و طراحی دقیق Formatterهای سفارشی، راه استفادهٔ مطمئن از این امکانات در پروژه‌های حرفه‌ای است.

## منابع و مطالعهٔ بیشتر

- [cppreference — Formatting library](https://en.cppreference.com/w/cpp/utility/format)
- [cppreference — Feature-test macros](https://en.cppreference.com/w/cpp/feature_test)
- [WG21 P2286R8 — Formatting Ranges](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2286r8.html)
- [WG21 P2418R2 — Add support for non-const-formattable types to std::format](https://wg21.link/P2418R2)
- [WG21 P2508R1 — Exposing std::basic_format_string](https://wg21.link/P2508R1)
- [WG21 P2572R1 — std::format fill character allowances](https://wg21.link/P2572R1)
- [WG21 P2675R1 — std::format's width estimation](https://wg21.link/P2675R1)
- [WG21 P2693R1 — Formatting thread::id and stacktrace](https://wg21.link/P2693R1)
- [WG21 P2713R1 — Escaping improvements in std::format](https://wg21.link/P2713R1)
- [libc++ — Format status](https://libcxx.llvm.org/Status/Format.html)


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>