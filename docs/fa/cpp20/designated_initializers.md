<div align="center">

[🇺🇸 English](../../en/cpp20/designated_initializers.md) | [🇮🇷 فارسی](./designated_initializers.md)

</div>

---

# معرفی Designated Initializers در ++C20

## مقدمه

قابلیت Designated Initializers یکی از ویژگی‌های کاربردی زبان ++C20 است که امکان مقداردهی اولیه اعضای یک `aggregate` را با استفاده از نام آن‌ها فراهم می‌کند.

پیش از معرفی این قابلیت، مقداردهی اولیه اعضای یک `struct` معمولاً بر اساس ترتیب تعریف اعضا انجام می‌شد. در نتیجه، برنامه‌نویس باید ترتیب فیلدها را به خاطر می‌سپرد و هنگام مشاهده کد نیز برای تشخیص ارتباط میان مقدارها و اعضای مقصد، به تعریف ساختار مراجعه می‌کرد.

برای مثال، ساختار زیر را در نظر بگیرید:

```
struct ServerConfig {
    int port;
    bool use_tls;
    int timeout_ms;
};
```

در روش سنتی، مقداردهی اولیه به شکل زیر انجام می‌شود:

```
ServerConfig config{443, true, 5000};
```

اگرچه این کد معتبر است، اما برای تشخیص معنای هر مقدار باید ترتیب اعضای `ServerConfig` را بدانیم.

قابلیت Designated Initializers این مشکل را با امکان مشخص کردن نام اعضا برطرف می‌کند:

```
ServerConfig config{
    .port = 443,
    .use_tls = true,
    .timeout_ms = 5000
};
```

اکنون ارتباط میان هر مقدار و عضو مربوط به آن کاملاً مشخص است.

این قابلیت به‌خصوص در ساختارهای داده‌ای دارای تعداد زیادی عضو، تنظیمات برنامه، پیام‌ها و رکوردهای داده کاربرد دارد.

نکته مهم این است که Designated Initializers یک قابلیت زبانی معرفی‌شده در ++C20 است و نباید آن را با ویژگی مشابه در زبان C یکسان در نظر گرفت. قواعد این قابلیت در ++C محدودیت‌های مشخصی دارند که در ادامه بررسی می‌شوند.

## بررسی مسئله‌ای که این قابلیت حل می‌کند

### ابهام در مقداردهی ترتیبی

در روش سنتی، مقدارها بر اساس ترتیب اعلان اعضای کلاس یا ساختار به آن‌ها اختصاص داده می‌شوند:

```
struct Window {
    int width;
    int height;
    bool resizable;
    int refresh_rate;
};

Window window{1920, 1080, true, 144};
```

برای فهم این عبارت باید تعریف `Window` را بررسی کنیم.

این مسئله زمانی جدی‌تر می‌شود که چند عضو دارای نوع یکسان باشند:

```
struct NetworkConfig {
    int server_port;
    int client_port;
    int timeout;
};

NetworkConfig config{8080, 9090, 3000};
```

در این مثال، جابه‌جایی تصادفی دو مقدار اول می‌تواند باعث ایجاد خطای منطقی شود؛ درحالی‌که کامپایلر هیچ خطایی گزارش نمی‌کند.

### افزایش خوانایی کد

با استفاده از Designated Initializers می‌توان هدف هر مقداردهی را مستقیماً مشخص کرد:

```
NetworkConfig config{
    .server_port = 8080,
    .client_port = 9090,
    .timeout = 3000
};
```

این روش چند مزیت دارد:

- ارتباط مقدارها با اعضای مقصد مشخص است.
- خواندن کد بدون مراجعه مکرر به تعریف ساختار آسان‌تر می‌شود.
- احتمال اشتباه در ترتیب مقدارها کاهش پیدا می‌کند.
- تغییرات آینده در ساختار داده بهتر قابل‌بررسی خواهند بود.

### مقداردهی گزینشی اعضا

یکی دیگر از مزایای مهم این قابلیت، امکان مقداردهی فقط اعضای موردنیاز است.

برای مثال:

```
struct ApplicationConfig {
    int max_connections = 100;
    int timeout = 30;
    bool enable_logging = false;
    bool enable_cache = true;
};

ApplicationConfig config{
    .timeout = 60,
    .enable_logging = true
};
```

در این مثال، فقط دو عضو به‌صورت صریح مقداردهی شده‌اند. دو عضو دیگر از مقدارهای پیش‌فرض تعریف‌شده در ساختار استفاده می‌کنند.

این ویژگی برای ساخت اشیای تنظیمات که تعداد زیادی گزینه اختیاری دارند، بسیار مفید است.

## پیش‌نیاز اصلی: Aggregate بودن نوع

مهم‌ترین محدودیت Designated Initializers این است که فقط برای `aggregate`ها قابل استفاده هستند.

بنابراین، پیش از استفاده از این قابلیت باید مشخص کنیم که نوع موردنظر واقعاً یک aggregate است یا خیر.

### تعریف Aggregate در ++C20

در استاندارد ++C20، یک کلاس برای اینکه `aggregate` محسوب شود باید شرایط زیر را داشته باشد:

- هیچ `user-declared constructor` یا `inherited constructor` نداشته باشد.
- هیچ عضو داده‌ای غیرایستای `private` یا `protected` نداشته باشد.
- هیچ تابع عضو `virtual` نداشته باشد.
- هیچ کلاس پایه `virtual`، `private` یا `protected` نداشته باشد.

کلاس‌های `struct` و `class` از نظر این قواعد تفاوت بنیادی ندارند. تفاوت اصلی آن‌ها در سطح دسترسی پیش‌فرض اعضاست.

ساختار زیر یک aggregate معتبر است:

```
struct Point {
    int x;
    int y;
};

Point point{
    .x = 10,
    .y = 20
};
```

اما ساختار زیر aggregate محسوب نمی‌شود:

```
struct Point {
    int x;
    int y;

    Point(int x_value, int y_value)
        : x{x_value}, y{y_value} {}
};
```

زیرا یک constructor تعریف‌شده توسط کاربر دارد.

در نتیجه، مقداردهی زیر معتبر نیست:

```
// Invalid: Point is not an aggregate.
Point point{
    .x = 10,
    .y = 20
};
```

### تأثیر Default Member Initializers

وجود `default member initializer` در ++C20 مانع aggregate بودن کلاس نیست.

برای مثال:

```
struct UserConfig {
    int max_attempts = 5;
    int timeout = 30;
    bool enabled = true;
};

UserConfig config{
    .timeout = 60
};
```

این ساختار همچنان aggregate است و می‌توان از Designated Initializers برای آن استفاده کرد.

این موضوع از نظر سازگاری نسخه‌های مختلف استاندارد اهمیت دارد:

| استاندارد | وضعیت Default Member Initializers   |
| --------- | ----------------------------------- |
| ++C11     | وجود آن‌ها مانع aggregate بودن بود. |
| ++C14     | این محدودیت حذف شد.                 |
| ++C17     | همچنان مجاز هستند.                  |
| ++C20     | همچنان مجاز هستند.                  |

### تفاوت Aggregate با کلاس دارای Constructor

Designated Initializers جایگزین عمومی constructorها نیستند.

یک constructor می‌تواند مسئولیت‌های مختلفی داشته باشد:

- اعتبارسنجی ورودی‌ها
- حفظ invariantهای کلاس
- مدیریت منابع
- اجرای منطق موردنیاز هنگام ساخت شیء
- جلوگیری از ایجاد وضعیت نامعتبر

در مقابل، Designated Initializers صرفاً اعضای یک aggregate را مقداردهی می‌کنند.

بنابراین، اگر ساخت یک شیء به منطق اعتبارسنجی یا مدیریت منابع نیاز دارد، استفاده از constructor یا factory function معمولاً انتخاب مناسب‌تری است.

## آشنایی با Syntax و قواعد اصلی

### ساختار کلی Designated Initializers

ساختار عمومی این قابلیت به شکل زیر است:

```
Type object{
    .member1 = value1,
    .member2 = value2,
    .member3 = value3
};
```

همچنین می‌توان از شکل زیر استفاده کرد:

```
Type object = {
    .member1 = value1,
    .member2 = value2
};
```

در هر دو حالت، نام عضو با نقطه (`.`) مشخص می‌شود و مقدار آن پس از علامت `=` قرار می‌گیرد.

### استفاده از مقادیر مختلف

مقدار اختصاص داده‌شده به هر عضو می‌تواند یک مقدار ثابت، متغیر یا عبارت معتبر ++C باشد:

```
#include <string>

struct Employee {
    int id;
    std::string name;
    double salary;
    bool active;
};

int employee_id = 1001;
double base_salary = 4500.0;

Employee employee{
    .id = employee_id,
    .name = "Alice",
    .salary = base_salary * 1.2,
    .active = true
};
```

در این مثال، مقدار اعضا می‌تواند از عبارت‌های مختلف به دست آید.

### رعایت ترتیب اعلان اعضا

یکی از مهم‌ترین قواعد Designated Initializers در ++C20 این است که اعضای مشخص‌شده باید مطابق ترتیب تعریف آن‌ها در کلاس نوشته شوند.

برای مثال:

```
struct Product {
    int id;
    double price;
    int quantity;
    bool available;
};
```

مقداردهی زیر معتبر است:

```
Product product{
    .id = 10,
    .quantity = 5,
    .available = true
};
```

زیرا ترتیب اعضای انتخاب‌شده با ترتیب تعریف آن‌ها سازگار است.

اما مثال زیر نامعتبر است:

```
// Invalid: designated initializers are out of declaration order.
Product product{
    .quantity = 5,
    .id = 10,
    .available = true
};
```

حتی اگر نام تمام اعضا درست باشد، ترتیب اشتباه باعث خطای کامپایل می‌شود.

نکته مهم این است که حذف برخی اعضا مجاز است، اما جابه‌جایی اعضای انتخاب‌شده مجاز نیست.

### استفاده از نام اعضای مستقیم

