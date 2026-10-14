<div align="center">

[🇺🇸 English](../../en/cpp14/shared_lock.md) | [🇮🇷 فارسی](./shared_lock.md)

</div>

---

# راهنمای جامع استفاده از `std::shared_lock` در ++C14

## معرفی و هدف `std::shared_lock`

قفل `std::shared_lock` یکی از ابزارهای Standard Library برای مدیریت قفل‌های اشتراکی در برنامه‌های چندنخی است که از ++C14 در دسترس قرار گرفته است. این کلاس به برنامه‌نویس اجازه می‌دهد یک Mutex سازگار با رابط قفل اشتراکی را به‌صورت RAII مدیریت کند؛ به این معنا که مالکیت قفل با طول عمر یک شیء مرتبط می‌شود و آزادسازی آن به‌طور خودکار انجام می‌گیرد.

مهم‌ترین کاربرد این کلاس، مدیریت دسترسی هم‌زمان چند Thread به داده‌هایی است که خواندن آن‌ها مجاز است، اما تغییرشان باید تحت کنترل یک قفل انحصاری انجام شود.

برای مثال، در یک Cache، چندین Thread ممکن است هم‌زمان به جست‌وجوی یک کلید بپردازند، در حالی که Thread دیگری گاهی اطلاعات جدیدی درج یا اطلاعات قبلی را حذف می‌کند. در چنین شرایطی، قفل اشتراکی می‌تواند اجازه دهد چند عملیات خواندن هم‌زمان انجام شوند، بدون آنکه ایمنی عملیات نوشتن به خطر بیفتد.

نکته مهم این است که `std::shared_lock` خودش یک Mutex نیست؛ بلکه یک ابزار مدیریت مالکیت قفل است. این کلاس معمولاً همراه با `std::shared_mutex` استفاده می‌شود که از ++C17 در دسترس قرار گرفته است. در ++C14، پیاده‌سازی چنین رفتاری به یک Mutex اشتراکی سازگار با Named Requirements کتابخانه استاندارد، مانند `std::shared_timed_mutex`، نیاز دارد.

## مفهوم قفل اشتراکی و قفل انحصاری

### تفاوت میان خواندن و نوشتن هم‌زمان

در طراحی سیستم‌های چندنخی، دو نوع دسترسی مهم وجود دارد:

- دسترسی اشتراکی (Shared Access): چند Thread می‌توانند به‌طور هم‌زمان قفل اشتراکی یک Mutex را در اختیار داشته باشند.
- دسترسی انحصاری (Exclusive Access): فقط یک Thread می‌تواند قفل انحصاری را در اختیار داشته باشد و این مالکیت با دسترسی‌های اشتراکی و انحصاری ناسازگار است.

این سازوکار امکان پیاده‌سازی الگوی Reader-Writer Lock را فراهم می‌کند؛ الگویی که در آن خوانندگان می‌توانند هم‌زمان کار کنند، اما نویسنده باید دسترسی‌های متعارض را متوقف کند.

جدول زیر رفتار متداول یک Mutex اشتراکی را نشان می‌دهد:

| وضعیت فعلی        | درخواست قفل اشتراکی | درخواست قفل انحصاری |
| ----------------- | ------------------- | ------------------- |
| بدون مالک         | مجاز                | مجاز                |
| دارای یک خواننده  | معمولاً مجاز        | باید منتظر بماند    |
| دارای چند خواننده | معمولاً مجاز        | باید منتظر بماند    |
| دارای نویسنده     | باید منتظر بماند    | باید منتظر بماند    |

واژه «معمولاً» در این جدول اهمیت دارد؛ زیرا سیاست زمان‌بندی، عدالت میان خوانندگان و نویسندگان و جلوگیری از گرسنگی (Starvation) به پیاده‌سازی Mutex وابسته است.

### تفاوت `std::shared_lock` با `std::unique_lock`

کلاس‌های `std::shared_lock` و `std::unique_lock` هر دو ابزارهای RAII برای مدیریت قفل هستند، اما نوع مالکیتی که مدیریت می‌کنند متفاوت است.

| ویژگی                                   | `std::shared_lock` | `std::unique_lock` |
| --------------------------------------- | ------------------ | ------------------ |
| نوع مالکیت                              | اشتراکی            | انحصاری            |
| قابلیت مالکیت هم‌زمان با سایر خوانندگان | بله                | خیر                |
| مناسب برای عملیات خواندن                | بله                | بله                |
| مناسب برای عملیات تغییر داده            | به‌تنهایی خیر      | بله                |
| قفل‌کردن با تأخیر                       | بله                | بله                |
| انتقال مالکیت با Move                   | بله                | بله                |
| آزادسازی خودکار                         | بله                | بله                |

توجه داشته باشید که نام `unique_lock` به معنای یکتابودن شیء قفل نیست؛ بلکه نشان می‌دهد مالکیت انحصاری Mutex را مدیریت می‌کند.

### تفاوت با `std::lock_guard`

کلاس `std::lock_guard` ابزار ساده‌تری برای مدیریت RAII است. این کلاس قفل را هنگام ساخت دریافت می‌کند و هنگام نابودی آزاد می‌کند، اما قابلیت‌هایی مانند انتقال مالکیت، قفل‌کردن با تأخیر و مدیریت وضعیت مالکیت را که در `std::unique_lock` و `std::shared_lock` وجود دارند، ارائه نمی‌دهد.

