<div align="center">

[🇺🇸 English](../../en/cpp14/make_shared.md) | [🇮🇷 فارسی](./make_shared.md)

</div>

---

# راهنمای جامع `std::make_shared` در ++C

## معرفی `std::make_shared`

تابع `std::make_shared` یکی از ابزارهای مهم Standard Library در ++C برای ساخت Objectهایی است که قرار است تحت مدیریت `std::shared_ptr` قرار بگیرند.

این تابع در Header زیر قرار دارد:

```cpp
#include <memory>
```

شکل معمول استفاده از آن به صورت زیر است:

```cpp
auto user = std::make_shared<User>("Ali", 30);
```

در این مثال، تابع `()std::make_shared` یک Object از نوع `User` می‌سازد و یک `std::shared_ptr<User>` برمی‌گرداند.

نکته مهم این است که `std::make_shared` خودش یک Smart Pointer نیست؛ بلکه یک Function Template است که برای ساخت یک `std::shared_ptr` استفاده می‌شود.

به زبان ساده، می‌توان رابطه را این‌گونه در نظر گرفت:

```text
std::make_shared<T>(...)
        │
        ▼
std::shared_ptr<T>
        │
        ▼
      Object
```

قابلیت اصلی `std::make_shared` از ++C11 در دسترس است و قابلیت‌های مربوط به Array و `std::make_shared_for_overwrite` در ++C20 اضافه شده‌اند. همچنین در ++C26 توابع مرتبط با `make_shared` به صورت `constexpr` نیز مشخص شده‌اند. ([Cppreference][1])

---

## مفهوم Shared Ownership

برای فهم درست `std::make_shared` ابتدا باید مفهوم `shared ownership` را بشناسیم.

فرض کنید یک Object داریم که چند بخش مختلف برنامه باید مالک آن باشند:

```cpp
auto user = std::make_shared<User>("Ali", 30);

auto service = user;
auto repository = user;
```

اکنون سه `std::shared_ptr` وجود دارند که یک Object را مدیریت می‌کنند:

```text
user ───────────────┐
                    │
service ────────────┼──────► User Object
                    │
repository ─────────┘
```

در چنین حالتی Object تا زمانی زنده می‌ماند که حداقل یک `shared_ptr` مالک آن باشد.

اگر یکی از Pointerها از بین برود:

```cpp
service.reset();
```

Object هنوز زنده است، زیرا Pointerهای دیگری مالک آن هستند.

اگر آخرین `shared_ptr` نیز از بین برود:

```cpp
user.reset();
repository.reset();
```

Object نیز Destroy می‌شود.

این مدل مالکیت همان چیزی است که `shared_ptr` برای آن طراحی شده است.

---

## تفاوت مالکیت با دسترسی

یکی از مهم‌ترین نکات در استفاده از `shared_ptr` این است که دسترسی به یک Object با مالکیت آن Object یکسان نیست.

فرض کنید:

```cpp
auto user = std::make_shared<User>();
```

اگر تابعی فقط برای مدت کوتاهی به User نیاز داشته باشد، الزاماً نباید `shared_ptr` دریافت کند.

مثلاً اگر تابع فقط Object را استفاده می‌کند و مالک آن نیست، ممکن است این طراحی مناسب‌تر باشد:

```cpp
void printUser(const User& user);
```

یا در صورت نیاز به Nullable بودن:

```cpp
void printUser(const User* user);
```

در مقابل، اگر تابع واقعاً باید مالکیت Object را Share کند، استفاده از:

```cpp
std::shared_ptr<User>
```

معنادار است.

بنابراین `shared_ptr` را نباید صرفاً به عنوان یک Pointer امن‌تر از Raw Pointer در نظر گرفت؛ مفهوم اصلی آن **Shared Ownership** است.

---

# تفاوت `new` با `std::make_shared`

برای درک مزایای `make_shared` ابتدا روش سنتی را بررسی کنیم.

## ساخت Object با `new`

روش مستقیم به صورت زیر است:

```cpp
User* user = new User("Ali", 30);
```

در اینجا یک Raw Pointer دریافت می‌کنیم.

در نتیجه مسئولیت Lifetime نیز بر عهده برنامه‌نویس است:

```cpp
delete user;
```

اگر `delete` فراموش شود، Memory Leak ایجاد می‌شود.

---

## ساخت Object با `shared_ptr`

برای مدیریت خودکار Lifetime می‌توان نوشت:

```cpp
std::shared_ptr<User> user(
    new User("Ali", 30)
);
```

در این حالت `shared_ptr` مالک Object می‌شود و زمانی که آخرین Owner از بین برود، Object را Destroy می‌کند.

اما این روش یک نکته مهم دارد: معمولاً ساخت Object و ساخت Control Block در دو Allocation جداگانه انجام می‌شود. در مقابل، `make_shared` معمولاً این دو را در یک Allocation قرار می‌دهد. ([Cppreference][1])

---

## ساخت Object با `make_shared`

روش مدرن‌تر به شکل زیر است:

```cpp
auto user = std::make_shared<User>("Ali", 30);
```

در اینجا:

* یک `User` ساخته می‌شود.
* یک `shared_ptr<User>` ایجاد می‌شود.
* Ownership از ابتدا تحت مدیریت `shared_ptr` قرار می‌گیرد.
* در پیاده‌سازی‌های معمول، Object و Control Block با یک Allocation تأمین می‌شوند.

این روش معمولاً ساده‌تر، خواناتر و از نظر Allocation بهینه‌تر از `shared_ptr(new T(...))` است. ([Cppreference][1])

---

# ساختار داخلی `shared_ptr`

برای فهم `make_shared` باید مفهوم `Control Block` را بشناسیم.

یک `shared_ptr` را می‌توان به صورت مفهومی دارای دو بخش در نظر گرفت:

```text
shared_ptr
├── pointer to managed object
└── pointer to control block
```

Control Block معمولاً اطلاعات مرتبط با Ownership را نگهداری می‌کند.

ساختار دقیق Control Block یک جزئیات Implementation-defined است، اما معمولاً شامل اطلاعاتی مانند Reference Count، Weak Count، Deleter و اطلاعات لازم برای مدیریت Lifetime است.

از نظر مفهومی می‌توان ساختار را این‌گونه تصور کرد:

```text
┌─────────────────────────────┐
│       Control Block         │
│                             │
│   shared ownership count    │
│   weak ownership count      │
│   deleter / allocator       │
│   implementation data       │
└──────────────┬──────────────┘
               │
               ▼
        ┌──────────────┐
        │     User     │
        └──────────────┘
```

