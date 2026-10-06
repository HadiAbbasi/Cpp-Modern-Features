<div align="center">

[🇺🇸 English](../../en/cpp14/generic_lambda.md) | [🇮🇷 فارسی](./generic_lambda.md)

</div>

---

# Generic Lambda در ++C14

## معرفی Generic Lambda

قابلیت Generic Lambda در استاندارد ++C14 معرفی شد و یکی از مهم‌ترین بهبودهای Lambda Expression نسبت به ++C11 است. این قابلیت اجازه می‌دهد پارامترهای Lambda به‌جای داشتن یک نوع مشخص، با `auto` تعریف شوند و در نتیجه یک Lambda بتواند با انواع مختلف داده کار کند. 

مفهوم اصلی این قابلیت را می‌توان نوعی **generic callable** دانست که کامپایلر برای `operator()` آن پارامترهای Template ایجاد می‌کند. بنابراین Generic Lambda برای سناریوهایی مناسب است که منطق یکسانی باید برای چند نوع مختلف اجرا شود.

نمونه ساده این قابلیت در ++C14 به شکل زیر است:

```cpp
#include <iostream>

int main() {
    auto print = [](auto value) {
        std::cout << value << '\n';
    };

    print(42);
    print(3.14);
    print("hello");
}
```

در این مثال، نوع پارامتر `value` هنگام هر فراخوانی مشخص می‌شود. در نتیجه یک Lambda واحد می‌تواند با `int`، `double` و رشته کار کند.

## مفهوم پارامترهای auto

ویژگی اصلی Generic Lambda استفاده از `auto` در فهرست پارامترها است.

برای Lambda معمولی باید نوع پارامتر از قبل مشخص باشد:

```cpp
auto square = [](int value) {
    return value * value;
};
```

در مقابل، Generic Lambda نوع پارامتر را به کامپایلر واگذار می‌کند:

```cpp
auto square = [](auto value) {
    return value * value;
};
```

در حالت دوم، عبارت `auto` از نظر مفهومی مشابه یک Template Parameter عمل می‌کند.

برای مثال، فراخوانی‌های زیر می‌توانند specializationهای متفاوتی از `operator()` ایجاد کنند:

```cpp
auto square = [](auto value) {
    return value * value;
};

auto a = square(10);
auto b = square(2.5);
```

در اینجا یک specialization برای `int` و specialization دیگری برای `double` مورد نیاز است.

## نحوه پیاده‌سازی مفهومی

برای درک Generic Lambda بهتر است آن را به یک Function Object مقایسه کنیم.

کد زیر یک Lambda Generic است:

```cpp
auto add = [](auto lhs, auto rhs) {
    return lhs + rhs;
};
```

از نظر مفهومی می‌توان آن را شبیه یک Closure Type با `operator()` Template در نظر گرفت:

```cpp
struct Closure {
    template <typename T, typename U>
    auto operator()(T lhs, U rhs) const {
        return lhs + rhs;
    }
};
```

کد بالا یک **مدل مفهومی** برای درک رفتار Lambda است و معادل دقیق specification سطح زبان برای closure type محسوب نمی‌شود. نوع واقعی closure توسط کامپایلر ایجاد می‌شود و نام قابل‌دسترسی مستقیمی ندارد.

نکته مهم این است که `auto` در پارامتر Generic Lambda عملاً باعث ایجاد پارامتر Template می‌شود؛ بنابراین Lambda به یک callable polymorphic در زمان کامپایل تبدیل می‌شود.

## تفاوت Lambda معمولی و Generic Lambda

تفاوت اصلی این دو نوع Lambda در نحوه تعیین نوع پارامتر است.

```cpp
auto normal = [](int value) {
    return value * 2;
};

auto generic = [](auto value) {
    return value * 2;
};
```

در Lambda اول، فقط آرگومان‌هایی که با `int` سازگار هستند قابل استفاده‌اند.

