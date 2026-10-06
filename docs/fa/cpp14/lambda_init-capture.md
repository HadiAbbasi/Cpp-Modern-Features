<div align="center">

[🇺🇸 English](../../en/cpp14/lambda_init-capture.md) | [🇮🇷 فارسی](./lambda_init-capture.md)

</div>

---

# مفهوم init-capture در Lambdaهای ++C14

## مقدمه

قابلیت `init-capture` در lambda expressionهای ++C14 یکی از مهم‌ترین امکانات اضافه‌شده به ++C14 است. این قابلیت اجازه می‌دهد هنگام ساخت closure object، مقدار اولیه یا حتی نوع متفاوتی برای یک capture تعیین شود.

پیش از ++C14، captureهای lambda عمدتاً به شکل capture معمولی انجام می‌شدند؛ برای مثال، `[x]` یک متغیر موجود در scope را با کپی capture می‌کرد و `[&x]` همان متغیر را با reference در اختیار closure قرار می‌داد. این مدل برای بسیاری از کاربردها کافی بود، اما برای سناریوهایی مانند انتقال ownership یک `std::unique_ptr` به داخل lambda محدودیت جدی داشت.

ویژگی `init-capture` این مسئله را با اجازه‌دادن به syntaxهایی مانند `[p = std::move(p)]` حل می‌کند. در این حالت، lambda یک member داخلی با نام `p` خواهد داشت که از expression سمت راست مقداردهی اولیه شده است.

این قابلیت پایه‌ای برای الگوهای مهمی مانند move-capture، ساخت closureهای self-contained، مدیریت lifetime و انتقال ownership به callbackها محسوب می‌شود.

## تعریف init-capture

در یک lambda معمولی، capture معمولاً به یک متغیر موجود اشاره می‌کند:

```cpp
int value = 42;

auto lambda = [value]() {
    return value;
};
```

در مقابل، در `init-capture` یک نام جدید داخل closure معرفی می‌شود و مقدار آن با یک initializer تعیین می‌شود:

```cpp
int value = 42;

auto lambda = [captured = value]() {
    return captured;
};
```

در این مثال، نام `captured` نامی است که داخل body lambda قابل استفاده است. این نام لزوماً نام متغیر خارجی نیست.

به بیان مفهومی، عبارت `[captured = value]` می‌گوید:

> یک state داخلی برای closure با نام `captured` ایجاد کن و آن را با `value` مقداردهی اولیه کن.

این تفاوت، مهم‌ترین ویژگی `init-capture` است: می‌توان state داخلی closure را مستقل از نام و حتی مستقل از lifetime متغیر محلی ساخت.

## دلیل وجود init-capture

یکی از محدودیت‌های مهم capture معمولی این بود که capture باید به یک متغیر موجود در surrounding scope متصل باشد.

برای مثال، این کد در ++C14 کاربرد مستقیم move-capture ندارد:

```cpp
std::unique_ptr<int> ptr = std::make_unique<int>(42);

auto lambda = [ptr]() {
    return *ptr;
};
```

کپی‌کردن `std::unique_ptr` مجاز نیست، بنابراین `[ptr]` نمی‌تواند ownership را به closure منتقل کند.

با `init-capture` می‌توان به‌صورت مستقیم ownership را منتقل کرد:

```cpp
std::unique_ptr<int> ptr = std::make_unique<int>(42);

auto lambda = [ptr = std::move(ptr)]() {
    return *ptr;
};
```

در اینجا `ptr` سمت چپ نام state داخل closure است و `std::move(ptr)` سمت راست expressionای است که برای ساخت آن state استفاده می‌شود.

پس از ایجاد lambda، متغیر بیرونی `ptr` در حالت moved-from قرار دارد و ownership داخل closure قرار گرفته است.

## نحو پایه init-capture

شکل اصلی `init-capture` به‌صورت زیر است:

```cpp
[identifier = initializer]
```

برای نمونه:

```cpp
int x = 10;

auto lambda = [value = x + 5]() {
    return value;
};
```

در این مثال، مقدار `value` برابر `15` خواهد بود.

همچنین می‌توان چند `init-capture` را هم‌زمان نوشت:

```cpp
int width = 10;
int height = 20;

auto area = [w = width, h = height]() {
    return w * h;
};
```

در اینجا closure دو state داخلی به نام‌های `w` و `h` دارد.

## تفاوت capture معمولی و init-capture

در capture معمولی، نام capture با نام متغیر بیرونی یکسان است:

```cpp
int value = 42;

auto lambda = [value]() {
    return value;
};
```

اما در `init-capture` می‌توان نام داخلی متفاوتی انتخاب کرد:

```cpp
int value = 42;

auto lambda = [capturedValue = value]() {
    return capturedValue;
};
```

این تفاوت فقط جنبه ظاهری ندارد. `init-capture` اجازه می‌دهد initializer یک expression دلخواه باشد:

```cpp
int x = 10;
int y = 20;

auto lambda = [sum = x + y]() {
    return sum;
};
```

در اینجا `sum` اصلاً متغیر مستقلی در scope بیرونی نیست؛ state آن هنگام ساخت closure ایجاد شده است.

## نحوه تعیین نوع در init-capture

در فرم معمول `init-capture`، نوع state داخلی بر اساس initializer تعیین می‌شود؛ از این نظر می‌توان آن را شبیه یک مقداردهی با `auto` در نظر گرفت.

برای مثال:

```cpp
const int value = 42;

auto lambda = [x = value]() {
    return x;
};
```

نوع `x` از expression مقداردهی تعیین می‌شود و `const` بودن متغیر بیرونی به‌صورت خودکار به یک `const` member متناظر تبدیل نمی‌شود.

این نکته مهم است، زیرا `init-capture` را نباید صرفاً به‌عنوان شکل دیگری از `[value]` در نظر گرفت. در واقع، initializer تعیین می‌کند که state داخلی closure چگونه ساخته شود.