جزئیات دقیق Layout این ساختار توسط Standard مشخص نشده و نباید به Layout یک Implementation خاص وابسته شد.

---

# مزیت Allocation در `make_shared`

یکی از مهم‌ترین مزایای `make_shared` مربوط به نحوه تخصیص حافظه است.

در حالت معمول و رایج:

```cpp
std::shared_ptr<User>(
    new User("Ali", 30)
);
```

معمولاً دو Allocation داریم:

```text
Allocation #1
┌─────────────────┐
│     User        │
└─────────────────┘

Allocation #2
┌─────────────────┐
│  Control Block  │
└─────────────────┘
```

اما:

```cpp
auto user = std::make_shared<User>("Ali", 30);
```

در پیاده‌سازی‌های شناخته‌شده معمولاً با یک Allocation انجام می‌شود:

```text
Allocation #1
┌───────────────────────────────┐
│       Control Block           │
│                               │
│       User Object             │
└───────────────────────────────┘
```

استاندارد این روش را الزام مطلق نکرده است، اما `make_shared` معمولاً به این صورت پیاده‌سازی می‌شود و تمام Implementationهای شناخته‌شده این مدل را به کار می‌گیرند. ([Cppreference][1])

---

# مزیت Performance در `make_shared`

کاهش تعداد Allocationها می‌تواند مزایای عملی داشته باشد.

هر Allocation می‌تواند شامل هزینه‌هایی مانند:

* مدیریت Allocator
* درخواست حافظه از Runtime
* Metadata مربوط به Allocation
* Fragmentation
* هزینه‌های Synchronization در برخی Allocatorها

باشد.

بنابراین کاهش Allocationها می‌تواند در برنامه‌هایی که تعداد زیادی Object کوچک ایجاد می‌کنند مفید باشد.

با این حال، عبارت زیر دقیق نیست:

> `make_shared` همیشه سریع‌تر است.

عبارت دقیق‌تر این است:

> `make_shared` معمولاً امکان کاهش تعداد Allocationها را فراهم می‌کند و در نتیجه می‌تواند هزینه Allocation و Locality حافظه را بهبود دهد.

Performance نهایی همچنان به نوع Object، اندازه Object، Allocator، الگوی Lifetime و نحوه استفاده از `weak_ptr` وابسته است.

---

# مفهوم Locality حافظه

قرار گرفتن Object و Control Block در یک Allocation می‌تواند Locality حافظه بهتری ایجاد کند.

در حالت جداگانه:

```text
Memory:

[User Object]                     [Control Block]
     ↑                                  ↑
     └──────────── فاصله ───────────────┘
```

در حالت معمول `make_shared`:

```text
Memory:

[Control Block | User Object]
```

در نتیجه داده‌های مرتبط با مدیریت Object معمولاً در یک محدوده حافظه قرار می‌گیرند.

این موضوع می‌تواند برای Cache و Memory Access Pattern مفید باشد، اما میزان سود واقعی کاملاً به Application و Implementation وابسته است.

---

# مزیت Exception Safety

یکی دیگر از دلایل مهم استفاده از `make_shared`، ساده‌تر شدن مدیریت Exception است.

به این کد توجه کنید:

```cpp
process(
    std::shared_ptr<User>(new User()),
    createSomething()
);
```

در استانداردهای قدیمی‌تر، ترتیب ارزیابی Argumentهای Function می‌توانست در چنین الگویی باعث ایجاد یک پنجره برای Memory Leak شود.

برای نمونه، اگر `new User()` با موفقیت انجام شود اما قبل از ساخته شدن `shared_ptr`، عبارت دیگری Exception ایجاد کند، Object ساخته‌شده ممکن بود بدون مالک باقی بماند.

در مقابل:

```cpp
process(
    std::make_shared<User>(),
    createSomething()
);
```

Object مستقیماً تحت مدیریت `shared_ptr` ایجاد می‌شود.

این تفاوت به‌خصوص در استانداردهای قبل از ++C17 اهمیت داشت. طبق مستندات Standard Library، سناریوی معروف `f(std::shared_ptr<int>(new int(42)), g())` در حالت مربوط به استانداردهای قدیمی می‌توانست منجر به Leak شود، در حالی که استفاده از `make_shared` این مشکل را برطرف می‌کرد. ([Cppreference][1])

---

# نحوه ارسال Argument به Constructor

یکی از ویژگی‌های مهم `make_shared` این است که Argumentهای آن برای ساخت Object استفاده می‌شوند.

فرض کنید:

```cpp
class User
{
public:
    User(std::string name, int age)
        : name_(std::move(name)),
          age_(age)
    {
    }

private:
    std::string name_;
    int age_;
};
```

اکنون می‌توان نوشت:

```cpp
auto user = std::make_shared<User>("Ali", 30);
```

Object به شکلی ساخته می‌شود که در اصل معادل ساخت `User` با همان Argumentها است.

به صورت مفهومی:

```cpp
User("Ali", 30)
```

در نتیجه `make_shared` برای Constructorهای مختلف قابل استفاده است:

```cpp
auto a = std::make_shared<User>();

auto b = std::make_shared<User>("Ali");

auto c = std::make_shared<User>("Ali", 30);
```

البته هر کدام فقط در صورتی معتبر هستند که Constructor متناظر در کلاس وجود داشته باشد.

---

# استفاده از `auto`

یکی از رایج‌ترین الگوهای Modern ++C استفاده از `auto` است:

```cpp
auto user = std::make_shared<User>("Ali", 30);
```

نوع `user` در این مثال:

```cpp
std::shared_ptr<User>
```

است.

بنابراین نوشتن صریح نوع نیز امکان‌پذیر است:

```cpp
std::shared_ptr<User> user =
    std::make_shared<User>("Ali", 30);
```

اما استفاده از `auto` معمولاً کد را کوتاه‌تر و خواناتر می‌کند.

---

# استفاده از `make_shared` برای Polymorphism

یکی از کاربردهای مهم `shared_ptr` مدیریت Objectهای Polymorphic است.

فرض کنید:

```cpp
class Animal
{
public:
    virtual ~Animal() = default;
    virtual void speak() = 0;
};
```

و کلاس مشتق‌شده:

```cpp
class Dog : public Animal
{
public:
    void speak() override
    {
    }
};
```

می‌توان نوشت:

