<div align="center">

[🇺🇸 English](../../en/cpp20/std_barrier.md) | [🇮🇷 فارسی](./std_barrier.md)

</div>

---

# **معرفی std::barrier در ++C20**

## **مقدمه**

در برنامه‌نویسی چندریسمانی (Multithreading)، یکی از چالش‌های مهم، هماهنگ‌سازی اجرای چند Thread است. در بسیاری از الگوریتم‌های موازی، لازم است چند Thread یک مرحله از محاسبات را به پایان برسانند و سپس همگی وارد مرحله بعدی شوند.

برای مثال، فرض کنید یک الگوریتم پردازش تصویر به چند Thread تقسیم شده است. هر Thread بخشی از تصویر را پردازش می‌کند، اما مرحله بعدی الگوریتم تنها زمانی می‌تواند آغاز شود که تمام Threadها پردازش مرحله فعلی را به پایان رسانده باشند.

استفاده از ابزارهایی مانند `std::mutex` و `std::condition_variable` امکان پیاده‌سازی چنین هماهنگ‌سازی‌هایی را فراهم می‌کند؛ اما طراحی یک Barrier با این ابزارها به کدنویسی بیشتر، مدیریت وضعیت مشترک و توجه به جزئیات هم‌زمانی نیاز دارد.

استاندارد ++C20 برای ساده‌سازی این نوع هماهنگ‌سازی، قابلیت `std::barrier` را به کتابخانه استاندارد اضافه کرده است.

کلاس `std::barrier` یک ابزار Synchronization از نوع Phase-based است که به مجموعه‌ای از Threadها اجازه می‌دهد در یک نقطه مشخص منتظر بمانند تا تمام شرکت‌کنندگان به آن نقطه برسند. پس از تکمیل یک Phase، تمام Threadهای منتظر می‌توانند اجرای خود را ادامه دهند.

این قابلیت به‌ویژه در الگوریتم‌های موازی تکرارشونده، پردازش مرحله‌ای داده‌ها، شبیه‌سازی‌های علمی و محاسباتی که به چند مرحله وابسته هستند، کاربرد دارد.

## **مفهوم Barrier چیست؟**

در برنامه‌نویسی موازی، Barrier یک نقطه هماهنگ‌سازی است که Threadها باید قبل از عبور از آن منتظر تکمیل کار سایر شرکت‌کنندگان بمانند.

فرض کنید چهار Thread داریم که باید یک الگوریتم را در سه مرحله اجرا کنند:

- مرحله اول: هر Thread بخشی از داده‌ها را پردازش می‌کند.
- مرحله دوم: نتایج مرحله اول ترکیب یا بررسی می‌شوند.
- مرحله سوم: محاسبات بعدی بر اساس نتایج مرحله دوم انجام می‌شوند.

در این شرایط، شروع مرحله دوم پیش از اتمام مرحله اول توسط تمام Threadها ممکن است باعث استفاده از داده‌های ناقص شود.

رفتار Barrier را می‌توان به شکل زیر تصور کرد:

```
Thread 1 ───── Phase 1 ───── Wait ───── Phase 2 ─────
Thread 2 ───── Phase 1 ───── Wait ───── Phase 2 ─────
Thread 3 ───── Phase 1 ───── Wait ───── Phase 2 ─────
Thread 4 ───── Phase 1 ───── Wait ───── Phase 2 ─────
                                |
                         All threads arrive
                                |
                         Barrier completes
```

در این مدل، هیچ Threadی نمی‌تواند از Barrier عبور کند، مگر اینکه تمام شرکت‌کنندگان مورد انتظار به آن رسیده باشند.

### **تفاوت Barrier با سایر ابزارهای Synchronization**

هر ابزار Synchronization برای حل نوع مشخصی از مسئله طراحی شده است.

| ابزار                     | کاربرد اصلی                                      |
| ------------------------- | ------------------------------------------------ |
| `std::mutex`              | محافظت از داده‌های مشترک در برابر دسترسی هم‌زمان |
| `std::condition_variable` | انتظار برای برقرار شدن یک شرط                    |
| `std::latch`              | انتظار برای رسیدن یک شمارنده به صفر              |
| `std::barrier`            | هماهنگ‌سازی چند Thread در چند مرحله متوالی       |
| `std::semaphore`          | کنترل تعداد دسترسی‌های هم‌زمان به یک منبع        |
| `std::atomic`             | انجام عملیات اتمیک روی داده‌ها                   |

تفاوت اصلی `std::barrier` با `std::latch` در قابلیت استفاده مجدد آن است.

یک `std::latch` پس از رسیدن شمارنده به صفر قابل Reset یا استفاده مجدد برای یک مرحله جدید نیست؛ اما `std::barrier` پس از تکمیل هر Phase می‌تواند وارد Phase بعدی شود.

## **معرفی std::barrier**

کلاس `std::barrier` در Header زیر تعریف شده است:

```
#include <barrier>
```

این کلاس بخشی از Standard Library استاندارد ++C20 است و برای هماهنگ‌سازی چند شرکت‌کننده طراحی شده است.

تعریف کلی این کلاس به شکل زیر است:

```
template<class CompletionFunction = /* implementation-defined */>
class barrier;
```

پارامتر Template آن یک تابع Completion است که هنگام تکمیل هر Phase اجرا می‌شود.

در حالت معمول، می‌توان از نوع پیش‌فرض استفاده کرد:

```
std::barrier barrier{4};
```

در این مثال، Barrier با چهار شرکت‌کننده ساخته شده است.

هر چهار شرکت‌کننده باید به Barrier برسند تا Phase جاری تکمیل شود.

### **ساختار کلی کلاس**

مهم‌ترین عملیات عمومی `std::barrier` عبارت‌اند از:

| عملیات              | توضیح                                                      |
| ------------------- | ---------------------------------------------------------- |
| `arrive()`          | اعلام رسیدن یک شرکت‌کننده به Barrier                       |
| `wait()`            | منتظر ماندن برای تکمیل Phase                               |
| `arrive_and_wait()` | اعلام رسیدن و انتظار برای تکمیل Phase                      |
| `arrive_and_drop()` | اعلام رسیدن و حذف دائمی شرکت‌کننده از Phaseهای بعدی        |
| `max()`             | ارائه حداکثر تعداد قابل پشتیبانی برای شمارنده شرکت‌کنندگان |

