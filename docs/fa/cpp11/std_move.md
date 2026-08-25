<div align="right">

[🇺🇸 English](../../en/cpp11/std_move.md) | [🇮🇷 فارسی](./std_move.md)

</div>
---

# مبحث Move Semantics، `std::move` و مفاهیم `lvalue` و `rvalue` در C++

**نسخه معرفی:** C++11

## مقدمه

مفهوم **Move Semantics** یکی از مهم‌ترین قابلیت‌های C++11 است. این قابلیت اجازه می‌دهد در بسیاری از موارد، به‌جای کپی کردن پرهزینه اشیاء، مالکیت منابع آن‌ها منتقل شود. در نتیجه سرعت اجرا بالاتر می‌رود و حافظه کمتری مصرف می‌شود.

برای درک بهتر Move Semantics، آشنایی با مفاهیمی مثل `lvalue`، `rvalue` و rvalue reference بسیار مهم است.

---

## مقادیر `lvalue` و `rvalue` یعنی چه؟

به زبان ساده:

- **`مقادیر نوع lvalue`:** عبارتی که به یک شیء با هویت مشخص در حافظه اشاره دارد و معمولاً می‌توان بعداً نیز به آن دسترسی داشت.
- **`مقادیر نوع rvalue`:** مقدار یا شیء موقتی که معمولاً فقط در همان عبارت وجود دارد و عمر کوتاهی دارد.

مثال:

```cpp
int x = 10;
```

در این مثال:

- متغیر `x` یک `lvalue` است.
- داده `10` یک `rvalue` است.

مثلاً:

```cpp
x = 20; // درست
```

چون `x` یک شیء با مکان مشخص در حافظه است. اما:

```cpp
10 = x; // غلط
```

چون `10` فقط یک مقدار است و محل ذخیره‌سازی قابل انتساب ندارد.

مثال دیگر:

```cpp
int a = 3;
int b = a;
```

در اینجا:

- متغیر `a` یک `lvalue` است.
- متغیر `b` یک `lvalue` است.
- داده `3` یک `rvalue` است.

---

## نتیجه تابع

اگر تابعی مقدار را by value برگرداند، نتیجه آن معمولاً یک `rvalue` است:

```cpp
int foo()
{
    return 5;
}
```

در این حالت:

```cpp
foo();
```

یک `rvalue` محسوب می‌شود، چون نتیجه تابع یک مقدار موقتی است.

اما اگر تابع مرجع برگرداند، نتیجه می‌تواند `lvalue` باشد:

```cpp
int x = 0;

int& foo()
{
    return x;
}
```

در این حالت:

```cpp
foo();
```

یک `lvalue` است، چون به یک شیء واقعی ارجاع می‌دهد.

---

## یک راه ساده برای تشخیص

مثلاً:

```cpp
int x = 5;
```

اگر بتوانیم از عبارت آدرس بگیریم، معمولاً با یک `lvalue` سروکار داریم:

```cpp
&x; // معمولاً درست
```

اما:

```cpp
&(x + 1); // غلط
```

چون `x + 1` یک مقدار موقتی است.

> نکته: در زبان C++ مدرن، دسته‌بندی دقیق‌تری وجود دارد: `lvalue`، `prvalue` و `xvalue`.  
> برای درک اولیه Move Semantics، همان تفکیک ساده `lvalue` و `rvalue` کافی است، اما بدانید که `std::move(x)` دقیقاً یک `xvalue` تولید می‌کند.

---

## چرا این تقسیم‌بندی وجود دارد؟

کامپایلر نیاز دارد بداند:

- آیا این شیء بعداً هم استفاده می‌شود؟
- یا فقط یک مقدار موقتی است و می‌توان از آن صرف‌نظر کرد؟

این اطلاعات برای بهینه‌سازی بسیار مهم است.

---

## قبل از C++11 چه مشکلی وجود داشت؟

فرض کنید:

```cpp
std::string s = "Hello";
```

و تابعی داریم که پارامتر را by value می‌گیرد:

```cpp
void print(std::string str) {}
```

اگر بنویسیم:

```cpp
print(s);
```

یک کپی از `s` برای ساختن `str` ایجاد می‌شود.

به صورت مفهومی:

```text
s
 ↓ copy
str
```

حالا اگر یک مقدار موقتی بفرستیم:

```cpp
print("Hello");
```

باز هم ممکن بود یک شیء موقتی ساخته و سپس کپی شود، در حالی که آن شیء موقتی دیگر قرار نیست بعداً استفاده شود. این باعث اتلاف حافظه و زمان می‌شود.

