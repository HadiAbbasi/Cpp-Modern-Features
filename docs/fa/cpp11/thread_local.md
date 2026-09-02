

[🇺🇸 English](../../en/cpp11/emplace.md) | [🇮🇷 فارسی](./emplace.md)

</div>


# `thread_local` در C++ (ذخیره‌سازی محلی هر نخ)

`thread_local` یکی از ویژگی‌های معرفی‌شده در **C++11** است که به هر **نخ (Thread)** اجازه می‌دهد نسخه‌ی مستقل خودش را از یک متغیر داشته باشد.

به زبان ساده، وقتی یک متغیر را با `thread_local` تعریف می‌کنید، **هر Thread یک کپی جداگانه از آن متغیر** دریافت می‌کند. بنابراین هر نخ می‌تواند مقدار متغیر را بخواند یا تغییر دهد، بدون اینکه روی نخ‌های دیگر اثری بگذارد.

```cpp
thread_local int counter = 0;

void do_work() {
    ++counter;   // هر Thread فقط نسخه‌ی خودش را تغییر می‌دهد
}
```

> **بهترین کاربرد:** زمانی که داده‌ها قابل تغییر، قابل استفادهٔ مجدد و مخصوص همان Thread باشند.

---

## ویژگی‌های Thread Local Storage (TLS)

- **طول عمر (Lifetime):** از زمان مقداردهی اولیه تا پایان عمر همان Thread.
- **قابلیت مشاهده (Visibility):** فقط در همان Thread قابل دسترسی است.
- **محدوده (Scope):** به محل تعریف متغیر بستگی دارد.

---

## تفاوت با سایر انواع Storage Duration

| نوع ذخیره‌سازی | طول عمر | بین Threadها مشترک؟ | ایمن در چندنخی؟ |
|---|---|---|---|
| **automatic** (متغیر محلی) | تا پایان بلاک | ❌ خیر | ✅ بله |
| **static** | کل اجرای برنامه | ✅ بله | ❌ خیر |
| **dynamic** (`new/delete`) | تا آزادسازی حافظه | بستگی دارد | بستگی دارد |
| **thread_local** | تا پایان Thread | ❌ خیر | ✅ بله |

**نتیجه:** `thread_local` ماندگاری `static` را با استقلال متغیر محلی ترکیب می‌کند.

---

## ویژگی‌های مهم `thread_local`

### ۱. یک نمونه برای هر Thread

```cpp
thread_local std::vector<int> cache;

void process() {
    cache.push_back(42);
}
```

- اولین بار که یک Thread به `cache` دسترسی پیدا کند، شیء ساخته می‌شود.
- دفعات بعد همان Thread از همان نمونه استفاده می‌کند.
- هر Thread کش مخصوص خودش را دارد.

### ۲. نابودی هنگام پایان Thread

پس از خاتمهٔ Thread:

- Destructor اجرا می‌شود.
- حافظه آزاد می‌شود.
- نیازی به مدیریت دستی حافظه نیست.

### ۳. بدون نیاز به همگام‌سازی

از آنجا که هر Thread نسخهٔ مستقل خود را دارد، استفاده از `mutex`، `atomic` یا قفل لازم نیست.

### ۴. دسترسی بسیار سریع

در معماری x86-64 دسترسی به متغیر `thread_local` معمولاً به یک دستور اسمبلی تبدیل می‌شود:

```asm
mov rax, fs:[offset]
```

بنابراین هزینهٔ دسترسی تقریباً مشابه یک متغیر سراسری است.

---

## چرا از `thread_local` استفاده کنیم؟

```cpp
// شناسه مخصوص هر Thread
thread_local int thread_id = -1;

// آخرین پیام خطا
thread_local std::string last_error;

// تولیدکننده اعداد تصادفی
thread_local std::mt19937 rng(std::random_device{}());
```

این ویژگی برای نگهداری اشیای پرهزینه‌ای که باید در هر Thread دوباره استفاده شوند، بسیار مناسب است.

---

## مثال ۱: شمارنده مستقل برای هر Thread

```cpp
#include <iostream>
#include <thread>
using namespace std;

thread_local int counter = 0;

void increment_counter()
{
    counter++;

    cout << "Thread "
         << this_thread::get_id()
         << " counter = "
         << counter << endl;
}

int main()
{
    thread t1(increment_counter);
    thread t2(increment_counter);

    t1.join();
    t2.join();
}
```

### خروجی نمونه

```text
Thread A counter = 1
Thread B counter = 1
```

هر Thread مقدار مستقل خودش را نگهداری می‌کند.

---

## مثال ۲: Singleton مخصوص هر Thread

```cpp
#include <iostream>
#include <thread>
using namespace std;

class Singleton {
public:
    static Singleton& getInstance()
    {
        thread_local Singleton instance;
        return instance;
    }

    void printMessage()
    {
        cout << "Hello from thread "
             << this_thread::get_id()
             << endl;
    }

private:
    Singleton() = default;
};

void workerThread()
{
    Singleton::getInstance().printMessage();
}

int main()
{
    thread t1(workerThread);
    thread t2(workerThread);

    t1.join();
    t2.join();
}
```

در این مثال، هر Thread یک شیء `Singleton` مستقل دارد.

---

## Static Thread Local Storage

```cpp
#include <iostream>
#include <thread>
using namespace std;

void thread_func()
{
    static thread_local int value = 0;

    value++;

    cout << "Thread: "
         << this_thread::get_id()
         << " Value = "
         << value << endl;
}
```

**رفتار:**

- برای هر Thread یک نسخه وجود دارد.
- مقدار بین فراخوانی‌های تابع در همان Thread حفظ می‌شود.
- Threadهای مختلف روی یکدیگر تأثیر ندارند.

---

## قوانین و محدودیت‌ها

1. `thread_local` فقط برای **تعریف متغیرها** استفاده می‌شود و برای توابع مجاز نیست.
2. برای متغیرهای محلی، `thread_local` به‌طور ضمنی رفتار `static` دارد؛ بنابراین این دو معادل هستند:

```cpp
thread_local int x;
```

```cpp
static thread_local int x;
```

3. اگر اعلان و تعریف در فایل‌های جداگانه باشند، هر دو باید دارای `thread_local` باشند.

---

## جمع‌بندی

`thread_local` مکانیزمی در C++ برای نگهداری **داده‌های اختصاصی هر Thread** است.

### مزایا

- یک نسخه مستقل برای هر Thread
- ماندگاری تا پایان عمر Thread
- بدون نیاز به `mutex` و قفل
- مناسب برای کش، وضعیت خطا، تولیدکننده اعداد تصادفی و اشیای قابل استفادهٔ مجدد

> اگر داده باید بین Threadها مشترک باشد از `static` استفاده کنید؛ اما اگر هر Thread به دادهٔ مستقل نیاز دارد، `thread_local` بهترین انتخاب است.

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | m.roodsarabi76@gmail.com | | [mbr](https://t.me/ad1mi2n) |

</div>