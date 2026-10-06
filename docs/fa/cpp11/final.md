<div align="center">

[🇺🇸 English](../../en/cpp11/final.md) | [🇮🇷 فارسی](./final.md)

</div>

---

# معرفی `final` در ++C11

در ++C11 واژهٔ کلیدی `final` برای مشخص‌کردن محدودیت‌های inheritance و جلوگیری از override شدن virtual functions معرفی شد. این قابلیت بخشی از زبان ++C است و به Standard Library وابسته نیست.

قابلیت `final` در دو موقعیت اصلی استفاده می‌شود: می‌توان یک `class` را به‌عنوان آخرین کلاس در یک سلسله‌مراتب inheritance علامت‌گذاری کرد، یا یک `virtual function` را به‌گونه‌ای مشخص کرد که کلاس‌های مشتق‌شده دیگر نتوانند آن را override کنند.

استفادهٔ درست از `final` می‌تواند قراردادهای طراحی کلاس را صریح‌تر کند و از inheritance یا override ناخواسته جلوگیری کند.

## مفهوم `final` در ++C11

در ++C11، `final` یک **contextual keyword** است؛ یعنی برخلاف keywordهای معمول، در برخی contextهای مشخص معنای ویژه پیدا می‌کند.

دو کاربرد اصلی آن عبارت‌اند از:

* جلوگیری از override شدن یک `virtual function`
* جلوگیری از مشتق شدن کلاس‌های دیگر از یک `class`

نمونهٔ سادهٔ استفاده از آن به شکل زیر است:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() final;
};
```

در این مثال، تابع `()process` در `Derived` آخرین override در این سلسله‌مراتب است. بنابراین کلاس دیگری نمی‌تواند آن را دوباره override کند.

کاربرد دوم برای کلاس است:

```cpp
class FinalClass final {
public:
    void process();
};
```

در این حالت هیچ کلاس دیگری نمی‌تواند از `FinalClass` مشتق شود.

## دلیل وجود `final`

قابلیت inheritance یکی از ویژگی‌های مهم ++C است، اما inheritance همیشه مطلوب نیست. گاهی طراحی یک کلاس به‌گونه‌ای است که override کردن یک تابع خاص یا مشتق شدن از یک کلاس می‌تواند invariantهای داخلی، قراردادهای API یا رفتار مورد انتظار را نقض کند.

برای نمونه، فرض کنید یک کلاس الگوریتمی دارد که ترتیب اجرای چند مرحله برای آن اهمیت دارد:

```cpp
class Processor {
public:
    virtual void start();
    virtual void execute();
    virtual void finish();
};
```

ممکن است طراحی کلاس ایجاب کند که `finish` دیگر قابل تغییر نباشد. در این شرایط می‌توان از `final` استفاده کرد:

```cpp
class SafeProcessor : public Processor {
public:
    void finish() final;
};
```

اکنون compiler هر تلاش بعدی برای override کردن `finish` را خطا اعلام می‌کند.

این ویژگی به جای اتکا به مستندات یا قراردادهای شفاهی، محدودیت طراحی را مستقیماً در type system و قواعد زبان ++C ثبت می‌کند.

## جلوگیری از Override با `final`

برای جلوگیری از override شدن یک `virtual function`، باید `final` بعد از declaration یا definition تابع قرار بگیرد:

```cpp
class Base {
public:
    virtual void run();
};

class Derived : public Base {
public:
    void run() final;
};
```

در این مثال، `run` همچنان یک virtual function است، اما آخرین override مجاز در سلسله‌مراتب محسوب می‌شود.

اگر کلاس دیگری از `Derived` مشتق شود، نمی‌تواند `run` را override کند:

```cpp
class MoreDerived : public Derived {
public:
    void run() override;
};
```

کد بالا باید توسط compiler رد شود، زیرا `run` در کلاس پایهٔ مستقیم با `final` مشخص شده است.

## ترکیب `final` و `override`

ترکیب `override` و `final` یکی از رایج‌ترین و مفیدترین کاربردهای این قابلیت است:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() override final;
};
```

در اینجا دو قرارداد مستقل بیان می‌شوند:

* `override` تضمین می‌کند که `process` واقعاً یک virtual function از کلاس پایه را override می‌کند.
* `final` تضمین می‌کند که کلاس‌های مشتق‌شده دیگر نمی‌توانند آن override را تغییر دهند.

