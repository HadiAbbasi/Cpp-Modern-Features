<div align="center">

[🇺🇸 English](../../en/cpp11/enum-class.md) | [🇮🇷 فارسی](./enum-class.md)

</div>

---

# معرفی enum class در ++C

مفهوم `enum class` یکی از قابلیت‌های زبان ++C برای تعریف مجموعه‌ای محدود و نام‌گذاری‌شده از مقادیر است. این قابلیت نسخهٔ type-safe و مدرن `enum` سنتی محسوب می‌شود و از ++C11 در دسترس است. موضوع این سند بر تعریف `enum class` و همچنین شکل `enum struct` متمرکز است. 

## مفهوم enumeration

مفهوم `enumeration` برای زمانی مناسب است که یک مقدار فقط باید یکی از چند گزینهٔ مشخص را داشته باشد. به‌جای استفاده از اعداد خام، می‌توان این گزینه‌ها را با نام‌های معنادار تعریف کرد.

برای مثال، اگر وضعیت یک درخواست فقط می‌تواند `Pending`، `Running` یا `Completed` باشد، استفاده از یک نوع enumeration این محدودیت را در خود مدل داده منعکس می‌کند.

```cpp
enum Status {
    Pending,
    Running,
    Completed
};
```

در این مثال، مقادیر enumeration معمولاً با مقادیر صحیح متناظر هستند و enumeration سنتی در برخی contextها می‌تواند به‌صورت ضمنی به نوع صحیح تبدیل شود.

این رفتار یکی از دلایل اصلی معرفی `enum class` در ++C11 بود؛ زیرا enumerationهای سنتی از نظر type safety و جلوگیری از برخورد نام‌ها محدودیت‌هایی داشتند.

## معرفی enum class

نوع `enum class` یک enumeration با `scoped` بودن و type safety بیشتر است.

```cpp
enum class Status {
    Pending,
    Running,
    Completed
};
```

اکنون برای دسترسی به مقادیر باید نام enumeration نیز مشخص شود:

```cpp
Status status = Status::Pending;
```

این ویژگی باعث می‌شود نام‌هایی مانند `Pending` یا `Completed` مستقیماً وارد scope بیرونی نشوند.

در نتیجه، استفاده از `enum class` معمولاً انتخاب مناسب‌تری برای کدهای مدرن ++C است، مگر اینکه رفتارهای خاص enumeration سنتی عمداً مورد نیاز باشند.

## تفاوت enum class و enum سنتی

مهم‌ترین تفاوت `enum class` با `enum` سنتی در scope و تبدیل‌های ضمنی است.

در enumeration سنتی:

```cpp
enum Color {
    Red,
    Green,
    Blue
};

Color color = Red;
```

اما در `enum class`:

```cpp
enum class Color {
    Red,
    Green,
    Blue
};

Color color = Color::Red;
```

مقادیر `enum class` در scope نوع enumeration قرار دارند. بنابراین `Color::Red` باید به‌صورت صریح نوشته شود.

این ویژگی از برخورد نام‌ها جلوگیری می‌کند:

```cpp
enum class TrafficLight {
    Red,
    Yellow,
    Green
};

enum class Color {
    Red,
    Green,
    Blue
};
```

در اینجا هر دو enumeration می‌توانند مقدار `Red` داشته باشند، زیرا این نام‌ها در scopeهای متفاوت قرار دارند.

## مزیت type safety

یکی از مهم‌ترین مزایای `enum class` جلوگیری از تبدیل ضمنی آن به انواع صحیح است.

کد زیر با `enum class` معتبر نیست:

```cpp
enum class Status {
    Pending,
    Completed
};

Status status = Status::Pending;

// int value = status;
```

برای تبدیل به یک نوع صحیح باید تبدیل صریح انجام شود:

```cpp
int value = static_cast<int>(status);
```

این محدودیت از اشتباهاتی جلوگیری می‌کند که در آن یک enumeration به‌صورت ناخواسته وارد محاسبات عددی، مقایسه‌ها یا APIهای دیگر می‌شود.

در مقابل، enumeration سنتی می‌تواند در بسیاری از contextها به‌صورت ضمنی به یک نوع صحیح تبدیل شود:

```cpp
enum Status {
    Pending,
    Completed
};

Status status = Pending;

int value = status;
```

بنابراین `enum class` معمولاً برای مدل‌سازی domain valueها انتخاب امن‌تری است.

