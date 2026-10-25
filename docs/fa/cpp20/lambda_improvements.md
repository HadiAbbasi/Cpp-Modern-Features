<div align="center">

[🇺🇸 English](../../en/cpp20/lambda_improvements.md) | [🇮🇷 فارسی](./lambda_improvements.md)

</div>

---

# بهبودهای Lambda در ++C20

## مقدمه

عبارت‌های `lambda` از ++C11 راهی برای تعریف تابع‌های بدون نام و معمولاً کوتاه در همان نقطه‌ای هستند که به آن‌ها نیاز داریم. در ++C14، با اضافه شدن `generic lambda`، پارامترهای دارای `auto` امکان نوشتن تابع‌های قابل استفاده با چند نوع را فراهم کردند. استاندارد ++C20 این سازوکار را با قابلیت‌هایی مانند فهرست صریح پارامترهای قالب، محدودسازی با `requires`، استفاده از `lambda` در contextهای ارزیابی‌نشده، پشتیبانی بهتر از closureهای بدون capture و گسترش `init-capture` برای `parameter pack` کامل‌تر کرد.

هدف این بهبودها صرفاً کوتاه‌تر کردن کد نیست؛ آن‌ها اجازه می‌دهند `lambda`ها در generic programming، طراحی API، metaprogramming، الگوریتم‌های استاندارد و کدهای compile-time نقش دقیق‌تر و قابل‌کنترل‌تری داشته باشند.

## مروری بر سازوکار Lambda

عبارت `lambda` یک شیء از نوعی ناشناس می‌سازد که معمولاً به آن **closure object** گفته می‌شود. نوع این شیء، **closure type**، توسط کامپایلر تعریف می‌شود و شامل سازوکاری برای فراخوانی بدنه‌ی `lambda` است. اگر متغیرهای محیط با `capture` دریافت شوند، closure معمولاً داده‌هایی برای نگهداری آن‌ها دارد.

نمونه‌ی پایه:

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{1, 5, 2, 8, 3};

    auto is_even = [](int value) {
        return value % 2 == 0;
    };

    const auto count = std::count_if(values.begin(), values.end(), is_even);
    return count == 2 ? 0 : 1;
}
```

در این مثال، متغیر `is_even` شیئی از نوع closure است. این شیء مانند یک تابع قابل فراخوانی عمل می‌کند و چون هیچ متغیری را capture نمی‌کند، closure آن فاقد state ذخیره‌شده از محیط است.

### تفاوت Lambda معمولی و Generic Lambda

از ++C14 به بعد، اگر نوع یک یا چند پارامتر `auto` باشد، `lambda` یک `generic lambda` محسوب می‌شود. در این حالت، سازوکار فراخوانی closure یک function template است و نوع پارامترها هنگام فراخوانی استنتاج می‌شود.

```cpp
auto twice = [](auto value) {
    return value + value;
};

auto a = twice(10);    // int
 auto b = twice(2.5);  // double
```

در نمونه‌ی بالا، نوع پارامتر در هر فراخوانی جداگانه استنتاج می‌شود. با این حال، پیش از ++C20 کنترل مستقیم بر template parameterها و نام‌گذاری آن‌ها در syntax خود `lambda` وجود نداشت؛ قابلیت جدید این محدودیت را برطرف می‌کند.

## معرفی فهرست صریح پارامترهای قالب

مهم‌ترین تغییر در generic lambdaهای ++C20، امکان نوشتن **explicit template parameter list** است. این syntax به `lambda` اجازه می‌دهد پارامترهای قالب را مستقیماً اعلام کند؛ مشابه function templateهای معمولی.

```cpp
auto combine = []<typename T>(T left, T right) {
    return left + right;
};