در Lambda دوم، نوع آرگومان می‌تواند متفاوت باشد، مشروط بر اینکه عبارت `value * 2` برای آن نوع معتبر باشد.

بنابراین Generic بودن به معنی «قابل استفاده بودن با هر نوعی» نیست؛ بلکه به معنی **وابسته بودن نوع به Template Instantiation** است.

## استفاده از چند پارامتر auto

یک Generic Lambda می‌تواند چندین پارامتر `auto` داشته باشد:

```cpp
auto add = [](auto lhs, auto rhs) {
    return lhs + rhs;
};
```

این Lambda می‌تواند حتی دو نوع متفاوت دریافت کند:

```cpp
auto a = add(10, 20);
auto b = add(1.5, 2.5);
auto c = add(10, 2.5);
```

در فراخوانی سوم، پارامتر اول می‌تواند `int` و پارامتر دوم `double` باشد.

در نتیجه، Generic Lambda از نظر قابلیت Template می‌تواند چندین ترکیب مختلف از انواع را پشتیبانی کند.

## تفاوت Generic Lambda با Function Template

از نظر مفهومی، Generic Lambda بسیار شبیه Function Template است:

```cpp
template <typename T>
auto square(T value) {
    return value * value;
}
```

نسخه Lambda همین ایده را به شکل local و قابل‌استفاده به‌عنوان یک object ارائه می‌کند:

```cpp
auto square = [](auto value) {
    return value * value;
};
```

تفاوت مهم این است که Lambda یک **object با closure type** ایجاد می‌کند، در حالی که Function Template یک تابع Template مستقل است.

Generic Lambda زمانی بسیار مفید است که callable موردنظر فقط در یک محدوده مشخص لازم باشد و نیازی به تعریف یک تابع Template در سطح namespace یا scope بزرگ‌تر وجود نداشته باشد.

## کاربرد در الگوریتم‌های Standard Library

یکی از مهم‌ترین کاربردهای Generic Lambda استفاده به‌عنوان callback برای الگوریتم‌های Standard Library است.

برای نمونه:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    std::for_each(values.begin(), values.end(), [](auto value) {
        std::cout << value << '\n';
    });
}
```

در این مثال Lambda وابستگی مستقیمی به `int` ندارد و می‌تواند در موقعیت‌هایی که نوع عنصر متفاوت است نیز قابل استفاده باشد.

نمونه مشابه برای `std::vector<double>` نیز بدون تغییر Lambda قابل استفاده است:

```cpp
std::vector<double> values{1.1, 2.2, 3.3};

std::for_each(values.begin(), values.end(), [](auto value) {
    std::cout << value << '\n';
});
```

این الگو در Generic Programming بسیار رایج است.

## کاربرد در مقایسه و مرتب‌سازی

Generic Lambda می‌تواند برای نوشتن comparatorهای قابل استفاده مجدد نیز به کار رود:

```cpp
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> values{4, 1, 3, 2};

    std::sort(values.begin(), values.end(), [](auto lhs, auto rhs) {
        return lhs < rhs;
    });
}
```

با این حال، استفاده از Generic Lambda فقط زمانی مناسب است که واقعاً نیاز به generic بودن comparator وجود داشته باشد. اگر نوع کاملاً مشخص است و generic بودن مزیتی ایجاد نمی‌کند، Lambda معمولی نیز می‌تواند خواناتر باشد.

## مدیریت نوع بازگشتی

Generic Lambda در ++C14 از return type deduction پشتیبانی می‌کند.

```cpp
auto multiply = [](auto lhs, auto rhs) {
    return lhs * rhs;
};
```

نوع نتیجه از expression موجود در `return` استنتاج می‌شود.

برای مثال:

```cpp
auto a = multiply(2, 3);
auto b = multiply(2.5, 4.0);
```

در حالت اول نتیجه معمولاً `int` و در حالت دوم `double` خواهد بود، زیرا قوانین معمول type deduction و operatorهای ++C اعمال می‌شوند.

اگر لازم باشد نوع بازگشتی به‌صورت صریح تعیین شود، می‌توان از trailing return type استفاده کرد:

```cpp
auto multiply = [](auto lhs, auto rhs) -> double {
    return lhs * rhs;
};
```

در این حالت نتیجه `double` خواهد بود، حتی اگر expression داخلی نوع دیگری داشته باشد.

## استفاده از decltype(auto)

استاندارد ++C14 امکان استفاده از `decltype(auto)` را نیز در return type Lambda فراهم می‌کند:

```cpp
auto get_value = [](auto& value) -> decltype(auto) {
    return value;
};
```

این روش زمانی مهم است که بخواهیم category و type دقیق expression بازگردانده‌شده، از جمله reference بودن آن، حفظ شود.

برای مثال، تفاوت میان `auto` و `decltype(auto)` می‌تواند در موارد زیر مهم باشد:

```cpp
auto get_copy = [](auto& value) {
    return value;
};

