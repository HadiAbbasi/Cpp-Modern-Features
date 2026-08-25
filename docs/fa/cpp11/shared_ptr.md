<div align="right">

[🇺🇸 English](../../en/cpp11/shared_ptr.md) | [🇮🇷 فارسی](./shared_ptr.md)

</div>
---

# آموزش کامل `std::shared_ptr` در C++11؛ مالکیت اشتراکی، شمارنده مرجع و تفاوت با `unique_ptr`

## فهرست مطالب

- [مقدمه](#مقدمه)
- [مشکل اصلی چه بود؟](#مشکل-اصلی-چه-بود)
- [shared_ptr چه مشکلی را حل کرد؟](#shared_ptr-چه-مشکلی-را-حل-کرد)
- [شمارندهٔ داخلی shared_ptr چیست؟](#شمارندهٔ-داخلی-shared_ptr-چیست)
- [شمارنده چگونه قابل مشاهده است؟](#شمارنده-چگونه-قابل-مشاهده-است)
- [یک نکته مهم درباره use_count()](#یک-نکته-مهم-درباره-use_count)
- [ساخت shared_ptr](#ساخت-shared_ptr)
- [چرا make_shared بهتر است؟](#چرا-make_shared-بهتر-است)
- [وارد کردن shared_ptr در setter](#وارد-کردن-shared_ptr-در-setter)
- [چرا setter را با value دریافت کنیم؟](#چرا-setter-را-با-value-دریافت-کنیم)
- [آیا باید shared_ptr را با const reference بگیریم؟](#آیا-باید-shared_ptr-را-با-const-reference-بگیریم)
- [گرفتن shared_ptr با getter](#گرفتن-shared_ptr-با-getter)
- [چرا getter از نوع shared_ptr می‌تواند مناسب باشد؟](#چرا-getter-از-نوع-shared_ptr-می‌تواند-مناسب-باشد)
- [آیا getter باید shared_ptr& برگرداند؟](#آیا-getter-باید-shared_ptr-برگرداند)
- [آیا کپی shared_ptr یعنی کپی شدن object؟](#آیا-کپی-shared_ptr-یعنی-کپی-شدن-object)
- [چگونه از shared_ptr یک Duplicate بگیریم؟](#چگونه-از-shared_ptr-یک-duplicate-بگیریم)
- [اگر منظور duplicate واقعی از خود object باشد چه؟](#اگر-منظور-duplicate-واقعی-از-خود-object-باشد-چه)
- [دسترسی به مقدار داخلی shared_ptr](#دسترسی-به-مقدار-داخلی-shared_ptr)
- [دسترسی به object با raw pointer](#دسترسی-به-object-با-raw-pointer)
- [خطرات raw pointer حاصل از get()](#خطرات-raw-pointer-حاصل-از-get)
- [get() مالکیت را منتقل نمی‌کند](#get-مالکیت-را-منتقل-نمی‌کند)
- [یک اشتباه بسیار خطرناک با raw pointer](#یک-اشتباه-بسیار-خطرناک-با-raw-pointer)
- [آیا دسترسی به داخل shared_ptr باعث Memory Leak می‌شود؟](#آیا-دسترسی-به-داخل-shared_ptr-باعث-memory-leak-می‌شود)
- [مشکل اصلی shared_ptr؛ چرخهٔ مالکیت](#مشکل-اصلی-shared_ptr-چرخهٔ-مالکیت)
- [چرا شمارندهٔ shared_ptr در این حالت به خوبی کار نمی‌کند؟](#چرا-شمارندهٔ-shared_ptr-در-این-حالت-به-خوبی-کار-نمی‌کند)
- [چرا weak_ptr وجود دارد؟](#چرا-weak_ptr-وجود-دارد)
- [shared_ptr و weak_ptr چه تفاوتی دارند؟](#shared_ptr-و-weak_ptr-چه-تفاوتی-دارند)
- [shared_ptr در یک کلاس](#shared_ptr-در-یک-کلاس)
- [آیا getter بهتر است shared_ptr برگرداند؟](#آیا-getter-بهتر-است-shared_ptr-برگرداند)
- [اگر فقط می‌خواهیم به object دسترسی بدهیم چه کنیم؟](#اگر-فقط-می‌خواهیم-به-object-دسترسی-بدهیم-چه-کنیم)
- [تفاوت مهم getter با unique_ptr](#تفاوت-مهم-getter-با-unique_ptr)
- [shared_ptr و انتقال با move](#shared_ptr-و-انتقال-با-move)
- [تفاوت copy و move در shared_ptr](#تفاوت-copy-و-move-در-shared_ptr)
- [reset در shared_ptr](#reset-در-shared_ptr)
- [reset با object جدید](#reset-با-object-جدید)
- [آیا shared_ptr را می‌توان خالی کرد؟](#آیا-shared_ptr-را-می‌توان-خالی-کرد)
- [shared_ptr و چند thread](#shared_ptr-و-چند-thread)
- [shared_ptr و destructor](#shared_ptr-و-destructor)
- [یک مثال کامل از طراحی کلاس](#یک-مثال-کامل-از-طراحی-کلاس)
- [یک نکته درباره return کردن shared_ptr](#یک-نکته-درباره-return-کردن-shared_ptr)
- [shared_ptr چه زمانی انتخاب مناسبی نیست؟](#shared_ptr-چه-زمانی-انتخاب-مناسبی-نیست)
- [تفاوت shared_ptr و unique_ptr](#تفاوت-shared_ptr-و-unique_ptr)
- [یک مقایسه ساده](#یک-مقایسه-ساده)
- [چه زمانی shared_ptr انتخاب درستی است؟](#چه-زمانی-shared_ptr-انتخاب-درستی-است)
- [چه زمانی unique_ptr بهتر است؟](#چه-زمانی-unique_ptr-بهتر-است)
- [یک قانون طلایی برای انتخاب smart pointer](#یک-قانون-طلایی-برای-انتخاب-smart-pointer)
- [نکات مهم درباره shared_ptr](#نکات-مهم-درباره-shared_ptr)
- [جمع‌بندی نهایی](#جمع‌بندی-نهایی)
- [مشارکت ها](#مشارکت-ها)

---

## مقدمه

اشاره گر `std::shared_ptr` یکی از مهم‌ترین ابزارهای مدیریت حافظه در ++C مدرن است که از استاندارد C++11 وارد زبان شد.

کارکرد اصلی `shared_ptr` زمانی اهمیت پیدا می‌کند که یک شیء در حافظهٔ پویا قرار دارد و **بیش از یک بخش از برنامه باید مالک آن شیء باشد**.

اشاره گر `unique_ptr` برای مالکیت یکتا طراحی شده است، اما `shared_ptr` زمانی استفاده می‌شود که مالکیت باید بین چند `shared_ptr` به‌صورت مشترک تقسیم شود.

عبارت ساده‌تر این است که اگر چند `shared_ptr` به یک شیء اشاره کنند، شیء تا زمانی که **آخرین `shared_ptr` مالک آن باقی مانده است** زنده می‌ماند.

---

# مشکل اصلی چه بود؟

مدیریت دستی حافظه قبل از استفاده گسترده از smart pointerها بر پایهٔ `new` و `delete` بود.

عبارت سادهٔ زیر را در نظر بگیرید:

```cpp
MyClass* ptr = new MyClass();

// استفاده از ptr

delete ptr;
```

مشکل این روش این است که باید دقیقاً مشخص کنیم چه زمانی شیء دیگر مورد نیاز نیست و در همان زمان `delete` را اجرا کنیم.

مشکل زمانی جدی‌تر می‌شود که چند بخش مختلف برنامه به یک شیء نیاز داشته باشند.

عبارت زیر را تصور کنید:

```cpp
MyClass* ptr = new MyClass();

useInModuleA(ptr);
useInModuleB(ptr);
useInModuleC(ptr);

delete ptr;
```

سؤال مهم اینجا این است که چه کسی باید `delete` را انجام دهد؟

اگر `ModuleA` فکر کند مالک شیء است و آن را حذف کند، `ModuleB` و `ModuleC` دیگر نمی‌توانند با اطمینان از آن استفاده کنند.

اگر هیچ‌کدام مسئولیت حذف را قبول نکنند، حافظه نشت خواهد کرد.

---

# shared_ptr چه مشکلی را حل کرد؟

اشاره گر `shared_ptr` برای همین سناریو طراحی شده است؛ یعنی زمانی که **مالکیت یک شیء باید بین چند بخش از برنامه مشترک باشد**.

عبارت زیر چند مالک برای یک شیء ایجاد می‌کند:

```cpp
auto ptr1 = std::make_shared<MyClass>();

auto ptr2 = ptr1;
auto ptr3 = ptr1;
```

اکنون هر سه `shared_ptr` به یک شیء اشاره می‌کنند و در مالکیت آن شریک هستند.

زمانی که `ptr1` از بین برود، شیء حذف نمی‌شود، زیرا `ptr2` و `ptr3` هنوز مالک آن هستند.

عبارت زیر نیز باعث حذف شیء نمی‌شود:

```cpp
ptr1.reset();
```

تنها زمانی شیء حذف می‌شود که **آخرین مالک** نیز از بین برود یا مالکیت خود را رها کند.

---

# شمارندهٔ داخلی shared_ptr چیست؟

اشاره گر `shared_ptr` برای مدیریت این مالکیت مشترک از مفهومی به نام **reference count** یا شمارندهٔ مراجع استفاده می‌کند.

عبارت زیر را در نظر بگیرید:

```cpp
auto p1 = std::make_shared<MyClass>();
```

در این لحظه یک مالک برای شیء وجود دارد.

عبارت زیر:

```cpp
auto p2 = p1;
```

باعث می‌شود تعداد مالکان افزایش پیدا کند.

عبارت دیگر:

```cpp
auto p3 = p1;
```

باعث می‌شود یک مالک دیگر نیز به همان شیء اضافه شود.

می‌توانیم وضعیت را به‌صورت مفهومی این‌گونه نمایش دهیم:

```text
p1 ──┐
     │
p2 ──┼──> MyClass
     │
p3 ──┘

Owners = 3
```

وقتی یکی از `shared_ptr`ها از بین می‌رود، شمارنده کاهش پیدا می‌کند.

عبارت مثلاً اگر `p2` از بین برود:

```text
p1 ──┐
     │
p3 ──┘──> MyClass

Owners = 2
```

زمانی که تعداد مالکان به صفر برسد، شیء حذف می‌شود.

---

# شمارنده چگونه قابل مشاهده است؟

می‌توانیم تعداد مالکانی را که در حال حاضر وجود دارند با `use_count()` مشاهده کنیم.

عبارت نمونه:

```cpp
auto p1 = std::make_shared<MyClass>();

std::cout << p1.use_count();
```

بعد از ایجاد `p1`، مقدار معمولاً `1` است.

عبارت زیر:

```cpp
auto p2 = p1;

std::cout << p1.use_count();
```

اکنون مقدار معمولاً `2` خواهد بود.

عبارت زیر:

```cpp
auto p3 = p1;

std::cout << p1.use_count();
```

اکنون مقدار معمولاً `3` خواهد بود.

متد `use_count()` برای مشاهده و debugging مفید است، اما نباید معمولاً منطق اصلی برنامه را بر اساس مقدار آن طراحی کنیم.

---

# یک نکته مهم درباره use_count()

متد `use_count()` یک ابزار مناسب برای پرسیدن این سؤال نیست که «آیا فقط من مالک هستم؟» در منطق پیچیده برنامه.

عبارت بهتر این است که طراحی برنامه را بر اساس قرارداد مالکیت انجام دهیم، نه اینکه دائماً شمارنده را بررسی کنیم.

همچنین در محیط‌های چندنخی، مقدار `use_count()` ممکن است بلافاصله بعد از دریافت آن دیگر همان مقدار قبلی نباشد.

عبارت بنابراین بهتر است از `use_count()` بیشتر برای مشاهده، تشخیص و debugging استفاده کنیم، نه برای تصمیم‌های حساس در طراحی.

---

# ساخت shared_ptr

در C++11 می‌توانیم مستقیماً از `new` استفاده کنیم:

```cpp
std::shared_ptr<MyClass> ptr(new MyClass());
```

روش ترجیحی این است که از `std::make_shared` استفاده کنیم:

```cpp
auto ptr = std::make_shared<MyClass>();
```

تابع `make_shared` هم خواناتر است و هم معمولاً می‌تواند تخصیص حافظه را به شکل بهینه‌تری انجام دهد.

عبارت مهم این است که اگر از C++11 استفاده می‌کنید، `make_shared` از همان ابتدا در استاندارد وجود داشته است.

---

# چرا make_shared بهتر است؟

در حالت معمول، `make_shared` می‌تواند حافظهٔ مربوط به object و control block را در یک allocation قرار دهد.

عبارت مفهومی آن به شکل زیر است:

```text
        Control Block
      ┌───────────────┐
      │ strong count  │
      │ weak count    │
      │ deleter       │
      └───────┬───────┘
              │
              ▼
           Object
```

این ساختار به‌طور کلی می‌تواند تعداد allocationهای لازم را کاهش دهد.

جزئیات دقیق layout وابسته به implementation کتابخانهٔ استاندارد است، بنابراین نباید به شکل دقیق و ثابت روی layout داخلی `shared_ptr` حساب کنیم.

---

# وارد کردن shared_ptr در setter

اگر یک کلاس مالکیت اشتراکی را قبول می‌کند، معمولاً می‌توان `shared_ptr` را به‌صورت value دریافت کرد.

عبارت نمونه:

```cpp
class Owner
{
private:
    std::shared_ptr<MyClass> value;

public:
    void setValue(std::shared_ptr<MyClass> ptr)
    {
        value = std::move(ptr);
    }
};
```

مفهوم caller می‌تواند یک `shared_ptr` موجود را ارسال کند:

```cpp
auto obj = std::make_shared<MyClass>();

owner.setValue(obj);
```

در این حالت `obj` و `value` هر دو مالک شیء هستند و شمارنده افزایش پیدا می‌کند.

---

# چرا setter را با value دریافت کنیم؟

دریافت `shared_ptr` به‌صورت value می‌تواند مالکیت اشتراکی را بسیار واضح بیان کند.

عبارت زیر:

```cpp
void setValue(std::shared_ptr<MyClass> ptr)
{
    value = std::move(ptr);
}
```

می‌گوید این تابع یک `shared_ptr` دریافت می‌کند و در صورت نیاز مالکیت اشتراکی آن را نگه می‌دارد.

اگر caller چنین چیزی داشته باشد:

```cpp
auto obj = std::make_shared<MyClass>();

owner.setValue(obj);
```

بعد از فراخوانی، هم `obj` و هم `owner.value` مالک شیء هستند.

---

# آیا باید shared_ptr را با const reference بگیریم؟

بستگی به قرارداد تابع دارد.

عبارت اگر تابع فقط می‌خواهد از `shared_ptr` استفاده کند و قرار نیست مالکیت جدیدی ایجاد کند، می‌توان از:

```cpp
void process(const std::shared_ptr<MyClass>& ptr);
```

استفاده کرد.

اگر تابع قرار است `shared_ptr` را ذخیره کند یا مالکیت اشتراکی ایجاد کند، دریافت آن به‌صورت value معمولاً طراحی واضح‌تری است:

```cpp
void setValue(std::shared_ptr<MyClass> ptr);
```

این تفاوت مهم است، زیرا نوع پارامتر باید قرارداد مالکیت را به خوانندهٔ کد منتقل کند.

---

# گرفتن shared_ptr با getter

این قسمت با `unique_ptr` تفاوت بسیار مهمی دارد.

چون `shared_ptr` برای مالکیت اشتراکی طراحی شده است، در بسیاری از APIها برگرداندن `shared_ptr` از getter کاملاً منطقی است.

عبارت نمونه:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

شی caller اکنون یک `shared_ptr` جدید دریافت می‌کند که با شیء داخلی کلاس مالکیت اشتراکی دارد.

عبارت استفاده:

```cpp
auto obj = owner.getValue();
```

اکنون `obj` نیز یکی از مالکان شیء است.

---

# چرا getter از نوع shared_ptr می‌تواند مناسب باشد؟

اگر هدف getter این است که caller بتواند **مالکیت اشتراکی و طول عمر شیء را حفظ کند**، برگرداندن `shared_ptr` انتخاب مناسبی است.

عبارت زیر را در نظر بگیرید:

```cpp
std::shared_ptr<MyClass> obj = owner.getValue();
```

اگر بعداً `owner` از بین برود، `obj` همچنان می‌تواند شیء را زنده نگه دارد.

این دقیقاً یکی از تفاوت‌های اساسی getter برای `shared_ptr` نسبت به getter برای `unique_ptr` است.

---

# آیا getter باید shared_ptr& برگرداند؟

معمولاً بهتر است از برگرداندن reference به `shared_ptr` داخلی خودداری کنیم، مگر اینکه دلیل مشخصی برای این کار داشته باشیم.

عبارت زیر:

```cpp
std::shared_ptr<MyClass>& getValue();
```

جزئیات مدیریت مالکیت داخلی کلاس را در اختیار caller قرار می‌دهد.

عبارت caller می‌تواند کارهایی مانند این انجام دهد:

```cpp
owner.getValue().reset();
```

اکنون caller می‌تواند مالکیت داخلی کلاس را تغییر دهد.

در بسیاری از طراحی‌ها بهتر است به‌جای آن، خود `shared_ptr` را به‌صورت value برگردانیم:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

این کار یک کپی از خود `shared_ptr` ایجاد می‌کند، نه یک کپی از شیء تحت مالکیت آن.

---

# آیا کپی shared_ptr یعنی کپی شدن object؟

این یکی از مهم‌ترین نکات `shared_ptr` است.

عبارت زیر:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = p1;
```

باعث کپی شدن `MyClass` نمی‌شود.

فقط یک `shared_ptr` دیگر ایجاد می‌شود که به همان object اشاره می‌کند.

عبارت تصویری آن:

```text
p1 ──┐
     │
p2 ──┼──> MyClass
     │
     └── Control Block
```

بنابراین `p1` و `p2` هر دو به **همان شیء** اشاره می‌کنند.

---

# چگونه از shared_ptr یک Duplicate بگیریم؟

اگر منظور از duplicate گرفتن این است که یک `shared_ptr` دیگر برای همان شیء ایجاد کنیم، کافی است آن را copy کنیم.

عبارت نمونه:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = p1;
```

اینک `p2` یک مالک جدید برای همان object است.

عبارت شمارنده:

```cpp
std::cout << p1.use_count();
```

اکنون معمولاً مقدار `2` را نشان می‌دهد.

---

# اگر منظور duplicate واقعی از خود object باشد چه؟

عملیات copy کردن `shared_ptr` با clone کردن object کاملاً متفاوت است.

عبارت زیر:

```cpp
auto p2 = p1;
```

در اینجا object جدیدی ایجاد نمی‌کند.

اگر واقعاً object مستقل می‌خواهیم، باید خود object را کپی کنیم، به شرط اینکه نوع مورد نظر قابل کپی باشد.

عبارت نمونه:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = std::make_shared<MyClass>(*p1);
```

اکنون `p1` و `p2` دو object متفاوت دارند.

عبارت تصویری:

```text
p1 ──> MyClass #1

p2 ──> MyClass #2
```

این دو object هیچ مالکیت مشترکی نسبت به یکدیگر ندارند.

---

# دسترسی به مقدار داخلی shared_ptr

اگر منظور از مقدار داخلی، همان object تحت مالکیت `shared_ptr` باشد، می‌توان از `*` استفاده کرد.

عبارت نمونه:

```cpp
(*ptr).doSomething();
```

روش رایج‌تر استفاده از عملگر `->` است:

```cpp
ptr->doSomething();
```

این دو از نظر دسترسی به object معادل هستند.

---

# دسترسی به object با raw pointer

برای دریافت raw pointer غیرمالک می‌توان از `get()` استفاده کرد.

عبارت نمونه:

```cpp
MyClass* raw = ptr.get();
```

شی `raw` مالک object نیست.

اشاره گر `shared_ptr` همچنان مالک object است و مسئول آزاد کردن آن باقی می‌ماند.

عبارت استفاده:

```cpp
if (raw)
{
    raw->doSomething();
}
```

این روش برای تعامل با APIهایی که raw pointer دریافت می‌کنند مفید است.

---

# خطرات raw pointer حاصل از get()

مهم‌ترین خطر این است که lifetime مربوط به raw pointer به مالک اصلی وابسته است.

عبارت خطرناک:

```cpp
auto ptr = std::make_shared<MyClass>();

MyClass* raw = ptr.get();

ptr.reset();

raw->doSomething(); // خطرناک
```

بعد از `reset()` اگر `ptr` آخرین مالک بوده باشد، object حذف شده است.

شی `raw` اکنون یک dangling pointer است.

---

# get() مالکیت را منتقل نمی‌کند

متد `get()` فقط آدرس object را برمی‌گرداند.

عبارت زیر:

```cpp
MyClass* raw = ptr.get();
```

به هیچ وجه به این معنی نیست که `raw` مالک object شده است.

عبارت بنابراین نباید نوشته شود:

```cpp
delete raw;
```

این کار می‌تواند باعث double deletion شود، زیرا `shared_ptr` هنوز object را مالک است و در نهایت تلاش خواهد کرد آن را حذف کند.

---

# یک اشتباه بسیار خطرناک با raw pointer

یکی از خطرناک‌ترین اشتباهات ساختن چند `shared_ptr` مستقل از یک raw pointer مشترک است.

عبارت زیر را نباید انجام داد:

```cpp
MyClass* raw = new MyClass();

std::shared_ptr<MyClass> p1(raw);
std::shared_ptr<MyClass> p2(raw);
```

اشیای `p1` و `p2` دو control block مستقل ایجاد می‌کنند.

عبارت تصویری:

```text
p1 ──> Control Block #1 ──> MyClass

p2 ──> Control Block #2 ──> MyClass
```

هر دو control block فکر می‌کنند مالک object هستند.

در نهایت هر دو ممکن است تلاش کنند object را حذف کنند و نتیجه undefined behavior خواهد بود.

اگر قرار است `shared_ptr`های متعدد ایجاد کنیم، باید آنها را از روی یک `shared_ptr` موجود copy کنیم:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = p1;
```

---

# آیا دسترسی به داخل shared_ptr باعث Memory Leak می‌شود؟

خود `get()` باعث memory leak نمی‌شود.

مشکل اصلی زمانی ایجاد می‌شود که lifetime و ownership را با raw pointer به‌صورت نادرست مدیریت کنیم.

عبارت خطرناک:

```cpp
auto ptr = std::make_shared<MyClass>();

MyClass* raw = ptr.get();

ptr.reset();

// raw نگهداری شده، اما object دیگر وجود ندارد
```

این مثال بیشتر باعث dangling pointer می‌شود تا memory leak.

فاجعه memory leak مهم‌تر در `shared_ptr` معمولاً از **cycle در مالکیت** ایجاد می‌شود.

---

# مشکل اصلی shared_ptr؛ چرخهٔ مالکیت

اشاره گر `shared_ptr` تا زمانی که reference count صفر نشود، object را حذف نمی‌کند.

اگر objectها به‌صورت چرخه‌ای مالک یکدیگر باشند، ممکن است شمارنده هیچ‌وقت به صفر نرسد.

عبارت یک مثال کلاسیک:

```cpp
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

اگر بنویسیم:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();

a->b = b;
b->a = a;
```

اکنون ساختار مالکیت به شکل زیر است:

```text
a ──> A ──shared_ptr──> B
       ▲                │
       │                │
       └──shared_ptr────┘
```

شی `A` مالک `B` است و `B` نیز مالک `A` است.

حتی اگر `a` و `b` از scope خارج شوند، این چرخه ممکن است باعث شود reference count هر object به صفر نرسد.

نتیجه این است که objectها آزاد نمی‌شوند.

---

# چرا شمارندهٔ shared_ptr در این حالت به خوبی کار نمی‌کند؟

نکتهٔ بسیار مهم این است که **reference counting خودش خراب نشده است**.

شمارنده دقیقاً مطابق قوانین مالکیت کار می‌کند.

مشکل این است که مدل مالکیت ما یک چرخه ایجاد کرده است.

عبارت `A` می‌گوید:

> من هنوز `B` را مالک هستم.

عبارت `B` نیز می‌گوید:

> من هنوز `A` را مالک هستم.

بنابراین هیچ‌کدام نمی‌توانند به‌صورت طبیعی به صفر برسند.

این همان مسئله‌ای است که `std::weak_ptr` برای حل بخشی از آن طراحی شده است.

---

# چرا weak_ptr وجود دارد؟

اشاره گر `weak_ptr` به یک object تحت مدیریت `shared_ptr` اشاره می‌کند، اما **مالک آن object نیست**.

عبارت نمونه:

```cpp
std::weak_ptr<MyClass> weak = shared;
```

این کار reference count مربوط به مالکیت shared را افزایش نمی‌دهد.

بنابراین می‌توانیم رابطه‌ای داشته باشیم که فقط object را مشاهده کند، بدون اینکه در lifetime آن مالکیت ایجاد کند.

عبارت در ساختار قبلی می‌توانیم یکی از روابط را به شکل زیر تغییر دهیم:

```cpp
class B
{
public:
    std::weak_ptr<A> a;
};
```

اکنون `B` دیگر مالک `A` نیست.

در نتیجه چرخهٔ مالکیت شکسته می‌شود و objectها می‌توانند در زمان مناسب آزاد شوند.

جزئیات عمیق‌تر `weak_ptr` و control block را می‌توان در آموزش جداگانه‌ای بررسی کرد.

---

# shared_ptr و weak_ptr چه تفاوتی دارند؟

اشاره گر `shared_ptr` مالک است.

عبارت:

```cpp
std::shared_ptr<MyClass>
```

یعنی:

> من در مالکیت object شریک هستم.

عبارت:

```cpp
std::weak_ptr<MyClass>
```

یعنی:

> من فقط object را مشاهده می‌کنم و مالک آن نیستم.

اشاره گر `weak_ptr` برای دسترسی به object معمولاً ابتدا باید به `shared_ptr` تبدیل موقت شود.

عبارت نمونه:

```cpp
if (auto ptr = weak.lock())
{
    ptr->doSomething();
}
```

اگر object هنوز زنده باشد، `lock()` یک `shared_ptr` معتبر ایجاد می‌کند.

اگر object قبلاً از بین رفته باشد، `lock()` یک `shared_ptr` خالی برمی‌گرداند.

---

# shared_ptr در یک کلاس

یکی از کاربردهای رایج `shared_ptr` این است که یک کلاس مالکیت اشتراکی یک object را نگه دارد.

عبارت نمونه:

```cpp
class Car
{
private:
    std::shared_ptr<Engine> engine;

public:
    explicit Car(std::shared_ptr<Engine> engine)
        : engine(std::move(engine))
    {
    }

    std::shared_ptr<Engine> getEngine() const
    {
        return engine;
    }
};
```

اکنون `Car` یکی از مالکان `Engine` است.

عبارت استفاده:

```cpp
auto engine = std::make_shared<Engine>();

Car car(engine);

auto engine2 = car.getEngine();
```

اکنون چند `shared_ptr` به همان `Engine` اشاره می‌کنند و همه در مالکیت آن شریک هستند.

---

# آیا getter بهتر است shared_ptr برگرداند؟

اگر هدف این است که caller بتواند lifetime object را مستقل از کلاس نگه دارد، بله، برگرداندن `shared_ptr` به‌صورت value کاملاً منطقی است.

عبارت:

```cpp
std::shared_ptr<Engine> getEngine() const
{
    return engine;
}
```

 این getter یک کپی سبک از خود smart pointer ایجاد می‌کند و object را کپی نمی‌کند.

 نتیجه یک مالک جدید است که reference count را افزایش می‌دهد.

---

# اگر فقط می‌خواهیم به object دسترسی بدهیم چه کنیم؟

اگر caller فقط برای مدت کوتاهی به object نیاز دارد و نباید مالکیت جدیدی ایجاد کند، شاید اصلاً نباید `shared_ptr` برگردانیم.

عبارت برای object غیرقابل تغییر:

```cpp
const Engine& getEngine() const
{
    return *engine;
}
```

این API به caller اجازهٔ دسترسی می‌دهد، اما مالکیت جدیدی ایجاد نمی‌کند.

اگر object ممکن است وجود نداشته باشد، می‌توان از raw pointer غیرمالک استفاده کرد:

```cpp
const Engine* getEngine() const
{
    return engine.get();
}
```

انتخاب بین این روش‌ها باید بر اساس قرارداد lifetime و ownership انجام شود.

---

# تفاوت مهم getter با unique_ptr

در `unique_ptr` معمولاً نمی‌خواهیم ownership داخلی کلاس را به‌سادگی در اختیار caller قرار دهیم، چون مالکیت باید یکتا باقی بماند.

در `shared_ptr` انتقال یک کپی از smart pointer معمولاً مشکلی ندارد، چون اساساً طراحی آن بر مالکیت مشترک است.

عبارت در نتیجه این طراحی کاملاً طبیعی است:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

شی caller یک مالک جدید دریافت می‌کند و کلاس نیز همچنان مالک باقی می‌ماند.

---

# shared_ptr و انتقال با move

اشاره گر `shared_ptr` مانند `unique_ptr` قابلیت move دارد.

عبارت نمونه:

```cpp
auto p1 = std::make_shared<MyClass>();

auto p2 = std::move(p1);
```

مالکیت `p1` به `p2` منتقل می‌شود.

بعد از move، `p1` خالی است و `p2` مالکیت را در اختیار دارد.

---

# تفاوت copy و move در shared_ptr

دستور copy باعث ایجاد یک مالک جدید می‌شود.

عبارت:

```cpp
auto p2 = p1;
```

متغیر reference count افزایش پیدا می‌کند.

عملیات move مالکیت `shared_ptr` را منتقل می‌کند.

عبارت:

```cpp
auto p2 = std::move(p1);
```

در این حالت معمولاً reference count افزایش پیدا نمی‌کند؛ ownership state خود `shared_ptr` منتقل می‌شود.

بنابراین انتخاب بین copy و move باید بر اساس این باشد که آیا می‌خواهیم مالک جدیدی ایجاد شود یا فقط همان مالکیت را منتقل کنیم.

---

# reset در shared_ptr

متد `reset()` برای رها کردن مالکیت فعلی استفاده می‌شود.

عبارت:

```cpp
ptr.reset();
```

اشاره گر `ptr` دیگر مالک object نیست.

اشاره گر اگر `ptr` آخرین مالک بوده باشد، object نیز حذف می‌شود.

عبارت نمونه:

```cpp
auto p1 = std::make_shared<MyClass>();
auto p2 = p1;

p1.reset();
```

بعد از `reset()`، object هنوز زنده است، زیرا `p2` همچنان مالک آن است.

عبارت اگر بعداً:

```cpp
p2.reset();
```

اشاره گر `p2` نیز آخرین مالک را رها کرده و object آزاد خواهد شد.

---

# reset با object جدید

متد `reset()` می‌تواند `shared_ptr` را به object جدید نیز متصل کند.

عبارت نمونه:

```cpp
ptr.reset(new MyClass());
```

اگر `ptr` قبلاً آخرین مالک object قبلی باشد، object قبلی آزاد می‌شود و `ptr` مالک object جدید خواهد شد.

برای ساخت object جدید همچنان استفاده از `make_shared` ترجیح دارد:

```cpp
ptr = std::make_shared<MyClass>();
```

---

# آیا shared_ptr را می‌توان خالی کرد؟

مفهوم بله.

عبارت:

```cpp
std::shared_ptr<MyClass> ptr;
```

مفهوم یک `shared_ptr` خالی ایجاد می‌کند.

عبارت بررسی:

```cpp
if (!ptr)
{
    // ptr is empty
}
```

می‌توان از `nullptr` نیز استفاده کرد:

```cpp
if (ptr == nullptr)
{
    // Empty
}
```

---

# shared_ptr و چند thread

یکی از نکات مهم این است که باید بین **thread-safe بودن control block** و **thread-safe بودن object** تفاوت بگذاریم.

عملیات مالکیت روی `shared_ptr`ها در شرایط استاندارد مشخصی thread-safe هستند، اما این به معنی thread-safe بودن خود object تحت مالکیت نیست.

عبارت اگر دو thread هم‌زمان این کار را انجام دهند:

```cpp
ptr->value++;
```

اشاره گر `shared_ptr` جلوی data race روی `value` را نمی‌گیرد.

در اینجا smart pointer مسئول مدیریت lifetime و ownership است، نه synchronization داخلی object.

---

# shared_ptr و destructor

یکی از مزایای اصلی `shared_ptr` این است که معمولاً نیازی به `delete` دستی نداریم.

عبارت:

```cpp
class Manager
{
private:
    std::shared_ptr<MyClass> value;
};
```

وقتی `Manager` از بین برود، `shared_ptr` عضو کلاس نیز از بین می‌رود.

اگر آن `shared_ptr` آخرین مالک باشد، object نیز آزاد می‌شود.

---

# یک مثال کامل از طراحی کلاس

می‌توانیم یک کلاس را به شکل زیر طراحی کنیم:

```cpp
class Manager
{
private:
    std::shared_ptr<MyClass> value;

public:
    explicit Manager(std::shared_ptr<MyClass> value)
        : value(std::move(value))
    {
    }

    void setValue(std::shared_ptr<MyClass> newValue)
    {
        value = std::move(newValue);
    }

    std::shared_ptr<MyClass> getValue() const
    {
        return value;
    }

    const MyClass* getRawValue() const
    {
        return value.get();
    }

    void resetValue()
    {
        value.reset();
    }
};
```

این کلاس مالکیت اشتراکی را به‌صورت واضح مدیریت می‌کند.

متد `setValue()` یک مالکیت اشتراکی جدید دریافت می‌کند.

متد `getValue()` یک `shared_ptr` جدید برای مالکیت اشتراکی برمی‌گرداند.

متد `getRawValue()` فقط دسترسی غیرمالکانه می‌دهد.

متد `resetValue()` مالکیت داخلی را رها می‌کند.

---

# یک نکته درباره return کردن shared_ptr

برخی برنامه‌نویسان نگران هستند که return کردن `shared_ptr` باعث کپی سنگین object شود.

این تصور اشتباه است.

عبارت:

```cpp
return value;
```

شی object تحت مالکیت را کپی نمی‌کند.

فقط خود smart pointer و ownership information مربوط به آن مدیریت می‌شود.

در کد مدرن، compiler نیز می‌تواند بسیاری از عملیات move و copy را بهینه کند.

---

# shared_ptr چه زمانی انتخاب مناسبی نیست؟

نباید صرفاً به این دلیل که `shared_ptr` مدیریت حافظه را آسان می‌کند، آن را همه‌جا استفاده کنیم.

اشاره گر `shared_ptr` هزینه و پیچیدگی بیشتری نسبت به `unique_ptr` دارد.

عبارت اگر فقط یک مالک داریم:

```cpp
std::unique_ptr<MyClass>
```

معمولاً انتخاب بهتری است.

عبارت اگر چند بخش واقعاً باید مالک object باشند:

```cpp
std::shared_ptr<MyClass>
```

می‌تواند انتخاب مناسبی باشد.

عبارت اگر فقط نیاز به دسترسی داریم:

```cpp
MyClass&
```

یا:

```cpp
MyClass*
```

ممکن است انتخاب ساده‌تر و دقیق‌تری باشد.

---

# تفاوت shared_ptr و unique_ptr

مهم‌ترین تفاوت این دو، **مدل مالکیت** است.

| ویژگی                      | `unique_ptr` | `shared_ptr`   |
| -------------------------- | ------------ | -------------- |
| نوع مالکیت                 | یکتا         | اشتراکی        |
| Copy                       | خیر          | بله            |
| Move                       | بله          | بله            |
| Reference Count            | ندارد        | دارد           |
| مالکیت چندگانه             | خیر          | بله            |
| هزینه مدیریتی              | کمتر         | بیشتر          |
| خطر چرخه مالکیت            | ندارد        | دارد           |
| نیاز احتمالی به `weak_ptr` | معمولاً خیر  | بله            |
| کاربرد اصلی                | یک مالک مشخص | چند مالک مستقل |

اگر بتوانیم مالکیت را به یک موجودیت مشخص محدود کنیم، `unique_ptr` معمولاً انتخاب بهتری است.

اگر چند موجودیت باید بتوانند lifetime object را مستقل از یکدیگر حفظ کنند، `shared_ptr` مناسب‌تر است.

---

# یک مقایسه ساده

اشاره گر `unique_ptr` را می‌توان مانند یک کلید واحد تصور کرد.

عبارت فقط یک نفر کلید را در اختیار دارد.

اشاره گر `shared_ptr` مانند سیستمی است که چند نفر نسخه‌ای از حق مالکیت یک منبع را دارند.

عبارت تا زمانی که حداقل یک نفر هنوز مالک است، منبع باقی می‌ماند.

اشاره گر `weak_ptr` مانند کسی است که فقط آدرس منبع را می‌داند، اما مالک آن نیست.

---

# چه زمانی shared_ptr انتخاب درستی است؟

استفاده از `shared_ptr` زمانی منطقی است که چند بخش مستقل برنامه باید بتوانند object را نگه دارند و lifetime آن را حفظ کنند.

عبارت نمونه‌های رایج عبارت‌اند از:

```text
مدل‌های اشتراکی بین چند component
Objectهایی که چند subsystem از آن‌ها استفاده می‌کنند
Factoryهایی که مالکیت نتیجه را به caller منتقل می‌کنند
Graphها و ساختارهای پیچیده مالکیتی
Cacheها در شرایط مناسب
مدیریت منابعی که چند مصرف‌کننده مستقل دارند
```

البته هر کدام از این موارد باید جداگانه از نظر ownership بررسی شوند و صرفاً وجود چند استفاده‌کننده به معنی نیاز قطعی به `shared_ptr` نیست.

---

# چه زمانی unique_ptr بهتر است؟

اگر مالکیت مشخص و یکتا است، بهتر است تا حد امکان از `unique_ptr` استفاده کنیم.

عبارت نمونه:

```cpp
class Car
{
private:
    std::unique_ptr<Engine> engine;
};
```

مفهوم این طراحی بسیار واضح است:

> `Car` مالک `Engine` است.

اگر `Engine` قرار نیست توسط بخش دیگری مالکیت شود، تبدیل آن به `shared_ptr` فقط پیچیدگی اضافه ایجاد می‌کند.

---

# یک قانون طلایی برای انتخاب smart pointer

می‌توانیم یک قانون ساده داشته باشیم:

```text
مالکیت یکتا:
unique_ptr

مالکیت اشتراکی:
shared_ptr

دسترسی بدون مالکیت و غیر nullable:
reference

دسترسی بدون مالکیت و nullable:
raw pointer

دسترسی بدون مالکیت به object تحت مدیریت shared_ptr،
با نیاز به بررسی زنده بودن object:
weak_ptr
```

این قانون مطلق نیست، اما نقطه شروع بسیار خوبی برای طراحی API است.

---

# نکات مهم درباره shared_ptr

اشاره گر `shared_ptr` مالکیت را با reference counting مدیریت می‌کند.

عملیات copy کردن `shared_ptr` باعث کپی شدن object نمی‌شود و فقط مالکیت اشتراکی ایجاد می‌کند.

استفاده از `std::make_shared<T>()` روش پیشنهادی برای ساخت `shared_ptr` در C++11 است.

متد `get()` فقط raw pointer غیرمالک را برمی‌گرداند.

متد `reset()` مالکیت `shared_ptr` را رها می‌کند و اگر آخرین مالک باشد، object نیز آزاد می‌شود.

متد `release()` در `shared_ptr` وجود ندارد؛ این تابع مربوط به `unique_ptr` است.

اشاره گر `shared_ptr` قابل copy و move است.

اشاره گر `unique_ptr` قابل move است اما قابل copy نیست.

اشاره گر `shared_ptr` می‌تواند در صورت طراحی نادرست، چرخهٔ مالکیت ایجاد کند.

اشاره گر `weak_ptr` برای ایجاد رابطهٔ غیرمالکانه و شکستن چنین چرخه‌هایی بسیار مهم است.

اشاره گر raw pointer حاصل از `get()` نباید با `delete` آزاد شود.

نباید از یک raw pointer واحد چند `shared_ptr` مستقل بسازیم.

---

# جمع‌بندی نهایی

مفهوم اصلی `std::shared_ptr` این است:

> **چند مالک می‌توانند یک object را به‌صورت مشترک مالک باشند و object تا زمانی که آخرین مالک باقی مانده است زنده می‌ماند.**

این رفتار با استفاده از reference counting پیاده‌سازی می‌شود.

عبارت ایجاد یک `shared_ptr`:

```cpp
auto ptr = std::make_shared<MyClass>();
```

ایجاد یک مالک جدید:

```cpp
auto other = ptr;
```

انتقال state یک `shared_ptr`:

```cpp
auto other = std::move(ptr);
```

دسترسی به object:

```cpp
ptr->doSomething();
```

دریافت raw pointer غیرمالک:

```cpp
MyClass* raw = ptr.get();
```

رها کردن مالکیت:

```cpp
ptr.reset();
```

مشاهده تعداد مالکان:

```cpp
ptr.use_count();
```

دریافت مالکیت اشتراکی از getter:

```cpp
std::shared_ptr<MyClass> getValue() const
{
    return value;
}
```

دریافت صرفاً دسترسی غیرمالکانه:

```cpp
MyClass* getValue()
{
    return value.get();
}
```

نکتهٔ بسیار مهم این است که `shared_ptr` برای حل مشکل **مالکیت مشترک** ایجاد شده است، نه صرفاً برای اینکه نوشتن `delete` را حذف کنیم.

اگر مالکیت فقط یک نفر دارد، `unique_ptr` معمولاً انتخاب بهتر و ساده‌تری است.

اگر چند بخش مستقل برنامه باید بتوانند lifetime یک object را حفظ کنند، `shared_ptr` می‌تواند انتخاب مناسبی باشد.

اگر فقط می‌خواهیم به object دسترسی داشته باشیم و مالک آن نیستیم، بهتر است اصلاً ownership را با `shared_ptr` منتقل نکنیم.

و در نهایت اگر مالکیت‌های `shared_ptr` به یک چرخه تبدیل شوند، reference counting به صفر نمی‌رسد و اینجا است که `weak_ptr` وارد طراحی می‌شود تا رابطه‌ای **غیرمالکانه** ایجاد کند.

بنابراین مهم‌ترین سؤال هنگام استفاده از `shared_ptr` این نیست که «آیا می‌توانم اینجا از آن استفاده کنم؟»، بلکه این است:

> **آیا واقعاً چند مالک مستقل برای این object وجود دارند؟**

عبارت اگر پاسخ این سؤال مثبت باشد، `shared_ptr` می‌تواند ابزار مناسبی برای بیان این مدل مالکیت باشد؛ در غیر این صورت، اغلب `unique_ptr` یا حتی یک reference/raw pointer غیرمالک طراحی ساده‌تر و دقیق‌تری ارائه می‌دهد.

---
## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>