```cpp
std::shared_ptr<Animal> animal =
    std::make_shared<Dog>();
```

در اینجا Pointer از نوع:

```text
shared_ptr<Animal>
```

است، اما Object واقعی:

```text
Dog
```

است.

بنابراین:

```cpp
animal->speak();
```

از طریق Virtual Dispatch به Implementation مربوط به `Dog` می‌رسد.

وجود Virtual Destructor در Base Class برای چنین مدل‌هایی اهمیت زیادی دارد:

```cpp
virtual ~Animal() = default;
```

---

# استفاده از `enable_shared_from_this`

یکی از مکانیزم‌های مرتبط و مهم `shared_ptr`، کلاس:

```cpp
std::enable_shared_from_this
```

است.

فرض کنید Object می‌خواهد یک `shared_ptr` به خودش برگرداند.

الگوی صحیح می‌تواند این باشد:

```cpp
class User
    : public std::enable_shared_from_this<User>
{
public:
    std::shared_ptr<User> getShared()
    {
        return shared_from_this();
    }
};
```

اکنون Object را با `make_shared` می‌سازیم:

```cpp
auto user = std::make_shared<User>();

auto another = user->getShared();
```

در این حالت `another` و `user` به همان Ownership مربوط هستند.

نکته مهم این است که نباید برای Objectی که قبلاً توسط `shared_ptr` مدیریت می‌شود، به صورت دستی چنین کاری انجام داد:

```cpp
std::shared_ptr<User>(this);
```

زیرا این کار می‌تواند Control Block مستقل ایجاد کند و در نهایت به مدیریت نادرست Lifetime و حتی Double Delete منجر شود.

مکانیزم `enable_shared_from_this` برای جلوگیری از چنین طراحی‌هایی ارائه شده است. `make_shared` نیز با این مکانیزم سازگار است. ([Cppreference][1])

---

# بررسی تعداد Ownerها با `use_count`

برای مشاهده تعداد `shared_ptr`هایی که در Shared Ownership شرکت دارند، می‌توان از `use_count` استفاده کرد.

مثلاً:

```cpp
auto p1 = std::make_shared<int>(42);

std::cout << p1.use_count() << '\n';

auto p2 = p1;

std::cout << p1.use_count() << '\n';
```

خروجی معمول:

```text
1
2
```

بعد از:

```cpp
p2.reset();
```

تعداد Ownerها دوباره به یک می‌رسد.

نکته طراحی مهم این است که `use_count()` معمولاً نباید مبنای منطق اصلی برنامه قرار بگیرد؛ زیرا مقدار آن در برنامه‌های Concurrent می‌تواند بین دو مشاهده تغییر کند.

---

# دسترسی به Object با `get`

تابع `get` Raw Pointer مربوط به Object را برمی‌گرداند:

```cpp
auto user = std::make_shared<User>();

User* raw = user.get();
```

در اینجا `raw` مالک Object نیست.

بنابراین نباید بنویسیم:

```cpp
delete raw;
```

زیرا Lifetime همچنان توسط `shared_ptr` مدیریت می‌شود.

تابع `get` بیشتر زمانی مفید است که یک API قدیمی فقط Raw Pointer دریافت می‌کند:

```cpp
void legacy_api(User* user);
```

در این حالت می‌توان نوشت:

```cpp
legacy_api(user.get());
```

اما باید مطمئن باشیم API موردنظر Pointer را ذخیره نمی‌کند و بعداً از آن استفاده نمی‌کند.

مفهوم `get()` صرفاً دسترسی به Stored Pointer است و Ownership را منتقل نمی‌کند. ([Cppreference][2])

---

# مدیریت Lifetime با `reset`

با `reset` می‌توان یک `shared_ptr` را از Object جدا کرد:

```cpp
auto user = std::make_shared<User>();

user.reset();
```

اگر `user` آخرین Owner باشد، Object Destroy می‌شود.

اما اگر Pointer دیگری وجود داشته باشد:

```cpp
auto user = std::make_shared<User>();

auto another = user;

user.reset();
```

Object همچنان زنده است، زیرا `another` هنوز مالک آن است.

---

# انتقال و کپی `shared_ptr`

کپی کردن `shared_ptr` یعنی اضافه شدن یک Owner دیگر:

```cpp
auto p1 = std::make_shared<User>();

auto p2 = p1;
```

در اینجا:

```text
p1 ───────┐
          ├────► User
p2 ───────┘
```

اما `move` مالکیت موجود را به Pointer دیگری منتقل می‌کند:

```cpp
auto p1 = std::make_shared<User>();

auto p2 = std::move(p1);
```

بعد از این عملیات، `p2` مالک Object است و `p1` در حالت moved-from قرار دارد و معمولاً خالی خواهد بود.

این تفاوت را می‌توان این‌گونه خلاصه کرد:

```text
copy
 └── shared ownership

move
 └── transfer of ownership state
```

---

# جلوگیری از Circular Ownership

یکی از مهم‌ترین مشکلات `shared_ptr` ایجاد Reference Cycle است.

فرض کنید:

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

سپس:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();

a->b = b;
b->a = a;
```

ساختار به شکل زیر خواهد شد:

```text
A ───── shared_ptr ─────► B
▲                         │
└──────── shared_ptr ─────┘
```

حتی اگر Ownerهای خارجی `a` و `b` از بین بروند، هر Object هنوز توسط Object دیگر مالکیت می‌شود.

در نتیجه Reference Count هیچ‌وقت به صفر نمی‌رسد.

---

# استفاده از `weak_ptr` برای حل Cycle

برای رابطه‌ای که مالکیت ایجاد نمی‌کند، باید از `weak_ptr` استفاده شود.

مثلاً:

```cpp
class B
{
public:
    std::weak_ptr<A> a;
};
```

اکنون ساختار به این شکل است:

```text
A ───── shared_ptr ─────► B
▲                         │
└───────── weak_ptr ──────┘
```

`weak_ptr` مالک Object نیست و Shared Ownership Count را افزایش نمی‌دهد.

برای دسترسی به Object نیز باید ابتدا `lock()` انجام شود:

```cpp
if (auto owner = b->a.lock())
{
    owner->doSomething();
}
```

این الگو برای Observer Relationship، Parent/Child Relationship و جلوگیری از Ownership Cycle بسیار مهم است.

---

# ارتباط `make_shared` با `weak_ptr`

استفاده از `make_shared` یک Trade-off مهم در ارتباط با `weak_ptr` دارد.

فرض کنید:

```cpp
auto object = std::make_shared<LargeObject>();

