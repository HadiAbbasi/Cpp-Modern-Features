<div align="center">

[🇺🇸 English](../../en/cpp20/coroutines.md) | [🇮🇷 فارسی](./coroutines.md)

</div>

---

# بررسی Coroutines در ++C

## مقدمه

قابلیت **Coroutines** در ++C20 یکی از امکانات مهم زبان برای نوشتن محاسبات قابل‌تعلیق و قابل‌ادامه است. یک Coroutine را می‌توان تعمیمی از Function دانست: مانند یک تابع فراخوانی می‌شود، اما برخلاف تابع معمولی می‌تواند در نقاط مشخصی اجرای خود را متوقف کند، وضعیت لازم را حفظ کند و بعداً از همان نقطه ادامه دهد.

این قابلیت برای ساختن جریان‌های داده‌ای، تولید تدریجی مقادیر، پیاده‌سازی عملیات ناهمگام (Asynchronous Operations)، مدیریت توالی‌های طولانی و ساده‌تر کردن کدهایی که ذاتاً چندمرحله‌ای هستند کاربرد دارد.

نکته مهم این است که **Coroutine به‌خودی‌خود Thread نیست**. تعلیق یک Coroutine الزاماً به معنی اجرای آن روی Thread دیگری نیست و استفاده از `co_await` نیز به‌تنهایی عملیات را Asynchronous نمی‌کند. زمان و نحوه ادامه اجرا به نوع Coroutine، قرارداد Awaitable و Scheduler یا کتابخانه‌ای بستگی دارد که در طراحی سیستم استفاده شده است.

## تفاوت Function و Coroutine

### رفتار Function معمولی

در یک Function معمولی، اجرای تابع از نقطه ورود آغاز می‌شود و با رسیدن به `return` یا پایان بدنه تابع خاتمه پیدا می‌کند. پس از بازگشت، وضعیت محلی تابع دیگر به‌عنوان وضعیت فعال آن فراخوانی در دسترس نیست.

برای مثال:

```cpp
int calculate() {
    int value = 10;
    value += 5;
    return value;
}
```

### رفتار Coroutine

یک Coroutine می‌تواند علاوه بر شروع و پایان، در نقاط مشخصی از اجرا Suspend شود. هنگام Suspend شدن، وضعیت موردنیاز برای ادامه اجرا در ساختاری به نام **Coroutine State** حفظ می‌شود. بعداً با Resume شدن، اجرای Coroutine از نقطه تعلیق‌شده ادامه پیدا می‌کند.

```cpp
generator<int> generate_numbers() {
    co_yield 10;
    co_yield 20;
    co_yield 30;
}
```

در این مثال، هر بار که مصرف‌کننده مقدار بعدی را درخواست می‌کند، Coroutine تا `co_yield` بعدی اجرا می‌شود، مقدار را در اختیار مصرف‌کننده قرار می‌دهد و Suspend می‌شود. تعریف `generator<T>` در این مثال صرفاً یک نوع فرضی برای نشان دادن کاربرد Generator است؛ ++C20 چنین نوعی را در Standard Library ارائه نمی‌کند.

### مقایسه رفتار دو نوع تابع

| ویژگی | Function معمولی | Coroutine |
|---|---|---|
| فراخوانی | بله | بله |
| بازگرداندن نتیجه | بله | بله، با سازوکار متناسب با نوع Promise |
| توقف موقت اجرا | به شکل عادی خیر | بله |
| ادامه از نقطه توقف | خیر | بله |
| حفظ وضعیت محلی در زمان تعلیق | پس از بازگشت تابع خیر | بله، برای متغیرهای زنده در Coroutine Frame |
| اجرای خودکار روی Thread دیگر | خیر | خیر |

## کاربردهای اصلی Coroutines

### تولید Lazy مقادیر

در روش Lazy، محاسبه تا زمانی که مصرف‌کننده به نتیجه نیاز پیدا نکرده است انجام نمی‌شود. یک Generator می‌تواند مقادیر را یکی‌یکی تولید کند، بدون اینکه لازم باشد تمام نتایج از ابتدا در یک Container ذخیره شوند.

### ساده‌سازی کد Asynchronous

در طراحی‌های Asynchronous، گاهی لازم است یک عملیات پس از پایان I/O یا رویدادی دیگر ادامه پیدا کند. با استفاده از Awaitable مناسب می‌توان نقطه انتظار را با `co_await` مشخص کرد و از نوشتن زنجیره‌ای از Callbackها اجتناب کرد.

### مدل‌سازی جریان داده

برای خواندن تدریجی فایل، پردازش رویدادها، پیمایش داده‌های تولیدشونده و ساخت Pipelineها می‌توان از Coroutineهای مبتنی بر Generator استفاده کرد.

## نحوه تشخیص Coroutine توسط کامپایلر

هر تابعی که در بدنه خود از یکی از سه Keyword زیر استفاده کند، از دید زبان یک Coroutine محسوب می‌شود:

- `co_await`
- `co_yield`
- `co_return`

