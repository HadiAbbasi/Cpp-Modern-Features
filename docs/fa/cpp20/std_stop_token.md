<div align="center">

[🇺🇸 English](../../en/cpp20/std_stop_token.md) | [🇮🇷 فارسی](./std_stop_token.md)

</div>

---

# راهنمای جامع std::stop_token در ++C20

## مقدمه

مدیریت چرخه عمر Threadها و کنترل نحوه پایان یافتن عملیات هم‌زمان، یکی از چالش‌های مهم در برنامه‌نویسی Concurrent در ++C است. در بسیاری از برنامه‌ها، یک Thread ممکن است وظیفه‌ای طولانی‌مدت مانند پردازش داده، دریافت اطلاعات از شبکه، اجرای یک حلقه محاسباتی یا مدیریت یک سرویس پس‌زمینه را بر عهده داشته باشد.

در چنین شرایطی، ممکن است بخش دیگری از برنامه تصمیم بگیرد که عملیات باید متوقف شود. مسئله این است که چگونه می‌توان این درخواست را به Thread اجراکننده منتقل کرد، بدون آنکه اجرای آن به‌صورت ناگهانی و در وضعیت نامناسب قطع شود.

استاندارد ++C20 با معرفی خانواده Stop Token یک راهکار استاندارد برای این مسئله ارائه کرده است. این مکانیزم بر اساس مفهوم Cooperative Cancellation طراحی شده است؛ یعنی یک بخش از برنامه درخواست توقف را ارسال می‌کند و بخش اجراکننده در زمان مناسب به آن واکنش نشان می‌دهد.

خانواده Stop Token از سه کلاس اصلی تشکیل شده است:

- `std::stop_source`: مسئول ارسال درخواست توقف.
- `std::stop_token`: مسئول مشاهده وضعیت درخواست توقف.
- `std::stop_callback`: مسئول اجرای Callback هنگام ثبت درخواست توقف.

این قابلیت به‌خصوص در کنار `std::jthread` و APIهای قابل لغو، نقش مهمی در طراحی برنامه‌های Concurrent مدرن دارد.

## مفهوم Cooperative Cancellation

### تعریف لغو مشارکتی

مفهوم Cooperative Cancellation به روشی گفته می‌شود که در آن درخواست توقف از یک بخش برنامه به بخش دیگری ارسال می‌شود، اما تصمیم‌گیری درباره زمان و نحوه پایان عملیات بر عهده بخش اجراکننده باقی می‌ماند.

برای مثال، فرض کنید یک Thread مسئول پردازش مجموعه‌ای از فایل‌ها است. Thread دیگری می‌تواند درخواست توقف ارسال کند، اما Thread پردازش‌کننده باید در نقاط مشخصی از اجرای خود این درخواست را بررسی کند.

پس از مشاهده درخواست، Thread می‌تواند:

- پردازش فعلی را تکمیل کند.
- منابع در اختیار خود را آزاد کند.
- وضعیت داخلی را به حالت معتبر بازگرداند.
- از حلقه پردازش خارج شود.
- نتیجه لغو عملیات را به فراخواننده گزارش دهد.

این رویکرد در مقایسه با توقف اجباری، ایمن‌تر و قابل‌کنترل‌تر است.

### تفاوت توقف مشارکتی و توقف اجباری

در توقف اجباری، یک Thread دیگر تلاش می‌کند اجرای Thread هدف را بدون همکاری آن متوقف کند. چنین رویکردی ممکن است باعث باقی ماندن منابع در وضعیت نامعتبر شود.

برای مثال، ممکن است Thread هدف هنگام توقف:

- یک Mutex را در اختیار داشته باشد؛
- در حال تغییر یک ساختار داده مشترک باشد؛
- یک تراکنش را هنوز Commit نکرده باشد؛
- در حال نوشتن اطلاعات در فایل باشد؛
- در حال مدیریت یک Socket یا منبع خارجی باشد.

در مقابل، Cooperative Cancellation به Thread اجازه می‌دهد در یک نقطه امن اجرای خود را متوقف کند.

نکته مهم: مکانیزم Stop Token در ++C20 یک ابزار Cooperative Cancellation است و قابلیت Terminate کردن اجباری Thread را ارائه نمی‌کند.

## معرفی خانواده Stop Token

### معرفی std::stop_source

کلاس `std::stop_source` مسئول مدیریت و ارسال درخواست توقف است.

این کلاس امکان ایجاد یک Stop State و دریافت Token مرتبط با آن را فراهم می‌کند.

مهم‌ترین توابع آن عبارت‌اند از:

- `get_token()`: دریافت Token مرتبط با Stop State.
- `request_stop()`: ثبت درخواست توقف.
- `stop_requested()`: بررسی وضعیت درخواست توقف.
- `stop_possible()`: بررسی امکان درخواست توقف.

نمونه:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token = source.get_token();

    std::cout << std::boolalpha;
    std::cout << token.stop_requested() << '\n';

    source.request_stop();

    std::cout << token.stop_requested() << '\n';
}
```

خروجی:

```
false
true
```

در این مثال، درخواست توقف از طریق `source` ارسال می‌شود، اما وضعیت آن از طریق `token` قابل مشاهده است.

### معرفی std::stop_token

کلاس `std::stop_token` یک Handle قابل‌کپی برای مشاهده وضعیت درخواست توقف است.

این کلاس به‌صورت مستقیم درخواست توقف ارسال نمی‌کند و مسئولیت مدیریت چرخه عمر Thread را نیز بر عهده ندارد.

ویژگی‌های اصلی آن عبارت‌اند از:

- امکان کپی و انتقال دارد.
- می‌تواند در اختیار چندین Thread قرار بگیرد.
- وضعیت Stop State مرتبط را مشاهده می‌کند.
- برای انتقال قابلیت Cancellation به توابع مستقل از Thread مناسب است.
- پس از ثبت درخواست توقف، تغییر وضعیت آن به حالت اولیه امکان‌پذیر نیست.

### معرفی std::stop_callback

کلاس `std::stop_callback` برای ثبت یک Callable استفاده می‌شود که هنگام ثبت درخواست توقف اجرا خواهد شد.

این قابلیت به‌خصوص برای بیدار کردن عملیات منتظرمانده یا ارسال Notification به بخش‌های دیگر برنامه کاربرد دارد.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    auto token = source.get_token();

    std::stop_callback callback(
        token,
        [] {
            std::cout << "Stop requested\n";
        }
    );

    source.request_stop();
}
```

خروجی:

```
Stop requested
```

## ساختار داخلی و مفهوم Stop State

### ارتباط میان کلاس‌های Stop Token

در پشت صحنه، کلاس‌های `std::stop_source`، `std::stop_token` و `std::stop_callback` با یک وضعیت مشترک به نام Stop State کار می‌کنند.

این وضعیت اطلاعات مربوط به درخواست توقف و Callbackهای ثبت‌شده را نگهداری می‌کند.

مدل مفهومی ارتباط این اشیا به شکل زیر است:

```
             std::stop_source
                    |
                    |
                Stop State
                /        \
               /          \
      std::stop_token   std::stop_callback
               |
               |
          Worker Thread
```

هر Token دریافت‌شده از یک Stop Source به همان Stop State متصل است.