همچنین این کلاس دارای مفهوم Completion Function و Phase است که در ادامه بررسی می‌شوند.

## **مفهوم Phase در std::barrier**

یکی از مهم‌ترین مفاهیم در درک `std::barrier`، مفهوم Phase است.

هر Barrier از مجموعه‌ای از Phaseهای متوالی تشکیل می‌شود. هر Phase زمانی آغاز می‌شود که Phase قبلی تکمیل شده باشد.

در زمان ساخت Barrier، تعداد اولیه شرکت‌کنندگان مشخص می‌شود. این تعداد، Expected Count نام دارد.

برای مثال:

```
std::barrier syncPoint{3};
```

در این مثال، مقدار اولیه Expected Count برابر سه است.

هر بار که یک Thread به Barrier می‌رسد، تعداد شرکت‌کنندگان باقی‌مانده کاهش پیدا می‌کند.

وقتی تعداد شرکت‌کنندگان مورد انتظار به صفر برسد:

- Phase جاری تکمیل می‌شود.
- Completion Function در صورت وجود اجرا می‌شود.
- Expected Count برای Phase بعدی تنظیم می‌شود.
- Threadهای منتظر اجازه ادامه اجرا پیدا می‌کنند.

البته مقدار Expected Count ممکن است با استفاده از `arrive_and_drop()` برای Phaseهای آینده تغییر کند.

### **چرخه عمر یک Phase**

ساختار کلی چرخه یک Phase به شکل زیر است:

```
       Phase N begins
              |
              v
    Participants arrive
              |
              v
    Expected Count reaches 0
              |
              v
    Completion Function executes
              |
              v
       Phase N completes
              |
              v
       Phase N+1 begins
              |
              v
       Participants continue
```

نکته مهم این است که Phaseها مستقل از تعداد دفعات فراخوانی `wait()` نیستند؛ آنچه تکمیل Phase را تعیین می‌کند، تعداد عملیات Arrival و تغییرات Expected Count است.

## **نحوه ساخت std::barrier**

برای ایجاد یک Barrier معمولی، کافی است تعداد شرکت‌کنندگان را به Constructor ارسال کنید.

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 4;

    std::barrier syncPoint{threadCount};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &syncPoint] {
            std::cout << "Thread " << i << " is working\n";

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " passed the barrier\n";
        });
    }
}
```

در این مثال:

- چهار Thread ایجاد می‌شوند.
- هر Thread عملیات اولیه خود را انجام می‌دهد.
- سپس با استفاده از `arrive_and_wait()` به Barrier می‌رسد.
- هیچ Threadی نمی‌تواند تا زمان رسیدن سایر شرکت‌کنندگان از Barrier عبور کند.

در این مثال از `std::jthread` استفاده شده است که از ++C20 در دسترس است و در پایان Scope به‌صورت خودکار Join می‌شود.

## **بررسی عملیات arrive()**

تابع `arrive()` برای اعلام رسیدن یک شرکت‌کننده به Barrier استفاده می‌شود.

```
std::barrier syncPoint{3};

auto token = syncPoint.arrive();
```

با فراخوانی این تابع:

- Arrival شرکت‌کننده ثبت می‌شود.
- Expected Count یک واحد کاهش پیدا می‌کند.
- Thread منتظر تکمیل Barrier نمی‌ماند.
- یک `arrival_token` بازگردانده می‌شود.

نوع بازگشتی این تابع به شکل زیر است:

```
std::barrier<>::arrival_token
```

این Token برای اتصال عملیات Arrival به عملیات Wait استفاده می‌شود.

### **کاربرد arrive() بدون انتظار فوری**

گاهی لازم است یک Thread رسیدن خود را اعلام کند، اما قبل از انتظار روی Barrier کار دیگری انجام دهد.

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 3;

    std::barrier syncPoint{threadCount};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &syncPoint] {
            std::cout << "Thread " << i << " completed its work\n";

            auto token = syncPoint.arrive();

            std::cout << "Thread " << i
                      << " is performing independent work\n";

            syncPoint.wait(std::move(token));

            std::cout << "Thread " << i
                      << " passed the barrier\n";
        });
    }
}
```

در این مثال، Thread ابتدا Arrival خود را اعلام می‌کند و سپس یک کار مستقل انجام می‌دهد.

پس از پایان آن کار، با استفاده از Token مربوط به همان Arrival منتظر تکمیل Phase می‌ماند.

این الگو برای زمانی مفید است که Thread می‌تواند بخشی از کارهای مستقل خود را بدون نیاز به نتایج سایر Threadها انجام دهد.

### **نکات مهم درباره arrival_token**

نوع `arrival_token` برای انتقال اطلاعات مربوط به یک Arrival به عملیات `wait()` استفاده می‌شود.

نکات مهم آن عبارت‌اند از:

- این Token باید برای همان Barrier استفاده شود که Arrival روی آن انجام شده است.
- Token مربوط به یک Phase را نباید برای Phase دیگری استفاده کرد.
- Token باید به‌صورت صحیح منتقل شود؛ زیرا این نوع برای Move طراحی شده است.
- نباید یک Token را برای چند Wait مستقل استفاده کرد.
- استفاده از Token نامعتبر یا متعلق به Barrier دیگر، رفتار تعریف‌نشده (Undefined Behavior) ایجاد می‌کند.

## **بررسی عملیات wait()**

تابع `wait()` برای منتظر ماندن تا تکمیل Phase استفاده می‌شود.

این تابع یک `arrival_token` دریافت می‌کند:

```
syncPoint.wait(std::move(token));
```

تابع `wait()` خودش Expected Count را کاهش نمی‌دهد.

این نکته اهمیت زیادی دارد؛ زیرا عملیات Arrival قبلاً توسط `arrive()` انجام شده است.

در واقع، `wait()` فقط منتظر تکمیل Phase مربوط به Arrival ثبت‌شده می‌ماند.

### **تفاوت arrive() و wait()**