صرفاً تغییر نوع بازگشتی تابع یا نام‌گذاری آن به‌عنوان Coroutine کافی نیست. وجود یکی از این Keywordها در بدنه تابع، آن را وارد قواعد Coroutine می‌کند.

در مقابل، هر تابعی در ++C نمی‌تواند Coroutine باشد. از جمله مواردی که Coroutine نیستند یا اجازه استفاده از این Keywordها را ندارند، می‌توان به `main`، Constructor، Destructor و تابع `constexpr` اشاره کرد. در مورد توابعی که محدودیت‌های دیگری مانند `consteval` یا برخی Contextهای خاص دارند نیز باید قواعد استاندارد همان نسخه زبان بررسی شود.

## سه Keyword اصلی Coroutine

### معرفی co_yield

عبارت `co_yield` یک مقدار را از Coroutine به مصرف‌کننده ارائه می‌کند و معمولاً اجرای Coroutine را Suspend می‌کند. این Keyword برای ساخت Generatorها بسیار مهم است.

```cpp
generator<int> numbers() {
    co_yield 10;
    co_yield 20;
    co_yield 30;
}
```

از دید مفهومی، عبارت زیر:

```cpp
co_yield value;
```

تقریباً به فراخوانی زیر در Promise Type تبدیل می‌شود:

```cpp
co_await promise.yield_value(value);
```

این تبدیل، مدل ذهنی مفیدی برای فهم رفتار `co_yield` است؛ جزئیات دقیق آن تابع قواعد Coroutine و نوع Promise است.

در این مثال، `co_yield 10` به معنی پایان Coroutine نیست. Coroutine مقدار ۱۰ را ارائه می‌کند و Suspend می‌شود. پس از درخواست بعدی مصرف‌کننده، اجرای تابع از دستور بعد از همان `co_yield` ادامه پیدا می‌کند.

### معرفی co_return

عبارت `co_return` اجرای Coroutine را به پایان می‌رساند. بسته به طراحی Promise Type، ممکن است مقدار نهایی نیز ثبت یا تحویل داده شود.

دو شکل اصلی آن عبارت‌اند از:

```cpp
co_return;
```

و:

```cpp
co_return result;
```

شکل اول از تابع `return_void()` در Promise Type استفاده می‌کند و شکل دوم از `return_value(result)`.

یک Promise Type نمی‌تواند هم‌زمان هر دو تابع `return_void()` و `return_value()` را برای یک Coroutine ارائه کند؛ انتخاب آن‌ها باید با شیوه پایان‌یافتن Coroutine سازگار باشد.

نکته مهم اینکه `co_return` با `co_yield` تفاوت دارد: اولی Coroutine را خاتمه می‌دهد، اما دومی مقداری را ارائه می‌کند و معمولاً اجرای Coroutine را برای ادامه بعدی Suspend می‌کند.

### معرفی co_await

عبارت `co_await` برای انتظار روی یک عملیات یا وضعیت قابل‌انتظار (Awaitable) استفاده می‌شود. این عبارت ممکن است Coroutine جاری را Suspend کند و پس از آماده شدن نتیجه، امکان Resume شدن آن را فراهم کند.

```cpp
auto result = co_await operation;
```

در این مثال، `operation` باید طبق قواعد زبان به یک Awaiter قابل‌استفاده تبدیل شود. اینکه Coroutine واقعاً Suspend شود یا نه، به نتیجه `await_ready()` و رفتار `await_suspend()` وابسته است.

`co_await` به‌تنهایی Thread جدید ایجاد نمی‌کند و تضمین نمی‌کند که ادامه اجرا روی Thread دیگری انجام شود. برای مثال، یک Awaitable ممکن است اگر نتیجه از قبل آماده باشد، بدون Suspend شدن مقدار را فوراً برگرداند.

## گردش کار Coroutine

### مراحل کلی اجرا

گردش کار یک Coroutine را می‌توان در چند مرحله خلاصه کرد:

1. کامپایلر با توجه به نوع بازگشتی Coroutine، نوع Promise متناظر را پیدا می‌کند.
2. فضای موردنیاز برای نگهداری وضعیت Coroutine یا همان Coroutine Frame فراهم می‌شود.
3. پارامترهای Coroutine و متغیرهای لازم برای ادامه اجرا در Frame نگهداری می‌شوند.
4. شیء Promise ساخته می‌شود.
5. تابع `get_return_object()` برای ساخت شیء بازگشتی Coroutine فراخوانی می‌شود.
6. رفتار `initial_suspend()` تعیین می‌کند که Coroutine پس از آماده‌سازی اولیه فوراً اجرا شود یا ابتدا Suspend بماند.
7. با Resume شدن Coroutine، دستورات بدنه اجرا می‌شوند.
8. در نقاط `co_await` یا `co_yield` ممکن است Coroutine Suspend شود.
9. با Resume بعدی، اجرا از نقطه تعلیق‌شده ادامه می‌یابد.
10. در پایان، `final_suspend()` و سپس سازوکار مدیریت طول عمر Frame تعیین می‌کنند که وضعیت Coroutine چگونه حفظ یا آزاد شود.