auto get_reference = [](auto& value) -> decltype(auto) {
    return value;
};
```

در تابع اول، `auto` می‌تواند باعث شود نتیجه به‌صورت مقدار بازگردانده شود. در تابع دوم، `decltype(auto)` نوع دقیق expression را مطابق قوانین `decltype` حفظ می‌کند.

## استفاده از auto&&

Generic Lambda می‌تواند با `auto&&` نیز تعریف شود:

```cpp
auto process = [](auto&& value) {
    // Process value
};
```

این الگو یک **forwarding reference** ایجاد می‌کند و اجازه می‌دهد Lambda با lvalue و rvalue کار کند.

برای مثال:

```cpp
int value = 42;

process(value);
process(42);
```

در این حالت deduction برای `auto&&` مطابق قواعد Template Argument Deduction انجام می‌شود.

استفاده از `auto&&` زمانی مفید است که callable باید بتواند value category ورودی را حفظ کند یا در ادامه از perfect forwarding استفاده کند.

## استفاده از Perfect Forwarding

یکی از کاربردهای حرفه‌ای Generic Lambda ترکیب `auto&&` با `std::forward` است:

```cpp
#include <utility>

auto forward_value = [](auto&& value) {
    return std::forward<decltype(value)>(value);
};
```

در این الگو، نوع `value` و value category ورودی حفظ می‌شود.

این تکنیک به‌خصوص در wrapperها، adapterها و utilityهای generic اهمیت دارد.

البته perfect forwarding صرفاً به دلیل generic بودن Lambda ضروری نیست. زمانی باید از آن استفاده کرد که حفظ lvalue/rvalue بودن آرگومان واقعاً بخشی از قرارداد callable باشد.

## Capture در Generic Lambda

Generic بودن پارامترها مستقل از capture است. یک Lambda می‌تواند هم Generic باشد و هم متغیرهای محیط اطراف خود را capture کند.

```cpp
int factor = 10;

auto multiply = [factor](auto value) {
    return value * factor;
};
```

در این مثال، `factor` توسط value capture شده و `value` یک پارامتر generic است.

همچنین می‌توان از reference capture استفاده کرد:

```cpp
int factor = 10;

