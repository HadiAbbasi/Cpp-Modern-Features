<div align="center">

[🇺🇸 English](../../en/cpp20/std_jthread.md) | [🇮🇷 فارسی](./std_jthread.md)

</div>

---

# راهنمای جامع `std::jthread` در ++C20

کلاس `std::jthread` یکی از مهم‌ترین قابلیت‌های هم‌زمانی (Concurrency) در استاندارد ++C20 است که برای مدیریت ایمن و خودکار Threadها معرفی شد. این کلاس جایگزینی مدرن برای `std::thread` نیست، بلکه نسخه‌ای بهبود‌یافته از آن است که دو مشکل اساسی را حل می‌کند:

- مدیریت خودکار پایان Thread از طریق RAII
- پشتیبانی داخلی از Cooperative Cancellation با استفاده از `stop_token`

در عمل، `std::jthread` بسیاری از خطاهای رایج مرتبط با `std::thread` را حذف می‌کند و نوشتن برنامه‌های چندریسمانی را ایمن‌تر و خواناتر می‌سازد.

## مقدمه

از زمان معرفی `std::thread` در ++C11، توسعه‌دهندگان توانستند به صورت استاندارد از Threadها استفاده کنند. با این حال، طراحی `std::thread` یک مسئولیت مهم را بر عهده برنامه‌نویس می‌گذاشت:

قبل از نابود شدن شیء Thread باید یکی از توابع `join()` یا `detach()` فراخوانی می‌شد.

در غیر این صورت برنامه با `std::terminate` متوقف می‌شد.

نمونه‌ای از این مشکل:

```
#include <thread>

int main() {
    std::thread t([]{});

    // Destructor calls std::terminate
}
```

این رفتار باعث می‌شد بسیاری از باگ‌ها تنها در زمان اجرا آشکار شوند.

برای حل این مشکل، استاندارد ++C20 کلاس `std::jthread` را معرفی کرد که هنگام نابود شدن، به صورت خودکار درخواست توقف ارسال می‌کند و سپس Thread را `join` می‌کند.

## مفهوم RAII در مدیریت Thread

فلسفه طراحی `std::jthread` بر پایه RAII است.

مفهوم RAII به این معناست که:

- هنگام ساخت شیء، منبع ایجاد می‌شود.
- هنگام نابود شدن شیء، منبع آزاد می‌شود.

همان‌طور که `std::lock_guard` قفل را آزاد می‌کند، `std::jthread` نیز Thread را به صورت ایمن خاتمه می‌دهد.

مقایسه رفتار:

| رفتار                 | `std::thread` | `std::jthread` |
| --------------------- | ------------- | -------------- |
| نیاز به `join()` دستی | بله           | خیر            |
| خطر `std::terminate`  | بله           | خیر            |
| RAII                  | خیر           | بله            |
| توقف همکاری‌شده       | خیر           | بله            |

## معرفی `std::jthread`

برای استفاده:

```
#include <thread>
```

ساده‌ترین نمونه:

```
#include <iostream>
#include <thread>

int main() {
    std::jthread worker([]{
        std::cout << "Worker running\n";
    });
}
```

در پایان محدوده:

- Destructor اجرا می‌شود.
- درخواست توقف ارسال می‌شود.
- سپس `join()` انجام می‌شود.

## نحوه ساخت `std::jthread`

### ساخت با Lambda

```
std::jthread worker([]{
    // work
});
```

### ساخت با تابع معمولی

```
void task() {
}

std::jthread worker(task);
```

### ارسال پارامتر

```
void print(int value) {
}

std::jthread worker(print, 42);
```

## رفتار Destructor

مهم‌ترین تفاوت `std::jthread` در Destructor آن است.

تقریباً معادل زیر عمل می‌کند:

```
if (joinable()) {
    request_stop();
    join();
}
```

این یعنی اگر Thread هنوز فعال باشد:

- ابتدا درخواست توقف ارسال می‌شود.
- سپس منتظر پایان Thread می‌ماند.

به همین دلیل نابود شدن شیء معمولاً ایمن است.

## مفهوم Cooperative Cancellation

یکی از بزرگ‌ترین قابلیت‌های `std::jthread`، سیستم توقف همکاری‌شده است.

نکته مهم آن است که Thread به زور متوقف نمی‌شود.

در عوض:

- درخواست توقف ارسال می‌شود.
- خود Thread تصمیم می‌گیرد چه زمانی خاتمه یابد.

این طراحی از مشکلات خطرناک توقف اجباری جلوگیری می‌کند.

