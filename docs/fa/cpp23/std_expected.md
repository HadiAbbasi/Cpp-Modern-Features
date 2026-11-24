<div align="center">

[🇺🇸 English](../../en/cpp23/std_expected.md) | [🇮🇷 فارسی](./std_expected.md)

</div>

---

# آموزش جامع std::expected در ++C23

## مقدمه

مدیریت خطا یکی از مهم‌ترین جنبه‌های طراحی نرم‌افزارهای قابل‌اعتماد است. در برنامه‌های ++C، روش‌های مختلفی برای مدیریت خطا وجود دارد؛ از جمله استفاده از Exceptionها، مقادیر بازگشتی ویژه، Pointerها، `std::optional` و الگوهای مبتنی بر Error Code.

هرکدام از این روش‌ها مزایا و محدودیت‌های خاص خود را دارند. برای مثال، استفاده از Exception امکان جدا کردن مسیر اجرای عادی از مسیر خطا را فراهم می‌کند، اما همیشه انتخاب مناسبی برای APIهایی نیست که خطا بخشی طبیعی از قرارداد آن‌ها محسوب می‌شود.

از طرف دیگر، استفاده از Error Code در کنار خروجی از طریق پارامترهای مرجع یا Pointer می‌تواند باعث پیچیده شدن رابط توابع و افزایش احتمال اشتباه شود.

برای حل این مشکل، استاندارد ++C23 نوع کتابخانه‌ای `std::expected` را معرفی کرده است.

این نوع امکان نمایش نتیجه یک عملیات را به شکلی فراهم می‌کند که مقدار موفقیت‌آمیز و اطلاعات خطا هر دو بخشی از نوع بازگشتی تابع باشند.

با استفاده از `std::expected` می‌توان:

- نتیجه موفقیت‌آمیز یک عملیات را نمایش داد.
- اطلاعات خطا را به‌صورت Type-Safe نگهداری کرد.
- خطاهای قابل‌پیش‌بینی را بدون استفاده از Exception مدیریت کرد.
- خطاها را در زنجیره‌ای از عملیات منتقل کرد.
- قرارداد توابع را از طریق Type System مشخص کرد.
- از ترکیب عملیات موفقیت‌آمیز و مدیریت خطا با Monadic Operations استفاده کرد.

در این آموزش، ساختار داخلی، نحوه استفاده، عملیات اصلی، قابلیت‌های پیشرفته، محدودیت‌ها و کاربردهای عملی این نوع بررسی می‌شود.

## معرفی std::expected

نوع `std::expected` یک Standard Library Type است که در استاندارد ++C23 و در Header زیر معرفی شده است:

```cpp
#include <expected>
```

تعریف اصلی آن به شکل زیر است:

```cpp
template<class T, class E>
class expected;
```

در این تعریف:

- `T` نوع مقداری است که در صورت موفقیت عملیات بازگردانده می‌شود.
- `E` نوع اطلاعاتی است که در صورت شکست عملیات نگهداری می‌شود.

بنابراین، یک نمونه از `std::expected<T, E>` همیشه یکی از دو وضعیت زیر را دارد:

- **Value State:** شامل یک مقدار معتبر از نوع `T` است.
- **Error State:** شامل یک مقدار خطا از نوع `E` است.

برای مثال:

```cpp
std::expected<int, std::string>
```

این نوع می‌تواند یک عدد صحیح یا یک پیام خطا از نوع `std::string` را نگهداری کند.

نکته مهم این است که `std::expected` همواره در یکی از این دو وضعیت قرار دارد و برخلاف بعضی Unionهای سنتی، وضعیت بدون مقدار یا Valueless State ندارد.

### دلیل طراحی std::expected

فرض کنید تابعی برای تبدیل یک رشته به عدد نوشته‌ایم.

این عملیات ممکن است موفق باشد یا به دلایل مختلفی شکست بخورد:

- ورودی خالی باشد.
- رشته شامل کاراکترهای غیرمجاز باشد.
- مقدار ورودی از محدوده نوع مقصد خارج باشد.

یکی از روش‌های سنتی، استفاده از یک مقدار بازگشتی ویژه است:

```cpp
int parseNumber(const std::string& input) {
    if (input.empty()) {
        return -1;
    }

    return std::stoi(input);
}
```

این روش مشکلاتی دارد:

- مقدار `-1` ممکن است یک نتیجه معتبر باشد.
- نوع بازگشتی اطلاعاتی درباره علت شکست ارائه نمی‌دهد.
- فراخواننده باید از قرارداد غیررسمی تابع اطلاع داشته باشد.
- اضافه کردن انواع مختلف خطا دشوار است.

روش دیگر، استفاده از پارامتر خروجی است:

```cpp
bool parseNumber(
    const std::string& input,
    int& result
);
```

در این روش، فراخواننده باید مقدار بازگشتی Boolean و پارامتر خروجی را هم‌زمان مدیریت کند.

با استفاده از `std::expected` می‌توان قرارداد تابع را به‌صورت دقیق‌تری تعریف کرد:

```cpp
std::expected<int, std::string>
parseNumber(const std::string& input);
```

اکنون نوع بازگشتی به‌وضوح مشخص می‌کند که تابع یا یک عدد صحیح تولید می‌کند یا اطلاعات خطا را برمی‌گرداند.

## تفاوت std::expected با std::optional

هر دو نوع `std::expected` و `std::optional` برای نمایش نتایجی استفاده می‌شوند که ممکن است مقدار موردنظر را در اختیار نداشته باشند؛ اما هدف طراحی آن‌ها متفاوت است.

نوع `std::optional<T>` در استاندارد ++C17 معرفی شده است.

این نوع دو وضعیت دارد:

- دارای مقدار است.
- فاقد مقدار است.

برای مثال:

```cpp
std::optional<int> findUserId();
```

این تابع ممکن است یک شناسه معتبر برگرداند یا نشان دهد که شناسه‌ای پیدا نشده است.

اما `std::optional` به‌تنهایی اطلاعاتی درباره علت نبود مقدار ارائه نمی‌کند.

در مقابل:

```cpp
std::expected<int, UserError> findUserId();
```

می‌تواند هم نتیجه موفقیت‌آمیز و هم علت شکست را مشخص کند.

| ویژگی | std::optional | std::expected |
|---|---|---|
| استاندارد معرفی | ++C17 | ++C23 |
| نگهداری مقدار موفقیت | دارد | دارد |
| نگهداری اطلاعات خطا | ندارد | دارد |
| تعداد وضعیت‌ها | دو | دو |
| استفاده برای خطاهای قابل‌توضیح | محدود | مناسب |
| عملیات Monadic | دارد | دارد |
| نگهداری هم‌زمان Value و Error | خیر | خیر |

### چه زمانی از std::optional استفاده کنیم؟

زمانی که نبود مقدار به‌خودی‌خود یک وضعیت معتبر است و نیازی به توضیح علت آن نداریم، `std::optional` انتخاب مناسبی است.

برای مثال:

```cpp
std::optional<User> findUser(UserId id);
```

اگر کاربر وجود نداشته باشد، بازگرداندن `std::nullopt` کاملاً معنادار است.

### چه زمانی از std::expected استفاده کنیم؟

زمانی که شکست عملیات نیازمند اطلاعات بیشتری است، بهتر است از `std::expected` استفاده کنیم.

برای مثال:

```cpp
enum class DatabaseError {
    connection_failed,
    query_failed,
    permission_denied
};

std::expected<User, DatabaseError>
loadUser(UserId id);
```

در این طراحی، فراخواننده می‌تواند بر اساس نوع خطا تصمیم متفاوتی بگیرد.

## ساختار وضعیت‌های داخلی

نوع `std::expected<T, E>` از یک فضای ذخیره‌سازی داخلی برای نگهداری یکی از دو مقدار استفاده می‌کند.

این رفتار مشابه یک Discriminated Union است؛ یعنی علاوه بر مقدار ذخیره‌شده، وضعیت فعال آن نیز مشخص است.

برای مثال:

```cpp
std::expected<int, std::string> result = 42;
```

در این وضعیت، شیء شامل مقدار `42` از نوع `int` است.

اما در مثال زیر:

```cpp
std::expected<int, std::string> result =
    std::unexpected(std::string{"Invalid input"});
```

وضعیت شیء به Error State تغییر می‌کند و پیام خطا در آن نگهداری می‌شود.

چند نکته مهم درباره این ساختار وجود دارد:

- در هر لحظه فقط یکی از دو مقدار `T` یا `E` فعال است.
- حافظه موردنیاز برای نگهداری مقدارها بخشی از خود شیء است.
- استفاده از `std::expected` به‌صورت ذاتی نیازمند Dynamic Memory Allocation نیست.
- اندازه شیء به پیاده‌سازی، Alignment و اندازه نوع‌های `T` و `E` وابسته است.
- در حالت عادی، ساختار داخلی برای نگهداری Value و Error از Heap استفاده نمی‌کند؛ البته خود نوع‌های `T` و `E` ممکن است درون خودشان حافظه پویا تخصیص دهند.

بنابراین، نباید فرض کرد که استفاده از `std::expected` همیشه بدون Allocation است؛ این موضوع به نوع‌های ذخیره‌شده نیز بستگی دارد.

## تعریف نوع خطا

یکی از مهم‌ترین تصمیم‌ها هنگام طراحی API مبتنی بر `std::expected`، انتخاب نوع مناسب برای `E` است.

این نوع باید اطلاعات کافی برای تشخیص و مدیریت خطا را فراهم کند.

### استفاده از enum class

برای مجموعه‌ای محدود و مشخص از خطاها، استفاده از `enum class` معمولاً انتخاب مناسبی است.

```cpp
enum class ParseError {
    empty_input,
    invalid_character,
    out_of_range
};

std::expected<int, ParseError>
parseInteger(const std::string& input);
```

مزایای این روش عبارت‌اند از:

- هزینه نگهداری پایین.
- مشخص بودن مجموعه خطاها.
- امکان استفاده از `switch`.
- عدم نیاز به تخصیص حافظه پویا برای خود Error Value.
- مناسب بودن برای APIهای دارای خطاهای محدود و شناخته‌شده.

### استفاده از ساختار اختصاصی

زمانی که خطا نیازمند اطلاعات بیشتری است، می‌توان یک Struct تعریف کرد.

```cpp
#include <string>

struct ParseError {
    int errorCode;
    std::size_t position;
    std::string message;
};
```

اکنون خطا علاوه بر نوع آن، اطلاعات تکمیلی نیز دارد.

برای مثال:

```cpp
std::expected<int, ParseError>
parseInteger(const std::string& input);
```

این طراحی برای Parserها، فایل‌خوان‌ها و سیستم‌هایی که نیازمند گزارش دقیق خطا هستند مفید است.

### استفاده از std::string

برای برنامه‌های کوچک یا APIهای داخلی، استفاده از `std::string` به‌عنوان Error Type امکان‌پذیر است.

```cpp
std::expected<int, std::string>
parseInteger(const std::string& input);
```

با این حال، در پروژه‌های حرفه‌ای بهتر است از Error Typeهای مشخص استفاده شود.

علت این موضوع آن است که بررسی Stringها برای تشخیص نوع خطا شکننده است و تغییر متن پیام می‌تواند منطق برنامه را تحت تأثیر قرار دهد.

### طراحی Error Typeهای سلسله‌مراتبی

در سیستم‌های بزرگ ممکن است چند زیرسیستم خطاهای متفاوتی داشته باشند.

برای مثال:

```cpp
enum class FileError {
    not_found,
    permission_denied,
    read_failed
};

enum class ParseError {
    invalid_format,
    unexpected_token
};
```

می‌توان برای هر زیرسیستم Error Type مستقل تعریف کرد و در لایه‌های بالاتر، خطاها را به یک نوع عمومی‌تر تبدیل کرد.

این روش باعث می‌شود وابستگی میان زیرسیستم‌ها کاهش پیدا کند.

## نحوه ایجاد مقدار موفقیت‌آمیز

برای ساخت یک `std::expected` در وضعیت موفقیت، کافی است مقدار موردنظر را به آن بدهیم.

```cpp
#include <expected>

std::expected<int, int> getValue() {
    return 42;
}
```

در این مثال، مقدار `42` در بخش Value ذخیره می‌شود.

همچنین می‌توان نوع بازگشتی را مستقیماً مقداردهی کرد:

```cpp
std::expected<std::string, int> getName() {
    return std::string{"Alice"};
}
```

### استفاده از std::in_place

برای ساخت مستقیم مقدار موفقیت‌آمیز داخل فضای ذخیره‌سازی `expected` می‌توان از `std::in_place` استفاده کرد.

```cpp
#include <expected>
#include <string>

std::expected<std::string, int> result{
    std::in_place,
    5,
    'A'
};
```

در این مثال، شیء `std::string` مستقیماً با پنج کاراکتر `A` ساخته می‌شود.

این روش در مواردی مفید است که ساخت مقدار مقصد از چند پارامتر انجام می‌شود.

همچنین می‌تواند از ساخت یک شیء موقت غیرضروری جلوگیری کند.

البته در بسیاری از موارد، Compiler قادر است با استفاده از Copy Elision هزینه ساخت موقت را حذف کند.

## نحوه ایجاد وضعیت خطا

برای ایجاد وضعیت خطا، از `std::unexpected` استفاده می‌شود.

این نوع یک Wrapper برای Error Value است.

ساختار کلی آن به شکل زیر است:

```cpp
std::unexpected<ErrorType>
```

برای مثال:

```cpp
#include <expected>

enum class MathError {
    division_by_zero
};

std::expected<double, MathError>
divide(double a, double b) {
    if (b == 0.0) {
        return std::unexpected(
            MathError::division_by_zero
        );
    }

    return a / b;
}
```

در این مثال، اگر مخرج صفر باشد، تابع یک مقدار خطا بازمی‌گرداند.

نکته مهم این است که `std::unexpected` خودش جایگزین `std::expected` نیست؛ بلکه برای ساخت و انتقال Error State به آن استفاده می‌شود.

### استفاده از std::unexpect

روش دیگری برای ساخت مستقیم وضعیت خطا، استفاده از Tag اختصاصی `std::unexpect` است.

```cpp
#include <expected>
#include <string>

std::expected<int, std::string> result{
    std::unexpect,
    "Invalid input"
};
```

این روش Error Value را مستقیماً داخل `expected` می‌سازد.

تفاوت کاربردی این دو روش:

- `std::unexpected` یک Wrapper برای Error Value تولید می‌کند.
- `std::unexpect` به Constructor مربوط به Error State اعلام می‌کند که باید خطا را مستقیماً درون شیء بسازد.

زمانی که ساخت مستقیم Error Value اهمیت دارد، `std::unexpect` می‌تواند مفید باشد.

## بررسی وضعیت با has_value

برای تشخیص اینکه یک شیء دارای مقدار موفقیت‌آمیز است یا خطا، از تابع `has_value` استفاده می‌شود.

```cpp
if (result.has_value()) {
    // Success
} else {
    // Failure
}
```

این تابع یک مقدار Boolean برمی‌گرداند.

همچنین می‌توان مستقیماً از خود شیء در شرط استفاده کرد.

```cpp
if (result) {
    // Success
}
```

دلیل امکان این کار وجود Explicit Conversion به `bool` در `std::expected` است.

استفاده از این قابلیت برای بررسی وضعیت بسیار رایج است.

با این حال، بررسی وضعیت به‌تنهایی مقدار را استخراج نمی‌کند.

## دسترسی به مقدار موفقیت‌آمیز

برای دسترسی به Value چند روش وجود دارد.

هرکدام از این روش‌ها رفتار متفاوتی در صورت قرار داشتن شیء در Error State دارند.

### استفاده از value

تابع `value` مقدار موفقیت‌آمیز را بازمی‌گرداند.

```cpp
auto result = getValue();

if (result.has_value()) {
    int value = result.value();
}
```

نکته مهم این است که اگر شیء در Error State باشد، فراخوانی `value` باعث پرتاب Exception از نوع زیر می‌شود:

```cpp
std::bad_expected_access<E>
```

در این Exception، اطلاعات خطا نیز قابل دسترسی است.

```cpp
try {
    int value = result.value();
} catch (const std::bad_expected_access<int>& ex) {
    int error = ex.error();
}
```

بنابراین، استفاده از `value` بدون اطمینان از وضعیت شیء ممکن است مسیر اجرای برنامه را وارد Exception Handling کند.

### استفاده از operator*

عملگر Dereference نیز برای دسترسی به مقدار موفقیت‌آمیز استفاده می‌شود.

```cpp
int value = *result;
```

این روش در صورت وجود مقدار، دسترسی مستقیم و کم‌هزینه‌ای فراهم می‌کند.

اما برخلاف `value`، در صورت قرار داشتن شیء در Error State، رفتار آن دارای Undefined Behavior است.