std::weak_ptr<LargeObject> weak = object;

object.reset();
```

بعد از `reset` شدن آخرین `shared_ptr`، Lifetime خود `LargeObject` پایان پیدا می‌کند.

اما Control Block هنوز ممکن است توسط `weak_ptr` مورد نیاز باشد.

در `make_shared` معمولاً Object و Control Block در یک Allocation قرار دارند:

```text
┌────────────────────────────┐
│ Control Block              │
│                            │
│ LargeObject                │
└────────────────────────────┘
```

بنابراین اگر Object بسیار بزرگ باشد، فضای Allocation مشترک ممکن است تا زمانی که آخرین `weak_ptr` از بین نرفته است، کاملاً آزاد نشود. این یکی از مهم‌ترین Trade-offهای `make_shared` است. ([Cppreference][1])

در مقابل، با:

```cpp
std::shared_ptr<LargeObject>(
    new LargeObject()
);
```

معمولاً Object و Control Block Allocationهای جداگانه دارند.

در نتیجه بعد از Destroy شدن Object، فضای Object می‌تواند مستقل از Control Block آزاد شود.

---

# محدودیت Custom Deleter

یکی از تفاوت‌های مهم `make_shared` با Constructorهای `shared_ptr` این است که `make_shared` امکان تعیین Custom Deleter را ندارد.

برای مثال این کد امکان تعریف Deleter اختصاصی را فراهم می‌کند:

```cpp
auto ptr = std::shared_ptr<Resource>(
    resource,
    [](Resource* resource)
    {
        release_resource(resource);
    }
);
```

اما چنین چیزی با `make_shared` وجود ندارد:

```cpp
// There is no custom deleter parameter here.
auto ptr = std::make_shared<Resource>();
```

بنابراین اگر Resource نیاز به نحوه خاصی برای Release شدن داشته باشد، Constructor مربوط به `shared_ptr` می‌تواند گزینه مناسب‌تری باشد. ([Cppreference][1])

---

# محدودیت Constructor خصوصی

فرض کنید Constructor کلاس `private` باشد:

```cpp
class SingletonLike
{
private:
    SingletonLike() = default;

public:
    static std::shared_ptr<SingletonLike> create();
};
```

ممکن است انتظار داشته باشیم این Factory داخلی بتواند به راحتی از `make_shared` استفاده کند:

```cpp
std::shared_ptr<SingletonLike>
SingletonLike::create()
{
    return std::make_shared<SingletonLike>();
}
```

اما دسترسی Constructor در Context داخلی `make_shared` انجام می‌شود و نه در Context تابع `create`.

در نتیجه `make_shared` نمی‌تواند صرفاً به دلیل اینکه Factory به Constructor دسترسی دارد، از آن Constructor خصوصی استفاده کند.

در مقابل، ساخت مستقیم با `new` در Contextی که Constructor قابل دسترسی است می‌تواند متفاوت عمل کند:

```cpp
std::shared_ptr<SingletonLike>(
    new SingletonLike()
);
```

این یکی از تفاوت‌های رسمی بین دو روش است. ([Cppreference][1])

---

# تفاوت با Class-Specific `operator new`

نکته حرفه‌ای دیگری که باید در نظر گرفته شود، رفتار `operator new` اختصاصی کلاس است.

فرض کنید کلاس چنین Allocation خاصی داشته باشد:

```cpp
class MyType
{
public:
    static void* operator new(std::size_t size);

    static void operator delete(void* ptr) noexcept;
};
```

در ساخت مستقیم:

```cpp
auto ptr = std::shared_ptr<MyType>(
    new MyType()
);
```

عبارت `new MyType()` می‌تواند از Class-Specific `operator new` استفاده کند.

اما `make_shared` از مکانیزم Allocation داخلی خودش استفاده می‌کند و استاندارد آن را معادل استفاده از Class-Specific `operator new` قرار نمی‌دهد.

مستندات Standard Library نیز صراحتاً اشاره می‌کنند که `make_shared` از `::new` استفاده می‌کند و رفتار آن در این زمینه می‌تواند با `shared_ptr(new T(...))` متفاوت باشد. ([Cppreference][1])

بنابراین در سیستم‌هایی با Memory Allocation اختصاصی، باید این تفاوت را جدی گرفت.

---

# پشتیبانی از Array در `make_shared`

در نسخه‌های قدیمی ++C، `make_shared` برای Arrayها پشتیبانی استاندارد نداشت.

از ++C20، Overloadهای مربوط به Array به `make_shared` اضافه شدند. ([Cppreference][1])

برای یک Array با اندازه Dynamic می‌توان نوشت:

```cpp
auto values = std::make_shared<int[]>(100);
```

در این حالت یک Array شامل 100 عنصر ایجاد می‌شود.

برای Array با اندازه ثابت نیز می‌توان نوشت:

```cpp
auto values = std::make_shared<int[10]>();
```

همچنین امکان مقداردهی اولیه Array با مقدار مشخص وجود دارد:

```cpp
auto values = std::make_shared<int[]>(100, 42);
```

در این حالت عناصر Array با مقدار مشخص‌شده مقداردهی می‌شوند. Overloadهای Array مربوط به `make_shared` از ++C20 در دسترس هستند. ([Cppreference][1])

---

# تفاوت Arrayهای Bounded و Unbounded

در Syntax زیر:

```cpp
std::make_shared<int[]>(100);
```

نوع:

```text
int[]
```

یک Unbounded Array Type است و اندازه در زمان Runtime مشخص می‌شود.

در مقابل:

```cpp
std::make_shared<int[10]>();
```

نوع:

```text
int[10]
```

یک Bounded Array Type است.

این دو حالت Overloadهای متفاوتی از `make_shared` را فعال می‌کنند.

---

# استفاده از `make_shared_for_overwrite`

از ++C20 تابع دیگری به نام `make_shared_for_overwrite` اضافه شد.

این تابع زمانی کاربرد دارد که می‌خواهیم Storage را ایجاد کنیم و سپس خودمان مقدار مناسب را در آن بنویسیم.

برای یک Object معمولی:

```cpp
auto object =
    std::make_shared_for_overwrite<MyType>();
```

برای Array:

```cpp
auto buffer =
    std::make_shared_for_overwrite<std::byte[]>(1024);