auto result = combine(10, 20);
```

در این کد، هر دو پارامتر از یک `T` استفاده می‌کنند. این موضوع با `[](auto left, auto right)` تفاوت دارد؛ در شکل دوم، هر پارامتر دارای پارامتر قالب مستقل خود است و بنابراین می‌توان دو نوع متفاوت را نیز به آن داد، مشروط بر اینکه بدنه با آن انواع معتبر باشد.

### کاربردهایی که فهرست صریح فراهم می‌کند

فهرست صریح پارامترهای قالب برای زمانی مفید است که لازم باشد نوع پارامترها به‌صورت هماهنگ استنتاج شود، نوع در بدنه نام داشته باشد، پارامترهای قالب غیرنوعی به کار روند یا محدودیت‌های `Concepts` به شکل خواناتری نوشته شوند.

```cpp
#include <concepts>

auto add_integral = []<std::integral T>(T left, T right) {
    return left + right;
};

int main() {
    const auto sum = add_integral(10, 20);
    // add_integral(1.5, 2.5); // Does not satisfy std::integral
    return sum == 30 ? 0 : 1;
}
```

در این مثال، `T` باید مفهوم `std::integral` را برآورده کند. در نتیجه، محدودیت نوع بخشی از interface تابع است و خطاهای مربوط به نوع نامعتبر می‌توانند در فرایند overload resolution تشخیص داده شوند، نه اینکه فقط به خطای نامفهوم داخل بدنه منتهی شوند.

فهرست صریح می‌تواند پارامترهای قالب غیرنوعی نیز داشته باشد:

```cpp
auto scale = []<typename T, int Factor>(T value) {
    return value * Factor;
};

// A generic lambda's call operator is a member template.
// Explicit template arguments are supplied through operator().
int main() {
    auto value = scale.operator()<int, 3>(7);
    return value == 21 ? 0 : 1;
}
```

در ++C20 نمی‌توان پارامترهای قالب را مستقیماً با نوشتن `scale<int, 3>(7)` به شیء `lambda` داد؛ برای تعیین صریح آن‌ها باید template مربوط به عملگر فراخوانی عضو را صدا زد. در کد وابسته به template ممکن است به `template` برای رفع ابهام نحوی نیاز باشد.

### استنتاج نوع و انتخاب بین auto و T

استفاده از `auto` برای هر پارامتر معمولاً انتخاب ساده‌تر و منعطف‌تری است. در مقابل، پارامتر صریح `T` زمانی مناسب است که رابطه‌ی نوعی میان چند پارامتر اهمیت دارد یا لازم است در `requires` و بدنه به خود `T` ارجاع داده شود.

```cpp
auto independent = [](auto first, auto second) {
    return first == second;
};

auto same_type = []<typename T>(T first, T second) {
    return first == second;
};
```

در نمونه‌ی اول، `first` و `second` می‌توانند انواع متفاوتی داشته باشند، البته فقط در صورتی که عبارت مقایسه معتبر باشد. در نمونه‌ی دوم، استنتاج `T` از هر دو پارامتر باید با یک نوع سازگار شود؛ بنابراین بعضی فراخوانی‌های دارای انواع متفاوت رد می‌شوند.

این تفاوت را باید آگاهانه انتخاب کرد: استفاده از `T` به معنی «بهتر بودن» آن نیست، بلکه نوعی constraint روی نحوه‌ی فراخوانی ایجاد می‌کند.

## محدودسازی Lambda با Concepts و requires

استاندارد ++C20 اجازه می‌دهد `requires-clause` در generic lambda نوشته شود. محدودیت را می‌توان پس از فهرست پارامترهای قالب یا پس از فهرست پارامترهای تابع قرار داد.

```cpp
#include <concepts>

auto integral_add = []<typename T>(T left, T right)
    requires std::integral<T>
{
    return left + right;
};

auto ordered = [](const auto& left, const auto& right)
    requires requires { left < right; }
{
    return left < right;
};
```

در نمونه‌ی اول، `requires` نوع `T` را محدود می‌کند. در نمونه‌ی دوم، از یک `requires-expression` برای بررسی معتبر بودن عبارت `left < right` استفاده شده است. این الگو می‌تواند برای generic codeهایی مفید باشد که لازم است فقط با انواع دارای یک عملیات مشخص قابل فراخوانی باشند.

### تفاوت requires مربوط به template و requires مربوط به تابع

در یک `lambda` دارای template parameter list صریح، محدودیت پیش از فهرست پارامترهای تابع، پارامترهای قالب را محدود می‌کند. محدودیت پس از فهرست پارامترهای تابع، callable بودن specialization را با توجه به signature و نوع‌های استنتاج‌شده محدود می‌سازد.

```cpp
#include <concepts>
#include <type_traits>

