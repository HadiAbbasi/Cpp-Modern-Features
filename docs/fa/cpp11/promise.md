# Promise

در برنامه‌نویسی چندنخی (Multithreading) در C++، **Thread** واحد اصلی قابل اجرای یک Process است. برای برقراری ارتباط و انتقال نتیجه بین دو یا چند Thread، می‌توان از `std::promise` در کنار `std::future` استفاده کرد.

```text
+-------------------+                          +------------------+
| std::promise<T>    |                          | std::future<T>   |
| (Worker/Producer)  |                          | (Main/Consumer)  |
+---------+---------+                          +--------+---------+
          |                                             |
          | set_value(data)                             | get() (blocks)
          v                                             v
    +---------------------------------------------------------+
    |                    Shared State                         |
    |  - Value / Exception storage                             |
    |  - Mutex & Condition Variable (internal sync)            |
    |  - Ready / Abandoned Flags                               |
    +---------------------------------------------------------+
```

## std::promise در C++ چیست؟

`std::promise` یک کلاس قالب (Template Class) است که در کنار کلاس `std::future` استفاده می‌شود.

`std::promise` به یک Thread اجازه می‌دهد مقداری را تولید و در Shared State قرار دهد تا Thread دیگری بتواند آن مقدار را از طریق `std::future` دریافت کند.

همچنین Producer می‌تواند به‌جای مقدار، یک Exception را در Promise قرار دهد تا Consumer بعداً آن Exception را از طریق `future.get()` دریافت کند.

به بیان ساده:

- `std::promise` برای **قرار دادن نتیجه** استفاده می‌شود.
- `std::future` برای **دریافت نتیجه** استفاده می‌شود.

این مکانیزم معمولاً در مسائل **Producer-Consumer** بسیار مفید است.

## نحوهٔ استفاده از std::promise

برای استفاده از `std::promise` باید Header زیر را اضافه کنیم:

```cpp
#include <future>
```

سپس مراحل زیر را دنبال می‌کنیم:

### 1. ایجاد یک std::promise

ابتدا یک شیء از نوع `std::promise` ایجاد می‌کنیم. این شیء مسئول نگهداری نتیجه‌ای است که در آینده تولید خواهد شد.

```cpp
std::promise<int> promise;
```

### 2. ایجاد یک std::future

با استفاده از تابع عضو `get_future()`، یک `std::future` مرتبط با Promise دریافت می‌کنیم.

```cpp
std::future<int> future = promise.get_future();
```

این Future برای دریافت نتیجه‌ای که بعداً توسط Promise تنظیم می‌شود، استفاده خواهد شد.

### 3. قرار دادن مقدار در std::promise

در Thread تولیدکننده (Producer)، می‌توانیم مقدار موردنظر را با استفاده از `set_value()` قرار دهیم:

```cpp
promise.set_value(42);
```

همچنین اگر در هنگام اجرای عملیات Exception رخ دهد، می‌توانیم آن را با استفاده از `set_exception()` در Promise قرار دهیم:

```cpp
promise.set_exception(std::current_exception());
```

### 4. دریافت مقدار از std::future

در Thread مصرف‌کننده (Consumer)، می‌توانیم با استفاده از `get()` نتیجه را دریافت کنیم:

```cpp
int result = future.get();
```

اگر Promise حاوی Exception باشد، فراخوانی `get()` آن Exception را پرتاب خواهد کرد.

## مثال

مثال زیر نحوهٔ استفاده از `std::promise` و `std::future` برای برقراری ارتباط بین دو Thread را نشان می‌دهد:

```cpp
#include <future>
#include <iostream>
#include <stdexcept>
#include <thread>

void RetrieveValue(std::promise<int>& result)
{
    try
    {
        int ans = 21095022;

        result.set_value(ans);
    }
    catch (...)
    {
        result.set_exception(std::current_exception());
    }
}

int main()
{
    std::promise<int> myPromise;

    std::future<int> myFuture =
        myPromise.get_future();

    std::thread computationThread(
        RetrieveValue,
        std::ref(myPromise)
    );

    try
    {
        int result = myFuture.get();

        std::cout << "Result: "
                  << result << '\n';
    }
    catch (const std::exception& e)
    {
        std::cerr << "Exception is: "
                  << e.what() << '\n';
    }

    computationThread.join();

    return 0;
}
```

در این مثال ابتدا یک `std::promise<int>` ایجاد می‌کنیم:

```cpp
std::promise<int> myPromise;
```

سپس Future مربوط به آن را دریافت می‌کنیم:

```cpp
std::future<int> myFuture =
    myPromise.get_future();
```

در مرحلهٔ بعد یک Thread جدید ایجاد می‌کنیم و Promise را با استفاده از `std::ref()` به تابع `RetrieveValue` ارسال می‌کنیم:

```cpp
std::thread computationThread(
    RetrieveValue,
    std::ref(myPromise)
);
```

