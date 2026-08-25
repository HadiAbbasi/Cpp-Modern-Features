# آموزش جامع `std::string_view` در C++17 و بالاتر

`std::string_view` یکی از قابلیت‌های مهم C++ مدرن است که برای **مشاهده و پردازش رشته‌ها بدون مالکیت و بدون کپی کردن داده‌ها** طراحی شده است.

اگر با `std::string`، رشته‌های C-style و `const std::string&` کار کرده باشید، احتمالاً با موقعیت‌هایی مواجه شده‌اید که یک تابع فقط می‌خواهد رشته را بخواند، اما نوع پارامتر باعث ایجاد تبدیل، ساخت temporary یا کپی غیرضروری می‌شود.

`std::string_view` دقیقاً برای حل بخش بزرگی از این مسائل طراحی شده است.

---

## 1. `std::string_view` چیست؟

برای درک ساده، می‌توانید `std::string_view` را تقریباً به این شکل تصور کنید:

```cpp
class string_view_like
{
    const char* data;
    std::size_t size;
};
```

پیاده‌سازی واقعی استاندارد الزاماً دقیقاً این ساختار نیست، اما این مدل ذهنی بسیار مناسبی است.

یعنی `string_view` تقریباً دو چیز را نگه می‌دارد:

```text
آدرس شروع داده
تعداد کاراکترهایی که باید دیده شوند
```

بنابراین خودش رشته را در اختیار ندارد و کاراکترها را نیز کپی نمی‌کند.

مثلاً:

```cpp
std::string text = "Hello World";

std::string_view view = text;
```

در اینجا دو رشته‌ی مستقل نداریم.

ساختار مفهومی به این شکل است:

```text
std::string
┌────────────────────────┐
│ Hello World            │
└────────────────────────┘
          ↑
          │
    std::string_view
```

`view` فقط به داده‌ی `text` اشاره می‌کند.

---

# 2. تفاوت اصلی `std::string` و `std::string_view`

مهم‌ترین تفاوت این دو مفهوم **Ownership** یا مالکیت داده است.

### `std::string`

```cpp
std::string text = "Hello";
```

`std::string` مالک داده است و مدیریت حافظه را بر عهده دارد.

به طور کلی:

```text
std::string
│
├── مالک داده
├── مدیریت حافظه
├── امکان تغییر اندازه
├── امکان تغییر محتوا
└── مسئول lifetime داده
```

### `std::string_view`

```cpp
std::string_view view = text;
```

اما:

```text
std::string_view
│
├── مالک داده نیست
├── حافظه را مدیریت نمی‌کند
├── داده را کپی نمی‌کند
├── فقط محدوده‌ای از داده را مشاهده می‌کند
└── مسئول lifetime داده نیست
```

پس یک تعریف بسیار مهم:

> `std::string` صاحب رشته است، اما `std::string_view` فقط روی رشته View ایجاد می‌کند.

---

# 3. مشکل `const std::string&` با String Literal

فرض کنید تابعی به این شکل دارید:

```cpp
void print(const std::string& text)
{
    std::cout << text;
}
```

و آن را این‌گونه فراخوانی می‌کنید:

```cpp
print("Hello");
```

نوع واقعی `"Hello"`، `std::string` نیست.

String literal از نوع آرایه‌ای از `const char` است:

```cpp
const char[6]
```

یعنی:

```text
"Hello"
   ↓
const char[6]
```

برای اینکه این مقدار بتواند به:

```cpp
const std::string&
```

داده شود، امکان ساخت یک `std::string` موقت وجود دارد:

```text
"Hello"
   ↓
const char[6]
   ↓
std::string temporary
   ↓
const std::string&
```

در چنین APIای، اگر تابع فقط قصد خواندن رشته را داشته باشد، ایجاد `std::string` موقت ضرورتی ندارد.

---

# 4. `std::string_view` این مشکل را چگونه حل می‌کند؟

تابع را می‌توان به شکل زیر طراحی کرد:

```cpp
void print(std::string_view text)
{
    std::cout << text;
}
```