```

تفاوت اصلی این است که این API برای Default Initialization طراحی شده است و مقدار اولیه مشخصی برای Object یا عناصر Array تضمین نمی‌کند. ([Cppreference][1])

بنابراین نباید قبل از مقداردهی مناسب، داده را از چنین Objectی بخوانیم.

---

# کاربرد مناسب `make_shared_for_overwrite`

یک سناریوی مناسب برای این API Bufferهایی است که قرار است بلافاصله توسط یک API خارجی پر شوند.

برای مثال:

```cpp
auto buffer =
    std::make_shared_for_overwrite<std::byte[]>(4096);

read_data(buffer.get(), 4096);
```

در چنین سناریویی اگر `read_data` تمام Buffer را قبل از خواندن پر کند، انجام Initialization اضافی ممکن است غیرضروری باشد.

با این حال، استفاده از `make_shared_for_overwrite` باید آگاهانه باشد؛ زیرا خواندن Object یا عناصر آن قبل از مقداردهی معتبر می‌تواند مشکل‌ساز باشد.

قابلیت `make_shared_for_overwrite` با Feature-Test Macro زیر شناخته می‌شود:

```cpp
__cpp_lib_smart_ptr_for_overwrite
```

و مقدار استاندارد آن برای ++C20 برابر `202002L` است. ([Cppreference][3])

---

# استفاده از `allocate_shared`

اگر کنترل بیشتری روی Allocation لازم باشد، Standard Library تابع `allocate_shared` را ارائه می‌کند.

شکل ساده استفاده:

```cpp
auto object =
    std::allocate_shared<MyType>(allocator);
```

تفاوت اصلی این است که در اینجا Allocator موردنظر را به `allocate_shared` می‌دهیم.

به صورت مفهومی:

```text
make_shared
    │
    └── Standard/default allocation strategy

allocate_shared
    │
    └── User-provided allocator
```

`allocate_shared` نیز مانند `make_shared` معمولاً Object و Control Block را در یک Allocation قرار می‌دهد. ([Cppreference][4])

---

# کاربرد `allocate_shared` در Memory Pool

این قابلیت برای سیستم‌هایی که Memory Resource یا Memory Pool دارند بسیار مفید است.

برای مثال در کنار `std::pmr` می‌توان Allocator مناسب ساخت و سپس Objectها را با `allocate_shared` ایجاد کرد.

نمونه ساده:

```cpp
#include <memory>
#include <memory_resource>

class Node
{
public:
    explicit Node(int value)
        : value_(value)
    {
    }

private:
    int value_;
};

int main()
{
    std::byte buffer[4096];

    std::pmr::monotonic_buffer_resource resource(
        buffer,
        sizeof(buffer)
    );

    std::pmr::polymorphic_allocator<Node> allocator(
        &resource
    );

    auto node =
        std::allocate_shared<Node>(
            allocator,
            42
        );
}
```

در چنین سناریویی Allocator علاوه بر ساخت Object، در مدیریت Allocation مربوط به Control Block نیز نقش دارد. ([Cppreference][4])

---

# تفاوت `make_shared` و `allocate_shared`

می‌توان تفاوت این دو را به شکل زیر خلاصه کرد:

| ویژگی                  | `make_shared` | `allocate_shared` |
| ---------------------- | ------------- | ----------------- |
| ساخت `shared_ptr`      | دارد          | دارد              |
| ساخت Object            | دارد          | دارد              |
| Custom Allocator       | ندارد         | دارد              |
| معمولاً یک Allocation  | بله           | بله               |
| Custom Deleter مستقل   | ندارد         | ندارد             |
| مناسب برای Memory Pool | محدود         | مناسب             |
| استاندارد              | ++C11         | ++C11             |

در `allocate_shared` Allocator بخشی از مکانیزم مدیریت Allocation و Lifetime است و معمولاً در Control Block نگهداری می‌شود تا هنگام پایان کامل Ownership برای Deallocation مورد استفاده قرار گیرد. ([Cppreference][4])

---

# تفاوت `make_shared` و `make_unique`

این دو تابع را نباید فقط از نظر Syntax مقایسه کرد.

تفاوت اصلی مربوط به Ownership است.

در:

```cpp
auto ptr = std::make_unique<User>();
```

فقط یک Owner وجود دارد.

در:

```cpp
auto ptr = std::make_shared<User>();
```

چند Owner می‌توانند وجود داشته باشند.

بنابراین:

```text
make_unique
    │
    ▼
unique_ptr
    │
    ▼
Unique Ownership
```

در مقابل:

```text
make_shared
    │
    ▼
shared_ptr
    │
    ▼
Shared Ownership
```

اگر واقعاً Shared Ownership لازم نیست، معمولاً استفاده از `unique_ptr` یا حتی Object معمولی طراحی ساده‌تری ایجاد می‌کند.

---

# انتخاب بین Object معمولی، `unique_ptr` و `shared_ptr`

یک تصمیم‌گیری مناسب را می‌توان این‌گونه تصور کرد:

```text
آیا Object باید Dynamic باشد؟
        │
        ├── خیر
        │    └── Object معمولی
        │
        └── بله
             │
             ▼
      آیا چند Owner وجود دارد؟
             │
             ├── خیر
             │    └── unique_ptr / make_unique
             │
             └── بله
                  └── shared_ptr / make_shared
```

این مدل از تصمیم‌گیری بهتر از این است که همه Objectها را با `shared_ptr` بسازیم.

---

# اشتباه رایج استفاده از `shared_ptr` برای همه چیز

این کد لزوماً طراحی خوبی نیست:

```cpp
auto a = std::make_shared<A>();
auto b = std::make_shared<B>();
auto c = std::make_shared<C>();
```

صرف اینکه `shared_ptr` مدیریت حافظه را خودکار انجام می‌دهد، دلیل کافی برای استفاده از آن نیست.

اگر Object `B` فقط متعلق به `A` است، شاید این طراحی مناسب‌تر باشد:

```cpp
class A
{
private:
    std::unique_ptr<B> b_;
};
```

و اگر `B` ذاتاً بخشی از `A` است، شاید حتی بهتر باشد:

```cpp
class A
{
private:
    B b_;
};
```

بنابراین Smart Pointer باید بر اساس Ownership Model انتخاب شود، نه صرفاً برای حذف `delete`.

---

# جلوگیری از ساخت چند Control Block

یکی از خطرناک‌ترین اشتباهات در کار با `shared_ptr` این است که برای یک Object واحد چند Control Block ایجاد کنیم.

مثلاً:

```cpp
auto p1 = std::make_shared<User>();