در ++C20، هر Designator باید به یک عضو داده‌ای غیرایستای مستقیم از همان aggregate اشاره کند.

برای مثال:

```
struct Address {
    int house_number;
    int postal_code;
};

struct Person {
    int id;
    Address address;
};
```

نحو زیر معتبر نیست:

```
// Invalid: nested member designators are not supported.
Person person{
    .id = 10,
    .address.postal_code = 12345
};
```

زیرا `postal_code` عضو مستقیم `Person` نیست.

روش صحیح، مقداردهی عضو تو‌در‌تو به‌صورت مستقل است:

```
Person person{
    .id = 10,
    .address = {
        .house_number = 25,
        .postal_code = 12345
    }
};
```

### ممنوعیت تکرار Designator

هر عضو حداکثر یک بار می‌تواند در فهرست مقداردهی نام‌دار ظاهر شود.

```
struct Item {
    int id;
    int quantity;
};
```

مثال زیر نامعتبر است:

```
// Invalid: id is designated more than once.
Item item{
    .id = 10,
    .id = 20,
    .quantity = 5
};
```

برای جلوگیری از چنین وضعیتی، هر عضو باید دقیقاً یک بار در فهرست مقداردهی مشخص شود یا در صورت نیاز از مقدار پیش‌فرض آن استفاده شود.

### ممنوعیت ترکیب مقداردهی ترتیبی و نام‌دار

در ++C20 نمی‌توان initializerهای ترتیبی و Designated Initializers را در یک فهرست مقداردهی ترکیب کرد.

برای مثال:

```
struct Color {
    int red;
    int green;
    int blue;
};
```

عبارت زیر نامعتبر است:

```
// Invalid: positional and designated initialization are mixed.
Color color{
    255,
    .green = 128,
    .blue = 64
};
```

باید یکی از دو روش مقداردهی ترتیبی یا نام‌دار را انتخاب کنید.

## بررسی رفتار اعضای مقداردهی‌نشده

یکی از نکات مهم در استفاده حرفه‌ای از Designated Initializers، درک رفتار اعضایی است که در فهرست مقداردهی مشخص نشده‌اند.

این رفتار از قواعد `Aggregate Initialization` پیروی می‌کند.

### اعضای دارای Default Member Initializer

اگر یک عضو دارای مقدار پیش‌فرض باشد و در فهرست Designated Initializers مشخص نشود، همان مقدار پیش‌فرض استفاده خواهد شد.

```
struct ServerSettings {
    int port = 8080;
    int timeout = 30;
    int max_connections = 100;
    bool enable_ssl = false;
};

ServerSettings settings{
    .port = 443,
    .enable_ssl = true
};
```

نتیجه مقداردهی به شکل زیر خواهد بود:

| عضو               | مقدار نهایی |
| ----------------- | ----------- |
| `port`            | 443         |
| `timeout`         | 30          |
| `max_connections` | 100         |
| `enable_ssl`      | true        |

در این مثال، مقدارهای پیش‌فرض اعضای حذف‌شده حفظ می‌شوند.

### اعضای بدون مقدار پیش‌فرض

اگر یک عضو نه در فهرست مقداردهی مشخص شده باشد و نه `default member initializer` داشته باشد، مطابق قواعد Aggregate Initialization با یک initializer فهرست‌وار خالی مقداردهی می‌شود.

برای مثال:

```
struct Statistics {
    int requests;
    double response_time;
    bool successful;
};

Statistics stats{
    .requests = 100
};
```

مقدار نهایی اعضا به شکل زیر خواهد بود:

- `requests` برابر ۱۰۰
- `response_time` برابر `0.0`
- `successful` برابر `false`

این نتیجه برای اعضای اسکالر معمولی حاصل می‌شود؛ اما نباید آن را به همه انواع تعمیم داد.

برای مثال، یک عضو مرجع باید به یک شیء معتبر متصل شود و نمی‌تواند بدون مقداردهی معتبر باقی بماند.

### تقدم مقدار صریح بر مقدار پیش‌فرض

اگر یک عضو هم مقدار پیش‌فرض داشته باشد و هم در Designated Initializers مشخص شود، مقدار صریح انتخاب خواهد شد.

```
struct RetryPolicy {
    int attempts = 3;
    int delay_ms = 100;
};

RetryPolicy policy{
    .attempts = 10
};
```

در این مثال:

- مقدار `attempts` برابر ۱۰ است.
- مقدار `delay_ms` برابر ۱۰۰ باقی می‌ماند.

## بررسی مقداردهی اعضای تو‌در‌تو

یکی از کاربردهای مهم این قابلیت، ساخت ساختارهای داده‌ای سلسله‌مراتبی است.

فرض کنید تنظیمات یک برنامه از چند بخش تشکیل شده باشد:

```
struct DatabaseConfig {
    std::string host;
    int port;
};

struct LoggingConfig {
    bool enabled;
    int level;
};

struct ApplicationConfig {
    DatabaseConfig database;
    LoggingConfig logging;
};
```

می‌توان ساختار اصلی را همراه با اعضای داخلی آن مقداردهی کرد:

```
ApplicationConfig config{
    .database = {
        .host = "localhost",
        .port = 5432
    },
    .logging = {
        .enabled = true,
        .level = 2
    }
};
```

در این مثال، هر عضو تو‌در‌تو به‌صورت مستقل مقداردهی می‌شود.

توجه داشته باشید که Designated Initializers امکان زنجیره‌کردن نام اعضای تو‌در‌تو را فراهم نمی‌کنند. هر سطح باید با initializer مربوط به عضو مستقیم خودش مقداردهی شود.

### مزایای ساختارهای تو‌در‌تو

این روش در پروژه‌هایی که تنظیمات زیادی دارند، مزایای زیر را فراهم می‌کند:

- تفکیک منطقی تنظیمات
- کاهش احتمال اشتباه در ترتیب مقدارها
- خوانایی بهتر ساختارهای پیچیده
- امکان استفاده از مقدارهای پیش‌فرض در هر سطح
- ساده‌تر شدن ساخت نمونه‌های مختلف از تنظیمات

با این حال، تو‌در‌تو کردن بیش از حد می‌تواند خوانایی را کاهش دهد. در چنین شرایطی می‌توان از متغیرهای میانی یا factory function استفاده کرد.

## بررسی کاربرد Designated Initializers در Union

قابلیت Designated Initializers برای `union` نیز قابل استفاده است.

در یک union، اعضا حافظه مشترک دارند و در هر لحظه فقط یکی از آن‌ها می‌تواند عضو فعال باشد.

برای مثال:

```
union Value {
    int integer;
    double real;
    bool boolean;
};
```

با استفاده از Designated Initializers می‌توان مشخص کرد کدام عضو باید مقداردهی اولیه شود:

```
Value value{
    .real = 3.14
};
```

در این مثال، عضو فعال union برابر `real` خواهد بود.

### محدودیت مقداردهی Union

در یک union نمی‌توان بیش از یک عضو را به‌صورت صریح مقداردهی کرد.

مثال زیر نامعتبر است:

```
// Invalid: only one union member can be explicitly initialized.
Value value{
    .integer = 10,
    .real = 2.5
};
```

همچنین اگر union دارای `default member initializer` برای یکی از اعضا باشد، در صورت حذف تمام اعضا، قواعد مقداردهی پیش‌فرض union باید در نظر گرفته شوند.

در نبود چنین مقداردهی‌ای، اولین عضو union با یک initializer فهرست‌وار خالی مقداردهی می‌شود.

### استفاده از Anonymous Union

این قابلیت برای `anonymous union` نیز کاربرد دارد:

```
struct Token {
    bool is_number;

    union {
        int number;
        char symbol;
    };
};
```

نمونه مقداردهی:

```
Token token{
    .is_number = true,
    .number = 42
};
```

در این مثال، `number` عضو فعال anonymous union خواهد بود.

در هر بار مقداردهی، تنها یکی از اعضای anonymous union باید انتخاب شود.

## بررسی ارتباط با Aggregate Initialization

Designated Initializers بخشی از سازوکار `Aggregate Initialization` در زبان ++C هستند.

برای درک بهتر این قابلیت، باید تفاوت آن را با سایر روش‌های مقداردهی اولیه بدانیم.

### مقایسه با Aggregate Initialization ترتیبی

در روش سنتی، مقداردهی بر اساس ترتیب اعضا انجام می‌شود:

```
struct Rectangle {
    int width;
    int height;
};

Rectangle a{100, 200};
```

در روش نام‌دار، مقصد هر مقدار مشخص است:

```
Rectangle b{
    .width = 100,
    .height = 200
};
```

هر دو روش نتیجه یکسانی دارند.

تفاوت اصلی در خوانایی و نحوه بیان هدف برنامه‌نویس است.

مقداردهی ترتیبی برای ساختارهای کوچک و ساده همچنان مناسب است؛ اما در ساختارهایی با اعضای زیاد، روش نام‌دار معمولاً انتخاب بهتری خواهد بود.

### مقایسه با Constructor

Constructorها برای کنترل فرآیند ساخت شیء طراحی شده‌اند.

برای مثال:

```
#include <stdexcept>

class Port {
public:
    explicit Port(int value)
        : value_{value} {
        if (value < 1 || value > 65535) {
            throw std::out_of_range("Invalid port");
        }
    }

    int get() const {
        return value_;
    }

private:
    int value_;
};
```

در این مثال، constructor محدوده معتبر پورت را بررسی می‌کند.

چنین کلاسی به دلیل داشتن constructor و عضو داده‌ای `private` یک aggregate نیست.

بنابراین نمی‌توان از Designated Initializers برای ساخت آن استفاده کرد.

این موضوع نشان می‌دهد که انتخاب میان aggregate و constructor باید بر اساس مسئولیت و طراحی نوع انجام شود، نه صرفاً کوتاه‌تر بودن Syntax.

### مقایسه با std::initializer_list

نوع `std::initializer_list` یکی از قابلیت‌های کتابخانه استاندارد ++C است که امکان دریافت فهرستی از مقادیر هم‌نوع را فراهم می‌کند.

