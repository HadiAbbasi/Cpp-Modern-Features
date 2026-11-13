<div align="center">

[🇺🇸 English](../../en/cpp20/structured_bindings_improvements.md) | [🇮🇷 فارسی](./structured_bindings_improvements.md)

</div>

---

# بهبودهای Structured Bindings در ++C20

## مقدمه

قابلیت Structured Bindings یکی از ویژگی‌های مهم زبان ++C17 است که امکان استخراج چند جزء از یک شیء و اتصال آن‌ها به نام‌های مستقل را فراهم می‌کند. این قابلیت باعث می‌شود کار با `std::pair`، `std::tuple`، آرایه‌ها و ساختارهای داده‌ای ساده خواناتر و طبیعی‌تر شود.

با وجود کاربرد گسترده این قابلیت، نسخه اولیه آن محدودیت‌هایی داشت. برای مثال، در ++C17 امکان capture مستقیم نام‌های حاصل از Structured Binding در `lambda` وجود نداشت. همچنین استفاده از برخی مشخصه‌های مربوط به طول عمر و ذخیره‌سازی متغیرها امکان‌پذیر نبود.

استاندارد ++C20 با توسعه این قابلیت، Structured Bindings را به declarationهای معمولی متغیر نزدیک‌تر کرد.

مهم‌ترین بهبودهای این نسخه عبارت‌اند از:

- امکان capture مستقیم Structured Bindings در Lambda Expressions.
- پشتیبانی از capture مقداری و ارجاعی.
- امکان استفاده از `static`.
- امکان استفاده از `thread_local`.
- بهبودهای مرتبط با attributeها و قواعد declaration.
- اصلاح قواعد دسترسی به اعضای کلاس از طریق پیشنهادهای استانداردسازی مرتبط.

برای درک صحیح این تغییرات، ابتدا باید سازوکار داخلی Structured Bindings و محدودیت‌های نسخه اولیه آن را بررسی کنیم.

## آشنایی با مبانی Structured Bindings

### مفهوم Structured Bindings

در ++C17 می‌توان با استفاده از Structured Bindings اجزای یک شیء را به نام‌های جداگانه متصل کرد.

برای مثال، تابع زیر یک `std::pair` برمی‌گرداند:

```
#include <iostream>
#include <string>
#include <utility>

std::pair<int, std::string> getUser() {
    return {42, "Alice"};
}

int main() {
    auto [id, name] = getUser();

    std::cout << id << '\n';
    std::cout << name << '\n';
}
```

در این مثال، نام‌های `id` و `name` به دو جزء شیء برگشتی متصل می‌شوند.

بدون Structured Bindings باید به اعضای `first` و `second` دسترسی پیدا می‌کردیم:

```
auto user = getUser();

auto id = user.first;
auto name = user.second;
```

استفاده از Structured Bindings باعث می‌شود کد کوتاه‌تر و خواناتر باشد؛ به‌خصوص زمانی که خروجی یک تابع شامل چند مقدار مرتبط است.

### سازوکار داخلی و متغیر پشتیبان

Structured Bindings برخلاف ظاهر ساده خود، صرفاً معادل تعریف چند متغیر مستقل نیستند.

استاندارد برای هر Structured Binding یک متغیر پشتیبان (Backing Variable) در نظر می‌گیرد. این متغیر شیء اصلی را نگه می‌دارد و نام‌های معرفی‌شده به اجزای آن متصل می‌شوند.

برای مثال:

```
auto [x, y] = std::pair{10, 20};
```

از نظر مفهومی می‌توان رفتار آن را به شکل زیر تصور کرد:

```
auto backing = std::pair{10, 20};

auto& x = backing.first;
auto& y = backing.second;
```

این کد صرفاً یک نمایش مفهومی است و معادل دقیق قواعد استاندارد نیست. نوع واقعی bindingها و نحوه اتصال آن‌ها به اجزا به نوع داده و قواعد مربوط به Structured Bindings وابسته است.

متغیر پشتیبان نیز یک جزئیات پیاده‌سازی قابل‌مشاهده در کد کاربر نیست؛ بلکه بخشی از مدل معنایی تعریف‌شده در استاندارد زبان است.

### انواع قابل استفاده در Structured Bindings