## تفاوت scoped بودن

مقادیر `enum class` در scope خود enumeration قرار دارند.

```cpp
enum class Direction {
    North,
    South,
    East,
    West
};

Direction direction = Direction::North;
```

نوشتن `North` به‌تنهایی صحیح نیست:

```cpp
// Direction direction = North;
```

این scoped بودن برای پروژه‌های بزرگ اهمیت زیادی دارد؛ زیرا تعداد زیادی enumeration می‌توانند مقادیر دارای نام‌های مشابه داشته باشند بدون اینکه با یکدیگر collision ایجاد کنند.

## تعیین نوع زیرین

به‌صورت پیش‌فرض، implementation نوع زیرین enumeration را انتخاب می‌کند. در صورت نیاز می‌توان underlying type را صریحاً مشخص کرد.

```cpp
enum class Status : std::uint8_t {
    Pending,
    Running,
    Completed
};
```

برای این کار معمولاً به `<cstdint>` نیاز است:

```cpp
#include <cstdint>

enum class Status : std::uint8_t {
    Pending,
    Running,
    Completed
};
```

تعیین underlying type زمانی مفید است که اندازهٔ داده، layout، serialization یا compatibility با یک interface مشخص اهمیت داشته باشد.

با این حال، تعیین `std::uint8_t` صرفاً برای کوچک‌تر کردن یک متغیر همیشه ضروری نیست. در طراحی معمول application code بهتر است underlying type فقط زمانی مشخص شود که یک نیاز واقعی وجود داشته باشد.

## مقداردهی صریح اعضا

می‌توان برای اعضای enumeration مقدار مشخص تعیین کرد.

```cpp
enum class ErrorCode {
    None = 0,
    NotFound = 404,
    PermissionDenied = 403,
    InternalError = 500
};
```

در این حالت مقدار هر enumerator مطابق مقدار مشخص‌شده خواهد بود.

مقادیر بعدی نیز می‌توانند بر اساس مقدار قبلی ادامه پیدا کنند:

```cpp
enum class Priority {
    Low = 1,
    Medium,
    High
};
```

در این مثال `Medium` مقدار `2` و `High` مقدار `3` خواهد داشت.

همچنین می‌توان مقادیر کاملاً مستقل تعریف کرد:

```cpp
enum class HttpStatus {
    Ok = 200,
    BadRequest = 400,
    NotFound = 404,
    InternalServerError = 500
};
```

## enum struct چیست؟

شکل `enum struct` نیز در ++C وجود دارد و از نظر semantics با `enum class` تفاوتی ندارد.

```cpp
enum struct Status {
    Pending,
    Running,
    Completed
};
```

این تعریف از نظر رفتار معادل تعریف زیر است:

```cpp
enum class Status {
    Pending,
    Running,
    Completed
};
```

هر دو scoped enumeration هستند و همان قواعد type safety را دارند.

بنابراین `enum struct` را می‌توان بیشتر یک syntax alternative برای `enum class` دانست.

در بیشتر پروژه‌ها `enum class` رایج‌تر است و معمولاً خوانایی بیشتری برای توسعه‌دهندگان دارد، اما انتخاب بین این دو از نظر semantics تفاوتی ایجاد نمی‌کند.

## استفاده از enum class در switch

یکی از کاربردهای رایج `enum class` استفاده در `switch` است.

```cpp
enum class Status {
    Pending,
    Running,
    Completed,
    Failed
};

void handleStatus(Status status) {
    switch (status) {
        case Status::Pending:
            break;

        case Status::Running:
            break;

        case Status::Completed:
            break;

        case Status::Failed:
            break;
    }
}
```

استفاده از `enum class` در چنین مواردی باعث می‌شود گزینه‌های مجاز کاملاً مشخص باشند.

در صورت اضافه شدن یک enumerator جدید، بررسی compiler warningها می‌تواند به پیدا کردن `switch`هایی که آن حالت را پوشش نمی‌دهند کمک کند. برای پروژه‌هایی که exhaustiveness اهمیت دارد، تنظیم warningهای compiler به‌صورت مناسب بسیار مفید است.

## مقدار پیش‌فرض enum class

اگر متغیری از نوع `enum class` به‌صورت default-initialization ساخته شود، مقدار آن لزوماً یکی از enumeratorهای تعریف‌شده نیست.