## Move-capture و انتقال ownership

یکی از مهم‌ترین کاربردهای `init-capture`، انتقال یک object move-only به داخل lambda است.

نمونه کلاسیک، `std::unique_ptr` است:

```cpp
#include <iostream>
#include <memory>

int main() {
    auto ptr = std::make_unique<int>(42);

    auto lambda = [ptr = std::move(ptr)]() {
        std::cout << *ptr << '\n';
    };

    lambda();
}
```

در این مثال، lambda مالک object pointed-to است.

این الگو برای callbackها بسیار مهم است:

```cpp
#include <memory>
#include <thread>

void startAsync(std::unique_ptr<int> data) {
    std::thread worker(
        [data = std::move(data)]() {
            // Use the owned data here.
        }
    );

    worker.detach();
}
```

در چنین سناریویی، ownership می‌تواند همراه callback حرکت کند و callback state مورد نیاز خود را مستقل از caller نگه دارد.

البته در برنامه‌های واقعی، استفاده از `detach()` نیازمند بررسی دقیق lifetime و synchronization است. خود `init-capture` هیچ تضمینی درباره thread safety یا synchronization ایجاد نمی‌کند.

## تفاوت move-capture با reference capture

این دو الگو رفتار کاملاً متفاوتی دارند:

```cpp
auto lambda1 = [value]() {
    return value;
};

auto lambda2 = [&value]() {
    return value;
};
```

در حالت اول، closure یک state مستقل دارد. تغییر `value` بیرونی پس از ایجاد closure روی state داخلی اثر نمی‌گذارد.

در حالت دوم، closure به object بیرونی reference دارد و lifetime آن object باید تا پایان استفاده از lambda معتبر باقی بماند.

`init-capture` می‌تواند به‌طور صریح ownership یا state مستقل بسازد:

```cpp
auto lambda = [value = std::move(value)]() {
    return value;
};
```

این تفاوت در طراحی callbackها بسیار مهم است: اگر callback باید state را با خود حمل کند، value capture یا init-capture معمولاً مدل ownership روشن‌تری نسبت به reference capture ارائه می‌دهد.

## ساخت state محاسبه‌شده برای closure

گاهی lambda به یک مقدار محاسبه‌شده نیاز دارد که بهتر است فقط یک بار، هنگام ساخت closure، محاسبه شود.

برای نمونه:

```cpp
std::string makePrefix();

auto print = [prefix = makePrefix()]() {
    // Use the already-created prefix.
};
```

در این حالت، `makePrefix()` هنگام ساخت lambda اجرا می‌شود، نه هر بار که lambda فراخوانی شود.

این ویژگی برای ساخت callbackهایی که configuration یا state از پیش آماده‌شده دارند مفید است:

```cpp
int timeout = 100;

auto callback = [
    timeoutMs = timeout,
    message = std::string("request completed")
]() {
    // Use timeoutMs and message.
};
```

چنین closureای state مورد نیاز خود را در زمان construction دریافت می‌کند.

## محدودکردن state به آنچه واقعاً لازم است

`init-capture` می‌تواند به جلوگیری از capture کردن scope بزرگ‌تر کمک کند.

برای مثال، به‌جای نگه‌داشتن یک object بزرگ فقط برای دسترسی به یک مقدار کوچک:

```cpp
struct Config {
    int timeout;
    std::string host;
    std::string certificate;
};

Config config;

auto callback = [config]() {
    // Use only config.timeout.
};
```

می‌توان فقط مقدار مورد نیاز را capture کرد:

```cpp
auto callback = [timeout = config.timeout]() {
    // Use timeout only.
};
```

این کار می‌تواند lifetime و ownership را شفاف‌تر کند و coupling closure به objectهای بزرگ‌تر را کاهش دهد.

البته اگر stateهای دیگر نیز واقعاً مورد نیاز باشند، نباید صرفاً برای کوچک‌کردن capture list طراحی را مصنوعی کرد.

## تغییر نام هنگام capture

یکی از کاربردهای ساده اما مفید `init-capture`، تغییر نام state داخلی است:

```cpp
std::string requestId = "abc";

auto callback = [id = requestId]() {
    // Use id here.
};
```

این تکنیک زمانی مفید است که نام متغیر بیرونی برای context داخل lambda مناسب نیست یا می‌خواهیم وابستگی closure به نام بیرونی را کمتر کنیم.

همچنین می‌توان expression پیچیده‌تری را پشت یک نام معنادار قرار داد:

```cpp
auto callback = [
    normalizedPath = normalizePath(inputPath)
]() {
    // Use normalizedPath here.
};
```

در این مثال، closure به نتیجه نهایی وابسته است، نه به جزئیات محاسبه آن.

## reference init-capture

`init-capture` فقط برای ساخت state value-like نیست و می‌تواند با reference نیز نوشته شود:

```cpp
int value = 42;

auto lambda = [&ref = value]() {
    ++ref;
};
```

در این حالت، `ref` یک reference داخلی برای object مورد نظر است.

استفاده از این فرم باید با دقت انجام شود، زیرا lifetime همچنان به object اصلی وابسته است.

برای مثال:

```cpp
auto makeLambda() {
    int value = 42;

    return [&ref = value]() {
        return ref;
    };
}
```

این کد مشکل lifetime دارد. پس از خروج از `makeLambda`، object مربوط به `value` از بین رفته است و lambda یک dangling reference خواهد داشت.

این مشکل با syntax زیباتر یا با استفاده از `init-capture` از بین نمی‌رود.

## تفاوت value init-capture و reference init-capture

دو الگوی زیر از نظر ownership و lifetime کاملاً متفاوت‌اند:

```cpp
auto byValue = [copy = value]() {
    return copy;
};

auto byReference = [&ref = value]() {
    return ref;
};
```

در حالت اول، closure state مستقل خود را دارد.

در حالت دوم، closure به object خارجی متکی است.

در طراحی callback، این سؤال باید همیشه روشن باشد:

> آیا callback باید object را با خود نگه دارد، یا فقط تا زمانی به object موجود دیگری دسترسی داشته باشد؟

اگر پاسخ اول باشد، value-based state معمولاً مدل مناسب‌تری است. اگر پاسخ دوم باشد، reference می‌تواند مناسب باشد، اما lifetime باید به‌صورت دقیق تضمین شود.

## init-capture و `mutable`

به‌صورت پیش‌فرض، `operator()` یک lambda مقدارهای captureشده را به‌صورت const مشاهده می‌کند. اگر بخواهیم state داخلی value-captured را تغییر دهیم، باید lambda را `mutable` کنیم:

```cpp
int value = 10;

auto counter = [value]() mutable {
    ++value;
    return value;
};

counter();
counter();
```

در این مثال، تغییر `value` فقط state داخلی closure را تغییر می‌دهد و متغیر بیرونی `value` تغییر نمی‌کند.

همین موضوع با `init-capture` نیز برقرار است:

```cpp
auto counter = [count = 0]() mutable {
    return ++count;
};
```

این الگو برای ساخت closureهای stateful بسیار رایج است.

نکته مهم این است که `mutable` به معنی thread-safe بودن closure نیست. اگر چند thread به یک closure stateful دسترسی داشته باشند، synchronization همچنان بر عهده برنامه است.

## init-capture و `const`

اگر initializer یک object `const` باشد، باید به تفاوت نوع object اصلی و state داخلی توجه کرد:

```cpp
const int value = 42;

auto lambda = [copy = value]() mutable {
    ++copy;
    return copy;
};
```

در اینجا `copy` می‌تواند تغییر کند، چون state داخلی lambda یک object مستقل است و نوع آن از initializer به شکل مناسب deduce شده است.

در مقابل، reference capture همان object اصلی را مشاهده می‌کند:

```cpp
const int value = 42;

auto lambda = [&value]() {
    return value;
};
```

این تفاوت یکی از دلایلی است که باید بین «capture کردن object» و «capture کردن reference به object» تمایز قائل شد.

## init-capture و `this`

در ++C14، lambda می‌تواند `this` را capture کند:

```cpp
class Widget {
public:
    int value = 42;

    auto makeLambda() {
        return [this]() {
            return value;
        };
    }
};
```

اما capture کردن `this` در این فرم به معنی نگه‌داشتن object به‌صورت value نیست. lambda یک pointer به object را در اختیار دارد و lifetime خود object همچنان باید تا زمان استفاده از lambda معتبر باشد.

یکی از مشکلات رایج این است:

```cpp
class Widget {
public:
    auto makeLambda() {
        return [this]() {
            return value();
        };
    }

    int value() const {
        return 42;
    }
};
```

اگر object مربوط به `Widget` قبل از اجرای lambda از بین برود، lambda dangling خواهد شد.

در ++C14، می‌توان با `init-capture` یک copy از object ساخت:

```cpp
class Widget {
public:
    int value = 42;

    auto makeLambda() {
        return [self = *this]() {
            return self.value;
        };
    }
};
```

این روش با `[this]` تفاوت اساسی دارد: در اینجا `self` یک object مستقل داخل closure است.

در ++C17، syntax مستقیم `[ *this ]` نیز برای value capture کردن object معرفی شد:

```cpp
class Widget {
public:
    int value = 42;

    auto makeLambda() {
        return [*this]() {
            return value;
        };
    }
};
```

بنابراین در کد ++C17 و جدیدتر، `[ *this ]` معمولاً بیان مستقیم‌تری برای همین مفهوم است؛ در ++C14 می‌توان از `[self = *this]` استفاده کرد.

## محدودیت‌های کپی‌کردن `*this`

ساختن copy از object با `[self = *this]` یا `[ *this ]` الزاماً همیشه انتخاب مناسبی نیست.

اگر class دارای state بزرگ باشد، هر closure می‌تواند یک copy مستقل از آن state ایجاد کند. همچنین اگر class دارای resourceهایی با semantics خاص باشد، copy شدن object ممکن است اصلاً ممکن نباشد.

برای مثال، objectی که شامل `std::unique_ptr` است به‌صورت پیش‌فرض copyable نیست:

```cpp
#include <memory>

class Widget {
    std::unique_ptr<int> data;

public:
    auto makeLambda() {
        return [self = *this]() {
            return *self.data;
        };
    }
};
```

این کد به دلیل copy کردن `*this` قابل کامپایل نیست، مگر اینکه class مورد نظر copyable باشد.

بنابراین value capture کردن `*this` باید با semantics کپی object سازگار باشد.

## init-capture و objectهای move-only

`init-capture` برای هر نوعی که move semantics دارد قابل استفاده است، نه فقط `std::unique_ptr`.

برای نمونه:

```cpp
#include <fstream>

auto file = std::make_unique<std::ofstream>("output.txt");

auto writer = [file = std::move(file)]() {
    *file << "hello\n";
};
```

همچنین می‌توان objectهای سفارشی را منتقل کرد:

```cpp
class Connection {
public:
    Connection(Connection&&) = default;
    Connection(const Connection&) = delete;

    void send() {
    }
};

Connection connection;

auto callback = [connection = std::move(connection)]() mutable {
    connection.send();
};
```

در این الگو، closure مالک یا دارنده state منتقل‌شده می‌شود.

## init-capture و perfect forwarding

در template code، `init-capture` می‌تواند برای تبدیل یک ورودی به state مناسب closure استفاده شود، اما باید تفاوت آن با perfect forwarding را در نظر گرفت.

برای مثال:

```cpp
template <typename T>
auto makeHandler(T&& value) {
    return [stored = std::forward<T>(value)]() mutable {
        return stored;
    };
}
```

در اینجا `std::forward<T>` تصمیم می‌گیرد object چگونه به closure منتقل شود و `stored` state نهایی closure خواهد بود.

