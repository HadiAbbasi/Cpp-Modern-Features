<div align="center">

[🇺🇸 English](../../en/cpp11/weak_ptr.md) | [🇮🇷 فارسی](./weak_ptr.md)

</div>

---

# استفاده از `std::weak_ptr` در C++11 — راهنمای کامل و ساده برای درک مالکیت، چرخه‌های `shared_ptr` و جلوگیری از Memory Leak

## فهرست مطالب

- [1. مقدمه](#section-1)
- [2. قبل از `weak_ptr` چه مشکلی وجود داشت؟](#section-2)
- [3. `shared_ptr` چگونه مالکیت را مدیریت می‌کند؟](#section-3)
- [4. مشکل اصلی `shared_ptr`: چرخه مالکیت و Memory Leak](#section-4)
- [5. `weak_ptr` دقیقاً چیست؟](#section-5)
- [6. تفاوت اساسی `shared_ptr` و `weak_ptr`](#section-6)
- [7. `weak_ptr` چگونه چرخه مالکیت را می‌شکند؟](#section-7)
- [8. ساختن و قرار دادن `weak_ptr` در Setter](#section-8)
- [9. گرفتن `weak_ptr` از Getter](#section-9)
- [10. کپی، Move و `reset()` در `weak_ptr`](#section-10)
- [11. بررسی خالی بودن و `expired()`](#section-11)
- [12. تبدیل `weak_ptr` به `shared_ptr` با `lock()`](#section-12)
- [13. دسترسی به اعضای شیء با `weak_ptr`](#section-13)
- [14. `weak_ptr` و Raw Pointer](#section-14)
- [15. استفاده از `weak_ptr` در کلاس و الگوی Parent/Child](#section-15)
- [16. الگوی Observer](#section-16)
- [17. مثال کامل از چرخه خراب و اصلاح آن با `weak_ptr`](#section-17)
- [18. `use_count()`، Strong Count، Weak Count و Control Block](#section-18)
- [19. تفاوت `weak_ptr` و `shared_ptr`](#section-19)
- [20. تفاوت `weak_ptr` و `unique_ptr`](#section-20)
- [21. چه زمانی باید یا نباید از `weak_ptr` استفاده کنیم؟](#section-21)
- [22. اشتباهات رایج](#section-22)
- [23. یک مثال کامل و واقعی‌تر](#section-23)
- [24. جمع‌بندی نهایی](#section-24)

---

<a id="section-1"></a>

# 1. مقدمه

ابزار `std::weak_ptr` یکی از مهم‌ترین بخش‌های مدیریت حافظه در C++11 است.

اشاره گر `weak_ptr` همراه با `shared_ptr` معنا پیدا می‌کند و برای حل مشکلی به وجود آمد که در مدل مالکیت اشتراکی `shared_ptr` بسیار مهم است.

کارکرد اصلی را اگر بخواهیم در یک جمله بیان کنیم، این است:

> **`weak_ptr` می‌تواند یک شیء را مشاهده کند، بدون اینکه مالک آن شیء باشد.**

عبارت بالا شاید در ابتدا ساده به نظر برسد، اما دقیقاً همین ویژگی یکی از مهم‌ترین مشکلات `shared_ptr` را حل می‌کند.

مشکل موردنظر، **چرخه مالکیت یا Ownership Cycle** است.

اگر چرخه مالکیت را به‌درستی درک کنیم، تقریباً دلیل اصلی وجود `weak_ptr` را نیز درک کرده‌ایم.

---

<a id="section-2"></a>

# 2. قبل از `weak_ptr` چه مشکلی وجود داشت؟

برای درک `weak_ptr` بهتر است ابتدا بدانیم `shared_ptr` چرا اصلاً به وجود آمد.

اشاره گر `shared_ptr` برای زمانی است که چند بخش مختلف برنامه باید مالک یک شیء باشند.

برای مثال:

```cpp
auto person = std::make_shared<Person>();

auto a = person;
auto b = person;
```

در اینجا سه `shared_ptr` داریم:

```text
person ──┐
         │
a ───────┼──> Person
         │
b ───────┘
```

اینجا مهم این است که هر سه `shared_ptr` مالک همان `Person` هستند.

مشخصه Reference Count نیز تعداد مالک‌های فعال را دنبال می‌کند.

```text
person + a + b = 3 owners
```

وقتی یکی از آنها از بین برود:

```text
person + a = 2 owners
```

وقتی دیگری نیز از بین برود:

```text
person = 1 owner
```

و در نهایت وقتی آخرین مالک از بین برود:

```text
0 owners
```

شیء نیز Destroy می‌شود.

این رفتار بسیار خوب است.

اما یک مشکل بسیار مهم وجود دارد.

---

<a id="section-3"></a>

# 3. `shared_ptr` چگونه مالکیت را مدیریت می‌کند؟

اشاره گر `shared_ptr` معمولاً از یک ساختار داخلی به نام **Control Block** استفاده می‌کند.

نمایش ساده‌شده آن می‌تواند چیزی شبیه این باشد:

```text
              Control Block
          ┌───────────────────┐
          │ Strong Count      │
          │ Weak Count        │
          │ Deleter           │
          │ Other information │
          └─────────┬─────────┘
                    │
                    ▼
                 Object
```

شمارنده Strong Count تعداد مالک‌های `shared_ptr` را نشان می‌دهد.

نکته مهم این است که تا زمانی که Strong Count صفر نشده باشد، شیء معمولاً Destroy نمی‌شود.

برای مثال:

```cpp
auto p1 = std::make_shared<int>(42);
auto p2 = p1;
auto p3 = p2;
```

اکنون:

```text
Strong Count = 3
```

وقتی:

```cpp
p1.reset();
```

اجرا شود:

```text
Strong Count = 2
```

و وقتی:

```cpp
p2.reset();
```

اجرا شود:

```text
Strong Count = 1
```

در نهایت:

```cpp
p3.reset();
```

باعث می‌شود آخرین مالک نیز از بین برود.

در نتیجه شیء می‌تواند Destroy شود.

---

<a id="section-4"></a>

# 4. مشکل اصلی `shared_ptr`: چرخه مالکیت و Memory Leak

نکته مهم این است که Reference Counting زمانی بسیار خوب کار می‌کند که مالکیت‌های اشاره گرهای Shared_ptr یک چرخه ایجاد نکنند.

برای درک مشکل، فرض کنید دو کلاس داریم:

```cpp
class B;

class A
{
public:
    std::shared_ptr<B> b;
};

class B
{
public:
    std::shared_ptr<A> a;
};
```

مفهوم کد بالا آن است که `A` مالک `B` است و `B` نیز مالک `A`. حالا:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();

a->b = b;
b->a = a;
```

اکنون ساختار مالکیت به شکل زیر است:

```text
             ┌───────────────┐
             │               │
             │               ▼
      ┌───────────┐      ┌───────────┐
      │  Object A │ ────▶│  Object B │
      └───────────┘      └───────────┘
             ▲               │
             │               │
             └───────────────┘
```

عبارت دقیق‌تر این است:

```text
A owns B
B owns A
```

این یک **Ownership Cycle** است.

---

نکته بسیار مهم این قسمت همین‌جاست.

فرض کنید متغیرهای محلی `a` و `b` از Scope خارج شوند.

ممکن است تصور کنیم:

```text
a destroyed
b destroyed

therefore:

A destroyed
B destroyed
```

اما چنین اتفاقی لزوماً رخ نمی‌دهد.چرا؟

دلیل این است که `A` هنوز یک `shared_ptr` به `B` دارد.

و `B` نیز هنوز یک `shared_ptr` به `A` دارد.

یعنی:

```text
A ──shared_ptr──> B
▲                │
│                │
└──shared_ptr────┘
```

بنابراین:

```text
A has 1 owner
B has 1 owner
```

حتی اگر هیچ کد دیگری از بیرون مالک آنها نباشد.

معنی کلیدی این است:

> هر شیء توسط شیء دیگر زنده نگه داشته شده است.

در نتیجه:

```text
A → B → A → B → A → ...
```

چرخه هیچ‌وقت شکسته نمی‌شود.

نکته بسیار مهم این است که **Reference Counting خراب نشده است**.

در اینجا Reference Counting دقیقاً همان کاری را انجام می‌دهد که برای آن طراحی شده است.

هر `shared_ptr` می‌گوید:

> من هنوز مالک این شیء هستم.

و چون `A` مالک `B` است و `B` مالک `A` است، شمارنده هیچ‌کدام نمی‌تواند به صفر برسد.

این همان مشکلی است که `weak_ptr` برای حل آن ایجاد شده است.

---

نکته مهم این است که `weak_ptr` خودش Memory Leak را به طور عمومی درمان نمی‌کند.

کار اصلی آن جلوگیری از **چرخه مالکیت** در ساختارهای `shared_ptr` است.

فرض کنید:

```cpp
A ──shared_ptr──> B
B ──shared_ptr──> A
```

چرخه ایجاد می‌شود.

اگر یکی از روابط را به:

```cpp
B ──weak_ptr──> A
```

تبدیل کنیم:

```text
A ──shared_ptr──> B
B ──weak_ptr────> A
```

دیگر `B` باعث زنده ماندن `A` نمی‌شود.

در نتیجه وقتی مالک واقعی `A` از بین برود، `A` می‌تواند Destroy شود.

این دقیقاً جایی است که `weak_ptr` می‌تواند از Memory Leak جلوگیری کند.

---

<a id="section-5"></a>

# 5. `weak_ptr` دقیقاً چیست؟

اشاره گر `weak_ptr` یک Smart Pointer است که می‌تواند به شیئی که توسط `shared_ptr` مدیریت می‌شود **اشاره کند، بدون اینکه مالک آن باشد**.

عبارت کلیدی:

> **Observe, but do not own.**

یعنی:

> مشاهده کن، اما مالک نباش.

برای مثال:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;
```

در اینجا:

```text
shared ──owns──> int

weak ──observes──> int
```

نکته مهم این است که ایجاد `weak` باعث افزایش Strong Count نمی‌شود.

اگر:

```cpp
shared.use_count()
```

قبل از ساخت `weak` برابر `1` باشد، بعد از ساخت `weak` نیز Strong Count همچنان `1` خواهد بود.

---

این مهم‌ترین ویژگی `weak_ptr` است.

عبارت زیر:

```cpp
std::weak_ptr<MyClass> weak;
```

به این معنا نیست که `weak` مالک `MyClass` است.

تفاوت آن فقط این است که `weak` یک ارتباط غیرمالکانه با یک Object دارد.

به همین دلیل، `weak_ptr` باعث افزایش Strong Reference Count نمی‌شود.

---

<a id="section-6"></a>

# 6. تفاوت اساسی `shared_ptr` و `weak_ptr`

عبارت ساده برای فهم تفاوت این دو این است:

```text
shared_ptr:
"I own the object."

weak_ptr:
"I know about the object, but I do not own it."
```

اشاره گر `shared_ptr` باعث زنده ماندن Object می‌شود.

اشاره گر `weak_ptr` باعث زنده ماندن Object نمی‌شود.

مثلاً:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

shared.reset();
```

اکنون اگر `shared` آخرین مالک بوده باشد، `int` Destroy می‌شود.

اما `weak` همچنان ممکن است وجود داشته باشد.

فقط دیگر نمی‌تواند به یک Object زنده دسترسی پیدا کند.

---

<a id="section-7"></a>

# 7. `weak_ptr` چگونه چرخه مالکیت را می‌شکند؟

نکته اصلی این است که در یک چرخه مالکیت، حداقل یکی از روابط مالکیت باید غیرمالکانه باشد.

برای مثال، به جای:

```cpp
class B
{
public:
    std::shared_ptr<A> a;
};
```

می‌توانیم بنویسیم:

```cpp
class B
{
public:
    std::weak_ptr<A> a;
};
```

اکنون:

```text
A ──shared_ptr──> B
B ──weak_ptr────> A
```

اینجا نکته مهم این است که `A` مالک `B` است.

اما `B` فقط `A` را مشاهده می‌کند.

در نتیجه:

```text
A owns B
B observes A
```

دیگر چرخه مالکیت وجود ندارد.

---

<a id="section-8"></a>

# 8. ساختن و قرار دادن `weak_ptr` در Setter

ساختن `weak_ptr` معمولاً از یک `shared_ptr` انجام می‌شود:

```cpp
auto shared = std::make_shared<MyClass>();

std::weak_ptr<MyClass> weak = shared;
```

حال `weak` فقط به همان Control Block متصل می‌شود.

همچنین `weak_ptr` نمی‌تواند به صورت مستقل یک Object را با `new` مدیریت کند.

برای مثال، این طراحی معمولاً معنای درستی ندارد:

```cpp
std::weak_ptr<MyClass> weak(new MyClass());
```

چون `weak_ptr` اساساً برای مشاهده Objectهایی طراحی شده که Ownership آنها توسط `shared_ptr` مدیریت می‌شود.

---

اگر یک کلاس قرار است یک رابطه غیرمالکانه با Objectی داشته باشد، Setter می‌تواند `weak_ptr` دریافت کند.

برای مثال:

```cpp
class Observer
{
private:
    std::weak_ptr<MyClass> value;

public:
    void setValue(std::weak_ptr<MyClass> ptr)
    {
        value = std::move(ptr);
    }
};
```

حالا:

```cpp
auto object = std::make_shared<MyClass>();

Observer observer;

observer.setValue(object);
```

در اینجا `observer` مالک `object` نشده است.

اگر:

```cpp
object.reset();
```

اجرا شود و `object` آخرین مالک باشد، Object Destroy خواهد شد.

---

<a id="section-9"></a>

# 9. گرفتن `weak_ptr` از Getter

اگر یک کلاس یک ارتباط غیرمالکانه را نگهداری می‌کند، می‌توانیم آن را با Getter برگردانیم:

```cpp
std::weak_ptr<MyClass> getValue() const
{
    return value;
}
```

نکته مهم این است که بازگرداندن `weak_ptr` باعث تبدیل آن به Ownership نمی‌شود.
 Caller می‌تواند یک کپی از `weak_ptr` دریافت کند.

---

در بسیاری از طراحی‌ها:

```cpp
std::weak_ptr<MyClass> getValue() const
{
    return value;
}
```

انتخاب مناسبی است.

این API به Caller می‌گوید:

> این Object ممکن است وجود داشته باشد، اما من مالک آن نیستم و نمی‌توانم صرفاً با دریافت این مقدار، عمر آن را افزایش دهم.

این موضوع در طراحی کلاس بسیار مهم است.

اگر Getter به جای آن:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value.lock();
}
```

برگرداند، Caller یک مالک جدید دریافت خواهد کرد.

بنابراین این دو Getter از نظر طراحی کاملاً متفاوت هستند.

---

<a id="section-10"></a>

# 10. عملیات `Copy`، `Move` و `reset()` در `weak_ptr`

اشاره گر `weak_ptr` نیز مانند سایر Objectهای معمولی قابل Copy است. برای مثال:

```cpp
auto shared = std::make_shared<MyClass>();

std::weak_ptr<MyClass> w1 = shared;
std::weak_ptr<MyClass> w2 = w1;
```

اکنون:

```text
w1 ──┐
     ├──> same Control Block
w2 ──┘
```

نکته مهم این است که با Copy کردن `weak_ptr`، دیگر Object کپی نمی‌شود.

حتی Ownership جدیدی نیز ایجاد نمی‌شود.

فقط یک Observer جدید ساخته می‌شود.

---

اشاره گر `weak_ptr` از Copy و Move پشتیبانی می‌کند.

کپی:

```cpp
auto w2 = w1;
```

یک Observer جدید ایجاد می‌کند.

Move:

```cpp
auto w2 = std::move(w1);
```

وضعیت `w1` را به `w2` منتقل می‌کند.

پس از Move، `w1` معمولاً Empty خواهد بود.

---

متد `reset()` ارتباط `weak_ptr` با Control Block را قطع می‌کند. برای مثال:

```cpp
std::weak_ptr<MyClass> weak = shared;

weak.reset();
```

اکنون `weak` دیگر Object را مشاهده نمی‌کند.

در اینجا `reset()` روی `weak_ptr` باعث Destroy شدن Object نمی‌شود.

چون `weak_ptr` مالک Object نیست.

اگر:

```cpp
shared.use_count() == 1
```

باشد، اجرای:

```cpp
weak.reset();
```

هیچ تأثیری روی عمر Object ندارد.

Object همچنان توسط `shared` مدیریت می‌شود.

---

<a id="section-11"></a>

# 11. بررسی خالی بودن و `expired()`

برای بررسی اینکه `weak_ptr` هیچ ارتباطی با Object ندارد، می‌توانیم از:

```cpp
weak.expired()
```

استفاده کنیم. برای مثال:

```cpp
std::weak_ptr<MyClass> weak;

if (weak.expired())
{
    // No live object is available through weak
}
```

اما یک نکته بسیار مهم وجود دارد.

متد `expired()` به تنهایی معمولاً برای تصمیم‌گیری نهایی درباره دسترسی به Object مناسب نیست.

---

متد `expired()` مشخص می‌کند که Object مورد مشاهده توسط `weak_ptr` دیگر زنده است یا خیر. برای مثال:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

std::cout << weak.expired();
```

تا زمانی که `shared` Object را زنده نگه داشته است، نتیجه `false` خواهد بود.

بعد:

```cpp
shared.reset();
```

اگر `shared` آخرین مالک باشد:

```cpp
weak.expired()
```

برابر `true` خواهد شد.

متد `expired()` فقط وضعیت را در یک لحظه بررسی می‌کند.

---

<a id="section-12"></a>

# 12. تبدیل `weak_ptr` به `shared_ptr` با `lock()`

این قسمت یکی از مهم‌ترین بخش‌های `weak_ptr` است.

اشاره گر `weak_ptr` مستقیماً اجازه استفاده از:

```cpp
weak->someFunction();
```

را نمی‌دهد.

ابتدا باید آن را با `lock()` به یک `shared_ptr` تبدیل کنیم:

```cpp
auto shared = weak.lock();
```

اگر Object هنوز زنده باشد، `shared` یک `shared_ptr` معتبر خواهد بود.

اگر Object از بین رفته باشد، `shared` خالی خواهد بود.

برای مثال:

```cpp
if (auto shared = weak.lock())
{
    shared->doSomething();
}
```

این یکی از مهم‌ترین الگوهای استفاده از `weak_ptr` است.

---

متد `lock()` فقط بررسی نمی‌کند Object زنده است. بلکه اگر Object زنده باشد، یک `shared_ptr` ایجاد می‌کند که به صورت موقت Ownership را حفظ می‌کند. این موضوع بسیار مهم است.فرض کنید:

```cpp
if (!weak.expired())
{
    // ...
}
```

و سپس در زمان دیگری بخواهیم Object را استفاده کنیم.

در محیط‌های چندتردی یا حتی طراحی‌های پیچیده‌تر، ممکن است بین بررسی و استفاده، Object Destroy شود.

به همین دلیل الگوی بهتر این است:

```cpp
if (auto shared = weak.lock())
{
    shared->doSomething();
}
```

مفهوم این کد این است:

> اگر Object هنوز زنده است، یک مالک موقت ایجاد کن و سپس از Object استفاده کن.

این کار بسیار امن‌تر است.

---

کاری که `lock()` انجام می‌دهد این است که روی **Control Block** به‌صورت اتمیک بررسی می‌کند که آیا شیء موردنظر هنوز زنده است یا خیر؛ و اگر زنده باشد، در همان عملیات یک `shared_ptr` جدید ایجاد می‌کند تا مالکیت شیء را به دست بگیرد.

اهمیت این رفتار زمانی مشخص می‌شود که هم‌زمان یک `shared_ptr` دیگر وجود داشته باشد که **آخرین مالک شیء** است و در همان لحظه قصد `reset()` کردن آن را دارد. در چنین شرایطی، `weak_ptr::lock()` و `reset()` به‌صورت اتمیک تعیین تکلیف می‌کنند که کدام وضعیت برقرار است:

* اگر `lock()` ابتدا موفق شود، یک `shared_ptr` جدید ایجاد می‌شود و **Reference Count از ۱ به ۲ می‌رسد**. سپس اگر `shared_ptr` قبلی `reset()` شود، Reference Count به ۱ کاهش پیدا می‌کند و شیء همچنان زنده می‌ماند.
* اگر `reset()` ابتدا انجام شود و در نتیجه Reference Count به صفر برسد، شیء از بین می‌رود و `lock()` نیز یک `shared_ptr` خالی (`nullptr`) برمی‌گرداند.

بنابراین نکته‌ی مهم این است که `lock()` صرفاً این کار را انجام نمی‌دهد که ابتدا **وجود شیء را بررسی کند** و بعد، در یک مرحله‌ی جداگانه، `shared_ptr` بسازد. چنین رویکردی می‌توانست یک Race Condition ایجاد کند:

```text
آیا شیء وجود دارد؟ → بله
                     ↓
                 [Thread دیگر]
                     ↓
                   reset()
                     ↓
                 object destroyed
                     ↓
          حالا استفاده از object ❌
```

در عوض، `lock()` عملیات **«بررسی زنده بودن شیء + به‌دست‌آوردن مالکیت»** را به‌صورت اتمیک انجام می‌دهد. در نتیجه، یا قبل از اینکه آخرین مالک `reset()` شود، مالکیت جدید را به دست می‌آورد و شیء را زنده نگه می‌دارد، یا اگر آخرین مالک قبلاً مالکیت خود را رها کرده باشد، `lock()` با نتیجه‌ی خالی مواجه می‌شود.

به بیان ساده‌تر، `weak_ptr::lock()` تضمین می‌کند که بین **«شیء هنوز زنده است»** و **«من مالکیت آن را به دست آوردم»** یک فاصله‌ی زمانیِ قابل سوءاستفاده وجود نداشته باشد.

---

<a id="section-13"></a>

# 13. دسترسی به اعضای شیء با `weak_ptr`

اشاره گر `weak_ptr` خودش `operator->` برای دسترسی مستقیم به Object ندارد.

بنابراین این کد صحیح نیست:

```cpp
weak->doSomething();
```

به جای آن:

```cpp
if (auto ptr = weak.lock())
{
    ptr->doSomething();
}
```

و برای Member:

```cpp
if (auto ptr = weak.lock())
{
    std::cout << ptr->value;
}
```

---

این تصمیم کاملاً عمدی است.

دلیل این است که Object ممکن است دیگر وجود نداشته باشد.

اگر `weak_ptr` اجازه می‌داد:

```cpp
weak->doSomething();
```

برنامه‌نویس ممکن بود بدون توجه به Lifetime از Object استفاده کند.

متد `lock()` برنامه‌نویس را مجبور می‌کند ابتدا وضعیت Lifetime را بررسی کند و در صورت امکان، یک `shared_ptr` موقت ایجاد کند.

---

<a id="section-14"></a>

# 14. `weak_ptr` و Raw Pointer

به صورت مستقیم، `weak_ptr` تابعی مانند:

```cpp
get()
```

که یک Raw Pointer به Object بدهد ندارد.

این موضوع نیز عمدی است.

برای رسیدن به Object باید ابتدا:

```cpp
auto shared = weak.lock();
```

انجام شود.

سپس:

```cpp
MyClass* raw = shared.get();
```

در اختیار ما خواهد بود.

---

اتفاق خطرناک این است که بعد از گرفتن Raw Pointer، مدیریت Lifetime  آن را خودمان به دست بگیریم.

برای مثال:

```cpp
auto shared = weak.lock();

if (shared)
{
    MyClass* raw = shared.get();

    raw->doSomething();
}
```

در این مثال مشکلی وجود ندارد، چون `shared` تا پایان Scope زنده است.

اما اگردر یک بلاک اصلی بنویسیم:

```cpp
MyClass* raw = weak.lock().get();
```

مشکل بسیار جدی ایجاد می‌شود.

تابع `lock()` یک Temporary `shared_ptr` ساخته است.

شاید قبل از پایان اجرای این بلاک از کد و بعد از گرفتن یک Raw Pointer، شی اصلی در Heap از بین برود و پس از از بین رفتنش، ما بدون اطلاع از این اتفاق، در انتهای این بلوک از کد و پس از Delete شدن شی اصلی، به واسطه این Raw Pointer بخواهیم به شی Delete شده دسترسی پیدا کنیم و با خطای زمان اجرا به جهت Dangling Pointer مواجه شویم!

در نتیجه Raw Pointer ممکن است Dangling شود.

بنابراین این کار بسیار خطرناک است:

```cpp
MyClass* raw = weak.lock().get();
```

به جای آن:

```cpp
if (auto shared = weak.lock())
{
    MyClass* raw = shared.get();

    raw->doSomething();
}
```

امن‌تر است.

---

<a id="section-15"></a>

# 15. استفاده از `weak_ptr` در کلاس و الگوی Parent/Child

یک مثال بسیار رایج، نگهداری Parent توسط Child است.

فرض کنید:

```cpp
class Parent;

class Child
{
private:
    std::weak_ptr<Parent> parent;

public:
    void setParent(std::shared_ptr<Parent> p)
    {
        parent = p;
    }

    std::shared_ptr<Parent> getParent() const
    {
        return parent.lock();
    }
};
```

در اینجا `Child` می‌داند Parent آن چه کسی است.

اما Child مالک Parent نیست.

این موضوع از ایجاد چرخه جلوگیری می‌کند.

---

مثال Parent/Child (الگوی Composition) یکی از بهترین مثال‌ها برای درک `weak_ptr` است.

فرض کنید:

```text
Parent
   │
   │ owns
   ▼
Child
```

اگر Child نیز Parent را با `shared_ptr` نگه دارد:

```text
Parent
   │
   │ shared_ptr
   ▼
Child
   │
   │ shared_ptr
   ▼
Parent
```

چرخه ایجاد می‌شود.

معمولا روش صحیح این است:

```text
Parent
   │
   │ shared_ptr
   ▼
Child
   │
   │ weak_ptr
   ▼
Parent
```

در این حالت Parent مالک Child است.

اما Child فقط Parent را مشاهده می‌کند.

---

<a id="section-16"></a>

# 16. الگوی Observer

یکی دیگر از کاربردهای مهم `weak_ptr` الگوی Observer است.

فرض کنید یک Object اصلی داریم:

```cpp
auto subject = std::make_shared<Subject>();
```

و چند Observer داریم که باید بدانند Subject چه زمانی وجود دارد.

اگر Observerها `shared_ptr` نگه دارند، ممکن است ناخواسته باعث زنده ماندن Subject شوند.

به جای آن:

```cpp
std::weak_ptr<Subject> subject;
```

مناسب‌تر است.

مفهوم Observer این است:

> من می‌خواهم در صورت وجود Object به آن دسترسی داشته باشم، اما نباید باعث زنده ماندن آن شوم.

---

<a id="section-17"></a>

# 17. مثال کامل از چرخه خراب و اصلاح آن با `weak_ptr`

مثال زیر یکی از مهم‌ترین مثال‌ها برای درک `weak_ptr` است:

```cpp
#include <iostream>
#include <memory>

class B;

class A
{
public:
    std::shared_ptr<B> b;

    ~A()
    {
        std::cout << "A destroyed\n";
    }
};

class B
{
public:
    std::shared_ptr<A> a;

    ~B()
    {
        std::cout << "B destroyed\n";
    }
};
```

سپس:

```cpp
{
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();

    a->b = b;
    b->a = a;
}
```

مفهوم ظاهری این است که با پایان Scope باید هر دو Object Destroy شوند.

اما چنین اتفاقی رخ نمی‌دهد.

دلیل این است:

```text
A owns B
B owns A
```

در نتیجه هر Object توسط دیگری زنده نگه داشته شده است.

---

برای اصلاح، یکی از روابط را غیرمالکانه می‌کنیم:

```cpp
class B
{
public:
    std::weak_ptr<A> a;

    ~B()
    {
        std::cout << "B destroyed\n";
    }
};
```

اکنون:

```text
A ──shared_ptr──> B
B ──weak_ptr────> A
```

مفهوم Ownership Cycle از بین رفته است.

وقتی Scope تمام شود:

```text
a destroyed
b destroyed
```

در نتیجه Strong Count مربوط به Objectها می‌تواند به صفر برسد.

پس Objectها نیز Destroy می‌شوند.

---

<a id="section-18"></a>

# 18. `use_count()`، Strong Count، Weak Count و Control Block

اشاره گر `weak_ptr` نیز تابع:

```cpp
use_count()
```

دارد.

برای مثال:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;

std::cout << weak.use_count();
```

مقدار نمایش‌داده‌شده تعداد **Strong Ownerها** است.

یعنی تعداد `shared_ptr`هایی که Object را مالک هستند.

حالا در اینجا اشاره گر `weak_ptr` به این عدد اضافه نمی‌کند.

---

مفهوم Control Block معمولاً دو مفهوم شمارشی مهم دارد:

```text
Strong Count
Weak Count
```

شمارنده Strong Count تعداد مالک‌های واقعی یعنی `shared_ptr`ها را نشان می‌دهد.

شمارنده Weak Count مربوط به وجود Observerهای `weak_ptr` و مدیریت خود Control Block است.

این دو Count را نباید با یکدیگر اشتباه گرفت.

مهم‌ترین قاعده برای Object این است:

```text
Strong Count == 0
```

می‌تواند باعث Destroy شدن Object شود.

اما ممکن است Control Block همچنان به دلیل وجود `weak_ptr` باقی بماند.

---

مفهوم Control Block یک ساختار داخلی است که `shared_ptr` و `weak_ptr` از آن برای مدیریت Ownership استفاده می‌کنند.

نمایش ساده‌شده:

```text
             Control Block
       ┌───────────────────────┐
       │ Strong Count          │
       │ Weak Count            │
       │ Deleter               │
       │ Other metadata        │
       └───────────┬───────────┘
                   │
                   ▼
                 Object
```

نکته مهم این است که `weak_ptr` مستقیماً مالک Object نیست.

بلکه به Control Block مربوط به Ownership آن Object متصل است.

به همین دلیل می‌تواند متوجه شود که Object هنوز زنده است یا نه.

---

این سؤال بسیار مهم است.

فرض کنید:

```cpp
auto shared = std::make_shared<int>(42);

std::weak_ptr<int> weak = shared;
```

اکنون:

```text
Strong Count = 1
```

و یک `weak_ptr` نیز وجود دارد.

اگر:

```cpp
shared.reset();
```

اجرا شود:

```text
Strong Count = 0
```

Object Destroy می‌شود.

اما ممکن است Control Block همچنان باقی بماند تا `weak` بتواند وضعیت آن را تشخیص دهد.

پس:

```text
Object lifetime
```

و:

```text
Control Block lifetime
```

دو مفهوم کاملاً متفاوت هستند.

این یکی از نکات مهم در درک داخلی `weak_ptr` است.

---

<a id="section-19"></a>

# 19. تفاوت `weak_ptr` و `shared_ptr`

جدول زیر تفاوت اصلی را نشان می‌دهد:

| ویژگی                          | `shared_ptr` | `weak_ptr`            |
| ------------------------------ | ------------ | --------------------- |
| مالک Object است؟               | بله          | خیر                   |
| Strong Count را افزایش می‌دهد؟ | بله          | خیر                   |
| Object را زنده نگه می‌دارد؟    | بله          | خیر                   |
| Copy دارد؟                     | بله          | بله                   |
| Move دارد؟                     | بله          | بله                   |
| `reset()` دارد؟                | بله          | بله                   |
| `get()` مستقیم دارد؟           | بله          | خیر                   |
| `operator->` دارد؟             | بله          | خیر                   |
| `lock()` دارد؟                 | خیر          | بله                   |
| `expired()` دارد؟              | خیر          | بله                   |
| مناسب چرخه مالکیت است؟         | خطرناک       | مناسب برای شکستن چرخه |
| Ownership ایجاد می‌کند؟        | بله          | خیر                   |

---

<a id="section-20"></a>

# 20. تفاوت `weak_ptr` و `unique_ptr`

اشاره گرهای `unique_ptr` و `weak_ptr` حتی از نظر فلسفه Ownership نیز متفاوت هستند.

اشاره گر `unique_ptr` یعنی:

> من تنها مالک این Object هستم.

اشاره گر `weak_ptr` یعنی:

> من اصلاً مالک این Object نیستم؛ فقط می‌خواهم آن را مشاهده کنم.

بنابراین:

```cpp
std::unique_ptr<MyClass>
```

برای **Unique Ownership** است.

اما:

```cpp
std::weak_ptr<MyClass>
```

برای **Non-owning Observation** از Objectی است که توسط `shared_ptr` مدیریت می‌شود.

---

<a id="section-21"></a>

# 21. چه زمانی باید یا نباید از `weak_ptr` استفاده کنیم؟

1- وقتی یک Object را می‌خواهیم مشاهده کنیم ولی نمی‌خواهیم باعث افزایش Lifetime آن شویم، `weak_ptr` گزینه مناسبی است.

2- وقتی روابط بین Objectها ممکن است چرخه ایجاد کنند، یکی از روابط Ownership را می‌توان با `weak_ptr` مدل کرد.

3- در Observer Pattern، Parent/Child Relationship و Cacheها، `weak_ptr` می‌تواند انتخاب مناسبی باشد.

مفهوم کلی این است:

> هر جا رابطه وجود دارد، الزاماً Ownership وجود ندارد.

این جمله یکی از بهترین راه‌ها برای درک `weak_ptr` است.

---

باید بدانیم نباید صرفاً به خاطر اینکه `weak_ptr` ابزار قدرتمندی است، همه‌جا از آن استفاده کنیم.

اگر یک کلاس واقعاً باید مالک Object باشد، `weak_ptr` انتخاب درستی نیست.

برای مثال:

```cpp
class Car
{
    std::weak_ptr<Engine> engine;
};
```

اگر Car بدون Engine نمی‌تواند کار کند و باید Engine را زنده نگه دارد، احتمالاً `shared_ptr` یا حتی `unique_ptr` انتخاب بهتری است.

اشاره گر `weak_ptr` زمانی معنا دارد که عدم Ownership بخشی از طراحی باشد.

---

<a id="section-22"></a>

# 22. اشتباهات رایج

یکی از اشتباهات رایج این است که فکر کنیم `weak_ptr` باعث زنده ماندن Object می‌شود.

اشتباه دیگر این است که `weak_ptr` را مستقیماً Dereference کنیم:

```cpp
weak->foo();
```

این امکان وجود ندارد.

روش صحیح:

```cpp
if (auto ptr = weak.lock())
{
    ptr->foo();
}
```

اشتباه دیگر استفاده از این متد است:

```cpp
if (!weak.expired())
{
    auto ptr = weak.lock();
}
```

در اغلب موارد این بررسی اضافی است.

روش بهتر:

```cpp
if (auto ptr = weak.lock())
{
    ptr->foo();
}
```

اشتباه خطرناک دیگر گرفتن Raw Pointer از Temporary است:

```cpp
auto raw = weak.lock().get();
```

این Raw Pointer می‌تواند بلافاصله Dangling شود.

عبارت صحیح‌تر:

```cpp
if (auto ptr = weak.lock())
{
    auto raw = ptr.get();

    raw->foo();
}
```

البته در بسیاری از موارد اصلاً نیازی به Raw Pointer نداریم و بهتر است مستقیماً از:

```cpp
ptr->foo();
```

استفاده کنیم.

---

<a id="section-23"></a>

# 23. یک مثال کامل و واقعی‌تر

کد زیر یک طراحی رایج Parent/Child را نشان می‌دهد:

```cpp
#include <iostream>
#include <memory>

class Parent;

class Child
{
private:
    std::weak_ptr<Parent> parent;

public:
    void setParent(const std::shared_ptr<Parent>& p)
    {
        parent = p;
    }

    void printParent() const
    {
        if (auto p = parent.lock())
        {
            std::cout << "Parent is alive\n";

            p->hello();
        }
        else
        {
            std::cout << "Parent no longer exists\n";
        }
    }
};

class Parent
{
private:
    std::shared_ptr<Child> child;

public:
    void setChild(std::shared_ptr<Child> c)
    {
        child = std::move(c);
    }

    void hello() const
    {
        std::cout << "Hello from Parent\n";
    }
};
```

سپس:

```cpp
auto parent = std::make_shared<Parent>();
auto child = std::make_shared<Child>();

parent->setChild(child);
child->setParent(parent);
```

رابطه Ownership اینجا به شکل زیر است:

```text
Parent ──shared_ptr──> Child
   ▲
   │
   │ weak_ptr
   │
 Child
```

در این طراحی Child باعث زنده ماندن Parent نمی‌شود.

---

<a id="section-24"></a>

# 24. جمع‌بندی نهایی

اشاره گر `shared_ptr` برای **Shared Ownership** ایجاد شده است.

اشاره گر`weak_ptr` برای زمانی ایجاد شده که می‌خواهیم با Object ارتباط داشته باشیم، اما نمی‌خواهیم مالک آن باشیم.

مشکل اصلی زمانی ایجاد می‌شود که چند `shared_ptr` به صورت چرخه‌ای یکدیگر را زنده نگه دارند.

عبارت:

```text
A owns B
B owns A
```

یک Ownership Cycle ایجاد می‌کند.

نتیجه این چرخه این است که Reference Count هیچ‌گاه به صفر نمی‌رسد.

اشاره گر `weak_ptr` می‌تواند یکی از این روابط را غیرمالکانه کند:

```text
A owns B
B observes A
```

در نتیجه چرخه شکسته می‌شود.

ایجاد `weak_ptr`:

```cpp
auto shared = std::make_shared<MyClass>();

std::weak_ptr<MyClass> weak = shared;
```

روش کپی کردن آن:

```cpp
auto weak2 = weak;
```

عملیات Move کردن آن:

```cpp
auto weak2 = std::move(weak);
```

روش Reset کردن آن:

```cpp
weak.reset();
```

عملیات بررسی منقضی شدن:

```cpp
weak.expired();
```

روش تبدیل امن به `shared_ptr`:

```cpp
auto shared = weak.lock();
```

دسترسی به Object:

```cpp
if (auto shared = weak.lock())
{
    shared->doSomething();
}
```

روش دسترسی به Raw Pointer:

```cpp
if (auto shared = weak.lock())
{
    MyClass* raw = shared.get();
}
```

نکته مهم این است که `weak_ptr` مستقیماً `get()` ندارد و برای دسترسی به Object باید ابتدا با `lock()` یک `shared_ptr` معتبر دریافت کنیم.

نکته مهم دیگر این است که `weak_ptr` مالک Object نیست و بنابراین نمی‌تواند Object را زنده نگه دارد.

---

اشاره گر Smart Pointerها را می‌توان با چهار سؤال ساده در ذهن نگه داشت:

```text
Who owns the object?
Who shares ownership?
Who only observes it?
Can ownership form a cycle?
```

پاسخ‌ها معمولاً چنین هستند:

```text
One owner
    → unique_ptr

Multiple owners
    → shared_ptr

Non-owning observer of shared ownership
    → weak_ptr
```

نکته نهایی این است که `weak_ptr` یک نسخه «ضعیف‌تر» از `shared_ptr` نیست.

عبارت `weak` در نام آن به این معنا نیست که قابلیت کمتری دارد.

مفهوم واقعی این نام این است:

> **Weak Ownership — یا دقیق‌تر، عدم وجود Strong Ownership.**

اشاره گر `weak_ptr` عمداً Object را زنده نگه نمی‌دارد.

و دقیقاً به همین دلیل است که می‌تواند چرخه‌های مالکیت `shared_ptr` را بشکند.

اگر یک برنامه‌نویس فقط یک جمله از این آموزش به خاطر بسپارد، بهتر است این جمله باشد:

> **`shared_ptr` می‌گوید «تا وقتی من هستم، Object باید زنده باشد»؛ `weak_ptr` می‌گوید «اگر Object هنوز زنده است، به من اجازه بده آن را ببینم، اما وجود من نباید مانع نابودی آن شود».**

---
## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>