auto integral_identity = []<std::integral T>(T value) {
    return value;
};

auto numeric_identity = [](auto value)
    requires std::is_arithmetic_v<decltype(value)>
{
    return value;
};
```

نمونه‌ی دوم به `<type_traits>` نیاز دارد. تفاوت عملی این است که در شکل اول constraint مستقیماً به پارامتر قالب نام‌گذاری‌شده متصل است؛ در شکل دوم constraint از نوع پارامتر استنتاج‌شده استفاده می‌کند.

استفاده از `requires` برای بیان محدودیت‌های معنایی واقعی مناسب است. افزودن constraintهای پیچیده و غیرضروری می‌تواند پیام‌های خطا و خوانایی را بدتر کند؛ همچنین `requires` تضمین نمی‌کند که تمام خطاهای داخل بدنه‌ی `lambda` به خطاهای قابل‌مدیریت در overload resolution تبدیل شوند. عباراتی که فقط در بدنه استفاده می‌شوند همچنان باید برای specialization انتخاب‌شده معتبر باشند.

## استفاده از Lambda در contextهای ارزیابی‌نشده

پیش از ++C20، قرار دادن یک lambda-expression در بعضی contextهای **unevaluated** مانند `decltype` مجاز نبود. از ++C20 می‌توان از `lambda` در چنین موقعیت‌هایی استفاده کرد؛ این قابلیت در تعریف type aliasها و بعضی الگوهای metaprogramming سودمند است.

```cpp
#include <type_traits>

using Transform = decltype([](int value) {
    return value * 2;
});

static_assert(std::is_empty_v<Transform>);
static_assert(std::is_default_constructible_v<Transform>);

int main() {
    Transform transform{};
    return transform(21) == 42 ? 0 : 1;
}
```

در این نمونه، عبارت `lambda` داخل `decltype` ارزیابی نمی‌شود؛ هدف، استخراج نوع closure است. سپس، در ++C20 می‌توان شیئی از این closure بدون capture را با مقداردهی پیش‌فرض ساخت.

هر عبارت `lambda` نوع closure مجزای خودش را دارد، حتی اگر متن دو عبارت از نظر ظاهری یکسان باشد. بنابراین دو عبارت مستقل `[](int x) { return x; }` لزوماً نوع یکسانی تولید نمی‌کنند. اگر لازم است یک نوع مشخص در چند محل استفاده شود، آن را یک‌بار در یک alias یا متغیر تعریف کنید و همان نوع را به کار ببرید.

## تغییرات مربوط به Closureهای بدون Capture

یکی دیگر از بهبودهای ++C20 این است که closure type مربوط به `lambda` بدون capture می‌تواند default-constructible و copy-assignable باشد. در استانداردهای قبلی، این قابلیت‌ها به همان شکل در دسترس نبودند.

```cpp
int main() {
    auto is_positive = [](int value) {
        return value > 0;
    };

    auto another = decltype(is_positive){};
    another = is_positive;
    return another(1) ? 0 : 1;
}
```

این قابلیت در APIهایی مفید است که callable را به‌عنوان یک policy، predicate یا پارامتر template نگه‌داری می‌کنند و ساختن یا جایگزین کردن آن باید ساده باشد.

### کاربرد در Templateهای استاندارد

برای نمونه، یک نوع closure بدون capture می‌تواند به‌عنوان predicate قابل ساخت و انتساب در سازوکارهایی به کار رود که نوع callable را نگه‌داری می‌کنند. این قابلیت به‌ویژه همراه با استفاده از lambdaها به‌عنوان template argument کاربرد دارد.

با این حال، نباید تصور کرد هر `lambda` قابل‌انتساب است. اگر lambda دارای capture باشد، closure معمولاً state ذخیره‌شده دارد و عملگر assignment آن به همان شکل closure بدون capture در دسترس نیست. این تفاوت در طراحی containerها و wrapperهای سفارشی مهم است.

## استفاده از Lambda بدون Capture به‌عنوان Non-type Template Parameter

از ++C20، closure type یک `lambda` بدون capture یک **structural type** است. این ویژگی در کنار پشتیبانی ++C20 از class-typeها به‌عنوان non-type template parameter (NTTP) اجازه می‌دهد خود یک lambda-expression بدون capture به template argument تبدیل شود.

```cpp
template<auto Operation>
struct ConstantOperation {
    template<typename T>
    static constexpr auto apply(T value) {
        return Operation(value);
    }
};

