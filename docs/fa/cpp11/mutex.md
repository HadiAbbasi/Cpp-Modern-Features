<div align="right">

[🇺🇸 English](../../en/cpp11/mutex.md) | [🇮🇷 فارسی](./mutex.md)

</div>
---

# Mutex

> **کلاس `mutex` یک primitive همگام‌سازی (Synchronization Primitive) است که می‌توان از آن برای محافظت از داده‌های مشترک در برابر دسترسی هم‌زمان چند نخ استفاده کرد.**

- دسترسی انحصاری به بخش‌های بحرانی (Critical Sections) را فراهم می‌کند.
- هنگام دسترسی به داده‌های مشترک، رفتار قابل پیش‌بینی‌تری ایجاد می‌کند.

**یک Mutex فقط می‌تواند در یکی از دو وضعیت زیر باشد:**

- `locked` — قفل‌شده
- `unlocked` — آزاد

### وقتی یک نخ Mutex را قفل می‌کند:

- نخ‌های دیگری که تلاش می‌کنند همان Mutex را قفل کنند، Block می‌شوند.
- فقط نخی که Mutex را قفل کرده است، می‌تواند آن را Unlock کند.

### محافظت از بخش‌های کد

این ویژگی به ما اجازه می‌دهد بخش‌هایی از کد را در برابر اجرای هم‌زمان توسط چند نخ محافظت کنیم.

### روند معمول استفاده از Mutex

1. ایجاد و مقداردهی اولیه یک متغیر Mutex
2. چند نخ تلاش می‌کنند Mutex را قفل کنند.
3. فقط یکی از نخ‌ها موفق می‌شود و مالک Mutex می‌شود.
4. نخ مالک، عملیات موردنظر را انجام می‌دهد.
5. نخ مالک Mutex را آزاد (`unlock`) می‌کند.
6. نخ دیگری Mutex را به دست می‌آورد و همین روند را تکرار می‌کند.
7. در پایان، Mutex باید به شکل مناسب مدیریت و آزاد شود.

![mutex](../../en/cpp11/assets/mutex.png)

## API در C++11

```cpp
#include <mutex>
```

- فایل Header مربوط به رابط Mutex را وارد می‌کند.

```cpp
void mutex.lock()
```

- Mutex را قفل می‌کند.
- اگر نخ دیگری Mutex را قفل کرده و مالک آن باشد، این تابع منتظر می‌ماند تا Mutex آزاد شود.

```cpp
void mutex.unlock()
```

- Mutex را آزاد می‌کند.
- پس از آزاد شدن Mutex، نخ‌های دیگر می‌توانند برای قفل کردن آن تلاش کنند.

```cpp
bool mutex.try_lock()
```

- تلاش می‌کند Mutex را قفل کند.
- برخلاف `lock()` منتظر نمی‌ماند و بلافاصله نتیجه را برمی‌گرداند.
- اگر قفل با موفقیت گرفته شود، `true` برمی‌گرداند.
- در غیر این صورت `false` برمی‌گرداند.

![lock](../../en/cpp11/assets/lock.png)

## Unique Lock — API

کلاس `unique_lock` استفاده از Mutexها را ساده‌تر می‌کند.

یکی از مزایای مهم آن این است که هنگام از بین رفتن شیء، Mutex به‌صورت خودکار Unlock می‌شود. این ویژگی به‌خصوص در هنگام رخ دادن Exception بسیار مفید است.

### ساخت `unique_lock` و قفل کردن Mutex

```cpp
unique_lock(mutex_type& m)
```

Mutex `m` را دریافت کرده و آن را قفل می‌کند.

### ساخت `unique_lock` بدون قفل کردن Mutex

```cpp
unique_lock(mutex_type& m, std::defer_lock_t t)
```

Mutex `m` را دریافت می‌کند، اما در ابتدا آن را قفل نمی‌کند.

### قفل کردن

```cpp
unique_lock.lock()
```

Mutex مربوط به `unique_lock` را قفل می‌کند.

### آزاد کردن

```cpp
unique_lock.unlock()
```

Mutex مربوط به `unique_lock` را آزاد می‌کند.

### مثال

```cpp
void doJob(int id)
{
    unique_lock<mutex> outputGuard(mMtx, defer_lock);

    this_thread::sleep_for(
        chrono::seconds((6 * id + 3) % 5)
    );

    outputGuard.lock();

    cout << "The job " << id
         << " has been completed!"
         << endl;

    // نتیجه‌ی Job می‌تواند در یک متغیر خصوصی ذخیره شود.
}
```

در این مثال، با استفاده از `defer_lock`، Mutex در زمان ساخت `unique_lock` قفل نمی‌شود و بعداً با فراخوانی `lock()` قفل می‌شود.

## مثال

در مثال زیر قصد داریم یک `counter` مشترک داشته باشیم و چند نخ آن را افزایش دهند.

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <mutex>

using namespace std;

int counter = 0;
mutex counter_mutex;

void counterThread()
{
    for (int i = 0; i < 10000000; ++i)
    {
        unique_lock<mutex> counter_lock(counter_mutex);
        counter++;
    }

    return;
}