```cpp
enum class Status {
    Pending,
    Completed
};

Status status;
```

در این حالت، برای متغیر local با storage duration خودکار، مقدار مشخص و قابل استفاده‌ای برای `status` وجود ندارد.

بهتر است در طراحی‌هایی که مقدار معتبر اولیه اهمیت دارد، مقدار را صریحاً تعیین کنید:

```cpp
Status status = Status::Pending;
```

یا از initialization مناسب دیگری استفاده کنید.

همچنین باید توجه داشت که enumeration از نظر زبان می‌تواند مقادیری داشته باشد که هیچ enumerator نام‌گذاری‌شده‌ای برای آن‌ها وجود ندارد. بنابراین داشتن یک نوع `enum class` الزاماً به این معنا نیست که هر مقدار ممکن، یک enumerator معتبر و نام‌گذاری‌شده است.

## مقادیر خارج از enumeratorها

در `enum class` نیز ممکن است از طریق تبدیل صریح مقداری ایجاد شود که با هیچ enumerator تعریف‌شده‌ای متناظر نیست.

```cpp
enum class Status {
    Pending = 0,
    Completed = 1
};

Status status = static_cast<Status>(42);
```

در اینجا `status` دارای یک مقدار enumeration است، اما `42` هیچ enumerator نام‌گذاری‌شده‌ای ندارد.

این موضوع هنگام کار با داده‌های خارجی، serialization، فایل‌ها یا network protocolها اهمیت زیادی دارد.

اگر داده از خارج از برنامه وارد می‌شود، نباید صرفاً با تبدیل مستقیم آن به `enum class` فرض کرد که مقدار معتبر است.

## اعتبارسنجی مقادیر

اگر مقدار enumeration از یک منبع خارجی دریافت می‌شود، بهتر است ابتدا آن را اعتبارسنجی کنید.

برای مثال:

```cpp
enum class Status : std::uint8_t {
    Pending = 0,
    Running = 1,
    Completed = 2
};

bool isValidStatus(std::uint8_t value) {
    switch (value) {
        case 0:
        case 1:
        case 2:
            return true;
        default:
            return false;
    }
}
```

سپس می‌توان فقط مقدارهای معتبر را به `Status` تبدیل کرد.

```cpp
std::uint8_t rawValue = 1;

if (isValidStatus(rawValue)) {
    Status status = static_cast<Status>(rawValue);
}
```

البته روش validation باید با domain واقعی سازگار باشد؛ به‌خصوص اگر enumeration دارای مقادیر sparse باشد.

## مقایسه مقادیر enum class

دو مقدار از یک `enum class` را می‌توان مستقیماً با یکدیگر مقایسه کرد.

```cpp
enum class Status {
    Pending,
    Completed
};

Status status = Status::Pending;

if (status == Status::Pending) {
    // Handle pending status
}
```

اما مقایسه مستقیم آن با یک integer مجاز نیست:

```cpp
// if (status == 0) {
// }
```

در صورت نیاز باید conversion صریح انجام شود:

```cpp
if (static_cast<int>(status) == 0) {
    // Handle zero
}
```

با این حال، در کد domain-level بهتر است تا حد امکان خود enumerator را با مقدار عددی آن مقایسه نکنید.

## استفاده از enum class در API

استفاده از `enum class` در interface توابع باعث می‌شود قرارداد API شفاف‌تر شود.

```cpp
enum class LogLevel {
    Debug,
    Info,
    Warning,
    Error
};

void setLogLevel(LogLevel level);
```

فراخوانی تابع نیز کاملاً گویا است:

```cpp
setLogLevel(LogLevel::Warning);
```

در مقابل، APIای مانند زیر اطلاعات کمتری دربارهٔ مقادیر مجاز منتقل می‌کند:

```cpp
void setLogLevel(int level);
```

استفاده از `enum class` باعث می‌شود compiler نیز بخشی از این قرارداد را enforce کند.

## enum class و overload

از آنجا که هر `enum class` یک type مشخص است، می‌توان overloadهایی با enumerationهای مختلف داشت.

```cpp
enum class Color {
    Red,
    Blue
};

enum class Direction {
    North,
    South
};

void process(Color value);
void process(Direction value);
```

این ویژگی type safety بیشتری نسبت به استفاده از چند `int` مشابه ایجاد می‌کند.

## enum class و bit flags

