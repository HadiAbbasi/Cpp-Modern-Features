<div align="center">

[🇺🇸 English](../../en/cpp14/[[maybe_unused]].md) | [🇮🇷 فارسی](./[[maybe_unused]].md)

</div>

---

# معرفی ویژگی `[[maybe_unused]]` در ++C

ویژگی `[[maybe_unused]]` یکی از **standard attribute**های زبان ++C است که از استاندارد C++17 در دسترس قرار گرفته و برای اعلام این موضوع استفاده می‌شود که ممکن است یک entity عمداً استفاده نشود. هدف اصلی آن جلوگیری از هشدارهای compiler درباره unused entities است، بدون اینکه لازم باشد ساختار کد صرفاً برای ساکت کردن warning تغییر کند. ([Cppreference][1])

این ویژگی بخشی از **زبان ++C** است و به Standard Library تعلق ندارد؛ بنابراین برای استفاده از آن به هیچ header خاصی نیاز نیست.

```cpp
[[maybe_unused]]
int debug_value = 42;
```

در این مثال، اگر `debug_value` در هیچ جای برنامه استفاده نشود، compiler می‌تواند warning مربوط به unused variable را سرکوب کند.

## مفهوم warning مربوط به unused entity

هشدارهای مربوط به unused entities معمولاً زمانی ظاهر می‌شوند که compiler تشخیص دهد یک نام، متغیر، تابع، پارامتر یا entity دیگر تعریف شده اما در نهایت مورد استفاده قرار نگرفته است.

برای مثال:

```cpp
void process(int value)
{
    // The parameter is intentionally unused.
}
```

بسته به warningهای فعال compiler، پارامتر `value` ممکن است باعث هشدار شود.

وجود چنین warningهایی معمولاً مفید است؛ زیرا یک متغیر واقعاً unused می‌تواند نشانه‌ای از bug، refactoring ناقص یا اشتباه در طراحی باشد. بااین‌حال، همه موارد unused بودن اشتباه نیستند.

برای نمونه، ممکن است یک پارامتر فقط برای سازگاری با یک interface وجود داشته باشد:

```cpp
class Handler
{
public:
    void on_event(int event_id)
    {
        // The interface requires the parameter.
    }
};
```

در چنین شرایطی `[[maybe_unused]]` امکان بیان صریح intent برنامه‌نویس را فراهم می‌کند.

## نحوه استفاده از `[[maybe_unused]]`

نحو اصلی این attribute به شکل زیر است:

```cpp
[[maybe_unused]] declaration;
```

این attribute می‌تواند در declarationهای مختلفی استفاده شود و از C++17 بخشی از استاندارد زبان است. ([Cppreference][1])

یک کاربرد ساده برای متغیر به صورت زیر است:

```cpp
[[maybe_unused]] int value = 42;
```

همچنین می‌توان attribute را در declaration متغیر بعد از نام آن قرار داد:

```cpp
int value [[maybe_unused]] = 42;
```

هر دو شکل از نظر هدف یکسان هستند، اما در یک codebase بهتر است convention مشخصی انتخاب شود تا سبک کد یکدست بماند.

## کاربرد برای متغیرها

رایج‌ترین کاربرد `[[maybe_unused]]` مربوط به local variableها و objectها است.

```cpp
void initialize()
{
    [[maybe_unused]] const auto configuration = load_configuration();
}
```

در این مثال ممکن است مقدار `configuration` در برخی build configurationها استفاده شود و در برخی دیگر استفاده نشود.

یک سناریوی متداول‌تر زمانی رخ می‌دهد که کد بین debug و release متفاوت رفتار کند:

```cpp
void process()
{
    [[maybe_unused]] const auto result = calculate_result();

#ifdef ENABLE_LOGGING
    log_result(result);
#endif
}
```

در buildهایی که `ENABLE_LOGGING` فعال نیست، `result` ممکن است unused باشد.

با این حال، استفاده از attribute نباید بهانه‌ای برای باقی گذاشتن متغیرهایی باشد که واقعاً دیگر مورد نیاز نیستند.

## کاربرد برای پارامترهای تابع

یکی از مهم‌ترین کاربردهای این attribute مربوط به function parameterها است.

```cpp
void callback([[maybe_unused]] int event_id)
{
    notify_observer();
}
```