using SquareOperation = ConstantOperation<[](int value) {
    return value * value;
}>;

static_assert(SquareOperation::apply(6) == 36);
```

در این مثال، `Operation` یک non-type template parameter است که مقدار آن callable بدون capture است. بدین ترتیب، رفتار می‌تواند در زمان کامپایل بخشی از هویت یک specialization باشد و نیازی به اشاره‌گر تابع یا type-erasure مانند `std::function` وجود ندارد.

### محدودیت‌های Structural Type

شرط کلیدی، بدون capture بودن `lambda` است. اگر closure متغیری را capture کند، نوع آن structural type نیست و نمی‌توان آن را به همین شیوه به‌عنوان NTTP استفاده کرد. همچنین template argument باید سایر الزامات constant template argument را نیز برآورده کند؛ صرفاً بدون capture بودن، تمام کاربردهای احتمالی را معتبر نمی‌کند.

این روش برای policyهای کوچک و ثابت، محاسبات compile-time و الگوریتم‌های generic با رفتار مشخص مفید است. در مقابل، اگر رفتار در زمان اجرا تغییر می‌کند یا داده‌ی runtime باید نگه‌داری شود، یک شیء callable معمولی یا پارامتر تابع ساده‌تر و مناسب‌تر خواهد بود.

## گسترش Init-capture برای Parameter Pack

قابلیت `init-capture` از ++C14 به بعد اجازه می‌دهد متغیری در closure با نامی تازه ساخته شود؛ برای نمونه، می‌توان آرگومان را با `std::move` یا `std::forward` در capture ذخیره کرد. در ++C20، syntax مربوط به init-capture با `pack expansion` گسترش پیدا کرد تا بتوان مجموعه‌ای از آرگومان‌ها را یک‌جا capture کرد.

```cpp
#include <tuple>
#include <type_traits>
#include <utility>

template<typename... Args>
auto make_tuple_factory(Args&&... args) {
    return [...values = std::forward<Args>(args)]() mutable {
        return std::tuple<std::decay_t<Args>...>{std::move(values)...};
    };
}

int main() {
    auto make_values = make_tuple_factory(10, 2.5, 30L);
    auto values = make_values();
    return std::get<0>(values) == 10 ? 0 : 1;
}
```

در این مثال، هر عضو از `args...` با `std::forward` مناسب خود به capture متناظر تبدیل می‌شود. سپس lambda یک tuple را با انتقال مقدارهای ذخیره‌شده می‌سازد. این روش برای ساخت taskها، callbackها یا factoryهایی مفید است که باید مجموعه‌ای از آرگومان‌ها را برای استفاده‌ی بعدی نگه دارند.

### ملاحظات مهم در Capture کردن Pack

چون closure در این مثال آرگومان‌ها را نگه می‌دارد، طول عمر مقادیر captureشده مستقل از طول عمر پارامترهای تابع factory خواهد بود؛ این موضوع یکی از مزیت‌های مهم آن است. با این وجود، کپی یا انتقال هر آرگومان به semantics نوع همان آرگومان وابسته است و نوع‌های move-only باید به‌درستی مدیریت شوند.

همچنین، این factory عمداً `mutable` است و اعضای ذخیره‌شده را هنگام فراخوانی به `std::move` می‌دهد. پس فراخوانی‌های بعدی روی همان closure ممکن است مقادیر منتقل‌شده یا حالت‌های moved-from را ببیند. اگر callable باید چندبار و بدون مصرف state اجرا شود، باید سیاست انتقال و کپی را متناسب با نیاز طراحی کرد.

## مدیریت Capture صریح this و هشدار درباره Capture ضمنی

در ++C، capture پیش‌فرض `[=]` به معنی کپی کردن کل شیء جاری نیست. اگر بدنه‌ی lambda به اعضای غیر static کلاس دسترسی داشته باشد، معمولاً `this` به‌صورت ضمنی capture می‌شود؛ در نتیجه closure اشاره‌گر `this` را نگه می‌دارد و همچنان به همان شیء اصلی دسترسی دارد.

از ++C20 می‌توان capture را به شکل `[=, this]` نوشت تا این قصد صریح باشد. در همین استاندارد، capture ضمنی `this` از طریق `[=]` deprecate شده است، چون ممکن است با کپی شدن خود شیء اشتباه گرفته شود.

```cpp
#include <functional>