زبان ++C سه دسته اصلی از انواع را برای Structured Bindings پشتیبانی می‌کند.

#### آرایه‌ها

آرایه‌هایی با تعداد عناصر مشخص را می‌توان مستقیماً تجزیه کرد:

```
int values[] = {10, 20, 30};

auto [first, second, third] = values;
```

در این حالت، تعداد نام‌ها باید دقیقاً با تعداد عناصر آرایه برابر باشد.

#### انواع Tuple-like

انواعی مانند موارد زیر از این قابلیت پشتیبانی می‌کنند:

- `std::pair`
- `std::tuple`
- `std::array`

این انواع از سازوکار tuple-like شامل `std::tuple_size` و `std::tuple_element` استفاده می‌کنند.

```
#include <array>

int main() {
    std::array<int, 3> values{10, 20, 30};

    auto [first, second, third] = values;
}
```

#### انواع Class

کلاس‌هایی که اعضای داده‌ای غیر `static` آن‌ها با قواعد Structured Bindings سازگار باشند نیز قابل تجزیه هستند:

```
struct Point {
    int x;
    int y;
};

int main() {
    Point point{10, 20};

    auto [x, y] = point;
}
```

در این حالت، اعضای داده‌ای باید شرایط استاندارد مربوط به تجزیه کلاس را رعایت کنند؛ از جمله محدودیت‌های مربوط به ساختار کلاس، ارث‌بری و دسترسی.

### تفاوت auto، auto& و auto&&

انتخاب نوع declaration در Structured Bindings اهمیت زیادی دارد.

```
#include <string>

struct User {
    int age;
    std::string name;
};

int main() {
    User user{25, "Alice"};

    auto [age1, name1] = user;
    auto& [age2, name2] = user;
    auto&& [age3, name3] = user;
}
```

رفتار این declarationها به شکل زیر است:

| Declaration     | رفتار                                                     |
| --------------- | --------------------------------------------------------- |
| `auto [x, y]`   | ایجاد متغیر پشتیبان مستقل با مقداردهی از initializer      |
| `auto& [x, y]`  | اتصال متغیر پشتیبان به lvalue اصلی                        |
| `auto&& [x, y]` | اتصال متغیر پشتیبان با توجه به value category initializer |

در مثال بالا، تغییر `age1` روی شیء اصلی اثر ندارد؛ زیرا متغیر پشتیبان مستقلی ایجاد شده است.

در مقابل، تغییر `age2` یا `age3` مستقیماً عضو `user` را تغییر می‌دهد.

نکته مهم این است که `auto [x, y]` الزاماً به معنی ایجاد دو کپی مستقل از اعضا نیست. تنها یک متغیر پشتیبان وجود دارد و bindingها به اجزای آن متصل می‌شوند.

### تفاوت نوع Binding و نوع Expression

یکی از جزئیات مهم Structured Bindings، رفتار `decltype` است.

نوعی که `decltype` برای یک binding گزارش می‌کند، طبق قواعد ویژه Structured Bindings تعیین می‌شود و نباید آن را با نوع ظاهری reference داخلی اشتباه گرفت.

```
#include <tuple>
#include <type_traits>

int main() {
    int value = 10;

    std::tuple<int, int&> data{20, value};

    auto [first, second] = data;

    static_assert(std::is_same_v<decltype(first), int>);
    static_assert(std::is_same_v<decltype(second), int&>);

    second = 50;
}
```

در این مثال، `second` همچنان به متغیر اصلی `value` متصل است؛ زیرا عنصر دوم tuple از نوع `int&` است.

این موضوع نشان می‌دهد که رفتار Structured Bindings تنها به نوع ظاهری declaration محدود نمی‌شود.

## بهبودهای اصلی در ++C20

### امکان Capture مستقیم در Lambda

یکی از مهم‌ترین محدودیت‌های ++C17، عدم امکان capture مستقیم نام‌های Structured Binding در Lambda Expressions بود.

برای مثال، کد زیر در ++C17 معتبر نبود:

```
auto [x, y] = std::pair{10, 20};

auto sum = [x, y] {
    return x + y;
};
```

در ++C20 این محدودیت برطرف شد.

اکنون می‌توان نام‌های Structured Binding را مستقیماً در capture-list قرار داد:

```
#include <iostream>
#include <utility>

int main() {
    auto [x, y] = std::pair{10, 20};

    auto sum = [x, y] {
        return x + y;
    };

    std::cout << sum() << '\n';
}
```

این تغییر باعث می‌شود دیگر برای capture کردن اجزای یک شیء نیازی به تعریف متغیرهای موقت یا استفاده از init-capture نباشد.

### Capture به‌صورت مقداری

در capture مقداری، مقدار binding در closure object ذخیره می‌شود:

```
#include <iostream>
#include <utility>

int main() {
    auto [x, y] = std::pair{10, 20};

    auto callback = [x, y] {
        return x + y;
    };

    x = 100;
    y = 200;

    std::cout << callback() << '\n';
}
```

خروجی:

```
30
```

در این مثال، lambda مقادیر اولیه `x` و `y` را نگه می‌دارد.

تغییر متغیرهای اصلی پس از ایجاد lambda، روی مقادیر ذخیره‌شده در closure object اثری ندارد.

این رفتار برای callbackهایی مناسب است که باید مستقل از تغییرات بعدی state محلی باشند.

### Capture به‌صورت ارجاعی

در capture ارجاعی، lambda به bindingهای موجود متصل می‌شود:

```
#include <iostream>
#include <utility>

int main() {
    auto [x, y] = std::pair{10, 20};

    auto callback = [&x, &y] {
        x += 5;
        y += 5;
    };

    callback();

    std::cout << x << ' ' << y << '\n';
}
```

خروجی:

```
15 25
```

در این حالت، تغییرهای داخل lambda مستقیماً روی اجزای شیء پشتیبان اعمال می‌شوند.

این روش زمانی مناسب است که lambda باید state موجود را تغییر دهد و عمر آن از عمر اشیای مرجع بیشتر نشود.

### Capture ضمنی با Default Capture

همانند متغیرهای معمولی، Structured Bindings نیز می‌توانند با استفاده از capture-default به‌صورت ضمنی capture شوند.

```
#include <iostream>
#include <utility>

int main() {
    auto [width, height] = std::pair{640, 480};

    auto calculateArea = [=] {
        return width * height;
    };

    auto scale = [&] {
        width *= 2;
        height *= 2;
    };

    scale();

    std::cout << calculateArea() << '\n';
}
```

در lambda اول، `[=]` باعث capture مقداری bindingهای استفاده‌شده می‌شود.

در lambda دوم، `[&]` باعث capture ارجاعی آن‌ها می‌شود.

نکته مهم این است که capture ضمنی تنها برای bindingهایی انجام می‌شود که در بدنه lambda مورد استفاده قرار گرفته‌اند.

### تفاوت Capture مستقیم با Init-capture

قابلیت init-capture از ++C14 در دسترس بوده است.

بنابراین پیش از ++C20 نیز می‌توانستیم مقادیر استخراج‌شده را به lambda منتقل کنیم:

```
#include <utility>

int main() {
    auto [x, y] = std::pair{10, 20};

    auto callback = [sum = x + y] {
        return sum;
    };
}
```

اما این روش با capture مستقیم متفاوت است.

در ++C20 می‌توان نوشت:

```
auto callback = [x, y] {
    return x + y;
};
```

مزیت capture مستقیم، حذف نام‌های اضافی و ساده‌تر شدن کد است.

با این حال، init-capture همچنان در موارد زیر کاربرد دارد:

- تغییر نام متغیرها هنگام capture.
- محاسبه مقدار جدید هنگام ایجاد lambda.
- انتقال مالکیت منابع.
- استفاده از `std::move`.
- ایجاد state مستقل با نوع یا مقدار متفاوت.

برای مثال:

```
#include <memory>

int main() {
    auto resource = std::make_unique<int>(42);

    auto callback = [ptr = std::move(resource)] {
        return *ptr;
    };
}
```

در این مثال، init-capture مالکیت منبع را به closure object منتقل می‌کند.

### محدودیت‌های Capture ارجاعی

Capture ارجاعی همیشه انتخاب مناسبی نیست.

اگر lambda پس از پایان عمر شیء اصلی استفاده شود، ممکن است dangling reference ایجاد شود.