| ویژگی                   | `arrive()` | `wait()` |
| ----------------------- | ---------- | -------- |
| ثبت Arrival             | بله        | خیر      |
| کاهش Expected Count     | بله        | خیر      |
| انتظار برای تکمیل Phase | خیر        | بله      |
| بازگرداندن Token        | بله        | خیر      |

استفاده از این دو تابع به‌صورت جداگانه، امکان کنترل دقیق‌تری روی زمان انتظار ایجاد می‌کند.

## **بررسی عملیات arrive_and_wait()**

تابع `arrive_and_wait()` رایج‌ترین روش استفاده از `std::barrier` است.

این تابع دو عملیات را به‌صورت یک مرحله منطقی انجام می‌دهد:

- ثبت Arrival شرکت‌کننده
- انتظار تا تکمیل Phase

نمونه:

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 4;

    std::barrier syncPoint{threadCount};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &syncPoint] {
            std::cout << "Starting phase 1: " << i << '\n';

            syncPoint.arrive_and_wait();

            std::cout << "Starting phase 2: " << i << '\n';

            syncPoint.arrive_and_wait();

            std::cout << "Starting phase 3: " << i << '\n';
        });
    }
}
```

در این مثال، هر Thread سه مرحله از محاسبات را اجرا می‌کند.

Barrier تضمین می‌کند که هیچ Threadی وارد مرحله دوم نمی‌شود، مگر اینکه تمام شرکت‌کنندگان مرحله اول را تکمیل کرده باشند.

همین قانون برای انتقال از مرحله دوم به مرحله سوم نیز برقرار است.

### **چه زمانی از arrive_and_wait() استفاده کنیم؟**

این تابع زمانی بهترین انتخاب است که Thread پس از رسیدن به Barrier هیچ کار مستقلی برای انجام دادن ندارد.

برای مثال:

- پایان یک مرحله از Parallel Processing
- انتظار برای تکمیل محاسبات تمام Workerها
- هماهنگی در Iterative Algorithms
- انتقال بین مراحل یک Simulation

اگر Thread پس از Arrival نیاز به انجام کار مستقل دارد، ترکیب `arrive()` و `wait()` انعطاف بیشتری فراهم می‌کند.

## **قابلیت Completion Function**

یکی از قابلیت‌های مهم `std::barrier` امکان تعریف Completion Function است.

این تابع زمانی اجرا می‌شود که تمام Arrivalهای مورد انتظار برای Phase جاری ثبت شده باشند.

Completion Function قبل از تکمیل Phase اجرا می‌شود و می‌تواند برای انجام عملیات مشترک بین دو Phase استفاده شود.

برای مثال، فرض کنید چند Thread داده‌های جداگانه‌ای را محاسبه کرده‌اند و پس از پایان محاسبات باید نتایج آن‌ها با یکدیگر ترکیب شوند.

می‌توان این کار را در Completion Function انجام داد.

### **تعریف Completion Function**

نوع Completion Function باید یک Callable بدون آرگومان باشد که با `noexcept` قابل فراخوانی باشد.

برای مثال:

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 4;

    auto completion = []() noexcept {
        std::cout << "All threads completed the phase\n";
    };

    std::barrier syncPoint{threadCount, completion};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &syncPoint] {
            std::cout << "Thread " << i << " is working\n";

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " continues execution\n";
        });
    }
}
```

در این مثال، Completion Function زمانی اجرا می‌شود که تمام چهار Thread به Barrier رسیده باشند.

نکته مهم این است که Completion Function توسط یک Thread از میان شرکت‌کنندگان اجرا می‌شود، نه لزوماً توسط یک Thread مشخص.

بنابراین نباید فرض کرد که همیشه Thread خاصی مسئول اجرای آن خواهد بود.

### **محدودیت‌های Completion Function**

در طراحی Completion Function باید به چند نکته توجه کرد:

- تابع باید `noexcept` باشد.
- تابع نباید عملیات مسدودکننده‌ای انجام دهد که به Threadهای منتظر روی همان Barrier وابسته هستند.
- تابع نباید باعث Deadlock شود.
- تابع باید در مدت‌زمان معقولی اجرا شود.
- بهتر است از انجام عملیات سنگین یا طولانی در آن خودداری شود.

همچنین اگر Completion Function یک تابع عضو باشد یا State مشترکی را تغییر دهد، باید مدیریت صحیح طول عمر و Synchronization داده‌های مربوط به آن تضمین شود.

### **بررسی نوع Completion Function**

در Template کلاس `std::barrier`، نوع Completion Function به‌عنوان پارامتر دوم Template مشخص می‌شود.

در صورت استفاده از تابع پیش‌فرض، می‌توان از این ساختار استفاده کرد:

```
std::barrier syncPoint{4};
```

اما در صورت تعریف Completion Function، نوع آن از طریق Class Template Argument Deduction قابل استنتاج است:

```
auto completion = []() noexcept {};

std::barrier syncPoint{4, completion};
```

این قابلیت باعث می‌شود نیازی به نوشتن صریح نوع Template نباشد.

## **مدیریت داده‌های مشترک با Completion Function**

یکی از کاربردهای مهم Completion Function، تجمیع نتایج تولیدشده توسط Threadهای مختلف است.

فرض کنید هر Thread بخشی از یک آرایه را پردازش می‌کند و نتیجه محاسبات خود را در یک متغیر محلی ذخیره می‌کند.

در پایان هر Phase، باید مجموع نتایج تمام Threadها محاسبه شود.

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>
#include <array>