### اجرای Eager و Lazy

در مدل **Eager**، Coroutine پس از فراخوانی، اجرای بدنه را آغاز می‌کند و در اولین نقطه تعلیق متوقف می‌شود. این رفتار معمولاً با پیاده‌سازی `initial_suspend()` به شکل `std::suspend_never` حاصل می‌شود.

در مدل **Lazy**، Coroutine بلافاصله پس از آماده‌سازی اولیه Suspend می‌شود و تا زمانی که مصرف‌کننده آن را Resume نکند، بدنه اجرا نمی‌شود. این رفتار معمولاً با `std::suspend_always` در `initial_suspend()` پیاده‌سازی می‌شود.

بنابراین Lazy بودن یک ویژگی ذاتی همه Coroutineها نیست؛ این رفتار از قرارداد Promise Type و نوع بازگشتی انتخاب‌شده ناشی می‌شود.

## زیرساخت داخلی Coroutine

++C20 سازوکارهای سطح پایین Coroutine را به زبان اضافه کرده است. این سازوکارها به کتابخانه‌ها اجازه می‌دهند انواع مختلف Coroutine مانند Task، Generator و عملیات Asynchronous را بسازند.

سه جزء مهم در این زیرساخت عبارت‌اند از:

- **Promise Type**: رفتار و قرارداد Coroutine را تعریف می‌کند.
- **Coroutine Handle**: امکان دسترسی مدیریتی به Coroutine و کنترل Resume یا Destroy آن را فراهم می‌کند.
- **Awaiter**: رفتار تعلیق و ادامه اجرای Coroutine را برای یک عبارت `co_await` تعریف می‌کند.

### مفهوم Coroutine Frame

کامپایلر برای نگهداری وضعیت لازم جهت ادامه Coroutine، ساختاری به نام Coroutine Frame ایجاد می‌کند. این Frame معمولاً شامل اطلاعاتی مانند موارد زیر است:

- پارامترهایی که برای ادامه اجرای Coroutine لازم‌اند.
- متغیرهای محلی که طول عمرشان از نقطه Suspend عبور می‌کند.
- وضعیت اجرای Coroutine و نقطه‌ای که باید از آن ادامه پیدا کند.
- شیء Promise و داده‌های موردنیاز سازوکار Coroutine.

نباید فرض کرد تمام متغیرهای محلی همیشه به یک شکل در Frame ذخیره می‌شوند؛ جزئیات پیاده‌سازی در اختیار کامپایلر است. همچنین محل تخصیص Frame الزاماً همیشه Heap نیست؛ کامپایلر در شرایطی که بتواند طول عمر و نیازهای آن را اثبات کند ممکن است تخصیص را بهینه‌سازی کند.

### معرفی Promise Type

Promise Type یک نوع C++ است که قرارداد رفتاری Coroutine را مشخص می‌کند. این نوع معمولاً از طریق نام‌گذاری وابسته به نوع بازگشتی Coroutine و قواعد `std::coroutine_traits` پیدا می‌شود.

Promise تعیین می‌کند:

- شیء بازگشتی Coroutine چگونه ساخته شود.
- Coroutine در شروع و پایان چه رفتاری داشته باشد.
- مقدار `co_return` چگونه مدیریت شود.
- مقدار `co_yield` چگونه ارائه شود.
- Exception مدیریت‌نشده چگونه دریافت شود.

توابع مهم Promise Type عبارت‌اند از:

| تابع | مسئولیت |
|---|---|
| `get_return_object()` | ساخت شیء بازگشتی Coroutine |
| `initial_suspend()` | تعیین رفتار تعلیق اولیه |
| `final_suspend()` | تعیین رفتار Coroutine در پایان اجرای بدنه |
| `unhandled_exception()` | دریافت Exceptionای که در بدنه Coroutine مدیریت نشده است |
| `return_void()` | پشتیبانی از `co_return;` |
| `return_value(value)` | پشتیبانی از `co_return value;` |
| `yield_value(value)` | تعریف رفتار `co_yield value;` |

این توابع باید مطابق قواعد استاندارد و نوع بازگشتی Coroutine پیاده‌سازی شوند. امضای دقیق و نوع بازگشتی آن‌ها اهمیت دارد؛ جدول بالا فقط نقش مفهومی هر تابع را نشان می‌دهد.

### معرفی Coroutine Handle

نوع `std::coroutine_handle` در Header استاندارد `<coroutine>` تعریف شده است. این Handle نماینده‌ای برای دسترسی به Coroutine Frame است و عملیات مدیریتی مهمی را در اختیار پیاده‌ساز نوع Coroutine قرار می‌دهد.

عملیات متداول آن عبارت‌اند از:

- `resume()` برای ادامه اجرای Coroutine معلق.
- `done()` برای بررسی اینکه Coroutine در وضعیت پایان‌یافته و Suspend نهایی قرار دارد یا نه.
- `destroy()` برای تخریب Coroutine Frame.
- `promise()` برای دسترسی به Promise متناظر، در Handleهای دارای Promise Type.

نکته مهم این است که Handle به‌تنهایی مالکیت ایمن و خودکار ایجاد نمی‌کند. اگر نوع بازگشتی Coroutine مالک Handle باشد، باید دقیقاً مشخص کند چه زمانی Frame را Destroy می‌کند. Destroy کردن Handle نامعتبر، Destroy کردن چندباره یا Resume کردن Coroutineی که در وضعیت مناسبی نیست، می‌تواند به رفتار تعریف‌نشده منجر شود.

### معرفی Awaitable و Awaiter

شیئی که می‌تواند در سمت راست `co_await` قرار بگیرد، در اصطلاح Awaitable نامیده می‌شود. در نهایت، قواعد زبان باید بتوانند از آن یک Awaiter استخراج کنند.

Awaiter شیئی است که سه عملیات اصلی زیر را ارائه می‌کند:

| تابع | نقش |
|---|---|
| `await_ready()` | مشخص می‌کند آیا نتیجه همین حالا آماده است و نیازی به Suspend نیست |
| `await_suspend(handle)` | در صورت نیاز، وضعیت تعلیق را مدیریت یا ادامه اجرا را زمان‌بندی می‌کند |
| `await_resume()` | پس از ادامه اجرا، نتیجه عبارت `co_await` را تولید می‌کند |

یک نمونه ساده از Awaiter:

```cpp
#include <coroutine>

struct ImmediateAwaiter {
    bool await_ready() const noexcept {
        return true;
    }

    void await_suspend(std::coroutine_handle<>) const noexcept {
    }

    int await_resume() const noexcept {
        return 42;
    }
};
```

استفاده از آن در یک Coroutine:

```cpp
Task<int> read_value() {
    int value = co_await ImmediateAwaiter{};
    co_return value;
}
```

در این مثال، چون `await_ready()` مقدار `true` برمی‌گرداند، Coroutine Suspend نمی‌شود و `await_resume()` بلافاصله فراخوانی می‌شود. نوع `Task<int>` در این قطعه صرفاً یک نوع بازگشتی نمونه است و باید توسط یک کتابخانه یا پیاده‌سازی جداگانه تعریف شده باشد.

### تفاوت Awaitable و Awaiter

Awaitable شیئی است که می‌توان آن را در عبارت `co_await` استفاده کرد. Awaiter شیئی است که سه تابع `await_ready`، `await_suspend` و `await_resume` را فراهم می‌کند.

گاهی خود Awaitable همان Awaiter است. در موارد دیگر، Awaitable با فراهم کردن `operator co_await` یا از طریق قواعد تبدیل مربوط به Promise، یک Awaiter تولید می‌کند.

### پیاده‌سازی Awaitable سفارشی

می‌توان با طراحی یک Class یا Struct و فراهم کردن Awaiter مناسب، رفتار انتظار سفارشی ایجاد کرد. برای نمونه، یک Awaitable واقعی در یک سیستم Asynchronous می‌تواند هنگام `await_suspend()`، Handle را به Event Loop یا Scheduler تحویل دهد و پس از آماده شدن عملیات، آن را Resume کند.

در چنین طراحی‌ای باید به چند نکته توجه کرد:

- Handle نباید پس از Destroy شدن Coroutine نگهداری یا Resume شود.
- عملیات زمان‌بندی باید از Resume هم‌زمان و ناخواسته جلوگیری کند.
- طول عمر داده‌هایی که Awaiter به آن‌ها ارجاع می‌دهد باید تا پایان عملیات معتبر بماند.
- خطاهای عملیات Asynchronous باید از مسیر `await_resume()` یا قرارداد خطای کتابخانه منتقل شوند.
- Thread-safety باید توسط Scheduler و قرارداد Awaitable مشخص شود.

## جزئیات مهم co_await

### ترتیب مفهومی ارزیابی

عبارت `co_await expr` پس از اعمال قواعد تبدیل زبان، به Awaiter می‌رسد. در حالت کلی، رفتار آن به این صورت است:

1. Awaiter از عبارت Awaitable به دست می‌آید.
2. تابع `await_ready()` فراخوانی می‌شود.
3. اگر `await_ready()` مقدار `true` بدهد، Coroutine Suspend نمی‌شود و `await_resume()` اجرا می‌شود.
4. اگر مقدار `false` باشد، Coroutine وارد مسیر تعلیق می‌شود و `await_suspend()` فراخوانی می‌شود.
5. پس از Resume شدن Coroutine، تابع `await_resume()` اجرا می‌شود و نتیجه عبارت `co_await` را تعیین می‌کند.

نوع بازگشتی `await_suspend()` می‌تواند بر کنترل جریان اثر بگذارد. این تابع می‌تواند `void`، `bool` یا یک Coroutine Handle برگرداند؛ هر کدام معنای مشخصی در قواعد استاندارد دارند. بنابراین نباید همیشه فرض کرد `await_suspend()` فقط یک تابع `void` است.