```
#include <functional>
#include <utility>

int main() {
    std::function<int()> callback;

    {
        auto [x, y] = std::pair{10, 20};

        callback = [&x, &y] {
            return x + y;
        };
    }

    // Calling callback here causes undefined behavior.
}
```

پس از خروج از scope داخلی، متغیر پشتیبان از بین می‌رود و referenceهای موجود در closure دیگر معتبر نیستند.

بنابراین اگر lambda قرار است ذخیره شود یا از scope فعلی فراتر برود، capture مقداری یا init-capture با انتقال مالکیت معمولاً انتخاب امن‌تری است.

### محدودیت Capture برای Bit-field

یکی از edge caseهای مهم، اعضای bit-field است.

Structured Bindings می‌توانند اعضای bit-field را تجزیه کنند؛ اما bit-field را نمی‌توان با reference capture در lambda استفاده کرد.

```
struct Flags {
    unsigned int enabled : 1;
    unsigned int visible : 1;
};

int main() {
    Flags flags{1, 0};

    auto [enabled, visible] = flags;

    auto callback = [enabled] {
        return enabled;
    };
}
```

در این مثال، capture مقداری مجاز است.

اما capture ارجاعی مستقیم به binding مربوط به bit-field مجاز نیست.

در چنین شرایطی، اگر نیاز به استفاده از مقدار در callback وجود داشته باشد، می‌توان از capture مقداری یا یک متغیر مستقل استفاده کرد.

## مدیریت طول عمر با مشخصه‌های ذخیره‌سازی

### استفاده از static

یکی از بهبودهای مهم ++C20، امکان استفاده از `static` در Structured Binding است.

```
#include <string>
#include <utility>

void updateUser() {
    static auto [id, name] =
        std::pair{42, std::string{"Alice"}};

    ++id;
    name += " Smith";
}
```

در این مثال، متغیر پشتیبان دارای static storage duration است.

در نتیجه:

- مقداردهی اولیه فقط یک بار انجام می‌شود.
- مقدارها بین فراخوانی‌های تابع حفظ می‌شوند.
- bindingها در طول عمر متغیر پشتیبان معتبر باقی می‌مانند.
- تمام فراخوانی‌های تابع به همان state دسترسی دارند.

این قابلیت برای نگه‌داری stateهای کوچک و چندجزئی مناسب است.

نکته مهم این است که `static` به declaration و backing object مربوط می‌شود؛ نه اینکه هر binding به‌عنوان یک متغیر مستقل با storage duration جداگانه تعریف شود.

### استفاده از thread_local

استاندارد ++C20 استفاده از `thread_local` را نیز مجاز کرده است.

```
#include <utility>

void processRequest() {
    thread_local auto [requestCount, errorCount] =
        std::pair{0, 0};

    ++requestCount;

    if (requestCount % 10 == 0) {
        ++errorCount;
    }
}
```

در این مثال، هر thread نسخه مستقل خود از متغیر پشتیبان را دارد.

این ویژگی برای موارد زیر مفید است:

- شمارنده‌های اختصاصی هر thread.
- cacheهای محلی thread.
- آمار پردازش درخواست‌ها.
- نگه‌داری state مستقل در worker threadها.

با این حال، `thread_local` جایگزین synchronization نیست.

اگر داده‌ای باید بین چند thread مشترک باشد، استفاده از `mutex`، atomic یا سازوکار هم‌زمانی مناسب همچنان ضروری است.

### ترکیب static و const

می‌توان `static` را با `const` ترکیب کرد:

```
#include <string>
#include <utility>

void getConfiguration() {
    static const auto [port, protocol] =
        std::pair{8080, std::string{"https"}};
}
```

در این مثال، متغیر پشتیبان const است و bindingها امکان تغییر مستقیم اعضای معمولی آن را ندارند.

این الگو برای configurationهای ثابت و چندجزئی می‌تواند خوانا باشد.

### استفاده از Attributeها

Structured Bindings از attributeهای مجاز در محل declaration پشتیبانی می‌کنند.

یکی از کاربردهای مفید، `[[maybe_unused]]` است:

```
#include <utility>

int main() {
    [[maybe_unused]] auto [width, height] =
        std::pair{640, 480};
}
```

این attribute زمانی مفید است که declaration از نظر منطقی ضروری باشد، اما برخی یا تمام نام‌های آن در ادامه برنامه استفاده نشوند.

