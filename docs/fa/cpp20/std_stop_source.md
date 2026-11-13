<div align="center">

[🇺🇸 English](../../en/cpp20/std_stop_source.md) | [🇮🇷 فارسی](./std_stop_source.md)

</div>

---

# معرفی و بررسی جامع std::stop_source در ++C20

## مقدمه

مدیریت توقف عملیات‌های هم‌زمان یکی از چالش‌های مهم در برنامه‌نویسی Concurrent و Multithreaded است. در بسیاری از برنامه‌ها، یک Thread وظیفه‌ای طولانی‌مدت را اجرا می‌کند، درحالی‌که Thread دیگری باید بتواند در شرایط مشخصی درخواست توقف آن عملیات را صادر کند.

برای مثال، ممکن است در یک برنامه:

- یک Thread در حال دانلود یک فایل بزرگ باشد و کاربر عملیات دانلود را لغو کند.
- یک Thread وظیفه پردازش داده‌ها را انجام دهد و برنامه بخواهد پیش از خاموش شدن، درخواست توقف آن را ارسال کند.
- چند Thread به‌صورت هم‌زمان یک عملیات مشترک را انجام دهند و لازم باشد همه آن‌ها از درخواست توقف مطلع شوند.
- یک Task در حال اجرا باشد و در صورت منقضی شدن مهلت زمانی، اجرای آن به‌صورت کنترل‌شده متوقف شود.

در نسخه‌های قدیمی ++C، کتابخانه استاندارد مکانیزم عمومی و یکپارچه‌ای برای ارسال درخواست توقف تعاونی به Threadها ارائه نمی‌کرد. توسعه‌دهندگان معمولاً مجبور بودند چنین قابلیتی را با استفاده از متغیرهای اشتراکی، `std::atomic`، `std::mutex` و `std::condition_variable` پیاده‌سازی کنند.

استاندارد ++C20 با معرفی مکانیزم Cooperative Cancellation این مشکل را تا حد زیادی حل کرد.

این مکانیزم شامل سه نوع اصلی است:

- `std::stop_source`
- `std::stop_token`
- `std::stop_callback`

نوع `std::stop_source` مسئول ایجاد و ارسال درخواست توقف است. نوع `std::stop_token` امکان بررسی وضعیت درخواست توقف را در اختیار کد دریافت‌کننده قرار می‌دهد و `std::stop_callback` امکان اجرای Callback هنگام ثبت درخواست توقف را فراهم می‌کند.

در این میان، `std::stop_source` نقش اصلی را در مدیریت و انتشار درخواست توقف بر عهده دارد.

## مفهوم Cooperative Cancellation

### مفهوم توقف تعاونی

مفهوم Cooperative Cancellation به این معناست که یک بخش از برنامه درخواست توقف عملیات را صادر می‌کند، اما بخش دیگری که عملیات را اجرا می‌کند، مسئول بررسی این درخواست و تصمیم‌گیری درباره زمان و نحوه متوقف شدن است.

در این مدل، درخواست توقف به معنی متوقف شدن فوری Thread نیست.

برای مثال، فرض کنید Thread زیر یک عملیات پردازشی طولانی را اجرا می‌کند:

```
#include <chrono>
#include <iostream>
#include <thread>

void process() {
    for (int i = 0; i < 100; ++i) {
        std::cout << "Processing: " << i << '\n';

        std::this_thread::sleep_for(
            std::chrono::milliseconds(100)
        );
    }
}

int main() {
    std::jthread worker(process);
}
```

در این مثال، عملیات پردازش تا پایان حلقه ادامه پیدا می‌کند.

اگر Thread دیگری بخواهد این عملیات را متوقف کند، صرفاً ارسال یک درخواست توقف کافی نیست؛ کد اجرایی باید امکان دریافت و بررسی آن درخواست را داشته باشد.

در ++C20 این ارتباط از طریق `std::stop_source` و `std::stop_token` برقرار می‌شود.

### تفاوت توقف تعاونی با توقف اجباری

در طراحی سیستم‌های Multithreaded باید میان دو مفهوم تفاوت قائل شد:

- Cooperative Cancellation: Thread در نقاط مشخصی وضعیت توقف را بررسی می‌کند و در صورت دریافت درخواست، به‌صورت کنترل‌شده خاتمه می‌یابد.
- Forced Termination: اجرای Thread بدون همکاری کد داخلی آن متوقف می‌شود.

مکانیزم Standard Stop Token در ++C20 از مدل اول استفاده می‌کند.

این طراحی مزایای مهمی دارد:

- امکان آزادسازی صحیح منابع فراهم می‌شود.
- اجرای Destructorها و عملیات Cleanup تحت کنترل برنامه باقی می‌ماند.
- احتمال ایجاد وضعیت ناسازگار در داده‌های اشتراکی کاهش پیدا می‌کند.
- منطق خاتمه عملیات می‌تواند متناسب با نیاز برنامه طراحی شود.

نکته مهم این است که `std::stop_source` هیچ مکانیزمی برای Kill کردن مستقیم یک Thread ندارد.

## معرفی std::stop_source

### تعریف و نقش اصلی

نوع `std::stop_source` یک کلاس از Standard Library در ++C20 است که امکان ایجاد، مدیریت و ارسال درخواست توقف را فراهم می‌کند.

این کلاس مالک یک Stop State مشترک است. بخش‌های مختلف برنامه می‌توانند از طریق `std::stop_token` به همان Stop State دسترسی داشته باشند و وضعیت درخواست توقف را بررسی کنند.

ساختار کلی ارتباط به شکل زیر است:

```
                 std::stop_source
                        |
                        |
                 request_stop()
                        |
                        v
                 Shared Stop State
                        |
             +----------+----------+
             |          |          |
             v          v          v
          Token 1    Token 2    Token 3
             |          |          |
             v          v          v
          Thread 1  Thread 2  Thread 3
```

در این ساختار، یک `std::stop_source` می‌تواند درخواست توقف را برای چندین `std::stop_token` منتشر کند.

تمام Tokenهایی که به یک Stop State مشترک متصل هستند، وضعیت یکسانی از درخواست توقف را مشاهده می‌کنند.

### هدر مورد نیاز

برای استفاده از `std::stop_source` باید هدر زیر را اضافه کنید:

```
#include <stop_token>
```

این هدر انواع و قابلیت‌های مرتبط با Stop Token را در اختیار برنامه قرار می‌دهد.

### فضای نام

نوع `std::stop_source` در فضای نام `std` قرار دارد.

```
#include <stop_token>

int main() {
    std::stop_source source;
}
```

### ارتباط با سایر اجزای Stop Token

اجزای اصلی این مکانیزم وظایف متفاوتی دارند:

| نوع                  | مسئولیت                                        |
| -------------------- | ---------------------------------------------- |
| `std::stop_source`   | ایجاد و ارسال درخواست توقف                     |
| `std::stop_token`    | مشاهده و بررسی وضعیت توقف                      |
| `std::stop_callback` | اجرای Callback هنگام درخواست توقف              |
| `std::nostopstate_t` | تعیین ساخت Stop Source بدون Stop State         |
| `std::nostopstate`   | شیء ثابت برای ساخت بدون Stop State             |
| `std::jthread`       | مدیریت Thread همراه با پشتیبانی از توقف تعاونی |

شناخت این تفکیک برای طراحی صحیح سیستم‌های Concurrent اهمیت زیادی دارد.

## مفهوم Stop State

### ساختار داخلی Stop State

هر `std::stop_source` معمولاً به یک Stop State مشترک اشاره می‌کند.

Stop State بخشی از وضعیت هم‌زمانی است که اطلاعات مرتبط با درخواست توقف و Callbackهای ثبت‌شده را نگهداری می‌کند.

مهم‌ترین اطلاعات آن عبارت‌اند از:

- وضعیت درخواست توقف.
- وضعیت اعتبار Stop State.
- Tokenهای متصل به آن.
- Callbackهای ثبت‌شده برای دریافت درخواست توقف.

Stop State یک مفهوم داخلی در طراحی کتابخانه استاندارد است و نباید آن را با یک کلاس عمومی قابل دسترسی اشتباه گرفت.

### ارتباط چند Token با یک Stop State