> در C++11 و بعد، با استفاده از move semantics و همچنین copy elision، بسیاری از این کپی‌ها حذف یا کاهش پیدا می‌کنند.

---

## ایده Move Semantics چیست؟

ایده ساده است:

> اگر شیء موقتی است یا دیگر نیازی به آن نداریم، به‌جای کپی کردن منابعش، منابع آن را منتقل کن.

مثال مفهومی برای کلاسی که حافظه پویا دارد:

```text
Copy:

A ------copy------> B

A -> data
B -> copy of data
```

یعنی دو حافظه جداگانه لازم داریم.

اما Move:

```text
Move:

A ----move----> B

B -> data
A -> resource released / empty / safe state
```

در move، معمولاً فقط اشاره‌گر یا مالکیت منبع منتقل می‌شود و داده اصلی کپی نمی‌شود.

> نکته: بعد از move، شیء مبدأ لزوماً `nullptr` یا خالی نیست. استاندارد برای بسیاری از کلاس‌های کتابخانه‌ای می‌گوید شیء مبدأ باید «معتبر ولی نامشخص» باشد.

---

## مفهوم Rvalue Reference

در C++11 نوع جدیدی از مرجع معرفی شد:

```cpp
&&
```

به آن **rvalue reference** می‌گویند.

مثال:

```cpp
void foo(std::string&& s) {}
```

این تابع می‌تواند rvalueها را بپذیرد.

مثلاً:

```cpp
foo(std::string("Hello")); // درست
```

یا:

```cpp
foo("Hello"); // درست
```

در حالت دوم، یک `std::string` موقتی از `"Hello"` ساخته می‌شود و به `foo` پاس داده می‌شود.

اما:

```cpp
std::string s = "Hello";
foo(s); // غلط
```

چون `s` یک `lvalue` است و نمی‌توان آن را مستقیم به `std::string&&` متصل کرد.

برای این کار می‌توان نوشت:

```cpp
foo(std::move(s)); // درست
```

---

## نکته مهم درباره پارامتر `std::string&&`

داخل بدنه تابع، خود پارامتر `s` یک `lvalue` است، چون نام دارد و آدرس آن قابل اشاره است.

مثال:

```cpp
void foo(std::string&& s)
{
    //متغیر s در اینجا یک lvalue است،
    // با وجود اینکه نوع آن std::string&& است.

    std::string local = std::move(s); // درخواست move از s
}
```

یعنی اگر بخواهید از یک rvalue reference داخل تابع move کنید، معمولاً باید دوباره از `std::move` استفاده کنید.

---

## دستور `std::move` چیست؟

فرض کنید:

```cpp
std::string a = "Hello";
```

اگر بنویسیم:

```cpp
std::move(a)
```

این دستور به‌تنهایی چیزی را جابه‌جا نمی‌کند.

`std::move` در واقع یک **cast** است که می‌گوید:

> با این عبارت مثل یک rvalue رفتار کن.

به بیان دقیق‌تر، `std::move` یک rvalue reference از شیء برمی‌گرداند:

```cpp
std::string b = std::move(a);
```

در این حالت، اگر move constructor با ورودی `std::string` داشته باشید، ممکن است عملیات move انجام شود.

بعد از این عملیات:

```text
b -> resource moved from a
a -> valid but unspecified
```

یعنی `a` هنوز یک شیء معتبر است، اما مقدار دقیق آن مشخص نیست و نباید روی محتوای قبلی آن حساب کرد.

> نکته مهم: `std::move` فقط «امکان» move را فراهم می‌کند. اگر move constructor یا move assignment مناسبی وجود نداشته باشد، ممکن است دوباره copy انجام شود.

---

## مفهوم Move Constructor

فرض کنید کلاسی داریم که منبع پویا مدیریت می‌کند:

```cpp
class Buffer
{
    int* data_ = nullptr;
    std::size_t size_ = 0;

public:
    ~Buffer()
    {
        delete[] data_;
    }

    // Move Constructor
    Buffer(Buffer&& other) noexcept
        : data_(other.data_), size_(other.size_)
    {
        other.data_ = nullptr;
        other.size_ = 0;
    }

    // Move Assignment Operator
    Buffer& operator=(Buffer&& other) noexcept
    {
        if (this != &other)
        {
            delete[] data_;

            data_ = other.data_;
            size_ = other.size_;

            other.data_ = nullptr;
            other.size_ = 0;
        }

        return *this;
    }
};
```

