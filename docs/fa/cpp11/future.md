<div align="right">

[🇺🇸 English](../../en/cpp11/future.md) | [🇮🇷 فارسی](./future.md)

</div>

# future

## std::future

`std::future` یک کلاس قالب (Template Class) در C++ است که نتیجهٔ یک عملیات غیرهمزمان را نشان می‌دهد. این نتیجه می‌تواند یک **مقدار** یا یک **Exception** باشد که در آینده در دسترس قرار خواهد گرفت.

از `std::future` برای دریافت نتیجهٔ یک وظیفه استفاده می‌شود که در یک Thread جداگانه یا از طریق `std::async` اجرا شده است.

### ویژگی‌ها

- **غیرهمزمانی (Asynchrony):**  
  امکان اجرای یک عملیات را بدون مسدود کردن Thread فعلی فراهم می‌کند.

- **دریافت نتیجه (Getting the result):**  
  متد `get()` برای دریافت نتیجه استفاده می‌شود. اگر نتیجه هنوز آماده نباشد، `get()` اجرای Thread فعلی را متوقف می‌کند تا نتیجه آماده شود یا یک Exception ایجاد شود.

- **انتقال داده (Data transfer):**  
  `std::future` می‌تواند با یک شیء `std::promise` مرتبط باشد. `std::promise` راهی برای قرار دادن نتیجه یا Exception در Future فراهم می‌کند.

- **وضعیت (State):**  
  `std::future` دارای یک وضعیت داخلی است که مشخص می‌کند آیا نتیجه آماده شده است یا خیر. برای بررسی این وضعیت می‌توان از متدهایی مانند `wait()` و `wait_for()` استفاده کرد.

- **دریافت نتیجه فقط یک بار (Single retrieval):**  
  متد `get()` برای هر شیء `std::future` فقط یک بار قابل استفاده است. پس از دریافت نتیجه، Future دیگر نتیجهٔ قابل دریافت مجددی ندارد.

- **نوع نتیجه (Result type):**  
  نوع `std::future<T>` با نوع مقداری که عملیات غیرهمزمان برمی‌گرداند مشخص می‌شود.

## std::future در C++ چیست؟

در C++، `std::future` یک کلاس قالب است که برای دریافت نتیجهٔ یک وظیفهٔ غیرهمزمان استفاده می‌شود؛ یعنی نتیجه‌ای که هنوز محاسبه نشده و قرار است در آینده آماده شود.

عملیات غیرهمزمان در C++ را می‌توان با ابزارهایی مانند:

- `std::async`
- `std::packaged_task`
- `std::promise`

مدیریت کرد. این ابزارها می‌توانند یک شیء `std::future` در اختیار ما قرار دهند.

سپس با استفاده از این `std::future` می‌توانیم:

- منتظر آماده شدن نتیجه بمانیم.
- وضعیت Future را بررسی کنیم.
- نتیجهٔ عملیات را دریافت کنیم.

نکتهٔ مهم این است که نتیجهٔ یک `std::future` را فقط **یک بار** می‌توان دریافت کرد. پس از فراخوانی `get()`، وضعیت Future تغییر می‌کند و دیگر نمی‌توان دوباره نتیجهٔ آن را با `get()` دریافت کرد.

## سینتکس std::future

تعریف یک شیء `std::future` بسیار ساده است:

```cpp
std::future<type> name;
```

که در آن:

- **`name`**: نام شیء Future است.
- **`type`**: نوع داده‌ای است که قرار است از عملیات غیرهمزمان دریافت شود.

برای مثال:

```cpp
std::future<int> result;
```

در اینجا `result` می‌تواند نتیجه‌ای از نوع `int` را در آینده دریافت کند.

## توابع عضو (Member Functions)

کلاس `std::future` دارای چندین تابع عضو برای مدیریت نتیجهٔ غیرهمزمان است. برخی از مهم‌ترین آن‌ها عبارت‌اند از:

| شماره | تابع | توضیح |
|---:|---|---|
| 1 | **`get()`** | نتیجهٔ عملیات را دریافت می‌کند. اگر نتیجه آماده نباشد، منتظر می‌ماند. |
| 2 | **`wait()`** | تا زمانی که نتیجه آماده شود، Thread فعلی را منتظر نگه می‌دارد. |
| 3 | **`wait_for()`** | برای مدت زمان مشخصی منتظر آماده شدن نتیجه می‌ماند. |
| 4 | **`wait_until()`** | تا رسیدن به یک زمان مشخص منتظر آماده شدن نتیجه می‌ماند. |
| 5 | **`valid()`** | بررسی می‌کند که آیا Future دارای یک Shared State معتبر است و می‌توان روی آن `get()` را اجرا کرد یا خیر. |

## مثال‌ها

### استفاده از std::future برای چاپ نتیجهٔ یک عملیات غیرهمزمان

در مثال زیر، تابع `returnTwo` به‌صورت غیرهمزمان اجرا می‌شود و نتیجهٔ آن از طریق `std::future` دریافت می‌شود:

```cpp
#include <future>
#include <iostream>

int returnTwo()
{
    return 2;
}

int main()
{
    std::future<int> f =
        std::async(std::launch::async, returnTwo);

    std::cout << f.get();

    return 0;
}
```

در اینجا:

```cpp
std::future<int> f =
    std::async(std::launch::async, returnTwo);
```

تابع `returnTwo()` به‌صورت غیرهمزمان اجرا می‌شود و `f` نمایندهٔ نتیجهٔ آیندهٔ این عملیات است.

سپس:

```cpp
f.get();
```

نتیجهٔ `2` را دریافت می‌کند.

---

### تلاش برای دریافت چندبارهٔ مقدار از std::future

متد `get()` برای یک `std::future` فقط یک بار قابل استفاده است.

بنابراین کد زیر مشکل دارد:

```cpp
#include <future>
#include <iostream>

int returnTwo()
{
    return 2;
}

int main()
{
    std::future<int> f =
        std::async(std::launch::async, returnTwo);

    std::cout << f.get();

    std::cout << f.get();

    return 0;
}
```

در اینجا اولین فراخوانی:

```cpp
f.get();
```

نتیجه را دریافت می‌کند.

اما پس از آن، `f` دیگر Shared State معتبری برای دریافت نتیجه ندارد. بنابراین فراخوانی دوم:

```cpp
f.get();
```

باعث پرتاب شدن `std::future_error` می‌شود.

خطای احتمالی:

```text
terminate called after throwing an instance of 'std::future_error'
what(): std::future_error: No associated state
```

### جلوگیری از خطای No Associated State با استفاده از valid()

راه‌حل این است که قبل از استفاده از `get()`، وضعیت Future را با `valid()` بررسی کنیم.

```cpp
#include <future>
#include <iostream>

int returnTwo()
{
    return 2;
}

int main()
{
    std::future<int> f =
        std::async(std::launch::async, returnTwo);

    if (f.valid())
    {
        std::cout << f.get() << '\n';
    }
    else
    {
        std::cout << "Invalid State, Please create another Task\n";
    }

    if (f.valid())
    {
        std::cout << f.get() << '\n';
    }
    else
    {
        std::cout << "Invalid State, Please create another Task\n";
    }

    return 0;
}
```

در این مثال، قبل از هر بار فراخوانی `get()` وضعیت Future بررسی می‌شود.

در اولین بررسی:

```cpp
if (f.valid())
```

مقدار `true` است، بنابراین `get()` اجرا می‌شود و نتیجه دریافت می‌شود.

اما بعد از اجرای `get()`، وضعیت Future دیگر معتبر نیست. بنابراین در بررسی دوم:

```cpp
if (f.valid())
```

مقدار `false` خواهد بود و بخش `else` اجرا می‌شود.

---

## استفاده از std::future با std::promise

علاوه بر `std::async`، می‌توان `std::future` را با استفاده از `std::promise` نیز ایجاد کرد.

`std::promise` به یک Thread اجازه می‌دهد نتیجه‌ای را تولید کند و آن نتیجه را از طریق `std::future` در اختیار Thread دیگری قرار دهد.

مثال:

```cpp
#include <future>
#include <iostream>
#include <thread>

void foo(std::promise<int> p)
{
    p.set_value(25);
}

int main()
{
    std::promise<int> p;

    std::future<int> f = p.get_future();

    std::thread t(foo, std::move(p));

    t.join();

    std::cout << f.get();

    return 0;
}
```

در این مثال ابتدا یک `std::promise<int>` ایجاد می‌کنیم:

```cpp
std::promise<int> p;
```

سپس Future مربوط به آن را دریافت می‌کنیم:

```cpp
std::future<int> f = p.get_future();
```

اکنون `p` و `f` به یک Shared State مشترک متصل هستند.

Thread جدید تابع `foo` را اجرا می‌کند:

```cpp
std::thread t(foo, std::move(p));
```

در داخل `foo`، مقدار `25` در Promise قرار می‌گیرد:

```cpp
p.set_value(25);
```

در نتیجه، این مقدار از طریق Future قابل دریافت خواهد بود:

```cpp
std::cout << f.get();
```

خروجی:

```text
25
```

به‌صورت ساده می‌توان رابطهٔ بین آن‌ها را این‌گونه تصور کرد:

```text
Thread A
   |
   | set_value(25)
   v
std::promise<int>
   |
   | Shared State
   v
std::future<int>
   |
   | get()
   v
Thread B
```

بنابراین `std::promise` معمولاً برای **تولید نتیجه** و `std::future` برای **دریافت نتیجه** استفاده می‌شود.

### نتیجه گیری
`std::future` یکی از ابزارهای مهم C++ برای ارتباط و انتقال نتیجه بین عملیات‌های غیرهمزمان است.

این کلاس به ما اجازه می‌دهد یک عملیات را در پس‌زمینه اجرا کنیم و نتیجهٔ آن را در زمان مناسب دریافت کنیم، بدون اینکه مجبور باشیم جزئیات پیچیدهٔ ارتباط بین Threadها را به‌صورت مستقیم مدیریت کنیم.

`std::future` به‌خصوص در شرایطی مفید است که:

- یک کار سنگین را در پس‌زمینه اجرا می‌کنیم.
- نمی‌خواهیم Thread اصلی بلافاصله منتظر بماند.
- در آینده به نتیجهٔ عملیات نیاز داریم.
- می‌خواهیم Exceptionهای ایجادشده در عملیات غیرهمزمان را نیز از طریق `future.get()` دریافت و مدیریت کنیم.

به‌طور خلاصه:

```text
std::promise  →  تولید نتیجه
                     ↓
                 Shared State
                     ↓
std::future   →  دریافت نتیجه
```

در کنار `std::async`، `std::promise` و `std::packaged_task`، `std::future` یکی از اجزای اصلی مدل مدیریت نتایج غیرهمزمان در C++ مدرن است.


## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | m.roodsarabi76@gmail.com | | [mbr](https://t.me/ad1mi2n) |

</div>