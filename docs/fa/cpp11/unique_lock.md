<div align="right">

[🇺🇸 English](../../en/cpp11/unique_lock.md) | [🇮🇷 فارسی](./unique_lock.md)

</div>
کلاس **`std::unique_lock`** در **C++11** (که در هدر `<mutex>` تعریف شده است) یک پوشش پیشرفته و مبتنی بر الگوی **RAII** برای مدیریت قفل‌های انحصاری (`std::mutex`) است.

اگر `std::lock_guard` را یک مدیر قفل «ساده و انعطاف‌ناپذیر» بدانیم، `std::unique_lock` همتای «منعطف و با امکانات فراوان» آن به شمار می‌رود.

---

## چرا با وجود `std::lock_guard` به `std::unique_lock` نیاز داریم؟

با استفاده از `std::lock_guard`، کنترلی بر زمان اکتساب یا آزادسازی قفل در طول حوزه (Scope) ندارید، نمی‌توانید قفل‌گذاری را به تعویق بیندازید و امکان انتقال آن میان توابع وجود ندارد.

کلاس `std::unique_lock` قابلیت‌های زیر را اضافه می‌کند:
۱. **قفل‌گذاری و آزادسازی دستی (`lock()` و `unlock()`)** در میانه‌ی حوزه.
۲. **قفل‌گذاری به تعویق افتاده (Deferred Locking)** (به تأخیر انداختن اکتساب قفل).
۳. **قابلیت تلاش برای قفل‌گذاری (Try-lock)** (تلاش برای دریافت قفل بدون مسدودسازی نخ یا با تعیین مهلت زمانی).
۴. **معناشناسی انتقال (Move Semantics)** (قابلیت انتقال مالکیت قفل).
۵. **یکپارچگی کامل با `std::condition_variable`** (استفاده از آن برای این متغیر شرطی الزامی است).

---

## نحو پایه (Basic Syntax)

در ساده‌ترین شکل، دقیقاً مانند `std::lock_guard`، قفل در سازنده (Constructor) به‌صورت خودکار دریافت شده و در مخرب (Destructor) آزاد می‌شود:

```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;

void printMessage(int id) {
    // قفل هنگام ایجاد شیء اکتساب شده و در پایان حوزه آزاد می‌گردد
    std::unique_lock<std::mutex> lock(mtx);
    std::cout << "Thread " << id << " is working\n";
}
```

---

## راهبردهای قفل‌گذاری (Locking Strategies)

هنگام ساخت شیء `std::unique_lock`، می‌توانید رفتار آن را با ارسال برچسب‌های استاندارد (Standard Tags) تعیین کنید:

### الف) `std::defer_lock` (قفل‌گذاری به تعویق افتاده)
متغیر Mutex تحت مدیریت قرار می‌گیرد، اما **بلافاصله قفل نمی‌شود**. می‌توانید بعداً آن را به‌صورت دستی قفل کنید:

```cpp
std::mutex mtx;

void task() {
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
    
    // کارهایی که نیازی به قفل ندارند...
    
    lock.lock(); // اکنون قفل دریافت می‌شود
    // عملیات در بخش بحرانی (Critical Section)
    lock.unlock(); // آزادسازی دستی در صورت نیاز
}
```

### ب) `std::try_to_lock` (تلاش غیرمسدودکننده)
تلاش می‌کند قفل را به دست آورد؛ اگر Mutex توسط نخ دیگری نگه‌داشته شده باشد، نخ جاری **مسدود (Block) نمی‌شود**:

```cpp
void tryTask() {
    std::unique_lock<std::mutex> lock(mtx, std::try_to_lock);

    if (lock.owns_lock()) {
        std::cout << "Lock acquired! Processing data...\n";
    } else {
        std::cout << "Mutex is busy, doing something else...\n";
    }
}
```

### ج) `std::adopt_lock` (پذیرش قفل موجود)
فرض می‌کند نخ فراخواننده **از قبل Mutex را قفل کرده است** و شیء تنها وظیفه آزادسازی آن را بر عهده می‌گیرد:

```cpp
mtx.lock(); // قفل‌گذاری دستی
std::unique_lock<std::mutex> lock(mtx, std::adopt_lock);
// در پایان حوزه، شیء lock متغیر Mutex را آزاد خواهد کرد
```

