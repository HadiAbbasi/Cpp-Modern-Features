<div align="center">

[🇺🇸 English](../../en/cpp11/default_delete.md) | [🇮🇷 فارسی](./default_delete.md)

</div>

---

# معرفی default و delete در ++C11

در ++C11 دو قابلیت مهم با نام‌های `default` و `delete` به زبان اضافه شدند که کنترل دقیق‌تری روی special member functions و رفتار کلاس‌ها فراهم می‌کنند. این قابلیت‌ها به‌خصوص برای طراحی classهای امن، قابل‌پیش‌بینی و سازگار با Rule of Zero، Rule of Five و Rule of Three اهمیت زیادی دارند.

مفهوم اصلی این است که برنامه‌نویس می‌تواند به‌جای تکیه بر رفتار ضمنی compiler، به‌صورت صریح مشخص کند که یک special member function باید با پیاده‌سازی پیش‌فرض تولید شود یا اساساً نباید وجود داشته باشد.

---

## مفهوم کلی default و delete

قابلیت `default` به compiler می‌گوید که برای یک special member function از پیاده‌سازی پیش‌فرض استفاده کند.

قابلیت `delete` به compiler می‌گوید که آن function وجود داشته باشد، اما استفاده از آن مجاز نباشد.

نمونه ساده:

```cpp
class User {
public:
    User() = default;
    User(const User&) = delete;
};
```

در این مثال، constructor پیش‌فرض به‌صورت معمول توسط compiler تولید می‌شود، اما copy constructor عمداً حذف شده است.

بنابراین کد زیر معتبر است:

```cpp
User user;
```

اما کد زیر خطای compile-time ایجاد می‌کند:

```cpp
User first;
User second = first;
```

این روش نسبت به تکنیک‌های قدیمی مانند private کردن یک function، شفاف‌تر و type-safeتر است.

---

## معرفی special member functions

برای درک درست `default` و `delete` ابتدا باید special member functions را بشناسیم.

در ++C معمولاً شش function زیر در این دسته قرار می‌گیرند:

* default constructor
* destructor
* copy constructor
* copy assignment operator
* move constructor
* move assignment operator

یک کلاس می‌تواند این functionها را خودش تعریف کند یا در شرایط مناسب اجازه دهد compiler آن‌ها را به‌صورت implicitly declare یا implicitly define کند.

برای نمونه:

```cpp
class Point {
public:
    int x;
    int y;
};
```

کلاس `Point` می‌تواند به‌صورت ضمنی special member functions مناسبی داشته باشد، بدون اینکه برنامه‌نویس آن‌ها را صریحاً بنویسد.

---

## معرفی default constructor با default

یکی از کاربردهای رایج `default` تعریف صریح default constructor است.

```cpp
class Person {
public:
    Person() = default;

private:
    std::string name;
    int age;
};
```

در این حالت، `Person()` یک default constructor است که compiler آن را به‌صورت پیش‌فرض تولید می‌کند.

نکته مهم این است که `= default` به معنی مقداردهی دستی اعضای کلاس نیست.

برای مثال:

```cpp
class Data {
public:
    Data() = default;

private:
    int value;
    std::string name;
};
```

عضو `name` با constructor خودش ساخته می‌شود، اما `value` در صورت default-initialization شدن شیء، به‌طور خودکار مقدار مشخصی مانند `0` دریافت نمی‌کند.

برای مثال:

```cpp
Data data;
```

در این وضعیت، مقدار `value` برای یک عضو built-in که initializer ندارد، مشخص نیست.

اگر مقداردهی مشخصی لازم است، بهتر است از member initializer استفاده شود:

```cpp
class Data {
public:
    Data()
        : value(0),
          name("unknown") {
    }

private:
    int value;
    std::string name;
};
```

در ++C11 می‌توان این کار را تمیزتر نیز انجام داد:

```cpp
class Data {
public:
    Data() = default;

private:
    int value = 0;
    std::string name = "unknown";
};
```

در این حالت member initializer مربوط به خود اعضا، رفتار مورد انتظار را مشخص می‌کند.

---

## بررسی کلاس دارای propertyهای متفاوت

یکی از موارد مهم، کلاس‌هایی است که propertyهای آن‌ها typeهای متفاوتی دارند.

برای مثال:

```cpp
#include <string>
#include <vector>

class User {
public:
    User() = default;

private:
    int id;
    double balance;
    bool active;
    std::string name;
    std::vector<int> permissions;
};
```

در این کلاس چند نوع مختلف وجود دارد:

* `int`
* `double`
* `bool`
* `std::string`
* `std::vector<int>`

اگر constructor به‌صورت `= default` تعریف شود، compiler تلاش می‌کند هر member را مطابق قواعد initialization همان type بسازد.

برای class-typeهایی مانند `std::string` و `std::vector<int>`، constructor پیش‌فرض آن type فراخوانی می‌شود.

اما برای typeهای primitive مانند `int`، `double` و `bool`، نباید تصور کرد که `= default` آن‌ها را حتماً به صفر تبدیل می‌کند.

راه مناسب برای تعیین مقدار پیش‌فرض propertyها استفاده از in-class member initializer است:

```cpp
#include <string>
#include <vector>

class User {
public:
    User() = default;

private:
    int id = 0;
    double balance = 0.0;
    bool active = false;
    std::string name = "unknown";
    std::vector<int> permissions;
};
```

اکنون رفتار کلاس واضح‌تر و قابل‌پیش‌بینی‌تر است.

---

## تفاوت default constructor و default member initialization

دو مفهوم زیر نباید با یکدیگر اشتباه گرفته شوند.

```cpp
class Example {
public:
    Example() = default;

private:
    int value = 42;
};
```

در اینجا `Example() = default` می‌گوید default constructor را compiler تولید کند.

اما `value = 42` می‌گوید member `value` هنگام initialization مناسب، مقدار پیش‌فرض `42` داشته باشد.

این دو قابلیت مستقل هستند.

به بیان ساده، `= default` درباره **function** است، در حالی که in-class initializer درباره **member initialization** است.

---

## مقایسه default constructor در حالت‌های مختلف

سه حالت زیر رفتار متفاوتی دارند.

### حالت تعریف بدون constructor

```cpp
class A {
private:
    int value = 10;
};
```

در این حالت compiler در صورت برقرار بودن شرایط، default constructor را به‌صورت implicit در اختیار کلاس قرار می‌دهد.

### حالت default کردن صریح constructor

```cpp
class B {
public:
    B() = default;

private:
    int value = 10;
};
```

در اینجا قصد برنامه‌نویس صریح است و default constructor به‌صورت explicitly defaulted تعریف شده است.

### حالت پیاده‌سازی دستی

```cpp
class C {
public:
    C()
        : value(10) {
    }

private:
    int value;
};
```

در این حالت constructor واقعاً توسط برنامه‌نویس تعریف شده است.

اگر رفتار موردنیاز دقیقاً همان رفتار compiler باشد، معمولاً `= default` انتخاب ساده‌تر و شفاف‌تری است.

---

## مفهوم delete

کلمه کلیدی `delete` در ++C11 امکان حذف کردن یک function را فراهم می‌کند.

برای مثال:

```cpp
class NonCopyable {
public:
    NonCopyable() = default;

    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
};
```

این کلاس قابل copy شدن نیست.

کد زیر خطا می‌دهد:

```cpp
NonCopyable first;
NonCopyable second(first);
```

و همچنین:

```cpp
NonCopyable first;
NonCopyable second;

second = first;
```

این ویژگی برای کلاس‌هایی که مالک resource هستند یا identity آن‌ها نباید duplicate شود، بسیار مفید است.

---

## کاربرد delete برای copy کردن

یکی از رایج‌ترین کاربردهای `delete` جلوگیری از copy است.

```cpp
class File {
public:
    File() = default;

    File(const File&) = delete;
    File& operator=(const File&) = delete;
};
```

فرض کنید هر شیء `File` مالک یک operating-system resource باشد. Copy کردن ساده آن ممکن است باعث double ownership یا double close شود.

در چنین شرایطی حذف copy operations می‌تواند از یک class طراحی خطرناک جلوگیری کند.

---

## کاربرد delete برای move کردن

`delete` فقط مخصوص copy نیست.

می‌توان move operations را نیز حذف کرد:

```cpp
class FixedObject {
public:
    FixedObject() = default;

    FixedObject(FixedObject&&) = delete;
    FixedObject& operator=(FixedObject&&) = delete;
};
```

در این صورت object قابل move شدن نیست.