Thread اجراشده مقدار `21095022` را در Promise قرار می‌دهد:

```cpp
result.set_value(ans);
```

در نتیجه، این مقدار از طریق Future در Thread اصلی قابل دریافت است:

```cpp
int result = myFuture.get();
```

اگر در Thread تولیدکننده Exception رخ دهد، آن Exception در Promise قرار می‌گیرد:

```cpp
result.set_exception(std::current_exception());
```

و سپس هنگام فراخوانی `get()` در Thread مصرف‌کننده، Exception دریافت و پرتاب می‌شود.

---

## ملاحظات سطح پایین و عملکرد

### Move-Only بودن std::promise

`std::promise` قابل کپی نیست، اما قابل انتقال (Move) است.

```cpp
std::promise<int> p1;

// Error: کپی کردن مجاز نیست
// std::promise<int> p2 = p1;

// Correct: انتقال مالکیت
std::promise<int> p2 = std::move(p1);
```

این ویژگی تضمین می‌کند که تنها یک Producer فعال مسئول تکمیل Shared State باشد.

پس از انتقال مالکیت از `p1` به `p2`، شیء `p1` دیگر Shared State را در اختیار ندارد.

بنابراین انجام عملیاتی مانند:

```cpp
p1.set_value(42);
```

باعث پرتاب شدن `std::future_error` با کد خطای زیر می‌شود:

```cpp
std::future_errc::no_state
```

---

### `get_future()` فقط یک بار قابل فراخوانی است

هر `std::promise` فقط می‌تواند یک `std::future` مرتبط داشته باشد.

بنابراین نمی‌توان `get_future()` را چندین بار روی یک Promise فراخوانی کرد:

```cpp
std::promise<int> promise;

std::future<int> future1 =
    promise.get_future();

std::future<int> future2 =
    promise.get_future();
```

فراخوانی دوم باعث ایجاد `std::future_error` می‌شود.

کد خطای مربوطه:

```cpp
std::future_errc::future_already_retrieved
```

اگر چند Consumer نیاز داشته باشند یک نتیجهٔ یکسان را مشاهده کنند، می‌توانیم Future را به `std::shared_future` تبدیل کنیم:

```cpp
std::promise<int> promise;

std::shared_future<int> sharedFuture =
    promise.get_future().share();
```

`std::shared_future` برخلاف `std::future` قابل کپی است و چند Thread می‌توانند به‌صورت ایمن منتظر نتیجه بمانند و همان نتیجه را دریافت کنند.

---

### یک Promise فقط یک بار می‌تواند تکمیل شود

یک `std::promise` فقط یک بار می‌تواند مقدار یا Exception دریافت کند.

یعنی می‌توانیم:

- یک مقدار را با `set_value()` قرار دهیم، **یا**
- یک Exception را با `set_exception()` قرار دهیم.

اما نمی‌توان یک Promise را بیشتر از یک بار تکمیل کرد.

برای مثال:

```cpp
std::promise<int> promise;

auto future = promise.get_future();

promise.set_value(42);

// Throws std::future_error
promise.set_value(100);
```

فراخوانی دوم `set_value()` باعث پرتاب شدن `std::future_error` می‌شود.

همین قانون هنگام ترکیب `set_value()` و `set_exception()` نیز برقرار است:

```cpp
promise.set_value(42);

// Also throws std::future_error
promise.set_exception(
    std::make_exception_ptr(
        std::runtime_error("Something failed")
    )
);
```

کد خطای مربوطه:

```cpp
std::future_errc::promise_already_satisfied
```

بنابراین، پس از اینکه Promise با یک مقدار یا Exception تکمیل شد، دیگر نمی‌توان آن را دوباره تکمیل کرد.

## جمع‌بندی

در C++ می‌توان از `std::promise` برای **ارتباط و انتقال نتیجه بین Threadها** در برنامه‌های همزمان استفاده کرد.

`std::promise` و `std::future` بخشی از امکانات همزمانی معرفی‌شده در **C++11** هستند و یک روش نسبتاً ساده و استاندارد برای انتقال نتیجه بین Threadها فراهم می‌کنند.

به‌طور خلاصه:

```text
Producer / Worker
       |
       | set_value()
       | set_exception()
       v
 std::promise<T>
       |
       | Shared State
       v
 std::future<T>
       |
       | get()
       v
Consumer / Main Thread
```

در این مدل:

- `std::promise` مسئول **تولید و قرار دادن نتیجه** است.
- `std::future` مسئول **دریافت نتیجه** است.
- `set_value()` برای ارسال مقدار استفاده می‌شود.
- `set_exception()` برای ارسال خطا استفاده می‌شود.
- `get()` برای دریافت مقدار یا دریافت Exception استفاده می‌شود.

به همین دلیل، `std::promise` و `std::future` ابزارهای مفیدی برای **انتقال داده، هماهنگ‌سازی Threadها و مدیریت Exceptionها** در برنامه‌های چندنخی C++ هستند.