<div align="right">

[🇺🇸 English](../../en/cpp11/lock_guard.md) | [🇮🇷 فارسی](./lock_guard.md)

</div>
کلاس **`std::lock_guard`** در **C++11** (که در هدر `<mutex>` تعریف شده است) ابزاری ساده و ایمن برای مدیریت قفل‌های انحصاری (Mutex Locks) است. این کلاس بر پایه الگوی شناخته‌شده‌ی **RAII** (مخفف Resource Acquisition Is Initialization یا «تخصیص منبع مساوی است با مقداردهی اولیه») پیاده‌سازی شده است.

---

## مسئله چیست و چرا به `lock_guard` نیاز داریم؟

در برنامه‌نویسی چندنخی (Multithreaded Programming)، برای جلوگیری از شرایط مسابقه (Race Conditions) از `std::mutex` استفاده می‌کنیم. روش سنتی قفل‌گذاری دستی به شکل زیر است:

```cpp
std::mutex mtx;

void unsafeFunction() {
    mtx.lock();
    // انجام عملیات در بخش بحرانی (Critical Section)
    mtx.unlock();
}
```

### مشکلات روش دستی
* **فراموشی آزادسازی قفل (Forgetting to unlock):** ممکن است برنامه‌نویس فراخوانی متد `unlock()` را فراموش کند.
* **خطاها و استثناها (Exceptions):** اگر در طول اجرای بخش بحرانی استثنایی پرتاب شود، تابع پیش از رسیدن به `mtx.unlock()` خارج می‌شود. در نتیجه، قفل به‌صورت نامحدود باز می‌ماند که می‌تواند منجر به بن‌بست (Deadlock) شود.
* **خروج زودهنگام (Early returns):** اگر تابع دارای چندین دستور `return` باشد، متد `unlock()` باید پیش از هر کدام از آن‌ها فراخوانی شود که این رویکرد مستعد خطا و دشوار در نگهداری است.

---

## فلسفه و نحوه عملکرد `std::lock_guard`

الگوی RAII در `std::lock_guard` به شرح زیر عمل می‌کند:

* **هنگام ساخت شیء (Constructor):** قفل به‌صورت خودکار اکتساب می‌شود (`mtx.lock()`).
* **هنگام از بین رفتن شیء (Destructor):** به محض اینکه شیء از حوزه دید (Scope) خارج شود — حتی به دلیل یک دستور `return` زودهنگام یا پرتاب یک استثنا — آزادسازی قفل تضمین می‌گردد (`mtx.unlock()`).

---

## ساختار پایه و نحو (Syntax)

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx; // متغیر مشترک Mutex
int counter = 0;

void increment() {
    std::lock_guard<std::mutex> lock(mtx);
    
    ++counter;
    std::cout << "Counter: " << counter << " from thread: " 
              << std::this_thread::get_id() << "\n";
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();
}
```

---

## ویژگی‌های کلیدی `std::lock_guard` در C++11

* **غیرقابل کپی و غیرقابل انتقال (Non-copyable & Non-movable):**
  نمی‌توانید یک `std::lock_guard` را کپی کنید یا با استفاده از `std::move` منتقل نمایید. مالکیت قفل به‌شدت به همان حوزه‌ای که در آن ایجاد شده است گره خورده است.
* **عدم وجود توابع دستی `lock()` یا `unlock()`:**
  بر خلاف `std::unique_lock`، شما نمی‌توانید در میانه‌ی کار قفل را به‌صورت دستی باز یا دوباره بسته (Relock) کنید. همه‌چیز به‌دقت به طول عمر (Lifetime) شیء وابسته است.
* **سربار صفر (Zero-overhead):**
  در زمان اجرا (Runtime)، در مقایسه با فراخوانی‌های دستی قفل و باز کردن، هیچ هزینه عملکردی اضافی به همراه ندارد.

---

## کنترل حوزه دقیق با استفاده از بلوک‌های `{}`

گاهی اوقات مایل نیستید تمام بدنه‌ی یک تابع در حالت قفل بماند (تا سایر نخ‌ها بی‌جهت معطل نشوند). می‌توانید با استفاده از یک جفت آکولاد ساده، حوزه (Scope) مربوط به `std::lock_guard` را محدودتر کنید:

```cpp
void processData() {
    // وظایف سنگینی که به قفل نیاز ندارند
    doHeavyPreparation();

    {
        // قفل صرفاً برای به‌روزرسانی متغیر مشترک اکتساب می‌شود
        std::lock_guard<std::mutex> lock(mtx);
        sharedList.push_back(result);
    } // <-- قفل در اینجا نابود شده و متغیر Mutex آزاد می‌گردد

    // سایر وظایفی که کماکان به قفل نیاز ندارند
    logCompletion();
}
```

---

## استفاده از `std::adopt_lock`

اگر یک متغیر Mutex از قبل توسط نخ جاری قفل شده باشد، نباید دوباره قفل شود (چون تلاش برای قفل کردن یک `std::mutex` استاندارد برای دومین بار باعث بن‌بست می‌شود). با ارسال آرگومان `std::adopt_lock` به `std::lock_guard`، به آن می‌گویید: «قفل از قبل اکتساب شده است؛ وظیفه شما تنها آزادسازی آن در پایان حوزه است!»

یک مورد کاربردی مهم برای این ویژگی، جلوگیری از بن‌بست هنگام قفل کردن هم‌زمان دو Mutex است:

```cpp
#include <mutex>

std::mutex mtxA;
std::mutex mtxB;

void safeSwap() {
    // ۱. قفل کردن هم‌زمان هر دو Mutex به روشی عاری از بن‌بست
    std::lock(mtxA, mtxB);

    // ۲. تفویض مسئولیت آزادسازی به مکانیزم RAII با استفاده از adopt_lock
    std::lock_guard<std::mutex> lockA(mtxA, std::adopt_lock);
    std::lock_guard<std::mutex> lockB(mtxB, std::adopt_lock);

    // انجام عملیات حیاتی روی داده‌های مشترک
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
| **قفل‌گذاری به تأخیر افتاده (Deferred Locking)** | ❌ ندارد | ✅ دارد |

> **قانون کلی:** همیشه به‌طور پیش‌فرض از **`std::lock_guard`** استفاده کنید، مگر اینکه به باز کردن دستی قفل یا استفاده از `std::condition_variable` نیاز داشته باشید؛ در این صورت باید به سراغ **`std::unique_lock`** بروید.
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | mb.roodsarabi76@gmail.com | | [mbr](https://t.me/ad1mi2n) |

</div>