بنابراین، کپی کردن Token باعث ایجاد یک وضعیت توقف مستقل نمی‌شود؛ تمام کپی‌ها وضعیت مشترکی را مشاهده می‌کنند.

### مالکیت و طول عمر Stop State

Stop State دارای مدیریت طول عمر اشتراکی است.

تا زمانی که یک `std::stop_source` یا `std::stop_token` مرتبط زنده باشد، Stop State می‌تواند زنده بماند.

بنابراین، از بین رفتن شیء `std::stop_source` لزوماً به معنای از بین رفتن Stop State نیست.

برای مثال:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_token token;

    {
        std::stop_source source;

        token = source.get_token();
        source.request_stop();
    }

    std::cout << std::boolalpha;
    std::cout << token.stop_possible() << '\n';
    std::cout << token.stop_requested() << '\n';
}
```

خروجی:

```
true
true
```

در این مثال، پس از Destroy شدن `source`، Token همچنان به Stop State دسترسی دارد و درخواست توقف ثبت‌شده را مشاهده می‌کند.

نکته: طول عمر Stop State با طول عمر اشیای خارجی مورد استفاده در Callback یکسان نیست. اگر Callback به یک شیء خارجی ارجاع داشته باشد، باید اعتبار آن ارجاع به‌صورت مستقل تضمین شود.

## نحوه ایجاد و دریافت Stop Token

### دریافت Token از Stop Source

متداول‌ترین روش دریافت Token، استفاده از تابع `get_token()` است.

```
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token = source.get_token();
}
```

تابع `get_token()` یک Token مرتبط با Stop State همان Source برمی‌گرداند.

می‌توان این Token را به تابع‌های مختلف ارسال کرد تا آن‌ها نیز قابلیت بررسی درخواست توقف را داشته باشند.

### دریافت Token از std::jthread

یکی از مهم‌ترین کاربردهای `std::stop_token` در کنار `std::jthread` است.

کلاس `std::jthread` در ++C20 معرفی شده و امکانات مناسبی برای مدیریت چرخه عمر Thread ارائه می‌دهد.

اگر Callable مربوط به `std::jthread` قابلیت دریافت `std::stop_token` به‌عنوان اولین آرگومان را داشته باشد، این Token به‌صورت خودکار به آن ارسال می‌شود.

```
#include <iostream>
#include <stop_token>
#include <thread>
#include <chrono>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Working...\n";

        std::this_thread::sleep_for(
            std::chrono::milliseconds(200)
        );
    }

    std::cout << "Worker stopped\n";
}

int main() {
    std::jthread thread(worker);

    std::this_thread::sleep_for(
        std::chrono::seconds(1)
    );

    thread.request_stop();
}
```

در این مثال:

- یک Thread با استفاده از `std::jthread` ایجاد می‌شود.
- Token مربوط به Thread به تابع `worker` ارسال می‌شود.
- Worker به‌صورت دوره‌ای وضعیت توقف را بررسی می‌کند.
- با فراخوانی `request_stop()` درخواست توقف ثبت می‌شود.
- Worker در بررسی بعدی از حلقه خارج می‌شود.
- در پایان، Destructor مربوط به `std::jthread` در صورت Join نشدن Thread، درخواست توقف ارسال می‌کند و سپس منتظر پایان آن می‌ماند.

نکته: ارسال درخواست توقف و Join کردن دو عملیات متفاوت هستند. تابع `request_stop()` به‌تنهایی منتظر پایان Thread نمی‌ماند.

### ارسال آرگومان‌های اضافی به Callable

در `std::jthread`، Token به‌عنوان اولین آرگومان Callable قرار می‌گیرد و آرگومان‌های دیگر پس از آن ارسال می‌شوند.

```
#include <iostream>
#include <stop_token>
#include <thread>
#include <chrono>

int main() {
    std::jthread worker(
        [](std::stop_token token, int value) {
            while (!token.stop_requested()) {
                std::cout << value << '\n';

                std::this_thread::yield();
            }
        },
        42
    );

    std::this_thread::sleep_for(
        std::chrono::milliseconds(100)
    );

    worker.request_stop();
}
```

در این مثال، مقدار `42` به‌عنوان آرگومان دوم به Lambda ارسال می‌شود.

### ایجاد Stop Source بدون Stop State

در برخی شرایط ممکن است بخواهیم یک `std::stop_source` داشته باشیم که اصلاً Stop State ایجاد نکند.

برای این منظور می‌توان از `std::nostopstate` استفاده کرد.

```
#include <stop_token>
#include <iostream>

int main() {
    std::stop_source source(std::nostopstate);

    std::stop_token token = source.get_token();

    std::cout << std::boolalpha;
    std::cout << token.stop_possible() << '\n';
    std::cout << token.stop_requested() << '\n';
}
```

خروجی:

```
false
false
```

این قابلیت برای APIهایی مفید است که Token اختیاری دارند و در برخی شرایط نیازی به ایجاد Stop State نیست.

## بررسی توابع و قابلیت‌های std::stop_token

### بررسی stop_requested

تابع `stop_requested()` مشخص می‌کند که آیا درخواست توقف برای Stop State مرتبط ثبت شده است یا خیر.

امضای تابع:

```
bool stop_requested() const noexcept;
```

این تابع در صورت ثبت درخواست توقف مقدار `true` و در غیر این صورت مقدار `false` برمی‌گرداند.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    auto token = source.get_token();

    std::cout << std::boolalpha;

    std::cout << token.stop_requested() << '\n';

    source.request_stop();

    std::cout << token.stop_requested() << '\n';
}
```

خروجی:

```
false
true
```

این تابع Thread-safe است و می‌توان آن را از چندین Thread به‌صورت هم‌زمان فراخوانی کرد.

نکته مهم: مقدار `false` فقط وضعیت Token را در زمان فراخوانی نشان می‌دهد. ممکن است درخواست توقف بلافاصله پس از بازگشت تابع ثبت شود.

بنابراین، نباید از این تابع به‌عنوان جایگزین Lock یا مکانیزم Synchronization استفاده کرد.

### بررسی stop_possible

تابع `stop_possible()` مشخص می‌کند که آیا Token اصولاً امکان دریافت درخواست توقف دارد یا خیر.

امضای تابع:

```
bool stop_possible() const noexcept;
```

این تابع در شرایط زیر می‌تواند مقدار `false` برگرداند:

- Token خالی باشد.
- Token به Stop Stateای مرتبط باشد که هیچ Stop Source فعالی ندارد و درخواست توقف نیز قبلاً ثبت نشده است.
- Token از Stop Sourceای بدون Stop State دریافت شده باشد.

مثال:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_token emptyToken;

    std::stop_source source;

    auto token = source.get_token();

    std::cout << std::boolalpha;

    std::cout << emptyToken.stop_possible() << '\n';
    std::cout << token.stop_possible() << '\n';
}
```

خروجی:

```
false
true
```

تفاوت این تابع با `stop_requested()` بسیار مهم است.

| وضعیت                        | stop_possible | stop_requested |
| ---------------------------- | ------------- | -------------- |
| Token خالی                   | false         | false          |
| Token فعال بدون درخواست توقف | true          | false          |
| Token فعال با درخواست توقف   | true          | true           |
| Token بدون Stop State        | false         | false          |

پس از ثبت درخواست توقف، Tokenهای باقی‌مانده همچنان می‌توانند وضعیت درخواست را مشاهده کنند؛ حتی اگر دیگر Stop Source فعالی وجود نداشته باشد.

### مقایسه Tokenها با operator==

کلاس `std::stop_token` از عملگر مقایسه برابری پشتیبانی می‌کند.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    auto token1 = source.get_token();
    auto token2 = source.get_token();

    std::stop_token emptyToken;

    std::cout << std::boolalpha;

    std::cout << (token1 == token2) << '\n';
    std::cout << (token1 == emptyToken) << '\n';
}
```

