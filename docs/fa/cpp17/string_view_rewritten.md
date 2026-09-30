# آموزش جامع `std::string_view` در ++C17 و نسخه‌های جدیدتر

`std::string_view` یکی از قابلیت‌های مهم ++C مدرن است که برای **مشاهده و پردازش رشته‌ها بدون مالکیت داده و بدون کپی کردن کاراکترها** طراحی شده است.

اگر با `std::string`، رشته‌های C-style و `const std::string&` کار کرده باشید، احتمالاً با موقعیت‌هایی روبه‌رو شده‌اید که یک تابع فقط می‌خواهد رشته را بخواند، اما نوع پارامتر می‌تواند باعث ساخت `std::string` موقت یا کپی غیرضروری شود.

نوع `std::string_view` برای حل بخش بزرگی از این مسئله طراحی شده است. این نوع یک View سبک روی محدوده‌ای از کاراکترهای موجود ایجاد می‌کند و خودش مالک آن داده‌ها نیست.

---

## معرفی `std::string_view`

برای درک ساده، می‌توان `std::string_view` را تقریباً به این شکل تصور کرد:

```cpp
class string_view_like
{
    const char* data;
    std::size_t size;
};
```

پیاده‌سازی واقعی استاندارد الزاماً دقیقاً چنین ساختاری ندارد، اما این مدل ذهنی برای درک رفتار آن بسیار مناسب است.

در این مدل، یک View دو اطلاعات اصلی را نگه می‌دارد:

```text
آدرس شروع داده
تعداد کاراکترهایی که باید مشاهده شوند
```

بنابراین View خودش رشته را در اختیار ندارد و هنگام ساخته‌شدن از یک رشته، کاراکترها را کپی نمی‌کند.

برای نمونه:

```cpp
std::string text = "Hello World";
std::string_view view = text;
```

در اینجا دو رشته‌ی مستقل وجود ندارد. View فقط محدوده‌ای از داده‌ی متعلق به `text` را مشاهده می‌کند.

---

## تفاوت مالکیت بین `std::string` و `std::string_view`

مهم‌ترین تفاوت این دو نوع، **Ownership** یا مالکیت داده است.

### معرفی `std::string`

نوع `std::string` مالک داده است و مدیریت storage و lifetime داده‌ی خودش را بر عهده دارد.

```cpp
std::string text = "Hello";
```

به‌صورت مفهومی می‌توان مسئولیت‌های آن را چنین در نظر گرفت:

```text
std::string
│
├── مالک داده
├── مدیریت حافظه
├── امکان تغییر محتوا
├── امکان تغییر اندازه
└── مسئول lifetime داده‌ی خودش
```

### معرفی `std::string_view`

در مقابل، `std::string_view` مالک داده نیست:

```cpp
std::string_view view = text;
```

مسئولیت‌های آن به‌صورت مفهومی چنین است:

```text
std::string_view
│
├── مالک داده نیست
├── حافظه را مدیریت نمی‌کند
├── کاراکترها را کپی نمی‌کند
├── فقط یک محدوده از داده را مشاهده می‌کند
└── مسئول lifetime داده نیست
```

پس یک قاعده‌ی مهم این است:

> `std::string` صاحب داده است، اما `std::string_view` فقط یک View روی داده ایجاد می‌کند.

---

## مشکل `const std::string&` با String Literal

فرض کنید تابعی فقط می‌خواهد یک متن را بخواند:

```cpp
void print(const std::string& text)
{
    std::cout << text;
}
```

اکنون می‌توان آن را با یک String literal فراخوانی کرد:

```cpp
print("Hello");
```

نوع `"Hello"`، `std::string` نیست. این مقدار یک آرایه از `const char` است:

```cpp
const char[6]
```

در چنین APIای امکان ساخته‌شدن یک `std::string` موقت برای فراهم‌کردن آرگومان وجود دارد:

```text
"Hello"
   ↓
const char[6]
   ↓
std::string temporary
   ↓
const std::string&
```

اگر تابع فقط قصد خواندن متن را داشته باشد، چنین ساختی ممکن است غیرضروری باشد.

---

## حل مسئله با `std::string_view`