برای مثال:

```
#include <initializer_list>

class NumberCollection {
public:
    NumberCollection(std::initializer_list<int> values)
        : values_{values} {}

private:
    std::initializer_list<int> values_;
};
```

اما Designated Initializers یک قابلیت نحوی زبان است که اعضای یک aggregate را با نام مشخص می‌کند.

بنابراین، این دو مفهوم از نظر هدف و نحوه عملکرد متفاوت هستند.

## بررسی تفاوت با Designated Initializers در زبان C

زبان C از نسخه‌های قدیمی‌تر خود از Designated Initializers پشتیبانی می‌کند.

با این حال، قواعد این قابلیت در زبان C با ++C20 یکسان نیستند.

مهم‌ترین تفاوت‌ها عبارت‌اند از:

| قابلیت                           | زبان C          | ++C20            |
| -------------------------------- | --------------- | ---------------- |
| نام‌گذاری اعضای ساختار           | پشتیبانی می‌شود | پشتیبانی می‌شود  |
| ترتیب دلخواه Designatorها        | پشتیبانی می‌شود | مجاز نیست        |
| Designator تو‌در‌تو              | پشتیبانی می‌شود | پشتیبانی نمی‌شود |
| Designator شاخص آرایه            | پشتیبانی می‌شود | پشتیبانی نمی‌شود |
| ترکیب مقداردهی ترتیبی و نام‌دار  | پشتیبانی می‌شود | پشتیبانی نمی‌شود |
| نام‌گذاری اعضای مستقیم aggregate | پشتیبانی می‌شود | پشتیبانی می‌شود  |

### تفاوت در ترتیب اعضا

در زبان C می‌توان اعضا را با ترتیبی متفاوت از ترتیب اعلان مشخص کرد.

اما در ++C20 ترتیب Designatorها باید با ترتیب اعلان اعضا یکسان باشد.

### تفاوت در مقداردهی آرایه

زبان C امکان استفاده از index designator را فراهم می‌کند:

```
int values[5] = {
    [2] = 42
};
```

اما این نحو در ++C20 پشتیبانی نمی‌شود.

برای مقداردهی آرایه در ++C باید از روش‌های استاندارد مربوط به Aggregate Initialization استفاده کرد.

### تفاوت در اعضای تو‌در‌تو

در زبان C می‌توان از Designatorهای زنجیره‌ای برای اعضای تو‌در‌تو استفاده کرد.

اما در ++C20 باید ابتدا عضو مستقیم و سپس اعضای داخلی آن را در initializer جداگانه مشخص کرد.

این تفاوت‌ها هنگام انتقال کدهای C به ++C اهمیت زیادی دارند.

## بررسی ارتباط با کلاس‌های دارای Base Class

در ++C20، داشتن کلاس پایه به‌خودی‌خود مانع aggregate بودن کلاس مشتق‌شده نیست.

البته کلاس پایه نباید شرایط ممنوع‌شده در تعریف aggregate را داشته باشد.

برای مثال:

```
struct Base {
    int id;
};

struct Derived : Base {
    int value;
};
```

کلاس `Derived` می‌تواند aggregate باشد.

با این حال، Designated Initializers در ++C20 فقط اعضای داده‌ای مستقیم کلاس را نام‌گذاری می‌کنند و امکان نام‌گذاری مستقیم اعضای کلاس پایه را ندارند.

برای مقداردهی بخش پایه باید از قواعد Aggregate Initialization استفاده کرد:

```
Derived object{
    {100},
    .value = 200
};
```

این مثال از مقداردهی ترتیبی برای بخش پایه و Designated Initializer برای عضو مستقیم استفاده می‌کند، اما چنین ترکیبی در یک فهرست initializer مجاز نیست.

بنابراین مثال بالا معتبر نیست و باید روش مقداردهی را به شکل سازگار با قواعد ++C20 انتخاب کرد.

روش ساده و قابل‌حمل، استفاده از مقداردهی ترتیبی برای کل aggregate است:

```
Derived object{
    {100},
    200
};
```

این نکته اهمیت ویژه‌ای دارد: در ++C20 نمی‌توان برای مقداردهی بخش پایه و اعضای مشتق‌شده، روش ترتیبی و نام‌دار را در یک initializer ترکیب کرد.

## بررسی Narrowing Conversion

Designated Initializers از قواعد `List Initialization` پیروی می‌کنند.

به همین دلیل، تبدیل‌های `narrowing` مجاز نیستند.

برای مثال:

```
struct Measurement {
    int count;
};
```

مقداردهی زیر نامعتبر است:

```
// Invalid: floating-point to integer narrowing conversion.
Measurement measurement{
    .count = 3.7
};
```

زیرا تبدیل مقدار اعشاری به عدد صحیح می‌تواند باعث از دست رفتن اطلاعات شود.

اگر چنین تبدیلی واقعاً موردنیاز است، باید آن را به‌صورت آگاهانه و همراه با بررسی محدوده و دقت داده انجام داد.

برای مثال:

```
double input = 3.7;

int converted = static_cast<int>(input);

Measurement measurement{
    .count = converted
};
```