User* raw = p1.get();

std::shared_ptr<User> p2(raw);
```

اکنون دو `shared_ptr` مستقل داریم که ممکن است تصور کنند هر کدام مالک Object هستند:

```text
Control Block #1 ──► User
Control Block #2 ──► User
```

این طراحی می‌تواند در نهایت به Double Delete منجر شود.

بنابراین نباید Raw Pointer مربوط به Object تحت مدیریت `shared_ptr` را دوباره داخل `shared_ptr` قرار داد.

---

# مدیریت صحیح Object در `enable_shared_from_this`

اگر یک Object نیاز دارد `shared_ptr` مربوط به خودش را تولید کند، طراحی مناسب استفاده از:

```cpp
std::enable_shared_from_this<T>
```

است.

نمونه:

```cpp
class Connection
    : public std::enable_shared_from_this<Connection>
{
public:
    std::shared_ptr<Connection> self()
    {
        return shared_from_this();
    }
};
```

ساخت:

```cpp
auto connection =
    std::make_shared<Connection>();
```

سپس:

```cpp
auto self =
    connection->self();
```

باعث می‌شود `self` با Control Block موجود Share شود، نه اینکه Control Block جدیدی ساخته شود.

---

# نکته مهم درباره Thread Safety

یکی از سوءبرداشت‌های رایج این است که Thread-Safe بودن بعضی عملیات مربوط به `shared_ptr` به معنی Thread-Safe بودن Object مدیریت‌شده است.

این دو موضوع کاملاً متفاوت هستند.

مثلاً:

```cpp
auto object = std::make_shared<Counter>();
```

ممکن است چند Thread بتوانند کپی‌های مستقل از `shared_ptr` داشته باشند.

اما این موضوع به این معنی نیست که:

```cpp
object->increment();
```

اگر از چند Thread هم‌زمان فراخوانی شود، حتماً Thread-Safe است.

به طور خلاصه:

```text
shared_ptr ownership safety
        ≠
managed object thread safety
```

اگر خود Object State قابل تغییر مشترک دارد، باید Synchronization مناسب برای خود Object در نظر گرفته شود.

---

# استفاده از `shared_ptr` در Containerها

یکی از کاربردهای متداول `make_shared` ساخت مجموعه‌ای از Objectهای دارای Shared Ownership است.

برای مثال:

```cpp
std::vector<std::shared_ptr<User>> users;

users.push_back(
    std::make_shared<User>("Ali", 30)
);

users.push_back(
    std::make_shared<User>("Sara", 25)
);
```

در اینجا Container مالک `shared_ptr`هاست و در نتیجه Ownerهای مربوط به Objectها را نگه می‌دارد.

با خروج Object از Container، اگر Owner دیگری وجود نداشته باشد، Object نیز Destroy خواهد شد.

---

# نکته مهم درباره `use_count`

اگر چنین کدی داشته باشیم:

```cpp
auto p = std::make_shared<User>();

auto p2 = p;
auto p3 = p;
```

می‌توان انتظار داشت:

```cpp
p.use_count() == 3
```

اما نباید از چنین بررسی‌ای برای تصمیم‌گیری حساس در برنامه‌های Concurrent استفاده کرد.

برای مثال چنین طراحی‌ای شکننده است:

```cpp
if (p.use_count() == 1)
{
    // Assume that this is the only owner
}
```

ممکن است Thread دیگری بلافاصله یک Copy از `p` ایجاد کند.

در نتیجه `use_count` بیشتر یک ابزار مشاهده و Debugging است تا مکانیزم طراحی Ownership.

---

# تفاوت Lifetime Object و Lifetime Control Block

این مفهوم برای درک حرفه‌ای `shared_ptr` بسیار مهم است.

دو Lifetime متفاوت داریم:

```text
Object Lifetime
        │
        ▼
تا زمانی که shared ownership باقی است

Control Block Lifetime
        │
        ▼
تا زمانی که shared و weak ownership هر دو تمام شوند
```

به صورت ساده:

```text
آخرین shared_ptr
       │
       ▼
Object destroyed
       │
       │
       ▼
Control Block ممکن است هنوز باقی باشد
       │
       ▼
آخرین weak_ptr
       │
       ▼
Control Block destroyed
```

این تفاوت دقیقاً همان چیزی است که در Trade-off مربوط به `make_shared` و Objectهای بزرگ اهمیت پیدا می‌کند.

---

# تفاوت `make_shared` با `new` از دید طراحی

عبارت:

```cpp
new User();
```

تنها مسئول ساخت Object در Dynamic Storage است.

اما:

```cpp
std::make_shared<User>();
```

هم‌زمان مفهوم ساخت Object و Shared Ownership را وارد طراحی می‌کند.

بنابراین این دو فقط دو Syntax مختلف برای یک کار نیستند.

می‌توان گفت:

```text
new
 └── ایجاد Dynamic Object

make_shared
 └── ایجاد Dynamic Object + Shared Ownership