خروجی:

```
true
false
```

دو Token زمانی برابر هستند که به یک Stop State متصل باشند یا هر دو خالی باشند.

این مقایسه وضعیت درخواست توقف را بررسی نمی‌کند، بلکه ارتباط Tokenها با Stop State را مقایسه می‌کند.

### کپی و انتقال Token

کلاس `std::stop_token` از Copy Construction، Copy Assignment، Move Construction و Move Assignment پشتیبانی می‌کند.

```
#include <stop_token>
#include <utility>

int main() {
    std::stop_source source;

    std::stop_token token1 = source.get_token();

    std::stop_token token2 = token1;

    std::stop_token token3 = std::move(token2);
}
```

کپی کردن Token باعث ایجاد یک Handle جدید به همان Stop State می‌شود.

در عملیات Move، ارتباط Token به شیء مقصد منتقل می‌شود. شیء مبدأ پس از Move همچنان معتبر است، اما مقدار آن مشخص نیست و نباید درباره وضعیت ارتباط آن با Stop State فرضی داشت.

## ارسال درخواست توقف با std::stop_source

### استفاده از request_stop

تابع `request_stop()` اصلی‌ترین روش ارسال درخواست توقف است.

امضای تابع:

```
bool request_stop() noexcept;
```

این تابع تلاش می‌کند درخواست توقف را برای Stop State مرتبط ثبت کند.

اگر درخواست توقف برای اولین بار ثبت شود، مقدار `true` برمی‌گرداند.

اگر درخواست توقف قبلاً ثبت شده باشد، مقدار `false` برمی‌گرداند.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::cout << std::boolalpha;

    std::cout << source.request_stop() << '\n';
    std::cout << source.request_stop() << '\n';
}
```

خروجی:

```
true
false
```

درخواست توقف یک‌باره است و پس از ثبت شدن امکان Reset کردن Stop State وجود ندارد.

### تفسیر مقدار بازگشتی request_stop

مقدار بازگشتی این تابع نشان می‌دهد که آیا درخواست توقف توسط فراخوانی فعلی برای اولین بار ثبت شده است یا خیر.

مقدار `true` به معنای پایان یافتن عملیات نیست.

برای مثال:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    if (source.request_stop()) {
        std::cout << "Stop request accepted\n";
    } else {
        std::cout << "Stop was already requested\n";
    }
}
```

در این مثال، پذیرش درخواست توقف از پایان یافتن Worker متمایز است.

### ارسال درخواست توقف از طریق std::jthread

کلاس `std::jthread` تابع `request_stop()` دارد که درخواست توقف را برای Stop State داخلی Thread ارسال می‌کند.

```
#include <iostream>
#include <stop_token>
#include <thread>

int main() {
    std::jthread worker(
        [](std::stop_token token) {
            while (!token.stop_requested()) {
                std::this_thread::yield();
            }
        }
    );

    bool requested = worker.request_stop();

    std::cout << std::boolalpha << requested << '\n';
}
```

این تابع به‌صورت مستقیم به Stop State مربوط به Thread دسترسی دارد.

### تفاوت درخواست توقف و پایان Thread

در طراحی برنامه‌های Concurrent باید سه مفهوم زیر را از یکدیگر جدا کرد:

- Stop Request: درخواست توقف ثبت شده است.
- Thread Completion: تابع اجراشونده به پایان رسیده است.
- Thread Join: Thread دیگر قابل Join شدن نیست و پایان اجرای آن توسط Thread دیگری مشاهده شده است.

این سه مفهوم معادل یکدیگر نیستند.

برای مثال، ممکن است درخواست توقف ثبت شود، اما Worker همچنان در حال تکمیل یک عملیات طولانی باشد.

## طراحی توابع قابل لغو

### جداسازی منطق پردازش از Thread

یکی از مزیت‌های اصلی `std::stop_token` این است که به Thread خاصی وابسته نیست.

می‌توان Token را به هر تابعی که قابلیت Cancellation نیاز دارد ارسال کرد.

```
#include <iostream>
#include <stop_token>
#include <thread>
#include <chrono>

bool processItems(
    std::stop_token token,
    int itemCount
) {
    for (int i = 0; i < itemCount; ++i) {
        if (token.stop_requested()) {
            return false;
        }

        std::cout << "Processing item " << i << '\n';

        std::this_thread::sleep_for(
            std::chrono::milliseconds(100)
        );
    }

    return true;
}

int main() {
    std::stop_source source;

    std::jthread worker(
        [&source] {
            bool completed = processItems(
                source.get_token(),
                100
            );

            std::cout << "Completed: "
                      << std::boolalpha
                      << completed << '\n';
        }
    );

    std::this_thread::sleep_for(
        std::chrono::milliseconds(350)
    );

    source.request_stop();
}
```

در این طراحی، تابع `processItems` هیچ وابستگی مستقیمی به `std::jthread` ندارد.

این موضوع باعث افزایش قابلیت استفاده مجدد و تست‌پذیری تابع می‌شود.

### انتقال Token به توابع تو در تو

در سیستم‌های بزرگ، یک عملیات ممکن است از چندین تابع تشکیل شده باشد.

اگر چندین مرحله از پردازش قابلیت لغو داشته باشند، می‌توان Token را به آن‌ها منتقل کرد.

```
#include <stop_token>

bool validateData(std::stop_token token) {
    if (token.stop_requested()) {
        return false;
    }

    return true;
}

bool transformData(std::stop_token token) {
    if (!validateData(token)) {
        return false;
    }

    if (token.stop_requested()) {
        return false;
    }

    return true;
}

bool executeTask(std::stop_token token) {
    return transformData(token);
}
```

در این معماری، هر مرحله می‌تواند به‌صورت مستقل به درخواست توقف واکنش نشان دهد.

البته بهتر است Token فقط به بخش‌هایی منتقل شود که واقعاً به Cancellation نیاز دارند.

### طراحی Token اختیاری

در بعضی APIها، قابلیت Cancellation اختیاری است.

در این شرایط می‌توان از Token خالی به‌عنوان مقدار پیش‌فرض استفاده کرد.

```
#include <stop_token>
#include <iostream>

void execute(std::stop_token token = {}) {
    if (token.stop_requested()) {
        return;
    }

    std::cout << "Executing operation\n";
}

int main() {
    execute();

    std::stop_source source;

    execute(source.get_token());
}
```

در این طراحی، فراخواننده می‌تواند بدون ساخت Stop Source از تابع استفاده کند.

Token خالی هیچ Stop Stateای ندارد و در نتیجه درخواست توقفی نیز برای آن ثبت نمی‌شود.

## استفاده از Stop Callback