همچنین `std::lock_guard` به‌طور معمول قفل انحصاری دریافت می‌کند؛ در نتیجه، جایگزین مستقیم `std::shared_lock` برای مدیریت قفل اشتراکی نیست.

## معرفی Mutex مناسب در ++C14

### انتخاب `std::shared_timed_mutex`

در ++C14، نوع `std::shared_timed_mutex` یک انتخاب استاندارد برای پیاده‌سازی قفل اشتراکی است. این کلاس علاوه بر عملیات قفل انحصاری، عملیات قفل اشتراکی و قابلیت تلاش برای دریافت قفل با محدودیت زمانی را فراهم می‌کند.

برای استفاده از آن باید سرآیند `<shared_mutex>` را وارد کنید.

```
#include <shared_mutex>

std::shared_timed_mutex mutex;
```

در این نسخه از استاندارد، کد زیر نیز معتبر است و قفل اشتراکی را به‌صورت خودکار دریافت و آزاد می‌کند:

```
#include <iostream>
#include <shared_mutex>

class Configuration {
public:
    int getValue() const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        return value_;
    }

    void setValue(int value) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        value_ = value;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    int value_ = 0;
};
```

در این مثال، متغیر `mutex_` از نوع `mutable` است تا متد `const` بتواند آن را قفل کند. متد خواندن از مالکیت اشتراکی و متد تغییر مقدار از مالکیت انحصاری استفاده می‌کند.

نکته مهم این است که برای استفاده از `std::unique_lock` باید سرآیند `<mutex>` را نیز وارد کنید. نسخه کامل و مستقل این مثال به شکل زیر است:

```
#include <iostream>
#include <mutex>
#include <shared_mutex>

class Configuration {
public:
    int getValue() const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        return value_;
    }

    void setValue(int value) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);
        value_ = value;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    int value_ = 0;
};

int main() {
    Configuration config;

    config.setValue(42);
    std::cout << config.getValue() << '\n';
}
```

در ++C17 نوع `std::shared_mutex` معرفی شد. این نوع برای دسترسی اشتراکی و انحصاری طراحی شده، اما قابلیت قفل‌کردن زمان‌دار را ندارد. بنابراین، اگر هدف استفاده از `std::shared_lock` در ++C14 باشد، باید نوع Mutex متناسب با آن نسخه انتخاب شود.

## نحوه ساخت و دریافت قفل

کلاس `std::shared_lock` چند روش برای مدیریت زمان دریافت قفل ارائه می‌دهد. انتخاب روش مناسب به این بستگی دارد که عملیات باید فوراً قفل را دریافت کند، بعداً آن را بگیرد یا در صورت مشغول‌بودن Mutex ادامه دهد.

### دریافت فوری قفل

ساده‌ترین روش، ارسال مرجع Mutex به سازنده است:

```
std::shared_timed_mutex mutex;

void readData() {
    std::shared_lock<std::shared_timed_mutex> lock(mutex);

    // Access shared data here.
}
```

در این حالت، سازنده عملیات `lock_shared()` را روی Mutex انجام می‌دهد. اگر قفل انحصاری در اختیار Thread دیگری باشد، فراخوانی منتظر می‌ماند تا مالکیت اشتراکی قابل دریافت شود.

با خروج از محدوده `readData`، شیء `lock` نابود می‌شود و در صورت مالکیت قفل، عملیات آزادسازی اشتراکی انجام می‌گیرد.

### دریافت قفل با تأخیر

گاهی لازم است ابتدا شیء مدیریت قفل ساخته شود و دریافت واقعی قفل در مرحله دیگری اتفاق بیفتد. برای این کار از `std::defer_lock` استفاده می‌شود:

```
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void readData() {
    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::defer_lock
    );

    // Perform preparatory work here.

    lock.lock();

    // Access shared data here.
}
```

در این مثال، هنگام ساخت شیء هیچ قفلی دریافت نمی‌شود. دریافت قفل تنها پس از فراخوانی `lock()` انجام می‌شود.

این روش هنگامی مفید است که ساختار کنترل برنامه به دریافت قفل در نقطه مشخصی نیاز دارد. بااین‌حال، نباید کار طولانی یا عملیات وابسته به داده محافظت‌شده را پیش از دریافت قفل انجام داد، اگر صحت آن عملیات به ثبات داده وابسته باشد.

### تلاش بدون انتظار برای دریافت قفل

اگر برنامه نمی‌خواهد برای دریافت قفل منتظر بماند، می‌تواند از `std::try_to_lock` استفاده کند:

```
#include <iostream>
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void tryReadData() {
    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::try_to_lock
    );

    if (lock.owns_lock()) {
        std::cout << "Lock acquired\n";
        // Access shared data here.
    } else {
        std::cout << "Lock unavailable\n";
    }
}
```

در این روش، سازنده تلاش می‌کند قفل اشتراکی را دریافت کند و بدون انتظار برای آزادشدن Mutex بازمی‌گردد. موفقیت عملیات از طریق `owns_lock()` قابل بررسی است.