auto multiply = [&factor](auto value) {
    return value * factor;
};
```

در این حالت، Lambda به متغیر `factor` موجود در scope بیرونی وابسته است.

## Capture با Init-Capture

استاندارد ++C14 علاوه بر Generic Lambda، init-capture را نیز معرفی کرد و این دو قابلیت اغلب در کنار یکدیگر استفاده می‌شوند.

```cpp
auto create_multiplier = [](int factor) {
    return [factor](auto value) {
        return value * factor;
    };
};
```

در این مثال Lambda داخلی generic است و مقدار `factor` را در closure خود نگه می‌دارد.

الگوی فوق برای ساختن callableهای تخصصی‌شده در زمان اجرا بسیار کاربردی است.

## Closure Type در Generic Lambda

هر Lambda دارای یک closure type منحصربه‌فرد و بدون نام قابل‌دسترسی مستقیم است.

برای مثال:

```cpp
auto lambda = [](auto value) {
    return value * 2;
};
```

متغیر `lambda` یک object از closure type مربوط به همان Lambda است.

نوع دقیق آن را می‌توان با `decltype` به دست آورد:

```cpp
using LambdaType = decltype(lambda);
```

اما نمی‌توان نام واقعی closure type را مستقیماً در کد ++C نوشت، زیرا چنین نامی توسط زبان در اختیار برنامه‌نویس قرار نمی‌گیرد.

## تفاوت Generic Lambda با Runtime Polymorphism

Generic Lambda یک روش **compile-time polymorphism** است، نه runtime polymorphism.

در مثال زیر:

```cpp
auto print = [](auto value) {
    std::cout << value << '\n';
};
```

نوع `value` در زمان کامپایل مشخص می‌شود.

در مقابل، polymorphism مبتنی بر inheritance و `virtual` معمولاً در زمان اجرا dispatch می‌شود.

بنابراین Generic Lambda برای سناریوهایی مناسب است که نوع‌ها در زمان کامپایل مشخص هستند و می‌خواهیم هزینه و پیچیدگی runtime polymorphism را نداشته باشیم.

## استفاده با Typeهای سفارشی

Generic Lambda محدود به typeهای primitive نیست.

```cpp
struct User {
    int id;
};

auto get_id = [](auto const& user) {
    return user.id;
};
```

این Lambda می‌تواند برای هر typeای که member موردنظر را داشته باشد قابل استفاده باشد.

برای نمونه:

```cpp
User user{42};

auto id = get_id(user);
```

اگر type ورودی member موردنظر را نداشته باشد، خطا در زمان instantiation رخ می‌دهد.

این ویژگی هم قدرت Generic Programming است و هم یکی از محدودیت‌های آن.

## محدودیت خطاهای Template

یکی از مشکلات Generic Lambda در ++C14 این است که محدودیت‌های ورودی را نمی‌توان مستقیماً در syntax Lambda بیان کرد.

برای مثال:

```cpp
auto increment = [](auto value) {
    return value + 1;
};
```

این Lambda برای typeهایی که operator مناسب ندارند معتبر نیست.

در ++C14 نمی‌توان چیزی مشابه مفهوم C++20 Concepts را مستقیماً برای این Lambda نوشت.

در نتیجه، خطا ممکن است فقط زمانی ظاهر شود که Lambda با یک type نامعتبر instantiate شود و پیام compiler نیز در برخی موارد بسیار طولانی باشد.

## بررسی Constraints در نسخه‌های جدیدتر

محدودیت مهم Generic Lambdaهای ++C14 این است که mechanism استانداردی برای بیان constraints در syntax خود Lambda ندارند.

در ++C20 امکان استفاده از Concepts و constraints این وضعیت را بسیار بهتر کرده است.

برای نمونه، در ++C20 می‌توان Lambda را با constraint تعریف کرد:

```cpp
#include <concepts>