در اینجا `event_id` ممکن است بخشی از یک callback interface باشد، اما implementation فعلی به آن نیازی نداشته باشد.

این تکنیک در مواردی مانند interface implementation، callbackها، framework APIها و template code بسیار مفید است.

برای چند پارامتر نیز می‌توان attribute را مستقل از یکدیگر اعمال کرد:

```cpp
void callback(
    [[maybe_unused]] int event_id,
    [[maybe_unused]] int flags)
{
    notify_observer();
}
```

قرار دادن attribute فقط روی پارامترهایی که واقعاً unused هستند، معمولاً intent را واضح‌تر می‌کند.

## کاربرد برای تابع‌ها

خود function declaration نیز می‌تواند با `[[maybe_unused]]` علامت‌گذاری شود:

```cpp
[[maybe_unused]] void debug_dump()
{
    // Debug-only implementation.
}
```

این روش زمانی مفید است که وجود تابع وابسته به configuration، build mode یا platform باشد.

برای مثال:

```cpp
[[maybe_unused]] void trace_message(const char* message)
{
    write_trace(message);
}
```

اگر در یک build خاص این تابع هیچ call site نداشته باشد، attribute می‌تواند warning مربوط به unused function را سرکوب کند.

بااین‌حال، اگر تابع واقعاً دیگر استفاده نمی‌شود و هیچ دلیل معماری برای باقی ماندن آن وجود ندارد، حذف آن معمولاً بهتر از استفاده از `[[maybe_unused]]` است.

## کاربرد برای typeها

این attribute می‌تواند روی بعضی type declarationها نیز اعمال شود.

```cpp
struct [[maybe_unused]] DebugState
{
    int counter;
};
```

اگر type فقط در بعضی configurationها مورد استفاده قرار گیرد، این کاربرد می‌تواند از warningهای مربوط به unused type جلوگیری کند.

همین قابلیت برای `enum` نیز وجود دارد:

```cpp
enum [[maybe_unused]] class LogLevel
{
    info,
    warning,
    error
};
```

در چنین مواردی attribute روی خود entity قرار می‌گیرد، نه روی اعضای آن.

## کاربرد برای enumeratorها

حتی یک enumerator مشخص نیز می‌تواند با `[[maybe_unused]]` علامت‌گذاری شود.

```cpp
enum class Status
{
    success,
    failure,
    legacy [[maybe_unused]]
};
```

این روش زمانی مفید است که مجموعه‌ای از enumeratorها برای ABI، protocol، compatibility یا interface خارجی حفظ شده باشند، اما یک مقدار در implementation فعلی استفاده نشود.

توجه کنید که unused بودن یک enumerator با unused بودن خود enum دو مفهوم متفاوت است.

## کاربرد برای `typedef` و alias

در C++17، `[[maybe_unused]]` می‌تواند روی `typedef` و alias declaration نیز اعمال شود. ([Cppreference][1])

برای نمونه:

```cpp
using FileHandle [[maybe_unused]] = int;
```

این حالت در کدهای generic، platform abstraction و headerهایی که برای چند configuration استفاده می‌شوند می‌تواند مفید باشد.

## کاربرد برای data member

این attribute برای non-static data member نیز قابل استفاده است:

```cpp
struct State
{
    int active;

    [[maybe_unused]] int debug_counter;
};
```

این کاربرد نسبت به local variableها کمتر رایج است، اما در typeهایی که بخشی از layout یا interface آن‌ها به دلایل دیگری حفظ شده است، می‌تواند مناسب باشد.

برای static data member نیز می‌توان از آن استفاده کرد:

```cpp
struct Statistics
{
    [[maybe_unused]] static int debug_counter;
};
```

## کاربرد برای structured binding

از `[[maybe_unused]]` می‌توان برای structured binding نیز استفاده کرد:

```cpp
#include <utility>

void process()
{
    [[maybe_unused]] auto [width, height] = std::pair{1920, 1080};
}
```

همچنین می‌توان attribute را برای bindingهای مشخص مورد استفاده قرار داد:

```cpp
#include <utility>

void process()
{
    auto [width, height [[maybe_unused]]] = std::pair{1920, 1080};
}
```

پشتیبانی استاندارد از `[[maybe_unused]]` برای structured bindings در C++17 با یک defect report اصلاح شد؛ بنابراین مستندات امروزی این قابلیت را برای C++17 در نظر می‌گیرند. ([Cppreference][1])