این الگو برای ساخت generic factoryها مفید است، زیرا می‌تواند هم lvalue و هم rvalue را با semantics مناسب دریافت کند.

در چنین کدی، باید به copy یا move شدن object و همچنین هزینه ساخت closure توجه کرد.

## init-capture در generic lambda

generic lambda از ++C14 در دسترس است و با `init-capture` ترکیب قدرتمندی ایجاد می‌کند:

```cpp
auto makePrinter = [prefix = std::string("value: ")](auto value) {
    std::cout << prefix << value << '\n';
};
```

در این مثال، `prefix` هنگام ساخت closure تعیین می‌شود، در حالی که نوع `value` هنگام هر invocation مشخص می‌شود.

این ترکیب برای ساخت function objectهای کوچک و type-safe بسیار کاربردی است.

## init-capture برای تبدیل نوع

می‌توان initializer را برای تبدیل نوع نیز به کار برد:

```cpp
double value = 42.5;

auto lambda = [integerValue = static_cast<int>(value)]() {
    return integerValue;
};
```

در این حالت، closure فقط نتیجه تبدیل را نگه می‌دارد.

این روش می‌تواند از capture کردن object اصلی جلوگیری کند و state مورد نیاز callback را دقیقاً مشخص کند.

## init-capture و lifetime

یکی از مهم‌ترین جنبه‌های حرفه‌ای `init-capture`، درک lifetime است.

در مثال زیر:

```cpp
auto makeLambda() {
    std::string text = "hello";

    return [copy = text]() {
        return copy;
    };
}
```

lambda امن است، چون `copy` داخل closure یک object مستقل است.

اما این نسخه مشکل دارد:

```cpp
auto makeLambda() {
    std::string text = "hello";

    return [&ref = text]() {
        return ref;
    };
}
```

در نسخه دوم، `ref` به object محلی اشاره می‌کند که پس از return از بین رفته است.

پس یک قاعده مهم این است:

> `init-capture` به‌خودی‌خود lifetime را ایمن نمی‌کند؛ نوع capture تعیین می‌کند که closure مالک state است یا فقط به state موجود دیگری متکی است.

## init-capture در callbackهای asynchronous

در callbackهای asynchronous، انتخاب capture strategy اهمیت بیشتری پیدا می‌کند، چون ممکن است callback بسیار دیرتر از scope ایجادکننده اجرا شود.

الگوی value-based:

```cpp
void schedule() {
    std::string message = "done";

    enqueue([message = std::move(message)]() {
        process(message);
    });
}
```

در این مثال، callback state مستقل خود را دارد.

الگوی reference-based:

```cpp
void schedule() {
    std::string message = "done";

    enqueue([&message]() {
        process(message);
    });
}
```

اگر `enqueue` callback را بعد از پایان `schedule` اجرا کند، reference می‌تواند dangling شود.

برای callbackهای asynchronous، باید lifetime قرارداد API و زمان اجرای callback دقیقاً مشخص باشد. هیچ capture syntaxای جایگزین طراحی صحیح lifetime نمی‌شود.

## init-capture در ساخت closureهای self-contained

یکی از مزیت‌های مهم این قابلیت، ساخت closureهایی است که وابستگی کمی به scope بیرونی دارند:

```cpp
auto makeMultiplier(int factor) {
    return [factor](int value) {
        return value * factor;
    };
}
```

همین ایده با initializer پیچیده‌تر نیز قابل استفاده است:

```cpp
auto makeMultiplier(int factor) {
    return [factorValue = std::make_shared<int>(factor)](int value) {
        return value * (*factorValue);
    };
}
```

در نسخه دوم، ownership یک `shared_ptr` به closure منتقل شده است.

استفاده از `shared_ptr` باید هدف مشخصی داشته باشد؛ صرفاً برای آسان‌کردن lifetime نباید بدون توجه به ownership model از آن استفاده کرد.

## تفاوت init-capture با `std::bind`

در کدهای قدیمی‌تر ممکن است برای نگه‌داشتن argumentها از `std::bind` استفاده شده باشد:

```cpp
auto handler = std::bind(process, std::move(resource));
```

`init-capture` در بسیاری از سناریوهای callback خواناتر و کنترل‌پذیرتر است:

```cpp
auto handler = [resource = std::move(resource)]() {
    process(resource);
};
```

lambda صریحاً نشان می‌دهد چه stateای ذخیره شده و body آن دقیقاً مشخص می‌کند هنگام invocation چه اتفاقی می‌افتد.

این به معنی منسوخ بودن `std::bind` در سطح language feature نیست؛ انتخاب بین آن‌ها باید بر اساس API، readability و نیاز واقعی انجام شود.

## تفاوت init-capture با local variable

این دو کد از نظر رفتار ممکن است مشابه به نظر برسند:

```cpp
auto value = calculate();

auto lambda = [value]() {
    return value;
};
```

و:

```cpp
auto lambda = [value = calculate()]() {
    return value;
};
```

اما نسخه دوم state مورد نیاز closure را مستقیماً در محل ساخت lambda تعریف می‌کند.

این موضوع می‌تواند scope را کوچک‌تر و intent را واضح‌تر کند، مخصوصاً زمانی که مقدار فقط برای lambda لازم است.

نسخه دوم همچنین مانع باقی‌ماندن یک local variable غیرضروری در scope می‌شود.

## ترتیب ارزیابی initializerها

هر `init-capture` initializer هنگام ساخت closure ارزیابی می‌شود.

برای مثال:

```cpp
int counter = 0;

auto lambda = [
    first = ++counter,
    second = ++counter
]() {
    return first + second;
};
```

در چنین مثال‌هایی نباید صرفاً بر اساس ظاهر کد درباره ترتیب side effectها حدس زد. در طراحی کد بهتر است initializerهای دارای side effect را تا حد امکان ساده و مستقل نگه داشت.