### مفهوم Callback هنگام توقف

گاهی بررسی دوره‌ای `stop_requested()` کافی نیست.

برای مثال، ممکن است Thread در انتظار دریافت داده از یک سیستم دیگر باشد. اگر Thread در یک عملیات Blocking قرار داشته باشد، ممکن است مدت زیادی طول بکشد تا دوباره به نقطه بررسی Token برسد.

در چنین شرایطی، `std::stop_callback` امکان اجرای یک Callable هنگام ثبت درخواست توقف را فراهم می‌کند.

### نحوه ثبت Callback

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    auto token = source.get_token();

    std::stop_callback callback(
        token,
        [] {
            std::cout << "Cancellation event received\n";
        }
    );

    source.request_stop();
}
```

Callback هنگام ثبت درخواست توقف اجرا می‌شود.

### رفتار Callback در Threadهای مختلف

یکی از نکات مهم درباره `std::stop_callback` این است که اجرای Callback الزاماً در Thread ثبت‌کننده آن اتفاق نمی‌افتد.

اگر درخواست توقف توسط یک Thread ارسال شود، Callbackهای ثبت‌شده معمولاً در همان Thread اجرا می‌شوند.

اما اگر Callback پس از ثبت درخواست توقف ایجاد شود، Callback در همان Threadی اجرا می‌شود که Constructor مربوط به `std::stop_callback` را اجرا کرده است.

بنابراین:

- Callback ممکن است در Thread درخواست‌دهنده اجرا شود.
- Callback ممکن است هنگام ساخت شیء Callback اجرا شود.
- Callback ممکن است هم‌زمان با سایر عملیات برنامه اجرا شود.

به همین دلیل، Callback باید کوتاه، قابل پیش‌بینی و Thread-safe باشد.

### ثبت Callback پس از درخواست توقف

اگر Stop State از قبل درخواست توقف دریافت کرده باشد، Callback هنگام ساخت شیء `std::stop_callback` بلافاصله اجرا می‌شود.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    source.request_stop();

    std::stop_callback callback(
        source.get_token(),
        [] {
            std::cout << "Callback executed immediately\n";
        }
    );
}
```

این رفتار اهمیت زیادی دارد؛ زیرا نباید فرض کرد Callback حتماً پس از Constructor اجرا خواهد شد.

### حذف Callback

Callback تا زمانی که شیء `std::stop_callback` زنده باشد، ثبت‌شده باقی می‌ماند.

با Destroy شدن این شیء، ثبت Callback حذف می‌شود.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    {
        std::stop_callback callback(
            source.get_token(),
            [] {
                std::cout << "Callback executed\n";
            }
        );
    }

    source.request_stop();
}
```

در این مثال، Callback پیش از درخواست توقف از بین رفته است؛ بنابراین اجرا نمی‌شود.

### نکات هم‌زمانی در Destructor مربوط به Callback

Destructor مربوط به `std::stop_callback` باید با اجرای هم‌زمان Callback هماهنگ شود.

اگر Callback در Thread دیگری در حال اجرا باشد، Destroy کردن شیء Callback می‌تواند تا پایان اجرای آن منتظر بماند.

بنابراین، Destroy کردن Callback از داخل خودش می‌تواند به Deadlock منجر شود.

همچنین اگر Callback هنگام اجرا Lockی را در اختیار داشته باشد و Thread حذف‌کننده برای ادامه کار به همان Lock نیاز داشته باشد، احتمال Deadlock وجود دارد.

توصیه: از عملیات طولانی، Blocking یا وابسته به Lockهای خارجی در Callback خودداری کنید.

### استفاده از Callback برای بیدار کردن Thread

یکی از کاربردهای مهم Callback، خارج کردن Thread از وضعیت انتظار است.

برای مثال، فرض کنید یک Thread منتظر دریافت داده از یک Queue است. اگر Queue خالی باشد، Thread ممکن است در یک عملیات Blocking باقی بماند.

در این شرایط می‌توان از Callback برای ارسال Notification استفاده کرد.

```
#include <condition_variable>
#include <mutex>
#include <stop_token>
#include <thread>
#include <iostream>

int main() {
    std::mutex mutex;
    std::condition_variable condition;
    bool ready = false;

    std::jthread worker(
        [&](std::stop_token token) {
            std::stop_callback callback(
                token,
                [&] {
                    condition.notify_all();
                }
            );

            std::unique_lock<std::mutex> lock(mutex);

            condition.wait(lock, [&] {
                return ready || token.stop_requested();
            });

            if (token.stop_requested()) {
                std::cout << "Worker cancelled\n";
                return;
            }

            std::cout << "Worker resumed\n";
        }
    );

    worker.request_stop();
}
```

در این مثال، Callback هنگام درخواست توقف، Condition Variable را بیدار می‌کند.

با این حال، برای این نوع انتظار در ++C20 راهکار مستقیم‌تری نیز وجود دارد که در بخش بعد بررسی می‌شود.

## انتظار قابل لغو با std::condition_variable_any

### مشکل عملیات Blocking

در بسیاری از برنامه‌های Concurrent، Threadها برای دریافت داده یا برقرار شدن یک شرط در وضعیت انتظار قرار می‌گیرند.

برای مثال:

```
condition.wait(lock);
```

اگر درخواست توقف ارسال شود، این عملیات به‌صورت خودکار الزاماً بیدار نمی‌شود.

در نتیجه، ممکن است درخواست توقف ثبت شده باشد، اما Thread همچنان در وضعیت انتظار باقی بماند.

استاندارد ++C20 برای حل این مسئله، Overloadهای جدیدی را به `std::condition_variable_any` اضافه کرده است که مستقیماً از `std::stop_token` پشتیبانی می‌کنند.

### استفاده از Overload قابل لغو wait

نمونه زیر یک Worker را نشان می‌دهد که تا زمان آماده شدن داده یا دریافت درخواست توقف منتظر می‌ماند.

```
#include <condition_variable>
#include <mutex>
#include <stop_token>
#include <thread>
#include <iostream>
#include <chrono>