هنگامی که از یک Stop Source چند Token ایجاد می‌کنیم، تمام آن‌ها به یک Stop State مشترک متصل می‌شوند.

بنابراین، اگر درخواست توقف از طریق یکی از Sourceهای متصل به آن Stop State ارسال شود، تمام Tokenهای مربوطه وضعیت توقف را مشاهده خواهند کرد.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token1 = source.get_token();
    std::stop_token token2 = source.get_token();

    source.request_stop();

    std::cout << token1.stop_requested() << '\n';
    std::cout << token2.stop_requested() << '\n';
}
```

خروجی:

```
1
1
```

در این مثال، هر دو Token به یک Stop State متصل هستند؛ به همین دلیل درخواست توقف برای هر دو قابل مشاهده است.

### طول عمر Stop State

طول عمر Stop State مستقل از طول عمر یک متغیر مشخص از نوع `std::stop_source` است.

این وضعیت تا زمانی حفظ می‌شود که حداقل یکی از Sourceها یا Tokenهای متصل به آن باقی مانده باشد.

به همین دلیل، اگر یک `std::stop_source` از بین برود اما یک `std::stop_token` همچنان به Stop State متصل باشد، Token همچنان می‌تواند وضعیت درخواست توقف را مشاهده کند.

این رفتار برای طراحی سیستم‌هایی که مالکیت درخواست توقف در بخش‌های مختلف برنامه توزیع شده است، اهمیت دارد.

## بررسی Constructorهای std::stop_source

کلاس `std::stop_source` چند Constructor مختلف دارد که هرکدام برای شرایط خاصی طراحی شده‌اند.

### Constructor پیش‌فرض

Constructor پیش‌فرض یک Stop Source معتبر همراه با Stop State جدید ایجاد می‌کند.

```
#include <stop_token>

int main() {
    std::stop_source source;
}
```

پس از ساخت شیء، هنوز درخواست توقفی ثبت نشده است.

بنابراین:

```
#include <cassert>
#include <stop_token>

int main() {
    std::stop_source source;

    assert(source.stop_possible());
    assert(!source.stop_requested());
}
```

در این مثال:

- تابع `()stop_possible` نشان می‌دهد که امکان درخواست توقف وجود دارد.
- تابع `()stop_requested` نشان می‌دهد که هنوز درخواست توقف ثبت نشده است.

### Constructor با std::nostopstate

گاهی لازم است یک Stop Source ایجاد کنیم که هیچ Stop State معتبری نداشته باشد.

برای این منظور می‌توان از `std::nostopstate` استفاده کرد.

```
#include <cassert>
#include <stop_token>

int main() {
    std::stop_source source{std::nostopstate};

    assert(!source.stop_possible());
    assert(!source.stop_requested());
}
```

در این حالت:

- Stop State ایجاد نمی‌شود.
- هیچ Token معتبری از این Source قابل دریافت نیست.
- درخواست توقف امکان‌پذیر نیست.

این قابلیت برای شرایطی مفید است که یک شیء باید ابتدا در وضعیت خالی ساخته شود و در مرحله بعد به یک Stop State متصل شود.

البته در بسیاری از طراحی‌ها استفاده از Constructor پیش‌فرض یا Assignment ساده‌تر و خواناتر است.

### Copy Constructor

کلاس `std::stop_source` قابلیت Copy Construction دارد.

هنگام کپی کردن یک Stop Source، Stop State جدیدی ایجاد نمی‌شود؛ بلکه شیء جدید به همان Stop State قبلی متصل می‌شود.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source1;

    std::stop_source source2 = source1;

    source2.request_stop();

    std::cout << source1.stop_requested() << '\n';
}
```

خروجی:

```
1
```

نکته مهم این است که کپی کردن `std::stop_source` به معنی کپی مستقل وضعیت توقف نیست.

هر دو شیء از یک وضعیت مشترک استفاده می‌کنند.

### Move Constructor

این کلاس قابلیت Move Construction نیز دارد.

```
#include <iostream>
#include <stop_token>
#include <utility>

int main() {
    std::stop_source source1;

    std::stop_source source2 = std::move(source1);

    std::cout << source2.stop_possible() << '\n';
    std::cout << source1.stop_possible() << '\n';
}
```

خروجی:

```
1
0
```

پس از Move Construction، شیء مقصد مالک اتصال قبلی به Stop State می‌شود و شیء مبدأ در وضعیت بدون Stop State قرار می‌گیرد.

در نتیجه، نباید پس از انتقال مالکیت فرض کنیم که شیء مبدأ همچنان یک Stop Source معتبر است.

## بررسی Member Functionهای std::stop_source

کلاس `std::stop_source` چند Member Function مهم دارد که کنترل وضعیت توقف و مدیریت Stop State را ممکن می‌کنند.

### تابع get_token

تابع `get_token` یک `std::stop_token` مرتبط با Stop State فعلی برمی‌گرداند.

امضای مفهومی آن:

```
std::stop_token get_token() const noexcept;
```

مثال:

```
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token = source.get_token();
}
```

ویژگی‌های مهم این تابع:

- مقدار بازگشتی یک Token است که به Stop State مشترک متصل می‌شود.
- فراخوانی آن درخواست توقف جدیدی ایجاد نمی‌کند.
- امکان فراخوانی آن روی Source معتبر و نامعتبر وجود دارد.
- در صورت نبود Stop State معتبر، Token بازگشتی نیز فاقد قابلیت توقف خواهد بود.

این تابع یکی از مهم‌ترین راه‌های انتقال قابلیت مشاهده درخواست توقف به Threadهای دیگر است.

### تابع request_stop

تابع `request_stop` درخواست توقف را برای Stop State مشترک ثبت می‌کند.

امضای تابع:

```
bool request_stop() noexcept;
```

مثال:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    bool first = source.request_stop();
    bool second = source.request_stop();

    std::cout << first << '\n';
    std::cout << second << '\n';
}
```

خروجی:

```
1
0
```

مقدار بازگشتی تابع نشان می‌دهد که آیا این فراخوانی برای اولین بار درخواست توقف را ثبت کرده است یا خیر.

رفتار آن به این صورت است:

- اگر Stop State معتبر باشد و هنوز درخواست توقف ثبت نشده باشد، مقدار `true` برگردانده می‌شود.
- اگر درخواست توقف قبلاً ثبت شده باشد، مقدار `false` برگردانده می‌شود.
- اگر Stop State وجود نداشته باشد، مقدار `false` برگردانده می‌شود.

نکته مهم این است که `request_stop` یک عملیات Idempotent از نظر وضعیت نهایی Stop State محسوب می‌شود؛ یعنی ارسال چند درخواست توقف باعث ایجاد چند وضعیت توقف مستقل نمی‌شود.

پس از ثبت درخواست توقف، وضعیت Stop State به حالت درخواست‌شده تغییر می‌کند و این وضعیت به حالت اولیه بازنمی‌گردد.

### اجرای Callbackها هنگام درخواست توقف

یکی از رفتارهای مهم `request_stop` اجرای Callbackهای ثبت‌شده روی همان Stop State است.

برای مثال:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token = source.get_token();

    std::stop_callback callback(
        token,
        [] {
            std::cout << "Stop callback executed\n";
        }
    );

    source.request_stop();
}
```

خروجی:

```
Stop callback executed
```

نکته مهم این است که اجرای Callbackها ممکن است در همان Threadای انجام شود که `request_stop` را فراخوانی کرده است.

بنابراین، Callbackها نباید عملیات سنگین، Blocking یا عملیات‌هایی انجام دهند که احتمال Deadlock دارند.

همچنین Callbackهای ثبت‌شده می‌توانند توسط چند Thread به‌صورت هم‌زمان با یکدیگر رقابت داشته باشند، اما مکانیزم Stop State از اجرای تکراری یک Callback ثبت‌شده جلوگیری می‌کند.

### تابع stop_requested

این تابع وضعیت درخواست توقف را بررسی می‌کند.

امضای تابع:

```
bool stop_requested() const noexcept;
```