auto increment = []<std::integral T>(T value) {
    return value + 1;
};
```

این syntax متعلق به ++C20 است و نباید آن را با Generic Lambdaهای ++C14 اشتباه گرفت.

در ++C14 باید محدودیت‌ها معمولاً به‌صورت غیرمستقیم از طریق interface، helperهای Template، type traits یا طراحی API اعمال شوند.

## تفاوت Syntax در ++C14 و ++C20

Generic Lambda در ++C14 به `auto` در parameter list متکی است:

```cpp
auto lambda = [](auto value) {
    return value;
};
```

در ++C20 امکان نوشتن Template Parameter List صریح برای Lambda اضافه شد:

```cpp
auto lambda = []<typename T>(T value) {
    return value;
};
```

این دو syntax از نظر هدف مرتبط هستند، اما قابلیت‌های یکسانی ندارند.

در ++C14 نمی‌توان Template Parameter List صریح را در Lambda به این شکل تعریف کرد.

بنابراین اگر پروژه الزاماً ++C14 است، باید از syntax مبتنی بر `auto` استفاده شود.

## استفاده با std::function

Generic Lambda را می‌توان در `std::function` قرار داد، اما `std::function` خودش generic نیست و باید یک signature مشخص داشته باشد.

```cpp
#include <functional>

auto square = [](auto value) {
    return value * value;
};

std::function<int(int)> integer_square = square;
std::function<double(double)> double_square = square;
```

در این حالت، هر `std::function` یک signature مشخص دارد و Generic Lambda متناسب با همان signature مورد استفاده قرار می‌گیرد.

نکته مهم این است که قرار دادن callable در `std::function` ممکن است نسبت به استفاده مستقیم از Lambda هزینه و type erasure اضافی داشته باشد. بنابراین اگر type erasure نیاز نیست، نگه‌داشتن callable به‌صورت generic معمولاً ساده‌تر و کم‌هزینه‌تر است.

## تبدیل Lambda بدون Capture به Function Pointer

Lambdaای که capture ندارد می‌تواند در شرایط مناسب به function pointer تبدیل شود.

برای Generic Lambda نیز این قابلیت در ++C14 وجود دارد، اما نوع target باید مشخص باشد تا specialization مناسب تعیین شود.

```cpp
auto add = [](auto lhs, auto rhs) {
    return lhs + rhs;
};

int (*int_add)(int, int) = add;
```

در اینجا target type مشخص می‌کند که specialization موردنیاز Lambda باید با دو `int` کار کند.

همچنین می‌توان نوع دیگری از function pointer را انتخاب کرد:

```cpp
double (*double_add)(double, double) = add;
```

این تبدیل فقط برای Lambdaهای بدون capture قابل استفاده است.

## تفاوت Capture و Function Pointer

Lambda دارای capture نمی‌تواند به function pointer معمولی تبدیل شود.

```cpp
int factor = 10;

auto multiply = [factor](int value) {
    return value * factor;
};
```

این Lambda به state داخلی closure وابسته است و بنابراین function pointer معمولی نمی‌تواند آن state را در خود نگه دارد.

در چنین شرایطی استفاده مستقیم از Lambda یا type-erased callable مانند `std::function` مناسب‌تر است، بسته به نیاز طراحی.

## Generic Lambda به‌عنوان Visitor

یکی از کاربردهای مهم Generic Lambda ساخت visitorهای ساده برای typeهای مختلف است.

در ++C14 هنوز `std::variant` وجود ندارد؛ بنابراین مثال‌های مبتنی بر `std::variant` به استاندارد ++C17 تعلق دارند.

با این حال، Generic Lambda می‌تواند در بسیاری از طراحی‌های visitor-like یا callback-based استفاده شود:

```cpp
auto visitor = [](auto const& value) {
    std::cout << value << '\n';
};
```

این الگو در کتابخانه‌ها و الگوریتم‌های generic بسیار رایج است.

## استفاده در Wrapperها

Generic Lambda برای ساخت wrapperهایی که فقط منطق کوچکی را به callable دیگری اضافه می‌کنند بسیار مناسب است.

```cpp
auto log_call = [](auto&& callable) {
    return [callable = std::forward<decltype(callable)>(callable)](auto&& value) {
        std::cout << "Calling callable\n";
        return callable(std::forward<decltype(value)>(value));
    };
};
```

این مثال چند قابلیت ++C14 را با هم ترکیب می‌کند:

* Generic Lambda
* `auto&&`
* init-capture
* perfect forwarding
* closure object

چنین الگوهایی در utility code و library code بسیار قدرتمند هستند، اما باید مراقب lifetime و ownership بود.

## نکات مربوط به Lifetime

وقتی Lambda چیزی را با reference capture می‌کند، lifetime آن object باید حداقل تا زمان استفاده از Lambda ادامه داشته باشد.

```cpp
auto create_lambda() {
    int value = 42;

    return [&value](auto x) {
        return value + x;
    };
}
```

کد بالا مشکل lifetime دارد، زیرا Lambda بعد از پایان `create_lambda` به متغیری reference می‌کند که دیگر وجود ندارد.

نسخه امن‌تر می‌تواند value capture باشد:

```cpp
auto create_lambda() {
    int value = 42;

    return [value](auto x) {
        return value + x;
    };
}
```

Generic بودن Lambda هیچ مشکل lifetime مربوط به capture را برطرف نمی‌کند.

## نکات مربوط به const و mutable

به‌صورت پیش‌فرض، `operator()` یک Lambda معمولی از نظر const بودن رفتار مشخصی دارد و mutable بودن Lambda را می‌توان با `mutable` تغییر داد.

برای مثال:

```cpp
int counter = 0;