int main()
{
    vector<thread> threads;

    for (int i = 0; i < 4; ++i)
    {
        threads.push_back(thread(counterThread));
    }

    for (int i = 0; i < 4; ++i)
    {
        threads[i].join();
    }

    cout << counter << endl;

    return 0;
}
```

در اینجا هر نخ قبل از تغییر `counter`، Mutex را قفل می‌کند. بنابراین در هر لحظه فقط یک نخ می‌تواند بخش مربوط به تغییر `counter` را اجرا کند.

به این ترتیب از **Race Condition** جلوگیری می‌شود.

> **نکته:** بهتر است `unique_lock` فقط در محدوده‌ای قرار بگیرد که واقعاً نیاز به قفل دارد. برای یک عملیات ساده مانند `counter++`، در C++ معمولاً `std::lock_guard` نیز انتخاب مناسب و ساده‌تری است.

## مثال — Deadlock

مثال زیر چند نخ دارد که هرکدام برای انجام کار خود به دو ابزار (`tool`) نیاز دارند.

```cpp
void* guyThread(void* args)
{
    argsStruct_t* tool = (argsStruct_t*)args;

    while (true)
    {
        {
            unique_lock<mutex> tool1_lock(*tool->tool1);

            cout << "Guy " << tool->threadID << " borrowed "
                 << tool->tool1Name << "." << endl;

            work();

            unique_lock<mutex> tool2_lock(*tool->tool2);

            cout << "Guy " << tool->threadID << " borrowed "
                 << tool->tool2Name << "." << endl;

            work();
        }

        if ((*tool->counter) > COUNTER_TRESHOLD)
            break;

        {
            unique_lock<mutex> counter_lock(*tool->counterMutex);
            (*tool->counter)++;
        }
    }

    return 0;
}

typedef struct argsStruct_t
{
    mutex* counterMutex;
    int* counter;

    mutex* tool1;
    string tool1Name;

    mutex* tool2;
    string tool2Name;

    int threadID;
};

void work()
{
    for (int i = 0; i < WORK_ITERATIONS; i++);
}

int main()
{
    vector<thread> threadGuys;
    vector<mutex> mutexes(MUTEXES_COUNT);

    int counter = 0;

    vector<argsStruct_t*> threadTools(THREADS_COUNT);

    threadTools[0] = {
        &mutexes[COUNTER],
        &counter,
        &mutexes[HAMMER],
        "hammer",
        &mutexes[SCREW_DRIVER],
        "screw driver",
        0
    };

    threadTools[1] = {
        &mutexes[COUNTER],
        &counter,
        &mutexes[SCREW_DRIVER],
        "screw driver",
        &mutexes[SAW],
        "saw",
        1
    };

    threadTools[2] = {
        &mutexes[COUNTER],
        &counter,
        &mutexes[SAW],
        "saw",
        &mutexes[HAMMER],
        "hammer",
        2
    };

    for (int i = 0; i < THREADS_COUNT; i++)
    {
        threadGuys.push_back(
            thread(guyThread, (void*)&threadTools[i])
        );
    }

    for (int i = 0; i < THREADS_COUNT; i++)
        threadGuys[i].join();

    return 0;
}
```

در این مثال ممکن است با مشکلی به نام **Deadlock (بن‌بست)** مواجه شویم.

برای مثال:

- `Thread 0` ابتدا Mutex مربوط به `hammer` را قفل می‌کند و سپس منتظر `screw driver` می‌ماند.
- `Thread 1` ابتدا Mutex مربوط به `screw driver` را قفل می‌کند و سپس منتظر `saw` می‌ماند.
- `Thread 2` ابتدا Mutex مربوط به `saw` را قفل می‌کند و سپس منتظر `hammer` می‌ماند.

اگر ترتیب اجرای نخ‌ها مناسب نباشد، هر نخ می‌تواند منبعی را در اختیار داشته باشد که نخ دیگری به آن نیاز دارد.

در نتیجه همه‌ی نخ‌ها منتظر یکدیگر می‌مانند و هیچ‌کدام نمی‌توانند ادامه دهند.

این وضعیت را **Deadlock** می‌نامیم.

---

# Condition Variables

**متغیر شرطی (`condition_variable`)** مکانیزمی برای هماهنگ‌سازی و اطلاع‌رسانی بین نخ‌هاست.

با استفاده از آن:

- یک نخ می‌تواند منتظر رخ دادن یک رویداد یا برقرار شدن یک شرط بماند.
- نخ دیگری می‌تواند نخ منتظر را بیدار کند و به آن اطلاع دهد که وضعیت موردنظر ممکن است برقرار شده باشد.
- نخ بیدار‌شده باید دوباره شرط را بررسی کند و اگر همه‌ی شرایط برقرار بود، اجرای خود را ادامه دهد.

![Condition Variable](../../en/cpp11/assets/condition_varible.png)

## Condition Variables — API

```cpp
#include <condition_variable>
```

- فایل Header مربوط به رابط `condition_variable` را وارد می‌کند.

### `notify_one()`

```cpp
void condition_variable.notify_one()
```

- به یکی از نخ‌هایی که روی این `condition_variable` منتظر هستند، سیگنال ارسال می‌کند.

### `notify_all()`

```cpp
void condition_variable.notify_all()
```

- به تمام نخ‌هایی که روی این `condition_variable` منتظر هستند، سیگنال ارسال می‌کند.

### `wait()`

```cpp
void condition_variable.wait(unique_lock<mutex>& lock)
```

- ابتدا `lock` را آزاد می‌کند.
- سپس نخ را به حالت انتظار می‌برد.
- نخ تا زمانی که نخ دیگری آن را با ارسال Notification بیدار کند، منتظر می‌ماند.
- پس از بیدار شدن، Mutex دوباره قفل می‌شود.

مثال:

```cpp
{
    unique_lock<mutex> lk(mtx);

    while (!condition_ready())
        cv.wait(lk);

    compute_something();
}
```

### `wait()` همراه با Predicate

```cpp
void condition_variable.wait(
    unique_lock<mutex>& lock,
    Predicate pred
)
```

این نسخه از `wait` تقریباً معادل کد زیر است:

```cpp
while (!pred())
    cv.wait(lk);