### رفتار await_suspend

اگر `await_suspend()` از نوع `void` باشد، Coroutine معلق می‌ماند و Awaiter مسئول فراهم کردن مسیر Resume بعدی است.

اگر مقدار `bool` برگرداند، مقدار `false` به معنی ادامه اجرای Coroutine بدون باقی ماندن در حالت Suspend است و مقدار `true` به معنی تعلیق آن است.

اگر یک `std::coroutine_handle` برگرداند، کنترل می‌تواند به Coroutine دیگری منتقل شود. این قابلیت در پیاده‌سازی بعضی الگوهای انتقال کنترل کاربرد دارد.

پیاده‌سازی `await_suspend()` باید با دقت انجام شود؛ ممکن است Scheduler یا Thread دیگری بلافاصله پس از ثبت کار، Coroutine را Resume کند. بنابراین دسترسی به داده‌های Awaiter پس از انتشار Handle باید با در نظر گرفتن احتمال Resume هم‌زمان طراحی شود.

## جزئیات مهم co_yield

### نقش yield_value در Promise

عبارت `co_yield value` از طریق `yield_value(value)` در Promise Type پردازش می‌شود. مقدار بازگشتی `yield_value` باید با قواعد `co_await` سازگار باشد؛ زیرا از نظر مفهومی، `co_yield` به یک عملیات Await تبدیل می‌شود.

این طراحی اجازه می‌دهد Generator تعیین کند مقدار چگونه به مصرف‌کننده ارائه شود و پس از آن Coroutine چگونه Suspend بماند.

### تفاوت yield و return

`co_yield` برای ارائه یک مقدار میانی و ادامه‌پذیر است. `co_return` برای پایان دادن به Coroutine به کار می‌رود. در یک Generator معمولی، مصرف‌کننده ممکن است چندین بار مقدار دریافت کند و در نهایت با پایان Coroutine روبه‌رو شود.

## جزئیات مهم co_return و مدیریت Exception

### پایان عادی Coroutine

هنگامی که اجرای بدنه به `co_return` برسد یا از انتهای بدنه Coroutine عبور کند، مسیر پایان بر اساس Promise Type اجرا می‌شود. برای Coroutineهایی که `return_void()` دارند، رسیدن به انتهای بدنه می‌تواند معادل `co_return;` باشد.

### مدیریت Exception

اگر Exceptionای از بدنه Coroutine خارج شود و در همان بدنه مدیریت نشود، تابع `unhandled_exception()` در Promise فراخوانی می‌شود. این تابع می‌تواند Exception را ذخیره کند، آن را به مصرف‌کننده منتقل کند یا بر اساس قرارداد نوع Coroutine رفتار دیگری داشته باشد.

برای مثال، یک Task می‌تواند `std::exception_ptr` را در Promise ذخیره کند و در `await_resume()` دوباره آن را با `std::rethrow_exception` پرتاب کند.

### تفاوت پایان اجرا و آزادسازی Frame

پایان یافتن بدنه Coroutine الزاماً به معنی آزاد شدن فوری Coroutine Frame نیست. معمولاً Coroutine ابتدا به نقطه `final_suspend()` می‌رسد. مالک Handle یا نوع بازگشتی باید طبق قرارداد خود تصمیم بگیرد چه زمانی Frame را Destroy کند.

این جداسازی برای آن مهم است که مصرف‌کننده بتواند نتیجه نهایی یا Exception را از Promise دریافت کند.

## ساخت یک Generator سفارشی

### هدف از پیاده‌سازی Generator

Generator نوعی Coroutine است که مقادیر را به‌صورت تدریجی تولید می‌کند. مصرف‌کننده هر بار مقدار بعدی را درخواست می‌کند و تولیدکننده فقط تا نقطه `co_yield` بعدی اجرا می‌شود.

در ادامه یک Generator ساده و آموزشی برای مقادیر `int` پیاده‌سازی می‌کنیم. این نمونه برای نشان دادن ارتباط Promise، Handle، Iterator و `co_yield` طراحی شده است و جایگزین یک کتابخانه تولیدی و کامل نیست.

### پیاده‌سازی Generator ساده