مثال:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::cout << source.stop_requested() << '\n';

    source.request_stop();

    std::cout << source.stop_requested() << '\n';
}
```

خروجی:

```
0
1
```

این تابع در Source و Token رفتار مشابهی از نظر مشاهده وضعیت توقف دارد.

اگر درخواست توقف ثبت شده باشد، مقدار `true` برگردانده می‌شود.

در غیر این صورت، مقدار `false` برگردانده می‌شود.

### تابع stop_possible

تابع `stop_possible` مشخص می‌کند که آیا شیء مورد نظر به یک Stop State معتبر متصل است یا خیر.

امضای تابع:

```
bool stop_possible() const noexcept;
```

مثال:

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source1;
    std::stop_source source2{std::nostopstate};

    std::cout << source1.stop_possible() << '\n';
    std::cout << source2.stop_possible() << '\n';
}
```

خروجی:

```
1
0
```

این تابع به‌ویژه برای تشخیص Sourceهای معتبر و Sourceهای فاقد Stop State کاربرد دارد.

تفاوت آن با `stop_requested` بسیار مهم است:

| تابع               | مفهوم                                                |
| ------------------ | ---------------------------------------------------- |
| `stop_possible()`  | آیا Stop State معتبر و امکان درخواست توقف وجود دارد؟ |
| `stop_requested()` | آیا درخواست توقف قبلاً ثبت شده است؟                  |

ممکن است `stop_possible()` مقدار `true` و `stop_requested()` مقدار `false` برگرداند.

این وضعیت برای یک Stop Source معمولی پیش از درخواست توقف طبیعی است.

## بررسی Assignment Operatorها

### عملگر Copy Assignment

کلاس `std::stop_source` از Copy Assignment پشتیبانی می‌کند.

در این حالت، شیء مقصد به Stop State شیء مبدأ متصل می‌شود.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source1;
    std::stop_source source2;

    source2 = source1;

    source2.request_stop();

    std::cout << source1.stop_requested() << '\n';
}
```

خروجی:

```
1
```

پس از Assignment، دو Source به یک Stop State مشترک متصل هستند.

اگر شیء مقصد پیش از Assignment به Stop State دیگری متصل بوده باشد، اتصال قبلی آن آزاد می‌شود.

### عملگر Move Assignment

Move Assignment نیز برای انتقال اتصال به Stop State استفاده می‌شود.

```
#include <iostream>
#include <stop_token>
#include <utility>

int main() {
    std::stop_source source1;
    std::stop_source source2;

    source2 = std::move(source1);

    std::cout << source2.stop_possible() << '\n';
    std::cout << source1.stop_possible() << '\n';
}
```

خروجی:

```
1
0
```

این عملیات برای انتقال مالکیت Source در طراحی‌های مبتنی بر Move Semantics مفید است.

### عملگر swap

تابع `swap` امکان جابه‌جایی Stop State دو Source را فراهم می‌کند.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source1;
    std::stop_source source2{std::nostopstate};

    source1.swap(source2);

    std::cout << source1.stop_possible() << '\n';
    std::cout << source2.stop_possible() << '\n';
}
```

خروجی:

```
0
1
```

این تابع صرفاً اتصال دو شیء را جابه‌جا می‌کند و وضعیت داخلی Stop Stateها را تغییر نمی‌دهد.

## مقایسه std::stop_source و std::stop_token

این دو نوع ارتباط نزدیکی با یکدیگر دارند، اما مسئولیت آن‌ها یکسان نیست.

### تفاوت در مسئولیت

نوع `std::stop_source` برای ارسال درخواست توقف طراحی شده است.

در مقابل، `std::stop_token` برای مشاهده و بررسی وضعیت توقف استفاده می‌شود.

به‌صورت معمول، Source در اختیار بخش کنترل‌کننده برنامه قرار می‌گیرد و Token به Thread یا Task اجرایی ارسال می‌شود.

```
#include <iostream>
#include <stop_token>
#include <thread>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Working...\n";

        std::this_thread::sleep_for(
            std::chrono::milliseconds(100)
        );
    }

    std::cout << "Worker stopped\n";
}

int main() {
    std::stop_source source;

    std::thread thread(worker, source.get_token());

    std::this_thread::sleep_for(
        std::chrono::milliseconds(500)
    );

    source.request_stop();

    thread.join();
}
```

در این مثال:

- Thread اصلی مالک Stop Source است.
- Thread اجرایی فقط Token دریافت می‌کند.
- Thread اجرایی در حلقه وضعیت توقف را بررسی می‌کند.
- Thread اصلی با ارسال درخواست توقف، عملیات را به‌صورت تعاونی خاتمه می‌دهد.

این الگو جداسازی مناسبی میان منطق کنترل و منطق اجرا ایجاد می‌کند.

### تفاوت قابلیت‌ها

| قابلیت                     | `std::stop_source` | `std::stop_token`            |
| -------------------------- | ------------------ | ---------------------------- |
| ایجاد Stop State           | بله                | خیر                          |
| ارسال درخواست توقف         | بله                | خیر                          |
| بررسی درخواست توقف         | بله                | بله                          |
| بررسی امکان توقف           | بله                | بله                          |
| ایجاد Token                | بله                | خیر                          |
| ثبت Callback               | خیر                | از طریق `std::stop_callback` |
| Copyable                   | بله                | بله                          |
| ارتباط با Stop State مشترک | بله                | بله                          |

این تفکیک باعث می‌شود کدی که صرفاً باید متوقف شدن عملیات را تشخیص دهد، نیازی به دسترسی به قابلیت ارسال درخواست توقف نداشته باشد.

## استفاده از std::stop_source همراه با std::thread

### طراحی توقف تعاونی برای std::thread

در ++C20 می‌توان از `std::stop_source` و `std::stop_token` برای مدیریت توقف Threadهایی استفاده کرد که با `std::thread` ساخته شده‌اند.

برخلاف `std::jthread`، کلاس `std::thread` به‌صورت خودکار Stop Token را به تابع اجرایی منتقل نمی‌کند.

بنابراین باید Token را به‌صورت دستی ارسال کنیم.

```
#include <chrono>
#include <iostream>
#include <stop_token>
#include <thread>

void worker(std::stop_token token) {
    int counter = 0;

    while (!token.stop_requested()) {
        std::cout << "Counter: " << counter++ << '\n';

        std::this_thread::sleep_for(
            std::chrono::milliseconds(200)
        );
    }

    std::cout << "Worker finished\n";
}

int main() {
    std::stop_source source;

    std::thread workerThread(
        worker,
        source.get_token()
    );

    std::this_thread::sleep_for(
        std::chrono::seconds(2)
    );

    source.request_stop();

    workerThread.join();
}
```

این مثال یکی از کاربردهای اصلی `std::stop_source` است.

### چرا استفاده از join ضروری است؟

در مثال قبل، پس از درخواست توقف، Thread اصلی باید منتظر پایان اجرای Thread فرزند بماند.

تابع `request_stop` فقط درخواست توقف را ثبت می‌کند و تضمینی برای پایان فوری Thread ارائه نمی‌دهد.

بنابراین، استفاده از `join` برای هماهنگ‌سازی پایان اجرای Thread اهمیت دارد.

اگر یک شیء `std::thread` هنگام نابودی همچنان Joinable باشد، برنامه با فراخوانی `std::terminate` مواجه خواهد شد.

به همین دلیل، مدیریت صحیح طول عمر Thread باید مستقل از مکانیزم ارسال درخواست توقف طراحی شود.

## استفاده از std::stop_source همراه با std::jthread

### معرفی std::jthread

کلاس `std::jthread` در ++C20 معرفی شد و نسبت به `std::thread` قابلیت‌های پیشرفته‌تری برای مدیریت طول عمر Thread ارائه می‌دهد.

مهم‌ترین ویژگی‌های آن عبارت‌اند از:

- پشتیبانی داخلی از `std::stop_source`.
- امکان دریافت خودکار `std::stop_token` توسط تابع اجرایی.
- درخواست توقف خودکار هنگام تخریب Thread.
- Join خودکار هنگام تخریب Thread.

به همین دلیل، در بسیاری از سناریوهای جدید ++C20 استفاده از `std::jthread` انتخاب مناسبی است.

### مدیریت توقف با Stop Token خودکار