استفاده از هر دو keyword معمولاً از استفادهٔ تنها از `final` دقیق‌تر است، زیرا اگر امضای تابع اشتباه باشد، `override` خطا را آشکار می‌کند.

برای مثال:

```cpp
class Base {
public:
    virtual void process(int value);
};

class Derived : public Base {
public:
    void process() final;
};
```

این کد معتبر نیست، زیرا `process()` تابعی با امضای متفاوت است و تابع پایه را override نمی‌کند.

در نتیجه، `override` یک لایهٔ مهم از بررسی compiler را فراهم می‌کند.

## تفاوت `final` و `override`

این دو keyword وظایف متفاوتی دارند.

| ویژگی                    | `override` | `final`                           |
| ------------------------ | ---------- | --------------------------------- |
| بررسی override شدن تابع  | بله        | بله، در صورت استفاده روی override |
| جلوگیری از override بعدی | خیر        | بله                               |
| جلوگیری از inheritance   | خیر        | بله، برای class                   |
| معرفی‌شده در             | ++C11      | ++C11                             |

به‌صورت مفهومی، `override` دربارهٔ **درستی رابطهٔ override** صحبت می‌کند، در حالی که `final` دربارهٔ **پایان دادن به امکان تغییر آن رابطه** صحبت می‌کند.

نمونهٔ زیر هر دو مفهوم را نشان می‌دهد:

```cpp
class Base {
public:
    virtual void draw();
};

class Button : public Base {
public:
    void draw() override final;
};
```

در اینجا `Button` اجازه دارد `draw` را override کند، اما کلاس‌های بعدی اجازه ندارند override دیگری برای آن ارائه دهند.

## جلوگیری از Inheritance با `final`

برای جلوگیری از مشتق شدن کلاس‌های دیگر، `final` پس از نام کلاس قرار می‌گیرد:

```cpp
class NetworkConnection final {
public:
    void connect();
};
```

اکنون چنین کدی غیرمجاز است:

```cpp
class SecureConnection : public NetworkConnection {
};
```

compiler باید خطایی مشابه تلاش برای inheritance از یک `final class` گزارش کند.

این کاربرد زمانی مفید است که کلاس اساساً برای inheritance طراحی نشده باشد یا ایجاد subclass برای آن از نظر طراحی نامعتبر باشد.

## تفاوت `final class` با کلاس بدون Virtual Function

کلاسی که virtual function ندارد، لزوماً `final` نیست.

برای مثال:

```cpp
class Configuration {
public:
    void load();
};
```

کلاس دیگری همچنان می‌تواند از آن مشتق شود:

```cpp
class ApplicationConfiguration : public Configuration {
};
```

اگر هدف طراحی این باشد که هیچ subclassای وجود نداشته باشد، باید این محدودیت صریحاً بیان شود:

```cpp
class Configuration final {
public:
    void load();
};
```

بنابراین نبودن `virtual` به‌تنهایی inheritance را ممنوع نمی‌کند.

## تأثیر `final` بر Dynamic Dispatch

استفاده از `final` رفتار پایهٔ virtual dispatch را تغییر نمی‌دهد؛ بلکه امکان overrideهای بعدی را محدود می‌کند.

برای مثال:

```cpp
class Base {
public:
    virtual void run();
};

class Derived : public Base {
public:
    void run() final;
};
```

اگر یک reference یا pointer به `Base` به شیء `Derived` اشاره کند، همچنان dynamic dispatch انجام می‌شود:

```cpp
Base* object = new Derived;
object->run();
```

تابع `Derived::run` اجرا خواهد شد.

بنابراین `final` به معنای تبدیل خودکار virtual function به non-virtual function در semantics زبان نیست.

## تأثیر `final` بر Performance

گاهی `final` در بحث optimization مطرح می‌شود، زیرا compiler در برخی شرایط می‌تواند از محدود بودن مجموعهٔ overrideهای ممکن برای بهینه‌سازی استفاده کند.

برای مثال:

```cpp
class Base {
public:
    virtual void run();
};

class Derived final : public Base {
public:
    void run() override;
};
```

اگر compiler بتواند ثابت کند که نوع runtime فقط از مجموعهٔ محدودی از typeها تشکیل می‌شود، ممکن است بتواند dynamic dispatch را devirtualize کند.

بااین‌حال، نباید `final` را صرفاً به‌عنوان یک optimization keyword در نظر گرفت.