شکست این عملیات لزوماً به معنای وجود یک نویسنده فعال نیست؛ ممکن است علت آن سیاست پیاده‌سازی Mutex یا وضعیت فعلی رقابت میان Threadها باشد.

### دریافت قفل با محدودیت زمانی

یکی از قابلیت‌های مهم `std::shared_lock` در ++C14، پشتیبانی از تلاش زمان‌دار است. این قابلیت برای برنامه‌هایی مفید است که نمی‌توانند مدت نامحدودی منتظر بمانند.

نمونه زیر از سازنده زمان‌دار استفاده می‌کند:

```
#include <chrono>
#include <iostream>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void readWithTimeout() {
    using namespace std::chrono_literals;

    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::chrono::milliseconds(100)
    );

    if (lock.owns_lock()) {
        std::cout << "Lock acquired\n";
        // Access shared data here.
    } else {
        std::cout << "Timeout or lock acquisition failure\n";
    }
}
```

عبارت `std::chrono_literals` و پسوندهایی مانند `100ms` در ++C14 قابل استفاده‌اند، اما در این مثال برای سازگاری روشن‌تر با محیط‌های مختلف، از `std::chrono::milliseconds(100)` استفاده شده است.

اگر بخواهید از پسوندهای زمانی استفاده کنید، نسخه زیر نیز معتبر است:

```
#include <chrono>
#include <mutex>
#include <shared_mutex>

using namespace std::chrono_literals;

std::shared_timed_mutex mutex;

void readWithTimeout() {
    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, 100ms
    );
}
```

باید توجه داشت که زمان تعیین‌شده تضمین نمی‌کند تابع دقیقاً در همان لحظه بازگردد. زمان‌بندی سیستم‌عامل و شرایط اجرایی می‌توانند باعث تأخیر بیشتر شوند.

### استفاده از زمان مطلق

در مواردی که یک Deadline مشخص دارید، استفاده از `std::chrono::time_point` مناسب‌تر است. سازنده زمان‌دار می‌تواند یک نقطه زمانی مطلق را دریافت کند:

```
#include <chrono>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void readUntilDeadline() {
    const auto deadline =
        std::chrono::steady_clock::now() +
        std::chrono::milliseconds(200);

    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, deadline
    );

    if (lock.owns_lock()) {
        // Access shared data here.
    }
}
```

در این مثال، `std::chrono::steady_clock` برای اندازه‌گیری فاصله زمانی مناسب است؛ زیرا برای حرکت یکنواخت زمان طراحی شده و تحت تأثیر تغییر ساعت تقویمی سیستم قرار نمی‌گیرد.

## بررسی متدهای اصلی `std::shared_lock`

کلاس `std::shared_lock` علاوه بر سازنده و مخرب، متدهایی برای دریافت، آزادسازی، انتقال و بررسی وضعیت قفل دارد.

| متد یا عملیات      | کاربرد                                      |
| ------------------ | ------------------------------------------- |
| `lock()`           | دریافت قفل اشتراکی به‌صورت مسدودکننده       |
| `try_lock()`       | تلاش برای دریافت قفل بدون انتظار            |
| `try_lock_for()`   | تلاش برای دریافت قفل تا پایان یک بازه زمانی |
| `try_lock_until()` | تلاش برای دریافت قفل تا یک Deadline         |
| `unlock()`         | آزادسازی قفل اشتراکی                        |
| `owns_lock()`      | بررسی مالکیت فعلی قفل                       |
| `operator bool()`  | بررسی ضمنی مالکیت قفل                       |
| `mutex()`          | دسترسی به اشاره‌گر Mutex مرتبط              |
| `release()`        | جداکردن Mutex از شیء بدون آزادکردن قفل      |
| `swap()`           | جابه‌جایی وضعیت دو شیء مدیریت قفل           |

قابلیت‌های زمان‌دار تنها زمانی معتبرند که نوع Mutex شرایط لازم برای عملیات اشتراکی زمان‌دار را برآورده کند. برای `std::shared_timed_mutex` این شرط برقرار است.

### مدیریت دستی دریافت و آزادسازی

با وجود آنکه RAII روش توصیه‌شده است، گاهی لازم می‌شود قفل را پیش از پایان محدوده آزاد کنید:

```
#include <iostream>
#include <shared_mutex>

std::shared_timed_mutex mutex;
int value = 42;

void readValue() {
    std::shared_lock<std::shared_timed_mutex> lock(mutex);

    const int localValue = value;

    lock.unlock();

    std::cout << localValue << '\n';
}
```

در این مثال، مقدار موردنیاز در یک متغیر محلی ذخیره می‌شود و سپس قفل آزاد می‌شود. بنابراین، عملیات خروجی نیازی به نگه‌داشتن قفل ندارد.

پس از `unlock()`، شیء `lock` همچنان به Mutex مرتبط است، اما دیگر مالک قفل نیست. اگر دوباره به قفل نیاز باشد، می‌توان از `lock()` یا `try_lock()` استفاده کرد.

فراخوانی `unlock()` زمانی مجاز است که شیء واقعاً مالک قفل باشد. تلاش برای آزادسازی قفلی که مالک آن نیستید، رفتار تعریف‌نشده ایجاد می‌کند.