## معرفی `stop_token`

زمانی که تابع Thread اولین پارامترش `std::stop_token` باشد، `std::jthread` آن را به صورت خودکار ارسال می‌کند.

نمونه:

```
#include <chrono>
#include <iostream>
#include <thread>

using namespace std::chrono_literals;

void worker(std::stop_token token) {

    while (!token.stop_requested()) {
        std::cout << "Working...\n";
        std::this_thread::sleep_for(300ms);
    }

    std::cout << "Stopping\n";
}

int main() {
    std::jthread t(worker);

    std::this_thread::sleep_for(1s);
}
```

پس از خروج از `main`:

- درخواست توقف ارسال می‌شود.
- حلقه پایان می‌یابد.
- Thread خاتمه پیدا می‌کند.

## درخواست توقف دستی

علاوه بر Destructor می‌توان به صورت دستی نیز توقف را درخواست کرد.

```
t.request_stop();
```

نمونه کامل:

```
#include <chrono>
#include <iostream>
#include <thread>

using namespace std::chrono_literals;

void worker(std::stop_token token) {

    while (!token.stop_requested()) {
        std::cout << "Running\n";
        std::this_thread::sleep_for(200ms);
    }
}

int main() {

    std::jthread t(worker);

    std::this_thread::sleep_for(1s);

    t.request_stop();
}
```

## بررسی وضعیت توقف

برای بررسی درخواست توقف:

```
token.stop_requested()
```

نمونه:

```
while (!token.stop_requested()) {
    // work
}
```

## تفاوت `stop_requested()` و `stop_possible()`

کلاس `std::stop_token` دو تابع مهم دارد.

| تابع               | کاربرد                                               |
| ------------------ | ---------------------------------------------------- |
| `stop_requested()` | آیا درخواست توقف ارسال شده است؟                      |
| `stop_possible()`  | آیا این Token اصلاً قابلیت دریافت درخواست توقف دارد؟ |

نمونه:

```
if (token.stop_possible()) {
}
```

در اغلب استفاده‌های `std::jthread` مقدار `stop_possible()` برابر `true` است.

## معرفی `stop_source`

کلاس `std::stop_source` منبع ارسال درخواست توقف است.

نمونه:

```
std::stop_source source;
auto token = source.get_token();

source.request_stop();
```

رابطه میان اجزا:

```
stop_source
      │
      ▼
stop_token
      │
      ▼
worker thread
```

در `std::jthread` این ارتباط به صورت خودکار مدیریت می‌شود.

## معرفی `stop_callback`

گاهی لازم است هنگام دریافت درخواست توقف عملی انجام شود.

برای این کار از `std::stop_callback` استفاده می‌شود.

نمونه:

```
void worker(std::stop_token token) {

    std::stop_callback callback(token, []{
        std::cout << "Stop callback\n";
    });

    while (!token.stop_requested()) {
    }
}
```

ویژگی‌ها:

- بلافاصله پس از درخواست توقف اجرا می‌شود.
- اجرای آن هم‌زمان با درخواست توقف تضمین می‌شود.
- مناسب برای آزادسازی منابع یا بیدار کردن عملیات منتظر است.

## تفاوت `join()` و `detach()`

در `std::jthread` همچنان هر دو تابع وجود دارند.

### رفتار `join()`

```
worker.join();
```

منتظر پایان Thread می‌ماند.

### رفتار `detach()`

```
worker.detach();
```

پس از `detach`:

- دیگر مدیریت Thread انجام نمی‌شود.
- Destructor نیز آن را `join` نخواهد کرد.

در نتیجه استفاده از `detach` همچنان باید با احتیاط انجام شود.

## انتقال مالکیت

مانند `std::thread`، کلاس `std::jthread` نیز Move-only است.

نمونه:

```
std::jthread t1([]{});

std::jthread t2 = std::move(t1);
```

کپی ممنوع است:

```
std::jthread t2 = t1; // Error
```

## بررسی وضعیت Thread

چند تابع مهم:

| تابع             | کاربرد                 |
| ---------------- | ---------------------- |
| `joinable()`     | بررسی فعال بودن Thread |
| `get_id()`       | شناسه Thread           |
| `join()`         | انتظار پایان           |
| `detach()`       | جدا کردن               |
| `request_stop()` | درخواست توقف           |

نمونه:

```
if (worker.joinable()) {
}
```

## استفاده همراه با Lambda و `stop_token`

نمونه متداول:

```
std::jthread worker([](std::stop_token token){

    while (!token.stop_requested()) {
    }

});
```

توجه کنید که `stop_token` باید اولین پارامتر باشد.

## استفاده در کلاس‌ها

یکی از بهترین کاربردهای `std::jthread` مدیریت Threadهای عضو کلاس است.

نمونه:

```
#include <chrono>
#include <thread>

using namespace std::chrono_literals;

class BackgroundWorker {

    std::jthread thread_;

public:

    BackgroundWorker()
        : thread_([](std::stop_token token){

            while (!token.stop_requested()) {
                std::this_thread::sleep_for(100ms);
            }

        })
    {
    }
};
```

مزیت:

نیازی به نوشتن Destructor برای `join()` نیست.

## مقایسه با `std::thread`

### تفاوت‌های اصلی

| ویژگی                          | `std::thread` | `std::jthread` |
| ------------------------------ | ------------- | -------------- |
| RAII                           | خیر           | بله            |
| `join()` خودکار                | خیر           | بله            |
| `stop_token`                   | ندارد         | دارد           |
| Cooperative Cancellation       | ندارد         | دارد           |
| `std::terminate` در Destructor | ممکن است      | خیر            |

### نمونه مقایسه

با `std::thread`:

```
std::thread t(worker);
t.join();
```

با `std::jthread`:

```
std::jthread t(worker);
```

نسخه دوم هم کوتاه‌تر است و هم ایمن‌تر.

## تفاوت با Cancellation قدیمی

پیش از ++C20 معمولاً از یک متغیر `atomic<bool>` استفاده می‌شد.

نمونه:

```
std::atomic<bool> stop = false;

while (!stop) {
}
```

مشکلات:

- نیاز به مدیریت دستی
- عدم هماهنگی استاندارد
- دشوار بودن اشتراک‌گذاری بین اجزا

در مقابل:

```
while (!token.stop_requested()) {
}
```

نسخه جدید خواناتر و استانداردتر است.

## تعامل با `condition_variable`

یکی از چالش‌های رایج، توقف Threadی است که منتظر `condition_variable` است.

راهکار مناسب استفاده از `stop_callback` است.

نمونه ساده:

```
#include <condition_variable>
#include <mutex>

void worker(std::stop_token token) {

    std::mutex mutex;
    std::condition_variable cv;

    std::unique_lock lock(mutex);

    std::stop_callback callback(token, [&]{
        cv.notify_all();
    });

    while (!token.stop_requested()) {
        cv.wait(lock);
    }
}
```

بدون `stop_callback` ممکن است Thread مدت زیادی در حالت انتظار باقی بماند.

## تعامل با `condition_variable_any`

در ++C20، کلاس `condition_variable_any` مستقیماً از `stop_token` پشتیبانی می‌کند.

نمونه:

```
#include <condition_variable>
#include <mutex>

void worker(std::stop_token token) {

    std::mutex mutex;
    std::condition_variable_any cv;

    std::unique_lock lock(mutex);

    cv.wait(lock, token, []{
        return false;
    });
}
```

این روش از ساده‌ترین راه‌های ایجاد انتظار قابل لغو است.

## تعامل با `sleep`

تابع `sleep_for` مستقیماً قابل لغو نیست.

نمونه مشکل:

```
while (!token.stop_requested()) {

    std::this_thread::sleep_for(10s);
}
```

در اینجا درخواست توقف ممکن است تا پایان خواب دیده نشود.

راه بهتر:

- استفاده از خواب‌های کوتاه‌تر
- استفاده از `condition_variable_any`

## مدیریت Exceptionها

رفتار Exceptionها تغییر نکرده است.

اگر Exception از تابع Thread خارج شود:

برنامه با `std::terminate` پایان می‌یابد.

نمونه صحیح:

```
void worker(std::stop_token token) {

    try {

        while (!token.stop_requested()) {
        }

    }
    catch (...) {

        // Handle exception

    }
}
```

## Edge Caseهای مهم

### درخواست توقف قبل از شروع واقعی کار

ممکن است `request_stop()` بسیار سریع فراخوانی شود.

بنابراین بهتر است ابتدای تابع نیز وضعیت بررسی شود.

```
if (token.stop_requested())
    return;
```

### توقف هنگام عملیات طولانی

اگر عملیات طولانی غیرقابل لغو باشد، `stop_token` کمکی نمی‌کند.

مثال:

- عملیات I/O بلوک‌شونده
- فراخوانی‌های سیستمی بدون قابلیت Interrupt