int main() {
    constexpr int threadCount = 4;

    std::array<int, threadCount> partialResults{};
    int totalResult = 0;

    auto completion = [&]() noexcept {
        totalResult = 0;

        for (int value : partialResults) {
            totalResult += value;
        }

        std::cout << "Total result: " << totalResult << '\n';
    };

    std::barrier syncPoint{threadCount, completion};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &partialResults, &syncPoint] {
            partialResults[i] = (i + 1) * 10;

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " sees total: " << totalResult << '\n';
        });
    }
}
```

در این مثال:

- هر Thread مقدار اختصاصی خود را در آرایه ثبت می‌کند.
- تمام Threadها به Barrier می‌رسند.
- Completion Function مجموع مقادیر را محاسبه می‌کند.
- پس از تکمیل Phase، Threadها می‌توانند مقدار نهایی را مشاهده کنند.

از آنجا که هر Thread در این مثال به عنصر متفاوتی از آرایه دسترسی نوشتاری دارد، عملیات نوشتن آن‌ها با یکدیگر تداخل ندارد.

همچنین Completion Function پس از Arrival تمام شرکت‌کنندگان اجرا می‌شود و تکمیل Phase ارتباط حافظه‌ای لازم را برای مشاهده نتایج توسط Threadهای بعدی فراهم می‌کند.

## **بررسی عملیات arrive_and_drop()**

گاهی ممکن است یک Thread پس از پایان کار خود دیگر نیازی به شرکت در Phaseهای بعدی نداشته باشد.

برای مثال، در یک الگوریتم موازی ممکن است بعضی Workerها پس از پردازش یک بخش از داده‌ها به کار خود پایان دهند.

برای چنین شرایطی، `std::barrier` تابع `arrive_and_drop()` را ارائه می‌دهد.

این تابع دو عملیات انجام می‌دهد:

- Arrival Thread جاری را برای Phase فعلی ثبت می‌کند.
- تعداد شرکت‌کنندگان مورد انتظار برای تمام Phaseهای آینده را یک واحد کاهش می‌دهد.

نمونه:

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    std::barrier syncPoint{3};

    std::vector<std::jthread> threads;

    for (int i = 0; i < 3; ++i) {
        threads.emplace_back([i, &syncPoint] {
            std::cout << "Thread " << i << " started\n";

            if (i == 2) {
                syncPoint.arrive_and_drop();
                std::cout << "Thread 2 dropped\n";
                return;
            }

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " passed the first phase\n";

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " passed the second phase\n";
        });
    }
}
```

در این مثال، Thread شماره دو پس از ثبت Arrival از Phaseهای آینده خارج می‌شود.

بنابراین:

- Phase اول همچنان به سه Arrival نیاز دارد.
- Phaseهای بعدی فقط به دو Arrival نیاز خواهند داشت.

### **تفاوت arrive_and_drop() با arrive()**

تفاوت اصلی این دو تابع در تأثیر آن‌ها بر Phaseهای آینده است.

| ویژگی                     | `arrive()` | `arrive_and_drop()` |
| ------------------------- | ---------- | ------------------- |
| ثبت Arrival فعلی          | بله        | بله                 |
| کاهش شمارنده Phase فعلی   | بله        | بله                 |
| کاهش Expected Count آینده | خیر        | بله                 |
| امکان شرکت در Phase بعدی  | بله        | خیر                 |

نکته مهم این است که `arrive_and_drop()` به معنی حذف فوری Thread از Phase فعلی نیست؛ Arrival آن همچنان در تکمیل Phase جاری محاسبه می‌شود.

### **موارد استفاده از arrive_and_drop()**

این تابع برای شرایطی مناسب است که عضویت Threadها در Barrier در طول اجرای برنامه کاهش پیدا می‌کند.

برای مثال:

- الگوریتم‌هایی که برخی Workerها پس از رسیدن به نتیجه نهایی خارج می‌شوند.
- پردازش‌هایی که تعداد شرکت‌کنندگان در هر مرحله کمتر می‌شود.
- سیستم‌هایی که Workerها پس از انجام وظیفه مشخصی دیگر نیازی به Synchronization ندارند.

البته این تابع برای حذف موقت Thread یا خارج شدن و بازگشت مجدد آن مناسب نیست.

## **مفهوم Expected Count و عملیات Arrival**

درک صحیح Expected Count برای جلوگیری از Deadlock و رفتار نامعتبر ضروری است.

فرض کنید Barrier با مقدار چهار ساخته شده باشد:

```
std::barrier syncPoint{4};
```

در شروع Phase، مقدار Expected Count برابر چهار است.

هر عملیات Arrival این مقدار را کاهش می‌دهد.

| عملیات        | Expected Count باقی‌مانده |
| ------------- | ------------------------- |
| مقدار اولیه   | 4                         |
| Arrival اول   | 3                         |
| Arrival دوم   | 2                         |
| Arrival سوم   | 1                         |
| Arrival چهارم | 0                         |

با رسیدن مقدار به صفر، Phase تکمیل می‌شود.

### **نکته مهم درباره تعداد Arrivalها**

هر شرکت‌کننده باید دقیقاً به اندازه سهم خود در Barrier عملیات Arrival انجام دهد.

اگر Barrier انتظار چهار Arrival داشته باشد اما فقط سه Thread به آن برسند، Phase تکمیل نمی‌شود.

همچنین اگر یک Thread بیش از سهم تعیین‌شده خود Arrival انجام دهد، ممکن است شمارنده به وضعیت نامعتبر برسد و رفتار برنامه Undefined Behavior شود.

بنابراین تعداد Arrivalها باید با طراحی الگوریتم هماهنگ باشد.

## **بررسی تابع max()**

تابع `max()` حداکثر مقدار قابل پشتیبانی برای شمارنده شرکت‌کنندگان را برمی‌گرداند.

امضای آن به شکل زیر است:

```
static constexpr std::ptrdiff_t max() noexcept;
```

این تابع Static است و نیازی به ساخت نمونه از Barrier ندارد.

نمونه:

```
#include <barrier>
#include <iostream>

int main() {
    std::cout << std::barrier<>::max() << '\n';
}
```

مقدار بازگشتی این تابع وابسته به Implementation است و نباید مقدار ثابتی برای تمام Compilerها و Platformها فرض کرد.

همچنین مقدار اولیه Expected Count نباید از این حد بیشتر باشد.

## **ارتباط حافظه‌ای و Memory Ordering**

یکی از مهم‌ترین ویژگی‌های `std::barrier`، تضمین ارتباط حافظه‌ای بین عملیات انجام‌شده توسط Threadها است.

در برنامه‌های چندریسمانی، صرفاً رسیدن Threadها به یک نقطه مشترک کافی نیست؛ بلکه باید تضمین شود تغییرات داده‌ها نیز به‌درستی برای سایر Threadها قابل مشاهده باشند.

این موضوع با مفهوم Happens-before در Memory Model استاندارد ++C ارتباط دارد.