حالا انواع مختلف ورودی می‌توانند به همین API داده شوند:

```cpp
print("Hello");
```

یا:

```cpp
std::string name = "Ali";

print(name);
```

یا:

```cpp
const char* text = "Hello";

print(text);
```

یا:

```cpp
std::string_view view = "Hello";

print(view);
```

در نتیجه به جای طراحی چند overload مختلف، می‌توان یک API عمومی‌تر داشت:

```cpp
void print(std::string_view text);
```

این یکی از مهم‌ترین کاربردهای `string_view` در طراحی API است.

---

# 5. آیا این موضوع روی Performance تأثیر دارد؟

مزیت اصلی `std::string_view` حذف **کپی و allocation غیرضروری** است.

بهتر است `string_view` را یک ابزار جادویی برای سریع‌تر شدن برنامه ندانیم.

مزیت آن بیشتر در چنین مواردی دیده می‌شود:

```text
کاهش temporary
کاهش allocation
کاهش copy
پردازش مستقیم داده‌ی موجود
ساخت substring بدون کپی
```

برای مثال، اگر تابع فقط می‌خواهد رشته را جستجو یا parse کند، ایجاد یک `std::string` جدید ممکن است کاملاً غیرضروری باشد.

با `string_view` می‌توان همان داده‌ی موجود را مشاهده کرد.

---

# 6. `string_view` خودش داده را Copy نمی‌کند

فرض کنید:

```cpp
std::string text = "Hello World";

std::string_view view = text;
```

این اتفاق نمی‌افتد:

```text
text
└── "Hello World"

view
└── "Hello World"   ← copy
```

بلکه:

```text
text
┌──────────────────────┐
│ Hello World          │
└──────────────────────┘
          ↑
          │
        view
```

پس ساخت `view` بسیار ارزان است.

---

# 7. ارسال `string_view` به تابع؛ Value یا Reference؟

برای یک API معمولی، این شکل مناسب است:

```cpp
void process(std::string_view text)
{
    // ...
}
```

و معمولاً نیازی به این شکل نیست:

```cpp
void process(const std::string_view& text)
{
    // ...
}
```

دلیل این است که `string_view` یک نوع کوچک و cheap-to-copy است.

از نظر مدل ذهنی، چیزی شبیه این دارد:

```cpp
const char* data;
std::size_t size;
```

بنابراین کپی کردن خود View هزینه‌ی بسیار کمی دارد.

به عنوان یک قاعده‌ی عملی:

```cpp
void foo(std::string_view value);
```

برای APIهای معمولی انتخاب مناسبی است.

---

# 8. مهم‌ترین نکته: `string_view` مالک داده نیست

این موضوع باید همیشه در طراحی با `string_view` در نظر گرفته شود.

مثلاً:

```cpp
std::string_view getName()
{
    std::string name = "Ali";

    return name;
}
```

این کد مشکل دارد.

چرا؟

چون:

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

بنابراین View بعد از پایان lifetime داده‌ی اصلی معتبر نیست.

---

# 9. Temporary و `string_view`

این مورد نیز خطرناک است:

```cpp
std::string_view view = std::string("Hello");
```

`std::string` موقت ساخته می‌شود و پس از پایان expression از بین می‌رود.

در نتیجه:

```text
temporary std::string
        ↓
string_view
        ↓
temporary destroyed
        ↓
dangling string_view
```

اما این حالت مشکلی ندارد:

```cpp
std::string text = "Hello";

std::string_view view = text;
```

تا زمانی که `text` زنده باشد و داده‌ی آن به شکلی که View را invalid کند تغییر نکند، View معتبر است.

---

# 10. String Literal و `string_view`

String literal برای `string_view` یک مورد بسیار مناسب است:

```cpp
std::string_view view = "Hello";
```

String literal طول عمر بسیار زیادی دارد و بنابراین برای چنین Viewای مناسب است.

حتی می‌توان مستقیماً آن را به تابع داد:

```cpp
void print(std::string_view text)
{
    std::cout << text;
}

print("Hello");
```

---

# 11. دسترسی به اندازه‌ی رشته

برای دریافت تعداد کاراکترهای View:

```cpp
view.size();
```

مثلاً:

```cpp
std::string_view view = "Hello";

std::cout << view.size();
```

خروجی:

```text
5
```

همچنین:

```cpp
view.length();
```

همین مفهوم را دارد.

---

# 12. بررسی خالی بودن

```cpp
if (view.empty())
{
    // ...
}
```

`empty()` بررسی می‌کند که View هیچ کاراکتری ندارد یا خیر.

---

# 13. `max_size()`

برای دریافت حداکثر اندازه‌ای که View می‌تواند نمایش دهد:

```cpp
view.max_size();
```

---

# 14. دسترسی به کاراکترها

مانند `std::string`:

```cpp
view[0]
```

مثلاً:

```cpp
std::string_view view = "Hello";

std::cout << view[0];
```

خروجی:

```text
H
```

---

## `at()`

برای دسترسی با bounds checking:

```cpp
view.at(0);
```

تفاوت مهم:

```cpp
view[0];
```

بررسی محدوده انجام نمی‌دهد.

در حالی که:

```cpp
view.at(0);
```

در صورت خارج بودن index از محدوده، خطا ایجاد می‌کند.

---

# 15. `front()` و `back()`

اولین کاراکتر:

```cpp
view.front();
```

آخرین کاراکتر:

```cpp
view.back();
```

مثلاً:

```cpp
std::string_view view = "Hello";

std::cout << view.front(); // H
std::cout << view.back();  // o
```

---

# 16. `data()`

برای دریافت pointer به داده:

```cpp
const char* ptr = view.data();
```

اما اینجا باید به یک نکته‌ی بسیار مهم توجه کرد.

`data()` تضمین نمی‌کند که داده‌ی View در انتهای محدوده حتماً `'\0'` داشته باشد.

مثلاً:

```cpp
std::string str = "Hello World";

std::string_view view(str.data() + 6, 5);
```

View فقط این قسمت است:

```text
World
```

اما `view.data()` صرفاً pointer به `W` است.

بنابراین استفاده‌ی مستقیم از آن به عنوان C-string همیشه صحیح نیست:

```cpp
printf("%s", view.data());
```

چون `%s` انتظار یک رشته‌ی null-terminated دارد.

---

# 17. Iteration روی `string_view`

می‌توان مانند container روی View حرکت کرد:

```cpp
for (char c : view)
{
    std::cout << c;
}
```

همچنین iteratorهای زیر وجود دارند:

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

بنابراین `string_view` با بسیاری از الگوریتم‌ها و range-based forها به‌خوبی کار می‌کند.

---

# 18. `substr()`؛ یکی از مهم‌ترین قابلیت‌ها

فرض کنید:

```cpp
std::string_view view = "Hello World";
```

می‌توان بخشی از آن را انتخاب کرد:

```cpp
auto sub = view.substr(6, 5);
```

نتیجه:

```text
World
```

اما نکته‌ی مهم این است که:

```cpp
sub
```

همچنان `std::string_view` است.

یعنی برای ایجاد این Sub-view نیازی به ساخت یک `std::string` مستقل نیست.

---

# 19. تفاوت `string::substr()` و `string_view::substr()`

در:

```cpp
std::string str = "Hello World";

auto result = str.substr(6, 5);
```

`result` یک `std::string` است.

اما:

```cpp
std::string_view view = "Hello World";

auto result = view.substr(6, 5);
```

`result` یک `std::string_view` است.

پس:

```text
std::string::substr()
        ↓
std::string جدید

std::string_view::substr()
        ↓
string_view جدید
```

این تفاوت در parserها و پردازش حجم زیادی از متن بسیار مهم است.

---

# 20. `remove_prefix()`

فرض کنید:

```cpp
std::string_view view = "Hello World";
```

با:

```cpp
view.remove_prefix(6);
```

View از این:

```text
Hello World
↑
```

به این تبدیل می‌شود:

```text
Hello World
      ↑
```

و محتوای قابل مشاهده‌ی View می‌شود:

```text
World
```

اما رشته‌ی اصلی تغییر نکرده است.

`remove_prefix()` فقط محدوده‌ی View را تغییر می‌دهد.

---

# 21. `remove_suffix()`

مشابه آن:

```cpp
std::string_view view = "Hello World";

view.remove_suffix(6);
```

اکنون View:

```text
Hello
```

را مشاهده می‌کند.

باز هم داده‌ی اصلی تغییر نکرده است.

---

# 22. تفاوت `substr()` و `remove_prefix()`

این دو کاربرد متفاوتی دارند.

### `substr()`

یک View جدید می‌سازد:

```cpp
auto part = view.substr(6, 5);
```

View اصلی دست‌نخورده باقی می‌ماند.

### `remove_prefix()`

خود View را تغییر می‌دهد:

```cpp
view.remove_prefix(6);
```

برای tokenizer و parserها این قابلیت بسیار مفید است.

---

# 23. جستجو با `find()`

مثلاً:

```cpp
std::string_view text = "Hello World";

auto pos = text.find("World");
```

نتیجه:

```text
6
```

همچنین می‌توان یک character را جستجو کرد:

```cpp
text.find('W');
```

---

# 24. `rfind()`

جستجو را از انتهای View انجام می‌دهد:

```cpp
auto pos = text.rfind('o');
```

این قابلیت برای پیدا کردن آخرین occurrence یک مقدار کاربرد دارد.

---

# 25. توابع مختلف جستجو

`string_view` مجموعه‌ی کاملی از توابع جستجو دارد:

```cpp
find()
rfind()

find_first_of()
find_last_of()

find_first_not_of()
find_last_not_of()
```

برای مثال:

```cpp
std::string_view text = "   Hello";

auto pos = text.find_first_not_of(' ');
```

اولین characterای که space نیست پیدا می‌شود.

---

# 26. `npos`

برای تشخیص اینکه چیزی پیدا نشده است:

```cpp
auto pos = text.find("World");

if (pos != std::string_view::npos)
{
    // پیدا شده
}
```

`npos` مقدار ویژه‌ای است که برای نمایش «پیدا نشد» استفاده می‌شود.

---

# 27. `starts_with()`

از C++20:

```cpp
std::string_view url = "https://example.com";

if (url.starts_with("https://"))
{
    // ...
}
```

این روش برای بررسی prefix بسیار خواناتر از جستجوی دستی است.

---

# 28. `ends_with()`

از C++20:

```cpp
std::string_view file = "image.png";

if (file.ends_with(".png"))
{
    // ...
}
```

برای بررسی suffix کاربرد دارد.

---

# 29. `contains()`

از C++23:

```cpp
std::string_view text = "Hello World";

if (text.contains("World"))
{
    // ...
}
```

برای character نیز قابل استفاده است:

```cpp
if (text.contains('W'))
{
    // ...
}
```

این قابلیت در C++23 اضافه شده است.

---

# 30. مقایسه‌ی String Viewها

می‌توان Viewها را با هم مقایسه کرد:

```cpp
std::string_view a = "Hello";
std::string_view b = "World";

if (a == b)
{
    // ...
}
```

عملگرهای مقایسه نیز در دسترس هستند:

```cpp
==
!=
<
<=
>
>=
<=>
```

همچنین تابع:

```cpp
a.compare(b);
```

برای مقایسه‌ی صریح وجود دارد.

---

# 31. `copy()`

اگر واقعاً لازم باشد بخشی از View را داخل یک buffer کپی کنید:

```cpp
char buffer[10];

view.copy(buffer, 5);
```

اینجا برخلاف `substr()`، واقعاً کاراکترها کپی می‌شوند.

پس `string_view` جلوی copy غیرضروری را می‌گیرد، اما اگر خودتان `copy()` را فراخوانی کنید، عملیات کپی انجام خواهد شد.