---

## ویژگی‌های کلیدی `std::unique_lock`

### ۱. آزادسازی و قفل‌گذاری مجدد دستی (`unlock` و `lock`)
امکان قفل و باز کردن مکرر در یک حوزه را فراهم می‌کند؛ امری که برای کوتاه کردن زمان نگه‌داشتن قفل در حلقه‌ها یا در حین محاسبات سنگین بسیار مفید است:

```cpp
std::unique_lock<std::mutex> lock(mtx);

// دسترسی به داده‌های مشترک
shared_data.process();

lock.unlock(); // آزادسازی قفل برای انجام محاسبات سنگین غیربحرانی
heavyNonCriticalComputation();

lock.lock(); // اکتساب مجدد قفل
shared_data.finish();
```

### ۲. معناشناسی صرفاً انتقالی (Move-Only Semantics)
نمی‌توانید یک `std::unique_lock` را کپی کنید، اما می‌توانید مالکیت آن را با استفاده از `std::move` منتقل نمایید:

```cpp
std::unique_lock<std::mutex> getLock() {
    std::unique_lock<std::mutex> lock(mtx);
    // کارهای مقدماتی
    return lock; // بازگشت همراه با انتقال خودکار مالکیت (Move Semantics)
}

void useLock() {
    std::unique_lock<std::mutex> lock = getLock(); // انتقال مالکیت به این نقطه
    // قفل همچنان فعال است!
} // قفل در پایان این بلوک آزاد می‌شود
```

---

## کاربرد حیاتی: استفاده همراه با `std::condition_variable`

یکی از دلایل بنیادین به‌کارگیری `std::unique_lock`، تعامل آن با `std::condition_variable` است. متد `wait` باید بتواند هنگام به خواب رفتن نخ، قفل را موقتاً آزاد کند و به محض بیدار شدن مجدداً آن را به دست آورد؛ کاری که `std::lock_guard` قادر به انجام آن نیست:

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void workerThread() {
    std::unique_lock<std::mutex> lock(mtx);
    
    // نخ منتظر می‌ماند تا متغیر ready مقدار true بگیرد.
    // 'cv' هنگام انتظار به‌طور خودکار Mutex را آزاد کرده و پس از بیدار شدن دوباره آن را قفل می‌کند.
    cv.wait(lock, [] { return ready; });

    std::cout << "Worker is doing work...\n";
}

void publisherThread() {
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one(); // بیدار کردن نخ کارگر (Worker)
}

int main() {
    std::thread t1(workerThread);
    std::thread t2(publisherThread);

    t2.join();
    t1.join();
}
```

---

## مقایسه سریع: `lock_guard` در مقابل `unique_lock`

| ویژگی | `std::lock_guard` | `std::unique_lock` |
| :--- | :--- | :--- |
| **سادگی و سرعت** | بسیار ساده، سبک و سریع (دارای سربار صفر) | کمی سنگین‌تر به دلیل نگهداری وضعیت قفل |
| **قابلیت آزادسازی دستی (`unlock`)** | ❌ ندارد | ✅ دارد |
| **انتقال مالکیت (Move Ownership)** | ❌ ندارد | ✅ دارد |
| **سازگاری با `condition_variable`** | ❌ ندارد | ✅ دارد (الزامی است) |
| **قفل‌گذاری به تعویق افتاده (Deferred Locking)** | ❌ ندارد | ✅ دارد |

---

## جمع‌بندی: مقایسه با `std::lock_guard`

* بار اجرایی (Performance Overhead) در `std::unique_lock` نسبت به `std::lock_guard` بسیار ناچیز است (تنها به دلیل نگهداری یک پرچم بولی برای ردیابی وضعیت قفل).
* **نتیجه‌گیری:**
  * اگر صرفاً به یک قفل ساده برای تمام طول یک حوزه نیاز دارید $\rightarrow$ از **`std::lock_guard`** استفاده کنید.
  * اگر به **`std::condition_variable`**، **قفل‌گذاری به تعویق افتاده**، **تلاش غیرمسدودکننده** یا **آزادسازی دستی قفل** نیاز دارید $\rightarrow$ همواره از **`std::unique_lock`** استفاده کنید.
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | mb.roodsarabi76@gmail.com | | [mbr](https://t.me/ad1mi2n) |

</div>