int main() {
    std::mutex mutex;
    std::condition_variable_any condition;

    bool ready = false;

    std::jthread worker(
        [&](std::stop_token token) {
            std::unique_lock<std::mutex> lock(mutex);

            bool completed = condition.wait(
                lock,
                token,
                [&] {
                    return ready;
                }
            );

            if (!completed) {
                std::cout << "Operation cancelled\n";
                return;
            }

            std::cout << "Data is ready\n";
        }
    );

    std::this_thread::sleep_for(
        std::chrono::milliseconds(500)
    );

    worker.request_stop();
}
```

در این مثال، Worker تا زمانی که Predicate برقرار نشده یا درخواست توقف دریافت نشده باشد، منتظر می‌ماند.

در صورت دریافت درخواست توقف، عملیات Wait بیدار می‌شود و مقدار `false` برمی‌گرداند.

### بررسی امضای wait

یکی از Overloadهای این تابع به شکل زیر است:

```
template<class Lock, class Predicate>
bool wait(
    Lock& lock,
    std::stop_token stoken,
    Predicate pred
);
```

رفتار این تابع به شکل زیر است:

- اگر Predicate برقرار باشد، مقدار `true` برمی‌گرداند.
- اگر درخواست توقف پیش از برقرار شدن Predicate مشاهده شود، مقدار `false` برمی‌گرداند.
- هنگام انتظار، Lock آزاد می‌شود.
- پیش از بازگشت تابع، Lock دوباره دریافت می‌شود.

نکته مهم این است که Predicate بر وضعیت توقف اولویت دارد. اگر هنگام بررسی نهایی، Predicate برقرار باشد، نتیجه تابع `true` خواهد بود؛ حتی اگر درخواست توقف نیز ثبت شده باشد.

بنابراین، مقدار بازگشتی تابع باید بر اساس سیاست موردنظر برنامه تفسیر شود.

### استفاده از wait_for و wait_until

علاوه بر `wait`، کلاس `std::condition_variable_any` در ++C20 از Overloadهای مبتنی بر Stop Token برای انتظار زمان‌دار نیز پشتیبانی می‌کند.

این Overloadها عبارت‌اند از:

```
wait_until(lock, token, abs_time, predicate);
```

و:

```
wait_for(lock, token, rel_time, predicate);
```

این توابع امکان ترکیب سه شرط را فراهم می‌کنند:

- برقرار شدن Predicate؛
- دریافت درخواست توقف؛
- پایان یافتن زمان انتظار.

مثال:

```
#include <condition_variable>
#include <mutex>
#include <stop_token>
#include <thread>
#include <chrono>
#include <iostream>

int main() {
    std::mutex mutex;
    std::condition_variable_any condition;

    bool ready = false;

    std::jthread worker(
        [&](std::stop_token token) {
            std::unique_lock<std::mutex> lock(mutex);

            bool completed = condition.wait_for(
                lock,
                token,
                std::chrono::seconds(5),
                [&] {
                    return ready;
                }
            );

            if (completed) {
                std::cout << "Operation completed\n";
            } else {
                std::cout << "Cancelled or timed out\n";
            }
        }
    );

    worker.join();
}
```

در این مثال، مقدار `false` به‌تنهایی مشخص نمی‌کند که انتظار به دلیل Timeout یا درخواست توقف خاتمه یافته است.

اگر برنامه به تشخیص دقیق علت نیاز داشته باشد، باید وضعیت Token و سیاست زمانی را نیز بررسی کند.

## تفاوت `std::condition_variable` و `std::condition_variable_any`

هر دو ابزار برای هماهنگ‌سازی Threadها و انتظار تا زمان برقرار شدن یک شرط استفاده می‌شوند؛ اما از نظر نوع Lock قابل استفاده و پشتیبانی از `std::stop_token` تفاوت‌هایی دارند.

### تفاوت در نوع Lock

کلاس `std::condition_variable` فقط با `std::unique_lock<std::mutex>` کار می‌کند. این محدودیت باعث می‌شود پیاده‌سازی آن بتواند از بهینه‌سازی‌های مخصوص `std::mutex` استفاده کند.

در مقابل، کلاس `std::condition_variable_any` با انواع Lock سازگار است؛ به شرط آنکه نوع Lock عملیات موردنیاز، مانند `lock()` و `unlock()`، را فراهم کند.

برای مثال، می‌توان از `std::condition_variable_any` همراه با `std::shared_mutex` یا Lockهای سفارشی استفاده کرد.

### تفاوت در پشتیبانی از Stop Token

مهم‌ترین تفاوت برای برنامه‌نویسی مبتنی بر Cancellation این است که `std::condition_variable_any` در ++C20 overloadهایی از توابع انتظار دارد که مستقیماً `std::stop_token` دریافت می‌کنند.

در مقابل، `std::condition_variable` چنین overloadهایی ندارد.

نمونه استفاده از `std::condition_variable_any`:

```
#include <condition_variable>
#include <mutex>
#include <stop_token>
#include <thread>
#include <iostream>

int main() {
    std::mutex mutex;
    std::condition_variable_any cv;
    bool ready = false;

    std::jthread worker([&](std::stop_token token) {
        std::unique_lock<std::mutex> lock(mutex);

        bool completed = cv.wait(lock, token, [&] {
            return ready;
        });

        if (!completed) {
            std::cout << "Operation cancelled\n";
            return;
        }

        std::cout << "Condition satisfied\n";
    });

    worker.request_stop();
}
```

در این مثال، Thread منتظر می‌ماند تا شرط `ready` برقرار شود یا درخواست توقف دریافت کند. اگر انتظار به دلیل درخواست توقف خاتمه پیدا کند و شرط برقرار نباشد، مقدار بازگشتی `wait()` برابر `false` خواهد بود.

### جدول مقایسه

| ویژگی                           | `condition_variable`     | `condition_variable_any`      |
| ------------------------------- | ------------------------ | ----------------------------- |
| نوع Lock                        | فقط `unique_lock<mutex>` | انواع Lock سازگار             |
| پشتیبانی مستقیم از `stop_token` | ندارد                    | دارد                          |
| سازگاری با Lock سفارشی          | ندارد                    | دارد                          |
| سربار پیاده‌سازی                | معمولاً کمتر             | ممکن است بیشتر باشد           |
| کاربرد رایج                     | انتظار با Mutex معمولی   | انتظار انعطاف‌پذیر و قابل لغو |

بنابراین، اگر به انتظار قابل لغو با `std::stop_token` نیاز دارید، `std::condition_variable_any` انتخاب طبیعی‌تری است. اگر چنین نیازی ندارید و فقط از `std::mutex` استفاده می‌کنید، معمولاً `std::condition_variable` انتخاب ساده‌تر و کم‌هزینه‌تری خواهد بود.

## الگوهای طراحی برای Cancellation

استفاده صحیح از `std::stop_token` فقط به فراخوانی `request_stop()` محدود نمی‌شود. طراحی API و ساختار Threadها باید از ابتدا با مفهوم Cooperative Cancellation سازگار باشد.

### طراحی تابع‌های قابل لغو

بهتر است توابعی که عملیات طولانی یا قابل توقف انجام می‌دهند، `std::stop_token` را به‌عنوان یکی از پارامترهای خود دریافت کنند.

برای مثال:

```
#include <stop_token>
#include <chrono>
#include <thread>

void process_data(std::stop_token token) {
    while (!token.stop_requested()) {
        perform_one_step();

        std::this_thread::sleep_for(
            std::chrono::milliseconds(100)
        );
    }
}
```

در این الگو، عملیات به بخش‌های کوچک تقسیم شده است و تابع پس از هر مرحله امکان بررسی درخواست توقف را دارد.

این روش چند مزیت دارد:

- تابع به سازوکار مدیریت Thread وابسته نمی‌شود.
- امکان استفاده از همان تابع در Threadهای مختلف وجود دارد.
- توقف عملیات به‌صورت تدریجی و کنترل‌شده انجام می‌شود.
- تابع می‌تواند منابعی را که در اختیار دارد، پیش از خروج آزاد کند.

در پروژه‌های بزرگ بهتر است بررسی Stop Token در نقاط مشخص و قابل پیش‌بینی انجام شود؛ برای مثال، پس از پایان هر مرحله از پردازش یا پیش از شروع یک عملیات پرهزینه.

### پشتیبانی از Stop Token اختیاری

گاهی لازم است یک تابع هم به‌صورت قابل لغو و هم بدون قابلیت لغو قابل استفاده باشد.

در این شرایط می‌توان `std::stop_token` را با مقدار پیش‌فرض دریافت کرد:

```
#include <stop_token>