## تفاوت `[[maybe_unused]]` با حذف متغیر

اگر متغیری واقعاً هیچ کاربردی ندارد، اولین سؤال نباید این باشد که چگونه warning آن را suppress کنیم.

برای مثال:

```cpp
void process()
{
    [[maybe_unused]] int unused_value = expensive_calculation();
}
```

ممکن است این کد warning نداشته باشد، اما همچنان `expensive_calculation()` اجرا می‌شود.

بنابراین `[[maybe_unused]]` **رفتار runtime را حذف نمی‌کند** و باعث نمی‌شود initialization انجام نشود.

اگر مقدار واقعاً مورد نیاز نیست، حذف کامل آن ممکن است انتخاب صحیح‌تری باشد:

```cpp
void process()
{
    perform_processing();
}
```

این تفاوت بسیار مهم است: `[[maybe_unused]]` درباره **diagnostic** است، نه درباره optimization یا حذف execution.

## تفاوت `[[maybe_unused]]` با cast به `void`

قبل از C++17 یکی از روش‌های متداول برای جلوگیری از warning مربوط به unused parameter این بود که مقدار به `void` cast شود:

```cpp
void callback(int event_id)
{
    static_cast<void>(event_id);
}
```

این روش هنوز معتبر است، اما `[[maybe_unused]]` intent را مستقیم‌تر بیان می‌کند:

```cpp
void callback([[maybe_unused]] int event_id)
{
}
```

در روش دوم، declaration به خواننده می‌گوید که unused بودن این parameter عمدی است.

در مقابل، `static_cast<void>(event_id)` یک expression واقعی در function body ایجاد می‌کند و بیشتر شبیه workaround برای warning است.

برای کد مدرن ++C، وقتی هدف صرفاً بیان intentional unused بودن یک entity است، `[[maybe_unused]]` معمولاً معنای واضح‌تری دارد.

## تفاوت `[[maybe_unused]]` با `std::ignore`

در بعضی کدها `std::ignore` برای نادیده گرفتن یک مقدار استفاده می‌شود:

```cpp
#include <tuple>

void process()
{
    auto [value, status] = get_result();
    std::ignore = status;
}
```

بااین‌حال، `std::ignore` اساساً برای استفاده همراه با `std::tie` طراحی شده است و کاربردهای دیگری نیز دارد. ([Cppreference][2])

اگر هدف فقط این باشد که یک named variable عمداً unused باشد، `[[maybe_unused]]` از نظر بیان intent مستقیم‌تر است:

```cpp
void process()
{
    [[maybe_unused]] auto [value, status] = get_result();
}
```

در مقابل، زمانی که واقعاً قصد دارید یک مقدار را در unpacking نادیده بگیرید، `std::ignore` می‌تواند انتخاب مناسب‌تری باشد.

## تفاوت `[[maybe_unused]]` با نادیده گرفتن return value

یکی از اشتباهات مهم این است که تصور کنیم `[[maybe_unused]]` راه‌حل عمومی برای discarded return value است.

فرض کنید تابعی با `[[nodiscard]]` تعریف شده باشد:

```cpp
[[nodiscard]] bool save();

void process()
{
    save();
}
```

در اینجا مشکل مربوط به **نادیده گرفتن return value** است، نه unused بودن یک declaration.

اگر قصد شما واقعاً این است که return value را آگاهانه نادیده بگیرید، راه‌حل باید متناسب با API و intent انتخاب شود.

برای مثال:

```cpp
void process()
{
    static_cast<void>(save());
}
```

این expression صریحاً نشان می‌دهد که discarded result عمدی است.

بنابراین `[[maybe_unused]]` را نباید جایگزین عمومی برای مدیریت `[[nodiscard]]` دانست.

## تفاوت با `[[nodiscard]]`

ویژگی `[[nodiscard]]` و `[[maybe_unused]]` تقریباً در جهت مخالف یکدیگر عمل می‌کنند.

ویژگی `[[nodiscard]]` به compiler می‌گوید که **نادیده گرفتن نتیجه یک entity مهم است و باید درباره آن هشدار داده شود**.

```cpp
[[nodiscard]] bool commit();
```

در مقابل، `[[maybe_unused]]` به compiler می‌گوید که **unused بودن یک entity مشخص می‌تواند عمدی باشد و نباید warning مربوط به آن صادر شود**.