در این حالت تبدیل پیش از مقداردهی انجام شده است.

البته استفاده از `static_cast` به‌تنهایی به معنای معتبر بودن مقدار ورودی از نظر منطق برنامه نیست و مسئولیت بررسی آن همچنان بر عهده برنامه‌نویس است.

## بررسی ترتیب ارزیابی و Side Effects

در Aggregate Initialization، اعضای aggregate مطابق ترتیب عناصر آن مقداردهی می‌شوند.

در نتیجه، ترتیب اجرای عبارت‌های مقداردهی نیز قابل پیش‌بینی است.

برای مثال:

```
#include <iostream>

int next_value() {
    static int value = 0;
    return ++value;
}

struct Pair {
    int first;
    int second;
};

int main() {
    Pair pair{
        .first = next_value(),
        .second = next_value()
    };

    std::cout << pair.first << '\n';
    std::cout << pair.second << '\n';
}
```

خروجی:

```
1
2
```

ابتدا `first` و سپس `second` مقداردهی می‌شود.

با وجود این، بهتر است از عبارت‌های پیچیده دارای Side Effects در initializerها اجتناب شود؛ زیرا چنین کدی می‌تواند فهم و نگهداشت برنامه را دشوار کند.

## کاربردهای عملی Designated Initializers

### ساخت اشیای تنظیمات

یکی از بهترین کاربردهای این قابلیت، ایجاد اشیای تنظیمات است.

```
#include <string>

struct DatabaseConfig {
    std::string host = "localhost";
    int port = 5432;
    int pool_size = 10;
    bool enable_tls = false;
};

DatabaseConfig config{
    .host = "database.internal",
    .port = 5433,
    .enable_tls = true
};
```

در این مثال، تنظیمات موردنیاز به‌صورت صریح مشخص شده‌اند و سایر اعضا از مقدارهای پیش‌فرض استفاده می‌کنند.

این الگو برای تنظیمات برنامه، ارتباط با سرویس‌ها و مدیریت منابع بسیار مناسب است.

### ساخت داده‌های تست

در Unit Testing معمولاً نیاز داریم نمونه‌هایی از یک ساختار داده با وضعیت‌های مختلف ایجاد کنیم.

Designated Initializers این کار را ساده‌تر می‌کند:

```
#include <string>

struct User {
    int id;
    std::string name;
    bool active = true;
};

User active_user{
    .id = 1,
    .name = "Alice"
};

User inactive_user{
    .id = 2,
    .name = "Bob",
    .active = false
};
```

در این مثال، هدف هر نمونه به‌راحتی قابل‌تشخیص است.

همچنین تغییر ترتیب اعضا در کدهای تست باعث جابه‌جایی تصادفی مقدارها نخواهد شد، زیرا مقصد هر مقدار مشخص است.

### ساخت پیام‌ها و رکوردهای داده

برای ساختارهایی که صرفاً داده را میان بخش‌های مختلف برنامه منتقل می‌کنند، این قابلیت می‌تواند خوانایی مناسبی ایجاد کند.

```
struct Message {
    int id;
    int priority;
    bool encrypted;
    int payload_size;
};

Message message{
    .id = 100,
    .priority = 2,
    .encrypted = true,
    .payload_size = 1024
};
```

با این حال، اگر این ساختار برای Serialization یا ارتباط شبکه استفاده می‌شود، نباید چیدمان حافظه آن را بدون طراحی مشخص به‌عنوان قالب انتقال داده در نظر گرفت.

مسائلی مانند `padding`، `endianness` و تفاوت پیاده‌سازی کامپایلرها باید به‌صورت مستقل مدیریت شوند.

### ساخت مقادیر پیش‌فرض سفارشی

این قابلیت امکان ایجاد چند نمونه با تنظیمات متفاوت را فراهم می‌کند:

```
struct ThreadPoolConfig {
    int worker_count = 4;
    int queue_size = 100;
    bool enable_monitoring = false;
};

ThreadPoolConfig development{
    .worker_count = 2,
    .enable_monitoring = true
};

ThreadPoolConfig production{
    .worker_count = 16,
    .queue_size = 1000
};
```

در این مثال، هر محیط فقط تنظیماتی را تغییر می‌دهد که با مقدار پیش‌فرض تفاوت دارند.

## بررسی محدودیت‌های مهم

### محدودیت مربوط به Constructor

هر constructor تعریف‌شده یا اعلام‌شده توسط کاربر می‌تواند مانع aggregate بودن کلاس شود.

این موضوع شامل constructorهایی است که با `= default` یا `= delete` مشخص شده‌اند؛ زیرا همچنان `user-declared` محسوب می‌شوند.

برای مثال:

```
struct Configuration {
    int value;

    Configuration() = default;
};
```

این کلاس aggregate نیست و نمی‌توان از Designated Initializers برای آن استفاده کرد.

### محدودیت مربوط به Access Control

وجود عضو داده‌ای غیرایستای `private` یا `protected` در کلاس مانع aggregate بودن آن می‌شود.

```
class User {
private:
    int id;

public:
    std::string name;
};
```