class Counter {
public:
    explicit Counter(int value) : value_(value) {}

    auto make_incrementer() {
        return [=, this](int amount) {
            value_ += amount;
            return value_;
        };
    }

private:
    int value_;
};

int main() {
    Counter counter{10};
    auto increment = counter.make_incrementer();
    return increment(5) == 15 ? 0 : 1;
}
```

در این نمونه، `[=, this]` اشاره‌گر `this` را capture می‌کند و تغییر `value_` روی خود `Counter` انجام می‌شود؛ مقدار اعضا به closure کپی نشده است.

### تفاوت Capture کردن this با Capture کردن *this

قابلیت `[ *this ]` از ++C17 معرفی شده است و با `[=, this]` تفاوت بنیادی دارد. در حالت `[=, this]`، اشاره‌گر `this` در closure ذخیره می‌شود؛ در حالت `[*this]`، خود شیء جاری با copy کردن در closure نگه‌داری می‌شود.

کپی کردن خود شیء می‌تواند برای snapshot کردن state مفید باشد، اما به copy-constructible بودن شیء و semantics مورد انتظار وابسته است. اگر شیء یک منبع خارجی را مدیریت می‌کند، کپی آن الزاماً به معنای استقلال کامل منبع نیست. در هر دو حالت باید lifetime را در نظر گرفت: capture کردن `this` به‌صورت اشاره‌گر، اگر closure پس از نابودی شیء استفاده شود، می‌تواند به dangling pointer و رفتار تعریف‌نشده منجر شود.

### انتخاب مناسب برای Capture

برای callback کوتاه و هم‌زمانی که در طول عمر شیء اجرا می‌شود، capture کردن `this` ممکن است انتخاب طبیعی باشد. برای callbackهایی که ذخیره می‌شوند، به thread دیگر فرستاده می‌شوند یا اجرای آن‌ها ممکن است پس از نابودی شیء ادامه یابد، بهتر است ownership و lifetime به‌صورت صریح طراحی شود؛ برای مثال با کپی کردن state مستقل یا capture کردن `std::shared_ptr` در صورت مناسب بودن مدل مالکیت.

استفاده از `[=, this]` به‌تنهایی مشکل lifetime را حل نمی‌کند؛ این syntax فقط قصد capture را روشن‌تر می‌کند.

## ارزیابی constexpr و Lambdaهای consteval

از ++C17، یک lambda در صورت برآورده کردن شرایط مربوط می‌تواند به‌صورت ضمنی `constexpr` باشد؛ لازم نیست حتماً کلمه‌ی کلیدی `constexpr` نوشته شود. ++C20 این مدل را با امکان نوشتن `consteval` برای عملگر فراخوانی lambda گسترش می‌دهد.

```cpp
auto square_at_compile_time = [](int value) consteval {
    return value * value;
};