یکی از کاربردهای پیشرفتهٔ enumeration استفاده از آن برای مجموعه‌ای از flagها است.

برای مثال:

```cpp
enum class Permission : unsigned int {
    None    = 0,
    Read    = 1 << 0,
    Write   = 1 << 1,
    Execute = 1 << 2
};
```

اما برخلاف enumeration سنتی، عملگرهای bitwise برای `enum class` به‌صورت خودکار فراهم نمی‌شوند.

بنابراین کد زیر به‌طور پیش‌فرض قابل استفاده نیست:

```cpp
// auto permissions = Permission::Read | Permission::Write;
```

اگر چنین APIای واقعاً مورد نیاز باشد، می‌توان operatorهای مناسب را به‌صورت کنترل‌شده تعریف کرد:

```cpp
#include <type_traits>

enum class Permission : unsigned int {
    None    = 0,
    Read    = 1 << 0,
    Write   = 1 << 1,
    Execute = 1 << 2
};

constexpr Permission operator|(Permission lhs, Permission rhs) {
    using Underlying = std::underlying_type_t<Permission>;

    return static_cast<Permission>(
        static_cast<Underlying>(lhs) |
        static_cast<Underlying>(rhs)
    );
}
```

پس از آن می‌توان از ترکیب flagها استفاده کرد:

```cpp
auto permissions = Permission::Read | Permission::Write;
```

در پروژه‌های بزرگ بهتر است چنین operatorهایی فقط برای enumerationهایی تعریف شوند که واقعاً به‌عنوان bitmask طراحی شده‌اند. نباید هر `enum class` را صرفاً به دلیل وجود underlying integer به bit flags تبدیل کرد.

## طراحی enum class به‌عنوان bitmask

اگر enumeration برای bitmask طراحی می‌شود، بهتر است این موضوع در طراحی type کاملاً مشخص باشد.

```cpp
enum class FileAccess : unsigned int {
    None    = 0,
    Read    = 1u << 0,
    Write   = 1u << 1,
    Execute = 1u << 2
};
```

در این مدل هر مقدار مستقل باید معمولاً یک bit مجزا داشته باشد.

مقادیر ترکیبی را نیز می‌توان به‌صورت enumerator تعریف کرد:

```cpp
enum class FileAccess : unsigned int {
    None       = 0,
    Read       = 1u << 0,
    Write      = 1u << 1,
    Execute    = 1u << 2,
    ReadWrite  = Read | Write
};
```

اما این شکل به‌دلیل type safety نیازمند دقت در expressionهای initializer است و در طراحی‌های پیچیده بهتر است operatorهای bitwise و helperهای لازم به‌صورت شفاف تعریف شوند.

## استفاده از underlying type

نوع زیرین enumeration را می‌توان با انواع integer مشخص کرد:

```cpp
enum class State : std::uint8_t {
    Idle,
    Running,
    Stopped
};
```

این کار می‌تواند برای serialization یا ساختارهایی که layout مشخصی دارند مفید باشد.

با این حال، underlying type نباید به‌عنوان راهی برای دور زدن type safety در نظر گرفته شود.

برای دسترسی به آن می‌توان از `std::underlying_type_t` استفاده کرد:

```cpp
#include <type_traits>

enum class Status : std::uint8_t {
    Pending,
    Completed
};

using StatusValue = std::underlying_type_t<Status>;
```

در این مثال `StatusValue` همان نوع زیرین `Status` خواهد بود.

## تبدیل enum class به integer

برای تبدیل صریح می‌توان از `static_cast` استفاده کرد:

```cpp
enum class Status {
    Pending = 10,
    Completed = 20
};

Status status = Status::Completed;

int value = static_cast<int>(status);
```

این تبدیل باید آگاهانه انجام شود، زیرا با تبدیل enumeration به integer بخشی از abstraction نوع از بین می‌رود.

اگر هدف serialization است، بهتر است این تبدیل در یک لایهٔ مشخص انجام شود، نه اینکه در سراسر برنامه پراکنده باشد.

## تبدیل integer به enum class

تبدیل معکوس نیز با `static_cast` ممکن است:

```cpp
int value = 20;

Status status = static_cast<Status>(value);
```

اما این conversion اعتبار مقدار را بررسی نمی‌کند.

بنابراین اگر `value` از کاربر، فایل، network یا یک سیستم خارجی آمده باشد، باید قبل از استفاده validation انجام شود.