در مثال زیر، `std::jthread` به‌صورت خودکار یک Stop Token به تابع اجرایی منتقل می‌کند.

```
#include <chrono>
#include <iostream>
#include <thread>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Worker is running\n";

        std::this_thread::sleep_for(
            std::chrono::milliseconds(200)
        );
    }

    std::cout << "Worker stopped\n";
}

int main() {
    std::jthread thread(worker);

    std::this_thread::sleep_for(
        std::chrono::seconds(2)
    );

    thread.request_stop();
}
```

در این مثال نیازی به ساخت دستی `std::stop_source` نیست، زیرا `std::jthread` خودش یک Stop Source داخلی دارد.

تابع `request_stop` متعلق به `std::jthread` از همان مکانیزم Stop Token استفاده می‌کند.

### استفاده از Stop Source مستقل در کنار std::jthread

گاهی لازم است کنترل‌کننده‌ای خارج از شیء `std::jthread` درخواست توقف را ارسال کند.

در چنین شرایطی می‌توان یک `std::stop_source` مستقل ایجاد کرد و Token آن را به تابع اجرایی ارسال کرد.

```
#include <chrono>
#include <iostream>
#include <stop_token>
#include <thread>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Processing task\n";

        std::this_thread::sleep_for(
            std::chrono::milliseconds(200)
        );
    }

    std::cout << "Task stopped\n";
}

int main() {
    std::stop_source source;

    std::jthread thread(
        worker,
        source.get_token()
    );

    std::this_thread::sleep_for(
        std::chrono::seconds(2)
    );

    source.request_stop();
}
```

در این حالت، درخواست توقف از طریق Source مستقل ارسال می‌شود.

نکته مهم این است که `std::jthread` همچنان Stop Source داخلی خودش را دارد، اما تابع اجرایی در این مثال Token مربوط به Source مستقل را دریافت می‌کند.

بنابراین، درخواست توقف خودکار `std::jthread` الزاماً باعث توقف تابع اجرایی نخواهد شد؛ زیرا تابع به Token دیگری متصل شده است.

این موضوع یکی از نکات مهم طراحی هنگام ترکیب `std::stop_source` مستقل و `std::jthread` است.

## بررسی std::stop_callback

### نقش Callback در مدیریت توقف

گاهی بررسی مداوم وضعیت توقف در یک حلقه مناسب نیست.

برای مثال، ممکن است Thread در حال انتظار برای دریافت داده، رویداد یا آزاد شدن یک منبع باشد.

در این شرایط، `std::stop_callback` امکان اجرای یک تابع را هنگام ثبت درخواست توقف فراهم می‌کند.

این کلاس نیز بخشی از مکانیزم Stop Token در ++C20 است.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source;

    std::stop_token token = source.get_token();

    std::stop_callback callback(
        token,
        [] {
            std::cout << "Cancellation requested\n";
        }
    );

    source.request_stop();
}
```

در این مثال، با ثبت درخواست توقف، Callback اجرا می‌شود.

### رفتار Callback هنگام ثبت پس از درخواست توقف

اگر Callback پس از ثبت درخواست توقف ساخته شود، Callback بلافاصله در همان Thread سازنده اجرا خواهد شد.

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

این رفتار اهمیت زیادی دارد؛ زیرا نباید فرض کنیم Callback فقط در آینده و پس از ساخت آن اجرا خواهد شد.

### رفتار هنگام نابودی Callback

اگر درخواست توقف هنوز ثبت نشده باشد، نابودی شیء `std::stop_callback` باعث حذف Callback ثبت‌شده می‌شود.

اما اگر Callback در حال اجرا باشد، عملیات تخریب باید با پایان اجرای آن هماهنگ شود.

بنابراین، Destructor این نوع می‌تواند در شرایط خاص Blocking باشد.

همچنین اگر Callback از همان Threadای که در حال تخریب شیء Callback است اجرا شود و به‌صورت مستقیم یا غیرمستقیم باعث تخریب همان شیء شود، رعایت محدودیت‌های استاندارد مربوط به Callback و طول عمر آن اهمیت پیدا می‌کند.

در طراحی Callbackها باید از وابستگی‌های پیچیده و چرخه‌های فراخوانی جلوگیری کرد.

### نکات مهم در طراحی Callback

Callbackها ممکن است در Thread درخواست‌دهنده اجرا شوند.

در نتیجه:

- از اجرای عملیات طولانی در Callback اجتناب کنید.
- از گرفتن Lockهایی که ممکن است در Thread درخواست‌دهنده نگهداری شوند خودداری کنید.
- از انتظار Blocking در Callback پرهیز کنید.
- از دسترسی به اشیایی که ممکن است پیش از Callback نابود شوند جلوگیری کنید.
- برای عملیات Cleanup سنگین، بهتر است فقط یک سیگنال یا Event ثبت شود و Cleanup در Thread مسئول انجام شود.

## استفاده از std::stop_source برای مدیریت چند Thread

### توقف چند Thread با یک Stop Source

یکی از کاربردهای مهم `std::stop_source`، ارسال یک درخواست توقف مشترک به چند Thread است.

از آنجا که چند Token می‌توانند به یک Stop State متصل باشند، می‌توان یک Source را میان چند Thread به اشتراک گذاشت.

```
#include <chrono>
#include <iostream>
#include <stop_token>
#include <thread>
#include <vector>

void worker(
    int id,
    std::stop_token token
) {
    while (!token.stop_requested()) {
        std::cout << "Worker " << id << " running\n";

        std::this_thread::sleep_for(
            std::chrono::milliseconds(200)
        );
    }

    std::cout << "Worker " << id << " stopped\n";
}

int main() {
    std::stop_source source;

    std::vector<std::thread> workers;

    for (int i = 0; i < 4; ++i) {
        workers.emplace_back(
            worker,
            i,
            source.get_token()
        );
    }

    std::this_thread::sleep_for(
        std::chrono::seconds(2)
    );

    source.request_stop();

    for (auto& thread : workers) {
        thread.join();
    }
}
```

در این مثال، چهار Thread به‌صورت هم‌زمان یک عملیات را انجام می‌دهند.

با فراخوانی `request_stop` تمام Threadها درخواست توقف یکسانی را مشاهده می‌کنند.

### مزایای استفاده از Stop Source مشترک

این روش مزایای زیر را دارد:

- نیازی به نگهداری یک متغیر توقف جداگانه برای هر Thread نیست.
- مدیریت توقف متمرکز می‌شود.
- اضافه کردن Threadهای جدید ساده‌تر خواهد بود.
- امکان پیاده‌سازی Shutdown مشترک فراهم می‌شود.
- ارتباط میان کنترل‌کننده و Threadهای اجرایی ساده‌تر می‌شود.

البته اشتراک یک Stop Source به معنی اشتراک تمام وضعیت‌های اجرایی Threadها نیست؛ هر Thread همچنان باید منطق توقف و Cleanup مخصوص خودش را داشته باشد.

## استفاده از std::stop_source در سیستم‌های Asynchronous

### کاربرد در مدیریت Taskها

در سیستم‌هایی که Taskها به‌صورت Asynchronous اجرا می‌شوند، معمولاً یک بخش از برنامه مسئول زمان‌بندی یا مدیریت Taskها است و بخش دیگری عملیات اصلی را انجام می‌دهد.

در این شرایط، `std::stop_source` می‌تواند برای انتشار درخواست لغو استفاده شود.

برای مثال، یک Task پردازش تصویر ممکن است چندین مرحله داشته باشد:

- دریافت داده تصویر.
- Decode کردن داده.
- اعمال فیلترها.
- ذخیره خروجی.

در صورتی که کاربر عملیات را لغو کند، Task می‌تواند در مرز میان مراحل وضعیت Token را بررسی کند.

```
#include <iostream>
#include <stop_token>
#include <vector>

void processImage(
    const std::vector<int>& pixels,
    std::stop_token token
) {
    for (std::size_t i = 0; i < pixels.size(); ++i) {
        if (token.stop_requested()) {
            std::cout << "Image processing cancelled\n";
            return;
        }

        // Process the current pixel
    }

    std::cout << "Image processing completed\n";
}