بنابراین، تنها زمانی از آن استفاده کنید که قبلاً وضعیت شیء را بررسی کرده باشید یا وجود Value از طریق منطق برنامه تضمین شده باشد.

### استفاده از operator->

برای دسترسی به اعضای شیء ذخیره‌شده می‌توان از `operator->` استفاده کرد.

```cpp
std::expected<std::string, int> result = "Hello";

if (result) {
    auto length = result->size();
}
```

این عملگر نیز فقط زمانی معتبر است که شیء در Value State باشد.

### استفاده از value_or

تابع `value_or` امکان تعیین یک مقدار پیش‌فرض را فراهم می‌کند.

```cpp
std::expected<int, std::string> result =
    std::unexpected("Calculation failed");

int value = result.value_or(0);
```

در این مثال، چون نتیجه در وضعیت خطا قرار دارد، مقدار `0` بازگردانده می‌شود.

این تابع در شرایطی مناسب است که داشتن یک مقدار جایگزین منطقی باشد.

اما باید توجه داشت که `value_or` اطلاعات خطا را حذف می‌کند.

همچنین پارامتر پیش‌فرض آن در زمان فراخوانی ارزیابی می‌شود؛ بنابراین حتی اگر `expected` دارای مقدار باشد، عبارت ارسال‌شده به `value_or` از قبل ارزیابی شده است.

برای عملیات پرهزینه، این نکته اهمیت دارد.

## دسترسی به اطلاعات خطا

برای استخراج Error Value از تابع `error` استفاده می‌شود.

```cpp
if (!result) {
    auto error = result.error();
}
```

این تابع اطلاعات خطا را برمی‌گرداند.

نکته مهم این است که فراخوانی `error` زمانی معتبر است که شیء در Error State باشد.

در صورت قرار داشتن شیء در Value State، رفتار آن مطابق قرارداد پیش‌شرط تابع نیست.

بنابراین، قبل از استفاده از آن باید وضعیت شیء بررسی شود.

### تفاوت error و value

دو تابع `value` و `error` برای استخراج دو وضعیت متفاوت طراحی شده‌اند.

| تابع | وضعیت معتبر | نتیجه |
|---|---|---|
| `value()` | Value State | مقدار `T` |
| `error()` | Error State | مقدار `E` |
| `value_or()` | هر دو وضعیت | مقدار `T` یا مقدار پیش‌فرض |

## مدیریت خطا در توابع

یکی از مهم‌ترین کاربردهای `std::expected` طراحی توابعی است که خطاهای قابل‌پیش‌بینی دارند.

برای درک بهتر، یک تابع تبدیل رشته به عدد را پیاده‌سازی می‌کنیم.

```cpp
#include <charconv>
#include <expected>
#include <string_view>
#include <system_error>

enum class ParseError {
    empty_input,
    invalid_input,
    out_of_range
};

std::expected<int, ParseError>
parseInteger(std::string_view input) {
    if (input.empty()) {
        return std::unexpected(
            ParseError::empty_input
        );
    }

    int value{};

    const char* begin = input.data();
    const char* end = begin + input.size();

    auto [ptr, ec] = std::from_chars(
        begin,
        end,
        value
    );

    if (ec == std::errc::result_out_of_range) {
        return std::unexpected(
            ParseError::out_of_range
        );
    }

    if (ec != std::errc{} || ptr != end) {
        return std::unexpected(
            ParseError::invalid_input
        );
    }

    return value;
}
```

در این پیاده‌سازی از `std::from_chars` استفاده شده است.

این تابع برای تبدیل رشته به عدد طراحی شده و برخلاف برخی روش‌های قدیمی‌تر، Exception پرتاب نمی‌کند.

همچنین می‌توان نتیجه آن را با `std::expected` ترکیب کرد تا قرارداد تابع سطح بالاتر نیز به‌صورت Type-Safe مشخص شود.

نکته مهم این است که `std::from_chars` ورودی را الزاماً به‌صورت کامل مصرف نمی‌کند؛ به همین دلیل در مثال بالا مقدار `ptr` نیز بررسی شده است.

برای مثال، ورودی `123abc` نباید به‌عنوان عدد `123` پذیرفته شود.

## استفاده از std::expected<void, E>

در بسیاری از توابع، عملیات موفقیت‌آمیز نیازی به بازگرداندن مقدار ندارد.

برای مثال:

- ذخیره اطلاعات در فایل.
- حذف یک رکورد از Database.
- ارسال یک پیام.
- اعتبارسنجی یک شیء.
- اجرای یک عملیات شبکه.

در چنین شرایطی می‌توان از Specialization زیر استفاده کرد:

```cpp
std::expected<void, E>
```

این نوع تنها دو وضعیت دارد:

- عملیات موفق بوده است.
- عملیات با خطا مواجه شده است.

برای مثال:

```cpp
#include <expected>
#include <string>

std::expected<void, std::string>
saveFile() {
    bool success = true;

    if (!success) {
        return std::unexpected(
            "Unable to save file"
        );
    }

    return {};
}
```

در این مثال، `return {};` یک `std::expected<void, std::string>` موفق ایجاد می‌کند.

برای بررسی نتیجه نیز همان روش معمول استفاده می‌شود:

```cpp
auto result = saveFile();

if (!result) {
    // Handle failure
}
```

در Specialization مربوط به `void`، تابع `value` همچنان وجود دارد، اما مقدار قابل‌توجهی برنمی‌گرداند.

همچنین `operator*` و `operator->` برای این Specialization وجود ندارند.

## مدیریت زنجیره‌ای عملیات با Monadic Operations

یکی از قابلیت‌های مهم `std::expected` در ++C23، مجموعه Monadic Operations است.

این عملیات امکان ترکیب چند تابع را بدون نوشتن مکرر شرط‌های بررسی خطا فراهم می‌کنند.

چهار تابع اصلی عبارت‌اند از:

- `and_then`
- `transform`
- `or_else`
- `transform_error`

این قابلیت‌ها بر اساس Proposal شماره P2505R5 به استاندارد ++C23 اضافه شده‌اند.

### مفهوم and_then

تابع `and_then` زمانی Callback را اجرا می‌کند که شیء دارای Value باشد.

اگر شیء در Error State باشد، Callback اجرا نمی‌شود و خطا منتقل می‌شود.

فرض کنید دو تابع داریم:

```cpp
std::expected<int, ParseError>
parseInteger(std::string_view input);

std::expected<double, MathError>
calculateSomething(int value);
```

برای ترکیب این دو تابع باید نوع Error آن‌ها نیز سازگار باشد.

در ساده‌ترین حالت، اگر هر دو تابع از Error Type یکسان استفاده کنند، می‌توان زنجیره را به شکل زیر ساخت:

```cpp
auto result = parseInteger("42")
    .and_then(calculateSomething);
```

در این مثال:

- اگر تبدیل رشته موفق باشد، تابع دوم اجرا می‌شود.
- اگر تبدیل رشته شکست بخورد، تابع دوم اجرا نمی‌شود.
- خطا بدون نیاز به شرط اضافی منتقل می‌شود.

نوع بازگشتی Callback در `and_then` باید یک `std::expected` سازگار باشد.

به بیان دقیق‌تر، این عملیات برای ترکیب توابعی مناسب است که خودشان امکان شکست دارند.

### مثال عملی and_then

```cpp
#include <expected>
#include <string>

enum class Error {
    invalid_input,
    invalid_operation
};

std::expected<int, Error>
parseValue(const std::string& input) {
    if (input.empty()) {
        return std::unexpected(
            Error::invalid_input
        );
    }

    return 10;
}

std::expected<int, Error>
multiplyByTwo(int value) {
    if (value < 0) {
        return std::unexpected(
            Error::invalid_operation
        );
    }

    return value * 2;
}

int main() {
    auto result = parseValue("123")
        .and_then(multiplyByTwo);

    if (result) {
        int value = *result;
    }
}
```

این روش باعث می‌شود مسیر موفقیت به‌صورت زنجیره‌ای نوشته شود و انتقال خطا به‌صورت خودکار انجام شود.

### مفهوم transform

تابع `transform` زمانی استفاده می‌شود که بخواهیم مقدار موفقیت‌آمیز را تغییر دهیم، اما عملیات جدید امکان شکست مستقل ندارد.

برای مثال:

```cpp
std::expected<int, Error> result = 10;

auto transformed = result.transform(
    [](int value) {
        return value * 2;
    }
);
```

نتیجه یک `std::expected<int, Error>` جدید است.