constexpr int result = square_at_compile_time(9);
static_assert(result == 81);
```

در این مثال، فراخوانی باید یک immediate invocation معتبر باشد؛ یعنی نتیجه در زمان کامپایل محاسبه شود. بر خلاف یک `constexpr` lambda معمولی که در صورت مجاز بودن می‌تواند در زمان اجرا نیز فراخوانی شود، تابع `consteval` برای فراخوانی‌های immediate طراحی شده است.

### تفاوت constexpr و consteval

کلمه‌ی کلیدی `constexpr` امکان ارزیابی در زمان کامپایل را فراهم می‌کند، اما به‌خودی‌خود تضمین نمی‌کند تمام فراخوانی‌ها در زمان کامپایل رخ دهند. در مقابل، `consteval` عملگر فراخوانی را immediate function می‌کند و فراخوانی غیرمجاز در زمان اجرا را نمی‌پذیرد.

برای تابع‌های محاسباتی ساده، `constexpr` معمولاً انعطاف بیشتری فراهم می‌کند. `consteval` زمانی مناسب است که محاسبه‌ی زمان کامپایل بخشی از قرارداد API باشد؛ برای مثال، ساخت مقدارهای ثابت یا اعتبارسنجی ورودی‌های ثابت.

### محدودیت‌های ارزیابی در زمان کامپایل

نوشتن `constexpr` یا `consteval` محدودیت‌های زبان را حذف نمی‌کند. بدنه و عملیات مورد استفاده باید برای نوع ارزیابی موردنظر مجاز باشند، و منابع یا مقدارهای runtime نمی‌توانند دلخواهانه به محاسبه‌ی زمان کامپایل وارد شوند. در نتیجه، این specifierها را باید بر اساس قرارداد واقعی تابع انتخاب کرد، نه فقط با هدف افزایش سرعت.

## مقایسه‌ی مهم‌ترین بهبودهای ++C20

| قابلیت | نسخه‌ی معرفی | مسئله‌ای که حل می‌کند | نکته‌ی طراحی |
|---|---|---|---|
| قابلیت فهرست صریح پارامترهای قالب | ++C20 | کنترل نوع‌ها، نام‌گذاری پارامترهای قالب و بیان رابطه‌ی نوعی | هنگامی به کار رود که `auto` به‌اندازه‌ی کافی گویا یا دقیق نیست |
| قید `requires` در lambda | ++C20 | محدود کردن دامنه‌ی انواع قابل‌قبول و بهبود generic interface | constraintها را بر اساس نیاز واقعی API تعریف کنید |
| استفاده از lambda در unevaluated context | ++C20 | استخراج closure type در `decltype` و الگوهای مرتبط | هر عبارت lambda همچنان نوع یکتای خود را دارد |
| سازنده و انتساب closure بدون capture | ++C20 | ساخت و جایگزینی ساده‌تر callableهای بدون state | برای closureهای دارای capture این نتیجه را تعمیم ندهید |
| استفاده از closure بدون capture به‌عنوان NTTP | ++C20 | وارد کردن رفتار ثابت در هویت specialization قالب | closure باید بدون capture و argument باید معتبر در زمان کامپایل باشد |
| گسترش pack در init-capture | ++C20 | ذخیره‌ی مجموعه‌ای از آرگومان‌های variadic در closure | مالکیت، move semantics و مصرف‌شدن state را بررسی کنید |
| capture صریح با `[=, this]` | ++C20 | روشن کردن تفاوت capture متغیرها و اشاره‌گر `this` | همچنان باید lifetime شیء هدف تضمین شود |
| استفاده از `consteval` در lambda | ++C20 | اجبار محاسبه‌ی فراخوانی در زمان کامپایل | برای APIهایی مناسب است که ارزیابی compile-time الزام واقعی است |

## الگوهای کاربردی در طراحی نرم‌افزار

### تعریف Predicateهای Generic

هنگامی که predicate باید تنها برای مجموعه‌ی خاصی از نوع‌ها معتبر باشد، ترکیب explicit template parameter list و `requires` قرارداد را در کنار پیاده‌سازی نگه می‌دارد. این الگو به‌ویژه در الگوریتم‌ها و utilityهای عمومی از خطاهای ناشی از instantiate شدن بدنه برای نوع‌های نامناسب می‌کاهد.

```cpp
#include <concepts>
#include <algorithm>
#include <vector>

auto is_positive = []<std::integral T>(T value) {
    return value > 0;
};