```

---

# تفاوت `shared_ptr(new T)` و `make_shared`

مقایسه اصلی را می‌توان این‌گونه خلاصه کرد:

| موضوع                           | `shared_ptr(new T(...))` | `make_shared<T>(...)`         |
| ------------------------------- | ------------------------ | ----------------------------- |
| Shared Ownership                | دارد                     | دارد                          |
| مدیریت خودکار Lifetime          | دارد                     | دارد                          |
| Allocation معمول                | حداقل دو                 | معمولاً یک                    |
| Custom Deleter                  | دارد                     | ندارد                         |
| Constructor غیرعمومی            | در Context مجاز ممکن است | محدود به دسترسی `make_shared` |
| Class-Specific `operator new`   | رفتار `new T` را دارد    | Allocation متفاوت             |
| Object بزرگ + `weak_ptr` طولانی | مزیت بالقوه              | ممکن است حافظه دیرتر آزاد شود |
| خوانایی                         | پایین‌تر                 | بالاتر                        |
| روش معمول Modern ++C            | معمولاً خیر              | معمولاً بله                   |

این تفاوت‌ها مستقیماً در مستندات Standard Library نیز به عنوان Trade-offهای `make_shared` ذکر شده‌اند. ([Cppreference][1])

---

# چه زمانی `make_shared` انتخاب مناسبی است؟

در حالت معمول، اگر واقعاً Shared Ownership لازم باشد و محدودیت خاصی وجود نداشته باشد، `make_shared` انتخاب طبیعی برای ساخت Object است.

سناریوهای مناسب شامل موارد زیر هستند:

* ساخت Objectهای معمولی با Shared Ownership.
* ساخت Objectهای کوچک و متوسط.
* ساخت تعداد زیادی Object که کاهش Allocation برای آن‌ها مفید است.
* استفاده از `enable_shared_from_this`.
* ساخت Objectهای Polymorphic.
* ساخت Objectهایی که Custom Deleter نیاز ندارند.

---

# چه زمانی `make_shared` انتخاب مناسبی نیست؟

استفاده از `make_shared` ممکن است مناسب نباشد اگر:

* Custom Deleter لازم باشد.
* Constructor انتخاب‌شده قابل دسترسی برای `make_shared` نباشد.
* Class-Specific Allocation behavior اهمیت داشته باشد.
* Object بسیار بزرگ باشد و `weak_ptr` مدت طولانی زنده بماند.
* نیاز به Allocator خاص داشته باشیم.
* اساساً Shared Ownership لازم نباشد.

در مورد نیاز به Allocator، `allocate_shared` می‌تواند گزینه مناسب‌تری باشد. ([Cppreference][4])

---

# نکات مهم درباره Objectهای بزرگ

فرض کنید Object دارای یک Buffer بسیار بزرگ است:

```cpp
struct LargeObject
{
    std::array<std::byte, 100 * 1024 * 1024> buffer;
};
```

اگر چنین Objectی با `make_shared` ساخته شود:

```cpp
auto object =
    std::make_shared<LargeObject>();
```

و سپس یک `weak_ptr` مدت زیادی باقی بماند:

```cpp
std::weak_ptr<LargeObject> observer = object;

object.reset();
```

خود Object Destroy می‌شود، اما Allocation مشترک مربوط به Control Block ممکن است تا پایان Lifetime آخرین `weak_ptr` باقی بماند.

در چنین شرایطی جداسازی Allocation Object و Control Block می‌تواند مزیت داشته باشد.

این یکی از معدود شرایطی است که استفاده مستقیم از:

```cpp
std::shared_ptr<LargeObject>(
    new LargeObject()
);
```

ممکن است از نظر Memory Lifetime منطقی‌تر باشد. ([Cppreference][1])

---

# بررسی Feature-Test Macroها

برای بررسی قابلیت‌های مرتبط با `make_shared` می‌توان از Feature-Test Macroها استفاده کرد.

برای Array support مربوط به `make_shared`:

```cpp
__cpp_lib_shared_ptr_arrays
```

این قابلیت در ++C20 با مقدار Feature-Test Macro زیر شناخته می‌شود:

```cpp
201707L
```

برای `make_shared_for_overwrite` نیز Macro زیر وجود دارد:

```cpp
__cpp_lib_smart_ptr_for_overwrite
```

که مقدار استاندارد آن برای ++C20 برابر:

```cpp
202002L
```

است. ([Cppreference][1])

---

# وضعیت قابلیت‌ها در نسخه‌های مختلف ++C

| قابلیت                          | استاندارد |
| ------------------------------- | --------- |
| `std::make_shared`              | ++C11     |
| `std::allocate_shared`          | ++C11     |
| `std::make_unique`              | ++C14     |
| Arrayهای `make_shared`          | ++C20     |
| `make_shared_for_overwrite`     | ++C20     |
| `allocate_shared_for_overwrite` | ++C20     |
| `constexpr make_shared`         | ++C26     |
| `constexpr allocate_shared`     | ++C26     |

اطلاعات مربوط به Standard Versionها و Featureهای فوق در مستندات Standard Library قابل مشاهده است. ([Cppreference][4])

---

# یک مثال کامل و واقعی

در این مثال یک سیستم ساده مدیریت User داریم:

```cpp
#include <iostream>
#include <memory>
#include <string>

class User
{
public:
    User(std::string name, int age)
        : name_(std::move(name)),
          age_(age)
    {
        std::cout << "User created\n";
    }

    ~User()
    {
        std::cout << "User destroyed\n";
    }

    void print() const
    {
        std::cout << name_
                  << " - "
                  << age_
                  << '\n';
    }

private:
    std::string name_;
    int age_;
};

int main()
{
    auto user =
        std::make_shared<User>("Ali", 30);

    user->print();

    {
        auto another_owner = user;

        another_owner->print();

        std::cout
            << "Owners: "
            << user.use_count()
            << '\n';
    }

    std::cout
        << "Owners: "
        << user.use_count()
        << '\n';
}
```

در این برنامه ابتدا یک Owner داریم.

سپس داخل Scope داخلی یک Owner دیگر ایجاد می‌شود.

با خروج `another_owner` از Scope، آن Owner از بین می‌رود، اما Object همچنان توسط `user` مدیریت می‌شود.

در نهایت با پایان `main`، آخرین `shared_ptr` Destroy می‌شود و `User` نیز Destroy خواهد شد.

---

# یک مثال برای مشاهده Lifetime

برای مشاهده دقیق‌تر Lifetime می‌توان از Constructor و Destructor استفاده کرد:

```cpp
#include <iostream>
#include <memory>

class Resource
{
public:
    Resource()
    {
        std::cout << "Resource constructed\n";
    }

    ~Resource()
    {
        std::cout << "Resource destroyed\n";
    }
};

int main()
{
    auto p1 = std::make_shared<Resource>();

    {
        auto p2 = p1;

        std::cout
            << p1.use_count()
            << '\n';
    }

    std::cout
        << p1.use_count()
        << '\n';
}
```

تا زمانی که `p1` وجود دارد، Resource زنده باقی می‌ماند.

بعد از خروج `p2` از Scope، تعداد Ownerها کاهش پیدا می‌کند، اما Resource هنوز توسط `p1` مدیریت می‌شود.

در پایان `main`، آخرین Owner از بین می‌رود و Destructor اجرا می‌شود.

---

# الگوی پیشنهادی برای Modern ++C

اگر Shared Ownership واقعاً موردنیاز است، الگوی معمول این است:

```cpp
auto object =
    std::make_shared<MyType>(
        constructor_arg1,
        constructor_arg2
    );
```

اگر فقط Unique Ownership داریم:

```cpp
auto object =
    std::make_unique<MyType>(
        constructor_arg1,
        constructor_arg2
    );