هدف اصلی `final` **بیان constraint طراحی و جلوگیری از override یا inheritance ناخواسته** است.

هرگونه بهبود performance وابسته به compiler، optimization level، context برنامه و اطلاعاتی است که compiler در اختیار دارد.

## تفاوت `final` با Non-Virtual کردن تابع

فرض کنید تابعی به شکل زیر تعریف شده است:

```cpp
class Base {
public:
    void process();
};
```

این تابع virtual نیست، بنابراین کلاس مشتق‌شده نمی‌تواند آن را به معنای polymorphic override بازتعریف کند.

اما ممکن است کلاس مشتق‌شده تابعی با همان نام تعریف کند:

```cpp
class Derived : public Base {
public:
    void process();
};
```

این رفتار **override** نیست؛ بلکه name hiding است.

در مقابل، در کد زیر:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() final;
};
```

تابع `Derived::process` واقعاً override است و `final` صراحتاً اجازهٔ override بعدی را می‌بندد.

این تفاوت برای درک صحیح `final` اهمیت زیادی دارد.

## استفاده از `final` در سلسله‌مراتب چندسطحی

محدودیت `final` از محل تعریف آن به پایین سلسله‌مراتب inheritance اعمال می‌شود.

برای مثال:

```cpp
class A {
public:
    virtual void process();
};

class B : public A {
public:
    void process() override;
};

class C : public B {
public:
    void process() final;
};

class D : public C {
public:
    void process() override;
};
```

کد بالا نامعتبر است، زیرا `C::process` با `final` مشخص شده است.

اما اگر `final` فقط در یک سطح خاص استفاده شده باشد، کلاس‌های قبل از آن همچنان محدودیت‌های معمول inheritance را دارند.

## استفاده از `final` روی Destructor

از نظر syntax، destructor نیز می‌تواند `final` باشد، زیرا destructor می‌تواند virtual باشد:

```cpp
class Base {
public:
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    ~Derived() final = default;
};
```

در این حالت کلاس‌های مشتق‌شده از `Derived` نمی‌توانند destructor خود را به‌عنوان override تعریف کنند.

این کاربرد بسیار تخصصی است و معمولاً برای طراحی عمومی کلاس‌ها ضرورتی ندارد.

اگر هدف فقط جلوگیری از inheritance است، معمولاً قرار دادن `final` روی خود کلاس شفاف‌تر است:

```cpp
class Derived final {
public:
    ~Derived() = default;
};
```

بنابراین `final` روی destructor باید با هدف مشخص استفاده شود، نه صرفاً به‌عنوان جایگزین `final class`.

## استفاده از `final` در APIهای Polymorphic

در طراحی APIهای polymorphic، می‌توان از `final` برای مشخص‌کردن مرزهای inheritance استفاده کرد.

برای مثال:

```cpp
class Document {
public:
    virtual ~Document() = default;
    virtual void save() = 0;
    virtual void validate() = 0;
};

class PdfDocument final : public Document {
public:
    void save() override;
    void validate() override;
};
```

در این طراحی، `Document` برای inheritance طراحی شده است، اما `PdfDocument` به‌عنوان یک implementation نهایی ارائه می‌شود.

این الگو زمانی مناسب است که API چند implementation داشته باشد، ولی برخی implementationها نباید دوباره تخصصی‌تر شوند.

## استفاده از `final` در Interfaceها

معمولاً interfaceهای polymorphic نباید `final` باشند، زیرا هدف آن‌ها فراهم کردن امکان implementation توسط کلاس‌های دیگر است.

برای مثال:

```cpp
class Renderer {
public:
    virtual ~Renderer() = default;
    virtual void render() = 0;
};
```

قرار دادن `final` روی `Renderer` با هدف چنین طراحی‌ای ناسازگار است:

```cpp
class Renderer final {
public:
    virtual void render() = 0;
};
```

در مقابل، implementation مشخص یک interface می‌تواند `final` باشد:

```cpp
class OpenGLRenderer final : public Renderer {
public:
    void render() override;
};
```

بنابراین باید `final` را بر اساس نقش کلاس در معماری استفاده کرد، نه به‌صورت مکانیکی.

## استفاده از `final` برای تثبیت Invariantها

یکی از دلایل مهم استفاده از `final` زمانی است که correctness کلاس به implementation خاصی وابسته باشد.

برای مثال:

```cpp
class Transaction {
public:
    virtual ~Transaction() = default;
    virtual void commit();
    virtual void rollback();
};