با این حال، حذف move constructor معمولاً باید با دلیل طراحی مشخصی انجام شود؛ زیرا در بسیاری از classهای resource-owning، move semantics راه‌حل بهتری از جلوگیری کامل از انتقال ownership است.

---

## کاربرد delete برای جلوگیری از تبدیل ناخواسته

یکی از کاربردهای قدرتمند `delete` حذف overloadهایی است که نباید قابل استفاده باشند.

برای مثال:

```cpp
class Number {
public:
    Number(int value) : value_(value) {}

    Number(double) = delete;

private:
    int value_;
};
```

در اینجا ساختن object با `int` مجاز است:

```cpp
Number a(10);
```

اما استفاده از `double` ممنوع است:

```cpp
Number b(10.5);
```

این روش برای جلوگیری از implicit conversionهای ناخواسته بسیار مفید است.

---

## کاربرد delete در overloadها

می‌توان یک overload خاص را حذف کرد و overloadهای دیگر را نگه داشت.

```cpp
void process(int value) {
}

void process(double) = delete;
```

اکنون:

```cpp
process(10);
```

مجاز است، اما:

```cpp
process(10.5);
```

خطای compile-time ایجاد می‌کند.

این تکنیک می‌تواند interface یک API را دقیق‌تر کند.

---

## مقایسه delete با private کردن function

قبل از ++C11 یکی از روش‌های رایج برای جلوگیری از copy این بود که copy constructor و copy assignment را private کنیم.

نمونه قدیمی:

```cpp
class NonCopyable {
private:
    NonCopyable(const NonCopyable&);
    NonCopyable& operator=(const NonCopyable&);

public:
    NonCopyable() {}
};
```

این روش چند مشکل دارد.

اول اینکه intent برنامه‌نویس به‌وضوح `delete` نیست.

دوم اینکه ممکن است خطای دسترسی (`access control`) دریافت شود، نه خطای مشخص مربوط به حذف شدن function.

در ++C11 روش استاندارد این است:

```cpp
class NonCopyable {
public:
    NonCopyable() = default;

    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
};
```

بنابراین در کد ++C11 و نسخه‌های جدیدتر، استفاده از `= delete` برای این هدف روش ترجیحی است.

---

## تفاوت default با implementation دستی

این دو declaration ظاهراً ممکن است مشابه به نظر برسند:

```cpp
class Example {
public:
    Example() = default;
};
```

و:

```cpp
class Example {
public:
    Example() {
    }
};
```

اما از دید زبان ++C کاملاً یکسان نیستند.

`= default` به compiler اجازه می‌دهد special member function را به‌عنوان یک defaulted function مدیریت کند.

در مقابل، body خالی یعنی function به‌صورت user-provided تعریف شده است.

این تفاوت می‌تواند روی ویژگی‌های type و بعضی type traits و همچنین شرایط تولید implicit special member functions تأثیر بگذارد.

بنابراین اگر واقعاً رفتار پیش‌فرض compiler را می‌خواهید، بهتر است همان intent را با `= default` بیان کنید.

---

## تفاوت implicitly declared و explicitly defaulted

این دو مفهوم نیز باید از هم جدا شوند.

در حالت implicit:

```cpp
class User {
private:
    int id = 0;
};
```

برنامه‌نویس constructor را ننوشته است و compiler در شرایط لازم آن را implicitly declare می‌کند.

در حالت explicitly defaulted:

```cpp
class User {
public:
    User() = default;

private:
    int id = 0;
};
```

برنامه‌نویس صراحتاً از compiler خواسته است constructor را default کند.

این explicit declaration می‌تواند برای بیان intent طراحی class بسیار ارزشمند باشد.

---

## نکته مهم درباره defaulted functionهای حذف‌شده

یک function که با `= default` تعریف شده است همیشه الزاماً قابل استفاده نیست.

اگر ساختن یکی از memberهای class ممکن نباشد، defaulted function ممکن است توسط compiler به‌عنوان `deleted` تعریف شود.

برای مثال:

```cpp
class Member {
public:
    Member() = delete;
};

class Container {
public:
    Container() = default;

private:
    Member member;
};
```

در اینجا `Container()` نمی‌تواند member خود را بسازد، بنابراین default constructor مربوط به `Container` نیز عملاً قابل استفاده نیست.

در نتیجه این کد خطا خواهد داشت:

```cpp
Container object;
```