### **تضمین Synchronization در Barrier**

در استاندارد ++C20، Completion Step دارای ارتباط حافظه‌ای مشخصی با عملیات Arrival و Wait است.

به‌طور خلاصه:

- عملیات Arrival شرکت‌کنندگان در Phase جاری، قبل از اجرای Completion Step قرار می‌گیرند.
- اجرای Completion Step قبل از بازگشت عملیات Wait مربوط به Phase قرار می‌گیرد.
- عملیات انجام‌شده توسط Threadها پیش از Barrier می‌توانند پس از Barrier توسط سایر Threadها مشاهده شوند، مشروط بر اینکه دسترسی‌ها به‌درستی در چارچوب این Synchronization قرار گرفته باشند.

این ویژگی باعث می‌شود در بسیاری از سناریوها نیازی به استفاده از Mutex برای انتقال نتایج بین Phaseها نباشد.

### **مثال مشاهده داده‌های مشترک**

```
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 3;

    int sharedData = 0;

    std::barrier syncPoint{threadCount};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &sharedData, &syncPoint] {
            if (i == 0) {
                sharedData = 100;
            }

            syncPoint.arrive_and_wait();

            std::cout << "Thread " << i
                      << " reads: " << sharedData << '\n';
        });
    }
}
```

در این مثال، Thread شماره صفر مقدار متغیر مشترک را تغییر می‌دهد.

سایر Threadها پس از عبور از Barrier می‌توانند مقدار جدید را مشاهده کنند.

این کار به دلیل ارتباط حافظه‌ای Barrier امکان‌پذیر است.

البته این تضمین به معنای مجاز بودن دسترسی‌های نوشتاری هم‌زمان و بدون Synchronization نیست.

اگر چند Thread به‌صورت هم‌زمان روی یک داده عملیات نوشتن انجام دهند، همچنان باید از ابزارهای مناسب مانند Mutex یا Atomic استفاده شود.

## **تفاوت std::barrier و std::latch**

هر دو کلاس `std::barrier` و `std::latch` از ++C20 به Standard Library اضافه شده‌اند و برای هماهنگ‌سازی Threadها استفاده می‌شوند.

با این حال، مدل استفاده آن‌ها متفاوت است.

### **معرفی std::latch**

کلاس `std::latch` یک شمارنده یک‌طرفه برای Synchronization است.

وقتی شمارنده آن به صفر برسد، تمام Threadهای منتظر آزاد می‌شوند.

پس از رسیدن شمارنده به صفر، این وضعیت قابل بازنشانی نیست.

نمونه:

```
#include <latch>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 4;

    std::latch completion{threadCount};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &completion] {
            std::cout << "Thread " << i << " working\n";

            completion.count_down();

            completion.wait();

            std::cout << "All threads completed\n";
        });
    }
}
```

### **مقایسه قابلیت‌ها**

| ویژگی                         | `std::barrier`       | `std::latch`                |
| ----------------------------- | -------------------- | --------------------------- |
| معرفی در استاندارد            | C++20                | C++20                       |
| استفاده مجدد                  | بله                  | خیر                         |
| پشتیبانی از چند Phase         | بله                  | خیر                         |
| Completion Function           | بله                  | خیر                         |
| کاهش تعداد شرکت‌کنندگان آینده | بله، با Drop         | خیر                         |
| کاربرد اصلی                   | هماهنگی مراحل متوالی | انتظار برای تکمیل یک عملیات |

### **انتخاب ابزار مناسب**

اگر هدف انتظار برای پایان یک مجموعه عملیات مستقل است، `std::latch` معمولاً انتخاب ساده‌تری است.

اما اگر Threadها باید چند مرحله از یک الگوریتم را به‌صورت هماهنگ اجرا کنند، `std::barrier` گزینه مناسب‌تری خواهد بود.

## **مقایسه std::barrier و std::condition_variable**

کلاس `std::condition_variable` از ++C11 در دسترس است و امکان انتظار برای برقرار شدن یک شرط را فراهم می‌کند.

این کلاس انعطاف بیشتری نسبت به Barrier دارد، اما مدیریت Synchronization آن پیچیده‌تر است.

برای استفاده صحیح از `std::condition_variable` معمولاً به موارد زیر نیاز داریم:

- Mutex برای محافظت از وضعیت مشترک
- شرطی که باید برقرار شود
- مدیریت Wake-upهای غیرمنتظره (Spurious Wakeups)
- مدیریت صحیح وضعیت انتظار و اطلاع‌رسانی

در مقابل، `std::barrier` این منطق را به شکل یک Abstraction آماده ارائه می‌کند.

### **چه زمانی condition_variable مناسب‌تر است؟**

اگر تعداد شرکت‌کنندگان ثابت نیست یا Threadها باید بر اساس یک شرط پویا آزاد شوند، `std::condition_variable` انعطاف بیشتری دارد.

برای مثال:

- انتظار برای ورود یک Task به Queue
- انتظار برای آماده شدن یک منبع
- هماهنگی Producer و Consumer
- انتظار برای یک شرط پیچیده

اما اگر هدف صرفاً هماهنگی تعداد مشخصی از Threadها در چند مرحله متوالی باشد، `std::barrier` معمولاً خواناتر و کم‌خطاتر است.

## **کاربرد عملی در الگوریتم‌های موازی**

یکی از مهم‌ترین موارد استفاده از `std::barrier`، الگوریتم‌هایی است که از چند مرحله محاسباتی متوالی تشکیل شده‌اند.

### **کاربرد در پردازش تصویر**

فرض کنید یک تصویر بزرگ به چند بخش تقسیم شده و هر Thread یک بخش را پردازش می‌کند.

پردازش تصویر ممکن است شامل مراحل زیر باشد:

- محاسبه اولیه Pixelها
- اعمال فیلتر
- ترکیب نتایج
- انجام مرحله بعدی پردازش

در چنین الگوریتمی، بعضی مراحل به نتایج تمام Threadها وابسته هستند.

استفاده از Barrier باعث می‌شود Threadها قبل از شروع مرحله بعدی منتظر تکمیل مرحله جاری بمانند.

### **کاربرد در شبیه‌سازی‌های علمی**