اگر مقدار اولیه موفق باشد، Callback اجرا می‌شود و مقدار جدید داخل `expected` قرار می‌گیرد.

اما اگر مقدار اولیه خطا باشد، خطا بدون تغییر منتقل می‌شود.

تفاوت اصلی `transform` و `and_then` در نوع خروجی Callback است:

- در `transform`، Callback یک مقدار عادی برمی‌گرداند.
- در `and_then`، Callback یک `std::expected` برمی‌گرداند.

این تفاوت در طراحی زنجیره عملیات اهمیت زیادی دارد.

### مثال کاربردی transform

```cpp
#include <expected>
#include <string>

enum class Error {
    invalid_input
};

std::expected<int, Error>
parseValue(const std::string& input) {
    if (input.empty()) {
        return std::unexpected(Error::invalid_input);
    }

    return 21;
}

int main() {
    auto result = parseValue("21")
        .transform([](int value) {
            return value * 2;
        });

    if (result) {
        int value = *result;
    }
}
```

در این مثال، عملیات تبدیل رشته ممکن است شکست بخورد، اما ضرب عدد در دو یک عملیات موفقیت‌آمیز عادی است.

بنابراین `transform` انتخاب مناسبی است.

### مفهوم or_else

تابع `or_else` زمانی اجرا می‌شود که شیء در Error State باشد.

این تابع برای بازیابی از خطا، اجرای عملیات جایگزین یا تبدیل مسیر شکست به یک نتیجه دیگر استفاده می‌شود.

برای مثال:

```cpp
auto result = getConfiguration()
    .or_else([](const ConfigError& error) {
        return loadDefaultConfiguration();
    });
```

اگر دریافت تنظیمات موفق باشد، Callback اجرا نمی‌شود.

در صورت شکست، تابع جایگزین اجرا خواهد شد.

نکته مهم این است که خروجی Callback در `or_else` باید یک `std::expected` سازگار باشد و نوع Value آن با نوع Value نتیجه اولیه سازگار بماند.

همچنین نوع Error خروجی باید با الزامات نوع بازگشتی حاصل از عملیات سازگار باشد.

### مفهوم transform_error

تابع `transform_error` برای تغییر Error Value استفاده می‌شود، بدون اینکه Value موفقیت‌آمیز تغییر کند.

برای مثال:

```cpp
enum class InternalError {
    invalid_input,
    timeout
};

enum class PublicError {
    bad_request,
    service_unavailable
};

std::expected<int, InternalError> internalOperation();

std::expected<int, PublicError>
publicOperation() {
    return internalOperation()
        .transform_error(
            [](InternalError error) {
                switch (error) {
                    case InternalError::invalid_input:
                        return PublicError::bad_request;

                    case InternalError::timeout:
                        return PublicError::service_unavailable;
                }

                return PublicError::bad_request;
            }
        );
}
```

در این مثال، خطاهای داخلی به Error Type مناسب برای API عمومی تبدیل می‌شوند.

این روش برای طراحی معماری چندلایه بسیار کاربردی است.

### مقایسه عملیات Monadic

| عملیات | زمان اجرای Callback | خروجی Callback | کاربرد |
|---|---|---|---|
| `and_then` | در صورت موفقیت | `expected` | زنجیره عملیات خطاپذیر |
| `transform` | در صورت موفقیت | مقدار عادی | تبدیل Value |
| `or_else` | در صورت شکست | `expected` | بازیابی یا جایگزینی |
| `transform_error` | در صورت شکست | Error Value | تبدیل نوع خطا |

## مدیریت چندین مرحله خطاپذیر

در پروژه‌های واقعی، معمولاً یک عملیات از چند مرحله تشکیل می‌شود.

برای مثال، پردازش یک درخواست ممکن است شامل مراحل زیر باشد:

- دریافت اطلاعات.
- اعتبارسنجی.
- تبدیل داده‌ها.
- اجرای عملیات اصلی.
- ذخیره نتیجه.

اگر هر مرحله بتواند شکست بخورد، استفاده از `std::expected` همراه با Monadic Operations می‌تواند ساختار برنامه را ساده‌تر کند.

برای نمونه:

```cpp
#include <expected>
#include <string>

enum class Error {
    invalid_input,
    invalid_format,
    processing_failed
};

std::expected<std::string, Error>
readInput();

std::expected<int, Error>
parseInput(const std::string& input);

std::expected<int, Error>
processValue(int value);

std::expected<std::string, Error>
formatResult(int value);

std::expected<std::string, Error>
execute() {
    return readInput()
        .and_then(parseInput)
        .and_then(processValue)
        .and_then(formatResult);
}
```

در این ساختار، هر مرحله فقط مسئول عملیات خودش است.

اگر یکی از مراحل شکست بخورد، مراحل بعدی اجرا نمی‌شوند و Error Value به انتهای زنجیره منتقل می‌شود.

این طراحی باعث کاهش Nesting و حذف بخش زیادی از کدهای تکراری مربوط به بررسی خطا می‌شود.

## مدیریت زنجیره با انواع مختلف خطا

یکی از نکات مهم در طراحی زنجیره عملیات، یکسان بودن نوع خطا یا امکان تبدیل آن است.

فرض کنید تابع اول چنین نوعی دارد:

```cpp
std::expected<int, ParseError>
```

و تابع دوم:

```cpp
std::expected<double, DatabaseError>
```

نمی‌توان این دو تابع را مستقیماً با `and_then` به یک زنجیره تبدیل کرد؛ زیرا Error Typeها با یکدیگر سازگار نیستند.

برای حل این مشکل چند روش وجود دارد.

### روش اول: استفاده از Error Type مشترک

می‌توان یک نوع خطای عمومی برای چند زیرسیستم تعریف کرد.

```cpp
enum class ApplicationError {
    parse_failed,
    database_failed,
    network_failed
};
```

سپس هر تابع خطای اختصاصی خود را به این نوع تبدیل کند.

این روش برای سیستم‌هایی که مجموعه خطاهای محدودی دارند مناسب است.

### روش دوم: استفاده از std::variant

در صورتی که بخواهیم نوع اصلی خطاهای مختلف را حفظ کنیم، می‌توان از `std::variant` استفاده کرد.

```cpp
#include <expected>
#include <variant>

using ApplicationError = std::variant<
    ParseError,
    DatabaseError,
    NetworkError
>;
```

اکنون تمام توابع می‌توانند یک Error Type مشترک داشته باشند، در حالی که اطلاعات نوع اصلی خطا حفظ می‌شود.

### روش سوم: تبدیل خطا در مرز زیرسیستم‌ها

می‌توان Error Typeهای داخلی را تا حد امکان مستقل نگه داشت و فقط هنگام عبور از مرز یک لایه، آن‌ها را به خطای عمومی‌تر تبدیل کرد.

برای این کار `transform_error` ابزار مناسبی است.

این روش معمولاً در معماری‌های بزرگ، استقلال زیرسیستم‌ها را بهتر حفظ می‌کند.

## بررسی Constructorها و Assignment

نوع `std::expected` Constructorهای مختلفی برای ساخت Value State و Error State ارائه می‌کند.

شناخت آن‌ها برای استفاده حرفه‌ای اهمیت دارد.

### Default Constructor

در صورتی که `T` قابل Default Construction باشد، می‌توان یک `expected` بدون مقدار اولیه صریح ساخت.

```cpp
std::expected<int, std::string> result;
```

در این حالت، Value پیش‌فرض نوع `int` ساخته می‌شود.

بنابراین نتیجه برابر با یک `expected` موفق شامل صفر خواهد بود.

این Constructor برای `std::expected<void, E>` نیز وجود دارد و یک وضعیت موفق ایجاد می‌کند.

### Copy Constructor

اگر نوع‌های ذخیره‌شده الزامات Copy Construction را داشته باشند، می‌توان یک `expected` را کپی کرد.

```cpp
auto first = std::expected<int, std::string>{42};

auto second = first;
```

در این مثال، مقدار موفقیت‌آمیز کپی می‌شود.

اگر شیء اولیه در Error State باشد، Error Value کپی خواهد شد.

### Move Constructor

برای انتقال مالکیت منابع از یک شیء به شیء دیگر می‌توان از Move Construction استفاده کرد.

```cpp
auto first =
    std::expected<std::string, std::string>{
        "Hello"
    };

auto second = std::move(first);
```

این عملیات در صورت پشتیبانی نوع‌های داخلی از Move Construction، امکان انتقال منابع را فراهم می‌کند.

توجه داشته باشید که شیء مبدأ همچنان یک `expected` معتبر است و وضعیت Value یا Error آن به‌صورت خودکار به Empty تبدیل نمی‌شود.