## enum class و constexpr

مقادیر enumeration در contextهای compile-time نیز قابل استفاده هستند.

```cpp
enum class Status {
    Pending,
    Running,
    Completed
};

constexpr Status defaultStatus = Status::Pending;
```

همچنین می‌توان از آن‌ها در بسیاری از contextهایی که به constant expression نیاز دارند استفاده کرد:

```cpp
enum class BufferSize {
    Small = 128,
    Large = 1024
};

std::array<char, static_cast<std::size_t>(BufferSize::Small)> buffer{};
```

توجه کنید که برای تبدیل enumeration به اندازهٔ عددی از conversion صریح استفاده شده است.

## enum class و templateها

از آنجا که `enum class` یک type واقعی است، می‌توان آن را در templateها نیز استفاده کرد.

```cpp
enum class Color {
    Red,
    Green,
    Blue
};

template <Color Value>
struct ColorTag {
    static constexpr Color value = Value;
};

using RedTag = ColorTag<Color::Red>;
```

این قابلیت در طراحی‌های compile-time، policy-based design و metaprogramming می‌تواند مفید باشد.

## enum class و specialization

Enumerationها می‌توانند به‌عنوان template argument استفاده شوند و برای هر مقدار specialization متفاوتی تعریف شود.

```cpp
enum class Operation {
    Read,
    Write
};

template <Operation>
struct OperationTraits;

template <>
struct OperationTraits<Operation::Read> {
    static constexpr bool modifiesData = false;
};

template <>
struct OperationTraits<Operation::Write> {
    static constexpr bool modifiesData = true;
};
```

در این الگو، `enum class` به‌عنوان بخشی از type-level design استفاده می‌شود.

## enum class و class

اگرچه `enum class` شامل کلمهٔ `class` است، اما نباید آن را با یک `class` معمولی اشتباه گرفت.

```cpp
enum class Status {
    Pending,
    Completed
};
```

این declaration یک enumeration تعریف می‌کند، نه یک class دارای member function و data member.

عبارت `class` در این syntax بیشتر نشان‌دهندهٔ scoped بودن enumeration است.

همین ویژگی در syntax زیر نیز وجود دارد:

```cpp
enum struct Status {
    Pending,
    Completed
};
```

## enum class و enum struct

از نظر semantics، دو تعریف زیر معادل هستند:

```cpp
enum class Status {
    Pending,
    Completed
};
```

و:

```cpp
enum struct Status {
    Pending,
    Completed
};
```

هر دو:

* scoped هستند؛
* type-safe هستند؛
* implicit conversion به integer ندارند؛
* اعضای خود را در scope enumeration قرار می‌دهند؛
* از underlying type صریح پشتیبانی می‌کنند.

تفاوت اصلی در syntax و convention است.

برای consistency در یک codebase بهتر است تیم یک convention مشخص داشته باشد. در بیشتر پروژه‌های ++C، فرم `enum class` انتخاب رایج‌تری است.

## enum class و forward declaration

در صورت مشخص بودن underlying type می‌توان enumeration را forward declare کرد.

```cpp
enum class Status : std::uint8_t;
```

سپس می‌توان آن را بعداً تعریف کرد:

```cpp
enum class Status : std::uint8_t {
    Pending,
    Completed
};
```

مشخص بودن underlying type در forward declaration اهمیت دارد.

این تکنیک می‌تواند در headerهای بزرگ برای کاهش coupling مفید باشد.

## enum class در headerها

وقتی enumeration بخشی از public API است، تعریف آن معمولاً در header قرار می‌گیرد.

```cpp
#pragma once

enum class ConnectionState {
    Disconnected,
    Connecting,
    Connected
};
```

سپس سایر بخش‌های برنامه می‌توانند همان type را استفاده کنند.

برای typeهایی که بین چند component مشترک هستند، نام‌گذاری و scope مناسب اهمیت زیادی دارد. قرار دادن enumeration در namespace مناسب نیز از collision جلوگیری می‌کند.

## استفاده از namespace

می‌توان enumeration را داخل namespace قرار داد:

```cpp
namespace network {

enum class ConnectionState {
    Disconnected,
    Connecting,
    Connected
};

}
```

اکنون استفاده به شکل زیر خواهد بود:

```cpp
network::ConnectionState state =
    network::ConnectionState::Connected;
```