int main() {
    std::vector<int> values{-2, 0, 4, 7};
    const auto count = std::count_if(values.begin(), values.end(), is_positive);
    return count == 2 ? 0 : 1;
}
```

### ساخت Policyهای زمان کامپایل

در بعضی APIها، پارامتر قالب تعیین می‌کند یک الگوریتم چگونه عمل کند. استفاده از lambda بدون capture به‌عنوان NTTP می‌تواند جایگزین مناسبی برای ساختن انواع policy جداگانه باشد، به‌خصوص وقتی رفتار کوتاه و محلی است. این روش از type erasure جلوگیری می‌کند، اما هر رفتار متمایز معمولاً specialization متمایزی تولید می‌کند؛ بنابراین تعداد زیاد رفتارهای منحصربه‌فرد ممکن است حجم کد کامپایل‌شده را افزایش دهد.

### ساخت Callbackهای مالک آرگومان

init-capture به‌همراه pack expansion برای ساخت callbackهایی مفید است که باید پارامترها را برای اجرا در آینده نگه دارند. برای callbackهای asynchronous، صرفاً درست بودن capture کافی نیست؛ باید مشخص شود آرگومان‌ها کپی می‌شوند یا منتقل، منابع به چه کسی تعلق دارند و آیا callback ممکن است بیش از یک‌بار اجرا شود.

## خطاهای رایج و نکات طراحی

- **اشتباه گرفتن `[=]` با کپی کامل شیء:** این capture به‌تنهایی اعضای شیء جاری را در closure کپی نمی‌کند. برای کپی خود شیء باید از `[*this]` با درک پیامدهای کپی استفاده شود.
- **نادیده گرفتن lifetime:** capture کردن اشاره‌گر `this` یا reference به متغیر محلی، طول عمر مقصد را افزایش نمی‌دهد. نگه‌داری closure پس از پایان عمر مقصد می‌تواند به رفتار تعریف‌نشده منجر شود.
- **استفاده‌ی بی‌دلیل از template parameter صریح:** اگر هر پارامتر مستقل باشد و نام‌گذاری `T` سودی نداشته باشد، `auto` ممکن است خواناتر باشد.
- **فرض اشتباه درباره‌ی نوع closure:** دو عبارت lambda ظاهراً یکسان، نوع یکسانی ندارند. برای تکیه بر یک closure type مشخص، همان عبارت یا alias نوع را بازاستفاده کنید.
- **فرض قابل‌انتساب بودن همه‌ی closureها:** ویژگی default construction و copy assignment در ++C20 به closureهای بدون capture مربوط است، نه تمام lambdaها.
- **استفاده از `consteval` بدون نیاز قراردادی:** این specifier آزادی فراخوانی در runtime را از بین می‌برد. اگر هر دو حالت لازم‌اند، `constexpr` انتخاب انعطاف‌پذیرتری است.
- **فراموش کردن اثر move در init-capture:** وقتی اعضای captureشده با `std::move` مصرف می‌شوند، اجرای دوباره‌ی callback ممکن است مقدارهای moved-from را ببیند. سیاست اجرای یک‌باره یا چندباره را مشخص کنید.
- **طراحی constraintهای بیش‌ازحد پیچیده:** `requires` باید قرارداد قابل‌فهمی ارائه کند. محدودیت‌های غیرضروری گاهی overload resolution و گزارش خطا را دشوارتر می‌کنند.
- **فرض پشتیبانی کامل کامپایلر:** پشتیبانی از ++C20 به نسخه‌ی کامپایلر، library و میزان پیاده‌سازی featureها بستگی دارد. فعال کردن `-std=c++20` به‌تنهایی تضمین نمی‌کند همه‌ی جزئیات در هر نسخه‌ی قدیمی به‌درستی پیاده‌سازی شده باشند.

## بررسی پشتیبانی کامپایلر و Feature-test Macroها

برای تشخیص بعضی قابلیت‌ها در زمان کامپایل می‌توان از feature-test macroها استفاده کرد. مقدار `__cpp_generic_lambdas` در پیاده‌سازی دارای پشتیبانی explicit template parameter list باید دست‌کم `201707L` باشد. برای pack expansion در init-capture، مقدار `__cpp_init_captures` باید دست‌کم `201803L` باشد.

```cpp
#if defined(__cpp_generic_lambdas) && __cpp_generic_lambdas >= 201707L
#define HAS_EXPLICIT_LAMBDA_TEMPLATE_LIST 1
#else
#define HAS_EXPLICIT_LAMBDA_TEMPLATE_LIST 0
#endif