وضعیت دقیق مقدار ذخیره‌شده پس از Move به رفتار Move Constructor نوع مربوطه بستگی دارد.

### استفاده از std::in_place

همان‌طور که قبلاً اشاره شد، این Tag برای ساخت مستقیم Value استفاده می‌شود.

```cpp
std::expected<std::string, int> result{
    std::in_place,
    5,
    'X'
};
```

### استفاده از std::unexpect

برای ساخت مستقیم Error Value می‌توان از این Constructor استفاده کرد.

```cpp
std::expected<std::string, int> result{
    std::unexpect,
    404
};
```

### عملگر Assignment

نوع `std::expected` از Assignment نیز پشتیبانی می‌کند.

```cpp
std::expected<int, std::string> result = 10;

result = 20;

result = std::unexpected(
    std::string{"Failed"}
);
```

در این مثال، شیء ابتدا دارای مقدار موفقیت‌آمیز است، سپس مقدار آن تغییر می‌کند و در نهایت وارد Error State می‌شود.

امکان استفاده از Assignment به قابلیت‌های موردنیاز نوع‌های `T` و `E` وابسته است.

## تغییر مقدار با emplace

تابع `emplace` برای جایگزین کردن Value موجود با یک Value جدید استفاده می‌شود.

```cpp
std::expected<std::string, int> result = "Hello";

result.emplace(5, 'A');
```

اکنون مقدار ذخیره‌شده برابر با رشته `AAAAA` است.

این تابع برای ساخت مستقیم Value جدید در فضای ذخیره‌سازی داخلی مفید است.

اگر شیء در Error State باشد، مقدار خطا جایگزین می‌شود و Value جدید ساخته خواهد شد.

نکته مهم این است که اگر ساخت Value جدید با Exception مواجه شود، شیء `expected` همچنان باید در یکی از دو وضعیت معتبر Value یا Error قرار داشته باشد.

در این شرایط، تضمین Exception Safety تابع و رفتار آن باید با توجه به مشخصات استاندارد و قابلیت‌های نوع‌های مورد استفاده بررسی شود.

## جابه‌جایی وضعیت‌ها با swap

تابع `swap` امکان جابه‌جایی محتویات دو شیء `expected` را فراهم می‌کند.

```cpp
std::expected<int, std::string> first = 10;

std::expected<int, std::string> second =
    std::unexpected("Error");

first.swap(second);
```

پس از این عملیات:

- `first` شامل Error Value خواهد بود.
- `second` شامل مقدار `10` خواهد بود.

این تابع برای جابه‌جایی وضعیت‌ها طراحی شده است و شرایط قابل‌استفاده بودن آن به قابلیت‌های Swap نوع‌های `T` و `E` وابسته است.

## استفاده از Reference و Pointer

یکی از محدودیت‌های مهم `std::expected` این است که نمی‌توان نوع‌های Reference را مستقیماً به‌عنوان `T` یا `E` استفاده کرد.

برای مثال، تعریف زیر معتبر نیست:

```cpp
std::expected<int&, Error> result;
```

زیرا `std::expected` برای نگهداری مستقیم Reference طراحی نشده است.

اگر لازم باشد یک Reference را به‌عنوان Value نگهداری کنیم، می‌توان از `std::reference_wrapper` استفاده کرد.

```cpp
#include <expected>
#include <functional>

int value = 42;

std::expected<std::reference_wrapper<int>, int>
getReference() {
    return std::ref(value);
}
```

در این مثال، Value از نوع `std::reference_wrapper<int>` است.

باید توجه داشت که `std::reference_wrapper` مالک شیء اصلی نیست و طول عمر شیء مرجع‌شده همچنان باید توسط برنامه مدیریت شود.

### استفاده از Pointer

استفاده از Pointer به‌عنوان Value مجاز است.

```cpp
std::expected<int*, Error> result;
```

اما باید میان دو وضعیت تفاوت قائل شد:

- `expected` در وضعیت خطا قرار دارد.
- `expected` موفق است، اما Pointer ذخیره‌شده برابر `nullptr` است.

این دو وضعیت از نظر Type System متفاوت هستند.

با این حال، اگر `nullptr` خودش معنای شکست یا نبود مقدار را دارد، ترکیب Pointer با `expected` ممکن است باعث ایجاد وضعیت‌های اضافی و مبهم شود.

در چنین شرایطی بهتر است قرارداد API دوباره بررسی شود.

## ترکیب std::expected و std::optional

این دو نوع می‌توانند در کنار یکدیگر استفاده شوند.

برای مثال، فرض کنید تابعی ممکن است به دلیل خطا شکست بخورد، اما در صورت موفقیت نیز ممکن است داده‌ای پیدا نکند.

در چنین شرایطی می‌توان از ترکیب زیر استفاده کرد:

```cpp
std::expected<std::optional<User>, DatabaseError>
findUser(UserId id);
```

این نوع سه وضعیت منطقی ایجاد می‌کند:

- خطای Database.
- اجرای موفق بدون پیدا شدن User.
- اجرای موفق همراه با User.

این طراحی در مواردی مفید است که نبود مقدار با شکست عملیات تفاوت معنایی دارد.

البته در طراحی API باید بررسی شود که آیا واقعاً به هر سه وضعیت نیاز داریم یا می‌توان قرارداد ساده‌تری تعریف کرد.

## مقایسه std::expected با Exception

هر دو روش برای انتقال اطلاعات شکست از یک تابع به فراخواننده استفاده می‌شوند، اما فلسفه طراحی متفاوتی دارند.

### مدیریت خطا با Exception

در مدل Exception، تابع در صورت شکست یک Exception پرتاب می‌کند.

```cpp
int divide(int a, int b) {
    if (b == 0) {
        throw std::runtime_error(
            "Division by zero"
        );
    }

    return a / b;
}
```

فراخواننده باید در صورت نیاز از `try` و `catch` استفاده کند.

مزیت این روش، جدا شدن مسیر عادی اجرا از مسیر خطاست.

اما انتقال Exception در مسیر فراخوانی همیشه به‌صورت صریح در امضای تابع مشخص نیست.

### مدیریت خطا با std::expected

در مدل `expected`، خطا بخشی از نوع بازگشتی تابع است.

```cpp
std::expected<int, MathError>
divide(int a, int b);
```

در این حالت، فراخواننده می‌تواند از روی Signature متوجه شود که تابع ممکن است شکست بخورد.

### مقایسه کاربردی

| معیار | std::expected | Exception |
|---|---|---|
| مشخص بودن خطا در Signature | بله | معمولاً خیر |
| مناسب برای خطاهای قابل‌پیش‌بینی | بسیار مناسب | قابل استفاده |
| نیاز به بررسی نتیجه | معمولاً بله | خیر |
| امکان انتقال خطا در زنجیره | بله | از طریق Stack Unwinding |
| نیاز به طراحی Error Type | بله | وابسته به طراحی |
| هزینه در مسیر موفق | معمولاً قابل‌پیش‌بینی | وابسته به پیاده‌سازی |
| مناسب برای خطاهای استثنایی | معمولاً خیر | بله |

### آیا std::expected جایگزین Exception است؟

خیر.

این دو ابزار برای سناریوهای متفاوتی طراحی شده‌اند.

`std::expected` معمولاً برای خطاهایی مناسب است که:

- بخشی از رفتار عادی سیستم هستند.
- قابل پیش‌بینی هستند.
- فراخواننده می‌تواند برای آن‌ها تصمیم مشخصی بگیرد.
- لازم است در قرارداد API نمایش داده شوند.

Exception برای شرایطی مناسب است که:

- عملیات نمی‌تواند طبق قرارداد خود ادامه پیدا کند.
- خطا در لایه فعلی قابل مدیریت نیست.
- شکست واقعاً استثنایی است.
- استفاده از Error Return باعث پیچیدگی غیرضروری می‌شود.

همچنین این دو روش می‌توانند در یک برنامه با یکدیگر استفاده شوند.

برای مثال، یک تابع می‌تواند خطاهای عادی را با `std::expected` گزارش کند، اما همچنان ممکن است عملیات داخلی آن به دلیل خطای غیرمنتظره Exception پرتاب کند.

نکته مهم این است که `std::expected` به‌صورت خودکار Exceptionهای پرتاب‌شده توسط توابع داخلی را به Error State تبدیل نمی‌کند.

## مقایسه std::expected با std::variant

نوع `std::variant` در استاندارد ++C17 معرفی شده است.