```cpp
#include <coroutine>
#include <exception>
#include <iterator>
#include <utility>

class Generator {
public:
    struct promise_type;
    using Handle = std::coroutine_handle<promise_type>;

    struct promise_type {
        int current_value{};

        Generator get_return_object() noexcept {
            return Generator{Handle::from_promise(*this)};
        }

        std::suspend_always initial_suspend() const noexcept {
            return {};
        }

        std::suspend_always final_suspend() const noexcept {
            return {};
        }

        std::suspend_always yield_value(int value) noexcept {
            current_value = value;
            return {};
        }

        void return_void() const noexcept {
        }

        void unhandled_exception() {
            exception = std::current_exception();
        }

        std::exception_ptr exception;
    };

    class iterator {
    public:
        using iterator_category = std::input_iterator_tag;
        using value_type = int;
        using difference_type = std::ptrdiff_t;
        using pointer = const int*;
        using reference = const int&;

        iterator() noexcept = default;

        explicit iterator(Handle handle) noexcept
            : handle_(handle) {
        }

        reference operator*() const noexcept {
            return handle_.promise().current_value;
        }

        pointer operator->() const noexcept {
            return &handle_.promise().current_value;
        }

        iterator& operator++() {
            handle_.resume();

            if (handle_.done()) {
                auto exception = handle_.promise().exception;
                if (exception) {
                    std::rethrow_exception(exception);
                }
            }

            return *this;
        }

        void operator++(int) {
            ++(*this);
        }

        friend bool operator==(const iterator& it,
                               std::default_sentinel_t) noexcept {
            return !it.handle_ || it.handle_.done();
        }

    private:
        Handle handle_{};
    };

    explicit Generator(Handle handle) noexcept
        : handle_(handle) {
    }

    Generator(const Generator&) = delete;
    Generator& operator=(const Generator&) = delete;

    Generator(Generator&& other) noexcept
        : handle_(std::exchange(other.handle_, {})) {
    }

    Generator& operator=(Generator&& other) noexcept {
        if (this != &other) {
            if (handle_) {
                handle_.destroy();
            }

            handle_ = std::exchange(other.handle_, {});
        }

        return *this;
    }

    ~Generator() {
        if (handle_) {
            handle_.destroy();
        }
    }

    iterator begin() {
        if (handle_) {
            handle_.resume();

            if (handle_.done()) {
                auto exception = handle_.promise().exception;
                if (exception) {
                    std::rethrow_exception(exception);
                }

                return iterator{};
            }
        }

        return iterator{handle_};
    }

    std::default_sentinel_t end() const noexcept {
        return {};
    }

private:
    Handle handle_{};
};

Generator generate_numbers() {
    co_yield 10;
    co_yield 20;
    co_yield 30;
}

#include <iostream>

int main() {
    for (int value : generate_numbers()) {
        std::cout << value << '\n';
    }
}
```

### تحلیل اجزای Generator

در این پیاده‌سازی، نوع `promise_type` قرارداد Coroutine را تعریف می‌کند. تابع `get_return_object()` یک شیء از نوع `Generator` می‌سازد و Handle مربوط به Coroutine را در اختیار آن قرار می‌دهد.

تابع `initial_suspend()` از `std::suspend_always` استفاده می‌کند؛ بنابراین Coroutine هنگام ساخته شدن اجرا نمی‌شود و در اولین درخواست `begin()` شروع به اجرا می‌کند.

تابع `yield_value()` مقدار تولیدشده را در `current_value` ذخیره می‌کند و با بازگرداندن `std::suspend_always`، اجرای Coroutine را Suspend می‌کند.

Iterator با `resume()` از Coroutine می‌خواهد تا مقدار بعدی را تولید کند. عملگر Dereference مقدار فعلی ذخیره‌شده در Promise را در اختیار مصرف‌کننده قرار می‌دهد.

نوع `Generator` مالک Handle است؛ به همین دلیل Copy غیرفعال شده و Move برای انتقال مالکیت پیاده‌سازی شده است. Destructor نیز Frame را آزاد می‌کند.

### محدودیت‌های پیاده‌سازی آموزشی

این Generator عمداً ساده نگه داشته شده است و برای استفاده عمومی در یک کتابخانه حرفه‌ای کافی نیست. از جمله محدودیت‌های آن:

- فقط مقدار `int` را تولید می‌کند و Template نیست.
- Iterator آن یک Input Iterator ساده است و تمام جزئیات مفهومی و الزامات یک Range عمومی را پوشش نمی‌دهد.
- برای استفاده هم‌زمان از چند Iterator یا پیمایش چندباره طراحی نشده است.
- قرارداد Exception و رفتار پس از پایان، حداقلی است.
- این نوع برای استفاده چند Thread یا انتقال کنترل بین Schedulerها طراحی نشده است.
- پشتیبانی از Reference، Move-only Values، Allocator سفارشی و Cancellation ندارد.

در پیاده‌سازی تولیدی باید مالکیت Handle، رفتار Iterator، Exception، Reentrancy، Thread-safety، تخصیص Frame و Cancellation به‌صورت دقیق طراحی و آزمایش شوند.

## استفاده از Generatorهای کتابخانه‌ای

### وضعیت Standard Library در نسخه‌های مختلف

در ++C20، زبان زیرساخت Coroutine را ارائه می‌کند، اما یک نوع عمومی و استاندارد مانند `std::generator` برای استفاده مستقیم در اختیار برنامه‌نویس قرار نمی‌دهد.

در ++C23، نوع `std::generator` به Standard Library اضافه شده است. این نوع در Header زیر تعریف می‌شود:

```cpp
#include <generator>
```

