<div align="right">

[🇺🇸 English](./thread.md) | [🇮🇷 فارسی](../../fa/cpp11/thread.md)

</div>

# Chrono

> کتابخانه `chrono` مجموعه‌ای از کلاس‌ها و توابع را برای **اندازه‌گیری زمان، نمایش مدت‌زمان و کار با ساعت‌ها (Clock)** فراهم می‌کند. این کتابخانه روشی قابل‌حمل و دقیق برای انجام عملیات مرتبط با زمان در برنامه‌های C++ ارائه می‌دهد.

- فراهم کردن ساعت‌ها، مدت‌زمان‌ها و نقاط زمانی برای اندازه‌گیری زمان
- پشتیبانی از عملیات زمانی با دقت بالا و مستقل از سیستم‌عامل

## از کدام Clock باید استفاده کنیم؟

- `std::chrono::steady_clock:` **همیشه برای اندازه‌گیری زمان سپری‌شده (Stopwatch) از این Clock استفاده کنید.** این ساعت یکنواخت (Monotonic) است؛ یعنی حتی اگر زمان سیستم‌عامل تغییر کند، زمان آن به عقب برنمی‌گردد.

- `std::chrono::system_clock:` برای زمان واقعی سیستم یا همان **Wall Clock** استفاده می‌شود؛ برای مثال زمانی که می‌خواهید تاریخ و ساعت فعلی را به کاربر نمایش دهید. این Clock با زمان سیستم هماهنگ است و ممکن است به جلو یا عقب تغییر کند.

- `std::chrono::high_resolution_clock:` معمولاً یک نام مستعار (Alias) برای یکی از Clockهای بالا است. برای کدنویسی مقاوم و قابل‌اعتماد، بهتر است برای اندازه‌گیری زمان از `steady_clock` استفاده کنید.

```cpp
#include <chrono>
#include <iostream>

using namespace std;
using namespace std::chrono;

int main() {

    auto start = high_resolution_clock::now();

    // کدی که می‌خواهیم زمان اجرای آن را اندازه‌گیری کنیم

    for (int i = 0; i < 1000000; i++);

    auto end = high_resolution_clock::now();

    auto duration = duration_cast<microseconds>(end - start);

    cout << "Execution Time: "
         << duration.count()
         << " microseconds";

}
```

**توضیح:**

برنامه زمان شروع و پایان اجرای یک بخش از کد را ثبت می‌کند و سپس با استفاده از کتابخانه `<chrono>`، زمان سپری‌شده را محاسبه می‌کند.

## اجزای اصلی `<chrono>`

کتابخانه `<chrono>` بر پایه سه مفهوم اصلی ساخته شده است:

### Duration

یک `Duration` یا **مدت‌زمان**، مقدار زمان بین دو رویداد را نمایش می‌دهد. این مقدار شامل یک عدد و واحد زمانی مربوط به آن است.

- با استفاده از `std::chrono::duration` نمایش داده می‌شود.
- می‌تواند واحدهایی مانند ثانیه، میلی‌ثانیه، میکروثانیه، نانوثانیه و... را ذخیره کند.
- از عملیات ریاضی مانند جمع و تفریق پشتیبانی می‌کند.
- تابع `count()` مقدار ذخیره‌شده را برمی‌گرداند.

```cpp
#include <chrono>
#include <iostream>

using namespace std;
using namespace std::chrono;

int main()
{
    seconds s(5);

    milliseconds ms = duration_cast<milliseconds>(s);

    cout << "Seconds: " << s.count() << endl;
    cout << "Milliseconds: " << ms.count() << endl;

    return 0;
}
```

### Clock

یک `Clock` یا **ساعت** زمان فعلی را فراهم می‌کند و به‌عنوان مرجعی برای اندازه‌گیری مدت‌زمان و ایجاد `Time Point` استفاده می‌شود.

در C++ سه نوع Clock استاندارد وجود دارد:

| Clock | توضیحات |
|---|---|
| `system_clock` | ساعت واقعی سیستم را نمایش می‌دهد. |
| `steady_clock` | یک Clock یکنواخت است که به عقب برنمی‌گردد و برای اندازه‌گیری زمان سپری‌شده ایده‌آل است. |
| `high_resolution_clock` | کوچک‌ترین دوره زمانی موجود را برای اندازه‌گیری‌های با دقت بالا فراهم می‌کند. |

### TimePoint