```

و اگر Dynamic Allocation اصلاً لازم نیست:

```cpp
MyType object(
    constructor_arg1,
    constructor_arg2
);
```

این سه حالت را نباید صرفاً بر اساس Syntax انتخاب کرد؛ مدل Ownership و Lifetime باید ابتدا مشخص شود.

---

# اشتباهات رایج در استفاده از `make_shared`

## ساخت `shared_ptr` دوم از `get`

این کار خطرناک است:

```cpp
auto p1 = std::make_shared<User>();

auto p2 =
    std::shared_ptr<User>(p1.get());
```

این دو Pointer لزوماً Control Block مشترک ندارند و می‌توانند باعث Double Delete شوند.

---

## استفاده از `shared_ptr` بدون نیاز به Shared Ownership

این طراحی:

```cpp
auto user = std::make_shared<User>();
```

فقط زمانی منطقی است که واقعاً چند بخش برنامه باید مالک User باشند.

اگر فقط یک مالک داریم، `unique_ptr` معمولاً مدل Ownership واضح‌تری ارائه می‌دهد.

---

## ایجاد Circular Ownership

این طراحی خطرناک است:

```cpp
class Parent;

class Child
{
public:
    std::shared_ptr<Parent> parent;
};
```

اگر Parent نیز Child را با `shared_ptr` نگه دارد، احتمال Cycle وجود دارد.

برای رابطه غیرمالکانه معمولاً `weak_ptr` انتخاب مناسب‌تری است.

---

## نگهداری طولانی Raw Pointer حاصل از `get`

این کد:

```cpp
auto user = std::make_shared<User>();

User* raw = user.get();

user.reset();
```

باعث می‌شود `raw` به Objectی اشاره کند که دیگر Lifetime آن پایان یافته است.

بنابراین Raw Pointer حاصل از `get()` نباید بیشتر از Lifetime Owner استفاده شود.

---

## استفاده از `use_count()` به عنوان Synchronization

این طراحی مناسب نیست:

```cpp
if (object.use_count() == 1)
{
    // Assume exclusive ownership
}
```

در محیط Concurrent وضعیت Ownership می‌تواند تغییر کند.

---

# چک‌لیست انتخاب `make_shared`

قبل از استفاده از `make_shared` این پرسش‌ها را بررسی کنید:

* آیا Object واقعاً باید Dynamic باشد؟
* آیا بیش از یک Owner وجود دارد؟
* آیا Shared Ownership واقعاً بخشی از Design است؟
* آیا Custom Deleter لازم نیست؟
* آیا Constructor برای `make_shared` قابل دسترسی است؟
* آیا Object بسیار بزرگ نیست؟
* آیا `weak_ptr`های طولانی‌مدت نداریم؟
* آیا Class-Specific Allocation behavior مهم نیست؟
* آیا Allocator اختصاصی لازم نیست؟
* آیا Reference Cycle ایجاد نمی‌شود؟

اگر پاسخ این پرسش‌ها با Design شما سازگار باشد، `make_shared` معمولاً انتخاب مناسبی برای ایجاد `shared_ptr` است.

---

# جمع‌بندی نهایی

`std::make_shared` یک Function Template در Standard Library است که برای ساخت Object تحت مدیریت `std::shared_ptr` استفاده می‌شود.

شکل معمول استفاده آن:

```cpp
auto ptr = std::make_shared<MyType>(args...);
```

است.

مهم‌ترین مزیت آن نسبت به:

```cpp
std::shared_ptr<MyType>(
    new MyType(args...)
);
```

این است که در پیاده‌سازی‌های معمول Object و Control Block در یک Allocation قرار می‌گیرند؛ در حالی که ساخت `shared_ptr` از Raw Pointer معمولاً حداقل دو Allocation نیاز دارد. ([Cppreference][1])

این موضوع می‌تواند باعث کاهش هزینه Allocation و بهبود Locality حافظه شود.

از طرف دیگر، `make_shared` محدودیت‌هایی نیز دارد:

* امکان Custom Deleter ندارد.
* برای Constructorهای غیرقابل‌دسترسی محدودیت دارد.
* با Class-Specific `operator new` می‌تواند رفتار متفاوتی داشته باشد.
* برای Objectهای بسیار بزرگ همراه با `weak_ptr`های طولانی‌مدت ممکن است باعث باقی ماندن Allocation بزرگ‌تری شود.
* اگر Allocator اختصاصی لازم باشد، `allocate_shared` گزینه مناسب‌تری است. ([Cppreference][4])

قابلیت‌های مهم مرتبط با آن نیز شامل موارد زیر هستند:

```text
std::make_shared
        │
        ├── std::shared_ptr
        │
        ├── std::weak_ptr
        │
        ├── std::enable_shared_from_this
        │
        ├── std::allocate_shared
        │
        ├── std::make_shared_for_overwrite
        │
        └── Array support
```

در نهایت مهم‌ترین اصل این است:

> مالکیت را ابتدا طراحی کنید و سپس Smart Pointer را انتخاب کنید.

اگر یک Object یک Owner دارد، معمولاً `unique_ptr` یا Object معمولی مناسب‌تر است.

اگر چند Owner واقعی وجود دارند، `shared_ptr` معنا پیدا می‌کند و در چنین شرایطی `make_shared` معمولاً روش ترجیحی برای ساخت آن است.

اگر رابطه فقط مشاهده‌ای و بدون مالکیت است، `weak_ptr` می‌تواند ابزار مناسب‌تری باشد.

بنابراین ارزش واقعی `make_shared` فقط در کوتاه‌تر شدن این کد:

```cpp
auto ptr = std::make_shared<MyType>();
```

---

نیست؛ بلکه در ترکیب **Shared Ownership، مدیریت خودکار Lifetime، Exception Safety بهتر و معمولاً Allocation کارآمدتر** است. ([Cppreference][1])

[1]: https://en.cppreference.com/%20%20cpp/memory/shared_ptr/make_shared?utm_source=chatgpt.com "std::make_shared, std::make_shared_for_overwrite - cppreference.com"
[2]: https://en.cppreference.com/cpp/memory/shared_ptr/get?utm_source=chatgpt.com "std::shared_ptr<T>::get - cppreference.com"
[3]: https://en.cppreference.com/cpp/feature_test?utm_source=chatgpt.com "Feature testing (since C++20) - cppreference.com"
[4]: https://en.cppreference.com/cpp/memory/shared_ptr/allocate_shared?utm_source=chatgpt.com "std::allocate_shared, std::allocate_shared_for_overwrite - cppreference.com"

---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>