این کلاس aggregate نیست.

این محدودیت با اصول Encapsulation ارتباط دارد و اجازه نمی‌دهد Designated Initializers راهی برای دور زدن سطح دسترسی اعضا باشند.

### محدودیت مربوط به Virtual Function

کلاسی که دارای تابع عضو `virtual` باشد، aggregate محسوب نمی‌شود.

```
struct Base {
    virtual void execute() {}
};
```

در نتیجه، نمی‌توان برای چنین کلاسی از Designated Initializers استفاده کرد.

### محدودیت مربوط به Static Data Members

اعضای داده‌ای `static` بخشی از زیرشیء aggregate نیستند و نمی‌توان آن‌ها را با Designated Initializers مقداردهی کرد.

برای مثال:

```
struct Config {
    int port;
    static int default_port;
};
```

عبارت زیر نامعتبر است:

```
// Invalid: static data members cannot be designated.
Config config{
    .port = 8080,
    .default_port = 443
};
```

عضو `default_port` باید مطابق قواعد معمول اعضای static مدیریت شود.

### محدودیت مربوط به Inheritance

اگر کلاس پایه `virtual`، `private` یا `protected` باشد، کلاس مشتق‌شده aggregate محسوب نمی‌شود.

همچنین Designated Initializers در ++C20 امکان نام‌گذاری مستقیم اعضای کلاس پایه را فراهم نمی‌کنند.

## بررسی خطاهای رایج

### تصور اینکه ترتیب اعضا اختیاری است

یکی از اشتباهات رایج این است که برنامه‌نویس تصور می‌کند چون اعضا با نام مشخص می‌شوند، ترتیب آن‌ها اهمیتی ندارد.

این تصور در ++C20 نادرست است.

ترتیب Designatorها باید با ترتیب اعلان اعضا مطابقت داشته باشد.

### تصور اینکه این قابلیت برای تمام کلاس‌ها قابل استفاده است

Designated Initializers فقط برای aggregateها قابل استفاده هستند.

بنابراین، وجود constructor، تابع مجازی یا عضو داده‌ای غیرعمومی می‌تواند مانع استفاده از این قابلیت شود.

### استفاده از Syntax زبان C

قواعد Designated Initializers در زبان C و ++C20 متفاوت هستند.

بنابراین، انتقال مستقیم کدهای C به ++C بدون بررسی قواعد زبان مقصد می‌تواند باعث خطای کامپایل شود.

### نادیده گرفتن Default Member Initializers

اگر یک عضو مقدار پیش‌فرض داشته باشد، حذف آن از فهرست مقداردهی باعث استفاده از همان مقدار پیش‌فرض می‌شود.

برنامه‌نویس باید این رفتار را در طراحی تنظیمات و داده‌های پیش‌فرض در نظر بگیرد.

### استفاده بیش از حد از Aggregate

صرفاً برای استفاده از Designated Initializers نباید تمام کلاس‌های برنامه را به aggregate تبدیل کرد.

اگر یک کلاس مسئول حفظ invariantها، اعتبارسنجی یا مدیریت منابع است، استفاده از constructor می‌تواند انتخاب مناسب‌تری باشد.

### نادیده گرفتن پایداری API

هنگامی که کاربران یک کتابخانه از Designated Initializers استفاده می‌کنند، نام اعضای عمومی به بخشی از رابط استفاده آن کتابخانه تبدیل می‌شود.

در نتیجه:

- تغییر نام اعضا می‌تواند کد مصرف‌کنندگان را خراب کند.
- حذف یک عضو ممکن است باعث ناسازگاری شود.
- اضافه کردن constructor می‌تواند aggregate بودن نوع را از بین ببرد.
- تغییر سطح دسترسی اعضا می‌تواند قابلیت مقداردهی نام‌دار را حذف کند.

بنابراین، استفاده از این قابلیت در APIهای عمومی باید با درنظرگرفتن هزینه تغییرات آینده انجام شود.

## بررسی ملاحظات طراحی

### زمان مناسب استفاده

Designated Initializers معمولاً در شرایط زیر انتخاب مناسبی هستند:

- ساختار داده دارای اعضای متعدد است.
- چند عضو دارای نوع یکسان هستند.
- تعداد زیادی از اعضا مقدار پیش‌فرض دارند.
- خوانایی کد اهمیت زیادی دارد.
- ساخت شیء نیازمند منطق پیچیده نیست.
- نوع موردنظر یک Data Transfer Object ساده است.

### زمان نامناسب استفاده

در شرایط زیر بهتر است از constructor یا factory function استفاده شود:

- ساخت شیء نیازمند اعتبارسنجی چند عضو با یکدیگر است.
- کلاس باید invariantهای مشخصی را حفظ کند.
- اعضای داده باید `private` باقی بمانند.
- ایجاد شیء نیازمند مدیریت منابع است.
- جزئیات پیاده‌سازی نباید بخشی از API عمومی باشند.
- مراحل ساخت شیء شامل منطق پیچیده هستند.

### انتخاب سبک یکپارچه