بنابراین این دو attribute مکمل یکدیگر هستند، نه جایگزین یکدیگر.

## محدود کردن استفاده به declaration مورد نیاز

یکی از اصول مهم استفاده از این attribute، محدود کردن دامنه آن است.

برای مثال، اگر فقط یک parameter unused است:

```cpp
void callback([[maybe_unused]] int event_id, int flags)
{
    use_flags(flags);
}
```

بهتر است فقط همان parameter علامت‌گذاری شود، نه اینکه کل function بدون دلیل با attribute مشخص شود.

این کار intent را دقیق‌تر می‌کند و باعث می‌شود warningهای واقعی همچنان قابل مشاهده باشند.

## کاربرد در conditional compilation

یکی از کاربردهای عملی مهم `[[maybe_unused]]` در کدهایی است که با `#if`، feature flags یا platform-specific compilation ساخته می‌شوند.

```cpp
void process()
{
    const int value = calculate_value();

#ifdef FEATURE_X
    use_feature_value(value);
#else
    (void)value;
#endif
}
```

در طراحی مدرن می‌توان declaration را طوری علامت‌گذاری کرد که در buildهای مختلف warning ایجاد نکند:

```cpp
void process()
{
    [[maybe_unused]] const int value = calculate_value();

#ifdef FEATURE_X
    use_feature_value(value);
#endif
}
```

این تکنیک مخصوصاً در headerها و codebaseهایی که برای چند platform یا configuration build می‌شوند مفید است.

## کاربرد در API و interface

فرض کنید interface یک framework پارامتر خاصی را به callback ارسال می‌کند، اما implementation فعلی نیازی به آن ندارد:

```cpp
class EventHandler
{
public:
    void handle(
        [[maybe_unused]] int event_id,
        const char* message)
    {
        print_message(message);
    }
};
```

در اینجا حذف پارامتر امکان‌پذیر نیست، زیرا signature بخشی از contract است.

استفاده از `[[maybe_unused]]` در این وضعیت intent را به شکل مستقیم بیان می‌کند.

## کاربرد در templateها

در template code نیز ممکن است یک پارامتر یا متغیر فقط در بعضی instantiationها استفاده شود.

```cpp
template <typename T>
void process(T value)
{
    [[maybe_unused]] auto size = sizeof(T);

    use(value);
}
```

البته باید توجه داشت که `[[maybe_unused]]` نباید برای پنهان کردن طراحی ضعیف template استفاده شود.

اگر یک entity در بسیاری از instantiationها unused است، ممکن است ساختار template نیاز به بازنگری داشته باشد.

## کاربرد در کدهای Debug و Release

یک سناریوی بسیار رایج، متغیری است که فقط در debug build استفاده می‌شود.

```cpp
void process()
{
    auto result = calculate();

#ifndef NDEBUG
    assert(result.valid());
#endif

    submit(result);
}
```

در این مثال `result` همچنان در release build استفاده می‌شود، بنابراین نیازی به `[[maybe_unused]]` وجود ندارد.

اما اگر variable فقط برای assertion وجود داشته باشد:

```cpp
void process()
{
    [[maybe_unused]] const auto invariant = calculate_invariant();

#ifndef NDEBUG
    assert(invariant);
#endif
}
```

در release build ممکن است variable unused شود و attribute می‌تواند این وضعیت را به شکل صریح بیان کند.

بااین‌حال، بهتر است ابتدا بررسی شود که آیا محاسبه‌ای که فقط برای debug انجام می‌شود خودش هزینه runtime غیرضروری ایجاد نمی‌کند.

## محدودیت مهم در runtime behavior

ویژگی `[[maybe_unused]]` هیچ تضمینی درباره حذف کد، حذف object یا کاهش هزینه runtime ارائه نمی‌دهد.

برای مثال:

```cpp
[[maybe_unused]] auto value = expensive_operation();
```

این attribute به تنهایی نمی‌گوید که compiler باید `expensive_operation()` را حذف کند.

اگر expression دارای side effect باشد، حذف آن می‌تواند semantics برنامه را تغییر دهد و compiler نیز نمی‌تواند صرفاً به دلیل `[[maybe_unused]]` آن را حذف کند.

بنابراین باید بین دو مفهوم تفاوت گذاشت:

* **unused diagnostic**: entity مورد استفاده قرار نگرفته است.
* **dead-code elimination**: compiler تشخیص می‌دهد بخشی از کد بدون observable effect است و می‌تواند آن را حذف کند.

`[[maybe_unused]]` فقط به مفهوم اول مربوط است.

## محدودیت در برابر bugهای واقعی

یکی از خطرهای استفاده بیش از حد از این attribute، مخفی کردن warningهای مفید است.

فرض کنید refactoring انجام شده و یک variable دیگر مورد استفاده نیست:

```cpp
void process()
{
    [[maybe_unused]] auto result = calculate_result();

    send_default_response();
}
```

ممکن است developer برای حذف warning به‌سرعت `[[maybe_unused]]` اضافه کند، در حالی که `result` باید در منطق جدید استفاده می‌شده است.

در چنین شرایطی attribute مشکل اصلی را حل نکرده، بلکه warning مفید را پنهان کرده است.

قاعده عملی مناسب این است که ابتدا دلیل unused بودن را پیدا کنید و فقط در صورت عمدی بودن آن از attribute استفاده کنید.

## تفاوت رفتار compilerها

`[[maybe_unused]]` یک **standard attribute** است و compilerهای مدرن ++C آن را به عنوان بخشی از زبان استاندارد پشتیبانی می‌کنند.

بااین‌حال، اینکه دقیقاً کدام warning توسط compiler صادر می‌شود، جزئیات warning system همان compiler است.

برای مثال، فعال بودن warningهای مربوط به unused entities می‌تواند با flagهای compiler تغییر کند.

در نتیجه نباید این دو موضوع با یکدیگر مخلوط شوند:

* استاندارد ++C مشخص می‌کند `[[maybe_unused]]` چه معنایی دارد.
* compiler مشخص می‌کند چه warningهایی درباره unused entities تولید می‌کند و سطح آن warningها چگونه تنظیم می‌شود.

این attribute یک standard mechanism برای اعلام intent است، اما policy مربوط به diagnostics همچنان compiler-specific است.

## تفاوت با attributeهای compiler-specific

قبل از استاندارد شدن `[[maybe_unused]]`، compilerهای مختلف روش‌های اختصاصی خودشان را داشتند.

برای نمونه، GNU toolchain دارای attributeهای compiler-specific برای چنین کاربردهایی است.

اما در کد قابل حمل مدرن ++C، وقتی قابلیت استاندارد در دسترس است، استفاده از:

```cpp
[[maybe_unused]] int value = 42;
```

معمولاً نسبت به راهکارهای vendor-specific ترجیح داده می‌شود.

استفاده از compiler extension ممکن است در پروژه‌ای که عمداً به یک compiler خاص وابسته است منطقی باشد، اما نباید آن را با قابلیت استاندارد زبان اشتباه گرفت.

## تفاوت نسخه‌های استاندارد

ویژگی `[[maybe_unused]]` در **C++17** استاندارد شد. ([Cppreference][1])

در نسخه‌های بعدی استاندارد، این attribute همچنان بخشی از زبان باقی مانده است.

در نتیجه، برای پروژه‌هایی که C++17 یا نسخه جدیدتر را هدف قرار می‌دهند، استفاده از آن راهکار استاندارد و قابل حملی است.

در پروژه‌های قبل از C++17، باید از راهکارهای سازگار با compiler و استاندارد هدف پروژه استفاده کرد؛ برای مثال، در برخی codebaseهای قدیمی ممکن است از `static_cast<void>(value)` یا macroهای وابسته به compiler استفاده شده باشد.

## تغییرات مرتبط با C++26

استانداردهای جدید دامنه کاربرد attributeها را در بعضی موارد گسترش می‌دهند. در C++26، `[[maybe_unused]]` علاوه بر entityهای قبلی می‌تواند برای labelهای unused نیز استفاده شود. ([Cppreference][1])

برای نمونه:

```cpp
void process(bool condition)
{
    if (condition)
        goto done;

    [[maybe_unused]] done:
    return;
}
```

این قابلیت مخصوصاً در codebaseهایی که از labelها استفاده می‌کنند می‌تواند برای کنترل diagnostics مفید باشد.

بااین‌حال، وجود قابلیت جدید به معنای توصیه به استفاده از `goto` نیست؛ کاربرد این ویژگی صرفاً به handling مربوط به unused label محدود می‌شود.