class AtomicTransaction final : public Transaction {
public:
    void commit() override;
    void rollback() override;
};
```

اگر منطق داخلی `AtomicTransaction` فرض کند که هیچ subclassای رفتار `commit` یا `rollback` را تغییر نمی‌دهد، `final` می‌تواند این قرارداد را enforce کند.

در چنین شرایطی `final` بخشی از طراحی correctness است، نه صرفاً یک ویژگی نحوی.

## اشتباه رایج در استفاده از `final`

یکی از اشتباهات رایج این است که `final` بدون وجود `virtual` استفاده شود.

برای مثال:

```cpp
class Base {
public:
    void process();
};

class Derived : public Base {
public:
    void process() final;
};
```

این کد صحیح نیست، زیرا `Derived::process` نمی‌تواند تابعی غیرvirtual را override کند.

اگر قصد استفاده از `final` روی تابع وجود دارد، تابع باید در زنجیرهٔ inheritance یک virtual function قابل override باشد.

استفاده از `override` نیز می‌تواند intent را واضح‌تر کند:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() override final;
};
```

## اشتباه رایج در ترتیب Keywordها

برای تابع می‌توان `override` و `final` را در declaration استفاده کرد:

```cpp
void process() override final;
```

یا:

```cpp
void process() final override;
```

هر دو از نظر ترتیب این دو specifier قابل استفاده‌اند.

بااین‌حال، استفاده از الگوی ثابت مانند `override final` معمولاً خوانایی کد را بیشتر می‌کند و بهتر است style پروژه یکدست باشد.

نکتهٔ مهم این است که `override` و `final` باید بعد از declarator تابع قرار بگیرند:

```cpp
void process() override final;
```

نه در قالبی که بخشی از نوع یا نام تابع تلقی شوند.

## اشتباه رایج در تصور `final` به‌عنوان Modifier دسترسی

قابلیت `final` هیچ ارتباطی با `public`، `protected` یا `private` ندارد.