در یک پروژه حرفه‌ای بهتر است برای هر نوع داده، سبک مقداردهی مشخص و یکپارچه‌ای انتخاب شود.

برای مثال، اگر یک ساختار تنظیمات در بخش‌های مختلف پروژه با Designated Initializers ساخته می‌شود، بهتر است همین سبک در نقاط مشابه نیز رعایت شود.

استفاده نامنظم از روش‌های مختلف، حتی اگر از نظر زبان معتبر باشد، می‌تواند خوانایی کد را کاهش دهد.

## بررسی تفاوت نسخه‌های استاندارد

Designated Initializers در ++C20 معرفی شدند، اما قواعد Aggregate Initialization در نسخه‌های مختلف ++C تغییرات مهمی داشته‌اند.

برخی از تغییرات مرتبط عبارت‌اند از:

| استاندارد | تغییر مرتبط                                                     |
| --------- | --------------------------------------------------------------- |
| ++C11     | معرفی List Initialization و Aggregate Initialization مدرن       |
| ++C14     | امکان aggregate بودن کلاس‌های دارای Default Member Initializers |
| ++C17     | گسترش تعریف Aggregate برای پشتیبانی از Base Classها             |
| ++C20     | معرفی Designated Initializers                                   |
| ++C23     | تغییرات و اصلاحات مرتبط با Aggregate Initialization             |

نکته مهم این است که پشتیبانی از یک قابلیت در نسخه جدیدتر استاندارد به معنای پشتیبانی از تمام نحوهای مشابه زبان C نیست.

هنگام توسعه کتابخانه‌های قابل‌حمل باید قواعد دقیق استاندارد هدف را رعایت کرد.

## بررسی پشتیبانی کامپایلرها

برای استفاده از Designated Initializers باید کامپایلر در حالت ++C20 یا استاندارد جدیدتر اجرا شود.

### کامپایل با GCC

```
g++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o main
```

### کامپایل با Clang

```
clang++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o main
```

### کامپایل با MSVC

```
cl /std:c++20 /W4 /EHsc main.cpp
```

پشتیبانی دقیق به نسخه کامپایلر و میزان پیاده‌سازی استاندارد در آن بستگی دارد.

برای پروژه‌های حرفه‌ای، بهتر است نسخه‌های هدف کامپایلر در فرآیند CI آزمایش شوند و از Extensionهای اختصاصی کامپایلر به‌عنوان رفتار استاندارد ++C استفاده نشود.

## جمع‌بندی

قابلیت Designated Initializers در ++C20 راهکاری خوانا و کاربردی برای مقداردهی اولیه اعضای `aggregate` با استفاده از نام آن‌ها فراهم می‌کند.

این قابلیت مشکلاتی مانند وابستگی ذهنی به ترتیب اعضا، دشواری تشخیص ارتباط مقدارها با مقصد و کاهش خوانایی ساختارهای داده‌ای بزرگ را برطرف می‌کند.

مهم‌ترین مزایای آن عبارت‌اند از:

- مشخص شدن مقصد هر مقداردهی
- افزایش خوانایی کد
- کاهش خطاهای ناشی از جابه‌جایی مقدارها
- امکان مقداردهی گزینشی اعضا
- استفاده مناسب از Default Member Initializers
- ساده‌تر شدن ساخت اشیای تنظیمات و رکوردهای داده

با این حال، محدودیت‌های آن نیز باید به‌خوبی شناخته شوند:

- فقط برای aggregateها قابل استفاده است.
- ترتیب Designatorها باید با ترتیب اعلان اعضا مطابقت داشته باشد.
- فقط اعضای داده‌ای غیرایستای مستقیم قابل نام‌گذاری هستند.
- مقداردهی تو‌در‌تو باید در سطح عضو مستقیم انجام شود.
- ترکیب مقداردهی ترتیبی و نام‌دار مجاز نیست.
- قواعد آن با Designated Initializers در زبان C تفاوت دارد.
- جایگزین constructorهای مسئول اعتبارسنجی و حفظ invariantها نیست.

در نهایت، Designated Initializers ابزاری ارزشمند برای طراحی و پیاده‌سازی کدهای مدرن ++C هستند؛ به‌شرط آنکه در موقعیت مناسب و با درک کامل قواعد Aggregate Initialization مورد استفاده قرار گیرند.

## منابع و مراجع

برای مطالعه جزئیات استاندارد و بررسی نمونه‌های بیشتر، منابع زیر مفید هستند:

- [cppreference — Aggregate initialization](https://en.cppreference.com/w/cpp/language/aggregate_initialization?utm_source=chatgpt.com)
- [cppreference — Aggregate](https://en.cppreference.com/w/cpp/language/aggregate?utm_source=chatgpt.com)
- [cppreference — List initialization](https://en.cppreference.com/w/cpp/language/list_initialization?utm_source=chatgpt.com)
- [WG21 N4810 — C++20 Working Draft](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/n4810.pdf?utm_source=chatgpt.com)
- [ISO C++ — C++ Standards](https://isocpp.org/std/the-standard?utm_source=chatgpt.com)
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines?utm_source=chatgpt.com)

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>