برای چنین تابعی می‌توان API را به شکل زیر طراحی کرد:

```cpp
void print(std::string_view text)
{
    std::cout << text;
}
```

اکنون منابع مختلف رشته‌ای می‌توانند از یک API مشترک استفاده کنند:

```cpp
print("Hello");
```

```cpp
std::string name = "Ali";
print(name);
```

```cpp
const char* text = "Hello";
print(text);
```

```cpp
std::string_view view = "Hello";
print(view);
```

در نتیجه به‌جای طراحی چند Overload برای منابع مختلف، می‌توان یک API عمومی‌تر داشت:

```cpp
void print(std::string_view text);
```

این الگو یکی از کاربردهای مهم `std::string_view` در طراحی API است.

---

## بررسی تأثیر `std::string_view` بر Performance

مزیت اصلی `std::string_view` حذف برخی **copy** و **allocation**های غیرضروری است.

بااین‌حال بهتر است آن را یک ابزار جادویی برای سریع‌ترشدن همه‌ی برنامه‌ها در نظر نگیریم. ارزش اصلی آن زمانی دیده می‌شود که داده‌ی موجود را بتوان بدون ساخت یک `std::string` جدید پردازش کرد.

مهم‌ترین مزایا در این سناریوها عبارت‌اند از:

```text
کاهش temporaryهای غیرضروری
کاهش allocation
کاهش copy
پردازش مستقیم داده‌ی موجود
ساخت Sub-view بدون کپی کاراکترها
```

برای نمونه، اگر تابع فقط می‌خواهد متنی را جست‌وجو یا Parse کند، ساخت یک رشته‌ی مستقل برای هر بخش از متن ممکن است ضرورتی نداشته باشد.

---

## ساخت View بدون Copy

فرض کنید:

```cpp
std::string text = "Hello World";
std::string_view view = text;
```

هنگام ساخت `view`، داده‌ی رشته به یک buffer جدید کپی نمی‌شود. مدل مفهومی بهتر چنین است:

```text
text
┌──────────────────────┐
│ Hello World          │
└──────────────────────┘
          ↑
          │
        view
```

در نتیجه ساخت خود View بسیار سبک است؛ اما این سبکی به این معنا نیست که lifetime داده‌ی اصلی دیگر اهمیتی ندارد.

---

## ارسال `string_view` به تابع به‌صورت Value

برای یک API معمولی، دریافت `std::string_view` به‌صورت Value انتخاب مناسبی است:

```cpp
void process(std::string_view text)
{
    // ...
}
```

معمولاً نیازی نیست آن را به‌صورت Reference ثابت دریافت کنید:

```cpp
void process(const std::string_view& text)
{
    // ...
}
```

دلیل اصلی این است که `std::string_view` یک نوع کوچک و Cheap-to-copy است و مدل ذهنی آن تقریباً به یک Pointer و یک Size نزدیک است:

```cpp
const char* data;
std::size_t size;
```

بنابراین برای APIهای معمولی این شکل ساده و مناسب است:

```cpp
void process(std::string_view value);
```

---

## نکته‌ی مهم درباره‌ی Lifetime

مهم‌ترین نکته در کار با `std::string_view` این است که View مالک داده نیست.

برای نمونه، این طراحی خطرناک است:

```cpp
std::string_view getName()
{
    std::string name = "Ali";
    return name;
}
```

دلیل مشکل این است که با پایان تابع، `name` از بین می‌رود، درحالی‌که View بازگردانده‌شده هنوز به storage آن اشاره می‌کند:

```text
name
 ↓
داده‌ی رشته
 ↓
return string_view
 ↓
پایان تابع
 ↓
name نابود می‌شود
 ↓
string_view به داده‌ی نامعتبر اشاره می‌کند
```

بنابراین View فقط تا زمانی معتبر است که داده‌ی مورد اشاره معتبر بماند.

---

## خطر Temporary در `string_view`

این الگو نیز خطرناک است:

```cpp
std::string_view view = std::string("Hello");
```

در این حالت یک `std::string` موقت ساخته می‌شود و پس از پایان Expression مربوطه از بین می‌رود. در نتیجه View می‌تواند به داده‌ای اشاره کند که دیگر lifetime آن تمام شده است.