### بررسی وضعیت با `owns_lock()`

برای مدیریت صحیح مسیرهای مختلف اجرا، بهتر است وضعیت قفل را صریح بررسی کنید:

```
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void process() {
    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::defer_lock
    );

    if (!lock.try_lock()) {
        return;
    }

    // The shared lock is owned here.
}
```

این الگو از ادامه اجرای بخش حساس برنامه در شرایطی که قفل دریافت نشده است جلوگیری می‌کند.

### تفاوت `release()` با `unlock()`

این دو عملیات رفتار متفاوتی دارند و نباید با یکدیگر اشتباه گرفته شوند.

- متد `unlock()` مالکیت قفل اشتراکی را خاتمه می‌دهد، اما ارتباط شیء مدیریت قفل با Mutex باقی می‌ماند.
- متد `release()` فقط اشاره‌گر داخلی Mutex را از شیء جدا می‌کند و خود قفل را آزاد نمی‌کند.

نمونه زیر نشان می‌دهد که استفاده از `release()` مسئولیت مدیریت قفل را به کد فراخواننده منتقل می‌کند:

```
#include <shared_mutex>

std::shared_timed_mutex mutex;

void transferResponsibility() {
    std::shared_lock<std::shared_timed_mutex> lock(mutex);

    std::shared_timed_mutex* rawMutex = lock.release();

    rawMutex->unlock_shared();
}
```

این مثال صرفاً برای توضیح رفتار `release()` است و در کد معمول برنامه توصیه نمی‌شود؛ زیرا مسئولیت آزادسازی قفل به‌صورت دستی به فراخواننده منتقل شده است.

اگر آزادسازی فراموش شود، ممکن است Threadهای دیگر برای همیشه منتظر بمانند. همچنین، استفاده نادرست از اشاره‌گر جداشده می‌تواند به رفتار تعریف‌نشده منجر شود.

## مدیریت مالکیت و ویژگی‌های Move

### دلیل Move-only بودن کلاس

کلاس `std::shared_lock` قابل کپی‌کردن نیست، اما از Move Construction و Move Assignment پشتیبانی می‌کند. این طراحی مانع از آن می‌شود که دو شیء مستقل، بدون هماهنگی، مالکیت یک قفل واحد را ادعا کنند.

نمونه زیر انتقال مالکیت قفل را نشان می‌دهد:

```
#include <mutex>
#include <shared_mutex>
#include <utility>

std::shared_timed_mutex mutex;

void transferLock() {
    std::shared_lock<std::shared_timed_mutex> first(mutex);

    std::shared_lock<std::shared_timed_mutex> second(
        std::move(first)
    );

    // second owns the shared lock.
    // first no longer owns the lock.
}
```

پس از انتقال موفق، شیء `second` مالک قفل است و در پایان محدوده آن را آزاد می‌کند. شیء `first` دیگر مالک قفل نیست و می‌تواند نابود شود یا در صورت نیاز برای عملیات دیگری مورد استفاده قرار گیرد.

استفاده از `std::move` به‌خودی‌خود قفل را آزاد نمی‌کند؛ این عبارت امکان انتخاب عملیات انتقال را فراهم می‌کند.

### انتقال مالکیت میان توابع

انتقال قفل به تابع دیگری در شرایطی مفید است که بخواهید محدوده مدیریت قفل را از یک تابع به تابع دیگر منتقل کنید.

```
#include <mutex>
#include <shared_mutex>
#include <utility>

using ReadLock = std::shared_lock<std::shared_timed_mutex>;

std::shared_timed_mutex mutex;

void consumeLock(ReadLock lock) {
    // The function owns the lock for its execution.
}

void caller() {
    ReadLock lock(mutex);

    consumeLock(std::move(lock));

    // The local lock no longer owns the mutex.
}
```

این الگو زمانی مفید است که تابع دریافت‌کننده واقعاً باید مالکیت قفل را در اختیار داشته باشد. اگر هدف صرفاً محافظت از داده در طول اجرای یک تابع باشد، معمولاً ساده‌تر است که تابع خودش قفل را دریافت کند.

### انتقال مالکیت با انتساب

عملگر انتساب انتقالی نیز در دسترس است. در این حالت، اگر شیء مقصد قبلاً مالک قفل دیگری باشد، ابتدا مالکیت قبلی آن خاتمه می‌یابد و سپس وضعیت شیء مبدأ منتقل می‌شود.

```
#include <mutex>
#include <shared_mutex>
#include <utility>

std::shared_timed_mutex firstMutex;
std::shared_timed_mutex secondMutex;

void moveAssign() {
    std::shared_lock<std::shared_timed_mutex> first(firstMutex);

    std::shared_lock<std::shared_timed_mutex> second(
        secondMutex
    );

    second = std::move(first);
}
```

در این مثال، انتساب انتقالی باعث آزادشدن قفل قبلی `second` و انتقال مالکیت قفل `first` به آن می‌شود.

باید دقت کنید که این عملیات می‌تواند باعث آزادشدن یک قفل و انتقال قفل دیگری شود؛ بنابراین، در طراحی‌های دارای چند Mutex، ترتیب قفل‌گیری و آزادسازی اهمیت زیادی دارد.