یک `Time Point` یا **نقطه زمانی**، لحظه مشخصی از زمان را نسبت به Epoch مربوط به یک Clock نمایش می‌دهد.

- با استفاده از `std::chrono::time_point` نمایش داده می‌شود.
- با استفاده از تابع `now()` یک Clock به دست می‌آید.
- می‌توان دو `Time Point` را از یکدیگر کم کرد تا مدت‌زمان بین آن‌ها محاسبه شود.
- معمولاً برای Benchmark و اندازه‌گیری زمان اجرای برنامه استفاده می‌شود.

```cpp
#include <chrono>
#include <iostream>
#include <thread>

using namespace std;
using namespace std::chrono;

int main()
{
    time_point<steady_clock> start = steady_clock::now();

    // شبیه‌سازی انجام یک عملیات
    this_thread::sleep_for(seconds(2));

    time_point<steady_clock> end = steady_clock::now();

    auto elapsed = duration_cast<milliseconds>(end - start);

    cout << "Elapsed Time: "
         << elapsed.count() << " ms";

    return 0;
}
```

## انواع رایج Duration

کتابخانه `<chrono>` چند نوع Duration از پیش تعریف‌شده در اختیار ما قرار می‌دهد.

| نوع Duration | واحد زمانی |
|---|---|
| `nanoseconds` | ۱۰⁻⁹ ثانیه |
| `microseconds` | ۱۰⁻⁶ ثانیه |
| `milliseconds` | ۱۰⁻³ ثانیه |
| `seconds` | ۱ ثانیه |
| `minutes` | ۶۰ ثانیه |
| `hours` | ۶۰ دقیقه |

## مثال‌ها

### اندازه‌گیری زمان اجرای برنامه (Stopwatch)

```cpp
#include <iostream>
#include <chrono>
#include <thread>

int main() {

    // 1. شروع تایمر
    auto start = std::chrono::steady_clock::now();

    // انجام عملیات
    // در اینجا برای شبیه‌سازی، برنامه را 500 میلی‌ثانیه متوقف می‌کنیم
    std::this_thread::sleep_for(std::chrono::milliseconds(500));

    // 2. توقف تایمر
    auto end = std::chrono::steady_clock::now();

    // 3. محاسبه مدت‌زمان
    // مدت‌زمان را به واحد موردنظر تبدیل می‌کنیم
    // مانند milliseconds یا microseconds
    auto duration =
        std::chrono::duration_cast<std::chrono::milliseconds>(
            end - start
        );

    std::cout << "Time elapsed: "
              << duration.count()
              << " ms"
              << std::endl;

    return 0;
}
```

### دریافت تاریخ و زمان فعلی

```cpp
#include <iostream>
#include <chrono>
#include <ctime>   // برای std::time_t

int main() {

    // 1. دریافت Time Point فعلی
    auto now = std::chrono::system_clock::now();

    // 2. تبدیل Time Point به time_t
    // برای نمایش ساده‌تر زمان
    std::time_t currentTime =
        std::chrono::system_clock::to_time_t(now);

    // 3. نمایش زمان
    // ctime مقدار time_t را به رشته‌ای مانند
    // "Wed Dec 31 17:00:00 2025\n"
    // تبدیل می‌کند.
    std::cout << "Current time: "
              << std::ctime(&currentTime);

    return 0;
}
```

## مزایای `<chrono>`

کتابخانه `<chrono>` مزایای مختلفی دارد:

- روی سیستم‌عامل‌ها و پلتفرم‌های مختلف به شکل یکسان کار می‌کند.
- Clock، Duration و Time Point را از یکدیگر جدا می‌کند و انعطاف‌پذیری بیشتری ایجاد می‌کند.
- فرآیند Benchmark و تحلیل عملکرد برنامه را ساده‌تر می‌کند.
- امکان اندازه‌گیری زمان با دقت بالا را فراهم می‌کند.

## محدودیت‌های `<chrono>`

با وجود مزایای زیاد، کتابخانه `<chrono>` محدودیت‌هایی نیز دارد:

- دقت Clock به سیستم‌عامل و سخت‌افزار زیرین بستگی دارد.
- Clockهای مختلف کاربردهای متفاوتی دارند و باید متناسب با نیاز انتخاب شوند.
- برخی قابلیت‌های پیشرفته مربوط به Time Zone به استانداردهای جدیدتر C++ نیاز دارند.

---

## 🤝 مشارکت‌کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](mailto:m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>