وقتی بنویسیم:

```cpp
Buffer a;
Buffer b = a; // Copy Constructor
```

اگر copy constructor تعریف شده باشد، عملیات copy انجام می‌شود.

اما:

```cpp
Buffer b = std::move(a); // Move Constructor
```

در این حالت، اگر move constructor وجود داشته باشد، منابع `a` به `b` منتقل می‌شوند.

---

## عملیات Move Assignment

همین ایده برای عملگر انتساب نیز وجود دارد.

Copy Assignment:

```cpp
a = b;
```

Move Assignment:

```cpp
a = std::move(b);
```

در Move Assignment، منابع شیء مقصد آزاد یا جایگزین می‌شوند و منابع شیء منبع به مقصد منتقل می‌شوند.

---

## چرا Move سریع‌تر است؟

فرض کنید کلاسی داریم که یک بافر بزرگ را مدیریت می‌کند:

```cpp
class Buffer
{
    int* data_;
};
```

اگر اندازه بافر مثلاً 10,000,000 عدد باشد، copy یعنی:

1. تخصیص حافظه جدید
2. کپی کردن 10 میلیون عدد

اما move می‌تواند فقط چند اشاره‌گر یا مقدار داخلی را جابه‌جا کند:

```cpp
data_ = other.data_;
other.data_ = nullptr;
```

در چنین حالتی، پیچیدگی عملیات از حدود `O(n)` به `O(1)` کاهش پیدا می‌کند.

> نکته: برای اشیاء کوچک یا trivially copyable، ممکن است copy به اندازه کافی سریع باشد و move سود چشمگیری نداشته باشد.

---

## یک مثال واقعی

```cpp
std::vector<int> v1(1000000);
std::vector<int> v2 = std::move(v1);
```

بدون move:

- تخصیص حافظه جدید
- کپی کردن 1,000,000 عدد

با move:

- انتقال اشاره‌گر داخلی
- انتقال size
- انتقال capacity

در پیاده‌سازی‌های معمول، این عملیات بسیار سریع است.

بعد از move:

- آرایه `v2` داده‌های قبلی `v1` را دارد.
- آرایه `v1` در وضعیت معتبر ولی نامشخص قرار دارد.

> نکته مهم: Move Semantics فقط زمانی امن است که شیء مبدأ دیگر نیازی نباشد. به همین دلیل C++ به‌طور خودکار فقط برای rvalueها یا عباراتی که با `std::move` به rvalue تبدیل شده‌اند، عملیات move را انتخاب می‌کند.

---

## انواع پارامترها

| نوع پارامتر | مثال | توضیح |
|---|---|---|
| ارسال مقدار | `void foo(std::string s);` | پارامتر با copy یا move مقداردهی می‌شود. برای lvalue معمولاً copy و برای rvalue معمولاً move یا copy elision رخ می‌دهد. |
| مرجع به lvalue غیرثابت | `void foo(std::string& s);` | فقط lvalueهای غیرثابت را می‌پذیرد. |
| مرجع ثابت | `void foo(const std::string& s);` | هم lvalue و هم rvalue را می‌پذیرد، اما نمی‌توان از طریق آن شیء را تغییر داد. |
| مرجع به rvalue | `void foo(std::string&& s);` | مقادیر rvalueها را می‌پذیرد و پایه‌ای برای Move Semantics است. |

---

## فرق Copy و Move

### Copy Constructor

```cpp
std::string b = a;
```

در اینجا یک شیء جدید با کپی از `a` ساخته می‌شود.

### Move Constructor

```cpp
std::string b = std::move(a);
```

در اینجا یک شیء جدید با انتقال منابع `a` ساخته می‌شود، البته اگر move constructor مناسب وجود داشته باشد.

### Copy Assignment Operator

```cpp
b = a;
```

محتوای `a` در `b` کپی می‌شود.

### Move Assignment Operator

```cpp
b = std::move(a);
```

منابع `a` به `b` منتقل می‌شود و معمولاً `a` در وضعیت معتبر ولی نامشخص قرار می‌گیرد.

---

## اشتباهات رایج

### 1. فکر کنیم `std::move` حتماً جابه‌جا می‌کند

دستور `std::move` به‌تنهایی هیچ داده‌ای را جابه‌جا نمی‌کند. این دستور فقط عبارت را به شکل rvalue درمی‌آورد.

```cpp
std::move(a); // فقط cast است
```

عملیات move زمانی اتفاق می‌افتد که move constructor یا move assignment مناسب فراخوانی شود.

---