auto increment = [counter]() mutable {
    ++counter;
    return counter;
};
```

در اینجا Lambda یک copy داخلی از `counter` دارد و `mutable` اجازه می‌دهد state داخلی آن تغییر کند.

Generic بودن پارامتر هیچ ارتباط مستقیمی با `mutable` ندارد:

```cpp
auto increment = [counter = 0](auto value) mutable {
    ++counter;
    return value + counter;
};
```

در این مثال هم Lambda generic است و هم state داخلی mutable دارد.

## تفاوت Generic Lambda و Overload Set

گاهی یک Lambda generic می‌تواند جای چند overload ساده را بگیرد:

```cpp
auto print = [](auto const& value) {
    std::cout << value << '\n';
};
```

در مقابل، اگر رفتار برای typeهای مختلف واقعاً متفاوت باشد، Generic Lambda ممکن است انتخاب مناسبی نباشد.

در چنین شرایطی می‌توان منطق را با overloadهای جداگانه یا طراحی‌های مناسب‌تر بیان کرد.

اصل مهم این است که generic بودن به‌خودی‌خود هدف نیست؛ باید باعث ساده‌تر و maintainableتر شدن طراحی شود.

## خطاهای رایج

### استفاده از Generic Lambda بدون نیاز واقعی

اگر Lambda فقط با یک type مشخص کار می‌کند، استفاده از `auto` الزاماً مزیتی ندارد.

```cpp
auto process = [](int value) {
    return value * 2;
};
```

در چنین سناریویی generic کردن بی‌دلیل می‌تواند قرارداد callable را مبهم‌تر کند.

### فرض اینکه هر typeای پذیرفته می‌شود

Generic بودن به معنی universal compatibility نیست.

```cpp
auto double_value = [](auto value) {
    return value * 2;
};
```

این Lambda فقط با typeهایی کار می‌کند که expression مربوط به multiplication برای آن‌ها معتبر باشد.

### بی‌توجهی به Reference Semantics

استفاده از `auto` و `auto&&` رفتار متفاوتی دارد:

```cpp
auto by_value = [](auto value) {
    // Process a copy
};