void process_data(
    std::stop_token token = {}
) {
    while (!token.stop_requested()) {
        perform_one_step();
    }
}
```

مقدار پیش‌فرض `{}` یک Token بدون Stop State ایجاد می‌کند. در نتیجه، `stop_requested()` برای آن همیشه `false` خواهد بود.

این طراحی باعث می‌شود فراخوانی تابع بدون ارسال Token نیز امکان‌پذیر باشد:

```
process_data();
```

همچنین می‌توان یک Token واقعی به آن ارسال کرد:

```
std::stop_source source;

process_data(source.get_token());

source.request_stop();
```

البته در این نمونه، چون فراخوانی تابع به‌صورت هم‌زمان انجام می‌شود، درخواست توقف باید از Thread دیگری صورت بگیرد یا ساختار برنامه به‌گونه‌ای تغییر کند که امکان درخواست توقف هنگام اجرای تابع وجود داشته باشد.

### استفاده از Callback برای آزادسازی منابع

گاهی Thread هنگام انتظار در یک API خارجی یا یک عملیات مسدودکننده قرار دارد و امکان بررسی مداوم `stop_requested()` وجود ندارد.

در چنین شرایطی می‌توان از `std::stop_callback` برای اطلاع‌رسانی به سازوکار مسدودکننده استفاده کرد.

```
#include <stop_token>
#include <mutex>
#include <condition_variable>

void worker_function(std::stop_token token) {
    std::mutex mutex;
    std::condition_variable_any cv;
    bool ready = false;

    std::stop_callback callback(token, [&] {
        cv.notify_all();
    });

    std::unique_lock<std::mutex> lock(mutex);

    cv.wait(lock, [&] {
        return ready || token.stop_requested();
    });
}
```

در این مثال، با درخواست توقف، Callback اجرا می‌شود و Thread منتظر را بیدار می‌کند.

البته در این نمونه، استفاده از overload مخصوص Stop Token در `condition_variable_any` نیز امکان‌پذیر است و معمولاً انتخاب ساده‌تری خواهد بود. Callback زمانی مفیدتر است که لازم باشد یک API دیگر یا چند سازوکار انتظار به‌صورت هم‌زمان مطلع شوند.

## پیاده‌سازی Thread Pool قابل لغو

یکی از کاربردهای مهم `std::stop_token` در طراحی Thread Pool است.

در یک Thread Pool معمولی، چند Worker از یک صف مشترک Task دریافت می‌کنند. هنگام خاموش شدن Pool، باید Workerها از انتظار خارج شوند و کارهای در حال اجرا نیز فرصت خاتمه کنترل‌شده داشته باشند.

نمونه زیر یک پیاده‌سازی ساده از این الگو را نشان می‌دهد:

```
#include <condition_variable>
#include <functional>
#include <mutex>
#include <queue>
#include <thread>
#include <vector>
#include <stop_token>

class ThreadPool {
public:
    explicit ThreadPool(std::size_t count) {
        for (std::size_t i = 0; i < count; ++i) {
            workers_.emplace_back(
                [this](std::stop_token token) {
                    worker_loop(token);
                }
            );
        }
    }

    void submit(std::function<void()> task) {
        {
            std::lock_guard<std::mutex> lock(mutex_);
            tasks_.push(std::move(task));
        }

        cv_.notify_one();
    }

    ~ThreadPool() {
        for (auto& worker : workers_) {
            worker.request_stop();
        }

        cv_.notify_all();
    }

private:
    void worker_loop(std::stop_token token) {
        while (true) {
            std::function<void()> task;

            {
                std::unique_lock<std::mutex> lock(mutex_);

                cv_.wait(lock, token, [&] {
                    return !tasks_.empty();
                });

                if (token.stop_requested()) {
                    return;
                }

                if (tasks_.empty()) {
                    continue;
                }

                task = std::move(tasks_.front());
                tasks_.pop();
            }

            task();
        }
    }

    std::mutex mutex_;
    std::condition_variable_any cv_;
    std::queue<std::function<void()>> tasks_;
    std::vector<std::jthread> workers_;
};
```

این پیاده‌سازی صرفاً یک نمونه آموزشی است و چند نکته مهم دارد:

- Workerها با `std::jthread` ساخته می‌شوند و هر کدام Token مخصوص خود را دریافت می‌کنند.
- صف Taskها با Mutex محافظت می‌شود.
- `condition_variable_any` امکان بیدار شدن در اثر Stop Request را فراهم می‌کند.
- هنگام تخریب Thread Pool، برای Workerها درخواست توقف ارسال می‌شود.
- پس از پایان Destructor، اعضای `workers_` تخریب می‌شوند و Destructor مربوط به `std::jthread` عملیات Join را انجام می‌دهد.

در این نمونه، Worker در صورت دریافت درخواست توقف از حلقه خارج می‌شود؛ بنابراین ممکن است Taskهایی که هنوز در صف هستند اجرا نشوند.

همچنین اگر یک Task به‌صورت طولانی اجرا شود و خودش قابلیت لغو نداشته باشد، درخواست توقف Worker نمی‌تواند آن Task را متوقف کند. برای پشتیبانی از لغو Taskهای در حال اجرا، باید خود Task نیز یک `std::stop_token` دریافت کند.

### تفاوت Shutdown فوری و Shutdown تدریجی

در طراحی Thread Pool باید مشخص شود که هنگام Shutdown چه اتفاقی برای Taskهای باقی‌مانده می‌افتد.

دو سیاست رایج وجود دارد:

- Shutdown فوری: Workerها در اولین فرصت از حلقه خارج می‌شوند و Taskهای اجرا نشده ممکن است کنار گذاشته شوند.
- Shutdown تدریجی: ابتدا ورود Task جدید متوقف می‌شود، سپس Workerها تمام Taskهای باقی‌مانده را اجرا می‌کنند و در نهایت خاتمه می‌یابند.

این دو رفتار یکسان نیستند و نباید صرفاً با استفاده از `request_stop()` فرض کرد که تمام Taskها اجرا خواهند شد.

برای پیاده‌سازی Shutdown تدریجی معمولاً باید یک وضعیت جداگانه مانند `accepting_tasks_` یا `shutdown_` در نظر گرفت و منطق خروج Workerها را بر اساس وضعیت صف و سیاست Shutdown طراحی کرد.

## مقایسه Stop Token با Atomic Flag

قبل از معرفی `std::stop_token`، یکی از روش‌های متداول برای توقف Thread استفاده از `std::atomic<bool>` بود.

برای مثال:

```
#include <atomic>
#include <thread>

std::atomic<bool> stop_requested = false;

void worker() {
    while (!stop_requested.load()) {
        perform_one_step();
    }
}