بنابراین ادعای اینکه Generator استاندارد قرار است در ++C23 اضافه شود، اکنون یک قابلیت استانداردشده است؛ البته دسترسی عملی به آن به نسخه Standard Library و میزان پشتیبانی Toolchain بستگی دارد.

### استفاده از std::generator در ++C23

یک مثال ساده:

```cpp
#include <generator>
#include <iostream>

std::generator<int> generate_numbers() {
    co_yield 10;
    co_yield 20;
    co_yield 30;
}

int main() {
    for (int value : generate_numbers()) {
        std::cout << value << '\n';
    }
}
```

در این مثال، `std::generator<int>` یک Range مبتنی بر Coroutine است که مقادیر را به شکل Lazy تولید می‌کند. نیازی نیست تمام مقادیر قبل از شروع پیمایش در یک `std::vector` ذخیره شوند.

### چه زمانی std::generator انتخاب مناسبی است؟

استفاده از `std::generator` برای تولید ترتیبی مقادیر، ساخت Rangeهای Lazy، پیمایش داده‌های محاسبه‌شونده و ساده کردن کدهایی که قبلاً با Iteratorهای دستی پیچیده نوشته می‌شدند مناسب است.

با این حال، `std::generator` به‌تنهایی ابزار عمومی Asynchronous Programming نیست. Generator استاندارد برای تولید مقادیر در یک جریان پیمایش طراحی شده است؛ برای عملیات Asynchronous باید از Task یا Abstraction مناسب کتابخانه و Scheduler استفاده شود.

## استفاده از کتابخانه‌های Third-party

### دلیل استفاده از کتابخانه‌های آماده

ساخت نوع Coroutine سفارشی از نظر فنی امکان‌پذیر است، اما پیاده‌سازی صحیح آن دشوارتر از نوشتن چند تابع Promise است. مدیریت طول عمر Frame، Exception، Handle، تعلیق و ادامه، Cancellation و Thread-safety می‌تواند پیچیدگی قابل‌توجهی ایجاد کند.

به همین دلیل، برای کاربردهای واقعی بهتر است ابتدا از قابلیت‌های Standard Library یا کتابخانه‌های معتبر استفاده شود و تنها در صورت وجود نیاز مشخص، نوع Coroutine اختصاصی ساخته شود.

### نمونه‌هایی از کتابخانه‌ها

- **cppcoro**: مجموعه‌ای از Abstractionهای Coroutine مانند Task، Generator و ابزارهای مرتبط است. این پروژه با نام `cppcoro` شناخته می‌شود؛ وضعیت نگهداری، شاخه فعال و سازگاری آن با Compiler موردنظر باید پیش از انتخاب بررسی شود.
- **Coro**: پروژه‌ای با مخزن `https://github.com/Quuxplusone/coro` است که در اسلایدهای منبع نیز به آن اشاره شده است. API و وضعیت پشتیبانی آن را باید از مستندات و مخزن همان پروژه بررسی کرد.
- **کتابخانه‌های Asynchronous مبتنی بر Coroutine**: بعضی Frameworkها و کتابخانه‌های شبکه یا I/O، Abstractionهای Task و Awaitable مخصوص خود را ارائه می‌کنند. انتخاب آن‌ها باید بر اساس Scheduler، مدل Cancellation، پشتیبانی از Executor و قرارداد Thread-safety انجام شود.

### نکات انتخاب کتابخانه

پیش از انتخاب یک کتابخانه Coroutine، موارد زیر را بررسی کنید:

- سازگاری با Compiler و نسخه استاندارد مورد استفاده.
- وضعیت نگهداری و کیفیت مستندات.
- مدل مالکیت و طول عمر Coroutine Frame.
- نحوه انتقال Exception.
- پشتیبانی از Cancellation و Timeout.
- مدل Scheduler و Executor.
- رفتار Thread-safety و امکان Resume از Threadهای مختلف.
- یکپارچگی با کتابخانه‌های I/O مورد استفاده در پروژه.
- مجوز استفاده و هزینه نگهداری وابستگی.

### تفاوت Generator و Task

Generator معمولاً برای تولید چند مقدار در طول زمان و ارائه آن‌ها به شکل یک Range استفاده می‌شود. Task معمولاً نماینده یک عملیات است که در نهایت یک نتیجه یا Exception تولید می‌کند و ممکن است برای عملیات Asynchronous استفاده شود.

این دو Abstraction هدف یکسانی ندارند. یک Generator الزاماً Asynchronous نیست و یک Task نیز الزاماً چند مقدار متوالی تولید نمی‌کند.

## محدودیت‌ها و نکات طراحی

### Coroutine الزاماً Asynchronous نیست

یک Coroutine می‌تواند کاملاً روی همان Thread فراخواننده اجرا شود. Suspend و Resume دو ویژگی کنترلی هستند؛ Asynchronous بودن به همکاری Awaitable، Scheduler یا زیرساخت I/O وابسته است.

### Coroutine الزاماً سریع‌تر از Function نیست