این موضوع اهمیت composition را در رفتار special member functions نشان می‌دهد.

---

## اثر نوع propertyها بر default constructor

نوع memberها مستقیماً روی امکان تولید default constructor اثر می‌گذارد.

برای مثال:

```cpp
class A {
private:
    int value;
};
```

ساختن default constructor مشکلی ندارد.

اما:

```cpp
class B {
public:
    B() = delete;
};

class A {
public:
    A() = default;

private:
    B value;
};
```

در این حالت `A` نمی‌تواند default construct شود، زیرا `B` قابل default construction نیست.

بنابراین برای کلاسی که propertyهای مختلف دارد، باید constructibility تمام memberها را نیز در نظر گرفت.

---

## طراحی کلاس دارای propertyهای مختلف

برای یک مدل ساده مانند `User` معمولاً طراحی زیر خوانا و قابل‌اعتماد است:

```cpp
#include <string>
#include <vector>

class User {
public:
    User() = default;

private:
    int id = 0;
    std::string username;
    double balance = 0.0;
    bool active = true;
    std::vector<int> roles;
};
```

در این مثال:

* `id` مقدار اولیه مشخص دارد.
* `username` با constructor پیش‌فرض `std::string` ساخته می‌شود.
* `balance` برابر `0.0` است.
* `active` برابر `true` است.
* `roles` به‌صورت خالی ساخته می‌شود.
* constructor اصلی به compiler سپرده شده است.

اگر کلاس هیچ invariant پیچیده‌ای ندارد، این طراحی معمولاً از نوشتن یک constructor طولانی بهتر است.

---

## زمانی که default constructor مناسب نیست

نباید صرفاً برای اینکه کلاس constructor داشته باشد، `= default` اضافه کرد.

اگر object بدون داده ضروری معنای معتبری ندارد، بهتر است default construction را اصلاً مجاز نکنیم.

برای مثال:

```cpp
class UserId {
public:
    explicit UserId(int value)
        : value_(value) {
    }

private:
    int value_;
};
```

در این طراحی، ساختن `UserId` بدون مقدار مشخص منطقی نیست.

می‌توانیم صریحاً نیز آن را حذف کنیم:

```cpp
class UserId {
public:
    UserId() = delete;

    explicit UserId(int value)
        : value_(value) {
    }

private:
    int value_;
};
```

اکنون:

```cpp
UserId id(42);
```

مجاز است، اما:

```cpp
UserId id;
```

غیرمجاز است.

---

## کاربرد default و delete در Rule of Zero

یکی از اصول مهم طراحی مدرن ++C، Rule of Zero است.

اگر class از resourceهای خودکار مانند:

* `std::string`
* `std::vector`
* `std::unique_ptr`
* `std::shared_ptr`
* سایر RAII typeها

استفاده کند، معمولاً نباید special member functions را دستی مدیریت کنیم.

برای مثال:

```cpp
#include <memory>
#include <string>

class User {
public:
    User() = default;

private:
    std::string name;
    std::unique_ptr<int> data;
};
```

در اینجا ownership توسط `std::unique_ptr` مدیریت می‌شود و لازم نیست destructor یا move constructor را دستی بنویسیم.

البته وجود `std::unique_ptr` باعث می‌شود copy operations به‌صورت پیش‌فرض قابل تولید نباشند.

اگر بخواهیم این intent را صریح کنیم، می‌توانیم copy operations را حذف کنیم:

```cpp
class User {
public:
    User() = default;

    User(const User&) = delete;
    User& operator=(const User&) = delete;

private:
    std::unique_ptr<int> data;
};
```

با این حال، اگر خود memberها همین رفتار را به‌درستی تحمیل می‌کنند، در بسیاری از طراحی‌ها نیازی به تکرار `= delete` نیست.

---

## ارتباط default و delete با Rule of Five

برای classهایی که resource را به‌صورت دستی مدیریت می‌کنند، Rule of Five اهمیت پیدا می‌کند.

این پنج function عبارت‌اند از:

* destructor
* copy constructor
* copy assignment operator
* move constructor
* move assignment operator

برای نمونه:

```cpp
class Resource {
public:
    Resource() = default;

    Resource(const Resource&) = delete;
    Resource& operator=(const Resource&) = delete;

    Resource(Resource&&) = default;
    Resource& operator=(Resource&&) = default;

    ~Resource() = default;
};
```