در شبیه‌سازی‌هایی مانند Particle Simulation یا مدل‌سازی سیستم‌های فیزیکی، وضعیت هر بخش از سیستم ممکن است در هر Iteration به‌روزرسانی شود.

در این شرایط، Barrier می‌تواند بین Iterationها قرار بگیرد تا اطمینان حاصل شود تمام Threadها وضعیت مرحله جاری را تکمیل کرده‌اند.

### **کاربرد در الگوریتم‌های Iterative**

در الگوریتم‌هایی مانند بعضی روش‌های Numerical Computing، محاسبات به شکل تکرارشونده انجام می‌شوند.

هر Iteration ممکن است شامل موارد زیر باشد:

- محاسبه بخش‌های مستقل
- تجمیع نتایج
- بررسی شرط پایان
- شروع Iteration بعدی

Barrier برای هماهنگ‌سازی این مراحل بسیار مناسب است.

## **مثال عملی: الگوریتم چندمرحله‌ای با تجمیع نتایج**

در مثال زیر، چهار Thread در چند Iteration مقدار اختصاصی خود را محاسبه می‌کنند.

در پایان هر Iteration، Completion Function مجموع مقادیر را محاسبه می‌کند.

```
#include <array>
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    constexpr int threadCount = 4;
    constexpr int iterations = 3;

    std::array<int, threadCount> results{};
    int total = 0;

    auto completion = [&]() noexcept {
        total = 0;

        for (int value : results) {
            total += value;
        }

        std::cout << "Iteration completed. Total = "
                  << total << '\n';
    };

    std::barrier syncPoint{threadCount, completion};

    std::vector<std::jthread> threads;

    for (int i = 0; i < threadCount; ++i) {
        threads.emplace_back([i, &results, &syncPoint] {
            for (int iteration = 0; iteration < iterations; ++iteration) {
                results[i] = (iteration + 1) * (i + 1);

                syncPoint.arrive_and_wait();
            }
        });
    }
}
```

در این برنامه، هر Thread در هر Iteration مقدار جدیدی را در بخش اختصاصی خود از آرایه ذخیره می‌کند.

پس از رسیدن تمام Threadها به Barrier، Completion Function اجرا می‌شود و مجموع نتایج را محاسبه می‌کند.

این فرآیند سه بار تکرار می‌شود.

نکته مهم این است که مقدار `total` در Completion Function تغییر می‌کند و Threadها پس از Barrier می‌توانند مقدار جدید آن را مشاهده کنند.

## **استفاده از Barrier در طراحی Thread Pool**

در بعضی طراحی‌های Thread Pool، ممکن است مجموعه‌ای از Workerها وظایف خود را در چند Batch پردازش کنند.

برای مثال، یک سیستم ممکن است به شکل زیر کار کند:

```
Batch 1:
    Worker 1 processes tasks
    Worker 2 processes tasks
    Worker 3 processes tasks
              |
           Barrier
              |
Batch 2:
    Worker 1 processes tasks
    Worker 2 processes tasks
    Worker 3 processes tasks
              |
           Barrier
              |
Batch 3
```

در چنین طراحی‌ای، Barrier می‌تواند تضمین کند تمام Workerها قبل از شروع Batch بعدی، Batch قبلی را به پایان رسانده‌اند.

البته استفاده از Barrier در Thread Poolهای عمومی همیشه مناسب نیست.

اگر Taskها مستقل هستند و ممکن است تعداد Workerهای فعال تغییر کند، استفاده از Queue، Condition Variable یا Task Scheduler معمولاً انعطاف بیشتری فراهم می‌کند.

## **محدودیت‌های std::barrier**

با وجود کاربردهای فراوان، `std::barrier` برای تمام مسائل Synchronization مناسب نیست.

### **ثابت بودن ساختار شرکت‌کنندگان**

تعداد اولیه شرکت‌کنندگان در زمان ساخت Barrier مشخص می‌شود.

این تعداد در حالت عادی در طول اجرای برنامه تغییر نمی‌کند.

تنها راه استاندارد برای کاهش تعداد شرکت‌کنندگان مورد انتظار در Phaseهای آینده، استفاده از `arrive_and_drop()` است.

امکان اضافه کردن شرکت‌کننده جدید به Barrier وجود ندارد.

### **وابستگی به تمام شرکت‌کنندگان**

تکمیل هر Phase به Arrival تمام شرکت‌کنندگان مورد انتظار وابسته است.

اگر یک Thread:

- متوقف شود؛
- به Deadlock برسد؛
- عملیات Arrival را اجرا نکند؛
- به دلیل خطا زودتر از Barrier خارج شود؛

ممکن است سایر Threadها برای همیشه منتظر بمانند.

### **نبود Timeout برای Wait**

تابع `wait()` دارای پارامتر Timeout نیست.

بنابراین نمی‌توان مستقیماً با این تابع مشخص کرد که Thread حداکثر تا چه مدت منتظر بماند.

اگر به Timeout نیاز دارید، باید از ابزارهای دیگری مانند `std::condition_variable` استفاده کنید.

### **عدم تضمین ترتیب اجرای Threadها**

Barrier فقط تکمیل Phase را هماهنگ می‌کند.

این کلاس هیچ تضمینی درباره ترتیب اجرای Threadها پس از عبور از Barrier ارائه نمی‌دهد.

بنابراین نمی‌توان فرض کرد Thread شماره یک همیشه قبل از Thread شماره دو ادامه اجرا خواهد داد.

### **عدم تضمین Fairness**

استاندارد تضمین نمی‌کند که Threadها پس از تکمیل Barrier با ترتیب مشخص یا به‌صورت عادلانه زمان CPU دریافت کنند.

زمان‌بندی Threadها همچنان تحت کنترل Operating System و Thread Scheduler قرار دارد.

## **خطاهای رایج هنگام استفاده از std::barrier**

### **عدم رسیدن تمام Threadها به Barrier**

یکی از رایج‌ترین اشتباهات، طراحی نادرست مسیرهای اجرای Threadها است.

برای مثال:

```
if (someCondition) {
    syncPoint.arrive_and_wait();
}
```

اگر بعضی Threadها وارد این شرط شوند و بعضی دیگر نشوند، ممکن است Barrier هرگز تکمیل نشود.