مدل مفهومی:

```text
temporary std::string
        ↓
string_view
        ↓
temporary destroyed
        ↓
dangling string_view
```

در مقابل، این حالت می‌تواند معتبر باشد:

```cpp
std::string text = "Hello";
std::string_view view = text;
```

مشروط بر اینکه `text` زنده بماند و عملیاتی روی owner انجام نشود که storage مورد اشاره را بی‌اعتبار کند.

---

## استفاده از String Literal

String literal یکی از منابع مناسب برای ساخت `std::string_view` است:

```cpp
std::string_view view = "Hello";
```

همچنین می‌توان آن را مستقیماً به تابعی که `std::string_view` می‌گیرد ارسال کرد:

```cpp
void print(std::string_view text)
{
    std::cout << text;
}

print("Hello");
```

در این حالت View به داده‌ای با lifetime مناسب برای چنین استفاده‌ای اشاره می‌کند.

---

## دسترسی به اندازه‌ی View

برای دریافت تعداد کاراکترهای موجود در View می‌توان از `size()` استفاده کرد:

```cpp
std::string_view view = "Hello";

std::cout << view.size();
```

خروجی:

```text
5
```

تابع `length()` نیز همین مفهوم را ارائه می‌کند:

```cpp
view.length();
```

همچنین `max_size()` حداکثر اندازه‌ی قابل نمایش توسط View را ارائه می‌کند:

```cpp
view.max_size();
```

---

## بررسی خالی بودن View

برای بررسی اینکه View هیچ کاراکتری ندارد، می‌توان از `empty()` استفاده کرد:

```cpp
if (view.empty())
{
    // ...
}
```

این روش معمولاً از مقایسه‌ی مستقیم `size()` با صفر خواناتر است.

---

## دسترسی به کاراکترها

مانند `std::string` می‌توان با `operator[]` به کاراکترهای View دسترسی داشت:

```cpp
std::string_view view = "Hello";

std::cout << view[0];
```

خروجی:

```text
H
```

### بررسی محدوده با `at()`

برای دسترسی همراه با Bounds Checking می‌توان از `at()` استفاده کرد:

```cpp
view.at(0);
```

تفاوت مهم این دو روش چنین است:

```cpp
view[0];
```

بررسی محدوده انجام نمی‌دهد، درحالی‌که:

```cpp
view.at(0);
```

برای Index نامعتبر رفتار خطای مناسب نوع را ایجاد می‌کند.

---

## دسترسی به ابتدا و انتهای View

برای دریافت اولین کاراکتر می‌توان از `front()` استفاده کرد:

```cpp
view.front();
```

برای دریافت آخرین کاراکتر نیز `back()` در دسترس است:

```cpp
view.back();
```

برای مثال:

```cpp
std::string_view view = "Hello";

std::cout << view.front(); // H
std::cout << view.back();  // o
```

---

## دریافت Pointer با `data()`

برای دریافت Pointer به ابتدای داده می‌توان از `data()` استفاده کرد:

```cpp
const char* ptr = view.data();
```

اما یک نکته‌ی بسیار مهم وجود دارد: `data()` تضمین نمی‌کند که کاراکتر بلافاصله پس از محدوده‌ی View، حتماً `'\0'` باشد.

برای نمونه:

```cpp
std::string str = "Hello World";
std::string_view view(str.data() + 6, 5);
```

View فقط بخش زیر را مشاهده می‌کند:

```text
World
```

اما `view.data()` فقط Pointer به `W` است. بنابراین استفاده‌ی مستقیم از آن به‌عنوان C-string همیشه صحیح نیست:

```cpp
printf("%s", view.data());
```

چنین APIای انتظار یک رشته‌ی Null-terminated دارد، درحالی‌که `string_view` فقط یک محدوده‌ی طول‌دار از کاراکترها را نمایش می‌دهد.

---

## پیمایش روی `string_view`

می‌توان مانند بسیاری از Containerها روی View پیمایش کرد:

```cpp
for (char c : view)
{
    std::cout << c;
}
```

Iteratorهای معمول نیز در دسترس هستند:

```cpp
begin()
end()

cbegin()
cend()

rbegin()
rend()

crbegin()
crend()
```