ترکیب namespace و scoped enumeration یکی از روش‌های مناسب برای جلوگیری از آلودگی namespace در پروژه‌های بزرگ است.

## enum class و نام‌گذاری

نام‌گذاری enumeration باید نمایانگر domain آن باشد.

برای مثال:

```cpp
enum class ConnectionState {
    Disconnected,
    Connecting,
    Connected
};
```

این طراحی از نام‌های بیش از حد عمومی مانند موارد زیر بهتر است:

```cpp
// Avoid overly generic names.
enum class State {
    Start,
    Stop
};
```

البته نام مناسب به domain و scope واقعی پروژه بستگی دارد.

در `enum class` نیازی نیست برای جلوگیری از collision به نام‌هایی مانند `StateConnected` یا `StateDisconnected` پناه ببرید، زیرا enumeratorها scoped هستند.

## انتخاب enum class یا enum سنتی

در کد مدرن ++C، `enum class` معمولاً انتخاب پیش‌فرض مناسب‌تری است.

استفاده از `enum class` معمولاً زمانی توصیه می‌شود که:

* type safety اهمیت دارد؛
* مقادیر یک domain مشخص را مدل می‌کنند؛
* نمی‌خواهید enumeratorها به scope بیرونی وارد شوند؛
* نمی‌خواهید conversion ضمنی به integer اتفاق بیفتد؛
* API باید self-documenting باشد.

`enum` سنتی ممکن است در برخی کدهای قدیمی، APIهای موجود، یا شرایطی که conversionهای ضمنی legacy بخشی از طراحی هستند لازم باشد.

اما نباید صرفاً برای کوتاه‌تر شدن syntax از `enum` سنتی استفاده کرد.

## محدودیت‌های enum class

با وجود مزایای زیاد، `enum class` همهٔ نیازهای طراحی را حل نمی‌کند.

یکی از محدودیت‌ها این است که enumeration به‌صورت خودکار string representation مناسبی ندارد.

برای مثال، از:

```cpp
enum class Status {
    Pending,
    Completed
};
```

نمی‌توان به‌صورت استاندارد انتظار داشت که `Status::Pending` خودکار به `"Pending"` تبدیل شود.

اگر logging، UI یا serialization به نام متنی نیاز دارد، باید mapping مناسبی طراحی شود.

```cpp
std::string_view toString(Status status) {
    switch (status) {
        case Status::Pending:
            return "Pending";
        case Status::Completed:
            return "Completed";
    }

    return "Unknown";
}
```

این رویکرد همچنین امکان کنترل رفتار برای مقادیر نامعتبر را فراهم می‌کند.

## طراحی تبدیل enum class به string

یک تابع تبدیل می‌تواند API مناسبی برای logging یا serialization ایجاد کند:

```cpp
#include <string_view>

enum class Status {
    Pending,
    Running,
    Completed
};

constexpr std::string_view toString(Status status) {
    switch (status) {
        case Status::Pending:
            return "Pending";
        case Status::Running:
            return "Running";
        case Status::Completed:
            return "Completed";
    }

    return "Unknown";
}
```

قرار دادن این mapping در یک نقطهٔ مشخص بهتر از پخش کردن `switch`های مشابه در بخش‌های مختلف برنامه است.

## خطاهای رایج

یکی از خطاهای رایج تلاش برای استفاده از enumerator بدون scope است:

```cpp
enum class Color {
    Red,
    Blue
};

// Color color = Red;
```

شکل صحیح استفاده چنین است:

```cpp
Color color = Color::Red;
```

خطای رایج دیگر انتظار conversion ضمنی به integer است:

```cpp
enum class Status {
    Pending,
    Completed
};

// int value = Status::Pending;
```

در صورت نیاز باید conversion صریح انجام شود:

```cpp
int value = static_cast<int>(Status::Pending);
```

خطای دیگری که در serialization رخ می‌دهد، فرض کردن معتبر بودن هر integer پس از conversion است:

```cpp
int value = readValue();
Status status = static_cast<Status>(value);
```

این کد به‌تنهایی validation انجام نمی‌دهد.

## نکات مربوط به switch

هنگام استفاده از `switch` روی `enum class` بهتر است حالت‌های ممکن به‌طور آگاهانه بررسی شوند.

```cpp
switch (status) {
    case Status::Pending:
        break;

    case Status::Running:
        break;

    case Status::Completed:
        break;
}
```