در طراحی Barrier باید مطمئن شوید تمام شرکت‌کنندگان مورد انتظار، مسیرهای Arrival خود را به‌درستی طی می‌کنند.

### **فراخوانی بیش از حد عملیات Arrival**

هر شرکت‌کننده باید تعداد Arrivalهای مجاز خود را رعایت کند.

فراخوانی اضافی `arrive()` یا `arrive_and_wait()` ممکن است باعث نقض قرارداد Barrier و Undefined Behavior شود.

### **اشتباه گرفتن wait و arrive**

تابع `wait()` شمارنده را کاهش نمی‌دهد.

بنابراین اگر انتظار دارید یک Thread با فراخوانی `wait()` به Barrier برسد، طراحی شما اشتباه است.

برای ثبت Arrival باید از `arrive()` یا `arrive_and_wait()` استفاده شود.

### **استفاده اشتباه از arrival_token**

Token باید با Arrival مربوط به همان Barrier و همان Phase سازگار باشد.

استفاده مجدد یا انتقال Token نامعتبر می‌تواند رفتار تعریف‌نشده ایجاد کند.

### **انجام عملیات مسدودکننده در Completion Function**

Completion Function در مسیر تکمیل Phase قرار دارد.

اگر این تابع منتظر Threadهایی بماند که برای تکمیل همان Phase به ادامه اجرای آن وابسته هستند، ممکن است Deadlock ایجاد شود.

### **از بین رفتن Barrier قبل از پایان استفاده**

طول عمر Barrier باید از تمام Threadهایی که به آن دسترسی دارند بیشتر باشد.

اگر Barrier پیش از پایان عملیات تمام Threadهای مرتبط از بین برود، رفتار برنامه تعریف‌شده نخواهد بود.

به همین دلیل، باید Join شدن Threadها و طول عمر اشیای مشترک به‌درستی مدیریت شوند.

## **نکات پیشرفته درباره طراحی Barrier**

### **استفاده از std::barrier با std::jthread**

کلاس `std::jthread` از ++C20 معرفی شده و مدیریت پایان عمر Thread را ساده‌تر می‌کند.

این کلاس در پایان Scope به‌صورت خودکار Join می‌شود.

استفاده از آن در کنار `std::barrier` می‌تواند ساختار برنامه را ساده‌تر کند.

البته توجه داشته باشید که Destructor مربوط به `std::jthread` ابتدا برای درخواست توقف، `request_stop()` را فراخوانی می‌کند و سپس Thread را Join می‌کند.

در نتیجه، اگر Threadهای شما از Stop Token استفاده نمی‌کنند، درخواست توقف لزوماً باعث خروج آن‌ها از انتظار روی Barrier نمی‌شود.

### **هماهنگی Stop Token و Barrier**

یکی از محدودیت‌های مهم `std::barrier` این است که تابع `wait()` مستقیماً از `std::stop_token` پشتیبانی نمی‌کند.

بنابراین درخواست توقف یک `std::jthread` به‌تنهایی نمی‌تواند Thread منتظر روی Barrier را آزاد کند.

برای طراحی سیستم‌های قابل توقف باید مسیر خروج از Barrier را نیز در نظر بگیرید.

برای مثال، می‌توان از `arrive_and_drop()` برای Threadهایی که تصمیم گرفته‌اند در Phaseهای آینده شرکت نکنند استفاده کرد؛ اما این عملیات نیز باید با قرارداد تعداد شرکت‌کنندگان هماهنگ باشد.

### **جلوگیری از Deadlock در Completion Function**

برای جلوگیری از Deadlock بهتر است Completion Function فقط عملیات کوتاه و غیرمسدودکننده انجام دهد.

عملیات مناسب عبارت‌اند از:

- تجمیع نتایج محاسباتی
- به‌روزرسانی وضعیت مشترک
- محاسبه مقادیر مورد نیاز Phase بعدی
- ثبت اطلاعات آماری ساده

در مقابل، انجام عملیات زیر معمولاً انتخاب مناسبی نیست:

- انتظار برای Threadهای دیگر
- انتظار روی Barrier دیگری با وابستگی چرخه‌ای
- عملیات I/O طولانی
- انتظار برای دریافت پاسخ از یک سرویس خارجی

### **استفاده از چند Barrier**

در بعضی الگوریتم‌های پیچیده، ممکن است چند Barrier برای هماهنگ‌سازی گروه‌های مختلف Thread مورد نیاز باشد.

برای مثال:

```
Group A:
    Worker 1
    Worker 2
    Worker 3
        |
     Barrier A
        |
        v
Group B:
    Worker 4
    Worker 5
    Worker 6
        |
     Barrier B
        |
        v
    Next phase
```

در چنین طراحی‌هایی باید وابستگی بین Barrierها به‌دقت بررسی شود.

اگر یک Thread برای عبور از Barrier اول منتظر Thread دیگری باشد که خود منتظر Barrier دوم است، ممکن است Deadlock ایجاد شود.

### **استفاده از std::barrier در محیط‌های Heterogeneous**

در برنامه‌هایی که محاسبات بین CPU و Acceleratorهایی مانند GPU تقسیم شده‌اند، `std::barrier` به‌تنهایی برای هماهنگ‌سازی تمام منابع کافی نیست.

این کلاس برای Threadهای مدل اجرای استاندارد ++C طراحی شده است و تضمینی برای Synchronization مستقیم عملیات GPU یا Deviceهای خارجی ارائه نمی‌دهد.

در این شرایط باید از مکانیزم‌های Synchronization مخصوص Runtime مربوط به Accelerator استفاده کرد.

## **نکات مربوط به Performance**

عملکرد `std::barrier` به عوامل مختلفی وابسته است:

- تعداد شرکت‌کنندگان
- تعداد Phaseها
- نحوه پیاده‌سازی کتابخانه استاندارد
- تعداد Coreهای پردازنده
- وضعیت Thread Scheduler
- مدت‌زمان انجام کار هر Thread
- میزان اختلاف زمان رسیدن Threadها به Barrier

### **هزینه انتظار**

اگر یک Thread خیلی زودتر از سایر Threadها به Barrier برسد، باید منتظر بماند.