int main() {
    std::stop_source source;

    std::vector<int> pixels(1000000, 255);

    processImage(pixels, source.get_token());
}
```

در این مثال، عملیات پردازش قابلیت دریافت درخواست توقف را دارد.

البته برای اینکه مثال واقعاً Asynchronous باشد، تابع باید در یک Thread یا Executor اجرا شود.

### تفاوت با std::future و std::async

مکانیزم `std::stop_source` به‌صورت مستقیم جایگزین `std::future` یا `std::async` نیست.

این ابزارها مسئولیت‌های متفاوتی دارند:

| قابلیت             | کاربرد                                |
| ------------------ | ------------------------------------- |
| `std::stop_source` | ارسال درخواست توقف                    |
| `std::stop_token`  | مشاهده درخواست توقف                   |
| `std::future`      | دریافت نتیجه یک عملیات ناهمگام        |
| `std::promise`     | تولید و تنظیم نتیجه برای Future       |
| `std::async`       | اجرای عملیات با مدل Asynchronous مشخص |

در استاندارد ++C20، `std::future` و `std::async` به‌صورت عمومی مکانیزم Stop Token را دریافت نمی‌کنند.

بنابراین، اگر نیاز به لغو یک Task داشته باشیم، باید Stop Token را به‌عنوان بخشی از طراحی Task خودمان منتقل کنیم.

همچنین ثبت درخواست توقف به معنی لغو خودکار عملیات `std::async` نیست.

## بررسی Thread Safety

### ایمنی فراخوانی هم‌زمان

کلاس‌های مرتبط با Stop Token برای استفاده در محیط‌های Multithreaded طراحی شده‌اند.

به‌طور خاص، عملیات مرتبط با Stop State دارای سازوکارهای داخلی لازم برای هماهنگی دسترسی هم‌زمان هستند.

برای مثال، چند Thread می‌توانند به‌صورت هم‌زمان تابع `request_stop` را روی Sourceهای متصل به یک Stop State فراخوانی کنند.

```
#include <iostream>
#include <stop_token>
#include <thread>

int main() {
    std::stop_source source;

    std::thread thread1([&source] {
        std::cout << source.request_stop() << '\n';
    });

    std::thread thread2([&source] {
        std::cout << source.request_stop() << '\n';
    });

    thread1.join();
    thread2.join();
}
```

در این مثال، فقط یکی از فراخوانی‌ها می‌تواند اولین درخواست توقف را ثبت کند.

در نتیجه، یکی از آن‌ها `true` و دیگری `false` برمی‌گرداند.

ترتیب خروجی دو Thread قطعی نیست.

### Thread Safety با Data Race متفاوت است

ایمن بودن Stop State به معنی Thread-Safe بودن تمام داده‌هایی نیست که در کنار آن استفاده می‌شوند.

برای مثال:

```
#include <stop_token>

int sharedCounter = 0;

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        ++sharedCounter;
    }
}
```

در این مثال، اگر چند Thread به‌صورت هم‌زمان تابع `worker` را اجرا کنند، دسترسی هم‌زمان به `sharedCounter` می‌تواند Data Race ایجاد کند.

مکانیزم Stop Token از چنین دسترسی‌هایی محافظت نمی‌کند.

برای مدیریت صحیح داده‌های اشتراکی باید از ابزارهایی مانند موارد زیر استفاده کرد:

- `std::mutex`
- `std::atomic`
- `std::shared_mutex`
- سایر مکانیزم‌های هم‌زمانی متناسب با مسئله

### استفاده از چند Stop Source مشترک

اگر چند `std::stop_source` به یک Stop State متصل باشند، تمام آن‌ها می‌توانند درخواست توقف را ارسال کنند.

```
#include <iostream>
#include <stop_token>

int main() {
    std::stop_source source1;
    std::stop_source source2 = source1;

    source2.request_stop();

    std::cout << source1.stop_requested() << '\n';
}
```

خروجی:

```
1
```

در این حالت، Copy کردن Source باعث ایجاد دو کانال مستقل توقف نمی‌شود؛ هر دو Source به یک Stop State متصل هستند.

## مدل حافظه و Memory Ordering

### تضمین‌های هم‌زمانی

مکانیزم Stop Token تنها یک متغیر Boolean ساده نیست.

عملیات مربوط به Stop State باید به‌صورت Thread-Safe پیاده‌سازی شوند و رفتار آن‌ها با قواعد Memory Model استاندارد هماهنگ است.

در استاندارد، ثبت درخواست توقف و مشاهده آن از طریق Token دارای تضمین‌های هم‌زمانی مشخصی هستند.

به‌ویژه، ثبت درخواست توقف از طریق عملیات مربوط به Stop State با مشاهده درخواست توقف در Threadهای دیگر هماهنگ می‌شود.

این ویژگی به برنامه اجازه می‌دهد از Stop Token به‌عنوان مکانیزم انتشار درخواست توقف استفاده کند.

بااین‌حال، نباید تصور کرد که هر نوع دسترسی به داده‌های اشتراکی صرفاً به دلیل استفاده از Stop Token ایمن خواهد بود.

اگر داده‌ای خارج از مکانیزم مناسب Synchronization به‌صورت هم‌زمان تغییر کند، همچنان احتمال Data Race وجود دارد.

### چرا Stop Token جایگزین Atomic نیست؟

یک `std::atomic<bool>` می‌تواند وضعیت ساده توقف را نمایش دهد.

اما Stop Token قابلیت‌های بیشتری ارائه می‌دهد:

- مدیریت طول عمر وضعیت توقف.
- اشتراک وضعیت میان چند Source و Token.
- ثبت Callback.
- هماهنگی با `std::jthread`.
- امکان استفاده در مکانیزم‌های Waiting سازگار با Stop Token.

بنابراین، مقایسه آن با یک Atomic Boolean باید بر اساس نیاز واقعی برنامه انجام شود.

## استفاده از Stop Token در عملیات Waiting

### مشکل حلقه‌های انتظار

یکی از مشکلات رایج در برنامه‌های Concurrent، انتظار برای دریافت داده یا رخ دادن یک رویداد است.

برای مثال:

```
while (!token.stop_requested()) {
    if (hasData()) {
        processData();
    }
}
```

این الگو یک Busy Loop ایجاد می‌کند و ممکن است مقدار زیادی از CPU را مصرف کند.

استفاده از `sleep_for` نیز همیشه راه‌حل مناسبی نیست؛ زیرا توقف ممکن است با تأخیر تشخیص داده شود.

### استفاده از condition_variable_any

در ++C20 کلاس `std::condition_variable_any` قابلیت انتظار همراه با Stop Token را ارائه می‌کند.

یکی از Overloadهای تابع `wait` امکان دریافت Stop Token را فراهم می‌کند.

مثال:

```
#include <chrono>
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <stop_token>
#include <thread>

int main() {
    std::mutex mutex;
    std::condition_variable_any cv;

    bool ready = false;

    std::jthread worker(
        [&](std::stop_token token) {
            std::unique_lock lock(mutex);

            bool completed = cv.wait(
                lock,
                token,
                [&] {
                    return ready;
                }
            );

            if (completed) {
                std::cout << "Work is ready\n";
            } else {
                std::cout << "Waiting cancelled\n";
            }
        }
    );

    std::this_thread::sleep_for(
        std::chrono::seconds(1)
    );

    worker.request_stop();
}
```

در این مثال، Thread می‌تواند هنگام انتظار نیز درخواست توقف را دریافت کند.

نکته مهم این است که این قابلیت به `std::condition_variable_any` تعلق دارد و نباید آن را با `std::condition_variable` یکسان دانست.

### تفاوت Predicate و Stop Request

در تابع `wait` دارای Stop Token، دو شرط متفاوت وجود دارد:

- Predicate مشخص می‌کند که آیا رویداد مورد انتظار اتفاق افتاده است.
- Stop Token مشخص می‌کند که آیا عملیات انتظار باید به دلیل درخواست توقف لغو شود.

مقدار بازگشتی تابع مشخص می‌کند که Predicate در نهایت برقرار شده است یا خیر.

اگر درخواست توقف ثبت شود، انتظار می‌تواند خاتمه پیدا کند، اما این موضوع لزوماً به معنی برقرار شدن Predicate نیست.

همچنین اگر Predicate هم‌زمان با درخواست توقف برقرار شود، نتیجه تابع باید بر اساس وضعیت نهایی Predicate بررسی شود.

## استفاده از Stop Source برای طراحی Shutdown

### مدیریت خاموش شدن کنترل‌شده برنامه

یکی از کاربردهای مهم `std::stop_source`، مدیریت Shutdown در برنامه‌هایی است که چند Thread دائماً در حال فعالیت هستند.

برای مثال، یک Server ممکن است چند Worker داشته باشد که درخواست‌های ورودی را پردازش می‌کنند.

هنگام Shutdown باید:

- از پذیرش کارهای جدید جلوگیری شود.
- به Workerها درخواست توقف ارسال شود.
- عملیات جاری در صورت امکان به‌صورت کنترل‌شده خاتمه پیدا کند.
- منابع آزاد شوند.
- Threadها به پایان برسند.

نمونه ساده:

```
#include <chrono>
#include <iostream>
#include <stop_token>
#include <thread>
#include <vector>