استفاده از compiler warnings مناسب می‌تواند در پیدا کردن caseهای فراموش‌شده کمک کند.

با این حال، نباید صرفاً به warning compiler به‌عنوان validation runtime تکیه کرد؛ زیرا مقدار enumeration ممکن است از دادهٔ خارجی یا از یک cast نامعتبر حاصل شده باشد.

## enum class و داده‌های خارجی

یکی از مهم‌ترین ملاحظات طراحی زمانی است که مقدار enumeration از خارج از برنامه وارد می‌شود.

برای مثال، اگر یک protocol مقدار `0`، `1` و `2` را برای وضعیت‌ها تعریف کند، بهتر است مرز بین representation خارجی و type داخلی مشخص باشد.

```cpp
enum class Status : std::uint8_t {
    Pending = 0,
    Running = 1,
    Completed = 2
};
```

در لایهٔ parsing می‌توان مقدار خام را بررسی کرد و سپس به type داخلی تبدیل کرد.

این separation باعث می‌شود بخش‌های داخلی برنامه کمتر به representation خارجی وابسته باشند.

## enum class و serialization

تعیین underlying type و مقادیر صریح می‌تواند برای protocolهای پایدار مفید باشد:

```cpp
enum class MessageType : std::uint8_t {
    Request = 1,
    Response = 2,
    Error = 3
};
```

اما صرفاً مشخص کردن underlying type، serialization قابل‌حمل را تضمین نمی‌کند.

برای protocolهای binary باید مسائلی مانند endianess، اندازهٔ دقیق داده، versioning و validation نیز در نظر گرفته شوند.

بنابراین `enum class` تنها بخشی از طراحی serialization است.

## enum class و ABI

اگر یک enumeration در public ABI استفاده می‌شود، تغییر underlying type یا مقادیر آن می‌تواند پیامدهای compatibility داشته باشد.

برای مثال، اگر مقدار enumeration بخشی از یک binary protocol یا interface پایدار باشد، تغییر مقادیر آن ممکن است برنامه‌های موجود را دچار مشکل کند.

در چنین شرایطی بهتر است underlying type و مقادیر مهم به‌صورت صریح طراحی شوند و تغییر آن‌ها با سیاست versioning پروژه هماهنگ باشد.

## enum class و حافظه

اندازهٔ object مربوط به enumeration به underlying type و implementation وابسته است، مگر اینکه underlying type به‌صورت مشخص تعیین شده باشد.

اگر اندازهٔ دقیق برای layout یا serialization اهمیت دارد، آن را صریح مشخص کنید:

```cpp
enum class PacketType : std::uint8_t {
    Data,
    Control,
    Error
};
```

اما در code عادی، نباید بدون نیاز واقعی underlying type کوچک انتخاب کرد.

انتخاب نوع بسیار کوچک ممکن است در برخی contextها باعث promotionهای عددی شود و لزوماً به معنای ساده‌تر یا سریع‌تر شدن همهٔ عملیات نیست.

## enum class و operatorهای سفارشی

می‌توان operatorهای مختلفی برای `enum class` تعریف کرد، اما این کار باید با semantic واقعی type سازگار باشد.

برای example، اگر type واقعاً یک مجموعهٔ flag است، تعریف `operator|` منطقی است.

اما تعریف operatorهای عددی مانند `+` برای یک status enumeration معمولاً طراحی مناسبی نیست:

```cpp
enum class Status {
    Pending,
    Running,
    Completed
};
```

این type یک مقدار domain را نشان می‌دهد، نه یک عدد قابل جمع.

بنابراین type safety فقط یک ویژگی compiler نیست؛ بخشی از طراحی semantic نوع نیز محسوب می‌شود.

## انتخاب underlying type مناسب

برای انتخاب underlying type می‌توان چند سناریو را در نظر گرفت.

در application code معمولاً می‌توان انتخاب نوع را به compiler واگذار کرد:

```cpp
enum class Status {
    Pending,
    Running,
    Completed
};
```

در protocolها یا ساختارهای دارای layout مشخص، انتخاب صریح می‌تواند مناسب باشد:

```cpp
enum class Status : std::uint8_t {
    Pending = 0,
    Running = 1,
    Completed = 2
};
```

در هر دو حالت، هدف باید روشن باشد. تعیین underlying type بدون نیاز مشخص ممکن است صرفاً complexity بیشتری ایجاد کند.