## بررسی سازنده‌های ویژه

### استفاده از `std::adopt_lock`

تگ `std::adopt_lock` برای حالتی است که Thread از قبل قفل اشتراکی Mutex را دریافت کرده باشد و بخواهد مدیریت آن را به یک شیء `std::shared_lock` بسپارد.

```
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void adoptExistingLock() {
    mutex.lock_shared();

    std::shared_lock<std::shared_timed_mutex> lock(
        mutex, std::adopt_lock
    );

    // The lock object now manages the existing shared lock.
}
```

در این مثال، سازنده قفل جدیدی دریافت نمی‌کند؛ بلکه فرض می‌کند قفل اشتراکی قبلاً توسط Thread جاری گرفته شده است.

این روش به دقت زیادی نیاز دارد. اگر پیش‌شرط مالکیت قفل برقرار نباشد، رفتار برنامه تعریف‌نشده است. به همین دلیل، دریافت مستقیم قفل از طریق سازنده معمولی `std::shared_lock` در بیشتر برنامه‌ها انتخاب امن‌تری است.

### استفاده از سازنده بدون Mutex

کلاس امکان ساخت یک شیء خالی را نیز فراهم می‌کند:

```
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;

void deferredInitialization() {
    std::shared_lock<std::shared_timed_mutex> lock;

    lock = std::shared_lock<std::shared_timed_mutex>(
        mutex, std::defer_lock
    );

    lock.lock();
}
```

شیء خالی نه Mutex مرتبطی دارد و نه مالک قفل است. این قابلیت برای مدیریت وضعیت‌های پویا و ساختارهایی مفید است که زمان انتخاب Mutex در آن‌ها از قبل مشخص نیست.

## کاربرد عملی در طراحی Cache

### پیاده‌سازی Cache با خواندن هم‌زمان

یکی از کاربردهای رایج قفل اشتراکی، محافظت از ساختارهایی است که خواندن از آن‌ها بسیار بیشتر از نوشتن است.

در مثال زیر، یک Cache ساده با `std::unordered_map` پیاده‌سازی شده است. چند Thread می‌توانند هم‌زمان مقدار یک کلید را جست‌وجو کنند، اما درج یا حذف اطلاعات با قفل انحصاری انجام می‌شود.

```
#include <mutex>
#include <optional>
#include <shared_mutex>
#include <string>
#include <unordered_map>

class Cache {
public:
    std::optional<std::string> get(
        const std::string& key
    ) const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);

        const auto it = data_.find(key);

        if (it == data_.end()) {
            return std::nullopt;
        }

        return it->second;
    }

    void set(std::string key, std::string value) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);

        data_[std::move(key)] = std::move(value);
    }

    bool erase(const std::string& key) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);

        return data_.erase(key) != 0;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    std::unordered_map<std::string, std::string> data_;
};
```

این مثال از نظر طراحی چند نکته مهم دارد:

- عملیات جست‌وجو با قفل اشتراکی انجام می‌شود؛ بنابراین، چند Thread می‌توانند هم‌زمان `find()` را روی Map اجرا کنند.
- عملیات درج و حذف از قفل انحصاری استفاده می‌کنند؛ در نتیجه، تغییرات ساختار داخلی Map با جست‌وجوهای هم‌زمان تداخل ندارند.
- متد `get` یک نسخه مستقل از مقدار را برمی‌گرداند، نه یک Reference به داده داخلی Cache.
- متد `get` از `std::optional` استفاده می‌کند که از ++C17 در دسترس است. بنابراین، این نسخه از مثال به ++C17 یا جدیدتر نیاز دارد، نه ++C14.

برای نسخه سازگار با ++C14، می‌توانید نتیجه جست‌وجو را از طریق پارامتر خروجی و یک مقدار بازگشتی `bool` مدیریت کنید:

```
#include <mutex>
#include <shared_mutex>
#include <string>
#include <unordered_map>
#include <utility>

class Cache {
public:
    bool get(const std::string& key, std::string& result) const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);

        const auto it = data_.find(key);

        if (it == data_.end()) {
            return false;
        }

        result = it->second;
        return true;
    }

    void set(std::string key, std::string value) {
        std::unique_lock<std::shared_timed_mutex> lock(mutex_);

        data_[std::move(key)] = std::move(value);
    }

private:
    mutable std::shared_timed_mutex mutex_;
    std::unordered_map<std::string, std::string> data_;
};
```

این نسخه فقط از امکانات استاندارد موجود در ++C14 استفاده می‌کند.

### چرا برگرداندن Reference می‌تواند خطرناک باشد؟

اگر Cache یک Reference یا Pointer به داده داخلی خود برگرداند و قفل پیش از استفاده از آن آزاد شود، Thread دیگری ممکن است داده را حذف یا تغییر دهد.

برای مثال، وجود یک قفل در داخل متد `get` به‌تنهایی امنیت Reference بازگشتی را تضمین نمی‌کند:

```
const std::string* getUnsafe(const std::string& key) const {
    std::shared_lock<std::shared_timed_mutex> lock(mutex_);

    const auto it = data_.find(key);

    if (it == data_.end()) {
        return nullptr;
    }

    return &it->second;
}
```