در این شرایط باید از APIهای قابل لغو سیستم‌عامل استفاده شود.

### استفاده پس از `detach`

پس از `detach`:

- دیگر Destructor مسئول مدیریت نیست.
- توقف خودکار انجام نمی‌شود.

## اشتباهات رایج

### فراموش کردن بررسی `stop_requested()`

نادرست:

```
while (true) {
}
```

درست:

```
while (!token.stop_requested()) {
}
```

### استفاده از خواب طولانی

نادرست:

```
sleep_for(1min);
```

درست:

```
sleep_for(100ms);
```

### خروج Exception از Thread

همیشه Exceptionها را داخل Thread مدیریت کنید.

### استفاده بی‌دلیل از `detach`

در بیشتر موارد `detach` انتخاب مناسبی نیست، زیرا مدیریت عمر Thread را دشوار می‌کند.

## ملاحظات طراحی

هنگام طراحی سیستم‌های چندریسمانی با `std::jthread` بهتر است:

- از `stop_token` در اولین پارامتر استفاده شود.
- حلقه‌های کاری به صورت دوره‌ای توقف را بررسی کنند.
- عملیات‌های طولانی به بخش‌های کوچک‌تر تقسیم شوند.
- منابع با RAII مدیریت شوند.
- از `condition_variable_any` برای انتظارهای قابل لغو استفاده شود.
- از `stop_callback` برای بیدار کردن عملیات منتظر استفاده شود.

## الگوی Worker قابل توقف

نمونه‌ای عملی:

```
#include <chrono>
#include <iostream>
#include <thread>

using namespace std::chrono_literals;

class Worker {

    std::jthread thread_;

public:

    Worker()
        : thread_([](std::stop_token token){

            while (!token.stop_requested()) {

                std::cout << "Processing\n";
                std::this_thread::sleep_for(200ms);

            }

            std::cout << "Worker stopped\n";

        })
    {
    }
};

int main() {

    Worker worker;

    std::this_thread::sleep_for(1s);
}
```

بدون نیاز به Destructor، هنگام خروج از `main`:

- درخواست توقف ارسال می‌شود.
- حلقه پایان می‌یابد.
- Thread به صورت ایمن `join` می‌شود.

## کارایی و سربار

از نظر کارایی، `std::jthread` بسیار نزدیک به `std::thread` است.

سربار اضافی عمدتاً شامل:

- نگهداری `stop_source`
- نگهداری `stop_token`
- مدیریت Callbackها

در بیشتر برنامه‌های واقعی این سربار ناچیز است و در مقابل ایمنی بیشتر ارزش استفاده را دارد.

## چه زمانی از `std::jthread` استفاده کنیم؟

موارد مناسب:

- سرویس‌های پس‌زمینه
- Worker Threadها
- پردازش دوره‌ای
- Threadهای عضو کلاس
- برنامه‌های مبتنی بر RAII
- سیستم‌های نیازمند توقف کنترل‌شده

مواردی که شاید مناسب نباشد:

- زمانی که از Threadهای جداشده (`detach`) استفاده می‌شود.
- زمانی که API خارجی مدیریت عمر Thread را بر عهده دارد.
- عملیات‌هایی که اصلاً امکان توقف همکاری‌شده ندارند.

## جمع‌بندی

کلاس `std::jthread` یکی از مهم‌ترین بهبودهای استاندارد ++C20 در حوزه هم‌زمانی است. این کلاس با ترکیب مدیریت خودکار عمر Thread و مکانیزم استاندارد Cooperative Cancellation، بسیاری از مشکلات رایج `std::thread` را برطرف می‌کند.

مهم‌ترین مزایای آن عبارت‌اند از:

- مدیریت خودکار `join()` بر اساس RAII
- جلوگیری از `std::terminate` ناشی از فراموش کردن `join()`
- پشتیبانی استاندارد از `stop_token`
- امکان توقف همکاری‌شده بدون خاتمه اجباری Thread
- ادغام مناسب با `condition_variable_any` و `stop_callback`
- طراحی ساده‌تر برای کلاس‌هایی که Thread داخلی دارند

برای توسعه نرم‌افزارهای مدرن در ++C20 و نسخه‌های جدیدتر، `std::jthread` در اغلب سناریوهای مدیریت Thread انتخاب مناسب‌تری نسبت به `std::thread` است، زیرا علاوه بر حفظ کارایی، ایمنی و قابلیت نگهداری کد را نیز به شکل محسوسی افزایش می‌دهد.

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>