همچنین باید توجه داشت که ترتیب و semantics دقیق initialization closure object بخشی از جزئیات استاندارد language هستند و نباید رفتارهای پیچیده را به assumptions غیرضروری درباره evaluation order وابسته کرد.

برای کد production، اگر ترتیب side effectها مهم است، بهتر است آن را به statements صریح تبدیل کرد:

```cpp
int first = ++counter;
int second = ++counter;

auto lambda = [first, second]() {
    return first + second;
};
```

## capture کردن reference به temporary

یکی از خطاهای خطرناک، ساختن reference به objectی است که lifetime مناسبی ندارد.

برای نمونه:

```cpp
auto lambda = [&ref = std::string("temporary")]() {
    return ref;
};
```

چنین الگوهایی نباید برای ایجاد referenceهای بلندمدت استفاده شوند. اگر closure باید object را نگه دارد، معمولاً باید object را به‌صورت value در closure ذخیره کرد:

```cpp
auto lambda = [text = std::string("temporary")]() {
    return text;
};
```

در نسخه دوم، object بخشی از state closure است و lifetime آن با closure مدیریت می‌شود.

## capture کردن `std::move` و استفاده مجدد از متغیر

پس از move کردن یک object به closure، نباید انتظار داشت object بیرونی همچنان همان state قبلی را داشته باشد:

```cpp
std::unique_ptr<int> ptr = std::make_unique<int>(42);

auto lambda = [ptr = std::move(ptr)]() {
    return *ptr;
};
```

بعد از این عملیات، `ptr` بیرونی در وضعیت moved-from قرار دارد.

برای `std::unique_ptr` معمولاً مقدار آن null می‌شود، اما قاعده عمومی برای همه typeها این نیست که object به یک مقدار خاص برگردد. قرارداد move constructor یا move assignment type مربوطه تعیین‌کننده است.

بنابراین اگر بعد از move به object اصلی نیاز دارید، باید طراحی را تغییر دهید.

## capture کردن objectهای بزرگ

`init-capture` می‌تواند objectهای بزرگ را در closure ذخیره کند:

```cpp
std::vector<int> values = createValues();

auto callback = [values = std::move(values)]() {
    process(values);
};
```

این کار از نظر ownership ممکن است عالی باشد، اما اندازه closure را نیز افزایش می‌دهد.

اگر closure در تعداد زیادی object یا container نگهداری شود، اندازه state و هزینه copy/move closure باید بررسی شود.

گاهی یک `std::shared_ptr<const T>` یا abstraction دیگر مناسب‌تر است، اما استفاده از pointer indirection باید بر اساس ownership و performance requirements تصمیم‌گیری شود، نه صرفاً برای کوچک‌کردن object.

## closure object و semantics کپی و move

lambda expression یک closure object تولید می‌کند و captureهای آن بخشی از state closure هستند.

اگر closure شامل objectهای move-only باشد، قابلیت copy شدن closure نیز تحت تأثیر قرار می‌گیرد.

برای نمونه:

```cpp
auto ptr = std::make_unique<int>(42);

auto lambda = [ptr = std::move(ptr)]() {
    return *ptr;
};
```

این closure دارای stateای از نوع move-only است و بنابراین نمی‌توان آن را مانند یک closure کاملاً copyable استفاده کرد.

این مسئله هنگام ارسال lambda به APIها بسیار مهم است. برخی APIها callback را copy می‌کنند و برخی فقط move می‌کنند.

اگر API نیازمند copyable callable باشد، یک lambda دارای `std::unique_ptr` ممکن است با آن سازگار نباشد.

## بررسی copyability در طراحی API

فرض کنید API زیر callback را چند بار کپی می‌کند:

```cpp
template <typename Callback>
void registerCallback(Callback callback) {
    auto another = callback;
    // ...
}
```

اگر callback دارای state move-only باشد، این کد مشکل خواهد داشت.

در مقابل، اگر API با move-only callable طراحی شده باشد:

```cpp
template <typename Callback>
void registerCallback(Callback&& callback) {
    // Store or forward the callback according to the API contract.
}
```

ممکن است بتواند lambda دارای move-only state را پشتیبانی کند.

بنابراین انتخاب `init-capture` فقط یک تصمیم محلی داخل lambda نیست؛ باید قرارداد callable مصرف‌کننده نیز بررسی شود.

## init-capture و `std::function`

در ++C14، `std::function` به‌طور کلی برای targetهایی طراحی شده که CopyConstructible باشند. در نتیجه، یک lambda که `std::unique_ptr` را با move-capture نگه می‌دارد، نمی‌تواند مستقیماً به‌عنوان target معمولی `std::function` استفاده شود.

برای مثال:

```cpp
auto ptr = std::make_unique<int>(42);

auto lambda = [ptr = std::move(ptr)]() {
    return *ptr;
};

// std::function<int()> fn = std::move(lambda); // Not valid in C++14.
```

در استانداردهای جدیدتر نیز باید به semantics type-erasure و محدودیت‌های نوع wrapper مورد استفاده توجه کرد. اگر callable باید move-only باشد، abstraction انتخاب‌شده باید واقعاً move-only callable را پشتیبانی کند.

این موضوع یکی از دلایل مهم بررسی API مقصد پیش از انتخاب move-capture است.

## init-capture و recursive lambda

یک کاربرد پیشرفته، ساخت closureهایی است که به خودشان دسترسی دارند. در ++C14، یکی از روش‌ها استفاده از `std::function` است:

```cpp
#include <functional>

std::function<int(int)> factorial;

factorial = [&factorial](int n) {
    if (n <= 1) {
        return 1;
    }

    return n * factorial(n - 1);
};
```

این روش کار می‌کند، اما overhead و type-erasure مربوط به `std::function` را به همراه دارد.

در استانداردهای جدیدتر، می‌توان الگوهای recursive generic lambda را با self-parameter پیاده‌سازی کرد و در بسیاری از موارد از `std::function` اجتناب کرد:

```cpp
auto factorial = [](auto&& self, int n) -> int {
    if (n <= 1) {
        return 1;
    }

    return n * self(self, n - 1);
};
```

در نتیجه، `init-capture` ابزار مهمی است، اما تنها ابزار مربوط به stateful یا recursive lambda نیست.

## init-capture و capture کردن parameterهای function

گاهی یک function parameter باید به callback منتقل شود:

```cpp
void submit(std::unique_ptr<Task> task) {
    enqueue([task = std::move(task)]() {
        task->run();
    });
}
```

این یکی از طبیعی‌ترین کاربردهای `init-capture` در APIهای asynchronous است.

در چنین طراحی‌هایی، ownership انتقال‌یافته باید مستند باشد. caller باید بداند که پس از فراخوانی `submit` دیگر مالک resource نیست.

## init-capture و جلوگیری از dangling reference

الگوی زیر معمولاً امن‌تر از capture کردن reference یک local variable است:

```cpp
std::string makeMessage();

auto makeCallback() {
    return [message = makeMessage()]() {
        process(message);
    };
}
```

در اینجا callback object را با خود حمل می‌کند.

اما اگر object بسیار بزرگ باشد یا ownership باید با چند consumer به اشتراک گذاشته شود، راهکار دیگری ممکن است مناسب‌تر باشد.

نکته اصلی این است که lifetime باید از روی semantics مورد نیاز طراحی شود، نه صرفاً از روی کوتاه‌ترین syntax.

## تفاوت `init-capture` با `std::move` به‌تنهایی

`std::move` خودش چیزی را move نمی‌کند؛ فقط expression را به شکل مناسب برای move کردن در اختیار عملیات بعدی قرار می‌دهد.

در این کد:

```cpp
auto lambda = [ptr = std::move(ptr)]() {
};
```

عمل واقعی move توسط initialization مربوط به state `ptr` انجام می‌شود.

پس بهتر است مفهوم را این‌گونه در ذهن نگه دارید:

* `std::move` یک cast به xvalue است.
* `init-capture` state closure را می‌سازد.
* move constructor یا move assignment نوع مربوطه عملیات انتقال resource/state را انجام می‌دهد.

این تفکیک برای درک دقیق semantics بسیار مهم است.

## ارتباط init-capture با value categoryها

در expression زیر:

```cpp
auto lambda = [value = std::move(object)]() {
};
```

عبارت `std::move(object)` یک xvalue تولید می‌کند. سپس state داخل closure با initializer مربوطه ساخته می‌شود.

اگر initializer یک lvalue باشد:

```cpp
auto lambda = [value = object]() {
};
```

state معمولاً با copy initialization از object ساخته می‌شود.

اگر initializer یک prvalue باشد:

```cpp
auto lambda = [value = createObject()]() {
};
```

object نتیجه expression برای ساخت state استفاده می‌شود و compiler می‌تواند طبق قواعد language، temporary materialization و copy elision مناسب را اعمال کند.

در نتیجه، `init-capture` به‌خوبی با move semantics و value categories زبان ++C یکپارچه شده است.

## init-capture و `decltype`

نام captureشده در body lambda مانند یک نام قابل استفاده در expressionهاست:

```cpp
auto lambda = [value = 42]() {
    using T = decltype(value);
    return value;
};
```

اما باید دقت کرد که semantics `decltype` در مورد نام‌های expression و اعضای object را با قواعد معمول `decltype` تحلیل کنیم. همچنین نوع دقیق closure member بخشی از implementation detail قابل اتکا برای نام‌گذاری نیست.

در طراحی عمومی بهتر است به behavior و interface closure تکیه شود، نه به assumptions درباره layout داخلی closure.

## نام capture و shadowing

`init-capture` می‌تواند نامی انتخاب کند که با نام دیگری در scopeهای اطراف یکسان باشد:

```cpp
int value = 10;

auto lambda = [value = value + 1]() {
    return value;
};
```

در این مثال، سمت راست initializer به `value` بیرونی اشاره می‌کند و نام سمت چپ state جدید را معرفی می‌کند.

هرچند این syntax قانونی و رایج است، در expressionهای پیچیده می‌تواند خوانایی را کاهش دهد. استفاده از نام‌های متفاوت در captureهای پیچیده معمولاً debugging و code review را ساده‌تر می‌کند:

```cpp
int value = 10;

auto lambda = [capturedValue = value + 1]() {
    return capturedValue;
};
```

## خطاهای رایج در استفاده از init-capture

یکی از خطاهای رایج، تصور این است که هر `init-capture` باعث ownership امن می‌شود. چنین تصوری درباره reference init-capture نادرست است.

خطای دیگر، فراموش‌کردن `mutable` هنگام تغییر state داخلی است:

```cpp
auto counter = [count = 0]() {
    ++count; // Error: operator() is const by default.
    return count;
};
```

نسخه درست:

```cpp
auto counter = [count = 0]() mutable {
    ++count;
    return count;
};
```

اشتباه دیگر، move کردن object و سپس استفاده از نسخه بیرونی بدون درنظرگرفتن moved-from state است:

```cpp
auto ptr = std::make_unique<int>(42);

auto callback = [ptr = std::move(ptr)]() {
    return *ptr;
};

// Do not assume ptr still owns the object here.
```

همچنین نباید تصور کرد که lambda دارای move-only state با هر API callable سازگار است.

## اشتباهات مربوط به lifetime

الگوی زیر خطرناک است:

```cpp
auto makeCallback() {
    int value = 42;

    return [&captured = value]() {
        return captured;
    };
}
```

مشکل، `init-capture` نیست؛ مشکل این است که state closure یک reference به local object نگه داشته است.

نسخه value-based:

```cpp
auto makeCallback() {
    int value = 42;

    return [captured = value]() {
        return captured;
    };
}
```

در این نسخه closure object مستقل خود را دارد.

برای callbackهای asynchronous، باید همین تحلیل را برای تمام objectهای captureشده انجام داد.