در این مدت، منابع پردازشی ممکن است به Threadهای دیگر اختصاص پیدا کنند.

بنابراین عملکرد کل الگوریتم معمولاً تحت تأثیر کندترین شرکت‌کننده قرار می‌گیرد.

### **اهمیت تعادل بار**

فرض کنید چهار Thread وظایف زیر را انجام می‌دهند:

```
Thread 1: 10 ms
Thread 2: 12 ms
Thread 3: 11 ms
Thread 4: 80 ms
```

در این شرایط، سه Thread اول باید تا پایان کار Thread چهارم منتظر بمانند.

بنابراین حتی اگر Barrier بسیار بهینه پیاده‌سازی شده باشد، عدم تعادل بار باعث کاهش Performance خواهد شد.

### **هزینه Completion Function**

Completion Function بخشی از مسیر تکمیل Phase است.

بنابراین اگر عملیات سنگینی در آن انجام شود، تمام Threadهایی که منتظر تکمیل Phase هستند تحت تأثیر قرار می‌گیرند.

بهتر است عملیات Completion Function تا حد امکان کوتاه باشد.

### **Busy Waiting و Blocking**

استاندارد نحوه پیاده‌سازی داخلی انتظار در `std::barrier` را به یک الگوریتم مشخص محدود نمی‌کند.

یک Implementation ممکن است از ترکیبی از Atomic Operations، Waiting و سازوکارهای سیستم‌عامل استفاده کند.

بنابراین نباید فرض کرد که `std::barrier` در تمام Compilerها و Platformها دقیقاً با یک روش پیاده‌سازی شده است.

اگر Performance برای برنامه اهمیت زیادی دارد، باید رفتار واقعی Implementation مورد استفاده را Benchmark کنید.

## **تفاوت std::barrier با std::atomic**

کلاس `std::atomic` برای انجام عملیات اتمیک روی داده‌ها طراحی شده است.

اما `std::barrier` یک ابزار هماهنگ‌سازی در سطح مجموعه‌ای از Threadها است.

برای مثال، می‌توان با Atomic Counter تعداد عملیات تکمیل‌شده را محاسبه کرد؛ اما برای ساخت یک Barrier کامل به منطق اضافی مانند مدیریت شمارنده Phase، انتظار و آزادسازی Threadها نیاز خواهد بود.

بنابراین:

- `std::atomic` برای مدیریت داده‌های اتمیک مناسب است.
- `std::barrier` برای هماهنگی مرحله‌ای چند Thread مناسب است.

این دو ابزار می‌توانند در یک برنامه در کنار یکدیگر استفاده شوند.

## **وضعیت پشتیبانی در Compilerها**

قابلیت `std::barrier` بخشی از Standard Library استاندارد ++C20 است.

با این حال، پشتیبانی واقعی آن به نسخه Compiler و کتابخانه استاندارد مورد استفاده بستگی دارد.

برای استفاده از این قابلیت معمولاً باید موارد زیر را بررسی کنید:

- نسخه Compiler
- نسخه `libstdc++` یا `libc++` در محیط‌های مبتنی بر Clang و GCC
- نسخه Microsoft Standard Library در محیط MSVC
- فعال بودن استاندارد ++C20 در تنظیمات Build

نمونه دستورات رایج برای فعال کردن استاندارد ++C20:

**GCC:**

```
g++ -std=c++20 -pthread main.cpp -o app
```

**Clang:**

```
clang++ -std=c++20 -pthread main.cpp -o app
```

**MSVC:**

```
cl /std:c++20 /EHsc main.cpp
```

نکته مهم این است که فعال کردن گزینه استاندارد ++C20 به‌تنهایی تضمین نمی‌کند که تمام اجزای Standard Library در نسخه نصب‌شده پشتیبانی شوند.

برای بررسی دقیق‌تر می‌توان از Compiler Feature Test Macro استفاده کرد.

### **بررسی Feature Test Macro**

برای قابلیت `std::barrier`، ماکروی زیر تعریف شده است:

```
__cpp_lib_barrier
```

این ماکرو در صورت پشتیبانی کتابخانه استاندارد از قابلیت مربوطه در دسترس قرار می‌گیرد.

نمونه:

```
#include <barrier>
#include <iostream>

int main() {
#ifdef __cpp_lib_barrier
    std::cout << "std::barrier is supported\n";
#else
    std::cout << "std::barrier is not supported\n";
#endif
}
```

مقدار استاندارد Feature Test Macro برای این قابلیت برابر `201907L` است.

این ماکرو برای بررسی پشتیبانی کتابخانه استاندارد مناسب‌تر از بررسی صرف نسخه Compiler است.

## **جمع‌بندی**

کلاس `std::barrier` یکی از ابزارهای مهم Synchronization است که در استاندارد ++C20 به کتابخانه استاندارد اضافه شده است.

این کلاس برای هماهنگ‌سازی چند Thread در مراحل متوالی یک الگوریتم موازی طراحی شده است.

مهم‌ترین قابلیت‌های آن عبارت‌اند از:

- هماهنگ‌سازی چند Thread در هر Phase
- پشتیبانی از چند Phase متوالی
- امکان ثبت Arrival بدون انتظار فوری با `arrive()`
- امکان انتظار با `wait()`
- ترکیب Arrival و Wait با `arrive_and_wait()`
- حذف دائمی شرکت‌کننده از Phaseهای آینده با `arrive_and_drop()`
- اجرای Completion Function هنگام تکمیل Phase
- فراهم کردن ارتباط حافظه‌ای لازم برای انتقال نتایج بین Threadها

در انتخاب این ابزار باید به ثابت بودن تعداد شرکت‌کنندگان، احتمال عدم رسیدن یک Thread به Barrier، طول عمر اشیای مشترک و هزینه انتظار توجه کرد.

به‌طور کلی، اگر الگوریتم شما از چند مرحله محاسباتی تشکیل شده است و Threadها باید در پایان هر مرحله با یکدیگر هماهنگ شوند، `std::barrier` یکی از مناسب‌ترین و خواناترین ابزارهای استاندارد ++C20 برای پیاده‌سازی این نوع Synchronization است.

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>