این نوع امکان نگهداری یکی از چند نوع مشخص را فراهم می‌کند.

برای مثال:

```cpp
std::variant<int, std::string> result;
```

این نوع می‌تواند یک عدد صحیح یا یک رشته نگهداری کند.

از نظر ساختاری، `std::expected<T, E>` شباهت زیادی به یک Variant دوحالته دارد، اما معنای آن مشخص‌تر است.

در `std::expected`:

- یک نوع برای نتیجه موفقیت‌آمیز در نظر گرفته شده است.
- نوع دیگر مخصوص خطاست.
- APIهای اختصاصی برای مدیریت خطا وجود دارند.
- Monadic Operations متناسب با جریان موفقیت و شکست ارائه شده‌اند.

در مقابل، `std::variant` برای نمایش مجموعه‌ای از حالت‌های جایگزین عمومی‌تر طراحی شده است.

بنابراین، اگر هدف صرفاً نگهداری یکی از چند نوع مستقل باشد، `std::variant` انتخاب مناسب‌تری است.

اما اگر هدف نمایش نتیجه یک عملیات همراه با احتمال شکست باشد، `std::expected` معنای دقیق‌تری دارد.

## طراحی APIهای حرفه‌ای با std::expected

استفاده صحیح از `std::expected` تنها به انتخاب این نوع به‌عنوان Return Type محدود نمی‌شود.

طراحی Error Type، مرزبندی زیرسیستم‌ها و مشخص کردن مسئولیت مدیریت خطا نیز اهمیت زیادی دارد.

### انتخاب Error Type مناسب

نوع خطا باید به‌اندازه کافی اطلاعات داشته باشد، اما نباید بیش از حد پیچیده شود.

برای مثال، استفاده از یک String عمومی ممکن است در پروژه‌های کوچک مناسب باشد، اما در APIهای عمومی معمولاً بهتر است Error Type ساختاریافته باشد.

```cpp
struct NetworkError {
    int statusCode;
    std::string message;
};
```

### جلوگیری از نشت جزئیات داخلی

فرض کنید یک Library از Database خاصی استفاده می‌کند.

اگر Error Type عمومی Library مستقیماً نوع خطای Database را در معرض مصرف‌کنندگان قرار دهد، وابستگی غیرضروری ایجاد می‌شود.

بهتر است خطاهای داخلی در مرز Library به Error Type مستقل تبدیل شوند.

### حفظ اطلاعات خطا

در هنگام تبدیل خطاها نباید اطلاعات مهم از بین برود.

برای مثال، اگر خطای Database شامل کد خطا، Query و پیام اصلی باشد، تبدیل آن به یک مقدار عمومی مانند `operation_failed` ممکن است اطلاعات موردنیاز برای Debugging را حذف کند.

بهتر است Error Type عمومی علاوه بر دسته‌بندی خطا، اطلاعات تشخیصی ضروری را نیز نگهداری کند.

### جداسازی Logging از Error Handling

بازگرداندن Error Value لزوماً به معنای ثبت Log نیست.

در بسیاری از معماری‌ها بهتر است تابع مسئول گزارش نتیجه باشد و لایه بالاتر تصمیم بگیرد که آیا خطا باید Log شود یا خیر.

این موضوع از ثبت چندباره یک خطا در لایه‌های مختلف جلوگیری می‌کند.

### طراحی خطاهای قابل توسعه

در Libraryهای عمومی باید در نظر داشت که اضافه کردن یک عضو جدید به `enum class` ممکن است رفتار مصرف‌کنندگان را تحت تأثیر قرار دهد.

به‌خصوص اگر مصرف‌کننده‌ها از `switch` استفاده کرده باشند.

بنابراین، سیاست توسعه Error Type باید از ابتدا مشخص شود.

## مدیریت خطاهای سیستم با std::error_code

کتابخانه استاندارد ++C از طریق Header زیر امکاناتی برای مدیریت خطاهای سیستم ارائه می‌کند:

```cpp
#include <system_error>
```

دو نوع مهم در این حوزه عبارت‌اند از:

- `std::error_code`
- `std::error_condition`

این نوع‌ها برای نمایش خطاهای استانداردشده و سیستم‌محور طراحی شده‌اند.

می‌توان آن‌ها را با `std::expected` ترکیب کرد.

```cpp
#include <expected>
#include <system_error>

std::expected<int, std::error_code>
readData();
```

این طراحی برای APIهایی مناسب است که از خطاهای سیستم‌عامل یا کتابخانه‌های سطح پایین استفاده می‌کنند.

مزیت `std::error_code` این است که علاوه بر مقدار خطا، Category مربوط به آن را نیز مشخص می‌کند.

بنابراین، برخلاف یک عدد صحیح ساده، امکان تشخیص منشأ و معنای خطا وجود دارد.

## بررسی قابلیت constexpr

یکی از ویژگی‌های مهم `std::expected` پشتیبانی از `constexpr` در بخش بزرگی از API آن است.

این قابلیت اجازه می‌دهد در صورت مناسب بودن نوع‌های `T` و `E`، عملیات مربوط به `expected` در زمان Compile انجام شوند.

برای مثال:

```cpp
#include <expected>

constexpr std::expected<int, int>
calculate(int value) {
    if (value < 0) {
        return std::unexpected(-1);
    }

    return value * 2;
}

constexpr auto result = calculate(10);

static_assert(result.has_value());
static_assert(*result == 20);
```

در این مثال، نتیجه در زمان Compile قابل محاسبه است.

البته استفاده از `constexpr` به محدودیت‌های مربوط به نوع‌های ذخیره‌شده و عملیات مورد استفاده وابسته است.

## نکات مربوط به Move Semantics

نوع `std::expected` از Move Construction و Move Assignment پشتیبانی می‌کند.

این ویژگی برای نگهداری نوع‌های غیرقابل‌کپی یا دارای منابع مهم است.

برای مثال:

```cpp
#include <expected>
#include <memory>

enum class Error {
    allocation_failed
};

std::expected<std::unique_ptr<int>, Error>
createValue() {
    return std::make_unique<int>(42);
}
```

در این مثال، Value از نوع `std::unique_ptr<int>` است که قابل کپی نیست.

با این حال، می‌توان آن را داخل `std::expected` نگهداری و منتقل کرد.

همچنین عملیات Monadic از دسته‌های مختلف Value Category پشتیبانی می‌کنند.

به‌طور کلی، نسخه‌های مختلف این عملیات برای اشیای Lvalue و Rvalue وجود دارند.

این موضوع به آن‌ها اجازه می‌دهد در شرایط مناسب، Value را با Reference یا Rvalue Reference به Callback منتقل کنند.

بنابراین، هنگام استفاده از انواع Move-Only باید به نوع Expression و نحوه انتقال شیء توجه کرد.

## نکات مهم درباره Reference Qualifiers

عملیات دسترسی و Monadic در `std::expected` برای حالت‌های مختلف `const` و Value Category طراحی شده‌اند.

برای مثال، تابع `value` نسخه‌های مختلفی برای حالت‌های زیر دارد:

- Lvalue
- Const Lvalue
- Rvalue
- Const Rvalue

این موضوع امکان استفاده صحیح از Move Semantics را فراهم می‌کند.

برای نمونه:

```cpp
std::expected<std::string, int> result = "Hello";

std::string value = std::move(result).value();
```

در این مثال، Value از شیء `expected` به رشته جدید منتقل می‌شود.

اما باید توجه داشت که `std::move` خودش انتقال انجام نمی‌دهد؛ تنها نوع Expression را به Rvalue تبدیل می‌کند و عملیات انتقال توسط Constructor یا Assignment مناسب انجام می‌شود.

همچنین استفاده از شیء منتقل‌شده باید با توجه به وضعیت پس از Move انجام شود.

## محدودیت‌ها و الزامات نوع‌ها

نوع `std::expected` محدودیت‌هایی درباره نوع‌های `T` و `E` دارد.

شناخت این محدودیت‌ها برای جلوگیری از خطاهای Compile-Time ضروری است.

### عدم پشتیبانی از Reference Type

استفاده مستقیم از Reference به‌عنوان Value یا Error مجاز نیست.

```cpp
std::expected<int&, Error> result;
```

این تعریف نامعتبر است.

برای نگهداری Reference باید از Wrapper مناسب استفاده کرد.

### عدم پشتیبانی از Function Type

Function Type نیز نمی‌تواند مستقیماً به‌عنوان `T` یا `E` استفاده شود.

در صورت نیاز باید از Pointer یا Wrapper مناسب استفاده کرد.