## بررسی جایگاه attribute در declaration

یکی از نکات نحوی مهم این است که attribute می‌تواند در موقعیت‌های مختلف declaration ظاهر شود.

برای مثال:

```cpp
[[maybe_unused]] int value = 42;
```

یا:

```cpp
int value [[maybe_unused]] = 42;
```

هر دو فرم توسط زبان پشتیبانی می‌شوند، اما محل attribute باید مطابق grammar entity مورد نظر باشد.

در code review بهتر است شکل استفاده‌شده با style guide پروژه سازگار باشد.

## نکات مربوط به headerها

در headerها، `[[maybe_unused]]` می‌تواند برای declarationهایی که در برخی translation unitها استفاده نمی‌شوند مفید باشد.

برای مثال:

```cpp
[[maybe_unused]] inline void debug_log()
{
    // Debug logging implementation.
}
```

این موضوع به‌ویژه در utility headerهایی که برای چند configuration استفاده می‌شوند اهمیت دارد.

بااین‌حال، باید مراقب بود که attribute جایگزین طراحی صحیح برای `inline`، `static`، internal linkage یا conditional compilation نشود.

`[[maybe_unused]]` صرفاً diagnostic مربوط به unused بودن را کنترل می‌کند و linkage را تغییر نمی‌دهد.

## نکات مربوط به ODR و linkage

ویژگی `[[maybe_unused]]` هیچ تغییر مستقیمی در linkage یا One Definition Rule ایجاد نمی‌کند.

برای مثال:

```cpp
[[maybe_unused]] void helper()
{
}
```

این declaration صرفاً entity را به عنوان ممکن است unused باشد علامت‌گذاری می‌کند.

اگر هدف شما جلوگیری از multiple definition، کنترل linkage یا مدیریت visibility باشد، باید از مکانیزم مناسب همان مسئله استفاده کنید.

بنابراین نباید `[[maybe_unused]]` را با `static`، `inline` یا visibility attributes اشتباه گرفت.

## خطاهای رایج در استفاده

یکی از خطاهای رایج، استفاده از attribute برای هر warning مربوط به unused است، بدون اینکه دلیل واقعی unused بودن بررسی شود.

اشتباه دیگر این است که تصور شود attribute باعث حذف variable یا function از binary می‌شود.

```cpp
[[maybe_unused]] auto result = expensive_operation();
```

این declaration همچنان می‌تواند دارای initialization و side effect باشد.

اشتباه دیگر، استفاده از attribute روی entity اشتباه است. اگر مشکل discarded return value است، `[[maybe_unused]]` الزاماً راه‌حل مناسبی نیست.

اشتباه مهم دیگر، علامت‌گذاری بیش از حد declarationها است:

```cpp
[[maybe_unused]]
void process(
    [[maybe_unused]] int a,
    [[maybe_unused]] int b,
    [[maybe_unused]] int c)
{
}
```

اگر فقط `b` واقعاً unused است، علامت‌گذاری `a` و `c` نیز اطلاعات مفیدی درباره intent ارائه نمی‌کند.

## راهنمای انتخاب روش مناسب

برای انتخاب روش مناسب، ابتدا باید نوع مسئله مشخص شود.

اگر یک variable عمداً ممکن است unused باشد، `[[maybe_unused]]` گزینه مناسبی است:

```cpp
[[maybe_unused]] int value = 42;
```

اگر یک function parameter به دلیل interface ضروری است اما implementation به آن نیاز ندارد، استفاده از attribute روی parameter مناسب است:

```cpp
void callback([[maybe_unused]] int event_id)
{
}
```

اگر return value یک تابع عمداً discarded می‌شود، بهتر است intent درباره result به شکل مستقیم بیان شود:

```cpp
static_cast<void>(save());
```

اگر یک variable واقعاً دیگر مورد نیاز نیست، حذف آن معمولاً بهتر از suppress کردن warning است.

اگر warning ناشی از compiler-specific policy است، باید ابتدا مشخص شود که آیا راهکار استاندارد `[[maybe_unused]]` نیاز واقعی را برطرف می‌کند یا خیر.

## الگوی مناسب برای code review

در code review می‌توان درباره هر استفاده از `[[maybe_unused]]` یک سؤال ساده مطرح کرد:

> آیا unused بودن این entity بخشی عمدی از طراحی است؟

اگر پاسخ مثبت باشد، attribute معمولاً intent مناسبی را منتقل می‌کند.

اگر پاسخ منفی باشد، بهتر است علت unused بودن بررسی شود.

این رویکرد باعث می‌شود `[[maybe_unused]]` به ابزاری برای **مستندسازی intent** تبدیل شود، نه ابزاری برای خاموش کردن کورکورانه warningها.

## مثال جامع

مثال زیر چند کاربرد واقعی را در کنار هم نشان می‌دهد:

```cpp
#include <cassert>

enum class Status
{
    success,
    failure,
    legacy [[maybe_unused]]
};

struct [[maybe_unused]] DebugContext
{
    int counter{};
};

[[maybe_unused]] void trace_event(const char* message)
{
    // Emit diagnostic information.
}

void handle_event(
    [[maybe_unused]] int event_id,
    Status status)
{
    [[maybe_unused]] const bool valid =
        status == Status::success;

#ifndef NDEBUG
    assert(valid);
#endif
}
```

در این مثال، `[[maybe_unused]]` برای چند entity مختلف استفاده شده است، اما در هر مورد دلیل مشخصی وجود دارد:

* مقدار `legacy` برای compatibility نگه داشته شده است.
* `DebugContext` ممکن است فقط در بعضی configurationها استفاده شود.
* تابع `trace_event` می‌تواند در build خاصی بدون caller باشد.
* `event_id` بخشی از callback contract است.
* متغیر `valid` در debug build استفاده می‌شود و در release ممکن است unused شود.

چنین استفاده‌ای بسیار بهتر از قرار دادن attribute روی تعداد زیادی entity بدون دلیل مشخص است.

## نکات مهم برای پروژه‌های مدرن

برای پروژه‌های مدرن ++C بهتر است `[[maybe_unused]]` بخشی از strategy مربوط به diagnostics باشد، نه جایگزینی برای آن.

فعال بودن warningهای مناسب همچنان اهمیت زیادی دارد. هدف attribute این نیست که warningها را ضعیف کند؛ بلکه هدف آن جدا کردن **unused بودن عمدی** از **unused بودن مشکوک** است.

همچنین بهتر است attribute تا حد امکان نزدیک به entityای قرار گیرد که واقعاً ممکن است unused باشد. این کار intent را برای compiler و developer روشن‌تر می‌کند.

در library code نیز باید به portability توجه شود و تا حد امکان از standard attribute به جای extensionهای اختصاصی compiler استفاده شود.

## جمع‌بندی نهایی

ویژگی `[[maybe_unused]]` از C++17 یک ابزار استاندارد و ساده برای اعلام intentional unused بودن entityها است. این attribute می‌تواند روی مواردی مانند variable، parameter، function، type، enumerator، data member، alias و structured binding اعمال شود و در C++26 دامنه آن برای unused label نیز گسترش یافته است. ([Cppreference][1])

نکته اصلی این است که `[[maybe_unused]]` **رفتار برنامه را تغییر نمی‌دهد**؛ این ویژگی برای کنترل diagnostics مربوط به unused entities است. بنابراین نباید آن را با optimization، حذف dead code، `[[nodiscard]]`، `std::ignore` یا روش‌های compiler-specific برای suppress کردن warningها یکی دانست.

بهترین استفاده از این attribute زمانی است که unused بودن entity واقعاً عمدی باشد؛ مانند پارامترهای اجباری یک interface، کدهای وابسته به build configuration، debug-only state، compatibility entities و templateهایی که در برخی حالت‌ها به entity خاصی نیاز ندارند.

در مقابل، اگر unused بودن entity ناشی از bug یا refactoring ناقص است، استفاده از `[[maybe_unused]]` فقط warning مفید را پنهان می‌کند. بنابراین ارزش اصلی این ویژگی نه صرفاً حذف warning، بلکه **بیان دقیق intent برنامه‌نویس در کد** است.

[1]: https://en.cppreference.com/cpp/language/attributes/maybe_unused?utm_source=chatgpt.com "C++ attribute: maybe_unused (since C++17) - cppreference.com"
[2]: https://en.cppreference.com/cpp/utility/tuple/ignore?utm_source=chatgpt.com "std::ignore - cppreference.com"


---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>