این class قابل move شدن است اما قابل copy شدن نیست.

با این حال، استفاده از این الگو باید بر اساس semantics واقعی resource باشد، نه صرفاً به‌عنوان یک template ثابت.

---

## تفاوت حذف copy با پشتیبانی از move

این دو تصمیم یکی نیستند.

یک class ممکن است:

* copyable و movable باشد؛
* فقط movable باشد؛
* نه copyable باشد و نه movable.

برای نمونه، `std::unique_ptr` تقریباً الگوی معروف نوع دوم است.

اگر class شما مالک resource انحصاری است، طراحی معمولاً به این شکل است:

```cpp
class ResourceOwner {
public:
    ResourceOwner() = default;

    ResourceOwner(const ResourceOwner&) = delete;
    ResourceOwner& operator=(const ResourceOwner&) = delete;

    ResourceOwner(ResourceOwner&&) = default;
    ResourceOwner& operator=(ResourceOwner&&) = default;
};
```

در این طراحی ownership می‌تواند منتقل شود، اما نمی‌تواند duplicate شود.

---

## نکته مهم درباره destructor و move operations

تعریف بعضی special member functions می‌تواند روی implicit generation سایر special member functions اثر بگذارد.

به‌خصوص تعریف destructor، copy constructor، copy assignment، move constructor یا move assignment می‌تواند شرایط تولید implicit move operations را تغییر دهد.

به همین دلیل، در طراحی مدرن بهتر است ابتدا بررسی کنیم آیا واقعاً به special member function دستی نیاز داریم یا خیر.

اگر resource با RAII typeهای استاندارد مدیریت می‌شود، Rule of Zero معمولاً انتخاب بهتری است.

---

## تفاوت default با مقداردهی پیش‌فرض propertyها

یک اشتباه رایج این است که تصور شود کد زیر همه propertyها را صفر می‌کند:

```cpp
class Data {
public:
    Data() = default;

    int count;
    double price;
    bool enabled;
};
```

چنین تضمینی وجود ندارد.

برای مقداردهی مشخص، باید initializer ارائه شود:

```cpp
class Data {
public:
    Data() = default;

    int count = 0;
    double price = 0.0;
    bool enabled = false;
};
```

در این حالت intent کاملاً روشن است.

---

## مقایسه مقداردهی اعضای مختلف

برای propertyهای class-type معمولاً constructor خود type اجرا می‌شود:

```cpp
class Example {
public:
    Example() = default;

private:
    std::string text;
};
```

در مقابل، برای scalar typeها مانند `int` و `double`، اگر initializer نداشته باشند، default initialization می‌تواند مقدار نامشخص ایجاد کند.

بنابراین این دو مورد را باید جداگانه در نظر گرفت:

```cpp
std::string text;
int number;
```

نوع اول constructor دارد، اما نوع دوم یک built-in type است و خودش مقدار اولیه مشخصی ایجاد نمی‌کند.

---

## کاربرد delete برای جلوگیری از constructor ناخواسته

گاهی یک constructor ممکن است از طریق conversionهای implicit ناخواسته در دسترس قرار گیرد.

برای مثال:

```cpp
class Identifier {
public:
    Identifier(int value)
        : value_(value) {
    }

    Identifier(double) = delete;

private:
    int value_;
};
```

اکنون API صریح‌تر است و compiler اجازه نمی‌دهد `double` به‌عنوان identifier استفاده شود.

این تکنیک به‌خصوص در domain types و APIهای حساس می‌تواند مفید باشد.

---

## محدودیت delete

تابع حذف‌شده همچنان بخشی از interface کلاس است، اما نمی‌توان آن را فراخوانی کرد.

برای مثال:

```cpp
class Example {
public:
    Example() = default;
    Example(const Example&) = delete;
};
```

وجود copy constructor حذف‌شده به این معنا نیست که copy constructor کاملاً از declarationهای class ناپدید شده است؛ بلکه استفاده از آن ill-formed است.

این تفاوت در overload resolution نیز اهمیت دارد، زیرا deleted function می‌تواند در انتخاب overload دخالت داشته باشد و سپس باعث خطای compile-time شود.

---

## تفاوت delete با عدم declaration

این دو حالت یکسان نیستند:

```cpp
class A {
public:
    A() = delete;
};
```

و:

```cpp
class B {
};
```