این تابع صرفاً نمونه‌ای از یک طراحی ناامن است. پس از خروج از تابع، قفل آزاد می‌شود؛ بنابراین، استفاده از Pointer بازگشتی بدون سازوکار محافظت دیگری ممکن است با حذف عنصر یا تغییر هم‌زمان آن تداخل پیدا کند.

برای طراحی Cache معمولاً بهتر است یکی از این راهکارها انتخاب شود:

- برگرداندن نسخه مستقل از داده؛
- استفاده از یک Handle مالکیت‌دار که طول عمر قفل را کنترل کند؛
- استفاده از ساختار داده‌ای با سیاست مالکیت و طول عمر مشخص؛
- طراحی APIای که عملیات موردنیاز را در محدوده قفل انجام دهد.

انتخاب مناسب به هزینه کپی داده، اندازه مقادیر و نیازهای کارایی بستگی دارد.

## نکات مهم درباره Thread Safety

### محافظت از تمام دسترسی‌های متعارض

برای جلوگیری از Data Race، تمام دسترسی‌های متعارض به یک شیء مشترک باید تحت یک سازوکار همگام‌سازی سازگار انجام شوند.

اگر یک Thread با `std::shared_lock` داده را بخواند، اما Thread دیگری همان داده را بدون قفل تغییر دهد، صرف وجود قفل در Thread اول کافی نیست.

```
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;
int value = 0;

void safeRead() {
    std::shared_lock<std::shared_timed_mutex> lock(mutex);
    const int localValue = value;
}

void safeWrite() {
    std::unique_lock<std::shared_timed_mutex> lock(mutex);
    value = 42;
}
```

در این مثال، خواندن و نوشتن از یک Mutex مشترک استفاده می‌کنند و دسترسی متعارض به `value` به‌درستی همگام‌سازی شده است.

در مقابل، اگر `safeWrite` مستقیماً مقدار را تغییر دهد، در حالی که Thread دیگری در حال خواندن آن است، امکان وقوع Data Race وجود خواهد داشت.

### استفاده از قفل در متدهای `const`

در بسیاری از کلاس‌ها، متدهای `const` صرفاً وضعیت منطقی شیء را تغییر نمی‌دهند، اما برای خواندن ایمن داده به همگام‌سازی نیاز دارند.

در چنین مواردی، تعریف Mutex به‌صورت `mutable` یک الگوی رایج است:

```
#include <shared_mutex>

class Counter {
public:
    int get() const {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        return value_;
    }

private:
    mutable std::shared_timed_mutex mutex_;
    int value_ = 0;
};
```

استفاده از `mutable` به این معنا نیست که داده محافظت‌شده را می‌توان بدون قفل تغییر داد؛ این کلمه صرفاً اجازه می‌دهد عضو Mutex در یک متد `const` مدیریت شود.

### محدودیت در بازگرفتن قفل

Mutexهای اشتراکی استاندارد، از جمله `std::shared_timed_mutex`، Recursive نیستند. یک Thread نباید بدون رعایت قرارداد Mutex، دوباره از همان Mutex مالکیت اشتراکی یا انحصاری دریافت کند.

برای مثال، اگر یک متد قفل اشتراکی بگیرد و سپس متدی دیگر را فراخوانی کند که دوباره همان Mutex را قفل می‌کند، ممکن است برنامه Deadlock شود یا رفتاری تعریف‌نشده داشته باشد.

```
#include <mutex>
#include <shared_mutex>

class Data {
public:
    void outer() {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        inner();
    }

    void inner() {
        std::shared_lock<std::shared_timed_mutex> lock(mutex_);
        // Access data here.
    }

private:
    std::shared_timed_mutex mutex_;
};
```

این مثال یک طراحی نادرست است؛ زیرا `outer()` و `inner()` هر دو برای دریافت قفل اشتراکی از یک Mutex استفاده می‌کنند. باید مالکیت قفل در سطح API مشخص شود؛ برای مثال، می‌توان متد داخلی را طوری طراحی کرد که فرض کند قفل از قبل گرفته شده است و خودش قفل جدیدی نگیرد.

## استفاده همراه با Condition Variable

### انتظار برای تغییر وضعیت داده

در بعضی برنامه‌ها، یک Thread باید تا زمان برقرارشدن شرطی منتظر بماند؛ برای مثال، تا زمانی که یک صف خالی نباشد.

کلاس `std::condition_variable` به `std::unique_lock<std::mutex>` نیاز دارد و مستقیماً با `std::shared_lock` کار نمی‌کند. اما `std::condition_variable_any` می‌تواند با انواع قفل سازگار، از جمله `std::shared_lock`، استفاده شود.

نمونه زیر نحوه انتظار اشتراکی برای برقرارشدن یک شرط را نشان می‌دهد:

```
#include <condition_variable>
#include <mutex>
#include <shared_mutex>

std::shared_timed_mutex mutex;
std::condition_variable_any condition;
bool ready = false;

void waitForReady() {
    std::shared_lock<std::shared_timed_mutex> lock(mutex);

    condition.wait(lock, [] {
        return ready;
    });

    // The predicate is true while the shared lock is held.
}

void setReady() {
    {
        std::unique_lock<std::shared_timed_mutex> lock(mutex);
        ready = true;
    }

    condition.notify_all();
}
```