class WorkerPool {
private:
    std::stop_source stopSource;
    std::vector<std::thread> workers;

public:
    void start(int count) {
        for (int i = 0; i < count; ++i) {
            workers.emplace_back(
                [this, i](std::stop_token token) {
                    while (!token.stop_requested()) {
                        std::cout
                            << "Worker "
                            << i
                            << " processing\n";

                        std::this_thread::sleep_for(
                            std::chrono::milliseconds(200)
                        );
                    }

                    std::cout
                        << "Worker "
                        << i
                        << " stopped\n";
                },
                stopSource.get_token()
            );
        }
    }

    void shutdown() {
        stopSource.request_stop();

        for (auto& worker : workers) {
            if (worker.joinable()) {
                worker.join();
            }
        }
    }

    ~WorkerPool() {
        shutdown();
    }
};

int main() {
    WorkerPool pool;

    pool.start(4);

    std::this_thread::sleep_for(
        std::chrono::seconds(2)
    );

    pool.shutdown();
}
```

در این مثال، یک Stop Source مشترک درخواست توقف را به تمام Workerها ارسال می‌کند.

تابع `shutdown` ابتدا درخواست توقف را ثبت می‌کند و سپس منتظر پایان تمام Threadها می‌ماند.

نکته طراحی: در یک پیاده‌سازی واقعی باید فراخوانی‌های هم‌زمان `start` و `shutdown` نیز مدیریت شوند و دسترسی به Container مربوط به Threadها با Synchronization مناسب محافظت شود. همچنین باید از فراخوانی `shutdown` از داخل یکی از Workerهای همان Pool جلوگیری کرد؛ زیرا Join کردن Thread جاری صحیح نیست.

## محدودیت‌های std::stop_source

### توقف فوری تضمین نمی‌شود

مهم‌ترین محدودیت این مکانیزم این است که درخواست توقف فقط یک سیگنال منطقی است.

اگر Thread درخواست توقف را بررسی نکند، ممکن است اجرای آن تا مدت نامحدودی ادامه پیدا کند.

برای مثال:

```
#include <stop_token>

void worker(std::stop_token token) {
    while (true) {
        // Long-running operation without cancellation checks
    }
}
```

در این مثال، ثبت درخواست توقف هیچ تأثیری بر حلقه ندارد.

بنابراین، طراحی عملیات قابل لغو باید از ابتدا با نقاط مشخص بررسی توقف انجام شود.

### عملیات Blocking ممکن است قابل لغو نباشند

فرض کنید Thread داخل یک تابع Blocking منتظر دریافت داده از یک API خارجی است.

اگر آن API هیچ مکانیزمی برای لغو انتظار نداشته باشد، صرفاً ارسال درخواست توقف باعث بیدار شدن Thread نخواهد شد.

برای حل این مشکل باید از روش‌هایی مانند موارد زیر استفاده کرد:

- APIهای Waiting سازگار با Stop Token.
- `std::condition_variable_any`.
- مکانیزم‌های Notification.
- Timeoutهای محدود.
- APIهای لغوپذیر سیستم‌عامل یا کتابخانه مورد استفاده.

### امکان بازنشانی Stop State وجود ندارد

پس از ثبت درخواست توقف، وضعیت Stop State به حالت اولیه بازنمی‌گردد.

بنابراین، برای شروع یک چرخه توقف جدید باید یک Stop Source جدید ایجاد شود.

```
#include <stop_token>

int main() {
    std::stop_source source1;

    source1.request_stop();

    std::stop_source source2;
}
```

در این مثال، Source دوم یک Stop State مستقل دارد.

### مدیریت Callbackها نیازمند دقت است

اجرای Callbackها در مسیر ارسال درخواست توقف می‌تواند باعث افزایش زمان اجرای `request_stop` شود.

بنابراین، اگر Callback عملیات سنگین یا Blocking انجام دهد، Thread درخواست‌دهنده نیز ممکن است برای مدت زیادی درگیر شود.

این مسئله به‌خصوص در برنامه‌هایی که Latency اهمیت زیادی دارد، مهم است.

### توقف یک عملیات به معنی Rollback نیست

درخواست توقف صرفاً به برنامه اعلام می‌کند که عملیات باید متوقف شود.

این مکانیزم هیچ تضمینی برای بازگرداندن تغییرات قبلی ارائه نمی‌دهد.

برای مثال، اگر یک Task بخشی از داده‌ها را در Database ذخیره کرده باشد، درخواست توقف باعث Rollback خودکار تراکنش نمی‌شود.

مدیریت Transaction، Cleanup و Consistency همچنان مسئولیت برنامه است.

## اشتباهات رایج در استفاده از std::stop_source

### اشتباه در تصور توقف اجباری Thread

یکی از اشتباهات رایج این است که تصور کنیم فراخوانی `request_stop` باعث پایان فوری Thread می‌شود.

درحالی‌که این تابع صرفاً درخواست توقف را ثبت می‌کند.

Thread باید خودش این درخواست را دریافت و پردازش کند.

### اشتباه در استفاده از Sourceهای مستقل

در مثال زیر دو Stop Source مستقل ایجاد شده‌اند:

```
#include <stop_token>

int main() {
    std::stop_source source1;
    std::stop_source source2;

    source1.request_stop();

    std::stop_token token = source2.get_token();
}
```

در این مثال، درخواست توقف `source1` هیچ تأثیری بر Token مربوط به `source2` ندارد.

اگر هدف اشتراک وضعیت توقف باشد، باید Source را Copy کرد یا Tokenها را از یک Source مشترک دریافت کرد.

### اشتباه در فرض مستقل بودن Copy

گاهی توسعه‌دهندگان تصور می‌کنند Copy کردن یک `std::stop_source` باعث ایجاد یک Stop State مستقل می‌شود.

این تصور اشتباه است.

Copy فقط اتصال جدیدی به همان Stop State ایجاد می‌کند.

برای ساخت وضعیت توقف مستقل باید یک Stop Source جدید با Constructor پیش‌فرض ایجاد شود.

### اشتباه در نادیده گرفتن وضعیت Moved-From

پس از انتقال یک Source با `std::move` نباید فرض کرد که شیء مبدأ همچنان به Stop State قبلی متصل است.

شیء مبدأ در وضعیت بدون Stop State قرار می‌گیرد.

بنابراین، استفاده از آن باید با توجه به وضعیت جدیدش انجام شود.

### اشتباه در انجام عملیات سنگین داخل Callback

از آنجا که Callback ممکن است در Thread درخواست‌دهنده اجرا شود، اجرای عملیات سنگین داخل آن می‌تواند باعث تأخیر غیرمنتظره شود.

بهتر است Callback صرفاً یک Notification سبک ایجاد کند.

### اشتباه در نادیده گرفتن طول عمر داده‌ها

اگر Callback به اشیایی ارجاع دهد که پیش از اجرای Callback نابود می‌شوند، ممکن است Dangling Reference یا Undefined Behavior ایجاد شود.

برای جلوگیری از این مشکل باید طول عمر اشیای مورد استفاده Callback به‌درستی مدیریت شود.

## **مقایسه&#x20;**`std::stop_source`**&#x20;با روش‌های قدیمی**

در نسخه‌های پیش از C++20، برای متوقف‌کردن یک Thread معمولاً از متغیرهای اشتراکی، `std::atomic<bool>`، ترکیب `std::mutex` و `std::condition_variable` یا سازوکارهای اختصاصی سیستم‌عامل استفاده می‌شد.

مشکل اصلی این روش‌ها این بود که برنامه‌نویس باید خودش پروتکل توقف را طراحی می‌کرد: وضعیت توقف را نگه می‌داشت، تغییر آن را به Threadهای دیگر اطلاع می‌داد و مراقب رقابت‌های داده‌ای و بیدارشدن Threadهای منتظر می‌بود.

`std::stop_source` و `std::stop_token` در C++20 یک سازوکار استاندارد برای **لغو مشارکتی (Cooperative Cancellation)** فراهم می‌کنند. در این مدل، یک بخش از برنامه درخواست توقف می‌دهد و بخش‌های دیگر با بررسی Token یا ثبت Callback به آن واکنش نشان می‌دهند. توقف اجباری نیست؛ کد اجراشونده باید خودش به درخواست توقف احترام بگذارد.

### **۱. مقایسه با&#x20;**`std::atomic<bool>`

روش ساده و رایج قدیمی، استفاده از یک متغیر اتمیک به‌عنوان پرچم توقف بود:

```
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<bool> stopRequested{false};