در حالت اول، برنامه‌نویس صریحاً اعلام کرده که default constructor نباید استفاده شود.

در حالت دوم، رفتار special member functions بر اساس قواعد implicit generation زبان تعیین می‌شود.

بنابراین `delete` ابزاری برای بیان explicit design intent است.

---

## اشتباه رایج در کلاس‌های دارای property

گاهی برنامه‌نویس برای جلوگیری از مقدار نامشخص، constructor دستی می‌نویسد:

```cpp
class User {
public:
    User()
        : id(0),
          balance(0.0),
          active(false),
          name("unknown") {
    }

private:
    int id;
    double balance;
    bool active;
    std::string name;
};
```

این کد معتبر است، اما در ++C11 می‌توان از in-class initializer استفاده کرد:

```cpp
class User {
public:
    User() = default;

private:
    int id = 0;
    double balance = 0.0;
    bool active = false;
    std::string name = "unknown";
};
```

این نسخه معمولاً خواناتر است، مخصوصاً زمانی که این مقادیر بخشی از default state کلاس هستند.

---

## زمانی که constructor باید invariant را تضمین کند

اگر object فقط با مجموعه‌ای از شرایط مشخص معتبر باشد، `= default` ممکن است انتخاب مناسبی نباشد.

برای مثال:

```cpp
class Temperature {
public:
    explicit Temperature(double value)
        : value_(value) {
        if (value < -273.15) {
            throw std::invalid_argument("Invalid temperature");
        }
    }

private:
    double value_;
};
```

در چنین طراحی‌ای، default construction ممکن است state نامعتبر ایجاد کند.

بنابراین حذف آن می‌تواند منطقی باشد:

```cpp
class Temperature {
public:
    Temperature() = delete;

    explicit Temperature(double value)
        : value_(value) {
        if (value < -273.15) {
            throw std::invalid_argument("Invalid temperature");
        }
    }

private:
    double value_;
};
```

در این حالت، class مجبور می‌کند object با یک مقدار معتبر ساخته شود.

---

## نکات مهم در طراحی حرفه‌ای

در طراحی class، `= default` را زمانی استفاده کنید که semantics پیش‌فرض compiler دقیقاً همان چیزی است که می‌خواهید.

برای جلوگیری از copy یا move ناخواسته، `= delete` را به‌عنوان ابزار صریح طراحی در نظر بگیرید.

برای propertyهای primitive که باید مقدار اولیه مشخص داشته باشند، از in-class member initializer استفاده کنید.

برای resource management ترجیحاً از RAII و standard library typeها استفاده کنید تا نیاز به مدیریت دستی special member functions کاهش یابد.

اگر class بدون مقداردهی اولیه معنای معتبری ندارد، به‌جای ساختن یک default state مصنوعی، default constructor را حذف کنید.

همچنین نباید همه special member functions را صرفاً برای «صریح بودن» با `= default` بنویسید. کد باید intent واقعی طراحی را بیان کند و از تکرار غیرضروری جلوگیری شود.

---

## جمع‌بندی

در ++C11، `= default` و `= delete` ابزارهای مهمی برای کنترل رفتار special member functions و overloadها هستند.

قابلیت `= default` زمانی مفید است که بخواهیم رفتار پیش‌فرض compiler را صریحاً درخواست کنیم:

```cpp
class Example {
public:
    Example() = default;
};
```

قابلیت `= delete` زمانی استفاده می‌شود که بخواهیم استفاده از یک function را ممنوع کنیم:

```cpp
class Example {
public:
    Example() = default;
    Example(const Example&) = delete;
};
```

برای کلاس‌هایی که propertyهای مختلفی دارند، باید تفاوت initialization در class-typeها و built-in typeها را در نظر گرفت. استفاده از in-class member initializer در ++C11 راه مناسبی برای تعیین default state است:

```cpp
class User {
public:
    User() = default;

private:
    int id = 0;
    double balance = 0.0;
    bool active = false;
    std::string name = "unknown";
};
```

در نهایت، بهترین طراحی معمولاً این است که تا حد امکان به Rule of Zero نزدیک بمانیم، ownership را با RAII typeهای استاندارد مدیریت کنیم و فقط زمانی از `= default` یا `= delete` استفاده کنیم که بیان صریح semantics کلاس واقعاً ارزش داشته باشد.

---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>