در این مثال، متد `wait` قفل را هنگام انتظار آزاد می‌کند و پس از بیدارشدن، پیش از بازگشت، دوباره آن را دریافت می‌کند. استفاده از Predicate نیز باعث می‌شود بیدارشدن‌های کاذب به‌درستی مدیریت شوند.

نکته مهم این است که `std::condition_variable_any` نسبت به `std::condition_variable` هزینه و پیچیدگی بیشتری دارد. بنابراین، اگر به انتظار با قفل انحصاری نیاز دارید، استفاده از `std::condition_variable` و `std::unique_lock` معمولاً انتخاب ساده‌تری است.

## مشکلات رایج و راهکارها

### نگه‌داشتن طولانی قفل

قفل اشتراکی به خوانندگان اجازه می‌دهد هم‌زمان فعالیت کنند، اما نویسندگان همچنان باید منتظر آزادشدن تمام مالکیت‌های اشتراکی متعارض بمانند.

اگر یک Thread قفل را در طول یک عملیات طولانی، درخواست شبکه یا عملیات ورودی و خروجی نگه دارد، ممکن است نویسندگان با تأخیر زیادی مواجه شوند.

راهکار مناسب این است که داده لازم را در محدوده قفل بخوانید، نسخه محلی آن را بسازید و سپس قفل را آزاد کنید؛ البته تنها زمانی که کپی‌کردن داده و ادامه کار بدون قفل از نظر معنایی امن باشد.

### فرض بر سریع‌تر بودن قفل اشتراکی

استفاده از قفل اشتراکی الزاماً عملکرد را بهبود نمی‌دهد. این نوع Mutex ممکن است برای مدیریت تعداد خوانندگان، ثبت مالکیت و هماهنگی با نویسندگان هزینه بیشتری نسبت به `std::mutex` داشته باشد.

اگر بخش بحرانی بسیار کوتاه باشد یا تعداد خوانندگان هم‌زمان کم باشد، یک Mutex ساده ممکن است عملکرد بهتری ارائه دهد.

بهترین تصمیم باید با اندازه‌گیری در محیط واقعی گرفته شود؛ نه صرفاً بر اساس این فرض که تعداد بیشتر خوانندگان به معنای سرعت بیشتر است.

### گرسنگی نویسندگان

بسته به پیاده‌سازی، ممکن است ورود پیوسته خوانندگان باعث تأخیر طولانی در دریافت قفل انحصاری شود. استاندارد، یک سیاست عمومی و قابل‌حمل برای تضمین عدالت میان خوانندگان و نویسندگان تعیین نمی‌کند.

اگر زمان پاسخ‌گویی نویسندگان اهمیت زیادی دارد، باید رفتار Mutex در محیط هدف اندازه‌گیری شود و در صورت نیاز، از سیاست‌های زمان‌بندی یا طراحی‌های جایگزین استفاده شود.

### تلاش برای ارتقای قفل

کلاس `std::shared_lock` متدی برای تبدیل مستقیم مالکیت اشتراکی به انحصاری ارائه نمی‌دهد.

اگر Thread بخواهد پس از خواندن داده آن را تغییر دهد، نباید بدون طراحی مناسب، قفل اشتراکی را نگه دارد و سپس همان Mutex را به‌صورت انحصاری قفل کند؛ چنین کاری می‌تواند باعث Deadlock یا رفتار تعریف‌نشده شود.

الگوی معمول این است که قفل اشتراکی آزاد شود و سپس قفل انحصاری دریافت شود. البته این کار باعث ایجاد یک فاصله زمانی می‌شود که طی آن داده ممکن است تغییر کند. در نتیجه، شرط موردنظر باید پس از دریافت قفل انحصاری دوباره بررسی شود.

### فراموش‌کردن طول عمر Mutex

شیء `std::shared_lock` یک اشاره‌گر یا ارتباط داخلی به Mutex را نگه می‌دارد؛ بنابراین، Mutex باید در تمام مدت عمر شیء مدیریت قفل معتبر باقی بماند.

نابودشدن Mutex در حالی که یک Thread هنوز مالک آن است یا یک شیء مدیریت قفل هنوز به آن وابسته است، می‌تواند رفتار تعریف‌نشده ایجاد کند.

در طراحی کلاس‌ها، معمولاً بهتر است Mutex به‌عنوان عضوی از همان شیئی تعریف شود که داده محافظت‌شده را در خود نگه می‌دارد. این کار مدیریت طول عمر را ساده‌تر می‌کند.

## مقایسه با روش‌های جایگزین

### انتخاب میان `std::mutex` و Mutex اشتراکی

| ویژگی                         | `std::mutex`         | `std::shared_timed_mutex`         |
| ----------------------------- | -------------------- | --------------------------------- |
| مالکیت انحصاری                | بله                  | بله                               |
| مالکیت اشتراکی                | خیر                  | بله                               |
| پشتیبانی از خواندن هم‌زمان    | خیر                  | بله                               |
| پشتیبانی استاندارد از Timeout | خیر                  | بله                               |
| هزینه احتمالی همگام‌سازی      | معمولاً کمتر         | ممکن است بیشتر باشد               |
| مناسب برای                    | بخش‌های بحرانی عمومی | بار کاری با خواندن هم‌زمان فراوان |