int main() {
    std::thread thread(worker);

    stop_requested.store(true);

    thread.join();
}
```

این روش ساده است و در بسیاری از سناریوها همچنان کاربرد دارد؛ اما `std::stop_token` قابلیت‌های بیشتری ارائه می‌دهد.

| ویژگی                                       | `atomic<bool>`        | `stop_token`             |
| ------------------------------------------- | --------------------- | ------------------------ |
| درخواست توقف                                | با تغییر مقدار Atomic | با `request_stop()`      |
| مشاهده وضعیت توقف                           | با `load()`           | با `stop_requested()`    |
| پشتیبانی از Callback                        | ندارد                 | دارد                     |
| اشتراک وضعیت بین چند Token                  | نیازمند طراحی دستی    | داخلی                    |
| اتصال به `jthread`                          | دستی                  | مستقیم                   |
| انتظار قابل لغو با `condition_variable_any` | ندارد                 | دارد                     |
| مدیریت Stop State                           | دستی                  | مدیریت‌شده توسط کتابخانه |

### چه زمانی Atomic Flag مناسب‌تر است؟

اگر تنها نیاز شما یک پرچم ساده برای هماهنگ‌سازی دو Thread است، استفاده از `std::atomic<bool>` می‌تواند کافی باشد.

اما اگر به چندین مصرف‌کننده، Callback، اتصال به `std::jthread` یا انتظار قابل لغو نیاز دارید، `std::stop_token` طراحی کامل‌تری ارائه می‌کند.

نکته مهم این است که Stop Token جایگزین تمام کاربردهای Atomic نیست. این دو ابزار اهداف متفاوتی دارند.

## Memory Model و Thread Safety

یکی از ویژگی‌های مهم `std::stop_token` این است که برای استفاده هم‌زمان از چند Thread طراحی شده است.

چند Thread می‌توانند یک Token مشترک داشته باشند و وضعیت توقف را مشاهده کنند.

برای مثال:

```
#include <stop_token>
#include <thread>
#include <vector>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        perform_one_step();
    }
}

int main() {
    std::stop_source source;
    std::vector<std::jthread> workers;

    auto token = source.get_token();

    for (int i = 0; i < 4; ++i) {
        workers.emplace_back([token] {
            worker(token);
        });
    }

    source.request_stop();
}
```

در این مثال، چهار Worker یک Stop Token مشترک دریافت می‌کنند. با فراخوانی `request_stop()`، همه آن‌ها می‌توانند درخواست توقف را مشاهده کنند.

### تضمین‌های هم‌زمانی

سازوکار Stop Token برای استفاده هم‌زمان از چند Thread ایمن طراحی شده است؛ اما این موضوع به معنای Thread-Safe بودن تمام داده‌هایی نیست که Callbackها یا Workerها به آن‌ها دسترسی دارند.

برای مثال، اگر چند Callback هم‌زمان به یک متغیر معمولی دسترسی داشته باشند و دست‌کم یکی از آن‌ها آن متغیر را تغییر دهد، ممکن است Data Race رخ دهد.

بنابراین:

- برای داده‌های مشترک از Mutex یا Atomic استفاده کنید.
- فرض نکنید اجرای Callback به‌صورت خودکار تمام داده‌های برنامه را محافظت می‌کند.
- به ترتیب اجرای Callbackها تکیه نکنید.
- برای هماهنگی بین Threadها از Synchronization Primitive مناسب استفاده کنید.

در ++C، درخواست توقف و اجرای Callbackها دارای تضمین‌های هم‌زمانی تعریف‌شده در استاندارد هستند؛ بااین‌حال، Stop Token جایگزین Mutex، Atomic یا سایر ابزارهای Synchronization نیست.

## محدودیت‌ها و نکات مهم

با وجود امکانات مفید `std::stop_token`، این سازوکار محدودیت‌هایی دارد که در طراحی سیستم‌های چندنخی باید در نظر گرفته شوند.

### توقف اجباری انجام نمی‌دهد

درخواست توقف فقط یک درخواست است. اگر Thread آن را بررسی نکند، ممکن است به اجرای خود ادامه دهد.

برای مثال، حلقه زیر قابل لغو نیست:

```
void worker(std::stop_token token) {
    while (true) {
        perform_one_step();
    }
}
```

حتی اگر Stop Request ارسال شود، تابع هیچ‌وقت وضعیت آن را بررسی نمی‌کند.

### عملیات Blocking خودکار لغو نمی‌شوند

اگر Thread در یک تابع Blocking قرار داشته باشد، ارسال درخواست توقف لزوماً باعث خروج آن از تابع نمی‌شود.

برای مثال، یک فراخوانی شبکه یا ورودی‌وخروجی فایل ممکن است تا زمان وقوع رویدادی دیگر مسدود بماند.

برای حل این مشکل باید از APIهایی استفاده کرد که Cancellation را پشتیبانی می‌کنند یا با استفاده از Callback و سازوکارهای بیدارسازی، امکان خروج از حالت انتظار را فراهم کرد.

### Callbackها می‌توانند باعث Deadlock شوند

Callbackهای مربوط به `std::stop_callback` به‌صورت هم‌زمان و در Thread درخواست‌دهنده اجرا می‌شوند.

بنابراین اگر Callback قفلی را دریافت کند که Thread درخواست‌دهنده از قبل در اختیار دارد، احتمال Deadlock وجود دارد.

همچنین بهتر است Callbackها کوتاه و کم‌هزینه باشند و از اجرای عملیات Blocking غیرضروری در آن‌ها خودداری شود.

### Stop Request معادل پایان Thread نیست

ارسال درخواست توقف به معنای پایان یافتن Thread نیست.

برای اطمینان از پایان Thread باید از `join()` استفاده شود یا از طول عمر مدیریت‌شده `std::jthread` بهره برد.

### Stop Token برای مدیریت منابع کافی نیست

Stop Token به‌تنهایی تضمین نمی‌کند که منابعی مانند فایل، Socket، Mutex یا حافظه آزاد شوند.

آزادسازی منابع باید با طراحی مناسب و استفاده از RAII انجام شود.

## اشتباهات رایج

در استفاده از Stop Token چند اشتباه متداول وجود دارد که بهتر است از آن‌ها اجتناب شود.

### تصور توقف فوری Thread

فراخوانی `request_stop()` فقط درخواست توقف را ثبت می‌کند. Thread باید در نقاط مناسب آن را بررسی کند یا در یک سازوکار انتظار قابل لغو قرار داشته باشد.

### فراموش کردن Join

در برنامه‌هایی که از `std::thread` استفاده می‌کنند، درخواست توقف به‌تنهایی جایگزین `join()` نیست.

### اجرای عملیات سنگین در Callback

از آنجا که Callback به‌صورت هم‌زمان در Thread درخواست‌دهنده اجرا می‌شود، انجام عملیات سنگین یا Blocking در آن می‌تواند باعث تأخیر و مشکلات هم‌زمانی شود.

### استفاده از Reference با طول عمر نامعتبر

اگر Callback به متغیری از طریق Reference دسترسی داشته باشد، آن متغیر باید تا پایان عمر Callback معتبر بماند.

### فرض ترتیب اجرای Callbackها

استاندارد ++C ترتیب اجرای چند Callback ثبت‌شده روی یک Stop State را تضمین نمی‌کند. بنابراین نباید برای اجرای مرحله‌ای عملیات به این ترتیب تکیه کرد.

### تصور لغو خودکار Taskها

Stop Token به‌صورت خودکار Taskهای موجود در صف یا عملیات در حال اجرا را متوقف نمی‌کند. هر Task باید سازوکار Cancellation مناسب خود را داشته باشد.

## بهترین روش‌ها

برای استفاده مطمئن‌تر از `std::stop_token` رعایت چند اصل توصیه می‌شود:

- توابع طولانی را به بخش‌های کوچک تقسیم کنید.
- در نقاط مناسب، وضعیت `stop_requested()` را بررسی کنید.
- برای انتظارهای قابل لغو از overloadهای `condition_variable_any` استفاده کنید.
- Callbackها را کوتاه، سریع و بدون عملیات Blocking غیرضروری نگه دارید.
- برای Threadهای جدید از `std::jthread` استفاده کنید، مگر اینکه دلیل مشخصی برای استفاده از `std::thread` وجود داشته باشد.
- در طراحی Thread Pool، سیاست Shutdown فوری یا تدریجی را از ابتدا مشخص کنید.
- برای Taskهای طولانی، Stop Token را به خود Task نیز منتقل کنید.
- منابع را با RAII مدیریت کنید.
- برای داده‌های مشترک از Mutex یا Atomic مناسب استفاده کنید.
- به ترتیب اجرای Callbackها وابسته نشوید.
- میان درخواست توقف، پایان عملیات و Join تفاوت قائل شوید.

## مثال کامل: Worker قابل لغو با صف کار

در مثال زیر یک Worker داریم که از صف مشترک Task دریافت می‌کند و با استفاده از `std::stop_token` و `std::condition_variable_any` قابل توقف است.

```
#include <condition_variable>
#include <functional>
#include <mutex>
#include <queue>
#include <stop_token>
#include <thread>
#include <iostream>