---

# 32. `swap()`

می‌توان دو View را جابه‌جا کرد:

```cpp
view1.swap(view2);
```

این کار داده‌ی اصلی را جابه‌جا نمی‌کند؛ فقط Viewها و محدوده‌های مورد اشاره‌ی آن‌ها تغییر می‌کنند.

---

# 33. String Literal با پسوند `sv`

در C++ می‌توان از literal مخصوص `string_view` استفاده کرد:

```cpp
using namespace std::literals;

std::string_view text = "Hello"sv;
```

این قابلیت برای کار با literalهای `string_view` طراحی شده است.

---

# 34. یک مثال کامل

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

یک تابع واحد:

```cpp
void analyze(std::string_view text);
```

می‌تواند ورودی‌های مختلفی را دریافت کند:

```cpp
std::string
const char*
string literal
std::string_view
```

---

# 35. کاربرد مهم `string_view` در Parserها

یکی از بهترین کاربردهای `string_view` پردازش متن بدون کپی کردن قسمت‌های مختلف آن است.

فرض کنید ورودی زیر را داریم:

```cpp
std::string input = "name=Ali&age=25";
```

View ایجاد می‌کنیم:

```cpp
std::string_view view = input;
```

جای `=` را پیدا می‌کنیم:

```cpp
auto equal = view.find('=');
```

سپس:

```cpp
auto key = view.substr(0, equal);
```

و:

```cpp
auto value = view.substr(equal + 1);
```

در اینجا:

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

`key` و `value` می‌توانند Viewهایی روی همان داده‌ی اصلی باشند.

این ویژگی برای tokenizerها، parserها، CSV parserها، HTTP parserها، configuration parserها و پردازش فایل‌ها بسیار مفید است.

---

# 36. پردازش رشته بدون ساخت Substring

یکی از الگوهای بسیار کاربردی:

```cpp
std::string_view input = "   Hello World   ";
```

می‌توان محدوده‌ی View را جلو آورد:

```cpp
input.remove_prefix(
    input.find_first_not_of(' ')
);
```

اکنون فضای ابتدای View حذف شده و View از `Hello` شروع می‌شود.

سپس می‌توان فضای انتهایی را نیز با `remove_suffix()` مدیریت کرد.

نکته این است که هیچ characterای از رشته‌ی اصلی حذف نشده است.

فقط محدوده‌ای که View مشاهده می‌کند تغییر کرده است.

---

# 37. `string_view` و تغییر `std::string`

فرض کنید:

```cpp
std::string text = "Hello";

std::string_view view = text;
```

سپس:

```cpp
text += " World";
```

ممکن است `std::string` مجبور شود حافظه‌ی جدیدی تخصیص دهد.

در چنین شرایطی View قبلی ممکن است دیگر معتبر نباشد.

بنابراین این قانون را باید همیشه در نظر گرفت:

> اعتبار `string_view` به اعتبار و lifetime داده‌ی اصلی وابسته است.

هر تغییری در owner که بتواند storage را جابه‌جا یا نابود کند، باید از نظر lifetime و validity بررسی شود.

---

# 38. `string_view` برای ذخیره کردن داده مناسب نیست؛ مگر با مدیریت دقیق Lifetime

فرض کنید کلاسی دارید:

```cpp
class User
{
    std::string_view name;
};
```

این طراحی ذاتاً اشتباه نیست، اما lifetime داده باید کاملاً مشخص باشد.

مثلاً اگر:

```cpp
User user;

std::string name = "Ali";

user.name = name;
```

و بعد `name` از بین برود، `user.name` به داده‌ای اشاره خواهد کرد که دیگر معتبر نیست.

اگر object قرار است **مالک نام** باشد، انتخاب مناسب‌تر معمولاً:

```cpp
class User
{
    std::string name;
};
```

است.

اما اگر object فقط برای مدت مشخصی یک View روی داده‌ی خارجی نگه می‌دارد و lifetime آن داده تضمین شده است، `string_view` می‌تواند انتخاب مناسبی باشد.