برای مثال:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
private:
    void process() final;
};
```

در این مثال `process` هم `private` است و هم `final`.

این دو ویژگی مستقل از یکدیگر هستند:

* access control مشخص می‌کند چه کدی می‌تواند به عضو دسترسی داشته باشد.
* `final` مشخص می‌کند آیا override بعدی مجاز است یا خیر.

## اشتباه رایج در استفاده از `final` به جای Composition

`final` نباید به‌عنوان راهی برای اصلاح یک hierarchy نامناسب استفاده شود.

اگر یک کلاس فقط برای reuse کردن implementation از کلاس دیگری مشتق شده است، ممکن است composition طراحی مناسب‌تری باشد.

برای مثال، inheritance رابطهٔ «is-a» را بیان می‌کند، در حالی که composition معمولاً رابطهٔ «has-a» را مدل می‌کند.

استفاده از `final` نمی‌تواند یک inheritance hierarchy اشتباه را به hierarchy درست تبدیل کند.

اگر subclass هیچ رابطهٔ معنایی مناسبی با base class ندارد، بهتر است قبل از افزودن `final` خود طراحی بررسی شود.

## تفاوت `final` با `private`

گاهی برای جلوگیری از inheritance از تکنیک‌های قدیمی مانند constructorهای `private` یا `protected` استفاده می‌شد.

در ++C11، اگر هدف واقعاً ممنوع کردن inheritance است، `final` معمولاً بیان صریح‌تر و ساده‌تری دارد:

```cpp
class Utility final {
public:
    static void process();
};
```

در مقابل، `private` constructor مسئلهٔ متفاوتی را حل می‌کند و می‌تواند روی نحوهٔ ایجاد object تأثیر بگذارد.

بنابراین نباید این دو مفهوم را جایگزین یکدیگر دانست.

## تفاوت `final` با تکنیک‌های قدیمی جلوگیری از Inheritance

پیش از ++C11، برای جلوگیری از inheritance روش‌هایی مانند private constructor، private destructor یا الگوهای خاص طراحی استفاده می‌شدند.

این روش‌ها معمولاً هدف را به‌صورت غیرمستقیم بیان می‌کردند.

در ++C11 می‌توان intent را مستقیماً بیان کرد:

```cpp
class NonExtendable final {
};
```

مزیت اصلی این روش، وضوح و تشخیص خطا در زمان compile است.

همچنین `final` به compiler و سایر توسعه‌دهندگان نشان می‌دهد که ممنوع بودن inheritance یک تصمیم طراحی آگاهانه است.

## تفاوت `final` با `sealed` در زبان‌های دیگر

در برخی زبان‌های دیگر، مفاهیمی مانند `sealed` برای جلوگیری از inheritance وجود دارند.

در ++C11، keyword استاندارد مورد استفاده برای این منظور `final` است.

به همین دلیل، استفاده از اصطلاح `sealed class` در مستندات ++C ممکن است برای توضیح مفهوم مفید باشد، اما syntax استاندارد ++C11 این است:

```cpp
class Example final {
};
```

در ++C، `final` علاوه بر class روی virtual function نیز کاربرد دارد.

## محدودیت‌های `final`

قابلیت `final` تنها در contextهایی که Standard برای آن تعریف کرده است معنا دارد.

این قابلیت نمی‌تواند:

* object creation را محدود کند.
* دسترسی به اعضا را کنترل کند.
* thread safety ایجاد کند.
* object immutability ایجاد کند.
* lifetime object را مدیریت کند.
* جلوی composition را بگیرد.
* به‌تنهایی correctness یک hierarchy را تضمین کند.

به بیان دیگر، `final` یک ابزار محدود برای کنترل inheritance و virtual overriding است و نباید مسئولیت‌هایی فراتر از آن به آن نسبت داده شود.

## ملاحظات ABI و Library Design

در کتابخانه‌های عمومی، تصمیم برای `final` می‌تواند بخشی از API contract محسوب شود.

اگر یک کلاس به‌صورت `final` منتشر شود، کاربران کتابخانه نمی‌توانند از آن مشتق شوند. بنابراین برداشتن یا اضافه‌کردن `final` ممکن است بر قابلیت‌های قابل استفادهٔ کاربران تأثیر بگذارد.

برای مثال، API زیر:

```cpp
class LibraryComponent final {
public:
    virtual ~LibraryComponent() = default;
    virtual void process();
};
```

عملاً به کاربران اعلام می‌کند که این کلاس نقطهٔ پایانی hierarchy است.

بنابراین `final` باید در public API با دقت و بر اساس intent طراحی استفاده شود.

## ملاحظات طراحی در پروژه‌های بزرگ

در پروژه‌های بزرگ بهتر است `final` بر اساس قراردادهای واقعی طراحی استفاده شود، نه برای افزایش ظاهری محدودیت‌ها.

اگر یک کلاس قرار نیست برای inheritance استفاده شود، استفاده از `final` می‌تواند intent را واضح کند:

```cpp
class JsonParser final {
public:
    void parse();
};
```

اما اگر احتمال واقعی extension از طریق inheritance وجود دارد، قرار دادن `final` می‌تواند انعطاف API را غیرضروری محدود کند.

قاعدهٔ مناسب این است که `final` زمانی استفاده شود که **ممنوع بودن extension بخشی از طراحی باشد**.

## استفاده از `final` همراه با Abstract Base Class

یک الگوی متداول این است که abstract base class برای extension طراحی شود و implementationهای خاص با `final` پایان یابند:

```cpp
class Storage {
public:
    virtual ~Storage() = default;
    virtual void read() = 0;
    virtual void write() = 0;
};

class FileStorage final : public Storage {
public:
    void read() override;
    void write() override;
};
```

در این طراحی، `Storage` نقطهٔ extension است، اما `FileStorage` implementation نهایی محسوب می‌شود.

این الگو در معماری‌هایی که dependency inversion و polymorphism اهمیت دارند، کاربرد زیادی دارد.

## استفاده از `final` در Templateها

`final` را می‌توان روی class template نیز استفاده کرد:

```cpp
template <typename T>
class Container final {
public:
    void process(const T& value);
};
```

هر specialization حاصل از این template نیز یک `final class` است.

بنابراین استفاده از template بودن کلاس، محدودیت `final` را تغییر نمی‌دهد.

همچنین می‌توان یک specialization را به‌صورت معمولی تعریف کرد، ولی محدودیت inheritance همچنان مطابق declaration نهایی همان type خواهد بود.

## استفاده از `final` در Multiple Inheritance

قواعد `final` در multiple inheritance نیز اعمال می‌شوند.

برای مثال:

```cpp
class A {
public:
    virtual void process();
};