Coroutine برای ساده‌سازی مدل کنترل جریان و مدیریت وضعیت طراحی شده است، نه برای تضمین افزایش سرعت. ایجاد و مدیریت Coroutine Frame، تخصیص حافظه، انتقال کنترل و Resume می‌تواند هزینه داشته باشد.

### طول عمر متغیرها

متغیرهایی که باید پس از Suspend شدن Coroutine همچنان زنده باشند، باید در وضعیت Coroutine حفظ شوند. با این حال، اگر Coroutine به داده‌ای بیرونی از طریق Pointer، Reference یا `std::string_view` ارجاع دهد، Coroutine Frame طول عمر آن داده بیرونی را تضمین نمی‌کند.

### مدیریت مالکیت و تخریب

هر طراحی باید روشن کند چه شیئی مالک Coroutine Handle است و چه زمانی Frame آزاد می‌شود. Handle خام را نباید بدون قرارداد روشن میان چند Owner به اشتراک گذاشت.

### خطرهای مربوط به Thread

Coroutineها به‌صورت ذاتی Thread-safe نیستند. اگر یک Awaitable، Coroutine را روی Thread دیگری Resume کند، دسترسی به داده‌های مشترک و وضعیت اشیای درگیر باید با قواعد هم‌زمانی C++ هماهنگ باشد.

### Cancellation

استاندارد Coroutine در ++C20 یک مدل عمومی و کامل Cancellation ارائه نمی‌کند. Cancellation باید توسط Abstraction سطح بالاتر، Awaitableها و Scheduler طراحی شود. صرف Destroy کردن Coroutine همیشه معادل Cancellation ایمن عملیات بیرونی نیست؛ برای مثال، ممکن است یک عملیات I/O همچنان در سیستم عامل یا کتابخانه در حال اجرا باشد.

### Exception Safety

Exception می‌تواند از بدنه Coroutine به `unhandled_exception()` برسد. نوع Coroutine باید سیاست روشنی برای ذخیره، انتقال یا تبدیل Exception داشته باشد. در طراحی Asynchronous باید مشخص باشد Exception در کدام نقطه و روی کدام Thread دوباره پرتاب یا گزارش می‌شود.

## اشتباهات رایج

- تصور اینکه `co_await` همیشه باعث Suspend شدن Coroutine می‌شود.
- تصور اینکه هر Coroutine روی Thread جداگانه اجرا می‌شود.
- فرض اینکه ++C20 یک `std::generator` آماده ارائه کرده است.
- اشتباه گرفتن `co_yield` با `co_return`.
- آزاد نکردن Coroutine Frame یا Destroy کردن آن در زمان نامناسب.
- Resume کردن Coroutineی که قبلاً به پایان رسیده یا Handle آن نامعتبر است.
- نگهداری Reference به داده‌ای که پیش از Resume بعدی از بین می‌رود.
- تصور اینکه Promise Type همان Coroutine Handle است.
- تصور اینکه هر شیء Awaitable الزاماً عملیات Asynchronous انجام می‌دهد.
- پیاده‌سازی یک Coroutine Type عمومی بدون تست دقیق طول عمر، Exception و مالکیت.

## تفاوت امکانات زبان و کتابخانه استاندارد

| قابلیت | نسخه استاندارد | توضیح |
|---|---|---|
| Coroutine Language Support | ++C20 | شامل `co_await`، `co_yield`، `co_return` و قواعد تبدیل Coroutine |
| `std::coroutine_handle` | ++C20 | ابزار سطح پایین برای مدیریت Coroutine Frame |
| `std::suspend_always` و `std::suspend_never` | ++C20 | Awaiterهای آماده برای تعلیق یا عدم تعلیق |
| `std::generator` | ++C23 | Generator استاندارد برای تولید Lazy مقادیر |
| Task عمومی استاندارد | در ++C20 وجود ندارد | معمولاً از کتابخانه یا Abstraction اختصاصی استفاده می‌شود |

## جمع‌بندی

Coroutines در ++C20 امکان Suspend و Resume کردن اجرای یک تابع را فراهم می‌کنند و به کمک آن‌ها می‌توان Generatorها، Taskها و Abstractionهای Asynchronous را ساخت.

سه Keyword اصلی این قابلیت عبارت‌اند از `co_await` برای انتظار روی یک Awaitable، `co_yield` برای ارائه مقدار و Suspend شدن معمولاً در Generatorها، و `co_return` برای پایان دادن به Coroutine.

در لایه زیرساخت، Promise Type قرارداد Coroutine را تعریف می‌کند، Coroutine Handle امکان کنترل Frame را فراهم می‌کند و Awaiter رفتار تعلیق و ادامه اجرای `co_await` را مشخص می‌کند.

برای استفاده روزمره، بهتر است تا حد امکان از Abstractionهای آماده و آزموده‌شده استفاده شود. در ++C23 می‌توان برای تولید Lazy مقادیر از `std::generator` بهره برد؛ اما برای Task و Asynchronous I/O همچنان باید Abstraction مناسب کتابخانه یا Framework مورد استفاده را انتخاب کرد.


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>