البته محل قرارگیری attribute و دامنه اثر آن باید مطابق قواعد استاندارد بررسی شود.

## اصلاحات مرتبط با قواعد زبان

### دسترسی به اعضای خصوصی کلاس

یکی از تغییرات مرتبط با Structured Bindings از طریق پیشنهاد استانداردسازی P0969R0 — Allow structured bindings to accessible members انجام شد.

در نسخه اولیه قواعد، تجزیه اعضای کلاس با محدودیت عمومی بودن اعضا همراه بود.

اصلاح انجام‌شده این محدودیت را به مفهوم accessible بودن اعضا در context مربوط به declaration تغییر داد.

برای مثال:

```
class Point {
private:
    int x;
    int y;

public:
    Point(int xValue, int yValue)
        : x{xValue}, y{yValue} {}

    int sum() const {
        auto [first, second] = *this;

        return first + second;
    }
};
```

در این مثال، تابع عضو کلاس به اعضای private دسترسی دارد؛ بنابراین Structured Binding داخل همان context معتبر است.

اما کد زیر همچنان نامعتبر است:

```
class Point {
private:
    int x;
    int y;

public:
    Point(int xValue, int yValue)
        : x{xValue}, y{yValue} {}
};

int main() {
    Point point{10, 20};

    // Private members are inaccessible here.
    // auto [x, y] = point;
}
```

این اصلاح به معنی حذف encapsulation نیست.

نکته استانداردی مهم این است که P0969R0 به‌عنوان defect resolution برای قواعد Structured Bindings نیز در نظر گرفته می‌شود و نباید آن را صرفاً قابلیتی انحصاری برای حالت کامپایل ++C20 تلقی کرد.

### پروتکل Tuple-like

در تجزیه انواع tuple-like، سازوکار استاندارد شامل سه بخش اصلی است:

- `std::tuple_size`: تعداد اجزای قابل تجزیه را مشخص می‌کند.
- `std::tuple_element`: نوع هر جزء را تعیین می‌کند.
- `get`: مقدار هر جزء را در اختیار Structured Binding قرار می‌دهد.

برای نمونه:

```
#include <iostream>
#include <tuple>

int main() {
    std::tuple<int, double, char> data{
        10,
        2.5,
        'A'
    };

    auto [count, ratio, marker] = data;

    std::cout << count << ' '
              << ratio << ' '
              << marker << '\n';
}
```

در این مسیر، قواعد lookup برای تابع `get` اهمیت دارند.

اگر lookup نام qualified مربوط به member `get` را پیدا کند، آن تابع member استفاده می‌شود. در غیر این صورت، lookup غیرعضو با استفاده از ADL انجام می‌شود.

هنگام طراحی یک نوع tuple-like سفارشی باید این قواعد و الزامات `std::tuple_size` و `std::tuple_element` به‌دقت رعایت شوند.

همچنین افزودن specialization به namespace `std` فقط در موارد مجاز استاندارد قابل انجام است. برای نوع‌های کاربرتعریف‌شده، specializationهای مجاز مربوط به `tuple_size` و `tuple_element` کاربرد دارند.

## کاربردهای عملی در توسعه نرم‌افزار

### تجزیه خروجی توابع

یکی از رایج‌ترین کاربردهای Structured Bindings، دریافت چند مقدار از یک تابع است.

```
#include <string>
#include <utility>

std::pair<bool, std::string> validateInput(int value) {
    if (value >= 0) {
        return {true, "valid"};
    }

    return {false, "negative value"};
}

int main() {
    auto [success, message] = validateInput(10);

    if (!success) {
        // Handle validation failure.
    }
}
```

این روش برای APIهایی که وضعیت و نتیجه عملیات را برمی‌گردانند مناسب است.

البته اگر تعداد مقادیر زیاد باشد یا هر مقدار معنای پیچیده‌ای داشته باشد، تعریف یک نوع اختصاصی با اعضای نام‌دار ممکن است طراحی بهتری باشد.

### استفاده در Containerها

یکی از کاربردهای بسیار رایج Structured Bindings، پیمایش `std::map` است.

```
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<int, std::string> users{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"}
    };

    for (const auto& [id, name] : users) {
        std::cout << id << ": " << name << '\n';
    }
}
```

در این مثال:

- `id` به کلید map متصل می‌شود.
- `name` به مقدار متناظر متصل می‌شود.
- `const auto&` از کپی شدن غیرضروری pair جلوگیری می‌کند.
- کلیدهای `std::map` به دلیل ویژگی‌های container قابل تغییر نیستند.

اگر لازم باشد مقدارهای map تغییر کنند، می‌توان از `auto&` استفاده کرد:

```
for (auto& [id, name] : users) {
    name += " Updated";
}
```

در این حالت، تغییر مقدار مستقیماً روی container اعمال می‌شود.

### استفاده در الگوریتم‌های استاندارد

Structured Bindings در ترکیب با الگوریتم‌های استاندارد می‌توانند خوانایی کد را افزایش دهند.

```
#include <algorithm>
#include <utility>
#include <vector>

int main() {
    std::vector<std::pair<int, int>> points{
        {1, 2},
        {3, 4},
        {5, 6}
    };

    auto [minX, maxX] = std::pair{2, 4};

    auto count = std::count_if(
        points.begin(),
        points.end(),
        [minX, maxX](const auto& point) {
            return point.first >= minX &&
                   point.first <= maxX;
        }
    );
}
```

در این مثال، capture مستقیم متغیرهای binding باعث ساده‌تر شدن lambda شده است.

### استفاده در Callbackها

در طراحی callbackها، capture مستقیم می‌تواند نیاز به متغیرهای واسط را کاهش دهد.

```
#include <functional>
#include <utility>

int main() {
    auto [minimum, maximum] = std::pair{10, 100};

    std::function<bool(int)> isValid =
        [minimum, maximum](int value) {
            return value >= minimum &&
                   value <= maximum;
        };
}
```

در این مثال، lambda یک محدوده عددی را در closure object نگه می‌دارد.

چون capture به‌صورت مقداری انجام شده است، callback به عمر متغیرهای محلی وابستگی ندارد.

### استفاده در State Management

ترکیب `static` و Structured Bindings برای نگه‌داری state چندجزئی مفید است.

```
#include <string>
#include <utility>

std::string nextLabel() {
    static auto [index, prefix] =
        std::pair{0, std::string{"item"}};

    ++index;

    return prefix + "-" + std::to_string(index);
}
```

در این مثال، شمارنده بین فراخوانی‌های تابع حفظ می‌شود.

البته اگر چند thread به‌طور هم‌زمان این تابع را فراخوانی کنند، تغییرهای بعدی روی state مشترک نیازمند synchronization هستند.

## محدودیت‌ها و نکات مهم

### عدم پشتیبانی از constexpr در ++C20

یکی از محدودیت‌های مهم ++C20 این است که نمی‌توان declaration یک Structured Binding را مستقیماً با `constexpr` مشخص کرد.

برای مثال:

```
// Not valid in C++20.
constexpr auto [x, y] = std::pair{10, 20};
```

پشتیبانی از `constexpr` برای Structured Bindings در تغییرات استانداردی بعدی، از جمله ++C26، مطرح و پذیرفته شده است.

بنابراین نباید قابلیت‌های استانداردهای بعدی را به ++C20 نسبت داد.

### محدودیت inline و extern

افزوده شدن `static` و `thread_local` به معنی مجاز شدن تمام storage-class specifierها نیست.

به‌طور مشخص، `inline` و `extern` را نباید به‌عنوان مشخصه‌های مجاز declaration مربوط به Structured Binding در ++C20 در نظر گرفت.

### محدودیت Constraintها

در ++C20 نمی‌توان مستقیماً از constrained placeholder برای محدود کردن Structured Binding استفاده کرد.

برای مثال، syntax زیر معتبر نیست:

```
// Not valid C++20 syntax.
template <typename T>
concept PairLike = requires(T value) {
    value.first;
    value.second;
};

// Constrained structured binding syntax is not supported.
```

راهکار مناسب این است که constraint روی template یا پارامتر تابع اعمال شود و سپس Structured Binding داخل بدنه استفاده شود:

```
template <typename T>
requires requires(T value) {
    value.first;
    value.second;
}
auto sumMembers(const T& value) {
    auto [first, second] = value;

    return first + second;
}
```