در نتیجه `std::string_view` می‌تواند با Range-based `for` و بسیاری از الگوریتم‌های استاندارد استفاده شود.

---

## ایجاد Sub-view با `substr()`

فرض کنید:

```cpp
std::string_view view = "Hello World";
```

می‌توان بخشی از آن را انتخاب کرد:

```cpp
auto sub = view.substr(6, 5);
```

مقدار مشاهده‌شده:

```text
World
```

نکته‌ی مهم این است که `sub` همچنان یک `std::string_view` است. بنابراین برای ایجاد این Sub-view نیازی به ساخت یک `std::string` مستقل نیست.

---

## تفاوت `std::string::substr()` و `std::string_view::substr()`

در این حالت:

```cpp
std::string str = "Hello World";
auto result = str.substr(6, 5);
```

`result` یک `std::string` جدید است.

اما در این حالت:

```cpp
std::string_view view = "Hello World";
auto result = view.substr(6, 5);
```

`result` یک `std::string_view` جدید است.

بنابراین مدل کلی چنین است:

```text
std::string::substr()
        ↓
std::string جدید

std::string_view::substr()
        ↓
string_view جدید
```

این تفاوت در Parserها و پردازش حجم زیادی از متن اهمیت زیادی دارد.

---

## تغییر محدوده با `remove_prefix()`

فرض کنید:

```cpp
std::string_view view = "Hello World";
```

با این عملیات:

```cpp
view.remove_prefix(6);
```

خود View از ابتدای محدوده‌ی قبلی جلو می‌رود و اکنون فقط این بخش را مشاهده می‌کند:

```text
World
```

رشته‌ی اصلی تغییر نمی‌کند. فقط محدوده‌ای که View مشاهده می‌کند تغییر کرده است.

این ویژگی برای Tokenizerها و Parserها بسیار کاربردی است.

---

## تغییر محدوده با `remove_suffix()`

به‌صورت مشابه، می‌توان انتهای محدوده را کوتاه کرد:

```cpp
std::string_view view = "Hello World";
view.remove_suffix(6);
```

اکنون View بخش زیر را مشاهده می‌کند:

```text
Hello
```

باز هم داده‌ی اصلی تغییر نکرده است و فقط محدوده‌ی View تغییر کرده است.

---

## تفاوت `substr()` و `remove_prefix()`

این دو عملیات هدف یکسانی ندارند.

### ایجاد View جدید با `substr()`

عملیات زیر یک View جدید می‌سازد و View اصلی را دست‌نخورده باقی می‌گذارد:

```cpp
auto part = view.substr(6, 5);
```

### تغییر خود View با `remove_prefix()`

عملیات زیر خود View را تغییر می‌دهد:

```cpp
view.remove_prefix(6);
```

برای Parserهایی که متن را مرحله‌به‌مرحله مصرف می‌کنند، `remove_prefix()` می‌تواند الگوی بسیار مناسبی باشد.

---

## جست‌وجو با `find()`

برای جست‌وجوی یک زیررشته یا Character می‌توان از `find()` استفاده کرد:

```cpp
std::string_view text = "Hello World";

auto pos = text.find("World");
```

نتیجه در این مثال:

```text
6
```

برای Character نیز می‌توان نوشت:

```cpp
text.find('W');
```

---

## جست‌وجو از انتها با `rfind()`

تابع `rfind()` جست‌وجو را از انتهای View انجام می‌دهد:

```cpp
auto pos = text.rfind('o');
```

این تابع برای پیدا کردن آخرین رخداد یک مقدار مفید است.

---

## بررسی انواع جست‌وجو

`string_view` مجموعه‌ای از عملیات جست‌وجوی متداول را ارائه می‌کند:

```cpp
find()
rfind()

find_first_of()
find_last_of()

find_first_not_of()
find_last_not_of()
```

برای نمونه:

```cpp
std::string_view text = "   Hello";

auto pos = text.find_first_not_of(' ');
```

در این مثال، اولین Characterای که Space نیست پیدا می‌شود.

---

## بررسی نتیجه‌ی جست‌وجو با `npos`

برای تشخیص اینکه مقدار موردنظر پیدا نشده است، می‌توان از `std::string_view::npos` استفاده کرد:

```cpp
auto pos = text.find("World");

if (pos != std::string_view::npos)
{
    // Found
}
```

`npos` مقدار ویژه‌ای برای نمایش وضعیت «پیدا نشد» است.

---

## بررسی Prefix با `starts_with()`

از C++20، `starts_with()` برای بررسی Prefix در دسترس است:

```cpp
std::string_view url = "https://example.com";

if (url.starts_with("https://"))
{
    // ...
}
```

این روش برای بررسی Prefix خواناتر از پیاده‌سازی دستی چنین مقایسه‌ای است.

---

## بررسی Suffix با `ends_with()`

از C++20 می‌توان با `ends_with()` انتهای View را بررسی کرد:

```cpp
std::string_view file = "image.png";

if (file.ends_with(".png"))
{
    // ...
}
```

این قابلیت برای بررسی پسوند فایل و سایر Suffixها کاربردی است.

---

## جست‌وجوی مستقیم با `contains()`

از C++23، تابع `contains()` برای بررسی وجود یک مقدار اضافه شده است:

```cpp
std::string_view text = "Hello World";

if (text.contains("World"))
{
    // ...
}
```

برای Character نیز قابل استفاده است:

```cpp
if (text.contains('W'))
{
    // ...
}
```

در نتیجه برای پرسش ساده‌ی «آیا این مقدار وجود دارد؟» دیگر لازم نیست نتیجه‌ی `find()` با `npos` مقایسه شود.

---

## مقایسه‌ی Viewها

می‌توان دو View را با عملگرهای مقایسه مقایسه کرد:

```cpp
std::string_view a = "Hello";
std::string_view b = "World";

if (a == b)
{
    // ...
}
```

عملگرهای مقایسه‌ی معمول نیز در دسترس هستند:

```cpp
==
!=
<
<=
>
>=
<=>
```

همچنین برای مقایسه‌ی صریح می‌توان از `compare()` استفاده کرد:

```cpp
a.compare(b);
```

---

## کپی صریح با `copy()`

`string_view` به‌طور پیش‌فرض هنگام ایجاد View کاراکترها را کپی نمی‌کند. اگر واقعاً لازم باشد بخشی از View در یک Buffer کپی شود، می‌توان از `copy()` استفاده کرد:

```cpp
char buffer[10];

view.copy(buffer, 5);
```

در اینجا برخلاف `substr()`، عملیات Copy واقعاً انجام می‌شود.

بنابراین `string_view` مانع Copy موردنیاز برنامه نمی‌شود؛ بلکه امکان می‌دهد Copyهای غیرضروری را حذف کنید.

---

## جابه‌جایی Viewها با `swap()`

برای جابه‌جایی دو View می‌توان از `swap()` استفاده کرد:

```cpp
view1.swap(view2);
```

این عملیات داده‌ی اصلی را جابه‌جا نمی‌کند؛ فقط وضعیت و محدوده‌ی خود Viewها جابه‌جا می‌شود.

---

## استفاده از Literal با پسوند `sv`

در ++C می‌توان از Literal مخصوص `string_view` استفاده کرد:

```cpp
using namespace std::literals;

std::string_view text = "Hello"sv;
```

این Syntax برای زمانی مفید است که بخواهید نوع Literal به‌صورت صریح یک `string_view` باشد.

---

## بررسی `std::string_view` در Parserها

یکی از مهم‌ترین کاربردهای `string_view` پردازش متن بدون ساخت Substringهای متعدد است.

فرض کنید ورودی زیر را داریم:

```cpp
std::string input = "name=Ali&age=25";
std::string_view view = input;
```

می‌توان محل `=` را پیدا کرد:

```cpp
auto equal = view.find('=');
```

سپس Key را به‌صورت یک View استخراج کرد:

```cpp
auto key = view.substr(0, equal);
```

و باقی متن را نیز بدون Copy به‌عنوان View دیگری مشاهده کرد:

```cpp
auto value = view.substr(equal + 1);
```

مدل مفهومی چنین است:

```text
input
┌──────────────────────┐
│ name=Ali&age=25      │
└──────────────────────┘

key
┌──────┐
│ name │
└──────┘

value
┌─────────────────┐
│ Ali&age=25      │
└─────────────────┘
```