auto by_reference = [](auto&& value) {
    // Preserve reference category
};
```

انتخاب بین این دو باید بر اساس نیاز واقعی انجام شود.

### استفاده نابجا از std::function

اگر فقط به یک callable محلی نیاز داریم، معمولاً نیازی نیست آن را فوراً داخل `std::function` قرار دهیم.

```cpp
auto operation = [](auto value) {
    return value * 2;
};
```

نگه‌داشتن مستقیم Lambda معمولاً type information را حفظ می‌کند و از type erasure غیرضروری جلوگیری می‌کند.

### نادیده گرفتن پیام خطای Template

وقتی یک Generic Lambda با type نامناسب استفاده می‌شود، خطای اصلی ممکن است در محل instantiation ظاهر شود و پیام compiler طولانی باشد.

به همین دلیل در APIهای عمومی و library code، مشخص کردن constraints یا طراحی interface محدودتر می‌تواند کیفیت خطاها را بهتر کند.

## محدودیت‌های Generic Lambda در ++C14

مهم‌ترین محدودیت‌های این قابلیت را می‌توان چنین خلاصه کرد:

* پارامترهای generic در ++C14 با `auto` تعریف می‌شوند.
* Template Parameter List صریح برای Lambda در ++C14 وجود ندارد.
* Concepts و requires-expression در دسترس نیستند.
* محدود کردن typeهای مجاز مستقیماً در syntax Lambda امکان‌پذیر نیست.
* خطاهای instantiation ممکن است پیچیده باشند.
* Lambda دارای capture نمی‌تواند به function pointer معمولی تبدیل شود.
* Generic بودن هیچ‌گونه تضمینی درباره lifetime objectهای captureشده ایجاد نمی‌کند.
* generic بودن ممکن است باعث ایجاد specializationهای متعدد در زمان کامپایل شود.
* استفاده بی‌دلیل از genericity می‌تواند خوانایی قرارداد API را کاهش دهد.

## تأثیر بر Compile Time و Code Size

Generic Lambda بر مبنای Template Instantiation کار می‌کند. بنابراین اگر یک Lambda با تعداد زیادی type متفاوت استفاده شود، کامپایلر می‌تواند specializationهای متعددی ایجاد کند.

برای مثال:

```cpp
auto process = [](auto value) {
    return value * 2;
};