```

مزیت این روش این است که شرط موردنظر مستقیماً در اختیار `wait` قرار می‌گیرد.

---

# مثال — Producer / Consumer

در این مثال، یک **Producer** داده تولید می‌کند و آن را در یک صف قرار می‌دهد و یک **Consumer** داده‌ها را از صف دریافت می‌کند.

صف حداکثر می‌تواند ۵ عنصر داشته باشد.

اگر صف پر باشد، Producer منتظر می‌ماند.

اگر صف خالی باشد، Consumer منتظر می‌ماند.

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <chrono>

std::queue<int> buffer;

std::mutex mtx;
std::condition_variable cv;

const int MAX_SIZE = 5;

// ---------------- Producer ----------------

void producer()
{
    for (int i = 1; i <= 10; ++i)
    {
        {
            std::unique_lock<std::mutex> lock(mtx);

            // اگر صف پر بود، Producer منتظر می‌ماند
            cv.wait(lock, [] {
                return buffer.size() < MAX_SIZE;
            });

            // تولید داده
            buffer.push(i);

            std::cout << "Producer: "
                      << i
                      << std::endl;
        }

        // اطلاع دادن به Consumer
        cv.notify_one();

        std::this_thread::sleep_for(
            std::chrono::milliseconds(500)
        );
    }
}

// ---------------- Consumer ----------------

void consumer()
{
    for (int i = 1; i <= 10; ++i)
    {
        int value;

        {
            std::unique_lock<std::mutex> lock(mtx);

            // اگر صف خالی بود، Consumer منتظر می‌ماند
            cv.wait(lock, [] {
                return !buffer.empty();
            });

            // برداشتن داده
            value = buffer.front();
            buffer.pop();

            std::cout << "Consumer: "
                      << value
                      << std::endl;
        }

        // اطلاع دادن به Producer
        cv.notify_one();

        std::this_thread::sleep_for(
            std::chrono::milliseconds(800)
        );
    }
}

// ---------------- main ----------------

int main()
{
    std::thread t1(producer);
    std::thread t2(consumer);

    t1.join();
    t2.join();

    return 0;
}
```

### نحوه‌ی کار مثال

**Producer:**

1. Mutex را قفل می‌کند.
2. بررسی می‌کند که آیا صف ظرفیت خالی دارد یا خیر.
3. اگر صف پر باشد، با `cv.wait()` منتظر می‌ماند.
4. اگر فضای خالی وجود داشته باشد، داده را وارد صف می‌کند.
5. Mutex آزاد می‌شود.
6. با `cv.notify_one()` به Consumer اطلاع می‌دهد که داده‌ای در صف قرار گرفته است.

**Consumer:**

1. Mutex را قفل می‌کند.
2. بررسی می‌کند که صف خالی نباشد.
3. اگر صف خالی باشد، با `cv.wait()` منتظر می‌ماند.
4. اگر داده‌ای وجود داشته باشد، آن را از صف خارج می‌کند.
5. Mutex آزاد می‌شود.
6. با `cv.notify_one()` به Producer اطلاع می‌دهد که یک فضای جدید در صف ایجاد شده است.

### چرا Predicate مهم است؟

در `condition_variable` ممکن است نخ بدون اینکه دقیقاً همان رویدادی که انتظارش را داشتیم رخ داده باشد، از حالت انتظار خارج شود. به همین دلیل باید **شرط واقعی را بعد از بیدار شدن دوباره بررسی کنیم**.

به همین دلیل الگوی زیر بسیار مهم است:

```cpp
cv.wait(lock, [] {
    return !buffer.empty();
});
```

یعنی:

> «منتظر بمان تا زمانی که `buffer` خالی نباشد.»

این الگو از نظر ایمنی و صحت برنامه بسیار بهتر از این است که صرفاً منتظر یک Notification باشیم.

---
## 🤝 مشارکت‌کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | m.roodsarabi76@gmail.com | | [mbr](@ad1mi2n) |

</div>