این نمونه برای کلاس‌هایی مناسب است که علاوه بر داشتن اعضای موردنیاز، از نظر قواعد زبان نیز قابل تجزیه باشند.

### الزام تطابق تعداد اجزا

تعداد نام‌های Structured Binding باید با تعداد اجزای قابل تجزیه برابر باشد.

امکان حذف دلخواه برخی نام‌ها وجود ندارد.

برای مثال:

```
#include <utility>

int main() {
    auto [value, unused] = std::pair{42, 100};

    (void)unused;
}
```

در این مثال، هر دو جزء باید نام‌گذاری شوند؛ حتی اگر فقط یکی از آن‌ها موردنیاز باشد.

### هزینه کپی و Move

استفاده از `auto` می‌تواند باعث کپی یا move شدن شیء initializer به متغیر پشتیبان شود.

بنابراین هنگام کار با اشیای بزرگ یا انواع گران‌قیمت باید به انتخاب reference qualifier توجه کرد.

برای پیمایش containerها معمولاً این الگو مناسب است:

```
for (const auto& [key, value] : container) {
    // Read-only processing.
}
```

برای تغییر عناصر نیز می‌توان نوشت:

```
for (auto& [key, value] : container) {
    // Modify elements when permitted.
}
```

در مقابل، استفاده از `auto` بدون reference در range-based for ممکن است در هر iteration یک کپی از عنصر ایجاد کند.

### ملاحظات Lifetime

در declarationهایی که از reference استفاده می‌کنند، طول عمر شیء اصلی اهمیت زیادی دارد.

همچنین اگر bindingها در lambda به‌صورت ارجاعی capture شوند، عمر closure نباید از عمر اشیای مرجع فراتر برود.

در طراحی callbackهای ذخیره‌شونده، بهتر است مالکیت منابع به‌صورت شفاف مشخص شود و در صورت نیاز از capture مقداری یا init-capture استفاده شود.

### انتخاب نام‌های مناسب

Structured Bindings زمانی مفید هستند که نام اجزا معنای روشنی داشته باشد.

برای مثال:

```
auto [width, height] = dimensions;
```

از نظر خوانایی بهتر از نام‌های عمومی زیر است:

```
auto [a, b] = dimensions;
```

اگر ساختار داده پیچیده باشد یا اجزای زیادی داشته باشد، استفاده از یک نوع اختصاصی با اعضای نام‌دار می‌تواند انتخاب بهتری باشد.

### محدودیت‌های مربوط به کلاس‌های ارث‌بری‌شده

در تجزیه مستقیم اعضای کلاس، قواعد زبان محدودیت‌هایی برای inheritance و اعضای داده‌ای دارند.

به‌طور مشخص، کلاس قابل تجزیه باید شرایط ساختاری تعیین‌شده در استاندارد را رعایت کند. وجود اعضای داده‌ای ارث‌بری‌شده یا ترکیب نامناسب اعضای کلاس پایه و مشتق می‌تواند مانع Structured Binding شود.

برای کلاس‌های پیچیده، استفاده از پروتکل tuple-like یا accessorهای صریح معمولاً طراحی قابل‌کنترل‌تری فراهم می‌کند.

## مقایسه نسخه‌های مختلف استاندارد

| قابلیت                        | نسخه استاندارد                   |
| ----------------------------- | -------------------------------- |
| معرفی Structured Bindings     | ++C17                            |
| تجزیه آرایه‌ها                | ++C17                            |
| تجزیه انواع Tuple-like        | ++C17                            |
| تجزیه اعضای داده‌ای کلاس      | ++C17                            |
| Capture مستقیم در Lambda      | ++C20                            |
| Capture مقداری و ارجاعی       | ++C20                            |
| استفاده از `static`           | ++C20                            |
| استفاده از `thread_local`     | ++C20                            |
| اصلاح قواعد دسترسی اعضای کلاس | Defect Resolution مرتبط با ++C17 |
| پشتیبانی از `constexpr`       | ++C26                            |
| Structured Binding Packs      | ++C26                            |

نکته مهم این است که برخی اصلاحات استاندارد به‌صورت Defect Resolution به نسخه‌های قبلی نیز اعمال می‌شوند.

بنابراین تفاوت رفتار compilerها همیشه فقط به گزینه استاندارد انتخاب‌شده وابسته نیست؛ نسخه compiler و میزان پیاده‌سازی اصلاحات استاندارد نیز اهمیت دارد.