process(1);
process(2L);
process(3.0);
process(4.0f);
```

در چنین حالتی، بسته به نوع‌ها و optimizationهای compiler، کد مربوط به specializationهای مختلف ممکن است تولید شود.

این موضوع ذاتاً مشکل نیست و یکی از ویژگی‌های compile-time polymorphism محسوب می‌شود، اما در libraryهای بزرگ باید اثر Template instantiation بر compile time و اندازه binary در نظر گرفته شود.

## نکات طراحی برای کد حرفه‌ای

استفاده حرفه‌ای از Generic Lambda بیشتر از صرفاً جایگزین کردن typeها با `auto` است.

بهتر است قبل از generic کردن یک Lambda این پرسش‌ها بررسی شوند:

* آیا منطق واقعاً برای چند type معتبر است؟
* آیا interface موردنظر باید generic باشد؟
* آیا constraints مشخصی وجود دارد؟
* آیا `auto`، `auto&`، `const auto&` یا `auto&&` انتخاب مناسبی است؟
* آیا value category ورودی باید حفظ شود؟
* آیا Lambda چیزی را capture می‌کند؟
* اگر capture reference است، lifetime آن تضمین شده است؟
* آیا `std::function` واقعاً لازم است؟
* آیا generic بودن باعث بهبود reuse می‌شود یا فقط قرارداد را مبهم می‌کند؟
* آیا خطاهای Template برای کاربران API قابل‌قبول خواهند بود؟

Generic Lambda زمانی بیشترین ارزش را دارد که genericity بخشی واقعی از طراحی باشد، نه صرفاً یک تکنیک کوتاه‌تر برای حذف نام type.

## مقایسه با رویکردهای قدیمی‌تر

پیش از Generic Lambda در ++C14، Lambdaهای ++C11 نمی‌توانستند پارامتر `auto` داشته باشند.

در ++C11 برای یک عملیات generic معمولاً باید از Function Template یا Function Object Template استفاده می‌شد:

```cpp
struct Multiplier {
    template <typename T>
    auto operator()(T value) const {
        return value * 2;
    }
};
```

در ++C14 همین ایده را می‌توان بسیار کوتاه‌تر نوشت:

```cpp
auto multiplier = [](auto value) {
    return value * 2;
};
```

بنابراین Generic Lambda یکی از مواردی است که syntax پیچیده‌تر Function Objectهای قدیمی را برای callableهای محلی و کوتاه کاهش می‌دهد.

## تفاوت با Generic Programming کامل

Generic Lambda فقط یک ابزار برای Generic Programming است و جایگزین تمام قابلیت‌های Templateها نیست.

اگر به موارد زیر نیاز داشته باشیم، ممکن است Function Template یا class template انتخاب مناسب‌تری باشد:

* interface عمومی و نام‌دار
* چندین overload مرتبط
* specializationهای Template
* constraints پیچیده در نسخه‌های قدیمی استاندارد
* state و behavior پیچیده
* reuse گسترده در بخش‌های مختلف برنامه
* APIای که باید به‌صورت مشخص توسط کاربران کتابخانه فراخوانی شود

در مقابل، برای callbackهای محلی، الگوریتم‌ها، adapterها و عملیات کوچک generic، Lambda معمولاً انتخاب بسیار مناسبی است.

## نسخه‌های مختلف استاندارد

قابلیت Generic Lambda با پارامترهای `auto` در **++C14** معرفی شد.

در **++C17** قابلیت‌های Lambda توسعه بیشتری پیدا کردند؛ از جمله امکان استفاده گسترده‌تر از Lambdaها در contextهای compile-time و ویژگی‌های مرتبط دیگر.

در **++C20** Lambdaها قابلیت‌های مهم‌تری مانند Template Parameter List صریح و constraints مبتنی بر Concepts دریافت کردند.

در نتیجه، اگر کد باید دقیقاً با ++C14 سازگار باشد، نباید syntax و قابلیت‌های جدیدتر را با Generic Lambdaهای ++C14 مخلوط کرد.

## مثال کاربردی کامل

مثال زیر یک Generic Lambda را برای محاسبه مجموع عناصر یک container نشان می‌دهد:

```cpp
#include <iostream>
#include <numeric>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5};

    auto add = [](auto lhs, auto rhs) {
        return lhs + rhs;
    };

    int result = std::accumulate(
        values.begin(),
        values.end(),
        0,
        add
    );

    std::cout << result << '\n';
}
```

در این مثال، Lambda مستقل از نوع دقیق پارامترهای خود تعریف شده است و می‌تواند در contextهای دیگری که عملیات `+` معتبر است نیز مورد استفاده قرار گیرد.

## جمع‌بندی

Generic Lambda در ++C14 راهی مختصر و قدرتمند برای ساخت callableهای generic در زمان کامپایل است. مهم‌ترین ویژگی آن استفاده از `auto` در parameter list است که باعث می‌شود `operator()` مربوط به closure به‌صورت Template عمل کند.

مهم‌ترین نکته این است که Generic Lambda را نباید صرفاً «Lambdaای که هر نوعی را قبول می‌کند» در نظر گرفت. در واقع، Lambda برای typeهایی معتبر است که body آن برای آن typeها قابل instantiate باشد.

ترکیب Generic Lambda با `auto&&`، perfect forwarding، init-capture و الگوریتم‌های Standard Library امکان ساخت utilityهای بسیار قدرتمند و concise را فراهم می‌کند.

با وجود این، محدودیت‌های ++C14 مانند نبود Concepts، نبود Template Parameter List صریح و پیچیده شدن برخی خطاهای Template باید در طراحی APIهای عمومی در نظر گرفته شوند.

برای کد حرفه‌ای، بهترین رویکرد این است که genericity زمانی استفاده شود که واقعاً بخشی از مسئله باشد؛ در غیر این صورت، Lambda با type مشخص معمولاً قرارداد روشن‌تر و کد قابل‌فهم‌تری ارائه می‌کند.


---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>