void worker()
{
    while (!stopRequested.load())
    {
        // Perform a unit of work
    }

    std::cout << "Worker stopped\n";
}

int main()
{
    std::thread t(worker);

    // Request the worker to stop
    stopRequested.store(true);

    t.join();
}
```

این روش ساده است، اما چند محدودیت دارد:

- باید متغیر اتمیک را به‌صورت دستی مدیریت کنید.
- برای چند Worker معمولاً باید پرچم را به‌درستی بین همه آن‌ها به اشتراک بگذارید.
- سازوکار استانداردی برای ثبت Callback هنگام درخواست توقف وجود ندارد.
- اگر Thread در یک عملیات مسدودکننده منتظر باشد، تغییر پرچم لزوماً آن را بیدار نمی‌کند.

همین مثال با `std::stop_source` و `std::stop_token`:

```
#include <stop_token>
#include <thread>
#include <iostream>

void worker(std::stop_token token)
{
    while (!token.stop_requested())
    {
        // Perform a unit of work
    }

    std::cout << "Worker stopped\n";
}

int main()
{
    std::stop_source source;

    std::thread t(worker, source.get_token());

    // Request cooperative cancellation
    source.request_stop();

    t.join();
}
```

مزیت مهم این طراحی، جداسازی مسئولیت‌هاست:

- `std::stop_source` مسئول درخواست توقف است.
- `std::stop_token` برای مشاهده وضعیت توقف در اختیار Worker قرار می‌گیرد.
- هر تعداد Token که از یک Source گرفته شوند، وضعیت توقف مشترکی را مشاهده می‌کنند.

**نکته:** `std::stop_source` جایگزین کامل `std::atomic<bool>` نیست. اگر فقط به یک پرچم اتمیک عمومی نیاز دارید و به قابلیت‌های لغو مشارکتی احتیاجی ندارید، استفاده از Atomic همچنان می‌تواند مناسب باشد.

### **۲. مقایسه با&#x20;**`std::mutex`**&#x20;و&#x20;**`std::condition_variable`

در بعضی برنامه‌ها Worker به‌جای اجرای مداوم، منتظر دریافت داده یا یک رویداد می‌ماند. در این حالت، تغییر یک پرچم توقف به‌تنهایی کافی نیست؛ Thread باید از حالت انتظار خارج شود.

روش سنتی معمولاً ترکیبی از Mutex، Condition Variable و یک پرچم توقف است:

```
#include <mutex>
#include <condition_variable>
#include <thread>

std::mutex mtx;
std::condition_variable cv;
bool stopRequested = false;

void worker()
{
    std::unique_lock<std::mutex> lock(mtx);

    cv.wait(lock, [] {
        return stopRequested;
    });

    // Continue shutdown
}

int main()
{
    std::thread t(worker);

    {
        std::lock_guard<std::mutex> lock(mtx);
        stopRequested = true;
    }

    cv.notify_all();

    t.join();
}
```

در این روش باید به چند نکته توجه کرد:

- تغییر پرچم باید با همان Mutex محافظت شود.
- پس از تغییر وضعیت باید Threadهای منتظر را بیدار کنید.
- شرط انتظار باید به‌صورت Predicate بررسی شود تا بیدارشدن‌های کاذب مشکلی ایجاد نکنند.
- اگر چند نوع رویداد وجود داشته باشد، مدیریت شرط‌های انتظار پیچیده‌تر می‌شود.

در C++20 می‌توان از `std::condition_variable_any` استفاده کرد تا انتظار به Stop Token متصل شود:

```
#include <condition_variable>
#include <mutex>
#include <stop_token>
#include <thread>
#include <iostream>

std::mutex mtx;
std::condition_variable_any cv;

void worker(std::stop_token token)
{
    std::unique_lock<std::mutex> lock(mtx);

    cv.wait(lock, token, [] {
        return false;
    });

    if (token.stop_requested())
    {
        std::cout << "Cancellation received\n";
    }
}

int main()
{
    std::jthread t(worker);

    // Request stop and wake the waiting worker
    t.request_stop();
}
```

در این مثال، انتظار با Stop Token یکپارچه شده است. با درخواست توقف، انتظار قابل لغو می‌شود و نیازی نیست برای این مورد یک پرچم جداگانه و فراخوانی دستی `notify_all()` بنویسید.

البته این قابلیت به معنای جایگزینی کامل `condition_variable` نیست؛ برای انتظار روی رویدادهای معمولی همچنان از Predicate و Notification استفاده می‌شود.

### **۳. مقایسه با&#x20;**`pthread_cancel`

در محیط POSIX، تابع `pthread_cancel()` برای درخواست لغو Thread وجود دارد. این روش با لغو مشارکتی C++20 تفاوت مهمی دارد.

`pthread_cancel()` می‌تواند Thread هدف را در نقاط مشخصی از اجرای آن لغو کند و رفتار آن به نوع لغو (Deferred یا Asynchronous Cancellation) وابسته است.

در مقابل، `std::stop_source`:

- Thread را به‌صورت اجباری متوقف نمی‌کند.
- فقط درخواست توقف را ثبت می‌کند.
- به کد Worker اجازه می‌دهد در نقاط امن و مناسب به درخواست واکنش نشان دهد.
- به سازوکارهای اختصاصی سیستم‌عامل وابسته نیست.

لغو اجباری Thread ممکن است در شرایط نامناسب باعث باقی‌ماندن Mutex قفل‌شده، منابع آزاد‌نشده یا وضعیت ناسازگار داده‌ها شود. مدل مشارکتی C++20 برای طراحی قابل‌کنترل‌تر چرخه عمر Threadها مناسب‌تر است.
| مشاهده وضعیت توقف      | با `load()`     | با بررسی پرچم                       | با `stop_requested()`                               |
| ---------------------- | --------------- | ----------------------------------- | --------------------------------------------------- |
| ثبت Callback برای توقف | ندارد           | باید دستی پیاده‌سازی شود            | با `std::stop_callback`                             |
| لغو انتظار مسدودکننده  | به‌تنهایی ندارد | با `notify_one()` یا `notify_all()` | با قابلیت‌های سازگار مانند `condition_variable_any` |
| توقف اجباری Thread     | خیر             | خیر                                 | خیر                                                 |
| پشتیبانی استاندارد C++ | از C++11        | از C++11                            | از C++20                                            |

### **۵. نکات پیشرفته در طراحی با&#x20;**`std::stop_source`

#### **۵.۱. لغو سلسله‌مراتبی (Hierarchical Cancellation)**

در برنامه‌های بزرگ ممکن است چند زیرسیستم وجود داشته باشند که هرکدام چند Worker دارند.

برای مثال:

- یک سرور چند Connection دارد.
- هر Connection چند عملیات هم‌زمان اجرا می‌کند.
- با بسته‌شدن سرور باید تمام عملیات مرتبط با آن متوقف شوند.

می‌توان برای هر سطح یک Stop Source مستقل داشت و با Callback، درخواست توقف سطح بالاتر را به سطح پایین‌تر منتقل کرد.

```
#include <stop_token>
#include <thread>
#include <iostream>