## پشتیبانی کامپایلرها

پشتیبانی از قابلیت‌های ++C20 به نسخه compiler وابسته است.

به‌خصوص قابلیت‌های زیر ممکن است در compilerهای قدیمی‌تر به‌طور کامل پیاده‌سازی نشده باشند:

- Capture مستقیم Structured Bindings در lambda.
- استفاده از `static` در declaration.
- استفاده از `thread_local`.
- برخی اصلاحات مرتبط با قواعد دسترسی.

برای آزمایش کدها می‌توان از دستورات زیر استفاده کرد:

```
g++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o app
```

```
clang++ -std=c++20 -Wall -Wextra -pedantic main.cpp -o app
```

توجه داشته باشید که انتخاب `-std=c++20` فقط نسخه زبان درخواستی را مشخص می‌کند و تضمینی برای پیاده‌سازی کامل تمام قابلیت‌ها در هر نسخه compiler نیست.

برای پروژه‌های حرفه‌ای، بررسی مستندات compiler و اجرای تست‌های سازگاری اهمیت زیادی دارد.

## اشتباهات رایج در استفاده از Structured Bindings

- تصور مستقل بودن bindingها: نام‌های حاصل از تجزیه به اجزای backing object متصل هستند و نباید همیشه آن‌ها را متغیرهای مستقل فرض کرد.
- بی‌توجهی به کپی: استفاده از `auto` ممکن است هزینه ساخت backing object را افزایش دهد.
- استفاده نادرست از reference capture: عمر closure نباید از عمر شیء مرجع بیشتر شود.
- فرض پشتیبانی constexpr در ++C20: این قابلیت به استانداردهای بعدی مربوط است.
- فرض دسترسی آزاد به private members: اصلاح قواعد دسترسی، encapsulation را حذف نمی‌کند.
- Capture ارجاعی bit-field: چنین captureای مجاز نیست.
- استفاده از static به‌عنوان ابزار synchronization: `static` طول عمر را مدیریت می‌کند، نه ایمنی دسترسی هم‌زمان را.
- استفاده از نام‌های مبهم: نام‌هایی مانند `a` و `b` ممکن است در ساختارهای پیچیده خوانایی را کاهش دهند.

## جمع‌بندی

Structured Bindings از ++C17 به‌عنوان راهکاری برای تجزیه مستقیم اشیا معرفی شدند و در ++C20 قابلیت‌های مهمی به دست آوردند.

مهم‌ترین بهبودهای ++C20 عبارت‌اند از:

- امکان capture مستقیم bindingها در lambda.
- پشتیبانی از capture مقداری و ارجاعی.
- امکان استفاده از `static`.
- امکان استفاده از `thread_local`.
- بهبود قابلیت استفاده از این ویژگی در طراحی کلاس‌ها و APIها.

این قابلیت‌ها باعث می‌شوند Structured Bindings در توسعه نرم‌افزارهای مدرن، پردازش containerها، طراحی callbackها و مدیریت state کاربردی‌تر باشند.

با این حال، استفاده حرفه‌ای از آن‌ها نیازمند توجه به جزئیاتی مانند نوع binding، تفاوت `auto` و referenceها، lifetime، هزینه کپی، محدودیت bit-fieldها و نسخه استاندارد هدف است.

در نهایت، Structured Bindings زمانی بیشترین ارزش را دارند که نام‌گذاری اجزا باعث افزایش خوانایی و شفافیت کد شود؛ نه اینکه صرفاً تعداد خطوط برنامه کاهش پیدا کند.

## منابع و مراجع

- [P1091R3 — Extending structured bindings to be more like variable declarations](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1091r3.html?utm_source=chatgpt.com)
- [P1381R1 — References to bindings](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1381r1.html?utm_source=chatgpt.com)
- [P0969R0 — Allow structured bindings to accessible members](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0969r0.pdf?utm_source=chatgpt.com)
- [cppreference — Structured binding declaration](https://en.cppreference.com/w/cpp/language/structured_binding?utm_source=chatgpt.com)
- [cppreference — Lambda expressions](https://en.cppreference.com/w/cpp/language/lambda?utm_source=chatgpt.com)

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>