### محدودیت‌های مربوط به Typeهای خاص

نوع `T` نباید از نوع‌های زیر باشد:

- `std::unexpected`
- `std::in_place_t`
- `std::unexpect_t`

همچنین نوع‌های مورد استفاده باید الزامات مربوط به Object Type و Destructibility را رعایت کنند.

### الزامات Constructor و Assignment

تمام عملیات برای همه نوع‌ها قابل استفاده نیستند.

برای مثال:

- Copy Constructor به قابلیت Copy Construction وابسته است.
- Move Constructor به قابلیت Move Construction وابسته است.
- Assignment به قابلیت‌های Assignment نوع‌های داخلی وابسته است.
- `emplace` به قابلیت ساخت Value جدید وابسته است.
- `swap` به الزامات Swap نوع‌های داخلی وابسته است.

به همین دلیل، هنگام استفاده از نوع‌های سفارشی پیچیده باید محدودیت‌های هر عملیات بررسی شود.

## اشتباهات رایج در استفاده از std::expected

### دسترسی بدون بررسی وضعیت

یکی از رایج‌ترین اشتباهات، استفاده از Dereference بدون بررسی وجود Value است.

کد زیر خطرناک است:

```cpp
std::expected<int, Error> result =
    std::unexpected(Error::invalid_input);

int value = *result;
```

در این وضعیت، برنامه دارای Undefined Behavior خواهد بود.

روش صحیح:

```cpp
if (result) {
    int value = *result;
}
```

### استفاده از value برای تمام شرایط

تابع `value` در Error State یک Exception پرتاب می‌کند.

اگر هدف مدیریت خطا بدون Exception است، بهتر است ابتدا وضعیت بررسی شود یا از عملیات Monadic استفاده شود.

### نادیده گرفتن Error Value

گاهی برنامه‌نویس تنها وضعیت موفقیت را بررسی می‌کند و اطلاعات خطا را نادیده می‌گیرد.

برای مثال:

```cpp
if (!result) {
    return;
}
```

این کد ممکن است از نظر منطقی معتبر باشد، اما در بسیاری از برنامه‌ها اطلاعات مهم خطا را از دست می‌دهد.

بهتر است در صورت نیاز، Error Value به لایه مناسب منتقل شود.

### استفاده از Error Type مبهم

استفاده از نوع‌هایی مانند `int` یا `std::string` برای تمام خطاها ممکن است در ابتدا ساده به نظر برسد.

اما با بزرگ‌تر شدن پروژه، تشخیص نوع خطا دشوار می‌شود.

استفاده از Error Type اختصاصی معمولاً انتخاب بهتری است.

### ترکیب نادرست and_then و transform

این دو عملیات رفتار متفاوتی دارند.

اگر Callback یک `std::expected` برگرداند، معمولاً باید از `and_then` استفاده شود.

اگر Callback یک مقدار عادی برگرداند، `transform` مناسب است.

استفاده نادرست از آن‌ها می‌تواند باعث ایجاد ساختارهای تو در تو یا خطاهای Type Compatibility شود.

### استفاده از value_or برای خطاهای مهم

تابع `value_or` خطا را به مقدار پیش‌فرض تبدیل می‌کند.

اگر خطا اهمیت عملیاتی دارد، این کار ممکن است باعث پنهان شدن مشکلات شود.

برای مثال، بازگرداندن صفر در زمان شکست یک محاسبه ممکن است نتیجه‌ای ظاهراً معتبر اما اشتباه ایجاد کند.

### فرض مالکیت منابع خارجی

استفاده از `std::expected` به این معنا نیست که همه منابع مورد استفاده توسط Value یا Error مدیریت می‌شوند.

اگر Value شامل Pointer یا Reference Wrapper باشد، طول عمر منبع خارجی همچنان باید مدیریت شود.

## نکات کارایی و Performance

از نظر کارایی، `std::expected` معمولاً گزینه مناسبی برای مدیریت خطاهای صریح است.

با این حال، نباید فرض کرد که استفاده از آن در تمام شرایط رایگان است.

### هزینه حافظه

هر شیء `expected` باید فضای کافی برای نگهداری یکی از دو نوع `T` و `E` و همچنین اطلاعات وضعیت را داشته باشد.

بنابراین، اندازه آن معمولاً تحت تأثیر بزرگ‌ترین نوع ذخیره‌شده و نیازهای Alignment قرار می‌گیرد.

### هزینه Copy و Move

اگر `T` یا `E` اشیای بزرگی باشند، کپی کردن `expected` ممکن است هزینه‌بر باشد.

در چنین شرایطی بهتر است از Move Semantics استفاده شود.

همچنین می‌توان Error Typeهای کوچک و مناسب طراحی کرد.

### هزینه Error Handling

در مسیر موفقیت، عملیات معمولاً شامل ساخت Value و نگهداری وضعیت مربوط به آن است.

در مسیر شکست نیز Error Value ساخته یا منتقل می‌شود.

این ساختار می‌تواند برای برنامه‌هایی که شکست بخشی از رفتار طبیعی آن‌هاست، نسبت به Exception مناسب‌تر باشد.

اما عملکرد واقعی به Compiler، نوع‌های داخلی، Optimization و الگوی استفاده بستگی دارد.

### عدم تضمین صفر بودن هزینه

استفاده از `std::expected` تضمین نمی‌کند که همیشه سریع‌تر از Exception باشد.

مقایسه Performance باید بر اساس سناریوی واقعی و Benchmark انجام شود.

## سازگاری با نسخه‌های مختلف استاندارد

نوع `std::expected` بخشی از Standard Library استاندارد ++C23 است.

برای استفاده از آن باید Compiler و Standard Library هر دو از این قابلیت پشتیبانی کنند.

فعال کردن استاندارد ++C23 به‌تنهایی تضمین نمی‌کند که تمام قابلیت‌های کتابخانه در محیط مورد استفاده در دسترس باشند.

### بررسی Feature Test Macro

برای بررسی وجود قابلیت می‌توان از Macro زیر استفاده کرد:

```cpp
#include <expected>
#include <version>

#if defined(__cpp_lib_expected)

#if __cpp_lib_expected >= 202202L
#define HAS_EXPECTED 1
#else
#define HAS_EXPECTED 0
#endif

#else
#define HAS_EXPECTED 0
#endif
```

مقدار اصلی Feature Test Macro برای `std::expected` برابر است با:

```cpp
202202L
```

برای Monadic Operations نیز مقدار زیر در نظر گرفته شده است:

```cpp
202211L
```

بنابراین، اگر به Monadic Operations نیاز دارید، باید مقدار Macro را جداگانه بررسی کنید.

```cpp
#include <expected>
#include <version>

#if defined(__cpp_lib_expected) && \
    __cpp_lib_expected >= 202211L

#define HAS_EXPECTED_MONADIC 1

#else

#define HAS_EXPECTED_MONADIC 0

#endif
```

این روش برای Libraryهایی که باید با چند Toolchain مختلف سازگار باشند مفید است.

### فعال کردن استاندارد ++C23

برای GCC و Clang معمولاً می‌توان از گزینه زیر استفاده کرد:

```bash
-std=c++23
```

برای Microsoft Visual C++ نیز می‌توان از گزینه زیر استفاده کرد:

```bash
/std:c++latest
```

البته پشتیبانی دقیق به نسخه Compiler و Standard Library وابسته است.

در برخی نسخه‌ها ممکن است لازم باشد از نسخه جدیدتری از Toolchain استفاده شود.

همچنین برخی پیاده‌سازی‌ها ممکن است بخش‌هایی از Standard Library را زودتر یا دیرتر از سایر بخش‌ها ارائه کنند.

بنابراین، برای بررسی سازگاری واقعی باید مستندات نسخه مورد استفاده بررسی و Feature Test Macro کنترل شود.

## کاربردهای عملی std::expected

### کاربرد در File I/O

در عملیات فایل، خطاهایی مانند نبود فایل، دسترسی غیرمجاز یا شکست خواندن بسیار رایج هستند.

می‌توان نتیجه عملیات را به‌صورت زیر تعریف کرد:

```cpp
#include <expected>
#include <string>

enum class FileError {
    not_found,
    permission_denied,
    read_failed
};

std::expected<std::string, FileError>
readFile(const std::string& path);
```

این طراحی اجازه می‌دهد فراخواننده بر اساس نوع خطا رفتار متفاوتی داشته باشد.

### کاربرد در Database

در لایه Database، عملیات‌هایی مانند دریافت، ثبت و حذف اطلاعات ممکن است شکست بخورند.