int main()
{
    std::stop_source serverSource;
    std::stop_source connectionSource;

    std::stop_callback connectionCallback(
        serverSource.get_token(),
        [&connectionSource] {
            connectionSource.request_stop();
        }
    );

    std::jthread worker(
        [token = connectionSource.get_token()](std::stop_token)
        {
            while (!token.stop_requested())
            {
                // Process connection work
            }

            std::cout << "Connection worker stopped\n";
        }
    );

    // Stop the server and propagate cancellation
    serverSource.request_stop();
}
```

در این طراحی، توقف سرور به Connection منتقل می‌شود و Worker با Token مربوط به Connection از توقف مطلع می‌شود.

نکته مهم این است که `std::stop_source` به‌صورت خودکار رابطه والد و فرزندی ایجاد نمی‌کند؛ انتشار درخواست توقف باید صریحاً پیاده‌سازی شود.

#### **۵.۲. ترکیب چند Stop Source**

گاهی یک عملیات باید در صورت وقوع هرکدام از چند اتفاق متوقف شود:

- کاربر عملیات را لغو کند.
- سرور در حال خاموش‌شدن باشد.
- مهلت زمانی عملیات تمام شود.

`std::stop_token` به‌صورت مستقیم قابلیت ترکیب چند Token را ندارد؛ اما می‌توان با Callbackها یک Token تجمیعی ساخت.

```
#include <stop_token>
#include <thread>

int main()
{
    std::stop_source userSource;
    std::stop_source serverSource;
    std::stop_source combinedSource;

    std::stop_callback userCallback(
        userSource.get_token(),
        [&combinedSource] {
            combinedSource.request_stop();
        }
    );

    std::stop_callback serverCallback(
        serverSource.get_token(),
        [&combinedSource] {
            combinedSource.request_stop();
        }
    );

    // Either source can stop the combined operation
    serverSource.request_stop();

    return combinedSource.stop_requested() ? 0 : 1;
}
```

در این مثال، درخواست توقف از هرکدام از دو Source باعث توقف Source تجمیعی می‌شود.

این روش برای سیستم‌هایی مانند Task Scheduler، Server و Pipelineهای پردازشی مفید است.

#### **۵.۳. رقابت هنگام ثبت Callback**

یکی از ویژگی‌های مهم `std::stop_callback` رفتار آن در شرایط رقابتی است.

فرض کنید یک Thread در حال اجرای `request_stop()` باشد و Thread دیگری هم‌زمان Callback ثبت کند.

رفتار استاندارد به این صورت است:

- اگر Callback پیش از درخواست توقف ثبت شده باشد، با درخواست توقف اجرا می‌شود.
- اگر توقف قبلاً درخواست شده باشد، Callback هنگام ثبت، بلافاصله و در همان Thread ثبت‌کننده اجرا می‌شود.
- Callback برای یک ثبت مشخص حداکثر یک بار اجرا می‌شود.

بنابراین نباید فرض کنید Callback همیشه در Thread درخواست‌کننده توقف اجرا خواهد شد.

همچنین Callback ممکن است در حین ثبت آن اجرا شود؛ پس بهتر است از ثبت Callback در شرایطی که قفل‌های حساس در اختیار دارید، خودداری کنید.

#### **۵.۴. جلوگیری از Deadlock و Reentrancy**

Callbackها می‌توانند در Thread درخواست‌کننده توقف اجرا شوند. بنابراین اگر Callback یک Mutex را قفل کند، باید مراقب باشید که همان Mutex در Thread درخواست‌کننده از قبل قفل نشده باشد.

مثال نامناسب:

```
#include <mutex>
#include <stop_token>

std::mutex mtx;

void example(std::stop_token token)
{
    std::lock_guard<std::mutex> lock(mtx);

    std::stop_callback callback(
        token,
        [] {
            std::lock_guard<std::mutex> innerLock(mtx);

            // Perform cleanup
        }
    );

    // Other operations
}
```

اگر Token از قبل متوقف شده باشد، Callback هنگام ثبت اجرا می‌شود. در نتیجه تلاش برای قفل‌کردن دوباره Mutex غیر بازگشتی می‌تواند باعث Deadlock شود.

راهکارهای مناسب:

- Callback را بیرون از محدوده قفل‌های حساس ثبت کنید.
- عملیات Callback را کوتاه و قابل پیش‌بینی نگه دارید.
- از فراخوانی توابع پیچیده یا کدهایی که ممکن است دوباره وارد همان مسیر شوند، اجتناب کنید.
- در صورت نیاز، عملیات پاک‌سازی را به یک Worker دیگر واگذار کنید.

### **۶. بررسی API و مشخصات استاندارد**

کلاس `std::stop_source` در هدر زیر تعریف شده است:

```
#include <stop_token>
```

اعضای اصلی آن عبارت‌اند از:

| عضو                            | کاربرد                             |
| ------------------------------ | ---------------------------------- |
| `std::stop_source()`           | ساخت Source جدید                   |
| `std::stop_source(token)`      | ساخت Source از Token موجود         |
| `get_token()`                  | دریافت Token                       |
| `stop_requested()`             | بررسی درخواست توقف                 |
| `stop_possible()`              | بررسی امکان درخواست توقف           |
| `request_stop()`               | درخواست توقف                       |
| `stop_source::stop_possible()` | بررسی وجود Stop State قابل استفاده |
| `swap()`                       | جابه‌جایی وضعیت دو Source          |
| `operator bool()`              | بررسی معتبر بودن Source            |

دو مفهوم مهم را نباید با یکدیگر اشتباه گرفت:

`stop_possible()`

مشخص می‌کند که Source دارای Stop State قابل استفاده برای درخواست توقف است یا خیر.

`stop_requested()`

مشخص می‌کند که آیا درخواست توقف قبلاً ثبت شده است یا خیر.

بنابراین ممکن است:

```
source.stop_possible() == true
source.stop_requested() == false
```

یعنی امکان درخواست توقف وجود دارد، اما هنوز توقف درخواست نشده است.

همچنین اگر Stop State وجود نداشته باشد، Token مربوطه نمی‌تواند درخواست توقف را مشاهده کند.

### **۷. سازگاری با کامپایلرها**

برای استفاده از `std::stop_source` باید کامپایلر و کتابخانه استاندارد از C++20 پشتیبانی کنند.

نمونه کامپایل با GCC:

```
g++ -std=c++20 -pthread main.cpp -o app
```

نمونه کامپایل با Clang:

```
clang++ -std=c++20 -pthread main.cpp -o app
```

نکته: صرفاً فعال‌کردن گزینه `-std=c++20` کافی نیست؛ نسخه کتابخانه استاندارد نیز باید پیاده‌سازی این قابلیت را داشته باشد.

### **۸. جمع‌بندی**

`std::stop_source` یکی از اجزای مهم مدل لغو مشارکتی در C++20 است. این کلاس به برنامه اجازه می‌دهد بدون متوقف‌کردن اجباری Threadها، درخواست توقف را به شکلی استاندارد و ایمن منتشر کند.

مهم‌ترین مزیت‌های آن عبارت‌اند از:

1. جداسازی مسئولیت درخواست توقف از مشاهده آن.
2. امکان اشتراک یک وضعیت توقف بین چند Thread.
3. پشتیبانی از Callback با `std::stop_callback`.
4. امکان انتشار درخواست توقف بین چند زیرسیستم.
5. یکپارچگی با `std::jthread` و برخی سازوکارهای انتظار در C++20.
6. کاهش نیاز به پیاده‌سازی دستی پروتکل‌های لغو.

در نهایت، باید به خاطر داشت که `std::stop_source` یک ابزار برای **درخواست توقف** است، نه ابزاری برای **توقف اجباری Thread**.

طراحی صحیح همچنان نیازمند آن است که Workerها در نقاط مناسب درخواست توقف را بررسی کنند، منابع خود را آزاد کنند و با یک روش مشخص از تابع یا Thread خارج شوند.
---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>