در این الگو `key` و `value` می‌توانند روی همان داده‌ی اصلی View باشند.

این ویژگی برای Tokenizerها، Parserهای متنی، CSV Parserها، HTTP Parserها، Configuration Parserها و پردازش فایل‌ها مفید است.

---

## پردازش متن بدون ساخت Substring

یک الگوی کاربردی دیگر، تغییر تدریجی محدوده‌ی View است:

```cpp
std::string_view input = "   Hello World   ";
```

برای حذف فاصله‌های ابتدای View می‌توان محدوده را جلو برد:

```cpp
input.remove_prefix(
    input.find_first_not_of(' ')
);
```

در این حالت هیچ Characterای از رشته‌ی اصلی حذف نمی‌شود؛ فقط View از نقطه‌ی دیگری شروع می‌شود.

به همین شکل می‌توان با `remove_suffix()` محدوده‌ی انتهایی را مدیریت کرد.

---

## تغییر `std::string` و اعتبار View

فرض کنید:

```cpp
std::string text = "Hello";
std::string_view view = text;
```

سپس رشته تغییر کند:

```cpp
text += " World";
```

اگر تغییر باعث شود storage رشته جابه‌جا یا Reallocate شود، View قبلی ممکن است دیگر به داده‌ی معتبر اشاره نکند.

بنابراین یک اصل مهم این است:

> اعتبار `string_view` به اعتبار و lifetime داده‌ی اصلی وابسته است.

هر عملیاتی روی Owner که بتواند storage را جابه‌جا، Reallocate یا نابود کند باید از نظر اعتبار View بررسی شود.

---

## ذخیره‌کردن `string_view` در Object

استفاده از `std::string_view` به‌عنوان Member یک Class ذاتاً اشتباه نیست، اما lifetime داده باید کاملاً مشخص باشد:

```cpp
class User
{
    std::string_view name;
};
```

اگر View به داده‌ی خارجی اشاره کند و آن داده زودتر از Object از بین برود، Member به داده‌ای نامعتبر تبدیل می‌شود.

برای نمونه:

```cpp
User user;

std::string name = "Ali";

user.name = name;
```

اگر `name` از بین برود، `user.name` دیگر نمی‌تواند به‌صورت ایمن از داده‌ی قبلی استفاده کند.

اگر Object باید مالک نام باشد، معمولاً `std::string` انتخاب مناسب‌تری است:

```cpp
class User
{
    std::string name;
};
```

در مقابل، اگر Object فقط برای مدت مشخصی یک View روی داده‌ی خارجی نگه می‌دارد و lifetime آن داده تضمین شده است، `string_view` می‌تواند انتخاب مناسبی باشد.

---

## مقایسه‌ی Ownership و View

یک مدل ذهنی ساده برای تفاوت این دو نوع چنین است:

```text
std::string
    ↓
"I own this data."

std::string_view
    ↓
"I only observe this data."
```

در طراحی API نیز می‌توان از همین اصل استفاده کرد.

اگر تابع فقط داده را می‌خواند:

```cpp
void parse(std::string_view input);
```

اگر یک Object باید داده را نگه دارد و مالک آن باشد:

```cpp
class Parser
{
    std::string input;
};
```

---

## مقایسه با `const char*`

سه نوع رایج را می‌توان از نظر مدل استفاده چنین مقایسه کرد:

```text
const char*
    │
    ├── فقط Pointer
    ├── معمولاً برای C-string استفاده می‌شود
    └── طول را به‌صورت مستقل همراه خود ندارد

std::string_view
    │
    ├── Pointer + Size
    ├── Null termination لازم نیست
    ├── Sub-view بدون Copy
    └── مالک داده نیست

std::string
    │
    ├── مالک داده
    ├── مدیریت lifetime
    ├── Size
    ├── قابلیت تغییر
    └── امکان Allocation
```

به همین دلیل `string_view` در بسیاری از APIهای متنی مدرن، Abstraction مناسبی برای نمایش یک محدوده‌ی رشته‌ای است.

---

## نکات مربوط به Null Termination

یک قانون مهم این است:

> `std::string_view` یک C-string نیست.

برای نمونه، ممکن است داده‌ی زیر Null-terminated باشد:

```cpp
std::string_view view("Hello");
```

اما در این مثال:

```cpp
std::string str = "Hello World";
std::string_view view(str.data() + 6, 5);
```

View فقط این محدوده را مشاهده می‌کند:

```text
World
```

بنابراین View الزاماً به یک `'\0'` در انتهای محدوده ختم نمی‌شود.

در نتیجه APIهایی که صرفاً:

```cpp
const char*
```

دریافت می‌کنند و انتظار C-string دارند، همیشه نمی‌توانند مستقیماً `string_view.data()` دریافت کنند.

---

## امکانات اصلی `std::string_view`

امکانات اصلی را می‌توان در چند گروه قرار داد.

### ساخت و مقداردهی

```cpp
std::string_view()
std::string_view(other)
std::string_view(ptr, count)
std::string_view(c_string)
```

در استانداردهای جدیدتر، قابلیت‌های مرتبط با Range نیز توسعه یافته‌اند.

### اطلاعات و Capacity

```cpp
size()
length()
max_size()
empty()
```

### دسترسی به عناصر

```cpp
operator[]()
at()
front()
back()
data()
```

### Iteratorها

```cpp
begin()
end()

cbegin()
cend()

rbegin()
rend()

crbegin()
crend()
```

### تغییر محدوده‌ی View

```cpp
remove_prefix()
remove_suffix()
swap()
```

### Substring و Copy

```cpp
substr()
copy()
```

### عملیات جست‌وجو

```cpp
find()
rfind()

find_first_of()
find_last_of()

find_first_not_of()
find_last_not_of()
```

### عملیات مقایسه

```cpp
compare()

==
!=
<
<=
>
>=
<=>
```

### قابلیت‌های C++20

```cpp
starts_with()
ends_with()
```

### قابلیت C++23

```cpp
contains()
```

### ثابت ویژه

```cpp
std::string_view::npos
```

### Literal

```cpp
"Hello"sv
```

### استفاده در ساختارهای Hash-based

برای `std::string_view`، Specialization مربوط به `std::hash` وجود دارد و می‌توان از آن در ساختارهایی مانند این موارد استفاده کرد:

```cpp
std::unordered_map
std::unordered_set
```

در این سناریو نیز lifetime داده‌ای که View به آن اشاره می‌کند باید با دقت مدیریت شود.

---

## نسخه‌های استاندارد و قابلیت‌های مرتبط

بر اساس محتوای این سند، قابلیت‌های اصلی مورد بحث در نسخه‌های زیر قرار دارند:

| قابلیت | استاندارد |
| --- | --- |
| `std::string_view` | C++17 |
| `starts_with()` | C++20 |
| `ends_with()` | C++20 |
| `<=>` | C++20 |
| `contains()` | C++23 |
| قابلیت‌های جدید Range-related | استانداردهای جدیدتر |

هنگام استفاده از قابلیت‌های جدید، Compiler و Language Standard پروژه نیز باید متناسب با آن تنظیم شده باشند.

برای نمونه:

```bash
-std=c++20
```

یا:

```bash
-std=c++23
```

---

## انتخاب `std::string_view` برای API

`string_view` معمولاً برای APIای مناسب است که:

- فقط رشته را می‌خواند.
- قرار نیست مالک داده شود.
- باید String literal را مستقیماً دریافت کند.
- می‌خواهد از ساخت `std::string` موقت جلوگیری کند.
- Substringهای زیادی را پردازش می‌کند.
- Parser یا Tokenizer است.
- می‌خواهد Copy و Allocation غیرضروری را کاهش دهد.
- باید چند نوع منبع رشته‌ای را با یک Interface دریافت کند.

در چنین APIهایی، پارامتر زیر معمولاً انتخاب طبیعی‌ای است:

```cpp
void process(std::string_view text);
```

---

## انتخاب `std::string` به‌جای View

در شرایط زیر معمولاً `std::string` مناسب‌تر است:

- Object باید مالک داده باشد.
- lifetime داده باید مستقل از منبع اصلی باشد.
- رشته باید ذخیره شود.
- رشته قرار است تغییر کند.
- داده‌ی اصلی ممکن است پیش از Object از بین برود.
- lifetime داده‌ی مورد اشاره قابل تضمین نیست.

در چنین شرایطی انتخابی مانند زیر با مفهوم Ownership سازگارتر است:

```cpp
std::string
```

---

## چک‌لیست استفاده از `string_view`

هنگام استفاده از `std::string_view` چند سؤال مهم را بررسی کنید.

### بررسی مالکیت

آیا تابع یا Object باید مالک داده شود؟

اگر پاسخ منفی است، `std::string_view` می‌تواند گزینه‌ی مناسبی باشد:

```cpp
std::string_view
```

### بررسی Lifetime

آیا داده تا زمانی که View استفاده می‌شود زنده می‌ماند؟

اگر پاسخ منفی باشد، View می‌تواند Dangling شود.

### بررسی Reallocation

آیا Owner ممکن است به شکلی تغییر کند که storage آن جابه‌جا شود؟

اگر بله، اعتبار View باید دوباره بررسی شود.

### بررسی Null Termination

آیا API مقصد به Null-terminated string نیاز دارد؟

اگر بله، صرفاً داشتن `string_view.data()` کافی نیست.

### بررسی نیاز به ذخیره‌سازی

آیا داده قرار است مستقل از منبع اصلی ذخیره شود؟

اگر بله، باید Ownership را در نظر گرفت و در بسیاری از موارد `std::string` انتخاب مناسب‌تری است.

---

## مثال کامل استفاده از `string_view`

مثال زیر چند قابلیت مهم را در یک API واحد نشان می‌دهد:

```cpp
#include <iostream>
#include <string>
#include <string_view>

void analyze(std::string_view text)
{
    std::cout << "Text: " << text << '\n';

    std::cout << "Size: "
              << text.size()
              << '\n';

    std::cout << "Empty: "
              << std::boolalpha
              << text.empty()
              << '\n';

    if (text.starts_with("Hello"))
    {
        std::cout << "Starts with Hello\n";
    }

    if (text.ends_with("World"))
    {
        std::cout << "Ends with World\n";
    }

    if (text.find("World") != std::string_view::npos)
    {
        std::cout << "World found\n";
    }
}

int main()
{
    std::string str = "Hello World";

    analyze(str);
    analyze("Hello World");

    std::string_view view = str;
    analyze(view);
}
```

در این مثال یک API واحد:

```cpp
void analyze(std::string_view text);
```

می‌تواند منابع مختلفی مانند `std::string`، String literal و `std::string_view` را دریافت کند.

---

## جمع‌بندی مفهومی

`std::string_view` یکی از ابزارهای مهم ++C مدرن برای **پردازش رشته بدون مالکیت و بدون Copy غیرضروری** است.

مدل ذهنی اصلی چنین است:

```text
std::string
    ↓
مالک داده

std::string_view
    ↓
مشاهده‌کننده‌ی داده
```

برای APIای که فقط می‌خواهد متن را بخواند، این طراحی:

```cpp
void process(std::string_view text);
```

می‌تواند Interface انعطاف‌پذیری ایجاد کند و منابع مختلف رشته‌ای را پوشش دهد.

مهم‌ترین مزیت‌های آن عبارت‌اند از:

```text
عدم نیاز به مالکیت داده
عدم کپی کاراکترها هنگام ایجاد View
ایجاد Sub-view بدون ساخت string جدید
کاهش Allocationهای غیرضروری
API ساده‌تر
مناسب برای Parser و Tokenizer
جست‌وجو و مقایسه‌ی مستقیم
پشتیبانی مناسب از String literal
```

در کنار این مزایا، یک اصل باید همیشه در ذهن باقی بماند:

```text
string_view → مالک داده نیست
```

بنابراین lifetime داده‌ی اصلی باید تا پایان استفاده از View معتبر بماند.

اگر این مفهوم به‌درستی درک شود، `std::string_view` فقط یک نوع برای کار با رشته نیست؛ بلکه نمونه‌ای روشن از یک اصل مهم در طراحی ++C مدرن است:

> **مالکیت داده را از مشاهده و پردازش داده جدا کنید.**