```cpp
enum class DatabaseError {
    connection_failed,
    query_failed,
    transaction_failed
};

std::expected<User, DatabaseError>
loadUser(int userId);
```

این روش به‌خصوص زمانی مفید است که خطاها بخشی از رفتار قابل‌پیش‌بینی سیستم هستند.

### کاربرد در Network Programming

در برنامه‌های شبکه‌ای، خطاهایی مانند Timeout، Connection Refused و Protocol Error متداول هستند.

```cpp
enum class NetworkError {
    timeout,
    connection_refused,
    protocol_error
};

std::expected<Response, NetworkError>
sendRequest(const Request& request);
```

با استفاده از این طراحی می‌توان خطاهای مختلف را در لایه‌های بالاتر مدیریت کرد.

### کاربرد در Parserها

Parserها معمولاً با انواع مختلف خطا مواجه می‌شوند.

برای مثال:

- Token نامعتبر.
- ساختار ناقص.
- پایان غیرمنتظره ورودی.
- خطای Syntax.

یک Error Type ساختاریافته می‌تواند موقعیت خطا و نوع آن را مشخص کند.

```cpp
struct ParserError {
    std::size_t line;
    std::size_t column;
    std::string message;
};

std::expected<AstNode, ParserError>
parseSource(std::string_view source);
```

این طراحی برای Compilerها، Interpreterها و ابزارهای پردازش زبان بسیار مناسب است.

### کاربرد در Configuration Management

در هنگام خواندن تنظیمات ممکن است مقادیر نامعتبر، کلیدهای ناشناخته یا فایل‌های ناقص وجود داشته باشند.

```cpp
enum class ConfigError {
    file_not_found,
    invalid_value,
    missing_required_key
};

std::expected<Config, ConfigError>
loadConfiguration();
```

این روش امکان تفکیک خطاهای مختلف را فراهم می‌کند.

### کاربرد در Validation

در اعتبارسنجی داده‌ها، ممکن است چندین شرط بررسی شوند و هرکدام خطای خاص خود را داشته باشند.

```cpp
enum class ValidationError {
    invalid_email,
    weak_password,
    invalid_age
};

std::expected<User, ValidationError>
validateUser(const UserInput& input);
```

با این طراحی، تابع علاوه بر اعلام موفقیت یا شکست، دلیل دقیق شکست را نیز مشخص می‌کند.

## طراحی تابعی با std::expected

عملیات Monadic در `std::expected` امکان نوشتن کدهایی با سبک Functional Programming را فراهم می‌کنند.

در این رویکرد، به‌جای کنترل دستی تمام مراحل، عملیات مختلف از طریق توابعی کوچک و مستقل به یکدیگر متصل می‌شوند.

برای مثال:

```cpp
auto result = parseInput(input)
    .and_then(validateInput)
    .and_then(processInput)
    .transform(formatOutput)
    .transform_error(mapError);
```

در این ساختار:

- هر مرحله مسئول یک عملیات مشخص است.
- شکست هر مرحله به‌صورت خودکار منتقل می‌شود.
- عملیات موفقیت‌آمیز به‌صورت زنجیره‌ای اجرا می‌شوند.
- تبدیل Value و Error از یکدیگر مستقل هستند.

البته استفاده بیش از حد از زنجیره‌های طولانی ممکن است خوانایی برنامه را کاهش دهد.

در چنین شرایطی بهتر است عملیات به چند تابع کوچک‌تر تقسیم شوند.

## نکات مهم درباره Thread Safety

خود نوع `std::expected` به‌صورت خودکار Thread-Safe نیست.

اگر چند Thread به یک شیء مشترک دسترسی داشته باشند و حداقل یکی از آن‌ها مقدار شیء را تغییر دهد، ممکن است Data Race رخ دهد.

برای مثال، استفاده مشترک از یک `expected` قابل‌تغییر باید با مکانیزم‌های Synchronization مناسب محافظت شود.

همچنین اگر Value یا Error شامل منابع مشترک باشند، ایمنی دسترسی به آن منابع به طراحی خود آن نوع‌ها وابسته است.

بنابراین، نباید صرفاً به دلیل استفاده از `std::expected` فرض کرد که مدیریت هم‌زمانی نیز به‌صورت خودکار انجام می‌شود.

## نکات مهم درباره APIهای عمومی

هنگام طراحی Libraryهای عمومی مبتنی بر `std::expected`، چند نکته اهمیت ویژه‌ای دارند.

- نوع Error باید بخشی آگاهانه از قرارداد عمومی API باشد.
- تغییر Error Type ممکن است باعث ناسازگاری منبعی شود.
- اضافه کردن خطاهای جدید باید با در نظر گرفتن مصرف‌کنندگان انجام شود.
- نباید جزئیات پیاده‌سازی داخلی بدون ضرورت به API عمومی نشت کنند.
- مستندات باید شرایط ایجاد هر Error Value را مشخص کنند.
- باید معلوم باشد کدام خطاها قابل بازیابی هستند.
- باید مشخص شود که کدام خطاها به لایه بالاتر منتقل می‌شوند.

در طراحی APIهای پایدار، سازگاری نوع‌های بازگشتی اهمیت زیادی دارد.

## تفاوت std::expected با Result در زبان‌های دیگر

الگوی طراحی `std::expected` منحصر به ++C نیست.

زبان Rust نوعی به نام `Result<T, E>` دارد که از نظر مفهومی شباهت زیادی به `std::expected` دارد.

در زبان‌های Functional نیز مفاهیمی مانند `Either` برای نمایش دو حالت جایگزین استفاده می‌شوند.

وجه مشترک این طراحی‌ها، نمایش صریح نتیجه موفقیت و شکست در Type System است.

با این حال، جزئیات Syntax، قوانین مالکیت حافظه، نحوه انتقال خطا و قابلیت‌های زبان‌ها با یکدیگر متفاوت هستند.

بنابراین، شباهت مفهومی نباید به معنای یکسان بودن رفتار آن‌ها در نظر گرفته شود.

## جمع‌بندی

نوع `std::expected` یکی از قابلیت‌های مهم Standard Library در استاندارد ++C23 است که مدیریت خطا را به شکلی صریح، Type-Safe و قابل ترکیب امکان‌پذیر می‌کند.

این نوع به توسعه‌دهندگان اجازه می‌دهد نتیجه موفقیت‌آمیز و اطلاعات خطا را در یک ساختار مشخص نگهداری کنند.

مهم‌ترین قابلیت‌های آن عبارت‌اند از:

- نگهداری Value و Error در یک نوع واحد.
- تعریف Error Type اختصاصی.
- دسترسی به نتیجه از طریق `value`، `error` و عملگرهای مربوطه.
- پشتیبانی از `std::expected<void, E>`.
- ساخت مستقیم Value و Error با `std::in_place` و `std::unexpect`.
- پشتیبانی از Move Semantics.
- پشتیبانی از `constexpr`.
- امکان ترکیب عملیات با `and_then` و `transform`.
- امکان بازیابی یا تبدیل خطا با `or_else` و `transform_error`.
- امکان استفاده در طراحی APIهای چندلایه و حرفه‌ای.

مهم‌ترین نکته در استفاده صحیح از `std::expected`، انتخاب آگاهانه آن برای خطاهایی است که بخشی از قرارداد و رفتار قابل‌پیش‌بینی عملیات هستند.

این نوع جایگزین مطلق Exception، `std::optional` یا `std::variant` نیست؛ بلکه ابزار تخصصی‌تری برای نمایش نتیجه عملیات همراه با احتمال شکست است.

در پروژه‌های مدرن ++C، استفاده صحیح از `std::expected` می‌تواند باعث شفاف‌تر شدن APIها، کاهش وابستگی به Exception برای خطاهای عادی، بهبود قابلیت تست و ساده‌تر شدن جریان مدیریت خطا شود.

## منابع و مراجع

- [cppreference — std::expected](https://en.cppreference.com/w/cpp/utility/expected)
- [cppreference — std::unexpected](https://en.cppreference.com/w/cpp/utility/expected/unexpected)
- [cppreference — std::bad_expected_access](https://en.cppreference.com/w/cpp/utility/expected/bad_expected_access)
- [WG21 P0323R12 — std::expected](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0323r12.html)
- [WG21 P2505R5 — Monadic Functions for std::expected](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2505r5.html)
- [WG21 P2549R1 — std::unexpected should have error() as member accessor](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2549r1.html)
- [C++23 Standard Library Features](https://en.cppreference.com/w/cpp/23)

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>