## اشتباهات مربوط به mutable state

lambda زیر state خود را تغییر می‌دهد:

```cpp
auto counter = [count = 0]() mutable {
    return ++count;
};
```

اما اگر چند copy از `counter` ساخته شود، هر copy state مستقل خودش را خواهد داشت:

```cpp
auto counter1 = counter;
auto counter2 = counter;
```

در اینجا تغییر `counter1` روی state `counter2` اثر نمی‌گذارد.

این رفتار برای طراحی function objectهای stateful مهم است. اگر state باید بین چند consumer مشترک باشد، باید یک shared state صریح طراحی شود، مثلاً با یک object مدیریت‌شده توسط `std::shared_ptr` در صورت مناسب بودن ownership model.

## performance و اندازه closure

هر `init-capture` معمولاً state را به closure object اضافه می‌کند. بنابراین این کد:

```cpp
auto callback = [
    largeObject = createLargeObject(),
    cache = createLargeCache()
]() {
    use(largeObject, cache);
};
```

ممکن است closure بزرگی تولید کند.

اگر چنین callbackهایی زیاد copy یا move شوند، هزینه مربوط به size و move/copy state می‌تواند مهم باشد.

راهکار مناسب به مسئله بستگی دارد و ممکن است شامل موارد زیر باشد:

* نگه‌داشتن فقط state ضروری
* move کردن object به‌جای copy
* استفاده از یک ownership abstraction مناسب
* جلوگیری از copyهای غیرضروری closure
* استفاده از reference در صورتی که lifetime و thread safety واقعاً تضمین شده باشد

نباید صرفاً برای performance از reference capture استفاده کرد؛ dangling reference معمولاً هزینه بسیار بیشتری از یک copy کنترل‌شده دارد.

## exception safety در initializer

initializerهای `init-capture` هنگام ساخت closure اجرا می‌شوند و می‌توانند exception پرتاب کنند:

```cpp
auto callback = [
    resource = createResource()
]() {
    use(resource);
};
```

اگر `createResource()` exception پرتاب کند، closure ساخته نمی‌شود.

در طراحی resource acquisition باید RAII و exception safety مربوط به type استفاده‌شده بررسی شود.

اگر چند initializer وجود داشته باشد، پیچیده‌کردن آن‌ها با side effectهای وابسته به یکدیگر می‌تواند reasoning درباره failure و cleanup را دشوار کند. در چنین شرایطی، ساخت state در یک helper function یا object مستقل ممکن است طراحی شفاف‌تری ایجاد کند.

## init-capture و استانداردهای بعدی

قابلیت پایه `init-capture` در ++C14 معرفی شد و همچنان در استانداردهای جدید معتبر است.

در ++C17 قابلیت value capture کردن `*this` با syntax مستقیم `[ *this ]` اضافه شد. این قابلیت برای سناریوهایی که lambda باید یک copy از object فعلی داشته باشد، syntax مستقیمی فراهم می‌کند.

در ++C20، قابلیت‌های lambda و templateها توسعه بیشتری پیدا کردند و امکاناتی مانند pack expansion در captureها برای سناریوهای variadic در دسترس قرار گرفت.

با وجود این پیشرفت‌ها، syntax پایه `identifier = initializer` همچنان یکی از ابزارهای اصلی برای ساخت closureهای stateful است.

## init-capture و packها در ++C20

در کدهای variadic مدرن، init-capture می‌تواند با pack expansion ترکیب شود. این قابلیت برای انتقال مجموعه‌ای از objectها به closure مفید است.

یک الگوی مفهومی در ++C20 می‌تواند به شکل زیر باشد:

```cpp
template <typename... Args>
auto makeCallback(Args&&... args) {
    return [...values = std::forward<Args>(args)]() {
        // Use the captured pack.
    };
}
```

این قابلیت در طراحی utilityهای generic پیشرفته مفید است، زیرا می‌توان مجموعه‌ای از argumentها را به state closure تبدیل کرد.

با این حال، کدهای variadic دارای ownership باید با دقت بیشتری از نظر copy/move semantics و lifetime بررسی شوند.

## مقایسه با captureهای معمولی

برای درک تفاوت‌ها، می‌توان این الگوها را کنار هم دید:

```cpp
int value = 42;

auto byValue = [value]() {
    return value;
};

auto byReference = [&value]() {
    return value;
};

auto renamed = [capturedValue = value]() {
    return capturedValue;
};

auto computed = [result = value * 2]() {
    return result;
};
```

در `byValue`، value capture معمولی استفاده شده است.

در `byReference`، closure به object بیرونی reference دارد.

در `renamed`، یک state جدید با نام متفاوت ساخته شده است.

در `computed`، state جدید مستقیماً از یک expression محاسبه می‌شود.

و برای move-only state:

```cpp
auto ptr = std::make_unique<int>(42);

auto moved = [ptr = std::move(ptr)]() {
    return *ptr;
};
```

این آخرین فرم یکی از مهم‌ترین کاربردهای عملی `init-capture` است.

## راهنمای انتخاب نوع capture

هنگام طراحی lambda، ابتدا باید ownership و lifetime را مشخص کرد.

اگر فقط یک copy مستقل از مقدار لازم است، capture معمولی value-based یا `init-capture` مناسب است:

```cpp
auto lambda = [value]() {
    return value;
};
```

اگر نام داخلی یا initializer متفاوت لازم است، `init-capture` مناسب‌تر است:

```cpp
auto lambda = [normalized = normalize(value)]() {
    return normalized;
};
```

اگر object باید منتقل شود، move-capture استفاده می‌شود:

```cpp
auto lambda = [resource = std::move(resource)]() {
    use(resource);
};
```

اگر باید همان object بیرونی مشاهده یا تغییر داده شود، reference capture ممکن است مناسب باشد:

```cpp
auto lambda = [&value]() {
    ++value;
};
```

اما در حالت reference، lifetime و synchronization باید به‌صورت صریح بررسی شوند.

