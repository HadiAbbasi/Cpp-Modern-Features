<div align="center">

[🇺🇸 English](../../en/cpp14/make_unique.md) | [🇮🇷 فارسی](./make_unique.md)

</div>

---

## فهرست مطالب

- [دستور `std::make_unique` چیست؟](#-دستور-stdmake_unique-چیست)
- [چرا اصلاً به `make_unique` نیاز داریم؟](#-چرا-اصلاً-به-make_unique-نیاز-داریم)
- [رابطه‌ی `make_unique` با Heap](#-رابطه‌ی-make_unique-با-heap)
- [تفاوت `new` با `make_unique`](#-تفاوت-new-با-make_unique)
- [ساختار کلی `make_unique`](#-ساختار-کلی-make_unique)
- [مثال ساده](#-مثال-ساده)
- [چرا `make_unique` بهتر از `new` خام است؟](#-چرا-make_unique-بهتر-از-new-خام-است)
- [مشکل Memory Leak با `new`](#-مشکل-memory-leak-با-new)
- [مزیت مهم‌تر: Exception Safety](#-مزیت-مهمتر-exception-safety)
- [تفاوت `make_unique` با `unique_ptr(new ...)`](#-تفاوت-make_unique-با-unique_ptrnew-)
- [انتقال Ownership](#-انتقال-ownership)
- [استفاده در توابع](#-استفاده-در-توابع)
- [استفاده به‌عنوان مقدار بازگشتی](#-استفاده-بهعنوان-مقدار-بازگشتی)
- [ساخت Object با Constructor Arguments](#-ساخت-object-با-constructor-arguments)
- [ساخت Array با `make_unique`](#-ساخت-array-با-make_unique)
- [ماهیت `make_unique_for_overwrite`](#-ماهیت-make_unique_for_overwrite)
- [تفاوت `make_unique` و `make_unique_for_overwrite`](#-تفاوت-make_unique-و-make_unique_for_overwrite)
- [دسترسی به Object](#-دسترسی-به-object)
- [متدهای مهم `unique_ptr`](#-متدهای-مهم-unique_ptr)
- [آیا `make_unique` سریع‌تر از `new` است؟](#-آیا-make_unique-سریعتر-از-new-است)
- [آیا `make_unique` یک Allocation انجام می‌دهد؟](#-آیا-make_unique-یک-allocation-انجام-میدهد)
- [`make_unique` و `shared_ptr`](#-make_unique-و-shared_ptr)
- [چه زمانی نباید از `make_unique` استفاده کرد؟](#-چه-زمانی-نباید-از-make_unique-استفاده-کرد)
- [خطاهای رایج](#-خطاهای-رایج)
- [الگوی پیشنهادی در C++ مدرن](#-الگوی-پیشنهادی-در-c-مدرن)
- [مهم‌ترین تفاوت دستورات `new` و `make_unique`](#-مهمترین-تفاوت-دستورات-new-و-make_unique)
- [یک مثال کامل](#-یک-مثال-کامل)
- [یک تصویر ذهنی بسیار مهم](#-یک-تصویر-ذهنی-بسیار-مهم)
- [خلاصه و جمع‌بندی](#-خلاصه-و-جمع‌بندی)
- [مشارکت‌ها](#-مشارکت-ها)

---

# دستور `std::make_unique` در ++C؛ ساخت امن و مدرن `unique_ptr`


# دستور `std::make_unique` چیست؟

دستور `std::make_unique` یک تابع از نوع Function Template در کتابخانه استاندارد C++ است که از C++14 به زبان اضافه شده است. وظیفه‌ی اصلی آن بسیار ساده است:

> یک Object را ایجاد می‌کند، آن را در Dynamic Storage قرار می‌دهد و نتیجه را داخل یک `std::unique_ptr` قرار می‌دهد.

این تابع در Header زیر قرار دارد:

```cpp
#include <memory>
```

مثلاً:

```cpp
auto user = std::make_unique<User>();
```

در اینجا:

```text
Heap
 └── User Object
       ↑
       │
   unique_ptr
```

یعنی Object به‌صورت Dynamic ساخته شده و `unique_ptr` مالک آن شده است.

طبق استاندارد، `make_unique` برای انواع معمولی تقریباً معادل ساختن `unique_ptr` با `new` است.

---

# چرا اصلاً به `make_unique` نیاز داریم؟

قبل از C++14 برای ساخت یک `unique_ptr` معمولاً چنین کدی نوشته می‌شد:

```cpp
std::unique_ptr<User> user(new User());
```

این کد از نظر فنی درست است. اما دو مشکل دارد:

###  Exception Safety

مزیت مهم‌ `make_unique` این است که در C++ مدرن، هدف این است که مدیریت حافظه مستقیماً در اختیار RAII قرار بگیرد و استفاده‌ی مستقیم از `new` تا حد ممکن حذف شود.

تابع `make_unique` دقیقاً برای همین الگو طراحی شده است. یکی از انگیزه‌های اصلی اضافه‌شدن آن به C++، ایمنی بیشتر در برابر Exceptionها بود.

---

# رابطه‌ی `make_unique` با Heap

`make_unique` فقط یک روش استاندارد و امن‌تر برای ساخت Object در Dynamic Storage و قرار دادن مالکیت آن در `unique_ptr` است. مثلاً:

```cpp
auto p = std::make_unique<int>(42);
```

از نظر مفهومی اتفاق زیر رخ می‌دهد:

```text
Dynamic Storage
┌───────────────┐
│      42       │
└───────────────┘
        ↑
        │
        │ pointer
        │
┌───────────────┐
│ unique_ptr<int>│
└───────────────┘
```

وقتی `p` از Scope خارج شود، Object نیز Destroy و حافظه‌ی آن آزاد می‌شود.

پس `unique_ptr` مالکیت Object را در اختیار دارد و هنگام نابودی، از Deleter خود برای آزادسازی Object استفاده می‌کند.

---

# تفاوت `new` با `make_unique`

مهم است بین این دو مفهوم تفاوت بگذاریم:

```cpp
new User();
```

و:

```cpp
std::make_unique<User>();
```

`new` یک **new-expression** است.

کار آن:

1. اختصاص حافظه
2. ساخت Object
3. برگرداندن Raw Pointer

مثلاً:

```cpp
User* p = new User();
```

نتیجه:

```text
p ────────► User
```

اما مسئولیت آزاد کردن Object نیز با برنامه‌نویس است:

```cpp
delete p;
```

---

در مقابل:

```cpp
auto p = std::make_unique<User>();
```

نتیجه:

```text
p
│
└──── unique_ptr ─────► User
```

و وقتی `p` از Scope خارج شود:

```text
unique_ptr destructor
        ↓
     delete User
```

بنابراین دیگر `delete` دستی لازم نیست.

اگر Raw Pointer (اشاره گر ستاره دار) حاصل از `new` گم شود، Memory Leak رخ می‌دهد؛ `unique_ptr` برای جلوگیری از چنین مدیریت دستی‌ای طراحی شده است.

---

# ساختار کلی `make_unique`

برای یک Object معمولی:

```cpp
std::make_unique<T>(arguments...);
```

مثلاً:

```cpp
auto p = std::make_unique<int>(10);
```

یا:

```cpp
auto user = std::make_unique<User>("Hadi", 42);
```

Arguments مستقیماً به Constructor کلاس ارسال می‌شوند.

مثلاً:

```cpp
class User
{
public:
    User(std::string name, int age)
        : name(std::move(name)),
          age(age)
    {
    }

private:
    std::string name;
    int age;
};
```

سپس:

```cpp
auto user =
    std::make_unique<User>("Ali", 30);
```

تقریباً معادل مفهومی این است:

```cpp
std::unique_ptr<User>(
    new User("Ali", 30)
);
```

استاندارد نیز برای overload مربوط به Object معمولی، همین مفهوم را مشخص می‌کند.

---

# مثال ساده

```cpp
#include <iostream>
#include <memory>

class Person
{
public:
    Person()
    {
        std::cout << "Person created\n";
    }

    ~Person()
    {
        std::cout << "Person destroyed\n";
    }
};

int main()
{
    auto person = std::make_unique<Person>();

    std::cout << "Using person...\n";
}
```

ترتیب اتفاقات:

```text
main()
  │
  ├── make_unique
  │      │
  │      └── ساخت Person
  │
  ├── استفاده از Person
  │
  └── خروج از Scope
           │
           └── unique_ptr destructor
                    │
                    └── Person destructor
```

بنابراین نباید بنویسیم:

```cpp
delete person.get();
```

---

# چرا `make_unique` بهتر از `new` خام است؟

مقایسه:

```cpp
auto user = new User();
```

در برابر:

```cpp
auto user = std::make_unique<User>();
```

### با `new`

باید Ownership را دستی مدیریت کرد:

```cpp
User* user = new User();

// ...

delete user;
```

اگر در مسیر اجرای برنامه فراموش شود که delete شود:

```cpp
return;
```

یا Exception رخ دهد:

```cpp
throw ...;
```

ممکن است `delete` اجرا نشود.

---

### با `make_unique`

```cpp
auto user = std::make_unique<User>();
```

مدیریت Lifetime به Scope سپرده می‌شود.

این همان RAII است:

```text
Scope شروع
    ↓
Object ساخته می‌شود
    ↓
استفاده
    ↓
Scope پایان
    ↓
unique_ptr destructor
    ↓
Object destroy
    ↓
memory آزاد
```

---

# مشکل Memory Leak با `new`

مثلاً:

```cpp
void foo()
{
    User* user = new User();

    doSomething();

    delete user;
}
```

تا زمانی که متد `doSomething()` ناگهان `Exception` ایجاد نکند، ممکن است همه چیز درست باشد. اما:

```cpp
void foo()
{
    User* user = new User();

    doSomething(); // ممکن است throw کند

    delete user;
}
```

اگر `doSomething()` ناگهان `Exception` ایجاد کند، اجرای تابع در همان نقطه متوقف می‌شود و:

```cpp
delete user;
```

دیگر اجرا نمی‌شود. در نتیجه:

```text
User
 ↓
memory leak
```

اما:

```cpp
void foo()
{
    auto user = std::make_unique<User>();

    doSomething();
}
```

حتی اگر Exception رخ دهد، Stack Unwinding باعث Destroy شدن `unique_ptr` می‌شود و Object آزاد خواهد شد.

---

# مزیت مهم‌تر: Exception Safety

این موضوع یکی از مهم‌ترین دلایل استفاده از `make_unique` است.

فرض کنید تابعی داریم:

```cpp
void process(
    std::unique_ptr<A> a,
    std::unique_ptr<B> b
);
```

می‌توان نوشت:

```cpp
process(
    std::make_unique<A>(),
    std::make_unique<B>()
);
```

هر Object از همان ابتدا تحت مدیریت RAII قرار دارد.

در مقابل، استفاده‌ی مستقیم از `new` در Expressionهای پیچیده می‌تواند در برابر Exceptionها خطرناک‌تر باشد. انگیزه‌ی اصلی `make_unique` نیز همین کاهش چنین خطراتی بوده است.

---

# تفاوت `make_unique` با `unique_ptr(new ...)`

این دو:

```cpp
auto p =
    std::make_unique<User>();
```

و:

```cpp
std::unique_ptr<User> p(
    new User()
);
```

از نظر Objectی که ایجاد می‌شود، بسیار نزدیک هستند.

حتی می‌توان مفهوم `make_unique` را برای نوع غیرآرایه‌ای تقریباً به شکل زیر تصور کرد:

```cpp
return std::unique_ptr<T>(
    new T(std::forward<Args>(args)...)
);
```

که در مستندات استاندارد نیز به همین شکل نشان داده شده است.

اما `make_unique`:

* خواناتر است
* Type را دوباره تکرار نمی‌کند
* Ownership را از همان ابتدا داخل Smart Pointer قرار می‌دهد
* برای ساخت Temporaryها و Expressionهای پیچیده‌تر، Exception Safety بهتری فراهم می‌کند

بنابراین در C++ مدرن معمولاً:

```cpp
std::make_unique<T>()
```

انتخاب بهتری است.

---

# انتقال Ownership

`unique_ptr` یعنی:

> فقط یک مالک وجود دارد.

مثلاً:

```cpp
auto p1 = std::make_unique<User>();
```

نمی‌توان نوشت:

```cpp
auto p2 = p1; // ERROR
```

چون Copy کردن `unique_ptr` ممنوع است. اما Ownership قابل انتقال است:

```cpp
auto p2 = std::move(p1);
```

بعد از آن:

```text
قبل:

p1 ─────► User


بعد:

p1 ─────► nullptr

p2 ─────► User
```

این ویژگی برای مدل کردن مالکیت انحصاری بسیار مهم است.

`unique_ptr` از نوع `Moveable ` است اما Copyable نیست.

---

# استفاده در توابع

مثلاً:

```cpp
void process(std::unique_ptr<User> user)
{
    user->run();
}
```

فراخوانی:

```cpp
process(
    std::make_unique<User>()
);
```

Ownership به تابع منتقل می‌شود.

وقتی `process` تمام شود، `user` از Scope خارج می‌شود و Object Destroy خواهد شد.

این الگو بسیار مناسب زمانی است که تابع **مالکیت Object را دریافت می‌کند**.

---

# استفاده به‌عنوان مقدار بازگشتی

یکی از کاربردهای بسیار خوب:

```cpp
std::unique_ptr<User> createUser()
{
    return std::make_unique<User>();
}
```

استفاده:

```cpp
auto user = createUser();
```

اینجا Ownership به Caller منتقل می‌شود.

هیچ `delete` دستی لازم نیست.

این یکی از الگوهای استاندارد در طراحی APIهای مدرن C++ است.

---

# ساخت Object با Constructor Arguments

فرض کنید:

```cpp
class Product
{
public:
    Product(
        int id,
        std::string name,
        double price
    )
        : id(id),
          name(std::move(name)),
          price(price)
    {
    }

private:
    int id;
    std::string name;
    double price;
};
```

می‌توان نوشت:

```cpp
auto product =
    std::make_unique<Product>(
        10,
        "Laptop",
        1500.0
    );
```

Arguments:

```cpp
10
"Laptop"
1500.0
```

مستقیماً به Constructor ارسال می‌شوند.

---

# ساخت Array با `make_unique`

دستور `make_unique` فقط برای Objectهای معمولی نیست. برای Array با اندازه‌ی runtime نیز overload وجود دارد:

```cpp
auto data =
    std::make_unique<int[]>(10);
```

یعنی:

```text
10 × int
```

در Dynamic Storage ساخته می‌شود. سپس:

```cpp
data[0] = 10;
data[1] = 20;
data[2] = 30;
```

و:

```cpp
std::cout << data[2];
```

قابل استفاده است. برای ArrayهایDynamic ، نوع داده unique_ptr<T[]>  دارای  operator[] است.

نکته مهم:

```cpp
std::make_unique<int[]>(10);
```

مجاز است. اما:

```cpp
std::make_unique<int[10]>();
```

برای `make_unique` پشتیبانی نمی‌شود؛ overload مربوط به Array با Bound ثابت حذف شده است.

---

# ماهیت `make_unique_for_overwrite`

از C++20 تابع دیگری اضافه شده است:

```cpp
std::make_unique_for_overwrite
```

مثلاً:

```cpp
auto p =
    std::make_unique_for_overwrite<int>();
```

یا:

```cpp
auto buffer =
    std::make_unique_for_overwrite<char[]>(1024);
```

این تابع برای زمانی طراحی شده که مقدار اولیه‌ی Object بلافاصله قرار است توسط برنامه نوشته شود.

---

# تفاوت `make_unique` و `make_unique_for_overwrite`

این دو را مقایسه کنیم:

```cpp
auto p =
    std::make_unique<int>();
```

و:

```cpp
auto p =
    std::make_unique_for_overwrite<int>();
```

اولی Object را **Value-Initialize** می‌کند.

برای `int` یعنی مقدار:

```cpp
0
```

خواهد بود.

اما نسخه‌ی `for_overwrite` از Default Initialization استفاده می‌کند و برای نوع‌هایی مثل `int` مقدار اولیه‌ای برای صفر کردن ایجاد نمی‌کند.

بنابراین:

```cpp
std::make_unique<int>();
```

برای:

```text
int = 0
```

مناسب است.

ولی:

```cpp
std::make_unique_for_overwrite<int>();
```

زمانی مناسب است که مقدار بلافاصله overwrite شود:

```cpp
auto p =
    std::make_unique_for_overwrite<int>();

*p = 42;
```

برای Bufferها نیز کاربرد مهمی دارد:

```cpp
auto buffer =
    std::make_unique_for_overwrite<char[]>(1024);

// بلافاصله buffer پر می‌شود
read(fd, buffer.get(), 1024);
```

`make_unique_for_overwrite` از C++20 اضافه شده است.

---

# دسترسی به Object

مثلاً:

```cpp
auto user =
    std::make_unique<User>();
```

دو روش اصلی:

```cpp
(*user).setName("Ali");
```

یا:

```cpp
user->setName("Ali");
```

عملگر:

```cpp
->
```

در `unique_ptr` به Object تحت مالکیت اشاره می‌کند.

---

# متدهای مهم `unique_ptr`

دستور `make_unique` خودش متدهای زیادی ندارد؛ چون یک **factory function** است و نتیجه‌ی آن `unique_ptr` است. بنابراین امکانات اصلی بعد از ساخت، از طریق `unique_ptr` در دسترس هستند.

مهم‌ترین‌ها:

```text
get()
release()
reset()
swap()
get_deleter()
operator bool
operator*
operator->
operator[]
```

و در C++ مدرن قابلیت‌های مقایسه و برخی امکانات دیگر نیز وجود دارند.

---

# آیا `make_unique` سریع‌تر از `new` است؟

این نکته بسیار مهم است:

> `make_unique` ذاتاً یک تکنیک Optimization برای سریع‌تر کردن Allocation نیست.

مثلاً:

```cpp
std::make_unique<User>();
```

در مقابل:

```cpp
std::unique_ptr<User>(
    new User()
);
```

قرار نیست الزاماً Allocation سریع‌تری داشته باشد.

حتی مستندات و توضیحات مربوط به تفاوت `make_unique` و ساخت مستقیم `unique_ptr` با `new` تأکید می‌کنند که دلیل اصلی `make_unique`، Exception Safety و طراحی بهتر Ownership است، نه یک بهینه‌سازی مشابه `make_shared`.

---

# آیا `make_unique` یک Allocation انجام می‌دهد؟

برای Object معمولی، از نظر مفهومی:

```cpp
auto p =
    std::make_unique<User>();
```

یک Object در Dynamic Storage ایجاد می‌کند.

در مقایسه با `make_shared`، `make_unique` دارای مکانیزم Control Block جداگانه‌ای نیست.

این تفاوت مهم است.

### `unique_ptr`

```text
unique_ptr
    │
    └────► Object
```

### `shared_ptr`

معمولاً مفهومی شبیه:

```text
shared_ptr ───► Control Block
                    │
                    └──► Object
```

بنابراین `make_unique` نباید با `make_shared` از نظر Allocation و Performance یکسان فرض شود.

---

# `make_unique` و `shared_ptr`

برای مالکیت انحصاری:

```cpp
auto p =
    std::make_unique<User>();
```

برای مالکیت اشتراکی:

```cpp
auto p =
    std::make_shared<User>();
```

تفاوت اصلی:

```text
unique_ptr
    ↓
یک مالک
```

در مقابل:

```text
shared_ptr
    ↓
چند مالک
```

در طراحی C++ مدرن، اگر واقعاً به Shared Ownership نیاز نیست، `unique_ptr` معمولاً انتخاب ساده‌تر و مناسب‌تری است.

---

# چه زمانی نباید از `make_unique` استفاده کرد؟

`make_unique` بسیار مناسب است، اما برای همه‌ی سناریوها نیست.

### 1. Object اصلاً نباید Dynamic باشد

اگر Lifetime Object محدود به Scope است:

```cpp
User user;
```

معمولاً بهتر از:

```cpp
auto user =
    std::make_unique<User>();
```

است. Heap را فقط به دلیل «مدرن بودن» نباید انتخاب کرد.

---

### 2. مالکیت اصلاً لازم نیست

اگر فقط می‌خواهید به Object موجود اشاره کنید:

```cpp
User user;

User* p = &user;
```

لزومی ندارد:

```cpp
std::unique_ptr<User>
```

ایجاد شود.

---

### 3. Custom Deleter نیاز است

در این حالت `make_unique` مستقیماً ابزار مناسبی برای ساخت `unique_ptr` با Deleter سفارشی نیست.

---

### 4. مالکیت اشتراکی لازم است

اگر چند بخش برنامه واقعاً باید مالک Object باشند:

```cpp
std::shared_ptr
```

ممکن است مناسب‌تر باشد.

---

# خطاهای رایج

## خطای اول: Delete کردن Object

اشتباه:

```cpp
auto p =
    std::make_unique<User>();

delete p.get();
```

چون `p` همچنان مالک Object است.

---

## خطای دوم: Copy کردن

اشتباه:

```cpp
auto a =
    std::make_unique<User>();

auto b = a;
```

`unique_ptr` قابل Copy نیست.

در صورت نیاز به انتقال:

```cpp
auto b =
    std::move(a);
```

---

## خطای سوم: استفاده‌ی غیرضروری از `get()`

مثلاً:

```cpp
auto user =
    std::make_unique<User>();

User* p = user.get();

p->run();
```

اگر Raw Pointer لازم نیست، ساده‌تر:

```cpp
user->run();
```

است.

---

## خطای چهارم: استفاده‌ی غیرضروری از Heap

این:

```cpp
auto user =
    std::make_unique<User>();
```

همیشه بهتر از:

```cpp
User user;
```

نیست. اگر Ownership Dynamic لازم نیست، Object معمولی روی Stack معمولاً انتخاب ساده‌تری است.

---

## خطای پنجم: استفاده از `release()` بدون برنامه برای Ownership

مثلاً:

```cpp
auto p =
    std::make_unique<User>();

User* raw = p.release();
```

از این لحظه:

```text
unique_ptr
     └── nullptr

raw
     └────► User
```

دیگر `unique_ptr` مسئول Object نیست.

بنابراین `release()` باید آگاهانه استفاده شود.

---

# الگوی پیشنهادی در C++ مدرن

برای ساخت Object با مالکیت انحصاری:

```cpp
auto object =
    std::make_unique<MyClass>();
```

برای Constructor:

```cpp
auto object =
    std::make_unique<MyClass>(
        arg1,
        arg2
    );
```

برای بازگرداندن مالکیت:

```cpp
std::unique_ptr<MyClass>
create()
{
    return std::make_unique<MyClass>();
}
```

برای انتقال مالکیت:

```cpp
auto b = std::move(a);
```

برای استفاده:

```cpp
a->method();
```

برای بررسی:

```cpp
if (a)
{
    a->method();
}
```

و در حالت معمول، از:

```cpp
new
delete
```

به‌صورت مستقیم استفاده نمی‌شود. این رویکرد با فلسفه‌ی RAII و Smart Pointerهای استاندارد C++ هماهنگ است.

---

# مهم ترین تفاوت دستورات `new` و `make_unique`

فرض کنید دو متد با ورودی‌های متفاوت داریم:

```cpp
void process(User* user);                    // Raw Pointer
void process(std::unique_ptr<User> user);    // unique_ptr
```

اگر با `new` فراخوانی کنیم:

```cpp
process(new User());
```

متد با ورودی Raw Pointer یعنی با ورودی `User*` انتخاب می‌شود.

اما اگر با `make_unique` فراخوانی کنیم:

```cpp
process(std::make_unique<User>());
```

متد `std::unique_ptr<User>` انتخاب می‌شود.

**نکته:** نوع عبارت ورودی، مشخص می‌کند کدام overload انتخاب شود.

---

# یک مثال کامل

```cpp
#include <iostream>
#include <memory>
#include <string>

class User
{
public:
    User(
        std::string name,
        int age
    )
        : name(std::move(name)),
          age(age)
    {
        std::cout << "User created\n";
    }

    ~User()
    {
        std::cout << "User destroyed\n";
    }

    void print() const
    {
        std::cout
            << name
            << " - "
            << age
            << '\n';
    }

private:
    std::string name;
    int age;
};

std::unique_ptr<User> createUser()
{
    return std::make_unique<User>(
        "Ali",
        30
    );
}

int main()
{
    auto user = createUser();

    if (user)
    {
        user->print();
    }

    auto another =
        std::make_unique<User>(
            "Reza",
            25
        );

    user.swap(another);
}
```

در این مثال:

```cpp
std::make_unique<User>()
```

Object را می‌سازد.

```cpp
std::unique_ptr<User>
```

مالکیت را مدیریت می‌کند.

```cpp
user->print();
```

به Object دسترسی پیدا می‌کند.

```cpp
user.swap(another);
```

Ownership را بین دو `unique_ptr` جابه‌جا می‌کند.

و در نهایت با خروج از Scope، Objectها به‌صورت خودکار Destroy می‌شوند.

---

# یک تصویر ذهنی بسیار مهم

اگر بخواهیم کل موضوع را در یک تصویر ذهنی خلاصه کنیم:

### روش قدیمی و دستی

```text
new
 │
 ▼
Object
 │
 ▼
Raw Pointer
 │
 │  باید مراقب باشیم
 │
 ▼
delete
```

مسئولیت Lifetime بیشتر روی دوش برنامه‌نویس است.

---

### روش C++ مدرن

```text
make_unique<T>()
       │
       ▼
 Dynamic Object
       ▲
       │
       │ ownership
       │
 unique_ptr
       │
       ▼
 Scope ends
       │
       ▼
automatic destruction
```

یعنی:

> **`make_unique` در اصل راه استاندارد و تمیز برای ساخت یک Object با مالکیت انحصاری در Dynamic Storage است.**

---

# خلاصه‌ و جمع بندی

`std::make_unique` را نباید صرفاً یک «جایگزین کوتاه‌تر برای `new`» دانست.

مفهوم اصلی آن این است:

```text
Object Creation
      +
Ownership
      +
RAII
      +
Exception Safety
```

در یک الگوی واحد.

به همین دلیل:

```cpp
std::unique_ptr<User>(
    new User(...)
);
```

از نظر فنی معتبر است و حتی مدیریت اتوماتیک آن توسط Unique_Ptr انجام می شود و مشکلی ندارد، اما در C++ مدرن معمولاً ترجیح داده می‌شود:

```cpp
std::make_unique<User>(...);
```

استفاده شود و نکته‌ی بسیار مهم این است که `make_unique` الزاماً باعث **Performance بهتر** نسبت به `unique_ptr(new T)` نمی‌شود؛ مزیت اصلی آن **Safety، Ownership شفاف‌تر، خوانایی بهتر و کاهش خطاهای مدیریت حافظه** است.

همچنین باید بین این سه مفهوم تفاوت گذاشته شود:

```cpp
new T
```

یعنی:

> Object را در Dynamic Storage بساز و Raw Pointer بده.

```cpp
std::unique_ptr<T>(new T)
```

یعنی:

> Object را با `new` بساز و مالکیتش را به `unique_ptr` بده.

```cpp
std::make_unique<T>()
```

یعنی:

> Object را بساز و از همان ابتدا آن را به شکل استاندارد و RAII-safe در یک `unique_ptr` قرار بده.

در نتیجه، در اکثر کدهای معمول C++ مدرن، اگر **مالکیت انحصاری و Dynamic Lifetime** لازم باشد، انتخاب طبیعی:

```cpp
auto object = std::make_unique<T>(...);
```

است.

---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>