class Worker {
public:
    void submit(std::function<void()> task) {
        {
            std::lock_guard<std::mutex> lock(mutex_);
            tasks_.push(std::move(task));
        }

        cv_.notify_one();
    }

    void run(std::stop_token token) {
        while (true) {
            std::function<void()> task;

            {
                std::unique_lock<std::mutex> lock(mutex_);

                cv_.wait(lock, token, [&] {
                    return !tasks_.empty();
                });

                if (token.stop_requested()) {
                    return;
                }

                if (tasks_.empty()) {
                    continue;
                }

                task = std::move(tasks_.front());
                tasks_.pop();
            }

            task();
        }
    }

private:
    std::mutex mutex_;
    std::condition_variable_any cv_;
    std::queue<std::function<void()>> tasks_;
};

int main() {
    Worker worker;

    worker.submit([] {
        std::cout << "Task 1\n";
    });

    worker.submit([] {
        std::cout << "Task 2\n";
    });

    std::jthread thread([&](std::stop_token token) {
        worker.run(token);
    });

    thread.request_stop();
}
```

در این مثال:

- کلاس `Worker` یک صف مشترک برای نگهداری Taskها دارد.
- تابع `submit()` وظیفه افزودن Task به صف را بر عهده دارد.
- تابع `run()` تا زمان دریافت Task یا درخواست توقف منتظر می‌ماند.
- `condition_variable_any` امکان لغو انتظار با Stop Token را فراهم می‌کند.
- با فراخوانی `request_stop()`، Worker از انتظار خارج می‌شود.
- با تخریب `std::jthread`، عملیات Join انجام می‌شود.

این مثال یک پیاده‌سازی آموزشی است. در یک سیستم واقعی، باید مشخص شود که هنگام توقف Worker، Taskهای باقی‌مانده چه وضعیتی خواهند داشت و آیا Taskهای در حال اجرا نیز باید قابل لغو باشند یا خیر.

همچنین در این مثال، چون `thread` پس از `worker` ساخته شده است، هنگام خروج از `main()` ابتدا `thread` تخریب و Join می‌شود و سپس `worker` تخریب خواهد شد. در نتیجه، Worker در طول اجرای Thread معتبر باقی می‌ماند.

## سازگاری با کامپایلرها و استاندارد

قابلیت‌های `std::stop_token` از ++C20 به استاندارد اضافه شده‌اند.

برای استفاده از آن‌ها باید کامپایلر و کتابخانه استاندارد مورد استفاده، پشتیبانی لازم از C++20 را داشته باشند.

Headerهای اصلی عبارت‌اند از:

| Header                 | کاربرد                                                |
| ---------------------- | ----------------------------------------------------- |
| `<stop_token>`         | تعریف `stop_source`، `stop_token` و `stop_callback`   |
| `<thread>`             | تعریف `std::jthread`                                  |
| `<condition_variable>` | تعریف `condition_variable` و `condition_variable_any` |
| `<mutex>`              | تعریف Mutex و Lockها                                  |
| `<chrono>`             | ابزارهای زمان‌بندی و Timeout                          |

برای بررسی پشتیبانی کتابخانه استاندارد می‌توان از Feature Test Macro استفاده کرد:

```
#include <version>

#if defined(__cpp_lib_jthread) && __cpp_lib_jthread >= 201911L
    // C++20 jthread and stop token support is available.
#else
    #error "Required C++20 thread support is unavailable"
#endif
```

در کامپایلر GCC معمولاً می‌توان از گزینه زیر استفاده کرد:

```
g++ -std=c++20 main.cpp -pthread
```

در Clang نیز:

```
clang++ -std=c++20 main.cpp -pthread
```

در MSVC:

```
cl /std:c++20 /EHsc main.cpp
```

البته پشتیبانی واقعی به نسخه کامپایلر و نسخه Standard Library وابسته است؛ بنابراین فعال کردن گزینه ++C20 به‌تنهایی تضمین نمی‌کند که تمام قابلیت‌های کتابخانه‌ای در دسترس باشند.

## جمع‌بندی

`std::stop_token` یکی از امکانات مهم ++C20 برای طراحی سیستم‌های چندنخی با قابلیت Cooperative Cancellation است.

این ابزار امکان جداسازی درخواست توقف از اجرای واقعی Thread را فراهم می‌کند و به کمک `std::stop_source`، `std::stop_callback` و `std::jthread` می‌توان چرخه عمر Threadها را بهتر مدیریت کرد.

مهم‌ترین نکاتی که باید به خاطر داشت عبارت‌اند از:

- `std::stop_source` برای درخواست توقف استفاده می‌شود.
- `std::stop_token` امکان مشاهده وضعیت توقف را فراهم می‌کند.
- `std::stop_callback` برای واکنش به درخواست توقف به کار می‌رود.
- `std::jthread` مدیریت Thread و درخواست توقف هنگام تخریب را ساده‌تر می‌کند.
- `std::condition_variable_any` امکان انتظار قابل لغو با Stop Token را فراهم می‌کند.
- درخواست توقف، Thread را به‌صورت اجباری خاتمه نمی‌دهد.
- برای پایان قطعی Thread باید از Join و مدیریت صحیح طول عمر استفاده کرد.

در نهایت، مهم‌ترین نکته این است که Cancellation باید بخشی از طراحی سیستم باشد، نه قابلیتی که در انتهای پیاده‌سازی به آن اضافه می‌شود. اگر APIها، Taskها و سازوکارهای انتظار از ابتدا با مفهوم توقف همکاری‌محور طراحی شوند، ساخت برنامه‌های چندنخی قابل پیش‌بینی‌تر، امن‌تر و قابل نگهداری‌تر خواهد بود.


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>