اگر فقط یک Thread در هر لحظه باید به داده دسترسی داشته باشد، `std::mutex` معمولاً گزینه ساده‌تر و مناسبی است.

اگر تعداد خوانندگان زیاد است و عملیات خواندن بدون تغییر وضعیت مشترک انجام می‌شود، قفل اشتراکی می‌تواند مفید باشد؛ اما سود عملکردی آن باید اندازه‌گیری شود.

### تفاوت با `std::shared_mutex` در ++C17

نوع `std::shared_mutex` در ++C17 معرفی شده است و از مالکیت اشتراکی و انحصاری پشتیبانی می‌کند. برخلاف `std::shared_timed_mutex`، این کلاس عملیات قفل‌کردن زمان‌دار را ارائه نمی‌دهد.

در نتیجه، انتخاب میان این دو نوع به نیاز برنامه و نسخه استاندارد بستگی دارد:

- در ++C14، برای استفاده از `std::shared_lock` با یک Mutex اشتراکی استاندارد، `std::shared_timed_mutex` انتخاب مناسبی است.
- در ++C17 و نسخه‌های جدیدتر، اگر به Timeout نیاز ندارید، `std::shared_mutex` نیز گزینه‌ای مناسب است.
- اگر به سیاست خاصی برای عدالت، اولویت نویسنده یا زمان‌بندی نیاز دارید، باید این نیاز را جداگانه بررسی کنید؛ نام نوع Mutex به‌تنهایی چنین تضمینی نمی‌دهد.

## نکات کامپایل و سازگاری

برای کامپایل برنامه‌ای که از `std::shared_lock` و `std::shared_timed_mutex` استفاده می‌کند، باید کامپایلر و کتابخانه استاندارد از ++C14 و Thread Support Library پشتیبانی کنند.

برای نمونه، در محیط‌های سازگار با GCC می‌توانید از دستور زیر استفاده کنید:

```
g++ -std=c++14 -pthread main.cpp -o main
```

گزینه `-std=c++14` نسخه زبان را مشخص می‌کند و `-pthread` پشتیبانی لازم برای Threadها را در محیط‌های متناسب فعال می‌کند.

در محیط‌های دیگر، روش فعال‌سازی Thread Support ممکن است متفاوت باشد؛ بنابراین، تنظیمات پروژه و مستندات کامپایلر نیز باید بررسی شوند.

## جمع‌بندی نهایی

کلاس `std::shared_lock` ابزاری برای مدیریت مالکیت اشتراکی Mutex در ++C14 است که با الگوی RAII، انتقال مالکیت و روش‌های مختلف دریافت قفل، مدیریت همگام‌سازی را ساده‌تر می‌کند.

برای استفاده حرفه‌ای از این کلاس، نکات زیر اهمیت ویژه‌ای دارند:

- در ++C14 از Mutex سازگار، مانند `std::shared_timed_mutex`، استفاده کنید.
- برای خواندن هم‌زمان از قفل اشتراکی و برای تغییر داده از قفل انحصاری استفاده کنید.
- تمام دسترسی‌های متعارض به داده را تحت یک سازوکار همگام‌سازی سازگار قرار دهید.
- از بازگرفتن قفل غیر بازگشتی در یک Thread و از تلاش برای ارتقای مستقیم قفل اشتراکی اجتناب کنید.
- طول عمر Mutex را از طول عمر شیء مدیریت قفل کوتاه‌تر نکنید.
- برای APIهایی که Reference یا Pointer به داده داخلی برمی‌گردانند، طول عمر و ایمنی دسترسی را به‌دقت طراحی کنید.
- مزیت عملکردی قفل اشتراکی را با Benchmark در شرایط واقعی بسنجید.

در نهایت، `std::shared_lock` زمانی بیشترین ارزش را دارد که برنامه واقعاً از دسترسی هم‌زمان چند خواننده سود ببرد و طراحی قفل‌گیری، طول عمر داده و رفتار نویسندگان نیز به‌درستی مدیریت شده باشد.

## منابع تکمیلی

- [مستندات ](https://en.cppreference.com/w/cpp/thread/shared_lock)[`std::shared_lock`](https://en.cppreference.com/w/cpp/thread/shared_lock)[ در cppreference](https://en.cppreference.com/w/cpp/thread/shared_lock)
- [مستندات ](https://en.cppreference.com/w/cpp/thread/shared_timed_mutex)[`std::shared_timed_mutex`](https://en.cppreference.com/w/cpp/thread/shared_timed_mutex)[ در cppreference](https://en.cppreference.com/w/cpp/thread/shared_timed_mutex)
- [مستندات سرآیند ](https://en.cppreference.com/w/cpp/header/shared_mutex)[`<shared_mutex>`](https://en.cppreference.com/w/cpp/header/shared_mutex)[ در cppreference](https://en.cppreference.com/w/cpp/header/shared_mutex)
- [متن استاندارد ++C و مشخصات Thread Support Library](https://eel.is/c++draft/thread)

---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>