#if defined(__cpp_init_captures) && __cpp_init_captures >= 201803L
#define HAS_LAMBDA_INIT_CAPTURE_PACK 1
#else
#define HAS_LAMBDA_INIT_CAPTURE_PACK 0
#endif
```

این macroها برای ساختن کد قابل‌حمل در پروژه‌هایی که چند کامپایلر یا چند نسخه‌ی استاندارد را هدف می‌گیرند مفیدند. در پروژه‌ای که حداقل استاندارد آن ++C20 است، معمولاً بهتر است به‌جای پراکنده کردن شرط‌های پیش‌پردازنده، حداقل نسخه‌ی کامپایلر موردنیاز را مستند و در CI آزمایش کنید.

## مرز قابلیت‌های ++C20 و نسخه‌های بعدی

برخی قابلیت‌هایی که در بحث lambda مطرح می‌شوند، در واقع متعلق به استانداردهای بعدی هستند و نباید با بهبودهای ++C20 اشتباه گرفته شوند. برای مثال، `static` lambda call operator و explicit object parameter برای lambda از ++C23 هستند. همچنین capture کردن `*this` با `[ *this ]` از ++C17 معرفی شده است، هرچند در کنار قواعد capture در ++C20 همچنان اهمیت دارد.

شناخت این مرزها هنگام تنظیم `-std=c++20`، بازبینی کد و بررسی گزارش‌های کامپایلر ضروری است؛ زیرا یک قابلیت معتبر در ++C23 لزوماً در حالت ++C20 قابل استفاده نیست.

## جمع‌بندی

بهبودهای lambda در ++C20 این ابزار را از یک روش کوتاه برای نوشتن callback، به گزینه‌ای قدرتمندتر برای generic programming و metaprogramming تبدیل می‌کنند. فهرست صریح پارامترهای قالب و `requires` امکان تعریف interfaceهای دقیق‌تر را فراهم می‌کنند؛ closureهای بدون capture می‌توانند در contextهای نوعی و به‌عنوان NTTP کاربرد داشته باشند؛ init-capture اکنون packهای variadic را پوشش می‌دهد؛ و قواعد مربوط به `this` قصد برنامه‌نویس را روشن‌تر می‌کنند.

در استفاده‌ی حرفه‌ای، انتخاب syntax باید تابع قرارداد API، نیاز به کنترل نوع، lifetime داده‌ها، semantics کپی و انتقال، و هزینه‌ی احتمالی تولید specializationهای متعدد باشد. این قابلیت‌ها زمانی بیشترین ارزش را دارند که کد را دقیق‌تر و ایمن‌تر کنند، نه صرفاً پیچیده‌تر.

## منابع پیشنهادی برای مطالعه‌ی بیشتر

- [مرجع Lambda expressions در cppreference](https://en.cppreference.com/w/cpp/language/lambda)
- [مرجع تغییرات استاندارد از ++C17 به ++C20](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p2131r0.html)
- [پیشنهاد P0428R2 درباره‌ی فهرست قالب در Generic Lambda](https://wg21.link/P0428R2)
- [پیشنهاد P0624R2 درباره‌ی سازنده و انتساب Closureهای بدون Capture](https://wg21.link/P0624R2)
- [پیشنهاد P0315R4 درباره‌ی Lambda در Unevaluated Contextها](https://wg21.link/P0315R4)
- [پیشنهاد P0780R2 درباره‌ی Pack Expansion در Init-capture](https://wg21.link/P0780R2)
- [پیشنهاد P0806R2 درباره‌ی Deprecate شدن Capture ضمنی this با `[=]`](https://wg21.link/P0806R2)

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>