class B {
public:
    virtual void process();
};

class C : public A, public B {
public:
    void process() override final;
};
```

در این حالت `C::process` می‌تواند override نهایی virtual functions مربوط به base classes باشد، مشروط بر اینکه امضای تابع و قواعد override آن‌ها سازگار باشد.

اگر کلاس دیگری از `C` مشتق شود، دیگر نمی‌تواند `process` را override کند:

```cpp
class D : public C {
public:
    void process() override;
};
```

این کد نامعتبر خواهد بود.

## نکات مهم دربارهٔ `final` و Virtual Destructor

اگر یک کلاس قرار است polymorphic باشد، معمولاً باید destructor آن virtual باشد:

```cpp
class Base {
public:
    virtual ~Base() = default;
};
```

این موضوع مستقل از `final` است.

اگر implementation نهایی یک کلاس باشد، می‌توان آن کلاس را `final` کرد:

```cpp
class Derived final : public Base {
public:
    ~Derived() override = default;
};
```

در اینجا `override` روی destructor نیز نشان می‌دهد که destructor پایه virtual بوده است.

در طراحی polymorphic، این الگو معمولاً از نظر intent خوانا است.

## نکات مهم دربارهٔ نام `final`

از آنجا که `final` یک contextual keyword است، ++C اجازه می‌دهد در contextهای خاصی همچنان به‌عنوان identifier استفاده شود، مشروط بر اینکه grammar مربوطه آن را به‌عنوان specifier تفسیر نکند.

بااین‌حال، استفاده از نام‌هایی مانند `final` در کد جدید معمولاً انتخاب مناسبی نیست، زیرا می‌تواند خوانایی را کاهش دهد و با مفهوم استاندارد آن تداخل ذهنی ایجاد کند.

بهتر است از identifierهای واضح و غیرمبهم استفاده شود.

## تفاوت نسخه‌های استاندارد ++C

قابلیت `final` از **++C11** وارد استاندارد زبان شد.

در نسخه‌های بعدی، یعنی ++C14، ++C17، ++C20 و ++C23، این قابلیت همچنان بخشی از زبان است.

بنابراین اگر کدی با `final` مشاهده می‌کنید، این ویژگی ذاتاً نیازمند ++C14 یا ++C17 یا استانداردهای جدیدتر نیست؛ `final` از ++C11 در دسترس است.

در ++C98 و ++C03، keyword استاندارد `final` برای این منظور وجود نداشت.

## مقایسه با روش‌های بدون `final`

بدون `final` ممکن است چنین hierarchyای داشته باشیم:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() override;
};

class MoreDerived : public Derived {
public:
    void process() override;
};
```

در این ساختار، هر سطح می‌تواند رفتار virtual function را تغییر دهد.

اگر طراحی ایجاب کند که `Derived` آخرین implementation باشد، می‌توان آن را به شکل زیر نوشت:

```cpp
class Base {
public:
    virtual void process();
};

class Derived final : public Base {
public:
    void process() override;
};
```

در این حالت:

```cpp
class MoreDerived : public Derived {
};
```

غیرمجاز است.

اگر فقط خود تابع باید نهایی باشد ولی سایر اعضای کلاس همچنان قابل inheritance باشند، باید `final` را روی تابع قرار داد:

```cpp
class Derived : public Base {
public:
    void process() override final;
};
```

این تفاوت یکی از مهم‌ترین تصمیم‌های طراحی هنگام استفاده از `final` است.

## نکات مهم برای Code Review

هنگام بررسی کدی که از `final` استفاده می‌کند، بهتر است چند سؤال مطرح شود:

* آیا کلاس واقعاً نباید قابل مشتق شدن باشد؟
* آیا تابع واقعاً نباید در subclassهای بعدی override شود؟
* آیا این محدودیت بخشی از contract کلاس است؟
* آیا استفاده از `override` نیز لازم است؟
* آیا `final` برای حل یک مشکل طراحی عمیق‌تر استفاده شده است؟
* آیا این محدودیت روی public API اثر نامطلوب دارد؟
* آیا composition گزینهٔ مناسب‌تری از inheritance است؟
* آیا `final` صرفاً با هدف فرضی optimization اضافه شده است؟

اگر پاسخ‌ها نشان دهند که محدودیت بخشی از طراحی نیست، ممکن است `final` انتخاب مناسبی نباشد.