## راهنمای انتخاب بین copy، move و reference

سه سؤال عملی معمولاً تصمیم را روشن می‌کنند:

* آیا lambda باید مالک یا دارنده مستقل state باشد؟
* آیا object قابل copy است یا فقط قابل move است؟
* آیا object بیرونی تضمین می‌کند که تا پایان lifetime lambda معتبر بماند؟

اگر object باید همراه callback زنده بماند، value یا move-based capture معمولاً semantics روشن‌تری دارد.

اگر object بزرگ و shared است، یک ownership abstraction مانند `std::shared_ptr` ممکن است مناسب باشد، ولی فقط در صورتی که shared ownership واقعاً بخشی از طراحی باشد.

اگر object فقط موقتاً در اختیار lambda قرار می‌گیرد و lifetime کاملاً کنترل‌شده است، reference capture می‌تواند مناسب باشد.

## نکات مهم برای code review

هنگام review یک lambda دارای `init-capture`، فقط syntax را بررسی نکنید. این موارد نیز باید بررسی شوند:

* نوع state داخلی چیست؟
* آیا initializer باعث copy یا move می‌شود؟
* object بیرونی بعد از move چه وضعیتی دارد؟
* آیا closure copyable است؟
* آیا API مقصد callback را copy می‌کند؟
* lifetime state تا چه زمانی تضمین شده است؟
* آیا reference capture می‌تواند dangling شود؟
* آیا closure در چند thread اجرا می‌شود؟
* آیا mutable state نیازمند synchronization است؟
* آیا اندازه closure بیش از حد بزرگ شده است؟
* آیا initializer دارای side effect یا exception behavior مهمی است؟
* آیا capture بیش از نیاز واقعی state نگه می‌دارد؟

این بررسی‌ها معمولاً از خود syntax مهم‌تر هستند.

## الگوی پیشنهادی برای callbackهای مالک resource

یک الگوی رایج و قابل فهم در ++C14 این است:

```cpp
void submit(std::unique_ptr<Job> job) {
    enqueue([job = std::move(job)]() mutable {
        job->run();
    });
}
```

در اینجا ownership به callback منتقل شده است.

اگر `enqueue` callback را move یا storage مناسب برای move-only callable پشتیبانی نکند، این طراحی ممکن است با API ناسازگار باشد. بنابراین implementation `enqueue` و قرارداد lifetime آن باید هم‌زمان بررسی شوند.

## الگوی پیشنهادی برای state محاسبه‌شده

اگر state فقط برای callback لازم است، می‌توان آن را مستقیماً در capture list ساخت:

```cpp
auto callback = [
    endpoint = normalizeEndpoint(config.endpoint),
    timeout = config.timeout
]() {
    connect(endpoint, timeout);
};
```

این سبک معمولاً از ایجاد local variableهای اضافی جلوگیری می‌کند و وابستگی‌های callback را در یک مکان نشان می‌دهد.

## الگوی پیشنهادی برای نگه‌داشتن copy از object

در ++C14 می‌توان از `init-capture` برای ساخت copy از object فعلی استفاده کرد:

```cpp
class Session {
public:
    int id = 42;

    auto makeCallback() const {
        return [session = *this]() {
            return session.id;
        };
    }
};
```

این الگو زمانی مناسب است که semantics کپی object مطلوب باشد و object شامل resourceهای غیرقابل‌کپی نباشد.

در ++C17، `[ *this ]` می‌تواند همین intent را مستقیم‌تر بیان کند:

```cpp
class Session {
public:
    int id = 42;

    auto makeCallback() const {
        return [*this]() {
            return id;
        };
    }
};
```

## محدودیت‌های اصلی

`init-capture` با وجود کاربرد گسترده، چند محدودیت مهم دارد.

نخست، syntax آن lifetime را به‌صورت خودکار حل نمی‌کند. reference init-capture همچنان می‌تواند dangling شود.

دوم، capture کردن objectهای بزرگ می‌تواند closure را بزرگ کند.

سوم، move-capture می‌تواند closure را move-only کند و در نتیجه با APIهایی که callable را copy می‌کنند ناسازگار شود.

چهارم، capture کردن `*this` با copy می‌تواند هزینه کپی object را ایجاد کند و فقط برای classهای مناسب از نظر copy semantics قابل استفاده است.

پنجم، `init-capture` جایگزین طراحی صحیح ownership، synchronization یا exception safety نیست.

## جمع‌بندی

`init-capture` یکی از قابلیت‌های کلیدی lambdaها در ++C14 است که امکان ساخت state داخلی closure را با یک initializer دلخواه فراهم می‌کند.

الگوی اصلی آن به‌صورت زیر است:

```cpp
auto lambda = [name = initializer]() {
    // Use name here.
};
```

مهم‌ترین کاربردهای آن عبارت‌اند از:

* انتقال ownership با move-capture
* نگه‌داشتن state محاسبه‌شده
* تغییر نام state داخلی
* ساخت closureهای self-contained
* مدیریت دقیق‌تر lifetime callbackها
* نگه‌داشتن objectهای move-only
* ساخت stateful lambdaها
* ترکیب با generic lambdaها و template code

در ++C14، syntaxهایی مانند `[ptr = std::move(ptr)]` راهی استاندارد و خوانا برای انتقال resource به closure فراهم کردند.

مهم‌ترین نکته حرفه‌ای این است که capture syntax را از ownership و lifetime جدا نکنیم. هنگام استفاده از `init-capture` باید مشخص باشد که closure یک copy مستقل می‌گیرد، object را move می‌کند، یا فقط یک reference نگه می‌دارد. همچنین باید copyability closure و قرارداد API مصرف‌کننده callback بررسی شود.

در عمل، `init-capture` زمانی بیشترین ارزش را دارد که state مورد نیاز lambda را صریح، محدود، دارای ownership مشخص و متناسب با lifetime واقعی callback تعریف کند.


---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>