### 2. فکر کنیم شیء move شده حتماً خالی یا `nullptr` است

بعد از:

```cpp
std::string b = std::move(a);
```

نباید فرض کنیم `a` حتماً خالی است. برای بسیاری از کلاس‌های استاندارد، وضعیت `a` معتبر ولی نامشخص است.

می‌توان معمولاً آن را destroy کرد یا مقدار جدیدی به آن assign کرد، اما نباید روی محتوای قبلی آن حساب کرد.

---

### 3. استفاده مجدد از شیء بعد از move

بعد از move، نباید از شیء مبدأ مثل قبل استفاده کنید.

```cpp
std::string a = "Hello";
std::string b = std::move(a);

```

استفاده از a بعد از move ممکن است از نظر منطقی اشتباه باشد،
مگر اینکه فقط عملیات‌های امن مثل assign جدید انجام شود.

---

### 4. استفاده از `std::move` روی شیء `const`

```cpp
const std::string a = "Hello";
std::string b = std::move(a);
```

در این حالت معمولاً move انجام نمی‌شود، زیرا move constructor معمولاً یک `std::string&&` غیرثابت می‌گیرد. در نتیجه ممکن است copy انجام شود.
به بیان بهتر چون در move constructor و move assignment قرار است مقدار داده اول پس از انتقال نا معتبر شود، معمولا ورودی این دو مورد را با شی rvalue reference && غیر const تعریف می کنند و ارسال داده const باعث می شود احتمالا این دو مورد move صدا زده نشوند و copy constructor یا copy assignment صدا زده شود!

---

### 5. استفاده از `return std::move(local)`

معمولاً بهتر است این کار را نکنید:

```cpp
std::string make()
{
    std::string s = "Hello";
    return s; // خوب
}
```

نه:

```cpp
std::string make()
{
    std::string s = "Hello";
    return std::move(s); // معمولاً توصیه نمی‌شود
}
```

دلیل آن این است که `return std::move(s)` ممکن است جلوی بهینه‌سازی‌هایی مثل NRVO را بگیرد.

---

### 6. فراموش کردن `noexcept`

اگر کلاس شما move constructor یا move assignment دارد، بهتر است آن‌ها را `noexcept` اعلام کنید:

```cpp
Buffer(Buffer&& other) noexcept;
Buffer& operator=(Buffer&& other) noexcept;
```

بسیاری از کانتینرها، مثل `std::vector`، فقط در صورتی از move در reallocation استفاده می‌کنند که move constructor `noexcept` باشد یا شرایط خاصی برقرار باشد. در غیر این صورت ممکن است برای حفظ امنیت استثنا، copy انجام شود.

---

## جمع‌بندی

| مفهوم | توضیح |
|---|---|
| `lvalue` | عبارت دارای هویت مشخص که معمولاً می‌توان به آن ارجاع داد و آدرسش را گرفت. |
| `rvalue` | مقدار یا شیء موقتی که معمولاً عمر کوتاهی دارد و می‌توان از آن عملیات Move انجام داد. |
| `prvalue` | مقدار موقتی بدون هویت؛ مانند `5` یا نتیجهٔ تابعی که مقدار را با Value برمی‌گرداند. |
| `xvalue` | مقدار در حال انقضا؛ مانند نتیجهٔ `std::move(x)`. |
| `&&` | عملگر **rvalue reference** که پایهٔ Move Semantics محسوب می‌شود. |
| `std::move` | تابعی که عبارت را به **rvalue reference** تبدیل می‌کند و به‌تنهایی عملیاتی برای Move انجام نمی‌دهد. |
| Move Constructor | سازنده‌ای که شیء جدید را با انتقال منابع از یک شیء موجود ایجاد می‌کند. |
| Move Assignment | عملگر انتسابی که منابع شیء مقصد را با منابع شیء مبدأ جایگزین می‌کند، بدون انجام Copy پرهزینه. |
| Copy Constructor | سازنده‌ای که شیء جدید را با کپی کردن شیء موجود ایجاد می‌کند. |
| Copy Assignment | عملگر انتسابی که محتوای شیء مقصد را با کپی از شیء مبدأ جایگزین می‌کند. |
| Moved-from object | شیئی که عملیات Move روی آن انجام شده و معمولاً معتبر است، اما مقدار آن مشخص نیست. |

---

## منابع

- https://en.cppreference.com/w/cpp/utility/move
- https://en.cppreference.com/w/cpp/language/move_constructor
- https://en.cppreference.com/w/cpp/language/value_category

---
## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>