## الگوی پیشنهادی برای Virtual Function

برای کلاسی که تابعی را override می‌کند و باید آخرین override باشد، الگوی زیر معمولاً خوانا و صریح است:

```cpp
class Base {
public:
    virtual ~Base() = default;
    virtual void process();
};

class Derived : public Base {
public:
    void process() override final;
};
```

در این الگو، compiler هم override بودن تابع و هم نهایی بودن آن را بررسی می‌کند.

## الگوی پیشنهادی برای کلاس نهایی

برای کلاسی که نباید subclass داشته باشد، الگوی سادهٔ زیر کافی است:

```cpp
class JsonParser final {
public:
    void parse();
};
```

در چنین حالتی نیازی نیست صرفاً برای اعلام نهایی بودن، تمام member functionها با `final` مشخص شوند.

`final` روی class خودش inheritance را ممنوع می‌کند.

## خطاهای معمول Compiler

اگر یک تابع `final` دوباره override شود، compiler باید برنامه را رد کند.

نمونهٔ خطا:

```cpp
class Base {
public:
    virtual void run();
};

class Derived : public Base {
public:
    void run() final;
};

class MoreDerived : public Derived {
public:
    void run() override;
};
```

در این مثال، `MoreDerived::run` نمی‌تواند `Derived::run` را override کند.

همچنین اگر از یک `final class` ارث‌بری شود:

```cpp
class Base final {
};

class Derived : public Base {
};
```

این کد نیز باید در مرحلهٔ compile رد شود.

پیام دقیق compiler بسته به implementation متفاوت است و بخشی از استاندارد نیست؛ چیزی که استاندارد الزام می‌کند، ill-formed بودن برنامه است.

## تفاوت Standard و Compiler Extension

قابلیت `final` یک ویژگی استاندارد زبان از ++C11 است و compiler extension محسوب نمی‌شود.

بنابراین compilerهای conforming که ++C11 را پشتیبانی می‌کنند باید قواعد مربوط به آن را پیاده‌سازی کنند.

در مقابل، جزئیاتی مانند متن دقیق diagnostic message یا optimizationهایی که compiler در نتیجهٔ استفاده از `final` انجام می‌دهد، implementation-specific هستند.

برای مثال، نباید به یک متن خاص از خطای compiler وابسته شد.

## توصیه‌های عملی

برای طراحی مدرن ++C می‌توان چند قاعدهٔ ساده را در نظر گرفت:

* برای جلوگیری از override از `final` روی virtual function استفاده کنید.
* برای جلوگیری از inheritance از `final` روی class استفاده کنید.
* برای overrideهای واقعی، `override` را ترجیحاً همراه با `final` به کار ببرید.
* از `final` صرفاً برای حدس دربارهٔ performance استفاده نکنید.
* در public API، تأثیر محدودکنندهٔ `final` را در نظر بگیرید.
* اگر inheritance از نظر مفهومی مناسب نیست، ابتدا composition را بررسی کنید.
* `final` را برای بیان یک تصمیم واقعی در طراحی استفاده کنید، نه برای تزئین declaration.
* تفاوت بین `override`، `final` و non-virtual functions را در code review در نظر بگیرید.

## جمع‌بندی نهایی

در ++C11، `final` ابزاری استاندارد برای محدود کردن polymorphism و inheritance است.

این keyword دو کاربرد اصلی دارد: می‌توان با آن یک `virtual function` را از override شدن بیشتر منع کرد و می‌توان با آن یک `class` را از مشتق شدن منع کرد.

استفادهٔ ترکیبی از `override` و `final` برای virtual functions معمولاً intent کد را به‌خوبی بیان می‌کند:

```cpp
class Base {
public:
    virtual void process();
};

class Derived : public Base {
public:
    void process() override final;
};
```

همچنین برای جلوگیری کامل از inheritance می‌توان از این الگو استفاده کرد:

```cpp
class Component final {
public:
    void process();
};
```

نکتهٔ اصلی این است که `final` بیشتر از آنکه یک ابزار optimization باشد، **ابزار بیان و enforce کردن قرارداد طراحی** است. این قابلیت باید زمانی استفاده شود که واقعاً بخشی از طراحی کلاس این باشد که implementation یا hierarchy در نقطهٔ مشخصی پایان پیدا کند.


---

## 🤝 مشارکت ها

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>