## تفاوت enum class با enum برای API

برای یک API جدید، این طراحی:

```cpp
enum class Mode {
    Read,
    Write
};

void open(Mode mode);
```

معمولاً از این طراحی امن‌تر است:

```cpp
enum Mode {
    Read,
    Write
};

void open(Mode mode);
```

زیرا caller مجبور است از `Mode::Read` و `Mode::Write` استفاده کند و compiler نیز از بسیاری از conversionهای ناخواسته جلوگیری می‌کند.

همچنین وجود scope مشخص باعث می‌شود API خواناتر باشد:

```cpp
open(Mode::Read);
```

این عبارت اطلاعات بیشتری دربارهٔ نوع مقدار دارد.

## enum class در طراحی domain

یکی از بهترین کاربردهای `enum class` مدل‌سازی state و categoryهای محدود است.

```cpp
enum class OrderState {
    Created,
    Paid,
    Shipped,
    Delivered,
    Cancelled
};
```

اکنون type سیستم اجازه نمی‌دهد هر `int` به‌صورت ضمنی به‌جای `OrderState` استفاده شود.

این موضوع domain model را دقیق‌تر می‌کند و بسیاری از خطاهای رایج را زودتر آشکار می‌سازد.

## چه زمانی enum class انتخاب مناسبی نیست؟

اگر domain واقعاً مجموعه‌ای از مقادیر باز و قابل توسعه است، enumeration ممکن است abstraction مناسبی نباشد.

برای مثال، اگر یک برنامه با شناسهٔ عددی دلخواه سروکار دارد:

```cpp
using UserId = std::uint64_t;
```

استفاده از enumeration برای User ID مناسب نیست، زیرا تعداد مقادیر مجاز محدود و از قبل مشخص نیست.

همچنین اگر مقدارها رفتار پیچیده، state اضافی یا invariantهای متعددی دارند، ممکن است یک `class` یا `struct` domain-specific انتخاب مناسب‌تری باشد.

## بهترین روش‌های طراحی

در طراحی مدرن ++C می‌توان چند guideline عملی را در نظر گرفت:

* برای enumerationهای جدید معمولاً `enum class` را به `enum` ترجیح دهید.
* نام enumeration را متناسب با domain انتخاب کنید.
* به‌جای نام‌های بسیار عمومی، typeهای domain-specific ایجاد کنید.
* underlying type را فقط زمانی صریح مشخص کنید که نیاز مشخصی وجود داشته باشد.
* برای protocol و serialization مقادیر را آگاهانه طراحی کنید.
* داده‌های خارجی را قبل از تبدیل به enumeration validation کنید.
* برای bit flags، semantic نوع را از ابتدا مشخص کنید.
* operatorهای سفارشی را فقط زمانی اضافه کنید که با مفهوم type سازگار باشند.
* از conversionهای غیرضروری به integer اجتناب کنید.
* mapping مربوط به string و serialization را در یک لایهٔ مشخص نگه دارید.
* warningهای مناسب compiler را فعال کنید تا caseهای ناقص در `switch` راحت‌تر شناسایی شوند.

## جمع‌بندی

مفهوم `enum class` در ++C11 یک روش type-safe و scoped برای تعریف مجموعه‌ای محدود از مقادیر نام‌گذاری‌شده است. برخلاف `enum` سنتی، اعضای آن وارد scope بیرونی نمی‌شوند و conversion ضمنی آن به integer انجام نمی‌شود.

شکل `enum struct` نیز از نظر semantics با `enum class` معادل است و بیشتر یک انتخاب syntax محسوب می‌شود.

برای طراحی‌های مدرن، `enum class` انتخاب مناسبی برای مواردی مانند stateها، modeها، categoryها، error codeها و سایر domain valueهای محدود است. در عین حال، هنگام استفاده از آن برای serialization، protocolها، bit flags و داده‌های خارجی باید underlying type، اعتبارسنجی و compatibility را به‌صورت آگاهانه طراحی کرد.

در نهایت، ارزش اصلی `enum class` فقط نام‌گذاری چند مقدار نیست؛ بلکه ایجاد یک type مشخص و محدود است که بخشی از قرارداد domain را به سیستم type زبان منتقل می‌کند و در نتیجه بسیاری از خطاها را پیش از اجرای برنامه قابل شناسایی می‌سازد.


---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>