---

# 39. تفاوت Ownership و View

این تفاوت را می‌توان به صورت یک قانون ساده به خاطر سپرد:

```text
std::string
    ↓
"I own this data."

std::string_view
    ↓
"I only observe this data."
```

در طراحی API:

### اگر تابع فقط داده را می‌خواند:

```cpp
void parse(std::string_view input);
```

### اگر object باید داده را نگه دارد و مالک آن باشد:

```cpp
class Parser
{
    std::string input;
};
```

---

# 40. `std::string_view` در مقایسه با `const char*`

سه نوع رایج را می‌توان این‌گونه مقایسه کرد:

```text
const char*
    │
    ├── فقط pointer
    ├── معمولاً انتظار null termination
    └── length را همراه خود ندارد


std::string_view
    │
    ├── pointer + size
    ├── null termination لازم نیست
    ├── substring بدون copy
    └── مالک داده نیست


std::string
    │
    ├── مالک داده
    ├── مدیریت lifetime
    ├── size
    ├── قابلیت تغییر
    └── امکان allocation
```

به همین دلیل `string_view` در بسیاری از APIهای مدرن، abstraction مناسب‌تری نسبت به `const char*` برای رشته‌های متنی است.

---

# 41. `string_view` و Null Termination

این نکته آن‌قدر مهم است که ارزش تکرار به صورت یک قانون را دارد:

> `std::string_view` یک C-string نیست.

مثلاً:

```cpp
std::string_view view("Hello");
```

در این حالت داده‌ی اصلی ممکن است null-terminated باشد.

اما:

```cpp
std::string str = "Hello World";

std::string_view view(str.data() + 6, 5);
```

View فقط:

```text
World
```

است.

View الزاماً به یک `'\0'` در انتهای محدوده ختم نمی‌شود.

بنابراین APIهایی که صرفاً:

```cpp
const char*
```

می‌گیرند و انتظار C-string دارند، همیشه نمی‌توانند مستقیماً `string_view.data()` دریافت کنند.

---

# 42. امکانات `std::string_view`

مجموعه‌ی اصلی امکانات `std::string_view` را می‌توان به این شکل دسته‌بندی کرد.

## ساخت و مقداردهی

```cpp
std::string_view()
std::string_view(other)
std::string_view(ptr, count)
std::string_view(c_string)
```

در استانداردهای جدیدتر، constructorهای مرتبط با range نیز اضافه شده‌اند.

---

## اطلاعات و Capacity

```cpp
size()
length()
max_size()
empty()
```

---

## دسترسی به عناصر

```cpp
operator[]()
at()
front()
back()
data()
```

---

## Iteratorها

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

---

## تغییر محدوده‌ی View

```cpp
remove_prefix()
remove_suffix()
swap()
```

---

## Substring و Copy

```cpp
substr()
copy()
```

در استانداردهای جدیدتر قابلیت‌های مرتبط با ایجاد Sub-view نیز توسعه پیدا کرده‌اند.

---

## Search

```cpp
find()
rfind()

find_first_of()
find_last_of()

find_first_not_of()
find_last_not_of()
```

---

## Comparison

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

---

## C++20

```cpp
starts_with()
ends_with()
```

---

## C++23

```cpp
contains()
```

---

## Constant

```cpp
std::string_view::npos
```

---

## Literal

```cpp
"Hello"sv
```

---

## Hash

برای `std::string_view` specialization مربوط به `std::hash` نیز وجود دارد و می‌توان از آن در ساختارهای hash-based مانند:

```cpp
std::unordered_map
std::unordered_set
```

استفاده کرد.

در این حالت نیز باید lifetime داده‌ای که View به آن اشاره می‌کند به دقت مدیریت شود.

---

# 43. نسخه‌ی استاندارد را نیز در نظر بگیرید

بعضی امکانات `string_view` بر اساس نسخه‌ی C++ در دسترس هستند:

| قابلیت                        | استاندارد           |
| ----------------------------- | ------------------- |
| `std::string_view`            | C++17               |
| `starts_with()`               | C++20               |
| `ends_with()`                 | C++20               |
| `<=>`                         | C++20               |
| `contains()`                  | C++23               |
| قابلیت‌های جدید range-related | استانداردهای جدیدتر |

بنابراین هنگام استفاده از قابلیت‌های جدید باید compiler و language standard پروژه را نیز در نظر گرفت.

برای مثال:

```bash
-std=c++20
```

یا:

```bash
-std=c++23
```

---

# 44. چه زمانی `std::string_view` انتخاب مناسبی است؟

`string_view` معمولاً گزینه‌ی مناسبی است وقتی:

* تابع فقط رشته را می‌خواند.
* تابع قرار نیست مالک داده شود.
* می‌خواهید string literal را مستقیماً دریافت کنید.
* می‌خواهید از ساخت `std::string` موقت جلوگیری کنید.
* substringهای زیادی پردازش می‌کنید.
* parser یا tokenizer می‌نویسید.
* می‌خواهید allocation و copy غیرضروری را کاهش دهید.
* API عمومی‌ای دارید که باید چند نوع منبع رشته‌ای را بپذیرد.

---

# 45. چه زمانی `std::string` مناسب‌تر است؟

وقتی:

* object باید مالک داده باشد.
* lifetime داده باید مستقل از منبع اصلی باشد.
* قرار است رشته ذخیره شود.
* قرار است رشته تغییر کند.
* داده‌ی اصلی ممکن است قبل از object از بین برود.
* نمی‌توانید lifetime داده‌ی مورد اشاره را تضمین کنید.

در چنین شرایطی:

```cpp
std::string
```

انتخاب طبیعی‌تری است.

---

# 46. یک چک‌لیست ذهنی برای استفاده از `string_view`

هر زمان خواستید از `std::string_view` استفاده کنید، این چند سؤال را در نظر بگیرید:

### 1. آیا تابع مالک داده می‌شود؟

اگر خیر:

```cpp
std::string_view
```

گزینه‌ی خوبی است.

### 2. آیا داده تا زمانی که View استفاده می‌شود زنده می‌ماند؟

اگر خیر، View می‌تواند dangling شود.

### 3. آیا داده ممکن است reallocate شود؟

اگر بله، اعتبار View باید دوباره بررسی شود.

### 4. آیا API مقصد به null-terminated string نیاز دارد؟

اگر بله، صرفاً داشتن `string_view.data()` کافی نیست.

### 5. آیا قرار است داده ذخیره شود؟

اگر بله، معمولاً باید به ownership و در نتیجه `std::string` فکر کرد.

---

# 47. جمع‌بندی

`std::string_view` را می‌توان یکی از ابزارهای کلیدی C++ مدرن برای **پردازش رشته بدون مالکیت و بدون کپی غیرضروری** دانست.

مدل ذهنی اصلی:

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

معمولاً انعطاف زیادی ایجاد می‌کند و ورودی‌هایی مانند:

```cpp
std::string
const char*
string literal
std::string_view
```

را پوشش می‌دهد.

مهم‌ترین مزیت‌های آن عبارت‌اند از:

```text
عدم نیاز به مالکیت داده
عدم کپی کاراکترها هنگام ایجاد View
substring بدون ساخت string جدید
کاهش allocationهای غیرضروری
API ساده‌تر
مناسب برای parser و tokenizer
جستجو و مقایسه‌ی مستقیم
پشتیبانی از string literal
```

اما در کنار این مزایا، یک اصل بسیار مهم وجود دارد:

```text
string_view → مالک داده نیست
```

بنابراین lifetime داده‌ی اصلی باید همیشه معتبر بماند.

اگر این مفهوم به‌درستی درک شود، `std::string_view` فقط یک کلاس برای کار با رشته نیست؛ بلکه نمونه‌ی بسیار خوبی از یکی از اصول مهم طراحی در C++ مدرن است:

> **مالکیت داده را از مشاهده و پردازش داده جدا کنید.**
